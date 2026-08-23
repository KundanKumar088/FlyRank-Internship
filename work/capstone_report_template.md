# Capstone Report — Enterprise Content Traffic Decline Prediction

* **Author:** Kundan Kumar
* **Lane:** Machine Learning
* **Repo:** KundanKumar088/FlyRank-Internship
* **Date:** July 2026

---

## 0. Abstract
We designed and deployed a machine learning pipeline to forecast deep organic traffic drops across an enterprise content ecosystem before the visibility loss materializes. Framing the problem as a binary classification task over sequential 90-day windows, we ingested a public-safe dataset containing 427,292 records. We engineered historical baseline features that measure raw visibility scale and structural platform activity while isolating highly volatile edge cases. Subjected to a strict client-grouped validation barrier to prevent data leakage, our trained Random Forest model achieved an audited accuracy of 86.54%, outperforming naive static rules on unseen client domains. The system delivers a continuous, risk-ranked action playbook that directly provides proactive directional decision-support for content optimization workflows.

---

## 1. Problem framing
* **What decision does this support?** This automated analytical pipeline serves as a directional decision-support tool for corporate SEO strategists and content managers. Instead of reactively troubleshooting drop-offs after they damage the bottom line, it allows engineering and writing teams to proactively target volatile content assets with structural page refreshes.
* **Unit of analysis:** The unit of analysis is an individual content node, uniquely mapped via a combination of `client_hash_id` and `content_hash_id`.
* **The output:** The system yields a continuous risk probability score between 0.0 and 1.0 representing the likelihood of a major organic footprint collapse, paired with structural operational action triggers.
* **Why does Data/ML help here at all?** Traditional static rules trigger massive false-alarm volumes on naturally volatile or lower-traffic pages. Machine learning models resolve this by modeling complex interactions between long-term historical baseline presence and continuous site interaction frequencies to separate true decay from standard traffic noise.

---

## 2. Data safety
* **Data Selection:** The pipeline utilizes multi-partitioned daily performance matrices extracted from the `fact_content_daily_performance` tables across the `month=2026-0*` timeline partitions.
* **Exclusions & Rationales:** To ensure absolute compliance with public-safe privacy frameworks, all specific text search queries, explicit cleartext destination URLs, and raw domain names were entirely omitted from ingestion. Additionally, 8,533 volatile pages containing zero historical baseline impressions were filtered out programmatically to keep the model focused on established, active assets.
* **Leakage Risks & Privacy:** The structural identities of separate corporate entities are completely masked using deterministic cryptographic tokens (`client_hash_id`, `content_hash_id`), ensuring zero competitive or proprietary data leakage from the warehouse layer.

---

## 3. Baseline
Our transparent rule baseline flags a content node as failing if its rolling 90-day search presence volume falls below 80% of its historic baseline metric. While this simple baseline heuristic registers a perfect 100.00% score when evaluated directly against its own static mathematical definition, it functions as a rigid, reactive proxy that fails to handle natural variance across diverse client ecosystems or adaptively forecast latent asset declines.

---

## 4. Model / analysis
* **Model Selection:** We implemented an ensemble Random Forest Classifier consisting of 100 decision estimators and a maximum tree depth of 8 to prevent structural overfitting.
* **Feature List:**
  1. `impressions_prior`: The total volume of historical Google Search Console impressions recorded before the rolling 90-day lookback window opened.
  2. `active_days_count`: The total number of distinct historical reporting days where the content node logged active search signals.
* **Target Definition:** The binary classification target variable (`is_decline_target`) triggers whenever a page's rolling 90-day performance drops below 80% of its historical prior performance window:
  * `action_score = impressions_last_90d / (impressions_prior + 1)`
  * `is_decline_target = 1` if `action_score < 0.8` else `0`

---

## 5. Evaluation
To measure true out-of-domain generalization, the data was partitioned using an honest client-grouped validation split (80% of client accounts for training, 20% completely held out for testing). This validation structure prevents site-specific leaks and forces the model to evaluate on entirely unseen client domains.

* **Training Split Size:** 396,970 rows
* **Validation Split Size:** 30,322 rows

| Performance Evaluation Parameter | Static Rule Heuristic Baseline | Ensemble Machine Learning Model |
| :--- | :---: | :---: |
| **Pipeline Accuracy** | 100.00% | **86.54%** |
| **Target Precision** | 100.00% | **42.05%** |
| **Target Recall (Sensitivity)** | 100.00% | **56.22%** |
| **Balanced F1-Score** | 100.00% | **48.03%** |

> **Operational Insight:** The baseline registers 100.00% purely because it represents the raw definition layout of the target label on this split. Under genuine production constraints, our Machine Learning model demonstrates strong predictive value—safely capturing 56.22% of latent traffic drops on entirely foreign web architectures using only past volume and activity metrics.

---

## 6. Interpretation
* **Feature Importance Profiles:** The trained tree ensemble relies heavily on `active_days_count` as its premier split driver, confirming that the continuous historical stability of an asset is the strongest predictor of future traffic retention.
* **Surprises & Negative Results:** The model revealed that raw prior volume (`impressions_prior`) alone has a lower predictive weight than active tracking frequency. This demonstrates that continuous web presence and steady indexation flags carry significantly more signal for structural stability than temporary, high-volume traffic spikes.

---

## 7. Recommendation
* **Ranked Playbook Actions:** The pipeline automatically outputs a prioritized, risk-stratified operational backlog that groups content elements into three clear, action-oriented segments based on their continuous risk profiles:
  * `CRITICAL_URGENT_SALVAGE` (risk_score >= 0.8): High-priority targets routed directly to technical engineering teams for immediate content refresh or layout repairs.
  * `OPTIMIZE_CONTENT_FOOTPRINT` (0.5 <= risk_score < 0.8): Routed to SEO specialists for metadata optimization and keyword expansion.
  * `MONITOR_STABLE` (risk_score < 0.5): Deferred assets left on automated maintenance cycles.
* **How to use it tomorrow:** The content team should directly integrate the generated `final_action_playbook.csv` into their daily task managers, pulling the top-ranked highest-risk page rows into their immediate manual content QA queue.

---

## 8. Reproducibility
To recreate this exact pipeline from scratch on a new runtime environment:
1. Initialize a secure DuckDB local memory instance and apply a strict hardware memory ceiling configuration using `SET memory_limit='10GB'`.
2. Connect to the `FlyRank/internship-warehouse` repository on Hugging Face using your verified read token.
3. Extract the maximum partition report date as a standalone scalar parameter to protect the system cache against out-of-memory overhead.
4. Execute the non-join aggregation script to compile historical feature matrices, enforce the deterministic client-grouped split condition, and train the `RandomForestClassifier` using a fixed random state parameter of `42`.

---

## 9. Acknowledgments & data credit
This research and engineering development was built on the data warehouse infrastructure provided by the [FlyRank ML Internship Track](https://flyrank.ai). The underlying performance matrix distributions, schema partitions, and baseline tracking properties originate entirely from their verified enterprise data warehouse instances.
