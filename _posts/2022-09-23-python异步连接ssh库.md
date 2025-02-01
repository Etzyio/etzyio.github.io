---
layout: post
title:  "python异步连接ssh库 Parallel-SSH"
date:   2022-09-23 18:30:48 +0800
tags:   [python, 运维]
categories: 运维
---

# python异步连接ssh库 Parallel-SSH

因为新工作是运维，一次要巡检百来台服务器，原来的运维脚本一次需要跑近1个小时，为了将巡检时间有效降低就去查了下是否有异步ssh库，就找到了Parallel-SSH这个库。
使用起来基本在一分钟以内即可完成服务器巡检内容，而且使用起来也非常简单，故在这里给大家介绍一下。

## 安装
如果是能连得上外网的机器可以加上源直接使用pip进行安装
```bash
pip install parallel-ssh
```
由于工作环境的机器无法连接外网，所以可以使用pip download
```bash
# 下载
pip download -d parallel-ssh 
# 在离线电脑上安装
pip install parallel_ssh-2.12.0-py3-none-any.whl
```

## 连接

#### 使用账号密码登陆
```python
from pssh.clients import ParallelSSHClient
# 几个需要连接到的服务器
hosts = ['localhost', 'localhost', 'localhost', 'localhost']
# 连接前设置用户名及密码
client = ParallelSSHClient(hosts, user='my_user', password='my_pass')
```

#### 使用密钥连接
```python
client = ParallelSSHClient(hosts, pkey="~/.ssh/my_key")
```

#### 使用多个账号密码登录
导入HostConfig设置，可以进行每台主机的账号密码设置
```python
from pssh.config import HostConfig
hosts = ['localhost', 'localhost']
host_config = [
    HostConfig(port=2222, user='user1',
               password='pass', private_key='my_pkey.pem'),
    HostConfig(port=2223, user='user2',
               password='pass', private_key='my_other_key.pem'),
]
client = ParallelSSHClient(hosts, host_config=host_config)
```

## 执行命令

```python
output = client.run_command(cmd)
for host_output in output:
    print(host_output.host)
    print(list(host_output.stdout))
    print(host_output.exit_code)
    for line in host_output.stdout:
        print(line)
```
使用run_command返回的是一个列表，这个列表中含有host和执行结果的命令，对其进行处理获取想要的结果。
host_output.stdout是一个迭代器，想直接输出需要list()。

#### 不同主机使用不同的命令

由于多台服务器中存在一定的差异，所以需要执行的命令也会不一样，可以使用以下方式针对修改。
```
output = client.run_command('%s', host_args=('host1_cmd',
                                             'host2_cmd',
                                             'host3_cmd',))
```
host1_cmd是第一台主机需要执行的命令，以此类推

#### 获取执行中数据或连接超时输出

join可以避免流程阻塞，在执行后即可获得返回的数据，也可以设置超时时间异常。
```python
# 连接超时
client.join(output, timeout=5)
# 读取现在内容
client.join()
```

#### 需要使用sudo时
需要使用sudo时，可能需要运行stdin.write()进行输入密码
```
output = client.run_command('cmd', sudo=True)
for host_out in output:
    host_out.stdin.write('my_password\n')
    host_out.stdin.flush()
client.join(output)
```

## sftp

主机端传输到服务器，如果/root/test文件夹不存在将会自动创建
```python
client.copy("test.py", "/root/test")
```

服务器传输文件到主机端，由于相同文件名的文件会被覆盖，copyt_remote_file将会自动将本地文件重命名。
```python
client.copy_remote_file("/root/test/test.py", "localtest")
```

## 等待执行完成

```python
client.join_shell(output)
```

几百台主机执行速度非常快，使用时对代码没有进行什么优化过，不过由于数量确实很多对cpu单核承受的压力也不小，如果有运维需求的可以使用一下。