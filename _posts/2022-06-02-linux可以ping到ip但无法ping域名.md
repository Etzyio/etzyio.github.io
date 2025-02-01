---
layout: post
title:  "linux可以ping到ip但无法ping域名"
date:   2022-06-02 14:26:49 +0800
tags:   [linux, error]
categories: linux
---

### 遇到域名无法被识别但是ping ip可以

修改 `/etc/systemd/resolved.com`添加`DNS=` 并加入dns服务器ip，

后重启`systemd-resolved` 

	`systemctl restart systemd-resolved`