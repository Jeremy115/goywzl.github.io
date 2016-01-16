---
layout: article
title:  "Python mail module"
date:   2015-12-30 09:20:00 +0800
categories: script
---

Python邮件模块简介

python中email模块使得处理邮件变得比较简单，今天着重学习了一下发送邮件的具体做法，这里写写自己的的心得,也请高手给些指点。

一、相关模块介绍

发送邮件主要用到了smtplib和email两个模块，这里首先就两个模块进行一下简单的介绍：

1、smtplib模块

{% highlight python %}
{% raw %}

smtplib.SMTP([host[, port[, local_hostname[, timeout]]]])

{% endraw %}
{% endhighlight %}
 
SMTP类构造函数，表示与SMTP服务器之间的连接，通过这个连接可以向smtp服务器发送指令，执行相关操作（如：登陆、发送邮件）。所有参数都是可选的。
	host：smtp服务器主机名
	port：smtp服务的端口，默认是25；如果在创建SMTP对象的时候提供了这两个参数，在初始化的时候会自动调用connect方法去连接服务器。
	smtplib模块还提供了SMTP_SSL类和LMTP类，对它们的操作与SMTP基本一致。

smtplib.SMTP提供的方法：

     SMTP.set_debuglevel(level)：设置是否为调试模式。默认为False，即非调试模式，表示不输出任何调试信息。

     SMTP.connect([host[, port]])：连接到指定的smtp服务器。参数分别表示smpt主机和端口。注意: 也可以在host参数中指定端口号（如：smpt.yeah.net:25），这样就没必要给出port参数。

     SMTP.docmd(cmd[, argstring])：向smtp服务器发送指令。可选参数argstring表示指令的参数。

     SMTP.helo([hostname]) ：使用"helo"指令向服务器确认身份。相当于告诉smtp服务器“我是谁”。

     SMTP.has_extn(name)：判断指定名称在服务器邮件列表中是否存在。出于安全考虑，smtp服务器往往屏蔽了该指令。

     SMTP.verify(address) ：判断指定邮件地址是否在服务器中存在。出于安全考虑，smtp服务器往往屏蔽了该指令。

     SMTP.login(user, password) ：登陆到smtp服务器。现在几乎所有的smtp服务器，都必须在验证用户信息合法之后才允许发送邮件。

     SMTP.sendmail(from_addr, to_addrs, msg[, mail_options, rcpt_options]) ：发送邮件。这里要注意一下第三个参数，msg是字符串，表示邮件。我们知道邮件一般由标题，发信人，收件人，邮件内容，附件等构成，发送邮件的时候，要注意msg的格式。这个格式就是smtp协议中定义的格式。

     SMTP.quit() ：断开与smtp服务器的连接，相当于发送"quit"指令。（很多程序中都用到了smtp.close()，具体与quit的区别google了一下，也没找到答案。）

2、email模块

emial模块用来处理邮件消息，包括MIME和其他基于RFC 2822 的消息文档。使用这些模块来定义邮件的内容，是非常简单的。其包括的类有（更加详细的介绍可见：http://docs.python.org/library/email.mime.html）：

{% highlight python %}
{% raw %}

class email.mime.base.MIMEBase  (_maintype,_subtype,**_params)：

{% endraw %}
{% endhighlight %}

这是MIME的一个基类。一般不需要在使用时创建实例。

其中_maintype是内容类型，如text或者image。

_subtype是内容的minor type 类型，如plain或者gif。 

**_params是一个字典，直接传递给Message.add_header()。

    class email.mime.multipart.MIMEMultipart([_subtype[, boundary[, _subparts[, _params]]]]：MIMEBase的一个子类，多个MIME对象的集合，_subtype默认值为mixed。boundary是MIMEMultipart的边界，默认边界是可数的。

    class email.mime.application.MIMEApplication(_data[, _subtype[, _encoder[, **_params]]])：MIMEMultipart的一个子类。

    class email.mime.audio. MIMEAudio(_audiodata[, _subtype[, _encoder[, **_params]]])： MIME音频对象
    class email.mime.image.MIMEImage(_imagedata[, _subtype[, _encoder[, **_params]]])：MIME二进制文件对象。

    class email.mime.message.MIMEMessage(_msg[, _subtype])：具体的一个message实例，使用方法如下：
   
		msg=mail.Message.Message()    #一个实例 
		msg['to']='XXX@XXX.com'      #发送到哪里 
		msg['from']='YYY@YYYY.com'       #自己的邮件地址 
		msg['date']='2012-3-16'           #时间日期 
		msg['subject']='hello world'    #邮件主题 
 
    class email.mime.text.MIMEText(_text[, _subtype[, _charset]])：MIME文本对象，其中_text是邮件内容，_subtype邮件类型，可以是text/plain（普通文本邮件），html/plain(html邮件),  _charset编码，可以是gb2312等等。
 
 
下面介绍几种不同形式的邮件
 
 
1.文本形式的邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.text import MIMEText  
from email.header import Header  
  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msg = MIMEText('你好','plain','utf-8')#中文需参数‘utf-8’，单字节字符不需要  
msg['Subject'] = Header(subject, 'utf-8')  
  
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msg.as_string())  
smtp.quit()
 
