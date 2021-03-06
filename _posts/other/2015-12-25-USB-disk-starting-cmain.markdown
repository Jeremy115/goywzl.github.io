---
layout: article
title:  "U盘启动出现starting cmain"
date:   2015-12-25 09:20:00 +0800
categories: other
---

U盘启动时提示“starting cmain”一般是这样子的：

这种情况，一般是制作好了PE启动U盘之后，启动不了才会这样，一般正常情况的话，这一句英文是一闪而过直接进入PE菜单的。

到这里卡住了，说明：已经从U盘启动了，可是加载不了U盘的PE文件。

当然，很多种情况都会导致这样的问题，网上基本上都是说“硬盘没有格式化”，醉了做了启动盘格式化了还怎么启动，下面介绍几个解决办法。

----------

我总结了一下，这种问题不外乎三种解决方案：

**方法一：换USB插口**

这种方法最简单，很多情况下换一个插口就可以了，建议不要使用3.0插口。

**方法二：修改BIOS**

如果你的本本买来的时候是自带win8系统的，那么可能需要在bios里简单设置一下

开机，看见本本LOGO 的时候按F2进入bios，如图找到“BOOT”选项，不同的电脑位置会有所不同，

默认的会是UEFI，改成legacy。不改的话U盘启动会进不去

如果以上两种方法都不行的话那么可以在其他电脑上尝试一下，如果都不行那么可以尝试下下面的方法：

**方法三：使用ZIP-FAT32模式** 

制作启动工具的时候，选择ZIP-FAT32模式 

默认是hdd-fat32的，那么点击下拉框，选择ZIP-FAT32模式 ，重新制作USB启动盘即可