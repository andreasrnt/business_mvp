# How to Run – Company Risk Scoring


## Requirements

- Python 3.8+
- pandas
- numpy


##  Run the Script

- Open the business_mvp.ipynb
- Run every cell


## What the Script Does

- Loads monthly company metrics
- Computes z-scores using as-of logic
- Aggregates z-scores into a company_risk_score
- Assigns risk profile using monthly quantiles:
  - Bottom 25% → Low Risk
  - Middle 50% → Moderate
  - Top 25% → High Risk
- Generates alerts based on metric-level deviations
- Exports API-ready JSON output


## Example output format:

[
  {
    "company_id": 1,
    "month_key": "2025-05-01",
    "company_risk_score": 0.42,
    "company_profile": "High Risk",
    "alert": [
      "High spend volatility",
      "Low invoice quality"
    ]
  }
]

## Reproducibility

- The script is deterministic.
- Risk thresholds are fully data-driven (quantile-based).
- No hard-coded heuristic cutoffs.


## To productionize:
- Schedule monthly batch run (Airflow / cron)
- Deploy as REST API (Flask / FastAPI)
- Add sector normalization
- Backtest against historical defaults