---
layout: article
title:  "Linux系统Iptables防火墙"
date:   2016-01-02 09:20:00 +0800
categories: linux
---

# 一、防火墙简介 #

1、功能：

    1）通过源端口，源IP地址，源MAC地址，包中特定标记和目标端口，IP，MAC来确定数据包是否可以通过防火墙
    2）分割内网和外网【附带的路由器的功能】
    3）划分要被保护的服务器

如果Linux服务器启用了防火墙，SELinux等的防护措施，那么，他的安全级别可以达到B2[原来是C2]
 
2、防火墙分类

    1）数据包过滤【绝大多数的防火墙】
            分析IP地址，端口和MAC是否符合规则，如果符合，接受
    2）代理服务器
 
3、防火墙的限制

    1）防火墙不能有效防止病毒，所以防火墙对病毒攻击基本无效，但是对木马还是有一定的限制作用的。
    2）防火墙一般不设定对内部[服务器本机]访问规则，所以对内部攻击无效

【附】现当今的杀毒软件对病毒的识别率大约在30%左右。也就是说，大部分的病毒是杀毒软件并不认识的！
 
4、防火墙配置原则【交叉使用】

    拒绝所有，逐个允许
    允许所有，逐个拒绝

【附：】防火墙规则：谁先配置，谁先申请！

# 二、防火墙规则 #

五个hook function（规则链）：Input ，Output，Forward，prerouting，postrouting。

四种处理机制（表）：过滤（filter） 主要用于过滤数据包，该表根据管理员预定义的一组规则过滤符合条件的数据包。filter表是iptables默认的表。

	INPUT：主要与想要进入我们 Linux 本机的数据包有关；
	OUTPUT：主要与我们 Linux 本机所要送出的数据包有关；
	FORWARD：与 Linux 本机比较没有关系， 他可以传递数据包到后台的计算机中，与下列 nat table 相关性较高。地址转换（NAT） 主要用于网络地址转换，该表可实现一对一。一对多，多对多等工作，iptables就是使用该表实现共享上网功能。
	PREROUTING：在进行路由判断之前所要进行的规则(DNAT/REDIRECT)
	POSTROUTING：在进行路由判断之后所要进行的规则(SNAT/MASQUERADE)
	OUTPUT：与发送出去的数据包包有关 包重构（mangle）对指定的数据包进行修改，例如更改TTL和TOS等，实际中很少使用。RAW  很少使用一条链可以包含一条或者数条规则。
	nat：prerouting，postrouting，output
	mangle：Input ，Output，Forward，prerouting，postrouting。
	RAW：prerouting，output

# 三、防火墙原理 #

1.当一个数据包进入网卡时，数据包首先进入PREROUTING链，内核根据数据包目的IP判断是否需要转送出去。

2.如果数据包就是进入本机的，数据包就会沿着图向下移动，到达INPUT链。数据包到达INPUT链后，任何进程都会收到它。本机上运行的程序可以发送数据包，这些数据包经过OUTPUT链，然后到达POSTROTING链输出。

3.如果数据包是要转发出去的，且内核允许转发，数据包就会向右移动，经过FORWARD链，然后到达POSTROUTING链输出。

# 四、防火墙书写规则： #

{% highlight bash %}
{% raw %}

iptables [-t table] command CHAIN [NUM ]   match criteria -j TARGET

{% endraw %}
{% endhighlight %}

CHAIN:  对链进行的操作

	-N：new     新建一条链
	-X          删除一条用户自定义链（空链）
	-F：flush   清空一条链，默认清空表中所有链
	-Z：zero    清空计数器，iptables中每条规则默认有两个计数器，用于记录本条规则所匹配到的数据包的个数和本条规则所匹配到的数据包的总大小
	-P：policy  定义链的默认处理策略
	-E  重命名链

RULE：对规则进行的操作

	-A:append   追加，在链的最后加一条规则
	-I:insert  插入一条规则  一般使用-I CHAIN NUM 给规则加一个编号。
	-R:replace 替换某条规则，规则被替换并不会改变顺序，必须要指定替换的规则编号：-R CHAIN NUM。
	-D:delete  删除一条规则，可以输入完整规则，或者直接指定标号加以删除：-D CHAIN NUM。

