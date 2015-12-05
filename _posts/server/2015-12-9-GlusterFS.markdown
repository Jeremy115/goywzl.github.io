---
layout: article
title:  "GlusterFS"
date:   2015-12-09 09:20:00 +0800
categories: server
---

GlusterFS是一个开源的分布式文件系统,于2011年被红帽收购.它具有高扩展性、高性能、高可用性、可横向扩展的弹性特点,无元数据服务器设计使glusterfs没有单点故障隐患。

## 1.环境准备 ##

机器名 server01

eth0 192.168.100.30/24

说明 Centos 6.3 64bit ，多准备一块磁盘

机器名 server02

eth0 192.168.100.31/24

说明 Centos 6.3 64bit，多准备一块磁盘

机器名 Client

eth0  192.168.100.60/24

说明 Centos 6.3 64bit

## 2.格式磁盘并挂载（在两个节点上均做如下设置） ##

安装mkfs.xfs命令包

{% highlight bash %}
{% raw %}

yum install xfsprogs
mkfs.xfs -i size=512 /dev/sdb
mkdir -p /export/brick1

{% endraw %}
{% endhighlight %}

修改分区表 `vi /etc/fstab` 末尾添加

{% highlight bash %}
{% raw %}

/dev/sdb /export/brick1 xfs defaults 1 2
mount –a &&mount

{% endraw %}
{% endhighlight %}


## 3.安装gluster（所有节点均需安装） ##
 
由于yum不能直接安装，所以添加源

{% highlight bash %}
{% raw %}

wget -P /etc/yum.repos.d http://download.gluster.org/pub/gluster/glusterfs/LATEST/EPEL.repo/glusterfs-epel.repo

{% endraw %}
{% endhighlight %}

安装gluster

{% highlight bash %}
{% raw %}

yum install glusterfs{-fuse,-server}

{% endraw %}
{% endhighlight %}

启动服务

{% highlight bash %}
{% raw %}

service glusterd start
service gluster status

{% endraw %}
{% endhighlight %}


## 4.配置gluster ##

注：以下配置在一个节点上操作即可

配置对端信任节点（即镜像服务器）

{% highlight bash %}
{% raw %}

gluster peer probe 192.168.100.31
 
{% endraw %}
{% endhighlight %}

创建一个功能为镜像的集群卷（可以使用域名）

{% highlight bash %}
{% raw %}

gluster volume create gv0 replica 2 192.168.100.30:/export/brick1 192.168.100.31:/export/brick1
gluster volume start gv0

{% endraw %}
{% endhighlight %}

查看卷信息

{% highlight bash %}
{% raw %}

[root@server01 brick1]# gluster volume info
Volume Name: gv0
Type: Replicate
Volume ID: 3bf26a6a-63ce-44ed-bd57-fffd52352130
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 192.168.100.30:/export/brick1
Brick2: 192.168.100.31:/export/brick1

{% endraw %}
{% endhighlight %}

## 5.验证 ##

在客户端192.168.100.60上验证gluster，

1.安装gluster也就是步骤3的server组件就包含了客户端

2.只安装客户端组件

{% highlight bash %}
{% raw %}

yum -y install fuse fuse-libs
yum install glusterfs glusterfs-fuse glusterfs-rdma

{% endraw %}
{% endhighlight %}

自动挂载

修改/etc/fstab 文件

{% highlight bash %}
{% raw %}

server1:/test-volume /mnt/glusterfs glusterfs defaults,_netdev 0 0
192.168.100.30:/gv0 /mnt glusterfs defaults,netdev 0 0
mount -t glusterfs 192.168.100.30:/gv0 /mnt

[root@node mnt]# mount -l

/dev/mapper/VolGroup-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext=”system_u:object_r:tmpfs_t:s0″)
/dev/sda1 on /boot type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
sunrpc on /var/lib/nfs/rpc_pipefs type rpc_pipefs (rw)
/dev/sr0 on /media/CentOS_6.3_Final type iso9660 (ro,nosuid,nodev,uhelper=udisks,uid=500,gid=500,iocharset=utf8,mode=0400,dmode=0500) [CentOS_6.3_Final]
/dev/sr0 on /media/cdrom type iso9660 (ro) [CentOS_6.3_Final]
192.168.100.30:/gv0 on /mnt type fuse.glusterfs (rw,default_permissions,allow_other,max_read=131072)

{% endraw %}
{% endhighlight %}

已经将gv0挂载至/mnt目录，在客户端目录下创建100个文件

{% highlight bash %}
{% raw %}

for i in `seq -w 1 100`; do cp -rp /var/log/messages /mnt/copy-test-$i; done

[root@node mnt]# ls

copy-test-001 copy-test-013

[root@node mnt]# ls -lA /mnt | wc -l

101

{% endraw %}
{% endhighlight %}

到服务器192.168.100.30上查看验证

