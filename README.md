# California Housing Price Prediction

**Team members**: Xuxu Pan, Michael Schulze, Jingwen Yu, Ivette Sulca

**Goal**: predict California monthly median sold price from Jan 2016 to Aug 2017.

## Problem Description
### 1.1 Data Description
#### 1.1.1 Data Source: Zillow
#### 1.1.2 Variables: 
![Image description](/tables/1.1.2.png)
#### 1.1.3 Sample rows of the data:
![Image description](/tables/1.1.3.png)
### 1.2 Forecast Goal:
Forecast the monthly median sold price for Jan 2016 to Aug 2017.

### 1.3 Questions to Explore:
Which model can best predict the monthly median sold price?
What’s the pattern of the median sold price?
Which variable is most predictive to the median sold price?

## Description of Methods
We analyzed the target variable MedianSoldPrice and observed that it was not stationary and the time series had a **clear trend** as we can see in the next line, ACF and PACF plots:
![Image description](/tables/2-1.png)
![Image description](/tables/2-2.png)

There is no clear seasonality and based on the original ACF/PACF plots we in fact conclude that there is not. Nevertheless, after differentiating the original data to get rid of the trend, we observed that the new ACF and PACF presente2d a clear seasonality of lag m=12, therefore, we decided to include also a seasonal model into the model candidates.
![Image description](/tables/2-3.png)

In addition, according to the correlation matrix we had three possibly exogenous variables (median mortgage, unemployment rate and monthly rental price) since it seems that there is no relationship between them in order to be considered as endogenous variables. However, in reality, some of those variables might influence each other such as MedianRentalPrice and MedianSoldPrice and for that reason we will also consider a VAR model between the model candidates as well. 
![Image description](/tables/2-4.png)

Finally, with the previous analysis we decided to select the next four model candidates that handle trend, seasonality and multivariate time series and then using RMSE metric to choose the best one.

### 2.1 SARIMA:
Using SARIMA univariate time series modelling, we estimate d and D order after differentiating the original MedianSoldPrice considering the time range 2010-2016 since those are closer to the forecasting time range and doesn’t include the unusual shape of previous years (probably due to economic recession). To get rid of the trend, we found that after differentiating two times the p-value of the ADF test is less than alpha=0.05, then we chose d=2 for the model.

_Results of Dickey-Fuller Test:
Test Statistic                 -3.246561
p-value                         0.017443
Num of Lags Used                     11.000000
Number of Observations Used    58.000000
Critical Value (1%)            -3.548494
Critical Value (5%)            -2.912837
Critical Value (10%)           -2.594129_

As a second step, we observed that the plot had a possible seasonality with lag m=12 based on ACF and PACF, therefore, we decided to differentiate again and found the next stationary line plot with D=1 :
![Image description](/tables/2.1.png)

To estimate the order of P,Q,p and q, we used auto arima which selected as the final model SARIMA (1, 2, 1) x (0, 1, 1, 12). The residuals diagnosis plot showed that residuals of the model satisfy assumptions of zero-mean, constant variance, zero correlation and normality.
![Image description](/tables/2.1-2.png)

Using the model and splitting the train-test data using the 80%-20% rule, we obtain a RMSE of 2511.58 on the validation set, which shows a good performance of the model since the values of median sold prices are around USD 300k.
![Image description](/tables/2.1-3.png)

### 2.2 SARIMAX (with only unemployment variable):
We fitted a reduced SARIMAX model with the UmemploymengRate variable from Jan 2004 to Dec 2015, because for two main reasons. One is that it is the only variable that has a correlation larger than 0.5 with the MedianSoldPrice, the other is that it is the only variable that would not be affected by the MedianSoldPrice(exogenous). MedianMortageRate might be affected by the MedianSoldPrice because high house prices is an indicator for inflation that might cause the federal bank to increase the interest rate. MedianRentalPrice might be affected by MedianSoldPrice because the higher the house prices, the higher the rent. 

Same as the SARIMA model, we estimate d and D order after differentiating the original MedianSoldPrice but with the time range 2004-2016 this time. We chose d=2, D=1 and m=12 according to the p-value of the ADF test, line plot and ACF plot.
Then we fit an auto arima model on MedSoldPrice and decide the order to be **(1, 2, 1) x (0, 1, 1, 12)**. 

We also split the data into training and validation set. Because the data from 2004 has more period than from 2010, and the model needs to more recent data to capture the recent trend, so we made a 10:90 split.

