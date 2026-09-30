# MT Quality Lab (EN→DE)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Gowrikamahesh17/mt-quality-lab/blob/main/mt_quality_lab.ipynb)

## Purpose
Compare two open MT systems (NLLB-200-distilled-600M, Marian opus-mt-en-de) on FLORES-200 EN→DE with chrF, BLEU and COMET, build and manually audit a rule-based filter for a noisy parallel corpus, and categorise translation errors by hand.

## Results: FLORES-200 devtest, EN→DE
n = NOT RUN sentences. COMET = `Unbabel/eamt22-cometinho-da`.

| system | chrF | BLEU | COMET |
|---|---|---|---|
| NOT RUN | NOT RUN | NOT RUN | NOT RUN |

sacreBLEU signatures: NOT RUN.

## Corpus filter
Filter: NOT RUN.

| # | rule | threshold | pairs failing rule | removed (first failing rule) |
|---|---|---|---|---|
| 1 | empty | empty / whitespace-only source or target | NOT RUN | NOT RUN |
| 2 | duplicate | exact duplicate after NFKC + lowercase + whitespace normalisation (first copy kept) | NOT RUN | NOT RUN |
| 3 | length | NOT RUN | NOT RUN | NOT RUN |
| 4 | lang_id | NOT RUN | NOT RUN | NOT RUN |
| 5 | number_mismatch | digit sets of source and target differ | NOT RUN | NOT RUN |

Pairs can fail several rules; the last column attributes each removed pair to the first rule it fails (rules applied in the order above).

## Filter audit (manual, 100 removed + 100 kept pairs)
- Removal precision (removed pairs that were truly bad): NOT RUN
- Residual noise in kept set (kept pairs that were bad): NOT RUN
- n=100 per group gives wide uncertainty; treat these as rough estimates.

## Error analysis (manual, 30 sentences from each of best and worst system)
NOT RUN

## How to run
**Colab:** click the badge, set runtime to T4 GPU, set `SMOKE_TEST = False`, run all. Outputs go to `MyDrive/mt_quality_lab/full/`. Run once to produce `audit_sheet.csv` and `error_sheet.csv`, label them (save as `audit_sheet_labeled.csv` / `error_sheet_labeled.csv` in the same folder), then rerun; finished steps are cached.

**Local smoke test:** `pip install -r requirements.txt`, keep `SMOKE_TEST = True`, run the notebook top to bottom (outputs in `outputs/smoke/`).

## Limitations
- FLORES-200 may overlap with the training data of the evaluated models (contamination), so scores may be optimistic.
- The audit has n=100 per group, which gives wide confidence intervals.
- One corpus and domain only (WikiMatrix, Wikipedia-derived); results may not transfer.
- Cometinho (a small distilled model) is used instead of the full COMET model; scores are not comparable to full COMET numbers.
- Labels come from a single annotator.

## Data notes
FLORES-200 is loaded from Meta's official tarball (the Hugging Face copies are gated or script-based). Model IDs, library versions and sacreBLEU signatures are recorded in `results.json`.

## What I learned
TODO: write this section.
