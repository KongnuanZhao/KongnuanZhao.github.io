title: LoadRunner中的事务和参数化
date: 2015-01-31 18:42:41
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
##目的##
> 掌握LoadRunner中的事务和参数化的基本概念和使用方法

##关于事务##
* 什么是事务？
* 为什么要有事务？
	* 我们需要知道不同的操作所花费的时间，这样我们就可以衡量不同的操作对被测系统所造成的影响，那么我们如何知道不同的操作所花费的时间，这就用到了事务，我们在操作之前插入一个事务开始标识，在操作完成后插入一个事务结束表示，这样我们就知道了这个操作所花费的时间。
* 添加事务：lr_start_transaction、lr_end_transaction解析
* 事务的4个状态
	* LR_AUTO是指事务的状态有系统自动根据默认规则来判断，结果为PASS/FAIL。
	* LR_PASS是指事务是以PASS状态通过的，说明该事务正确的完成了，并且记录下对应的时间，这个时间就是指做这件事情所需要消耗的响应时间。
	* LR_FAIL是指事务以FAIL状态结束，该事务是一个失败的事务，没有完成事务中脚本应该达到的效果，得到的时间不是正确操作的时间，这个时间在后期的统计中将被独立统计。
	* LR_STOP将事务以STOP状态停止。事务的PASS和FAIL状态会在场景的对应计数器中记录，包括通过的次数和事务的响应时间，方便后期分析该事务的吞吐量以及响应时间的变化情况。
* 子事务的概念：嵌套在一个事务里面的事务就是子事务。
Action.c
```c
Action()
{

	int status;
	lr_start_transaction("open");

	status = web_url("WebTours", 
		"URL=http://127.0.0.1:1080/WebTours/", 
		"Resource=0", 
		"RecContentType=text/html", 
		"Referer=", 
		"Mode=HTML", 
		LAST);


	//lr_end_transaction("open", LR_AUTO);

	if(status==0){
		lr_end_transaction("open",LR_PASS);
	}else{
		lr_end_transaction("open",LR_FAIL);
	}


	return 0;
}

```
运行结果：
```bash
Virtual User Script started at : 2015-01-31 19:22:09
Starting action vuser_init.
Web Turbo Replay of LoadRunner 11.0.0 for Windows 7; build 9409 (Jan 11 2012 11:34:16)  	[MsgId: MMSG-27143]
Run Mode: HTML  	[MsgId: MMSG-26000]
Run-Time Settings file: "C:\Users\Administrator\AppData\Local\Temp\noname1\\default.cfg"  	[MsgId: MMSG-27141]
Ending action vuser_init.
Running Vuser...
Starting iteration 1.
Starting action Action.
Action.c(5): Notify: Transaction "open" started.
Action.c(7): Detected non-resource "http://127.0.0.1:1080/WebTours/header.html" in "http://127.0.0.1:1080/WebTours/"  	[MsgId: MMSG-26574]
……
Action.c(7): web_url("WebTours") was successful, 6449 body bytes, 1562 header bytes  	[MsgId: MMSG-26386]
Action.c(19): Notify: Transaction "open" ended with "Pass" status (Duration: 2.9054 Wasted Time: 0.9062).
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```

##对事务的探讨##
* 思考:通过事务获得了响应时间，那么就够了吗？
	* Wasted Time：除了事务请求本身之外消耗的时间，如LoadRunner工具本身的损耗，或者代码中如for循环等非事务请求本身的损耗。
	事务请求本身的损耗=Duration总共持续的时间 - Wasted Time其它原因消耗的时间。（在Controller和Analysis中就无需考虑Wasted Time，本身就是事务请求时间）
	需要具体分析。下面是一个简单的例子，
```c
Action()
{

	int status;
    merc_timer_handle_t timer;
	int i;
	char save[1000];
	double wastetime;
	lr_start_transaction("open");

	status = web_url("WebTours", 
		"URL=http://127.0.0.1:1080/WebTours/", 
		"Resource=0", 
		"RecContentType=text/html", 
		"Referer=", 
		"Mode=HTML", 
		LAST);


	timer =lr_start_timer();
	for(i=0;i<1000;i++){
		sprintf(save,"this is the way waste time in a script= %d",i);
	}
	wastetime=lr_end_timer(timer);
	lr_wasted_time(wastetime*1000);
	lr_end_transaction("open", LR_AUTO);
	return 0;
}

```
运行结果：
```bash
……

Action.c(11): web_url("WebTours") was successful, 6445 body bytes, 1562 header bytes  	[MsgId: MMSG-26386]
Action.c(26): Notify: Transaction "open" ended with "Pass" status (Duration: 6.3489 Wasted Time: 5.8070).
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```
* 需要注意的
	* 事务中不要插入日志函数
	* 不要插入集合点函数
		* 插入集合点是为了衡量在加重负载的情况下的性能情况。在计划中，可能会要求系统能够承受1000人同时提交数据，在LoadRunner中可以通过在提交数据操作前面加入集合点，这样当虚拟用户运行到提交数据的集合点时，LoadRunner就会检查同时有多少用户运行到集合点，如果不到1000人，LoadRunner就会命令已经到集合点的用户在此等待，当在集合点等待的用户达到1000人时，LoadRunner命令1000人同时去提交数据，从而达到计划中的需求。
	* 尽量不要插入思考时间lr_think_time()
		* 录制脚本时 我们一般会选择记录思考时间record think time,Loadrunner做为性能测试工具，录制时记录的是客户端和服务端的交互，如果要精确模拟用户的行为，那么客户操作客户端时花费了很多时间要怎么模拟呢?录入填写提交的内容，从列表中下拉搜索选择特定的值等，这时LOADRUNNER不会记录用户的客户端操作，而是记录了用户这段时间，成为思考时间(Think-time)，因为用户的这些客户端操作不会影响服务端，只是让服务器端在这段时间内没有请求而已。所以加入思考时间就能模拟出熟练的或者生疏的用户操作，接近实际对于服务端的压力。

