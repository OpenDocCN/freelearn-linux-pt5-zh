# 第十二章：调优 Linux 系统

本章将涵盖以下内容：

+   确定服务

+   使用`ss`收集套接字数据

+   使用`dstat`收集系统 I/O 使用情况

+   使用`pidstat`识别资源消耗大户

+   使用`sysctl`调整 Linux 内核

+   使用配置文件调优 Linux 系统

+   使用`nice`命令改变调度器优先级

# 简介

没有任何系统能以我们希望的速度运行，任何计算机的性能都可以提升。

我们可以通过关闭未使用的服务、调整内核参数或添加新硬件来提高系统的性能。

调优系统的第一步是理解需求是什么，以及这些需求是否得到了满足。不同类型的应用有不同的关键需求。你需要问自己以下问题：

+   对于该系统，CPU 是否是关键资源？一个进行工程模拟的系统比其他资源更需要 CPU 周期。

+   对于该系统，网络带宽是否至关重要？一个文件服务器几乎不进行计算，但可能会使其网络容量饱和。

+   对于该系统，磁盘访问速度是否至关重要？文件服务器或数据库服务器对磁盘的需求要比计算引擎高。

+   对于该系统，RAM 是否是关键资源？所有系统都需要 RAM，但数据库服务器通常会构建大型内存表以执行查询，而文件服务器则通过更大的内存来提高磁盘缓存的效率。

+   你的系统是否被黑客攻击了？系统可能会突然变得无响应，因为它运行了意外的恶意软件。虽然这在 Linux 机器上不常见，但拥有众多用户的系统（例如大学或企业网络）容易受到暴力破解密码攻击。

接下来的问题是：我如何衡量系统的使用情况？了解一个系统的使用方式能帮助你提出问题，但可能不会直接给出答案。一个文件服务器会将常访问的文件缓存到内存中，因此内存不足的服务器可能会受到磁盘/内存的限制，而非网络限制。

Linux 有一些用于分析系统的工具。许多工具已经在第八章，*老男孩网络*，第九章，*戴上监视器的帽子*，和第十一章，*追踪线索*中讨论过了。本章将介绍更多的监控工具。

以下是需要检查的子系统和工具列表。书中已经讨论了这些工具中的许多（但并非全部）。

+   CPU：`top`，`dstat`，`perf`，`ps`，`mpstat`，`strace`，`ltrace`

+   网络：`netstat`，`ss`，`iotop`，`ip`，`iptraf`，`nicstat`，`ethtool`，`lsof`

+   磁盘：`ftrace`，`iostat`，`dstat`，`blktrace`

+   内存：top，`dstat`，`perf`，`vmstat`，`swapon`

这些工具中的许多是标准 Linux 发行版的一部分。其他工具可以通过包管理器加载。

# 确定服务

一个 Linux 系统可以同时运行数百个任务。大部分任务是操作系统环境的一部分，但你可能会发现有些守护进程是你不需要的。

Linux 发行版支持三种用于启动守护进程和服务的工具之一。传统的`SysV`系统使用位于`/etc/init.d`的脚本。较新的`systemd`守护进程使用相同的`/etc/init.d`脚本，并且还使用`systemctl`调用。一些发行版使用 Upstart，它将配置脚本存储在`/etc/init`中。

SysV `init`系统正在逐步淘汰，取而代之的是`systemd`套件。`upstart`工具是由 Ubuntu 开发并使用的，但在 14.04 版本中被抛弃，改为使用`systemd`。本章将重点介绍`systemd`，因为它是大多数发行版使用的系统。

# 准备工作

第一步是确定你的系统使用的是 SysV `init`调用、`systemd`还是`upstart`。

Linux/Unix 系统必须有一个初始化进程作为`PID 1`运行。这个进程执行一个 fork 和 exec 来启动其他所有进程。`ps`命令可以告诉你哪个初始化进程正在运行：

```
    $ ps -p 1 -o cmd
 /lib/system/systemd

```

在上面的例子中，系统肯定正在运行`systemd`。然而，在一些发行版中，SysV `init`程序会被`符号链接`到实际的`init`进程，`ps`命令总是显示`/sbin/init`，无论实际上是使用的 SysV `init`、`upstart`还是`systemd`：

```
    $ ps -p 1 -o cmd
 /sbin/init

```

`ps`和`grep`命令提供了更多线索：

```
    $ ps -eaf | grep upstart

```

或者，它们也可以这样使用：

