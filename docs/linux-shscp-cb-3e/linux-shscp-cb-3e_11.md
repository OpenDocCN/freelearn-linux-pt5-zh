# 跟踪线索

本章将涉及以下主题：

+   使用`tcpdump`跟踪数据包

+   使用`ngrep`查找数据包

+   使用`ip`跟踪网络路由

+   使用`strace`跟踪系统调用

+   使用`ltrace`跟踪动态库函数

# 介绍

没有跟踪，什么也不会发生。在 Linux 系统中，我们可以通过第九章中讨论的日志文件来跟踪事件，*佩戴显示器帽*。`top`命令显示哪些程序使用了最多的 CPU 时间，`watch`、`df`和`du`让我们监控磁盘使用情况。

本章将描述获取更多关于网络数据包、CPU 使用情况、磁盘使用情况和动态库调用的信息的方法。

# 使用 tcpdump 跟踪数据包

仅知道哪些应用程序正在使用某个端口，可能不足以追踪问题。有时你还需要检查正在传输的数据。

# 准备工作

你需要是 root 用户才能运行`tcpdump`。`tcpdump`应用程序可能在你的系统中没有默认安装，因此需要通过包管理器安装：

```
$ sudo apt-get install tcpdump
$ sudo yum install libpcap tcpdump

```

# 如何做到……

`tcpdump`应用程序是 Wireshark 和其他网络嗅探程序的前端。图形界面支持我们将很快描述的许多选项。

该应用程序的默认行为是显示在主以太网链路上看到的每个数据包。数据包报告的格式如下：

```
    TIMESTAMP SRC_IP:PORT > DEST_IP:PORT: NAME1 VALUE1, NAME2 VALUE2,...

```

名称-值对包括：

+   `Flags`：与此数据包相关的标志如下：

+   +   `S`代表**SYN**（**开始连接**）

    +   `F`代表**FIN**（**结束连接**）

    +   `P`代表**PUSH**（**推送数据**）

    +   `R`代表**RST**（**重置连接**）

    +   句号`.`表示没有标志

+   `seq`：这指的是数据包的序列号。它将在 ACK 中回显，以识别被确认的数据包。

+   `ack`：这指的是确认，表示已收到数据包。其值为来自先前数据包的序列号。

+   `win`：这表示目的地缓冲区的大小。

+   `options`：这指的是为此数据包定义的 TCP 选项。它以逗号分隔的键值对形式报告。

以下输出显示了来自 Windows 计算机到 SAMBA 服务器的请求，夹杂着 DNS 请求。不同源和应用程序的数据包交错，使得追踪特定应用程序或主机上的流量变得困难。然而，`tcpdump`命令有一些标志，可以让我们的生活变得更轻松：

```
$ tcpdump 
22:00:25.269277 IP 192.168.1.40.49182 > 192.168.1.2.microsoft-ds: Flags [P.], seq 3265172834:3265172954, ack 850195805, win 257, length 120SMB PACKET: SMBtrans2 (REQUEST) 
22:00:25.269417 IP 192.168.1.44.33150 > 192.168.1.7.domain: 13394+ PTR? 2.1.168.192.in-addr.arpa. (42) 
22:00:25.269917 IP 192.168.1.2.microsoft-ds > 192.168.1.40.49182: Flags [.], ack 120, win 1298, length 0 
22:00:25.269927 IP 192.168.1.2.microsoft-ds > 192.168.1.40.49182: Flags [P.], seq 1:105, ack 120, win 1298, length 104SMB PACKET: SMBtrans2 (REPLY)

```

`-w`标志将`tcpdump`的输出发送到文件，而不是终端。输出格式为二进制形式，可以通过`-r`标志读取。嗅探数据包必须以 root 权限进行，但从先前保存的文件显示结果可以作为普通用户进行。

默认情况下，`tcpdump`运行并收集数据，直到通过 Ctrl-C 或**SIGTERM**结束。`-c`标志限制数据包的数量：

```
# tcpdump -w /tmp/tcpdump.raw -c 50
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
50 packets captured
50 packets received by filter
0 packets dropped by kernel

```

通常，我们希望检查单个主机上的活动，可能是某个特定的应用程序。

`tcpdump` 命令行的最后几个值构成了一个表达式，帮助我们过滤数据包。这个表达式是由带有修饰符和布尔运算符的键值对组成的。接下来的示例演示了如何使用过滤器。

