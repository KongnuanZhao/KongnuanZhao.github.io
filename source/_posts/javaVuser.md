title: LoadRunner中的JavaVuser
date: 2015-03-28 14:16:10
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
越学越觉得，软件性能测试涉及面非常广泛。需得一个知识点一个知识点往前推啊。
##引入Java Vuser##
* 为什么要使用Java Vuser？
	* 有些使用C解决起来很麻烦，但是使用java很方便，存在强大的开源类库
	* 更方便的使用自定义的类，节省开发脚本的时间。特别是对于自定义的一些算法或类，更是方便。

##LoadRunner中两种语言的区别##
* Java Vuser与C的区别
	* 语言不同、语法不同，支持的函数不同
	* C是解释的，Java Vuser的是编译的，出错提示更准确。
（Java Vuser是编译的，使用Java 的编译器Javac,这样出错提示更准确）
关于C脚本开发参看之前写的[LoadRunner脚本开发和常用函数](http://www.zhaokongnuan.com/2015/01/20/LoadRunner%E8%84%9A%E6%9C%AC%E5%BC%80%E5%8F%91%E5%92%8C%E5%B8%B8%E7%94%A8%E5%87%BD%E6%95%B0/)
* 运行java Vuser要配置JDK，如下图
![jvuserjdk](/image/jvuserjdk.png)
* Java Vuser缺点和不足
	* 学习更多的知识，要会使用Java语言。
	* 有些语法支持的不好，例如静态代码块，尽量少用不常用的语法和功能。
```java
public class StaticBlock {
	static{
		System.out.println("hello java Vuser !");
	}
	
    public static void main(String[] args) {	
	StaticBlock sBlock=new StaticBlock();
	}
}
```
运行结果 ：`hello java Vuser !`
```java
public class StaticBlock {
	static{
		System.out.println("hello java Vuser !");
	}
	
    public static void main(String[] args) {	
	    System.out.println("i am main ~");
	}
}
```
运行结果：
```bash
hello java Vuser !
i am main ~
```
静态代码块是在类加载的时候执行。加载完类之后然后从main开始执行，所以输出结果是上面的顺序。
在loadrunner中会报错。
```java
import lrapi.lr;
public class Actions
{
        static{
		System.out.println("this is static !");
		lr.think_time(10);
	}

	public int init() throws Throwable {
	    System.out.println("hello, this is init()!");
		return 0;
	}//end of init

	public int action() throws Throwable {
		return 0;
	}//end of action

	public int end() throws Throwable {
		return 0;
	}//end of end
}
```
```bash
Error (-17998): Failed to get [param not passed in call] thread TLS entry.
Error (-17998): Failed to get [param not passed in call] thread TLS entry.

Virtual User Script started at : 2015-03-28 21:14:51
Starting action vuser_init.
System.out: hello, this is init()!                                                                                                                                                      Notify:
Ending action vuser_init.
Running Vuser...
Starting iteration 1.
Starting action Actions.
Ending action Actions.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.
```

##Java Vuser需要注意##
* 如果在脚本中包括自定义的类，你需要确保这个类是线程安全的，如果不确定，使用进程方式运行，这样可以每个进程一个虚拟机，充分的隔离。
	* 线程安全的含义：线程安全就是说多线程访问修改同一代码，不会产生不确定的结果。关于线程的基础概念理解参看[操作系统之线程管理](http://www.zhaokongnuan.com/2015/01/09/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E4%B9%8B%E7%BA%BF%E7%A8%8B%E7%AE%A1%E7%90%86/)
	* 没有修改，只有读肯定是线程安全的
	* 操作不同的对象或者变量是线程安全的
	* 线程安全是很难检测的。往往在很少用户并发时很难发现，甚至很多程序运行多年才发现，相对来说，在大压力下或CPU资源紧张的时候更容易发现。
* 不要在Java Vuser衍生出来的线程中使用Java Vuser Api

##编写Java Vuser的一般步骤##
* 了解被测项目业务和所使用的技术，分析是否使用JavaVuser
* 在eclipse中编写一个正确的压力模拟代码
* 将eclipse中的代码移到LoadRunner中的JavaVuser
* 结合业务特点，对脚本进一步增强
* 使用Comtroller运行多用的JavaVuser
* ……

##从web转化为Java Vuser##
1）将要转换的web脚本复制出来并保存到文本文件中
2）参数界定符部分需要由{}手动修改成<>
3）打开CMD
4）切换到 C:\Program Files\HP\LoadRunner\dat
5）运行 ..\bin\sed -f web_to_java.sed c:\web.txt > c:\java.txt
6 ) 创建参数并进行其他操作

#课后作业#
<b>LoadRunner中的JavaVuser有哪些特点？具体有哪些使用场景？</b>
* JavaVuser有哪些特点？
	* 有些使用C解决起来很麻烦，但是使用java很方便，存在强大的开源类库
	* 更方便的使用自定义的类，节省开发脚本的时间。特别是对于自定义的一些算法或类，更是方便。
* 具体有哪些使用场景？没有用过o(╯□╰)o，应该什么场景都能用吧。

<b>2、线程安全的含义是什么？如何写出线程安全的代码？</b>
线程安全就是说多线程访问修改同一代码，不会产生不确定的结果。一个简单的栗子：
```java
public class Count {
	int num = 0;
	public void count() {
		for (int i = 1; i <= 10; i++) {
			num += i;
		}
		System.out.println(Thread.currentThread().getName() + " : " + num);
	}

	public static void main(String[] args) {
		Runnable runnable = new Runnable() {
			Count count = new Count();
			public void run() {
				count.count();
			}
		};
		for (int i = 0; i < 10; i++) {
			new Thread(runnable).start();
		}
	}
}
```
输出结果
```bash
Thread-0 : 55
Thread-1 : 110
Thread-2 : 165
Thread-3 : 220
Thread-4 : 275
Thread-5 : 330
Thread-7 : 440
Thread-6 : 385
Thread-8 : 495
Thread-9 : 550
```
可以看到是结果是对num累加的。
再运行几次，可能出现下面这个结果
```bash
Thread-0 : 110
Thread-5 : 165
Thread-3 : 275
Thread-1 : 110
Thread-4 : 275
Thread-9 : 330
Thread-8 : 440
Thread-7 : 385
Thread-2 : 495
Thread-6 : 550
```
现在把num这个变量拿到count函数里面去
```bash
public class Count {
	
	public void count() {
		int num = 0;
		for (int i = 1; i <= 10; i++) {
			num += i;
		}
		System.out.println(Thread.currentThread().getName() + " : " + num);
	}

	public static void main(String[] args) {
		Runnable runnable = new Runnable() {
			Count count = new Count();
			public void run() {
				count.count();
			}
		};
		for (int i = 0; i < 10; i++) {
			new Thread(runnable).start();
		}
	}
}
```
运行结果：
```bash
Thread-2 : 55
Thread-6 : 55
Thread-3 : 55
Thread-7 : 55
Thread-0 : 55
Thread-1 : 55
Thread-4 : 55
Thread-5 : 55
Thread-8 : 55
Thread-9 : 55
```
从第一个例子可以看出，线程之间共享成员变量，再看第一个程序的第二个结果，也就是说存在成员变量的类用于多线程时是不安全的，不安全体现在这个成员变量可能发生非原子性的操作。而变量定义在方法内也就是局部变量是线程安全的。
（对于运行结果的解释：在多个线程之间共享了Count类的一个对象，这个对象是被创建在主内存(堆内存)中，每个线程都有自己的工作内存(线程栈)，工作内存存储了主内存Count对象的一个副本，当线程操作Count对象时，首先从主内存复制Count对象到工作内存中，然后执行代码count.count()，改变了num值，最后用工作内存Count刷新主内存Count。当一个对象在多个内存中都存在副本时，如果一个内存修改了共享变量，其它线程也应该能够看到被修改后的值，此为可见性。多个线程执行时，CPU对线程的调度是随机的，我们不知道当前程序被执行到哪步就切换到了下一个线程，于是就有了不同的运行结果，显然这不是我们所期望的。）
如何写出线程安全的代码？线程安全性非常类似于在描述 ACID(原子性、一致性、独立性和持久性)事务时使用的一致性,对于事务的一致性，数据库都是用锁机制来实现的。java也可以采用锁机制比如上述java代码加上`synchronized`关键字：
```java
public class Count {
	int num = 0;
	public synchronized void count() {
		
		for (int i = 1; i <= 10; i++) {
			num += i;
		}
		System.out.println(Thread.currentThread().getName() + " : " + num);
	}
	

	public static void main(String[] args) {
		Runnable runnable = new Runnable() {
			Count count = new Count();
			public  void run() {
				count.count();
			}
		};
		for (int i = 0; i < 10; i++) {
			new Thread(runnable).start();
		}
	}
}
```
运行结果：
```bash
Thread-0 : 55
Thread-1 : 110
Thread-4 : 165
Thread-2 : 220
Thread-5 : 275
Thread-3 : 330
Thread-7 : 385
Thread-6 : 440
Thread-9 : 495
Thread-8 : 550
```
就只会出现累加的情况，就不会出现发生非原子性的操作。
另外还有生产者/消费者模式、通过`volatile`关键字等方式以保证线程安全。
<b>3、使用JavaVuser的方式完成如下描述的操作，将生成的JavaVuser代码打包上传。
使用JavaVuser完成LoadRunner自带的例子中的注册新用户操作，脚本需要参数化新注册用户的用户名、密码和确认密码。需要三个参数化变量从同一个文件取值。参数化后生成10个新用户，名字是test01--test10，密码随便但不要重复，需要使用same line as ，使用关联判断是否注册成功。将JavaVuser脚本目录打包作为附件提交。</b>
（代码见附件）


---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^