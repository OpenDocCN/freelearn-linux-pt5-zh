# 管理调用

本章将涵盖以下主题：

+   收集有关进程的信息

+   各种命令的功能 - which、whereis、whatis 和 file

+   杀死进程，发送和响应信号

+   向用户终端发送消息

+   `/proc` 文件系统

+   收集系统信息

+   使用 `cron` 调度任务

+   数据库风格与用途

+   编写和读取 SQLite 数据库

+   从 Bash 编写和读取 MySQL 数据库

+   用户管理脚本

+   批量调整图像大小和格式转换

+   从终端截屏

+   从一个管理多个终端

# 介绍

从一个 GNU/Linux 系统管理多个终端：GNU/Linux 生态系统由网络、硬件、操作系统内核（负责资源分配）、接口模块、系统工具和用户程序组成。管理员需要监控整个系统，以确保一切顺利运行。Linux 管理工具从集成的图形用户界面应用程序到专为脚本设计的命令行工具不等。

# 收集有关进程的信息

在这种情况下，**进程**一词指的是程序的运行实例。计算机上同时运行着多个进程。每个进程都会分配一个唯一的标识号，称为**进程 ID**（**PID**）。同名的多个程序实例可以同时运行，但它们会拥有不同的 PID 和属性。进程属性包括拥有进程的用户、程序使用的内存量、程序使用的 CPU 时间等。此教程展示了如何收集有关进程的信息。

# 准备就绪

与进程管理相关的重要命令有 `top`、`ps` 和 `pgrep`。这些工具在所有 Linux 发行版中都有提供。

# 如何执行...

`ps` 命令报告活动进程的信息。它提供有关哪个用户拥有进程、进程何时启动、执行进程的命令路径、PID、附加的终端（**TTY**，即**电传打字机**）、进程使用的内存、进程使用的 CPU 时间等信息。考虑以下示例：

```
$ ps
PID TTY       TIME CMD
1220 pts/0    00:00:00 bash
1242 pts/0    00:00:00 ps

```

默认情况下，`ps` 只会显示当前终端（TTY）启动的进程。第一列显示 PID，第二列表示终端（TTY），第三列表示自进程启动以来经过的时间，最后一列是 CMD（命令）。

`ps` 命令报告可以通过命令行参数进行修改。

`-f (full)` 选项显示更多列的信息：

```
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
slynux    1220  1219  0 18:18 pts/0    00:00:00 -bash
slynux    1587  1220  0 18:59 pts/0    00:00:00 ps -f

```

`-e`（所有）和 `-ax`（全部）选项提供了有关系统上所有正在运行进程的报告。

`-x` 参数（与 `-a` 一起使用）表示去除 `ps` 默认的 TTY 限制。通常，如果没有使用任何参数，`ps` 只会打印与当前终端连接的进程。

命令 `ps -e`、`ps -ef`、`ps -ax` 和 `ps -axf` 生成有关所有进程的报告，并提供比 `ps` 更多的信息：

```
$ ps -e | head -5
PID TTY    TIME CMD
1 ?        00:00:00 init
2 ?        00:00:00 kthreadd
3 ?        00:00:00 migration/0
4 ?        00:00:00 ksoftirqd/0

```

`-e` 选项生成一个长报告。这个示例使用 `head` 过滤输出，只显示前五个条目。

`-o PARAMETER1`, `PARAMETER2` 选项指定要显示的数据。

`-o` 参数之间用逗号（`,`）分隔。逗号运算符与下一个参数之间没有空格。

`-o` 选项可以与 `-e`（每个）选项（`-eo`）结合使用，以列出系统中运行的每个进程。但是，当你使用类似于限制 `ps` 到指定用户的过滤器与 `-o` 一起使用时，`-e` 将不会被使用。`-e` 选项会覆盖过滤器并显示所有进程。

在这个例子中，`comm` 代表 COMMAND，`pcpu` 代表 CPU 使用百分比：

```
$ ps -eo comm,pcpu | head -5
COMMAND          %CPU
init             0.0
kthreadd         0.0
migration/0      0.0
ksoftirqd/0      0.0

```

# 它是如何工作的...

支持以下 `-o` 选项的参数：

| **参数** | **描述** |
| --- | --- |
| `pcpu` | CPU 使用百分比 |
| `pid` | 进程 ID |
| `ppid` | 父进程 ID |
| `pmem` | 内存使用百分比 |
| `comm` | 可执行文件名 |
| `cmd` | 简单命令 |
| `user` | 启动进程的用户 |
| `nice` | 优先级（亲和度） |
| `time` | 累积 CPU 时间 |
| `etime` | 进程启动以来的经过时间 |
| `tty` | 关联的 TTY 设备 |
| `euid` | 有效用户 |
| `stat` | 进程状态 |

# 还有更多内容...

`ps` 命令、`grep` 和其他工具可以结合使用，以生成自定义报告。

# 显示进程的环境变量

一些进程依赖于它们的环境变量定义。了解环境变量及其值可以帮助你调试或定制进程。

`ps` 命令通常不会显示命令的环境信息。命令末尾的 `e` 输出修饰符会将这些信息添加到输出中：

```
$ ps e

```

这是一个环境信息的示例：

```
$ ps -eo pid,cmd  e | tail -n 1
1238 -bash USER=slynux LOGNAME=slynux HOME=/home/slynux PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAIL=/var/mail/slynux SHELL=/bin/bash SSH_CLIENT=10.211.55.2 49277 22 SSH_CONNECTION=10.211.55.2 49277 10.211.55.4 22 SSH_TTY=/dev/pts/0 

```

环境信息有助于通过 `apt-get` 包管理器追踪问题。如果你使用 HTTP 代理连接到互联网，可能需要通过 `http_proxy=host:port` 设置环境变量。如果未设置此变量，`apt-get` 命令将不会选择代理，从而返回错误。知道没有设置 `http_proxy` 会使问题变得显而易见。

当使用调度工具（如本章后面讨论的 `cron`）来运行应用程序时，预期的环境变量可能没有设置。这个 `crontab` 条目不会打开 GUI 窗口应用程序：

```
00 10 * * * /usr/bin/windowapp

```

它失败是因为 GUI 应用程序需要 `DISPLAY` 环境变量。要确定所需的环境变量，请手动运行 `windowapp`，然后运行 `ps -C windowapp -eo cmd e`。

确定所需的环境变量后，在 `crontab` 中的命令名称之前定义它们：

```
00 10 * * * DISPLAY=:0 /usr/bin/windowapp

```

或者

```
DISPLAY=0
00 10 * * * /usr/bin/windowapp

```

定义 `DISPLAY=:0` 是通过 `ps` 输出获取的。

# 创建进程树视图

`ps`命令可以报告进程 PID，但从子进程追踪到最终父进程很麻烦。将`f`添加到`ps`命令的末尾会创建一个进程树视图，显示任务之间的父子关系。下一个示例显示了从 bash shell 中运行的`ssh`会话，该会话是在`xterm`内启动的：

```
$ ps -u clif f | grep -A2 xterm | head -3
15281  ?      S     0:00 xterm 
15284 pts/20  Ss+   0:00 \_ bash
15286 pts/20  S+    0:18 \_ ssh 192.168.1.2

```

# 对 ps 输出进行排序

默认情况下，`ps`命令的输出是未排序的。`-sort`参数强制`ps`对输出进行排序。可以通过在参数前添加`+`（升序）或`-`（降序）前缀来指定升序或降序：

```
$ ps [OPTIONS] --sort -paramter1,+parameter2,parameter3..

```

例如，要列出前五个 CPU 占用率最高的进程，请使用以下命令：

```
$ ps -eo comm,pcpu --sort -pcpu | head -5
COMMAND         %CPU
Xorg             0.1
hald-addon-stor  0.0
ata/0            0.0
scsi_eh_0        0.0

```

这会显示前五个进程，按 CPU 使用百分比降序排序。

`grep`命令可以过滤`ps`输出。要仅报告当前正在运行的 Bash 进程，请使用以下命令：

```
$ ps -eo comm,pid,pcpu,pmem | grep bash
bash             1255  0.0  0.3
bash             1680  5.5  0.3

```

