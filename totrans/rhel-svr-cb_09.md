# Chapter 9. Securing RHEL 7

In this chapter, you will learn all about:

*   Installing and configuring IPA
*   Securing the system login
*   Configuring privilege escalation with sudo
*   Securing the network with `firewalld`
*   Using kdump and SysRq
*   Using ABRT
*   Auditing the system

# Introduction

Security is an important aspect of your environment. The recipes provided in this chapter are not a definitive set of how-tos; rather, they are a start to addressing security in an environment as every environment is different. This chapter is meant to give you an idea of what you can do with a simple set of tools included in Red Hat Enterprise Server 7.

In this chapter, I will not attempt explaining where the system stores syslog messages and what they mean as this can be quite an exhaustive topic. The most important security-related syslog messages can be found in `/var/log/secure` and `/var/log/audit/audit.log`.

# Installing and configuring IPA

The **IPA** (**Identity Policy Audit**) server allows you to manage your kerberos, DNS, hosts, users, sudo rules, password policies, and automounts in a central location. IPA is a combination of packages, including—but not limited to—`bind`, `ldap`, `pam`, and so on. It combines all of these to provide identity management for your environment.

## Getting ready

In this recipe, I will opt for an integrated DNS setup, although it is possible to use your existing DNS infrastructure.

## How to do it…

First, we'll install the server component, followed by what needs to be done on an IPA client.

### Installing the IPA server

Follow these instructions to install an IPA server:

1.  Install the necessary packages via the following command:

    ```
    ~]# yum install -y ipa-server bind bind-dyndb-ldap

    ```

2.  When the packages are installed, invoke the `ipa` installer, as follows:

    ```
    ~]# ipa-server-install

    ```

At this stage, you will be asked a couple of questions on how to set up your IPA server.

1.  Configure integrated DNS as follows:

    ```
    Do you want to configure integrated DNS (BIND)? [no]: yes

    ```

2.  Overwrite existing `/etc/resolv.conf` as follows:

    ```
    Existing BIND configuration detected, overwrite? [no]: yes

    ```

3.  Provide the IPA server's hostname, as follows:

    ```
    Server host name [localhost.localdomain]: master.example.com

    ```

4.  Now, confirm the DNS domain name for the IPA server as follows:

    ```
    Please confirm the domain name [example.com]:

    ```

5.  Provide an IP address for the IPA server as follows:

    ```
    Please provide the IP address to be used for this host name: 192.168.0.1

    ```

6.  Next, provide a Kerberos `realm` name, as follows:

    ```
    Please provide a realm name [EXAMPLE.COM]:

    ```

7.  Create the directory manager's password and confirm it as follows:

    ```
    Directory Manager password:

    ```

8.  Create the IPA manager's password and confirm it as follows:

    ```
    IPA admin password:

    ```

9.  Now, configure the DNS forwarders as follows:

    ```
    Do you want to configure DNS forwarders? [yes]: no

    ```

10.  Finally, configure the reverse DNS zones as follows:

    ```
    Do you want to configure the reverse zone? [yes]:
    Please specify the reverse zone name [0.168.192.in-addr.arpa.]:

    ```

    The installer will now provide an overview similar to the following:

    ```
    The IPA Master Server will be configured with:
    Hostname:      master.example.com
    IP address:    192.168.0.1
    Domain name:   example.com
    Realm name:    EXAMPLE.COM

    BIND DNS server will be configured to serve IPA domain with:
    Forwarders:    No forwarders
    Reverse zone:  0.168.192.in-addr.arpa.

    ```

11.  Now, confirm the information by typing "yes", as follows:

    ```
    Continue to configure the system with these values? [no]: yes

    ```

At this point, you will see a lot of information scrolling on your screen, indicating what the installer is doing: installing or configuring NTP, LDAP, BIND, Kerberos, HTTP, the certificate server, and IPA-related modifications to the preceding examples.

The installation and configuration process can take a while, so be patient.

### Installing the IPA client

Perform these steps to install and configure the IPA client on your system:

### Tip

Ensure that the hostname of your system is different from `localhost.localdomain`. If it is not, the client configuration will fail.

1.  Install the necessary packages via the following command:

    ```
    ~]# yum install -y ipa-client

    ```

