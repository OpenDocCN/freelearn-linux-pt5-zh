# 第九章：戴上监控帽

本章将涵盖以下食谱：

+   监控磁盘使用情况

+   计算命令的执行时间

+   收集已登录用户的信息、启动日志和启动失败信息

+   列出一个小时内 CPU 消耗最多的前十个进程

+   使用 `watch` 监控命令输出

+   记录文件和目录的访问

+   使用 syslog 进行日志记录

+   使用 `logrotate` 管理日志文件

+   监控用户登录以发现入侵者

+   监控远程磁盘使用健康状况

+   确定系统上的活跃用户时段

+   测量和优化电源使用

+   监控磁盘活动

+   检查磁盘和文件系统的错误

+   检查磁盘健康状况

+   获取磁盘统计信息

# 介绍

计算机系统是硬件和控制它的软件组件的集合。软件包括操作系统内核，它分配资源，以及许多执行各个任务的模块，从读取磁盘数据到提供网页服务等。

管理员需要监控这些模块和应用程序，以确认它们是否正常工作，并了解是否需要重新分配资源（例如将用户分区迁移到更大的磁盘、提供更快的网络等）。

Linux 提供了交互式程序来检查系统当前的性能，以及用于记录性能随时间变化的模块。

本章介绍了监控系统活动的命令，并讨论了日志记录技术。

# 监控磁盘使用情况

磁盘空间始终是有限的资源。我们监控磁盘使用情况，以便在磁盘空间不足时及时发现，然后搜索大文件或文件夹以删除、移动或压缩。本食谱展示了磁盘监控命令。

# 准备工作

`du`（磁盘使用）和 `df`（磁盘空闲）命令用于报告磁盘使用情况。这些工具报告哪些文件和文件夹正在占用磁盘空间，以及剩余多少可用空间。

# 如何操作...

要查找文件（或文件）所占用的磁盘空间，请使用以下命令：

```
    $ du  FILENAME1 FILENAME2 ..

```

请参考这个例子：

```
    $ du file.txt

```

要获取目录内所有文件的磁盘使用情况，并在每一行中显示每个文件的单独磁盘使用情况，请使用以下命令：

```
    $ du -a DIRECTORY

```

`-a` 选项递归地输出指定目录或目录中所有文件的结果。

运行 `du DIRECTORY` 将输出类似的结果，但它只会显示子目录所占用的大小。然而，这并不显示每个文件的磁盘使用情况。要打印每个文件的磁盘使用情况，必须使用 `-a`。

请参考这个例子：

```
    $  du -a test
 4  test/output.txt
 4  test/process_log.sh
 4  test/pcpu.sh
 16  test

```

`du` 命令可以用于查看目录：

```
    $ du test
 16  test

```

# 还有更多...

`du` 命令包括定义如何报告数据的选项。

# 以 KB、MB 或块为单位显示磁盘使用情况

默认情况下，磁盘使用命令显示文件所使用的总字节数。更易读的格式以 KB、MB 或 GB 等单位表示。`-h` 选项以人类可读的格式显示结果：

```
    du -h FILENAME

```

请参考这个例子：

```
    $ du -h test/pcpu.sh
 4.0K  test/pcpu.sh
 # Multiple file arguments are accepted

```

或者，像这样使用：

```
    # du -h DIRECTORY
 $ du -h hack/
 16K  hack/

```

# 显示磁盘使用情况的总和

`-c`选项将计算文件或目录使用的总大小，并显示每个文件的大小：

```
    $ du -c FILENAME1 FILENAME2..
 du -c process_log.sh pcpu.sh
 4  process_log.sh
 4  pcpu.sh
 8  total

```

或者，可以像这样使用它：

```
    $ du  -c DIRECTORY
 $ du -c test/
 16  test/
 16  total

```

或者：

```
 $ du -c *.txt
 # Wildcards

```

`-c`选项可以与`-a`和`-h`等选项一起使用，生成常规输出，并附加一行包含总大小的额外信息。

`-s`选项（汇总），将打印总计作为输出。`-h`标志可以与其一起使用，以以人类可读的格式打印：

```
    $ du -sh /usr/bin
 256M   /usr/bin

```

# 以指定单位打印大小

`-b`、`-k`和`-m`选项将强制`du`以指定的单位打印磁盘使用情况。请注意，这些选项不能与`-h`选项一起使用：

+   以字节为单位打印大小（默认）：

```
        $ du -b FILE(s)

```

+   以千字节为单位打印大小：

```
        $ du -k FILE(s)

```

+   以兆字节为单位打印大小：

```
        $ du -m FILE(s)

```

+   以指定的给定`BLOCK`大小打印大小：

```
        $ du -B BLOCK_SIZE FILE(s)

```

这里，`BLOCK_SIZE`以字节为单位指定。

请注意，返回的文件大小并不直观明显。使用`-b`选项时，`du`报告文件的确切字节数。使用其他选项时，`du`报告文件使用的磁盘空间量。由于磁盘空间是以固定大小的块（通常是 4K）分配的，一个 400 字节的文件将占用一个块（4K）的空间：

```
        $ du pcpu.sh
 4  pcpu.sh
 $ du -b pcpu.sh
 439 pcpu.sh
 $ du -k pcpu.sh
 4  pcpu.sh
 $ du -m pcpu.sh
 1  pcpu.sh
 $ du -B 4 pcpu.sh
 1024  pcpu.sh

```

# 在磁盘使用计算中排除文件

`--exclude`和`-exclude-from`选项会导致`du`排除文件的磁盘使用计算。

+   `-exclude`选项可以与通配符或单个文件名一起使用：

```
        $ du --exclude "WILDCARD" DIRECTORY

```

考虑以下示例：

```
        # Excludes all .txt files from calculation
 $ du --exclude "*.txt" *
 # Exclude temp.txt from calculation
 $ du --exclude "temp.txt" *

```

+   `--exclude`选项将排除一个文件或与模式匹配的文件。`-exclude-from`选项允许排除更多的文件或模式。每个文件名或模式必须单独占一行。

```
        $ ls *.txt >EXCLUDE.txt
        $ ls *.odt >>EXCLUDE.txt
 # EXCLUDE.txt contains a list of all .txt and .odt files. 
 $ du --exclude-from EXCLUDE.txt DIRECTORY

```

`-max-depth`选项限制`du`将检查的子目录层级。深度为`1`时计算当前目录的磁盘使用情况，深度为`2`时计算当前目录和下一级子目录的使用情况：

```
    $ du --max-depth 2 DIRECTORY

```

`-x`选项将`du`限制为单个文件系统。`du`的默认行为是跟随链接和挂载点。

`du`命令需要对所有文件的读取权限，并且对所有目录需要读取和执行权限。如果执行该命令的用户没有适当的权限，`du`命令将抛出错误。

# 从给定目录中找到十个最大的文件

将`du`和排序命令结合起来，找出应删除或移动的大文件：

```
    $ du -ak SOURCE_DIR | sort -nrk 1 | head

```

`-a`选项使`du`显示`SOURCE_DIR`中所有文件和目录的大小。输出的第一列是大小。`-k`选项使其以千字节为单位显示。第二列包含文件或文件夹名称。

