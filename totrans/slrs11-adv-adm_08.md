# Chapter 8. Administering and Monitoring Processes

In this chapter, we will cover the following topics:

*   Monitoring and handling process execution
*   Managing processes' priority on Solaris 11
*   Configuring FSS and applying it to projects

# Introduction

When working with Oracle Solaris 11, many of the executing processes compose applications, and even the operating system itself runs many other processes and threads, which takes care of the smooth working of the environment. So, administrators have a daily task of monitoring the entire system and taking some hard decisions, when necessary. Furthermore, not all processes have the same priority and urgency, and there are some situations where it is suitable to give higher priority to one process than another (for example, rendering images). Here, we introduce a key concept: scheduling classes.

Oracle Solaris 11 has a default process scheduler (`svc:/system/scheduler:default`) that controls the allocation of the CPU for each process according to its scheduling class. There are six important scheduling classes, as follows:

*   **Timesharing** (**TS**): By default, all processes or threads (non-GUI) are assigned to this class, where the priority value is dynamic and adjustable according to the system load (-60 to 60). Additionally, the system scheduler switches a process/thread with a lower priority from a processor to another process/thread with higher priority.
*   **Interactive** (**IA**): This class has the same behavior as the TS class (dynamic and with an adjustable priority value from -60 to 60), but the IA class is suitable for GUI processes/threads that have an associated window. Additionally, when the mouse focuses on a window, the bound process or thread receives an increase of 10 points of its priority. When the mouse focus is taken off the window, the bound process loses the same 10 points.
*   **Fixed** (**FX**): This class has the same behavior as that of TS, except that any process or thread that is associated with this class has its priority value fixed. The value range is from 0 to 59, but the initial priority of the process or thread is kept from the beginning to end of the life process.
*   **System** (**SYS**): This class is used for kernel processes or threads where the possible priority goes from 60 to 99\. However, once the kernel process or thread begins processing, it's bound to the CPU until the end of its life (the system scheduler doesn't take it off the processor).
*   **Realtime** (**RT**): Processes and threads from this class have a fixed priority that ranges from 100 to 159\. Any process or thread of this class has a higher priority than any other class.
*   **Fair share scheduler** (**FSS**): Any process or thread managed by this class is scheduled based on its share value (and not on its priority value) and in the processor's utilization. The priority range goes from -60 to 60.

Usually, the FSS class is used when the administrator wants to control the resource distribution on the system using processor sets or when deploying Oracle zones. It is possible to change the priority and class of any process or thread (except the system class), but it is uncommon, such as using FSS. When handling a processor set (a group of processors), the processes bound to this group must belong to only one scheduling class (FSS or FX, but not both). It is recommended that you don't use the RT class unless it is necessary because RT processes are bound to the processor (or core) up to their conclusion, and it only allows any other process to execute when it is idle.

The FSS class is based on shares, and personally, I establish a total of 100 shares and assign these shares to processes, threads, or even Oracle zones. This is a simple method to think about resources, such as CPUs, using percentages (for example, 10 shares = 10 percent).

# Monitoring and handling process execution

Oracle Solaris 11 offers several methods to monitor and control process execution, and there isn't one best tool to do this because every technique has some advantages.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running Oracle Solaris 11 installed with a 2 GB RAM at least. It's recommended that the system has more than one processor or core.

## How to do it…

A common way to monitor processes on Oracle Solaris 11 is using the old and good `ps` command:

```
root@solaris11-1:~# ps -efcl -o s,uid,pid,zone,class,pri,vsz,rss,time,comm | more

```

![How to do it…](img/00027.jpeg)

According to the output shown in the previous screenshot, we have:

*   **S** (status)
*   **UID** (user ID)
*   **PID** (process ID)
*   **ZsONE** (zone)
*   **CLS** (scheduling class)
*   **PRI** (priority)
*   **VSZ** (virtual memory size)
*   **RSS** (resident set size)
*   **TIME** (the time that the process runs on the CPU)
*   **COMMAND** (the command used to start the process)

Additionally, possible process statuses are as follows:

*   O (running on a processor)
*   S (sleeping—waiting for an event to complete)
*   R (runnable—process is on a queue)
*   T (process is stopped either because of a job control signal or because it is being traced)
*   Z (zombie—process finished and parent is not waiting)
*   W (waiting—process is waiting for the CPU usage to drop to the CPU-caps enforced limit)

### Note

Do not get confused between the **virtual memory size** (**VSZ**) and **resident set size** (**RSS**). The VSZ of a process includes all information on a physical memory (RAM) plus all mapped files and devices (swap). On the other hand, the RSS value only includes the information in the memory (RAM).

Other important command to monitor processes on Oracle Solaris 11 is the `prstat` tool. For example, it is possible to list the threads of each process by executing the following command:

```
root@solaris11-1:~# prstat –L
PID USERNAME  SIZE   RSS STATE   PRI NICE      TIME  CPU PROCESS/LWPID   
  2609 root      129M   18M sleep    15    0   0:00:24 1.1% gnome-terminal/1
  1238 root       88M   74M sleep    59    0   0:00:41 0.5% Xorg/1
  2549 root      217M   99M sleep     1    0   0:00:45 0.3% java/22
  2549 root      217M   99M sleep     1    0   0:00:30 0.2% java/21
  2581 root       13M 2160K sleep    59    0   0:00:24 0.2% VBoxClient/3
  1840 root       37M 7660K sleep     1    0   0:00:26 0.2% pkg.depotd/2
(truncated output)

```

The `LWPID` column shows the number of threads of each process.

Other good options are `–J` (summary per project), `-Z` (summary per zone), and `–mL` (includes information about thread microstates). To collect some information about processes and projects, execute the following command:

```
root@solaris11-1:~# prstat –J 
   PID USERNAME  SIZE   RSS STATE   PRI NICE      TIME  CPU PROCESS/NLWP      
  2549 root      217M   99M sleep    55    0   0:01:56 0.8% java/25
  1238 root       88M   74M sleep    59    0   0:00:44 0.4% Xorg/3
  1840 root       37M 7660K sleep     1    0   0:00:55 0.4% pkg.depotd/64
(truncated output)
PROJID    NPROC  SWAP   RSS MEMORY      TIME  CPU PROJECT                     
     1       43 2264M  530M    13%   0:03:46 1.9% user.root                   
     0       79  844M  254M   6.1%   0:03:12 0.9% system                      
     3        2   11M 5544K   0.1%   0:00:55 0.0% default                     

Total: 124 processes, 839 lwps, load averages: 0.23, 0.22, 0.22
```

Pay attention to the last column (`PROJECT`) from the second part of the output. It is very interesting to know that Oracle Solaris already works using projects and some of them are created by default. By the way, it is always appropriate to remember that the structure of a project is project | tasks | processes.

Collecting information about processes and zones is done by executing the following command:

```
root@solaris11-1:~# prstat -Z 
   PID USERNAME  SIZE   RSS STATE   PRI NICE      TIME  CPU PROCESS/NLWP      
  3735 root       13M   12M sleep    59    0   0:00:13 4.2% svc.configd/17
  3733 root       17M 8676K sleep    59    0   0:00:05 2.0% svc.startd/15
  2532 root      219M   83M sleep    47    0   0:00:15 0.8% java/25
  1214 root       88M   74M sleep     1    0   0:00:09 0.6% Xorg/3
   746 root        0K    0K sleep    99  -20   0:00:02 0.5% zpool-myzones/138
  (truncated output)

ZONEID    NPROC  SWAP   RSS MEMORY      TIME  CPU ZONE
     1       11   92M   36M   0.9%   0:00:18 6.7% zone1
     0      129 3222M  830M    20%   0:02:09 4.8% global
     2        5   18M 6668K   0.2%   0:00:00 0.2% zone2
```

According to the output, there is a `global` zone and two other nonglobal zones (`zone1` and `zone2`) in this system.

Finally, to gather information about processes and their respective microstate information, execute the following command:

```
root@solaris11-1:~# prstat –mL 
   PID USERNAME USR SYS TRP TFL DFL LCK SLP LAT VCX ICX SCL SIG PROCESS/LWPID 
  1925 pkg5srv  0.8 5.9 0.0 0.0 0.0 0.0  91 2.1 286   2  2K   0 htcacheclean/1
  1214 root     1.6 3.4 0.0 0.0 0.0 0.0  92 2.7 279  24  3K   0 Xorg/1
  2592 root     2.2 2.1 0.0 0.0 0.0 0.0  94 1.7 202   9  1K   0 gnome-termin/1
  2532 root     0.9 1.4 0.0 0.0 0.0  97 0.0 1.2 202   4 304   0 java/22
  5809 root     0.1 1.2 0.0 0.0 0.0 0.0  99 0.0  55   1  1K   0 prstat/1
  2532 root     0.6 0.5 0.0 0.0 0.0  98 0.0 1.3 102   6 203   0 java/21
(truncated output)

```

The output from `prtstat –mL` (gathering microstates information) is very interesting because it can give us some clues about performance problems. For example, the `LAT` column (latency) indicates the percentage of time wait for the CPU (possible problems with the CPU) and in this case, a constant value above zero could mean a CPU performance problem.

Continuing the explanation, a possible problem with the memory can be highlighted using the `TFL` (the percentage of time the process has spent processing text page faults) and `DFL` columns (the percentage of time the process has spent processing data page faults), which shows whether and how many times (in percentage) a thread is waiting for memory paging.

In a complementary manner, when handling processes, there are several useful commands, as shown in the following table:

| Objective | Command |
| --- | --- |
| To show the stack process |  
```
pstack <pid>
```

 |
| To kill a process |  
```
pkill <process name>
```

 |
| To get the process ID of a process |  
```
pgrep –l <pid>
```

 |
| To list the opened files by a process |  
```
pfiles <pid>
```

 |
| To get a memory map of a process |  
```
pmap –x <pid>
```

 |
| To list the shared libraries of a process |  
```
pldd <pid>
```

 |
| To show all the arguments of a process |  
```
pargs –ea <pid>
```

 |
| To trace a process |  
```
truss –p <pid>
```

 |
| To reap a zombie process |  
```
preap <pid>
```

 |

For example, to find out which shared libraries are used by the `top` command, execute the following sequence of commands:

```
root@solaris11-1:~# top
root@solaris11-1:~# ps -efcl | grep top
 0 S     root  2672  2649   IA  59        ?   1112        ? 05:32:53 pts/3       0:00 top
 0 S     root  2674  2606   IA  54        ?   2149        ? 05:33:01 pts/2       0:00 grep top
root@solaris11-1:~# pldd 2672
2672:  top
/lib/amd64/libc.so.1
/usr/lib/amd64/libkvm.so.1
/lib/amd64/libelf.so.1
/lib/amd64/libkstat.so.1
/lib/amd64/libm.so.2
/lib/amd64/libcurses.so.1
/lib/amd64/libthread.so.1
```

To find the top-most stack, execute the following command:

```
root@solaris11-1:~# pstack 2672
2672:  top
 ffff80ffbf54a66a pollsys  (ffff80ffbfffd070, 1, ffff80ffbfffd1f0, 0)
 ffff80ffbf4f1995 pselect () + 181
 ffff80ffbf4f1e14 select () + 68
 000000000041a7d1 do_command () + ed
 000000000041b5b3 main () + ab7
 000000000040930c ???????? ()
```

To verify which files are opened by an application as the Firefox browser, we have to execute the following commands:

```
root@solaris11-1:~# firefox &
root@solaris11-1:~# ps -efcl | grep firefox
 0 S     root  2600  2599   IA  59        ?  61589        ? 13:50:14 pts/1       0:07 firefox
 0 S     root  2616  2601   IA  58        ?   2149        ? 13:51:18 pts/2       0:00 grep firefox
root@solaris11-1:~# pfiles 2600
2600:  firefox
  Current rlimit: 1024 file descriptors
   0: S_IFCHR mode:0620 dev:563,0 ino:45703982 uid:0 gid:7 rdev:195,1
      O_RDWR
      /dev/pts/1
      offset:997
   1: S_IFCHR mode:0620 dev:563,0 ino:45703982 uid:0 gid:7 rdev:195,1
      O_RDWR
      /dev/pts/1
      offset:997
   2: S_IFCHR mode:0620 dev:563,0 ino:45703982 uid:0 gid:7 rdev:195,1
      O_RDWR
      /dev/pts/1
      offset:997
(truncated output)

```

Another excellent command from the previous table is `pmap`, which shows information about the address space of a process. For example, to see the address space of the current shell, execute the following command:

```
root@solaris11-1:~# pmap -x $$
2675:  bash
 Address  Kbytes     RSS    Anon  Locked Mode   Mapped File
08050000    1208    1184       -       - r-x--  bash
0818E000      24      24       8       - rw---  bash
08194000     188     188      32       - rw---    [ heap ]
EF470000      56      52       -       - r-x--  methods_unicode.so.3
EF48D000       8       8       -       - rwx--  methods_unicode.so.3
EF490000    6744     248       -       - r-x--  en_US.UTF-8.so.3
EFB36000       4       4       -       - rw---  en_US.UTF-8.so.3
FE550000     184     148       -       - r-x--  libcurses.so.1
FE58E000      16      16       -       - rw---  libcurses.so.1
FE592000       8       8       -       - rw---  libcurses.so.1
FE5A0000       4       4       4       - rw---    [ anon ]
FE5B0000      24      24       -       - r-x--  libgen.so.1
FE5C6000       4       4       -       - rw---  libgen.so.1
FE5D0000      64      16       -       - rwx--    [ anon ]
FE5EC000       4       4       -       - rwxs-    [ anon ]
FE5F0000       4       4       4       - rw---    [ anon ]
FE600000      24      12       4       - rwx--    [ anon ]
FE610000    1352    1072       -       - r-x--  libc_hwcap1.so.1
FE772000      44      44      16       - rwx--  libc_hwcap1.so.1
FE77D000       4       4       -       - rwx--  libc_hwcap1.so.1
FE780000       4       4       4       - rw---    [ anon ]
FE790000       4       4       4       - rw---    [ anon ]
FE7A0000       4       4       -       - rw---    [ anon ]
FE7A8000       4       4       -       - r--s-    [ anon ]
FE7B4000     220     220       -       - r-x--  ld.so.1
FE7FB000       8       8       4       - rwx--  ld.so.1
FE7FD000       4       4       -       - rwx--  ld.so.1
FEFFB000      16      16       4       - rw---    [ stack ]
-------- ------- ------- ------- -------
total Kb   10232    3332      84       -
```

