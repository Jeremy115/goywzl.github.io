---
layout: article
title:  "Centos的主机名"
date:   2015-11-21 15:34:30 +0800
categories: linux
---


时间：2014年
环境：centos系统


Linux 更改主机名可不像windows那样的简单。

Linux 安装好后，其默认的主机名是 localhost（当然，如果你安装的时候设置了，那就是别的了）。


----------

但是你要是安装好了之后想要修改 Linux 主机名，你就需要以下3步。

再做之前，先用hostname查看一下自己的主机名

1.使用 hostname 修改当前主机名。
 
{% highlight bash %}
{% raw %}

hostname new-hostname

{% endraw %}
{% endhighlight %}

做完了这一步，主机名也就变了，不过重启后，还是会变回到之前的主机名。

2.修改 `/etc/sysconfig/network ` 配置文件，以便下次重启的时，使用新的主机名。

打开 `/etc/sysconfig/network` 文件，修改

{% highlight bash %}
{% raw %}

HOSTNAME=new-hostname.domainname

{% endraw %}
{% endhighlight %}

修改后的 `/etc/sysconfig/network` 文件如下：

{% highlight bash %}
{% raw %}

NETWORKING=yes
HOSTNAME=new-hostname.localdomain

{% endraw %}
{% endhighlight %}

3.修改本机的域名解析文件 `/etc/hosts` ，使得本机的应用程序能够解析新的主机名。

编辑文件： `/etc/hosts`

修改： `xxx.xxx.xxx.xxx new-hostname.domainname new-hostname`（这里的xxx代表本机的网络地址，也可以是环回地址127.0.0.1）

修改后的 /etc/hosts 文件如下：

{% highlight bash %}
{% raw %}

127.0.0.1  localhost.localdomain localhost
127.0.0.1  new-hostname.localdomain new-hostname

{% endraw %}
{% endhighlight %}

之后，你就可以reboot重启一下服务器了，更改后不成功的话，那我也没办法了。
