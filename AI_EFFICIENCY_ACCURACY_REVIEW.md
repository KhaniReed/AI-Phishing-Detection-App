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


**Articles & videos to implement this**
- Article: Django custom management commands (official docs)  
  https://docs.djangoproject.com/en/stable/howto/custom-management-commands/
- Article: scikit-learn model persistence (joblib/pickle)  
  https://scikit-learn.org/stable/model_persistence.html
- Video: Django management commands tutorial  
  https://www.youtube.com/results?search_query=django+custom+management+command+tutorial
- Video: Save/load ML models with joblib  
  https://www.youtube.com/results?search_query=joblib+save+load+sklearn+model

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

**Articles & videos to implement this**
- Article: imbalanced-learn `Pipeline` docs  
  https://imbalanced-learn.org/stable/references/generated/imblearn.pipeline.Pipeline.html
- Article: imbalanced-learn common pitfalls (data leakage)  
  https://imbalanced-learn.org/stable/common_pitfalls.html
- Article: scikit-learn metrics guide (precision/recall/F1/PR curves)  
  https://scikit-learn.org/stable/modules/model_evaluation.html
- Video: SMOTE + train/test split correctly  
  https://www.youtube.com/results?search_query=smote+train+test+split+data+leakage
- Video: Precision/Recall and PR-AUC explained (StatQuest)  
  https://www.youtube.com/results?search_query=statquest+precision+recall+pr+auc

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

**Articles & videos to implement this**
- Article: scikit-learn text classification tutorial (sparse pipelines)  
  https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html
- Article: `TfidfVectorizer` docs  
  https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html
- Article: linear models that work well on sparse features (`SGDClassifier`)  
  https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.SGDClassifier.html
- Video: TF-IDF with sparse matrix workflow in scikit-learn  
  https://www.youtube.com/results?search_query=tfidf+scikit+learn+sparse+matrix+tutorial
- Video: Logistic Regression / Linear SVM for text classification  
  https://www.youtube.com/results?search_query=linear+svm+logistic+regression+text+classification+sklearn

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

**Articles & videos to implement this**
- Article: Python `urllib.parse` docs (`urlparse`)  
  https://docs.python.org/3/library/urllib.parse.html
- Article: `tldextract` for reliable domain/TLD parsing  
  https://github.com/john-kurkowski/tldextract
- Article: Python regex docs (only for supporting checks, not sole routing)  
  https://docs.python.org/3/library/re.html
- Video: URL parsing in Python (`urllib.parse`)  
  https://www.youtube.com/results?search_query=python+urlparse+tutorial
- Video: Building robust URL validators in Python  
  https://www.youtube.com/results?search_query=python+url+validation+tutorial

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

**Articles & videos to implement this**
- Article: scikit-learn probability calibration  
  https://scikit-learn.org/stable/modules/calibration.html
- Article: `precision_recall_curve` docs for threshold selection  
  https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_recall_curve.html
- Article: SHAP documentation (feature-level explanations)  
  https://shap.readthedocs.io/en/latest/
- Video: Choosing classification thresholds with precision-recall tradeoff  
  https://www.youtube.com/results?search_query=classification+threshold+precision+recall+tutorial
- Video: SHAP explainability tutorial in Python  
  https://www.youtube.com/results?search_query=shap+python+tutorial

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

**Articles & videos to implement this**
- Article: Django testing overview (official docs)  
  https://docs.djangoproject.com/en/stable/topics/testing/
- Article: pytest docs (parametrization for edge-case tests)  
  https://docs.pytest.org/en/stable/
- Article: pytest-django docs  
  https://pytest-django.readthedocs.io/en/latest/
- Video: Django unit testing tutorial  
  https://www.youtube.com/results?search_query=django+unit+testing+tutorial
- Video: pytest parametrized tests tutorial  
  https://www.youtube.com/results?search_query=pytest+parametrize+tutorial

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

**Articles & videos to implement this**
- Article: Ruff docs (unused imports, linting for cleanup)  
  https://docs.astral.sh/ruff/
- Article: Python logging HOWTO (replace print metrics with structured logging)  
  https://docs.python.org/3/howto/logging.html
- Video: Clean architecture for ML projects  
  https://www.youtube.com/results?search_query=ml+project+structure+training+inference+separation
- Video: Ruff + Python linting setup  
  https://www.youtube.com/results?search_query=ruff+python+tutorial

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

**Articles & videos to implement this**
- Article: Django deployment checklist (official docs)  
  https://docs.djangoproject.com/en/stable/howto/deployment/checklist/
- Article: Twelve-Factor App config (env vars)  
  https://12factor.net/config
- Article: `python-decouple` (manage secrets/config)  
  https://pypi.org/project/python-decouple/
- Video: Django production settings & environment variables  
  https://www.youtube.com/results?search_query=django+production+settings+environment+variables
- Video: Structured logging in Python applications  
  https://www.youtube.com/results?search_query=python+structured+logging+tutorial

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

**Articles & videos to implement this**
- Article: pip requirements file format  
  https://pip.pypa.io/en/stable/reference/requirements-file-format/
- Article: Python packaging user guide (dependency management basics)  
  https://packaging.python.org/en/latest/
- Video: requirements.txt best practices  
  https://www.youtube.com/results?search_query=requirements+txt+best+practices+python
- Video: handling file encoding in Python/VS Code  
  https://www.youtube.com/results?search_query=convert+file+encoding+utf-8+vscode

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