# 使用 ps 过滤真实用户或 ID、有效用户或 ID

`ps`命令可以根据指定的实际和有效用户名或 ID 对进程进行分组。`ps`命令通过检查每个条目是否属于有效用户或真实用户，从参数列表中过滤输出。

+   使用`-u EUSER1`、`EUSER2`等指定有效用户列表

+   使用`-U RUSER1`、`RUSER2`等指定真实用户列表

这是一个示例：

```
 # display user and percent cpu usage for processes with real user
 # and effective user of root
 $ ps -u root -U root -o user,pcpu

```

`-o`选项可以与`-e`一起使用，如`-eo`，但当应用过滤器时，`-e`不应使用，它会覆盖过滤器选项。

# ps 的 TTY 过滤器

`ps`输出可以通过指定进程所在的 TTY 来选择。使用`-t`选项指定 TTY 列表：

```
 $ ps -t TTY1, TTY2 ..

```

这是一个示例：

```
 $ ps -t pts/0,pts/1
 PID TTY          TIME CMD
 1238 pts/0    00:00:00 bash
 1835 pts/1    00:00:00 bash
 1864 pts/0    00:00:00 ps

```

# 进程线程信息

`ps`的`-L`选项将显示进程线程的信息。此选项会在线程 ID 旁边添加 LWP 列。将`-f`选项与`-L`（`-Lf`）一起使用会添加两列：NLWP（线程数）和 LWP（线程 ID）：

```
 $ ps -Lf
 UID  PID  PPID  LWP  C  NLWP  STIME  TTY  TIME     
        CMD
 user 1611 1     1612 0  2     Jan16  ?    00:00:00    
        /usr/lib/gvfs/gvfsd

```

该命令列出了五个进程及其最大线程数：

```
$ ps -eLf --sort -nlwp | head -5
UID        PID  PPID   LWP  C NLWP STIME TTY  TIME     
     CMD
root       647     1   647  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon
root       647     1   654  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon
root       647     1   656  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon
root       647     1   657  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon

```

# 指定输出宽度和要显示的列

`ps`命令支持多种选项来选择要显示的字段并控制其显示方式。以下是一些常见选项：

| `-f` | 这指定了一个完整的格式，包含父 PID 的启动时间、用户 ID 等。 |
| --- | --- |
| `-u` userList | 选择由列表中的用户拥有的进程，默认选择当前用户。 |
| `-l` | 长格式列出，显示用户 ID、父 PID、大小等信息。 |

# 什么是什么 – which、whereis、whatis 和 file

可能有多个文件具有相同的名称。知道正在调用哪个可执行文件以及文件是编译代码还是脚本是有用的信息。

# 如何操作...

`which`、`whereis`、`file`和`whatis`命令报告文件和目录的信息。

+   `which`：which 命令报告命令的位置：

```
        $ which ls
 /bin/ls

```

+   我们经常在不知道可执行文件存储目录的情况下使用命令。根据 `PATH` 变量的定义方式，你可能会使用来自 `/bin`、`/usr/local/bin` 或 `/opt/PACKAGENAME/bin` 的命令。

+   当我们输入命令时，终端会在一组目录中查找该命令，并执行它找到的第一个可执行文件。要搜索的目录由 `PATH` 环境变量指定：

```
 $ echo $PATH /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

```

+   我们可以添加要搜索的目录并导出新的 `PATH`。要将 `/opt/bin` 添加到 `PATH`，请使用以下命令：

```
        $ export PATH=$PATH:/opt/bin
 # /opt/bin is added to PATH

```

+   **whereis**：`whereis` 类似于 which 命令。它不仅返回命令的路径，还打印命令的 man 页位置（如果有的话）以及命令源代码的路径（如果有的话）：

```
        $ whereis ls
 ls: /bin/ls /usr/share/man/man1/ls.1.gz

```

+   **whatis**：`whatis` 命令输出作为参数传递的命令的单行描述。它从 `man` 页解析信息：

```
        $ whatis ls
 ls (1)    - list directory contents

```

`file` 命令报告文件类型。其语法如下：

```
        $ file FILENAME

```

+   报告的文件类型可能包含几个词或一个长描述：

```
        $file /etc/passwd
 /etc/passwd: ASCII text
 $ file /bin/ls
 /bin/ls: ELF 32-bit LSB executable, Intel 80386, version 1   
        (SYSV), dynamically linked (uses shared libs), for GNU/Linux    
        2.6.15, stripped

```

apropos

有时候我们需要搜索与主题相关的命令。`apropos` 命令会在 man 页中搜索关键字。以下是执行此操作的代码：**Apropos 主题**

# 从给定的命令名中查找进程 ID

假设正在执行多个命令实例。在这种情况下，我们需要每个进程的 PID。`ps` 和 `pgrep` 命令都会返回此信息：

```
 $ ps -C COMMAND_NAME

```

或者，以下内容将被返回：

```
 $ ps -C COMMAND_NAME -o pid=

```

当 `=` 被附加到 `pid` 时，它会从 `ps` 的输出中移除 PID 标头。要从某一列中移除标头，只需在参数后附加 `=`。

该命令列出 Bash 进程的进程 ID：

```
 $ ps -C bash -o pid=
 1255
 1680

```

`pgrep` 命令也会返回一个命令的进程 ID 列表：

```
 $ pgrep bash
 1255
 1680

```

`pgrep` 只需要命令名的一部分作为输入参数来提取 Bash 命令；例如，`pgrep ash` 或 `pgrep bas` 也能工作。但 `ps` 命令要求你输入准确的命令。`pgrep` 支持这些输出过滤选项。

`-d` 选项指定一个与默认换行符不同的输出分隔符：

```
 $ pgrep COMMAND -d DELIMITER_STRING
 $ pgrep bash -d ":"
 1255:1680

```

`-u` 选项过滤出用户列表：

```
 $ pgrep -u root,slynux COMMAND

```

在这个命令中，`root` 和 `slynux` 是用户。

`-c` 选项返回匹配进程的数量：

```
 $ pgrep -c COMMAND

```

# 确定系统的繁忙程度

系统要么空闲，要么超载。`load average` 值描述了运行系统的总负载。它描述了系统上可运行进程的平均数量，这些进程拥有除 CPU 时间片外的所有资源。

系统的负载平均值由 uptime 和 top 命令报告。它会报告三个值。第一个值表示 1 分钟的平均值，第二个表示 5 分钟的平均值，第三个表示 15 分钟的平均值。

它由 uptime 报告：

```
 $ uptime
 12:40:53 up  6:16,  2 users,  load average: 0.00, 0.00, 0.00

```

# top 命令

默认情况下，`top` 命令显示消耗最多 CPU 的进程列表以及基本的系统统计信息，包括进程列表中的任务数、CPU 核心数和内存使用情况。输出会每隔几秒钟更新一次。

此命令显示多个参数以及消耗最多 CPU 的进程：

```
    $ top
 top - 18:37:50 up 16 days, 4:41,7 users,load average 0.08 0.05 .11
 Tasks: 395 total,  2 running, 393 sleeping, 0 stopped 0 zombie

```

# 另请参见...

+   本章中的 *使用 cron 调度任务* 说明了如何调度任务

# 杀死进程，并发送和响应信号

如果你需要降低系统负载，或者在重启之前，可能需要杀死一些进程（如果它们失控并开始消耗过多资源）。信号是一种进程间通信机制，可以中断正在运行的进程，并迫使其执行某些操作。这些操作包括强制进程以受控或立即的方式终止。

# 准备就绪

信号向运行中的程序发送中断。当进程收到信号时，它会通过执行信号处理程序来响应。编译后的应用程序通过 `kill` 系统调用生成信号。信号可以通过命令行（或 shell 脚本）中的 `kill` 命令生成。`trap` 命令可用于脚本中处理接收到的信号。

每个信号都有一个名称和一个整数值。`SIGKILL (9)` 信号会立即终止进程。按键事件 *Ctrl* + *C* 和 *Ctrl* + *Z* 会发送信号中止任务或将任务放入后台。

# 如何操作...

1.  `kill -l` 命令将列出可用的信号：

```
        $ kill -l
 SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP
 ...

```

1.  终止进程：

