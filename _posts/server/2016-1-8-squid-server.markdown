---
layout: article
title:  "Squid Server"
date:   2016-01-08 09:20:00 +0800
categories: server
---

# 一．缓存代理概述 #

1.代理服务器的作用

     通过缓存的方式为用户提供Web访问加速
     对用户的Web访问进行过滤控制

2.普通代理服务

     即标准的、传统的代理服务
     需要客户机在浏览器中指定代理服务器的地址、端口

3.透明代理服务

     适用于企业的网关主机（共享接入Internet）中
     客户机不需要指定代理服务器地址、端口等信息
     需要设置防火墙策略将客户机的Web访问数据转交给代理服务程序处理

4.反向代理服务

     为Internet用户访问企业Web站点提供缓存加速
 
# 二．Squid基本配置 #

1.squid软件包

    a)软件包名：squid-2.6.STABLE6-3.el5
    b)服务名：squid
    c)主程序：/usr/sbin/squid
    d)配置目录：/etc/squid/
    e)主配置文件：/etc/squid/squid.conf 
    f)默认监听端口：TCP 3128
    g)默认访问日志文件：/var/log/squid/access.log

2.常用配置项

    a) http_port 3128   监听端口
    b) cache_mem 64 MB 缓冲内存大小
    c) maximum_object_size 4096 KB 高速缓存的最大对象文件
    d) reply_body_max_size 10240000 allow all  允许用户下载的最大文件大小，默认为B（字节）
    e) access_log /var/log/squid/access.log squid  访问日志位置
    f) visible_hostname proxy.benet.com 代理服务器名称
    g) dns_testnames www.google.com www.163.com  dns测试服务器
    h) cache_dir ufs /var/spool/squid 100 16 256 缓存目录格式/位置/大小/一级缓存目录数量/二级缓存目录大小
 
# 三．ACL访问控制 #

1.ACL（Access Control List，访问控制列表）

     可以从客户机的IP地址、请求访问的URL/域名/文件类型、访问时间、并发请求数等各方面进行控制

2.应用访问控制的方式

     定义acl列表
        acl 列表名称 列表类型 列表内容 … 
     针对acl列表进行限制
        http_access allow或deny 列表名…… 

3.访问控制规则的匹配顺序

	没有设置任何规则时
		—— 将拒绝所有客户端的访问请求
	有规则但找不到相匹配的项时
		—— 将采用与最后一条规则相反的权限，即如果最后一条规则是allow，那么就拒绝客户端的请求，否则允许该请求
 
# 四．配置透明代理 #

1.实现透明代理的基本条件

前提：
	客户机的Web访问数据要能经过防火墙
	代理服务构建在网关（防火墙）主机中
配置要求：
	代理服务程序能够支持透明代理
	设置防火墙规则，将客户机的Web访问数据自动重定向给代理服务程序处理
 
# 五．配置反向代理 #

1.基本实现步骤

a)修改squid.conf文件，并重新加载该配置

{% highlight bash %}
{% raw %}

http_port  218.29.30.31:80 vhost 
cache_peer 192.168.2.11 parent 80 0 originserver weight=5 max-conn=30
cache_peer 192.168.2.12 parent 80 0 originserver weight=5 max-conn=30

{% endraw %}
{% endhighlight %}

*注意：*

cache_peer   Web服务器地址   服务器类型    http端口   icp端口   [可选项]

cache_peer配置项可以用于指定真正的Web服务器的位置。其中，服务器类型对应到目标主机的缓存级别，上游Web主机一般使用“parent”（父服务器）；icp端口用于连接相邻的ICP（Internet Cache Protocol）缓存服务器（通常为另一台Squid主机），如果没有，则使用0；可选项是提供缓存时的一些附件参数，例如“originserver”表示该服务器作为提供Web服务的原始主机，“weight=n”指定服务器的优先权重，n为整数，数字越大优先级越高（缺省为1）；“max-conn=n”指定反向代理主机到该web服务器的最大连接数