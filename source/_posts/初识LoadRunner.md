title: 初识LoadRunner
date: 2015-01-15 15:10:17
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
##LoadRunner初相识##
* 安装（略）
![LoadRunner](/image/LoadRunner.png)
* LoadRunner基本组成
![hploadrunner](/image/hploadrunner.png)
	* Virtual User Generator：录制最终用户业务流程并创建自动化测试脚本，即Vuser脚本。
	* Controller：组织、驱动、管理并监控负载测试。
	* Analysis：用于查看、剖析和比较性能结果。
	* Load Generator：通过运行 Vuser 产生负载。

![LR](/image/LR.png)

##LoadRunner初体验之录制脚本##
* 录制的原理
	* 和fiddler一样还是代理
	* 将客户端和服务器当时说的话（消息、协议）拦截到，以LoadRunner的方式展现。
	![load2](/image/load2.png)
* 协议的选择
![newvirtualuser](/image/newvirtualuser.png)
	* 如何选择？
		* 问开发等项目组相关人员
		* 自身的经验
		* 看项目文档分析
		* 使用协议探测器
	* 选择协议，选择的是与压力模拟直接交互的中间件或者服务器，和后面的无关。
	* LoadRunner录制只与协议有关，和操作界面关系不大。
	![load](/image/load.png)
* 界面及结构解析
	* Vuser界面简单解析（略）
	* 保存目录解析
	* 脚本结构解析
* 两种录制模式的解析
	* HTML-based script和URL-based script的区别
	* HTML-based script下的两种模式
		* A script describing user actions
		* A script containing explicit URLs only
* 模式的选择
选择哪种方式录制，有以下参考原则：
	* 基于浏览器的应用程序推荐使用HTML-based Script
	* 不是基于浏览器的应用程序推荐使用URL-based Script
	* 如果基于浏览器的应用程序中包含了JavaScript 并且该脚本向服务器产生了请求，比如DataGrid的分页按钮等，flash等，也要使用URL-based 方式录制
	* 基于浏览器的应用程序中使用了HTTPS 安全协议，使用URL-based 方式录制

下面的这段引用来自用户指导手册
> 基于HTML 的脚本：此选项为Web (HTTP/HTML) Vuser 的默认录制级别。它指示VuGen 在当前网页的上下文中录制HTML 操作。它在录制会话期间不会录制所有资源，而是在回放期间下载这些资源。建议对带有小程序和VB 脚本的浏览器应用程序使用此选项。
基于URL 的脚本：录制来自服务器的所有请求和资源。它将每个HTTP 资源自动作为URL 步骤 （web_url 语句）进行录制，将表单作为web_submit_data 进行录制。它不生成 web_link、web_image 和 web_submit_form 函数，也不录制框架。建议对非浏览器应用程序使用此选项。

* 运行时设置解析
	* Think Time(思考时间)
	* Run Logic(Action执行顺序和迭代)
	* Browser Emulation(模拟浏览器缓存)

#课后作业#
<b>1.描述使用LoadRunner分别基于HTML-based script和URL-based script方式录制的区别和各自的使用场景。</b>
分别使用HTML-based script和URL-based script录制，截取一小部分
HTML-based script
```bash
web_url("www.baidu.com", 
		"URL=http://www.baidu.com/", 
		"Resource=0", 
		"RecContentType=text/html", 
		"Referer=", 
		"Snapshot=t36.inf", 
		"Mode=HTML", 
		EXTRARES, 
		"Url=http://p0.qhimg.com/d/webidnameplate/4e323a1ec97c6d0eabaf3952cbe376c5.png", "Referer=", ENDITEM, 
		"Url=http://suggestion.baidu.com/su?wd=&json=1&p=3&sid=8555_1459_11085_10873_11111_11066_10922_11150_10700_10991_10618&req=2&cb=jQuery110209753620238043368_1421395272889&_=1421395272890", ENDITEM, 
		"Url=/cache/fpid/chromelib_1_1.js?_=1421395272891", ENDITEM, 
		"Url=http://dj1.baidu.com/v.gif?mod=superplus%3Aweather&submod=weather_tpl&utype=&superver=superplus&portrait=c978b6b5c0ef45b8f6ccc7bb06&glogid=4027434301&type=2011&pid=315&version=PCHome&terminal=PC&qid=0xc049195f0000585b&sid=8555_1459_11085_10873_11111_11066_10922_11150_10700_10991_10618&super_frm=&from_login=&from_reg=&query=&curcard=2&curcardtab=&_r=0.8888055183924735&m=superplus%3Aweather_weatherShow&logactid=5000000000&showType=weatherMod", ENDITEM, 
		LAST);
```
URL-based script
```bash
web_url("4e323a1ec97c6d0eabaf3952cbe376c5.png", 
		"URL=http://p0.qhimg.com/d/webidnameplate/4e323a1ec97c6d0eabaf3952cbe376c5.png", 
		"Resource=1", 
		"RecContentType=image/png", 
		"Referer=", 
		"Snapshot=t46.inf", 
		LAST);
web_url("su", 
		"URL=http://suggestion.baidu.com/su?wd=&json=1&p=3&sid=8555_1459_11085_10873_11111_11066_10922_11150_10700_10991_10618&req=2&cb=jQuery110207037915396504104_1421395381446&_=1421395381447", 
		"Resource=1", 
		"RecContentType=text/javascript", 
		"Referer=http://www.baidu.com/", 
		"Snapshot=t50.inf", 
		LAST);
web_url("v.gif", 
		"URL=http://dj1.baidu.com/v.gif?mod=superplus%3Aweather&submod=weather_tpl&utype=&superver=superplus&portrait=c978b6b5c0ef45b8f6ccc7bb06&glogid=4038189714&type=2011&pid=315&version=PCHome&terminal=PC&qid=0x8d8956a9000066c7&sid=8555_1459_11085_10873_11111_11066_10922_11150_10700_10991_10618&super_frm=&from_login=&from_reg=&query=&curcard=2&curcardtab=&_r=0.8510286659002304&m=superplus%3Aweather_weatherShow&logactid=5000000000&showType=weatherMod", 
		"Resource=1", 
		"RecContentType=image/gif", 
		"Referer=http://www.baidu.com/", 
		LAST);
```
1.可以看出HTML-based script是将一个页面的多次请求URL放在一个web_url当中，而URL-based script则是将每条客户端发出的请求单独为一个web_url.
下面的这段引用来自用户指导手册
> 基于HTML 的脚本：此选项为Web (HTTP/HTML) Vuser 的默认录制级别。它指示VuGen 在当前网页的上下文中录制HTML 操作。它在录制会话期间不会录制所有资源，而是在回放期间下载这些资源。建议对带有小程序和VB 脚本的浏览器应用程序使用此选项。
基于URL 的脚本：录制来自服务器的所有请求和资源。它将每个HTTP 资源自动作为URL 步骤 （web_url 语句）进行录制，将表单作为web_submit_data 进行录制。它不生成 web_link、web_image 和 web_submit_form 函数，也不录制框架。建议对非浏览器应用程序使用此选项。

