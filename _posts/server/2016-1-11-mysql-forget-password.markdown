---
layout: article
title:  "Mysql forget password"
date:   2016-01-11 09:20:00 +0800
categories: server
---


# MySQL 忘记口令的解决办法 #

如果 MySQL 正在运行，首先杀之： `killall -TERM mysqld`。

启动 MySQL ：`bin/safe_mysqld –skip-grant-tables &`

就可以不需要密码就进入 MySQL 了。

然后就是

{% highlight bash %}
{% raw %}

>use mysql

>update user set password=password("new_pass") where user="root";

>flush privileges;

{% endraw %}
{% endhighlight %}

重新杀 MySQL ，用正常方法启动 MySQL 。

mysql密码清空

## Windows: ##

1.用系统管理员登陆系统。

2.停止MySQL的服务。

3.进入命令窗口，然后进入MySQL的安装目录，比如我的安装目录是c:mysql,进入C:mysqlbin

4.跳过权限检查启动MySQL，

c:mysqlbin>mysqld-nt –skip-grant-tables

5.重新打开一个窗口，进入c:mysqlbin目录，设置root的新密码

{% highlight bash %}
{% raw %}

c:mysqlbin>mysqladmin -u root flush-privileges password "newpassword"

c:mysqlbin>mysqladmin -u root -p shutdown

{% endraw %}
{% endhighlight %}

将newpassword替换为你要用的root的密码，第二个命令会提示你输入新密码，重复第一个命令输入的密码。

6.停止MySQL Server，用正常模式启动Mysql

7．你可以用新的密码链接到Mysql了。

## Unix&Linux： ##

1.用root或者运行mysqld的用户登录系统；

2．利用kill命令结束掉mysqld的进程；

3．使用–skip-grant-tables参数启动MySQL Server

{% highlight bash %}
{% raw %}

shell>mysqld_safe –skip-grant-tables &

{% endraw %}
{% endhighlight %}

4.为root@localhost设置新密码

{% highlight bash %}
{% raw %}

shell>mysqladmin -u root flush-privileges password "newpassword"

{% endraw %}
{% endhighlight %}

5．重启MySQL Server

----------

# mysql修改密码 #

	mysql修改，可在mysql命令行执行如下：

	mysql -u root mysql

	mysql> Update user SET password=PASSWORD("new password") Where user='name';

	mysql> FLUSH PRIVILEGES;

	mysql> QUIT

教你如何将MySQL数据库的密码恢复

因为MySQL密码存储于数据库mysql中的user表中，所以只需要将我windows 2003下的MySQL中的user表拷贝过来覆盖掉就行了。

在c:mysqldatamysql(linux 则一般在/var/lib/mysql/mysql/)目录下有三个user表相关文件user.frm、user.MYD、user.MYI

user.frm //user表样式文件

user.MYD //user表数据文件

user.MYI //user表索引文件

为保险起见，三个都拷贝过来，不过其实如果之前在要恢复的那个MySQL上没有更改过表结构的话，只要拷贝user.MYD就行了

然后

{% highlight bash %}
{% raw %}

#. /etc/rc.d/init.d/mysql stop

#. /etc/rc.d/init.d/mysql start

#mysql -u root -p XXXXXX

{% endraw %}
{% endhighlight %}

好了，可以用windows 2003下mysql密码登陆了

{% highlight bash %}
{% raw %}

mysql>use mysql

mysql>update user set Password=PASSWORD('xxxxxx') where User='root';

{% endraw %}
{% endhighlight %}

这时候会出错，提示user表只有读权限

我分析了一下原因，只这样的，因为user.*文件的权限分配是windows 2003下的，在windows 2003下我ls -l一看权限是666

在linux下我一看，拷过来后权限变成了600(其实正常情况下600就行了，只不过这里的文件属主不是mysql，拷过来后的属主变为了root,所以会出现权限不够，这时候如果你改成权限666则可以了，当然这样不好，没有解决问题的实质)，在/var/lib/mysql/mysql/下ls -l看了一下

再

{% highlight bash %}
{% raw %}

#chown -R mysql:mysql user.*

#chmod 600 user.*

//OK,DONE

{% endraw %}
{% endhighlight %}

重起一下MYSQL

重新连接

{% highlight bash %}
{% raw %}

mysql>use mysql

mysql>update user set Password=PASSWORD('xxxxxx') where User='root';

mysql>FLUSH PRIVILEGES;

{% endraw %}
{% endhighlight %}

有一点值得注意:如果你windows 下mysql如果是默认配置的话，注意要还要执行

