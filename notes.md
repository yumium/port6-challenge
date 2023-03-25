## Data Challenge
- Data collected at 50Hz for ~100 seconds, so ~5000 samples in total in each take. There are 51 takes.
- Input data is from reflective optical sensors from the wrist area and has 30 dimensions, taking values with 10-bit resolution (0 ~ 1024) but mostly in range 100 - 500
- Label data has 5 dimensions (one for each finger) and the value is calculated as a dot product between 2 (normalized) vectors (-1 ~ 1)
- We can assume that during data collection, the fingers are pointing straight forward

*You will be judged on model results and your approach*

## Problem statement & Evaluation
- We want to take in sensor data and predict finger vectors (the 5-tuple of floats between [-1,1])
    - Success will have low regression error between predicted number and real dot product
        - MSE
        - RMSE <- this produces the same optimal points as MSE, as this metric is just sqrt(MSE)
        - MAE (mean absolute error) <- this metric is less sensitive to outliers
        - *MSE or RMSE probably makes the most sense as we don't mind if prediction is a little off, but if it's very off then we have a problem*
    - At production, inference will need to be fast to avoid "lagging" between prediction & the current position of fingers
- Data split (51 takes)
    - 30 takes as training data (60%)
    - 10 takes as validation data, if needed (20%)
    - last 11 takes as test data (20%)
- We also need to consider which finger is more important
- We also need to consider the variance of the loss, large offsets are more problematic over smaller offsets

## Exploratory data analysis
- Fingers 1-4 (indices) seem to mostly stay at 1 and when flexed will go down to -0.75
- Finger 0 have more variance (thumb?) ranging between 0.5 and -0.5
- The four fingers (excluding thumb) have similar label distribution, but prediction should be different as they should be triggering different tendons in the wrist
- **There may be fewer datapoints for labels not at the extremes**
- As the positions of other fingers will impact the tendon positions when the finger of focus remains at the same position, it's potentially difficult to train a model to find the relationship between tendons and 1 finger. Rather, it may be more fruitful to model the relationship between tendons and position of all fingers
- There is a lot of correlation between channels of the input data

## Features
- Continuous features:
    - 30 floats, basically a D30 vector
- Derived features:
    - Vectors are strongly correlated so maybe PCA can be helpful
    - SVMs?
    - Clustering?

## Models
- Pick a baseline model first
- Ideas
    - [X] Linear model for each finger (MLE for noise estimate) => perhaps w/ regularisation like Ridge or Lasso
        - Centering the data?
        - By extension, SVMs
    - [X] KNN -> advantage is that output will be in range => tune this
    - Basis expansion & RBF kernels 
    - [ ] NN -> small NN (use example from MLSS)
        architecture: input -> fully connected -> ReLU -> fully connected
        size: 30 -> 70 -> 5
    - [ ] CNN -> 
    - [ ] RNN -> if we assume that hand position is a function of the tendon positions, it's hard to justify why previous hand positions will help
            Other recurrent structures like Transformers

- Tradeoffs to consider
    - Interpretability -> don't need to be very interpretable, accuracy is more important
    - Data volume
    - Model complexity -> would prefer simpler models for faster online inference time


## More ideas
- I see a distribution shift (translation) in some of the datapoints, I wonder if there's systematic error between takes. An idea would be to centre the data in each take to make them uniform
- Normalizing features 



## Results

Plain Linear Regression
---Model Metrics for Finger 1---
CoD score: -2.4925515028723817
---Model Metrics for Finger 2---
CoD score: -0.24508146028350009
---Model Metrics for Finger 3---
CoD score: 0.5530381904364323
---Model Metrics for Finger 4---
CoD score: 0.5268586631158505
---Model Metrics for Finger 5---
CoD score: 0.47974075266095695

