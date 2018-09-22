---
layout: default
title: 注册发现服务Eureka server介绍
sub_title: 
meta-description: introduce to eureka server
categories: spring-cloud
tags: spring-cloud,eureka
description: 注册发现服务Eureka server介绍,及高可用配置，以及源代码分析
---

Eureka是netflix的注册发现服务组件，eureka客户端可以通过发现服务获取注册到eureka服务器的实例。

## pom配置

以下代码放到全局的pom文件中，后面的项目还会用到的。

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-netflix</artifactId>
                <version>2.0.1.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    
以下代码放到eureka项目的pom文件中






参考列表：

1. [spring cloud netflix quick start](https://cloud.spring.io/spring-cloud-netflix/#quick-start)