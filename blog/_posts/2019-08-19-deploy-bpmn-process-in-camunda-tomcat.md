---
layout: post
title:  "Camunda - Deploy process in a shared camunda tomcat server"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [BPM, Camunda]
tags: [Spring, Camunda, Tomcat]
---

## Overview

Camunda documentation clearly explains what are the options to deploy a process. In this article we will go through the steps and configuration of creating a bpm process artifact which can be deployed in a shared , container managed process engine.

## Maven Configuration
        
        <packaging>war</packaging>

        <dependency>
            <groupId>org.camunda.bpm</groupId>
            <artifactId>camunda-engine-spring</artifactId>
            <version>7.11.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.camunda.bpm</groupId>
                    <artifactId>camunda-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.camunda.bpm</groupId>
            <artifactId>camunda-engine</artifactId>
            <version>7.11.0</version>
            <scope>provided</scope>
        </dependency>

## BPMN Diagram

## Implementation

The application must have a process application class in order to delegate the process deployment in the container. Annotate your process application class with the annotation @ProcessApplication provided by camunda.

        @ProcessApplication("Bundle Ordering Process Application")
        public class BundleOrderingProcessApplication extends ServletProcessApplication {
            private  static final Logger LOG = LoggerFactory.getLogger(BundleOrderingProcessApplication.class);

            @PostDeploy
            public void initProcessApplication(ProcessApplicationInfo info){
                LOG.info("Application started");
            }
        }

Register the process application class as a spring bean.

        @Bean
        public BundleOrderingProcessApplication bundleOrderingProcessApplication(){
            return new BundleOrderingProcessApplication();
        }

The main class of the process should implements the WebApplicationInitializer of springframework and register the bean configuration class using AnnotationConfigWebApplicationContext.

        public void onStartup(ServletContext servletContext) throws ServletException {
            AnnotationConfigWebApplicationContext root = new AnnotationConfigWebApplicationContext();
            root.register(ApplicationConfig.class);
            servletContext.addListener(new ContextLoaderListener(root));
        }

# Create Artifact

Create the war artifact using the maven-war-plugin and then deploy it in the tomcat container.

        <build>
            <plugins>
                <plugin>
                    <artifactId>maven-war-plugin</artifactId>
                    <version>3.2.0</version>
                    <configuration>
                        <failOnMissingWebXml>false</failOnMissingWebXml>
                    </configuration>
                </plugin>
            </plugins>
        </build>


## Conclusion

In this article, the deployment of the camunda BPMN process in a shared tomcat container explained with a simple project. The complete source code of this article is available in the [github repository](https://github.com/asyncstream/bundle-ordering-bpm).
