title: DNS学习手记
date: 2015-01-02 11:28:15
author: 赵空暖
tags:
- 计算机网络
categories:
- 计算机网络
---
#物理层#

#域名解析#
域名系统等级结构解析流程
![dns](/image/dns.jpg)
* 假如没有自己的本地DNS服务器，就要找DNS提供商的主机如电信、网通。
* 第一步是ns找自己区域内的DNS服务器，如果没有ns就去找根服务器，然后根让ns找com,com让ns去找163.com
* 加入直接在自己的本地DNS服务器直接找到即直接返回IP则为权威答案。如此请求分为两种：
	* 递归请求：即只需发出一次请求，直接返回IP
	* 迭代查询： 发出n次请求返回IP
* 当然因为会发出多次请求，所以DNS要缓存，当发出www.163.com先找比如浏览器缓存，然后找hosts……
以[DNSPod](https://www.dnspod.cn/)为例, Time to live 缓存的生存时间。指地方dns缓存域名记录信息的时间，缓存失效后会再次到DNSPod获取记录值。
![TTL](/image/TTL.png)

以上为DNS的正向解析，一般而言，物必有反，所以DNS的反向解析也是理所应当存在的。<b>反向解析的作用</b>
解析文件中记录着 域名及其对应的ip 如 www.163.com 对应 1.1.1.1，则反向解析则记录着  1.1.1.1 对应www.163.com。那么计算机是如何识别此记录(record)是正向的还是反向的呢？区别记录的为record.type,record.type有如下几种:
以[DNSPod](https://www.dnspod.cn/)为例,	
![dnspod](/image/dnspod.png)
其中，
* A记录：（ADDRESS）地址记录，也就是www.zhaokongnuan.com 对应记录值 1.1.1.1,那么在github上搭建博客，使用github提供的空间，A记录怎么做呢？
	* 1.获取github pages ip ![2](/image/2.png)
	* 2.到dnspod添加设置两条主机记录分别是@和www，记录类型为A，ip地址填上个步骤获取的IP地址。![3](/image/3.png)
* CNAME: （Canonical name）正式名称，权威名称。例如有一台计算机名为 "jsj"(A记录)，它同时提供www和mail服务，为了便于用户访问服务，可以为该计算机设置两个别名(CNAME):WWW和MAIL，这两个别名的全称就是www开头的域名和以mail开头的域名。实际上他们都指向"jsj"。 同样的方法可以用于当您拥有多个域名需要指向同一服务器IP，此时就可以将一个域名做A记录指向服务器IP然后将其他的域名做别名到之前做A记录的域名上，那么当您的服务器IP地址变更时您就可以不必麻烦的一个一个域名更改指向了 只需要更改做A记录的那个域名其他做别名的那些域名的指向也将自动更改到新的IP地址上了。
(注：我们知道用hexo和github搭建自己的博客，一个很重要的原因是因为github允许自己绑定域名，只需在Hexo init根目录下source目录下的CNAME文件中写入如www.zhaokongnuan.com,Github服务器会设置www.zhaokongnuan.com为你的主域名，然后将zhaokongnuan.github.com重定向到zhaokongnuan.com)
* SOA：(Start of Autority)起始授权结构，此记录指定区域的起点。它所包含的信息有区域名、区域管理员电子邮件地址，以及指示辅 DNS 服务器如何更新区域数据文件的设置等。
* NS：(name server)，名称服务器，此记录指定负责此DNS区域的权威名称服务器。
* MX：(mail exchange)，邮件交换器，此记录列出了负责接收发到域中的电子邮件的主机 ，通常用于邮件的收发。MX是有优先级的0~99，如上图，数字越小优先级越高，邮件来的时候数字越小的越先处理。

另外还有
* P记录，PTR ,POINTER指针记录常被用于反向地址解析，主要应用在电子邮件系统中用于邮件交换。

#附#
关于[DNSPod](https://www.dnspod.cn/)官网
> DNSPod 建立于2006年3月份，是一款免费智能DNS产品。 DNSPod 可以为同时有电信、网通、教育网服务器的网站提供智能的解析，让电信用户访问电信的服务器，网通的用户访问网通的服务器，教育网的用户访问教育网的服务器，达到互联互通的效果。DNSPod是中国第一大DNS解析服务提供商、第一大域名托管商。它除了实时生效、不限制用户添加的域名和记录数量、提供URL转发、搜索引擎优化、域名共享管理、域名锁定、IPv6的支持、动态域名解析、API接口、批量修改管理等先进功能外，还拥有：云DNS、DNSPod DNS Protector（DNSPod 自主研发的DNS 防护软件）、宕机监控、安全中心、7*24小时专业技术支持。并且所有功能都是免费向所有用户提供。

买了国外的域名和空间之后要做的就是把域名解析到国内，这个用什么解析呢？这里就用到了dnspod，一个国内的免费的域名解析。注册账号添加域名之后dnspod会给我们两个国内的解析地址，如下图
![dnspod1](/image/dnspod1.png)
这两个地址是你要在你的域名上绑定的，这一步你要打开你的域名商给你提供的后台，里面有解析到别的地址，比如说[godaddy](https://www.godaddy.com/?isc=bsfndom4&ci=90231)，进入我的设置然后如下图,设置域名的DNS,在相应域名的Custom DNS里，设置DNS service,添加两条记录即dnspod提供给我们的f1g1ns1.dnspod.net和f1g1ns2.dnspod.net
![nameserver](/image/nameserver.png)
最后等待一段时间之后ping一下自己的设置对不对
![4](/image/4.png)

关于github(https://github.com/)
> GitHub 是一个用于使用Git版本控制系统的项目的基于互联网的存取服务,Github虽然是一个代码仓库，但是Github还免费为大家提供一个免费开源Github Pages空间，利用这个空间可以搭建轻量级的博客系统，绑定自己的域名，存放一些图片和文件等等。
想要加快博客访问的速度 ，可以移步这篇文章[hexo博客同步到github和gitcafe](http://www.zhaokongnuan.com/2015/03/01/github-gitcafe)