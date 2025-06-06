---
layout: default
title: Power-Outage-Risks-Analysis
---
# Outage Severity Across the U.S.: Predicting Risk Factors Behind Major Power Failures

**Yuxing Liu, Rocio Saez**

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

This dataset will help us answer the question: **which combination of factors—specifically outage cause, geographic location, climate region, population density, and time‐of‐year—best predicts how long (in minutes) a major U.S. power outage will last?**


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

After completing these steps, the cleaned DataFrame (`df_clean`) contains only the columns and rows relevant for our exploratory analysis and modeling of outage duration. Below is the head of the cleaned DataFrame for verification:


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

Then, we tried to see if there is a relationship between percentage (%) of urban population (POPPCT_URBAN) and its severity through a **scatterplot**.

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

## Grouping % Aggregates
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
I believe the column CUSTOMERS.AFFECTED is likely Not Missing At Random (NMAR). This is because the decision to leave the value blank may depend on the value itself — for example, if only a small number of customers were affected and the utility deemed it not worth reporting. In such cases, the missingness depends directly on the (unseen) value of CUSTOMERS.AFFECTED, making it NMAR.

To determine if this missingness could instead be MAR, we would need additional metadata about the reporting policies or criteria used by each utility during the event, such as thresholds for whether customer impact was logged.

Does DEMAND.LOSS.MW Missingness Depend on CAUSE.CATEGORY?
We test if the missing values are related to the cause of the outage.

<iframe
  src="assets/demand_missing_permutation.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>
As seen in the histogram (see above), the observed standard deviation is far outside the range of the null distribution, with a p-value < 0.001.

This indicates that missingness in DEMAND.LOSS.MW is dependent on CAUSE.CATEGORY, suggesting Missing At Random (MAR). In other words, the missingness can be explained by another observed column.

Test Missingness of DEMAND.LOSS.MW vs. U.S._STATE
<iframe
  src="assets/time_missingness_permutation.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

We conducted a permutation test to assess whether the missingness in the DEMAND.LOSS.MW column depends on OUTAGE.START.TIME. The test statistic (standard deviation of missingness across times) closely matches the null distribution generated by permutation.

Since the observed value is not extreme and falls well within the range of expected variation under the null hypothesis, we conclude that the missingness does not depend on OUTAGE.START.TIME.

Therefore, with respect to this column, the missingness is consistent with Missing Completely At Random (MCAR).


# Step 4 Hypothesis 
Hypothesis Test: Are summer outages more severe?
Null Hypothesis (H₀): Outages in summer are not longer than outages in other seasons.

Alternative Hypothesis (H₁): Outages in summer have significantly greater durations than those in other seasons.
<iframe
  src="assets/summer_vs_others_boxplot.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>
There is no significant evidence that power outages in summer are longer than those in other seasons. The distribution of durations appears similar across seasons based on the boxplot and statistical test.

<iframe
  src="assets/summer_duration_permutation.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

We fail to reject the null hypothesis. There is no statistically significant evidence that outages in summer are more severe (longer in duration) than those in other seasons.

# Step 5: Framing a Prediction Problem
Identify a prediction problem. Feel free to use one of the example prediction problems stated in the “Example Questions and Prediction Problems” section of your dataset’s description page or pose a hypothesis test of your own. The prediction problem you come up with doesn’t have to be related to the question you were answering in Steps 1-4, but ideally, your entire project has some sort of coherent theme.

Before building any predictive model, we need to turn our descriptive and inferential insights into a concrete supervised‐learning task. Based on our analyses so far—especially the finding in Step 4 that “summer outages tend to be longer,” plus our EDA showing that cause category, climate region, and population density all influence outage length—we will set up a **regression problem**: 
> We will build a regression model that, given (`CAUSE.CATEGORY`, `CLIMATE.REGION`, `MONTH`, `POPDEN_URBAN`), predicts how many minutes an outage will last.  
### Distribution of Outage Durations

This histogram shows the distribution of outage durations (in minutes) across all events in the dataset. Most outages are relatively short, but some last for tens of thousands of minutes, indicating extreme severity in certain cases.

<iframe
  src="assets/outage_duration_hist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>


## Step 6: Baseline Model
We begin with a baseline regression model that uses only the two categorical predictors: `CAUSE.CATEGORY` and `CLIMATE.REGION`. We split the cleaned data (`df_clean`) into 80 % for training and 20 % for testing. A `ColumnTransformer` one‐hot encodes both categorical columns, and a `RandomForestRegressor` (with 100 trees) is fit on the encoded training data. After training, we predict on the held‐out test set and compute test‐set RMSE and R². This baseline establishes a performance floor: it shows how well we can predict outage duration using only cause and region information. Any additional features in later steps should improve upon these baseline metrics.
The baseline model—which used only cause category and climate region—has an RMSE of 7 398 minutes (≈ 123 hours), meaning its outage‐duration predictions are off by about five days on average. This is due to only providing 2 categories. Its R² of 0.108 indicates it explains only 10.8 % of the variance in true outage durations. In other words, **cause and region alone capture very little of the signal, and most of the variability in outage length remains unexplained at this baseline level.** To improve this, we need more information/indicators to reach our target which is what we will do in step 7.


