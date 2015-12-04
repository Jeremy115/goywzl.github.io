---
layout: article
title:  "linux-raid"
date:   2015-11-29 09:20:00 +0800
categories: linux
---

RAID我想大家都不陌生，我公司买过几块硬盘，要做一下raid，我装在了服务器上，一般服务器上如果带raid卡的话，就直接做了，这是硬raid，我也有文档，这里主要介绍的是linux系统下做的软raid。

在介绍之前，还是先介绍一下raid模式

RAID0 : 称为条带化(Striping)存储，将数据分段存储于 各个磁盘中，读写均可以并行处理。因此其读写速率为单个磁盘的N倍(N为组成RAID0的磁盘个数)，但是却没有数 据冗余，单个磁盘的损坏会导致数据的不可修复。

RAID1 : 镜像存储(mirroring)，没有数据校验。数据被同等地写入两个或多个磁盘中，可想而知，写入速度会比较 慢，但读取速度会比较快。读取速度可以接近所有磁盘吞吐量的总和，写入速度受限于最慢 的磁盘。 RAID1也是磁盘利用率最低的一个。如果用两个不同大小的磁盘建立RAID1，可以用空间较小 的那一个，较大的磁盘多出来的部分可以作他用，不会浪费。

RAID2 ： RAID0的改良版，加入了汉明码(Hanmming Code)错误校验。使用一定的编码技术来提供错误检查及恢复。

RAID3 ： 类似于RAID2，数据条带化(stripe)存储于不同的硬盘，数据以字节为单位，只是RAID3使用单块磁盘存储简单的 奇偶校验信息，所以最终磁盘数量为 N+1 。当这N+1个硬盘中的其中一个硬盘出现故障时， 从其它N个硬盘中的数据也可以恢复原始数据，当更换一个新硬盘后，系统可以重新恢复完整 的校验容错信息。

RAID4 ： 与RAID3类似，但RAID4是按块(扇区)存取。无须像RAID3那样，哪怕每一次小I/O操作也要涉 及全组，只需涉及组中两块硬盘（一块数据盘，一块校验盘）即可，从而提高了小量数据 I/O速度。

RAID5 ： 奇偶校验(XOR)，数据以块分段条带化存储。校验信息交叉地存储在所有的数据盘上。RAID5把数据和相对应的奇偶校验信息存储到组成RAID5的各个磁盘上，并且奇偶校验信息和 相对应的数据分别存储于不同的磁盘上，其中任意N-1块磁盘上都存储完整的数据，也就是 说有相当于一块磁盘容量的空间用于存储奇偶校验信息。

RAID6 ： 类似RAID5，但是增加了第二个独立的奇偶校验信息块，两个独立的奇偶系统使用不同的算法， 数据的可靠性非常高，即使两块磁盘同时失效也不会影响数据的使用。但RAID 6需要分配给 奇偶校验信息更大的磁盘空间，相对于RAID 5有更大的“写损失”，因此“写性能”非常差。

混合RAID ： 如RAID10和RAID01就是组合RAID0于RAID1，两个RAID先后顺序不同而已。


----------

下面介绍我是如何在linux系统下RAID的。

Linux下使用mdadm创建和管理软raid注：本次操作以RHEL4为例，但应该可以应用到其它大部分的distro上（guess）。

mdadm的几个常用参数

    -C 创建Raid，后面跟参数，代表raid设备的名称。比如：/dev/md0，/dev/md1。 
    -n 用于创建磁盘阵列的磁盘个数。 
    -l Raid的级别。 
    -x 指定用于hotspare（热备盘）的磁盘个数。如果阵列中有一块硬盘坏了，它会立刻顶上，并rebuild； 
    -D 显示软raid的详细信息； 
    -s 扫描配置文件（/etc/mdadm.conf）或'/proc/mdstat'来查看遗漏的信息f；创建软raid的大体流程使用fdisk工具为新磁盘创建分区； 

使用mkfs.XXXX工具将刚才划分好的分区格式化成某种格式的文件系统。比如：ext3，reiserfs等； 

使用mdadm来创建软raid；
 
创建/etc/mdadm.conf文件（注意文件的格式，包括是否有逗号等等。该文件是为了系统在重启后能够自动启用软raid。可以查看/etc/rc.sysinit脚本，搜索'mdadm'字符串就明白了）；

示例：创建软raid5（+hotspare）以下是我的一次实际操作的完整过程：这是用'fdisk -l'命令查看到的我当前的磁盘和分区情况（只有/dev/sda在使用，其它四个都是新磁盘，没有分区，没有格式化）：

查看磁盘信息：

{% highlight bash %}
{% raw %}

