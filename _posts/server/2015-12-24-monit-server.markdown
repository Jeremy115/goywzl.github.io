---
layout: article
title:  "Monit监控"
date:   2015-12-24 09:20:00 +0800
categories: server
---

在一篇文章里看到了Monit这个词语，说是能自动监控进程或服务，停止了直接自动启动，好东西啊，所以就上网查了查，了解下NB之处。

# Monit介绍： #

Monit是一款功能非常丰富的进程、文件、目录和设备以及文件系统等（processes,programs, files, directories and filesystems）监控软件。

比如时间戳，校验和，或者大小发生改变都能监测到！用于Unix平台。它可以自动修复那些已经停止运作的程序，特使适合处理那些由于多种原因导致的软件错误。monit运行于应用层。

环境：

    系统：Centos6.3
    软件：monit-5.12.tar.gz
    时间：2015-4-1


----------

# Monit安装 #

`http://mmonit.com/monit/dist/ ` 这个地址下都是monit的源码包，不过似乎被墙了，我是翻过去的。

我下载的是最新(就今天而言)的了： `wget  http://mmonit.com/monit/dist/monit-5.12.tar.gz`

下载完成就是安装了具体步骤如下，不做解释了！

{% highlight bash %}
{% raw %}

tar -zxvf monit-5.12.tar.gz 
cd monit-5.12

{% endraw %}
{% endhighlight %}

使用./configure的时候出现以下错误

{% highlight bash %}
{% raw %}

Couldn't find your SSL header files.
Use --with-ssl-incl-dir option to fix this problem or disable
the SSL support with --without-ssl

{% endraw %}
{% endhighlight %}

只好用下面的这个方法安装

{% highlight bash %}
{% raw %}

./configure --without-ssl
make
make install

{% endraw %}
{% endhighlight %}

到此就安装完成了，如果你遇到报错，看看是不是缺包，我遇到了一个缺少pam 的使用yum install -y pam pam-devel 之后就可以了，如果看不懂，就把报错信息谷歌下！

# Monit配置 #

配置文件不是自动生成到/etc下的，你需要自己手动把monit-5.12目录下的monitrc文件cp到/etc下

{% highlight bash %}
{% raw %}

cp monitrc /etc/

{% endraw %}
{% endhighlight %}

之后就是这个配置文件的内容了

{% highlight bash %}
{% raw %}

set daemon  60                      # 设置monit作为守护进程运行，并且每1分钟监视一次|我这里是默认的60秒
set logfile /var/log/monit.log    # 设置日志文件的位置，如果要写入系统日志可以
set httpd port 2812 and          # 设置的一个web页面访问的，2812是访问端口
use address localhost         # 设置这个服务器的地址
allow localhost                  # 允许本地访问
allow admin:monit            # 设置用户名密码（我都是用的默认的，只是给大家做一下解释）
set mailserver 192.168.1.100      #设置邮件服务器
set alert zhengsenlin@gmail.com     # 收邮件地址,如果要发送到多个地址

{% endraw %}
{% endhighlight %}

基本的配置介绍完了，现在就介绍监控项，如：system 、file、process、filesystem、network等等，简单介绍几个，功能太强大，不一一介绍了

## 首先：平均负载、内存使用、cpu使用 ##

{% highlight bash %}
{% raw %}

check system 192.168.1.100
   if loadavg (1min) > 4 then alert
   if loadavg (5min) > 2 then alert
   if memory usage > 75% then alert
   if cpu usage (user) > 70% then alert
   if cpu usage (system) > 30% then alert
   if cpu usage (wait) > 20% then alert

{% endraw %}
{% endhighlight %}

## 其次：Tomcat服务 ##

{% highlight bash %}
{% raw %}

check process tomcat with pidfile /var/run/catalina.pid     #这个是tomcat的进程pid文件（tomcat需要自行设置下）
start program = "/etc/init.d/tomcat start"              # 设置启动命令
stop program  = "/etc/init.d/tomcat stop"               # 设置停止命令
if 9 restarts within 10 cycles then timeout             # 设置在10个监视周期内重启了9次则超时,不再监视这个服务。
if cpu usage > 90% for 5 cycles then alert          # 如果在5个周期内该服务的cpu使用率都超过90%则提示
if failed url http://127.0.0.1:4000/ timeout 120 seconds for 5 cycles then restart        #若连续5个周期打开url都失败（120秒超时，超时也认为失败）则重启服务

{% endraw %}
{% endhighlight %}

最后，介绍一个监控ssh服务，毕竟这个都用

## 监视ssh服务： ##

{% highlight bash %}
{% raw %}

check process sshd with pidfile /var/run/sshd.pid

   start program  "/etc/init.d/sshd start"

   stop program  "/etc/init.d/sshd stop"

   if failed port 22 protocol SSH then restart

   if 5 restarts within 5 cycles then timeout

{% endraw %}
{% endhighlight %}

`include /etc/monit.d/* `    如果感觉一个文件监控这个多个项目麻烦，可以在这个目录（这个目录自己创建）以一个监控项目创建一个文件，把这个#号删除，便于管理，我是这么想的

### *Monit友情提示* ###

由于将monit设置成了守护进程,并且在inittab中加入了随系统启动的设置,则monit进程如果停止,init进程会将其重启,而monit又监视着其它的服务,这意味着monit所监视的服务不能使用一般的方法来停止,因为一停止,monit又会将其启动.要停止monit所监视的服务,应该使用monit stop name这样的命令。

### *例如：* ###

    要停止tomcat:monit stop tomcat
    要停止全部monit所监视的服务可以使用  monit stop all
    要启动某个服务可以用monit stop name这样的命令,启动全部则是monit start all