## Step 7: Final Model
Building on the baseline and our ideas to improve our model, we will now include six predictors: CAUSE.CATEGORY, CLIMATE.REGION, U.S._STATE, POPDEN_URBAN, a log‐transformed residential customers column (res_cust_log), and MONTH (encoded as cyclical month_sin/month_cos), along with day‐of‐week (dow) and weekend flag (is_weekend). We begin by dropping the top 10 % of longest outages to reduce extreme‐value influence. Again, we use an 80/20 train/test split. Our ColumnTransformer one‐hot encodes the three categorical features, standardizes the three numeric features (MONTH, POPDEN_URBAN, res_cust_log), and passes the four engineered features (month_sin, month_cos, dow, is_weekend) through unchanged. Instead of a Random Forest, we fit a HistGradientBoostingRegressor within a Pipeline and perform a grid search over max_iter (200, 400, 600), max_leaf_nodes (15, 31, 63), and learning_rate (0.1, 0.05, 0.01) using 5‐fold cross‐validation on the training set, optimizing for RMSE. With that we get this: 

>>Filtered‐Data Model Test RMSE = 1329.56 minutes
>>Filtered‐Data Model Test R²  = 0.4004

Here, we can see improvement over the baseline, indicating that adding state‐level, population-density, customer-count, and temporal features—and using a boosted tree with hyperparameter tuning—provides substantial additional predictive power.


### Final Model Results: Actual vs. Predicted Outage Duration

This interactive plot shows predicted vs. actual outage durations on the test set. Each vertical gray line represents the error between the actual value and the model’s prediction.

<iframe
  src="assets/actual_vs_predicted.html"
  width="900"
  height="500"
  frameborder="0"
></iframe>

Now our model has improved. We achieved an RMSE of 1329.56 minutes (≈ 22.1 hours) and an R² of 0.4004. The most likely reason our R² is **still** low and RMSE high is that **outage durations are still extremely skewed**. In other words, **the model’s predictions are off by roughly 22 hours on average, and it explains only about 40 % of the variance in outage durations**. 

Our process in improving our model was introducing more hyper parameters, attemtping to get rid of extreme outlier, and engineering new temporal features (month_sin/month_cos, day‐of‐week, weekend flag) and transforming residential customer counts via log scaling. Since our model improved this shows these efforts had a positive impact on the model, however, since we never reached our goal of R^2 > 0.75 which indicates a good model, this indicates that while the HistGradientBoostingRegressor captures some predictive structure, most of the variation in duration remains unaccounted for.

## Step 8: Fairness Analysis
Before running the permutation test, we clearly define our groups, metric, and hypotheses for fairness. We compare the model’s performance on two subgroups: outages in `CLIMATE.REGION == "South"` (Group X) versus outages in `CLIMATE.REGION == "East North Central"` (Group Y). Because our task is regression, we use **Root Mean Squared Error (RMSE)** as the evaluation metric. 

> **null hypothesis (H₀):** RMSE for South and East North Central are equal—any observed difference arises purely by chance.

> **alternative hypothesis (H₁):** RMSE for South is greater than RMSE_ENC, meaning the model performs worse for the South.

To test this, we compute the observed difference Δ_obs = RMSE_South − RMSE_ENC on the held‐out test set. We then randomly permute the “region” labels among the test examples 1,000 times, recomputing Δ_perm = RMSE_perm_South − RMSE_perm_ENC for each shuffle to build a null distribution. The one‐sided p‐value is the fraction of Δ_perm ≥ Δ_obs. If p < 0.05, we reject H₀ and conclude a significant fairness gap; otherwise, we fail to reject H₀, indicating no evidence that the model’s error is systematically higher for South.  
<iframe
  src="assets/fairness_rmse_hist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>
Here, we compared RMSE for “South” versus “East North Central” outages. Our **null hypothesis** was that RMSE_South = RMSE_ENC (no difference), and the **alternative hypothesis** was RMSE_South > RMSE_ENC (worse performance for South). After computing Δ_obs = RMSE_South − RMSE_ENC and running 1 000 label‐permutations to build a null distribution of Δ_perm, we found that Δ_obs fell well within the bulk of Δ_perm and the one‐sided p‐value was much greater than 0.05. In other words, the observed RMSE gap could easily occur by chance, so we fail to reject H₀. Thus, there is no evidence that our model’s error is significantly higher for South compared to East North Central.
