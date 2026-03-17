# Improvement Resources for AI Phishing Detection App

This curated list maps common improvement areas for phishing-detection projects to high-quality research papers and practical video tutorials.

## 1) Data quality, datasets, and labeling strategy

### Research articles
1. **Nazario, J. (2007). _Phishing Corpus_**  
   URL: https://monkey.org/~jose/wiki/doku.php?id=PhishingCorpus  
   Why useful: Historical baseline corpus for building/benchmarking email-phishing filters.

2. **Marchal, S., Armano, G., Gröndahl, T., Asokan, N., Sako, K., & Singh, N. (2014). _Off-the-Hook: An Efficient and Usable Client-Side Phishing Prevention Application_ (IEEE TDSC).**  
   URL: https://ieeexplore.ieee.org/document/6683034  
   Why useful: Demonstrates feature engineering and practical anti-phishing deployment constraints.

3. **Mao, J., Bian, J., Tian, W., Zhu, S., Wei, T., Li, A., & Liang, Z. (2018). _Phishing Page Detection via Learning Classifiers from Page Layout Feature_ (EURASIP JIS).**  
   URL: https://jis-eurasipjournals.springeropen.com/articles/10.1186/s13635-018-0076-0  
   Why useful: Useful for UI/layout feature extraction for URL + webpage classifiers.

### YouTube videos
1. **Data-Centric AI (Andrew Ng / DeepLearning.AI talks)**  
   URL: https://www.youtube.com/results?search_query=data+centric+ai+andrew+ng  
   Why useful: Practical guidance to improve label quality and class definitions.

2. **Kaggle - Feature Engineering tutorials**  
   URL: https://www.youtube.com/results?search_query=kaggle+feature+engineering+tutorial  
   Why useful: Actionable workflows for creating robust, leakage-safe features.

---

## 2) Stronger model architecture (traditional ML + deep learning)

### Research articles
1. **Sahingoz, O. K., Buber, E., Demir, O., & Diri, B. (2019). _Machine Learning Based Phishing Detection from URLs_ (Expert Systems with Applications).**  
   URL: https://www.sciencedirect.com/science/article/pii/S0957417419302433  
   Why useful: Strong baseline comparison across classical models for URL-based detection.

2. **Aljofey, A., Jiang, Q., Rasool, A., et al. (2020). _An Effective Phishing Detection Model Based on Character-Level CNN, URL and HTML Features_ (Electronics).**  
   URL: https://www.mdpi.com/2079-9292/9/9/1514  
   Why useful: Combines lexical URL signals + content features with deep learning.

3. **Bahnsen, A. C., Bohorquez, E. C., Villegas, S., Vargas, J., & González, F. A. (2017). _Classifying Phishing URLs Using Recurrent Neural Networks_ (eCrime Researchers Summit).**  
   URL: https://arxiv.org/abs/1708.08579  
   Why useful: Sequence-based URL modeling ideas for detecting obfuscation patterns.

### YouTube videos
1. **StatQuest - Gradient Boosting / XGBoost / Evaluation**  
   URL: https://www.youtube.com/results?search_query=statquest+xgboost  
   Why useful: Clear conceptual grounding for high-performing tabular baselines.

2. **DeepLearningAI NLP specialization overviews**  
   URL: https://www.youtube.com/results?search_query=deeplearningai+nlp+sequence+models  
   Why useful: Helps when moving from handcrafted features to sequence models.

---

## 3) Explainability, trust, and analyst workflow

### Research articles
1. **Ribeiro, M. T., Singh, S., & Guestrin, C. (2016). _“Why Should I Trust You?” Explaining the Predictions of Any Classifier_ (KDD).**  
   URL: https://arxiv.org/abs/1602.04938  
   Why useful: Foundation for local explanations (LIME) in phishing-alert UX.

2. **Lundberg, S. M., & Lee, S.-I. (2017). _A Unified Approach to Interpreting Model Predictions_ (NeurIPS).**  
   URL: https://arxiv.org/abs/1705.07874  
   Why useful: SHAP for feature-level transparency and triage dashboards.

3. **Arrieta, A. B., et al. (2020). _Explainable Artificial Intelligence (XAI): Concepts, Taxonomies, Opportunities and Challenges toward Responsible AI_ (Information Fusion).**  
   URL: https://www.sciencedirect.com/science/article/pii/S1566253519308103  
   Why useful: Broad framework to design interpretable and responsible detection systems.

