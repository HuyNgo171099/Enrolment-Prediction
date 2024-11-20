# Enrollment Prediction with Machine Learning

## Project Overview

Scheduling classrooms at Rotterdam School of Management (RSM), Erasmus University is a challenge, due to growing student numbers and a fixed number of classrooms. Allocation of classrooms to the core MSc courses begins on March 15, when the eventual number of enrolled students is highly uncertain. In order to begin planning, the scheduling team relies on predictions for the number of enrolled students in each MSc program. These predictions are currently made using simple heuristics.

The goal of this project is to make good predictions using data provided by the admissions team. These data describe enrollment offers for the academic years 2020/21–2023/24. The first three years of data (2020/21–AY2022/23) will be used to tune, select, and assess a predictive model. Subsequently, this model will be retrained in order to predict enrollment for the 2023/24 academic year for each MSc program.

## Project Background

The number of students who eventually enroll in each of the MSc programs is unknown as of March 15, when classroom scheduling begins. There are two main reasons for this. One is that for most programs, offers are still being sent out after March 15. The other is that students with offers do not need to decide to enroll until the start of the academic year.

To help the scheduling team begin the process of allocating classrooms to courses in mid-March, the admissions team provides an initial forecast for the number of students who will attend each program at the start of the academic year. This forecast is based on all information available to them at that point in time (the information is integrated into a forecast using a simple heuristic, the details of which are irrelevant to this project). As the academic year approaches and more data becomes available, better forecasts are possible. The focus of this project, however, is on improving the initial forecast on March 15.

The data for this project describe enrollment offers for the academic years 2020/21–2023/24, four years in total. The first three years of data (2020/21–2022/23) will be used to tune, select, and assess a predictive model. After selecting the final model, it will be retrained and used to predict enrollment for the fourth year in the data set (2023/24).

## Data Structure

The data are stored in a data frame called `offers`, which can be accessed by loading the file `offers_censored.RData`. The data have been anonymized to protect student privacy.

Each row describes a single offer of admission for a particular academic year. An applicant receives at most one offer of admission each year, so within a given academic year, each offer describes a different person. The following table describes the columns in `offers`:

| **Variable** | **Type** | **Description** |
| :----------- | :------: | :------------- |
| `Status`     | factor   | Target variable. Binary indicator of whether the student eventually enrolled in the program based on information from OSIRIS. Classes: `Enrolled` (positive class), `Not enrolled`. For `AppYear == 2023` (the prediction year), `Status` is set to `NA` to prevent data leakage. |
| `AppYear`    | integer  | The academic year for the application and offer of admission. 2020 = academic year 2020/21, etc. Values are 2020…2023. |
| `Program`    | factor   | The MSc program in which the student was offered a spot. Pre-masters programs, MSc programs with capped enrollment, and one program with data integrity issues (due to a name change for the program) have been excluded. |
| `AppDate`    | date     | The date on which the admissions office received the student’s application. |
| `OfferDate`  | date     | The date on which the student was offered a spot in the named program. |
| `ResponseDate` | date   | The date on which the student responded to the admissions office (see `Response`) after receiving their offer. If the student never responds, the value of this field is `NA`. |
| `Response`   | factor   | The student’s response to the admissions office after receiving their offer. Although correlated with eventual enrollment status, this response is not binding. Categorical values are `Accepted` indicating intent to enroll, `Declined` and `Deferred` both indicating an intent not to enroll in the upcoming academic year, and Unknown, indicating no response has been received (i.e., if `Response == 'Unknown'` then `is.na(ResponseDate) == TRUE`). |
| `Demo1`-`Demo3` | factor | Demographic information about the student. |
| `Edu1`-`Edu3` | factor  | Information about the student’s educational background as it pertains to their offer of enrollment. |
| `App1`-`App4` | factor | Information about other aspects of the student’s application to RSM. |
| `HowFirstHeard` | factor | The student’s response to the question “How did you first hear about RSM?”, collected at the time of application. |

## Censored 2023 Data

Predictions for enrollment in academic year 2023/24 (hereafter referred to as AY2023) will be made using only the data that were available as of 2023-03-15. For this reason, observations in `offers` that contain information that would not have been available before 2023-03-15 have been censored or dropped entirely. Specifically:

