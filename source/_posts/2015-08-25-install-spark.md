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

4. 查看sparkContext等信息

当spark运行的时候，每一个SparkContext都会启动一个web UI, 默认的端口为4040，通这个端口，可以查看一些关于这个应用非常有用的信息。
包括一下信息：
* A list of scheduler stages and tasks
* A summary of RDD sizes and memory usage
* Environmental information.
* Information about the running executors

所以第一个SparkContext，可以通过下面的网址产看job信息

http://master:4040/jobs/

当有多个sparkcontext在同一个host上运行的时候，他们将顺序绑定4040自后的端口，例如（4041,4042等等)

## 查看sparkcontext的history信息

Note that this information is only available for the duration of the application by default. To view the web UI after the fact, set spark.eventLog.enabled to true before starting the application. This configures Spark to log Spark events that encode the information displayed in the UI to persisted storage.


When using the file-system provider class (see spark.history.provider below), the base logging directory must be supplied in the spark.history.fs.logDirectory configuration option,


in the `/spark/conf/spark-defaults.conf` add:

```
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://hadoop:9000/spark/event_log
spark.history.fs.logDirectory      hdfs://hadoop:9000/spark/log_directory
```

注意：spark.eventLog.dir要与spark.history.fs.logDirectory要是一致的。

then go to the `sbin` directory:
```
 ./start-history-server.sh
```

then can open spark history on port 18080:
http://spark-host:18080/

更多信息可以参考：[spark Monitoring and Instrumentation](http://spark.apache.org/docs/latest/monitoring.html)

## reference

[http://blog.csdn.net/jediael_lu/article/details/45310321]()