# 5

# Working with Processes, Daemons, and Signals

Linux is a multitasking operating system. Multiple programs or tasks can run in parallel, each with its own identity, scheduling, memory space, permissions, and system resources. **Processes** encapsulate the execution context of any such program. Understanding how processes work and communicate with each other is an important skill for any seasoned Linux system administrator and developer to have.

This chapter explores the basic concepts behind Linux processes. We’ll look at different types of processes, such as **foreground** and **background** processes, with special emphasis being placed on **daemons** as a particular type of background process. We’ll closely study the anatomy of a process and various inter-process communication mechanisms in Linux – **signals** in particular. Along the way, we’ll learn about some of the essential command-line utilities for managing processes and daemons and working with signals. We will also introduce you to **scripts** for the first time in this book, which are described in detail later in [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), *Linux Shell Scripting*. If you feel like you need more information when dealing with the scripts in this chapter, take a look at [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164) in advance.

In this chapter, we will cover the following topics:

*   Introducing processes
*   Working with processes
*   Working with daemons
*   Exploring inter-process communication

Important note

As we navigate through the content, we will occasionally reference signals *before* their formal introduction in the second half of this chapter. In Linux, signals are almost exclusively used in association with processes, hence our approach of becoming familiar with processes first. Yet, leaving the signals out from some of the process’ internals would do a disservice to understanding how processes work. Where signals are mentioned, we’ll point to the related section for further reference. We hope that this approach provides you with a better grasp of the overall picture and the inner workings of processes and daemons.

Now, before we start, let’s look at the essential requisites for our study.

# Technical requirements

Practice makes perfect. Running the commands and examples in this chapter by hand would go a long way toward you learning about processes. As with any chapter in this book, we recommend that you have a working Linux distribution installed on a VM or PC desktop platform. We’ll be using Ubuntu or Fedora, but most of the commands and examples would be similar on any other Linux platform.

# Introducing processes

A **process** represents the running instance of a program. In general, a program is a combination of instructions and data, compiled as an executable unit. When a program runs, a process is created. In other words, a process is simply a program in action. Processes execute specific tasks, and sometimes, they are also referred to as **jobs** (or **tasks**).

There are many ways to create or start a process. In Linux, every command starts a process. A command could be a user-initiated task in a Terminal session, a script, or a program (executable) that’s invoked manually or automatically.

Usually, the way a process is created and interacts with the system (or user) determines its process type. Let’s take a closer look at the different types of processes in Linux.

## Understanding process types

At a high level, there are two major types of processes in Linux:

*   **Foreground** (*interactive*)
*   **Background** (*non-interactive* or *automated*)

Interactive processes assume some kind of user interaction during the lifetime of the process. Non-interactive processes are unattended, which means that they are either automatically started (for example, on system boot) or are scheduled to run at a particular time and date via job schedulers (for example, using the `at` and `cron` command-line utilities).

Our approach to exploring process types mainly pivots around the preceding classification. There are various other views or taxonomies surrounding process definitions, but they could ultimately be reduced to either foreground or background processes.

For example, batch processes and daemons are essentially background processes. Batch processes are automated in the sense that they are not user-generated but invoked by a scheduled task instead. Daemons are background processes that are usually started during system boot and run indefinitely.

There’s also the concept of parent and child processes. A parent process may create other subordinate child processes.

We’ll elaborate on these types (and beyond) in the following sections. Let’s start with the pivotal ones – foreground and background processes.

### Foreground processes

`stdout` or `stderr`) or accept user input. The lifetime of a foreground process is tightly coupled to the Terminal session (parent process). If the user who launched the foreground process exits the Terminal while the process is still running, the process will be abruptly terminated (via a `SIGHUP` signal sent by the parent process; see *Signals* in the *Exploring inter-process communication* section for more details).

A simple example of a foreground process is invocating the system reference manual (`man`) for an arbitrary Linux command (for example, `ps`):

```
man ps
```

The `ps` command displays information about active processes. You will learn more about process management tools and command-line utilities in the *Working with* *processes* section.

Once a foreground process has been initiated, the user prompt is captured and controlled by the spawned process interface. The user can no longer interact with the initial command prompt until the interactive process relinquishes control to the Terminal session.

Let’s look at another example of a foreground process, this time invoking a long-lived task. The following command (one-liner) runs an infinite loop while displaying an arbitrary message every few seconds:

```
while true; do echo "Wait..."; sleep 5; done
```

So long as the command runs without being interrupted, the user won’t have an interactive prompt in the Terminal. Using *Ctrl* + *C* would stop (interrupt) the execution of the related foreground process and yield a responsive command prompt:

![](img/Figure_05_01_B19682.jpg)

Figure 5.1 – A long-lived foreground process

Important note

When you press *Ctrl* + *C* while a foreground process is running, a `SIGINT` signal is sent to the running process by the current (parent) Terminal session, and the foreground process is interrupted. For more information, see the *Signals* section.

If we want to maintain an interactive command prompt in the Terminal session while running a specific command or script, we should use a background process.

### Background processes

**Background processes** – also referred to as **non-interactive** or **automatic processes** – run independently of a Terminal session, without expecting any user interaction. A user may invoke multiple background processes within the same Terminal session without waiting on any of them to complete or exit.

Background processes are usually long-lived tasks that don’t require direct user supervision. The related process may still display its output in the Terminal console, but such background tasks typically write their results to different files instead (such as log files).

The simplest invocation of a background process appends an ampersand (`&`) to the end of the related command. Building on our previous example (in the *Foreground processes* section), the following command creates a background process that runs an infinite loop, echoing an arbitrary message every few seconds:

```
while true; do echo "Wait..."; sleep 10; done &
```

Note the ampersand (`&`) at the end of the command. By default, a background process would still direct the output (`stdout` and `stderr`) to the console when invoked with the ampersand (`&`), as shown previously. However, the Terminal session remains interactive. In the following figure, we are using the `echo` command while the previous process is still running:

![Figure 5.2 – Running a background process](img/Figure_05_02_B19682.jpg)

Figure 5.2 – Running a background process

As shown in the preceding screenshot, the background process is given a `983`. While the process is running, we can still control the Terminal session and run a different command, like so:

```
echo "Interactive prompt..."
```

Eventually, we can force the process to terminate with the `kill` command:

```
kill -9 983
```

The preceding command *kills* our background process (with PID `983`). The corresponding signal that’s sent by the parent Terminal session to terminate this process is `SIGKILL` (see the *Signals* section for more information) through the `-9` argument in our command.

Both foreground and background processes are typically under the direct control of a user. In other words, these processes are created or started manually as a result of a command or script invocation. There are some exceptions to this rule, particularly when it comes to batch processes, which are launched automatically via scheduled jobs.

There’s also a select category of background processes that are automatically started during system boot and terminated at shutdown without user supervision. These background processes are also known as daemons.

### Introducing daemons

A`root` or other) and runs with the related privileges.

Daemons usually serve client requests or communicate with other foreground or background processes. Here are some common examples of daemons, all of which are generally available on most Linux platforms:

*   `systemd`: The parent of all processes (formerly known as `init`)
*   `crond`: A job scheduler that runs tasks in the background
*   `ftpd`: An FTP server that handles client FTP requests
*   `httpd`: A web server (Apache) that handles client HTTP requests
*   `sshd`: A Secure Shell server that handles SSH client requests

Typically, system daemons in Linux are named with `d` at the end, denoting a daemon process. Daemons are controlled by shell scripts usually stored in the `/etc/init.d/` or `/lib/systemd/` system directory, depending on the Linux platform. Ubuntu, for example, stores daemon script files in `/etc/init.d/`, while Fedora stores them in `/lib/systemd/`. The location of these daemon files depends on the platform implementation of `init`, a system-wide service manager for all Linux processes.

The Linux init-style startup process generally invokes these shell scripts at system boot. But the same scripts can also be invoked via service control commands, usually run by privileged system users, to manage the lifetime of specific daemons. In other words, a privileged user or system administrator can *stop* or *start* a particular daemon through the command-line interface. Such commands would immediately return the user’s control to the Terminal while performing the related action in the background.

Let’s take a closer look at the `init` process.

### The init process

Throughout this chapter, we’ll refer to `init` as the *generic* system initialization engine and service manager on Linux platforms. Over the years, Linux distributions have evolved and gone through various `init` system implementations, such as `SysV`, `upstart`, `OpenRC`, `systemd`, and `runit`. There’s an ongoing debate in the Linux community about the supremacy or advantages of one over the other. For now, we will simply regard `init` as a system process, and we will briefly look at its relationship with other processes.

`init` (or `systemd`, and others) is essentially a system daemon, and it’s among the first process to start when Linux boots up. The related daemon process continues to run in the background until the system is shut down. `init` is the root (parent) process of all other processes in Linux, in the overall process hierarchy tree. In other words, it is a direct or indirect ancestor of all the processes in the system.

In Linux, the `pstree` command displays the whole process tree, and it shows the `init` process at its root – in our case, `systemd` (on Ubuntu or Fedora).

The output of the preceding command can be seen in the following screenshot:

![Figure 5.3 – init (systemd), the parent of all processes](img/Figure_05_03_B19682.jpg)

Figure 5.3 – init (systemd), the parent of all processes

The `pstree` command’s output illustrates a hierarchy tree representation of the processes, where some appear as parent processes while others appear as child processes. Let’s look at the parent and child process types and some of the dynamics between them.

### Parent and child processes

A `SIGHUP` signal that’s invoked by the parent process upon termination (for example, via the `nohup` command). See the *Signals* section for more information.

In Linux, all processes except the `init` process (with its variations) are children of a specific process. Terminating a child process won’t stop the related parent process from running. A good practice for terminating a parent process when the child is done processing is to exit from the parent process itself after the child process completes.

There are cases when processes run unattended, based on a specific schedule. Running a process without user interaction is known as batch processing. We’ll look at batch processes next.

### Batch processes

A `at` and `cron`. While `cron` is better suited to scheduled task management complexities, `at` is a more lightweight utility, better suited for one-off jobs. A detailed study of these commands is beyond the scope of this chapter. You may refer to the related system reference manuals for more information (`man at` and `man cron`).

We’ll conclude our study of process types with orphan and zombie processes.

### Orphan and zombie processes

When a child process is terminated, the related parent process is notified with a `SIGCHILD` signal. The parent can go on running other tasks or may choose to spawn another child process. However, there may be instances when the parent process is terminated before a related child process completes execution (or exits). In this case, the child process becomes an `init` process – the parent of all processes – automatically becomes the new parent of the orphan process.

`ps` command).

The main difference between the zombie and orphan processes is that a zombie process is dead (terminated), while an orphan process is still running.

As we differentiate between various process types and their behavior, a significant part of the related information is reflected in the composition or data structure of the process itself. In the next section, we’ll take a closer look at the makeup of a process, which is mostly echoed through the `ps` command-line utility – an ordinary yet very useful process explorer on Linux systems.

## The anatomy of a process

