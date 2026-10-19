---
layout: default
title: Last-Mile Delivery Risk Predictor
---

[← Back to portfolio](../)

# Last-Mile Delivery Risk Predictor

**Will this order break the 30-minute promise? Predicting late deliveries at the moment of dispatch.**

| | |
|---|---|
| **Domain** | Logistics operations · Machine learning · Classification |
| **Data** | 19,815 orders from one day (20 Sep 2022) of a quick-service-restaurant delivery operation in Egypt, run on LogiNext's auto-allocation platform (my former employer) |
| **Tools** | Python, pandas, scikit-learn, matplotlib, seaborn, Google Colab |
| **Context** | Rice University, RCEL 506 Data Science for Engineering Leaders. I later reused it as the teaching case for my classification lecture as the course's TA |
| **Code** | [View the notebook](https://github.com/akshattripathi26/akshat_tripathi.github.io/blob/main/notebooks/last_mile_delivery_classification.ipynb) |

## Results at a glance

| ROC-AUC (test) | 5-fold CV ROC-AUC | Late orders caught | Auto vs manual on-time rate |
|:---:|:---:|:---:|:---:|
| **0.86** | **0.855 ± 0.011** | **71%** | **86% vs 50%** |

---

## The business problem

The restaurant promises delivery within 30 minutes. In this dataset, **17% of delivered orders (3,139) broke that promise**, with a median wait of 20 minutes. A supervisor watching hundreds of orders an hour can't tell which ones are at risk until it's too late.

**The question:** at the moment a rider leaves the restaurant, can we flag the orders that are likely to arrive late, so a supervisor can step in (re-assign, prioritise, or warn the customer)?

## How the approach evolved

**Phase 1: predicting the exact minutes (regression).** My first approach predicted delivery time in minutes. It didn't work well. Linear regression explained almost none of the variance (R² 0.03, MAE 7.3 min), and gradient boosting (XGBoost) only got to R² 0.16 (MAE 5.6 min). Delivery minutes are noisy because of traffic, customers who don't answer the door, and kitchen delays.

<img src="../assets/img/projects/delivery-phase1-xgboost.jpg" alt="Phase 1: XGBoost regression of predicted vs actual delivery time, R² 0.16" width="420">

**Phase 2: changing the question (classification).** The operation doesn't need the exact minute. It needs to know whether the order will miss the 30-minute promise. Reframing the problem as a yes/no classification made the model useful.

## Data preparation

- Parsed text timestamps with an explicit format, and kept only delivered orders that had complete timestamps and plausible durations: **18,395 clean orders**.
- Dropped constant, duplicate, and personal-data columns (rider names, phone numbers, employee IDs).
- Engineered **dispatch-time features only**: allocation delay, the system's ride ETA, **slack** (30 min − allocation delay − ETA), distance, hour of day, branch load, order value, payment type, city, branch, and allocation method (auto vs manual).
- Deliberately **excluded leakage features** such as actual ride time, because they're only known after delivery.

## Key findings

**1. Auto-allocation is the biggest operational lever.** Auto-allocated orders were on time **86%** of the time vs **50%** for manually assigned ones. The median time from order to dispatch was **1 minute (auto) vs 13 minutes (manual)**.

![On-time rate and dispatch delay by allocation method](../assets/img/projects/delivery-auto-vs-manual.jpg)

**2. Slack at dispatch follows an S-curve.** Orders that left the restaurant with negative slack arrived on time only **44%** of the time, vs **92%** for orders with 10 or more minutes of slack. That S-shape is exactly what logistic regression models.

![On-time rate by slack bucket](../assets/img/projects/delivery-slack-curve.jpg)

**3. Where the risk sits.** The busiest hour (15:00, about 1,300 orders) was also the worst for on-time delivery (77%). City performance ranged from 95% (Mansoura) to 79% (6th of October).

## The model

A logistic regression inside a scikit-learn `Pipeline`: scaling for the numeric features, one-hot encoding for categories, and target encoding for the 100+ branches. All encoders were fit on training data only. I used a stratified 75/25 split and `class_weight='balanced'`, because late orders are the minority class.

| Metric | Score |
|---|---|
| ROC-AUC (test) | **0.859** |
| 5-fold cross-validated ROC-AUC | 0.855 ± 0.011 |
| Recall on late orders | **0.71** (vs 0.38 for an unweighted model) |
| Accuracy | 0.80 (baseline of always predicting "on time": 0.83) |

![Confusion matrix and ROC curve](../assets/img/projects/delivery-confusion-roc.jpg)

**Why accuracy is below the baseline, and why that's the right trade.** A "model" that always predicts on time scores 83% accuracy but never catches a single late order. I accepted slightly lower accuracy to catch **7 in 10 late orders**, which is the job this tool is meant to do.

**What drives on-time delivery (odds ratios):** allocation delay is the strongest negative driver (0.37 per standard deviation). Branch track record (2.39) and slack (1.75) are the strongest positive ones.

![Odds ratios by feature](../assets/img/projects/delivery-odds-ratios.jpg)

## Guarding against leakage

Adding actual ride time pushes the AUC to 1.000, but that information only exists after the delivery is finished. I report the honest 0.86 and treat any near-perfect score as a red flag for leakage.

![ROC-AUC with and without leaked features](../assets/img/projects/delivery-leakage.jpg)

## From model to operations

On the held-out test set, the model's risk bands separate cleanly:

| Risk band | Orders | Actually on time |
|---|---:|---:|
| High (P < 0.4) | 945 | 48% |
| Medium (0.4–0.6) | 707 | 79% |
| Low (P > 0.6) | 2,947 | 95% |

**Recommendations**
1. Expand auto-allocation and keep manual assignment for exceptions only.
2. Alert supervisors when an order is dispatched with less than 5 minutes of slack.
3. Review kitchen and rider capacity at the weakest branches.
4. Use the risk score to prioritise riders and warn customers before delivery.

**Limitations:** one day of data. The model should be validated on several weeks of orders, and it would gain from rider workload, live traffic, and weather data, before any rollout.

[← Back to portfolio](../)
