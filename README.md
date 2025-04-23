What is the projected delay of your Delta flight?
By: Julia Klepko (julialuk@umich.edu)

Provide an introduction to your dataset, and clearly state the one question your project is centered around. Why should readers of your website care about the dataset and your question specifically? Report the number of rows in the dataset, the names of the columns that are relevant to your question, and descriptions of those relevant columns.

# Introduction

Every day, millions of flights occur all around the world. More often than not, these flights do not always depart on time. Being influenced by a large amount of factors, delays can often mess with individual schedules and travel plans. This dataset originates from a repository of flights within the United States from 2019 - 2023, found [here]([https://www.kaggle.com/datasets/patrickzel/flight-delay-and-cancellation-dataset-2019-2023/versions/6]). PrediThis analysis only focuses on flights operated by Delta Air Lines. The data was automatically filtered when initially read in, leading to a starting row amount of 3870800 rows. The dataset features 32 columns of information, 11 of which will be of use in this analysis:

| Column     | Description |
|:----------:|:------------|
| `FL_DATE`    | YYYY-MM-DD of flight |
| `AIRLINE`    | Airline associated with flight (All Delta Air Lines Inc.) |
| `FL_NUMBER`    | Flight number associated with flight |
| `ORIGIN`    | IATA Airport code of departure |
| `ORIGIN_CITY`    | City where origin airport is located |
| `DEST`    | IATA Airport code of arrival |
| `DEST_CITY`    | City where origin airport is located |
| `CRS_DEP_TIME`    | Scheduled departure time of flight (Military Flight) |
| `DEP_TIME`    | Actual departure time of flight (Military Time) |
| `DEP_DELAY`    | Time difference betweeen `CRS_DEP_TIME` and `DEP_TIME` (minutes) |
| `AIR_TIME`    | Time plane is airborne (minutes) |

