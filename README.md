# TakeMeter

A fine-tuned text classifier that sorts comments from **r/nba** by *what kind of
discourse* they are: reasoned **analysis**, an evidence-free **hot take**, or a
raw emotional **reaction**. The goal is to measure the *form* of a take, not
whether it's "good" — a distinction r/nba users argue about constantly.

This README is self-contained. Deeper design notes (label stress-tests,
annotation decisions, the full edge-case rulebook) live in
[planning.md](planning.md).

---

## Community

**r/nba** — the main NBA discussion subreddit. I chose it because the
quality-of-discourse question is something the community polices *explicitly*
("that's a hot take," "great breakdown"), and a single post-game thread contains
the full range: a 200-word tactical breakdown sitting next to "WE WON 😭" and a
confident one-liner like "Jokic is washed." That spread is what makes the
classification task meaningful.

## Labels

Three mutually exclusive labels, ordered by how much they argue from evidence.

| Label | Definition |
|-------|------------|
| `analysis` | A structured argument backed by **specific, verifiable evidence** (stats, a named source, a historical comparison backed by specifics, or concrete tactical observation). If you removed the opinion, the evidence would still stand on its own. |
| `hot_take` | An **evaluative claim asserted without verifiable evidence**, regardless of tone. Any "evidence" is vague, cherry-picked, or decorative. |
| `reaction` | An **immediate emotional response to a specific event** — venting a feeling in the moment, with little or no argument. |

**Examples:**
- `analysis`: *"Embiid is awful on defense against this team. Everyone can shoot, you can't play drop coverage on Brunson, and once he's out of the paint there's no rim protection."*
- `hot_take`: *"The 2026 Nuggets couldn't even beat an insanely injured Timberwolves team. They got cooked by Ayo Dosunmu."*
- `reaction`: *"Almost fucking choked it, my heart."*

