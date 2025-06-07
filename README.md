# Grid Breaks

_UC San Diego DSC 80 — Final Project_  
**Author:** Uzair Gheewala

## Introduction

Major power outages disrupt lives and economies. The U.S. Department of
Energy’s Major Power Outage database (2000 – 2016, ≈ 190 k events)
records **what** happened (`CAUSE.CATEGORY`) and **how bad** it was
(duration, customers affected, MW lost, geography).

During an initial scan we noticed one label – **fuel-supply emergency** – whose
outages last _months_, unlike storms or equipment failures. That raised two
questions:

1. **Descriptive** Do numeric outage characteristics naturally cluster in a
   way that mirrors the official cause codes?
2. **Predictive** Can we infer the cause _at the moment an outage starts_,
   using only information available in real time?

---

## Data Cleaning and Exploratory Data Analysis

### Cleaning steps

- Dropped non-data header rows from the DOE Excel file.
- Parsed start / restoration timestamps and computed `duration_hours`.
- Replaced obviously bad durations (≤ 0 h) with `NaN` and median-imputed all
  numeric features inside modelling pipelines.
- Renamed unwieldy columns:  
  `POPDEN_RURAL`, `POPDEN_UC`, … for clarity.

```text
|   YEAR | U.S._STATE   | CAUSE.CATEGORY     |   duration_hours |   CUSTOMERS.AFFECTED |   DEMAND.LOSS.MW |
|-------:|:-------------|:-------------------|-----------------:|---------------------:|-----------------:|
|   2011 | Minnesota    | severe weather     |             3060 |                70000 |              nan |
|   2014 | Minnesota    | intentional attack |                1 |                  nan |              nan |
|   2010 | Minnesota    | severe weather     |             3000 |                70000 |              nan |
|   2012 | Minnesota    | severe weather     |             2550 |                68200 |              nan |
|   2015 | Minnesota    | severe weather     |             1740 |               250000 |              250 |
```

### Univariate view

<iframe src="assets/duration_hist.html" width="850" height="480" frameborder="0"></iframe>

**Fig 1.** **fuel-supply emergencies** (red) tend to extend to greater extremes,

> 10 000 h (≈ 14 months).

### Bivariate & aggregates

<iframe src="assets/box_by_cause.html" width="900" height="500" frameborder="0"></iframe>

**Fig 2.** Durations by cause — the fuel-supply box literally towers above all
others.

```text
| CAUSE.CATEGORY                |   mean_duration |   median_customers |   count |
|:------------------------------|----------------:|-------------------:|--------:|
| fuel supply emergency         |         13484.0 |                0.0 |    38.0 |
| severe weather                |          3899.7 |           111555.0 |   741.0 |
| equipment failure             |          1850.6 |            51500.0 |    54.0 |
| public appeal                 |          1468.4 |                0.0 |    69.0 |
| system operability disruption |           747.1 |            69000.0 |   120.0 |
| intentional attack            |           521.9 |                0.0 |   332.0 |
| islanding                     |           200.5 |             2342.5 |    44.0 |
```

The table confirms that fuel-supply emergencies last on average
~13 000 h (≈ 1½ years) while other causes stay below 4 000 h.

Severe-weather events are most common but not the longest; fuel-supply
emergencies have mean duration > 3000 h.

---

### Assessment of Missingness

_One high-missingness column_ – **`DEMAND.LOSS.MW`** (~45 % NaN) – reflects the
fact that DOE only records MW loss when utilities supply it. The remaining
model features have ≤30 % missingness, with location and calendar measures
fully observed. These gaps are most likely **MAR** (measurement not always
reported) rather than MCAR.

A qualitative review suggests **`CAUSE.CATEGORY`** itself can be **NMAR**:
utilities may delay labelling politically sensitive _intentional attacks_,
leaving the field blank until investigations conclude.

<iframe
  src="assets/missing_bar.html"
  width="700"
  height="450"
  frameborder="0">
</iframe>

**Fig 3.** Fraction of NaNs per feature; we median-impute in pipelines.

---

## Hypothesis Testing

We formally test the anecdotal observation:

> _Fuel-supply emergencies last longer than intentional attacks._

- **H₀ (null):** Durations follow the same distribution.
- **H₁ (alt):** Fuel-supply durations are stochastically **greater**
  (`KS alternative="less"`).
- **Test statistic:** one-sided two-sample Kolmogorov–Smirnov.
- **Significance:** α = 0.05.

<iframe src="assets/ecdf_fse_vs_attack.html" width="750" height="480" frameborder="0"></iframe>

**Fig 4.** ECDF plot of duration show months-long tail for
fuel-supply outages.

> **Result:** KS = 0.65, p < 1 × 10⁻¹³ ⇒ **reject H₀**.
> Fuel-supply emergencies are dramatically longer (median ≈ 3900 h) than
> intentional attacks (≈ 30 h).

---

## Framing a Prediction Problem

**Task** Predict `CAUSE.CATEGORY` at outage start (multiclass
classification, 7 labels).

**Why** Early inference helps utilities dispatch the correct crews
(e.g. security vs fuel-logistics).

**Features known at time 0**

| kind              | features                                                           |
| ----------------- | ------------------------------------------------------------------ |
| Real-time numeric | `duration_hours` (current), `CUSTOMERS.AFFECTED`, `DEMAND.LOSS.MW` |
| Geography         | `POPDEN_RURAL`, `POPDEN_UC`                                        |
| Calendar          | `MONTH` (and engineered sin/cos)                                   |

**Metric** *Macro-averaged F1-score* — treats minority classes equally.

---

## Baseline Model

_StandardScaler ➜ LogisticRegression (multinomial)_ with median imputation.

- **Macro-F1 = 0.20** – the model all but ignores rare labels.

---

## Final Model

_Random-Forest_ (600 trees, class-weight balanced)

- engineered logs & cyclic month features.

* **Best parameters:** `n_estimators = 600`, `max_depth = None`
* **Macro-F1 = 0.53** (↑ 0.33 absolute).

<iframe src="assets/confusion_matrix.html" width="700" height="650" frameborder="0"></iframe>

**Fig 6.** Confusion matrix – most errors now occur among the three rarest
labels.

---

## Fairness Analysis

**Question** Does the model perform worse for outages in **rural**
(counties ≥ median `POPDEN_RURAL`) versus **urban** locations?

- **H₀:** Macro-F1 identical. **H₁:** Different.
- **Observed ΔF1 = -0.14** (rural lower).
- **Permutation p = 0.18** ⇒ fail to reject H₀.

<iframe src="assets/permutation_fairness.html" width="800" height="480" frameborder="0"></iframe>

**Fig 7.** Null distribution of ΔF1; observed gap (red) is not statistically
significant.

While the point estimate hints at a rural performance dip, evidence is
inconclusive given sample size. Future work could collect more rural events
or incorporate grid topology to close any potential fairness gap.

---

_Last updated: 2025-06-06_
