---
layout: post
title:  "onlyoffice编译"
date:   2025-02-01 18:56:00 +0800
tags:   [协同办公, 编译]
categories: onlyoffice
---

# onlyoffice

最近受不了内网中使用samba，导致第二个人编辑文件是只读的情况，想保存需要重新创建新文件，还会导致文件结果不能同步。找了找开源的协同办公方案，发现有onlyoffice和Collabora这两种。

collabora因为我菜没搭建成功，而onlyoffice有20人限制，需要重新编译代码，编译过程参考了以下几位的文档。

[解除OnlyOffice社区版连接数限制并在Docker下配置Ubuntu环境部署使用](https://zsy314.wordpress.com/tag/onlyoffice/)\
[OnlyOffice验证（一）DocumentServer编译验证](https://blog.csdn.net/baidu_32377671/article/details/129181119)

其中与onlyoffice搭配的是[seafile](https://www.seafile.com/home/)文档比较清晰，使用也毕竟简单。

## 编译环境

环境使用的国外的云主机2c2g50g，因为内存较小给了8g的swap，之前给了16gswap，但是主机只有50g，很快被onlyoffice环境占满了，最后采取8g编译，软件环境是在ubuntu16的docker镜像中。
编译过程中会使用到google的p[chromium的工具](https://chromium.googlesource.com/chromium/tools/depot_tools.git)，需要开启vpn进行编译。

## 操作流程

```
#拉取镜像
docker pull ubuntu:16.04
#启动容器
docker run -i -t -p 8022:22 --name OnlyOffice ubuntu:16.04
#链接docker命令行（使用Docker Desktop的终端也可）
docker exec -i -t OnlyOffice bash
```

进入镜像中先安装必要软件，及将onlyoffice的编译环境克隆下来，选择的版本是v8.2.2.32。

```
#准备软件环境
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y python git sudo
#拉取build_tools工具
git clone --depth=1 --recursive --branch v8.2.2.32 https://github.com/ONLYOFFICE/build_tools.git
#编译
cd build_tools/tools/linux
./automate.py server
```

需要修改server\Common\sources\constants.js下的内容。
```
exports.LICENSE_CONNECTIONS = 20;
exports.LICENSE_USERS = 3;
```
再进行修改build_tools\tools\linux\automate.py下的
```
build_tools_params = ["--branch", branch, 
                      "--module", modules, 
                      "--update", "1", # 改为0使得下次编译不再更新文件
                      "--qt-dir", os.getcwd() + "/qt_build/Qt-5.9.9"] + params
```
再次执行
```
./automate.py server
```
