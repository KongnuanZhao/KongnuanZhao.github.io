title: LoadRunner脚本开发和常用函数
date: 2015-01-20 21:15:11
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
> 脚本开发的原则：简单、正确、高效

LoadRunner实现机制
![cccc](/image/cccc.png)
```c
#include<globals.h>
main(){
	vuser_init();
	Action();
	vuser_end();
}
```
解释执行C代码，不需要像java等一样需要编译。
关于解释型语言和编译型语言的概念和区别：
概念：
* 编译型语言：把源程序全部编译成二进制代码的可运行程序。之后，可直接运行这个程序无需再次编译。
![complicate](/image/complicate.jpg)
* 解释型语言：把源程序交给翻译器翻译一句就提交给计算机，然后计算机执行一句并不行程目标程序，直到运行结束。
![jieshi](/image/jieshi.jpg)
区别：
* 编译型语言，编译完成之后执行速度快，效率高；依赖编译器 如C,C++,Delphi,Fortran.
* 解释型语言，执行速度慢，效率低；依赖解释器；如java、Basic.
总结：编译型语言在编译成目标程序后是可以通过操作系统直接运行，而解释语言的执行则需要一个解释环境（相当于一台虚拟机）,解释器在程序执行期间一直随之运行，在这里LoadRunner充当了解释器的角色，一般来说，解释器比编译有更好的灵活性，而编译有着很好的性能。
其中java是特殊的，java程序也需要编译，但是没有直接编译成为机器语言，而是编译成字节码。 
java字节码的执行有两种方式：
1.即时编译方式：解释器先将字节码编译成机器码，然后再执行该机器码。
2.解释执行方式：解释器通过每次解释并执行一段代码来完成java字节码程序的所有操作。
通常采用的是第二种党法，由于JVM具有较好的灵活性，使字节码翻译为机器码效率较高。对于那些对于执行速度要求较高的应用程序，解释器可将java字节码即时编译为机器码，从而很好的保证了java代码的可移植性和高性能。
![java](/image/java.jpg)

##局部变量和全局变量##
* 局部变量
```c
Action()
{
    int a=10;
	
	printf("%d",a);
    int b=2;
	lr_output_message("%d",a);
	lr_output_message("%d",a);
	lr_output_message("%d",a);

	return 0;
}
```
```bash
Action.c (6): illegal statement termination
Action.c (6): skipping `int'
Action.c (6): undeclared identifier `b'
e:\\hp\\mytest\\bian\\\\combined_bian.c (5): 3 errors, not writing pre_cci.ci
```
以上例子可以看出LoadRunner虽然基本完全支持C，但是和C还有一点细微的差别。这里定义变量要提到前面去。
```c
int a=10;
int b=2;
printf("%d",a);
```
实例二在函数外面定义，
```c
int a=100;
Action()
{
   // int a=10;
	printf("%d",a);

	lr_output_message("%d",a);
	lr_output_message("%d",a);
	lr_output_message("%d",a);

	
	return 0;
}
```
输出结果
```bash
Starting iteration 1.
Starting action Action.
Action.c(7): 100
Action.c(8): 100
Action.c(9): 100
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```
去掉注释，输出，可以看出，局部变量会覆盖文件级变量（即在函数外面定义的变量）
```bash
Starting iteration 1.
Starting action Action.
Action.c(7): 10
Action.c(8): 10
Action.c(9): 10
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```
* 全局变量在`globals.h`定义
```bash
//--------------------------------------------------------------------
// Include Files
#include "lrun.h"
#include "web_api.h"
#include "lrw_custom_body.h"

//--------------------------------------------------------------------
// Global Variables
// 
char *p;
```

总结，
* 在init、action、end中定义的变量就是局部变量
* 在globals.h中定义的变量是全局变量
* 什么时候定义全局变量? 
整个过程中固定不变的，例如URL地址、KEY、其他
* 支持指针、数组、控制流等

##函数##
LoadRunner中的几种类型的函数：
* 通用函数lr开头的（例如日志函数，思考时间函数...）
* 与编程语言相关的函数
* 与协议相关的函数
* 自定义函数

<b>(F1查看帮助文档)</b>
<table><tr><td>通用函数lr开头的</td><td>①lr_think_time()、lr_get_host_name()、lr_whoami()、lr_get_attrib_string()</br>
②LoadRunner错误处理机制/lr_continue_on _error函数</br>③日志函数解析/lr_output_message、lr_log_message 、lr_message 、lr_error_message</br>④lr_load_dll</td></tr><tr><td>与编程语言相关的函数</td><td>strcpy与strcat、strcmp、atoi、sprinf、time、文件操作..略</td></tr><tr><td>与协议相关的函数</td><td>web_link与web_url(get)、web_submit_form与web_submit_data(POST)、web_submit_form中的hidden自动发送、web_custom_request、web_add_header、web_get_int_property</td></tr></table>

