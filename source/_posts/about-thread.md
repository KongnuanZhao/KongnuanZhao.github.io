title: 线程的自白
date: 2015-04-22 19:36:56
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
目的：了解进程和线程的基本概念。了解操作系统线程模型的演化，加深对线程概念的理解。

回顾之前做的笔记：
[操作系统之线程管理](http://www.zhaokongnuan.com/2015/01/09/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E4%B9%8B%E7%BA%BF%E7%A8%8B%E7%AE%A1%E7%90%86/)
[操作系统之进程管理](http://www.zhaokongnuan.com/2015/01/08/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E4%B9%8B%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86/)

多核多线程  查看多核多线程
top ps -efL pstree -c | grep 进程名字

#课堂作业#
<b>1、Linux操作系统中ps命令如何查看线程？其中输出的各列的含义都是什么？</b>
```bash
[zkn@hadoop1 Desktop]$ ps -aux
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.9  0.1  19352  1540 ?        Ss   08:26   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    08:26   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    08:26   0:00 [migration/0]
root         4  0.0  0.0      0     0 ?        S    08:26   0:00 [ksoftirqd/0]
……
```
USER：该程序的属主
PID：进程的ID
%CPU：进程占用的CPU百分比
%MEM：占用内存的百分比
VSZ： 该进程使用的虚拟內存量（KB）
RSS：  该进程占用的固定內存量（KB）（驻留中页的数量）
TTY：是否为登入者执行的程序？若为 tty1-tty6 则为本机登入者，若为 pts/?? 则为远程登入者执行的程序
STAT：该程序的状态（ "S":进程处在睡眠状态,表明这些进程在等待某些事件发生--可能是用户输入或者系统资源的可用性）
START：该程序启动时间
TIME：该程序运行的时间
COMMAND：命令的名称和参数
```bash
[zkn@hadoop1 Desktop]$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 08:26 ?        00:00:02 /sbin/init
root         2     0  0 08:26 ?        00:00:00 [kthreadd]
root         3     2  0 08:26 ?        00:00:00 [migration/0]
root         4     2  0 08:26 ?        00:00:00 [ksoftirqd/0]
root         5     2  0 08:26 ?        00:00:00 [migration/0]
root         6     2  0 08:26 ?        00:00:00 [watchdog/0]
root         7     2  0 08:26 ?        00:00:00 [events/0]
root         8     2  0 08:26 ?        00:00:00 [cgroup]
root         9     2  0 08:26 ?        00:00:00 [khelper]
root        10     2  0 08:26 ?        00:00:00 [netns]
……
```
<b>2、操作系统线程模型有哪些？各自的含义是什么？</b>
1) 多对一模型
将多个用户级线程映射到一个内核级线程，线程管理在用户空间完成。
此模式中，用户级线程对操作系统不可见（即透明）。
优点：线程管理是在用户空间进行的，因而效率比较高。
缺点：当一个线程在使用内核服务时被阻塞，那么整个进程都会被阻塞；多个线程不能并行地运行在多处理机上。
2) 一对一模型
将每个用户级线程映射到一个内核级线程。
优点：当一个线程被阻塞后，允许另一个线程继续执行，所以并发能力较强。
缺点：每创建一个用户级线程都需要创建一个内核级线程与其对应，这样创建线程的开销比较大，会影响到应用程序的性能。
3) 多对多模型
将 n 个用户级线程映射到 m 个内核级线程上，要求 m <= n。
特点：在多对一模型和一对一模型中取了个折中，克服了多对一模型的并发度不高的缺点，又克服了一对一模型的一个用户进程占用太多内核级线程，开销太大的缺点。又拥有多对一模型和一对一模型各自的优点，可谓集两者之所长。
<b>3、Linux进程状态有哪些？各状态含义是什么？</b>
Linux进程状态：R (TASK_RUNNING)，可执行状态。
Linux进程状态：S (TASK_INTERRUPTIBLE)，可中断的睡眠状态。
Linux进程状态：D (TASK_UNINTERRUPTIBLE)，不可中断的睡眠状态。
Linux进程状态：Z (TASK_DEAD - EXIT_ZOMBIE)，退出状态，进程成为僵尸进程。
Linux进程状态：T (TASK_STOPPED or TASK_TRACED)，暂停状态或跟踪状态。
Linux进程状态：X (TASK_DEAD - EXIT_DEAD)，退出状态，进程即将被销毁。 

