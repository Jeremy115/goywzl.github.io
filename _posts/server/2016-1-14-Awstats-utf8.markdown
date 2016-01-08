---
layout: article
title:  "Awstats中文关键字乱码"
date:   2016-01-14 09:20:00 +0800
categories: server
---

对站点进行静态全面性的访问分析，awstats应该算是其中的佼佼者。在awstats对日志进行分析结果查看时，在查看搜索的关键字和短语排名时，发现部分中文关键字显示乱码。所以急忙求助谷歌大神。下面将解决方法分享下：

1、修改awstats的配置文件 

在使用centos/redhat系统的rpm包安装的awstats的配置路径为/etc/awstats下。打开所要统计网站的配置文件awstats.xxxxx.conf,将下面一行的#去掉：

{% highlight bash %}
{% raw %}

#LoadPlugin="decodeutfkeys"

{% endraw %}
{% endhighlight %}

改完了，刷新下页面。一般都能解决。而如果刷新完页面有下面的提示：

{% highlight bash %}
{% raw %}

Error: Plugin load for plugin 'decodeutfkeys' failed with return code: Error: Can't locate URI/Escape.pm in @INC (@INC contains: /etc/perl /usr/local/lib/perl/5.10.1 /usr/local/share/perl/5.10.1 /usr/lib/perl5 /usr/share/perl5 /usr/lib/perl/5.10 /usr/share/perl/5.10 /usr/local/lib/site_perl . /usr/share/awstats/lib /usr/share/awstats/plugins) at (eval 3) line 1.
Setup ('/etc/awstats/awstats.bbsnuaa.conf' file, web server or permissions) may be wrong.
Check config file, permissions and AWStats documentation (in 'docs' directory).

{% endraw %}
{% endhighlight %}

就是少了`URI/Escape`模块。而该模块可通过两种方法安装。

2、安装URI/Escape模块

一种是通过perl shell网络安装：

{% highlight bash %}
{% raw %}

root@server ~]# perl -MCPAN -e shell
>install URI::Escape
 
{% endraw %}
{% endhighlight %}


另一种就是通过从CPAN站点上下则URI/Escape模块包，再手动安装：

{% highlight bash %}
{% raw %}

wget http://search.cpan.org/CPAN/authors/id/G/GA/GAAS/URI-1.36.tar.gz
tar zxvf URI-1.36.tar.gz
cd URI-1.36
perl Makefile.PL
make make install

{% endraw %}
{% endhighlight %}

安装完了，通过下面的命令更新下配置文件。

{% highlight bash %}
{% raw %}

/var/www/awstats/awstats.pl -config=XXXXX -update
 
{% endraw %}
{% endhighlight %}


*注*：rpm包方式安装的awstats.pl在上面写的路径。如果是源码包方式的安装。将路径改成相应的路径即可。