`-n`选项对`sort`执行数字排序。`-1`选项指定列`1`，`-r`选项则会反转排序顺序。`head`命令从输出中提取前十行：

```
    $ du -ak /home/slynux | sort -nrk 1 | head -n 4
 50220 /home/slynux
 43296 /home/slynux/.mozilla
 43284 /home/slynux/.mozilla/firefox
 43276 /home/slynux/.mozilla/firefox/8c22khxc.default

```

这个单行命令的一个缺点是它包括了目录在结果中。我们可以改进这个命令，通过`find`命令只输出大文件：

```
    $ find . -type f -exec du -k {} \; | sort -nrk 1 | head

```

`find`命令仅选择文件名供`du`处理，而不是让`du`遍历文件系统来选择要报告的项目。

请注意，`du`命令报告一个文件所需的字节数。这不一定与文件实际占用的磁盘空间相同。磁盘空间是按块分配的，所以一个 1 字节的文件会占用一个磁盘块，通常大小在 512 到 4096 字节之间。

下一部分描述如何使用`df`命令来确定实际可用的空间。

# 磁盘空闲信息

`du`命令提供使用情况的信息，而`df`提供有关可用磁盘空间的信息。使用`-h`选项与`df`一起使用，可以以人类可读的格式打印磁盘空间。考虑这个例子：

```
    $ df -h
 Filesystem            Size  Used Avail Use% Mounted on
 /dev/sda1             9.2G  2.2G  6.6G  25% /
 none                  497M  240K  497M   1% /dev
 none                  502M  168K  501M   1% /dev/shm
 none                  502M   88K  501M   1% /var/run
 none                  502M     0  502M   0% /var/lock
 none                  502M     0  502M   0% /lib/init/rw
 none                  9.2G  2.2G  6.6G  25%   
    /var/lib/ureadahead/debugfs

```

`df`命令可以与文件夹名称一起调用。在这种情况下，它将报告包含该目录的磁盘分区的空闲空间。如果你不知道哪个分区包含该目录，这将非常有用：

```
    $ df -h /home/user
 Filesystem            Size  Used Avail Use% Mounted on
 /dev/md1              917G  739G  133G  85% /raid1

```

# 计算命令的执行时间

执行时间是分析应用程序效率或比较算法的标准。

# 如何操作...

1.  `time`命令用于衡量应用程序的执行时间。

考虑以下例子：

```
        $ time APPLICATION

```

`time`命令执行`APPLICATION`。当`APPLICATION`完成时，`time`命令将实际时间、系统时间和用户时间统计信息报告到`stderr`，并将`APPLICATION`的正常输出发送到`stdout`。

```
        $ time ls
 test.txt
 next.txt
 real    0m0.008s
 user    0m0.001s
 sys     0m0.003s

```

`time`命令的可执行二进制文件位于`/usr/bin/time`。如果你使用的是 bash，默认情况下会得到 shell 内置的`time`命令。内置的`time`命令选项有限。使用绝对路径（`/usr/bin/time`）可以访问扩展功能。

1.  `-o`选项将时间统计信息写入文件：

```
        $ /usr/bin/time -o output.txt COMMAND

```

文件名必须紧跟在`-o`标志后面。

`-a`标志可以与`-o`一起使用，将时间统计信息附加到文件中：

```
        $ /usr/bin/time -a -o output.txt COMMAND

```

1.  `-f`选项指定要报告的统计信息及输出格式。格式字符串包括一个或多个以`%`为前缀的参数。格式参数包括以下内容：

+   实际时间：`%e`

+   用户时间：`%U`

+   系统时间：`%S`

+   系统页面大小：`%Z`

我们可以通过将这些参数与额外的文本结合，创建格式化的输出：

```
        $ /usr/bin/time -f "FORMAT STRING" COMMAND

```

考虑这个例子：

```
        $ /usr/bin/time -f "Time: %U" -a -o timing.log uname
 Linux

```

`%U`参数指定用户时间。

**time**命令将目标应用程序的输出发送到`stdout`，并将`time`命令的输出发送到`stderr`。我们可以使用重定向操作符（`>`）重定向输出，并使用错误重定向操作符（`2>`）重定向时间信息输出。

考虑以下例子：

```
        $ /usr/bin/time -f "Time: %U" uname> command_output.txt   
        2>time.log
 $ cat time.log
 Time: 0.00
 $ cat command_output.txt
 Linux

```

1.  格式命令可以报告内存使用情况以及计时信息。`%M`标志显示最大使用内存（以 KB 为单位），`%Z`参数使得时间命令报告系统页面大小：

```
        $ /usr/bin/time -f "Max: %M K\nPage size: %Z bytes" \                      
          ls>   
        /dev/null
 Max: 996 K
 Page size: 4096 bytes

```

在这个例子中，目标应用程序的输出并不重要，因此标准输出被重定向到`/dev/null`，而不是显示出来。

# 它是如何工作的...

`time`命令默认报告以下时间：

+   **Real**：这是墙钟时间——命令从开始到结束的时间。包括其他进程使用的时间片以及进程在被阻塞时所花费的时间（例如，等待 I/O 完成的时间）。

+   **User**：这是进程在用户模式代码（内核外部）中消耗的 CPU 时间量。它是执行进程所用的 CPU 时间。其他进程以及这些进程在被阻塞时所花费的时间不计入此项。

+   **Sys**：这是进程在内核中消耗的 CPU 时间量；指的是在内核中进行系统调用时消耗的 CPU 时间，而不是运行在用户空间中的库代码。像用户时间一样，这只是进程使用的 CPU 时间。请参考下表，简要描述内核模式（也称为主管模式）和系统调用机制。

`time` 命令可以报告进程的许多细节，包括退出状态、接收到的信号数量和进行的上下文切换次数。每个参数都可以在提供适当格式字符串给 `-f` 选项时显示。

下表展示了一些有趣的参数：

| **Parameter** | **Description** |
| --- | --- |
| `%C` | 显示正在计时的命令的名称和命令行参数。 |
| `%D` | 显示进程未共享数据区域的平均大小，单位是千字节。 |
| `%E` | 显示进程使用的实际（墙钟）时间，格式为 [小时：]分钟：秒。 |
| `%x` | 显示命令的退出状态。 |
| `%k` | 显示发送到进程的信号数量。 |
| `%W` | 显示进程被交换出主内存的次数。 |
| `%Z` | 显示系统的页面大小，单位是字节。这是一个每个系统的常量，但在不同的系统之间会有所不同。 |
| `%P` | 显示该作业获得的 CPU 百分比。它是用户时间 + 系统时间除以总运行时间的结果，并显示百分号。 |
| `%K` | 显示进程的平均总内存使用量（数据 + 堆栈 + 文本），单位是千字节。 |
| `%w` | 显示程序自愿进行上下文切换的次数，例如，在等待 I/O 操作完成时。 |
| `%c` | 显示进程被强制上下文切换的次数（因为时间片用完）。 |

# 收集有关登录用户、启动日志和启动失败的信息

