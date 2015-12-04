---
layout: article
title:  "nginx-start-script"
date:   2015-12-04 09:20:00 +0800
categories: server
---


源码包搭建nginx的人都知道，它没有开机启动，需要自己编辑启动脚本，放到指定的位置。

我这里做了一下笔记。只需要四步进行曲即可完成。


----------


第一步，创建启动文件

{% highlight bash %}
{% raw %}

vim nginx

#!/bin/sh
# description: nginx is a World Wide Web server. It is used to serve
set -e
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DESC="nginx daemon"
NAME=nginx
DAEMON=/usr/local/nginx/sbin/$NAME
SCRIPTNAME=/etc/init.d/$NAME

# if the daemon file is not found,terminate the script
test -x $DAEMON || exit 0

d_start() {
        $DAEMON || echo -n "already running"
}

d_stop() {
        $DAEMON -s quit || echo -n "not runnin"
}

d_reload() {
        $DAEMON -s reload || echo -n "could not reload"
}

case "$1" in
start)
        echo -n "Starting $DESC: $NAME"
                d_start
                echo "."
;;
stop)
        echo -n "Stopping $DESC: $NAME"
                d_stop
                echo "."
;;
reload)
        echo -n "Reloading $DESC configuration..."
                d_reload
                echo "reloaded."
;;
restart)
        echo -n "Restarting $DESC: $NAME"
                d_stop
#Sleepp for two seconds before starting again,this should give the
#Nginx daemon some time to perform a graceful stop
                sleep 2
                d_start
                echo "."
;;
*)
        echo "Usage: $SCRIPTNAME {start|stop|restart|reload}" >&2
;;
esac

exit 0

{% endraw %}
{% endhighlight %}

第二步，赋予它完美的执行权限

{% highlight bash %}
{% raw %}

chmod +x nginx

{% endraw %}
{% endhighlight %}

第三步，放到它所在的岗位：`/etc/init.d`目录下


{% highlight bash %}
{% raw %}

mv nginx /etc/init.d/

{% endraw %}
{% endhighlight %}

就可以通过一下命令进行nginx操作了

{% highlight bash %}
{% raw %}

/etc/init.d/nginx start 命令启动nginx

/etc/init.d/nginx stop 命令停止nginx

/etc/init.d/nginx restart 命令重启nginx

{% endraw %}
{% endhighlight %}

第四步，让它为你开机自启动

需要chkconfig这个牛X的命令

{% highlight bash %}
{% raw %}

chkconfig --add ningx

chkconfig --level nginx 2345 on

ubuntu下执行update-rc.d -f nginx defaults

{% endraw %}
{% endhighlight %}

现在就OK了，开机自启动完成，有问题，找我，有求必应！
