# 13 Logging and Log Security

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file82.png)

System logs are an important part of every IT administrator's life. They can tell you how well your system is performing, how to troubleshoot problems, and what the users—both authorized and unauthorized—are doing on the system.

In this chapter, I'll give you a brief tour of the Linux logging systems, and then show you a cool trick to help make your log reviews easier. Then, I'll show you how to set up a remote logging server, complete with **Transport Layer Security** (**TLS**)-encrypted connections to the clients.

The topics that we will be covering are:

*   Understanding the Linux system log files
*   Understanding `rsyslog`
*   Understanding `journald`
*   Making things easier with Logwatch
*   Setting up a remote log server

The focus of this chapter is on logging tools that are either already built into your Linux distro or that are available in your distro repositories. Other Packt Publishing books, such as the *Linux Administration Cookbook*, by Adam K. Dean, show you some of the fancier, more advanced third-party log aggregation and analysis tools.

So, if you're ready and raring to go, let's look at those Linux log files.

## Understanding the Linux system log files

You'll find the Linux log files in the `/var/log/` directory. The structure of Linux log files is pretty much the same across all Linux distros. But, in the Linux tradition of trying to keep us all confused, the main log files have different names on different distros. On Red Hat-type systems, the main log file is the `messages` file, and the log for authentication-related events is the `secure` file. On Debian/Ubuntu-type systems, the main log file is the `syslog` file, and the authentication log is the `auth.log` file. Other log files you'll see include the following:

*   `/var/log/kern.log`: On Debian/Ubuntu-type systems, this log contains messages about what's going on with the Linux kernel. As we saw in *Chapter 4, Securing Your Server with a Firewall - Part 1*, and *Chapter 5, Securing Your Server with a Firewall - Part 2*, this includes messages about what's going on with the Linux firewall. So, if you want to see whether any suspicious network packets have been blocked, this is the place to look. Red Hat-type systems don't have this file. Instead, Red Hat systems send their kernel messages to the `messages` file.
*   `/var/log/wtmp` and `/var/run/utmp`: The two files do essentially the same thing. They both record information about users who are logged in to the system. The main difference is that `wtmp` holds historical data from `utmp`. Unlike most Linux log files, these are in binary format, rather than normal text-mode format. The `utmp` file is the only file we'll look at that isn't in the `/var/log/` directory.
*   `/var/log/btmp`: This binary file holds information about failed login attempts. The `pam_tally2` module that we looked at in *Chapter 3, Securing Normal User Accounts*, uses the information that's in this file.
*   `/var/log/lastlog`: This binary file holds information about the last time that users logged in to the system.
*   `/var/log/audit/audit.log`: This text-mode file records information from the auditd daemon. We already discussed it in *Chapter 12*, *Scanning, Hardening, and Auditing*, so I won't discuss it here.

There are quite a few other log files that contain information about applications and system boot-ups. But the log files that I've listed here are the main ones we're concerned about when looking at system security.

Now that we've looked at what log files we have, let's look at them in more detail.

### The system log and the authentication log

It doesn't matter whether you're talking about the `syslog` and `auth.log` files on Debian/Ubuntu or the `messages` and `secure` files on RHEL/CentOS/AlmaLinux. On any of these systems, the files are the same, just with different names. The system log files and the authentication log files have the same basic structure and are all plaintext files. This makes it easy to search for specific information with tools that are already built into Linux. It doesn't really matter which virtual machine we use for this, other than to keep the names of the files straight.

To begin, let's look at a simple message from the system log:

```
Jul  1 18:16:12 localhost systemd[1]: Started Postfix Mail Transport Agent.
```

Here's the breakdown:

*   `Jul 1 18:16:12`: This is the date and time that the message was generated.
*   `localhost`: This is the hostname of the machine that generated the message. This is important because one Linux machine can serve as the central log repository for other Linux machines. By default, messages from other machines will just get dumped into the same log file that the local machine uses. So, we need this field to let us know what's happening on each machine.
*   `systemd[1]`: This is the service that generated the message. In this case, it was the `systemd` daemon.
*   The rest of the line is the specific message.

There are several ways to extract information from the text-mode log files. For now, we'll just open the files in `less`, as in this example:

```
sudo less syslog
```

