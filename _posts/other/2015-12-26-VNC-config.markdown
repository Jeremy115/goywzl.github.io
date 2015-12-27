---
layout: article
title:  "VNC配置"
date:   2015-12-26 09:20:00 +0800
categories: other
---



很简单的一个VNC的配置

安装VNC后 执行 vncserver 命令 配置密码

执行 service vncserver restart 命令重启网卡

编辑

{% highlight bash %}
{% raw %}

vi /root/.vnc/xstartup
#!/bin/sh
# Uncomment the following two lines for normal desktop:
unset SESSION_MANAGER
exec /etc/X11/xinit/xinitrc
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
xterm -geometry 80×24+10+10 -ls -title "$VNCDESKTOP Desktop" &
twm &

{% endraw %}
{% endhighlight %}

重启VNC服务即可！

----------

上述讲的比较简单，下面有详细介绍
 
## 1.确认VNC是否安装 ##

默认情况下，Red Hat Enterprise Linux安装程序会将VNC服务安装在系统上。

确认是否已经安装VNC服务及查看安装的VNC版本

{% highlight bash %}
{% raw %}

[root@testdb ~]# rpm -q vnc-server
vnc-server-4.1.2-9.el5


{% endraw %}
{% endhighlight %}

若系统没有安装,可以到操作系统安装盘的Server目录下找到VNC服务的RPM安装包vnc-server-4.1.2-9.el5.x86_64.rpm，安装命令如下

{% highlight bash %}
{% raw %}

rpm -ivh /mnt/Server/vnc-server-4.1.2-9.el5.x86_64.rpm

{% endraw %}
{% endhighlight %}

## 2.启动VNC服务 ##

使用vncserver命令启动VNC服务，命令格式为“vncserver :桌面号”，其中“桌面号”用“数字”的方式表示，每个用户连个需要占用1个桌面
启动编号为1的桌面示例如下

{% highlight bash %}
{% raw %}

[root@testdb ~]# vncserver :1
You will require a password to access your desktops.
Password:
Verify:
xauth:  creating new authority file /root/.Xauthority
New 'testdb:1 (root)' desktop is testdb:1
Creating default startup script. /root/.vnc/xstartup
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/testdb:1.log

{% endraw %}
{% endhighlight %}

以上命令执行的过程中，因为是第一次执行，需要输入密码，这个密码被加密保存在用户主目录下的.vnc子目录（/root/.vnc/passwd）中；

同时在用户主目录下的.vnc子目录中为用户自动建立xstartup配置文件（/root/.vnc/xstartup），在每次启动VND服务时，都会读取该文件中的配置信息。

BTW：/root/.vnc/目录下还有一个“testdb:1.pid”文件，这个文件记录着启动VNC后对应后天操作系统的进程号，用于停止VNC服务时准确定位进程号。

## 3.VNC服务使用的端口号与桌面号的关系 ##

VNC服务使用的端口号与桌面号相关，VNC使用TCP端口从5900开始，对应关系如下

    桌面号为“1”  —- 端口号为5901
    桌面号为“2”  —- 端口号为5902
    桌面号为“3”  —- 端口号为5903
    ……

基于Java的VNC客户程序Web服务TCP端口从5800开始，也是与桌面号相关，对应关系如下

    桌面号为“1”  —- 端口号为5801
    桌面号为“2”  —- 端口号为5802
    桌面号为“3”  —- 端口号为5803
    ……

基于上面的介绍，如果Linux开启了防火墙功能，就需要手工开启相应的端口，以开启桌面号为“1”相应的端口为例，命令如下

{% highlight bash %}
{% raw %}

[root@testdb ~]# iptables -I INPUT -p tcp –dport 5901 -j ACCEPT
[root@testdb ~]# iptables -I INPUT -p tcp –dport 5801 -j ACCEPT

{% endraw %}
{% endhighlight %}

## 4.测试VNC服务 ##

第一种方法是使用VNC Viewer软件登陆测试，操作流程如下

    启动VNC Viewer软件 –> Server输入“144.194.192.183:1” –> 点击“OK” –> Password输入登陆密码 –> 点击“OK”登陆到X-Window图形桌面环境 –> 测试成功

第二种方法是使用Web浏览器（如Firefox,IE,Safari）登陆测试，操作流程如下

    地址栏输入http://144.194.192.183:5801/ –> 出现VNC viewer for Java（此工具是使用Java编写的VNC客户端程序）界面，同时跳出VNC viewer对话框，在Server处输入“144.194.192.183:1”点击“OK” –> Password输入登陆密码 –> 点击“OK”登陆到X-Window图形桌面环境 –> 测试成功
    
（注：VNC viewer for Java需要JRE支持，如果页面无法显示，表示没有安装JRE，可以到http://java.sun.com/javase/downloads/index_jdk5.jsp这里下载最新的JRE进行安装）

