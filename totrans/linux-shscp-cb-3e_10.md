# Administration Calls

In this chapter, we will cover the following topics:

*   Gathering information about processes
*   What's what – which, whereis, whatis, and file  
*   Killing processes, and sending and responding to signals
*   Sending messages to user terminals
*   The `/proc` filesystem
*   Gathering system information
*   Scheduling with a `cron`
*   Database styles and uses
*   Writing and reading SQLite databases
*   Writing and reading a MySQL database from Bash
*   User administration scripts
*   Bulk image resizing and format conversion
*   Taking screenshots from the terminal
*   Managing multiple terminals from one

# Introduction

Managing multiple terminals from one GNU/Linux ecosystem consists of the network, each set of hardware, the OS Kernel that allocates resources, interface modules, system utilities, and user programs. An administrator needs to monitor the entire system to keep everything running smoothly. Linux administration tools range from all-in-one GUI applications to command-line tools designed for scripting.

# Gathering information about processes

The term **process** in this case means the running instance of a program. Many processes run simultaneously on a computer. Each process is assigned a unique identification number, called a **process ID** (**PID**). Multiple instances of the same program with the same name can run at the same time, but they will each have different PIDs and attributes. Process attributes include the user who owns the process, the amount of memory used by the program, the CPU time used by the program, and so on. This recipe shows how to gather information about processes.

# Getting ready

Important commands related to process management are `top`, `ps`, and `pgrep`. These tools are available in all Linux distributions.

# How to do it...

`ps` reports information about active processes. It provides information about which user owns the process, when the process started, the command path used to execute the process, the PID, the terminal it is attached to (**TTY**, for **TeleTYpe**), the memory used by the process, the CPU time used by the process, and so on. Consider the following example:

```
$ ps
PID TTY       TIME CMD
1220 pts/0    00:00:00 bash
1242 pts/0    00:00:00 ps

```

Be default, `ps` will display the processes initiated from the current terminal (TTY). The first column shows the PID, the second column refers to the terminal (TTY), the third column indicates how much time has elapsed since the process started, and finally we have CMD (the command).

The `ps` command report can be modified with command-line parameters.

The `-f (full)` option displays more columns of information:

```
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
slynux    1220  1219  0 18:18 pts/0    00:00:00 -bash
slynux    1587  1220  0 18:59 pts/0    00:00:00 ps -f

```

The `-e` (every) and `-ax` (all) options provide a report on every process that is running on the system.

The `-x` argument (along with `-a`) specifies the removal of the default TTY restriction imparted by `ps`. Usually, if you use `ps` without arguments, it'll only print processes attached to the current terminal.

The commands `ps -e`, `ps -ef`, `ps -ax`, and `ps -axf` generate reports on all processes and provide more information than `ps`:

```
$ ps -e | head -5
PID TTY    TIME CMD
1 ?        00:00:00 init
2 ?        00:00:00 kthreadd
3 ?        00:00:00 migration/0
4 ?        00:00:00 ksoftirqd/0

```

The `-e` option generates a long report. This example filters the output with `head` to display the first five entries.

The `-o PARAMETER1`, `PARAMETER2` option specifies the data to be displayed.

Parameters for `-o` are delimited with a comma (`,`). There is no space between the comma operator and the next parameter.
The `-o` option can be combined with the `-e` (every) option (`-eo`) to list every process running in the system. However, when you use filters similar to the ones that restrict `ps` to the specified users along with `-o`, `-e` is not used. The -e option overrules the filter and displays all the processes.

In this example, `comm` stands for COMMAND and `pcpu` represents the percentage of CPU usage:

```
$ ps -eo comm,pcpu | head -5
COMMAND          %CPU
init             0.0
kthreadd         0.0
migration/0      0.0
ksoftirqd/0      0.0

```

# How it works...

The following parameters for the `-o` option are supported:

| **Parameter** | **Description** |
| `pcpu` | Percentage of CPU |
| `pid` | Process ID |
| `ppid` | Parent process ID |
| `pmem` | Percentage of memory |
| `comm` | Executable filename |
| `cmd` | A simple command |
| `user` | The user who started the process |
| `nice` | The priority (niceness) |
| `time` | Cumulative CPU time |
| `etime` | Elapsed time since the process started |
| `tty` | The associated TTY device |
| `euid` | The effective user |
| `stat` | Process state |

# There's more...

The `ps` command, `grep`, and other tools can be combined to produce custom reports.

# Showing environment variables for a process

Some processes are dependent on their environment variable definitions. Knowing the environment variables and values can help you debug or customize a process.

The `ps` command does not normally show the environment information of a command. The `e` output modifier at the end of the command adds this information to the output:

```
$ ps e

```

Here's an example of environment information:

```
$ ps -eo pid,cmd  e | tail -n 1
1238 -bash USER=slynux LOGNAME=slynux HOME=/home/slynux PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAIL=/var/mail/slynux SHELL=/bin/bash SSH_CLIENT=10.211.55.2 49277 22 SSH_CONNECTION=10.211.55.2 49277 10.211.55.4 22 SSH_TTY=/dev/pts/0 

```

Environment information helps trace problems using the `apt-get` package manager. If you use an HTTP proxy to connect to the Internet, you may need to set environment variables using `http_proxy=host:port`. If this is not set, the `apt-get` command will not select the proxy and hence returns an error. Knowing that `http_proxy` is not set makes the problem obvious.

When a scheduling tool, such as `cron` (discussed later in this chapter), is used to run an application, the expected environment variables may not be set. This `crontab` entry will not open a GUI-windowed application:

```
00 10 * * * /usr/bin/windowapp

```

It fails because GUI applications require the `DISPLAY` environment variable. To determine the required environment variables, run `windowapp` manually and then `ps -C windowapp -eo cmd e`.

After you've identified the required environment variables, define them before the command name in `crontab`:

```
00 10 * * * DISPLAY=:0 /usr/bin/windowapp

```

OR

```
DISPLAY=0
00 10 * * * /usr/bin/windowapp

```

The definition `DISPLAY=:0` was obtained from the `ps` output.

# Creating a tree view of processes

The `ps` command can report a process PID, but tracking from a child to the ultimate parent is tedious. Adding `f` to the end of the `ps` command creates a tree view of the processes, showing the parent-child relationship between tasks. The next example shows an `ssh` session invoked from a bash shell running inside `xterm`:

```
$ ps -u clif f | grep -A2 xterm | head -3
15281  ?      S     0:00 xterm 
15284 pts/20  Ss+   0:00 \_ bash
15286 pts/20  S+    0:18 \_ ssh 192.168.1.2

```

# Sorting ps output

By default, the `ps` command output is unsorted. The -sort parameter forces `ps` to sort the output. The ascending or descending order can be specified by adding the `+` (ascending) or `-` (descending) prefix to the parameter:

```
$ ps [OPTIONS] --sort -paramter1,+parameter2,parameter3..

```

For example, to list the top five CPU-consuming processes, use the following:

```
$ ps -eo comm,pcpu --sort -pcpu | head -5
COMMAND         %CPU
Xorg             0.1
hald-addon-stor  0.0
ata/0            0.0
scsi_eh_0        0.0

```

This displays the top five processes, sorted in descending order by percentage of CPU usage.

The `grep` command can filter the `ps` output. To report only those Bash processes that are currently running, use the following:

```
$ ps -eo comm,pid,pcpu,pmem | grep bash
bash             1255  0.0  0.3
bash             1680  5.5  0.3

```

# Filters with ps for real user or ID, effective user or ID

