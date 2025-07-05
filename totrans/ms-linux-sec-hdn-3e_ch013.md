# 12 Scanning, Auditing, and Hardening

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file74.png)

A common misconception is that Linux users never need to worry about malware. Yes, it's true that Linux is much more resistant to viruses than Windows is. But viruses are only one type of malware, and other types of malware can be planted on Linux machines. And, if you're running a server that will share files with Windows users, you'll want to make sure that you don't share any virus-infected files with them.

While Linux system log files are nice, they don't always give a good picture of who does what or who accesses what. It could be that either intruders or insiders are trying to access data that they're not authorized to access. What we really want is a good auditing system to alert us when people do things that they're not supposed to do.

And then there's the issue of regulatory compliance. Your organization may have to deal with one or more regulatory bodies that dictate how you harden your servers against attacks. If you're not in compliance, you could be fined or put out of business.

Fortunately, we have ways to deal with all of these issues, and they aren't all that complicated.

In this chapter, we'll cover the following topics:

*   Installing and updating ClamAV and maldet
*   Scanning with ClamAV and maldet
*   SELinux considerations
*   Scanning for rootkits with Rootkit Hunter
*   Performing quick malware analysis with strings and VirusTotal
*   Controlling the `auditd` daemon
*   Creating audit rules
*   Using the `ausearch` and `aureport` utilities to search the audit logs for problems
*   Using `inotifywait` for quick and easy auditing
*   `oscap`, the command-line utility for managing and applying OpenSCAP policies
*   OpenSCAP Workbench, the GUI utility for managing and applying OpenSCAP policies
*   OpenSCAP policy files and the compliance standards that each of them is designed to meet
*   Applying a policy during operating system installation

If you're ready, let's begin by looking at a Linux-based virus scanning solution.

## Installing and updating ClamAV and maldet

```
Although we don't have to worry much about viruses infecting our Linux machines, we do need to worry about sharing infected files with Windows users. ClamAV is a Free Open Source Software (FOSS) antivirus solution that is available for Linux, Windows, and macOS. The included freshclam utility allows you to update virus signatures.
Linux Malware Detect, which you'll often see abbreviated as either LMD or maldet, is another FOSS antivirus program that can work alongside ClamAV. (To save typing, I'll just refer to it as either LMD or maldet from now on.) As far as I know, it's not available in the repositories of any Linux distro, but it's still simple enough to install and configure. One of its features is that it automatically generates malware detection signatures when it sees malware on the network's edge intrusion detection systems. End users can also submit their own malware samples. When you install it, you'll get a systemd service that's already enabled and a cron job that will periodically update both the malware signatures and the program itself. It works with the Linux kernel's inotify capability to automatically monitor directories for files that have changed.
```

> You can get all the nitty-gritty details about LMD at [https://www.rfxn.com/projects/linux-malware-detect/](https://www.rfxn.com/projects/linux-malware-detect/).

The reason that we're installing ClamAV and LMD together is that, as the LMD folk freely admit, the ClamAV scan engine gives much better performance when scanning large file sets. Also, by having them both together, ClamAV can use the LMD malware signatures as well as its own malware signatures.

> Just to be clear...
> 
> > Viruses are a real problem for computers that run the Windows operating system. But, as far as anyone has been able to tell, there's no such thing as a virus that can harm a Linux-based operating system. So, the only real reason to run an antivirus solution on a Linux machine is to prevent infecting any Windows machines on your network. This means that you don't need to worry about installing an antivirus product on your Linux-based DNS servers, DHCP servers, and so forth. But, if you have a Linux-based email server, Samba server, download server, or any other Linux-based machine that shares files with Windows computers, then installing an antivirus solution is a good idea.

All right, so much for the theory. Let's get our hands dirty, shall we?

### Hands-on lab – installing ClamAV and maldet

We'll begin by installing ClamAV. (It's in the normal repository for Ubuntu, but not for CentOS or AlmaLinux. For CentOS and AlmaLinux, you'll need to install the EPEL repository, as I showed you in *Chapter 1*, *Running Linux in a Virtual Environment*.) We'll also install `wget`, which we'll use to download LMD. For this lab, you can use Ubuntu, CentOS 7, or either version of AlmaLinux. Let's get started:

*   The following command will install ClamAV, `inotify-tools` and `wget` on Ubuntu:

```
donnie@ubuntu3:~$ sudo apt install clamav wget inotify-tools
```

The following command will install ClamAV, `inotify-tools`, and `wget` on CentOS 7:

```
[donnie@localhost ~]$ sudo yum install clamav clamav-update wget inotify-tools
```

For AlmaLinux 8 or AlmaLinux 9, do this:

```
[donnie@localhost ~]$ sudo dnf install clamav clamav-update wget inotify-tools
[donnie@localhost ~]$ sudo systemctl enable --now clamav-freshclam
```

Note that if you chose the **Minimal** installation option when creating a CentOS or an AlmaLinux virtual machine (VM), you may also have to install the `perl` and the `tar` packages.

For Ubuntu, the `clamav` package contains everything you need. For CentOS or AlmaLinux, you'll need to also install `clamav-update` in order to obtain virus updates.

The rest of the steps will be the same for either VM.

1.  Next, you'll download and install LMD.

Here, you'll want to do something that I rarely tell people to do. That is, you'll want to log in to the root user shell. The reason is that, although the LMD installer works fine with `sudo`, you'll end up with the program files being owned by the user who performed the installation, instead of by the root user. Performing the installation from the root user's shell saves us the trouble of tracking down those files and changing the ownership. So, download the file as follows:

```
sudo su -
wget http://www.rfxn.com/downloads/maldetect-current.tar.gz
```

Now, you'll have the file in the root user's home directory.

1.  Extract the archive and enter the resultant directory:

```
tar xzvf maldetect-current.tar.gz
cd maldetect-1.6.4/
```

*   Run the installer. Once the installer finishes, copy the `README` file to your own home directory so that you can have it for ready reference. (This `README` file is the documentation for LMD.) Then, exit from the root user's shell back to your own shell:

```
root@ubuntu3:~/maldetect-1.6.4# ./install.sh
Created symlink from /etc/systemd/system/multi-user.target.wants/maldet.service to /usr/lib/systemd/system/maldet.service.
update-rc.d: error: initscript does not exist: /etc/init.d/maldet
. . .
. . .
maldet(22138): {sigup} signature set update completed
maldet(22138): {sigup} 15218 signatures (12485 MD5 | 1954 HEX | 779 YARA | 0 USER)
root@ubuntu3:~/maldetect-1.6.4# cp README /home/donnie
root@ubuntu3:~/maldetect-1.6.4# exit
logout
donnie@ubuntu3:~$
```

As you see, the installer automatically creates the symbolic link that enables the `maldet` service, and it also automatically downloads and installs the newest malware signatures.

1.  For CentOS or AlmaLinux, the `maldet.service` file that the installer copied to the `/lib/systemd/system/` directory has the wrong SELinux context, which will prevent `maldet` from starting. Correct the SELinux context like this:

```
sudo restorecon /lib/systemd/system/maldet.service
```

You've reached the end of the lab – congratulations!

### Hands-on lab – configuring maldet

In previous versions, maldet was configured by default to automatically monitor and scan users' home directories. In its current version, the default is for it to only monitor the `/dev/shm/`, `/var/tmp/`, and `/tmp/` directories. We're going to reconfigure it so that we can add some directories. Let's get started:

1.  Open the `/usr/local/maldetect/conf.maldet` file for editing. Find these two lines:

```
default_monitor_mode="users"
# default_monitor_mode="/usr/local/maldetect/monitor_paths"
```

Change them to look like this:

```
# default_monitor_mode="users"
default_monitor_mode="/usr/local/maldetect/monitor_paths"
```

1.  At the top of the file, enable email alerts and set your username as the email address. The two lines should now look something like this:

```
email_alert="1"
email_addr="donnie"
```

1.  LMD isn't already configured to move suspicious files to the `quarantine` folder, and we want to make it do that. Further down in the `conf.maldet` file, look for the line that says:

```
quarantine_hits="0"
```

1.  Change it to this:

```
quarantine_hits="1"
```

> You'll see a few other quarantine actions that you can configure, but, for now, this is all we need.

*   Save the `conf.maldet` file, because that's all the changes that we need to make to it.
*   Open the `/usr/local/maldetect/monitor_paths` file for editing. Add the directories that you want to monitor, like this:

```
/var/tmp
/tmp
/home
/root
```

> Since viruses affect Windows and not Linux, just monitor the directories with files that will be shared with Windows machines.

*   After you save the file, start the `maldet` daemon:

```
sudo systemctl start maldet
```

> You can add more directories to the `monitor_paths` file at any time, but remember to restart the `maldet` daemon any time that you do, in order to read in the new additions.

You've reached the end of the lab – congratulations!

Now, let's talk about keeping ClamAV and maldet updated.

### Updating ClamAV and maldet

The good news for busy admins is that you don't have to do anything to keep either of these updated. To verify that they are getting updated, we can look in the system log file:

```
Dec 8 20:02:09 localhost freshclam[22326]: ClamAV update process started at Fri Dec 8 20:02:09 2017
Dec 8 20:02:29 localhost freshclam[22326]: Can't query current.cvd.clamav.net
Dec 8 20:02:29 localhost freshclam[22326]: Invalid DNS reply. Falling back to HTTP mode.
Dec 8 20:02:29 localhost freshclam[22326]: Reading CVD header (main.cvd):
Dec 8 20:02:35 localhost freshclam[22326]: OK
Dec 8 20:02:47 localhost freshclam[22326]: Downloading main-58.cdiff [100%]
Dec 8 20:03:19 localhost freshclam[22326]: main.cld updated (version: 58, sigs: 4566249, f-level: 60, builder: sigmgr)
. . .
. . .
Dec 8 20:04:45 localhost freshclam[22326]: Downloading daily.cvd [100%]
. . .
. . .
```

You'll see these same entries in either the Ubuntu logs, the CentOS logs, or the AlmaLinux logs. However, there is a difference between how the updates get run automatically.

Look in the `/lib/systemd/system/` directory of your Ubuntu or AlmaLinux VMs, and you’ll see the `clamav-freshclam.service` file:

```
[donnie@localhost ~]$ cd /lib/systemd/system
[donnie@localhost system]$ ls -l clamav-freshclam.service 
-rw-r--r--. 1 root root 389 Nov  7 06:51 clamav-freshclam.service
[donnie@localhost system]$
```

This service is automatically enabled and started on Ubuntu, but you’ll need to enable and start it yourself on AlmaLinux, like so:

```
[donnie@localhost ~]$ sudo systemctl enable --now clamav-freshclam
Created symlink /etc/systemd/system/multi-user.target.wants/clamav-freshclam.service → /usr/lib/systemd/system/clamav-freshclam.service.
[donnie@localhost ~]$
```

Without a `freshclam.conf` configuration file, AlmaLinux just runs the update service every two hours. Ubuntu, on the other hand, uses the `/etc/clamav/freshclam.conf` file to change the update interval to every hour, as you see at the bottom of the file:

```
# Check for new database 24 times a day
Checks 24
DatabaseMirror db.local.clamav.net
DatabaseMirror database.clamav.net
```

> If you have the crypto policy set to `FUTURE` mode on your AlmaLinux 8/9 machine, the ClamAV database update won’t work. That’s because the ClamAV site uses a security certificate that’s not compatible with `FUTURE` mode. So, if you want to run ClamAV on any type of RHEL 8 or 9 machine, you’ll just have to leave the crypto policy set to `DEFAULT` mode.

On your CentOS 7 machine, you'll see a `clamav-update` `cron` job in the `/etc/cron.d/` directory that looks like this:

```
## Adjust this line...
MAILTO=root
## It is ok to execute it as root; freshclam drops privileges and becomes
## user 'clamupdate' as soon as possible
0  */3 * * * root /usr/share/clamav/freshclam-sleep
```

The `*/3` in the second column from the left indicates that ClamAV will check for updates every three hours. You can change that if you like, but you'll also need to change the setting in the `/etc/sysconfig/freshclam` file.

Let's say that you want CentOS 7 to also check for ClamAV updates every hour. In the `cron` job file, change `*/3` to `*`. (You don't need to do `*/1` because the asterisk by itself in that position already indicates that the job will run every hour.) Then, in the `/etc/sysconfig/freshclam` file, look for this line:

```
# FRESHCLAM_MOD=
```

Uncomment that line and add the number of minutes that you want between updates. To set it to 1 hour, so that it matches the `cron` job, it will look like this:

```
FRESHCLAM_MOD=60
```

> A disabled `clamav-freshclam.service` file also gets installed on CentOS 7\. If you’d rather use the service instead of the `cron` job, just delete the `/etc/cron.d/clamav-update` file, and then enable the `clamav-freshclam` service.

To prove that `maldet` is being updated, you can look in its own log files in the `/usr/local/maldetect/logs/` directory. In the `event_log` file, you'll see these messages:

```
Dec 06 22:06:14 localhost maldet(3728): {sigup} performing signature update check...
Dec 06 22:06:14 localhost maldet(3728): {sigup} local signature set is version 2017070716978
Dec 06 22:07:13 localhost maldet(3728): {sigup} downloaded https://cdn.rfxn.com/downloads/maldet.sigs.ver
Dec 06 22:07:13 localhost maldet(3728): {sigup} new signature set (201708255569) available
Dec 06 22:07:13 localhost maldet(3728): {sigup} downloading https://cdn.rfxn.com/downloads/maldet-sigpack.tgz
. . .
. . .
Dec 06 22:07:43 localhost maldet(3728): {sigup} unpacked and installed maldet-clean.tgz
Dec 06 22:07:43 localhost maldet(3728): {sigup} signature set update completed
Dec 06 22:07:43 localhost maldet(3728): {sigup} 15218 signatures (12485 MD5 | 1954 HEX | 779 YARA | 0 USER)
Dec 06 22:14:55 localhost maldet(4070): {scan} signatures loaded: 15218 (12485 MD5 | 1954 HEX | 779 YARA | 0 USER)
```

In the `/usr/local/maldetect/conf.maldet` file, you'll see these two lines, but with some comments in between them:

```
autoupdate_signatures="1"
autoupdate_version="1"
```

Not only will LMD automatically update its malware signatures, but it will also ensure that you have the latest version of LMD itself.

## Scanning with ClamAV and maldet

LMD's maldet daemon constantly monitors the directories that you specify in the `/usr/local/maldetect/monitor_paths` file. When it finds a suspicious file, it will perform the action that you specified in the `conf.maldet` file.

```
You can test your setup by downloading a simulated virus file from the European Institute for Computer Antivirus Research (EICAR) site.
```

