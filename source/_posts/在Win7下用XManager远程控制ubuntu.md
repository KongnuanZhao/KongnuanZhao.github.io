title: 在Win7下用XManager远程控制ubuntu
date: 2015-01-02 18:44:18
author: 赵空暖
tags: linux
categories: linux
---
在VMware Player 装完Ubuntu 12.04 就巴掌大的界面，很不舒服。
所以用Xmanager来远程连接桌面
1.在12.04上安装GNOME
```bash
sudoapt-get install gnome-panel
```
安装完成后注销用户，登陆的时候选择以GNOME界面模式登录
![ubuntu](/image/ubuntu.png)
![Ubuntu1](/image/Ubuntu1.png)
登陆进入GNOME界面，选择桌面共享然后进行如下设置：
![Ubuntu2](/image/Ubuntu2.png)
![Ubuntu3](/image/Ubuntu3.png)
2.Ubuntu下GDM的安装
首先将防火墙关闭
```bash
sudo ufw disable
```
Win7远程连接上Ubuntu，所使用的协议是rdp，如果没有先装上去
```bash
sudo apt-get install xrdp
sudo apt-get install vnc4server tightvncserver
```
安装gdm包
```bash
sudo apt-get install gdm
```
确认gdm是否安装上去了
```bash
$ gdm –version
```
修改custom.conf配置文件：
```bash
$ sudo gedit /etc/gdm/custom.conf
```
添加如下两个字段：
```xml
[security]
DisallowTCP=false
[xdmcp]
Enable=true
Port=177
DisplaysPerHost=10  
//注：DisplaysPerHost表示显示主机的数量
```
3、修改schemas配置文件
```bash
sudo gedit /etc/gdm/gdm.schemas
```
修改xdmcp/Enable字段：
```xml
<schema>
<key>xdmcp/Enable</key>
<signature>b</signature>
<default>true</default>
</schema> 
```
开启177端口，保证177端口可用
```bash
sudo ufw allow 177 
```
重启gdm
```bash
sudo /etc/init.d/gdm restart
```
在windows系统上运行xmanager4里的Xbrowser程序在里面新建一个Xmanager Session在Host这里输入ip地址其它配置都不要改变包括端口号确定退出。然后双击这个New Xmanager Session就可以看到登录界面
![Ubuntu4](/image/Ubuntu4.png)
还是大桌面爽~
![Ubuntu5](/image/Ubuntu5.png)
