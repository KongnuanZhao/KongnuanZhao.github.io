title: 关于索引的一些总结和思考（上）
date: 2015-03-24 18:49:38
author: 赵空暖
tags: oracle
categories: oracle
---
![about_index](/image/about_index.jpg)
#索引之基础#
##什么是索引##
数据库的索引类似于书籍的索引。在书籍中，索引允许用户不必翻阅完整个书就能迅速地找到所需要的信息。在数据库中，索引也允许数据库程序迅速地找到表中的数据，而不必扫描整张表。
在Oracle中索引和表一样，是段的一种，当我们建一个表，就产生一个表的segment，当在表上的某些列上建索引，就产生一个索引的segment。索引的目的就是让表的查询更快。
##如何实现##
这是因为索引最低级的块（叶块）存储了索引键跟rowid来实现快速获取记录的。查询时的条件（索引键)定位到rowid即数据行的物理地址，从而通过rowid读取到目标数据。
![index](/image/index.png)
这个树底层的块称为叶子节点/叶子块(leaf)，其中分别包含各个索引列具体值（key column value）以及能具体定位到数据块所在位置的rowid(它是指向所索引的行)。叶子节点之上的内部块称为分支块(branch block)/导航块，这些节点用于实现导航。例如，如果想在索引中找到值20，要从树顶开始，找到左分支，我们检查这个块，并发现需要找到范围"20..25"的块，这个块将是叶子块，其中会指示包含数20的行。
《收获，不止oracle》中的一个例子
如果我们要执行`select * from t where id=12;`该t表有10050条记录，而id=12仅仅返回一条，t表的id列建了一个索引，通过索引是如何快速的检索到数据的呢？如下图所示，
![index_select](/image/index_select.png)
在这个索引高度下，定位到`select * from t where id=12`需要三次IO找到id=12所在的行的rowid。假设Oracle全表扫描，遍历所有的数据块，IO的次数必然会超过3次。所以有了这个索引，Oracle就只会去定位扫描部分索引块，提高了效率。
（这里 `select *`中的`*`是指一行，而该索引只是对id列建索引，因此查询到rowid后必然通过rowid去访问数据块，取出该条数据共4次IO ）

##索引的特点##
还是如上图所示，最底层的叶子块因为装具体的数据，所以比较容易填满，特别是对长度很长的列建索引时更是如此。但在第一层之上的第二层的索引块就不容易填满了，因为第二层只是装第一层的指针而已，而第三层是装第二层的索引块的指针……
索引是顺序从表里取数据，再顺序插入到块里形成索引块的，所以说索引块是有序的。所以总结索引的特点：
* 索引主要有三大特点
	* 索引树的高度一般都比较低。
	* 索引由索引列存储的值以及rowid组成。
	* 索引本身是有序的。

