# 什么是HBase

HBase的原型是Google的Big Table论文，受到了该论文思想的启发，目前作为Hadoop的子项目来开发维护，用于支持结构化的数据存储，**实现在HDFS上的随机读写(核心)**

HBase是一个高可靠、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群

HBase的目标是存储并处理大型的数据，仅需使用普通的硬件配置，就能够处理成千上万的行和列所组成的大型数据

> HBase是Google Bigtable的开源实现，但也有很多不同之处：

- Google Bigtable利用GFS作为其文件存储系统，HBase利用Hadoop HDFS作为其文件存储系统
- Google Bigtable运行MapReduce来处理Bigtable中的海量出局，HBase同样利用Hadoop MapReduce来处理HBase中的海量数据
- Google Bigtable利用Chubby作为协同服务，HBase利用Zookeeper作为协同服务

> HBase特点

+ 海量存储

  HBase适合存储PB级别的海量数据，在PB级别的数据以及采用廉价PC存储的情况下，能在几十到百毫秒内返回数据，这与HBase的记忆拓展性息息相关，为海量数据的存储提供了便利

+ 列式存储(列族存储)

  HBase是根据列族来存储数据的，列族下面可以有非常多的列，列族在创建HBase表的时候就必须指定，列(在HBase下为真实的数据)不需要指定

+ 极易扩展

  HBase的扩展性主要体现在两个方面，一个是基于上层处理能力(RegionServer)的扩展，一个是基于存储的扩展(HDFS)。通过横向添加RegionServer的机器，进行水平扩展，提升HBase上层的处理能力，提升HBase服务更多Region的能力

+ 高并发

  在并发的情况下，HBase的单个IO延迟下降并不多，能够获得高并发、低延迟的服务

+ 稀疏

  稀疏主要针对HBase列的灵活性，在列族中，可以指定任意多的列，在列数据为空的情况下，是不会占用存储空间的

> HBase架构