The `ps` command can group processes based on the real and effective usernames or IDs specified. The `ps` command filters the output by checking whether each entry belongs to a specific effective user or a real user from the list of arguments.

*   Specify an effective user's list with `-u EUSER1`, `EUSER2`, and so on
*   Specify a real user's list with `-U RUSER1`, `RUSER2`, and so on

Here's an example of this:

```
 # display user and percent cpu usage for processes with real user
 # and effective user of root
 $ ps -u root -U root -o user,pcpu

```

The `-o` may be used with `-e` as `-eo` but when filters are applied, `-e` should not be used. It overrides the filter options.

# TTY filter for ps

The `ps` output can be selected by specifying the TTY to which the process is attached. Use the `-t` option to specify the TTY list:

```
 $ ps -t TTY1, TTY2 ..

```

Here's an example of this:

```
 $ ps -t pts/0,pts/1
 PID TTY          TIME CMD
 1238 pts/0    00:00:00 bash
 1835 pts/1    00:00:00 bash
 1864 pts/0    00:00:00 ps

```

# Information about process threads

The `-L` option to `ps` will display information about process threads. This option adds an LWP column to the thread ID. Adding the `-f` option to `-L` (`-Lf`) adds two columns: NLWP, the thread count, and LWP, the thread ID:

```
 $ ps -Lf
 UID  PID  PPID  LWP  C  NLWP  STIME  TTY  TIME     
        CMD
 user 1611 1     1612 0  2     Jan16  ?    00:00:00    
        /usr/lib/gvfs/gvfsd

```

This command lists five processes with a maximum number of threads:

```
$ ps -eLf --sort -nlwp | head -5
UID        PID  PPID   LWP  C NLWP STIME TTY  TIME     
     CMD
root       647     1   647  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon
root       647     1   654  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon
root       647     1   656  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon
root       647     1   657  0   64 14:39 ?    00:00:00 
     /usr/sbin/console-kit-daemon --no-daemon

```

# Specifying the output width and columns to be displayed

The `ps` command supports many options to select fields in order to display and control how they are displayed. Here are some of the more common options:

| `-f` | This specifies a full format. It includes the starting time of the parent PID user ID. |
| `-u` userList | This selects processes owned by the users in the list. By default, it selects the current user. |
| `-l` | Long listing. It displays the user ID, parent PID, size, and more. |

# What's what – which, whereis, whatis, and file

There may be several files with the same name. Knowing which executable is being invoked and whether a file is compiled code or a script is useful information.

# How to do it...

The `which`, `whereis`, `file`, and `whatis` commands report information about files and directories.

*   `which`: The which command reports the location of a command:

```
        $ which ls
 /bin/ls

```

*   We often use commands without knowing the directory where the executable file is stored. Depending on how your `PATH` variable is defined, you may use a command from `/bin`, `/usr/local/bin`, or `/opt/PACKAGENAME/bin`.
*   When we type a command, the terminal looks for the command in a set of directories and executes the first executable file it finds. The directories to search are specified in the `PATH` environment variable:

```
 $ echo $PATH /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

```

*   We can add directories to be searched and export the new `PATH`. To add `/opt/bin` to `PATH`, use the following command:

```
        $ export PATH=$PATH:/opt/bin
 # /opt/bin is added to PATH

```

*   **whereis**: `whereis` is similar to the which command. It not only returns the path of the command, but also prints the location of the man page (if available) and the path for the source code of the command (if available):

```
        $ whereis ls
 ls: /bin/ls /usr/share/man/man1/ls.1.gz

```

*   **whatis**: The `whatis` command outputs a one-line description of the command given as the argument. It parses information from the `man` page:

```
        $ whatis ls
 ls (1)    - list directory contents

```

The `file` command reports a file type. Its syntax is as follows:

```
        $ file FILENAME

```

*   The reported file type may comprise a few words or a long description:

```
        $file /etc/passwd
 /etc/passwd: ASCII text
 $ file /bin/ls
 /bin/ls: ELF 32-bit LSB executable, Intel 80386, version 1   
        (SYSV), dynamically linked (uses shared libs), for GNU/Linux    
        2.6.15, stripped

```

apropos
Sometimes we need to search for a command that is related to the topic. The `apropos` command will search the man pages for a keyword. Here's the code to do this: **Apropos topic**

# Finding the process ID from the given command names

Suppose several instances of a command are being executed. In such a scenario, we need the PID of each process. Both the `ps` and `pgrep` command return this information:

```
 $ ps -C COMMAND_NAME

```

Alternatively, the following is returned:

```
 $ ps -C COMMAND_NAME -o pid=

```

When `=` is appended to `pid`, it removes the header PID from the output of `ps`. To remove headers from a column, append `=` to the parameter.

This command lists the process IDs of Bash processes:

```
 $ ps -C bash -o pid=
 1255
 1680

```

The `pgrep` command also returns a list of process IDs for a command:

```
 $ pgrep bash
 1255
 1680

```

`pgrep` requires only a portion of the command name as its input argument to extract a Bash command; `pgrep ash` or `pgrep bas` will also work, for example. But `ps` requires you to type the exact command. `pgrep` supports these output-filtering options.

The `-d` option specifies an output delimiter other than the default new line:

```
 $ pgrep COMMAND -d DELIMITER_STRING
 $ pgrep bash -d ":"
 1255:1680

```

The `-u` option filters for a list of users:

```
 $ pgrep -u root,slynux COMMAND

```

In this command, `root` and `slynux` are users.

The `-c` option returns the count of matching processes:

```
 $ pgrep -c COMMAND

```

# Determining how busy a system is

Systems are either unused or overloaded. The `load average` value describes the total load on the running system. It describes the average number of runnable processes, processes with all resources except CPU time slices, on the system.

Load average is reported by the uptime and top commands. It is reported with three values. The first value indicates the average in 1 minute, the second indicates the average in 5 minutes, and the third indicates the average in 15 minutes.

It is reported by uptime:

```
 $ uptime
 12:40:53 up  6:16,  2 users,  load average: 0.00, 0.00, 0.00

```

# The top command

By default, the `top` command displays a list of the top CPU-consuming processes as well as basic system statistics, including the number of tasks in the process list, CPU cores, and memory usage. The output is updated every few seconds.

This command displays several parameters along with the top CPU-consuming processes:

```
    $ top
 top - 18:37:50 up 16 days, 4:41,7 users,load average 0.08 0.05 .11
 Tasks: 395 total,  2 running, 393 sleeping, 0 stopped 0 zombie

```

# See also...

*   The *Scheduling with a cron* recipe in this chapter explains how to schedule tasks

# Killing processes, and sending and responding to signals

You may need to kill processes (if they go rogue and start consuming too many resources) if you need to reduce system load, or before rebooting. Signals are an inter-process communication mechanism that interrupts a running process and forces it to perform some action. These actions include forcing a process to terminate in either a controlled or immediate manner.

# Getting ready

Signals send an interrupt to a running program. When a process receives a signal, it responds by executing a signal handler. Compiled applications generate signals with the `kill` system call. A signal can be generated from the command line (or shell script) with the `kill` command. The `trap` command can be used in a script to handle received signals.

Each signal is identified by a name and an integer value. The `SIGKILL (9)` signal terminates a process immediately. The keystroke events *Ctrl* + *C* and *Ctrl* + *Z* send signals to abort or put the task in the background.

# How to do it...

1.  The kill `-l` command will list the available signals:

```
        $ kill -l
 SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP
 ...

```

