# Introduction

I recently ran across [What makes predicting customer churn a challenge](https://medium.com/@b.khaleghi/what-makes-predicting-customer-churn-a-challenge-be195f35366e) for churn prediction. Since I do a lot of work with churn data and churn is fundamentally a time-to-event problem, I decided to check them out. I have decided to demo with the most well-known survival analysis models with the dummy player dataset.


# The problem

Suppose you work on a Top 10 mobile game. You and your producers have concerned why caused some of the players leave the game and never come back. You want to know how long your players are likely to stay with the game and whether players with a certain demographic profile tend to churn more slowly.


# Kaplan-Meier Estimators

Kaplan-Meier (KM) estimators predict survival probabilities over a given period of time for “right-censored” data. “Right-censored” just means that some of the observations in the data weren’t observed for as long as the period the researcher is interested in analyzing. (For example, we want to look at a year of churn, but some of our players installed a week ago). Kaplan-Meier estimators reliably incorporate all available data at each individual time interval to estimate how many observations are still “surviving” at that time.

To do simple survival analysis using these estimators, all you need is a table of players with a binary value indicating whether they’ve churned, and a “follow-up time.” If the player churned, it’s the number of days (or weeks, months) between the day they installed and the day they quit (never come back since then). Otherwise, it’s just the number of days between the day they installed and censored (or the day the data was pulled).

For this post, we’ll first create a dummy dataset as an example. The data includes player's demographic information (e.g. age and gender), follow-up time, a churn binary. The first few observations are displayed below. There are rules to create these dummy variables. For example, follow-up time usually is in a exponential distribution. I find there are more female players in our game. Therefore, a non-uniform random sample with certain probability in each gender is more apporiate. Age follows the same rules as well.

```python

import numpy as np

censor_after = 30
actual_lifetimes = np.random.exponential(15, size=500)
observed_lifetimes = np.minimum( actual_lifetimes, censor_after*np.ones(500) )
C = (actual_lifetimes < censor_after) #boolean array
female=np.random.choice([0, 1], 500, p=[1./4, 3./4])

age=np.random.normal(23, 8, 500).round(0)

df = pd.DataFrame({'Female':female,'Age':age,'Time': observed_lifetimes,'Churned': C})
df['Churned'] = np.where(df['Churned']==True, 1, 0)
```
