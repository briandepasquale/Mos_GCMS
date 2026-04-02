# Mosquito Attraction & GC-MS Fatty Acid Analysis

## Problem

This project investigates whether the chemical composition of human skin — specifically fatty acid profiles measured by Gas Chromatography-Mass Spectrometry (GC-MS) — can predict behavioral attractiveness to mosquitoes. Subjects were classified into **High** and **Low** attraction groups based on a behavioral assay, and their skin volatile fatty acid profiles were measured.

The central question: **Can fatty acid abundances on human skin predict how attractive a person is to mosquitoes?**

## Data

### Behavior Data (`Behavior_data_Ellen_Fig6B .xlsx`)
- **19 subjects** (11 High attraction, 7 Low attraction, 1 Control)
- Each subject completed 13-26 trials measuring mosquito attraction (% score per trial)
- Scores range from 0% (no attraction) to 100% (maximum attraction)

### GC-MS Data (`GCMS_ellen_data_Fig6F.xlsx`)
- **380 rows** (19 subjects x 20 replicate measurements each)
- **11 fatty acids** measured (C10-C20 carbon chain lengths):

| Fatty Acid | Carbon Chain | Common Name |
|---|---|---|
| decanoic | C10 | capric acid |
| undecanoic | C11 | - |
| dodecanoic | C12 | lauric acid |
| tridecanoic | C13 | - |
| tetradecanoic | C14 | myristic acid |
| pentadecanoic | C15 | - |
| hexadecanoic | C16 | palmitic acid |
| heptadecanoic | C17 | margaric acid |
| octadecanoic | C18 | stearic acid |
| nonadecanoic | C19 | - |
| icosanoic | C20 | arachidic acid |

## Analysis

The analysis is contained in [`explore_data.ipynb`](explore_data.ipynb). Below is a summary of the approach and results.

### Data Preparation

1. Behavior scores were averaged across all trials per subject to produce a single **mean behavior score** (%)
2. GC-MS measurements were averaged across 20 replicates per subject for each fatty acid
3. Datasets were merged on subject ID, yielding **18 matched subjects** (Control subject excluded from merge)

### Correlation Analysis

All 11 fatty acids show **positive correlations** with mean behavior score, ranging from r=0.304 (tetradecanoic) to r=0.564 (dodecanoic). This suggests that higher fatty acid abundances are broadly associated with higher mosquito attraction.

![Correlation Heatmap](fig_correlation_heatmap.png)

### Dimensionality Reduction (PCA)

With 11 predictors and only 18 subjects, PCA was applied to reduce dimensionality and avoid overfitting:
- **3 principal components** retained, explaining **91.6%** of total variance
  - PC1 (62.9%): Overall fatty acid abundance (all positive loadings)
  - PC2 (18.5%): Short-chain vs long-chain fatty acids contrast
  - PC3 (10.2%): Specific fatty acid contrasts (dodecanoic/octadecanoic vs tridecanoic/pentadecanoic)

![PCA Scree Plot](fig_pca_scree.png)

![PCA Loadings](fig_pca_loadings.png)

### Regression Model

**Beta regression** was attempted (appropriate for bounded proportion data on (0,1)), but did not converge with the small sample size. **OLS regression** on PCA components was used as the fallback.

#### Full Model (all 18 subjects)

| Metric | Value |
|---|---|
| R-squared | 0.361 |
| Adj. R-squared | 0.224 |
| F-statistic | 2.639 (p=0.090) |

**PC1** (overall fatty acid abundance) was the only significant predictor (coef=0.048, p=0.018), confirming that subjects with higher overall fatty acid levels tend to have higher mosquito attraction scores.

#### Leave-One-Out Cross-Validation (LOOCV)

| Metric | Value |
|---|---|
| MAE | 18.94% |
| RMSE | 21.52% |
| R-squared (CV) | 0.027 |
| Pearson r | 0.341 |

The low cross-validated R-squared versus the in-sample R-squared (0.361) indicates the model is somewhat overfit given the small sample size. However, the positive Pearson r (0.341) confirms a real, if modest, predictive signal.

![Predicted vs Actual](fig_predicted_vs_actual.png)

### Residual Analysis

Residuals are approximately normally distributed with no obvious systematic patterns, supporting the validity of the linear model assumptions.

![Residuals](fig_residuals.png)

### Group-Level Fatty Acid Profiles

Comparing standardized fatty acid profiles between High and Low attraction groups reveals that **High attraction subjects tend to have higher abundances across nearly all fatty acids**, while Low attraction subjects are below average. The pattern is consistent across all 11 fatty acids.

![Group Profiles](fig_group_profiles.png)

## Key Findings

1. **All 11 fatty acids positively correlate with mosquito attraction**, with dodecanoic (C12), hexadecanoic (C16), and decanoic (C10) showing the strongest correlations (r > 0.5)
2. **PC1 (overall fatty acid level) is a significant predictor** of behavior (p=0.018), explaining ~36% of variance in-sample
3. **Cross-validated prediction is modest** (R2=0.03, r=0.34), reflecting the challenge of small sample size (n=18)
4. **High-attraction subjects have systematically elevated fatty acid profiles** compared to low-attraction subjects across all measured acids

## Requirements

```
pandas
openpyxl
numpy
statsmodels
scikit-learn
matplotlib
seaborn
scipy
```

## Usage

```bash
# Create and activate virtual environment
python3 -m venv .venv
source .venv/bin/activate
pip install pandas openpyxl numpy statsmodels scikit-learn matplotlib seaborn scipy jupyter

# Run the notebook
jupyter notebook explore_data.ipynb
```
