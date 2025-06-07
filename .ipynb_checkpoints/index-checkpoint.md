---
layout: default
title: Power-Outage-Risks-Analysis
---
# Outage Severity Across the U.S.: Predicting Risk Factors Behind Major Power Failures

**Yuxing Liu, Rocio Saez Aguirre**

## Introduction

Power outages often strike without warning and can leave communities without power for hours or even days resulting in safety risks, lost productivity, and costly economic disruptions. By understanding which factors (such as outage cause, geographic region, or population density) are most predictive of how long an outage will last, utilities and emergency planners can proactively allocate repair crews, pre‐stage backup resources, and communicate realistic restoration timelines to customers. 

In this project, we will examine the Major Power Outage Risks in the U.S. dataset provided by Purdue University which comprises **1,534** U.S. outage events reported between 2000 and 2016, with **57** total columns. This dataset captures detailed information on when and where outages occurred, what caused them, and how many customers were affected. The relevant columns we have are: 

**Relevant Columns:**

| Column Name                                  | Description                                                                                                                                      |
| :------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| `OUTAGE.DURATION`                            | Total outage length in minutes. This is our response variable (the “duration” we aim to predict).                                                |
| `CUSTOMERS.AFFECTED`                         | Number of customers impacted. Outages affecting more customers often require larger restoration efforts and thus take longer.                     |
| `DEMAND.LOSS.MW`                             | Peak demand lost (in megawatts). Higher demand loss typically signals more extensive damage, correlating with longer repair times.                |
| `CAUSE.CATEGORY`                             | High-level cause (e.g. “severe weather,” “equipment failure,” “intentional attack”). Certain causes (like hurricanes) are known to produce prolonged outages. |
| `CLIMATE.REGION`                             | U.S. census-based climate region (e.g. “South,” “East North Central,” etc.). Captures broad weather-pattern differences and infrastructure vulnerabilities. |
| `U.S._STATE`                                 | State where the outage occurred. Reflects variations in grid age, regulatory environment, and utility practices that can affect repair speed.     |
| `POPPCT_URBAN`                               | Percentage of the state’s population living in urban areas. Higher urban density can mean faster crew mobilization but also more customers to restore. |
| `OUTAGE.START.DATE` & `OUTAGE.START.TIME`     | Combined into `OUTAGE.START` timestamp. Enables extraction of month, day-of-week, and holiday/weekday indicators, all of which influence crew availability. |
| `OUTAGE.RESTORATION.DATE` & `OUTAGE.RESTORATION.TIME` | Combined into `OUTAGE.RESTORATION`. Used (with `OUTAGE.START`) to compute `OUTAGE.DURATION` and to assess missingness when restoration times are unreported. |


We will initially clean and explore the outage dataset, assess missingness patterns, and use hypothesis tests to answer targeted questions about outage characteristics. Doing this will allow us to have a clear understanding of which features appear most related to outage severity and whether any statistical relationships hold up under formal testing. Then, we will leverage the insights we gathered to build a regression model that predicts the duration of a power outage at the moment it begins. We start with a simple baseline, engineer additional features and tune hyperparameters for a final model, and then evaluate whether that model treats certain subgroups (e.g., by region) fairly.

This dataset will help us answer the question: **which combination of factors (outage cause, geographic location, climate region, population density, time‐of‐year, etc.) best predicts how long (in minutes) a major U.S. power outage will last?**


# Data Cleaning & Exploratory Data Analysis

## Data Cleaning

### Remove Unnecessary Rows
- The first three rows of the raw data contain metadata and unit‐definition information, not actual outage records so they were removed to ensure that subsequent processing only includes valid event entries.

### Drop Irrelevant Columns

#### Land‐Use Columns
The following land‐use variables (related to total land area and water surface) were dropped because they do not directly inform outage severity:
- `AREAPCT_URBAN`
- `AREAPCT_UC`
- `PCT_LAND`
- `PCT_WATER_TOT`
- `PCT_WATER_INLAND`

#### State‐Level Economic Output Columns
The following state‐economic columns (e.g., gross state product metrics, utility sector contributions) were dropped because they focus on broad economic analyses rather than immediate outage impact:
- `PC.REALGSP.STATE`
- `PC.REALGSP.USA`
- `PC.REALGSP.REL`
- `PC.REALGSP.CHANGE`
- `UTIL.REALGSP`
- `TOTAL.REALGSP`
- `UTIL.CONTRI`
- `PI.UTIL.OFUSA`

> **Why?:**  
> These variables, while informative, were not directly relevant to my research question about outage severity. Since my focus is on population density, climate, electricity use, and economic characteristics, I retained variables more closely tied to human impact and energy infrastructure rather than surface geography.


