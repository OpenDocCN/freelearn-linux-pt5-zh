# Put On the Monitors Cap

In this chapter, we will cover the following recipes:

*   Monitoring disk usage
*   Calculating the execution time for a command
*   Collecting information about logged in users, boot logs, and boot failures
*   Listing the top ten CPU– consuming processes in an hour
*   Monitoring command outputs with watch
*   Logging access to files and directories
*   Logging with syslog
*   Managing log files with `logrotate`
*   Monitoring user logins to find intruders
*   Monitoring remote disk usage health
*   Determining active user hours on a system
*   Measuring and optimizing power usage
*   Monitoring disk activity
*   Checking disks and filesystems for errors
*   Examining disk health
*   Getting disk statistics

# Introduction

A computing system is a set of hardware and the software components that control it. The software includes the operating system kernel which allocates resources and many modules that perform individual tasks, ranging from reading disk data to serving web pages.

An administrator needs to monitor these modules and applications to confirm that they are working correctly and to understand whether resources need to be reallocated (moving a user partition to a larger disk, providing a faster network, and so on).

Linux provides both interactive programs for examining the system's current performance and modules for logging performance over time.

This chapter describes the commands that monitor system activity and discusses logging techniques.

# Monitoring disk usage

Disk space is always a limited resource. We monitor disk usage to know when it's running low, then search for large files or folders to delete, move, or compress. This recipe illustrates disk monitoring commands.

# Getting ready

The `du` (disk usage) and `df` (disk free) commands report disk usage. These tools report what files and folders are consuming disk space and how much space is available.

# How to do it...

To find the disk space used by a file (or files), use the following command:

```
    $ du  FILENAME1 FILENAME2 ..

```

Consider this example:

```
    $ du file.txt

```

To obtain the disk usage for all files inside a directory, along with the individual disk usage for each file shown in each line, use this command:

```
    $ du -a DIRECTORY

```

The `-a` option outputs results for all files in the specified directory or directories recursively.

Running `du DIRECTORY` will output a similar result, but it will show only the size consumed by subdirectories. However, this does not show the disk usage for each of the files. For printing the disk usage by files, `-a` is mandatory.

Consider this example:

```
    $  du -a test
 4  test/output.txt
 4  test/process_log.sh
 4  test/pcpu.sh
 16  test

```

The `du` command can be used on a directory:

```
    $ du test
 16  test

```

# There's more...

The `du` command includes options to define how the data is reported.

# Displaying disk usage in KB, MB, or blocks

By default, the disk usage command displays the total bytes used by a file. A more human-readable format is expressed in units such as KB, MB, or GB. The `-h` option displays the results in a human-readable format:

```
    du -h FILENAME

```

Consider this example:

```
    $ du -h test/pcpu.sh
 4.0K  test/pcpu.sh
 # Multiple file arguments are accepted

```

Alternatively, use it like this:

```
    # du -h DIRECTORY
 $ du -h hack/
 16K  hack/

```

# Displaying the grand total sum of disk usage

The `-c` option will calculate the total size used by files or directories, as well as display individual file sizes:

```
    $ du -c FILENAME1 FILENAME2..
 du -c process_log.sh pcpu.sh
 4  process_log.sh
 4  pcpu.sh
 8  total

```

Alternatively, use it like one of these:

```
    $ du  -c DIRECTORY
 $ du -c test/
 16  test/
 16  total

```

Or:

```
 $ du -c *.txt
 # Wildcards

```

The `-c` option can be used with options such as `-a` and `-h` to produce the usual output, with an extra line containing the total size.

The `-s` option (summarize), will print the grand total as the output. The `-h` flag can be used with it to print in a human-readable format:

```
    $ du -sh /usr/bin
 256M   /usr/bin

```

# Printing sizes in specified units

The `-b`, `-k`, and `-m` options will force `du` to print the disk usage in specified units. Note that these cannot be used with the `-h` option:

*   Print the size in bytes (by default):

```
        $ du -b FILE(s)

```

*   Print the size in kilobytes:

```
        $ du -k FILE(s)

```

*   Print the size in megabytes:

```
        $ du -m FILE(s)

```

*   Print the size in the given `BLOCK` size specified:

```
        $ du -B BLOCK_SIZE FILE(s)

```

Here, `BLOCK_SIZE` is specified in bytes.

Note that the file size returned is not intuitively obvious. With the `-b` option, `du` reports the exact number of bytes in the file. With other options, `du` reports the amount of disk space used by the file. Since disk space is allocated in fixed-size chunks (commonly 4 K), the space used by a 400-byte file will be a single block (4 K):

```
        $ du pcpu.sh
 4  pcpu.sh
 $ du -b pcpu.sh
 439 pcpu.sh
 $ du -k pcpu.sh
 4  pcpu.sh
 $ du -m pcpu.sh
 1  pcpu.sh
 $ du -B 4 pcpu.sh
 1024  pcpu.sh

```

# Excluding files from the disk usage calculation

The `--exclude` and `-exclude-from` options cause `du` to exclude files from the disk usage calculation.

*   The `-exclude` option can be used with wildcards or a single filename:

```
        $ du --exclude "WILDCARD" DIRECTORY

```

Consider this example:

```
        # Excludes all .txt files from calculation
 $ du --exclude "*.txt" *
 # Exclude temp.txt from calculation
 $ du --exclude "temp.txt" *

```

*   The `--exclude` option will exclude one file or files that match a pattern. The `-exclude-from` option allows more files or patterns to be excluded. Each filename or pattern must be on a single line.

```
        $ ls *.txt >EXCLUDE.txt
        $ ls *.odt >>EXCLUDE.txt
 # EXCLUDE.txt contains a list of all .txt and .odt files. 
 $ du --exclude-from EXCLUDE.txt DIRECTORY

```

The `-max-depth` option restricts how many subdirectories du will examine. A depth of `1` calculates disk usage in the current directory. A depth of `2` calculates usage in the current directory and the next subdirectory:

```
    $ du --max-depth 2 DIRECTORY

```

The `-x` option limits `du` to a single filesystem. The default behavior for du is to follow links and mount points.

The `du` command requires read permission for all files, and read and execute for all directories. The `du` command will throw an error if the user running it does not have proper permissions.

# Finding the ten largest size files from a given directory

Combine the `du` and sort commands to find large files that should be deleted or moved:

```
    $ du -ak SOURCE_DIR | sort -nrk 1 | head

```

The `-a` option makes du display the size of all the files and directories in the `SOURCE_DIR`. The first column of the output is the size. The `-k` option causes it to be displayed in kilobytes. The second column contains the file or folder name.

