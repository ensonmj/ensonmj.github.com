---
layout: post
title: "git-svn更新svn地址"
date: 2012-05-31 19:33
comments: true
categories: [Git]
---
今天一个项目的svn地址发生了变化，如果本地使用svn客户端，比如tortoise-svn，可以直接relocate更新地址，但我一直使用git-svn管理项目代码，这个变换svn地址可就没那么容易了。

网上搜了下，git的[wiki][1]里面介绍了三种方法：第一种发送比较繁琐，但好像是使用最广泛的，因为好多搜索结果都是使用的此方法；第二种方法引自与[winterstream's blog][2]；第三种方法一看也比较繁琐，最终决定使用第二种方法。

详细阅读了下winterstream的方法，具体步骤可以概括为三步：
<!--more-->
1. 使用`git-filter-branch`修改所有提交消息中的旧地址为新地址（git-svn-id那行记录）
2. 删除.git/svn文件夹，下次使用`git svn rebase`更新代码时，会自动重建
3. 编辑.git/config中的`svn-remote.svn.url`指向新地址

其中第一步中`git-filter-branch`命令需要用到版本库所有的`refs`作为参数，winterstream使用的方法略显繁琐（优秀的程序员都是懒惰的，^_^)，其实可以有更简单的做法。

最终，我切换svn地址的整个操作流程如下：

1. 备份版本库，以防万一，毕竟是第一次尝试。

    cp -r local-proj local-proj-bak

2. 修改所有的提交消息（注意最后的参数）

    git filter-branch --msg-filter 'sed "/s/older-url/new-url/g"' **-- --all**

3. 编辑.git/config文件，替换[svn-remote "svn"]中的url的值为新地址，保存退出。

    git config -e

4. 删除.git/svn文件夹

    rm -rf .git/svn

5. 更新代码，此步会自动生产.git/svn文件夹，当然用户名、密码需要重新填写以下。

    git svn rebase

[1]: https://git.wiki.kernel.org/index.php/GitSvnSwitch
[2]: http://translate.org.za/blogs/wynand/en/content/changing-your-svn-repository-address-git-svn-setup