LIST：查看

	-L：list 列出链中的所有规则
辅助性子命令：-n  numeric 以数字的形式来显示地址，默认显示主机名称
	-v  verbose 显示详细信息 ，支持-vv  -vvv格式，v越多，信息越详细。
	-x  显示原有信息，不要做单位换算
	–line-numbers  显示规则的行号

Match Creteria（匹配规则）：

基本匹配

	-s，–src，–source  匹配数据包的源地址
	-d，–dst，–destination 匹配数据包的目标地址
	-i， 指定数据包的流入接口（逻辑接口）
	-o， 指定数据包的流出接口
	-p， 做协议匹配 protocol，（tcp|udp|icmp）

扩展匹配：对某一种功能的扩展

	隐含扩展 ：对某一种协议的扩展
	-p tcp
		–sport 指定源端口
		–dport 指定目的端口
		–tcp-flags（SYN，ACK，FIN，PSH，URG，RST，ALL，NONE）指定TCP的标志位
		需要跟两个标志位列表，如：SYN，ACK，FIN，RST SYN 第一个列表表示要检查的位，第二个    
		列表表示第一个列表中出现的位必须为1，未出现的必须为0
	–syn  只允许新连接
	-p  udp  
		–sport   指定源端口
		–dport   指定目的端口
	-p  icmp
		–icmp-type  echo-request，8（ping出去，请求回应，） echo-reply，0（给予回应）

显式扩展 ：额外附加的更多的匹配规则,功能性地扩展

	-m state   状态检测扩展
		NEW         用户发起一个全新的请求
		ESTABLISHED 对一个全新的请求进行回应
		RELATED   两个完整连接之间的相互关系，一个完整的连接，需要依赖于另一个完整的连接
		INVALID   无法识别的状态
	-m multiport   –sports  22,80,443 指定多个源端口
        –dports  22，80,443 指定多个目标端口
        –ports    非连续端口
	-m connlimit   限定并发连接速率
        ！–connlimit-above 5  高于五个将拒绝
	-m string   字符串匹配
        –algo bm|kmp 指定算法 
        –string pattern
    -m time  基于时间的匹配
        –timestart
        –timestop
        –days
	-j  TARGET    处理动作
     	ACCEPT    接受
     	DROP      悄悄丢弃，请求端没有任何回应
     	REJECT    明确拒绝
     	SNAT      源地址转换
     	DNAT      目标地址转换
     	REDIRECT  端口重定向
     	LOG       将访问记录下来

----------

# Iptables防火墙示例 #


#### 允许本机通过SSH连接192.168.0.0网段的其他主机 ####

{% highlight bash %}
{% raw %}

