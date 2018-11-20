---
layout: default
comments: true
title: Mysql Hacks
categories: Linux
tags: Linux,Nginx
---



- 如何统计某个字段中单词的数量
   
    
    SELECT SUM( LENGTH( bodytext ) -  LENGTH( REPLACE( bodytext, ' ', '' ) ) +1 )
    FROM tt_content

[参考来源：MySQL: Count Words in Column][how_to_count_words_in_column]

[how_to_count_words_in_column]:https://ben.lobaugh.net/blog/476/mysql-count-words-in-column


