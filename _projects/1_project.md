---
layout: page
title: Driver Risk Prediction and Scoring
description: Prediction of crash risk based on driving behavior using machine learning.
img: assets/img/12.jpg
importance: 1
category: work
related_publications: false
---

# Project Summary

## Introduction

This project focuses on predicting risky driving behavior using a dataset of participant trip data, including features like speed, acceleration, and turning dynamics. The goal was not only to predict crash events but also to develop a meaningful driver-level risk score that could serve as a proxy for future risk—even among drivers who have never crashed.

The project was conducted with an analytical mindset, blending exploratory data analysis, iterative model development, and insight-driven segmentation of participants. The tone throughout this work is both rigorous and thoughtful, with an emphasis on making the results actionable and interpretable.

---

# Phase 1: Data Exploration & Preparation

We first explored the structure and reliability of the dataset:
- **Unique participant IDs:** 3,534
- **Unique vehicle IDs:** 3,307
- **Unique event IDs:** 28,414

Key data preparation steps included:
- Removing rows with missing or non-numeric values in key features
- Converting `timemoving` from strings with commas or "nan" to integers
- Labeling the `event_severity` column:
  - 0 = Baseline (no incident)
  - 1 = Near-Crash
  - 2 = Crash

```python
# Example: Loading and cleaning data
import pandas as pd

df_cleaned = pd.read_csv("records_cleaned.csv")
df_cleaned['timemoving'] = df_cleaned['timemoving'].astype(str).str.replace(',', '', regex=False)
df_cleaned['timemoving'] = df_cleaned['timemoving'].replace('nan', pd.NA)
df_cleaned = df_cleaned.dropna(subset=['timemoving'])
df_cleaned['timemoving'] = df_cleaned['timemoving'].astype(int)
```

![Mean Speed vs Max Acceleration (All Data)](/assets/img/1.jpg)

Initially, it was difficult to separate safe vs risky drivers by plotting `meanspeed` and `maxaccel`. The distributions largely overlapped.

---

# Phase 2: Feature Selection & Modeling Strategy

We began with a core set of nine behavioral features:
- **Speed:** `maxspeed`, `meanspeed`
- **Acceleration:** `maxaccel`, `maxdecel`
- **Lateral movement:** `maxlataccel`, `minlataccel`, `maxturnrate`, `minturnrate`
- **Duration:** `timemoving`

### Model Training

We trained three binary classification models using XGBoost:
- Crash vs No Crash
- Near-Crash vs No Crash
- Any Risk (Crash or Near-Crash) vs No Crash

Each model was evaluated using:
- Train/Test accuracy
- Confusion matrices
- Classification reports

**Model Results Summary:**

| Model                  | Train Accuracy | Test Accuracy | Notes                     |
|-------------------------|----------------|---------------|---------------------------|
| Crash vs No Crash       | 98.37%          | 94.48%        | Strong on Crash Detection |
| Near-Crash vs No Crash  | 96.26%          | 91.97%        | Good Near-Crash Separation|
| Any Risk vs No Crash    | 94.53%          | 88.81%        | Broader risk detection    |

---

# Phase 3: Insight — Crashers Are a Different Group

Drivers who eventually crashed exhibited risky behavior even during normal trips.

![Mean Speed vs Max Acceleration (Safe Participants)](/assets/img/2.jpg)

![Mean Speed vs Max Acceleration (Crash Participants)](/assets/img/3.jpg)

We observed clear shifts in behavior between safe and crash participants, motivating a split in model testing.

**Feature Correlations:**

![Correlation Heatmap (Safe Participants)](/assets/img/4.jpg)

![Correlation Heatmap (Crash Participants)](/assets/img/5.jpg)

![Correlation Heatmap (All Participants)](/assets/img/6.jpg)

---

# Phase 4: Scoring Drivers Based on Multi-Model Flags

Recognizing model variability, we developed a **weighted risk scoring system**.

| Model Type  | Weight |
|-------------|--------|
| Crash       | 3      |
| Near-Crash  | 2      |
| Any Risk    | 1      |

Each trip received a score between 0-6 depending on model flags.

**Risk Categories and Distribution:**

| Risk Score Range | High-Risk Trip % | Final Risk Label |
|------------------|------------------|------------------|
| ≤ 3              | < 5%              | Very Low Risk    |
| ≤ 4              | < 10%             | Low Risk         |
| ≥ 5              | 10–30%           | Moderate Risk    |
| ≥ 5              | 30–60%           | High Risk        |
| ≥ 5              | > 60%             | Very High Risk   |

### Risk Summary:

| Participant ID | Total Trips | High Risk Trips | High Risk % | Risk Label     |
|----------------|-------------|-----------------|-------------|----------------|
| 100926         | 9           | 0               | 0%          | Very Low Risk  |
| 101017         | 16          | 3               | 18.75%      | Moderate Risk  |
| 101134         | 15          | 4               | 26.67%      | Moderate Risk  |
| 101626         | 3           | 0               | 0%          | Very Low Risk  |
| 101681         | 4           | 0               | 0%          | Very Low Risk  |

```python
# Risk scoring function example
from risk_score_builder import build_risk_scores

df_scored, driver_risk_summary = build_risk_scores(df_never_crashed, crash_model, near_model, risk_model)
```

---

# Final Thoughts

Key lessons from the project:
- Understanding segment behavior greatly improves predictive modeling.
- Combining multiple models with intelligent weighting improves robustness.
- Creating user-level risk scores transforms prediction into actionable insights.

---

# Future Work

- Temporal modeling of driving behavior
- Testing ensemble models beyond XGBoost
- Tailoring scoring systems for insurance, fleet safety, etc.

---


