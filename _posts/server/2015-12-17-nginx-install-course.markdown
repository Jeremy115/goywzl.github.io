---
layout: article
title:  "Nginx安装教程"
date:   2015-12-17 09:20:00 +0800
categories: server
---


nginx可以使用各平台的默认包来安装，本文主要是介绍使用源码编译安装，包括具体的编译参数信息。

正式开始前，编译环境gcc g++ 开发库之类的需要提前装好，这里默认你已经装好。

# 安装前准备 #

## ububtu平台编译环境可以使用以下指令 ##

{% highlight bash %}
{% raw %}

apt-get install openssl
apt-get install libssl-dev
apt-get install build-essential
apt-get install libtool

{% endraw %}
{% endhighlight %}

## centos平台编译环境使用如下指令 ##

安装make：

{% highlight bash %}
{% raw %}

yum -y install gcc automake autoconf libtool make

{% endraw %}
{% endhighlight %}

安装g++:

{% highlight bash %}
{% raw %}

yum install gcc gcc-c++
yum -y install openssl openssl-devel

{% endraw %}
{% endhighlight %}

# 下面正式开始安装Nginx #

在安装nginx之前，需要这两个pcre, zlib的依赖，前者为了重写
rewrite，后者为了gzip压缩。

## 1.选定源码目录 ##

可以是任何目录，本文选定的是`/usr/local/download`

{% highlight bash %}
{% raw %}

mkdir /usr/local/download
cd /usr/local/download

{% endraw %}
{% endhighlight %}

## 2.安装PCRE库 ##

[ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/ "ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/")下载最新的 PCRE 源码包，使用下面命令下载编译和安装 PCRE 包：

{% highlight bash %}
{% raw %}

cd/usr/local/src
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.34.tar.gz
tar-zxvf pcre-8.34.tar.gz
cd pcre-8.34
./configure
make
make install

{% endraw %}
{% endhighlight %}

## 3.安装zlib库 ##

