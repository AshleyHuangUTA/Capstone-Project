# Target: Baby Care Shelf Space Optimization
**UT Austin MSBA Capstone — April 2026**

**Team:** Ashley Huang · Hiram Gonzalez · Matt Sinclair · Peter Delgado · Sally Hwang · Willington Gahona

---

## Problem

Target uses a single planogram across ~2,000 stores, leaving significant revenue on the table:

- **Space Misalignment** — shelf space is over/under allocated vs. actual demand, causing lost revenue at the store level
- **Silo Management** — categories managed independently, missing cross-item halo and cannibalization effects
- **No Scalability** — one-size-fits-all planograms do not adapt across different store environments

---

## Pipeline

```
Raw Data (Baby Care, $3.6B)
        ↓
LASSO Regression — Feature Selection
        ↓
K-Means Clustering — Store Segmentation
        ↓
Elastic Net — Halo & Cannibalization Modeling
        ↓
Gurobi — Shelf Space Optimization
        ↓
Pearson r — Competitive Signal Analysis
```

---

## Step-by-Step

**1 — LASSO Feature Selection**

Started with 393 features (70 census, 8 competitor distances, 5 store formats, 304 brand/item type indicators, facings, climate). LassoCV with 5-fold cross-validation reduced this to 87 features that actually drive baby care sales share. R² = 0.861.

**2 — K-Means Store Clustering**

Applied K-Means on surviving census and competitor features to segment ~2,000 stores into 6 clusters. Used the elbow method (WCSS + % drop in WCSS) to select k=6. Each cluster gets its own optimization so recommendations adapt to local demographics rather than applying one uniform planogram.

**3 — Elastic Net Cannibalization Engine**

Ran a separate ElasticNetCV model per item type within Cluster 1 stores. Each model estimates how a brand's facing share affects other brands' sales within the same subcategory:

```
New Sales%(Target Brand) = Current Sales%(Target Brand)
                         + β(Lever→Target) × ΔFacing%(Lever Brand)
```

This produces three effect types:
- **Direct** — a brand's own facing share drives its own sales
- **Halo** — growing brand A's space lifts brand B's sales
- **Cannibalization** — growing brand A's space steals from brand B

Betas are validated through a 4-step reliability check: CV table, Beta vs CV scatter, VIF audit, Trust Score.

**4 — Gurobi Shelf Space Optimizer**

Uses Elastic Net betas as a cross-elasticity matrix to maximize projected category revenue subject to:
- Category capacity conserved — total facings per item type unchanged
- ±3% guardrail — no brand shifts more than 3% of its current share
- Facing share cannot drop below 0%

Only categories passing VIF ≤ 10 + CV ≥ 10% enter the optimizer.

**5 — Competitive Signal Analysis**

Pearson correlation between brand-level store sales and distance to 8 competitors (Walmart, Kroger, Costco, Amazon Fresh, CVS, Walgreens, Aldi). Combined with Gurobi output to assign a final action per brand:

| | Gurobi ↑ Space | Gurobi ↓ Space |
|---|---|---|
| **STRONG** (beats rival when nearby) | GROW — confident ✅ | REDUCE — over-spaced |
| **WEAK** (loses to rival when nearby) | CAUTION — flag for review 🟡 | REDUCE — rival compounds loss 🔴 |
| **Neutral** | GROW — standard | REDUCE — standard |

> **Note on synthetic data:** The 120-store synthetic dataset does not produce statistically significant Pearson correlations on its own. The final competition cell includes a hardcoded Shampoos and Conditioners example showing exactly how the output looks — matching the presentation.

---

## Repo Contents

```
├── Target_Capstone_Model_With_Synthetic_Dataset.ipynb   ← Main notebook
├── baby_care_synthetic_v2.csv                           ← Synthetic dataset (120 stores)
└── README.md
```

**To run in Colab:** Upload both files, then update the load cell:
```python
flat = pd.read_csv('/content/baby_care_synthetic_v2.csv')
```

---

## Tech Stack

`Python` · `pandas` · `NumPy` · `scikit-learn` (LassoCV · ElasticNetCV · KMeans · StandardScaler) · `Gurobi` · `scipy.stats` · `seaborn` · `matplotlib` · `plotly` · `statsmodels` · `geopy`

---

## Data Note

All data in this repo is **synthetic** — generated to match the structure and statistical properties of real retail data without exposing any proprietary Target information. Store demographics, sales figures, and competitor distances are all simulated.

---

## Run in Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/AshleyHuangUTA/Capstone-Project/blob/main/Target_Capstone_Model_With_Synthetic_Dataset.ipynb)
