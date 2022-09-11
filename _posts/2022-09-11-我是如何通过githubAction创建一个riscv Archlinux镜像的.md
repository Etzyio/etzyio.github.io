---
layout: post
title:  "我是如何通过githubAction创建一个riscv Archlinux镜像的"
date:   2022-09-11 20:26:24 +0800
tags:   [github, CICD]
categories: github
---
# 通过github Action创建一个riscv Archlinux镜像

> github Action提供了多重构建环境，ubuntu/macos/windows

[sehraf/riscv-arch-image-builder](https://github.com/sehraf/riscv-arch-image-builder)此项目给出了创建riscv Archlinux镜像的脚本，但是其默认的环境是Archlinux中，需要安装的包也是一样，但github Action未提供此环境。所以我首先想到是通过Archlinux的docker镜像去创建镜像。

## github Action简单讲解

```yml
name: Image Builder for Archlinux on an Allwinner D1 / Sipeed Lichee RV CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Build and push Docker images
      run: 
        docker build . --file Dockerfile --tag riscv-arch-image:${{ github.event.commits[0].id }}

    - name: build img
      run: sh build_img.sh ${{ github.event.commits[0].id }}

    - name: Upload archive
      uses: actions/upload-artifact@v3
      with:
        name: archlinux_riscv.img.tar.gz
        path: /home/runner/work/riscv-arch-image-builder/riscv-arch-image-builder/archlinux_riscv.img.tar.gz

```

>name 是此CICD的名称\
>on 触发此CICD的时机\
>jobs 需要进行的工作\
>build 名称\
>runs-on 使用的环境\
>steps 此流程进行的步骤\
>uses 使用的插件，Action存在很多插件可供选择\
>actions/checkout@v2 克隆此项目\
>actions/upload-artifact@v3 发布文件\
>github.event.commits[0].id 是此github commit的id号，为将其push镜像到本机测试时使用的tag，实际操作中仅用于docker run。

## 构建Docker镜像

### 确认使用镜像

在项目文档中声明，在写入镜像时使用用到了[AUR](https://aur.archlinux.org/)源，故我在[dockerhub](https://hub.docker.com/)上找到了一个[greyltc-org/archlinux-aur](https://hub.docker.com/r/greyltc/archlinux-aur)可以使用，当然其存在github镜像的代理[ghcr.io/greyltc-org/archlinux-aur](https://github.com/greyltc-org/docker-archlinux-aur/pkgs/container/archlinux-aur)，这样docker的镜像确定了。

镜像确定后，直接尝试编写dockerfile，并使用docker build进行构建。

```Dockerfile
FROM ghcr.io/greyltc-org/archlinux-aur

WORKDIR /root

ADD 1_compile.sh 1_compile.sh
ADD 2_create_sd.sh 2_create_sd.sh
ADD consts.sh consts.sh
ADD licheerv_linux_defconfig licheerv_linux_defconfig
ADD uboot-makefile.patch uboot-makefile.patch
ADD upload_kernel.sh upload_kernel.sh

RUN pacman-key --init 
RUN pacman-key --populate archlinux
RUN pacman -Syyu --noconfirm riscv64-linux-gnu-gcc swig cpio python3 python-setuptools base-devel bc git arch-install-scripts parted
RUN aur-install qemu-user-static-bin binfmt-qemu-static

RUN sh 1_compile.sh output
RUN dd if=/dev/zero of=./archlinux_riscv.img bs=1M count=1500
RUN losetup /dev/loop17 archlinux_riscv.img
RUN sh 2_create_sd.sh /dev/loop17
CMD ["bash"]
```
dockerfile很简单的将需要的包及所需要进行的操作写入进去，实际进行构建时发现存在一定的问题。

#### 问题1 在构建镜像dd时，会存在需要很长的时间

时间长短并不会影响自动构建项目，故并没有太过在意此问题。

#### 问题2 在构建Docker镜像时使用losetup，出现无法使用

构建时遇到此问题，认为可能只存在于docker build镜像时，通过docker run --privileged=true提供足够的权限即可。尝试后发现在容器内依然无法使用。
思路是在主机中先挂载出来/dev/loop17，通过-v将/dev/loop17映射进去，将最后执行CMD修改为以下状态
```Dockerfile
CMD ["./2_create_sd.sh", "/dev/loop17"]
```
使用后发现依旧存在问题，更改方法为在主机中挂载，并通过docker cp将1_compile.sh中构建的output文件夹及其他内容拷贝出来。

#### 问题3 linux自动创建的makefile文件存在绝对路径目录地址

因为makefile中存在绝对路径目录地址，故查看Action构建的路径，并将docker中的工作目录设置为Action的当前目录

```Dockerfile
WORKDIR /home/runner/work/riscv-arch-image-builder/riscv-arch-image-builder
```

因为实际写入镜像是在Action中使用，故将dd创建空镜像一同在shell脚本中使用。

```shell
#!/bin/bash
dd if=/dev/zero of=./archlinux_riscv.img bs=1M count=1548
sudo losetup /dev/loop17 ./archlinux_riscv.img
docker run --name build_riscv riscv-arch-image:${1}
docker cp build_riscv:/home/runner/work/riscv-arch-image-builder/riscv-arch-image-builder ../
sed -i 's/USE_CHROOT=1/USE_CHROOT=0/g' consts.sh
sed -i 's/read -r  confirm/#read -r  confirm/g' 2_create_sd.sh
sudo ./2_create_sd.sh /dev/loop17
tar -zcvf archlinux_riscv.img.tar.gz archlinux_riscv.img
```
由于实际进行写入镜像在Action中，docker中的AUR“qemu-user-static-bin binfmt-qemu-static”并未参与到实际使用中，故通过sed将其设置为默认不使用。以及一个读取用户输入的操作禁止掉。完成此脚本。

最后在Action中使用actions/upload-artifact@v3发布，archlinux_riscv镜像构建完成。