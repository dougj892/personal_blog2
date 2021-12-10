---
title: Three Stage Sampling
author: Doug
date: '2017-10-04'
slug: 3-stage
categories: []
tags:
  - Statistics
---


*Caveat emptor: This blog post has not been thoroughly checked for errors.*

One of IDinsight's project teams is in the process of designing the sampling strategy for a large scale household survey and is considering using a three stage sampling design in which they would first select districts, then villages (or urban wards), and then households.  In addition, someone was asking about three stage clustering for an RCT somewhere on Slack (I can't seem to find the slack post now) so I thought it might be useful to write a short post on three stage designs.  

In this post, I'll try to answer four questions:
1. When do you need to take into account both stages of clustering in a survey or evaluation?
2. How do you properly account for a three stage design when performing sample size / power calculations?
3. How should you estimate the inputs required for these calculations?
4. How do you properly account for a three stage design when analyzing data?

# When you do you need to take into both stages?
With an RCT, it's pretty rare that you really need to take into account two stages of clustering. Remember that just because units exhibit some sort of clustering doesn't mean that you need to adjust for clustering in your analysis.  For example, if you randomize at the student level it doesn't matter that student learning outcomes exhibit clustering at the classroom level.  An example of when you might want to take into account two stages of clustering is when you randomize large clusters (e.g. schools) and then only collect data from units in a randomly sampled set of smaller clusters (e.g. kids in classrooms).  With surveys, anytime you have a three stage design you should theoretically take into account the clustering at both levels.  

Even when it makes sense in theory to take into account both stages of clustering, you can usually get by with just considering the most aggregate (highest) level of clustering.  We'll see below why that makes sense.  In some cases, e.g. when you are trying to find the optimal survey design for a given budget, you do really need to take into account both stages of clustering.

# How do you account for a three stage design?
*(The advice given below is tailored to someone performing sampling size calcs for a survey with a three stage design.  All of the advice holds true for power calcs as well.  You just need to multiply the final variance by 2 (since you have 2 groups -- treatment and control) and then use the standard adjustment to the standard error for power calcs -- i.e. instead of multiplying the standard error by +/-1.96 to create a 95% confidence interval you multiply by ~2.8 to calculate an MDE for alpha .05 and power .8.)*

Let's first recap how one stage of clustering affects the variance of your estimator.  Let's say that you will use a two stage sampling strategy in which you will first randomly sample J clusters and then randomly sample K units from each cluster to estimate the mean of some variable y.  Further assume that the total number of units per cluster does not vary and is pretty large.  If values of y are correlated within each cluster, we can think of the values for y as being made up of a cluster component and an independent within-cluster component, i.e.

$$y_{j,k}=\eta_j+\phi_{j,k}$$

This allows us to calculate the variance the of y as:

$$\sigma^2_y=\sigma^2_{\eta}+\sigma^2_{\phi}$$

And the variance of the mean as:

$$Var(\bar{y})=\frac{\sigma^2_{\eta}}{J}+\frac{\sigma^2_{\phi}}{JK}=\sigma^2_y\left(\frac{\rho}{J}+\frac{(1-\rho)}{JK}\right)$$

Where \\( \rho=\frac{\sigma^2_{\eta}}{\sigma^2_y} \\).  It's also useful to calculate the design effect, or the ratio of the variance of this estimator to the ratio of the estimator if the sample had been collected using simple random sampling (SRS). Since the variance under SRS would be \\( \frac{\sigma^2_y}{JK} \\) the design effect\\(=1+(K-1)\rho\\).

Let's now suppose that we have a higher level sampling stage. We first pick Q mega-clusters, then J clusters from each mega-cluster, and then K households from each cluster.  Similarly, we can think of the values y as made of three components:

$$y_{q,j,k}=\gamma_q+\eta_{q,j}+\phi_{q,j,k}$$

The variance of y is then:

$$\sigma^2_y=\sigma^2_{\gamma}+\sigma^2_{\eta}+\sigma^2_{\phi}$$

And the variance of the mean is:

$$Var(\bar{y})=\frac{\sigma^2_{\gamma}}{Q}+\frac{\sigma^2_{\eta}}{QJ}+\frac{\sigma^2_{\phi}}{QJK}=\sigma^2_y\left(  \frac{\rho_{\gamma}}{Q}+\frac{\rho_{\eta}}{QJ}+\frac{(1-\rho_{\gamma}-\rho_{\eta})}{QJK} \right)$$

Where \\(\rho_{\eta}=\frac{\sigma^2_{\eta}}{\sigma^2_y}\\) and \\(\rho_{\gamma}=\frac{\sigma^2_{\gamma}}{\sigma^2_y}\\).  For our three stage sampling design, the design effect is:

$$DEFF=1+(K-1)\rho_{\eta}+(JK-1)\rho_{\gamma}$$

This also shows why just looking at the most aggregate level of clustering is usually pretty reasonable -- assuming the two ICCs are relatively similar in size, the adjustment to the variance will be driven primarily by the most aggregate level of clustering.  

# How should you estimate the inputs required for these calculations?
Now that we know how to adjust our estimate of the variance using the ICC at both levels, the next obvious questions is where to find the different values for the ICC.  

If you are lucky, you might find a dataset which includes both levels of clustering and the variable you are interested (or some similiar variable).  If so, then you can use Stata's anova command to estimate the two ICCs.  I'm not really sure how to do this is Stata, but I think that it should be relatively straightforward if you search the help file for "nested anova."  (And if you figure out how to do it please let me know!) 

Alternatively, you can resort to the typical hack of using a design effect calculated from another survey with a similar three stage design.  [Here](https://unstats.un.org/unsd/hhsurveys/pdf/Chapter_7.pdf) are a few design effects from which to draw from. (As an aside, it would also be useful for us to start recording the design effects for key variables from our own surveys.  To get the design effect for a survey dataset in Stata first "svyset" your dataset, then estimate the population mean of a variable using "svy: mean <variable>", and then run "estat effects" to get the design effect for the estimate.)

# How do you properly account for a three stage design when analyzing data?
Unfortunately, most Stata commands only allow for a single stage of clustering.  To account for two or more stages of clustering, you need to first "svyset" your data and then use the "svy" prefix before running any command. 




