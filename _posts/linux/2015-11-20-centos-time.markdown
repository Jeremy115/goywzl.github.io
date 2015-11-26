---
layout: article
title:  "Centos的时间解说"
date:   2015-11-20 15:34:30 +0800
categories: home linux
---



时间：2014年的文档

环境：centos


----------


**时间**，有多重要我就不用多说了吧，当你配置多台服务器时，经常需要让各个服务器之间的时间保持同步。


----------

**首先**介绍一下查看系统时间的`date`命令

`date`命令可以按照指定格式，显示系统时间，直接输入`date`则显示默认格式时间

如果你想要得到自己想要的格式显示的时间，需要使用"+"开头的字母串指定格式，下面有一些常用的指令，其他的自己摸索吧。

%Y ： 完整的年份 （0000 - 9999）

%y ： 年费的后两位数字 （00 - 99）

%m ： 月份 （01 - 12）

%d ： 日 （01 - 31）

%H ： 小时 （00 - 23）

%M ： 分钟 （00 - 59）

%S ： 秒 （00 - 59）

%s ： 从1970年1月1日00:00:00 UTC 到目前为止的秒数，相当于`time`函数

%w ： 一周中的第几天 （0 - 6）

示例：

查看当前时间

{% highlight bash %}
{% raw %}
date "+%Y-%m-%d %H:%M:%S" 
2015-11-20 13:41:05
{% endraw %}
{% endhighlight %}

如果想要经过运算得出时间，则可以使用*-d*选项，后面的随你自己的需求变化，如月、日

{% highlight bash %}
{% raw %}

date "+%Y-%m-%d %H:%M:%S" -d "-2 year"
2013-11-20 13:45:30

{% endraw %}
{% endhighlight %}

有时候需要获取当前时间距离1970年0时0分0秒所经历的秒数，保存在变量中，这个在写脚本的时候用到过

{% highlight bash %}
{% raw %}

time=`date "+%s"`
1447998474   

{% endraw %}
{% endhighlight %}

当然了，你可以查看系统时间，也可以通过这条命令设置系统时间

使用*-s*选项设置,有很多种格式，我比较喜欢第一种，习惯了，简单写三种，如下

{% highlight bash %}
{% raw %}

date -s "2015-11-18 11:12:00"
date -s "20151118 11:12:00"
date -s "2015/11/18 11:12:00"

{% endraw %}
{% endhighlight %}

其他的还有没说到的，就看你的英文读写能力，使用`date --help `命令或者`man date`了



----------

有些时候，会因为不定因素，导致系统时间“跳”了，这个经常是会发生的，如果时间崩溃了，你配置的很多服务，都会随之崩溃掉，这是硬伤，所以下面也查询了一些资料，用来同步自己的时间。

这有两种情况，一种就是你的服务器可以上网，另一种就是你的内部服务器连接不上网络。


----------


**如果**服务器有外网环境，可以直接与外部的时间服务器同步更新时间

可以采用`rdate`命令更新时间:

命令：

{% highlight bash %}
{% raw %}

rdate -s ntp.sjtu.edu.cn

{% endraw %}
{% endhighlight %}

也可以使用`ntpdate`命令更新时间：

命令：

{% highlight bash %}
{% raw %}

ntpdate ntp.sjtu.edu.cn

{% endraw %}
{% endhighlight %}

关于后面的`ntpdate ntp.sjtu.edu.cn`地址，网上搜索，有好多时间同步地址，我这里选择的是上海交大的ntp服务器地址

可以写个脚本放在/etc/cron.hourly*（这个目录下是每小时执行的文件）*中每小时校正一下时间。
做一个计划任务即可

使用 `rdate --help` 可以看到其它命令参数，这里不做过多的介绍

----------

**如果**是内网环境下，可以自己配置一个时间服务器

以CentOS为例，配置时间服务器的方法如下：

1. 先安装xinetd

{% highlight bash %}
{% raw %}

yum install -y xinetd rdate

{% endraw %}
{% endhighlight %}

2. 修改/etc/xinetd.d/time-stream, 修改：

{% highlight bash %}
{% raw %}

disable = yes  改为  disable = no

{% endraw %}
{% endhighlight %}

3. 启动xinetd

{% highlight bash %}
{% raw %}

service xinetd start

{% endraw %}
{% endhighlight %}

4. 这样其它机器就可以通过rdate 与该机器进行时间同步

{% highlight bash %}
{% raw %}

rdate -s ip

{% endraw %}
{% endhighlight %}