[root@server27 ~]# iptables -A INPUT -s 192.168.0.0/24 -d 192.168.0.127 -p tcp –dport 22 -j ACCEPT
[root@server27 ~]# iptables -A OUTPUT -s 192.168.0.127 -d 192.168.0.0/24 -p tcp –sport 22 -j ACCEPT
[root@server27 ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination        
ACCEPT     tcp  —  192.168.0.0/24       192.168.0.127       tcp dpt:22
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination        
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination        
ACCEPT     tcp  —  192.168.0.127        192.168.0.0/24      tcp spt:22

{% endraw %}
{% endhighlight %}

#### 一些默认的防火墙示例 ####

{% highlight bash %}
{% raw %}

iptables –F

{% endraw %}
{% endhighlight %}

删除已经存在的规则 

{% highlight bash %}
{% raw %}

iptables -P INPUT DROP 

{% endraw %}
{% endhighlight %}

配置默认的拒绝规则。基本规则是：先拒绝所有的服务，然后根据需要再添加新的规则。 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p tcp –dport 80 -j ACCEPT 

{% endraw %}
{% endhighlight %}

打开WEB服务端口的tcp协议 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p tcp –dport 110 -j ACCEPT
 
{% endraw %}
{% endhighlight %}

打开POP3服务端口的tcp协议 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p tcp –dport 25 -j ACCEPT 

{% endraw %}
{% endhighlight %}

打开SMTP服务端口的tcp协议 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p tcp –dport 21 -j ACCEPT 

{% endraw %}
{% endhighlight %}

打开FTP服务端口的tcp协议 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p tcp -s 202.106.12.130 –dport 22 -j ACCEPT 

{% endraw %}
{% endhighlight %}

允许IP地址为202.106.12.130这台主机连接本地的SSH服务端口 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p tcp –dport 53 -j ACCEPT 

{% endraw %}
{% endhighlight %}

允许DNS服务端口的tcp数据包流入 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p udp –dport 53 -j ACCEPT 

{% endraw %}
{% endhighlight %}

允许DNS服务端口的udp数据包流入 

{% highlight bash %}
{% raw %}

iptables -A INPUT -p icmp -icmp-type echo-request -i eth1 -j DROP 

{% endraw %}
{% endhighlight %}

防止死亡之ping，从接口eth1进入的icmp协议的请求全部丢弃。 

{% highlight bash %}
{% raw %}

iptables -A FORWARD -p tcp –syn -m limit –limit 1/s -j ACCEPT 

{% endraw %}
{% endhighlight %}

防止SYN Flood (拒绝服务攻击)

{% highlight bash %}
{% raw %}

iptables -t nat -A POSTROUTING -o eth1 -s 192.168.0.226 -j MASQUERADE

{% endraw %}
{% endhighlight %}

允许 192.168.0.226通过eth1 IP伪装出外网

{% highlight bash %}
{% raw %}

iptables -t nat -A POSTROUTING -o eth0 -s 192.168.0.4 -p tcp –dport 25 -j MASQUERADE

{% endraw %}
{% endhighlight %}

允许 192.168.0.4通过eth0 伪装访问外网的 25端口

#### 一些特定功能的示例 ####

time；匹配指定时间范围 结束时间大于起始时间 结束日期于于起始日期

    –datestart 起始日期 YYYY[-MM[-DD[Thh[:mm[:ss]]]]]
    –datestop 结束日期 YYYY[-MM[-DD[Thh[:mm[:ss]]]]]
    –timestart hh:mm[:ss] 起始时间
    –timestop hh:mm[:ss] 结束时间
    –weekdays day [Mon, Tue, Wed, Thu, Fri, Sat, Sun ]也可以是1-7 以周为日期格式做规则

示例；限制本机901端口只允许在工作时间内（周一至周五）的08点到18点才可访问；

{% highlight bash %}
{% raw %}

iptables -A INPUT -d 172.16.34.30 -p tcp –dport 901 -m time –weekdays Mon,Tue,Wed,Thu,Fri –timestart 08:00:00 -timestop 18:00:00 -j ACCEPT

{% endraw %}
{% endhighlight %}

由于做了限制客户的进站访问规则，因此客户只有规定的工作日才能访问，因此下面这条规则就可以简写。

（服务器可以随时放行用户的请求，但是客户不是随时可以访问的。）

{% highlight bash %}
{% raw %}

iptables -A OUTPUT -s 172.16.34.30 -p tcp –sport 901 -j ACCEPT

{% endraw %}
{% endhighlight %}

    string；对字符串匹配
    –algo {bm|kmp}; 字符匹配查找时使用的算法
    –string “STRING”; 要查找的字符串

示例；如果客户端口访问的网页当中有规则里定义的字符串则不显示给客户
安装httpd服务，新建两个首页文件，写上不一样的内容，其中一个首页文件里的内容须有你需要匹配的字符串如“hello”.测试这个web服务的两个首页在没有写规则前都可以正常访问。对其中一个web网页做字符串匹配。

{% highlight bash %}
{% raw %}

iptables -I OUTPUT -s 172.16.34.30 -p tcp –sport 80 -m string –algo kmp –string “hello” -f DROP

{% endraw %}
{% endhighlight %}

写完这个匹配字符串的规则后再次访问时显示是无法打开被匹配到的网页

# Iptabls防SYN攻击 #

首先说一下SYN的攻击原理：

在TCP/IP协议中，TCP协议提供可靠的连接服务，采用三次握手建立一个连接。

第一次握手：建立连接时，客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认;

第二次握手：服务器收到syn包，必须确认客户的SYN(ack=j+1)，同时自己也发送一个SYN包(syn=k)，即SYN+ACK包，此时服务器进入SYN_RECV状态;

第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。 完成三次握手，客户端与服务器开始传送数据.

如果用户与服务器发起连接请求只进行到第二次握手而不再响应服务器，服务器就会不停地等待用户的确认，如果过多这样的连接就会把服务器端的连接队列占满就会导致正常的用户无法建立连接。所以我们直接从SYN的连接上进行如下改动：

查看linux默认的syn配置：

{% highlight bash %}
{% raw %}

[root@web ~]# sysctl -a | grep _syn
net.ipv4.tcp_max_syn_backlog = 1024
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_synack_retries = 5
net.ipv4.tcp_syn_retries = 5

{% endraw %}
{% endhighlight %}

`tcp_max_syn_backlog` 是SYN队列的长度，加大SYN队列长度可以容纳更多等待连接的网络连接数。

`tcp_syncookies`是一个开关，是否打开SYN Cookie 功能，该功能可以防止部分SYN攻击。

`tcp_synack_retries`和`tcp_syn_retries`定义SYN 的重试连接次数，将默认的参数减小来控制SYN连接次数的尽量少。

以下是我修改后的参数，可以根据自己服务器的实际情况进行修改：

{% highlight bash %}
{% raw %}

[root@web ~]# more /etc/rc.d/rc.local
#!/bin/sh
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don’t
# want to do the full Sys V style init stuff.
touch /var/lock/subsys/local
ulimit -HSn 65535
/usr/local/apache2/bin/apachectl start
#####
sysctl -w net.ipv4.tcp_max_syn_backlog=2048
sysctl -w net.ipv4.tcp_syncookies=1
sysctl -w net.ipv4.tcp_synack_retries=3
sysctl -w net.ipv4.tcp_syn_retries=3

{% endraw %}
{% endhighlight %}

为了不重启服务器而使配置立即生效，可以执行

{% highlight bash %}
{% raw %}

#sysctl -w net.ipv4.tcp_max_syn_backlog=2048
#sysctl -w net.ipv4.tcp_syncookies=1
#sysctl -w net.ipv4.tcp_synack_retries=3
#sysctl -w net.ipv4.tcp_syn_retries=3

{% endraw %}
{% endhighlight %}

也有的人喜欢用访问控制列表来防止SYN的攻击，在一定程度上减缓了syn的攻击：

Syn 洪水攻击

{% highlight bash %}
{% raw %}

#iptables -A INPUT -p tcp –syn -m limit –limit 1/s -j ACCEPT

{% endraw %}
{% endhighlight %}

`–limit 1/s` 限制syn并发数每秒1次

防端口扫描

{% highlight bash %}
{% raw %}

# iptables -A FORWARD -p tcp –tcp-flags SYN,ACK,FIN,RST RST -m limit –limit 1/s -j ACCEPT

{% endraw %}
{% endhighlight %}

死亡之ping

{% highlight bash %}
{% raw %}

# iptables -A FORWARD -p icmp –icmp-type echo-request -m limit –limit 1/s -j ACCEPT
#>iptables-save >/etc/sysconfig/iptables

{% endraw %}
{% endhighlight %}

进行查看，`#iptables -L`

{% highlight bash %}
{% raw %}

ACCEPT tcp — anywhere anywhere tcp flags:FIN,SYN,RST,ACK/SYN limit: avg 1/sec burst 5
ACCEPT tcp — anywhere anywhere tcp flags:FIN,SYN,RST,ACK/RST limit: avg 1/sec burst 5
ACCEPT icmp — anywhere anywhere icmp echo-request limit: avg 1/sec burst 5

{% endraw %}
{% endhighlight %}

使用下面命令进行查看syn连接：

{% highlight bash %}
{% raw %}

[root@web ~]# netstat -an | grep SYN | awk ‘{print $5}’ | awk -F: ‘{print $1}’ | sort | uniq -c | sort -nr | more

{% endraw %}
{% endhighlight %}
