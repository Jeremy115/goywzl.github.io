---
layout: article
title:  "Screen and Tmux"
date:   2015-12-02 09:20:00 +0800
categories: home linux
---


介绍一下linux多路复用终端命令

由于经常使用ssh远程linux主机，但是你要进行一段很长的命令操作（如ftp传输，wget下载，scp传输等）的时候，不能关闭终端，如果关闭了，你的命令也会中断，很不理想，所以出现了终端复用的命令，可以让你的命令在终端断开的情况下使其后台运行。

我这里介绍两种终端复用命令：Screen 、 Tmux


----------


# 首先说一下古老的Screen #

功能：会话恢复、多窗口、会话共享

## 安装Screen ##

1.Cenots下安装：


{% highlight bash %}
{% raw %}

yum install -y screen

{% endraw %}
{% endhighlight %}

2.Ubuntu下安装：

{% highlight bash %}
{% raw %}

apt-get install screen

{% endraw %}
{% endhighlight %}

我经常用到这俩，就先说这俩种方法

## 语法介绍 ##

{% highlight bash %}
{% raw %}

screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]

{% endraw %}
{% endhighlight %}

参数说明：

    -A 　将所有的视窗都调整为目前终端机的大小。
    -d <作业名称> 　将指定的screen作业离线。
    -h <行数> 　指定视窗的缓冲区行数。
    -m 　即使目前已在作业中的screen作业，仍强制建立新的screen作业。
    -r <作业名称> 　恢复离线的screen作业。
    -R 　先试图恢复离线的作业。若找不到离线的作业，即建立新的screen作业。
    -s 　指定建立新视窗时，所要执行的shell。
    -S <作业名称> 　指定screen作业的名称。
    -v 　显示版本信息。
    -x 　恢复之前离线的screen作业。
    -ls或--list 　显示目前所有的screen作业。
    -wipe 　检查目前所有的screen作业，并删除已经无法使用的screen作业。


命令参数介绍完了，下面说一些快捷方式

在每个`screen session`下，所有命令都以 ctrl+a(C-a) 开始。

    C-a ? -> 显示所有键绑定信息，即帮助信息
    C-a c -> 创建一个新的运行shell的窗口并切换到该窗口，相当于登录系统的时多图形的命令`ctrl + alt + F1-FN`
    C-a n -> Next，切换到下一个 `screen session`
    C-a p -> Previous，切换到前一个 `screen session`
    C-a 0..9 -> 切换到第 0..9 个 `screen session`
    Ctrl+a [Space] -> 由视窗0循序切换到视窗9相当于`C-a n`
    C-a C-a -> 在两个最近使用的 `screen session` 互相切换 
    C-a x -> 锁住当前的 `screen session`，需用用户密码解锁
    C-a d -> detach，暂时离开当前session，相当于`screen -d`
    C-a z -> 把当前session放到后台执行，用 shell 的 fg 命令则可回去。
    C-a w -> 显示所有窗口列表，会在左下角显示
    C-a t -> Time，显示当前时间，和系统的 load 
    C-a k -> kill window，强行关闭当前的 `screen session`
    C-a [ -> 进入 copy mode，在 copy mode 下可以回滚、搜索、复制就像用使用 vi 一样
    	C-b Backward，PageUp 
    	C-f Forward，PageDown 
    	H(大写) High，将光标移至左上角 
    	L Low，将光标移至左下角 
    	0 移到行首 
    	$ 行末 
    	w forward one word，以字为单位往后移 
    	b backward one word，以字为单位往前移 
    	Space 第一次按为标记区起点，第二次按为终点 
    	Esc 结束 copy mode 
    C-a ] -> Paste，把刚刚在 copy mode 选定的内容贴上

*小提示：如果在screen窗口上输入`exit`，则是直接退出screen，不能在切换回来，所以，请使用`screen -d`离线退出*
    
## 实例操示范 ##

1.创建新的session


{% highlight bash %}
{% raw %}

screen -S test

{% endraw %}
{% endhighlight %}

screen启动后，会创建第一个窗口，也就是窗口No. 0，并在其中打开一个系统默认的shell，一般都会是bash。

{% highlight bash %}
{% raw %}

screen vi text.txt

{% endraw %}
{% endhighlight %}

screen创建一个执行`vi text.txt`的单窗口会话，退出vi 将退出该窗口/会话。

2.离线退出当前所使用的session


{% highlight bash %}
{% raw %}

screen -d test

{% endraw %}
{% endhighlight %}


3.列出当前所有的session

{% highlight bash %}
{% raw %}

[root@localhost ~]# screen -ls
There are screens on:
	3624.test	(Attached)
	3709.xg	(Detached)
2 Sockets in /var/run/screen/S-root.
	
{% endraw %}
{% endhighlight %}

4.回到某个session


{% highlight bash %}
{% raw %}

screen -r test

{% endraw %}
{% endhighlight %}


## screen高级应用 ##

1.会话共享

还有一种比较好玩的会话恢复，可以实现会话共享。假设你在和朋友在不同地点以相同用户登录一台机器，然后你创建一个screen会话，你朋友可以在他的终端上命令：


{% highlight bash %}
{% raw %}

[root@TS-DEV ~]# screen -x

{% endraw %}
{% endhighlight %}


这个命令会将你朋友的终端Attach到你的Screen会话上，并且你的终端不会被Detach。这样你就可以和朋友共享同一个会话了，如果你们当前又处于同一个窗口，那就相当于坐在同一个显示器前面，你的操作会同步演示给你朋友，你朋友的操作也会同步演示给你。当然，如果你们切换到这个会话的不同窗口中去，那还是可以分别进行不同的操作的。

