# Search Intelligence: Prioritizing Content Refresh Cycles with Machine Learning

**Author:** Mohamed Fathy  
**Track:** Machine Learning — Foundations & Practice  
**Data Credit:** Built on the [FlyRank ML Internship Dataset](https://flyrank.ai)  
**Repository:** [GitHub Repository](https://github.com/mohamedalangr/flyrank-ml-internship)

---

## Abstract
How can content marketing teams systematically prioritize editorial refresh cycles before organic traffic decay becomes irreversible? We present a machine learning scoring engine designed to identify and rank pages exhibiting high traffic decay risk. Utilizing a Random Forest classifier evaluated under client-grouped cross-validation (`GroupKFold`), we model 90-day historical performance signals across anonymized content items. Our model achieves a cross-validated ROC AUC of **0.7621**, significantly outperforming traditional static heuristic rules (ROC AUC **0.5410**). By transforming continuous probability scores into an actionable four-tier decision playbook, this workflow streamlines editorial bandwidth toward high-impact content updates.

---

## 1. Introduction & Problem Statement
Digital content decays naturally as search engine algorithms evolve, competitors publish newer material, and user search intent shifts. Traditional content refresh strategies rely heavily on static heuristics—such as flagging all pages older than 365 days or manually reviewing high-traffic URLs when traffic collapses. 

These static approaches suffer from two major flaws:
1. **High False Positive Rates:** Age alone does not dictate decay; evergreen articles can remain top performers for years without intervention.
2. **Reactive Overhead:** Manual editorial review across thousands of published URLs wastes hundreds of human hours evaluating stable content.

This project addresses a specific operational decision: **How to accurately rank existing content by predicted decay risk so editorial teams spend time only on high-yield refresh opportunities.**

---

## 2. Data & Exclusions
We analyze an anonymized slice of content performance logs comprising ~30,000 unique records across 32 clients. 

*   **Grain (Unit of Analysis):** One row represents the aggregated 90-day historical performance metrics of a single unique content item (`content_id`) for a specific client (`client_id`).
*   **Temporal Window:** 90-day aggregated historical window prior to the decision prediction point.
*   **Feature Set:** `impressions_90d`, `sessions_90d`, `content_age_days`, `avg_position`, and `word_count`.
*   **Target Label Proxy:** Binary flag `target_decline` ($1$ if `trend_direction == 'down'`, $0$ otherwise).
*   **Exclusions:** Records with missing client identifiers or unverified trend directions were dropped to ensure strict validation integrity and group-wise isolation.

---

## 3. Methodology & Validation Design

### Baseline Heuristic
Our baseline rule combined normalized staleness (`content_age_days`) and CTR deficit relative to rank position into a linear risk index:
$$\text{Baseline Score} = (\text{Staleness Risk} \times 0.5) + (\text{CTR Defect Score} \times 0.5)$$

### Machine Learning Model
We trained a **Random Forest Classifier** (`n_estimators=100`, `max_depth=5`) on historical feature representations to learn non-linear risk interactions.

### Honest Validation Design
To prevent intra-client pattern leakage, we strictly evaluated model performance using **5-fold `GroupKFold` cross-validation grouped by `client_id`**. This guarantees that no page from a test client is ever visible during training, proving the model's ability to generalize to entirely unseen domains.

### Leakage Checks
All target-derived features, post-decision outcome metrics, and artificial product flags were excluded from training matrices.

---

## 4. Results (Model vs. Baseline)

Evaluating both systems under identical 5-fold client-grouped splits demonstrates a massive performance lift when transitioning from static heuristics to supervised learning:

| Strategy | Validation Scheme | Mean ROC AUC | Key Takeaway |
|---|---|---|---|
| **Static Heuristic Rule** | Linear Combination | **0.5410** | Barely better than random guessing |
| **Random Forest ML Engine** | **5-Fold `GroupKFold`** | **0.7621** | Strong, generalization across unseen clients |

---

## 5. Limitations & Honest Framing
*   **Observational Scope:** Model predictions reflect statistical associations within historical logs, not causal guarantees that an editorial rewrite will restore lost search engine rankings.
*   **Static Windowing:** Using 90-day aggregated windows creates a detection lag during sudden, intra-week core algorithm updates.
*   **Missing Technical Context:** On-page technical bugs (e.g., broken JavaScript rendering or indexation tags) and backlink profile changes are unmodeled in the current schema.

---

## 6. Ranked Content Action Playbook

Our output maps continuous risk probabilities into four operational action tiers with explicit diagnostic reason codes:

| Priority Tier | Action Label | Primary Reason Code | Trigger Conditions |
|---|---|---|---|
| **Priority 1** | `URGENT_REFRESH` | `HIGH_DECAY_HIGH_TRAFFIC` | Decay Risk $> 0.65$ AND 90d Sessions $> 300$ |
| **Priority 2** | `OPT_TITLE_META` | `LOW_CTR_GOOD_POSITION` | Rank Position $\le 15.0$ AND CTR $< 1.0\%$ |
| **Priority 3** | `CONTENT_PRUNE` | `LOW_IMPRESSIONS_OLD_AGE` | Age $> 365$ days AND 90d Impressions $< 50$ |
| **Priority 4** | `MONITOR_ONLY` | `STABLE_PERFORMANCE` | Decay Risk $\le 0.40$ or Stable Trend |

### Automation No-Go List (Human-in-the-Loop)
AI-generated content rewrites and URL deletions must **never** be auto-published or programmatically executed without human editorial review. Editors must manually verify search intent shifts and technical indexability before executing content rewrites.

---

## 7. Reproducibility & Artifacts
All code, data preparation scripts, baseline implementations, and validation notebooks are committed and open-source in the project repository:
*   `work/notebooks/w02_ml_task_framing.ipynb`
*   `work/notebooks/w03_data_contract.ipynb`
*   `work/notebooks/w04_baseline_score.ipynb`
*   `work/notebooks/w06_validation_audit.ipynb`
*   `work/notebooks/w07_action_playbook.ipynb`
*   `work/notebooks/capstone.ipynb`

---

## Acknowledgments & Data Credit
This research paper was built on the [FlyRank ML Internship Dataset](https://flyrank.ai). We thank FlyRank for providing real-world, anonymized search intelligence data to support open data science education and research.
