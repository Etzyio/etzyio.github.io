---
layout: post
title:  "archlinux安装i3wm"
date:   2022-09-21 18:30:36 +0800
tags:   [linux, 桌面环境]
categories: shell脚本
---
# 安装archlinux
这个很简单从官网或源(https://mirrors.cnnic.cn/archlinux/iso/)下载最新的官方镜像，将其写入u盘中，启动后执行archinstall，将配置修改为自己所需的内容即可，最后进行安装非常方便。(https://wiki.archlinux.org/title/Archinstall_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))。
使用archlinux也是为了方便安装使用各种软件。

# i3wm
i3wm一款动态的平铺窗口管理器，对于我来说可以尽量少的使用鼠标，将所有操作放在键盘上。方便以各种方式使用电脑。键位可以自定义配置想进行的操作，而且资源占用低对于很多配置较低的电脑也能流畅使用。

首先进行安装，运行`sudo pacman -S i3wm`会显示以下情况，可以选择全部进行安装，我下方也会介绍代替i3status的其他软件。
![安装i3wm](/assets/images/安装i3wm/DeepinScreenshot_select-area_20220921184800.png)

也需要安装一些其他的必须软件，例如\
sddm # 桌面环境登录管理器\
xorg # X窗口系统的开源实现
xfce4-terminal # 终端\
bolybar # 可以替换i3-status的状态栏\
xss-lock # 自动锁屏\
rofi # 窗口切换器\
fcitx5 # 输入法\
deepin-screenshot # deepin截图\
compton # 启用窗口淡入淡出和透明度等效果



