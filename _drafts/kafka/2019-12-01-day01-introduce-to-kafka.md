---
title: 第一天-初步接触kafka分布式消息队列
tags: kafka
aside:
  toc: true
categories: kafka
---

读过kafka文档之后，对kafka有了一定的认识，打算从以下几方面进行总结：

- kafka系统架构介绍
- kafka涉及到的相关术语
- 分区、副本与日志介绍
- 消息生产过程介绍
- 消息消费过程介绍





## 一、Kafka 的架构解读

### 1.1. kafka系统架构

基于kafka-zookeeper的系统架构如下所示：

![system architecture of kafka](http://blog.19881101.com/images/kafka/20191205_system_architecture_of_kafka.png)

从上图可以看出，一个典型的kafka系统包括，若干服务实例broker，若干消息生产者producer，若干消息消费者consumer（Group），以及zookeeper集群。kafka集群通过zookeeper实现服务配置与服务发现，实现leader选举和consumer group 的rebalance，消息生产者通过push方式把消息提交到broker，消息消费者通过pull方式从broker订阅并消费消息。

### kafka内部实现

从上图的系统架构图中，大致勾勒出kafka系统的组成结构，并没有对broker进行深入刻画。要深入了解broker内部结构，需要涉及到主题(topic)、分区(partition)、副本(replicas)等相关概念，如下图所示：





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