---
layout: post
title: "开博第一篇:部署Typo到Ｈeroku上"
date: 2012-01-05 20:00
comments: true
categories: [Web]
---
最近一直在折腾自己的私人博客，花了几天时间在自己机器上把[Typo](https://github.com/fdv/typo)跑起来了，尝试了一下，还挺不错。不过，博客放在自己机器上别人就没法访问就变成了日记，这可不是我的本意。于是开始研究怎么把博客放到公网上。放狗一搜，支持RoR的免费虚拟主机[Heroku](http://www.heroku.com")做的很不错，就它了。

本以为很容易就可以搞定的事，结果折腾了好几天。由于Heroku的只读文件系统限制，刚开始Typo一上传上去就崩溃，根本没法运行，搞的我差点想放弃。怎么办呢，问题总有解决的办法，继续放狗吧。不得不说，狗是人类的好基友啊，作为天朝子民，忍不住要吐槽，擦，擦，擦。言归正传，最后搜到一篇[博文](https://huydinh.eu/2011/07/22/typo-on-heroku")，按照它的说明，总算搞定了。记录一下将Typo部署到heroku的完整过程。
<!--more-->
1. 获取源码  
下载最新的Typo的[源码](https://github.com/fdv/typo.git)
        git clone https://github.com/fdv/typo.git
    
2. 在config/目录下新建文件preinitializer.rb，文件内容如下（防止写文件）
        require fileutils
        file_utils_no_write = FileUtils::NoWrite
        Object.send :remove_const, :FileUtils
        FileUtils = file_utils_no_write

3. 修改`vendor/plugins/easy-ckeditor/init.rb`，注释掉两行代码（*这个很重要，防止ckeditor写文件造成网站崩溃*）
        require ckeditor_file_utils
        #CkeditorFileUtils.check_and_install

4. 修改config/enviroments/production.rb文件（之前存在侥幸心理，没有修改这一行，事实证明是不行的）
        config.action_controller.perform_caching = false

5. 修改数据库配置文件config/database.yml.example，重命名为database.yml。
可以在配置文件中指定要连接的数据库，默认是typo_dev，指定方法：database：yourdatabase
Typo默认使用mysql作为本地数据库，heroku默认使用postgresql作为服务端数据库，可以同步，*但会出现中文乱码导致网站崩溃*。  
解决方法：换用`mysql2(0.2.18)` gem（在Gemfile中修改）  
先安装taps：
        gem install taps
上传：
        heroku db:push mysql://localdb_username:localdb_passwd@localhost/localdb_name?encoding=utf8
下载：
        heroku db:pull mysql://localdb_username:localdb_passwd@localhost/localdb_name?encoding=utf8

6. 修改Gemfile  
Typo使用了一个比较灵活的Gemfile，可以在多个数据库adapter中选择一个，同时还支持debug。这些我都不需要，同时ruby-debug19在rvm下也无法编译通过，缺少头文件，所以我简化了一下Gemfile
主要修改如下：  
修改source：`source http://ruby.taobao.org`  
添加gem：`gem mysql2,0.2.18`，解决中文乱码导致网站崩溃问题，最新的mysql2是配合rails3.1.x的，3.0.x只能使用0.2.x，得从rubygems上下载  
修改gem：`gem factory_girl`，去掉指定版本，taobao中无法下载老版本的gem包  
删除ruby-debug19，缺少头文件，无法编译过  
执行`bundle install`，生成Gemfile.lock文件  

7. 执行本地数据迁移任务  
`rake db:migrate RAILS_ENV=production`  
生成了三个文件（这个不重要，我是一股脑儿把所有文件都加入git库了）
> db/schema.rb  
> log/production.log  
> public/javascripts/ckeditor/config.js

8. 创建本地git版本库,将所有文件都添加到版本库中，特别注意有没有public/files/和tmp/cache/文件夹。这个好像与之前部署失败有关，最好在这两个文件夹下面都建立一个.gitkeep文件（*与崩溃有关系*）
        mkdir -p public/files tmp/cache
        touch public/files/.gitkeep tmp/cache/.gitkeep
        git add -f public/files/.gitkeep tmp/cache/.gitkeep

9. 部署到Heroku上  
        heroku create yourwebname
        git push heroku master
        heroku rake db:migrate
执行到最后一个数据迁移任务103_really_update_existing_articles_with_nil_or_empty_post_type.rb时会失败，这个好像没啥影响

10. 大功告成，赶快浏览下吧  
        heroku open
或者手动输入URL: http://yourwebname.heroku.com
