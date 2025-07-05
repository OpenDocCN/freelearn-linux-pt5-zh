# Chapter 10. FreeBSD and Command Line Tools

FreeBSD is the bedrock of the FreeNAS server. In this chapter, we will look at some simple FreeBSD commands and also some fundamental FreeBSD administration tasks, including stop and starting different services as well as controlling RAID from the command line.

# Introduction to FreeBSD

Every computer has what is known as an operating system that is a specialized software to control and manage the different resources in the computer including memory, video, networking, and hard disks. Some popular operating systems today include Microsoft Windows, Apple OS X, Linux, and FreeBSD.

As an operating system, FreeBSD has a very respectable heritage and can trace its parentage to the original UNIX operating system of the late 1970's. Version 1 of FreeBSD saw the light of day in the 1993 and as such has had over 15 years of development. The result is a stable, robust, well-designed, and scalable operating system that can compete with the best.

At its core, FreeBSD is a terminal or console-based operating system. This means that you don't need a fancy graphics card, a high-resolution monitor, and an optical mouse to run FreeBSD. All commands are executed by typing them at the keyboard and hitting *ENTER*. This is essential when FreeBSD is being used as a server. Servers traditionally don't have powerful graphic capabilities and often don't even have a monitor attached to them. So, the ability to be able to connect and administer the server using simple command line tools is very important.

As well as being a solid server operating system, FreeBSD also serves as a desktop OS and comes with a full windowing desktop environment.

## Your First FreeBSD Commands

The easiest way to get to the FreeBSD command line is via the FreeNAS console menu. With FreeNAS up and running, go to the FreeNAS machine and press *ENTER* to make the splash screen disappear and the console menu will appear. Option 6 is for Shell, so type 6 and press ENTER.

You will now see a prompt like this:

```
freenas:~# 

```

### Print the Working Directory with pwd

This is FreeBSD! To run your first FreeBSD command type `pwd` and press ENTER.

```
freenas:~# pwd
/root 

```

`pwd` means Print Working Directory. This tells you your current folder. In this case, it is the folder called `root`. In FreeBSD, the administrator user is known as root. When you connect to FreeBSD via the FreeNAS console, you are automatically logged in as root and you have full administration rights. The `/root` folder is the home directory for the user root.

### Note

**Consider Disabling the Console Menu**

As you can see, by default, the console gives you unfettered access to the FreeBSD command line and gives you full administrator rights from the go. If your FreeNAS server is in an environment where others can access the console menu then you should consider disabling the console (on the **System: Advanced** page in the web interface). This will stop unauthorized and potentially dangerous access to your FreeNAS server.

With the console menu disabled, you will still be able to access the FreeBSD command line via the SSH protocol.

### Directory Listings (ls)

To see the contents of the current folder, you use the `ls` (list) command:

```
freenas:~# ls
.cshrc .dialogrc .history .profile 

```

By default, there isn't very much in the `/root` folder. Here, we can see that there are 4 files. Notice that they all start with a dot. This means they are hidden files but because you are the administrator, hidden files are shown by default. If a normal user uses the `ls` command then the files starting with dot are not shown.

To get more information about the files like their size and their read/write permissions, use the `-l` (long format) option:

```
freenas:~# ls -l
total 8
-rw-r--r-- 1 root wheel 843 Apr 14 10:52 .cshrc
-rw-r--r-- 1 root wheel 57 Feb 22 21:16 .dialogrc
-rw------- 1 root wheel 123 Apr 2 13:13 .history
-rw-r--r-- 1 root wheel 236 Feb 22 21:16 .profile 

```

This long format shows more information about each of the files starting with the file permissions, number of links, owner name, group name, size of the file in bytes, the date and time the file was lost or modified, and of course, the file name.

The file permissions field can look a little complicated but it is easy to understand with a little guidance. The field, if made up of 10 flags, which are either a letter, like r or w, or the hyphen sign (-).

| File type(1 flag) | User permissions(3 flags) | Group permissions(3 flags) | Other permissions(3 flags) |
| --- | --- | --- | --- |
| d for directory, - for a regular file. | r, w, and x meaning user readable, user writable, and user executable. | r, w, and x meaning group readable, group writable and group executable. | r, w, and x meaning world readable, world writable, and world executable. |

