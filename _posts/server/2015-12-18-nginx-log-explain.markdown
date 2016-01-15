---
layout: article
title:  "Nginx日志参数"
date:   2015-12-18 09:20:00 +0800
categories: server
---


# 日志说明简介 #

nginx日志主要有两种：访问日志和错误日志。

访问日志主要记录客户端访问nginx的每一个请求，格式可以自定义；

错误日志主要记录客户端访问nginx出错时的日志，格式不支持自定义。

两种日志都可以选择性关闭。

通过访问日志，你可以得到用户地域来源、跳转来源、使用终端、某个URL访问量等相关信息；

通过错误日志，你可以得到系统某个服务或server的性能瓶颈等。

因此，将日志好好利用，你可以得到很多有价值的信息。

----------

# 配置说明 #

访问日志[Access.log]

{% highlight bash %}
{% raw %}

log_format  main  ‘$remote_addr $remote_user [$time_local] “$request” $http_host ‘
‘$status $upstream_status $body_bytes_sent “$http_referer” ‘
‘”$http_user_agent” $ssl_protocol $ssl_cipher $upstream_addr ‘
‘$request_time $upstream_response_time’;

{% endraw %}
{% endhighlight %}


<table>
<tr><td>变量名称</td> <td>变量描述</td> <td>举例说明</td></tr>
<tr><td>$remote_addr</td> <td>客户端地址</td> <td>113.140.15.90</td></tr>
<tr><td>$remote_user</td> <td>客户端用户名称</td> <td>-</td></tr>
<tr><td>$time_local</td> <td>访问时间和时区</td> <td>18/Jul/2012:17:00:01 +0800</td></tr>
<tr><td>$request</td> <td>请求的URI和HTTP协议</td><td>“GET /pa/img/home/logo-alipay-t.png HTTP/1.1″</td></tr>
<tr><td>$http_host</td> <td>请求地址，即浏览器中你输入的地址（IP或域名）</td><td>img.alipay.com10.253.70.103</td></tr>
<tr><td>$status</td> <td>HTTP请求状态</td><td>200</td></tr>
<tr><td>$upstream_status</td> <td>upstream状态</td><td>200</td></tr>
<tr><td>$body_bytes_sent</td><td>发送给客户端文件内容大小</td><td>547</td></tr>
<tr><td>$http_referer</td><td>跳转来源</td><td> “https://cashier.alipay.com…/”</td></tr>
<tr><td>$http_user_agent</td><td>用户终端代理</td><td>“Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 5.1; Trident/4.0; SV1; GTB7.0; .NET4.0C;</td></tr>
<tr><td>$ssl_protocol</td><td>SSL协议版本</td><td>TLSv1</td></tr>
<tr><td>$ssl_cipher</td><td>交换数据中的算法</td><td>RC4-SHA</td></tr>
<tr><td>$upstream_addr</td><td>后台upstream的地址，即真正提供服务的主机地址</td><td> 10.228.35.247:80</td></tr>
<tr><td>$request_time</td><td>整个请求的总时间</td><td>0.205</td></tr>
<tr><td>$upstream_response_time</td><td>请求过程中，upstream响应时间</td><td>0.002</td></tr>
</table>

----------
		
## request_time 和 upstream_response_time 差别 ##

根据nginx的accesslog中$request_time进行程序优化，发现有个接口，直接返回数据，平均的$request_time也比较大。

原来$request_time包含了用户数据接收时间，而真正程序的响应时间应该用$upstream_response_time。

### 下面介绍下2者的差别： ###

#### 1、request_time ####

官网描述：

request processing time in seconds with a milliseconds resolution; time elapsed between the first bytes were read from the client and the log write after the last bytes were sent to the client 。

指的就是从接受用户请求的第一个字节到发送完响应数据的时间，即包括接收请求数据时间、程序响应时间、输出响应数据时间。

#### 2、upstream_response_time ####

官网描述：

keeps times of responses obtained from upstream servers; times are kept in seconds with a milliseconds resolution. Several response times are separated by commas and colons like addresses in the $upstream_addrvariable 

