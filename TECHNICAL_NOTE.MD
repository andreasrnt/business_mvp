# Technical Note – Company Risk Scoring MVP

---

## 1. Objective

Build a minimal, reproducible risk scoring pipeline using monthly inbound B2B invoice metrics.

The goal is to:
- Identify companies showing abnormal invoice and vendor behavior
- Produce a monthly risk score
- Assign interpretable risk profiles
- Generate actionable alerts
- Output results in API-ready JSON format

No labeled defaults or ground-truth risk outcomes are available.

---

## 2. Data Assumptions

Input dataset contains monthly aggregated metrics per company:

- company_id
- month_key
- monthly_spend_volatility
- ammend_rate
- friction_value_ratio
- hard_failure_rate
- vendor_concentration_index

Assumptions:
- Metrics are numeric and pre-aggregated monthly
- Each row represents one company in one month
- No missing critical columns
- No default labels available

---

## 3. Methodology

### 3.1 As-of Logic (No Data Leakage)

For each month t:

- Z-scores are computed using only data from months ≤ t
- No future information is used
- This ensures reproducibility and prevents look-ahead bias

This makes the pipeline safe for real monthly deployment.

---

### 3.2 Standardization

For each metric:

Z-score is computed as:

z = (value − historical_mean) / historical_std

Where:
- historical_mean and historical_std are calculated using data up to that month
- Population standard deviation (ddof=0) is used
- Division-by-zero is handled safely

This allows metrics with different scales to be combined.

---

### 3.3 Risk Score Construction

Company risk score is computed as:

company_risk_score = mean(z-scores across selected metrics)

All metrics are equally weighted.

Rationale:
- No supervised labels available
- Equal weighting avoids arbitrary bias
- Simplicity improves interpretability

---

### 3.4 Risk Profile Assignment (Data-Driven)

For each month:

- 25th percentile → Low Risk
- 25th–75th percentile → Moderate
- 75th percentile → High Risk

This ensures:
- Fully data-driven segmentation
- No hard-coded thresholds
- Adaptive behavior as distribution changes

---

### 3.5 Alert Generation

Alerts are triggered when metric-level z-scores exceed the 75th percentile of that metric in the same month.

Examples:
- High spend volatility
- High amendment rate
- Low invoice quality (hard failure rate)
- High friction
- Vendor concentration risk

Alerts are relative to peer distribution, not fixed values.

---

## 4. Validation Checks

Basic sanity checks performed:

- Verify month_key parsed as datetime
- Ensure sorting by company_id and month_key
- Avoid division by zero during z-score computation
- Confirm no future data used in scoring
- Confirm profile distribution per month roughly matches quantile expectation


---

## 5. Trade-offs

### Strengths
- Fully reproducible
- No heuristic thresholds
- Data-driven segmentation
- Simple and interpretable
- API-ready output

### Limitations
- No ground-truth default labels
- Equal weighting may not reflect true risk importance
- No sector normalization
- Quantile-based thresholds depend on distribution shape
- Small datasets may produce unstable percentiles

---


## 7. Future Improvements

- Weighted risk scoring using empirical calibration
- Time-series trend features (acceleration / deterioration)
- Rolling-window normalization
- Portfolio-level risk aggregation
- Early warning signal detection (momentum change)

---

## 8. Output Format

Final output:

- company_id
- month_key
- company_risk_score
- company_profile
- alert (list of strings)

Exported as structured JSON, suitable for API or dashboard integration.


End of Technical Note.