2.  Ensure that the IPA server is used as a DNS server through the following:

    ```
    ~]# cat /etc/resolv.conf
    search example.com
    nameserver 192.168.0.1

    ```

3.  Invoke the IPA client configuration by running this command line:

    ```
    ~]# ipa-client-install --enable-dns-updates

    ```

The installer will now show an overview of the detected IPA server and ask for a user (the IPA manager) and password to register your system, as shown in the following screenshot:

![Installing the IPA client](img/00060.jpeg)

## There's more…

Once installed, you can manage your IPA environment using the command line tool IPA or the web interface, which can be accessed by pointing your browser to your IPA master server over HTTPS. In this case, the URL is `https://master.example.com`.

By default, the IPA client doesn't create `homedirs` for new users at first login. If you want to enable this, use the `--mkhomedir` argument with `ipa-client-install`. If you happen to have forgotten about this, there's no need to reinstall the IPA client. You can just reconfigure this by executing the following command:

```
~]# authconfig --enablemkhomedir --update

```

## See also

For more in-depth information about installing and configuring your IPA server, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/installing-ipa.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/installing-ipa.html).

For more information about managing your IPA environment through the command line, read the *ipa (1)* man pages.

# Securing the system login

The default settings applied to system login are based on what Red Hat deems basic security. If, for some reason, you want to change this, this recipe will show you a couple of examples. Authconfig has two tools that you can use to configure authentication: `authconfig` and `authconfig-tui`.

These two tools configure `pam` for you in such a way that the changes are consistent throughout rpm updates.

The `authconfig-tui` tool is not as feature-rich as the plan `authconfig` tool, which I personally recommend you to use as it allows you to do more.

You can manually edit the files located in `/etc/pam.d` if and when you know what you're doing, but this is not recommended.

## How to do it…

Perform the following steps:

First, change the hash encryption of the passwords stored in `/etc/shadow` to `sha512`, as follows:

```
~]# authconfig --passalgo=sha512 --update

```

Enable NIS authentication through the following command:

```
~]# authconfig --enablenis –nisdomain=NISDOMAIN --nisserver=nisserver.example.com --update

```

Now, set the minimum length requirement for passwords to `16` via the following:

```
~]# authconfig --passminlen=16 --update

```

The user requires at least one lowercase letter in the password; you can set this requirement by running the following:

```
~]# authconfig --enablereqlower --update

```

Also, the user requires at least one uppercase letter in the password, for which you can run the following:

```
~]# authconfig --enablerequpper --update

```

Now, the user requires at least one number in the password. Execute the following command for this:

```
~]# authconfig --enablereqdigit --update

```

Finally, the user requires at least one nonalphanumeric character in the password, which you can set using the following command:

```
~]# authconfig --enablereqother --update

```

## How it works…

`authconfig` and `authconfig-tui` are wrapper scripts that modify a variety of files, including, but not limited to, `/etc/nsswitch.conf`, `/etc/pam.d/*`, `/etc/sssd.conf`, `/etc/openldap/ldap.conf`, and `/etc/sysconfig/network`.

The advantage of the tool is that it uses the correct syntax, which can sometimes be a little tricky, especially for the files in `/etc/pam.d`.

## There's more…

One of the interesting features of this tool is the backup and restore functions. In case you do not use any centralized identification and authentication infrastructure, such as IPA, you can use this to make a backup of a correctly configured machine and distribute this through whichever means you wish to use.

To back up your `authconf` configuration, execute the following:

```
~]# authconfig --savebackup=/tmp/auth.conf

```

This will create a `/tmp/auth.conf` directory, which contains all the files modified by `authconfig`.

Copy this directory over to another server and restore the configuration by executing the following:

```
~]# authconfig –-restorebackup=/tmp/auth.conf

```

All of the security changes you apply through `authconfig` can also be managed through IPA.

## See also

For information about and more configuration options, take a look at the *authconfig (8)* man pages.

You can also find more information on Red Hat's page on authentication at [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/Configuring_Authentication.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/Configuring_Authentication.html).

# Configuring privilege escalation with sudo

Sudo allows users to run applications and scripts with the security privileges of another user.

## Getting ready

