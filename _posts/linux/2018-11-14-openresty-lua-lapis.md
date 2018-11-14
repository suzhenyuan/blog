---
layout: post_layout
comments: true
title: openResty lua lapis环境安装
sub_title: 
meta-keyword: openResty,lua,lapis
meta-description: openResty、lua、lapis入门介绍
categories: linux
tags: openrest,lua
description:  openResty、lua、lapis入门介绍
date: 2018-11-14
---


## 基于windows系统

## openResty安装

* 下载

    下载win64位版本，下载地址为`https://openresty.org/download/openresty-1.13.6.2-win64.zip`
* 安装

    解压后，设置环境变量。设解压目录为`D:\ProgramFiles\openresty-1.13.6.2-win64`，把该目录添加到`PATH`环境变量中
* 测试
   
    打开命令行终端，输入以下代码`resty -e 'print("hello,world")'`,看到输出`hello,world`表明openResty已经安装成功了。

## 安装luarocks
* 下载

	下载地址`http://luarocks.github.io/luarocks/releases/luarocks-3.0.4-win32.zip`
* 安装
	
	解压`luarocks-3.0.4-win32.zip`，打开命令行终端，并进入该目录。输入以下命令

    <code>
        set PREFIX=D:\ProgramFiles\openresty-1.13.6.2-win64
        install /P %PREFIX%\luarocks /SELFCONTAINED /INC %PREFIX%\include\luajit-2.1 /LIB %PREFIX% /BIN %PREFIX% /MW

    </code>

* 环境变量配置

    <code>
        You may want to add the following elements to your paths;
        Lua interpreter;
          PATH     :   D:\ProgramFiles\openresty-1.13.6.2-win64
          PATHEXT  :   .LUA
        LuaRocks;
          PATH     :   D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks
          LUA_PATH :   D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks\lua\?.lua;D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks\lua\?\init.lua
        Local user rocktree (Note: %APPDATA% is user dependent);
          PATH     :   %APPDATA%\LuaRocks\bin
          LUA_PATH :   %APPDATA%\LuaRocks\share\lua\5.1\?.lua;%APPDATA%\LuaRocks\share\lua\5.1\?\init.lua
          LUA_CPATH:   %APPDATA%\LuaRocks\lib\lua\5.1\?.dll
        System rocktree
          PATH     :   D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks\systree\bin
          LUA_PATH :   D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks\systree\share\lua\5.1\?.lua;D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks\systree\share\lua
        \5.1\?\init.lua
          LUA_CPATH:   D:\ProgramFiles\openresty-1.13.6.2-win64\luarocks\systree\lib\lua\5.1\?.dll
    </code>

## mingw-w64 的安装
    
   下载`mingw-w64`安装包，安装时，选择x86_64架构即可。

## 安装lapi

   由于网络原因，通过luarocks命令安装时，rock文件无法安装，只好把相关包下载到本地来安装了。

    
    https://luarocks.org/date-2.1.2-1.src.rock
    https://luarocks.org/lapis-1.7.0-1.src.rock
    https://luarocks.org/etlua-1.3.0-1.src.rock


   执行安装命令
    
    luarocks.bat install date-2.1.2-1.src.rock
    
  其他包的安装同上.

   由于在安装luaossl时，CRYPTO库引用,以及lib版本问题无法解决. 基于windows系统的安装,从入门到放弃.

## 基于centos 7

## 安装openresty

    wget https://openresty.org/download/openresty-1.13.6.2.tar.gz
    tar xzvf openresty-1.13.6.2.tar.gz
    cd openresty-1.13.6.2
    
    ./configure -j2
    make -j2
    sudo make install

## 安装luarocks

    wget http://luarocks.github.io/luarocks/releases/luarocks-3.0.4.tar.gz
    tar xzvf luarocks-3.0.4.tar.gz
    cd luarocks-3.0.4
    ./configure --prefix=/usr/local/ --with-lua=/usr/local/openresty/luajit
    make && make install

## 安装lapis
    
    luarocks install lapis

## 测试

    lapis new --lua
    lapis server 

访问`http://localhost:8080/`将会看到

    Welcome to Lapis 1.7.0
   
至此,lapis的环境安装完成了