In this section, we will explore some of the common attributes of a Linux process through the lens of the `ps` and `top` command-line utilities. We hope that taking a practical approach based on these tools will help you gain a better understanding of process internals, at least from a Linux administrator’s perspective. Let’s start by taking a brief look at these commands. The `ps` command displays a current snapshot of the system processes. This command has the following syntax:

```
ps [OPTIONS]
```

The following command displays the processes owned by the current Terminal session:

```
ps
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.4 – Displaying processes owned by the current shell](img/Figure_05_04_B19682.jpg)

Figure 5.4 – Displaying processes owned by the current shell

Let’s look at each field in the top (header) row of the output and explain their meaning in the context of our relevant process – that is, the `bash` Terminal session:

*   `PID`: Each process in Linux has a `PID` value automatically assigned by the kernel when the process is created. The `PID` value is a positive integer and is always guaranteed to be unique.

In our case, the relevant process is `bash` (the current shell), with a PID of `171233`.

*   `TTY`: `TTY` attribute denotes the type of Terminal the process interacts with. In our example, the `bash` process representing the Terminal session has `pts/0` as its TTY type. `pts` stands for `/0` indicates the ordinal sequence of the related Terminal session. For example, an additional SSH session would have `pts/1`, and so on.
*   `TIME`: The `TIME` field represents the cumulative CPU utilization (or time) spent by the process (in `[DD-]hh:mm:ss` format). Why is it zero (`00:00:00`) for the `bash` process in our example? We may have run multiple commands in our Terminal session, yet the CPU utilization could still be zero. That’s because the CPU utilization measures (and accumulates) the time spent for each command, and not the parent Terminal session overall. If the commands complete within a fraction of a second, they will not amount to a significant CPU utilization being shown in the `TIME` field.
*   `CMD`: The `CMD` field stands for command and indicates the name or full path of the command (including the arguments) that created the process. For well-known system commands (for example, `bash`), `CMD` displays the command’s name, including its arguments.

The process attributes we’ve explored thus far represent a relatively simple view of Linux processes. There are situations when we may need more information. For example, the following command provides additional details about the processes running in the current Terminal session:

```
ps -l
```

The `-l` option parameter invokes the so-called *long format* for the `ps` output:

![Figure 5.5 – A more detailed view of processes](img/Figure_05_05_B19682.jpg)

Figure 5.5 – A more detailed view of processes

Here are just a few of the more relevant output fields of the `ps` command:

*   `F`: Process flags (for example, `0` – none, `1` – forked, and `4` – superuser privileges)
*   `S`: Process status code (for example, `R` – running, `S` – interruptible sleep, and so on)
*   `UID`: The username or owner of the process (the user ID)
*   `PID`: The process ID
*   `PPID`: The process ID of the parent process
*   `PRI`: The priority of the process (a higher number means lower priority)
*   `SZ`: The virtual memory usage

There are many more such attributes and exploring them all is beyond the scope of this book. For additional information, refer to the `ps` system reference manual (`man ps`).

The `ps` command examples we’ve used so far have only displayed the processes that are owned by the current Terminal session. This approach, we thought, would add less complexity to analyzing process attributes.

Besides `ps`, another command that’s used is `top`, and it provides a live (real-time) view of all the running processes in a system. Its syntax is as follows:

```
top [OPTIONS]
```

Many of the process output fields displayed by the `ps` command are also reflected in the `top` command, albeit some of them with slightly different notations. Let’s look at the `top` command and the meaning of the output fields that are displayed. The following command displays a real-time view of running processes:

```
top
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.6 – A real-time view of the current processes](img/Figure_05_06_B19682.jpg)

Figure 5.6 – A real-time view of the current processes

Here are some of the output fields, briefly explained:

*   `USER`: The username or owner of the process
*   `PR`: The priority of the process (a lower number means higher priority)
*   `NI`: The nice value of the process (a sort of dynamic/adaptive priority)
*   `VIRT`: The virtual memory size (in KB) – the total memory used by the process
*   `RES`: The resident memory size (in KB) – the physical (non-swapped) memory used by the process
*   `SHR`: The shared memory size (in KB) – a subset of the process memory shared with other processes
*   `S`: The process’ status (for example, `R` – running, `S` – interruptible sleep, `I` – idle, and so on)
*   `%CPU`: CPU usage (percentage)
*   `%MEM`: `RES` memory usage (percentage)
*   `COMMAND`: Command name or command line

Each of these fields (and many more) are explained in detail in the `top` system reference manual (`man top`).

Every day, Linux administration tasks frequently use process-related queries based on the preceding presented fields. The *Working with processes* section will explore some of the more common usages of the `ps` and `top` commands, and beyond.

An essential aspect of a process’s lifetime is the `ps` and `top` commands provide information about the status of the process via the `S` field. Let’s take a closer look at these states.

### Process states

During its lifetime, a process may change states according to circumstances. According to the `S` (status) field of the `ps` and `top` commands, a Linux process can have any of the following states:

*   `D`: Uninterruptible sleep
*   `I`: Idle
*   `R`: Running
*   `S`: Sleeping (interruptible sleep)
*   `T`: Stopped by a job control signal
*   `t`: Stopped by the debugger during a trace
*   `Z`: Zombie

At a high level, any of these states can be identified with the following process states:

*   `R` state) or is an idle process (the `I` state). In Linux, an idle process is a specific task that’s assigned to every processor (CPU) in the system and is scheduled to run only when there’s no other process running on the related CPU. The time that’s spent on idle tasks accounts for the idle time that’s reported by the `top` command.
*   `S` state) and uninterruptible sleep (the `D` state). Interruptible sleep can be disturbed by specific process signals, yielding further process execution. On the other hand, uninterruptible sleep is a state where the process is blocked in a system call (possibly waiting on some hardware conditions), and it cannot be interrupted.
*   `T` state) or a debugging signal (the `t` state).
*   `Z` state) – it’s terminated without being reaped by its parent. A zombie process is essentially a dead reference for an already terminated process in the system’s process table. This will be discussed in more detail in the *Orphan and zombie* *processes* section.

To conclude our analysis of process states, let’s look at the lifetime of a Linux process. Usually, a process starts with a running state (`R`) and terminates once its parent has reaped it from the zombie state (`Z`). The following diagram provides an abbreviated view of the process states and the possible transitions between them:

![Figure 5.7 – The lifetime of a Linux process](img/Figure_05_07_B19682.jpg)

Figure 5.7 – The lifetime of a Linux process

Now that we’ve introduced processes and provided you with a preliminary idea of their type and structure, we’re ready to interact with them. In the following sections, we will explore some standard command-line utilities for working with processes and daemons. Most of these tools operate with input and output data, which we covered in the *Anatomy of a process* section. We’ll look at working with processes next.

# Working with processes

This section serves as a practical guide to managing processes via resourceful command-line utilities that are used in everyday Linux administration tasks. Some of these tools were mentioned in previous sections (for example, `ps` and `top`) when we covered specific process internals. Here, we will summon most of the knowledge we’ve gathered so far and take it for a real-world spin by covering some hands-on examples.

Let’s start with the `ps` command – the Linux process explorer.

## Using the ps command

We described the `ps` command and its syntax in the *Anatomy of a process* section. The following command displays a selection of the current processes running in the system:

```
ps -e | head
```

The `-e` option (or `-A`) selects *all* the processes in the system. The `head` pipe invocation displays only the first few lines (10 by default):

![Figure 5.8 – Displaying the first few processes](img/Figure_05_08_B19682.jpg)

Figure 5.8 – Displaying the first few processes

The preceding information may not always be particularly useful. Perhaps we’d like to know more about each process, beyond just the `PID` or `CMD` fields in the `ps` command’s output. (We described some of these process attributes in the *Anatomy of a* *process* section).

The following command lists the processes owned by the current user more elaborately:

```
ps -fU $(whoami)
```

The `-f` option specifies the full-format listing, which displays more detailed information for each process. The `-U $(whoami)` option parameter specifies the current user (`packt`) as the real user (owner) of the processes we’d like to retrieve. In other words, we want to list all the processes we own:

![Figure 5.9 – Displaying the processes owned by the current user](img/Figure_05_09_B19682.jpg)

Figure 5.9 – Displaying the processes owned by the current user

There are situations when we may look for a specific process, either for monitoring purposes or to act upon them. Let’s take a previous example, where we showcased a long-lived process and wrapped the related command into a simple script. The command is a simple `while` loop that runs indefinitely:

```
while true; do x=1; done
```

Using an editor of our preference (for example, `nano`), we can create a script file (for example, `test.sh`) with the following content:

![Figure 5.10 – A simple test script running indefinitely](img/Figure_05_10_B19682.jpg)

Figure 5.10 – A simple test script running indefinitely

We can make the test script executable and run it as a background process:

```
chmod +x test.sh
./test.sh &
```

Note the ampersand (`&`) at the end of the command, which invokes the background process:

![Figure 5.11 – Running a script as a background process](img/Figure_05_11_B19682.jpg)

Figure 5.11 – Running a script as a background process

The background process running our script has a process ID (`PID`) of `1094`. Suppose we want to find our process by its name (`test.sh`). For this, we can use the `ps` command with a `grep` pipe:

```
ps -ef | grep test.sh
```

The output of the preceding command can be seen in the following screenshot:

![ Figure 5.12 – Finding a process by name using the ps command](img/Figure_05_12_B19682.jpg)

Figure 5.12 – Finding a process by name using the ps command

The preceding output shows that our process has a `PID` value of `1094` and a `CMD` value of `/bin/bash ./test.sh`. The `CMD` field contains the full command invocation of our script, including the command-line parameters.

We should note that the first line of the `test.sh` script contains `#!/bin/bash`, which prompts the OS to invoke `bash` for the script’s execution. This line is also known as the `CMD` field, the command in our case is `/bin/bash` (according to the shebang invocation), and the related command-line parameter is the `test.sh` script. In other words, `bash` executes the `test.sh` script.

The output of the preceding `ps` command also includes our `ps | grep` command’s invocation, which is somewhat irrelevant. A refined version of the same command is as follows:

```
ps -ef | grep test.sh | grep -v grep
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.13 – Finding a process by name using the ps command (refined)](img/Figure_05_13_B19682.jpg)

Figure 5.13 – Finding a process by name using the ps command (refined)

The `grep -v grep` pipe filters out the unwanted `grep` invocation from the `ps` command’s results.

If we want to find a process based on a process ID (`PID`), we can invoke the `ps` command with the `-p|--pid` option parameter. For example, the following command displays detailed information about our process with `PID` set to `1094` (running the `test.sh` script):

![Figure 5.14 – Finding a process by PID using the ps command](img/Figure_05_14_B19682.jpg)

Figure 5.14 – Finding a process by PID using the ps command

The `-f` option displays the detailed (*long-format*) process information.

There are numerous other use cases for the `ps` command, and exploring them all is well beyond the scope of this book. The invocations we’ve enumerated here should provide a basic exploratory guideline for you. For more information, please refer to the `ps` system reference manual (`man ps`).

## Using the pstree command

`pstree` shows the running processes in a hierarchical, tree-like view. In some respects, `pstree` acts as a visualizer of the `ps` command. The root of the `pstree` command’s output is either the `init` process or the process with the `PID` value specified in the command. The syntax of the `pstree` command is as follows:

```
pstree [OPTIONS] [PID] [USER]
```

The following command displays the process tree of our current Terminal session:

```
pstree $(echo $$)
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.15 – The process tree of the current Terminal session](img/Figure_05_15_B19682.jpg)

