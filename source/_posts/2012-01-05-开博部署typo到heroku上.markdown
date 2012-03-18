---
layout: post
title: !binary |-
  5byA5Y2a56ys5LiA56+H77ya6YOo572yVHlwb+WIsEhlcm9rdeS4ig==
typo_id: 3
---
<p>
	&nbsp;&nbsp;&nbsp; 最近一直在折腾自己的私人博客，花了几天时间在自己机器上把<a href="https://github.com/fdv/typo">Typo</a>跑起来了，尝试了一下，还挺不错。不过，博客放在自己机器上别人就没法访问就变成了日记，这可不是我的本意。于是开始研究怎么把博客放到公网上。放狗一搜，支持RoR的免费虚拟主机<a href="http://www.heroku.com">Heroku</a>做的很不错，就它了。<br />
	&nbsp;&nbsp;&nbsp;&nbsp; 本以为很容易就可以搞定的事，结果折腾了好几天。由于Heroku的只读文件系统限制，刚开始Typo一上传上去就崩溃，根本没法运行，搞的我差点想放弃。怎么办呢，问题总有解决的办法，继续放狗吧。不得不说，狗是人类的好基友啊，作为天朝子民，忍不住要吐槽，擦，擦，擦。言归正传，最后搜到一篇<a href="https://huydinh.eu/2011/07/22/typo-on-heroku">博文</a>，按照它的说明，总算搞定了。记录一下将Typo部署到heroku的完整过程。</p>
<p>
	1）下载最新的Typo的<a href="https://github.com/fdv/typo.git">源码</a>，拷贝到一个目录中(eg. blog/)，可以使用git clone下来:<br />
	<strong>git clone https://github.com/fdv/typo.git</strong></p>
<p>
	<br />
	2）在blog/config/目录下新建文件<strong>preinitializer.rb</strong>，文件内容如下（<strong>防止写文件</strong>）：<br />
	<strong>require &lsquo;fileutils&rsquo;<br />
	file_utils_no_write = FileUtils::NoWrite<br />
	Object.send :remove_const, :FileUtils<br />
	FileUtils = file_utils_no_write</strong></p>
<p>
	<br />
	3）修改blog/vendor/plugins/easy-ckeditor/init.rb，注释掉两行代码（<strong>这个很重要，防止ckeditor写文件造成网站崩溃</strong>）<br />
	<strong>#require &#39;ckeditor_file_utils&#39;<br />
	#CkeditorFileUtils.check_and_install</strong></p>
<p>
	&nbsp;</p>
<p>
	4）修改blog/config/enviroments/production.rb文件（之前存在侥幸心理，没有修改这一行，事实证明是不行的）<br />
	<strong>config.action_controller.perform_caching = false</strong></p>
<p>
	<br />
	5）修改数据库配置文件blog/config/database.yml.example，重命名为database.yml。<br />
	可以在配置文件中指定要连接的数据库，默认是typo_dev，指定方法：database：yourdatabase<br />
	Typo默认使用mysql作为本地数据库，heroku默认使用postgresql作为服务端数据库，可以同步，<strong>但会出现中文乱码导致网站崩溃</strong>。<br />
	解决方法：换用mysql2(0.2.18) gem（在Gemfile中修改）<br />
	先安装taps：<strong>gem install taps</strong><br />
	上传：<strong>heroku db:push mysql://localdb_username:localdb_passwd@localhost/localdb_name?encoding=utf8</strong><br />
	下载：<strong>heroku db:pull mysql://localdb_username:localdb_passwd@localhost/localdb_name?encoding=utf8</strong></p>
<p>
	<br />
	6）修改Gemfile<br />
	Typo使用了一个比较灵活的Gemfile，可以在多个数据库adapter中选择一个，同时还支持debug。这些我都不需要，同时ruby-debug19在rvm下也无法编译通过，缺少头文件，所以我简化了一下Gemfile<br />
	主要修改如下：<br />
	修改source：<strong>source &rdquo;http://ruby.taobao.org&quot;</strong><br />
	添加gem：<strong>gem &#39;mysql2&#39;,&#39;0.2.18</strong>&#39;，解决中文乱码导致网站崩溃问题，最新的mysql2是配合rails3.1.x的，3.0.x只能使用0.2.x，得从rubygems上下载<br />
	修改gem：<strong>gem &#39;factory_girl&#39;</strong>，去掉指定版本，taobao中无法下载老版本的gem包<br />
	删除ruby-debug19，缺少头文件，无法编译过<br />
	执行<strong>bundle install</strong>，生成Gemfile.lock文件</p>
<p>
	<br />
	7）执行本地数据迁移任务<br />
	<strong>rake db:migrate RAILS_ENV=production</strong>，生成了三个文件（这个不重要，我是一股脑儿把所有文件都加入git库了）<br />
	#&nbsp;&nbsp;&nbsp; db/schema.rb<br />
	#&nbsp;&nbsp;&nbsp; log/production.log<br />
	#&nbsp;&nbsp;&nbsp; public/javascripts/ckeditor/config.js</p>
<p>
	<br />
	8）创建本地git版本库,将所有文件都添加到版本库中，特别注意有没有public/files/和tmp/cache/文件夹。这个好像与之前部署失败有关，最好在这两个文件夹下面都建立一个.gitkeep文件（<strong>与崩溃有关系</strong>）<br />
	<strong>mkdir -p public/files tmp/cache<br />
	touch public/files/.gitkeep tmp/cache/.gitkeep<br />
	git add -f public/files/.gitkeep tmp/cache/.gitkeep</strong><br />
	<br />
	9）部署到Heroku上<br />
	<strong>heroku create yourwebname<br />
	git push heroku master<br />
	heroku rake db:migrate</strong><br />
	执行到最后一个数据迁移任务103_really_update_existing_articles_with_nil_or_empty_post_type.rb时会失败，这个好像没啥影响</p>
<p>
	<br />
	10）大功告成，赶快浏览下吧<br />
	<strong>heroku open</strong> 或者手动输入URL：http://yourwebname.heroku.com</p>
