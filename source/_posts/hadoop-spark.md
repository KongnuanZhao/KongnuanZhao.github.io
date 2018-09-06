title: Hadoop和Spark集群搭建
date: 2016-12-13 10:57:39
author: 赵空暖
tags: 
- hadoop
- spark
categories:
- spark
- hadoop
---

#安装环境#
1) 三台PC机：
192.168.1.11 master
192.168.1.12  slave1
192.168.1.13  slave2
2) Linux版本：
ubuntu@ubuntu00:~/.ssh$ uname -a
Linux ubuntu00 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:08:14 UTC 2014 i686 i686 i686 GNU/Linux
3) 本实验所安装其它环境版本：
JDK：jdk8
Hadoop：hadoop2.7.3
Scala：scala-2.12.0
Spark：spark2.0.1

#安装步骤#
##准备工作##
* 检查三台机器连通性
![ping连通性](/image/ping.png)
* 安装openssh和配置SSH
```bash
sudo apt-get install openssh-server
```
192.168.1.11是主节点，切换到ubuntu用户
```bash
$su ubuntu
cd /home/ubuntu
$ssh-keygen -trsa
```
然后一直按回车,完成后，在home跟目录下会产生隐藏文件夹`.ssh`
```bash
$cd .ssh
```
之后ls查看文件
```bash
cp id_rsa.pub authorized_keys
```
测试localhost：
`ssh localhost`发现链接成功，并且无需密码。
复制到192.168.1.12/192.168.1.13上
```bash
$scp authorized_keys 192.168.1.12:/home/ubuntu/.ssh/
$scp authorized_keys 192.168.1.13:/home/ubuntu/.ssh/
```
测试
```bash
ssh 192.168.1.12和192.168.1.13
```
![ssh有效性](/image/ssh.png)

* 卸载原有JDK安装JDK8+

1)卸载自带的openjdk：
```bash
sudo apt-get remove openjdk*
```
2)下载jdk8,解压
```bash
tar -xzvf jdk-8u111-linux-i586.tar.gz
```
3)设置环境变量
```bash
sudo gedit /etc/profile
export JAVA_HOME=/usr/jdk1.8.0_111
CLASSPATH=$CLASSPATH.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
```
4)测试
```bash
ubuntu@ubuntu00:~/.ssh$ java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) Server VM (build 25.111-b14, mixed mode)
```
##安装Hadoop##
* 下载解压hadoop2.7.3

1)解压：
tar -zxvf hadoop-2.7.3.tar.gz
2)解压好之后配置环境变量`sudo gedit /etc/profile`继续追加以下内容：
```bash
export HADOOP_HOME=/home/ubuntu/hadoop-2.7.3
export PATH=$PATH:$HADOOP_HOME/bin
```
修改完之后`source /etc/profile`使之生效。
3)配置hadoop目录下的`/etc/hadoop/hadoop-env.sh`文件，在如下注释处添加：
```bash
# The java implementation to use.
export JAVA_HOME=/usr/jdk1.8.0_111/
```

* 集群配置

1)配置文件`hadoop-2.7.3/etc/hadoop/core-site.xml`添加如下内容:
```bash
<configuration>
<property>
 <name>fs.default.name</name>
  <value>hdfs://192.168.1.11:49000</value>
</property>
<property>
  <name>hadoop.tmp.dir</name>
 <value>/home/ubuntu/hadoop-2.7.3/var</value>
</property>
</configuration>
```
注：(1) `fs.default.name`是NameNode的URI。hdfs://主机名:端口/
    (2) `hadoop.tmp.dir `：Hadoop的默认临时路径，这个最好配置，如果在新增节点或者其他情况下莫名其妙的DataNode启动不了，就删除此文件中的tmp目录即可。不过如果删除了NameNode机器的此目录，那么就需要重新执行NameNode格式化的命令。
