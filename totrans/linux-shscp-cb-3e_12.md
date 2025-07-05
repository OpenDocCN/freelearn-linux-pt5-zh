# Tuning a Linux System

In this chapter, we will cover the following recipes:

*   Identifying services
*   Gathering socket data with `ss`
*   Gathering system I/O usage with `dstat`
*   Identifying a resource hog with `pidstat`
*   Tuning the Linux kernel with `sysctl`
*   Tuning a Linux system with config files
*   Changing scheduler priority using the `nice` command

# Introduction

No system runs as fast as we need it to run, and any computer's performance can be improved.

We can improve the performance of a system by turning off unused services, by tuning the kernel parameters, or by adding new hardware.

The first step in tuning a system is understanding what the demands are and whether they are being met. Different types of applications have different critical needs. The questions to ask yourself include the following:

*   Is the CPU the critical resource for this system? A system doing engineering simulations requires CPU cycles more than other resources.
*   Is network bandwidth critical for this system? A file server does little computation, but can saturate its network capacity.
*   Is disk access speed critical for this system? A file server or database server will put more demand on the disks than a calculation engine does.
*   Is RAM the critical resource for this system? All systems need RAM, but a database server commonly builds large in-memory tables to perform queries, and file servers are more efficient with larger RAM for disk caches.
*   Has your system been hacked? A system can suddenly become unresponsive because it's running unexpected malware. This is not common on Linux machines, but a system with many users (like a college or business network) is vulnerable to a brute-force password attack.

The next question to ask is: How do I measure usage? Knowing how a system is being used will lead you to the questions, but may not lead you to the answer. A fileserver will cache commonly accessed files in the memory, so one with too little memory may be disk/RAM limited rather than network limited.

Linux has tools for analyzing a system. Many have been discussed in [Chapter 8](5ba784d5-fa8b-4840-b4c5-cac906e484f9.xhtml), *The Old-Boy Network*, [Chapter 9](39e9cad3-701a-48c5-9b88-59e8b7c0ce41.xhtml), *Put on The Monitor's Cap*, and [Chapter 11](5bf336a4-5338-4a7a-b27e-1ebecac4704d.xhtml), *Tracing The Clues*. This chapter will introduce more monitoring tools.

Here is a list of subsystems and tools to examine them. Many (but not all) of these tools have been discussed in this book.

*   CPU: `top`, `dstat`, `perf`, `ps`, `mpstat`, `strace`, `ltrace`
*   Network: `netstat`, `ss`, `iotop`, `ip`, `iptraf`, `nicstat`, `ethtool`, `lsof`
*   Disk: `ftrace`, `iostat`, `dstat`, `blktrace`
*   RAM: top, `dstat`, `perf`, `vmstat`, `swapon`

Many of these tools are part of a standard Linux distribution. The others can be loaded with your package manager.

# Identifying services

A Linux system can run hundreds of tasks at a time. Most of these are part of the operating system environment, but you might discover you're running a daemon or two you don't need.

Linux distributions support one of the three utilities that start daemons and services. The traditional `SysV` system uses scripts in `/etc/init.d`. The newer `systemd` daemon uses the same `/etc/init.d` scripts and also uses a `systemctl` call. Some distributions use Upstart, which stores configuration scripts in `/etc/init`.

The SysV `init` system is being phased out in favor of the `systemd` suite. The `upstart` utility was developed and used by Ubuntu, but discarded in favor of `systemd` with the 14.04 release. This chapter will focus on `systemd`, since that's the system used by most distributions.

# Getting ready

The first step is to determine whether your system is using the SysV `init` calls, `systemd`, or `upstart`.

Linux/Unix systems must have an initialization process running as `PID 1`. This process executes a fork and exec to start every other process. The `ps` command may tell you which initialization process is running:

```
    $ ps -p 1 -o cmd
 /lib/system/systemd

```

