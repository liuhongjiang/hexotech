title: Install spark
date: 2015-08-25 10:55:55
tags:
categories: hadoop
---

前面一片文章[install hadoop](/2015/08/24/install-hadoop/)介绍了hadoop的安装，这一片安装spark的文章，就是在hadoop已经安装好的基础上进行的。

## 准备

下载[spark](http://spark.apache.org/downloads.html)

hadoop的版本为2.7.1，下载的是spark-1.4.1-bin-hadoop2.6.tgz。

这里下载是已经编译好的，也可以下载源码，自己编译。

<!-- more -->

## 配置spark
本章节的内容都是在master操作。

解压spark-1.4.1-bin-hadoop2.6.tgz，移至`/usr/`目录下
```
mv spark-1.4.1-bin-hadoop2.6 /usr/spark-1.4.1
```

1. 配置spark-env.sh
在目录`${SPARK_HOME}/conf`中，将`spark-env.sh.template`拷贝为`spark-env.sh`，添加以下内容：
```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
export SPARK_DIST_CLASSPATH=$(/usr/hadoop/bin/hadoop classpath)
```

2. 配置spark-defaults.conf
在目录`${SPARK_HOME}/conf`中，将`spark-defaults.conf.template`拷贝为`spark-defaults.conf`，修改以下内容：
```
# Example:
spark.master                     spark://master.hadoop:7077
# spark.eventLog.enabled           true
# spark.eventLog.dir               hdfs://namenode:8021/directory
# spark.serializer                 org.apache.spark.serializer.KryoSerializer
spark.driver.memory              500m
# spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
```

3. 配置slaves文件
在目录`${SPARK_HOME}/conf`中，将`slaves.template`拷贝为`slaves`，修改以下内容：
```
slave1.hadoop
slave2.hadoop
```

4. 修改/etc/profile
```
# set spark path
export SPARK_HOME=/usr/spark-1.4.1
export SPARK_MASTER_IP=master.hadoop
```

## 拷贝到slave节点
在master上操作完成后，需要将spark-1.4.1的目录拷贝到所有的slave节点上。
```
scp -r spark-1.4.1 slave2.hadoop:/usr/
```

然后将`/etc/profile`拷贝到slave节点上。
或者在profile文件中添加配置
```
# set spark path
export SPARK_HOME=/usr/spark-1.4.1
export SPARK_MASTER_IP=master.hadoop
```

## 启动spark
启动了hadoop以后，可以通过下面的命令启动spark
```
${SPARK_HOME}/sbin/start-all.sh
```

## 验证安装情况
1. 运行自带示例
$ bin/run-example  org.apache.spark.examples.SparkPi

2. 查看集群环境
http://master:8080/

3. 进入spark-shell
```
${SPARK_HOME}/bin/spark-shell
```

4. 查看jobs等信息

当spark运行的时候，可以通过下面的网址产看job信息

http://master:4040/jobs/

## reference

[http://blog.csdn.net/jediael_lu/article/details/45310321]()