## 5.配置VNC图形桌面环境为KDE或GNOME桌面环境 ##

如果您是按照我的上面方法进行的配置的，登陆到桌面后效果是非常简单的，只有一个Shell可供使用，这是为什么呢？怎么才能看到可爱并且美丽的KDE或GNOME桌面环境呢？回答如下

之所以那么的难看，是因为VNC服务默认使用的是twm图形桌面环境的，可以在VNC的配置文件xstartup中对其进行修改，先看一下这个配置文件

{% highlight bash %}
{% raw %}

[root@testdb ~]# cat /root/.vnc/xstartup
#!/bin/sh
# Uncomment the following two lines for normal desktop:
# unset SESSION_MANAGER
# exec /etc/X11/xinit/xinitrc
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
xsetroot -solid grey
vncconfig -iconic &
xterm -geometry 80×24+10+10 -ls -title "$VNCDESKTOP Desktop" &
twm &

{% endraw %}
{% endhighlight %}

将这个xstartup文件的最后一行修改为“startkde &”，再重新启动vncserver服务后就可以登陆到KDE桌面环境

将这个xstartup文件的最后一行修改为“gnome-session &”，再重新启动vncserver服务后就可以登陆到GNOME桌面环境

重新启动vncserver服务的方法：

{% highlight bash %}
{% raw %}

[root@testdb ~]# vncserver -kill :1
[root@testdb ~]# vncserver :1

{% endraw %}
{% endhighlight %}

## 6.配置多个桌面 ##

可以使用如下的方法启动多个桌面的VNC

    vncserver :1
    vncserver :2
    vncserver :3
    ……

但是这种手工启动的方法在服务器重新启动之后将失效，因此，下面介绍如何让系统自动管理多个桌面的VNC，方法是将需要自动管理的信息添加到/etc/sysconfig/vncservers配置文件中，先以桌面1为root用户桌面2为oracle用户为例进行配置如下：

格式为：VNCSERVERS="桌面号:使用的用户名桌面号:使用的用户名"

{% highlight bash %}
{% raw %}

[root@testdb ~]# vi /etc/sysconfig/vncservers
VNCSERVERS="1:root 2:oracle"
VNCSERVERARGS[1]="-geometry 1024×768"
VNCSERVERARGS[2]="-geometry 1024×768"

{% endraw %}
{% endhighlight %}

## 7.修改VNC访问的密码 ##

使用命令vncpasswd对不同用户的VNC的密码进行修改，一定要注意，如果配置了不同用户的VNC需要分别到各自用户中进行修改，例如在我的这个实验中，root用户和oracle用户需要分别修改，修改过程如下：

{% highlight bash %}
{% raw %}

[root@testdb ~]# vncpasswd
Password:
Verify:
[root@testdb ~]#

{% endraw %}
{% endhighlight %}

## 8.启动和停止VNC服务 ##

### 1）启动VNC服务命令 ###

{% highlight bash %}
{% raw %}

[root@testdb ~]# /etc/init.d/vncserver start
Starting VNC server: 1:root
New 'testdb:1 (root)' desktop is testdb:1
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/testdb:1.log
2:oracle
New 'testdb:2 (oracle)' desktop is testdb:2
Starting applications specified in /home/oracle/.vnc/xstartup
Log file is /home/oracle/.vnc/testdb:2.log
                                                           [  OK  ]

{% endraw %}
{% endhighlight %}

### 2）停止VNC服务命令 ###

{% highlight bash %}
{% raw %}

[root@testdb ~]# /etc/init.d/vncserver stop
Shutting down VNC server: 1:root 2:oracle                  [  OK  ]

{% endraw %}
{% endhighlight %}

### 3）重新启动VNC服务命令 ###

{% highlight bash %}
{% raw %}

[root@testdb ~]# /etc/init.d/vncserver restart
Shutting down VNC server: 1:root 2:oracle                  [  OK  ]
Starting VNC server: 1:root
New 'testdb:1 (root)' desktop is testdb:1
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/testdb:1.log
2:oracle
New 'testdb:2 (oracle)' desktop is testdb:2
Starting applications specified in /home/oracle/.vnc/xstartup
Log file is /home/oracle/.vnc/testdb:2.log
                                                           [  OK  ]

{% endraw %}
{% endhighlight %}

### 4）设置VNC服务随系统启动自动加载 ###

第一种方法：使用“ntsysv”命令启动图形化服务配置程序，在vncserver服务前加上星号，点击确定，配置完成。

第二种方法：使用“chkconfig”在命令行模式下进行操作，命令使用如下（预知chkconfig详细使用方法请自助式man一下）

{% highlight bash %}
{% raw %}

[root@testdb ~]# chkconfig vncserver on
[root@testdb ~]# chkconfig –list vncserver
vncserver       0:off   1:off   2:on    3:on    4:on    5:on    6:off

{% endraw %}
{% endhighlight %}
