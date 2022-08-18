---
title: 清华大学校内设置isatap隧道使用ipv6方法
date: 2018-05-16 13:58:02
tags:
---

本文介绍了在清华大学校内使用ipv6的方法，包括windows系统和ubuntu系统。

<!--more-->

# 准备

首先，在往下看以前，先试一下，你所在的宿舍楼，实验室，能不能使用原生的ipv6。
所谓原生的ipv6，指的是，插上网线就可以自动连接ipv6。
原生的ipv6是最好用的，不需要配置。但是清华校内只有部分地方支持，我不知道为什么。。。

好了，假如说你所在的网络不支持原生ipv6，那么你需要建立isatap隧道来连接ipv6。

# Windows

首先，需要禁用系统自带的6to4服务、teredo服务、以及原生ipv6环境。

- 禁用6to4:
```bat
netsh interface 6to4 set state disable
```

- 禁用teredo:
```bat
netsh interface teredo set state disable
```

- 禁用原生ipv6环境:
打开控制面板->网络和Internet->网络连接，找到你正在使用的网络连接，右键->属性，在打开的对话框内，找到“Internet协议版本6(TCP/IPv6)”，取消勾选。

接下来设置isatap。

输入下面的语句来配置：
```bat
netsh interface isatap set route isatap.tsinghua.edu.cn
netsh interface isatap set state enable
```

注意事项：
学校的isatap服务器是`isatap.tsinghua.edu.cn`，ip地址是`166.111.21.1`。


# Ubuntu

```bash
#!/bin/bash
sudo modprobe ipv6
sudo ip tunnel del sit1
MYIP=$(ifconfig enp2s0 | grep "inet "|awk '{print $2}')
echo My ip address: ${MYIP}
sudo ip tunnel add sit1 mode sit remote 166.111.21.1 local ${MYIP}
sudo ifconfig sit1 up
sudo ifconfig sit1 add 2402:f000:1:1501:200:5efe:${MYIP}/64
sudo ip route add ::/0 via 2402:f000:1:1501::1 metric 1
```

注意事项：
第4行，是获取本机IP地址的命令。`enp2s0`是网卡名称。
第4行的命令目的是提取本机的IP地址。由于每个电脑的`ifconfig`命令运行结果不同，所以这个命令在其他电脑上很可能无法正确运行。到时候，可以直接手写IP地址，就像这样：
```bash
MYIP=255.255.255.255
```
第6行，学校的isatap服务器是`isatap.tsinghua.edu.cn`，ip地址是`166.111.21.1`。

# 测试

无论windows还是ubuntu，均可以使用
```bash
ping ipv6.google.com
```
来测试是否成功连接ipv6。
