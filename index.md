---
layout: default
title: Power-Outage-Risks-Analysis
---
# Outage Severity Across the U.S.: Predicting Risk Factors Behind Major Power Failures

** Yuxing Liu, Rocio Saez

**Website Link**:

## Step 1: Introduction
- What are the characteristics of major power outages with higher severity? Variables to consider include location, time, climate, land-use characteristics, electricity consumption patterns, economic characteristics, etc. What risk factors may an energy company want to look into when predicting the location and severity of its next major power outage?

- Predict the severity (in terms of number of customers, duration, or demand loss) of a major power outage.
add a duration using the start adn 
single variable analysis : time trend, location and cause 
bivariate analysis outage duration and cause category, how server it is based on the location and duration, how lomg does each causes usually last. see if the states in certain region has a correratltion?  performed grouping with a pivot table, on Climate Region and Cause Category to see which regions experienced severe weather outages the most.
different cause and what's the most likely cause for that region and feedback based on that. (Risk)

show the NaN table. where has the most customer and where they should close.Missingness Dependency 
Cause Category
First, I examine the distribution of Cause Category when Duration is missing vs not missing.

Null Hypothesis: The distribution of Cause Category is the same when Duration is missing vs not missing.
indicating that the missingness of Duration is dependent on Cause Category. Null Hypothesis: The distribution of Month is the same when Duration is missing vs not missing.

Alternate Hypothesis: The distribution of Month is different when Duration is missing vs not missing.

hypothesis testing: cause relate to duration, if it comes from the same distribution 

hypothesies test on if the region impact on if they have the certain cause. 

# Step 2 EDA Data Cleaning
### Column Removal
I removed the first three row of the data that includes the side note information about this dataset and the unit row
The following land-use variables related to total land area and water surface were dropped from the dataset:
The following columns related to state-level economic output (e.g., GSP, utility sector contribution) were dropped:
- PC.REALGSP.STATE, PC.REALGSP.USA, PC.REALGSP.REL, PC.REALGSP.CHANGE
- UTIL.REALGSP, TOTAL.REALGSP, UTIL.CONTRI, PI.UTIL.OFUSA

These variables are valuable for economic resilience or investment-focused studies, but they were not directly relevant to my focus on outage severity and its common causes by region. My analysis emphasizes event characteristics, regional climate, electricity use, and land/population patterns.


- AREAPCT_URBAN, AREAPCT_UC, PCT_LAND, PCT_WATER_TOT, PCT_WATER_INLAND

These variables, while informative, were not directly relevant to my research question about outage severity. Since my focus is on population density, climate, electricity use, and economic characteristics, I retained variables more closely tied to human impact and energy infrastructure rather than surface geography.

Since one of the key goals of this project is to understand the **severity** of major power outages, I focus on three main severity metrics: 

- `CUSTOMERS.AFFECTED`
- `DEMAND.LOSS.MW`
- `OUTAGE.DURATION`

Among these, `OUTAGE.DURATION` (measured in minutes) is a direct indicator of how long an outage lasted — a critical dimension of impact for residents, businesses, and infrastructure. An outage without a known duration cannot be meaningfully compared or modeled for severity.

Therefore, I dropped all rows with missing values in the `OUTAGE.DURATION` column. These rows (58 total) represent cases where the restoration time was either not recorded or not reported, making them unsuitable for this analysis focused on quantifying outage severity.

The power outage start date and time is given by 'OUTAGE.START.DATE' and 'OUTAGE.START.TIME'. I convert these two columns were combined into one pd.Timestamp column. Combine 'OUTAGE.START.DATE' and 'OUTAGE.START.TIME' into a new pd.Timestamp column called 'OUTAGE.START'. Similarly, combine 'OUTAGE.RESTORATION.DATE' and 'OUTAGE.RESTORATION.TIME' into a new pd.Timestamp column called 'OUTAGE.RESTORATION'.

