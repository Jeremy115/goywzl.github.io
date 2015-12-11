---
layout: article
title:  "lvs+keepalived+ftp"
date:   2015-12-15 09:20:00 +0800
categories: server
---


# 双节点FTP负载均衡服务环境部署  #

部署环境 

操作系统

Red Hat Enterprise Linux 6 Server x86_64 

Keepalived Master ：172.16.2.101 

Keepalived Slave ：172.16.2.102 

描述： 

Keepalived服务器既负责LVS分发同时也提供FTP服务。
 
环境中需要将FTP配置为被动连接方式，并需要iptables对FTP服务端口进行MARK标记。
 
*注意事项： 

FTP负载均衡服务需要iptables的配合，所以iptables不可以关闭。* 

# 一、配置Linux操作系统 # 

## 1、修改本地解析文件，添加如下行  ##

{% highlight bash %}
{% raw %}

# grep lvs-node /etc/hosts 
172.16.2.101 lvs-node1-el6.rhev.com 
172.16.2.102 lvs-node2-el6.rhev.com

{% endraw %}
{% endhighlight %}

## 2、为回环网卡添加检测IP  ##

### 1）创建ifcfg-lo:1配置文件  ###

{% highlight bash %}
{% raw %}

# cat /etc/sysconfig/network-scripts/ifcfg-lo\:1 
DEVICE=lo:1 
ONBOOT=yes 
NM_CONTROLLED=no 
BOOTPROTO=static 
IPADDR=172.16.2.100 
NETMASK=255.255.255.255 

{% endraw %}
{% endhighlight %}

### 2）重新启动网络服务  ###

{% highlight bash %}
{% raw %}

# service network restart 

{% endraw %}
{% endhighlight %}

## 3、开启IP转发并忽略ARP ## 

### 1）添加如下参数  ###

{% highlight bash %}
{% raw %}

# echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf 
# echo 'net.ipv4.conf.eth0.arp_ignore = 1' >> /etc/sysctl.conf 
# echo 'net.ipv4.conf.eth0.arp_announce = 2' >> /etc/sysctl.conf 

{% endraw %}
{% endhighlight %}

### 2）立即生效并验证  ###

{% highlight bash %}
{% raw %}

# sysctl -p 2> /dev/null|grep -E 'net.ipv4.ip_forward|net.ipv4.conf.eth0.arp' 
net.ipv4.ip_forward = 1 
net.ipv4.conf.eth0.arp_ignore = 1 
net.ipv4.conf.eth0.arp_announce = 2
 
{% endraw %}
{% endhighlight %}

## 4、开启iptables的MARK标记  ##

### 1）添加新规则  ###

{% highlight bash %}
{% raw %}

# iptables -t mangle -A PREROUTING -p tcp -d 172.16.2.200 --dport 21 -j MARK --set-mark 21 
# iptables -t mangle -A PREROUTING -p tcp -d 172.16.2.200 --dport 10000:20000 -j MARK --set-mark 21 

{% endraw %}
{% endhighlight %}

### 2）保存规则  ###

{% highlight bash %}
{% raw %}

# /etc/init.d/iptables save 

{% endraw %}
{% endhighlight %}

# 二、安装并配置FTP服务 #

## 1、安装FTP服务  ##

{% highlight bash %}
{% raw %}

# yum install vsftpd -y 

{% endraw %}
{% endhighlight %}

## 2、对FTP服务添加如下行  ##

{% highlight bash %}
{% raw %}

# grep pasv /etc/vsftpd/vsftpd.conf 
pasv_min_port=10000 
pasv_max_port=20000 

{% endraw %}
{% endhighlight %}

注意FTP服务需要采用被动方式。 

## 3、启动FTP服务，并配置为开机运行  ##

{% highlight bash %}
{% raw %}

# service vsftpd start 
# chkconfig vsftpd on 

{% endraw %}
{% endhighlight %}

# 三、安装并配置Keepalived环境 #
 
## 1、安装Keepalived组件  ##

{% highlight bash %}
{% raw %}

# yum install keepalived 

{% endraw %}
{% endhighlight %}

## 2、编辑配置Keepalived组件，模板如下  ##

{% highlight bash %}
{% raw %}

# cat /etc/keepalived/keepalived.conf 

! Configuration File for keepalived 

global_defs { 
notification_email { 
acassen@firewall.loc 
failover@firewall.loc 
sysadmin@firewall.loc 
} 
notification_email_from Alexandre.Cassen@firewall.loc 
smtp_server 192.168.200.1 
smtp_connect_timeout 30 
router_id LVS_DEVEL 
} 

vrrp_instance VI_1 { 
state MASTER <-- 备机修改为SLAVE 
interface eth0 
virtual_router_id 51 
priority 100 <-- 备机修改为小于该值 
advert_int 1 
authentication { 
auth_type PASS 
auth_pass 1111 
} 
virtual_ipaddress { 
172.16.2.100 
fwmark 21 
} 
} 

virtual_server fwmark 21 { 
delay_loop 6 
lb_algo rr 
lb_kind DR 
persistence_timeout 50 
protocol TCP 

real_server 172.16.2.101 21 { 
weight 1 
TCP_CHECK { 
connect_port 21 
connect_timeout 3 
nb_get_retry 3 
delay_before_retry 3 
} 
} 

real_server 172.16.2.102 21 { 
weight 1 
TCP_CHECK { 
connect_port 21 
connect_timeout 3 
nb_get_retry 3 
delay_before_retry 3 
} 
} 

} 

{% endraw %}
{% endhighlight %}

## 3、同步Keepalived组件配置文件  ##

{% highlight bash %}
{% raw %}

# scp /etc/keepalived/keepalived.conf lvs-node2-el6.rhev.com:/etc/keepalived/ 

{% endraw %}
{% endhighlight %}

## 4、启动Keepalived组件，并配置为开机运行（主备节点均需要开启） ## 

{% highlight bash %}
{% raw %}

# service keepalived start 
# chkconfig keepalived on 

{% endraw %}
{% endhighlight %}

# 四、验证  #

实时检测分发状态 

{% highlight bash %}
{% raw %}

# watch -n1 ipvsadm -Ln 

{% endraw %}
{% endhighlight %}

创建测试文件 

{% highlight bash %}
{% raw %}

# echo "$(hostname)" > var/ftp/$(hostname).txt 

{% endraw %}
{% endhighlight %}

使用不同主机进行测试。