---
layout: default
comments: true
title: 如何优雅地停止spring boot应用
sub_title: 
meta-keyword: spring boot,actuator, shutdown,spring-security, csrf
meta-description: How to shutdown a spring boot application with the help of /actuator/shutdown。[spring boot 2.0.x]
categories: spring-boot
tags: spring-security
description: 在不同的security配置下，如何优雅地通过actuator的shutdown端点来停止spring boot应用
date: 2018-10-12
---

Spring Boot Actuator提供了一套完备的监控方案用来监控Spring Boot应用。本文将会根据在不同的场景下，采用不同的方式来关闭一个spring boot应用的。这些场景包括：

* 普通场景，没有启用任何安全设置
* 启用了security配置，同时关闭了csrf
* 启用了security配置，没有关闭csrf

## pom配置

引入actuator和security相关的包，spring-boot的版本号为`2.0.4.RELEASE`，主要pom配置如下：

    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.0.4.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>


## application.properties
在application.properties中，需要启用shutdown相关的endpoint：

    
    management.endpoint.shutdown.enabled=true
    #注，此处只是配置了shutdown，如果需要启用其他endpoints，在shutdown后面添加即可，也可以直接用*号，启用所有的endpoints
    management.endpoints.web.exposure.include= shutdown

## 关闭脚本

假设应用运行地址是`http://localhost:8080`

* 在没有做其他配置的情况下，直接执行以下命令即可。


    curl -X POST http://localhost:8080/actuator/shutdown

* 复杂一点的情况，如果需要启用安全设置,同时关闭了csrf，

    application.properties中增加以下安全设置：

        spring.security.user.name=user
        spring.security.user.password=password

    关闭csrf的configuration如下：

        import org.springframework.context.annotation.Configuration;
        import org.springframework.security.config.annotation.web.builders.HttpSecurity;
        import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

        @Configuration
        public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {
            @Override
            protected void configure(HttpSecurity http) throws Exception {
                http.csrf().disable();
                super.configure(http);
            }
        }

    则，对应的命令为：


        curl -X POST http://user:password@localhost:8080/actuator/shutdown

* 更复杂的情况，启用了安全设置，而没有关闭csrf，操作则会稍微复杂一点。思路如下：
    - 先从登录页获取当前csrf值
    - 登录
    - 再从登录页面获取一次csrf
    - 调用关闭url
    - 注意事项：使用curl时，必须带上前一个请求的cookie，并保存响应后的cookie

    具体脚本如下：

    

        #/bin/bash
        #description: shutdown a spring boot application with the protection of spring security without csrf disable
        #1、get csrf token from login page
        #2、login
        #3、get csrf token from login page
        #4、post to /actuator/shutdown

        url_of_login="http://localhost:8080/login"
        url_of_shutdown="http://localhost:8080/actuator/shutdown"
        fcookie=cookie.txt
        config_username=config_user
        config_password=config_password

        #get csrf from login page, save cookie to cookie.txt
        csrf=`curl ${url_of_login} -c ${fcookie} | grep "csrf" | sed 's/\(.*\)value\="\([^""]*\)"\(.*\)/\2/g'`
        echo 'csrf: ' ${csrf}
        #login
        curl -X POST ${url_of_login} -b ${fcookie} -c ${fcookie} -d "username=${config_username}&password=${config_password}&submit=Login&_csrf=${csrf}"
        #get csrf from login page
        csrf=`curl ${url_of_login} -b ${fcookie} -c ${fcookie} | grep "csrf" | sed 's/\(.*\)value\="\([^""]*\)"\(.*\)/\2/g'`
        #post to /actuator/shutdown
        result=`curl -X POST -b ${fcookie} -H "X-CSRF-TOKEN: ${csrf}" ${url_of_shutdown}`

        if [ "$result" = '{"message":"Shutting down, bye..."}' ]; then
            echo -e "\033[49;31;1;5m application shutdown...\033[0m"
        fi

