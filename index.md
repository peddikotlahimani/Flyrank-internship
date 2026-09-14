# Do Freshness, Length, and Format Actually Predict Click-Through Rate?

## Abstract

Content teams often assume that longer, more detailed articles perform better in search — but is that actually true? This project tested whether page freshness, word count, and content type are associated with click-through rate (CTR) using 30,000 anonymized pages from FlyRank's internship dataset. Using simple signal tests and a trained logistic regression model, freshness and content type showed strong, consistent links to CTR, while word count did not — shorter pages actually outperformed longer ones. A trained model using these signals barely beat random guessing, showing these three signals alone aren't enough to reliably predict CTR on their own. The findings point content teams toward prioritizing freshness and ranking position over content length when deciding what to fix first.

## Introduction

Imagine a content team with limited time and a long list of pages to review. Should they rewrite short pages to make them longer? Focus on updating old content? Or look at ranking position first? Without data, this is a guess. This project set out to answer a narrower, testable version of that question: **which safe, available signals are actually associated with a page getting a good click-through rate?** The goal isn't to build a perfect predictive model — it's to give a content team evidence-backed priorities instead of assumptions.

## Data

This analysis used the starter dataset from FlyRank's ML internship program: `content_refresh_anonymized.csv`, containing 30,000 rows (one per content page) across 32 pseudonymized clients. All metrics are aggregated over a trailing 90-day window. This is a teaching-sized slice, not the full ~79-million-row warehouse release — a scope limitation noted below.

**Excluded from this analysis:**
- `trend_direction` and `trend_pct` — reserved as label-source columns for a separate decline-prediction pipeline; using them here risked leakage
- `provider_used` and `model_used` — marked as non-feature columns in the data dictionary
- No client names, URLs, or private identifiers appear anywhere; all IDs are pre-existing pseudonyms

## Methodology

**Label:** A page was defined as having "good CTR" if its CTR was at or above the dataset average (0.51%).

**Features tested:** `word_count`, `impressions_90d`, and `days_since_last_update` — all safe, non-leaking signals a content team could realistically act on.

**Baseline:** A transparent, hand-written rule scoring pages on content length, visibility, and freshness. An initial version of this baseline accidentally used `ctr` directly to build its score, producing an artificially perfect result; this was identified and corrected to use only the same three features available to the model, for a fair comparison.

**Validation design:** Data was split by `client_id` (not randomly) into 80% training and 20% test, ensuring no client's pages appeared in both sets — verified with a zero-overlap check. This guards against the model simply memorizing a client's typical behavior rather than learning a generalizable pattern.

**Leakage checks:** Excluded columns were explicitly tested against the feature list used in modeling, confirming no label-source or disallowed columns were used.

## Results

| Method | Precision@50 |
|---|---|
| Random guessing (base rate) | 0.13 |
| Fair rule-based baseline | 0.18 |
| Trained model (Logistic Regression) | 0.14 |

The rule-based baseline modestly outperformed the trained model, and both only slightly beat random guessing. Examining the model's behavior directly explains why: of 6,163 test pages, the model predicted "high CTR" for only 5 — it defaulted to "low CTR" almost universally, reflecting weak confidence across all three features (all feature weights were extremely small in magnitude).

**Signal test results:**
- **Freshness → CTR:** recently updated pages averaged 0.73 CTR vs. 0.26 for older pages (≈3× higher) — **confirmed**
- **Content type → CTR:** feedly articles averaged 2.79 CTR vs. 0.34 (keyword) and 0.13 (comparison) — **confirmed**, large effect
- **Word count → CTR:** short pages averaged 0.88 CTR vs. 0.32 for long pages — **opposite** of the common assumption
- **Ranking position → CTR:** top-3 positioned pages averaged 1.48 CTR vs. 0.15 for deep positions (≈10× higher) — **confirmed**, the strongest single association observed

## Limitations & Honest Framing

These results are based on a 30,000-row teaching slice, not the full warehouse release, and reflect a single 90-day window — seasonal or scale effects are untested. All relationships reported are **observed associations, not proven causes**: freshness and CTR moving together does not establish that updating a page directly causes higher CTR (better-performing pages may simply get updated more often for unrelated reasons). The trained model's weak performance indicates that word count, impressions, and freshness alone are insufficient to reliably predict CTR — other unmeasured factors likely matter more. These findings should be treated as **directional, decision-support signals**, not guarantees.

## Ranked Recommendations

1. **Prioritize ranking position first.** This showed the strongest association with CTR (≈10× difference, top-3 vs. deep positions) of any signal tested.
2. **Keep content updated on a regular cadence.** Freshness showed a consistent, confirmed ≈3× association with CTR across two independent checks.
3. **Investigate content-type differences further.** The feedly-format advantage is large but may reflect topic or audience differences rather than format alone — worth a targeted follow-up study before acting on it broadly.
4. **Don't assume longer content improves CTR.** This dataset showed the opposite; length alone is not a safe proxy for click-worthiness.
5. **Treat impressions and CTR as separate problems.** High visibility does not predict high engagement — a page can be seen often without being compelling enough to click.

## Reproducibility

All analysis is available in the linked repository under `work/notebooks/`: `w03_data_contract.ipynb`, `w03_feature_leakage_check.ipynb`, `w04_baseline_score.ipynb`, `w04_signal_audit.ipynb`, `w05_model.ipynb`, and `capstone.ipynb`. All random operations use a fixed seed (`random_state=42`) for reproducibility. Data can be reloaded directly from the repository's raw CSV path referenced in each notebook.

**Repository:** [github.com/peddikotlahimani/Flyrank-internship](https://github.com/peddikotlahimani/Flyrank-internship)

## Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset — [flyrank.ai](https://flyrank.ai)
