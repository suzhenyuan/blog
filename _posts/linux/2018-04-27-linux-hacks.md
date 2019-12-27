---

comments: true
title: Linux Hacks
categories: Linux
tags: Linux,Shell
description: linux基本命令，如sed、awk、ps、netstat、 sort、 uniq
---


## sed

sed是Unix常见的命令行程序。sed用来把文档或字符串里面的文字经过一系列编辑命令转换为另一种格式输出。sed通常用来匹配一个或多个正则表达式的文本进行处理。 分号可以用作分隔命令的指示符。尽管sed脚本固有的很多限制，一连串的sed指令加起来可以编程像仓库番、快打砖块、甚至俄罗斯方块等电脑游戏的复杂程序
    
应用示例
* 获取csrf对应value值 &lt;input name="csrf" value="slkajfdklasjdflkajsdflka"/&gt;


        echo '<input name="csrf" value="slkajfdklasjdflkajsdflka"/>' | grep "csrf" | sed 's/\(.*\)value\="\([^""]*\)"\(.*\)/\2/g'

* 使用变量


        t=123
        echo "adasdf" | sed 's/a/'"$t"'/g'

## xargs 

xargs的作用是将参数列表转换成小块分段传递给其他命令，以避免参数列表过长的问题

* 查找包含特定字符的文件


        find . -type f -print | xargs grep "hello,world"


## nohup

nohup可以不挂断地运行命令.  
    
        nohup [command] & 

nohup默认会生成名称为nohup.out的日志文件，如果不需要记录日志文件，可以重定向到/dev/null。如：

        nohup [command] >/dev/null &
