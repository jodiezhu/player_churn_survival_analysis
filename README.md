# Introduction

I recently ran across [What makes predicting customer churn a challenge](https://medium.com/@b.khaleghi/what-makes-predicting-customer-churn-a-challenge-be195f35366e) for churn prediction. Since I do a lot of work with churn data and churn is fundamentally a time-to-event problem, I decided to check them out. I have decided to demo with the most well-known survival analysis models with the dummy player dataset.


# The problem

Suppose you work on a Top 10 mobile game. You and your producers have concerned why caused some of the players leave the game and never come back. You want to know how long your players are likely to stay with the game and whether players with a certain demographic profile tend to churn more slowly.


# Kaplan-Meier Estimators

Kaplan-Meier (KM) estimators predict survival probabilities over a given period of time for “right-censored” data. “Right-censored” just means that some of the observations in the data weren’t observed for as long as the period the researcher is interested in analyzing. (For example, we want to look at a year of churn, but some of our players installed a week ago). Kaplan-Meier estimators reliably incorporate all available data at each individual time interval to estimate how many observations are still “surviving” at that time.
