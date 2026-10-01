# MT Quality Lab (EN-DE)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Gowrikamahesh17/mt-quality-lab/blob/main/mt_quality_lab.ipynb)

## Purpose
Compare two open MT systems (NLLB-200-distilled-600M, Marian opus-mt-en-de) on FLORES-200 EN-DE with chrF, BLEU and COMET, build and manually audit a rule-based filter for a noisy parallel corpus, and categorise translation errors by hand.

## Results: FLORES-200 devtest, EN-DE
n = 1012 sentences. COMET model: `Unbabel/eamt22-cometinho-da` (scores are only comparable within this same model; they run lower than the full `wmt22-comet-da` model).

| system | chrF | BLEU | COMET |
|---|---|---|---|
| NLLB-200-600M | 62.0 | 34.7 | 0.551 |
| opus-mt-en-de | 63.7 | 36.2 | 0.553 |

Best system: **opus-mt-en-de** (chrF 63.7, BLEU 36.2, COMET 0.553 on `Unbabel/eamt22-cometinho-da`).

sacreBLEU signatures: `nrefs:1|case:mixed|eff:yes|nc:6|nw:0|space:no|version:2.6.0`, `nrefs:1|case:mixed|eff:no|tok:13a|smooth:exp|version:2.6.0`.

## Corpus filter
Removed **1669 of 20000** pairs (8.3%); kept 18331. Source: `sentence-transformers/parallel-sentences-wikimatrix` (en-de), datasets shuffle(seed=42).select(range(20000)), from 344476 pairs.

| # | rule | threshold | pairs failing rule | removed (first failing rule) |
|---|---|---|---|---|
| 1 | empty | empty / whitespace-only source or target | 0 | 0 |
| 2 | duplicate | exact duplicate after NFKC + lowercase + whitespace normalisation (first copy kept) | 0 | 0 |
| 3 | length | chars < 5 or words > 100 or length ratio > 2.0 | 21 | 21 |
| 4 | lang_id | fastText lid.176: source not en or target not de, or confidence < 0.5 | 987 | 981 |
| 5 | number_mismatch | digit sets of source and target differ | 729 | 667 |

Pairs can fail several rules; the last column attributes each removed pair to the first rule it fails (rules applied in the order above).

## Filter audit (manual, 100 removed + 100 kept pairs)
- Removal precision (removed pairs that were truly bad): 44/100 = 44.0% (Wilson 95% CI 34.7-53.8%)
- Residual noise in kept set (kept pairs that were bad): 4/100 = 4.0% (Wilson 95% CI 1.6-9.8%)
- n=100 per group gives wide uncertainty; treat these as rough estimates.

**Root cause.** Breaking the audit sample down by rule shows `number_mismatch` is responsible for most of the bad removals:

| rule (first failing rule) | sampled removals | false-removal rate |
|---|---|---|
| number_mismatch | 73 | 82.2% |
| lang_id | 26 | 38.5% |
| length | 1 | 100.0% (n=1, not meaningful) |

The rule compares raw digit characters only, so it flags pairs where English spells a number as a word and German writes it as a digit (e.g. "Twenty-three" / "23", "sixties" / "1960er") as a mismatch even though the translation is correct. A word-number-normalising version of the rule was tried (mapping common English/German number words and ordinals to digits before comparing), but it made precision worse, not better: it missed inflected German ordinal forms ("viertes", "vierten", ...) and mishandled "million"/"thousand" as literal digit values instead of multipliers (so "2 million" vs "zwei Millionen" still mismatched), which raised removals from 1,669 to 3,257 and dropped removal precision from 44.0% to 29.0% on re-audit. That fix was reverted; the simpler rule above (and the 44.0%/4.0% audit numbers) is what's reported. A correct fix would need real German morphological handling and multiplier-aware number parsing - noted as a limitation below rather than shipped half-working.

## Error analysis (manual, 30 sentences from each of best and worst system)
| category | NLLB-200-600M | opus-mt-en-de | total |
|---|---|---|---|
| terminology | 5 | 4 | 9 |
| named entity | 0 | 0 | 0 |
| number/date | 1 | 0 | 1 |
| fluency | 4 | 5 | 9 |
| omission/addition | 0 | 0 | 0 |
| mistranslation | 1 | 2 | 3 |
| none | 19 | 19 | 38 |

Best system by COMET: opus-mt-en-de; worst: NLLB-200-600M. Labeled errors: 60. Most frequent error categories (excluding "none"): **terminology and fluency are tied at 9 each** (out of 60 labeled sentences); mistranslation is next at 3. No single category dominates at this sample size.

## How to run
**Colab:** click the badge, set runtime to T4 GPU, set `SMOKE_TEST = False`, run all. Outputs go to `MyDrive/mt_quality_lab/full/`. Run once to produce `audit_sheet.csv` and `error_sheet.csv`, label them (save as `audit_sheet_labeled.csv` / `error_sheet_labeled.csv` in the same folder), then rerun; finished steps are cached.

**Local smoke test:** `pip install -r requirements.txt`, keep `SMOKE_TEST = True`, run the notebook top to bottom (outputs in `outputs/smoke/`).

## Limitations
- FLORES-200 may overlap with the training data of the evaluated models (contamination), so scores may be optimistic.
- The audit has n=100 per group, which gives wide confidence intervals.
- One corpus and domain only (WikiMatrix, Wikipedia-derived); results may not transfer.
- Cometinho (a small distilled model) is used instead of the full COMET model (e.g. `wmt22-comet-da`); scores are not comparable to full COMET numbers.
- Labels come from a single annotator.
- The number_mismatch rule compares digit characters only and has an 82.2% false-removal rate in the audit sample (see "Root cause" above): it misses correct translations where a number is spelled as a word on one side and written as a digit on the other. A fix was attempted and reverted because it made things worse; this is a known, unresolved weakness of the filter.

## Data notes
FLORES-200 is loaded from Meta's official tarball (the Hugging Face copies are gated or script-based). Model IDs, library versions and sacreBLEU signatures are recorded in `results.json`.

## What I learned

- The two systems scored close on chrF/BLEU/COMET (opus-mt-en-de slightly ahead on all three), but COMET absolute values were modest (about 0.55) for both: a reminder that chrF/BLEU and COMET can agree on ranking while still signalling real translation issues in absolute terms.
- Rule-based filtering removed 8.3% of the noisy corpus automatically, but the blind audit showed only 44% removal precision. Breaking that down by rule found the cause: `number_mismatch` alone had an 82.2% false-removal rate, because it compared raw digits only and flagged correct translations where English spells a number as a word and German writes a digit (or vice versa). I tried fixing it by normalising word-numbers to digits before comparing, but the fix introduced its own bugs (missed German ordinal inflections, treated "million"/"thousand" as literal digits instead of multipliers) and made precision worse (29% instead of 44%) on re-audit, so I reverted it. That was a useful lesson: diagnosing the failure mode is not the same as having a correct fix, and a plausible-looking patch needs to be re-measured, not just shipped because it addresses the right rule.
- The n=100 audit groups gave wide Wilson confidence intervals (roughly +/-9-10 points), which is a real limitation when trying to trust a specific precision number: directionally useful, not precise.
- In the 60-sentence error analysis, both systems were mostly "none" (no major error) for sampled sentences; terminology and fluency tied as the next most common issues (9 each), with mistranslation less common (3) and number/date and named-entity errors rare at this sample size.
