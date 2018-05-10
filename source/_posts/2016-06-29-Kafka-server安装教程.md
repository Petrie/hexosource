---
title: Kafka server安装教程
date: 2016-06-29 00:01:13
description:
category:
- 消息队列
tags:
- kafka
- Linux
- 消息队列
---

#### 下载解压
>下载地址：
>https://www.apache.org/dyn/closer.cgi?path=/kafka/0.10.0.0/kafka_2.11-0.10.0.0.tgz
``` bash
> tar -xzf kafka_2.11-0.10.0.0.tgz
> cd kafka_2.11-0.10.0.0
```

#### 启动Kafka服务
Kafka服务需要安装ZooKeeper。如果没有你可以使用kafka附带的临时脚本来启动ZooKeeper
``` bash
#启动ZooKeeper
> bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
```

``` bash
#启动kafka
> bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
```

#### 创建测试Topic
Let's create a topic named "test" with a single partition and only one replica:
``` bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
We can now see that topic if we run the list topic command:
``` bash
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
```

#### Producer发送测试消息
使用Kafka提供的命令行工具发送消息
``` bash
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

#### Consumer消费消息
使用Kafka提供的命令行工具接收消息
``` bash
> bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
This is a message
This is another message
```

#### 设置一个多broker的集群
上面的使用场景都是单broker。下面体验下kafka的多broker模式。
首先创建为每一个broker创建一个broker：
``` bash
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
```
编辑配置文件
>file:config/server-1.properties:
``` file 
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dir=/tmp/kafka-logs-1
```

>file:config/server-2.properties:
``` file
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dir=/tmp/kafka-logs-2
```
配置中的broker.id是唯一的用来索引集群中的节点。端口和日志路径必须不同，否则实列间会相互覆盖日志。
之前已经启动了两个节点了，现在启动新的两个。
``` bash
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```

Now create a new topic with a replication factor of three:
``` bash
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

现在我们有了一个集群，如果监控每个broker呢？使用下面的命令：
``` bash
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
Topic: my-replicated-topic	Partition: 0	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
```


