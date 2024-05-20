# pairs_trading_classifier_model

## Introduction

This project builds on a standard pairs trading model for US equities to build confidence levels in trading signals that can then be used to further decide on which trades to make. The model is a bagging decision tree classifier which takes in information on trade signals - autocorrelation of the spread, correlation between the asset pair, and the coefficient on the AR1 process of the spread over a past window - and classifies the combination of those signals as likely to be profitable or not. The strategy then uses the confidence level of the model's classification as an indicator for taking positions, trading on the top 5 most confident predictions amongst the pairs analyzed. The strategy yields a 68.9% rate of successful trades compared to 62.1% success for a standard AR1 process prediction on the same highly correlated asset pairs, and produces 0.1% average returns per day compared to .04% for the standard method.

## Model design

### Motivation and training structure
The motivation for the model came from the observation that certain signals/ statistics of asset pair spreads were fairly well correlated with successful pairs trades. This was analyzed using biserial correlations on past data against a 0, 1 classification on successful trades that determined the autocorrelation of an asset pair's spread, the correlation between the two asset pairs (small percentage differences such as 93% vs 97% correlation were still impactful on returns), and the percentile of the last trading day's return, all produced statistically significant correlations with profitable trades while also being relatively uncorrelated with eachother. Thus, these 3 factors were likely strong inputs for a classification model. For signal design, the trading model at each day analyzes only the top 20 most correlated pairs over a past-30 day window, the model then takes the past 40 rolling windows of 30-day size for each of the 20 assets, for a total of 800 windows of data. These windows are built up to the second to last day of data as not to train on current data and future returns. For each window of data each of the 3 factors are calculated along with the AR1 process trade prediction, a data point is then built using the 3 factors as input and a binary output on whether the AR1 process predicted the future return correctly or not (data point structure: input = percentile of yesterday's return, pair correlation coefficient, spread autocorrelation, output = 1 if AR1 process correctly predicted trade 0 otherwise). 800 such data points are built using the past window and a bagging decision tree classifier is trained on that data. This will be the model used to predict confidence levels at each day. 

### Signal structure and trade execution
Given the model trained each day to predict the success of the AR1 process prediction, the 3 factors for each of the top 20 correlated pairs of that day are fed into the model and the probability of a 1 classification for each pair according to the model (using predict_proba function) is interpreted as a confidence level in a correct signal and the top 5 most confident signals are selected and trades executed on based on the predictions of the AR1 process. This process is repeated each day as since it is a daily pairs trading strategy time efficiency of signal production is not an issue as positions are just based on beginning and end of day executions.

## Results

### AR1 Process
Just as a baseline the following plot displays the returns of a pairs trading strategy just using AR1 proceess predictions trained once for each of the top 20 correlated pairs over a past year are used to predict the daily spread over the next year. In a successful pairs trade, the coefficients of the AR1 process are expected to be negative and thus suggest a reversion in spreads from time *t-1* to *t*. As seen in the results over a 1 year time period, this strategy works fairly well with an average return of .04% per day and a successful trade rate of 62.1%:

![AR1_process_pairs_trading_results](https://github.com/sayuj-choudhari/pairs_trading_classifier_model/assets/122761001/4af56666-310d-489f-a966-7ae1f459243e)

### Confidence classification
The following plot displays the results of the confidence classification model over the same trading period, with positions being taken daily on the 5 most confident signals versus all 20 correlated pairs. The model performs favorably to the baseline, with an average return of .09% per day and a successful trade rate of 68.9%:


![classification_model_pairs_trading_results](https://github.com/sayuj-choudhari/pairs_trading_classifier_model/assets/122761001/a3467f6a-5faa-4cd4-879c-7351cd8c708c)

Both models have a 2% stop loss on a position, which was only activated <5 times for both models out of over 250 trades.

## Continuation / Future work

The results from this project support the confidence classification as a signficant improvement to trade signal identification. The results of the strategy against the baseline show favorable returns but also a bit more volatility which may be expected given the 75% reduction in positions (20 to 5), making the portfolio more vulnerable to big moves in just a single pair. An alternative strategy would be to use the classification model and confidences as a weighting for each of the 20 pairs, putting signficantly higher weight on stronger confidences compared to the weaker ones at each day. This change could either improve returns from the baseline and/ or improve stability of the portfolio returns.






