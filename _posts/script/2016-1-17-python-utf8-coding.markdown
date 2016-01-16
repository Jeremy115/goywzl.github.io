---
layout: article
title:  "Python utf8 coding"
date:   2016-1-17 09:20:00 +0800
categories: script
---

今天学习python，菜鸟级人物，刚看完python搭建，我感觉这个就很简单，所以就不单独发表文章，在这里带过了

安装：去官网下载（http://www.python.org/download/）

下载及解压压缩包。

如果你需要自定义一些选项修改`Modules/Setup`

{% highlight bash %}
{% raw %}

./configure
make
make install

{% endraw %}
{% endhighlight %}

这就安装完成了

下面就是我说到的python中文编码

{% highlight python %}
{% raw %}

#!/usr/bin/python
print “你好”
 
{% endraw %}
{% endhighlight %}

当你执行这个脚本的时候，会报错

{% highlight python %}
{% raw %}

File “hello.py”, line 2
SyntaxError: Non-ASCII character ‘\xe4’ in file hello.py on line 2, but no encoding declared; see http://www.python.org/peps/pep-0263.html for details
 
{% endraw %}
{% endhighlight %}

如果你细心，就会看到上面有告诉你方法的地址，你打开那个地址，就能看到解决方法，我感觉挺智能了。

我这篇文章记录的是使用#coding=utf-8的方法，也就是如下编辑

{% highlight python %}
{% raw %}

#!/usr/bin/python
#coding=utf-8
print “你好”
  
{% endraw %}
{% endhighlight %}

这样当你在执行命令的时候，就会发现，没有任何问题了。

可是，我记得我领导编辑的时候，提过这个，他用的是# -*- coding: utf-8 -*-这个东西

{% highlight python %}
{% raw %}

#!/usr/bin/python
# -*- coding: utf-8 -*-
print “你好”
  
{% endraw %}
{% endhighlight %}

当然，如果你打开上面的报错提示地址，还有一种方法是下面这样，不过我是没有见过

{% highlight bash %}
{% raw %}

#!/usr/bin/python
# vim: set fileencoding=<encoding name> :
 
{% endraw %}
{% endhighlight %}

根据我领导说得，似乎第二种方法比较实用，第一种有时候会不好用，具体原因没有深究，不过听有经验的人说，应该会好点。