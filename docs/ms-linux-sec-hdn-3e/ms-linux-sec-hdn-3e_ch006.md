# 5 使用防火墙保护您的服务器 - 第二部分

## 加入我们的书籍社区 Discord

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/file35.png)

在 *第四章* 的 *使用防火墙保护您的服务器 - 第一部分* 中，我们讨论了 iptables 和 nftables，这是直接与 netfilter 接口的管理实用工具。虽然熟悉 iptables 和 nftables 命令可以帮助创建高级防火墙配置，但经常需要使用这些命令来执行日常操作可能会有些不便。在本章中，我们将看看 ufw 和 firewalld，这些是辅助工具，可以简化与 iptables 或 nftables 的工作过程。

首先，我们将看看 ufw。我们将查看它的结构、命令和配置。然后，我们将对 firewalld 做同样的操作。在两种情况下，您都将获得大量的实践操作。

我们将在本章中涵盖以下主题：

+   **ufw** 适用于 Ubuntu 系统

+   **firewalld** 适用于 Red Hat 系统

## 技术要求

本章的代码文件在这里可用：[`github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-Second-Edition.`](https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-Second-Edition.)

## 适用于 Ubuntu 系统的简单防火墙

ufw 已经安装在 Ubuntu 20.04 和 Ubuntu 22.04 上。在 Ubuntu 20.04 上仍使用 iptables 后端，在 Ubuntu 22.04 上使用 nftables 后端。对于日常操作，它提供了大大简化的命令集。执行一个简单的命令以打开所需端口，再执行另一个简单的命令以激活它，您就有了一个良好的基本防火墙。每次执行`ufw`命令时，它将自动配置 IPv4 和 IPv6 规则。这单独就是一个巨大的时间节省器，很多我们之前需要手动配置的内容在默认情况下已经包含了。尽管我们的两个 Ubuntu 版本使用了不同的后端，但 ufw 的配置对它们两者来说是相同的。

> 提示：
> 
> > ufw 也适用于 Debian 和其他基于 Debian 的发行版，但可能未安装。如果是这种情况，请通过发出`sudo apt install ufw`命令来安装它。

### 配置 ufw

在 Ubuntu 20.04 和 Ubuntu 22.04 上，ufw 服务已默认启用，但防火墙本身尚未激活。换句话说，系统的服务正在运行，但尚未执行任何防火墙规则。（稍后我们将向您展示如何在我们讨论完需要打开的端口后激活它。）使用以下两个命令检查 ufw 状态：

```
systemctl status ufw
sudo ufw status
```

systemctl 命令应该显示服务已启用，而 ufw 命令应该显示防火墙未激活。

我们首先要做的事情是打开端口`22`，以允许通过安全外壳连接到机器，如下所示：

```
sudo ufw allow 22/tcp
```

好的，看起来不错。现在让我们像这样激活防火墙：

```
sudo ufw enable
```

通过在 Ubuntu 20.04 上使用 `sudo iptables -L`，你将看到新的 Secure Shell 规则出现在 `ufw-user-input` 链中：

```
Chain ufw-user-input (1 references)
 target prot opt source destination
 ACCEPT tcp -- anywhere anywhere tcp dpt:ssh
```

在 Ubuntu 22.04 上，使用 `sudo nft list ruleset` 命令查看 `ufw-user-input` 链中的新规则：

```
chain ufw-user-input {
        meta l4proto tcp tcp dport 22 counter packets 0 bytes 0 accept
    }
```

你还会看到，这两个命令的输出非常冗长，因为我们用裸 iptables 或 nftables 必须做的许多工作，ufw 已经为我们完成了。事实上，这里甚至有比我们用 iptables 和 nftables 做的更多的内容。例如，使用 ufw 时，我们已经有了速率限制规则，帮助我们防范 **拒绝服务** (**DoS**) 攻击，我们还有记录被阻止的包的日志规则。这几乎是设置防火墙的“轻松不麻烦”的方式。（稍后我会讲讲那个 *几乎* 的部分。）

在前面的 `sudo ufw allow 22/tcp` 命令中，我们必须指定 TCP 协议，因为 Secure Shell 只需要 TCP 协议。如果你没有指定协议，也可以同时为 TCP 和 UDP 打开一个端口。例如，如果你正在设置 DNS 服务器，你需要为两个协议都打开端口 `53`。（你会看到端口 `53` 的条目列为域名端口）。在任一版本的 Ubuntu 上，执行：

```
 sudo ufw allow 53
```

在 Ubuntu 20.04 中，通过以下命令查看结果：

```
 sudo iptables -L
. . .
. . .
    Chain ufw-user-input (1 references)
    target prot opt source destination
    ACCEPT tcp -- anywhere anywhere tcp dpt:ssh
    ACCEPT tcp -- anywhere anywhere tcp dpt:domain
    ACCEPT udp -- anywhere anywhere udp dpt:domain
```

在 Ubuntu 22.04 中，通过以下命令查看结果：

```
sudo nft list ruleset
chain ufw-user-input {
        meta l4proto tcp tcp dport 22 counter packets 0 bytes 0 accept
        meta l4proto tcp tcp dport 53 counter packets 0 bytes 0 accept
        meta l4proto udp udp dport 53 counter packets 0 bytes 0 accept
    }
```

如果你在 20.04 机器上执行 `sudo ip6tables -L`，你会看到针对 IPv6 的规则也已经添加，适用于之前的两个示例。同样，你会看到我们使用 ip6tables 命令时需要做的大部分工作，ufw 已经帮我们处理好了。（尤其好的是，我们不需要处理那些麻烦的 ICMP 规则。）在 22.04 机器上，你之前执行的 `sudo nft list ruleset` 命令会在 `ufw6-user-input` 段落中显示 IPv6 配置。

要快速查看防火墙配置的摘要，请使用 `status` 选项。输出应该类似如下：

```
donnie@ubuntu-ufw:~$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     LIMIT       Anywhere                  
53                         LIMIT       Anywhere                 
22/tcp (v6)                LIMIT       Anywhere (v6)             
53 (v6)                    LIMIT       Anywhere (v6)             

donnie@ubuntu-ufw:~$
```

接下来，我们将查看 ufw 配置文件。

### 使用 ufw 配置文件

你可以在 `/etc/ufw/` 目录中找到 ufw 防火墙规则。正如你所看到的，规则存储在多个不同的文件中：

```
donnie@ubuntu-ufw:/etc/ufw$ ls -l
total 48
-rw-r----- 1 root root  915 Aug  7 15:23 after6.rules
-rw-r----- 1 root root 1126 Jul 31 14:31 after.init
-rw-r----- 1 root root 1004 Aug  7 15:23 after.rules
drwxr-xr-x 3 root root 4096 Aug  7 16:45 applications.d
-rw-r----- 1 root root 6700 Mar 25 17:14 before6.rules
-rw-r----- 1 root root 1130 Jul 31 14:31 before.init
-rw-r----- 1 root root 3467 Aug 11 11:36 before.rules
-rw-r--r-- 1 root root 1391 Aug 15  2017 sysctl.conf
-rw-r--r-- 1 root root  313 Aug 11 11:37 ufw.conf
-rw-r----- 1 root root 3014 Aug 11 11:37 user6.rules
-rw-r----- 1 root root 3012 Aug 11 11:37 user.rules
donnie@ubuntu-ufw:/etc/ufw$
```

在列表的底部，你会看到 `user6.rules` 和 `user.rules` 文件。你不能手动编辑这两个文件。虽然你可以在编辑后保存文件，但当你使用 `sudo ufw reload` 加载新更改时，你会发现你的编辑已被删除。让我们查看 `user.rules` 文件，看看里面有什么内容。

