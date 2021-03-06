---
layout: post
title: "A minimal Java implementation for Double Exponential Smoothing for forecasting time series with linear trend"
date: 2017-11-12 19:57:23 +0530
tags: [statistics, java, R]
comments: true
---

[Exponential Smoothing](https://en.wikipedia.org/wiki/Exponential_smoothing){:target="_blank"} is one of the popular statistical methods for smoothing and forecasting [time series](https://en.wikipedia.org/wiki/Time_series){:target="_blank"} data.

This post won't cover much theoretical details. You can follow Wikipedia links
inline with above sentence or [this](https://www.otexts.org/book/fpp){:target="_blank"} book for more detailed understanding of time series forecasting.

This post will mainly focus on [Double Exponential Smoothing](https://en.wikipedia.org/wiki/Exponential_smoothing#Double_exponential_smoothing){:target="_blank"} method for Smoothing and forecasting of non-seasonal time series with linear trend. [This](https://www.otexts.org/fpp/6/1 ){:target="_blank"} section from the above mentioned book very well explains the components of time series like trend , seasonality etc. There are multiple
algorithms for Double Exponential Smoothing. However, this post will
focus on Holt-Winters method for Double Exponential Smoothing for forecasting
time-series with linear trend.

The dataset we will be using is "Air transportation data for all passengers with an Australian airline"
which is same dataset used in examples of [this book](https://www.otexts.org/fpp/6/1 ){:target="_blank"} which I have used for reference.

Following is a minimal Java implementation for the same:

{% highlight java %}
import java.util.Arrays;

public class DoubleExponentialSmoothingForLinearSeries {

    /**
     * Performs double exponential smoothing for given time series.
     * <p/>
     * This method is suitable for fitting series with linear trend.
     *
     * @param data  An array containing the recorded data of the time series
     * @param alpha Smoothing factor for data (0 < alpha < 1)
     * @param beta  Smoothing factor for trend (0 < beta < 1)
     * @return Instance of model that can be used to forecast future values
     */
    public static Model fit(double[] data, double alpha, double beta) {
        validateParams(alpha, beta);                        //validating values of alpha and beta

        double[] smoothedData = new double[data.length];    //array to store smoothed values

        double[] trends = new double[data.length + 1];
        double[] levels = new double[data.length + 1];

        //initializing values of parameters
        smoothedData[0] = data[0];
        trends[0] = data[1] - data[0];
        levels[0] = data[0];

        for (int t = 0; t < data.length; t++) {
            smoothedData[t] = trends[t] + levels[t];
            levels[t+1] = alpha * data[t] + (1 - alpha) * (levels[t] + trends[t]);
            trends[t+1] = beta * (levels[t+1] - levels[t]) + (1 - beta) * trends[t];
        }
        return new Model(smoothedData, trends, levels, calculateSSE(data,smoothedData));
    }

    private static double calculateSSE(double[] data, double[] smoothedData) {
        double sse = 0;
        for (int i = 0; i < data.length; i++) {
            sse+= Math.pow(smoothedData[i] - data[i],2);
        }
        return sse;
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
        System.out.println("Sum of squared error: " + model.getSSE());
    }

    static class Model {
        private final double[] smoothedData;
        private final double[] trends;
        private final double[] levels;
        private final double sse;

        public Model(double[] smoothedData, double[] trends, double[] levels, double sse) {
            this.smoothedData = smoothedData;
            this.trends = trends;
            this.levels = levels;
            this.sse = sse;
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

        public double getSSE() {
            return sse;
        }
    }
}

{% endhighlight %}

Output:

{% highlight text %}
Input values: [17.55, 21.86, 23.89, 26.93, 26.89, 28.83, 30.08, 30.95, 30.19, 31.58, 32.58, 33.48, 39.02, 41.39, 41.6]
Smoothed values: [21.86, 22.0324, 25.487295999999997, 27.546707840000003, 30.2919169536, 30.264652063744002, 31.581654755573762, 32.60479053304791, 33.24065120325508, 32.271719144775695, 33.079257669915705, 33.9608841477572, 34.78026797968434, 40.05450186932028, 43.21902834815622]
Trend: [4.309999999999999, 3.6203999999999987, 3.592815999999999, 3.3372486400000003, 3.2385753856000004, 2.694268673024, 2.4647243428249603, 2.2244595819331585, 1.959693096645493, 1.4715889041246812, 1.360913840960569, 1.2810326137740562, 1.2040911501329044, 1.8824482733834111, 2.0961279742921657, 1.83708343858717]
Level: [17.55, 18.412, 21.894479999999998, 24.2094592, 27.053341568, 27.57038339072, 29.1169304127488, 30.38033095111475, 31.28095810660958, 30.800130240651015, 31.718343828955135, 32.67985153398314, 33.576176829551436, 38.17205359593687, 41.122900373864056, 41.92380566963124]
Forecast: [43.760889108218414, 45.59797254680558, 47.43505598539275, 49.27213942397992, 51.109222862567094]
Sum of squared error: 72.80766062490889

{%endhighlight%}

Now in order to assert the correctness of above code, let us compare the result
with the results of equivalent R library which is a very well established library.

{% highlight R %}

data = c(17.55, 21.86, 23.89, 26.93, 26.89, 28.83, 30.08, 30.95, 30.19, 31.58, 32.58, 33.48, 39.02, 41.39, 41.60);
data_ts = ts(data);   //creating time series from data

require("forecast");  //importing package containing forecasting libraries

// h provided number of future values to forecast
fit <- holt(data_ts, alpha=0.8, beta=0.2, initial="simple", h=5);  
fit$model$state;
fitted(fit);          //gives the smoothened values for input time series
fit$mean;             //gives forecasted values
{%endhighlight%}

Output:
{%highlight R%}
> fit1$model$state;
Time Series:
Start = 0
End = 15
Frequency = 1
          l        b
 0 17.55000 4.310000
 1 18.41200 3.620400
 2 21.89448 3.592816
 3 24.20946 3.337249
 4 27.05334 3.238575
 5 27.57038 2.694269
 6 29.11693 2.464724
 7 30.38033 2.224460
 8 31.28096 1.959693
 9 30.80013 1.471589
10 31.71834 1.360914
11 32.67985 1.281033
12 33.57618 1.204091
13 38.17205 1.882448
14 41.12290 2.096128
15 41.92381 1.837083
> fitted(fit1);
Time Series:
Start = 1
End = 15
Frequency = 1
 [1] 21.86000 22.03240 25.48730 27.54671 30.29192 30.26465 31.58165 32.60479 33.24065 32.27172
[11] 33.07926 33.96088 34.78027 40.05450 43.21903
> fit1$mean;
Time Series:
Start = 16
End = 20
Frequency = 1
[1] 43.76089 45.59797 47.43506 49.27214 51.10922
{% endhighlight %}

As you can see, the results of our Java implementation matches with R's forecast
library results.

You might be wondering whether how I decided the values of _alpha_ and _beta_ in
above implementation. I have taken the same values used as given in the book example.
However, approach for deciding optimum values for alpha and beta is to try out
different values of _alpha_ and _beta_ and compare the [sum of squared error(SSE)](https://en.wikipedia.org/wiki/Residual_sum_of_squares ){:target="_blank"}
for each generated models. Values of _alpha_ and _beta_ with the least SSE is the
most optimum values.

_**References:**_

[Wikipedia - Double exponential smoothing](https://en.wikipedia.org/wiki/Exponential_smoothing#Double_exponential_smoothing ){:target="_blank"}

[Forecasting: principles and practice](https://www.otexts.org/fpp/7/2 ){:target="_blank"}

[Forecast package R](https://cran.r-project.org/web/packages/forecast/index.html ){:target="_blank"}
