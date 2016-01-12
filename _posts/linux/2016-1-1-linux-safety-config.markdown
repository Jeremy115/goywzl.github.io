---
layout: article
title:  "Linux服务器的简单安全配置"
date:   2016-01-01 09:20:00 +0800
categories: linux
---


# 一、  禁用root以外的超级用户 #

1.检测方法：

{% highlight bash %}
{% raw %}

cat /etc/passwd   查看口令文件，文件格式如下
login_name：password：user_ID：group_ID：comment：home_dir：command
  
{% endraw %}
{% endhighlight %}

若user_ID=0，则该用户拥有超级用户的权限。查看此处是否有多个ID=0

2.检测命令:

{% highlight bash %}
{% raw %}

cat /etc/passwd | awk -F ‘:’ ‘{print$1,$3}’ | grep ‘ 0$’
  
{% endraw %}
{% endhighlight %}

3.备份方法：

{% highlight bash %}
{% raw %}

cp -p /etc/passwd /etc/passwd_bak
  
{% endraw %}
{% endhighlight %}

4.加固方法:

使用命令passwd -l <用户名>锁定不必要的超级账户

使用命令passwd -u <用户名>解锁需要恢复的超级账户

或把用户shell改为/sbin/nologin

 
# 二、   删除不必要的账号 #

1.应该删除所有默认的被操作系统本身启动的并且不必要的账号，Linux提供了很多默认账号，而账号越多，系统就越容易受到攻击。

2.可删除的用户,如

{% highlight bash %}
{% raw %}

adm,lp,sync,shutdown,halt,news,uucp,operator,games,gopher等
  
{% endraw %}
{% endhighlight %}

3.可删除的组,如

{% highlight bash %}
{% raw %}

adm,lp,news,uucp,games,dip,pppusers,popusers,slipusers等
  
{% endraw %}
{% endhighlight %}

4.删除命令

{% highlight bash %}
{% raw %}

userdel username
groupdel groupname
   
{% endraw %}
{% endhighlight %}

# 三、   用户口令设置 #

用户口令是Linux/Unix安全的一个基本起点，很多人使用的用户口令过于简单，这等于给侵入者敞开了大门，虽然从理论上说，只要有足够的时间和资源可以利用，就没有不能破解的用户口令，但选取得当的口令是难于破解的。较好的用户口令是那些只有他自己容易记得并理解的一串字符，最好不要把密码记录出来，如果有需要的话，也要保管好记录密码的文件，或者将这个文件加密。

生产环境口令要求:包含大写字母、小写字母、数字和特殊字符四种中的三种，并且口令整体长度大于10位，每台服务器的口令不相同。
 
# 四、   检查空口令账号 #

如果发现有账号口令为空，需要强制加入符合规格的口令

检查方法： 

{% highlight bash %}
{% raw %}

#awk -F: ‘($2 == “”) { print $1}’ /etc/shadow
  
{% endraw %}
{% endhighlight %}

# 五、   口令文件加锁 #

chattr命令给下面的文件加上不可更改属性，从而防止非授权用户获得权限。

{% highlight bash %}
{% raw %}

#chattr +i/etc/passwd
#chattr +i/etc/shadow
#chattr +i/etc/group
#chattr +i/etc/gshadow
  
{% endraw %}
{% endhighlight %}

# 六、   设置root账户自动注销时限 #

修改环境引导文件/etc/profile中的TMOUT参数，TMOUT参数按秒计算
vim/etc/profile

在”HISTFILESIZE=”后面加入下面这行

{% highlight bash %}
{% raw %}

TMOUT=300
 
{% endraw %}
{% endhighlight %}

改变这项设置后，必须先注销用户，再用该用户登陆才能激活这个功能
如果想修改某个用户的自动注销时限，可以在用户目录下的”.bashrc”文件中添加该值，以便系统对该用户实行特殊的自动注销时间
 
# 七、   限制su命令 #

禁止任何人能够su切换为root，编辑/etc/pam.d/su文件，增加如下行：
authrequired pam_wheel.so use_uid

这时，仅wheel组的用户可以su作为root。此后，如果希望用户admin能够su作为root，可以运行如下命令：

{% highlight bash %}
{% raw %}

#usermod –G 10 admin
  
{% endraw %}
{% endhighlight %}

# 八、   限制普通用户无法执行关机、重启、配置网络等敏感操作 #

删除/etc/security/console.apps下的halt、reboot、poweroff、shutdown等程序的访问控制文件，以禁止普通用户执行该命令

也可以整体删除/etc/security/console.apps下的所有配置文件

{% highlight bash %}
{% raw %}

