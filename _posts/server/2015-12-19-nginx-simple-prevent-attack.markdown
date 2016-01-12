---
layout: article
title:  "Nginx一些简单配置"
date:   2015-12-19 09:20:00 +0800
categories: server
---

# 三种简单的防攻击方法 #

## ngx_http_limit_conn_module 可以用来限制单个IP的连接数： ##

**ngx_http_limit_conn_module **模块可以按照定义的键限定每个键值的连接数。特别的，可以设定单一 IP 来源的连接数。

并不是所有的连接都会被模块计数；只有那些正在被处理的请求（这些请求的头信息已被完全读入）所在的连接才会被计数。

	limit_conn_zone
	语法: limit_conn_zone $variable zone=name:size;
	默认值: none
	配置段: http

该指令描述会话状态存储区域。键的状态中保存了当前连接数，键的值可以是特定变量的任何非空值（空值将不会被考虑）。

$variable定义键，zone=name定义区域名称，后面的limit_conn指令会用到的。size定义各个键共享内存空间大小。

如：limit_conn_zone $binary_remote_addr zone=addr:10m;

注释：客户端的IP地址作为键。

注意，这里使用的是$binary_remote_addr变量，而不是$remote_addr变量。

$remote_addr变量的长度为7字节到15字节，而存储状态在32位平台中占用32字节或64字节，在64位平台中占用64字节。

$binary_remote_addr变量的长度是固定的4字节，存储状态在32位平台中占用32字节或64字节，在64位平台中占用64字节。

1M共享空间可以保存3.2万个32位的状态，1.6万个64位的状态。

如果共享内存空间被耗尽，服务器将会对后续所有的请求返回 503 (Service Temporarily Unavailable) 错误。

**limit_zone** 指令和**limit_conn_zone**指令同等意思，已经被弃用，就不再做说明了。

	limit_conn_log_level
	语法：limit_conn_log_level info | notice | warn | error
	默认值：error
	配置段：http, server, location

当达到最大限制连接数后，记录日志的等级。

	limit_conn
	语法：limit_conn zone_name number
	默认值：none
	配置段：http, server, location

指定每个给定键值的最大同时连接数，当超过这个数字时被返回503 (Service Temporarily Unavailable)错误。
如：

{% highlight bash %}
{% raw %}

limit_conn_zone $binary_remote_addr zone=addr:10m;
server {
location /www.ttlsa.com/ {
limit_conn addr 1;
}
}

{% endraw %}
{% endhighlight %}

同一IP同一时间只允许有一个连接。

当多个 limit_conn 指令被配置时，所有的连接数限制都会生效。比如，下面配置不仅会限制单一IP来源的连接数，同时也会限制单一虚拟服务器的总连接数：

{% highlight bash %}
{% raw %}

limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;
server {
limit_conn perip 10;
limit_conn perserver 100;
}

{% endraw %}
{% endhighlight %}

[warning]limit_conn指令可以从上级继承下来。[/warning]

	limit_conn_status
	语法: limit_conn_status code;
	默认值: limit_conn_status 503;
	配置段: http, server, location

该指定在1.3.15版本引入的。指定当超过限制时，返回的状态码。默认是503。

	limit_rate
	语法：limit_rate rate
	默认值：0
	配置段：http, server, location, if in location

对每个连接的速率限制。参数rate的单位是字节/秒，设置为0将关闭限速。 按连接限速而不是按IP限制，因此如果某个客户端同时开启了两个连接，那么客户端的整体速率是这条指令设置值的2倍。

### 实例配置 ###

{% highlight bash %}
{% raw %}

http {
limit_conn_zone $binary_remote_addr zone=limit:10m;
limit_conn_log_level info;
server {
location ^~ /download/ {
limit_conn limit 4;
limit_rate 200k;
alias /data/www.ttlsa.com/download/;
}
}
}

{% endraw %}
{% endhighlight %}

### 注意事项 ###

事务都具有两面性的。ngx_http_limit_conn_module 模块虽说可以解决当前面临的并发问题，但是会引入另外一些问题的。

如前端如果有做LVS或反代，而我们后端启用了该模块功能，那不是非常多503错误了？这样的话，可以在前端启用该模块，要么就是设置白名单，白名单设置参见后续的文档，我会整理一份以供读者参考。

## ngx_http_limit_req_module 可以用来限制单个IP每秒请求数 ##

ngx_http_limit_req_module模块(0.7.21)可以通过定义的键值来限制请求处理的频率。特别的，它可以限制来自单个IP地址的请求处理频率。限制的方法是通过一种“漏桶”的方法——固定每秒处理的请求数，推迟过多的请求处理。

	limit_req_zone
	语法: limit_req_zone $variable zone=name:size rate=rate;
	默认值: none
	配置段: http

