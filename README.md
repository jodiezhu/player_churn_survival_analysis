# Introduction

I recently ran across [What makes predicting customer churn a challenge](https://medium.com/@b.khaleghi/what-makes-predicting-customer-churn-a-challenge-be195f35366e) for churn prediction. Since I do a lot of work with churn data and churn is fundamentally a time-to-event problem, I decided to check them out. I have decided to demo with the most well-known survival analysis models with the dummy player dataset.


# The problem

Suppose you work on a Top 10 mobile game. You and your producers have concerned why caused some of the players leave the game and never come back. You want to know how long your players are likely to stay with the game and whether players with a certain demographic profile tend to churn more slowly.


# Kaplan-Meier Estimators

Kaplan-Meier (KM) estimators predict survival probabilities over a given period of time for “right-censored” data. “Right-censored” just means that some of the observations in the data weren’t observed for as long as the period the researcher is interested in analyzing. (For example, we want to look at a year of churn, but some of our players installed a week ago). Kaplan-Meier estimators reliably incorporate all available data at each individual time interval to estimate how many observations are still “surviving” at that time.

To do simple survival analysis using these estimators, all you need is a table of players with a binary value indicating whether they’ve churned, and a “follow-up time.” If the player churned, it’s the number of days (or weeks, months) between the day they installed and the day they quit (never come back since then). Otherwise, it’s just the number of days between the day they installed and censored (or the day the data was pulled).


# Create Dummy Dataset

For this post, we’ll first create a dummy dataset as an example. The data includes player's demographic information (e.g. age and gender), follow-up time, a churn binary. There are rules to create these dummy variables. For example, follow-up time usually is in a exponential distribution. I find there are more female players in our game. Therefore, a non-uniform random sample with certain probability in each gender is more apporiate. Age follows the same rules as well.

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

The first few observations. Note only the second player is not churned for a follow-up time of 30, while the other were all churned with a follow-up time less than 30. It seems like the churned rate is pretty high based on the first few observations.

```python
from lifelines import KaplanMeierFitter

kmf = KaplanMeierFitter()
kmf.fit(df["Time"], df["Churned"])
kmf.plot()
```

# Looking for Trends
We’re really interested in is understanding and analyzing churn. We want to know what makes a player more likely to churn, and what causes them to stick around.

One easy way to do that is to create different Kaplan-Meier survival curves for each subset of players you want to look at. The statistical significance of the differences can be tested in many ways, including the Log-Rank test, which we’ll apply below. The Log-Rank test simply evaluates whether the underlying population survival curves for the two sampled groups are likely to be the same. The p-value is essentially the probability that the curves are the same, so statistical significance (I’ll use p < .05) is good!

```python
#Multiple groups
groups = df['Female']
ix = (groups == 1)

kmf.fit(df["Time"][~ix], df["Churned"][~ix], label='Male')
ax2 = kmf.plot()

kmf.fit(df["Time"][ix], df["Churned"][ix], label='Female')
ax_gender =kmf.plot(ax=ax2)
ax_gender.get_figure().savefig("km_plot2.png")


#log rank
from lifelines.statistics import logrank_test

a=logrank_test(df["Time"][ix], df["Time"][~ix], alpha=0.95)
print a
```
```python
<lifelines.StatisticalResult: 

df=1, alpha=0.95, t_0=-1, null_distribution=chi squared

test_statistic      p   
        0.0012 0.9723
```

# Cox Regression

```python
from lifelines import CoxPHFitter

# Using Cox Proportional Hazards model
cph = CoxPHFitter()
cph.fit(df, duration_col="Time", event_col="Churned")

cph.print_summary()  # access the results using cph.summary
```
```python
n=500, number of events=416

          coef  exp(coef)  se(coef)       z      p  lower 0.95  upper 0.95   
Age    -0.0069     0.9931    0.0036 -1.9429 0.0520     -0.0139      0.0001  .
Female -0.0100     0.9900    0.1149 -0.0872 0.9305     -0.2353      0.2153   
```
