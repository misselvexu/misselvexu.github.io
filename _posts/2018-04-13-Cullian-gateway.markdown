---
layout:     post
ignore:     true
title:      "[Develop] Cullian Gateway for Next Microservice"
subtitle:   "Micro-Service Gateway"
date:       2018-04-13 09:59:59
author:     "Elve Xu"
header-img: "img/cullian/cullian-bg-1.png"
catalog: true
tags:
    - Microservices
    - Dynamic
    - Gateway
    
---

> Code is Not Only Code .

<h3 align="left">
  <img src="{{ "/img/cullian/cullian-icon.png" | prepend: site.baseurl }}" alt="cullian Logo" />
</h3>

# Cullian Project

Cullian is an edge service that provides dynamic routing, monitoring, resiliency, security, and more.

## Why Need Cullian Gateway

### Traditional gateway deployment architecture

<h3 align="center">
  <img src="{{ "/img/cullian/gateway-old.png" | prepend: site.baseurl }}" alt="gateway-old" />
</h3>

#### Is Any defect ?

- Un-support RPC Framework ,Cant route directly! Like: Dubbo ,Motan ,gRPC ...
- TCP/UDP cant proxy
- ...

### Cullian ca solve upper defects

Deployment architecture like this:

<h3 align="center">
  <img src="{{ "/img/cullian/gateway-cullian.png" | prepend: site.baseurl }}" alt="gateway-cullian" />
</h3>

### Cullian Core Framework

<h3 align="center">
  <img src="{{ "/img/cullian/cullian-core-framework.png" | prepend: site.baseurl }}" alt="cullian-core-framework" />
</h3>

### Cullian Deploy Structure (Recommend) 

<h3 align="center">
  <img src="{{ "/img/cullian/cullian-deploy-env.png" | prepend: site.baseurl }}" alt="cullian-deploy-env" />
</h3>


# Roadmap
## Gateway

### Protocol
 
- [x] HTTP/1.x
- [ ] HTTP/2.0
- [ ] TCP/IP
- [ ] UDP
- [ ] Dubbo
- [ ] Motan
- [ ] gRPC

### Configuration
- [x] Java API
- [x] YAML

### Security

- [x] SSL

### RPC Framework Support

- [x] Spring Cloud (HTTP)
- [ ] Motan
- [ ] Dubbo
- [ ] gRPC

### Transport Protocol

- [x] JSON
- [ ] Google Protobuf
- [ ] Thrift

## Github

> `https://github.com/CiNC0/Cullian`

> If you like this project, Welcome to `star`

## JOIN ME

> If U want to developing with us

> Email to Me : `iskp.me@gmail.com`
