---
title: 消息队列之Kafka与RocketMQ的对比
date: 2018-08-09 16:28:03
tags: 消息队列 
---
## 消息队列之Kafka与RocketMQ的对比 ##
- 部署和模型
	- 部署
		- RocketMQ 主从架构 一主多从 
		- Kafka 互为主从 一个Broker既可以是某个分区的主副本也可以是从副本
	- 存储模型
		- Kafka:Topic_A的每个partition都有commitLog来存储消息，每个CommitLog有多个Segment,当Segment写满，或则Segment对应的index写满，或者Segment超时，就会重新创建一个Segment来存储该Partition的消息。每个Segment都有一个index文件，是一个稀疏索引，当Segment写入的消息超过一定数量后，往index写入一条记录。
		- RocketMQ:Topic_A有3各ConsummeQueue，两主一备，Broker_A,B,Master对外提供服务。Index可以根据key和temeStamp来查询。
- 有序消息
	- kafka:都是一个topic多个partition场景，保证每个Partition里面的消息都是有序的。一个topic可以配置多个partition,producer在发送消息的时候，按照一定的策略选择partition来写消息，保证同一个订单的消息，写到同一个partition上。
	- RocketMQ:一个Topic可以有多个consumerQueue,每个queue里面消息是有序的。
- 发送端负载均衡
	- hash,随机缓存
- 消费端负载均衡
	- kafka当有consumer加入到comsumerGroup,会发送JoinGroup消息给Coordinator,Coordinator收集到所有的消息，会选择一个主Consumer，把执行的Comsumer消息以及Partition消息发送给他。然后同步告知其他Consumer
- 数据可靠性
	- kafka:支持同步/异步复制，异步刷盘
- 消息投递的实时性
	- kafka使用段轮询方式，实时性取决于轮询周期
	- RocketMQ使用长轮询，通过push方式实现
- 消费失败重试
	- Kafka不支持
	- RocketMQ支持定时重试