---
layout: post
title:  "Create a simple rule engine with Spring Boot and Camunda DMN"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Spring Boot]
tags: [Spring Boot, Camunda]
---

## Overview

In this blog post we will focus on setting up a simple rule engine using spring boot and camunda DMN engine.

Spring boot is a wrapper project on top of the Spring framework which adds the required dependencies and auto configure all the features you want to start the application. In the world of micro services Spring boot saves your time to setup a project.

DMN, Decision Model and Notation, is a standard published by [OMG](https://www.omg.org/) which provides modeling notation for decision that will support business decision management and business rules.

There are many provides of DMN engine. In this article we will use Camunda DMN engine to setup our cloud message charging rules project.

## Maven Configuration

Create a project with [Spring Initializr](https://start.spring.io/) so that you will get a basic setup of the project. Add the camunda engine dependency to the project.

        <dependency>
            <groupId>org.camunda.bpm.dmn</groupId>
            <artifactId>camunda-engine-dmn</artifactId>
        </dependency>


## Rule Engine API Objects

Create the model objects which holds the values of rule criteria and result

RuleCriteria.java

        public abstract class RuleCriteria {
            protected Map<String,Object> contextMap=new HashMap<>();

            public abstract Map<String,Object> contextMap();
        }

RuleResult.java -- [View full code here](https://github.com/asyncstream/cloudmessage-fabric/blob/master/cloudmessage-rule-api/src/main/java/com/asyncstream/cloudmessage/rule/api/model/RuleResult.java)

        public class RuleResult<T> {
            private String ruleCode;
            private T result;
        }

As this project is about cloud message charging create the criteria and result objects which extends the above classes

MessageChargingCriteria.java -- [View full code here](https://github.com/asyncstream/cloudmessage-fabric/blob/master/cloudmessage-rule-api/src/main/java/com/asyncstream/cloudmessage/rule/api/model/MessageChargingCriteria.java)

        public class MessageChargingCriteria extends RuleCriteria{
            private final String LICENSE_TYPE="licenseType";
            private final String MESSAGE_COUNT="messageCount";

            private final String licenseType;
            private final long messageCount;

            public MessageChargingCriteria(String licenseType, long messageCount) {
                this.licenseType = licenseType;
                this.messageCount = messageCount;
            }
        }    

MessageChargeResult.java

        public class MessageChargeResult extends RuleResult<Double> {

            public MessageChargeResult(String ruleCode, Double result) {
                super(ruleCode, result);
            }

        }

[RuleEngine.java](https://github.com/asyncstream/cloudmessage-fabric/blob/master/cloudmessage-rule-api/src/main/java/com/asyncstream/cloudmessage/rule/api/engine/RuleEngine.java) interface has required methods to execute the rules. 

        public MessageChargeResult executeMessageChargingRule(String code, MessageChargingCriteria criteria);

## Rule Engine Implementation

The DMN file [MESSAGE-CHARGE.dmn](), whose representation is in XML holds the business decision rules , resides in the dmn folder of resources.

This dmn is loaded into a [Rule Registry](https://github.com/asyncstream/cloudmessage-fabric/blob/master/cloudmessage-rule-impl/src/main/java/com/asyncstream/cloudmessage/rule/impl/RuleRegistry.java) when the rule engine starts. This registry has a concurrent hash map to hold the dmn decision objects. 

        private InputStream ruleFileStream(String rulePath){
            Path path = Paths.get(rulePath);
            if(Files.isReadable(path)){
                try{
                    return Files.newInputStream(path, StandardOpenOption.READ);
                }catch (IOException ioe){
                    throw new RuleEngineException("Error while reading the DMN file definition from path: "+rulePath,ioe);
                }
            }
            return this.getClass().getResourceAsStream(rulePath);
        }

        private DmnDecision createDmnDecision(String ruleCode){
            String rulePath=new StringBuilder(this.rulesRootPath).append("/").append(ruleCode).append(".dmn").toString();

            InputStream input = ruleFileStream(rulePath);
            return this.dmnEngine.parseDecision(ruleCode,input);
        }

        private void register(String ruleCode){this.registry.put(ruleCode,createDmnDecision(ruleCode));}

Each dmn rule has an executor object which execute the rule and return the result back to the caller

MessageChargingRuleExecutor.java extends [RuleExecutor](https://github.com/asyncstream/cloudmessage-fabric/blob/master/cloudmessage-rule-impl/src/main/java/com/asyncstream/cloudmessage/rule/impl/RuleExecutor.java)

        public class MessageChargingRuleExecutor extends RuleExecutor<MessageChargeResult>{

            public MessageChargingRuleExecutor(final RuleRegistry registry,final DmnEngine dmnEngine) {
                super(registry,dmnEngine);
            }

            @Override
            public MessageChargeResult execute(DmnDecision decision, Map<String, Object> contextMap) {
                try {
                    final DmnDecisionTableResult tableResult = this.getDmnEngine().evaluateDecisionTable(decision, contextMap);
                    final DmnDecisionRuleResult ruleResult = tableResult.getSingleResult();
                    final Double charge = (Double) ruleResult.get("charge");
                    return new MessageChargeResult( decision.getKey(),charge);
                } catch (NullPointerException npe) {
                    return new MessageChargeResult(decision.getKey(),0d);
                }
            }
        }

RuleEngineImpl class provides the concrete implementation of the methods in RuleEngine.

        public class RuleEngineImpl implements RuleEngine {

            private final MessageChargingRuleExecutor messageChargingRuleExecutor;

            public RuleEngineImpl(MessageChargingRuleExecutor messageChargingRuleExecutor){
                this.messageChargingRuleExecutor=messageChargingRuleExecutor;
            }

            @Override
            public MessageChargeResult executeMessageChargingRule(String code, MessageChargingCriteria criteria) {
                return this.messageChargingRuleExecutor.execute(code,criteria);
            }
        }

The configuration class,[RuleEngineConfig.java](https://github.com/asyncstream/cloudmessage-fabric/blob/master/cloudmessage-rule-impl/src/main/java/com/asyncstream/cloudmessage/rule/impl/RuleEngineConfig.java) of the rule engine project registers the beans for DmnEngine, RuleEngine, RuleRegistry and the executors.

## Conclusion

In this article, we looked at the setup of a simple rules projects using camunda dmn engine and spring boot. The code for this article can be found in the [cloudmessage fabric Github repository](https://github.com/asyncstream/cloudmessage-fabric).