title: Oracle学习交流（二）--oracle体系结构及启动流程
date: 2015-01-03 21:16:19
author: 赵空暖
tags: oracle
categories: oracle
---
#Oracle体系结构#
首先回顾一下Oracle的体系结构
![oraclestruts](/image/oraclestruts.png)
每一个运行的Oracle 数据库都与一个Oracle 实例关联。在数据库服务器上启动数据库后，
Oracle 软件会分配一个称为系统全局区(SGA) 的共享内存区，还会启动若干个Oracle 后
台进程。这种SGA 和Oracle 进程的组合就称为一个Oracle 实例。
启动实例后，Oracle 软件会将实例与特定的数据库关联。这个过程称为装载数据库。接下
来可以打开数据库，以便授权用户访问数据库。在同一台计算机上可以并发执行多个实例，
每一个实例只访问自己的物理数据库。
这里要联系oracle 启动的三个过程,分为三个阶段：
1.启动实例，包括以下操作：1）读取参数文件  2）根据参数文件分配SGA  3）启动后台进程
![startora](/image/startora.png)
2.装载数据库 :根据读取参数文件获得控制文件的位置加载控制文件,让实例和数据库相关联
![mount](/image/mount.png)
3.打开数据库：根据控制文件找到并打开数据文件和日志文件，从而打开数据库
![open](/image/openora.png)
4.关闭数据库
![shutdown](/image/shutdown.png)
#物理数据结构#
这里说一下物理数据结构里面的参数文件也就是oracle启动实例的第一步要读取参数文件。
* 参数文件：存储ORACLE数据库的配置信息。记录控制文件的保存地址。
	* pfile：可编辑，位置：$ORACLE_HOME/database/init<SID>.ora
	* spfile：二进制文件，位置：$ORACLE_HOME/dbs/spfile<SID>.ora
简单看一下我本地的pfile`E:\app\Administrator\product\11.2.0\dbhome_1\database\INITorcl.ORA`
```bash
orcl.__db_cache_size=721420288
orcl.__java_pool_size=16777216
orcl.__large_pool_size=16777216
orcl.__oracle_base='E:\app\Administrator'#ORACLE_BASE set from environment
orcl.__pga_aggregate_target=889192448
orcl.__sga_target=1677721600
orcl.__shared_io_pool_size=0
orcl.__shared_pool_size=738197504
orcl.__streams_pool_size=33554432
*.audit_file_dest='E:\app\Administrator\admin\orcl\adump'
*.audit_trail='db'
*.compatible='11.2.0.0.0'
*.control_files='E:\APP\ADMINISTRATOR\ORADATA\ORCL\CONTROL01.CTL','E:\APP\ADMINISTRATOR\FLASH_RECOVERY_AREA\ORCL\CONTROL02.CTL'  #Restore Controlfile
*.db_16k_cache_size=117440512
*.db_block_size=8192 #一个 Oracle 数据库块的大小 (以字节计)。该值在创建数据库时设置，而且此后无法更改。
*.db_create_file_dest='E:\app\Administrator\oradata\orcl'
*.db_domain=''
*.db_name='orcl' #一个数据库标识符，应与CREATE DATABASE 语句中指定的名称相对应
*.db_recovery_file_dest='E:\app\Administrator\flash_recovery_area'
*.db_recovery_file_dest_size=10737418240
*.diagnostic_dest='E:\app\Administrator'
*.dispatchers='(PROTOCOL=TCP) (SERVICE=orclXDB)'
*.local_listener='LISTENER_ORCL'
*.log_archive_dest_1='LOCATION=E:\app\Administrator\flash_recovery_area'
*.log_archive_format='arch_%d_%t_%r_%s.log'
*.memory_target=2555379712
*.open_cursors=300 #库高速缓存 指定一个会话一次可以打开的游标 (环境区域) 的最大数量，并且限制 PL/SQL 使用的 PL/SQL 
#游标高速缓存的大小，以避免用户再次执行语句时重新进行语法分析。请将该值设置得足够高，这样才能防止应用程序耗尽打开的
#游标。              
*.processes=150
*.remote_login_passwordfile='EXCLUSIVE' #指定操作系统或一个文件是否检查具有权限的用户的口令。如果设置为 NONE，Oracle
#将忽略口令文件。如果设置为EXCLUSIVE，将使用数据库的口令文件对每个具有权限的用户进行验证。如果设置为SHARED，多个数据
#库将共享 SYS 和INTERNAL口令文件用户
*.undo_tablespace='UNDOTBS1'
```
ORACLE启动时，默认先读取spfile。
![spfile](/image/spfile.png)
##spfile和pfile的区别##
* 启动次序spfile优于pfile
* PFILE是静态文件，修改之后不会马上生效，数据库必须重新启动读取这个文件才行。
* SPFILE是动态参数文件,是二进制文件,不可以直接用记事本等等程序做修改,可以用ALTER命令做修改,不用重起数据库也能生效。

