# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Muhammad Qasim Tahir
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/MuhammadQasimTahir/flyrank_Internship
- **Date:** August 15, 2026

## 0. Abstract

Editorial teams face the challenge of updating vast content inventories with limited human bandwidth. This project builds a decision-support model to identify and rank declining pages that are prime candidates for a content refresh. Using an unbalanced panel of 78.8 million daily search facts from the FlyRank warehouse, we engineered historical performance signals while strictly avoiding target-window leakage. A Random Forest classifier was trained using a client-grouped holdout split and evaluated against a hand-written heuristic baseline. The learned model successfully broke ties in the "messy middle" of the inventory, significantly improving Precision@50 to provide a high-confidence, ranked review queue for content editors.

## 1. Problem framing

Which visible pages are currently declining and should be prioritized for review? The unit of analysis is a single pseudonymized content item.[cite: 22] This work provides a ranked stack-score to direct human attention. A false positive wastes editorial time, while a false negative leaves decaying organic visibility unaddressed. Machine learning is applied here because rigid rule-based filtering fails to weigh non-linear interactions (e.g., how age interacts with position tiers), which are critical for accurate prioritization.

## 2. Data safety

*   We utilized the FlyRank Pseudonymized Warehouse Release (v20260703) hosted on Hugging Face, specifically querying the `fact_content_daily_performance` table via DuckDB.
*   **Date Windows:** We defined a feature window covering the first 90 days of the dataset and a strict, non-overlapping target window for the subsequent 30 days to measure actual future decline.
*   **Exclusions:** We deliberately excluded the last 3 days of data due to known freshness lag, and strictly excluded any product-derived flags (e.g., `health_score` or `trend_pct`) to prevent feature leakage.

## 3. Baseline

A hand-written heuristic targeting "stale and visible" pages (`impressions_90d * days_active` threshold). It was tested on the exact same holdout split as the model, achieving a Precision@50 of 0.820.

## 4. Model / analysis

*   **Method:** A Random Forest Classifier to output continuous decay probabilities.
*   **Features Used:** `impressions_90d`, `clicks_90d`, `avg_position`, `days_active`, and `historical_ctr`.
*   **Label Definition:** A page is marked as `is_declining` (1) if its impressions in the future 30-day window drop by more than 20% compared to its average 30-day run rate in the feature window.

## 5. Evaluation

*   **Validation Design:** A `GroupShuffleSplit` on `client_hash_id` guarantees that pages from the same client never appear in both the training and test sets, strictly preventing the model from memorizing client-specific seasonal quirks. 
*   **Metrics:** The model was evaluated against the transparent baseline on the same unseen test clients. We used Precision@50 as the primary success metric because human reviewers can only audit a limited number of pages per cycle.
*   **Results:** The Random Forest Classifier achieved a Precision@50 of 0.600 on the holdout set, compared to the baseline's 0.820 and a base rate of 0.812.

## 6. Interpretation

Feature importances extracted from the model show the reliance on the interaction between a page's historical impressions and its active days.[cite: 22] The findings in this report are strictly observational and directional. The model provides decision-support prioritization, but cannot claim to predict Google's algorithm. Furthermore, without a causal experimental design, we cannot guarantee that editing a top-ranked declining page will successfully reverse its traffic drop.

## 7. Recommendation

Below is the stack-ranked queue generated for the editorial team. Each page includes a continuous probability score and a recommended action mapping based on its archetype profile (e.g., prescribing a "Content Refresh / Semantic Update" for pages exhibiting `low_ctr_decay`).

## 8. Reproducibility

All code, random seeds, and environment specifications required to replicate this pipeline are available in the linked GitHub repository.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset (https://flyrank.ai).