2.会话锁定与解锁

Screen允许使用快捷键`C-a s`锁定会话。锁定以后，再进行任何输入屏幕都不会再有反应了。但是要注意虽然屏幕上看不到反应，但你的输入都会被Screen中的进程接收到。快捷键C-a q可以解锁一个会话。

也可以使用`C-a x`锁定会话，不同的是这样锁定之后，会话会被Screen所属用户的密码保护，需要输入密码才能继续访问这个会话。

3.发送命令到screen会话

在Screen会话之外，可以通过screen命令操作一个Screen会话，这也为使用Screen作为脚本程序增加了便利。关于Screen在脚本中的应用超出了入门的范围，这里只看一个例子，体会一下在会话之外对Screen的操作：

{% highlight bash %}
{% raw %}

[root@TS-DEV ~]# screen -S test -X screen ping www.baidu.com

{% endraw %}
{% endhighlight %}


这个命令在一个叫做sandy的screen会话中创建一个新窗口，并在其中运行ping命令。

4.屏幕分割

现在显示器那么大，将一个屏幕分割成不同区域显示不同的Screen窗口显然是个很酷的事情。可以使用快捷键`C-a S`将显示器水平分割，Screen 4.00.03版本以后，也支持垂直分屏，快捷键是`C-a |`。分屏以后，可以使用`C-a <tab>`在各个区块间切换，每一区块上都可以创建窗口并在其中运行进程。

可以用`C-a X`快捷键关闭当前焦点所在的屏幕区块，也可以用`C-a Q`关闭除当前区块之外其他的所有区块。关闭的区块中的窗口并不会关闭，还可以通过窗口切换找到它。

5.C/P模式和操作

screen的另一个很强大的功能就是可以在不同窗口之间进行复制粘贴了。使用快捷键C-a <Esc>或者`C-a `[可以进入copy/paste模式，这个模式下可以像在vi中一样移动光标，并可以使用空格键设置标记。其实在这个模式下有很多类似vi的操作，譬如使用/进行搜索，使用y快速标记一行，使用w快速标记一个单词等。关于C/P模式下的高级操作，其文档的这一部分有比较详细的说明。

一般情况下，可以移动光标到指定位置，按下空格设置一个开头标记，然后移动光标到结尾位置，按下空格设置第二个标记，同时会将两个标记之间的部分储存在copy/paste buffer中，并退出copy/paste模式。在正常模式下，可以使用快捷键C-a ]将储存在buffer中的内容粘贴到当前窗口。


----------


# 然后说一下Tmux这个软件 #

## 安装Tmux ##

1.centos下安装

由于没法使用yum安装，所以我这里就说源码包安装了

{% highlight bash %}
{% raw %}

wget http://downloads.sourceforge.net/tmux/tmux-1.8.tar.gz
tar -zxvf tmux-1.8.tar.gz
cd tmux-1.8
./configure --prefix=/usr/local/tmux
make
make install

{% endraw %}
{% endhighlight %}

2.ubuntu下安装


{% highlight bash %}
{% raw %}

apt-get install tmux

{% endraw %}
{% endhighlight %}

## 语法介绍 ##

{% highlight bash %}
{% raw %}

tmux [-28lquvV] [-c shell-command] [-f file] [-L socket-name] [-S socket-path] [command [flags]]

{% endraw %}
{% endhighlight %}

    tmux ls 查看有几个可以复用的窗口
    tmux a -t name  进入到name的窗口 

快捷键参数

tmux所有自带命令都默认需要先按`Ctrl + b`，然后再键入对应的命令
    
    Ctrl+b " --- split pane horizontally
    Ctrl+b % --- 将当前窗格垂直划分
    Ctrl+b 方向键 --- 在各窗格间切换
    Ctrl+b，并且不要松开Ctrl，方向键 --- 调整窗格大小
    Ctrl+b c --- (c)reate 生成一个新的窗口
    Ctrl+b n --- (n)ext 移动到下一个窗口
    Ctrl+b p --- (p)revious 移动到前一个窗口.
    Ctrl+b 空格键 --- 采用下一个内置布局 
    Ctrl+b q --- 显示分隔窗口的编号 
    Ctrl+b o --- 跳到下一个分隔窗口 
    Ctrl+b & --- 确认后退出 tmux 
    
这几个命令都试几遍，这个工具基本上也就算上手了，简单才是最重要的。

这个支持多布局，可以调出很多界面，写一个脚本可以使打开tmux就是你想要的布局界面，内容如下：

{% highlight bash %}
{% raw %}

vim ~/.tmux/mylayout
    
selectp -t 0   #选中第0个窗格
splitw -h -p 50  #将其分成左右两个
selectp -t 1   #选中第一个，也就是右边那个
splitw -v -p 50   #将其分成上下两个，这样就变成了图中的布局了
selectp -t 0   #选回第一个

{% endraw %}
{% endhighlight %}

之后在`.tmux.conf`后面加上一句`bind D source-file ~/.tmux/mylayout`指定你配置的脚本位置,*注意：这个`D`你自己设置成想要的快捷键，不要冲突就好。*

这样每次进入tmux后，键入 `Ctrl + b D`，即会自动执行mylayout脚本，生成图示布局。如果 `.tmux.conf `文件不存在的话，请自己生成。注意前面有个`.`

*    附加：
    
    通过在.tmux.conf中添加相应的命令打开对应的功能即可：

    鼠标可以选中窗格  set-option -g mouse-select-pane on

    鼠标滚轮可以用    set-window-option -g mode-mouse on

