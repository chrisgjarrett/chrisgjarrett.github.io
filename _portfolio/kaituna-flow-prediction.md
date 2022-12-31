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

# Summary of tech stack and skills
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

### Data resolution
Most of the data is collected at 5 minute intervals, while rainfall is measured on an hourly basis. I decided that hourly resolution was too granular. Typically, the gate levels are set in the morning or late at night which results in long periods of the day where the gate level does not change. Therefore, I decided that it was most useful to try and predict the flow on a daily basis.

### Target variable
I averaged all three gate levels to produce one value for each hour of the day, then took the median of all values to represent the day's gate level. By using the median rather than mean, my average value is not unfairly biased by short amounts of data in the early morning or late evening where the gate level changes. 

# Data collection
To gather the data, I built a web-scraper in Python, the source code for which is [here](https://github.com/chrisgjarrett/kaituna-model/blob/development/web_scraper/kaituna_web_scraper.py). The function grabs the level of lake rotoiti, the gate levels, flow rates and rainfall data. The output is a dataframe with hourly resolution data over the specified time range.

# Data exploration
After aggregating the data into daily resolution, I began data exploration. Initially, I wanted to get a feel for what the data looked like, before diving into exploring the relationships of different features on the output

## Average gate levels
![ Average gate levels](/assets/images/kaituna-project/flowrate-against-time.jpg "Average gate levels")

Immediately, it is clear that there is strong seasonality in the average gate levels. In addition to annual seasonality, where gate levels rise during winter, there is also a 4-5 year cycle, possibly corresponding to the La Nina/El Nino cycles. This is confirmed by a periodogram:

![ Periodogram of average flow](/assets/images/kaituna-project/periodogram.jpg "Rainfall against time")

## Rainfall and lake levels
| ![ Rainfall against time](/assets/images/kaituna-project/rainfall-against-time.jpg "Rainfall against time") | ![ Lake level against time](/assets/images/kaituna-project/rainfall-against-time.jpg "Lake level against time") |

| ![ Rainfall against gate level](/assets/images/kaituna-project/rainfall-x-gate-level.jpg "Rainfall against gate level") | ![ Lake level against gate level](/assets/images/kaituna-project/lake-level-x-gate-level.jpg "Lake level against gate level") |

Interestingly, neither rainfall nor lake level seem to exhibit the seasonality trends as strongly and neither seems to correlate too strongly with the average gate level. Lake level looks the most correlated but it is by no means a clear correlation. This motivates the investigation of nonlinear or interaction effects. It may also suggest that the dam operators have some sort of seasonal model that is independent of rainfall. 

Another possibility is that the lake level/rainfall on a given day does not correlate, but that the rainfall/lake level from earlier in the week does. This would be especially true if there is a delay between rainfall and the lake level rising.

TODO: Add in a plot seeing if correlation exists between the previous days' lake level, rainfall and current day's gate levels.

## Effect of previous flow rate on current
To evaluate whether there is any use in adding previous gate levels as a feature, I used partial auto-correlation and plotted the gate levels against previous days' levels:

| ![ Partial autocorrelation](/assets/images/kaituna-project/pcaf.jpg "PCAF") |
| ![ Lag plots](/assets/images/kaituna-project/lag-plots.jpg "Lag plots") |

It can be seen that correlation exists potentially up to 7 days prior to any given day. However, it's important to consider an actual reason why this might be the case. In my opinion, it's unlikely that the flow from 7 days ago is actually going to impact the flow today. Furthermore, the only truly strong effect is from the previous day's flow. There is some nonlinear effect at higher lags, but this could be due to noise. While it's worth checking if using 7 days of data helps, I would tend towards only considering the previous day's flow as important.

## Mutual information
I also looked at mutual information, particularly the amount contained within rainfall and lake level. I found that rainfall had 0.11 units and lake level had 0.62 units.

TODO: Investigate the MI of the lagged features

## PCA 
Finally, I decided to evaluate the PCA components of rainfall and lake level. I would expect these to be correlated and so using PCA may reveal new underlying relationships that are useful.

|           |      PC1 |       PC2 |
|:-----------:|:----------:|:-----------:|
|  Rainfall | 0.707107 |  0.707107 |
| LakeLevel | 0.707107 | -0.707107 |

![ PCA explained variance](/assets/images/kaituna-project/pca-variance-plots.jpg "PCA explained variance")

The mutual information of these components was 0.47 and 0.41 respectively. The loadings are interesting - one indicates a positive correlation and one indicates a negative correlation. It may be that this represents the increase in lake level because of rainfall, and the pre-emptive emptying of the lake in response to high rainfall/high forecasted rainfall.

## Summary
From data exploration, I observed that there is a strong seasonal component to the gate levels, with a particularly interesting multi-year cycle. Rainfall and lake level both have significant mutual information with the gate level, and when PCA was used, the contribution of each to the mutual information was more equal.

# Training a model
I went through several iterations of model selection, but settled on using a neural network to predict three days' worth of gate levels. I tried several architectures but focused mainly on variations of LSTMs, as they are particularly suited to timeseries data. 

## Feature selection

## MLFlow
I used MLFlow to set up an experiment tracker. I wasn't very rigorous about using it, but it was good to be introduced to the concept of experiment tracking. I set it up to record the model, the training dataset and several training/model parameters.

## Cross validation
My main training script had two modes of operation. In one, I performed 5-fold cross validation on my model. This is where I would compare different models. When I was ready for training, I performed a different split (95/5) and trained a final model. The trained model, and a trained preprocessor, were saved to operationalise the system.

## Final model architecture

## Results/Graphs

# Operationalisation
I wanted to serve the machine learning model up as a website, with a graph illustrating the last 3-4 days' gate levels and the predictions. The final website can be found [here](https://chrisgjarrett.github.io/kaituna-web-app/).

## Model training/creation
As described above, the model was locally trained in a Python script and the model and preprocessor were saved to file. While training locally isn't ideal and certainly not "enterprise-standard", I was keen to save money. Sagemaker, Vertex AI, etc are quite expensive in my experience.

## Making inferences
I created a second function to make inferences. This re-uses the web-scraper from before to collect data from the BOP council website, loads the preprocessor and model, then makes a prediction. Next, it creates a JSON structure containing the predictions, dates and the time the model has been updated, and uploads the JSON to an AWS S3 bucket.

The next step was to deploy this script as a Lambda function. AWS places an upper limit on the size of a lambda function and its dependencies. If this limit is exceeded, you must create the function as a docker image rather than directly uploading the code. I wrote a Dockerfile to package the code, create the Python environment, and execute the inference function.

Additionally, I wrote a .yml pipeline that is triggered whenever I push to the 'release-predictions' branch of the repo. The yml file compiles the Docker image, uploads it to my Elastic Container Registry on AWS, and then re-deploys the image to a Lambda function. The Lambda function is configured to run every 6 hours, meaning it runs at 9am, 3pm, 9pm and 3am NZ time. These times were chosen intentionally so as to be useful - i.e. you could check the predicted flows in the morning, afternoon and evening.

## Website
As mentioned, the model predictions are served to users through a [website](https://chrisgjarrett.github.io/kaituna-web-app/). The website pulls the JSON file containing the predictions from my AWS S3 bucket to populate its graph. In this way, client behaviour does not trigger a new prediction, which is beneficial from a resource use standpoint.