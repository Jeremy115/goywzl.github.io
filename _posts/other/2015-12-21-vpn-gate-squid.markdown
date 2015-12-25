---
layout: article
title:  "配置VPN-gate与Squid使内网访问某某网站"
date:   2015-12-21 09:20:00 +0800
categories: other
---


时间：2014-11-11

环境：windows 7-64

IP:192.168.1.66

软件：VPN gate、Squid For windows

这个实验的目地，是为了让内网的朋友可以访问被墙的网站
 
1.安装VPN-gate，这么简单的安装方法日后我要是不会了，也就不用混了，所以不做解释，这样这台win7就能翻墙了。

2.解压Squid文件，放到C盘根目录下，名字不要变，就是squid（其实无所谓）

3.拷贝C:\squid\etc\mime.conf.default 到 C:\squid\etc\mime.conf

4.既然已经有配置文件了，现在咱们就改配置文件

1）首先，定义squid允许的访问自己的IP范围，在”acl Safe_ports port 777”下面添加如下两行，看准了IP书写语法，在做添加

{% highlight bash %}
{% raw %}

acl Safe_ports port 777		# multiling http
acl name1 src 192.168.1.0/255.255.255.0	#squid2
acl CONNECT method CONNECT

{% endraw %}
{% endhighlight %}

2）之后，需要把定义的这些IP，进行允许的操作，在”http_access deny CONNECT”下添加如下三行，这里最后定义了拒绝所有

{% highlight bash %}
{% raw %}

http_access deny CONNECT !SSL_ports
http_access allow name1
http_access deny all

{% endraw %}
{% endhighlight %}

3）然后，我们来改Squid的缓存，这个关系到本机的性能，你懂得，之前是“cache_mem 8 MB ”我这里改成了3072MB，这个看你电脑的配置了，不做解释

{% highlight bash %}
{% raw %}

cache_mem 3072 MB

{% endraw %}
{% endhighlight %}

4）最后，要设置hostname 的，不然squid是无法启动的，这个要搜索到这个名字，然后在下面添加”visible_hostname testsquid”其中这个名字，随便你写啦

{% highlight bash %}
{% raw %}

visible_hostname testsquid

{% endraw %}
{% endhighlight %}

5）附加，这个可有可无，是设置链接端口的，你可以不改

{% highlight bash %}
{% raw %}

http_port 3128

{% endraw %}
{% endhighlight %}

5.到了这里，squid的配置文件算是改好了，接下来就需要启动这个服务

打开CMD的，然后，进入到squid的sbin目录“cd c:\squid\sbin”之后执行，”squid –I”这个命令，进行系统服务的添加

这样我的电脑——>管理——>服务中就有“squid”了，然后执行”squid -z”创建缓存目录，之后执行“net start squid”启动squid服务
我都操作完了，下面是命令介绍

Microsoft Windows [版本 6.1.7601]

版权所有 (c) 2009 Microsoft Corporation。保留所有权利。

C:\Users\Administrator>cd c:\squid\sbin

c:\squid\sbin>squid -i

CreateService failed

c:\squid\sbin>squid -z

2015/03/18 19:34:22| WARNING cache_mem is larger than total disk cache space!

2015/03/18 19:34:22| Creating Swap Directories

c:\squid\sbin>net start squid

请求的服务已经启动。

请键入 `NET HELPMSG 2182` 以获得更多的帮助。

c:\squid\sbin>


6.到了这里，squid算是配置完成了，现在找一台squid允许的网段的机器，打开IE，进行代理设置：internet 选项——>链接——>局域网设置——>勾选代理服务器，配置结果如下图:

![framework](/images/other/vpn-gate+squid.jpg)

7.至此，别的人就能通过这台Squid代理翻墙了。
