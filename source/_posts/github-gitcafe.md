title: hexo博客同步到github和gitcafe
date: 2015-03-01 18:54:34
author: 赵空暖
tags:
- github
- hexo
- gitcafe
categories: hexo
---
![sad](/image/sad.jpg)
最近几天有几位同学反应自己发的链接打不开，看了一下百度站长统计的数据，错误率虽然不说多吧，但最近几天也不少
![error](/image/error.png)
所以下午的时候就花时间解决了一下O(∩_∩)O~
前段时间花了点时间用hexo把博客搭在了github上，爱我大中华访问国外的服务器有时候真令人捉急。好在大中华的学习能力是超强的^_^然后有了中国版的[github](https://github.com)--[gitcafe](https://gitcafe.com)。
#关于gitcafe#
[百科介绍](http://baike.baidu.com/link?url=JH5kYao_-dc2zbhZ6qyaL5pBkHv4xJjOux2vWrhGAl90yDPNtQl37KPUpjORrEoCliJYplD4m5hvQ4GlrIc1NK)：
> GitCafe是一个基于代码托管服务打造的技术协作与分享平台，程序开发爱好者们可以通过使用代码版本控制系统git来将他们所写的开源或商业项目的代码托管在GitCafe上，与其他程序员针对这些项目在线协作开发。

为了解决国内访问慢的问题采取同步github和gitcafe的方案。有以下优点：
* 通过DNSPod自定义线路类型，使电信的走电信的，网通的走网通的，国内的走Gitcafe，国外的走Github,速度大增。（ DNSPod 可以为同时有电信、网通、教育网服务器的网站提供智能的解析，让电信用户访问电信的服务器，网通的用户访问网通的服务器，教育网的用户访问教育网的服务器，达到互联互通的效果。）
* 多机备份

注册好了Gitcafe账号，[如何安装和设置 Git](https://gitcafe.com/GitCafe/Help/wiki/%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85%E5%92%8C%E8%AE%BE%E7%BD%AE%20Git)。
测试连接成功后，新建项目，项目名要和用户名一致，然后自定义好域名。
之后需要修改hexo下的`_config.yml`,不同版本可能写法不一样，我的hexo版本是
```bash
$ hexo version
hexo: 2.8.3
os: Windows_NT 6.1.7601 win32 x64
http_parser: 1.0
node: 0.10.33
v8: 3.14.5.9
ares: 1.9.0-DEV
uv: 0.10.29
zlib: 1.2.3
modules: 11
openssl: 1.0.1j
```
修改方式：
```bash
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: github
  repo: git@github.com:KongnuanZhao/KongnuanZhao.github.io.git
  type: github
  repo: git@gitcafe.com:KongnuanZhao/KongnuanZhao.git
  branch: gitcafe-pages
```

然后在DNSPod里面设置好
![podhubcafe](/image/podhubcafe.png)

大约等个几分钟就生效了，来对比一下前后的速度~
首先分别单独访问github Pages和gitcafe Pages
![timecompare](/image/timecompare.png)
可以看到明显 从gitcafe访问较快。
没在DNSpod启用所有gitcafe的记录之前，`ping www.zhaokongnuan.com`
![github](/image/github.png)
看到走的是github服务器。
在DNSpod添加上gitcafe服务器的ip之后，几分钟后再ping一下，看到这次访问走的是gitcafe，速度如下
![gitcafeserver](/image/gitcafeserver.png)
[站长工具](http://ping.chinaz.com/)ping一下，国外访问果然快
![chinaz](/image/chinaz.png)
棒棒哒！