设置一块共享内存限制域用来保存键值的状态参数。 特别是保存了当前超出请求的数量。 键的值就是指定的变量（空值不会被计算）。

如：limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

说明：区域名称为one，大小为10m，平均处理的请求频率不能超过每秒一次。键值是客户端IP。

使用$binary_remote_addr变量， 可以将每条状态记录的大小减少到64个字节，这样1M的内存可以保存大约1万6千个64字节的记录。

如果限制域的存储空间耗尽了，对于后续所有请求，服务器都会返回 503 (Service Temporarily Unavailable)错误。

速度可以设置为每秒处理请求数和每分钟处理请求数，其值必须是整数，所以如果你需要指定每秒处理少于1个的请求，2秒处理一个请求，可以使用 “30r/m”。

	limit_req_log_level
	语法: limit_req_log_level info | notice | warn | error;
	默认值: limit_req_log_level error;
	配置段: http, server, location

设置你所希望的日志级别，当服务器因为频率过高拒绝或者延迟处理请求时可以记下相应级别的日志。 延迟记录的日志级别比拒绝的低一个级别；

比如， 如果设置“limit_req_log_level notice”， 延迟的日志就是info级别。

	limit_req_status
	语法: limit_req_status code;
	默认值: limit_req_status 503;
	配置段: http, server, location

该指令在1.3.15版本引入。设置拒绝请求的响应状态码。

	limit_req
	语法: limit_req zone=name [burst=number] [nodelay];
	默认值: —
	配置段: http, server, location

设置对应的共享内存限制域和允许被处理的最大请求数阈值。 

如果请求的频率超过了限制域配置的值，请求处理会被延迟，所以所有的请求都是以定义的频率被处理的。 超过频率限制的请求会被延迟，直到被延迟的请求数超过了定义的阈值，这时，这个请求会被终止，并返回503 (Service Temporarily Unavailable) 错误。这个阈值的默认值为0。

如：

{% highlight bash %}
{% raw %}

limit_req_zone $binary_remote_addr zone=ttlsa_com:10m rate=1r/s;
server {
location /www.ttlsa.com/ {
limit_req zone=ttlsa_com burst=5;
}
}

{% endraw %}
{% endhighlight %}

限制平均每秒不超过一个请求，同时允许超过频率限制的请求数不多于5个。

如果不希望超过的请求被延迟，可以用nodelay参数,如：
limit_req zone=ttlsa_com burst=5 nodelay;

### 实例配置 ###

{% highlight bash %}
{% raw %}

http {
limit_req_zone $binary_remote_addr zone=ttlsa_com:10m rate=1r/s;
server {
location ^~ /download/ {
limit_req zone=ttlsa_com burst=5;
alias /data/www.ttlsa.com/download/;
}
}
}

{% endraw %}
{% endhighlight %}

可能要对某些IP不做限制，需要使用到白名单

## nginx_limit_speed_module 可以用来对IP限速 ##

该模块能够限制从一个地址同时连接的总速度。

    limit_speed_zone
    语法：limit_speed_zone zone_name $variable memory_max_size
    默认值：no
    配置段：http
    
定义会话状态存储空间。会话的数目由所分配的变量$variable决定，该值取决于memory_max_size值。

如：limit_speed_zone one $binary_remote_addr 10m;

客户端的地址被用作会话。注意：该变量$binary_remote_addr是用来代替$remote_addr。

$remote_addr变量值的长度是7到15个字节。因此状态大小等于32或64字节。
$binary_remote_addr变量值的长度总是4个字节，因此状态大小始终是32字节。

1M共享空间可以处理3.2万个会话，每个会话32字节。

    limit_speed
    语法：limit_speed zone_name max_speed
    默认值：no
    配置段：http，server，location

该指令指定同一个IP的最大速度。

例如：如果限制每个IP地址的最大速度为100KB，同时同一个IP有10个并发连接，那么每个连接的速度为10KB。

### 这个需要安装 ###

{% highlight bash %}
{% raw %}

# ./configure –prefix=/usr/local/nginx –-add-module=./nginx_limit_speed_module-master
# make
# make install

{% endraw %}
{% endhighlight %}

### 实例配置 ###

{% highlight bash %}
{% raw %}

http {
limit_speed_zone www_ttlsa_com $binary_remote_addr 10m;
server {
location /download/ {
limit_speed www_ttlsa_com 100k;
}
}
}

{% endraw %}
{% endhighlight %}

----------

