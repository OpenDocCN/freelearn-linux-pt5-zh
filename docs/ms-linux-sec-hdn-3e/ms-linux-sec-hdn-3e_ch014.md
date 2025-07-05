# 13 日志记录与日志安全

## 加入我们的书籍社区，参与 Discord 讨论

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/file82.png)

系统日志是每个 IT 管理员生活中的重要部分。它们可以告诉你系统的性能如何，如何排除故障，以及用户（无论是授权用户还是未授权用户）在系统上做了什么。

在本章中，我将带你简要了解 Linux 日志系统，然后展示一个很酷的技巧，帮助你更轻松地审查日志。接下来，我将向你展示如何设置一个远程日志服务器，并为客户端提供 **传输层安全性** (**TLS**)-加密连接。

我们将要讨论的主题包括：

+   理解 Linux 系统日志文件

+   理解 `rsyslog`

+   理解 `journald`

+   使用 Logwatch 让事情变得更简单

+   设置远程日志服务器

本章的重点是讨论那些已经内置在你的 Linux 发行版中或可以从发行版仓库中获得的日志工具。其他 Packt Publishing 书籍，例如 Adam K. Dean 的 *Linux 管理食谱*，会向你展示一些更高级、更复杂的第三方日志聚合和分析工具。

所以，如果你已经准备好并充满干劲，让我们来看看那些 Linux 日志文件。

## 理解 Linux 系统日志文件

你可以在 `/var/log/` 目录中找到 Linux 日志文件。Linux 日志文件的结构在所有 Linux 发行版中基本相同。但为了保持 Linux 传统的困惑，各个发行版的主要日志文件名称不同。在 Red Hat 系统中，主要日志文件是 `messages` 文件，认证相关事件的日志是 `secure` 文件。在 Debian/Ubuntu 系统中，主要日志文件是 `syslog` 文件，认证日志是 `auth.log` 文件。你还会看到其他一些日志文件，包括：

+   `/var/log/kern.log`：在 Debian/Ubuntu 系统中，这个日志包含有关 Linux 内核运行情况的消息。正如我们在 *第四章：用防火墙保护你的服务器 - 第一部分* 和 *第五章：用防火墙保护你的服务器 - 第二部分* 中所看到的，这包括有关 Linux 防火墙运行情况的消息。所以，如果你想查看是否有任何可疑的网络数据包被阻止，这就是你需要查看的地方。Red Hat 系统没有这个文件。相反，Red Hat 系统会将其内核消息发送到 `messages` 文件中。

+   `/var/log/wtmp` 和 `/var/run/utmp`：这两个文件本质上执行相同的功能。它们都记录登录到系统的用户信息。主要区别在于，`wtmp` 保存来自 `utmp` 的历史数据。与大多数 Linux 日志文件不同，这些文件采用二进制格式，而非普通的文本模式格式。`utmp` 文件是我们将要查看的唯一不在 `/var/log/` 目录下的文件。

+   `/var/log/btmp`：这个二进制文件包含失败登录尝试的信息。我们在*第三章 安全保护普通用户账户*中看过的`pam_tally2`模块使用了该文件中的信息。

+   `/var/log/lastlog`：这个二进制文件包含用户上次登录系统的时间信息。

+   `/var/log/audit/audit.log`：这个文本模式文件记录了来自 auditd 守护进程的信息。我们在*第十二章 扫描、加固与审计*中已经讨论过它，所以在这里就不再详细讲解。

还有许多其他日志文件包含关于应用程序和系统启动的信息。但我在这里列出的日志文件是我们在查看系统安全时最关心的主要文件。

现在我们已经看过了有哪些日志文件，让我们更详细地了解它们。

### 系统日志和认证日志

无论你是在讨论 Debian/Ubuntu 上的`syslog`和`auth.log`文件，还是在 RHEL/CentOS/AlmaLinux 上的`messages`和`secure`文件，这些文件在不同系统中是相同的，只是名字不同。系统日志文件和认证日志文件有相同的基本结构，都是纯文本文件。这使得我们可以使用 Linux 内置的工具方便地搜索特定信息。其实我们使用哪个虚拟机并不重要，主要是确保文件名的一致性。

首先，让我们来看一下系统日志中的一条简单信息：

```
Jul  1 18:16:12 localhost systemd[1]: Started Postfix Mail Transport Agent.
```

下面是具体内容：

+   `Jul 1 18:16:12`：这是消息生成的日期和时间。

+   `localhost`：这是生成信息的机器的主机名。这很重要，因为一台 Linux 机器可以作为其他 Linux 机器的中央日志存储库。默认情况下，其他机器的消息会被直接丢到本地机器使用的同一个日志文件中。因此，我们需要这个字段来了解每台机器上发生了什么。

+   `systemd[1]`：这是生成该消息的服务。在这个例子中，是`systemd`守护进程。

+   这一行的其余部分是具体的消息内容。

提取文本模式日志文件信息有多种方式。目前，我们仅仅在`less`中打开文件，如这个示例所示：

```
sudo less syslog
```