# 只显示 HTTP 数据包

`port` 键只显示发送到或从指定端口的数据包：

```
$ tcpdump -r /tmp/tcpdump.raw port http
reading from file /tmp/tcpdump.raw, link-type EN10MB (Ethernet)
10:36:50.586005 IP 192.168.1.44.59154 > ord38s04-in-f3.1e100.net.http: Flags [.], ack 3779320903, win 431, options [nop,nop,TS val 2061350532 ecr 3014589802], length 0

10:36:50.586007 IP ord38s04-in-f3.1e100.net.http > 192.168.1.44.59152: Flags [.], ack 1, win 350, options [nop,nop,TS val 3010640112 ecr 2061270277], length 0

```

# 只显示由该主机生成的 HTTP 数据包

如果你尝试追踪网络上的网页使用情况，可能只需要查看你网站上生成的数据包。`src` 修饰符仅指定这些数据包，以及给定的值，来自源文件。`dst` 修饰符则指定目的地：

```
$ tcpdump -r /tmp/tcpdump.raw src port http
reading from file /tmp/tcpdump.raw, link-type EN10MB (Ethernet)

10:36:50.586007 IP ord38s04-in-f3.1e100.net.http > 192.168.1.44.59152: Flags [.], ack 1, win 350, options [nop,nop,TS val 3010640112 ecr 2061270277], length 0
10:36:50.586035 IP ord38s04-in-f3.1e100.net.http > 192.168.1.44.59150: Flags [.], ack 1, win 350, options [nop,nop,TS val 3010385005 ecr 2061270277], length 0

```

# 查看数据包的有效载荷以及头部

如果你需要追踪某个淹没网络的主机，你只需要头部信息。如果你正在调试一个 web 或数据库应用程序，你可能需要查看数据包的内容以及头部信息。

`-X` 标志将把数据包数据包含在输出中。

`host` 关键字可以与端口信息结合，限制报告仅显示往返于指定主机的数据。

这两个测试用 **and** 连接，执行布尔 **and** 操作，并且仅报告那些来自或去往 noucorp.com 和/或 `HTTP` 服务器的数据包。示例输出显示了一个 `GET` 请求的开始以及服务器的回复：

```
$ tcpdump -X -r /tmp/tcpdump.raw host noucorp.com and port http
reading from file /tmp/tcpdump.raw, link-type EN10MB (Ethernet)
11:12:04.708905 IP 192.168.1.44.35652 > noucorp.com.http: Flags [P.], seq 2939551893:2939552200, ack 1031497919, win 501, options [nop,nop,TS val 2063464654 ecr 28236429], length 307
 0x0000:  4500 0167 1e54 4000 4006 70a5 c0a8 012c  E..g.T@.@.p....,
 0x0010:  98a0 5023 8b44 0050 af36 0095 3d7b 68bf  ..P#.D.P.6..={h.
 0x0020:  8018 01f5 abf1 0000 0101 080a 7afd f8ce  ............z...
 0x0030:  01ae da8d 4745 5420 2f20 4854 5450 2f31  ....GET./.HTTP/1
 0x0040:  2e31 0d0a 486f 7374 3a20 6e6f 7563 6f72  .1..Host:.noucor
 0x0050:  702e 636f 6d0d 0a55 7365 722d 4167 656e  p.com..User-Agen
 0x0060:  743a 204d 6f7a 696c 6c61 2f35 2e30 2028  t:.Mozilla/5.0.(
 0x0070:  5831 313b 204c 696e 7578 2078 3836 5f36  X11;.Linux.x86_6
 0x0080:  343b 2072 763a 3435 2e30 2920 4765 636b  4;.rv:45.0).Geck
 0x0090:  6f2f 3230 3130 3031 3031 2046 6972 6566  o/20100101.Firef
 0x00a0:  6f78 2f34 352e 300d 0a41 6363 6570 743a  ox/45.0..Accept:
...
11:12:04.731343 IP noucorp.com.http > 192.168.1.44.35652: Flags [.], seq 1:1449, ack 307, win 79, options [nop,nop,TS val 28241838 ecr 2063464654], length 1448
 0x0000:  4500 05dc 0491 4000 4006 85f3 98a0 5023  E.....@.@.....P#
 0x0010:  c0a8 012c 0050 8b44 3d7b 68bf af36 01c8  ...,.P.D={h..6..
 0x0020:  8010 004f a7b4 0000 0101 080a 01ae efae  ...O............
 0x0030:  7afd f8ce 4854 5450 2f31 2e31 2032 3030  z...HTTP/1.1.200
 0x0040:  2044 6174 6120 666f 6c6c 6f77 730d 0a44  .Data.follows..D
 0x0050:  6174 653a 2054 6875 2c20 3039 2046 6562  ate:.Thu,.09.Feb
 0x0060:  2032 3031 3720 3136 3a31 323a 3034 2047  .2017.16:12:04.G
 0x0070:  4d54 0d0a 5365 7276 6572 3a20 5463 6c2d  MT..Server:.Tcl-
 0x0080:  5765 6273 6572 7665 722f 332e 352e 3220  Webserver/3.5.2.

```

