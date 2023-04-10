---
title: "Analysis of airline ratings"
excerpt: "Exploring and visualising airline rating data."
header:
  teaser: /assets/images/airline-ratings/portfolio-logo.png
gallery:
  - url: /assets/images/airline-ratings/portfolio-logo.png
    image_path: /assets/images/airline-ratings/portfolio-logo.png
    alt: "Airline analytics stock image"
---

<!--{% include gallery caption="This is a sample gallery to go along with this case study." %} -->
<!-- # Table of contents -->


#### Summary of tech stack and skills
* Data visualisation
* Data exploration
* Tableau

# Background
For this project, I wanted to start to learn how to use Tableau and gain further practice with data analytics. I downloaded the Maven airline ratings dataset and started to explore. I aimed to answer the following question:
* What contributes to customer satisfaction and what does the profile of a satisfied customer look like?

I also looked at the profile of a returning customer, however, this analysis consisted largely of one dashboard and is best explained visually, so please view the dashboard if this is of interest.

A tableau story detailing the satisfaction analysis can be found [here](https://public.tableau.com/shared/JKPWBPMMC?:display_count=n&:origin=viz_share_link) and a dashboard profiling returning customers can be found [here](https://public.tableau.com/views/Returningcustomerprofile/Returningcustomerprofile?:language=en-GB&publish=yes&:display_count=n&:origin=viz_share_link). It's difficult to export plots from Tableau Public so I've attempted to summarise the key findings here, along with 'business recommendations'.

# Key takeaways
## Overview
* 43.4% of customers said they were satisfied, while 56.6% of customers said they felt neutral or dissatisfied.
* Unsurprisingly, 69.4% of business class customers were satisfied, compared to just 18.8% of Economy and 24.6% of Economy plus passengers.

## By subtype of passenger
* Across both travel type and customer type (business vs personal and first-time vs returning), the majority of customers in Economy classes were dissatisfied.
* When considering customer type, those travelling for personal reasons are overwhelmingly less satisfied than those travelling for business reasons, even when flying in business class.
* Interestingly, first-time flyers in business class were also majority neutral/dissatisfied (albeit close to 50/50), bucking the trend of business class flyers in general being more satisfied.


## Why are there cases where there are more dissatisfied than satisfied customers in business class?
Intuitively business class flyers should be more satisfied, a hypothesis backed up by the fact that 69.4% of business class flyers said they were satisfied. However, there are two sub-categories (first time flyers and those flying for personal reasons) where this trend was reversed.

### Returning customers vs first-time
* When considering returning vs first time customers in business class, we can see that there are many more respondants for returning customers vs first time. 
* The dataset contained several ratings for specific aspects of the travel (entertainment, check-in, etc). In general, there was not much difference in the median ratings given in each category by first time and returning customers in business class. 
* One explanation for why the number of people who marked their time as satisfied when flying as a returning customer, vs first time, could be selection bias. Customers who fly business class repeatedly are likely to do so because they satisfied - you would not fly business class and spend significant amounts of money as a returning customer if you were not already satisfied with the service.

### Personal vs business flyers in business class
* As with the first-time vs returning analysis, I compared the median ratings for each of the travel aspects. In particular, check in service, leg room, in-flight entertainment and cleanliness were among those categories that personal flyers rated less highly than business flyers and could serve as areas to focus on for the airlines.
* As mentioned above, however, there are significantly less people flying for personal reasons than business reasons. Combined with the fact that the responses are only 1 point apart, it may be that the small numbers of responses are reflecting slight differences more strongly. Statistical tests are needed to determine if these differences are significant or not. 
* It's also possible given that the difference in score is so slight, yet the difference in percentage of satisfied customers is so high, that the survey questions are not accurately collecting the information required to determine why these customers are not satisfied.
* It is also likely that those flying business class for personal reasons have used their own money, whereas those flying for business reasons have had their travel paid for. The investment of one's own money may lead to higher expectations, which again could explain the increased neutral/negative response. 

## Flight distance
Finally, I compared the satisfaction ratings across different flight distances. This yielded an interesting result whereby those customers flying short distances were mostly neutral/dissatisfied, but the percentage of satisfied customers increases and outnumbers neutral/dissatisfied as flight distance increases, peaking at about 3000-4000km before decreasing again slightly.

This could be down to psychology and/or differences in flight service. For example, the entertainment and food may not be as good on short-haul flights as long-haul. Customers may also be braced for longer flights and therefore more susceptible to/easily pleased by good service, even if the service were fundamentally the same as for short haul flights.