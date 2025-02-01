---
layout: post
title:  "prometheus和grafana监控服务器状态"
date:   2022-09-25 17:50:50 +0800
tags:   [linux, 运维]
categories: 运维
---
# prometheus和grafana监控服务器状态

## 介绍
舍友把软路由安装到路由器上了，原本装软路由的设备就重装了系统，安装了docker并被我知道了密码！！！
屋里的linux设备也有5个了，就找了找可以监控设备运行状态的软件，选择了prometheus和grafana的，用舍友的设备，可以先看下效果。
![图1](../assets/images/服务器状态监控/FireShot%20Capture%20002%20-%201%20Node%20Exporter%20Dashboard%2022_04_13%20ConsulManager自动同步版%20-%20Grafana_%20-%20192.168.1.105.png)
![图2](/assets/images/服务器状态监控/FireShot%20Capture%20003%20-%201%20Node%20Exporter%20Dashboard%2022_04_13%20ConsulManager自动同步版%20-%20Grafana_%20-%20192.168.1.105.png)
![图3](/assets/images/服务器状态监控/FireShot%20Capture%20004%20-%201%20Node%20Exporter%20Dashboard%2022_04_13%20ConsulManager自动同步版%20-%20Grafana_%20-%20192.168.1.105.png)
基本信息，硬盘空间等数据都可以在此显示出来。其中也有各种各样的监控内容，也存在专门监控mysql运行情况的prometheus-mysqld-exporter。

##  安装
### 服务端
首先确定环境中存在docker以及docker-compose，通过这两个安装prometheus和grafana的服务端。
```docker-compose.yml
version: "3"
services:
  prom:
    image: quay.io/prometheus/prometheus:latest
    restart: unless-stopped
    volumes:
     - ./data/monitor/prometheus.yml:/etc/prometheus/prometheus.yml
    command: "--config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/prometheus"
    ports:
     - 9090:9090
  grafana:
    image: grafana/grafana
    restart: unless-stopped
    ports:
     - "3000:3000"
    environment:
    - “GF_SECURITY_ADMIN_PASSWORD=Gz2020@”
    - “GF_INSTALL_PLUGINS=alexanderzobnin-zabbix-app”
    volumes:
      - ./data/grafana-etc:/etc/grafana/
      - ./data/grafana:/var/lib/grafana:rw
      - /etc/localtime:/etc/localtime
    depends_on:
      - prom
```
在docker-compose.yml的文件夹下创建data/monitor目录，在文件夹内创建prometheus.yml，以安装服务端的机器为例，设置job_name为server，需要监控的ip是192.168.1.31:9100，安装客户端在下文会给出。
```prometheus.yml
scrape_configs:
  - job_name: 'server'
    static_configs:
    - targets: ['192.168.1.31:9100']
```
配置好服务端直接运行docker-compose up -d运行，可以在服务端的9090和3000端口看到状态信息。

### 客户端
在需要的机器上安装prometheus-node-exporter。可以使用linux上对应的包管理工具进行安装。

安装成功后启动prometheus-node-exporter

```systemctl start prometheus-node-exporter```

### 确认是否安装成功

登陆服务端9090端口，`192.168.1.31:9090/targets?search=`可以看到设备已经在这里状态是up了，代表安装成功。

## 配置grafana

进入`http://192.168.1.31:3000`grafana会进行系统的安装引导，按照引导执行即可

### 配置数据源

在`http://192.168.1.31:3000/datasources`配置所需要的数据源，点击`Add data source`，搜索并点击`prometheus`，在`URL`中设置`http://prom:9090`，如果进行过其他设置，也可在其他配置项中进行设置。
配置后点击`Save & test`即可完成数据源配置。

### 设置仪表盘

在`http://192.168.1.31:3000/dashboards`中创建仪表盘，可以通过官方网站`https://grafana.com/grafana/dashboards/?pg=hp&plcmt=lt-box-dashboards`寻找自己认为好看的仪表盘进行直接导入。
以我现在使用的为例，在官方网站搜索到的仪表盘点击进入后在右下方会有`Copy ID to clipboard`和`Download JSON`，ID可以通过公网直接导入，JSON文件可以离线导入。
在仪表盘中点击`Import`输入ID`8919`，点击`Load`，点击后可以设置名称和`UID`，一定要记得进行配置数据源，在此界面选择`Prometheus`，设置好自己的数据源后即可完成设置。

现在进入仪表盘就可以看到自己的配置好的界面显示出来了，点击查看即可。