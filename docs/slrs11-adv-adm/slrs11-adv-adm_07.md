# 第七章 配置和管理 RBAC 与最小权限

本章我们将涵盖以下内容：

+   配置和使用 RBAC

+   操作最小权限

# 引言

**基于角色的访问控制**（**RBAC**）是一个了不起的功能，它同样存在于 Oracle Solaris 11 中（其起源可以追溯到 Oracle Solaris 8），主要功能是使得能够限制普通用户执行任务时所获得的权限。换句话说，RBAC 使得能够仅授权普通用户完成管理任务所需的最小权限，这种方式类似于 sudo 程序。与 sudo 程序相比，RBAC 的主要区别在于它完全集成在操作系统中，并且在用户登录 Oracle Solaris 11 时就会使用它。此外，RBAC 提供比 sudo 更精细的权限访问，并且与 Oracle Solaris 11 中的另一个出色功能——最小权限（least privilege）——集成，最小权限用于从进程和程序中删除不必要的权限，从而可以减少黑客攻击的表面。

# 配置和使用 RBAC

在解释和实现 RBAC 功能之前，需要先记住为什么 RBAC 是必要的，然后学习一些基本概念。

根据我们之前对 Oracle Solaris 11 的研究，普通用户是无法重启 Oracle Solaris 11 系统的，如下所示的命令所示：

```
root@solaris11-1:~# useradd -d /export/home/aborges -m -s /bin/bash aborges
80 blocks
root@solaris11-1:~# passwd aborges
New Password: hacker123!
Re-enter new Password: hacker123!
passwd: password successfully changed for aborges
root@solaris11-1:~# su - aborges
Oracle Corporation  SunOS 5.11  11.1  September 2012
aborges@solaris11-1:~$ reboot
reboot: permission denied
aborges@solaris11-1:~$
```

一个简单且完全不合适的解决方案是将 `root` 账户的密码提供给用户 `aborges`。然而，这在专业公司中是无法想象的。另一种且推荐的解决方案是使用 RBAC，这是一项安全功能，它允许普通用户完成管理任务，比如重启系统，如我们之前所做的尝试。

RBAC 框架包含以下对象：

+   **角色**：这是一种特殊类型的用户，创建它是为了执行管理任务，尽管无法直接登录系统，正确的做法是以普通用户身份登录，然后通过 `su` 命令来假设角色。由于角色是一种特殊的用户类型，它在 `/etc/passwd` 文件中配置，并且在 `/etc/shadow` 文件中定义了密码。然而，与普通用户不同，无法通过角色登录 Oracle Solaris 11。用户必须使用普通账户登录，然后可以通过 `su` 命令假设角色。

+   **配置文件**：这是一个命令集合。任何分配给配置文件的角色都可以执行该配置文件中的任何命令。所有系统配置文件都定义在 `/etc/security/prof_attr.d/core-os` 文件中，本地配置文件可以在 `/etc/security/prof_attr` 文件中定义。要列出所有配置文件，请使用以下命令：

    ```
    root@solaris11-1:~# getent prof_attr | more
    Software Installation:RO::Add application software to the system:auths=solaris.smf.manage.servicetags;profiles=ZFS File System Management;help=RtSoftwareInst
    all.html
    NTP Management:RO::Manage the NTP service:auths=solaris.smf.manage.ntp,solaris.smf.value.ntp
    Desktop Configuration:RO::Configure graphical desktop software:auths=solaris.smf.manage.dt.login,solaris.smf.manage.x11,solaris.smf.manage.font,solaris.smf.m
    anage.opengl
    Device Security:RO::Manage devices and Volume Manager:auths=solaris.smf.manage.dt.login,solaris.device.*,solaris.smf.manage.vt,solaris.smf.manage.allocate;he
    lp=RtDeviceSecurity.html
    Desktop Removable Media User:RO::Access removable media for desktop user:
    (truncated output)

    ```

+   **授权**：这表示为完成特定任务而设置的特殊权限，例如访问 CD-ROM、管理 CUPS 打印服务、NTP 服务、区域、SMF 框架等。通常，授权是在 Oracle Solaris 安装过程中或通过新安装的软件创建的。所有系统授权都定义在`/etc/security/auth_attr.d/core-os`文件中，本地授权定义在`/etc/security/auth_attr`文件中。要列出所有授权，我们运行以下命令：

    ```
    root@solaris11-1:~# getent auth_attr | more
    solaris.smf.read.ocm:::Read permissions for protected Oracle Configuration Manager Service Properties::
    solaris.smf.value.ocm:::Change Oracle Configuration Manager System Repository Service values::
    solaris.smf.manage.ocm:::Manage Oracle Configuration Manager System Repository Service states::
    solaris.smf.manage.cups:::Manage CUPS service states::help=ManageCUPS.html
    solaris.smf.manage.zfs-auto-snapshot:::Manage the ZFS Automatic Snapshot Service::
    solaris.smf.value.tcsd:::Change TPM Administation value properties::
    (truncated output)

    ```

