title: Oracle的导入导出
date: 2015-03-20 13:11:20
author: 赵空暖
tags: oracle
categories: oracle
---
从Oracle10g往后导入导出就有两种方式，一种是传统的exp/imp,另一种则是更为高效的expdp/impdp。
#为什么有#
* 传统的imp/exp更简单一点点。
* 数据泵:使用数据泵可以在数据库之间高速传输数据。
  * 用于替代老的exp/imp,速度更快，机制上不同于老的 exp/imp，提供更多的功能。
  * 始于Oracle10g,从Oracle11g开始，不再提供老的imp/exp的支持。
  * expdp差不多2倍于传统的exp操作，impdp差不多15-40倍与传统的imp操作。
  * 如果启用并行操作，性能会更好。更好的利用并行技术，具有对大量数据处理的优势。
    * 全部操作在server端完成，避免了数据在网络上的传输。
    * 导出文件的格式更接近于数据库本身文件的格式，避免了数据写入文件时的转换。
    * 直接路径的加载，使效率更高。
    * 元数据和数据在导出过程中可以重叠进行，提高导出的效率。

#实际操作#
##exp/imp方式##
###exp导出###
* 将数据库orcl完全导出,用户名zkn 密码zkn 导出到D:\zkn.dmp中
```bash
exp zkn/zkn@orcl file=d:\zkn.dmp full=y
```
* 导出test用户数据
```bash
C:\Users\Administrator\Desktop>exp test/test@orcl file=d:\test.dmp

Export: Release 11.2.0.1.0 - Production on 星期五 3月 20 13:36:45 2015

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, Oracle Label Security, OLAP, Data Mining
and Real Application Testing options
已导出 ZHS16GBK 字符集和 AL16UTF16 NCHAR 字符集

即将导出指定的用户...
. 正在导出 pre-schema 过程对象和操作
. 正在导出用户 TEST 的外部函数库名
. 导出 PUBLIC 类型同义词
. 正在导出专用类型同义词
. 正在导出用户 TEST 的对象类型定义
即将导出 TEST 的对象...
. 正在导出数据库链接
. 正在导出序号
. 正在导出簇定义
. 即将导出 TEST 的表通过常规路径...
. . 正在导出表                               T导出了           2 行
. . 正在导出表                             ZKN导出了           4 行
. . 正在导出表                            city导出了           2 行
. 正在导出同义词
. 正在导出视图
. 正在导出存储过程
. 正在导出运算符
. 正在导出引用完整性约束条件
. 正在导出触发器
. 正在导出索引类型
. 正在导出位图, 功能性索引和可扩展索引
. 正在导出后期表活动
. 正在导出实体化视图
. 正在导出快照日志
. 正在导出作业队列
. 正在导出刷新组和子组
. 正在导出维
. 正在导出 post-schema 过程对象和操作
. 正在导出统计信息
成功终止导出, 没有出现警告。

```
将数据库中test用户与scott用户的表导出
```bash
exp system/zkn@orcl file=d:\test_scott.dmp owner=(test,scott)
```
* 将数据库中的表table1 、table2导出
```bash
exp system/zkn@orcl file=d:\tables.dmp tables=(table1,table2) 
```

###导入imp###
* 将D:\zkn.dmp 中的数据导入 TEST数据库中。
```bash
imp zkn/zkn@orcl file=d:\zkn.dmp
```
如果有的表已经存在，然后它就报错，对该表就不进行导入。在后面加上 ignore=y 就可以了。
* 将test用户数据导入
若用户不存在要创建且授权
```bash
SQL> create user test identified by test;
用户已创建。
SQL> grant dba to test;
授权成功。
```
```bash
C:\Users\Administrator\Desktop>imp test/test@orcl file=d:\test.dmp full=y

Import: Release 11.2.0.1.0 - Production on 星期五 3月 20 13:47:01 2015

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.


连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, Oracle Label Security, OLAP, Data Mining
and Real Application Testing options

经由常规路径由 EXPORT:V11.02.00 创建的导出文件
已经完成 ZHS16GBK 字符集和 AL16UTF16 NCHAR 字符集中的导入
. 正在将 TEST 的对象导入到 TEST
. . 正在导入表                             "T"导入了           2 行
. . 正在导入表                           "ZKN"导入了           4 行
. . 正在导入表                          "city"导入了           2 行
成功终止导入, 没有出现警告。
```
* 将d:\tables.dmp中的表table1 导入
```bash
imp zkn/zkn@orcl  file=d:\tables.dmp  tables=(table1) 
```

