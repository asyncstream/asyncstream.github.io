---
layout: post
title:  "Springboot with Cassandra"
author: PrincipalStream
comments: true
excerpt_separator: <!--more-->
categories: [Spring Boot]
tags: [Spring Boot, Cassandra]
---

## Overview

In this blog post we will focus on setting up a project with cassandra and spring boot.
Apache Cassandra is an open-source, distributed, wide cloumn store which handles large amount of data, especially time series data.

## Maven Dependencies

        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.1.4.RELEASE</version>
        </parent>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-cassandra</artifactId>
        </dependency>
        <dependency>
            <groupId>com.datastax.cassandra</groupId>
            <artifactId>cassandra-driver-extras</artifactId>
            <version>3.1.4</version>
        </dependency>

## Create Cassandra table

        CREATE TABLE IF NOT EXISTS cloudmessagestream.tbl_cloud_message(
            sender text,
            receiver text,
            id timeuuid,
            message text,
            PRIMARY KEY (sender,id)
        )WITH CLUSTERING ORDER BY (id DESC);

## Cassandra DB Configuration 

The configuration of connection factory is given in the DbConfiguration class. This class extends the class AbstractCassandraConfiguration of spring data framework.
The configuration values for cassandra connection is loaded from application.yml file.
        
        spring:
            data:
                cassandra:
                cluster-name: cloudmessagecluster
                contact-points: localhost
                port: 9042
                schema-action: CREATE_IF_NOT_EXISTS
                jmx-enabled: false


## About the sample project

CloudMessage stream is captures the messages between a sender and receiver. The database of this project has a single table to capture the message and the key of this table is a composite key consiting of sender(text) and id(timeuuid).

The domain entities are annotated with the spring data annotation @Table. It is always good to specify the actual table name in the annotation.

        @Table("tbl_cloud_message")
        public class CloudMessageEntity

As the primary key is a composite key , create a separate class to hold the values/fields of the primary key and annotate it with @PrimaryKeyClass

The first field in the primary is the partition key and the remaining fields are clustering keys.    

        @PrimaryKeyColumn(name = "sender", ordinal = 0, type = PrimaryKeyType.PARTITIONED)
        private String sender;

        @PrimaryKeyColumn(name = "id", ordinal = 1, type = PrimaryKeyType.CLUSTERED, ordering = Ordering.ASCENDING)
        private UUID id;

As in any spring data projects the repository class handles the database CRUD operations.In this project , the class CloudMessageRepository handles the CRUD operations.

        public interface CloudMessageRepository extends CrudRepository<CloudMessageEntity, CloudMessageKey> {

            List<CloudMessageEntity> findByPkSender(String sender);

        }

## Testing the project

I have not used Embedded cassandra as the main objective of this project is to show connectivity to a separately running cassandra database.

        @RunWith(SpringRunner.class)
        @SpringBootTest
        public class CloudMessageServiceTest {

            @Autowired
            CloudMessageService cloudMessageService;

            @Test
            public void createMessageTest(){
                CloudMessage message = new CloudMessage();
                message.setSender("tester1@asyncstream.com");
                message.setReceiver("tester2@asyncstream.com");
                message.setMessage("Hello World");
                cloudMessageService.create(message);
            }

        }

## Conclusion
In this article, we looked at the cloudmessage-stream project to get a quick overview of setting up a spring boot with cassandra database. The source code of this project is available in the [cloudmessage stream Github repository](https://github.com/asyncstream/cloudmessage-stream).