```
        $ kill PROCESS_ID_LIST

```

`kill` 命令默认会发送 `SIGTERM` 信号。进程 ID 列表使用空格作为分隔符。

1.  `-s` 选项指定要发送给进程的信号：

```
        $ kill -s SIGNAL PID

```

`SIGNAL` 参数可以是信号名称或信号编号。不同用途的信号有很多种，最常见的如下：

+   `SIGHUP 1`：控制进程或终端死亡时的挂起检测

+   `SIGINT 2`：这是在按下 *Ctrl* + *C* 时发出的信号

+   `SIGKILL 9`：这是用于强制终止进程的信号

+   `SIGTERM 15`：这是默认用于终止进程的信号

+   `SIGTSTP 20`：这是在按下*Ctrl* + *Z* 时发出的信号

1.  我们经常使用强制杀死进程的方式。使用时需谨慎。这是一个立即的操作，它不会保存数据或执行正常的清理操作。应首先尝试使用 `SIGTERM` 信号；`SIGKILL` 应仅在极端情况下使用：

```
        $ kill -s SIGKILL PROCESS_ID

```

或者，使用此方法执行清理操作：

```
        $ kill -9 PROCESS_ID

```

# 还有更多...

Linux 支持其他命令来发送信号或终止进程。

# kill 系列命令

`kill` 命令将进程 ID 作为参数。`killall` 命令通过进程名终止进程：

```
    $ killall process_name

```

`-s` 选项指定要发送的信号。默认情况下，`killall` 发送 `SIGTERM` 信号：

```
    $ killall -s SIGNAL process_name

```

`-9` 选项通过进程名强制杀死进程：

```
    $ killall -9 process_name

```

以下是前面的示例：

```
    $ killall -9 gedit

```

`-u` 所有者指定进程的用户：

```
    $ killall -u USERNAME process_name

```

`-I` 选项使 `killall run` 以交互模式运行：

`pkill` 命令类似于 `kill` 命令，但默认情况下它接受的是进程名而不是进程 ID：

```
    $ pkill process_name
 $ pkill -s SIGNAL process_name

```

`SIGNAL` 是信号号。`SIGNAL` 名称不支持 `pkill`。`pkill` 命令提供了与 `kill` 命令相同的许多选项。有关更多细节，请查看 `pkill` 的手册页。

# 捕获并响应信号

良好的程序在接收到 `SIGTERM` 信号时会保存数据并正常关闭。`trap` 命令为脚本中的信号分配信号处理程序。一旦通过 `trap` 命令将函数分配给信号，当脚本接收到该信号时，这个函数会被执行。

语法如下：

```
    trap 'signal_handler_function_name' SIGNAL LIST

```

`SIGNAL LIST` 是空格分隔的。它可以包含信号号和信号名称。

这个 shell 脚本响应 `SIGINT` 信号：

```
#/bin/bash 
#Filename: sighandle.sh 
#Description: Signal handler  

function handler() 
{ 
  echo Hey, received signal : SIGINT 
} 

# $$ is a special variable that returns process ID of current  
# process/script 

echo My process ID is $$ 

#handler is the name of the signal handler function for SIGINT signal 
trap 'handler' SIGINT 

while true; 
do 
  sleep 1 
done 

```

在终端中运行这个脚本。当脚本正在运行时，按下 *Ctrl* + *C*，它会通过执行与之关联的信号处理程序来显示消息。*Ctrl* + *C* 对应于 `SIGINT` 信号。

`while` 循环用于保持进程永远运行而不被终止。这样做是为了让脚本能够响应信号。保持进程无限存活的循环通常被称为 **事件循环**。

如果给定了脚本的进程 ID，`kill` 命令可以向其发送信号：

```
    $ kill -s SIGINT PROCESS_ID

```

执行时，会打印出前述脚本的进程 ID；或者，你可以使用 `ps` 命令找到它。

如果没有为信号指定信号处理程序，脚本将调用操作系统分配的默认信号处理程序。通常，按 *Ctrl* + *C* 会终止程序，因为操作系统提供的默认处理程序会终止该进程。这里定义的自定义处理程序会覆盖默认处理程序。

我们可以使用 `trap` 命令为任何可用的信号（`kill -l`）定义信号处理程序。一个信号处理程序可以处理多个信号。

# 向用户终端发送消息

Linux 支持三种应用程序向其他用户的屏幕显示消息。`write` 命令向用户发送消息，`talk` 命令让两个用户进行对话，`wall` 命令向所有用户发送消息。

在执行可能导致系统中断的操作之前（例如，重启服务器），系统管理员应该向系统或网络中每个用户的终端发送一条消息。

# 准备工作

`write` 和 `wall` 命令是大多数 Linux 发行版的一部分。如果一个用户登录了多个会话，你可能需要指定要发送消息的终端。

你可以使用 `who` 命令来确定用户的终端：

```
    $> who
 user1    pts/0    2017-01-16 13:56 (:0.0)
 user1    pts/1    2017-01-17 08:35 (:0.0)

```

第二列（`pts/#`）是用户的终端标识符。

`write` 和 `wall` 程序仅在单一系统上工作。`talk` 程序可以在网络中连接多个用户。

`talk` 程序并不常安装。在使用 `talk` 的任何机器上，`talk` 程序和 `talk` 服务器都必须安装并运行。在基于 Debian 的系统上安装 `talk` 应用程序为 `talk` 和 `talkd`，在基于 Red Hat 的系统上为 `talk` 和 `talk-server`。你可能需要编辑 `/etc/xinet.d/talk` 和 `/etc/xinet.d/ntalk` 来将 `disable` 字段设置为 `no`。完成后，重新启动 `xinet`：

```
    # cd /etc/xinet.d
 # vi ntalk
 # cd /etc/init.d
 #./xinetd restart

```

# 如何操作...

# 向单个用户发送消息

`write` 命令将向单个用户发送消息：

```
    $ write USERNAME [device]

```

你可以从文件、回显或者交互式方式重定向消息。交互式写入通过 Ctrl-D 结束。

通过将伪终端标识符附加到命令中，可以将消息定向到特定会话：

```
    $ echo "Log off now. I'm rebooting the system" | write user1 pts/3

```

# 与其他用户进行对话

`talk` 命令会在两个用户之间开启一个交互式对话。其语法为 `$ talk user@host`。

下一个命令将在用户 2 的工作站上发起与其的对话：

```
    $ talk user2@workstation2.example.com

```

输入 `talk` 命令后，你的终端会被清空，并分成两个窗口。在其中一个窗口，你将看到如下文本：

```
    [Waiting for your party to respond]

```

你尝试与之交谈的人将看到如下消息：

```
    Message from Talk_Daemon@workstation1.example.com
 talk: connection requested by user1@workstation.example.com
 talk: respond with talk user1@workstation1.example.com

```

当他们调用 `talk` 时，他们的终端会话也会被清空并分裂。你在一个窗口输入的内容将在他们的屏幕上显示，而他们输入的内容则会出现在你的屏幕上：

```
    I need to reboot the database server.
 How much longer will your processing take?
 ---------------------------------------------
 90% complete. Should be just a couple more minutes.

```

# 向所有用户发送消息

**wall**（WriteALL）命令会广播一条消息到所有用户和终端会话：

```
    $ cat message | wall

```

或：

```
    $ wall < message
 Broadcast Message from slynux@slynux-laptop
 (/dev/pts/1) at 12:54 ...

 This is a message

```

消息头显示了消息的发送者：包括用户和主机。

`write`、`talk` 和 `wall` 命令仅在启用写消息选项时才能在用户之间传递消息。来自 root 的消息无论写消息选项如何都将显示。

消息选项通常是启用的。`mesg` 命令将启用或禁用接收消息：

```
    # enable receiving messages
 $ mesg y
 # disable receiving messages
 $ mesg n

```

# `/proc` 文件系统

`/proc` 是一个内存中的伪文件系统，提供对许多 Linux 内核内部数据结构的用户空间访问。大多数伪文件是只读的，但有些文件，如 `/proc/sys/net/ipv4/forward`（在第八章，*老男孩网络*中有描述），可以用来微调系统行为。

# 如何操作...