# Step 2 EDA  Univariate Analysis
This bar chart displays the distribution of major power outages across different cause categories. The most common cause by far is severe weather, followed by intentional attacks and system operability disruptions. This highlights the significant impact of natural events and security threats on the power grid. Understanding the dominant causes helps utilities prioritize infrastructure improvements and disaster response strategies.

<iframe
  src="assets/cause_category_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This plot shows the number of major outages by U.S. state from 2000 to 2016. California leads with the highest number of outages, followed by Texas, Michigan, and Washington. States with large populations, varied terrain, or weather exposure tend to experience more frequent outages. The long tail of the distribution shows many states with relatively few major outages, indicating regional variation in outage frequency.
<iframe
  src="assets/outages_by_state.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


### Choropleth Map of Power Outages by State
This interactive map shows the number of major power outages by U.S. state from 2000–2016. Darker shades indicate states with more frequent outages.
<iframe
  src="assets/outages_choropleth_map.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Outage Duration by Climate Region
Are certain climate regions more prone to severe outages (longer duration or more affected customers)?
<iframe
  src="assets/outage_by_climate_region.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This box plot shows how outage duration varies across different U.S. climate regions. The East North Central and Northeast regions display longer and more variable outage durations, as seen from the wider spread and presence of high outliers. This suggests that regional climate factors—like extreme snow or storms—may affect how long it takes to restore power.

### Urban Population % vs. Customers Affected
Is there a relationship between % urban population (POPPCT_URBAN) and severity?
<iframe
  src="assets/urban_vs_customers.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
This scatter plot explores the relationship between how urban a state is and how many customers are affected during major outages. While most outages cluster below 1 million affected customers, there is a noticeable upward spread in highly urbanized states (above 80% urban population), suggesting denser regions might be more vulnerable to large-scale service disruption.

### Outage Duration by Month
Do outages in summer or winter tend to be longer or larger?

<iframe
  src="assets/duration_by_month.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This box plot breaks down outage duration by month to reveal potential seasonal effects. August, September, and October show slightly higher median durations and more high-end outliers, which may correspond to peak storm or hurricane seasons. These findings suggest that certain months may require increased readiness and response resources.

### Average Outage Duration by Climate Region and Cause
| CLIMATE.REGION     |   equipment failure |   fuel supply emergency |   intentional attack |   islanding |   public appeal |   severe weather |   system operability disruption |
|:-------------------|--------------------:|------------------------:|---------------------:|------------:|----------------:|-----------------:|--------------------------------:|
| Central            |               322   |                 10035.2 |                346.1 |       125.3 |          1410   |           3250   |                          2695.2 |
| East North Central |             26435.3 |                 33971.2 |               2376   |         1   |           733   |           4434.8 |                          2610   |
| Northeast          |               215.8 |                 14629.6 |                196   |       881   |          2655   |           4429.9 |                           773.5 |
| Northwest          |               702   |                     1   |                373.8 |        73.3 |           898   |           4838   |                           141   |
| South              |               295.8 |                 17482.5 |                325.6 |       493.5 |          1164   |           4391.3 |                           866.1 |
| Southeast          |               554.5 |                   nan   |                504.7 |       nan   |          2865.4 |           2662.6 |                           169.3 |
| Southwest          |               113.8 |                    76   |                265.7 |         2   |          2275   |          11572.9 |                           329.2 |
| West               |               524.8 |                  6154.6 |                857.7 |       214.9 |          2028.1 |           2928.4 |                           363.7 |
| West North Central |                61   |                   nan   |                 23.5 |        68.2 |           439.5 |           2442.5 |                           nan   |

This pivot table reveals clear regional patterns in how long major power outages last, depending on their cause. For instance:

Severe weather causes especially long outages in the Southwest and Northwest (11,572.9 and 4,838.0 minutes, respectively).

Fuel supply emergencies lead to very prolonged outages in East North Central and South.

Outages due to equipment failure are unusually high in East North Central, potentially pointing to aging or fragile infrastructure.

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