Linear Regression w/ Mean Normalization
---Model Metrics for Finger 1---
CoD score: 0.22161273989755548
---Model Metrics for Finger 2---
CoD score: 0.3599282224511101
---Model Metrics for Finger 3---
CoD score: 0.45244225454617537
---Model Metrics for Finger 4---
CoD score: 0.4463311108683772
---Model Metrics for Finger 5---
CoD score: 0.3895001244070522


Capping seem to improve performance -> though this should be obvious

Linear Regression with Capping
---Model Metrics for Finger 1---
CoD score: -0.976478786188187
---Model Metrics for Finger 2---
CoD score: -0.00885851561279849
---Model Metrics for Finger 3---
CoD score: 0.6172717559691311
---Model Metrics for Finger 4---
CoD score: 0.5901567606487776
---Model Metrics for Finger 5---
CoD score: 0.5086572493643581

Linear Regression w/ Mean Normalization and Capping
---Model Metrics for Finger 1---
CoD score: 0.23113533484318705
---Model Metrics for Finger 2---
CoD score: 0.451403481121183
---Model Metrics for Finger 3---
CoD score: 0.6263006634431207
---Model Metrics for Finger 4---
CoD score: 0.6368193886358684
---Model Metrics for Finger 5---
CoD score: 0.4864840018769735



Lasso at alpha=0.1 with centering
---Model Metrics for Finger 1---
CoD score: 0.21250322412915823
---Model Metrics for Finger 2---
CoD score: 0.1941164940022837
---Model Metrics for Finger 3---
CoD score: 0.5131486734447976
---Model Metrics for Finger 4---
CoD score: 0.5067631916275264
---Model Metrics for Finger 5---
CoD score: 0.582427083436889

Maybe bagging can be used? Also the model output no longer overfits for last 3 fingers and points are less jumpy
For without centering, alpha=1 work best for both Ridge and Lasso, though accuracy are both worse than centered option
With centering, Ridge is best at alpha=1 but performs worse than Lasso.




Different dimensions
Linear model on dimensions => very bad
Linear model on dimensions with centering => very bad
Linear model with regularization (with centering)

Lasso with alpha=0.1 (with centering) over 2D polynomial basis expansion. Best results for thumb, but not as great for others
---Model Metrics for Finger 1---
CoD score: 0.3181681994834006
---Model Metrics for Finger 2---
CoD score: 0.14032708339672317
---Model Metrics for Finger 3---
CoD score: 0.458420766246907
---Model Metrics for Finger 4---
CoD score: 0.4762958183675696
---Model Metrics for Finger 5---
CoD score: 0.34901124354738455

Haven't had a chance to try Ridge regression or use other alphas






KNN; K=10
---Model Metrics for Finger 1---
CoD score: -0.5301566151793293
---Model Metrics for Finger 2---
CoD score: -0.4119775950333282
---Model Metrics for Finger 3---
CoD score: 0.32379590423475335
---Model Metrics for Finger 4---
CoD score: 0.34373768420630657
---Model Metrics for Finger 5---
CoD score: 0.3284730342282973

KNN; K=10; centered
---Model Metrics for Finger 1---
CoD score: 0.35666574562995
---Model Metrics for Finger 2---
CoD score: 0.43557966703990914
---Model Metrics for Finger 3---
CoD score: 0.645909085790972
---Model Metrics for Finger 4---
CoD score: 0.7385912212740045
---Model Metrics for Finger 5---
CoD score: 0.4917853123696915

KNNs seem to overcome 2 challenges faced by linear models. 1. the output will always be in range [-1,1] and 2. it is less affected by distribution shifts in the test data, as it seeks neighbours in the training data which will only concern the nearest datapoints, which are probably ones with the same distribution shift

Model training and validation complete for k value 32
---Model Metrics for Finger 1---
CoD score: 0.4787963091437202
---Model Metrics for Finger 2---
CoD score: 0.4562276970632142
---Model Metrics for Finger 3---
CoD score: 0.7050286216275041
---Model Metrics for Finger 4---
CoD score: 0.7549996735304396
---Model Metrics for Finger 5---
CoD score: 0.49125453153419574

Maybe the larger the k the better???