`/proc` 目录包含多个文件和目录。你可以使用 `cat`、`less` 或 `more` 查看 `/proc` 及其子目录中的大部分文件。它们以纯文本形式显示。

系统上运行的每个进程都有一个 `/proc` 目录，目录名称为该进程的 PID。

假设 Bash 正在运行，PID 为 `4295`（通过 `pgrep bash` 查找）；在这种情况下，`/proc/4295` 将存在。这个文件夹将包含关于该进程的信息。`/proc/PID` 下的文件包括：

+   `environ`：它包含与进程相关的环境变量。`cat /proc/4295/environ` 将显示传递给进程 `4295` 的环境变量。

+   `cwd`：这是指向进程工作目录的 `symlink`。

+   `exe`：这是指向进程可执行文件的 `symlink`：

```
        $ readlink /proc/4295/exe
 /bin/bash

```

+   `fd`：这是一个包含进程使用的文件描述符条目的目录。值 0、1 和 2 分别表示 stdin、stdout 和 stderr。

+   `io`：此文件显示进程读取或写入的字符数。

# 收集系统信息

描述计算机系统需要多组数据。这些数据包括网络信息、主机名、内核版本、Linux 发行版名称、CPU 描述、内存分配、磁盘分区等。这些信息可以通过命令行获取。

# 如何操作...

1.  `hostname` 和 `uname` 命令打印当前系统的主机名：

```
        $ hostname

```

或者，它们会打印以下内容：

```
        $ uname -n
 server.example.com

```

1.  `uname` 的 `-a` 选项打印有关 Linux 内核版本、硬件架构等的详细信息：

```
        $ uname -a
 server.example.com 2.6.32-642.11.1.e16.x86_64 #1 SMP Fri Nov 18   
        19:25:05 UTC 2016 x86_64 x86_64 GNU/Linux

```

1.  `-r` 选项将报告限制为内核版本：

```
        $ uname -r
 2.6.32-642.11.1.e16.x86_64

```

1.  `-m` 选项打印机器类型：

```
        $ uname -m
 x86_64

```

1.  `/proc/` 目录包含关于系统、模块和正在运行的进程的信息。`/proc/cpuinfo` 包含 CPU 详细信息：

```
        $ cat /proc/cpuinfo
 processor     : 0
 vendor_id     : GenuineIntel
 cpu family    : 6
 model         : 63
 model name    : Intel(R)Core(TM)i7-5820K CPU @ 3.30GHz
 ...

```

如果处理器有多个核心，这些行会重复 n 次。要提取单项信息，请使用`sed`。第五行包含处理器名称：

```
        $ cat /proc/cpuinfo | sed -n 5p
 Intel(R)CORE(TM)i7-5820K CPU @ 3.3 GHz

```

1.  `/proc/meminfo` 包含有关内存和当前 RAM 使用情况的信息：

```
        $ cat /proc/meminfo
 MemTotal:     32777552 kB
 MemFree:      11895296 kB 
 Buffers:        634628 kB
 ...

```

`meminfo` 的第一行显示系统的总 RAM：

```
        $ cat /proc/meminfo  | head -1
 MemTotal:        1026096 kB

```

1.  `/proc/partitions` 描述磁盘分区：

```
        $ cat /proc/partitions
 major minor  #blocks  name
 8        0 976762584 sda
 8        1    512000 sda1
 8        2 976248832 sda2
 ...

```

`fdisk` 程序编辑磁盘的分区表并报告当前分区表。以 `root` 身份运行此命令：

```
        $ sudo fdisk -l

```

1.  `lshw` 和 `dmidecode` 应用程序生成关于系统的长时间且完整的报告。报告包括主板、BIOS、CPU、内存插槽、接口插槽、磁盘等信息。这些命令必须以 root 身份运行。`dmidecode` 通常是可用的，但你可能需要安装 `lshw`：

```
        $ sudo lshw
 description: Computer
 product: 440BX
 vendor: Intel
 ...

 $ sudo dmidecode
 SMBIOS 2.8 present
 115 structures occupying 4160 bytes.
 Table at 0xDCEE1000.

 BIOS Information
 Vendor: American Megatrends Inc
 ...

```

# 使用 cron 调度

GNU/Linux 系统支持几种调度任务的工具。`cron` 工具是最广泛支持的。它允许你安排任务在后台以固定间隔执行。`cron` 工具使用一个表（crontab），列出要执行的脚本或命令及其执行时间。

Cron 用于安排系统日常任务，例如执行备份、将系统时钟与 `ntpdate` 同步、删除临时文件等。

普通用户可能会使用 `cron` 安排互联网下载任务，在晚上网络提供商允许的时间进行下载，并且带宽较高。

# 准备就绪

`cron` 调度工具随所有 GNU/Linux 发行版提供。它扫描 `cron` 表以确定是否有命令应当执行。每个用户都有自己的 `cron` 表，这是一个纯文本文件。`crontab` 命令用于操作 `cron` 表。

# 如何操作...

`crontab` 条目指定执行命令的时间和要执行的命令。`cron` 表中的每一行定义一个单独的命令。该命令可以是脚本或二进制应用程序。当 `cron` 运行任务时，它以创建条目的用户身份运行，但不会加载该用户的 `.bashrc` 文件。如果任务需要环境变量，则必须在 `crontab` 中定义它们。

每行 cron 表由六个以空格分隔的字段组成，顺序如下：

+   `分钟` (0 - 59)

+   `小时` (0 - 23)

+   `天` (1 - 31)

+   `月份` (1 - 12)

+   `星期几` (0 - 6)

+   `命令`（在指定时间执行的脚本或命令）

前五个字段指定命令执行的时间。多个值之间用逗号分隔（没有空格）。星号表示任何时间或任何一天都能匹配。除号表示按每 /Y 间隔触发事件（例如，`*/5` 表示每五分钟触发一次）。

1.  在每小时的第 2 分钟执行 `test.sh` 脚本：

```
        02 * * * * /home/slynux/test.sh

```

1.  在每天的第 5、6、7 小时执行 **test.sh**：

```
        00 5,6,7 * * /home/slynux/test.sh

```

1.  每隔一小时在星期天执行 `script.sh`：

```
        00 */2 * * 0 /home/slynux/script.sh

```

1.  每天凌晨 2 点关机：

```
        00 02 * * * /sbin/shutdown -h

```

1.  `crontab` 命令既可以交互式使用，也可以与预写的文件一起使用。

使用 `-e` 选项与 `crontab` 一起编辑 `cron` 表：

```
        $ crontab -e
 02 02 * * * /home/slynux/script.sh

```

输入 `crontab -e` 时，默认文本编辑器（通常是 `vi`）将被打开，用户可以输入 `cron` 作业并保存。`cron` 作业将在指定的时间间隔内进行调度并执行。

1.  `crontab` 命令可以从脚本中调用，以用新的内容替换当前的 crontab。具体操作如下：

    +   创建一个文本文件（例如，`task.cron`），其中包含 `cron` 任务，然后运行 `crontab` 并将该文件名作为命令参数：

```
                $ crontab task.cron

```

+   或者，将 `cron` 任务作为内联函数指定，而不创建单独的文件。例如，参考以下内容：

```
                $ crontab<<EOF
 02 * * * * /home/slynux/script.sh
 EOF

```

`cron` 任务需要写在 `crontab<<EOF` 和 `EOF` 之间。

# 它是如何工作的...

星号（`*`）表示在给定时间段内每次都执行该命令。`cron` 任务中的 `小时` 字段如果是 `*`，则表示每小时都执行该命令。要在时间段的多个实例中执行命令，请在该时间字段中指定用逗号分隔的时间间隔。例如，要在第 5 和第 10 分钟执行命令，可以在 `分钟` 字段中输入 `5,10`。斜杠（除号）符号表示按时间的分割执行命令。例如，在 `分钟` 字段中输入 `0-30/6` 将会在每小时的前半小时，每隔 5 分钟执行一次命令。`小时` 字段中的字符串 `*/12` 将表示每隔一小时执行一次命令。

Cron 作业以创建 `crontab` 的用户身份执行。如果需要执行需要更高权限的命令，例如关闭计算机，请以 root 用户身份运行 `crontab` 命令。

在 cron 作业中指定的命令需要写出命令的完整路径。这是因为 cron 不会加载你的`.bashrc`，因此 cron 作业执行时的环境与我们在终端中执行 bash 时的环境不同。因此，`PATH`环境变量可能没有设置。如果你的命令需要特定的环境变量，必须显式地设置它们。

# 还有更多...

`crontab`命令有更多选项。

# 指定环境变量

许多命令要求环境变量正确设置才能执行。cron 命令将 SHELL 变量设置为`"/bin/sh"`，并且还会从`/etc/passwd`中的值设置`LOGNAME`和`HOME`。如果需要其他变量，可以在`crontab`中定义。这些可以为所有任务定义，或者为单个任务单独定义。

如果定义了`MAILTO`环境变量，`cron`会通过电子邮件将命令的输出发送给该用户。

`crontab`通过在用户的`cron`表中插入带有变量赋值语句的一行来定义环境变量。

以下的`crontab`定义了一个`http_proxy`环境变量，用于在互联网交互时使用代理服务器：

```
    http_proxy=http://192.168.0.3:3128
 MAILTO=user@example.com
 00 * * * * /home/slynux/download.sh

```

该格式由`vixie-cron`支持，`vixie-cron`用于 Debian、Ubunto 和 CentOS 发行版。对于其他发行版，环境变量可以按命令逐个定义：

```
    00 * * * * http_proxy=http:192.168.0.2:3128;   
    /home/sylinux/download.sh

```

# 在系统启动时/开机时运行命令

在系统启动时（或开机时）运行特定命令是一个常见需求。一些`cron`实现支持`@reboot`时间字段，用于在重启过程中运行任务。请注意，并非所有`cron`实现都支持此功能，且在某些系统上只有 root 才能使用此功能。现在，看看以下代码：

```
    @reboot command

```

这将在运行时以你的用户身份执行该命令。

# 查看 cron 表

`-l`选项可以列出当前用户的 crontab：

```
    $ crontab -l
 02 05 * * * /home/user/disklog.sh

```

添加`-u`选项将指定一个用户的 crontab 进行查看。你必须以 root 身份登录才能使用`-u`选项：

```
    # crontab -l -u slynux
 09 10 * * * /home/slynux/test.sh

```

# 删除 cron 表

`-r`选项将删除当前用户的 cron 表：

```
    $ crontab -r

```

`-u`选项指定要删除的 crontab。你必须是 root 用户才能删除其他用户的 crontab：

```
    # crontab -u slynux -r

```

# 数据库样式及用途

Linux 支持多种数据库样式，从简单的文本文件（`/etc/passwd`）到低级 B-树数据库（Berkely DB 和 bdb），轻量级 SQL（sqlite）以及功能全面的关系型数据库服务器，如 Postgres、Oracle 和 MySQL。

选择数据库样式的一个经验法则是使用对你的应用最适合且最简单的系统。当字段已知且固定时，文本文件和`grep`足以作为一个小型数据库。

一些应用程序需要引用。例如，书籍和作者的数据库应创建两个表，一个用于书籍，一个用于作者，以避免为每本书重复作者信息。

如果表的读取频率远高于修改频率，那么 SQLite 是一个不错的选择。这个数据库引擎不需要服务器，便于移植并易于嵌入到其他应用程序中（如 Firefox）。

如果数据库经常被多个任务修改（例如，网店的库存系统），那么像 Postgres、Oracle 或 MySQL 这样的关系数据库管理系统（RDBMS）是合适的选择。

# 准备就绪

你可以使用标准 Shell 工具创建一个基于文本的数据库。SqlLite 通常默认安装；其可执行文件是 `sqlite3`。你需要安装 MySQL、Oracle 和 Postgres。下一节将介绍如何安装 MySQL。你可以从 www.oracle.com 下载 Oracle。Postgres 通常可以通过包管理器安装。

# 如何操作...

可以使用常见的 Shell 工具构建一个文本文件数据库。

要创建一个地址列表，创建一个每行一个地址的文件，并且字段由已知字符分隔。在这种情况下，字符是波浪号（`~`）：

```
    first last~Street~City, State~Country~Phone~

```

例如：

```
    Joe User~123 Example Street~AnyTown, District~1-123-123-1234~

```

然后添加一个函数，找到匹配模式的行并将每一行翻译成更易于理解的格式：

```
    function  addr {
 grep $1 $HOME/etc/addr.txt | sed 's/~/\n/g'
 }

```

使用时，这将类似于以下内容：

```
    $ addr Joe
 Joe User
 123 Example Street 
 AnyTown District
 1-123-123-1234

```

# 还有更多...

SQLite、Postgres、Oracle 和 MySQL 提供了一种更强大的数据库模型，称为关系型数据库。关系型数据库存储表之间的关系，例如书籍与作者之间的关系。

与关系型数据库交互的常见方式是使用 SQL 语言。SQLite、Postgres、Oracle、MySQL 以及其他数据库引擎都支持这种语言。

SQL 是一种功能丰富的语言，你可以阅读专门介绍它的书籍。幸运的是，使用 SQL 有效地只需要几个命令。

# 创建表格

使用 `CREATE TABLE` 命令定义表：

```
 CREATE TABLE tablename (field1 type1, field2 type2,...); 

```

下一行命令会创建一本书籍和作者的表格：

```
 CREATE TABLE book (title STRING, author STRING); 

```

# 向 SQL 数据库插入一行数据

插入命令会将一行数据插入到数据库中。

```
 INSERT INTO table (columns) VALUES (val1, val2,...); 

```

以下命令会插入你当前正在阅读的书籍：

```
 INSERT INTO book (title, author) VALUES ('Linux Shell Scripting 
 Cookbook', 'Clif Flynt'); 

```

# 从 SQL 数据库中选择行

select 命令会选择所有符合测试条件的行：

```
 SELECT fields FROM table WHERE test; 

```

这个命令会从书籍表中选择包含 "Shell" 这个词的书名：

```
 SELECT title FROM book WHERE title like '%Shell%'; 

```

# 编写和读取 SQLite 数据库

SQLite 是一个轻量级的数据库引擎，广泛应用于 Android 应用、Firefox 以及美国海军的库存系统等多个领域。由于其广泛的应用，运行 SQLite 的程序数量超过了其他任何数据库。

SQLite 数据库是一个单一文件，多个数据库引擎可以访问它。数据库引擎是一个 C 库，可以与应用程序链接；它也可以作为库加载到脚本语言中，如 TCL、Python 或 Perl，或者作为独立程序运行。

独立应用程序 sqlite3 是在 Shell 脚本中最容易使用的工具。

# 准备就绪

`sqlite3` 可执行文件可能没有在你的系统中安装。如果没有，你可以通过包管理器安装 `sqlite3` 包。

对于 Debian 和 Ubuntu，请使用以下命令：

```
    apt-get install sqlite3 libsqlite3-dev

```

对于 Red Hat、SuSE、Fedora 和 Centos，请使用以下命令：

```
    yum install sqlite sqlite-devel

```

# 如何操作...

`sqlite3`命令是一个交互式数据库引擎，连接到 SQLite 数据库，并支持创建表、插入数据、查询表等操作。

`sqlite3`命令的语法如下：

```
    sqlite3 databaseName

```

如果`databaseName`文件存在，`sqlite3`将打开它。如果文件不存在，`sqlite3`将创建一个空的数据库。在本教程中，我们将创建一个表，插入一行数据，并检索该条目：

```
    # Create a books database
 $ sqlite3 books.db
 sqlite> CREATE TABLE books (title string, author string);
 sqlite> INSERT INTO books (title, author) VALUES ('Linux Shell      
    Scripting Cookbook', 'Clif Flynt');
 sqlite> SELECT * FROM books WHERE author LIKE '%Flynt%';
 Linux Shell Scripting Cookbook|Clif Flynt

```

# 它的工作原理...

`sqlite3`应用程序创建一个名为`books.db`的空数据库，并显示`sqlite>`提示符以接受 SQL 命令。

`CREATE TABLE`命令创建一个具有两个字段的表：title（标题）和 author（作者）。

`INSERT`命令将一本书插入到数据库中。SQL 中的字符串用单引号括起来。

