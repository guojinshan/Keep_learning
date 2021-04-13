# 一、介绍、体系结构、特点

## 1. 什么是数据仓库

数据仓库是为了帮助企业各个级别进行**决策支持、分析性报告**，提供所有数据类型的战略集合，是为了便于**多维分析和多角度展现**而将数据按特定的模式进行存储所建立起来的关系型数据库，它的数据基于**OLTP源系统**。数据仓库是依照**分析需求、分析维度、分析指标**进行设计的

## 2. 数据仓库体系结构

数据源 -- ETL工程 - 数据仓库存储和管理 - 数据集市(OLAP) - BI工具(数据查询、数据报表、数据分，各类应用)

+ 数据源：业务数据、数据库数据(OLTP)、文档数据、日志数据等

![数据仓库体系结构](https://pic2.zhimg.com/80/v2-d7dbc4c757152f64ebc6cd52009ddbad_720w.jpg)

## 3. 数据仓库的特点

+ 面向主题的(Subject-Oriented): 数据仓库中的数据是按照一定的主题域进行组织

  主题：是指用户使用数据仓库进行决策时所关心的重点方面,一个主题通常与多个操作型信息系统(数据库)相关

+ 集成的(Integrated): 数据仓库中的综合数据不能从原来分散的数据库系统直接得到，而是多个不同时间点的数据库快照的集合，基于这些快照进行统计、综合和重组后增加到数据仓库中去

+ 反应历史变化的(Time Variant)：数据仓库随着时间变化不断增加新的数据内容、不断删去旧的数据内容、不断更新综合数据

+ 不可修改的(Unchangeable): 数据仓库涉及到的数据操作主要是数据查询，一般情况下并不进行修改操作

  注意：此时的‘不可修改’是针对应用来说的，也就是说，数据仓库的用户进行分析处理是不进行数据更新操作的。但并不是说，在从数据集成输入数据仓库开始到最后被删除的整个生存周期中，所有的数据仓库数据都是永远不变的

# 二、数据仓库粒度确定

## 1. 粒度级别

**粒度：**是指数据仓库的数据单位中**保存数据的细化或综合程度的级别**，细节程度越高，粒度级别就越低；细节程度越低，粒度级别就越高

+ 当数据仓库的空间很有限时，用高粒度级表示数据将比使用低粒度级表示数据的效率要高得多，并且存放数据所需要的字节数和索引项会少很多
+ 低粒度级实际上可以回答任何问题，但在高粒度级上，数据所能处理的问题的数量是有限的
+ 总结：**高粒度查询快，而低粒度可以解决的问题比较多**

>  设计数据仓库时需要考虑的三大因素
>
> + 数据量大小
> + 原始空间
> + 处理能力

通常，为了节省所用的存储空间、所需的索引项、以及处理数据的处理器资源，在数据仓库中会将数据进行压缩。

在一个**DSS（决策支持系统）**环境中**查询总体性的问题**比查询单个事件要常见的**多**，它既可以在高粒度级上也可以在低粒度级上得到回答，在不同的粒度级上所使用的资源具有很大的差异。在低粒度级需要查询每一条记录，所以需要大量的资源来回答这个问题。但在高粒度级上，数据进行了很大的压缩，只需要查询很少的记录就能得到一个答案。**如果在高粒度级上包括了足够的细节，则使用高粒度级数据的效率将会高得多**

## 2. 数仓分类

基于粒度将数据仓库分为以下三种类型：

+ 低粒度高细节数仓：如每个客户每个月的每个通话记录
+ 高粒度低细节数仓：如每个客户每个月的综合通话记录
+ 双粒度数仓：两者共存，当用户十分需要提高存储与访问数据的效率，以及非常详细地分析数据的能力

![双粒度](https://pic1.zhimg.com/80/v2-a2bd0ff308cf83dc5d7fad9cc99d3f8c_720w.jpg)

确定数据粒度是数据仓库设计的基础，当数据粒度合理确定后，设计和实现的其他问题就会变得非常容易。相反，如果没有合理地确定粒度，后续的工作就会很难进行下去。在设计和构造数据仓库之初就必须仔细考虑这种权衡。

# 三、数据仓库和数据库的区别

+ 数据量：数据仓库要比数据库庞大得多
+ 用途：数据库用来处理实际的业务，在生产环境就是用来干活的；数据仓库主要用于数据挖掘和数据分析**，辅助领导做决策
+ 数据来源：数据库的数据来自于业务，数据仓库的数据来源于多个数据库
+ 时间维度：数据库只保留当前信息，数据仓库保留数据时间轴上的全部信息
+ 组织方式：数据库是面向业务组织，数据仓库是面向主题组织
+ 建模方式：数据库服从范式建模，数据仓库一般是合理冗余的
+ 设计目的：数据库以存储、管理为主，数据仓库以组织、计算为主

> **数据库与数据仓库的区别实际讲的是OLTP与OLAP的区别**

操作型处理，叫联机事务处理OLTP(On-Line Transaction Processing)：也可以称面向交易的处理系统，它是针对具体业务在数据库联机的日常操作，通常对**少数记录**进行查询、修改。用户较为关心操作的**响应时间、数据的安全性、完整性和并发的支持用户数**等问题。传统的数据库系统作为数据管理的主要手段，主要用于操作型处理

分析型处理，叫联机分析处理OLAP(On-Line Analytical Processing)：一般针对某些主题历史数据进行分析，支持管理决策

# 五、数据仓库和数据库的区别
在设计数据仓库模型和架构时，我们需要懂具体的技术，也需要了解行业的知识和经验来帮助我们对业务进行抽象、处理，进而生成各个阶段的模型

## 1. 数据模型架构
+ 系统记录域：数据仓库业务数据存储区，保证数据的一致性
+ 内部管理域：统一地管理内部的元数据
+ 汇总域：汇总来自系统记录域的数据，保证分析域的主题分析性能，满足部分报表查询
+ 分析域：对各个业务部分的具体主题进行业务分析，可以单独存储在相应的数据集市中
+ 反馈域：用于相应的前端的反馈数据，视业务的需要设置这个域

## 2. 多维数据模型
多维数据模型是为了满足用户从多角度多层次进行数据查询和分析的需要而建立起来的基于事实和维的数据库模型，其基本的应用是为了实现OLAP

当然，通过多维数据模型的数据展示、查询和获取就是其作用的展现，但其真的作用的实现在于，通过数据仓库可以根据不同的数据需求建立起各类多维模型，并组成数据集市开放给不同的用户群体使用，也就是根据需求定制的各类数据商品摆放在数据集市中供不同的数据消费者进行采购

> 事实表
事实表(Fact Table)：是指存储有事实记录的表，如系统的日志、销售记录、用户访问日志等信息，事实表的记录是动态的增长的，其体积是大于维度表

> 维度表
维度表(Dimension Table)：也称为查找表（Lookup Table）是与事实表相对应的表，这个表保存了维度的属性值，可以跟事实表做关联，相当于是将事实表中经常重复的数据抽取、规范出来用一张表管理。如日期（日、周、月、季度、年等属性）、地区表等

+ 维度表的变化通常不会太大， 主要用来描述用户关心的业务数据，如销售数量，库存数量，销售金额等
+ 维度表的存在缩小了事实表的大小，便于维度的管理和CURD维度的属性，不必对事实表的大量记录进行改动，并且可以给多个事实表重用
+ 基于事实表和维表就可以构建出多种多维模型，包括星形模型、雪花模型和星座模型，在设计逻辑型数据的模型的时候，就应考虑数据是按照星型模型还是雪花型模型进行组织

## 3. 星型模型和雪花型模型的区别
> 星型模型： 所有维表都直接连接到事实表上
![星型模型](https://pic3.zhimg.com/v2-1d39380d9238ca7c5876ac92d27750b2_b.jpg)
+ 星型架构是一种非正规化的结构，有一张事实表和多张维度表，设计与实现都比较简单
+ 多维数据集的每一个维度都直接与事实表相连接，不存在渐变维度，所以数据有一定的冗余，因为维度表的数据冗余，所以统计查询时不需要做过多外部连接, 因此一般情况下效率比雪花型模型要高
+ 事实表和维度表通过主外键相关联，维度表之间是没有关联

> 雪花型模型：有一个或多个维表没有直接连接到事实表上，而是通过其他维表连接到事实表上
![雪花型模型](https://pic4.zhimg.com/v2-e7e1a7403be3ffb217f623d89771a573_b.jpg)
+ 雪花模型是一种正规化的结构，是对星型模型的扩展，它对星型模型的维表进一步层次化，原有的各维表可能被扩展为小的事实表，形成一些局部的 "层次 " 区域，这些被分解的表都连接到主维度表而不是事实表
+ 雪花模型通过最大限度地减少数据存储量以及联合较小的维表来改善查询性能，去除了数据冗余

> 星型模型 VS 雪花型模型
+ 雪花模型使用的是规范化数据，也就是说数据在数据库内部是组织好的，以便消除冗余，因此它能够有效地减少数据量。通过引用完整性，其业务层级和维度都将存储在数据模型之中
+ 星形模型实用的是反规范化数据。在星形模型中，维度直接指的是事实表，业务层级不会通过维度之间的参照完整性来部署
+ 在雪花模型中，数据模型的业务层级是由一个不同维度表主键-外键的关系来代表的。而在星形模型中，所有必要的维度表在事实表中都只拥有外键
+ 在冗余可以接受的前提下，实际运用中星型模型使用更多，也更有效率
+ 雪花模型在维度表、事实表之间的连接很多，因此性能方面会比较低，星形模型的连接就少的多，性能较好
+ 雪花模型加载数据集市，因此ETL操作在设计上更加复杂，而且由于附属模型的限制，不能并行化
+ 星形模型加载维度表，不需要再维度之间添加附属模型，因此ETL就相对简单，而且可以实现高度的并行化

总结：
+ 雪花模型使得维度分析更加容易，星形模型用来做指标分析更适合
+ 星型有时候规范化和效率是一组矛盾。一般我们会采取牺牲空间（规范化）来换取好的性能，把尽可能多的维度信息存在一张“大表”里面是最快的。通常会视情况而定，采取折中的策略
+ 星型有时会造成数据大量冗余，并且很有可能将事实表变的及其臃肿（上百万条数据×上百个维度）。每次遇到需要更新维度成员的情况时，都必须连事实表也同时更新。而雪花型，有时只需要更新雪花维度中的一层即可，无需更改庞大的事实表。具体问题具体分析，如时间维度，年，季就没必要做雪花，而涉及到产品和产品的分类，如果分类信息也是我们需要分析的信息，那么，我肯定是建关于分类的查找表，也就是采用雪花模式
+ 雪花型结构是一种正规化结构，它去除了数据仓库中的冗余数据。比如有一张销售事实表，然后有一张产品维度表与之相连，然后有一张产品类别维度表与产品维度表连。这种结构就是雪花型结构。雪花型结构取除了数据冗余，所以有些统计就需要做连接才能产生，所以效率不一定有星型架构高。正规化也是一种比较复杂的过程，相应数据库结构设计、数据的ETL、以及后期的维护都要复杂一些
+ 星型架构是一种非正规化的结构，多维数据集中的每一个维度都与事实表相连接，不存在渐变维度，所以数据有一定的冗余，正因为数据的冗余所以很多统计查询不需要做外部的连接所以一般情况下效率比雪花型要高。星型结构不用考虑很多正规化的因素，设计与实现都比较简单

## 4. 数据模型建立的过程
1. 业务模型：业务分解和程序化，确定好业务的边界及业务流程，如订单、支付都是一个单独的业务模块
2. 领域模型：业务概念的抽象、分组，整理分组之间的关联，比如用户购物的业务，抽成一个更大的模型，这个模型一般相对于行业
3. 逻辑建模：领域模型中的业务概念实体化，并考虑实体的具体属性及实体与实体之间的关系，比如订单（订单号、付款人…）和支付（金额、支付时间…）的关系
4. 物理模型：解决实际应用的落地开发、上线等问题，及性能等一些具体的技术问题