The `-n` option to `sort` performs a numerical sort. The `-1` option specifies column `1` and the `-r` option reverses the sort order. The `head` command extracts the first ten lines from the output:

```
    $ du -ak /home/slynux | sort -nrk 1 | head -n 4
 50220 /home/slynux
 43296 /home/slynux/.mozilla
 43284 /home/slynux/.mozilla/firefox
 43276 /home/slynux/.mozilla/firefox/8c22khxc.default

```

One of the drawbacks of this one-liner is that it includes directories in the result. We can improve the one-liner to output only the large files with the `find` command:

```
    $ find . -type f -exec du -k {} \; | sort -nrk 1 | head

```

The find command selects only filenames for du to process, rather than having du traverse the filesystem to select items to report.

Note that the du command reports the number of bytes that a file requires. This is not necessarily the same as the amount of disk space the file is consuming. Space on the disk is allocated in blocks, so a 1-byte file will consume one disk block, usually between 512 and 4096 bytes.

The next section describes using the `df` command to determine how much space is actually available.

# Disk free information

The `du` command provides information about the usage, while `df` provides information about free disk space. Use `-h` with `df` to print the disk space in a human-readable format. Consider this example:

```
    $ df -h
 Filesystem            Size  Used Avail Use% Mounted on
 /dev/sda1             9.2G  2.2G  6.6G  25% /
 none                  497M  240K  497M   1% /dev
 none                  502M  168K  501M   1% /dev/shm
 none                  502M   88K  501M   1% /var/run
 none                  502M     0  502M   0% /var/lock
 none                  502M     0  502M   0% /lib/init/rw
 none                  9.2G  2.2G  6.6G  25%   
    /var/lib/ureadahead/debugfs

```

The `df` command can be invoked with a folder name. In that case, it will report free space for the disk partition that contains that directory. This is useful if you don't know which partition contains a directory:

```
    $ df -h /home/user
 Filesystem            Size  Used Avail Use% Mounted on
 /dev/md1              917G  739G  133G  85% /raid1

```

# Calculating the execution time for a command

Execution time is the criteria for analyzing an application's efficiency or comparing algorithms.

# How to do it...

1.  The `time` command measures an application's execution time.

Consider the following example:

```
        $ time APPLICATION

```

The `time` command executes `APPLICATION`. When `APPLICATION` is complete, the `time` command reports the real, system, and user time statistics to `stderr` and sends the APPLICATION's normal output to `stdout`.

```
        $ time ls
 test.txt
 next.txt
 real    0m0.008s
 user    0m0.001s
 sys     0m0.003s

```

An executable binary of the `time` command is found in `/usr/bin/time`. If you are running bash, you'll get the shell built-in `time` by default. The shell built-in `time` has limited options. Use an absolute path (`/usr/bin/time`) to access the extended functionality.

2.  The `-o` option will write the time statistics to a file:

```
        $ /usr/bin/time -o output.txt COMMAND

```

The filename must appear immediately after the `-o` flag.

The `-a` flag can be used with `-o` to append the time statistics to a file:

```
        $ /usr/bin/time -a -o output.txt COMMAND

```

3.  The `-f` option specifies the statistics to report and the format for the output. A format string includes one or more parameters prefixed with a `%`. Format parameters include the following:

*   Real time: `%e`

*   User time: `%U`

*   System time: `%S`

*   System Page size: `%Z`

We can create a formatted output by combining these parameters with extra text:

```
        $ /usr/bin/time -f "FORMAT STRING" COMMAND

```

Consider this example:

```
        $ /usr/bin/time -f "Time: %U" -a -o timing.log uname
 Linux

```

The `%U` parameter specifies user time.

The **time** command sends the target application's output to `stdout` and the time command output to `stderr`. We can redirect the output with a redirection operator (`>`) and redirect the time information output with the (`2>`) error redirection operator.

Consider the following example:

```
        $ /usr/bin/time -f "Time: %U" uname> command_output.txt   
        2>time.log
 $ cat time.log
 Time: 0.00
 $ cat command_output.txt
 Linux

```

4.  The format command can report memory usage as well as timing information. The `%M` flag shows the maximum memory used in KB and `%Z` parameter causes the time command to report the system page size:

```
        $ /usr/bin/time -f "Max: %M K\nPage size: %Z bytes" \                      
          ls>   
        /dev/null
 Max: 996 K
 Page size: 4096 bytes

```

In this example, the output of the target application is unimportant, so the standard output is directed to `/dev/null` rather than being displayed.

# How it works...

The time command reports these times by default:

*   **Real**: This is the wall clock time-the time from start to finish of the command. This is the elapsed time including time slices used by other processes and the time the process spends when blocked (for example, time spent waiting for I/O to complete).
*   **User**: This is the amount of CPU time spent in user-mode code (outside the kernel) within the process. This is the CPU time used to execute the process. Other processes, and the time these processes spend when blocked do not count toward this figure.
*   **Sys**: This is the amount of CPU time spent in the kernel within the process; the CPU time spent in system calls within the kernel, as opposed to the library code, which runs in the user space. Like user time, this is only the CPU time used by the process. Refer to the following table for a brief description of the kernel mode (also known as supervisor mode) and the system call mechanism.

Many details regarding a process can be reported by the `time` command. These include exit status, number of signals received, and number of context switches made. Each parameter can be displayed when a suitable format string is supplied to the `-f` option.

The following table shows some of the interesting parameters:

| **Parameter** | **Description** |
| --- | --- |
| `%C` | This shows the name and command-line arguments of the command being timed. |
| `%D` | This shows the average size of the process's unshared data area, in kilobytes. |
| `%E` | This shows the elapsed real (wall clock) time used by the process in [hours:] minutes:seconds. |
| `%x` | This shows the exit status of the command. |
| `%k` | This shows the number of signals delivered to the process. |
| `%W` | This shows the number of times the process was swapped out of the main memory. |
| `%Z` | This shows the system's page size in bytes. This is a per-system constant, but varies between systems. |
| `%P` | This shows the percentage of the CPU that this job got. This is just user + system times divided by the total running time. It also prints a percentage sign. |
| `%K` | This shows the average total (data + stack + text) memory usage of the process, in Kilobytes. |
| `%w` | This shows the number of times that the program was context-switched voluntarily, for instance, while waiting for an I/O operation to complete. |
| `%c` | This shows the number of times the process was context-switched involuntarily (because the time slice expired). |

# Collecting information about logged in users, boot logs, and boot failures

Linux supports commands to report aspects of the runtime system including logged in users, how long the computer has been powered on, and boot failures. This data is used to allocate resources and diagnose problems.

# Getting ready

