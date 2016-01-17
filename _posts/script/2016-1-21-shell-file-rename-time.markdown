---
layout: article
title:  "Shell脚本按文件时间戳改名"
date:   2016-01-21 09:20:00 +0800
categories: script
---

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

想想其实挺简单的，结果写的时间还是挺长，看来技术还是有待提高啊