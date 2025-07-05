# 老男孩网络

本章将介绍以下配方：

+   设置网络

+   让我们开始 Ping！

+   跟踪 IP 路由

+   列出网络上所有可用的机器

+   使用 SSH 在远程主机上运行命令

+   在远程机器上运行图形命令

+   通过网络传输文件

+   连接到无线网络

+   无密码自动登录 SSH

+   使用 SSH 进行端口转发

+   将远程驱动器挂载到本地挂载点

+   网络流量和端口分析

+   测量网络带宽

+   创建任意套接字

+   构建桥接

+   共享互联网连接

+   使用`iptables`构建基础防火墙

+   创建虚拟私人网络

# 介绍

网络是将计算机连接起来以便交换信息的行为。最广泛使用的网络协议栈是 TCP/IP，其中每个节点都被分配一个唯一的 IP 地址以进行标识。如果你已经熟悉网络，可以跳过这部分介绍。

TCP/IP 网络通过将数据包从一个节点传输到另一个节点来工作。每个数据包都包含目标的 IP 地址以及可以处理该数据的应用程序的端口号。

当一个节点接收到一个数据包时，它会检查该数据包是否是其目标。如果是，它会检查端口号，并调用相应的应用程序来处理数据。如果该节点不是目标节点，它会根据网络状况判断，并将数据包传递给更接近最终目标的节点。

Shell 脚本可用于配置网络中的节点、测试机器的可用性、自动化在远程主机上的命令执行等。本章提供了涉及网络的工具和命令的配方，并展示如何有效使用它们。

# 设置网络

在深入了解基于网络的操作之前，首先需要具备设置网络、相关术语以及分配 IP 地址、添加路由等命令的基本知识。本篇提供了 GNU/Linux 网络中使用的命令概览。

# 准备工作

网络接口通过有线或无线连接将计算机与网络物理连接。Linux 使用诸如`eth0`、`eth1`或`enp0s25`（指以太网接口）这样的名称来表示网络接口。其他接口，如`usb0`、`wlan0`和`tun0`，分别用于 USB 网络接口、无线局域网和隧道。

在本配方中，我们将使用以下命令：`ifconfig`、`route`、`nslookup`和`host`。

`ifconfig`命令用于配置和显示有关网络接口、子网掩码等的详细信息。它应该位于`/sbin/ifconfig`。

# 如何操作...

1.  列出当前的网络接口配置：

```
 $ ifconfig 
 lo        Link encap:Local Loopback 
 inet addr:127.0.0.1  Mask:255.0.0.0 
 inet6addr: ::1/128 Scope:Host 
 UP LOOPBACK RUNNING  MTU:16436  Metric:1 
 RX packets:6078 errors:0 dropped:0 overruns:0 frame:0 
 TX packets:6078 errors:0 dropped:0 overruns:0 carrier:0 
 collisions:0 txqueuelen:0 
 RX bytes:634520 (634.5 KB)  TX bytes:634520 (634.5 KB) 
 wlan0     Link encap:EthernetHWaddr 00:1c:bf:87:25:d2 
 inet addr:192.168.0.82  Bcast:192.168.3.255  Mask:255.255.252.0 
 inet6addr: fe80::21c:bfff:fe87:25d2/64 Scope:Link 
 UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1 
 RX packets:420917 errors:0 dropped:0 overruns:0 frame:0 
 TX packets:86820 errors:0 dropped:0 overruns:0 carrier:0 
 collisions:0 txqueuelen:1000 
 RX bytes:98027420 (98.0 MB)  TX bytes:22602672 (22.6 MB) 

```

`ifconfig`输出的最左侧列显示网络接口的名称，右侧列则展示与对应网络接口相关的详细信息。

1.  设置网络接口的 IP 地址，请使用以下命令：

```
 # ifconfig wlan0 192.168.0.80 

```

你需要以 root 身份运行上述命令

`192.168.0.80` 被定义为无线设备 wlan0 的地址

要设置子网掩码以及 IP 地址，请使用以下命令：

```
 # ifconfig wlan0 192.168.0.80  netmask 255.255.252.0 

```

1.  许多网络使用**动态主机配置协议**（**DHCP**）在计算机连接到网络时自动分配 IP 地址。当您的计算机连接到自动分配 IP 地址的网络时，`dhclient` 命令会分配 IP 地址。如果通过 DHCP 分配地址，请使用 `dhclient` 而不是手动选择可能与网络上另一台机器冲突的地址。许多 Linux 发行版在检测到网络电缆连接时会自动调用 `dhclient`。

```
 # dhclient eth0 

```

# 还有更多内容...

`ifconfig` 命令可以与其他 Shell 工具结合使用，生成特定的报告。

# 打印网络接口列表

下面这个单行命令序列会显示系统上可用的网络接口：

```
$ ifconfig | cut -c-10 | tr -d ' ' | tr -s 'n' 
lo 
wlan0 

```

`ifconfig` 输出的每行的前十个字符用于写入网络接口的名称。因此，我们使用 `cut` 提取每行的前十个字符。`tr -d ' '` 删除每行中的所有空格字符。现在，使用 `tr -s '\n'` 压缩每行的 `n` 换行符，以生成接口名称列表。

# 显示 IP 地址

`ifconfig` 命令显示系统上每个活动网络接口的详细信息。但是，我们可以使用以下命令将其限制为特定接口：

```
$ ifconfig iface_name 

```

考虑以下示例：

```
$ ifconfig wlan0 
wlan0     Link encap:EthernetHWaddr 00:1c:bf:87:25:d2 
inet addr:192.168.0.82 Bcast:192.168.3.255 Mask:255.255.252.0 
inet6 addr: fe80::3a2c:4aff:6e6e:17a9/64 Scope:Link 
UP BROADCAST RUNNINT MULTICAST  MTU:1500 Metric:1 
RX Packets... 

```

要控制设备，我们需要 IP 地址、广播地址、硬件地址和子网掩码：

+   `HWaddr 00:1c:bf:87:25:d2`: 这是硬件地址（MAC 地址）

+   `inet addr:192.168.0.82`: 这是 IP 地址

+   `Bcast:192.168.3.255`: 这是广播地址

+   `Mask:255.255.252.0`: 这是子网掩码

要从 `ifconfig` 输出中提取 IP 地址，请使用此命令：

```
$ ifconfig wlan0 | egrep -o "inetaddr:[^ ]*" | grep -o "[0-9.]*" 
192.168.0.82 

```

`egrep -o "inetaddr:[^ ]*"` 命令返回 `inet addr:192.168.0.82`。该模式以 `inetaddr:` 开头，并以任何非空格字符序列结束（由 `[^ ]*` 指定）。下一个命令 `grep -o "[0-9.]*"` 将其输入减少为仅数字和点号，并打印出一个 IP4 地址。

# 伪造硬件地址（MAC 地址）

当认证或过滤基于硬件地址时，我们可以使用硬件地址伪造。硬件地址在 `ifconfig` 输出中显示为 `HWaddr 00:1c:bf:87:25:d2`。

`ifconfig` 的 `hw` 子命令将定义设备类和 MAC 地址：

```
# ifconfig eth0 hw ether 00:1c:bf:87:25:d5 

```

在上述命令中，`00:1c:bf:87:25:d5` 是要分配的新 MAC 地址。当我们需要通过 MAC 身份验证的服务提供商访问互联网时，这是非常有用的，这些服务提供商为单台计算机提供互联网访问。

注意：此定义仅在计算机重新启动之前有效。

# 名称服务器和 DNS（域名服务）