Before allowing someone to elevate their security context for a specific application or script, you need to figure out which user or group you wish to elevate from and to, which applications/scripts you use, and on which systems to run them.

The default syntax for a sudo entry is the following:

```
who where = (as_whom) what

```

## How to do it…

These simple five steps will guide you through setting up privilege escalation:

1.  Create a new `sudoers` definition file in `/etc/sudoers.d/` called clustering through the following command:

    ```
    ~]# visudo -f /etc/sudoers.d/clustering

    ```

2.  Create a command alias for the most-used clustering tools called `CLUSTERING` by executing the following:

    ```
    Cmnd_Alias CLUSTERING = /sbin/ccs, /sbin/clustat, /sbin/clusvcadm
    ```

3.  Now, create a host alias group for all the clusters called `CLUSTERS`, as follows:

    ```
    Host_Alias CLUSTERS = cluster1, cluster2
    ```

4.  Next, create a user alias for all cluster admins called `CLUSTERADMINS` by executing the following:

    ```
    User_Alias CLUSTERADMINS = spalpatine, dvader, okenobi, qjinn
    ```

5.  Now, let's create a sudo rule that allows the users from `CLUSTERADMINS` to execute commands from `CLUSTERING` on all servers within the `CLUSTERS` group, as follows:

    ```
    CLUSTERADMINS CLUSTERS = (root) CLUSTERING
    ```

## There's more…

To edit the `sudoers` file, you can either use a text editor and edit `/etc/sudoers`, the `visudo` tool, which automatically checks your syntax when exiting.

It's always a good idea to leave the original `/etc/sudoers` file alone and modify the files located in `/etc/sudoers.d/`. This allows the sudo rpm to update the `sudoers` file should it be necessary.

## See also

For more information about sudo, take a look at the *sudoers (5)* man page.

# Secure the network with firewalld

`firewalld` is a set of scripts and a daemon that manage `netfilter` on your RHEL system. It aims at creating a simple command-line interface to manage the firewall on your systems.

## How to do it…

By default, `firewalld` is included in the "core" rpm group, but it may not be installed for some reason (that you left it out of your kickstart would be one!). Perform the following steps:

1.  Install `firewalld` via the following command line:

    ```
    ~]# yum install -y firewalld

    ```

2.  Now, enable `firewalld` through the following:

    ```
    ~]# systemctl enable firewalld

    ```

3.  Finally, ensure that `firewalld` is started by executing the following command line:

    ```
    ~]# systemctl restart firewalld

    ```

### Showing the currently allowed services and ports on your system

List all the allowed services using the following command:

```
~]# firewall-cmd –list-services

```

You can see the output as follows, where all the allowed services are listed:

![Showing the currently allowed services and ports on your system](img/00061.jpeg)

Now, show the `tcp`/`udp` ports that are allowed by your firewall using the following command:

```
~]# firewall-cmd --list-ports

```

Here's what the output should look like:

![Showing the currently allowed services and ports on your system](img/00062.jpeg)

### Allowing incoming requests for NFS (v4)

Perform the following steps to allow NFSv4 traffic on your system:

1.  First, allow `nfs` traffic via this command:

    ```
    ~]# firewall-cmd --add-service nfs –-permanent
    success
    ~]#

    ```

2.  Then, reload the configuration as follows:

    ```
    ~]# firewall-cmd --reload
    success
    ~]#

    ```

3.  Now, check the newly applied rule by executing the following command line:

    ```
    ~]# firewall-cmd –-list-services
    nfs
    ~]#

    ```

### Allowing incoming requests on an arbitrary port

Perform the following steps to allow incoming traffic on port `1234` over both `tcp` and `udp`:

1.  First, allow traffic on port `1234` over `tcp` and `udp` by running the following:

    ```
    ~]# firewall-cmd --add-port 1234/tcp --permanent
    success
    ~]# firewall-cmd --add-port 1234/udp --permanent
    success
    ~]#

    ```

2.  Reload the configuration by executing the following command:

    ```
    ~]# firewall-cmd –-reload
    success
    ~]#

    ```

3.  Check the newly applied rule via the following:

    ```
    ~]# firewall-cmd –-list-ports
    1234/tcp 1234/udp
    ~]#

    ```

## There's more…