+   **权限**：这是一种单一的权利，可以分配给用户、角色、命令甚至系统。

+   **执行属性**：这些是定义在`/etc/security/exec_attr.d/core-os`（系统执行属性）或`/etc/security/exec_attr`文件（本地定义）中的命令，它们被分配给一个或多个配置文件。要列出所有执行属性，我们运行以下命令：

    ```
    root@solaris11-1:~# getent exec_attr | more
    DTrace Toolkit:solaris:cmd:::/usr/dtrace/DTT/*/*:privs=dtrace_kernel,dtrace_proc,dtrace_user
    Desktop Configuration:solaris:cmd:RO::/usr/bin/scanpci:euid=0;privs=sys_config
    Desktop Configuration:solaris:cmd:RO::/usr/X11/bin/scanpci:euid=0;privs=sys_config
    OpenLDAP Server Administration:suser:cmd:RO::/usr/sbin/slapd:uid=openldap;gid=openldap;privs=basic,net_privaddr
    OpenLDAP Server Administration:suser:cmd:RO::/usr/sbin/slapacl:uid=openldap;gid=openldap
    (truncated output)

    ```

+   **配置文件外壳**：这是一种特殊的配置文件（`pfbash`、`pfsh`、`pfcsh`或`pfzsh`），在`su`命令中分配给用户，以假设一个角色或登录外壳，允许访问特定的权限。必须使用其中任何一种配置文件外壳。

+   **安全策略**：这定义了用户的默认权限和配置文件。相关的配置文件是`/etc/security/policy.conf`，如下所示：

    ```
    root@solaris11-1:~# more /etc/security/policy.conf

    ```

使用 RBAC 有两种方法。第一种方法更简单、直接；你可以直接创建并分配一个配置文件给用户账户，以便作为普通用户登录并使用`pfexec`命令从分配的配置文件中执行额外的命令。

第二种方法是将所有提到的 RBAC 概念（命令、授权、配置文件、角色和用户）结合在一起，按照以下方案从右到左排列：

用户 <-- 角色 <-- 配置文件 <-- 命令和/或授权

第二种方法较为复杂，使用 RBAC 所需的步骤如下所述：

1.  使用`roleadd`命令创建一个角色。

1.  创建一个配置文件，编辑`/etc/security/prof_attr`文件。

1.  在`/etc/security/exec_attr`中为创建的配置文件（步骤 2）分配命令，或在`/etc/security/prof_attr`文件中为配置文件分配授权（`/etc/security/auth_attr`）。

1.  使用`rolemod`命令将配置文件分配给角色。

1.  使用`passwd`命令为角色创建一个密码。

1.  使用`usermod`命令将一个或多个用户分配给该角色。

1.  当用户需要使用指定命令时，执行`su - <rolename>`。

这很好！这是管理 RBAC 所需概念的总结。我们将学习如何执行逐步操作，以便了解两种方法。

## 准备工作

这个配方需要一台虚拟机（VirtualBox 或 VMware），运行 Oracle Solaris 11，并且至少有 2GB 的 RAM。

## 如何操作…

我们将学习两种方法，使普通用户能够重启系统，即使用`pfexec`命令（较简单）和 RBAC 角色（更复杂）。

使用`pfexec`命令很简单。首先，创建`aborges`普通用户，密码为`hacker123!`，如以下命令所示：

```
root@solaris11-1:~# useradd -d /export/home/aborges -m -s /bin/bash aborges
80 blocks
root@solaris11-1:~# passwd aborges
New Password: hacker123!
Re-enter new Password: hacker123!
passwd: password successfully changed for aborges
```

主要的思路是将一个配置文件（即一组命令）直接关联到用户（`aborges`）。在此情况下，所需的配置文件已经存在；如果没有，我们需要创建一个新的。为了避免创建不必要的配置文件，可以通过执行以下命令来验证`/etc/security/exec_attr.d/core-os`文件中是否已经有包含`reboot`命令的行：

```
root@solaris11-1:~# cat /etc/security/exec_attr.d/core-os | grep reboot
Maintenance and Repair:solaris:cmd:RO::/usr/sbin/reboot:uid=0
```

