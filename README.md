# Predicting In Vitro Drug Release from PLGA Microparticles Using Machine Learning
**Authors:** Andy Kapoor, Tyler Corazao, & Yiyao Wang

---
## Repository Structure
```
.
├── FormulationAnalyses.ipynb   # Main analysis notebook
├── PLGA_ML_Paper               # Paper associated with the analysis notebook
├── README.md
└── data/                       # PLGA microparticle dataset (see source below)
```
---

## Overview

This project applies machine learning to predict and optimize *in vitro* drug release behavior from PLGA (poly(lactide-co-glycolide)) nanoparticles. Formulation development for drug-polymer systems is traditionally empirical and expensive. This work explores whether data-driven methods can capture the multivariate relationships between formulation descriptors and drug release kinetics, reducing the need for costly trial-and-error experimentation.

---

## Background

PLGA is one of the most widely used biodegradable polymers in drug delivery, but selecting the right formulation parameters for a desired sustained-release profile still requires extensive lab work. Estimates for bringing a new drug to market range from ~$172M to over $1.3B, with >90% of clinical trials failing. Computational approaches that can guide formulation design have significant potential to reduce these costs.

---

## Dataset

- **Source:** Bao et al., *Scientific Data* (2025) — open-access dataset compiled from 321 literature studies
- **Size:** 4,913 rows × 13 columns
- **Drugs covered:** 89 unique drugs
- **Features include:**
  - Polymer molecular weight (kDa)
  - LA/GA ratio
  - Initial drug-to-polymer ratio
  - Particle size (µm)
  - Drug loading capacity (%)
  - Drug encapsulation efficiency (%)
  - Solubility enhancer concentration (%)
  - Time (days) and fractional drug release
  - Molecular descriptors from SMILES strings: molecular weight, TPSA, LogP (via RDKit)

---

## Methods

### Data Preprocessing
- Cubic spline interpolation to evaluate fractional release at shared target timepoints (6, 8, 10 days), enabling single-timepoint regression across heterogeneous literature studies
- StandardScaler normalization applied before all supervised models

### Unsupervised Analysis
- **PCA** applied to understand variance structure; 8 components required for 95% variance retention, 5 for 75%

### Single-Timepoint Regression Models
| Model | Notes |
|---|---|
| Ridge Regression (L2) | Linear baseline |
| Support Vector Regression (RBF kernel) | Nonlinear, kernel-based |
| Random Forest Regressor | Ensemble, nonlinear; 5-fold CV + GridSearchCV tuning |

### Full Kinetic Modeling (Weibull Approach)
- Weibull release model fit to each of the 321 formulation profiles (avg. R² = 0.981)
- ML models trained to predict Weibull parameters (A, τ, β) from formulation descriptors, enabling full curve reconstruction
- Models evaluated: Ridge Regression, Random Forest, 7-layer Neural Network

### SMILES-Based Molecular Featurization
- **Morgan/Circular Fingerprints** (1024-bit): fast, robust, less interpretable
- **Physicochemical Features** (217-feature vectors via DeepChem): slower, more interpretable
- Both compared against base formulation features

---

## Results Summary

| Model | Target Time | R² | MAE |
|---|---|---|---|
| Ridge Regression | t = 6 days | 0.282 | — |
| SVR (RBF) | t = 6 days | 0.244 | — |
| Random Forest (baseline) | t = 6 days | 0.458 | 0.137 |
| Random Forest (tuned) | t = 6 days | 0.424 | 0.140 |
| **Random Forest (baseline)** | **t = 10 days** | **0.522** | **0.158** |

- Weibull parameter prediction from formulation descriptors proved difficult: best RF R² ≈ 0.197 for a single parameter; ridge R² ≤ 0.017; neural network R² = -0.167
- SMILES-based features provided marginal improvement at short timepoints but not at longer timescales

**Top predictive features (Random Forest):** Drug encapsulation efficiency, polymer molecular weight, molecular flexibility index, Balaban J index, hydrophobicity, van der Waals surface area, molecular polarizability

---

## Dependencies
```
pandas
numpy
matplotlib
seaborn
scikit-learn
scipy
rdkit
deepchem
```

Install with:
```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy rdkit deepchem
```

---

**Dataset download:** [Bao et al., Scientific Data (2025)](https://doi.org/10.1038/s41597-025-04621-9)

---

## Limitations & Future Work

- Dataset compiled from hundreds of labs with differing protocols. Unavoidable inter-study noise
- 10 formulation descriptors may be insufficient to fully constrain release kinetics
- Future directions:
  - Incorporate polymer chemistry details and processing metadata
  - Explore physics-informed hybrid ML approaches
  - Evaluate alternative kinetic parameterizations more amenable to supervised learning
  - Transfer learning from larger drug-excipient datasets

---

## References

1. Bao et al., *Sci. Data* **2025**, 12, 364.
2. Sertkaya et al., *JAMA Netw. Open* **2024**, 7(6), e2415445.
3. Wouters et al., *JAMA* **2020**, 323(9), 844–853.
4. Sun et al., *Acta Pharm. Sin. B* **2022**, 12(7), 3049–3062.
5. Sadybekov & Katritch, *Nature* **2023**, 616, 673–685.
6. Sabe et al., *Eur. J. Med. Chem.* **2021**, 224, 113705.
7. Antipas et al., *Sci. Rep.* **2024**, 14, 15106.
8. Dignon & Dill, *J. Chem. Theory Comput.* **2024**, 20(3), 1479–1488.
9. Panigrahi et al., *SN Appl. Sci.* **2021**, 3, 638.
10. Yamada et al., *ACS Cent. Sci.* **2019**, 5(10), 1717–1730.
