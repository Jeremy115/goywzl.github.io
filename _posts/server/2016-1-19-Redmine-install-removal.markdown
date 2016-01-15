---
layout: article
title:  "Redmine install and removal"
date:   2016-1-19 09:20:00 +0800
categories: server
---

公司有一台redmine，不过版本太老了，很多功能都实现不了

领导让我升级一下，然后我就找升级方法，但是毫无头绪,我完全不知道这台机器是怎么安装的redmine，也有源码报，但是文件都不对，让我煞是费解

最后百般无奈，我只好进行迁移的想法，顺便学习一下搭建,以为很简单，开始做了之后发现，真的是各种问题各种出啊

下面就是我的安装方法，很详细了，为我日后查看使用

    日期：2015-7-16
    环境：ubuntu14.04-server
    软件包：redmine-2.6.6.tar.gz
 

----------

# 首要任务就是下载软件包 #

地址：http://www.redmine.org/projects/redmine/wiki/Download
 
安装前需要准备redmine环境

# 首先安装ruby #

查看下表
<table>
<tr><td>Redmine version</td> <td>Supported Ruby versions</td> <td>Rails version used</td></tr>
<tr><td>current trunk</td> <td>	ruby 1.9.33, 2.0.02, 2.1, 2.21</td> <td>	Rails 4.20</td></tr>
<tr><td>3.0</td> <td>ruby 1.9.33, 2.0.02, 2.1, 2.21</td> <td>Rails 4.20</td></tr>
<tr><td>2.6</td> <td>	ruby 1.8.74, 1.9.2, 1.9.33, 2.0.02, 2.1, 2.2, jruby-1.7.6</td> <td>Rails 3.2</td></tr>
</table>

查看上表的版本支持,别看错了

查找以下我这可以安装的ruby，ruby1.9.1可以支持redmine2.6，我就安装1.9.1了

{% highlight bash %}
{% raw %}

apt-get install ruby1.9.1

{% endraw %}
{% endhighlight %}

安装下面gem所依赖的包

{% highlight bash %}
{% raw %}

apt-cache search ruby
apt-get install ruby1.9.1-dev
apt-get install ruby-dev
apt-get install make
aptitude install libmysqlclient-dev
apt-get install libmagickwand-dev
apt-get install build-essential
apt-get install imagemagick libmagickwand-dev
 
{% endraw %}
{% endhighlight %}

----------

# 下面就是安装数据库 #

redmine支持很多种数据库，不过我就会mysql，所以我就是用mysql了

{% highlight bash %}
{% raw %}

apt-get install mysql-server mysql-client
apt-get install php5-mysql
start mysql
 
{% endraw %}
{% endhighlight %}

安装完启动后，msyql需要做一些少许的调试：

{% highlight bash %}
{% raw %}

mysql -u root –p
 
mysql> show databases;
+——————–+
| Database           |
+——————–+
| information_schema |
| mysql              |
| performance_schema |
+——————–+
3 rows in set (0.00 sec)
 
mysql> CREATE DATABASE redmine CHARACTER SET utf8;
Query OK, 1 row affected (0.00 sec)
 
mysql> CREATE USER ‘redmine’@’localhost’ IDENTIFIED BY ‘password’;
Query OK, 0 rows affected (0.00 sec)
 
mysql> GRANT ALL PRIVILEGES ON redmine.* TO ‘redmine’@’localhost’;
Query OK, 0 rows affected (0.00 sec)
 
mysql> show databases;
+——————–+
| Database           |
+——————–+
| information_schema |
| mysql              |
| performance_schema |
| redmine            |
+——————–+
4 rows in set (0.01 sec)
  
{% endraw %}
{% endhighlight %}

至此，mysql数据库的操作就基本完成了。

----------

# 下面配置redmine #

{% highlight bash %}
{% raw %}

root@Redmine:/redmine-3.0.4# cp config/database.yml.example config/database.yml
  
{% endraw %}
{% endhighlight %}

编辑database.yml修改文件，这是建立redmine链接数据库的文件，如果mysql运行的端口号不是3306，被你更改了，你要加上`port: 3307`这个参数

{% highlight bash %}
{% raw %}

production:
adapter: mysql2
database: redmine
host: localhost
username: redmine
password: password
   
{% endraw %}
{% endhighlight %}

现在安装一些依赖包

{% highlight bash %}
{% raw %}

gem install bundler
bundle install –without development test
  
{% endraw %}
{% endhighlight %}

在执行上条命令时，有可能会报错，报错了，就证明你应该是跟我一个国家的，我这里有两个方法，不知道适合不适合你。

    1.更改redmine目录下的Gemfile文件，把`source 'https://rubygems.org'`更改为`source "https://ruby.taobao.org"`，然后在执行上面命令，这个方法是改变你的源地址，因为上面的地址你访问不了，就指向淘宝了
    2.这个方法就比较笨了，就是他提示缺少什么，就安装什么，很漫长。

 
安装这个出错的话执行

{% highlight bash %}
{% raw %}

apt-get install imagemagick libmagickwand-dev
  
{% endraw %}
{% endhighlight %}

我安装的是mysql所以需要执行下面的命令

{% highlight bash %}
{% raw %}

bundle exec rake generate_secret_token
  
{% endraw %}
{% endhighlight %}

创建数据库结构

{% highlight bash %}
{% raw %}

RAILS_ENV=production bundle exec rake db:migrate
  
{% endraw %}
{% endhighlight %}

插入默认数据库数据

{% highlight bash %}
{% raw %}

RAILS_ENV=production bundle exec rake redmine:load_default_data
  
{% endraw %}
{% endhighlight %}

测试安装运行WEBrick web服务器:

{% highlight bash %}
{% raw %}

bundle exec ruby script/rails server webrick -e production
  
{% endraw %}
{% endhighlight %}

使用http://127.0.0.1:3000访问可以测试redmine安装情况

测试之后，需要配置config文件

把这个文件config/configuration.yml.example复制到config/configuration.yml后

更改attachments_storage_path: /var/redmine/files

我只是更改了上传文件的路径，其他的自行参考

执行下面命令，把redmine当成一个服务启动

{% highlight bash %}
{% raw %}

root@Redmine:/var/redmine# ruby script/rails server -e production -d
=> Booting WEBrick
=> Rails 3.2.22 application starting in production on http://0.0.0.0:3000
  
{% endraw %}
{% endhighlight %}

访问：http://127.0.0.1:3000即可访问
 

----------

# 如果你要进行数据迁移 #

那么，只需要把老版本的config文件拷贝到新的redmine目录下

然后备份老版本的mysql数据库，恢复到新的redmine上

别忘了给数据库权限

创建数据库与授权用户

{% highlight bash %}
{% raw %}

CREATE DATABASE redmine CHARACTER SET utf8;
CREATE USER ‘redmine’@’localhost’ IDENTIFIED BY ‘my_password’;
GRANT ALL PRIVILEGES ON redmine.* TO ‘redmine’@’localhost’;
  
{% endraw %}
{% endhighlight %}

最后执行rake db:migrate RAILS_ENV=production这个命令，把老数据库类型更新成新数据库类型

还有files文件全部拷贝到新的redmine目录下

之后就可以重启服务了
 
*小提醒*：如果你发现每次输入地址http://127.0.0.1:3000都要加上3000，让你很不舒服，那么，你可以把启动命令改成`ruby script/rails server –p80 -e production –d`那就是80端口启动了