`firewalld` comes with a set of predefined port configurations, such as HTTP and HTTPS. You can find all such definitions in `/lib/firewalld/services`. When creating your own port definitions or modifying the existing ones, you should create new port definition files in `/etc/firewalld/services`.

When creating new "rules" by adding ports, services, and so on, you need to add the `--permanent` option, or your changes would be lost upon the rebooting of the system or the reloading of the `firewalld` policy.

## See also

For more information on configuring your firewall, check the man pages for *firewall-cmd(1)*.

# Using kdump and SysRq

The kdump mechanism is a Linux kernel feature, which allows you to create dumps if your kernel crashes. It produces an exact copy of the memory, which can be analyzed for the root cause of the crash.

SysRq is a feature supported by the Linux kernel, which allows you to send key combinations to it even when your system becomes unresponsive.

## How to do it…

First, we'll set up kdump and SysRq, and afterwards, I'll show you how to use it to debug a dump.

### Installing and configuring kdump and SysRq

Let's take a look at how this is installed and configured:

1.  Install the necessary packages for kdump by executing the following command:

    ```
    ~]# yum install -y kexec-tools

    ```

2.  Ensure that `crashkernel=auto` is present in the `GRUB_CMDLINE_LINUX` variable declaration in the `/etc/sysconfig/grub` file using this command:

    ```
    GRUB_CMDLINE_LINUX="rd.lvm.lv=system/usr rd.lvm.lv=system/swap vconsole.keymap=us rd.lvm.lv=system/root vconsole.font=latarcyrheb-sun16 crashkernel=auto"
    ```

3.  Start `kdump` by running the following:

    ```
    ~]# systemctl start kdump

    ```

4.  Now, enable `kdump` to start at boot, as follows:

    ```
    ~]# sysctl enable kdump

    ```

5.  Configure SysRq to accept all commands via the following commands:

    ```
    ~]# echo "kernel.sysrq = 1" >> /etc/sysctl.d/sysrq.conf
    ~]# systemctl -q -p /etc/sysctl.d/sysrq.conf

    ```

6.  Regenerate your **intramfs** (**initial RAM file system**) to contain the necessary information for kdump by executing the following command:

    ```
    ~]# dracut --force

    ```

7.  Finally, reboot through the following command:

    ```
    ~]# reboot

    ```

### Using kdump tools to analyze the dump

Although you'll find most of the information you're looking for in the `vmcode-dmesg.txt` file, it can be useful sometimes to look into the bits and bytes of the `vmcore` dump, even if it is just to know what the people at Red Hat do when they ask you to send you a `vmcore` dump. Perform the following steps:

1.  Install the necessary tools to debug the `vmcore` dump via the following command:

    ```
    ~]# yum install -y --enablerepo=\*debuginfo crash kernel-debuginfo

    ```

2.  Locate your `vmcore` by executing the following:

    ```
    ~]# find /var/crash -name 'vmcore'
    /var/crash/127.0.0.1-2015.10.31-12:03:06/vmcore

    ```

    ### Note

    If you don't have a core dump, you can trigger this yourself by executing the following:

    ```
    ~]# echo c > /proc/sysrq-trigger

    ```

3.  Use `crash` to analyze the contents, as follows:

    ```
    ~]# crash /var/crash/127.0.0.1-2015.10.31-12:03:06/vmcore /usr/lib/debug/lib/modules/<kernel>/vmlinux

    ```

    Here, `<kernel>` must be the same kernel as the one that the dump was created for:

    ![Using kdump tools to analyze the dump](img/00063.jpeg)
4.  Display the kernel message buffer (this can also be found in the `vmcore-dmesg.txt` dump file) by running the following command:

    ```
    crash> log

    ```

    Here's what the output should look like:

    ![Using kdump tools to analyze the dump](img/00064.jpeg)
5.  Display the kernel stack trace through the following:

    ```
    crash> bt

    ```

    Here's what the output should look like:

    ![Using kdump tools to analyze the dump](img/00065.jpeg)
6.  Now, show the processes at the time of the core dump, as follows:

    ```
    crash> ps

    ```

    Here's what the output should look like:

    ![Using kdump tools to analyze the dump](img/00066.jpeg)

## There's more…