### Focus on Severity Metrics & Handle Missing Duration
Since one of the key goals of this project is to understand the **severity** of major power outages, we focus on three core metrics:
- `CUSTOMERS.AFFECTED` (number of customers impacted)
- `DEMAND.LOSS.MW` (peak megawatt demand lost)
- `OUTAGE.DURATION` (total outage length in minutes)

Among these, `OUTAGE.DURATION` is a direct indicator of how long an outage lasted — a critical dimension of impact for residents, businesses, and infrastructure. An outage without a known duration cannot be meaningfully compared or modeled for severity. Therefore:
- **All rows with missing `OUTAGE.DURATION`** (58 total) were dropped from the dataset.
- This ensures that every remaining outage event has a known start‐to‐restoration interval, allowing for consistent severity modeling.

### Combine Date & Time into Timestamps
The original data provides separate columns for date and time. To streamline temporal analysis, we:
1. **Combine**  
   - `OUTAGE.START.DATE` + `OUTAGE.START.TIME` → new column `OUTAGE.START` (type: `pd.Timestamp`)  
   - `OUTAGE.RESTORATION.DATE` + `OUTAGE.RESTORATION.TIME` → new column `OUTAGE.RESTORATION` (type: `pd.Timestamp`)
2. **Drop** the original separate date/time columns (`OUTAGE.START.DATE`, `OUTAGE.START.TIME`, `OUTAGE.RESTORATION.DATE`, `OUTAGE.RESTORATION.TIME`).

> **Why?**  
> Having a single timestamp for start and restoration allows us to easily extract:
> - Month and day‐of‐week (for seasonality analysis)  
> - Holiday versus weekday indicators (for resource availability)  
> - Direct computation of `OUTAGE.DURATION`  

After completing these steps, the cleaned DataFrame contains only the columns and rows relevant for our exploratory analysis and modeling of outage duration. Below is the head of the cleaned DataFrame for verification:


## Univariate Analysis
In our exploratory data analysis, we performed univariate analysis to examine the distribution of single variables.

We first wanted to see  the distribution of major power outages across different cause categories so we used a **bar chart** to display this. 

<iframe
  src="assets/cause_category_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The most common cause by far is severe weather, followed by intentional attacks and system operability disruptions. This highlights the significant impact of natural events and security threats on the power grid. Understanding the dominant causes helps utilities prioritize infrastructure improvements and disaster response strategies.

Then, we also wanted to see the number of major outages by U.S. state from 2000 to 2016, so we used another **bar chart**. 

<iframe
  src="assets/outages_by_state.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

To better understand, we created a **choropleth map** of power outages by state where the darker shades indicate states with more frequent outages.

<iframe
  src="assets/outages_choropleth_map.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Through both of these graphs, we can see that California leads with the highest number of outages, followed by Texas, Michigan, and Washington. States with large populations, varied terrain, or weather exposure tend to experience more frequent outages. The long tail of the distribution shows many states with relatively few major outages, indicating regional variation in outage frequency.


## Bivariate Analysis
We then performed bivariate analysis to examine the relationship between two of our columns. 

We first tried to see if certain climate regions were more prone to severe outages (longer duration or more affected customers) through our **box plot**.

<iframe
  src="assets/outage_by_climate_region.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot allows us to see how outage duration varies across different U.S. climate regions. The East North Central and Northeast regions display longer and more variable outage durations, as seen from the wider spread and presence of high outliers. This suggests that regional climate factors—like extreme snow or storms—may affect how long it takes to restore power.

Then, we tried to see if there is a relationship between percentage (%) of urban population (`POPPCT_URBAN`) and its severity through a **scatterplot**.

<iframe
  src="assets/urban_vs_customers.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This scatter plot explores the relationship between how urban a state is and how many customers are affected during major outages. While most outages cluster below 1 million affected customers, there is a noticeable upward spread in highly urbanized states (above 80% urban population), suggesting denser regions might be more vulnerable to large-scale service disruption.

Lastly, we tried to see if outages in summer or winter tend to be longer or larger through a **box plot**.

<iframe
  src="assets/duration_by_month.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This box plot breaks down outage duration by month to reveal potential seasonal effects. August, September, and October show slightly higher median durations and more high-end outliers, which may correspond to peak storm or hurricane seasons. These findings suggest that certain months may require increased readiness and response resources.

## Grouping & Aggregates
Below is a pivot table showing the **mean outage duration (in minutes)** for each combination of climate region and cause category. Rows correspond to `CLIMATE.REGION`, and columns correspond to `CAUSE.CATEGORY`:

| CLIMATE.REGION      | equipment failure | fuel supply emergency | intentional attack | islanding | public appeal | severe weather | system operability disruption |
| :------------------ | -----------------:| ---------------------:| ------------------:| --------:| -------------:| --------------:| ------------------------------:|
| Central             |               322 |             10035.2   |             346.1  |    125.3 |          1410 |           3250 |                       2695.2 |
| East North Central  |            26435.3|             33971.2   |            2376    |      1   |           733 |          4434.8|                        2610  |
| Northeast           |             215.8 |             14629.6   |             196    |    881   |          2655 |          4429.9|                         773.5|
| Northwest           |             702   |                 1     |             373.8  |     73.3 |           898 |           4838 |                         141   |
| South               |             295.8 |             17482.5   |             325.6  |    493.5 |          1164 |          4391.3|                         866.1 |
| Southeast           |             554.5 |                 nan   |             504.7  |     nan  |         2865.4|          2662.6|                         169.3 |
| Southwest           |             113.8 |                76     |             265.7  |      2   |         2275  |         11572.9|                         329.2 |
| West                |             524.8 |              6154.6   |             857.7  |    214.9 |         2028.1|          2928.4|                         363.7 |
| West North Central  |              61   |                 nan   |              23.5  |     68.2 |          439.5|          2442.5|                          nan   |

This pivot table reveals clear regional patterns in how long major power outages last, depending on their cause. For instance:
- Severe weather causes especially long outages in the Southwest and Northwest (11,572.9 and 4,838.0 minutes, respectively).
- Fuel supply emergencies lead to very prolonged outages in East North Central and South.
- Outages due to equipment failure are unusually high in East North Central, potentially pointing to aging or fragile infrastructure.

The regional disparities underscore that the same cause may have drastically different impacts depending on local conditions, infrastructure readiness, and climate challenges. This insight can guide location-specific planning for utilities and policymakers.


# Assessment of Missingness

## NMAR Analysis
We believe the column `CUSTOMERS.AFFECTED` is likely Not Missing At Random (NMAR) since the decision to leave the value blank may depend on the value itself — for example, if only a small number of customers were affected and the utility deemed it not worth reporting. In such cases, the missingness depends directly on the (unseen) value of `CUSTOMERS.AFFECTED`, making it NMAR.

To determine if this missingness could instead be MAR, we would need additional metadata about the reporting policies or criteria used by each utility during the event, such as thresholds for whether customer impact was logged.

We will test if the `DEMAND.LOSS.MW` missingness is related to/depends on the cause of the outage.

<iframe
  src="assets/demand_missing_permutation.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

As seen in the histogram above, the observed standard deviation is far outside the range of the null distribution, with a p-value < 0.001.

This indicates that missingness in `DEMAND.LOSS.MW` is dependent on `CAUSE.CATEGORY`, suggesting Missing At Random (MAR). In other words, the missingness can be explained by another observed column.

## Missingness Dependency
We will conduct a permutation test to assess whether the missingness in the `DEMAND.LOSS.MW` column depends on `OUTAGE.START.TIME`.

>**Null Hypothesis**: The distribution of `DEMAND.LOSS.MW` is the **same** when `OUTAGE.START.TIME` is missing vs not missing.

>**Alternate Hypothesis**: The distribution of `DEMAND.LOSS.MW` is **different** when `OUTAGE.START.TIME` is missing vs not missing.

<iframe
  src="assets/time_missingness_permutation.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The test statistic (standard deviation of missingness across times) closely matches the null distribution generated by permutation. Since the observed value is not extreme and falls well within the range of expected variation under the null hypothesis, we conclude that the missingness of `DEMAND.LOSS.MW` **does not** depend on `OUTAGE.START.TIME`.

Therefore, with respect to this column, the missingness is consistent with Missing Completely At Random (MCAR).


# Hypothesis Testing

We will now test whether summer outages are more severe (last longer in duration) than in all other seasons. Since outage durations are right‐skewed and may not be normally distributed, we chose a **Mann–Whitney U test** (a nonparametric test) rather than a t‐test, which assumes normality of the underlying distributions. Our hypothesis are as so:

>**Null Hypothesis**: Outages in summer are not longer than outages in other seasons.
>**Alternative Hypothesis**: Outages in summer have significantly greater durations than those in other seasons.

<iframe
  src="assets/summer_vs_others_boxplot.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

After running our test, we find out there is no significant evidence that power outages in summer are longer than those in other seasons. The distribution of durations appears similar across seasons based on the boxplot and statistical test.

<iframe
  src="assets/summer_duration_permutation.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

