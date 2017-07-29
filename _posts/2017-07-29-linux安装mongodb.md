---
layout: post
title:  linux安装mongodb
date:   2017-07-29 14:08:00 +0800
categories: 环境搭建
tag: node.js
---

* content
{:toc}


====================================


一、下载安装包
------------------------------------
登录到官网 选择对应的版本进行下载，可以存储到任意文件夹，这里以**`linux-x86_64-ubuntu1404-3.4.6`**版本为例

官网地址：[https://www.mongodb.com/download-center?jmp=nav#community](https://www.mongodb.com/download-center?jmp=nav#community)

二、解压安装包
------------------------------------
进入到安装包的文件夹

使用`tar -zxvf`命令解压安装包

将解压好的文件复制到你想安装的mongodb文件夹下（这里是/usr/local/mongodb）

{% highlight bash %}
tar -zxvf mongodb-linux-x86_64-ubuntu1404-3.4.6.tgz

sudo mv mongodb-linux-x86_64-ubuntu1404-3.4.6 /usr/local/mongodb
{% endhighlight %}

三、修改环境变量
------------------------------------
MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 PATH 路径中：

{% highlight bash %}
export PATH=/usr/local/mongodb/bin:$PATH
{% endhighlight %}

如果只是修改当前用户的环境变量
{% highlight bash %}
vi ~/.bash_profile
export PATH=/usr/local/mongodb/bin:$PATH
:wq
{% endhighlight %}

查看环境变量是否添加成功
{% highlight bash %}
echo $PATH
{% endhighlight %}

四、创建数据存储目录
------------------------------------
MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。

以下实例中我们将data目录创建于根目录下(/)。

注意：/data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。
{% highlight bash %}
sudo mkdir -p /data/db
sudo chmod 777 data/ -R
{% endhighlight %}

五、运行MongoDB 服务
------------------------------------
运行MongoDB 服务
{% highlight bash %}
/usr/local/mongodb/bin/mongod
{% endhighlight %}

如果想在后台运行，则输入命令
{% highlight bash %}
/usr/local/mongodb/bin/mongod &
{% endhighlight %}

六、运行MongoDB的后台管理 Shell
------------------------------------
如果你需要进入MongoDB后台管理，你需要先打开mongodb装目录的下的bin目录，然后执行mongo命令文件。

MongoDB Shell是MongoDB自带的交互式Javascript shell,用来对MongoDB进行操作和管理的交互式环境。
{% highlight bash %}
/usr/local/mongodb/bin/mongo
{% endhighlight %}

接下来就可以在命令行中执行mongodb语法了

七、测试mongodb是否运行成功
------------------------------------
在浏览器输入

[http://192.168.1.100:27017/](http://192.168.1.100:27017/)（mogondb的服务器地址+端口号）

如果出现

**`it looks like you are trying to access MongoDB over HTTP on the native driver port.`**

这样的文字，说明mongdb已经运行成功
