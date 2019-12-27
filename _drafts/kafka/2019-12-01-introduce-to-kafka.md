---
title: 深入了解kafka分布式消息队列
tags: kafka
aside:
  toc: true
categories: spring-cloud
---



## 一、Kafka 的架构解读

### 1.1. kafka的系统架构图

基于kafka-zookeeper的系统架构如下所示：

![system architecture of kafka](http://blog.19881101.com/images/kafka/20191205_system_architecture_of_kafka.png)





从上图可以看出，一个典型的kafka系统包括什么内容？

《然后又深入描述了broker内部的清醒》

1.2. kafka内部架构图：topic、partition、comsumer group

	topic、particle是如何负载均衡的

1.3、术语到的术语

- comsumer
- producer
- broker
- kafka cluster
- topic
- partition
- offset
- comsumer group
- stream







## Kafka 为什么要将 Topic 进行分区；

## Kafka 高可靠性实现基础解读；
## Kafka 复制原理和同步方式；
## Leader 选举机制，及如何确保新选举出的 Leader 是优选；
## 同步副本 ISR；
## Kafka 数据可靠性和持久性保证；
## 深入解读 HW 机制；
## Kafka 架构中 ZooKeeper 以怎样的形式存在；
## 全程解析：Producer -> kafka -> consumer。