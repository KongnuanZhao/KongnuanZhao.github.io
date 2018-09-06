title: LoadRunner中的Controller和Analysis
date: 2015-03-15 09:57:27
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
##Controller##
###Controller的引入###
* 需要Controller的原因？
做性能测试需要模拟比如一个网站数以万计的用户级的数量，如果要手工测试这是不可能的，所以需要LoadRunner来完成。Controller用来管理和维护场景，可以在一台工作站控制一个场景中的所有虚拟用户Vuser。执行场景时，Controller会将该场景中的每个Vuser分配给一个负载生成器。负载生成器执行脚本，从而使Vuser可以模拟真实用户操作计算机。解决了手工操作不同步和人力、物力、资源浪费的问题。
* Controller的启动方式
	* 【LoadRunner Launcher】 → 【Run Load Tests】
	* 在开始菜单中
	* 在VuGen中

* Design视图和Run视图
①设计视图:设计视图显示场景中的所有Vuser组/脚本的列表、负载生成器(Load Generator)计算机一级分配给每个组/脚本的Vuser数。该视图还显示有关场景计划(手动场景)或目标(面向目标的场景)的基本信息。
![lrdesign](/image/lrdesign.png)
在设计视图中，场景有两种类型：手工场景和面向目标场景，其中手工场景有百分比模式和组模式。
	* 手工场景(Manual Scenario):创建虚拟用户组，设置虚拟用户数目以及其他Run-time信息。
	* 面向目的场景(Goal-Oriented Scenario)：在面向目标场景中，先定义测试要达到的目标，然后LoadRunner自动基于这些目标创建场景，运行过程中，会不断地把结果和目标相比较，以决定下一步怎么走。

②运行视图:场景一旦开始运行，Controller自动切换到运行视图，运行视图显示有关运行的Vuser和Vuser组的信息以及联机监视器图。
![lrrun](/image/lrrun.png)
	

* Controller中的运行时设置
	* 与VuGen中的不同，各自有各自的，不要认为是同一个
	* Think time默认设置的不同
	* 系统日志，真正运行的时候就设置成仅仅当错误时才发送，提高效率。
	* 事务的设置
	* 带宽的设置

* Run视图解析
	* Schedule by 和 Run mode解析
	<table><caption>Schedule by解析</caption><tr><th style="width:20%">Scenario(场景)</th><td>当按场景计划执行时，Controller会同时运行所有参与场景的Vuser组，Controller会将操作按比例应用于所有的Vuser。</td></tr><tr><th>Group(组)</th><td>当按组计划执行时，参与场景的每个Vuser组按其自己单独计划运行。对于每个Vuser组，可以指定合适开始运行，在制定的时间间隔内开始和停止运行运行组中多少个Vuser,以及改组应该继续运行多长时间。比如订票系统这种场景时存在业务处理先后顺序，这个时候就需要用到组方式，只有注册用户业务先完成后，才能执行订票业务。</td></tr></table>
	<table><caption>Run mode解析 </caption><tr><th style="width:20%">Real-World Schedule(实际计划)</th><td>Vuser组根据设置定义来运行，但是可以自己定义每次运行多少个Vuser,Vuser应该持续运行多长时间以及每次多少个Vuser停止运行。</td></tr><tr><th>Basic schedule(基本计划)</th><td>可以计划一次运行多少个Vuser,以及停止之前应运行多长时间。</td></tr></table>
	* Run视图解析
		*  init状态解析

###负载生成器###
* 为什么有
负载是需要消耗系统资源的，如：CPU、内存、磁盘空间等，模拟越多的用户，也就意味着消耗更多的资源，当一台机器资源模拟不了太多的虚拟用户时，负载机就成为了性能测试的瓶颈。针对这个问题，LoadRunner提供了负载生成器（LoadGenerator）来解决。Load Generator是在场景运行中运行虚拟用户脚本的计算机。它将负载的虚拟用户分配给多个负载机，利用这些机器的硬件资源模拟大量的虚拟用户对被测系统施加更大的压力。
* 负载生成器和Controller的关系
Controller发号施令，Load Generator负责实施和执行。一个Controller可以控制多台机器上的Load Generator，只需要在其他机器上装有Load Generator启动agent并在网络上找到它，让他们听从指挥，共同完成任务。负载生成器(Load Generators)的使用要保证负载生成器自己不要成为瓶颈。
* 负载机器的软硬件配置
硬件环境（PC、笔记本电脑、服务器、小型机、大型机等）。
软件环境（操作系统，如win200、winXp,win7,winNT,UNIX，Linux等；web服务器，如Tomcat、Weblogic、IIS、WebSphere
等；数据库，如Oracle、SQL Server、MySQL、DB2等；还有一些其他软件，如办公软件，杀毒软件等）。软件环境的配置还需呀考虑软件的具体版本和补丁的安装情况。
* 脚本的复杂程度
* 负载生成器的连接
添加Load Generator后，执行"Connect"操作，使Status为Ready，表示该机器连接正常了如果为Failed，表示该机器不能连接，要检查原因。
![loadGenerators](/image/loadGenerators.png)
* mmdrv解析
Controller使用驱动程序mmdrv运行Vuser.可以在controller的【run-time settiing】中选择Vuser的运行方式，是多进程还是多线程的方式。
![process-thread](/image/process-thread.png)
	* 如果选择以线程方式运行：假设并发用户为30，则只启动一个线程。最多一个线程最大用户数是50。也就是说如果并发用户为125则是三个进程。
	* 如果选择以进程的方式运行：假设并发用户为30，则启动30个进程。
	* 在启动了IP欺骗之后，所需启动的进程数，还与选择的是按进程还是线程来分配IP地址有关，下面介绍的是IP欺骗。
		* 按进程分IP，每个ip就需要启动一个进程。
		* 按线程分IP，每个ip就不需要启动一个进程。