太棒了！有一个名为`"维护和修复"`的配置文件，其中包括重启命令。为了完成我们的任务，将此配置文件（使用`–P`选项）与`aborges`用户关联，如下所示的命令：

```
root@solaris11-1:~# usermod -P "Maintenance and Repair" aborges
root@solaris11-1:/# more /etc/user_attr.d/local-entries | grep aborges
aborges::::profiles=Maintenance and Repair
```

正如我们所发现的，它在`/etc/user_attr.d/local-entries`文件中为`aborges`用户创建了一个条目。然而，即便包含了这个条目，将`aborges`用户与`"维护和修复"`配置文件关联起来，用户仍然无法重启系统，如下所示的命令：

```
root@solaris11-1:/# su – aborges
Oracle Corporation  SunOS 5.11  11.1  September 2012
aborges@solaris11-1:~$ reboot
reboot: permission denied
```

然而，如果`aborges`用户想要使用`pfexec`执行相同的命令，结果会不同，如下所示的命令：

```
aborges@solaris11-1:~$ pfexec reboot

```

它成功了！系统将按预期重启。

使用`pfexec`命令的方法很棒，但选择的配置模式（使用现有的配置文件）可能会带来两个小副作用：

+   `"维护和修复"`配置文件包含其他命令，我们也已将这些命令分配给`aborges`用户，如下所示的命令：

    ```
    root@solaris11-1:~# cat /etc/security/exec_attr.d/core-os | grep -i "Maintenance and Repair"
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/mdb:privs=all
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/coreadm:euid=0;privs=proc_owner
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/croinfo:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/date:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/ldd:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/vmstat:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/eeprom:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/halt:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/init:uid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/pcitool:privs=all
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/poweroff:uid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/prtconf:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/reboot:uid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/syslogd:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/bootadm:euid=0
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/ucodeadm:privs=all
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/cpustat:privs=basic,cpc_cpu
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/pgstat:privs=basic,cpc_cpu
    Maintenance and Repair:solaris:cmd:RO::/usr/bin/kstat:privs=basic,cpc_cpu
    Maintenance and Repair:solaris:cmd:RO::/usr/sbin/ilomconfig:privs=sys_config,sys_ip_config,sys_dl_config
    Maintenance and Repair:solaris:cmd:RO::/usr/lib/ilomconfig.builtin:privs=sys_config,sys_ip_config,sys_dl_config
    ```

    为了防止这种情况，最好创建一个新配置文件，并仅将重启命令分配给它。

+   第二个副作用是，使用`pfexec`命令的过程需要为每个需要使用`reboot`命令的用户进行，但这可能需要额外的时间。

达成目标的第二种方法是结合使用角色、配置文件和/或授权。在这种情况下的优点是，权限不是直接与用户关联，而是分配给角色。然后，如果普通用户需要重启系统（例如），它可以通过`su`命令获取角色并执行相应的命令。

创建另一个用户（与前一个用户不同）并用于此方法，执行以下命令：

```
root@solaris11-1:~# useradd -d /export/home/rbactest -m -s /bin/bash rbactest
80 blocks
root@solaris11-1:~# passwd rbactest
New Password: oracle123!
Re-enter new Password: oracle123!
passwd: password successfully changed for rbactest
```

为了确认`brbactest`用户不能重启系统，执行以下命令：

```
root@solaris11-1:~# su - rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012

rbactest@solaris11-1:~$ reboot
reboot: permission denied
```

创建一个将在之后配置的角色，执行以下命令：

```
root@solaris11-1:~# roleadd -m -d /export/home/r_reboot -s /bin/pfbash r_reboot
80 blocks
root@solaris11-1:~# grep r_reboot /etc/passwd
r_reboot:x:103:10::/export/home/r_reboot:/bin/bash
root@solaris11-1:~# grep r_reboot /etc/shadow
r_reboot:UP:::::::
```

如我们之前所提到的，配置文件在 RBAC 配置中非常重要。系统已经有一些已定义的系统配置文件，这些配置文件在`/etc/security/prof_attr.d/core-os`文件中进行配置，如下所示的命令：

```
root@solaris11-1:~# more /etc/security/prof_attr.d/core-os 
(truncated output)
All:RO::\
Execute any command as the user or role:\
help=RtAll.html

Administrator Message Edit:RO::\
Update administrator message files:\
auths=solaris.admin.edit/etc/issue,\
solaris.admin.edit/etc/motd;\
help=RtAdminMsg.html

Audit Configuration:RO::\
Configure Solaris Audit:\
auths=solaris.smf.value.audit;\
help=RtAuditCfg.html

Audit Control:RO::\
Control Solaris Audit:\
auths=solaris.smf.manage.audit;\
help=RtAuditCtrl.html
(truncated output)

```

