---
layout: page
title: Driver Risk Prediction and Scoring
description: Prediction of crash risk based on driving behavior using machine learning.
img: assets/img/12.jpg
importance: 1
category: work
related_publications: true
---

# Project Summary

## Introduction

This project focuses on predicting risky driving behavior using a dataset of participant trip data, including features like speed, acceleration, and turning dynamics. The goal was not only to predict crash events but also to develop a meaningful driver-level risk score that could serve as a proxy for future risk—even among drivers who have never crashed.

The project was conducted with an analytical mindset, blending exploratory data analysis, iterative model development, and insight-driven segmentation of participants. The tone throughout this work is both rigorous and thoughtful, with an emphasis on making the results actionable and interpretable.

---

# Phase 1: Data Exploration & Preparation

The first step was understanding the structure and reliability of the dataset. The data contained multiple trips per participant and various driving behavior metrics (e.g., `maxspeed`, `maxaccel`, `minturnrate`).

Key data preparation steps included:

- Removing rows with missing or non-numeric values in key features
- Converting columns like `timemoving` from string representations (e.g., with commas or "nan") to integers
- Labeling the `event_severity` column:
  - 0 = Baseline (no incident)
  - 1 = Near-Crash
  - 2 = Crash

<!-- Insert: Cleaned data preview table or screenshot here -->

---

# Phase 2: Feature Selection & Modeling Strategy

We began with a core set of nine behavioral features that intuitively capture risky driving:

- **Speed**: `maxspeed`, `meanspeed`
- **Acceleration**: `maxaccel`, `maxdecel`
- **Lateral and turning behavior**: `maxlataccel`, `minlataccel`, `maxturnrate`, `minturnrate`
- **Time spent moving**: `timemoving`

### Model Training:

Three binary classification models were trained using XGBoost:

- Crash vs No Crash
- Near-Crash vs No Crash
- Any Risk (Crash or Near-Crash) vs No Crash

Each model was evaluated using:

- Train/Test accuracy
- Confusion matrices
- Classification reports

<!-- Insert: Model performance metrics and confusion matrices here -->

### Additional Features:

We explored adding bucketed time/distance features, but found marginal gains (<5%).  
The core feature set was retained for clarity and consistency.

---

# Phase 3: Insight — Crashers Are a Different Group

A major insight emerged: drivers who eventually crashed often had high-risk metrics even during their "normal" trips without a crash.

This revealed a key modeling challenge:

- Risky driving patterns appeared both when a crash happened and when it didn’t.
- Generalizing from crash-only labels was not always sufficient.

### Improving Realism:

- We isolated drivers who **never** crashed.
- Models were tested on these "safe" drivers to simulate real-world deployment.

<!-- Insert: Graph comparing flagged trips across crashers vs non-crashers -->

---

# Phase 4: Scoring Drivers Based on Multi-Model Flags

Recognizing that different models had different predictive strengths, we developed a **weighted scoring system**:

| Model Type  | Weight |
|-------------|--------|
| Crash       | 3      |
| Near-Crash  | 2      |
| Any Risk    | 1      |

Each trip received a score (0–6) based on the models that flagged it.  
Driver-level scores were aggregated to create a full risk profile.

### Risk Categories:

| Risk Score Range | High-Risk Trip % | Final Risk Label |
|------------------|------------------|------------------|
| ≤ 3              | < 5%              | Very Low Risk    |
| ≤ 4              | < 10%             | Low Risk         |
| ≥ 5              | 10–30%            | Moderate Risk    |
| ≥ 5              | 30–60%            | High Risk        |
| ≥ 5              | > 60%             | Very High Risk   |

<!-- Insert: Table of sample driver-level risk summaries here -->
<!-- Insert: Bar chart showing number of drivers by risk category -->

---

# Final Thoughts

This project demonstrates several key lessons:

- **Understanding segment behavior** dramatically improves modeling strategies.
- **Combining multiple models** using a weighted scoring system leads to more reliable insights.
- **Creating user-level risk profiles** is a powerful extension beyond basic classification tasks.

The result is a predictive system that can:

- Identify risky driving behavior patterns,
- Forecast potential risk among safe drivers,
- Provide actionable, interpretable scoring for future applications.

---

# Future Work

- Incorporating temporal driving trends (e.g., changes in behavior over time)
- Testing ensemble models beyond XGBoost
- Fine-tuning thresholds for different use cases (insurance pricing, driver training feedback)

---

# Planned Inserts

✅ Cleaned Data Preview  
✅ Model Metrics & Confusion Matrices  
✅ Comparison Graph: Crashers vs Non-Crashers  
✅ Sample Driver Risk Summary Table  
✅ Risk Distribution Bar Chart  
✅ Example Code Snippets (Preprocessing, Modeling, Scoring)

---

