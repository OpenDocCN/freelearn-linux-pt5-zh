# Chapter 4. Configuring Your New System

Here's an overview of the recipes that we'll be covering in this chapter:

*   The `systemd` service and setting runlevels
*   Starting and stopping `systemd` services
*   Configuring the `systemd` journal for persistence
*   Monitoring services using `journalctl`
*   Configuring `logrotate`
*   Managing time
*   Configuring your boot environment
*   Configuring `smtp`

# Introduction

Once your system is installed and the network is configured, it's time to start configuring everything else.

RHEL 7 comes with the `systemd init` daemon, which takes care of your daemon or service housekeeping and more, replacing the old SysV (UNIX System V) init system.

Its main advantages are automatic dependency handling, parallel startup of services, and the monitoring of started services with the ability to restart crashed services.

For a good read on `systemd` and its inner workings, head over to [https://n0where.net/understanding-systemd](https://n0where.net/understanding-systemd).

# The systemd service and setting runlevels

The `systemd` service doesn't use runlevels as SysV or Upstart do. The alternatives for `systemd` are called targets. Their purpose is to group a set of `systemd` units (not only services, but also sockets, devices, and so on) through a chain of dependencies.

## How to do it…

Managing targets with `systemd` is pretty simple, as shown through the following steps:

1.  List all target units, as follows:

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

    This list shows all target units available followed by information regarding whether the target is enabled or not.

2.  Now, show the currently loaded target units.

    The `systemd` targets can be chained unlike SysV runlevels, so you'll not only see one target but a whole bunch of them, as follows:

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

3.  Next, change the default `systemd` target by running the following commands:

    ```
    ~]# systemctl set-default graphical.target
    rm '/etc/systemd/system/default.target'
    ln -s '/usr/lib/systemd/system/graphical.target' '/etc/systemd/system/default.target'
    ~]#

    ```

## There's more…

Sometimes, you want to change targets on the fly as you would in the past with runlevel or telinit. With `systemd`, this is accomplished in the following way:

```
~]# systemctl isolate <target name>

```

Here's an example:

```
~]# systemctl isolate graphical.target

```

Let's take an overview of the former runlevels versus the `systemd` targets in the following table:

| Runlevel | Target units | Description |
| --- | --- | --- |
| `0` | `runlevel0.target` or `poweroff.target` | This is used to shut down and power off the system |
| `1` | `runlevel1.target` or `rescue.target` | This is used to enter a rescue shell |
| `2` | `runlevel2.target` or `multi-user.target` | This is used to set up a command-line multiuser system |
| `3` | `runlevel3.target` or `multi-user.target` | This is used to set up a command-line multiuser system |
| `4` | `runlevel4.target` or `multi-user.target` | This is used to set up a command-line multiuser system |
| `5` | `runlevel5.target` or `graphical.target` | This is used to set up a graphical multiuser system |
| `6` | `runlevel6.target` or `reboot.target` | This is used to reboot the system |

## See also

For more in-depth information about RHEL 7 and `systemd` targets, refer to the following link: [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Targets.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Targets.html)

# Starting and stopping systemd services

Although this recipe uses services by their base name, they can also be addressed by their full filename. For example, `sshd` can be substituted by `sshd.service`.

## How to do it…

The following steps need to be performed to successfully start or stop `systemd` services:

1.  List all available `systemd` services, as follows:

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

    This shows all service units available followed by information regarding whether the service is enabled or not.

2.  Now, list all the loaded `systemd` services and their status, as follows:

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

3.  Next, get the status of a service.

    To get the status of a particular service, execute the following, substituting `<service>` with the name of the service:

    ```
    ~]# systemctl status <service>

    ```

    Here's an example:

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

4.  Now, start and stop the `systemd` services.

    To stop a `systemd` service, execute the following, substituting `<service>` with the name of the service:

    ```
    ~]# systemctl stop <service>

    ```

    Here's an example:

    ```
    ~]# systemctl stop sshd

    ```

    To start a `systemd` service, execute the following, substituting `<service>` with the name of the service:

    ```
    ~]# systemctl start <service>

    ```

    Here's an example:

    ```
    ~]# systemctl start sshd

    ```

5.  Next, enable and disable the `systemd` services.

    To enable a `systemd` service, execute the following, substituting `<service>` with the name of the service:

    ```
    ~]# systemctl enable <service>

    ```

    Here's an example:

    ```
    ~]# systemctl enable sshd
    ln -s '/usr/lib/systemd/system/sshd.service' '/etc/systemd/system/multi-user.target.wants/sshd.service'
    ~]#

    ```

    To disable a `systemd` service, execute the following, substituting `<service>` with the name of the service:

    ```
    ~]# systemctl disable <service>

    ```

    Here's an example:

    ```
    ~]# systemctl disable sshd
    rm '/etc/systemd/system/multi-user.target.wants/sshd.service'
    ~]#

    ```

6.  Now, configure a service to restart when crashed.

    Let's make the `ntpd` service restart if it crashes after 1 minute.

    1.  First, create the directory, as follows: `/etc/systemd/system/ntpd.service.d`.

        ```
        ~]# mkdir -p /etc/systemd/system/ntpd.service.d

        ```

    2.  Create a new file in that directory named `restart.conf` and add the following to it:

        ```
        [Service]
        Restart=on-failure
        RestartSec=60s
        ```

    3.  Next, reload the unit files and recreate the dependency tree using the following command:

        ```
        ~]# systemctl daemon-reload

        ```

    4.  Finally, restart the `ntpd` service by executing the following command:

        ```
        ~]# systemctl restart ntpd

        ```

## There's more…

When requesting the status of a service, the most recent log entries are also shown when executed as `root`.

The service status information can be seen in the following table:

| Field | Description |
| --- | --- |
| `Loaded` | This provides information on whether the service is loaded and enabled. It also includes the absolute path to the service file. |
| `Active` | This provides information on whether the service is running, followed by the time it started. |
| `Main PID` | This provides PID of the corresponding service, followed by its name. |
| `Status` | This provides information about the corresponding service. |
| `Process` | This provides information about the related process. |
| `Cgroup` | This provides information about related control groups. |

In some (rare) cases, you want to prevent a service from being started, either manually or by another service; there is an option to mask the service, which is as follows:

```
~]# systemctl mask <service>

```

To unmask, execute the following:

```
~]# systemctl unmask <service>

```

When modifying service unit files (and this is not limited to services only), it is best practice to copy the original service file, which is located at `/lib/systemd/system` to `/etc/systemd/service`. Alternatively, you can create a directory in `/etc/systemd/service` appended with `.d`, in which you will create `conf` files containing only the directives that you wish to add or change, as in the previous recipe. The advantage of the latter is that you don't need to keep up with changes in the original service file as it will be "updated" with whatever is located in the `service.d` directory.

## See also

For more information about managing `systemd` services, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/sect-Managing_Services_with_systemd-Services.html).

