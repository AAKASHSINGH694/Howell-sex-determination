# Forensic Sex Estimation from Cranial Measurements

Sex estimation of unidentified skeletal remains using **Mahalanobis distance** classification and **Linear Discriminant Analysis (LDA)** on the Howells craniometric dataset, implemented in a Google Colab notebook.

Sex is one component of the forensic *biological profile* (sex, ancestry, stature, age at death), which is used to match unidentified remains against missing-person records. This project applies the multivariate discriminant-function approach that underlies the Fordisc software, as reviewed in:

> Austin D, King RE. *The Biological Profile of Unidentified Human Remains in a Forensic Context.* Acad Forensic Pathol. 2016;6(3):370-390. doi:[10.23907/2016.039](https://doi.org/10.23907/2016.039) (PMCID: PMC6474556)

## Background

Fordisc compares an unknown cranium's measurements with reference groups using discriminant function analysis. Groups are ranked by squared Mahalanobis distance (D²) from each group centroid, and posterior probabilities and typicalities are reported. This notebook applies the same idea to two classes (male/female):

- **Mahalanobis classifier**: assigns a case to the sex with the smaller D², using a pooled covariance matrix.
- **LDA**: scikit-learn `LinearDiscriminantAnalysis`, used as a comparison baseline.

## Dataset

The Howells craniometric dataset (`Howell.csv`): 2,524 individuals, 86 columns (ID, Sex, PopNum, Population, plus 82 cranial measurements). The data file is **not included** in this repository. Download it from its original source (W. W. Howells' craniometric data) and keep it locally.

## Pipeline

1. Load the data and treat `0` as a missing measurement.
2. Stratified 80/20 train/test split (`random_state=42`): 2,019 training and 505 test cases.
3. Remove variables with more than 20% missing values (82 to 71 variables, based on training data only).
4. Median imputation, with medians computed from the training data only to avoid leakage.
5. Remove highly correlated variables (|r| > 0.90): 71 to 60 variables.
6. Standardize, then forward sequential feature selection with LDA (10 variables, 5-fold CV).
7. Fit and evaluate the Mahalanobis classifier and LDA on the held-out test set.

## Results (test set, n = 505)

| Method | Accuracy |
|---|---|
| Mahalanobis distance | 0.893 |
| LDA | 0.887 |

Selected variables: `ZYB, MDH, ZMB, MLS, SOS, FRC, PAF, FOL, NAR, PAA`

Confusion matrix, Mahalanobis (rows = true M, F; columns = predicted M, F):

|  | Pred. M | Pred. F |
|---|---|---|
| **True M** | 239 | 35 |
| **True F** | 19 | 212 |

Both methods fall within the cranial accuracy range (roughly 83-90%) that the review cites for cranial sex classification.

## How to run

### Option 1: Google Colab (recommended)

1. Open the notebook in Colab (click the notebook file above, then use the **Open in Colab** button, or upload it at [colab.research.google.com](https://colab.research.google.com)).
2. Run the cells in order.
3. When the upload cell appears, choose your local `Howell.csv`.

### Option 2: Locally

```bash
git clone https://github.com/<your-username>/forensic-sex-estimation.git
cd forensic-sex-estimation
pip install -r requirements.txt
jupyter notebook
```

The notebook uses `google.colab.files` for uploading. If running locally, replace that cell with:

```python
df = pd.read_csv("Howell.csv")
```

A standalone script version is also available: `python src/sex_estimation.py --data Howell.csv --out results`

## Repository structure

```
forensic-sex-estimation/
├── notebooks/              # Colab notebook (upload here)
├── src/sex_estimation.py   # standalone script version of the pipeline
├── requirements.txt
├── LICENSE
└── README.md
```

## Limitations

- Howells is a historical, global reference sample. Under-represented groups (for example Hispanic individuals) can be misclassified because of sample-size and population differences, as the review notes for Fordisc.
- Postcranial measurements generally outperform cranial ones for sex estimation, and the pelvis remains the most reliable indicator. Metric results should be combined with morphological assessment by a trained forensic anthropologist.
- This is an educational/research project, not a validated casework tool.

## Reference

Austin D, King RE. The Biological Profile of Unidentified Human Remains in a Forensic Context. *Acad Forensic Pathol.* 2016;6(3):370-390.

## License

MIT. See [LICENSE](LICENSE).