> There are four different simulated virus files that you can download from [https://www.eicar.org/download-anti-malware-testfile/](https://www.eicar.org/download-anti-malware-testfile/). Note that if you’re running a Windows host machine, these files could get flagged by the Windows antivirus. So, your best bet is to download the files directly to your Linux virtual machines.

Just download one or all of the EICAR test files and transfer them to your home directory on the virtual machines. Your best bet is to download the files directly to your virtual machines, with these four commands:

```
wget https://secure.eicar.org/eicar.com
wget https://secure.eicar.org/eicar.com.txt
wget https://secure.eicar.org/eicar_com.zip
wget https://secure.eicar.org/eicarcom2.zip
```

Wait just a few moments, and you should see the files disappear. Then, look in the `/usr/local/maldetect/logs/event_log` file to verify that the LMD moved the files to quarantine:

```
Dec 01 15:18:31 localhost maldet(6388): {hit} malware hit {HEX}EICAR.TEST.3 found for /home/donnie/eicar.com.txt
Dec 01 15:18:31 localhost maldet(6388): {quar} malware quarantined from '/home/donnie/eicar.com.txt' to '/usr/local/maldetect/quarantine/eicar.com.txt.113345162'
Dec 01 15:18:31 localhost maldet(6388): {mon} scanned 5 new/changed files with clamav engine
Dec 01 15:20:32 localhost maldet(6388): {mon} warning clamd service not running; force-set monitor mode file scanning to every 120s
. . .
. . .
Dec 01 15:20:56 localhost maldet(6388): {quar} malware quarantined from '/home/donnie/eicar_com.zip' to '/usr/local/maldetect/quarantine/eicar_com.zip.216978719'
```

> Ignore the `warning clamd service not running` messages, because we don’t need to use that service.

There's still a bit more to LMD than what I can show you here. However, you can read all about it in the `README` file that comes with it.

### SELinux considerations

It used to be that doing an antivirus scan on a Red Hat-type system would trigger an SELinux alert. But, in the course of proofing this chapter, the scans all worked as they should, and SELinux never bothered me once.

If you ever do generate any SELinux alerts with your virus scans, all you need to do to fix it is to change one Boolean:

```
[donnie@localhost ~]$ getsebool -a | grep 'virus'
antivirus_can_scan_system --> off
antivirus_use_jit --> off
[donnie@localhost ~]$
```

What interests us here is the `antivirus_can_scan_system` Boolean, which is off by default. To turn it on to enable virus scans, just do this:

```
[donnie@localhost ~]$ sudo setsebool -P antivirus_can_scan_system on
[sudo] password for donnie:
[donnie@localhost ~]$ getsebool antivirus_can_scan_system
antivirus_can_scan_system --> on
[donnie@localhost ~]$
```

That should fix any SELinux-related scan problems that you may have. But, as things stand now, you probably won't need to worry about it.

## Scanning for rootkits with Rootkit Hunter

Rootkits are exceedingly nasty pieces of malware that can definitely ruin your day. They can listen for commands from their masters, steal sensitive data and send it to their masters, or provide an easy-access back door for their masters. They're designed to be stealthy, with the ability to hide themselves from plain view. Sometimes, they'll replace utilities such as `ls` or `ps` with their own trojaned versions that will show all files or processes on the system except for the ones that are associated with the rootkit. Rootkits can infect any operating system, even our beloved Linux.

In order to plant a rootkit, an attacker has to have already gained administrative privileges on a system. This is one of the many reasons why I always cringe when I see people doing all of their work from the root user's shell and why I'm a firm advocate of using `sudo` whenever possible. I mean, really, why should we make it easy for the bad guys?

> Several years ago, back in the dark days of Windows XP, Sony Music got into a bit of trouble when someone discovered that they had planted a rootkit on their music CDs. They didn't mean to do anything malicious, but only wanted to stop people from using their computers to make illegal copies. Of course, most people ran Windows XP with an administrator account, which made it really easy for the rootkit to infect their computers. Windows users still mostly run with administrator accounts, but they at least now have User Access Control to help mitigate these types of problems.

There are a couple of different programs that scan for rootkits, and both are used pretty much the same way. One is called **Rootkit Hunter**, and the other is called **chkrootkit**. Now, understand that I’m showing you these programs because as a Linux administrator, you’ll be expected to know about them. In reality, they’re not very useful, because there are a whole lot of rootkits that neither of them will detect. If you really want to prove that, just go to Github and do a keyword search for *rootkit*. Find a rootkit that will run on Linux, download the source code to a virtual machine, and then follow the included directions for how to compile and install it. Once it’s installed, do a scan with either one of the rootkit scan programs. Most likely, the rootkit won’t get detected. Also, don’t expect AppArmor or SELinux to prevent someone from installing a rootkit, because they won’t.

> Not every rootkit on Github will compile correctly for you, so finding ones that work will involve a bit of trial-and-error. One that I did get to compile and install correctly is the Reptile rootkit, which you can download from here: [https://github.com/f0rb1dd3n/Reptile](https://github.com/f0rb1dd3n/Reptile)

Okay, let's move on to the lab.

### Hands-on lab – installing and updating Rootkit Hunter

For Ubuntu, Rootkit Hunter is in the normal repository. For CentOS or AlmaLinux, you'll need to install the EPEL repository, as I showed you how to do in *Chapter 1*, *Running Linux in a Virtual Environment*. For all of these Linux distros, the package name is `rkhunter`. Let's get started:

1.  Use one of the following commands to install Rootkit Hunter, as appropriate. For Ubuntu, do this:

```
sudo apt install rkhunter
```

For CentOS 7, do the following:

```
sudo yum install rkhunter
```

For AlmaLinux 8 or AlmaLinux 9, do this:

```
sudo dnf install rkhunter
```

1.  After it's been installed, you can look at its options with this command:

```
man rkhunter
```

*   Next, update the rootkit signatures using the `--update` option:

```
[donnie@localhost ~]$ sudo rkhunter --update
[ Rootkit Hunter version 1.4.4 ]
Checking rkhunter data files...
 Checking file mirrors.dat [ Updated ]
 Checking file programs_bad.dat [ Updated ]
 Checking file backdoorports.dat [ No update ]
 Checking file suspscan.dat [ Updated ]
 Checking file i18n/cn [ No update ]
 Checking file i18n/de [ Updated ]
 Checking file i18n/en [ Updated ]
 Checking file i18n/tr [ Updated ]
 Checking file i18n/tr.utf8 [ Updated ]
 Checking file i18n/zh [ Updated ]
 Checking file i18n/zh.utf8 [ Updated ]
 Checking file i18n/ja [ Updated ]
```

1.  Now, we're ready to scan.

You've reached the end of the lab – congratulations!

### Scanning for rootkits

To run your scan, use the `-c` option. (That's `-c` for *check*.) Be patient, because it will take a while:

```
sudo rkhunter -c
```

When you run the scan in this manner, Rootkit Hunter will periodically stop and ask you to hit the Enter key to continue. When the scan completes, you'll find a `rkhunter.log` file in the `/var/log/` directory.

To have Rootkit Hunter automatically run as a `cron` job, use the `--cronjob` option, which will cause the program to run all the way through without prompting you to keep hitting the Enter key. You might also want to use the `--rwo` option, which will cause the program to only report warnings, instead of also reporting on everything that's good. From the command line, the command would look like this:

```
sudo rkhunter -c --cronjob --rwo
```

To create a `cron` job that will automatically run Rootkit Hunter on a nightly basis, open the `crontab` editor for the root user:

```
sudo crontab -e -u root
```

Let's say that you want to run Rootkit Hunter every night at 20 minutes past 10\. Enter this into the `crontab` editor:

```
20 22 * * * /usr/bin/rkhunter -c --cronjob --rwo
```

Since `cron` only works with 24-hour clock time, you'll have to express 10:00 P.M. as 22\. (Just add 12 to the normal P.M. clock times that you're used to using.) The three asterisks mean that the job will run every day of the month, every month of the year, and every day of the week, respectively. You'll need to list the entire path for the command. Otherwise, `cron` won't be able to find it.

You'll find more options that might interest you in the `rkhunter` man page, but this should be enough to get you going with it.

> A few moments ago, I told you that these rootkit scanner programs aren’t very effective, because there are many rootkits that they won’t detect. That’s why the best way to deal with rootkits is to prevent them from getting installed in the first place. So, be sure to keep your systems locked down to prevent malicious actors from gaining root privileges.

Next, let's look at a couple of quick techniques for analyzing malware.

## Performing a quick malware analysis with strings and VirusTotal

Malware analysis is one of those advanced topics that I can't cover in detail here. However, I can show you a couple of quick ways to analyze a suspicious file.

### Analyze a file with strings

Executable files often have strings of text embedded in them. You can use the `strings` utility to look at those strings. (Yeah, that makes sense, right?) Depending on your distro, `strings` might or might not already be installed. It's already on CentOS and AlmaLinux, but to get it on Ubuntu, you'll need to install the `binutils` package, like so:

```
sudo apt install binutils
```

As an example, let's look at a `Your File Is Ready To Download_2285169994.exe` file that was automatically downloaded from a cryptocoin faucet site. To examine the file, I’ll do this:

```
strings "Your File Is Ready To Download_2285169994.exe" > output.txt
vim output.txt
```

I saved the output to a text file that I can open in `vim` so that I can view the line numbers. To see the line numbers, I entered `:set number` at the bottom of the `vim` screen. (In `vim` parlance, we're using the last line mode.)

It's hard to say exactly what to search for, so you'll just need to browse through until you see something interesting. In this case, look at what I've found starting at line `386`:

```
386 The Setup program accepts optional command line parameters.
387 /HELP, /?
388 Shows this information.
389 /SP-
390 Disables the This will install... Do you wish to continue? prompt at the beginning of Setup.
391 /SILENT, /VERYSILENT
392 Instructs Setup to be silent or very silent.
393 /SUPPRESSMSGBOXES
394 Instructs Setup to suppress message boxes.
. . .
399 /NOCANCEL
400 Prevents the user from cancelling during the installation process.
. . .
```

It's saying that the installation process of this program can be made to run in `SILENT` mode, without popping up any dialog boxes. It can also be made to run in such a way that the user can't cancel the installation. Of course, the line at the top says that these are `optional command line parameters`. But, are they really optional, or are they hard coded in as the default? It's not clear, but in my mind, any installer that can be made to run in `SILENT` mode and that can't be canceled looks a bit suspicious, even if we're talking about `optional` parameters.

> Okay, so you're probably wondering, *What is a cryptocoin faucet?* Well, it's a website where you can go to claim a small amount of cryptocoin, such as Bitcoin, Ethereum, or Monero, in exchange for viewing the advertising and solving some sort of CAPTCHA puzzle. Most faucet operators are honest, but the advertising they allow on their sites often isn't and is often laden with malware, scams, and Not-Safe-For-Work images.

Now, this little trick works fine sometimes, but not always. More sophisticated malware might not contain any text strings that can give you any type of a clue. So, let's look at another little quick trick for malware analysis.

### Scanning the malware with VirusTotal

**VirusTotal** is a website where you can upload suspicious files for analysis. It uses a multitude of various virus scanners, so if one scanner misses something, another is likely to find it. Here are the results of scanning the `Your File Is Ready To Download_2285169994.exe` file:

![Figure 12.1: The VirusTotal scanner](img/file75.png)

Figure 12.1: The VirusTotal scanner

Here, we see that different virus scanners classify this file in different ways. But whether it's classified as `Win.Malware.Installcore`, `Trojan.InstallCore`, or whatever else, it's still bad.

> As good as VirusTotal is, you'll want to use it with caution. Don't upload any files that contain sensitive or confidential information, because it will get shared with other people.

So, what is this particular piece of malware all about? Well, it's actually a fake Adobe Flash installer. Of course, you don't want to test that by installing it on a production Windows machine. But, if you have a Windows VM handy, you can test the malware on it. (Either make a snapshot of the VM before you begin or be prepared to trash the VM afterward.)

As I said at the beginning, malware analysis is quite an in-depth topic and there are lots of more sophisticated programs to use for it. However, if you have suspicions about something and need to just do a quick check, these two techniques might be all you need.

Next, let's look at how to automatically audit the system for different events.

## Understanding the auditd daemon

So, you have a directory full of super-secret files that only a very few people need to see, and you want to know when unauthorized people try to see them. Or, maybe you want to see when a certain file gets changed, or you want to see when people log into the system and what they're doing once they do log in. For all this and more, you have the `auditd` system. It's a really cool system, and I think that you'll like it.

> One of the beauties of `auditd` is that it works at the Linux kernel level, rather than at the user-mode level. This makes it much harder for attackers to subvert.

On Red Hat-type systems, `auditd` comes installed and enabled by default. So, you'll find it already there on your CentOS and AlmaLinux machines. On Ubuntu, it won't be installed, so you'll have to do it yourself:

```
sudo apt install auditd
```

On Ubuntu, you can control the `auditd` daemon with the normal `systemctl` commands. So, if you need to restart `auditd` to read in a new configuration, you can do that with:

```
sudo systemctl restart auditd
```

On RHEL-type machines, `auditd` is configured to not work with the normal `systemctl` commands. (For all other daemons, they do.) So, on your CentOS and AlmaLinux machines, you'll restart the `auditd` daemon with the old-fashioned `service` command, like so:

```
sudo service auditd restart
```

Other than this minor difference, everything I tell you about `auditd` from here on will apply to all of your virtual machines.

### Creating audit rules

Okay, let's start with something simple and work our way up to something awesome. First, let's check to see whether any audit rules are in effect:

```
[donnie@localhost ~]$ sudo auditctl -l
[sudo] password for donnie:
No rules
[donnie@localhost ~]$
```

As you see, the `auditctl` command is what we use to manage audit rules. The `-l` option lists the rules.

### Auditing a file for changes

Now, let's say that we want to see when someone changes the `/etc/passwd` file. (The command that we'll use will look a bit daunting, but I promise that it will make sense once we break it down.) Here goes:

```
[donnie@localhost ~]$ sudo auditctl -w /etc/passwd -p wa -k passwd_changes
[sudo] password for donnie:
[donnie@localhost ~]$ sudo auditctl -l
-w /etc/passwd -p wa -k passwd_changes
[donnie@localhost ~]$
```

Here's the breakdown:

*   `-w`: This stands for where, and it points to the object that we want to monitor. In this case, it's `/etc/passwd`.
*   `-p`: This indicates the object's permissions that we want to monitor. In this case, we're monitoring to see when anyone either tries to (w)rite to the file or tries to make (a)ttribute changes. (The other two permissions that we can audit are (r)ead and e(x)ecute.)
*   `-k`: The `k` stands for key, which is just `auditd`'s way of assigning a name to a rule. So, `passwd_changes` is the key, or the name, of the rule that we're creating.

The `auditctl -l` command shows us that the rule is indeed there.

Now, the slight problem with this is that the rule is only temporary and will disappear when we reboot the machine. To make it permanent, we need to create a custom `rules` file in the `/etc/audit/rules.d/` directory. Then, when you restart the `auditd` daemon, the custom rules will be inserted into the `/etc/audit/audit.rules` file. Because the `/etc/audit/` directory can only be accessed by someone with root privileges, I'll just open the file by listing the entire path to the file, rather than trying to enter the directory:

```
sudo less /etc/audit/audit.rules
```

There's not a whole lot in this default file:

```
## This file is automatically generated from /etc/audit/rules.d
-D
-b 8192
-f 1
```

Here's the breakdown for this file:

*   `-D`: This will cause all rules and watches that are currently in effect to be deleted so that we can start from a clean slate. So, if I were to restart the `auditd` daemon right now, it would read this `audit.rules` file, which would delete the rule that I just created.
*   `-b 8192`: This sets the number of outstanding audit buffers that we can have going at one time. If all of the buffers get full, the system can't generate any more audit messages.
*   `-f 1`: This sets the failure mode for critical errors, and the value can be either `0`, `1`, or `2`. `-f 0` would set the mode to silent, meaning that `auditd` wouldn't do anything about critical errors. `-f 1`, as we see here, tells `auditd` to only report the critical errors, while `-f 2` would cause the Linux kernel to go into panic mode. According to the `auditctl` man page, anyone in a high-security environment would likely want to change this to `-f 2`. For our purposes, though, `-f1` works.

You could use your text editor to create a new `rules` file in the `/etc/audit/rules.d/` directory. Alternatively, you could just redirect the `auditctl -l` output into a new file, like this:

```
[donnie@localhost ~]$ sudo sh -c "auditctl -l > /etc/audit/rules.d/custom.rules"
[donnie@localhost ~]$ sudo service auditd restart
```

On Ubuntu:

sudo systemctl restart auditd

Since the Bash shell doesn't allow me to directly redirect information into a file in the `/etc/` directory, even with `sudo`, I have to use the `sudo sh -c` command in order to execute the `auditctl` command. After restarting the `auditd` daemon, our `audit.rules` file now looks like this:

```
## This file is automatically generated from /etc/audit/rules.d
-D
-b 8192
-f 1
-w /etc/passwd -p wa -k passwd_changes
```

Now, the rule will take effect every time the machine is rebooted, and every time that you manually restart the `auditd` daemon.

### Auditing a directory

Vicky and Cleopatra, my solid gray kitty and my gray-and-white tabby kitty, have some super sensitive secrets that they need to safeguard. So, I created the `secretcats` group and added them to it. Then, I created the `secretcats` shared directory and set the access controls on it, as I showed you how to do in *Chapter 9, Access Control Lists and Shared Directory Management*:

```
[donnie@localhost ~]$ sudo groupadd secretcats
[sudo] password for donnie:
[donnie@localhost ~]$ sudo usermod -a -G secretcats vicky
[donnie@localhost ~]$ sudo usermod -a -G secretcats cleopatra
[donnie@localhost ~]$ sudo mkdir /secretcats
[donnie@localhost ~]$ sudo chown nobody:secretcats /secretcats/
[donnie@localhost ~]$ sudo chmod 3770 /secretcats/
[donnie@localhost ~]$ ls -ld /secretcats/
drwxrws--T. 2 nobody secretcats 6 Dec 11 14:47 /secretcats/
[donnie@localhost ~]$
```

Vicky and Cleopatra want to be absolutely sure that nobody gets into their stuff, so they requested that I set up an auditing rule for their directory:

```
[donnie@localhost ~]$ sudo auditctl -w /secretcats/ -k secretcats_watch
[sudo] password for donnie:
[donnie@localhost ~]$ sudo auditctl -l
-w /etc/passwd -p wa -k passwd_changes
-w /secretcats -p rwxa -k secretcats_watch
[donnie@localhost ~]$
```

As before, the `-w` option denotes what we want to monitor, while the `-k` option denotes the name of the audit rule. This time, I left out the `-p` option because I want to monitor for every type of access. In other words, I want to monitor for any read, write, attribute change, or execute actions. (Because this is a directory, the execute action happens when somebody tries to `cd` into the directory.) You can see in the `auditctl -l` output that by leaving out the `-p` option, we will now monitor for everything. However, let's say that I only want to monitor for when someone tries to `cd` into this directory. I could have made the rule look like this:

```
sudo auditctl -w /secretcats/ -p x -k secretcats_watch
```

Easy enough so far, right?

> Plan carefully when you create your own custom audit rules. Auditing more files and directories than you need to can have a bit of a performance impact and could drown you in excessive information. Just audit what you really need to audit, as called for by either the scenario or what any applicable governing bodies require.

Now, let's look at something a bit more complex.

### Auditing system calls

Creating rules to monitor when someone performs a certain action isn't hard, but the command syntax is a bit trickier than what we've seen so far. With this rule, we're going to be alerted every time Charlie either tries to open a file or tries to create a file:

```
[donnie@localhost ~]$ sudo auditctl -a always,exit -F arch=b64 -S openat -F auid=1006
[sudo] password for donnie:
[donnie@localhost ~]$ sudo auditctl -l
-w /etc/passwd -p wa -k passwd_changes
-w /secretcats -p rwxa -k secretcats_watch
-a always,exit -F arch=b64 -S openat -F auid=1006
[donnie@localhost ~]$
```

Here's the breakdown:

*   `-a always,exit`: Here, we have the action and the list. The `exit` part means that this rule will be added to the system call `exit` list. Whenever the operating system exits from a system call, the `exit` list will be used to determine if an audit event needs to be generated. The `always` part is the action, which means that an audit record for this rule will always be created on exit from the specified system call. Note that the action and list parameters have to be separated by a comma.
*   `-F arch=b64`: The `-F` option is used to build a rule field, and we can see two rule fields in this command. This first rule field specifies the machine's CPU architecture. `b64` means that the computer is running with an x86_64 CPU. (Whether it's Intel or AMD doesn't matter.) Considering that 32-bit machines are dying off and that Sun SPARC and PowerPC machines aren't all that common, `b64` is what you'll now mostly see.
*   `-S openat`: The `-S` option specifies the system call that we want to monitor. `openat` is the system call that either opens or creates a file.
*   `-F auid=1006`: This second audit field specifies the user ID number of the user that we want to monitor. (Charlie's user ID number is `1006`.)

> A complete explanation of system calls, or syscalls, is a bit too esoteric for our present purpose. For now, suffice it to say that a syscall happens whenever a user issues a command that requests that the Linux kernel provide a service. If you're so inclined, you can read more about syscalls at [https://blog.packagecloud.io/eng/2016/04/05/the-definitive-guide-to-linux-system-calls/](https://blog.packagecloud.io/eng/2016/04/05/the-definitive-guide-to-linux-system-calls/).

What I've presented here are just a few of the many things that you can do with auditing rules. To see more examples, check out the `auditctl` man page:

```
man auditctl
```

So, now you're wondering, *Now that I have these rules, how do I know when someone tries to violate them?* As always, I'm glad that you asked.

## Using ausearch and aureport

The `auditd` daemon logs events to the `/var/log/audit/audit.log` file. Although you could directly read the file with something such as `less`, you really don't want to. The `ausearch` and `aureport` utilities will help you translate the file into a language that makes some sort of sense.

### Searching for file change alerts

Let's start by looking at the rule that we created that will alert us whenever a change is made to the `/etc/passwd` file:

```
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
```

Now, let's make a change to the file and look for the alert message. Rather than add another user, since I'm running out of cats whose names I can use, I'll just use the `chfn` utility to add contact information to the comment field for Cleopatra's entry:

```
[donnie@localhost etc]$ sudo chfn cleopatra
Changing finger information for cleopatra.
Name []: Cleopatra Tabby Cat
Office []: Donnie's back yard
Office Phone []: 555-5555
Home Phone []: 555-5556
Finger information changed.
[donnie@localhost etc]
```

Now, I'll use `ausearch` to look for any audit messages that this event may have generated:

```
[donnie@localhost ~]$ sudo ausearch -i -k passwd_changes
----
type=CONFIG_CHANGE msg=audit(12/11/2017 13:06:20.665:11393) : auid=donnie ses=842 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 op=add_rule key=passwd_changes li
st=exit res=yes
----
type=CONFIG_CHANGE msg=audit(12/11/2017 13:49:15.262:11511) : auid=donnie ses=842 op=updated_rules path=/etc/passwd key=passwd_changes list=exit res=yes
[donnie@localhost ~]$
```

Here's the breakdown:

*   `-i`: This takes any numeric data and, whenever possible, converts it into text. In this case, it takes user ID numbers and converts them into the actual username, which shows up here as `auid=donnie`. If I were to leave the `-i` out, the user information would show up as `auid=1000`, which is my user ID number.
*   `-k passwd_changes`: This specifies the key, or the name, of the audit rule for which we want to see the audit messages.

Here, you see that there are two parts to this output. The first part just shows when I created the audit rule, so we're not interested in that. In the second part, you can see when I triggered the rule, but it doesn't show how I triggered it. So, let's use `aureport` to see whether it will give us a bit more detail:

```
[donnie@localhost ~]$ sudo aureport -i -k | grep 'passwd_changes'
1\. 12/11/2017 13:06:20 passwd_changes yes ? donnie 11393
2\. 12/11/2017 13:49:15 passwd_changes yes ? donnie 11511
3\. 12/11/2017 13:49:15 passwd_changes yes /usr/bin/chfn donnie 11512
4\. 12/11/2017 14:54:11 passwd_changes yes /usr/sbin/usermod donnie 11728
5\. 12/11/2017 14:54:25 passwd_changes yes /usr/sbin/usermod donnie 11736
[donnie@localhost ~]$
```

What's curious is that with `ausearch`, you have to specify the name, or key, of the audit rule that interests you after the `-k` option. With `aureport`, the `-k` option means that you want to look at all log entries that have to do with all audit rule keys. To see log entries for a specific key, just pipe the output into `grep`. The `-i` option does the same thing that it does for `ausearch`.

As you see, `aureport` parses the cryptic language of the `audit.log` file into plain language that's easier to understand. I wasn't sure about what I had done to generate events 1 and 2, so I looked in the `/var/log/secure` file to see whether I could find out. I saw these two entries for those times:

```
Dec 11 13:06:20 localhost sudo: donnie : TTY=pts/1 ; PWD=/home/donnie ; USER=root ; COMMAND=/sbin/auditctl -w /etc/passwd -p wa -k passwd_changes
. . .
. . .
Dec 11 13:49:24 localhost sudo: donnie : TTY=pts/1 ; PWD=/home/donnie ; USER=root ; COMMAND=/sbin/ausearch -i -k passwd_changes
```

So, event 1 was from when I initially created the audit rule, and event 2 happened when I did an `ausearch` operation.

I must confess that the events in lines *4* and *5* are a bit of a mystery. Both were created when I invoked the `usermod` command, and both of them correlate to the secure log entries where I added Vicky and Cleopatra to the `secretcats` group:

```
Dec 11 14:54:11 localhost sudo:  donnie : TTY=pts/1 ; PWD=/home/donnie ; USER=root ; COMMAND=/sbin/usermod -a -G secretcats vicky
Dec 11 14:54:11 localhost usermod[14865]: add 'vicky' to group 'secretcats'
Dec 11 14:54:11 localhost usermod[14865]: add 'vicky' to shadow group 'secretcats'
Dec 11 14:54:25 localhost sudo:  donnie : TTY=pts/1 ; PWD=/home/donnie ; USER=root ; COMMAND=/sbin/usermod -a -G secretcats cleopatra
Dec 11 14:54:25 localhost usermod[14871]: add 'cleopatra' to group 'secretcats'
Dec 11 14:54:25 localhost usermod[14871]: add 'cleopatra' to shadow group 'secretcats'
```

The strange part about this is that adding a user to a secondary group doesn't modify the `passwd` file. So, I really don't know why the rule was triggered to create the events in lines *4* and *5*.

This leaves us with the event in line *3*, which is where I used `chfn` to actually modify the `passwd` file. Here's the `secure` log entry for that:

```
Dec 11 13:48:49 localhost sudo:  donnie : TTY=pts/1 ; PWD=/etc ; USER=root ; COMMAND=/bin/chfn cleopatra
```

So, out of all of these events, the one in line *3* is the only one where the `/etc/passwd` file was actually modified.

> The `/var/log/secure` file that I keep mentioning here is on Red Hat-type operating systems, such as CentOS and AlmaLinux. On your Ubuntu machine, you'll see the `/var/log/auth.log` file instead.

### Searching for directory access rule violations

For our next scenario, we’ll create a shared directory for Vicky and Cleopatra and then create an audit rule for it that looks like this:

```
sudo auditctl -w /secretcats/ -k secretcats_watch
```

So, all access or attempted access to this directory should trigger an alert. First, let's have Vicky enter the `/secretcats/` directory and run an `ls -l` command:

```
[vicky@localhost ~]$ cd /secretcats
[vicky@localhost secretcats]$ ls -l
total 4
-rw-rw-r--. 1 cleopatra secretcats 31 Dec 12 11:49 cleopatrafile.txt
[vicky@localhost secretcats]$
```

Here, we see that Cleopatra has already been there and has created a file. (We'll get back to that in a moment.) When an event triggers an `auditd` rule, it often creates multiple records in the `/var/log/audit/audit.log` file. If you look through each record for an event, you'll see that each one covers a different aspect of that event. When I do an `ausearch` command, I see a total of five records just from that one `ls -l` operation. For the sake of saving space, I'll just put the first one here:

```
sudo ausearch -i -k secretcats_watch | less
type=PROCTITLE msg=audit(12/12/2017 12:15:35.447:14077) : proctitle=ls --color=auto -l
type=PATH msg=audit(12/12/2017 12:15:35.447:14077) : item=0 name=. inode=33583041 dev=fd:01 mode=dir,sgid,sticky,770 ouid=nobody ogid=secretcats rdev=00:00 obj=unconfined_u:object_r:default_t:s0 objtype=NORMAL
type=CWD msg=audit(12/12/2017 12:15:35.447:14077) :  cwd=/secretcats
type=SYSCALL msg=audit(12/12/2017 12:15:35.447:14077) : arch=x86_64 syscall=openat success=yes exit=3 a0=0xffffffffffffff9c a1=0x2300330 a2=O_RDONLY|O_NONBLOCK|O_DIRECTORY|O_CLOEXEC a3=0x0 items=1 ppid=10805 pid=10952 auid=vicky uid=vicky gid=vicky euid=vicky suid=vicky fsuid=vicky egid=vicky sgid=vicky fsgid=vicky tty=pts0 ses=1789 comm=ls exe=/usr/bin/ls subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=secretcats_watch
```

I'll put the last one here:

```
type=PROCTITLE msg=audit(12/12/2017 12:15:35.447:14081) : proctitle=ls --color=auto -l
type=PATH msg=audit(12/12/2017 12:15:35.447:14081) : item=0 name=cleopatrafile.txt inode=33583071 dev=fd:01 mode=file,664 ouid=cleopatra ogid=secretcats rdev=00:00 obj=unconfined_u:object_r:default_t:s0 objtype=NORMAL
type=CWD msg=audit(12/12/2017 12:15:35.447:14081) :  cwd=/secretcats
type=SYSCALL msg=audit(12/12/2017 12:15:35.447:14081) : arch=x86_64 syscall=getxattr success=no exit=ENODATA(No data available) a0=0x7fff7c266e60 a1=0x7f0a61cb9db0 a2=0x0 a3=0x0 items=1 ppid=10805 pid=10952 auid=vicky uid=vicky gid=vicky euid=vicky suid=vicky fsuid=vicky egid=vicky sgid=vicky fsgid=vicky tty=pts0 ses=1789 comm=ls exe=/usr/bin/ls subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=secretcats_watch
```

In both records, you see the action that was taken (`ls -l`) and information about the person – or cat, in this case – that took the action. Since this is a RHEL-type machine, you also see SELinux context information. In the second record, you can also see the name of the file that Vicky saw when she did the `ls` command.

Next, let's say that that sneaky Charlie guy logs in and tries to get into the `/secretcats/` directory:

```
[charlie@localhost ~]$ cd /secretcats
-bash: cd: /secretcats: Permission denied
[charlie@localhost ~]$ ls -l /secretcats
ls: cannot open directory /secretcats: Permission denied
[charlie@localhost ~]$
```

Charlie isn't a member of the `secretcats` group and doesn't have permission to go into the `secretcats` directory. So, he should trigger an alert message. Well, he actually triggered one that consists of four records, and I'll again just list the first one and the last one. Here's the first one:

```
sudo ausearch -i -k secretcats_watch | less
type=PROCTITLE msg=audit(12/12/2017 12:32:04.341:14152) : proctitle=ls --color=auto -l /secretcats
type=PATH msg=audit(12/12/2017 12:32:04.341:14152) : item=0 name=/secretcats inode=33583041 dev=fd:01 mode=dir,sgid,sticky,770 ouid=nobody ogid=secretcats rdev=00:00 obj=unconfined_u:object_r:default_t:s0 objtype=NORMAL
type=CWD msg=audit(12/12/2017 12:32:04.341:14152) :  cwd=/home/charlie
type=SYSCALL msg=audit(12/12/2017 12:32:04.341:14152) : arch=x86_64 syscall=lgetxattr success=yes exit=35 a0=0x7ffd8d18f7dd a1=0x7f2496858f8a a2=0x12bca30 a3=0xff items=1 ppid=11637 pid=11663 auid=charlie uid=charlie gid=charlie euid=charlie suid=charlie fsuid=charlie egid=charlie sgid=charlie fsgid=charlie tty=pts0 ses=1794 comm=ls exe=/usr/bin/ls subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=secretcats_watch
```

Here's the last one:

```
type=PROCTITLE msg=audit(12/12/2017 12:32:04.341:14155) : proctitle=ls --color=auto -l /secretcats
type=PATH msg=audit(12/12/2017 12:32:04.341:14155) : item=0 name=/secretcats inode=33583041 dev=fd:01 mode=dir,sgid,sticky,770 ouid=nobody ogid=secretcats rdev=00:00 obj=unconfined_u:object_r:default_t:s0 objtype=NORMAL
type=CWD msg=audit(12/12/2017 12:32:04.341:14155) :  cwd=/home/charlie
type=SYSCALL msg=audit(12/12/2017 12:32:04.341:14155) : arch=x86_64 syscall=openat success=no exit=EACCES(Permission denied) a0=0xffffffffffffff9c a1=0x12be300 a2=O_RDONLY|O_NONBLOCK|O_DIRECTORY|O_CLOEXEC a3=0x0 items=1 ppid=11637 pid=11663 auid=charlie uid=charlie gid=charlie euid=charlie suid=charlie fsuid=charlie egid=charlie sgid=charlie fsgid=charlie tty=pts0 ses=1794 comm=ls exe=/usr/bin/ls subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=secretcats_watch
```

There are two things to note here. First, just attempting to `cd` into the directory doesn't trigger an alert. However, using `ls` to try to read the contents of the directory does. Secondly, note the `Permission denied` message that shows up in the second record.

The last set of alerts that we'll look at were created when Cleopatra created her `cleopatrafile.txt` file. This event triggered an alert that consists of 30 records. I'll just show you two of them, with the first one here:

```
type=PROCTITLE msg=audit(12/12/2017 11:49:37.536:13856) : proctitle=vim cleopatrafile.txt
type=PATH msg=audit(12/12/2017 11:49:37.536:13856) : item=0 name=. inode=33583041 dev=fd:01 mode=dir,sgid,sticky,770 ouid=nobody ogid=secretcats rdev=00:00 obj=unconfined_u:o
bject_r:default_t:s0 objtype=NORMAL
type=CWD msg=audit(12/12/2017 11:49:37.536:13856) :  cwd=/secretcats
type=SYSCALL msg=audit(12/12/2017 11:49:37.536:13856) : arch=x86_64 syscall=open success=yes exit=4 a0=0x5ab983 a1=O_RDONLY a2=0x0 a3=0x63 items=1 ppid=9572 pid=9593 auid=cle
opatra uid=cleopatra gid=cleopatra euid=cleopatra suid=cleopatra fsuid=cleopatra egid=cleopatra sgid=cleopatra fsgid=cleopatra tty=pts0 ses=1779 comm=vim exe=/usr/bin/vim sub
j=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=secretcats_watch
```

Here’s the second one:

```
type=PROCTITLE msg=audit(12/12/2017 11:49:56.001:13858) : proctitle=vim cleopatrafile.txt
type=PATH msg=audit(12/12/2017 11:49:56.001:13858) : item=1 name=/secretcats/.cleopatrafile.txt.swp inode=33583065 dev=fd:01 mode=file,600 ouid=cleopatra ogid=secretcats rdev
=00:00 obj=unconfined_u:object_r:default_t:s0 objtype=DELETE
type=PATH msg=audit(12/12/2017 11:49:56.001:13858) : item=0 name=/secretcats/ inode=33583041 dev=fd:01 mode=dir,sgid,sticky,770 ouid=nobody ogid=secretcats rdev=00:00 obj=unc
onfined_u:object_r:default_t:s0 objtype=PARENT
type=CWD msg=audit(12/12/2017 11:49:56.001:13858) :  cwd=/secretcats
type=SYSCALL msg=audit(12/12/2017 11:49:56.001:13858) : arch=x86_64 syscall=unlink success=yes exit=0 a0=0x15ee7a0 a1=0x1 a2=0x1 a3=0x7ffc2c82e6b0 items=2 ppid=9572 pid=9593
auid=cleopatra uid=cleopatra gid=cleopatra euid=cleopatra suid=cleopatra fsuid=cleopatra egid=cleopatra sgid=cleopatra fsgid=cleopatra tty=pts0 ses=1779 comm=vim exe=/usr/bin
/vim subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=secretcats_watch
```

You can tell that the first of these two messages happened when Cleopatra saved the file and exited `vim` because the second message shows `objtype=DELETE`, where her temporary `vim` swap file was deleted.

Okay, this is all good, but what if this is too much information? What if you just want a quick and sparse list of all of the security events that got triggered by this rule? For that, we'll use `aureport`. We'll use it just like we did previously.

First, let's pipe the `aureport` output into `less` instead of into `grep` so that we can see the column headers:

```
[donnie@localhost ~]$ sudo aureport -i -k | less
Key Report
===============================================
# date time key success exe auid event
===============================================
1\. 12/11/2017 13:06:20 passwd_changes yes ? donnie 11393
2\. 12/11/2017 13:49:15 passwd_changes yes ? donnie 11511
3\. 12/11/2017 13:49:15 passwd_changes yes /usr/bin/chfn donnie 11512
4\. 12/11/2017 14:54:11 passwd_changes yes /usr/sbin/usermod donnie 11728
5\. 12/11/2017 14:54:25 passwd_changes yes /usr/sbin/usermod donnie 11736
. . .
. . .
```

The status in the `success` column will be either `yes` or `no`, depending upon whether the user was able to successfully perform an action that violated a rule. Or, it could be a question mark if the event isn't the result of the rule being triggered.

For Charlie, we see a `yes` event in line *48*, with the events in lines *49* through *51* all having a `no` status. We can also see that all of these entries were triggered by Charlie's use of the `ls` command:

[donnie@localhost ~]$ sudo aureport -i -k | grep 'secretcats_watch'

6\. 12/11/2017 15:01:25 secretcats_watch yes ? donnie 11772

8\. 12/12/2017 11:49:29 secretcats_watch yes /usr/bin/ls cleopatra 13828

9\. 12/12/2017 11:49:37 secretcats_watch yes /usr/bin/vim cleopatra 13830

10\. 12/12/2017 11:49:37 secretcats_watch yes /usr/bin/vim cleopatra 13829

48\. 12/12/2017 12:32:04 secretcats_watch yes /usr/bin/ls charlie 14152

49\. 12/12/2017 12:32:04 secretcats_watch no /usr/bin/ls charlie 14153

50\. 12/12/2017 12:32:04 secretcats_watch no /usr/bin/ls charlie 14154

51\. 12/12/2017 12:32:04 secretcats_watch no /usr/bin/ls charlie 14155

[donnie@localhost ~]$

You'd be tempted to think that the `yes` event in line *48* indicates that Charlie was successful in reading the contents of the `secretcats` directory. To analyze this further, let's look at the event numbers at the end of each line and correlate them to the output of our previous `ausearch` command. You'll see that event numbers `14152` through `14155` belong to records that all have the same timestamp. We can see this in the first line of each record:

```
[donnie@localhost ~]$ sudo ausearch -i -k secretcats_watch | less
type=PROCTITLE msg=audit(12/12/2017 12:32:04.341:14152) : proctitle=ls --color=auto -l /secretcats
type=PROCTITLE msg=audit(12/12/2017 12:32:04.341:14153) : proctitle=ls --color=auto -l /secretcats
type=PROCTITLE msg=audit(12/12/2017 12:32:04.341:14154) : proctitle=ls --color=auto -l /secretcats
type=PROCTITLE msg=audit(12/12/2017 12:32:04.341:14155) : proctitle=ls --color=auto -l /secretcats
```

As we noted previously, the last record of this series shows `Permission denied` for Charlie, and that's what really counts.

> Space doesn't permit me to give a full explanation of each individual item in an audit log record. However, you can read about it here, in the official Red Hat documentation: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/auditing-the-system_security-hardening#understanding-audit-log-files_auditing-the-system](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/auditing-the-system_security-hardening#understanding-audit-log-files_auditing-the-system).

### Searching for system call rule violations

The third rule that we created was to monitor that sneaky Charlie. This rule will alert us whenever Charlie tries to open or create a file. (As we noted previously, `1006` is Charlie's user ID number.):

```
sudo auditctl -a always,exit -F arch=b64 -S openat -F auid=1006
```

Even though Charlie hasn't done that much on this system, this rule gives us a lot more log entries than what we bargained for. We'll look at just a couple of entries:

```
time->Tue Dec 12 11:49:29 2017
type=PROCTITLE msg=audit(1513097369.952:13828): proctitle=6C73002D2D636F6C6F723D6175746F
type=PATH msg=audit(1513097369.952:13828): item=0 name="." inode=33583041 dev=fd:01 mode=043770 ouid=99 ogid=1009 rdev=00:00 obj=unconfined_u:object_r:default_t:s0 objtype=NO
RMAL
type=CWD msg=audit(1513097369.952:13828):  cwd="/secretcats"
type=SYSCALL msg=audit(1513097369.952:13828): arch=c000003e syscall=257 success=yes exit=3 a0=ffffffffffffff9c a1=10d1560 a2=90800 a3=0 items=1 ppid=9572 pid=9592 auid=1004 u
id=1004 gid=1006 euid=1004 suid=1004 fsuid=1004 egid=1006 sgid=1006 fsgid=1006 tty=pts0 ses=1779 comm="ls" exe="/usr/bin/ls" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0
:c0.c1023 key="secretcats_watch"
```

This record was generated when Charlie tried to access the `/secretcats/` directory. So, we can expect to see this one. But, what we didn't expect to see was the exceedingly long list of records of files that Charlie indirectly accessed when he logged into the system through **Secure Shell** (**SSH**). Here's one:

```
time->Tue Dec 12 11:50:28 2017
type=PROCTITLE msg=audit(1513097428.662:13898): proctitle=737368643A20636861726C6965407074732F30
type=PATH msg=audit(1513097428.662:13898): item=0 name="/proc/9726/fd" inode=1308504 dev=00:03 mode=040500 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:unconfined_r:unconfined_t
:s0-s0:c0.c1023 objtype=NORMAL
type=CWD msg=audit(1513097428.662:13898):  cwd="/home/charlie"
type=SYSCALL msg=audit(1513097428.662:13898): arch=c000003e syscall=257 success=yes exit=3 a0=ffffffffffffff9c a1=7ffc7ca1d840 a2=90800 a3=0 items=1 ppid=9725 pid=9726 auid=1
006 uid=1006 gid=1008 euid=1006 suid=1006 fsuid=1006 egid=1008 sgid=1008 fsgid=1008 tty=pts0 ses=1781 comm="sshd" exe="/usr/sbin/sshd" subj=unconfined_u:unconfined_r:unconfin
ed_t:s0-s0:c0.c1023 key=(null)
```

Here's another one:

```
time->Tue Dec 12 11:50:28 2017
type=PROCTITLE msg=audit(1513097428.713:13900): proctitle=737368643A20636861726C6965407074732F30
type=PATH msg=audit(1513097428.713:13900): item=0 name="/etc/profile.d/" inode=33593031 dev=fd:01 mode=040755 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:bin_t:s0 objtype=
NORMAL
type=CWD msg=audit(1513097428.713:13900):  cwd="/home/charlie"
type=SYSCALL msg=audit(1513097428.713:13900): arch=c000003e syscall=257 success=yes exit=3 a0=ffffffffffffff9c a1=1b27930 a2=90800 a3=0 items=1 ppid=9725 pid=9726 auid=1006 u
id=1006 gid=1008 euid=1006 suid=1006 fsuid=1006 egid=1008 sgid=1008 fsgid=1008 tty=pts0 ses=1781 comm="bash" exe="/usr/bin/bash" subj=unconfined_u:unconfined_r:unconfined_t:s
0-s0:c0.c1023 key=(null)
```

In the first record, we see that Charlie accessed the `/usr/sbin/sshd` file. In the second, we see that he accessed the `/usr/bin/bash` file. It's not that Charlie chose to access those files. The operating system accessed those files for him in the course of just a normal login event. So as you see, when you create audit rules, you have to be careful what you wish for because there's a definite danger that the wish might be granted. If you really need to monitor someone, you'll want to create a rule that won't give you quite this much information.

While we're at it, we might as well see what the `aureport` output for this looks like:

```
[donnie@localhost ~]$ sudo aureport -s -i | grep 'openat'
1068\. 12/12/2017 11:49:29 openat 9592 ls cleopatra 13828
1099\. 12/12/2017 11:50:28 openat 9665 sshd charlie 13887
1100\. 12/12/2017 11:50:28 openat 9665 sshd charlie 13889
1101\. 12/12/2017 11:50:28 openat 9665 sshd charlie 13890
1102\. 12/12/2017 11:50:28 openat 9726 sshd charlie 13898
1103\. 12/12/2017 11:50:28 openat 9726 bash charlie 13900
1104\. 12/12/2017 11:50:28 openat 9736 grep charlie 13901
1105\. 12/12/2017 11:50:28 openat 9742 grep charlie 13902
1108\. 12/12/2017 11:50:51 openat 9766 ls charlie 13906
1110\. 12/12/2017 12:15:35 openat 10952 ls vicky 14077
1115\. 12/12/2017 12:30:54 openat 11632 sshd charlie 14129
. . .
. . .
```

In addition to what Charlie did, we also see what Vicky and Cleopatra did. That's because the rule that we set for the `/secretcats/` directory generated `openat` events when Vicky and Cleopatra accessed, viewed, or created files in that directory.

### Generating authentication reports

You can generate user authentication reports without having to define any audit rules. Just use `aureport` with the `-au` option switch. (Remember `au`, the first two letters of *authentication*.):

```
[donnie@localhost ~]$ sudo aureport -au
[sudo] password for donnie:
Authentication Report
============================================
# date time acct host term exe success event
============================================
1\. 10/28/2017 13:38:52 donnie localhost.localdomain tty1 /usr/bin/login yes 94
2\. 10/28/2017 13:39:03 donnie localhost.localdomain /dev/tty1 /usr/bin/sudo yes 102
3\. 10/28/2017 14:04:51 donnie localhost.localdomain /dev/tty1 /usr/bin/sudo yes 147
. . .
. . .
239\. 12/12/2017 11:50:20 charlie 192.168.0.222 ssh /usr/sbin/sshd no 13880
244\. 12/12/2017 12:10:06 cleopatra 192.168.0.222 ssh /usr/sbin/sshd no 13992
. . .
```

For login events, this tells us whether the user logged in at the local terminal or remotely through SSH. To see the details of any event, use `ausearch` with the `-a` option, followed by the event number that you see at the end of a line. (Strangely, the `-a` option stands for an *event*.)

Let's look at event number `14122` for Charlie:

```
[donnie@localhost ~]$ sudo ausearch -a 14122
----
time->Tue Dec 12 12:30:49 2017
type=USER_AUTH msg=audit(1513099849.322:14122): pid=11632 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='op=pubkey acct="charlie" exe="/usr/sbin/sshd" hostname=? addr=192.168.0.222 terminal=ssh res=failed'
```

The problem with this is that it really doesn't make any sense. I'm the one who did the logins for Charlie, and I know for a fact that Charlie never had any failed logins. In fact, we can correlate this with the matching entry from the `/var/log/secure` file:

```
Dec 12 12:30:53 localhost sshd[11632]: Accepted password for charlie from 192.168.0.222 port 34980 ssh2
Dec 12 12:30:54 localhost sshd[11632]: pam_unix(sshd:session): session opened for user charlie by (uid=0)
```

The timestamps for these two entries are a few seconds later than the timestamp for the `ausearch` output, but that's okay. There's nothing in this log file to suggest that Charlie ever had a failed login, and these two entries clearly show that Charlie's login really was successful. The lesson here is that when you see something strange in either the `ausearch` or `aureport` output, be sure to correlate it with the matching entry in the proper authentication log file to get a better idea of what's going on. (By authentication log file, I mean `/var/log/secure` for Red Hat-type systems and `/var/log/auth.log` for Ubuntu systems. The names may vary for other Linux distros.)

### Using pre-defined rulesets

In the `/usr/share/doc/audit-version_number/rules/` directory of your CentOS 7 machine and the `/usr/share/audit/sample-rules/` directory of your AlmaLinux machines, you'll see some pre-made rulesets for different scenarios. Once you install `auditd` on Ubuntu, you'll have audit rules in the `/usr/share/doc/auditd/examples/rules/` directory. In any case, some of the rulesets are common among all three of these distros. Let's look at the AlmaLinux 9 machine to see what we have there:

```
[donnie@localhost ~]$ cd /usr/share/audit/sample-rules/
[donnie@localhost sample-rules]$ ls -l
total 160
. . .
-rw-r--r--. 1 root root 4943 Oct 27 07:15 30-nispom.rules
. . .
-rw-r--r--. 1 root root 6179 Oct 27 07:15 30-pci-dss-v31.rules
-rw-r--r--. 1 root root 6624 Oct 27 07:15 30-stig.rules
-rw-r--r--. 1 root root 1458 Oct 27 07:15 31-privileged.rules
-rw-r--r--. 1 root root  213 Oct 27 07:15 32-power-abuse.rules
. . .
[donnie@localhost sample-rules]$
```

The three files I want to focus on are the `nispom`, `pci-dss`, and `stig` files. Each of these three rulesets is designed to meet the auditing standards of a particular certifying agency. In order, these rulesets are:

*   **nispom**: The National Industrial Security Program – you'll see this ruleset used at either the US Department of Defense or its contractors.
*   **pci-dss**: Payment Card Industry Data Security Standard – if you work in the banking or financial industries, or even if you're just running an online business that accepts credit cards, you'll likely become very familiar with this.
*   **stig**: Security Technical Implementation Guides – if you work for the US government, or possibly other governments, you'll be dealing with this one.

To use one of these rules sets, just copy the appropriate files over to the `/etc/audit/rules.d/` directory:

```
[donnie@localhost rules]$ sudo cp 30-pci-dss-v31.rules /etc/audit/rules.d
[donnie@localhost rules]$
```

After you've copied the rule file over, restart the `auditd` daemon to read in the new rules.

For Red Hat, CentOS, or AlmaLinux, do this:

```
sudo service auditd restart
```

For Ubuntu, do this:

```
sudo systemctl restart auditd
```

Of course, there's always the chance that a particular rule in one of these sets might not work for you or that you might need to enable a rule that's currently disabled. If so, just open the appropriate rules file in your text editor and comment out what doesn't work or uncomment what you need to enable.

Even though `auditd` is very cool, bear in mind that it only alerts you about potential security breaches. It doesn't do anything to harden the system against them.

### Hands-on lab – using auditd

In this lab, you'll practice using the features of `auditd`. Let's get started:

1.  For Ubuntu only, install `auditd`:

```
sudo apt update
sudo apt install auditd
```

1.  View the rules that are currently in effect:

```
sudo auditctl -l
```

1.  From the command line, create a temporary rule that audits the `/etc/passwd` file for changes. Verify that the rule is in effect:

```
sudo auditctl -w /etc/passwd -p wa -k passwd_changes
sudo auditctl -l
```

1.  Create a user account for Lionel. On Ubuntu, do this:

```
sudo adduser lionel
```

1.  On CentOS or AlmaLinux, do this:

```
sudo useradd lionel
sudo passwd lionel
```

1.  Search for audit messages regarding any changes to the `passwd` file:

```
sudo ausearch -i -k passwd_changes
sudo aureport -i -k | grep 'passwd_changes'
```

1.  Log out of your own account and log in as Lionel. Then, log out of Lionel's account and back in to your own.
2.  Do an authentication report:

```
sudo aureport -au
```

1.  Create the `/secrets` directory and set the permissions so that only the root user can access it:

```
sudo mkdir /secrets
sudo chmod 700 /secrets
```

1.  Create a rule that monitors the `/secrets` directory:

```
sudo auditctl -w /secrets -k secrets_watch
sudo auditctl -l
```

1.  Log out of your account, and log in as Lionel. Have him try to view what's in the `/secrets` directory:

```
ls -l /secrets
```

1.  Log out of Lionel's account and log in to your own. View the alerts that Lionel created:

```
sudo ausearch -i -k secrets_watch | less
```

1.  You now have two temporary rules that will disappear when you reboot the machine. Make them permanent by creating a `custom.rules` file:

```
sudo sh -c "auditctl -l > /etc/audit/rules.d/custom.rules"
```

1.  Reboot the machine and verify that the rules are still in effect:

```
sudo auditctl -l
```

You've reached the end of the lab – congratulations!

### Hands-on lab –Using pre-configured rules with auditd

In this lab, we’ll simulate that the US government is our client, and that we need to set up a server that will meet their **Security Technical Implementation Guides** (**STIG**) auditing standards. To do that, we’ll use several pre-configured rulesets that get installed when you install `auditd`. Note that this lab will work on any of your virtual machines:

1.  Delete the `custom.rules` file that you created in the previous lab, and then restart the `auditd` service.
2.  Copy the `10-base-config.rules`, `30-stig.rules`, `31-privileged.rules`, and `99-finalize.rules` files to the `/etc/audit/rules.d/` directory. (These rules files are in the `/usr/share/doc/auditd/examples/rules/` directory on Ubuntu, and in the `/usr/share/audit/sample-rules/` directory on AlmaLinx.):

```
[donnie@almalinux9 sample-rules]$ pwd
/usr/share/audit/sample-rules
[donnie@almalinux9 sample-rules]$ sudo cp 10-base-config.rules 30-stig.rules 31-privileged.rules 99-finalize.rules /etc/audit/rules.d
[donnie@almalinux9 sample-rules]$
```

1.  Restart the `auditd` service, and then use `sudo auditctl -l` to view the new active ruleset.
2.  End of lab.

> You’ve just seen that we can sometimes use several different pre-configured rulesets at once *if* they complement each other. Understand though that you’ll never use *all* of the pre-configured rulesets at once.

In this section, you looked at some examples of how to work with `auditd`. Next, let’s look at a simpler method of auditing files and directories.

## Auditing files and directories with inotifywait

There might be times when you’ll just want a quick and easy way to monitor a file or a directory in real time. Instead of having audit messages sent to the `audit.log` file, you can use `inotifywait` to have a message pop up in your terminal as soon as someone accesses a designated file or directory. This tool is part of the `inotify-tools` package on both Ubuntu and AlmaLinux. It’s not installed by default, so go ahead and install it if it isn’t already.

To monitor a single file, just do:

```
donnie@donnie-ca:~$ sudo inotifywait -m /secrets/donnie_file.txt
[sudo] password for donnie: 
Setting up watches.
Watches established.
/secrets/donnie_file.txt OPEN 
/secrets/donnie_file.txt CLOSE_NOWRITE,CLOSE
```

The `/secrets/` directory is set so that only someone with root privileges can access it, so I have to use `sudo` to make this work. The `-m` option causes `inotifywait` to perform continuous monitoring, instead of exiting as soon as something happens. The **OPEN** message came up when I opened the file with `less`, and the **CLOSE_NOWRITE,CLOSE** message came up when I exited `less`. Now, let’s close this down and monitor the whole directory. All we have to do is to add the `-r` option and leave out the file name, like this:

```
donnie@donnie-ca:~$ sudo inotifywait -m -r /secrets/
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
/secrets/ OPEN donnie_file.txt
/secrets/ CREATE .donnie_file.txt.swp
/secrets/ OPEN .donnie_file.txt.swp
/secrets/ CREATE .donnie_file.txt.swx
. . .
. . .
/secrets/ CLOSE_NOWRITE,CLOSE donnie_file.txt
/secrets/ OPEN donnie_file.txt
/secrets/ CLOSE_NOWRITE,CLOSE donnie_file.txt
/secrets/ MODIFY .donnie_file.txt.swp
/secrets/ MODIFY .donnie_file.txt.swp
```

This time, I opened the `donnie_file.txt` file in `vim`, which caused a whole bunch of messages to come up. That’s because when you open a file in `vim`, it creates some temporary files that will get cleared out when you exit `vim`. (Note that I haven’t actually edited the file yet, and that more messages will get created when I do.)

As good as `inotifywait` seems to be, there is one downside. It’s just that to use it, you’ll need to stayed glued to your workstation, keep the terminal window from which you’re running `inotifywait` open, and watch for messages to pop up. There’s no logging mechanism, and no daemon mode. But, if you need to monitor something in real-time, this could be useful.

That’s all there is to it for `inotifywait`. Next, we'll look at OpenSCAP, which can actually remediate a less-than-secure system.

## Applying OpenSCAP policies with oscap

The **Security Content Automation Protocol** (**SCAP**) was created by the US **National Institute of Standards and Technology** (**NIST**). It consists of hardening guides, hardening templates, and baseline configuration guides for setting up secure systems. OpenSCAP is a set of FOSS tools that can be used to implement SCAP. It consists of the following:

*   Security profiles that you can apply to a system. There are different profiles for meeting the requirements of several different certifying agencies.
*   Security guides to help with the initial setup of your system.
*   The `oscap` command-line utility to apply security templates.
*   On systems that have a desktop interface, you have SCAP Workbench, a GUI-type utility.

You can install OpenSCAP on either the Red Hat or the Ubuntu distros, but it's much better implemented on the Red Hat distros. For one thing, when you install a Red Hat-type operating system, you can choose to apply a SCAP profile during installation. You can't do that with Ubuntu. All of the Red Hat-type distros come with a fairly complete set of ready-to-use profiles. Ubuntu 22.04 comes with outdated profiles for Fedora 14 and RHEL 6, and none for Ubuntu 22.04, which I think is totally bizarre. Not to worry though, because I’ll show you how to get some good Ubuntu profiles in just a bit.

When doing initial system builds, it's desirable to start with a security checklist that's appropriate for your scenario, because there are certain things the OpenSCAP can’t automate for you. Then, use OpenSCAP to do the rest. I'll tell you more about security checklists at the end of *Chapter 16*, *Security Tips and Tricks for the Busy Bee*.

All right, let's learn how to install OpenSCAP and how to use the command-line utility that's common to all of our distros.

### Installing OpenSCAP

On your CentOS 7 machine, assuming that you didn't install OpenSCAP during the operating system installation, do this:

```
sudo yum install openscap-scanner scap-security-guide
```

Do the following for either AlmaLinux 8 or AlmaLinux 9:

sudo dnf install openscap-scanner scap-security-guide

Do the following on an Ubuntu 22.04 machine:

```
sudo apt install python-openscap
```

### Viewing the profile files

On the CentOS 7 machine and the AlmaLinux machines, you'll see the profile files in the `/usr/share/xml/scap/ssg/content/` directory.

As I already mentioned, Ubuntu only gives us some outdated Fedora 14 and RHEL 6 profiles in the `/usr/share/openscap/` directory, and none at all for any flavor of Ubuntu. (Why that is, I have no clue.) The profile files are in `.xml` format, and each one contains one or more profiles that you can apply to the system. For example, here are some from the CentOS 7 machine:

```
[donnie@localhost content]$ pwd
/usr/share/xml/scap/ssg/content
[donnie@localhost content]$ ls -l
total 50596
-rw-r--r--. 1 root root  6734643 Oct 19 19:40 ssg-centos6-ds.xml
-rw-r--r--. 1 root root  1596043 Oct 19 19:40 ssg-centos6-xccdf.xml
-rw-r--r--. 1 root root 11839886 Oct 19 19:41 ssg-centos7-ds.xml
-rw-r--r--. 1 root root  2636971 Oct 19 19:40 ssg-centos7-xccdf.xml
-rw-r--r--. 1 root root      642 Oct 19 19:40 ssg-firefox-cpe-dictionary.xml
. . .
-rw-r--r--. 1 root root 11961196 Oct 19 19:41 ssg-rhel7-ds.xml
-rw-r--r--. 1 root root   851069 Oct 19 19:40 ssg-rhel7-ocil.xml
-rw-r--r--. 1 root root  2096046 Oct 19 19:40 ssg-rhel7-oval.xml
-rw-r--r--. 1 root root  2863621 Oct 19 19:40 ssg-rhel7-xccdf.xml
[donnie@localhost content]$
```

You’ll see a somewhat similar list on your AlmaLinux 8 machine, except that they’ll be specific to AlmaLinux 8\. On AlmaLinux 9, things are a bit different. At the time of this writing, all we have is just one profile file. That’s because the RHEL 9 distros are quite new, so the development of SCAP profiles for them isn’t yet complete. Anyway, here’s AlmaLinux 9 file:

```
[donnie@almalinux9 content]$ pwd
/usr/share/xml/scap/ssg/content
[donnie@almalinux9 content]$ ls -l
total 21524
-rw-r--r--. 1 root root 22040119 Oct 27 08:37 ssg-almalinux9-ds.xml
[donnie@almalinux9 content]$
```

The command-line utility for working with OpenSCAP is `oscap`. On our AlmaLinux 9 machine, let's use this with the `info` switch to view information about any of the profile files. Let's look at the `ssg-almalinux9-ds.xml` file:

```
[donnie@almalinux9 content]$ pwd
/usr/share/xml/scap/ssg/content
[donnie@almalinux9 content]$ sudo oscap info ssg-almalinux9-ds.xml 
. . .
. . .
    Profiles:
        Title: ANSSI-BP-028 (enhanced)
        Id: xccdf_org.ssgproject.content_profile_anssi_bp28_enhanced
        Title: ANSSI-BP-028 (high)
        Id: xccdf_org.ssgproject.content_profile_anssi_bp28_high
        Title: ANSSI-BP-028 (intermediary)
        Id: xccdf_org.ssgproject.content_profile_anssi_bp28_intermediary
        Title: ANSSI-BP-028 (minimal)
        Id: xccdf_org.ssgproject.content_profile_anssi_bp28_minimal
        . . .
```

Due to formatting constraints, I can’t show you the entire list of profiles. So, do this for yourself and scroll down the list. You’ll see profiles for **STIG** and **PCI-DSS**, just as we had for the auditing rules. There’s also a **HIPAA** profile for medical facilities here in the US, several benchmark profiles from the **Center for Internet Security**, (**CIS**), and several that are specific for certain non-US countries, among others.

### Getting the missing profiles for Ubuntu

We’ve seen that there aren’t any SCAP profiles for Ubuntu in the Ubuntu repositories. So, is all hope lost for Ubuntu users? Absolutely not. Fortunately, the `scap-security-guide` package that you can install on a Fedora Server virtual machine comes with SCAP profiles for a variety of other Linux distros, including the newest versions of Ubuntu. So, your best bet for setting up OpenSCAP on Ubuntu is to create a Fedora Server VM, install the `scap-security-guide` package in the same manner that you just did for AlmaLinux, and then copy the appropriate profile file from Fedora’s `/usr/share/xml/scap/ssg/content/` directory to your Ubuntu machine. After that, you’re golden.

### Scanning the system

In this section, we'll work with our AlmaLinux 9 VM.

> This procedure works the same for most all Linux distros, except that the names of the profiles will differ.

Now, let's say that we need to ensure that our systems are compliant with **Payment Card Industry** standards. First, we'll scan the AlmaLinux 9 machine to see what needs remediation. (Note that the following command is very long and wraps around on the printed page.):

```
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_pci-dss --fetch-remote --results scan-xccdf-results.xml /usr/share/xml/scap/ssg/content/ssg-almalinux9-ds.xml
```

As we always like to do, let's break this down:

*   `xccdf eval`: The **Extensible Configuration Checklist Description Format** is one of the languages that we can use to write security profile rules. We're going to use a profile that was written in this language to perform an evaluation of the system.
*   `--profile xccdf_org.ssgproject.content_profile_pci-dss`: Here, I specified that I want to use the Payment Card Industry-Data Security Standard profile to evaluate the system. (Profile names come from the `Id` lines in the profile file.)
*   `--fetch-remote`: Use this option to fetch additional rules. (Note that this option won’t work if you have the system crypto policy set to `FUTURE` mode.)
*   `--results scan-xccdf-results.xml`: I'm going to save the scan results to this `.xml` format file. When the scan has finished, I'll create a report from this file.
*   `/usr/share/xml/scap/ssg/content/ssg-almalinux9-ds.xml`: This is the profile file that contains the `xccdf_org.ssgproject.content_profile_pci-dss` profile.

As the scan progresses, the output will be sent to the screen, as well as to the designated output file. It's a long list of items, so I'll only show you a few of them. Here are a couple of items that look okay:

```
Title   Ensure Software Patches Installed
Rule    xccdf_org.ssgproject.content_rule_security_patches_up_to_date
OVAL Definition ID  oval:org.almalinux.alsa:def:20227967
OVAL Definition Title   ALSA-2022:7967: qemu-kvm security, bug fix, and enhancement update (Moderate)
Result  pass
Title   Ensure Software Patches Installed
Rule    xccdf_org.ssgproject.content_rule_security_patches_up_to_date
OVAL Definition ID  oval:org.almalinux.alsa:def:20227959
OVAL Definition Title   ALSA-2022:7959: guestfs-tools security, bug fix, and enhancement update (Low)
Result  pass
```

Here are a couple of items that need to be fixed:

```
Title   Ensure PAM Displays Last Logon/Access Notification
Rule    xccdf_org.ssgproject.content_rule_display_login_attempts
Result  fail
Title   Limit Password Reuse
Rule    xccdf_org.ssgproject.content_rule_accounts_password_pam_unix_remember
Result  fail
Title   Lock Accounts After Failed Password Attempts
Rule    xccdf_org.ssgproject.content_rule_accounts_passwords_pam_faillock_deny
Result  fail
Title   Set Lockout Time for Failed Password Attempts
Rule    xccdf_org.ssgproject.content_rule_accounts_passwords_pam_faillock_unlock_time
Result  fail
```

So, we have patches for certain security vulnerabilities installed, which is good. However, it seems that we have some problems with our password policies.

Now that I've run the scan and created an output file with the results, I can build my report:

```
sudo oscap xccdf generate report scan-xccdf-results.xml > scan-xccdf-results.html
```

This extracts information from the `.xml` format file that isn't meant to be read by humans and transfers it to a `.html` file that you can open in your web browser. (For the record, the report says that there are 49 problems that need to be fixed.)

### Remediating the system

So, we have 49 problems that we need to fix before our system can be considered compliant with Payment Card Industry standards. Let's see how many of them `oscap` can fix for us:

```
sudo oscap xccdf eval --remediate --profile xccdf_org.ssgproject.content_profile_pci-dss --fetch-remote --results scan-xccdf-results.xml /usr/share/xml/scap/ssg/content/ssg-almalinux9-ds.xml
```

This is the same command that I used to perform the initial scan, except that I added the `--remediate` option and I'm saving the results to a different file. You'll want to have a bit of patience when you run this command because fixing some problems involves downloading and installing software packages. In fact, even as I type this, `oscap` is busy downloading and installing the missing AIDE intrusion detection system package.

Okay, here are some of the things that were fixed:

```
Title   Verify and Correct File Permissions with RPM
Rule    xccdf_org.ssgproject.content_rule_rpm_verify_permissions
Result  fixed
Title   Install AIDE
Rule    xccdf_org.ssgproject.content_rule_package_aide_installed
Result  fixed
Title   Build and Test AIDE Database
Rule    xccdf_org.ssgproject.content_rule_aide_build_database
Result  fixed
Title   Configure Periodic Execution of AIDE
Rule    xccdf_org.ssgproject.content_rule_aide_periodic_cron_checking
```

Result fixed

There are a couple of errors because of things that `oscap` couldn't fix, but that's normal. At least you know about them so that you can try to fix them yourself.

Now, check this out. Do you remember how in *Chapter 3, Securing User Accounts*, I made you jump through hoops to ensure that users had strong passwords that expire on a regular basis? Well, by applying this OpenSCAP profile, you get all that fixed for you automatically. Here are a few of the items that were fixed:

```
Title   Lock Accounts After Failed Password Attempts
Rule    xccdf_org.ssgproject.content_rule_accounts_passwords_pam_faillock_deny
Result  fixed
Title   Set Lockout Time for Failed Password Attempts
Rule    xccdf_org.ssgproject.content_rule_accounts_passwords_pam_faillock_unlock_time
Result  fixed
Title   Ensure PAM Enforces Password Requirements - Minimum Digit Characters
Rule    xccdf_org.ssgproject.content_rule_accounts_password_pam_dcredit
Result  fixed
Title   Ensure PAM Enforces Password Requirements - Minimum Lowercase Characters
Rule    xccdf_org.ssgproject.content_rule_accounts_password_pam_lcredit
Result  fixed
```

So, yeah, OpenSCAP is pretty cool, and even the command-line tools aren't hard to use. However, if you have to use a GUI, we have a tool for that, which we'll look at next.

### Using SCAP Workbench

For machines with a desktop environment installed, we have SCAP Workbench. However, if you've ever worked with early versions of the tool, you were likely quite disappointed. Indeed, the early versions of Workbench were so bad that they weren't even usable. Thankfully, things have since improved. Now, Workbench is quite a nice little tool.

To get SCAP Workbench, just use the appropriate installation command. On CentOS 7, do this:

```
sudo yum install scap-workbench
```

On AlmaLinux 8 or AlmaLinux 9, do this:

```
sudo dnf install scap-workbench
```

On Ubuntu, do the following:

```
sudo apt install scap-workbench
```

Yeah, the package name is just `scap-workbench` instead of `openscap-workbench`. I don't know why, but I do know that you'll never find it if you're searching for `openscap` packages.

Once you've installed it, you'll see its menu item on the **Show Applications** portion of the **Activities** page:

![Figure 12.2: SCAP Workbench on the Gnome 3 desktop](img/file76.png)

Figure 12.2: SCAP Workbench on the Gnome 3 desktop

When you first open the program, you would think that the system would ask you for a root or sudo password. But, it doesn't. We'll see if that affects us in a moment.

The first thing you'll see on the opening screen is a drop-down list for you to select the type of content that you want to load. I'll select **AlmaLinux9** and then click on the **Load Content** button:

![Figure 12.3: Select content to load](img/file77.png)

Figure 12.3: Select content to load

Next, you'll see the top panel, where you can select the desired profile. You can also choose to customize the profile, and whether you want to run the scan on the local machine or on a remote machine. In the bottom pane, you'll see a list of rules for that profile. You can expand each rule item to get a description of that rule:

![Figure 12.4: Viewing the rules, and generating a remediation role](img/file78.png)

Figure 12.4: Viewing the rules, and generating a remediation role

At the bottom of this screen, you see some cool options. Click on the **Generate remediation role** button, and you can choose to create a Puppet manifest, an Ansible playbook, or a Bash shell script that you can distribute and apply to other AlmaLinux 9 machines on your network. You can also choose to **Fetch remote resources** and to **Remediate**.

Now, let's click that **Scan** button to see what happens:

![Figure 12.5: Enter your password](img/file79.png)

Figure 12.5: Enter your password

Cool! As I had hoped, it prompts you for your sudo password. Beyond that, I'll leave you to play with it. It's just another one of those GUI-thingies, so the rest of it should be fairly easy to figure out.

Next, we'll take a look at how to choose an appropriate OpenSCAP profile.

### Choosing an OpenSCAP profile

So, now, you're saying, *Okay, this is all good, but how do I find out what's in these profiles and which one I need?* Well, there are several ways.

The first way, which I've just shown you, is to install SCAP Workbench on a machine with a desktop interface and read through the descriptions of all the rules for each profile.

The second way, which might be a bit easier, is to go to the OpenSCAP website and look through the documentation that they have there.

> You'll find information about the available OpenSCAP profiles at [https://www.open-scap.org/security-policies/choosing-policy/](https://www.open-scap.org/security-policies/choosing-policy/).

As far as knowing which profile to choose, there are a few things to consider:

*   If you work in the financial sector or in a business that does online financial transactions, then go with the `pci-dss` profile.
*   If you work for a government agency, especially if it's the US government, then go with either the `stig` profile or the `nispom` profile, as dictated by the particular agency.
*   If neither of these two considerations applies to your situation, then you'll just want to do some research and planning in order to figure out what really needs to be locked down. Look through the rules in each profile and read through the documentation on the OpenSCAP website to help you decide what you need.

With Red Hat and its offspring, you can even apply a policy as you install the operating system. We'll look at that next.

### Applying an OpenSCAP profile during system installation

One of the things that I love about the Red Hat folk is that they totally get this whole security thing. Yeah, we can lock down other distros and make them more secure, as we've already seen. But with Red Hat distros, it's a bit easier. For a lot of things, the maintainers of the Red Hat-type distros have set secure default options that aren't securely set by default on other distros. (For example, prior to the release of Ubuntu 22.04, Red Hat distros had been the only ones that come with users' home directories locked down by default.) For other things, the Red Hat-type distros come with tools and installation options that help make life easier for a busy, security-conscious administrator.

When you install a Red Hat-type distro, you'll be given the chance to apply an OpenSCAP profile during the operating system installation. Here, on this AlmaLinux 9 installer screen, you'll see the option to choose a security profile in the bottom right-hand corner of the screen:

![Figure 12.6: Select a SCAP profile during installation](img/file80.png)

Figure 12.6: Select a SCAP profile during installation

All you have to do is click on that and then choose your profile:

![Figure: 12.7: Selecting the PCI-DSS profile](img/file81.png)

Figure: 12.7: Selecting the PCI-DSS profile

Okay, that pretty much wraps it up for our discussion of OpenSCAP. The only thing left to add is that, as great as OpenSCAP is, it won't do everything. For example, some security standards require that you have certain directories, such as `/home/` or `/var/`, on their own separate partitions. An OpenSCAP scan will alert you if that's not the case, but it can't change your existing partitioning scheme. So, for things like that, you'll need to get a checklist from the governing body that dictates your security requirements and do a bit of advanced work before you even touch OpenSCAP.

## Summary

We covered a lot of ground in this chapter, and we saw some really cool stuff. We began by looking at a couple of antivirus scanners so that we can prevent infecting any Windows machines that access our Linux servers. In the *Scanning for rootkits with Rootkit Hunter* section, we saw how to scan for those nasty rootkits. We also saw a couple of quick techniques to examine a potentially malicious file. It's important to know how to audit systems, especially in high-security environments, and we saw how to do that. Finally, we wrapped things up with a discussion of hardening our systems with OpenSCAP.

In the next chapter, we'll look at logging and log file security. I'll see you there.

## Questions

1.  Which of the following is true about rootkits?
    1.  They only infect Windows operating systems.
    2.  The purpose of planting a rootkit is to gain root privileges to a system.
    3.  An intruder must have already gained root privileges in order to plant a rootkit.
    4.  A rootkit isn't very harmful.
2.  Which of the following methods would you use to keep `maldet` updated?
    1.  Manually create a `cron` job that runs every day.
    2.  Do nothing, because `maldet` automatically updates itself.
    3.  Once a day, run the normal update command for your operating system.
    4.  Run the `maldet update` utility from the command line.
3.  Which of the following is true about the `auditd` service?
    1.  On an Ubuntu system, you'll need to stop or restart it with the `service` command.
    2.  On a Red Hat-type system, you'll need to stop or restart it with the `service` command.
    3.  On an Ubuntu system, it comes already installed.
    4.  On a Red Hat-type system, you'll need to install it yourself.
4.  You need to create an auditing rule that will alert you every time a particular person reads or creates a file. Which of the following syscalls would you use in that rule?
    1.  `openfile`
    2.  `fileread`
    3.  `openat`
    4.  `fileopen`
5.  Which file does the `auditd` service use to log auditing events?
    1.  `/var/log/messages`
    2.  `/var/log/syslog`
    3.  `/var/log/auditd/audit`
    4.  `/var/log/audit/audit.log`
6.  You need to create custom auditing rules for `auditd`. Where would you place the new rules?
    1.  `/usr/share/audit-version_number/`
    2.  `/etc/audit/`
    3.  `/etc/audit.d/rules/`
    4.  `/etc/audit/rules.d/`
7.  You're setting up a web server for a bank's customer portal. Which of the following SCAP profiles might you apply?
    1.  STIG
    2.  NISPOM
    3.  PCI-DSS
    4.  Sarbanes-Oxley
8.  Which of the following is true about OpenSCAP?
    1.  It can't remediate everything, so you'll need to do advance planning with a checklist before setting up a server.
    2.  It can automatically remediate every problem on your system.
    3.  It's only available for Red Hat-type distros.
    4.  Ubuntu comes with a better selection of SCAP profiles.
9.  Which of the following commands would you use to generate a user authentication report?
    1.  `sudo ausearch -au`
    2.  `sudo aureport -au`
    3.  Define an audit rule, then do `sudo ausearch -au`.
    4.  Define an audit rule, then do `sudo aureport -au`.
10.  Which set of Rootkit Hunter options would you use to have a rootkit scan automatically run every night?
    1.  `-c`
    2.  `-c --rwo`
    3.  `--rwo`
    4.  `-c --cronjob --rwo`

## Further reading

*   How to install and configure maldet: [https://www.servernoobs.com/how-to-install-and-configure-maldet-linux-malware-detect-lmd/](https://www.servernoobs.com/how-to-install-and-configure-maldet-linux-malware-detect-lmd/)
*   Symbiote: Evasive Linux rootkit malware: [https://www.theregister.com/2022/06/10/symbiote_linux_malware/](https://www.theregister.com/2022/06/10/symbiote_linux_malware/)
*   Configuring and auditing Linux systems with `auditd` daemon: [https://linux-audit.com/configuring-and-auditing-linux-systems-with-audit-daemon/](https://linux-audit.com/configuring-and-auditing-linux-systems-with-audit-daemon/)
*   Monitor changes in directories with `inotifywatch`: [https://distrowatch.com/weekly.php?issue=20220905](https://distrowatch.com/weekly.php?issue=20220905)
*   The OpenSCAP portal: [https://www.open-scap.org/](https://www.open-scap.org/)
*   Practical OpenSCAP: [https://www.redhat.com/files/summit/session-assets/2016/SL45190-practical-openscap_security-standard-compliance-and-reporting.pdf](https://www.redhat.com/files/summit/session-assets/2016/SL45190-practical-openscap_security-standard-compliance-and-reporting.pdf)
*   Center for Internet Security (CIS) benchmarks: [https://www.cisecurity.org/cis-benchmarks/](https://www.cisecurity.org/cis-benchmarks/)
*   Auditing the System documentation for RHEL 9: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/auditing-the-system_security-hardening#doc-wrapper](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/auditing-the-system_security-hardening#doc-wrapper)

## Answers

1.  c
2.  b
3.  b
4.  c
5.  d
6.  d
7.  c
8.  a
9.  b
10.  d