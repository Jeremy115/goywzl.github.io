---
layout: article
title:  "Selinux简单介绍"
date:   2015-12-11 09:20:00 +0800
categories: linux
---

SELinux全称 Security Enhanced Linux (安全强化 Linux)，是一种基于 域-类型 模型（domain-type）的强制访问控制（MAC）安全系统，它由NSA编写并设计成内核模块包含到内核中，相应的某些安全相关的应用也被打了SELinux的补丁，最后还有一个相应的安全策略。虽然CentOS系统相比较而言相对安全稳定。

# Selinux三种状态： #

	Disabled 代表 SELinux 被禁用
	Permissive 代表仅记录安全警告但不阻止可疑行为
	Enforcing 代表记录警告且阻止可疑行为。

# 查看SELinux状态： #
 
{% highlight bash %}
{% raw %}

/usr/sbin/sestatus -v      ##如果SELinux status参数为enabled即为开启状态

SELinux status:                 enabled

getenforce                 ##也可以用这个命令查看

{% endraw %}
{% endhighlight %}

# 设置SELinux： #

1、临时设置（不用重启机器）：
 
{% highlight bash %}
{% raw %}

setenforce 0                  ##设置SELinux 成为permissive模式

setenforce 1                  ##设置SELinux 成为enforcing模式

{% endraw %}
{% endhighlight %}

*注：临时设置不能设置为disabled状态，只能通过更改配置文件才行。*

2、修改配置文件达到永久生效需要重启机器：

修改`/etc/selinux/config `文件

将SELINUX=enforcing改为SELINUX=disabled

重启机器即可

# 总结 #

有的人会为了省事，直接关闭selinux，但我感觉，其实它是很好的安全防护系统，只要略微的深究一下，就可以掌控它的基本功能了。