# Configuring the systemd journal for persistence

By default, the journal doesn't store log files on disk, only in memory or the `/run/log/journal` directory. This is sufficient for the recent log history (with the journal) but not for long-term log retention should you decide to go with journal only and not with any other `syslog` solution.

## How to do it…

Configuring `journald` to keep more logs than memory allows is fairly simple, as follows:

1.  Open `/etc/systemd/journald.conf` with your favorite text editor with root permissions by executing the following command:

    ```
    ~]# vim /etc/systemd/journald.conf

    ```

2.  Ensure that the line containing `Storage` is either remarked or set to `auto` or `persistent` and save it, as follows:

    ```
    Storage=auto
    ```

3.  If you select `auto`, the journal directory needs to be manually created. The following command would be useful for this:

    ```
    ~]# mkdir -p /var/log/journal

    ```

4.  Now, restart the journal service by executing the following command:

    ```
    ~]# systemctl restart systemd-journald

    ```

## There's more…

There are many other options that can be set for the journal daemon.

By default, all the data stored by `journald` is compressed, but you could disable this using `Compress=no`.

It is recommended to limit the size of the journal files by either specifying a maximum retention age (`MaxRetentionSec`), a global maximum size usage (`SystemMaxUse`), or a maximum size usage per file (`SystemMaxFileSize`).

## See also

For more information about using the journal with RHEL 7, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html).

Take a look at the man page for *journald (5)* for more information on what can be configured.

# Monitoring services using journalctl

Systemd's journal has the added advantage that its controls allow you to easily narrow down on messages generated by specific services.

## How to do it…