In the previous example, the system is definitely running `systemd`. However, on some distributions, the SysV `init` program is `sym-linked` to the actual `init` process, and `ps` will always show `/sbin/init`, whether it's SysV `init`, `upstart`, or `systemd` that's actually being used:

```
    $ ps -p 1 -o cmd
 /sbin/init

```

The `ps` and `grep` commands give more clues:

```
    $ ps -eaf | grep upstart

```

Alternatively, they can be used like this:

```
    ps -eaf | grep systemd 

```

If either of these commands return tasks such as `upstart-udev-bridge` or `systemd/systemd`, the system is running `upstart` or `systemd`, respectively. If there are no matches, then your system is probably running the SysV `init` utility.

# How to do it...

The `service` command is supported on most distributions. The `-status-all` option will report the current status of all services defined in `/etc/init.d`. The output format varies between distributions:

```
    $> service -status-all

```

Debian:

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

CentOS:

```
abrt-ccpp hook is installed
abrtd (pid  4009) is running...
abrt-dump-oops is stopped
acpid (pid  3674) is running...
atd (pid  4056) is running...
auditd (pid  3029) is running...
...

```

The `grep` command will reduce the output to only running tasks:

Debian:

```
    $ service -status-all | grep +

```

CentOS:

```
    $ service -status-all | grep running

```

You should disable any unnecessary services. This reduces the load on the system and improves the system security.

Services to check for include the following:

*   `smbd`, nmbd: These are the Samba daemons used to share resources between Linux and Windows systems.
*   `telnet`: This is the old, insecure login program. Unless there is an overwhelming need for this, use SSH.
*   `ftp`: This is the old, insecure File Transfer Protocol. Use SSH and scp instead.
*   `rlogin`: This is remote login. SSH is more secure.
*   `rexec`: This is remote exec. SSH is more secure.
*   `automount`: If you are not using NFS or Samba you probably don't need this.
*   `named`: This daemon provides **Domain Name Service** (**DNS**). It's only necessary if the system is defining the local names and IP addresses. You don't need it to resolve names and access the net.
*   `lpd`: The **Line Printer Daemon** lets other systems use this system's printer. If this is not a print server, you don't need this service.
*   `nfsd`: This is the **Network File System** daemon. It lets remote machines mount this computer's disk partitions. If this is not a file server, you probably don't need this service.
*   `portmap`: This is part of the NFS support. If the system is not using NFS, you don't need this.
*   `mysql`: The **mysql** application is a database server. It may be used by your webserver.
*   `httpd`: This is the HTTP daemon. It sometimes gets installed as part of a **Server System** set of packages.

There are several potential ways to disable an unnecessary service depending on whether your system is Redhat or Debian derived, and whether it's running `systemd`, SysV, or Upstart. All of these commands must be run with root privileges.

# systemd-based computers

The `systemctl` command enables and disables services. The syntax is as follows:

```
    systemctl enable SERVICENAME

```

Alternatively, it can also be as follows:

```
    systemctl disable SERVICENAME

```

To disable an FTP server, use the following command:

```
    # systemctl disable ftp

```

# RedHat-based computers

The `chkconfig` utility provides a frontend for working with SysV style initialization scripts in `/etc/rc#.d`. The `-del` option disables a service, while the `-add` option enables a service. Note that an initialization file must already exist to add a service.

The syntax is as follows:

```
    # chkconfig -del SERVICENAME
 # chkconfig -add SERVICENAME

```

To disable the HTTPD daemon, use the following command:

```
    # chkconfig -del httpd

```

# Debian-based computers

Debian-based systems provide the `update-rc.d` utility to control SysV style initialization scripts. The `update-rc.d` command supports `enable` and `disable` as subcommands:

To disable the telnet daemon, use the following command:

```
    # update-rc.d disable telnetd

```

# There's more

These techniques will find services that have been started at root with the SysV or systemd initialization scripts. However, services may be started manually, or in a boot script, or with `xinetd`.

