title: install hadoop
date: 2015-08-24 11:16:09
tags:
categories: hadoop
---

## 准备工作

centos版本是centos7。因为最新的hadoop版本（2.7.1）需要libc2.14以上，但是centos6.5只能升级到libc2.12所以需要使用centos7.
1个moaster节点，2个slave节点。

1. 修改服务器名字
centos7现在哟一个hostname的查看和设置命令
```
$ hostnamectl status
$ sudo hostnamectl set-hostname myhost.xmodulo.com
```
我设置之后的情况如下：
```
10.34.130.40 master.hadoop.pt2
10.34.130.41 slave1.hadoop.pt2
10.34.130.42 slave2.hadoop.pt2
````

2. 修改hosts文件
在所有server的`/etc/hosts`中加入这几行：
```
10.34.130.40 master.hadoop.pt2
10.34.130.41 slave1.hadoop.pt2
10.34.130.42 slave2.hadoop.pt2
```
设置完毕以后，在每一台server上使用以下命令确保hosts配置正确
```
ping master.hadoop.pt2
ping slave1.hadoop.pt2
ping slave2.hadoop.pt2
```

<!-- more -->

3. 查看GLIBC的版本, 使用这个命令查看
```
strings /lib64/libc.so.6 | grep GLIBC
```
如果命令执行时，没有/lib64/lib.so.6这个文件，那么就安装glibc，命令是:`yum install glibc`

运行的结果应该是这个样子的
```
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_PRIVATE
```
如果没有大于等于GLIBC_2.14，那么需要更新glibc，因为最新的hadoop版本（2.7.1）是需要GLIBC_2.14及以上版本。


4. 安装jdk
在所有的节点上安装jdk
```
yum install java-1.8.0-openjdk
yum install java-1.8.0-openjdk-devel
```
在centos上`/etc/profile`配置openjdk环境：
```
# set java environment
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
export JRE_HOME=/usr/lib/jvm/java-1.8.0-openjdk/jre
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
```


5. 添加普通用户hadoop
```
adduser hadoop
```

6. 添加信任关系
需要配置所有的Slave无密码登录Master
step1. 安装ssh和rsync
step2. 生成public key: `ssh-keygen`
step3. 将id_rsa.pub文件中的内容添加到authorized_keys中，authorized_keys文件的全新需要是`600`.
step4. 通过ssh命令验证信任关系，例如：
```
ssh slave2.hadoop.pt2
```

7. 关闭防火墙，或者配置防火墙
防火墙的配置可以参考网上的资料，这里仅仅关闭防火墙
centos 6 直接关闭 iptables

```
service iptables stop
```
也可以通过命令`ntsysv`，关闭开机时启动iptables服务

centos 7 需要通过下面的命令关闭firewall (这个很关键，弄了半天才解决)
```
systemctl stop firewalld
```
可以参考[CentOS 7.0 关闭firewalld防火墙指令 及更换Iptables防火墙](http://www.612459.com/fuwuq/1063.html)

## Install hadoop
以下操作，可以先在master节点上操作完成，然后将hadoop目录拷贝到其他节点上

1. 登录root账户，下载hadoop的安装文件，这里下载的是hadoop-2.7.1，同时解压到`/usr/`目录下
2. 修改hadoop-2.7.1的属性：`chown -R hadoop:hadoop hadoop-2.7.1`。
3. 在hadoop的目录下创建tmp
```
mkdir /usr/hadoop-2.7.1/tmp
chown -R hadoop:hadoop /usr/hadoop-2.7.1/tmp
```
4. 添加hadoop的路径到`/etc/profile`
```
# set hadoop path
export HADOOP_HOME=/usr/hadoop-2.7.1
export PATH=$PATH:$HADOOP_HOME/bin
```

## 配置hadoop

1. 配置hadoop-env.sh
修改`${HADOOP_HOME}/etc/hadoop/hadoop-env.sh`
在文件中修改这一行
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
```

2. 配置core-site.xml

修改`${HADOOP_HOME}/etc/hadoop/core-site.xml`文件, 在configuration的tag中添加：

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master.hadoop.pt2:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/hadoop-2.7.1/tmp</value>
        <description>A base for other temporary directories.</description>
    </property>
</configuration>
```

备注：如没有配置hadoop.tmp.dir参数，此时系统默认的临时目录为：/tmp/hadoo-hadoop。而这个目录在每次重启后都会被干掉，必须重新执行format才行，否则会出错。


3. 配置hdfs-site.xml

修改`${HADOOP_HOME}/etc/hadoop/hdfs-site.xml`, 

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

备注：dfs.replication 是数据副本数量，默认为3，salve少于3台就会报错

4. 配置mapred-site.xml

修改`${HADOOP_HOME}/etc/hadoop/mapred-site.xml`

```
<configuration>
	<property>
    	<name>mapreduce.framework.name</name>
    	<value>yarn</value>
	</property>
</configuration>
```

5. 配置yarn-site.xml

修改`${HADOOP_HOME}/etc/hadoop/yarn-site.xml`

```
<configuration>
	<property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master.hadoop.pt2</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

5. 配置slaves文件
修改`${HADOOP_HOME}/etc/hadoop/slaves`, 添加：

```
slave1.hadoop.pt2
slave2.hadoop.pt2
```

## 拷贝到slave节点

通过以下命令拷贝到slave上

```
scp -r hadoop-2.7.1 root@slave2.hadoop.pt2:/usr/
```

在各个slave节点上，进入'/usr'目录下改变目录的属性

```
chown -R hadoop:hadoop hadoop-2.7.1/
```

并且在`/etc/profile`中添加hadoop的配置：

```
# set hadoop path
export HADOOP_HOME=/usr/hadoop-2.7.1
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```


## 启动和验证

在"master"上使用普通用户hadoop进行操作

```
hdfs namenode -format
```

启动hadoop
```
start-all.sh
or
start-dfs.sh
start-yarn.sh
```

查看状态：
```
hdfs dfsadmin -report
```

例子如下：
{% img center /images/blogimages/2015/hadoop-status.png %}

另外可以通过网页查看例如：http://master:50070

查看hadoop的任务运行情况：http://master:8088

### 问题

如果无法查看到datanote，可能有很多问题引起的。
有一个办法，就是重新format namenode，然后再排除其他问题。
具体：删除slave和master的hadoop.tmp.dir内容，然后再次运行`hdfs namenode -format`。
这种方式，将会清除所有数据。


## reference:

http://www.powerxing.com/install-hadoop-cluster/
http://www.centoscn.com/image-text/install/2014/1121/4158.html