The `pmap` output shows us the following essential information:

*   `Address`: This is the starting virtual address of each mapping
*   `Kbytes`: This is the virtual size of each mapping
*   `RSS`: The amount of RAM (in KB) for each mapping, including shared memory
*   `Anon`: The number of pages of anonymous memory, which is usually and roughly defined as the sum of heap and stack pages without a counterpart on the disk (excluding the memory shared with other address spaces)
*   `Lock`: The number of pages locked in the mapping
*   Permissions: Virtual memory permissions for each mapping. The possible and valid permissions are as follows:

    *   `x` Any instructions inside this mapping can be executed by the process
    *   `w` The mapping can be written by the process
    *   `r` The mapping can be read by the process
    *   `s` The mapping is shared with other processes
    *   `R` There is no swap space reserved for this process

*   Mapped File: The name for each mapping such as an executable, a library, and anonymous pages (heap and stack)

Finally, there is an excellent framework, DTrace, where you can get information on processes and anything else related to Oracle Solaris 11.

What is DTrace? It is a clever instrumentation tool that is used for troubleshooting and, mainly, as a suitable framework for performance and analysis. DTrace is composed of thousands of probes (sensors) that are scattered through the Oracle Solaris kernel. To explain this briefly, when a program runs, any touched probe from memory, CPU, or I/O is triggered and gathers information from the related activity, giving us an insight on where the system is spending more time and making it possible to create reports.

DTrace is nonintrusive (it does not add a performance burden on the system) and safe (by default only the root user has enough privileges to use DTrace) and uses the Dscript language (similar to AWK). Different from other tools such as `truss`, `apptrace`, `sar`, `prex`, `tnf`, `lockstat`, and `mdb`, which allow knowing only the problematic area, DTrace provides the exact point of the problem.

The fundamental structure of a DTrace probe is as follows:

```
provider:module:function:name
```

The previous probe is explained as follows:

*   `provider`: These are libraries that instrument regions of the system, such as `syscall` (system calls), `proc` (processes), `fbt` (function boundary tracing), `lockstat`, and so on
*   `module`: This represents the shared library or kernel module where the probe was created
*   `function`: This is a program, process, or thread function that contains the probe
*   `name`: This is the probe's name

When using DTrace, for each probe, it is possible to associate an action that will be executed if this probe is touched (triggered). By default, all probes are disabled and don't consume CPU processing.

DTrace probes are listed by executing the following command:

```
root@solaris11-1:~# dtrace -l | more

```

The output of the previous command is shown in the following screenshot:

![How to do it…](img/00028.jpeg)

The number of available probes on Oracle Solaris 11 are reported by the following command:

```
root@solaris11-1:~# dtrace -l | wc –l
   75899
```

DTrace is a very interesting and massive subject. Certainly, we could dedicate entire chapters or even a whole book to explain DTrace's world.

After this brief introduction to DTrace, we can use it for listing any new processes (including their respective arguments) by running the following command:

```
root@solaris11-1:~# dtrace -n 'proc:::exec-success { trace(curpsinfo->pr_psargs); }'
dtrace: description 'proc:::exec-success ' matched 1 probe
 CPU     ID                    FUNCTION:NAME
   3   7639         exec_common:exec-success   bash                             
   2   7639         exec_common:exec-success   /usr/bin/firefox                 
   0   7639         exec_common:exec-success   sh -c ps -e -o 'pid tty time comm'> /var/tmp/aaacLaiDl
   0   7639         exec_common:exec-success   ps -e -o pid tty time comm       
   0   7639         exec_common:exec-success   ps -e -o pid tty time comm       
   1   7639         exec_common:exec-success   sh -c ps -e -o 'pid tty time comm'> /var/tmp/caaeLaiDl
   2   7639         exec_common:exec-success   sh -c ps -e -o 'pid tty time comm'> /var/tmp/baadLaiDl
   2   7639         exec_common:exec-success   ps -e -o pid tty (truncated output)

```

There are very useful one-line tracers, as shown previously, available from Brendan Gregg's website at [http://www.brendangregg.com/DTrace/dtrace_oneliners.txt](http://www.brendangregg.com/DTrace/dtrace_oneliners.txt).

It is feasible to get any kind of information using DTrace. For example, get the system call count per program by executing the following command:

```
root@solaris11-1:~# dtrace -n 'syscall:::entry { @num[pid,execname] = count(); }'
dtrace: description 'syscall:::entry ' matched 213 probes
^C
       11  svc.startd                                           2
       13  svc.configd                                          2
       42  netcfgd                                              2
(truncated output)
     2610  gnome-terminal                                     1624
     2549  java                                               2464
     1221  Xorg                                               5246
     2613  dtrace                                             5528
     2054  htcacheclean                                       9503
```

To get the total number of read bytes per process, execute the following command:

```
root@solaris11-1:~# dtrace -n 'sysinfo:::readch { @bytes[execname] = sum(arg0); }'
dtrace: description 'sysinfo:::readch ' matched 4 probes
^C
  in.mpathd                                                      1
  named                                                         56
  sed                                                          100
  wnck-applet                                                  157
 (truncated output)
  VBoxService                                                20460
  svc.startd                                                 40320
  Xorg                                                       65294
  ps                                                       1096780
  thunderbird-bin                                          3191863
```

To get the number of write bytes by process, run the following command:

```
root@solaris11-1:~# dtrace -n 'sysinfo:::writech { @bytes[execname] = sum(arg0); }'
dtrace: description 'sysinfo:::writech ' matched 4 probes
^C
  dtrace                                                         1
  gnome-power-mana                                               8
  xscreensaver                                                  36
  gnome-session                                                367
  clock-applet                                                 404
  named                                                        528
  gvfsd                                                        748
  (truncated output)
  metacity                                                   24616
  ps                                                         59590
  wnck-applet                                                65523
  gconfd-2                                                   83234
  Xorg                                                      184712
  firefox                                                   403682
```

To know the number of pages paged-in by process, execute the following command:

```
root@solaris11-1:~# dtrace -n 'vminfo:::pgpgin { @pg[execname] = sum(arg0); }'
dtrace: description 'vminfo:::pgpgin ' matched 1 probe
^C
(no output)
```

To list the disk size by process, run the following command:

```
root@solaris11-1:~# dtrace -n 'io:::start { printf("%d %s %d",pid,execname,args[0]->b_bcount); }'
dtrace: description 'io:::start ' matched 3 probes
 CPU     ID                    FUNCTION:NAME
   1   6962              bdev_strategy:start 5 zpool-rpool 4096
   1   6962              bdev_strategy:start 5 zpool-rpool 4096
   2   6962              bdev_strategy:start 5 zpool-rpool 4096
   2   6962              bdev_strategy:start 2663 firefox 3584
   2   6962              bdev_strategy:start 2663 firefox 3584
   2   6962              bdev_strategy:start 2663 firefox 3072
   2   6962              bdev_strategy:start 2663 firefox 4096
^C
(truncated output)

```

From Brendan Gregg's website ([http://www.brendangregg.com/dtrace.html](http://www.brendangregg.com/dtrace.html)), there are other good and excellent scripts. For example, `prustat.d` (which we can save in our home directory) is one of them and its output is self-explanatory; it can be obtained using the following commands:

```
root@solaris11-1:~# chmod u+x prustat.d 
root@solaris11-1:~# ./prustat.d 
  PID   %CPU   %Mem  %Disk   %Net  COMM
 2537   0.91   2.38   0.00   0.00  java
 1218   0.70   1.81   0.00   0.00  Xorg
 2610   0.51   0.47   0.00   0.00  gnome-terminal
 2522   0.00   0.96   0.00   0.00  nautilus
 2523   0.01   0.78   0.00   0.00  updatemanagerno
 2519   0.00   0.72   0.00   0.00  gnome-panel
 1212   0.42   0.20   0.00   0.00  pkg.depotd
  819   0.00   0.53   0.00   0.00  named
  943   0.17   0.36   0.00   0.00  poold
   13   0.01   0.47   0.00   0.00  svc.configd
 (truncated output)

```

