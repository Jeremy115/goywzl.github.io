---
layout: article
title:  "Nginx+Doxygen+Graphviz实现源代码网页访问"
date:   2016-01-10 09:20:00 +0800
categories: server
---

开发人员写代码，咱们一个一个看，肯定一头雾水，找批注也是蒙圈的干活！

那么有心人士就发明了一个把代码进行文件产生的工具，可以以html等形式访问

首先介绍一下

    Nginx：都懂得，http服务器啊
    Doxygen：是一个程序的文件产生工具，可将程序中的特定批注转换成为说明文件
    Graphviz:是AT Labs-Research开发的图形绘制工具
    Dot:是一种文本图形描述语言

首先：进行安装，这里我用的Ubuntu系统安装，当然Centos也一样啦

安装nginx，使用`sudo apt-get install nginx` 安装成功（需要配置）

安装Doxygen，使用`sudo apt-get install doxygen`安装成功（需要配置）

安装Graphviz、使用`sudo apt-get install graphviz` 安装陈功（安装即可）

安装dot，使用`sudo apt-get install dot` 安装成功（安装即可）
安装完成就可以进行配置了

执行`doxygen –g test.conf` 得到配置文件

编辑配置文件，更改了以下内容
 
	PROJECT_NAME               Project 的名字，以一个单字为主，多个单字请使用双引号括住。
	PROJECT_NUMBER              Project的版本号码。
	OUTPUT_DIRECTORY            输出路径。产生的文件会放在这个路径之下。如果没有填，将会以目前所在路径来作为输出路径
	OUTPUT_LANGUAGE            输出语言。预设为English。我改的是Chinese
	INPUT                   指定加载或找寻要处理的程序代码档案路径。
	RECURSIVE                设定为YES时，INPUT所指定目录的所有子目录都会被处理。
	SOURCE_BROWSER             设定为YES，则Doxygen会产生出源文件的列表，以供查阅。
	INLINE_SOURCES             设定为YES ，则程序代码也会被嵌入于说明文件中。
	ALPHABETICAL_INDEX          设定为YES，则一个依照字母排序的列表会加入在产生的文件中。
	GENERATE_HTML              设定为YES ，就会产生HTML版本的说明文件。
	GENERATE_TREEVIEW           设定为YES，Doxygen会帮您产生一个树状结构，在画面左侧。

上面都是直接更改的，这里添加一行

	set FILE_PATTERNS = *.*       表示这个目录下所有的文件被转换

执行`doxygen test.conf`就会执行这个配置文件，并且把源文件进行显示，会生成html文件夹与latex文件夹，我们只要html

然后用nginx设置访问html文件即可，后期做了一个域名绑定，一个用户名验证。