```
    ps -eaf | grep systemd 

```

如果这些命令返回的任务包括`upstart-udev-bridge`或`systemd/systemd`，则系统分别正在运行`upstart`或`systemd`。如果没有匹配项，则系统可能正在运行 SysV `init`工具。

# 如何操作...

`service`命令在大多数发行版中得到支持。`-status-all`选项将报告`/etc/init.d`中定义的所有服务的当前状态。不同发行版之间的输出格式有所不同：

```
    $> service -status-all

```

Debian：

```
 [ + ]  acpid
 [ - ]  alsa-utils
 [ - ]  anacron
 [ + ]  atd
 [ + ]  avahi-daemon
 [ - ]  bootlogs
 [ - ]  bootmisc.sh
...

```

CentOS：

```
abrt-ccpp hook is installed
abrtd (pid  4009) is running...
abrt-dump-oops is stopped
acpid (pid  3674) is running...
atd (pid  4056) is running...
auditd (pid  3029) is running...
...

```

`grep`命令将输出结果过滤为仅显示正在运行的任务：

Debian：

```
    $ service -status-all | grep +

```

CentOS：

```
    $ service -status-all | grep running

```

你应该禁用任何不必要的服务。这可以减轻系统负担并提高系统安全性。

需要检查的服务包括以下内容：

+   `smbd`、`nmbd`：这些是用于在 Linux 和 Windows 系统之间共享资源的 Samba 守护进程。

+   `telnet`：这是一个古老的、不安全的登录程序。除非有强烈的需求，否则使用 SSH。

+   `ftp`：这是一个古老的、不安全的文件传输协议。应使用 SSH 和 scp 代替。

+   `rlogin`：这是远程登录。SSH 更加安全。

+   `rexec`：这是远程执行。SSH 更加安全。

+   `automount`：如果你不使用 NFS 或 Samba，你可能不需要这个。

+   `named`：这个守护进程提供**域名服务**（**DNS**）。只有在系统定义本地名称和 IP 地址时才需要它。你不需要它来解析名称和访问网络。

+   `lpd`：**行打印守护进程**允许其他系统使用此系统的打印机。如果这不是打印服务器，则不需要此服务。

+   `nfsd`：这是 **网络文件系统** 守护进程。它允许远程计算机挂载此计算机的磁盘分区。如果这不是文件服务器，您可能不需要此服务。

+   `portmap`：这是 NFS 支持的一部分。如果系统未使用 NFS，则不需要此服务。

+   `mysql`：**mysql** 应用程序是一个数据库服务器。它可能被您的 Web 服务器使用。

+   `httpd`：这是 HTTP 守护进程。有时它作为 **服务器系统** 软件包的一部分被安装。

根据系统是基于 RedHat 还是 Debian，且是否运行 `systemd`、SysV 或 Upstart，禁用不必要服务的方式有多种。这些命令都必须以 root 权限运行。

# 基于 systemd 的计算机

`systemctl` 命令用于启用和禁用服务。语法如下：

```
    systemctl enable SERVICENAME

```

另外，也可以按如下方式进行：

```
    systemctl disable SERVICENAME

```

要禁用 FTP 服务器，请使用以下命令：

```
    # systemctl disable ftp

```

# 基于 RedHat 的计算机

`chkconfig` 工具提供了一个前端，用于处理 `/etc/rc#.d` 中的 SysV 风格初始化脚本。`-del` 选项用于禁用服务，而 `-add` 选项用于启用服务。请注意，必须已有初始化文件才能添加服务。

语法如下：

```
    # chkconfig -del SERVICENAME
 # chkconfig -add SERVICENAME

```

要禁用 HTTPD 守护进程，请使用以下命令：

```
    # chkconfig -del httpd

```

# 基于 Debian 的计算机

基于 Debian 的系统提供 `update-rc.d` 工具来控制 SysV 风格的初始化脚本。`update-rc.d` 命令支持 `enable` 和 `disable` 作为子命令：

要禁用 telnet 守护进程，请使用以下命令：

```
    # update-rc.d disable telnetd

```

# 还有更多

这些技术将找到已通过 SysV 或 systemd 初始化脚本以 root 用户身份启动的服务。然而，服务可能是手动启动的，或者在启动脚本中，或通过 `xinetd` 启动的。

`xinetd` 守护进程的功能与 init 类似：它启动服务。与 init 不同，`xinetd` 守护进程只有在请求时才启动服务。对于像 SSH 这样的服务，它们不常需要，且一旦启动后运行时间较长，这可以减轻系统负担。像 `httpd` 这样执行小操作（提供网页）且频繁的服务，最有效的方式是启动一次并保持运行。

