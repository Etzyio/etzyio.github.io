---
layout: post
title:  "keepalived的安装和使用"
date:   2022-09-27 17:21:50 +0800
tags:   [linux, 运维]
categories: 运维
---

# keepalived
朋友突然问我keepalived和nginx如何部署前后端项目，想着以前部署过一个mysql集群也有是用到keepalived，就简单的讲了讲。

>keepalived是为了Linux系统和基于linux的基础架构提供简单且强大的负载均衡和高可用的软件。

根据具体实现下总结下就是，保证一台服务器挂了之后切到另一台上，因为ip是虚拟出来的所以ip不会变化正常可用。大概给我这样的感觉。

## 安装

作为一个非常喜欢docker的人，那么keepalived的部署还是用到docker。

官网提供的运行方法，因为keepalived需要使用到宿主机的网卡，需要添加`--cap-add=NET_RAW`权限，

>docker run --volume /data/my-keepalived.conf:/container/service/keepalived/assets/keepalived.conf --detach osixia/keepalived:2.0.20

当然作为一个记不住docker run的人，肯定是要把它改写`docker-compose.yml`的，创建一个`docker-compose.yml`并且在系统中安装docker和docker-compose，直接运行docker-compose up -d即可完成安装。

```yaml
version: "3"
services:
  keepalived:
    image: osixia/keepalived:2.0.20
    volumes:
      - ./data/my-keepalived.conf:/container/service/keepalived/assets/keepalived.conf
    cap_add:
      - "NET_RAW"
    # 如果需要传递环境变量修改使用environment
    # environment:

```

## keepalived.conf配置

安装完keepalived，还需要进行编写conf配置。以配置mysql的keepalived.conf为例。

```conf
! Configuration File for keepalived

global_defs {
	script_user root
	enable_script_security
	router_id  mysql_node1
}


vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.50/24
    }
}
virtual_server 192.168.1.50 3306 {
   delay_loop 5   
   lb_algo rr	
   lb_kind DR 
   persistence_timeout 50
   protocol TCP
   real_server 192.168.1.51 3306 {
       weight 3
       notify_down    /etc/keepalived/mysql.sh
       TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 3306
       }
   }
}
```

> global_defs 全局定义\
> script_user 运行此脚步默认用户\
> enable_script_security 由非root用户运行脚本\
> router_id 默认是主机名\
> virtual_router_id 唯一值范围在1-255之间\
> priority 成为master节点的优先级，范围在1-255\
> advert_int VRRP发送的间隔时间默认为0.92\
> nopreempt 避免高优先级设备上线时抢占低优先级设备\
> authentication 身份验证\
> virtual_ipaddress 虚拟化出来的ip\
> virtual_server 声明虚拟化出来的ip，除了virtual_server ip port 也可以是virtual_server group groupname\
> delay_loop 验证时间\
> lb_algo lvs调度算法rr|wrr|lc|wlc|lblc|sh|dh\
> lb_kind 负载均衡转发规则NAT|DR|RUN\
> persistence_timeout lvs超时时间默认是6分钟\
> protocol 协议\
> real_server 真实ip\
> weight 真实ip的权重，默认是1\
> notify_down 当服务器挂掉时需要运行的脚本\
> connect_timeout 连接超时时间\
> nb_get_retry 失败重试次数\
> delay_before_retry 失败重试间隔时间\
> connect_port 连接的端口

通过这份keepalived的配置文件可以理解为，虚拟化出192.168.1.50这个ip，再判断此太机器上的3306端口是否保持连接，当连接不存在时取消此结点上作为主节点切换到另一台服务上。
通过keepalived可以构建出一个高可用的集群。