因此，根据本配方介绍中建议的步骤，在配置文件的末尾创建一个名为`Reboot`的配置文件，如下所示的命令所示：

```
root@solaris11-1:~# vi /etc/security/prof_attr
#
# The system provided entries are stored in different files
# under "/etc/security/prof_attr.d".  They should not be
# copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
Reboot:RO::\
For authorized users to reboot the system:\
help=RebootByRegularUser.html

```

从这个文件中我们知道，配置文件的名称是`Reboot`，而`RO`（只读）字符表示它不能被任何修改此数据库的工具修改。接下来的行表示描述和帮助文件（不必创建它）。可以绑定授权（`auths`键）、其他配置文件（`profiles`键）和权限（`priv`键）到此`Reboot`配置文件。

配置文件创建后，我们需要为该配置文件分配一个或多个命令，局部修改通过编辑`/etc/security/exec_attr`文件来完成，如下所示的命令：

```
root@solaris11-1:~# vi /etc/security/exec_attr
#
# The system provided entries are stored in different files
# under "/etc/security/exec_attr.d".  They should not be
# copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
Reboot:solaris:cmd:RO::/usr/sbin/reboot:uid=0

```

```
 explained as follows:
```

+   `Reboot`：这是配置文件的名称。

+   `solaris`：这是与`Reboot`配置文件相关联的安全策略。此安全策略能够识别权限。Oracle Solaris 11 为此字段提供了另一个可能的值，名为`suser`（之前未显示），它与`solaris`值非常相似，但无法理解和识别权限。

+   `cmd`：这是一个对象类型。在本例中，它是一个由 shell 执行的命令。

+   `RO`：这表示这一行不能被任何修改此文件的工具修改。

+   `/usr/sbin/reboot`：这是用户在假定具有此`Reboot`配置文件的角色时要执行的命令。

+   `Uid=0`：该命令以用户的根用户的真实 ID（`uid=0`）运行。当用户必须运行命令时，该命令将作为根用户执行。其他常见且有用的键包括`euid`（有效用户 ID，类似于使用`setuid`设置可执行文件时运行命令）和`privs`（权限）。

再次查看已经存在的系统执行属性，定义在`/etc/security/exec_attr.d/core-os`文件中，非常有趣，如下所示的命令：

```
root@solaris11-1:~# more /etc/security/exec_attr.d/core-os 
(truncated output)
All:solaris:cmd:RO::*:
Audit Control:solaris:cmd:RO::/usr/sbin/audit:privs=proc_owner,sys_audit
Audit Configuration:solaris:cmd:RO::/usr/sbin/auditconfig:privs=sys_audit
Audit Review:solaris:cmd:RO::/usr/sbin/auditreduce:euid=0
Audit Review:solaris:cmd:RO::/usr/sbin/auditstat:privs=proc_audit
Audit Review:solaris:cmd:RO::/usr/sbin/praudit:privs=file_dac_read
Contract Observer:solaris:cmd:RO::/usr/bin/ctwatch:\
  privs=contract_event,contract_observer
Cron Management:solaris:cmd:RO::/usr/bin/crontab:euid=0
Crypto Management:solaris:cmd:RO::/usr/sbin/cryptoadm:euid=0
Crypto Management:solaris:cmd:RO::/usr/bin/kmfcfg:euid=0
Crypto Management:solaris:cmd:RO::/usr/sfw/bin/openssl:euid=0
Crypto Management:solaris:cmd:RO::/usr/sfw/bin/CA.pl:euid=0
DHCP Management:solaris:cmd:RO::/usr/lib/inet/dhcp/svcadm/dhcpconfig:uid=0
DHCP Management:solaris:cmd:RO::/usr/lib/inet/dhcp/svcadm/dhtadm:uid=0
DHCP Management:solaris:cmd:RO::/usr/lib/inet/dhcp/svcadm/pntadm:uid=0
(truncated output)

```

现在是时候将`r_reboot`角色绑定到`Reboot`配置文件（使用`–P`选项），通过执行以下命令：

```
root@solaris11-1:~# rolemod -P Reboot r_reboot
root@solaris11-1:~# more /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;profiles=Reboot;roleauth=role

```

根据之前的输出，`r_reboot`是`role`类型，并且与`Reboot`配置文件相关联。