**xinet** 的配置文件是 `/etc/xinetd.conf`。各个服务文件通常存储在 `/etc/xinetd.d`。

各个服务文件大致如下：

```
# cat /etc/xinetd.d/talk
# description: The talk server accepts talk requests for chatting \
# with users on other systems.
service talk
{
 flags   = IPv4
 disable   = no
 socket_type  = dgram
 wait   = yes
 user   = nobody
 group   = tty
 server   = /usr/sbin/in.talkd
}

```

通过更改 `disable` 字段的值，可以启用或禁用服务。如果 `disable` 为 `no`，则服务已启用。如果 `disable` 为 `yes`，则服务已禁用。

编辑服务文件后，必须重新启动 `xinetd`：

```
 # cd /etc/init.d
 # ./inetd restart

```

# 使用 ss 收集套接字数据

由 `init` 和 `xinetd` 启动的守护进程可能不是系统中唯一运行的服务。守护进程可以通过 `init` 本地文件 `(/etc/rc.d/rc.local)` 中的命令、`crontab` 条目，甚至是有权限的用户启动。

`ss`命令返回套接字统计信息，包括使用套接字的服务和当前的套接字状态。

# 准备工作

`ss`工具包含在`iproute2`包中，该包在大多数现代发行版中已经预装。

# 如何执行...

`ss`命令显示的信息比`netstat`命令更多。这些示例将介绍它的一些功能。

# 显示 TCP 套接字的状态

每次 HTTP 访问、每个 SSH 会话等，都会为每个`tcp`套接字连接打开一个连接。`-t`选项报告 TCP 连接的状态：

```
 $ ss -t
 ESTAB      0      0   192.168.1.44:740          192.168.1.2:nfs 
 ESTAB      0      0   192.168.1.44:35484        192.168.1.4:ssh

 CLOSE-WAIT 0      0   192.168.1.44:47135         23.217.139.9:http 

```

这个例子显示了一个连接到 IP 地址`192.168.1.2`的 NFS 服务器，以及一个到`192.168.1.4`的 SSH 连接。

`CLOSE-WAIT`套接字状态意味着`FIN`信号已发送，但套接字尚未完全关闭。套接字可能永远保持在此状态（或直到你重启）。终止拥有该套接字的进程可能释放该套接字，但不能保证。

# 跟踪监听端口的应用程序

系统上的服务将以`listen`模式打开一个套接字，以接受来自远程站点的网络连接。SSHD 应用程序就是这样做的，用来监听 SSH 连接，HTTP 服务器也是如此，用来接受 HTTP 请求，依此类推。

如果你的系统被黑客攻击，可能会有一个新的应用程序正在监听来自其主人的指令。

`ss`命令的`-l`选项将列出以`listen`模式打开的套接字。`-u`选项指定报告 UDP 套接字。`-t`选项报告 TCP 套接字。

该命令显示了 Linux 工作站上监听的 UDP 套接字的一个子集：

```
$ ss -ul
State          Recv-Q  Send-Q      Local Address:Port         Peer Address:Port 
UNCONN         0       0           *:sunrpc                   *:* 
UNCONN         0       0           *:ipp                      *:* 
UNCONN         0       0           *:ntp                      *:* 
UNCONN         0       0           127.0.0.1:766              *:* 
UNCONN         0       0           *:898                      *:* 

```

该输出显示系统将接受**远程过程调用**（**sunrpc**）。此端口由`portmap`程序使用。`portmap`程序控制对 RPC 服务的访问，并且被`nfs`客户端和服务器使用。

`ipp`和`ntp`端口分别用于**互联网打印协议**和**网络时间协议**。这两个工具都很有用，但在某些系统上可能不需要。

端口`766`和`898`没有列在`/etc/services`中。`lsof`命令的`-I`选项将显示打开端口的任务。你可能需要具有 root 权限才能查看：

```
    # lsof -I :898

```

或者：

```
 # lsof -n -I :898
 COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
 rpcbind 3267  rpc    7u  IPv4  16584      0t0  UDP *:898 
 rpcbind 3267  rpc   10u  IPv6  16589      0t0  UDP *:898 

```

该命令显示，监听端口`898`的任务是 RPC 系统的一部分，而不是黑客。

# 工作原理

