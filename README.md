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
核心思想：分而治之，化繁为简<br>
### 整体架构
Hadoop由HDFS、MapReduce、HBase、Hive和ZooKeeper等成员组成，其中Hadoop的框架最核心的设计就是：HDFS和MapReduce。HDFS为海量的数据提供了存储，则MapReduce为海量的数据提供了计算。<br>
整体架构图<br>
* Pig是一个基于Hadoop的大规模数据分析平台，Pig为复杂的海量数据并行计算提供了一个简单的操作和编程接口； 
* Hive是基于Hadoop的一个工具，提供完整的SQL查询，可以将sql语句转换为MapReduce任务进行运行； 
* ZooKeeper:高效的，可拓展的协调系统，负责服务器节点和进程间的通信； 
* HBase是一个开源的，基于列存储模型的分布式数据库； 
* HDFS是一个分布式文件系统，有着高容错性的特点，适合那些超大数据集的应用程序； 
* MapReduce是一种编程模型，用于大规模数据集（大于1TB）的并行运算。
在分布式存储和分布式计算方面，Hadoop都是采用主/从（Master/Slave）架构。NameNode、Secondary NameNode、JobTracker运行在Master节点上，而在每个Slave节点上，部署一个DataNode和TaskTracker，以便这个Slave服务器运行的数据处理程序能尽可能直接处理本机的数据。对Master节点需要特别说明的是，在小集群中，Secondary NameNode可以属于某个从节点；在大型集群中，NameNode和JobTracker被分别部署在两台服务器上。<br>
### HDFS
HDFS, Hadoop Distributed File System, 分布式文件系统<br>
#### NameNode
* NameNode位于Master节点上，且只有一个。
* 功能：对整个分布式文件系统进行总控制，存储文件的元数据(文件名，长度，分成多少数据块，每个数据块分布在哪些DataNode上)，是核心节点。
* Namenode不持久化存储数据块的位置信息，因为这些信息会在系统启动时从DataNode重建。
> 数据块：基本存储单位，默认大小为64M，每个数据块独立地分布存储在DataNode上，默认每个数据块存储3份，在3个不同的DataNode上。
#### DataNode
* DataNode位于Slave节点上，且每个Slave节点都有一个
* 功能：存储数据块；负责数据的读写操作和复制操作
* DataNode启动时会向NameNode报告当前存储的数据块信息，后续也会定时报告修改信息
* DataNode之间会进行通信，复制数据块，保证数据的冗余性
#### SecondNameNode
* 定时与NameNode进行同步（定期合并文件系统镜像和编辑日志，然后把合并后的传给NameNode，替换其镜像，并清空编辑日志），但NameNode失效后仍需要手工将其设置成主机。
#### 文件操作
##### 打开操作
* 打开文件时，与NameNode通信一次
##### 读操作
* 之后的读操作，直接与DataNode通信，绕过了NameNode
* 可以从多个副本中选择最佳的DataNode读取数据
* 支持并发的读请求
##### 写操作
* Client向NameNode发出写数据块请求，NameNode返回应写的DataNodes
* Client向DataNodes发送数据块，数据依次沿流水线传递到Primary和secondary DataNode，形成一个数据传递的pipeline
* DataNode在内存中缓存数据, 并回复确认
* Client向Primary DataNode发送写命令，Primary DataNode将写命令转发给Secondary DataNode，DataNodes收到写命令时才进行真正地写操作，把缓存的数据写到文件系统中，并返回结果
Zookeeper
----------

Hbase
----------

Kafka
----------

spark
----------
