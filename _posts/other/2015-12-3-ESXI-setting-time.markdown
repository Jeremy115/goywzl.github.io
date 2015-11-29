---
layout: article
title:  "ESXI setting time"
date:   2015-12-03 09:20:00 +0800
categories: home other
---

虚拟机服务器环境：ESXI 5.5

虚拟机：windows2008

虚拟机：centos6.5

我遇到过这样的情况。

1.windows2008在设置“internet时间同步”的时候总是提示失败，然后选择重试好几次才能成功。

2.在给虚拟机做快照，或者重启，恢复快照等系列操作的时候，虚拟机时间就会变动。

经过反复的测试研究，终于明白是什么原因导致的了。

因为虚拟机在执行快照，重启，恢复快照操作之后，会自动向ESXI服务器时间同步，（我记得是同步的ESXI主机的硬件时间）不仅如此，虚拟机还会定期向ESXI服务器时间同步。

所以有必要更改一下ESXI服务器的时间。

----------

首先开启ESXI的SSH远程登录功能，我想这个大家都开启了，毕竟都是做运维的。

如果没开启，就只能去机房接上显示器操作了，找到“Troubleshooting Options”选项，进去后选择“Disable ESXI Shell”改成enable即可。

然后远程登录到ESXI

执行`date`查看系统时间

{% highlight bash %}
{% raw %}

~ # date
Tue Mar  3 03:10:28 UTC 2015

{% endraw %}
{% endhighlight %}

执行`hwclock`查看硬件时间

{% highlight bash %}
{% raw %}

~ # hwclock
03:11:09   03/03/2015   UTC

{% endraw %}
{% endhighlight %}

用 `esxcli system time set –help` 来设置系统时间，得出来的东西，我相信你看得懂

{% highlight bash %}
{% raw %}

~ # esxcli system time set --help
Usage: esxcli system time set [cmd options]

Description: 
  set                   Set the system clock time. Any missing parameters will default to the current time

Cmd options:
  -d|--day=<long>       Day
  -H|--hour=<long>      Hour
  -m|--min=<long>       Minute
  -M|--month=<long>     Month
  -s|--sec=<long>       Second
  -y|--year=<long>      Year

{% endraw %}
{% endhighlight %}

这是我设置的

{% highlight bash %}
{% raw %}

~ # esxcli system time set -m 11
~ # esxcli system time set -H 11

{% endraw %}
{% endhighlight %}

然后可以设置硬件时间了
用 `hwclock –help`来设置硬件时间，得出来的东西，这个你也看得懂的

{% highlight bash %}
{% raw %}

~ # hwclock --help
hwclock     -d --date <date>         Set date
            -t --time <time>         Set time

Valid date format: MM/DD/YYYY
Valid time format: HH:MM:SS

{% endraw %}
{% endhighlight %}

然后设置即可

这样以后windows时间就不会在自己变动了，呼呼，如果你的还变动，那自求多福

