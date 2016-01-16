---
layout: article
title:  "Sendmail server"
date:   2016-1-20 09:20:00 +0800
categories: server
---

Sendmail是UNIX/Linux环境中稳定性较好的一款邮件服务器软件，通过对Sendmail服务器的配置可以实现基本的邮件转发功能.

----------

# Ubuntu install 'sendmail' #

## 安装： ##

{% highlight bash %}
{% raw %}

sudo apt-get install sendmail
sudo apt-get install sendmail-cf

{% endraw %}
{% endhighlight %}

Ubuntu下使用最常用的mail功能，需要安装mailutils，

安装命令：`sudo apt-get install mailutils`

使用带附件的功能，则还需要安装sharutils，

安装命令：`sudo apt-get install sharutils`；（`yum install sharutils` )

上面这两个是必需的,还有几个可选的:

    squirrelmail 提供webmail
    spamassassin 提供邮件过滤
    mailman 提供邮件列表支持
    dovecot 提供IMAP和POP接收邮件服务器守护进程
    ubuntu sendmail安装好后就可以配置文件,发送邮件了,具体使用参见man.

## 配置 ##

sendmail 默认只会为本机用户发送邮件，只有把它扩展到整个Internet，才会成为真正的邮件服务器。

1.打开sendmail的配置宏文件：/etc/mail/sendmail.mc

{% highlight bash %}
{% raw %}

vi  /etc/mail/sendmail.mc

{% endraw %}
{% endhighlight %}

2.找到如下行：

{% highlight bash %}
{% raw %}

DAEMON_OPTIONS(`Family=inet,  Name=MTA-v4, Port=smtp, Addr=127.0.0.1′)dnl

{% endraw %}
{% endhighlight %}

修改Addr=0.0.0.0  ，表明可以连接到任何服务器。

3.生成新的配置文件：

{% highlight bash %}
{% raw %}

#cd /etc/mail
#mv sendmail.cf sendmail.cf~      //做一个备份
m4 sendmail.mc > sendmail.cf      //做一个备份 #m4 sendmail.mc > sendmail.cf   //>的左右有空格，提示错误没有安装sendmail-cf

{% endraw %}
{% endhighlight %}

## 测试发送邮件 ##

常用发送邮件方式如下：

    1.如何写一般的邮件： mail test@126.com  Cc 编辑抄送对象，Subject:邮件主题,输入回车，邮件正文后，按Ctrl-D结束
    2.快速发送方式： echo "邮件正文" | mail -s 邮件主题 test@126.com
    3.以文件内容作为邮件正文来发送： mail -s test test@126.com < test.txt
    4.发送带附件的邮件： uuencode 附件名称 附件显示名称 | mail -s 邮件主题 发送地址
		例如： uuencode test.txt test.txt | mail -s Test flynewton@gmail.com

小提醒：将sendmail使用的域名进行相应的修改，系统默认为localhost.localdomain,其实不改也行，不过有些pop3服务器会过滤掉来自localhost.localdomain的邮件，导致邮件不能正常查收，所以，最好是改一下 hostname，确保邮件发送的成功率。

----------

# Centos install 'sendmail' #

## 安装： ##

{% highlight bash %}
{% raw %}

#yum install -y sendmail
#yum install -y sendmail-cf

{% endraw %}
{% endhighlight %}

如果需要SMTP验证就安装并启动saslauthd服务：

{% highlight bash %}
{% raw %}

# yum install -y saslauthd
# service saslauthd start

{% endraw %}
{% endhighlight %}

## 配置 ##

(1) 配置Senmail的SMTP认证#

{% highlight bash %}
{% raw %}

vi /etc/mail/sendmail.mc 

dnl TRUST_AUTH_MECH(`EXTERNAL DIGEST-MD5 CRAM-MD5 LOGIN PLAIN’)dnl
dnl define(`confAUTH_MECHANISMS’, `EXTERNAL GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN PLAIN’)dnl

{% endraw %}
{% endhighlight %}

将上面两行的dnl去掉。在sendmail文件中，dnl表示该行为注释行，是无效的，因此通过去除行首的dnl字符串可以开启相应的设置行。

(2) 设置Sendmail服务的网络访问权限（因为我是直接本机调用所以我没有操作这个步骤）

{% highlight bash %}
{% raw %}

# vi /etc/mailndmail.mc
DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA’)dnl

{% endraw %}
{% endhighlight %}

将127.0.0.1改为0.0.0.0，意思是任何主机都可以访问Sendmail服务。如果仅让某一个网段能够访问到Sendmail服务，将127.0.0.1改为形如192.168.1.0/24的一个特定网段地址。

(3) 生成配置文件

Sendmail的配置文件由m4来生成，m4工具在sendmail-cf包中。如果系统无法识别m4命令，说明sendmail-cf软件包没有安装。

{% highlight bash %}
{% raw %}

m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf

{% endraw %}
{% endhighlight %}

(4) 启动服务

{% highlight bash %}
{% raw %}

service sendmail start

{% endraw %}
{% endhighlight %}

检查服务是否加入自启行列

{% highlight bash %}
{% raw %}

chkconfig –list |grep sendmail
 
{% endraw %}
{% endhighlight %}

## 测试发送邮件 ##

常用发送邮件方式如下：

    1.如何写一般的邮件： mail test@126.com  Cc 编辑抄送对象，Subject:邮件主题,输入回车，邮件正文后，按Ctrl-D结束
    2.快速发送方式： echo “邮件正文” | mail -s 邮件主题 test@126.com
    3.以文件内容作为邮件正文来发送： mail -s test test@126.com < test.txt
    4.发送带附件的邮件： uuencode 附件名称 附件显示名称 | mail -s 邮件主题 发送地址
		例如： uuencode test.txt test.txt | mail -s Test test@gmail.com