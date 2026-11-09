# AI-IS-SSIC-Labs--Med-amine-Aminou
# Phishing URL Detection with Logistic Regression + SHAP

Lab project for AI in Cybersecurity. The goal is to detect phishing websites using Logistic Regression and to understand *why* the model makes its decisions (like a SOC analyst would need).

## Dataset

**PhiUSIIL Phishing URL Dataset** (from Kaggle)
- 235,795 URLs, 56 columns
- No missing values, no duplicates
- About 57% legitimate and 43% phishing, so the classes are fairly balanced

Some features: `URLLength`, `IsHTTPS`, `LetterRatioInURL`, `NoOfJS`, `HasSocialNet`, `HasCopyrightInfo`, etc.

## What I did

1. **Data check**: shape, missing values, duplicates, class balance
2. **Target fix**: I created `IsPhishing = 1 - label` so that 1 = phishing and 0 = legitimate (in the dataset `label=1` means legitimate)
3. **Visualisation**: class balance, binary indicators per class, KDE plots for numeric features
4. **Feature selection**: removed text/ID columns (`URL`, `Domain`, `Title`, ...) and `URLSimilarityIndex` -> 49 features left. Looked at correlations with the target
5. **Train/test split**: 75% / 25%, stratified, then `StandardScaler` fitted **only on train** (to avoid data leakage)
6. **Trained 5 Logistic Regression models** and compared them:

| Model | Idea |
|---|---|
| M1 | No regularization |
| M2 | L2 strong (C=0.001) |
| M3 | L2 standard (C=1.0) |
| M4 | L1 Lasso (C=0.01) |
| M5 | L2 with `class_weight='balanced'` |

7. **L1 regularization path**: tested different C values to see how many features stay active
8. **Interpretation**: log-odds weights and odds ratios
9. **SHAP**: global importance (bar + beeswarm) and local explanations (waterfall plots)
10. **Threshold analysis**: compared thresholds 0.20, 0.50 and 0.85

## Results (threshold = 0.50)

| Model | Precision | Recall | F1 | False Alarms | Missed Phish |
|---|---|---|---|---|---|
| M1 | 0.9996 | 0.9990 | 0.9993 | 11 | 25 |
| M2 | 0.9985 | 0.9958 | 0.9971 | 39 | 105 |
| M3 | 0.9996 | 0.9989 | 0.9992 | 10 | 28 |
| M4 (L1) | 0.9993 | 0.9971 | 0.9982 | 18 | 72 |
| M5 | 0.9995 | 0.9990 | 0.9992 | 13 | 25 |

## Main findings

- All models are very accurate (F1 > 0.997). The dataset is quite easy for a linear model.
- **L1 (Lasso)** removed 24 of the 49 features at C=0.01 and still got F1 = 0.998, so it is a good choice when we want a simpler model.
- Too much regularization (M2) hurts performance a bit.
- Features that push toward **phishing**: `LetterRatioInURL`, `DegitRatioInURL`, `SpacialCharRatioInURL`, `LargestLineLength`.
- Features that push toward **legitimate**: `IsHTTPS`, `NoOfJS`, `NoOfSelfRef`, `NoOfImage`, `HasSocialNet`.
- SHAP waterfall plots let us explain a single URL (true positive, true negative, false positive) in a way an analyst can understand.
- Changing the threshold changes the trade-off: low threshold = catch more phishing (threat hunting), high threshold = fewer false alarms (automatic blocking).

## Notes

- `IsHTTPS` is 100% for legitimate sites in this dataset, which is probably why the model relies on it so much. In real life this might not be so reliable.
- The L1 loop with `liblinear` was slow (around 30 min) and gave some convergence warnings.

## How to run

```bash
pip install numpy pandas matplotlib seaborn scikit-learn shap
```

1. Download the dataset from Kaggle (`PhiUSIIL_Phishing_URL_Dataset.csv`)
2. Change the path in the first cell of the notebook
3. Open `notebook-lab-ai.ipynb` and run all cells (Python 3, seed = 42)

## Files

- `notebook-lab-ai.ipynb` - the full lab notebook
- `README.md` - this file

## Tools

Python, pandas, NumPy, matplotlib, seaborn, scikit-learn, SHAP