For example, the above directory list contains an entry for the `.cshrc file`.

```
-rw-r--r-- 1 root wheel 843 Apr 14 10:52 .cshrc 

```

This means that it is a regular file (as the first flag is `-`) and that it is readable and writable by the user, but not executable (`rw-`). It is normal that the file isn't executable as it isn't a program file. The group permissions (`r--`) mean that users in the same group (wheel) can read the file but can't write to it and the same is true of other users (`r--`).

### Change Directory with cd

To complete the simple file system commands, there is the `cd` (change directory) command. This will change the current working directory to another directory as specified. To change directory to the very top of the file system you would type:

```
freenas:~# cd / 

```

From here, you can see all the folders that exist below the top, in a tree like structure.

```
freenas:/# ls
conf etc mnt usr
bin ftmp root var
boot dev lib sbin
cf entropy libexec tmp 

```

To move to another directory, you just enter `cd` followed by its name:

```
freenas:/# cd /usr
freenas:/usr# ls
X11R6 bin lib libexec local sbin share 

```

And then, deeper still:

```
freenas:/usr# cd bin
freenas:/usr/bin# pwd
/usr/bin 

```

To go back up a directory level, use the special name :

```
freenas:/usr/bin# cd ..
freenas:/usr# pwd
/usr 

```

You can also go directly to a deep folder by specifying its full path in the `cd` command. Before that, we changed directory into the `/usr` directory and then deeper down into the `bin` directory. To do that in one go type:

```
freenas:/# cd /usr/bin
freenas:/usr/bin# pwd
/usr/bin 

```

### Copy a File and Change Its Permissions (cp and chmod)

To copy a file, you need to use the `cp` (copy) command. To copy the `.cshrc` file to `test`, you would type:

```
freenas:~# cp .cshrc test
freenas:~# ls
.cshrc .dialogrc .history .profile test 

```

Using the `ls` command afterwards, shows that the file has been copied. To see the file permissions of the file `test` type:

```
freenas:~# ls -l test
-rw-r--r-- 1 root wheel 843 Apr 14 12:06 test 

```

To change the file so that only the user has read and write permissions and the no others (including those in the same group) can read it, use the `chmod` (change file mode) command.

```
freenas:~# chmod 600 test
freenas:~# ls -l test
-rw------- 1 root wheel 843 Apr 14 12:06 test 

```

`chmod` takes two parameters: the first is a 3 digit number representing the file permissions you wish to set, and the second is the name of the file or directory you wish to change. Each digit represents the file permissions for either the user, group or world (in that order).

The numbers for the file permissions are as follows:

| File permission | Flags | Meaning |
| --- | --- | --- |
| 0 | --- | Nothing, no Read, Write or Execute |
| 1 | --x | Execute |
| 2 | -w- | Write |
| 3 | -wx | Execute & Write |
| 4 | r-- | Read |
| 5 | r-x | Execute & Read |
| 6 | rw- | Read & Write |
| 7 | rwx | Execute & Read & Write |

Therefore, setting the file permission to 600 means read and write for the user and nothing for the group or world (rw-------). Similarly 640 means read and write for the user, read for the group, and nothing for others (rw-r-----). The most open you can make a file is 777, which grants read, write, and executable permission to user, group, and world (rwxrwxrwx ).

Optionally, chmod all takes a -R flag that can be used on a directory and will cause chmod to set the file permission of all the file and subfolders of the directory.

## Connecting to FreeBSD Using Putty

To get access to the FreeBSD command line, without using the console, you can connect to the FreeNAS server via SSH.

SSH (Secure Shell) is a network protocol that allows data to be exchanged over an encrypted (secure) channel between two computers. It is most commonly used as a secure command line interface to a remote computer. This means that you can access the command line interface of the FreeNAS server from a remote computer without having to have access to the keyboard and monitor of the FreeNAS server.

By default, SSH access is disabled, to enable it, go to **Services: SSHD** and enable the **SSH Daemon** (server) by ticking **Enable** in the title of the configuration data table. Click **Save and Restart** to start the SSH server.

