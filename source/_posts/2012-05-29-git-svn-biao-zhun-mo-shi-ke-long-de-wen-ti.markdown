---
layout: post
title: "git-svn 标准模式克隆的问题"
date: 2012-05-29 22:04
comments: true
categories: [Git]
---
公司采用svn托管代码，个人比较偏向于使用更加灵活的Git。由于项目代码包括windows客户端，linux后台，以及web客户端，很自然的形成三个文件夹：

    project/Trunk/windows  
    project/Trunk/linux  
    project/Trunk/web
    project/Branches/V2/windows
    project/Branches/V2/linux
    project/Branches/V2/web
    ...
<!--more-->
本人目前主要负责windows客户端开发，对于其他两个文件夹下的代码不关心，所以以前克隆代码只克隆windows目录下的，一条命令搞定：

    git svn clone svn://server/repo/project/Trunk/windows

最近一直在阅读蒋鑫的《Git权威指南》,个人认为这应该算是国内关于Git写的做好的一本书了。书中也提到了`git-svn`克隆svn代码的一些细节。于是心血来潮，想照着书练习一下，练习对象自然就是项目代码了。

第一次我使用了这样一条命令：

    git svn clone svn://server/repo/project

克隆下载的代码组织方式很奇怪，看着很像svn的组织方式：

    project/Trunk/windows  
    project/Trunk/linux  
    project/Trunk/web
    project/Branches/V2/windows
    project/Branches/V2/linux
    project/Branches/V2/web
    ...

使用`git branch -a`查看，只有一个默认的`master`分支。

于是又尝试其他的方式：

    git svn clone -s svn://server/repo/project
    git svn clone svn://server/repo/project -T trunk -b branches -t tags

结果这两种写法的结果倒是很一致，就创建了个本地仓库，啥代码也没下下来。

最后采用下面的写法成功克隆了整个版本库（代码库大了，时间很长）:
    
    git svn clone svn://server/repo/project -T Trunk -b Branches -t Tags

对比一下就会发现区别仅在于大小写不一样。`git-svn`默认采用的小写字母开头的`trunk,branches,tags`，而公司代码库使用的是大写字母开头的`Trunk,Branches,Tags`，不知道svn关于这个是否有什么规定。