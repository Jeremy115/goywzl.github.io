---
layout: article
title:  "ubuntu-network-setting"
date:   2015-12-16 09:20:00 +0800
categories: linux
---

经常用centos，突然用ubunut，网卡都不会配置了，找到了资料，分享下


Ubuntu的网络配置文件是：/etc/network/interfaces

1、以DHCP 方式配置网卡

{% highlight bash %}
{% raw %}

auto eth0
iface eth0 inet dhcp

{% endraw %}
{% endhighlight %}

用`sudo /etc/init.d/networking restart`命令使网络设置生效

2、为网卡配置静态IP地址

{% highlight bash %}
{% raw %}

sudo vi /etc/network/interfaces
auto eth0
iface eth0 inet static
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.1
sudo /etc/init.d/networking restart

{% endraw %}
{% endhighlight %}

3、设定第二个IP地址(虚拟IP地址）

{% highlight bash %}
{% raw %}

sudo vi /etc/network/interfaces
auto eth0:1
iface eth0:1 inet static
address 192.168.1.101
netmask 255.255.0
gateway 192.168.1.1
sudo /etc/init.d/networking restart

{% endraw %}
{% endhighlight %}

4、设置主机名称(hostname)

使用下面的命令来查看当前主机的主机名称：

{% highlight bash %}
{% raw %}

sudo /bin/hostname

{% endraw %}
{% endhighlight %}

使用下面的命令来设置当前的主机名称：

{% highlight bash %}
{% raw %}

sudo /bin/hostname newname

{% endraw %}
{% endhighlight %}

5、配置DNS

(1) /etc/hosts中加入一些主机名称和这些主机名称对应的IP地址，这是本机的静态查询

(2) /etc/resolv.conf
 
{% highlight bash %}
{% raw %}

nameserver *.*.*.*

{% endraw %}
{% endhighlight %}