* 关于DLL
> 在Windows中，许多应用程序并不是一个完整的可执行文件，它们被分割成一些相对独立的动态链接库，即DLL文件，放置于系统中。当我们执行某一个程序时，相应的DLL文件就会被调用。一个应用程序可使用多个DLL文件，一个DLL文件也可能被不同的应用程序使用，这样的DLL文件被称为共享DLL文件。

LoadRunner也支持dll`lr_load_dll("user32.dll")`

#课后作业#
<b>作业1： 详细阐述HTTP协议中的GET和POST请求方式的区别和各自的使用场景。</b>
区别：
* HTTP-GET以使用MIME类型application/x-www-form-urlencoded的urlencoded文本的格式传递参数。Urlencoding是一种字符编码，保证被传送的参数由遵循规范的文本组成，例如一个空格的编码是"%20"。附加参数还能被认为是一个查询字符串，即传送数据在URL中可以看到。
* 与HTTP-GET类似，HTTP POST参数也是被URL编码的。然而，变量名/变量值不作为URL的一部分被传送，而是放在实际的HTTP请求消息内部HTML HEADER内被传送，所以相对来说更安全一些。
*  GET方式提交的数据最多只能有1024字节，而POST则没有此限制。

以下情况使用POST，
* 传送的消息过长
* 要传送的数据不是采用ASCII编码
* 数据安全性要求高

<b>作业2： 使用VuGen编写一个脚本，脚本中需要包含定义局部变量、全局变量、指针、数组，将以上变量赋值后（初值自己随便指定，符合语法即可）使用LoadRunner中的输出函数打印出，将脚本保存后打包作为附件上传。</b>
```c


int a=100;
Action()
{
    int a=10;
	char p[]={'Z','K','N'};//以字符形式赋值的时候系统不会默认加上/0,所以认为没有结束。
	char s[]="hello loadrunner!";//以字符串形式赋值的时候，系统会默认加上/0
	char t[]="test end!!";
    char * host;
	lr_output_message("%d",a);

	lr_output_message("p=%s",p);
	lr_output_message("s=%s",s);
	lr_output_message("t=%s",t);
lr_output_message("---------------------------我是华丽丽的分隔线------------------------------------------");
	lr_output_message("sizeof(s)=%d",sizeof(s));//比实际的多了一个，是系统默认加上的/0
	lr_output_message("strlen(s)=%d",strlen(s));
lr_output_message("---------------------------我是华丽丽的分隔线------------------------------------------");
	if(a<10){
		lr_output_message("a小于10");
	}else if(a==10){
		lr_output_message("a等于10");
	}else{
		lr_output_message("a大于10");
	}
lr_output_message("---------------------------我是华丽丽的分隔线------------------------------------------");
	while(a>0){
		lr_output_message("我是好人！！");
		a--;
	}

lr_output_message("---------------------------我是华丽丽的分隔线------------------------------------------");
    host=lr_get_host_name();
	lr_output_message("my_host_name=%s",host);
	return 0;
}

//int add(int a,int b){
//    return a+b;
//}

```
<b>作业3： LoadRunner包含哪几种类型函数？在VuGen中使用C语言中的文件操作函数编写一个脚本，脚本中包含在C盘创建一个文件，并将自己的姓名的全拼写入该文件并保存，将脚本保存上打包作为附件上传。</b>
LoadRunner中的几种类型的函数：
* 通用函数lr开头的（例如日志函数，思考时间函数...）
* 与编程语言相关的函数
* 与协议相关的函数
* 自定义函数
Action.c
```c
#include "myname.h"
Action()
{
 lr_output_message("my name=%s",printName("zhaokongnuan"));
	return 0;
}
```
myname.h
```c
char * printName(char * name){
    return name;
}

```
<b>作业4： 使用web_url函数随便在互联网上下载一个图片（自己找个图片的URL），之后使用web_get_int_property函数判断下载的图片的大小，当下载大于0时，就打印出下载成功，否则打印出下载失败。将脚本保存上打包作为附件上传。</b>
Action.c
```c
Action()
{
    int i; 
	int j;

   lr_start_transaction("DownloadPicture"); 
	web_url("ac4bd11373f082026b041f2248fbfbedab641b1c.jpg",
			"URL=http://b.hiphotos.baidu.com/image/w%3D1366%3Bcrop%3D0%2C0%2C1366%2C768/sign=5fada6b8097b02080cc93be254efc9b0/ac4bd11373f082026b041f2248fbfbedab641b1c.jpg",
			"Resource=1",
			"RecContentType=image/jpg",
			 LAST);
	lr_output_message("*******图片存在********");
    i = web_get_int_property(HTTP_INFO_DOWNLOAD_SIZE); 
    lr_output_message("The download size of the Picture was: %d",i); 
    if(i >0) 
    { 
		lr_output_message("下载成功");
		lr_end_transaction("DownloadPicture", LR_PASS); 

	}else{
		lr_output_message("下载失败");
		lr_end_transaction("DownloadPicture", LR_FAIL); 
	}
	return 0;
}


```

---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^