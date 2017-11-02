---
layout: post
title: "Java implementation for Double Exponential Smoothing for time series with linear trend"
date: 2017-11-03 01:48:23 +0530
categories: [machine-learning, database]
comments: true
---

[Exponential Smoothing](https://en.wikipedia.org/wiki/Exponential_smoothing target="_blank") is one of the popular statistical methods for smoothing and forecasting [time series](https://en.wikipedia.org/wiki/Time_series target="blank") data.

This post won't cover much theoretical details. You can follow Wikipedia links
inline with above sentence or [this](https://www.otexts.org/book/fpp target="_blank") book for even more detailed understanding.

This post will mainly focus on [Double Exponential Smoothing](https://en.wikipedia.org/wiki/Exponential_smoothing#Double_exponential_smoothing target="_blank") method for Smoothing and forecasting of non-seasonal time series with linear trend. [This](https://www.otexts.org/fpp/6/1 target="_blank") section from the above mentioned book very well explains the components of time series like trend , seasonality etc. There are multiple
algorithms for Double Exponential Smoothing. However, in this post we will
focus on Holt-Winters method for Double Exponential Smoothing.

Statistical languages like [R](https://www.r-project.org/ target="_blank") contains
[forecast package](https://cran.r-project.org/web/packages/forecast/index.html target="_blank")
which provide several handy tools for time series analysis and forecasting.
However, I found it difficult to get some reliable Java library for the same.

Following is a Java implementation for Holt-Winters Double Exponential Smoothing
method:


Now in order to assert the correctness of above code, let us compare the result
with the results of equivalent R library which is a very well established library.


As you can see, the result of our Java implementation matches with
