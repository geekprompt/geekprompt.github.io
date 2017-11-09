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

{% highlight Java%}

import java.util.Arrays;

/**
 * Provides methods to perform Holt-Winter's double exponential smoothing.
 * <p>
 * This algorithm is suitable for non-seasonal time series with linear trends.
 *
 * @author Vatsal Mevada
 */
public class DoubleExponentialSmoothingForLinearSeries {

    /**
     * Performs double exponential smoothing for given time unit using Holt Winter's method.
     * <p/>
     * This method is suitable for fitting series with linear trend.
     *
     * @param data  An array containing the recorded data of the time series
     * @param alpha Smoothing factor for data (0 < alpha < 1)
     * @param beta  Smoothing factor for trend (0 < beta < 1)
     * @return Instance of model that can be used to forecast future values
     */
    public static Model fit(double[] data, double alpha, double beta) {
        validateParams(alpha, beta);    //validating values of alpha and beta

        double[] smoothedData = new double[data.length];    //array to store smoothed values
        smoothedData[0] = data[0];

        double[] trends = new double[data.length+1];
        double[] levels = new double[data.length+1];
        trends[0] = data[1] - data[0];
        levels[0] = data[0];

        for (int t = 0; t < data.length; t++) {
            smoothedData[t] = trends[t] + levels[t];
            levels[t+1] = alpha * data[t] + (1 - alpha) * (levels[t] + trends[t]);
            trends[t+1] = beta * (levels[t+1] - levels[t]) + (1 - beta) * trends[t];
        }
        return new Model(smoothedData, trends, levels);
    }

    private static void validateParams(double alpha, double beta) {
        if (alpha < 0 || alpha > 1) {
            throw new RuntimeException("The value of alpha must be between 0 and 1");
        }

        if (beta < 0 || beta > 1) {
            throw new RuntimeException("The value of beta must be between 0 and 1");
        }
    }

    public static void main(String[] args) {
        double[] testData = {17.55, 21.86, 23.89, 26.93, 26.89, 28.83, 30.08, 30.95, 30.19, 31.58, 32.58, 33.48, 39.02, 41.39, 41.60};

        Model model = DoubleExponentialSmoothingForLinearSeries.fit(testData, 0.8, 0.2);

        System.out.println("Input values: " + Arrays.toString(testData));
        System.out.println("Smoothed values: " + Arrays.toString(model.getSmoothedData()));
        System.out.println("Trend: " + Arrays.toString(model.getTrend()));
        System.out.println("Level: " + Arrays.toString(model.getLevel()));
        System.out.println("Forecast: " + Arrays.toString(model.forecast(5)));
    }

    static class Model {
        private final double[] smoothedData;
        private final double[] trends;
        private final double[] levels;

        public Model(double[] smoothedData, double[] trends, double[] levels) {
            this.smoothedData = smoothedData;
            this.trends = trends;
            this.levels = levels;
        }

        /**
         * Forecasts future values.
         *
         * @param size no of future values that you need to forecast
         * @return forecast data
         */
        double[] forecast(int size) {
            double[] forecastData = new double[size];
            for (int i = 0; i < size; i++) {
                forecastData[i] = levels[levels.length - 1] + (i + 1) * trends[trends.length - 1];
            }
            return forecastData;
        }

        public double[] getSmoothedData() {
            return smoothedData;
        }

        public double[] getTrend() {
            return trends;
        }

        public double[] getLevel() {
            return levels;
        }
    }
}
{%endhightlight%}
Output:

{%highlight java%}
Input values: [17.55, 21.86, 23.89, 26.93, 26.89, 28.83, 30.08, 30.95, 30.19, 31.58, 32.58, 33.48, 39.02, 41.39, 41.6]
Smoothed values: [21.86, 22.0324, 25.487295999999997, 27.546707840000003, 30.2919169536, 30.264652063744002, 31.581654755573762, 32.60479053304791, 33.24065120325508, 32.271719144775695, 33.079257669915705, 33.9608841477572, 34.78026797968434, 40.05450186932028, 43.21902834815622]
Trend: [4.309999999999999, 3.6203999999999987, 3.592815999999999, 3.3372486400000003, 3.2385753856000004, 2.694268673024, 2.4647243428249603, 2.2244595819331585, 1.959693096645493, 1.4715889041246812, 1.360913840960569, 1.2810326137740562, 1.2040911501329044, 1.8824482733834111, 2.0961279742921657, 1.83708343858717]
Level: [17.55, 18.412, 21.894479999999998, 24.2094592, 27.053341568, 27.57038339072, 29.1169304127488, 30.38033095111475, 31.28095810660958, 30.800130240651015, 31.718343828955135, 32.67985153398314, 33.576176829551436, 38.17205359593687, 41.122900373864056, 41.92380566963124]
Forecast: [43.760889108218414, 45.59797254680558, 47.43505598539275, 49.27213942397992, 51.109222862567094]
{%endhighlight%}
Now in order to assert the correctness of above code, let us compare the result
with the results of equivalent R library which is a very well established library.

{%highlight R%}
data = c(17.55,21.86,23.89,26.93,26.89,28.83,30.08,30.95,30.19,31.58,32.58,33.48,39.02,41.39,41.60);
data_ts = ts(data);
require("forecast");
fit <- holt(data_ts, alpha=0.8, beta=0.2, initial="simple", exponential=TRUE, h=5);
fit$model$state;
fitted(fit);
fit$mean;
{%endhighlight%}

Output:
{%highlight R%}

> fit1$model$state;
Time Series:
Start = 0 
End = 15 
Frequency = 1 
          l         b
 0 17.55000 4.3100000
 1 18.53351 2.6949784
 2 21.71589 2.9316139
 3 24.06286 2.6477643
 4 26.87994 2.7299683
 5 27.51067 1.7107781
 6 28.91932 1.5640985
 7 30.17206 1.4129297
 8 31.09490 1.1749901
 9 30.66462 0.3956241
10 31.46139 0.5903854
11 32.45946 0.7883174
12 33.42701 0.8753333
13 37.94346 2.6431104
14 41.20666 2.9441669
15 42.18208 1.9883326
> fitted(fit);
Time Series:
Start = 1 
End = 15 
Frequency = 1 
 [1] 21.86000 22.21022 26.38726 28.90217 32.02668 31.88191 33.10630 34.00151 34.46896 33.23356 33.88749 34.66063 35.38978 40.85454 44.13572
> fit$mean;
Time Series:
Start = 16 
End = 20 
Frequency = 1 
[1] 44.60310 47.24700 50.04763 53.01426 56.15675
> 
{%endhighlight%}
As you can see, the result of our Java implementation matches with