`ss`命令使用系统调用从内部内核表中提取信息。系统上已知的服务和端口在`/etc/services`中定义。

# 使用 dstat 收集系统 I/O 使用情况

知道正在运行哪些服务可能并不能告诉你哪些服务正在拖慢系统。`top`命令（在第九章中讨论，*戴上显示器的帽子*）会告诉你 CPU 使用情况以及等待 I/O 的时间，但它可能无法告诉你足够的信息来追踪超负荷的任务。

跟踪 I/O 和上下文切换可以帮助追踪问题的根源。

`dstat`工具可以帮助你找到潜在的瓶颈。

# 准备工作

**dstat** 应用程序通常不会预装。需要通过包管理器安装。它需要 Python 2.2，现代 Linux 系统默认安装此版本：

```
    # apt-get install dstat
 # yum install dstat

```

# 操作方法...

dstat 应用程序定期显示磁盘、网络、内存使用情况和运行任务信息。默认输出提供了系统活动的概览。默认情况下，这份报告每秒会更新一次并显示在新的一行，方便与之前的值进行比较。

默认输出让你能够跟踪整体的系统活动。该应用程序还支持更多选项，以跟踪资源消耗最多的进程。

# 查看系统活动

如果不带任何参数调用 dstat，它会以每秒一次的间隔显示 CPU 活动、磁盘 I/O、网络 I/O、分页、硬件中断和上下文切换。

以下示例显示了默认的 `dstat` 输出：

```
$ dstat
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
 1   2  97   0   0   0|5457B   55k|   0     0 |   0     0 |1702  3177 
 1   2  97   0   0   0|   0     0 |  15k 2580B|   0     0 |2166  4830 
 1   2  96   0   0   0|   0    36k|1970B 1015B|   0     0 |2122  4794 

```

你可以忽略第一行。那些值是 dstat 从表格中提取的初始内容。接下来的行显示了在时间片期间的活动。

在此示例中，CPU 大部分时间处于空闲状态，磁盘活动很少。系统正在生成网络流量，但每秒只有少量的数据包。

这个系统没有进行分页。Linux 只有在主内存耗尽时，才会将内存分页到磁盘。分页可以让系统运行比没有分页时更多的应用程序，但磁盘访问比内存访问慢几千倍，所以如果需要分页，计算机会变得非常缓慢。

如果你的系统出现持续的分页活动，它需要更多的内存或更少的应用程序。

一个数据库应用程序可能会在评估需要构建大规模内存数组的查询时导致间歇性的分页。可以通过将查询改写为使用 IN 操作而不是 JOIN 来减少内存需求。（这是比本书中介绍的 SQL 更高级的用法。）

**上下文切换**（**csw**）发生在每次系统调用时（请参见 第十一章，*追踪线索* 中的 strace 和 ltrace 讨论），以及当时间片到期时，系统将另一个应用程序的访问权限转交给 CPU。每当执行 I/O 操作或程序进行自我调整时，都会发生系统调用。

如果系统每秒执行数万个上下文切换，这是潜在问题的症状。

# 工作原理

`dstat` 工具是一个 Python 脚本，收集并分析来自 `/proc` 文件系统的数据，详见 第十章，*管理调用*。

# 还有更多...

`dstat` 工具可以识别某一类别中资源消耗最多的进程：

+   **-top-bio 磁盘使用情况**：此项报告执行最多块 I/O 的进程。

+   -**top-cpu CPU 使用情况**：这是报告使用最多 CPU 资源的进程。

+   **-top-io I/O 使用情况**：此项报告执行最多 I/O（通常是网络 I/O）的进程。

+   **-top-latency 系统负载**：此项显示具有最高延迟的进程。

+   **-top-mem 内存使用情况**：此选项显示使用最多内存的进程

以下示例显示了每个类别的 CPU 和网络使用情况以及各个类别中的顶级用户：

```
$ dstat -c -top-cpu -n -top-io
----total-cpu-usage---- -most-expensive- -net/total- ----most-expensive----
usr sys idl wai hiq siq|  cpu process   | recv  send|     i/o process 
 1   2  97   0   0   0|vmware-vmx   1.0|   0     0 |bash         26k    2B
 2   1  97   0   0   0|vmware-vmx   1.7|  18k 3346B|xterm       235B 1064B
 2   2  97   0   0   0|vmware-vmx   1.9| 700B 1015B|firefox      82B   32k

```

