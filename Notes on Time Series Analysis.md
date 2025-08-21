# Notes on Time Series Analysis

Notes I made through watching the following YouTube videos and doing my own practice. 

ritvikmath: stock forecasting with GARCH https://www.youtube.com/watch?v=NKHQiN-08S8

Egor Howell: Time Series Crash Course  https://www.youtube.com/watch?v=i7HARZlJv7Y&list=PLKmQjl_R9bYd32uHImJxQSFZU5LPuXfQe

## Stationarity

Augmented Dickey-Fuller (ADF) test

## Exponential Smoothing Models

Exponential smoothing models include simple exponential smoothing, Holt's linear trend and Holt-Winters model. The handles different aspects of time series data. 

| Model                        | Handles                   | use case example                      |
| ---------------------------- | ------------------------- | ------------------------------------- |
| Simple exponential smoothing | Level                     | Flat, stable series                   |
| Holt's linear trend          | Level, trend              | Trending data                         |
| Holt-Winters model           | Level, trend, seasonality | Seasonal + trend (e.g. monthly sales) |

## Residual Analysis

Residual analysis can identify *autocorrelation* and *bias*, help diagnose the problem with the current model and improve forecast through the next iteration. 

### Methods

- histogram plot: check if there is bias
- ACF and PACF plot
- Ljung-box test: reject null (independently distributed, no autocorrelation) if p-value <= 0.01, 0.05, 0.1 

# Implementing Forecasting models

1. turn series into stationary, e.g. using box-cox transform
2. apply forecast model to the stationary series
3. apply inverse box-cox transform to get the predicted series 

### Forecasting models

| Model                         | Components                                                | Comparison                                                   | Base model               |
| ----------------------------- | --------------------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
| Autoregressive                |                                                           |                                                              |                          |
| Moving Average                |                                                           |                                                              |                          |
| ARIMA                         | Autoregression, integrated (differencing), moving average | lacking awareness of any seasonality                         | Regression               |
| SARIMA                        | add seasonallity to ARIMA                                 | cannot model multiple seasonality; cannot handle non-integer seasons (e.g. yearly 365.25 days) | regression               |
| GARCH                         |                                                           |                                                              |                          |
| Dynamic Harmonic Regression   |                                                           |                                                              |                          |
| Long Short-Term Memory (LSTM) | Sigmoid and Tanh activation function                      | can work with non-stationary data                            | Recurrent neural network |
|                               |                                                           |                                                              |                          |

Steps to implement an ARIMA model in Python

1. determine d: take differencing, and check ADF test until stationary 
2. after taking differencing (mean constant), need to take box-cox transform to make the variance constant
3. determine p, q using PACF and autocorrelation graphs; or iterate through choices and choose the one with best AIC/BIC

4. call the package which apply the model for us



GARCH model is used to predict volatility. It well suits the case when we need to predict the volatility of the data, for example, stock returns. It can potentially guide us to make buy decisions. So rather than apply it to price data, I applied it to returns in my code. Notice that usually not all coefficients will be significant, whether we will choose the order depends on specific cases. For example, if $/beta_1$ is not siginificant but $\beta_2$ is significant, this means our choice of order 2 is appropriate. 



## Notes on Harmonic regression

Harmonic regression with Fourier series assumes that the target variable exhibits smooth, repeating cycles over time. However, stock prices are typically non-stationary, highly volatile, and often driven by irregular external shocks. As a result, this model fails to capture the rapid fluctuations and nonlinear trends in the data—leading to flat or unrealistic forecasts, as shown in the plot.

Additionally, the use of `auto_arima` relies on the assumption of **linear relationships**, while the **Box-Cox transformation** assumes the transformed series is approximately **Gaussian**. These assumptions generally **do not hold** for financial time series, particularly when there is **volatility clustering** or complex, non-linear dynamics in the price movements.



## Notes on LSTM

LSTM utilizes the long-term memory and short-term memory. It uses the short-term memory and the input to decide what percentage of the long-term memory will be remembered. The short-term memory and input are also used to create potential long-term memory. This will update the long-term memory. At the last step, a new short-term memory is created and is the output of the LSTM unit. The LSTM algorithm will keep rotating through new LSTM unit with new input from the time series.

#### 🔍 Challenge Encountered

During the development of the LSTM model for stock price prediction, I initially observed that the model's predictions deviated significantly from the actual values — even within the training set.

#### 🛠️ Troubleshooting Process

- Verified the data preprocessing steps, including normalization with `MinMaxScaler` and the inverse transformation. No issues were found.
- Reviewed the sequence generation logic and LSTM model architecture.
- Experimented with different hyperparameters and model settings.
- Discovered that the number of training epochs was too low for the model to effectively learn the underlying patterns.

#### ✅ Solution

By increasing the number of training epochs, the model’s performance improved significantly. The predictions aligned more closely with actual prices, and the training error was notably reduced.





# Summary

Models like ARIMA, Harmonic did not perform well on stock data. Here I summarize the data each model is best at predicting. 

![截屏2025-07-29 16.20.06](/Users/ye/Library/Application Support/typora-user-images/截屏2025-07-29 16.20.06.png)