2.HTML-based script方式录制一般是web_submit_form
```bash
	web_submit_form("login.pl", 
		"Snapshot=t2.inf", 
		ITEMDATA, 
		"Name=username", "Value=jojo", ENDITEM, 
		"Name=password", "Value=bean", ENDITEM, 
		LAST);
```
URL-based script方式录制一般是web_submit_data
```bash
web_submit_data("login.pl", 
		"Action=http://127.0.0.1:1080/WebTours/login.pl", 
		"Method=POST", 
		"RecContentType=text/html", 
		"Referer=http://127.0.0.1:1080/WebTours/nav.pl?in=home", 
		"Snapshot=t28.inf", 
		"Mode=HTTP", 
		ITEMDATA, 
		"Name=userSession", "Value=115148.735034427fQccQzDpAfiDDDDDDffDHpHHiDf", ENDITEM, 
		"Name=username", "Value=jojo", ENDITEM, 
		"Name=password", "Value=bean", ENDITEM, 
		"Name=login.x", "Value=67", ENDITEM, 
		"Name=login.y", "Value=1", ENDITEM, 
		"Name=JSFormSubmit", "Value=off", ENDITEM, 
		LAST);
```
后者更全活些，无需从cache中取。
关于web_submit_data和web_submit_form，HTML-based script也可以出现web_submit_data,可以在高级选项中设置。但是URL-based script一定不会出现web_submit_form。
选择哪种方式录制，有以下参考原则：
* 基于浏览器的应用程序推荐使用HTML-based Script
* 不是基于浏览器的应用程序推荐使用URL-based Script
* 如果基于浏览器的应用程序中包含了JavaScript 并且该脚本向服务器产生了请求，比如DataGrid的分页按钮等，flash等，也要使用URL-based 方式录制
* 基于浏览器的应用程序中使用了HTTPS 安全协议，使用URL-based 方式录制

<b>2.录制LoadRunner安装后自带的订票网站的打开首页和登录操作，录制在action中，录制生成脚本后，在登录前添加等待3秒的思考时间，同时，在登录函数后面添加lr_log_message函数，使用这个函数打印出自己姓名的全拼，将该录制脚本保存目录中的action.c文件作为附件提交作为作业。</b>
见附件（略）

<b>3.HTTP协议中的If-Modified-Since请求头的含义是什么？LoadRunner中通过什么方式使得请求中包括If-Modified-Since请求头？</b>
If-Modified-Since,从上次请求之后是否修改，即最后的修改时间。不需要每次都去服务器请求完整的资源，只需要看If-Modified-Since得知最后是什么时间修改的，然后对比缓存中的，如果没有修改就直接从缓存中读取。
![newer](/image/newer.png)

<b>4.使用LoadRunner创建一个HTTP协议的空脚本，修改脚本，保证脚本中有两个action,分别叫Action1和Action2,通过修改运行时设置中的配置，需要达到在运行脚本时，50%的概率运行Action1，50%的概率运行Action2。将该脚本保存后打包成压缩包作为附件形式提交作为作业。<b>
循环10次 ，各占50%
![action](/image/action.png)
见附件（略）




---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^(俺不是打广告滴……)