So, we **fail to reject the null hypothesis**. There is no statistically significant evidence that outages in summer are more severe (last longer in duration) than those in other seasons. This helps us understand that seasons don't affect duration, therefore, we won't be using this column to help us with our prediction model.

# Framing Our Prediction Problem

Our goal is to predict how long (in minutes) a power outage will last using only information available at the moment the outage begins. We chose to predict outage duration since accuretly forecasting duration helps utilities allocate crews, pre‐stage backup resources, and communicate realistic timelines to affected customers. 

This histogram shows the distribution of outage durations (in minutes) across all events in the dataset. Most outages are relatively short, but some last for tens of thousands of minutes, indicating extreme severity in certain cases.

<iframe
  src="assets/outage_duration_hist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

At the time of prediction, we would know these given features: 
- `CAUSE.CATEGORY` 
- `CLIMATE.REGION` 
- `U.S._STATE`
- `OUTAGE.START`
- `POPDEN_URBAN`

This information will allow us to predict how long (in minutes) a power outage will last.

Based on our analyses so far —our finding that the summer outages don't have difference in duration than other season and our EDA showing how certain features influence outage length— we will set up a **regression problem**: 

> We will build a regression model that, given certain features, predicts how many minutes an outage will last.

The metric we are using the evaluate my model is the **Root Mean Squared Error (RMSE)** since itpenalizes large errors more heavily, which is important be cause very prolonged outages (tens of thousands of minutes) have outsized consequences and it is measured in the same units as the response (minutes), making it directly interpretable for utilities.

## Baseline Model
We begin with the **baseline regression model** that uses only the two categorical predictors: `CAUSE.CATEGORY` and `CLIMATE.REGION`. We split the cleaned data into 80 % for training and 20 % for testing.

To prepare them for modeling:
- We apply a `ColumnTransformer` with a `OneHotEncoder(handle_unknown="ignore")` to both `CAUSE.CATEGORY` and `CLIMATE.REGION`.  
  - This creates one binary indicator column for each unique category value 
  - Since there are no ordinal features in the baseline, no ordinal encoding is needed.  
  - There are also no numeric features at this stage, so no scaling or other transformation is performed.

The transformed features feed into a `RandomForestRegressor` with **100 trees**. We train using an 80%/20% train/test split of the cleaned dataset. After training, we predict on the held‐out test set and compute test‐set RMSE and R². This baseline establishes a performance floor: it shows how well we can predict outage duration using only cause and region information. 

Baseline Model Performance:
  • RMSE = 7398.31 minutes
  • R²   = 0.1080

The baseline model has an RMSE of 7 398 minutes (≈ 123 hours ≈ 5.12 days), meaning its outage‐duration predictions are off by about five days on average. Its R² of 0.108 indicates it explains only 10.8 % of the variance in true outage durations. This model is not good since our RMSE is too large for operational decision‐making, and our low R² shows that predicting outage duration based only on cause and region is insufficient. 

In other words, **cause and region alone capture very little of the signal, and most of the variability in outage length remains unexplained at this baseline level.** To improve this, we need more information/indicators to reach our target which is what we will do in step 7.


## Step 7: Final Model
Building on the baseline and our ideas to improve our model, we will now include, apart from `CAUSE.CATEGORY` and `CLIMATE.REGION`,  **2 additional predictors** and **2 engineered features**: 

### Current Predictors
1. **`CAUSE.CATEGORY`** (nominal categorical)  
   - **Why:** Different causes have inherently different repair complexities. For example, severe weather events (hurricanes, ice storms) often require more extensive crew mobilization and infrastructure repair, leading to longer outages than equipment failures or intentional attacks.

2. **`CLIMATE.REGION`** (nominal categorical)  
   - **Why:** Geographic regions experience distinct weather patterns and grid‐hardening standards. The South, for instance, is prone to hurricanes, while the Northeast faces Nor’easters—each affecting repair time differently.

### Additional Predictors
3. **`U.S._STATE`** (nominal categorical)  
   - **Why:** State‐level regulatory environments, utility practices, and infrastructure age vary. Two outages from the same cause in different states can have very different restoration timelines depending on local crew availability, permitting processes, and grid resilience measures.

4. **`POPDEN_URBAN`** (quantitative)  
   - **Why:** Urban density influences both the number of customers affected and how quickly crews can access damaged equipment. High‐density areas may have more complex logistics (e.g., traffic delays, multi‐failure circuits) but can also mobilize urban‐focused resources faster.


### Feature Engineered