This recipe introduces the who, w, users, uptime, last, and lastb commands.

# How to do it...

1.  The `who` command reports information about the current users:

```
        $ who
 slynux   pts/0   2010-09-29 05:24 (slynuxs-macbook-pro.local)
 slynux   tty7    2010-09-29 07:08 (:0) 

```

This output lists the login name, the TTY used by the users, login time, and remote hostname (or X display information) about logged in users.

**TTY** (the term comes from **TeleTYpewriter**) is the device file associated with a text terminal that is created in `/dev` when a terminal is newly spawned by the user (for example, `/dev/pts/3`). The device path for the current terminal can be found out by executing the `tty` command.

2.  The `w` command provides more detailed information:

```
 $ w
 07:09:05 up  1:45,  2 users,  load average: 0.12, 0.06, 0.02
 USER     TTY     FROM    LOGIN@   IDLE  JCPU PCPU WHAT
 slynux   pts/0   slynuxs 05:24  0.00s  0.65s 0.11s sshd: slynux 
 slynux   tty7    :0      07:08  1:45m  3.28s 0.26s bash

```

This first line lists the current time, system uptime, number of users currently logged on, and the system load averages for the past 1, 5, and 15 minutes. Following this, the details about each login session are displayed with each line containing the login name, the TTY name, the remote host, login time, idle time, total CPU time used by the user since login, CPU time of the currently running process, and the command line of their current process.

Load average in the `uptime` command's output indicates system load. This is explained in more detail in [Chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Calls*.

3.  The users command lists only the name of logged-in users:

```
        $ users
 slynux slynux slynux hacker

```

If a user has multiple sessions open, either by logging in remotely several times or opening several terminal windows, there will be an entry for each session. In the preceding output, the `slynux` user has opened three terminals sessions. The easiest way to print unique users is to filter the output through `sort` and `uniq`:

```
        $ users | tr ' ' '\n' | sort | uniq
 slynux
 hacker

```

The `tr` command replaces each `' '` character with `'\n'`. Then a combination of `sort` and `uniq` reduces the list to a unique entry for each user.

4.  The `uptime` command reports how long the system has been powered on:

```
        $ uptime
 21:44:33 up 6 days, 11:53, 8 users, load average: 0.09, 0.14,   
        0.09

```

The time that follows the `up` word is how long the system has been powered on. We can write a one-liner to extract the uptime only:

```
       $ uptime | sed 's/.*up \(.*\),.*users.*/\1/'

```

This uses `sed` to replace the line of output with only the string between the word up and the comma before users.

5.  The `last` command provides a list of users who have logged onto the system since the `/var/log/wtmp` file was created. This may go back a year or more:

```
 $ last
        aku1  pts/3   10.2.1.3   Tue May 16 08:23 - 16:14  (07:51)    
        cfly  pts/0   cflynt.com Tue May 16 07:49   still logged in   
        dgpx  pts/0   10.0.0.5   Tue May 16 06:19 - 06:27  (00:07)    
        stvl  pts/0   10.2.1.4   Mon May 15 18:38 - 19:07  (00:29)

```

The `last` command reports who logged in, what `tty` they were assigned, where they logged in from (IP address or local terminal), the login, logout, and session time. Reboots are marked as a login by a pseudo-user named `reboot`.

6.  The `last` command allows you to define a user to get only information about that user:

```
        $ last USER

```

7.  USER can be a real user or the pseudo-user `reboot`:

```
        $ last reboot
 reboot   system boot  2.6.32-21-generi Tue Sep 28 18:10 - 21:48    
        (03:37)
 reboot   system boot  2.6.32-21-generi Tue Sep 28 05:14 - 21:48    
        (16:33)

```

8.  The `lastb` command will give you a list of the failed login attempts:

```
        # lastb
 test     tty8         :0               Wed Dec 15 03:56 - 03:56    
        (00:00) 
 slynux   tty8         :0               Wed Dec 15 03:55 - 03:55    
        (00:00)

```

The `lastb` command must be run as the root user.

Both `last` and `lastb` report the contents of `/var/log/wtmp`. The default is to report month, day, and time of the event. However, there may be multiple years of data in that file, and the month/day can be confusing.

The `-F` flag will report the full date:

```
        # lastb -F
 hacker   tty0       1.2.3.4          Sat Jan 7 11:50:53 2017 -  
        Sat Jan 7 11:50:53 2017 (00:00)

```

# Listing the top ten CPU– consuming processes in an hour

The CPU is another resource that can be exhausted by a misbehaving process. Linux supports commands to identify and control the processes hogging the CPU.

# Getting ready

