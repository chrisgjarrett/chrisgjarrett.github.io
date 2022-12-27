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

Kaituna blah lbah blah
<!--{% include gallery caption="This is a sample gallery to go along with this case study." %}-->

# Background
The Kaituna river is located at Okere Falls, Rotorua, New Zealand. For many, it's a spiritual home. The river, or awa, as it's known in Māori is one of the premier whitewater kayaking spots in New Zealand. It's a dam-controlled river and the gate levels are regulated in response to the level of Lake Rotoiti.

From a kayaking standpoint, the level of the gates greatly affects the river features. Paddlers typically describe the river level in terms of the average gate level, as read at the gates themselves at the beginning of the run. Thus, '100s' refers to all three gates being at the 100 mark, 200s at the 200 mark, and so on. If, say two gates are at 300 and one at 400, this could be referred to as anything from 300s to 350s, to 300/400s.

Fortunately, highly granulated data can be found at [the BOP regional council website|https://www.boprc.govt.nz]. Here, we can find all sorts of datasets, including lake level, flow rate, the level of all three gates and rainfall data.