###IP Spoofer和集合点###
* IP欺骗（IP Spoofer）
	* 为什么需要IP Spoofer？
	默认情况下，同一个Load Generator上的所有虚拟用户都是用该Generator的IP地址来访问服务器，即在同一个负载生成器计算机上的Vuser具有相同的IP地址，当有大量虚拟用户并发运行时，就会出现多个用户使用同一个IP地址对网站进行加压的的情况。与此同时，应用程序服务器经常缓存来自同一台计算机上的客户端信息，而路由器则缓存源信息和目标信息，以提高处理能力。经过服务器和路由器进行优化处理后，Load Generator产生的压力可能无法反映真实的情况，尤其遇到服务器对同一IP访问进行限制的情况，将会导致每个Load Generator仅能创建一个虚拟用户，除非修改服务器配置。
	LaodRunner可以通过IP Spoofer技术来解决上述问题，以保证每个虚拟用户使用自己的IP地址来访问服务器，IP Spoofer也称为“IP 欺骗”。
	其实，在大多数性能测试过程中，使用多个IP地址和使用同一个IP对网站进行访问并不影响实际被测应用的性能表现，但在某些情况下，使用多个IP地址和使用一个IP地址对应用会造成不同的运行情况，此时就需要更加真实的模拟，在同一台LaodGenerator Machine上，让每个虚拟用户使用不同的IP地址。
	* 必须使用IP Spoofer的情况
		* 网站采用了“根据IP确定负载分布”的负载均衡方式。
		* 出于安全或者公平目的，网站限制同一个IP地址只能在网站上产生有限个任务。（如投票）

	* 如何使用IP Spoofer？
		* Load Generator Machine必须使用静态IP地址，而不是使用通过DHCP自动分配的IP地址。
		* 为了使LoadRunner能使用这些IP地址，还需要在Controller中对场景进行设置，须选中菜单【Scenario】—>【EnableIP Spoofer】选项。
		* 打开菜单【Tools】—>【Expert Mode】，进入专家模式，对多个IP地址进行全局设置，进入【Tools】—>【Options】中的“General”选项卡，根据虚拟用户的情况来配置IP地址的加载方式。如果虚拟用户按线程启动，则选择为每个线程分配一个IP；如果虚拟用户按进程来运行，则选择为每个进程（含50个线程）分配一个IP。

	* IP Spoofer配置步骤：（可以手动在机器上配置多个IP，但是太繁琐了）单击【开始】—>【程序】—>【HP LoadRunner】—>【Tools】—>【IP Wizard】，打开设置向导进行设置。

* Controller中的集合点
Controller默认的集合点策略是：在所有Running状态的Vuser达到集合点后才释放。如果要改变集合点策略，步骤如下：
在【Scenario】菜单中选中【Rendezvous】—> 打开设定同步点的详细设置对话框 —> 单击【policy】按钮，进入策略设置窗口。
Policy有三个选项：
	* 第一个选项表示所有的用户到达集合点之后，再允许等待的用户继续场景执行。
	* 第二个选项表示所有正在运行的用户到达集合点之后，再允许等待的用户继续场景执行。
	* 第三个选项表示当指定的目的用户到达集合点之后，就允许等待的用户继续场景执行。（Timeout的设定表示，当第一个用户到达集合点后，等待30秒，如果30秒内上面三个选项设定的释放条件满足，就继续执行场景；30秒后，没有满足上述释放条件。也不再等待，开始释放等待的用户，继续场景执行。）

* 自定义数据采集
	* lr_user_data_point解析

##Analysis##
* 首先必须明确：光靠Analysis是不行的，只要能通过Analysis分析出部分问题就已经很不错了，善于利用它才是最关键的。
* 如何启动Analysis？
	* 【LoadRunner Launcher】 → 【Analyze Test Results】
	* 在开始菜单中
	* 在VuGen中
* Analysis中的Analysis Summary介绍
* Analysis中各常用图表介绍
* 详解Web Page Diagnostics
* 图表的合并（必须X轴的单位是一致的（X轴的度量单位一致））
* 拐点
![point](/image/point.jpg)