# fdisk -lDisk /dev/sda: 6442 MB, 6442450944 bytes
255 heads, 63 sectors/track, 783 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes   Device Boot      Start         End      Blocks   Id System
/dev/sda1   *           1         720     5783368+ 83 Linux
/dev/sda2             721         783      506047+ 82 Linux swapDisk /dev/sdb: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytesDisk /dev/sdb doesn't contain a valid partition tableDisk /dev/sdc: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytesDisk /dev/sdc doesn't contain a valid partition tableDisk /dev/sdd: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytesDisk /dev/sdd doesn't contain a valid partition tableDisk /dev/sde: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytesDisk /dev/sde doesn't contain a valid partition table

{% endraw %}
{% endhighlight %}

使用fdisk创建分区（本例中将整块磁盘划分为一个主分区。其余几块磁盘也做相同的操作。）：
 
{% highlight bash %}
{% raw %}

# fdisk /dev/sdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel. Changes will remain in memory only,
until you decide to write them. After that, of course, the previous
content won't be recoverable.Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-204, default 1):
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-204, default 204):
Using default value 204Command (m for help): w
The partition table has been altered!Calling ioctl() to re-read partition table.
Syncing disks.

{% endraw %}
{% endhighlight %}

为刚才新建的分区建立文件系统（其余几个分区依次做相同的操作）： 
 
{% highlight bash %}
{% raw %}

# mkfs.ext3 /dev/sdb1
mke2fs 1.35 (28-Feb-2004)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
52416 inodes, 208880 blocks
10444 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=67371008
26 block groups
8192 blocks per group, 8192 fragments per group
2016 inodes per group
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729, 204801Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: doneThis filesystem will be automatically checked every 37 mounts or
180 days, whichever comes first. Use tune2fs -c or -i to override.

{% endraw %}
{% endhighlight %}

用'fdisk -l'查看磁盘及分区状态：

{% highlight bash %}
{% raw %}

# fdisk -lDisk /dev/sda: 6442 MB, 6442450944 bytes
255 heads, 63 sectors/track, 783 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes   Device Boot      Start         End      Blocks   Id System
/dev/sda1   *           1         720     5783368+ 83 Linux
/dev/sda2             721         783      506047+ 82 Linux swapDisk /dev/sdb: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes   Device Boot      Start         End      Blocks   Id System
/dev/sdb1               1         204      208880   83 LinuxDisk /dev/sdc: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes   Device Boot      Start         End      Blocks   Id System
/dev/sdc1               1         204      208880   83 LinuxDisk /dev/sdd: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes   Device Boot      Start         End      Blocks   Id System
/dev/sdd1               1         204      208880   83 LinuxDisk /dev/sde: 214 MB, 214748160 bytes
64 heads, 32 sectors/track, 204 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes   Device Boot      Start         End      Blocks   Id System
/dev/sde1               1         204      208880   83 Linux

{% endraw %}
{% endhighlight %}

使用mdadm创建一个软raid，raid级别：5；并有一个hotspare盘： 

# mdadm -C /dev/md0 -l5 -n3 -x1 /dev/sd[b-e]1
mdadm: array /dev/md0 started.
输出信息显示软raid(/dev/md0)已经启用了。使用mdadm的-D参数（--detail）可以查
看软raid状态：

{% highlight bash %}
{% raw %}

# mdadm -D /dev/md0
/dev/md0:
        Version : 00.90.01
Creation Time : Wed Aug 23 15:10:19 2006
     Raid Level : raid5
     Array Size : 417536 (407.75 MiB 427.56 MB)
    Device Size : 208768 (203.88 MiB 213.78 MB)
   Raid Devices : 3
Total Devices : 4
Preferred Minor : 0
    Persistence : Superblock is persistent    Update Time : Wed Aug 23 15:10:21 2006
          State : clean
Active Devices : 3
Working Devices : 4
Failed Devices : 0
Spare Devices : 1         Layout : left-symmetric
     Chunk Size : 64K    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
       2       8       49        2      active sync   /dev/sdd1
       3       8       65       -1      spare   /dev/sde1
           UUID : f8283de5:39c73d89:b9fbc266:fdceb416
         Events : 0.2生成配置文件（/etc/mdadm.conf）：# mdadm -D -s >/etc/mdadm.conf

{% endraw %}
{% endhighlight %}

查看一下：

{% highlight bash %}
{% raw %}

# cat /etc/mdadm.conf
ARRAY /dev/md0 level=raid5 num-devices=3 UUID=f8283de5:39c73d89:b9fbc266:fdceb416
   devices=/dev/sdb1,/dev/sdc1,/dev/sdd1,/dev/sde1

{% endraw %}
{% endhighlight %}

