---
layout: post
title: "Arch HDMI 声音输出"
date: 2012-05-04 22:36
comments: true
categories: 
---
最近又开始习惯性折腾了，这次折腾的对象是HDMI的音频输出。自从上次显卡的DVI接口导致电脑自动重启之后就开始使用HDMI接口连接显示器了。HDMI是可以同时传输视频和音频信号的，正好我的显示器也自带扬声器，可以充分利用起来。但理想很丰满，现实却很骨感，测试无法输出声音。
<!--more-->
按照Arch Wiki上有关[ALSA](https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture)和[PulseAudio](https://wiki.archlinux.org/index.php/PulseAudio)的说明跑了一遍，电脑上现有的声音输出设备如下：

    $aplay -l
    **** List of PLAYBACK Hardware Devices ****　
    card 0: NVidia [HDA NVidia], device 3: HDMI 0 > [HDMI 0]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 0: NVidia [HDA NVidia], device 7: HDMI 0 > [HDMI 0]
      Subdevices: 0/1
      Subdevice #0: subdevice #0
    card 0: NVidia [HDA NVidia], device 8: HDMI 0 > [HDMI 0]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 0: NVidia [HDA NVidia], device 9: HDMI 0 > [HDMI 0]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 1: PCH [HDA Intel PCH], device 0: ALC889 > Analog [ALC889 Analog]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
    card 1: PCH [HDA Intel PCH], device 1: ALC889 > Digital [ALC889 Digital]
      Subdevices: 1/1
      Subdevice #0: subdevice #0
然后测试哪个能输出声音，我的显卡是card 0,device 7能真正的输出声音。

    $aplay -D plughw:0,7 /usr/share/sounds/alsa/Noise.wav 
    Playing WAVE '/usr/share/sounds/alsa/Noise.wav' : Signed 16 bit Little Endian, Rate 48000 Hz, Mono

到这儿说明硬件都没有问题，但为什么默认不能使用呢？后来搜索到问题的原因：PulseAudio默认只会创建一个HDMI的输出设备,具体到我的电脑对应于card 0,device 3,所以无法听到声音。最终的解决办法是编辑`/etc/pulse/default.pa`文件，找到相应的加载模块的地方，添加一行：

    load-module module-alsa-sink device=hw:0,7
OK,大喇叭开始广播啦