The default kdump configuration uses `/var/crash` to dump its memory on. This MUST be on the root filesystem. Some systems are configured with a separate filesystem for `/var`, so you need to change the location in `/etc/kdump.conf` or use a different target type, such as `raw`, `nfs`, and so on. If your crash directory is located on a nonroot filesystem, the kdump service will fail!

Although the crash utility can provide a lot of details about the crash, usually you're set with the contents of the `vmcore-dmesg.txt` file, which resides in the same directory as the `vmcore` file. So, I suggest that you parse this file before digging into the bits and bytes of the memory dump.

SysRq, as stated before, allows you to control your system even if it is in a state that doesn't allow you to do anything at all. However, it does require you to have access to the system's console.

By default, kdump creates a dump and reboots your system. In the event that this doesn't happen and you don't want to push the power button on your (virtual) system, SysRq allows you to send commands through the console to your kernel.

The key combination needed to send the information differs a little from architecture to architecture. Take a look at the following table for reference:

| Architecture | Key combination |
| --- | --- |
| x86 | `<Alt><SysRq><command key>` |
| Sparc | `<Alt><Stop><command key>` |
| Serial console (PC style only) | This sends a `BREAK` and, within 5 seconds, the command key.Sending `BREAK` twice is interpreted as a normal `BREAK`. |
| PowerPC | `<Alt><Print Screen>`(or `<F13>`)`<command key>` |

So, on an x86 system, you would attempt to sync your disks before rebooting it by executing the following commands:

```
<Alt><SysRq><s>
<Alt><SysRq><b>

```

Alternatively, if you still have access to your terminal, you can do the same by sending characters to `/proc/sysrq-trigger`, as follows:

```
~]# echo s > /proc/sysrq-trigger
~]# echo b > /proc/sysrq-trigger

```

The following key commands are available:

| Command key | Function |
| --- | --- |
| `b` | This immediately reboots your system. It does not sync or unmount disks. This can result in data corruption! |
| `c` | This performs a system crash by a `NULL` pointer dereference. A crashdump is taken if kdump is configured. |
| `d` | This shows all the locks held. |
| `e` | This sends a `SIGTERM` signal to all your processes, except for `init`. |
| `f` | This calls `oom_kill` to kill any process hogging the memory. |
| `g` | This is used by the **kernel debugger** (**kgdb**). |
| `h` | This shows help. (Memorize this option!) |
| `i` | This sends a `SIGKILL` signal to all your processes, except for `init`. |
| `j` | This freezes your filesystems with the `FIFREEZE` ioctl. |
| `k` | This kills all the programs on the current virtual console.It enables a secure login from the console as this kills all malware attempting to grab your keyboard input, for example. |
| `l` | This shows a stack trace for all active CPUs. |
| `m` | This dumps the current memory info to your console. |
| `n` | You can use this to make real-time tasks niceable. |
| `o` | This shuts down your system and turns it off (if configured and supported). |
| `p` | This dumps the current registers and flags to your console |
| `q` | This will dump a list of all armed `hrtimers` (except for `timer_list` timers) per CPU together with detailed information about all clockevent devices. |
| `r` | This turns off your keyboard's raw mode and sets it to `XLATE`. |
| `s` | This attempts to sync all your mounted filesystems, committing unwritten data to them. |
| `t` | This dumps a list of current tasks and their information to your console. |
| `u` | This attempts to remount all your filesystems as read-only volumes. |
| `v` | This causes the ETM buffer to dump (this is ARM-specific). |
| `w` | This dumps all the tasks that are in an uninterruptable (blocked) state. |
| `x` | This is used by xmon on ppc/powerpc platforms. This shows the global PMU registers on SPARC64. |
| `y` | This shows global CPU registers (this is SPARC64-specific). |
| `z` | This dumps the `ftrace` buffer. |
| `0` - `9` | This sets the console's log level, controlling which messages will be printed. The higher the number, the more the output. |

## See also…

