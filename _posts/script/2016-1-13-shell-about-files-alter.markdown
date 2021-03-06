---
layout: article
title:  "Shell脚本之文件修改"
date:   2016-01-13 09:20:00 +0800
categories: script
---

# 批量删除目录下文件 #

通常ls列出的文件,想直接管道通过rm -rf删除是无效的.这时就要配合命令xargs使用了:

例如:

`ls -t|tail -10|xargs rm -rf  `  //按时间排序,删除最后的10个文件

当然,也可以用`ls -lt|tail -100|awk ‘{ print $9 }’|xargs rm -rf  ` 两者效果是相同的同理,如果想按时间倒序排列,也就是说离现在最近的时间越排在最后显示,就用ls -rt当然,也可以使用find 配合rm删除。

也可以用下面的语句清空内容

{% highlight bash %}
{% raw %}

#!/bin/bash
for i in `ls test.log.??` ;do
echo "">$i
done

{% endraw %}
{% endhighlight %}

当然，如果是多个目录下面，再清除文件，也是可以的。只不过多了一层嵌套而忆。我的目录结构如下：

{% highlight bash %}
{% raw %}

[root@web tomcat]# ll
总计 260
drwxr-xr-x 2 www www 20480 12-28 09:23 bbs
drwxr-xr-x 2 www www 16384 12-28 00:00 comment
drwxr-xr-x 2 www www 36864 12-28 03:50 enterprise
drwxr-xr-x 2 www www 24576 12-28 00:00 expert
drwxr-xr-x 2 www www 36864 12-28 00:02 feedback
drwxr-xr-x 2 www www 36864 11-15 12:30 generator
drwxr-xr-x 2 www www 24576 12-28 00:02 passport
drwxr-xr-x 2 www www 20480 12-28 00:00 search
drwxr-xr-x 2 www www 20480 12-28 09:35 usercenter

{% endraw %}
{% endhighlight %}

{% highlight bash %}
{% raw %}

[root@web comment]# ll
总计 936
-rw-rw-r-- 1 www www       2 12-28 09:57 catalina.2012-12-24.log
-rw-rw-r-- 1 www www       2 12-28 09:57 catalina.2012-12-25.log
-rw-rw-r-- 1 www www       2 12-28 09:57 catalina.2012-12-26.log
-rw-rw-r-- 1 www www       2 12-28 09:57 catalina.2012-12-27.log
-rw-r--r-- 1 www www   34155 12-28 10:01 catalina.out
-rw-rw-r-- 1 www www       2 12-28 09:57 localhost.2012-12-14.log
-rw-rw-r-- 1 www www       2 12-28 09:57 localhost.2012-12-17.log
-rw-rw-r-- 1 www www       2 12-28 09:57 localhost_access_log.2012-12-25.txt
-rw-rw-r-- 1 www www       2 12-28 09:57 localhost_access_log.2012-12-26.txt
-rw-rw-r-- 1 www www  629729 12-28 09:57 localhost_access_log.2012-12-27.txt

{% endraw %}
{% endhighlight %}

如果想清空以上所有目录里的所有文件，就可以这样做：

{% highlight bash %}
{% raw %}

for i in `ls`;do (cd $i ;for m in `ls`;do echo " ">$m;done);done

{% endraw %}
{% endhighlight %}

*注*：上面的圆括号是不能少的。

而如果想要清空一个文件的内容，再使用xargs配合echo “”>file，发现确不能正常生效。如：

{% highlight bash %}
{% raw %}

find . -name "test.*" |xargs echo "">

{% endraw %}
{% endhighlight %}

因为，find和echo的不是这样配合使用的。其两者简单配合有另外一个妙用：

{% highlight bash %}
{% raw %}

find . -name "file*" -print | xargs echo "" > /tmp/find.log

{% endraw %}
{% endhighlight %}

该语句的作用，是将当前目录下，所有以file开头的文件查找到，并将其相对路径及名称输入到find.log文件中，输入方式为每行一个。对符合条件的原文件不会做任何改变。

而如果想实现find查找并清空文件，难道我们就没办法了吗？很显然，这是不可能的。菜鸟们的办法是：

{% highlight bash %}
{% raw %}

