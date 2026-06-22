# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in [your community].

## Labels

| Label | Definition |
|-------|------------|
| `label_1` | |
| `label_2` | |
| `label_3` | |

See [planning.md](planning.md) for full definitions, examples, and edge-case rules.

## Dataset

- **Where it came from:**
- **How I labeled it:**
- **Label distribution (count per label):**

  | Label | Count |
  |-------|-------|
  | label_1 | |
  | label_2 | |
  | label_3 | |

- **3 examples I found genuinely hard to label, and what I decided:**
  1.
  2.
  3.

- **Train / validation / test split:** (e.g. 140 / 30 / 30)

## Model & training

- **Base model:** distilbert-base-uncased
- **Training approach:**
- **Key hyperparameter decision (and why):** (e.g. 3 epochs, lr 2e-5, batch size 16)

## Baseline

- **Zero-shot model:** Groq `llama-3.3-70b-versatile`
- **Prompt used:** (paste it here)

## Evaluation report

| Metric | Fine-tuned DistilBERT | Groq zero-shot |
|--------|----------------------|----------------|
| Accuracy | | |
| F1 (per class) | | |

**Confusion matrix:** see [outputs/confusion_matrix.png](outputs/confusion_matrix.png)

### 3 examples the model got wrong + why

1.
2.
3.

### What the model learned vs. what I intended

> Reflection goes here.

## How to run

1. Open the Colab notebook: [link]
2. Set runtime to T4 GPU.
3. Upload `data/dataset.csv`.
4. Run all cells.
5. Download `evaluation_results.json` and `confusion_matrix.png` into `outputs/`.