Figure 5.15 – The process tree of the current Terminal session

In the preceding command, `echo $$` provides the `PID` value of the current Terminal session. `$$` is a Bash built-in variable that contains the `PID` value of the shell that is running. The `PID` value is wrapped as the argument for the `pstree` command. To show the related PIDs, we can invoke the `pstree` command with the `-``p|--show-pids` option:

```
pstree -p $(echo $$)
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.16 – The process tree (along with its PIDs) of the current Terminal session](img/Figure_05_16_B19682.jpg)

Figure 5.16 – The process tree (along with its PIDs) of the current Terminal session

The following command shows the processes owned by the current user:

```
pstree $(whoami)
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.17 – The process tree owned by the current user](img/Figure_05_17_B19682.jpg)

Figure 5.17 – The process tree owned by the current user

For more information about the `pstree` command, please refer to the related system reference manual (`man pstree`).

## Using the top command

When it comes to monitoring processes in real time, the `top` utility is among the most common tool to be used by Linux administrators. The related command-line syntax is as follows:

```
top [OPTIONS]
```

The following command displays all the processes currently running in the system, along with real-time updates (on memory, CPU usage, and so on):

```
top
```

Pressing *Q* will exit the `top` command. By default, the `top` command sorts the output by CPU usage (shown in the `%``CPU` field/column).

We can also choose to sort the output of the `top` command by a different field. While `top` is running, press *Shift* + *F* (`F`) to invoke interactive mode.

Using the arrow keys, we can select the desired field to sort by (for example, `%MEM`), then press *S* to set the new field, followed by *Q* to exit interactive mode. The alternative to interactive mode sorting is invoking the `-o` option parameter of the `top` command, which specifies the sorting field.

For example, the following command lists the top 10 processes, sorted by CPU usage:

```
top -b -o %CPU | head -n 17
```

Similarly, the following command lists the top 10 processes, sorted by CPU and memory usage:

```
top -b -o +%MEM | head -n 17
```

The `-b` option parameter specifies the batch mode operation (instead of the default interactive mode). The `-o +%MEM` option parameter indicates the additional (`+`) sorting field (`%MEM`) in tandem with the default `%CPU` field. The `head -n 17` pipe selects the first 17 lines of the output, accounting for the seven-line header of the `top` command:

![Figure 5.18 – The top 10 processes sorted by CPU and memory usage](img/Figure_05_18_B19682.jpg)

Figure 5.18 – The top 10 processes sorted by CPU and memory usage

The following command lists the top five processes by CPU usage, owned by the current user (`packt`):

```
top -u $(whoami) -b -o %CPU | head -n 12
```

The `-u $(whoami)` option parameter specifies the current user for the `top` command.

With the `top` command, we can also monitor specific processes using the `-p` PID option parameter. For example, the following command monitors our test process (with PID `243436`):

```
top -p 1094
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.19 – Monitoring a specific PID with the top command](img/Figure_05_19_B19682.jpg)

Figure 5.19 – Monitoring a specific PID with the top command

We may choose to *kill* the process by pressing *K* while using the `top` command. We’ll get prompted for this by the PID of the process we want to terminate:

![Figure 5.20 – Killing a process with the top command](img/Figure_05_20_B19682.jpg)

Figure 5.20 – Killing a process with the top command

The `top` utility can be used in many creative ways. We hope that the examples we’ve provided in this section have inspired you to explore further use cases based on specific needs. For more information, please refer to the system reference manual for the `top` command (`man top`).

## Using the kill and killall commands

We use the `kill` command to terminate processes. The command’s syntax is as follows:

```
kill [OPTIONS] [ -s SIGNAL | -SIGNAL ] PID [...]
```

The `kill` command sends a *signal* to a process, attempting to stop its execution. When no signal is specified, `SIGTERM` (`15`) is sent. A signal can either be specified by the signal’s name without the `SIG` prefix (for example, `KILL` for `SIGKILL`) or by value (for example, `9` for `SIGKILL`).

The `kill -l` and `kill -L` commands provide a full list of signals that can be used in Linux:

![Figure 5.21 – The Linux signals](img/Figure_05_21_B19682.jpg)

Figure 5.21 – The Linux signals

Each signal has a numeric value, as shown in the preceding output. For example, `SIGKILL` equals `9`. The following command will kill our test process (with PID `243436`):

```
kill -9 1094
```

The following command will also do the same as the preceding command:

```
kill -KILL 1094
```

In some scenarios, we may want to kill multiple processes in one go. The `killall` command comes to the rescue here. The syntax for the `killall` command is as follows:

```
killall [OPTIONS] [ -s SIGNAL | -SIGNAL ] NAME...
```

`killall` sends a signal to all the processes running any of the commands specified. When no signal is specified, `SIGTERM` (`15`) is sent. A signal can either be specified by the signal name without the `SIG` prefix (for example, `TERM` for `SIGTERM`) or by value (for example, `15` for `SIGTERM`).

For example, the following command terminates all the processes running the `test.sh` script:

```
killall -e -TERM test.sh
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.22 – Terminating multiple processes with killall](img/Figure_05_22_B19682.jpg)

