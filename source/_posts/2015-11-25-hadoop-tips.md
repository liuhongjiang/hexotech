title: hadoop常用命令
date: 2015-11-25 15:41:31
tags:
categories: hadoop
---

## 两个集群直接拷贝数据distcp

```
hadoop distcp hdfs://namenode:port/source hdfs://namenode:port/destination
例子：
hadoop distcp hdfs://dev-hdfs-001.server:8020/data/logs/ hdfs://mylocalhdfs:8020/data/logs/
```

拷贝的时候可能有权限问题，在目标目录所在的服务骑上，修改目标目录的权限为777

另外一个命令是：

```
hdfs dfs -cp hdfs://dev-hdfs-001.server:8020/data/logs/ /data/logs/
```

```
hdfs dfs -chmod 777 /path/to/directory
```

或者使用一个与目标集群里面的管理员相同的名字的账号，来运行这个命令。或者将运行命令的这个账号，在目前集群上，加入到集群的用户组中。

<!-- more -->

## 查看目录大小

```
hdfs dfs -du -s -h /path/to/directory
```

## 配置spark history server

* step1

add following configrations in the `spark-default.conf` of spark conf directory, and copy to all spark node:

```
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://10.114.207.140:8020/logs/spark-event-log/

spark.history.fs.logDirectory	hdfs://10.114.207.140:8020/logs/spark-event-log/
```

* step2

on one node which chose to be the history sever, run this command in the `sbin` directory of spark:

```
./sbin/start-history-server.sh
```

then you can visit the app history by: `http://<server-url>:18080`

reference: http://spark.apache.org/docs/latest/monitoring.html