`r_reboot`角色没有密码，因此我们应该通过运行以下命令为其设置一个新密码：

```
root@solaris11-1:~# passwd r_reboot
New Password: hacker321!
Re-enter new Password: hacker321!
passwd: password successfully changed for r_reboot
root@solaris11-1:~# grep r_reboot /etc/shadow
r_reboot:$5$q75Eiy5/$u9mgnYsvlszbNXkSuH4kZwVVnFOhemnCTMF//cvBWD9:16178::::::19216
```

RBAC 配置几乎完成。为了假定此`r_reboot`角色，`rbactest`用户必须通过使用`usermod`命令中的`-R`选项来分配该角色，如下所示的命令：

```
root@solaris11-1:~# usermod -R r_reboot rbactest
root@solaris11-1:~# more /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;profiles=Reboot;roleauth=role
rbactest::::roles=r_reboot

```

为了确认到目前为止执行的每个任务，请运行以下命令：

```
root@solaris11-1:~# roles rbactest
r_reboot
root@solaris11-1:~# profiles rbactest
rbactest:
          Basic Solaris User
          All
root@solaris11-1:~# profiles r_reboot
r_reboot:
          Reboot
 Basic Solaris User
 All

```

值得记住的是，`rbactest` 是一个用户，而 `r_reboot` 是一个角色，正如之前所解释的，无法使用角色登录系统。此外，现有的配置文件有 `Basic Solaris User`，允许用户根据已设定的安全限制使用系统，和 `All`，它提供对没有任何安全属性的命令的访问。

继续验证，我们需要检查 `r_reboot` 角色和 `rbactest` 用户的授权，以及分配给 `r_reboot` 角色的配置文件。这些任务通过执行以下命令序列完成：

```
root@solaris11-1:~# auths r_reboot
solaris.admin.wusb.read,solaris.mail.mailq,solaris.network.autoconf.read
root@solaris11-1:~# auths rbactest
solaris.admin.wusb.read,solaris.mail.mailq,solaris.network.autoconf.read
root@solaris11-1:~# profiles -l r_reboot
r_reboot:
      Reboot
          /usr/sbin/reboot           uid=0

      Basic Solaris User
      auths=solaris.mail.mailq,solaris.network.autoconf.read,solaris.admin.wusb.read
      profiles=All
              /usr/bin/cdrecord.bin      privs=file_dac_read,sys_devices,proc_lock_memory,proc_priocntl,net_privaddr
          /usr/bin/readcd.bin        privs=file_dac_read,sys_devices,net_privaddr
          /usr/bin/cdda2wav.bin      privs=file_dac_read,sys_devices,proc_priocntl,net_privaddr

      All
          *
```

有几点需要特别强调：

+   `rbactest` 用户被分配给了 `r_reboot` 角色。

+   `rbactest` 用户和 `r_reboot` 角色都没有被分配任何授权。

+   `All` 配置文件授予对 Oracle Solaris 11 所有不受限制命令的完全访问权限。在这种情况下，`r_reboot` 角色与三个配置文件相关联：`Reboot`、`Basic Solaris User` 和 `All`。

+   `Basic Solaris User` 配置文件可以使用特定的权限执行一些相关的 CD-ROM 命令。

最后，我们可以通过执行以下命令验证 `rbactest` 用户是否能够重启系统：

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# su - rbactest
Oracle Corporation	SunOS 5.11	11.1	September 2012
rbactest@solaris11-1:~$ id
uid=102(rbactest) gid=10(staff)
rbactest@solaris11-1:~$ profiles
          Basic Solaris User
          All
rbactest@solaris11-1:~$ su - r_reboot
Password: hacker321!
Oracle Corporation	SunOS 5.11	11.1	September 2012
r_reboot@solaris11-1:~$ id
uid=103(r_reboot) gid=10(staff)
r_reboot@solaris11-1:~$ profiles
          Reboot
          Basic Solaris User
          All
r_reboot@solaris11-1:~$ reboot

```

系统立即重新启动。太棒了！

RBAC 允许你将你所学到的所有概念（角色、配置文件、授权和命令）与特权相结合；因此，它为我们提供了比 sudo 程序更细粒度且集成的控制。

在使用 Oracle Solaris 11 时，我们可以将 RBAC 与 SMF 框架中的服务一起使用。例如，DNS 客户端和 DHCP 服务器有以下授权：

```
root@solaris11-1:~# svcprop -p general/action_authorization dns/client
solaris.smf.manage.name-service.dns.client
root@solaris11-1:~# svcprop -p general/action_authorization dhcp/server:ipv4
solaris.smf.manage.dhcp
```

没有这些适当的授权，`rbactest` 用户将无法管理这些服务，以下命令显示了这一点：

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# su - rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ id
uid=102(rbactest) gid=10(staff)
rbactest@solaris11-1:~$ svcadm restart dns/client
svcadm: svc:/network/dns/client:default: Permission denied.
rbactest@solaris11-1:~$ svcadm restart dhcp/server:ipv4
svcadm: svc:/network/dhcp/server:ipv4: Permission denied.
```

