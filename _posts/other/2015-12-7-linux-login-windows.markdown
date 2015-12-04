---
layout: article
title:  "通过外网linux主机登录到内网windows主机"
date:   2015-12-07 09:20:00 +0800
categories: other
---

如果公司内网有一台windows，外网有linux主机，你在家的时候，你想通过远程桌面（mstsc）登录你公司内网的windows主机？很简单，看以下教程。


----------

# 首先介绍一下需要什么，环境是什么。 #

环境：windows一台（内网IP：192.168.1.10）、Linux主机一台（外网IP：10.10.10.10）

软件：Portfwd-0.29.tar.gz编译文件一个、ssh-portfwd文件夹一个（网上有好多，去下载哦）

需求：要求通过外网能够mstsc到内网的windows

----------

# 具体操作步骤如下 #

1.首先需要把ssh-portfwd文件夹拷贝到windows上，然后更改ssh.bat文件，改成如下内容。

@echo off
taskkill /F /IM plink.exe
plink.exe -C  -v -N -R 10.10.10.10:8087:192.168.1.10:3389 -l user -pw "password"  10.10.10.10 -P 3389

首先，更改一下IP，以及远程linux主机的“user”OR“password”，然后执行这个脚本，即可把本地的3389端口映射到远程linux主机的8087端口。

在linux上执行netstat –lntp 即可查看到，相应端口为127.0.0.1:8087的ssh东东。

linux ~$ netstat -ltnp | grep 8087
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:8087          0.0.0.0:*               LISTEN      - 

但是，这个脚本并不稳定的，所以要用到windows计划任务，把这个脚本加进去，进行每五分钟执行一次，这样windows上就操作完毕了。

2.之后就把Portwd-0.29.tar.gz拷贝到linux上进行编译安装，然后更改配置文件

linux ~$ cat /etc/portfwd.cfg 
tcp { 8086
{ => 127.0.0.1:8087 } }

把127.0.0.1:8087端口映射到8086端口

之后执行

/usr/local/sbin/portfwd -c /etc/portfwd.cfg

就把这个配置文件执行了，然后就出现下面的0.0.0.0:8086

linux ~$ netstat -ltnp | grep 8086
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:8086            0.0.0.0:*               LISTEN      -  

3.这个时候外网用mstsc就可以登录了
