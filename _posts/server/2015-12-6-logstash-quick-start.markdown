---
layout: article
title:  "Logstash Quick Start"
date:   2015-12-06 09:20:00 +0800
categories: home server
---

logstash 是一个应用程序日志、事件的传输、处理、管理和搜索的平台。你可以用它来统一对应用程序日志进行收集管理，提供 Web 接口用于查询和统计。

Logstash提供了多种多样的 input,filters,codecs和output组件，让使用者轻松实现强大的功能。

----------

# Logstash安装 #

## 1.安装依赖语言 ##

Logstash依赖于JAVA，所以在使用logstash的时候，你要先安装JAVA，然后才能使用。

越新版本的Logstash，你最好安装越新的JAVA，可以使用`java -version`查看

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~# java -version
java version "1.7.0_75"
OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-1~trusty1)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)

{% endraw %}
{% endhighlight %}

提供一个下载地址：

	http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

## 2.安装logstash ##

既然准备环境已经做好了，那现在就开始搭建使用吧。

Logstash的下载地址，也是Logstash的官网地址，自己看喽！

	http://logstash.net/

下载完成之后解压就能用了，就是这么简单(我用的版本是logstash-1.4.2.tar.gz)

{% highlight bash %}
{% raw %}

wget https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz
tar -zxvf logstash-1.4.2.tar.gz
cd logstash-1.4.2

{% endraw %}
{% endhighlight %}

# 测试logstash #

现在做一个小测试，运行一下logstash

{% highlight bash %}
{% raw %}

bin/logstash -e ‘input { stdin { } } output { stdout {} }’

{% endraw %}
{% endhighlight %}

此句话的意思是，input接收屏幕输入，output输出到屏幕打印

测试：输入`this is test `

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~/logstash-1.4.2$ bin/logstash -e 'input { stdin { } } output { stdout {} }'
this is test
2015-03-09T07:17:33.171+0000 huihui-virtual-machine this is test

{% endraw %}
{% endhighlight %}

以上例子我们在运行logstash中，定义了一个叫”stdin”的input还有一个”stdout”的output，无论我们输入什么字符，Logstash都会按照某种格式来返回我们输入的字符。这里注意我们在命令行中使用了-e参数，该参数允许Logstash直接通过命令行接受设置。

上面的做完，证明你的logstash是没有问题的，可以做以下操作

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~/logstash-1.4.2$ bin/logstash -e 'input { stdin { } } output { stdout { codec => rubydebug } }'
this is test
{
       "message" => "this is test",
      "@version" => "1",
    "@timestamp" => "2015-03-09T07:20:09.013Z",
          "host" => "huihui-virtual-machine"
}

{% endraw %}
{% endhighlight %}

以上示例通过重新设置了叫”stdout”的output(添加了”codec”参数)，我们就可以改变Logstash的输出表现。类似的我们可以通过在你的配置文件中添加或者修改inputs、outputs、filters，就可以使随意的格式化日志数据成为可能，从而订制更合理的存储格式为查询提供便利。

# 事件的生命周期 #

Inputs,Outputs,Codecs,Filters构成了Logstash的核心配置项。Logstash通过建立一条事件处理的管道，从你的日志提取出数据保存到Elasticsearch中，为高效的查询数据提供基础。为了让你快速的了解Logstash提供的多种选项，让我们先讨论一下最常用的一些配置。更多的信息，请参考Logstash事件管道。

## Inputs ##

input 及输入是指日志数据传输到Logstash中。其中常见的配置如下： file：从文件系统中读取一个文件，很像UNIX命令 “tail -0a”syslog：监听514端口，按照RFC3164标准解析日志数据redis：从redis服务器读取数据，支持channel(发布订阅)和list模式。redis一般在Logstash消费集群中作为”broker”角色，保存events队列共Logstash消费。lumberjack：使用lumberjack协议来接收数据，目前已经改为 logstash-forwarder。 

## Filters ##

