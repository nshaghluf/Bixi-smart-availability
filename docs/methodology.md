# Methodology and reproducibility checklist

## Problem

For a particular station and journey time, anticipate whether a return dock or rental bike will be available when the rider arrives. The approach separates **historical flow forecasting** (measurable using trip records) from **physical station availability** (requires time-stamped station-status observations).

## BIXI-only inputs

1. **Historical trips:** start/end station names, station coordinates, start/end timestamps in milliseconds.
2. **Station information (GBFS):** station identifier, display name, coordinates, capacity.
3. **Station status (GBFS):** available bikes/docks, renting/returning flags, last-reported timestamps.

No external weather or event data were used in the reported models.

## Pipeline

1. Convert trip milliseconds into 15-minute bins; preserve UTC for alignment and derive Montréal local-time/calendar features for modeling.
2. Normalize station names, then resolve historical renamed stations conservatively by unique short-distance geographic matches. 
3. Aggregate starts as departures and ends as arrivals, retaining coverage information. Build the complete station × time grid only across validated active station-days; fill truly observed no-trip intervals with zero.
4. Fit separate CatBoost count regressors for arrivals and departures. Reported features: `station`, `year`, `month`, `day_of_week`, `hour`, `time_of_day_slot`. The station feature is categorical.
5. Train using history through 2024 and evaluate on held-out 2025 and 2026. For the year-to-year comparison, restrict to common matched stations and common station-calendar-days.
6. Measure MAE and prediction bias against a predict-zero count baseline. Measure net-flow MAE separately because errors can compound.
7. For short-horizon availability, start with a validated GBFS current inventory, account for predicted demand during the trip, and estimate uncertainty of inventory changes. Check both the number of physical docks/bikes and the station's ability to rent/accept returns.

## Availability probability modeling

One exploratory assumption is to model arrival and departure counts as conditionally independent Poisson random variables over the horizon. Their difference then follows a Skellam distribution. This is a convenient baseline but ignores correlated bursts, station rebalancing and capacity constraints. Predictions should be clipped to feasible inventory bounds and compared against persistent-status forecasts and better empirical models.

## Experiments still required for publication-quality claims

- Save deterministic train/test date ranges, dataset versions, feature definitions, model hyperparameters and evaluation code.
- Audit station identity, relocations, seasonal activity, DST transitions, missing end-station trip fields and station availability flags.
- Use multiple weeks/months of real GBFS data, including varied demand regimes and rare full/empty periods.
- Compare against predict-zero, time-of-week historical averages, and current-status persistence.
- Evaluate error by station, traffic decile, hour, weekday/weekend, inventory bucket and forecast horizon.
- Report classification precision/recall for unavailable docks/bikes, Brier score, reliability curves, confidence intervals and performance when only 0–2 slots remain.
- Make experiment notebooks/scripts runnable before advertising the repository as a reproducible benchmark.

## Ethics and attribution

Source data: BIXI Montréal public trip records and GBFS feeds. Independent project
