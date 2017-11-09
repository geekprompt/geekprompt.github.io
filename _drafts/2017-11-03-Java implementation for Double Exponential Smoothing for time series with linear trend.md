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

Now in order to assert the correctness of above code, let us compare the result
with the results of equivalent R library which is a very well established library.


As you can see, the result of our Java implementation matches with
