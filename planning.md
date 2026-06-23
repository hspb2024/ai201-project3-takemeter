# TakeMeter — Planning

## Community

**r/nba** — the main NBA discussion subreddit (~16M members), plus game threads
and post-game threads where comment volume is highest.

I chose r/nba because the "good take vs. bad take" question is something the
community argues about *explicitly* — users constantly accuse each other of
having "hot takes" or praise a "great breakdown." The discourse is unusually
varied in quality: in a single post-game thread you'll find a 300-word tactical
breakdown of a pick-and-roll coverage right next to "WE WASHED 🔥🔥" and a
confident one-liner like "Jokic is already a top-5 center ever." That spread is
exactly what makes the classification task interesting — the labels correspond
to real distinctions regulars already make.

## Labels

Three mutually exclusive labels, ordered from most to least argued-from-evidence.

### `analysis`
A post that makes a structured argument backed by **specific, verifiable
evidence** — stats, a historical comparison *backed by specific facts*, or
concrete tactical observation. If you removed the opinion framing, the evidence
would still stand on its own. Note: a bare comparison with no supporting specifics
("this team is more dominant than the '17 Warriors, no question") is **not**
analysis — it's an unsupported claim → `hot_take`.

- Example 1: *"More drives do NOT lead to more free throws. [links NBA.com drives &
  FTA data] If you run a regression the r² is just 0.01 — variance in free throws is
  essentially unexplained by drives, and they're actually slightly negatively
  correlated (~0.02 fewer FTs per extra drive)..."* — cites a verifiable dataset and
  reasons to a conclusion; strip the opinion and the evidence still stands.
- Example 2: *"Wembanyama from three in the Finals: 2/9, 2/6, 2/4, 2/8, 1/6.
  Atrocious. It speaks to his resistance to contact and conditioning — easier to jack
  threes than rim run. He was exposed vs. KAT, Robinson, and OG..."* — specific
  game-by-game shooting lines anchoring a tactical argument.

### `hot_take`
An **evaluative claim asserted without verifiable evidence** — *regardless of
tone*. The claim might be true, but the post states it rather than argues it. Any
"evidence" is vague, cherry-picked, or decorative. Confidence/boldness is common
but **not required**: a calm, hedged opinion ("I think Tatum is slightly overrated
but still top 10") is still a `hot_take`, because it makes an evaluative claim
with no supporting evidence. The test is *evidence*, not *heat*.

- Example 1: *"Most overrated: Mazzulla. Aside from threes falling I didn't see many
  adjustments or good plays down the stretch — a lot of iso JT/JB ball. Yes he won a
  chip but I don't think he's THAT great. Donovan is underrated; the Bulls would be
  significantly worse without him."* — confident evaluation with only vague,
  unverifiable backup ("didn't see many adjustments").
- Example 2: *"The 2026 Nuggets couldn't even beat an insanely injured Timberwolves
  team. They got cooked by Ayo Dosunmu."* — bold dismissive claim, zero evidence; an
  enduring-quality claim, so the tie-breaker sends it here, not to `reaction`.

### `reaction`
An **immediate emotional response to a specific event** — venting a feeling in
the moment, with little to no argument. Only makes sense in the context of the
thing that just happened.

- Example 1: *"I mean what are we doing here??"* — posted on a highlight of a chaotic
  Brunson/Vassell + refs-restart sequence; pure in-the-moment exasperation, no argument.
- Example 2: *"Extending the series — 22 FTs to 4 in the second half 😂😂"* — sarcastic
  venting about ref disparity during the game. (Borderline: contains a stat, but it's
  used for emotional effect, not reasoning → `reaction` per the tie-breaker.)

## Hard edge cases

### Primary edge case: emotional venting phrased as a claim (hot_take vs reaction)
A post-game comment like *"Embiid is a choker, I'm done with this team"* sits
between `reaction` and `hot_take`. It's emotional and event-triggered (looks like
reaction) but also makes a portable evaluative claim (looks like hot_take).

**Decision rule:** Judge the *dominant function*.
- If the post is primarily **venting a feeling tied to the moment** and would
  read as nonsense a week later → `reaction`.
- If it states a **standing claim that still makes sense out of context** → `hot_take`.
- For mixed posts, ask: is the player-evaluation claim the point, or is it just
  the vehicle for the venting? "Done with this team" = venting → `reaction`.
  "Embiid is a choker, never won anything that matters" = standing claim → `hot_take`.

**Tie-breaker for short emotional posts that also contain a claim** (added after
label stress-testing surfaced repeated coin-flips on this seam): default to
`reaction` *unless* the claim is about a player's or team's **enduring quality or
legacy** ("washed," "overrated," "best ever," "choker as a player"). Legacy/ability
claims → `hot_take`; in-the-moment grievances about *this game or result* (refs,
effort, coaching, "season's over") → `reaction`. Worked cases:
- "SGA gets every call, refs handed them that game" → grievance about this game → `reaction`
- "Luka is washed at 25" → enduring-ability claim → `hot_take`
- "worst officiated game in playoff history" → grievance about this game → `reaction`

