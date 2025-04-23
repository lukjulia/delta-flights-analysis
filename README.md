# What is the projected delay of your Delta flight?
### By: Julia Klepko (julialuk@umich.edu)

# Introduction

Every day, millions of flights occur all around the world. More often than not, these flights do not always depart on time. Being influenced by a large amount of factors, delays can often mess with individual schedules and travel plans. This dataset originates from a repository of flights within the United States from 2019 - 2023, found [here](https://www.kaggle.com/datasets/patrickzel/flight-delay-and-cancellation-dataset-2019-2023/versions/6). This analysis only focuses on flights operated by Delta Air Lines. The data was automatically filtered when initially read in, leading to a starting row amount of 3.870,800 rows. The dataset features 32 columns of information, 11 of which will be of use in this analysis:

| Column     | Description |
|:----------:|:------------|
| `FL_DATE`    | YYYY-MM-DD of flight |
| `AIRLINE`    | Airline associated with flight (All Delta Air Lines Inc.) |
| `FL_NUMBER`    | Airline flight number associated for identification purposes |
| `ORIGIN`    | IATA Airport code of departure |
| `ORIGIN_CITY`    | Cite, State where origin airport is located |
| `DEST`    | IATA Airport code of arrival |
| `DEST_CITY`    | City, State where origin airport is located |
| `CRS_DEP_TIME`    | Scheduled departure time of flight (Military TIme) |
| `DEP_TIME`    | Actual departure time of flight (Military Time) |
| `DEP_DELAY`    | Time difference betweeen `CRS_DEP_TIME` and `DEP_TIME` (minutes) |
| `AIR_TIME`    | Time plane is airborne (minutes) |


# Data Cleaning and Exploratory Data Analysis

NOTE: Some visual graphs shown below were completed on a 200,000 random sample of 3,805,204 usuable rows of the dataset (see Imputation). I am currently using an 8gb RAM laptop, and even attempting to reopen the original notebook with the presence of one 3 million datapoint histogram leaves the file unusuable. This random snample was set to a random state for replciability purposes. If a smaller sample was used, it is noted in the title of the graph. 

## Data Cleaning
A significant amount of data cleaning involved the creatiion of additional columns for further computation. The table below describes each added column and its implicaiton. Additional steps are listed as well:

| Column     | Reasoning for Addition/Method of Computation|
|:----------:|:------------|
| `FL_DATE_TIME`    | Combines `FL_DATE` and `FL_TIME` into a YYYY-MM-DD TTTT format. Flights are now able to be sorted in a chronological departure order, and is necessary for calculations depending on previous flights of that day. |
| `AVG_DELAY_FL_NUM_PER_WEEK`   | Calculates the mean delay for the specific flight (by flight number) over the last 7 days. This is not based off of this calendar week; rather, a rolling prior 7 calendar window to this sepcific flight. This prevents skewed mean calculations that can occur at the beginning of each calendar week. |
| `DELAY_SUM_DAY`    | Total delay so far in that calendar day for that origin airport. Used in model training. |
| `DELAYED`    | Binary column of if the flight was delayed or not; used in other column calculations. |
| `FL_AMT_FROM_DAY_ORIGIN`    | Count of flights out of origin airport for this day, used in other column calculations. |
| `AMT_LATE_PROP`    | Proportion of flights this day that have been delayed, calculted involving the flight date and origin, divided by `FL_AMT_FROM_DAY_ORIGIN`. Used in model training. |


- `FL_DATE` was converted into a datetime object for additional date/time calculations
- `AVG_DELAY_FL_NUM_PER_WEEK` NaN converted to 0 - only relates if that flight did not exist in the dataset prior to this mention, therefore there was no prior delay to take into consideration. 
- `AMT_LATE_PROP` NaN converted to 0 - if no flights were delayed . Converted to show first 5 decimal places.
- `DEP_DELAY` and `AIR_TIME` NaN had those flights removed from analysis - see Imputation.


Below is the head of the altered dataframe.

| FL_DATE   | AIRLINE                | FL_NUMBER | ORIGIN | ORIGIN_CITY           | DEST | DEST_CITY             | CRS_DEP_TIME | DEP_TIME | DEP_DELAY | AIR_TIME | FL_DATE_TIME         | AVG_DELAY_FL_NUM_PER_WEEK | DELAY_SUM_DAY | DELAYED | FL_AMT_FROM_DAY_ORIGIN | AMT_LATE_PROP |
|-----------|------------------------|-----------|--------|-----------------------|------|-----------------------|--------------|----------|-----------|----------|----------------------|---------------------------|----------------|---------|------------------------|---------------|
| 2019-01-01| Delta Air Lines Inc.    | 822       | ANC    | Anchorage, AK         | SEA  | Seattle, WA           | 30           | 23.0     | -7.0      | 173.0    | 2019-01-01 30        | 0.0                       | -7.0           | 0       | 3                      | 0.00000       |
| 2019-01-01| Delta Air Lines Inc.    | 1344      | LAX    | Los Angeles, CA       | MSP  | Minneapolis, MN       | 40           | 42.0     | 2.0       | 182.0    | 2019-01-01 40        | 0.0                       | 2.0            | 1       | 82                     | 0.42683       |
| 2019-01-01| Delta Air Lines Inc.    | 688       | FAI    | Fairbanks, AK         | SEA  | Seattle, WA           | 45           | 41.0     | -4.0      | 178.0    | 2019-01-01 45        | 0.0                       | -4.0           | 0       | 1                      | 0.00000       |
| 2019-01-01| Delta Air Lines Inc.    | 2082      | SLC    | Salt Lake City, UT    | ATL  | Atlanta, GA           | 59           | 54.0     | -5.0      | 185.0    | 2019-01-01 59        | 0.0                       | -5.0           | 0       | 95                     | 0.22105       |
| 2019-01-01| Delta Air Lines Inc.    | 1081      | BOI    | Boise, ID             | MSP  | Minneapolis, MN       | 520          | 516.0    | -4.0      | 148.0    | 2019-01-01 520       | 0.0                       | -4.0           | 0       | 2                      | 0.00000       |



## Univariate Analysis 
I was interested in creating a view of the scheduled departure timeacross all flights (`CRS_DEP_TIME`)

 <iframe
 src="assets/uni1.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
This was produced in a 24 bin histogram to see the amount of flights departing with that hour. It is interesting to note the stop of commercial flight departures during the night (most likely for crew to rest and for enviornmental noise), yet a significant amount of flights are in the early hours of the morning. 

This can be compared with the actual departure time across all flights (`DEP_TIME`) as shown below:
 <iframe
 src="assets/uni2.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
This keeps a similar shape as the first histogram, yet the amount of flights is more averaged across the bins, mainly due to the amount of flights that are either slightly delayed or depart a few minutes ahead of schedule. 

## Bivariate Analysis

I wanted to see if there was a visual correclation between the flight delay and the duration that the flight was in the air (`AIR_TIME` and `DEP_DELAY`).
 <iframe
 src="assets/bi1.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
This graph shows soome really unique results, with a large amount of shorter flights leading to large delays. It could be possible that there exists a specific cutoff to cancel any flights longer than 360 minutes (6 hours) if the delay is greater than 2 hours; this can be to have operations be more smooth in the longer distanced flights. 

## Interesting Aggregates
There are a lot of manipulations that can be completed with this data, one of which is the the average flight duration based on origin airport. I decided to aggregate the table by using groupby for both `ORIGIN` and `AIR_TIME`, and then plotting a bar for each airport. 
 <iframe
 src="assets/agg1.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>
The longest flight duration averages were from airports in Hawaii, as most are outbound to the continental United States. This is a useful perspective in recognizing the estimated traffic and uses for the various airports.

## Imputation
Though `DEP_DELAY` and `AIR_TIME` could theoretically have their missigng values imputation, it would be impractical and not accurately possible for this analysis. Due to  `DEP_DELAY` being the prediction column (see Baseline Model), NaN values should not be present in this prediction - attempting to impute averages can lead to the model incorrectly producing predictions. This removed around 57217 flights, or around 1.5% for this analysis. 8,379 `AIR_TIME` flights were removed as well; a visual track of the rows with this missing value was seemingly random and could seem to be easily predictable other than case-by- case. This left 3,805,204 rows for the model analysis.


Clearly state your prediction problem and type (classification or regression). If you are building a classifier, make sure to state whether you are performing binary classification or multiclass classification. Report the response variable (i.e. the variable you are predicting) and why you chose it, the metric you are using to evaluate your model and why you chose it over other suitable metrics (e.g. accuracy vs. F1-score).

Note: Make sure to justify what information you would know at the “time of prediction” and to only train your model using those features. For instance, if we wanted to predict your Final Exam grade, we couldn’t use your Final Project grade, because we (probably) won’t have the Final Project graded before the Final Exam! Feel free to ask questions if you’re not sure.


# Framing a Prediction Problem
As seen from the previous graphs, there are a multitude of components and information that can influence the traffic of airports an flight delays. Projecting the delay of a future Delta flight is a regression problem, as the actual delay is a numerical indicator of the minutes that the flight is delayed; it is not a category of wheteher or not the flight will be delayed as a whole. The prediction problem involes the `DEP_DELAY` variable, as this is the variable that would tell me the amount of minutes that my flight is delayed by.  The metric to evaluate this prediction model is MSE, as it alculates the mean squared difference between the hypothetical flight delay and the actual amount of time the flight was delayed by. A lower MSE will indicate that the model is better at accurately predicting the actual departure delay of the flight. 

# Baseline Model

I will be using the LinearRegression Model as a baseline for my analysis; there are 3 categorical and 3 numerical featuers for each model.

| Feature | Info |
|:----------:|:---------|
| `FL_NUMBER`| Categorical, scaled with OneHotEcncoding() |
| `FL_DATE_TIME`| Categorical, scaled with OneHotEcncoding() |
| `ORIGIN`| Categorical, scaled with OneHotEcncoding() |
| `AVG_DELAY_FL_NUM_PER_WEEK`| Numerical, scaled with StandardScaler()  |
| `AMT_LATE_PROP`| Numerical, scaled with StandardScaler()  |
| `AIR_TIME`| Numerical, scaled with StandardScaler()  |

All cateogirical variables had handle_unknown=’ignore’ and drop=first to tackle issues with multicollinearity
Train-Test Split was then performed on a test size of 0.2, causing 5 splits of data.

MSE Training Set: 1275.8967436033279
MSE Testing Set: 2465.2639053700495

This base set ended up being rather suffiicient, ebign ble to have some basis of the minutes the flight. 

# Final Model

I ended up also running my final model on the LinearRegression Model, adding one additional categorical and one numerical feature.

| Feature | Info|
|:---------:|:---------|
| `DELAY_SUM_DAY`| Categorical, scaled with OneHotEcncoding() |
| `DEST`| Numerical, scaled with StandardScaler() |

I believe `DELAY_SUM_DAY` assists in that if there is a large total amount of delay at a specific airport, that flight is even more likely to be delated , and `DEST` helps account for variance in the model for when a flight number and origin stay the same for example, but the airline later changes the ultimate testination of the flight.  Hyperparameters were not ran due to issues with a low RAM system; see additional notes in notebook for further information. However, the goal was to play around with the degrees to see if LinearRegression() was still optimal.

Train-Test Split was then performed on a test size of 0.2, causing 5 splits of data.
MSE Training Set: 1174.7723011375483
MSE Testing Set: 2460.340202319307

Even with the additional features, the model itself did not improve much; with the baseline model showing to be mostly suffieicent as well. The model already itself is "good", but can be possible better with taking other algorithms into consideration instead, such as possibly DescisionTreeClassifier().