2)配置文件 `mapred-site.xml`
```bash
<configuration>
<property>
  <name>mapred.job.tracker</name>
  <value>192.168.1.11:49001</value>
</property>
<property>
  <name>mapred.local.dir</name>
 <value>/home/ubuntu/hadoop-2.7.3/var</value>
</property>
</configuration>
```
注：`mapred.job.tracker`是JobTracker的主机（或者IP）和端口。主机:端口。
3)配置文件`hdfs-site.xml`
```bash
<configuration>
<property>
<name>dfs.name.dir</name>
<value>/home/ubuntu/name1,/home/ubuntu/name2</value>
<description>  </description>
</property>
<property>
<name>dfs.data.dir</name>
<value>/home/ubuntu/data1,/home/ubuntu/data2</value>
<description> </description>
</property>
<property>
  <name>dfs.replication</name>
  <value>3</value>
</property>
</configuration>
```
注：(1)`dfs.name.dir`是NameNode持久存储名字空间及事务日志的本地文件系统路径。 当这个值是一个逗号分割的目录列表时，nametable数据将会被复制到所有目录中做冗余备份。
(2)`dfs.data.dir`是DataNode存放块数据的本地文件系统路径，逗号分割的列表。 当这个值是逗号分割的目录列表时，数据将被存储在所有目录下，通常分布在不同设备上。
(3)`dfs.replication`是数据需要备份的数量，默认是3，如果此数大于集群的机器数会出错。
(4)此处的name1、name2、data1、data2目录不能预先创建，hadoop格式化时会自动创建，如果预先创建反而会有问题。
4)配置`masters`和`slaves`主从结点
```bash
vi masters 
输入：192.168.1.11
vi slaves
输入：192.168.1.12
192.168.1.13
```
配置结束，把配置好的`hadoop-2.7.3`文件夹拷贝到其他集群的机器中，并且保证上面的配置对于其他机器而言正确:
```bash
$scp -r /home/ubuntu/hadoop-2.7.3 192.168.1.12: /home/ubuntu/
$scp -r /home/ubuntu/hadoop-2.7.3 192.168.1.13: /home/ubuntu/     
```       

* hadoop启动

1) 格式化一个新的分布式文件系统
```bash
$ bin/hadoop namenode -format
```
查看输出保证分布式文件系统格式化成功
执行完后可以到master机器上看到`/home/ubuntu/name1`和`/home/ubuntu/name2`两个目录。
![name12](/image/name12.png)
2)启动所有节点
$ sbin/start-all.sh
查看结果：
![name12](/image/00namenode.png)
![name12](/image/01datanode.png)
![name12](/image/03datenode.png)
执行完后可以到slave(192.168.1.12,192.168.1.13)机器上看到`/home/ubuntu/data1`和`/home/ubuntu/data2`两个目录。

##安装Scala##
* 下载scala-2.12.0.tgz
解压 `tar –zxvf scala-2.12.0.tgz`
* 配置环境变量
`/etc/profile`在下面添加路径：
```bash
export SCALA_HOME=/usr/scala-2.11.8
export PATH=$PATH:$SCALA_HOME/bin
```
使修改生效`source /etc/profile`
* 在命令行输入scala测试:
![scala](/image/00scala-test.png)
![scala](/image/01scala-test.png)
![scala](/image/03scala-test.png)

##安装Spark##
* 下载spark2.0.1
解压作个软链：
```bash
tar -zxvf spark-2.0.1-bin-hadoop2.7.tgz
ln -s spark-2.0.2-bin-hadoop2.7 spark2
```
* 配置环境变量
```bash
vi /etc/profile
#Spark 2.0.1
export SPARK_HOME=/home/ubuntu/spark2
export PATH=$PATH:$SPARK_HOME/bin
```
* 配置文件`/usr/local/spark2/conf`
1) spark-env.sh
```bash
cp -a spark-env.sh.template spark-env.sh
vi spark-env.sh
export JAVA_HOME=/usr/java/jdk1.8
export SPARK_MASTER_HOST=192.168.1.11
```
2)slaves
```bash
cp -a slaves.template slaves
vi slaves
192.168.1.12
192.168.1.13
```bash
* 复制到其他节点
```bash
scp -r spark-2.0.1-bin-hadoop2.7 192.168.1.12:/home/ubuntu/
scp -r spark-2.0.1-bin-hadoop2.7 192.168.1.13:/home/ubuntu/
```
4.5启动
```bash
$SPARK_HOME/sbin/start-all.sh
```
查看三个节点上的结果：
![00spark](/image/00spark.png)
![01spark](/image/01spark.png)
![02spark](/image/03spark.png)
* Web界面
http://192.168.1.11:8080/jobs/
http://192.168.1.11:4040/jobs/
![spark](/image/spark.png)

至此，搭建完成，之后在上面要做几个实验。







