---
layout: post
title: "git历史搜索的各种方法"
date: 2012-05-23 20:09
comments: true
categories: 
---
* 搜索提交日志

    git log --grep

示例
> $git log --oneline  
> 939439b Update arch hdmi config  
> bda8277 add disqus  
> a128978 write a blog about hdmi audio in arch  
> 9ae3805 modify path for articles

> $git log --grep "arch hdmi"  
> commit 939439b3b011f05c7e86ab7efd38c67a6da3f4ad  
> Author: ensonmj <ensonmj@gmail.com>  
> Date:   Mon May 14 20:49:32 2012 +0800  
> 
>    Update arch hdmi config


* 搜索变动

    git log -S< string >  
    git log -G< regex >

示例
> $git log -G"Arch HDMI"  
> commit 939439b3b011f05c7e86ab7efd38c67a6da3f4ad  
> Author: ensonmj <ensonmj@gmail.com>  
> Date:   Mon May 14 20:49:32 2012 +0800  
> 
>    Update arch hdmi config

> commit a1289784169fcba713c2fa5ad7a860f70e10f289  
> Author: ensonmj <ensonmj@gmail.com>  
> Date:   Fri May 4 23:02:56 2012 +0800  
> 
>    write a blog about hdmi audio in arch

* 搜索工作区

    git grep

示例
> $git grep "Arch HDMI"  
> source/_posts/2012-05-04-arch-hdmi-sheng-yin-shu-chu.markdown:title: "Arch HDMI 声音输出"  
> source/_posts/2012-05-14-arch-hdmisheng-yin-shu-chu-(xu-).markdown:title: "Arch HDMI声音输出（续）"