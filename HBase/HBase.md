# 什么是HBase

HBase的原型是Google的Big Table论文，受到了该论文思想的启发，目前作为Hadoop的子项目来开发维护，**用于支持结构化的数据存储，实现在HDFS上的随机读写(核心)**

HBase是一个高可靠、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群

HBase的目标是存储并处理大型的数据，仅需使用普通的硬件配置，就能够处理成千上万的行和列所组成的大型数据

> HBase是Google Bigtable的开源实现，但也有很多不同之处：

- Google Bigtable利用GFS作为其文件存储系统，HBase利用Hadoop HDFS作为其文件存储系统
- Google Bigtable运行MapReduce来处理Bigtable中的海量出局，HBase同样利用Hadoop MapReduce来处理HBase中的海量数据
- Google Bigtable利用Chubby作为协同服务，HBase利用Zookeeper作为协同服务

# HBase特点

+ 海量存储

  HBase适合存储PB级别的海量数据，在PB级别的数据以及采用廉价PC存储的情况下，能在几十到百毫秒内返回数据，这与HBase的记忆拓展性息息相关，为海量数据的存储提供了便利

+ 列式存储(列族存储)

  HBase是根据列族来存储数据的，列族下面可以有非常多的列，列族在创建HBase表的时候就必须指定，列(在HBase下为真实的数据)不需要指定

+ 极易扩展

  HBase的扩展性主要体现在两个方面，一个是基于上层处理能力(RegionServer)的扩展，一个是基于HDFS的DataNode存储的扩展。通过横向添加RegionServer的机器，进行水平扩展，提升HBase上层的处理能力，提升HBase服务更多Region的能力

+ 高并发

  在并发的情况下，HBase的单个IO延迟下降并不多，能够获得高并发、低延迟的服务

+ 稀疏性

  稀疏主要针对HBase列的灵活性，在列族中，可以指定任意多的列，在列数据为空的情况下，是不会占用存储空间的

# HBase架构
![HBase架构图](https://github.com/guojinshan/Keep_learning/blob/main/HBase/HBase%E6%9E%B6%E6%9E%84%E5%9B%BE.jpg)

+ Client: 触发读写请求，包含了访问HBase的接口，还维护了对应的cache来加速HBase的访问，如cache的Meta元数据信息
+ DataNode：实质是一个Java进程，不存储数据，只管理Linux系统上的文件夹
+ Zookeeper(ZK)：实现Hmaster的高可用、HRegionServer的监控、元数据的入口及集群配置的维护等工作，启动HBase之前必须先启动Zookeeper和Hadoop
+ HMaster：为HRegionServer分配Region、维护整个集群的负载均衡和元数据信息、发现失效的Region并将其分配到正常的RegionServer上
+ HRegionServer：处理来自客户端的读写请求，是真正"干活"的结点、管理HMaster为其分配的Region、负责和底层HDFS的交互，存储数据到HDFS、负责Region变大以后的拆分、负责Storefile的合并工作 (HMaster负责发现HRegion的切分，HRegionServer实现真正的切分操作)
+ HLog：存储修改操作日志，当HBase执行写数据时，会先在HLog中追加一条日志，然后将数据写入内存。即使在系统故障时，数据可以通过该日志文件重建。一台HRegionServer中只能有一个HLog
+ HRegion：HBase表的分片，HBase表会根据RowKey值被切分成不同的HRegion存储在HRegionServer中，在一个HRegionServer可以有多个不同的HRegion
+ Store：一个Store只对应HBase表中的一个列族(但一个列族切分后可以对应多个Store)，一个HRegion中可以包含多个Store, 即多个列族，不建议包含多个列族，容易导致HRegion切分的问题
+ Mem Store：写入数据在内存中存储
+ StoreFile: Flush刷写到HDFS中存储，是实际的存储文件
+ HFile：为StoreFile在HDFS上的存储格式
+ HDFS Client: 实现HBase与HDFS的Flush操作
+ HDFS: 为HBase提供最终的底层数据存储服务和高可用的支持

# HBase数据结构
+ RowKey(行键): 用来检索记录的主键，访问HBase table中的行，表结构中是唯一的，包括三种方式：
  1. 通过单个RowKey访问
  2. 通过RowKey的range(正则)
  3. 全表扫描
  RowKey可以是任意字符串(最大长度为64KB，实际应用中长度一般为70-100bytes)，在HBase内部，RowKey保存为字节数组。存储时，数据按照RowKey的字典序(byte order)排序存储.设计RowKey时，要充分利用排序存储这个特性，将经常一起读取的行存储放到一起(位置相关性)
+ Column Family(列族): HBase表中的每个列，都归属于每个列族。列族是表的Schema的一部分(而列不是)，必须在使用表之前定义。列名都以列族作为前缀，例如courses:history, courses:math都属于courses这个列族
+ Cell(单元): 由{rowkey, column Family:column, version}唯一确定的单元，cell中的数据是没有类型的，全部是字节码形式存储
+ Time Stamp(时间戳): 数据的多版本基于时间戳实现，每一个Cell中都保存着同一份数据的多个版本，版本通过时间戳来索引。时间戳的类型是64位整型，可以由HBasez在数据写入时自动赋值，精确到毫秒的当前系统时间，也可以由客户显示赋值。如果应用程序要避免版本冲突，就必须自己生成唯一性的时间戳。每个Cell中，不同版本的数据按照时间倒序排序，即最新的数据排在前面
  
  为了避免数据存在过多版本造成的管理(存储和索引)负担，HBase提供了两种版本回收方式：一是保存数据的最后n个版本，二是保存最近一段时间内的版本(如最近七天)。用户可以针对每个列族进行设置
+ 命名空间：类似于MySQL中的database，用于管理所有Region/表
  1. 查看命名空间: list_namespace
  2. 创建命名空间: create_namespace 'bigdata'
  3. 命名空间下创建表: create 'bigdata:student(namespace:table)', 'info(column family)'
  4. 删除命名空间: drop_namespace 'bigdata' (namespace must be empty)

# HBase原理

> 读流程
![HBase读流程](https://github.com/guojinshan/Keep_learning/blob/main/HBase/HBase%E8%AF%BB%E6%B5%81%E7%A8%8B.jpg)

1. Client先访问zookeeper，从meta表读取region的位置，然后读取meta表中的数据。meta表中又存储了用户表的region信息
2. 根据namespace、表名和rowkey在meta表中找到对应的region信息
3. 找到这个region对应的regionserver
4. 查找对应的region
5. 先从MemStore中找数据，如果没有，再到BlockCache里面读
6. BlockCache还没有，再到StoreFile上读
7. 如果是从StoreFile中读取的数据，不是直接返回给客户端，而是先写入BlockCache，再返回给客户端

> 写流程
![HBase写流程](https://github.com/guojinshan/Keep_learning/blob/main/HBase/HBase%E5%86%99%E6%B5%81%E7%A8%8B1.jpg)



