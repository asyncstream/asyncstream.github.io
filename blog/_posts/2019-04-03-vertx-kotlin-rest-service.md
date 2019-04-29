---
layout: post
title:  "Create a simple rest service with Vert.x & Kotlin - Part1"
author: Principal Streamer
comments: true
excerpt_separator: <!--more-->
categories: [Vert.x, Kotlin]
tags: [Vertx, Kotlin, Rest Service]
---

## Overview

In this article, we will walk through the code to setup a REST WebService project in Vert.x using Kotlin Language.
Eclipse Vert.x is a tool-kit for building reactive applications on the JVM. Vert.x is event driven, non blocking and polyglot, which makes it an excellent platform for building microservices.
Vert.x Starter is helpfull in generating starter settings. Add the dependencies Vert.x Kotlin coroutines and Web. 

## Maven Configuration
        <properties>
            <kotlin.coroutines.version>1.1.1</kotlin.coroutines.version>
            <jackson-module-kotlin.version>2.9.8</jackson-module-kotlin.version>
        </properties>
        <dependency>
            <groupId>org.jetbrains.kotlinx</groupId>
            <artifactId>kotlinx-coroutines-core</artifactId>
            <version>${kotlin.coroutines.version}</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.module</groupId>
            <artifactId>jackson-module-kotlin</artifactId>
            <version>${jackson-module-kotlin.version}</version>
        </dependency>        
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-lang-kotlin-coroutines</artifactId>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-web</artifactId>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-config</artifactId>
        </dependency>                

## The Coroutine & Vert.x Verticle

Essentially, coroutines are light-weight threads which are launched in a context of a given scope. The coroutine feature in Kotlin simplify the work of asynchronous tasks and make the code more readable. Suspend and resume is the idea behind coroutine but writting it sequentially to avoid the congnitive load.

A verticle is an execution unit of code (a task) that can be deployed by Vert.x. This is something similar to an actor in the actor model frameworks like Akka. There can be multiple verticles coexists in a JVM and these are communicate each other through an even bus.

## Vert.x Launcher 

In this project we write a Launcher class which create Vert.x instance and deploy a verticle. This class is reading the -conf parameter and creating the corresponding deployment options when deploying verticle specified in the argument list..

Main.kt

        package com.asyncstream.cloudmessage.rest

        import io.vertx.core.Launcher

        fun main() {
            Launcher.executeCommand("run", "-conf", "src/main/resources/application-conf.json", "com.asyncstream.cloudmessage.rest.ResourceAPIVerticle")
        }

## Vert.x CoroutineVerticle for REST Service

CoroutineVerticle is a verticle which run its start and stop methods in coroutine. The ResourceAPIVerticle class in this project extends the CoroutineVerticle of vertx-kotlin-coroutine package.

The start method of this class creates a router which handles the requests landing into this verticle. As the json data mapping is handled by the jackson's KotlinModule register it to the vert.x core.

        override suspend fun start() {
            val router = createRouter()

            val server = vertx.createHttpServer(httpServerOptionsOf()
                    .setSsl(true)
                    .setUseAlpn(true)
                    .setPemKeyCertOptions(
                            PemKeyCertOptions().setKeyPath(location.resolve("ca.key").toString())
                                    .setCertPath(
                                            location.resolve("ca.crt").toString())))

            Json.mapper.registerModule(KotlinModule())
            Json.prettyMapper.registerModule(KotlinModule())

            server.requestHandler { router.handle(it) }
                    .listenAwait(config.getInteger("https.port", 7443))
        }

Creates an extention method on the Route class to handle the task in the coroutine scope.

        private fun Route.coroutineHandler(fn: suspend (RoutingContext) -> Unit): Route {
            return handler { ctx ->
                launch(ctx.vertx().dispatcher()) {
                    try {
                        fn(ctx)
                    } catch (e: Exception) {
                        ctx.fail(e)
                    }
                }
            }
        }

The createRouter method creates the router in vert.x which handles all REST calls to the verticle

        private suspend fun createRouter() = Router.router(vertx).apply {
                    // To get the access token.
                post("/login").handler(BodyHandler.create()).coroutineHandler { rc ->
                    val userJson = rc.bodyAsJson
                    val user = keycloak.authenticateAwait(userJson)
                    rc.response().end(user.principal().toString())
                }

                get("/test").produces("application/json").coroutineHandler {
                    it.response()
                            .putHeader("content-type", "application/json; charset=utf-8")
                            .end(Json.encodePrettily("Test"));
                }                

                // Exception handling code goes here.
        }

Exception raised by wrong resource path and any internal errors are handled in the following code.

        
            route().handler { ctx ->
                ctx.fail(HttpStatusException(HttpResponseStatus.NOT_FOUND.code()))
            }

            route().failureHandler { failureRoutingContext ->
                val statusCode = failureRoutingContext.statusCode()
                val failure = failureRoutingContext.failure()

                if (statusCode == -1) {
                    if (failure is HttpStatusException)
                        response(failureRoutingContext.response(), failure.statusCode, failure.message)
                    else
                        response(failureRoutingContext.response(), HttpResponseStatus.INTERNAL_SERVER_ERROR.code(),
                                failure.message)
                } else {
                    response(failureRoutingContext.response(), statusCode, failure?.message)
                }
            }

## Conclusion

In the part-1 of this article explains the project setup and the code to create a REST service in Vert.x using Kotlin coroutine.
The source code of this article is available in the [github.](https://github.com/asyncstream/cloudmessage-rest-engine.git)