# 工作原理...

`tcpdump` 应用程序设置了一个混杂标志，导致网络接口卡（NIC）将所有数据包传递给处理器。它这样做，而不是仅过滤那些与该主机相关的数据包。这个标志允许记录主机所在物理网络上的任何数据包，而不仅仅是传送到该主机的数据包。

这个应用程序用于追踪超负荷网络段、生成意外流量的主机、网络环路、故障的 NIC、格式错误的数据包等问题。

使用 `-w` 和 `-r` 选项，`tcpdump` 会以原始格式保存数据，允许你稍后以常规用户身份查看它。例如，如果凌晨 3 点网络数据包发生了大量碰撞，你可以设置一个 `cron` 任务，在凌晨 3 点运行 `tcpdump`，然后在正常工作时间查看数据。

# 使用 ngrep 查找数据包

`ngrep` 应用程序是 `grep` 和 `tcpdump` 的结合体。它监视网络端口并显示与模式匹配的数据包。你必须拥有 root 权限才能运行 `ngrep`。

# 准备工作

你可能没有安装 `ngrep` 包。不过，你可以通过大多数包管理器安装它：

```
# apt-get install ngrep
# yum install ngrep

```

# 如何操作...

`ngrep` 应用程序接受一个模式进行匹配（如同 `grep`），一个过滤字符串（如同 `tcpdump`），以及许多命令行标志来微调其行为。

以下示例监视 `80` 端口的流量，并报告任何包含字符串 `Linux` 的数据包：

```
$> ngrep -q -c 64 Linux port 80
interface: eth0 (192.168.1.0/255.255.255.0)
filter: ( port 80 ) and (ip or ip6)
match: Linux

T 192.168.1.44:36602 -> 152.160.80.35:80 [AP]
 GET /Training/linux_detail/ HTTP/1.1..Host: noucorp.com..Us
 er-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:45.0) Gecko/20
 100101 Firefox/45.0..Accept: text/html,application/xhtml+xm
 l,application/xml;q=0.9,*/*;q=0.8..Accept-Language: en-US,e
 n;q=0.5..Accept-Encoding: gzip, deflate..Referer: http://no
 ucorp.com/Training/..Connection: keep-alive..Cache-Control:
 max-age=0.... 

```

`-q` 标志使 `ngrep` 只打印头部和有效载荷。

`-c` 标志定义了用于负载数据的列数。默认情况下，列数是四，这对于基于文本的数据包来说并不实用。

在标志之后是匹配字符串（Linux），然后是使用与 `tcpdump` 相同过滤语言的过滤器表达式。

# 它是如何工作的...

`ngrep` 应用程序还设置了混杂模式标志，允许它嗅探所有可见的数据包，无论这些数据包是否与主机相关。

上面的例子展示了所有的 HTTP 流量。如果主机系统连接在无线网络或通过集线器（而非交换机）进行有线连接，它会显示所有活跃用户造成的所有 Web 流量。

# 还有更多...

`ngrep` 中的 `-x` 选项显示十六进制转储和可打印形式。将其与 `-X` 结合使用，可以搜索二进制字符串（例如病毒签名或某些已知模式）。

这个例子监视来自 HTTPS 连接的二进制流：