> 提示：
> 
> > 如你所见，Ubuntu 20.04 和 22.04 的所有文件都包含 iptables 格式的防火墙规则，即使 22.04 使用 nftables 作为后端。这是因为 Ubuntu 22.04 可以自动将 iptables 规则转换为 nftables 规则。因此，20.04 和 22.04 的文件是相同的，这让我们操作起来非常方便。

在文件的顶部，你会看到定义了 iptables 过滤表以及它的关联链表：

```
*filter
:ufw-user-input - [0:0]
:ufw-user-output - [0:0]
:ufw-user-forward - [0:0]
. . .
. . .
```

接下来，在`### RULES ###`部分，我们列出了使用`ufw`命令创建的规则。以下是我们打开 DNS 端口的规则示例：

```
### tuple ### allow any 53 0.0.0.0/0 any 0.0.0.0/0 in
-A ufw-user-input -p tcp --dport 53 -j ACCEPT
-A ufw-user-input -p udp --dport 53 -j ACCEPT
```

正如你所看到的，ufw 在其配置文件中使用 iptables 语法，即使是在 Ubuntu 22.04 上也是如此。

在`### RULES ###`部分下方，我们可以看到有关防火墙阻止的任何数据包的日志消息规则：

```
### LOGGING ###
-A ufw-after-logging-input -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-A ufw-after-logging-forward -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-I ufw-logging-deny -m conntrack --ctstate INVALID -j RETURN -m limit --limit 3/min --limit-burst 10
-A ufw-logging-deny -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-A ufw-logging-allow -j LOG --log-prefix "[UFW ALLOW] " -m limit --limit 3/min --limit-burst 10
### END LOGGING ###
```

这些消息会发送到`/var/log/kern.log`文件。为了避免在大量数据包被阻止时压垮日志系统，我们将每分钟最多发送三条消息到日志文件，并设置每分钟 10 条消息的突发限制。大多数规则会在日志消息中插入`[UFW BLOCK]`标签，方便我们查找它们。最后一条规则会创建带有`[UFW ALLOW]`标签的消息，奇怪的是，`INVALID`规则并不会插入任何标签。

最后，我们有了速率限制规则，每个用户每分钟只允许三次连接：

```
### RATE LIMITING ###
-A ufw-user-limit -m limit --limit 3/minute -j LOG --log-prefix "[UFW LIMIT BLOCK] "
-A ufw-user-limit -j REJECT
-A ufw-user-limit-accept -j ACCEPT
### END RATE LIMITING ###
```

超过该限制的任何数据包将会以`[UFW LIMIT BLOCK]`标签记录在`/var/log/kern.log`文件中。

`/etc/ufw user6.rules`文件看起来几乎一样，只不过它是用于 IPv6 规则的。每次你使用`ufw`命令创建或删除规则时，它都会同时修改`user.rules`文件和`user6.rules`文件。

为了存储在`user.rules`和`user6.rules`文件之前运行的规则，我们有`before.rules`文件和`before6.rules`文件。为了存储在`user.rules`和`user6.rules`文件之后运行的规则，我们有——你猜对了——`after.rules`文件和`after6.rules`文件。如果你需要添加不能通过`ufw`命令添加的自定义规则，只需手动编辑这对文件之一。（稍后我们会详细讨论这个问题。）

如果你查看`before`和`after`文件，你会看到很多已经为我们处理好的内容。这些都是我们曾经需要使用 iptables/ip6tables 或 nftables 手动完成的工作。

然而，正如你可能知道的，这些 ufw 的优势中有一个小小的注意事项。你可以使用 ufw 工具执行简单的任务，但任何更复杂的操作都需要手动编辑文件。（这就是我说 ufw 是*几乎*不麻烦、不复杂的原因。）

> 提示：
> 
> > 要查看更多使用`ufw`命令的示例，可以通过执行以下命令查看其手册页：

```
man ufw
```

例如，在`before`文件中，你会看到其中一个阻止无效数据包的规则已经被实现。以下是`before.rules`文件中的代码片段，通常可以在文件的顶部找到：

```
# drop INVALID packets (logs these in loglevel medium and higher)
-A ufw-before-input -m conntrack --ctstate INVALID -j ufw-logging-deny
-A ufw-before-input -m conntrack --ctstate INVALID -j DROP
```

这两条规则中的第二条实际上丢弃了无效的数据包，而第一条则记录了它们。但正如我们在 *第四章* 中的 *iptables 概述* 部分所见，*通过防火墙保护服务器第一部分*，这一特定的 `DROP` 规则并不会阻止所有无效的数据包。而且，为了提高性能，我们宁愿将这个规则放在 mangle 表中，而不是现在所在的 filter 表中。为了解决这个问题，我们将编辑两个 `before` 文件。在你喜欢的文本编辑器中打开 `/etc/ufw/before.rules` 文件，寻找文件底部的以下一对行：

```
# don't delete the 'COMMIT' line or these rules won't be processed
COMMIT
```

在 `COMMIT` 行下方，添加以下代码片段以创建 mangle 表规则：

```
# Mangle table added by Donnie
*mangle
:PREROUTING ACCEPT [0:0]
-A PREROUTING -m conntrack --ctstate INVALID -j DROP
-A PREROUTING -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j DROP
COMMIT
```

现在，我们将对 `/etc/ufw/before6.rules` 文件重复此过程。然后，通过以下命令重新加载规则：

```
sudo ufw reload
```

通过在 Ubuntu 20.04 上使用 `iptables -L` 和 `ip6tables -L` 命令，或在 Ubuntu 22.04 上使用 `nft list ruleset` 命令，你将看到新规则出现在 mangle 表中，正是我们希望它们出现的位置。

#### 基本 ufw 使用的实操实验

你需要在 Ubuntu 20.04 或 Ubuntu 22.04 的干净快照虚拟机上完成此实验。让我们开始吧：

1.  关闭你的 Ubuntu 虚拟机，并恢复快照以删除你刚才所做的所有 iptables 或 nftables 设置。（或者，如果你更喜欢，可以从一台全新的虚拟机开始。）

1.  重启虚拟机后，验证 `iptables` 规则是否已经消失。在 Ubuntu 20.04 上执行：

```
sudo iptables -L
On Ubuntu 22.04 do:
sudo nft list ruleset
```

1.  查看 `ufw` 的状态。打开端口 `22/TCP`，然后启用 `ufw`。接着，查看结果：

```
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status
On Ubuntu 20.04 do:
sudo iptables -L
sudo ip6tables -L
On Ubuntu 22.04 do:
sudo nft list ruleset
```

1.  这次，打开端口 `53`，同时为 TCP 和 UDP 都开放：

```
sudo ufw allow 53
sudo ufw status
On Ubuntu 20.04 do:
sudo iptables -L
sudo ip6tables -L
On Ubuntu 22.04 do:
sudo nft list ruleset
```

1.  `cd` 进入 `/etc/ufw/` 目录。熟悉该目录下文件的内容。

1.  在你喜欢的文本编辑器中打开 `/etc/ufw/before.rules` 文件。在文件底部，`COMMIT` 行下面，添加以下代码片段：

```
# Mangle table added by Donnie
*mangle
:PREROUTING ACCEPT [0:0]
-A PREROUTING -m conntrack --ctstate INVALID -j DROP
-A PREROUTING -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j DROP
COMMIT
(Note that the second PREROUTING command wraps around on the printed page.)
```

1.  对 `/etc/ufw/before6.rules` 文件重复执行步骤 *6*。

1.  使用以下命令重新加载防火墙：

```
sudo ufw reload
```

1.  在 Ubuntu 20.04 上，执行以下命令查看规则：

```
sudo iptables -L
sudo iptables -t mangle -L
sudo ip6tables -L
sudo ip6tables -t mangle -L
On Ubuntu 22.04, observe the rules by doing:
sudo nft list ruleset
```

