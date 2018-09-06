title: 性能测试之探索Java虚拟机
date: 2015-04-02 15:42:34
author: 赵空暖
tags:
- Java
- 性能测试
categories:
- Java
- 性能测试
---
> 知识点好多，我都醉了。

#课堂笔记#
##风华正茂的Java##
* Sun不仅发明Java语言，也弄出Java虚拟机。
	* 有哪些虚拟机？VMWare、Visual Box、JVM
	* 区别？VMWare或者Visual Box都是使用软件模拟物理CPU的指令集；JVM使用软件模拟Java 字节码的指令集

* 虚拟化技术：所谓虚拟化技术就是将事物从一种形式转变成另一种形式，最常用的虚拟化技术有操作系统中内存的虚拟化，实际运行时用户需要的内存空间可能远远大于物理机器的内存大小，利用内存的虚拟化技术，用户可以将一部分硬盘虚拟化为内存，而这对用户是透明的。
4）Java的可移植性
越到底层，越不可移植，越与硬件息息相关，越到上层，越脱离底层，越好移植。
Java虚拟机在哪里？
```bash
C:\Users\Administrator\Desktop>java -version
java version "1.7.0_51"
Java(TM) SE Runtime Environment (build 1.7.0_51-b13)
Java HotSpot(TM) Client VM (build 24.51-b03, mixed mode, sharing)
```
* HotSpot大事记
	* HotSpot为Longview Technologies开发被SUN收购
	* 2006年Java开源并建立OpenJDK，HotSpot成为Sun JDK和OpenJDK中所带的虚拟机
	* 2008 年 Oracle收购BEA，得到JRockit VM
	* 2010年Oracle 收购 Sun，得到Hotspot
	* Oracle宣布在JDK8时整合JRockit和Hotspot，优势互补，在Hotspot基础上，移植JRockit优秀特性

* JVM的启动流程
![jvm_start](/image/jvm_start.jpg)
* 查看虚拟机进程
```bash
package com.test;
public class Test {
public static void main(String[] args) throws InterruptedException {
	System.out.println("************start*************");
	Thread.sleep(30000000);
	System.out.println("*************end**************");
}
}
```
![javax](/image/javax.png)

##一个进程-一个世界##
1、Java虚拟机仍然是一个进程，因此符合操作系统进程的特征，且是多线程的。整个虚拟机仍然脱离不开操作系统约束。
2、在Java虚拟机中，内存管理仍然重点。
Java虚拟机运行时数据区结构图
![c_jvm](/image/c_jvm.png)
将内存划分成不同的区域，有些是在java虚拟机启动的时候就存在了，有些是随着线程的生成和销毁而存在的。
* 方法区
	* 保存装载的类信息
		* 类型的常量池
		* 字段，方法信息
		* 方法字节码
	* 通常和永久区(Perm)关联在一起

* Java堆
	* 和程序开发密切相关
	* 应用系统对象都保存在Java堆中
	* 所有线程共享Java堆
	* 对分代GC来说，堆也是分代的
		* 分代思想：依据对象的存活周期进行分类，短命对象归为新生代，长命对象归为老年代。
	* GC的主要工作区间
	![gc](/image/gc.png)

* Java栈(VM栈)
	* 线程私有
	* 栈由一系列帧组成（因此Java栈也叫做帧栈）
	* 帧保存一个方法的局部变量、操作数栈、常量池指针
	* 每一次方法调用创建一个帧，并压栈
* 栈、堆、方法区交互
```java
package com.test;

public class AppMain
//运行时, jvm 把appmain的信息都放入方法区
{
	public static void main(String[] args){
	//main 方法本身放入方法区。 
	Sample test1 = new Sample("测试1");
	//test1是引用，所以放到栈区里， Sample是自定义对象应该放到堆里面 
	Sample test2 = new Sample( " 测试2 " );
	test1.printName();
	test2.printName();
	}
}

```
Sample类
```java
package com.test;
public class Sample
//运行时,jvm把appmain的信息都放入方法区
{
	private String name;
//new Sample实例后， name引用放入栈区里，name 对象放入堆里 
	public Sample(String name){
	this.name = name;
	}
//print方法本身放入 方法区里。
	public void printName(){
	System.out.println(name);
	}
}

```
![running_time](/image/running_time.jpg)

##OOM--Out Of Memory##
1）操作系统与OOM killer
（对一个进程来说，所能看到的虚拟地址空间，并不是真正的物理内存空间，每一个进程都认为自己占了整个的4G地址空间）
2）Java虚拟机中的OOM,一般由三种情况：
* 永久区溢出 Exception in thread "main" java.lang.OutOfMemoryError: PermGen space
这一部分用于存放Class和Meta的信息，Class在被 Load的时候被放入PermGen space区域(包括常量池: 静态变量)，它和存放Instance的Heap区域不同，GC(Garbage Collection)不会在主程序运行期对PermGen space进行清理，所以如果你的APP会LOAD很多CLASS的话,就很可能出现PermGen space错误。
可以通过设置jvm启动参数来解决 `-XX:MaxPermSize=256m`
* 堆溢出 java.lang.OutOfMemoryError: Java heap space
这部分用于存放类的实例。被缓存的实例(Cache)对象，大的map,list引用大的对象等等，都会保存于此。
堆内存会在jvm启动时自动设置，初始值 -Xms为物理内存的1/64，最大值-Xmx为1/4;可以通过参数-Xmn、-Xms、-Xmx设置，一般-Xms和-Xmx不超过80%，-Xmn为-Xmx的1/4;
* 栈溢出 Exception in thread "main" java.lang.StackOverflowError
这部分用于存放局部变量、方法栈帧信息。栈帧太多，也就是函数调用层级过多时就会出现此异常，检查是否有死递归的情况。