Here are the steps you need to perform for this recipe:

1.  First, display all the messages generated by your system.

    This will show all the messages generated on the system; run the following commands:

    ```
    ~]# journalctl
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:30:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpuset
    ...
    ~]#

    ```

2.  Now, display all system-related messages.

    This command shows all the messages related to the system and not its users:

    ```
    ~]# journalctl –-system
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Sat 2015-07-25 00:30:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan systemd-journal[106]: Runtime journal is using 8.0M (max 396.0M, leaving 594.0M of free 3.8G, current limit 396.0M).
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: Initializing cgroup subsys cpuset
    ...
    ~]#

    ```

3.  Display all the current user messages.

    This command shows all messages related to the user that you are logged on with:

    ```
    ~]# journalctl --user
    No journal files were found.
    ~]#

    ```

4.  Next, display all messages generated by a particular service using the following command line:

    ```
    ~]# journalctl --unit=<service>

    ```

    Here's an example:

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

5.  Now, display messages by priority.

    Priorities can be specified by a keyword or number, such as `debug` (7), `info` (6), `notice` (5), `warning` (4), `err` (3), `crit` (2), `alert` (1), and `emerg` (0). When specifying a priority, this includes all the lower priorities as well. For example, `err` implies that `crit`, `alert`, and `emerg` are also shown. Take a look at the following command line:

    ```
    ~]# journalctl -p <priority>

    ```

    Here's an example:

    ```
    ~]# journalctl -p err
    -- Logs begin at Fri 2015-06-26 23:37:30 CEST, end at Fri 2015-07-24 22:30:01 CEST. --
    Jun 26 23:37:30 rhel7.mydomain.lan kernel: ioremap error for 0xdffff000-0xe0000000, requested 0x10, got 0x0
    Jun 26 23:38:49 rhel7.mydomain.lan systemd[1]: Failed unmounting /usr.
    ...
    ~]#

    ```

6.  Next, display messages by time.

    You can show all messages from the current boot through the following commands:

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

    You can even show all the messages within a specific time range by running the following:

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

## There's more…

The examples presented in this recipe can all be combined. For instance, if you want to show all the error messages between 8:00 and 9:00 on 2015-07-24, your command would be the following:

```
~]# journalctl -p err --since="2015-07-24 08:00:00" --until="2015-07-24 09:00:00"

```

A lot of people tend to "follow" log files to determine what is happening, hoping to figure out any issues. The `journalctl` binary is an executable one, so it is impossible to use the traditional "following" techniques such as `tail –f` or using `less` and pressing *CTRL* + *F*. The good folks that coded `systemd` and `systemctl` have provided a solution to this: simply add `-f` or `--follow` as an argument to the `journalctl` command.

Although most environments are used to create `syslog` messages to troubleshoot, the journal does provide the added value of being able to create simple filters that allow you to monitor their messages live.

## See also

For more information about using the journal with RHEL 7, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-Using_the_Journal.html).

Take a look at the man page of *journalctl (1)* for more information on what can be configured.

# Configuring logrotate

The `logrotate` tool allows you to rotate the logs that are generated by applications and scripts

It keeps your log directories clutter-free and minimizes disk usage when correctly configured.

## How to do it…

The `logrotate` tool is installed by default, but I will include the installation instructions here for completeness. This recipe will show you how to rotate logs for `rsyslog`. We will rotate the logs everyday, add an extension based on the date, compress them with a one-day delay, and keep them for 365 days. Perform the following steps:

1.  First, to install `logrotate`, perform the following command:

    ```
    ~]# yum install -y logrotate

    ```

2.  Ensure that it's enabled through the following:

    ```
    ~]# systemctl restart crond

    ```

3.  Open `/etc/logrotate.d/syslog` with your favorite editor. The contents of this file are the following, by default:

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

4.  Now, replace this with the following code:

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

5.  Finally, save the file.

## How it works…

The `logrotate` tool is a script that is launched by cron everyday.

The directives added to the default `logrotate` definition are `compress`, `daily`, `delaycompress`, `dateext`, `missingok`, and `rotate`.

The `compress` directive compresses old versions of the log files with gzip. This behavior is somewhat changed by specifying `delaycompress`. This causes us to always have the most recently rotated log file available uncompressed.

