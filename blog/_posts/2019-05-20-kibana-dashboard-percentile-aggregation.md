---
layout: post
title:  "Elastic Stack - Kibana Percentile Aggregation"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Elastic Series,Big Data]
tags: [Elasticsearch, Kibana]
---

## Introduction

In this article, we will create a Perceteil Table to show the 50,75,95 percentiles data of trip tips.

## Use Case

The Taxi operators would like to know the 50,75,90 percentiles of trip tips during different period of travel. 

## Creating Pie Chart in Kibana

1. Go to the visualization page.
2. Select the index which has the taxi trip data.
3. Select the Data Table 

__Configure Metrics__

In the Aggregation filed select "Count" and give the custom label "Trip Count". Add another metric of aggregation "Percentiles" on the field "tip_amount". As the operator wants to know 50, 75, 90 percentiles select those in the percent boxes.

__Buckets Configuration__

Select pickup and drop off locations in the bucket configuration as it will give the percetile of tip between two locations. If we add the 3rd bucket on trip period it will give a better picture on tip amount given for the travel between two locations during normal,rush and overnight periods.


![](/assets/img/elastic/taxi_tip_95percentile_report.png)

Play the data table configuration and then save it with the name "Trip Tip Percentile"

## Dashboard Setup

We will be using the same Dashboard which we have created in the article [Elastic Stack - Setting up a dashboard in Kibana for data analysis](/blog/kibana-dashboard-for-log-analysis).
Click on the Add button and select the "Trip Tip Percentile" visualization panel and then save the dashboard.  

## Conclusion

As we have seen, we are now able to configure Percentile Data Table with fields in the document.

__**Your feedback is a Reward!**__