如何创建pfile和spfile，创建的时候，最好数据库处于关闭状态：
```bash
create pfile from spfile;
create spfile from pfile;
```

* 控制文件：存储数据库名称，数据文件位置，重做日志等信息。默认的控制文件位置$ORACLE_BASE/oradata/<SID>/controlxx.ctl
![control](/image/control.png)
控制文件很重要，所以ORACLE要求控制文件是多路复用的，也就是会有多个控制文件，但是每个控制文件的信息都是一样的。当一个控制文件损坏时，只需要将数据库停掉，然后把其他控制文件复制，重新命名后，重新启动数据库即可。

* 数据文件：含有存储在数据库中的信息。简单理解就是数据都记录在数据文件上，如果数据文件损坏，那么数据就会丢失，可以通过相关方法进行恢复。
查看表空间物理文件的名称，路径及大小
![data](/image/data.png)
* 联机日志文件：保存在数据库上进行的全部修改。一个数据库至少有两个联机日志文件，并且进行多路复用的设置。通过LGWR进程进行日志文件的写入。
日志文件以循环方式对日志进行写入。
当用户会话更新了高速缓冲区中的数据时，也会将最小的变化写入重做日志缓存区。LGWR不断的将重做日志缓冲区的转存到联机日志组中。当第一个日志组写满后，会自动切换到下一个日志组，当所有日志组写满后，会切换到第一个日志组，进行重写。
```bash
select * from v$logfile;  查询日志组状态和数量
alter system switch logfile;  手动切换日志组。
select tablespace_name, file_id,file_name, round(bytes/(1024*1024),0) total_space from dba_data_files order by tablespace_name;
```
* 密码文件：oracle数据库中拥有administrative权限的（即sysdba权限的）的用户登陆oracle数据库的其中一种方式，即这些用户可以通过oracle的密码文件来登陆数据库。
默认的密码文件地址：$ORACLE_HOME/database/pwd<SID>
* 归档日志文件：简单的说就是联机日志文件的备份。属于可选择项，一般情况下，测试环境中不会开启归档模式，在生产库中，为了保证数据可恢复性，都是开启归档模式。归档模式也有一定缺点，因为是保存先前的日志信息，所以很占用硬盘空间，所以管理员一般会做定期的处理，对归档日志进行删除工作，保留最新的归档日志文件。
![archive](/image/archive.png)
打开归档模式命令：首先需要将数据库关闭，然后启动到mount状态下，输入命令：`SQL>alter database archivelog;`
关闭归档模式命令：在mount状态下输入命令：`SQL>alter database noarchivelog; `
* 告警日志alert.log
告警日志，在产生错误时，启动和关闭实例时，都会记录信息到告警日志中，此外还记录了不同于默认值的初始参数的列表，alter system，alter database命令，对表空间，数据文件的操作，空间不足，损坏的文件等。告警日志也会变得很大，可在任意时间重命名或删除告警日志，但是告警日志记录了数据库的各种安全信息，维护和恢复等信息，因此可根据时间先后来选择性删除。

#Oracle内存结构#
![oraclecache](/image/oraclecache.png)
与Oracle 实例关联的基本内存结构包括：
* 系统全局区(SGA)：由所有服务器进程和后台进程共享
* 程序全局区(PGA)：专用于每一个服务器进程或后台进程。每一个进程使用一个PGA 。

SGA 包含以下数据结构：
* 数据库缓冲区高速缓存：缓存从数据库检索的数据块
* 重做日志缓冲区：高速缓存重做信息（用于实例恢复），直到可以将其写入磁盘中存储的物理重做日志文件
* 共享池：缓存可在用户间共享的各个结构
* 大型池：是一个可选区域，可为某些大型进程（如Oracle 备份和恢复操作、I/O 服务器进程）提供大量内存分配
* Java 池：用于Java 虚拟机(JVM) 中特定会话的所有Java 代码和数据
* Streams 池：由Oracle Streams 使用