rm –rf /etc/security/console.apps/*
 
{% endraw %}
{% endhighlight %}

# 九、   禁用Ctrl+Alt+Delete组合键重新启动机器命令 #

修改/etc/inittab文件，将”ca::ctrlaltdel:/sbin/shutdown-t3-rnow”一行注释掉。
 
# 十、   设置开机启动服务文件夹权限 #

设置/etc/rc.d/init.d/目录下所有文件的许可权限，此目录下文件
为开机启动项，运行如下命令：

{% highlight bash %}
{% raw %}

#chmod –R 700 /etc/rc.d/init.d/

{% endraw %}
{% endhighlight %}

这样便仅有root可以读、写或执行上述所有脚本文件。
 
# 十一、避免login时显示系统和版本信息 #

删除信息文件：

{% highlight bash %}
{% raw %}

rm –rf /etc/issue
rm –rf /etc/issue.net

{% endraw %}
{% endhighlight %}

十二、用防火墙关闭不须要的任何端口，别人PING不到服务器，威胁自然减少了一大半。

防止别人ping的方法：

1）命令提示符下打，0表示允许，1表示禁止

{% highlight bash %}
{% raw %}

echo 1　> /proc/sys/net/ipv4/icmp_ignore_all

{% endraw %}
{% endhighlight %}

2）用防火墙禁止(或丢弃) icmp 包

{% highlight bash %}
{% raw %}

iptables -A INPUT -p icmp -j DROP

{% endraw %}
{% endhighlight %}

3）对所有用ICMP通讯的包不予响应，比如：

{% highlight bash %}
{% raw %}

PING TRACERT 

{% endraw %}
{% endhighlight %}

# 十三、开启安全模式（做为商业应用的服务器不建议开启） #

{% highlight bash %}
{% raw %}

#vi /usr/local/Zend/etc/php.ini(没装ZO时php.ini文件位置为：/etc/php.ini) .
safe_mode = On

{% endraw %}
{% endhighlight %}

# 十四、锁定PHP程序应用目录 #

{% highlight bash %}
{% raw %}

#vi /etc tpd/conf.d irtualhost.conf
php_admin_value open_basedir /home/*** （***为站点目录） !

{% endraw %}
{% endhighlight %}

# 十五、屏蔽PHP不安全的参数(webshell) #

{% highlight bash %}
{% raw %}

#vi /usr/local/Zend/etc/php.ini (没装ZO时php.ini文件位置为：/etc/php.ini)
disable_functions = system,exec,shell_exec,passthru,popen
 
{% endraw %}
{% endhighlight %}

以下为我的服务器屏蔽参数：

{% highlight bash %}
{% raw %}

disable_functions = passthru,exec,shell_exec,system,set_time_limit,ini_alter,dl, .
pfsockopen,openlog,syslog,readlink,symlink,link,leak,fsockopen,popen,escapeshell ..
cmd,error_log .

{% endraw %}
{% endhighlight %}

# 十六、千万不要给不必要的目录给写权限，也就是777权限，根目录保持为711权限，如果不能运行PHP请改为755 。 #

# 十七、禁用不必要的用户登录 #

在Linux上，有多种方式让不安份的用户无法登录。

### 1.修改用户配置文件/etc/shadow，将第二栏设置为“*” ###

如下。那么该用户就无法登录。但是使用这种方式会导致该用户的密码丢失，要再次使用时，需重设密码[再次启用这个帐号的方法是把“*”去掉就可以了

{% highlight bash %}
{% raw %}

test:*:15230:0:99999:7:::

{% endraw %}
{% endhighlight %}

### 2.使用命令usermod，这个比较好用 ###

`usermod -L test`  锁定帐号test

`usermod -U test`  解锁帐号test

### 3.通过修改shell类型 ###

这种方式会更加人性化一点，因为你不仅可以禁止用户登录，还可以告诉他你这么做的原因。如下：

`chsh test -s /sbin/nologin` 将用户testid的shell进行更改

修改`/etc/nologin.txt`(没有就新建一个)，

在里面添加给被禁止用户的提示

解禁用户的方式就是把shell改为他原有的就可以了。

### 4.禁止所有的用户登录 ###

如果你是root用户，当你不想让所有用户登录时（比如你要维护系统升级什么的），如果按上面的方式，一

个一个地去禁止用户登录，这将是很……无聊的事。而且还容易出错。下面有一种简洁有效的方式：

在/etc目录下建立一个nologin文档

`touch /etc/nologin` 如果该文件存在，那么Linux上的所有用户（除了root以外）都无法登录

在/etc/nologin（注意：这可不是3中的nologin.txt啊！）写点什么，告诉用户为何无法登录

{% highlight bash %}
{% raw %}

cat /etc/nologin
9：00－10：00 

系统升级，所有用户都禁止登录！

{% endraw %}
{% endhighlight %}

解禁帐号也简单，直接将/etc/nologin删除就行了！