##数据泵expdp/impdp##
###导出empdp###
* 创建DIRECTORY
```bash
create directory dir_dp as 'D:\oracle\dir_dp';
``` 
* 授权
```bash
Grant read,write on directory dir_dp to test;
```
* 查看目录及权限
```bash
SELECT privilege, directory_name, DIRECTORY_PATH FROM user_tab_privs t, all_directories d
 WHERE t.table_name(+) = d.directory_name ORDER BY 2, 1;
 ```
* 执行导出
```bash
expdp test/test@orcl schemas=test directory=dir_dp dumpfile =expdp_test1.dmp logfile=expdp_test1.log;

C:\Users\Administrator\Desktop>expdp test/test@orcl schemas=test directory=dir_dp dumpfile=expdp_tes
t1.dmp;

Export: Release 11.2.0.1.0 - Production on 星期五 3月 20 13:59:37 2015

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, Oracle Label Security, OLAP, Data Mining
and Real Application Testing options
自动启用 FLASHBACK 以保持数据库完整性。
启动 "TEST"."SYS_EXPORT_SCHEMA_01":  test/********@orcl schemas=test directory=dir_dp dumpfile=expdp
_test1.dmp;
正在使用 BLOCKS 方法进行估计...
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE_DATA
使用 BLOCKS 方法的总估计: 192 KB
处理对象类型 SCHEMA_EXPORT/USER
处理对象类型 SCHEMA_EXPORT/SYSTEM_GRANT
处理对象类型 SCHEMA_EXPORT/ROLE_GRANT
处理对象类型 SCHEMA_EXPORT/DEFAULT_ROLE
处理对象类型 SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE
处理对象类型 SCHEMA_EXPORT/TABLE/INDEX/INDEX
处理对象类型 SCHEMA_EXPORT/TABLE/CONSTRAINT/CONSTRAINT
处理对象类型 SCHEMA_EXPORT/TABLE/INDEX/STATISTICS/INDEX_STATISTICS
处理对象类型 SCHEMA_EXPORT/TABLE/COMMENT
处理对象类型 SCHEMA_EXPORT/PROCEDURE/PROCEDURE
处理对象类型 SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
处理对象类型 SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
处理对象类型 SCHEMA_EXPORT/POST_SCHEMA/PROCACT_SCHEMA
. . 导出了 "TEST"."T"                                  5.007 KB       2 行
. . 导出了 "TEST"."ZKN"                                5.078 KB       4 行
. . 导出了 "TEST"."city"                               6.851 KB       2 行
已成功加载/卸载了主表 "TEST"."SYS_EXPORT_SCHEMA_01"
******************************************************************************
TEST.SYS_EXPORT_SCHEMA_01 的转储文件集为:
  D:\ORACLE\DIR_DP\EXPDP_TEST1.DMP;
作业 "TEST"."SYS_EXPORT_SCHEMA_01" 已于 14:01:00 成功完成

```
注意：本地一定要有`dir_dp`即`D:\oracle\dir_dp`这个文件夹。
* 导出的各种模式
按表模式导出：
```bash
expdp test/test@orcl  tables=zkn,city dumpfile=expdp_test2.dmp logfile=expdp_test2.log directory=dir_dp job_name=my_job
```
按查询条件导出：
```bash
expdp test/test@orcl  tables=zkn dumpfile=expdp_test3.dmp logfile=expdp_test3.log directory=dir_dp job_name=my_job query='"where rownum<11"'
```
按表空间导出：
```bash
expdp test/test@orcl dumpfile=expdp_tablespace.dmp tablespaces=TEST.DBF logfile=expdp_tablespace.log directory=dir_dp job_name=my_job
```
导出方案
```bash
expdp test/test@orcl DIRECTORY=dir_dp DUMPFILE=schema.dmp SCHEMAS=ZKN,TEST
```
导出整个数据库：
```bash
expdp test/test@orcl dumpfile =full.dmp full=y logfile=full.log directory=dir_dp job_name=my_job
```