{% highlight bash %}
{% raw %}

[root@server01 brick1]# ls
copy-test-001 copy-test-013
[root@server01 brick1]# ls -l | wc -l
101

{% endraw %}
{% endhighlight %}

服务器192.168.100.31上查看验证

{% highlight bash %}
{% raw %}

[root@server02 brick1]# ls
copy-test-001 copy-test-013
[root@server02 brick1]# ls -l | wc -l
101

{% endraw %}
{% endhighlight %}

说明：上述例子用2台服务器上的一块硬盘做了镜像冗余，客户端上传文件会自动同步到2台服务器的gluster卷上。

## 6.故障恢复验证 ##

查看当前挂载情况，挂载的是server01的地址

{% highlight bash %}
{% raw %}

[root@node mnt]# mount
192.168.100.30:/gv0 on /mnt type fuse.glusterfs (rw,default_permissions,allow_other,max_read=131072)

{% endraw %}
{% endhighlight %}

查看当前数据server01和server02里均为空
在客户端创建测试数据，数据同步到2台服务器上

{% highlight bash %}
{% raw %}

[root@server02 brick1]# ls
copy-test-001 copy-test-013 copy-test-025

{% endraw %}
{% endhighlight %}

由于当前mount的地址为server01的地址，现在当server01断开网络后，验证客户端是否仍然可以与server02的卷交互保障业务不中断。

{% highlight bash %}
{% raw %}

[root@node mnt]# ping 192.168.100.30
PING 192.168.100.30 (192.168.100.30) 56(84) bytes of data.
From 192.168.100.60 icmp_seq=1 Destination Host Unreachable
^C
— 192.168.100.30 ping statistics —
3 packets transmitted, 0 received, +1 errors, 100% packet loss, time 2423ms
[root@node mnt]# ls
copy-test-001 copy-test-013 copy-test-025

{% endraw %}
{% endhighlight %}

由此可见当server01断开网络后，客户端仍然可以与server02交互数据。

此时在客户端删除并更新数据，server02仍然是断开前的数据，恢复后是否可以从server01同步回数据。

在客户端删除创建的测试数据，并新建文件test，server01跟进同步了

{% highlight bash %}
{% raw %}

[root@node mnt]# rm -rf *
[root@node mnt]# touch test
[root@node mnt]# ls
Test
[root@node mnt]# netstat -antulp |grep 192.168.*
tcp 0 52 192.168.100.60:22 192.168.100.1:49994 ESTABLISHED 2106/sshd
tcp 0 1 192.168.100.60:1023 192.168.100.30:24009 SYN_SENT 2358/glusterfs
tcp 0 1 192.168.100.60:1022 192.168.100.30:24007 SYN_SENT 2358/glusterfs
tcp 0 0 192.168.100.60:1021 192.168.100.31:24009 ESTABLISHED 2358/glusterfs
[root@server02 brick1]# ls
Test
server01里由于网络故障，导致数据无法更新
[root@server01 brick1]# ls
copy-test-001 copy-test-013

{% endraw %}
{% endhighlight %}

网络恢复后，间隔一段时间，发现server02同步了server01的数据

{% highlight bash %}
{% raw %}

[root@server01 brick1]# ls
Test
[root@node mnt]# netstat -antulp |grep 192.168.*
tcp 0 52 192.168.100.60:22 192.168.100.1:49994 ESTABLISHED 2106/sshd
tcp 0 0 192.168.100.60:1023 192.168.100.30:24009 ESTABLISHED 2358/glusterfs
tcp 0 0 192.168.100.60:1022 192.168.100.30:24007 ESTABLISHED 2358/glusterfs
tcp 0 0 192.168.100.60:1021 192.168.100.31:24009 ESTABLISHED 2358/glusterfs

{% endraw %}
{% endhighlight %}
 
由此可见当服务器中有一台故障后，客户端的业务仍然可以继续，不会受到影响，故障服务器恢复后，会从没有发生故障的服务器同步数据，来确保数据的一致性。

## 7.扩展gluster volume ##

Server01和server02均再增加一块10G大小的硬盘，原有的是20G的硬盘互相镜像，现在为gluster卷增加10G大小的空间

{% highlight bash %}
{% raw %}

[root@server01 ~]# mkfs.xfs -i size=512 /dev/sdc
mkdir -p /export/brick2

{% endraw %}
{% endhighlight %}
 
编辑vi/etc/fstab 末尾添加

{% highlight bash %}
{% raw %}

/dev/sdc                /export/brick2          xfs     defaults       1 2

{% endraw %}
{% endhighlight %}

原卷空间大小

{% highlight bash %}
{% raw %}

[root@node /]# df -h /mnt
Filesystem Size Used Avail Use% Mounted on
192.168.100.30:/gv0 20G 33M 20G 1% /mnt

{% endraw %}
{% endhighlight %}

Gluster卷情况