Figure 5.22 – Terminating multiple processes with killall

Killing a process will usually remove the related reference from the system process table. The terminated process won’t show up anymore in the output of `ps`, `top`, or similar commands.

For more information about the `kill` and `killall` commands, please refer to the related system reference manuals (`man kill` and `man killall`).

## Using the pgrep and pkill commands

`pgrep` and `pkill` are pattern-based lookup commands for exploring and terminating running processes. They have the following syntax:

```
pgrep [OPTIONS] PATTERN
pkill [OPTIONS] PATTERN
```

`pgrep` iterates through the current processes and lists the PIDs that match the selection pattern or criteria. Similarly, `pkill` terminates the processes that match the selection criteria.

The following command looks for our test process (`test.sh`) and displays the `PID` value if the related process is found. Start the process again before using the following command as we killed it in the previous section. This will lead to a different `PID` value:

```
pgrep -f test.sh
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.23 – Looking for a PID based on name using pgrep](img/Figure_05_23_B19682.jpg)

Figure 5.23 – Looking for a PID based on name using pgrep

The `-f|--full` option enforces a full name match of the process we’re looking for. We may use `pgrep` in tandem with the `ps` command to get more detailed information about the process, like so:

```
pgrep -f test.sh | xargs ps -fp
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.24 – Chaining pgrep and ps for more information](img/Figure_05_24_B19682.jpg)

Figure 5.24 – Chaining pgrep and ps for more information

In the preceding one-liner, we piped the output of the `pgrep` command (with PID `243436`) to the `ps` command, which has been invoked with the `-f` (long-format) and `-p|--pid` options. The `-p` option parameter gets the piped PID value.

The `xargs` command takes the input from the `pgrep` command and converts it into an argument for the `ps` command. Thus, when piping from `pgrep` to `ps`, the output of the first command was automatically converted as the argument for the second command. By default, `xargs` reads the standard input.

To terminate our `test.sh` process, we simply invoke the `pkill` command, as follows:

```
pkill -f test.sh
```

The preceding command will *silently* kill the related process, based on the full name lookup enforced by the -`f|--full` option. To get some feedback from the action of the `pkill` command, we need to invoke the `-e|--echo` option, like so:

```
pkill -ef test.sh
```

The output of the preceding command can be seen in the following screenshot:

![Figure 5.25 – Killing a process by name using pkill](img/Figure_05_25_B19682.jpg)

Figure 5.25 – Killing a process by name using pkill

For more information, please refer to the `pgrep` and `pkill` system reference manuals (`man pgrep` and `man pkill`).

This section covered some command-line utilities that are frequently used in everyday Linux administration tasks involving processes. Keep in mind that in Linux, most of the time, there are many ways to accomplish a specific task. We hope that the examples in this section will help you come up with creative methods and techniques for working with processes.

Next, we’ll look at some common ways of interacting with daemons.

# Working with daemons

As noted in the introductory sections, daemons are a special breed of background process. Consequently, the vast majority of methods and techniques for working with processes also apply to daemons. However, there are specific commands that strictly operate on daemons when it comes to managing (or controlling) the lifetime of the related processes.

As noted in the *Introducing daemons* section, daemon processes are controlled by shell scripts, usually stored in the `/etc/init.d/` or `/lib/systemd/` system directories, depending on the Linux platform. On legacy Linux systems (for example, RHEL 6) and Ubuntu (even in the latest distros), the daemon script files are stored in `/etc/init.d/`. On RHEL 7/Ubuntu 18.04 and newer platforms, they are typically stored in `/lib/systemd/`. Feel free to do a listing of those two directories to see the contents.

The location of the daemon files and the daemon command-line utilities largely depends on the `init` initialization system and service manager. In *The init process* section, we briefly mentioned a variety of `init` systems across Linux distributions. To illustrate the use of daemon control commands, we will explore the `init` system called `systemd`, which is extensively used across various Linux platforms.

## Working with systemd daemons

The `init` system’s essential requirement is to initialize and orchestrate the launch and startup dependencies of various processes when the Linux kernel is booted. These processes are also known as `init` engine also controls the services and daemons while the system is running.

Over the last few years, most Linux platforms have transitioned to `systemd` as their default `init` engine. Due to its extensive adoption, being familiar with `systemd` and its related command-line tools is of paramount importance. With that in mind, this section’s primary focus is on `systemctl` – the central command-line utility for managing `systemd` daemons.

The syntax of the `systemctl` command is as follows:

```
systemctl [OPTIONS] [COMMAND] [UNITS...]
```

The actions that are invoked by the `systemctl` command are directed at units, which are system resources that are managed by `systemd`. Several unit types are defined in `systemd` (for example, service, mount, socket, and so on). Each of these units has a corresponding file. These file types are inferred from the suffix of the related filename; for example, `httpd.service` is the service unit file of the Apache web service (daemon). For a comprehensive list of `systemd` units and detailed descriptions of them, please refer to the `systemd.unit` system reference manual (`man systemd.unit`).

The following command enables a daemon (for example, `httpd`, the web server) to start at boot:

```
sudo systemctl enable httpd
```

Typically, invoking `systemctl` commands requires superuser privileges. We should note that `systemctl` does not require the `.service` suffix when we’re targeting service units. The following invocation is also acceptable:

```
sudo systemctl enable httpd.service
```

The command to disable the `httpd` service from starting at boot is as follows:

```
sudo systemctl disable httpd
```

To query the status of the `httpd` service, we can run the following command:

```
sudo systemctl status httpd
```

Alternatively, we can check the status of the `httpd` service with the following command:

```
sudo systemctl is-active httpd
```