1.  快速查看 `ufw` 的状态：

```
sudo ufw status
```

实验结束了——恭喜你！

我想你会同意，`ufw` 是非常酷的技术。它用来执行基本任务的命令比等效的 iptables 或 nftables 命令更容易记住，而且只需一个命令就能同时处理 IPv4 和 IPv6。在我们的任一版本的 Ubuntu 上，你仍然可以通过手动编辑 ufw 配置文件来做一些复杂的事情。但，ufw 并不是唯一一个非常酷的防火墙管理工具。接下来的部分，我们将看看 Red Hat 的开发者给我们提供了什么。

## Red Hat 系统的 firewalld

接下来，我们将注意力转向 **firewalld**，它是 Red Hat Enterprise Linux 7 至 9 以及所有衍生版本的默认防火墙管理工具。

就像我们在 Ubuntu 上看到的 ufw 一样，firewalld 可以是 iptables 或 nftables 的前端。在 RHEL/CentOS 7 上，firewalld 使用 iptables 引擎作为后端。在 RHEL 8 和 9 类型的发行版中，firewalld 使用 nftables 作为后端。不管怎样，当 firewalld 启用时，你不能使用普通的 iptables 或 nftables 命令来创建规则，因为 firewalld 将规则存储在不兼容的格式中。

> 提示：
> 
> > 直到最近，firewalld 仅在较新的 RHEL 版本及其衍生版本中可用。然而，现在 firewalld 也可以在 Ubuntu 的软件库中找到。所以，如果你想在 Ubuntu 上运行 firewalld，现在终于可以选择了。此外，firewalld 和 nftables 的组合现在已经在 SUSE 系统中预安装并启用。

如果你在桌面机器上运行 Red Hat、CentOS 或 AlmaLinux，你会发现应用程序菜单中有一个 firewalld 的 GUI 前端。但是在文本模式服务器上，你只有 firewalld 命令。出于某种原因，Red Hat 并没有为文本模式服务器创建一个类似 ncurses 的程序，就像他们为旧版本的 Red Hat 中的 iptables 配置做的那样。

firewalld 的一个大优势是它是动态管理的。这意味着你可以在不重启防火墙服务的情况下更改防火墙配置，并且不会中断与服务器的现有连接。

在我们查看 RHEL 7/CentOS 7 与 RHEL/AlmaLinux 8 和 9 版本的 firewalld 差异之前，让我们先看一下两者相同的部分。

### 验证 firewalld 状态

对于本节内容，你可以使用 CentOS 7、AlmaLinux 8 或 AlmaLinux 9 虚拟机。我们先从验证 firewalld 的状态开始。这样有两种方法。第一种方法是使用 `firewall-cmd` 的 `--state` 选项：

```
[donnie@localhost ~]$ sudo firewall-cmd --state
 running
 [donnie@localhost ~]$
```

另外，如果我们需要更详细的状态信息，我们可以检查守护进程，就像我们在 systemd 系统上检查其他守护进程一样：

```
[donnie@localhost ~]$ sudo systemctl status firewalld
 firewalld.service - firewalld - dynamic firewall daemon
  Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled;
 vendor preset: enabled)
  Active: active (running) since Fri 2017-10-13 13:42:54 EDT; 1h 56min ago
  Docs: man:firewalld(1)
  Main PID: 631 (firewalld)
  CGroup: /system.slice/firewalld.service
  └─631 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
. . .
 Oct 13 15:19:41 localhost.localdomain firewalld[631]: WARNING: reject-
 route: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
 [donnie@localhost ~]$
```

接下来，让我们看看 firewalld 区域。

### 使用 firewalld 区域

`firewalld` 是一种非常独特的工具，它带有几个预配置的区域和服务。如果你查看任何 CentOS 或 AlmaLinux 机器的 `/usr/lib/firewalld/zones/` 目录，你会看到所有的区域文件，它们都是 `.xml` 格式：

```
[donnie@localhost ~]$ cd /usr/lib/firewalld/zones
[donnie@localhost zones]$ ls
block.xml dmz.xml drop.xml external.xml home.xml internal.xml public.xml
trusted.xml work.xml
[donnie@localhost zones]$
```

每个区域文件指定了在不同情况下需要开放的端口和需要阻止的端口。区域还可以包含 ICMP 消息、转发端口、伪装信息和丰富语言规则。例如，设置为默认的公共区域的 `.xml` 文件看起来是这样的：

```
<?xml version="1.0" encoding="utf-8"?> 
<zone> 
 <short>Public</short> 
 <description>For use in public areas. You do not trust the other 
computers on networks to not harm your computer. Only selected incoming 
connections are accepted.</description> 
 <service name="ssh"/> 
 <service name="dhcpv6-client"/> 
</zone> 
```

在 `service name` 行中，你可以看到唯一开放的端口是用于安全外壳访问和 DHCPv6 发现的端口。如果你查看 `home.xml` 文件，你会发现它还开放了用于多播 DNS 的端口，以及允许此机器从 Samba 服务器或 Windows 服务器访问共享目录的端口：

```
<?xml version="1.0" encoding="utf-8"?> 
<zone> 
 <short>Home</short> 
 <description>For use in home areas. You mostly trust the other computers 
on networks to not harm your computer. Only selected incoming connections 
are accepted.</description> 
 <service name="ssh"/>
 <service name="mdns"/> 
 <service name="samba-client"/> 
 <service name="dhcpv6-client"/> 
</zone>
```

`firewall-cmd`工具是用来配置`firewalld`的。你可以使用它查看系统中区域文件的列表，而无需`cd`进入区域文件目录：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-zones
[sudo] password for donnie:
block dmz drop external home internal public trusted work
[donnie@localhost ~]$
```

快速查看每个区域配置的方法是使用`--list-all-zones`选项：

```
[donnie@localhost ~]$ sudo firewall-cmd --list-all-zones
 block
  target: %%REJECT%%
  icmp-block-inversion: no
  interfaces:
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
. . .
. . .
```

当然，这只是输出的一部分，因为所有区域的列表比我们在这里显示的要多。你更有可能只想查看一个特定区域的信息：

```
 [donnie@localhost ~]$ sudo firewall-cmd --info-zone=internal
 internal
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh mdns samba-client dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
 source-ports:
  icmp-blocks:
  rich rules:
 [donnie@localhost ~]$
```

所以，`internal`区域允许`ssh`、`mdns`、`samba-client`和`dhcpv6-client`服务。这对于在内部局域网上设置客户端机器非常方便。

每个给定的服务器或客户端都将拥有一个或多个已安装的网络接口适配器。每个适配器在一台机器中只能分配一个、且仅能分配一个 firewalld 区域。要查看默认区域，可以执行以下操作：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-default-zone
 public
[donnie@localhost ~]$
```

这很好，除了它没有告诉你与此区域关联的网络接口是什么。要查看该信息，请执行以下操作：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-active-zones
 public
  interfaces: enp0s3
[donnie@localhost ~]$
```

当你第一次安装 Red Hat、CentOS 或 AlmaLinux 时，防火墙会默认启用，并且公共区域是默认区域。现在，假设你在 DMZ 中设置服务器，并希望确保它的防火墙为此进行锁定。你可以将默认区域更改为`dmz`区域。我们来看一下`dmz.xml`文件，看看这对我们有什么帮助：

```
<?xml version="1.0" encoding="utf-8"?> 
<zone> 
 <short>DMZ</short> 
 <description>For computers in your demilitarized zone that are publicly- 
accessible with limited access to your internal network. Only selected 
incoming connections are accepted.</description> 
 <service name="ssh"/> 
