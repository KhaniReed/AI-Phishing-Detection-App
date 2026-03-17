
# Task-Focused Resources for AI Phishing Detection Improvements

This file intentionally contains only implementation-focused articles and YouTube videos, mapped to each recommended improvement task.

## 1) Move training out of web request path
- Article: https://docs.djangoproject.com/en/stable/howto/custom-management-commands/
- Article: https://scikit-learn.org/stable/model_persistence.html
- Video: https://www.youtube.com/results?search_query=django+custom+management+command+tutorial
- Video: https://www.youtube.com/results?search_query=joblib+save+load+sklearn+model

## 2) Fix leakage (split first, then SMOTE)
- Article: https://imbalanced-learn.org/stable/common_pitfalls.html
- Article: https://imbalanced-learn.org/stable/references/generated/imblearn.pipeline.Pipeline.html
- Article: https://scikit-learn.org/stable/modules/model_evaluation.html
- Video: https://www.youtube.com/results?search_query=smote+train+test+split+data+leakage
- Video: https://www.youtube.com/results?search_query=statquest+precision+recall+pr+auc

## 3) Keep TF-IDF sparse (avoid `.toarray()`)
- Article: https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html
- Article: https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html
- Article: https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.SGDClassifier.html
- Video: https://www.youtube.com/results?search_query=tfidf+scikit+learn+sparse+matrix+tutorial
- Video: https://www.youtube.com/results?search_query=linear+svm+logistic+regression+text+classification+sklearn

## 4) Improve URL/email routing
- Article: https://docs.python.org/3/library/urllib.parse.html
- Article: https://github.com/john-kurkowski/tldextract
- Article: https://docs.python.org/3/library/re.html
- Video: https://www.youtube.com/results?search_query=python+urlparse+tutorial
- Video: https://www.youtube.com/results?search_query=python+url+validation+tutorial

## 5) Add confidence + threshold tuning
- Article: https://scikit-learn.org/stable/modules/calibration.html
- Article: https://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_recall_curve.html
- Article: https://shap.readthedocs.io/en/latest/
- Video: https://www.youtube.com/results?search_query=classification+threshold+precision+recall+tutorial
- Video: https://www.youtube.com/results?search_query=shap+python+tutorial

## 6) Add tests and regression checks
- Article: https://docs.djangoproject.com/en/stable/topics/testing/
- Article: https://docs.pytest.org/en/stable/
- Article: https://pytest-django.readthedocs.io/en/latest/
- Video: https://www.youtube.com/results?search_query=django+unit+testing+tutorial
- Video: https://www.youtube.com/results?search_query=pytest+parametrize+tutorial

## 7) Separate experimentation from runtime inference
- Article: https://docs.astral.sh/ruff/
- Article: https://docs.python.org/3/howto/logging.html
- Video: https://www.youtube.com/results?search_query=ml+project+structure+training+inference+separation
- Video: https://www.youtube.com/results?search_query=ruff+python+tutorial

## 8) Production security/config hardening
- Article: https://docs.djangoproject.com/en/stable/howto/deployment/checklist/
- Article: https://12factor.net/config
- Article: https://pypi.org/project/python-decouple/
- Video: https://www.youtube.com/results?search_query=django+production+settings+environment+variables
- Video: https://www.youtube.com/results?search_query=python+structured+logging+tutorial

## 9) Dependency file normalization
- Article: https://pip.pypa.io/en/stable/reference/requirements-file-format/
- Article: https://packaging.python.org/en/latest/
- Video: https://www.youtube.com/results?search_query=requirements+txt+best+practices+python
- Video: https://www.youtube.com/results?search_query=convert+file+encoding+utf-8+vscode