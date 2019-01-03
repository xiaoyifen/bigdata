大数据学习笔记
=============
目录
----
* [分布式系统](#分布式系统)
* [Hadoop](#Hadoop)
* [Zookeeper](#Zookeeper)
* [Hbase](#Hbase)
* [Kafka](#Kafka)
* [spark](#spark)

分布式系统
----------

Hadoop
----------
Hadoop是一个开源框架，允许使用简单的编程模型在跨计算机集群的分布式环境中存储和处理大数据。它的设计是从单个服务器扩展到数千个机器，每个都提供本地计算和存储。<br>
###整体架构
Hadoop由HDFS、MapReduce、HBase、Hive和ZooKeeper等成员组成，其中Hadoop的框架最核心的设计就是：HDFS和MapReduce。HDFS为海量的数据提供了存储，则MapReduce为海量的数据提供了计算。
整体架构图
（1）Pig是一个基于Hadoop的大规模数据分析平台，Pig为复杂的海量数据并行计算提供了一个简单的操作和编程接口； 
（2）Hive是基于Hadoop的一个工具，提供完整的SQL查询，可以将sql语句转换为MapReduce任务进行运行； 
（3）ZooKeeper:高效的，可拓展的协调系统，负责服务器节点和进程间的通信； 
（4）HBase是一个开源的，基于列存储模型的分布式数据库； 
（5）HDFS是一个分布式文件系统，有着高容错性的特点，适合那些超大数据集的应用程序； 
（6）MapReduce是一种编程模型，用于大规模数据集（大于1TB）的并行运算。

Zookeeper
----------

Hbase
----------

Kafka
----------

spark
----------
