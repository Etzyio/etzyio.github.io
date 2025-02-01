---
layout: post
title:  "openstack命令行创建虚拟机"
date:   2023-05-09 22:39:49 +0800
tags:   [linux, openstack]
categories: openstack
---

# openstack命令行创建虚拟机
```
nova flavor-create test_vir 10 1024 30 1创建虚拟机模板按顺序，名称、id、内存、硬盘、cpu核心数
nova flavor-list 查看虚拟机模板
neutron net-list 查看当前已有的网络配置
glance image-list 查看当前镜像状态
nova boot --flavor 10 --image  e57fe8cc-6cbd-4659-b2fc-ffe85035253e --nic net-id=f8211619-c758-4d68-b271-52e59cf1a60f virmachin 创建虚拟机 flavor为虚拟机模板id image为镜像id net-id为网络配置id 最后virmachin为虚拟机名称
```