```
# ngrep -xX '1703030034' port 443
interface: eth0 (192.168.1.0/255.255.255.0)
filter: ( port 443 ) and (ip or ip6)
match: 0x1703030034
#################################################
T 172.217.6.1:443 -> 192.168.1.44:40698 [AP]
 17 03 03 00 34 00 00 00    00 00 00 00 07 dd b0 02    ....4...........
 f5 38 07 e8 24 08 eb 92    3c c6 66 2f 07 94 8b 25    .8..$...<.f/...%
 37 b3 1c 8d f4 f0 64 c3    99 9e b3 45 44 14 64 23    7.....d....ED.d#
 80 85 1b a1 81 a3 d2 7a    cd                         .......z.

```

哈希标记表示已扫描的数据包；它们不包括目标模式。`ngrep` 有许多其他选项；请阅读 `man` 页面查看完整列表。

# 使用 ip 路由追踪网络

`ip` 工具报告有关网络状态的信息。它可以告诉你有多少数据包正在发送和接收，发送的是哪种类型的数据包，数据包是如何路由的，等等。

# 准备工作

在 第八章 中描述的 `netstat` 工具，*老式网络*，是所有 Linux 发行版的标准工具；然而，它现在正被更高效的工具所取代，例如 `ip`。这些新工具包含在 `iproute2` 包中，已经安装在大多数现代发行版中。

# 如何做...

`ip` 工具有很多功能。本节将讨论一些在追踪网络行为时非常有用的功能。

# 使用 ip route 报告路由

当数据包无法到达目标（`ping` 或 `traceroute` 失败）时，一个有经验的用户首先检查的是电缆。接下来要检查的是路由表。如果系统缺少默认网关（`0.0.0.0`），它只能找到物理网络上的机器。如果你有多个网络在同一条线路上运行，你需要添加路由以允许连接到一个网络的机器向另一个网络发送数据包。

`ip route` 命令报告已知的路由：

```
$ ip route
10.8.0.2 dev tun0  proto kernel  scope link  src 10.8.0.1 
192.168.87.0/24 dev vmnet1  proto kernel  scope link  src 192.168.87.1 
192.168.1.0/24 dev eth0  proto kernel  scope link  src 192.168.1.44 
default via 192.168.1.1 dev eth0  proto static 

```

`ip route` 报告是以空格分隔的。在第一个元素之后，它由一组键值对组成。

上面的代码中的第一行描述了 `10.8.0.2` 地址作为一个使用内核协议的隧道设备，该地址仅在这个隧道设备上有效。第二行描述了 `192.168.87.x` 网络，用于与虚拟机通信。第三行是该系统的主网络，它连接到 `/dev/eth0`。最后一行定义了默认路由，通过 `eth0` 路由到 `192.168.1.1`。

`ip route` 报告的键包括以下内容：

+   `via`：指的是下一个跳跃的地址。

+   `proto`：这是路由的协议标识符。内核协议是由内核安装的路由，而静态路由由管理员定义。

+   `scope`：这是指地址有效的作用范围。链路作用域仅在此设备上有效。

+   `dev`：这是与该地址关联的设备。

# 追踪最近的 IP 连接和 ARP 表

`ip neighbor` 命令报告已知的 IP 地址、设备和硬件 MAC 地址之间的关系。它报告该关系是否已重新建立，或者是否已经过时：

```
$> ip neighbor
192.168.1.1 dev eth0 lladdr 2c:30:33:c9:af:3e STALE
192.168.1.4 dev eth0 lladdr 00:0a:e6:11:c7:dd STALE
172.16.183.138 dev vmnet8 lladdr 00:50:56:20:3d:6c STALE

192.168.1.2 dev eth0 lladdr 6c:f0:49:cd:45:ff REACHABLE

```

`ip neighbor` 命令的输出显示当前系统与默认网关之间，或者当前系统与 `192.168.1.4` 之间没有最近的活动。它还显示虚拟机与 `192.168.1.2` 之间没有最近活动，而该主机最近连接。

上述输出中的 `REACHABLE` 状态意味着 `arp` 表是最新的，并且主机认为它知道远程系统的 MAC 地址。此处的 `STALE` 值并不表示系统不可达，它仅仅表示 `arp` 表中的值已过期。当系统尝试使用这些路由时，它首先会发送 ARP 请求以验证与 IP 地址关联的 MAC 地址。

MAC 地址与 IP 地址之间的关系应仅在硬件更换或设备重新分配时发生变化。