</zone> 
```

所以，DMZ 区域允许通过的唯一内容是安全外壳（SSH）。好吧；现在这样就足够了，让我们将`dmz`区域设置为默认区域：

```
[donnie@localhost ~]$ sudo firewall-cmd --set-default-zone=dmz
 [sudo] password for donnie:
 success
[donnie@localhost ~]$
```

让我们验证一下：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-default-zone
 dmz
[donnie@localhost ~]$
```

一切都好了。然而，位于 DMZ 中的面向互联网的服务器可能需要允许的不仅仅是 SSH 连接。这时我们将使用 firewalld 服务。但在查看这些之前，我们先考虑一个更重要的点。

> 小贴士：
> 
> > 设置默认区域时，你不需要使用`--permanent`选项。事实上，如果你使用了该选项，会出现错误信息。

你永远不应修改`/usr/lib/firewalld/`目录下的文件。每当你修改 firewalld 配置时，修改后的文件会出现在`/etc/firewalld/`目录下。到目前为止，我们只修改了默认区域。因此，我们将在`/etc/firewalld/`目录下看到以下文件：

```
[donnie@localhost ~]$ sudo ls -l /etc/firewalld
 total 12
 -rw-------. 1 root root 2003 Oct 11 17:37 firewalld.conf
 -rw-r--r--. 1 root root 2006 Aug 4 17:14 firewalld.conf.old
 . . .
```

我们可以对这两个文件做一个`diff`，查看它们之间的差异：

```
[donnie@localhost ~]$ sudo diff /etc/firewalld/firewalld.conf /etc/firewalld/firewalld.conf.old
 6c6
 < DefaultZone=dmz
 ---
 > DefaultZone=public
 [donnie@localhost ~]$
```

所以，较新的这两个文件显示`dmz`区域现在是默认区域。

> 小贴士：
> 
> > 要了解更多关于 firewalld 区域的信息，可以输入`man firewalld.zones`命令。

### 将服务添加到 firewalld 区域

每个服务文件都包含需要为特定服务打开的端口列表。可选地，服务文件可能包含一个或多个目标地址，或者调用所需的任何模块，例如连接跟踪。对于某些服务，你只需打开一个端口。其他服务，如 Samba 服务，则要求打开多个端口。无论哪种情况，记住与每个服务对应的服务名称，有时比记住端口号更方便。

服务文件位于`/usr/lib/firewalld/services/`目录中。你可以使用`firewall-cmd`命令查看它们，就像查看区域列表一样：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-services
 RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync elasticsearch freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kibana klogin kpasswd kshell ldap ldaps libvirt libvirt-tls managesieve mdns mosh mountd ms-wbt mssql mysql nfs nrpe ntp openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster quassel radius rpc-bind rsh rsyncd samba samba-client sane sip sips smtp smtp-submission smtps snmp snmptrap spideroak-lansync squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
[donnie@localhost ~]$
```

在添加更多服务之前，让我们检查一下哪些服务已经启用了：

```
[donnie@localhost ~]$ sudo firewall-cmd --list-services
[sudo] password for donnie: 
ssh dhcpv6-client
[donnie@localhost ~]$
```

这里，ssh 和 dhcpv6-client 就是我们拥有的所有服务。

`dropbox-lansync`服务对我们这些 Dropbox 用户来说非常方便。让我们看看它打开了哪些端口：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-service=dropbox-lansync
 [sudo] password for donnie:
 dropbox-lansync
  ports: 17500/udp 17500/tcp
  protocols:
  source-ports:
  modules:
  destination:
[donnie@localhost ~]$
```

看起来 Dropbox 使用 UDP 和 TCP 的端口`17500`。

现在，假设我们将 Web 服务器设置在 DMZ 中，并将`dmz`区域设为其默认区域：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

正如我们之前看到的，只有 Secure Shell 端口是开放的。让我们修复它，这样用户就能实际访问我们的网站了：

```
[donnie@localhost ~]$ sudo firewall-cmd --add-service=http
 success
[donnie@localhost ~]$
```

当我们再次查看`dmz`区域的信息时，我们会看到以下内容：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh http
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

在这里，我们可以看到`http`服务现在已经允许通过了。但是，当我们在这个`info`命令中添加`--permanent`选项时，看看会发生什么：

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --info-zone=dmz
 dmz
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

哎呀！`http`服务不见了。发生了什么事？

几乎每个命令行更改区域或服务时，你都需要添加`--permanent`选项以使更改在重启后保持生效。但是如果没有`--permanent`选项，更改会立即生效。加上`--permanent`选项后，你需要重新加载防火墙配置，才能使更改生效。为了演示这一点，我将重启虚拟机以去掉`http`服务。

好的，我已经重启了，`http`服务现在已经不见了：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

这次，我将通过一个命令添加两个服务，并指定更改将是永久的：

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --add-service={http,https}
 [sudo] password for donnie:
 success
[donnie@localhost ~]$
```

你可以通过一个命令添加多个服务，但必须用逗号分隔它们，并将整个列表包含在一对花括号内。而且，与我们刚才看到的 nftables 不同，花括号内不能有空格。让我们看看结果：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
 target: default
 icmp-block-inversion: no
 interfaces: enp0s3
 sources:
 services: ssh
 ports:
 protocols:
 masquerade: no
 forward-ports:
 source-ports:
 icmp-blocks:
 rich rules:
[donnie@localhost ~]$
```

由于我们决定将此配置设置为永久生效，但它尚未生效。然而，如果我们在`--info-zone`命令中添加`--permanent`选项，就会看到配置文件确实已经发生了变化：

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --info-zone=dmz
 dmz
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh http https
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

现在，我们需要重新加载配置，以使其生效：

```
[donnie@localhost ~]$ sudo firewall-cmd --reload
 success
[donnie@localhost ~]$
```

现在，如果你再次运行`sudo firewall-cmd --info-zone=dmz`命令，你会看到新的配置已经生效。

要从某个区域中移除一个服务，只需将`--add-service`替换为`--remove-service`。

> 提示：
> 
> > 注意，我们在这些服务命令中从未指定我们正在使用哪个区域。这是因为，如果我们不指定区域，firewalld 会默认认为我们是在使用默认区域。如果你想将服务添加到默认区域以外的区域，只需在命令中添加`--zone=`选项。

### 向 firewalld 区域添加端口

拥有服务文件非常方便，除了并非每个你需要运行的服务都有自己预定义的服务文件。假设你在服务器上安装了 Webmin，它需要端口 `10000/tcp` 开放。快速使用 grep 操作会显示端口 `10000` 并不在我们的预定义服务列表中：

```
donnie@localhost services]$ pwd
 /usr/lib/firewalld/services
[donnie@localhost services]$ grep '10000' *
[donnie@localhost services]$
```

那么，让我们将该端口添加到我们的默认区域，仍然是 `dmz` 区域：

```
donnie@localhost ~]$ sudo firewall-cmd --add-port=10000/tcp
 [sudo] password for donnie:
 success
[donnie@localhost ~]$
```

再次说明，这不是永久性的，因为我们没有包含 `--permanent` 选项。我们重新执行并重新加载：

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --add-port=10000/tcp
 success
[donnie@localhost ~]$ sudo firewall-cmd --reload
 success
[donnie@localhost ~]$
```

你还可以通过将逗号分隔的端口列表放入一对大括号中来一次性添加多个端口，就像我们处理服务一样。（我故意没有包含 `--permanent` 选项，稍后你会明白为什么）：

```
[donnie@localhost ~]$ sudo firewall-cmd --add-port={636/tcp,637/tcp,638/udp}
 success