Linux 支持报告运行时系统方面的命令，包括登录用户、计算机开机时长和启动失败。这些数据用于分配资源和诊断问题。

# 准备工作

本文介绍了 who、w、users、uptime、last 和 lastb 命令。

# 如何操作...

1.  `who` 命令报告当前用户的信息：

```
        $ who
 slynux   pts/0   2010-09-29 05:24 (slynuxs-macbook-pro.local)
 slynux   tty7    2010-09-29 07:08 (:0) 

```

该输出列出了登录名、用户使用的 TTY、登录时间以及关于登录用户的远程主机名（或 X 显示信息）。

**TTY**（这个术语源自**TeleTYpewriter**）是与文本终端相关联的设备文件，它在用户新创建终端时会在`/dev`目录下生成（例如，`/dev/pts/3`）。可以通过执行`tty`命令找到当前终端的设备路径。

1.  `w`命令提供更详细的信息：

```
 $ w
 07:09:05 up  1:45,  2 users,  load average: 0.12, 0.06, 0.02
 USER     TTY     FROM    LOGIN@   IDLE  JCPU PCPU WHAT
 slynux   pts/0   slynuxs 05:24  0.00s  0.65s 0.11s sshd: slynux 
 slynux   tty7    :0      07:08  1:45m  3.28s 0.26s bash

```

第一行列出了当前时间、系统开机时间、当前登录的用户数，以及过去 1、5 和 15 分钟的系统负载平均值。接下来，关于每个登录会话的详细信息会显示出来，每一行都包含登录名、TTY 名称、远程主机、登录时间、空闲时间、用户自登录以来的总 CPU 时间、当前进程的 CPU 时间以及当前进程的命令行。

`uptime`命令输出中的负载平均值表示系统负载。更详细的说明可以参见第十章，*系统管理调用*。

1.  `users`命令仅列出登录用户的名字：

```
        $ users
 slynux slynux slynux hacker

```

如果用户打开了多个会话，比如多次远程登录或打开多个终端窗口，那么每个会话都会有一个条目。在上述输出中，`slynux`用户打开了三个终端会话。最简单的方法是通过`sort`和`uniq`命令来过滤输出，以打印唯一的用户：

```
        $ users | tr ' ' '\n' | sort | uniq
 slynux
 hacker

```

`tr`命令将每个空格字符`' '`替换为换行符`'\n'`。然后，`sort`和`uniq`的组合将列表减少为每个用户的唯一条目。

1.  `uptime`命令报告系统已开机的时长：

```
        $ uptime
 21:44:33 up 6 days, 11:53, 8 users, load average: 0.09, 0.14,   
        0.09

```

紧随`up`一词后的时间表示系统已开机的时长。我们可以编写一个一行命令，仅提取系统的开机时间：

```
       $ uptime | sed 's/.*up \(.*\),.*users.*/\1/'

```

这使用`sed`命令将输出行替换为仅包含“up”一词和“users”前面的逗号之间的字符串。

1.  `last`命令提供自`/var/log/wtmp`文件创建以来，所有登录系统的用户列表。这个列表可能会回溯一年或更长时间：

```
 $ last
        aku1  pts/3   10.2.1.3   Tue May 16 08:23 - 16:14  (07:51)    
        cfly  pts/0   cflynt.com Tue May 16 07:49   still logged in   
        dgpx  pts/0   10.0.0.5   Tue May 16 06:19 - 06:27  (00:07)    
        stvl  pts/0   10.2.1.4   Mon May 15 18:38 - 19:07  (00:29)

```

`last`命令报告谁登录了，分配给他们的`tty`，他们从哪里登录（IP 地址或本地终端），登录时间、注销时间以及会话时长。重启会被伪用户`reboot`标记为登录。

1.  `last`命令允许你定义一个用户，仅获取该用户的信息：

```
        $ last USER

```

1.  用户（USER）可以是真实用户或伪用户`reboot`：

```
        $ last reboot
 reboot   system boot  2.6.32-21-generi Tue Sep 28 18:10 - 21:48    
        (03:37)
 reboot   system boot  2.6.32-21-generi Tue Sep 28 05:14 - 21:48    
        (16:33)

```

1.  `lastb`命令会列出所有失败的登录尝试：

```
        # lastb
 test     tty8         :0               Wed Dec 15 03:56 - 03:56    
        (00:00) 
 slynux   tty8         :0               Wed Dec 15 03:55 - 03:55    
        (00:00)

```

`lastb`命令必须以 root 用户身份运行。

`last`和`lastb`命令报告`/var/log/wtmp`文件的内容。默认情况下，报告事件的月份、日期和时间。然而，该文件中可能有多年的数据，月份和日期可能会让人困惑。

`-F`标志会报告完整日期：

```
        # lastb -F
 hacker   tty0       1.2.3.4          Sat Jan 7 11:50:53 2017 -  
        Sat Jan 7 11:50:53 2017 (00:00)

```

# 列出一个小时内 CPU 消耗最多的十个进程

CPU 是另一个可以被不当行为的进程耗尽的资源。Linux 支持命令来识别并控制占用 CPU 资源的进程。

# 准备工作

`ps`命令显示系统中运行的进程的详细信息。它报告诸如 CPU 使用率、运行命令、内存使用情况和进程状态等详细信息。`ps`命令可以在脚本中使用，以识别在一小时内消耗最多 CPU 资源的进程。有关`ps`命令的更多详细信息，请参阅第十章《系统管理命令》。

# 如何实现...

这个 shell 脚本监视并计算 CPU 使用率，持续一个小时：

```
#!/bin/bash
#Name: pcpu_usage.sh
#Description: Script to calculate cpu usage by processes for 1 hour

#Change the SECS to total seconds to monitor CPU usage.
#UNIT_TIME is the interval in seconds between each sampling

SECS=3600
UNIT_TIME=60

STEPS=$(( $SECS / $UNIT_TIME ))

echo Watching CPU usage... ;

# Collect data in temp file

for((i=0;i<STEPS;i++))
do
 ps -eocomm,pcpu | egrep -v '(0.0)|(%CPU)' >> /tmp/cpu_usage.$$
 sleep $UNIT_TIME
done

# Process collected data
echo
echo CPU eaters :

cat /tmp/cpu_usage.$$ | \
awk '
{ process[$1]+=$2; }
END{ 
 for(i in process)
 {
 printf("%-20s %s\n",i, process[i]) ;
 }

 }' | sort -nrk 2 | head

#Remove the temporary log file
rm /tmp/cpu_usage.$$

```

输出类似如下：

```
$ ./pcpu_usage.sh
Watching CPU usage...
CPU eaters :
Xorg        20
firefox-bin   15
bash        3
evince      2
pulseaudio    1.0
pcpu.sh         0.3
wpa_supplicant  0
wnck-applet     0
watchdog/0      0
usb-storage     0

```

# 它是如何工作的...

