title: LoadRunner中的关联和检查点
date: 2015-02-21 20:03:26
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
##目的##
目的：掌握LoadRunner中的关联的基本概念和使用方法

##概念之关联##
* 关联的定义
	因为LoadRunner录制的时候SessionID是写死的，会话结束后该SessionID便会失效，脚本回访就会报错，所以我们需要对类似SessionID从服务器返回动态变化的值进行关联。所以在脚本回放过程中，客户端发出请求，通过关联函数所定义的左右边界值，在服务器所响应的内容中查找，得到相应的值，以变量的形式替换录制时的静态值，从而向服务器发出正确的请求，这种动态获得服务器响应内容的方法被称作关联。
* 关联和参数化的区别
	参数化是在客户端可以预先定义的，一般来自于一个文件、一个定义的table、通过sql写的一个结果集等，而关联是从服务器获取动态变化的，如SessionID.
* 什么时候需要关联
	服务器返回的动态变化且对业务有影响的.
* 举个栗子
首次，关联的方法
![correlation](/image/correlation.png)
```c
//Correlation comment - Do not change! Original value='115392.831834751fQDffVipHiHfDfzVDpVVcHcf' Name ='CorrelationParameter_1'

	web_reg_save_param_ex(
		"ParamName=CorrelationParameter_1",
		"LB=userSession value=",
		"RB=>\n<table border",
		SEARCH_FILTERS,
		"Scope=All",
		"IgnoreRedirections=Yes",
		"RequestUrl=*/nav.pl*",
		LAST);
……
	lr_think_time(5);

	web_submit_data("login.pl",
		"Action=http://127.0.0.1:1080/WebTours/login.pl",
		"Method=POST",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Snapshot=t13.inf",
		"Mode=HTTP",
		ITEMDATA,
		"Name=userSession", "Value={CorrelationParameter_1}", ENDITEM,
		"Name=username", "Value=jojo", ENDITEM,
		"Name=password", "Value=bean", ENDITEM,
		"Name=login.x", "Value=50", ENDITEM,
		"Name=login.y", "Value=12", ENDITEM,
		"Name=JSFormSubmit", "Value=off", ENDITEM,
		LAST);

```
* 注：`web_reg_save_param_ex`这个函数是通过左右边界来取出相应的值，如`SessionID`的`value`是`115392.831834751fQDffVipHiHfDfzVDpVVcHcf`要从服务器返回的HTML代码中取出`<input type=hidden name=userSession value=115392.929688133fQDffcDpicAiDDDDDfzVDpVtcQcf>` 它的左边LB是`userSession value=`,右边RB为`>`
* `web_reg_save_param`用于web_submit_form.用法：`web_reg_save_param("outFlightVal", "LB=outboundFlight value=", "RB= checked >", LAST ); `
* Scope 的值可以为ALL、HEADER、Body、Cookies 是值搜索的范围
![Scope](/image/zhuabao.png)

##概念补充之集合点##
* 集合点的概念
	集合点的意思时等到特定的用户数后再一起执行某个操作，比如一起保存，一起提交。比如说某个促销品的促销时间在8点到8点30，这样的话，就可能出现在8点时很多人一起提交的场景，这个时候需要用到集合点。
* 解析集合点函数 `lr_rendezvous("test")`
* 只能在Action中添加集合点
* 一般不要在事务中添加集合点，会拉长事务完成的时间。

##概念补充之检查点##
* 检查点的概念
	至于为什么要用检查点可以用个小例子做个测试，例如一个登陆脚本登陆的账号为jojo，密码为jojo，可以正确登陆，当把账号或密码改掉再执行，发现脚本并没有报错，也顺利执行下来了。原因是什么呢 ？Loadrunner以用户角色向服务器发送一个登陆请求，却不会判断请求的返回消息是什么，只要有返回，即使这是个拒绝登陆的返回，Loadrunner也认为登陆成功了。所以在登录或者其他有重要页面跳转的地方，很有必要做检查点。
* 实例
	使用上面关联的例子：
