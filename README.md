# Market Analysis and Stock Segmentation Using RFM

**Module:** 6BUIS017W CRM & CM with Business Intelligence – Individual Coursework (CWK1)
**Author:** Hasindu Senvirathne
**Repository:** https://github.com/hasinduS8/6BUIS017C.1_cwk1_Hasindu_Senvirathne

This repository contains the code, data snapshots and outputs for a business-intelligence analysis of S&P 500 shares over **1 January 2021 – 1 January 2025 (end date exclusive)**. Shares are segmented in two separate ways:

1. **Risk and return:** mean weekly return and weekly beta against the S&P 500 index (^GSPC).
2. **Trading activity:** adapted RFM measures (Recency, Frequency, Monetary) built from high-volume trading days.

Each feature set is clustered with **Ward agglomerative clustering** and **K-Means**. A sensitivity analysis then removes Monetary from the RFM solution. RFM is adapted here to describe share trading activity. It does not measure company value, customer loyalty or investment profitability.

---

## Repository contents

| File | Description |
|---|---|
| `Hasindu_CWK1_SP500_RFM.ipynb` | Complete Google Colab notebook covering Tasks 1–3 |
| `cwk1_data.zip` | Preserved data snapshot used for the reported results (see below) |
| `cwk1_all_outputs.zip` | All exported tables (32 CSV files), figures (12 PNG files) and run metadata |
| `requirements.txt` | Exact package versions used for the reported run |
| `README.md` | This file |

### Data snapshot (`cwk1_data.zip`)
| File | Contents |
|---|---|
| `data/sp500_constituents_snapshot.csv` | S&P 500 constituent list from Wikipedia (symbol, security, GICS sector) |
| `data/market_prices.csv.gz` | Daily Open, High, Low, Close, Adj Close, Volume, Dividends and Stock Splits for all tickers |
| `data/benchmark_prices.csv.gz` | Daily ^GSPC benchmark data |
| `data/snapshot_metadata.json` | Sources, retrieval timestamp, study window, yfinance version, download settings |

### Main outputs (`cwk1_all_outputs.zip` → `outputs/`)
| Area | Files |
|---|---|
| Validation (Task 1.2) | `validation_log.csv`, `inclusion_log.csv`, `exclusion_log.csv`, `exclusion_summary.csv`, `corporate_action_review.csv`, `weekly_date_log.csv`, `validation_rules_summary.csv` |
| Features (Task 1.3–1.5) | `feature_table.csv` (one row per included share), `risk_return_features.csv`, `rfm_features.csv`, `feature_summary.csv`, `risk_return_summary.csv`, `feature_exclusion_log.csv`, `rfm_worked_example.png` |
| Clustering (Task 2.1–2.4) | `clustering_features.csv`, `clustering_evaluation.csv`, `candidate_cluster_profiles.csv`, `selected_cluster_counts.csv`, `final_cluster_profiles.csv`, `algorithm_comparison.csv`, `feature_table_with_clusters.csv` (cluster assignments for all four baseline solutions) |
| Sensitivity (Task 2.5) | `rfm_correlation.csv`, `monetary_removal_membership.csv`, `monetary_removal_profiles.csv`, `monetary_removal_summary.csv` |
| Business review (Task 3.2) | `sector_composition.csv`, `sector_counts.csv`, `overall_sector_distribution.csv`, `cross_segmentation.csv`, `cross_segmentation_ari.csv`, `return_dependency_summary.csv`, `price_adjustment_summary.csv` |
| Figures | `ward_risk_return.png`, `ward_rfm.png` (dendrograms), `silhouette_comparison.png`, `kmeans_elbow_plots.png`, `risk_return_clusters.png`, `rfm_clusters_pca.png`, `rfm_correlation_heatmap.png`, `monetary_transformation.png`, `standardised_feature_boxplots.png`, `monetary_removal_comparison.png`, `sector_composition.png`, `rfm_worked_example.png` |
| Reproducibility | `run_metadata.json` (calculation version, sources, observation date, parameters, package versions) |

---

## How to reproduce the results

1. Open `Hasindu_CWK1_SP500_RFM.ipynb` in Google Colab.
2. To reproduce the **exact reported results**, set the following in the configuration cell:
   ```python
   USE_CACHED_SNAPSHOT = True
   ```
   The notebook then loads `cwk1_data.zip` from this repository (`CACHE_ZIP_URL`) instead of downloading live data. Yahoo Finance revises adjusted prices over time, so the snapshot is the version of record.
3. Choose **Runtime → Run all**.
4. With `USE_CACHED_SNAPSHOT = False`, the notebook downloads fresh data from Wikipedia and Yahoo Finance. The results may then differ slightly from the report.

### Environment (reported run)
Python 3.13.18 · pandas 2.2.3 · numpy 2.4.6 · yfinance 0.2.66 · matplotlib 3.10.0 · seaborn 0.13.2 · scipy 1.16.3 · scikit-learn 1.6.1 · lxml 6.1.3