# Nginx下载限速配置 #

Nginx可以通过`HTTPLimitZoneModule`和`HTTPCoreModule`两个模块来实现对目录和IP进行下载限速。

先来一个配置示例看下：

{% highlight bash %}
{% raw %}

limit_zone one $binary_remote_addr 10m;
server {
listen       80;
server_name  test.361way.com;
location / {
root   /var/www/html;
index  index.html index.htm index.php;
autoindex on;
autoindex_exact_size off;
autoindex_localtime on;
limit_conn one 2;
limit_rate 10k;
}

{% endraw %}
{% endhighlight %}

该配置中分了两部分。上面一部分用到了模块HTTPLimitZoneModule的用法。

上面的配置中定义了一个名字为one大小为10M的容器，用于存储每个IP的session状态。该容器的大小要求大于等于32K，即每个session的大小为大于等于32k。按本例中10M大小来算，可以处理320000个session 。

配置完该容器后，HTTPLimitZoneModule模块下还有另外一个参数limit_conn，配合limit_zone参数使用。

如本例中，指定了one容器中，限制每个IP只能发起来两个连接。HTTPLimitZoneModule模块的详细用法可以参看其官方wiki页面。

本示例中的配置是只针对根目录的。如果要对其他目录设置，改为其应的location /path 即可。

下面接着看第二部分，即HTTPCoreModule模块部分。该模块所该的参数比较多。但对于速度方面的限制主要为limit_rate参数。该参数用于限制每个连接的速度大小。

本例中限制每个连接的最大下限速度为10k/s 。不过本例中对于每个IP的下载速度的峰值是多大呢？

很简间，单个IP的最大连接为2，每个连接的最大速度为10k，每个IP的最大速度即为：10k * 2 = 20k/s 。HTTPCoreModule模块的其他用法，也可以参看该模块的官方wiki 。

*总结：*

nginx以按默认方式编译安装的话自动会带以上两个模块的，不像apache需要通过编译时指定或动态加载第三方模块。从配置上来看，nginx的配置比较比较简洁，但功能上不如apache的mod_bw模块丰富。

----------

# Nginx查看并发访问量 #

apache下有mod_status和mod_info模块用于查看apache的运行状态。nginx是不是也带有相同的模块呢？答案当然是肯定的。

编译时只需要加上`–with-http_stub_status_module `参数即可在安装时编译出HttpStubStatusModule模块（不需要记，默认情况下都会自动加该参数的）。

安装完成后，可以通过`stub_status on`开启状态查看项。当然如果想设置的更安全些，可以增加上IP控制和密码认证。具体配置如下：

{% highlight bash %}
{% raw %}

location /nginx-status {
stub_status on;
auth_basic "NginxStatus";
allow 127.0.0.1;
deny all;
access_log off
auth_basic_user_file /App/nginx/conf/htpasswd;
} 

{% endraw %}
{% endhighlight %}

注：密码文件的生成需要依赖apache的htpasswd工具生成。

配置完成后，在浏览器中输入`http://127.0.0.1/nginx-status` 输入用户名密码，即可查看nginx的当前状态，示例如下：

{% highlight bash %}
{% raw %}

Active connections: 9787
server accepts handled requests
 515611486 515611486 1487172614
Reading: 208 Writing: 331 Waiting: 9248

{% endraw %}
{% endhighlight %}

而各个参数含义如下：

`reading — nginx `读取到客户端的 Header 信息数。

`writing — nginx `返回给客户端的 Header 信息数。

waiting — 开启 keep-alive 的情况下，这个值等于 active – (reading + writing)，意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接。

而从我上面的服务器状态上来看，nginx的waitng状态非常高，后来我到网上查到了资料并结合自己的分析，发现这是正常的。网上一些资料显示：在访问效率高,请求很快被处理完毕的情况下,Waiting数比较多是正常的 。

而我自己的本机只做nginx proxy转发，不负责数据处理。所以处理效率当然是非常高的。而从官方给出的waiting参数的意思上看来，是已经完成数据的处理。即已经将数据返回给用户，而停留在该状态是等待下一次的连接或都连接超时中断。（不知道这样理解对不对，希望大牛板砖）

需要注意的是如果reading或writing的值很高，说明正在处理的数据量很大，可能是因为后端的动态就用程序处理慢（如php、jsp) ，拖了后腿。而一般来说,动态应用之后以慢。一般有两方面的原因，一是因为数据库,另一个原因很可能就是IO慢（目前的机器CPU或内存不够用的情况很少，毕竟这玩意廉价。而设备的主要瓶颈在硬盘IO上）,或者客户端的网络。