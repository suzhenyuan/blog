---

comments: true
title: Mysql Hacks
categories: Linux
tags: mysql
---



### 统计某个字段中单词的数量

```sql
SELECT SUM( LENGTH( bodytext ) -  LENGTH( REPLACE( bodytext, ' ', '' ) ) +1 )
FROM tt_content
```



