---
title: "Remote work output analytics"
excerpt: "An analysis on processing medical results."
header:
  teaser: /assets/images/remote-work-analytics/remote-work-icon.png
gallery:
  - url: /assets/images/remote-work-analytics/remote-work-icon.png
    image_path: /assets/images/remote-work-analytics/remote-work-icon.png
    alt: "Remote work icon"
---

# Table of contents
- [Executive summary](#executive-summary)
- [Tech stack and skills used](#tech-stack-and-skills-used)
- [Background](#background)
- [Basic demographics](#basic-demographics)
- [Comparing output rate on computer systems](#comparing-output-rate-on-computer-systems)
- [Copied results vs ordered results](#copied-results-vs-ordered-results)
- [Distribution of result type](#distribution-of-result-type)
- [Quantifying the makeup of results on time](#quantifying-the-makeup-of-results-on-time)

# Executive summary
- Female GPs appear to receive proportionally more results than male GPs.
- The choice of computer system does not impact the time taken to process results.
- Different categories of result are not distributed equally between regions, nor between genders.
- A negative binomial regression model was built to understand the influence of each type of result on the total time to clear the inbox. The results were inconclusive but allow certain general trends to be drawn.

# Tech stack and skills used
* Excel
* R
* Data visualisation
* Statistical inference

# Background
This project employed data analytics and statistics to analyse the work done by a general practitioner (GP) responsible for virtual inbox management. Virtual inbox management refers to the processing of medical results that get emailed to doctors. Doctors typically prescribe medical tests and get notified of the results in their inbox. This creates a huge amount of paperwork for them to process and they often struggle to fit it into a standard workday. I was approached by a GP who was managing several inboxes for clinics across to New Zealand, to analyse their output and help to indicate the value of their service to future clinics.

## Code repository
Most work was done in Excel, with a small amount in R. This can be viewed [here.](https://github.com/chrisgjarrett/remote-gp-inbox)

## Process
The GP kept a spreadsheet of data, with each row corresponding to the results cleared from one doctor's inbox on a given day, and each column representing the total number of each result type cleared from the inbox. Briefly, the categories of result are:
  - Laboratory
  - Laboratory.copy
  - Imaging
  - Imaging.copy
  - Referral.comms
  - Referral.ext.copy
  - Discharge.summary
  - Medical.letter
  - Allied.health.letter
  - Admin
  - Screening
  - Telehealth.consult
  - Other

  They also recorded the total time taken to clear the inbox. This table was imported to a second sheet for analytics and left-joined with a second table containing details about the doctors (patient numbers, location, etc). From this second sheet, analytics were performed. This streamlined the workflow, allowing the process of data entry and analytics to be separated.

# Basic demographics
I began by creating some plots to illustrate some basic demographics and do an exploration into gender-related aspects of the data. 

## Analysing results across doctor gender 
First, we can examine the gender ratio split amongst the doctors themselves:

![Gender split amongst doctors](/assets/images/remote-work-analytics/doctor-gender-split.png "Gender split amongst doctors")

Next we can examine the number of patients enrolled to male and female doctors:

![Number of patients enrolled by doctor gender](/assets/images/remote-work-analytics/number-of-patients-by-doctor-gender.png "Number of patients enrolled by doctor gender")

Finally we can examine the number of results doctors receive by gender:

![Number of results received by doctor gender](/assets/images/remote-work-analytics/number-of-results-by-gender.png "Number of results received by doctor gender")

The most interesting insight here is that although there are less female doctors (45.5%) with less enrolled patients (41.6%), the female doctors collectively have more results in their inbox to clear (51.8%). 

This is an interesting result and it may imply that female doctors are receiving proportionally more results overall. It may be worth conducting further analyses to determine if it's a statistically significant difference, and diving deeper into the data to establish if it's a meaningful difference. 

# Comparing output rate on computer systems
The GP encountered two different computer systems during the inbox processing work: 32 and Evolution. They were interested to understand if one system was quicker to use than the other. For this, I used a t-test and compared the average time to complete results per minute.

### Confirming normality
The first step was to confirm if the data had a normal distribution. This was done with a Jacque-Bera normality test, which tests normality on the basis of kurtosis and skew.

The p-values for the normality test are given below:

|System|p-value|
|:--:|:--:|
|Evolution|0.86|
|32|0.001|

This tells us that the data collected on the Evolution system was not normally distributed, while the data on the 32 system was normally distributed. This is confirmed by examining the box and whisker plots:

![Results per minute for each computer system.](/assets/images/remote-work-analytics/32-vs-evolution.png "Results per minute for each computer system.")

It is interesting that 32 has a heavily right-skewed distribution. It may speak to outliers where a particular doctor is receiving certain type of results that are easier to process. For more insights, it may be beneficial to control for result type when making this comparison.

However, despite the non-normal distribution we can proceed with the t-test, given that the sample size for both systems is > 30 (Evolution: n=59, 32: n=90).

### Confirming equality of variance
An F-test was used to test whether equality of variance could be assumed for the t test. The F-test had a p-value of 2.7e<sup>-6</sup>, indicating that the two samples do not have equal variance.

### Comparing results per minute
Finally, I performed an unequal variance t test comparing the results per minute for the two computer systems. The p-value of 0.12 indicates that there is no difference between the number of results per minute that the GP was processing, which in turn suggests that the choice of computer system has no bearing on the speed with which results can be processed.

# Copied results vs ordered results
In certain cases, the results that appear in a doctor's inbox are instances of them being copied into results rather than results they have actually ordered. It's interesting to analyse this proportion because the results are typically generated by external specialists and sent to the patient's GP. There is currently no formal policy on who is responsible for communicating these results to the patient: it is implied that the receiving GP will do it. Understanding how much of a GPs results are copied vs non-copied could help inform such a a policy. It could also help GPs understand how much of their inbox is work they have created, vs work that someone else has created.

I analysed this with a pivot table, summing the results across regions. This allowed me to create the following bar chart, which shows us that for Imaging results, the overwhelming majority of results in doctors' inboxes are instances where they have been copied in. The opposite is true for Laboratory results, where the majority tend to be results that the doctors themselves have ordered. These were the only categories of results where someone could be copied in.

![Imaging vs Imaging copy by region](/assets/images/remote-work-analytics/lab-copy-ratio-by-region.png "Imaging vs Imaging copy by region")

![Lab vs Lab copy by region](/assets/images/remote-work-analytics/imaging-copy-ratio-by-region.png "Lab vs Lab copy by region")


# Distribution of result type
Across each GP, there are several common categories of result. In this section, an analysis of the distribution of result types is presented. This was done for regions and genders.

## General approach
For each test, I conducted a chi-squared goodness-of-fit test, first confirming that in each case, no more than 20% of the expected values evaluated to 5. This was not the case for either test and thus the validity of the chi-squared approach was confirmed.

## Across genders
The p value for this test was 2.4e<sup>-8</sup>, indicating that there is a significant difference in how test results are distributed between males and females. This is indicated in the graph below, where I have compared the expected numbers of each result to the actual numbers for both genders.

![Results distributions for male doctors.](/assets/images/remote-work-analytics/results-distribution-males.png "Results distribution for male doctors")

![Results distributions for female doctors.](/assets/images/remote-work-analytics/results-distribution-female.png "Results distribution for female doctors")

It can be seen that females tend to have proportionally more laboratory results to process than expected, while males get less. There also looks to be a large difference in expected medical letters, where males are getting more medical letters to process and females less. 

Statistics such as these can help to understand what kind of results doctors are ordering and can also point towards the kinds of patients doctors get assigned. A deeper dive is needed to understand whether the differences are problematic from a gender equality perspective, but these sorts of insights could help doctors at the administrative level, ensuring that different types of patients are assigned equally across doctors. 

One limitation is that we do not know the gender of the patients, which probably has a large effect on which doctor they are assigned to. More interesting insights could be obtained if patient gender could be controlled for.

## Across regions
Similar results are seen across regions, with a p value of 0 for the chi-squared test. This indicates that results are not equally distributed across regions, with doctors in some regions having more types of results than others.

![Results distributions for Christchurch doctors.](/assets/images/remote-work-analytics/results-distribution-christchurch.png "Results distribution for Christchurch doctors")

![Results distributions for North Shore doctors.](/assets/images/remote-work-analytics/results-distribution-north-shore.png "Results distribution for North Shore doctors")

![Results distributions for South Auckland doctors.](/assets/images/remote-work-analytics/results-distribution-south-auckland.png "Results distribution for South Auckland doctors")

![Results distributions for Tauranga doctors.](/assets/images/remote-work-analytics/results-distribution-tauranga.png "Results distribution for Tauranga doctors")

![Results distributions for Wellington doctors.](/assets/images/remote-work-analytics/results-distribution-wellington.png "Results distribution for Wellington doctors")

Similarly to the gender comparison, these results provide insights into the types of results being processed at national level. This could be beneficial when deciding on resource allocation, or examining population health statistics at a national level. For example, Wellington receives proportionally many more laboratory results than would be expected. This could point to a justification to have dedicated processing facilities in Wellington to help process them faster. Or it could be that doctors from those clinics are ordering too many laboratory results. The analysis helps to raise these sorts of questions and highlights areas of interest for further investigation.

# Quantifying the makeup of results on time
The GP also wished to understand how the different types of results affected the total time spent on processing the results. Because the predictors are count variables (numbers of results) and the dependent variable is continuous (time), I used a negative binomial regrssion analysis and compared the resulting coefficients.

## Negative binomial vs Poisson regression
The negative binomial regression analysis was preferred to a Poisson regression because the variance of the time taken (85.4 min<sup>2</sup>) was much larger than the mean (14.8 minutes). Regardless, I trialled both models and compared them by examining residual plots and also performing a likelihood ratio test to compare the models

First, a likelihood ratio is computed - how well each model fits the data. A chi-squared test is then used to determine if the difference is statistically significant. In this case, the likelihood scores for negative binomial and poisson were -637.97 and -Inf respectively. This gives a likelihood ratio of Inf, which of course resulted in a p value of 0, indicating that the negative binomial model gave a significantly better fit. This is confirmed by the residual plots below:

![Binomial residuals.](/assets/images/remote-work-analytics/binomial-residuals.svg "Binomial residuals")

![Poisson residuals.](/assets/images/remote-work-analytics/poisson-residuals.svg "Binomial residuals")

The standardised residuals of the negative binomial model are more tightly clustered within the +/- 2 standard deviation range, whereas the poisson residuals are spread across a larger range.

In both cases, there are appears to be on prediction that is an extremely large outlier. It may be an innocuous data entry error, or a more serious problem with the underlying model. It warrants further investigation, but this shall not be covered here.

## Examining regression coefficients
Finally, we can compare the regression coefficient magnitudes to get a sense of relative importance of the different result types on the total time taken. A plot of the estimated coefficients with standard error bars is shown below:

![Regression coefficients.](/assets/images/remote-work-analytics/coefficient-comparison-time-taken.svg "Regression coefficients")

Because of the log-scale, it is tricky to draw conclusions on the absolute impact of each type of result. Moreover, the error bars consistently overlap with each other, further impacting the conclusions that can be drawn. At best, we can draw general conclusions where error bars do not overlap, and identify a general trend. For example, we can probably conclude that medical letters take longer than laboratory results (no overlap), but cannot confidently say whether  admin or screening takes longer (due to overlap in error bars).

Perhaps more interestingly, this model could serve as a foundation to build a predictive tool that lets the GP ballpark how long different workloads will take to complete. This could be used to improve workflow efficiency, or possibly for advertising/marketing purposes.