The `daily` directive makes `logrotate` execute the definition every day. The `rotate` directive only keeps `x` rotated log files before deleting the oldest. In this case, we have specified this to be 365, which means that while rotating daily, the logs are kept for 365 days.

The `missingok` directive makes it alright for `syslog` to not create a file, which, however unlikely, is possible.

The `dateext` directive appends a date to the rotated file in the form of `yyyymmdd` instead of a number, which is the default.

## There's more…

The `/etc/logrotate.conf` file contains the defaults directives for all definitions. If you don't specifically use a directive within a definition for a file, the values in this file will be used if specified.

It would make sense to change the settings in this file so that all the definitions are affected, but this is not practical; not all log files are made equal. The `syslog` service generates a lot of messages, and it would probably clutter up your system before long. However, yum, for instance, doesn't generate a lot of messages, and it keeps this log file readable for much longer than your `syslog` files. This, by the way, is reflected in the definition for yum.

If you want to debug your new configuration, this can be achieved by executing the following to test just one configuration:

```
~# /usr/sbin/logrotate -v /etc/logrotate.d/<config file>

```

Alternatively, you can use the following to test everything:

```
~]# /usr/sbin/logrotate -v /etc/logrotate.conf

```

Here's an example:

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

## See also

Take a look at the man page of *logrotate (8)* for more information on configuring `logrotate`.

# Managing time

RHEL 7 comes preinstalled with Chrony. While everybody knows Ntpd, Chrony is a newcomer to the game of timekeeping.

Chrony is a set of programs that maintains the time on your computer using different time sources, such as NTP servers, your system's clock, and even custom-made scripts/programs. It also calculates the rate at which the computer loses or gains time to compensate while no external reference is present—for example, if your NTP server(s) is(are) down.

Chrony is a good solution for systems which are intermittently disconnected and reconnected to a network.

Ntpd should be considered for systems that are normally kept on permanently.

## How to do it…

When talking about managing time in RHEL, it can be done through:

*   Chrony
*   Ntpd

We'll take a look at each of the methods separately.

### Managing time through chrony

Ensure that `chrony` is installed and enabled, and perform the following steps:

1.  First, install `chrony` through the following command:

    ```
    ~]# yum install -y chrony

    ```

2.  Enable `chrony`, as follows:

    ```
    ~]# systemctl enable chrony
    ~]# systemctl start chrony

    ```

3.  Now, open `/etc/chrony.conf` with your favorite editor and look for lines starting with the `server` directive using the following commands:

    ```
    server 0.rhel.pool.ntp.org iburst
    server 1.rhel.pool.ntp.org iburst
    server 2.rhel.pool.ntp.org iburst
    server 3.rhel.pool.ntp.org iburst
    ```

4.  Next, replace these lines with NTP servers that are near you and save the file:

    ```
    server 0.pool.ntp.mydomain.lan iburst
    server 1.pool.ntp.mydomain.lan iburst
    ```

    The `iburst` option causes NTP to send a burst of eight packets at the next poll instead of just one if the time master is unavailable, causing the NTP daemon to speed up time synchronization.

5.  Finally, restart `chrony` by executing the following command:

    ```
    ~]# systemctl restart chrony

    ```

### Managing time through ntpd

Ensure that `ntpd` is installed and enabled, and perform the following steps:

1.  First, install `ntpd` by running the following:

    ```
    ~]# yum install -y ntpd

    ```

2.  Enable `ntpd` through this command:

    ```
    ~]# systemctl enable ntpd

    ```

3.  Open `/etc/ntp.conf` with your favorite editor and look for the lines starting with the `server` directive. Run the following:

    ```
    server 0.rhel.pool.ntp.org iburst
    server 1.rhel.pool.ntp.org iburst
    server 2.rhel.pool.ntp.org iburst
    server 3.rhel.pool.ntp.org iburst
    ```

4.  Replace these lines with the NTP servers near you and save the file:

    ```
    server 0.pool.ntp.mydomain.lan iburst
    server 1.pool.ntp.mydomain.lan iburst
    ```

5.  Replace the contents of `/etc/ntp/step-tickers` with all your NTP servers, one per line:

    ```
    0.pool.ntp.mydomain.lan
    1.pool.ntp.mydomain.lan
    ```