{% endraw %}
{% endhighlight %}

2.文件形式的邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.text import MIMEText  
from email.header import Header  
  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msg = MIMEText('你好','text','utf-8')#中文需参数‘utf-8’，单字节字符不需要  
msg['Subject'] = Header(subject, 'utf-8')  
  
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msg.as_string())  
smtp.quit()  

{% endraw %}
{% endhighlight %}

3.HTML形式邮件
 
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.text import MIMEText  
  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msg = MIMEText('<html><h1>你好</h1></html>','html','utf-8')  
  
msg['Subject'] = subject  
  
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msg.as_string())  
smtp.quit() 

{% endraw %}
{% endhighlight %}

4.带图片的HTML邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.multipart import MIMEMultipart  
from email.mime.text import MIMEText  
from email.mime.image import MIMEImage  
  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msgRoot = MIMEMultipart('related')  
msgRoot['Subject'] = 'test message'  
  
msgText = MIMEText('<b>Some <i>HTML</i> text</b> and an image.<br><img src="cid:image1"><br>good!','html','utf-8')  
msgRoot.attach(msgText)  
  
fp = open('h:\\python\\1.jpg', 'rb')  
msgImage = MIMEImage(fp.read())  
fp.close()  
  
msgImage.add_header('Content-ID', '<image1>')  
msgRoot.attach(msgImage)  
  
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msgRoot.as_string())  
smtp.quit() 

{% endraw %}
{% endhighlight %}

5.带附件的邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.multipart import MIMEMultipart  
from email.mime.text import MIMEText  
from email.mime.image import MIMEImage  
  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msgRoot = MIMEMultipart('related')  
msgRoot['Subject'] = 'test message'  
  
#构造附件  
att = MIMEText(open('h:\\python\\1.jpg', 'rb').read(), 'base64', 'utf-8')  
att["Content-Type"] = 'application/octet-stream'  
att["Content-Disposition"] = 'attachment; filename="1.jpg"'  
msgRoot.attach(att)  
          
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msgRoot.as_string())  
smtp.quit()  
 
{% endraw %}
{% endhighlight %}

6.发送群邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.text import MIMEText  
  
sender = '***'  
receiver = ['***','****',……]  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msg = MIMEText('你好','text','utf-8')  
  
msg['Subject'] = subject  
  
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msg.as_string())  
smtp.quit() 

{% endraw %}
{% endhighlight %}

7.包含各种元素的邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.multipart import MIMEMultipart  
from email.mime.text import MIMEText  
from email.mime.image import MIMEImage  
  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
# Create message container - the correct MIME type is multipart/alternative.  
msg = MIMEMultipart('alternative')  
msg['Subject'] = "Link"  
  
# Create the body of the message (a plain-text and an HTML version).  
text = "Hi!\nHow are you?\nHere is the link you wanted:\nhttp://www.python.org"  
html = """\ 
<html> 
  <head></head> 
  <body> 
    <p>Hi!<br> 
       How are you?<br> 
       Here is the <a href="http://www.python.org">link</a> you wanted. 
    </p> 
  </body> 
</html> 
"""  
  
# Record the MIME types of both parts - text/plain and text/html.  
part1 = MIMEText(text, 'plain')  
part2 = MIMEText(html, 'html')  
  
# Attach parts into message container.  
# According to RFC 2046, the last part of a multipart message, in this case  
# the HTML message, is best and preferred.  
msg.attach(part1)  
msg.attach(part2)  
#构造附件  
att = MIMEText(open('h:\\python\\1.jpg', 'rb').read(), 'base64', 'utf-8')  
att["Content-Type"] = 'application/octet-stream'  
att["Content-Disposition"] = 'attachment; filename="1.jpg"'  
msg.attach(att)  
     
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msg.as_string())  
smtp.quit()  

{% endraw %}
{% endhighlight %}

8.基于SSL的邮件
 
{% highlight python %}
{% raw %}

#!/usr/bin/env python3  
#coding: utf-8  
import smtplib  
from email.mime.text import MIMEText  
from email.header import Header  
sender = '***'  
receiver = '***'  
subject = 'python email test'  
smtpserver = 'smtp.163.com'  
username = '***'  
password = '***'  
  
msg = MIMEText('你好','text','utf-8')#中文需参数‘utf-8’，单字节字符不需要  
msg['Subject'] = Header(subject, 'utf-8')  
  
smtp = smtplib.SMTP()  
smtp.connect('smtp.163.com')  
smtp.ehlo()  
smtp.starttls()  
smtp.ehlo()  
smtp.set_debuglevel(1)  
smtp.login(username, password)  
smtp.sendmail(sender, receiver, msg.as_string())  
smtp.quit()  

{% endraw %}
{% endhighlight %}
