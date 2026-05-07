
# Multivariate Statistical Process Control for Semiconductor Manufacturing

Traditional control charts monitor one sensor at a time. In semiconductor manufacturing, that approach breaks down because hundreds of sensors interact simultaneously.

This project builds a **multivariate SPC framework** using the SECOM Dataset to detect abnormal wafers, identify hidden process drift, and evaluate whether statistical anomalies translate into actual product failures.

### Core Question

> Can multivariate process monitoring detect abnormal wafers earlier than final quality inspection?

---

# Dataset
https://archive.ics.uci.edu/dataset/179/secom
* **1567 wafers**
* **590 sensor variables**
* **104 failed wafers**
* Missing values present
* Pass/fail quality labels included

Each row = one wafer
Each column = one process signal

Examples of real-world equivalents:

* chamber temperature
* pressure
* deposition rate
* vibration
* machine operating parameters

(Sensor names are anonymized in this dataset.)

---

# Project Workflow

---

## 1) Data Cleaning

Raw manufacturing datasets are noisy.

### Removed high missing-value sensors

Dropped sensors with >40% missing values.

```text id="7u3n0e"
590 → 562 sensors
```

---

## Median imputation

Filled remaining missing values using median values to reduce outlier impact.

---

## Removed constant sensors

Sensors showing zero variation were removed because they add no statistical value.

```text id="wz1fef"
562 → 446 sensors
```

---

# 2) Standardization

Used `StandardScaler`

Why?

A temperature sensor may range in thousands while pressure may range in decimals.

Without scaling:

large-value sensors dominate analysis.

After scaling:

all variables contribute equally.

---

# 3) PCA Dimensionality Reduction

This was the most important part of the project.

Instead of monitoring **446 sensors individually**, PCA compresses correlated variables into fewer process indicators.

---

## Step A: Build covariance matrix

After scaling:

```text id="2sfx4g"
1567 rows × 446 sensors
```

A covariance matrix was created:

```text id="m4xjlwm"
446 × 446
```

This matrix answers:

> Which sensors move together?

Example:

* temperature ↑
* pressure ↑

High covariance.

---

## Step B: Compute eigenvectors & eigenvalues

This is where PCA finds major process patterns.

### Eigenvectors

These represent **directions of variation**

Example:

A combination like:

```text id="11m3gr"
sensor_1 + sensor_8 + sensor_20
```

may move together.

That becomes a principal component direction.

---

### Eigenvalues

These tell:

> How much variation each principal component explains

Higher eigenvalue = more important variation pattern.

---

# Step C: Generate Principal Components

Each wafer is projected onto these new directions:

```text id="nwdxv8"
PC1
PC2
PC3
...
```

Each PC score represents how strongly a wafer behaves along that variation pattern.

---

# Step D: Retain only useful PCs

Used cumulative explained variance:

```text id="w0nrd4"
446 sensors → 162 principal components
```

while preserving:

```text id="ubf5zu"
95% of total process variance
```

This reduced monitoring complexity by ~64%.

---

# 4) Hotelling T² Control Chart

Once PCA reduced dimensionality:

each wafer now had 162 PC scores.

Hotelling T² measures how far each wafer deviates from normal process behavior.

### Formula

```text id="j5w3xv"
T² = (x − μ)ᵀ S⁻¹ (x − μ)
```

Where:

* `x` = wafer PC values
* `μ` = average process behavior
* `S⁻¹` = inverse covariance matrix

Higher T² = wafer behaves unusually.

---

## Control Limit

Used:

95th percentile threshold

to define Out-of-Control wafers.

---

## Result

* **79 wafers flagged**
* **5.04% abnormal wafers**

---

# 5) Root Cause Analysis

After identifying abnormal wafers:

the next question was:

> Which process variables caused this?

---

## Top abnormal PCs

Example:

* PC17
* PC15
* PC19

---

## Mapped back to original sensors

Top contributors:

* sensor_197
* sensor_480
* sensor_208
* sensor_200

This helps engineers narrow investigation.

---

# 6) Yield Analysis

Compared SPC signals with actual final inspection outcomes.

| Result | Within Control | Above UCL |
| ------ | -------------- | --------- |
| Fail   | 93             | 11        |
| Pass   | 1395           | 68        |

---

# Key Insights

### 11 failures were correctly detected

SPC successfully identified abnormal process behavior.

---

### 93 failures were missed

Indicates some failures may come from:

* localized defects
* downstream issues
* low-variance signals

---

### 68 passing wafers showed abnormal drift

Potential:

* early warning signals
* near misses
* hidden instability

---

# Analysis Impact

This framework helps manufacturers:

* detect drift earlier
* reduce scrap
* prioritize inspection
* improve root cause analysis
* move from reactive quality checks to proactive monitoring

---

# Tech Stack

* Python
* Pandas
* NumPy
* Scikit-learn
* Matplotlib
* Jupyter Notebook