Then, to search for a specific text string, hit the **/** key, type in the string that you want to find, and hit Enter.

So, what kind of security-related information can we expect to find in these files? To start, let's look at the permissions on the server's private SSH keys:

```
donnie@orangepione:/etc/ssh$ ls -l
total 580
. . .
-rw-------+ 1 root root   1679 Feb 10  2019 ssh_host_rsa_key
-rw-r--r--  1 root root    398 Feb 10  2019 ssh_host_rsa_key.pub
donnie@orangepione:/etc/ssh$
```

This private key, the `ssh_host_rsa_key` file, has to have permissions set for only the root user. But, the `+` sign at the end of the permissions settings denotes that someone has set an **access-control list** (**ACL**) on that file. `getfacl` will show us exactly what's going on:

```
donnie@orangepione:/etc/ssh$ getfacl ssh_host_rsa_key
# file: ssh_host_rsa_key
# owner: root
# group: root
user::rw-
user:sshdnoroot:r--
group::---
mask::r--
other::---
donnie@orangepione:/etc/ssh$
```

So, someone has created the `sshdnoroot` user and assigned it the read permission for the server's private SSH keys. Now, if I try to restart the OpenSSH daemon, it will fail. A peek into the system log—the `syslog` file, in this case—will tell me why:

```
Mar 13 12:47:46 localhost sshd[1952]: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Mar 13 12:47:46 localhost sshd[1952]: @ WARNING: UNPROTECTED PRIVATE KEY FILE! @
Mar 13 12:47:46 localhost sshd[1952]: @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Mar 13 12:47:46 localhost sshd[1952]: Permissions 0640 for '/etc/ssh/ssh_host_rsa_key' are too open.
Mar 13 12:47:46 localhost sshd[1952]: It is required that your private key files are NOT accessible by others.
Mar 13 12:47:46 localhost sshd[1952]: This private key will be ignored.
Mar 13 12:47:46 localhost sshd[1952]: key_load_private: bad permissions
Mar 13 12:47:46 localhost sshd[1952]: Could not load host key: /etc/ssh/ssh_host_rsa_key
```

So, the SSH daemon won't start if someone other than the root user has any access permissions for the server's private keys. But how did this happen? Let's search through the authentication file—`auth.log`, in this case—to see if there's a clue:

```
Mar 13 12:42:54 localhost sudo:   donnie : TTY=tty1 ; PWD=/etc/ssh ; USER=root ; COMMAND=/usr/bin/setfacl -m u:sshdnoroot:r ssh_host_ecdsa_key ssh_host_ed25519_key ssh_host_rsa_key
```

Ah, so that `donnie` character did this. Why, this is an outrage! Fire that guy immediately! Oh wait, that's me. On second thought, let's not fire him. But seriously, this shows the value of forcing users to use `sudo` instead of allowing them to do everything from the root shell. If I had done this from the root shell, the authentication log would have shown where I logged in as the root user, but it wouldn't have shown anything I did as the root user. With `sudo`, every root-level action gets logged, along with who did it.

There are several ways to obtain specific information from the log files. These include:

*   Using the search feature of the `less` utility, as I mentioned earlier
*   Using `grep` to search for text strings through either one file or multiple files at once
*   Writing scripts in languages such as `bash`, Python, or `awk`

Here's an example of using `grep`:

```
sudo grep 'fail' syslog
```

In this case, I'm searching through the `syslog` file for all lines that contain the text string `fail`. By default, `grep` is case-sensitive, so this command won't find any instances of `fail` with uppercase letters. Also, by default, `grep` finds text strings that are embedded within other text strings. So, in addition to just finding `fail`, this command will also find `failed`, `failure`, or any other text string that contains the text string `fail`.

To make the search case-insensitive, add the `-i` option, like so:

```
sudo grep -i 'fail' syslog
```

This will find all forms of `fail` in either uppercase or lowercase letters. To only search for the `fail` text string, and to exclude where it's embedded in other text strings, use the `-w` option, like so:

```
sudo grep -w 'fail' syslog
```

You can combine the two options like this:

```
sudo grep -iw 'fail' syslog
```

In general, if you don't know exactly what you're looking for, start off with a more generic search that will probably show you too much. Then, narrow things down until you find what you want.

Now, this is all good when you just want to search through the log files for specific information. But it's rather tedious when you need to do your daily log review. Later on, I'll show you a tool that will make that much easier. For now, let's look at the binary log files.

### The utmp, wtmp, btmp, and lastlog files

Unlike the system log files and the authentication log files, all of these files are binary files. So, we can't use our normal text tools, such as `less` or `grep`, to read them or extract information from them. Instead, we'll use some special tools that can read these binary files.

The `w` and `who` commands pull information about who's logged in and what they're doing from the `/var/run/utmp` file. Both commands have their own option switches, but you likely won't ever need them. If you just want to see the list of users who are currently logged in, use `who` like so:

```
donnie@orangepione:/var/log$ who
donnie   tty7         2019-08-02 18:18 (:0)
donnie   pts/1        2019-11-21 16:21 (192.168.0.251)
donnie   pts/2        2019-11-21 17:01 (192.168.0.251)
katelyn  pts/3        2019-11-21 18:15 (192.168.0.11)
lionel   pts/4        2019-11-21 18:21 (192.168.0.15)
donnie@orangepione:/var/log$
```

It shows me with three different logins. The `tty7` line is my local terminal session, and the `pts/1` and `pts/2` lines are my two remote SSH sessions from the `192.168.0.251` machine. Katelyn and Lionel are remotely logged in from two other machines.

The `w` command shows you not only who's logged in, but also what they're doing:

```
donnie@orangepione:/var/log$ w
 18:29:42 up 2:09, 5 users, load average: 0.00, 0.00, 0.00
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
donnie tty7 :0 02Aug19 111days 6.28s 0.05s /bin/sh /etc/xdg/xfce4/xinitrc -- /etc/X11/xinit/xserverrc
donnie pts/1 192.168.0.251 16:21 4.00s 2.88s 0.05s w
donnie pts/2 192.168.0.251 17:01 7:10 0.81s 0.81s -bash
katelyn pts/3 192.168.0.11 18:15 7:41 0.64s 0.30s vim somefile.txt
lionel pts/4 192.168.0.15 18:21 8:06 0.76s 0.30s sshd: lionel [priv] 
donnie@orangepione:/var/log$
```

This shows five users, but there are really only three since it counts each of my login sessions as a separate user. The `:0` under the `FROM` column for my first login means that this login is at the machine's local console. The `/bin/sh` part shows that I have a terminal window open, and the `/etc/xdg/xfce4/xinitrc -- /etc/X11/xinit/xserverrc` stuff means that the machine is in graphical mode, with the XFCE desktop. The `pts/1` line shows that I've run the `w` command in that window, and the `pts/2` line shows that I'm not doing anything in that window, other than just having the bash shell open.

Next, we see that Katelyn is editing a file. So, I think that she's all good. But look at Lionel. The `[priv]` in his line indicates that he's doing some sort of privileged action. To see what that action is, we'll peek into the authentication file, where we see this:

```
Nov 21 18:21:42 localhost sudo:   lionel : TTY=pts/4 ; PWD=/home/lionel ; USER=root ; COMMAND=/usr/sbin/visudo
```

Oh, come now. What fool gave Lionel the privileges to use `visudo`? I mean, we know that Lionel isn't supposed to have that privilege. Well, we can investigate. Further up in the authentication file, we see this:

```
Nov 21 18:17:53 localhost sudo:   donnie : TTY=pts/2 ; PWD=/home/donnie ; USER=root ; COMMAND=/usr/sbin/visudo
```

This shows that that `donnie` character opened `visudo`, but it doesn't show what edits he made to it. But since this line comes soon after the line where `donnie` created Lionel's account, and no other users have used `visudo`, it's a safe bet that `donnie` is the one who gave Lionel that `visudo` privilege. So, we can surmise that that `donnie` character is a real loser who deserves to be fired. Oh, wait. That was me again, wasn't it? Okay, never mind.

In normal usage, the `last` command pulls information from the `/var/log/wtmp` file, which archives historical data from the `/var/run/utmp` file. Without any option switches, `last` shows when each user has logged in or out, and when the machine has been booted:

```
donnie@orangepione:/var/log$ last
lionel   pts/4        192.168.0.15     Thu Nov 21 18:21   still logged in
lionel   pts/4        192.168.0.15     Thu Nov 21 18:17 - 18:17  (00:00)
katelyn  pts/3        192.168.0.11     Thu Nov 21 18:15   still logged in
katelyn  pts/3        192.168.0.251    Thu Nov 21 18:02 - 18:15  (00:12)
donnie   pts/2        192.168.0.251    Thu Nov 21 17:01   still logged in
donnie   pts/1        192.168.0.251    Thu Nov 21 16:21   still logged in
donnie   tty7         :0               Fri Aug  2 18:18    gone - no logout
reboot   system boot  4.19.57-sunxi    Wed Dec 31 19:00   still running
. . .
wtmp begins Wed Dec 31 19:00:03 1969
donnie@orangepione:/var/log$
```

To show a list of failed login attempts, use the `-f` option to read the `/var/log/btmp` file. The catch is that this requires `sudo` privileges because we generally want to keep information about failed logins confidential:

```
donnie@orangepione:/var/log$ sudo last -f /var/log/btmp
[sudo] password for donnie: 
katelyn ssh:notty 192.168.0.251 Thu Nov 21 17:57 gone - no logout
katelyn ssh:notty 192.168.0.251 Thu Nov 21 17:57 - 17:57 (00:00)
katelyn ssh:notty 192.168.0.251 Thu Nov 21 17:57 - 17:57 (00:00)
btmp begins Thu Nov 21 17:57:35 2019
donnie@orangepione:/var/log$
```

Of course, we could see about Katelyn's three failed logins in the `auth.log` or `secure` file, but it's handier and quicker to see about them here.

Finally, there's the `lastlog` command, which pulls information from—you guessed it—the `/var/log/lastlog` file. This shows a record of all users on the machine, even system users, and when they logged in last:

```
donnie@orangepione:/var/log$ lastlog
Username         Port     From             Latest
root             tty1                      Tue Mar 12 15:29:09 -0400 2019
. . .
messagebus                                 **Never logged in**
sshd                                       **Never logged in**
donnie           pts/2    192.168.0.251    Thu Nov 21 17:01:03 -0500 2019
sshdnoroot                                 **Never logged in**
. . .
katelyn          pts/3    192.168.0.11     Thu Nov 21 18:15:44 -0500 2019
lionel           pts/4    192.168.0.15     Thu Nov 21 18:21:33 -0500 2019
donnie@orangepione:/var/log$
```

There are a lot more logs in the `/var/log/` directory, but I've just given you the quick tour of the logs that pertain to system security. Next, we'll look at the two major logging systems that are built into most Linux distros, starting with the `rsyslog` system.

## Understanding rsyslog

The old `syslog` logging system was created back in the 1980s for use on Unix and other Unix-like systems. It finally saw its last days in the Linux world only a few years ago. Nowadays, we use `rsyslog`, which is a bit more robust and has a few more features. It works mainly the same on both Debian/Ubuntu-based and Red Hat-based distros, with only some differences in how the configuration files are set up. But, before we look at the differences, let's look at what's the same.

### Understanding rsyslog logging rules

Logging rules define where to record messages for each particular system service:

*   On Red Hat/CentOS/AlmaLinux systems, the rules are stored in the `/etc/rsyslog.conf` file. Just scroll down until you see the `#### RULES ####` section.
*   On Debian/Ubuntu systems, the rules are in separate files in the `/etc/rsyslog.d/` directory. The main file that we care about for now is the `50-default.conf` file, which contains the main logging rules.

To explain the structure of an `rsyslog` rule, let's look at this example from an AlmaLinux machine:

```
authpriv.*           /var/log/secure
```

Here's the breakdown:

*   `authpriv`: This is the facility, which defines the type of message.
*   `.`: The dot separates the facility from the level, which is the next field.
*   `*`: This is the level, which indicates the importance of the message. In this case, we just have a wildcard, which means that all levels of the `authpriv` facility get logged.
*   `/var/log/secure`: This is the action, which is really the destination of this message. (I have no idea why someone decided to call this an action.)

When we put this all together, we see that `authpriv` messages of all levels will get sent to the `/var/log/secure` file.

Here's a handy list of the predefined `rsyslog` facilities:

*   `auth`: Messages generated by the authorization system (`login`, `su`, `sudo`, and so forth)
*   `authpriv`: Messages generated by the authorization system but which are only readable by selected users
*   `cron`: Messages generated by the `cron` daemon
*   `daemon`: Messages generated by all system daemons (for example, `sshd`, `ftpd`, and so forth)
*   `ftp`: Messages for `ftp`
*   `kern`: Messages generated by the Linux kernel
*   `lpr`: Messages generated by the line printer spooling
*   `mail`: Messages generated by the mail system
*   `mark`: Periodic timestamp message in the system log
*   `news`: Messages generated by network news system
*   `rsyslog`: Messages generated internally by `rsyslog`
*   `user`: Messages generated by users
*   `local0-7`: Custom messages for writing your own scripts

Here's a list of the different levels:

*   `none`: Disables logging for a facility
*   `debug`: Debug only
*   `info`: Information
*   `notice`: Issues to review
*   `warning`: Warning messages
*   `err`: Error conditions
*   `crit`: Critical conditions
*   `alert`: Urgent messages
*   `emerg`: Emergency

Except for the `debug` level, whatever level you set for a facility will cause messages of that level up through `emerg` to get logged. For example, when you set the `info` level, all messages of the `info` levels through `emerg` get logged. With that in mind, let's look at a more complex example of a logging rule, also from an AlmaLinux machine:

```
*.info;mail.none;authpriv.none;cron.none   /var/log/messages
```

Here's the breakdown:

*   `*.info`: This refers to messages from all facilities of the `info` level and higher.
*   `;`: This is a compound rule. The semicolons separate the different components of this rule from each other.
*   `mail.none;authpriv.none;cron.none`: These are the three exceptions to this rule. Messages from the `mail`, `authpriv`, and `cron` facilities will not get sent to the `/var/log/messages` file. These three facilities have their own rules for their own log files. (The `authpriv` rule that we just looked at earlier is one of them.)

The rules on an Ubuntu machine aren't exactly the same as the ones on an AlmaLinux machine. But, if you understand these examples, you won't have any trouble figuring out the Ubuntu rules.

If you ever make changes to the `rsyslog.conf` file or add any rules files to the `/etc/rsyslog.d/` directory, you'll need to restart the `rsyslog` daemon to read in the new configuration. Do that like this:

```
[donnie@localhost ~]$ sudo systemctl restart rsyslog
[sudo] password for donnie: 
[donnie@localhost ~]$
```

Now that you have a basic understanding of `rsyslog`, let's look at `journald`, which is the new kid in town.

## Understanding journald

You'll find the `journald` logging system on any Linux distro that uses the `systemd` ecosystem. Instead of sending its messages to text files, `journald` sends messages to binary files. Instead of using normal Linux text file utilities to extract information, you have to use the `journalctl` utility. At the time of this writing, I don’t know of any Linux distro that has made the complete transition to `journald`. Current Linux distros that use `systemd` run `journald` and `rsyslog` side by side. Currently, the default on RHEL-type systems is for `journald` log files to be temporary files that get erased every time you reboot the machine. (You can configure `journald` to make its log files persistent, but there's probably not much point as long as we still need to keep the old `rsyslog` files.) On Ubuntu, the default is for both `journald` and `rsyslogd` to maintain persistent log files.

On RHEL 8/9-type distros, `journald`, instead of `rsyslog`, is now what actually collects log messages from the rest of the operating system. But `rsyslog` is still there, collecting the messages from `journald` and sending them to the old-fashioned `rsyslog` text files. So, the way you do log file management hasn't really changed.

It will likely take a few more years to completely transition away from `rsyslog`. One reason is that third-party log aggregation and analysis utilities, such as LogStash, Splunk, and Nagios, are still set up to read text files instead of binary files. Another reason is that, at this point, using `journald` as a remote, central log server is still in a proof-of-concept stage that isn't ready for production use. So, for now, `journald` isn't a suitable substitute for `rsyslog`.

> Several years ago, the Fedora team released a version of Fedora that only used `journald`, and that left out `rsyslog`. Too many people complained about that, so they had to bring back `rsyslog` for the next version of Fedora.

To view the `journald` log file in its entirety, use the `journalctl` command. With Ubuntu, the person who installed the operating system has been added to the `adm` group, which allows that person to use `journalctl` without sudo or root privileges. Any users who are added later would only be able to see their own messages. In fact, here's what happened for Frank:

```
frank@ubuntu4:~$ journalctl
Hint: You are currently not seeing messages from other users and the system.
      Users in groups 'adm', 'systemd-journal' can see all messages.
      Pass -q to turn off this notice.
-- Logs begin at Tue 2019-11-26 17:43:28 UTC, end at Tue 2019-11-26 17:43:28 UTC. --
Nov 26 17:43:28 ubuntu4 systemd[10306]: Listening on GnuPG cryptographic agent and passphrase cache.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Reached target Timers.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Listening on GnuPG cryptographic agent and passphrase cache (restricted).
. . .
. . .
Nov 26 17:43:28 ubuntu4 systemd[10306]: Reached target Basic System.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Reached target Default.
Nov 26 17:43:28 ubuntu4 systemd[10306]: Startup finished in 143ms.
frank@ubuntu4:~$
```

To see messages from either the system or from other users, these new users would have to be added to either the `adm` or the `systemd-journal` group, or granted the proper sudo privileges. With RHEL/CentOS/AlmaLinux, no users are automatically added to either the `adm` or `systemd-journal` group. So, initially, only users who have sudo privileges can view the `journald` logs.

Doing either `journalctl` or `sudo journalctl`, as appropriate, automatically opens the log in the `less` pager. What you'll see looks pretty much the same as what you'd see in the normal `rsyslog` log files, with the following exceptions:

*   Long lines run past the right-hand edge of the screen. To see the rest of the lines, use the right cursor key.
*   You'll also see color-coding and highlighting to make different types of messages stand out. Messages of `ERROR` level and higher are in red, while messages from `NOTICE` level up to `ERROR` level are highlighted with bold characters.

There are lots of options that can display different types of information in various formats. For example, to only see messages about the SSH service on CentOS or AlmaLinux, use the `--unit` option, like so:

```
[donnie@localhost ~]$ sudo journalctl --unit=sshd
-- Logs begin at Tue 2019-11-26 12:00:13 EST, end at Tue 2019-11-26 15:55:19 EST. --
Nov 26 12:00:41 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Nov 26 12:00:42 localhost.localdomain sshd[825]: Server listening on 0.0.0.0 port 22.
Nov 26 12:00:42 localhost.localdomain sshd[825]: Server listening on :: port 22.
Nov 26 12:00:42 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Nov 26 12:22:08 localhost.localdomain sshd[3018]: Accepted password for donnie from 192.168.0.251 port 50797 ssh2
Nov 26 12:22:08 localhost.localdomain sshd[3018]: pam_unix(sshd:session): session opened for user donnie by (uid=0)
Nov 26 13:03:33 localhost.localdomain sshd[4253]: Accepted password for goldie from 192.168.0.251 port 50912 ssh2
Nov 26 13:03:34 localhost.localdomain sshd[4253]: pam_unix(sshd:session): session opened for user goldie by (uid=0)
[donnie@localhost ~]$
```

You can't use the grep utility with these binary logs, but you can search for a string with the `-g` option. By default, it's case-insensitive and finds your desired text string even when it's embedded in another text string. Here, we see it finding the text string, `fail`:

```
. . .
```

There are lots more options besides just these. To see them, just do:

```
man journalctl
```

Now that you've seen the basics of using both `rsyslog` and `journald`, let's look at a cool utility that can help to ease the pain of doing log reviews.

## Making things easier with Logwatch

You know how important it is to do a daily log review. But you also know how much of a drag it is, and that you'd rather take a severe beating. Fortunately, there are various utilities that can make the job easier. Of the various choices in the normal Linux distro repositories, Logwatch is my favorite.

Logwatch doesn't have the fancy bells and whistles that the third-party log aggregators have, but it's still quite good. Every morning, you'll find a summary of the previous day's logs delivered to your mail account. Depending on how your mail system is configured, you can have the summaries delivered to your user account on the local machine or to an email account that you can access from anywhere. It's as easy as can be to set up, so let's demonstrate with a hands-on lab.

### Hands-on lab – installing Logwatch

To deliver its messages, Logwatch requires that the machine also has a running mail server daemon. Depending on the options you chose when installing the operating system, you might or might not already have the Postfix mail server installed. When Postfix is set up as a local server, it will deliver system messages to the root user's local account.

To view the Logwatch summaries on the local machine, you'll also need to install a text-mode mail reader, such as mutt.

For this lab, you can use any of your VMs:

1.  Install Logwatch, mutt, and Postfix. (On Ubuntu, choose the `local` option when installing Postfix. With CentOS or AlmaLinux, the `local` option is already the default.) For Ubuntu, do this:

```
sudo apt install postfix mutt logwatch
```

For CentOS 7, do this:

```
sudo yum install postfix mutt logwatch
```

For AlmaLinux, do this:

```
sudo dnf install postfix mutt logwatch
```

1.  On Ubuntu only, create a mail spool file for your user account:

```
sudo touch /var/mail/your_user_name
```

1.  Open the `/etc/aliases` file in your favorite text editor. Configure it to forward the root user's mail to your own normal account by adding the following line at the bottom of the file:

```
root:     your_user_name
```

1.  Save the file, and then copy the information from it to a binary file that the system can read. Do that with this:

```
sudo newaliases
```

1.  At this point, you have a fully operational implementation of Logwatch that will deliver daily log summaries with a *low level* of detail. To see the default configuration, look at the default configuration file:

```
less /usr/share/logwatch/default.conf/logwatch.conf
```

1.  To change the configuration, edit the `/etc/logwatch/conf/logwatch.conf` file on CentOS and AlmaLinux, or create the file on Ubuntu. Change to a medium level of logging detail by adding this line:

```
Detail = Med
```

> Logwatch is a Python script that runs every night on a scheduled basis. So, there's no daemon that you have to restart to make configuration changes take effect.

1.  Perform some actions that will generate some log entries. You can do that by performing a system update, installing some software packages, and using `sudo fdisk -l` to view the partition configuration.
2.  If possible, allow your VM to run overnight. In the morning, view your log summary by doing this:

```
mutt
```

When prompted to create a `Mail` directory in your home directory, hit the *y* key.

1.  End of lab.

Now that you've seen the easy way of doing a log review, let's move on to the final topic of this chapter, which is how to set up a central log server.

## Setting up a remote log server

So far, we've just been dealing with log files on a local machine. But instead of having to log into each individual machine to review log files, wouldn't it be nice to just have all of the log files from every machine on just one server? Well, you can do that. The best part is that it's easy.

But convenience isn't the only reason to collect log files on one central server. There's also the matter of log file security. If we leave all log files on each individual host, it's easier for network intruders to find the files and modify them to delete any messages about their nefarious activities. (That's easy to do since most log files are just plaintext files that can be edited in a normal text editor.)

### Hands-on lab – setting up a basic log server

Setting up the server is identical on Ubuntu, CentOS, and AlmaLinux. There's only one minor difference in setting up the clients. For best results, ensure that the server VM and the client VM each have a different hostname:

1.  On the log-collecting server VM, open the `/etc/rsyslog.conf` file in your favorite text editor and look for these lines, which are near the top of the file:

```
# Provides TCP syslog reception
#module(load="imtcp") # needs to be done just once
#input(type="imtcp" port="514")
```

1.  Uncomment the bottom two lines and save the file. The stanza should now look like this:

```
# Provides TCP syslog reception
module(load="imtcp") # needs to be done just once
input(type="imtcp" port="514")
```

1.  Restart the `rsyslog` daemon:

```
sudo systemctl restart rsyslog
```

1.  If the machine has an active firewall, open port `514/tcp`.
2.  Next, configure the client machines. For Ubuntu, add the following line to the bottom of the `/etc/rsyslog.conf` file, substituting the IP address of your own server VM:

```
@@192.168.0.161:514
```

For CentOS and AlmaLinux, look for this stanza at the bottom of the `/etc/rsyslog.conf` file:

```
# ### sample forwarding rule ###
#action(type="omfwd"  
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
#queue.filename="fwdRule1"       # unique name prefix for spool files
#queue.maxdiskspace="1g"         # 1gb space limit (use as much as possible)
#queue.saveonshutdown="on"       # save messages to disk on shutdown
#queue.type="LinkedList"         # run asynchronously
#action.resumeRetryCount="-1"    # infinite retries if host is down
# Remote Logging (we use TCP for reliable delivery)
# remote_host is: name/ip, e.g. 192.168.0.1, port optional e.g. 10514
#Target="remote_host" Port="XXX" Protocol="tcp"
```

Remove the comment symbols from each line that isn't obviously a real comment. Add the IP address and port number for the log server VM. The finished product should look like this:

```
# ### sample forwarding rule ###
action(type="omfwd"
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
queue.filename="fwdRule1"       # unique name prefix for spool files
queue.maxdiskspace="1g"         # 1gb space limit (use as much as possible)
queue.saveonshutdown="on"       # save messages to disk on shutdown
queue.type="LinkedList"         # run asynchronously
action.resumeRetryCount="-1"    # infinite retries if host is down
# Remote Logging (we use TCP for reliable delivery)
# remote_host is: name/ip, e.g. 192.168.0.1, port optional e.g. 10514
Target="192.168.0.161" Port="514" Protocol="tcp")
```

1.  Save the file and then restart the `rsyslog` daemon.
2.  On the server VM, verify that messages from both the server VM and the client VM are getting sent to the log files. (You can tell by the different hostnames for different messages.)
3.  This is the end of the lab.

As cool as this is, there are still a couple of flaws with the setup. One is that we're using a non-encrypted, plaintext connection to send the log files to the server. Let's fix that.

### Creating an encrypted connection to the log server

We'll use the `stunnel` package to create our encrypted connection. It's easy, except that the procedures for Ubuntu and AlmaLinux are different. These differences are:

*   With AlmaLinux 8/9, FIPS modules are available free of charge, as I showed you in *Chapter 6*, *Encryption Technologies*. They're not available for CentOS 7, and they're only available for Ubuntu if you're willing to purchase a support contract. So, for now, the only way we can take advantage of FIPS mode in `stunnel` is to set it up on either AlmaLinux 8/9 or some other RHEL 8/9 clone.
*   On AlmaLinux, `stunnel` runs as a `systemd` service. On Ubuntu, for some bizarre reason, it's still set up to run with an old-fashioned `init` script. So, we have to deal with two different methods of controlling the `stunnel` daemon.

Let's begin with the AlmaLinux procedure.

#### Creating a stunnel connection on AlmaLinux 9 – server side

For this lab, we're using an AlmaLinux 9 VM that's been set to run in FIPS-compliant mode (see the steps for that in *Chapter 6*, *Encryption Technologies*):

1.  On an AlmaLinux VM, install `stunnel`:

```
sudo dnf install stunnel
```

1.  On the server, within the `/etc/stunnel/` directory, create a new `stunnel.conf` file with the following contents:

```
cert=/etc/stunnel/stunnel.pem
fips=yes
[hear from client]
accept=30000
connect=127.0.0.1:6514
```

1.  On the server, while still within the `/etc/stunnel/` directory, create the `stunnel.pem` certificate file:

```
sudo openssl req -new -x509 -days 3650 -nodes -out stunnel.pem -keyout stunnel.pem
```

1.  On the server, open port `30000` on the firewall, and close port `514`:

```
sudo firewall-cmd --permanent --add-port=30000/tcp
sudo firewall-cmd --permanent --remove-port=514/tcp
sudo firewall-cmd --reload
```

> Port `6514`, which you see in the `stunnel.conf` file, is strictly for internal communication between `rsyslog` and `stunnel`. So, for that, we don't need to open a firewall port. We're configuring `stunnel` to listen on port `30000` on behalf of `rsyslog`, so we no longer need to have port `514` open on the firewall.

1.  Enable and start the `stunnel` daemon by doing this:

```
sudo systemctl enable --now stunnel
```

1.  In the `/etc/rsyslog.conf` file, look for this line at the top of the file:

```
input(type="imtcp" port="514")
```

Change it to this:

```
input(type="imtcp" port="6514")
```

1.  After saving the file, restart `rsyslog`:

```
sudo systemctl restart rsyslog
```

1.  The server is now ready to receive log files from remote clients via an encrypted connection.

Next, we'll configure an AlmaLinux VM to send its logs to this server.

#### Creating an stunnel connection on AlmaLinux – client side

In this procedure, we'll configure an AlmaLinux machine to send its logs to the log server (it doesn't matter whether the log server is running on CentOS, AlmaLinux, or Ubuntu):

1.  Install `stunnel`:

```
sudo dnf install stunnel
```

1.  In the `/etc/stunnel/` directory, create the `stunnel.conf` file with the following contents:

```
client=yes
fips=yes
[speak to server]
accept=127.0.0.1:6514
connect=192.168.0.161:30000
```

> In the `connect` line, substitute the IP address of your own log server for the one you see here.

1.  Enable and start the `stunnel` daemon:

```
sudo systemctl enable --now stunnel
```

1.  At the bottom of the `/etc/rsyslog.conf` file, look for this line:

```
Target="192.168.0.161" Port="514" Protocol="tcp")
```

Change it to this:

```
Target="127.0.0.1" Port="6514" Protocol="tcp")
```

1.  After saving the file, restart the `rsyslog` daemon:

```
sudo systemctl restart rsyslog
```

1.  On the client, use `logger` to send a message to the log file:

```
logger "This is a test of the stunnel setup."
```

1.  On the server, verify that the message got added to the `/var/log/messages` file.
2.  This is the end of the lab.

Let's now turn our attention to Ubuntu.

#### Creating a stunnel connection on Ubuntu – server side

For this, we’ll use an Ubuntu 22.04 VM. I don’t understand why, but Ubuntu still uses an old-style `init` script for `stunnel`, instead of a `systemd` service. So, the commands that you’ll use for this will be different than what you’re used to using.

1.  Install `stunnel`:

```
sudo apt install stunnel
```

1.  In the `/etc/stunnel/` directory, create the `stunnel.conf` file with the following contents:

```
cert=/etc/stunnel/stunnel.pem
fips=no
[hear from client]
accept=30000
connect=6514
```

1.  While still in the `/etc/stunnel/` directory, create the `stunnel.pem` certificate:

```
sudo openssl req -new -x509 -days 3650 -nodes -out stunnel.pem -keyout stunnel.pem
```

1.  Start the `stunnel` daemon:

```
sudo service stunnel4 start
```

1.  To make it automatically start when you reboot the system, create a cron job for the root user. First, open the crotab editor, like this:

```
sudo crontab -e -u root
```

Add this line to the end of the file:

```
@reboot service stunnel4 start
```

1.  In the `/etc/rsyslog.conf` file, look for this line at the top:

```
input(type="imtcp" port="514")
```

Change it to this:

```
input(type="imtcp" port="6514")
```

1.  After saving the file, restart the `rsyslog` daemon:

```
sudo systemctl restart rsyslog
```

1.  Using the appropriate `iptables`, `ufw`, or `nftables` command, open port `30000/tcp` on the firewall, and close port `514`.
2.  This is the end of the lab.

Next, we'll configure the client.

#### Creating a stunnel connection on Ubuntu – client side

Using this procedure on an Ubuntu client will allow it to send its files to either an AlmaLinux or an Ubuntu log server:

1.  Install `stunnel`:

```
sudo apt install stunnel
```

1.  In the `/etc/stunnel/` directory, create the `stunnel.conf` file with the following contents:

```
client=yes
fips=no
[speak to server]
accept = 127.0.0.1:6514
connect=192.168.0.161:30000
```

> Note that even though we can't use FIPS mode on the Ubuntu clients, we can still have them send log files to an AlmaLinux log server that is configured to use FIPS mode. (So, yes, we can mix and match.)

1.  Start the `stunnel` daemon:

```
sudo service stunnel4 start
```

1.  To make it automatically start when you reboot the system, create a cron job. Open the crontab editor by doing:

```
sudo crontab -e -u root
```

Add this line to the end of the file:

```
@reboot service stunnel4 start
```

1.  At the bottom of the `/etc/rsyslog.conf` file, look for the line that has the IP address of the log server. Change it to this:

```
@@127.0.0.1:6514
```

1.  After saving the file, restart the `rsyslog` daemon:

```
sudo systemctl restart rsyslog
```

1.  Use `logger` to send a message to the log server:

```
 logger "This is a test of the stunnel connection."
```

1.  On the server, verify that the message is in the `/var/log/messages` or `/var/log/syslog` file, as appropriate.
2.  End of lab.

Okay, we now have a secure connection, which is a good thing. But the messages from all of the clients still get jumbled up in the server's own log files. Let's fix that.

### Separating client messages into their own files

This is something else that's easy-peasy. We'll just make a couple of simple edits to the `rsyslog` rules on the log server and restart the `rsyslog` daemon. For our demo, I'll use the AlmaLinux 9 VM.

> **Important**
> 
> > You won’t be able to use Logwatch if you implement this trick. Well, you actually can, except that Logwatch will just take all of the events from all of the client files and jumble them up into one big summary. So, you won’t be able to see which client machines generate the events.

In the RULES section of the `/etc/rsyslog.conf` file, look for this line:

```
*.info;mail.none;authpriv.none;cron.none   /var/log/messages
```

Change it to this:

```
*.info;mail.none;authpriv.none;cron.none ?Rmessages
```

Above that line, insert this line:

```
$template Rmessages,"/var/log/%HOSTNAME%/messages"
```

Do likewise for the `auth` messages:

```
# authpriv.* /var/log/secure
$template Rauth,"/var/log/%HOSTNAME%/sec
auth.*,authpriv.* ?Rauth
```

Finally, restart `rsyslog`:

```
sudo systemctl restart rsyslog
```

Look in the `/var/log/` directory, and you’ll see directories for each of the clients that are sending logs to this server. Pretty slick, eh?

> Tip
> 
> > The trick here is to always have a `$template` line *precede* the affected rule.

And that wraps it up for another chapter. You now know about what to look for in log files, how to make log reviews easier, and how to set up a secure remote log server.

## Summary

In this chapter, we looked at the different types of log files, with an emphasis on files that contain security-related information. Then, we looked at the basic operation of the `rsyslog` and `journald` logging systems. To make log reviews a bit easier, we introduced Logwatch, which automatically creates a summary of the preceding day's log files. We wrapped things up by setting up a central, remote log server that collects log files from other network hosts.

In the next chapter, we'll look at how to do vulnerability scanning and intrusion detection. I'll see you there.

## Questions

1.  Which two of the following are log files that record authentication-related events?
    1.  `syslog`
    2.  `authentication.log`
    3.  `auth.log`
    4.  `secure.log`
    5.  `secure`
2.  Which log file contains the current record about who is logged into the system and what they're doing?
    1.  `/var/log/syslog`
    2.  `/var/log/utmp`
    3.  `/var/log/btmp`
    4.  `/var/run/utmp`
3.  Which of the following is the main logging system that runs on pretty much every modern Linux distro?
    1.  `syslog`
    2.  `rsyslog`
    3.  `journald`
    4.  `syslog-ng`
4.  Which of the following is peculiar to RHEL 8/9 and their offspring, such as AlmaLinux 8/9?
    1.  On RHEL 8/9 systems, `journald` collects log data from the rest of the system and sends it to `rsyslog`.
    2.  On RHEL 8/9 systems, `journald` has completely replaced `rsyslog`.
    3.  On RHEL 8/9 systems, `rsyslog` collects data from the rest of the system and sends it to `journald`.
    4.  RHEL 8/9 systems use `syslog-ng`.
5.  Which of the following is a consideration when setting up `stunnel`?
    1.  On AlmaLinux systems, FIPS mode is not available.
    2.  On Ubuntu systems, FIPS mode is not available.
    3.  On Ubuntu systems, FIPS mode is available, but only if you purchase a support contract.
    4.  On AlmaLinux 8/9, FIPS mode is available, but only if you purchase a support contract.
6.  Which of the following two statements are true about `stunnel`?
    1.  On RHEL systems, `stunnel` runs as a normal `systemd` service.
    2.  On RHEL systems, `stunnel` still runs under an old-fashioned `init` script.
    3.  On Ubuntu systems, `stunnel` runs as a normal `systemd` service.
    4.  On Ubuntu systems, `stunnel` runs under an old-fashioned `init` script.
7.  Which file must you edit to have the root user's messages forwarded to your own user account?
8.  After you edit the file that's referenced in *Question 7*, which command must you run to transfer the information to a binary file that the system can read?
9.  To create an `stunnel` setup for your remote log server, you must create a security certificate for both the server and for each client.
    1.  True
    2.  False
10.  Which of the following commands would you use to find the `fail` text string in `journald` log files?
    1.  `sudo grep fail /var/log/journal/messages`
    2.  `sudo journalctl -g fail`
    3.  `sudo journalctl -f fail`
    4.  `sudo less /var/log/journal/messages`

## Further reading

*   Five open source log management programs: [https://fosspost.org/lists/open-source-log-management](https://fosspost.org/lists/open-source-log-management)
*   *What Is a SIEM?*: [https://www.tripwire.com/state-of-security/incident-detection/log-management-siem/what-is-a-siem/](https://www.tripwire.com/state-of-security/incident-detection/log-management-siem/what-is-a-siem/)
*   *12 Critical Linux Log Files You Must be Monitoring*: [https://www.eurovps.com/blog/important-linux-log-files-you-must-be-monitoring/](https://www.eurovps.com/blog/important-linux-log-files-you-must-be-monitoring/)
*   *Analyzing Linux Logs*: [https://www.loggly.com/ultimate-guide/analyzing-linux-logs/](https://www.loggly.com/ultimate-guide/analyzing-linux-logs/)
*   Linux log files with examples: [https://www.poftut.com/linux-log-files-varlog/](https://www.poftut.com/linux-log-files-varlog/)
*   The `rsyslog` home page: [https://www.rsyslog.com/](https://www.rsyslog.com/)
*   *Why Journald?*: [https://www.loggly.com/blog/why-journald/](https://www.loggly.com/blog/why-journald/)
*   Journalctl cheat sheet: [https://www.golinuxcloud.com/view-logs-using-journalctl-filter-journald/](https://www.golinuxcloud.com/view-logs-using-journalctl-filter-journald/)
*   *The Linux Administration Cookbook*, by Adam K. Dean: [https://www.packtpub.com/virtualization-and-cloud/linux-administration-cookbook](https://www.packtpub.com/virtualization-and-cloud/linux-administration-cookbook)
*   The Logwatch project page: [https://sourceforge.net/projects/logwatch/](https://sourceforge.net/projects/logwatch/)
*   The `stunnel` home page : [https://www.stunnel.org/](https://www.stunnel.org/)
*   *Linux Service Management Made Easy with systemd*, by Donald A. Tevault: [https://www.packtpub.com/product/linux-service-management-made-easy-with-systemd/](https://www.packtpub.com/product/linux-service-management-made-easy-with-systemd/)

## Answers

1.  c, e
2.  d
3.  b
4.  a
5.  c
6.  a, d
7.  `/etc/aliases`
8.  `sudo newaliases`
9.  b
10.  c