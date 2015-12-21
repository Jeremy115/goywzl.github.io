---
layout: article
title:  "zabbix升级与迁移"
date:   2015-12-20 09:20:00 +0800
categories: server
---

公司有一台zabbix，然后领导突然说把zabbix放到另外一台上去，这才有心仔细查看一下我们公司的zabbix。

发现版本很低，并且迁移还没有做过，所以网上查找各种文档，这里做了很简单的笔记。


# zabbix迁移 #

网上查找文档，一阵抓狂之后，发现如此简单。我也就简单的写了。

1.首先需要搭建一个新的zabbix，很简单的

2.搭建版本基本相同mysql数据库

3.进行mysql备份，并恢复到新的mysql上即可



# zabbix升级版本 #


1.下载软件，上传到zabbix server服务器解压，然后进入database目录下的mysql目录查找有无path.sql;

2.关闭zabbix_server服务

{% highlight bash %}
{% raw %}

# /etc/init.d/zabbix_server stop
# ps -ef |grep zabbix_server

{% endraw %}
{% endhighlight %}

3.关闭数据库

{% highlight bash %}
{% raw %}

# /etc/init.d/mysqld stop

{% endraw %}
{% endhighlight %}

4.备份

备份数据库：将数据文件的目录完整拷贝一份；或者使用命令喽

备份配置文件：通常是/usr/local/zabbix 、还有php网站源码

{% highlight bash %}
{% raw %}

#cd /usr/local/
# \cp -rfp zabbix/ zabbix_bak
# cd /var/www/
# \cp -rfp zabbix/ zabbix_bak

{% endraw %}
{% endhighlight %}

5.升级程序

进入解压的新程序目录下执行

{% highlight bash %}
{% raw %}

#./configure –prefix=/usr/local/zabbix
–enable-server –enable-agent –with-mysql –with-libcurl
–with-libxml2–with-ssh2

{% endraw %}
{% endhighlight %}

    1).检查zabbix_server.conf
    2).升级php文件：拷贝新解压目录下的文件到以前php文件目录

{% highlight bash %}
{% raw %}

#cd
zabbix-2.4.3/frontends/php/
# \cp -rfp * /var/www/zabbix/

{% endraw %}
{% endhighlight %}

6.验证程序：

    1).开启mysql数据库
    2).开启zabbix_server进程,确认进程是否正常
    3).执行WEB页面的更新
    3).字体处理

{% highlight bash %}
{% raw %}

#cp /var/www/zabbix_bak/fonts/DejaVuSans.ttf
/var/www/zabbix/fonts/

{% endraw %}
{% endhighlight %}

7.开启中文支持

{% highlight bash %}
{% raw %}

#vi /var/www/zabbix/include/locales.inc.php

{% endraw %}
{% endhighlight %}

将这个‘zh_CN’ => array(‘name’ => _(‘Chinese (zh_CN)’), ‘display’ => true)