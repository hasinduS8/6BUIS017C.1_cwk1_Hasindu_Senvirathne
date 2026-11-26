# Market Analysis and Stock Segmentation Using RFM

**Module:** 6BUIS017W CRM & CM with Business Intelligence – Individual Coursework (CWK1)
**Author:** Hasindu Senvirathne
**Repository:** https://github.com/hasinduS8/6BUIS017C.1_cwk1_Hasindu_Senvirathne

## Overview
This project analyses S&P 500 shares from 1 January 2021 to 1 January 2025 (end date exclusive). It segments the shares in two separate ways:
- **Risk and return:** mean weekly return and weekly beta against ^GSPC.
- **Trading activity:** adapted RFM (Recency, Frequency, Monetary) based on high-volume trading days.

Each feature set is clustered with Ward agglomerative clustering and K-Means. A sensitivity analysis then removes Monetary from the RFM solution.

## Repository contents
| File | Description |
|---|---|
| `Hasindu_CWK1_SP500_RFM.ipynb` | Complete Google Colab notebook (Tasks 1–3) |
| `cwk1_data.zip` | Data snapshot: constituent list, daily share prices, ^GSPC prices and `snapshot_metadata.json` |
| `cwk1_all_outputs.zip` | Feature table, cluster assignments, validation logs, figures and `run_metadata.json` |
| `requirements.txt` | Package versions used |

## Reproducing the results
1. Open the notebook in Google Colab.
2. Select **Runtime → Run all**.

**Environment:** Python 3.13 · pandas 2.2.3 · numpy 2.4.6 · yfinance 0.2.66 · scipy 1.16.3 · scikit-learn 1.6.1 · matplotlib 3.10.0 · seaborn 0.13.2

## Data and key results
| Item | Value |
|---|---|
| Tickers requested | 503 (Wikipedia) |
| Shares included after validation | 486 (17 excluded: unequal history 10, no data 5, volume errors 2) |
| Aligned weekly returns per share | 207 (Friday-ending weeks) |
| RFM observation date and window | 2024-12-31; latest 60 sessions |
| High-volume day | Volume > 2 × mean of the preceding 20 sessions |
| Selected clusters | k = 2 for all four models |
| Ward vs K-Means agreement (ARI) | Risk & return 0.598 · RFM 0.700 |
| RFM vs R+F-only Ward solution (ARI) | 0.874 |
| Risk-return vs RFM agreement (ARI) | 0.029 (Ward) |

## Limitations
- **Survivorship bias:** the current constituent list is applied to historical data.
- **Price basis:** ^GSPC is a price index, while share returns use dividend-adjusted prices.
- **Liquidity proxy:** Monetary approximates traded value only.
- **Descriptive only:** the clusters describe past behaviour and do not predict future returns or reduced portfolio risk.
