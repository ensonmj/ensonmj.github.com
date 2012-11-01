---
layout: post
title: "OpenWrt简单配置"
date: 2012-11-01 22:12
comments: true
categories: [Linux]
---

1. 使用UUID挂载U盘或移动硬盘至/opt目录

2. 修改/etc/opkg.conf添加一行
    dest usb /opt/Packages

3. 修改/etc/profile添加

    if [ -d /opt/Packages ]; then   
      export USB=/opt/Packages   
      export PATH=$PATH:$USB/usr/bin:$USB/usr/sbin   
      export LD_LIBRARY_PATH=$USB/lib:$USB/usr/lib   
    fi
<!--more-->
4. 下载host文件,屏蔽广告

    #!/bin/sh 

    logger Downloading HostsX.orzhosts  
    if [ -s /tmp/HostsX.orzhosts ]; then  
      rm /tmp/HostsX.orzhosts  
    fi  
    wget -P /tmp http://hostsx.googlecode.com/svn/trunk/HostsX.orzhosts

    logger Downloading Smarthosts  
    if [ -s /tmp/Smart.hosts ]; then  
      rm /tmp/Smart.hosts  
    fi  
    wget http://smarthosts.googlecode.com/svn/trunk/hosts -O /tmp/Smart.hosts

5. 配置dnsmasq

    addn-hosts=/tmp/HostsX.orzhostsn  
    addn-hosts=/tmp/Smart.hosts

6. 配置Samba共享目录/opt

7. 配置miniDLNA共享目录

    media_dir=/opt
    db_dir=/opt/.minidlna(可选)

8. 安装ffmpeg支持miniDLNA共享各种格式的video

9. 安装python，python-openssl, pyopenssl至U盘

    opkg install -d usb python python-openssl pyopenssl(好像不需要这个）

注意，安装软件至U盘中后，某些启动脚本需要链接到相应位置，比如

    ln -s /opt/Package/etc/init.d/xx /etc/init.d/xx  
    ln -s /opt/Package/etc/config/yy /etc/config/yy

10. 安装coreutils-nohup

    opkg install coreutils-nohup

11. 运行goagent

  nohup python /opt/goagent-x.y.z/proxy.py >> /dev/null 2>&1 &

