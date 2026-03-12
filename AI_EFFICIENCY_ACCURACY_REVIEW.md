# AI Phishing Detection: Efficiency & Accuracy Improvement Review

## Scope reviewed
- Django application structure and configuration.
- Inference/training pipeline in `detector/views.py`.
- Basic project hygiene (dependencies, tests, docs).

## High-impact improvements (prioritized)

### 1) Move model training out of request-serving module (highest impact)
**Current state**
- Datasets are loaded and two XGBoost models are trained at import time in `views.py`.
- This executes whenever the Django process starts/reloads.

**Why this hurts efficiency**
- Cold start is expensive.
- Memory and CPU are consumed in every app process.
- Django auto-reload in development can retrain multiple times.

**Recommendation**
- Create an offline training script/management command (e.g. `python manage.py train_models`).
- Persist artifacts (`tfidf_vectorizer`, `scaler`, `email_model`, `url_model`) with `joblib`.
- On app startup, only load serialized artifacts; never retrain in web workers.

**Learning resources mapped to this improvement**
- Article: **Sculley et al. (2015), _Hidden Technical Debt in Machine Learning Systems_**  
  https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems
- Video: **Evidently AI / FSDL model monitoring & MLOps talks**  
  https://www.youtube.com/results?search_query=evidently+ai+model+monitoring

### 2) Fix data leakage in model evaluation (highest impact)
**Current state**
- SMOTE oversampling runs *before* train/test split for both email and URL datasets.

**Why this hurts accuracy validity**
- Synthetic samples are influenced by full data distribution, leaking test-set information.
- Reported accuracy can be over-optimistic and not representative in production.

**Recommendation**
- Split first, then apply SMOTE only on the training fold.
- Prefer `imblearn.pipeline.Pipeline` + cross-validation for robust evaluation.
- Track precision, recall, F1, and PR-AUC (not just accuracy).

**Learning resources mapped to this improvement**
- Article: **Lipton et al. (2014), _Thresholding Classifiers to Maximize F1 Score_**  
  https://arxiv.org/abs/1402.1892
- Article: **Sahingoz et al. (2019), _Machine Learning Based Phishing Detection from URLs_**  
  https://www.sciencedirect.com/science/article/pii/S0957417419302433
- Video: **StatQuest confusion matrix, ROC-AUC, precision/recall deep dives**  
  https://www.youtube.com/results?search_query=statquest+roc+auc+precision+recall

### 3) Avoid dense TF-IDF conversion for email text
**Current state**
- Sparse TF-IDF matrix is converted to dense with `.toarray()`.

**Why this hurts efficiency**
- Significant memory blow-up.
- Slower preprocessing and fit time.

**Recommendation**
- Keep sparse matrices when possible.
- If classifier requires dense input, reduce feature size and compare alternatives (`LinearSVC`, `LogisticRegression`, `SGDClassifier`) that are strong and efficient on sparse text.

**Learning resources mapped to this improvement**
- Article: **Sahingoz et al. (2019), _Machine Learning Based Phishing Detection from URLs_**  
  https://www.sciencedirect.com/science/article/pii/S0957417419302433
- Video: **Kaggle feature engineering tutorials**  
  https://www.youtube.com/results?search_query=kaggle+feature+engineering+tutorial

### 4) Improve URL/email type routing logic
**Current state**
- Routing between URL and email model depends on one regex in `is_url`.

**Why this hurts accuracy**
- Regex misses many valid URLs and can misroute mixed content.
- Wrong model selection causes avoidable false positives/negatives.

**Recommendation**
- Use robust parsing (`urlparse`) plus heuristics (scheme, netloc/TLD checks).
- Add fallback logic for mixed text containing URLs.
- Consider a lightweight first-stage classifier to detect input type.

**Learning resources mapped to this improvement**
- Article: **Bahnsen et al. (2017), _Classifying Phishing URLs Using Recurrent Neural Networks_**  
  https://arxiv.org/abs/1708.08579
- Article: **Mao et al. (2018), _Phishing Page Detection via Learning Classifiers from Page Layout Feature_**  
  https://jis-eurasipjournals.springeropen.com/articles/10.1186/s13635-018-0076-0
- Video: **DeepLearning.AI sequence model overviews**  
  https://www.youtube.com/results?search_query=deeplearningai+nlp+sequence+models

### 5) Calibrate thresholds and return confidence
**Current state**
- Output is hard label only: "Phishing" / "Legitimate".

