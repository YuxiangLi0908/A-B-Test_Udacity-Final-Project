# A/B-Test_Udacity-Final-Project

- [1. Introduction](#1-introduction)
- [2. Experiment Overview: Free Trial Screener](#2-experiment-overview-free-trial-screener)
- [3. Metric Choice](#3-metric-choice)
  - [3.1 Invariant Metrics](#31-invariant-metrics)
  - [3.1 Evaluation Metrics](#32-evaluation-metrics)
- [4. Measuring Variability](#4-measuring-variability)
- [5. Sizing](#5-sizing)
- [6. Sanity Check](#6-sanity-check)
  - [6.1 Pageviews](#61-pageviews)
  - [6.2 Clicks](#62-clicks)
  - [6.3 Click-Through-Probability](#63-click-through-probability)
- [7. Effect Size Tests](#7-effect-size-tests)
- [8. Sign Tests](#8-sign-tests)
- [9. Recommendation](#9-recommendation)

## 1 Introduction

This report is the final project of Udacity A/B Testing Course, which helps to test the effects of changing the UI after clicking the "start free trail" button on Udacity website.

## 2 Experiment Overview: Free Trial Screener

At the time of this experiment, Udacity courses currently have two options on the home page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead.

The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

## 3 Metric Choice

We need two kinds of metrics to make sure our experiment is not inherently wrong. One is invariant metrics, which will not change significantly between control and experiment groups. Another is evaluation metrics, which are used to test the effect of the changes we made.

### 3.1 Invariant Metrics

* **Number of cookies:** number of unique cookies to view the course overview page.
* **Number of clicks:** number of unique cookies to click the "Start free trial" button (which happens before the free trial screener is trigger).
* **Click-through-probability:** number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page.


### 3.2 Evaluation Metrics

* **Gross conversion:** number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button.
* **Retention:** number of user-ids to remain enrolled past the 14-day boundary (andthus make at least one payment) divided by number of user-ids to complete checkout.
* **Net conversion:** number of user-ids to remain enrolled past the 14-dayboundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button.

## 4 Measuring Variability

Before collecting experimental data, we need to estimate the standard deviation of our evaluation metrics using historical data. By measuring variability of our metrics, we can then decide how many samples we need to collect.

```python
# Calculate standard deviation of evaluation metrics given a sample of 5000 cookies
baseline = {'cookies': 5000,
            'click': 400,
            'enrollment': 82.5,
            'CTP': 0.08,
            'GC': 0.20625,
            'R': 0.53,
            'NC': 0.109313}
            
std_GC = np.sqrt(baseline['GC']*(1-baseline['GC'])/baseline['click'])
std_R = np.sqrt(baseline['R']*(1-baseline['R'])/baseline['enrollment'])
std_NC = np.sqrt(baseline['NC']*(1-baseline['NC'])/baseline['click'])

[std_GC, std_R, std_NC]
```
```python
[0.020230604137049392, 0.05494901217850908, 0.015601575884425905]
```

## 5 Sizing

In any experiment design, determination of the sample size is a fundamental step. Based on the distribution of the metric, the sample size is a function of some parameters such as alpha, i.e. significance level, beta, false negative probability or type II error, baseline conversion, practical significance and so.

[Online_Sample_Size_Calculator](http://www.evanmiller.org/ab-testing/sample-size.html) is a convenient method to calculate the sample size. Here are the results:

```python
# Sample size using online calculator
sample_size_calculator = {'GC': [25835, round(25835/baseline['CTP']*2)],
                          'R': [39115, round(39115/baseline['GC']/baseline['CTP']*2)],
                          'NC': [27413, round(27413/baseline['CTP']*2)]}

sample_size_calculator = pd.DataFrame.from_dict(sample_size_calculator, orient='index', columns=['Sample_size', 'Number_of_cookies'])
sample_size_calculator
```

|   |Sample_size|Number_of_cookies|
|---|---|---|
|GC|25835|645875|
|R|39115|4741212|
|NC|27413|685325|

We notice that we need at least 4741212 cookies to perform an A/B test on retention. However, we only get 40000 cookies per day, which means it would take us 119 days to collect data. Thus we have to drop this evaluation metric in this experiment. By doing this, we then need only 685325 cookies to perform our test.

Also, we do not want to expose all our traffic to this experiment, because the experiment might cause side-effect on our users. Thus, we choose to divert 50% of our traffic to this experiment. Thus, it will take us roughly a month for this experiment. This is a more realistic duration.

## 6 Sanity Check

### 6.1 Pageviews

### 6.2 Clicks

### 6.3 Click-Through-Probability

## 7 Effect Size Tests

## 8 Sign Tests

## 9 Recommendation