The following commands stop or start the `httpd` service:

```
sudo systemctl stop httpd
sudo systemctl start httpd
```

For more information on `systemctl`, please refer to the related system reference manual (`man systemctl`). For more information about `systemd` internals, please refer to the corresponding reference manual (`man systemd`).

Working with processes and daemons is a constant theme of everyday Linux administration tasks. Mastering the related command-line utilities is an essential skill for any seasoned user. Yet, a running process or daemon should also be considered in relationships with other processes or daemons running either locally or on remote systems. The way processes communicate with each other could be a slight mystery to some. We will address this in the next section, in which we will explain how inter-process communication works.

# Explaining inter-process communication

**Inter-process communication** (**IPC**) is a way of interacting between processes using a shared mechanism or interface. In this section, we will take a short theoretical approach to exploring various communication mechanisms between processes. For more details on this matter and some of the mechanisms used, head to [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), *Linux* *Shell Scripting*.

Linux processes can typically share data and synchronize their actions via the following interfaces:

*   **Shared storage** (**files**): In its simplest form, the shared storage of an IPC mechanism can be a simple file that’s been saved to disk. The producer then writes to a file while the consumer reads from the same file. In this simple use case, the obvious challenge is the integrity of the read/write operations due to possible race conditions between the underlying operations. To avoid race conditions, the file must be locked during write operations to prevent overlapping I/O with another read or write action. To keep things simple, we’re not going to resolve this problem in our naive examples, but we thought it’s worth calling it out.
*   `/dev/shm` temporary file storage system, which uses the system’s RAM as its backing store (that is, RAM disk).

With `/dev/shm` being used as shared memory, we can reuse our producer-consumer model from the example in the previous point on *Shared storage*, where we simply point the storage file to `/dev/shm/storage`.

The shared memory and shared storage IPC models may not perform well with large amounts of data, especially massive data streams. The alternative would be to use IPC channels, which can be enabled through the pipe, message queue, or socket communication layers.

*   **Named and unnamed pipes**: **Unnamed** or **anonymous pipes**, also known as **regular pipes**, feed the output of a process to the input of another one. Using our producer-consumer model, the simplest way to illustrate an unnamed pipe as an IPC mechanism between the two processes would be to do the following:

    ```
    producer.sh | consumer.sh
    ```

The key element of the preceding code is the pipe (`|`) symbol. The left-hand side of the pipe produces an output that’s fed directly to the right-hand side of the pipe for consumption.

**Named pipes**, also known as **First-In, First-Outs** (**FIFOs**), are similar to traditional (unnamed) pipes but substantially different in terms of their semantics. An unnamed pipe only persists for as long as the related process is running. However, a named pipe has backing storage and will last as long as the system is up, regardless of the running status of the processes attached to the related IPC channel. Typically, a named pipe acts as a file, and it can be deleted when it’s no longer being used.

*   **Message queues**: A message queue is an asynchronous communication mechanism that’s typically used in a distributed system architecture. Messages are written and stored in a queue until they are processed and eventually deleted. A message is written (published) by a producer and is processed only once, typically by a single consumer. At a very high level, a message has a sequence, a payload, and a type. Message queues can regulate the retrieval (order) of messages (for example, based on priority or type):

![](img/Figure_05_26_B19682.jpg)

Figure 5.26 – Message queue (simplified view)

A detailed analysis of message queues or a mock implementation thereof is far from trivial, and it’s beyond this chapter’s scope. There are numerous open source message queue implementations available for most Linux platforms (RabbitMQ, ActiveMQ, ZeroMQ, MQTT, and so on).

IPC mechanisms based on message queues and pipes are unidirectional. One process writes the data; another one reads it. There are bidirectional implementations of named pipes, but the complexities involved would negatively impact the underlying communication layer. For bidirectional communication, you can think of using socket-based IPC channels (detailed in [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), *Linux* *Shell Scripting*).

*   **Sockets**: There are two types of IPC socket-based facilities:
    *   **IPC sockets**: Also known as Unix domain sockets, IPC sockets use a local file as a socket address and enable bidirectional communication between processes on the same host.
    *   **Network sockets**: **Transport Control Protocol** (**TCP**) and **User Datagram Protocol** (**UDP**) sockets. They extend the IPC data connectivity layer beyond the local machine via TCP/UDP networking.

Apart from the obvious implementation differences, the IPC socket’s and network socket’s data communication channels behave the same.

Both sockets are configured as streams, support bidirectional communication, and emulate a client/server pattern. The socket’s communication channel is active until it’s closed on either end, thereby breaking the IPC connection.

*   **Signals**: In Linux, a signal is a one-way asynchronous notification mechanism that’s used in response to a specific condition. A signal can act in any of the following directions:
    *   From the Linux kernel to an arbitrary process
    *   From process to process
    *   From a process to itself

We mentioned at the beginning of this section that signals are yet another IPC mechanism. Indeed, they are a somewhat limited form of IPC in the sense that through signals, processes can coordinate synchronization with each other. But signals don’t carry any data payloads. They simply notify processes about events, and processes may choose to take specific actions in response to these events.

In the next section, we will detail working with signals in Linux.

## Working with signals

Signals typically alert a Linux process about a specific event, such as a segmentation fault (`SIGSEGV`) that’s raised by the kernel or execution being interrupted (`SIGINT`) by the user pressing *Ctrl* + *C*. In Linux, processes are controlled via signals. The Linux kernel defines a few dozen signals. Each signal has a corresponding non-zero positive integer value.

The following command lists all the signals that have been registered in a Linux system:

```
kill -l
```

