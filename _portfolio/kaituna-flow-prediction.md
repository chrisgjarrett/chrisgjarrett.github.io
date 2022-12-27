---
title: "Predicting the flow of the Kaituna river"
excerpt: "A project on using machine learning for timeseries predictions."
header:
  image: /assets/images/kaituna-project.jpg
  teaser: /assets/images/kaituna-project-th.jpg
gallery:
  - url: /assets/images/kaituna-project.jpg
    image_path: /assets/images/kaituna-project-th.jpg
    alt: "Ferrying the chute at kaituna"
---

<!--{% include gallery caption="This is a sample gallery to go along with this case study." %} -->

# Background

The Kaituna river is located at Okere Falls, Rotorua, New Zealand. For many, it's a spiritual home. The river, or awa, as it's known in Māori is one of the premier whitewater kayaking spots in New Zealand. It's a dam-controlled river and the gate levels are regulated in response to the level of Lake Rotoiti.
From a kayaking standpoint, the level of the gates greatly affects the river features. Paddlers typically describe the river level in terms of the average gate level, as read at the gates themselves at the beginning of the run. Thus, '100s' refers to all three gates being at the 100 mark, 200s at the 200 mark, and so on. If, say two gates are at 300 and one at 400, this could be referred to as anything from 300s to 350s, to 300/400s. 
Fortunately, highly granulated data can be found at [the BOP regional council website](https://www.boprc.govt.nz). Here, we can find all sorts of datasets, including lake level, flow rate, the level of all three gates and rainfall data. This inspired me to see if I could predict the next days' gate levels based on the available data and create a tool for paddlers to use.

## Problem definition

## Data resolution
An initial exploration of the data revealed that most data is collected at 5 minute intervals, while rainfall is measured on an hourly basis. I decided that hourly resolution was too granular. Usually, paddlers will plan to go to the Kaituna the night before, or morning of, and flow changes during the day are rare. It's more typical that the gate levels are set in the morning or late at night. Therefore, I decided that it was most useful to try and predict the flow on a daily basis.

## Target variable
I averaged all three gate levels to produce one value for each hour of the day, then took the median of all values to represent the day's gate level. By using the median rather than mean, my average value is not unfairly biased by short amounts of data gathered prior to the day's gate level being set. 

## 

# Data gathering
To gather the data, I built a web-scraper in Python, the source code for which is [here](https://github.com/chrisgjarrett/kaituna-model/tree/development/web_scraper). 