2.  Terminate the process:

```
        $ kill PROCESS_ID_LIST

```

The `kill` command issues a `SIGTERM` signal by default. The process ID list is specified with spaces for delimiters.

3.  The `-s` option specifies the signal to be sent to the process:

```
        $ kill -s SIGNAL PID

```

The `SIGNAL` argument is either a signal name or a signal number. There are many signals available for different purposes. The most common ones are as follows:

*   `SIGHUP 1`: Hangup detection on the death of the controlling process or terminal

*   `SIGINT 2`: This is the signal emitted when *Ctrl* + *C* is pressed

*   `SIGKILL 9`: This is the signal used to forcibly kill the process

*   `SIGTERM 15`: This is the signal used to terminate a process by default

*   `SIGTSTP 20`: This is the signal emitted when *Ctrl* + *Z* is pressed

4.  We frequently use force kill for processes. Use this with caution. This is an immediate action, and it will not save data or perform a normal cleanup operation. The `SIGTERM` signal should be tried first; `SIGKILL` should be saved for extreme measures:

```
        $ kill -s SIGKILL PROCESS_ID

```

Alternatively, use this to perform the cleanup operation:

```
        $ kill -9 PROCESS_ID

```

# There's more...

Linux supports other commands to signal or terminate processes.

# The kill family of commands

The `kill` command takes the process ID as the argument. The `killall` command terminates the process by name:

```
    $ killall process_name

```

The `-s` option specifies the signal to send. By default, `killall` sends a `SIGTERM` signal:

```
    $ killall -s SIGNAL process_name

```

The `-9` option forcibly kills a process by name:

```
    $ killall -9 process_name

```

Here's an example of the preceding:

```
    $ killall -9 gedit

```

The `-u` owner specifies the process's user:

```
    $ killall -u USERNAME process_name

```

The `-I` option makes `killall run` in interactive mode:

The `pkill` command is similar to the `kill` command, but by default it accepts a process name instead of a process ID:

```
    $ pkill process_name
 $ pkill -s SIGNAL process_name

```

`SIGNAL` is the signal number. The `SIGNAL` name is not supported with `pkill`. The `pkill` command provides many of the same options as the `kill` command. Check the `pkill` man pages for more details.

# Capturing and responding to signals

Well-behaved programs save data and shut down cleanly when they receive a `SIGTERM` signal. The `trap` command assigns a signal handler to signals in a script. Once a function is assigned to a signal using the `trap` command, when a script receives a signal, this function is executed.

The syntax is as follows:

```
    trap 'signal_handler_function_name' SIGNAL LIST

```

`SIGNAL LIST` is space-delimited. It can include both signal numbers and signal names.

This shell script responds to the `SIGINT` signal:

```
#/bin/bash 
#Filename: sighandle.sh 
#Description: Signal handler  

function handler() 
{ 
  echo Hey, received signal : SIGINT 
} 

# $$ is a special variable that returns process ID of current  
# process/script 

echo My process ID is $$ 

#handler is the name of the signal handler function for SIGINT signal 
trap 'handler' SIGINT 

while true; 
do 
  sleep 1 
done 

```

Run this script in a terminal. When the script is running, pressing *Ctrl* + *C* it will show the message by executing the signal handler associated with it. *Ctrl* + *C* corresponds to a `SIGINT` signal.

The `while` loop is used to keep the process running forever without being terminated. This is done so the script can respond to signals. The loop to keep a process alive infinitely is often called the **event loop**.

If the process ID of the script is given, the `kill` command can send a signal to it:

```
    $ kill -s SIGINT PROCESS_ID

```

The process ID of the preceding script will be printed when it is executed; alternatively, you can find it with the `ps` command.

If no signal handlers are specified for signals, a script will call the default signal handlers assigned by the operating system. Generally, pressing *Ctrl* + *C* will terminate a program, as the default handler provided by the operating system will terminate the process. The custom handler defined here overrides the default handler.

We can define signal handlers for any signals available (`kill -l`) with the `trap` command. A single signal handler can process multiple signals.

# Sending messages to user terminals

Linux supports three applications to display messages on another user's screen. The `write` command sends a message to a user, the `talk` command lets two users have a conversation, and the `wall` command sends a message to all users.

Before doing something potentially disruptive (say, rebooting the server), the system administrator should send a message to the terminal of every user on the system or network.

# Getting ready

The `write` and `wall` commands are part of most Linux distributions. If a user is logged in multiple times, you may need to specify the terminal you wish to send a message to.

You can determine a user's terminals with the `who` command:

```
    $> who
 user1    pts/0    2017-01-16 13:56 (:0.0)
 user1    pts/1    2017-01-17 08:35 (:0.0)

```

The second column (`pts/#`) is the user's terminal identifier.

The `write` and `wall` programs work on a single system. The `talk` program can connect users across a network.

The talk program is not commonly installed. Both the talk program and talk server must be installed and running on any machine where talk is used. Install the talk application as `talk` and `talkd` on Debian-based systems or as `talk` and `talk-server` on Red Hat-based systems. You will probably need to edit `/etc/xinet.d/talk` and `/etc/xinet.d/ntalk` to set the `disable` field to `no`. Once you do this, restart `xinet`:

```
    # cd /etc/xinet.d
 # vi ntalk
 # cd /etc/init.d
 #./xinetd restart

```

# How to do it...

# Sending one message to one user

The write command will send a message to a single user:

```
    $ write USERNAME [device]

```

You can redirect a message from a file or an echo or write interactively. An interactive write is terminated with Ctrl-D.

The message can be directed to a specific session by appending the pseudo terminal identifier to the command:

```
    $ echo "Log off now. I'm rebooting the system" | write user1 pts/3

```

# Holding a conversation with another user

The talk command opens an interactive conversation between two users. The syntax for this is `$ talk user@host`.

The next command initiates a conversation with user2 on their workstation:

```
    $ talk user2@workstation2.example.com

```

After typing the talk command, your terminal session is cleared and split into two windows. In one of the windows, you'll see text like this:

```
    [Waiting for your party to respond]

```

The person you're trying to talk to will see a message like this:

```
    Message from Talk_Daemon@workstation1.example.com
 talk: connection requested by user1@workstation.example.com
 talk: respond with talk user1@workstation1.example.com

```

When they invoke talk, their terminal session will also be cleared and split. What you type will appear in one window on their screen and what they type will appear on yours:

```
    I need to reboot the database server.
 How much longer will your processing take?
 ---------------------------------------------
 90% complete. Should be just a couple more minutes.

```

# Sending a message to all users

The **wall** (WriteALL) command broadcasts a message to all the users and terminal sessions:

```
    $ cat message | wall

```

Or:

```
    $ wall < message
 Broadcast Message from slynux@slynux-laptop
 (/dev/pts/1) at 12:54 ...

 This is a message

```

The message header shows who sent the message: which user and which host.

The write, talk, and wall commands only deliver messages between users when the write message option is enabled. Messages from the root are displayed regardless of the write message option.

The message option is usually enabled. The `mesg` command will enable or disable the receiving of messages:

```
    # enable receiving messages
 $ mesg y
 # disable receiving messages
 $ mesg n

```

# The /proc filesystem

`/proc` is an in-memory pseudo filesystem that provides user-space access to many of the Linux kernel's internal data structures. Most pseudo files are read-only, but some, such as `/proc/sys/net/ipv4/forward` (described in [Chapter 8](5ba784d5-fa8b-4840-b4c5-cac906e484f9.xhtml), *The Old-Boy Network*), can be used to fine-tune your system's behavior.

