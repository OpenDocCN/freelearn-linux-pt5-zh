# 15 Prevent Unwanted Programs from Running

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file104.png)

Once upon a time, we didn’t have to worry much about Linux malware. While it’s still true that Linux users don’t have to worry about viruses, there are other types of malware that can definitely ruin a Linux user’s day. Cryptomining programs planted on your server can eat up memory and CPU cycles, causing your server to work much harder and use more power than it should. Ransomware, which can encrypt either important files or a system’s bootloader, can make these important files or even the whole system inaccessible. Even paying the demanded ransom isn’t always a guarantee that your system will be returned to proper order. One way to prevent these programs from doing their damage is to only allow authorized programs to run, and to block everything else. We have two ways of doing that, which are the topics of this chapter:

*   Mount partitions with the *no* options
*   Use `fapolicyd` on Red Hat-type systems

So, if you’re ready, let’s get going.

## Mount Partitions with the no options

In *Chapter 12*, *Scanning, Auditing, and Hardening*, I showed you how OpenSCAP can automatically bring your Linux systems into compliance with the security standards of certain regulatory bodies. I also told you the inconvenient truth that there are certain things that OpenSCAP can’t do, and that you’ll have to do for yourself. One thing that it can’t do is to partition your system drives as some of these regulatory bodies require. For example, the **Security Technical Implementation Guides** (**STIG**s) that the U.S. Government uses require the following Linux system and data directories to be mounted on their own partitions:

*   `/var`
*   `/var/log/`
*   `/var/tmp/`
*   `/var/log/audit/`
*   `/tmp/`
*   `/home/`
*   `/boot/`

```
/boot/efi/ (You’ll only have this one if your machine is set up in EFI mode.)
```

The reason for this is twofold:

*   If the root (`/`) partition of a Linux operating system becomes too full, it can cause the operating system to completely lock up. Mounting these directories in their own partitions can help prevent the `/` partition from filling up.
*   The STIGs, and possibly other security regulations, require that these partitions be mounted with options that prevent executable programs from running on them, SGID and SUID file permissions from being effective, and device files to be created on them.

As I mentioned, OpenSCAP won’t automatically set up this partitioning scheme for you. So, you’ll need to set it up as you install the operating system. This requires careful planning in order to get the partitions sized correctly. I mean, you don’t want to waste space by making certain partitions too large, and you don’t want to run out of space on partitions that really need the extra space.

> RHEL 9.1 and all of its clones were released a few weeks before I began writing this chapter. You might already have noticed that there’s a bug in the 9.1 installer that wasn’t in the 9.0 installer. That is, the option to create a normal user account isn’t visible on the installer screen. I mean, it’s there, but you can’t see it and can’t scroll down to it. To bring it up, just keep hitting the Tab key until you’ve highlighted the option to create the root user password. Then, hit the Tab key once more, and then hit the Enter key. (Of course, there’s always the chance that the problem will get fixed by the time you read this.)

To get this set up, you’ll need to select the installer option to create a custom partitioning scheme. To make this somewhat realistic, set the size of your virtual machine’s virtual drive to about 1 TB. (Don’t worry if you don’t have that much space on your host machine’s drive. VirtualBox will create a dynamically-sized drive that won’t use 1 TB worth of space on the host drive unless you put 1 TB worth of files on it.) Let’s see what this looks like on AlmaLinux 9:

![Figure 15.1: Choose to create a custom partitioning scheme](img/file105.png)

Figure 15.1: Choose to create a custom partitioning scheme

After selecting the **Custom** option, hit the **Done** button at the top of the screen. On the next screen, click the `+` box to create a mount point. Note that you can create a standard partition, a logical volume, or a thin-provisioned logical volume. (I’m going to go with standard partitions.)

![Figure 15.2: Create a mount point](img/file106.png)

Figure 15.2: Create a mount point

Let’s start by creating the `/boot/` mount point:

![Figure 15.3: Creating the first mount point](img/file107.png)

Figure 15.3: Creating the first mount point

Create the rest of the required mount points for the partitions that I mentioned in the above list. Your completed scheme might look something like this:

![Figure 15.4: Create the rest of the mount points](img/file108.png)

Figure 15.4: Create the rest of the mount points

Of course, your own use case will dictate how large you make each of these partitions. Here, you see that the `/home/` directory is the largest, which suggests that I want to use this machine as a Samba file server. If I were to use this machine for some other purpose, such as a database server, I would resize these partitions as required.

> There’s a long-standing upstream bug in the RHEL installer that also affects the RHEL clones. That is, regardless of how much or how little space you need for each partition, you’ll have to make each one at least 1 GB in size. Otherwise, the installation will fail with an `error in POSTTRANS scriptlet in rpm package kernel-core` message. This has been a known problem for a long time, but it still hasn’t been fixed. (Yes, it does waste some disk space, but there’s nothing we can do about it.)

Now, here’s where we’re going to cheat a bit. We’re going to pretend that we’re dealing with the US government, which requires us to meet the STIG specifications. So, on the installer screen, we’ll click on the option to apply a security profile. On the next screen, we’ll scroll down to where we see the STIG profile, and select it. At the bottom, you’ll see that this profile adds the `noexec`, `nodev`, and `nosuid` options to the partitions, as applicable. (The `/var/` partition only requires the `nodev` option, and the `/boot/` partition only requires the `nodev` and `nosuid` options.)

![Figure 15.5: Applying the STIG profile](img/file109.png)

Figure 15.5: Applying the STIG profile

Here’s what these three mount options do for us:

*   **noexec**: Executable files cannot run from any partition that’s mounted with this option. (This includes executable shell scripts, unless you invoke the script with `sh`. I’ll show you more about this in just a bit.)
*   **nodev**: Users can’t create any device files on partitions that are mounted with this option.
*   **nosuid**: On partitions that are mounted with this option, adding either the SUID or SGID permission to files will have no effect.

When the installation completes, our `/etc/fstab` file will look something like this:

```
UUID=72d0a3b3-cd07-45c0-938e-4e3377750adb /             xfs     defaults        0 0
UUID=7bf3315e-525e-4940-b562-7e0b634d65de /boot       xfs     defaults,nosuid,nodev 0 0
UUID=4df2f723-e875-4194-9ccd-b4a2733fd617 /home       xfs     defaults,noexec,nosuid,nodev 0 0
UUID=d89b01b1-c3ee-48b6-bb40-0311fdd2838a /tmp        xfs     defaults,nodev,noexec,nosuid 0 0
UUID=d79889b0-1635-47d5-950d-8dbca088c464 /var         xfs     defaults,nodev        0 0
UUID=be9a3d41-0e07-4466-8eb7-57fb850df2d4 /var/log     xfs     defaults,nodev,noexec,nosuid 0 0
UUID=ed001588-333b-4027-bef1-754fcc5e868d /var/log/audit     xfs     defaults,nodev,noexec,nosuid 0 0
UUID=05c1b1e9-7f32-4791-9d41-492fce6f5166 /var/tmp     xfs     defaults,nodev,noexec,nosuid 0 0
UUID=c4fdabb8-7e45-4717-b7f5-1cad5f8e7720 none            swap    defaults        0 0
tmpfs /dev/shm tmpfs defaults,relatime,inode64,nodev,noexec,nosuid 0 0
```

(Note that some of these lines might wrap around on the printed page.)

Now, let’s see if we can run an executable script from any of these directories. In my own home directory, I created a shell script that looks like this:

```
#!/bin/bash
echo "This is a test of the noexec option."
exit
```

After adding the executable permission for myself, I tried to run it. Then, I copied it to the `/tmp/` directory and tried to run it again. Here’s what I got:

```
[donnie@localhost ~]$ chmod u+x donnie_script.sh 
[donnie@localhost ~]$ ./donnie_script.sh
-bash: ./donnie_script.sh: Permission denied
[donnie@localhost ~]$ cp donnie_script.sh /tmp
[donnie@localhost ~]$ cd /tmp
[donnie@localhost tmp]$ ./donnie_script.sh
-bash: ./donnie_script.sh: Permission denied
[donnie@localhost tmp]$
```

So, I can’t run it, at least not as a normal user. But, what if I were to try it with `sudo`? Let’s see:

```
[donnie@localhost tmp]$ sudo ./donnie_script.sh
[sudo] password for donnie: 
sudo: unable to execute ./donnie_script.sh: Permission denied
[donnie@localhost tmp]$
```

Cool, the `noexec` option actually works. Well, for this it does. What would happen if we were to invoke the script with `sh?` Let’s see:

```
[donnie@localhost ~]$ sh ./donnie_script.sh
This is a test of the noexec option.
[donnie@localhost ~]$ cd /tmp
[donnie@localhost tmp]$ sh ./donnie_script.sh
This is a test of the noexec option.
[donnie@localhost tmp]$
```

So, with shell scripts, the blocking isn’t perfect. Let’s see what happens with a compiled executable file. Start by downloading the command-line wallet/mining program for the DERO cryptocurrency project from here:

[https://dero.io/download.html#linux](https://dero.io/download.html#linux)

Transfer the file to your virtual machine and untar it:

```
[donnie@localhost ~]$ tar xzvf dero_linux_amd64.tar.gz 
./dero_linux_amd64/
./dero_linux_amd64/Start.md
./dero_linux_amd64/explorer-linux-amd64
./dero_linux_amd64/simulator-linux-amd64
./dero_linux_amd64/dero-miner-linux-amd64
./dero_linux_amd64/derod-linux-amd64
./dero_linux_amd64/dero-wallet-cli-linux-amd64
[donnie@localhost ~]$
```

Note that the executable permission is already set for all of the executable files, so you won’t have to add it.

Now, for the moment of truth. Enter the `dero_linux_amd64` directory and attempt to run the `derod-linux-amd64` program:

```
[donnie@localhost ~]$ cd dero_linux_amd64/
[donnie@localhost dero_linux_amd64]$ ./derod-linux-amd64
-bash: ./derod-linux-amd64: Permission denied
[donnie@localhost dero_linux_amd64]$
```

Since this is a compiled executable instead of a shell script, prefacing the command with `sh` won’t do anything for us in any case. Anyway, keep this DERO stuff handy, because we’ll use it again in the next section.

> If you’re wondering what DERO is, think of it as a private version of Ethereum. You can build other tokens on it and create smart contract applications on it, just like you can do with Ethereum. The difference is that DERO protects your privacy, and Ethereum doesn’t.

In *Chapter 12*, *Scanning, Auditing, and Hardening*, I showed you that only the RHEL-type distros give us the option of applying a SCAP profile as we install the operating system. On non-RHEL distros, you’ll need to apply the SCAP profile after the installation has completed, assuming that an appropriate profile is available for your distro. In any case, if you don’t need to apply a whole SCAP profile but still want to add these security options to your partitions, or if no SCAP profile is available for your distro, just hand-edit the `/etc/fstab` file to add them in.

Next, we’ll look at another control mechanism that, so far at least, is exclusive to the world of Red Hat.

## Understanding fapolicyd

The **File Access Policy Daemon** (**fapolicyd**) is a fairly new addition to Red Hat Enterprise Linux and its various clones. It’s free-as-in-speech software so that anyone can use it, but so far neither Ubuntu nor SUSE have made it available for their distros. To get a quick feel for how it works, go back to the virtual machine that you’ve just been using. First, move the entire `derod-linux-amd64` directory over to the top level of the `/` partition:

```
[donnie@localhost ~]$ sudo mv dero_linux_amd64/ /
[sudo] password for donnie: 
[donnie@localhost ~]$ 
```

By moving the directory instead of copying it, your ownership of the directory and its files will be preserved:

```
[donnie@localhost /]$ ls -ld dero_linux_amd64/
drwx------. 3 donnie donnie 4096 Jan  2 15:42 dero_linux_amd64/
[donnie@localhost /]$
```

Now, copy the script that you created over to `/usr/local/bin/`:

```
[donnie@localhost dero_linux_amd64]$ cd
[donnie@localhost ~]$ sudo cp donnie_script.sh /usr/local/bin
[sudo] password for donnie: 
[donnie@localhost ~]$
```

When you look at the permissions settings on this script file, you’ll see something very unusual:

```
[donnie@localhost ~]$ cd /usr/local/bin
[donnie@localhost bin]$ ls -l
total 16364
-rwx------. 1 root root       61 Dec 31 16:01 donnie_script.sh
[donnie@localhost bin]$
```

You see that doing a `cp` operation automatically changes the ownership of this file to the owner of the target directory, which in this case is the root user. That’s just normal operation, so there’s nothing to see there. What’s so unusual is that we have a permissions setting of `700` on this file. That’s because of something else that our STIG profile has done. That is, the STIG profile has set a **UMASK** of `077` on this system, as we see here:

```
[donnie@localhost ~]$ umask
0077
[donnie@localhost ~]$
```

This means that any normal files that you create will have read and write permissions for only the owner, and any directories that you create will have read, write, and execute permissions for only the owner. To make this demo work we’ll need to change the permissions settings to a value of `755`, like so:

```
[donnie@localhost bin]$ sudo chmod 755 donnie_script.sh
[sudo] password for donnie: 
[donnie@localhost bin]$ ls -l
total 16364
-rwxr-xr-x. 1 root root       61 Dec 31 16:01 donnie_script.sh
[donnie@localhost bin]$
```

Cool. We can now make the demo work. We’ll start by entering the `/dero-linux-amd64/` directory and trying to invoke the `derod-linux-amd64` executable:

```
[donnie@localhost ~]$ cd /dero_linux_amd64/
[donnie@localhost dero_linux_amd64]$ derod-linux-amd64 
-bash: /dero_linux_amd64/derod-linux-amd64: Operation not permitted
[donnie@localhost dero_linux_amd64]$
```

Even though you’re now invoking this program from a partition that isn’t mounted with the `noexec` option, you’re still not allowed to run it. That’s because it’s now being blocked by `fapolicyd`. That is, you can’t run it with your normal user privileges, even though both the directory and the executable file belong to you.

> There’s an idiosyncrasy with `fapolicyd` that I haven’t seen documented anywhere, and that I only found out by accident. That is, it will only block untrusted programs when a normal, unprivileged user tries to run them. But, you can run them just fine with the proper `sudo` privileges. (This is all the more reason to grant only limited `sudo` privileges to all but your most trusted administrators.)

Next, let’s see what we can do with the shell script:

```
[donnie@localhost bin]$ donnie_script.sh 
This is a test of the noexec option.
[donnie@localhost bin]$
```

So, why can I invoke this script here, but not in my home directory? It’s because in my home directory, the `noexec` mount option is blocking the script. But here in the `/usr/local/bin/` directory, we don’t have that mount option. Instead, all we have is just `fapolicyd`. We can use the `fapolicyd-cli -list` command to view the rules that are in effect, which might explain why I was able to run this script. (Note that formatting constraints don’t allow me to show the entire output.):

```
[donnie@localhost ~]$ sudo fapolicyd-cli -list
[sudo] password for donnie: 
. . .
. . .
11\. deny_audit perm=any all : ftype=%languages
12\. allow perm=any all : ftype=text/x-shellscript
13\. deny_audit perm=execute all : all
14\. allow perm=open all : all
[donnie@localhost ~]$
```

Look at rule number 12\. This rule allows shell scripts to run on all partitions that don’t have the `noexec` mount option, even by unprivileged users. That makes sense, considering that even unprivileged users make extensive use of shell scripts in order to automate repetitive tasks. But, if you’re absolutely certain that no unprivileged user will ever have cause to run shell scripts on a system, you can always disable that rule. And, in any case, you’d still be able to run shell scripts if you have the proper `sudo` privileges.

And, speaking of rules, let’s look at them next.

### Understanding the fapolicyd rules

The `fapolicyd` framework uses rules in the `/etc/fapolicyd/rules.d/` directory to create a list of programs that are either allowed or denied to execute on the system. When you install `fapolicyd`, you’ll get a set of default rules that are already set up and ready-to-go. If you need to allow more than what the default rules allow, you can create your own custom rules or add your desired program to the list of trusted applications.

In the `/etc/fapolicyd/rules.d/` directory, there are 11 rules files. Each one serves a different purpose:

```
[donnie@localhost ~]$ sudo ls -l /etc/fapolicyd/rules.d
[sudo] password for donnie: 
total 44
-rw-r--r--. 1 root fapolicyd 456 Dec 29 14:42 10-languages.rules
-rw-r--r--. 1 root fapolicyd 131 Dec 29 14:42 20-dracut.rules
-rw-r--r--. 1 root fapolicyd 192 Dec 29 14:42 21-updaters.rules
-rw-r--r--. 1 root fapolicyd 225 Dec 29 14:42 30-patterns.rules
-rw-r--r--. 1 root fapolicyd 101 Dec 29 14:42 40-bad-elf.rules
-rw-r--r--. 1 root fapolicyd 248 Dec 29 14:42 41-shared-obj.rules
-rw-r--r--. 1 root fapolicyd  71 Dec 29 14:42 42-trusted-elf.rules
-rw-r--r--. 1 root fapolicyd 143 Dec 29 14:42 70-trusted-lang.rules
-rw-r--r--. 1 root fapolicyd  96 Dec 29 14:42 72-shell.rules
-rw-r--r--. 1 root fapolicyd  76 Dec 29 14:42 90-deny-execute.rules
-rw-r--r--. 1 root fapolicyd  69 Dec 29 14:42 95-allow-open.rules
[donnie@localhost ~]$
```

The numbers at the beginning of the file names indicate the order in which these rules files will be processed, because the order in which the rules get processed really does matter. Rather than try to explain what these different classes of rules do for us, I’ll just let you open each file and read the contents. They’re all very short and include a comment to explain what each file does.

Although you can create custom rules for your own custom applications, that’s not the recommended method. For performance and safety reasons, it’s better to just add your application to the trusted list, like so:

```
[donnie@localhost ~]$ sudo fapolicyd-cli --file add /dero_linux_amd64/derod-linux-amd64
[sudo] password for donnie: 
[donnie@localhost ~]$
```

> I mentioned *safety* reasons because when you write your own custom rules, it’s easy to make a mistake that will lock up the entire system. You don’t have to worry about that so much if you’re just adding files to the trusted list.

This command adds the desired file, along with its associated SHA256 hash value, to the `/etc/fapolicyd/fapolicyd.trust` file, as we see here:

```
[donnie@localhost ~]$ sudo cat /etc/fapolicyd/fapolicyd.trust
[sudo] password for donnie: 
# AUTOGENERATED FILE VERSION 2
# This file contains a list of trusted files
#
#  FULL PATH        SIZE                             SHA256
# /home/user/my-ls 157984 61a9960bf7d255a85811f4afcac51067b8f2e4c75e21cf4f2af95319d4ed1b87
/dero_linux_amd64/derod-linux-amd64 16750936 847ea80b83a1df887d245085db60a9b0626aacb6cd4f0f192eb2e982643c5529
[donnie@localhost ~]$
```

To make this change take effect, we need to update the database and restart the `fapolicyd` service, like so:

```
[donnie@localhost ~]$ sudo fapolicyd-cli --update
[sudo] password for donnie: 
Fapolicyd was notified
[donnie@localhost ~]$ sudo systemctl restart fapolicyd
[sudo] password for donnie: 
[donnie@localhost ~]$
```

Now, when I invoke this application with my normal user privileges, it will run just fine:

```
[donnie@localhost ~]$ cd /dero_linux_amd64/
[donnie@localhost dero_linux_amd64]$ ./derod-linux-amd64 
02/01 16:13:31  INFO    derod   DERO HE daemon :  It is an alpha version, use it for testing/evaluations purpose only.
02/01 16:13:31  INFO    derod   Copyright 2017-2021 DERO Project. All rights reserved.
02/01 16:13:31  INFO    derod           {"OS": "linux", "ARCH": "amd64", "GOMAXPROCS": 1}
02/01 16:13:31  INFO    derod           {"Version": "3.5.2-114.DEROHE.STARGATE+01102022"}
. . .
. . .
```

So now, you’re likely wondering if you have to manually add each new application that you would install to the trusted list. Well, that depends upon how you install it. If you just download a compiled program as we did in the previous example, or compile one yourself, then yeah, you will have to manually add it to the trusted list. But, by default, every program that gets installed by the system package manager is automatically trusted. That means that if you use either `dnf` to install a package from the repository, or `rpm` to install an `rpm` package that you either downloaded or created, then the associated application is automatically trusted.

So far, we’ve looked at how the three *no* mount options and `fapolicyd` work together and complement each other. In this case, the mount options and `fapolicyd` all got set up automatically because we applied the STIG OpenSCAP profile as we installed the operating system. We can also install `fapolicyd` without the STIG profile, which is what we’ll look at next.

### Installing fapolicyd

Normally, `fapolicyd` isn’t automatically installed on AlmaLinux. In this case it was, because the STIG profile that we applied requires it as well as the restrictive mounting options for our partitions. To install `fapolicyd` on a system on which it hasn’t already been installed, just do:

```
[donnie@localhost ~]$ sudo dnf install fapolicyd
. . .
. . .
[donnie@localhost ~]$ sudo systemctl enable --now fapolicyd
Created symlink /etc/systemd/system/multi-user.target.wants/fapolicyd.service → /usr/lib/systemd/system/fapolicyd.service.
[donnie@localhost ~]$
```

There’s still a bit more about `fapolicyd` that I haven’t shown you, but I think you’ve seen enough to get the gist of it. To get more details about it and to see how to also use it as a file-integrity checker, be sure to visit the official Red Hat documentation for it. (The link is below in the *Further reading* section.)

> Adding the `noexec`, `nosuid`, and `nodev` mount options to your partitions works well, except that you can’t add them to all of your partitions. Obviously, you can’t add them to any partitions that are supposed to have executable files in them, or else your system would never work. The `fapolicyd` framework gives you a way to prevent rogue programs from running on those partitions, as long as the malicious intruder hasn’t already gained root privileges.

All right, let’s wrap this baby up.

## Summary

In this chapter, we looked at two ways to prevent untrusted programs from running on your systems. The first method, which can be used on any Linux distro, is to separate the various system and data directories into their own separate partitions, and then to mount each of these partitions with the appropriate combination of the `noexec`, `nosuid`, and `nodev` options. The second method, which so far is only available on Red Hat and its clones, is to use the `fapolicyd` framework. We saw how to automatically enable both of these methods by applying the STIG OpenSCAP profile as we install the operating system. Finally, we saw how to install `fapolicyd` separately, without having to apply the STIG profile.

In the next chapter, we’ll be wrapping things up with a quick look at various topics that didn’t neatly fit into any of the preceding chapters. I’ll see you there.

## Further reading

The bug in the RHEL installer: [https://forums.rockylinux.org/t/kernel-core-error-at-install/3683](https://forums.rockylinux.org/t/kernel-core-error-at-install/3683)

The STIG for Red Hat 8: [https://www.stigviewer.com/stig/red_hat_enterprise_linux_8/](https://www.stigviewer.com/stig/red_hat_enterprise_linux_8/)

Linux Ransomware: [https://phoenixnap.com/blog/linux-ransomware](https://phoenixnap.com/blog/linux-ransomware)

Linux File Access Policy Daemon (`fapolicyd`) video: [https://youtu.be/txThobi7oqc](https://youtu.be/txThobi7oqc)

Official `fapolicyd` documentation at Red hat: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_blocking-and-allowing-applications-using-fapolicyd_security-hardening](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_blocking-and-allowing-applications-using-fapolicyd_security-hardening)

## Questions

1.  Which of the following statements is true?
    1.  You can use the `noexec`, `nosuid`, and `nodev` mount options on any Linux distro.
    2.  You can use `fapolicyd` on any Linux distro.
    3.  You can prevent rogue programs from running by using the `noexec` mounting option on the `/` partition.
    4.  To use the `noexec`, `nosuid`, and `nodev` mount options, you can edit the `/etc/mtab` file.
2.  You need to run a program that `fapolicyd` normally won’t allow. What is the best way to deal with this?
    1.  Add it by hand-editing the `/etc/fapolicyd/fapolicyd.trust` file.
    2.  Add it by creating a custom rule.
    3.  Add it by running the `sudo fapolicyd-cli --file add` command.
    4.  Add it by hand-editing the `/etc/fapolicyd/fapolicyd.conf` file.
3.  When you apply the STIG OpenSCAP profile, what permissions settings will files and directories have when you create them?
    1.  644 for files, 755 for directories.
    2.  600 for files, 700 for directories.
    3.  640 for files, 750 for directories.
    4.  755 for files, 755 for directories.
4.  Which of the following is true about applying the STIG OpenSCAP profile?
    1.  You can apply the profile to any Linux operating system during the installation process.
    2.  Applying the STIG profile to the operating system during the installation process does everything for you.
    3.  Before you apply the STIG profile, you’ll need to set up a custom partition scheme to separate certain directories onto their own partitions.
    4.  On Red Hat-type systems, you can only apply the STIG profile after you’ve installed the system.
5.  What type of hash value does `fapolicyd` use in its `fapolicyd.trust` file?
    1.  SHA1
    2.  Blowfish
    3.  MD5
    4.  SHA256

## Answers

1.  a
2.  c
3.  b
4.  c
5.  d