如果网络上的设备显示间歇性连接，可能意味着两个设备被分配了相同的 IP 地址。也可能是两个 DHCP 服务器正在运行，或者有人手动分配了一个已经在使用的地址。

当两个设备具有相同的 IP 地址时，给定 IP 地址的 MAC 地址会间歇性变化，`ip neighbor` 命令可以帮助追踪错误配置的设备。

# 路由追踪

在 第八章《老男孩网络》中讨论的 `traceroute` 命令追踪从当前主机到目标的整个数据包路径。`route get` 命令报告当前机器的下一跳：

```
$ ip route get 172.16.183.138
172.16.183.138 dev vmnet8 src 172.16.183.1
cache mtu 1500 hoplimit 64

```

上述返回结果显示通往虚拟机的路由通过位于 `172.16.183.1` 的 vmnet8 接口。发送到此站点的数据包如果大于 1,500 字节，将被分割，并且在 64 跳后丢弃：

```
$ in route get 148.59.87.90
148.59.87.90 via 192.168.1.1 dev eth0 src 192.168.1.3
cache mtu 1500 hoplimit 64

```

要访问 Internet 上的一个地址，数据包需要通过默认网关离开本地网络，通往该网关的连接是主机的 `eth0` 设备，地址为 `192.168.1.3`。

# 它是如何工作的...

`ip` 命令在用户空间运行并与内核表接口。使用此命令，普通用户可以检查网络配置，而超级用户可以配置网络。

# 使用 strace 跟踪系统调用

一台 GNU/Linux 计算机可能同时运行数百个任务，但它只会拥有一个网络接口、一个磁盘驱动器、一个键盘，等等。Linux 内核分配这些有限的资源并控制任务如何访问它们。这可以防止两个任务在磁盘文件中不小心混淆数据，例如。

当你运行一个应用程序时，它会使用**用户空间库**（如`printf`和`fopen`）和系统空间库（如`write`和`open`）的组合。当你的程序调用`printf`（或者脚本调用`echo`命令）时，它会调用一个用户空间库的`printf`来格式化输出字符串；接着会调用系统空间的`write`函数。系统调用确保只有一个任务可以同时访问一个资源。

在一个完美的世界里，所有计算机程序都能无故障地运行。在一个几乎完美的世界里，你会有源代码，程序会使用调试支持进行编译，并且它会始终一致地出错。

在现实世界中，你有时不得不应对一些没有源代码、且间歇性失败的程序。除非你提供一些数据给开发者，否则他们无法帮你解决问题。

Linux 的`strace`命令报告应用程序所做的系统调用；即使我们没有源代码，它也能帮助我们理解程序在做什么。

# 准备工作

`strace`命令作为开发者包的一部分安装；也可以单独安装：

```
$ sudo apt-get install strace
$ sudo yum install strace

```

# 如何做到这一点……

理解`strace`的一种方式是编写一个简短的 C 程序，并使用`strace`查看它调用了哪些系统调用。

这个测试程序分配内存，使用内存，打印一条简短的消息，释放内存，并退出。

`strace`输出显示了这个程序调用的系统函数：

```
$ cat test.c  
#include <stdio.h> 
#include <stdlib.h> 
#include <string.h> 

main () { 
  char *tmp; 
  tmp=malloc(100); 
  strcat(tmp, "testing"); 
  printf("TMP: %s\n", tmp); 
  free(tmp); 
  exit(0); 
} 
$ gcc test.c 
$ strace ./a.out 
execve("./a.out", ["./a.out"], [/* 51 vars */]) = 0 
brk(0)                                  = 0x9fc000 
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc85c7f5000 
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory) 
open("/etc/ld.so.cache", O_RDONLY)      = 3 
fstat(3, {st_mode=S_IFREG|0644, st_size=95195, ...}) = 0 
mmap(NULL, 95195, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7fc85c7dd000 
close(3)                                = 0 
open("/lib64/libc.so.6", O_RDONLY)      = 3 
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0000\356\1\16;\0\0\0"..., 832) = 832 
fstat(3, {st_mode=S_IFREG|0755, st_size=1928936, ...}) = 0 
mmap(0x3b0e000000, 3750184, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x3b0e000000 
mprotect(0x3b0e18a000, 2097152, PROT_NONE) = 0 
mmap(0x3b0e38a000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x18a000) = 0x3b0e38a000 
mmap(0x3b0e390000, 14632, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x3b0e390000 
close(3)                                = 0 
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc85c7dc000 
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc85c7db000 
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc85c7da000 
arch_prctl(ARCH_SET_FS, 0x7fc85c7db700) = 0 
mprotect(0x3b0e38a000, 16384, PROT_READ) = 0 
mprotect(0x3b0de1f000, 4096, PROT_READ) = 0 
munmap(0x7fc85c7dd000, 95195)           = 0 
brk(0)                                  = 0x9fc000 
brk(0xa1d000)                           = 0xa1d000 
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 11), ...}) = 0 
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7fc85c7f4000 
write(1, "TMP: testing\n", 13)          = 13 
exit_group(0)                           = ? 
+++ exited with 0 +++ 

```

