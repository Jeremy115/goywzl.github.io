---
layout: article
title:  "linux memtester tools"
date:   2016-1-18 09:20:00 +0800
categories: linux
---

昨天领导让我测试一下内存压力，我有点蒙，上网查找了一下，找到了这个memtester压力测试工具，进行一下收集。

首先进行一下工具的介绍：

Memtester主要是捕获内存错误和一直处于很高或者很低的坏位, 其测试的主要项目有随机值,异或比较,减法,乘法,除法,与或运算等等. 通过给定测试内存的大小和次数, 可以对系统现有的内存进行上面项目的测试。

测试环境：

    服务器：Linux服务器一台
    内存：4G
    软件工具：memtester-4.3.0.tar.gz

现在进行安装介绍

首先：知道软件工具下载地址：`http://pyropus.ca/software/memtester/`

然后：找到想要的版本进行安装，安装步骤如下

{% highlight bash %}
{% raw %}

tar -zxvf memtester-4.3.0.tar.gz
cd memtester-4.3.0
make
make install

{% endraw %}
{% endhighlight %}

如此简单，不报错，这就安装完成了

然后查看以下这个命令怎么用

{% highlight bash %}
{% raw %}

root@localhost:~# memtester -h
memtester version 4.3.0 (64-bit)
Copyright (C) 2001-2012 Charles Cazabon.
Licensed under the GNU General Public License version 2 (only).
pagesize is 4096
pagesizemask is 0xfffffffffffff000
memtester: invalid option — ‘h’
Usage: memtester [-p physaddrbase [-d device]] <mem>[B|K|M|G] [loops]

{% endraw %}
{% endhighlight %}

看到上面的介绍，这里的使用方法很简单：`memtest 2G 1`

上面的意思是，测试2G内存，测试次数为1次。下面是测试结果

{% highlight bash %}
{% raw %}

root@localhost:~# memtester 2G 1
memtester version 4.3.0 (64-bit)
Copyright (C) 2001-2012 Charles Cazabon.
Licensed under the GNU General Public License version 2 (only).
pagesize is 4096
pagesizemask is 0xfffffffffffff000
want 2048MB (2147483648 bytes)
got  2048MB (2147483648 bytes), trying mlock …locked.
Loop 1/1:
  Stuck Address       : ok
  Random Value        : ok
  Compare XOR         : ok
  Compare SUB         : ok
  Compare MUL         : ok
  Compare DIV         : ok
  Compare OR          : ok
  Compare AND         : ok
  Sequential Increment: ok
  Solid Bits          : ok
  Block Sequential    : ok
  Checkerboard        : ok
  Bit Spread          : ok
  Bit Flip            : ok
  Walking Ones        : ok
  Walking Zeroes      : ok
  8-bit Writes        : ok
  16-bit Writes       : ok
Done.

{% endraw %}
{% endhighlight %}
 
上面是测试的内容，当然，时间很长，你要是不想测试上面中的某一项的话，你就需要更改配置文件了

打开memtester.c配置文件，找到`struct test tests`这一行，下面的就是要测试的内容，如果不想要测试，直接注释掉那一项。

{% highlight bash %}
{% raw %}

struct test tests[] = {
    { “Random Value”, test_random_value },
    { “Compare XOR”, test_xor_comparison },
    { “Compare SUB”, test_sub_comparison },
    { “Compare MUL”, test_mul_comparison },
    { “Compare DIV”,test_div_comparison },
    { “Compare OR”, test_or_comparison },
    { “Compare AND”, test_and_comparison },
    { “Sequential Increment”, test_seqinc_comparison },
    { “Solid Bits”, test_solidbits_comparison },
    { “Block Sequential”, test_blockseq_comparison },
    { “Checkerboard”, test_checkerboard_comparison },
    { “Bit Spread”, test_bitspread_comparison },
    { “Bit Flip”, test_bitflip_comparison },
    { “Walking Ones”, test_walkbits1_comparison },
    { “Walking Zeroes”, test_walkbits0_comparison },
#ifdef TEST_NARROW_WRITES
    { “8-bit Writes”, test_8bit_wide_random },
    { “16-bit Writes”, test_16bit_wide_random },
#endif
    { NULL, NULL }
};

{% endraw %}
{% endhighlight %}
 
到这里，基本上就介绍完了，如果你有更好的见解，求留言。