##参数化##
* 重温脚本录制和回放的过程
* 脚本开发遇到的问题：
	* 被业务场景所迫：  所有用户都输入相同的数据，不能体现出真实的业务环境。（搜索操作）
	* 被系统体系所迫：  存在缓存，不能体现出真正的性能。
	* 被系统业务约束所迫：  有些系统禁止一个用户多次登陆的系统,也就严重到无法测试的地步了。

Loadrunner中提供一种机制帮助解决上述问题， 叫参数化（parameterization）

* LoadRunner中的<b>参数化</b>和函数的<b>参数</b>之间的关系
大部分情况下只有函数的参数才能参数化，但也不是所有函数的参数都可以参数化。
* 哪些需要参数化？
	* 登陆认证信息
	* 一些和时间相关的，违反时间约束的
	* 一些受其他字段约束的
	* 一些来自于其他数据源（例如数据库的）
	* 其他在运行过程中需要变动的

简单的例子:
```c
Action()
{
	web_url("hello",
		"URL=http://www.iciba.com/{searchword}",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://www.iciba.com/",
		"Snapshot=t29.inf",
		"Mode=HTML",
		LAST);

	
return 0;
}
```
在日志里找到这一句
```bash
Action.c(3): Notify: Parameter Substitution: parameter "searchword" =  "hello"
```
参数化之后，写死了的代码变活，代码也变得更加美好！
* 参数化取值方法
select next row : 获取下一行数据的方式
update value on : 重新获取下一个参数的时机（条件）
![properties](/image/properties.png)
* 总结：
1、根据你需要的参数获取方式设置不同的组合，满足你的需求
2、select next row能随即尽量随机
3、使用不同用户登录一次使用 Unique + Once模式
* 参数化和变量的区别
参数作用域远远大于局部变量，在一个action中的参数可以再另一个函数使用，而局部变量不行，除非是全局
变量。
* 参数和变量的转换
	* 参数转换成变量 lr_eval_string
	* 变量转换成参数 lr_save_string

注：
LR参数化后highest severity level was"ERROR"的解决方法：
处理两种方法如下：
1.  选择使用URL＿based script录制。
2.  打开recording options，在开始录制option下的recording中选择recording level为HTML-based script，点击HTML Advanced，选择script type为A script containing explicit.即可。


#课程作业#
<b>1、LoadRunner中的事务的含义是什么？随便录制或找一个使用LoadRunner脚本，添加一个事务，然后添加自己的其他代码（目的是要加深学习wasted time这个概念），使用lr_wasted_time将自己代码浪费的时间添加到wasted time中。（例子自己随便选，如果实在不知如果弄，按照课程中的例子也可以），将这个作为附件提交。</b>
事务又称为Transaction，在LoadRunner中的定义如下:An end-to-end(browser-to-browser) measurement of one or more user actions within action file。中文理解如下：事务(Transaction)这样一个点，我们为了衡量某个action的性能，需要在action的开始和结束位置插入这样一个范围，这就定义了一个transaction。
例子见上面笔记，代码见附件~
<b>2、Loadrunner中的参数化的目的是什么？ 一般什么情况下进行参数化，如果有实际经验，结合实际经验阐述更好。</b>
目的：实现脚本和数据分离，使代码变活。一般在有数据变动的情况下需要进行参数化。
举个简单的参数化的例子，`.dat`文件内容：
```bash
NewParam
hello
word
found
text
log
               //这里有个空行，不可没有，也不可多一行
```
Action.c
```c
Action()
{
	web_url("hello",
		"URL=http://www.iciba.com/{NewParam}",
		"Resource=0",
		"RecContentType=text/html",
		"Referer=http://www.iciba.com/",
		"Snapshot=t29.inf",
		"Mode=HTML",
		LAST);

	
return 0;
}

```
迭代5次，查看部分日志
```bash
Running Vuser...
Starting iteration 1.
Starting action Action.
Action.c(3): Notify: Parameter Substitution: parameter "NewParam" =  "hello"
……
Starting action Action.
Action.c(3): Notify: Parameter Substitution: parameter "NewParam" =  "word"
……
Starting action Action.
Action.c(3): Notify: Parameter Substitution: parameter "NewParam" =  "found"
……
Starting action Action.
Action.c(3): Notify: Parameter Substitution: parameter "NewParam" =  "text"
……
Starting action Action.
Action.c(3): Notify: Parameter Substitution: parameter "NewParam" =  "log"
……
```
<b>3、Loadrunner参数化中 select next row 和update value on一共有几种组合方式？阐述如果select next row的选择是random，和它对应的3种update value on分别组合，对这3组组合的含义进行描述。</b>
select next row : 获取下一行数据的方式
update value on : 重新获取下一个参数的时机（条件）
一共十种组合方式，如下图所示：
![para](/image/para.jpg)
random的三次组合，含义如下图：
![random](/image/random.jpg)



---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^