# How to do it...

The `/proc` directory contains several files and directories. You can view most files in `/proc` and their subdirectories with `cat`, `less`, or `more`. They are displayed as plain text.

Every process running on a system has a directory in `/proc`, named according to the process's PID.

Suppose Bash is running with PID `4295` (`pgrep bash`); in this case, `/proc/4295` will exist. This folder will contain information about the process. The files under `/proc/PID` include:

*   `environ`: This contains the environment variables associated with the process. `cat /proc/4295/environ` will display the environment variables passed to the process `4295`.
*   `cwd`: This is a `symlink` to the process's working directory.
*   `exe`: This is a `symlink` to the process's executable:

```
        $ readlink /proc/4295/exe
 /bin/bash

```

*   `fd`: This is the directory consisting of entries on file descriptors used by the process. The values 0, 1, and 2 are stdin, stdout, and stderr, respectively.
*   `io`: This file displays the number of characters read or written by the process.

# Gathering system information

Describing a computer system requires many sets of data. This data includes network information, the hostname, kernel version, Linux distribution name, CPU description, memory allocation, disk partitions, and more. This information can be retrieved from the command line.

# How to do it...

1.  The `hostname` and `uname` commands print the hostname of the current system:

```
        $ hostname

```

Alternatively, they print the following:

```
        $ uname -n
 server.example.com

```

2.  The `-a` option to `uname` prints details about the Linux kernel version, hardware architecture, and more:

```
        $ uname -a
 server.example.com 2.6.32-642.11.1.e16.x86_64 #1 SMP Fri Nov 18   
        19:25:05 UTC 2016 x86_64 x86_64 GNU/Linux

```

3.  The `-r` option limits the report to the kernel release:

```
        $ uname -r
 2.6.32-642.11.1.e16.x86_64

```

4.  The `-m` option prints the machine type:

```
        $ uname -m
 x86_64

```

5.  The `/proc/` directory holds information about the system, modules, and running processes. `/proc/cpuinfo` contains CPU details:

```
        $ cat /proc/cpuinfo
 processor     : 0
 vendor_id     : GenuineIntel
 cpu family    : 6
 model         : 63
 model name    : Intel(R)Core(TM)i7-5820K CPU @ 3.30GHz
 ...

```

If the processor has multiple cores, these lines will be repeated n times. To extract only one item of information, use `sed`. The fifth line contains the processor name:

```
        $ cat /proc/cpuinfo | sed -n 5p
 Intel(R)CORE(TM)i7-5820K CPU @ 3.3 GHz

```

6.  `/proc/meminfo` contains information about the memory and current RAM usage:

```
        $ cat /proc/meminfo
 MemTotal:     32777552 kB
 MemFree:      11895296 kB 
 Buffers:        634628 kB
 ...

```

The first line of `meminfo` shows the system's total RAM:

```
        $ cat /proc/meminfo  | head -1
 MemTotal:        1026096 kB

```

7.  `/proc/partitions` describes the disk partitions:

```
        $ cat /proc/partitions
 major minor  #blocks  name
 8        0 976762584 sda
 8        1    512000 sda1
 8        2 976248832 sda2
 ...

```

 The `fdisk` program edits a disk's partition table and also reports the current partition table. Run this command as  `root`:

```
        $ sudo fdisk -l

```

8.  The `lshw` and `dmidecode` applications generate long and complete reports about your system. The report includes information about the motherboard, BIOS, CPU, memory slots, interface slots, disks, and more. These must be run as root. `dmidecode` is commonly available, but you may need to install `lshw`:

```
        $ sudo lshw
 description: Computer
 product: 440BX
 vendor: Intel
 ...

 $ sudo dmidecode
 SMBIOS 2.8 present
 115 structures occupying 4160 bytes.
 Table at 0xDCEE1000.

 BIOS Information
 Vendor: American Megatrends Inc
 ...

```

# Scheduling with a cron

The GNU/Linux system supports several utilities for scheduling tasks. The `cron` utility is the most widely supported. It allows you to schedule tasks to be run in the background at regular intervals. The `cron` utility uses a table (crontab) with a list of scripts or commands to be executed and the time when they are to be executed.

Cron is used to schedule system housekeeping tasks, such as performing backups, synchronizing the system clocking with `ntpdate`, and removing temporary files.

A regular user might use `cron` to schedule Internet downloads to happen late at night when their ISP allows drop caps and the available bandwidth is higher.

# Getting ready

The `cron` scheduling utility comes with all GNU/Linux distributions. It scans the `cron` tables to determine whether a command is due to be run. Each user has their own `cron` table, which is a plain text file. The `crontab` command manipulates the `cron` table.

# How to do it...

A `crontab` entry specifies the time to execute a command and the command to be executed. Each line in the `cron` table defines a single command. The command can either be a script or a binary application. When `cron` runs a task, it runs as the user who created the entry, but it does not source the user's `.bashrc`. If the task requires environment variables, they must be defined in the `crontab`.

Each cron table line consists of six space-delimited fields in the following order:

*   `Minute` (0 - 59)
*   `Hour` (0 - 23)
*   `Day` (1 - 31)
*   `Month` (1 - 12)
*   `Weekday` (0 - 6)
*   `COMMAND` (the script or command to be executed at the specified time)

The first five fields specify the time when an instance of the command is to be executed. Multiple values are delimited by commas (no spaces). A star signifies that any time or any day will match. A division sign schedules the event to trigger every /Y interval *(*/5* in minutes means every five minutes).

1.  Execute the `test.sh` script at the 2^(nd) minute of all hours on all days:

```
        02 * * * * /home/slynux/test.sh

```

2.  Execute **test.sh** on the 5^(th), 6^(th), and 7^(th) hours on all days:

```
        00 5,6,7 * * /home/slynux/test.sh

```

3.  Execute `script.sh` every other hour on Sundays:

```
        00 */2 * * 0 /home/slynux/script.sh

```

4.  Shut down the computer at 2 a.m. every day:

```
        00 02 * * * /sbin/shutdown -h

```

5.  The `crontab` command can be used interactively or with prewritten files.

Use the `-e` option with `crontab` to edit the `cron` table:

```
        $ crontab -e
 02 02 * * * /home/slynux/script.sh

```

When `crontab -e` is entered, the default text editor (usually `vi`) is opened and the user can type the `cron` jobs and save them. The `cron` jobs will be scheduled and executed at specified time intervals.

6.  The `crontab` command can be invoked from a script to replace the current crontab with a new one. Here's how you do this:
    *   Create a text file (for example, `task.cron`) with the `cron` job in it and then run `crontab` with this filename as the command argument:

```
                $ crontab task.cron

```

*   Alternatively, specify the `cron` job as an inline function without creating a separate file. For example, refer to the following:

```
                $ crontab<<EOF
 02 * * * * /home/slynux/script.sh
 EOF

```

 The `cron` job needs to be written between `crontab<<EOF` and `EOF`.

# How it works...

An asterisk (`*`) specifies that the command should be executed at every instance during the given time period. A `*` in the `Hour` field in the `cron` job will cause the command to be executed every hour. To execute the command at multiple instances of a time period, specify the time intervals separated by a comma in this time field. For example, to run the command at the 5^(th) and 10^(th) minute, enter `5,10` in the `Minute` field. A slash (divide by) symbol will cause the command to run as per a division of the time. For example 0-30/6 in the Minutes field will run a command every 5 minutes during the first half of each hour. The string `*/12` in the Hours field will run a command every other hour.

