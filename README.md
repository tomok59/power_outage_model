# Power Outage Model

by Tom Hocquet and Julia Ma

## Framing the Problem

---

> Introduction 

Electricity is essential for daily life, as it is used to maintain many modern-day necessities such as machinery, electronics, public transportation systems, etc. It serves as a basis for food production, clean water, sanitation, education services, health care, social services and other fundamental human needs.  When power outages occur, these fundamental human needs become at risk and poses a great issue. 

In this project, we use this [dataset](https://www.sciencedirect.com/science/article/pii/S2352340918307182#bib6) that reports on all the major power outages in the U.S. from 2000 to 2016. Specifically, we want to use the dataset to create a classification model that will aid us in predicting power outages, specifically predicting the cause type of the power outage.

> Classification Model 

We want to attempt to predict the `CAUSE.CATEGORY` using other features found in the dataset (more information on the specific features will be revealed in later parts). Since we are predicting discrete, categorical variables in the `CAUSE.CATEGORY` column, we will be using classification to model our predictions. 

> Multiclass Classification

We will be performing a multiclass classification, since the observations in `CAUSE.CATEGORY` can be classified into 7 different categories (as shown in the value counts series below). 

> Response Variable

The response variable we chose to predict is `CAUSE.CATEGORY`, because it had the most complete data to work with, with 0 missing values. We were also more interested in using classification rather than regression. 

> Model Metric 

The F1-score is the metric we used to evaluate our model. Since the `CAUSE.CATEGORY` column is less balanced and more skewed (as shown in the value counts below), the F1-score is more suitable to evaluate the potential false postives and negatives. The F1-score also considered precision and recall

#### Data Cleaning

_*Note_: Since we are using the same dataset from a previous analysis [project](https://tomok59.github.io/power_outages/), we started off using the same data cleaning process found in Project 3, such as dropping uncessary rows, and converting to the correct dtypes and datetimes of certain columns. 

We also performed the additional data cleaning below: 

> Converting OUTAGE.START and OUTAGE.RESTORATION.DATE

`OUTAGE.START.DATE`, `OUTAGE.START.TIME`, `OUTAGE.RESTORATION.DATE`, `OUTAGE.RESTORATION.TIME` are all datetime columns. In order to incorporate them into our model, we decided to convert `OUTAGE.START.TIME` and `OUTAGE.RESTORATION.TIME` to only include the hour the power outage started and was restored. These are now int columns, and the columns were respectively renamed to `START.HOUR` and `END.HOUR`. We also converted `OUTAGE.START.DATE` and `OUTAGE.RESTORATION.DATE` to only include the day of the month the power outage started and was restored. These are also now int columns, and the columns were respectively renamed to `START.DAY` and `END.DAY`. The new columns are shown below:

|    |   START.DAY |   START.HOUR |   END.DAY |   END.HOUR |
|---:|------------:|-------------:|----------:|-----------:|
|  0 |          01 |           17 |        03 |         20 |
|  1 |          11 |           18 |        11 |         18 |
|  2 |          26 |           20 |        28 |         22 |
|  3 |          19 |           04 |        20 |         23 |
|  4 |          18 |           02 |        19 |         07 |

> Filtering Features 

Since we can't choose features that we wouldn't know during the "time of prediction", which in our case means during the time the outage's `CAUSE.CATEGORY` was reported, we must do additional data cleaning to take out those features from the dataset we'll use to train our models. 

The following features cannot be used with the following reasons:

- `OBS`: This column is redundant to the index column. 
- `POSTAL.CODE`: Since we've decided to include the `U.S._STATE`, `POSTAL.CODE` is redundant. 
- `CAUSE.CATEGORY.DETAIL`: This feature can only be recorded after reporting the `CAUSE.CATEGORY`.
- `HURRICANE.NAMES`: This feature can only be recorded after reporting the `CAUSE.CATEGORY`.

> Filtering Column NaNs

In the dataset, we decided to convert the NaNs in every column except `CLIMATE.REGION`, `ANOMALY.LEVEL`, and `CLIMATE.CATEGORY` to -1 to analyze if the missingness in some columns will affect the outcome of our classiifcation model. We decided to use the value of -1 to fill in the NaNs, since every column except `CLIMATE.REGION`, `ANOMALY.LEVEL`, and `CLIMATE.CATEGORY` do not contain negative values, since it wouldn't make sense in the context of the columns. 

Finally, we removed the rows that contained NaNs from `CLIMATE.REGION`, `ANOMALY.LEVEL`, and `CLIMATE.CATEGORY` columns. This removed 14 rows, which is less than 1% of our total rows. Our dataset has no NaNs now!

> Final Cleaned Dataset

Below shows the head of the dataset:

|    |   YEAR |   MONTH | U.S._STATE   | NERC.REGION   | CLIMATE.REGION     |   ANOMALY.LEVEL | CLIMATE.CATEGORY   | CAUSE.CATEGORY     |   OUTAGE.DURATION |   DEMAND.LOSS.MW |   CUSTOMERS.AFFECTED |   RES.PRICE |   COM.PRICE |   IND.PRICE |   TOTAL.PRICE |   RES.SALES |   COM.SALES |   IND.SALES |   TOTAL.SALES |   RES.PERCEN |   COM.PERCEN |   IND.PERCEN |   RES.CUSTOMERS |   COM.CUSTOMERS |   IND.CUSTOMERS |   TOTAL.CUSTOMERS |   RES.CUST.PCT |   COM.CUST.PCT |   IND.CUST.PCT |   PC.REALGSP.STATE |   PC.REALGSP.USA |   PC.REALGSP.REL |   PC.REALGSP.CHANGE |   UTIL.REALGSP |   TOTAL.REALGSP |   UTIL.CONTRI |   PI.UTIL.OFUSA |   POPULATION |   POPPCT_URBAN |   POPPCT_UC |   POPDEN_URBAN |   POPDEN_UC |   POPDEN_RURAL |   AREAPCT_URBAN |   AREAPCT_UC |   PCT_LAND |   PCT_WATER_TOT |   PCT_WATER_INLAND |   START.DAY |   START.HOUR |   END.DAY |   END.HOUR |
|---:|-------:|--------:|:-------------|:--------------|:-------------------|----------------:|:-------------------|:-------------------|------------------:|-----------------:|---------------------:|------------:|------------:|------------:|--------------:|------------:|------------:|------------:|--------------:|-------------:|-------------:|-------------:|----------------:|----------------:|----------------:|------------------:|---------------:|---------------:|---------------:|-------------------:|-----------------:|-----------------:|--------------------:|---------------:|----------------:|--------------:|----------------:|-------------:|---------------:|------------:|---------------:|------------:|---------------:|----------------:|-------------:|-----------:|----------------:|-------------------:|------------:|-------------:|----------:|-----------:|
|  0 |   2011 |       7 | Minnesota    | MRO           | East North Central |            -0.3 | normal             | severe weather     |              3060 |               -1 |                70000 |       11.6  |        9.18 |        6.81 |          9.28 |     2332915 |     2114774 |     2113291 |       6562520 |      35.5491 |      32.225  |      32.2024 |     2.30874e+06 |          276286 |           10673 |       2.5957e+06  |        88.9448 |        10.644  |       0.411181 |              51268 |            47586 |          1.07738 |                 1.6 |           4802 |          274182 |       1.75139 |             2.2 |  5.34812e+06 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |          01 |           17 |        03 |         20 |
|  1 |   2014 |       5 | Minnesota    | MRO           | East North Central |            -0.1 | normal             | intentional attack |                 1 |               -1 |                   -1 |       12.12 |        9.71 |        6.49 |          9.28 |     1586986 |     1807756 |     1887927 |       5284231 |      30.0325 |      34.2104 |      35.7276 |     2.34586e+06 |          284978 |            9898 |       2.64074e+06 |        88.8335 |        10.7916 |       0.37482  |              53499 |            49091 |          1.08979 |                 1.9 |           5226 |          291955 |       1.79    |             2.2 |  5.45712e+06 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |          11 |           18 |        11 |         18 |
|  2 |   2010 |      10 | Minnesota    | MRO           | East North Central |            -1.5 | cold               | severe weather     |              3000 |               -1 |                70000 |       10.87 |        8.19 |        6.07 |          8.15 |     1467293 |     1801683 |     1951295 |       5222116 |      28.0977 |      34.501  |      37.366  |     2.30029e+06 |          276463 |           10150 |       2.58690e+06 |        88.9206 |        10.687  |       0.392361 |              50447 |            47287 |          1.06683 |                 2.7 |           4571 |          267895 |       1.70627 |             2.1 |  5.3109e+06  |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |          26 |           20 |        28 |         22 |
|  3 |   2012 |       6 | Minnesota    | MRO           | East North Central |            -0.1 | normal             | severe weather     |              2550 |               -1 |                68200 |       11.79 |        9.25 |        6.71 |          9.19 |     1851519 |     1941174 |     1993026 |       5787064 |      31.9941 |      33.5433 |      34.4393 |     2.31734e+06 |          278466 |           11010 |       2.60681e+06 |        88.8954 |        10.6822 |       0.422355 |              51598 |            48156 |          1.07148 |                 0.6 |           5364 |          277627 |       1.93209 |             2.2 |  5.38044e+06 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |          19 |           04 |        20 |         23 |
|  4 |   2015 |       7 | Minnesota    | MRO           | East North Central |             1.2 | warm               | severe weather     |              1740 |              250 |               250000 |       13.07 |       10.16 |        7.74 |         10.43 |     2028875 |     2161612 |     1777937 |       5970339 |      33.9826 |      36.2059 |      29.7795 |     2.37467e+06 |          289044 |            9812 |       2.67353e+06 |        88.8216 |        10.8113 |       0.367005 |              54431 |            49844 |          1.09203 |                 1.7 |           4873 |          292023 |       1.6687  |             2.2 |  5.48959e+06 |          73.27 |       15.28 |           2279 |      1700.5 |           18.2 |            2.14 |          0.6 |    91.5927 |         8.40733 |            5.47874 |          18 |           02 |        19 |         07 |


## Baseline Model

---

> Model Description

Our baseline model uses K-Nearest Neighbors (KNN) algorithm to analyze the following features on `CAUSE.CATEGORY`: `OUTAGE.DURATION`, `YEAR`, `MONTH`, `U.S._STATE`, `TOTAL.PRICE`, `TOTAL.SALES`, `TOTAL.CUSTOMERS`, `AREAPCT_URBAN`. 

> Feature Descriptions 

Here are the descriptions for each feature (as taken from the dataset [website](https://www.sciencedirect.com/science/article/pii/S2352340918307182)):
- `OUTAGE.DURATION`: Duration of outage events (in minutes)
- `YEAR`: Indicates the year when the outage event occurred
- `MONTH`: Indicates the month when the outage event occurred
- `U.S._STATE`: Represents all the states in the continental U.S.
- `TOTAL.PRICE`: Average monthly electricity price in the U.S. state (cents/kilowatt-hour)
- `TOTAL.SALES`: Total electricity consumption in the U.S. state (megawatt-hour)
- `TOTAL.CUSTOMERS`: Annual number of total customers served in the U.S. state
- `AREAPCT_URBAN`: Percentage of the land area of the U.S. state represented by the land area of the urban areas (in %)

Here are the labels for the data found in each feature: 
- Quantitative: `OUTAGE.DURATION`, `TOTAL.PRICE`, `TOTAL.SALES`, `TOTAL.CUSTOMERS`, `AREAPCT_URBAN` 
- Nominal: `U.S._STATE`
- Ordinal: `YEAR`, `MONTH` 

_*Note_: Although `YEAR` and `MONTH` can argue to be either quantative or categorical, in the context of our classifier model, we are treating `YEAR` and `MONTH` as categorical variables, therefore they are considered to be categorical and ordinal.

Overall, there are 5 quantitative variables, 1 nominal variable, and 2 ordinal variables. 

> Feature Transformations

For this model, we One-Hot Encoded the categorical features `U.S._STATE`, `YEAR`, and `MONTH` to convert them to numerical form using `OneHotEncoder()`. One-Hot Encoding is when we convert each unique value in the categorical feature into a binary, with 1 representing the feature is present and 0 representing the feature is not.

We also used standardized the quantiative feature `TOTAL.SALES` using `StandardScaler()`. This standardizes the total electricity consumption reported in `TOTAL.SALES` by using the z-score (a.k.a. removing the mean and scaling to a unit variance). We decided to standarize because we believe the distribution of `TOTAL.SALES` is not normal. 

> Model Performance

To assess our KNN model performance, we decided to use a Confusion Matrix to analyze the amount of false positives and false negatives our model got. Overall, our model had a 66.32% training accuracy and a 63.95% testing accuracy.

Based on our training and testing accuracy, our current model is not that good. Both the training and testing accuracies average around 63 to 66%, which means our model uses the given features to predict the correct `CAUSE.CATEGORY` only a bit more than half the time. We decided that a model with 85% accuracy or above is considered to be good, so our current model falls below the threshold and is not considered adequate enough. 

To describe more on the Confusion Matrix shown below, our Confusion Matrix reported our model to have a precision score of 0.638 and a recall score of 0.639. There are 243 total true positives (TP) and 108 false positives (FP).

<iframe src="assets/confusion_matrix_knn.png" width=800 height=600 frameBorder=0></iframe>

## Final Model

---

Below is a graph of F1 train error and F1 test error based on max_depth. We picked a max_depth of 15 as it has the lowest validation error.
<iframe src="assets/max_depth.html" width=800 height=600 frameBorder=0></iframe>


## Fairness Analysis

For our fairness analysis we decided to go with F1 score as it incoporates the fact that our data is uneavenly distributed. We picked the `CAUSE.CATEGORY` 


We create a null hypothesis that our F1 score for each classifier group is the same. We create the alternate hypothesis that the F1 score for each group is not the same. 