---
layout: article
title:  "logstash elasticsearch redis Kibana"
date:   2015-12-05 09:20:00 +0800
image:
	teaser: /teaser/logstash+elasticsearch+redis+Kibana.jpg
categories: home server
---


正在底层阶级阶段混的我，师傅Q我，说让我搭建一个这个日志监控工具，我深深的醉了，2015年2月初，开始研究原理，之后过年放假必然玩耍，抛之于脑后不再管，回来后，开始研究搭建，真是各种问题各种出啊，费时约10天！！！


----------

# 搭建此架构前的思路 #

首先：你需要知道这四个东西的各个用途，这样才能有助于搭建使用

其次：你要知道他们之间怎么联系的，你才能实现相互间的配合

最后：搭建吧，没脾气、没脾气的。

# 这个架构的具体工作流程如下： #
 
logstash 主要处理两个事情，shipper和indexer。shipper 即进行日志的收集，indexer则对数据进行分析过滤处理以及持久化保存到elasticsearch中。redis 在中间起着中转的作用。Kibana 是一个日志可视化web程序。

在通俗点，就是logstash的shipper端把日志数据发送到redis，然后logstash的indexer端，在从redis把数据读取出来发送给elasticsearch中，最后kibana再把elasticsearch中的日志内容，以图形化展示出来。

如此简单的原理，大家肯定都懂得！

![](/images/server/logstash+elasticsearch+redis+Kibana.jpg)

----------

# 我的系统环境： #

系统：Ubuntu 14.04  (192.168.10.163 indexer端)

系统：Ubuntu 14.04  shipper端

软件：logstash-1.4.2.tar.gz、elasticsearch-1.4.2.tar.gz、kibana-3.1.2.tar.gz

友情提示：这三个之间是有版本兼容性的，注意安全哈，否则肯定不会成功的。切记！切记！


----------


# 搭建此服务 #

首先：是监控端（indexer端安装）
 
## 1.安装elasticsearch ##

{% highlight bash %}
{% raw %}
        
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.tar.gz
tar -zxvf elasticsearch-1.4.2.tar.gz
cd elasticsearch-1.4.2
vim config/elasticsearch.yml

{% endraw %}
{% endhighlight %}

添加以下内容

{% highlight bash %}
{% raw %}

cluster.name: elasticsearch
node.ame: "logstash test"
http.cors.allow-origin: "/.*/"
http.cors.enabled: true

{% endraw %}
{% endhighlight %}

这个的原因，我还没有查到，不过后面两个是为了开启传送给kibana用的参数
然后保存退出，执行以下命令

{% highlight bash %}
{% raw %}

./bin/elasticsearch

{% endraw %}
{% endhighlight %}

那么`elasticsearch`就算安装成功了

不要惊慌，确实就是这么简单

之后执行`netstat -ltnp` 查看是否有9200 端口 跟9300 端口存在

或者`http://localhost:9200`访问这个地址

{% highlight bash %}
{% raw %}

{
  "status" : 200,
  "name" : "Master Khan",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.4.2",
    "build_hash" : "927caff6f05403e936c20bf4529f144f0c89fd8c",
    "build_timestamp" : "2014-12-16T14:11:12Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.2"
  },
  "tagline" : "You Know, for Search"
}

{% endraw %}
{% endhighlight %}

看到如上内容，也是成功的意思

## 2.安装redis ##

这个的安装就更加的简单了，我用的ubuntu，所以，你懂得       

{% highlight bash %}
{% raw %}

sudo apt-get install redis-server    
vim /etc/redis/redis.conf 

{% endraw %}
{% endhighlight %}

更改如下内容

{% highlight bash %}
{% raw %}

daemonize yes
bind 0.0.0.0
appendonly yes

{% endraw %}
{% endhighlight %}

之后

{% highlight bash %}
{% raw %}

service redis-server restart

{% endraw %}
{% endhighlight %}

就行了
如果你要是centos就按照上面的redis资料安装把

## 3.安装logstash ##

这个就比较费事了，因为logstash运行仅仅依赖java运行环境(jre)。各位可以在命令行下运行`java -version`命令 显示类似如下结果：                   

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~#java -version
java version "1.7.0_75"
OpenJDK Runtime Environment (IcedTea 2.5.4) (7u75-2.5.4-1~trusty1)
OpenJDK 64-Bit Server VM (build 24.75-b04, mixed mode)

