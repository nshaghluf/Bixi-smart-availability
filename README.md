# BIXI Smart Availability

**Forecasting bike-share station demand and exploring availability at arrival.**

Arriving at a full BIXI station is inconvenient—especially when the next station is far from your destination. This independent applied-ML project investigates a practical question: **Will there be at least one free dock (or one bike) when I arrive in 15–30 minutes?**

The current research prototype combines historical trip-demand models with live BIXI station status. 

## At a glance

- **Data:** BIXI Montréal trip records; 15-minute station-level arrival/departure counts; BIXI GBFS station-status feed for short-horizon availability experiments.
- **Model:** CatBoost count regression for expected arrivals and departures, trained on historical records through 2024.
- **Out-of-year testing:** independently evaluated on 2025 and 2026 records, without retraining on either test year.
- **Matched comparison:** 708 stations, 91,846 comparable station-calendar-days and 8,817,216 station-intervals per test year.
- **Best verified outcome:** 2025 arrival MAE improved **10.26%** versus a predict-zero baseline; departure MAE improved **9.85%**. On 2026 data, improvements were **7.93%** and **7.44%**.

## Historical-demand results

Lower MAE is better. Each row evaluates the same matched set of station/calendar-day records for its year.

| Test year | Forecast | CatBoost MAE | Predict-zero MAE | Error reduction |
|---|---|---:|---:|---:|
| 2025 | Arrivals / 15 min | 0.6541 | 0.7289 | **10.26%** |
| 2025 | Departures / 15 min | 0.6599 | 0.7320 | **9.85%** |
| 2026 | Arrivals / 15 min | 0.6499 | 0.7059 | **7.93%** |
| 2026 | Departures / 15 min | 0.6553 | 0.7080 | **7.44%** |

These are useful gains over a simple benchmark, but **they do not mean availability is predicted with 90%+ accuracy**. Trip-count forecasting and binary bike/dock availability are different tasks.

## How the prototype works

1. Prepare official BIXI trip records as station × 15-minute demand counts.
2. Resolve historical station-name changes with conservative one-to-one matching.
3. Fit separate CatBoost models for arrivals and departures using station and calendar/time features.
4. Read current station inventory and operating flags from the BIXI GBFS feed.
5. Estimate plausible inventory changes over a journey and turn them into exploratory bike/dock availability probabilities.

## Short-horizon availability: early evidence

An initial **brief live observation on October 26, 2026** produced 2,412 matched 15-minute examples. The preliminary probabilistic model improved the **Brier score** (lower is better) over a persistence baseline:

| Availability target | Persistence Brier | Prototype Brier | Relative reduction |
|---|---:|---:|---:|
| At least one bike | 0.0211 | 0.0165 | 21.8% |
| At least one dock | 0.0149 | 0.0109 | 26.8% |

Overall binary classification accuracy did **not** improve versus persistence on this test. These data cover a short observation period and are insufficient to claim generalizable arrival-time availability performance. Further multi-day testing and calibration are planned.

## Tools

Python · Pandas · DuckDB · CatBoost · Parquet · Jupyter · GBFS · Git

## Project documentation

- [Evaluation tables and caveats](docs/results.md)
- [Data processing and methodology](docs/methodology.md)
- [Portfolio profile template](docs/github-profile.md)

## Reproducing the work

The documented metrics summarize completed local experiments. **The notebooks, model weights, and training scripts need to be added from the original local BIXI workspace before the repository can be called reproducible.** Raw trip data and large processed datasets are deliberately excluded from version control. See the methodology document for experiment definitions and the data checks required to reproduce the tables.

## Acknowledgment

This is an **independent personal portfolio project**, not affiliated with or endorsed by BIXI. The project uses BIXI's published trip data and public GBFS information; trademarks and original data remain with their respective owners.