CPU 使用数据是由第一个循环生成的，该循环运行一小时（3600 秒）。每分钟，`ps -eocomm,pcpu`命令会生成该时刻的系统活动报告。`-e`选项指定收集所有进程的数据，而不仅仅是当前会话的任务。`-o`选项指定输出格式。`comm`和`pcpu`字段分别指定报告命令名和 CPU 使用百分比。该`ps`命令为每个正在运行的进程生成一行，显示命令名称和当前的 CPU 使用百分比。这些行通过`grep`过滤，去除没有 CPU 使用的行（%CPU 为 0.0）和`COMMAND %CPU`表头。感兴趣的行将附加到临时文件中。

临时文件命名为`/tmp/cpu_usage.$$`。其中，`$$`是一个脚本变量，表示当前脚本的进程 ID（PID）。例如，如果脚本的 PID 为`1345`，则临时文件名为`/tmp/cpu_usage.1345`。

统计文件将在一小时后准备好，并包含 60 组条目，分别对应每分钟的系统状态。`awk`脚本将每个进程的总 CPU 使用率加总到一个名为 process 的关联数组中。该数组以进程名称作为数组索引。最后，`awk`按照总 CPU 使用率对结果进行数字反向排序，并使用 head 命令限制报告为前 10 个使用量条目。

# 另见

+   第四章《文本处理与驾驶》中的*使用 awk 进行高级文本处理*方法解释了`awk`命令

+   第三章《文件输入，文件输出》中的*使用 head 和 tail 打印最后或最前 10 行*方法解释了`tail`命令

# 使用`watch`监控命令输出

`watch`命令会定期执行命令并显示该命令的输出。你可以使用终端会话和在第十章《系统管理命令》中描述的`screen`命令，创建一个自定义仪表盘，通过`watch`监控系统。

# 如何实现...

`watch`命令定期监视终端上命令的输出。`watch`命令的语法如下：

```
    $ watch COMMAND

```

考虑这个例子：

```
    $ watch ls

```

或者，它可以这样使用：

```
    $ watch 'df /home'

```

请考虑以下示例：

```
    # list only directories
 $ watch 'ls -l | grep "^d"'

```

该命令将在默认的两秒间隔内更新输出。

`-n SECONDS` 选项定义了更新输出的时间间隔：

```
    # Monitor the output of ls -l every of 5 seconds
 $ watch -n 5 'ls -l'

```

# 还有更多

`watch` 命令可以与任何生成输出的命令一起使用。有些命令会频繁更改其输出，而这些变化比整个输出更为重要。`watch` 命令会突出显示连续执行之间的差异。请注意，这种高亮显示仅持续到下一次更新。

# 在 `watch` 输出中突出显示差异

`-d` 选项用于突出显示被监视命令连续执行之间的差异：

```
    $ watch -d 'COMMANDS'

 # Highlight new network connections for 30 seconds
 $ watch -n 30 -d 'ss | grep ESTAB'

```

# 记录对文件和目录的访问

有很多原因可能需要在文件被访问时得到通知。你可能想知道何时修改了文件，以便进行备份，或者你可能想知道 `/bin` 目录中的文件是否被黑客修改。

# 准备中

`inotifywait` 命令用于监视文件或目录，并在发生事件时报告。并非每个 Linux 发行版都默认包含它。你需要安装 `inotify-tools` 包。它需要 Linux 内核中的 `inotify` 支持。大多数新的 GNU/Linux 发行版将 `inotify` 支持编译进内核。

# 如何做...

`inotify` 命令可以监视一个目录：

```
    #/bin/bash
 #Filename: watchdir.sh
 #Description: Watch directory access
 path=$1
 #Provide path of directory or file as argument to script

 $ inotifywait -m -r -e create,move,delete $path  -q 

```

示例输出如下所示：

```
    $ ./watchdir.sh .
 ./ CREATE new
 ./ MOVED_FROM new
 ./ MOVED_TO news
 ./ DELETE news

```

# 工作原理...

上述脚本会记录指定路径中的创建、移动和删除事件。`-m` 选项使 `watch` 命令保持活跃，并持续监控更改，而不是在事件发生后退出。`-r` 选项启用递归监视目录（符号链接会被忽略）。`-e` 选项指定要监视的事件列表，`-q` 选项减少冗长信息，仅打印所需的内容。此输出可以重定向到日志文件。

`inotifywait` 可以检查的事件包括以下内容：

| **事件** | **描述** |
| --- | --- |
| `access` | 当文件发生读取操作时 |
| `modify` | 当文件内容被修改时 |
| `attrib` | 当文件元数据发生变化时 |
| `move` | 当文件发生移动操作时 |
| `create` | 当新文件被创建时 |
| `open` | 当文件进行打开操作时 |
| `close` | 当文件进行关闭操作时 |
| `delete` | 当文件被删除时 |

# 使用 `syslog` 记录日志

与守护进程和系统进程相关的日志文件位于 `/var/log` 目录中。这些日志文件使用一种名为 **syslog** 的标准协议，由 `syslogd` 守护进程处理。每个标准应用程序都会使用 `syslogd` 记录信息。本食谱描述了如何使用 `syslogd` 从 shell 脚本中记录信息。

# 准备中

日志文件有助于你推断系统出现问题的原因。记录进度和操作信息是一个很好的实践，使用日志文件消息。`logger` 命令会将数据记录到日志文件中，并使用 `syslogd` 进行管理。

以下是一些标准的 Linux 日志文件。有些发行版使用不同的文件名：

| **日志文件** | **描述** |
| --- | --- |
| `/var/log/boot.log` | 启动日志信息 |
| `/var/log/httpd` | Apache 网络服务器日志 |
| `/var/log/messages` | 启动后内核信息 |
| `/var/log/auth.log``/var/log/secure` | 用户认证日志 |
| `/var/log/dmesg` | 系统启动消息 |
| `/var/log/mail.log``/var/log/maillog` | 邮件服务器日志 |
| `/var/log/Xorg.0.log` | X 服务器日志 |

# 如何操作...

`logger` 命令允许脚本创建和管理日志消息：

1.  将消息放入系统日志文件 `/var/log/messages`：

```
        $ logger LOG_MESSAGE

```

请考虑以下示例：

```
        $ logger This is a test log line

 $ tail -n 1 /var/log/messages
 Sep 29 07:47:44 slynux-laptop slynux: This is a test log line

```

`/var/log/messages` 日志文件是一个通用日志文件。当使用 `logger` 命令时，它默认记录到 `/var/log/messages`。

1.  `-t` 标志定义消息的标签：

```
        $ logger -t TAG This is a message

 $ tail -n 1 /var/log/messages
 Sep 29 07:48:42 slynux-laptop TAG: This is a message

```

`-p` 选项控制 logger 和 `/etc/rsyslog.d` 中的配置文件，决定日志消息的保存位置。

要保存到自定义文件，请按照以下步骤操作：

+   在 `/etc/rsyslog.d` 中创建一个新的配置文件

+   为优先级和日志文件添加模式

+   重启日志守护进程

请考虑以下示例：

```
        # cat /etc/rsyslog.d/myConfig
 local7.* /var/log/local7
 # cd /etc/init.d
 # ./syslogd restart
 # logger -p local7.info A line to be placed in /var/log/local7

```

