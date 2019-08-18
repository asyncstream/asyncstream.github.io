---
layout: post
title:  "Camunda BPMN testing with Mockito"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Spring Boot]
tags: [Spring Boot, Camunda, Mockito]
---

## Overview

Unit testing of a BPMN is not simple as that of a java class. Camunda mockito extension has provided a simple implementation to create unit tests for BPMN.
In this article we will walk through a sample test case of customer activation BPMN in the cloudmessage platform.

## Maven Configuration


        <dependency>
            <groupId>org.camunda.bpm.extension.mockito</groupId>
            <artifactId>camunda-bpm-mockito</artifactId>
            <version>4.10.0</version>
            <scope>test</scope>
        </dependency>

## BPMN Diagram

In the customer activation process, systems sends a message to the customer with activation link and then wait for the customer action.
When the customer clicks on the link a message will be sent back to the system to resume the process.

![](/assets/img/bpmn/customer-activation.png)

## Testcase Implementation

The full implementation of this testcase can be viewed [here](https://github.com/asyncstream/cloudmessage-stream/blob/master/src/test/java/com/asyncstream/cloudmessage/stream/service/impl/CloudMessageServiceTest.java)

First step is to create a process engine.

        @Rule
        public final ProcessEngineRule processEngineRule = new ProcessEngineRule(testAwareProcessEngineConfiguration().buildProcessEngine());

As this BPMN has message send and recieve tasks , we need to create two constants for message name and recieve task id.

        private static final String MESSAGE_ACTIVATION_RESULT = "Message_Activation_Result";
        private static final String RECIEVE_ACTIVITY="process_activation_result";

In the test method call the autoMock for the bpmn process which registers mocks for TaskListner, ExecutinListner and JavaDelegate.

        autoMock("bpmn/customer-activation.bpmn");

Asserts that all the necessary mocks are available.

        assertThat(Mocks.get("startActivation")).isNotNull();
        assertThat(Mocks.get("processActivationResult")).isNotNull();
        assertThat(Mocks.get("completeActivation")).isNotNull();
        assertThat(Mocks.get("sendActivationRequest")).isNotNull();

Configure the deployment with a mock to the recieve message activity.

        processEngineRule.manageDeployment(registerCallActivityMock(RECIEVE_ACTIVITY)
            .onExecutionWaitForMessage(MESSAGE_ACTIVATION_RESULT)
            .deploy(processEngineRule));            

Verify the mocks for the first to activities using the verify methods of DelegateExpressions.
    
        verifyExecutionListenerMock("startActivation").executed(); // For Listeners
        verifyJavaDelegateMock("sendActivationRequest").executed(); // For Delegates

As the 3rd stage of the BPMN is message receieve activity, automate it by correlateMessage with message name.

        processEngineRule.getRuntimeService().correlateMessage(MESSAGE_ACTIVATION_RESULT);

## Conclusion

In this article, the testing of a BPMN process using camunda and mockito is covered with a simple project setup . The source code of this project is available in the [cloudmessage stream Github repository](https://github.com/asyncstream/cloudmessage-stream).