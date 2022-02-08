---
layout: post
title:  "使用frp将内网端口暴露到外网以提供服务"
date:   2017-08-20 14:34:12 +0800
categories: dev linux
---


场景：大部分国内的网络环境下都无法分配到公网ip，尤其是多人同时使用一条宽带的情况下（比如合租）；如果需要将自己内网的服务暴露到外网来管理和访问就需要特殊的技巧。

## 环境准备

内网环境下有一台7*24工作的树莓派机器，为了高效利用机器的内存/磁盘/CPU资源，有时候需要工作环境去访问这台机器。要完成对此树莓派的访问，需要准备一下的事项！

1. 准备一台树莓派，作为访问目标
2. 下载frp的运行环境，[下载地址](https://github.com/fatedier/frp)
3. 准备一台可以有公网ip的vps机器，可以在国内网的各大vps提供商购买
4. 内网环境下的树莓派需要可以正常访问vps机器

## 服务端环境运行

`frp`在github下载源码之后，最好进行本地编译，使用go的运行环境可以方便的进行跨平台编译，比如我的目标运行环境是amd64-linux，可以使用如下的编译指令：

```bash
env GOOS=linux GOARCH=amd64 go build
```

`frp`编译成功后在bin目录有两个重要的二进制运行文件，`frps`-服务端运行和`frpc`-客户端运行，编译后的文件体积比较大，可以通过一下的命令来减小体积：

```bash
strip bin/*
```

编译完成后需要将编译成功的二进制文件拷贝到远程vps的环境中：

```bash
cp -r bin xxx@x.x.x.x:
```

将用户名和ip替换为自己的vps机器的用户名和ip就可以了，默认拷贝到用户的`home`目录中，登录远程的服务器，就可以看到bin目录出现在了服务器中，在正式运行之前，先来看一下`frps`的配置文件：

```ini
[common]
bind_addr = 0.0.0.0
bind_port = 7000
log_file = console
log_level = info

[rpi-web]
auth_token = 123456
bind_addr = 0.0.0.0
listen_port = 80
```

其中`[common]`配置了服务端的监听端口，`[rpi-web]`配置了映射端口，可以将服务端的`80`访问转发到客户端，也就是内网机器-树莓派！

## 客户端环境运行

上一步编译出的`bin/frpc`就是客户端的运行命令，与服务端的配置文件相对应客户端的配置文件大致上应该是这个样子的：

```ini
[common]
server_addr = x.x.x.x
server_port = 7000
log_file = console
log_level = info

[rpi-web]
local_port = 8080
remote_port = 80
use_encryption = false
auth_token = 123456
```

客户端和服务端配置的配置整合起来的意思就是，将树莓派的8080端口映射到服务端的80端口，这样所有对vps`80`的访问都会转发到树莓派来处理！

要注意一点的是：树莓派是arm架构的，所有起运行的客户端需要单独编译，编译的指令为：

```bash
env GOOS=linux GOARCH=arm go build
```

## 环境运行

1. 在服务端开启`frps`
2. 在客户端开启任何一个监听`8080`的服务
3. 客户端开启`frpc`

接下来，访问x.x.x.x:80就可以直接穿透到树莓派了！

## 写在最后

众所周知，通过ssh也可以将内网的访问映射到外网，但是其访问比较容易断线，造成访问中断，综合考虑还是frp高效一点（frp支持kcp）！