In the last step we fit the SARIMAX model with training set and  forecast the validation set.
![Image description](/tables/2.2.png)
The rmse is around 4312.09, decent but not as good as the SARIMA model.

### 2.3 SARIMAX v2 (imputing missing values):
For the second SARIMAX model, we decided to use all the columns from 2004-2016. 
Since the Median Rental Price data during 2004 to 2009 is missing, we first imputed these missing values using univariate SARIMA model, the order is (2, 1, 3)x(0, 1, 1, 12). Now that we have all the data we need, we first differencing the data and chose d=2, D=1 and m=12  for the model. The final model is SARIMAX(1, 2, 1)x(0, 1, 0, 12) with Median mortgage rate, Unemployment rate and Median rental price.
The prediction plot for the test set is as below and rmse = 92123.83:
![Image description](/tables/2.3.png)

As we can see both in the prediction plot and from the model rmse, the prediction is pretty bad. We considered the reason for the bad result is that we are using the imputing data to do the prediction instead of the real data, the error of the imputation would cause larger error for the prediction.

### 2.4 VAR: 
As alluded to earlier, we also implemented a VAR (Vector Auto-Regressive) model to see if we could better model this data by also taking into account the relationships among our exogenous variables. VAR essentially models the relationship between variables by using explanatory variables and their changes over time to attempt to make better predictions on our target variable.
![Image description](/tables/2.4.png)

Unfortunately, our VAR model performed poorly with an RMSE for MedianSoldPrice of about 21908.52 even with stationary data, but we gained some information from this model. After taking a Granger causal test I was able to statistically reinforce that our independent variables have at least some causal relationship to our dependent variable with a p-value of 0.002 .

## Selection of the model and forecasting

### 3.1 Model selection
![Image description](/tables/3.1.png)

We decided to select the SARIMA model as our final model, because it gives the lowest rmse.

### 3.2 Residual diagnosis
![Image description](/tables/3.2.png)

#### 3.2.1 White noises behaviors
From the line plot, the residual bounce around zero within a range, therefore it is zero-mean and has constant variance.

From the ACF plot, there is only an obvious autocorrelation exist at h=0. So, there is no autocorrelation in the residual.

#### 3.2.2 Normality
From the QQ plot, the line is close to the diagonal line. Also, from the Histogram plus estimated density, the curve is close to the normal distribution curve. So, the normality of residual is good.

#### 3.3 Forecast
We forecasted the MedianSoldPrice from Jan 2016 to Aug 2017.
![Image description](/tables/3.3-1.png)

Forecasting results
![Image description](/tables/3.3-2.png)


## Conclusions
While we were able to confidently forecast at a decent rate, around 2511 dollars is still too high say this model was in any way groundbreaking or impressive. A few things we might try if we were to solve this problem again in the future is to apply VAR more strategically, use more powerful models, collect more data, and do more research into macroeconomics to have a better intuition for building a more intelligent model. 

Starting with VAR, we naively applied it for all explanatory variables. While this might help show some relationship to the data, if we wanted to use to for more powerful forecasting we would want to make sure our variables we put into VAR were chosen more judiciously. Another option is instead to choose more powerful models that help us decipher which independent variables are more important. If we had the time we could have attempted some of the more cutting edge models like decision trees or Long Short Term Memory networks and compare them to see how they perform compared to our SARIMA model. Despite what models we choose the most important piece to this puzzle is our data and there are a number of things we could have done to our data to help improve our results. Firstly we discussed at length if it would be worth imputing data from 2004-2010 since that data was not included for MedianRentalPrice, but we decided instead to not include that data. Our reasoning was that the time period we would be imputing included the great recession which was a large deviation from how the economy was performing afterwards meaning imputing that data correctly might be difficult and inconsistent. If we had more time we might have tried to either collect the missing data or instead more creatively impute or model the missing data to make our model more accurate to reality. Lastly a brief understanding of macroeconomics might have provided us with better ideas of how the Median sold price might be affected by other things such as the federal interest rate since lower rates would theoretically incentivise more people to buy more homes increasing demand and thus pushing up the median price. With knowledge like this and additional time we could have more strategically sought out valuable data and thus greatly improve our model. 

While there are definitely improvements that could have been made to our model we are still satisfied with how it performed overall and believe it can be a great model for getting a general forecast of the housing price for California in the future.
