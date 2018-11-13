---
layout: post_layout
comments: true
title: Spring Secutiry安全配置WebSecurityConfigurerAdapter
sub_title: 
meta-keyword: spring boot,actuator, shutdown,spring-security, csrf, WebSecurityConfigurerAdapter
meta-description: Introduce to spring-security-config. 。[spring boot 2.0.x]
categories: spring-boot
tags: spring-security
description:  Spring Security 之 WebSecurityConfigurerAdapter介绍
date: 2018-10-19
---

Spring Security是一个功能强大且可高度定制的身份验证和访问控制框架。 它是保护基于Spring的应用程序的事实上的标准。

Spring Security是一个专注于为Java应用程序提供身份验证和授权的框架。 与所有Spring项目一样，Spring Security的真正强大之处在于它可以轻松扩展以满足自定义要求。

spring-security-config的版本号为`5.0.7.RELEASE`

本文将会介绍一下WebSecurityConfigurerAdapter中两个常用的configure()函数，他们分别是：

    /**
	 * Override this method to configure {@link WebSecurity}. For example, if you wish to
	 * ignore certain requests.
	 */
	public void configure(WebSecurity web) throws Exception {
	}


    /**
	 * Override this method to configure the {@link HttpSecurity}. Typically subclasses
	 * should not invoke this method by calling super as it may override their
	 * configuration. The default configuration is:
	 *
	 * <pre>
	 * http.authorizeRequests().anyRequest().authenticated().and().formLogin().and().httpBasic();
	 * </pre>
	 *
	 * @param http the {@link HttpSecurity} to modify
	 * @throws Exception if an error occurs
	 */
    protected void configure(HttpSecurity http) throws Exception {
		logger.debug("Using default configure(HttpSecurity). If subclassed this will potentially override subclass configure(HttpSecurity).");

		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.formLogin().and()
			.httpBasic();
	}

## configure(WebSecurity web)

在子类中重新该函数，可以在过滤器中排除特定的url的处理，比如:
    
    @Override
	public void configure(WebSecurity web) throws Exception {
		web.ignoring().antMatchers("/actuator/info");
	}
该功能使用起来比较简单（或者，是我理解的比较肤浅）。

## protected void configure(HttpSecurity http) 
该函数可以控制资源的访问权限，csrf配置等。比如：

- 关闭csrf


    @Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable();
	}
  很多博客提到关闭csrf都是使用了以上的代码，但是，上述代码存在一个问题，授权访问相关的配置没了。查看源代码，实际上，`config(HttpSecurity http)` 是有默认实现的`http.authorizeRequests().anyRequest().authenticated().and().formLogin().and().httpBasic();`。 在不作太多改变的情况下，应该用下面的代码：

    
    @Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable();
		super.configure(http);
	}

- 定制url访问控制

比如，对/actuator/shutdown保持授权要求，其他/actuator/* 不作要求，则配置如下：

    @Override
	protected void configure(HttpSecurity http) throws Exception {
		http.csrf().disable();
		http.authorizeRequests().antMatchers("/actuator/shutdown").authenticated();
		http.authorizeRequests().antMatchers("/actuator/**").permitAll();
		super.configure(http);
	}

- 顺便提一下spring boot admin

  在spring boot admin客户端的配置如下：


    spring.boot.admin.client.url=http://localhost:8750
    spring.boot.admin.client.username=bootadmin_user
    spring.boot.admin.client.password=bootadmin_password
    
    spring.boot.admin.client.instance.metadata.user.name=${spring.security.user.name}
    spring.boot.admin.client.instance.metadata.user.password=${spring.security.user.password}

  在客户端的配置中，是可以通过`spring.boot.admin.client.instance.metadata`把username和password传递给spring boot admin,而客户端是不需要关闭security的，从而确保客户端不受安全威胁。