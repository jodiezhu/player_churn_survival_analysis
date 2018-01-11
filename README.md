# Intro

I recently ran across [Weibull Time-to-event Recurrent Neural Networks](https://ragulpr.github.io/2016/12/22/WTTE-RNN-Hackless-churn-modeling/ "WTTE-RNN Original Post") (WTTE-RNNs from here on out) for survival prediction. These are the brainchild of Egil Martinsson, a master's degree candidate at the Chalmers University of Technology ([here's his thesis](https://ragulpr.github.io/assets/draft_master_thesis_martinsson_egil_wtte_rnn_2016.pdf "Egil Martinsson Thesis")). Since I do a lot of work with churn data and churn is fundamentally a time-to-event problem, I decided to check them out.

Distilling all of the work in the thesis, the original GitHub post, and the example code down to the bare essentials took quite a bit of doing (or maybe I'm just slow). However, I eventually got my head wrapped around the internals, and decided to code up a bare-bones example using Keras. This is that bare-bones example, trained on some [jet engine failure data from Nasa](https://ti.arc.nasa.gov/tech/dash/pcoe/prognostic-data-repository/ "NASA Prognostics Data Repository").

# The idea

The basic idea of the WTTE-RNN network is this: we want to design a model that can look at a timeline of historical features (jet engine sensor readings, customer behavior, whatever) leading up to the present, and predict a _distribution_ describing the likelihood that a particular event (engine failure, churn) will happen as time moves into the future. If the model is good, it will learn to predict a distribution that is weighted closer to the present for samples that are very close to experiencing an event, and predict a much wider distribution for samples that are unlikely to experience an event any time soon.

If you're a graphical person, we want our model to be able to generate something kind of like this:
