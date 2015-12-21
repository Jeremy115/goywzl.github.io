---
layout: article
title:  "LVS+Keepalived(DR模式)搭建简介"
date:   2015-12-23 09:20:00 +0800
categories: server
---


环境描述：

     操作系统：Centos6.5_x64 mini
     负载均衡模式：DR（直接路由）

软件简述：

LVS是Linux Virtual Server的简称，也就是Linux虚拟服务器, 是一个由章文嵩博士发起的自由软件项目，它的官方站点是`www.linuxvirtualserver.org`。

现在LVS已经是 Linux标准内核的一部分，在Linux2.4内核以前，使用LVS时必须要重新编译内核以支持LVS功能模块，但是从Linux2.4内核以后，已经完全内置了LVS的各个功能模块，无需给内核打任何补丁，可以直接使用LVS提供的各种功能。

使用LVS技术要达到的目标是：通过LVS提供的负载均衡技术和Linux操作系统实现一个高性能、高可用的服务器群集，它具有良好可靠性、可扩展性和可操作性。从而以低廉的成本实现最优的服务性能。

LVS自从1998年开始，发展到现在已经是一个比较成熟的技术项目了。可以利用LVS技术实现高可伸缩的、高可用的网络服务，例如WWW服务、Cache服务、DNS服务、FTP服务、MAIL服务、视频/音频点播服务等等，有许多比较著名网站和组织都在使用LVS架设的集群系统

例如：Linux的门户网站（www.linux.com）、向RealPlayer提供音频视频服务而闻名的Real公司（www.real.com）、全球最大的开源网站（sourceforge.net）等。

Keepalived的作用是检测web服务器的状态，如果有一台web服务器死机，或工作出现故障，Keepalived将检测到，并将有故障的web服务器从系统中剔除，当web服务器工作正常后Keepalived自动将web服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的web服务器

----------

# 一、配置主从LVS服务器 #

安装依赖包：

{% highlight bash %}
{% raw %}

[root@localhost ~]# yum install-y gcc gcc-c++ makepcre pcre-devel kernel-devel openssl-devel libnl-devel popt-devel

{% endraw %}
{% endhighlight %}

## 1、检查linux内核是否集成lvs模块： ##

{% highlight bash %}
{% raw %}

modprobe -l | grep ipvs

{% endraw %}
{% endhighlight %}

## 2、开启路由转发功能： ##

{% highlight bash %}
{% raw %}

echo “1”>/proc/sys/net/ipv4/ip_forward

{% endraw %}
{% endhighlight %}

## 3、安装ipvsadm: ##

{% highlight bash %}
{% raw %}

yum install -y ipvsadm

{% endraw %}
{% endhighlight %}

## 4、安装keepalived ##

{% highlight bash %}
{% raw %}

wget http://www.keepalived.org/software/keepalived-1.2.7.tar.gz 
tar zxvf keepalived-1.2.7.tar.gz 
cd keepalived-1.2.7 
./configure--prefix=/usr/local/keepalived
make && make install

{% endraw %}
{% endhighlight %}

将keepalived配置成系统服务

{% highlight bash %}
{% raw %}

cp  /usr/local/keepalived/etc/rc.d/init.d/keepalived/etc/init.d/ 
cp /usr/local/keepalived/etc/sysconfig/keepalived/etc/sysconfig/
mkdir /etc/keepalived/
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
cp /usr/local/keepalived/sbin/keepalived/usr/sbin/

{% endraw %}
{% endhighlight %}

## 5、修改主keepalived配置文件（备只修改router_id、state和priority） ##

{% highlight bash %}
{% raw %}

[root@localhost ~]# vi /etc/keepalived/keepalived.conf 
! Configuration File forkeepalived 
global_defs { 
notification_email { 
test@sina.com    #故障接受联系人 
 } 
notification_email_from admin@test.com  #故障发送人 
smtp_server 127.0.0.1  #本机发送邮件 
smtp_connect_timeout 30 
router_id LVS_MASTER  #BACKUP上修改为LVS_BACKUP 
 } 