The hardest boundaries — and the explicit tie-breaker rules I used (e.g.
emotional-venting-with-a-claim, decorative stats, sarcasm about non-game events)
— are documented in [planning.md](planning.md#hard-edge-cases).

## Dataset

- **Source:** Public comments from r/nba, collected manually by reading whole
  threads and copying comment text. I sampled deliberately across thread types,
  because each skews toward one label: **post-game / game threads** → `reaction`;
  **discussion / ranking / "give me your take" threads** → `hot_take`; **long
  text / OC posts and r/nbadiscussion** → `analysis`. Comments span ~14 different
  games across both conferences (regular season, play-in, and playoffs) so the
  model isn't memorizing one team's vocabulary.
- **Labeling process:** Each comment was read individually and labeled with the
  definitions above. An LLM was used to *suggest* labels for collected comments,
  which were then reviewed (see [AI Usage](#ai-usage)); borderline cases were
  flagged in a `notes` column with the deciding rule. Out-of-scope text (news
  drops, pure jokes, off-topic banter, meta-commentary about the subreddit) was
  excluded rather than forced into a label.
- **Total examples:** 163

  | Label | Count | Share |
  |-------|-------|-------|
  | `analysis` | 58 | 36% |
  | `hot_take` | 48 | 29% |
  | `reaction` | 57 | 35% |

  No class exceeds 70% and each is above 20%, so the model can't win by always
  guessing one label.

- **Train / validation / test split:** 70% / 15% / 15% (≈114 / 24 / 25),
  produced automatically by the notebook. The test set used for all metrics below
  is 25 examples (9 `analysis`, 7 `hot_take`, 9 `reaction`).

### 3 examples that were genuinely hard to label

1. **"The 2026 Nuggets would beat the healthy 2026 Timberwolves… [long matchup
   argument]"** — the author *literally called it a hot take*, but it argues from
   specific matchup reasoning (Gordon/Watson's defensive impact, the Murray–Jokic
   two-man game). **Decision → `analysis`.** Rule: classify by *how it reasons*,
   not what the author calls it.
2. **"Extending the series — 22 FTs to 4 in the second half 😂😂"** — contains a
   real stat, which looks like evidence. But the number is used for emotional
   effect (mocking the refs), not to reason toward a conclusion. **Decision →
   `reaction`.** Rule: a decorative stat doesn't make it analysis; the dominant
   function is venting.
3. **"Traded 5 firsts for someone who had more turnovers (4) than total points,
   rebounds and assists combined (0/1/2)."** — a snarky one-liner that *feels*
   like a hot take, but the specific, verifiable stats genuinely carry the
   argument. **Decision → `analysis`** (over `hot_take`). Rule: if real evidence
   does the argumentative work, it's analysis even when the tone is dismissive.

## Model & Training

- **Base model:** `distilbert-base-uncased` (HuggingFace).
- **Training approach:** Sequence classification head fine-tuned on the 3-label
  task using the `transformers` Trainer on a Colab T4 GPU. Training ran in well
  under the expected window.
- **Key hyperparameter decision:** I kept the defaults — **3 epochs, learning
  rate 2e-5, batch size 16**. Reasoning: with only ~114 training examples,
  longer/heavier training risks overfitting a tiny dataset, and 2e-5 is the
  standard stable fine-tuning rate for DistilBERT. (In hindsight the near-random
  prediction confidences suggest the model actually *underfit* — see the
  reflection — so more epochs is the first thing I'd change next.)

## Baseline

- **Zero-shot model:** Groq `llama-3.3-70b-versatile`, no task-specific training.
- **Prompt:** the full prompt is in [baseline_prompt.txt](baseline_prompt.txt).
  It states the task, defines each label, gives one example per label, includes
  the key tie-breaker rules, and instructs the model to output **only** the label
  name. On the test set, **0% of responses were unparseable.**

---

## Evaluation Report

### Overall accuracy

| Model | Accuracy | Macro-F1 |
|-------|----------|----------|
| Groq zero-shot (baseline) | **0.84** | 0.83 |
| Fine-tuned DistilBERT | **0.52** | 0.46 |

Fine-tuning **regressed by 0.32**. The fine-tuned model lost decisively to the
zero-shot baseline — a result worth diagnosing honestly (below), not hiding.

### Per-class metrics

**Groq zero-shot (baseline):**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| `analysis` | 1.00 | 0.78 | 0.88 | 9 |
| `hot_take` | 0.71 | 0.71 | 0.71 | 7 |
| `reaction` | 0.82 | 1.00 | 0.90 | 9 |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|-------|-----------|--------|----|---------|
| `analysis` | 0.69 | 1.00 | 0.82 | 9 |
| `hot_take` | 0.14 | 0.14 | 0.14 | 7 |
| `reaction` | 0.60 | 0.33 | 0.43 | 9 |

### Confusion matrix — fine-tuned model

Rows = true label, columns = predicted. (Supplementary image:
[outputs/confusion_matrix.png](outputs/confusion_matrix.png).)

| true ↓ \ pred → | analysis | hot_take | reaction |
|-----------------|----------|----------|----------|
| **analysis** | 9 | 0 | 0 |
| **hot_take** | 4 | 1 | 2 |
| **reaction** | 0 | 6 | 3 |

Two error cells dominate: **true `reaction` → predicted `hot_take` (6)** and
**true `hot_take` → predicted `analysis` (4)**. `hot_take`, the middle class,
effectively dissolved — only 1 of 7 was caught — while `analysis` became a sink
(predicted for all 9 real analysis *plus* 4 hot takes).

### Error pattern analysis

I pasted all 12 misclassifications into an LLM and asked it to surface common
themes, then re-read each example to verify. The patterns that held up:

- **Directional, not random.** Errors cluster on two boundaries: `reaction`→
  `hot_take` and `hot_take`→`analysis`. There are **zero** `analysis`
  misclassifications and **zero** `analysis↔reaction` confusions — the model
  separates the two *extremes* fine but can't locate the middle.
- **The model learned surface features, not evidence.** Comments that name
  players or make comparisons ("LeBron beyond Jordan, Kobe…") got pulled to
  `analysis` even with zero evidence; declarative opinion sentences got pulled to
  `hot_take`; pure emotion got read as opinion.
- **Near-random confidence.** Every misclassification fired at ~0.33–0.37
  confidence (random for 3 classes = 0.33). The model barely committed to
  anything — a signature of underfitting on too little data.
- *Discarded pattern:* the LLM initially suggested "short posts fail more," but
  re-reading disproved it — short reactions like "Time to blow it up, Warriors"
  and long ones both failed, so length wasn't the driver. The driver is the
  label *boundary*, not post length.

### 3 specific failures, analyzed

1. **#9 — "The 2026 Nuggets couldn't even beat an insanely injured Timberwolves
   team. They got cooked by Ayo Dosunmu."** True `hot_take`, predicted `analysis`
   (0.35). *Why:* it name-drops a team and a player, and the model learned that
   specific nouns ≈ evidence. But there's no actual reasoning — it's a dismissive
   claim. **Boundary:** `analysis↔hot_take`. **Cause:** data/boundary, not
   labeling — I labeled this consistently; the model just keys on surface
   specifics instead of whether evidence supports the claim.
2. **#3 — "This game was a blast. Trash v trash… I had a great time."** True
   `reaction`, predicted `hot_take` (0.35). *Why:* it's pure in-the-moment
   emotion with no claim, but the model can't isolate "venting" as its own
   category and defaults emotional-but-declarative sentences to `hot_take`.
   **Boundary:** `reaction↔hot_take` — the single biggest error cell (6 of 9
   reactions). **Cause:** too few `reaction` examples for the model to carve out a
   distinct region.
3. **#11 — "LeBron is a class act beyond Jordan, Kobe, Magic or Kareem…"** True
   `hot_take`, predicted `analysis` (0.37). *Why:* a bare ranking with no support
   — but four legendary names read as "analysis-like" to the model. This is the
   clearest case of it rewarding *name density* over *reasoning*. **Boundary:**
   `analysis↔hot_take`. **Cause:** the boundary itself + insufficient data.

**What would fix it:** (a) far more training data — ~38 examples per class is too
few for a 66M-parameter model to learn a subjective 3-way split; (b) more
`hot_take` and `reaction` examples specifically, since those are the confused
classes; (c) deliberately oversampling the hard cases (evidence-free name-drops,
long emotional rants) so the model sees that specifics ≠ analysis and length ≠
argument; (d) more training epochs to address the underfitting the confidences
revealed.

### Sample classifications (fine-tuned model)

Each row is a real test comment run through the fine-tuned model, with its
predicted label and confidence.

| Comment | Predicted | Confidence | Correct? |
|---------|-----------|------------|----------|
| "GGs Wemby. The guy is 22 and still very raw offensively. If he stays healthy teams better get ready to see this guy until May/June every year." | `analysis` | 0.34 | ❌ (true `hot_take`) |
| "Time to blow it up, Warriors" | `reaction` | 0.36 | ❌ (true `hot_take`) |
| "This game was a blast. Trash v trash… I had a great time." | `hot_take` | 0.35 | ❌ (true `reaction`) |
| "Embiid is awful on defense against this team. Everyone can shoot, you can't play drop coverage on Brunson, and once he's out of the paint there's no rim protection." | `analysis` | **‹paste confidence›** | ✅ |

> **Why the correct one is reasonable:** this comment was correctly predicted
> `analysis` because it does exactly what the label is meant to capture — it makes
> a concrete, verifiable tactical claim (the spacing/drop-coverage problem) and
> reasons toward a conclusion, rather than just asserting an opinion. It's the one
> boundary the model handles well: clear evidence-bearing structure reads as
> `analysis` (the confusion matrix shows zero `analysis` errors).
>
> _⚠️ One number to fill: run this comment through the classifier cell in the
> notebook and paste its confidence score above where it says ‹paste confidence›.
> If for any reason the model doesn't predict `analysis` on it, swap in another
> clearly evidence-based comment that it gets right._

### Reflection: what the model captured vs. what I intended

I intended the model to learn **"is there real evidence behind this take?"** —
the line between reasoning (`analysis`), assertion (`hot_take`), and feeling
(`reaction`). What it actually learned was a much shallower **surface-feature**
proxy: *named entities and comparisons → analysis, declarative sentences →
hot_take, and everything emotional collapses toward hot_take.* It captured the
two extremes (the confusion matrix shows zero `analysis`↔`reaction` errors) but
completely missed the middle — `hot_take` requires understanding *absence of
evidence*, which is far subtler than spotting the *presence* of stat-words, and
the model defaulted to keyword-spotting instead.

The gap is most visible in the near-random confidences: the model never built a
confident decision boundary at all. With ~114 examples it underfit, and on a
genuinely subjective task it fell back on the easiest correlations in the data
rather than the concept I was trying to teach. The honest takeaway: a strong
general LLM (Groq, 0.84) already "understands" this distinction from broad
pretraining, while a small model fine-tuned on a tiny, subjective dataset learns
a caricature of it. Fine-tuning was not free — it actively made things worse
here, which is itself the most useful thing this project taught me.

---

## Spec Reflection

- **How the spec helped:** the requirement to define **mutually exclusive,
  exhaustive, community-grounded** labels *before* annotating forced me to
  pressure-test the taxonomy (I used an LLM to generate boundary cases, found four
  weak spots, and wrote explicit tie-breaker rules) instead of discovering
  ambiguity halfway through 163 labels. The "baseline before fine-tuning"
  requirement is what made the regression *legible* — without the 0.84 reference
  point, 0.52 would have looked like a number rather than a clear failure to beat
  a general model.
- **How my implementation diverged:** the spec targets **200+ examples** and a
  fine-tuned model that is a *usable* improvement. I locked the dataset at **163**
  (instructor-approved) and the fine-tuned model *regressed* rather than improved.
  So the deliverable diverged from "a working classifier" into "a documented,
  diagnosable failure mode" — which the spec explicitly treats as a valid outcome
  to investigate and report.

## AI Usage

> **Note:** edit this section so it matches exactly what *you* did, especially the
> degree of human review during annotation — keep it honest.

1. **Label stress-testing (before annotating).** I directed an LLM to generate
   8–10 r/nba comments sitting on the `hot_take`↔`reaction` and
   `analysis`↔`hot_take` boundaries and to classify each with my definitions. It
   produced cases I couldn't classify cleanly, which exposed four weaknesses: my
   `hot_take` definition over-indexed on *tone*, "historical comparison" wrongly
   implied analysis, sarcasm about non-game events had no rule, and mixed posts
   had no tie-breaker. **I changed the definitions** in response (see
   [planning.md](planning.md)) before labeling.
2. **Annotation assistance (disclosed — significant).** This was heavy AI
   involvement and I want to be precise about it. I selected the community, chose
   and pasted the source threads, and set the label definitions and edge-case
   rules. An LLM then did the bulk of the annotation work: it extracted the usable
   comments from each pasted thread, discarded out-of-scope text, and **assigned
   the label for essentially all 163 examples**, recording the deciding rule for
   borderline cases in the `notes` column. Every row is marked `prelabeled = ai`.
   I directed the process and reviewed/approved labels as batches were added,
   including questioning specific borderline calls, but I did not independently
   re-label every example from scratch — the AI's first-pass labels stand for most
   rows, with the genuinely ambiguous cases flagged for review in the `notes`
   column. The taxonomy, edge-case rules, and source-thread selection are mine; the
   first-pass labels were AI-generated.
3. **Failure analysis.** I pasted the 12 misclassifications into an LLM and asked
   it to surface common themes. It proposed a "short posts fail more" pattern,
   which I **rejected** after re-reading the examples (both short and long posts
   failed); I kept the directional-boundary pattern, which the confusion matrix
   independently confirmed.

## How to Run

1. Open the TakeMeter Colab notebook and set **Runtime → T4 GPU**.
2. Add your Groq API key via Colab Secrets (`GROQ_API_KEY`).
3. **Section 1:** set `LABEL_MAP = {"analysis":0, "hot_take":1, "reaction":2}` and
   upload [data/dataset.csv](data/dataset.csv).
4. **Section 2:** run the split + tokenization.
5. **Section 5:** paste the prompt from [baseline_prompt.txt](baseline_prompt.txt)
   and run the Groq baseline.
6. **Section 3:** fine-tune DistilBERT.
7. **Section 4:** evaluate + generate the confusion matrix.
8. **Section 6:** print the comparison and write `evaluation_results.json`.
9. Download `evaluation_results.json` and `confusion_matrix.png` into
   [outputs/](outputs/).

Committed results: [outputs/evaluation_results.json](outputs/evaluation_results.json),
[outputs/confusion_matrix.png](outputs/confusion_matrix.png).
