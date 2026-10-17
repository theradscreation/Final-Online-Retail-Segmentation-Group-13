# Online Retail II: Customer Segmentation (RFM + K-Means)

Data Science Group Assignment, **Option C: Identifying Patterns in Global E-Commerce Sales**
Course: DSAI501, Professor Hunter Yan

**Team:** Prem Jasani (BMT28115) · Radhika Gupta (BMT28063) · Rishi (BMT28067) · Lars Godoy (BMT28046)

## Overview
We analyse two years of transactions from a UK online giftware retailer (UCI *Online Retail II*, 1,067,371 raw rows),
clean them, build one row per customer with Recency / Frequency / Monetary (RFM) features, and group customers with
K-Means (k = 4). A market-basket analysis on UK orders then shows which products are bought together.
The aim is a set of customer groups a retention or merchandising team could act on.

## Main results
| Item | Value |
|---|---|
| Transactions after cleaning | 776,577 (from 1,067,371) |
| Identified customers | 5,852 |
| Revenue covered | £17.07M |
| Champions | 20.2% of customers, 73.3% of revenue |
| At-Risk / Need Attention | 24.9% of customers, 16.9% of revenue (about £2.9M historic) |
| Loyal / Promising | 21.3% of customers, 6.1% of revenue |
| Hibernating / Low-Value | 33.6% of customers, 3.7% of revenue |
| Silhouette at k = 4 | 0.365 (moderate separation; k = 2 scores higher but is too coarse to act on) |
| Stability over 10 seeds | mean ARI 0.997 |

These are descriptive patterns in one retailer's 2009-2011 data, not causal claims, and revenue is not profit.

## Repository layout
```
.
├── README.md
├── DATASET_SOURCE.md              dataset source, licence, access date, cleaning steps
├── requirements.txt
├── Segmentation_Analysis.ipynb    full workflow, outputs saved
├── data/
│   └── customer_rfm_segments.csv  per-customer RFM values and segment (notebook output)
├── models/
│   ├── segmentation_pipeline.joblib
│   ├── cluster_personas.json      cluster number -> segment name
│   └── model_metadata.json        features, k, seed, library versions
├── figures/                       the 11 charts used in the analysis (PNG)
└── demo/
    └── demo_customer_test_case.csv  one example customer and the expected segment
```
The raw workbook `online_retail_II.xlsx` is **not** stored here. See `DATASET_SOURCE.md`.

## Reproduce
1. `pip install -r requirements.txt` (Python 3.11+).
2. Download `online_retail_II.xlsx` from the UCI page (link in `DATASET_SOURCE.md`) and save it as `data/online_retail_II.xlsx`.
3. Run `Segmentation_Analysis.ipynb` from the repository root, top to bottom. It rebuilds the figures, the model and the
   customer table. A random seed of 42 is used throughout.

The saved notebook already shows all outputs, so it can be read without re-running.

## Score a new customer
```python
import joblib, json, pandas as pd

model = joblib.load("models/segmentation_pipeline.joblib")
names = json.load(open("models/cluster_personas.json"))

customer = pd.DataFrame([{"Recency": 12, "Frequency": 15, "Monetary": 6000.0}])  # days, orders, GBP
cluster = int(model.predict(customer)[0])
print(cluster, "->", names[str(cluster)])   # 1 -> Champions
```
The model was saved with scikit-learn 1.9.1; use that version to avoid warnings.

## Method in short
Clean with a documented row-count waterfall → compute RFM at a fixed snapshot date (10 Dec 2011) → log-transform and
standardise → pick k with the elbow and silhouette → K-Means → name segments from their profiles → check against a
quartile-score baseline, seed stability, outlier removal and returns → market-basket rules ranked by lift.

## Limitations
About 22.8% of rows have no Customer ID and are left out of the segmentation. There is no cost or margin data, revenue is
about 84% UK, many buyers are wholesalers, and the data covers a single retailer in 2009-2011.

## Data
Chen, D. (2012). *Online Retail II* [Data set]. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D (CC BY 4.0).
