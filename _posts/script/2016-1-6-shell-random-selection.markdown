---
layout: article
title:  "Shell脚本之员工抽随机抽奖"
date:   2016-01-06 09:20:00 +0800
categories: script
---

前些日子，同事找我，说让我帮忙写一个shell脚本，我必然说可以啊，这是锻炼我啊！

但是，他来句写一个员工抽奖随机获取的，我丫的瞬间懵了，这对于我这个只会for循环，if判断这点皮毛的人来说，这就好比登天啊！

不过答应了都，就跟他一起研究，一起琢磨，嘿，还真让我写出来了，也不是很复杂！

**脚本功能**：在一个目录下，分别以员工名字命名文件，总共有N个文件（此处代表N个员工），然后，在公司老大（Boss）点抽取按钮，也就执行这个脚本，然后随机选出这个文件。

上面仅供参考，淡定，我只是做一个思路，毕竟打开文件，以及点击按钮执行脚本，这些功能都不是我的事，好了，废话那么多了，上脚本！

脚本执行方法:  `/bin/bash random.sh test`

	random.sh 脚本名字  test 目录

以下是脚本内容

	[root@localhost shell]# cat ./random.sh 

{% highlight bash %}
{% raw %}

#!/bin/bash
#this is Employees winning random drawing script
line_number=`ls $1 |wc -l`
if [ “$line_number” != “0” ];
    then
        all_number=`ls $1 | sort |awk '{print NR":" $0}' |tail -n 1 | awk -F ":" '{print $1}'`
        random_number=$(($RANDOM%$all_number+1))
        win_number=`ls $1 | sort |awk '{print NR":" $0}' |grep ^"$random_number"`
        for result in $win_number;
            do
                extraction_number=`echo $result |awk -F ":" '{print $1}'`
                extraction_name=`echo $result |awk -F ":" '{print $2}'`
                if [ “$extraction_number” = “$random_number” ];
                    then
                        echo $extraction_name
                fi
            done
    else
        echo "Bye Bye"
fi

{% endraw %}
{% endhighlight %}

还是那句，上面脚本供大家复制使用，个人比较喜欢看懂了脚本，学会之后都自己手打，下面做一下脚本的解释

{% highlight bash %}
{% raw %}

[root@localhost shell]# cat ./random.sh 
#!/bin/bash
#this is Employees winning random drawing script    #很明显，这是注释，脚本用途
line_number=`ls $1 |wc -l`                   #统计一下，这个目录下是否还有文件
if [ “$line_number” != “0” ];                 #判断目录下是否有文件，如果没有，就执行下面的“Bye Bye”
    then
#这个意思是，查看一下这个目录下总共有多少文件，也就是多少员工
        all_number=`ls $1 | sort |awk '{print NR":" $0}' |tail -n 1 | awk -F ":" '{print $1}'`
#这个意思是，在数字1到上述得到的所有员工如N之间，随机抽取一个数字
        random_number=$(($RANDOM%$all_number+1))
#这个意思是，查看随机抽取数字对应的中奖者，这里做一下详解：经测试，有可能会得到1，那么序列号为10的11的凡是带有1的都能选取到，所以有了下面的for循环，来只选出一个准确的中奖员工。
        win_number=`ls $1 | sort |awk '{print NR":" $0}' |grep ^"$random_number"`
#在上面得到的好几个中奖员工中循环
        for result in $win_number;
            do
#打印所有员工中第一个中奖的数字
                extraction_number=`echo $result |awk -F ":" '{print $1}'`
#打印所有员工中第一个中奖数字对应员工名字
                extraction_name=`echo $result |awk -F ":" '{print $2}'`
#然后，判断这个数字是否等于我们抽取随机数的那个数
                if [ “$extraction_number” = “$random_number” ];
#如果等于，那么就打印出这个中奖员工名字，如果不等于，继续for循环
                    then
                        echo $extraction_name
                fi
            done
    else
        echo "Bye Bye"
fi

{% endraw %}
{% endhighlight %}

*注：*

一些命令

	$0是第0个参数，也就是脚本本身   $1是1个参数，也就是test目录
	$(($RANDOM%2+1))　　 随机1到2 之间的数字
	Awk ‘{print NR”:” $0}’    进行排序，用：分隔