##Java与C++之间的围城##
垃圾回收GC(Garbage Collection)
* 引用计数法(传统简单做法)
	* 通过引用计算来回收垃圾
	* 引用计数器的实现很简单，对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1，当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，则对象A就不可能再被使用。
![counter](/image/counter.png)
	* 问题
		* 引用和去引用伴随加法和减法，影响性能
		* 很难处理循环引用

* 标记-清除（进化）
标记-清除算法是现代垃圾回收算法的思想基础。标记-清除算法将垃圾回收分为两个阶段：标记阶段和清除阶段。一种可行的实现是，在标记阶段，首先通过根节点，标记所有从根节点开始的可达对象。因此，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。
![mark](/image/mark.png)

* 复制算法(高效化)
	* 将原有的内存空间分为两块，每次只使用其中一块，在垃圾回收时，将正在使用的内存中的存活对象复制到未使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收。
![copy](/image/copy.png)
	* 问题：空间浪费 

* 根据不同代的特点，选取合适的收集算法
	* 少量对象存活，适合复制算法
	* 大量对象存活，适合标记清理或者标记压缩

##Java虚拟机的无奈##
Java虚拟机是复杂的，但是站在它的角度来看，也是无奈的。
一方面，程序越来越复杂，时刻都存在对象的出生和死亡，另一方面，还要尽量避免Stop the World。相当于制造垃圾的同时也在清理垃圾。
* Stop-The-World
	* Java中一种全局暂停的现象
	* 全局停顿，所有Java代码停止，native代码可以执行，但不能和JVM交互
	* 多半由于GC引起
		* 为什么呢？看到的一个很好类比：在聚会时打扫房间，聚会时很乱，又有新的垃圾产生，房间永远打扫不干净，只有让大家停止活动了，才能将房间打扫干净。

#课堂作业#
<b>1、HotSpot虚拟机判断对象是否可以被回收的方式或方法是什么？</b>
通过一些算法来判断，比如引用计数法、 标记-清除法和复制算法等。
* 引用计数器：对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1，当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，则对象A就不可能再被使用，然后就可以回收啦。
* 标记-清除：在标记阶段，首先通过根节点，标记所有从根节点开始的可达对象。因此，未被标记的对象就是未被引用的垃圾对象。然后，在清除阶段，清除所有未被标记的对象。
* 复制算法：将原有的内存空间分为两块，每次只使用其中一块，在垃圾回收时，将正在使用的内存中的存活对象复制到未使用的内存块中，之后，清除正在使用的内存块中的所有对象，交换两个内存的角色，完成垃圾回收。

<b>2、使用jmap命令生成堆Dump文件的命令是什么？jmap命令还可以有什么其他的用处？</b>
将内存使用的详细情况输出到文件`jmap -dump:format=b,file=m.datpid`
jmap -heap pid 查看java 堆（heap）使用情况。
jmap -histo pid 查看堆内存(histogram)中的对象数量，大小。

<b>3、HotSpot虚拟机对堆内存是如何划分的？各部分的作用都是什么？如果查看各部分的使用情况？</b>
Java虚拟机运行时数据区结构图
![c_jvm](/image/c_jvm.png)
将内存划分成不同的区域，有些是在java虚拟机启动的时候就存在了，有些是随着线程的生成和销毁而存在的。
* 方法区
	* 保存装载的类信息
		* 类型的常量池
		* 字段，方法信息
		* 方法字节码
	* 通常和永久区(Perm)关联在一起

* Java堆
	* 和程序开发密切相关
	* 应用系统对象都保存在Java堆中
	* 所有线程共享Java堆
	* 对分代GC来说，堆也是分代的
		* 分代思想：依据对象的存活周期进行分类，短命对象归为新生代，长命对象归为老年代。
	* GC的主要工作区间
	![gc](/image/gc.png)

* Java栈(VM栈)
	* 线程私有
	* 栈由一系列帧组成（因此Java栈也叫做帧栈）
	* 帧保存一个方法的局部变量、操作数栈、常量池指针
	* 每一次方法调用创建一个帧，并压栈
* 栈、堆、方法区交互来说明各部分的作用
```java
package com.test;

public class AppMain
//运行时, jvm 把appmain的信息都放入方法区
{
	public static void main(String[] args){
	//main 方法本身放入方法区。 
	Sample test1 = new Sample("测试1");
	//test1是引用，所以放到栈区里， Sample是自定义对象应该放到堆里面 
	Sample test2 = new Sample( " 测试2 " );
	test1.printName();
	test2.printName();
	}
}

```
Sample类
```java
package com.test;
public class Sample
//运行时,jvm把appmain的信息都放入方法区
{
	private String name;
//new Sample实例后， name引用放入栈区里，name 对象放入堆里 
	public Sample(String name){
	this.name = name;
	}
//print方法本身放入 方法区里。
	public void printName(){
	System.out.println(name);
	}
}

```
![running_time](/image/running_time.jpg)



---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^