#索引之实验#
```bash
C:\Users\Administrator\Desktop>sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on 星期二 3月 24 19:07:38 2015

Copyright (c) 1982, 2010, Oracle.  All rights reserved.

连接到:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, Oracle Label Security, OLAP, Data Mining
and Real Application Testing options

SQL> create table t1 as select rownum as id ,rownum+1 as id2 from dual connect by level<=5;

表已创建。

SQL> create table t2 as select rownum as id ,rownum+1 as id2 from dual connect by level<=50;

表已创建。

SQL> create table t3 as select rownum as id ,rownum+1 as id2 from dual connect by level<=500;

表已创建。

SQL> create table t4 as select rownum as id ,rownum+1 as id2 from dual connect by level<=5000;

表已创建。

SQL> create table t5 as select rownum as id ,rownum+1 as id2 from dual connect by level<=50000;

表已创建。

SQL> create table t6 as select rownum as id ,rownum+1 as id2 from dual connect by level<=500000;

表已创建。

SQL> create table t7 as select rownum as id ,rownum+1 as id2 from dual connect by level<=5000000;
create table t7 as select rownum as id ,rownum+1 as id2 from dual connect by level<=5000000
                                                             *
第 1 行出现错误:
ORA-30009: CONNECT BY 操作内存不足

SQL> create index idx_id_t1 on t1(id);

索引已创建。

SQL> create index idx_id_t2 on t2(id);

索引已创建。

SQL> create index idx_id_t3 on t3(id);

索引已创建。

SQL> create index idx_id_t4 on t4(id);

索引已创建。

SQL> create index idx_id_t5 on t5(id);

索引已创建。

SQL> create index idx_id_t6 on t6(id);

索引已创建。

```
查看这些索引的大小
```bash
SQL> col SEGMENT_NAME format a30
SQL> select segment_name,bytes/1024 as kb from user_segments where segment_name in ('IDX_ID_T1','IDX
_ID_T2','IDX_ID_T3','IDX_ID_T4','IDX_ID_T5','IDX_ID_T6');

SEGMENT_NAME                           KB
------------------------------ ----------
IDX_ID_T1                              64
IDX_ID_T2                              64
IDX_ID_T3                              64
IDX_ID_T4                             128
IDX_ID_T5                             896
IDX_ID_T6                            9216

已选择6行。
```
查看索引高度
```bash
SQL> set linesize 1000
SQL> select index_name,blevel,leaf_blocks,num_rows,distinct_keys,clustering_factor from user_ind_sta
tistics where table_name in('T1','T2','T3','T4','T5','T6');

INDEX_NAME          BLEVEL LEAF_BLOCKS   NUM_ROWS DISTINCT_KEYS CLUSTERING_FACTOR
--------------- ---------- ----------- ---------- ------------- -----------------
IDX_ID_T1                0           1          5             5                 1
IDX_ID_T2                0           1         50            50                 1
IDX_ID_T3                1           2        500           500                 1
IDX_ID_T4                1          11       5000          5000                 9
IDX_ID_T5                1         110      50000         50000               101
IDX_ID_T6                2        1113     500000        500000              1033

已选择6行。
```
可以看出索引的大小从64KB到9216KB，而BLEVEL最小的是0层（0的取值是指第一个BLOCK还没有被索引填满，还没有产生管理的索引块，相当于最底下的第一层，以此类推……），最大的也不过2层而已，这再一次印证了索引的特点：索引树的高度一般都比较低。
* 简单总结
	* 记录数差异显著的表索引大小可以相同
	* 记录数差异显著的表索引扫描性能可以相同（比如500,5000,50000条的BLEVEL都是1 即两层,找到rowid的IO数是一样的）