1.  `-f` 选项将从另一个文件中记录行：

```
        $ logger -f /var/log/source.log

```

# 参见

+   第三章 *文件输入与输出* 中的 *使用 head 和 tail 打印最后或前 10 行* 配方解释了 head 和 tail 命令

# 使用 logrotate 管理日志文件

日志文件跟踪系统上的事件。它们对于调试问题和监控实时机器至关重要。随着时间的推移，日志文件会增长，记录更多事件。由于较旧的数据不如当前数据有用，当日志文件达到大小限制时，它们会被重命名，并删除最旧的文件。

# 准备工作

`logrotate` 命令可以限制日志文件的大小。系统日志工具会将信息附加到日志文件的末尾，而不会删除之前的数据。因此，日志文件会随着时间推移而增大。`logrotate` 命令会扫描配置文件中定义的日志文件。它将保留日志文件的最后 100 千字节（例如，指定 S*IZE = 100 k*），并将其余数据（较旧的日志数据）移动到一个新文件 `logfile_name.1`。当旧数据文件（`logfile_name.1`）超过 `SIZE` 时，`logrotate` 会将该文件重命名为 `logfile_name.2` 并开始一个新的 `logfile_name.1`。`logrotate` 命令还可以将旧日志压缩为 `logfile_name.1.gz`、`logfile_name.2.gz` 等。

# 如何操作...

系统的 `logrotate` 配置文件保存在 `/etc/logrotate.d` 中。大多数 Linux 发行版在该文件夹中都有多个文件。

我们可以为日志文件（例如 `/var/log/program.log`）创建自定义配置：

```
$ cat /etc/logrotate.d/program 
/var/log/program.log { 
missingok 
notifempty 
size 30k 
  compress 
weekly 
  rotate 5 
create 0600 root root 
} 

```

这是一个完整的配置。`/var/log/program.log` 字符串指定了日志文件路径。Logrotate 会将旧日志存档到同一目录中。

# 它是如何工作的...

`logrotate` 命令支持配置文件中的这些选项：

| **参数** | **描述** |
| --- | --- |
| `missingok` | 如果日志文件丢失，则忽略并返回，不进行日志轮换。 |
| `notifempty` | 仅在源日志文件不为空时才进行轮换。 |
| `size 30k` | 这限制了要进行轮换的日志文件的大小。可以设置为 1M，即 1MB。 |
| `compress` | 启用 gzip 压缩以压缩旧日志。 |
| `weekly` | 这指定了轮换执行的间隔。可以是每周、每年或每天。 |
| `rotate 5` | 这是要保留的日志文件存档的旧副本数量。指定为 5 时，将保留`program.log.1.gz`、`program.log.2.gz`，依此类推，直到`program.log.5.gz`。 |
| `create 0600 root root` | 这指定了日志文件存档的权限、用户和组。 |

表中的选项是可以指定的示例。更多选项可以在`logrotate`配置文件中定义。有关更多信息，请参考`http://linux.die.net/man/8/logrotate`的手册页。

# 监控用户登录以寻找入侵者

日志文件可以用于收集系统状态和系统攻击的详细信息。

假设我们有一个连接到互联网的系统，并且启用了 SSH。许多攻击者正在尝试登录系统。我们需要设计一个入侵检测系统，以识别那些登录失败的用户。这些失败的尝试可能是黑客使用字典攻击进行的。脚本应生成一个报告，包含以下详细信息：

+   登录失败的用户

+   尝试次数

+   攻击者的 IP 地址

+   IP 地址的主机映射

+   登录尝试发生的时间

# 正在准备

一个 Shell 脚本可以扫描日志文件并收集所需的信息。登录详细信息记录在`/var/log/auth.log`或`/var/log/secure`中。脚本扫描日志文件以查找失败的登录尝试并分析数据。它使用`host`命令根据 IP 地址映射主机。

# 如何操作...

入侵检测脚本类似于如下：

```
#!/bin/bash
#Filename: intruder_detect.sh
#Description: Intruder reporting tool with auth.log input
AUTHLOG=/var/log/auth.log

if [[ -n $1 ]];
then
 AUTHLOG=$1
 echo Using Log file : $AUTHLOG
fi

# Collect the failed login attempts
LOG=/tmp/failed.$$.log
grep "Failed pass" $AUTHLOG > $LOG

# extract the users who failed
users=$(cat $LOG | awk '{ print $(NF-5) }' | sort | uniq)

# extract the IP Addresses of failed attempts
ip_list="$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" $LOG | sort | uniq)"

printf "%-10s|%-3s|%-16s|%-33s|%s\n" "User" "Attempts" "IP address" \       
    "Host" "Time range"

# Loop through IPs and Users who failed.

for ip in $ip_list;
do
 for user in $users;
 do
 # Count attempts by this user from this IP

 attempts=`grep $ip $LOG | grep " $user " | wc -l`

 if [ $attempts -ne 0 ] 
 then
 first_time=`grep $ip $LOG | grep " $user " | head -1 | cut -c-16`
 time="$first_time"
 if [ $attempts -gt 1 ] 
 then
 last_time=`grep $ip $LOG | grep " $user " | tail -1 | cut -c-16`
 time="$first_time -> $last_time"
 fi
 HOST=$(host $ip 8.8.8.8 | tail -1 | awk '{ print $NF }' )
 printf "%-10s|%-3s|%-16s|%-33s|%-s\n" "$user" "$attempts" "$ip"\     
          "$HOST" "$time";
 fi
 done
done

rm $LOG

```

输出类似如下：

```
Using Log file : secure
User |Attempts|IP address|Host        |Time range
pi   |1  |10.251.90.93   |3(NXDOMAIN) |Jan  2 03:50:24 
root |1  |10.56.180.82   |2(SERVFAIL) |Dec 26 04:31:29 
root |6  |10.80.142.25   |example.com |Dec 19 07:46:49  -> Dec 19 07:47:38 

```

# 它是如何工作的...

`intruder_detect.sh`脚本默认使用`/var/log/auth.log`作为输入日志。或者，我们可以通过命令行参数提供日志文件。失败的登录记录会被收集到临时文件中，以减少处理。

当登录尝试失败时，SSH 日志行类似如下：

```
    sshd[21197]: Failed password for bob1 from 10.83.248.32 port 50035 

```

脚本`greps`用于查找`Failed passw`字符串，并将这些行放入`/tmp/failed.$$.log`文件中。

下一步是提取未能登录的用户。`awk`命令从末尾提取第五个字段（用户名），然后将其传递给`sort`和`uniq`命令，创建用户列表。

接下来，使用正则表达式和`egrep`命令提取唯一的 IP 地址。

嵌套的 for 循环遍历 IP 地址和用户，提取每个 IP 地址和用户组合的行。如果该 IP/用户组合的尝试次数大于 0，则使用`grep`、head 和 cut 提取第一次出现的时间。如果尝试次数大于 1，则使用 tail 而不是 head 提取最后一次的时间。

然后使用格式化的`printf`命令报告此登录尝试。

最后，临时文件被删除。

# 监控远程磁盘使用健康状况

