title: Install zookeeper
date: 2015-08-26 11:54:16
categories: hadoop
tags:
---

由于要安装kafka，需要安装zookeeper。

这一片文章是关于安装zookeeper的，但是是基于前面几篇文章的基础之上的，请参考前面的安装hadoop。

## 准备

在其中一个节点中做以下操作。

下载[zookeeper](http://zookeeper.apache.org/releases.html), 我们这里下载的是[3.4.6版本](http://apache.mirror.cdnetworks.com/zookeeper/zookeeper-3.4.6/)。

解压zookeeper-3.4.6.tar.gz文件，并将目录拷贝到/usr/目录下。并修改文件权限
```
chown -R hadoop:hadoop zookeeper-3.4.6
```

<!-- more -->

## 配置zookeeper

修改`${ZOOKEEPER_HOME}/conf/zoo.cfg`
```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/zookeeper-3.4.6/tmp/zookeeper
clientPort=2181

server.1=master.hadoop:2888:3888
server.2=slave1.hadoop:2888:3888
server.3=slave2.hadoop:2888:3888
```

进入data目录创建myid文件，在文件中加入以下内容
```
1
```

## 启动和验证

zookeeper的启动和停止命令为：
```
bin/zkServer.sh start
bin/zkServer.sh stop
```

验证过程，连接zookeeper
```
bin/zkCli.sh -server master.hadoop:2181
```

或者使用[文档](http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_zkMulitServerSetup)中的java或者c-client进行连接。

下面图片就是使用zookeeper的`create/get/set/delete`命令的测试。

{% img center /images/blogimages/2015/zk_test.png %}

一点说明：

* 配置文件

Every machine that is part of the ZooKeeper ensemble should know about every other machine in the ensemble. You accomplish this with the series of lines of the form server.id=host:port:port. The parameters host and port are straightforward. You attribute the server id to each machine by creating a file named myid, one for each server, which resides in that server's data directory, as specified by the configuration file parameter dataDir.

Finally, note the two port numbers after each server name: " 2888" and "3888". Peers use the former port to connect to other peers. Such a connection is necessary so that peers can communicate, for example, to agree upon the order of updates. More specifically, a ZooKeeper server uses this port to connect followers to the leader. When a new leader arises, a follower opens a TCP connection to the leader using this port. Because the default leader election also uses TCP, we currently require another port for leader election. This is the second port in the server entry.

* myid

The myid file consists of a single line containing only the text of that machine's id. So myid of server 1 would contain the text "1" and nothing else. The id must be unique within the ensemble and should have a value between 1 and 255.


## 将zookeeper复制到其他节点上
例如如下的命令：
```
scp -r zookeeper-3.4.6 slave1.hadoop:/usr/
```

然后在这些节点上修改文件的权限
```
chown -R hadoop:hadoop zookeeper-3.4.6
```

然后修改对应的`/usr/zookeeper-3.4.6/tmp/zookeeper/myid`文件, 将myid文中的数字改为了与zoo.cfg中对应的id。


## 参考
<http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html>
<http://zookeeper.apache.org/doc/trunk/zookeeperAdmin.html#sc_zkMulitServerSetup>