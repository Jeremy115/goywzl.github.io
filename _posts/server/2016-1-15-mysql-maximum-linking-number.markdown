---
layout: article
title:  "Mysql maximum linking number"
date:   2016-01-15 09:20:00 +0800
categories: server
---

mysql安装好后，有很多参数需要调优。几乎所有的涉及到调优的内容我们都都可以在my.cnf文件中设置完成。而mysql的连接数也是较为重要的调优参数之一。mysql 的默认最大连接数为100， 对于大负载量的并发需求是不够的，这时你可以修改mysql的最大连接数。

#### 1、查看当前mysql的最大连接数的方法： ####

{% highlight bash %}
{% raw %}

mysqladmin -uroot -ppassword variables | grep max_connections
或者
mysql> SHOW GLOBAL VARIABLES WHERE Variable_name='max_connections';
或者
mysql> SHOW GLOBAL like '%conn%';

{% endraw %}
{% endhighlight %}

#### 2、修改方法有如下几种 ####

{% highlight bash %}
{% raw %}

mysql> SET GLOBAL max_connections=1000;

{% endraw %}
{% endhighlight %}

修改后会立即生效，不需要重启mysql服务，但是重启后会失效。

修改/etc/my.cnf,
在[mysqld] 下面添加：

{% highlight bash %}
{% raw %}

max_connections=1000

{% endraw %}
{% endhighlight %}

修改后需要重启mysql服务才会重效。


----------


以上两种方法是较为常见的方法，不过也有变态人士经常使用一些不常用的方法。如下：

#### 1、修改/usr/bin/mysqld_safe ####

{% highlight bash %}
{% raw %}

if test -z "$args"
then
$NOHUP_NICENESS $ledir/$MYSQLD $defaults --basedir=$MY_BASEDIR_VERSION --datadir=$DATADIR $USER_OPTION --pid-file=$pid_file --skip-external-locking >> $err_log 2>&1
else
eval "$NOHUP_NICENESS $ledir/$MYSQLD $defaults --basedir=$MY_BASEDIR_VERSION --datadir=$DATADIR $USER_OPTION --pid-file=$pid_file --skip-external-locking $args >> $err_log 2>&1"
 
{% endraw %}
{% endhighlight %}


修改为：

 

{% highlight bash %}
{% raw %}

if test -z "$args"
then
$NOHUP_NICENESS $ledir/$MYSQLD $defaults --basedir=$MY_BASEDIR_VERSION --datadir=$DATADIR $USER_OPTION --pid-file=$pid_file --skip-external-locking 　　-O max_connections=1000 >> $err_log 2>&1
else
eval "$NOHUP_NICENESS $ledir/$MYSQLD $defaults --basedir=$MY_BASEDIR_VERSION --datadir=$DATADIR $USER_OPTION --pid-file=$pid_file --skip-external-locking $args -O max_connections=1000 >> $err_log 2>&1"
 

{% endraw %}
{% endhighlight %}

修改后重启mysql服务后有效。

#### 2、修改原代码: ####

解开MySQL的原代码，进入里面的sql目录修改mysqld.cc找到下面一行：

 

{% highlight bash %}
{% raw %}

{"max_connections", OPT_MAX_CONNECTIONS,
　　"The number of simultaneous clients allowed.", (gptr*) &max_connections,
　　(gptr*) &max_connections, 0, GET_ULONG, REQUIRED_ARG, 100, 1, 16384, 0, 1,
　　0},
 
{% endraw %}
{% endhighlight %}


把它改为：

 

{% highlight bash %}
{% raw %}

{"max_connections", OPT_MAX_CONNECTIONS,
　　"The number of simultaneous clients allowed.", (gptr*) &max_connections,
　　(gptr*) &max_connections, 0, GET_ULONG, REQUIRED_ARG, 1500, 1, 16384, 0, 1,
　　0},
 
{% endraw %}
{% endhighlight %}


存盘退出，然后./configure ;make;make install可以获得同样的效果。