**Sarcasm / emotion about a non-game event (trade, signing, coaching hire).** The
game/legacy tie-breaker above doesn't cover these. Classify by the *real content*:
if the post's point is an evaluative claim with no evidence → `hot_take`; if it's
pure venting with no actual position → `reaction`.
- "Oh yeah, trading a 25-yr-old All-NBA guard for an aging wing is GREAT asset
  management 🙄" → sarcastic, but the point is an (unevidenced) claim that the trade
  is bad → `hot_take`.

### General rule for mixed posts (any two labels)
When a post spans two labels, assign the one matching its **primary purpose**. A
substantive, evidence-backed argument **dominates** emotional framing or a bare
opinion: if real evidence-and-reasoning is present and is the point, it's
`analysis`, even with an emotional opener.
- "I'm so done with this team 😤 — but real talk, when Murray sits the offense
  craters to 0.92 PPP, that's the actual issue" → the evidence-backed argument is
  the point → `analysis`.

### Secondary edge case: decorative stats (analysis vs hot_take)
*"LeBron is overrated — his playoff win rate vs. top seeds is below .500."* One
stat, accusatory framing.

**Decision rule:** If stripping the opinion leaves evidence that genuinely
supports the claim → `analysis`. If the stat is cherry-picked or decorative —
just enough to sound credible, not actually reasoning toward the conclusion →
`hot_take`. The example above is `hot_take` (single selected stat, framing is the point).

### Out-of-scope posts
Pure questions ("who's your GOAT?"), news-link drops, and off-topic banter are
**not takes** and will be excluded during collection, not forced into a label. I
expect these to be <10% of opinion-comments once I filter to discussion/game
threads. If they're more common, I'll narrow my source threads rather than add a
catch-all label.

## Data collection plan

- **Source:** Public r/nba comments, pulled from a mix of (a) post-game threads
  (rich in `reaction`), (b) "[Highlight]" / discussion posts (rich in `hot_take`),
  and (c) longer text/OC posts and top comments (rich in `analysis`). Sampling
  across thread types deliberately, because each thread type skews toward one label.
- **Target:** ~240 labeled comments, aiming for **roughly 80 per label** (≥20%
  each, per the project's balance guidance). `analysis` is the rarest in the wild,
  so I'll over-sample text posts and top-of-thread comments to find enough.
- **Split:** ~70% train / ~15% validation / ~15% test (≈168 / 36 / 36), stratified
  so each label appears in every split.
- **If a label is underrepresented after 200:** I'll do a targeted second pass for
  that label (e.g. if `analysis` is short, scrape more from r/nbadiscussion or
  long-form text posts) rather than relabel borderline cases to inflate the count.

## Evaluation metrics

Accuracy alone is misleading here because the classes won't be perfectly balanced
and `reaction` is the easiest to predict — a lazy model could score okay just by
favoring it. So I'll report:

- **Overall accuracy** — headline number, and the apples-to-apples comparison
  point against the Groq baseline.
- **Per-class precision, recall, and F1** — to see *which* distinction the model
  actually learned. I especially care about **`analysis` recall** (does it catch
  genuine breakdowns?) and **`hot_take` vs `reaction` confusion**, since that's the
  fuzziest boundary.
- **Macro-F1** — averages F1 across classes equally, so the rare-but-important
  `analysis` class can't be drowned out by the majority class.
- **Confusion matrix** — to read the failure structure directly (I expect most
  errors on the hot_take↔reaction edge).

## Definition of success

- **Minimum bar:** macro-F1 ≥ **0.65** on the test set, AND the fine-tuned model
  **beats the Groq zero-shot baseline** on both accuracy and macro-F1. If it
  doesn't beat zero-shot, fine-tuning didn't earn its keep.
- **"Genuinely useful" bar (deployment):** macro-F1 ≥ **0.75** with **per-class
  recall ≥ 0.60 on every class** — no class is being ignored — and `analysis`
  **precision ≥ 0.70**, because a tool that surfaces "high-quality takes" is only
  trustworthy if what it flags as analysis usually is.
- I will *not* trust >0.95 accuracy on this subjective task — that would signal a
  train/test leak or labels that are too easy, and I'll re-check my split.

## AI Tool Plan

This project has little code to generate, so AI tooling is used at three points:

- **Label stress-testing (will do, before annotating):** I'll give an LLM my three
  definitions + edge-case rules and ask it to generate 8–10 r/nba posts that sit
  on the hot_take↔reaction and analysis↔hot_take boundaries. If I can't cleanly
  classify its outputs with my own rules, the definitions need tightening — I'll
  revise before labeling 200.
- **Annotation assistance (will do, with disclosure):** I'll use an LLM to
  *pre-label* batches, then review every label myself and correct it; the model's
  guess is a starting point, never the final label. I'll add a `prelabeled` column
  to the working CSV to track which rows were AI-suggested, and disclose this in
  the README's AI-usage section. The gold labels are my own decisions.
- **Failure analysis (will do, with verification):** After evaluation I'll hand the
  list of wrong predictions to an LLM and ask it to find systematic patterns
  (e.g. "misses sarcasm," "confuses short hot_takes with reactions"). I'll then
  verify each proposed pattern by hand against the actual examples before putting
  it in the report — the LLM proposes, I confirm.

_(Update this section before starting any stretch features.)_
