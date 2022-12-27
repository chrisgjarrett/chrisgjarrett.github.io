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

## Background

The Kaituna river is located at Okere Falls, Rotorua, New Zealand. For many, it's a spiritual home. The river, or awa, as it's known in Māori is one of the premier whitewater kayaking spots in New Zealand. It's a dam-controlled river and the gate levels are regulated in response to the level of Lake Rotoiti.

From a kayaking standpoint, the level of the gates greatly affects the river features. Paddlers typically describe the river level in terms of the average gate level, as read at the gates themselves at the beginning of the run. Thus, '100s' refers to all three gates being at the 100 mark, 200s at the 200 mark, and so on. If, say two gates are at 300 and one at 400, this could be referred to as anything from 300s to 350s, to 300/400s. 

Fortunately, highly granulated data can be found at [the BOP regional council website](https://www.boprc.govt.nz). It has all sorts of datasets pertaining to river flow and this inspired me to see if I could predict the next days' gate levels based on the available data and create a tool for paddlers to use.

### Problem definition

### Data resolution
An initial exploration of the data revealed that most data is collected at 5 minute intervals, while rainfall is measured on an hourly basis. I decided that hourly resolution was too granular. Usually, paddlers will plan to go to the Kaituna the night before, or morning of, and flow changes during the day are rare. It's more typical that the gate levels are set in the morning or late at night. Therefore, I decided that it was most useful to try and predict the flow on a daily basis.

### Target variable
I averaged all three gate levels to produce one value for each hour of the day, then took the median of all values to represent the day's gate level. By using the median rather than mean, my average value is not unfairly biased by short amounts of data gathered prior to the day's gate level being set. 

### Important feature variables
From my own domain knowledge, I know that lake level and rainfall are likely to be the primary indicators of the gate level. I used these, along with historical gate levels, as the 'base' features for my prediction system.

## Data collection
To gather the data, I built a web-scraper in Python, the source code for which is [here](https://github.com/chrisgjarrett/kaituna-model/blob/development/web_scraper/kaituna_web_scraper.py). The function grabs the level of lake rotoiti, the gate levels, flow rates and rainfall data. The output is a dataframe with hourly resolution data over the specified time range.

## Data exploration
After aggregating the data into daily resolution, I began to conduct data exploration. Initially, I wanted to get a feel for what the data looked like, before diving into exploring the relationships of different features on the output

### Average gate levels
![ Average gate levels](/assets/images/kaituna-project/flowrate-against-time.jpg "Average gate levels")

Immediately, it is clear that there is strong seasonality in the average gate levels. In addition to annual seasonality, where gate levels rise during winter, there is also a 4-5 year cycle, possibly corresponding to the La Nina/El Nino cycles.

## Rainfall and lake levels
| ![ Rainfall against time](/assets/images/kaituna-project/rainfall-against-time.jpg "Rainfall against time")| ![ Lake level against time](/assets/images/kaituna-project/rainfall-against-time.jpg "Lake level against time")|

| ![ Rainfall against gate level](/assets/images/kaituna-project/rainfall-x-gate-level.jpg "Rainfall against gate level") | ![ Lake level against gate level](/assets/images/kaituna-project/lake-level-x-gate-level.jpg "Lake level against gate level") |

Interestingly, neither rainfall nor lake level seem to exhibit the seasonality trends as strongly and neither seems to correlate too strongly with the average gate level. Lake level looks the most correlated but it is by no means a clear correlation. This motivates the investigation of nonlinear or interaction effects. It may also suggest that the dam operators have some sort of seasonal model that is independent of rainfall. Or, my rainfall data has been aggregated incorrectly.

## Quantifying seasonality
To get a better measure of the seasonality, I constructed a periodogram:
![ Periodogram of average flow](/assets/images/kaituna-project/periodogram.jpg "Rainfall against time")
The plot confirms that there is a strong seasonal component existing on roughly a 4-5 year cycle and a strong annual component.

In my opinion it is debatable whether adding a seasonal indicator feature will benefit the model hugely. It is likely that the seasonality observed is in response to rainfall - so if we include rainfall as a feature, will it add significantly more information?

## Effect of previous flow rate on current
To evaluate whether there is any use in adding previous gate levels as a feature, I used partial auto-correlation and plotted the gate levels against previous days' levels:
| ![ Partial autocorrelation](/assets/images/kaituna-project/pcaf-plots.jpg "PCAF") | ![ Lag plots](/assets/images/kaituna-project/lag-plots.jpg "Lag plots") |

It can be seen that correlation exists potentially up to 7 days prior to any given day. Interestingly, the 2nd day's lag is not significant. The lag plots indicate some sort of nonlinear effect at longer lags.

## Mutual information
I also looked at mutual information, particularly the amount contained within rainfall and lake level. I found that rainfall had 0.11 units and lake level had 0.62 units.

## PCA 
Finally, I decided to evaluate the PCA components of rainfall and lake level. I would expect these to be correlated and so using PCA may reveal new underlying relationships that are useful.

|           |      PC1 |       PC2 |
|-----------|----------|-----------|
|  Rainfall | 0.707107 |  0.707107 |
| LakeLevel | 0.707107 | -0.707107 |

| ![ Percent Explained variance](/assets/images/kaituna-project/explained-variance.jpg "% Explained variance") | ![ Cumulative variance](/assets/images/kaituna-project/cumulative-variance.jpg "Cumulative variance") |

The mutual information of these components was 0.47 and 0.41 respectively. The loadings are interesting - one indicates a positive correlation and one indicates a negative correlation. It may be that this represents the increase in lake level because of rainfall, and the pre-emptive emptying of the lake in response to high rainfall/high forecasted rainfall.

## Summary
From data exploration, I observed that there is a strong seasonal component to the gate levels, with a particularly interesting multi-year cycle. Rainfall and lake level both have significant mutual information with the gate level, and when PCA was used, the contribution of each to the mutual information was more equal.