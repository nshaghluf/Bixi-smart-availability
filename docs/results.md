# Model evaluation: 2025–2026

## Scope of the comparable out-of-year test

The evaluation compares historical demand predictions against observed trip counts for **708 stations** on **91,846 common station-calendar-days**, using the same 15-minute station/time slots in each year (**8,817,216 intervals per year**). The independent test years are **2025 and 2026**; 2026 trip records run through September 26, not the full year. Training excludes the test years.

The baseline always predicts **zero arrivals** or **zero departures** in each 15-minute interval. MAE is the mean absolute difference between predicted and observed counts; improvements are relative to this baseline.

| Year | Target | Intervals | CatBoost MAE | Zero-baseline MAE | Improvement |
|---|---|---:|---:|---:|---:|
| 2025 | Arrivals | 8,817,216 | 0.6541 | 0.7289 | **10.2602%** |
| 2025 | Departures | 8,817,216 | 0.6599 | 0.7320 | **9.8456%** |
| 2026 | Arrivals | 8,817,216 | 0.6499 | 0.7059 | **7.9254%** |
| 2026 | Departures | 8,817,216 | 0.6553 | 0.7080 | **7.4415%** |

## Prediction bias

| Year | Target | Actual average count | Predicted average count | Bias |
|---|---|---:|---:|---:|
| 2025 | Arrivals | 0.7289 | 0.6935 | -0.0355 |
| 2025 | Departures | 0.7320 | 0.7007 | -0.0313 |
| 2026 | Arrivals | 0.7059 | 0.6931 | -0.0128 |
| 2026 | Departures | 0.7080 | 0.7003 | -0.0077 |

**Interpretation:** Predictions slightly underestimate mean demand, more noticeably in 2025. The model retains positive arrival/departure MAE gains on later-year data but needs stronger calibration and hard-case analysis.

## A negative result worth retaining

The **net-flow** estimate (arrivals minus departures) underperforms the zero-net-flow baseline in the same evaluation:

| Year | Net-flow model MAE | Zero-net-flow MAE | Relative change |
|---|---:|---:|---:|
| 2025 | 0.7311 | 0.7064 | **3.51% worse** |
| 2026 | 0.7207 | 0.6952 | **3.68% worse** |

This matters because dock availability depends on net inventory movement, not simply accurate separate arrival and departure averages. The current demand model is **a promising component, not an end-to-end validated solution**.

## Initial 15-minute availability experiment

An approximately 35-minute collection window on **October 26, 2026**, contained eight GBFS snapshots. After matching and validity checks, the 15-minute test had **2,412 examples**. A persistence baseline assumed that the current availability state would remain unchanged until arrival.

| Target | Persistence accuracy | Prototype accuracy | Persistence Brier | Prototype Brier |
|---|---:|---:|---:|---:|
| Bike available | 97.8856% | 97.8856% | 0.0211 | **0.0165** |
| Dock available | 98.5075% | 98.5075% | 0.0149 | **0.0109** |

The smaller Brier scores suggest more informative probabilities on this narrow sample, despite unchanged binary accuracy. Availability was extremely common during collection, so overall accuracy is not a strong indicator of safety at nearly empty/full stations.

For **139 observations starting with only one or two docks**, prototype accuracy remained **84.89%**, while Brier improved from **0.1511 to 0.1144**. For **315 observations starting with one or two bikes**, Brier deteriorated from **0.0540 to 0.0609**. These slices are too small for broad conclusions, especially for completely full/empty stations.

## Data and evaluation cautions

- A 2025–2026 station match does not itself confirm unchanged station capacity or lack of relocations. Station matching must be auditable.
- Data include many zero-count intervals; compare with stronger time-of-week/seasonality/persistence baselines before claiming practical superiority.
- Only count intervals during verified station operation; unknown inactive periods should not automatically become zero-demand labels.
- Forecast evaluation should separate temporal and geographic effects and report error for high-traffic locations and evening/weekend residential stations.
- An availability prediction also depends on **current inventory, disabled docks/bikes, operational flags, and unobserved rebalancing**; trip-count data alone cannot establish true bike/dock availability.
- Short-horizon availability experiments need substantially more days, horizons, and rare-event examples plus calibration curves, reliability diagrams, and risk-weighted metrics.

## What a user-facing metric should mean

For an arrival time `t + h`, the main app outputs should be interpreted as probabilities:

- **P(at least one dock available at arrival)** for returns.
- **P(at least one bike available at arrival)** for departures.

Until robust validation, the prototype should show uncertainty and should not promise that a dock or bike will be available.