To install locally:
```bash
pip install -r requirements.txt
```

---

## Data and method summary

### Data acquisition (Task 1.1)
* **Tickers:** the S&P 500 constituent list from Wikipedia was extracted with `pandas.read_html` on **2026-11-26 16:17:06 UTC** (503 tickers). `.` is replaced by `-` for Yahoo symbols.
* **Prices:** daily data for the shares and ^GSPC were downloaded with **yfinance 0.2.66** using `auto_adjust=False` and `actions=True`, so Close, Adj Close, Volume, dividends and splits are all available.
* **Study window:** 2021-01-01 to 2025-01-01 (exclusive), giving **1,005 benchmark trading sessions** (2021-01-04 to 2024-12-31).

### Validation (Task 1.2)
Every share is aligned to the benchmark trading calendar and checked for duplicate dates, missing values, non-positive prices, volume errors, unequal histories and corporate actions. **No missing prices are filled.**

| Outcome | Shares |
|---|---|
| Included | **486** |
| Unequal trading history (listed or spun off during the period) | 10 |
| No data returned | 5 |
| Volume errors | 2 |

53 retained shares had stock splits and none showed signs of unadjusted splits. Seven extreme price movements were reviewed manually (see `corporate_action_review.csv`).

### Weekly return and weekly beta (Task 1.3)
* Adjusted closing prices are resampled to **Friday-ending weeks** (`W-FRI`, last available price). The incomplete final week (ending 2025-01-03) is excluded and holiday-shortened weeks are retained.
* **207 aligned weekly returns** per share (2021-01-15 to 2024-12-27), on identical week labels for shares and benchmark.
* Weekly return = adjusted close at the end of this week / adjusted close at the end of the preceding week − 1. The return feature is the arithmetic mean.
* **Weekly beta** = Cov(share, benchmark weekly returns) / Var(benchmark weekly returns), over the full period.

### Adapted RFM (Task 1.4)
* **Observation date:** 2024-12-31 (final common completed trading date). **RFM window:** the latest 60 sessions, 2024-10-07 to 2024-12-31.
* **High-volume day:** volume > 2 × the mean volume of the preceding 20 sessions (current day excluded), calculated separately for every share and day.
* **Recency** = calendar days from the latest high-volume day to the observation date (missing if there is no event, never zero).
* **Frequency** = number of high-volume days in the latest 60 sessions.
* **Monetary** = Σ (Close × Volume) over the latest 60 sessions, in USD. This is approximate traded value, not market capitalisation.
* **Worked example (AAPL):** events on 2024-11-25 and 2024-12-20 → R = 11, F = 2, M ≈ $623.3 bn, verified by an independent manual calculation.

### Clustering (Task 2)
* **Common share set:** 486 shares are used for all four baseline solutions.
* Monetary is transformed with `log1p`. Each feature matrix is standardised separately with `StandardScaler`.
* **Ward:** `linkage(method="ward", metric="euclidean")`, with one dendrogram per matrix.
* **K-Means:** `random_state=42`, `n_init=50`.
* k = 2–8 were compared using silhouette scores, cluster sizes and profiles (plus inertia for K-Means). **k = 2 was selected for all four models.**

| Model | k | Silhouette | Cluster sizes |
|---|---|---|---|
| Ward – risk & return | 2 | 0.340 | 394 / 92 |
| K-Means – risk & return | 2 | 0.387 | 343 / 143 |
| Ward – RFM | 2 | 0.368 | 404 / 82 |
| K-Means – RFM | 2 | 0.423 | 418 / 68 |

Agreement between Ward and K-Means on the same shares (Adjusted Rand Index): risk & return **0.598**, RFM **0.700**.

### RFM sensitivity (Task 2.5)
* Pearson correlations: R–F −0.346, R–log1p(M) 0.152, F–log1p(M) 0.013.
* Repeating Ward RFM with **R and F only** (re-standardised, same 486 shares, k = 2) gives **ARI = 0.874** against the full RFM solution.

### Business review (Task 3)
* Agreement between the risk-return and RFM segmentations is close to zero: ARI = 0.029 (Ward) and 0.005 (K-Means). The two feature sets therefore describe different aspects of the shares.
* The mean pairwise weekly return correlation across shares is 0.30. Within-GICS-sector correlation (0.41) is higher than within-cluster correlation (≈0.30–0.31), so cluster membership alone does not establish diversification.

---

## Limitations
* **Survivorship bias:** the current constituent list is applied to historical data, so shares that left the index during 2021–2024 are missing.
* **Price basis:** shares use dividend-adjusted prices, while ^GSPC is a price index (its Adj Close equals Close).
* **Liquidity proxy:** Monetary approximates traded value only. Bid-ask spread and price impact are not measured.
* **Descriptive only:** the clusters describe historical behaviour. They do not predict future returns or show that portfolio risk would be reduced.

## Academic integrity
This repository supports an individual coursework submission. GenAI assistance was used in line with the module lecturer's guidance; see the GenAI declaration in the report.