The output of the preceding command can be seen back in *Figure 5**.21*. From the output, `SIGHUP`, for example, has a signal value of `1`, and it’s invoked by a Terminal session to all its child processes when it exits. `SIGKILL` has a signal value of `9` and is most commonly used for terminating processes. Processes can typically control how signals are handled, except for `SIGKILL` (`9`) and `SIGSTOP` (`19`), which always end or stop a process, respectively.

Processes handle signals in either of the following fashions:

*   Perform the default action implied by the signal; for example, stop, terminate, core-dump a process, or do nothing.
*   Perform a custom action (except for `SIGKILL` and `SIGSTOP`). In this case, the process catches the signal and handles it in a specific way.

When a program implements a custom handler for a signal, it usually defines a signal handler function that alters the execution of the process, as follows:

*   When the signal is received, the process’ execution is interrupted at the current instruction
*   The process’ execution immediately jumps to the signal-handler function
*   The signal handler function runs
*   When the signal handler function exits, the process resumes execution, starting from the previously interrupted instruction

Here’s some brief terminology related to signals:

*   A signal is raised by the process that generates it
*   A signal is caught by the process that handles it
*   A signal is ignored if the process has a corresponding **no-operation** or **no-op** (**NOOP**) handler
*   A signal is handled if the process implements a specific action when the signal is caught

Out of all the signals, `SIGKILL` and `SIGSTOP` are the only ones that cannot be caught or ignored.

Let’s explore a few use cases for handling signals:

*   When the kernel raises a `SIGKILL`, `SIGFPE` (floating-point exception), `SIGSEGV` (segmentation fault), `SIGTERM`, or similar signals, typically, the process that receives the signal immediately terminates execution and may generate a core dump – the image of the process that’s used for debugging purposes.
*   When a user types *Ctrl* + *C* – otherwise known as an `SIGINT` signal is sent to the process. The process will terminate unless the underlying program implements a special handler for `SIGINT`.
*   Using the `kill` command, we can send a signal to any process based on its PID. The following command sends a `SIGHUP` signal to a Terminal session with a PID of `3741`:

    ```
    kill -HUP 3741
    ```

In the preceding command, we can either specify the signal value (for example, `1` for `SIGHUP`) or just the signal name without the `SIG` prefix (for example, `HUP` for `SIGHUP`).

With `killall`, we can signal that multiple processes are running a specific command (for example, `test.sh`). The following command terminates all processes running the `test.sh` script and outputs the result to the console (via the `-``e` option):

```
killall -e -TERM test.sh
```

The output of this command can be seen in *Figure 5**.22*.

Linux processes and signals are a vast domain. The information we’ve provided here is far from a comprehensive guide on the topic. We hope that this short spin and hands-on approach to presenting some common use cases has inspired you to take on and possibly master more challenging issues.

# Summary

A detailed study of Linux processes and daemons could be a major undertaking. Where worthy volumes on the topic have admirably succeeded, a relatively brief chapter may pale in comparison. Yet in this chapter, we tried to put on a real-world, down-to-earth, practical coat on everything we’ve considered to make up for our possible shortcomings in the abstract or scholarly realm.

At this point, we hope you are comfortable working with processes and daemons. The skills you’ve gathered so far should include a relatively good grasp of process types and internals, with a reasonable understanding of process attributes and states. Special attention has been paid to inter-process communication mechanisms, and signals in particular. For each of these topics, we will take a more detailed approach in [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164). For now, we consider the information we’ve provided to be sufficient for understanding how inter-process communication works.

The next chapter will take our journey further into working with Linux disks and filesystems. We’ll explore the Linux storage, disk partitioning, and **Logical Volume Management** (**LVM**) concepts. Rest assured that everything we’ve learned so far will be immediately put to good use in the chapters that follow.

# Questions

If you managed to skim through some parts of this chapter, you might want to recap a few essential details about Linux processes and daemons:

1.  Think of a few process types. How would they compare to each other?
2.  Think of the anatomy of a process. Can you come up with a few essential process attributes (or fields in the `ps` command-line output) that you may look for when inspecting processes?

**Hint**: What would be relevant for you, except CPU, RAM, or disk usage, for example?

1.  Can you think of a few process states and some of the dynamics or possible transitions between them?
2.  If you are looking for a process that takes up most of the CPU on your system, how would you proceed?
3.  Can you write a simple script and make it a long-lived background process?

**Hint**: Take a peek at [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), where we will teach you how to create and use shell scripts.

1.  Enumerate at least four process signals that you can think of. When or how would those signals be invoked?

`kill -l` command. For more information, read the manual.

1.  Think of a couple of IPC mechanisms. Try to come up with some pros and cons for them.

**Hint**: The information in [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164) could help you.

# Further reading

For more information about what was covered in this chapter, you can refer to the following Packt titles:

*   *Linux Administration Best Practices*, by Scott Alan Miller
*   *Linux Service Management Made Easy with systemd*, by Donald A. Tevault

# Part 2:Advanced Linux Administration

In this second part, you will learn about advanced Linux system administration tasks, including working with disks and configuring networking, hardening Linux security, and system-specific troubleshooting and diagnostics.

This part has the following chapters:

*   [*Chapter 6*](B19682_06.xhtml#_idTextAnchor124), *Working with Disks and Filesystems*
*   [*Chapter 7*](B19682_07.xhtml#_idTextAnchor139), *Networking with Linux*
*   [*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), *Linux Shell Scripting*
*   [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing Linux*
*   [*Chapter 10*](B19682_10.xhtml#_idTextAnchor212), *Disaster Recovery, Diagnostics, and Troubleshooting*