[donnie@localhost ~]$
```

当然，你也可以通过用 `--remove-port` 替代 `--add-port` 来从区域中删除端口。

如果你不想每次创建永久规则时都输入 `--permanent`，可以省略这个参数。然后，当你完成创建规则后，使用以下命令一次性将所有规则设置为永久：

```
sudo firewall-cmd --runtime-to-permanent
```

现在，让我们把注意力转向控制 ICMP。

### 阻止 ICMP

再次查看默认公共区域的状态：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: ssh dhcpv6-client
  ports: 53/tcp 53/udp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[donnie@localhost ~]$
```

在页面底部，我们可以看到 `icmp-block` 行，旁边没有任何内容。这意味着我们的公共区域允许所有 ICMP 数据包通过。当然，这并不理想，因为我们有些 ICMP 类型的数据包是希望阻止的。在阻止任何东西之前，让我们先看看所有可用的 ICMP 类型：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-icmptypes
[sudo] password for donnie: 
address-unreachable bad-header communication-prohibited destination-unreachable echo-reply echo-request fragmentation-needed host-precedence-violation host-prohibited host-redirect host-unknown host-unreachable ip-header-bad neighbour-advertisement neighbour-solicitation network-prohibited network-redirect network-unknown network-unreachable no-route packet-too-big parameter-problem port-unreachable precedence-cutoff protocol-unreachable redirect required-option-missing router-advertisement router-solicitation source-quench source-route-failed time-exceeded timestamp-reply timestamp-request tos-host-redirect tos-host-unreachable tos-network-redirect tos-network-unreachable ttl-zero-during-reassembly ttl-zero-during-transit unknown-header-type unknown-option
[donnie@localhost ~]$
```

和处理区域及服务一样，我们可以查看不同 ICMP 类型的信息。在这个例子中，我们查看一种 ICMPv4 类型和一种 ICMPv6 类型：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-icmptype=network-redirectnetwork-redirect  destination: ipv4
[donnie@localhost ~]$ sudo firewall-cmd --info-icmptype=neighbour-advertisementneighbour-advertisement  
destination: ipv6
[donnie@localhost ~]$
```

我们已经看到没有阻止任何 ICMP 数据包。我们还可以查看是否有阻止特定的 ICMP 数据包：

```
[donnie@localhost ~]$ sudo firewall-cmd --query-icmp-block=host-redirect
no
[donnie@localhost ~]$
```

我们已经确定，重定向可能是一个坏东西，因为它们可能被恶意利用。所以，让我们阻止主机重定向数据包：

```
[donnie@localhost ~]$ sudo firewall-cmd --add-icmp-block=host-redirect
success
[donnie@localhost ~]$ sudo firewall-cmd --query-icmp-block=host-redirect
yes
[donnie@localhost ~]$
```

现在，让我们检查状态：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: ssh dhcpv6-client
  ports: 53/tcp 53/udp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: host-redirect
  rich rules: 
[donnie@localhost ~]$
```

很棒—它成功了。现在，我们看看是否能通过一个命令同时阻止两种 ICMP 类型：

```
[donnie@localhost ~]$ sudo firewall-cmd --add-icmp-block={host-redirect,network-redirect}
success
[donnie@localhost ~]$ 
```

如之前所述，我们将检查状态：

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: host-redirect network-redirect
  rich rules: 
[donnie@localhost ~]$
```

这也成功了，意味着我们已经实现了“酷”操作。然而，由于我们没有在这些命令中加入 `--permanent`，这些 ICMP 类型的阻止仅在重启前有效。因此，我们将它们设置为永久：

```
[donnie@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
success
[donnie@localhost ~]$
```

这样，我们就实现了更多的“酷”。（当然，我所有的猫已经觉得我很酷了。）

### 使用恐慌模式

你刚刚看到了坏人试图破坏你的系统的证据。你该怎么办？其中一个选择是激活 `panic` 模式，这会切断所有网络通信。

> 我现在可以想象在周六早上的卡通片里，某个卡通人物大喊：“*恐慌模式，启动！*”

要激活 `panic` 模式，使用以下命令：

```
[donnie@localhost ~]$ sudo firewall-cmd --panic-on
[sudo] password for donnie: 
success
[donnie@localhost ~]$
```

当然，如果你是远程登录的，访问会被切断，你需要去本地终端重新连接。要关闭 `panic` 模式，可以使用以下命令：

```
[donnie@localhost ~]$ sudo firewall-cmd --panic-off
[sudo] password for donnie: 
success
[donnie@localhost ~]$
```

如果你是远程登录的，则无需检查`panic`模式的状态。如果它开启，你就无法访问这台机器。但如果你坐在本地控制台上，可能想要检查它。只需执行：

```
[donnie@localhost ~]$ sudo firewall-cmd --query-panic
[sudo] password for donnie: 
no
[donnie@localhost ~]$
```

这就是`panic`模式的全部内容。

### 记录丢弃的数据包

这是另一个省时的技巧，你一定会喜欢。如果你希望每当数据包被阻止时创建日志条目，只需使用`--set-log-denied`选项。在我们这么做之前，先看看它是否已经启用：

```
[donnie@localhost ~]$ sudo firewall-cmd --get-log-denied
[sudo] password for donnie: 
off
[donnie@localhost ~]$
```

它不是开启状态，所以让我们开启它并再次检查状态：

```
[donnie@localhost ~]$ sudo firewall-cmd --set-log-denied=all
success
[donnie@localhost ~]$ sudo firewall-cmd --get-log-denied
all
[donnie@localhost ~]$
```

我们已经设置了记录所有被拒绝的数据包。然而，你可能并不总是希望这样。你的其他选择有`unicast`、`broadcast`和`multicast`。

例如，如果你只想记录被阻止并且指向多播地址的数据包，可以执行如下操作：

```
[donnie@localhost ~]$ sudo firewall-cmd --set-log-denied=multicast
[sudo] password for donnie: 
success
[donnie@localhost ~]$ sudo firewall-cmd --get-log-denied
multicast
[donnie@localhost ~]$
```

到目前为止，我们只是设置了运行时配置，一旦重启机器，这些配置将会消失。为了使其永久生效，我们可以使用我们已经使用过的任何方法。现在，我们只需执行这个操作：

```
[donnie@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
success
[donnie@localhost ~]$
```

与我们在 Debian/Ubuntu 发行版中看到的不同，我们没有专门的`kern.log`文件来记录数据包被拒绝的消息。相反，RHEL 类型的发行版将数据包拒绝消息记录在`/var/log/messages`文件中，这是 RHEL 世界中的主日志文件。已经定义了多个不同的消息标签，这将使得审计丢弃的数据包日志变得更容易。例如，以下是一个消息，告诉我们被阻止的广播数据包：

```
Aug 20 14:57:21 localhost kernel: FINAL_REJECT: IN=enp0s3 OUT= MAC=ff:ff:ff:ff:ff:ff:00:1f:29:02:0d:5f:08:00 SRC=192.168.0.225 DST=255.255.255.255 LEN=140 TOS=0x00 PREC=0x00
 TTL=64 ID=62867 DF PROTO=UDP SPT=21327 DPT=21327 LEN=120
```

这个标签是`FINAL_REJECT`，它告诉我们这条消息是由我们输入链末尾的通用`REJECT`规则创建的。`DST=255.255.255.255`部分告诉我们这是一个广播消息。

这是另一个示例，我对这台机器进行了 Nmap NULL 扫描：

```
sudo nmap -sN 192.168.0.8
Aug 20 15:06:15 localhost kernel: STATE_INVALID_DROP: IN=enp0s3 OUT= MAC=08:00:27:10:66:1c:00:1f:29:02:0d:5f:08:00 SRC=192.168.0.225 DST=192.168.0.8 LEN=40 TOS=0x00 PREC=0x00 TTL=42 ID=27451 PROTO=TCP SPT=46294 DPT=23 WINDOW=1024 RES=0x00 URGP=0
```