通过执行以下命令，分配相应的授权给 `r_reboot` 角色，可以轻松解决这些问题：

```
root@solaris11-1:~# rolemod -A solaris.smf.manage.name-service.dns.client,solaris.smf.manage.dhcp r_reboot

```

为了验证之前的命令是否生效，请检查已修改的文件：

```
root@solaris11-1:~# more /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;auths=solaris.smf.manage.name-service.dns.client,solaris.smf.manage.dhcp;profiles=Reboot;defaultpriv=basic,file_dac_read;roleauth=role
rbactest::::defaultpriv=basic,file_dac_read;roles=r_reboot
```

太好了！现在是时候通过执行以下命令来测试我们的修改是否有效：

```
root@solaris11-1:~# su - rbactest
Oracle Corporation  SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ su - r_reboot
Password: hacker321!
Oracle Corporation  SunOS 5.11  11.1  September 2012
r_reboot@solaris11-1:~$ svcadm -v restart dns/client
Action restart set for svc:/network/dns/client:default.
r_reboot@solaris11-1:~$ svcadm -v restart dhcp/server:ipv4
Action restart set for svc:/network/dhcp/server:ipv4.
```

太棒了！RBAC 与 SMF 的集成非常完美，像 `rbactest` 这样的普通用户能够像 root 用户一样管理服务（DNS 客户端和 DHCP 服务器）。

如果我们想要解除 `rbactest` 用户与 `r_reboot` 角色的绑定，以防止他们重启系统或执行其他系统操作，请执行以下命令：

```
root@solaris11-1:~# roles rbactest
r_reboot
root@solaris11-1:~# usermod -R "" rbactest
root@solaris11-1:~# roles rbactest
root@solaris11-1:~#
```

最后还有一点补充：可以在 `/etc/security/policy.conf` 文件中为每个用户配置默认的 RBAC 授权和配置文件。同样，也可以配置默认的特权及其限制，如以下命令所示：

```
root@solaris11-1:~# more /etc/security/policy.conf 
(truncated output)
AUTHS_GRANTED=
PROFS_GRANTED=Basic Solaris User
CONSOLE_USER=Console User
(truncated output)
#
#PRIV_DEFAULT=basic
#PRIV_LIMIT=all
#
(truncated output)

```

### 配方概览

在本节中，我们学习了如何使用 RBAC 使普通用户能够重启系统。此外，我们还测试了如何查找并授予必要的授权来管理 SMF 框架中的服务。相同的步骤适用于任何用户和任何数量的命令。

# 使用最少权限

Oracle Solaris 11，像其他优秀的类 UNIX 操作系统一样，在其起步时存在一个缺陷；有一个名为 root 的特权账户，它在系统上拥有所有特权权限，而其他账户则拥有有限的权限，如普通用户。在这种模式下，进程要么拥有所有特权，要么没有任何特权。因此，如果我们授予普通用户运行程序的权限，通常我们授予的权限远远超出了所需的权限，而如果黑客破解了应用程序或系统，这可能会成为一个问题。

在 Oracle Solaris 10 中，开发者引入了一个极好的功能，使得权限更加灵活；**最少权限**。基本概念很简单；建议只授予进程、用户或程序所需的最小权限，以减少严重安全漏洞发生时造成的损害。例如，当我们通过应用读取、写入和执行权限来管理文件系统安全时，我们通常会为文件授予比实际需要的更多权限，这是一个大问题。如果我们只授予足够的权限（如简单且个别的权限），以满足角色、用户、命令或甚至进程的需求，那将更为理想。

一个进程有四种权限集合：

+   **有效** (**E**)：表示当前正在使用的权限集合。

+   **继承** (**I**)：表示子进程在执行 `fork()`/`exec()` 调用后可以继承的权限集合。

+   **许可** (**P**)：表示可供使用的权限集合。

+   **限制** (**L**)：表示可以提供给许可集合的所有可用权限。

Oracle Solaris 11 具有几种权限类别，如文件权限、系统权限、网络权限、进程权限和 IPC 权限。这些权限类别（有些人称之为分类）中的每一类都有许多不同的权限，其中一些被选为分配给任何用户的基本权限。

