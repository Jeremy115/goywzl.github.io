---
layout: article
title:  "DNS server"
date:   2015-12-30 09:20:00 +0800
categories: server
---


一．DNS系统概述

1.DNS系统的作用

       a.正向解析：根据主机名称（域名）查找对应的IP地址
       b.反向解析：根据IP地址查找对应的主机域名

2.DNS系统的分布式数据结构

![dhcp](/images/server/dns.jpg)

3.DNS查询方式:

       a.递归查询
              大多数客户机向DNS服务器解析域名的方式
       b.迭代查询
              大多数DNS服务器向其他DNS服务器解析域名的方式
 
二．     DNS服务器的类型（角色）

1.缓存域名服务器

    a.也称为 唯高速缓存服务器
    b.通过向其他域名服务器查询获得域名->IP地址记录
    c.将域名查询结果缓存到本地，提高重复查询时的速度

2.主域名服务器

    a.特定DNS区域的官方服务器，具有唯一性
    b.负责维护该区域内所有域名->IP地址的映射记录

3.从域名服务器

    a.也称为 辅助域名服务器
    b.其维护的 域名->IP地址记录 来源于主域名服务器

三．     BIND域名服务基础
 
1.BIND（Berkeley Internet Name Daemon）

    伯克利Internet域名服务
    官方站点：https://www.isc.org/

2.相关软件包

    bind-9.3.3-7.el5.i386.rpm        主程序包
    bind-utils-9.3.3-7.el5.i386.rpm    工具
    bind-chroot-9.3.3-7.el5.i386.rpm  安全牢笼(伪装根目录)
    caching-nameserver-9.3.3-7.el5.i386.rpm  模板

3.BIND服务器端程序

    主要执行程序：/usr/sbin/named
    服务脚本：/etc/init.d/named
    默认监听端口：53
    主配置文件：/var/named/chroot/etc/named.conf
    保存DNS解析记录的数据文件位于：/var/named/chroot/var/named/

注意: named默认监听TCP、UDP协议的53端口，以及TCP的953端口：

    其中UDP 53端口一般对所有客户机开放，以提供解析服务；
    TCP 53端口一般只对特定从域名服务器开放，提高解析记录传输通道；

	TCP 953端口默认只对本机（127.0.0.1）开放，用于为rndc远程管理工具提供控制通道

如果没有安装bind-chroot软件包，则主配置文件默认位于 /etc/named.conf，数据文件默认保存在 /var/named/ 目录

四．主配置文件named.conf

全局配置部分

{% highlight bash %}
{% raw %}

options {
    listen-on port 53 { 173.16.16.1; };监听端口号
    directory   "/var/named";
    allow-query  { 192.168.1.0/24; 173.16.16.0/24; };允许查询的客户机地址
    recursion yes; 是否允许为客户机进行递归查询
};

{% endraw %}
{% endhighlight %}

根区域配置部分

{% highlight bash %}
{% raw %}

zone "." IN { 定义根区域
    type hint; hint表示根区域;master表示主区域;slave辅助区域       
    file "named.ca"; file 用于设置 该区域对应的数据文件名
};

{% endraw %}
{% endhighlight %}

自己定义的区域配置部分

{% highlight bash %}
{% raw %}

zone "benet.com" IN {  benet域名的正向区域
    type master;       主dns
    file "benet.com.zone"; 正向区域的配置文件名字
    allow-transfer  { 173.16.16.2; };允许下载该区域解析记录的从域名服务的地址
    allow-update    { none; }; 允许动态更新哪些客户机地址，none 表示全部禁止
};

zone "16.16.173.in-addr.arpa" IN {benet域名的反向区域
    type master;               主dns
    file "173.16.16.arpa";       反向区域配置文件
};

{% endraw %}
{% endhighlight %}


五．区域数据配置文件

1）正向区域文件内容

{% highlight bash %}
{% raw %}

$TTL    86400              ; 有效地址解析记录的默认缓存时间
@  IN  SOA  benet.com.  admin.benet.com.  (
        2009021901      ;更新序列号
        3H          ;刷新时间
        15M             ;重试延时
        1W               ;失效时间
        1D           ;无效地址解析记录的默认缓存时间
)
@    IN    NS    ns1.benet.com.指定谁是域名服务器
  IN       MX  10  mail.benet.com.指定邮件解析记录
ns1        IN    A     173.16.16.1
mail       IN    A     173.16.16.1
www      IN    A     173.16.16.1
ftp         IN    CNAME     www别名记录

{% endraw %}
{% endhighlight %}

2）反向区域文件内容

把A记录换为PTR记录，别的内容保持不变

{% highlight bash %}
{% raw %}

1     IN       PTR         www.benet.com.
4     IN       PTR         study.benet.com.  4表示最后一位ip
 
{% endraw %}
{% endhighlight %}

六．区域数据文件的几个特殊应用
 
1.基于域名解析的负载均衡

同一域名对应到多个IP地址

例如有一个网站是movie.benet.com

{% highlight bash %}
{% raw %}

movie       IN       A        173.16.16.11
movie       IN       A        173.16.16.12
movie       IN       A        173.16.16.13
 
{% endraw %}
{% endhighlight %}

2.泛域名解析

    找不到精确对应的A记录时，使用“*”进行匹配

{% highlight bash %}
{% raw %}

*           IN       A        173.16.16.173

{% endraw %}
{% endhighlight %}

3.子域授权

{% highlight bash %}
{% raw %}

cn		      IN		A		173.16.16.2
IN		      NS		ns.jv.net.cn.
ns.jv.net.cn.	IN		A		173.16.16.2
 
{% endraw %}
{% endhighlight %}

七．     对配置文件进行语法检查

1.named-checkconf工具

{% highlight bash %}
{% raw %}

named-checkconf   named.conf

{% endraw %}
{% endhighlight %}

2.named-checkzone工具    后面加域名  然后是正/反向文件

例如:named-checkzone benet.com benet.com.zone

八. 在dns的主配文件当中,可以用”#”,”//”,或者”/*……*/”

    但区域文件当中,只能以”;”来表示注释.   切记!!!!

补充：下面的的命令可以更新根区域配置文件

{% highlight bash %}
{% raw %}

dig @a.root-servers.net  ns > /var/named/chroot/var/named/named.ca

{% endraw %}
{% endhighlight %}