在运行虚拟机的系统上，虚拟机使用最多的 CPU 时间，但占用的大部分 IO 资源不多。CPU 大部分时间处于空闲状态。

`-c` 和 `-n` 选项分别指定显示 CPU 使用情况和网络使用情况。

# 使用 pidstat 识别资源占用者

`-top-io` 和 `-top-cpu` 标志将标识一个主要的资源使用者，但如果有多个资源占用者实例，可能无法提供足够的信息来识别问题。

`pidstat` 程序将报告每个进程的统计数据，可以按需排序，以提供更多的洞察信息。

# 准备就绪

`pidstat` 应用程序可能不会默认安装。可以通过以下命令进行安装：

```
    # apt-get install sysstat

```

# 如何操作...

pidstat 应用程序有多个选项用于生成不同的报告：

+   `-d`：此选项报告 IO 统计数据

+   `-r`：此选项报告页面错误和内存利用率

+   `-u`：此选项报告 CPU 利用率

+   `-w`：此选项报告任务切换情况

报告上下文切换活动：

```
 $ pidstat -w | head -5
 Linux 2.6.32-642.11.1.el6.x86_64 (rtdaserver.cflynt.com)    
    02/15/2017  _x86_64_ (12 CPU)

 11:18:35 AM       PID   cswch/s nvcswch/s  Command
 11:18:35 AM         1      0.00      0.00  init
 11:18:35 AM         2      0.00      0.00  kthreadd

```

pidstat 应用程序按 PID 编号对报告进行排序。数据可以通过排序工具进行重新组织。以下命令显示每秒产生最多上下文切换的五个应用程序（`-w` 输出中的*字段 4*）：

```
 $ pidstat -w | sort -nr -k 4 | head -5
 11:13:55 AM     13054    351.49      9.12  vmware-vmx
 11:13:55 AM      5763     37.57      1.10  vmware-vmx
 11:13:55 AM      3157     27.79      0.00  kondemand/0
 11:13:55 AM      3167     21.18      0.00  kondemand/10
 11:13:55 AM      3158     21.17      0.00  kondemand/1

```

# 它是如何工作的

pidstat 应用程序查询内核以获取任务信息。排序和头部（head）工具会将数据缩减，找出占用资源最多的程序。

# 使用 sysctl 调整 Linux 内核

Linux 内核大约有 1,000 个可调参数。这些默认值适用于常见用法，意味着它们并不适合所有人。

# 入门

`sysctl` 命令在所有 Linux 系统上均可用。你必须是 root 用户才能修改内核参数。

`sysctl` 命令会立即更改参数值，但除非你在 `/etc/sysctl.conf` 中添加一行定义该参数，否则重启后值会恢复为原始值。

在修改 `sysctl.conf` 之前，最好手动更改值并进行测试。错误的值可能导致系统无法启动，从而无法应用 `/etc/sysctl.conf`。

# 如何操作...

`sysctl` 命令支持多个选项：

+   `-a`：此选项报告所有可用的参数

+   `-p FILENAME`：此选项从 `FILENAME` 读取值。默认从 `/etc/sysctl.conf`

+   `PARAM`：此选项报告 `PARAM` 的当前值

+   `PARAM=NEWVAL`：此选项设置 `PARAM` 的值

# 调整任务调度器

任务调度器针对桌面环境进行了优化，在这种环境中，快速响应用户比整体效率更为重要。增加任务驻留时间有助于提升服务器系统的性能。以下示例检查了 `kernel.sched_migration_cost_ns` 的值：

```
 $ sysctl.kernel.shed_migration_cost_ns
 kernel.sched_migration_cost_ns = 500000

```

`kernel_sched_migration_cost_ns`（在旧版本内核中为`kernel.sched_migration_cost`）控制任务在交换前会保持活跃多久。在任务或线程较多的系统上，这可能会导致过多的上下文切换开销。默认值`500000`纳秒对运行 Postgres 或 Apache 服务器的系统来说太小。建议将值更改为 5 毫秒：

```
    # sysctl kernel.sched_migration_cost_ns=5000000

```

在某些系统（特别是 Postgres 服务器）上，取消设置`sched_autogroup_enabled`参数可以提高性能。

# 调整网络

对于执行大量网络操作的系统（如 NFS 客户端、NFS 服务器等），网络缓冲区的默认值可能太小。

检查最大读取缓冲区内存的值：

```
 $ sysctl net.core.rmem_max
 net.core.rmem_max = 124928

```

增加网络服务器的值：