# 它是如何工作的……

前几行是任何应用程序的标准启动命令。`execve`调用是初始化新可执行文件的系统调用。`brk`调用返回当前内存地址，`mmap`调用为动态库和其他加载的程序分配了 4,096 字节的内存。

尝试访问`ld.so.preload`失败，因为`ld.so.preload`是一个用于预加载库的钩子。大多数生产系统中并不需要它。

`ld.so.cache`文件是`/etc/ld.so.conf.d`的内存驻留副本，包含了加载动态库的路径。这些值保存在内存中，以减少启动程序时的开销。

接下来的几行包含`mmap`、`mprotect`、`arch_prctl`和`munmap`调用，继续加载库并将设备映射到内存中。

这两个`brk`调用是由程序的`malloc`调用触发的。它从堆中分配了 100 字节的内存。

`strcat`调用是一个用户空间函数，不会生成任何系统调用。

`printf`调用不会生成系统调用来格式化数据，但它会发起调用将格式化后的字符串发送到`stdout`。

`fstat`和`mmap`调用加载并初始化`stdout`设备。这些调用在生成输出到`stdout`的程序中只会发生一次。

`write`系统调用将字符串发送到`stdout`。

最后，`exit_group`调用退出程序，释放资源，并终止与可执行文件相关的所有线程。

请注意，没有与释放内存相关联的`brk`调用。`malloc`和`free`函数是用户空间函数，管理任务的内存。只有在程序的整体内存占用发生变化时，它们才会调用`brk`函数。当程序分配*N*字节时，它需要将这么多字节添加到可用内存中。当它释放这个块时，内存被标记为可用，但它仍然是这个程序内存池的一部分。接下来的`malloc`会使用来自可用内存池的内存空间，直到内存池耗尽。此时，另一个`brk`调用会将更多内存添加到程序的内存池中。

# 使用 ltrace 跟踪动态库函数

知道调用了哪些用户空间的库函数和知道调用了哪些系统函数一样有用。`ltrace`命令提供了类似于`strace`的功能；不过，它跟踪的是用户空间的库函数调用，而不是系统调用。

# 准备就绪

安装开发工具以使用`ltrace`命令。

# 如何操作...

要跟踪用户空间的动态库调用，执行`strace`命令，然后是你想要跟踪的命令：

```
$ ltrace myApplication

```

下一个示例是带有子例程的程序：

```
$ cat test.c 
#include <stdio.h> 
#include <stdlib.h> 
#include <string.h> 

int print (char *str) { 
  printf("%s\n", str); 
} 
main () { 
  char *tmp; 
  tmp=malloc(100); 
  strcat(tmp, "testing"); 
  print(tmp); 
  free(tmp); 
  exit(0); 
} 
$ gcc test.c 
$ ltrace ./a.out 
(0, 0, 603904, -1, 0x1f25bc2)                            = 0x3b0de21160 
__libc_start_main(0x4005fe, 1, 0x7ffd334a95f8, 0x400660, 0x400650 <unfinished ...> 
malloc(100)                                              = 0x137b010 
strcat("", "testing")                                    = "testing" 
puts("testing")                                          = 8 
free(0x137b010)                                          = <void> 
exit(0 <unfinished ...> 
+++ exited (status 0) +++ 

```

在`ltrace`输出中，我们看到了对动态链接的`strcat`的调用；但是我们没有看到静态链接的本地函数，即`print`。`printf`的调用被简化为对`puts`的调用。`malloc`和`free`的调用被显示出来，因为它们是用户空间函数调用。

# 它是如何工作的...

