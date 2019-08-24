---
layout: post
title:  "Camunda BPMN - Invoking sub process with call activity"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [BPM, Camunda]
tags: [Spring, Camunda, Tomcat]
---

## Overview
There are two ways to call sub process in camunda , either using an embedded process in the same main process or using call activity. In this article we will walk through the diagram and code to create and invoke sub process using call activity.

## BPMN Diagram

The bpmn diagram is given in the article [Camunda - Deploy process in a shared camunda tomcat server](../deploy-bpmn-process-in-camunda-tomcat.html)

When the bundle ordering process is triggered , it internally trigger single item process from the step Single Order Process as many time as the number of individual order items.

## Configuration

In order to trigger sub process , the activity type of the Single Order Processing task should be "Call Activity". The next question come to the mind is whether the processing of single item can be done in parallel or sequential. Based on the requirement select the parallel or sequential multiple instance.

Since in this usecase, we are using an independent subprocess, bind the call activity task with the subprocess using the process id.

![](/assets/img/bpmn/bpmn_call_activity_settings.png)

The Order object has multiple order items, use it as the collection variable to the multi instance configuration. Given a variable name in the Element Variable field to which each oder item will be assigned during iterating the collection.

        for(OrderItem orderItem:order.orderItems){

        }

![](/assets/img/bpmn/call_activity_multi_instance.png)

By the above configuration, we will be able to invoke the subprocess from the main process. Now it is the time to think about how to pass the variable orderItem to the subprocess. 
In the variables section of the the BPMN process, configure the variables which need to be passed to the sub process.

![](/assets/img/bpmn/call_activity_variables.png)

If you wants to send an inner object in the order,eg:shipping address, this can be achieved by selecting Source Expression as the mapping type.

![](/assets/img/bpmn/call_activity_variables_mapping.png)


## Conclusion

In this article, calling a sub process in camunda using call activity is explained with a simple project setup. The complete source code of this article is available in the [github repository](https://github.com/asyncstream/bundle-ordering-bpm).

__**Your feedback is a Reward!**__
