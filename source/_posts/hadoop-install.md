title: Ubuntu搭建hadoop
date: 2015-03-17 20:42:04
author: 赵空暖
tags: Hadoop
categories: Hadoop
---
#Hadoop单机模式#
##安装准备##
* Ubuntu
```bash
ubu@ubuntu:~$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```
* JDK

sudo gedit ~/.bashrc （注：修改根目录下的bashrc文件，以便设置java环境变量）
在bashrc最后追加以下内容：
```bash
export JAVA_HOME=/usr/java/jdk1.7.0_45
export CLASSPATH=.:JAVAHOME/lib/dt.jar:JAVA_HOME/lib/tools.jar
export PATH=PATH:JAVA_HOME/bin
```
查看java版本以及环境变量
```bash
ubu@ubuntu:~$ java -version
java version "1.7.0_75"
OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-1~trusty1)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)

ubu@ubuntu:~$ echo $JAVA_HOME
/home/ubu/java/jdk1.7.0_75
```

##安装Hadoop##
* 创建hadoop目录
```bash
mkdir hadoop
```
* 下载以及解压hadoop
Hadoop下载[链接在此](http://www.apache.org/dyn/closer.cgi/hadoop/common/)
```bash
cd hadoop
sudo tar -zxf hadoop-1.2.1.tar.gz
```
* 修改hadoop配置文件
```bash
gedit hadoop-1.2.1/conf/hadoop-env.sh
#去掉注释，变设置成正确的路径，即：
export JAVA_HOME=/home/ubu/java/jdk1.7.0_75
```
* 验证hadoop是否安装完成
```bash
cd hadoop-1.2.1/bin
./hadoop version
ubu@ubuntu:~/java$ hadoop version
Hadoop 1.2.1
Subversion https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152
Compiled by mattf on Mon Jul 22 15:23:09 PDT 2013
From source with checksum 6923c86528809c4e7e6f493b6b413a9a
This command was run using /home/ubu/hadoop/hadoop-1.2.1/hadoop-core-1.2.1.jar 
```
至此hadoop单机模式安装已经安装完成，可以配置好HADOOP_HOME方面以后启动查看hadoop
`export PATH=$PATH:/home/ubu/hadoop/hadoop-1.2.1/bin`

#Hadoop伪分布模式配置#
* 修改`conf/core-site.xml`，执行命令`gedit core-site.xml`
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!-- Put site-specific property overrides in this file. -->
<configuration>
 <property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:9000</value>
 </property>
 <property>
  <name>hadoop.tmp.dir</name>
  <value>/home/ubu/hadoopdata/tmp</value>
 </property>
</configuration> 
```
* 修改`hdfs-site.xml`，配置数据备份(这是配置写数据时，数据同时写几份（出于学习目的，这里只写一个副本，实际应用中，至少配置成3）)
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
* 修改`mapred-site.xml`,配置map/reduce服务器ip和端口
```bash
<configuration>
    <property>
        <name>mapred.job.tracker</name>
        <value>localhost:9001</value>
    </property>
</configuration>
```
* 安装openssh-server `sudo apt-get install openssh-server`因为伪分布模式下，即使所有节点都在一台机器上，hadoop也需要通过ssh登录，这一步的目的是配置本机无密码ssh登录`ssh-keygen -t rsa`,然后一路回车。
```bash
cd ~/.ssh
cat id_rsa.pub>>authorized_keys
ssh localhost //测试一下
```
首次运行会提示是否继续，输入yes，回车，如果不要求输入密码，就表示成功了。
* 首次运行，格式化hdfs
```bash
ubu@ubuntu:~/hadoop/hadoop-1.2.1/bin$ ./hadoop namenode -format
15/03/14 21:46:52 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = ubuntu/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 1.2.1
STARTUP_MSG:   build = https://svn.apache.org/repos/asf/hadoop/common/branches/branch-1.2 -r 1503152; compiled by 'mattf' on Mon Jul 22 15:23:09 PDT 2013
STARTUP_MSG:   java = 1.7.0_75
************************************************************/
15/03/14 21:46:52 INFO util.GSet: Computing capacity for map BlocksMap
15/03/14 21:46:52 INFO util.GSet: VM type       = 64-bit
15/03/14 21:46:52 INFO util.GSet: 2.0% max memory = 932184064
15/03/14 21:46:52 INFO util.GSet: capacity      = 2^21 = 2097152 entries
15/03/14 21:46:52 INFO util.GSet: recommended=2097152, actual=2097152
15/03/14 21:46:53 INFO namenode.FSNamesystem: fsOwner=ubu
15/03/14 21:46:53 INFO namenode.FSNamesystem: supergroup=supergroup
15/03/14 21:46:53 INFO namenode.FSNamesystem: isPermissionEnabled=true
15/03/14 21:46:53 INFO namenode.FSNamesystem: dfs.block.invalidate.limit=100
15/03/14 21:46:53 INFO namenode.FSNamesystem: isAccessTokenEnabled=false accessKeyUpdateInterval=0 min(s), accessTokenLifetime=0 min(s)
15/03/14 21:46:53 INFO namenode.FSEditLog: dfs.namenode.edits.toleration.length = 0
15/03/14 21:46:53 INFO namenode.NameNode: Caching file names occuring more than 10 times 
15/03/14 21:46:53 INFO common.Storage: Image file /home/ubu/hadoopdata/tmp/dfs/name/current/fsimage of size 109 bytes saved in 0 seconds.
15/03/14 21:46:54 INFO namenode.FSEditLog: closing edit log: position=4, editlog=/home/ubu/hadoopdata/tmp/dfs/name/current/edits
15/03/14 21:46:54 INFO namenode.FSEditLog: close success: truncate to 4, editlog=/home/ubu/hadoopdata/tmp/dfs/name/current/edits
15/03/14 21:46:54 INFO common.Storage: Storage directory /home/ubu/hadoopdata/tmp/dfs/name has been successfully formatted.
15/03/14 21:46:54 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at ubuntu/127.0.1.1
************************************************************/

```
* 启动单节点集群
```bash
ubu@ubuntu:~/hadoop/hadoop-1.2.1/bin$ ./start-all.sh
starting namenode, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-namenode-ubuntu.out
localhost: starting datanode, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-datanode-ubuntu.out
localhost: starting secondarynamenode, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-secondarynamenode-ubuntu.out
starting jobtracker, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-jobtracker-ubuntu.out
localhost: starting tasktracker, logging to /home/ubu/hadoop/hadoop-1.2.1/libexec/../logs/hadoop-ubu-tasktracker-ubuntu.out
ubu@ubuntu:~/hadoop/hadoop-1.2.1/bin$ jps
3404 DataNode
3825 TaskTracker
3677 JobTracker
3916 Jps
3252 NameNode
3580 SecondaryNameNode

```
* 查看状态
http://localhost:50030/ 这是Hadoop管理界面
http://localhost:50060/ 这是Hadoop Task Tracker 状态
http://localhost:50070/ 这是Hadoop DFS 状态

至此，Hadoop伪分布模式配置完成。