For more information about SysRq and systemd, refer to the following page: [https://github.com/systemdaemon/systemd/blob/master/src/linux/Documentation/sysrq.txt](https://github.com/systemdaemon/systemd/blob/master/src/linux/Documentation/sysrq.txt)

Red Hat has a complete crash dump guide at [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Kernel_Crash_Dump_Guide/index.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Kernel_Crash_Dump_Guide/index.html).

# Using ABRT

**ABRT** (**Automatic Bug Reporting Tool**), is a set of tools that help users detect and analyze application crashes.

## How to do it…

First, we'll install the necessary packages and then take a look at how to use these tools.

### Installing and configuring abrtd

Let's install `abrt` and get it running:

1.  Install the `abrt` daemon and tools via the following command line:

    ```
    ~]# yum install -y abrt-cli

    ```

2.  Now, enable and start the `abrt` daemon through these commands:

    ```
    ~]# systemctl enable abrtd
    ~]# systemctl restart abrtdThere's more...

    ```

### Using abrt-cli

List all detected segmentation faults by executing the following command:

```
~]# abtr-cli list

```

Here's what the output should look like:

![Using abrt-cli](img/00067.jpeg)

The displayed location contains all the information about the segmentation fault. You can use this to analyze what went wrong, and if you need help from Red Hat, you can use `abrt-cli report` to report to Red Hat Support.

## There's more…

When your RHEL 7 system is registered with a satellite, all bugs will automatically be reported to the satellite system.

You can install additional plugins to automatically report bugs in the following ways:

*   to Bugzilla (`libreport-plugin-bugzilla`)
*   via ftp upload (`libreport-plugin-reportuploader`)
*   to Red Hat Support (`libreport-plugin-rhtsupport`)
*   to an `abrt` server (`libreport-plugin-ureport`)

Besides the basic bug reporting, you can also create automatic bug reports for Java by installing the `abrt-java-connector` package.

## See also

For more information on how to use the abrt tool, refer to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-abrt.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-abrt.html).

# Auditing the system

The Linux audit system allows you to track security-related information about your systems. It allows you to watch security events, filesystem access, network access, commands run by users, and system calls.

## How to do it…

By default, audit is installed as part of the core packages. So, there's no need to install this.

### Configuring a centralized syslog server to accept audit logs

Perform these steps to set up the `syslog` server:

1.  On the `syslog` server, create a `/etc/rsyslog.d/audit_server.conf` file containing the following:

    ```
    # Receive syslog audit messages via TCP over port 65514
    $ModLoad imtcp
    $InputTCPServerRun 65514
    $AllowedSender TCP, 127.0.0.1, 192.168.1.0/24
    $template HostAudit, "/var/log/audit/%$YEAR%%$MONTH%%$DAY%-%HOSTNAME%/audit.log"
    $template auditFormat, "%msg%\n" local6.*  ?HostAudit;auditFormat
    ```

2.  On the `syslog` server, restart `rsyslog`, as follows:

    ```
    ~]# systemctl restart rsyslog

    ```

3.  On the client, create a `/etc/rsyslog.d/audit_client.conf` file containing the following:

    ```
    $ModLoad imfile
    $InputFileName /var/log/audit/audit.log
    $InputFileTag tag_audit_log:
    $InputFileStateFile audit_log
    $InputFileFacility local6
    $InputFileSeverity info
    $InputRunFileMonitor local6.* @@logserver.example.com:65514
    ```

4.  Next, on the client, restart `rsyslog`, as follows:

    ```
    ~]# systemctl restart syslog

    ```

### Some audit rules

You can use the following command to log activity on `/etc/resolv.conf`:

```
~]# auditctl -w /etc/resolv.conf -p w -k resolv_changes

```

You can execute the following commands to log all the commands executed by root:

```
~]# echo "session   required pam_tty_audit.so disable=* enable=root" >> /etc/pam.d/system-auth-ac
~]# echo "session   required pam_tty_audit.so disable=* enable=root" >> /etc/pam.d/password-auth-ac

```

### Showing audit logs for the preceding rules

You can search for the audit events that have changed `/etc/resolv.conf` using the preceding rule by executing the following command:

```
~]# ausearch -k resolv_changes

```

Here's what the output should look like:

![Showing audit logs for the preceding rules](img/00068.jpeg)

To check all the commands executed by root today, you can run the following:

```
~]# aureport --tty -ts today

```

Here's what the output should look like:

![Showing audit logs for the preceding rules](img/00069.jpeg)

## See also

For more in-depth information about audit, refer to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/chap-system_auditing.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/chap-system_auditing.html).