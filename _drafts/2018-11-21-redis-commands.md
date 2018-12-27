---
layout: post_layout
comments: true
title: redis命令大全
sub_title: 
meta-keyword:redis, command
meta-description: 整理了redis相关命令及用法，以及常见应用场景分析
categories: spring-boot
tags: spring-boot, docker
description:  整理了redis相关命令及用法，以及常见应用场景分析
date: 2018-11-21
---

## Connection相关命令
  * AUTH password 连接到密码保护的redis服务器时，请求授权
  
  * ECHO message  输出信息
  
  * PING [message] 用于测试连接是否有效，如果message为空，则会返回PONG
  
  * QUIT 请求服务器关闭连接，总是返回OK
  
  * SELECT 切换数据库，默认0
  
  * SWAPDB index index 交换两个数据库的数据，切换成功后返回OK

## GEO 地理信息相关命令
  * GEOADD key lon lat member [log lat member] 增加一个地理位置信息。底层使用的是sorted set,删除用ZREM。
  
  * GEODIST key member1 member2 [unit] 获取两个member的距离，unit为单位：m(米),km(公里),mi(英里),ft(英尺)
  
  * GEOHASH key member [member ...] 返回GEOHASH的短字符串值
  
  * GEOPOS key member [member ...] 返回member的经纬度(log,lat)
  
  * GEORADIUS key lon lat radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key] 查找以lon lat为中心，指定radius半径的位置信息，返回的数据结构为：["member","distance","hash",["lon","lat"]]

  * GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key] 该命令与GEORADIUS不同的地方在于，该查找是以sorted set中的某个member为中心进行查找的，其他的相同

## Hashes
  * HDEL key field [field] 从hash中删除一个或多个field。在2.4版本之前，需要用MULT/EXEC块来实现原子操作
  
  * HEXISTS key field 判断hash中是否包含key，包含返回1，否则返回0

  * HGET key field 返回hash中field对应的值
  
  * HGETALL key 返回hash中所有的字段和值，返回的长度是hash长度的两倍，奇数字段返回field，偶数字段发挥value
  
  * HINCRBY key field increment key对应的hash存储的字段增加increment。如果key不存在，则初始化为0后，再实施加increment操作。
  * HINCRBYFLOAT key field increment 同HINCRBY，只是increment为浮点数
  
  * HKEYS key 返回hash中所有的字段名 
  
  * HLEN key 返回hash中所有字段名的数量

  * HMGET key field [field] 返回指定field对应的值，不存在的field则返回nil，返回顺序同field的顺序
   
  * HMSET key field value [field value] 同时设置多个field和value
  
  * HSCAN key cursor [MATCH pattern] [COUNT count] 见SCAN
  
  * HSET key field value 设置一个值，如果不存在则创建，返回值为1，否则更新，返回值为0
   
  * HSETNX  key field value 设置一个值，如果不存在则创建，返回1，否则不做任何操作，返回0

  * HSTRLEN key field 返回field对应字符串的长度，如果key或者field不存在，则返回0.

  * HVALS key 返回key对应hash结构的所有值

## HyperLogLog 基数统计
   HyperLogLog在Redis中的每个键占用的内容都是12k。可以用少量固定的内存去存储并识别集合中的唯一元素，他只能统计数量，却不记录具体数值。可以用于统计每日访问ip量，注册数量等。。。

  * PFADD key element [element] 添加元素到HyperLogLog结构中
  * PFCOUNT key 返回基数统计的近似值，容忍度大概是0.81%
  * PFMERGE destkey sourcekey [sourcekey] 合并多个sourcekey到destkey

## keys
  * DEL key [key ...] 删除key
  * DUMP key 使用redis特定的格式来序列化存储在key中的值，恢复用RESTORE
  * EXISTS key [key ...] 判断key是否存在，存在返回1，否则返回0。传入多个key，则返回存在的key的数量。
  * EXPIRE key seconds 为key指定过期时间，超过指定时间后，该key就会被删除。过期时间在key被删除或者内容重写后被清除，包括DEL, SET GETSET 还有所有*STORE命令.如果key被RENAME了,则新key还是保持该过期时间.如果在RENAME时,新的key已经存在,新的key不会和原来的key有管理.新key会继承修改前的key的所有内容.
  * EXPIPEAT key timestamp 同EXPIRE，只是传入的是时间戳
  * KEYS pettern 返回所有匹配的key
  * MIGRATE host port key|"" destination-db timeout [COPY] [REPLACE] [KEYS key [key ...]] 从原redis实例中，原子性地转移一个key到目标redis实例。会堵塞
  * MOVE key db 从当前数据库中移动key到另外一个数据库
  * OBJECT subcommand [arguments [arguments ...]] 用于探测内部Redis Objects，用于debug或者弄明白存储结构
  * PERSIST key 清除key的超时时间，把key由过期集合转移至永久集合。
  * PEXIRE key milliseconds 同EXPIRE,只是时间变味了毫秒
  * PEXPIREAT key milliseconds-timestamp 桶PEXPIREAT
  * PTTL 同TTL,返回的是毫米,如果key不存在，则返回-2，没有设置过期时间，则返回-1。
  * RANDOMKEY 返回一个随机key
  * RENAME key newkey 重命名一个key，newkey已存在，则重写.在数据量比较大的情况下，与DEL会有冲突，因此删除一个包含大数据的key时，会有较大的延迟。
  * RENAMEENX key newkey 重命名一个key,如果key不存在，则返回错误。
  * RESTORE key ttl serialized-value [REPLACE] 恢复一个序列化的内容
  * SACN cursor [MATCH pattern] [COUNT count] 迭代的是所有数据库的键。
  * SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE destination] 返回或者存储在对应key的list,set或者sorted set
  * TOUCH key [key ...] 刷新key的最近访问时间
  * TTL key 查看key剩余过期时间，单位(s)
  * TYPE key 查看key对应value的存储类型
  * UNLINK key [key ...] 删除对应的key，与DEL类似，不同的地方在于，他会起另外的线程去删除，而DEL是阻塞的。
  * WAIT numreplicas timeout 阻塞当前客户端，直到之前的写操作成功传输并至少由指定数量的从站确认。
  
  * SETEX key seconds value 原子操作