{% highlight bash %}
{% raw %}

mysql>delete from user where User='';

mysql>delete from user where Host='%';

mysql>FLUSH PRIVILEGES;

{% endraw %}
{% endhighlight %}

好了，到这里恢复密码过程就完成了

这个方法么就是有点局限性，你必须也具备另外的user表文件

## 其他还有几种方法 ##

其它方法一(这个是网上流传较广的方法,mysql中文参考手册上的)

1.向mysqld server 发送kill命令关掉mysqld server(不是 kill -9),存放进程ID的文件通常在MYSQL的数据库所在的目录中。

{% highlight bash %}
{% raw %}

Killall -TERM mysqld

{% endraw %}
{% endhighlight %}

你必须是UNIX的root用户或者是你所运行的SERVER上的同等用户，才能执行这个操作。

2.使用`–skip-grant-tables' 参数来启动 mysqld。 LINUX下：

{% highlight bash %}
{% raw %}

/usr/bin/safe_mysqld –skip-grant-tables , windows下c:mysqlbinmysqld –skip-grant-tables

{% endraw %}
{% endhighlight %}

3.然后无密码登录到mysqld server ，

{% highlight bash %}
{% raw %}

>use mysql

>update user set password=password("new_pass") where user="root";

>flush privileges;

{% endraw %}
{% endhighlight %}

你也可以这样做：

{% highlight bash %}
{% raw %}

'mysqladmin -h hostname -u user password 'new password''

{% endraw %}
{% endhighlight %}

4.载入权限表：

{% highlight bash %}
{% raw %}

'mysqladmin -h hostname flush-privileges'

{% endraw %}
{% endhighlight %}

或者使用 SQL 命令

{% highlight bash %}
{% raw %}

'FLUSH PRIVILEGES'

{% endraw %}
{% endhighlight %}

5.`killall -TERM mysqld`

6.用新密码登陆

## 其它方法二 ##

直接用十六进制编辑器编辑user.MYD文件

不过这个里面我要说明一点，我这里编辑的时候发现个问题，加密的密码串有些是连续存储的，有些的最后两位被切开了，后两位存储在后面其他地方.这一点我还没想明白.还有注意一点就是编辑的是加密过的密码串，也就是说你还是需要另外有user表文件。这种方法和我最上面介绍的方法的区别在于，这种方法直接编辑linux下的user表文件，就不需要重新改文件属主和权限了

//后记，因为恢复过程比较好玩，所以写了篇笔记，不要砸我啊

//各位高手还有什么其他好玩的方法不要忘了告诉我哦:)

修正一下：我在Windows下的实际操作如下

1.关闭正在运行的MySQL。

2.打开DOS窗口，转到mysqlbin目录。

3.输入

{% highlight bash %}
{% raw %}

mysqld-nt –skip-grant-tables

{% endraw %}
{% endhighlight %}

回车。如果没有出现提示信息，那就对了。

4.再开一个DOS窗口（因为刚才那个DOS窗口已经不能动了），转到mysqlbin目录。

5.输入mysql回车，如果成功，将出现MySQL提示符 >

6. 连接权限数据库

{% highlight bash %}
{% raw %}

>use mysql;

{% endraw %}
{% endhighlight %}

(>是本来就有的提示符,别忘了最后的分号)

6.改密码：

{% highlight bash %}
{% raw %}

> update user set password=password("123456") where user="root"; (别忘了最后的分号)

{% endraw %}
{% endhighlight %}

7.刷新权限（必须的步骤）

{% highlight bash %}
{% raw %}

>flush privileges;

{% endraw %}
{% endhighlight %}

8.退出

{% highlight bash %}
{% raw %}

> q

{% endraw %}
{% endhighlight %}

9.注销系统，再进入，开MySQL，使用用户名root和刚才设置的新密码123456登陆。

据说可以用直接修改user表文件的方法：

关闭MySQL，Windows下打开Mysqldatamysql，有三个文件user.frm,user.MYD,user.MYI找个知道密码的MySQL，替换相应的这三个文件，如果user表结构没改过，一般也没人去改，替换user.MYD就可以了。

也可以直接编辑user.MYD，找个十六进制编辑器，UltraEdit就有这个功能。关闭MySQL，打开user.MYD。将用户名root后面的八个字符改为565491d704013245，新密码就是123456。或者将它们对应的十六进制数字，（左边那里，一个字符对应两个数字），改为 00 02 02 02 02 02 02 02,这就是空密码，在编辑器右边看到的都是星号*，看起来很象小数点。重开MySQL，输入root和你的新密码。