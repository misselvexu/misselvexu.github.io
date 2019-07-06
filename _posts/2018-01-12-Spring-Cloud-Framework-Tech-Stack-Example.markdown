---
layout:     post
title:      "[Develop] Micro-Service-Framework Base ON Spring-Cloud"
subtitle:   "MicroService Example"
date:       2018-01-12 02:59:59
author:     "Elve Xu"
header-img: "img/in-post/ssr-bg.jpg"
catalog: true
tags:
    - Spring Cloud
    - Netflix OSS
    - Spring Cloud Alibaba
    - MicroService
---

> Code is Not Only Code .

## Micro-Service-Framework Base ON Spring-Cloud

　　Spring cloud based distributed system architecture. Provide a complete set of microservice components, including service discovery, service governance, link tracking, service monitoring, task scheduling and more. All services are supported in docker.

### Git
```bash
   git clone https://github.com/CiNC0/Tiffany.git
```

### Project Structure

``` lua
Project Structure
-- 👇 Microservices public module
├── cloudx-eureka-server -- Service discovery
├── cloudx-config-server -- Distributed Configuration Center
├── cloudx-config-repo -- Profile
├── cloudx-admin-server -- Service monitoring
├── cloudx-gateway-server -- Gateway
├── cloudx-turbine-server -- Dashboard aggregation services
├── cloudx-zipkin-server -- Link monitoring
├── cloudx-jobs -- Distributed task scheduling
├── cloudx-jobs-console -- Distributed task scheduling console

-- 👇 Micro-Servce-Modules
├── cloudx-common -- Public modules, tools and so on
├── cloudx-base -- Basic information, entity beans and more
├── cloudx-auth-api --  Pass licensing api, providing feign interface
├── cloudx-auth-provider -- Pass authorization service
├── .....Other service modules.....


```

### Technical selection

Technical selection | Description | Official website
----|------|----
Spring cloud eureka | Cloud Services Discovery, a REST-based service for locating services for cloud middle-tier service discovery and failover。 | [https://projects.spring.io/spring-cloud/](https://projects.spring.io/spring-cloud/)
Spring cloud config server | Allows you to put the configuration on a remote server, centralized management of cluster configuration, currently supports local storage, Git and Subversion  | [https://projects.spring.io/spring-cloud/](https://projects.spring.io/spring-cloud/)
Spring cloud zuul | Zuul is a framework that provides edge services such as dynamic routing, monitoring, resiliency, and security on a cloud platform  | [https://projects.spring.io/spring-cloud/](https://projects.spring.io/spring-cloud/)
Spring Cloud Sleuth | Log Collection Toolkit encapsulates Dapper and log-based tracing as well as Zipkin and HTrace operations and implements a distributed tracking solution for SpringCloud applications。 | [https://projects.spring.io/spring-cloud/](https://projects.spring.io/spring-cloud/)
Spring boot admin | Service monitoring  | [http://projects.spring.io/spring-boot/](http://projects.spring.io/spring-boot/)
Feign | A declarative, templated HTTP client.
Redis | Distributed cache database  | [https://redis.io/](https://redis.io/)
Swagger2 | Interface test framework  | [http://swagger.io/](http://swagger.io/)
Maven | Project Construction Management  | [http://maven.apache.org/](http://maven.apache.org/)
Elastic-job | Distributed task scheduling | [http://elasticjob.io](http://elasticjob.io)
Zookeeper | Distributed framework | [https://zookeeper.apache.org](https://zookeeper.apache.org)

### Environment to build

> Development environment

- 1、Native installation Jdk8, Mysql, Zookeeper, Redis and **start the relevant services**, the default port to use the default configuration
- 2、Clone source code to local and open, **IntelliJ IDEA recommended**

> Ready to work

- Create a database

- Modify the corresponding configuration information cloudx-config-repo (mysql, zookeeper, redis)

- Configure spring-cloud-config configuration information (git) in cloudx-config-server

> Start the service

- There are three ways to start the service:

- 1、Execute the spring boot main method

- 2、Execute the maven package command mvn clean install -Dmaven.test.skip = true

- 3、mvn package docker:build