互联网的底层地址方案是点分十进制格式（如 `83.166.169.231`）。人类更喜欢使用单词而不是数字，因此互联网上的资源通过被称为 **URL** 或 **域名** 的单词串来标识。例如， [www.packtpub.com](http://www.packtpub.com) 是一个域名，它对应着一个 IP 地址。该站点可以通过数字或字符串名称来标识。

将 IP 地址映射到符号名称的技术称为 **域名服务** (**DNS**)。当我们输入 [www.google.com](http://www.google.com) 时，我们的计算机会使用 DNS 服务器将域名解析为相应的 IP 地址。在本地网络中，我们设置本地 DNS，通过符号名称为本地机器命名。

名称服务器在 `/etc/resolv.conf` 中定义：

```
$ cat /etc/resolv.conf 
# Local nameserver 
nameserver 192.168.1.1 
# External nameserver 
nameserver 8.8.8.8 

```

我们可以通过编辑该文件手动添加名称服务器，也可以使用一行命令：

```
# sudo echo nameserver IP_ADDRESS >> /etc/resolv.conf 

```

获取 IP 地址的最简单方法是使用 `ping` 命令访问域名。回复中包括了 IP 地址：

```
$ ping google.com 
PING google.com (64.233.181.106) 56(84) bytes of data. 

```

数字 `64.233.181.106` 是 google.com 服务器的 IP 地址。

一个域名可能映射到多个 IP 地址。在这种情况下，`ping` 显示列表中的一个地址。要获取与域名关联的所有地址，我们应使用 DNS 查找工具。

# DNS 查找

几个 DNS 查找工具提供从命令行进行名称和 IP 地址解析。`host` 和 `nslookup` 是两个常见的实用工具。

`host` 命令列出所有与域名关联的 IP 地址：

```
$ host google.com 
google.com has address 64.233.181.105 
google.com has address 64.233.181.99 
google.com has address 64.233.181.147 
google.com has address 64.233.181.106 
google.com has address 64.233.181.103 
google.com has address 64.233.181.104 

```

`nslookup` 命令将名称映射到 IP 地址，也可以将 IP 地址映射到名称：

```
$ nslookup google.com 
Server:    8.8.8.8 
Address:  8.8.8.8#53 

Non-authoritative answer: 
Name:  google.com 
Address: 64.233.181.105 
Name:  google.com 
Address: 64.233.181.99 
Name:  google.com 
Address: 64.233.181.147 
Name:  google.com 
Address: 64.233.181.106 
Name:  google.com 
Address: 64.233.181.103 
Name:  google.com 
Address: 64.233.181.104 

Server:    8.8.8.8 

```

前述命令行片段中的最后一行对应用于解析的默认名称服务器。

通过在 `/etc/hosts` 文件中添加条目，可以为 IP 地址解析添加符号名称。

`/etc/hosts` 中的条目遵循此格式：

```
IP_ADDRESS name1 name2 ... 

```

你可以这样更新 `/etc/hosts`：

```
# echo IP_ADDRESS symbolic_name>> /etc/hosts 

```

请考虑以下示例：

```
# echo 192.168.0.9 backupserver>> /etc/hosts 

```

添加此条目后，任何对 `backupserver` 的解析都将解析为 `192.168.0.9`。

如果 `backupserver` 有多个名称，你可以在同一行中包括它们：

```
# echo 192.168.0.9 backupserver backupserver.example.com >> /etc/hosts 

```

# 显示路由表信息

拥有互联网络是常见的。例如，工作或学校的不同部门可能在不同的网络中。当一个网络中的设备想要与另一个网络中的设备通信时，它需要通过一个对两个网络都通用的设备发送数据包。这个设备称为 `网关`，它的功能是将数据包从一个网络路由到另一个网络。

操作系统维护一个称为 `路由表` 的表格，其中包含有关如何在网络上通过机器转发数据包的信息。`route` 命令显示路由表：

```
$ route 
Kernel IP routing table 
Destination      Gateway   GenmaskFlags  Metric  Ref  UseIface 
192.168.0.0         *      255.255.252.0  U     2      0     0wlan0 
link-local          *      255.255.0.0    U     1000   0     0wlan0 
default          p4.local  0.0.0.0        UG    0      0     0wlan0 

```

或者，你也可以使用以下方法：

```
$ route -n 
Kernel IP routing table 
Destination   Gateway      Genmask       Flags Metric Ref  UseIface 
192.168.0.0   0.0.0.0      255.255.252.0   U     2     0     0   wlan0 
169.254.0.0   0.0.0.0      255.255.0.0     U     1000  0     0   wlan0 
0.0.0.0       192.168.0.4  0.0.0.0         UG    0     0     0   wlan0 

```

使用 `-n` 参数指定显示数字地址。默认情况下，route 会将数字地址映射到名称。

当系统不知道到达目标的路径时，它会将数据包发送到默认网关。默认网关可能是通向互联网的链接或是一个部门间的路由器。

`route add` 命令可以添加默认网关：

```
# route add default gw IP_ADDRESS INTERFACE_NAME 

```

请看这个例子：

```
# route add default gw 192.168.0.1 wlan0 

```

# 另见

+   第一章《Shell 脚本》中的 *使用变量和环境变量* 章节，解释了 `PATH` 变量。

+   第四章《开车与发短信》中的 *使用 grep 搜索和挖掘文件内容* 章节，解释了 `grep` 命令。

# 让我们来 ping 吧！

`ping` 命令是一个基本的网络命令，支持所有主流操作系统。Ping 用于验证网络中主机之间的连接性，并识别可访问的计算机。

# 如何操作...

ping 命令使用 **互联网控制消息协议**（**ICMP**）数据包来检查网络中两台主机的连接性。当这些回显数据包被发送到目标时，如果连接成功，目标会做出回应。如果没有到达目标的路由，或者目标到请求者的路由不可知，ping 请求可能会失败。

对一个地址进行 ping 测试将检查主机是否可达：

```
$ ping ADDRESS 

```

`ADDRESS` 可以是主机名、域名或 IP 地址本身。

默认情况下，`ping` 将持续发送数据包，回复信息会打印在终端上。通过按 *Ctrl* + *C* 停止 ping 测试过程。

考虑以下示例：

+   当主机可达时，输出将类似于以下内容：

```
 $ ping 192.168.0.1 
 PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data. 
 64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=1.44 ms 
 ^C 
 --- 192.168.0.1 ping statistics --- 
 1 packets transmitted, 1 received, 0% packet loss, time 0ms 
 rtt min/avg/max/mdev = 1.440/1.440/1.440/0.000 ms 

 $ ping google.com 
 PING google.com (209.85.153.104) 56(84) bytes of data. 
 64 bytes from bom01s01-in-f104.1e100.net (209.85.153.104):    
        icmp_seq=1 ttl=53 time=123 ms 
 ^C 
 --- google.com ping statistics --- 
 1 packets transmitted, 1 received, 0% packet loss, time 0ms 
 rtt min/avg/max/mdev = 123.388/123.388/123.388/0.000 ms 

```

+   当主机不可达时，输出将类似于此：

```
 $ ping 192.168.0.99 
 PING 192.168.0.99 (192.168.0.99) 56(84) bytes of data. 
 From 192.168.0.82 icmp_seq=1 Destination Host Unreachable 
 From 192.168.0.82 icmp_seq=2 Destination Host Unreachable 

```

如果目标无法到达，ping 命令会返回 `Destination Host Unreachable` 错误消息。

网络管理员通常会配置设备，如路由器，不响应 `ping`。这样做是为了降低安全风险，因为攻击者（通过暴力攻击）可以利用 `ping` 查找机器的 IP 地址。

# 还有更多...

除了检查网络中两点之间的连接性外，`ping` 命令还会返回其他信息。往返时间和丢包报告可用于判断网络是否正常工作。

# 往返时间

`ping` 命令会显示每个发送并返回的数据包的 **往返时间**（**RTT**）。RTT 以毫秒为单位报告。在内部网络中，RTT 小于 1ms 很常见。在 ping 测试互联网站点时，RTT 通常为 10-400 ms，可能超过 1000 ms：

```
--- google.com ping statistics --- 
5 packets transmitted, 5 received, 0% packet loss, time 4000ms 
rtt min/avg/max/mdev = 118.012/206.630/347.186/77.713 ms 

```

这里，最小 RTT 为 `118.012 ms`，平均 RTT 为 `206.630 ms`，最大 RTT 为 `347.186 ms`。ping 输出中的 `mdev`（`77.713 ms`）参数代表均方差。

# 序列号

每个 ping 发送的数据包都会分配一个序号，从 1 开始，直到 ping 停止。如果网络接近饱和，数据包可能会因为碰撞和重试而乱序返回，或者完全丢失：

```
$> ping example.com 
64 bytes from example.com (1.2.3.4): icmp_seq=1 ttl=37 time=127.2 ms 
64 bytes from example.com (1.2.3.4): icmp_seq=3 ttl=37 time=150.2 ms 
64 bytes from example.com (1.2.3.4): icmp_seq=2 ttl=30 time=1500.3 ms 

```

在这个例子中，第二个包被丢弃，然后在超时后重试，导致其顺序被打乱，并且返回的往返时间更长。

# 生存时间

每个 ping 包都有一个预定义的跳数限制，超过这个值就会被丢弃。每个路由器都会将该值减去 1。这个值表示从你的系统到你正在 ping 的网站之间有多少个路由器。初始的**生存时间**（**TTL**）值会根据你的平台或 ping 版本有所不同。你可以通过 ping 回环连接来确定初始值：

```
$> ping 127.0.0.1 
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.049 ms 
$> ping www.google.com 
64 bytes from 173.194.68.99: icmp_seq=1 ttl=45 time=49.4 ms 

```

在这个例子中，我们 ping 回环地址以确定 TTL 值（没有跳数的情况下为 64）。然后我们 ping 一个远程站点，并将 TTL 值从我们的无跳值中减去，以确定两个站点之间有多少跳数。在这个例子中，64-45 等于 19 跳。

TTL 值在两个站点之间通常是恒定的，但当条件需要选择替代路径时可能会发生变化。

# 限制发送的包数

`ping` 命令发送回显包并等待回显回复，直到通过按 *Ctrl* + *C* 停止。`-c` 参数将限制发送的回显包的数量：

```
-c COUNT 

```

请看这个例子：

```
$ ping 192.168.0.1 -c 2  
PING 192.168.0.1 (192.168.0.1) 56(84) bytes of data.  
64 bytes from 192.168.0.1: icmp_seq=1 ttl=64 time=4.02 ms 
64 bytes from 192.168.0.1: icmp_seq=2 ttl=64 time=1.03 ms 

--- 192.168.0.1 ping statistics ---  
2 packets transmitted, 2 received, 0% packet loss, time 1001ms  
rtt min/avg/max/mdev = 1.039/2.533/4.028/1.495 ms 

```

在前一个例子中，`ping` 命令发送两个回显包后停止。这在我们需要通过脚本从 IP 地址列表中 ping 多台机器并检查它们的状态时非常有用。

# `ping` 命令的返回状态

`ping` 命令在成功时返回退出状态`0`，失败时返回非零值。`成功`表示目标主机可达，而`失败`表示目标主机不可达。

返回状态可以通过以下方式获取：

```
$ ping domain -c2 
if [ $? -eq0 ]; 
then 
 echo Successful ; 
else 
 echo Failure 
fi 

```

# 路由追踪

当一个应用通过互联网请求服务时，服务器可能位于遥远的地方，并通过多个网关或路由器进行连接。`traceroute` 命令显示数据包在到达目标之前经过的所有中间网关的地址。`traceroute` 信息帮助我们了解每个数据包到达目标所经过的跳数。中间网关的数量表示网络中两个节点之间的有效距离，这个距离可能与物理距离无关。每次跳跃都会增加传输时间。路由器需要时间来接收、解码并转发数据包。

# 如何操作...

`traceroute` 命令的格式如下：

```
traceroute destinationIP 

```

`destinationIP` 可以是数字或字符串：

```
$ traceroute google.com 
traceroute to google.com (74.125.77.104), 30 hops max, 60 byte packets 
1  gw-c6509.lxb.as5577.net (195.26.4.1)  0.313 ms  0.371 ms  0.457 ms 
2  40g.lxb-fra.as5577.net (83.243.12.2)  4.684 ms  4.754 ms  4.823 ms 
3  de-cix10.net.google.com (80.81.192.108)  5.312 ms  5.348 ms  5.327 ms 
4  209.85.255.170 (209.85.255.170)  5.816 ms  5.791 ms 209.85.255.172 (209.85.255.172)  5.678 ms 
5  209.85.250.140 (209.85.250.140)  10.126 ms  9.867 ms  10.754 ms 
6  64.233.175.246 (64.233.175.246)  12.940 ms 72.14.233.114 (72.14.233.114)  13.736 ms  13.803 ms 
7  72.14.239.199 (72.14.239.199)  14.618 ms 209.85.255.166 (209.85.255.166)  12.755 ms 209.85.255.143 (209.85.255.143)  13.803 ms 
8  209.85.255.98 (209.85.255.98)  22.625 ms 209.85.255.110 (209.85.255.110)  14.122 ms 
*  
9  ew-in-f104.1e100.net (74.125.77.104)  13.061 ms  13.256 ms  13.484 ms 

```

现代 Linux 发行版还附带了 `mtr` 命令，它类似于 traceroute，但显示实时数据，并会不断刷新。它对于检查网络承载质量非常有用。

# 列出网络中所有可用的机器

当我们监控一个大型网络时，需要检查所有机器的可用性。机器不可用可能有两个原因：未开机，或由于网络问题。我们可以编写一个 Shell 脚本来判断并报告哪些机器在网络中可用。

# 准备工作

在这个例子中，我们展示了两种方法。第一种方法使用 ping，第二种方法使用 `fping`。`fping` 命令比 ping 命令更适合脚本，并且有更多的功能。它可能不是你 Linux 发行版的一部分，但可以通过包管理器安装。

# 如何操作...

下一个示例脚本将使用 ping 命令找到网络中可见的机器：

```
#!/bin/bash 
#Filename: ping.sh 
# Change base address 192.168.0 according to your network. 

for ip in 192.168.0.{1..255} ; 
do 
 ping $ip -c 2 &> /dev/null ; 

 if [ $? -eq 0 ]; 
 then 
 echo $ip is alive 
 fi 
done 

```

输出类似于以下内容：

```
$ ./ping.sh 
192.168.0.1 is alive 
192.168.0.90 is alive 

```

# 它是如何工作的...

这个脚本使用 `ping` 命令来查找网络上可用的机器。它使用 `for` 循环遍历由 `192.168.0.{1..255}` 表达式生成的 IP 地址列表。`{start..end}` 表示法会生成从开始到结束的值。在这个例子中，它会创建从 `192.168.0.1` 到 `192.168.0.255` 的 IP 地址。

`ping $ip -c 2 &> /dev/null` 运行一个 `ping` 命令到相应的 IP 地址。`-c` 选项让 ping 只发送两个数据包。`&> /dev/null` 将 `stderr` 和 `stdout` 重定向到 /dev/null，因此终端上不会打印任何内容。脚本使用 `$?` 来评估退出状态。如果成功，退出状态为 `0`，并打印响应我们 ping 的 IP 地址。

在这个脚本中，为每个地址依次执行一个独立的 `ping` 命令。当某个 IP 地址没有回复时，脚本会变得很慢，因为每次 ping 必须等待超时才能开始下一个 ping。

# 还有更多...

接下来的脚本展示了如何改进 ping 脚本以及如何使用 `fping`。

# 并行 ping 测试

之前的脚本按顺序测试每个地址。每次测试的延迟会累积并变得很大。并行运行 ping 命令可以加快速度。将循环体括在 `{}&` 中会使 `ping` 命令并行运行。`( )` 用来将一组命令括起来作为子 shell 执行，`&` 将其发送到后台：

```
#!/bin/bash 
#Filename: fast_ping.sh 
# Change base address 192.168.0 according to your network. 

for ip in 192.168.0.{1..255} ; 
do 
   ( 
      ping $ip -c2 &> /dev/null ; 

      if [ $? -eq0 ]; 
      then 
       echo $ip is alive 
      fi 
   )& 
  done 
wait 

```

在 `for` 循环中，我们执行了多个后台进程并退出循环，终止脚本。`wait` 命令会阻止脚本在所有子进程退出之前终止。

输出将按照 ping 响应的顺序显示。如果某些机器或网络段较慢，响应的顺序将与发送的顺序不同。

# 使用 fping

第二种方法使用了一种不同的命令，叫做 `fping`。`fping` 命令向多个 IP 地址发送 ICMP 消息，然后等待查看哪些地址会回复。它比第一种脚本运行得快得多。

`fping` 提供的选项包括以下内容：

+   `fping` 的 `-a` 选项指定显示可用机器的 IP 地址

+   `fping` 的 `-u` 选项指定显示不可达的机器

+   `-g` 选项指定从指定的 IP/mask 或起始和结束 IP 地址的斜线子网掩码表示法生成一系列 IP 地址：

```
 $ fping -a 192.160.1/24 -g 

```

另外，可以使用以下方法：

```
 $ fping -a 192.160.1 192.168.0.255 -g 

```

+   `2>/dev/null` 用于将由于主机不可达而打印的错误信息转储到一个空设备。

也可以手动指定一组 IP 地址作为命令行参数或通过`stdin`作为列表。考虑以下示例：

```
$ fping -a 192.168.0.1 192.168.0.5 192.168.0.6 
# Passes IP address as arguments 
$ fping -a <ip.list 
# Passes a list of IP addresses from a file 

```

# 另见

+   第一章中*操作文件描述符和重定向*的配方，*Shell 一些操作*，解释了数据重定向

+   第一章中*比较与测试*的配方，*Shell 一些操作*，解释了数字比较

# 使用 SSH 在远程主机上运行命令

**SSH** 代表**安全外壳**。它通过加密隧道连接两台计算机。SSH 允许您访问远程计算机上的 Shell，在该 Shell 中可以交互地运行单个命令并接收结果，或者启动交互式会话。

# 准备工作

SSH 并不是所有 GNU/Linux 发行版中预装的。您可能需要使用包管理器安装 `openssh-server` 和 `openssh-client` 包。默认情况下，SSH 运行在端口号 `22` 上。

# 如何操作...

1.  要连接到运行 SSH 服务器的远程主机，可以使用以下命令：

```
 $ ssh username@remote_host 

```

该命令中的选项如下：

+   `username` 是远程主机上存在的用户

+   `remote_host` 可以是域名或 IP 地址

考虑这个示例：

```
 $ ssh mec@192.168.0.1 
 The authenticity of host '192.168.0.1 (192.168.0.1)' can't be   
        established. 
 RSA key fingerprint is   
        2b:b4:90:79:49:0a:f1:b3:8a:db:9f:73:2d:75:d6:f9. 
 Are you sure you want to continue connecting (yes/no)? yes 
 Warning: Permanently added '192.168.0.1' (RSA) to the list of   
        known hosts. 
 Password: 

 Last login: Fri Sep  3 05:15:21 2010 from 192.168.0.82 
 mec@proxy-1:~$ 

```

SSH 会要求输入密码，成功验证后，它会连接到远程计算机的登录 Shell。

SSH 会执行指纹验证，以确保我们确实连接到目标远程计算机。这是为了避免所谓的**中间人攻击**，即攻击者试图冒充另一台计算机。SSH 默认会在第一次连接到服务器时存储指纹，并验证后续连接时指纹是否没有变化。

默认情况下，SSH 服务器运行在端口`22`。然而，某些服务器会在不同的端口运行 SSH 服务。在这种情况下，使用 `-p port_num` 选项与 `ssh` 命令一起指定端口。

1.  连接到在端口 `422` 上运行的 SSH 服务器：

```
 $ ssh user@locahost -p 422 

```

在 Shell 脚本中使用 `ssh` 时，我们不需要交互式 Shell，我们只想在远程系统上执行命令并处理命令的输出。

每次都输入密码对自动化脚本来说不太实际，因此应该配置 SSH 密钥的无密码登录。*无密码自动登录与 SSH* 配方在本章中解释了设置这一功能的 SSH 命令。

1.  要在远程主机上运行命令并显示其输出到本地 Shell，可以使用以下语法：

```
 $ sshuser@host 'COMMANDS' 

```

考虑这个示例：

```
 $ ssh mec@192.168.0.1 'whoami' 
 mec

```

您可以通过用分号分隔命令来提交多个命令：

```
 $ ssh user@host "command1 ; command2 ; command3" 

```

考虑以下示例：

```
 $ ssh mec@192.168.0.1  "echo user: $(whoami);echo OS: $(uname)" 
 Password: user: mec OS: Linux 

```

在这个示例中，执行的命令如下：

```
 echo user: $(whoami); 
 echo OS: $(uname) 

```

我们可以通过使用 `( )` 子 Shell 操作符，在命令序列中传递一个更复杂的子 Shell。

1.  下一个示例是一个基于 SSH 的 Shell 脚本，用于收集一组远程主机的正常运行时间。正常运行时间是自上次开机以来的时间长度，可以通过 `uptime` 命令获取。

假设`IP_LIST`中的所有系统都使用公共用户`test`。

```
 #!/bin/bash 
 #Filename: uptime.sh 
 #Description: Uptime monitor 

 IP_LIST="192.168.0.1 192.168.0.5 192.168.0.9" 
 USER="test" 

 for IP in $IP_LIST; 
 do 
 utime=$(ssh ${USER}@${IP} uptime  |awk '{ print $3 }' ) 
 echo $IP uptime:  $utime 
 done 

```

期望输出：

```
 $ ./uptime.sh 
 192.168.0.1 uptime: 1:50, 
 192.168.0.5 uptime: 2:15, 
 192.168.0.9 uptime: 10:15, 

```

# 还有更多内容...

`ssh`命令可以执行多个附加选项。

# 带压缩的 SSH

SSH 协议支持数据传输压缩。当带宽有限时，这个功能特别有用。使用`ssh`命令的`-C`选项来启用压缩：

```
$ ssh -C user@hostname COMMANDS 

```

# 将数据重定向到远程主机 shell 命令的 stdin

SSH 允许你将本地系统任务的输出作为输入传递给远程系统：

```
$ echo 'text' | ssh user@remote_host 'echo' 
text 

```

或者，可以使用以下方法：

```
# Redirect data from file as: 
$ ssh user@remote_host 'echo'  < file 

```

在远程主机上，`echo`会打印通过`stdin`接收到的数据，而这些数据又是从本地主机的`stdin`传递过来的。

该功能可以用于将 tar 归档从本地主机传输到远程主机。详细内容请参见第七章，*备份计划*：

```
$> tar -czf - LOCALFOLDER | ssh 'tar -xzvf-' 

```

# 在远程机器上运行图形命令

如果你尝试在远程机器上运行一个需要图形窗口的命令，你会看到类似`cannot open display`的错误信息。这是因为`ssh`终端尝试（并未成功）连接到远程机器上的 X 服务器。

# 如何操作...

要在远程服务器上运行图形应用程序，你需要设置`$DISPLAY`变量，以强制该应用程序连接到本地机器的 X 服务器：

```
ssh user@host "export DISPLAY=:0 ; command1; command2""" 

```

这将在远程主机上启动图形输出。

如果你想在本地机器上显示图形输出，可以使用 SSH 的 X11 转发选项：

```
ssh -X user@host "command1; command2" 

```

这将会在远程机器上运行命令，但会将图形显示在本地机器上。

# 另见

+   本章中的*无密码自动登录 SSH*配方解释了如何配置自动登录，执行命令时无需输入密码。

# 通过网络传输文件

网络计算机的一个主要用途是资源共享。文件是常见的共享资源。不同的文件传输方法包括使用 USB 闪存、`sneakernet`，以及通过 NFS 和 Samba 等网络链接进行文件传输。这些配方介绍了如何使用常见协议 FTP、SFTP、RSYNC 和 SCP 来传输文件。

# 准备工作

执行网络文件传输的命令在 Linux 安装时大多是默认可用的。可以使用传统的`ftp`命令或更新的`lftp`命令通过 FTP 传输文件，或通过 SSH 连接使用`scp`或`sftp`命令传输文件。还可以使用`rsync`命令在不同系统间同步文件。

# 如何操作...

**文件传输协议**（**FTP**）是一个较旧的协议，许多公共网站使用它来共享文件。此服务通常运行在`21`端口。FTP 要求远程主机上安装并运行 FTP 服务器。我们可以使用传统的`ftp`命令或更新的`lftp`命令来访问启用 FTP 的服务器。以下命令同时支持`ftp`和`lftp`。FTP 被广泛用于许多公共网站的文件共享。

要连接到 FTP 服务器并进行文件传输，使用以下命令：

```
$ lftpusername@ftphost 

```

它将提示输入密码，然后显示已登录的提示符：

```
lftp username@ftphost:~> 

```

你可以在此提示符下输入命令，如下所示：

+   `cd directory`：这将在远程系统上更改目录

+   `lcd`：这将更改本地机器上的目录

+   `mkdir`：这将在远程机器上创建一个目录

+   `ls`：这将列出远程机器上当前目录中的文件

+   `get FILENAME`：这将把文件下载到本地机器的当前目录：

```
 lftp username@ftphost:~> get filename 

```

+   `put filename`：这将从远程机器的当前目录上传文件：

```
 lftp username@ftphost:~> put filename 

```

+   `quit`命令将终止一个`lftp`会话。

`lftp`提示符支持自动补全

# 还有更多内容...

让我们了解通过网络进行文件传输时使用的额外技术和命令。

# 自动化 FTP 传输

`lftp`和`ftp`命令打开一个与用户的交互式会话。我们可以通过 Shell 脚本自动化 FTP 文件传输：

```
#!/bin/bash 

#Automated FTP transfer 
HOST=example.com' 
USER='foo' 
PASSWD='password' 
lftp  -u ${USER}:${PASSWD} $HOST <<EOF 

binary 
cd /home/foo 
put testfile.jpg 

quit 
EOF 

```

上述脚本的结构如下：

```
<<EOF 
DATA 
EOF 

```

这用于通过`stdin`将数据发送到`lftp`命令。*文件描述符和重定向操作*，第一章的食谱《Shell Something Out》解释了重定向到*stdin*的各种方法。

`-u`选项使用我们定义的`USER`和`PASSWD`登录远程站点。`binary`命令将文件模式设置为二进制。

# SFTP（安全 FTP）

SFTP 是一种文件传输系统，运行在 SSH 连接之上，并模拟 FTP 接口。它需要在远程系统上运行 SSH 服务器，而不是 FTP 服务器。它提供一个带有`sftp`提示符的交互式会话。

SFTP 支持与`ftp`和`lftp`相同的命令。

要启动`sftp`会话，使用以下命令：

```
$ sftp user@domainname 

```

与`lftp`类似，`sftp`会话可以通过输入`quit`命令终止。

有时，SSH 服务器不会在默认端口`22`上运行。如果它在其他端口上运行，我们可以在`-oPort=PORTNO`后指定端口。请参考以下示例：

```
$ sftp -oPort=422 user@slynux.org 

```

`-oPort`应该是`sftp`命令的第一个参数。

# rsync 命令

`rsync`命令广泛用于通过网络复制文件以及进行备份快照。本章中详细描述了*使用 rsync 进行备份快照*。

第七章的食谱，*备份计划*。

# SCP（安全拷贝程序）

SCP 是一种安全的文件拷贝命令，类似于旧版、不安全的远程拷贝工具`rcp`。文件通过加密通道使用 SSH 进行传输：

```
$ scp filename user@remotehost:/home/path 

```

这将提示输入密码。像`ssh`一样，传输可以通过自动登录 SSH 技术实现免密登录。本章中的*免密自动登录 SSH*食谱解释了 SSH 自动登录。一旦 SSH 登录自动化，就可以在没有交互式密码提示的情况下执行 scp 命令。

`remotehost`可以是 IP 地址或域名。`scp`命令的格式如下：

```
$ scp SOURCE DESTINATION 

```

`SOURCE`或`DESTINATION`可以采用`username@host:/path`格式：

```
$ scp user@remotehost:/home/path/filename filename 

```

上述命令将文件从远程主机复制到当前目录，并使用给定的文件名。

如果 SSH 运行在与`22`不同的端口上，请使用`-oPort`，并采用相同的语法，`sftp`。

# 使用 scp 递归复制

`-r`参数告诉`scp`在两台机器之间递归地复制目录：

```
$ scp -r /home/usernameuser@remotehost:/home/backups 
# Copies the directory /home/usernameto the remote backup 

```

`-p`参数将使`scp`在复制文件时保留权限和模式。

# 另见

+   第一章中*玩转文件描述符和重定向*的食谱，解释了使用 EOF 的标准输入。

# 连接到无线网络

以太网连接配置简单，因为它是通过有线电缆连接的，没有像认证之类的特殊要求。然而，无线局域网需要一个**扩展服务集标识符**（**ESSID**）网络标识符，可能还需要一个密码短语。

# 准备工作

要连接到有线网络，只需使用`ifconfig`工具分配 IP 地址和子网掩码。无线网络连接需要`iwconfig`和`iwlist`工具。

# 如何操作...

本脚本将连接到使用**WEP**（**有线等效隐私**）的无线局域网：

```
#!/bin/bash 
#Filename: wlan_connect.sh 
#Description: Connect to Wireless LAN 

#Modify the parameters below according to your settings 
######### PARAMETERS ########### 
IFACE=wlan0 
IP_ADDR=192.168.1.5 
SUBNET_MASK=255.255.255.0 
GW=192.168.1.1 
HW_ADDR='00:1c:bf:87:25:d2' 
#Comment above line if you don't want to spoof mac address 

ESSID="homenet" 
WEP_KEY=8b140b20e7 
FREQ=2.462G 
################################# 

KEY_PART="" 

if [[ -n $WEP_KEY ]]; 
then 
 KEY_PART="key $WEP_KEY" 
fi 

if [ $UID -ne 0 ]; 
then 
 echo "Run as root" 
 exit 1; 
fi 

# Shut down the interface before setting new config 
/sbin/ifconfig $IFACE down 

if [[ -n $HW_ADDR  ]]; 
then 
 /sbin/ifconfig $IFACE hw ether $HW_ADDR 
 echo Spoofed MAC ADDRESS to $HW_ADDR 
fi 

/sbin/iwconfig $IFACE essid $ESSID $KEY_PART freq $FREQ 

/sbin/ifconfig $IFACE $IP_ADDR netmask $SUBNET_MASK 

route add default gw $GW $IFACE 

echo Successfully configured $IFACE 

```

# 它是如何工作的...

`ifconfig`、`iwconfig`和`route`命令必须以 root 身份运行。因此，在脚本中执行任何操作之前会检查是否为 root 用户。

无线局域网连接需要诸如`essid`、`key`和`frequency`等参数。`essid`是要连接的无线网络的名称。一些网络使用 WEP 密钥进行认证，通常是一个五个或十个字符的十六进制密码。网络分配的频率是`iwconfig`命令所需的，用来将无线网卡与正确的无线网络连接。

`iwlist`工具将扫描并列出可用的无线网络：

```
# iwlist scan 
wlan0     Scan completed : 
 Cell 01 - Address: 00:12:17:7B:1C:65 
 Channel:11 
 Frequency:2.462 GHz (Channel 11) 
 Quality=33/70  Signal level=-77 dBm 
 Encryption key:on 
 ESSID:"model-2" 

```

`Frequency`参数可以从扫描结果中提取，来自`Frequency:2.462 GHz (Channel 11)`这一行。

示例中使用了 WEP。注意，WEP 是不安全的。如果你在管理无线网络，建议使用**Wi-Fi 保护接入 2**（**WPA2**）的变体。

# 另见

+   第一章中*比较和测试*的食谱，解释了字符串比较。

# 使用 SSH 实现无密码自动登录

SSH 在自动化脚本中广泛使用，因为它使得能够远程执行命令并读取远程主机的输出。通常，SSH 使用用户名和密码进行身份验证，这些信息会在执行 SSH 命令时提示输入。在自动化脚本中提供密码是不切实际的，因此我们需要实现自动登录。SSH 具有允许会话自动登录的功能。本食谱介绍了如何为自动登录创建 SSH 密钥。

# 准备工作

SSH 使用一种称为非对称密钥的加密技术，包含两个密钥——公钥和私钥，用于自动认证。`ssh-keygen` 应用程序会创建一对认证密钥。为了实现自动认证，公钥必须放置在服务器上（通过将公钥追加到 `~/.ssh/authorized_keys` 文件中），而私钥文件应存在于客户端机器的 `~/.ssh` 目录中。可以通过修改 `/etc/ssh/sshd_config` 配置文件来修改 SSH 配置选项（例如，`authorized_keys` 文件的路径和名称）。

# 如何操作...

实现 SSH 自动认证有两个步骤，具体如下：

+   在本地机器上创建 SSH 密钥

+   将公钥传输到远程主机并将其添加到 `~/.ssh/authorized_keys`（这需要访问远程机器）

要创建 SSH 密钥，请运行 `ssh-keygen` 命令，并指定加密算法类型为 RSA：

```
$ ssh-keygen -t rsa 
Generating public/private rsa key pair.  
Enter file in which to save the key (/home/username/.ssh/id_rsa):  
Created directory '/home/username/.ssh'.  
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in /home/username/.ssh/id_rsa.  
Your public key has been saved in /home/username/.ssh/id_rsa.pub.  
The key fingerprint is:  
f7:17:c6:4d:c9:ee:17:00:af:0f:b3:27:a6:9c:0a:05 username@slynux-laptop 
The key'srandomart image is:  
+--[ RSA 2048]----+  
|           .     |  
|            o . .| 
|     E       o o.| 
|      ...oo |  
|       .S .+  +o.|  
|      .  . .=....|  
|     .+.o...|  
|      . . + o.  .| 
|       ..+       |  
+-----------------+  

```

您需要输入一个密码短语以生成公私钥对。虽然可以在不输入密码短语的情况下生成密钥对，但这样做不安全。

如果您打算编写使用自动登录的脚本来访问多台机器，应该留空密码短语，以防脚本在运行时要求输入密码短语。

`ssh-keygen` 程序会创建两个文件。`~/.ssh/id_rsa.pub` 和 `~/.ssh/id_rsa:id_rsa.pub` 是生成的公钥，`id_rsa` 是私钥。公钥必须添加到需要从当前主机进行自动登录的远程服务器的 `~/.ssh/authorized_keys` 文件中。

此命令将追加一个密钥文件：

```
$ ssh USER@REMOTE_HOST \
    "cat >> ~/.ssh/authorized_keys" < ~/.ssh/id_rsa.pub
Password:

```

在之前的命令中提供登录密码。

从现在开始，自动登录已设置完成，因此 SSH 在执行过程中不会提示输入密码。使用以下命令测试：

```
$ ssh USER@REMOTE_HOST uname 
Linux 

```

您将不会被提示输入密码。大多数 Linux 发行版包括 `ssh-copy-id`，它会将您的私钥追加到远程服务器的适当 `authorized_keys` 文件中。这比前面描述的 `ssh` 技术更简洁：

```
ssh-copy-id USER@REMOTE_HOST 

```

# 使用 SSH 进行端口转发

端口转发是一种将 IP 连接从一台主机重定向到另一台主机的技术。例如，如果您将 Linux/Unix 系统用作防火墙，则可以将连接重定向到端口 `1234`，并指向像 `192.168.1.10:22` 这样的内部地址，从外部世界通过 SSH 隧道访问内部机器。

# 如何操作...

您可以将本地机器上的端口转发到另一台机器上，也可以将远程机器上的端口转发到另一台机器。在以下示例中，一旦转发完成，您将得到一个 shell 提示符。保持此 shell 打开以使用端口转发，并在任何时候退出以停止端口转发。

1.  此命令将把您本地机器的端口 `8000` 转发到 [www.kernel.org](http://www.kernel.org) 的端口 `80`：

```
 ssh -L 8000:www.kernel.org:80user@localhost 

```

将用户替换为您本地机器上的用户名。

1.  该命令将会把远程机器上的端口 8000 转发到[www.kernel.org](http://www.kernel.org)的端口`80`：

```
 ssh -L 8000:www.kernel.org:80user@REMOTE_MACHINE 

```

这里，将`REMOTE_MACHINE`替换为远程机器的主机名或 IP 地址，将`user`替换为你具有 SSH 访问权限的用户名。

# 还有更多内容...

使用非交互模式或反向端口转发时，端口转发更为有用。

# 非交互式端口转发

如果你只想设置端口转发，而不是保持一个打开的 Shell 会话，可以使用以下格式的`ssh`命令：

```
ssh -fL8000:www.kernel.org:80user@localhost -N 

```

`-f`选项指示`ssh`在执行命令之前先分叉到后台。`-N`告诉`ssh`没有命令要运行；我们只是想进行端口转发。

# 反向端口转发

反向端口转发是 SSH 最强大的功能之一。它在以下情况特别有用：你有一台机器无法从互联网公开访问，但你希望别人能够访问这台机器上的服务。在这种情况下，如果你可以 SSH 连接到一台可以从互联网公开访问的远程机器，你可以在该远程机器上设置一个反向端口转发，将其转发到正在运行该服务的本地机器。

```
ssh -R 8000:localhost:80 user@REMOTE_MACHINE 

```

该命令将把远程机器上的端口`8000`转发到本地机器上的端口`80`。请记得将`REMOTE_MACHINE`替换为远程机器的主机名或 IP 地址。

使用此方法时，如果你在远程机器上浏览`http://localhost:8000`，将会连接到本地机器上运行在端口`80`的 Web 服务器。

# 将远程驱动器挂载到本地挂载点

拥有一个本地挂载点来访问远程主机文件系统可以方便地进行读写数据传输操作。SSH 是常见的传输协议。`sshfs`应用程序利用 SSH 协议，让你可以将远程文件系统挂载到本地挂载点。

# 准备工作

`sshfs`在 GNU/Linux 发行版中默认并不安装。使用包管理器安装`sshfs`。`sshfs`是 FUSE 文件系统包的扩展，允许用户将各种数据挂载为本地文件系统。FUSE 的变体在 Linux、Unix、Mac OS/X、Windows 等系统上得到支持。

欲了解更多有关 FUSE 的信息，请访问其官方网站：[`fuse.sourceforge.net/`](http://fuse.sourceforge.net/)。

# 如何操作...

要将远程主机上的文件系统位置挂载到本地挂载点：

```
# sshfs -o allow_otheruser@remotehost:/home/path /mnt/mountpoint 
Password: 

```

在提示时输入密码。密码被接受后，远程主机`/home/path`上的数据可以通过本地挂载点`/mnt/mountpoint`进行访问。

要卸载，请使用以下命令：

```
# umount /mnt/mountpoint 

```

# 另见

+   本章的*使用 SSH 在远程主机上运行命令*食谱解释了`ssh`命令。

# 网络流量和端口分析

每个访问网络的应用程序都通过端口进行。列出打开的端口、使用端口的应用程序和运行该应用程序的用户是一种跟踪系统预期和意外使用情况的方法。这些信息可以用于分配资源以及检查 rootkit 或其他恶意软件。

# 准备工作

有多种命令可用于列出网络节点上运行的端口和服务。`lsof` 和 `netstat` 命令在大多数 GNU/Linux 发行版中都可用。

# 如何操作...

`lsof` （列出打开文件）命令将列出打开的文件。`-i` 选项将其限制为打开的网络连接：

```
$ lsof -i 
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF NODE 
    NAME

firefox-b 2261 slynux   78u  IPv4  63729      0t0  TCP 
    localhost:47797->localhost:42486 (ESTABLISHED)

firefox-b 2261 slynux   80u  IPv4  68270      0t0  TCP 
    slynux-laptop.local:41204->192.168.0.2:3128 (CLOSE_WAIT)

firefox-b 2261 slynux   82u  IPv4  68195      0t0  TCP 
    slynux-laptop.local:41197->192.168.0.2:3128 (ESTABLISHED)

ssh       3570 slynux    3u  IPv6  30025      0t0  TCP 
    localhost:39263->localhost:ssh (ESTABLISHED)

ssh       3836 slynux    3u  IPv4  43431      0t0  TCP 
    slynux-laptop.local:40414->boney.mt.org:422 (ESTABLISHED)

GoogleTal 4022 slynux   12u  IPv4  55370      0t0  TCP 
    localhost:42486 (LISTEN)

GoogleTal 4022 slynux   13u  IPv4  55379      0t0  TCP 
    localhost:42486->localhost:32955 (ESTABLISHED)

```

`lsof` 输出中的每一项都对应一个具有活动网络端口的服务。输出的最后一列包含类似以下内容的行：

```
laptop.local:41197->192.168.0.2:3128 

```

在此输出中，`laptop.local:41197` 对应 `localhost`，而 `192.168.0.2:3128` 对应远程主机。`41197` 是当前机器上使用的端口，`3128` 是服务连接的远程主机端口。

要列出当前机器上打开的端口，请使用以下命令：

```
$ lsof -i | grep ":[0-9a-z]+->" -o | grep "[0-9a-z]+" -o  | sort | uniq 

```

# 它是如何工作的...

`:[0-9a-z]+->` 正则表达式用于 `grep` 提取 `lsof` 输出中的主机端口部分 `(:34395-> 或 :ssh->)`。接下来的 `grep` 移除前导冒号和尾随箭头，留下端口号（该端口号是字母数字组合）。通过同一端口可能会发生多个连接，因此，可能会出现相同端口的多个条目。输出将被排序并通过 `uniq` 过滤，确保每个端口只显示一次。

# 还有更多...

还有更多实用工具报告打开的端口和与网络流量相关的信息。

# 使用 netstat 查看打开的端口和服务

`netstat` 还会返回网络服务统计信息。它有许多功能，超出了本配方的范围。

使用 `netstat -tnp` 列出打开的端口和服务：

```
$ netstat -tnp 
Proto Recv-Q Send-Q Local Address           Foreign Address         
    State       PID/Program name 

tcp        0      0 192.168.0.82:38163      192.168.0.2:3128        
    ESTABLISHED 2261/firefox-bin 

tcp        0      0 192.168.0.82:38164      192.168.0.2:3128        
    TIME_WAIT   -               

tcp        0      0 192.168.0.82:40414      193.107.206.24:422      
    ESTABLISHED 3836/ssh

tcp        0      0 127.0.0.1:42486         127.0.0.1:32955         
    ESTABLISHED 4022/GoogleTalkPlug

tcp        0      0 192.168.0.82:38152      192.168.0.2:3128        
    ESTABLISHED 2261/firefox-bin 

tcp6       0      0 ::1:22                  ::1:39263               
    ESTABLISHED -               

tcp6       0      0 ::1:39263               ::1:22                  
    ESTABLISHED 3570/ssh

```

# 测量网络带宽

前面对 `ping` 和 `traceroute` 的讨论涉及测量网络延迟和节点间的跳数。

`iperf` 应用程序提供了更多网络性能指标。`iperf` 默认没有安装，但大多数发行版的包管理器都提供该应用程序。

# 如何操作...

`iperf` 应用程序必须在链路的两端（主机和客户端）安装。安装好 `iperf` 后，启动服务器端：

```
$ iperf -s 

```

然后运行客户端以生成吞吐量统计信息：

```
$ iperf -c 192.168.1.36 
------------------------------------------------------------ 
Client connecting to 192.168.1.36, TCP port 5001 
TCP window size: 19.3 KByte (default) 
------------------------------------------------------------ 
[  3] local 192.168.1.44 port 46526 connected with 192.168.1.36 port 5001 
[ ID] Interval       Transfer     Bandwidth 
[  3]  0.0-10.0 sec   113 MBytes  94.7 Mbits/sec 

```

`-m` 选项指示 `iperf` 还要查找 **最大传输大小** (**MTU**):

```
$ iperf -mc 192.168.1.36 
------------------------------------------------------------ 
Client connecting to 192.168.1.36, TCP port 5001 
TCP window size: 19.3 KByte (default) 
------------------------------------------------------------ 
[  3] local 192.168.1.44 port 46558 connected with 192.168.1.36 port 5001 
[ ID] Interval       Transfer     Bandwidth 
[  3]  0.0-10.0 sec   113 MBytes  94.7 Mbits/sec 
[  3] MSS size 1448 bytes (MTU 1500 bytes, ethernet) 

```

# 创建任意套接字

对于文件传输和安全外壳等操作，有像 ftp 和 `ssh` 这样的预构建工具。我们也可以编写自定义脚本作为网络服务。下一个配方演示了如何创建简单的网络套接字并用它们进行通信。

# 准备工作

`netcat` 或 `nc` 命令将创建网络套接字，通过 TCP/IP 网络传输数据。我们需要两个套接字：一个用于监听连接，另一个连接到监听器。

# 如何操作...

1.  使用以下命令设置监听套接字：

```
 nc -l 1234 

```

这将创建一个在本地机器上监听`1234`端口的套接字。

1.  使用以下命令连接到套接字：

```
 nc HOST 1234 

```

如果你在与监听套接字相同的机器上运行此操作，将`HOST`替换为 localhost；否则，将其替换为机器的 IP 地址或主机名。

1.  在你执行步骤 2 的终端上输入一些内容并按*Enter*，消息将在你执行步骤 1 的终端上显示。

# 还有更多内容...

网络套接字不仅可以用于文本通信，还可以用于其他用途，接下来的章节将介绍这些应用。

# 快速通过网络复制文件

我们可以利用`netcat`和 shell 重定向来通过网络复制文件。以下命令将把文件发送到监听机器：

1.  在监听机器上，运行以下命令：

```
 nc -l 1234 >destination_filename 

```

1.  在发送端机器上，运行以下命令：

```
 nc HOST 1234 <source_filename 

```

# 创建一个广播服务器

你可以使用`netcan`来创建一个自定义的服务器。接下来的例子展示了一个每 10 秒钟发送时间的服务器。通过连接到端口并使用客户端`nc`会话（telnet）可以接收时间：

```
# A script to echo the date out a port 
while [ 1 ] 
do 
 sleep 10 
 date 
done | nc -l 12345  
echo exited 

```

# 它是如何工作的...

使用`nc`复制文件之所以有效，是因为`nc`将一个套接字的输入回显到另一个套接字的输出。

广播服务器稍微复杂一些。`while [ 1 ]`循环将会永远运行。在循环中，脚本休眠 10 秒钟，然后调用`date`命令，并将输出传递给`nc`命令。

你可以使用`nc`来创建一个客户端，如下所示：

```
$ nc 127.0.0.1 12345 

```

# 构建桥接

如果你有两个独立的网络，可能需要一种方式将数据从一个网络传递到另一个网络。通常通过使用路由器、集线器或交换机将两个子网连接起来。

Linux 系统可以用作网络桥接。

桥接是一种低级连接方式，它根据 MAC 地址传递数据包，而不是通过 IP 地址进行标识。因此，桥接占用的计算机资源较少，效率更高。

你可以使用桥接将虚拟机连接到私有的非路由网络，或者将公司内部的不同子网连接起来，例如，将制造子网与运输子网连接，以便共享生产信息。

# 准备工作

Linux 内核自 2.2 版本以来就支持网络桥接。目前定义桥接的工具是 iproute2（`ip`）命令，这在大多数发行版中是标准工具。

# 如何操作...

`ip`命令通过命令/子命令模型执行多个操作。要创建桥接，我们使用`ip link`命令。

在将以太网适配器添加到桥接时，应该确保该适配器没有配置 IP 地址。桥接将配置一个地址，而不是网卡本身。

在这个例子中，有两个网卡：`eth0`已经配置并连接到`192.168.1.0`子网，而`eth1`尚未配置，但将通过桥接连接到`10.0.0.0`子网：

```
 # Create a new bridge named br0 
 ip link add br0 type bridge 

 # Add an Ethernet adapter to the bridge 
 ip link set dev eth1 master br0 

 # Configure the bridge's IP address 
 ifconfig br0 10.0.0.2 

 # Enable packet forwarding 
 echo 1 >/proc/sys/net/ipv4/ip_forward 

```

这创建了一个桥接，使数据包能够从`eth0`发送到`eth1`并返回。在桥接能够发挥作用之前，我们需要将这个桥接添加到路由表中。

在`10.0.0.0/24`网络中的机器上，我们添加一个到`192.168.1.0/16`网络的路由：

```
route add -net 192.168.1.0/16 gw 10.0.0.2 

```

`192.168.1.0/16`子网中的机器需要知道如何找到`10.0.0.0/24`子网。如果`eth0`网卡配置为 IP 地址`192.168.1.2`，则路由命令如下：

```
route add -net 10.0.0.0/24 gw 192.168.1.2 

```

# 共享互联网连接

大多数防火墙/路由器都能够与家里或办公室的设备共享互联网连接。这叫做**网络地址转换**（**NAT**）。一个带有两个**网络接口卡**（**NIC**）的 Linux 计算机可以充当路由器，提供防火墙保护和连接共享。

防火墙和 NAT 支持由内核中内置的 iptables 支持提供。这个配方介绍了`iptables`，通过它将计算机的以太网连接通过无线接口共享到互联网，从而让其他无线设备通过主机的以太网网卡访问互联网。

# 准备工作

这个配方使用`iptables`定义**网络地址转换**（**NAT**），它允许网络设备与其他设备共享连接。你需要无线接口的名称，可以通过`iwconfig`命令获得。

# 如何实现……

1.  连接到互联网。在这个配方中，我们假设主要的有线网络连接`eth0`已连接到互联网。根据你的设置进行更改。

1.  使用你的发行版的网络管理工具，创建一个新的临时无线连接，设置如下：

    +   IP 地址：10.99.66.55

+   子网掩码：255.255.0.0（16）

1.  使用以下脚本共享互联网连接：

```
 #!/bin/bash 
        #filename: netsharing.sh

        echo 1 > /proc/sys/net/ipv4/ip_forward 

        iptables -A FORWARD -i $1 -o $2 \
            -s 10.99.0.0/16 -m conntrack --ctstate NEW -j ACCEPT 

        iptables -A FORWARD -m conntrack --ctstate \
            ESTABLISHED,RELATED -j ACCEPT

        iptables -A POSTROUTING -t nat -j MASQUERADE 

```

1.  运行脚本：

```
 ./netsharing.sh eth0 wlan0 

```

这里`eth0`是连接到互联网的接口，而`wlan0`是将共享互联网连接给其他设备的无线接口。

1.  将你的设备连接到刚刚创建的无线网络，设置如下：

    +   IP 地址：10.99.66.56（依此类推）

    +   子网掩码：255.255.0.0

为了方便起见，你可能希望在你的机器上安装 DHCP 和 DNS 服务器，这样就不需要手动配置设备的 IP 地址。一个方便的工具是`dnsmasq`，它同时执行 DHCP 和 DNS 操作。

# 它是如何工作的

有三个 IP 地址段是为非路由用途保留的。这意味着没有可见于互联网的网络接口可以使用这些地址。它们仅供本地内部网络中的机器使用。地址分别是`10.x.x.x`、`192.168.x.x`和`172.16.x.x-> 172.32.x.x`。在这个配方中，我们使用`10.x.x.x`地址空间的一部分作为我们的内部网络。

默认情况下，Linux 系统会接收或生成数据包，但不会回显它们。这由`in/proc/sys/net/ipv4/ip_forward`的值控制。

将`1`回显到该位置会告诉 Linux 内核转发它无法识别的任何数据包。这使得位于`10.99.66.x`子网中的无线设备能够使用`10.99.66.55`作为它们的网关。它们将把目标是互联网站点的数据包发送到`10.99.66.55`，然后通过`eth0`的网关将其转发到互联网，并路由到目标地址。

`iptables`命令是我们与 Linux 内核的 iptables 子系统交互的方式。这些命令添加规则，将所有内部网络的数据包转发到外部世界，并将预期的数据包从外部世界转发到我们的内部网络。

下一个示例将讨论更多使用 iptables 的方法。

# 使用 iptables 的基本防火墙

防火墙是用于过滤网络流量、阻止不需要的流量并允许所需流量通过的网络服务。Linux 的标准防火墙工具是`iptables`，它在最新版本中集成到内核中。

# 如何操作...

`iptables`在所有现代 Linux 发行版中默认存在。它易于为常见场景进行配置：

1.  如果你不想联系某个特定站点（例如，一个已知的恶意软件站点），你可以阻止到该 IP 地址的流量：

```
 #iptables -A OUTPUT -d 8.8.8.8 -j DROP 

```

如果你在另一个终端中使用`PING 8.8.8.8`，然后运行`iptables`命令，你将看到以下内容：

```
 PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data. 
 64 bytes from 8.8.8.8: icmp_req=1 ttl=56 time=221 ms 
 64 bytes from 8.8.8.8: icmp_req=2 ttl=56 time=221 ms 
 ping: sendmsg: Operation not permitted 
 ping: sendmsg: Operation not permitted 

```

在这里，ping 在第三次尝试时失败，因为我们使用了`iptables`命令来丢弃所有到`8.8.8.8`的流量。

1.  你也可以阻止到特定端口的流量：

```
 #iptables -A OUTPUT -p tcp -dport 21 -j DROP 
 $ ftp ftp.kde.org 
 ftp: connect: Connection timed out 

```

如果你在`/var/log/secure`或`/var/log/messages`文件中发现类似的消息，那你遇到了一个小问题：

```
 Failed password for abel from 1.2.3.4 port 12345 ssh2 
 Failed password for baker from 1.2.3.4 port 12345 ssh2 

```

这些消息意味着一个机器人正在探测你的系统以寻找弱密码。你可以通过一个 INPUT 规则来防止机器人访问你的网站，该规则会丢弃来自该网站的所有流量。

```
 #iptables -I INPUT -s 1.2.3.4 -j DROP 

```

# 如何工作...

`iptables`是用于配置 Linux 防火墙的命令。`iptables`中的第一个参数是-A，指示`iptables`将新规则附加到链中，或者是-I，它将新规则放在规则集的开始。下一个参数定义了链。链是规则的集合，在之前的示例中我们使用了`OUTPUT`链，它用于评估出站流量，而在最后的示例中我们使用了`INPUT`链，它用于评估入站流量。

`-d`参数指定了与发送的数据包匹配的目标地址，而`-s`则指定了数据包的源地址。最后，`-j`参数指示`iptables`跳转到特定的动作。在这些示例中，我们使用了 DROP 动作来丢弃数据包。其他动作包括`ACCEPT`和`REJECT`。

在第二个示例中，我们使用`-p`参数指定该规则仅匹配指定端口的 TCP 流量，这会阻止仅出站的`FTP`流量。

# 还有更多...

你可以使用`-flush`参数清除对`iptables`链所做的更改：

```
#iptables -flush 

```

# 创建虚拟私人网络

**虚拟专用网络**（**VPN**）是一个跨公共网络运行的加密通道。加密确保你的信息是私密的。VPN 用于连接远程办公室、分布式制造站点和远程工作人员。

我们已经讨论过使用`nc`、`scp`或`ssh`来复制文件。通过 VPN 网络，你可以通过 NFS 挂载远程驱动器，并像访问本地网络资源一样访问远程网络上的资源。

Linux 有多个 VPN 系统的客户端，并且支持 OpenVPN 的客户端和服务器。

本节的配方将描述如何设置 OpenVPN 服务器和客户端。此配方用于配置一个单一服务器来服务多个客户端，采用集线器和辐射式模型。OpenVPN 支持更多拓扑结构，超出了本章的范围。

# 准备工作

OpenVPN 并不是大多数 Linux 发行版的一部分。你可以使用包管理器安装它：

```
apt-get install openvpn 

```

另外，也可以使用以下命令：

```
yum install openvpn 

```

请注意，你需要在服务器和每个客户端上执行此操作。

确认隧道设备（`/dev/net/tun`）存在。在服务器和客户端系统上测试此项。在现代 Linux 系统上，隧道应该已经存在：

```
ls /dev/net/tun 

```

# 如何操作...

设置 OpenVPN 网络的第一步是为服务器和至少一个客户端创建证书。处理此问题的最简单方法是使用 OpenVPN 2.3 版本之前附带的`easy-rsa`包制作自签名证书。如果你使用的是 OpenVPN 的较新版本，可以通过包管理器获得`easy-rsa`。

这个包可能安装在`/usr/share/easy-rsa`中。

# 创建证书

首先，确保清理干净，之前的安装没有任何残留：

```
# cd /usr/share/easy-rsa 
# . ./vars 
# ./clean-all 

```

注意：如果你运行`./clean-all`，我会对`/usr/share/easy-rsa/keys`进行`rm -rf`操作。

接下来，使用`build-ca`命令创建**证书颁发机构**（CA）密钥。此命令会提示你输入有关你站点的信息。你需要多次输入这些信息。在此配方中，将你的名字、电子邮件、站点名称等替换为相应的值。不同命令所需的信息略有不同，只有唯一的部分会在这些配方中重复出现：

```
# ./build-ca 
Generating a 2048 bit RSA private key 
......+++ 
.....................................................+++ 
writing new private key to 'ca.key' 
----- 
You are about to be asked to enter information that will be incorporated 
into your certificate request. 
What you are about to enter is what is called a Distinguished Name or a DN. 
There are quite a few fields but you can leave some blank 
For somefieldsthere will be a default value, 
If you enter '.', the field will be left blank. 
----- 
Country Name (2 letter code) [US]: 
State or Province Name (full name) [CA]:MI 
Locality Name (eg, city) [SanFrancisco]:WhitmoreLake 
Organization Name (eg, company) [Fort-Funston]:Example 
Organizational Unit Name (eg, section) [MyOrganizationalUnit]:Packt 
Common Name (eg, your name or your server's hostname) [Fort-Funston CA]:vpnserver 
Name [EasyRSA]: 
Email Address [me@myhost.mydomain]:admin@example.com 

Next, build the server certificate with the build-key command: 
# ./build-key server 
Generating a 2048 bit RSA private key 
..................................+++ 
.....................+++ 
writing new private key to 'server.key' 
----- 
You are about to be asked to enter information that will be incorporated 
into your certificate request.... 

Please enter the following 'extra' attributes 
to be sent with your certificate request 
A challenge password []: 

```

为至少一个客户端创建证书。你需要为每个希望连接到此 OpenVPN 服务器的机器创建一个独立的客户端证书：

```
# ./build-key client1 
Generating a 2048 bit RSA private key 
.......................+++ 
.................................................+++ 
writing new private key to 'client1.key' 
----- 
You are about to be asked to enter information that will be incorporated 
into your certificate request. 
...  

Please enter the following 'extra' attributes 
to be sent with your certificate request 
A challenge password []: 
An optional company name []: 
Using configuration from /usr/share/easy-rsa/openssl-1.0.0.cnf 
Check that the request matches the signature 
Signature ok 
The Subject's Distinguished Name is as follows 
countryName  :PRINTABLE:'US' 
stateOrProvinceName  :PRINTABLE:'MI' 
localityName  :PRINTABLE:'WhitmoreLake' 
organizationName  :PRINTABLE:'Example' 
organizationalUnitName:PRINTABLE:'Packt' 
commonName  :PRINTABLE:'client1' 
name                  :PRINTABLE:'EasyRSA' 
emailAddress:IA5STRING:'admin@example.com' 
Certificate is to be certified until Jan  8 15:24:13 2027 GMT (3650 days) 
Sign the certificate? [y/n]:y 

1 out of 1 certificate requests certified, commit? [y/n]y 
Write out database with 1 new entries 
Data Base Updated 

```

最后，使用`build-dh`命令生成**Diffie-Hellman**密钥。这个过程需要几秒钟，并会生成几屏点和加号：

```
# ./build-dh 
Generating DH parameters, 2048 bit long safe prime, generator 2 
This is going to take a long time 
......................+............+........ 

```

这些步骤将会在密钥文件夹中创建多个文件。下一步是将它们复制到将要使用的文件夹中。

将服务器密钥复制到`/etc/openvpn`：

```
# cp keys/server* /etc/openvpn 
# cp keys/ca.crt /etc/openvpn 
# cp keys/dh2048.pem /etc/openvpn 

```

将客户端密钥复制到客户端系统：

```
# scp keys/client1* client.example.com:/etc/openvpn 
# scp keys/ca.crt client.example.com:/etc/openvpn 

```

# 配置 OpenVPN 服务器

OpenVPN 包括几乎可以直接使用的示例配置文件。你只需要根据你的环境自定义几行。这些文件通常位于`/usr/share/doc/openvpn/examples/sample-config-files`：

```
# cd /usr/share/doc/openvpn/examples/sample-config-files 
# cp server.conf.gz /etc/openvpn 
# cd /etc/openvpn 
# gunzip server.conf.gz 
# vim server.conf 

```

设置本地 IP 地址以供监听。这是连接到您希望允许 VPN 连接的网络上的网卡的 IP 地址：

```
local 192.168.1.125 
Modify the paths to the certificates: 

ca /etc/openvpn/ca.crt 
cert /etc/openvpn/server.crt 
key /etc/openvpn/server.key  # This file should be kept secret 

```

最后，检查`diffie-hellman`参数文件是否正确。OpenVPN 示例`config`文件可能指定了 1024 位长度的密钥，而`easy-rsa`创建的是 2048 位（更安全）的密钥。

```
#dh dh1024.pem 
dh dh2048.pem 

```

# 配置客户端上的 OpenVPN

每个客户端都有一套类似的配置。

将客户端配置文件复制到`/etc/openvpn`：

```
# cd /usr/share/doc/openvpn/examples/sample-config-files 
# cpclient.conf /etc/openvpn 

```

编辑`client.conf`文件：

```
# cd /etc/openvpn 
# vim client.conf 

```

更改证书路径，指向正确的文件夹：

```
ca /etc/openvpn/ca.crt 
cert /etc/openvpn/server.crt 
key /etc/openvpn/server.key  # This file should be kept secret 

```

设置服务器的远程站点：

```
#remote my-server-1 1194 
remote server.example.com 1194 

```

# 启动服务器

现在可以启动服务器。如果一切配置正确，您将看到几行输出。需要注意的关键行是`Initialization Sequence Completed`。如果没有该行，查看输出中是否有错误信息：

```
# openvpnserver.conf 
Wed Jan 11 12:31:08 2017 OpenVPN 2.3.4 x86_64-pc-linux-gnu [SSL (OpenSSL)] [LZO] [EPOLL] [PKCS11] [MH] [IPv6] built on Nov 12 2015 
Wed Jan 11 12:31:08 2017 library versions: OpenSSL 1.0.1t  3 May 2016, LZO 2.08... 

Wed Jan 11 12:31:08 2017 client1,10.8.0.4 
Wed Jan 11 12:31:08 2017 Initialization Sequence Completed 

```

使用`ifconfig`，您可以确认服务器正在运行。您应该看到隧道设备（tun）列出：

```
$ ifconfig 
tun0      Link encap:UNSPECHWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00   
inet addr:10.8.0.1  P-t-P:10.8.0.2  Mask:255.255.255.255 
 UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1 
 RX packets:0 errors:0 dropped:0 overruns:0 frame:0 
 TX packets:0 errors:0 dropped:0 overruns:0 carrier:0 
 collisions:0 txqueuelen:100 
 RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B) 

```

# 启动并测试客户端

一旦服务器运行，您可以启动客户端。与服务器一样，OpenVPN 客户端通过`openvpn`命令创建。同样，输出中的关键部分是`Initialization Sequence Completed`行：

```
# openvpn client.conf 
Wed Jan 11 12:34:14 2017 OpenVPN 2.3.4 i586-pc-linux-gnu [SSL (OpenSSL)] [LZO] [EPOLL] [PKCS11] [MH] [IPv6] built on Nov 19 2015 
Wed Jan 11 12:34:14 2017 library versions: OpenSSL 1.0.1t  3 May 2016, LZO 2.08... 

Wed Jan 11 12:34:17 2017 /sbin/ipaddr add dev tun0 local 10.8.0.6 peer 10.8.0.5 
Wed Jan 11 12:34:17 2017 /sbin/ip route add 10.8.0.1/32 via 10.8.0.5 
Wed Jan 11 12:34:17 2017 Initialization Sequence Completed 

```

使用`ifconfig`命令，您可以确认隧道已经初始化：

```
$ /sbin/ifconfig 

tun0      Link encap:UNSPECHWaddr 00-00-00-00-00-00-00-00...00-00-00-00   
inet addr:10.8.0.6  P-t-P:10.8.0.5  Mask:255.255.255.255 
 UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1 
 RX packets:2 errors:0 dropped:0 overruns:0 frame:0 
 TX packets:4 errors:0 dropped:0 overruns:0 carrier:0 
 collisions:0 txqueuelen:100 
 RX bytes:168 (168.0 B)  TX bytes:336 (336.0 B) 

```

使用`netstat`命令确认新网络的路由是否正确：

```
$ netstat -rn 
Kernel IP routing table 
Destination   Gateway       Genmask         Flags   MSS Window  irttIface 
0.0.0.0       192.168.1.7   0.0.0.0         UG        0 0          0 eth0 
10.8.0.1      10.8.0.5      255.255.255.255 UGH       0 0          0 tun0 
10.8.0.5      0.0.0.0       255.255.255.255 UH        0 0          0 tun0 
192.168.1.0   0.0.0.0       255.255.255.0   U         0 0          0 eth0 

```

该输出显示了隧道设备已连接到`10.8.0.x`网络，网关是`10.8.0.1`。

最后，您可以使用`ping`命令测试连接性：

```
$ ping 10.8.0.1 
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data. 
64 bytes from 10.8.0.1: icmp_seq=1 ttl=64 time=1.44 ms 

```