`ltrace`和`strace`工具使用`ptrace`函数来重写**过程链接表**（**PLT**），该表将动态库调用映射到被调用函数的实际内存地址。这意味着`ltrace`可以捕获任何动态链接的函数调用，但不能捕获静态链接的函数。

# 还有更多内容...

`ltrace`和`strace`命令非常有用，但如果能够同时跟踪用户空间和系统空间的函数调用，那就更好了。`ltrace`的`-S`选项可以做到这一点。下一个示例显示了从之前的可执行文件中获取的`ltrace -S`输出：

```
$> ltrace -S ./a.out
SYS_brk(NULL)                                            = 0xa9f000
SYS_mmap(0, 4096, 3, 34, 0xffffffff)                     = 0x7fcdce4ce000
SYS_access(0x3b0dc1d380, 4, 0x3b0dc00158, 0, 0)          = -2
SYS_open("/etc/ld.so.cache", 0, 01)                      = 4
SYS_fstat(4, 0x7ffd70342bc0, 0x7ffd70342bc0, 0, 0xfefefefefefefeff) = 0
SYS_mmap(0, 95195, 1, 2, 4)                              = 0x7fcdce4b6000
SYS_close(4)                                             = 0
SYS_open("/lib64/libc.so.6", 0, 00)                      = 4
SYS_read(4, "\177ELF\002\001\001\003", 832)              = 832
SYS_fstat(4, 0x7ffd70342c20, 0x7ffd70342c20, 4, 0x7fcdce4ce640) = 0
SYS_mmap(0x3b0e000000, 0x393928, 5, 2050, 4)             = 0x3b0e000000
SYS_mprotect(0x3b0e18a000, 0x200000, 0, 1, 4)            = 0
SYS_mmap(0x3b0e38a000, 24576, 3, 2066, 4)                = 0x3b0e38a000
SYS_mmap(0x3b0e390000, 14632, 3, 50, 0xffffffff)         = 0x3b0e390000
SYS_close(4)                                             = 0
SYS_mmap(0, 4096, 3, 34, 0xffffffff)                     = 0x7fcdce4b5000
SYS_mmap(0, 4096, 3, 34, 0xffffffff)                     = 0x7fcdce4b4000
SYS_mmap(0, 4096, 3, 34, 0xffffffff)                     = 0x7fcdce4b3000
SYS_arch_prctl(4098, 0x7fcdce4b4700, 0x7fcdce4b3000, 34, 0xffffffff) = 0
SYS_mprotect(0x3b0e38a000, 16384, 1, 0x3b0de20fd8, 0x1f25bc2) = 0
SYS_mprotect(0x3b0de1f000, 4096, 1, 0x4003e0, 0x1f25bc2) = 0
(0, 0, 987392, -1, 0x1f25bc2)                            = 0x3b0de21160
SYS_munmap(0x7fcdce4b6000, 95195)                        = 0
__libc_start_main(0x4005fe, 1, 0x7ffd703435c8, 0x400660, 0x400650 <unfinished ...>
malloc(100 <unfinished ...>
SYS_brk(NULL)                                            = 0xa9f000
SYS_brk(0xac0000)                                        = 0xac0000
<... malloc resumed> )                                   = 0xa9f010
strcat("", "testing")                                    = "testing"
puts("testing" <unfinished ...>
SYS_fstat(1, 0x7ffd70343370, 0x7ffd70343370, 0x7ffd70343230, 0x3b0e38f040) = 0
SYS_mmap(0, 4096, 3, 34, 0xffffffff)                     = 0x7fcdce4cd000
SYS_write(1, "testing\n", 8)                             = 8
<... puts resumed> )                                     = 8
free(0xa9f010)                                           = <void>
exit(0 <unfinished ...>
SYS_exit_group(0 <no return ...>
+++ exited (status 0) +++

```

这与`strace`示例显示的启动调用（如`sbrk`、`mmap`等）相同。

当一个用户空间的函数调用了系统空间的函数（如`malloc`和 puts 调用）时，显示会显示该用户空间函数被中断（`malloc(100 <unfinished...>)`），并在系统调用完成后恢复（`(<... malloc resumed>)`）。

请注意，`malloc`调用需要将控制权传递给`sbrk`，以便为应用程序分配更多内存。然而，`free`调用不会缩小应用程序的内存，它只是释放内存供未来该应用程序使用。
