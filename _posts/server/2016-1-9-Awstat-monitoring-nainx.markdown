---
layout: article
title:  "Awstat monitoring nginx"
date:   2016-01-09 09:20:00 +0800
categories: server
---

# 目地：内网Awstats监控外网nginx日志 #

    时间：2014-11-3
    环境：日志Nginx一台，内网Awstats一台，代理Nginx一台，DNSPod解析网站
    软件：Awstats，autossh

# 首先，介绍一下Awstats #

Awstats是一个免费非常简洁而且强大有个性的统计工具。它可以统计您站点的如下信息：

    1.访问量，访问次数，页面浏览量，点击数，数据流量等
    2.精确到每月、每日、每小时的数据
    3.访问者国家
    4.访问者IP
    5.Robots/Spiders的统计
    6.纺客持续时间
    7.对不同Files type 的统计信息
    8.Pages-URL的统计
    9.访客操作系统浏览器等信息
    10.其它信息（搜索关键字等等）

# 然后，就是搭建Awstats了 #

安装nginx，更改配置文件，在location中index后面加上`awstats.xxx.html`,其他的配置，具体不做解释。

{% highlight bash %}
{% raw %}

wget http://prdownloads.sourceforge.net/awstats/awstats-7.0.tar.gz
tar xzf awstats-7.0.tar.gz
mv awstats-7.0 /usr/local/awstats
cd /usr/local/awstats/tools
mkdir /etc/awstats /var/lib/awstats

{% endraw %}
{% endhighlight %}

既然Awstats安装完成了，就可以说一下配置了

执行`awstats_configure.pl`脚本进行配置。

{% highlight bash %}
{% raw %}

perl awstats_configure.pl

{% endraw %}
{% endhighlight %}

配置过程如下：

{% highlight bash %}
{% raw %}

Config file path ('none' to skip web server setup):
none
—–> Need to create a new config file ?
Do you want me to build a new AWStats config/profile
file (required if first install) [y/N] ? y
—–> Define config file name to create
What is the name of your web site or profile analysis ?
Example: nginx
Example: demo
Your web site, virtual server or profile name:
> ipaloma
—–> Define config file path
In which directory do you plan to store your config file(s) ?
Default: /etc/awstats
Directory path to store config file(s) (Enter for default):   回车

{% endraw %}
{% endhighlight %}

1.接着需要编辑配置文件awstats.nginx.conf。

{% highlight bash %}
{% raw %}

vi /etc/awstats/awstats.nginx.conf


{% endraw %}
{% endhighlight %}


2.只需要定义日志的路径，如：

{% highlight bash %}
{% raw %}

LogFile="/Nginx_log_backup/nginx.log"

{% endraw %}
{% endhighlight %}


3.复制css和icon目录到网站根目录。

{% highlight bash %}
{% raw %}

cp -R /usr/local/awstats/wwwroot/css /home/www/default
cp -R /usr/local/awstats/wwwroot/icon /home/www/default

{% endraw %}
{% endhighlight %}


4.手动执行命令更新日志统计数据库及生成静态文件到目录/usr/local/nginx/html:

{% highlight bash %}
{% raw %}

/usr/local/awstats/tools/awstats_buildstaticpages.pl -config=ipaloma -update -lang=cn -awstatsprog=/usr/local/awstats/wwwroot/cgi-bin/awstats.pl -dir=/usr/local/nginx/html

{% endraw %}
{% endhighlight %}


5．之后你就可以使用http://ip访问日志统计页面。

6.添加IP地址解析插件

把qqhostinfo.pm，QQWry.dat，qqwry.pl

三个文件放到 `/usr/local/awstats/wwwroot/cgi-bin/plugins`下
修改`qqwry.pl`

{% highlight bash %}
{% raw %}

#my $ipfile="./QQWry.Dat";  改为 my $ipfile="${DIR}/plugins/qqwry.dat"

{% endraw %}
{% endhighlight %}

在配置文件`awstats.ipaloma.conf`上添加一行

{% highlight bash %}
{% raw %}

LoadPlugin="qqhostinfo"


{% endraw %}
{% endhighlight %}


至此，awstats配置完成

*注：*
`awstats_buildstaticpages.pl`脚本使用说明：
语法：

{% highlight bash %}
{% raw %}

awstats_buildstaticpages.pl (awstats_options) [awstatsbuildstaticpages_options]

{% endraw %}
{% endhighlight %}


awstats_options可选参数为：

    -config=configvalue：定义配置文件，如www.centos.bz，就会搜索   
    /etc/awstats/awstats.www.centos.bz.conf文件。
    -update ：该选项定义生成静态页面之前先更新数据库。
    -lang ：统计页面的语言，如-lang=cn，语言为中文。
    awstatsbuildstaticpages_options可选参数为：
    -awstatsprog=pathtoawstatspl ：定义awstats.pl路径。
    -dir ：定义输出静态页面的目录。