The `ps` command displays details about the processes running on the system. It reports details such as CPU usage, running commands, memory usage, and process status. The `ps` command can be used in a script to identify who consumed the most CPU resource over an hour. For more details on the `ps` command, refer to [Chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Calls*.

# How to do it...

This shell script monitors and calculates CPU usages for one hour:

```
#!/bin/bash
#Name: pcpu_usage.sh
#Description: Script to calculate cpu usage by processes for 1 hour

#Change the SECS to total seconds to monitor CPU usage.
#UNIT_TIME is the interval in seconds between each sampling

SECS=3600
UNIT_TIME=60

STEPS=$(( $SECS / $UNIT_TIME ))

echo Watching CPU usage... ;

# Collect data in temp file

for((i=0;i<STEPS;i++))
do
 ps -eocomm,pcpu | egrep -v '(0.0)|(%CPU)' >> /tmp/cpu_usage.$$
 sleep $UNIT_TIME
done

# Process collected data
echo
echo CPU eaters :

cat /tmp/cpu_usage.$$ | \
awk '
{ process[$1]+=$2; }
END{ 
 for(i in process)
 {
 printf("%-20s %s\n",i, process[i]) ;
 }

 }' | sort -nrk 2 | head

#Remove the temporary log file
rm /tmp/cpu_usage.$$

```

The output resembles the following:

```
$ ./pcpu_usage.sh
Watching CPU usage...
CPU eaters :
Xorg        20
firefox-bin   15
bash        3
evince      2
pulseaudio    1.0
pcpu.sh         0.3
wpa_supplicant  0
wnck-applet     0
watchdog/0      0
usb-storage     0

```

# How it works...

The CPU usage data is generated by the first loop that runs for one hour (3600 seconds). Once each minute, the `ps -eocomm,pcpu` command generates a report on the system activity at that time. The `-e` option specifies to collect data on all processes, not just this session's tasks. The `-o` option specifies an output format. The `comm` and `pcpu` words specify reporting the command name and percentage of CPU, respectively. This `ps` command generates a line with the command name and current percentage of CPU usage for each running process. These lines are filtered with `grep` to remove lines where there was no CPU usage (%CPU is 0.0) and the `COMMAND %CPU` header. The interesting lines are appended to a temporary file.

The temporary file is named `/tmp/cpu_usage.$$`. Here, `$$` is a script variable that holds the process ID (PID) of the current script. For example, if the script's PID is `1345`, the temporary file will be named `/tmp/cpu_usage.1345`.

The statistics file will be ready after one hour and will contain 60 sets of entries, corresponding to the system status at each minute. The `awk` script sums the total CPU usage for each process into an associative array named process. This array uses the process name as array index. Finally, `awk` sorts the result with a numeric reverse sort according to the total CPU usage and uses head to limit the report to the top 10 usage entries.

# See also

*   The *Using awk for advanced text processing* recipe of [Chapter 4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*, explains the `awk` command
*   The *Using head and tail for printing the last or first 10 lines* recipe of [Chapter 3](19219dcb-e034-408e-8fe2-db6daa9278c8.xhtml), *File In, File Out*, explains the `tail` command

# Monitoring command outputs with watch

The watch command will execute a command at intervals and display that command's output. You can use a terminal session and the screen command described in [chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Calls* to create a customized dashboard to monitor your systems with watch.

# How to do it...

The `watch` command monitors the output of a command on the terminal at regular intervals. The syntax of the `watch` command is as follows:

```
    $ watch COMMAND

```

Consider this example:

```
    $ watch ls

```

Alternatively, it can be used like this:

```
    $ watch 'df /home'

```

Consider the following example:

```
    # list only directories
 $ watch 'ls -l | grep "^d"'

```

This command will update the output at a default interval of two seconds.

The `-n SECONDS` option defines the time interval for updating the output:

```
    # Monitor the output of ls -l every of 5 seconds
 $ watch -n 5 'ls -l'

```

# There's more

The `watch` command can be used with any command that generates output. Some commands change their output frequently, and the changes are more important than the entire output. The watch command will highlight the difference between consecutive runs. Note that this highlight only lasts until the next update.

# Highlighting the differences in the watch output

The `-d` option highlights differences between successive runs of the command being watched:

```
    $ watch -d 'COMMANDS'

 # Highlight new network connections for 30 seconds
 $ watch -n 30 -d 'ss | grep ESTAB'

```

# Logging access to files and directories

There are many reasons you may need to be notified when a file is accessed. You might want to know when a file is modified so it can be backed up, or you might want to know when files in `/bin` are modified by a hacker.

# Getting ready

The `inotifywait` command watches a file or directory and reports when an event occurs. It doesn't come by default with every Linux distribution. You have to install the `inotify-tools` package. It requires the `inotify` support in the Linux kernel. Most new GNU/Linux distributions compile the `inotify` support into the kernel.

# How to do it...

The `inotify` command can monitor a directory:

```
    #/bin/bash
 #Filename: watchdir.sh
 #Description: Watch directory access
 path=$1
 #Provide path of directory or file as argument to script

 $ inotifywait -m -r -e create,move,delete $path  -q 

```

A sample output resembles the following:

```
    $ ./watchdir.sh .
 ./ CREATE new
 ./ MOVED_FROM new
 ./ MOVED_TO news
 ./ DELETE news

```

# How it works...

The previous script will log create, move, and delete events in the given path. The `-m` option causes watch to stay active and monitor changes continuously, rather than exiting after an event happens. The `-r` option enables a recursive watch of the directories (symbolic links are ignored). The `-e` option specifies the list of events to be watched and `-q` reduces the verbose messages and prints only the required ones. This output can be redirected to a log file.

The events that `inotifywait` can check include the following:

| **Event** | **Description** |
| `access` | When a read happens to a file |
| `modify` | When file contents are modified |
| `attrib` | When metadata is changed |
| `move` | When a file undergoes a move operation |
| `create` | When a new file is created |
| `open` | When a file undergoes an open operation |
| `close` | When a file undergoes a close operation |
| `delete` | When a file is removed |

# Logging with syslog

Log files related to daemons and system processes are located in the `/var/log` directory. These log files use a standard protocol called **syslog**, handled by the `syslogd` daemon. Every standard application makes use of `syslogd` to log information. This recipe describes how to use `syslogd` to log information from a shell script.

# Getting ready

Log files help you deduce what is going wrong with a system. It is a good practice to log progress and actions with log file messages. The logger command will place data into log files with `syslogd`.

These are some of the standard Linux log files. Some distributions use different names for these files:

| **Log file** | **Description** |
| `/var/log/boot.log` | Boot log information |
| `/var/log/httpd` | Apache web server log |
| `/var/log/messages` | Post boot kernel information |
| `/var/log/auth.log``/var/log/secure` | User authentication log |
| `/var/log/dmesg` | System boot up messages |
| `/var/log/mail.log``/var/log/maillog` | Mail server log |
| `/var/log/Xorg.0.log` | X server log |

# How to do it...

The `logger` command allows scripts to create and manage log messages:

1.  Place a message in the syslog file `/var/log/messages`:

```
        $ logger LOG_MESSAGE

```

Consider this example:

```
        $ logger This is a test log line

 $ tail -n 1 /var/log/messages
 Sep 29 07:47:44 slynux-laptop slynux: This is a test log line

```

The `/var/log/messages` log file is a general purpose log file. When the `logger` command is used, it logs to `/var/log/messages` by default.

2.  The `-t` flag defines a tag for the message:

```
        $ logger -t TAG This is a message

 $ tail -n 1 /var/log/messages
 Sep 29 07:48:42 slynux-laptop TAG: This is a message

```

The `-p` option to logger and configuration files in `/etc/rsyslog.d` control where log messages are saved.

To save to a custom file, follow these steps:

*   Create a new configuration file in `/etc/rsyslog.d`

*   Add a pattern for a priority and the log file

*   Restart the log daemon

Consider the following example:

```
        # cat /etc/rsyslog.d/myConfig
 local7.* /var/log/local7
 # cd /etc/init.d
 # ./syslogd restart
 # logger -p local7.info A line to be placed in /var/log/local7

```

3.  The `-f` option will log the lines from another file:

```
        $ logger -f /var/log/source.log

```

# See also

*   The *Using head and tail for printing the last or first 10 lines* recipe of [Chapter 3](19219dcb-e034-408e-8fe2-db6daa9278c8.xhtml), *File In, File Out*, explains the head and tail commands

# Managing log files with logrotate

Log files keep track of events on the system. They are essential for debugging problems and monitoring live machines. Log files grow as time passes and more events are recorded. Since the older data is less useful than the current data, log files are renamed when they reach a size limit and the oldest files are deleted.

# Getting ready

The `logrotate` command can restrict the size of the log file. The system logger facility appends information to the end of a log file without deleting earlier data. Thus a log file will grow larger over time. The `logrotate` command scans log files defined in the configuration file. It will keep the last 100 kilobytes (for example, specified S*IZE = 100 k*) from the log file and move the rest of the data (older log data) to a new file `logfile_name.1`. When the old-data file (`logfile_name.1`) exceeds `SIZE`, `logrotate` renames that file to `logfile_name.2` and starts a new `logfile_name.1`. The `logrotate` command can compress the older logs as `logfile_name.1.gz`, `logfile_name.2.gz`, and so on.

# How to do it...

The system's `logrotate` configuration files are held in `/etc/logrotate.d`. Most Linux distributions have many files in this folder.

We can create a custom configuration for a log file (say `/var/log/program.log`):

```
$ cat /etc/logrotate.d/program 
/var/log/program.log { 
missingok 
notifempty 
size 30k 
  compress 
weekly 
  rotate 5 
create 0600 root root 
} 

```

This is a complete configuration. The `/var/log/program.log` string specifies the log file path. Logrotate will archive old logs in the same directory.

# How it works...

The `logrotate` command supports these options in the configuration file:

| **Parameter** | **Description** |
| `missingok` | This ignores if the log file is missing and return without rotating the log. |
| `notifempty` | This only rotates the log if the source log file is not empty. |
| `size 30k` | This limits the size of the log file for which the rotation is to be made. It can be 1 M for 1 MB. |
| `compress` | This enables compression with gzip for older logs. |
| `weekly` | This specifies the interval at which the rotation is to be performed. It can be weekly, yearly, or daily. |
| `rotate 5` | This is the number of older copies of log file archives to be kept. Since 5 is specified, there will be `program.log.1.gz`, `program.log.2.gz`, and so on up to `program.log.5.gz`. |
| `create 0600 root root` | This specifies the mode, user, and the group of the log file archive to be created. |

The options in the table are examples of what can be specified. More options can be defined in the `logrotate` configuration file. Refer to the man page at `http://linux.die.net/man/8/logrotate`, for more information.

# Monitoring user logins to find intruders

Log files can be used to gather details about the state of the system and attacks on the system.

Suppose we have a system connected to the Internet with SSH enabled. Many attackers are trying to log in to the system. We need to design an intrusion detection system to identify users who fail their login attempts. Such attempts may be of a hacker using a dictionary attack. The script should generate a report with the following details:

*   User that failed to log in
*   Number of attempts
*   IP address of the attacker
*   Host mapping for the IP address
*   Time when login attempts occurred

# Getting ready

A shell script can scan the log files and gather the required information. Login details are recorded in `/var/log/auth.log` or `/var/log/secure`. The script scans the log file for failed login attempts and analyzes the data. It uses the `host` command to map the host from the IP address.

# How to do it...

The intrusion detection script resembles this:

```
#!/bin/bash
#Filename: intruder_detect.sh
#Description: Intruder reporting tool with auth.log input
AUTHLOG=/var/log/auth.log

if [[ -n $1 ]];
then
 AUTHLOG=$1
 echo Using Log file : $AUTHLOG
fi

# Collect the failed login attempts
LOG=/tmp/failed.$$.log
grep "Failed pass" $AUTHLOG > $LOG

# extract the users who failed
users=$(cat $LOG | awk '{ print $(NF-5) }' | sort | uniq)

# extract the IP Addresses of failed attempts
ip_list="$(egrep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" $LOG | sort | uniq)"

printf "%-10s|%-3s|%-16s|%-33s|%s\n" "User" "Attempts" "IP address" \       
    "Host" "Time range"

# Loop through IPs and Users who failed.

for ip in $ip_list;
do
 for user in $users;
 do
 # Count attempts by this user from this IP

 attempts=`grep $ip $LOG | grep " $user " | wc -l`

 if [ $attempts -ne 0 ] 
 then
 first_time=`grep $ip $LOG | grep " $user " | head -1 | cut -c-16`
 time="$first_time"
 if [ $attempts -gt 1 ] 
 then
 last_time=`grep $ip $LOG | grep " $user " | tail -1 | cut -c-16`
 time="$first_time -> $last_time"
 fi
 HOST=$(host $ip 8.8.8.8 | tail -1 | awk '{ print $NF }' )
 printf "%-10s|%-3s|%-16s|%-33s|%-s\n" "$user" "$attempts" "$ip"\     
          "$HOST" "$time";
 fi
 done
done

rm $LOG

```

The output resembles the following:

```
Using Log file : secure
User |Attempts|IP address|Host        |Time range
pi   |1  |10.251.90.93   |3(NXDOMAIN) |Jan  2 03:50:24 
root |1  |10.56.180.82   |2(SERVFAIL) |Dec 26 04:31:29 
root |6  |10.80.142.25   |example.com |Dec 19 07:46:49  -> Dec 19 07:47:38 

```

# How it works...

The `intruder_detect.sh` script defaults to using `/var/log/auth.log` as input. Alternatively, we can provide a log file with a command-line argument. The failed logins are collected in a temporary file to reduce processing.

When a login attempt fails, SSH logs lines are similar to this:

```
    sshd[21197]: Failed password for bob1 from 10.83.248.32 port 50035 

```

The script `greps` for the `Failed passw` string and puts those lines in `/tmp/failed.$$.log`.

The next step is to extract the users who failed to login. The `awk` command extracts the fifth field from the end (the user name) and pipes that to sort and `uniq` to create a list of the users.

Next, the unique IP addresses are extracted with a regular expression and the `egrep` command.

Nested for loops iterate through the IP address and users extracting the lines with each IP address and user combination. If the number of attempts for this IP/User combination is > 0, the time of the first occurrence is extracted with `grep`, head, and cut. If the number of attempts is > 1, then the last time is extracted using tail instead of head.

This login attempt is then reported with the formatted `printf` command.

Finally, the temporary file is removed.

# Monitoring remote disk usage health

Disks fill up and sometimes wear out. Even RAIDed storage systems can fail if you don't replace a faulty drive before the others fail. Monitoring the health of the storage systems is part of an administrator's job.

The job gets easier when an automated script checks the devices on the network and generates a one-line report, the date, IP address of the machine, device, capacity of device, used space, free space, percentage usage, and alert status. If the disk usage is under 80 percent, the drive status is reported as `SAFE`. If the drive is getting full and needs attention, the status is reported as `ALERT`.

# Getting ready

The script uses SSH to log in to remote systems, collect disk usage statistics, and write them to a log file in the central machine. This script can be scheduled to run at a particular time.

The script requires a common user account on the remote machines so the `disklog` script can log in to collect data. We should configure auto-login with SSH for the common user (the *Password-less auto-login with SSH* recipe of [Chapter 8](5ba784d5-fa8b-4840-b4c5-cac906e484f9.xhtml), *The Old-Boy Network*, explains auto-login).

# How to do it...

Here's the code:

```
#!/bin/bash 
#Filename: disklog.sh 
#Description: Monitor disk usage health for remote systems 

logfile="diskusage.log" 

if [[ -n $1 ]] 
then 
  logfile=$1 
fi 

   # Use the environment variable or modify this to a hardcoded value 
user=$USER 

#provide the list of remote machine IP addresses  
IP_LIST="127.0.0.1 0.0.0.0" 
# Or collect them at runtime with nmap 
# IP_LIST=`nmap -sn 192.168.1.2-255 | grep scan | grep cut -c22-` 

if [ ! -e $logfile ] 
then 
  printf "%-8s %-14s %-9s %-8s %-6s %-6s %-6s %s\n" \
    "Date" "IP address" "Device" "Capacity" "Used" "Free" \
    "Percent" "Status" > $logfile
fi 
 ( 
for ip in $IP_LIST; 
do 
 ssh $user@$ip 'df -H' | grep ^/dev/ > /tmp/$$.df 

 while read line; 
 do 
 cur_date=$(date +%D) 
 printf "%-8s %-14s " $cur_date $ip 
 echo $line | \
     awk '{ printf("%-9s %-8s %-6s %-6s %-8s",$1,$2,$3,$4,$5); }' 

 pusg=$(echo $line | egrep -o "[0-9]+%") 
 pusg=${pusg/\%/}; 
 if [ $pusg -lt 80 ]; 
 then 
 echo SAFE 
 else 
 echo ALERT 
 fi 

 done< /tmp/$$.df 
done 

) >> $logfile

```

The `cron` utility will schedule the script to run at regular intervals. For example, to run the script every day at 10 a.m., write the following entry in `crontab`:

```
00 10 * * * /home/path/disklog.sh /home/user/diskusg.log 

```

Run the `crontab -e` command and add the preceding line.

You can run the script manually as follows:

```
$ ./disklog.sh

```

The output for the previous script resembles this:

```
01/18/17 192.168.1.6    /dev/sda1   106G    53G    49G     52%     SAFE
01/18/17 192.168.1.6    /dev/md1    958G  776G   159G     84%     ALERT

```

# How it works...

The `disklog.sh` script accepts the log file path as a command-line argument or uses the default log file. The `-e $logfile` checks whether the file exists or not. If the log file does not exist, it is initialized with a column header. The list of remote machine IP addresses can be hardcoded in `IP_LIST`, delimited with spaces, or the `nmap` command can be used to scan the network for available nodes. If you use the `nmap` call, adjust the IP address range for your network.

A for loop iterates through each of the IP addresses. The `ssh` application sends the `df -H` command to each node to retrieve the disk usage information. The `df` output is stored in a temporary file. A `while` loop reads that file line by line and invokes `awk` to extract the relevant data and output it. An `egrep` command extracts the percent full value and strips `%`. If this value is less than 80, the line is marked `SAFE`, else it's marked `ALERT`. The entire output string must be redirected to the `log` file. Hence, the `for` loop is enclosed in a subshell `()` and the standard output is redirected to the log file.

# See also

*   The *Scheduling with a cron* recipe in [Chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Calls*, explains the `crontab` command

# Determining active user hours on a system

This recipe makes use of the system logs to find out how many hours each user has spent on the server and ranks them according to the total usage hours. A report is generated with the details, including rank, user, first logged in date, last logged in date, number of times logged in, and total usage hours.

# Getting ready

The raw data about user sessions is stored in a binary format in the `/var/log/wtmp` file. The `last` command returns details about login sessions. The sum of the session hours for each user is that user's total usage hours.

# How to do it...

This script will determine the active users and generate the report:

```
#!/bin/bash 
#Filename: active_users.sh 
#Description: Reporting tool to find out active users 

log=/var/log/wtmp 

if [[ -n $1 ]]; 
then 
  log=$1 
fi 

printf "%-4s %-10s %-10s %-6s %-8s\n" "Rank" "User" "Start" \ 
 "Logins" "Usage hours" 

last -f $log | head -n -2   > /tmp/ulog.$$ 

cat /tmp/ulog.$$ |  cut -d' ' -f1 | sort | uniq> /tmp/users.$$ 

( 
while read user; 
do 
  grep ^$user /tmp/ulog.$$ > /tmp/user.$$ 
  minutes=0 

  while read t  
  do 
    s=$(echo $t | awk -F: '{ print ($1 * 60) + $2 }') 
    let minutes=minutes+s 
  done< <(cat /tmp/user.$$ | awk '{ print $NF }' | tr -d ')(') 

  firstlog=$(tail -n 1 /tmp/user.$$ | awk '{ print $5,$6 }') 
  nlogins=$(cat /tmp/user.$$ | wc -l)  
  hours=$(echo "$minutes / 60.0" | bc) 

  printf "%-10s %-10s %-6s %-8s\n"  $user "$firstlog" $nlogins $hours 
done< /tmp/users.$$  

) | sort -nrk 4 | awk '{ printf("%-4s %s\n", NR, $0) }'  
rm /tmp/users.$$ /tmp/user.$$ /tmp/ulog.$$ 

```

The output resembles the following:

```
$ ./active_users.sh
Rank User       Start      Logins Usage hours
1    easyibaa   Dec 11     531    349
2    demoproj   Dec 10     350    230
3    kjayaram   Dec 9      213    55
4    cinenews   Dec 11     85     139
5    thebenga   Dec 10     54     35
6    gateway2   Dec 11     52     34
7    soft132    Dec 12     49     25
8    sarathla   Nov 1      45     29
9    gtsminis   Dec 11     41     26
10   agentcde   Dec 13     39     32

```

# How it works...

The `active_users.sh` script reads from `/var/log/wtmp` or a `wtmp` log file defined on the command line. The `last -f` command extracts the log file contents. The first column in the log file is the username. The `cut` command extracts the first column from the log file. The `sort` and `uniq` commands reduce this to a list of unique users.

The script's outer loop iterates through the users. For each user, `grep` is used to extract the log lines corresponding to a particular user.

The last column of each line is the duration of this login session. These values are summed in the inner `while read t` loop.

The session duration is formatted as `(HOUR:SEC)`. This value is extracted with awk to report the last field and then piped to `tr -d` to remove the parentheses. A second `awk` command converts the *HH::MM* string to minutes and the minutes are totaled. When the loop is complete, the total minutes are converted to hours by dividing `$minutes` with 60.

The first login time for a user is the last line in the temporary file of user data. This is extracted with tail and `awk`. The number of login sessions is the number of lines in this file, calculated with `wc`.

The users are sorted by the total usage hours with sort's `-nr` option for the numeric and descending order and `-k4` to specify the sort column (usage hour). Finally, the output of the sort is passed to `awk`, which prefixes each line with a line number representing the rank of each user.

# Measuring and optimizing power usage

Battery capacity is a critical resource on mobile devices, such as notebook computers and tablets. Linux provides tools that measure power consumption, one such command is `powertop`.

# Getting ready

The `powertop` application doesn't come preinstalled with many Linux distributions, you will have to install it using your package manager.

# How to do it...

The `powertop` application measures per-module power consumption and supports interactively optimizing power consumption:

With no options, `powertop` presents a display on the terminal:

```
# powertop

```

The `powertop` command takes measurements and displays detailed information about power usage, the processes using the most power, and so on:

```
PowerTOP 2.3  Overview  Idle stats  Frequency stats  Device stats  Tunable

Summary: 1146.1 wakeups/sec,  0.0 GPU ops/secs, 0.0 VFS ops/sec and 73.0% C  Usage Events/s Category Description
407.4 ms/s 258.7 Process /usr/lib/vmware/bin/vmware
64.8 ms/s 313.8 Process /usr/lib64/firefox/firefox

```

The `-html` tag will cause `powertop` to take measurements over a period of time and generate an HTML report with the default filename `PowerTOP.html`, which you can open using any web browser:

```
    # powertop --html

```

In the interactive mode, you can optimize power usage. When `powertop` is running, use the arrow or tab keys to switch to the Tunables tab; this shows a list of attributes `powertop` can tune to for consuming less power. Choose the ones you want, press Enter to toggle from Bad to Good.

If you want to monitor the power consumption from a portable device's battery, it is required to remove the charger and use the battery for `powertop` to make measurements.

# Monitoring disk activity

A popular naming convention for monitoring tools is to end the name with the `'top'` word (the command used to monitor processes). The tool to monitor disk I/O is called `iotop`.

# Getting ready

The **iotop** application doesn't come preinstalled with most Linux distributions, you will have to install it using your package manager. The iotop application requires root privileges, so you'll need to run it as `sudo` or root user.

# How to do it...

The `iotop` application can either perform continuous monitoring or generate reports for a fixed period:

1.  For continuous monitoring, use the command as follows:

```
        # iotop -o

```

 The `-o` option tells `iotop` to show only those processes that are doing active I/O while it is running, which reduces the noise in the output.

2.  The `-n` option tells iotop to run for *N* times and exit:

```
        # iotop -b -n 2

```

3.  The `-p` option monitors a specific process:

```
       # iotop -p PID

```

 `PID` is the process you wish to monitor.

In most modern distributions, instead of finding the PID and supplying it to `iotop`, you can use the `pidof` command and write the preceding command as follows: # iotop -p `pidof cp`

# Checking disks and filesystems for errors

Linux filesystems are incredibly robust. Despite that, a filesystem can become corrupted and data can be lost. The sooner you find a problem, the less data loss and corruption you need to worry about.

# Getting ready

The standard tool for checking filesystems is `fsck`. This command is installed on all modern distributions. Note that you'll need to run `fsck` as root or via a `sudo`.

# How to do it...

Linux will run `fsck` automatically at boot time if the filesystem has been unchecked for a long time or there is a reason (unsafe reboot after a power glitch) to suspect it's been corrupted. You can run `fsck` manually.

1.  To check for errors on a partition or filesystem, pass the path to `fsck`:

```
 # fsck /dev/sdb3
 fsck from util-linux 2.20.1
 e2fsck 1.42.5 (29-Jul-2012)
 HDD2 has been mounted 26 times without being checked, check forced.
 Pass 1: Checking inodes, blocks, and sizes
 Pass 2: Checking directory structure
 Pass 3: Checking directory connectivity
 Pass 4: Checking reference counts
 Pass 5: Checking group summary information
 HDD2: 75540/16138240 files (0.7% non-contiguous),      
        48756390/64529088 blocks

```

2.  The `-A` flag checks all the filesystems configured in `/etc/fstab`:

```
        # fsck -A

```

This will go through the `/etc/fstab` file, checking each filesystem. The `fstab` file defines the mapping between physical disk partitions and mount points. It's used to mount filesystems during boot.

3.  The `-a` option instructs `fsck` to automatically attempt to fix errors, instead of interactively asking us whether or not to repair them. Use this option with caution:

```
        # fsck -a /dev/sda2

```

4.  The `-N` option simulates the actions `fsck` will perform:

```
 # fsck -AN
 fsck from util-linux 2.20.1
 [/sbin/fsck.ext4 (1) -- /] fsck.ext4 /dev/sda8
 [/sbin/fsck.ext4 (1) -- /home] fsck.ext4 /dev/sda7
 [/sbin/fsck.ext3 (1) -- /media/Data] fsck.ext3 /dev/sda6

```

# How it works...

The `fsck` application is a frontend for filesystem specific `fsck` applications. When we run `fsck`, it detects the type of the filesystem and runs the appropriate `fsck.fstype` command, where `fstype` is the type of the filesystem. For example, if we run `fsck` on an `ext4` filesystem, it will end up calling the `fsck.ext4` command.

Because of this, `fsck` supports only the common options across all filesystem-specific tools. To find more detailed options, read the application specific man pages such as `fsck.ext4`.

It's very rare, but possible, for `fsck` to lose data or make a badly damaged filesystem worse. If you suspect severe corruption of a filesystem, you should use the `-N` option to list the actions that `fsck` will perform without actually performing them. If `fsck` reports more than a dozen problems it can fix or if these include damaged directory structures, you may want to mount the drive in the read-only mode and try to extract critical data before running `fsck`.

# Examining disk health

Modern disk drives run for years with no problems, but when a disk fails, it's a major disaster. Modern disk drives include a **Self-Monitoring, Analysis, and Reporting Technology** (**SMART**) facility to monitor the disk's health so you can replace an ailing drive before a major failure occurs.

# Getting ready

Linux supports interacting with the drives SMART utilities via the `smartmontools` package. This is installed by default on most distributions. If it's not present, you can install it with your package manager:

```
    apt-get install smartmontools

```

Alternatively, this command can be used:

```
    yum install smartmontools

```

# How to do it...

The user interface to `smartmontools` is the `smartctl` application. This application initiates tests on the disk drive and reports the status of the SMART device.

Since the `smartctl` application accesses the raw disk device, you must have root access to run it.

The `-a` option reports the full status of a device:

```
    $ smartctl -a /dev/sda

```

The output will be a header of basic information, a set of raw data values and the test results. The header includes details about the drive being tested and a datestamp for this report:

```
 smartctl 5.43 2012-06-30 r3573 [x86_64-linux-2.6.32-  
    642.11.1.el6.x86_64] (local build)
 Copyright (C) 2002-12 by Bruce Allen,    
    http://smartmontools.sourceforge.net

 === START OF INFORMATION SECTION ===
 Device Model:     WDC WD10EZEX-00BN5A0
 Serial Number:    WD-WCC3F1HHJ4T8
 LU WWN Device Id: 5 0014ee 20c75fb3b
 Firmware Version: 01.01A01
 User Capacity:    1,000,204,886,016 bytes [1.00 TB]
 Sector Sizes:     512 bytes logical, 4096 bytes physical
 Device is:        Not in smartctl database [for details use: -P    
    showall]
 ATA Version is:   8
 ATA Standard is:  ACS-2 (unknown minor revision code: 0x001f)
 Local Time is:    Mon Jan 23 11:26:57 2017 EST
 SMART support is: Available - device has SMART capability.
 SMART support is: Enabled
 ...

```

The raw data values include error counts, spin-up time, power-on hours, and more. The last two columns (`WHEN_FAILED` and `RAW_VALUE`) are of particular interest. In the following sample, the device has been powered on 9823 hours. It was powered on and off 11 times (servers don't get power-cycled a lot) and the current temperature is 30° C. When the value for power on gets close to the manufacturer's **Mean Time Between Failures** (**MTBF**), it's time to start considering replacing the drive or moving it to a less critical system. If the Power Cycle count increases between reboots, it could indicate a failing power supply or faulty cables. If the temperature gets high, you should consider checking the drive's enclosure. A fan may have failed or a filter might be clogged:

```
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  
    WHEN_FAILED RAW_VALUE

 9 Power_On_Hours          0x0032   087   087   000    Old_age   Always       
     -       9823

12 Power_Cycle_Count       0x0032   100   100   000    Old_age   Always
     -       11

194 Temperature_Celsius     0x0022   113   109   000    Old_age   Always
     -       30

```

The last section of the output will be the results of the tests:

```
SMART Error Log Version: 1
No Errors Logged

SMART Self-test log structure revision number 1

Num  Test_Description    Status                  Remaining  LifeTime(hours)  
        LBA_of_first_error
# 1  Extended offline    Completed without error       00%      9825   
        - 

```

The `-t` flag forces the SMART device to run the self-tests. These are non-destructive and can be run on a drive while it is in service. SMART devices can run a long or short test. A short test will take a few minutes, while the long test will take an hour or more on a large device:

```
$ smartctl -t [long][short] DEVICE

$ smartctl -t long /dev/sda

smartctl 5.43 2012-06-30 r3573 [x86_64-linux-2.6.32-642.11.1.el6.x86_64] (local build)
Copyright (C) 2002-12 by Bruce Allen, http://smartmontools.sourceforge.net

=== START OF OFFLINE IMMEDIATE AND SELF-TEST SECTION ===
Sending command: "Execute SMART Extended self-test routine immediately in off-line mode".
Drive command "Execute SMART Extended self-test routine immediately in off-line mode" successful.
Testing has begun.
Please wait 124 minutes for test to complete.
Test will complete after Mon Jan 23 13:31:23 2017

Use smartctl -X to abort test.

```

In a bit over two hours, this test will be completed and the results will be viewable with the `smartctl -a` command.

# How it works

Modern disk drives are much more than a spinning metal disk. They include a CPU, ROM, memory, and custom signal processing chips. The `smartctl` command interacts with the small operating system running on the disk's CPU to requests tests and reports.

# Getting disk statistics

The `smartctl` command provides many disk statistics and tests the drives. The `hdparm` command provides more statistics and examines how the disk performs in your system, which may be influenced by controller chips, cables, and so on.

# Getting ready

The `hdparm` command is standard on most Linux distributions. You must have root access to use it.

# How to do it...

The `-I` option will provide basic information about your device:

```
    $ hdparm -I DEVICE
 $ hdparm -I /dev/sda

```

The following sample output shows some of the data reported. The model number and firmware are the same as reported by `smartctl`. The configuration includes parameters that can be tuned before a drive is partitioned and a filesystem is created:

```
/dev/sda:

ATA device, with non-removable media
 Model Number:       WDC WD10EZEX-00BN5A0 
 Serial Number:      WD-WCC3F1HHJ4T8
 Firmware Revision:  01.01A01
 Transport:          Serial, SATA 1.0a, SATA II Extensions, SATA Rev 2.5, SATA Rev 2.6, SATA Rev 3.0
Standards:
 Used: unknown (minor revision code 0x001f) 
 Supported: 9 8 7 6 5 
 Likely used: 9
Configuration:
 Logical  max current
 cylinders 16383 16383
 heads  16 16
 sectors/track 63 63
 --
 CHS current addressable sectors:   16514064
 LBA    user addressable sectors:  268435455
 LBA48  user addressable sectors: 1953525168
 Logical  Sector size:                   512 bytes
 Physical Sector size:                  4096 bytes
 device size with M = 1024*1024:      953869 MBytes
 device size with M = 1000*1000:     1000204 MBytes (1000 GB)
 cache/buffer size  = unknown
 Nominal Media Rotation Rate: 7200

...
Security: 
 Master password revision code = 65534
 supported
 not enabled
 not locked
 not frozen
 not expired: security count
 supported: enhanced erase
 128min for SECURITY ERASE UNIT. 128min for ENHANCED SECURITY ERASE UNIT. 
Logical Unit WWN Device Identifier: 50014ee20c75fb3b
 NAA  : 5
 IEEE OUI : 0014ee
 Unique ID : 20c75fb3b
Checksum: correct

```

# How it works

The `hdparm` command is a user interface into the kernel libraries and modules. It includes support for modifying parameters as well as reporting them. Use extreme caution when changing these parameters!

# There's more

The `hdparm` command can test a disk's performance. The `-t` and `-T` options performs timing tests on buffered and cached reads, respectively:

```
# hdparm -t /dev/sda
Timing buffered disk reads: 486 MB in  3.00 seconds = 161.86 MB/sec

# hdparm -T /dev/sda
Timing cached reads:   26492 MB in  1.99 seconds = 13309.38 MB/sec

```