Fillters 在Logstash处理链中担任中间处理组件。他们经常被组合起来实现一些特定的行为来，处理匹配特定规则的事件流。常见的filters如下： grok：解析无规则的文字并转化为有结构的格式。Grok 是目前最好的方式来将无结构的数据转换为有结构可查询的数据。有120多种匹配规则，会有一种满足你的需要。mutate：mutate filter 允许改变输入的文档，你可以从命名，删除，移动或者修改字段在处理事件的过程中。drop：丢弃一部分events不进行处理，例如：debug events。clone：拷贝 event，这个过程中也可以添加或移除字段。geoip：添加地理信息(为前台kibana图形化展示使用)

## Outputs ##

outputs是logstash处理管道的最末端组件。一个event可以在处理过程中经过多重输出，但是一旦所有的outputs都执行结束，这个event也就完成生命周期。一些常用的outputs包括： elasticsearch：如果你计划将高效的保存数据，并且能够方便和简单的进行查询…Elasticsearch是一个好的方式。是的，此处有做广告的嫌疑,呵呵。file：将event数据保存到文件中。graphite：将event数据发送到图形化组件中，一个很流行的开源存储图形化展示的组件。http://graphite.wikidot.com/。statsd：statsd是一个统计服务，比如技术和时间统计，通过udp通讯，聚合一个或者多个后台服务，如果你已经开始使用statsd，该选项对你应该很有用。 

## Codecs ##

codecs 是基于数据流的过滤器，它可以作为input，output的一部分配置。Codecs可以帮助你轻松的分割发送过来已经被序列化的数据。流行的codecs包括 json,msgpack,plain(text)。 json：使用json格式对数据进行编码/解码multiline：将汇多个事件中数据汇总为一个单一的行。比如：java异常信息和堆栈信息 获取完整的配置信息，请参考 Logstash文档中 “plugin configuration”部分。

# logstash的配置 #

## 配置文件 ##

我们总不能一直输那么长的一段内容，现在咱们可以使用一下配置文件，来方便日后使用。

{% highlight bash %}
{% raw %}

mkdir conf
vim conf/server.conf
input { stdin { } }
output {
  stdout { codec => rubydebug }
}
bin/logstash -f conf/server.conf

{% endraw %}
{% endhighlight %}

他就会按照配置文件中的内容执行，跟上面的列子是一样的，结果也是一样的。

## 过滤器 ##

filters是一个行处理机制将提供的为格式化的数据整理成你需要的数据，首先介绍一个例子，叫grok filter的过滤器。

{% highlight bash %}
{% raw %}

input { stdin { } }

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
  date {
    match => [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
  }
}

output {
  stdout { codec => rubydebug }
}
bin/logstash -f conf/server.conf

{% endraw %}
{% endhighlight %}

启动之后，就可以放一条日志信息做测试了

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~/logstash-1.4.2$ ./bin/logstash -f etc/test.conf 
127.0.0.1 - - [09/Mar/2015:00:01:45 -0800] "GET /xampp/status.php HTTP/1.1" 200 3891 "http://cadenza/xampp/navi.php" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36"
{
        "message" => "127.0.0.1 - - [09/Mar/2015:00:01:45 -0800] \"GET /xampp/status.php HTTP/1.1\" 200 3891 \"http://cadenza/xampp/navi.php\" \"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36\"",
       "@version" => "1",
     "@timestamp" => "2015-03-09T08:01:45.000Z",
           "host" => "root-virtual-machine",
       "clientip" => "127.0.0.1",
          "ident" => "-",
           "auth" => "-",
      "timestamp" => "09/Mar/2015:00:01:45 -0800",
           "verb" => "GET",
        "request" => "/xampp/status.php",
    "httpversion" => "1.1",
       "response" => "200",
          "bytes" => "3891",
       "referrer" => "\"http://cadenza/xampp/navi.php\"",
          "agent" => "\"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.76 Safari/537.36\""
}

