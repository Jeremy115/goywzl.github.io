---
layout: article
title:  "Linux检测硬盘I/O读写速度"
date:   2016-01-03 09:20:00 +0800
categories: linux
---

想要知道磁盘的I/O读写速度吗？试一试下面两种测试方法把

下面是两种测试方法：

## (1)使用hdparm命令 ##

这是一个是用来获取ATA/IDE硬盘的参数的命令,是由早期Linux IDE驱动的开发和维护人员 Mark Lord开发编写的( hdparm has been written by Mark Lord <mlord@pobox.com>, the primary developer and maintainer of the (E)IDE driver for Linux, with suggestions from many netfolk).该命令应该也是仅用于Linux系统,对于UNIX系统，ATA/IDE硬盘用的可能比较少，一般大型的系统都是使用磁盘阵列的.

功能说明：显示与设定硬盘的参数。

语　　法：hdparm [-CfghiIqtTvyYZ][-a <快取分区>][-A <0或1>][-c <I/O模式>][-d <0或1>][-k <0或1>][-K <0或1>][-m <分区数>][-n <0或1>][-p <PIO模式>][-P <分区数>][-r <0或1>][-S <时间>][-u <0或1>][-W <0或1>][-X <传输模式>][设备]

补充说明：hdparm可检测，显示与设定IDE或SCSI硬盘的参数。

参　　数：

      -a<快取分区>   设定读取文件时，预先存入块区的分区数，若不加上<快取分区>选项，则显示目前的设定。
      -A<0或1>   启动或关闭读取文件时的快取功能。
      -c<I/O模式>   设定IDE32位I/O模式。
      -C   检测IDE硬盘的电源管理模式。
      -d<0或1>   设定磁盘的DMA模式。
      -f   将内存缓冲区的数据写入硬盘，并清楚缓冲区。
      -g   显示硬盘的磁轨，磁头，磁区等参数。
      -h   显示帮助。
      -i   显示硬盘的硬件规格信息，这些信息是在开机时由硬盘本身所提供。
      -I   直接读取硬盘所提供的硬件规格信息。
      -k<0或1>   重设硬盘时，保留-dmu参数的设定。
      -K<0或1>   重设硬盘时，保留-APSWXZ参数的设定。
      -m<磁区数>   设定硬盘多重分区存取的分区数。
      -n<0或1>   忽略硬盘写入时所发生的错误。
      -p<PIO模式>   设定硬盘的PIO模式。
      -P<磁区数>   设定硬盘内部快取的分区数。
      -q   在执行后续的参数时，不在屏幕上显示任何信息。
      -r<0或1>   设定硬盘的读写模式。
      -S<时间>   设定硬盘进入省电模式前的等待时间。
      -t   评估硬盘的读取效率。
      -T   平谷硬盘快取的读取效率。
      -u<0或1>   在硬盘存取时，允许其他中断要求同时执行。
      -v   显示硬盘的相关设定。
      -W<0或1>   设定硬盘的写入快取。
      -X<传输模式>   设定硬盘的传输模式。
      -y   使IDE硬盘进入省电模式。
      -Y   使IDE硬盘进入睡眠模式。
      -Z   关闭某些Seagate硬盘的自动省电功能。

介绍一个简单的实例

{% highlight bash %}
{% raw %}

# hdparm -Tt /dev/sda
/dev/sda:
Timing cached reads: 6676 MB in 2.00 seconds = 3340.18 MB/sec
Timing buffered disk reads: 218 MB in 3.11 seconds = 70.11 MB/sec
 
{% endraw %}
{% endhighlight %}

可以看到,2秒钟读取了6676MB的缓存,约合3340.18 MB/sec;

在3.11秒中读取了218MB磁盘(物理读),读取速度约合70.11 MB/sec

## (2)使用dd命令 ##

这不是一个专业的测试工具，不过如果对于测试结果的要求不是很苛刻的话,平时可以使用来对磁盘的读写速度作一个简单的评估.
另外由于这是一个免费软件,基本上×NIX系统上都有安装,对于Oracle裸设备的复制迁移,dd工具一般都是首选.

在使用前首先了解两个特殊设备

    /dev/null 伪设备,回收站.写该文件不会产生IO
    /dev/zero 伪设备,会产生空字符流,对它不会产生IO

测试方法:

#### a.测试磁盘的IO写速度 ####

{% highlight bash %}
{% raw %}

# time dd if=/dev/zero of=/test.dbf bs=8k count=300000
300000+0 records in
300000+0 records out
10.59s real 0.43s user 9.40s system
# du -sm /test.dbf
2347 /test.dbf
 
{% endraw %}
{% endhighlight %}

可以看到,在10.59秒的时间里，生成2347M的一个文件,IO写的速度约为221.6MB/sec;

当然这个速度可以多测试几遍取一个平均值,符合概率统计.

#### b.测试磁盘的IO读速度 ####

{% highlight bash %}
{% raw %}

# df -m
Filesystem 1M-blocks Used Available Use% Mounted on
/dev/mapper/VolGroup00-LogVol00
19214 9545 8693 53% /
/dev/sda1 99 13 82 14% /boot
none 506 0 506 0% /dev/shm
# time dd if=/dev/mapper/VolGroup00-LogVol00 of=/dev/null bs=8k
2498560+0 records in
2498560+0 records out
247.99s real 1.92s user 48.64s system
 
{% endraw %}
{% endhighlight %}

上面的试验在247.99秒的时间里读取了19214MB的文件,计算下来平均速度为77.48MB/sec

#### c.测试IO同时读和写的速度 ####

{% highlight bash %}
{% raw %}

# time dd if=/dev/sda1 of=test.dbf bs=8k
13048+1 records in
13048+1 records out
3.73s real 0.04s user 2.39s system
# du -sm test.dbf
103 test.dbf
 
{% endraw %}
{% endhighlight %}

上面测试的数据量比较小，仅作为参考.