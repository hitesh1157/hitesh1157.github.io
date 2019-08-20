---
layout: post
title: Concept of Distributed Tracing
published: true
---

Distributed Tracing/Monitoring - very useful tooling for debugging and trying to understand where things go wrong. If you have a very deep chain of microservices, a request comes in to the frontend, it gets passed around. You don't really know where potential slowdowns are; where things are going wrong unless you have some sort of tracing. Unless you are able to put a tag or an identifier as a request comes into your system and you are able to pass that tag all around to your different services and everyone is logging etc. so you can search for it. If you have logs go to a central place you can search for the lifecycle of a given request. And that's a good plug for an open source project called [Zipkin](https://zipkin.io/).

> Zipkin is a distributed tracing system. It helps gather timing data needed to troubleshoot latency problems in microservice architectures. It manages both the collection and lookup of this data. Zipkin’s design is based on the [Google Dapper paper](https://ai.google/research/pubs/pub36356).