比较有无索引的差别，对t6表做查询，执行语句执行两遍以上，消除物理读和递归调用的干扰。
```bash
SQL> set autotrace traceonly
SQL> set timing on
SQL> select * from t6 where id=10;

已用时间:  00: 00: 00.20

执行计划
----------------------------------------------------------
Plan hash value: 1902844584

-----------------------------------------------------------------------------------------
| Id  | Operation                   | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
-----------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |           |     1 |    26 |     4   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID| T6        |     1 |    26 |     4   (0)| 00:00:01 |
|*  2 |   INDEX RANGE SCAN          | IDX_ID_T6 |     1 |       |     3   (0)| 00:00:01 |
-----------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("ID"=10)

Note
-----
   - dynamic sampling used for this statement (level=2)


统计信息
----------------------------------------------------------
         52  recursive calls
          0  db block gets
         85  consistent gets
          2  physical reads
          0  redo size
        593  bytes sent via SQL*Net to client
        519  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed

SQL> select * from t6 where id=10;

已用时间:  00: 00: 00.01

执行计划
----------------------------------------------------------
Plan hash value: 1902844584

-----------------------------------------------------------------------------------------
| Id  | Operation                   | Name      | Rows  | Bytes | Cost (%CPU)| Time     |
-----------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |           |     1 |    26 |     4   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID| T6        |     1 |    26 |     4   (0)| 00:00:01 |
|*  2 |   INDEX RANGE SCAN          | IDX_ID_T6 |     1 |       |     3   (0)| 00:00:01 |
-----------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("ID"=10)

Note
-----
   - dynamic sampling used for this statement (level=2)


统计信息
----------------------------------------------------------
          0  recursive calls
          0  db block gets
          5  consistent gets
          0  physical reads
          0  redo size
        593  bytes sent via SQL*Net to client
        519  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed

SQL>
```
可以看到通过索引查询已用时间为00.01，5次consistent gets（逻辑读）， Cost (%CPU)为4，接下来比较没有索引即全表扫描的情况，
```bash
SQL> drop index IDX_ID_T6;

索引已删除。

已用时间:  00: 00: 00.20
SQL> select * from t6 where id=10;

已用时间:  00: 00: 00.05

执行计划
----------------------------------------------------------
Plan hash value: 1930642322

--------------------------------------------------------------------------
| Id  | Operation         | Name | Rows  | Bytes | Cost (%CPU)| Time     |
--------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |      |    11 |   286 |   290   (3)| 00:00:04 |
|*  1 |  TABLE ACCESS FULL| T6   |    11 |   286 |   290   (3)| 00:00:04 |
--------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - filter("ID"=10)

Note
-----
   - dynamic sampling used for this statement (level=2)


统计信息
----------------------------------------------------------
        117  recursive calls
          0  db block gets
       1122  consistent gets
          0  physical reads
          0  redo size
        589  bytes sent via SQL*Net to client
        519  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          2  sorts (memory)
          0  sorts (disk)
          1  rows processed

SQL> select * from t6 where id=10;

已用时间:  00: 00: 00.04

执行计划
----------------------------------------------------------
Plan hash value: 1930642322

--------------------------------------------------------------------------
| Id  | Operation         | Name | Rows  | Bytes | Cost (%CPU)| Time     |
--------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |      |    11 |   286 |   290   (3)| 00:00:04 |
|*  1 |  TABLE ACCESS FULL| T6   |    11 |   286 |   290   (3)| 00:00:04 |
--------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - filter("ID"=10)

Note
-----
   - dynamic sampling used for this statement (level=2)


统计信息
----------------------------------------------------------
          0  recursive calls
          0  db block gets
       1039  consistent gets
          0  physical reads
          0  redo size
        589  bytes sent via SQL*Net to client
        519  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed

SQL>
```
所用时间为00.04，是原来的四倍，产生1039次consistent gets（逻辑读）是原来的200倍，Cost (%CPU)为290为原来的70倍。
但也不是什么场合用索引都合适，比如如果索引的高度为3，查询一条记录大致需要3到4次IO，如果查询返回100万条记录，就是100万乘上3或者4，那就等于三四百万的IO数量，而这简单的100万条记录不需三四百万个块来装，不如全表扫描呢。全表扫描有一个好处，就是一次可以读取多个块，不仅是一次读取一个块，这样IO的次数还可以大大降低。当然如果这100万条记录字段长度很长，遍历的数据块也会成倍增加，全表扫描的速度也会慢下来了。所以一个优化的方向就是精简字段。
通过这个实验也可以总结出索引改进性能的程度取决于：
* 数据的选择性
如果数据非常具有选择性，则表中将只有很少的行匹配索引值。Oracle将能够快速查询匹配索引值的Rowid的索引，并且可以快速查询少量的相关表块。如果数据选择性不高，则索引可能返回许多rowid，导致从表中查询许多单独的块，则其访问的数据块可能比全表扫描还高。
* 在表的块之间分布数据的方式
如果数据非常具有选择性，但相关的数据的行在表中的存储位置并不互相靠近，则会进一步降低索引的作用。如果匹配索引值的数据分散在多个块中，则必须从表中选择多个单独的块以满足查询。在一些情况中，您会发现当数据分散在表的多个块中时，最好是不使用索引，而是执行全表扫描。执行全表扫描时，Oracle使用多块读取以满足快速扫描表。基于索引的读取是单块读取，因此在使用索引的时的目标是减少解决查询所需要的单个块的数量。 
* 降低了数据库插入、修改、删除等的速度，原因很简单，基表DML操作的时候需要维护索引。

今天的总结到这里啦！


