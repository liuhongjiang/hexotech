title: hadoop port
date: 2016-01-20 13:48:18
tags:
categories: hadoop
---

hadoop各个模块的默认port：

|	Application				|	Port	|	Describe 											|
|:-------------------------:|:---------:|:-----------------------------------------------------:|
|	zookeeper				|	2181	|	zookeeper client port								|
|							|	3888	|	server port to connect to leader					|
|	kafka					|	9092	|														|
|	hadoop-ResourceManager	|	8088	|	yarn.resourcemanager.webapp.address 				|
|							|	8030	|	yarn.resourcemanager.scheduler.address 				|
|							|	8031	|	yarn.resourcemanager.resource-tracker.address 		|
|							|	8032	|	yarn.resourcemanager.address 						|
|							|	8033	|	yarn.resourcemanager.admin.address 					|
|	hadoop-Nodemanager		|	8040	|	yarn.nodemanager.localizer.address    				|
|							|	8042	|	yarn.nodemanager.webapp.address 					|
|	hadoop-DataNode			|	50010	|	dfs.datanode.address 								|
|							|	50020	|	dfs.datanode.ipc.address 							|
|	hadoop-NameNode			|	8020	|	dfs.namenode.rpc-address 							|
|							|	50070	|	dfs.namenode.http-address 							|
|	hadoop-ha				|	8485	|	dfs.namenode.shared.edits.dir 						|
|							|	8480	|	dfs.journalnode.http-address 						|
|							|	8019	|	dfs.ha.zkfc.port(for DFSZKFailoverController)		|
|	spark					|	7077	|	master												|
|							|	8080	|	web 												|