`SELECT`命令检索与测试匹配的行。百分号（`%`）是 SQL 中的通配符，类似于 shell 中的星号（`*`）。

# 还有更多...

一个 shell 脚本可以使用`sqlite3`访问数据库，并提供一个简单的用户界面。下一个脚本实现了前述的地址数据库，使用`sqlite`代替平面文本文件。它提供了三个命令：

+   `init`：这是创建数据库的操作

+   `insert`：这是添加新行的操作

+   `query`：这是选择与查询匹配的行

使用时，它看起来像这样：

```
    $> dbaddr.sh init
 $> dbaddr.sh insert 'Joe User' '123-1234' 'user@example.com'
 $> dbaddr.sh query name Joe
 Joe User
 123-1234
 user@example.com

```

以下脚本实现了这个数据库应用：

```
    #!/bin/bash
    # Create a command based on the first argument

    case $1 in
      init )
        cmd="CREATE TABLE address \
          (name string, phone string, email string);" ;;
      query )
        cmd="SELECT name, phone, email FROM address \
          WHERE $2 LIKE '$3';";;
      insert )
        cmd="INSERT INTO address (name, phone, email) \
          VALUES ( '$2', '$3', '$4' );";;
    esac

    # Send the command to sqlite3 and reformat the output

    echo $cmd | sqlite3 $HOME/addr.db | sed 's/|/\n/g'

```

这个脚本使用 case 语句来选择 SQL 命令字符串。其他命令行参数被替换为这个字符串，然后该字符串被发送到`sqlite3`进行评估。`$1`、`$2`、`$3`和`$4`分别是脚本的第一个、第二个、第三个和第四个参数。

# 从 Bash 写入和读取 MySQL 数据库

MySQL 是一个广泛使用的数据库管理系统。2009 年，Oracle 收购了 SUN，并随之收购了 MySQL 数据库。MariaDB 是 MySQL 的一个独立分支，与 Oracle 无关。MariaDB 可以访问 MySQL 数据库，但 MySQL 引擎并不总是能访问 MariaDB 数据库。

MySQL 和 MariaDB 都提供许多语言的接口，包括 PHP、Python、C++、Tcl 等。它们都使用`mysql`命令提供交互式会话，以便访问数据库。这是 shell 脚本与 MySQL 数据库交互的最简单方式。这些示例适用于 MySQL 或 MariaDB。

一个 bash 脚本可以将文本或**逗号分隔值**（**CSV**）文件转换为 MySQL 表格和行。例如，我们可以通过运行查询从 shell 脚本中读取存储在留言簿程序数据库中的所有电子邮件地址。

接下来的脚本演示了如何将文件的内容插入到学生数据库表中，并生成报告，同时对每个学生进行部门内排名。

# 准备就绪

MySQL 和 MariaDB 并非总是包含在基础 Linux 发行版中。它们可以作为 `mysql-server` 和 `mysql-client` 或 `mariadb-server` 包进行安装。MariaDB 发行版使用 MySQL 作为命令，并且有时在请求 MySQL 包时会一起安装。

MySQL 支持使用用户名和密码进行身份验证。在安装过程中，系统会提示输入密码。

使用 `mysql` 命令在全新安装的系统上创建数据库。在通过 `CREATE DATABASE` 命令创建数据库后，可以通过 `use` 命令选择使用该数据库。一旦选择了数据库，就可以使用标准 SQL 命令创建表并插入数据：

```
$> mysql -user=root -password=PASSWORD

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 44
Server version: 10.0.29-MariaDB-0+deb8u1 (Debian)

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE test1;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> use test1;

```

`quit` 命令或 Ctrl-D 将终止 `mysql` 的交互式会话。

# 如何操作...

本方案包含三个脚本：一个用于创建数据库和表，一个用于插入学生数据，另一个用于从表中读取并显示数据。

创建数据库和表的脚本：

```
#!/bin/bash 
#Filename: create_db.sh 
#Description: Create MySQL database and table 

USER="user" 
PASS="user" 

mysql -u $USER -p$PASS <<EOF 2> /dev/null 
CREATE DATABASE students; 
EOF 

[ $? -eq 0 ] && echo Created DB || echo DB already exist  
mysql -u $USER -p$PASS students <<EOF 2> /dev/null 
CREATE TABLE students( 
id int, 
name varchar(100), 
mark int, 
dept varchar(4) 
); 
EOF 

[ $? -eq 0 ] && echo Created table students || \
    echo Table students already exist  

mysql -u $USER -p$PASS students <<EOF 
DELETE FROM students; 
EOF 

```

该脚本将数据插入到表中：

```
#!/bin/bash 
#Filename: write_to_db.sh 
#Description: Read from CSV and write to MySQLdb 

USER="user" 
PASS="user" 

if [ $# -ne 1 ]; 
then 
  echo $0 DATAFILE 
  echo 
  exit 2 
fi 

data=$1 

while read line; 
do 

  oldIFS=$IFS 
  IFS=, 
  values=($line) 
  values[1]="\"`echo ${values[1]} | tr ' ' '#' `\"" 
  values[3]="\"`echo ${values[3]}`\"" 

  query=`echo ${values[@]} | tr ' #' ', ' ` 
  IFS=$oldIFS 

  mysql -u $USER -p$PASS students <<EOF 
INSERT INTO students VALUES($query); 
EOF 

done< $data 
echo Wrote data into DB 

```

最后的脚本查询数据库并生成报告：

```
#!/bin/bash 
#Filename: read_db.sh 
#Description: Read from the database 

USER="user" 
PASS="user" 

depts=`mysql -u $USER -p$PASS students <<EOF | tail -n +2 
SELECT DISTINCT dept FROM students; 
EOF` 

for d in $depts; 
do 

echo Department : $d 

result="`mysql -u $USER -p$PASS students <<EOF 
SET @i:=0; 
SELECT @i:=@i+1 as rank,name,mark FROM students WHERE dept="$d" ORDER BY mark DESC; 
EOF`" 

echo "$result" 
echo 

done 

```

输入的 CSV 文件（`studentdata.csv`）的数据大致如下：

```
1,Navin M,98,CS
2,Kavya N,70,CS
3,Nawaz O,80,CS
4,Hari S,80,EC
5,Alex M,50,EC
6,Neenu J,70,EC
7,Bob A,30,EC
8,Anu M,90,AE
9,Sruthi,89,AE
10,Andrew,89,AE

```

按照以下顺序执行脚本：

```
$ ./create_db.sh 
Created DB
Created table students

$ ./write_to_db.sh studentdat.csv
Wrote data into DB

$ ./read_db.sh 
Department : CS
rank  name  mark
1  Navin M  98
2  Nawaz O  80
3  Kavya N  70

Department : EC
rank  name  mark
1  Hari S  80
2  Neenu J 70
3  Alex M  50
4  Bob A   30

Department : AE
rank  name  mark
1  Anu M    90
2  Sruthi   89
3  Andrew   89

```

# 如何操作...

第一个脚本 `create_db.sh` 创建一个名为 `students` 的数据库，并在其中创建一个名为 `students` 的表。`mysql` 命令用于执行 MySQL 操作。`mysql` 命令通过 `-u` 指定用户名，通过 `-pPASSWORD` 指定密码。`USER` 和 `PASS` 变量用于存储用户名和密码。

`mysql` 命令的另一个参数是数据库名称。如果指定了数据库名称作为 `mysql` 命令的参数，则会使用该数据库；否则，我们必须通过 **use** `database_name` 命令显式定义要使用的数据库。

`mysql` 命令通过标准输入（`stdin`）接收要执行的查询。通过 `<<EOF` 方法提供多行输入是一种方便的方式。在 `<<EOF` 和 `EOF` 之间出现的文本将作为标准输入传递给 `mysql`。

`CREATE DATABASE` 和 `CREATE TABLE` 命令将 `stderr` 重定向到 `/dev/null`，以防止显示错误信息。该脚本检查存储在 `$?` 中的 `mysql` 命令的退出状态，以确定是否发生故障；如果发生故障，通常是因为表或数据库已存在。如果数据库或表已存在，脚本会显示一条消息通知用户；否则，将创建数据库和表。

`write_to_db.sh` 脚本接受学生数据 CSV 文件的文件名。它在 `while` 循环中读取 CSV 文件的每一行。在每次迭代中，CSV 文件中的一行被读取并重新格式化为 SQL 命令。脚本将逗号分隔的行数据存储到一个数组中。数组赋值的形式如下：`array=(val1 val2 val3)`。在这里，空格字符是**内部字段分隔符**（**IFS**）。这些数据是逗号分隔的值。通过将 IFS 设置为逗号，我们可以轻松地将值赋给数组（`IFS=,`）。

逗号分隔行中的数据元素是 `id`、`name`、`mark` 和 `department`。`id` 和 `mark` 值是整数，而 `name` 和 `dept` 是必须引用的字符串。

姓名可能包含空格字符，这会与 IFS 冲突。脚本将名称中的空格替换为字符（`#`），并在生成查询后恢复。

为了引用字符串，数组中的值会重新分配，前后加上 `\"` 前缀和后缀。`tr` 命令将名称中的每个空格替换为 `#`。

最后，通过将空格字符替换为逗号并将 `#` 替换为空格，生成查询。然后，执行 SQL 的 `INSERT` 命令。

第三个脚本 `read_db.sh` 生成按排名顺序排列的各个部门的学生名单。第一个查询查找部门的不同名称。我们使用 `while` 循环遍历每个部门并运行查询，按最高分数显示学生的详细信息。`SET @i=0` 是一个 SQL 结构，用于设置 `i=0`。在每一行中，它会递增并显示为学生的排名。

# 用户管理脚本

GNU/Linux 是一个多用户操作系统，允许多个用户同时登录并执行活动。涉及用户管理的管理任务包括设置用户的默认 shell、将用户添加到组、禁用 shell 账户、添加新用户、删除用户、设置密码、设置用户账户的到期日期等。本食谱展示了一个用户管理工具来处理这些任务。

# 如何操作……

该脚本执行常见的用户管理任务：

```
#!/bin/bash 
#Filename: user_adm.sh 
#Description: A user administration tool 

function usage() 
{ 
  echo Usage: 
  echo Add a new user 
  echo $0 -adduser username password 
  echo 
  echo Remove an existing user 
  echo $0 -deluser username 
  echo 
  echo Set the default shell for the user 
  echo $0 -shell username SHELL_PATH 
  echo 
  echo Suspend a user account 
  echo $0 -disable username 
  echo 
  echo Enable a suspended user account 
  echo $0 -enable username 
  echo 
  echo Set expiry date for user account 
  echo $0 -expiry DATE  
  echo 
  echo Change password for user account 
  echo $0 -passwd username 
  echo 
  echo Create a new user group 
  echo $0 -newgroup groupname 
  echo 
  echo Remove an existing user group 
  echo $0 -delgroup groupname 
  echo 
  echo Add a user to a group 
  echo $0 -addgroup username groupname 
  echo 
  echo Show details about a user 
  echo $0 -details username 
  echo 
  echo Show usage 
  echo $0 -usage 
  echo 

  exit 
} 

if [ $UID -ne 0 ]; 
then 
  echo Run $0 as root. 
  exit 2 
fi 

case $1 in 

  -adduser) [ $# -ne 3 ] && usage ; useradd $2 -p $3 -m ;;  
  -deluser) [ $# -ne 2 ] && usage ; deluser $2 --remove-all-files;; 
  -shell)    [ $# -ne 3 ] && usage ; chsh $2 -s $3 ;; 
  -disable) [ $# -ne 2 ] && usage ; usermod -L $2 ;;  
  -enable) [ $# -ne 2 ] && usage ; usermod -U $2  ;; 
  -expiry) [ $# -ne 3 ] && usage ; chage $2 -E $3 ;; 
  -passwd) [ $# -ne 2 ] && usage ; passwd $2 ;; 
  -newgroup) [ $# -ne 2 ] && usage ; addgroup $2 ;; 
  -delgroup) [ $# -ne 2 ] && usage ; delgroup $2 ;; 
  -addgroup) [ $# -ne 3 ] && usage ; addgroup $2 $3 ;; 
  -details) [ $# -ne 2 ] && usage ; finger $2 ; chage -l $2 ;; 
  -usage) usage ;; 
  *) usage ;; 
esac 

```

一个示例输出类似于以下内容：

```
# ./user_adm.sh -details test
Login: test                 Name: 
Directory: /home/test                 Shell: /bin/sh
Last login Tue Dec 21 00:07 (IST) on pts/1 from localhost
No mail.
No Plan.
Last password change          : Dec 20, 2010
Password expires          : never
Password inactive         : never
Account expires             : Oct 10, 2010
Minimum number of days between password change    : 0
Maximum number of days between password change    : 99999
Number of days of warning before password expires  : 7

```

# 它是如何工作的……

`user_adm.sh` 脚本执行几个常见的用户管理任务。`usage()` 文本解释了当用户提供错误参数或包含 `-usage` 参数时，如何使用该脚本。一个 `case` 语句解析命令参数并执行适当的命令。

`user_adm.sh` 脚本的有效命令选项有：`-adduser`、`-deluser`、`-shell`、`-disable`、`-enable`、`-expiry`、`-passwd`、`-newgroup`、`-delgroup`、`-addgroup`、`-details` 和 `-usage`。当匹配到 `*)` 的情况下，表示没有识别到选项，因此会调用 `usage()`。

以 root 用户身份运行此脚本。在检查参数之前，它会确认用户 ID（root 的用户 ID 为 `0`）。

当参数匹配时，`[ $# -ne 3 ] &&` 测试用法检查参数数量。如果命令的参数数量与要求的数量不匹配，则会调用 `usage()` 函数，并退出脚本。

以下脚本支持这些选项：

+   `-useradd`：`useradd` 命令用于创建新用户：

```
        useradd USER -p PASSWORD -m

```

+   `-m` 选项会创建主目录。

+   `-deluser`：`deluser` 命令用于删除用户：

```
        deluser USER --remove-all-files

```

+   `--remove-all-files` 选项会删除与用户相关的所有文件，包括 `home` 目录。

+   `-shell`：`chsh` 命令用于更改用户的默认 shell：

```
        chsh USER -s SHELL

```

+   `-disable` 和 `-enable`：`usermod` 命令用于操作与用户账户相关的多个属性。`usermod -L USER` 锁定用户账户，`usermod -U USER` 解锁用户账户。

+   `-expiry`：`change` 命令用于操作用户账户的过期信息：

```
        chage -E DATE

```

支持以下选项：

+   `-m MIN_DAYS`：此选项设置密码更改之间的最小天数为 `MIN_DAYS`。

+   +   `-M MAX_DAYS`：此选项设置密码的最大有效天数。

    +   `-W WARN_DAYS`：此选项设置在密码更改前提供警告的天数。

+   `-passwd`：`passwd` 命令用于更改用户密码：

```
        passwd USER

```

该命令将提示输入新密码：

+   `-newgroup` 和 `-addgroup`：`addgroup` 命令用于向系统添加新用户组：

```
        addgroup GROUP

```

如果包含用户名，它将把该用户添加到一个组中：

```
        addgroup USER GROUP
 -delgroup

```

`delgroup` 命令用于删除用户组：

```
        delgroup GROUP

```

+   `-details`：`finger USER` 命令显示用户信息，包括主目录、最后登录时间、默认 shell 等。`chage -l` 命令显示用户账户过期信息。

# 批量图像缩放和格式转换

我们所有人都会从手机和相机中下载照片。在通过电子邮件发送图像或发布到网上之前，我们可能需要调整其大小，或许还需要更改格式。我们可以使用脚本批量修改这些图像文件。本食谱描述了图像管理的相关操作。

# 准备就绪

