---
layout: article
title:  "Ftp server"
date:   2015-12-29 09:20:00 +0800
categories: server
---


一．FTP服务概述

FTP连接方式

      控制连接：标准端口为21，用于发送FTP命令信息
      数据连接：标准端口为20，用于上传、下载数据
      数据连接的建立类型：

主动模式：服务端从20端口主动向客户端发起连接

被动模式：服务端在指定范围内的某个端口被动等待客户端发起连接

FTP传输模式

      文本模式：ASCII模式，以文本序列传输数据
      二进制模式：Binary模式，以二进制序列传输数据

FTP用户的类型

      匿名用户：anonymous或ftp
      本地用户：
		帐号名称、密码等信息保存在passwd、shadow文件中
      虚拟用户：
   		使用独立的帐号/密码数据文件

常见的FTP服务器程序

      IIS、Serv-U
      wu-ftpd、Proftpd
      vsftpd（Very Secure FTP Daemon）

常见的FTP客户端程序

      ftp命令
      CuteFTP、FlashFXP、LeapFTP、Filezilla
      gftp、kuftp
 
二．vsftpd服务基础

vsftpd软件包

    官方站点:http://vsftpd.beasts.org/
    主程序：/usr/sbin/vsftpd
    服务名：vsftpd
    用户控制列表文件
	  /etc/vsftpd/ftpusers
	  /etc/vsftpd/user_list
    主配置文件
	  /etc/vsftpd/vsftpd.conf

常用的全局配置项

      listen=YES：是否以独立运行的方式监听服务
      listen_address=192.168.4.1：设置监听的IP地址
      listen_port=21：设置监听FTP服务的端口号
      write_enable=YES：是否启用写入权限
      download_enable＝YES：是否允许下载文件
      userlist_enable=YES：是否启用user_list列表文件
      userlist_deny=YES：是否禁用user_list中的用户
      max_clients=0：限制并发客户端连接数
      max_per_ip=0：限制同一IP地址的并发连接数

常用的匿名FTP配置项

      anonymous_enable=YES：启用匿名访问
      anon_umask=022：匿名用户所上传文件的权限掩码
      anon_root=/var/ftp：匿名用户的FTP根目录
      anon_upload_enable=YES：允许上传文件
      anon_mkdir_write_enable=YES：允许创建目录
      anon_other_write_enable=YES：开放其他写入权
      anon_max_rate=0：限制最大传输速率，单位为字节

常用的本地用户FTP配置项

      local_enable=YES：是否启用本地系统用户
      local_umask=022：本地用户所上传文件的权限掩码
      local_root=/var/ftp：设置本地用户的FTP根目录
      chroot_local_user=YES：是否将用户禁锢在主目录
      local_max_rate=0：限制最大传输速率（字节/秒）
 
三．构建可匿名上传的vsftpd服务器

调整上传目录的属主或权限

确保匿名用户ftp有权写入文件

{% highlight bash %}
{% raw %}

   chown ftp /var/ftp/pub
  
{% endraw %}
{% endhighlight %}

修改vsftpd.conf主配置文件

{% highlight bash %}
{% raw %}

[root@filesvr ~]#
vi /etc/vsftpd/vsftpd.conf
anonymous_enable=YES
local_enable=NO
write_enable=YES
anon_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
userlist_enable=NO
……
   
{% endraw %}
{% endhighlight %}

四．构建本地用户验证的vsftpd服务器
 
修改vsftpd.conf配置文件

      启用本地用户访问
      并可以结合user_list文件灵活控制用户访问

修改主配文件

{% highlight bash %}
{% raw %}

[root@filesvr ~]#
vi /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
local_umask=022
userlist_enable=YES
userlist_deny=YES
……
  
{% endraw %}
{% endhighlight %}

五．构建基于虚拟用户的vsftpd服务器

    1.建立虚拟FTP用户的帐号数据库文件
    2.创建FTP根目录及虚拟用户映射的系统用户
    3.建立支持虚拟用户的PAM认证文件
    4.在vsftpd.conf文件中添加支持配置
    5.为个别虚拟用户建立独立的配置文件
    6.重新加载vsftpd配置
    7.使用虚拟FTP账户访问测试