{% endraw %}
{% endhighlight %}

我用的是这个版本哦，如果你不会安装，就看一下资料吧，这里不做介绍了
然后就可以安装logstash了，也是超级简单的！

{% highlight bash %}
{% raw %}

wget https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz
tar -zxvf logstash-1.4.2.tar.gz
cd logstash-1.4.2
mkdir conf logs

{% endraw %}
{% endhighlight %}

然后基本配置就完成了

之后就需要大做文章了，创建配置文件server.conf

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~/logstash-1.4.2$ cat conf/server.conf 
input {
  redis {
    host => "192.168.10.163"
    type => "redis"
    data_type => "list"
    key => "logstash"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
stdout { codec => rubydebug }
  elasticsearch {
    host => "127.0.0.1"
    cluster => "elasticsearch"
    codec => rubydebug
  }
}

{% endraw %}
{% endhighlight %}

这个配置文件的意思是：
input从redis中接受数据，filte进行过滤，output发送给elasticsearch并且打印到屏幕输出

这里做个小介绍，具体的命令意思，我也不做太多介绍了，去看我的logstash资料就行了
这个文件保存后就可以启动logstash了，当然，如果没有安装jdk，你必然启动失败
执行如下命令

{% highlight bash %}
{% raw %}

sudo ./bin/logstash -f conf/server.conf

{% endraw %}
{% endhighlight %}

然后就正在启动中了  

现在，配置另外一台ubuntu，

## 4.安装logstash   ## 

这个安装的方法同上

{% highlight bash %}
{% raw %}

wget https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz
tar -zxvf logstash-1.4.2.tar.gz
cd logstash-1.4.2
mkdir conf logs

{% endraw %}
{% endhighlight %}

不过配置文件要有变动了

{% highlight bash %}
{% raw %}

root@root-virtual-machine:~/logstash-1.4.2$ cat etc/agent.conf 
input {
  file {
    type => "apache-access"
    path => "/var/log/test.log"
  }
}

filter {
  multiline {
    pattern => "^[^\[]"
    what => "previous"
  }
  }

output {
  stdout { }
  redis {
    host => "192.168.10.163"
    data_type => "list"
    key => "logstash"
  }
}

{% endraw %}
{% endhighlight %}

这个配置文件的意思是，input读取日志文件/var/log/test.log文件，filter过滤日志文件“一条多行”的不规则日志变成一条日志，output发送给192.168.10.163（服务器端）的redis服务

那么只需要启动logstash即可

{% highlight bash %}
{% raw %}

./bin/logstash -f conf/agent.conf

{% endraw %}
{% endhighlight %}

这样，就开始监控test.log这个日志文件了，
现在都配置好了，就可以配置kibana了

## 5.安装kibana ##

这个kibana是个牛X的东西，随便放到apache、nginx、lighttp等等就能用，所以，这里就先安装一个nginx就行了，我用的是lighttp

安装方法见我的资料吧，这里介绍的是kibana                

{% highlight bash %}
{% raw %}

wget https://download.elasticsearch.org/kibana/kibana/kibana-3.1.2.tar.gz
tar -zxvf kibana-3.1.2.tar.gz
vim kibana-3.1.2/config.js

{% endraw %}
{% endhighlight %}

更改如下

{% highlight bash %}
{% raw %}

elasticsearch: "http://"+window.location.hostname+":9200",

{% endraw %}
{% endhighlight %}

改成

{% highlight bash %}
{% raw %}

elasticsearch: "http://192.168.10.163:9200",

{% endraw %}
{% endhighlight %}

自己server服务器的IP地址

保存退出就行

然后把这个目录放到lighttp网页目录下

{% highlight bash %}
{% raw %}

mv kibana-3.1.2 /var/www/kibana

{% endraw %}
{% endhighlight %}

之后就可以通过网页访问了

    http://192.168.10.163/kibana

你会进入kibana 的默认界面，点击右下方的(Logstash Dashboard)就能进入logstash的界面了

当你看到数据，那就证明你所做的一切，都没有白费

如果你没有看到数据，那你就去排查吧，或者问一下我，我会的话，一定帮你

