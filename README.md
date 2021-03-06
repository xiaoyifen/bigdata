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
核心思想：`分而治之，化繁为简`<br>
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
* 不支持并行的写操作
* 支持并行的append
#### 基本操作命令
1. 查询命令
```　
hdfs dfs -ls /   // 查询/目录下的所有文件和文件夹
hdfs dfs -ls -R  // 以递归的方式查询/目录下的所有文件
```
2. 创建文件夹
```
hdfs dfs -mkdir /test    // 创建test文件夹
```
3. 创建新的空文件
```
hdfs dfs -touchz /aa.txt   // 在/目录下创建一个空文件aa.txt
```
4. 添加文件
```
hdfs dfs -put aa.txt /test // 将当前目录下的aa.txt文件复制到/test目录下（把-put换成-copyFromLocal效果一样-moveFromLocal会移除本地文件）
```
5. 查看文件内容
```
hdfs dfs -cat /test/aa.txt     // 查看/test目录下文件aa.txt的内容（将-cat 换成-text效果一样）
```
6. 复制文件 
```
hdfs dfs -get /test/aa.txt .   // 将/test/aa.txt文件复制到当前目录（.是指当前目录，也可指定其他的目录，把-get换成-copyToLocal效果一样）
```
7. 删除文件或文件夹
```
hdfs dfs -rm -r /test/aa.txt   // 删除/test/aa.txt文件（/test/aa.txt可以替换成文件夹就是删除文件夹）
```
8. 重命名文件
```
hdfs dfs -mv /aa.txt /bb.txt   // 将/aa.txt文件重命名为/bb.txt
```
9. 将源目录中的所有文件排序合并到一个本地文件
```
hdfs dfs -getmerge / local-file     // 将/目录下的所有文件合并到本地文件local-file中
```
  
### MapReduce
MapReduce是一种编程模型，用于大规模数据集的并行运算。Map（映射）和Reduce（化简），采用分而治之思想，先把任务分发到集群多个节点上，并行计算，然后再把计算结果合并，从而得到最终计算结果。<br>
在MapReduce中，一个准备提交执行的应用程序称为“作业（job）”，而从一个作业划分出的运行于各个计算节点的工作单元称为“任务（task）”。
#### JobTracker
* 作业跟踪器，运行到Master节点上，是MapReduce体系的调度器。用于处理作业的后台程序，决定有哪些文件参与作业的处理，然后把作业切割成为一个个的小task，并把它们分配到所需要的数据所在的子节点。
* 每个集群只有唯一一个JobTracker
#### TaskTracker
* 任务跟踪器，位于每个slave节点上，与DataNode结合（代码与数据一起的原则），管理各自节点上的task（由jobtracker分配）
* 每个节点只有一个tasktracker，但一个tasktracker可以启动多个JVM，用于并行执行map或reduce任务
* 可以与jobtracker交互通信，告知jobtracker子任务完成情况。
### YARN
YARN是Hadoop 2.0中的资源管理系统，它的基本设计思想是将MRv1中的JobTracker拆分成了两个独立的服务：一个全局的资源管理器ResourceManager和每个应用程序特有的ApplicationMaster。其中ResourceManager负责整个系统的资源管理和分配，而ApplicationMaster负责单个应用程序的管理。<br>
    
YARN的基本组成结构，YARN主要由ResourceManager、NodeManager、ApplicationMaster和Container等几个组件构成。<br>
* ResourceManager: ResourceManager是Master上一个独立运行的进程，负责集群统一的资源管理、调度、分配等等
* ApplicationMaster: 管理YARN内运行的应用程序的每个实例，运行于某一个Slave节点的Container中。为应用程序申请资源并进一步分配给内部任务；协调应用程序内的所有任务的执行；任务监控与容错。
* NodeManager: NodeManager是Slave上一个独立运行的进程，负责每个节点上的资源和使用，负责上报节点的状态
* Container: yarn中分配资源的一个单位，包括内存、CPU等等资源，yarn以Container为单位分配资源

Zookeeper
----------
ZooKeeper 是一个开源的分布式协调服务，它是集群的管理者，监视着集群中各个节点的状态，根据节点提交的反馈进行下一步合理操作。最终，将简单易用的接口和性能高效、功能稳定的系统提供给用户。分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、命名服务、分布式协调/通知、集群管理、Master 选举、配置维护，名字服务、分布式同步、分布式锁和分布式队列等功能。
* 多个ZooKeeper维护一组共同的数据状态
* 支持分布式的读和写操作
* 2f+1个ZooKeeper节点可以容忍f个节点故障，仍然正确
* ZooKeeper 的数据模型 ：Data Tree（类似文件系统）
* Zookeeper是Hadoop和Hbase的重要组件

Hbase
----------
Apache HBase是基于Hadoop构建的一个分布式的、可伸缩的海量数据存储系统。HBase常被用来存放一些结构简单，但数据量非常大的数据(通常在TB级别以上)，如历史订单记录，日志数据，监控Metris数据等等，HBase提供了简单的基于Key值的快速查询能力。
* 面向列
* NoSQL的Key/vale数据库

Kafka
----------

spark
----------