Cron jobs are executed as the user who created `crontab`. If you need to execute commands that require higher privileges, such as shutting down the computer, run the `crontab` command as root.

The commands specified in a cron job are written with the full path to the command. This is because cron does not source your `.bashrc`, so the environment in which a cron job is executed is different from the bash shell we execute on a terminal. Hence, the `PATH` environment variable may not be set. If your command requires certain environment variables, you must explicitly set them.

# There's more...

The `crontab` command has more options.

# Specifying environment variables

Many commands require environment variables to be set properly for execution. The cron command sets the SHELL variable to `"/bin/sh``"` and also sets `LOGNAME` and `HOME` from the values in `/etc/passwd`. If other variables are required, they can be defined in the `crontab`. These can be defined for all tasks or individually for a single task.

If the `MAILTO` environment variable is defined, `cron` will send the output of the command to that user via an e-mail.

The `crontab` defines environment variables by inserting a line with a variable assignment statement in the user's `cron` table.

The following `crontab` defines an `http_proxy` environment variable to use a proxy server for Internet interactions:

```
    http_proxy=http://192.168.0.3:3128
 MAILTO=user@example.com
 00 * * * * /home/slynux/download.sh

```

This format is supported by `vixie-cron`, used in Debian, Ubunto, and CentOS distributions. For other distributions, environment variables can be defined on a per-command basis:

```
    00 * * * * http_proxy=http:192.168.0.2:3128;   
    /home/sylinux/download.sh

```

# Running commands at system start-up/boot

Running specific commands when the system starts (or boots) is a common requirement. Some `cron` implementations support a `@reboot` time field to run a job during the reboot process. Note that this feature is not supported by all `cron` implementations and only root is allowed to use this feature on some systems. Now check out the following code:

```
    @reboot command

```

This will run the command as your user at runtime.

# Viewing the cron table

The `-l` option to crontab will list the current user's crontab:

```
    $ crontab -l
 02 05 * * * /home/user/disklog.sh

```

Adding the `-u` option will specify a user's crontab to view. You must be logged in as root to use the `-u` option:

```
    # crontab -l -u slynux
 09 10 * * * /home/slynux/test.sh

```

# Removing the cron table

The `-r` option will remove the current user's cron table:

```
    $ crontab -r

```

The `-u` option specifies the crontab to remove. You must be a root user to remove another user's crontab:

```
    # crontab -u slynux -r

```

# Database styles and uses

Linux supports many styles of databases, ranging from simple text files (`/etc/passwd`) to low level B-Tree databases (Berkely DB and bdb), lightweight SQL (sqlite), and fully featured relational database servers, such as Postgres, Oracle, and MySQL.

One rule of thumb for selecting a database style is to use the least complex system that works for your application. A text file and `grep` is sufficient for a small database when the fields are known and fixed.

Some applications require references. For example, a database of books and authors should be created with two tables, one for books and one for the authors, to avoid duplicating the author information for each book.

If the table is read more often than it's modified, then SQLite is a good choice. This database engine does not require a server, which makes it portable and easy to embed in another application (as Firefox does).