#课堂作业#
<b>1、详细阐述Controller中面向目的的场景的含义、运行原理和使用方法。</b>
面向目的场景(Goal-Oriented Scenario)：在面向目标场景中，先定义测试要达到的目标，然后LoadRunner自动基于这些目标创建场景，运行过程中，会不断地把结果和目标相比较，以决定下一步怎么走。
* 使用方法
单击【新建】 ，打开【新建场景】对话框。
![newScenario](/image/newScenario.png)
Controller 面向目的窗口的【Design视图】包含两个主要部分：
* 场景目标：可以看到测试目标、达到该目标要使用的用户数、场景持续时间和负载行为。使用【编辑场景目标】对话框可以对目标设置进
行定义。
![scenariogoal](/image/scenariogoal.png)
* 场景脚本：可以确定 Vuser 脚本、脚本路径、分配到每个脚本的目标的百分比以及负载生成器计算机。可以在此处对场景设置进行配置。
* 如何定义目标？
![editscenariogoal](/image/editscenariogoal.png)
①：目标类型有五种类型：虚拟用户数、每秒单击次数、每秒事务数、每分钟页面数或事务响应时间（主要用于衡量要达到预期的事务响应时间，系统所容纳的最多用户数，如果系统已经加载了最大的用户数，响应时间仍然低于设定的值，说明系统还有能力容纳更多的用户，如果使用一部分用户就达到了设定的响应时间，说明系统是无法容纳设定的最大数量的用户的，必须通过完善应用程序来达到目的。）
②：到达目标，如目标设定为每秒点击次数为100次
③：设置要在LoadRunner上运行的Vuser数目的最大值和最小值。（首先加载最小的用户数，如果使用最小的用户数不能达到目标，系统会自动逐渐增加用户直到能够达到设定的目标为止。当加载的用户数达到最大值仍然不能满足要求时，就需要重新设置场景，增加最大用户数。）
④⑤：指定测试在达到目标后继续运行n分钟，并选择“继续运行场景，无需达到目标”。
⑥：请勿使用录制的思考时间。如果启用该选项， LoadRunner 将使用脚本中录制的思考时间运行场景，这样您可能需要通过增加场景中的 Vuser 数来达到目标。

<b>2、IP欺骗的使用场景是什么？在LoadRunner中如何使用IP欺骗？</b>
使用IP Spoofer的场景
* 网站采用了“根据IP确定负载分布”的负载均衡方式。
* 出于安全或者公平目的，网站限制同一个IP地址只能在网站上产生有限个任务。（如投票）

如何使用IP Spoofer？
* Load Generator Machine必须使用静态IP地址，而不是使用通过DHCP自动分配的IP地址。
* 为了使LoadRunner能使用这些IP地址，还需要在Controller中对场景进行设置，须选中菜单【Scenario】—>【EnableIP Spoofer】选项。
* 打开菜单【Tools】—>【Expert Mode】，进入专家模式，对多个IP地址进行全局设置，进入【Tools】—>【Options】中的“General”选项卡，根据虚拟用户的情况来配置IP地址的加载方式。如果虚拟用户按线程启动，则选择为每个线程分配一个IP；如果虚拟用户按进程来运行，则选择为每个进程（含50个线程）分配一个IP。
* IP Spoofer配置步骤：（可以手动在机器上配置多个IP，但是太繁琐了）单击【开始】—>【程序】—>【HP LoadRunner】—>【Tools】—>【IP Wizard】，打开设置向导进行设置。

<b>3、Web Page Diagnostics中请求各阶段都有哪些？它们的具体含义是什么？</b>
Connections Time：连接服务器所需要的时间。TCP三次握手的时间。
DNS Resolution Time：通过DNS服务器解析域名所需要的时间，解析受到DNS服务器的影响。
Error Time：服务器返回错误响应时间，这个时间反映了服务器处理错误的速度，一般是Web服务器直接返回的，包含了网络时间和Web服务器返回错误的时间。
First Buffer Time：连接到服务器，从我们发出请求到服务器返回第一个字节所需要的时间，反映了系统对于正常请求的处理时间开销，包含网络时间和服务器正常处理的时间。
FTP Authentication Time ：FTP认证时间，这是进行FTP登录等操作所需要消耗的认证时间。
Receive Time：接受数据的时间，这个时间反映了带宽的大小，带宽越大，下载时间越短。
SSL Handshaking Time：SSL加密握手的时间，当发出https时才会进行SSL加密处理。
Receive Time:接收返回的时间。
Client Time：客户端浏览接收所需要使用的时间，可以不用考虑。

<b>4、详细阐述拐点的含义。并说明如何在图表中识别出拐点？</b>
基本思想就是性能产生瓶颈的主要原因就是因为某个资源的使用达到了极限，此时表现为随着压力的增大，系统性能却出现急剧下降，这样就产生了"拐点"现象。当得到"拐点"附近的资源使用情况时，就能定位出系统的性能瓶颈。"拐点分析"方法举例，如系统随着用户的增多，事务响应时间缓慢增加，当用户数达到100个虚拟用户时，系统响应时间急剧增加，表现为一个明显的"折线"，这就说明了系统承载不了如此多的用户做这个事务，也就是存在性能瓶颈。


---------------

学习的课程来自[炼数成金](http://www.dataguru.cn/)广州八神王磊老师的软件性能测试（第二期）^_^