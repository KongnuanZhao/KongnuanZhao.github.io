title: 问题之yum一二三
date: 2015-04-02 21:01:53
author: 赵空暖
tags: linux
categories: linux
---
#yum和rpm是什么#
yum，是Yellow dog Updater, Modified 的简称。黄狗~挺有意思，贴一段百科解释：
> yum，是Yellow dog Updater, Modified 的简称，是杜克大学为了提高RPM软件包安装性而开发的一种软件包管理器。起初是由yellow dog 这一发行版的开发者Terra Soft 研发，用python 写成，那时还叫做yup(yellow dog updater)，后经杜克大学的Linux@Duke 开发团队进行改进，遂有此名。yum的宗旨是自动化地升级，安装/移除rpm 包，收集rpm 包的相关信息，检查依赖性并自动提示用户解决。yum 的关键之处是要有可靠的repository，顾名思义，这是软件的仓库，它可以是http 或ftp 站点，也可以是本地软件池，但必须包含rpm 的header，header 包括了rpm包的各种信息，包括描述，功能，提供的文件，依赖性等。正是收集了这些header并加以分析，才能自动化地完成余下的任务。

rpm,是RPM Package Manager（原Red Hat Package Manager，现在是一个递归缩写）类似Windows里面的“添加/删除程序”
> rpm是由红帽公司开发的软件包管理方式，使用rpm我们可以方便的进行软件的安装、查询、卸载、升级等工作。但是rpm软件包之间的依赖性问题往往会很繁琐,尤其是软件由多个rpm包组成时。

##区别：##
* .rpm是安装本地存在的rpm包，如果存在依赖也需要安装上,如果某个rpm是自己修改编译的，那么只能用rpm安装了
* yum是通过yum库（可以理解为一个软件库，网络上或者是本地），通过yum命令，让系统自动下载相关组件库及程序文件，可以新全新安装或者是升级安装（具体的多熟悉yum命令）

#问题#
前几天装Oracle安装一系列的依赖包，遇到yum 失败（This system is not registered with RHN.）,由于 redhat的yum在线更新是收费的，如果没有注册的话不能使用，如果要使用，需将redhat的yum卸载后，重启安装，再配置其他源，以下为详细过程：
1.卸载redhat自带的yum组件
```bash
[root@localhost ~]# rpm -qa|grep yum|xargs rpm -e --nodeps
```

2.安装CentOS的yum包
```bash
[root@localhost ~]# wget  http://centos.ustc.edu.cn/centos/5/os/i386/CentOS/yum-metadata-parser-1.1.2-3.el5.centos.i386.rpm
[root@localhost ~]# wget  http://centos.ustc.edu.cn/centos/5/os/i386/CentOS/yum-fastestmirror-1.1.16-16.el5.centos.noarch.rpm
[root@localhost ~]# wget  http://centos.ustc.edu.cn/centos/5/os/i386/CentOS/yum-3.2.22-37.el5.centos.noarch.rpm
[root@localhost ~]# rpm -ivh *.rpm
warning: yum-3.2.22-37.el5.centos.noarch.rpm: Header V3 DSA signature: NOKEY, key ID e8562897
Preparing...                ########################################### [100%]
   1:yum-metadata-parser    ########################################### [ 33%]
   2:yum-fastestmirror      ########################################### [ 67%]
   3:yum                    ########################################### [100%]
```
3. 下载更新源，并存放在系统目录中
4.生成缓存并进行安装
```bash
[root@localhost ~]# yum makecache
[root@localhost ~]# yum install xxx
```

最后了解一下为什么能够用CentOS的yum源替换redhat的：
在构成RHEL的大多数软件包中，都是基于GPL协议发布的，也就是我们常说的开源软件，正因为是这样，Red Hat公司也遵循这个协议，将构成RHEL的软件包公开发布，只要是遵循GPL协议，任何人都可以在原有的软件构成的基础上再开发和发布。CentOS就是这样在RHEL发布的基础上将RHEL的构成克隆再现的一个Linux发行版本。RHEL的克隆版本不只CentOS一个，还有White Box Enterprise Linux和TAO Linux 和Scientific Linux。

虽然说是RHEL的克隆，但并不是一模一样，所说的克隆是具有100%的互换性。但并不保障对应RHEL的软件在CentOS上面能也够100%的正常工作。并且安全漏洞的修正和软件包的升级对应RHEL的有偿服务和技术支持来说，数日数星期数个月的延迟情况也有。

redhat企业版若要适用yum源等于是适用了红帽的商业支持，需要付费注册。但 Red Hat Enterprise版和centOS从实质上说是一回事，只不过前者会获得redhat提供的商业服务。那么，我们只需要将 Red Hat Enterprise版中的yum配置成centOS的即可。

参考文章：
[yum失败解决方法](http://www.linuxidc.com/Linux/2012-06/63619.htm)
[CentOS yum 源的配置与使用](http://www.cnblogs.com/mchina/archive/2013/01/04/2842275.html)

(未完待续)