**Why this hurts practical accuracy**
- No confidence shown to users.
- No threshold tuning for high-recall or high-precision operating modes.

**Recommendation**
- Use `predict_proba`; tune threshold by use-case.
- Display confidence + key feature explanations (e.g., top tokens/features) for trust.

**Learning resources mapped to this improvement**
- Article: **Lipton et al. (2014), _Thresholding Classifiers to Maximize F1 Score_**  
  https://arxiv.org/abs/1402.1892
- Article: **Ribeiro et al. (2016), _Why Should I Trust You?_ (LIME)**  
  https://arxiv.org/abs/1602.04938
- Video: **LIME explainable AI tutorials**  
  https://www.youtube.com/results?search_query=lime+explainable+ai+tutorial
- Video: **SHAP explainability tutorials**  
  https://www.youtube.com/results?search_query=shap+model+explainability+tutorial

### 6) Add proper test coverage and regression checks
**Current state**
- Test module exists but has no real tests.

**Why this hurts reliability**
- Accuracy and behavior can regress silently.

**Recommendation**
- Add tests for:
  - URL feature extraction edge cases.
  - `is_url` behavior.
  - `predict_phishing` smoke tests with known examples.
- Add an offline evaluation script that logs metrics to file for each model version.

**Learning resources mapped to this improvement**
- Article: **Sculley et al. (2015), _Hidden Technical Debt in Machine Learning Systems_**  
  https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems
- Video: **StatQuest model evaluation series**  
  https://www.youtube.com/results?search_query=statquest+roc+auc+precision+recall

### 7) Remove dead imports and metrics code, tighten model experimentation loop
**Current state**
- `RandomForestClassifier`, `classification_report`, and `numpy` are imported but not used.
- Accuracy is printed at import time.

**Why this hurts efficiency/maintainability**
- Signals experimentation code is mixed with production path.
- Adds noise and makes profiling harder.

**Recommendation**
- Keep training/evaluation code in dedicated module.
- Keep runtime inference module minimal and deterministic.

**Learning resources mapped to this improvement**
- Article: **Sculley et al. (2015), _Hidden Technical Debt in Machine Learning Systems_**  
  https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems
- Video: **MLOps architecture talks (FSDL/Evidently AI)**  
  https://www.youtube.com/results?search_query=evidently+ai+model+monitoring

### 8) Improve deployment/security configuration for production correctness
**Current state**
- `DEBUG=True` and hard-coded secret key in settings.

**Why this matters**
- Not directly model accuracy, but impacts reliability and safety of the AI service.

**Recommendation**
- Use environment variables for `SECRET_KEY`, `DEBUG`, allowed hosts.
- Add structured logging around inference latency and prediction distribution.

**Learning resources mapped to this improvement**
- Article: **Barreno et al. (2010), _The Security of Machine Learning_**  
  https://link.springer.com/article/10.1007/s10994-010-5188-5
- Article: **Biggio & Roli (2018), _Wild Patterns_**  
  https://arxiv.org/abs/1712.03141
- Video: **OWASP phishing defense talks**  
  https://www.youtube.com/results?search_query=owasp+phishing+defense

### 9) Normalize dependency file encoding
**Current state**
- `setup.txt` appears UTF-16-encoded with null bytes.

**Why this hurts efficiency**
- Some tooling may fail or parse slowly; onboarding friction increases.

**Recommendation**
- Convert to UTF-8 and consider renaming to `requirements.txt`.

**Learning resources mapped to this improvement**
- Article: **Sculley et al. (2015), _Hidden Technical Debt in Machine Learning Systems_**  
  https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems
- Video: **Python packaging/dependency management tutorials**  
  https://www.youtube.com/results?search_query=python+requirements+txt+best+practices

## Suggested 2-week implementation plan

### Week 1 (foundation)
1. Refactor into separate modules:
   - `training.py` (offline training/eval)
   - `inference.py` (artifact loading + predict)
2. Fix leakage (split before SMOTE) and add metric reporting.
3. Persist artifacts and load them in Django view.

### Week 2 (quality/performance)
4. Add unit tests for routing + feature extraction + inference smoke tests.
5. Replace URL detection regex with parser-based logic.
6. Add confidence scores, threshold config, and basic monitoring logs.

## What to measure after changes
- Startup time of web app.
- Inference latency p50/p95.
- Memory footprint per worker.
- Precision/recall/F1/PR-AUC on held-out set.
- False-negative rate on phishing samples (critical safety metric).
