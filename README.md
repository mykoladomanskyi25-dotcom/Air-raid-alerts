# Ukrainian Air Raid Sirens — Time-Series Analysis

EDA and baseline forecasting of oblast-level air raid alerts across Ukraine (2022–2025).

## Dataset

Source: [Vadimkin/ukrainian-air-raid-sirens-dataset](https://github.com/Vadimkin/ukrainian-air-raid-sirens-dataset)

Columns used: `oblast`, `level`, `started_at`, `finished_at`

**`level` column:** records exist at three administrative levels simultaneously (oblast, raion, hromada). Grouping without filtering `level == 'oblast'` inflates daily counts by ~2.1×.

## Setup

```bash
pip install -r requirements.txt
```

Download the CSV from the dataset link above and place it at `data/ukraine_alarms.csv`. Notebook `02_cleaning.ipynb` will produce `data/daily_counts.csv` used by subsequent notebooks.

## Notebooks

| # | Notebook | What it does |
|---|----------|-------------|
| 1 | `01_eda.ipynb` | Raw data overview: per-oblast alarm counts, hour-of-day distribution, timezone impact |
| 2 | `02_cleaning.ipynb` | Timezone conversion (UTC → Kyiv), level filtering, duration outlier removal, builds `daily_counts.csv` |
| 3 | `03_forecast.ipynb` | Three pure-pandas baselines (naive persistence, 7-day rolling mean, seasonal naive); MAE/RMSE on a 60-day test set (Jul–Aug 2025) |
| 4 | `04_reflection.ipynb` | Findings synthesis, data limitations, why conflict-driven series resist forecasting |

Run in order: `02` depends on `01`'s exploration; `03` and `04` read `daily_counts.csv` from `02`.

## Key Findings

- **Forecast baseline:** best model is 7-day rolling mean, MAE = **14.28** alarms/day on the Jul–Aug 2025 test set (mean = 46.6/day, so error ≈ 31% of the mean). It wins not by finding signal but by smoothing noise — it still systematically misses mass-strike spikes.
- **Geographic inequality:** Donetsk, Zaporizhzhia, and Kharkiv oblasts accumulate 3–5× more alarm events than western oblasts. The pattern is structural and stable across the full observation window.
- **Attack timing:** alarm starts peak at **12:00 Kyiv time** (UTC peak 10:00, shifted +2 h after timezone conversion). The distribution by hour is a concrete demonstration of why UTC grouping produces misleading results.
- **Non-stationarity:** the daily count shows at least two visible regime shifts. A model trained in one period does not transfer cleanly to the next.
- **Data cutoff:** all analysis is clipped at `2025-08-31`. Data after approximately November 2025 collapses to near-zero due to a collection artifact, not a real drop in activity.

## Limitations

- **Collection artifact after ~Nov 2025:** the dataset's daily counts fall to near-zero from approximately November 2025 onward. This is a data-collection failure, not a real change in alarm frequency. Any analysis or model trained on post-cutoff data will produce misleading results.
- **Oblast granularity:** one alarm record covers an entire oblast (~26,000 km²). The dataset does not indicate which city or district was under actual threat.
- **Duration threshold:** duration analysis excludes alarms longer than 6 hours as likely stale records (alarm ended but `finished_at` was never updated). This removes a small number of edge cases; it does not affect alarm counts.
- **Forecasting ceiling:** the primary drivers of daily alarm counts — missile inventory, weather, strategic priorities — are not in this dataset. A univariate autoregressive model is fitting correlated noise. The MAE of ~31% of the daily mean is close to an irreducible floor for this class of model.

## License

Analysis code: MIT. Dataset: see original repository license.