PGA：是有用户连oracle时，oracle给开辟的一个内存区, 只供该用户使用,该用户断开后oracle就会将这块内存回收
![sga](/image/sga.png)
五大重要进程：
* DBWR：数据库写入进程。会话只更新数据库高速缓存区中的数据，所有更新有DBWR负责将数据写入磁盘中。例如执行commit命令后，数据被写入到磁盘等。
* LGWR：日志写入进程。将数据库高速缓冲区中数据的变化写入重做日志文件。这个写入的是尽可能接近实时完成比DBWR要快。
* PMON：进程监视进程。管理用户会话，负责监视数据库的处理，负责清除死掉的进程等。
* SMON：系统监视进程。负责在实例启动时恢复实例。恢复因系统崩溃而中断的事务。
* CKPT：检查点进程。向DBWR进程发送信号，执行一次检查点，更新数据库的所有数据和控制文件。

#逻辑数据结构#
* 表空间
![tablespace](/image/tablespace.png)
* 段:在Oracle中，凡是分配了空间的对象，都称之为段。
![segment](/image/segment.png)
* 区
![extent](/image/extent.png)
* 数据块：最小的逻辑单位，默认是8K，可以设置为16K，32K等。
![datablock](/image/datablock.png)

* 表空间
在数据库系统中，存储空间是较为重要的资源，合理利用空间，不但能节省空间，还可以提高系统的效率和工作性能。Oracle 可以存放海量数据，所有数据都在数据文件中存储。而数据文件大小受操作系统限制，并且过大的数据文件对数据的存取性能影响非常大。同时Oracle 是跨平台的数据库，Oracle 数据可以轻松的在不同平台上移植，那么如何才能提供统一存取格式的大容量呢？Oracle 采用表空间来解决。
表空间只是一个逻辑概念，若干操作系统文件（文件可以不是很大）可以组成一个表空间。表空间统一管理空间中的数据文件，一个数据文件只能属于一个表空间。一个数据库空间由若干个表空间组成。如图所示：
![tablespace1](/image/tablespace1.png)
Oracle 中所有的数据（包括系统数据），全部保存在表空间中，常见的表空间有：
* 系统表空间存放系统数据，系统表空间在数据库创建时创建。表空间名称为SYSTEM。存放数据字典和视图以及数据库结构等重要系统数据信息，在运行时如
果 SYSTEM 空间不足，对数据库影响会比较大，虽然在系统运行过程中可以通过命令扩充空间，但还是会影响数据库的性能，因此有必要在创建数据库时适当的把数
据文件设置大一些。
* 临时表空间:临时表空间，安装数据库时创建，可以在运行时通过命令增大临时表空间。临时表空间的重要作用是数据排序。比如当用户执行了诸如 Order by 等命令后，服务器需要对所选取数据进行排序，如果数据很大，内存的排序区可能装不下太大数据，就需要把一些中间的排序结果写在硬盘的临时表空间中。
* 用户自定义表空间，用户可以通过`CREATE TABLESPACE`命令创建表空间。
* undo表空间：回滚表空间，用于存放 undo数据，当执行DML操作(INSERT，UPDATE、DELETE)时，oracle会将这些操作的旧数据写入到 undo段。
![tablespace2](/image/tablespace2.png)

最后说一下上次遇到的一个问题：
db_recovery_file空间不够
当我们将数据库的模式修改为归档模式的时候，如果没有指定归档目录，默认的归档文件就会放到FlashRecovery Area的目录（这个位置在参数文件中有记录），但是这个目录是有大小限制的，如果超过了这个大小，就会导致2个问题，一是不能完成归档，二是，在出现问题后，如果此时重启数据库，那么数据库就法正常启动。
这个问题有3个解决办法
（1）扩大Flash Recovery Area的容量（治标不治本）
```bash
#查看Flash Recovery Area空间的使用状态
SQL> show parameter db_recovery_file;
SQL> alter system set db_recovery_file_dest_size=3G scope=both; #增加容量    
```
（2）删除不用的归档日志文件
```bash
进入RMAN
　　rman
　　RMAN> connect target /
　　RMAN> crosscheck archivelog all;
　　RMAN> delete expired archivelog all;
　　RMAN>exit;
```
（3）指定归档日志文件到其他目录（推荐）
```bash
alter system set log_archive_dest_1='location=/db/oracle/oradata/archive_log'
```