磁盘会被填满，有时也会损坏。即使是 RAID 存储系统，如果在其他驱动器故障之前不更换有问题的驱动器，也可能会失败。监控存储系统的健康状况是管理员工作的一部分。

当一个自动化脚本检查网络上的设备并生成一行报告时，工作变得更加轻松，报告内容包括日期、机器的 IP 地址、设备、设备容量、已用空间、剩余空间、使用百分比和警告状态。如果磁盘使用率低于 80％，驱动器状态将报告为`SAFE`。如果驱动器快满并且需要注意，则状态将报告为`ALERT`。

# 准备工作

脚本使用 SSH 登录远程系统，收集磁盘使用统计数据，并将其写入中央机器的日志文件中。此脚本可以调度在特定时间运行。

脚本需要在远程机器上拥有一个普通用户账户，以便`disklog`脚本可以登录收集数据。我们应该为普通用户配置 SSH 自动登录（第八章《老男孩网络》中介绍了*无密码 SSH 自动登录*的操作）。

# 如何操作...

以下是代码：

```
#!/bin/bash 
#Filename: disklog.sh 
#Description: Monitor disk usage health for remote systems 

logfile="diskusage.log" 

if [[ -n $1 ]] 
then 
  logfile=$1 
fi 

   # Use the environment variable or modify this to a hardcoded value 
user=$USER 

#provide the list of remote machine IP addresses  
IP_LIST="127.0.0.1 0.0.0.0" 
# Or collect them at runtime with nmap 
# IP_LIST=`nmap -sn 192.168.1.2-255 | grep scan | grep cut -c22-` 

if [ ! -e $logfile ] 
then 
  printf "%-8s %-14s %-9s %-8s %-6s %-6s %-6s %s\n" \
    "Date" "IP address" "Device" "Capacity" "Used" "Free" \
    "Percent" "Status" > $logfile
fi 
 ( 
for ip in $IP_LIST; 
do 
 ssh $user@$ip 'df -H' | grep ^/dev/ > /tmp/$$.df 

 while read line; 
 do 
 cur_date=$(date +%D) 
 printf "%-8s %-14s " $cur_date $ip 
 echo $line | \
     awk '{ printf("%-9s %-8s %-6s %-6s %-8s",$1,$2,$3,$4,$5); }' 

 pusg=$(echo $line | egrep -o "[0-9]+%") 
 pusg=${pusg/\%/}; 
 if [ $pusg -lt 80 ]; 
 then 
 echo SAFE 
 else 
 echo ALERT 
 fi 

 done< /tmp/$$.df 
done 

) >> $logfile

```

`cron`工具将定期调度脚本运行。例如，要每天上午 10 点运行该脚本，可以在`crontab`中写入以下条目：

```
00 10 * * * /home/path/disklog.sh /home/user/diskusg.log 

```

运行`crontab -e`命令并添加上述行。

你可以通过以下方式手动运行脚本：

```
$ ./disklog.sh

```

之前脚本的输出类似如下：

```
01/18/17 192.168.1.6    /dev/sda1   106G    53G    49G     52%     SAFE
01/18/17 192.168.1.6    /dev/md1    958G  776G   159G     84%     ALERT

```

# 它是如何工作的...

`disklog.sh`脚本接受日志文件路径作为命令行参数，或者使用默认日志文件。`-e $logfile`检查文件是否存在。如果日志文件不存在，则使用列标题进行初始化。远程机器的 IP 地址列表可以在`IP_LIST`中硬编码，以空格分隔，或者可以使用`nmap`命令扫描网络以查找可用节点。如果使用`nmap`调用，请根据您的网络调整 IP 地址范围。

一个 for 循环遍历每个 IP 地址。`ssh` 应用程序向每个节点发送 `df -H` 命令以获取磁盘使用情况信息。`df` 输出存储在临时文件中。一个 `while` 循环逐行读取该文件，并调用 `awk` 提取相关数据并输出。一个 `egrep` 命令提取百分比值并去除 `%`。如果该值小于 80，则该行标记为 `SAFE`，否则标记为 `ALERT`。整个输出字符串必须重定向到 `log` 文件。因此，`for` 循环被包含在子 shell `()` 中，标准输出被重定向到日志文件。

# 参见

+   第十章《调度与 cron》中的*调度与 cron* 介绍了 `crontab` 命令。

# 确定系统上活跃用户的小时数

这个配方利用系统日志来查明每个用户在服务器上花费了多少小时，并根据总使用时长对他们进行排名。报告包括排名、用户、首次登录日期、最后登录日期、登录次数和总使用时长等详细信息。

# 准备工作

关于用户会话的原始数据以二进制格式存储在 `/var/log/wtmp` 文件中。`last` 命令返回有关登录会话的详细信息。每个用户的会话小时数之和即为该用户的总使用时长。

# 如何操作...

该脚本将确定活跃用户并生成报告：

```
#!/bin/bash 
#Filename: active_users.sh 
#Description: Reporting tool to find out active users 

log=/var/log/wtmp 

if [[ -n $1 ]]; 
then 
  log=$1 
fi 

printf "%-4s %-10s %-10s %-6s %-8s\n" "Rank" "User" "Start" \ 
 "Logins" "Usage hours" 

last -f $log | head -n -2   > /tmp/ulog.$$ 

cat /tmp/ulog.$$ |  cut -d' ' -f1 | sort | uniq> /tmp/users.$$ 

( 
while read user; 
do 
  grep ^$user /tmp/ulog.$$ > /tmp/user.$$ 
  minutes=0 

  while read t  
  do 
    s=$(echo $t | awk -F: '{ print ($1 * 60) + $2 }') 
    let minutes=minutes+s 
  done< <(cat /tmp/user.$$ | awk '{ print $NF }' | tr -d ')(') 

  firstlog=$(tail -n 1 /tmp/user.$$ | awk '{ print $5,$6 }') 
  nlogins=$(cat /tmp/user.$$ | wc -l)  
  hours=$(echo "$minutes / 60.0" | bc) 

  printf "%-10s %-10s %-6s %-8s\n"  $user "$firstlog" $nlogins $hours 
done< /tmp/users.$$  

) | sort -nrk 4 | awk '{ printf("%-4s %s\n", NR, $0) }'  
rm /tmp/users.$$ /tmp/user.$$ /tmp/ulog.$$ 

```

输出类似于以下内容：

```
$ ./active_users.sh
Rank User       Start      Logins Usage hours
1    easyibaa   Dec 11     531    349
2    demoproj   Dec 10     350    230
3    kjayaram   Dec 9      213    55
4    cinenews   Dec 11     85     139
5    thebenga   Dec 10     54     35
6    gateway2   Dec 11     52     34
7    soft132    Dec 12     49     25
8    sarathla   Nov 1      45     29
9    gtsminis   Dec 11     41     26
10   agentcde   Dec 13     39     32

```

# 它是如何工作的...

`active_users.sh` 脚本从 `/var/log/wtmp` 或命令行中定义的 `wtmp` 日志文件中读取数据。`last -f` 命令提取日志文件内容。日志文件中的第一列是用户名。`cut` 命令提取日志文件中的第一列。`sort` 和 `uniq` 命令将其减少为唯一用户的列表。

