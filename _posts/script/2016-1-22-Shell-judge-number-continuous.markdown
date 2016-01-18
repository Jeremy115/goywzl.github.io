---
layout: article
title:  "Shell judge number continuous"
date:   2016-1-22 09:20:00 +0800
categories: script
---

昨日朋友跟我说，他有一个文本，文本内容是连续的数字，一个数字占用一行，总共四万多个数字，如下图：

![continuous_number](/images/script/continuous_number.jpg)

他让我帮忙想想办法找到其中不连续的，刚才发布的是python的实现的，也是同事帮我完成的，因为python有好多不懂得，好吧，这是我不会的理由
然后知道思路之后，我用shell也写了一个，花了将近一个小时.

好了，思路还是python脚本—–数字是否连续，现在上脚本
 
# 脚本原来内容 #

{% highlight bash %}
{% raw %}

#!/bin/bash
# Filename : read_continuous.sh
# Explain : This is check file number continuous scripts!
file_content=`cat 982.txt`
old_number=0
rm -rf result.txt
for new_number in $file_content
        do
                content_number=$[new_number-1]
                if [ $content_number = $old_number ]
                        then
                                old_number=$new_number
                                continue
                else
                                result_number=$[new_number-old_number-1]
                                echo 这个$new_number数字之前少了$result_number个数 >> result.txt
                fi
                old_number=$new_number
done

{% endraw %}
{% endhighlight %}

# 下面是脚本内容的解释，以便我回忆 #

{% highlight bash %}
{% raw %}

#!/bin/bash
# Filename : read_continuous.sh
# Explain : This is check file number continuous scripts!
file_content=`cat 982.txt`
#打开文件内容赋值给file_content
old_number=0
#因为第一个数字是1，所以我把old_number赋值为0
rm -rf result.txt
#方便重复执行，删除结果文件
for new_number in $file_content
#循环得到的内容
        do
                content_number=$[new_number-1]
#把得到的第一个数字-1
                if [ $content_number = $old_number ]
#如果这个数字跟原内容相等
                        then
                                old_number=$new_number
#则把获取的数字赋值给old_number，方便下次对比
                                continue
#跳过下面的环节，直接进行下一次循环
                else
                                result_number=$[new_number-old_number-1]
#如果不相等，则把结果数字，也就是数字之间的差得出来，以便查阅之间少了几个
                echo 这个$new_number数字之前少了$result_number个数 >> result.txt
#把得到的结果追加到文件result.txt中
                fi
                old_number=$new_number
#这个循环结束后赋值
done
#提示：这里有两个old_number=$new_number，是因为上面有一个continue，当结果相等，会直接停止循环，也就不会执行第二个old_number=$new_number
#所以在continue之前，我也加上了一个old_number=$new_number

{% endraw %}
{% endhighlight %}
