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

```markdown
| variables | OBS | YEAR | MONTH | U.S.\_STATE | POSTAL.CODE | NERC.REGION | CLIMATE.REGION | ANOMALY.LEVEL | CLIMATE.CATEGORY | OUTAGE.START.DATE | OUTAGE.START.TIME | OUTAGE.RESTORATION.DATE | OUTAGE.RESTORATION.TIME | CAUSE.CATEGORY | CAUSE.CATEGORY.DETAIL | HURRICANE.NAMES | OUTAGE.DURATION | DEMAND.LOSS.MW | CUSTOMERS.AFFECTED | RES.PRICE | COM.PRICE | IND.PRICE | TOTAL.PRICE | RES.SALES | COM.SALES | IND.SALES | TOTAL.SALES | RES.PERCEN | COM.PERCEN | IND.PERCEN | RES.CUSTOMERS | COM.CUSTOMERS | IND.CUSTOMERS | TOTAL.CUSTOMERS | RES.CUST.PCT | COM.CUST.PCT | IND.CUST.PCT | PC.REALGSP.STATE | PC.REALGSP.USA | PC.REALGSP.REL | PC.REALGSP.CHANGE | UTIL.REALGSP | TOTAL.REALGSP | UTIL.CONTRI | PI.UTIL.OFUSA | POPULATION | POPPCT_URBAN | POPPCT_UC | POPDEN_URBAN | POPDEN_UC | POPDEN_RURAL | AREAPCT_URBAN | AREAPCT_UC | PCT_LAND | PCT_WATER_TOT | PCT_WATER_INLAND | OUTAGE.START | OUTAGE.RESTORATION | duration_hours |\n|------------:|------:|-------:|--------:|:-------------|:--------------|:--------------|:-------------------|----------------:|:-------------------|:--------------------|:--------------------|:--------------------------|:--------------------------|:-------------------|:------------------------|------------------:|------------------:|-----------------:|---------------------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|:--------------------|:---------------------|-----------------:|\n| nan | 1 | 2011 | 7 | Minnesota | MN | MRO | East North Central | -0.3 | normal | 2011-07-01 00:00:00 | 17:00:00 | 2011-07-03 00:00:00 | 20:00:00 | severe weather | nan | nan | 3060 | nan | 70000 | 11.6 | 9.18 | 6.81 | 9.28 | 2332915 | 2114774 | 2113291 | 6562520 | 35.5491 | 32.225 | 32.2024 | 2.30874e+06 | 276286 | 10673 | 2.5957e+06 | 88.9448 | 10.644 | 0.411181 | 51268 | 47586 | 1.07738 | 1.6 | 4802 | 274182 | 1.75139 | 2.2 | 5.34812e+06 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2011-07-01 17:00:00 | 2011-07-03 20:00:00 | 3060 |\n| nan | 2 | 2014 | 5 | Minnesota | MN | MRO | East North Central | -0.1 | normal | 2014-05-11 00:00:00 | 18:38:00 | 2014-05-11 00:00:00 | 18:39:00 | intentional attack | vandalism | nan | 1 | nan | nan | 12.12 | 9.71 | 6.49 | 9.28 | 1586986 | 1807756 | 1887927 | 5284231 | 30.0325 | 34.2104 | 35.7276 | 2.34586e+06 | 284978 | 9898 | 2.64074e+06 | 88.8335 | 10.7916 | 0.37482 | 53499 | 49091 | 1.08979 | 1.9 | 5226 | 291955 | 1.79 | 2.2 | 5.45712e+06 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2014-05-11 18:38:00 | 2014-05-11 18:39:00 | 1 |\n| nan | 3 | 2010 | 10 | Minnesota | MN | MRO | East North Central | -1.5 | cold | 2010-10-26 00:00:00 | 20:00:00 | 2010-10-28 00:00:00 | 22:00:00 | severe weather | heavy wind | nan | 3000 | nan | 70000 | 10.87 | 8.19 | 6.07 | 8.15 | 1467293 | 1801683 | 1951295 | 5222116 | 28.0977 | 34.501 | 37.366 | 2.30029e+06 | 276463 | 10150 | 2.5869e+06 | 88.9206 | 10.687 | 0.392361 | 50447 | 47287 | 1.06683 | 2.7 | 4571 | 267895 | 1.70627 | 2.1 | 5.3109e+06 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2010-10-26 20:00:00 | 2010-10-28 22:00:00 | 3000 |\n| nan | 4 | 2012 | 6 | Minnesota | MN | MRO | East North Central | -0.1 | normal | 2012-06-19 00:00:00 | 04:30:00 | 2012-06-20 00:00:00 | 23:00:00 | severe weather | thunderstorm | nan | 2550 | nan | 68200 | 11.79 | 9.25 | 6.71 | 9.19 | 1851519 | 1941174 | 1993026 | 5787064 | 31.9941 | 33.5433 | 34.4393 | 2.31734e+06 | 278466 | 11010 | 2.60681e+06 | 88.8954 | 10.6822 | 0.422355 | 51598 | 48156 | 1.07148 | 0.6 | 5364 | 277627 | 1.93209 | 2.2 | 5.38044e+06 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2012-06-19 04:30:00 | 2012-06-20 23:00:00 | 2550 |\n| nan | 5 | 2015 | 7 | Minnesota | MN | MRO | East North Central | 1.2 | warm | 2015-07-18 00:00:00 | 02:00:00 | 2015-07-19 00:00:00 | 07:00:00 | severe weather | nan | nan | 1740 | 250 | 250000 | 13.07 | 10.16 | 7.74 | 10.43 | 2028875 | 2161612 | 1777937 | 5970339 | 33.9826 | 36.2059 | 29.7795 | 2.37467e+06 | 289044 | 9812 | 2.67353e+06 | 88.8216 | 10.8113 | 0.367005 | 54431 | 49844 | 1.09203 | 1.7 | 4873 | 292023 | 1.6687 | 2.2 | 5.48959e+06 | 73.27 | 15.28 | 2279 | 1700.5 | 18.2 | 2.14 | 0.6 | 91.5927 | 8.40733 | 5.47874 | 2015-07-18 02:00:00 | 2015-07-19 07:00:00 | 1740 |
```

### Univariate view

<iframe src="assets/duration_hist.html" width="850" height="480" frameborder="0"></iframe>

**Fig 1.** **fuel-supply emergencies** (red) tend to extend to greater extremes,

> 10 000 h (≈ 14 months).

### Bivariate & aggregates

<iframe src="assets/box_by_cause.html" width="900" height="500" frameborder="0"></iframe>

**Fig 2.** Durations by cause — the fuel-supply box literally towers above all
others.

| CAUSE.CATEGORY | mean_duration | median_customers | count |\n|:------------------------------|----------------:|-------------------:|--------:|\n| fuel supply emergency | 13484.0 | 0.0 | 38.0 |\n| severe weather | 3899.7 | 111555.0 | 741.0 |\n| equipment failure | 1850.6 | 51500.0 | 54.0 |\n| public appeal | 1468.4 | 0.0 | 69.0 |\n| system operability disruption | 747.1 | 69000.0 | 120.0 |\n| intentional attack | 521.9 | 0.0 | 332.0 |\n| islanding | 200.5 | 2342.5 | 44.0 |

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

```

```