在这种情况下，我触发了阻止`INVALID`数据包的规则，如`STATE_INVALID_DROP`标签所示。

那么，现在你可能会说，*等一下。我们刚刚测试的这两条规则在我们查看过的 firewalld 配置文件中都没有找到。这是怎么回事？* 你说得对。这些默认的、预配置的规则的位置显然是 Red Hat 的人希望对我们隐藏的。不过，在接下来的专门针对 RHEL/CentOS 7 和 RHEL/AlmaLinux 8 以及 9 的部分中，我们将揭示他们的秘密，因为我能告诉你这些规则在哪里。

### 使用 firewalld 丰富语言规则

到目前为止，我们所看的内容可能就是一般使用场景下所需的全部，但如果需要更细致的控制，你会想了解**丰富语言规则**。（没错，这就是它们的名称。）

与 iptables 规则相比，丰富语言规则稍微不那么晦涩，更接近普通英语。因此，如果你是编写防火墙规则的新手，可能会觉得丰富语言更容易学习。另一方面，如果你已经习惯了编写 iptables 规则，可能会觉得丰富语言的某些元素有点怪异。让我们看一个例子：

```
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="200.192.0.0/24" service name="http" drop'
```

在这里，我们添加了一个丰富规则，阻止来自整个 IPv4 地址地理块的网站访问。请注意，整个规则被一对单引号包围，每个参数的赋值被一对双引号包围。通过这个规则，我们声明我们正在使用 IPv4，并且我们希望静默阻止`http`端口接受来自`200.192.0.0/24`网络的数据包。我在这里使用了`--permanent`选项，因为如果不使用它，AlmaLinux 9 会有点怪异。让我们看看应用这个新规则后我们的区域是什么样的：

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --info-zone=dmz
  dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh http https
  ports: 10000/tcp 636/tcp 637/tcp 638/udp
. . .
. . .
  rich rules:
  rule family="ipv4" source address="200.192.0.0/24" service name="http"
 drop
 [donnie@localhost ~]$
```

丰富规则显示在底部。

你可以通过将`family="ipv4"`替换为`family="ipv6"`并提供相应的 IPv6 地址范围，轻松编写 IPv6 的规则。

有些规则是通用的，适用于 IPv4 或 IPv6。例如，假设我们希望记录关于**网络时间协议**（**NTP**）数据包的消息，适用于 IPv4 和 IPv6，并且我们希望每分钟不超过记录一条消息。创建该规则的命令如下：

```
sudo firewall-cmd --add-rich-rule='rule service name="ntp" audit limit value="1/m" accept'
```

当然，firewalld 丰富语言规则的内容远不止我们在这里呈现的内容。但现在，您已经掌握了基本知识。欲了解更多信息，请查阅 man 页面：

```
man firewalld.richlanguage
```

> 如果你访问 Red Hat Enterprise Linux 8 的官方文档页面，你会发现没有提到丰富规则。然而，我刚刚在一台 RHEL 8 类型的机器和一台 RHEL 9 类型的机器上测试了它们，它们工作得很好。
> 
> > 要阅读丰富规则的相关内容，您需要访问 Red Hat Enterprise Linux 7 的文档页面。这里的内容同样适用于 RHEL 8/9。但即便如此，那里也没有太多详细内容。要了解更多信息，请查阅 RHEL/CentOS 7 或 RHEL/CentOS 8，或者 RHEL/AlmaLinux 9 的 man 页面。

要使规则永久生效，只需使用我们已经讨论过的任何方法。当您这样做时，规则将出现在默认区域的`.xml`文件中。就我而言，默认区域仍然设置为公共区域。所以，让我们看看`/etc/firewalld/zones/public.xml`文件：

```
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <rule family="ipv4">
    <source address="192.168.0.225"/>
    <service name="http"/>
    <drop/>
  </rule>
