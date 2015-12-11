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

		
		
		
		
		