{% endraw %}
{% endhighlight %}

Logstash(使用了grok过滤器)能够将一行的日志数据(Apache的”combined log”格式)分割设置为不同的数据字段。这一点对于日后解析和查询我们自己的日志数据非常有用。

比如：HTTP的返回状态码，IP地址相关等等，非常的容易。很少有匹配规则没有被grok包含，所以如果你正尝试的解析一些常见的日志格式，或许已经有人为了做了这样的工作。

# logstash更多实用功能 #

这里说一下有需要的人，比如说你的日志不是默认类型的，是有回车的，那么他就会出现不一样的情况了，明明一条日志，会出现两个记录，这里就体现出Filters的强大了。

这个叫合并日志

php的错误日志中常常会出现这样的日志

{% highlight php %}
{% raw %}

[03-Jun-2013 13:15:29] PHP Fatal error:  Uncaught exception 'Leb_Exception' in /data1/www/bbs.xman.com/htdocs/framework/xbox/ufo.php:68
Stack trace:
#0 /data/www/bbs.xman.com/htdocs/framework/dao/abstract.php(299): Leb_Dao_Pdo->connect(Array, 'read')
#1 /data/www/bbs.xman.com/htdocs/framework/dao/pdo.php(108): Leb_Dao_Abstract->initConnect(false)
#2 /data/www/bbs.xman.com/htdocs/framework/dao/abstract.php(1123): Leb_Dao_Pdo->query('SELECT * FROM `...')
#3 /data/www/bbs.xman.com/htdocs/framework/dao/abstract.php(1217): Leb_Dao_Abstract->select(Array)
#4 /data/www/bbs.xman.com/htdocs/framework/model.php(735): Leb_Dao_Abstract->daoSelect(Array, false)
#5 /data/www/bbs.xman.com/htdocs/app/configure/model/configure.php(40): Leb_Model->find()
#6 /data/www/bbs.xman.com/htdocs/app/search/default.php(131): Configure->get_configure_by_type('news')
#7 /data/www/bbs.xman.com/htdocs/framework/dispatcher.php(291): defaultController->indexAction()
#8 /data/www/bbs.xman.com/htdocs/framework/dispatcher.php(222): Leb_Di in /data1/www/bbs.xman.com/htdocs/framework/dao/pdo.php on line 68

{% endraw %}
{% endhighlight %}

这个时候 logstash一般会只记录上面一行，所以这类的日志就看不全了。怎么办呢？logstash提供了一个功能解决了这个问题就是”multiline”

这个filter的功能顾名思义就是对多行的日志进行处理 这个是官网上的说明

	multiline filter
	This filter will collapse multiline messages into a single event.
	The multiline filter is for combining multiple events from a single source into the same event. 

下面看下格式

{% highlight bash %}
{% raw %}

filter {
  multiline {
    type => "type"   #类型,不多说
    pattern => "pattern, a regexp" #参数,也可以认为是字符,有点像grep ,如果符合什么字符就交给下面的 what 去处理
    negate => boolean
    what => "previous" or "next" #这个是符合上面 pattern 的要求后具体怎么处理,处理方法有两种,合并到上面一条日志或者下面的日志
  }
}

{% endraw %}
{% endhighlight %}

	The ‘negate’ can be “true” or “false” (defaults false). 
	If true, a message not matching the pattern will constitute 
	a match of the multiline filter and the what will be applied. 

	(vice-versa is also true)

    这个 negate 有两种 true 或者 false，默认是 true,如果选了false 的话估计就是取反的意思。

看看例子

{% highlight bash %}
{% raw %}

filter {
  multiline {
    pattern => "^[^\[]"
    what => "previous"
  }
  }

{% endraw %}
{% endhighlight %}

这个例子是针对我上面的php日志写的，意思就是 如果不是以 “[“开头的日志 都跟上一个日志合并在一起。以此类推遇到其他的多行日志也可以按照这个方法来做合并。