然后，要搜索特定的文本字符串，按下**/**键，输入你想查找的字符串，然后按回车。

那么，我们可以在这些文件中找到哪些与安全相关的信息呢？首先，让我们看一下服务器私钥的权限：

```
donnie@orangepione:/etc/ssh$ ls -l
total 580
. . .
-rw-------+ 1 root root   1679 Feb 10  2019 ssh_host_rsa_key
-rw-r--r--  1 root root    398 Feb 10  2019 ssh_host_rsa_key.pub
donnie@orangepione:/etc/ssh$
```

这个私钥，`ssh_host_rsa_key`文件，必须仅对 root 用户设置权限。但是，权限设置末尾的`+`符号表示该文件上设置了**访问控制列表**（**ACL**）。`getfacl`命令会显示具体的情况：

```
donnie@orangepione:/etc/ssh$ getfacl ssh_host_rsa_key
# file: ssh_host_rsa_key
# owner: root
# group: root
user::rw-
user:sshdnoroot:r--
group::---
mask::r--
other::---
donnie@orangepione:/etc/ssh$
```

所以，有人创建了 `sshdnoroot` 用户，并为其分配了对服务器私有 SSH 密钥的读取权限。现在，如果我尝试重启 OpenSSH 守护进程，它将失败。查看系统日志——在此案例中是 `syslog` 文件——将告诉我原因：

```
Mar 13 12:47:46 localhost sshd[1952]: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Mar 13 12:47:46 localhost sshd[1952]: @ WARNING: UNPROTECTED PRIVATE KEY FILE! @
Mar 13 12:47:46 localhost sshd[1952]: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Mar 13 12:47:46 localhost sshd[1952]: Permissions 0640 for '/etc/ssh/ssh_host_rsa_key' are too open.
Mar 13 12:47:46 localhost sshd[1952]: It is required that your private key files are NOT accessible by others.
Mar 13 12:47:46 localhost sshd[1952]: This private key will be ignored.
Mar 13 12:47:46 localhost sshd[1952]: key_load_private: bad permissions
Mar 13 12:47:46 localhost sshd[1952]: Could not load host key: /etc/ssh/ssh_host_rsa_key
```

所以，如果除了 root 用户之外的其他人对服务器的私钥有任何访问权限，SSH 守护进程就无法启动。但这是怎么发生的呢？让我们在认证文件中搜索——在这个例子中是 `auth.log`——看看是否能找到线索：

```
Mar 13 12:42:54 localhost sudo:   donnie : TTY=tty1 ; PWD=/etc/ssh ; USER=root ; COMMAND=/usr/bin/setfacl -m u:sshdnoroot:r ssh_host_ecdsa_key ssh_host_ed25519_key ssh_host_rsa_key
```

啊，原来是那个 `donnie` 做的这件事。什么？这太离谱了！立刻解雇那家伙！等等，那是我自己。再想想，还是不要解雇他了。不过说真的，这显示了强制用户使用 `sudo` 而不是允许他们从 root shell 做任何事的价值。如果我从 root shell 执行这些操作，认证日志只会显示我作为 root 用户登录的地方，但不会显示我以 root 用户身份做了什么。而使用 `sudo`，每个 root 权限的操作都会被记录下来，且会显示执行操作的人。

有几种方法可以从日志文件中获取特定信息，包括：

+   使用前面提到的 `less` 工具的搜索功能

+   使用 `grep` 来搜索一个或多个文件中的文本字符串

+   用诸如 `bash`、Python 或 `awk` 等语言编写脚本

这是使用 `grep` 的一个示例：

```
sudo grep 'fail' syslog
```

在这种情况下，我正在搜索 `syslog` 文件中包含文本字符串 `fail` 的所有行。默认情况下，`grep` 是区分大小写的，因此此命令不会找到任何包含大写字母的 `fail`。此外，默认情况下，`grep` 会找到嵌入在其他文本字符串中的文本。因此，除了找到 `fail`，这个命令还会找到 `failed`、`failure` 或任何包含 `fail` 的其他文本字符串。

若要使搜索不区分大小写，请添加 `-i` 选项，如下所示：

```
sudo grep -i 'fail' syslog
```

这将找到所有形式的 `fail`，无论是大写还是小写字母。若只搜索 `fail` 这个文本字符串，并排除它嵌入在其他文本字符串中的情况，请使用 `-w` 选项，如下所示：

```
sudo grep -w 'fail' syslog
```

你可以像这样结合两个选项：

```
sudo grep -iw 'fail' syslog
```

通常，如果你不确定自己在寻找什么，可以先从一个更通用的搜索开始，这可能会显示过多信息。然后逐步缩小范围，直到找到你想要的内容。

现在，当你只想在日志文件中搜索特定信息时，以上方法是有效的。但如果你需要进行日常日志审查，这就有些繁琐了。稍后我将向你展示一款可以简化这项工作的工具。现在，让我们来看看二进制日志文件。

### `utmp`、`wtmp`、`btmp` 和 `lastlog` 文件

与系统日志文件和认证日志文件不同，所有这些文件都是二进制文件。所以，我们不能使用常规的文本工具，如 `less` 或 `grep`，来读取它们或从中提取信息。相反，我们将使用一些特殊的工具来读取这些二进制文件。

`w`和`who`命令从`/var/run/utmp`文件中提取有关已登录用户及其活动的信息。这两个命令都有自己的选项切换，但您可能永远不会需要它们。如果只想查看当前已登录用户的列表，请像这样使用`who`：

```
donnie@orangepione:/var/log$ who
donnie   tty7         2019-08-02 18:18 (:0)
donnie   pts/1        2019-11-21 16:21 (192.168.0.251)
donnie   pts/2        2019-11-21 17:01 (192.168.0.251)
katelyn  pts/3        2019-11-21 18:15 (192.168.0.11)
lionel   pts/4        2019-11-21 18:21 (192.168.0.15)
donnie@orangepione:/var/log$
```

它显示我有三个不同的登录。`tty7`行是我的本地终端会话，`pts/1`和`pts/2`行是我从`192.168.0.251`机器的两个远程 SSH 会话登录的。凯特琳和莱昂纳德从另外两台机器上远程登录。

`w`命令不仅显示谁登录了，还显示他们在做什么：

```
donnie@orangepione:/var/log$ w
 18:29:42 up 2:09, 5 users, load average: 0.00, 0.00, 0.00
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
donnie tty7 :0 02Aug19 111days 6.28s 0.05s /bin/sh /etc/xdg/xfce4/xinitrc -- /etc/X11/xinit/xserverrc
donnie pts/1 192.168.0.251 16:21 4.00s 2.88s 0.05s w
donnie pts/2 192.168.0.251 17:01 7:10 0.81s 0.81s -bash
katelyn pts/3 192.168.0.11 18:15 7:41 0.64s 0.30s vim somefile.txt
lionel pts/4 192.168.0.15 18:21 8:06 0.76s 0.30s sshd: lionel [priv] 
donnie@orangepione:/var/log$
```

这显示了五个用户，但实际上只有三个，因为它将我每个登录会话都视为一个单独的用户。在我的第一个登录的`FROM`列下的`:0`表示这个登录在机器的本地控制台。`/bin/sh`部分显示我有一个终端窗口打开，`/etc/xdg/xfce4/xinitrc -- /etc/X11/xinit/xserverrc`部分表示机器处于图形模式，使用 XFCE 桌面。`pts/1`行显示我在那个窗口中运行了`w`命令，而`pts/2`行显示我在那个窗口中什么都没做，除了打开了 bash shell。

下一步，我们看到凯特琳正在编辑一个文件。所以，我认为她一切都好。但是看看莱昂纳德。他行中的`[priv]`表明他正在执行某种特权操作。要查看这个操作是什么，我们将窥探认证文件，我们在那里看到了这个：

```
Nov 21 18:21:42 localhost sudo:   lionel : TTY=pts/4 ; PWD=/home/lionel ; USER=root ; COMMAND=/usr/sbin/visudo
```

哦，来吧。什么傻瓜给了莱昂纳德使用`visudo`的特权？我是说，我们知道莱昂纳德不应该有那个特权。好吧，我们可以调查一下。在认证文件的更上方，我们看到了这个：

```
Nov 21 18:17:53 localhost sudo:   donnie : TTY=pts/2 ; PWD=/home/donnie ; USER=root ; COMMAND=/usr/sbin/visudo
```

这显示了那个`donnie`角色打开了`visudo`，但没有显示他对其进行了什么编辑。但由于这一行紧随`donnie`创建莱昂纳德账户的行之后，并且没有其他用户使用`visudo`，可以有把握地说`donnie`是给了莱昂纳德那个`visudo`特权的人。因此，我们可以推断，那个`donnie`角色是一个真正该被解雇的失败者。哦，等等。那不是我吗？好吧，不管了。

在正常使用中，`last`命令从`/var/log/wtmp`文件中提取信息，该文件归档了来自`/var/run/utmp`文件的历史数据。如果没有任何选项切换，`last`会显示每个用户的登录或注销时间，以及机器的启动时间：

```
donnie@orangepione:/var/log$ last
lionel   pts/4        192.168.0.15     Thu Nov 21 18:21   still logged in
lionel   pts/4        192.168.0.15     Thu Nov 21 18:17 - 18:17  (00:00)
katelyn  pts/3        192.168.0.11     Thu Nov 21 18:15   still logged in
katelyn  pts/3        192.168.0.251    Thu Nov 21 18:02 - 18:15  (00:12)
donnie   pts/2        192.168.0.251    Thu Nov 21 17:01   still logged in
donnie   pts/1        192.168.0.251    Thu Nov 21 16:21   still logged in
donnie   tty7         :0               Fri Aug  2 18:18    gone - no logout
reboot   system boot  4.19.57-sunxi    Wed Dec 31 19:00   still running
. . .
wtmp begins Wed Dec 31 19:00:03 1969
donnie@orangepione:/var/log$
```

要显示失败的登录尝试列表，请使用`-f`选项读取`/var/log/btmp`文件。关键在于，这需要`sudo`权限，因为我们通常希望保持有关失败登录的信息机密：

```
donnie@orangepione:/var/log$ sudo last -f /var/log/btmp
[sudo] password for donnie: 
katelyn ssh:notty 192.168.0.251 Thu Nov 21 17:57 gone - no logout
katelyn ssh:notty 192.168.0.251 Thu Nov 21 17:57 - 17:57 (00:00)
katelyn ssh:notty 192.168.0.251 Thu Nov 21 17:57 - 17:57 (00:00)
btmp begins Thu Nov 21 17:57:35 2019
donnie@orangepione:/var/log$
```

当然，我们可以查看凯特琳在`auth.log`或`secure`文件中的三次登录失败，但在这里查看它们更方便和更快。

最后是`lastlog`命令，它从—你猜到了—`/var/log/lastlog`文件中提取信息。该命令显示了机器上所有用户（甚至系统用户）的登录记录，以及他们最后一次登录的时间：

```
donnie@orangepione:/var/log$ lastlog
Username         Port     From             Latest
root             tty1                      Tue Mar 12 15:29:09 -0400 2019
. . .
messagebus                                 **Never logged in**
sshd                                       **Never logged in**
donnie           pts/2    192.168.0.251    Thu Nov 21 17:01:03 -0500 2019
sshdnoroot                                 **Never logged in**
. . .
katelyn          pts/3    192.168.0.11     Thu Nov 21 18:15:44 -0500 2019
lionel           pts/4    192.168.0.15     Thu Nov 21 18:21:33 -0500 2019
donnie@orangepione:/var/log$
```

`/var/log/`目录下有很多其他日志，但我这里只给你简要介绍与系统安全相关的日志。接下来，我们将看看大多数 Linux 发行版中内置的两个主要日志系统，从`rsyslog`系统开始。

## 了解 rsyslog

旧的`syslog`日志系统创建于 1980 年代，用于 Unix 及其他类 Unix 系统。直到几年前，它才在 Linux 世界中走向终结。如今，我们使用`rsyslog`，它更加健壮，功能也多了一些。它在基于 Debian/Ubuntu 和基于 Red Hat 的发行版上主要是相同的，只是在配置文件的设置上有所不同。但在看差异之前，让我们先看看相同之处。

### 了解 rsyslog 日志规则

日志规则定义了将每个特定系统服务的消息记录到哪里：

+   在 Red Hat/CentOS/AlmaLinux 系统上，规则存储在`/etc/rsyslog.conf`文件中。只需向下滚动，直到看到`#### RULES ####`部分。

+   在 Debian/Ubuntu 系统上，规则存储在`/etc/rsyslog.d/`目录中的不同文件中。目前我们关心的主要文件是`50-default.conf`，它包含了主要的日志规则。

为了说明`rsyslog`规则的结构，我们来看一个来自 AlmaLinux 机器的例子：

```
authpriv.*           /var/log/secure
```

以下是详细分类：

+   `authpriv`：这是设施，定义了消息的类型。

+   `.`：点号将设施与级别分开，后者是下一个字段。

+   `*`：这是级别，表示消息的重要性。在这种情况下，我们使用通配符，意味着`authpriv`设施的所有级别都会被记录。

+   `/var/log/secure`：这是动作，实际上是该消息的目标位置。（我不知道为什么有人决定称其为动作。）

当我们将这些内容汇总时，可以看到所有级别的`authpriv`消息会被发送到`/var/log/secure`文件中。

以下是预定义的`rsyslog`设施的简要列表：

+   `auth`：由授权系统生成的消息（如`login`、`su`、`sudo`等）

+   `authpriv`：由授权系统生成的消息，但仅供选定用户读取

+   `cron`：由`cron`守护进程生成的消息

+   `daemon`：所有系统守护进程生成的消息（例如，`sshd`、`ftpd`等）

+   `ftp`：`ftp`生成的消息

+   `kern`：由 Linux 内核生成的消息

+   `lpr`：由行打印机排队生成的消息

+   `mail`：由邮件系统生成的消息

+   `mark`：系统日志中的定期时间戳消息

+   `news`：由网络新闻系统生成的消息

+   `rsyslog`：`rsyslog`内部生成的消息

+   `user`：由用户生成的消息

+   `local0-7`：用于编写自定义脚本的自定义消息

以下是不同日志级别的列表：

+   `none`：禁用某个设施的日志记录

+   `debug`：仅调试

+   `info`：信息

+   `notice`：需要检查的问题

+   `warning`：警告消息

+   `err`：错误条件

+   `crit`：严重条件

+   `alert`：紧急消息

+   `emerg`：紧急情况

除了`debug`级别外，你为某个设施设置的任何级别都会导致该级别及以上（直到`emerg`）的消息被记录。例如，当你设置`info`级别时，所有`info`级别及以上的消息都会被记录。考虑到这一点，让我们看一个更复杂的日志规则示例，这也是来自 AlmaLinux 机器的：

```
*.info;mail.none;authpriv.none;cron.none   /var/log/messages
```

这是详细的步骤：

+   `*.info`：这指的是来自所有设施的`info`级别及以上的消息。

+   `;`：这是一个复合规则。分号将该规则的不同部分彼此分开。

+   `mail.none;authpriv.none;cron.none`：这些是这个规则的三个例外。来自`mail`、`authpriv`和`cron`设施的消息不会被发送到`/var/log/messages`文件中。这三个设施有自己的日志文件规则。（我们之前看到的`authpriv`规则就是其中之一。）

Ubuntu 机器上的规则与 AlmaLinux 机器上的规则不完全相同。但如果你理解了这些示例，就不会有什么问题去弄懂 Ubuntu 的规则。

如果你对`rsyslog.conf`文件进行更改或向`/etc/rsyslog.d/`目录添加任何规则文件，你需要重新启动`rsyslog`守护进程以读取新的配置。你可以按如下方式进行操作：

```
[donnie@localhost ~]$ sudo systemctl restart rsyslog
[sudo] password for donnie: 
[donnie@localhost ~]$
```

现在你已经对`rsyslog`有了基本了解，接下来我们来看一下`journald`，它是新来的“新手”。

## 了解 journald

你会在任何使用`systemd`生态系统的 Linux 发行版上找到`journald`日志系统。`journald`不再将消息发送到文本文件，而是发送到二进制文件。你必须使用`journalctl`工具来提取信息，而不是使用普通的 Linux 文本文件工具。截止到本文编写时，我还不知道有哪个 Linux 发行版已经完全转向使用`journald`。目前使用`systemd`的 Linux 发行版同时运行`journald`和`rsyslog`。目前，RHEL 类型系统的默认设置是`journald`日志文件为临时文件，每次重启机器时都会被删除。（你可以配置`journald`使其日志文件持久化，但只要我们仍需要保留旧的`rsyslog`文件，这可能没有太大意义。）在 Ubuntu 上，默认设置是`journald`和`rsyslogd`都保持持久化日志文件。

在 RHEL 8/9 类型的发行版中，`journald`代替了`rsyslog`，现在实际收集操作系统其他部分的日志消息。但`rsyslog`仍然存在，收集来自`journald`的消息并将其发送到传统的`rsyslog`文本文件中。因此，日志文件管理的方式并没有真正改变。

完全从`rsyslog`过渡可能还需要几年时间。一个原因是，像 LogStash、Splunk 和 Nagios 这样的第三方日志聚合和分析工具，仍然被设置为读取文本文件，而不是二进制文件。另一个原因是，目前使用`journald`作为远程中央日志服务器仍处于概念验证阶段，尚未准备好投入生产使用。因此，目前`journald`还不能替代`rsyslog`。

> 几年前，Fedora 团队发布了一个只使用`journald`的版本，省略了`rsyslog`。太多人对此提出投诉，因此他们不得不在下一个版本的 Fedora 中重新引入`rsyslog`。

要查看完整的`journald`日志文件，请使用`journalctl`命令。在 Ubuntu 中，安装操作系统的人已经被添加到`adm`组，这使得该用户可以在没有 sudo 或 root 权限的情况下使用`journalctl`。任何后期添加的用户只能查看自己的消息。实际上，Frank 就是这样：

```
frank@ubuntu4:~$ journalctl
Hint: You are currently not seeing messages from other users and the system.
      Users in groups 'adm', 'systemd-journal' can see all messages.
      Pass -q to turn off this notice.
-- Logs begin at Tue 2019-11-26 17:43:28 UTC, end at Tue 2019-11-26 17:43:28 UTC. --
Nov 26 17:43:28 ubuntu4 systemd[10306]: Listening on GnuPG cryptographic agent and passphrase cache.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Reached target Timers.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Listening on GnuPG cryptographic agent and passphrase cache (restricted).
. . .
. . .
Nov 26 17:43:28 ubuntu4 systemd[10306]: Reached target Basic System.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Reached target Default.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Startup finished in 143ms.
frank@ubuntu4:~$
```

要查看来自系统或其他用户的消息，这些新用户必须被添加到`adm`或`systemd-journal`组，或者被授予适当的 sudo 权限。在 RHEL/CentOS/AlmaLinux 中，没有用户会自动加入`adm`或`systemd-journal`组。因此，最初，只有具有 sudo 权限的用户才能查看`journald`日志。

无论是`journalctl`还是`sudo journalctl`，根据需要，都会自动在`less`分页器中打开日志。你看到的内容与正常的`rsyslog`日志文件几乎一样，唯一的例外是：

+   长行会超出屏幕的右边缘。要查看剩余的行，可以使用右箭头键。

+   你还会看到颜色编码和高亮显示，使不同类型的消息更加突出。`ERROR`级别及更高级别的消息显示为红色，而从`NOTICE`级别到`ERROR`级别的消息则以粗体字符突出显示。

有许多选项可以以不同的格式显示不同类型的信息。例如，若要仅查看关于 CentOS 或 AlmaLinux 上 SSH 服务的消息，可以使用`--unit`选项，如下所示：

```
[donnie@localhost ~]$ sudo journalctl --unit=sshd
-- Logs begin at Tue 2019-11-26 12:00:13 EST, end at Tue 2019-11-26 15:55:19 EST. --
Nov 26 12:00:41 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Nov 26 12:00:42 localhost.localdomain sshd[825]: Server listening on 0.0.0.0 port 22.
Nov 26 12:00:42 localhost.localdomain sshd[825]: Server listening on :: port 22.
Nov 26 12:00:42 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Nov 26 12:22:08 localhost.localdomain sshd[3018]: Accepted password for donnie from 192.168.0.251 port 50797 ssh2
Nov 26 12:22:08 localhost.localdomain sshd[3018]: pam_unix(sshd:session): session opened for user donnie by (uid=0)
Nov 26 13:03:33 localhost.localdomain sshd[4253]: Accepted password for goldie from 192.168.0.251 port 50912 ssh2
Nov 26 13:03:34 localhost.localdomain sshd[4253]: pam_unix(sshd:session): session opened for user goldie by (uid=0)
[donnie@localhost ~]$
```

你不能使用 grep 工具来处理这些二进制日志，但你可以通过`-g`选项搜索一个字符串。默认情况下，它是不区分大小写的，即使目标文本字符串嵌入在另一个文本字符串中，它也能找到。在这里，我们看到它找到了文本字符串`fail`：

```
. . .
```

除了这些，还有很多其他选项。要查看它们，只需执行：

```
man journalctl
```

现在你已经了解了如何使用`rsyslog`和`journald`的基本操作，让我们看看一个很棒的工具，它能帮助你减轻日志审核的痛苦。

## 使用 Logwatch 简化操作

你知道每天审查日志有多重要，但你也知道这有多么令人沮丧，简直宁愿挨一顿狠打。幸运的是，有各种工具可以让这项工作变得更轻松。在正常的 Linux 发行版仓库中，Logwatch 是我最喜欢的工具。

Logwatch 没有第三方日志聚合器那样花哨的功能，但它依然非常不错。每天早晨，你会在你的邮箱账户中收到前一天日志的摘要。根据你的邮件系统配置，你可以将摘要发送到本地机器上的用户账户，或者发送到你可以随时访问的邮箱账户。设置起来非常简单，下面我们通过一个实际操作来演示。

### 实操实验 – 安装 Logwatch

为了传递消息，Logwatch 要求机器上必须有正在运行的邮件服务器守护进程。根据你在安装操作系统时选择的选项，你可能已经安装了 Postfix 邮件服务器，也可能没有。当 Postfix 被设置为本地服务器时，它会将系统消息发送到 root 用户的本地账户。

若要在本地机器上查看 Logwatch 摘要，你还需要安装一个文本模式的邮件阅读器，如 mutt。

对于这个实验，你可以使用任何一台虚拟机：

1.  安装 Logwatch、mutt 和 Postfix。（在 Ubuntu 上，安装 Postfix 时选择 `local` 选项。在 CentOS 或 AlmaLinux 上，`local` 选项已是默认选项。）在 Ubuntu 上，执行以下操作：

```
sudo apt install postfix mutt logwatch
```

对于 CentOS 7，执行以下操作：

```
sudo yum install postfix mutt logwatch
```

对于 AlmaLinux，请执行以下操作：

```
sudo dnf install postfix mutt logwatch
```

1.  仅限 Ubuntu，创建一个邮件存储文件到你的用户账户：

```
sudo touch /var/mail/your_user_name
```

1.  打开你喜欢的文本编辑器中的 `/etc/aliases` 文件。配置它，将 root 用户的邮件转发到你自己的普通账户，在文件底部添加以下行：

```
root:     your_user_name
```

1.  保存文件后，将其中的信息复制到系统可以读取的二进制文件中。你可以通过以下方式实现：

```
sudo newaliases
```

1.  到此为止，你已经完成了 Logwatch 的完整实现，它将以 *低级别* 的详细信息提供每日日志摘要。要查看默认配置，请查看默认配置文件：

```
less /usr/share/logwatch/default.conf/logwatch.conf
```

1.  要更改配置，编辑 CentOS 和 AlmaLinux 上的 `/etc/logwatch/conf/logwatch.conf` 文件，或者在 Ubuntu 上创建该文件。通过添加以下行来更改为中等级别的日志详细信息：

```
Detail = Med
```

> Logwatch 是一个 Python 脚本，每晚按计划运行。因此，不需要重启守护进程来使配置更改生效。

1.  执行一些操作来生成一些日志条目。你可以通过进行系统更新、安装一些软件包，以及使用 `sudo fdisk -l` 查看分区配置来实现。

1.  如果可能的话，让你的虚拟机运行整晚。第二天早上，通过以下方式查看你的日志摘要：

```
mutt
```

当提示是否在你的主目录中创建一个 `Mail` 目录时，按 *y* 键。

1.  实验结束。

既然你已经看过了简单的日志审查方法，接下来让我们进入本章的最后一个主题——如何搭建中央日志服务器。

## 设置远程日志服务器

到目前为止，我们只处理了本地机器上的日志文件。但如果不必登录到每台机器来查看日志文件，而是将每台机器的所有日志文件集中到一台服务器上，岂不是很方便？你完全可以做到，最棒的是，这非常简单。

但方便性并不是将日志文件集中收集到一个服务器上的唯一原因，还有日志文件的安全性问题。如果我们把所有日志文件都保留在每台主机上，网络入侵者就更容易找到这些文件并修改它们，删除有关其恶意活动的任何信息。（这很容易做到，因为大多数日志文件只是普通文本文件，可以在普通文本编辑器中编辑。）

### 实验操作——设置基本日志服务器

服务器的设置在 Ubuntu、CentOS 和 AlmaLinux 中是完全相同的。只是设置客户端时有一个小差异。为了获得最佳效果，请确保服务器虚拟机和客户端虚拟机具有不同的主机名：

1.  在日志收集服务器虚拟机上，打开`/etc/rsyslog.conf`文件，并使用你喜欢的文本编辑器查找这些行，它们通常位于文件的顶部：

```
# Provides TCP syslog reception
#module(load="imtcp") # needs to be done just once
#input(type="imtcp" port="514")
```

1.  取消注释底部的两行并保存文件。此时该段应该如下所示：

```
# Provides TCP syslog reception
module(load="imtcp") # needs to be done just once
input(type="imtcp" port="514")
```

1.  重启`rsyslog`守护进程：

```
sudo systemctl restart rsyslog
```

1.  如果机器上有启用防火墙，打开`514/tcp`端口。

1.  接下来，配置客户端机器。对于 Ubuntu，在`/etc/rsyslog.conf`文件底部添加以下行，并替换为你自己服务器虚拟机的 IP 地址：

```
@@192.168.0.161:514
```

对于 CentOS 和 AlmaLinux，查找`/etc/rsyslog.conf`文件底部的这一段：

```
# ### sample forwarding rule ###
#action(type="omfwd"  
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
#queue.filename="fwdRule1"       # unique name prefix for spool files
#queue.maxdiskspace="1g"         # 1gb space limit (use as much as possible)
#queue.saveonshutdown="on"       # save messages to disk on shutdown
#queue.type="LinkedList"         # run asynchronously
#action.resumeRetryCount="-1"    # infinite retries if host is down
# Remote Logging (we use TCP for reliable delivery)
# remote_host is: name/ip, e.g. 192.168.0.1, port optional e.g. 10514
#Target="remote_host" Port="XXX" Protocol="tcp"
```

删除每行不是显然为注释的注释符号。然后添加日志服务器虚拟机的 IP 地址和端口号。完成后应该如下所示：

```
# ### sample forwarding rule ###
action(type="omfwd"
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
queue.filename="fwdRule1"       # unique name prefix for spool files
queue.maxdiskspace="1g"         # 1gb space limit (use as much as possible)
queue.saveonshutdown="on"       # save messages to disk on shutdown
queue.type="LinkedList"         # run asynchronously
action.resumeRetryCount="-1"    # infinite retries if host is down
# Remote Logging (we use TCP for reliable delivery)
# remote_host is: name/ip, e.g. 192.168.0.1, port optional e.g. 10514
Target="192.168.0.161" Port="514" Protocol="tcp")
```

1.  保存文件后，重启`rsyslog`守护进程。

1.  在服务器虚拟机上，验证来自服务器虚拟机和客户端虚拟机的消息是否已成功发送到日志文件中。（你可以通过不同的主机名来区分不同的消息。）

1.  这就是实验的结束部分。

尽管这样很酷，但这个设置仍然存在一些缺点。其中之一是我们正在使用一个未加密的明文连接将日志文件发送到服务器。让我们来解决这个问题。

### 创建与日志服务器的加密连接

我们将使用`stunnel`包来创建加密连接。这很简单，只是 Ubuntu 和 AlmaLinux 的操作步骤不同。这些差异如下：

+   对于 AlmaLinux 8/9，FIPS 模块可以免费使用，正如我在*第六章*、*加密技术*中向你展示的那样。CentOS 7 不支持 FIPS 模块，而 Ubuntu 只有在购买支持合同的情况下才能使用。因此，目前唯一能利用`stunnel`中的 FIPS 模式的方法是将其配置在 AlmaLinux 8/9 或其他 RHEL 8/9 克隆版本上。

+   在 AlmaLinux 上，`stunnel`作为`systemd`服务运行。而在 Ubuntu 上，由于某些奇怪的原因，它仍然使用老式的`init`脚本。所以，我们不得不处理两种不同的方式来控制`stunnel`守护进程。

我们从 AlmaLinux 的操作开始。

#### 在 AlmaLinux 9 上创建 stunnel 连接——服务器端

对于本实验，我们使用的是一台已设置为 FIPS 合规模式的 AlmaLinux 9 虚拟机（查看*第六章*，*加密技术*中的步骤）：

1.  在 AlmaLinux 虚拟机上，安装`stunnel`：

```
sudo dnf install stunnel
```

1.  在服务器上，在`/etc/stunnel/`目录内，创建一个新的`stunnel.conf`文件，内容如下：

```
cert=/etc/stunnel/stunnel.pem
fips=yes
[hear from client]
accept=30000
connect=127.0.0.1:6514
```

1.  在服务器上，仍然在`/etc/stunnel/`目录中，创建`stunnel.pem`证书文件：

```
sudo openssl req -new -x509 -days 3650 -nodes -out stunnel.pem -keyout stunnel.pem
```

1.  在服务器上，打开防火墙上的端口`30000`，并关闭端口`514`：

```
sudo firewall-cmd --permanent --add-port=30000/tcp
sudo firewall-cmd --permanent --remove-port=514/tcp
sudo firewall-cmd --reload
```

> 端口`6514`，你在`stunnel.conf`文件中看到的端口，仅用于`rsyslog`和`stunnel`之间的内部通信。所以，对于这一点，我们不需要打开防火墙端口。我们将`stunnel`配置为在`rsyslog`的代表下监听端口`30000`，因此不再需要在防火墙上开放端口`514`。

1.  通过以下方式启用并启动`stunnel`守护进程：

```
sudo systemctl enable --now stunnel
```

1.  在`/etc/rsyslog.conf`文件中，查找文件顶部的这一行：

```
input(type="imtcp" port="514")
```

将其改为如下：

```
input(type="imtcp" port="6514")
```

1.  保存文件后，重启`rsyslog`：

```
sudo systemctl restart rsyslog
```

1.  现在服务器已经准备好通过加密连接接收来自远程客户端的日志文件。

接下来，我们将配置 AlmaLinux 虚拟机将其日志发送到此服务器。

#### 在 AlmaLinux 上创建 stunnel 连接——客户端

在这个步骤中，我们将配置一台 AlmaLinux 机器将其日志发送到日志服务器（无论日志服务器是运行在 CentOS、AlmaLinux 还是 Ubuntu 上，都没关系）：

1.  安装`stunnel`：

```
sudo dnf install stunnel
```

1.  在`/etc/stunnel/`目录中，创建`stunnel.conf`文件，内容如下：

```
client=yes
fips=yes
[speak to server]
accept=127.0.0.1:6514
connect=192.168.0.161:30000
```

> 在`connect`行中，将这里看到的日志服务器的 IP 地址替换为你自己日志服务器的 IP 地址。

1.  启用并启动`stunnel`守护进程：

```
sudo systemctl enable --now stunnel
```

1.  在`/etc/rsyslog.conf`文件的底部，查找这一行：

```
Target="192.168.0.161" Port="514" Protocol="tcp")
```

将其改为如下：

```
Target="127.0.0.1" Port="6514" Protocol="tcp")
```

1.  保存文件后，重启`rsyslog`守护进程：

```
sudo systemctl restart rsyslog
```

1.  在客户端，使用`logger`将消息发送到日志文件：

```
logger "This is a test of the stunnel setup."
```

1.  在服务器上，验证消息是否已添加到`/var/log/messages`文件中。

1.  本实验到此结束。

现在让我们转向 Ubuntu。

#### 在 Ubuntu 上创建 stunnel 连接——服务器端

为此，我们将使用 Ubuntu 22.04 虚拟机。我不明白为什么，但 Ubuntu 仍然使用旧式的`init`脚本来运行`stunnel`，而不是使用`systemd`服务。所以，你将使用的命令与平时的有所不同。

1.  安装`stunnel`：

```
sudo apt install stunnel
```

1.  在`/etc/stunnel/`目录中，创建`stunnel.conf`文件，内容如下：

```
cert=/etc/stunnel/stunnel.pem
fips=no
[hear from client]
accept=30000
connect=6514
```

1.  在`/etc/stunnel/`目录中，仍然创建`stunnel.pem`证书：

```
sudo openssl req -new -x509 -days 3650 -nodes -out stunnel.pem -keyout stunnel.pem
```

1.  启动`stunnel`守护进程：

```
sudo service stunnel4 start
```

1.  为了在系统重启时自动启动该守护进程，创建一个 root 用户的定时任务。首先，打开 crontab 编辑器，方法如下：

```
sudo crontab -e -u root
```

将此行添加到文件的末尾：

```
@reboot service stunnel4 start
```

1.  在`/etc/rsyslog.conf`文件的顶部，查找这一行：

```
input(type="imtcp" port="514")
```

将其修改为如下：

```
input(type="imtcp" port="6514")
```

1.  保存文件后，重新启动`rsyslog`守护进程：

```
sudo systemctl restart rsyslog
```

1.  使用适当的`iptables`、`ufw`或`nftables`命令，打开防火墙上的`30000/tcp`端口，并关闭`514`端口。

1.  这就是实验的结束。

接下来，我们将配置客户端。

#### 在 Ubuntu 上创建 stunnel 连接 – 客户端配置

使用此程序，在 Ubuntu 客户端上操作将允许它将日志文件发送到 AlmaLinux 或 Ubuntu 日志服务器：

1.  安装`stunnel`：

```
sudo apt install stunnel
```

1.  在`/etc/stunnel/`目录下，创建`stunnel.conf`文件，内容如下：

```
client=yes
fips=no
[speak to server]
accept = 127.0.0.1:6514
connect=192.168.0.161:30000
```

> 请注意，尽管我们无法在 Ubuntu 客户端上使用 FIPS 模式，但仍然可以将日志文件发送到配置为使用 FIPS 模式的 AlmaLinux 日志服务器。（所以，确实可以混用不同的系统。）

1.  启动`stunnel`守护进程：

```
sudo service stunnel4 start
```

1.  为了在系统重启时自动启动该守护进程，创建一个定时任务。通过以下方法打开 crontab 编辑器：

```
sudo crontab -e -u root
```

将此行添加到文件末尾：

```
@reboot service stunnel4 start
```

1.  在`/etc/rsyslog.conf`文件的底部，查找包含日志服务器 IP 地址的行。将其更改为如下：

```
@@127.0.0.1:6514
```

1.  保存文件后，重新启动`rsyslog`守护进程：

```
sudo systemctl restart rsyslog
```

1.  使用`logger`命令将消息发送到日志服务器：

```
 logger "This is a test of the stunnel connection."
```

1.  在服务器上，验证消息是否出现在`/var/log/messages`或`/var/log/syslog`文件中，具体取决于情况。

1.  实验结束。

好了，现在我们已经建立了一个安全的连接，这很好。但是，来自所有客户端的消息仍然混杂在服务器的日志文件中。我们来解决这个问题。

### 将客户端消息分隔到各自的文件中

这也是非常简单的操作。我们只需对日志服务器上的`rsyslog`规则做几个简单的编辑，然后重启`rsyslog`守护进程。为了演示，我将使用 AlmaLinux 9 虚拟机。

> **重要**
> 
> > 如果你实现了这个技巧，就无法使用 Logwatch 了。嗯，实际上你是可以使用的，只不过 Logwatch 会将所有客户端文件中的事件合并成一个大摘要。所以，你无法看到哪些客户端机器生成了哪些事件。

在`/etc/rsyslog.conf`文件的 RULES 部分，查找以下这一行：

```
*.info;mail.none;authpriv.none;cron.none   /var/log/messages
```

将其修改为如下：

```
*.info;mail.none;authpriv.none;cron.none ?Rmessages
```

在该行上方，插入以下行：

```
$template Rmessages,"/var/log/%HOSTNAME%/messages"
```

对`auth`消息进行同样的操作：

```
# authpriv.* /var/log/secure
$template Rauth,"/var/log/%HOSTNAME%/sec
auth.*,authpriv.* ?Rauth
```

最后，重启`rsyslog`：

```
sudo systemctl restart rsyslog
```

查看`/var/log/`目录，你会看到每个客户端的目录，这些客户端正在向该服务器发送日志。相当酷，对吧？

> 提示
> 
> > 这里的技巧是：始终让`$template`行*位于*受影响规则之前。

本章到此结束。你现在知道如何查看日志文件，如何使日志审查更加轻松，并且了解如何设置一个安全的远程日志服务器。

## 总结

在本章中，我们介绍了不同类型的日志文件，重点是包含与安全相关信息的文件。接着，我们看了 `rsyslog` 和 `journald` 日志系统的基本操作。为了简化日志审查，我们介绍了 Logwatch，它会自动创建前一天日志文件的摘要。最后，我们通过设置一个集中式的远程日志服务器，收集来自其他网络主机的日志文件。

在下一章中，我们将探讨如何进行漏洞扫描和入侵检测。下次见。

## 问题

1.  以下哪两个是记录身份验证相关事件的日志文件？

    1.  `syslog`

    1.  `authentication.log`

    1.  `auth.log`

    1.  `secure.log`

    1.  `secure`

1.  哪个日志文件包含有关当前登录系统的用户及其操作的记录？

    1.  `/var/log/syslog`

    1.  `/var/log/utmp`

    1.  `/var/log/btmp`

    1.  `/var/run/utmp`

1.  以下哪一个是几乎每个现代 Linux 发行版上都运行的主要日志系统？

    1.  `syslog`

    1.  `rsyslog`

    1.  `journald`

    1.  `syslog-ng`

1.  以下哪个是 RHEL 8/9 及其衍生版本（如 AlmaLinux 8/9）特有的？

    1.  在 RHEL 8/9 系统上，`journald` 从系统的其他部分收集日志数据，并将其发送到 `rsyslog`。

    1.  在 RHEL 8/9 系统上，`journald` 已完全取代了 `rsyslog`。

    1.  在 RHEL 8/9 系统上，`rsyslog` 从系统的其他部分收集数据，并将其发送到 `journald`。

    1.  RHEL 8/9 系统使用 `syslog-ng`。

1.  在设置 `stunnel` 时，以下哪项是需要考虑的？

    1.  在 AlmaLinux 系统上，无法使用 FIPS 模式。

    1.  在 Ubuntu 系统上，无法使用 FIPS 模式。

    1.  在 Ubuntu 系统上，FIPS 模式是可用的，但前提是购买了支持合同。

    1.  在 AlmaLinux 8/9 系统上，FIPS 模式是可用的，但前提是购买了支持合同。

1.  以下哪两个关于 `stunnel` 的说法是正确的？

    1.  在 RHEL 系统上，`stunnel` 作为普通的 `systemd` 服务运行。

    1.  在 RHEL 系统上，`stunnel` 仍然通过旧式的 `init` 脚本运行。

    1.  在 Ubuntu 系统上，`stunnel` 作为普通的 `systemd` 服务运行。

    1.  在 Ubuntu 系统上，`stunnel` 通过旧式的 `init` 脚本运行。

1.  你必须编辑哪个文件才能将 root 用户的消息转发到你自己的用户账户？

1.  编辑完 *问题 7* 中提到的文件后，必须运行哪个命令才能将信息传输到系统可读的二进制文件中？

1.  要为你的远程日志服务器创建一个 `stunnel` 设置，你必须为服务器和每个客户端都创建一个安全证书。

    1.  正确

    1.  错误

1.  以下哪个命令可用于查找 `journald` 日志文件中的 `fail` 字符串？

    1.  `sudo grep fail /var/log/journal/messages`

    1.  `sudo journalctl -g fail`

    1.  `sudo journalctl -f fail`

    1.  `sudo less /var/log/journal/messages`

## 深入阅读

+   五个开源日志管理程序：[`fosspost.org/lists/open-source-log-management`](https://fosspost.org/lists/open-source-log-management)

+   *什么是 SIEM?*: [`www.tripwire.com/state-of-security/incident-detection/log-management-siem/what-is-a-siem/`](https://www.tripwire.com/state-of-security/incident-detection/log-management-siem/what-is-a-siem/)

+   *你必须监控的 12 个关键 Linux 日志文件*: [`www.eurovps.com/blog/important-linux-log-files-you-must-be-monitoring/`](https://www.eurovps.com/blog/important-linux-log-files-you-must-be-monitoring/)

+   *分析 Linux 日志*: [`www.loggly.com/ultimate-guide/analyzing-linux-logs/`](https://www.loggly.com/ultimate-guide/analyzing-linux-logs/)

+   Linux 日志文件示例: [`www.poftut.com/linux-log-files-varlog/`](https://www.poftut.com/linux-log-files-varlog/)

+   `rsyslog`主页: [`www.rsyslog.com/`](https://www.rsyslog.com/)

+   *为什么使用 Journald?*: [`www.loggly.com/blog/why-journald/`](https://www.loggly.com/blog/why-journald/)

+   Journalctl 备忘单: [`www.golinuxcloud.com/view-logs-using-journalctl-filter-journald/`](https://www.golinuxcloud.com/view-logs-using-journalctl-filter-journald/)

+   *Linux 管理实用手册*，作者：Adam K. Dean: [`www.packtpub.com/virtualization-and-cloud/linux-administration-cookbook`](https://www.packtpub.com/virtualization-and-cloud/linux-administration-cookbook)

+   Logwatch 项目页面: [`sourceforge.net/projects/logwatch/`](https://sourceforge.net/projects/logwatch/)

+   `stunnel`主页: [`www.stunnel.org/`](https://www.stunnel.org/)

+   *使用 systemd 简化 Linux 服务管理*，作者：Donald A. Tevault: [`www.packtpub.com/product/linux-service-management-made-easy-with-systemd/`](https://www.packtpub.com/product/linux-service-management-made-easy-with-systemd/)

## 回答

1.  c, e

1.  d

1.  b

1.  a

1.  c

1.  a, d

1.  `/etc/aliases`

1.  `sudo newaliases`

1.  b

1.  c
