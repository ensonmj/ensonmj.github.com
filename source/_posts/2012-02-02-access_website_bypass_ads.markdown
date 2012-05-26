---
layout: post
title: "加快网页加载速度方法之过滤广告"
date: 2012-02-02 20:00
comments: true
categories: [Web]
---
自从google将大部分业务搬离中国大陆之后，google的好多服务不是都要被GFW给强奸一下，造成无法访问或速度巨慢。
最近发现访问`newsmth.net`就出现了加载速度过慢的情况，刚开始懒得折腾，今天实在忍不了了。于是用firebug看了下，发现是`pagead2.googlesyndication.com/pagead/show_ads.js`这个地址加载比较慢。既然找到问题的关键了，那接下来就简单了，直接用`adblockplus`把这个地址屏蔽掉。
搞定之后，再刷新页面，以前嗖嗖的感觉又回来了。
