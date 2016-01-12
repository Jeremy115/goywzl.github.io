---
layout: article
title:  "Python simple sendmail"
date:   2016-1-17 09:20:00 +0800
categories: script
---

{% highlight python %}
{% raw %}

 
#!/usr/bin/env python3  
#coding: utf-8 
import smtplib
from email.mime.text import MIMEText
from email.header import Header
sender = 'sendmail@sina.com'                   发送邮箱账号
receiver = 'test@sina.com'                    接受邮箱账号
subject = 'python email test'                  邮件主题
smtpserver = 'mail.sina.com'                   邮件服务器地址
username = 'sendmail@ipaloma.com'                发送邮箱登录账号
password = 'passwd'                         发送邮箱登录密码

msg = MIMEText('<html><h1>这是Python发送邮件测试</h1></html>','html','utf-8')
msg[‘Subject’] = subject
smtp = smtplib.SMTP()
smtp.connect('smtp.qq.com')
smtp.login(username, password)
smtp.sendmail(sender, receiver, msg.as_string())
smtp.quit()

{% endraw %}
{% endhighlight %}