6.  Now, restart `ntpd` by executing the following:

    ```
    ~]# systemctl restart ntpd

    ```

## There's more…

While `ntpd` is the obvious choice for time synchronization, it doesn't fare well in environments where time masters are intermittently accessible (for whatever reason). In these environments, `chronyd` thrives. Also, `ntpd` can be quite complex to configure correctly, whereas `chronyd` is a little bit simpler.

The reason for modifying `/etc/ntp/step-tickers` when using the `ntpd` file is for the startup of the service. It uses `ntpdate` to synchronize time in one step before actually starting the NTP daemon itself, which is a lot slower in synchronizing time.

To figure out whether your system is synchronized, use the following command:

*   For `chrony`, use the following command:

    ```
    ~]# chronyc sources

    ```

*   For `ntpd`, run the following:

    ```
    ~]# ntpq -p

    ```

Your output will be similar to:

```
remote       refid      st t when poll reach   delay   offset  jitter
=====================================================================
 LOCAL(0)    .LOCL.      5 l  60m   64    0    0.000    0.000   0.000
*master.exam 178.32.44.208 3 u  35 128 377     0.214   -0.651  14.285

```

The asterisk (`*`) in front of an entry means that your system is synchronized to this remote system's clock.

## See also

For more information on configuring `chrony` for RHEL 7, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_the_chrony_Suite.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_the_chrony_Suite.html).

For more information on configuring `ntpd` for RHEL 7, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_ntpd.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Configuring_NTP_Using_ntpd.html).

# Configuring your boot environment

GRUB2 is the default boot loader for RHEL 7\. By default, it doesn't use any fancy configuration options, but it is wise to at least secure your grub boot loader.

## How to do it…

There are many advantages to having your grub and boot environment output to serial console in an enterprise environment. Many vendors integrate virtual serial ports in their remote control systems, as does KVM. This allows you to connect to the serial port and easily grab whatever is displayed in a text editor.

Setting a password on the GRUB2 boot loader mitigates possible hacking attempts on your system when you have physical access to the server or console. Perform the following steps for this recipe:

1.  First, edit `/etc/sysconfig/grub` with your favorite editor.
2.  Now, modify the `GRUB_TERMINAL_OUTPUT` line to include both console and serial access by executing the following command line:

    ```
    GRUB_TERMINAL_OUTPUT="console serial"
    ```

3.  Add the `GRUB_SERIAL_COMMAND` entry, as follows:

    ```
    GRUB_SERIAL_COMMAND="serial --speed=9600 --unit=0 --word=8 --parity=no –stop=1"
    ```

4.  Now, save the file.
5.  Create the `/etc/grub.d/01_users` file with the following contents:

    ```
    cat << EOF
    set superusers="root"
    password root SuperSecretPassword
    EOF

    ```

6.  Next, update your `grub` configuration by running the following commands:

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

## How it works…

The behavior of `grub2-mkconfig` is defined by the directives of the files in `/etc/grub.d`. These files, based on the configuration in `/etc/sysconfig/grub`, autogenerate all the menu entries in the `grub.cfg` file. You can modify its behavior by adding files with bash code in this directory.

For instance, you could add a script that would add a menu entry to boot from the CD/DVD ROM drive.

The user root, which is added to `/etc/grub.d/01_users`, is the only one allowed to edit menu entries from the console, mitigating the weakness in GRUB to force rescue mode by adding `1` or `rescue` at the end of the `kernel` line.

## There's more…

The `grub2-mkconfig` command is specific for BIOS-based systems. In order to do the same on UEFI systems, modify the command as follows:

```
~]# grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg

```

In order to access the GRUB terminal over the same serial connection, you need to specify an additional kernel option: `console=ttyS0,9600n8`.

You can either modify the kernel lines in `/boot/grub2/grub.cfg` (or `/boot/efi/EFI/redhat/grub.cfg` manually, but you do risk losing the change when your kernel is updated), or manually regenerate the file using `grub2-mkconfig`.

It's best to add it to the `GRUB_CMDLINE_LINUX` directive in `/etc/sysconfig/grub` and regenerate your `grub.cfg` file.

Passwords for GRUB users can be encrypted using the `grub2-mkpasswd-pbkdf2` command, as follows:

```
~]# grub2-mkpasswd-pbkdf2
Enter password:
Reenter password:
PBKDF2 hash of your password is grub.pbkdf2.sha512.10000.C208DD5E318B1D6477C4E51035649C197411259C214D0B83E3E83753AD58F7676B62CDF48E31AF0E739844A5CF9A95F76AF5008AF340336DB50ECA23906ECC13.9D20A66F0CADA12AA617B293B5BBF7AAD44423ECA513F302FEBF5CB92A0DC54436E16D7CD6E09685323084A27462C2A981054D52F452F5C2F71FBACD2C31AEFA
~]#

```

Then, you can substitute the clear text password in `/etc/grub.d/01_users` with the generated hash. Here's an example:

```
password root grub.pbkdf2.sha512.10000.C208DD5E318B1D6477C4E51035649C197411259C214D0B83E3E83753AD58F7676B62CDF48E31AF0E739844A5CF9A95F76AF5008AF340336DB50ECA23906ECC13.9D20A66F0CADA12AA617B293B5BBF7AAD44423ECA513F302FEBF5CB92A0DC54436E16D7CD6E09685323084A27462C2A981054D52F452F5C2F71FBACD2C31AEFA
```

All the entries that are automatically generated are bootable but not editable from the console, unless you know the user and password. If you have custom menu entries and want to protect them in a similar way, add `--unrestricted` to the menu entry definition before the accolades. Here's an example:

```
menuentry 'My custom grub boot entry' <options> --unrestricted {
```

## See also

For more information about working with the GRUB2 boot loader, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Working_with_the_GRUB_2_Boot_Loader.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/ch-Working_with_the_GRUB_2_Boot_Loader.html).

# Configuring smtp

Many programs use (or can be configured to use) SMTP to send messages about their status and so on. By default, postfix is configured to deliver all messages locally and not respond to incoming mails. If you have an environment of multiple servers, this can become quite tedious to log on to each server to check for new mail. This recipe will show you how to relay messages to a central mail relay or message store that also uses SMTP.

Postfix is installed by default on RHEL 7.

## How to do it…

In this recipe, we'll combine several options:

*   We'll allow the server to accept incoming mails
*   We'll only allow the server to relay messages from recipients in the `mydomain.lan` domain
*   We'll forward all mails to the `mailhost.mydomain.lan` mailserver

To complete this recipe, perform the following steps:

1.  Edit `/etc/postfix/main.cf` with your favorite editor.
2.  Modify `inet_interface` to accept mails on any interface through the following command:

    ```
    inet_interface = all
    ```

3.  Add the `smtpd_recipient_restrictions` directive to only allow incoming mails from the `mydomain.lan` domain, as follows:

    ```
    smtpd_recipient_restrictions =
        check_sender_access hash:/etc/postfix/sender_access,
        reject
    ```

    As you can see, the last two lines are indented. The `postfix` considers this block as one line instead of three separate lines.

4.  Add the `relayhost` directive to point to `mailhost.mydomain.lan`, as follows:

    ```
    relayhost = mailhost.mydomain.lan
    ```

5.  Now, save the `postfix` file.
6.  Create `/etc/postfix/sender_access` with the following contents:

    ```
    mydomain.lan OK
    ```

7.  Next, hash the `/etc/postfix/access` file using the following command:

    ```
    ~]# postmap /etc/postfix/access

    ```

8.  Finally, restart `postfix`, as follows:

    ```
    ~]# systemctl restart postfix

    ```

## There's more…

To monitor your mail queue on the system, execute the following:

```
~]# postqueue -p

```

Whenever your mail relay cannot forward mails, it stores them locally and tries to resend them at a later time. When you restore the mailflow, you can flush the queue and attempt delivery by executing the following:

```
~]# postqueue -f

```

The kind of setup presented in this recipe is quite simple and assumes that you don't have malicious users on your network. There are software that allow you to mitigate spam and viruses. Popular solutions for this are `spamassassin` and `amavis`.

## See also

For more information on using postfix with RHEL 7, go to [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-email-mta.html#s2-email-mta-postfix](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System_Administrators_Guide/s1-email-mta.html#s2-email-mta-postfix).

For more information on postfix, check out the postfix rpm (`rpm -ql postfix`) or go to [http://www.postfix.org/](http://www.postfix.org/). This site provides good documentation and *how to*'s for a large number of scenarios.