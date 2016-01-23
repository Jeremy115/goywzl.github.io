---
layout: article
title:  "Dos check memory"
date:   2016-1-23 09:20:00 +0800
categories: script
---

同事让我写个批处理，只查看单个程序的使用内存，以及电脑的剩余内存。
网上搜索了一下，写出了简单的脚本。

{% highlight bash %}
{% raw %}
 
@echo off
:start
ping 127.0.0.1 > null
wmic process list brief | find “cmd”
wmic path win32_operatingsystem get freephysicalmemory
wmic process list brief | find “cmd”  >> mem.txt
wmic path win32_operatingsystem get freephysicalmemory >> mem.txt
goto :start
pause
 
{% endraw %}
{% endhighlight %}

*脚本解释：*

通过ping本地的127地址，来进行每秒一次间隔。

通过wmic process … 得到这个单独进程(cmd)的使用内存，打印到屏幕

通过wmic path … 得到电脑剩余内存打印到屏幕

下面通过>> 把结果放入mem.txt文件中

最后通过goto 重复执行

后面的pause是不会让命令行关闭，因为是循环，有无都可