From the DTraceToolkit website ([http://www.brendangregg.com/dtracetoolkit.html](http://www.brendangregg.com/dtracetoolkit.html)), we can download and save the `topsysproc.d` script in our home directory. Then, by executing it, we are able to find which processes execute more system calls, as shown in the following commands:

```
root@solaris11-1:~/DTraceToolkit-0.99/Proc# ./topsysproc  10
2014 May  4 19:25:10, load average: 0.38, 0.30, 0.28   syscalls: 12648
   PROCESS                          COUNT
   isapython2.6                        20
   sendmail                            20
   dhcpd                               24
   httpd.worker                        30
   updatemanagernot                    40
   nautilus                            42
   xscreensaver                        50
   tput                                59
   gnome-settings-d                    62
   metacity                            75
   VBoxService                         81
   ksh93                              118
   clear                              163
   poold                              201
   pkg.depotd                         615
   VBoxClient                         781
   java                              1249
   gnome-terminal                    2224
   dtrace                            2712
   Xorg                              3965
```

### An overview of the recipe

You learned how to monitor processes using several tools such as `prstat`, `ps`, and `dtrace`. Furthermore, you saw several commands that explain how to control and analyze a process.

# Managing processes' priority on Solaris 11

Oracle Solaris 11 allows us to change the priority of processes using the `priocntl` command either during the start of the process or after the process is run.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running Oracle Solaris 11 with 2 GB RAM at least. It is recommended that the system have more than one processor or core.

## How to do it…

In the *Introduction* section, we talked about scheduling classes and this time, we will see more information on this subject. To begin, list the existing and active classes by executing the following command:

```
root@solaris11-1:~# priocntl -l
CONFIGURED CLASSES
==================
SYS (System Class)

TS (Time Sharing)
  Configured TS User Priority Range: -60 through 60

SDC (System Duty-Cycle Class)

FSS (Fair Share)
  Configured FSS User Priority Range: -60 through 60

FX (Fixed priority)
  Configured FX User Priority Range: 0 through 60

IA (Interactive)
  Configured IA User Priority Range: -60 through 60

RT (Real Time)
  Configured RT User Priority Range: 0 through 59
```

When handling priorities, which we learned in this chapter, only the positive part is important and we need to take care because the values shown in the previous output have their own class as the reference. Thus, they are not absolute values.

To show a simple example, start a process with a determined class (FX) and priority (55) by executing the following commands:

```
root@solaris11-1:~# priocntl -e -c FX -m 60 -p 55 gcalctool
root@solaris11-1:~# ps -efcl | grep gcalctool
 0 S     root  2660  2646   FX  55        ?  33241        ? 04:48:52 pts/1       0:01 gcalctool
 0 S     root  2664  2661  FSS  22        ?   2149        ? 04:50:09 pts/2       0:00 grep gcalctool
```

As can be seen previously, the process is using exactly the class and priority that we have chosen. Moreover, it is appropriate to explain some options such as `-e` (to execute a specified command), `-c` (to set the class), `-p` (the chosen priority inside the class), and `-m` (the maximum limit that the priority of a process can be raised to).

The next exercise is to change the process priority after it starts. For example, by executing the following command, the `top` tool will be executed in the FX class with an assigned priority equal to 40, as shown in the following command:

```
root@solaris11-1:~# priocntl -e -c FX -m 60 -p 40 top
root@solaris11-1:~# ps -efcl | grep top
 0 S     root  2662  2649   FX  40        ?   1112        ? 05:16:21 pts/3       0:00 top
 0 S     root  2664  2606   IA  33        ?   2149        ? 05:16:28 pts/2       0:00 grep top
```

Then, to change the priority that is running, execute the following command:

```
root@solaris11-1:~# priocntl -s -p 50 2662
root@solaris11-1:~# ps -efcl | grep top
 0 S     root  2662  2649   FX  50        ?   1112        ? 05:16:21 pts/3       0:00 top
 0 S     root  2667  2606   IA  55        ?   2149        ? 05:17:00 pts/2       0:00 grep top
```

This is perfect! The `-s` option is used to change the priorities' parameters, and the `–p` option assigns the new priority to the process.

If we tried to use the TS class, the results would not have been the same because this test system does not have a serious load (it's almost idle) and in this case, the priority would be raised automatically to around 59.

### An overview of the recipe

You learned how to configure a process class as well as change the process priority at the start and during its execution using the `priocntl` command.

# Configuring FSS and applying it to projects

The FSS class is the best option to manage resource allocation (for example, CPU) on Oracle Solaris 11\. In this section, we are going to learn how to use it.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running Oracle Solaris 11 with 4 GB RAM at least. It is recommended that the system has only one processor or core.

## How to do it…

In Oracle Solaris 11, the default scheduler class is TS, as shown by the following command:

```
root@solaris11-1:~# dispadmin -d
TS  (Time Sharing)
```

This default configuration comes from the `/etc/dispadmin.conf` file:

```
root@solaris11-1:~# more /etc/dispadmin.conf
#
# /etc/dispadmin.conf
#
# Do NOT edit this file by hand -- use dispadmin(1m) instead.
#
DEFAULT_SCHEDULER=TS
```

If we need to verify and change the default scheduler, we can accomplish this task by running the following commands:

```
root@solaris11-1:~# dispadmin -d FSS
root@solaris11-1:~# dispadmin -d 
FSS  (Fair Share)

root@solaris11-1:~# more /etc/dispadmin.conf 
#
# /etc/dispadmin.conf
#
# Do NOT edit this file by hand -- use dispadmin(1m) instead.
#
DEFAULT_SCHEDULER=FSS 

```

Unfortunately, this new setting only takes effect for newly created processes that are run after the command, but current processes still are running using the previously configured classes (TS and IA), as shown in the following command:

```
root@solaris11-1:~# ps -efcl -o s,uid,pid,zone,class,pri,comm | more
S   UID   PID     ZONE  CLS PRI COMMAND
T     0     0   global  SYS  96 sched
S     0     5   global  SDC  99 zpool-rpool
S     0     6   global  SDC  99 kmem_task
S     0     1   global   TS  59 /usr/sbin/init
S     0     2   global  SYS  98 pageout
S     0     3   global  SYS  60 fsflush
S     0     7   global  SYS  60 intrd
S     0     8   global  SYS  60 vmtasks
S 60002  1173   global   TS  59 /usr/lib/fm/notify/smtp-notify
S     0    11   global   TS  59 /lib/svc/bin/svc.startd
S     0    13   global   TS  59 /lib/svc/bin/svc.configd
S    16    99   global   TS  59 /lib/inet/ipmgmtd
S     0   108   global   TS  59 /lib/inet/in.mpathd
S    17    40   global   TS  59 /lib/inet/netcfgd
S     0   199   global   TS  59 /usr/sbin/vbiosd
S     0   907   global   TS  59 /usr/lib/fm/fmd/fmd
(truncated output)

```

To change the settings from all current processes (the `-i` option) to using FSS (the `-c` option) without rebooting the system, execute the following command:

```
root@solaris11-1:~# priocntl -s -c FSS -i all
root@solaris11-1:~# ps -efcl -o s,uid,pid,zone,class,pri,comm | more
S   UID   PID     ZONE  CLS PRI COMMAND
T     0     0   global  SYS  96 sched
S     0     5   global  SDC  99 zpool-rpool
S     0     6   global  SDC  99 kmem_task
S     0     1   global   TS  59 /usr/sbin/init
S     0     2   global  SYS  98 pageout
S     0     3   global  SYS  60 fsflush
S     0     7   global  SYS  60 intrd
S     0     8   global  SYS  60 vmtasks
S 60002  1173   global  FSS  29 /usr/lib/fm/notify/smtp-notify
S     0    11   global  FSS  29 /lib/svc/bin/svc.startd
S     0    13   global  FSS  29 /lib/svc/bin/svc.configd
S    16    99   global  FSS  29 /lib/inet/ipmgmtd
S     0   108   global  FSS  29 /lib/inet/in.mpathd
S    17    40   global  FSS  29 /lib/inet/netcfgd
S     0   199   global  FSS  29 /usr/sbin/vbiosd
S     0   907   global  FSS  29 /usr/lib/fm/fmd/fmd
S     0  2459   global  FSS  29 gnome-session
S    15    66   global  FSS  29 /usr/sbin/dlmgmtd
S     1    88   global  FSS  29 /lib/crypto/kcfd
S     0   980   global  FSS  29 /usr/lib/devchassis/devchassisd
S     0   138   global  FSS  29 /usr/lib/pfexecd
S     0   277   global  FSS  29 /usr/lib/zones/zonestatd
O     0  2657   global  FSS   1 more
S    16   638   global  FSS  29 /lib/inet/nwamd
S    50  1963   global  FSS  29 /usr/bin/dbus-launch
S     0   291   global  FSS  29 /usr/lib/dbus-daemon
S     0   665   global  FSS  29 /usr/lib/picl/picld
 (truncated output)

```

It's almost done, but the `init` process (PID equal to 1) was not changed to the FSS class, unfortunately. This change operation is done manually, by executing the following commands:

```
root@solaris11-1:~# priocntl -s -c FSS -i pid 1
root@solaris11-1:~# ps -efcl -o s,uid,pid,zone,class,pri,comm | more
S   UID   PID     ZONE  CLS PRI COMMAND
T     0     0   global  SYS  96 sched
S     0     5   global  SDC  99 zpool-rpool
S     0     6   global  SDC  99 kmem_task
S     0     1   global  FSS  29 /usr/sbin/init
S     0     2   global  SYS  98 pageout
S     0     3   global  SYS  60 fsflush
S     0     7   global  SYS  60 intrd
S     0     8   global  SYS  60 vmtasks
S 60002  1173   global  FSS  29 /usr/lib/fm/notify/smtp-notify
S     0    11   global  FSS  29 /lib/svc/bin/svc.startd
S     0    13   global  FSS  29 /lib/svc/bin/svc.configd
S    16    99   global  FSS  29 /lib/inet/ipmgmtd
S     0   108   global  FSS  29 /lib/inet/in.mpathd
(truncated output)

```

From here, it would be possible to use projects (a very nice concept from Oracle Solaris), tasks, and FSS to make an attractive example. It follows a quick demonstration.

You should know that one project can have one or more tasks, and each task has one or more processes (as shown previously in this chapter). From an initial installation, Oracle Solaris 11 already has some default projects, as shown by the following commands:

```
root@solaris11-1:~# projects 
user.root default 
root@solaris11-1:~# projects -l
system
  projid : 0
  comment: ""
  users  : (none)
  groups : (none)
  attribs: 
user.root
  projid : 1
  comment: ""
  users  : (none)
  groups : (none)
  attribs: 
(truncated output)
root@solaris11-1:~# more /etc/project 
system:0::::
user.root:1::::
noproject:2::::
default:3::::
group.staff:10::::  
```

In this exercise, we are going to create four new projects: `ace_proj_1`, `ace_proj_2`, `ace_proj_3`, and `ace_proj_4`. For each project will be associated an amount of shares (40, 30, 20, and 10 respectively). Additionally, it will create some useless but CPU-consuming tasks by starting a Firefox instance.

Therefore, execute the following commands to perform the tasks:

```
root@solaris11-1:~# projadd -U root -K "project.cpu-shares=(priv,40,none)" ace_proj_1
root@solaris11-1:~# projadd -U root -K "project.cpu-shares=(priv,30,none)" ace_proj_2
root@solaris11-1:~# projadd -U root -K "project.cpu-shares=(priv,20,none)" ace_proj_3
root@solaris11-1:~# projadd -U root -K "project.cpu-shares=(priv,10,none)" ace_proj_4
root@solaris11-1:~# projects 
user.root default ace_proj_1 ace_proj_2 ace_proj_3 ace_proj_4

```

Here is where the trick comes in. The FSS class only starts to act when:

*   The total CPU consumption by all processes is over 100 percent
*   The sum of processes from defined projects is over the current number of CPUs

Thus, to be able to see the FSS effect, as explained previously, we have to repeat the next four commands several times (using the Bash history is suitable here), shown as follows:

```
root@solaris11-1:~# newtask -p ace_proj_1 firefox &
[1] 3016
root@solaris11-1:~# newtask -p ace_proj_2 firefox &
[2] 3032
root@solaris11-1:~# newtask -p ace_proj_3 firefox &
[3] 3037
root@solaris11-1:~# newtask -p ace_proj_4 firefox &
[4] 3039
```

As time goes by and the number of tasks increase, each project will be approaching the FSS share limit (40 percent, 30 percent, 20 percent, and 10 percent of processor, respectively). We can follow this trend by executing the next command:

```
root@solaris11-1:~# prstat -JR
PID USERNAME  SIZE   RSS STATE   PRI NICE      TIME  CPU PROCESS/NLWP
  3516 root     8552K 1064K cpu1     49    0   0:01:25  25% dd/1
  3515 root     8552K 1064K run       1    0   0:01:29 7.8% dd/1
  1215 root       89M   29M run      46    0   0:00:56 0.0% Xorg/3
  2661 root       13M  292K sleep    59    0   0:00:28 0.0% VBoxClient/3
   750 root       13M 2296K sleep    55    0   0:00:02 0.0% nscd/32
  3518 root       11M 3636K cpu0     59    0   0:00:00 0.0% 

(truncated output)

PROJID    NPROC  SWAP   RSS MEMORY      TIME  CPU PROJECT
   100        4   33M 4212K   0.1%   0:01:49  35% ace_proj_1
   101        4   33M 4392K   0.1%   0:01:14  28% ace_proj_2
   102        4   33M 4204K   0.1%   0:00:53  20% ace_proj_3
   103        4   33M 4396K   0.1%   0:00:30  11% ace_proj_4
     3        2   10M 4608K   0.1%   0:00:06 0.8% default
     1       41 2105M  489M    12%   0:00:09 0.7% user.root
     0       78  780M  241M   5.8%   0:00:20 0.3% system
```

The `prstat` command with the `–J` option shows a summary of the existing projects, and `–R` requires the kernel to execute the `prstat` command in the RT scheduling class. If the reader faces some problem getting the expected results, it is possible to swap the `firefox` command with the `dd if=/dev/zero of=/dev/null &` command to get the same results.

It is important to highlight that while not all projects take their full share of the CPU, other projects can borrow some shares (percentages). This is the reason why `ace_proj_4` has 11 percent, because `ace_proj_1` has taken only 35 percent (the maximum is 40 percent).

### An overview of the recipe

In this section, you learned how to change the default scheduler from TS to FSS in a temporary and persistent way. Finally, you saw a complete example using projects, tasks, and FSS.

# References

*   *Solaris Performance and Tools: DTrace and MDB Techniques for Solaris 10 and OpenSolaris*; *Brendan Gregg*, *Jim Mauro*, *Richard McDougall*; *Prentice Hall*; ISBN-13: 978-0131568198
*   DTraceToolkit website at [http://www.brendangregg.com/dtracetoolkit.html](http://www.brendangregg.com/dtracetoolkit.html)
*   Dtrace.org website at [http://dtrace.org/blogs/](http://dtrace.org/blogs/)