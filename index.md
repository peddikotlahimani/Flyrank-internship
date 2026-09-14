# Do Freshness, Length, and Format Actually Predict Click-Through Rate?

## Abstract

I wanted to find out if things like how fresh a page is, how long it is, and what type it is actually affect how many people click on it. I used 30,000 pages of real (but anonymous) website data. I tested some ideas, built a simple rule, then trained a model and compared them. Turns out freshness and content type mattered a lot, but longer content didn't help — shorter pages actually did better. This could help someone running a website figure out what to fix first.


## Introduction

If you run a website with lots of pages, you can't fix all of them at once. Should you make pages longer? Update old ones? I wanted to actually check what helps, instead of just guessing.


## Data

I used a file with 30,000 web pages. It had stuff like how many people saw each page, how many clicked, how long the page was, and when it was last updated. I skipped a few columns on purpose because using them would've been like peeking at the answer early.

**Excluded from this analysis:**

I used a file with 30,000 web pages. It had stuff like how many people saw each page, how many clicked, how long the page was, and when it was last updated. I didn't use two columns called trend_direction and trend_pct, because those were meant to be used as the "answer" for a different question, so using them here would've been cheating. I also skipped two columns about which AI tool wrote the content, since I was told those aren't supposed to be used as signals. No real names or private info are anywhere in this data -- everything is already anonymous.

## Methodology
**Assumptions:**I assumed the 90-day numbers in this data are pretty normal, not from some one-time event. I also assumed word count, impressions, and freshness are things a real team could actually check and fix, which is why I picked those three.

**Features tested:** `word_count`, `impressions_90d`, and `days_since_last_update` — all safe, non-leaking signals a content team could realistically act on.

**Label:** A page was defined as having "good CTR" if its CTR was at or above the dataset average (0.51%). 

**Baseline:** A transparent, hand-written rule scoring pages on content length, visibility, and freshness. An initial version of this baseline accidentally used `ctr` directly to build its score, producing an artificially perfect result; this was identified and corrected to use only the same three features available to the model, for a fair comparison.

**Validation design:** Data was split by `client_id` (not randomly) into training and test, ensuring no client's pages appeared in both sets — verified with a zero-overlap check. This guards against the model simply memorizing a client's typical behavior rather than learning a generalizable pattern.

**Leakage checks:** Excluded columns were explicitly tested against the feature list used in modeling, confirming no label-source or disallowed columns were used.

## Results

| Method | Precision@50 |
|---|---|
| Random guessing (base rate) | 0.13 |
| Fair rule-based baseline | 0.18 |
| Trained model (Logistic Regression) | 0.14 |

My simple hand-made rule did a little better than my trained model, and both barely beat just guessing randomly. Freshness and content type really did matter. But longer content didn't help at all -- shorter pages actually got more clicks, which I didn't expect.

**Signal test results:**
- **Freshness → CTR:** recently updated pages averaged 0.73 CTR vs. 0.26 for older pages (≈3× higher) — **confirmed**
- **Content type → CTR:** feedly articles averaged 2.79 CTR vs. 0.34 (keyword) and 0.13 (comparison) — **confirmed**, large effect
- **Word count → CTR:** short pages averaged 0.88 CTR vs. 0.32 for long pages — **opposite** of the common assumption
- **Ranking position → CTR:** top-3 positioned pages averaged 1.48 CTR vs. 0.15 for deep positions (≈10× higher) — **confirmed**, the strongest single association observed

## Limitations & Honest Framing

I only used a small chunk of data, not everything, and just one 90-day period, so I don't know if this would look the same at a different time.

I can't say for sure that updating a page causes more clicks -- I just observed that they tend to happen together. So this is more of a directional hint, not a proven fact.

My model wasn't very confident either, so I can't fully trust its guesses.

Because of all this, everything in this paper should be treated as decision-support, meaning it's a helpful starting point to guide what to check next, not a guaranteed answer.

## Ranked Recommendations

1. **Prioritize ranking position first.** This showed the strongest association with CTR (≈10× difference, top-3 vs. deep positions) of any signal tested.
2. **Keep content updated on a regular cadence.** Freshness showed a consistent, confirmed ≈3× association with CTR across two independent checks.
3. **Investigate content-type differences further.** The feedly-format advantage is large but may reflect topic or audience differences rather than format alone — worth a targeted follow-up study before acting on it broadly.
4. **Don't assume longer content improves CTR.** This dataset showed the opposite; length alone is not a safe proxy for click-worthiness.
5. **Treat impressions and CTR as separate problems.** High visibility does not predict high engagement — a page can be seen often without being compelling enough to click.

## Reproducibility

All analysis is available in the linked repository under `work/notebooks/`: `w03_data_contract.ipynb`, `w03_feature_leakage_check.ipynb`, `w04_baseline_score.ipynb`, `w04_signal_audit.ipynb`, `w05_model.ipynb`, and `capstone.ipynb`. All my work is saved in my GitHub repo, in notebooks anyone can open and rerun.

**Repository:** [github.com/peddikotlahimani/Flyrank-internship](https://github.com/peddikotlahimani/Flyrank-internship)

## Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset — [flyrank.ai](https://flyrank.ai)
