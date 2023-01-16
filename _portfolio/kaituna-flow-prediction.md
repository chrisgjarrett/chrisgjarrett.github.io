---
title: "Predicting the flow of the Kaituna river"
excerpt: "A project on using machine learning for predicting future values in a timeseries."
header:
  image: /assets/images/kaituna-project/kaituna-project.jpg
  teaser: /assets/images/kaituna-project/kaituna-project-th.jpg
gallery:
  - url: /assets/images/kaituna-project/images/kaituna-project.jpg
    image_path: /assets/images/kaituna-project/kaituna-project-th.jpg
    alt: "Ferrying the chute at kaituna"
---

<!--{% include gallery caption="This is a sample gallery to go along with this case study." %} -->

##### Table of Contents  
[Headers](#headers)  
[Emphasis](#emphasis)  
...snip...    
<a name="headers"/>
## Headers

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
The Kaituna river, located at Okere Falls, Rotorua, is one of the premier whitewater kayaking spots in New Zealand. It's a dam-controlled river and serves as the main outlet for Lake Rotoiti, with outflow regulated by the gate levels.

For kayaking, the level of the gates greatly affects the river features. Paddlers typically describe the river level in terms of the average gate level, which can be read at the gates themselves. River flow is described in terms of these numbers, so '100s' refers to all three gates being at the 100 mark, 200s at the 200 mark, and so on. 

The gate levels, and several other data points, are all recorded and can be found at [the BOP regional council website](https://www.boprc.govt.nz). This inspired me to see if I could create a machine learning tool that predicts the gate levels in coming days and serve it up as a tool for paddlers to use.

## Problem definition

#### Data resolution
Most of the data is collected at 5 minute intervals, while rainfall is measured on an hourly basis. I decided that hourly resolution was too granular. Typically, the gate levels are set in the morning or late at night which results in long periods of the day where the gate level does not change. Therefore, I decided that it was most useful to try and predict the flow on a daily basis.

#### Target variable
I averaged all three gate levels to produce one value for each hour of the day, then took the median of all values to represent the day's gate level. By using the median rather than mean, my average value is not unfairly biased by short amounts of data in the early morning or late evening where the gate level changes. I also decided to make a forecast of 3 days' worth of predictions.

## Feature selection
I decided to use previous gate levels, lake level and rainfall as the 'base' features. I know that lake level and rainfall are the most significant drivers of gate level, as the river is used to maintain lake level. Additionally, there may be utility in including previous gate levels as predictors of the next days' levels.

# Data collection
To gather the data, I built a web-scraper in Python, the source code for which is [here](https://github.com/chrisgjarrett/kaituna-model/blob/development/web_scraper/kaituna_web_scraper.py). The function grabs the level of Lake Rotoiti, the gate levels, flow rates and rainfall data. The output is a dataframe with hourly resolution data over the specified time range.

# Data exploration
After aggregating the data into daily resolution, I began data exploration. Initially, I wanted to get a feel for what the data looked like, before diving into exploring the relationships of different features on the output

## Gate levels
![ Average gate levels](/assets/images/kaituna-project/gate-level-against-time.jpg "Average gate levels")

Immediately, it is clear that there is strong seasonality in the daily median gate levels. In addition to annual seasonality, where gate levels rise during winter, there appears to be a 4-5 year cycle, possibly corresponding to the La Nina/El Nino cycles. This is confirmed by a periodogram:

![ Periodogram of average flow](/assets/images/kaituna-project/periodogram.jpg "Rainfall against time")

I modelled the 4-5 yr seasonal component with a pair of sinusoids, who period was set to 365.25 * 4 (365.25 days/yr and a 4 year cycle).

## Effect of previous gate levels on current
To evaluate whether there is any use in adding previous gate levels as a feature, I used partial auto-correlation and plotted the gate levels against previous days' levels:

| ![ Partial autocorrelation](/assets/images/kaituna-project/pcaf.jpg "PCAF") |
| ![ Lag plots](/assets/images/kaituna-project/lag-plots.jpg "Lag plots") |

It can be seen that correlation exists potentially up to 7 days prior to any given day. However, the only truly strong effect is from the previous day's flow. There is some nonlinear effect at higher lags, but this could be due to noise. While it's worth checking if using 7 days of data helps, I would tend towards only considering the previous day's flow as important.

## Rainfall and lake level
First, I examine the rainfall and lake level against time and their correlation with the day's gate level. 

|Rainfall|Lake level|
|:--:|:--:|
| ![ Rainfall against time](/assets/images/kaituna-project/rainfall-against-time.jpg "Rainfall against time") | ![ Lake level against time](/assets/images/kaituna-project/lake-level-against-time.jpg "Lake level against time") |

| ![ Rainfall against gate level](/assets/images/kaituna-project/rainfall-x-gate-level.jpg "Rainfall against gate level") | ![ Lake level against gate level](/assets/images/kaituna-project/lake-level-x-gate-level.jpg "Lake level against gate level") |

Interestingly, neither rainfall nor lake level seem to exhibit the seasonality trends as strongly and neither seems to correlate too strongly with the average gate level. This is likely to be due to the fact that lake level is regulated. Lake level looks the most correlated but it is by no means a strong correlation. This could suggest that the dam operators have a seasonal model of operation that is independent of rainfall, or that there are some nonlinear or interaction effects between rainfall and lake level that are significant. The correlation for each is quantified in the next section.

#### Delayed effect of rainfall/lake level on gate levels
It is important to consider that there may be a delayed effect of rainfall or lake level on the gate levels. That is, that rainfall on a given day may result in the lake rising one or two days later. To this end, I can plot the rainfall and lake levels against a lagged gate level to investigate this effect. This is shown below for 0,1,2 and 3 days, with corresponding Pearson correlation coefficients. 

|![ Lagged rainfall against gate level](/assets/images/kaituna-project/lagged-rainfall-against-gate.jpg "Lagged rainfall against gate level")|![ Lagged lake against gate levels](/assets/images/kaituna-project/lagged-lake-against-gate.jpg "Lagged lake against gate levels")|

* Pearson correlation coefficients for rainfall/lake level against gate level

||0 days' lag|1 day's lag|2 days' lag| 3 days' lag|
|:-----------:|:----------:|:-----------:|:-----------:|:-----------:|
|Rainfall|0.15|0.26|0.28|0.24|
|Lake level|0.55|0.57|0.57|0.55|

*p value for all coefficients was 0

For rainfall, the correlation between a given day's rainfall and a given day's gate levels is not as strong as comparing the given day's gate levels to the rainfall in the preceding days. That is, the rainfall appears to have a delayed effect on the gate levels. For lake level, the correlation with gate level is stronger, however, it does not change as much with previous days. 

Of course, the Pearson correlation coefficient only tests for linear correlation, nonetheless the results are interesting. They suggest that the previous day's rainfall is more important than the current rainfall in determining the current day's gate level. It also shows that lake level is important, and more correlated with the gate levels than rainfall. However, each day appears to be equally correlated with the current day's gate levels.

#### Correlation between rainfall and lake level
It is also interesting to examine the correlation of lake levels and rainfall, and quantify the delayed effect of rainfall on lake level. Here, I have plotted lake level against the rainfall from 0, 1, 2 and 3 days prior and computed the Pearson correlation coefficient for each. It appears that there is only weak correlation for all cases, but that the rainfall from days prior is indeed more correlated than the same-day relationship. 

![ Lake level against past rainfall](/assets/images/kaituna-project/lagged-rainfall-x-lake-level.jpg "Lake level against past rainfall")

* Pearson correlation coefficients for rainfall and lake level

|Days of lag|0 days' lag|1 day's lag|2 days' lag| 3 days' lag|
|:-----------:|:----------:|:-----------:|:-----------:|:-----------:|
|Correlation coefficient||0.19|0.29|0.27|0.24|

*p value for all correlation coefficients was 0

Of course, this only tests for a linear relationship. Nonetheless, given there is some correlation, it is interesting to consider PCA as it may reveal new underlying relationships that are useful.

* PCA loadings

|           |      PC1 |       PC2 |
|:-----------:|:----------:|:-----------:|
|  Rainfall | 0.707 |  0.707 |
| Lake level | 0.707 | -0.707 |

![ PCA explained variance](/assets/images/kaituna-project/pca-variance-plots.jpg "PCA explained variance")

The mutual information of these components was 0.47 and 0.41 respectively. The loadings are interesting - one indicates a positive correlation and one indicates a negative correlation. It may be that this represents the increase in lake level because of rainfall, and the pre-emptive emptying of the lake in response to high rainfall/high forecasted rainfall.

## Mutual information
I used mutual information to get an idea of how much variance in gate level each variable explained. I did this for rainfall, lake level and the two PCA components. For each, I also investigated how much variance was explained by lagging the features by 1,2, and 3 days.

* Mutual information between features and gate level for varying lags

||0 days' lag|1 day's lag|2 days' lag|3 days' lag|
|:-----------:|:----------:|:----------:|:----------:|:----------:|
|Rainfall|0.11|0.15|0.12|0.12|
|Lake level|0.62|0.69|0.67|0.66|
|PCA component 1|0.48|0.54|0.54|0.51|
|PCA component 2|0.41|0.44|0.43|0.43|

The mutual information results confirm the correlation analysis: across all features, the values from previous days have more of an effect on a given day's gate levels than the current day's values.

## Rainfall forecast
Finally, I also considered what future information was available to help with predictions. Of the variables considered so far, rainfall is the only one for which we have can have prior information, from weather forecasts. To create this feature, I shifted the rainfall data by 1, 2 and 3 days, to match each day of the prediction horizon. In reality, this will be replaced by a rain forecast, rather than the actual rainfall. For the other features, where we cannot have future information, these were filled with 0s for the 3 day forecast horizon.

## Summary of data exploration
From data exploration, I observed that there is a strong seasonal component to the gate levels, with a particularly interesting multi-year cycle. Lake level is more strongly correlated with the gate levels than rainfall, and PCA analyses indicated that inclduing PCA components may be useful. For all features, the values on a given day are not as important as what the values were in previous days: i.e. the rainfall and lake level from days prior are more important for explaining the variance in gate level than the same-day values.

The final featues were selected as:
* Rainfall
* 3 days' future forecast of rainfall
* Lake level
* Rainfall/Lake level PCA component 1
* Rainfall/Lake level PCA component 2
* Previous gate level

The feature set was normalised between 0 and 1 before being fed to the model.

# Training a model
The fact that this is a timeseries prediction problem lends itself to the use of LSTMs. I initially started out with some non-systematic pilot investigations, then used a more systematic approach to narrow down the final parameters and model structure. To help in this, I used MLFlow, an experiment tracker. It was set up to record the model, the training dataset and several training/model parameters. I split the data into a 60:20:20 split: 60% for training, 20% for validating the parameters discussed below, and 20% for testing.

## Experiment setup
I set up a cross-validation experiment, where the training data was split into 5 folds for a k-fold leave-one-out validation. An early stopping routine was employed when the validation score did not exceed 1 unit for 200 epochs. For a given 5-fold validation, we get 5 result metrics, representing the validation score for the fold that was left out. I took the median of the 5 results to get the overall result for one cross-validation run. However, the runs are stochastic and so the score is not repeatable. To mitigate this, I repeated each cross-validation experiment 5 times, and took the "median of medians", that is, the median score across all 5 cross-validation experiments, to evaluate each type of model. 

I performed several experiments using the cross-validation routine described above and tuned the following parameters: model structure (i.e. layers and neuron counts), learning rate, different feature combinations and how many days' of data to get from the past (i.e. sample length for the LSTM). 

## Final model architecture and training
To summarise, the model contained 3 hidden layers, with 20, 10 and 5 LSTM units respectively, a batch size equal to the length of the training data, a learning rate of 0.001 and 1 day of historic data combined with 3 days' rainfall forecast. The output layer had a custom activation layer that limited the output to between 0 and 1500 (minimum and maximum possible gate levels). The final data was trained in the same manner as cross-validation: trained on 60% of the data, validated on 20% and tested on the final 20%.

## Results/Graphs
The model trained with an RMS error of 137 units, validated with an RMS error of 63.5 units and predicted the test gate levels with an RMS error of 248.1 units. The plots below illustrate the fit for the training (top), validation (middle) and test (bottom) sets. Each colour on the graph is a new prediction lasting three days. 

![Model fitting results](/assets/images/kaituna-project/train-test-plot.jpg "Model fitting results")

# Operationalising the model
I wanted to serve the machine learning model up as a website, with a graph illustrating the last 3-4 days' gate levels and the predictions. The final website can be found [here](https://chrisgjarrett.github.io/kaituna-web-app/).

## Model training/creation
As described above, the model was locally trained in a Python script and the model and preprocessor were saved to file. While training locally isn't ideal and certainly not "enterprise-standard", I was keen to save money. Sagemaker, Vertex AI, etc are quite expensive in my experience.

## Making inferences
I created a second function to make inferences. This re-uses the web-scraper from before to collect data from the BOP council website, loads the preprocessor and model, then makes a prediction. Next, it creates a JSON structure containing the predictions, dates and the time the model has been updated, and uploads the JSON to an AWS S3 bucket.

The next step was to deploy this script as a Lambda function. AWS places an upper limit on the size of a lambda function and its dependencies. If this limit is exceeded, you must create the function as a docker image rather than directly uploading the code. I wrote a Dockerfile to package the code, create the Python environment, and execute the inference function.

Additionally, I wrote a .yml pipeline that is triggered whenever I push to the 'release-predictions' branch of the repo. The yml file compiles the Docker image, uploads it to my Elastic Container Registry on AWS, and then re-deploys the image to a Lambda function. The Lambda function is configured to run every 6 hours, meaning it runs at 9am, 3pm, 9pm and 3am NZ time. These times were chosen intentionally so as to be useful - i.e. you could check the predicted flows in the morning, afternoon and evening.

## Website
As mentioned, the model predictions are served to users through a [website](https://chrisgjarrett.github.io/kaituna-web-app/). The website pulls the JSON file containing the predictions from my AWS S3 bucket to populate its graph. In this way, client behaviour does not trigger a new prediction, which is beneficial from a resource use standpoint. 

# Future work
No project is ever complete, but I eventually reached a point where I wanted to develop other aspects of my portfolio. I have documented the areas to explore for future work here:

* Use a cloud-based ML service to host/deploy the model. I initially avoided this because it's expensive and probably won't implement it because of cost, however, it would be better practice and more maintainable.
* Schedule retraining for the model. This would involve periodically scraping new data, checking it for significant changes in its properties and retraining a new model for deployment.
* Make website ui smarter. Currently some UI decisions are made on the Python side and embedded into the JSON document. This is expedient, since dates are easier in Python than Javascript, but not best-practice. 
* The current solution architecture requires that a new image be deployed every time a new model is developed. However, it may be easier to deploy the model and preprocessor to an S3 bucket, which the lambda function references. This would mean that new models could be trained on new data without needing to redeploy an image.
* The model is good at predicting the gate levels when there are no changes, but poor at predicting when the gate levels will change, which is really when it is useful. An improvement to this could involve penalising the model more harshly if it mis-predicts changes in flow.
