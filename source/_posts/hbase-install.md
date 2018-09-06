title: Ubuntu中安装HBase
date: 2015-03-18 20:24:24
author: 赵空暖
tags: Hbase
categories: 
- Hbase
- nosql
---
#环境准备#
* Ubuntu
```bash
ubu@ubuntu:~$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

```bash
ubu@ubuntu:~$ java -version
java version "1.7.0_75"
OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-1~trusty1)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)
```

```bash
cd hadoop-1.2.1/bin
./hadoop version
ubu@ubuntu:~/java$ hadoop version
Hadoop 1.2.1
```
很多人都选择替换Hbase中的jar包，用{HADOOP_HOME}下的hadoop-1.2.1-core.jar替换掉{HBASE_HOME}/lib目录下的hadoop-1.2.1-append-r1056497.jar。如果不替换jar文件Hbase启动时会因为hadoop和Hbase的客户端协议不一致而导致HMaster启动异常。这里选择Hbase和Hadoop的版本对应除了找对应表格说明，或者替换jar之外，可以先下稳定版本的Hbase，然后看其lib下的hadoop-x.x.x-core.jar的hadoop的版本然后选择相应的hadoop版本下载安装。

#Hbase安装#
* 创建hbase目录
```bash
mkdir hbase
```
* 下载以及解压hbase
Hbase下载[下载链接](http://www.apache.org/dyn/closer.cgi/hbase/)，这里我安装的版本是hbase-0.98.11
```bash
cd hbase
ubu@ubuntu:~/hbase$ sudo tar -zxf hbase-0.98.11-bin.tar.gz
```
#配置HBase#
* 修改启动文件`hbase-env.sh`,执行命令`vi conf/hbase-env.sh`
```bash
#打开注释
export JAVA_HOME=/usr/java/jdk1.7.0_45
export HBASE_CLASSPATH=/home/ubu/hadoop/hadoop-1.2.1/conf
export HBASE_MANAGES_ZK=true
```
* 修改配置文件`hbase-site.xml`
```
<configuration>
<property>
<name>hbase.rootdir</name>
<value>hdfs://localhost:9000/hbase</value>
</property>

<property>
<name>hbase.cluster.distributed</name>
<value>true</value>
</property>

<property>
<name>dfs.replication</name>
<value>1</value>
</property>
</configuration>
```
* 复制hadoop环境的配置文件(类库不统一的话还需复制类库)
```bash
cp ~/hadoop/hadoop-1.2.1/conf/hdfs-site.xml conf/
```
* 启动hadoop和hbase
```bash
ubu@ubuntu:~$ start-all.sh
starting namenode, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-namenode-ubuntu.out
localhost: starting datanode, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-datanode-ubuntu.out
localhost: starting secondarynamenode, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-secondarynamenode-ubuntu.out
starting jobtracker, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-jobtracker-ubuntu.out
localhost: starting tasktracker, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-tasktracker-ubuntu.out
ubu@ubuntu:~$ jps
3080 NameNode
3706 Jps
3594 TaskTracker
3206 DataNode
3445 JobTracker
3362 SecondaryNameNode

ubu@ubuntu:~$ start-hbase.sh
localhost: starting zookeeper, logging to /home/ubu/hbase/hbase-0.98.11-hadoop1/bin/../logs/hbase-ubu-zookeeper-ubuntu.out
starting master, logging to /home/ubu/hbase/hbase-0.98.11-hadoop1/bin/../logs/hbase-ubu-master-ubuntu.out
localhost: starting regionserver, logging to /home/ubu/hbase/hbase-0.98.11-hadoop1/bin/../logs/hbase-ubu-regionserver-ubuntu.out
ubu@ubuntu:~$ jps
3080 NameNode
4348 Jps
4268 HRegionServer
3594 TaskTracker
3206 DataNode
3445 JobTracker
3362 SecondaryNameNode
4091 HMaster
4026 HQuorumPeer

```
* 打开HBase命令行客户端访问Hbase
```bash
ubu@ubuntu:~$ hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 0.98.11-hadoop1, r6e6cf74c1161035545d95921816121eb3a516fe0, Mon Mar  2 23:21:46 PST 2015

hbase(main):001:0> create 'student','info'
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/ubu/hbase/hbase-0.98.11-hadoop1/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/ubu/hadoop/hadoop-1.2.1/lib/slf4j-log4j12-1.4.3.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
0 row(s) in 1.1370 seconds
```
这个错误说是有两个slf4j-log4j12.jar删掉hbase下的一个就可以了。
```bash
hbase(main):003:0> create 'student','info'
0 row(s) in 0.4310 seconds

=> Hbase::Table - students
hbase(main):004:0> list
TABLE                                                                           
student                                                                                                     
1 row(s) in 0.0200 seconds

hbase(main):005:0> describe 'student'
Table student is ENABLED                                                        
student                                                                         
COLUMN FAMILIES DESCRIPTION                                                     
{NAME => 'info', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REPLICATIO
N_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0', TTL
 => 'FOREVER', KEEP_DELETED_CELLS => 'FALSE', BLOCKSIZE => '65536', IN_MEMORY =>
 'false', BLOCKCACHE => 'true'}                                                 
1 row(s) in 0.0530 seconds

hbase(main):008:0> put 'student','zhaokongnuan','info:sex','gril'
0 row(s) in 0.0170 seconds

hbase(main):009:0> get 'student','zhaokongnuan'
COLUMN                CELL                                                      
 info:sex             timestamp=1426506697772, value=gril                       
1 row(s) in 0.0130 seconds

```
* 配置Hbase环境变量
```bash
export HBASE_HOME=/home/ubu/hbase/hbase-0.98.11-hadoop1
export PATH=$PATH:$HBASE_HOME/bin
```
至此，HBase安装完成。