### YouTube videos
1. **SHAP official/introduction talks**  
   URL: https://www.youtube.com/results?search_query=shap+model+explainability+tutorial  
   Why useful: Practical steps to generate and interpret SHAP plots.

2. **LIME tutorials**  
   URL: https://www.youtube.com/results?search_query=lime+explainable+ai+tutorial  
   Why useful: Quick prototyping of local explanations in Flask/Streamlit apps.

---

## 4) Robust evaluation and production monitoring

### Research articles
1. **Sculley, D., et al. (2015). _Hidden Technical Debt in Machine Learning Systems_ (NeurIPS).**  
   URL: https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems  
   Why useful: Essential guidance for production-quality ML lifecycle.

2. **Lipton, Z. C., Elkan, C., & Narayanaswamy, B. (2014). _Thresholding Classifiers to Maximize F1 Score_**  
   URL: https://arxiv.org/abs/1402.1892  
   Why useful: Helps set decision thresholds for phishing false positive/negative trade-offs.

3. **Gama, J., Žliobaitė, I., Bifet, A., Pechenizkiy, M., & Bouchachia, A. (2014). _A Survey on Concept Drift Adaptation_ (ACM CSUR).**  
   URL: https://dl.acm.org/doi/10.1145/2523813  
   Why useful: Critical for adapting phishing models as attacker tactics evolve.

### YouTube videos
1. **MLOps and model monitoring talks (Evidently AI / Full Stack Deep Learning)**  
   URL: https://www.youtube.com/results?search_query=evidently+ai+model+monitoring  
   Why useful: Hands-on drift, quality, and data-monitoring setup.

2. **Confusion matrix, ROC-AUC, PR-AUC deep dives (StatQuest)**  
   URL: https://www.youtube.com/results?search_query=statquest+roc+auc+precision+recall  
   Why useful: Better metric selection for imbalanced phishing datasets.

---

## 5) Security engineering and adversarial resilience

### Research articles
1. **Goodfellow, I. J., Shlens, J., & Szegedy, C. (2014). _Explaining and Harnessing Adversarial Examples_**  
   URL: https://arxiv.org/abs/1412.6572  
   Why useful: Core ideas for defending ML detectors against adversarial manipulation.

2. **Biggio, B., & Roli, F. (2018). _Wild Patterns: Ten Years After the Rise of Adversarial Machine Learning_ (Pattern Recognition).**  
   URL: https://arxiv.org/abs/1712.03141  
   Why useful: Threat modeling and attack/defense strategies for security ML.

3. **Barreno, M., Nelson, B., Sears, R., Joseph, A. D., & Tygar, J. D. (2010). _The Security of Machine Learning_ (Machine Learning).**  
   URL: https://link.springer.com/article/10.1007/s10994-010-5188-5  
   Why useful: Canonical taxonomy of poisoning, evasion, and privacy attacks.

### YouTube videos
1. **Adversarial ML lectures (MIT / CMU / university channels)**  
   URL: https://www.youtube.com/results?search_query=adversarial+machine+learning+lecture  
   Why useful: Security-focused perspective to harden phishing classifiers.

2. **OWASP talks on phishing and social engineering defenses**  
   URL: https://www.youtube.com/results?search_query=owasp+phishing+defense  
   Why useful: Complements model improvements with policy and user-layer mitigations.

---

## Suggested 4-week learning plan

- **Week 1:** Reproduce strong classical baselines (LogReg, RF, XGBoost), improve dataset split strategy, and calibrate thresholds.
- **Week 2:** Add deep URL sequence model (CNN/RNN/Transformer-lite) and compare with baseline under identical evaluation protocol.
- **Week 3:** Integrate SHAP/LIME explanations into prediction outputs and error analysis dashboard.
- **Week 4:** Add monitoring metrics (drift, false positive trend, confidence shift) and write retraining triggers.

## Fast implementation checklist

- Build a **single evaluation harness** with fixed train/val/test splits.
- Track at minimum: **precision, recall, F1, PR-AUC, confusion matrix**.
- Choose threshold based on business cost of false positives vs false negatives.
- Save model cards and feature documentation for every trained model.
- Add monthly data-drift checks and quarterly model retraining review.
