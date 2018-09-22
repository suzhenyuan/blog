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

以下代码放到全局的pom文件中，后面的项目还会用到的。版本号为`2.0.1.RELEASE`

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

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	</dependency>

注：1.0.x的artifactId是`spring-cloud-starter-eureka`,而2.0.x的artifactId则改为了`spring-cloud-starter-netflix-eureka-server`，后文用到的client同样改为了`spring-cloud-starter-netflix-eureka-client`

## application.properties
新建src/main/resources目录，新建application.properties, 添加如下配置：


    spring.application.name=demo_eureka_server
    #运行端口为8761
    server.port=8761    

    #eureka.server.enableSelfPreservation=false
    eureka.instance.appname=eureka
    #自己不需要向自己注册 
    eureka.client.register-with-eureka=false
    #不需要获取注册的服务
    eureka.client.fetch-registry=false
    #eureka service url
    eureka.client.service-url.defaultZone=http://localhost:${server.port}/eureka

## 启动类EurekaApplication.java
新建EurekaApplication.java，添加如下注解：

    package org.suzy.demo.eureka;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;


    /**
    * @author SuZhenYuan
    * Eureka Server startup
    */
    @SpringBootApplication
    @EnableEurekaServer
    public class EurekaApplication {
        public static void main(String[] args) {
            SpringApplication.run(EurekaApplication.class, args);
        }
    }


## eureka server启动结果
浏览器访问http://localhost:8761/，即可看到eureka启动的界面，如下图所示：
![Eureka Server startup][img_of_eureka_server_startup]


<a href="https://github.com/suzhenyuan/MyIntroduceToSpringCloud/tree/eureka_server"  onclick="ga('send', 'event', 'Navbar', 'Github links', 'eureka_server');" class="btn btn-primary btn-lg" role="button">相关代码请见github</a>



## 参考资料

1. [spring cloud netflix quick start](https://cloud.spring.io/spring-cloud-netflix/#quick-start)


<!-- 引用的图片列表 -->
[img_of_eureka_server_startup]: /images/springcloud/eureka/eureka_server_startup.png "Eureka Server startup"