## 准备工作

本教程需要一台运行 Oracle Solaris 11 的虚拟机（VirtualBox 或 VMware），并且至少需要 2 GB 内存。

## 如何操作…

什么是现有的权限？这个问题可以通过查看主页面（`main privileges` 命令）或运行以下命令来回答：

```
root@solaris11-1:~# ppriv -vl | more
contract_event
  Allows a process to request critical events without limitation.
  Allows a process to request reliable delivery of all events on
  any event queue.
contract_identity
  Allows a process to set the service FMRI value of a process
  contract template.
 (truncated output)

```

然而，在所有现有权限中，只有一部分是基本且必需的，用于进程操作：

```
root@solaris11-1:~# ppriv -vl basic
file_link_any
  Allows a process to create hardlinks to files owned by a uid
  different from the process' effective uid.
file_read
  Allows a process to read objects in the filesystem.
file_write
  Allows a process to modify objects in the filesystem.
net_access
  Allows a process to open a TCP, UDP, SDP or SCTP network endpoint.
proc_exec
  Allows a process to call execve().
proc_fork
  Allows a process to call fork1()/forkall()/vfork()
proc_info
  Allows a process to examine the status of processes other
  than those it can send signals to.  Processes which cannot
  be examined cannot be seen in /proc and appear not to exist.
proc_session
  Allows a process to send signals or trace processes outside its
  session.
```

在处理进程权限时，我们可以通过使用 `ppriv` 命令来管理它们。例如，要列出当前 shell 的权限，可以运行以下命令：

```
root@solaris11-1:~# ppriv $$
2590:  bash
flags = <none>
  E: all
  I: basic
  P: all
  L: all
```

我们也可以通过执行`ppriv 2590`得到相同的结果，并且在两种情况下，都可以通过使用`-v`选项获得更详细的输出（`ppriv –v 2590`或`ppriv –v $$`）。此外，这里可能出现两个常见的标志：`PRIV_AWARE`（进程已意识到权限框架）和`PRIV_DEBUG`（进程处于权限调试模式）。

我们已经了解了可能的权限，现在是时候在实际案例中应用这些概念了。例如，如果一个普通用户（上节中的`rbactest`用户）尝试读取`/etc/shadow`文件内容，他们将看不到任何内容，如下命令所示：

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# ls -l /etc/shadow
-r--------   1 root     sys          949 Apr 18 22:57 /etc/shadow
root@solaris11-1:~# su – rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ more /etc/shadow
/etc/shadow: Permission denied
```

如果我们没有合适的解决方案，这可能会对我们造成严重问题，因为我们不希望授予`rbactest`用户任何不必要的权限，但我们需要授予足够的权限来完成读取`/etc/shadow`文件的任务。如果我们将读取权限（R）授予`/etc/shadow`文件中的其他权限组，我们就允许其他用户读取该文件。使用**访问控制列表**（**ACL**）可以解决这个问题，因为我们可以仅为`rbactest`用户授予`/etc/shadow`的读取权限（R），但对于这样一个重要的文件来说，授予过多的权限是危险的。

这个问题的真正解决方法是使用最小权限原则。换句话说，建议只为`rbactest`用户分配查看`/etc/shadow`内容所必需的权限。然而，正确的权限是什么？通过运行`ppriv`命令并使用`–De`选项（调试和执行），可以找到正确的权限，如下命令所示：

```
rbactest@solaris11-1:~$ ppriv -De more /etc/shadow
more[2615]: missing privilege "file_dac_read" (euid = 102, syscall = 69) for "/etc/shadow" needed at zfs_zaccess+0x245
/etc/shadow: Permission denied
```

缺少的权限是`file_dac_read`，其描述如下：

```
rbactest@solaris11-1:~$ ppriv -vl file_dac_read
file_dac_read
  Allows a process to read a file or directory whose permission
  bits or ACL do not allow the process read permission.
```

失败的系统调用可以通过以下命令查看：

```
root@solaris11-1:~# grep 69 /etc/name_to_sysnum
openat64    69
```

通过执行以下命令，获取`mkdirat`系统调用的更多信息是可行的：

```
rbactest@solaris11-1:~$ man openat
System Calls                                              open(2)
NAME
     open, openat - open a file

SYNOPSIS
     #include <sys/types.h>
     #include <sys/stat.h>
     #include <fcntl.h>

     int open(const char *path, int oflag, /* mode_t mode */);

     int openat(int fildes, const char *path, int oflag,
          /* mode_t mode */);

