# TakeMeter — Planning

## 1. Community

> Which online community are you classifying discourse from? (e.g. r/nba, a music
> theory Discord, an anime forum.) Why this one? What do "takes" look like there?

**Community:**

**Why this community:**

**Where I'll collect data from (be specific — subreddit, thread, export tool):**

---

## 2. Label taxonomy (2–4 labels)

> Labels must be: mutually exclusive (each post gets exactly ONE), exhaustive
> enough (90%+ of posts fit without an "other" bucket), and grounded in real
> community norms.

| Label | Definition (1–2 sentences) | Example post |
|-------|----------------------------|--------------|
| `label_1` | | |
| `label_2` | | |
| `label_3` | | |

**Why these distinctions matter to people in this community:**

---

## 3. Edge case rules

> Decide these BEFORE labeling 200 posts. What do you do when a post is sarcastic?
> Off-topic? Both insightful AND hyperbolic? Write the tie-breaker rules here.

- If a post is X, label it as ...
- If a post is both A and B, ...
- Off-topic / non-take posts: ...

---

## 4. Plan of attack

- [ ] Read 30–40 real posts to sanity-check that labels apply cleanly
- [ ] Revise labels if needed
- [ ] Collect ~200 posts into `data/dataset.csv`
- [ ] Label all 200
- [ ] Split into train / validation / test
- [ ] Fine-tune DistilBERT in Colab
- [ ] Run Groq zero-shot baseline
- [ ] Write evaluation report in README

## 5. Stretch features (optional — update before starting each)

- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [ ] Deployed interface
