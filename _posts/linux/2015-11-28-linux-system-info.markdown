---
layout: article
title:  "linux-system-info"
date:   2015-11-28 09:20:00 +0800
categories: linux
---


查看硬件信息，或者查看系统资源，在windows下，可以打开电脑属性，或者计算机管理器里查看，查看机器负载可以用任务管理器，也可以下载某某某软件程序查看，但是在linux下，只需要使用命令就好了，虽然不直观，但是相对还是比较方便的。

查看系统信息的命令

    uname -a # 查看内核/操作系统/CPU信息
    head -n 1 /etc/issue # 查看操作系统版本
    cat /proc/cpuinfo # 查看CPU信息
    hostname # 查看计算机名
    lspci -tv # 列出所有PCI设备
    lsusb -tv # 列出所有USB设备
    lsmod # 列出加载的内核模块
    env # 查看环境变量

查看系统资源的命令

    free -m # 查看内存使用量和交换区使用量
    df -h # 查看各分区使用情况
    du -sh <目录名> # 查看指定目录的大小
    grep MemTotal /proc/meminfo # 查看内存总量
    grep MemFree /proc/meminfo # 查看空闲内存量
    uptime # 查看系统运行时间、用户数、负载
    cat /proc/loadavg # 查看系统负载

查看磁盘和分区的命令

    mount | column -t # 查看挂接的分区状态
    fdisk -l # 查看所有分区  www.2cto.com  
    swapon -s # 查看所有交换分区
    hdparm -i /dev/hda # 查看磁盘参数(仅适用于IDE设备)
    dmesg | grep IDE # 查看启动时IDE设备检测状况

查看网络信息命令

    ifconfig # 查看所有网络接口的属性
    iptables -L # 查看防火墙设置
    route -n # 查看路由表
    netstat -lntp # 查看所有监听端口
    netstat -antp # 查看所有已经建立的连接
    netstat -s # 查看网络统计信息

查看系统进程命令

    ps -ef # 查看所有进程
    top # 实时显示进程状态

查看活动用户命令

    w # 查看活动用户
    id <用户名> # 查看指定用户信息
    last # 查看用户登录日志
    cut -d: -f1 /etc/passwd # 查看系统所有用户
    cut -d: -f1 /etc/group # 查看系统所有组
    crontab -l # 查看当前用户的计划任务

查看系统服务命令

    chkconfig –list # 列出所有系统服务
    chkconfig –list | grep on # 列出所有启动的系统服务

查看系统程序命令

    rpm -qa # 查看所有安装的软件包

查看硬件信息命令

    dmidecode | grep -i 'serial number'#查看主板序列号
    service kudzu start #硬件检测程序kudzu探测新硬件
    cat /proc/cpuinfo [dmesg | grep -i ‘cpu’][dmidecode -t processor]   #查看cpu信息
    cat /proc/meminfo [free -m][vmstat]   #查看内存信息
    cat /proc/pci #查看板卡信息
    lspci |grep -i ‘VGA’[dmesg | grep -i ‘VGA’]   查看显卡、声卡信息
    dmesg | grep -i eth’[cat /etc/sysconfig/hwconf | grep -i eth][lspci | grep -i ‘eth’]    #查看网卡信息
	dmesg | more   #查看硬件信息
    lspci #查看外设信息、PCI信息（比cat /proc/pci更直观）
    cat /proc/bus/usb/devices   #查看usb设备
    cat /proc/bus/input/devices   #查看键盘、鼠标
    fdsik & disk -l & df   #查看系统硬盘信息和使用情况
    cat /proc/interrupts   #查看各设备的中断请求（IRQ）
	lshw    #检测硬件信息
	psrinfo -v    #查看当前处理器的类型和速度（主频）
	prtconf -v    #打印当前的OBP版本号
	iostat –E    #查看硬盘物理信息(vendor, RPM, Capacity)
	prtvtoc /dev/rdsk/c0t0d0s   #查看磁盘的几何参数和分区信息
	df –F ufs –o i      #显示已经使用和未使用的i-node数目


查看系统信息命令

    uname -a   #查看系统体系结构
    isalist –v [isainfo –v][isainfo –b] #查看及启动系统的32位或64位内核模式
    dmidecode #查看硬件信息，包括bios、cpu、内存等信息
    /usr/sbin/ffbconfig –rev \? #测定当前的显示器刷新频率
    /usr/platform/sun4u/sbin/prtdiag –v#查看系统配置
    showrev –p  #查看当前系统中已经应用的补丁
    who –rH #显示当前的运行级别
    nslookup –class=chaos –q=txt version.bind #查看当前的bind版本信息
    lsnod  #查看已加载的驱动

看过linux文件系统结构的，都知道"/proc"目录下是存放硬件信息的

对于“/proc”中文件可使用文件查看命令浏览其内容，文件中包含系统特定信息：

    Cpuinfo 主机CPU信息
    Dma 主机DMA通道信息
    Filesystems 文件系统信息
    Interrupts 主机中断信息
    Ioprots 主机I/O端口号信息
    Meninfo 主机内存信息
    Version Linux内存版本信息
    备注： proc – process information pseudo-filesystem 进程信息伪装文件系统

汇总常用的proc文件查看信息命令：

    cat /proc/cpuinfo  #查看CPU信息，内容很全哦！
    cat /proc/meminfo  #查看内存信息。
    cat /proc/ioports  #查看IO端口
    cat /proc/swaps  #查看交换分区信息(/proc)
    cat /proc/interrupts  #中断信息
    cat /proc/partitions  #查看磁盘分区
    cat /proc/bus/usb/devices  #查看USB设备
    cat /proc/bus/input/devices  #查看输入设备：键盘鼠标
    cat /proc/bus/pci/devices  #查看PCI设备
    cat /proc/loadavg  #查看系统负载
    cat /var/log/demsg  #查看开机检查的硬件，可以使用grep过虑：eth,cpu,mem,pci,usb,vga,sda……

汇总常用的系统查看命令：

    lscpu  #查看CPU信息
    lspci  #查看PCI设备
    lsusb  #查看USB设备
    vmstat  #报告虚拟内存统计信息
    fdisk -l  #查看分区信息
    hdparm -i /dev/sda  #查看磁盘参数
    df -h  #查看磁盘分信息
    dmidecode  #读取系统DMI表来显示硬件和BIOS信息。
    lsmod  #当前加载的驱动
    dmesg  #查看开机检查的硬件，可以使用grep过虑：eth,cpu,mem,pci,usb,vga,sda……
    uptime  #查看系统负载

有一些会重复说道，主要还是方便我查看，请见谅

