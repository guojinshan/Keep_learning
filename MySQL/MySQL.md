# 1.MySQL环境

## 1.1.环境安装

```shell
# 查看Linux服务器上是否安装过MySQL
rpm -qa | grep -i mysql # 查询出所有mysql依赖包

# 1、拉取镜像
docker pull mysql:5.7

# 2、创建实例并启动
docker run -p 3306:3306 --name mysql \
-v /root/mysql/log:/var/log/mysql \
-v /root/mysql/data:/var/lib/mysql \
-v /root/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=333 \
-d mysql:5.7

# 3、mysql配置 /root/mysql/conf/my.conf
[client]
#mysqlde utf8字符集默认为3位的，不支持emoji表情及部分不常见的汉字，故推荐使用utf8mb4
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
#设置client连接mysql时的字符集,防止乱码
init_connect='SET collation_connection = utf8_general_ci'
init_connect='SET NAMES utf8'

#数据库默认字符集
character-set-server=utf8

#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server=utf8_general_ci

# 跳过mysql程序起动时的字符参数设置 ，使用服务器端字符集设置
skip-character-set-client-handshake

# 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求！
skip-name-resolve

# 4、重启mysql容器
docker restart mysql

# 5、进入到mysql容器
docker exec -it mysql /bin/bash

# 6、查看修改的配置文件
cat /etc/mysql/my.conf
```

## 1.2.安装位置

`Docker`容器就是一个小型的`Linux`环境，进入到`MySQL`容器中。

```shell
docker exec -it mysql /bin/bash
```

`Linux`环境下`MySQL`的安装目录。

| 路径                | 解释                     |
| ------------------- | ------------------------ |
| `/var/lib/mysql`    | MySQL数据库文件存放位置  |
| `/usr/share/mysql`  | 错误消息和字符集文件配置 |
| `/usr/bin`          | 客户端程序和脚本         |
| `/etc/init.d/mysql` | 启停脚本相关             |

## 1.3.修改字符集

```shell
# 1、进入到mysql数据库并查看字符集
# show variables like 'character%';
# show variables like '%char%';

mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

mysql> show variables like '%char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```



`MySQL5.7`配置文件位置是`/etc/my.cnf`或者`/etc/mysql/my.cnf`，如果字符集不是`utf-8`直接进入配置文件修改即可

```shell
[client]
default-character-set=utf8

[mysql]
default-character-set=utf8

[mysqld]
# 设置client连接mysql时的字符集,防止乱码
init_connect='SET NAMES utf8'
init_connect='SET collation_connection = utf8_general_ci'

# 数据库默认字符集
character-set-server=utf8

#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server=utf8_general_ci

# 跳过mysql程序起动时的字符参数设置 ，使用服务器端字符集设置
skip-character-set-client-handshake

# 禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求！
skip-name-resolve
```

**注意：安装`MySQL`完毕之后，第一件事就是修改字符集编码**

## 1.4.配置文件

**`MySQL`配置文件讲解：https://www.cnblogs.com/gaoyuechen/p/10273102.html**

1、二进制日志`log-bin`：主从复制

```
# my.cnf
# 开启mysql binlog功能
log-bin=mysql-bin
```

2、错误日志`log-error`：默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等

```
# my.cnf
# 数据库错误日志文件
log-error = error.log
```

3、查询日志`log`：默认关闭，记录查询的`sql`语句，如果开启会降低`MySQL`整体的性能，因为记录日志需要消耗系统资源

```
# my.cnf
# 慢查询sql日志设置
slow_query_log = 1
slow_query_log_file = slow.log
```

4、数据文件

- `frm文件`：存放表结构
- `myd文件`：存放表数据
- `myi文件`：存放表索引

```
# mysql5.7 使用.frm文件来存储表结构
# 使用.ibd文件来存储表索引和表数据
-rw-r-----  1 mysql mysql   8988 Jun 25 09:31 pms_category.frm
-rw-r-----  1 mysql mysql 245760 Jul 21 10:01 pms_category.ibd
```

`MySQL5.7`的`Innodb`存储引擎可将所有数据存放于`ibdata`的共享表空间，也可将每张表存放于独立的`.ibd`文件的独立表空间,
共享表空间以及独立表空间都是针对数据的存储方式而言的

- 共享表空间: 某一个数据库的所有的表数据，索引文件全部放在一个文件中，默认这个共享表空间的文件路径在`data`目录下。 默认的文件名为`ibdata1`初始化为`10M`
- 独立表空间: 每一个表都将会生成以独立的文件方式来进行存储，每一个表都有一个`.frm`表结构描述文件，还有一个`.ibd`文件，其中这个文件包括了单独一个表的索引及数据信息，默认情况下它的存储位置也是在表的位置之中。在配置文件`my.cnf`中设置： `innodb_file_per_table`

# 2.MySQL逻辑架构

