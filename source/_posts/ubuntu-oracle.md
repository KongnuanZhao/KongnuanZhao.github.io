title: Linux上安装Oracle
date: 2015-03-17 21:46:17
author: 赵空暖
tags: oracle
categories: oracle
---
装Oracle的次数还蛮多的，记录一下Oracle的安装，方便自己回忆。
Oracle数据库10g下载 [链接在此](http://www.oracle.com/technetwork/cn/database/index-084376-zhs.html)
官网上说Oracle Database 10.2 and 11.1 are no longer available for download. 现在只能下Oracle 10g快捷版（Express Edition） 
ORACLE把10G的下载从官网拿掉了，这里oracle10g下载的链接方便大家下载[链接在此](http://blog.itpub.net/27006877/viewspace-739059/)
Oracle10g联机文档[Oracle Database Online Documentation 10g Release 2 (10.2)](https://docs.oracle.com/cd/B19306_01/nav/portal_2.htm)
关于Oracle10gEX介绍如下：
> Oracle 数据库 10g 快捷版（Oracle 数据库 XE）是一款基于 Oracle 数据库 10g 第 2 版代码库的小型入门级数据库，它具备以下优点：免费进行开发、部署和分发；下载速度快；并且管理简单。Oracle 数据库 XE 是一款优秀的入门级数据库，可供以下用户使用：
致力于 PHP、Java、.NET、XML 和开放源代码应用程序的开发人员
需要免费的入门级数据库进行培训和部署的 DBA
需要入门级数据库进行免费分发的独立软件供应商 (ISV) 和硬件供应商
需要在课程中使用免费数据库的教育机构和学生
现在，利用 Oracle 数据库 XE，您可以使用强大的、公认的、行业领先的基础架构来开发和部署应用程序，然后在必要时进行升级而不必进行昂贵和复杂的移植。
Oracle 数据库 XE 对安装到的主机的规模和 CPU 数量不作限制（每台计算机一个数据库），但 XE 将最多存储 4GB 的用户数据，最多使用 1GB 内存，并在主机上使用一个 CPU。

关于下载的介绍：
> database 是ORACLE数据库系统软件，一般只要这个包就可以了。
client是database包的精减包，不包括数据库，只包括客户端访问需要用的相关软件包，主要用于安装在客户端机器上。
gateways是指透明网关用的，如果要从oracle访问其它数据库系统(sqlserver,sybase...)则需要这个包了
clusterware是安装ORACLE集群(rac)时用的

Oracle数据库11g及以上版本下载 [链接在此](http://www.oracle.com/technetwork/database/enterprise-edition/downloads/index.html)
Oracle11g联机文档[Oracle Database Online Documentation 11g Release 2 (11.2)](http://docs.oracle.com/cd/E11882_01/nav/portal_11.htm)

参考文章[Linux Oracle10g安装](http://www.cnblogs.com/quanweiru/archive/2012/11/09/2762353.html)
照着官方文档和参考这篇文章开始漫漫装吧。