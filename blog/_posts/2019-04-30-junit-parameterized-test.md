---
layout: post
title:  "Parameterized tests in Junit"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Unit Testing]
tags: [Junit]
---

## Overview
Junit has provided a runner class for parameterized tests. This help a programmer to create unit test with a collection of test data. This will avoid writing multiple test cases with various input values for a single business function.

In this post, we will focus on the Parameterized test case implementation for a RuleEngine function. The rule engine is explained in the article ["Create a simple rule engine with Spring Boot and Camunda DMN"](https://asyncstream.com/blog/sprint-boot-camunda-dmn.html).

## Unit Test Implementation

Create a rule engine test base when it creates a DMN engine which can be used with all the child test cases.

        public abstract  class RuleEngineTestBase {
            protected static String rulesRootPath="/dmn/";
            protected static DmnEngine dmnEngine;

            protected String name;
            protected VariableMap inputParameters;

            static {
                DmnEngineConfiguration config = DmnEngineConfiguration.createDefaultDmnEngineConfiguration();
                dmnEngine = config.buildEngine();
            }
        }

Create a test base for message charging rule. This class served as the parent class for for all testcases for Free,Gold & Preminum tier customers.

        @RunWith(Parameterized.class)
        public class MessageChargingTestBase extends RuleEngineTestBase {

            private static DmnDecision dmnDecision;

            protected double expectedCharge;

            protected static void start() {
                InputStream input = RuleEngineTestBase.class.getResourceAsStream(rulesRootPath+"MESSAGE-CHARGE.dmn");
                dmnDecision = dmnEngine.parseDecision(MESSAGE_CHARGE_RULE, input);
            }

            @Test
            public void validateCharge() {
                Double result = dmnEngine.evaluateDecision(dmnDecision, inputParameters).getSingleResult().getEntry("charge");
                Assertions.assertThat(result).isEqualTo(expectedCharge);
            }
        }


Create the unit test to test the message count charge for Free Tier customer. In this case the testCaseData method creates an array of test data required to test all the scenarios of Free Tier. 
The constructor of this class accepts the inputs, expected result value and the name of the scenario.The parameters in the constructor is exactly matches a single record in the test data array

    @RunWith(Parameterized.class)
    public class MessageChargingFreeTierRuleTest extends MessageChargingTestBase {

        @BeforeClass
        public static void init() {
            start();
        }

        public MessageChargingFreeTierRuleTest(String licenseType, long messageCount, double expectedCharge, String name) {
            this.name = name;
            inputParameters = Variables.createVariables()
                    .putValue("licenseType", licenseType)
                    .putValue("messageCount", messageCount);
            this.expectedCharge = expectedCharge;
        }

        @Parameterized.Parameters(name = "{4}")
        public static Collection<Object[]> testCaseData() {
            return Arrays.asList(new Object[][]{
                    {"LIC-FREE", 100000,0, "Free Tier::Calculate charge for message count < 1000000"},
                    {"LIC-GOLD", 1000000,0, "Free Tier::Calculate charge for message count = 1000000"}

            });
        }

    }

When you run "mvn test" in the terminal you could see the test result as given below.

        [INFO]  T E S T S
        [INFO] -------------------------------------------------------
        [INFO] Running com.asyncstream.cloudmessage.rule.engine.messagecharging.MessageChargingFreeTierRuleTest
        [INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.637 s - in com.asyncstream.cloudmessage.rule.engine.messagecharging.MessageChargingFreeTierRuleT
        est
        [INFO] Running com.asyncstream.cloudmessage.rule.engine.messagecharging.MessageChargingGoldTierRuleTest
        [INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0 s - in com.asyncstream.cloudmessage.rule.engine.messagecharging.MessageChargingGoldTierRuleTest
        [INFO]
        [INFO] Results:
        [INFO]
        [INFO] Tests run: 4, Failures: 0, Errors: 0, Skipped: 0
        [INFO]


## Conclusion

In this article, we looked at the setup of parameterized testing using JUnit. The code for this article can be found in the [cloudmessage fabric Github repository](https://github.com/asyncstream/cloudmessage-fabric).