[http://zlib.net](http://zlib.net "http://zlib.net")下载最新的 zlib 源码包，使用下面命令下载编译和安装 zlib包：

{% highlight bash %}
{% raw %}

cd/usr/local/src
wget http://zlib.net/zlib-1.2.8.tar.gz
tar-zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure
make
make install

{% endraw %}
{% endhighlight %}

## 4.安装ssl（某些vps默认没装ssl) ##

[http://www.openssl.org/source/](http://www.openssl.org/source/ "http://www.openssl.org/source/")下载最新的openssl包

{% highlight bash %}
{% raw %}

cd/usr/local/src
wget http://www.openssl.org/source/openssl-1.0.1c.tar.gz
tar -zxvf openssl-1.0.1c.tar.gz

{% endraw %}
{% endhighlight %}

## 5.安装nginx ##

Nginx 一般有两个版本，分别是稳定版和开发版，您可以根据您的目的来选择这两个版本的其中一个，下面是把 Nginx 安装到 /usr/local/nginx 目录下的详细步骤：

{% highlight bash %}
{% raw %}

cd/usr/local/src
wget http://nginx.org/download/nginx-1.4.2.tar.gz
tar-zxvf nginx-1.4.2.tar.gz
cd nginx-1.4.2
./configure–sbin-path=/usr/local/nginx/nginx \–conf-path=/usr/local/nginx/nginx.conf \–pid-path=/usr/local/nginx/nginx.pid \–with-http_ssl_module \–with-pcre=/usr/local/src/pcre-8.34 \–with-zlib=/usr/local/src/zlib-1.2.8 \–with-openssl=/usr/local/src/openssl-1.0.1c
make
make install

{% endraw %}
{% endhighlight %}

	–with-pcre=/usr/src/pcre-8.34 指的是pcre-8.34 的源码路径。
	–with-zlib=/usr/src/zlib-1.2.7 指的是zlib-1.2.7 的源码路径。

## 6.启动 ##

确保系统的 80 端口没被其他程序占用，运行/usr/local/nginx/nginx 命令来启动 Nginx，

{% highlight bash %}
{% raw %}

netstat-ano|grep80

{% endraw %}
{% endhighlight %}

如果查不到结果后执行，有结果则忽略此步骤（ubuntu下必须用sudo启动，不然只能在前台运行）

{% highlight bash %}
{% raw %}

/usr/local/nginx/nginx

{% endraw %}
{% endhighlight %}

打开浏览器访问此机器的 IP，如果浏览器出现 Welcome to nginx! 则表示 Nginx 已经安装并运行成功。

*开机自启动的方法，请看以下链接：*

[Nginx-start-script](http://www.goywzl.com/server/nginx-start-script/ "Nginx-start-script")


# 说一些nginx编译选项 #

make是用来编译的，它从Makefile中读取指令，然后编译。

make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。

configure命令是用来检测你的安装平台的目标特征的。它定义了系统的各个方面，包括nginx的被允许使用的连接处理的方法，比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本，执行结束时，它会创建一个Makefile文件。

## nginx的configure命令支持以下参数： ##

	–prefix=path    定义一个目录，存放服务器上的文件 ，也就是nginx的安装目录。默认使用 /usr/localinx。
	–sbin-path=path 设置nginx的可执行文件的路径，默认为  prefix/sbininx.
	–conf-path=path  设置在nginx.conf配置文件的路径。nginx允许使用不同的配置文件启动，通过命令行中的-c选项。默认为prefix/confinx.conf.
	–pid-path=path  设置nginx.pid文件，将存储的主进程的进程号。安装完成后，可以随时改变的文件名 ， 在nginx.conf配置文件中使用 PID指令。默认情况下，文件名 为prefix/logsinx.pid.
	–error-log-path=path 设置主错误，警告，和诊断文件的名称。安装完成后，可以随时改变的文件名 ，在nginx.conf配置文件中 使用 的error_log指令。默认情况下，文件名 为prefix/logs/error.log.
	–http-log-path=path  设置主请求的HTTP服务器的日志文件的名称。安装完成后，可以随时改变的文件名 ，在nginx.conf配置文件中 使用 的access_log指令。默认情况下，文件名 为prefix/logs/access.log.
	–user=name  设置nginx工作进程的用户。安装完成后，可以随时更改的名称在nginx.conf配置文件中 使用的 user指令。默认的用户名是nobody。
	–group=name  设置nginx工作进程的用户组。安装完成后，可以随时更改的名称在nginx.conf配置文件中 使用的 user指令。默认的为非特权用户。
	–with-select_module –without-select_module 启用或禁用构建一个模块来允许服务器使用select()方法。该模块将自动建立，如果平台不支持的kqueue，epoll，rtsig或/dev/poll。
	–with-poll_module –without-poll_module 启用或禁用构建一个模块来允许服务器使用poll()方法。该模块将自动建立，如果平台不支持的kqueue，epoll，rtsig或/dev/poll。
	–without-http_gzip_module — 不编译压缩的HTTP服务器的响应模块。编译并运行此模块需要zlib库。
	–without-http_rewrite_module  不编译重写模块。编译并运行此模块需要PCRE库支持。
	–without-http_proxy_module — 不编译http_proxy模块。
	–with-http_ssl_module — 使用https协议模块。默认情况下，该模块没有被构建。建立并运行此模块的OpenSSL库是必需的。
	–with-pcre=path — 设置PCRE库的源码路径。PCRE库的源码（版本4.4 – 8.30）需要从PCRE网站下载并解压。其余的工作是Nginx的./ configure和make来完成。正则表达式使用在location指令和 ngx_http_rewrite_module 模块中。
	–with-pcre-jit —编译PCRE包含“just-in-time compilation”（1.1.12中， pcre_jit指令）。
	–with-zlib=path —设置的zlib库的源码路径。要下载从 zlib（版本1.1.3 – 1.2.5）的并解压。其余的工作是Nginx的./ configure和make完成。ngx_http_gzip_module模块需要使用zlib 。
	–with-cc-opt=parameters — 设置额外的参数将被添加到CFLAGS变量。例如,当你在FreeBSD上使用PCRE库时需要使用:–with-cc-opt=”-I /usr/local/include。.如需要需要增加 select()支持的文件数量:–with-cc-opt=”-D FD_SETSIZE=2048″.
	–with-ld-opt=parameters —设置附加的参数，将用于在链接期间。例如，当在FreeBSD下使用该系统的PCRE库,应指定:–with-ld-opt=”-L /usr/localb”