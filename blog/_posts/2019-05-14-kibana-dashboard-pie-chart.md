---
layout: post
title:  "Elastic Stack - Kibana Pie Chart"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Elastic Series,Big Data]
tags: [Elasticsearch, Kibana]
---

## Introduction

In the article [Elastic Stack - Setting up a dashboard in Kibana for data analysis](/blog/kibana-dashboard-for-log-analysis) we have created a simple data table to view the number of trips during normal hours, rush hours and overnight.

In this article, we will create a Pie Chart to depict it which gives a much better understanding of the data.

## Use Case

The Taxi operators would like to know the percentage of trips during normal hours, rush hours and overnight so that they can deploy appropriate number of taxis.
In this case we can use the trip_period fields which we have dreived and the count of trips to depict it in a pie chart. 

## Creating Pie Chart in Kibana

1. Go to the visualization page.
2. Select the index which has the taxi trip data.
3. Select the Pie Chart 

__Configure Metrics__

In the Aggregation filed select "Count" and give the custom label "Number of Trips". 

__Buckets Configuration__

In the Buckets configuration choose Split Slices and then use Terms aggregation on trip_period field. 

![](/assets/img/elastic/taxi_trip_pie_chart_trip_period.png)

Play the pie chart configuration and then save it with the name "Trip Period Chart"

## Dashboard Setup

We will be using the same Dashboard which we have created in the article [Elastic Stack - Setting up a dashboard in Kibana for data analysis](/blog/kibana-dashboard-for-log-analysis).
Click on the Add button and select the "Trip Period Chart" visualization panel and then save the dashboard.  

## Conclusion

As we have seen, we are now able to configure Pie chart with fields in the document.

__**Your feedback is a Reward!**__