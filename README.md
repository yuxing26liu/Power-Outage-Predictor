# Outage Severity Across the U.S.: Predicting Risk Factors Behind Major Power Failures
** Yuxing Liu, Rocio Saez

**Website Link**:

## Introduction

This project investigates major power outages in the continental United States from January 2000 to July 2016. These events are defined by the Department of Energy as incidents affecting at least 50,000 customers or causing an unplanned load loss of 300 MW or more. The dataset, provided by Purdue University’s LASCI lab, includes outage-specific information as well as regional characteristics like climate, electricity usage, population density, and economic indicators.

Our goal is to answer the following question:
**What are the key characteristics of major power outages with higher severity?**
Severity is defined by:

* Number of customers affected
* Duration of the outage
* Demand loss (in Megawatts)

We examine how severity varies with location, time, electricity usage patterns, and urban-rural distribution, and build predictive models for outage duration.

## Data Cleaning and Exploratory Data Analysis

We began by removing irrelevant columns to focus on severity-related variables. Columns about land area and economic indicators were dropped. We also dropped rows with missing `OUTAGE.DURATION`, as this is one of our target variables.

Next, we created a unified timestamp column `OUTAGE.START` by combining `OUTAGE.START.DATE` and `OUTAGE.START.TIME`. Similarly, `OUTAGE.RESTORATION` was created to represent restoration timestamps.

We explored the distribution of outage causes and found that weather-related causes dominate, particularly in states like Texas, California, and New York. The most frequent causes include equipment failure, storms, and transmission disruptions.

### Example Univariate Plots

* A bar chart showing outage frequency by cause
* A bar chart showing outage frequency by state

## Assessment of Missingness

We focused on `DEMAND.LOSS.MW`, which had 705 missing values. We performed permutation tests and found that its missingness **depends on the outage cause category**, suggesting that certain types of outages are less likely to report demand loss. On the other hand, we tested and found that its missingness **does not depend** on `OUTAGE.START.TIME`, implying that reporting time of day doesn’t influence whether demand loss is reported.

Additionally, we reasoned that `CUSTOMERS.AFFECTED` may be **Not Missing At Random (NMAR)** because utilities may underreport small-scale outages or ones resolved quickly.

## Hypothesis Testing

We tested the hypothesis: **Are summer outages more severe (in duration) than those in other seasons?**

* **Null Hypothesis**: The average outage duration in summer is the same as in other seasons.
* **Alternative Hypothesis**: The average outage duration in summer is greater.

We performed a permutation test using the difference in means between summer and non-summer outages. The observed difference was significantly higher than the null distribution, leading us to reject the null hypothesis. Thus, **summer outages tend to last longer**.

## Framing a Prediction Problem

We attempt to **predict `OUTAGE.DURATION`**, a key indicator of outage severity. This is a **regression problem**, as duration is a continuous variable.

## Baseline Model

Our baseline model predicts the mean outage duration for all examples. This sets a simple benchmark (e.g., RMSE around the overall mean duration). We'll use this to evaluate more complex models.

## Final Model

We trained a multiple linear regression model using features such as:

* `MONTH`
* `CAUSE.CATEGORY`
* `CLIMATE.REGION`
* `RES.CUSTOMERS`, `TOTAL.PRICE`, `TOTAL.SALES`

This model performs better than the baseline, capturing regional and temporal trends affecting outage duration.

## Fairness Analysis

We explored whether prediction errors differed significantly by region. For instance, we computed residuals grouped by `CLIMATE.REGION` and found that some regions (e.g., Southeast) had higher average errors. This suggests potential **regional bias**, likely due to imbalanced data. Future work could include region-specific models or reweighting.