</zone>
```

我们的丰富规则显示在文件底部的`rule family`块中。

现在我们已经覆盖了 RHEL/CentOS 7 与 RHEL/CentOS/AlmaLinux 8/9 版本的 firewalld 之间的共同点，接下来我们来看看每个版本的特有内容。

### 查看 RHEL/CentOS 7 firewalld 中的 iptables 规则

RHEL 7 及其衍生版本使用 iptables 引擎作为 firewalld 的后端。在 firewalld 启用时，不能使用常规的 iptables 命令创建规则。然而，每次使用`firewall-cmd`命令创建规则时，iptables 后端会创建相应的 iptables 规则并将其插入到正确的位置。您可以通过`iptables -L`查看活动规则。以下是非常长的输出的第一部分：

```
[donnie@localhost ~]$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere            
INPUT_direct  all  --  anywhere             anywhere            
INPUT_ZONES_SOURCE  all  --  anywhere             anywhere            
INPUT_ZONES  all  --  anywhere             anywhere            
DROP       all  --  anywhere             anywhere             ctstate INVALID
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited
```

就像在 Ubuntu 上的 ufw 一样，很多配置已经为我们完成了。在 `INPUT` 链的顶部，我们可以看到连接状态规则和阻止无效数据包的规则已经存在。该链的默认策略是 `ACCEPT`，但是链的最后一条规则设置为 `REJECT` 所有没有明确允许的内容。在这些规则之间，我们可以看到将其他数据包指向其他链进行处理的规则。现在，让我们看下一个部分：

```
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh ctstate NEW
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain ctstate NEW
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain ctstate NEW
Chain IN_public_deny (1 references)
target     prot opt source               destination         
REJECT     icmp --  anywhere             anywhere             icmp host-redirect reject-with icmp-host-prohibited
```

在很长的输出的底部，我们可以看到 `IN_public_allow` 链，其中包含我们为开放防火墙端口而创建的规则。紧接着是 `IN_public_deny` 链，里面包含了用于阻止不需要的 ICMP 类型的 `REJECT` 规则。在 `INPUT` 链和 `IN_public_deny` 链中，`REJECT` 规则会返回 ICMP 消息，通知发送者数据包已被阻止。

现在，请记住，我们没有展示很多 `IPTABLES -L` 的输出内容。所以，自己看看输出，看看里面有什么。当你查看时，你可能会问自己，*这些默认规则存储在哪里？为什么我在* `/etc/firewalld/` *目录下没有看到它们？*

为了回答这个问题，我进行了相当广泛的调查。由于某些真正奇怪的原因，Red Hat 的人们完全没有文档化这一内容。我最终在`/usr/lib/python2.7/site-packages/firewall/core/`目录下找到了答案。在这里，有一组 Python 脚本用于设置初始的默认防火墙：

```
[donnie@localhost core]$ ls
base.py fw_config.pyc fw_helper.pyo fw_ipset.py fw_policies.pyc fw_service.pyo fw_zone.py icmp.pyc ipset.pyc logger.pyo rich.py base.pyc fw_config.pyo fw_icmptype.py fw_ipset.pyc fw_policies.pyo fw_test.py fw_zone.pyc icmp.pyo ipset.pyo modules.py rich.pyc base.pyo fw_direct.py fw_icmptype.pyc fw_ipset.pyo fw.py fw_test.pyc fw_zone.pyo __init__.py ipXtables.py modules.pyc rich.pyo ebtables.py fw_direct.pyc fw_icmptype.pyo fw_nm.py fw.pyc fw_test.pyo helper.py __init__.pyc ipXtables.pyc modules.pyo watcher.py ebtables.pyc fw_direct.pyo fw_ifcfg.py fw_nm.pyc fw.pyo fw_transaction.py helper.pyc __init__.pyo ipXtables.pyo prog.py watcher.pyc ebtables.pyo fw_helper.py fw_ifcfg.pyc fw_nm.pyo fw_service.py fw_transaction.pyc helper.pyo io logger.py prog.pyc watcher.pyo fw_config.py fw_helper.pyc fw_ifcfg.pyo fw_policies.py fw_service.pyc fw_transaction.pyo icmp.py ipset.py logger.pyc prog.pyo
[donnie@localhost core]$
```

执行大部分工作的脚本是 `ipXtables.py` 脚本。如果你查看它，你会发现其中的 iptables 命令列表与 `iptables -L` 输出相匹配。

### 在 RHEL/CentOS 7 上创建直接规则

正如我们所看到的，每当我们在 RHEL/CentOS 7 上使用普通的 `firewall-cmd` 命令时，firewalld 会自动将这些命令转换为 iptables 规则，并将它们插入到正确的位置。（或者，如果你发出了删除命令，它会删除规则。）然而，有一些事情我们无法通过普通的 `firewall-cmd` 命令来做。例如，我们无法使用普通的 `firewall-cmd` 命令将规则放置在特定的 iptables 链或表中。要做这样的事情，我们需要使用直接配置命令。

`firewalld.direct` 手册页以及 Red Hat 网站上的文档都警告你，只有在其他方法都不起作用时，才应使用直接配置。这是因为，与普通的 `firewall-cmd` 命令不同，直接命令不会自动将新规则放入正确的位置，从而确保一切正常工作。使用直接命令时，如果将规则放错位置，可能会导致整个防火墙崩溃。

在上一节的示例输出中，在默认的规则集中，你会看到在过滤器表的`INPUT`链中有一条规则阻止无效数据包。在*第四章*中*使用防火墙保护服务器 - 第一部分*的*阻止无效数据包与 iptables*部分中，你看到这条规则漏掉了一些类型的无效数据包。所以，我们想添加第二条规则来阻止第一条规则漏掉的内容。我们还希望将这些规则放入 mangle 表的`PREROUTING`链中，以提高防火墙性能。为此，我们需要创建几条直接规则。（如果你熟悉常规的 iptables 语法，这并不难。）那么，让我们开始吧。

首先，让我们验证一下是否没有任何有效的直接规则，方法如下：

```
sudo firewall-cmd --direct --get-rules ipv4 mangle PREROUTING
sudo firewall-cmd --direct --get-rules ipv6 mangle PREROUTING
```

你应该不会看到任何命令输出。现在，让我们通过以下四个命令，为 IPv4 和 IPv6 添加我们的两条新规则：

```
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
```

`direct`命令的语法与正常的 iptables 命令非常相似。所以，我不会重复在 iptables 部分中已经介绍过的解释。不过，我确实想指出在每个命令中的`PREROUTING`后面跟着的`0`和`1`。它们代表规则的优先级。数字越低，优先级越高，规则在链中的位置越靠前。因此，优先级为`0`的规则是各自链中的第一条规则，而优先级为`1`的规则是各自链中的第二条规则。如果你给每个规则分配相同的优先级，不能保证每次重启时规则的顺序保持不变。所以，确保为每个规则分配不同的优先级。

现在，让我们验证一下我们的规则是否生效：

```
[donnie@localhost ~]$ sudo firewall-cmd --direct --get-rules ipv4 mangle PREROUTING
0 -m conntrack --ctstate INVALID -j DROP
1 -p tcp '!' --syn -m conntrack --ctstate NEW -j DROP
[donnie@localhost ~]$ sudo firewall-cmd --direct --get-rules ipv6 mangle PREROUTING
0 -m conntrack --ctstate INVALID -j DROP
1 -p tcp '!' --syn -m conntrack --ctstate NEW -j DROP
[donnie@localhost ~]$
```

我们可以看到它们是有效的。当你使用`iptables -t mangle -L`命令和`ip6tables -t mangle -L`命令时，你会看到这些规则出现在`PREROUTING_direct`链中。（由于两条命令的输出相同，我只显示一次输出。）

```
. . .
. . .
Chain PREROUTING_direct (1 references)
target prot opt source destination 
DROP all -- anywhere anywhere ctstate INVALID
DROP tcp -- anywhere anywhere tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW
. . .
. . .
```

为了证明它有效，我们可以对虚拟机执行一些 Nmap 扫描，就像我在*第四章*中*使用防火墙保护服务器 - 第一部分*的*阻止无效数据包与 iptables*部分中向你展示的那样。（如果你不记得怎么做，不用担心，接下来的动手实验中你会看到操作步骤。）然后，我们可以使用`sudo iptables -t mangle -L -v`和`sudo ip6tables -t mangle -L -v`来查看这两条规则阻止的包和字节。

我们在这些命令中没有使用`--permanent`选项，所以它们还不是永久性的。现在让我们将它们设置为永久：

```
[donnie@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
[sudo] password for donnie: 
success
[donnie@localhost ~]$
```

现在，让我们看看`/etc/firewalld/`目录。在这里，你会看到一个之前没有的`direct.xml`文件：

```
[donnie@localhost ~]$ sudo ls -l /etc/firewalld
total 20
-rw-r--r--. 1 root root  532 Aug 26 13:17 direct.xml
. . .
. . .
[donnie@localhost ~]$
```

打开文件，你会看到新的规则：

```
<?xml version="1.0" encoding="utf-8"?>
<direct>
  <rule priority="0" table="mangle" ipv="ipv4" chain="PREROUTING">-m conntrack --ctstate INVALID -j DROP</rule>
  <rule priority="1" table="mangle" ipv="ipv4" chain="PREROUTING">-p tcp '!' --syn -m conntrack --ctstate NEW -j DROP</rule>
  <rule priority="0" table="mangle" ipv="ipv6" chain="PREROUTING">-m conntrack --ctstate INVALID -j DROP</rule>
  <rule priority="1" table="mangle" ipv="ipv6" chain="PREROUTING">-p tcp '!' --syn -m conntrack --ctstate NEW -j DROP</rule>
</direct>
```

官方的 Red Hat 7 文档页面确实覆盖了直接规则，但只做了简要介绍。如需详细信息，请参阅`firewalld.direct`手册页面。

### 查看 RHEL/AlmaLinux 8 和 9 中的 nftables 规则以及 firewalld

RHEL 8/9 及其衍生版本使用 nftables 作为默认的 firewalld 后端。每次你使用 `firewall-cmd` 命令创建规则时，适当的 nftables 规则会被创建并插入到正确的位置。为了查看当前生效的规则集，我们将使用与在 Ubuntu 上使用 nftables 时相同的 nft 命令：

```
[donnie@localhost ~]$ sudo nft list ruleset
. . .
. . .
table ip firewalld {
    chain nat_PREROUTING {
        type nat hook prerouting priority -90; policy accept;
        jump nat_PREROUTING_ZONES_SOURCE
        jump nat_PREROUTING_ZONES
    }
    chain nat_PREROUTING_ZONES_SOURCE {
    }
. . .
. . .
[donnie@localhost ~]$
```

我们再次看到了一长串默认的、预配置的防火墙规则。（要查看完整的列表，请自行运行命令。）你会在 RHEL 8 类型机器的 `/usr/lib/python3.6/site-packages/firewall/core/nftables.py` 脚本中找到这些默认规则，在 RHEL 9 类型机器中则是在 `/usr/lib/python3.9/site-packages/firewall/core/nftables.py` 脚本中。每次启动机器时，这个脚本都会运行。

### 在 RHEL/AlmaLinux firewalld 中创建直接规则

好吧，事情开始变得相当奇怪了。即使直接规则命令创建了 iptables 规则，而 RHEL 8/9 发行版使用 nftables 作为 firewalld 后端，你仍然可以创建直接规则。只需像在 *RHEL/CentOS 7 firewalld 中创建直接规则* 部分那样创建并验证它们。显然，firewalld 允许这些 iptables 规则与 nftables 规则和平共存。然而，如果你需要在生产系统中这样做，务必在投入生产之前彻底测试你的设置。

在 Red Hat 8/9 文档中没有关于此的内容，但如果你想了解更多，可以查看 `firewalld.direct` 的手册页。

#### firewalld 命令操作实验

完成此实验后，你将练习一些基本的 firewalld 命令：

1.  登录到你的 CentOS 7 虚拟机或任一 AlmaLinux 虚拟机，并运行以下命令。观察每次执行后的输出：

```
 sudo firewall-cmd --get-zones
 sudo firewall-cmd --get-default-zone
 sudo firewall-cmd --get-active-zones
```

1.  简要查看处理 `firewalld.zones` 的手册页：

```
 man firewalld.zones
 man firewalld.zone
```

（是的，确实有两个。一个解释了区域配置文件，另一个解释了区域本身。）

1.  查看所有可用区域的配置详情：

```
sudo firewall-cmd --list-all-zones
```

1.  查看预定义服务的列表。然后，查看 `dropbox-lansync` 服务的相关信息：

```
 sudo firewall-cmd --get-services
 sudo firewall-cmd --info-service=dropbox-lansync
```

1.  将默认区域设置为 `dmz`。查看关于 `zone` 的信息，添加 `http` 和 `https` 服务，然后再次查看 `zone` 信息：

```
 sudo firewall-cmd --set-default-zone=dmz
 sudo firewall-cmd --permanent --add-service={http,https}
 sudo firewall-cmd --info-zone=dmz
 sudo firewall-cmd --permanent --info-zone=dmz
```

1.  重新加载 **防火墙** 配置并再次查看 `zone` 信息。同时，查看被允许的服务列表：

```
 sudo firewall-cmd --reload
 sudo firewall-cmd --info-zone=dmz
 sudo firewall-cmd --list-services
```

1.  永久打开端口 `10000/tcp` 并查看结果：

```
 sudo firewall-cmd --permanent --add-port=10000/tcp
 sudo firewall-cmd --list-ports
 sudo firewall-cmd --reload
 sudo firewall-cmd --list-ports
 sudo firewall-cmd --info-zone=dmz
```

1.  移除你刚刚添加的端口：

```
 sudo firewall-cmd --permanent --remove-port=10000/tcp
 sudo firewall-cmd --reload
 sudo firewall-cmd --list-ports
 sudo firewall-cmd --info-zone=dmz
```

1.  添加一条丰富语言规则来阻止一个地理范围的 IPv4 地址：

```
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="200.192.0.0/24" service name="http" drop'
```

1.  阻止 `host-redirect` 和 `network-redirect` ICMP 类型：

```
sudo firewall-cmd --permanent --add-icmp-block={host-redirect,network-redirect}
```

1.  添加指令以记录所有被丢弃的报文：

```
sudo firewall-cmd --set-log-denied=all
```

1.  查看 `runtime` 和 `permanent` 配置，并注意它们之间的差异：

```
sudo firewall-cmd --info-zone=dmz
sudo firewall-cmd --info-zone=dmz --permanent
```

1.  将 `runtime` 配置变为 `permanent` 并验证其生效：

```
sudo firewall-cmd --runtime-to-permanent
sudo firewall-cmd --info-zone=dmz --permanent
```

1.  在 CentOS 7 上，通过以下方式查看有效的防火墙规则完整列表：

```
sudo iptables -L
```

1.  在 AlmaLinux 8 或 9 上，通过以下命令查看所有有效的防火墙规则：

```
sudo nft list ruleset
```

1.  创建 `direct` 规则，以阻止来自 mangle 表的 `PREROUTING` 链的无效数据包：

```
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
```

1.  验证 **规则** 是否生效，并将其设置为 **永久**：

```
sudo firewall-cmd --direct --get-rules ipv4 mangle PREROUTING
sudo firewall-cmd --direct --get-rules ipv6 mangle PREROUTING
sudo firewall-cmd --runtime-to-permanent
```

1.  查看你刚刚创建的 `direct.xml` 文件的内容：

```
sudo less /etc/firewalld/direct.xml
```

1.  对虚拟机执行 XMAS Nmap 扫描，支持 IPv4 和 IPv6。然后，观察哪个规则被扫描触发：

```
sudo nmap -sX ipv4_address_of_Test-VM
sudo nmap -6 -sX ipv6_address_of_Test-VM
sudo iptables -t mangle -L -v
sudo ip6tables -t mangle -L -v
```

1.  重复 *第 19 步*，但这次使用 Windows 扫描：

```
sudo nmap -sW ipv4_address_of_Test-VM
sudo nmap -6 -sW ipv6_address_of_Test-VM
sudo iptables -t mangle -L -v
sudo ip6tables -t mangle -L -v
```

1.  查看 firewalld 的主要页面列表：

```
apropos firewall
```

实验到此结束，恭喜你！

## 总结

在本章中，我们介绍了两种辅助工具，可以简化使用 iptables 或 nftables。我们首先介绍了 ufw，它适用于 Debian 和 Ubuntu 系列的系统。接着，我们介绍了 firewalld，虽然最初只在 Red Hat 系列发行版中使用，但现在也可以在 Ubuntu 仓库中找到，并且已经预安装并在 SUSE 系统上启用。

在我所分配的空间中，我展示了如何使用这些技术设置单主机保护的基本知识。我还展示了一些 firewalld 的内部细节，这些信息你在任何地方，包括官方的 Red Hat 文档中，都找不到。

在下一章，我们将讨论各种加密技术，它们能帮助你保护数据隐私。到时见。

## 问题

1.  RHEL 7 系列和 RHEL 8/9 系列的 firewalld 有什么主要区别？

1.  firewalld 以哪种格式存储规则？

    1.  `.txt`

    1.  `.config`

    1.  `.html`

    1.  `.xml`

1.  以下哪个命令可以列出系统上所有的 firewalld 区域？

    1.  sudo firewalld --get-zones

    1.  sudo firewall-cmd --list-zones

    1.  sudo firewall-cmd --get-zones

    1.  sudo firewalld --list-zones

1.  使用 ufw，你所需的所有操作都可以通过 ufw 工具完成。

    1.  真

    1.  假

1.  你的系统已经安装了 firewalld，并且你需要打开端口 `10000/tcp`。你会使用哪个命令？

    1.  sudo firewall-cmd --add-port=10000/tcp

    1.  sudo firewall-cmd --add-port=10000

    1.  sudo firewalld --add-port=10000

    1.  sudo firewalld --add-port=10000/tcp

1.  以下哪个 ufw 命令可以用来打开默认的安全外壳端口？

    1.  sudo ufw allow 22

    1.  sudo ufw permit 22

    1.  sudo ufw allow 22/tcp

    1.  sudo ufw permit 22/tcp

## 进一步阅读

+   使用 ufw 限制速率：[`45squared.com/rate-limiting-with-ufw/`](https://45squared.com/rate-limiting-with-ufw/)

+   RHEL 7 的 firewalld 文档：[`access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls)

+   RHEL 8 的 firewalld 文档：[`access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/using-and-configuring-firewalls_securing-networks`](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/using-and-configuring-firewalls_securing-networks)  

+   firewalld 主页：[`firewalld.org/`](https://firewalld.org/)  

+   UFW 社区帮助维基：[`help.ubuntu.com/community/UFW`](https://help.ubuntu.com/community/UFW)

+   如何在 Ubuntu 18.04 上使用 UFW 设置 Linux 防火墙：[`linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-ubuntu-18-04/`](https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-ubuntu-18-04/)  

## 答案  

1.  RHEL 7 发行版使用 iptables 作为 firewalld 的后端，而 RHEL 8/9 发行版使用 nftables 作为 firewalld 的后端。  

1.  D  

1.  C  

1.  B  

1.  A  

1.  C  