是指从Nginx向后端（php-cgi)建立连接开始到接受完数据然后关闭连接为止的时间。

从上面的描述可以看出，$request_time肯定比$upstream_response_time值大，特别是使用POST方式传递参数时，因为Nginx会把request body缓存住，接受完毕后才会把数据一起发给后端。

所以如果用户网络较差，或者传递数据较大时，$request_time会比$upstream_response_time大很多。

所以如果使用nginx的accesslog查看php程序中哪些接口比较慢的话，记得在log_format中加入$upstream_response_time

----------

前面介绍了Nginx访问日志参数，这里面介绍以下错误日志Error.log怎么看，其中都有哪些参数需要注意的

### error_log指令 ###

语法: error_log file | stderr | syslog:server=address[,parameter=value] [debug | info | notice | warn | error | crit | alert | emerg];

    默认值: error_log logs/error.log error;
    配置段: main, http, server, location

### 配置错误日志。 ###

错误日志[Error.log]文件内容介绍
<table>
<tr><td>错误信息</td> <td>错误说明</td></tr>
<tr><td>“upstream prematurely（过早的）closed connection”</td> <td>请求uri的时候出现的异常，是由于upstream还未返回应答给用户时用户断掉连接造成的，对系统没有影响，可以忽略</td></tr>
<tr><td>“recv() failed (104: Connectionresetby peer)”</td> <td>（1）服务器的并发连接数超过了其承载量，服务器会将其中一些连接Down掉； （2）客户关掉了浏览器，而服务器还在给客户端发送数据； （3）浏览器端按了Stop</td></tr>
<tr><td>“(111: Connection refused)whileconnecting to upstream”</td> <td>用户在连接时，若遇到后端upstream挂掉或者不通，会收到该错误</td></tr>
<tr><td>“(111: Connection refused)whilereading response header from upstream”</td> <td>用户在连接成功后读取数据时，若遇到后端upstream挂掉或者不通，会收到该错误</td></tr>
<tr><td>“(111: Connection refused) whilesending request to upstream”</td> <td>Nginx和upstream连接成功后发送数据时，若遇到后端upstream挂掉或者不通，会收到该错误</td></tr>
<tr><td>“(110: Connection timed out)whileconnecting to upstream”</td> <td>nginx连接后面的upstream时超时</td></tr>
<tr><td>“(110: Connection timed out) whilereading upstream”</td> <td>nginx读取来自upstream的响应时超时</td></tr>
<tr><td>“(110: Connection timed out) whilereading response header from upstream”</td> <td>nginx读取来自upstream的响应头时超时</td></tr>
<tr><td>“SSL_do_handshake() failed”</td> <td>SSL握手失败</td></tr>
<tr><td>“(104: Connection reset bypeer)whileconnecting to upstream”</td> <td>upstream发送了RST，将连接重置</td></tr>
<tr><td>“upstream sent invalid header whilereading response header from upstream”</td> <td>upstream发送的响应头无效</td></tr>
<tr><td>“upstream sent no valid HTTP/1.0header whilereading responseheaderfrom upstream”</td> <td>upstream发送的响应头无效</td></tr>
<tr><td>“client intended to sendtoo large body”</td> <td>用于设置允许接受的客户端请求内容的最大值，默认值是1M，client发送的body超过了设置值</td></tr>
<tr><td>“reopening logs”</td> <td>	用户发送kill  -USR1命令</td></tr>
<tr><td>“gracefully shutting down”,</td> <td>	用户发送kill  -WINCH命令</td></tr>
<tr><td>“no servers are inside upstream”</td> <td>upstream下未配置server</td></tr>
<tr><td>“no live upstreams whileconnectingto upstream”</td> <td>upstream下的server全都挂了</td></tr>
<tr><td>“could not add new SSLsessiontothesession cache whileSSLhandshaking”</td> <td>ssl_session_cache大小不够等原因造成</td></tr>
<tr><td>“ngx_slab_alloc() failed: nomemoryinSSL session shared cache”</td> <td>ssl_session_cache大小不够等原因造成</td></tr>
</table>
	 
