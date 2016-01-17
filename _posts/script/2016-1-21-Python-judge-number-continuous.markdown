---
layout: article
title:  "Python judge number continuous"
date:   2016-1-21 09:20:00 +0800
categories: script
---

昨日朋友跟我说，他有一个文本，文本内容是连续的数字，一个数字占用一行，总共四万多个数字，如下图：

continuous_number

他让我帮忙想想办法找到其中不连续的，当时的第一想法，当然是用shell，后来用shell也写了出来

可是感觉既然开始学习了python，就要实践，于是，自己准备用python做一下。

起初我的想法是：打开文件—>for循环文件内容—>然后再次循环1到四万多—>最后if判断这两个内容是否相等，不想等就是不连续，后来发现是错误的
没办法，只好咨询同事，请教大拿！

大拿的想法是取出一个老数字（也就是上一个数字），然后获取新数字（也就是下一行的数字）并且减去1（因为他们的递增关系是+1，所以判断是否相等的话需要-1）

如果相等，则跳过，如果不想等，则打印这个新数字，并且运算出这个新数字减去旧数字剩余的数（也就是这个数字跟上一个数字之间相差几个数），得出的结果！

我现在明确感觉，开发的思路问题，决定一切，确实要经常编写脚本，把自己的思路改变，才能更快、更方便、更简洁的编写脚本。

好了，废话那么多，改说脚本了。
 
# 脚本原内容： #

{% highlight python %}
{% raw %}

#!/usr/bin/python
# -*- coding: utf-8 -*-
# Filename : read_continuous.py
# Explain : This is check file number continuous scripts!
original_file = open(‘982.txt’)
result_file = open(‘result.txt’, ‘w’)
file_content = original_file.read()
file_number = file_content.split(‘\n’)
old_number = file_number[0]
for item in file_number[1:]:
        if not item.isdigit():
                continue
        if not old_number.isdigit():
                old_number = item
                continue
        if int(old_number) == int(item) – 1:
                old_number = item
        else:
                result_file.write(‘这个 %s 数字之前少了 %d 个数字\n’ %(item, int(item) – int(old_number) – 1))
                old_number = item
original_file.close()
result_file.close()

{% endraw %}
{% endhighlight %}

# 下面是对脚本内容的解释，以便我以后回忆 #

{% highlight python %}
{% raw %}

#!/usr/bin/python
#表明python程序位置
# -*- coding: utf-8 -*-
#可以显示中文
# Filename : read_continuous.py
#文件名字
# Explain : This is check file number continuous scripts!
#脚本用处说明 
original_file = open(‘982.txt’)
#首先使用open方法打开982这个文件
result_file = open(‘result.txt’, ‘w’)
#然后再创建并打开result这个打印结果的文件，不输出到屏幕上，放到此文件中
file_content = original_file.read()
#赋值变量：把文件内容赋值给file_content，它是一个字符串
file_number = file_content.split(‘\n’)
#这里使用split(‘\n’)方法，使字符串以\n为分隔符，把内容变换成一个列表
old_number = file_number[0]
#这里我们要取得第一个列表内容，0代表第一个，1代表第二个
for item in file_number[1:]:
#使用for循环列表内容，从第二个开始，一直到最后一个
        if not item.isdigit():
#判断获取的列表内容是否为digit（数字）
                continue
#如果不是数字，则跳过，进行下一个循环
        if not old_number.isdigit():
#判断第一个获取的old_number是否为数字
                old_number = item
#如果不是数字，则把item获取的值赋值给它
                continue
#并跳过循环，进行下一个
        if int(old_number) == int(item) – 1:
#在进行数字对比的时候，需要把它们进行整数改变（int），然后进行比较
                old_number = item
#如果相等，则把现在获取的item赋值给old_number
        else:
                result_file.write(‘这个 %s 数字之前少了 %d 个数字\n’ %(item, int(item) – int(old_number) – 1))
#如果不相等，则把内容写到result_file文件中，其中%s 跟 %d 是获取后面item跟下一行与上一行相差几个数运算得出的结果
                old_number = item
#然后再把新的item的值赋值给old_number
original_file.close()
#这两个close是关闭文件用的，脚本执行完会自动关闭，这里体现我的B格而已，哈哈，可以省略不要
result_file.close()

{% endraw %}
{% endhighlight %}
