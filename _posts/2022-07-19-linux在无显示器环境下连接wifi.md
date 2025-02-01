---
layout: post
title:  "linux在无显示器情况下连接wifi"
date:   2022-07-19 14:27:49 +0800
tags:   [linux, 终端]
categories: linux
---

## linux在无显示器情况下，通过另一台linux挂载硬盘设置连接wifi

首先将linux硬盘挂载到一个目录下，以本地 root 目录为例 mount /dev/sdX1 root ,运行

```wpa_passphrase "ssid" "pass"```

导出wifi名及密码的所需配置
将其写入到 root/etc/wpa_supplicant/wpa_supplicant-wlan0.conf 中，
然后运行

```ln -s /usr/lib/systemd/system/wpa_supplicant@.service root/etc/systemd/system/multi-user.target.wants/wpa_supplicant@wlan0.service```

当然如果获得ip还需要开启dhcp，或者固定ip，我这里偷懒直接将dhcpcd也一块软连接到这里

```ln -s /usr/lib/systemd/system/dhcpcd.service root/etc/systemd/system/multi-user.target.wants/dhcpcd.service```

这样启动linux后就能使其自动连接到wifi。

## 通过arch-chroot进入

安装```arch-install-scripts qemu-user-binfmt qemu-user-static libarchive-tools```等软件
运行命令```arch-chroot /mnt /bin/bash```/mnt为挂载路径，进入硬盘中的系统