###导入impdp###
* 按表导入
```bash
SQL> conn test/test;
已连接。
SQL> select * from zkn;

        ID
----------
         1
         1
         1
         3

SQL> drop table zkn purge;

表已删除。
```
expdp_test1.dmp;文件中的表，此文件是以test用户按schemas=test导出的：
```bash
C:\Users\Administrator\Desktop>impdp test/test@orcl directory=dir_dp dumpfile=expdp_test1.dmp; table
s=ZKN

Import: Release 11.2.0.1.0 - Production on 星期五 3月 20 14:19:45 2015

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, Oracle Label Security, OLAP, Data Mining
and Real Application Testing options
已成功加载/卸载了主表 "TEST"."SYS_IMPORT_TABLE_01"
启动 "TEST"."SYS_IMPORT_TABLE_01":  test/********@orcl directory=dir_dp dumpfile=expdp_test1.dmp; ta
bles=ZKN
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE_DATA
. . 导入了 "TEST"."ZKN"                                5.078 KB       4 行
处理对象类型 SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
作业 "TEST"."SYS_IMPORT_TABLE_01" 已于 14:19:49 成功完成
```
重新查看
```bash
SQL> select * from zkn;

        ID
----------
         1
         1
         1
         3
```
导入成功。
* 按用户导入（可以将用户信息直接导入，即如果用户信息不存在的情况下也可以直接导入）
```bash
C:\Users\Administrator\Desktop>impdp test/test@orcl schemas=test dumpfile=EXPDP_TEST1.DMP; directory
=dir_dp job_name=my_job

Import: Release 11.2.0.1.0 - Production on 星期五 3月 20 14:18:07 2015

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.

连接到: Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, Oracle Label Security, OLAP, Data Mining
and Real Application Testing options
已成功加载/卸载了主表 "TEST"."MY_JOB"
启动 "TEST"."MY_JOB":  test/********@orcl schemas=test dumpfile=EXPDP_TEST1.DMP; directory=dir_dp jo
b_name=my_job
处理对象类型 SCHEMA_EXPORT/USER
ORA-31684: 对象类型 USER:"TEST" 已存在
处理对象类型 SCHEMA_EXPORT/SYSTEM_GRANT
处理对象类型 SCHEMA_EXPORT/ROLE_GRANT
处理对象类型 SCHEMA_EXPORT/DEFAULT_ROLE
处理对象类型 SCHEMA_EXPORT/PRE_SCHEMA/PROCACT_SCHEMA
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE
ORA-39151: 表 "TEST"."T" 已存在。由于跳过了 table_exists_action, 将跳过所有相关元数据和数据。
ORA-39151: 表 "TEST"."city" 已存在。由于跳过了 table_exists_action, 将跳过所有相关元数据和数据。
处理对象类型 SCHEMA_EXPORT/TABLE/TABLE_DATA
. . 导入了 "TEST"."ZKN"                                5.078 KB       4 行
处理对象类型 SCHEMA_EXPORT/PROCEDURE/PROCEDURE
ORA-31684: 对象类型 PROCEDURE:"TEST"."XX" 已存在
ORA-31684: 对象类型 PROCEDURE:"TEST"."xx" 已存在
处理对象类型 SCHEMA_EXPORT/PROCEDURE/ALTER_PROCEDURE
ORA-39082: 对象类型 ALTER_PROCEDURE:"TEST"."xx" 已创建, 但带有编译警告
处理对象类型 SCHEMA_EXPORT/TABLE/STATISTICS/TABLE_STATISTICS
处理对象类型 SCHEMA_EXPORT/POST_SCHEMA/PROCACT_SCHEMA
作业 "TEST"."MY_JOB" 已经完成, 但是有 6 个错误 (于 14:18:16 完成)
```

以上是简单的导入导出工作。