---
title: "Predicting the flow of the Kaituna river"
excerpt: "A project on using machine learning for predicting timeseries values."
header:
  image: /assets/images/kaituna-project/kaituna-project.jpg
  teaser: /assets/images/kaituna-project/kaituna-project-th.jpg
gallery:
  - url: /assets/images/kaituna-project/images/kaituna-project.jpg
    image_path: /assets/images/kaituna-project/kaituna-project-th.jpg
    alt: "Ferrying the chute at kaituna"
---

<!--{% include gallery caption="This is a sample gallery to go along with this case study." %} -->
# Table of contents
- [Summary of tech stack and skills](#summary-of-tech-stack-and-skills)
- [Background](#background)
- [Data collection](#data-collection)
- [Data exploration](#data-exploration)
- [Training a model](#training-a-model)
- [Operationalising the model](#operationalising-the-model)
- [Future work](#future-work)


#### Summary of tech stack and skills
* Neural networks
  * Tensorflow
  * Keras
  * Timeseries predictions
  * LSTMs
* Docker
* Amazon Web Services
  * S3 
  * Lambda functions
  * Scheduling
* HTML/Javascript

# Background
The Kaituna river, located at Okere Falls, Rotorua, is one of the premier whitewater kayaking spots in New Zealand. The river is the main outlet for Lake Rotoiti and the flow is controlled by dam gates in order to maintain a consistent lake level.

The river level is typically described of the average gate level, which can be read at the gates themselves. '100s' refers to all three gates being at the 100 mark, 200s at the 200 mark, and so on. Each level of flow presents a distinctive kayaking run with various river features.

The gate levels, and several other data points, are all recorded and can be found at [the BOP regional council website](https://www.boprc.govt.nz). This inspired me to see if I could create a machine learning tool that predicts the gate levels over time and operationalise it as a webpage for paddlers to use.

## Problem definition

#### Target variable
Most of the data is collected at 5 minute intervals, while rainfall is measured on an hourly basis. Typically, however, the gate levels are set in the morning or late at night and do not change for long periods of the day. Therefore, I decided that it was most useful to try and predict the flow on a daily basis.

I averaged all three gate levels to produce one value for each hour of the day, then took the median of all values to represent the day's gate level. By using the median rather than mean, my average value is not unfairly biased by short amounts of data in the early morning or late evening where the gate level changes. I also decided to make a forecast of 3 days' worth of predictions.

## Base features
I decided to use previous gate levels, lake level and rainfall as the 'base' features. I know that lake level and rainfall are the most significant drivers of gate level, as the river is used to maintain the lake level. Additionally, there may be utility in including previous gate levels as predictors of the next days' levels.

# Data collection
To gather the data, I built a web-scraper in Python, the source code for which is [here](https://github.com/chrisgjarrett/kaituna-model/blob/development/web_scraper/kaituna_web_scraper.py). The function grabs the level of Lake Rotoiti, rainfall data and the gate levels.

# Data exploration
After aggreagating the data into a daily summary, I began the data exploration phase. Initially, I wanted to get a feel for what the data looked like, before diving into exploring the relationships of different features on the output

## Gate levels
![ Average gate levels](/assets/images/kaituna-project/gate-level-against-time.jpg "Average gate levels")

Immediately, it is clear that there is strong seasonality in the daily median gate levels. In addition to annual seasonality, where gate levels rise during winter, there appears to be a potential 4-5 year cycle, possibly corresponding to the La Nina/El Nino cycles. This is confirmed by a periodogram:

![ Periodogram of average flow](/assets/images/kaituna-project/periodogram.jpg "Rainfall against time")

I modelled the 4-5 yr seasonal component with a pair of cyclic sinusoid indicators and considered these as additional candidate features for my model.

## Effect of previous gate levels on current
To evaluate whether there is any use in adding previous gate levels as a feature, I used partial auto-correlation and plotted the gate levels against previous days' levels:

 ![ Partial autocorrelation](/assets/images/kaituna-project/pcaf.jpg "PCAF") 
 ![ Lag plots](/assets/images/kaituna-project/lag-plots.jpg "Lag plots")

Partial autocorrelation shows that correlation exists potentially up to 7 days prior to any given day. However, the only truly strong effect is from the previous day's flow. The lag plots show nonlinear effect at higher lags, but this could be due to noise. Overall, I would tend towards only considering the previous day's flow as important based on these results

## Rainfall and lake level
I also examined the rainfall and lake level against time and their correlation with the day's gate level. 

|Rainfall|Lake level|
|:--:|:--:|
| ![ Rainfall against time](/assets/images/kaituna-project/rainfall-against-time.jpg "Rainfall against time") | ![ Lake level against time](/assets/images/kaituna-project/lake-level-against-time.jpg "Lake level against time") |
![ Rainfall against gate level](/assets/images/kaituna-project/rainfall-x-gate-level.jpg "Rainfall against gate level") | ![ Lake level against gate level](/assets/images/kaituna-project/lake-level-x-gate-level.jpg "Lake level against gate level") |

Interestingly, neither rainfall nor lake level seem to exhibit the seasonality trends as strongly and neither visually appears to correlate too strongly with the average gate level for that day. 

This could suggest that the dam operators have a seasonal model of operation that is independent of rainfall, or that there are some nonlinear or interaction effects between rainfall and lake level that are significant.

#### Delayed effect of rainfall/lake level on gate levels
Intuitively, it is important to consider that rainfall is likely to have a delayed effect on lake level, and thus gate level. To this end, I plotted the rainfall and lake levels against a lagged gate level to investigate this effect. This is shown below for 0,1,2 and 3 days, with corresponding Pearson correlation coefficients. 

![ Lagged rainfall against gate level](/assets/images/kaituna-project/lagged-rainfall-against-gate.jpg "Lagged rainfall against gate level") 
![ Lagged lake against gate levels](/assets/images/kaituna-project/lagged-lake-against-gate.jpg "Lagged lake against gate levels")

**Pearson correlation coefficients for rainfall/lake level against gate level**

||0 days' lag|1 day's lag|2 days' lag| 3 days' lag|
|:-----------:|:----------:|:-----------:|:-----------:|:-----------:|
|Rainfall|0.15|0.26|0.28|0.24|
|Lake Level|0.55|0.57|0.57|0.55|

*p value for all coefficients was 0

For rainfall, the correlation between a given day's rainfall and a given day's gate levels is not as strong as comparing the given day's gate levels to the rainfall in the preceding days. That is, the rainfall appears to have a delayed effect on the gate levels. 
For lake level, the correlation with gate level is stronger, but consistent over each lag.

#### Correlation between rainfall and lake level
It is also interesting to examine whether lake level and rainfall are correlated, and quantify the delayed effect of rainfall on lake level. Here, I have plotted lake level against the rainfall from 0, 1, 2 and 3 days prior and computed the Pearson correlation coefficient for each. 

It appears that there is only weak correlation for all cases, but that the lake level for a given day is more strongly correlated with rainfall from previous days, compared to the same day. Given there is some correlation, it is interesting to consider PCA as an additional feature.

![ Lake level against past rainfall](/assets/images/kaituna-project/lagged-rainfall-x-lake-level.jpg "Lake level against past rainfall")

**Pearson correlation coefficients for rainfall and lake level**

|Days of lag|0 days' lag|1 day's lag|2 days' lag| 3 days' lag|
|:-----------:|:----------:|:-----------:|:-----------:|:-----------:|
|Correlation coefficient||0.19|0.29|0.27|0.24|

*p value for all correlation coefficients was 0

**PCA loadings**

|           |      PC1 |       PC2 |
|:-----------:|:----------:|:-----------:|
|  Rainfall | 0.707 |  0.707 |
| Lake level | 0.707 | -0.707 |

![ PCA explained variance](/assets/images/kaituna-project/pca-variance-plots.jpg "PCA explained variance")

The PCA loadings are interesting - one indicates a positive correlation and one indicates a negative correlation. It may be that these represent the increase in lake level because of rainfall, and the pre-emptive emptying of the lake in response to high rainfall/high forecasted rainfall.

## Mutual information
I used mutual information to get an idea of how much variance in gate level each potential feature explained. I did this for rainfall, lake level and the two PCA components. For each, I also investigated how much variance was explained by lagging the features by 1,2, and 3 days.

* Mutual information between features and gate level for varying lags

||0 days' lag|1 day's lag|2 days' lag|3 days' lag|
|:-----------:|:----------:|:----------:|:----------:|:----------:|
|Rainfall|0.11|0.15|0.12|0.12|
|Lake level|0.62|0.69|0.67|0.66|
|PCA component 1|0.48|0.54|0.54|0.51|
|PCA component 2|0.41|0.44|0.43|0.43|

The mutual information results confirm the correlation analysis: across all features, the values from previous days are generally more important than the current day's values.

## Rainfall forecast
The final feature I added was future rainfall. When training the model, I shifted the historic rainfall data by 1, 2 and 3 days, to get future values for each day of the forecast. Obviously this is not possible when using the model to make predictions in operation: here I can use rainfall forecasts instead. For the other features, where we cannot have future information, these were filled with 0s over the forecast horizon.

## Summary of data exploration
The final featues selected as:
* Rainfall
* 3 days' future forecast of rainfall
* Lake level
* Rainfall/Lake level PCA component 1
* Rainfall/Lake level PCA component 2
* Previous gate level
* A 4-5 year seasonal cyclic indicator

The feature set was normalised between 0 and 1 before being fed to the model.

# Training a model
From the outset I wanted to try and use an LSTM neural network to predict the gate levels. I used MLFlow, an ML experiment tracker, to help design the model structure and tune parameters. My data was split into a 60:20:20 ratio: 60% for training, 20% for validation, and 20% for testing.

## Experiment setup
I set up a cross-validation experiment, where the training data was split into 5 folds for leave-one-out validation. Each run stopped when the score on validation data reached a certain threshold. For a given 5-fold cross-validation, we get 5 validation scores (the experiment runs 5 times, leaving a different fold out each time). I took the median of these as a measure of one cross-validation run. 

However, the runs are stochastic and so this median score is not repeatable. To mitigate this, I repeated the above experiment 5 times and took the "median of medians". It was this value that I used to evaluate a given set of model parameters.

I performed several experiments using this routine and tuned the following parameters: model structure (i.e. layers and neuron counts), learning rate, different feature combinations and how many days' of data to get from the past (i.e. sample length for the LSTM). 

## Final model architecture and training
The final model contained 4 hidden layers, with 20, 20, 10 and 5 LSTM units respectively. It trained with a batch size equal to the length of the training data and a learning rate of 0.001. 

Feature-wise, it used the lake level, rainfall, and their PCA components from one day prior (e.g. using today's data to predict tomorrow's gate levels) and the PCA loadings obtained from combining rainfall and lake level. The 4-5 year seasonal component was dropped as it worsened model performance.

The output layer had a custom activation layer that limited the output to between 0 and 1500 (minimum and maximum possible gate levels). Once the model parameters had been chosen, the final model was retrained on the full training set (60% of the data), validated on 20% (used for early stopping, as per the cross-validation tuning experiments) and tested on the final 20%.

## Results/Graphs
The model had an RMS error of 137 units in training, 63.5 units in validation and 248.1 units for the test data (averaged over the three days). The plots below illustrate the fit for the training (top), validation (middle) and test (bottom) sets. Each colour on the graph is a new prediction lasting three days. 

![Model fitting results](/assets/images/kaituna-project/train-test-plot.jpg "Model fitting results")

# Operationalising the model
I wanted to serve the machine learning model up as a website, with a graph illustrating the last 3-4 days' gate levels and the predictions. If you just want to see the website, click [here](https://chrisgjarrett.github.io/kaituna-web-app/). Otherwise read on to learn the operational flow.

## Model training/creation
The model itself was locally trained in a Python [script](https://github.com/chrisgjarrett/kaituna-model/blob/development/train.py) and the model and preprocessor were saved to file. While training locally isn't ideal and certainly not "enterprise-standard", I was keen to save money - Sagemaker, Vertex AI, etc are quite expensive in my experience. 

## Making inferences
I created a second function to make inferences. This re-uses the web-scraper from before to collect data from the BOP council website, loads the preprocessor and model saved from the training script, then makes a prediction. Next, it creates a JSON structure containing the predictions, dates and an update time, and uploads this to an AWS S3 bucket.

The next step was to deploy this script as a Lambda function. I wrote a Dockerfile to package the code, create the Python environment, and execute the inference function. A .yml pipeline, triggered by a push to a release branch, was created to build the image, upload it to my Elastic Container Registry on AWS, and then re-deploy the image to a Lambda function. An AWS EventBridge scheduler executes the function every 6 hours at 9am, 3pm, 9pm and 3am NZ time. 

## Website
Finally, the model predictions are served to users through a [website](https://chrisgjarrett.github.io/kaituna-web-app/). The website downloads the JSON file containing the predictions from my AWS S3 bucket and uses it to populate a graph. In this way, client behaviour does not trigger a new prediction, which is beneficial from a resource use standpoint. 

# Future work
No project is ever complete, but I eventually reached a point where this was working end-to-end and I wanted to develop other aspects of my portfolio. I have documented the areas to explore for future work here. If you've made it this far and want to give me feedback, please do feel free to contact me via LinkedIn!

* Use a cloud-based ML service to host/deploy the model. I initially avoided this because it's expensive and probably won't implement it because of cost, however, it would be better practice and more maintainable.
* Schedule retraining for the model. This would involve periodically scraping new data, checking it for significant changes in its properties and retraining a new model for deployment.
* Make website ui smarter. Currently some UI decisions are made on the Python side and embedded into the JSON document. This is expedient, since dates are easier in Python than Javascript, but not best-practice. 
* The current solution architecture requires that a new image be deployed every time a new model is developed. However, it may be easier to deploy the model and preprocessor to an S3 bucket, which the lambda function references. This would mean that new models could be trained on new data without needing to redeploy an image.
* The model is good at predicting the gate levels when there are no changes, but poor at predicting when the gate levels will change, which is really when it is useful. An improvement to this could involve penalising the model more harshly if it mis-predicts changes in flow.
