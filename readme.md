# ML-Based Intrusion Detection System (NSL-KDD)

A machine learning project that classifies network connections as normal or as one of four attack types, with an interactive Streamlit dashboard that explains how the models work and where they fail.

Built as a college project. It is a learning project, not a production-ready IDS.

## Overview

Random forest classifiers are trained on the [NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html) dataset, a standard benchmark for network intrusion detection. Each record describes one network connection with 41 features (bytes sent, connection counts, login status, error rates and more).

The models answer two questions:

- **Binary:** is a connection normal or an attack?
- **Multi-class:** is it normal, **DoS** (denial of service), **Probe** (scanning), **R2L** (remote to local, such as password guessing) or **U2R** (user to root, privilege escalation)?

## Results

Evaluated on the official NSL-KDD test file (22,544 connections).

| Model | Accuracy | Attack recall |
|---|---|---|
| Logistic regression (binary) | 0.67 | 0.44 |
| Random forest (binary) | 0.76 | 0.60 |

Random forest, 5 categories (accuracy 0.75, macro-average recall 0.50):

| Category | Recall |
|---|---|
| Normal | 0.97 |
| DoS | 0.77 |
| Probe | 0.67 |
| R2L | 0.04 |
| U2R | 0.03 |

### Key finding: rare attacks are missed

The model catches most DoS and probe attacks but almost no R2L or U2R attacks. Error analysis points to a mismatch between training and test data rather than simple class imbalance:

- 89% of R2L training records are a single attack type (`warezclient`), while the test set is dominated by `guess_passwd` and `warezmaster`.
- About 24% of R2L test records are attack types that never appear in training, and none of them were detected.
- Reweighting classes (`class_weight="balanced"`) did not improve rare-class recall.
- The most important features are traffic volume (`src_bytes`, `dst_bytes`) and repeated connections to the same service, which suit DoS and probes but not password guessing, since that resembles an ordinary login.

## Features

- Preprocessing: categorical encoding (with handling of values unseen in training) and feature scaling
- Logistic regression baseline and random forest models
- Per-class precision and recall, confusion matrices, feature importance
- R2L error analysis
- Streamlit dashboard with a live prediction tab: pick a real test connection, edit its values, and see the predicted category

## Project structure

```
ML-Based-IDS/
├── ids_notebook.ipynb   # data loading, preprocessing, model training, evaluation, error analysis
├── ids_app.py           # Streamlit dashboard (trains the models and explains the results)
├── requirements.txt     # Python dependencies
├── README.md            # project documentation
├── .gitignore           # keeps the dataset files out of the repository
│
├── KDDTrain+.txt        # NSL-KDD training set (download separately, not in repo)
└── KDDTest+.txt         # NSL-KDD test set (download separately, not in repo)
```

| File | Purpose |
|---|---|
| `ids_notebook.ipynb` | Step-by-step analysis: loading, EDA, encoding and scaling, binary and 5-class models, confusion matrices, feature importance, R2L error analysis |
| `ids_app.py` | Interactive dashboard with tabs for how the pipeline works, the data, results, why rare attacks are missed, and a live prediction demo |
| `requirements.txt` | Libraries needed to run the notebook and the dashboard |
| `KDDTrain+.txt`, `KDDTest+.txt` | Dataset files, expected in the same folder as `ids_app.py` |

The dataset files are not included in this repository.

## Getting started

1. **Get the data.** Download NSL-KDD from [Kaggle](https://www.kaggle.com/datasets/hassan06/nslkdd) and place `KDDTrain+.txt` and `KDDTest+.txt` in the project folder.
2. **Install dependencies.**
   ```
   pip install -r requirements.txt
   ```
3. **Run the dashboard.**
   ```
   streamlit run ids_app.py
   ```
   The first launch trains three random forests, which takes a minute or two. After that the results are cached.

To reproduce the analysis, open the notebook in Jupyter and run the cells in order.

## Tech stack

Python, pandas, NumPy, scikit-learn, Streamlit, Jupyter

## Limitations

- NSL-KDD is an older, simulated dataset, and real network traffic looks different.
- The test set deliberately differs from the training set, which lowers scores on rare attack types.
- Results were produced with a single train/test split and no hyperparameter tuning.

## Future work

- Try gradient boosting or XGBoost and compare with random forest
- Oversampling (SMOTE) for rare classes
- Cross-validation for more reliable estimates
- Evaluate on a more modern dataset such as CICIDS2017

## Reference

M. Tavallaee, E. Bagheri, W. Lu and A. A. Ghorbani, "A detailed analysis of the KDD CUP 99 data set," IEEE Symposium on Computational Intelligence for Security and Defense Applications, 2009.

## Author

Deepak S
