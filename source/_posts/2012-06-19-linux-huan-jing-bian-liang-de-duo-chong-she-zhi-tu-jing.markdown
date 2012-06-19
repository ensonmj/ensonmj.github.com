---
layout: post
title: "Linux 环境变量的多种设置途径"
date: 2012-06-19 20:15
comments: true
categories: [Linux]
---
上一篇提到有空写写Linux设置环境变量的方法，没想到一眨眼就是一周，怎一个懒字了得。

其实这篇文章主要翻译自一篇[英文blog][1]，有喜欢阅读英文资料的可以前去一观。

### 会话级环境变量
会话级的环境变量只影响特定的登录用户，用户主目录下有几个隐藏文件可以用于进行此设置。

* `~/.profile` - 这个脚本在启动DM（Display Manager）时被自动加载；如果你使用控制台方式登录，它也会被加载，其实确切的说，只要是使用** login shell **，此文件就会被加载。
<!--more-->
* `~/.bash_profile`或者`~/.bash_login` - 这两个文件中只要有一个存在，启动login shell的时候就会加载这两个文件之一，而不会加载`~/.profile`，如果两个都存在，bash优先加载`~/.bash_profile`；这两个文件对图形环境不会有影响，DM启动时不会加载。  
其实一般情况下`～/.bash_profile`的内容只有一行，就是加载下面要讲到的`~/.bashrc`文件。
    
    $cat ~/.bash_profile  
    . $HOME/.bashrc

* `~/.bashrc` - 这个文件一般使用比较广泛，也比较方便。每次执行bash时都会加载一次此文件（非login shell），因此有点小小的冗余——变量会被多次设置，不过这不是什么大问题；这个文件在启动图形界面时也会被加载。

我个人的设置是`~/.bashrc`搞定图形界面和非login shell，`~/.bash_profile`搞定login shell。

### 系统级环境变量
系统级的环境变量则会影响到所有使用这台机器的用户，相关配置文件都在`/etc`目录下。

* `/etc/profile` - 这个文件在每次启动login shell或者DM时执行，系统默认其实只做了很少一部分设置，其他设置还是加载`/etc/bash.bashrc`来完成的。

* `/etc/bash.bashrc` - 这是`~/.bashrc`文件的系统级版本，作用基本类似。

* `/etc/environment` - 这个有别于上面提到的所有文件，它不是一个脚本文件，仅仅是一个包含“键值对”的配置文件，这个文件一般用来设置系统级的`locale`和`path`。上一篇中使用`systemd`替换`sysVinit`的过程中，就用了此文件来设置系统级的locale。

[1]: https://numberformat.wordpress.com/2010/01/24/setting-environment-variables-in-ubuntu/