```
 # sysctl net.core.rmem_max=16777216
 # sysctl net.core.wmem_max=16777216
 # sysctl net.ipv4.tcp_rmem="4096 87380 16777216"
 # sysctl net.ipv4.tcp_wmem="4096 65536 16777216"
 # sysctl net.ipv4.tcp_max_syn_backlog=4096

```

# 它是如何工作的

`sysctl`命令让你可以直接访问内核参数。默认情况下，大多数发行版会为普通工作站优化这些参数。

如果你的系统内存较大，你可以通过增加分配给缓冲区的内存量来提高性能。如果内存较少，你可能需要缩减这些缓冲区的大小。如果系统是服务器，你可能希望让任务在内存中驻留更长时间，而不是单用户工作站那样频繁交换。

# 还有更多...

`/proc`文件系统在所有 Linux 发行版中都可用。它包括一个文件夹用于每个正在运行的任务，还包括所有主要内核子系统的文件夹。这些文件夹中的文件可以通过`cat`命令查看和更新。

sysctl 支持的参数通常也被`/proc`文件系统支持。

因此，`net.core.rmem_max`也可以通过`/proc/sys/net/core/rmem_max`来访问。

# 使用配置文件调整 Linux 系统

Linux 系统包含多个文件来定义如何挂载磁盘等。可以在这些文件中设置一些参数，而不是使用`/proc`或`sysctl`。

# 准备工作

在`/etc`目录下有多个文件控制着系统的配置。这些文件可以使用标准文本编辑器（如`vi`或`emacs`）进行编辑。更改可能需要重启系统才能生效。

# 如何操作...

`/etc/fstab`文件定义了磁盘的挂载方式以及支持的选项。

Linux 系统记录了文件的创建、修改和读取时间。知道文件是否已被读取并没有太大意义，而每次常见的工具（如`cat`）被访问时更新`Accessed`时间戳会导致额外开销。

`noatime`和`relatime`挂载选项将减少磁盘的频繁访问：

```
 $ cat /dev/fstab
 /dev/mapper/vg_example_root  /    ext4 defaults,noatime  1 1
 /dev/mapper/gb_example_spool /var ext4 defaults,relatime 1 1

```

# 它是如何工作的

上面的示例挂载了`/`分区（其中包含`/bin`和`/usr/bin`），使用了常见的默认选项，并增加了`noatime`参数，以禁用每次访问文件时都更新磁盘。`/var`分区（包含邮件队列文件夹）设置了 realtime 选项，这将确保至少每天更新一次时间，而不是每次访问文件时都更新。

# 使用 nice 命令更改调度器优先级

每个 Linux 系统上的任务都有一个优先级。优先级的值范围从 -20 到 19。优先级值越低（**-20**），任务分配到的 CPU 时间就越多。默认优先级是 **0**。

并非所有任务都需要相同的优先级。交互式应用程序需要快速响应，否则会变得难以使用。通过 `crontab` 运行的后台任务只需要在计划再次运行之前完成。

`nice` 命令将修改任务的优先级。可以用它来调用一个修改过优先级的任务。提高任务的优先级值会为其他任务释放资源。

# 如何操作...

如果没有参数调用 `nice` 命令，将报告任务的当前优先级：

```
    $ cat nicetest.sh
 echo "my nice is `nice`"
 $ sh nicetest.sh
 my nice is 0

```

调用 `nice` 命令并跟随另一个命令名称，将以 `10` 的 *niceness* 执行第二个命令——它会将 10 加到任务的默认优先级上：

```
    $ nice sh nicetest.sh
 my nice is 10

```

在命令前调用 `nice` 命令并设置一个值，将以定义的 *niceness* 执行该命令：

```
    $ nice -15 sh nicetest.sh
 my nice is 15

```

只有超级用户才能通过分配负的 niceness 值为任务设置更高的优先级（更低的优先级数字）：

```
    # nice -adjustment=-15 nicetest.sh
 my nice is -15

```

# 工作原理

`nice` 命令修改内核的调度表，以更高或更低的优先级运行任务。优先级值越低，调度器分配给此任务的时间就越多。

# 还有更多

`renice` 命令修改正在运行的任务的优先级。那些消耗大量资源但对时间要求不高的任务，可以使用这个命令将其变得*更加友好*。`top` 命令有助于找到最占用 CPU 的任务。

`renice` 命令通过新优先级值和程序 ID（PID）来调用：

```
    $ renice 10 12345
 12345: old priority 0, new priority 10

```
