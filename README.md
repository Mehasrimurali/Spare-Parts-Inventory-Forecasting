# New X Services: Service Data Analysis and Spare-Parts Demand Forecasting

Weekly demand forecasting for stocked spare parts and fluids at a two-wheeler service centre, so the centre can move towards just-in-time (JIT) stocking.

| | |
|---|---|
| **Client** | NewX Services |
| **Domain** | Automotive service and spare-parts inventory |
| **Target** | Weekly number of billed stocked-item lines (parts and fluids) |
| **Horizon** | 1 week ahead (rolling one-step forecasts) |
| **Metrics** | MAE, RMSE, WAPE (primary), sMAPE |
| **Data** | 28,482 invoice lines, 30 May 2017 to 6 Jan 2019 (~83 weeks) |

## Project Structure

```
.
├── NEW__X__SERVICES_converted__1_.ipynb   # Full analysis notebook
├── requirements.txt
├── README.md
├── service_data_snapshot.pkl              # Created on first DB load (local cache)
├── data/
│   ├── raw/
│   └── processed/                         # Cleaned dataframes and weekly series (.pkl)
└── outputs/
    ├── figures/                           # Actual vs forecast plots (.png)
    └── tables/                            # Model comparison, item results, next-week forecast (.csv)
```

The `data/` and `outputs/` folders are created automatically when the notebook runs.

## Setup

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab
```

Python 3.9 or newer is recommended.

## Database Access

The data is loaded from a MySQL database (schema `project_service_data`, table `service_data`). The password is **not** stored in the notebook. Provide it in one of two ways:

```bash
export SERVICE_DB_PASSWORD="your_password"     # Windows (PowerShell): $env:SERVICE_DB_PASSWORD="your_password"
```

or type it into the hidden prompt when the loading cell runs.

On the first run the data is saved to `service_data_snapshot.pkl`. To work offline afterwards, set `REFRESH_FROM_DATABASE = False` in the loading cell; the notebook will then read the local snapshot and skip the database. Note that the database inspection cell only runs when a live connection exists.

## Running the Notebook

Run all cells from top to bottom (`Kernel > Restart & Run All`). The notebook is organised in three tasks:

1. **Service Data Analysis**: data quality checks, text and date cleaning, duplicate and kilometre-reading investigation, typo correction, item classification (PART / FLUID / SERVICE / CHARGE / UNSURE), and weekly demand patterns.
2. **Weekly Demand Forecasting**: baselines, exponential smoothing, ARIMA, calendar (open-day) adjustment, Random Forest and Gradient Boosting, item-level forecasts, and robustness checks.
3. **Recommendations**: inventory and operations guidance, challenges and techniques, and the final summary.

## Method Summary

- **Cleaning:** block-repeated invoice lines (visits with 4 or more repeats) and blank item names are excluded; genuine repeat purchases are kept. Original kilometre readings are preserved and a `km_clean` column flags zero and impossible values.
- **Series:** parts and fluids only, with two sprocket items (`SPROCKET BEARING`, `SPROCKET RUBBER`) separated out because of a sudden change in billing behaviour.
- **Evaluation:** rolling one-week-ahead forecasts using only prior weeks (no look-ahead), scored on a validation window (A) and a test window (B) of 12 weeks each.
- **Final model:** simple exponential smoothing on the rate per weekday-weighted open day, multiplied by the week's exposure.

## Key Results

| Model | WAPE validation | WAPE test |
|---|---|---|
| **SES on rate x weighted exposure** | **18.7%** | **20.7%** |
| 8-week moving average | 21.9% | 23.4% |
| Naive (last week) | 30.1% | 30.3% |
| Random Forest | n/a | 22.6% |
| Gradient Boosting | n/a | 25.7% |

- Item-level forecasts are reliable for high-volume items (`ENGINE OIL`, `AIR FILTER`) and weak for low-volume items (`WHEEL RUBBER`, `SPARK PLUG`).
- Forecast for the week starting 7 Jan 2019: about **291** core stocked lines (10th to 90th percentile: about 220 to 395), assuming a full working week.

## Outputs

| File | Description |
|---|---|
| `outputs/figures/fig_core_actual_vs_forecast.png` | Total weekly actual vs forecast |
| `outputs/figures/fig_items_actual_vs_forecast.png` | Item-level actual vs forecast |
| `outputs/tables/model_comparison_core_total.csv` | All models compared on both windows |
| `outputs/tables/item_level_results.csv` | WAPE per regular item |
| `outputs/tables/next_week_forecast.csv` | Next-week forecast with uncertainty range |
| `data/processed/*.pkl` | Cleaned dataframes and weekly series |

## Limitations

- Short history (~83 weeks) rules out yearly seasonality and favours simple models.
- The late-November 2018 demand surge has no explanation in the data.
- A day with no stocked-item invoices is assumed to be a closed day.
- The model should be refitted weekly and re-evaluated monthly.

## Notes

- Vehicle numbers and business partner names in the dataset are masked.
- Pickle files should only be loaded from sources you trust.
