---
layout: default
title: spring cloud开始之旅，说在前面的话
sub_title: 无版本号不成参考。本系列涉及到的版本号为Finchley.SR1(基本都是2.0.x.RELEASE)
meta-description: spring cloud before started, version of spring boot & cloud must be declare first
categories: spring-cloud
tags: spring-cloud,eureka
description: 在开始spring cloud开发之旅之前，先了解一下spring boot & cloud的版本情况。spring cloud系列的博文会使用Finchley.SR1的版本
---

在csdn和博客园上阅览了无数千遍一律的spring cloud博文，我觉得，我需要指明目前使用到的版本号。无版本号不成参考。

## spring boot & cloud 版本


Component	|	Edgware.SR4	|	Finchley.SR1	|	Finchley.BUILD-SNAPSHOT
---------------|------------|------------------|----------------------
spring-cloud-aws	|	1.2.3.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-cloud-bus	|	1.3.3.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-cloud-cli	|	1.4.1.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-cloud-commons	|	1.3.4.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-contract	|	1.2.5.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-config	|	1.4.4.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-netflix	|	1.4.5.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-security	|	1.2.3.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-cloud-cloudfoundry	|	1.1.2.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-cloud-consul	|	1.3.4.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-sleuth	|	1.3.4.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-stream	|	Ditmars.SR4	|	Elmhurst.SR1	|	Elmhurst.BUILD-SNAPSHOT
spring-cloud-zookeeper	|	1.2.2.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-boot	|	1.5.14.RELEASE	|	2.0.4.RELEASE	|	2.0.4.BUILD-SNAPSHOT
spring-cloud-task	|	1.2.3.RELEASE	|	2.0.0.RELEASE	|	2.0.1.BUILD-SNAPSHOT
spring-cloud-vault	|	1.1.1.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-gateway	|	1.0.2.RELEASE	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-openfeign	|	-	|	2.0.1.RELEASE	|	2.0.2.BUILD-SNAPSHOT
spring-cloud-function	|	1.0.0.RELEASE	|	1.0.0.RELEASE	|	1.0.1.BUILD-SNAPSHOT
{:.table}


由于本人刚接触spring-cloud不久，截至2018-09-22日，也不过一个多月的时间。开始就使用`Finchley.SR1`的版本。在后面的博文中也会着重标记相关组件使用的版本号。

## 工欲善其事，必先利其器

强烈建议eclipse安装spring tool插件。非eclipse用户可以无视。


参考资料：

1. [spring cloud quick start](https://projects.spring.io/spring-cloud/#quick-start)