The `xinetd` daemon functions in a similar way to init: it starts services. Unlike init, the `xinitd` daemon only starts a service when it's requested. For services such as SSH, which are required infrequently and run for a long time once started, this reduces the system load. Services such as `httpd` that perform small actions (serve a web page) frequently are more efficient to start once and keep running.

The configuration file for **xinet** is `/etc/xinetd.conf`. The individual service files are commonly stored in `/etc/xinetd.d`.

The individual service files resemble this:

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

A service can be enabled or disabled by changing the value of the `disable` field. If `disable` is `no`, the service is enabled. If disable is `yes`, the service is disabled.

After editing a service file, you must restart `xinetd`:

```
 # cd /etc/init.d
 # ./inetd restart

```

# Gathering socket data with ss

The daemons started by `init` and `xinetd` may not be the only services running on a system. Daemons can be started by commands in an `init` local file `(/etc/rc.d/rc.local`), a `crontab` entry, or even by a user with privileges.

The `ss` command returns socket statistics, including services using sockets, and current socket status.

# Getting ready

The `ss` utility is included in the `iproute2` package that is already installed on most modern distributions.

# How to do it...

The `ss` command displays more information than the `netstat` command. These recipes will introduce a few of its features.

# Displaying the status of tcp sockets

A `tcp` socket connection is opened for every HTTP access, every SSH session, and so on. The `-t` option reports the status of TCP connections:

```
 $ ss -t
 ESTAB      0      0   192.168.1.44:740          192.168.1.2:nfs 
 ESTAB      0      0   192.168.1.44:35484        192.168.1.4:ssh

 CLOSE-WAIT 0      0   192.168.1.44:47135         23.217.139.9:http 

```

This example shows an NFS server connected at IP address `192.168.1.2` and an SSH connection to `192.168.1.4`.

The `CLOSE-WAIT` socket status means that the `FIN` signal has been sent, but the socket has not been fully closed. A socket can remain in this state forever (or until you reboot). Terminating the process that owns the socket may free the socket, but that's not guaranteed.

# Tracing applications listening on ports

A service on your system will open a socket in the `listen` mode to accept network connections from a remote site. The SSHD application does this to listen for SSH connections, http servers do this to accept HTTP requests, and so on.

If your system has been hacked, it might have a new application listening for instructions from its master.

The `-l` option to `ss` will list sockets that are open in the `listen` mode. The `-u` option specifies to report UDP sockets. A `-t` option reports TCP sockets.

This command shows a subset of the listening UDP sockets on a Linux workstation:

```
$ ss -ul
State          Recv-Q  Send-Q      Local Address:Port         Peer Address:Port 
UNCONN         0       0           *:sunrpc                   *:* 
UNCONN         0       0           *:ipp                      *:* 
UNCONN         0       0           *:ntp                      *:* 
UNCONN         0       0           127.0.0.1:766              *:* 
UNCONN         0       0           *:898                      *:* 

```

This output shows that this system will accept **Remote Procedure Calls** (**sunrpc**). This port is used by the `portmap` program. The `portmap` program controls access to the RPC services and is used by the `nfs` client and server.

The `ipp` and `ntp` ports are used for **Internet Printing Protocol** and **Network Time Protocol**. Both are useful tools, but may not be required on a given system.

Ports `766` and `898` are not listed in `/etc/services`. The `-I` option of the `lsof` command will display the task that has a port open. You may need to have root access to view this:

```
    # lsof -I :898

```

Or:

```
 # lsof -n -I :898
 COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
 rpcbind 3267  rpc    7u  IPv4  16584      0t0  UDP *:898 
 rpcbind 3267  rpc   10u  IPv6  16589      0t0  UDP *:898 

```

This command shows that the tasks listening on port `898` are part of the RPC system, not a hacker.

# How it works

The `ss` command uses system calls to extract information from the internal kernel tables. The known services and ports on your system are defined in `/etc/services`.

# Gathering system I/O usage with dstat