vrrp_instance VI_1 { 
state MASTER    #BACKUP上修改为BACKUP 
interface eth0 
virtual_router_id 51  #虚拟路由标识，主从相同 
priority 100  #BACKUP上修改为90 
advert_int 1 
authentication { 
auth_type PASS 
auth_pass 1111  #主从认证密码必须一致 
 } 
virtual_ipaddress {    #Web虚拟IP（VTP） 
172.0.0.10 
  } 
 } 
virtual_server 172.0.0.10 80 { #定义虚拟IP和端口 
delay_loop 6    #检查真实服务器时间，单位秒 
lb_algo rr      #设置负载调度算法，rr为轮训 
lb_kind DR      #设置LVS负载均衡DR模式 
persistence_timeout 50 #同一IP的连接60秒内被分配到同一台真实服务器 
protocol TCP    #使用TCP协议检查realserver状态 
real_server 172.0.0.13 80 {  #第一个web服务器 
weight 3          #节点权重值 
TCP_CHECK {      #健康检查方式 
connect_timeout 3 #连接超时 
nb_get_retry 3    #重试次数 
delay_before_retry 3  #重试间隔/S 
  } 
} 
real_server 172.0.0.14 80 {  #第二个web服务器 
weight 3    #权重（服务器权重改为0，表示不可用）
TCP_CHECK { 
connect_timeout 3 
nb_get_retry 3 
delay_before_retry 3 
    } 
  } 
} 

{% endraw %}
{% endhighlight %}

# 二、分别在两台Web服务器编写脚本并启动 #

{% highlight bash %}
{% raw %}

[root@localhost ~]# vi /etc/init.d/real.sh 

#description : start realserver 
VIP=172.0.0.10 
. /etc/init.d/functions
case “$1” in
start) 
/sbin/ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up 
echo “1” >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo “2” >/proc/sys/net/ipv4/conf/lo/arp_announce
echo “1” >/proc/sys/net/ipv4/conf/all/arp_ignore
echo “2” >/proc/sys/net/ipv4/conf/all/arp_announce
echo “LVS RealServer Start OK”
;; 
stop) 
/sbin/ifconfig lo:0 down 
echo “0” >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo “0” >/proc/sys/net/ipv4/conf/lo/arp_announce
echo “0” >/proc/sys/net/ipv4/conf/all/arp_ignore
echo “0” >/proc/sys/net/ipv4/conf/all/arp_announce
echo “LVS RealServer Stoped OK”
;; 
*) 
echo “Usage: $0 {start|stop}”
exit 1 
esac
[root@localhost ~]# chmod +x /etc/init.d/real.sh 
[root@localhost ~]# /etc/init.d/real.sh start 
LVS RealServer Start OK 
[root@localhost ~]# echo “/etc/init.d/real.sh start” >> /etc/rc.local
[root@localhost ~]# service httpd start 
[root@localhost ~]# echo “192.168.0.30″ > /var/www/html/index.html 
[root@localhost ~]# echo “192.168.0.40″ > /var/www/html/index.html 
[root@localhost ~]# service iptables stop  #关闭防火墙 
[root@localhost ~]# setenforce 0  #临时关闭selinux

{% endraw %}
{% endhighlight %}

# 三、测试及常用命令 #

*http://172.0.0.10* #访问一直刷新会轮训显示172.0.0.11/12

模拟宕掉主LVS，服务器照常工作，再宕掉Web1，这时只会显示Web2，这样就实现ip负载均衡，高可用集群。

当主LVS恢复后，会切换成主动服务器，如果Keepalived监控模块检测web故障恢复后，恢复的主机又将此节点加入集群系统中。

常用命令：

{% highlight bash %}
{% raw %}

[root@localhost ~]# ipvsadm -ln #显示集群中服务器ip信息
[root@localhost ~]# ip addr #显示VTP绑定在哪个服务器上
[root@localhost ~]# tail -f /var/log/messger

{% endraw %}
{% endhighlight %}

(从日志中可知，主机出现故障后，备机立刻检测到，此时备机变为MASTER角色，并且接管了主机的虚拟IP资源，最后将虚拟IP绑定在etho设备上)