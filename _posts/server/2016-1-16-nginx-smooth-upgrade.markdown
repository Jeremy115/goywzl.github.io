---
layout: article
title:  "Nginx smooth upgrade"
date:   2016-1-16 09:20:00 +0800
categories: server
---

nginx本身支持平滑升级，这已不是什么新鲜事。不过在对现上环境操作时我们总是慎之又慎，生错出现一丁点的问题。

公司的一个web入口运行的nginx是N年前的旧版本。一直没有升级，很长一段时间打开网站时，偶尔会出现一下 502 的毛病（F5刷新会发现又正常了，怀疑是nginx早期版本的bug），同时又出于安全方面的考量。决定将其升级为最新的tengine 。

### 1、下载升级所用到的源码包文件 ###

*注：你下载的时候要找最新的版本*

{% highlight bash %}
{% raw %}

wget http://tengine.taobao.org/download/tengine-1.4.4.tar.gz
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.32.tar.gz
wget http://www.canonware.com/download/jemalloc/jemalloc-3.3.1.tar.bz2

{% endraw %}
{% endhighlight %}

### 2、备份旧的nginx程序 ###

如果升级失败，可以还原，这一步最好做好

{% highlight bash %}
{% raw %}

cp /App/nginx /App/nginx20130409

{% endraw %}
{% endhighlight %}

### 3、解压源码包 ###

{% highlight bash %}
{% raw %}

tar zxvf tengine-1.4.4.tar.gz
tar zxvf pcre-8.32.tar.gz
tar jvxf jemalloc-3.3.1.tar.bz2

{% endraw %}
{% endhighlight %}

### 4、修改tengine的源文件 ###

出去安全考虑，要隐藏tengine的版本 。要改动的两个源文件同nginx相同，分别是`src/core/nginx.h`和`src/http/ngx_http_header_filter_module.c`两个文件 。

### 5、编译 ###

此处我编译时直接编译到了原nginx所在的目录—— /App/nginx ，这里经测试是完全可行的。旧的nginx程序会被更名为nginx.old ，而conf配置文件不会被改动，仍和原版本保持一致，具体操作如下：

{% highlight bash %}
{% raw %}

cd jemalloc-3.3.1
./configure
make && make install
echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
/sbin/ldconfig
cd pcre-8.32
./configure --prefix=/usr/local/pcre
make && make install
cd tengine-1.4.4
./configure --prefix=/App/nginx/ --with-jemalloc=/usr/local/src/jemalloc-3.3.1 --with-pcre=/usr/local/src/pcre-8.32
make
make install
/App/nginx/sbin/nginx -t

{% endraw %}
{% endhighlight %}

现在已经把旧的文件覆盖，则可以进行Nginx服务的操作了———— 平滑升级。

### 6、平滑升级 ###

首先，先向nginx程序发送一个USR2的信号

{% highlight bash %}
{% raw %}

kill -USR2 `cat /usr/local/webserver/nginx/nginx.pid`

{% endraw %}
{% endhighlight %}

运行完该步后，会发现此时的nginx.pid文件会变成nginx.pid.oldbin 。而通过ps 查看时，会发现有两个nginx master主进程。一个是刚刚启动的。接下来让老的nginx 从容而有颜面的退役：

{% highlight bash %}
{% raw %}

kill -QUIT `cat /App/nginx/logs/nginx.pid.oldbin`

{% endraw %}
{% endhighlight %}

总结来说：上面两个操作就是：让nginx 重新用目前的sbin目录下的nginx再运行，同时保持旧的存在。接着向旧版本的nginx发送一个quit信号，使其不再接受新的请求（新的请求由新的nginx处理），同时对未完成的请求，完成后退出。

● Nginx 的信号控制

    TERM, INT 快速关闭
    QUIT 从容关闭
    HUP 平滑重启，重新加载配置文件
    USR1 重新打开日志文件，在切割日志时用途较大
    USR2 平滑升级可执行程序
    WINCH 从容关闭工作进程