If the database is modified frequently by multiple tasks (for example, a webstore's inventory system), then one of the RDBMS systems, such as Postgres, Oracle, or MySQL, is appropriate.

# Getting ready

You can create a text-based database with standard shell tools. SqlLite is commonly installed by default; the executable is `sqlite3`. You'll need to install MySQL, Oracle, and Postgres. The next section will explain how to install MySQL. You can download Oracle from www.oracle.com. Postgres is usually available with your package manager.

# How to do it...

A text file database can be built with common shell tools.

To create an address list, create a file with one line per address and fields separated by a known character. In this case, the character is a tilde (`~`):

```
    first last~Street~City, State~Country~Phone~

```

For instance:

```
    Joe User~123 Example Street~AnyTown, District~1-123-123-1234~

```

Then add a function to find lines that match a pattern and translate each line into a human-friendly format:

```
    function  addr {
 grep $1 $HOME/etc/addr.txt | sed 's/~/\n/g'
 }

```

When in use, this would resemble the following:

```
    $ addr Joe
 Joe User
 123 Example Street 
 AnyTown District
 1-123-123-1234

```

# There's more...

The SQLite, Postgres, Oracle, and MySQL database applications provide a more powerful database paradigm known as relational databases. A relational database stores relations between tables, for example, the relation between a book and its author.

A common way to interact with a relational database is using SQL. This language is supported by SQLite, Postgres, Oracle, MySQL, and other database engines.

SQL is a rich language. You can read books devoted to it. Luckily, you just need a few commands to use SQL effectively.

# Creating a table

Tables are defined with the `CREATE TABLE` command:

```
 CREATE TABLE tablename (field1 type1, field2 type2,...); 

```

The next line creates a table of books and authors:

```
 CREATE TABLE book (title STRING, author STRING); 

```

# Inserting a row into an SQL database

The insert command will insert a row of data into the database.

```
 INSERT INTO table (columns) VALUES (val1, val2,...); 

```

The following command inserts the book you're currently reading:

```
 INSERT INTO book (title, author) VALUES ('Linux Shell Scripting 
 Cookbook', 'Clif Flynt'); 

```

# Selecting rows from a SQL database

The select command will select all the rows that match a test:

```
 SELECT fields FROM table WHERE test; 

```

This command will select book titles that include the word Shell from the book table:

```
 SELECT title FROM book WHERE title like '%Shell%'; 

```

# Writing and reading SQLite databases

SQLite is a lightweight database engine that is used in applications ranging from Android apps and Firefox to US Navy inventory systems. Because of the range of use, there are more applications running SQLite than any other database.

A SQLite database is a single file that is accessed by one or more database engines. The database engine is a C library that can be linked to an application; it is loaded as a library to a scripting language, such as TCL, Python, or Perl, or run as a standalone program.

The standalone application sqlite3 is the easiest to use within a shell script.

# Getting ready

The `sqlite3` executable may not be installed in your installation. If it is not, it can be installed by loading the `sqlite3` package with your package manager.

For Debian and Ubuntu, use the following:

```
    apt-get install sqlite3 libsqlite3-dev

```

For Red Hat, SuSE, Fedora, and Centos, use the following:

```
    yum install sqlite sqlite-devel

```

# How to do it...

The `sqlite3` command is an interactive database engine that connects to a SQLite database and supports the process of creating tables, inserting data, querying tables, and so on.

The syntax of the `sqlite3` command is this:

```
    sqlite3 databaseName

```

If the `databaseName` file exists, `sqlite3` will open it. If the file does not exist, `sqlite3` will create an empty database. In this recipe, we will create a table, insert one row, and retrieve that entry:

```
    # Create a books database
 $ sqlite3 books.db
 sqlite> CREATE TABLE books (title string, author string);
 sqlite> INSERT INTO books (title, author) VALUES ('Linux Shell      
    Scripting Cookbook', 'Clif Flynt');
 sqlite> SELECT * FROM books WHERE author LIKE '%Flynt%';
 Linux Shell Scripting Cookbook|Clif Flynt

```

# How it works...

The `sqlite3` application creates an empty database named `books.db` and displays the `sqlite> prompt` to accept SQL commands.

The `CREATE TABLE` command creates a table with two fields: title and author.

The `INSERT` command inserts one book into the database. Strings in SQL are delimited with single quotes.

The `SELECT` command retrieves the rows that match the test. The percentage symbol (`%`) is the SQL wildcard, similar to a star (`*`) in the shell.

# There's more...

A shell script can use `sqlite3` to access a database and provide a simple user interface. The next script implements the previous address database with `sqlite` instead of a flat text file. It provides three commands:

*   `init`: This is to create the database
*   `insert`: This is to add a new row
*   `query`: This is to select rows that match a query

In use, it would look like this:

```
    $> dbaddr.sh init
 $> dbaddr.sh insert 'Joe User' '123-1234' 'user@example.com'
 $> dbaddr.sh query name Joe
 Joe User
 123-1234
 user@example.com

```

The following script implements this database application:

```
    #!/bin/bash
    # Create a command based on the first argument

    case $1 in
      init )
        cmd="CREATE TABLE address \
          (name string, phone string, email string);" ;;
      query )
        cmd="SELECT name, phone, email FROM address \
          WHERE $2 LIKE '$3';";;
      insert )
        cmd="INSERT INTO address (name, phone, email) \
          VALUES ( '$2', '$3', '$4' );";;
    esac

    # Send the command to sqlite3 and reformat the output

    echo $cmd | sqlite3 $HOME/addr.db | sed 's/|/\n/g'

```

This script uses the case statement to select the SQL command string. The other command-line arguments are replaced with this string and the string is sent to `sqlite3` to be evaluated. The `$1`, `$2`, `$3`, and `$4` are the first, second, third, and fourth arguments, respectively, to the script.

# Writing and reading a MySQL database from Bash

MySQL is a widely used database management system. In 2009, Oracle acquired SUN and with that the MySQL database. The MariaDB package is a fork of the MySQL package that is independent of Oracle. MariaDB can access MySQL databases, but MySQL engines cannot always access MariaDB databases.

Both MySQL and MariaDB have interfaces for many languages, including PHP, Python, C++, Tcl, and more. All of them use the `mysql` command to provide an interactive session in order to access a database. This is the easiest way for a shell script to interact with a MySQL database. These examples should work with either MySQL or MariaDB.

A bash script can convert a text or **Comma-Separated Values** (**CSV**) file into MySQL tables and rows. For example, we can read all the e-mail addresses stored in a guestbook program's database by running a query from the shell script.

The next set of scripts demonstrates how to insert the contents of the file into a database table of students and generate a report while ranking each student within the department.

# Getting ready

MySQL and MariaDB are not always present in the base Linux distribution. They can be installed as either `mysql-server` and `mysql-client` or the `mariadb-server` package. The MariaDB distribution uses MySQL as a command and is sometimes installed when the MySQL package is requested.

MySQL supports a username and password for authentication. You will be prompted for a password during the installation.

Use the `mysql` command to create a new database on a fresh installation. After you create the database with the `CREATE DATABASE` command, you can select it for use with the use command. Once a database is selected, standard SQL commands can be used to create tables and insert data:

```
$> mysql -user=root -password=PASSWORD

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 44
Server version: 10.0.29-MariaDB-0+deb8u1 (Debian)

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE DATABASE test1;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> use test1;

```

The `quit` command or Ctrl-D will terminate a `mysql` interactive session.

# How to do it...

This recipe consists of three scripts: one to create a database and table, one to insert student data, and one to read and display data from the table.

Create the database and table script:

```
#!/bin/bash 
#Filename: create_db.sh 
#Description: Create MySQL database and table 

USER="user" 
PASS="user" 

mysql -u $USER -p$PASS <<EOF 2> /dev/null 
CREATE DATABASE students; 
EOF 

[ $? -eq 0 ] && echo Created DB || echo DB already exist  
mysql -u $USER -p$PASS students <<EOF 2> /dev/null 
CREATE TABLE students( 
id int, 
name varchar(100), 
mark int, 
dept varchar(4) 
); 
EOF 

[ $? -eq 0 ] && echo Created table students || \
    echo Table students already exist  

mysql -u $USER -p$PASS students <<EOF 
DELETE FROM students; 
EOF 

```

This script inserts data in the table:

```
#!/bin/bash 
#Filename: write_to_db.sh 
#Description: Read from CSV and write to MySQLdb 

USER="user" 
PASS="user" 

if [ $# -ne 1 ]; 
then 
  echo $0 DATAFILE 
  echo 
  exit 2 
fi 

data=$1 

while read line; 
do 

  oldIFS=$IFS 
  IFS=, 
  values=($line) 
  values[1]="\"`echo ${values[1]} | tr ' ' '#' `\"" 
  values[3]="\"`echo ${values[3]}`\"" 

  query=`echo ${values[@]} | tr ' #' ', ' ` 
  IFS=$oldIFS 

  mysql -u $USER -p$PASS students <<EOF 
INSERT INTO students VALUES($query); 
EOF 

done< $data 
echo Wrote data into DB 

```

The last script queries the database and generates a report:

```
#!/bin/bash 
#Filename: read_db.sh 
#Description: Read from the database 

USER="user" 
PASS="user" 

depts=`mysql -u $USER -p$PASS students <<EOF | tail -n +2 
SELECT DISTINCT dept FROM students; 
EOF` 

for d in $depts; 
do 

echo Department : $d 

result="`mysql -u $USER -p$PASS students <<EOF 
SET @i:=0; 
SELECT @i:=@i+1 as rank,name,mark FROM students WHERE dept="$d" ORDER BY mark DESC; 
EOF`" 

echo "$result" 
echo 

done 

```

The data for the input CSV file (`studentdata.csv`) will resemble this:

```
1,Navin M,98,CS
2,Kavya N,70,CS
3,Nawaz O,80,CS
4,Hari S,80,EC
5,Alex M,50,EC
6,Neenu J,70,EC
7,Bob A,30,EC
8,Anu M,90,AE
9,Sruthi,89,AE
10,Andrew,89,AE

```

Execute the scripts in the following sequence:

```
$ ./create_db.sh 
Created DB
Created table students

$ ./write_to_db.sh studentdat.csv
Wrote data into DB

$ ./read_db.sh 
Department : CS
rank  name  mark
1  Navin M  98
2  Nawaz O  80
3  Kavya N  70

Department : EC
rank  name  mark
1  Hari S  80
2  Neenu J 70
3  Alex M  50
4  Bob A   30

Department : AE
rank  name  mark
1  Anu M    90
2  Sruthi   89
3  Andrew   89

```

# How it works...

The first script, `create_db.sh`, creates a database called `students` and a table named `students` inside it. The `mysql` command is used for MySQL manipulations. The `mysql` command specifies the username with `-u` and the password with `-pPASSWORD`. The variables `USER` and `PASS` are used to store the username and password.

The other command argument for the `mysql` command is the database name. If a database name is specified as an argument to the `mysql` command, it will use that database; otherwise, we have to explicitly define the database to be used with the **use** `database_name` command.

The `mysql` command accepts the queries to be executed through standard input (`stdin`). A convenient way of supplying multiple lines through `stdin` is using the `<<EOF` method. The text that appears between `<<EOF` and `EOF` is passed to `mysql` as standard input.

The `CREATE DATABASE` and `CREATE TABLE` commands redirect `stderr` to `/dev/null` to prevent the display of error messages. The script checks the exit status for the `mysql` command stored in `$?` to determine whether a failure has occurred; it assumes that a failure occurs because a table or database already exists. If the database or table already exists, a message is displayed to notify the user; otherwise, the database and table are created.

The `write_to_db.sh` script accepts the filename of the student data CSV file. It reads each line of the CSV file in the `while` loop. On each iteration, a line from the CSV file is read and reformatted into a SQL command. The script stores the data from the comma-separated line in an array. Array assignment is done in this form: `array=(val1 val2 val3)`. Here, the space character is the **Internal****Field****Separator** (**IFS**). This data has comma-separated values. By changing the IFS to a comma, we can easily assign values to the array (`IFS=,`).

The data elements in the comma-separated line are `id`, `name`, `mark`, and `department`. The `id` and `mark` values are integers, while `name` and `dept` are strings that must be quoted.

The name could contain space characters that would conflict with the IFS. The script replaces the space in the name with a character (`#`) and restores it after formulating the query.

To quote the strings, the values in the array are reassigned with a prefix and suffixed with `\"`. The `tr` command substitutes each space in the name with `#`.

Finally, the query is formed by replacing the space character with a comma and replacing `#` with a space. Then, SQL's `INSERT` command is executed.

The third script, `read_db.sh`, generates a list of students for each department ordered by rank. The first query finds distinct names of departments. We use a `while` loop to iterate through each department and run the query to display student details in the order of highest marks obtained. `SET @i=0` is an SQL construct to set this: `i=0`. On each row, it is incremented and displayed as the rank of the student.

# User administration scripts

GNU/Linux is a multiuser operating system that allows many users to log in and perform activities at the same time. Administration tasks involving user management include setting the default shell for the user, adding a user to a group, disabling a shell account, adding new users, removing users, setting a password, setting an expiry date for a user account, and so on. This recipe demonstrates a user management tool to handle these tasks.

# How to do it...

This script performs common user management tasks:

```
#!/bin/bash 
#Filename: user_adm.sh 
#Description: A user administration tool 

function usage() 
{ 
  echo Usage: 
  echo Add a new user 
  echo $0 -adduser username password 
  echo 
  echo Remove an existing user 
  echo $0 -deluser username 
  echo 
  echo Set the default shell for the user 
  echo $0 -shell username SHELL_PATH 
  echo 
  echo Suspend a user account 
  echo $0 -disable username 
  echo 
  echo Enable a suspended user account 
  echo $0 -enable username 
  echo 
  echo Set expiry date for user account 
  echo $0 -expiry DATE  
  echo 
  echo Change password for user account 
  echo $0 -passwd username 
  echo 
  echo Create a new user group 
  echo $0 -newgroup groupname 
  echo 
  echo Remove an existing user group 
  echo $0 -delgroup groupname 
  echo 
  echo Add a user to a group 
  echo $0 -addgroup username groupname 
  echo 
  echo Show details about a user 
  echo $0 -details username 
  echo 
  echo Show usage 
  echo $0 -usage 
  echo 

  exit 
} 

if [ $UID -ne 0 ]; 
then 
  echo Run $0 as root. 
  exit 2 
fi 

case $1 in 

  -adduser) [ $# -ne 3 ] && usage ; useradd $2 -p $3 -m ;;  
  -deluser) [ $# -ne 2 ] && usage ; deluser $2 --remove-all-files;; 
  -shell)    [ $# -ne 3 ] && usage ; chsh $2 -s $3 ;; 
  -disable) [ $# -ne 2 ] && usage ; usermod -L $2 ;;  
  -enable) [ $# -ne 2 ] && usage ; usermod -U $2  ;; 
  -expiry) [ $# -ne 3 ] && usage ; chage $2 -E $3 ;; 
  -passwd) [ $# -ne 2 ] && usage ; passwd $2 ;; 
  -newgroup) [ $# -ne 2 ] && usage ; addgroup $2 ;; 
  -delgroup) [ $# -ne 2 ] && usage ; delgroup $2 ;; 
  -addgroup) [ $# -ne 3 ] && usage ; addgroup $2 $3 ;; 
  -details) [ $# -ne 2 ] && usage ; finger $2 ; chage -l $2 ;; 
  -usage) usage ;; 
  *) usage ;; 
esac 

```

A sample output resembles the following:

```
# ./user_adm.sh -details test
Login: test                 Name: 
Directory: /home/test                 Shell: /bin/sh
Last login Tue Dec 21 00:07 (IST) on pts/1 from localhost
No mail.
No Plan.
Last password change          : Dec 20, 2010
Password expires          : never
Password inactive         : never
Account expires             : Oct 10, 2010
Minimum number of days between password change    : 0
Maximum number of days between password change    : 99999
Number of days of warning before password expires  : 7

```

# How it works...

The `user_adm.sh` script performs several common user management tasks. The `usage()` text explains how to use the script when the user provides incorrect parameters or includes the `-usage` parameter. A case statement parses command arguments and executes the appropriate commands.

The valid command options for the `user_adm.sh` script are: `-adduser`, `-deluser`, `-shell`, `-disable`, `-enable`, `-expiry`, `-passwd`, `-newgroup`, `-delgroup`, `-addgroup`, `-details`, and `-usage`. When the `*)` case is matched, it means no option was recognized; hence, `usage()` is invoked.

Run this script as the root. It confirms the user ID (the root's user ID is `0`) before the arguments are examined.

When an argument is matched, the `[ $# -ne 3 ] &&` test usage checks the number of arguments. If the number of command arguments does not match the required number, the `usage()` function is invoked and the script exits.

These options are supported by the following scripts:

*   `-useradd`: The `useradd` command creates a new user:

```
        useradd USER -p PASSWORD -m

```

*   The `-m` option creates the home directory.
*   `-deluser`: The `deluser` command removes the user:

```
        deluser USER --remove-all-files

```

*   The `--remove-all-files` option removes all the files associated with the user, including the `home` directory.
*   `-shell`: The `chsh` command changes the default shell of the user:

```
        chsh USER -s SHELL

```

*   `-disable` and `-enable`: The `usermod` command manipulates several attributes related to user accounts. `usermod -L USER` locks the user account and `usermod -U USER` unlocks the user account.
*   `-expiry`: The `change` command manipulates user account expiry information:

```
        chage -E DATE

```

These options are supported:

*   `-m MIN_DAYS`: This sets the minimum number of days between password changes to `MIN_DAYS`

*   *   `-M MAX_DAYS`: This sets the maximum number of days during which a password is valid
    *   `-W WARN_DAYS`: This sets the number of days to provide a warning before a password change is required
*   `-passwd`: The `passwd` command changes a user's password:

```
        passwd USER

```

The command will prompt to enter a new password:

*   `-newgroup` and `-addgroup`: The `addgroup` command adds a new user group to the system:

```
        addgroup GROUP

```

If you include a username, it will add this user to a group:

```
        addgroup USER GROUP
 -delgroup

```

The `delgroup` command removes a user group:

```
        delgroup GROUP

```

*   `-details`: The `finger USER` command displays user information, including the home directory, last login time, default shell, and so on. The `chage -l` command displays the user account expiry information.

# Bulk image resizing and format conversion

All of us download photos from our phones and cameras. Before we e-mail an image or post it to the Web, we may need to resize it or perhaps change the format. We can use scripts to modify these image files in bulk. This recipe describes recipes for image management.

# Getting ready

The `convert` command from the **ImageMagick** suite contains tools for manipulating images. It supports many image formats and conversion options. Most GNU/Linux distributions don't include ImageMagick by default. You need to manually install the package. For more information, point your web browser at [www.imagemagick.org](http://www.imagemagick.org).

# How to do it...

The convert program will convert a file from one image format to another:

```
    $ convert INPUT_FILE OUTPUT_FILE

```

Here's an example of this:

```
    $ convert file1.jpg file1.png

```

We can resize an image by specifying the scale percentage or the width and height of the output image. To resize an image by specifying `WIDTH` or `HEIGHT`, use this:

```
    $ convert imageOrig.png -resize WIDTHxHEIGHT imageResized.png

```

Here's an example of this:

```
    $ convert photo.png -resize 1024x768 wallpaper.png

```

If either `WIDTH` or `HEIGHT` is missing, then whatever is missing will be automatically calculated to preserve the image aspect ratio:

```
    $ convert image.png -resize WIDTHx image.png

```

Here's an example of this:

```
    $ convert image.png -resize 1024x image.png

```

To resize the image by specifying the percentage scale factor, use this:

```
    $ convert image.png -resize "50%" image.png

```

This script will perform a set of operations on all the images in a directory:

```
#!/bin/bash 
#Filename: image_help.sh 
#Description: A script for image management 

if [ $# -ne 4 -a $# -ne 6 -a $# -ne 8 ]; 
then 
  echo Incorrect number of arguments 
  exit 2 
fi 

while [ $# -ne 0 ]; 
do 

  case $1 in 
  -source) shift; source_dir=$1 ; shift ;; 
  -scale) shift; scale=$1 ; shift ;; 
  -percent) shift; percent=$1 ; shift ;; 
  -dest) shift ; dest_dir=$1 ; shift ;; 
  -ext) shift ; ext=$1 ; shift ;; 
  *) echo Wrong parameters; exit 2 ;; 
  esac; 

done 

for img in `echo $source_dir/*` ; 
do 
  source_file=$img 
  if [[ -n $ext ]]; 
  then 
    dest_file=${img%.*}.$ext 
  else 
    dest_file=$img 
  fi 

  if [[ -n $dest_dir ]]; 
  then 
    dest_file=${dest_file##*/} 
    dest_file="$dest_dir/$dest_file" 
  fi 

  if [[ -n $scale ]]; 
  then 
    PARAM="-resize $scale" 
  elif [[ -n $percent ]];   then 
    PARAM="-resize $percent%"  
  fi 

  echo Processing file : $source_file 
  convert $source_file $PARAM $dest_file 

done 

```

The following example scales the images in the `sample_dir` directory to `20%`:

```
$ ./image_help.sh -source sample_dir -percent 20%
Processing file :sample/IMG_4455.JPG
Processing file :sample/IMG_4456.JPG
Processing file :sample/IMG_4457.JPG
Processing file :sample/IMG_4458.JPG

```

To scale images to a width of `1024`, use this:

```
$ ./image_help.sh -source sample_dir -scale 1024x

```

To scale and convert files into a specified destination directory, use this:

```
# newdir is the new destination directory
$ ./image_help.sh -source sample -scale 50% -ext png -dest newdir

```

# How it works...

The preceding `image_help.sh` script accepts these arguments:

*   `-source`: This specifies the source directory of the images.
*   `-dest`: This specifies the destination directory of the converted image files. If `-dest` is not specified, the destination directory will be the same as the source directory.
*   `-ext`: This specifies the target file format for conversions.
*   `-percent`: This specifies the percentage of scaling.
*   `-scale`: This specifies the scaled width and height.
*   Both the `-percent` and `-scale` parameters may not appear.
*   The script starts by checking the number of command arguments. Either four, six, or eight parameters are valid.

The command line is parsed with a `while` loop and the case statement and values are assigned to appropriate variables. `$#` is a special variable that contains the number of arguments. The `shift` command shifts the command arguments one position to the left. With this, every time the shifting happens, we can access the next command argument as `$1` rather than using `$1`, `$2`, `$3`, and so on.

The case statement is like a switch statement in the C programming language. When a case is matched, the corresponding statements are executed. Each match statement is terminated with `;;`. Once all the parameters are parsed into the variables `percent`, `scale`, `source_dir`, `ext`, and `dest_dir`, a `for` loop iterates through each file in the source directory and the file is converted.

Several tests are done within the `for` loop to fine-tune the conversion.

If the variable `ext` is defined (if `-ext` is given in the command argument), the extension of the destination file is changed from `source_file.extension` to `source_file.$ext`.

If the `-dest` parameter is provided, the destination file path is modified by replacing the directory in the source path with the destination directory.

If -scale or -percent are specified, the resize parameter (`-resize widthx` or `-resize perc%`) is added to the command.

After the parameters are evaluated, the `convert` command is executed with proper arguments.

# See also

*   The *Slicing filenames based on extensions* recipe in [Chapter 2](36986eeb-141a-496a-a6b1-4f78f612c14e.xhtml), *Have a Good Command*, explains how to extract a portion of the filename

# Taking screenshots from the terminal

As GUI applications proliferate, it becomes important to take screenshots, both to document your actions and to report unexpected results. Linux supports several tools for grabbing screenshots.

# Getting ready

This section will describe the **xwd** application and a tool from ImageMagick, which was used in the previous recipe. The xwd application is usually installed with the base GUI. You can install ImageMagick using your package manager.

# How to do it...

The xwd program extracts visual information from a window, converts it into X Window Dump format, and prints the data to `stdout`. This output can be redirected to a file, and the file can be converted into GIF, PNG, or JPEG format, as shown in the previous recipe.

When xwd is invoked, it changes your cursor to a crosshair. When you move this crosshair to an X Window and click on it, the window is grabbed:

```
    $ xwd >step1.xwd

```

ImageMagick's `import` command supports more options for taking screenshots:

To take a screenshot of the whole screen, use this:

```
    $ import -window root screenshot.png

```

You can manually select a region and take a screenshot of it using this:

```
    $ import screenshot.png

```

To take a screenshot of a specific window, use this:

```
    $ import -window window_id screenshot.png

```

The `xwininfo` command will return a window ID. Run the command and click on the window you want. Then, pass this `window_id` value to the `-window` option of `import`.

# Managing multiple terminals from one

SSH sessions, Konsoles, and xterms are heavyweight solutions for applications you want to run for a long time, but they perform a check infrequently (such as monitoring log files or disk usage).

The GNU screen utility creates multiple virtual screens in a terminal session. The tasks you start in a virtual screen continue to run when the screen is hidden.

# Getting ready

To achieve this, we will use a utility called **GNU screen**. If the screen is not installed on your distribution by default, install it using the package manager:

```
    apt-get install screen

```

# How to do it...

1.  Once the screen utility has created a new window, all the keystrokes go to the task running in that window, except Control-A (*Ctrl*-*A*), which marks the start of a screen command.
2.  **Creating screen windows**: To create a new screen, run the command screen from your shell. You will see a welcome message with information about the screen. Press Space or Return to return to the shell prompt. To create a new virtual terminal, press *Ctrl* + *A* and then *C* (these are case-sensitive) or type screen again.
3.  **Viewing a list of open windows**: While running the screen, pressing *Ctrl*+*A* followed by a quote (`"`) will list your terminal sessions.

4.  **Switching between windows**: The keystrokes *Ctrl* + *A* and *Ctrl* + *N* display the next window and *Ctrl* + *A* and *Ctrl* + *P* the previous window.
5.  **Attaching to and detaching screens**: The screen command supports saving and loading screen sessions, called detaching and attaching in screen terminology. To detach from the current screen session, press *Ctrl* + *A* and *Ctrl* + *D*. To attach to an existing screen when starting the screen, use:

```
        screen -r -d

```

6.  This tells the screen to attach the last screen session. If you have more than one detached session, the screen will output a list; then use:

```
        screen -r -d PID

```

Here, `PID` is the PID of the screen session you want to attach.