脚本的外部循环遍历所有用户。对于每个用户，使用 `grep` 提取对应用户的日志行。

每行的最后一列是该次登录会话的持续时间。这些值在内部的 `while read t` 循环中进行求和。

会话持续时间的格式为 `(HOUR:SEC)`。该值通过 `awk` 提取最后一个字段，然后通过 `tr -d` 去除括号。第二个 `awk` 命令将 *HH::MM* 字符串转换为分钟数，然后将分钟数相加。当循环完成时，使用 `$minutes` 除以 60 将总分钟数转换为小时。

用户的首次登录时间是用户数据临时文件中的最后一行。这通过 `tail` 和 `awk` 提取。登录会话的数量是该文件中的行数，通过 `wc` 计算。

用户按总使用时长排序，使用 `sort` 的 `-nr` 选项进行数字降序排序，`-k4` 用于指定排序的列（使用时长）。最后，排序后的输出传递给 `awk`，它会在每行前加上行号，表示每个用户的排名。

# 测量和优化功耗

电池容量是移动设备（如笔记本电脑和平板电脑）上的一个关键资源。Linux 提供了测量功耗的工具，其中一个命令是`powertop`。

# 准备就绪

`powertop`应用程序在许多 Linux 发行版中没有预安装，你需要使用你的包管理器安装它。

# 如何操作...

`powertop`应用程序测量每个模块的功耗，并支持交互式优化功耗：

如果没有选项，`powertop`将在终端上显示一个界面：

```
# powertop

```

`powertop`命令进行测量，并显示有关功耗、最耗电的进程等详细信息：

```
PowerTOP 2.3  Overview  Idle stats  Frequency stats  Device stats  Tunable

Summary: 1146.1 wakeups/sec,  0.0 GPU ops/secs, 0.0 VFS ops/sec and 73.0% C  Usage Events/s Category Description
407.4 ms/s 258.7 Process /usr/lib/vmware/bin/vmware
64.8 ms/s 313.8 Process /usr/lib64/firefox/firefox

```

`-html`标签将使`powertop`在一段时间内进行测量，并生成一个默认文件名为`PowerTOP.html`的 HTML 报告，你可以使用任何网页浏览器打开：

```
    # powertop --html

```

在交互模式下，你可以优化功耗。当`powertop`运行时，使用箭头键或 Tab 键切换到 Tunables 标签页；这将显示`powertop`可以调整的属性列表，以减少功耗。选择你想要的选项，按回车键在“Bad”和“Good”之间切换。

如果你想监控便携设备电池的电力消耗，需要拔掉充电器并使用电池，以便`powertop`进行测量。

# 监控磁盘活动

监控工具的常见命名约定是以`'top'`结尾（用于监控进程的命令）。用于监控磁盘 I/O 的工具叫做`iotop`。

# 准备就绪

**iotop**应用程序在大多数 Linux 发行版中没有预安装，你需要使用你的包管理器安装它。`iotop`应用程序需要 root 权限，所以你需要以`sudo`或 root 用户身份运行它。

# 如何操作...

`iotop`应用程序可以执行连续监控或在固定时间段内生成报告：

1.  对于连续监控，请使用以下命令：

```
        # iotop -o

```

`-o`选项告诉`iotop`只显示那些在运行时进行活跃 I/O 的进程，从而减少输出中的噪声。

1.  `-n`选项告诉 iotop 运行*N*次后退出：

```
        # iotop -b -n 2

```

1.  `-p`选项用于监控特定进程：

```
       # iotop -p PID

```

`PID`是你希望监控的进程。

在大多数现代发行版中，你可以使用`pidof`命令，而不是查找 PID 并将其提供给`iotop`，可以将前面的命令写成：# iotop -p `pidof cp`

# 检查磁盘和文件系统的错误

Linux 文件系统非常强大。尽管如此，文件系统可能会损坏并导致数据丢失。尽早发现问题，能减少数据丢失和损坏的风险。

# 准备就绪

检查文件系统的标准工具是`fsck`。该命令在所有现代发行版中都已安装。请注意，你需要以 root 用户身份或通过`sudo`运行`fsck`。

# 如何操作...

如果文件系统长时间未检查，或者有原因（如电力中断后不安全的重启）怀疑它已经损坏，Linux 将在启动时自动运行`fsck`。你也可以手动运行`fsck`。

1.  要检查分区或文件系统中的错误，传递路径给`fsck`：

```
 # fsck /dev/sdb3
 fsck from util-linux 2.20.1
 e2fsck 1.42.5 (29-Jul-2012)
 HDD2 has been mounted 26 times without being checked, check forced.
 Pass 1: Checking inodes, blocks, and sizes
 Pass 2: Checking directory structure
 Pass 3: Checking directory connectivity
 Pass 4: Checking reference counts
 Pass 5: Checking group summary information
 HDD2: 75540/16138240 files (0.7% non-contiguous),      
        48756390/64529088 blocks

```

1.  `-A`标志检查`/etc/fstab`中配置的所有文件系统：

```
        # fsck -A

```

这将遍历`/etc/fstab`文件，检查每个文件系统。`fstab`文件定义了物理磁盘分区与挂载点之间的映射。它在启动时用于挂载文件系统。

1.  `-a`选项指示`fsck`自动尝试修复错误，而不是交互式地询问我们是否修复它们。使用此选项时要小心：

```
        # fsck -a /dev/sda2

```

1.  `-N`选项模拟`fsck`将执行的操作：

```
 # fsck -AN
 fsck from util-linux 2.20.1
 [/sbin/fsck.ext4 (1) -- /] fsck.ext4 /dev/sda8
 [/sbin/fsck.ext4 (1) -- /home] fsck.ext4 /dev/sda7
 [/sbin/fsck.ext3 (1) -- /media/Data] fsck.ext3 /dev/sda6

```

# 它是如何工作的...

`fsck`应用程序是文件系统特定`fsck`应用程序的前端。当我们运行`fsck`时，它会检测文件系统的类型并运行相应的`fsck.fstype`命令，其中`fstype`是文件系统的类型。例如，如果我们在`ext4`文件系统上运行`fsck`，它将调用`fsck.ext4`命令。

因此，`fsck`仅支持所有文件系统特定工具中共有的选项。要查找更详细的选项，请阅读特定应用程序的手册页，如`fsck.ext4`。

虽然非常罕见，但`fsck`可能会丢失数据或使已经严重损坏的文件系统更糟。如果你怀疑文件系统严重损坏，应该使用`-N`选项列出`fsck`将执行的操作，而不实际执行它们。如果`fsck`报告它可以修复的超过十个问题，或者这些问题包括损坏的目录结构，你可能想要以只读模式挂载驱动器，并在运行`fsck`之前尽量提取关键数据。

# 检查磁盘健康

现代磁盘驱动器运行多年没有问题，但当磁盘故障时，它是一次重大灾难。现代磁盘驱动器配备了**自监控、分析和报告技术**（**SMART**）设施，以监控磁盘健康状态，这样你可以在发生重大故障之前更换即将故障的驱动器。

