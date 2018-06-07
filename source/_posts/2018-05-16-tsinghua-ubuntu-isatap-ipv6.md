---
title: 清华大学校内ubuntu设置isatap隧道上ipv6方法
date: 2018-05-16 13:58:02
tags:
---

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
第6行，`166.111.21.1`是`isatap.tsinghua.edu.cn`的地址，可以自己ping一下看看。