- Applications received, and offers sent *after* 2023-03-14 have been dropped.
- Observations for which the student’s response was received *after* 2023-03-14 have been censored:
    --    The value of `ResponseDate` is set to `NA`
    --    The value of `Response` is set to `Unknown`
- `Status` is set to `NA` for all observations with `AppYear == 2023`.

The result of censoring and dropping observations in AY2023 can be seen in this summary:

| | **AY2020** | **AY2021** | **AY2022** | **AY2023** | 
| --- | :---: | :---: | :---: | :---: |
| Num observations | 2835 | 2986 | 2950 | 2069 |
| `Status` is NA | 0 | 0 | 0 | 2069 |
| Max `AppDate` | 2020-07-20 | 2021-06-18 | 2022-06-08 | 2023-03-01 |
| Max `OfferDate` | 2020-09-23 | 2021-08-30 | 2022-09-08 | 2023-03-14 |
| Max `ResponseDate` | 2020-09-29 | 2021-09-08 | 2022-09-16 | 2023-03-13 |
| `ResponseDate` is NA | 195 | 321 | 183 | 677 |

## Analysis, Validation, Final Training, and Prediction Sets

The relationship between the available data and the information available when making predictions is somewhat complicated. It is recommended to use data from earlier years to train models, and evaluate their predictions against data from the year that immediately follows. It is also necessary to make at least two splits of the data.

### Final Training and Prediction Sets
The first split separates the **final training set** from the **prediction set**. As mentioned, the prediction set includes all observations with `AppYear == 2023`, and nothing else. The final training set might include observations from AY2020, AY2021, and AY2022; or just AY2022; or both AY2021 and AY2022. And it may include only a subset of observations from these years (e.g., only observations that would have been available before March 15 in each year), or it may include all observations. 

### Analysis and Assessment Sets
All of the data before AY2023 is available for tuning and assessing models. This pre-2023 data can be divided into an **analysis set** and an **assessment set**. It is probably straightforward to use observations with `AppYear == 2022` as the assessment set. The assessment set will not be used for anything except assessing and selecting the final model, before retraining the final model using the final training set. It is also possible to use observations with `AppYear <= 2021` for tuning, or only observations from `AppYear == 2021`, or something else entirely. And again, it is also logical to include only a subset of observations for these years.

### Summary of the Data Sets

The following table summarizes the four data sets mentioned so far:

| **Name** | **Purpose** | **Contents** | 
| :--- | :--- | :--- |
| Prediction set | Generating predictions for AY2023 | AY2023 observations that would have been available before 2023-03-15 |
| Final training set | Training the final model to predict the target variable in the prediction set | All or a subset of observations from AY2020–AY2022 |
| Assessment set | Selecting the final model and estimating test error prior to final training and predictions | A subset of observations from AY2020–AY2022, but most likely all or a subset of observations from AY2022 | 
| Analyis set | Tuning and refining a candidate set of models | A subset of observations from AY2020–AY2022, but most likely all or a subset of observations from AY2020–AY2021 |

### Optional Censoring of the Data Sets

Predictions are made on March 15 for offers sent out before March 15. If students who apply after this date are very different from students who apply before this date, then it is logical to perform the analysis using only the information available before March 15 in some or all years. There are many ways to carry out this “censoring” of post-March 15 information. A few options include:

- Use only offers made before March 15 for both training and assessment. If it is believed that students with offers before and after March 15 are very different from each other, and the model is to be assessed in an environment that is similar to how it will be deployed, then this option might make sense.
- Use all available offers in each year to train models, but assess those models using only offers that were made before March 15. If it is believed students who receive offers before and after March 15 are not very different, but the model is to be assessed in an environment that is similar to how it will be deployed, this option might make sense.
- Use all of the available offers for both training and assessment. If it is believed that students receiving offers before and after March 15 are not very different, and it is desirable to use the maximum amount of data, this option might make sense.

**There is no best way to use the available data, make a few assumptions, and split the data accordingly.**

## Project Details

To see the details of the project (the code, the explanations, and the results), please check the markdown file `enrollment_prediction.Rmd`. 