DESCRIPTION
     The open() function establishes  the  connection  between  a
     file and a file descriptor. It creates an open file descrip-
     tion that refers to a file and a file descriptor that refers 
(truncated output)

```

现在我们知道了正确的权限，所以有两种方法可以纠正这种情况：要么直接将`file_dac_read`权限授予`rbactest`用户，要么将其分配给一个角色（例如，上一节中的`r_reboot`）。

要为`rbactest`用户分配权限并将权限分配给角色，请执行以下命令：

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# usermod -R r_reboot rbactest
root@solaris11-1:~# rolemod -K defaultpriv=basic,file_dac_read r_reboot
root@solaris11-1:~# cat /etc/user_attr
#
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;defaultpriv=basic,file_dac_read;profiles=Reboot;roleauth=role
rbactest::::roles=r_reboot 
```

根据前面的步骤，我们已经将`rbactest`用户与`r_reboot`角色关联（如果你之前已经做过），并保留了现有的基本权限。此外，添加了一个新的权限（`file_dac_read`）。要验证配置是否正确，请运行以下命令：

```
root@solaris11-1:~# su - rbactest
Oracle Corporation  SunOS 5.11  11.1  September 2012

rbactest@solaris11-1:~$ su - r_reboot
Password: hacker321!
Oracle Corporation  SunOS 5.11  11.1  September 2012
r_reboot@solaris11-1:~$ profiles
          Reboot
          Basic Solaris User
          All
r_reboot@solaris11-1:~$ more /etc/shadow
root:$5$7X5pLA3o$ZTJJeO.MfVLlBGzJI.yzh3vqhvW.xUWBknCCMHRvP79:16179::::::18384
daemon:NP:6445::::::
bin:NP:6445::::::
sys:NP:6445::::::
adm:NP:6445::::::
(truncated output)

```

它已经生效了！另一种获得相同结果的方法是直接将`file_dac_read`权限授予`rbactest`用户，但这不是推荐的方法：

```
root@solaris11-1:~# id
uid=0(root) gid=0(root)
root@solaris11-1:~# usermod -K defaultpriv=basic,file_dac_read rbactest
root@solaris11-1:~# more /etc/user_attr
# The system provided entries are stored in different files
# under "/etc/user_attr.d".  They should not be copied to this file.
#
# Only local changes should be stored in this file.
# This line should be kept in this file or it will be overwritten.
#
ale::::lock_after_retries=no;profiles=System Administrator;roles=root
r_reboot::::type=role;defaultpriv=basic,file_dac_read;profiles=Reboot;roleauth=role
rbactest::::defaultpriv=basic,file_dac_read;roles=r_reboot

root@solaris11-1:~# su – rbactest
Oracle Corporation	SunOS 5.11  11.1  September 2012
rbactest@solaris11-1:~$ more /etc/shadow
root:$5$oXapLA3o$UTJJeO.MfVlTBGzJI.yzhHvqhvW.xUWBknCCKHRvP79:16179::::::18384
daemon:NP:6445::::::
bin:NP:6445::::::
sys:NP:6445::::::
adm:NP:6445::::::
(truncated output)

```

这也已经生效了！

### 配方概述

在本节中，我们学习了如何使用`pfexec`命令、RBAC 概念和最小权限原则。此外，我们还通过示例展示了如何将这些技术应用于日常管理中。

# 参考资料

+   *RBAC 访问控制*，链接地址：[`docs.oracle.com/cd/E23824_01/html/821-1456/rbac-1.html`](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbac-1.html)

+   *权限*，链接地址：[`docs.oracle.com/cd/E23824_01/html/821-1456/prbac-2.html#scrolltoc`](http://docs.oracle.com/cd/E23824_01/html/821-1456/prbac-2.html#scrolltoc)

+   *查看和使用 RBAC 默认设置*，链接地址：[`docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-new-1.html#scrolltoc`](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-new-1.html#scrolltoc)

+   *为您的站点定制 RBAC*，链接地址：[`docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-30.html#scrolltoc`](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-30.html#scrolltoc)

+   *管理 RBAC*，链接地址：[`docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-4.html#scrolltoc`](http://docs.oracle.com/cd/E23824_01/html/821-1456/rbactask-4.html#scrolltoc)

+   *使用权限*，链接地址：[`docs.oracle.com/cd/E23824_01/html/821-1456/privtask-1.html#scrolltoc`](http://docs.oracle.com/cd/E23824_01/html/821-1456/privtask-1.html#scrolltoc)
