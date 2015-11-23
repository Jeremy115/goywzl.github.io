---
layout: article
title:  "CentOS下缓存"
date:   2015-11-22 15:34:30 +0800
categories: home linux
---


在linux的内存分配机制中，优先使用物理内存，在物理内存接近饱和时，系统会自动将不常用的内存文件转储到`SWAP`中，但`SWAP`使用率达30%的时候对系统性能可能有一定影响。

下面介绍`SWAP`的一些操作
 
1.关闭`SWAP`

一般用于大物理内存的服务器，执行如下命令，即关闭`SWAP`分区

    swapoff -a

2.开启`SWAP`

说了关闭，肯定要说如何开启的，`SWAP`对于某些程序还是有用的，我似乎记得`oracle`安装的时候，似乎报过没有`SWAP`分区的错误，记不清了。你可以适当的分配少一点

    swapon -a

3.刷新`SWAP`

当`SWAP`占用率高达30%，对系统性能可能会有一定影响，所以在适当情况下，我们可以执行上述的两个命令刷新一次`SWAP`（将`SWAP`里的数据转储回内存，并清空`SWAP`里的数据）刷新的原理，就是把swap关闭，然后再开启达到重启效果，执行如下命令，即可

    swapoff -a && swapon -a

注意：这样清理有个前提条件，空闲的内存必须比已经使用的swap空间大


----------

Cache：高速缓存，是位于CPU与主内存间的一种容量较小但速度很高的存储器。

由于CPU的速度远高于主内存，CPU直接从内存中存取数据要等待一定时间周期，Cache中保存着CPU刚用过或循环使用的一部分数据，当CPU再次使用该部分数据时可从Cache中直接调用,这样就减少了CPU的等待时间,提高了系统的效率。

Cache又分为一级Cache(L1 Cache)和二级Cache(L2 Cache)，L1Cache集成在CPU内部，L2 Cache早期一般是焊在主板上,现在也都集成在CPU内部，常见的容量有256KB或512KB L2 Cache。

Buffer：缓冲区，一个用于存储速度不同步的设备或优先级不同的设备之间传输数据的区域。通过缓冲区，可以使进程之间的相互等待变少，从而使从速度慢的设备读入数据时，速度快的设备的操作进程不发生间断。
 
在Free命令中显示的buffer和cache，它们都是占用内存：

buffer : 作为buffer cache的内存，是块设备的读写缓冲区，更靠近存储设备，或者直接就是disk的缓冲区。

cache: 作为page cache的内存, 文件系统的cache，是memory的缓冲区

如果 cache 的值很大，说明cache住的文件数很多。如果频繁访问到的文件都能被cache住，那么磁盘的读IO 必会非常小。

cache清理：

    sync; sync; sync;&& echo3 >/proc/sys/vm/drop_caches
    
    sleep 2
    
    echo 0>/proc/sys/vm/drop_caches


To free pagecache：

    echo 1>/proc/sys/vm/drop_caches


To free dentries and inodes：

    echo 2>/proc/sys/vm/drop_caches


To free pagecache,dentries and inodes：

    echo 3>/proc/sys/vm/drop_caches

`/proc/sys/vm/drop_caches` 的值默认为0（所以我们清空后，还再恢复它的值为0）
