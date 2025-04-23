#What is the projected delay of your Delta flight?
### By: Julia Klepko (julialuk@umich.edu)

# Introduction

Every day, millions of flights occur all around the world. More often than not, these flights do not always depart on time. Being influenced by a large amount of factors, delays can often mess with individual schedules and travel plans. This dataset originates from a repository of flights within the United States from 2019 - 2023, found [here](https://www.kaggle.com/datasets/patrickzel/flight-delay-and-cancellation-dataset-2019-2023/versions/6). This analysis only focuses on flights operated by Delta Air Lines. The data was automatically filtered when initially read in, leading to a starting row amount of 3.870,800 rows. The dataset features 32 columns of information, 11 of which will be of use in this analysis:

| Column     | Description |
|:----------:|:------------|
| `FL_DATE`    | YYYY-MM-DD of flight |
| `AIRLINE`    | Airline associated with flight (All Delta Air Lines Inc.) |
| `FL_NUMBER`    | Airline flight number associated for identification purposes |
| `ORIGIN`    | IATA Airport code of departure |
| `ORIGIN_CITY`    | City where origin airport is located |
| `DEST`    | IATA Airport code of arrival |
| `DEST_CITY`    | City where origin airport is located |
| `CRS_DEP_TIME`    | Scheduled departure time of flight (Military TIme) |
| `DEP_TIME`    | Actual departure time of flight (Military Time) |
| `DEP_DELAY`    | Time difference betweeen `CRS_DEP_TIME` and `DEP_TIME` (minutes) |
| `AIR_TIME`    | Time plane is airborne (minutes) |


# Data Cleaning and Exploratory Data Analysis

## Data Cleaning
A significant amount of data cleaning involved the creatiion of additional columns for further computation. The table below describes each added column and its implicaiton. Additional steps are listed as well:


| Column     | Reasoning for Addition/Method of Computation|
|:----------:|:------------|
| `FL_DATE_TIME`    | Combines `FL_DATE` and `FL_TIME` into a YYYY-MM-DD TTTT format. Flights are now able to be sorted in a chronological departure order, and is necessary for calculations depending on previous flights of that day. |
- `FL_DATE` was converted into a datetime object for additional date/time calculations
| `AVG_DELAY_FL_NUM_PER_WEEK`   | Calculates the mean delay for the specific flight (by flight number) over the last 7 days. This is not based off of this calendar week; rather, a rolling prior 7 calendar window to this sepcific flight. This prevents skewed mean calculations that can occur at the beginning of each calendar week. |
- `AVG_DELAY_FL_NUM_PER_WEEK` NaN converted to 0 - only relates if that flight did not exist in the dataset prior to this mention, therefore there was no prior delay to take into consideration. 
| `DELAY_SUM_DAY`    | Total delay so far in that calendar day for that origin airport. Used in model training. |
| `DELAYED`    | Binary column of if the flight was delayed or not; used in other column calculations. |
| `FL_AMT_FROM_DAY_ORIGIN`    | Count of flights out of origin airport for this day, used in other column calculations. |
| `AMT_LATE_PROP`    | Proportion of flights this day that have been delayed, calculted involving the flight date and origin, divided by `FL_AMT_FROM_DAY_ORIGIN`. Used in model training. |
- `AMT_LATE_PROP` NaN converted to 0 - if no flights were delayed . Converted to show first 5 decimal places.