# 准备工作

Linux 通过`smartmontools`软件包支持与磁盘的 SMART 工具交互。大多数发行版默认安装此软件包。如果它未安装，你可以通过包管理器安装：

```
    apt-get install smartmontools

```

另外，可以使用以下命令：

```
    yum install smartmontools

```

# 如何操作...

`smartmontools`的用户界面是`smartctl`应用程序。此应用程序启动磁盘驱动器的测试并报告 SMART 设备的状态。

由于`smartctl`应用程序访问原始磁盘设备，因此你必须具有根权限才能运行它。

`-a`选项报告设备的完整状态：

```
    $ smartctl -a /dev/sda

```

输出将包括基本信息的标题、一组原始数据值和测试结果。标题包括关于被测试驱动器的详细信息以及此报告的日期戳：

```
 smartctl 5.43 2012-06-30 r3573 [x86_64-linux-2.6.32-  
    642.11.1.el6.x86_64] (local build)
 Copyright (C) 2002-12 by Bruce Allen,    
    http://smartmontools.sourceforge.net

 === START OF INFORMATION SECTION ===
 Device Model:     WDC WD10EZEX-00BN5A0
 Serial Number:    WD-WCC3F1HHJ4T8
 LU WWN Device Id: 5 0014ee 20c75fb3b
 Firmware Version: 01.01A01
 User Capacity:    1,000,204,886,016 bytes [1.00 TB]
 Sector Sizes:     512 bytes logical, 4096 bytes physical
 Device is:        Not in smartctl database [for details use: -P    
    showall]
 ATA Version is:   8
 ATA Standard is:  ACS-2 (unknown minor revision code: 0x001f)
 Local Time is:    Mon Jan 23 11:26:57 2017 EST
 SMART support is: Available - device has SMART capability.
 SMART support is: Enabled
 ...

```

原始数据值包括错误计数、启动时间、开机小时数等。最后两列（`WHEN_FAILED` 和 `RAW_VALUE`）尤其值得关注。在以下示例中，设备已开机 9823 小时。它已经开关机了 11 次（服务器不常进行断电重启），当前温度为 30°C。当开机时间接近制造商的**平均故障间隔时间**（**MTBF**）时，就该考虑更换硬盘或将其移至不太关键的系统。如果重启之间的电源循环次数增加，可能表示电源故障或电缆有问题。如果温度过高，应考虑检查硬盘的外壳，可能是风扇故障或过滤器堵塞：

```
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  
    WHEN_FAILED RAW_VALUE

 9 Power_On_Hours          0x0032   087   087   000    Old_age   Always       
     -       9823

12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always
     -       11

194 Temperature_Celsius     0x0022   113   109   000    Old_age   Always
     -       30

```

输出的最后一部分将是测试结果：

```
SMART Error Log Version: 1
No Errors Logged

SMART Self-test log structure revision number 1

Num  Test_Description    Status                  Remaining  LifeTime(hours)  
        LBA_of_first_error
# 1  Extended offline    Completed without error       00%      9825   
        - 

```

`-t` 标志强制 SMART 设备运行自检。这些测试是非破坏性的，可以在硬盘运行时进行。SMART 设备可以运行长时间或短时间的测试。短时间测试需要几分钟，而长时间测试在大容量设备上可能需要一小时或更长时间：

```
$ smartctl -t [long][short] DEVICE

$ smartctl -t long /dev/sda

smartctl 5.43 2012-06-30 r3573 [x86_64-linux-2.6.32-642.11.1.el6.x86_64] (local build)
Copyright (C) 2002-12 by Bruce Allen, http://smartmontools.sourceforge.net

=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
Sending command: "Execute SMART Extended self-test routine immediately in off-line mode".
Drive command "Execute SMART Extended self-test routine immediately in off-line mode" successful.
Testing has begun.
Please wait 124 minutes for test to complete.
Test will complete after Mon Jan 23 13:31:23 2017

Use smartctl -X to abort test.

```

在两个多小时后，此测试将完成，并可以通过 `smartctl -a` 命令查看结果。

# 工作原理

现代硬盘远不止一个旋转的金属盘。它们包括 CPU、ROM、内存和定制的信号处理芯片。`smartctl` 命令通过与硬盘 CPU 上运行的小型操作系统交互，来请求测试并报告结果。

# 获取磁盘统计信息

`smartctl` 命令提供了许多磁盘统计信息，并测试硬盘。`hdparm` 命令提供更多统计信息，并检查磁盘在系统中的表现，可能会受到控制器芯片、电缆等的影响。

# 准备就绪

`hdparm` 命令在大多数 Linux 发行版中都是标准命令。使用它需要 root 权限。

# 如何操作...

`-I` 选项将提供有关设备的基本信息：

```
    $ hdparm -I DEVICE
 $ hdparm -I /dev/sda

```

以下示例输出展示了报告的部分数据。型号和固件与 `smartctl` 报告的一致。配置包括可以在分区和创建文件系统之前调节的参数：

```
/dev/sda:

ATA device, with non-removable media
 Model Number:       WDC WD10EZEX-00BN5A0 
 Serial Number:      WD-WCC3F1HHJ4T8
 Firmware Revision:  01.01A01
 Transport:          Serial, SATA 1.0a, SATA II Extensions, SATA Rev 2.5, SATA Rev 2.6, SATA Rev 3.0
Standards:
 Used: unknown (minor revision code 0x001f) 
 Supported: 9 8 7 6 5 
 Likely used: 9
Configuration:
 Logical  max current
 cylinders 16383 16383
 heads  16 16
 sectors/track 63 63
 --
 CHS current addressable sectors:   16514064
 LBA    user addressable sectors:  268435455
 LBA48  user addressable sectors: 1953525168
 Logical  Sector size:                   512 bytes
 Physical Sector size:                  4096 bytes
 device size with M = 1024*1024:      953869 MBytes
 device size with M = 1000*1000:     1000204 MBytes (1000 GB)
 cache/buffer size  = unknown
 Nominal Media Rotation Rate: 7200

...
Security: 
 Master password revision code = 65534
 supported
 not enabled
 not locked
 not frozen
 not expired: security count
 supported: enhanced erase
 128min for SECURITY ERASE UNIT. 128min for ENHANCED SECURITY ERASE UNIT. 
Logical Unit WWN Device Identifier: 50014ee20c75fb3b
 NAA  : 5
 IEEE OUI : 0014ee
 Unique ID : 20c75fb3b
Checksum: correct

```

# 工作原理

`hdparm` 命令是进入内核库和模块的用户界面。它支持修改参数和报告参数。修改这些参数时必须小心谨慎！

# 还有更多

`hdparm` 命令可以测试磁盘性能。`-t` 和 `-T` 选项分别对缓冲和缓存读取进行时间测试：

```
# hdparm -t /dev/sda
Timing buffered disk reads: 486 MB in  3.00 seconds = 161.86 MB/sec

# hdparm -T /dev/sda
Timing cached reads:   26492 MB in  1.99 seconds = 13309.38 MB/sec

```
