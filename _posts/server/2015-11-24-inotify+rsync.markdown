---
layout: article
title:  "文件实时同步inotify+rsync"
date:   2015-11-24 09:20:00 +0800
categories: home server
---

搭建了一个公司内部的共享服务器，让公司员工存放自己的文件，其实这个做一个每日备份即可，或者每周，不过上次领导放的文件，莫名其妙的没有了，让我这一顿好找，为了避免这样的事情发生，我只能做一个实时备份了， 研究了一下inotify+rsync。

理论：Inotify是一种文件变化通知机制（也有人叫它文件系统事件监控机制），Linux内核从2.6.13开始引入。在BSD和Mac OS系统中比较有名的是kqueue，它可以高效地实时跟踪Linux文件系统的变化。
Inotify+rsync组合，采用系统级别监控各种变化，当Inotify监控到文件发生任何变化，就会触发rsync同步，解决效率实时性问题。

一、主服务器(rsync+inotify)192.168.1.8

1、准备软件包

{% highlight bash %}
{% raw %}

[root@localhost ~]# mkdir /data/ftpdata
    
[root@localhost ~]# wget http://rsync.samba.org/ftp/rsync/rsync-3.1.1.tar.gz
    
[root@localhost ~]# wget http://github.com/downloads/rvoicilas/inotify-tools/inotify-tools-3.14.tar.gz


{% endraw %}
{% endhighlight %}

2、安装Rsync并配置

{% highlight bash %}
{% raw %}

[root@localhost ~]# tar -zxvf rsync-3.1.1.tar.gz
    
[root@localhost ~]# ./configure --prefix=/usr/local/rsync
    
[root@localhost ~]# make && make install

{% endraw %}
{% endhighlight %}
 
建立密码认证文件

{% highlight bash %}
{% raw %}
    
[root@localhost ~]# echo "pasword">/etc/rsyncd.secrets
    
[root@localhost ~]# cat /etc/rsyncd.secrets 
    
pasword

{% endraw %}
{% endhighlight %}

其中password可以自己设置密码，rsyncd.secrets名字也可以自己设置；
权限：要将/etc/rsyncd.secrets设置为root拥有, 且权限为600。

{% highlight bash %}
{% raw %}

[root@localhost ~]# chmod 600 /etc/rsyncd.secrets
    
[root@localhost ~]# ll /etc/rsyncd.secrets
    
-rw------- 1 root root 7 Jun9 21:24 /etc/rsyncd.secrets

{% endraw %}
{% endhighlight %}
 
3、安装inotify

{% highlight bash %}
{% raw %}

[root@localhost ~]# tar -zxvf inotify-tools-3.14.tar.gz 
    
[root@localhost ~]# cd inotify-tools-3.14
    
[root@localhost inotify-tools-3.14]# ./configure --prefix=/usr/local/inotify
    
[root@localhost inotify-tools-3.14]# make && make install

{% endraw %}
{% endhighlight %}

4、创建rsync同步脚本

此项功能主要是将主服务器端的目录/data/里的内容，如果修改了（无论是添加、修改、删除文件）能够通过inotify监控到，并通过rsync实时的同步给backup的/data/里，下面是通过shell脚本实现的。

{% highlight bash %}
{% raw %}
    
[root@localhost ~]# vim /opt/rsync.sh
    
#!/bin/bash
SRC=/data/
    
DST=root@192.168.1.7::data
    
/usr/local/inotify/bin/inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w %f' -e modify,delete,create,attrib $SRC | while read file DATE TIME DIR;
do
    
/usr/local/rsync/bin/rsync -vzrtopg --delete --password-file=/etc/rsyncd.secrets $SRC $DST> /dev/null

done

[root@localhost ~]# chmod u+x /opt/rsync.sh
 
rsync.sh脚本加入开机启动项

[root@localhost ~]# echo "/opt/rsync.sh" >> /etc/rc.local
 
防火墙开启rsync端口：873

[root@localhost ~]# vim /etc/sysconfig/iptables

添加：

-A INPUT -m state --state NEW -m tcp -p tcp --dport 873 -jACCEPT

重启：

[root@localhost ~]# /etc/init.d/iptables restart

{% endraw %}
{% endhighlight %}

二、备份服务器(rsync)192.168.1.7


1、准备工作

创建备份目录：

{% highlight bash %}
{% raw %}
    
[root@backup ~]# mkdir /data/

{% endraw %}
{% endhighlight %}
 