```c
Action()
{	

//Correlation comment - Do not change! Original value='115393.197731875fQDffDHpzAiDDDDDDfzVDpccfzf' Name ='CorrelationParameter_1'

	web_reg_save_param_ex(
		"ParamName=CorrelationParameter_1",
		"LB=userSession value=",
		"RB=>\n<table border",
		SEARCH_FILTERS,
		"Scope=All",
		"IgnoreRedirections=Yes",
		"RequestUrl=*/nav.pl*",
		LAST);

	web_url("nav.pl",
		"URL=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/welcome.pl?signOff=true",
		"Snapshot=t22.inf",
		"Mode=HTTP",
		LAST);

	web_submit_data("login.pl",
		"Action=http://127.0.0.1:1080/WebTours/login.pl",
		"Method=POST",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Snapshot=t23.inf",
		"Mode=HTTP",
		ITEMDATA,
		"Name=userSession", "Value={CorrelationParameter_1}", ENDITEM,
		"Name=username", "Value=jojo", ENDITEM,
		"Name=password", "Value=bean", ENDITEM,
		"Name=login.x", "Value=65", ENDITEM,
		"Name=login.y", "Value=8", ENDITEM,
		"Name=JSFormSubmit", "Value=off", ENDITEM,
		LAST);


	web_url("login.pl_2",
		"URL=http://127.0.0.1:1080/WebTours/login.pl?intro=true",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/login.pl",
		"Snapshot=t24.inf",
		"Mode=HTTP",
		LAST);
return 0;
}

```
输出结果：
```bash
Running Vuser...
Starting iteration 1.
Starting action Action.
Action.c(12): Registering web_reg_save_param_ex was successful  	[MsgId: MMSG-26390]
Action.c(22): Notify: Saving Parameter "CorrelationParameter_1 = 0AA".
Action.c(22): web_url("nav.pl") was successful, 1354 body bytes, 253 header bytes  	[MsgId: MMSG-26386]
Action.c(33): Notify: Parameter Substitution: parameter "CorrelationParameter_1" =  "0AA"
Action.c(33): web_submit_data("login.pl") was successful, 437 body bytes, 404 header bytes  	[MsgId: MMSG-26386]
Action.c(50): web_url("login.pl_2") was successful, 912 body bytes, 225 header bytes  	[MsgId: MMSG-26386]
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```
可以看到was successful！登陆成功的页面是显示用户名的如下图
![jojo](/image/jojo.png)
修改用户名和密码
```c
web_submit_data("login.pl",
		"Action=http://127.0.0.1:1080/WebTours/login.pl",
		"Method=POST",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Snapshot=t23.inf",
		"Mode=HTTP",
		ITEMDATA,
		"Name=userSession", "Value={CorrelationParameter_1}", ENDITEM,
		"Name=username", "Value=aaa", ENDITEM,
		"Name=password", "Value=aaa", ENDITEM,
		"Name=login.x", "Value=65", ENDITEM,
		"Name=login.y", "Value=8", ENDITEM,
		"Name=JSFormSubmit", "Value=off", ENDITEM,
		LAST);
```
输出结果：
```bash
Starting action Action.
Action.c(6): Registering web_reg_save_param_ex was successful  	[MsgId: MMSG-26390]
Action.c(16): Notify: Saving Parameter "CorrelationParameter_1 = 0AA".
Action.c(16): web_url("nav.pl") was successful, 1354 body bytes, 253 header bytes  	[MsgId: MMSG-26386]
Action.c(25): Notify: Parameter Substitution: parameter "CorrelationParameter_1" =  "0AA"
Action.c(25): web_submit_data("login.pl") was successful, 406 body bytes, 225 header bytes  	[MsgId: MMSG-26386]
Action.c(42): web_url("login.pl_2") was successful, 908 body bytes, 225 header bytes  	[MsgId: MMSG-26386]
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```
还是successful！所以就有问题了。检查点应运而生。
在提交数据的前面加上检查点`web_reg_find("Text=jojo",LAST);`
```bash
Action()
{	

//Correlation comment - Do not change! Original value='115393.197731875fQDffDHpzAiDDDDDDfzVDpccfzf' Name ='CorrelationParameter_1'

	web_reg_save_param_ex(
		"ParamName=CorrelationParameter_1",
		"LB=userSession value=",
		"RB=>\n<table border",
		SEARCH_FILTERS,
		"Scope=All",
		"IgnoreRedirections=Yes",
		"RequestUrl=*/nav.pl*",
		LAST);
	web_url("nav.pl",
		"URL=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/welcome.pl?signOff=true",
		"Snapshot=t22.inf",
		"Mode=HTTP",
		LAST);	
	web_submit_data("login.pl",
		"Action=http://127.0.0.1:1080/WebTours/login.pl",
		"Method=POST",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Snapshot=t23.inf",
		"Mode=HTTP",
		ITEMDATA,
		"Name=userSession", "Value={CorrelationParameter_1}", ENDITEM,
		"Name=username", "Value=aaa", ENDITEM,
		"Name=password", "Value=aaa", ENDITEM,
		"Name=login.x", "Value=65", ENDITEM,
		"Name=login.y", "Value=8", ENDITEM,
		"Name=JSFormSubmit", "Value=off", ENDITEM,
		LAST);
		web_reg_find("Text=jojo",
				 LAST);
	web_url("login.pl_2",
		"URL=http://127.0.0.1:1080/WebTours/login.pl?intro=true",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/login.pl",
		"Snapshot=t24.inf",
		"Mode=HTTP",
		LAST);
return 0;
}

```
输出结果：
```bash
Starting iteration 1.
Starting action Action.
Action.c(6): Registering web_reg_save_param_ex was successful  	[MsgId: MMSG-26390]
Action.c(15): Registering web_reg_find was successful  	[MsgId: MMSG-26390]
Action.c(17): Notify: Saving Parameter "CorrelationParameter_1 = 0AA".
Action.c(17): Error -26366: "Text=jojo" not found for web_reg_find  	[MsgId: MERR-26366]
Action.c(40): web_url("login.pl_2") highest severity level was "ERROR", 908 body bytes, 225 header bytes  	[MsgId: MMSG-26388]
Ending action Action.
Ending iteration 1.
```
可以看到有ERROR,`Text=jojo" not found for web_reg_find。`<b>所以通过检查点得到的警示：一定要在业务层面判断是否成功，不能单单从协议层面判断。</b>
```c
Action()
{	

//Correlation comment - Do not change! Original value='115393.197731875fQDffDHpzAiDDDDDDfzVDpccfzf' Name ='CorrelationParameter_1'

	web_reg_save_param_ex(
		"ParamName=CorrelationParameter_1",
		"LB=userSession value=",
		"RB=>\n<table border",
		SEARCH_FILTERS,
		"Scope=All",
		"IgnoreRedirections=Yes",
		"RequestUrl=*/nav.pl*",
		LAST);
	web_url("nav.pl",
		"URL=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/welcome.pl?signOff=true",
		"Snapshot=t22.inf",
		"Mode=HTTP",
		LAST);	
	web_submit_data("login.pl",
		"Action=http://127.0.0.1:1080/WebTours/login.pl",
		"Method=POST",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/nav.pl?in=home",
		"Snapshot=t23.inf",
		"Mode=HTTP",
		ITEMDATA,
		"Name=userSession", "Value={CorrelationParameter_1}", ENDITEM,
		"Name=username", "Value=jojo", ENDITEM,
		"Name=password", "Value=bean", ENDITEM,
		"Name=login.x", "Value=65", ENDITEM,
		"Name=login.y", "Value=8", ENDITEM,
		"Name=JSFormSubmit", "Value=off", ENDITEM,
		LAST);

	web_reg_find("Text=jojo",
				 LAST);	
	web_url("login.pl_2",
		"URL=http://127.0.0.1:1080/WebTours/login.pl?intro=true",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://127.0.0.1:1080/WebTours/login.pl",
		"Snapshot=t24.inf",
		"Mode=HTTP",
		LAST);
return 0;
}
```
输出结果:
```bash
Starting iteration 1.
Starting action Action.
Action.c(6): Registering web_reg_save_param_ex was successful  	[MsgId: MMSG-26390]
Action.c(15): Notify: Saving Parameter "CorrelationParameter_1 = 0AA".
Action.c(15): web_url("nav.pl") was successful, 1354 body bytes, 253 header bytes  	[MsgId: MMSG-26386]
Action.c(23): Notify: Parameter Substitution: parameter "CorrelationParameter_1" =  "0AA"
Action.c(23): web_submit_data("login.pl") was successful, 437 body bytes, 404 header bytes  	[MsgId: MMSG-26386]
Action.c(39): Registering web_reg_find was successful  	[MsgId: MMSG-26390]
Action.c(41): Registered web_reg_find successful for "Text=jojo" (count=1)  	[MsgId: MMSG-26364]
Action.c(41): web_url("login.pl_2") was successful, 912 body bytes, 225 header bytes  	[MsgId: MMSG-26386]
Ending action Action.
Ending iteration 1.
```
可以看到`Registered web_reg_find successful for "Text=jojo" (count=1)`
![jojojo](/image/jojojo.png)
* 检查点需要注意的事项
	1）尽量使用web_reg_find函数
	Web_reg_find是注册类型函数，它本身并不执行，不能通过它的返回值来作为事务的判断条件（因为web_reg_find()的返回值0和1表示web_reg_find()是否注册成功，并不代表查找的内容是否存在，也就是说无论查找的文本内容是否存在，都返回0。它是从返回的缓冲区扫描而不是在接收的页面中查找。这是比web_find更高效的一个函数。web_reg_save_param也是注册类函数，需要放到请求的页面之前，而且查找的内容是服务器返回的缓冲数据中查找，所以查找内容应该看html源代码的内容。
	2）检查的字符串尽量不要是中文，避免不必要的麻烦
	3）运行时设置中的检查点选项对注册函数无效
![checkpoint](/image/checkpoint.png)



---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^