---
layout: post
title:  "Automation testing with Cucumber and Spring Boot"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Spring Boot, Testing]
tags: [Spring Boot, Cucumber]
---

## Overview

In this post, we will create a simple integration-test project to learn how to use cucumber packages along with spring. This project is a sub project of cloudmessage-fabric project and it uses rule sub project.

Cucumber is a software tool which runs automated acceptance tests written in business-readable format. Gherkin is the business specification format which Cucumber interprets. 

## Maven Configuration

        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java8</artifactId>
            <version>${cucumber.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit</artifactId>
            <version>${cucumber.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-spring</artifactId>
            <version>${cucumber.version}</version>
            <scope>test</scope>
        </dependency>

## Feature File & Gherkin

A feature file is where you write various scenarios involved in a product feature. This file will be written in descriptive language so that both business and tester can use it as a blueprint document. Feature files for cucumber are written in Gherkin format and it has the below given keywords

        Feature: Defines what feature you will be testing in the tests below
        Description: This is an optional field but can be use to describes the test in detail.
        Background: Define a step or series of steps which are common to all the tests in the feature file.
        Given: Tells the pre-requisite of the test
        When: Defines the condition of the test
        And: Defines additional conditions of the test
        Then: States the post condition. You can say that it is expected result of the test.

In order for Cucumber to automatically identify the feature files given the extention .feature. In this project, i have created a feature file [free-tier.feature]() to test the feature of message charging for free tier customer.

        @messagecharging
        Feature: Calculate charge for free tier customers
        Description: The purpose of this feature is to calculate the message charge for free tier customers
        Background:
            Given the license type is LIC-FREE
        @important
        Scenario: Calculate the charge for message count < 100000
            When the message count is 30000
            Then calculate the charge
            Then the charge is 0.0
        Scenario: Calculate the charge for message count = 100000
            When the message count is 100000
            Then calculate the charge
            Then the charge is 0.0


## Create Automation Test

Create a Spring boot test class which has the bean configurations required in the test. This class is the base for all cucumber feature tests.

RuleEngineTestBase.java [View](https://github.com/asyncstream/cloudmessage-fabric/blob/master/integration-tests/src/test/java/com/asyncstream/cloudmessage/it/ruleengine/RuleEngineTestBase.java)

        @ContextConfiguration(classes = {RuleEngineConfig.class})
        @SpringBootTest
        public abstract class RuleEngineTestBase {

            @Autowired
            protected RuleEngine ruleEngine;

        }

Create a step definition file corresponds to the feature file. This class makes the actual calls to the feature implementation logic/methods. This class gets the scenarios from the feature file and extract the actual inputs for the test execution.

MessageChargingRuleStepDef.java [View](https://github.com/asyncstream/cloudmessage-fabric/blob/master/integration-tests/src/test/java/com/asyncstream/cloudmessage/it/ruleengine/messagecharging/MessageChargingRuleStepDef.java)

        public class MessageChargingRuleStepDef extends RuleEngineTestBase implements En {

            private String licenseType;
            private Long messageCount;
            private Double result;

            public MessageChargingRuleStepDef(){

                Given("the license type is LIC-FREE", () -> {
                    this.licenseType = "LIC-FREE";
                });

                When("the message count is {long}", (Long messageCount) -> {
                    this.messageCount = messageCount;
                });

                Then("calculate the charge", () -> {
                    MessageChargingCriteria messageChargingCriteria = new MessageChargingCriteria(this.licenseType,this.messageCount);
                    MessageChargeResult messageChargeResult=this.ruleEngine.executeMessageChargingRule(MESSAGE_CHARGE_RULE, messageChargingCriteria);
                    this.result = messageChargeResult.getResult();
                });

                Then("the charge is {double}", (Double expectedResult) -> {
                    Assert.assertEquals(expectedResult,this.result);
                });
            }
        }

Add the Test file which glues the feature file with step definition class and run the test cases.

MessageChargingRulesIT.java [View](https://github.com/asyncstream/cloudmessage-fabric/blob/master/integration-tests/src/test/java/com/asyncstream/cloudmessage/it/ruleengine/messagecharging/MessageChargingRulesIT.java)

        @RunWith(Cucumber.class)
        @CucumberOptions(features = "src/test/resources/rules-test/messagecharging/free-tier.feature",
                glue = "com.asyncstream.cloudmessage.it.ruleengine.messagecharging",
                plugin = {"pretty", "html:target/cucumber"})
        public class MessageChargingRulesIT {
        }

When you run the MessageChargingRulesIT class , you will get the result similar to the one given below.

        @messagecharging
        Feature: Calculate charge for free tier customers
        Description: The purpose of this feature is to calculate the message charge for free tier customers

        Background:                          
            Given the license type is LIC-FREE # MessageChargingRuleStepDef.java:18

        @messagecharging @important
        Scenario: Calculate the charge for message count < 100000 
            When the message count is 30000                        
            Then calculate the charge                               
            Then the charge is 0.0                                  

        Background:                         
            Given the license type is LIC-FREE # MessageChargingRuleStepDef.java:18

        @messagecharging
        Scenario: Calculate the charge for message count = 100000
            When the message count is 100000                        
            Then calculate the charge                               
            Then the charge is 0.0                                  

        2 Scenarios (2 passed)
        8 Steps (8 passed)
        0m2.679s

## Conclusion

In this article, we looked at the setup of automation testing using Cucumber and Gherkin. The code for this article can be found in the [cloudmessage fabric Github repository](https://github.com/asyncstream/cloudmessage-fabric).
