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

### 3) Avoid dense TF-IDF conversion for email text
**Current state**
- Sparse TF-IDF matrix is converted to dense with `.toarray()`.

**Why this hurts efficiency**
- Significant memory blow-up.
- Slower preprocessing and fit time.

**Recommendation**
- Keep sparse matrices when possible.
- If classifier requires dense input, reduce feature size and compare alternatives (`LinearSVC`, `LogisticRegression`, `SGDClassifier`) that are strong and efficient on sparse text.

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

### 5) Calibrate thresholds and return confidence
**Current state**
- Output is hard label only: "Phishing" / "Legitimate".

**Why this hurts practical accuracy**
- No confidence shown to users.
- No threshold tuning for high-recall or high-precision operating modes.

**Recommendation**
- Use `predict_proba`; tune threshold by use-case.
- Display confidence + key feature explanations (e.g., top tokens/features) for trust.

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

### 8) Improve deployment/security configuration for production correctness
**Current state**
- `DEBUG=True` and hard-coded secret key in settings.

**Why this matters**
- Not directly model accuracy, but impacts reliability and safety of the AI service.

**Recommendation**
- Use environment variables for `SECRET_KEY`, `DEBUG`, allowed hosts.
- Add structured logging around inference latency and prediction distribution.

### 9) Normalize dependency file encoding
**Current state**
- `setup.txt` appears UTF-16-encoded with null bytes.

**Why this hurts efficiency**
- Some tooling may fail or parse slowly; onboarding friction increases.

**Recommendation**
- Convert to UTF-8 and consider renaming to `requirements.txt`.

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