修改（把上面devices=后面的磁盘，都放到DEVICE后面，并且不要逗号。而/dev/md0之后
的内容，都要用逗号来分隔）：DEVICE /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
ARRAY /dev/md0 level=raid5,num-devices=3,UUID=f8283de5:39c73d89:b9fbc266:fdceb416

重启一下，检测配置好的软raid是否能够在系统重启后自动启用。重启后，查看'/proc/mdstat'文件就可以看到软raid的状态：

{% highlight bash %}
{% raw %}

# cat /proc/mdstat
Personalities : [raid5]
md0 : active raid5 sdb1[0] sde1[3] sdd1[2] sdc1[1]
      417536 blocks level 5, 64k chunk, algorithm 2 [3/3] [UUU]unused devices: <none>

{% endraw %}
{% endhighlight %}

That's all.出现故障后的恢复

这里指的出现故障，是指raid中的一块磁盘出现了故障，无法使用。这时候需要使用额外的
磁盘来代替它。这里以强制将某块磁盘标记为已损坏，来模拟实际出现故障（注：新的磁盘
的容量最好和已损坏的磁盘一致）：将/dev/sdb1标记为已损坏：

{% highlight bash %}
{% raw %}

# mdadm /dev/md0 -f /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md0

{% endraw %}
{% endhighlight %}

这时候使用mdadm的-D参数来查看状态，可以看到/dev/sdb1已经被认为是faulty，而
hotspare（热备）盘'/dev/sde1'已经顶替了它的位置（这就是hotspare的作用）：

{% highlight bash %}
{% raw %}


# mdadm -D /dev/md0
/dev/md0:
        Version : 00.90.01
Creation Time : Wed Aug 23 15:10:19 2006
     Raid Level : raid5
     Array Size : 417536 (407.75 MiB 427.56 MB)
    Device Size : 208768 (203.88 MiB 213.78 MB)
   Raid Devices : 3
Total Devices : 4
Preferred Minor : 0
    Persistence : Superblock is persistent    Update Time : Wed Aug 23 15:42:24 2006
          State : clean
Active Devices : 3
Working Devices : 3
Failed Devices : 1
Spare Devices : 0         Layout : left-symmetric
     Chunk Size : 64K    Number   Major   Minor   RaidDevice State
       0       8       65        0      active sync   /dev/sde1
       1       8       33        1      active sync   /dev/sdc1
       2       8       49        2      active sync   /dev/sdd1
       3       8       17       -1      faulty   /dev/sdb1
           UUID : f8283de5:39c73d89:b9fbc266:fdceb416
         Events : 0.4

{% endraw %}
{% endhighlight %}

既然'/dev/sdb1'出现了故障，当然就要将它移除：

{% highlight bash %}
{% raw %}


# mdadm /dev/md0 -r /dev/sdb1
mdadm: hot removed /dev/sdb1

{% endraw %}
{% endhighlight %}

现在可以关机了。关机之后拔下这块已损坏的磁盘了，换上你的新磁盘。换好之
后，分区，mkfs.XXXX。然后将它加入到软raid中：

{% highlight bash %}
{% raw %}


# mdadm /dev/md0 -a /dev/sdb1
mdadm: hot added /dev/sdb1

{% endraw %}
{% endhighlight %}

这时候再使用mdadm的'-D'参数，可以看到sdb1已经作为hotspare盘了：

{% highlight bash %}
{% raw %}

# mdadm -D /dev/md0
/dev/md0:
        Version : 00.90.01
Creation Time : Wed Aug 23 15:10:19 2006
     Raid Level : raid5
     Array Size : 417536 (407.75 MiB 427.56 MB)
    Device Size : 208768 (203.88 MiB 213.78 MB)
   Raid Devices : 3
Total Devices : 4
Preferred Minor : 0
    Persistence : Superblock is persistent    Update Time : Wed Aug 23 16:19:36 2006
          State : clean
Active Devices : 3
Working Devices : 4
Failed Devices : 0
Spare Devices : 1         Layout : left-symmetric
     Chunk Size : 64K    Number   Major   Minor   RaidDevice State
       0       8       65        0      active sync   /dev/sde1
       1       8       33        1      active sync   /dev/sdc1
       2       8       49        2      active sync   /dev/sdd1
       3       8       17       -1      spare   /dev/sdb1
           UUID : f8283de5:39c73d89:b9fbc266:fdceb416
         Events : 0.6
misc

{% endraw %}
{% endhighlight %}

假如创建了RAID，但是没有生成 /etc/mdadm.conf 文件，那么系统重启后是
不会启用RAID的，这时候需要这样做： 

{% highlight bash %}
{% raw %}

# mdadm -A /dev/md0 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1

{% endraw %}
{% endhighlight %}

本文讲的是不支持硬raid的服务器，如果支持，最好还是用硬raid
