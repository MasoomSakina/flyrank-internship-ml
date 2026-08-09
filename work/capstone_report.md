# Capstone Report — Growth and Momentum Prediction

- **Author:** Masoom Sakina
- **Lane:** Freestyle (Content Refresh & Traffic Decay Anomaly Detection)
- **Repo:** https://github.com/MasoomSakina/flyrank-internship-ml
- **Date:** 2026-08-09

## 0. Abstract
This analysis systematically identifies high-impact content pages experiencing organic traffic decay. Utilizing the anonymized content refresh dataset from the FlyRank warehouse release, we engineered a rigorous, leak-free classification pipeline powered by a Random Forest classifier. Evaluated against a transparent rule-based baseline on an 80/20 stratified validation split, the model achieved an F1-Score of 0.7219, successfully outperforming the baseline score of 0.6672. The output is deployed as an automated, data-driven prioritization queue (`action_playbook_ranked.csv`) designed to direct editorial and content refresh resources toward high-exposure anomalies efficiently.

## 1. Problem framing
This work supports the operational decision of identifying declining content pages that yield the highest risk of lost traffic and require immediate content interventions versus pages experiencing normal operational variance. 
* **Unit of Analysis:** Individual content pages within the catalog.
* **Primary Output:** A prioritized action queue (`RECOMMEND_REFRESH` vs. `NO_ACTION`) supplemented by risk scores and standardized reason codes.
* **Human Action:** Content strategists and SEO leads use the queue to allocate editorial refresh resources effectively.
* **Cost of Error:** A false positive expends limited editorial bandwidth on stable pages, whereas a false negative permits compounding organic traffic erosion and missed conversion opportunities. 
Machine learning provides value here by scaling anomaly detection across thousands of pages faster, more consistently, and more objectively than manual oversight.

## 2. Data safety
We utilized the anonymized content refresh dataset from the FlyRank warehouse release (`content_refresh_anonymized.csv`), spanning historical windows from late 2024 through mid-2026. To guarantee public safety and strict privacy compliance:
* All client-specific identifiers, domain names, and URLs were explicitly excluded.
* Future traffic windows were excluded to prevent retrospective leakage.
* Internal product flags and label-derived proxy fields (such as `trend_numeric` and `trend_pct`) were systematically removed from feature vectors to preserve absolute model integrity.
* We confirm that no client-identifying or sensitive information exists anywhere within the `work/` directory.

## 3. Baseline
The transparent baseline model is a fixed business-logic rule that flags content pages exhibiting an impression volume $\ge 1,000$ alongside a downward trend direction. This serves as a fair and direct comparison because it mirrors the incumbent heuristic used by content teams prior to algorithmic deployment. Evaluated on the exact same validation split, this baseline achieved an F1-Score of 0.6672.

## 4. Model / analysis
We framed this objective as a binary classification task to predict whether a content page is experiencing a performance-decaying anomaly or normal variance. A Random Forest classifier was selected for its robustness against non-linear interactions and its transparency in calculating feature importances. 
* **Feature Set:** Restricted strictly to safe, independent historical metrics (`impressions_90d`). Label-derived covariates were intentionally omitted to prevent target leakage.
* **Target Definition:** Binary proxy defined as 1 if the historical trend direction is 'down' and 0 otherwise.

## 5. Evaluation
We implemented an 80% training and 20% validation split utilizing stratified sampling (`stratify=y`) to maintain proportional representation of decaying anomalies across subsets. The primary evaluation metric is the F1-Score to balance precision and recall constraints. 
* **Comparative Performance:** On the validation split, the Random Forest model achieved an F1-Score of **0.7219**, outperforming the Week-4 Baseline Rule score of **0.6672**.
* **Error Analysis:** Model misclassifications concentrate primarily around borderline pages sitting near the 1,000-impression threshold, where minor daily variance introduces noise into the historical trend direction label.

## 6. Interpretation
The model successfully isolates high-exposure performance drops by anchoring decision boundaries on historical impression volume. By strictly restricting features to prevent data leakage, the model establishes an honest baseline that effectively prioritizes high-impact operational targets while maintaining conservative assumptions on stable pages. A transparently acknowledged trade-off on absolute recall for stable pages represents a natural outcome of privacy-safe, leak-free feature curation.

## 7. Recommendation
The output delivers an actionable, ranked queue of high-exposure pages exhibiting significant negative variance relative to baseline performance (`RECOMMEND_REFRESH`). FlyRank editors and content leads should deploy this queue to prioritize editorial updates on top-ranked anomalies rather than expending resources on low-traffic long-tail items. 
* **Confidence & Limits:** Confidence is high regarding historical observation and prioritization utility. However, explicit boundaries apply: this work cannot claim causal proof regarding *why* a page is declining (e.g., core algorithm updates versus shifting user intent), nor does it guarantee that refreshing a flagged page will reverse traffic decay.

## 8. Reproducibility
To replicate the complete pipeline from a fresh clone:
1. Ensure the required Python environment dependencies are installed (`pip install -r requirements.txt`).
2. Execute the pipeline notebooks sequentially starting from `work/notebooks/capstone.ipynb` under Python 3.14.5.
3. Random seeds are globally fixed (`random_state=42`) across data splitting and model training to guarantee deterministic reproduction.
4. Auditable runtime receipts and evaluation artifacts are serialized and permanently stored in `work/outputs/metrics.json` and `work/outputs/action_playbook_ranked.csv`.

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset [https://flyrank.ai](https://flyrank.ai). Crediting this data source reflects standard research practice and fulfills the required deployment specification.