![MySQL逻辑架构](https://img-blog.csdn.net/20180831173911997?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

- `Connectors`：指的是不同语言中与SQL的交互
- `Connection Pool`：管理缓冲用户连接，线程处理等需要缓存的需求
- ` Management Serveices & Utilities`：系统管理和控制工具，主要提供备份、安全、复制、集群、分区等功能
- `SQL Interface`：接受用户的SQL命令，并且返回用户需要查询的结果
- `Parser`：SQL语句解析器
- `Optimizer`：查询优化器，SQL语句在查询之前会使用查询优化器对查询进行优化。**就是优化客户端请求query**，根据客户端请求的query语句，和数据库中的一些统计信息，在一系列算法的基础上进行分析，得出一个最优的策略，告诉后面的程序如何取得这个query语句的结果。**For Example**： `select uid,name from user where gender = 1;`这个`select `查询先根据`where `语句进行选取，而不是先将表全部查询出来以后再进行`gender`过滤；然后根据`uid`和`name`进行属性投影，而不是将属性全部取出以后再进行过滤，最后将将两个查询条件联接起来生成最终查询结果
- `Caches & Buffers`：查询缓存
- `Pluggable Storage Engines`：存储引擎接口M,ySQL区别于其他数据库的最重要的特点就是其**插件式的表存储引擎(注意：存储引擎是基于表的，而不是数据库)**
- `File System`：数据落地到磁盘上，就是文件的存储

MySQL数据库和其他数据库相比，最大的不同主要体现在存储引擎的架构上，**插件式的存储引擎架构将查询处理和其他的系统任务，以及数据的存储提取相分离**， 这种架构可以根据实际的业务需求来选择合适的存储引擎

> 逻辑架构分层

![MySQL逻辑架构](https://img-blog.csdnimg.cn/20200801165252510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

- 连接层：最上层是一些客户端和连接服务，包含本地`Socket`通信和大多数基于客户端/服务端`(C/S)`工具实现的类似于`Tcp/Ip`的通信。主要完成一些类似于连接处理、授权认证、以及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程，同样在该层上可以实现基于`SSL`的安全链接，服务器也会为安全接入的每个客户端验证它所具有的操作权限
- 服务层：MySQL的核心服务功能层，该层是MySQL的核心，包括查询缓存，解析器，解析树，预处理器，查询优化器。主要进行查询解析、分析、查询缓存、内置函数、存储过程、触发器、视图等，Select操作会先检查是否命中查询缓存，命中则直接返回缓存数据，否则解析查询并创建对应的解析树
- 引擎层：存储引擎层，存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，这样我们可以根据自己的实际需要进行选取
- 存储层：数据存储层，主要是将数据存储在运行于裸设备的文件系统之上，并完成与存储引擎的交互

# 3.存储引擎

`SHOW ENGINES;`命令查看MySQL5.7支持的存储引擎

```
mysql> SHOW ENGINES;
```

![存储引擎](https://img-blog.csdnimg.cn/20200801170442428.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

`SHOW VARIABLES LIKE 'default_storage_engine%';`查看当前数据库正在使用的存储引擎

```
mysql> SHOW VARIABLES LIKE 'default_storage_engine%';
+------------------------+--------+
| Variable_name          | Value  |
+------------------------+--------+
| default_storage_engine | InnoDB |
+------------------------+--------+
1 row in set (0.01 sec)
```

> InnoDB和MyISAM对比

| 对比项   |                           MyISAM                           |                            InnoDB                            |
| :------- | :--------------------------------------------------------: | :----------------------------------------------------------: |
| 主外键   |                           不支持                           |                             支持                             |
| 事务     |                           不支持                           |                             支持                             |
| 行表锁   | 表锁，即使操作一条记录也会锁住整张表，**不适合高并发操作** | 行锁，操作时只锁某一行，不对其他行有影响，**适合高并发操作** |
| 缓存     |                 只缓存索引，不缓存真实数据，**非聚簇索引**                 | 不仅缓存索引还要缓存真实数据，対内存要求较高，而且内存大小対性能有决定性影响，**聚簇索引**  |
| 表空间   |                             小                             |                              大                              |
| 关注点   |                            性能                            |                             事务                             |
| 默认安装 |                             Y                              |                              Y                               |

# 4.SQL性能下降
- 表现
	- 执行时间长
	- 等待时间长  
- 原因
	- 查询语句写得差
	- 索引失效：建了索引单是没有用上，包括单值索引和复合索引
		- `SELECT * FROM Student WHERE Name='张三' AND Email = '123456@qq.com';`
		- 单值索引: 只对表中的一列建立索引 - `CREATE INDEX id_Student_Name ON Student(Name);` 	
		- 多值索引：对表中的多列进行索引 - `CREATE INDEX id_Student_Name_Email ON Student(Name, Email);`
	- 关联查询太多`join`（设计缺陷或者不得已的需求）
	- 服务器调优以及各个参数的设置（缓存、线程数等）不合理或者比例不恰当

# 5.SQL执行顺序

```
SELECT [DISTINCT]             			# 7
	<select_List> 
FROM                				# 1
	<left_table> 
[INNER|LEFT|RIGHT] JOIN 			# 3
	<right_table> 
ON  						# 2	
	<join_condition>
WHERE               				# 4
	<where_condition> 
GROUP BY            				# 5
	<group_by_list> 
HAVING              				# 6
	<having_condiction> 
ORDER BY            				# 8
	<order_by_condiction> [DESC] 
LIMIT               				# 9
        <limit_number>, [OFFSET offset_number] 
```

[MySQL解析SQL语句过程

![MySQL解析过程](https://github.com/guojinshan/Keep_learning/blob/main/MySQL/Picture/Resolve.jpg)


# 6.七种JOIN理论

![七种JOIN理论](https://img-blog.csdnimg.cn/20200801212011559.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

```
/* 1 */ 【A独有部分 + AB公有部分】
SELECT <select_list> FROM TableA A LEFT JOIN TableB B ON A.Key = B.Key;

/* 2 */ 【B独有部分 + AB公有部分】
SELECT <select_list> FROM TableA A RIGHT JOIN TableB B ON A.Key = B.Key;

/* 3 */ 【AB公有部分】
SELECT <select_list> FROM TableA A INNER JOIN TableB B ON A.Key = B.Key;

/* 4 */ 【A独有部分】
SELECT <select_list> FROM TableA A LEFT JOIN TableB B ON A.Key = B.Key WHERE B.Key IS NULL;

/* 5 */ 【B独有部分】
SELECT <select_list> FROM TableA A RIGHT JOIN TableB B ON A.Key = B.Key WHERE A.Key IS NULL;

/* 6 */ 【A独有部分 + AB公有部分 + B独有部分】
SELECT <select_list> FROM TableA A FULL OUTER JOIN TableB B ON A.Key = B.Key;

/* MySQL不支持FULL OUTER JOIN这种语法, ORACAL支持，可以改成 1+2 */
SELECT <select_list> FROM TableA A LEFT JOIN TableB B ON A.Key = B.Key
UNION
SELECT <select_list> FROM TableA A RIGHT JOIN TableB B ON A.Key = B.Key;

/* 7 */ 【A独有部分 + B独有部分】
SELECT <select_list> FROM TableA A FULL OUTER JOIN TableB B ON A.Key = B.Key WHERE A.Key IS NULL OR B.Key IS NULL;

/* MySQL不支持FULL OUTER JOIN这种语法，ORACAL支持，可以改成 4+5 */
SELECT <select_list> FROM TableA A LEFT JOIN TableB B ON A.Key = B.Key WHERE B.Key IS NULL;
UNION
SELECT <select_list> FROM TableA A RIGHT JOIN TableB B ON A.Key = B.Key WHERE A.Key IS NULL;
```

# 7.索引

## 7.1.索引简介

> 索引是什么?[参考好文，太详细](https://www.cnblogs.com/ibigboy/p/12357787.html#:~:text=%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9D%9E%E4%B8%BB%E9%94%AE%E7%B4%A2%E5%BC%95%E7%BB%93%E6%9E%84,%E5%8D%A0%E7%94%A8%E5%B0%B1%E4%BC%9A%E7%BF%BB%E5%80%8D%E3%80%82)

索引（INDEX）是帮助MySQL高效获取数据的**排好序的快速查找数据结构(本质)**, 可以类比字典的目录, 提高查找效率

在MySQL中，除了数据本身之外，**数据库系统还维护着一个满足特定查找算法的数据结构**，这些数据结构以某种方式引用(指向)数据，这样就可以在这些数据结构的基础上实现高级查找算法，这种数据结构就是索引。一般来说，索引本身也很大，不可能全部存储在内存中，往往以索引文件的形式**存储在磁盘**上，所以每次查找都需要进行磁盘访问，涉及到I/O开销

**重点：索引会影响到MySQL查找(WHERE的查询条件)和排序(ORDER BY)两大功能！**

【以二叉搜索树为例】

![基于BST实现的索引](https://github.com/guojinshan/Keep_learning/blob/main/MySQL/Picture/Index_BST.jpg?raw=true)

为了加快Col2的查找，可以维护一个二叉搜索树，每个结点同时存储索引键值和一个指向对应数据记录的物理地址的指针，这样就可以在一定的复杂度内找到索引值，返回其物理地址后获取相应记录，再处理后续的SQL语句

但在某种极端情况下，二叉搜索树容易退化成链表型二叉树，因此需要引入平衡因子，即使用AVL树使得树达到平衡，减少树的访问深度。但AVL树一个节点只能存储一个索引，磁盘上的每一页也只存储一个索引，当数据量庞大时，有N个数据需要进行N次的IO访问，开销极大。同样如果使用红黑树，在极端情况下，难以保证树的深度在3-5之间。如果使用B树（多路搜索树), 每一个节点可以存储多个索引，虽然能解决深度和多IO访问问题，但由于每个结点(叶子结点和非叶子结点)由索引和数据组成，当加载进内存时，暂用空间较大

为了解决以上数据结构所带来的问题，InnoDB中的索引使用B+树进行实现，非叶子结点只存储索引(结点大小为16KB, 使用`SHOW GLOBAL STATUS LIKE 'Innodb_page_size'`可以查询)，叶子结点存储全部索引和数据，当依次按层将索引加载进内存时，相比B树而言，能够在磁盘的单页访问中一次性查询到更多的索引节点，最大程度地减少IO访问次数所带来的开销。其中聚簇索引，覆盖索引，复合索引，前缀索引，唯一索引等，默认都是使用B+树实现，统称索引。当然，除了B+树这种数据结构的索引之外，还有哈希索引（Hash Index）， 全文索引(Full-text Index)等

```
# Linux下查看磁盘空间命令 df -h 
[root@Ringo ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        40G   16G   23G  41% /
devtmpfs        911M     0  911M   0% /dev
tmpfs           920M     0  920M   0% /dev/shm
tmpfs           920M  480K  920M   1% /run
tmpfs           920M     0  920M   0% /sys/fs/cgroup
overlay          40G   16G   23G  41% 
```

> 索引的优势和劣势

优势：
- 快速查找：类似大学图书馆的书目索引，提高数据检索的效率，**降低数据库的I/O成本**
- 排序：通过索引対数据进行排序，降低数据排序的成本，**降低了CPU计算的消耗**

劣势：
- 实际上索引也是一张表，该表保存了主键、索引字段、指向实体表记录的信息，所以索引列也是要**占用空间**的
- 虽然索引大大**提高了查询速度**，但是同时会**降低表的更新速度**，例如对表频繁地进行`INSERT`、`UPDATE`和`DELETE`等更新操作时，`MySQL`不仅要保存数据，还要保存索引文件每次更新添加的索引列的字段，并更新索引信息
- 索引只是提高效率的一个因素，如果`MySQL`有大数据量的表，就需要**花时间**研究**建立**最优秀的索引，**或优化**查询语句

## 7.2.MySQL索引分类

> [索引分类](https://www.cnblogs.com/luyucheng/p/6289714.html)：
- 单值索引：一个索引只包含单个列，一个表可以有多个单列索引
- 复合索引：一个索引包含多个列，只有在查询条件中使用了创建索引时的第一字段，索引才会被使用,遵循最左前缀集合
- 唯一索引：索引列的值必须唯一，但是允许空值，如果是组合索引，则列值的组合必须是唯一的
- 主键索引：索引列的值必须唯一，但是不允许空值(创建主键必创建索引)
- 全文索引：主要用来查找文本中的关键字，而不是直接与索引中的值相比较，目前只有在Char、Varchar、Text列上可以创建全文索引

**建议：一张表建的索引最好不要超过5个！**

> 创建索引
```
/* 基本语法 */

/* 1、创建索引 [UNIQUE|FULLTEXT] 可以省略*/
/* 如果只写一个字段就是单值索引，写多个字段就是复合索引 */
CREATE [UNIQUE|FULLTEXT] INDEX idx_tabName_indexName ON tabName(columnName1[length], [columnName2[length],...])[ASC|DESC];

/* 2、删除索引 */
DROP INDEX [idx_tabName_indexName] ON tabName;

/* 3、查看索引 */
/* 加上\G就可以以列的形式查看了 不加\G就是以表的形式查看 */
SHOW INDEX FROM tabName \G;

/* 使用`ALTER`命令来为数据表添加索引 */

/* 1、该语句创建普通索引，索引值可以出现多次 */
ALTER TABLE tabName ADD INDEX indexName(column_list);

/* 2、该语句添加一个主键，这意味着索引值必须是唯一的，并且不能为NULL */
ALTER TABLE tabName ADD PRIMARY KEY(column_list);

/* 3、该语句创建索引的键值必须是唯一的(除了NULL之外，NULL可能会出现多次) */
ALTER TABLE tabName ADD UNIQUE indexName(column_list);

/* 4、该语句指定了索引为FULLTEXT，用于全文检索 */
ALTER TABLE tabName ADD FULLTEXT indexName(column_list);
```

## 7.3.MySQL索引数据结构

> 索引数据结构：
- `BTree`B/B+树索引
- `Hash`哈希索引：MD5, CRC16/32
- `Full-text`全文索引
- `R-Tree`R数索引

> `BTree`B树索引检索原理：

![BTree](https://github.com/guojinshan/Keep_learning/blob/main/MySQL/Picture/B-Tree.png)

- 叶子结点具有相同的深度，叶节点的指针为空
- 所有结点都存储索引数据项+真实数据，且所有索引元素不重复，从左到右递增排列

> `BTree`B+树索引检索原理：

![B+Tree](https://img-blog.csdnimg.cn/20200801233134931.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

【初始化介绍】

一颗B+树(相比B树而言，能够减少磁盘I/O)，浅蓝色的块称之为磁盘块，每个磁盘块包含几个索引数据项(深蓝色所示)和指针(黄色所示), 如磁盘块1包含数据项17和35，包含指针P1,P2,P3；其中P1表示小于17的磁盘块地址，P2表示17和35之间的磁盘块地址，P3表示大于35的磁盘块地址。 **真实的数据只存在于叶子结点**，即3、5、9、10、13、15、28....，**非叶子结点不存储真实数据，只存储索引(冗余)**, 如**17，35等数据项并不真实存在于数据表**中，这样一个非叶子结点可以放更多的索引。同时**叶子结点间用双向指针连接（图中未画出），方便范围查询，提高区间访问的性能**(Hash索引虽然定位效率相比B+树而言更高，等值查询更快，但不能实现范围查找或模糊查找，同样B树的叶子结点间也没有使用指针，不支持范围查找)

【查找过程】

如果查找数据项29，那么首先会把磁盘1由磁盘加载到内存，此时发生一次IO，在内存中**即结点内,用二分查找**确定29在17和35之间锁定磁盘块1的P2指针，因为内存访问时间相比IO磁盘IO访问非常短，可以忽略不记，通过磁盘块1的P2指针的磁盘地址把磁盘块3由磁盘加载到内存，发生第二次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存通过二分查找找到29，结束查询，总计三次IO

**真实的情况是**:3层的B+树可以表示上百万的数据，如果上百万的数据查找只需要三次IO，性能提高将是巨大的。如果没有索引，每个索引数据项都要发生一次IO，那么总共需要上百万次的IO，显然成本非常非常高！！

> 聚簇索引 & 非聚簇索引

聚簇索引与非聚簇索引的区别是：**叶节点是否包含完整的数据记录**。InnoDB主键使用的是聚簇索引，MyISAM不管是主键索引，还是二级索引(非主键)使用的都是非聚簇索引

- 非聚簇索引：表索引文件(`.MYI文件`)和数据文件(`.MYD文件`)是分离的(非聚集)，使用B+树组织实现；主键索引和非主键索引没有任何区别；所有的节点都可以看作是索引，叶子节点存储的是索引+索引所对应的**数据的磁盘文件地址指针**(而不是真实的数据)

- 聚簇索引：表索引文件和数据文件是聚集的(`.ibd文件`), 同样使用B+树组织实现；所有的非叶子节点都是索引，叶子结点叶子节点存储的是索引+真实的数据；InnoDB的主键索引就是聚簇索引，主键索引的叶子结点存储行数据(包含了主键值+具体数据内容)，二级索引的叶结点存储行的主键值
	- 为什么InnoDB必须有主键，并且推荐使用整型的自增主键？
		- 如果没有建主键，MySQL会在InnoDB表中查找可以建**唯一索引**的列并在后台建立；如果没找到，则默认添加**主键索引**列来帮助维护和组织整张表的数据
		- 使用整型主键方便比较大小且暂用空间小，如果使用36位字符串型UUID(通用用户唯一标识符，随机的)，需要先转换成ASCII码后才能进行对比且暂用空间大
		- 自增：使得结点内从左到右每个元素依次递增(有序序列)，当插入新节点时减少结点分裂的次数，方便二叉查找和树的结构调整，便于维护
	- 为什么非主键索引结构叶子节点存储的是主键值？ 
		- 保证一致性:都通过主键索引来找到最终的数据，避免维护多份数据导致不一致的情况
		- 节省存储空间： 已经维护了一套主键索引+数据的B+Tree结构，如果再有其他的非主键索引的话，索引的叶子节点存储的是主键，当继续存数据的话，就导致一份数据存了多份，空间占用就会翻倍
	- [分布式自增ID算法-Snowflake雪花算法](https://blog.csdn.net/en_joker/article/details/79806061): 按照时间有序生成全局唯一ID，ID整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞，效率较高(经测试snowflake每秒能够产生26万个ID)

- 聚簇索引的优点
	- 当需要取出一定范围的数据时，用聚簇索引比非聚簇索引好
	- 当通过聚簇索引查找目标数据时，理论上要比非聚簇索引快，因为非聚簇索引定位到对应的主键时还需要多次一次目标记录寻址，即多一次I/O
	- 使用覆盖索引扫描的查询可以直接使用叶结点中的主键值
- 聚簇索引的缺点
	- 插入速度严重依赖于插入顺序，按主键的顺序插入时最快的方式，否则将会出现结点分裂，严重影响性能。因此，对于InnooDB表，我们一般都会自定义一个自增的ID列作为主键
	-  更新主键的代价很高，因为将会导致被更新的行移动。因此，对于InnoDB表，我们一般定义主键为不可更新
	-  二级索引访问需要两次索引查找，第一次找到主键值，第二次根据主键值找到行数据。二级索引的叶结点存储的是主键值，而不是行指针，这是为了当出现行移动或结点分裂时，减少二级索引的维护工作，但会让二级索引占用更多的空间
	-  采用聚簇索引插入新值比采用非聚簇索引插入新值的速度要慢很多，因为插入要保证主键不能重复，判断主键不能重复，采用的方式在不同的索引下面会有很大的性能差距，聚簇索引遍历所有的叶子节点，非聚簇索引也判断所有的叶子节点，但是聚簇索引的叶子节点除了带有主键还有记录值，记录的大小往往比主键要大的多，这样就会导致聚簇索引在判定新记录携带的主键是否重复时，产生昂贵的I/O代价


## 7.4.哪些情况需要建索引
- 主键自动建立主键索引（唯一 + 非空）
- 频繁作为查询条件的字段应该创建索引
- 查询中与其他表关联的字段，外键关系建立索引
- 查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度（最好与`ORDER BY`指定的字段顺序一致）
- 查询中统计或者分组字段（`GROUP BY`也和索引有关）
- **在高并发下倾向创建组合索引**

## 7.5.哪些情况不要建索引
- 记录太少的表
- 经常增删改(`Insert`, `Delete`, `Update`)的表, 因为频繁更新的字段不适合创建索引（每次更新不仅需要更新记录，还需要更新索引）
- `WHERE`条件里**用不到的字段**不创建索引
- 数据重复且分布均匀的字段，应该只为最经常查询和最经常排序的数据建立索引
	- 假如一个表有10万行记录，有一个字段A只有True和False两种值，并且每个值的分布概率大约为50%，那么对A字段建索引一般不会提高数据库的查询速度
	- **索引的选择性**是指索引列中不同值的数目与表中记录数的比。如果一个表中有2000条记录，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99，**索引的选择性越接近于1，索引的效率就越高**


# 8.性能分析

## 8.1.MySQL访问优化器(MySQL Query Optimizer)

MySQL中有专门负责优化SELECT语句的优化器模块，其主要功能是通过计算分析系统中收集到的统计信息，为客户端请求的Query提供他认为最优的执行计划（问题是他认为最优的数据检索方式，不见得是BDA认为的最优的方式，这部分非常耗时）

当客户端向MySQL请求一条Query，命令解析器模块完成请求分类，区别出是SELECT并转发给MySQL访问优化器时，优化器首先会对整条Query进行优化，处理掉一些常量表达式的预算，直接换成常量值。并对Query中的查询条件进行简化和转换，如去掉一些无用或显而易见的条件，结构调整等。然后分析Query中的Hint信息(如果有)，看显示Hint信息是否可以完全确定该Query的执行计划，如果没有Hint或Hint信息还不足以完全确定执行计划，则会读取所有涉及对象的统计信息，根据Query进行写相应的计算分析，然后再得出最后的执行计划。

## 8.2.MySQL常见瓶颈

+ CPU饱和：一般会发生将数据装入内存或从从磁盘上读取数据的时候
+ 磁盘I/O瓶颈：发生在装入数据远大于内存容量的时候
+ 服务器硬件的性能瓶颈：使用top，free，iostat和vmstat来查看系统的性能状态

## 8.3.EXPLAIN简介
> EXPLAIN是什么？

MySQL的查询执行计划，使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理SQL语句的，便于分析查询语句或是表结构的性能瓶颈

> EXPLAIN如何使用？

语法：`EXPLAIN` + `SQL语句`
```
mysql> EXPLAIN SELECT * FROM pms_category \G;
*************************** 1. row ***************************
+----+-------------+--------------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | 	 table	  | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | pms_category |    NULL    | ALL  |      NULL     | NULL |   NULL  | NULL |   4  |   100 	|  NULL |
+----+-------------+--------------+------------+------+---------------+------+---------+------+------+----------+-------+
```

## 8.4.EXPLAIN结果字段详细解读

> `id`：表的读取和加载顺序

值有以下三种情况：
- `id`相同：执行顺序由上至下
- `id`不同：如果是子查询，id的序号会递增，**id值越大优先级越高，越先被执行**
- `id`相同不同，同时存在：**永远是id大的优先级最高，id相等的时候顺序执行**

> `select_type`：数据查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询

- `SIMPLE`：简单的`SELECT`查询，查询中不包含子查询或者`UNION`
- `PRIMARY`：查询中若包含任何复杂的子部分，最外层查询则被标记为`PRIMARY`
- `SUBQUERY`：在`SELECT`或者`WHERE`子句中包含了子查询
- `DERIVED`：在`FROM`子句中包含的子查询被标记为`DERIVED(衍生)`，MySQL会递归执行这些子查询，把结果放在临时表中
- `UNION`：如果第二个`SELECT`出现在`UNION`之后，则被标记为`UNION`；若`UNION`包含在`FROM`子句的子查询中，外层`SELECT`将被标记为`DERIVED`
- `UNION RESULT`：从`UNION`表获取结果的`SELECT`

> `type`：访问类型排列

**从最好到最差依次是：**`system`>`const`>`eq_ref`>`ref`>`range`>`index`>`ALL`，除了`ALL`没有用到索引，其他级别都用到了索引

一般来说，得保证查询至少达到`range`级别，最好达到`ref`

- `system`：表只有一行记录（等于系统表），这是`const`类型的特例，平时不会出现，这个也可以忽略不计
- `const`：表示通过**索引1次**就找到了，`const`用于比较`primary key`或者`unique`索引。因为只匹配一行数据，所以很快。如将主键置于`where`列表中，MySQL就能将该查询转化为一个常量
- `eq_ref`：唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常用于主键或唯一索引扫描。除 了 `system` 和` const` 类型之外, 这是最好的联接类型
- `ref`：非唯一性索引扫描，返回本表和关联表某个值匹配的所有行，可能查出来有多条记录
- `range`：只检索给定范围的行，一般就是在`WHERE`语句中出现了`BETWEEN`、`< >`、`in`等的查询。这种范围扫描索引比全表扫描要好，因为它只需要开始于索引树的某一点，而结束于另一点，不用扫描全部索引
- `index`：`Full Index Scan`，全索引扫描，`index`和`ALL`的区别为`index`类型只遍历索引树，**也就是说虽然`ALL`和`index`都是读全表，但是`index`是从索引中读的，`ALL`是从磁盘中读取的**
- `ALL`：`Full Table Scan`，没有用到索引，全表扫描

> possible_keys 和 key

`possible_keys`：显示可能应用在这张表中的索引，一个或者多个。查询涉及到的字段上若存在索引，则该索引将被列出，**但实际上不一定被查询使用**   
`key`：实际使用的索引，如果为`NULL`，则没有使用索引

+ 如果`possible_keys`不为`NULL`，`key`为`NULL`，称为索引失效
+ 如果`possible_keys`为`NULL`，`key`不为`NULL`，即该索引仅仅出现在`key`列表中，称为索引覆盖
+ 当`SELECT`查询的字段和所键`复合索引`的`个数`和`顺序`刚好吻合时，会出现索引覆盖，此时`type`为`index`
+ 当使用`SELECT *`，只要没有建立索引，都是使用全表扫描, `type`为`ALL`
	
> key_len

`key_len`：表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。`key_len`显示的值为索引字段的最大可能长度，并非实际使用长度，即`key_len`是根据表定义计算而得，不是通过表内检索出的。**在不损失精度的情况下，长度越短越好**

`key_len`计算规则：**https://blog.csdn.net/qq_34930488/article/details/102931490**

```
mysql> desc pms_category;
+---------------+------------+------+-----+---------+----------------+
| Field         | Type       | Null | Key | Default | Extra          |
+---------------+------------+------+-----+---------+----------------+
| cat_id        | bigint(20) | NO   | PRI | NULL    | auto_increment |
| name          | char(50)   | YES  |     | NULL    |                |
| parent_cid    | bigint(20) | YES  |     | NULL    |                |
| cat_level     | int(11)    | YES  |     | NULL    |                |
| show_status   | tinyint(4) | YES  |     | NULL    |                |
| sort          | int(11)    | YES  |     | NULL    |                |
| icon          | char(255)  | YES  |     | NULL    |                |
| product_unit  | char(50)   | YES  |     | NULL    |                |
| product_count | int(11)    | YES  |     | NULL    |                |
+---------------+------------+------+-----+---------+----------------+
9 rows in set (0.00 sec)

mysql> explain select cat_id from pms_category where cat_id between 10 and 20 \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: pms_category
   partitions: NULL
         type: range
possible_keys: PRIMARY
          key: PRIMARY  # 用到了主键索引，通过查看表结构知道，cat_id是bigint类型，占用8个字节
      key_len: 8        # 这里只用到了cat_id主键索引，所以长度就是8！
          ref: NULL
         rows: 11
     filtered: 100.00
        Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```

> ref

`ref`：显示索引的哪一列被使用了，如果可能的话，是一个常数`const`，指明哪些列或常量被用于查找索引列上的值

> rows

`rows`：根据表统计信息及索引选用情况，大致估算出找到所需的记录需要读取的行数，即每张表有多少行被优化器查询，行数越少越好

> Extra

`Extra`：包含不适合在其他列中显示但十分重要的额外信息

- `Using filesort`：说明MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。**MySQL中无法利用索引完成的排序操作称为"文件内排序"(要避免-九死一生)** 。解决方法：在`ORDER BY`后使用所有未使用的索引，并遵循索引创建时的顺序

```
# 排序没有使用索引
mysql> EXPLAIN SELECT name FROM pms_category WHERE name='Tangs' ORDER BY cat_level \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: pms_category
   partitions: NULL
         type: ref
possible_keys: idx_name_parentCid_catLevel
          key: idx_name_parentCid_catLevel
      key_len: 201
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using where; Using index; Using filesort
1 row in set, 1 warning (0.00 sec)

#~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
# 排序使用到了索引 (使用所有升序未使用的索引进行排序)
mysql> EXPLAIN SELECT name FROM pms_category WHERE name='Tangs' ORDER BY parent_cid,cat_level \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: pms_category
   partitions: NULL
         type: ref
possible_keys: idx_name_parentCid_catLevel 
          key: idx_name_parentCid_catLevel
      key_len: 201
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```

- `Using temporary`：使用了临时表保存中间结果，MySQL在対查询结果排序时使用了临时表。常见于排序`ORDER BY`和分组查询`GROUP BY`。**临时表対系统性能损耗很大(绝对要避免-十死无生)**. 解决方法：在`GROUP BY`后使用所有`WHERE`查询字段中对应的索引，并遵循索引创建时的顺序
```
# 分组没有使用索引
mysql> EXPLAIN SELECT cat_level FROM pms_category WHERE cat_level in ('High','Very High') GROUP BY parentCid \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: pms_category
   partitions: NULL
         type: range
possible_keys: idx_name_parentCid_catLevel
          key: idx_name_parentCid_catLevel
      key_len: 201
          ref: NULL
         rows: 569
     filtered: 100.00
        Extra: Using where; Using index; Using temporary; Using filesort
1 row in set, 1 warning (0.00 sec)

#~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
# 分组使用到了索引
mysql> EXPLAIN SELECT cat_level FROM pms_category WHERE cat_level in ('High','Very High') GROUP BY  pms_category, parentCid \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: pms_category
   partitions: NULL
         type: range
possible_keys: idx_name_parentCid_catLevel 
          key: idx_name_parentCid_catLevel
      key_len: 402
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using where; Using index for group-by
1 row in set, 1 warning (0.00 sec)
```

- `Using index`：表示相应的`SELECT`操作中使用了覆盖索引(Convering Index)，避免访问了表的数据行，效率不错！如果同时出现`Using where`，表示索引被用来执行索引键值的查找；如果没有同时出现`Using where`，表明索引用来读取数据而非执行查找动作

```
# 覆盖索引
# 就是select的数据列只用从索引中就能够取得，不必从数据表中读取，即MySQ可以利用索引返回SELECT列表中的字段，而不必根据索引再次读取数据文件，换句话说查询列要被所使用的索引覆盖
# 注意：如果要使用覆盖索引，一定不能写SELECT *，要写出具体的字段
mysql> EXPLAIN SELECT cat_id FROM pms_category \G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: pms_category
   partitions: NULL
         type: index
possible_keys: NULL       
          key: PRIMARY
      key_len: 8
          ref: NULL
         rows: 1425
     filtered: 100.00
        Extra: Using index   # SELECT的数据列只用从索引中就能够取得，不必执行查找动作从数据表中读取   
1 row in set, 1 warning (0.00 sec)
```

- `Using where`：表明使用了`WHERE`过滤
- `Using join buffer`：使用了连接缓存
- `impossible where`：`WHERE`子句的值总是false，不能用来获取任何元组
- `select tables optimized way`：在没有GROUP BY子句的情况下，基于索引优化MIN/MAX操作或者对于MyISAM存储引擎执行优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化
- `distinct`：优化distinct操作，在找到第一匹配的元组后即停止找同样的动作

```
mysql> EXPLAIN SELECT name FROM pms_category where name = 'zs' and name = 'ls'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: NULL
   partitions: NULL
         type: NULL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: NULL
     filtered: NULL
        Extra: Impossible WHERE   # 不可能字段同时查到两个名字
1 row in set, 1 warning (0.00 sec)
```

# 9.索引分析

## 9.1.单表索引分析

> 数据准备

```sql
DROP TABLE IF EXISTS `article`;

CREATE TABLE IF NOT EXISTS `article`(
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
`author_id` INT(10) UNSIGNED NOT NULL COMMENT '作者id',
`category_id` INT(10) UNSIGNED NOT NULL COMMENT '分类id',
`views` INT(10) UNSIGNED NOT NULL COMMENT '被查看的次数',
`comments` INT(10) UNSIGNED NOT NULL COMMENT '回帖的备注',
`title` VARCHAR(255) NOT NULL COMMENT '标题',
`content` VARCHAR(255) NOT NULL COMMENT '正文内容'
) COMMENT '文章';

INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES(1,1,1,1,'1','1');
INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES(2,2,2,2,'2','2');
INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES(3,3,3,3,'3','3');
INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES(1,1,3,3,'3','3');
INSERT INTO `article`(`author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES(1,1,4,4,'4','4');
```

> 案例：查询`category_id`为1且`comments`大于1的情况下，`views`最多的`article_id`。

1、编写SQL语句并查看SQL执行计划

```shell
# 1、sql语句
SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;

# 2、sql执行计划
mysql> EXPLAIN SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: article
   partitions: NULL
         type: ALL 
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 5
     filtered: 20.00
        Extra: Using where; Using filesort
1 row in set, 1 warning (0.00 sec)
```
结论：很显然，type为ALL(全表扫描), 即最坏的情况；Extra中出现了Using filesort(文件内排序), 也是最坏的情况。必须手动建立索引进行优化


2、创建索引`idx_article_ccv`

```sql
CREATE INDEX idx_article_ccv ON article(category_id,comments,views);
```

3、查看当前索引

![show index](https://img-blog.csdnimg.cn/20200803134154162.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

4、查看现在SQL语句的执行计划

![explain](https://img-blog.csdnimg.cn/20200803134549914.png)

我们发现，创建符合索引`idx_article_ccv`之后，虽然解决了全表扫描的问题，但是在`ORDER BY`排序的时候没有用到索引，并且依然出现`Using filesort`

5、我们试试把SQL修改为`SELECT id,author_id FROM article WHERE category_id = 1 AND comments = 1 ORDER BY views DESC LIMIT 1;`看看SQL的执行计划

![explain](https://img-blog.csdnimg.cn/20200803135228945.png)

结论：当`comments > 1`的时候`ORDER BY`排序`views`字段索引就用不上，但是当`comments = 1`的时候`ORDER BY`排序`views`字段索引就可以用上！！！这是因为按照Btree索引的工作原理，先排序`category_id`，如果遇到相同的`category_id`则再排序`comments`，如果遇到相同的`comments`,则再排序`views`。当`comments`字段在联合索引里处于中间位置时，因为`comments>1`条件是一个范围值，使得MySQL无法利用索引再对后面的`views`进行检索，即range类型查询字段后的索引失效。

6、我们现在知道**范围之后的索引会失效**，那么我们如果删除`comments`这个索引，创建`idx_article_cv`索引呢？ (大胆假设，小心求证)

```
/* 删除索引 idx_article_cv */
DROP INDEX idx_article_ccv ON article;
/* 创建索引 idx_article_cv */
CREATE INDEX idx_article_cv ON article(category_id,views);
```

7、查看当前的索引

![show index](https://img-blog.csdnimg.cn/20200803140542912.png)

8、当前索引是`idx_article_cv`，来看一下SQL执行计划

![explain](https://img-blog.csdnimg.cn/20200803140951803.png)

结论：可以看到type为ref，Extra中的Using filesort也消失了，结果非常理想

9、同样，如果调整检索时`comments`和`views`的顺序，上述问题也能够迎刃而解，因此索引需要在不断地验证，尝试
```
/* 创建索引 idx_article_cvc */
CREATE INDEX idx_article_cvc ON article(category_id,views,comments);
```

## 9.2.两表索引分析

> 数据准备

```
DROP TABLE IF EXISTS `class`;
DROP TABLE IF EXISTS `book`;

CREATE TABLE IF NOT EXISTS `class`(
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
`card` INT(10) UNSIGNED NOT NULL COMMENT '分类' 
) COMMENT '商品类别';

CREATE TABLE IF NOT EXISTS `book`(
`bookid` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
`card` INT(10) UNSIGNED NOT NULL COMMENT '分类'
) COMMENT '书籍';

各执行20次，在class表和book表中中插入20条数据
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
```

> 两表连接查询的SQL执行计划

1、不创建索引的情况下，SQL的执行计划

![explain](https://img-blog.csdnimg.cn/20200803143557187.png)

`book`和`class`两张表都是没有使用索引，全表扫描，那么如果进行优化，索引是创建在`book`表还是创建在`class`表呢？下面进行大胆的尝试！

2、左表(`book`表)创建索引

创建索引`idx_book_card`

```sql
/* 在book表创建索引 */
CREATE INDEX idx_book_card ON book(card);
```

在`book`表中有`idx_book_card`索引的情况下，查看SQL执行计划

![explain](https://img-blog.csdnimg.cn/20200803144429349.png)



3、删除`book`表的索引，右表(`class`表)创建索引

创建索引`idx_class_card`

```sql
/* 在class表创建索引 */
CREATE INDEX idx_class_card ON class(card);
```

在`class`表中有`idx_class_card`索引的情况下，查看SQL执行计划

![explain](https://img-blog.csdnimg.cn/20200803145030597.png)

由此可见，**左连接将索引创建在右表上更合适**，这是由左连接特性决定的，左连接条件用于确定如何从右边开始搜索行，左边一定都有，所以右边是我们的关键点，一定需要建立索引。同样，**右连接将索引创建在左表上更合适**

## 9.3.三张表索引分析

> 数据准备

```
DROP TABLE IF EXISTS `phone`;

CREATE TABLE IF NOT EXISTS `phone`(
`phone_id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT COMMENT '主键',
`card` INT(10) UNSIGNED NOT NULL COMMENT '分类' 
) COMMENT '手机';

INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
```

> 三表连接查询SQL优化

1、不加任何索引，查看SQL执行计划

![explain](https://img-blog.csdnimg.cn/20200803160631786.png)

2、根据两表查询优化的经验，左连接需要在右表上添加索引，所以分别尝试在`book`表和`phone`表上添加索引

```
/* 在book表创建索引 */
CREATE INDEX idx_book_card ON book(card);

/* 在phone表上创建索引 */
CREATE INDEX idx_phone_card ON phone(card);
```

3、再次查看SQL的执行计划

![explain](https://img-blog.csdnimg.cn/20200803161013880.png)

总结：Type都是ref且总rows优化很好，效果不错，因此索引最好建在经常需要查询的字段中

## 9.4.结论

`JOIN`语句的优化：

- 尽可能减少`JOIN`语句中的`NestedLoop`（嵌套循环）的总次数：**永远都是小的结果集驱动大的结果集**
- 优先优化`NestedLoop`的内层循环
- 保证`JOIN`语句中被驱动表上`JOIN`条件字段已经被索引
- 当无法保证被驱动表的`JOIN`条件字段被索引且内存资源充足的前提下，不要太吝惜`Join Buffer` 的设置

# 10.索引失效

> 数据准备

```sql
CREATE TABLE `staffs`(
`id` INT(10) PRIMARY KEY AUTO_INCREMENT,
`name` VARCHAR(24) NOT NULL DEFAULT '' COMMENT '姓名',
`age` INT(10) NOT NULL DEFAULT 0 COMMENT '年龄',
`pos` VARCHAR(20) NOT NULL DEFAULT '' COMMENT '职位',
`add_time` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间'
)COMMENT '员工记录表';

INSERT INTO `staffs`(`name`,`age`,`pos`) VALUES('Ringo', 18, 'manager');
INSERT INTO `staffs`(`name`,`age`,`pos`) VALUES('张三', 20, 'dev');
INSERT INTO `staffs`(`name`,`age`,`pos`) VALUES('李四', 21, 'dev');

/* 创建索引 */
CREATE INDEX idx_staffs_name_age_pos ON `staffs`(`name`,`age`,`pos`);
```

## 10.1.索引失效的情况

- 全值匹配我最爱
- 最佳左前缀法则
- 不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描
- 索引中范围条件右边的字段会全部失效
- 尽量使用覆盖索引（只访问索引的查询，索引列和查询列一致），减少`SELECT *`
- MySQL在使用`!=`或者`<>`的时候无法使用索引会导致全表扫描
- `is null`、`is not null`也无法使用索引
- `like`以通配符开头`%abc`索引失效会变成全表扫描
- 字符串不加单引号索引失效
- 少用`or`，用它来连接时会索引失效

## 10.2.最佳左前缀法则(最重要原则)

> 案例

```sql
/* 用到了idx_staffs_name_age_pos索引中的name字段 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo';

/* 用到了idx_staffs_name_age_pos索引中的name, age字段 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo' AND `age` = 18;

/* 用到了idx_staffs_name_age_pos索引中的name，age，pos字段 这是属于全值匹配的情况！！！*/
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo' AND `age` = 18 AND `pos` = 'manager';

/* 索引没用上，ALL全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `age` = 18 AND `pos` = 'manager';

/* 索引没用上，ALL全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `pos` = 'manager';

/* 用到了idx_staffs_name_age_pos索引中的name字段，pos字段索引失效 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo' AND `pos` = 'manager';
```

> 概念

最佳左前缀法则：如果索引是多字段的复合索引，要遵守最佳左前缀法则。指的是查询从索引的**最左前列开始并且不跳过索引中的字段**

**口诀：带头大哥不能死，中间兄弟不能断**

## 10.3.索引列上不计算

> 案例

```
# 现在要查询`name` = 'Ringo'的记录，下面有两种方式来查询！

# 1、直接使用 字段 = 值的方式来计算
mysql> SELECT * FROM `staffs` WHERE `name` = 'Ringo';
+----+-------+-----+---------+---------------------+
| id | name  | age | pos     | add_time            |
+----+-------+-----+---------+---------------------+
|  1 | Ringo |  18 | manager | 2020-08-03 08:30:39 |
+----+-------+-----+---------+---------------------+
1 row in set (0.00 sec)

# 2、使用MySQL内置的函数
mysql> SELECT * FROM `staffs` WHERE LEFT(`name`, 5) = 'Ringo';
+----+-------+-----+---------+---------------------+
| id | name  | age | pos     | add_time            |
+----+-------+-----+---------+---------------------+
|  1 | Ringo |  18 | manager | 2020-08-03 08:30:39 |
+----+-------+-----+---------+---------------------+
1 row in set (0.00 sec)
```

我们发现以上两条SQL的执行结果都是一样的，但是执行效率有没有差距呢？我们通过分析两条SQL的执行计划来分析性能：

![explain](https://img-blog.csdnimg.cn/20200803171857325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

结论：不要在索引列上做任何操作(计算、函数、(自动或手动)类型转换)，会导致索引失效而转向全表扫描

**口诀：索引列上少/不计算**

## 10.4.范围之后全失效

> 案例

```
/* 用到了idx_staffs_name_age_pos索引中的name，age，pos字段 这是属于全值匹配的情况！！！*/
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo' AND `age` = 18 AND `pos` = 'manager';


/* 用到了idx_staffs_name_age_pos索引中的name，age字段，pos字段索引失效 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = '张三' AND `age` > 18 AND `pos` = 'dev';
```

查看上述SQL的执行计划

![explain](https://img-blog.csdnimg.cn/20200803173357787.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

结论：查询范围的字段使用到了索引，但是范围之后的索引字段会失效，存储引擎不能使用索引中范围条件右边的列

**口诀：范围之后全失效**

## 10.5.覆盖索引尽量用

```sql
/* 没有用到覆盖索引 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo' AND `age` = 18 AND `pos` = 'manager';

/* 用到了覆盖索引 */
EXPLAIN SELECT `name`, `age`, `pos` FROM `staffs` WHERE `name` = 'Ringo' AND `age` = 18 AND `pos` = 'manager';
```

![使用覆盖索引](https://img-blog.csdnimg.cn/20200803213031893.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

结论：尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少`SELECT *`，用什么字段就查询什么字段

**口诀：查询一定不用`*`**

## 10.6.不等有时会失效

```
/* 会使用到覆盖索引 */
EXPLAIN SELECT `name`, `age`, `pos` FROM `staffs` WHERE `name` != 'Ringo';

/* 索引失效 全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` != 'Ringo';

/* 索引失效 全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` <> 'Ringo';
```

结论：MySQL使用不等于(!= 或者 <>)的时候，无法使用索引会导致全表扫描

## 10.7.like百分加右边

```
/* 索引失效 全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` LIKE '%ing%';

/* 索引失效 全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` LIKE '%ing';

/* 使用索引范围查询 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` LIKE 'Ring%';
```

结论：like以通配符开头('%abc...'), MySQL索引失效会变成全表扫描

**口诀：`like`百分加右边**

如果一定要使用`%字符串%`，而且还要保证索引不失效，那么使用**覆盖索引**来编写SQL：

```
/* 使用到了覆盖索引 */
EXPLAIN SELECT `id` FROM `staffs` WHERE `name` LIKE '%in%';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `name` FROM `staffs` WHERE `name` LIKE '%in%';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `age` FROM `staffs` WHERE `name` LIKE '%in%';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `pos` FROM `staffs` WHERE `name` LIKE '%in%';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `id`, `name` FROM `staffs` WHERE `name` LIKE '%in%';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `id`, `age` FROM `staffs` WHERE `name` LIKE '%in%';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `id`,`name`, `age`, `pos` FROM `staffs` WHERE `name` LIKE '%in';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `id`, `name` FROM `staffs` WHERE `pos` LIKE '%na';

/* 索引失效 全表扫描 */
EXPLAIN SELECT `name`, `age`, `pos`, `add_time` FROM `staffs` WHERE `name` LIKE '%in';
```

![模糊查询百分号一定加前边](https://img-blog.csdnimg.cn/20200803220743206.png)

**口诀：覆盖索引保两边**


## 10.8.字符要加单引号（一定不要忘记，开发过程中会被骂死）

```
/* 使用到了覆盖索引 */
EXPLAIN SELECT `id`, `name` FROM `staffs` WHERE `name` = 'Ringo';

/* 使用到了覆盖索引 */
EXPLAIN SELECT `id`, `name` FROM `staffs` WHERE `name` = 2000;

/* 索引失效 全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 2000;
(这里name = 2000在MySQL中会发生强制类型转换，将数字转成字符串)
```

**口诀：字符要加单引号**

## 10.9.少用OR

```
/* 索引失效 全表扫描 */
EXPLAIN SELECT * FROM `staffs` WHERE `name` = 'Ringo' OR name='张三';
```

## 10.10.索引相关题目

**假设index(a,b,c)**

| Where语句                                               | 索引是否被使用                               |
| ------------------------------------------------------- | ------------------------------------------ |
| where a = 3                                             | Y，使用到a                                  |
| where a = 3 and b = 5                                   | Y，使用到a，b                               |
| where a = 3 and b = 5 and c = 4                         | Y，使用到a，b，c                            |
| where b = 3 或者 where b = 3 and c = 4 或者 where c = 4  | N，没有用到a字段                            |
| where a = 3 and c = 5                                   | Y，使用到a，但是没有用到c，因为b断了          |
| where a = 3 and b > 4 and c = 5                         | Y， 使用到a，b，但是没有用到c，因为c在范围之后 |
| where a = 3 and b like 'kk%' and c = 4                  | Y， 使用到a,b,c, 范围查询		         |
| where a = 3 and b like '%kk' and c = 4                  | Y，只用到a		           	    |
| where a = 3 and b like '%kk%' and c = 4                 | Y，只用到a		           	    |
| where a = 3 and b like 'k%kk%' and c = 4                | Y， 使用到a,b,c, 范围查询		         |


## 10.11.面试题分析

> 数据准备

```sql
/* 创建表 */
CREATE TABLE `test03`(
`id` INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
`c1` CHAR(10),
`c2` CHAR(10),
`c3` CHAR(10),
`c4` CHAR(10),
`c5` CHAR(10)
);

/* 插入数据 */
INSERT INTO `test03`(`c1`,`c2`,`c3`,`c4`,`c5`) VALUES('a1','a2','a3','a4','a5');
INSERT INTO `test03`(`c1`,`c2`,`c3`,`c4`,`c5`) VALUES('b1','b22','b3','b4','b5');
INSERT INTO `test03`(`c1`,`c2`,`c3`,`c4`,`c5`) VALUES('c1','c2','c3','c4','c5');
INSERT INTO `test03`(`c1`,`c2`,`c3`,`c4`,`c5`) VALUES('d1','d2','d3','d4','d5');
INSERT INTO `test03`(`c1`,`c2`,`c3`,`c4`,`c5`) VALUES('e1','e2','e3','e4','e5');

/* 创建复合索引 */
CREATE INDEX idx_test03_c1234 ON `test03`(`c1`,`c2`,`c3`,`c4`);
```

> 题目：创建了复合索引idx_test03_c1234，根据以下SQL分析索引的使用情况：

```
/* 最好索引怎么创建的，就怎么用，按照顺序使用，避免让MySQL自己再去翻译一次 */

/* 1.全值匹配, 用到索引c1 c2 c3 c4全字段 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c3` = 'a3' AND `c4` = 'a4';

/* 2.用到索引c1 c2 c3 c4全字段, MySQL的查询优化器会优化SQL语句的顺序*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` = 'a4' AND `c3` = 'a3';

/* 3.用到索引c1 c2 c3 c4全字段, MySQL的查询优化器会优化SQL语句的顺序*/
EXPLAIN SELECT * FROM `test03` WHERE `c4` = 'a4' AND `c3` = 'a3' AND `c2` = 'a2' AND `c1` = 'a1';

/* 4.用到索引c1 c2 c3字段，c4字段失效，范围之后全失效 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c3` > 'a3' AND `c4` = 'a4';

/* 5.用到索引c1 c2 c3 c4全字段, MySQL的查询优化器会优化SQL语句的顺序*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` > 'a4' AND `c3` = 'a3';

/* 6.用到了索引c1 c2 c3三个字段, c1和c2两个字段用于查找,  c3字段用于排序了但是没有统计到key_len中，c4字段失效*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c4` = 'a4' ORDER BY `c3`;

/* 7.用到了索引c1 c2 c3三个字段，c1和c2两个字段用于查找, c3字段用于排序了但是没有统计到key_len中*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' ORDER BY `c3`;

/* 8.用到了索引c1 c2两个字段，c4失效，c1和c2两个字段用于查找，c4字段排序产生了Using filesort说明排序没有用到c4字段*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' ORDER BY `c4`;

/* 9.用到了索引c1 c2 c3三个字段，c1用于查找，c2和c3用于排序*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c5` = 'a5' ORDER BY `c2`, `c3`;

/* 10.用到了c1一个字段，c1用于查找，c3和c2两个字段索引失效，产生了Using filesort */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c5` = 'a5' ORDER BY `c3`, `c2`;

/* 11.用到了c1 c2 c3三个字段，c1 c2用于查找，c2 c3用于排序 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND  `c2` = 'a2' ORDER BY c2, c3;

/* 12.用到了c1 c2 c3三个字段，c1 c2用于查找，c2 c3用于排序 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND  `c2` = 'a2' AND `c5` = 'a5' ORDER BY c2, c3;

/* 13.用到了c1 c2 c3三个字段，c1 c2用于查找，c2 c3用于排序 没有产生Using filesort, 因为之前c2这个字段已经确定了是'a2'了，这是一个常	量，再去ORDER BY c3,c2 这时候c2已经不用排序了！所以没有产生Using filesort 和(10)进行对比学习！
*/
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c2` = 'a2' AND `c5` = 'a5' ORDER BY c3, c2;

/* GROUP BY 表面上是叫做分组，但是分组之前必排序*/

/* 14.用到c1 c2 c3三个字段，c1用于查找，c2 c3用于排序，c4失效 */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c4` = 'a4' GROUP BY `c2`,`c3`;

/* 15.用到c1这一个字段，c4失效，c2和c3排序失效产生了Using filesort */
EXPLAIN SELECT * FROM `test03` WHERE `c1` = 'a1' AND `c4` = 'a4' GROUP BY `c3`,`c2`;
```

`GROUP BY`基本上都需要进行排序, 其索引优化几乎和`ORDER BY`一致，但是`GROUP BY`会有临时表的产生

## 10.12.总结

索引优化的一般性建议：

- 对于单值索引，尽量选择针对当前`query`过滤性更好的索引
- 在选择复合索引的时候，当前`query`中过滤性最好的字段在索引字段顺序中，位置越靠左越好
- 在选择复合索引的时候，尽量选择可以能够包含当前`query`中的`where`子句中更多字段的索引
- 尽可能通过分析统计信息和调整`query`的写法来达到选择合适索引的目的

口诀：

- 全值匹配我最爱
- 带头大哥不能死
- 中间兄弟不能断
- 索引列上不计算
- 范围之后全失效
- like百分加右边
- 覆盖索引保两边
- 覆盖索引尽量用
- 不等有时会失效
- 字符要加单引号
- 一般SQL少用OR

# 11.分析慢SQL的步骤

分析：

1、观察，至少跑1天，看看生产的慢SQL情况   
2、开启慢查询日志，设置阈值，比如超过5秒钟的就是慢SQL，并将它抓取出来   
3、EXPLAIN + 慢SQL分析   
4、SHOW PROFILE   
5、运维经理或DBA，进行MySQL数据库服务器的参数调优

总结（大纲）：

1、慢查询的开启并捕获   
2、EXPLAIN + 慢SQL分析   
3、SHOW PROFILE查询SQL在MySQL数据库中的执行细节和生命周期情况   
4、MySQL数据库服务器的参数调优


# 12.查询优化

## 12.1.小表驱动大表

> 优化原则：对于MySQL数据库而言，永远都是小表驱动大表

```java
/**
* 举个例子：可以使用嵌套的for循环来理解小表驱动大表
* 以下两个循环结果都是一样的，但是对于MySQL来说不一样
* 第一种可以理解为，和MySQL建立5次连接每次查询1000次
* 第一种可以理解为，和MySQL建立1000次连接每次查询5次
*/
for(int i = 1; i <= 5; i ++){
    for(int j = 1; j <= 1000; j++){
        
    }
}
// ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
for(int i = 1; i <= 1000; i ++){
    for(int j = 1; j <= 5; j++){
        
    }
}
```

> IN和EXISTS

```
/* 优化原则：小表驱动大表，即小的数据集驱动大的数据集 */

/* IN适合B表比A表数据小的情况*/
SELECT * FROM `A` WHERE `id` IN (SELECT `id` FROM `B`)

/* EXISTS适合B表比A表数据大的情况 */
SELECT * FROM `A` WHERE EXISTS (SELECT 1 FROM `B` WHERE `B`.id = `A`.id);
```

**EXISTS：**

- 语法：`SELECT....FROM tab WHERE EXISTS(subquery);`
- 该语法可以理解为：将主查询的数据，放到子查询中做条件验证，根据验证结果（`True`或是`False`）来决定主查询的数据结果是否得以保留

**提示：**

- `EXISTS(subquery)`子查询只返回`True`或者`False`，因此子查询中的`SELECT *`可以是`SELECT 1 OR SELECT X`，它们并没有区别
- `EXISTS(subquery)`子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比，如果担心效率问题，可进行实际检验以确定是否有效率问题
- `EXISTS(subquery)`子查询往往也可以用条件表达式，其他子查询或者`JOIN`替代，何种最优需要具体问题具体分析

## 12.2.ORDER BY优化

> 数据准备

```sql
CREATE TABLE `talA`(
`age` INT,
`birth` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO `talA`(`age`) VALUES(18);
INSERT INTO `talA`(`age`) VALUES(19);
INSERT INTO `talA`(`age`) VALUES(20);
INSERT INTO `talA`(`age`) VALUES(21);
INSERT INTO `talA`(`age`) VALUES(22);
INSERT INTO `talA`(`age`) VALUES(23);
INSERT INTO `talA`(`age`) VALUES(24);
INSERT INTO `talA`(`age`) VALUES(25);

/* 创建索引 */
CREATE INDEX idx_talA_age_birth ON `talA`(`age`, `birth`);
```

> 案例

```sql
/* 1.使用索引进行排序了 不会产生Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `age`;

/* 2.使用索引进行排序了 不会产生Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `age`,`birth`;

/* 3.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `birth`;

/* 4.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `age` > 20 ORDER BY `birth`,`age`;

/* 5.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` ORDER BY `birth`;

/* 6.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `birth` > '2020-08-04 07:42:21' ORDER BY `birth`;

/* 7.使用索引进行排序了 不会产生Using filesort */
EXPLAIN SELECT * FROM `talA` WHERE `birth` > '2020-08-04 07:42:21' ORDER BY `age`;

/* 8.没有使用索引进行排序 产生了Using filesort */
EXPLAIN SELECT * FROM `talA` ORDER BY `age` ASC, `birth` DESC;
```

`ORDER BY`子句，尽量使用索引排序，避免使用`Using filesort`排序

MySQL支持两种方式的排序，`FileSort`和`Index`，`Index`的效率高，它指MySQL扫描索引本身完成排序，`FileSort`方式效率较低

`ORDER BY`满足以下两中情况，会使用`Index`方式排序：

- `ORDER BY`语句使用索引最左前列
- 使用`WHERE`子句与`ORDER BY`子句条件列组合满足索引最左前列

**结论：尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀原则**

> 如果不在索引列上，File Sort有两种算法，包括双路排序算法和单路排序算法：

1、双路排序算法：MySQL4.1之前使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和`ORDER BY`列，対他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出。**一句话，从磁盘取排序字段，在`buffer`中进行排序，再从磁盘取其他字段**

取一批数据，要对磁盘进行两次扫描，众所周知，IO是很耗时的，所以在MySQL4.1之后，出现了改进的算法，就是单路排序算法

2、单路排序算法：从磁盘读取查询需要的所有列，按照`ORDER BY`列在`buffer`対它们进行排序，然后将扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据，并且把随机IO变成了顺序IO，但是它会使用更多的空间，因为它把每一行都保存在内存中了

由于单路排序算法是后出的，总体而言效率好过双路排序算法，但是单路排序算法有问题：如果`SortBuffer`缓冲区太小，导致从磁盘中读取所有的列不能完全保存在`SortBuffer`缓冲区中进行排序，此时会创建tmp文件并进行多路合并，排序完再取`SortBuffer`容量大小再排，这时候单路复用算法就会变成多路，反而性能不如双路复用算法，得不偿失

**单路复用算法的优化策略：**

- 增大`sort_buffer_size`参数的设置
- 增大`max_length_for_sort_data`参数的设置

**提高ORDER BY排序的速度：**

- `ORDER BY`时使用`SELECT *`是大忌，查什么字段就写什么字段，这点非常重要。在这里的影响是：
  - 当查询的字段大小总和小于`max_length_for_sort_data`而且排序字段不是`TEXT|BLOB`类型时，会使用单路排序算法，否则使用多路排序算法
  - 两种排序算法的数据都有可能超出`sort_buffer`缓冲区的容量，超出之后，会创建`tmp`临时文件进行合并排序，导致多次IO，但是单路排序算法的风险会更大一些，所以要增大`sort_buffer_size`参数的设置
- 尝试提高`sort_buffer_size`：不管使用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高，因为这个参数是针对每个进程的
- 尝试提高`max_length_for_sort_data`：提高这个参数，会增加用单路排序算法的概率。但是如果设置的太高，数据总容量`sort_buffer_size`的概率就增大，明显症状是高的磁盘IO活动和低的处理器使用率

## 12.3.GORUP BY优化

- `GROUP BY`实质是先排序后进行分组，遵照索引建的最佳左前缀
- 当无法使用索引列时，会使用`Using filesort`进行排序，增大`max_length_for_sort_data`参数的设置和增大`sort_buffer_size`参数的设置，会提高性能
- `WHERE`执行顺序高于`HAVING`，能写在`WHERE`限定条件里的就不要写在`HAVING`中了

## 12.4.总结

**为排序使用索引**

- MySQL两种排序方式：`Using filesort`文件内排序和`Index`扫描有序索引排序
- MySQL能为排序与查询使用相同的索引，创建的索引既可以用于排序，也可以用于查询

```
/* 创建a b c三个字段的索引 */
idx_table_a_b_c(a, b, c)

/* 1.ORDER BY 能使用索引最左前缀 */
ORDER BY a;
ORDER BY a, b;
ORDER BY a, b, c;
ORDER BY a DESC, b DESC, c DESC;

/* 2.如果WHERE子句中使用索引的最左前缀定义为常量，则ORDER BY能使用索引 */
WHERE a = 'Ringo' ORDER BY b, c;
WHERE a = 'Ringo' AND b = 'Tangs' ORDER BY c;
WHERE a = 'Ringo' AND b > 2000 ORDER BY b, c;

/* 3.不能使用索引进行排序 */
ORDER BY a ASC, b DESC, c DESC;  /* 排序不一致 */
WHERE g = const ORDER BY b, c;   /* 丢失a字段索引 */
WHERE a = const ORDER BY c;      /* 丢失b字段索引 */
WHERE a = const ORDER BY a, d;   /* d字段不是索引的一部分 */
WHERE a IN (...) ORDER BY b, c;  /* 对于排序来说，多个相等条件(a=1 or a=2)也是范围查询 */
```

# 13.慢查询日志

## 13.1.基本介绍

> 慢查询日志是什么？

- MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阈值的语句，具体指运行时间超过`long_query_time`值的SQL，则会被记录到慢查询日志中，`long_query_time`的默认值为10，意思是运行10秒以上的语句
- 由慢查询日志来查看哪些SQL超出了我们的最大忍耐时间值，比如一条SQL执行超过5秒钟，我们就认为是慢SQL，希望能收集超过5秒钟的SQL，结合之前`explain`进行全面分析

> 特别说明

**默认情况下，MySQL数据库没有开启慢查询日志**，需要我们手动来设置这个参数。

**当然，如果不是调优需要的话，一般不建议启动该参数**，因为开启慢查询日志会或多或少带来一定的性能影响，慢查询日志支持将日志记录写入文件


> 查看慢查询日志是否开以及如何开启

- 查看慢查询日志是否开启：`SHOW VARIABLES LIKE '%slow_query_log%';`

- 开启慢查询日志：`SET GLOBAL slow_query_log = 1;`，**使用该方法开启MySQL的慢查询日志只对当前数据库生效，如果MySQL重启后会失效**

```
# 1、查看慢查询日志是否开启
mysql> SHOW VARIABLES LIKE '%slow_query_log%';
+---------------------+--------------------------------------+
| Variable_name       | Value                                |
+---------------------+--------------------------------------+
| slow_query_log      | OFF                                  |
| slow_query_log_file | /var/lib/mysql/1dcb5644392c-slow.log |
+---------------------+--------------------------------------+
2 rows in set (0.01 sec)

# 2、开启慢查询日志
mysql> SET GLOBAL slow_query_log = 1;
Query OK, 0 rows affected (0.00 sec)
```

- 如果要使慢查询日志永久开启，需要修改`my.cnf`配置文件(其他系统变量也是如此)，在`[mysqld]`下增加或修改参数后，重启MySQL服务器：

```
# my.cnf
[mysqld]
# 1.这个是开启慢查询，注意ON需要大写
slow_query_log=ON 

# 2.这个是存储慢查询的日志文件，这个文件不存在的话，需要自己创建
slow_query_log_file=/var/lib/mysql/slow.log
```

> 开启了慢查询日志后，什么样的SQL才会被记录到慢查询日志里面呢？

这个是由参数`long_query_time`控制的，默认情况下`long_query_time`的值为10秒，只有SQL的执行时间>10才会被记录，MySQL中查看`long_query_time`的时间如下：

```
# 查看long_query_time
mysql> SHOW VARIABLES LIKE 'long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)

# 修改long_query_time
mysql> SET GLOBAL long_query_time=1;

设置后为什么看不出变化？ 解决方法：
	需要重新连接或新开一个会话才能看到修改值，并再次使用SHOW VARIABLES LIKE 'long_query_time%';
	也可以使用SHOW GLOBAL VARIABLES LIKE 'long_query_time%';
	
# 如需永久修改`long_query_time`的时间，需要修改`my.cnf`配置文件
# my.cnf
[mysqld]
# 这个是设置慢查询的时间，我设置的为1秒
long_query_time=1
log_output = FILE
```

> 如何测试

```
# 模拟一个慢SQL
mysql> SELECT SLEEP(4);

# 在日志文件中查看
shell> cat /var/lib/mysql/slow.log
```

> 查询慢查询日志的总记录条数(记录数越多，性能越坏)

```
mysql> SHOW GLOBAL STATUS LIKE '%Slow_queries%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Slow_queries  | 1     |
+---------------+-------+
1 row in set (0.00 sec)
```

## 13.2.日志分析工具

日志分析工具`mysqldumpslow`：在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具`mysqldumpslow`

```
# 1、mysqldumpslow --help 来查看mysqldumpslow的帮助信息
root@1dcb5644392c:/usr/bin# mysqldumpslow --help
Usage: mysqldumpslow [ OPTS... ] [ LOGS... ]

Parse and summarize the MySQL slow query log. Options are

  --verbose    verbose
  --debug      debug
  --help       write this text to standard output

  -v           verbose
  -d           debug
  -s ORDER     what to sort by (al, at, ar, c, l, r, t), 'at' is default  # 按照何种方式排序
                al: average lock time # 平均锁定时间
                ar: average rows sent # 平均返回记录数
                at: average query time # 平均查询时间
                 c: count  # 访问次数
                 l: lock time  # 锁定时间
                 r: rows sent  # 返回记录
                 t: query time  # 查询时间 
  -r           reverse the sort order (largest last instead of first)
  -t NUM       just show the top n queries  # 返回前面多少条记录
  -a           don't abstract all numbers to N and strings to 'S'
  -n NUM       abstract numbers with at least n digits within names
  -g PATTERN   grep: only consider stmts that include this string # 后面搭配一个正则匹配模式，大小写不敏感
  -h HOSTNAME  hostname of db server for *-slow.log filename (can be wildcard),
               default is '*', i.e. match all
  -i NAME      name of server instance (if using mysql.server startup script)
  -l           don't subtract lock time from total time
  
# 2、 案例
# 2.1 得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/slow.log
 
# 2.2 得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/slow.log
 
# 2.3 得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/slow.log

# 2.4 另外建议使用这些命令时结合|和more使用，否则出现爆屏的情况
mysqldumpslow -s r -t 10 /var/lib/mysql/slow.log | more
```

# 14.批量插入数据脚本

## 14.1.环境准备

> 1、建表SQL

```mysql
/* 1.dept表 */
CREATE TABLE `dept` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `deptno` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '部门id',
  `dname` varchar(20) NOT NULL DEFAULT '' COMMENT '部门名字',
  `loc` varchar(13) NOT NULL DEFAULT '' COMMENT '部门地址',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='部门表';

/* 2.emp表 */
CREATE TABLE `emp` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `empno` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '员工编号',
  `ename` varchar(20) NOT NULL DEFAULT '' COMMENT '员工名字',
  `job` varchar(9) NOT NULL DEFAULT '' COMMENT '职位',
  `mgr` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '上级编号',
  `hiredata` date NOT NULL COMMENT '入职时间',
  `sal` decimal(7,2) NOT NULL COMMENT '薪水',
  `comm` decimal(7,2) NOT NULL COMMENT '分红',
  `deptno` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '部门id',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='员工表';
```

> 2、由于开启过慢查询日志，开启了`bin-log`，我们就必须为`function`指定一个参数，否则使用函数会报错'This function has none of DETERMINISTIC......'

```
# 在mysql中设置 
# log_bin_trust_function_creators 默认是关闭的 需要手动开启
mysql> SHOW VARIABLES LIKE 'log_bin_trust_function_creators';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin_trust_function_creators | OFF   |
+---------------------------------+-------+
1 row in set (0.00 sec)

mysql> SET GLOBAL log_bin_trust_function_creators=1;
Query OK, 0 rows affected (0.00 sec)
```

上述修改方式MySQL重启后会失败，在`my.cnf`配置文件下修改永久有效

```shell
[mysqld]
log_bin_trust_function_creators=ON
```

## 14.2.创建函数

```mysql
# 1、函数：随机产生字符串,保证每条数据都不同
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
    DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwsyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
    DECLARE return_str VARCHAR(255) DEFAULT '';
    DECLARE i INT DEFAULT 0;
    WHILE i < n DO
    SET return_str = CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
    SET i = i + 1;
    END WHILE;
    RETURN return_str;
END $$

# 2、函数：随机产生部门编号
DELIMITER $$
CREATE FUNCTION rand_num() RETURNS INT(5)
BEGIN
    DECLARE i INT DEFAULT 0;
    SET i = FLOOR(100 + RAND() * 10);
    RETURN i;
END $$
```

## 14.3.创建存储过程

```mysql
# 1、函数：向dept表批量插入
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
    SET autocommit = 0; # 将自动提交设置为0，重要！否则每执行一条SQL就自动提交一次
    REPEAT
    SET i = i + 1;
    INSERT INTO dept(deptno,dname,loc) VALUES((START + i),rand_string(10),rand_string(8));
    UNTIL i = max_num
    END REPEAT;
    COMMIT; # 一次性提交
END $$

# 2、函数：向emp表批量插入
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
    SET autocommit = 0; # 将自动提交设置为0，重要！否则每执行一条SQL就自动提交一次
    REPEAT
    SET i = i + 1;
    INSERT INTO emp(empno,ename,job,mgr,hiredata,sal,comm,deptno) VALUES((START + i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
    UNTIL i = max_num
    END REPEAT;
    COMMIT;  # 一次性提交
END $$
```

## 14.4.调用存储过程

```mysql
# 1、调用存储过程向dept表插入10个部门
DELIMITER ;
CALL insert_dept(100,10);

# 2、调用存储过程向emp表插入50万条数据
DELIMITER ;
CALL insert_emp(100001,500000);
```

# 15.Show Profile

> Show Profile是什么？

`Show Profile`：MySQL提供可以用来分析当前会话中语句执行的资源消耗情况，可以用于SQL的细粒度调优的测量。**默认情况下，参数处于关闭状态，并保存最近15次的运行结果**

> 分析步骤

1、是否支持，看看当前的MySQL版本是否支持

```
# 查看Show Profile功能是否开启
mysql> SHOW VARIABLES LIKE 'profiling%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```

2、开启`Show Profile`功能，默认是关闭的，使用前需要开启

```
# 开启Show Profile功能
mysql> SET profiling=ON;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

3、运行SQL

```
SELECT * FROM `emp` GROUP BY `id`%10 LIMIT 150000;

SELECT * FROM `emp` GROUP BY `id`%20 ORDER BY 5;
```

4、查看结果，执行`SHOW PROFILES;`

```
mysql> SHOW PROFILES;
+----------+---------------------+---------------------------------------------------+
| Query_ID | Duration(持续时间)   | Query(查询语句)                                             |
+----------+---------------------+---------------------------------------------------+
|        1 | 	 0.00156100 	 | SHOW VARIABLES LIKE 'profiling'                   |
|        2 | 	 0.56296725      | SELECT * FROM `emp` GROUP BY `id`%10 LIMIT 150000 |
|        3 | 	 0.52105825      | SELECT * FROM `emp` GROUP BY `id`%10 LIMIT 150000 |
|        4 | 	 0.51279775      | SELECT * FROM `emp` GROUP BY `id`%20 ORDER BY 5   |
+----------+------------+---------------------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```

5、诊断SQL，`SHOW PROFILE cpu,block io FOR QUERY Query_ID;`

```
# 这里的3是第四步中的Query_ID。
# 可以在SHOW PROFILE中看到一条SQL中完整的生命周期
mysql> SHOW PROFILE CPU,BLOCK IO FOR QUERY 3;
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
| starting             | 0.000097 | 0.000090 |   0.000002 |            0 |             0 |
| checking permissions | 0.000010 | 0.000009 |   0.000000 |            0 |             0 |
| Opening tables       | 0.000039 | 0.000058 |   0.000000 |            0 |             0 |
| init                 | 0.000046 | 0.000046 |   0.000000 |            0 |             0 |
| System lock          | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |
| optimizing           | 0.000005 | 0.000000 |   0.000000 |            0 |             0 |
| statistics           | 0.000023 | 0.000037 |   0.000000 |            0 |             0 |
| preparing            | 0.000014 | 0.000000 |   0.000000 |            0 |             0 |
| Creating tmp table   | 0.000041 | 0.000053 |   0.000000 |            0 |             0 |
| Sorting result       | 0.000005 | 0.000000 |   0.000000 |            0 |             0 |
| executing            | 0.000003 | 0.000000 |   0.000000 |            0 |             0 |
| Sending data         | 0.520620 | 0.516267 |   0.000000 |            0 |             0 |
| Creating sort index  | 0.000060 | 0.000051 |   0.000000 |            0 |             0 |
| end                  | 0.000006 | 0.000000 |   0.000000 |            0 |             0 |
| query end            | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |
| removing tmp table   | 0.000006 | 0.000000 |   0.000000 |            0 |             0 |
| query end            | 0.000004 | 0.000000 |   0.000000 |            0 |             0 |
| closing tables       | 0.000009 | 0.000000 |   0.000000 |            0 |             0 |
| freeing items        | 0.000032 | 0.000064 |   0.000000 |            0 |             0 |
| cleaning up          | 0.000019 | 0.000000 |   0.000000 |            0 |             0 |
+----------------------+----------+----------+------------+--------------+---------------+
20 rows in set, 1 warning (0.00 sec)
```

`Show Profile`查询参数备注：

- `ALL`：显示所有的开销信息
- `BLOCK IO`：显示块IO相关开销（通用）
- `CONTEXT SWITCHES`：上下文切换相关开销
- `CPU`：显示CPU相关开销信息（通用）
- `IPC`：显示发送和接收相关开销信息
- `MEMORY`：显示内存相关开销信息
- `PAGE FAULTS`：显示页面错误相关开销信息
- `SOURCE`：显示和Source_function
- `SWAPS`：显示交换次数相关开销的信息

6、`Show Profile`查询列表，日常开发需要注意的结论：

- `converting HEAP to MyISAM`：查询结果太大，内存都不够用了，往磁盘上搬了
- `Creating tmp table`：创建临时表-拷贝数据到临时表-用完再删除，非常耗费数据库性能
- `Copying to tmp table`：把内存中的临时表复制到磁盘，非常危险！！！
- `locked`：死锁

> 全局查询日志(只能用在测试环境，永远不要用在生产环境, 否则人就没了, 一般不要使用，优先选择`Show Profile`)

```
# 查看是否开启全局日志查询
mysql> SHOW VARIABLES LIKE 'general_log%';
+------------------+---------------------+
| Variable_name    | Value               |
+------------------+---------------------+
| general_log      | OFF                 |
| general_log_file | WIN-GVOTK8E4SKI.log |
+------------------+---------------------+
2 rows in set, 1 warning (0.00 sec)

# 开启全局日志查询
mysql> SET GLOBAL general_log = 1;
Query OK, 0 rows affected (0.00 sec)

# 设置输出形式, 此后编写的SQL语句将会记录到MySQL库里的general_log表中
mysql> SET GLOBAL log_output='TABLE';
Query OK, 0 rows affected (0.00 sec)

# 使用以下命令查看general_log表中的记录
mysql> SELECT * FROM mysql.general_log;
+----------------------------+------------------+-----------+-----------+--------+-----------------+
|       event_time           |  user_host       | thread_id | server_id | command_type | argument|
+----------------------------+------------------------------+----+---+--------+---------------------------------+
| 2021-03-21 10:23:38.500321 | root[root] @ localhost [::1] |  8 | 1 | Query  | SELECT * FROM dept              |
| 2021-03-21 10:23:43.567267 | root[root] @ localhost [::1] |  8 | 1 | Query  | SELECT * FROM mysql.general_log |
+----------------------------+------------------------------+----+---+--------+---------------------------------+
```

# 16.数据库事务理论

## 16.1.传统事务

> 事务(Transaction)及其ACID属性

- `原子性(Atomicity)`：指事务包含的所有操作要么全部成功(Commit)，要么全部失败回滚(Rollback), 因此事务的操作如果成功就必须要完全应用到数据库，如果操作失败则不能对数据库有任何影响
- `一致性(Consistency)`：一个事务执行之前和执行之后都必须处于一致性状态。事务结束时，所有的内部数据结构（如 B+树索引或双向链表）都必须是正确的
- `隔离性(Isolation)`：指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰
- `持久性（Durability)`： 指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作

> 事务的实现原理

- 事务日志(Transaction Log)
	+ 目的：为了实现确保事务能在执行的任意过程中回滚（**原子性**）并且提交的事务会永久保存在数据库中(**持久性**)，我们使用事务日志来存储事务执行过程中的数据库的变动，每一条事务日志中都包含事务的 ID、当前被修改的元素、变动前以及变动后的值
	+ 分类：一种是回滚日志（Undo Log），另一种是重做日志（Redo Log），其中前者保证事务的原子性，后者保证事务的持久性
	+ 补充：当一个事务尝试对数据库进行修改时，会先生成一条日志并刷新到磁盘上，写日志的操作由于是追加的所以非常快，在这之后才会向数据库中写入或者更新对应的记录

- 并发控制(Concurrency Control)
	+ 目的：为了避免并发带来的一致性问题、满足数据库对于隔离性要求，数据系统往往都会使用并发控制尽可能充分利用机器的效率，实现一致性、隔离性与性能之间的权衡
	+ 常见的并发控制机制包括：锁、时间戳、MVCC
	+ 锁：使用在更新资源之前，以对资源进行锁定的方式保证多个数据库的会话同时修改某一行记录时不会出现脱离预期的行为
	+ 时间戳：在每次提交时对资源是否被改变进行检查


> 并发事务处理会带来的问题

- `更新丢失(Lost Update)`：两个事务同时更新一条数据，使得一个事务的更新数据被另一个事务的更新数据所覆盖
- `脏读(Dirty Reads)`：一个事务读到了另一个未提交事务修改过的数据
- `不可重复读(Non-Repeatable Reads)`: 一个事务只能读到另一个已经提交的事务修改过的数据，并且其他事务每对该数据进行一次修改并提交后，该事务都能查询到最新值   
	脏读和不可重复读的区别：脏读是某一个事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据
- `幻读(Phantom Reads)`:一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来

> 四大事务隔离级别

- 作用：定义事务处理数据读写操作的隔离程度，使事务之间互相隔离，互不影响，保证事务并发操作数据的正确性和一致性
- `读未提交(Read Uncommitted)`: 在该事务隔离级别下，查询语句在无锁的情况下运行，事务B可以读取到事务A修改过但未提交的数据，所有事务都可以看到其他未提交事务的执行结果，可能发生脏读，不可重复度和幻读问题，一般很少使用
- `读已提交(Read Committed)`: 在该事务隔离级别下，事务B只能在事务A修改过并且已提交后才能读取到事务A修改的数据，解决了脏读的问题，但可能发生不可重复读和幻读，一般很少使用
- `可重复读(Repeatable Read)`: 在该事务隔离级别下，事务B只能在事务A修改过数据并提交后，自己也提交事务后，才能读取到事务A修改的数据。即一个事务内的两次无锁查询返回的数据都是一样的，但别的事务的新增数据也能读取到，解决了脏读和不可重复读的问题，但可能发生幻读
- `可序列化(Serializable)`：在该事务隔离级别下, 可串行化读会给每个查询数据行加上共享锁(读锁)，排他锁(写锁)，意味着所有的读操作之间不阻塞，但读操作会阻塞别的事务的写操作，写操作也阻塞别的事务的读操作和写操作，解决了脏读，不可重复读，幻读的问题，但可能导致大量的超时现象和锁竞争

| 读数据一致性及运训的并发副作用隔离级别  |     		读数据一致性            |     脏读     |     不可重复读     |      幻读   |
| ------------------------------------ | ------------------------------------- | ------------| ------------------ | ---------- |
|       读未提交(Read Uncommitted)      | 最低级别，只能保证不读取物理上损坏的数据 |      是      |         是         |     是     |
|       读已提交(Read Committed)        | 	             语句级		 |      否      |         是         |     是     |
|       可重复读(Repeatable Read)      | 		     事务级 		 |      否      |         否         |     是     |
|       可序列化(Serializable)         | 	          最高级别，事务级           |      否      |         否         |     是 |

- 隔离级别和对性能影响的比较：可串行化>可重复读>读已提交>读未提交，事务隔离级别越高，所需要消耗的MySQL的性能越大(事务并发严重)，执行效率越低。为了平衡二者，一般建议设置的隔离级别为可重复读，MySQL默认的隔离级别为可重复读
- 事务的隔离级别设置一定要在开始事务之前，隔离级别的设置只对当前会话有效。对于使用MySQL命令窗口而言，一个窗口就相当于一个会话，当前窗口设置的隔离级别只对当前窗口中的事务有效
- 查看/设置当前会话的隔离级别：   
	+ `SHOW VARIABLES LIKE 'transaction_isolation';`   
	+ `SELECT @@[global.|session.]transaction_isolation;`   
	+ `SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL [REPEATABLE READ|READ UNCOMMITTED|READ COMMITTED|SERIALIZABLE];`
- 隔离级别的实现原理：每条数据在更新的时候都会同时记录一条在回滚操作日志(Undo log)中记录一条回滚操作，通过回滚(Rollback)可以回到前一个状态的值

> 相关问题
- 为什么上了写锁，别的事务还可以进行读操作？因为InnoDB有MVCC(多版本并发控制),可以使用快照读，而不会被阻塞

- 回滚操作日志什么时候删除？MYSQL会判断当没有事务需要用到这些回滚日志的时候，即当系统中有比这个回滚日志更早的read-view的时候，回滚日志就会被删除

- MySQL的事务隔离级别和Spring中的事务隔离级别有什么必然的联系呢？ Spring就是对数据库事务进行了封装，并提出了5种事务隔离级别(ISOLATION_DEFAULT/ISOLATION_READ_UNCOMMITTED/ISOLATION_READ_COMMITTED/ISOLATION_REPEATABLE_READ/ISOLATION_SERIALIZABLE)和7种事务传播机制，规定了事务方法之间发生嵌套调用时，事务该如何进行传播

>7种事务传播机制

- `PROPAGATION_REQUIRED`：如果当前事务方法有事务则加入事务，没有则创建一个事务
- `PROPAGATION_SUPPORTS`: 支持当前事务，如果当前没事务，也支持非事务状态运行
- `PROPAGATION_NOT_SUPPORTED`:不支持事务，以非事务方式执行操作。如果当前有事务，则挂起事务运行
- `PROPAGATION_MANDATORY`: 强制当前事务方法使用事务运行，如果当前没有事务则抛出异常
- `PROPAGATION_REQUIRES_NEW`:创建新事务，无论当前存不存在事务，都创建新事务
- `PROPAGATION_NEVER`: 以非事务状态运行，当前事务方法不能存在事务，如果存在事务则抛出异常
- `PROPAGATION_NESTED`：如果当前存在事务，则在嵌套事务内执行。嵌套事务的提交-回滚与父事务没有任何关系，反之，当父事务提交，嵌套事务也一起提交，父事务回滚，嵌套事务也会回滚。如果当前没有事务，则新建一个事务运行，即执行PROPAGATION_REQUIRED类似操作

## 16.2.分布式理论拓展

> CAP理论

CAP是一个已经经过证实的理论：一个分布式系统最多只能同时满足一致性(Consistency)、可用性(Availability)和分区容错性(Partition Tolerence)三项中的两项

- `一致性`：和事务的一致性不同，分布式环境中的一致性是指**数据在多个副本之间能够保持一致的特性**。在分布式系统中，数据一般会存在于不同节点的副本中，如果对第一个节点的数据成功进行了更新操作，而第二个节点上的数据却没有得到相应的更新，这时候读取第二个节点的数据依然是更新前的数据，即脏数据，此时就是分布式系统数据不一致的情况。在分布式系统中，如果能够做到针对一个数据项的更新操作执行成功后，所有用户都能读取到最新的值，那么就认为这样的系统具有**强一致性**(或严格一致性)
- `可用性`：指**系统提供的服务必须一直处于可用的状态**，对于用户的每一个请求操作总是能够**在有限的时间内返回结果**，如果超过了这个时间范围，则认为系统是不可用的
	- 有限的时间内：是在系统的运行指标，不同系统会有差别，比如搜索引擎通常在0.5秒内需要给用户检索结果
	- 返回结果：是可用性的一个重要指标，它要求系统完成对用户请求后，返回一个正常的响应结果，要明确地反应出对请求处理的成功或失败。如果返回的结果是显示操作失败，则认为此时系统是不可用的
- `分区容错性`：一个分布式系统中，节点组成的网络本来应该是连通的。然而可能因为某些故障，使得有些节点之间不连通了，整个网络就分成了几块区域，而数据就散布在了这些不连通的区域中，这就叫分区。当你一个数据项只在一个节点中保存，那么分区出现后，和这个节点不连通的部分就访问不到这个数据了。这时分区就是无法容忍的
	- 提高分区容忍性的办法就是一个数据项复制到多个节点上，那么出现分区之后，这一数据项仍然能在其他区中读取，容忍性就提高了。然而，把数据复制到多个节点，就会带来一致性的问题，就是多个节点上面的数据可能是不一致的。要保证一致，每次写操作就都要等待全部节点写成功，而这等待又会带来可用性的问题
	- 总的来说就是，数据存的节点越多，分区容忍性越高，但要复制更新的数据就越多，一致性就越难保证。为了保证一致性，更新所有节点数据所需要的时间就越长，可用性就会降低

对于**多数大型互联网应用的场景**，主机众多、部署分散，而且现在的集群规模越来越大，所以节点故障、网络故障是常态，而且要保证服务可用性达到N个9，即**保证可用性和分区容错性，舍弃一致性**


> BASE理论

BASE理论是对CAP理论的延伸，思想是即使无法做到强一致性（CAP的一致性就是强一致性），但可以适当地采取弱一致性，即最终一致性。BASE是指基本可用(Basically Available)、软状态(Soft State)、最终一致(Eventual Consistency)

- `基本可用`： 指**分布式系统在出现故障的时候，允许损失部分可用性（响应时间、部分功能）**，需要注意的是，基本可用绝对不等价于系统不可用
	- 响应时间上的损失：正常的情况下搜索引擎需要在0.5秒内给用户返回相应的查询结果，但由于出现故障（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加了1-2秒，可以接受
	- 功能上的损失：淘宝购物网站在双十一购物高峰时，为了保护系统的稳定性，部分消费者可能会被引导到一个降级页面
- `软状态`：指**系统存在中间状态, 而中间状态不会影响系统整体的可用性**，分布式存储中一般一份数据会有多个副本，允许不同副本同步的延时就是软状态的体现，MySQL Replication异步复制就是软状态的一种体现
- `最终一致性`：指**系统中的所有数据副本经过一定的时间后，最终达到一直的状态**，弱一致性和强一致性相反，最终一致性时弱一致性的一种特殊情况


## 16.3.高并发 

> 什么是高并发？
高并发（High Concurrency）是一种系统运行过程中遇到的一种“短时间内遇到大量操作请求”的情况，主要发生在web系统集中大量访问收到大量请求（例如：12306的抢票情况；天猫双十一活动），该情况的发生会导致系统在这段时间内执行大量操作，例如对资源的请求，数据库的操作等。。如果高并发处理不好，不仅仅降低了用户的体验度（请求响应时间过长），同时可能导致系统宕机，严重的甚至导致OOM异常，系统停止工作等

> 高并发的处理指标

+ 响应时间（Response Time）：系统对请求做出响应的时间
+ 吞吐量（Throughput）： 单位时间内处理的请求数量
+ 每秒查询率(Query Per Second): 每秒响应请求数
+ 并发用户数：同时承载正常使用系统功能的用户数量，。例如一个即时通讯系统，同时在线量一定程度上代表了系统的并发用户数

> 多线程处理高并发(Java)

+ 并发编程三要素
	+ 原子性：指的是一个或多个操作要么全部执行成功要么全部执行失败
	+ 有序性：程序执行的顺序按照代码的先后顺序执行（处理器可能会对指令进行重排序）
	+ 可见性：当多个线程访问同一个变量时，如果其中一个线程对其作了修改，其他线程能立即获取到最新的值

+ 线程的五大状态 [线程VS进程详细讲解-基于JVM](https://blog.csdn.net/ThinkWon/article/details/102021274#:~:text=%E8%BF%9B%E7%A8%8B%E4%B8%8E%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%8C%BA%E5%88%AB%E8%BF%9B%E7%A8%8B%E6%98%AF%E8%B5%84%E6%BA%90%E5%88%86%E9%85%8D%E6%9C%80%E5%B0%8F,%E5%A0%86%E6%A0%88%E6%AE%B5%E5%92%8C%E6%95%B0%E6%8D%AE%E6%AE%B5%EF%BC%8C&text=%E4%B8%80%E4%B8%AA%E8%BF%9B%E7%A8%8B%E5%8F%AF%E4%BB%A5%E6%8B%A5%E6%9C%89%E5%A4%9A,%E6%89%80%E5%B1%9E%E8%BF%9B%E7%A8%8B%E7%9A%84%E6%A0%88%E7%A9%BA%E9%97%B4%E3%80%82)

进程是操作系统分配资源的基本单位，线程是处理器任务调度和执行的基本单位。一个进程至少包含一个线程，可以包含多个线程。在同一个进程中进行线程切换不会引起进程切换，由于线程共享进程的资源和内存空间，切换速度快，消耗小。但在不同进程中进行线程切换时，会引起进程切换，改变进程的上下文，切换速度慢，消耗大

	+ 创建：当用new操作符创建一个线程的时候
	+ 就绪：调用start方法，处于就绪状态的线程并不一定马上就会执行run方法，还需要等待CPU的调度
	+ 运行：CPU开始调度线程，并开始执行run方法
	+ 阻塞：线程的执行过程中由于一些原因进入到阻塞状态， 如调用sleep方法, 尝试去得到一个锁等等
	+ 死亡：run方法正常执行完或者执行过程中遇到了一个异常

+ 线程的分类
	+ 用户级线程(User-Level Thread, ULT)   
	由应用程序所支持的线程实现，对内核不可见，存在于用户空间中。有关线程的控制(创建、撤销、同步与通信功能等)都是由应用程序通过使用**线程库**(库调度器)来完成，无法利用**系统调用**(操作系统调度器)来实现,如Unix操作系统
	![用户及线程实现方式](https://img-blog.csdn.net/20180320042241255?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMDc5MDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
	
	库调度器从进程中的多个用户线程中选取一个线程，然后该线程和该进程与一个内核线程关联起来，内核线程被操作系统调度器指派到处理器内核运行。用户级线程是一种”多对一“的线程映射
	
	+ 内核级线程(Kerbel-Level Thread, KLT):
	由内核所支持的线程实现，存在于内核空间当中。有关线程的控制都是在内核支持下，由操作系统负责管理，通过系统调用完成， 内核级线程是一种”一对一“的线程映射

	 + 用户级线程和内核级线程的区别
		 + 用户级线程是操作系统内核不可感知的，内核级线程是操作系统内核可感知的
		 + 用户级线程的创建，撤销和调度不需要操作系统内核的支持，是在编程语言这一级处理的；而内核级线程的创建，撤销和调度都需要操作系统内核的支持，而且进程的创建，撤销和调度大体相同
		 + 用户级线程执行系统调用指令时将导致其所属进程被中断(用户态切换到内核态)，而内核级线程执行系统调用指令时只导致该线程被中断
		 + 在只有用户级线程的系统内，CPU调度还是以进程为单位。处于运行状态的进程中的多个线程，由用户程序控制的线程轮换运行；在有内核支持线程的系统内，CPU调度则以线程为单位，有操作系统的线程调度程序负责线程的调度
		 + 用户级线程的程序实体是运行在用户态下的程序，而内核支持线程的程序实体则是可以运行在任何状态下的程序
	
	 + 用户级线程与内核级线程的组合实现
	 ![组合实现方式](https://img-blog.csdn.net/20180320045459981?watermark/2/text/Ly9ibG9nLmNzZG4ubmV0L3UwMTMwMDc5MDA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
	 
	 库调度器从进程中的多个用户线程中选取多个线程，然后该用户级线程集合和该进程与一个内核线程关联起来，内核线程被操作系统调度器指派到处理器内核运行

+ 悲观锁与乐观锁
	+ 悲观锁：每次操作都会加锁，会造成线程阻塞
	+ 乐观锁：每次操作不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止，不会造成线程阻塞

+ 线程之间的协作及synchronized关键字： wait、notify、notifyAll等

+ CAS(Compare And Swap): 即比较替换，是实现并发应用到的一种技术。操作包含三个操作数 —— 内存位置(V）、预期原值(A)和新值(B), 如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值, 否则，处理器不做任何操作

+ 线程池: 通过复用可以大大减少线程频繁创建与销毁带来的性能上的损耗


> 高并发计数方案：提升高并发能力

+ 分布式缓存：Redis、Memcached等，结合CDN来解决图片文件等访问
+ 消息队列中间件：ActiveMQ等，解决大量消息的异步处理能力
+ 应用拆分:一个工程被拆分为多个工程部署，利用Dubbo解决多工程之间的通信
+ 数据库垂直拆分和水平拆分、分库分表
+ 数据库主从复制读实现写分离，解决大数据的查询问题
+ 还可以利用Nosql ，例如MongoDB/PostreSQL配合MySQL组合使用
+ 还需要建立大数据访问情况下的服务降级以及限流机制等


## 16.4.MySQL分库分表(Sharding)

> 可能会面临的问题?
+ 用户请求量太大：单个服务器TPS，内存，I/O都是有限的 - 解决方法：分散请求到多个服务器上，堆机器就行
+ 单库太大：单个数据库处理能力有限；单库所在服务器上磁盘空间不足；单库上操作的IO瓶颈 - 解决方法：切分成更多更小的库
+ 单表太大：CRUD都成问题；索引膨胀，查询超时 - 解决方法：切分成多个数据集更小的表

> 方式方法
+ 垂直拆分
	+ 垂直分表：单张表的数据列太多
	+ 垂直分库：按业务需求进行拆分，切分后放在多个服务器上，在高并发场景下，垂直分库一定程度上能够突破IO、连接数及单机硬件资源的瓶颈
+ 水平拆分
	+ 水平分表：单张表的数据量太大，按照某种规则（RANGE、HASH取模、按地理区域、按时间进行”冷热数据分离“”)切分成多张表， 但是这些表还是在同一个库中，所以库级别的数据库操作还是有IO瓶颈，不建议采用
	+ 水平分库分表：将单张表的数据切分到多个服务器上去，每个服务器具有相应的库与表，只是表中数据集合不同。 水平分库分表能够有效的缓解单机和单库的性能瓶颈和压力，突破IO、连接数、硬件资源等的瓶颈

分库分表的顺序应该是**先垂直分，后水平分**，因为垂直分库更简单，更符合我们处理现实世界问题的方式(按业务)

> 分库分表后所面临的问题

+ 分库分表后，单机数据库上的传统事务就变成分布式是事务了，如果依赖数据库本身的分布式事务管理功能去执行事务，将付出高昂的性能代价； 如果由应用程序去协助控制，形成程序逻辑上的事务，又会造成编程方面的负担
+ 跨库Join: 分库分表后表之间的关联操作将受到限制，我们无法Join位于不同分库的表，也无法Join分表粒度不同的表， 结果原本一次查询能够完成的业务，可能需要多次查询才能完成。粗略的解决方法：
	+  全局表：基础数据，所有库都拷贝一份
	+  字段冗余：这样有些字段就不用join去查询了
	+  统层组装：分别查询出所有，然后组装起来，较复杂


> 分库分表中间件
+ 基于代理方式：MySQL Proxy和Amoeba
+ 基于Hibernate框架的：Hibernate Shards
+ 基于JDBC的：当当Sharding-JDBC
+ 基于Mybatis的类似Maven插件式：蘑菇街的蘑菇街TSharding
+ 通过重写Spring的Ibatis Template类的Cobar Client
+ 些大公司的开源产品

![开源产品](https://user-gold-cdn.xitu.io/2018/7/30/164e9fe9ff548c7e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


 **注意**：高并发 ≠ 多线程， 多线程只是处理高并发的一种编程方法

# 17.数据库锁理论

> 什么是锁？

锁是计算机协调多个进程或线程并发访问某一资源的机制，在数据库中，除传统的计算资源(如CPU, RAM，I/O等)的争用外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性，有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素，从这个角度来讲，锁对数据库而言显得尤为重要，也更加复杂

当你想在在淘宝买一样商品，此时商品只有一件库存，加入这个时候还有另一个人要买，那么如何解决是你买到还是另一个人买到的问题呢？这里肯定要用到事务，我们先从库存中取出商品的数量，然后插入订单，付款后插入付款表信息，然后更新商品数量。在这个过程中，锁可以对有限的资源进行保护，解决隔离和并发的矛盾

> 锁的分类

+ 按对操作数据的类型分为：读锁和写锁   
	读锁(共享锁)：针对同一份数据，读个读操作可以同时进行而不会互相影响
	
	写锁(排他锁)：当前写操作没有完成前，它会阻断其他锁和读锁

+ 按对数据操作的粒度分为：表锁、行锁、页锁   
	基于开销、加锁速度、死锁、粒度、并发性能等因素的考量，只能就具体应用的特点说明哪种锁更适合

# 16.1表锁(偏读)

**表锁特点：**

- 表锁偏向`MyISAM`存储引擎，开销小，加锁快，无死锁，锁定粒度大，发生锁冲突的概率最高，并发度最低

## 17.1.1环境准备

```mysql
# 1、创建表
CREATE TABLE `mylock`(
`id` INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
`name` VARCHAR(20)
)ENGINE=MYISAM DEFAULT CHARSET=utf8 COMMENT='测试表锁';

# 2、插入数据
INSERT INTO `mylock`(`name`) VALUES('ZhangSan');
INSERT INTO `mylock`(`name`) VALUES('LiSi');
INSERT INTO `mylock`(`name`) VALUES('WangWu');
INSERT INTO `mylock`(`name`) VALUES('ZhaoLiu');
```

## 17.1.2.锁表的命令

> 1、查看数据库表锁的命令

```
# 查看数据库表锁的命令
mysql> SHOW OPEN TABLES;
```

> 2、给`mylock`表上读锁，给`book`表上写锁

```
# 给mylock表上读锁，给book表上写锁
mysql> LOCK TABLE `mylock` READ, `book` WRITE;

# 查看当前表的状态
mysql> SHOW OPEN TABLES;
+--------------------+------------------------------------------------------+--------+-------------+
| Database           | Table                                                | In_use | Name_locked |
+--------------------+------------------------------------------------------+--------+-------------+
| sql_analysis       | book                                                 |      1 |           0 |
| sql_analysis       | mylock                                               |      1 |           0 |
+--------------------+------------------------------------------------------+--------+-------------+
```

> 3、释放表锁

```mysql
# 释放给表添加的锁
mysql> UNLOCK TABLES;

# 查看当前表的状态
mysql> SHOW OPEN TABLES;
+--------------------+------------------------------------------------------+--------+-------------+
| Database           | Table                                                | In_use | Name_locked |
+--------------------+------------------------------------------------------+--------+-------------+
| sql_analysis       | book                                                 |      0 |           0 |
| sql_analysis       | mylock                                               |      0 |           0 |
+--------------------+------------------------------------------------------+--------+-------------+
```

## 17.1.3.读锁案例

> 1、打开两个会话，`SESSION1`为`mylock`表添加读锁

```
# 为mylock表添加读锁
mysql> LOCK TABLE `mylock` READ;
```

> 2、打开两个会话，`SESSION1`是否可以读自己锁的表？是否可以修改自己锁的表？是否可以读其他的表？那么`SESSION2`呢？

```
# SESSION1

# 问题1：SESSION1为mylock表加了读锁，可以读mylock表！
mysql> SELECT * FROM `mylock`;
+----+----------+
| id | name     |
+----+----------+
|  1 | ZhangSan |
|  2 | LiSi     |
|  3 | WangWu   |
|  4 | ZhaoLiu  |
+----+----------+
4 rows in set (0.00 sec)

# 问题2：SESSION1为mylock表加了读锁，不可以修改mylock表！
mysql> UPDATE `mylock` SET `name` = 'abc' WHERE `id` = 1;
ERROR 1099 (HY000): Table 'mylock' was locked with a READ lock and can't be updated

# 问题3：SESSION1为mylock表加了读锁，不可以读其他未锁定的表！
mysql> SELECT * FROM `book`;
ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES


# SESSION2

# 问题1：SESSION1为mylock表加了读锁，SESSION2可以读mylock表！
mysql> SELECT * FROM `mylock`;
+----+----------+
| id | name     |
+----+----------+
|  1 | ZhangSan |
|  2 | LiSi     |
|  3 | WangWu   |
|  4 | ZhaoLiu  |
+----+----------+
4 rows in set (0.00 sec)

# 问题2：SESSION1为mylock表加了读锁，SESSION2修改mylock表会被阻塞，需要等待SESSION1释放mylock表！
mysql> UPDATE `mylock` SET `name` = 'abc' WHERE `id` = 1;
^C^C -- query aborted
ERROR 1317 (70100): Query execution was interrupted

# 问题3：SESSION1为mylock表加了读锁，SESSION2可以读其他未锁定表！
mysql> SELECT * FROM `book`;
+--------+------+
| bookid | card |
+--------+------+
|      1 |    1 |
|      7 |    4 |
|      8 |    4 |
|      9 |    5 |
|      5 |    6 |
|     17 |    6 |
|     15 |    8 |
+--------+------+
24 rows in set (0.00 sec)
```


## 17.1.4.写锁案例

> 1、打开两个会话，`SESSION1`为`mylock`表添加写锁

```mysql
# 为mylock表添加写锁
LOCK TABLE `mylock` WRITE;
```

> 2、打开两个会话，`SESSION1`是否可以读自己锁的表？是否可以修改自己锁的表？是否可以读其他的表？那么`SESSION2`呢？

```shell
# SESSION1

# 问题1：SESSION1为mylock表加了写锁，可以读mylock的表！
mysql> SELECT * FROM `mylock`;
+----+----------+
| id | name     |
+----+----------+
|  1 | ZhangSan |
|  2 | LiSi     |
|  3 | WangWu   |
|  4 | ZhaoLiu  |
+----+----------+
4 rows in set (0.00 sec)

# 问题2：SESSION1为mylock表加了写锁，可以修改mylock表!
mysql> UPDATE `mylock` SET `name` = 'abc' WHERE `id` = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# 问题3：SESSION1为mylock表加了写锁，不能读其他未锁定的表!
mysql> SELECT * FROM `book`;
ERROR 1100 (HY000): Table 'book' was not locked with LOCK TABLES

# SESSION2

# 问题1：SESSION1为mylock表加了写锁，SESSION2读mylock表会阻塞，等待SESSION1释放！
mysql> SELECT * FROM `mylock`;
^C^C -- query aborted
ERROR 1317 (70100): Query execution was interrupted

# 问题2：SESSION1为mylock表加了写锁，SESSION2更新mylock表会阻塞，等待SESSION1释放！
mysql> UPDATE `mylock` SET `name` = 'abc' WHERE `id` = 1;
^C^C -- query aborted
ERROR 1317 (70100): Query execution was interrupted

# 问题3：SESSION1为mylock表加了写锁，SESSION2可以读其他表！
mysql> SELECT * FROM `book`;
+--------+------+
| bookid | card |
+--------+------+
|      1 |    1 |
|      7 |    4 |
|      8 |    4 |
|      9 |    5 |
|      5 |    6 |
|     17 |    6 |
|     15 |    8 |
+--------+------+
24 rows in set (0.00 sec)
```

## 17.1.5.案例结论

**`MyISAM`引擎在执行查询语句`SELECT`之前，会自动给涉及到的所有表加读锁，在执行增删改之前，会自动给涉及的表加写锁**

MySQL的表级锁有两种模式：

- 表共享读锁（Table Read Lock）

- 表独占写锁（Table Write Lock）

対`MyISAM`表进行操作，会有以下情况：

- 対`MyISAM`表的读操作（加读锁），不会阻塞其他线程対同一表的读操作，但是会阻塞其他线程対同一表的写操作，只有当读锁释放之后，才会执行其他线程的写操作
- 対`MyISAM`表的写操作（加写锁），会阻塞其他线程対同一表的读和写操作，只有当写锁释放之后，才会执行其他线程的读写操作

简而言之就是：读锁会阻塞写，但不会阻塞读，而写锁会把读和写都阻塞

## 17.1.6.表锁分析

```
mysql> SHOW STATUS LIKE 'table%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Table_locks_immediate      | 173   |
| Table_locks_waited         | 0     |
| Table_open_cache_hits      | 5     |
| Table_open_cache_misses    | 8     |
| Table_open_cache_overflows | 0     |
+----------------------------+-------+
5 rows in set (0.00 sec)
```

可以通过`Table_locks_immediate`和`Table_locks_waited`状态变量来分析系统上的表锁定。具体说明如下：

`Table_locks_immediate`：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1。

`Table_locks_waited`：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁值加1），此值越高，则说明存在较严重的表级锁争用竞争情况，实际工程中看此值就够了

**此外，`MyISAM`的读写锁调度是写优先，这也是`MyISAM`不适合作为主表的引擎。因为写锁后，其他线程不能进行任何操作，大量的写操作会使查询很难得到锁，从而造成永远阻塞**


# 17.2.行锁(偏写)

> 行锁特点

- 偏向`InnoDB`存储引擎，开销大，加锁慢，会出现死锁，锁定粒度最小，发生锁冲突的概率最低，并发度最高
- `InnoDB`存储引擎和`MyISAM`存储引擎最大不同有两点：**一是支持事务，二是采用行级锁**

## 17.2.1.环境准备

```
# 建表语句
CREATE TABLE `test_innodb_lock`(
`a` INT,
`b` VARCHAR(16)
)ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT='测试行锁'; 

# 插入数据
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(1, 'b2');
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(3, '4000');
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(4, '5000');
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(5, '6000');
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(6, '7000');
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(7, '8000');
INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(8, '9000');

# 创建索引
CREATE INDEX idx_test_a ON `test_innodb_lock`(a);
CREATE INDEX idx_test_b ON `test_innodb_lock`(b);
```

## 17.2.2.行锁案例

> 1、开启手动提交

打开`SESSION1`和`SESSION2`两个会话，都开启手动提交

```
# 开启MySQL数据库的手动提交
mysql> SET autocommit=0;
Query OK, 0 rows affected (0.00 sec)
```

> 2、读己之所写

```
# SESSION1 

# SESSION1対test_innodb_lock表做写操作，但是没有commit
# 执行修改SQL之后，查询一下test_innodb_lock表，发现数据被修改了
mysql> UPDATE `test_innodb_lock` SET `b` = '88' WHERE `a` = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM `test_innodb_lock`;
+------+------+
| a    | b    |
+------+------+
|    1 | 88   |
|    3 | 4000 |
|    4 | 5000 |
|    5 | 6000 |
|    6 | 7000 |
|    7 | 8000 |
|    8 | 9000 |
+------+------+
8 rows in set (0.00 sec)

# SESSION2 

# SESSION2这时候来查询test_innodb_lock表
# 发现SESSION2是读不到SESSION1未提交的数据的，MySQL默认事务隔离级别是可重复读，不允许脏读
mysql> SELECT * FROM `test_innodb_lock`;
+------+------+
| a    | b    |
+------+------+
|    1 | b2   |
|    3 | 4000 |
|    4 | 5000 |
|    5 | 6000 |
|    6 | 7000 |
|    7 | 8000 |
|    8 | 9000 |
+------+------+
8 rows in set  (0.00 sec)
```

> 3、行锁两个SESSION同时対一条记录进行写操作

```
# SESSION1 対test_innodb_lock表的`a`=1这一行进行写操作，但是没有commit
mysql> UPDATE `test_innodb_lock` SET `b` = '99' WHERE `a` = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# SESSION2 也对test_innodb_lock表的`a`=1这一行进行写操作，但是发现阻塞了！！！
# 等SESSION1执行commit语句之后，SESSION2的SQL就会执行了
mysql> UPDATE `test_innodb_lock` SET `b` = 'asdasd' WHERE `a` = 1;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

只有等对方提交并且自己也提交后才可以看到最新数据，允许重复可读

> 4、行锁两个SESSION同时对不同记录进行写操作

```
# SESSION1 対test_innodb_lock表的`a`=6这一行进行写操作，但是没有commit
mysql> UPDATE `test_innodb_lock` SET `b` = '8976' WHERE `a` = 6;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

# SESSION2 対test_innodb_lock表的`a`=4这一行进行写操作，没有阻塞！！！
# SESSION1和SESSION2同时对不同的行进行写操作互不影响
mysql> UPDATE `test_innodb_lock` SET `b` = 'Ringo' WHERE `a` = 4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

## 17.2.3.字符串未加单引号，导致索引失效，行锁变表锁

```
# SESSION1 执行SQL语句，没有执行commit
# 由于`b`字段是字符串，但是没有加单引号导致索引失效
mysql> UPDATE `test_innodb_lock` SET `a` = 888 WHERE `b` = 8000;
Query OK, 1 row affected, 1 warning (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 1

# SESSION2 和SESSION1操作的并不是同一行，但是也被阻塞了？？？
# 由于SESSION1执行的SQL索引失效，导致行锁升级为表锁
mysql> UPDATE `test_innodb_lock` SET `b` = '1314' WHERE `a` = 1;
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

## 17.2.4.间隙锁的危害

> 什么是间隙锁？

当我们用范围条件而不是相等条件检索数据，并请求共享或者排他锁时，`InnoDB`会给符合条件的已有数据记录的索引项加锁，对于键值在条件范围内但并不存在的记录，叫做"间隙(GAP)"。`InnoDB`也会对这个"间隙"加锁，这种锁的机制就是所谓的"间隙锁(Next-key锁)"

> 间隙锁的危害

因为`Query`执行过程中通过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值不存在。间隙锁有一个比较致命的缺点，就是**当锁定一个范围的键值后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据**，在某些场景下这可能会対性能造成很大的危害

```
如：原始数据中a列的值不连续，缺失`a` = 2的记录

# SESSION1 対test_innodb_lock表进行更新操作，但是没有commit
mysql> UPDATE `test_innodb_lock` SET `b` = '0629' WHERE `a` > 1 and `a` < 6;
Query OK, 3 row affected (0.00 sec)
Rows matched: 3  Changed: 3  Warnings: 0

# SESSION2 対test_innodb_lock表新增一条`a` = 2的记录，发生了阻塞，SESSION1提交后阻塞被解除
# SESSION1和SESSION2同时对不同的行进行写操作互不影响
mysql> INSERT INTO `test_innodb_lock`(`a`, `b`) VALUES(2, '3');
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```


## 17.2.5.如何锁定一行（面试题）

![锁定一行](https://img-blog.csdnimg.cn/2020080616050355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

`SELECT ..... FOR UPDATE`在锁定某一行后，其他写操作会被阻塞，直到锁定行的会话执行`COMMIT`


## 17.2.6.案例结论

`InnoDB`存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于`MyISAM`的表级锁定的。当系统并发量较高的时候，`InnoDB`的整体性能和`MyISAM`相比就会有比较明显的优势了。但是，`InnoDB`的行级锁定同样也有其脆弱的一面，当我们使用不当的时候(如行锁变表锁)，可能会让`InnoDB`的整体性能表现不仅不能比`MyISAM`高，甚至可能会更差


## 17.2.7.行锁分析

```
mysql> SHOW STATUS LIKE 'innodb_row_lock%';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| Innodb_row_lock_current_waits | 0     |
| Innodb_row_lock_time          | 43497 |
| Innodb_row_lock_time_avg      | 21748 |
| Innodb_row_lock_time_max      | 32599 |
| Innodb_row_lock_waits         | 2     |
+-------------------------------+-------+
5 rows in set (0.00 sec)
```

対各个状态量的说明如下：

- `Innodb_row_lock_current_waits`：当前正在等待锁定的数量
- `Innodb_row_lock_time`：从系统启动到现在锁定总时间长度**（重要）**
- `Innodb_row_lock_time_avg`：每次等待所花的平均时间**（重要）**
- `Innodb_row_lock_time_max`：从系统启动到现在等待最长的一次所花的时间
- `Innodb_row_lock_waits`：系统启动后到现在总共等待的次数**（重要）**

尤其是当等待次数`Innodb_row_lock_waits`很高，而且每次等待平均时长`Innodb_row_lock_time_avg`也不小的时候，我们就需要通过`Show Profiles`分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优化策略

## 17.2.8.优化建议

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁(Varchar类型不加单引号是重罪！)
- 合理设计索引，尽量缩小锁的范围(避免间隙锁的危害)
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可能低级别事务隔离

## 17.3.页锁

开销和加锁时间介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发性一般(使用较少，了解即可)


# 18.主从复制

## 18.1.复制基本原理

![主从复制](https://img-blog.csdnimg.cn/20200806170415401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1JyaW5nb18=,size_16,color_FFFFFF,t_70)

MySQL复制过程分为三步：

- Master将改变记录到二进制日志中(Binary Log)，这些记录过程叫做二进制日志事件(`Binary Log Events`)
- Slave将Master的`Binary Log Events`拷贝到它的中继日志(Replay  Log)中
- Slave重做中继日志中的事件，将改变应用到自己的数据库中，MySQL复制是异步且串行化的

## 18.2.复制基本原则

- 每个Slave只有一个Master
- 每个Slave只能有一个唯一的服务器ID
- 每个Master可以有多个Slave

## 18.3.一主一从常见配置

> 1、基本要求：主机Master和从机Slave的MySQL服务器版本一致且后台以服务运行

说明：此处主机在Windows上，从机在Linux上

```
# 检查主从机网络是否互通
linux> ping ip地址

# 创建mysql-slave1实例(使用docker)
docker run -p 3307:3306 --name mysql-slave1 \
-v /root/mysql-slave1/log:/var/log/mysql \
-v /root/mysql-slave1/data:/var/lib/mysql \
-v /root/mysql-slave1/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=333 \
-d mysql:5.7
```

> 2、主从配置都是配在[mysqld]节点下，都是小写

```
# 主机修改my.ini配置文件(windows版)
[mysqld]
# 必须设置项
server-id=1  # 主服务器唯一ID
log-bin=自己本地路径/mysqlbin # 启用二进制日志 

#可选项
log-err=自己本地路劲/mysqlerr # 启用错误日志
basedir = '自己本地路径' # 根目录
tmpdir = '自己本地路径' # 临时目录
datadir = 自己本地路径/Data/ # 数据目录
read-only=0 # 主机，读写都可以
binlog-ignore-db=mysql # 设置不要复制的数据库
binlog-do-db=需要复制的主数据库名字 # 设置需要复制的数据库

# 从机修改my.cnf配置文件(Linux版)
linux> vim /etc/my.cnf
[mysqld]
# 必须设置项
server-id=2 # 从服务器唯一ID
# 可选项
log-bin=/var/lib/mysql/mysql-bin # 启用二进制日志

# 重启数据库
linux> service mysql stop
linux> service mysql start
linux> ps -ef | grep mysql

# 主从机都关闭防火墙
windows手动关闭
linux> service iptables stop
```

> 3、Master主机配置

在Windows主机上建立账户并授权从机

```
# Windows上启用MySQL
直接切换到MySQL安装的bin目录下，执行mysql -u root -p

# Windows上授权：GRANT REPLICATION SLAVE ON *.* TO 'username'@'从机数据库IP地址' IDENTIFIED BY 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'zhangsan'@'172.18.0.3' IDENTIFIED BY '123456';
Query OK, 0 rows affected, 1 warning (0.01 sec)

# 刷新命令
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

# 记录下File和Position
# 每次配从机的时候都要SHOW MASTER STATUS; 
# 查看最新的File和Position
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      602 |              | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

> 4、Slave从机配置

```
# Linux上配置：CHANGE MASTER TO MASTER_HOST='主机IP地址', MASTER_USER='username', MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.File的编号', MASTER_LOG_POS=Position的最新值;

# 使用用户名密码登录进Master
mysql> CHANGE MASTER TO MASTER_HOST='172.18.0.4',
    -> MASTER_USER='zhangsan',
    -> MASTER_PASSWORD='123456',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=602;
Query OK, 0 rows affected, 2 warnings (0.02 sec)

# 开启Slave从机的复制
mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)

# 查看Slave状态
# Slave_IO_Running 和 Slave_SQL_Running 必须同时为Yes 说明主从复制配置成功！
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event # Slave待命状态
                  Master_Host: 172.18.0.4
                  Master_User: zhangsan
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 602
               Relay_Log_File: b030ad25d5fe-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes  [必须为Yes]
            Slave_SQL_Running: Yes [必须为Yes]
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 602
              Relay_Log_Space: 534
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: bd047557-b20c-11ea-9961-0242ac120002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

> 5、测试主从复制

```
# Master主机创建数据库(Windows)
mysql> CREATE DATABASE test_replication;
Query OK, 1 row affected (0.01 sec)

# 使用数据库
mysql> USE test_replication;
Database changed

# 新建表
mysql> CREATE TABLE dog (id INT NOT NULL, name VARCHAR(20));
Query OK, 0 row affected (0.01 sec)

# 插入记录
mysql> INSERT INTO dog VALUES(1, 'ww1');
Query OK, 1 row affected (0.01 sec)

# Slave从机查询数据库 (Linux)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test_replication   |
+--------------------+
5 rows in set (0.00 sec)

# 使用数据库
mysql> USE test_replication;
Database changed

mysql> SELECT * FROM dog;
+------+----------+
|  id  |   name   |
+------+----------+
|  1   |   ww1    |
+------+----------+
```

> 6、停止主从复制功能

```
# 停止从机复制功能，运行在重新配置主从之前(必须)
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.00 sec)

# 重新配置主从，每次都要使用SHOW MASTER STATUS;，找到最新的MASTER_LOG_FILE 和 MASTER_LOG_POS来更新配置
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      797 |              | mysql            |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> CHANGE MASTER TO MASTER_HOST='172.18.0.4',
    -> MASTER_USER='zhangsan',
    -> MASTER_PASSWORD='123456',
    -> MASTER_LOG_FILE='mysql-bin.000001',
    -> MASTER_LOG_POS=797;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

# 开启Slave从机的复制
mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)


# 查看Slave状态
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.18.0.4
                  Master_User: zhangsan
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 797
               Relay_Log_File: b030ad25d5fe-relay-bin.000002
                Relay_Log_Pos: 320
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 797
              Relay_Log_Space: 534
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: bd047557-b20c-11ea-9961-0242ac120002
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

# 19.MySQL表分区

[参考文章](https://www.jianshu.com/p/1cdd3e3c5b3c)

## 19.1 分区表

> 是什么？

通俗地讲表分区是**将一大表，根据条件分割成若干个小表**，Mysql5.1开始支持数据表分区。如：某用户表的记录超过了600万条，那么就可以根据入库日期将表分区，也可以根据所在地将表分区，当然也可根据其他的条件分区

> 实现原理

分区表是由多个相关的底层表实现，这些底层表也是由句柄对象表示，所以我们也可以直接访问各个分区。存储引擎管理分区的各个底层表和管理普通表一样（所有的底层表都必须使用相同的存储引擎），分区表的索引只是在各个底层表上各自加上一个相同的索引，从存储引擎的角度来看，底层表和一个普通表没有任何不同，存储引擎也无须知道这是一个普通表还是一个分区表的一部分

在分区表上的操作按照下面的操作逻辑进行：

- `Select`查询：当查询一个分区表的时候，分区层先打开并锁住所有的底层表，优化器判断是否可以过滤部分分区，然后再调用对应的存储引擎接口访问各个分区的数据
- `Insert`操作：当写入一条记录时，分区层打开并锁住所有的底层表，然后确定哪个分区接受这条记录，再将记录写入对应的底层表
- `Delete`操作：当删除一条记录时，分区层先打开并锁住所有的底层表，然后确定数据对应的分区，最后对相应底层表进行删除操作
- `Update`操作：当更新一条数据时，分区层先打开并锁住所有的底层表，Mysql先确定需要更新的记录在哪个分区，然后取出数据并更新，再判断更新后的数据应该放在哪个分区，然后对底层表进行写入操作，并对原数据所在的底层表进行删除操作

虽然每个操作都会打开并锁住所有的底层表，但这并不是说分区表在处理过程中是锁住全表的，如果存储引擎能够自己实现行级锁，如InnoDB，则会在分区层释放对应的表锁，这个加锁和解锁过程与普通InnoDB上的查询类似

> 优势与劣势

优势：

+ 与单个磁盘或文件系统分区相比，可以存储更多的数据
+ 对于那些已经失去保存意义的数据，通常可以通过删除与那些数据有关的分区，很容易地删除那些数据。相反地，在某些情况下，添加新数据的过程又可以通过为那些新数据专门增加一个新的分区，来很方便地实现
+ 一些查询可以得到极大的优化，这主要是借助于满足一个给定WHERE语句的数据可以只保存在一个或多个分区内，这样在查找时就不用查找其他剩余的分区
+ 涉及到例如SUM()和COUNT()这样聚合函数的查询，可以很容易地进行并行处理。这种查询的一个简单例子如 `SELECT salesperson_id, COUNT (orders) as order_total FROM sales GROUP BY salesperson_id；`, 通过“并行”，这意味着该查询可以在每个分区上同时进行，最终结果只需通过总计所有分区得到的结果
+ 通过跨多个磁盘来分散数据查询，来获得更大的查询吞吐量

劣势(更多的是限制)：

+ 一个表最多只能有1024个分区（`MySQL5.6`之后支持8192个分区）
+ 在`MySQL5.1`中分区表达式必须是整数，或者是返回整数的表达式，在`MySQL5.5`之后，某些场景可以直接使用字符串列和日期类型列来进行分区
+ 如果分区字段中有主键或者唯一索引列，那么所有主键列和唯一索引列都必须包含进来，如果表中有主键或唯一索引，那么分区键必须是主键或唯一索引
+ 分区表中无法使用外键约束
+ MySQL数据库支持的分区类型为水平分区，并不支持垂直分区，因此，MySQL数据库的分区中索引是局部分区索引，一个分区中既存放了数据又存放了索引，而全局分区是指的数据库放在各个分区中，但是所有的数据的索引放在另外一个对象中
+ 目前MySQL不支持空间类型和临时表类型进行分区。不支持全文索引


## 19.2 表分区的类型

> RANGE分区(常用)

基于属于一个给定连续区间的列值，把多行分配给分区，这些区间要连续且不能相互重叠，通过使用`PARTITION BY RANGE(expr)`和`VALUES LESS THAN (num)`操作符来进行定义

```
mysql> create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by range (store_id) (
    partition p0 values less than (6),
    partition p1 values less than (11),
    partition p2 values less than (16),
    partition p3 values less than (21)，
    partition p4 values less than maxvalue
);
```

按照这种分区方案，在商店1到5工作的雇员相对应的所有行被保存在分区P0中，商店6到10的雇员保存在P1中，依次类推。注意，每个分区都是按顺序进行定义，从最低到最高。这是PARTITION BY RANGE 语法的要求；在这点上，它类似于C或Java中的“switch ... case”语句

> LIST分区

类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。通过使用`PARTITION BY LIST(expr)`和`VALUES IN(value_list)`操作符来进行定义， 将要匹配的任何值都必须在值列表(value_list)中全部列出

```
假定有20个音像店，分布在4个有经销权的地区，如下所示：
====================================
地区      商店ID 号
------------------------------------
北区      3, 5, 6, 9, 17
东区      1, 2, 10, 11, 19, 20
西区      4, 12, 13, 14, 18
中心区   7, 8, 15, 16
====================================

要按照属于同一个地区商店的行保存在同一个分区中的方式来分割表，可以使用下面的“CREATE TABLE”语句：

mysql> create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by list(store_id)
    partition pNorth values in (3,5,6,9,17),
    partition pEast values in (1,2,10,11,19,20),
    partition pWest values in (4,12,13,14,18),
    partition pCentral values in (7,8,15,16)
)；
```

这使得在表中增加或删除指定地区的雇员记录变得容易起来。例如，假定西区的所有音像店都卖给了其他公司。那么与在西区音像店工作雇员相关的所有记录（行）可以使用查询`ALTER TABLE employees DROP PARTITION pWest；`来进行删除，它与具有同样作用的`DELETE query DELETE FROM employees WHERE store_id IN (4,12,13,14,18)；`比起来效率要高很多

> HASH分区

基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算，通过使用`PARTITION BY HASH(expr)`和`PARTITIONS num`操作符来进行定义

```
mysql> create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by hash(store_id)
partitions 4；  #表将要被分割成分区的数量， 默认为1
```

> KEY分区

类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL 服务器提供其自身的哈希函数。必须有一列或多列包含整数值

```
mysql> create table tk (
    col1 int not null,
    col2 char(5),
    col3 date
) partition by linear key (col1)
partitions 3;
```

在KEY分区中使用关键字LINEAR和在HASH分区中使用具有同样的作用，分区的编号是通过2的幂（powers-of-two）算法得到，而不是通过模数算法


## 19.3 表分区的相关操作

```
# 建立表分区
mysql> create table employees (
    id int not null,
    fname varchar(30),
    lname varchar(30),
    hired date not null default '1970-01-01',
    separated date not null default '9999-12-31',
    job_code int not null,
    store_id int not null
) partition by range (store_id) (
    partition p0 values less than (6),
    partition p1 values less than (11),
    partition p2 values less than (16),
    partition p3 values less than (21)，
    partition p4 values less than maxvalue
);

# 增加表分区
mysql> ALTER TABLE sale_data ADD PARTITION (PARTITION s20100402 VALUES LESS THAN (20100403));

# 删除表分区
mysql> ALTER TABLE sale_data DROP PARTITION s20100406 ;

# 向表分区插入数据
mysql> insert into sale_data values('2010-04-01','11',11.11);

# 查看表分区
mysql> SELECT PARTITION_NAME,TABLE_ROWS FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'sale_data';
```

## 19.4 分区使用场景

- 当数据量很大(过T)时，肯定不能把数据一次性加载到内存中，这样查询一个或一定范围的记录是很耗时。另外一般这情况下，历史数据或不常访问的数据占很大部分，最新或热点数据占的比例不是很大，这时可以根据某些条件进行表分区
- 分区表的更易于管理，比如删除过去某一时间的历史数据，直接执行`TRUNCATE`，或者狠点`DROP`整个分区，这比`DELETE`删除效率更高
- 当数据量很大，或者将来很大的，但单块磁盘的容量不够，或者想提升I/O效率的时候，可以把未分区中的子分区挂载到不同的磁盘上
- 使用分区表可避免某些特殊的瓶颈，例如InnoDB的单个索引的互斥访问
- 在某些场景下，单个分区表的备份和恢复更有效率
- 项目中需要动态新建、删除分区。如新闻表，按照时间维度中的月份对其分区，为了防止新闻表过大，只保留最近6个月的分区，同时预建后面3个月的分区，这个删除、预建分区的过程就是分区表的动态管理

总结：可伸缩性，可管理性，提高数据库查询效率


# 20.实际应用场景

> MySQL如何快速删除大量数据(千万级别)？ [参考文章+Python代码实现](https://blog.csdn.net/weixin_33817140/article/details/114344233?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

+ 方法一：如果删除整表数据，直接使用TRUNCATE TABLE命令就好
+ 方法二：使用DELETE进行批量删除，通过LIMIT每次限定一定的数量，然后循环删除知道全部数据删除完毕，同时key_buffer_size由默认的8M提高512M
+ 方法三：使用HASH分区通过PARTITION BY给表创建分区，通过EXPLAIN PARTITIONS获取分区后, 使用TRUNCATE PARTITION直接按分区进行删除,特别快

注意：如果删除的数据超过表数据的百分之50，建议拷贝所需数据到临时表，创建备份，然后删除原表，再重命名临时表为原表