Knowing what services are running may not tell you which services are slowing down your system. The top command (discussed in [Chapter 9](39e9cad3-701a-48c5-9b88-59e8b7c0ce41.xhtml), *Put on the Monitor's Cap*) will tell you about CPU usage and how much time is spent waiting for IO, but it might not tell you enough to track down a task that's overloading the system.

Tracking I/O and context switches can help trace a problem to its source.

The `dstat` utility can point you to a potential bottleneck.

# Getting ready

The **dstat** application is not commonly installed. It will need to be installed with your package manager. It requires Python 2.2, which is installed by default on modern Linux systems:

```
    # apt-get install dstat
 # yum install dstat

```

# How to do it...

The dstat application displays disk, network, memory usage, and running task information at regular intervals. The default output gives you an overview of the system activity. By default, this report is updated every second on a new line, allowing easy comparison with previous values.

The default output lets you track overall system activity. The application supports more options to track top resource users.

# Viewing system activity

Invoking dstat with no arguments will show CPU activity, disk I/O, network I/O, paging, interrupts, and context switches at one second intervals.

The following example shows the default `dstat` output:

```
$ dstat
----total-cpu-usage---- -dsk/total- -net/total- ---paging-- ---system--
usr sys idl wai hiq siq| read  writ| recv  send|  in   out | int   csw 
 1   2  97   0   0   0|5457B   55k|   0     0 |   0     0 |1702  3177 
 1   2  97   0   0   0|   0     0 |  15k 2580B|   0     0 |2166  4830 
 1   2  96   0   0   0|   0    36k|1970B 1015B|   0     0 |2122  4794 

```

You can ignore the first line. Those values are the initial contents of the tables dstat mines. The subsequent lines show the activity during a time slice.

In this sample, the CPU is mostly idle, and there is little disk activity. The system is generating network traffic, but only a few packets a second.

There is no paging on this system. Linux only pages out memory to disk when the main memory is exhausted. Paging lets a system run more applications than it could run without paging, but disk access is thousands of times slower than memory access, so a computer will slow to a crawl if it needs to page.

If your system sees consistent paging activity, it needs more RAM or fewer applications.

A database application can cause intermittent paging when queries that require building large in-memory arrays are evaluated. It may be possible to rewrite these queries using the IN operation instead of a JOIN to reduce the memory requirement. (This is a more advanced SQL than what is covered in this book.)

**Context switches** (**csw**) happen with every system call (refer to the strace and ltrace discussion in [Chapter 11](5bf336a4-5338-4a7a-b27e-1ebecac4704d.xhtml), *Tracing the Clues*) and when a timeslice expires and another application is given access to the CPU. A system call happens whenever I/O is performed or a program resizes itself.

If the system is performing tens of thousands of context switches per second, it's a symptom of a potential problem.

# How it works

The `dstat` utility is a Python script that collects and analyzes data from the `/proc` filesystem described in [Chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Calls*.

# There's more...

The `dstat` utility can identify the top resource user in a category:

*   **-top-bio Disk Usage**: This reports the process performing the most block I/O
*   -**top-cpu CPU Usage**: This reports the process using the most CPU resources
*   **-top-io I/O usage**: This reports the process performing the most I/O (usually network I/O)
*   **-top-latency System Load**: This shows the process with the highest latency
*   **-top-mem Memory Usage**: This shows the process using the most memory

The following example displays the CPU and Network usage and the top users in each category:

```
$ dstat -c -top-cpu -n -top-io
----total-cpu-usage---- -most-expensive- -net/total- ----most-expensive----
usr sys idl wai hiq siq|  cpu process   | recv  send|     i/o process 
 1   2  97   0   0   0|vmware-vmx   1.0|   0     0 |bash         26k    2B
 2   1  97   0   0   0|vmware-vmx   1.7|  18k 3346B|xterm       235B 1064B
 2   2  97   0   0   0|vmware-vmx   1.9| 700B 1015B|firefox      82B   32k

```

On a system running an active virtual machine, the VM uses the most CPU time, but not the bulk of the IO. The CPU is spending most of its time in the idle state.

The `-c` and `-n` options specify showing the CPU usage and Network usage, respectively.

# Identifying a resource hog with pidstat

The `-top-io` and `-top-cpu` flags will identify a top resource user, but might not provide enough information to identify the problem if there are multiple instances of a resource hog.

The `pidstat` program will report per-process statistics, which can be sorted to provide more insight.

# Getting ready

The `pidstat` application may not be installed by default. It can be installed with this command:

```
    # apt-get install sysstat

```

# How to do it...

The pidstat application has several options for generating different reports:

*   `-d`: This reports IO statistics
*   `-r`: This reports page faults and memory utilization
*   `-u`: This reports CPU utilization
*   `-w`: This reports task switches

Report Context Switch activity:

```
 $ pidstat -w | head -5
 Linux 2.6.32-642.11.1.el6.x86_64 (rtdaserver.cflynt.com)    
    02/15/2017  _x86_64_ (12 CPU)

 11:18:35 AM       PID   cswch/s nvcswch/s  Command
 11:18:35 AM         1      0.00      0.00  init
 11:18:35 AM         2      0.00      0.00  kthreadd

```

The pidstat application sorts its report by the PID number. The data can be re-organized with the sort utility. The following command displays the five applications that generate the most context switches per second (*Field 4* in the `-w` output):

```
 $ pidstat -w | sort -nr -k 4 | head -5
 11:13:55 AM     13054    351.49      9.12  vmware-vmx
 11:13:55 AM      5763     37.57      1.10  vmware-vmx
 11:13:55 AM      3157     27.79      0.00  kondemand/0
 11:13:55 AM      3167     21.18      0.00  kondemand/10
 11:13:55 AM      3158     21.17      0.00  kondemand/1

```

# How it works

The pidstat application queries the kernel to get task information. The sort and head utilities reduce the data to pinpoint the program hogging a resource.

# Tuning the Linux kernel with sysctl

The Linux kernel has about 1,000 tunable parameters. These default to reasonable values for common usage, which means they are not perfect for anyone.

# Getting started

The `sysctl` command is available on all Linux systems. You must be root to modify kernel parameters.

The `sysctl` command will change the parameter value immediately, but the value will revert to the original value upon reboot unless you add a line to define the parameter to `/etc/sysctl.conf`.

It's a good policy to change a value manually and test it before modifying `sysctl.conf`. You can make a system unbootable by applying bad values to `/etc/sysctl.conf`.

# How to do it...

The `sysctl` command supports several options:

*   `-a`: This reports all available parameters
*   `-p FILENAME`: This reads values from `FILENAME`. By default from `/etc/sysctl.conf`
*   `PARAM`: This reports the current value of `PARAM`
*   `PARAM=NEWVAL`: This sets the value of `PARAM`

# Tuning the task scheduler

The task scheduler is optimized for a desktop environment, where a fast response to a user is more important than overall efficiency. Increasing the time a task stays resident improves the performance of server systems. The following example examines the value of `kernel.sched_migration_cost_ns`:

```
 $ sysctl.kernel.shed_migration_cost_ns
 kernel.sched_migration_cost_ns = 500000

```

The `kernel_sched_migration_cost_ns` (and `kernel.sched_migration_cost` in older kernels) controls how long a task will remain active before being exchanged for another task. On systems with many tasks or many threads, this can result in too much overhead being used for context switching. The default value of `500000` ns is too small for systems running Postgres or Apache servers. It's recommended that you change the value to 5 ms:

```
    # sysctl kernel.sched_migration_cost_ns=5000000

```

On some systems (postgres servers in particular), unsetting the `sched_autogroup_enabled` parameter improves performance.

# Tuning a network

The default values for network buffers may be too small on a system performing many network operations (NFS client, NFS server, and so on).

Examine the values for maximum read buffer memory:

```
 $ sysctl net.core.rmem_max
 net.core.rmem_max = 124928

```

Increase values for network servers:

```
 # sysctl net.core.rmem_max=16777216
 # sysctl net.core.wmem_max=16777216
 # sysctl net.ipv4.tcp_rmem="4096 87380 16777216"
 # sysctl net.ipv4.tcp_wmem="4096 65536 16777216"
 # sysctl net.ipv4.tcp_max_syn_backlog=4096

```

# How it works

The `sysctl` command lets you directly access kernel parameters. By default, most distributions optimize these parameters for a normal workstation.

If your system has lots of memory, you can improve performance by increasing the amount of memory devoted to buffers. If it's short on memory, you may want to shrink these. If the system is a server, you may want to keep tasks resident longer than you would for a single-user workstation.

# There's more...

The `/proc` filesystem is available on all Linux distributions. It includes a folder for every running task and folders for all the major kernel subsystems. The files within these folders can be viewed and updated with `cat`.

The parameters supported by sysctl are commonly supported by the `/proc` filesystem as well.

Thus, `net.core.rmem_max` can also be accessed as `/proc/sys/net/core/rmem_max`.

# Tuning a Linux system with config files

The Linux system includes several files to define how disks are mounted, and so on. Some parameters can be set in these files instead of using `/proc` or `sysctl`.

# Getting ready

There are several files in `/etc` that control how a system is configured. These can be edited with a standard text editor such as `vi` or `emacs`. The changes may not take effect until the system is rebooted.

# How to do it...

The `/etc/fstab` file defines how disks are to be mounted and what options are supported.

The Linux system records when a file is created, modified, and read. There is little value in knowing that a file has been read, and updating the `Acessed` timestamp every time a common utility like cat is accessed gets expensive.

The `noatime` and `relatime` mount options will reduce disk thrashing:

```
 $ cat /dev/fstab
 /dev/mapper/vg_example_root  /    ext4 defaults,noatime  1 1
 /dev/mapper/gb_example_spool /var ext4 defaults,relatime 1 1

```

# How it works

The preceding example mounts the / partition (which includes `/bin` and `/usr/bin`) with the usual default options, plus the `noatime` parameter to disable updating the disk every time a file is accessed. The `/var` partition (which includes the mail spool folder) has the realtime option set, which will update time at least once every day, but not every time a file is accessed.

# Changing scheduler priority using the nice command

Every task on a Linux system has a priority. The priority values range from -20 to 19\. The lower the priority (**-20**), the more CPU time a task will be allocated. The default priority is **0**.

Not all tasks need the same priority. An interactive application needs to respond quickly or it becomes difficult to use. A background task run via `crontab` only needs to finish before it is scheduled to run again.

The `nice` command will modify a task's priority. It can be used to invoke a task with modified priority. Raising a task's priority value will free resources for other tasks.

# How to do it...

Invoking the `nice` command with no arguments will report a task's current priority:

```
    $ cat nicetest.sh
 echo "my nice is `nice`"
 $ sh nicetest.sh
 my nice is 0

```

Invoking the `nice` command followed by another command name will run the second command with a *niceness* of `10`–it will add 10 to the task's default priority:

```
    $ nice sh nicetest.sh
 my nice is 10

```

Invoking the `nice` command with a value before the command will run the command with a defined *niceness*:

```
    $ nice -15 sh nicetest.sh
 my nice is 15

```

Only a superuser can give a task a higher priority (lower priority number), by assigning a negative niceness value:

```
    # nice -adjustment=-15 nicetest.sh
 my nice is -15

```

# How it works

The `nice` command modifies the kernel's scheduling table to run a task with a greater or lesser priority. The lower the priority value, the more time the scheduler will give to this task.

# There's more

The `renice` command modifies the priority of a running task. Tasks that use a lot of resources, but are not time-critical, can be made *nicer* with this command. The `top` command is useful to find tasks that are utilizing the CPU the most.

The `renice` command is invoked with a new priority value and the program ID (PID):

```
    $ renice 10 12345
 12345: old priority 0, new priority 10

```