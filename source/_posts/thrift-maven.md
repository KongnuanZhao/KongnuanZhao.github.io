title: Ubuntu搭建Thrift和Maven
date: 2015-03-18 21:14:47
author: 赵空暖
tags:
- hbase
- hadoop
categories:
- hbase
- hadoop
---
#关于thrift和maven简介#
* Thrift简介
目前流行的服务调用方式有很多种，例如基于 SOAP 消息格式的 Web Service，基于 JSON 消息格式的 RESTful 服务等。其中所用到的数据传输方式包括 XML，JSON 等，然而 XML 相对体积太大，传输效率低，JSON 体积较小，新颖，但还不够完善。由 Facebook 开发的远程服务调用框架 Apache Thrift，它采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk 等创建高效的、无缝的服务，其传输数据采用二进制格式，相对 XML 和 JSON 体积更小，对于高并发、大数据量和多语言的环境更有优势。

* Maven简介
是一个采用纯Java编写的开 源项目管理工具。Maven采用了一种被称之为project object model (POM)概念来管理项目，所有的项目配置信息都被定义在一个叫做POM.xml的文件中，通过该文件，Maven可以管理项目的整个声明周期，包括编 译，构建，测试，发布，报告等等。目前Apache下绝大多数项目都已经采用Maven进行管理。

#环境准备#
```bash
ubu@ubuntu:~$ uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```

```bash
ubu@ubuntu:~$ java -version
java version "1.7.0_75"
```

```bash
./hadoop version
ubu@ubuntu:~/java$ hadoop version
Hadoop 1.2.1
```
```bash
hbase版本
hbase-0.98.11
```

#Thrift安装#
Thrift下载 [链接在此](http://thrift.apache.org/download)
```bash
wget http://apache.fayea.com/thrift/0.9.2/thrift-0.9.2.tar.gz
tar -xvf thrift-0.9.2.tar.gz
~ mv thrift-0.9.2/ /home/ubu/modules/
~ cd /home/ubu/modules/thrift-0.9.2
```
* 安装Thrift的依赖包(如果希望支持PHP,Python,C++等语言的访问，就需要在系统中装一些额外的类库。可以根据自己的要求，安装对应的软件包并设置Thrift的编译参数。)
```bash
sudo apt-get install libboost-test-dev libboost-program-options-dev libevent-dev automake libtool flex bison pkg-config g++ libssl-dev
```
* 生成配置脚本
```bash
./configure
```
* 编译
```bash
make
make[3]: \D5\FD\D4\F8\C8\EB `/home/ubu/modules/thrift-0.9.2/tutorial/js'
make[3]: \D3\D0\BF\C9\D2\D4\D7\F6\B5\C4 `install-exec-am'\A1\A3
make[3]: \D3\D0\BF\C9\D2\D4\D7\F6\B5\C4 `install-data-am'\A1\A3
make[3]:\D5\FD\D4\DA\C0 `/home/ubu/modules/thrift-0.9.2/tutorial/js'
make[2]:\D5\FD\D4\DA\C0 `/home/ubu/modules/thrift-0.9.2/tutorial/js'
Making install in py
.......此处省略若干日志....
make[2]: \D5\FD\D4\F8\C8\EB `/home/ubu/modules/thrift-0.9.2'
make[2]: \D3\D0\BF\C9\D2\D4\D7\F6\B5\C4 `install-exec-am'\A1\A3
make[2]: \D3\D0\BF\C9\D2\D4\D7\F6\B5\C4 `install-data-am'\A1\A3
make[2]:\D5\FD\D4\DA\C0 `/home/ubu/modules/thrift-0.9.2'
make[1]:\D5\FD\D4\DA\C0 `/home/ubu/modules/thrift-0.9.2'
```
* 安装
```bash
sudo make install
BUILD SUCCESSFUL
Total time: 2 seconds
```
* 查看thrift版本
```bash
ubu@ubuntu:~/modules/thrift-0.9.2$ thrift -version
Thrift version 0.9.2
```
* 启动HBase的Thrift Server服务
```bash
ubu@ubuntu:~/modules/thrift-0.9.2$ hbase-daemon.sh start thrift
starting thrift, logging to /home/ubu/hbase/hbase-0.98.11-hadoop1/bin/../logs/hbase-ubu-thrift-ubuntu.out
ubu@ubuntu:~/modules/thrift-0.9.2$ jps
4558 Main
3080 NameNode
4268 HRegionServer
20406 Jps
3594 TaskTracker
3206 DataNode
3445 JobTracker
3362 SecondaryNameNode
4091 HMaster
20320 ThriftServer
4026 HQuorumPeer
```
ThriftServer已被启动，后面我们就可以使用多种语言通过Thrift来访问HBase了。
至此，完成了Thrift的安装。

#Maven#
Maven下载 [链接在此](http://maven.apache.org/download.cgi)
```bash
ubu@ubuntu:~/modules$ wget http://mirrors.cnnic.cn/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz
```
* 解压至/home/ubu/modules/，配置环境变量
```bash
ubu@ubuntu:~/modules/apache-maven-3.2.5$ export MAVEN_HOME=/home/ubu/modules/apache-maven-3.2.5
ubu@ubuntu:~/modules/apache-maven-3.2.5$ export PATH=$PATH:$MAVEN_HOME/bin
ubu@ubuntu:~/modules/apache-maven-3.2.5$ source /etc/profile
```
查看Maven版本
```bash
ubu@ubuntu:~/modules/apache-maven-3.2.5$ mvn -version
Apache Maven 3.2.5 (12a6b3acb947671f09b81f49094c53f426d8cea1; 2014-12-15T01:29:23+08:00)
Maven home: /home/ubu/modules/apache-maven-3.2.5
Java version: 1.7.0_75, vendor: Oracle Corporation
Java home: /home/ubu/java/jdk1.7.0_75/jre
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "3.13.0-32-generic", arch: "amd64", family: "unix"
```
至此，说明Maven安装完成。这些前期工作都是为后面项目做准备。
