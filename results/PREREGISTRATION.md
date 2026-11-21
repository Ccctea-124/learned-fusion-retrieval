# Evaluation protocol and amendment log — 2026-10-30

The 2026-10-29 claim-verification protocol was superseded on 2026-10-30 when the scope was narrowed. Its original text remains in Git history. The two-method retrieval protocol was amended again after exploratory test results showed complementary signals. This file records the chronology; the fusion comparison is **not preregistered**.

## Question

On BEIR SciFact, does a four-feature learned fusion of BM25 and frozen MiniLM improve ranking quality over each alone? RRF is an untrained reference.

## Data and metrics

- Train the fusion head only on train qrels; use train-only cross-validation for a stability check. Compare BM25, MiniLM, RRF (k=60), and logistic fusion on test qrels.
- Primary metric: nDCG@10. Secondary: Recall@100, per-query latency, and top-100 union candidate recall.
- Use query-paired bootstrap intervals with a recorded seed and 5,000 draws. Show per-query success and failure cases.
- The BEIR test split was used in an earlier TF-IDF smoke test **and** in method exploration that selected fusion. It is **not** a blind holdout. The intervals are exploratory and do not support a confirmatory significance claim.
- Measure how often gold test documents also appear in train qrels; report a sensitivity analysis for test queries without such overlap.

## Interpretation

We will report all methods under the same metrics and discuss complementary signals without presenting test-selected gains as independent confirmation. Any later change to methods or metrics must be dated here.

## Amendment log

| Date | Change | Reason |
| --- | --- | --- |
| 2026-10-29 | Original claim-verification protocol | Initial project concept |
| 2026-10-30 | Replaced with a two-method retrieval study | Scope narrowed |
| 2026-10-30 | Added four-feature learned fusion, RRF, Recall@100, paired intervals | Exploratory test comparison made the two-method contrast weak; this amendment followed test inspection |
