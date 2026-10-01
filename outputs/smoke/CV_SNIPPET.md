> **SMOKE TEST OUTPUT - these numbers come from a tiny debug run and are NOT real results.**

- Evaluated 2 MT systems (NLLB-200, Marian opus-mt) on FLORES-200 EN-DE with chrF, BLEU and COMET; best system reached chrF 60.8 and COMET 0.482
- Built a parallel-corpus filter (deduplication, length ratio, language ID, number mismatch) that removed 5.4% of 500 sentence pairs, with [NOT RUN] filter precision in a manual check of [NOT RUN] pairs
- Analyzed [NOT RUN] translation errors by category across the best and worst system
