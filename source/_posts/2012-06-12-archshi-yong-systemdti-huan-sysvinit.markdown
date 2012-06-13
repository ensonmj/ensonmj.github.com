---
layout: post
title: "Arch使用systemd替换sysVinit"
date: 2012-06-12 20:34
comments: true
categories: [Linux]
---
忘了最早是怎么知道`systemd`的了，最近想起来折腾它还是因为无意中看到了Arch论坛上一篇[帖子][1]，帖子内容跨度了两年，对于技术贴来说，我第一次见到。要想把整个帖子内容都看完有点不现实，大体上整个帖子就是在讨论Arch中怎么用`systemd`替换现有的`sysVinit`。

`systemd`是一个新的init系统，有关它的详细介绍参见Lennart的[blog][2]，文章很长，推荐技术宅们详细阅读下，看完你会觉得受益匪浅，甚至也许你就会像我一样迫不及待的想要尝试一下了。

其实整个切换过程并不复杂，按照[Arch wiki][3]上一步一步来就行了，我这里只是记录下我自己的整个操作过程，算是Wiki的中文简化版吧。

### 安装`systemd` ###

    $sudo pacman -S systemd

Arch当前安装完会有一些提示，可以暂时不用管，后面会用到。

    ln -s '/usr/lib/systemd/system/getty@.service' '/etc/systemd/system/getty.target.wants/getty@tty1.service'
    :: Append 'init=/bin/systemd' to your kernel command line in your
       bootloader to replace sysvinit with systemd
<!--more-->
### 创建新的启动项以使用`systemd`管理整个系统 ###

说一下我当前系统的环境，我现在使用的是Arch x86_64

    $uname -a
    Linux ENSONMJ-PC **3.3.7-1-ARCH** #1 SMP PREEMPT Tue May 22 00:26:26 CEST 2012 x86_64 GNU/Linux

bootloader使用的是Grub，不是Grub2，根分区在第一块硬盘的第三个分区上，所以启动项的配置是修改`/boot/grub/menu.lst`文件，添加如下启动条目（参数因人而异，不能照抄）：

    # (0) Arch Linux
    title  Arch Linux [systemd]
    root   (hd0,**2**)
    kernel /boot/vmlinuz-linux root=/dev/sda**3** vga=795 ro **init=/bin/systemd**
    initrd /boot/initramfs-linux.img

### 配置系统以使用native的systemd ###

这部分就是按照Wiki的说明按部就班的来：

1）hostname 设置

编辑`/etc/hostname`文件，里面写上你的hostname
    YOUR_HOSTNAME

2）console and keymap 设置

编辑`/etc/vconsole.conf`文件, 内容如下：
    KEYMAP="us"
    FONT=lat9w-16 #rc.conf中留空了
    FONT_MAP=8859-1_to_uni #rc.conf中留空了

其实`FONT`和`FONT_MAP`在原来`/etc/rc.conf`中我都留空了，没有设置值，这里只是照抄了wiki，效果不好可以后续再调整；关KEYMAP的介绍可以参见[Arch wiki][4]。

3）locale 设置

编辑`/etc/locale.conf`文件，内容如下：

    LANG="en_US.UTF-8"
    LC_COLLATE="C"

4) timezone设置

编辑`/etc/timezone`文件，内容如下：

    Asia/Shanghai

注意：`systemd`不支持`localtime`，如果之前使用的是`localtime`（装双系统出于兼容windows的可能会这么设置），那么需要改为`utc`:

    hwclock --systohc --utc

5) 配置内核模块

如果需要在启动时加载内核模块，需要在`/etc/modules-load.d/`下创建相应的文件，我只用了个`kvm-intel`模块，某次鼓捣`kvm`的遗留产物，于是创建`/etc/modules-load.d/kvm.conf`文件，内容如下：

    kvm-intel

其实文件名是可以任意取的，只要内容正确就行。同时还可以配置内核模块黑名单，有兴趣的自己研究，[参考链接][5]，我暂不需要这功能。

6）临时文件

可以用来定义哪些文件该何时清理，这个未配置，有空再研究。

7）日志

从Version 38开始，`systemd`开始有了自带的日志系统，默认日志写入`/run/systemd/journal/`目录下,重启后会消失；如果需要非易失的日志,可以创建`/var/log/journal/`目录。另外也可以考虑将`systemd`的日志重定向到`syslog`中，不感兴趣。

8）网络

我适用NetWorkManager配置网络，启用`NetworkManager.service`就行

    systemctl enable NetworkManager.service

9) 启用图形界面(默认）

`systemd`安装后，默认是适用图形界面的，此步可以省略了。

    $ls -l /usr/lib/systemd/system/default.target 
    lrwxrwxrwx 1 root root 16 Jun  5 04:12 /usr/lib/systemd/system/default.target -> graphical.target

10) 使用DE(Display Manager),具体对于我来说就是`gdm`

目前需要安装AUR中的`systemd-arch-units`包

    yaourt -S systemd-arch-units

然后启用`gdm.service`:

    systemctl enable gdm.service

如果在`/etc/locale.conf`文件中设置了locale，则为了DE使用同样的locale，需要在`/etc/environment`文件中
设置相应的系统级环境变量，其文件内容如下：

    LANG="en_US.utf8"

关于环境变量的多种设置方法，我准备在下一篇总结一下。

11）Daemon 设置

我原来`/etc/rc.conf`中`DAEMONS`数组设置如下：

    DAEMONS=(!hwclock syslog-ng dbus netfs @alsa @crond @networkmanager @sshd @sensors @libvirtd @dropboxd)

其中`hwclock`不使用；`syslog-ng`被`systemd`自带的`journald`取代，不用设置；`dbus`默认enable；`netfs`忘了我用来干嘛的了，暂不管；`alsa`好像也不需要特别设置；`crond`我一直没用，不设置；`networkmanager`已经设置；`sshd`家里电脑也一直用不上；最后三个`sensors`,`libvirtd`,`dropboxd`没发现有现成的service文件，需要自己创建，暂时搁置。

12）支持休眠

到目前位置，整个`systemd`就配置完了，重启之后就可以使用了。但是如果就这样结束的话，`gnome`中的`suspend`就无法使用，要想支持休眠还需要安装`systemd-sysvcompat`包。

    pacman -S systemd-sysvcompat

安装`systemd-sysvcompat`时会提示冲突，需要同时卸载`sysvint`和`initscripts`。

**另外，在安装过`systemd-sysvcompat`之后，就完全用`systemd`取代了`sysVinit`了，此时，grub中的内核启动参数`init=/bin/systemd`也可以去掉了。**

最后附一张我使用`systemd`后系统启动时序图，可以看到整个启动过程只用了9s，神速啊。时序图可以使用如下命令生成：

    systemd-analyze plog > boot.svg

![Boot with systemd in arch](/images/boot.svg "Boot with systemd in arch")

[1]: https://bbs.archlinux.org/viewtopic.php?id=96316
[2]: http://0pointer.de/blog/projects/systemd.html
[3]: https://wiki.archlinux.org/index.php/Systemd
[4]: https://wiki.archlinux.org/index.php/KEYMAP
[5]: https://wiki.archlinux.org/index.php/Systemd#Configure_kernel_modules_blacklist