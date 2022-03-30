---
layout: post
title:  "基于archlinux构建软路由的方法和过程"
date:   2022-03-30 19:34:12 +0800
categories: archlinux router openwrt iptables
---


不同于硬件路由，软路由没有硬件报文转发的设施，依靠的是内核的路由控制；和硬件路由相同的是，软路由也是工作在IP协议层处理报文转发。

openwrt有着更小的体积和资源占用，但是扩展性不强，可用的软件包和应用有限，且需要一些学习的成本。

我已经使用archlinux3年多的时间，在已经了解archlinux的情况下，将家中的闲置设备打造为路由器是一直以来的想法，去年做成这件事之后对网络和系统的理解也有了更新。汇总如下，以飨后者。

## 硬件要求

软路由必须使用一个以上的网卡（可以是有线也可以是无线网卡），其中之一用于外网接入，接上级路由（wan）；另外的网卡用于接局域网内部链接，是内部路由（lan）。可以使用集线器来扩展lan网的网卡数量，但是对于软路由本机而言，lan口必不可少。

如果是软路由不接局域网（不设内容）则可以省去内部路由网卡，只使用单个网卡，此时无法建立隔离的内网网段。但可以通过将外网网卡所在的防火墙设置为允许进出和转发来对外网路由服务，外网的IP可以通过设置网关IP到软路由IP来使用软路由的分组转发能力。

**以上所涉及的内网与外网的概念，是相对与软路由本身而言。**

```shell
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc cake master lan0 state UP group default qlen 1000
    link/ether 5c:85:7e:43:2d:d3 brd ff:ff:ff:ff:ff:ff
3: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc cake master wan0 state UP group default qlen 1000
    link/ether 5c:85:7e:43:2d:d4 brd ff:ff:ff:ff:ff:ff
4: wlp4s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 5c:80:b6:b5:3f:3d brd ff:ff:ff:ff:ff:ff
```

可以使用`ip a` 命令查看当前的系统中所有可用的网卡，如上，其中lo是本地环回网卡，不可应用与软路由。`enp1s0`和`enp2s0`都是有线网卡，`wlp4s0`是无线网卡。

## 建立网桥

为了方便对网络的关闭，我们尽量避免直接使用物理网卡，而是使用网桥来管理和对应物理网卡，这样可以方便的建立和调整网络网口的关联关系，避免网络和网口出现严格绑定，不利于后续的管理和扩展。

archlinux默认使用systemd-networkd来管理网络，使用systemd-networkd建立网桥只需要在`/etc/systemd/network`新增*.netdev配置文件：

```
[NetDev]
Name=lan0
Kind=bridge
```

按照以上的内容可以建立lan0和wan0两个网桥，分别对应局域网和外部网络。

然后通过*.network文件将物理网卡和网桥建立关联关系，如：`20-wired.network`：

```
[Match]
Name=enp2s0

[Network]
Bridge=wan0
```

如上对的参数配置将enp2s0关联到了wan0网桥，也可以以同样的方式将enp1s0关联到lan0网桥，后续的网络设置将只针对wan0和lan0操作，而不再涉及物理网卡。

## 启用转发

通过sysctl配置启用ipv4的报文转发，以允许不同ip指向的报文在wan0口和lan0网络的转发；

```shell
# sysctl net.ipv4.ip_forward=1
```

还可以编辑`/etc/sysctl.d/30-ipforward.conf`文件来持久化ip_forward的配置项。

```
net.ipv4.ip_forward=1
net.ipv6.conf.default.forwarding=1
net.ipv6.conf.all.forwarding=1
```

## 启用NAT

软路由负责联通外部网络和内部网络，由于IP段的不一致，内部网络方位外部网络必须经过NAT网关，见source ip修改为外部网络的IP才能对外沟通，没有SNAT内部网络所在网段又是局域网IPv4网段的情况，内部网络将无法联网。

NAT是使用iptable来设置的，这在archlinux的base包中就做了包含，我们要做的知识启用iptable的rule读取服务，这样可以些一些持久化的iptable设置，在系统重启后依然可以使用。

```shell
# systemctl enable --now iptables.service
```

在`/etc/iptables/iptables.rules`应用如下的规则，既可以启用lan0到wan0的NAT网关，报文在转发后会自动或者Source的处理。

```
*nat
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o wan0 -j MASQUERADE
COMMIT

*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [0:0]
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i lan0 -o wan0 -j ACCEPT
COMMIT
```

## 规划网络

规划网络主要是指规划内网网络，选用一段局域网C类IPv4的网段，比如`192.168.8.0/24`作为内部局域网IP的网络，我们先不考虑局域网内的DHCP，那是网络治理的另外一个问题。

接下来，设置lan0的IP为局域网IP即可，由于局域网不存在DHCPServer（而且DHCPServer也依赖一个网关IP），需要设置lan0为静态IP，此静态IP也是局域网网络的网关IP。

同样是用systemd-networkd来配置lan0的静态IP，配置文件为：`/etc/systemd/network/lan0.network`：

```
[Match]
Name=lan0

[Network]
Address=192.168.8.1/24
ConfigureWithoutCarrier=yes
IPMasquerade=true
IPForward=true
```

局域网设置是至关重要的一步，设置完lan0就启动了局域网络。

## 连接外网

连接外网就是设置wan0的ip地址，连接外网有不同的方案，外部网络有可能是直接接入，有可能是wifi接入也有可能是pppoe家庭宽带的拨号接入，这里我们只讨论pppoe宽带接入的情况。

pppoe不在archlinux的base包的支持范围之内，需要安装`netctl`包和`ppp`包。

可以在建立`/etc/netctl/pppoe-wan0`配置文件，配置内容如下：

```
Description='PPPoE connection'
Interface=wan0
Connection=pppoe

User='<user>'
Password='<password>'

ConnectionMode='persist'
UsePeerDNS=false
PPPoEIP6=yes
```

PPPoEIP6参数配置用于启用IPv6的支持，填入正确的宽带供应商提供的pppoe拨号用户名和密码，然后启动netctl即可获取外部网络的连接。

```shell
# netctl enable pppoe-wan0
# netctl start pppoe-wan0
```

需要十分注意的一点是，由于使用pppoe拨号，ip报文会出现拆包的情况，需要在iptable的rule中增加配置mangle表的如下表项：

```
*mangle
:PREROUTING ACCEPT [0:0]
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:POSTROUTING ACCEPT [0:0]
-A FORWARD -o ppp0 -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu
COMMIT
```

至此，使用archlinux系统制作的软路由就架构成功了。接下来可以使用dnsmasq来配置局域网内的dhcp server和dns server 以将软路由效果最大化。