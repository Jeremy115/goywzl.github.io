---
layout: article
title:  "DHCP server"
date:   2015-12-27 09:20:00 +0800
categories: server
---


dhcp=dynamic host configure  protocol动态主机配置协议

一．Dhcp服务的作用以及分配的主要信息：

       DHCP服务
       为大量客户机自动分配地址，提供集中管理
       减轻管理和维护成本、提高网络配置效率
       可分配的地址信息主要包括
       网卡的IP地址、子网掩码
       对应的网络地址、广播地址
       缺省网关地址
       DNS服务器地址

二．安装DHCP服务器

       DHCP服务器软件
       RHEL5光盘中的 dhcp-3.0.5-3.el5.i386.rpm
       dhcp软件包的主要文件
       主配置文件：/etc/dhcpd.conf
       执行程序：/usr/sbin/dhcpd、/usr/sbin/dhcrelay
       服务脚本：/etc/init.d/dhcpd、/etc/init.d/dhcrelay
       执行参数配置：/etc/sysconfig/dhcpd
       DHCP中继配置：/etc/sysconfig/dhcrelay

三．主配置文件 /etc/dhcp.conf

编辑/etc/dhcpd.conf,修改如下内容：

全局设置 

{% highlight bash %}
{% raw %}

ddns-update-style    interim;(动态dns更新方式)  
default-lease-time 21600;(默认租约时间)
max-lease-time 43200; (最大租约时间)
option domain-name              “abc.com”;（服务器名称）
option domain-name-servers 192.168.1.1，202.106.0.20；（dns）注意格式

{% endraw %}
{% endhighlight %}

具体子网设置

{% highlight bash %}
{% raw %}

Subnet   192.168.1.0   netmask 255.255.255.0 {设置子网}
        range       192.168.1.100 192.168.1.200;（设置可分配地址范围）
        option subnet-mask              255.255.255.0;（子网掩码）
        option routers                  192.168.1.1;（网关）
   host Server01 {       指定特定主机，分配特定的ip地址（经理的ip）
        hardware ethernet b0:c0:c3:22:46:81;客户端的mac地址
        fixed-address 192.168.1.11;分配给客户端的ip
   }
}

{% endraw %}
{% endhighlight %}

四．查看地址租用记录：/var/lib/dhcpd/dhcpd.leases

{% highlight bash %}
{% raw %}

tail -7 /var/lib/dhcpd/dhcpd.leases

lease 192.168.1.253 {
    starts 0 2015/04/04 19:00:44;
    ends 1 2015/04/05 01:00:44;
    binding state active;
    next binding state free;
    hardware ethernet 00:0a:22:f1:1a:2e;
}

{% endraw %}
{% endhighlight %}

五．使用DHCP客户端

三种方法配置

1）使用setup命令配置为dhcp

2）修改网卡配置文件（如 ifcfg-eth0）

{% highlight bash %}
{% raw %}

BOOTPROTO=dhcp

{% endraw %}
{% endhighlight %}

3）使用dhclient命令

格式：dhclient  [-d]  [网络接口名]   （很少用）

六．DHCP中继服务，如图所示的网络当中，如何为不同网段分配ip？

七．配置DHCP中继服务器

1.开启服务器的路由转发功能

编辑  /etc/sysctl.conf,

     把net.ipv4.ip_forward = 0改为1
     然后执行：sysctl  -p (让刚才修改的内容立即生效)

2.设置中继接口及DHCP服务器的地址

{% highlight bash %}
{% raw %}

vi /etc/sysconfig/dhcrelay
INTERFACES=”eth0 eth1″（指定侦听服务的网卡名称）
DHCPSERVERS=”192.168.1.2″（指定谁是dhcp服务器）

{% endraw %}
{% endhighlight %}

3.启动dhcrelay中继服务程序

{% highlight bash %}
{% raw %}

Server  dhcrelay   start

{% endraw %}
{% endhighlight %}
