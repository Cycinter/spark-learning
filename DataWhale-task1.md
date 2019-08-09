# Spark入门第八期 task1
## 操作部分-Windows下spark部署与程序运行

校验文件哈希值
https://jingyan.baidu.com/article/67662997a9b06654d51b84a1.html
Get-FileHash D:\spark\spark-2.4.3-bin-hadoop2.7.tgz -Algorithm SHA256| Format-List


### 用python安装
直接先pip3 --upgrade pip 报错, 按网上删掉报错的pip(会重新安装的)
后又报错


pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyspark

![e13702e0889ce14cee21b068bdb9d608.png](en-resource://database/8733:1)
```python
import findspark
#可在环境变量中进行设置，即PATH中加入如下地址
findspark.init("D:\spark\spark-2.4.1-bin-hadoop2.7")
from pyspark import SparkContext as sc
from pyspark import SparkConf as conf
```
- 程序运行
```python
import findspark
#可在环境变量中进行设置，即PATH中加入如下地址
findspark.init("D:\spark\spark-2.4.3-bin-hadoop2.7")
from pyspark import SparkContext as sc
from pyspark import SparkConf as conf
from pyspark.sql import SparkSession

sc = sc( 'local', 'test')
logFile = "D:/spark/spark-2.4.3-bin-hadoop2.7/README.md"
logData = sc.textFile(logFile, 2).cache()
numAs = logData.filter(lambda line: 'a' in line).count()
numBs = logData.filter(lambda line: 'b' in line).count()
print('Lines with a: %s, Lines with b: %s' % (numAs, numBs))
```

## 内容学习笔记
### 简介

Spark仅使用了十分之一的计算资源，获得了比Hadoop快3倍的速度。新纪录的诞生，使得Spark获得多方追捧，也表明了Spark可以作为一个更加快速、高效的大数据计算平台。

#### 特点

-  运行速度快：Spark使用先进的DAG（Directed Acyclic Graph，有向无环图）执行引擎
- 容易使用：Spark支持使用Scala、Java、Python和R语言进行编程，简洁的API
- 通用性：Spark提供了完整而强大的技术栈，包括SQL查询、流式计算、机器学习和图算法组件
- 运行模式多样：Spark可运行于独立的集群模式中，或者运行于Hadoop中，也可运行于Amazon EC2等云环境中，并且可以访问HDFS、Cassandra、HBase、Hive等多种数据源。

Spark会在更多的应用场景中发挥重要作用。

#### Spark相对于Hadoop的优势
Hadoop最主要的缺陷是其MapReduce计算模型延迟过高，无法胜任实时、快速计算的需求，因而只适用于离线批处理的应用场景。

1. 从Hadoop的工作流程可以发现如下**缺点**：
- 表达能力有限。计算都必须要转化成Map和Reduce两个操作，但这并不适合所有的情况，难以描述复杂的数据处理过程；
- 磁盘IO开销大。每次执行时都需要从磁盘读取数据，并且在计算完成后需要将中间结果写入到磁盘中，IO开销较大；
- 延迟高。一次计算可能需要分解成一系列按顺序执行的MapReduce任务，任务之间的衔接由于涉及到IO开销，会产生较高延迟。而且，在前一个任务执行完成之前，其他任务无法开始，难以胜任复杂、多阶段的计算任务。

2. Spark在借鉴Hadoop MapReduce优点并解决其问题. 相比于MapReduce，Spark主要具有如下**优点**：
- Spark的计算模式也属于MapReduce，但不局限于Map和Reduce操作，还提供了多种数据集操作类型，编程模型比MapReduce更灵活；
- Spark提供了内存计算，中间结果直接放到内存中，带来了更高的迭代运算效率；
- Spark基于DAG的任务调度执行机制，要优于MapReduce的迭代执行机制。


3. Spark最大的**特点**就是将计算数据、中间结果都存储在内存中，大大减少了IO开销，因而，Spark更**适合于迭代运算比较多的数据挖掘与机器学习运算。**使用Hadoop进行迭代计算非常耗资源，因为每次迭代都需要从磁盘中写入、读取中间数据，IO开销大。而Spark将数据载入内存后，之后的迭代计算都可以直接使用内存中的中间结果作运算，避免了从磁盘中频繁读取数据。

4. 在实际进行开发时，使用Hadoop需要编写不少相对**底层的代码，不够高效**. 相对而言，Spark提供了多种高层次、简洁的API，通常情况下，对于实现相同功能的应用程序，Spark的**代码量要比Hadoop少2-5倍**。更重要的是，Spark提供了**实时交互式编程反馈**，可以方便地验证、调整算法。

5. 尽管Spark相对于Hadoop而言具有较大优势，但Spark并不能完全替代Hadoop，**主要用于替代Hadoop中的MapReduce计算模型**。实际上，Spark已经很好地融入了Hadoop生态圈，并成为其中的重要一员，它可以借助于YARN实现资源调度管理，借助于HDFS实现分布式存储。此外，Hadoop可以使用廉价的、异构的机器来做分布式存储与计算，但是，**Spark对硬件的要求稍高一些，对内存与CPU有一定的要求**。


#### Spark生态系统
**在实际应用中，大数据处理主要包括以下三个类型**：
- 复杂的批量数据处理：时间跨度通常在数十分钟到数小时之间；
- 基于历史数据的交互式查询：时间跨度通常在数十秒到数分钟之间；
- 基于实时数据流的数据处理：时间跨度通常在数百毫秒到数秒之间。

Spark可以部署在资源管理器YARN之上，提供一站式的大数据解决方案。因此，Spark所提供的生态系统足以应对上述三种场景，即同时支持批处理、交互式查询和流数据处理。

BDAS的架构如图所示，Spark专注于数据的处理分析，而数据的存储还是要借助于Hadoop分布式文件系统HDFS、Amazon S3等来实现的。因此，Spark生态系统可以很好地实现与Hadoop生态系统的兼容，使得现有Hadoop应用程序可以非常容易地迁移到Spark系统中。
![cfa0666f6425bb5168a56f3c4d67b065.jpeg](en-resource://database/8735:0)

图 BDAS架构
#### Spark的生态系统
主要包含了Spark Core、Spark SQL、Spark Streaming、MLLib和GraphX 等组件，各个组件的具体功能如下：
*  Spark Core：Spark Core包含Spark的基本功能，如内存计算、任务调度、部署模式、故障恢复、存储管理等。Spark建立在统一的抽象RDD之上，使其可以以基本一致的方式应对不同的大数据处理场景；通常所说的Apache Spark，就是指Spark Core；
*  Spark SQL：Spark SQL允许开发人员直接处理RDD，同时也可查询Hive、HBase等外部数据源。Spark SQL的一个重要特点是其能够统一处理关系表和RDD，使得开发人员可以轻松地使用SQL命令进行查询，并进行更复杂的数据分析；
*  Spark Streaming：Spark Streaming支持高吞吐量、可容错处理的实时流数据处理，其核心思路是将流式计算分解成一系列短小的批处理作业。Spark Streaming支持多种数据输入源，如Kafka、Flume和TCP套接字等；
*  MLlib（机器学习）：MLlib提供了常用机器学习算法的实现，包括聚类、分类、回归、协同过滤等，降低了机器学习的门槛，开发人员只要具备一定的理论知识就能进行机器学习的工作；
*  GraphX（图计算）：GraphX是Spark中用于图计算的API，可认为是Pregel在Spark上的重写及优化，Graphx性能良好，拥有丰富的功能和运算符，能在海量数据上自如地运行复杂的图算法。


# 问题拓展
### 1. spark 和 mapreduce有哪些区别，请用具体的例子说明？
### 2. rdd的本质是什么？
