---
layout: post_layout
comments: true
title: openResty lua lapis环境安装
sub_title: 
meta-keyword: openResty,lua,lapis
meta-description:  openResty、lua、lapis入门介绍
categories: spring-boot
tags: spring-security
description:  openResty、lua、lapis入门介绍
date: 2018-11-14
---

## openResty安装

* 下载

    下载win64位版本，下载地址为`https://openresty.org/download/openresty-1.13.6.2-win64.zip`
* 安装

    解压后，设置环境变量。设解压目录为`D:\ProgramFiles\openresty-1.13.6.2-win64`，把该目录添加到`PATH`环境变量中
* 测试
   
    打开命令行终端，输入以下代码`resty -e 'print("hello,world")'`,看到输出`hello,world`表明openResty已经安装成功了。

## 安装luarocks
* 下载

	下载地址`http://luarocks.github.io/luarocks/releases/luarocks-3.0.4-windows-32.zip`
* 安装
	
	把下载的文件解压到openResty目录，里面只有两个exe文件，分别为`luarocks.exe`与`luarocks-admin.exe`