2、准备软件包

{% highlight bash %}
{% raw %}

[root@backup ~]# wget http://rsync.samba.org/ftp/rsync/rsync-3.1.1.tar.gz

{% endraw %}
{% endhighlight %}
 
3、安装rsync（备份服务器只安装rsync）

{% highlight bash %}
{% raw %}

[root@localhost ~]# tar -zxvf rsync-3.1.1.tar.gz
    
[root@localhost ~]# ./configure --prefix=/usr/local/rsync
    
[root@localhost ~]# make && make install

{% endraw %}
{% endhighlight %}
 
4、建立用户与密码认证文件

{% highlight bash %}
{% raw %}
    
[root@backup ~]# echo "root:password" > /etc/rsyncd.secrets
    
[root@backup ~]# cat /etc/rsyncd.secrets
    
root:password 

{% endraw %}
{% endhighlight %}

注意：

请记住，在主服务器端建立的密码文件，只有密码，没有用户名；而在备份服务端backup里建立的密码文件，用户名与密码都有。

权限：要将`/etc/rsyncd.secrets`设置为root拥有, 且权限为600。

{% highlight bash %}
{% raw %}

[root@backup ~]#chmod 600 /etc/rsyncd.secrets

{% endraw %}
{% endhighlight %}
 
5、建立rsync配置文件

{% highlight bash %}
{% raw %}

[root@backup ~]# vim /etc/rsyncd.conf

uid = root

gid = root

port = 873

use chroot = yes

read only = yes

hosts allow=192.168.1.0/255.255.255.0

hosts deny=*

max connections = 5

log file =/var/log/rsyncd.log

pid file =/var/run/rsyncd.pid

lock file =/var/run/rsyncd.lock

log format = %t %a %m %f%b

syslog facility = local3

timeout = 300
 
[data]

path = /data/

list = no

read only = no

ignore errors

auth users = root

secrets file =/etc/rsyncd.secrets

{% endraw %}
{% endhighlight %}    
 
启动rsync服务


{% highlight bash %}
{% raw %}

[root@backup ~]# /usr/local/rsync/bin/rsync --daemon 	--config=/etc/rsyncd.conf
    
[root@backup ~]# ps -ef |grep rsync 

{% endraw %}
{% endhighlight %}

Rsync服务加入开机启动项

{% highlight bash %}
{% raw %}

[root@backup ~]# echo "/usr/local/rsync/bin/rsync --daemon --config=/etc/rsyncd.conf" >> /etc/rc.local

{% endraw %}
{% endhighlight %}
 
防火墙开启rsync端口：873

{% highlight bash %}
{% raw %}

[root@backup ~]# vim /etc/sysconfig/iptables

{% endraw %}
{% endhighlight %}

添加：

{% highlight bash %}
{% raw %}

-A INPUT -m state --state NEW -m tcp -p tcp --dport 873 -jACCEPT

{% endraw %}
{% endhighlight %}
 
重启：

{% highlight bash %}
{% raw %}

[root@backup ~]# /etc/init.d/iptables restart

{% endraw %}
{% endhighlight %}
 
现在rsync与inotify在主服务器端安装完成，rsync在备份服务器backup端也安装完成！
 
重启

{% highlight bash %}
{% raw %}

[root@localhost ~]# reboot
    
[root@backup ~]# reboot

{% endraw %}
{% endhighlight %}
 
三、测试验证

1、在主服务器端/data/ 目录上创建一个文件夹：


{% highlight bash %}
{% raw %}

[root@localhost data]# mkdir zl

{% endraw %}
{% endhighlight %}
 
2、在backup端查看/data/目录是否相同；

{% highlight bash %}
{% raw %}

[root@backup ~]# ll /data/

{% endraw %}
{% endhighlight %}

四、备份与恢复

1、手动备份

{% highlight bash %}
{% raw %}

192.168.1.7---------->192.168.1.8 
    
[root@ localhost ~]# /usr/bin/rsync -vzrtopg --delete--password-file=/etc/
    
rsyncd.secrets /data/ root@192.168.1.8::data

{% endraw %}
{% endhighlight %}
 
2、手动恢复

{% highlight bash %}
{% raw %}

192.168.1.8 ---------->192.168.1.7
    
[root@localhost ~]# /usr/bin/rsync -vzrtopg --delete--password-file=/etc/rsyncd.secrets 
    
root@192.168.1.7::data /data/

{% endraw %}
{% endhighlight %} 
