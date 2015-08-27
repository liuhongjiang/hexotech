title: Install kafka
date: 2015-08-26 14:22:24
categories: hadoop
tags:
---

这一片文章是关于安装kafka的，但是是基于前面几篇文章的基础之上的，请参考前面的安装hadoop，安装zookeeper。

## 准备

kafka的运行需要zookeeper，kafka自身是带有zookeeper的，但是是一个单节点，可以参考kafka的文档，了解如何使用。

这里使用的是上一篇文章中安装好的zookeeper。

下载[kafka](http://kafka.apache.org/downloads.html)，这里下载的是[kafka_2.11-0.8.2.1.tgz](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz)

解压文件包，将目录拷贝到/user/目录下，并修改权限：
```
chown -R hadoop:hadoop kafka_2.11-0.8.2.1
```

<!-- more -->

## 配置kafka

修改`${KAFKA_HOME}/config/server.properties`:
```
broker.id=1
log.dirs=/usr/kafka_2.11-0.8.2.1/logs
zookeeper.connect=master.hadoop:2181,slave1.hadoop:2181,slave2.hadoop:2181
```

对于具体的配置内容，请参考[kafka broker configs](http://kafka.apache.org/documentation.html#brokerconfigs)。

针对上面的配置，需要创建日志目录：`/usr/kafka_2.11-0.8.2.1/logs`

## 拷贝到其他节点

将配置好的kafka拷贝到其他节点的相应的目录下。

需要注意的是拷贝到其他节点以后，需要修改`${KAFKA_HOME}/config/server.properties`中的`broker.id`, 需要保证`broker.id`是一个唯一的整数。

## 启动和验证

启动和停止kafka：
```
bin/kafka-server-start.sh config/server.properties &
bin/kafka-server-stop.sh
```

* 连通性测试


在一台服务器上运行consumer：
```
bin/kafka-console-consumer.sh --zookeeper master.hadoop:2181 --topic test --from-beginning
```

在另外一台服务器上运行producer：
```
bin/kafka-console-producer.sh --broker-list slave1.hadoop:9092 --topic test 
```

这时在producer中，输入任何消息，在consumer端就可以看到。下面的图片：

producer的情况：

{% img center /images/blogimages/2015/kafka_producer.png %}

consumer的情况：

{% img center /images/blogimages/2015/kafka_consumer.png %}

## kafka Utility

1. Create topic
```
bin/kafka-topics.sh --create --zookeeper ${zk_host}:${zk_port}[${zk_chroot}] --replication-factor ${replication_factor} --partitions ${partition_count} --topic ${topic_name}
```

Note: for zookeeper connect string ${zk_host}:${zk_port}[${zk_chroot}], ${zk_chroot} is optional

2. Describe topic

* Describe topic all topics
```
bin/kafka-topics.sh --describe --zookeeper ${zk_host}:${zk_port}[${zk_chroot}]
```

* Describe specified topic
```
bin/kafka-topics.sh --describe --zookeeper ${zk_host}:${zk_port}[${zk_chroot}] --topic ${topic_name}
```

3. Publish messages. You can use this utility to test you topic.
```
bin/kafka-console-producer.sh --broker-list ${kafka_server}:{kafka_port} --topic ${topic_name}
```
Note: if you local kafka server is started, you can use localhost:9092 as broker-list.

4. Consume messages
```
bin/kafka-console-consumer.sh --zookeeper ${zk_host}:${zk_port}[${zk_chroot}] [--from-beginning] --topic ${topic_name}
```

5. Modifying topics
Currently, support increase partitions well:
```
bin/kafka-topics.sh --zookeeper ${zk_host}:${zk_port}[${zk_chroot}] --alter --topic ${topic_name} --partitions ${partition_count}
```
Kafka does not currently support reducing the number of partitions for a topic or changing the replication factor.

6. Delete topic
```
bin/kafka-topics.sh --zookeeper ${zk_host}:${zk_port}[${zk_chroot}] --delete --topic ${topic_name}
```
Topic deletion option is disabled by default. To enable it set the server config:
```
delete.topic.enable=true
```

## 参考

<http://kafka.apache.org/documentation.html#quickstart>