#!/bin/bash
for i in `find ./server* -name "test.log" `
do
cat /dev/null > $i
done

{% endraw %}
{% endhighlight %}

高手显然不屑于使用这么长的语句，高手们的办法是：

{% highlight bash %}
{% raw %}

[root@localhost log]# find . -name "maillog*" |awk '{ print "echo>"$0}'|bash

{% endraw %}
{% endhighlight %}

或者把索性把echo也去掉

{% highlight bash %}
{% raw %}

[root@localhost log]# find . -name "maillog*" |awk '{ print ">"$0}'|bash

{% endraw %}
{% endhighlight %}

该语句是如何变化为来的得呢？

{% highlight bash %}
{% raw %}

[root@localhost log]# find . -name "maillog*"|xargs -i ls -l {}
-rw------- 1 root root 0 11-09 05:06 ./maillog
-rw------- 1 root root 0 11-09 05:06 ./maillog.1

{% endraw %}
{% endhighlight %}

以上为找出所有文件，而再利用强大的awk，可以将所有输出的文件前加 echo “”> 以bash语句的方式出现

{% highlight bash %}
{% raw %}

[root@localhost log]# find . -name "maillog*"|awk '{print "echo > "$0}'
echo > ./maillog
echo > ./maillog.1

{% endraw %}
{% endhighlight %}

注：我上面的例子，只是列了一层目录，而find查找时，是会将其下面的子目录内符合条件的语句也会包含进来。

也可以按下面的指令进行find查找，批量清空文件：

{% highlight bash %}
{% raw %}

find . –type f –size +5M –exec cp /dev/null {} \;

{% endraw %}
{% endhighlight %}


----------

# 按文件时间戳改名 #

领导有一些日志文件，是按照1287331200这样记载的，是从1970年到现在用的秒数

然后领导要我对这个目录下，所有这样命名的文件更改名字

文件名字如下：`system_log@1438173635192.txt.gz`

特此，我写了一个脚本，以便日后怀念

{% highlight bash %}
{% raw %}

#!/bin/bash
find . -name “*.gz” | xargs gunzip -v
for file in `find ./ -name “*.txt”`
do
    time_name=`echo $file | awk -F ‘@’ ‘{print substr($2,1,10)}’`
    new_time_name=`date -d “1970-01-01 UTC $time_name seconds” “+%F_%T”`
    front_name=`echo $file | awk -F ‘@’ ‘{print $1}’`
    mv $file  $front_name$new_time_name.txt
done

{% endraw %}
{% endhighlight %}

**原理解说：**

首先，需要解压缩文件

{% highlight bash %}
{% raw %}

find . -name “*.gz” | xargs gunzip -v
#查找当前目录下以.gz结尾的文件，并解压

{% endraw %}
{% endhighlight %}

然后，使用for循环，过滤当前目录下，所有以.txt结尾的文件（因为怕有别的类型文件，不然ls多好）

{% highlight bash %}
{% raw %}

for file in `find ./ -name “*.txt”`
#查找当前目录下以.txt结尾的文件，并挨个给file变量赋值，进行循环

{% endraw %}
{% endhighlight %}

之后，因为文件名`system_log@1438173635192.txt`是这样的，所以给变量time_name赋值，通过awk截取@后面的时间戳

{% highlight bash %}
{% raw %}

time_name=`echo $file | awk -F ‘@’ ‘{print substr($2,1,10)}’`

{% endraw %}
{% endhighlight %}

下一步给new_time_name变量赋值，使用date命令，把时间戳改为`2015-07-30_04:14:5`这样子的格式

{% highlight bash %}
{% raw %}

new_time_name=`date -d “1970-01-01 UTC $time_name seconds” “+%F_%T”`

{% endraw %}
{% endhighlight %}

最后变量front_name，通过awk命令获取的就是@前面的名字，这样在改名的时候用的到

{% highlight bash %}
{% raw %}

front_name=`echo $file | awk -F ‘@’ ‘{print $1}’`

{% endraw %}
{% endhighlight %}

最后就是用mv更改名字了，这个就很简单了

{% highlight bash %}
{% raw %}

mv $file  $front_name$new_time_name.txt

{% endraw %}
{% endhighlight %}

