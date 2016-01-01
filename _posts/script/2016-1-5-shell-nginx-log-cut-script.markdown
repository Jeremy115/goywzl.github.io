---
layout: article
title:  "shell nginx log cut script"
date:   2016-01-05 09:20:00 +0800
categories: script
---


# Nginx日志切割脚本 #

这个脚本，是我在做Awstats监控Nginx日志的时候所写的脚本。

因为Nginx不自带切割，所以这个脚本是做切割Nginx日志用的。

{% highlight bash %}
{% raw %}

root@linux:~/shell# cat cut_logs.sh
#!/bin/sh
#this nginx logs pathyy
yesterday=`date -d “yesterday” +“%Y%m%d”`
before_yesterday=`date -d “-2 day” +“%Y%m%d”`
Nginx_logs=“/var/log/nginx”
Log_Name=“ipaloma”
cd /tmp
[ -d $Nginx_Logs ] && cd $Nginx_logs || exit 1
[ -f $Log_Name.log ] && /bin/mv $Log_Name.log $Nginx_logs/backup/${Log_Name}_${yesterday}.log || exit 1
killall -s USR1 /usr/local/nginx/sbin/nginx
/bin/cp $Nginx_logs/backup/${Log_Name}_${yesterday}.log /home/ipaloma/log_backup/
/bin/chown ipaloma:ipaloma /home/ipaloma/log_backup/${Log_Name}_${yesterday}.log
cd /var/log/nginx/backup
[ -f  ${Log_Name}_${before_yesterday}.log ] && gzip ${Log_Name}_${before_yesterday}.log|| exit 1

{% endraw %}
{% endhighlight %}

上面的为大家复制用，下面的做一下解释，我作为一个初学者，一般我都会看明白了脚本，再手打上去，当然，也有偷懒的时候。

{% highlight bash %}
{% raw %}

root@linux:~/shell# cat cut_logs.sh
#!/bin/sh
#this nginx logs pathyy                  #养成一个好习惯，说明这个脚本是做啥的
yesterday=`date -d “yesterday” +”%Y%m%d”`      #定义昨天时间
before_yesterday=`date -d “-2 day” +”%Y%m%d”`   #定义前天时间
Nginx_logs=”/var/log/nginx”               #定义日志位置
Log_Name=”linux”                      #定义名字
cd /tmp                            #更换目录
[ -d $Nginx_Logs ] && cd $Nginx_logs || exit 1  #判断目录是否存在，存在则更换到此目录，不存在则退出
#下面一行的意思是：判断日志文件是否存在，如果存在则更改名字，不存在则退出
[ -f $Log_Name.log ] && /bin/mv $Log_Name.log $Nginx_logs/backup/${Log_Name}_${yesterday}.log || exit 1
killall -s USR1 /usr/local/nginx/sbin/nginx     #平滑重启Nginx
/bin/cp $Nginx_logs/backup/${Log_Name}_${yesterday}.log /home/log_backup/   #复制备份文件到指定目录
/bin/chown ipaloma:ipaloma /home/log_backup/${Log_Name}_${yesterday}.log   #更改属主以及属组
cd /var/log/nginx/backup                #切换目录
#下面的命令是：判断前天的文件是否存在，如果存在，则用压缩
[ -f  ${Log_Name}_${before_yesterday}.log ] && gzip ${Log_Name}_${before_yesterday}.log|| exit 1
 
{% endraw %}
{% endhighlight %}

----------

上面写的是单个日志

这次领导让做很多日志，则出现以下脚本

{% highlight bash %}
{% raw %}


root@linux:~/scripts# cat cut_logs.sh  

#!/bin/sh  

#this nginx logs pathyy   

yesterday=`date -d “yesterday” +“%Y%m%d”`   

before_yesterday=`date -d “-2 day” +“%Y%m%d”`   

Nginx_logs=“/var/log/nginx”  

Nginx_old_logs=“/var/log/nginx/old”  

All_log_name=`ls $Nginx_logs | grep “.log”`   

cd $Nginx_old_logs   

for all_log in $All_log_name   

do  

    /bin/mv $Nginx_logs/$all_log $Nginx_old_logs/${yesterday}_${all_log}   

    [ -f ${before_yesterday}_${all_log} ] && gzip ${before_yesterday}_${all_log}   

done   

/usr/local/nginx/sbin/nginx -s reload  

{% endraw %}
{% endhighlight %}

 
只是用了一个for循环，别的没什么技术含量了