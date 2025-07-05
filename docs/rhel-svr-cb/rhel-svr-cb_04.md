# 第四章：配置你的新系统

这是我们将在本章中涵盖的所有配方概览：

+   `systemd`服务和设置运行级别

+   启动和停止`systemd`服务

+   配置`systemd`日志以实现持久化

+   使用`journalctl`监控服务

+   配置`logrotate`

+   管理时间

+   配置你的启动环境

+   配置`smtp`

# 介绍

一旦你的系统安装完成并且网络配置好，就可以开始配置其他内容了。

RHEL 7 配备了`systemd init`守护进程，负责处理守护进程或服务的管理等任务，取代了旧的 SysV（UNIX System V）初始化系统。

它的主要优点是自动处理依赖关系、并行启动服务，并能够监控启动的服务并重启崩溃的服务。

要深入了解`systemd`及其内部工作原理，请访问[`n0where.net/understanding-systemd`](https://n0where.net/understanding-systemd)。

# `systemd`服务和设置运行级别

`systemd`服务不像 SysV 或 Upstart 那样使用运行级别。`systemd`的替代方案叫做目标（targets）。它们的目的是通过依赖链来组织一组`systemd`单元（不仅仅是服务，还包括套接字、设备等）。

## 如何操作……

使用`systemd`管理目标非常简单，如下所示：

1.  列出所有目标单元，如下所示：

    ```
    ~]# systemctl list-unit-files --type target
    UNIT FILE                 STATE 
    anaconda.target           static 
    basic.target              static 
    bluetooth.target          static 
    cryptsetup.target         static 
    ctrl-alt-del.target       disabled
    default.target            enabled
    ...

    sysinit.target            static 
    system-update.target      static 
    time-sync.target          static 
    timers.target             static 
    umount.target             static 

    58 unit files listed.
    ~]#

    ```

    此列表显示所有可用的目标单元，并提供目标是否已启用的信息。

1.  现在，显示当前加载的目标单元。

    `systemd`目标可以像 SysV 运行级别一样串联使用，因此你不仅会看到一个目标，还会看到一大堆目标，如下所示：

    ```
    ~]# systemctl list-units --type target
    UNIT                  LOAD   ACTIVE SUB    DESCRIPTION
    basic.target          loaded active active Basic System
    cryptsetup.target     loaded active active Encrypted Volumes
    getty.target          loaded active active Login Prompts
    local-fs-pre.target   loaded active active Local File Systems (Pre)
    local-fs.target       loaded active active Local File Systems
    multi-user.target     loaded active active Multi-User System
    network-online.target loaded active active Network is Online
    network.target        loaded active active Network
    nfs-client.target     loaded active active NFS client services
    paths.target          loaded active active Paths
    remote-fs-pre.target  loaded active active Remote File Systems (Pre)
    remote-fs.target      loaded active active Remote File Systems
    slices.target         loaded active active Slices
    sockets.target        loaded active active Sockets
    swap.target           loaded active active Swap
    sysinit.target        loaded active active System Initialization
    time-sync.target      loaded active active System Time Synchronized
    timers.target         loaded active active Timers

    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.

    18 loaded units listed. Pass --all to see loaded but inactive units, too.
    To show all installed unit files use 'systemctl list-unit-files'.
    ~]#

    ```

1.  接下来，通过运行以下命令更改默认的`systemd`目标：

    ```
    ~]# systemctl set-default graphical.target
    rm '/etc/systemd/system/default.target'
    ln -s '/usr/lib/systemd/system/graphical.target' '/etc/systemd/system/default.target'
    ~]#

    ```

## 还有更多内容……

有时，你可能希望像以前使用 runlevel 或 telinit 那样动态地更改目标。在`systemd`中，可以通过以下方式实现：

```
~]# systemctl isolate <target name>

```

这里是一个例子：

```
~]# systemctl isolate graphical.target

```

让我们通过以下表格概览以前的运行级别与`systemd`目标之间的关系：

| 运行级别 | 目标单元 | 描述 |
| --- | --- | --- |
| `0` | `runlevel0.target` 或 `poweroff.target` | 用于关闭并关闭系统电源 |
| `1` | `runlevel1.target` 或 `rescue.target` | 用于进入抢救模式外壳 |
| `2` | `runlevel2.target` 或 `multi-user.target` | 用于设置命令行多用户系统 |
| `3` | `runlevel3.target` 或 `multi-user.target` | 用于设置命令行多用户系统 |
| `4` | `runlevel4.target` 或 `multi-user.target` | 用于设置命令行多用户系统 |
| `5` | `runlevel5.target` 或 `graphical.target` | 用于设置图形化多用户系统 |
| `6` | `runlevel6.target` 或 `reboot.target` | 用于重启系统 |

## 另见

要了解有关 RHEL 7 和 `systemd` 目标的更多详细信息，请参考以下链接：[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Targets.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Targets.html)

# 启动和停止 systemd 服务

尽管这个示例使用的是服务的基本名称，但它们也可以通过完整的文件名来表示。例如，`sshd`可以替换为`sshd.service`。

## 如何操作…

成功启动或停止`systemd`服务需要执行以下步骤：

1.  列出所有可用的`systemd`服务，如下所示：

    ```
    ~]# systemctl list-unit-files --type service
    UNIT FILE                                   STATE 
    atd.service                                 enabled
    auditd.service                              enabled
    auth-rpcgss-module.service                  static 
    autovt@.service                             disabled
    avahi-daemon.service                        disabled
    blk-availability.service                    disabled
    brandbot.service                            static 

    ...

    systemd-udev-trigger.service                static 
    systemd-udevd.service                       static 
    systemd-update-utmp-runlevel.service        static 
    systemd-update-utmp.service                 static 
    systemd-user-sessions.service               static 
    systemd-vconsole-setup.service              static 
    tcsd.service                                disabled
    teamd@.service                              static 
    tuned.service                               enabled
    wpa_supplicant.service                      disabled
    xinetd.service                              enabled

    161 unit files listed.

    ```

    这将显示所有可用的服务单元，后面跟着关于服务是否启用的信息。

1.  现在，列出所有已加载的`systemd`服务及其状态，如下所示：

    ```
    ~]# systemctl list-units --type service --all
    UNIT                        LOAD   ACTIVE   SUB     DESCRIPTION
    atd.service                 loaded active   running Job spooling tools
    auditd.service              loaded active   running Security Auditing Service
    auth-rpcgss-module.service  loaded inactive dead    Kernel Module supporting RPC
    brandbot.service            loaded inactive dead    Flexible Branding Service
    cpupower.service            loaded inactive dead    Configure CPU power related
    crond.service               loaded active   running Command Scheduler
    cups.service                loaded inactive dead    CUPS Printing Service
    dbus.service                loaded active   running D-Bus System Message Bus
    ...

    systemd-...es-setup.service loaded active   exited  Create Volatile Files and Di
    systemd-...-trigger.service loaded active   exited  udev Coldplug all Devices
    systemd-udevd.service       loaded active   running udev Kernel Device Manager
    systemd-update-utmp.service loaded active   exited  Update UTMP about System Reb
    systemd-...sessions.service loaded active   exited  Permit User Sessions
    systemd-...le-setup.service loaded active   exited  Setup Virtual Console
    tuned.service               loaded active   running Dynamic System Tuning Daemon
    xinetd.service              loaded active   running Xinetd A Powerful Replacemen
    LOAD   = Reflects whether the unit definition was properly loaded.
    ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
    SUB    = The low-level unit activation state, values depend on unit type.

    103 loaded units listed.
    To show all installed unit files use 'systemctl list-unit-files'.
    ~]#

    ```

1.  接下来，获取服务的状态。

    要获取特定服务的状态，执行以下命令，将 `<service>` 替换为服务的名称：

    ```
    ~]# systemctl status <service>

    ```

    这里是一个示例：

    ```
    ~]# systemctl status sshd
    sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled)
     Active: active (running) since Fri 2015-07-17 09:13:55 CEST; 1 weeks 0 days ago
     Main PID: 11880 (sshd)
     CGroup: /system.slice/sshd.service
     └─11880 /usr/sbin/sshd -D

    Jul 22 12:07:31 rhel7.mydomain.lan sshd[10340]: Accepted publickey for root...
    Jul 22 12:12:29 rhel7.mydomain.lan sshd[10459]: Accepted publickey for root...
    Jul 22 12:13:33 rhel7.mydomain.lan sshd[10473]: Accepted publickey for root...
    Jul 24 21:27:24 rhel7.mydomain.lan sshd[28089]: Accepted publickey for root...
    Hint: Some lines were ellipsized, use -l to show in full.
    ~]#

    ```

1.  现在，启动和停止`systemd`服务。

    要停止`systemd`服务，执行以下命令，将 `<service>` 替换为服务的名称：

    ```
    ~]# systemctl stop <service>

    ```

    这里是一个示例：

    ```
    ~]# systemctl stop sshd

    ```

    要启动`systemd`服务，执行以下命令，将 `<service>` 替换为服务的名称：

    ```
    ~]# systemctl start <service>

    ```

    这里是一个示例：

    ```
    ~]# systemctl start sshd

    ```

1.  接下来，启用和禁用`systemd`服务。

    要启用`systemd`服务，执行以下命令，将 `<service>` 替换为服务的名称：

    ```
    ~]# systemctl enable <service>

    ```

    这里是一个示例：

    ```
    ~]# systemctl enable sshd
    ln -s '/usr/lib/systemd/system/sshd.service' '/etc/systemd/system/multi-user.target.wants/sshd.service'
    ~]#

    ```

    要禁用`systemd`服务，执行以下命令，将 `<service>` 替换为服务的名称：

    ```
    ~]# systemctl disable <service>

    ```

    这里是一个示例：

    ```
    ~]# systemctl disable sshd
    rm '/etc/systemd/system/multi-user.target.wants/sshd.service'
    ~]#

    ```

1.  现在，配置服务在崩溃时重新启动。

    如果`ntpd`服务在 1 分钟后崩溃，让我们使其重新启动。

    1.  首先，创建目录，如下所示：`/etc/systemd/system/ntpd.service.d`。

        ```
        ~]# mkdir -p /etc/systemd/system/ntpd.service.d

        ```

    1.  在该目录中创建一个名为`restart.conf`的新文件，并将以下内容添加到文件中：

        ```
        [Service]
        Restart=on-failure
        RestartSec=60s
        ```

    1.  接下来，使用以下命令重新加载单元文件并重新创建依赖关系树：

        ```
        ~]# systemctl daemon-reload

        ```

    1.  最后，执行以下命令重启`ntpd`服务：

        ```
        ~]# systemctl restart ntpd

        ```

## 还有更多…

当请求服务的状态时，如果以 `root` 身份执行，还会显示最新的日志条目。

以下表格可以查看服务状态信息：

| 字段 | 描述 |
| --- | --- |
| `Loaded` | 提供关于服务是否已加载和启用的信息，还包括服务文件的绝对路径。 |
| `Active` | 提供服务是否正在运行的信息，后面跟着服务启动的时间。 |
| `Main PID` | 提供相应服务的 PID，后面跟着服务的名称。 |
| `Status` | 提供有关相应服务的信息。 |
| `Process` | 提供关于相关进程的信息。 |
| `Cgroup` | 提供关于相关控制组的信息。 |

在某些（罕见）情况下，您可能希望防止某个服务启动，无论是手动启动还是由另一个服务启动；可以通过以下方式屏蔽该服务：

```
~]# systemctl mask <service>

```

要取消屏蔽，请执行以下命令：

```
~]# systemctl unmask <service>

```

修改服务单元文件时（这不仅限于服务），最佳实践是复制原始服务文件，该文件位于 `/lib/systemd/system`，然后将其放到 `/etc/systemd/service`。或者，您可以在 `/etc/systemd/service` 中创建一个以 `.d` 结尾的目录，在其中创建包含您希望添加或更改的指令的 `conf` 文件，如上例所示。后者的优点是，您无需跟踪原始服务文件的更改，因为它会与 `service.d` 目录中的内容“同步”更新。

## 另见

有关管理 `systemd` 服务的更多信息，请访问 [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html)。

# 配置 `systemd` 日志以实现持久化

默认情况下，日志不会存储在磁盘上，只会保存在内存或 `/run/log/journal` 目录中。这对于最近的日志历史（使用日志）是足够的，但如果您决定只使用日志而不使用其他 `syslog` 解决方案，则不足以进行长期日志保留。

## 如何操作…

配置 `journald` 保留比内存允许的更多日志非常简单，如下所示：

1.  使用 root 权限通过以下命令打开 `/etc/systemd/journald.conf` 文件，使用您喜欢的文本编辑器：

    ```
    ~]# vim /etc/systemd/journald.conf

    ```

1.  确保包含 `Storage` 的行被注释掉或设置为 `auto` 或 `persistent`，并保存，示例如下：

    ```
    Storage=auto
    ```

1.  如果选择 `auto`，日志目录需要手动创建。以下命令可以帮助完成此操作：

    ```
    ~]# mkdir -p /var/log/journal

    ```

1.  现在，执行以下命令重启日志服务：

    ```
    ~]# systemctl restart systemd-journald

    ```

## 还有更多内容…

还有许多其他选项可以为日志守护进程设置。

默认情况下，`journald` 存储的所有数据都是压缩的，但您可以通过设置 `Compress=no` 来禁用此功能。

建议通过指定最大保留时间（`MaxRetentionSec`）、全局最大大小使用（`SystemMaxUse`）或每个文件的最大大小使用（`SystemMaxFileSize`）来限制日志文件的大小。

## 另见

有关在 RHEL 7 中使用日志的更多信息，请访问 [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html)。

查看 *journald (5)* 的手册页，了解更多可以配置的内容。

# 使用 `journalctl` 监控服务

Systemd 的日志有一个额外的优点，它的控制选项允许你轻松缩小范围，只查看由特定服务生成的消息。

## 如何操作……

这是你在此食谱中需要执行的步骤：

1.  首先，显示系统生成的所有消息。

    这将显示系统上生成的所有消息；运行以下命令：

    ```
    ~]# journalctl
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:30:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpuset
    ...
    ~]#

    ```

1.  现在，显示所有与系统相关的消息。

    此命令显示与系统相关的所有消息，而不包括用户相关的消息：

    ```
    ~]# journalctl –-system
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:30:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpuset
    ...
    ~]#

    ```

1.  显示当前用户的所有消息。

    此命令显示与你登录的用户相关的所有消息：

    ```
    ~]# journalctl --user
    No journal files were found.
    ~]#

    ```

1.  接下来，使用以下命令行显示由特定服务生成的所有消息：

    ```
    ~]# journalctl --unit=<service>

    ```

    以下是一个示例：

    ```
    ~]# journalctl --unit=sshd
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:45:01 CEST. --
    Jun 26 23:40:18 rhel7.mydomain.lan systemd[1]: Starting OpenSSH server daemon...
    Jun 26 23:40:18 rhel7.mydomain.lan systemd[1]: Started OpenSSH server daemon.
    Jun 26 23:40:20 rhel7.mydomain.lan sshd[817]: Server listening on 0.0.0.0 port 22.
    Jun 26 23:40:20 rhel7.mydomain.lan sshd[817]: Server listening on :: port 22.
    Jun 27 11:30:08 rhel7.mydomain.lan sshd[4495]: Accepted publickey for root from 10.0.0.2 port 42748 ssh2: RSA cf:8a:a0:b4:4c:3d:d7:4d:93:c6:e0:fe:c0:66:e4
    ...
    ~]#

    ```

1.  现在，按优先级显示消息。

    可以通过关键字或数字指定优先级，例如 `debug` (7)、`info` (6)、`notice` (5)、`warning` (4)、`err` (3)、`crit` (2)、`alert` (1) 和 `emerg` (0)。指定优先级时，这也包括所有更低的优先级。例如，`err` 表示 `crit`、`alert` 和 `emerg` 也会显示。看看以下命令行：

    ```
    ~]# journalctl -p <priority>

    ```

    以下是一个示例：

    ```
    ~]# journalctl -p err
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Fri 2015-07-24 22:30:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: ioremap error for 0xdffff000-0xe0000000, requested 0x10, got 0x0
    Jun 26 23:38:49 rhel7.mydomain.lan systemd[1]: Failed unmounting /usr.
    ...
    ~]#

    ```

1.  接下来，按时间显示消息。

    你可以通过以下命令显示从当前启动以来的所有消息：

    ```
    ~]# journalctl -b
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:45:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpuset
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpu
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpuacct
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Linux version 3.10.0-229.4.2.el7.x86_64 (gcc version 4.8.2 20140120 (Red Hat 4.8.2-
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Command line: BOOT_IMAGE=/vmlinuz-3.10.0-229.4.2.el7.x86_64 root=/dev/mapper/rhel7_system-root ro vconsole.keymap=
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: e820: BIOS-provided physical RAM map:
    ~]# 

    ```

    你甚至可以通过运行以下命令，显示特定时间范围内的所有消息：

    ```
    ~]# journalctl --since="2015-07-24 08:00:00" --until="2015-07-24 09:00:00"
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:45:01 CEST. --
    Jul 24 08:00:01 rhel7.mydomain.lan systemd[1]: Created slice user-48.slice.
    Jul 24 08:00:01 rhel7.mydomain.lan systemd[1]: Starting Session 3331 of user apache.
    J
    ...
    Jul 24 08:45:01 rhel7.mydomain.lan systemd[1]: Starting Session 3335 of user apache.
    Jul 24 08:45:01 rhel7.mydomain.lan systemd[1]: Started Session 3335 of user apache.
    Jul 24 08:45:01 rhel7.mydomain.lan CROND[22909]: (apache) CMD (php -f /var/lib/owncloud/cron.php)
    ~]#

    ```

## 还有更多内容……

本食谱中的示例都可以组合使用。例如，如果你想显示 2015-07-24 8:00 到 9:00 之间的所有错误消息，你的命令将是：

```
~]# journalctl -p err --since="2015-07-24 08:00:00" --until="2015-07-24 09:00:00"

```

很多人倾向于“跟踪”日志文件，以确定发生了什么，试图找出任何问题。`journalctl` 是一个可执行文件，因此无法使用传统的“跟踪”技术，如 `tail -f` 或使用 `less` 并按 *CTRL* + *F*。编写 `systemd` 和 `systemctl` 的好心人提供了一个解决方案：只需在 `journalctl` 命令中添加 `-f` 或 `--follow` 参数。

尽管大多数环境习惯于创建 `syslog` 消息来进行故障排除，但日志系统提供了额外的价值，它能创建简单的过滤器，让你实时监控消息。

## 另见

关于在 RHEL 7 中使用日志系统的更多信息，请访问 [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html)。

查看 *journalctl (1)* 的手册页，了解可以配置的更多信息。

# 配置 logrotate

`logrotate` 工具允许你旋转由应用程序和脚本生成的日志文件。

它能保持你的日志目录整洁，并在正确配置时最小化磁盘使用。

## 如何操作……

`logrotate` 工具默认已安装，但为了完整性，我将包括安装说明。此教程将向你展示如何为 `rsyslog` 轮换日志。我们将每日轮换日志，基于日期添加扩展名，延迟一天压缩，并将其保存 365 天。请执行以下步骤：

1.  首先，安装 `logrotate`，请执行以下命令：

    ```
    ~]# yum install -y logrotate

    ```

1.  确保通过以下方式启用：

    ```
    ~]# systemctl restart crond

    ```

1.  使用你喜欢的编辑器打开 `/etc/logrotate.d/syslog`。默认情况下，这个文件的内容如下：

    ```
    /var/log/cron
    /var/log/maillog
    /var/log/messages
    /var/log/secure
    /var/log/spooler
    {
        sharedscripts
        postrotate
            /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
        endscript
    }
    ```

1.  现在，将其替换为以下代码：

    ```
    /var/log/cron
    /var/log/maillog
    /var/log/messages
    /var/log/secure
    /var/log/spooler
    {
        compress
        daily
        delaycompress
        dateext
        missingok
        rotate 365
        sharedscripts
        postrotate
            /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
        endscript
    }
    ```

1.  最后，保存文件。

## 它是如何工作的…

`logrotate` 工具是一个由 cron 每天启动的脚本。

默认的 `logrotate` 定义中添加的指令有 `compress`、`daily`、`delaycompress`、`dateext`、`missingok` 和 `rotate`。

`compress` 指令使用 gzip 压缩旧版本的日志文件。通过指定 `delaycompress`，这种行为有所变化。这样可以确保我们始终能够得到最新的轮换日志文件且未压缩。

`daily` 指令使得 `logrotate` 每天执行一次定义。`rotate` 指令在删除最旧的日志文件之前，只保留 `x` 个已轮换的日志文件。在此例中，我们将其指定为 365，意味着在每日轮换的同时，日志文件将保留 365 天。

`missingok` 指令允许 `syslog` 不创建文件，虽然这种情况很少发生，但它是可能的。

`dateext` 指令将日期以 `yyyymmdd` 格式附加到轮换文件上，而不是默认的数字形式。

## 还有更多…

`/etc/logrotate.conf` 文件包含了所有定义的默认指令。如果在文件定义中没有特别使用某个指令，则会使用此文件中指定的值（如果有）。

修改此文件中的设置使所有定义都受影响是有意义的，但这并不实际；并非所有日志文件都相同。`syslog` 服务会生成大量消息，可能会很快让你的系统变得杂乱无章。然而，像 yum 这样的服务并不会生成很多消息，它可以保持日志文件的可读性，远比 `syslog` 文件长时间有效。顺便提一下，这在 yum 的定义中有所体现。

如果你想调试新的配置，可以通过执行以下命令测试单个配置：

```
~# /usr/sbin/logrotate -v /etc/logrotate.d/<config file>

```

或者，你可以使用以下命令测试所有配置：

```
~]# /usr/sbin/logrotate -v /etc/logrotate.conf

```

这是一个示例：

```
~]# /usr/sbin/logrotate -v /etc/logrotate.d/syslog
reading config file /etc/logrotate.d/syslog

Handling 1 logs

rotating pattern: /var/log/cron
/var/log/maillog
/var/log/messages
/var/log/secure
/var/log/spooler
 1048576 bytes (no old logs will be kept)
empty log files are rotated, old logs are removed
considering log /var/log/cron
 log does not need rotating
considering log /var/log/maillog
 log does not need rotating
considering log /var/log/messages
 log does not need rotating
considering log /var/log/secure
 log does not need rotating
considering log /var/log/spooler
 log does not need rotating
not running postrotate script, since no logs were rotated
~]#

```

## 另见

查看 *logrotate (8)* 的手册页，了解更多关于配置 `logrotate` 的信息。

# 管理时间

RHEL 7 默认安装了 Chrony。虽然每个人都知道 Ntpd，但 Chrony 是时间同步领域的“新手”。

Chrony 是一套程序，通过使用不同的时间源（如 NTP 服务器、系统时钟，甚至是自定义脚本/程序）来保持计算机的时间。它还会计算计算机丧失或增加时间的速率，以便在没有外部参考的情况下进行补偿——例如，如果你的 NTP 服务器出现故障。

Chrony 是一个适合那些间歇性断开和重新连接到网络的系统的好解决方案。

对于通常长时间保持开启的系统，应考虑使用`ntpd`。

## 如何操作…

在谈到 RHEL 中管理时间时，可以通过以下方式进行：

+   Chrony

+   Ntpd

我们将分别查看每种方法。

### 通过`chrony`管理时间

确保已安装并启用`chrony`，并执行以下步骤：

1.  首先，通过以下命令安装`chrony`：

    ```
    ~]# yum install -y chrony

    ```

1.  启用`chrony`，如下所示：

    ```
    ~]# systemctl enable chrony
    ~]# systemctl start chrony

    ```

1.  现在，使用你喜欢的编辑器打开`/etc/chrony.conf`，并使用以下命令查找以`server`指令开头的行：

    ```
    server 0.rhel.pool.ntp.org iburst
    server 1.rhel.pool.ntp.org iburst
    server 2.rhel.pool.ntp.org iburst
    server 3.rhel.pool.ntp.org iburst
    ```

1.  接下来，用你附近的 NTP 服务器替换这些行并保存文件：

    ```
    server 0.pool.ntp.mydomain.lan iburst
    server 1.pool.ntp.mydomain.lan iburst
    ```

    `iburst`选项会使 NTP 在下次轮询时发送八个数据包，而不是仅发送一个，如果时间主服务器不可用，从而加快 NTP 守护进程的时间同步速度。

1.  最后，通过执行以下命令重新启动`chrony`：

    ```
    ~]# systemctl restart chrony

    ```

### 通过`ntpd`管理时间

确保已安装并启用`ntpd`，并执行以下步骤：

1.  首先，通过以下命令安装`ntpd`：

    ```
    ~]# yum install -y ntpd

    ```

1.  通过以下命令启用`ntpd`：

    ```
    ~]# systemctl enable ntpd

    ```

1.  使用你喜欢的编辑器打开`/etc/ntp.conf`，并查找以`server`指令开头的行。运行以下命令：

    ```
    server 0.rhel.pool.ntp.org iburst
    server 1.rhel.pool.ntp.org iburst
    server 2.rhel.pool.ntp.org iburst
    server 3.rhel.pool.ntp.org iburst
    ```

1.  用你附近的 NTP 服务器替换这些行并保存文件：

    ```
    server 0.pool.ntp.mydomain.lan iburst
    server 1.pool.ntp.mydomain.lan iburst
    ```

1.  用你的所有 NTP 服务器替换`/etc/ntp/step-tickers`文件的内容，每行一个：

    ```
    0.pool.ntp.mydomain.lan
    1.pool.ntp.mydomain.lan
    ```

1.  现在，通过执行以下命令重新启动`ntpd`：

    ```
    ~]# systemctl restart ntpd

    ```

## 还有更多内容…

虽然`ntpd`是时间同步的明显选择，但它在时间主服务器间歇性可访问的环境中表现不佳（无论什么原因）。在这些环境中，`chronyd`表现优异。而且，`ntpd`的配置较为复杂，而`chronyd`稍微简单一些。

在使用`ntpd`文件时修改`/etc/ntp/step-tickers`的原因是为了服务的启动。它使用`ntpdate`在实际启动 NTP 守护进程之前进行一次同步，这比 NTP 守护进程本身同步时间要快。

要检查你的系统是否已同步，可以使用以下命令：

+   对于`chrony`，使用以下命令：

    ```
    ~]# chronyc sources

    ```

+   对于`ntpd`，运行以下命令：

    ```
    ~]# ntpq -p

    ```

你的输出将类似于：

```
remote       refid      st t when poll reach   delay   offset  jitter
=====================================================================
 LOCAL(0)    .LOCL.      5 l  60m   64    0    0.000    0.000   0.000
*master.exam 178.32.44.208 3 u  35 128 377     0.214   -0.651  14.285

```

在条目前面的星号（`*`）表示你的系统与该远程系统的时钟已同步。

## 另请参见

有关如何为 RHEL 7 配置`chrony`的更多信息，请访问[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_the_chrony_Suite.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_the_chrony_Suite.html)。

有关如何为 RHEL 7 配置`ntpd`的更多信息，请访问[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_ntpd.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_ntpd.html)。

# 配置你的启动环境

GRUB2 是 RHEL 7 的默认引导加载程序。默认情况下，它不使用任何复杂的配置选项，但至少保护你的 grub 引导加载程序是明智的。

## 如何操作…

在企业环境中，将 grub 和启动环境输出到串口控制台有许多优点。许多供应商在他们的远程控制系统中集成了虚拟串口，KVM 也不例外。这允许你连接到串口，并轻松地获取在文本编辑器中显示的内容。

在 GRUB2 引导加载程序上设置密码可以减轻当物理访问服务器或控制台时可能发生的黑客攻击。请按照以下步骤进行此操作：

1.  首先，使用你喜欢的编辑器编辑`/etc/sysconfig/grub`文件。

1.  现在，通过执行以下命令行，修改`GRUB_TERMINAL_OUTPUT`行，以同时包括控制台和串口访问：

    ```
    GRUB_TERMINAL_OUTPUT="console serial"
    ```

1.  添加`GRUB_SERIAL_COMMAND`条目，如下所示：

    ```
    GRUB_SERIAL_COMMAND="serial --speed=9600 --unit=0 --word=8 --parity=no –stop=1"
    ```

1.  现在，保存文件。

1.  创建`/etc/grub.d/01_users`文件，内容如下：

    ```
    cat << EOF
    set superusers="root"
    password root SuperSecretPassword
    EOF

    ```

1.  接下来，通过运行以下命令来更新你的`grub`配置：

    ```
    ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-3.10.0-229.4.2.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-229.4.2.el7.x86_64.img
    Found linux image: /boot/vmlinuz-3.10.0-229.1.2.el7.x86_64
    Found initrd image: /boot/initramfs-3.10.0-229.1.2.el7.x86_64.img
    Found linux image: /boot/vmlinuz-0-rescue-fe045089e49942cb97db675892395bc8
    Found initrd image: /boot/initramfs-0-rescue-fe045089e49942cb97db675892395bc8.img
    done
    ~]#

    ```

## 它是如何工作的…

`grub2-mkconfig`的行为由`/etc/grub.d`中的文件指令定义。这些文件基于`/etc/sysconfig/grub`中的配置，自动生成`grub.cfg`文件中的所有菜单项。你可以通过在此目录中添加包含 bash 代码的文件来修改其行为。

例如，你可以添加一个脚本，用于将一个菜单项添加到从 CD/DVD ROM 驱动器启动。

被添加到`/etc/grub.d/01_users`中的用户 root 是唯一被允许从控制台编辑菜单项的用户，这可以缓解 GRUB 的弱点，即通过在`kernel`行的末尾添加`1`或`rescue`来强制进入恢复模式。

## 还有更多内容……

`grub2-mkconfig`命令是特定于 BIOS 系统的。为了在 UEFI 系统上执行相同操作，请按如下方式修改命令：

```
~]# grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

```

为了通过相同的串口连接访问 GRUB 终端，你需要指定一个额外的内核选项：`console=ttyS0,9600n8`。

你可以修改`/boot/grub2/grub.cfg`中的内核行（或者手动修改`/boot/efi/EFI/redhat/grub.cfg`，但你有可能在内核更新时丢失更改），或者使用`grub2-mkconfig`手动重新生成文件。

最好将其添加到`/etc/sysconfig/grub`中的`GRUB_CMDLINE_LINUX`指令，并重新生成`grub.cfg`文件。

GRUB 用户的密码可以使用`grub2-mkpasswd-pbkdf2`命令进行加密，如下所示：

```
~]# grub2-mkpasswd-pbkdf2
Enter password:
Reenter password:
PBKDF2 hash of your password is grub.pbkdf2.sha512.10000.C208DD5E318B1D6477C4E51035649C197411259C214D0B83E3E83753AD58F7676B62CDF48E31AF0E739844A5CF9A95F76AF5008AF340336DB50ECA23906ECC13.9D20A66F0CADA12AA617B293B5BBF7AAD44423ECA513F302FEBF5CB92A0DC54436E16D7CD6E09685323084A27462C2A981054D52F452F5C2F71FBACD2C31AEFA
~]#

```

然后，你可以将`/etc/grub.d/01_users`中的明文密码替换为生成的哈希值。以下是一个示例：

```
password root grub.pbkdf2.sha512.10000.C208DD5E318B1D6477C4E51035649C197411259C214D0B83E3E83753AD58F7676B62CDF48E31AF0E739844A5CF9A95F76AF5008AF340336DB50ECA23906ECC13.9D20A66F0CADA12AA617B293B5BBF7AAD44423ECA513F302FEBF5CB92A0DC54436E16D7CD6E09685323084A27462C2A981054D52F452F5C2F71FBACD2C31AEFA
```

所有自动生成的条目都是可引导的，但不能从控制台编辑，除非你知道用户名和密码。如果你有自定义菜单条目并希望以类似方式保护它们，可以在菜单条目定义之前加上`--unrestricted`。以下是一个示例：

```
menuentry 'My custom grub boot entry' <options> --unrestricted {
```

## 另见

有关使用 GRUB2 引导加载程序的更多信息，请访问[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Working_with_the_GRUB_2_Boot_Loader.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Working_with_the_GRUB_2_Boot_Loader.html)。

# 配置 smtp

许多程序使用（或可以配置为使用）SMTP 来发送关于其状态等的消息。默认情况下，Postfix 配置为将所有消息本地发送，并且不响应传入邮件。如果你的环境中有多个服务器，那么每次都登录到每个服务器检查新邮件可能会变得非常麻烦。本教程将展示如何将邮件转发到使用 SMTP 的中央邮件中继或消息存储。

在 RHEL 7 上，Postfix 默认已安装。

## 如何操作…

在这个教程中，我们将结合多个选项：

+   我们将允许服务器接受传入邮件。

+   我们只允许服务器转发来自`mydomain.lan`域的收件人的邮件。

+   我们将所有邮件转发到`mailhost.mydomain.lan`邮件服务器。

完成此教程，请执行以下步骤：

1.  使用你喜欢的编辑器编辑`/etc/postfix/main.cf`文件。

1.  修改`inet_interface`，通过以下命令使邮件能够在任何接口上接受：

    ```
    inet_interface = all
    ```

1.  将`smtpd_recipient_restrictions`指令添加到配置文件中，仅允许来自`mydomain.lan`域的传入邮件，如下所示：

    ```
    smtpd_recipient_restrictions =
        check_sender_access hash:/etc/postfix/sender_access,
        reject
    ```

    如你所见，最后两行是缩进的。`postfix`会将这个块视为一行，而不是三行。

1.  将`relayhost`指令添加到配置文件中，指向`mailhost.mydomain.lan`，如下所示：

    ```
    relayhost = mailhost.mydomain.lan
    ```

1.  现在，保存`postfix`文件。

1.  创建`/etc/postfix/sender_access`并添加以下内容：

    ```
    mydomain.lan OK
    ```

1.  接下来，使用以下命令对`/etc/postfix/access`文件进行哈希处理：

    ```
    ~]# postmap /etc/postfix/access

    ```

1.  最后，按如下方式重新启动`postfix`：

    ```
    ~]# systemctl restart postfix

    ```

## 还有更多…

要监控系统上的邮件队列，请执行以下命令：

```
~]# postqueue -p

```

当你的邮件中继无法转发邮件时，它会将邮件存储在本地，并尝试稍后重新发送。恢复邮件流后，你可以通过执行以下操作来刷新队列并尝试投递：

```
~]# postqueue -f

```

本文档中展示的设置非常简单，假设你的网络中没有恶意用户。有一些软件可以帮助你减少垃圾邮件和病毒。常见的解决方案包括 `spamassassin` 和 `amavis`。

## 另请参见

有关在 RHEL 7 中使用 postfix 的更多信息，请访问 [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-email-mta.html#s2-email-mta-postfix`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-email-mta.html#s2-email-mta-postfix)。

有关 postfix 的更多信息，请查看 postfix rpm（`rpm -ql postfix`）或访问 [`www.postfix.org/`](http://www.postfix.org/)。该站点提供了良好的文档和*如何做*指南，涵盖了大量场景。