{% highlight bash %}
{% raw %}

[root@server01 ~]# gluster volume info
Volume Name: gv0
Type: Replicate
Volume ID: 3bf26a6a-63ce-44ed-bd57-fffd52352130
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: 192.168.100.30:/export/brick1
Brick2: 192.168.100.31:/export/brick1

{% endraw %}
{% endhighlight %}

增加新建的10G大小的卷

{% highlight bash %}
{% raw %}

# gluster volume add-brick gv0 192.168.100.30:/export/brick2 192.168.100.31:/export/brick2
[root@server01 brick1]# gluster volume info
Volume Name: gv0
Type: Distributed-Replicate
Volume ID: 3bf26a6a-63ce-44ed-bd57-fffd52352130
Status: Started
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: 192.168.100.30:/export/brick1
Brick2: 192.168.100.31:/export/brick1
Brick3: 192.168.100.30:/export/brick2
Brick4: 192.168.100.31:/export/brick2

{% endraw %}
{% endhighlight %}

客户端查看扩容后的卷大小

{% highlight bash %}
{% raw %}

[root@node /]# df -h /mnt
Filesystem Size Used Avail Use% Mounted on
192.168.100.30:/gv0 30G 65M 30G 1% /mnt

{% endraw %}
{% endhighlight %}

至此顺利的完成了对gluster镜像卷的扩容。

## 8.移除卷 ##

移除卷会将卷上的数据迁移到其他可用的卷上

{% highlight bash %}
{% raw %}

gluster volume remove-brick gv0192.168.100.32:/export/brick1 192.168.100.33:/export/brick1 start
gluster volume remove-brick gv0192.168.100.32:/export/brick1 192.168.100.33:/export/brick1 commit

{% endraw %}
{% endhighlight %}

查看信息发现server3和server4的卷已经移除，并且卷上的数据未丢失,已迁移到可用的卷上。

{% highlight bash %}
{% raw %}

gluster> volume info
Volume Name: gv0
Type: Distributed-Replicate
Volume ID:3bf26a6a-63ce-44ed-bd57-fffd52352130
Status: Started
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: 192.168.100.30:/export/brick1
Brick2: 192.168.100.31:/export/brick1
Brick3: 192.168.100.30:/export/brick2
Brick4: 192.168.100.31:/export/brick2

{% endraw %}
{% endhighlight %}

## 9.访问限制 ##

通过访问限制，允许某些特定主机访问存储，拒绝某些特定主机访问存储
 
{% highlight bash %}
{% raw %}

gluster volume set gv0 auth.allow192.168.100.32,192.168.100.60
gluster volume set gv0 auth.reject <ip-address>
gluster> volume info
Volume Name: gv0
Type: Distributed-Replicate
Volume ID:3bf26a6a-63ce-44ed-bd57-fffd52352130
Status: Started
Number of Bricks: 2 x 2 = 4
Transport-type: tcp
Bricks:
Brick1: 192.168.100.30:/export/brick1
Brick2: 192.168.100.31:/export/brick1
Brick3: 192.168.100.30:/export/brick2
Brick4: 192.168.100.31:/export/brick2
Options Reconfigured:
auth.reject: none
auth.allow: 192.168.100.32,192.168.100.60
diagnostics.latency-measurement: on
diagnostics.count-fop-hits: on


{% endraw %}
{% endhighlight %}


## 10.单机做gluster ##

Server03 有2块硬盘sdb、sdc希望通过gluster在这两块硬盘之间做镜像备份
格式化两块磁盘并挂载，如下：
 
{% highlight bash %}
{% raw %}
 
/dev/sdc on /export/brick2 type xfs (rw)
/dev/sdb on /export/brick1 type xfs (rw)
gluster volume create gv0 replica 2192.168.100.32:/export/brick1 192.168.100.32:/export/brick2
Multiple bricks of a replicate volume arepresent on the same server. This setup is not optimal.
Do you still want to continue creating thevolume? (y/n) y
Creation of volume gv0 has been successful.Please start the volume to access data.
[root@server03 /]# gluster volume start gv0
Starting volume gv0 has been successful
[root@server03 /]# gluster volume status
Status of volume: gv0
Gluster process Port Online Pid
——————————————————————————
Brick 192.168.100.32:/export/brick1 24009 Y 2252
Brick 192.168.100.32:/export/brick2 24010 Y 2259
NFS Server on localhost 38467 Y 2265
Self-heal Daemon on localhost N/A Y 2271

{% endraw %}
{% endhighlight %}

去客户端挂载测试
  
{% highlight bash %}
{% raw %}

[root@node /]# mount -t glusterfs192.168.100.32:/gv0 /volume/
[root@node /]# mount
192.168.100.32:/gv0 on /volume typefuse.glusterfs (rw,default_permissions,allow_other,max_read=131072)

{% endraw %}
{% endhighlight %}