There are two types of SSH users on the FreeNAS server. The first is the normal user without administrator access. For each user created on the **Access: Users** page, you can enable Full Shell access that will allow the users to connect and use the FreeBSD command line on the server via SSH.

The second type of user is root. By default, root is not allowed to log in to the FreeNAS server via SSH. To allow root to log in, go to **Services: SSHD** and tick **Permit root login**. Then click **Save and Restart** to finish start the SSH server.

### Note

**Root Password**

The root password is the same as the web interface password, which by default is *freenas*. If the web interface password is changed, so does the root password.

To connect via SSH on Linux or Apple OS X, you can use the SSH command line program. So, to connect to the FreeNAS server, you would use:

```
ssh -l root 192.168.1.250

```

The `-l` parameter allows you to specify the user name which in this case was root.

Windows doesn't come with a SSH utility by default so you need to use a free utility called PuTTY. PuTTY is a great tool written by Simon Tatham.

You can download PuTTY from [http://www.chiark.greenend.org.uk/~sgtatham/putty/](http://www.chiark.greenend.org.uk/~sgtatham/putty/)

![Connecting to FreeBSD Using Putty](img/4688_10_01.jpg)

Once downloaded, double click on the executable (there is no installer for PuTTY, you just use the downloaded file). The main PuTTY window has lots of options but all you need to do to use it is enter the IP address of the FreeNAS server in the Host Name field in the top half of the window. Leave everything else as it is and click Open. A window with a black background will appear. If this is the first time you have connected to this FreeNAS server using PuTTY, you will also be asked if you trust the machine to which you have connected. Click Yes.

At the login as: prompt type root and press *ENTER* and then enter the password which will be the same as the web interface password. You will then see the, hopefully now, familiar *freenas*:~# prompt.

From herein, you have access to FreeBSD as you did from the console menu shell.

## Monitoring your FreeNAS Server from the Command Line

FreeBSD contains several tools for system monitoring including monitoring the disk space and the system processes.

### See Which Disks are Mounted with mount

To see which disks are mounted on the FreeNAS server, use the `mount` command:

```
 freenas:~# mount
/dev/ad0s1a on / (ufs, local, soft-updates)
devfs on /dev (devfs, local)
/dev/raid5/raid5p1 on /mnt/raid5 (ufs, local, soft-updates, acls) 

```

This yields the same output as the **Diagnostics: Information: Mounts** page on the web interface. Each device is listed along with its mount point and what type of filing system it is and any options. From the above, we can see the top most directory/(also refereed to as root but not to be confused with the user root) that contains the FreeBSD and FreeNAS software is on the first IDE drive (ad0). We can also see that this FreeNAS server has a raid5 configuration mounted on /mnt/raid.

### Check Disk Space Usage with df

Another useful command (which is also available on the web interface at **Diagnostics: Information: Space Used** ) is the df command. df shows the disk space usage. It has an optional parameter `-h` (the human readable flag), which makes the output more friendly.

```
 freenas:~# df -h
Filesystem Size Used Avail Capacity Mounted on
/dev/ad0s1a 121M 56M 56M 50% /
devfs 1.0K 1.0K 0B 100% /dev
/dev/raid5/raid5p1 3.9G 239M 3.3G 7% /mnt/raid5 

```

Each filesystem is listed with its total size, space used, space available, and what percentage of the disk is used.

### Discover the Size of Directories Using du

Another very useful command which isn't included in the web interface is the *du* command. The *du* command displays the disk space usage for each file and for each folder given including the subfolders or for the current folder if none is given.

From the example above, we see that the RAID 5 array has 239MB used. If we change directory into `the /mnt/raid5` directory and then run the `du` command (with the `-h` flag for human readability and `-s` for summary) we see that the 239MB listed in the `df` command is also listed from the `du` command:

```
 freenas:~# cd /mnt/raid5/
freenas:/mnt/raid5# du -hs
239M . 

```

Inside the `/mnt/raid5`, there is a folder called `pictures`, to discover how much disk space is used by the pictures folder use the `du` command, either by changing directory to that folder or specifying it directly:

```
 freenas:~# du -h /mnt/raid5/pictures
2.5M /mnt/raid5/pictures 

```

### Process Monitoring Using ps and top

FreeBSD is able to run many programs at the same time. The FreeNAS server includes a web server, an FTP server, and a SSH server etc which all run at the same time. Each of these programs runs as a separate process. Each process uses time on a system's CPU, as well as other system resources such as memory and disk space. If a program goes wrong, it can start to use too much CPU time or memory and so deny other programs the resources they need to run. There are some FreeBSD commands to monitor the status of the process running on your server.

`ps` shows the current processes running on the machine. `ps` has many different options, but one of the most useful invocations is `ps aux`, which shows every process on the system.

A normal FreeNAS server will have some 60 to 70 processes running after boot up, so the output from the `ps` command can be quite long. Here are the first few lines from a FreeNAS server:

```
 freenas:~# ps aux
USER PID %CPU %MEM VSZ RSS TT STAT STARTED TIME COMMAND
root 10 84.6 0.0 0 8 ?? RL 10:28AM 69:12.51 [idle: cpu0]
root 0 0.0 0.0 0 0 ?? WLs 10:28AM 0:00.00 [swapper]
root 1 0.0 0.2 772 388 ?? SLs 10:28AM 0:00.09 /sbin/init --
root 2 0.0 0.0 0 8 ?? DL 10:28AM 0:01.22 [g_event]
root 3 0.0 0.0 0 8 ?? DL 10:28AM 0:01.14 [g_up]
root 4 0.0 0.0 0 8 ?? DL 10:28AM 0:01.60 [g_down]
root 5 0.0 0.0 0 8 ?? DL 10:28AM 0:00.00 [crypto]
root 6 0.0 0.0 0 8 ?? DL 10:28AM 0:00.00 [crypto returns] 

```

Here is a brief explanation of each of the columns:

| Column | Meaning |
| --- | --- |
| USER | This is the name of the user that owns the processes. |
| PID | Each process has a unique process ID (or PID for short). |
| %CPU | Shows the CPU utilization of the process. It is a decaying average over up to a minute of previous (real) time. |
| %MEM | This is the amount of the physical memory the process is using. |
| VSZ | Shows the virtual memory size of the process in kilobytes. |
| RSS | This is similar to VSZ, but rather than virtual memory size, RSS shows how much non-swapped, physical memory the process is using in kilobytes. |
| TT | The controlling terminal. Means there isn't one. |
| STAT | The status of the process, where S means the process is sleeping and can be woken at any time, L means the process is waiting to acquire a lock. R marks a runnable process. |
| STARTED | Shows when the process was started. |
| TIME | Is the accumulated CPU time. This includes time spent running the processes and time spent in the kernel on behalf of that process. |
| COMMAND | Shows the command which was given to launch the program. |

Finding a specific process in such a long list can be a problem. To help, you can use the grep command to look for matches in the text. For example, to look for the ftp server process, use the command:

```
 freenas:~# ps aux | grep ftp
root 981 0.0 0.8 3636 1904 ?? Ss 0:00.07 pure-ftpd
root 1407 0.0 0.4 1528 984 p0 R+ 0:00.02 grep ftp 

```

When you run it, the `grep` command itself will be shown (in this case PID 1407) as it matches the string we are looking for, namely ftp. But of course, it isn't part of the ftp service.

While `ps` shows only a snapshot of the system process, the `top` program provides a dynamic real-time view of a system. It displays a system summary (with CPU usage, memory usage, and other statistics) as well as a list of running processes that changes dynamically as the system is in use. It lists the processes using the most CPU first.

```
 The first few lines of top look something like this:
last pid: 1410; load av: 0.00, 0.00, 0.00 up 0+01:39:09 12:07
23 processes: 2 running, 21 sleeping
CPU: 0.0% user, 0.0% nice, 3.8% sys, 0.0% interrupt, 96.2% idle
Mem: 10M Active, 12M Inact, 13M Wired, 68K Cache, 9648K Buf, 207M Free
Swap:
PID USERNAME THR PRI NICE SIZE RES STATE TIME WCPU COMMAND
1087 root 1 4 0 3168K 2212K kqread 0:02 0.00% lighttpd
1023 root 3 20 0 7160K 4104K kserel 0:01 0.00% mediatom
1250 root 1 76 0 5640K 2720K RUN 0:01 0.00% sshd
927 root 1 76 0 5496K 3208K select 0:01 0.00% nmbd
1253 root 1 20 0 4000K 2780K pause 0:01 0.00% csh 

```

The bottom part of the output is similar to the output from the `ps` command.

# Advanced FreeBSD Commands for FreeNAS

Up until now, we have really been in read-only mode as far as using the underlying FreeBSD system. We have looked and monitored but we haven't actually changed anything. That is about to change.

## Starting and Stopping Services

You will have probably noticed that many of the configuration pages on the web interface say **Save and Restart**. This is because many of the FreeNAS server components need to restart to accept new configurations. As such, it is also possible to restart a service manually using the command line. This might be necessary if a particular service, for example the FTP server or the AFP server stopped responding (this isn't a slur on the FTP server or the AFP server, just merely an example).

Using the command line, it is possible to start, stop, and restart each individual service. All of the scripts to control the various services are kept in `/etc/rc.d` and to manage a service, you call the respective script directly from that directory. To restart the AFP service, you would type:

```
 freenas:~# /etc/rc.d/afpd restart
Stopping afpd.
Starting afpd. 

```

Restart is not the only command that is accepted by the scripts:

| Command | Meaning |
| --- | --- |
| start | Starts the service. If the service is already running, no action will be taken. |
| stop | Stop the service. If the service isn't running, nothing will happen. |
| restart | Performs a stop and then a start. If the service isn't running, the stop will fail but the script will continue to start the service. |
| status | Shows if the service is running. |

Here is a table of the possible services you can start and stop:

| Name | Service description |
| --- | --- |
| afpd | The Apple Filing Protocol Daemon. This provides the connectivity to Apple Mac computers. |
| lighttpd | The built-in web server for the web interface. |
| mediatomb.sh | The UPnP server. |
| nfsd | The NFS server for sharing files with UNIX type clients. |
| nfslocking | Part of the NFS server and needs to be controlled separately. |
| pureftpd | The FTP server. |
| rsync_client | The RSYNC client. |
| rsync_local | The local RSYNC client for synchronization between two local disks. |
| rsyncd | The RSYNC server. |
| samba | The CIFS/SMB server for Windows connectivity. |
| smartd | The hard disk monitoring service. |
| sshd | The secure shell service. |
| unison | The unison synchronization service. |

### Getting Drastic with kill and killall

The `kill` command attempts to shut down a running process. In FreeBSD, a process is stopped when the operating system sends it a signal telling it to shut down. The default signal for `kill` is TERM (signal 15), meaning software terminate. When the process receives the signal, it should shut down in an orderly way. If the process has become rogue, chances are that it won't respond to being told politely to shut down. In that case, you have to send the KILL signal (signal 9 for short). So to kill off a running process (e.g. process 1234) we would use `kill -9 1234`.

The `killall` command kills running processes by name rather than by PID. This has the advantage that to kill a process you don't need to look for the PID using the `ps` command. As with `kill, killall` takes a signal parameter, and `-9` is used to terminate the processes. So to kill off all the ftp processes you would use:

```
 freenas:~# killall -9 pure-ftpd 

```

One thing to note about `killall` is that you need to specify the exact name of the process, using `killall -9 pure or killall -9 ftp` will not stop the FTP server.

A couple of useful parameters to `killall` are `-s` and `-v`.

`-s` will show only what would be done, but does not send any signal.

`-v` will show a similar output to that of `-s` but will actually send the signal while reporting what has been done.

```
 freenas:~# killall -9 -s pure-ftpd
kill -KILL 2045 

```

## RAID Command Line Tools

Reading the FreeNAS support forums on sourceforge.net seem to show that a fair percentage of users have troubles with RAID configurations. The RAID software in FreeNAS is of the highest quality but things can go wrong. All the RAID functions (and more) that are available in the web interface are available on the command line.

Each different type of RAID level (RAID 0, RAID 1, and so on) uses a different command as it is a specialized program to deal with that RAID level. A utility to manage RAID 1 sets knows nothing RAID 5 and vice-versa. The RAID utilities are: gconcat, gstripe, gmirror, and graid5.

### Warning

A word of warning before starting. With the command line, you have complete freedom to manage and control your RAID sets. But this also means you have complete freedom to wreck your RAID sets. Be careful that you don't destroy your RAID sets by mistake. If you are uncomfortable with managing the RAID sets via the command line, you should return to using the web interface as that offers some level of protection.

Another possible problem is that the web interface can become out-of-sync with the current server configuration when you are using the command line. In one sense it is like doing things behind the back of the web interface and it doesn't know what has changed. For example if disk *da0* fails in a RAID set, and another disk *da1* is added to the system and that disk is used to repair the RAID array, then although the RAID will function correctly, the web interface will know nothing about the disk *da1*. At its worst, this is just an annoyance, especially if the new disk *da1* saved your valuable data.

### List and Status Commands

Although each of the utilities for managing RAID levels is different, they do have some common commands. Every utility accepts the `list` and `status` commands.

The `status` command provides a short summary of the disks that make up the RAID set and the current status of the RAID set. Here is the example output of status for a RAID 5 array:

```
 freenas:~# graid5 status
Name Status Components
raid5/myraid5 COMPLETE CALM da0
ad3
ad1 

```

From this, we can see that the RAID5 is COMPLETE (no disks missing), which also means it isn't rebuilding, and that disks *da0, ad3*, and *ad1* make up the RAID set.

The output from the list command is more comprehensive and in many ways is debug information. When your RAID array is working well, this information isn't very interesting, but when you are having troubles with your RAID set, this information can be very valuable.

To get the list information for a RAID 5, you would type:

```
 freenas:~# graid5 list 

```

I have split the output into different sections to aid easier reading:

```
 Geom name: myraid5
State: COMPLETE CALM
Status: Total=3, Online=3 

```

This first section shows us that this RAID set is called `myraid5` and that it is `COMPLETE` and `CALM`. It is made up of 3 disks of which all 3 are online.

```
 Type: AUTOMATIC
Pending: (wqp 0 // 0)
Stripesize: 131072
MemUse: 0 (msl 0)
Newest: -1
ID: 1419279684 

```

The next section shows different internal information about how the RAID set is implemented. Each RAID level will have different information in this section.

```
 Providers:
1\. Name: raid5/myraid5
Mediasize: 4294705152 (4.0G)
Sectorsize: 512
Mode: r1w1e2 

```

This section shows us what this RAID set provides for the system. It provides a RAID 5 array that is 4 Gigabytes in size.

```
 Consumers:
1\. Name: da0
Mediasize: 2147483648 (2.0G)
Sectorsize: 512
Mode: r2w2e3
DiskNo: 2
Error: No
2\. Name: ad3
Mediasize: 2147483648 (2.0G)
Sectorsize: 512
Mode: r2w2e3
DiskNo: 1
Error: No
3\. Name: ad1
Mediasize: 2147483648 (2.0G)
Sectorsize: 512
Mode: r2w2e3
DiskNo: 0
Error: No 

```

The consumers are the disks that are used to make this RAID set. The disk name is listed along with its size, disk number (the order the disks are used in), and its error state.

### JBOD and gconcat

Because JBOD (Just a Bunch of Disks) doesn't offer any kind of protection against disk failure, there isn't much that can be done on the command line. The status of the JBOD array can be checked using `gconcat status` as follows:

```
 freenas:~# gconcat status
Name Status Components
concat/myjbod UP ad3
ad1 

```

It is possible to create, format, and mount JBOD arrays completely from the command line but it is of little value as the web interface will know nothing about the newly created array and as such it can be used (via the web interface). Also, because the array isn't saved in the FreeNAS configuration it will be forgotten when the machine is rebooted.

### RAID 0 and gstripe

Like JBOD, RAID 0 doesn't provide any protection against disk failure and so there is little that can be done on the command line. The status of the RAID 0 array can be checked using the `gstripe status` command:

```
 freenas:~# gstripe status
Name Status Components
stripe/myraid0 UP da0
da1 

```

It is possible to create, format, and mount RAID1 arrays completely from the command line but it is of little value as the web interface will know nothing about the newly created array and as such, it can be used (via the web interface). Also, because the array isn't saved in the FreeNAS configuration, it will be forgotten when the machine is rebooted.

### RAID 1 and gmirror

RAID 1 (mirroring) is the first of the 4 basic RAID levels provided by FreeBSD/FreeNAS that offers some kind of protection against disk failure.

Here is the output from gmirror status when one of the disks (da0) from the mirror set is missing:

```
 freenas:~# gmirror status
Name Status Components
mirror/mymirror DEGRADED ad1 

```

The steps to rebuild a mirror array are the same as those set down is chapter 9\. The command line can be used to rebuild the array rather than the web interface. Once the new disk has been put back into the system, you need to use the `forget` command. This command sounds a bit harsh but don't worry it isn't going to forget the whole mirror set, only the drives that are not currently available. Having issued the forget command, you can insert the new disk and the array will rebuild.

```
 freenas:~# gmirror forget mymirror
freenas:~# gmirror insert mymirror da0
freenas:~# gmirror status
Name Status Components
mirror/mymirror DEGRADED ad0
da0 (17%) 

```

Once the mirror set is rebuilt, the status will be like this:

```
 freenas:~# gmirror status
Name Status Components
mirror/mymirror COMPLETE ad1
da1 

```

### RAID 5 and graid5

The procedure to repair a RAID 5 array after disk failure is exactly the same as described in chapter 9 but this time we will use the command line rather than the web interface.

Using the status command, we can see that there is a problem with this array:

```
 freenas:~# graid5 status
Name Status Components
raid5/myraid5 DEGRADED CALM ad3
ad1 

```

The disk `da0` is missing and the RAID is running in a degraded state. Having replaced the disk, we are ready to synchronize it with the other disks in the RAID set.

The new disk needs to be placed back into the array. This is done using the `graid5 insert` command.

```
 freenas:~# graid5 insert myraid5 da0 

```

The array will now start rebuilding. You can check that it is rebuilding by using the `graid5 status` command again:

```
 freenas:~# graid5 status
Name Status Components
raid5/myraid5 REBUILDING CALM ad3
ad1
da0 (543162368 / 25% (p:0)) 

```

Once the array is rebuilt, it will return to its `COMPLETE` status.

```
 freenas:~# graid5 status
Name Status Components
raid5/myraid5 COMPLETE CALM ad3
ad1
da0 

```

# Where the FreeNAS Stores Things

There are several places like /etc/rc.d where the FreeNAS server stores important files. Here is a summary of some key directories from FreeBSD that the FreeNAS server uses.

| Directory | Importance |
| --- | --- |
| /root | The home directory of the root user |
| /mnt | All disks and RAID sets are mounted under this directory |
| /bin & /usr/bin | Store all the user runnable utilities like *chmod* and *kill* |
| /sbin & /usr/sbin | Store all the root runnable utiliteis like the RAID utilities |
| /etc | This directory contains the various configuration files needed by FreeNAS. Many of them are created at bootup. |
| /usr/local/www | Here are the web pages for the FreeNAS web interface. |
| /var/log | Here and in directories below it are all the log files stored by the FreeNAS server. |

### Note

Note that the embedded version of FreeNAS only runs from RAM (and is initially loaded from the hard disk or USB flash disk) and any changes made to the operating system files will only be temporary, and when the system is rebooted it will return to its original state.

# Miscellaneous & Sundries

There are a variety of commands that can be useful when using the command line but they aren't quite big enough to warrant a section of their own, so I have grouped them all together here.

## Using ping and arp from the Command Line

In chapter 9, we looked at the ping and arp commands. These are available from the web interface but they are also available from the command line. To ping another machine from the command line type:

```
 freenas:~# ping 192.168.1.42
PING 192.168.1.42 (192.168.1.42): 56 data bytes
64 bytes from 192.168.1.42: icmp_seq=0 ttl=64 time=0.718 ms
64 bytes from 192.168.1.42: icmp_seq=1 ttl=64 time=0.613 ms
64 bytes from 192.168.1.42: icmp_seq=2 ttl=64 time=0.536 ms
64 bytes from 192.168.1.42: icmp_seq=3 ttl=64 time=0.697 ms
^C
--- 192.168.1.42 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.536/0.641/0.718/0.072 ms

```

The ping command is a bit different to that of other platforms (like Windows) in that it will keep on pinging until you press *CTRL+C* and then it will stop. Alternatively, you can use the `-c` parameter that sends only the specified number of pings:

```
 freenas:~# ping -c 1 192.168.1.42
PING 192.168.1.42 (192.168.1.42): 56 data bytes
64 bytes from 192.168.1.42: icmp_seq=0 ttl=64 time=0.685 ms
--- 192.168.1.42 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.685/0.685/0.685/0.000 ms 

```

Also, to look at the arp tables type:

```
 freenas:~# arp -a
? (192.168.1.42) at 00:08:02:5a:9b:f5 on lnc0 [ethernet]
Mac-Mini.lan (192.168.1.110) at 00:16:cb:a3:72:1c on lnc0
Toshiba-Laptop.lan (192.168.1.242) at 00:1b:9e:36:d9:ad on lnc0
speedtouch.lan (192.168.1.254) at 00:14:7f:2e:32:2d on lnc0 

```

## Creating Directories and Deleting Things

Earlier on, we looked at some simple file system command like change directory (cd), print working directory (pwd), and copy (cp). Here are a few more commands that you might find useful when working with the file system from the command line:

| Command | Description |
| --- | --- |
| mv | Move a file from one place to another, which also has the effect of doing a rename. eg. *mv oldfilename newfilename* |
| mkdir | Create a directory. eg. *mkdir temp* |
| rmdir | Remove a directory. The directory must be empty. eg. *rmdir temp* |
| rm | Remove (delete) a file. eg. *rm deleteme* |
| rm -rf | Remove (delete) a non empty directory and delete all the file and sub directories in it. Use with care! eg. *rm -rf goodbyeworld* |

## Editing Files Using nano

Included in the FreeNAS software is a small text editor called nano. Provided by GNU, nano is small and friendly. Besides basic text editing, nano offers many extra features like an interactive search and replace, go to line and column number, auto-indentation, feature toggles, internationalization support, and filename tab completion.

To edit a file, say a text file on one of your disks you would type:

```
 freenas:~# nano /mnt/store/readme.txt 

```

The basic keys are displayed at the bottom of the text editor to help you quickly find the key you need. This is especially handy if you are not familiar with the editor. The ^ symbol means press the *CTRL* key and the letter mentioned at the same time, so ^X means *CTRL+X*.

Basic keys include:

^O WriteOut (Save)

^R Read File

^Y Prev Page

^X Exit

^J Justify

^V Next Page

You can get more information on nano at: [http://www.nano-editor.org/](http://www.nano-editor.org/)

## Shutting Down Using the Command Line

We are now at the last section of the last chapter this book and as the book draws to an end, it seems appropriate to show you how to shutdown the FreeNAS server from the command line. To shutdown the server, use the `shutdown` command. Used with the `-p` parameter, the server will be shut down and powered off (if the hardware supports it) and with `-r` the server will be rebooted. The command also needs a time on when to shutdown, and for immediate shutdowns use the word `now`.

```
 freenas:~# shutdown -p now 

```

or

```
 freenas:~# shutdown -r now 

```

If you don't want to shutdown now but want to schedule the shutdown for some time in the future then you can change the word `now` to be a number of minutes until shutdown with a plus sign in front. So to shutdown in 5 minutes from now use:

```
 freenas:~# shutdown -p +5 

```

Finally, if you want to schedule a shutdown for a certain day at a certain time then the time of shutdown can be specified as *yymmddhhmm* if you leave out the *yymmdd* then the time will be take to be today.

Shutdown at 23:15 tonight is:

```
 freenas:~# shutdown -p 2315 

```

And to shutdown on the 1^(st) May 2009 at 22:30 (which happens to be a Friday) is:

```
 freenas:~# shutdown -p 0905012230 

```

# Summary

In this chapter, we have looked at FreeBSD. We have covered the basic commands for file manipulation as well as some more complex commands for managing processing including starting and stopping the various FreeNAS services. We also looked at commands for managing RAID sets.