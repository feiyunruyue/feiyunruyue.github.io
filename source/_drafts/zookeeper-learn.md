title: zookeeper-learn
date: 2016-04-03 21:20:58
categories:
tags:
---
ZooKeeper 为分布式应用提供高性能的协调服务。它提供了一些分布式开发要用到的基础服务，我们可以在这些服务基础上进行编程。它的设计参考了文件系统，但是与常见的文件系统不同的是，目录也可以有数据。ZooKeeper的一个设计目标是提供简单的API，只有新增、获取数据、设置数据、删除节点等操作。

standalone模式的ZooKeeper很容易搭建，本篇博客着重记录下如何在一台服务器上模拟ZooKeeper集群效果。

	1.  新建3个conf文件，需要修改dataDir和clientPort
	tickTime=2000
	dataDir=/var/lib/zookeeper1
	clientPort=2181
	initLimit=5
	syncLimit=2
	server.1=127.0.0.1:2888:3888
	server.2=127.0.0.1:2889:3889
	server.3=127.0.0.1:2890:3890

	2. 在每个dataDir目录下新建一个文件myid，文件内容分别为1、2、3
	对应conf文件中的server.X
	3. 启动3个进程，可以进行测试了。