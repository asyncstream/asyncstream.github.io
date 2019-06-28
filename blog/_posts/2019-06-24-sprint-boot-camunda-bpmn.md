---
layout: post
title:  "How to organize BPM project with Spring boot & Camunda having separate modules for each process."
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Spring Boot]
tags: [Spring Boot, Camunda]
---

## Overview

In this blog post we will focus on setting up a BPM project using spring boot and camunda BPMN engine and how to create independent sub modules for each business processes.

## Maven Configuration

Create a simple maven project and then add the camunda bpm spring boot starter dependency to the project. Add the spring boot jdbc starter and the database driver dependency.

        <dependency>
            <groupId>org.camunda.bpm.springboot</groupId>
            <artifactId>camunda-bpm-spring-boot-starter</artifactId>
            <version>${camunda.spring.boot.starter.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>


## Project Structure 

        cloudmessage-bpm
            |
            |
            ----> application
                    |
                    -----> resources
                            |
                            ----> META-INF
                                    |
                                    ----> processes.xml

            ----> customer-registration-process
                    |
                    -----> resources
                            |
                            ----> bpmn
                                    |
                                    ----> customer-registration.bpmn

            ----> customer-registration-service-adapter
            ----> customer-cessation-process 
                    |
                    -----> resources
                            |
                            ----> bpmn
                                    |
                                    ----> customer-cessation.bpmn
            ----> customer-cessation-service-adapter
            pom.xml (root pom)

The application sub module contains the spring boot initializer application class and the processes.xml which configures the path of bpmn files.

processes.xml

        <process-archive name="customer-registration-process">
            <resource>bpmn/customer-registration.bpmn</resource>
            <properties>
                <property name="isDeleteUponUndeploy">false</property>
                <property name="isScanForProcessDefinitions">true</property>
            </properties>
        </process-archive>
        <process-archive name="customer-cessation-process">
            <resource>bpmn/customer-cessation.bpmn</resource>
            <properties>
                <property name="isDeleteUponUndeploy">false</property>
                <property name="isScanForProcessDefinitions">true</property>
            </properties>
        </process-archive>

Name of the process-archive in the processess.xml should be the name of the artifact which is included in the application jar file. In this case the sub modules customer-registration-process and customer-cessation-process produces the dependency jars (process archives) which has the corresponding bpmn files.  

Each business process has its own process module and service adapter module. The process module has the process task delegate classes and the bpmn files. 

The service adapter provides the implementation of services which is to be called from process task delegates.


Example of Task delegate

        public class CustomerRegistration implements JavaDelegate {

            private final Logger LOG = LoggerFactory.getLogger(CustomerRegistration.class);
            private CustomerRegistrationService customerRegistrationService;

            public  CustomerRegistration(CustomerRegistrationService customerRegistrationService){
                this.customerRegistrationService = customerRegistrationService;
            }

            @Override
            public void execute(DelegateExecution delegateExecution) throws Exception {
                RegistrationRequest registrationRequest = new RegistrationRequest();
                RegistrationResult registrationResult = this.customerRegistrationService.register(registrationRequest);
                LOG.info("Customer registration is completed with id:: {}",registrationResult.getRegistrationId());
            }
        }

## Conclusion

In this article, we looked at the setup of a simple bpm project using camunda and spring boot. In this sample project I tried to explain how processes.xml can be used to create separate jar files for each business processes.
The code for this article can be found in the [cloudmessage bpm Github repository](https://github.com/asyncstream/cloudmessage-bpm).