5. **`dow`** (quantitative, ordinal 0–6)  
   - **Derived From:** `OUTAGE.START` timestamp, which returns 0 for Monday through 6 for Sunday.  
   - **Why:** Day‐of‐week can influence crew staffing and resource availability. For example, outages beginning on a Monday may be addressed more quickly than those on a Saturday because more crews and parts suppliers are fully operational during the workweek.

6. **`is_weekend`** (binary categorical)  
   - **Derived From:** `dow`,  assigning 1 for weekend outages, 0 otherwise.  
   - **Why:** Weekend outages often face slower mobilization—fewer on‐call crews, reduced staffing, and limited access to external contractors, so a binary flag allows the model to adjust its duration prediction when an outage starts on a Saturday or Sunday versus a weekday.  

We apply a `ColumnTransformer` with a `OneHotEncoder(handle_unknown="ignore")` to the three categorical features, standardizes the one numeric features (POPDEN_URBAN), and passes the two engineered features (dow, is_weekend) through unchanged. 

Instead of a random forest, we chose a **Histogram-Based Gradient Boosting Regressor** (`HistGradientBoostingRegressor`) within a Pipeline because:

- It handles heavy‐tailed and skewed numerical distributions well without extensive preprocessing.
- It tends to converge faster on large datasets and is robust to outliers.
- It natively supports early stopping and monotonic constraints (future work) if needed.

We then performed a GridSearchCV over these hyperparameters (5‐fold cross‐validation, optimizing for RMSE) over max_iter (200, 400, 600), max_leaf_nodes (15, 31, 63), and learning_rate (0.1, 0.05, 0.01) using 5‐fold cross‐validation on the training set, optimizing for RMSE. Our best hyperparamters were:
- 'hgb__learning_rate': 0.01
- 'hgb__max_iter': 200
- 'hgb__max_leaf_nodes': 15

With that we get this: 

>Filtered‐Data Model Test RMSE = 1399.90 minutes
>Filtered‐Data Model Test R²  = 0.3353

This interactive plot shows predicted vs. actual outage durations on the test set. Each vertical gray line represents the error between the actual value and the model’s prediction.

<iframe
  src="assets/actual_vs_predicted.html"
  width="900"
  height="500"
  frameborder="0"
></iframe>

Here, we can see **improvement** over the baseline since our RMSE decreased and our R² increased. RMSE = 1329.56 (≈ 23.33 hours), meaning its outage‐duration predictions are **off by about 23 hours** on average and its R² of 0.3353 indicates it now **explains 33.53%** of the variance. The most likely reason our R² is **still** low and RMSE high is that **outage durations are still extremely skewed**. This indicates that adding state‐level, population-density, and temporal features—and using a boosted tree with hyperparameter tuning—provides substantial additional predictive power. Since our model improved this shows these efforts had a positive impact on the model, however, since we never reached our goal of R^2 > 0.75 which indicates a good model, this indicates that while the HistGradientBoostingRegressor captures some predictive structure, most of the variation in duration remains unaccounted for.

## Step 8: Fairness Analysis

In this step, we investigate whether our final regression model exhibits disparate performance across two climate regions. Specifically, we compare:

- **Group X:** Outages where `CLIMATE.REGION == "South"`  
- **Group Y:** Outages where `CLIMATE.REGION == "East North Central"`  

Since duration is continuous, we use **Root Mean Squared Error (RMSE)** as our evaluation metric. Ou hypotheses are: 

> **Null hypothesis:** RMSE for South and East North Central are equal—any observed difference arises purely by chance.

> **Alternative hypothesis:** RMSE for South is greater than RMSE_ENC, meaning the model performs worse for the South.

Permutation Test Procedure
1. Calculate the observed RMSE difference on the held‐out test set:  
   - **Δ_obs = RMSE_South − RMSE_ENC**
2. Perform 1,000 permutations:  
   - Shuffle the `CLIMATE.REGION` labels among test examples.  
   - Compute **Δ_perm = RMSE_perm_South − RMSE_perm_ENC** for each shuffled assignment.   
3. The one‐sided p‐value is the fraction of Δ_perm ≥ Δ_obs.  

If p < 0.05, we reject the null hyporthesis and conclude a significant fairness gap; otherwise, we fail to reject the null hypothesis, indicating no evidence that the model’s error is systematically higher for South.  

<iframe
  src="assets/fairness_rmse_hist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

After computing Δ_obs = RMSE_South − RMSE_ENC and running 1 000 label‐permutations to build a null distribution of Δ_perm, we found that Δ_obs fell well within the bulk of Δ_perm and the one‐sided p‐value of **0.4590** which is much greater than 0.05. In other words, we **fail to reject the null hypothesis**. Thus, there is no evidence that our model’s error is significantly higher for South compared to East North Central.