**ImageMagick** 套件中的 `convert` 命令包含用于操作图像的工具。它支持多种图像格式和转换选项。大多数 GNU/Linux 发行版默认不包含 ImageMagick，你需要手动安装该软件包。有关更多信息，请在浏览器中访问 [www.imagemagick.org](http://www.imagemagick.org)。

# 如何操作...

`convert` 程序将文件从一种图像格式转换为另一种格式：

```
    $ convert INPUT_FILE OUTPUT_FILE

```

这是一个示例：

```
    $ convert file1.jpg file1.png

```

我们可以通过指定缩放百分比或输出图像的宽度和高度来调整图像大小。要通过指定 `WIDTH` 或 `HEIGHT` 来调整图像大小，请使用此命令：

```
    $ convert imageOrig.png -resize WIDTHxHEIGHT imageResized.png

```

这是一个示例：

```
    $ convert photo.png -resize 1024x768 wallpaper.png

```

如果缺少 `WIDTH` 或 `HEIGHT`，那么缺少的部分将自动计算以保持图像的长宽比：

```
    $ convert image.png -resize WIDTHx image.png

```

这是一个示例：

```
    $ convert image.png -resize 1024x image.png

```

要通过指定百分比缩放因子来调整图像大小，请使用此命令：

```
    $ convert image.png -resize "50%" image.png

```

这个脚本将对目录中的所有图像执行一系列操作：

```
#!/bin/bash 
#Filename: image_help.sh 
#Description: A script for image management 

if [ $# -ne 4 -a $# -ne 6 -a $# -ne 8 ]; 
then 
  echo Incorrect number of arguments 
  exit 2 
fi 

while [ $# -ne 0 ]; 
do 

  case $1 in 
  -source) shift; source_dir=$1 ; shift ;; 
  -scale) shift; scale=$1 ; shift ;; 
  -percent) shift; percent=$1 ; shift ;; 
  -dest) shift ; dest_dir=$1 ; shift ;; 
  -ext) shift ; ext=$1 ; shift ;; 
  *) echo Wrong parameters; exit 2 ;; 
  esac; 

done 

for img in `echo $source_dir/*` ; 
do 
  source_file=$img 
  if [[ -n $ext ]]; 
  then 
    dest_file=${img%.*}.$ext 
  else 
    dest_file=$img 
  fi 

  if [[ -n $dest_dir ]]; 
  then 
    dest_file=${dest_file##*/} 
    dest_file="$dest_dir/$dest_file" 
  fi 

  if [[ -n $scale ]]; 
  then 
    PARAM="-resize $scale" 
  elif [[ -n $percent ]];   then 
    PARAM="-resize $percent%"  
  fi 

  echo Processing file : $source_file 
  convert $source_file $PARAM $dest_file 

done 

```

以下示例将`sample_dir`目录中的图像缩放至`20%`：

```
$ ./image_help.sh -source sample_dir -percent 20%
Processing file :sample/IMG_4455.JPG
Processing file :sample/IMG_4456.JPG
Processing file :sample/IMG_4457.JPG
Processing file :sample/IMG_4458.JPG

```

要将图像的宽度调整为`1024`，请使用以下命令：

```
$ ./image_help.sh -source sample_dir -scale 1024x

```

要将文件缩放并转换到指定的目标目录，请使用以下命令：

```
# newdir is the new destination directory
$ ./image_help.sh -source sample -scale 50% -ext png -dest newdir

```

# 它是如何工作的...

上述`image_help.sh`脚本接受以下参数：

+   `-source`：指定图像的源目录。

+   `-dest`：指定转换后的图像文件的目标目录。如果未指定`-dest`，目标目录将与源目录相同。

+   `-ext`：指定转换后的目标文件格式。

+   `-percent`：指定缩放的百分比。

+   `-scale`：指定缩放后的宽度和高度。

+   `-percent`和`-scale`两个参数可能都不出现。

+   脚本首先检查命令参数的数量。四个、六个或八个参数是有效的。

命令行通过`while`循环和`case`语句进行解析，值被分配到适当的变量中。`$#`是一个特殊变量，包含参数的数量。`shift`命令将命令参数向左移动一个位置。这样，每次移位时，我们可以通过`$1`访问下一个命令参数，而不必使用`$1`、`$2`、`$3`等。

`case`语句类似于 C 语言中的 switch 语句。当匹配到某个 case 时，执行相应的语句。每个匹配的语句以`;;`结束。所有参数被解析到变量`percent`、`scale`、`source_dir`、`ext`和`dest_dir`后，`for`循环将遍历源目录中的每个文件并进行转换。

在`for`循环中执行了几个测试，以精细调整转换。

如果定义了变量`ext`（即命令参数中给出了`-ext`），目标文件的扩展名将从`source_file.extension`更改为`source_file.$ext`。

如果提供了`-dest`参数，目标文件路径将通过将源路径中的目录替换为目标目录来修改。

如果指定了-scale 或-percent 选项，则会将调整大小参数（`-resize widthx`或`-resize perc%`）添加到命令中。

参数评估后，执行`convert`命令并使用适当的参数。

# 另请参见

+   第二章《掌握命令》中的*基于扩展名切割文件名*食谱解释了如何提取文件名的某一部分。

# 从终端截图

随着图形用户界面（GUI）应用程序的增多，截图变得越来越重要，既可以记录操作过程，又可以报告意外结果。Linux 支持多种抓取截图的工具。

# 准备工作

本节将介绍**xwd**应用程序以及之前食谱中使用的 ImageMagick 工具。xwd 应用程序通常与基本的 GUI 一起安装。您可以使用包管理器安装 ImageMagick。

# 如何操作...

xwd 程序从窗口中提取视觉信息，将其转换为 X Window Dump 格式，并将数据输出到 `stdout`。该输出可以重定向到文件，文件可以转换为 GIF、PNG 或 JPEG 格式，如前面的示例所示。

当调用 xwd 时，它会将光标更改为十字准线。将该准线移动到一个 X Window 并点击它，窗口即被抓取：

```
    $ xwd >step1.xwd

```

ImageMagick 的 `import` 命令支持更多截屏选项：

要截取整个屏幕的截图，使用以下命令：

```
    $ import -window root screenshot.png

```

你可以手动选择一个区域并使用以下命令截取该区域的截图：

```
    $ import screenshot.png

```

要截取特定窗口的截图，使用以下命令：

```
    $ import -window window_id screenshot.png

```

`xwininfo` 命令会返回一个窗口 ID。运行该命令并点击你想要的窗口。然后，将这个 `window_id` 值传递给 `import` 命令的 `-window` 选项。

# 从一个终端管理多个终端

SSH 会话、Konsoles 和 xterms 是适用于长期运行应用的重型解决方案，但它们执行检查的频率较低（例如监视日志文件或磁盘使用情况）。

GNU screen 工具在终端会话中创建多个虚拟屏幕。你在虚拟屏幕中启动的任务，在屏幕隐藏时仍会继续运行。

# 准备就绪

为了实现这一点，我们将使用一个名为 **GNU screen** 的工具。如果你的发行版默认没有安装 screen，可以通过包管理器安装：

```
    apt-get install screen

```

# 如何操作...

1.  一旦屏幕工具创建了一个新窗口，所有的键盘输入都会传递给该窗口中运行的任务，除了 Control-A (*Ctrl*-*A*)，它标志着一个屏幕命令的开始。

1.  **创建屏幕窗口**：要创建一个新的屏幕，运行 shell 中的 screen 命令。你将看到包含屏幕信息的欢迎消息。按空格键或回车键返回到 shell 提示符。要创建一个新的虚拟终端，按 *Ctrl* + *A* 然后按 *C*（区分大小写），或者再次输入 screen。

1.  **查看打开窗口的列表**：在运行 screen 时，按 *Ctrl*+*A* 然后按引号 (`"`) 会列出你的终端会话。

1.  **在窗口间切换**：按 *Ctrl* + *A* 和 *Ctrl* + *N* 可以显示下一个窗口，按 *Ctrl* + *A* 和 *Ctrl* + *P* 可以显示上一个窗口。

1.  **附加和分离屏幕**：screen 命令支持保存和加载屏幕会话，在 screen 术语中称为分离和附加。要从当前屏幕会话分离，按 *Ctrl* + *A* 然后按 *Ctrl* + *D*。要在启动屏幕时附加到现有屏幕，使用：

```
        screen -r -d

```

1.  这告诉屏幕附加到最后一个屏幕会话。如果你有多个分离的会话，屏幕会输出一个列表；然后使用：

```
        screen -r -d PID

```

这里，`PID` 是你想要附加的屏幕会话的 PID。
