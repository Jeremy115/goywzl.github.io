---
layout: article
title:  "Samba server"
date:   2015-12-28 09:20:00 +0800
categories: server
---


## 一．Samba服务基础 ##

SMB协议

     Server Message Block，服务消息块

CIFS协议

     Common Internet File System，通用互联网文件系统

Samba的软件包组成

      samba-3.0.23c-2.i386.rpm
      samba-client-3.0.23c-2.i386.rpm
      samba-common-3.0.23c-2.i386.rpm
      samba-swat-3.0.23c-2.i386.rpm
      system-config-samba-1.2.39-1.el5.noarch.rpm

注意：

1.只需安装samba、samba-client、samba-common这3个软件包，即可实现Samba服务器和客户端的基本功能

2.在以上5个软件包中，samba和samba-client软件包分别用于提供服务器和客户端程序文件，samba-common软件包提供了服务器和客户端都需要使用的公共文件，system-config-samba软件包用于提供图形界面管理程序（需要在X图形环境中使用），而samba-swat软件包是一个Web方式的管理工具

Samba服务器的主要程序

      smbd：提供对服务器中文件、打印资源的共享访问
      nmbd：提供基于NetBIOS主机名称的解析

Samba的服务脚本

{% highlight bash %}
{% raw %}

      /etc/init.d/smb

{% endraw %}
{% endhighlight %}

Samba的配置目录及文件

{% highlight bash %}
{% raw %}

      /etc/samba/
      /etc/samba/smb.conf
 
{% endraw %}
{% endhighlight %}

## 二．smb.conf主配置文件（/ec/samba/smb.conf） ##

{% highlight bash %}
{% raw %}

[global]
   workgroup = MYGROUP  所在工作组名称
   server string = Samba Server   服务器描述信息
 security = user   安全级别，可用值如下：share、user、server、domain
   log file = /var/log/samba/%m.log  日志文件位置，“%m”变量表示客户机地址
   max log size = 50  日志文件的最大容量，单位为KB
   ……
[homes]
   comment = Home Directories
   browseable = no
   writable = yes
[printers]
   ……
  
{% endraw %}
{% endhighlight %}

## 三．常见共享目录配置项的含义 ##

      comment：对共享目录的注释、说明信息
      path：共享目录在服务器中对应的实际路径
      browseable：该共享目录在“网上邻居”中是否可见
      guest ok：是否允许所有人访问，等效于“public”
      writable：是否可写，与read only的作用相反
      read only = yes 只读     
      create mask = 755   创建文件时 权限是755
      only guest = yes   仅以匿名用户登录
      valid users = liu,@liu  只运行liu用户可以访问(@liu,运行liu这个组访问)
 
## 四．建立可匿名访问的文件共享 ##

修改smb.conf配置文件

{% highlight bash %}
{% raw %}

      security = share
      public = yes
  
{% endraw %}
{% endhighlight %}

检查配置的正确性

{% highlight bash %}
{% raw %}

      testparm命令工具
  
{% endraw %}
{% endhighlight %}

启动smb服务

{% highlight bash %}
{% raw %}

      service smb start
  
{% endraw %}
{% endhighlight %}

例如：建立一个只读的匿名共享

修改/etc/samba/smb.conf,修改如下内容：

{% highlight bash %}
{% raw %}

[global]
   workgroup = WORKGROUP
   security = share
[movie]
   comment = Public share with movie files
   path = /var/public/movies
   public = yes
   read only = yes
  
{% endraw %}
{% endhighlight %}
 
## 五．建立带验证的文件共享 ##
 
建立Samba用户数据库文件

      默认数据库文件位于：/etc/samba/smbpasswd （rhel5默认没有）
       （ 注意： 注释掉passdb backend = tdbsam，

然后加入smb passwd file = /etec/samba/smbpasswd）

      系统用户帐号 -> Samba用户帐号（samba帐户必须是一个系统帐户）

Samba帐号的别名设置

      在smb.conf文件中需要启用如下配置

       username map = /etc/samba/smbusers

      默认的别名映射文件：/etc/samba/smbusers

格式： 真实名字= 别名1   别名2   别名3  

例如：建立一个只要vina和root组可以登陆的samba共享

{% highlight bash %}
{% raw %}

[global]
   workgroup = WORKGROUP
   security = user
[movie]
   comment = Public share with movie files
   path = /var/public/movies
   public = no
   read only = no
   valid users = vina, @root
   write list = root
   directory mask = 0744   创建目录的权限
   create mask = 0600      创建文件的权限
  
{% endraw %}
{% endhighlight %}

smb.conf文件设置客户机访问授权

      一般用在全局配置[global]部分
      hosts allow配置项：仅允许特定的客户机
      hosts deny配置项：仅拒绝特定的客户机
      客户机地址表示形式：
      以空格分隔多个地址
      主机名或IP地址，例如： 192.168.168.11 或者 prtsvr
      网络地址，例如：173.17. 或者 173.17.0.0/255.255.0.0
 
## 六．在客户机中访问共享目录 ##

使用Windows客户端访问文件共享服务

      网上邻居、UNC路径

使用Linux客户端访问文件共享服务

      smbclient命令，查看及登录使用共享
      smbclient -L 192.168.168.1
      smbclient -U vina //192.168.168.1/movie
      mount命令，将共享目录挂载到本地使用
      mount -o username=vina //192.168.168.1/movie /mnt