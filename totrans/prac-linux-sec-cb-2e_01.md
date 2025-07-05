# Linux Security Problem

A Linux machine is only as secure as the administrator configures it to be. Once we have installed the Linux distribution of our choice and have removed all the unnecessary packages post installation, we can start working on the security aspect of the system by fine-tuning the installed software and services.

In this chapter, we will discuss the following topics:

*   Configuring server security
*   Security policy—server security
*   Defining security controls
*   Missing backup plans

The following recipes will be covered in the chapter:

*   Checking the integrity of installation medium using checksum
*   Using LUKS disk encryption
*   Making use of `sudoers`—configuring `sudo` access
*   Scanning hosts with Nmap
*   Gaining root on a vulnerable Linux system
*   Missing backup plans

# Security policy

A security policy is a definition that outlines the rules and practices to be followed for computer network security in an organization. How the organization should manage, protect, and distribute sensitive data is defined in the security policy.

# Developing a security policy

When creating a security policy you should keep in mind that it should be simple and easy for all the users to follow. The objective of the policy should be to protect the data while keeping the privacy of the users.

It should be developed around these points:

*   Accessibility to the system
*   Software installation rights on the system
*   Data permission
*   Recovery from failure

When developing a security policy, a user should be using only those services for which permission has been granted. Anything that is not permitted should be restricted in the policy. Let's look at some common Linux security myths.

# Linux security myths

You might feel nervous while planning to use Linux-based systems in your business. This may be due to some false rumors about security in Linux that the systems might have fallen prey to any of the myths out there.

# Myth – as Linux is open source, it is considered to be insecure

Linux, being a free and open source operating system, has its own advantages. It includes a large base of developers who constantly audit the source code for any possible security risks; the Linux community can provide fast support and fixes for any potential security problem. Patches are released quickly for testing by the community so they don't have to deal with the clumsy administration that other Unix vendors may have to deal with.

Due to the massive worldwide user base, Linux's security gets tested across huge range of computing environments, thus making it one of the most stable and secure operating systems. As Linux is open to scrutiny by developers across the world, it helps Linux derive superior security in the ways the privileges are assigned. The way in which these privileges are assigned in a Linux system is also a security feature derived from the open source code of the system.

# Myth – Linux is an experts-only system, and only they know how to configure their systems in terms of security

Assuming that Linux is for experts who know how to deal with viruses is a misconception. Linux has evolved to become one of the friendliest OSes that can be used by anyone, whether novice or experts.

Linux is secure because of its strong architecture. Regular users on a Linux system possess low-privileged accounts rather than having root privileges.

# Myth – Linux is virus free

Due to its strong architecture, even if a Linux system gets compromised, viruses would not have root access and thus will not be able to cause any major damage to the system.

Even on Linux servers, several levels of security are implemented and they are updated more often, again helping to secure the servers from viruses.

There are still a number of viruses that target Linux, thus making it not completely virus free. But most of the viruses that exist for Linux are non-destructive in nature.

# Configuring server security

Once a Linux server is created, the immediate next step is to implement security procedures to make sure that any kind of threat should not cause the system to be compromised. A major reason for malicious attacks on Linux servers have been poorly implemented security or existing vulnerabilities. When configuring a server, the security policies need to be implemented properly to create a secure environment that will help prevent your business from getting hacked.

# How to do it...

Let us have a look for each and every configuration.

# User management

Follow these steps to configure server security:

1.  When a Linux server is created, the first user created by default is always the root user. This root user should be used for initial configuration only.
2.  Once initial configuration is done, this root user should be disabled via SSH. This will make it difficult for any hacker to gain access to your Linux machine.
3.  Further, a secondary user should be created to log in and administer the machine. This user can be allowed sudo permissions if administrative actions need to be performed.

# Password policy

Follow these steps to configure server security:

1.  When creating user accounts, ensure the use of strong passwords. If allowed, keep the length of the password to between 12 to 14 characters.
2.  If possible, generate passwords randomly, and include lowercase and uppercase letters, numbers, and symbols.
3.  Avoid using password combinations that could be easily guessed, such as dictionary words, keyboard patterns, usernames, ID numbers, and so on.
4.  Avoid using the same password twice.

# Configuration policy

Follow these steps to configure server security:

1.  The operating system on the server should be configured in accordance with the guidelines approved for InfoSec.
2.  Any service or application not being used should be disabled, wherever possible.
3.  Every access to the services and applications on the server should be monitored and logged. It should also be protected through access-control methods. An example of this will be covered in [Chapter 3](dd9ad20f-3479-47df-8819-358a6d9354ca.xhtml), *Local Filesystem Security*.
4.  The system should be kept updated and any recent security patches, if available, should be installed as soon as possible
5.  Avoid using the root account as much as possible. It is better to use the security principles that require least access to perform a function.

6.  Any kind of privileged access must be performed over a secure channel connection (SSH) wherever possible.
7.  Access to the server should be in a controlled environment.

# Monitoring policy

1.  All security-related actions on server systems must be logged and audit reports should be saved as follows:

*   For a period of one month, all security-related logs should be kept online
*   For a period of one month, the daily backups, as well as the weekly backups should be retained
*   For a minimum of two years, the monthly full backups should be retained

2.  Any event related to security being compromised should be reported to the InfoSec team. They shall then review the logs and report the incident to the IT department.
3.  Some examples of security-related events are as follows:

*   Port-scanning-related attacks
*   Access to privileged accounts without authorization
*   Unusual occurrences due to a particular application on the host

# How it works...

Following the policies as given here helps in the base configuration of the internal server that is owned or operated by the organization. Implementing the policy effectively will minimize unauthorized access to any sensitive and proprietary information.

# Security policy – server security

A major reason for malicious attacks on Linux servers has been poorly implemented security or existing vulnerabilities. When configuring a server, the security policies need to be implemented properly and ownership needs to be taken for proper customization of the server.

# How to do it…

Let's have a look and various security policies

# General policy

Let's discuss the various security policies:

1.  The administration of all the internal servers in an organization is the responsibility of a dedicated team that should also keep watch for any kind of compliance issues. If a compliance issues occurs, the team should immediately review and implement an updated security policy.
2.  When configuring internal servers, they must be registered in such a way that the identification of the servers can be done on the basis of the following information:
    *   Location of the server
    *   Operating system version and hardware configuration
    *   Services and applications running on the server
3.  Any kind of information in the organization's management system must always be kept up to date.

# Configuration policy

Let's discuss the various security policies:

1.  The operating system on the server should be configured in accordance with the guidelines approved for InfoSec.
2.  Any service or application not being used should be disabled, wherever possible.
3.  Every access to the services and applications on the server should be monitored and logged. It should also be protected through access-control methods. An example of this will be covered in [Chapter 3](dd9ad20f-3479-47df-8819-358a6d9354ca.xhtml), *Local FileSystem Security.*
4.  The system should be kept updated and any recent security patches, if available, should be installed as soon as possible
5.  Avoid using the root account as much as possible. It is better to use security principles that require least access to perform a function.
6.  Any kind of privileged access must be performed over a secure channel connection (SSH), wherever possible.
7.  Access to the server should be in a controlled environment.

# Monitoring policy

Let's discuss the various security policies:

1.  All security-related actions on server systems must be logged and audit reports should be saved as follows:
    *   For a period of one month, all the security-related logs should be kept online
    *   For a period of one month, the daily backups, as well as the weekly backups, should be retained
    *   For a minimum of two years, the monthly full backups should be retained
2.  Any event related to security being compromised should be reported to the InfoSec team. They shall then review the logs and report the incident to the IT department.
3.  Some examples of security related events are as follows:
    *   Port-scanning-related attacks
    *   Access to privileged accounts without authorization
    *   Unusual occurrences due to a particular application on the host

# How it works…

Following the policies as given here helps the base configuration of the internal server that is owned or operated by the organization. Implementing the policy effectively will minimize unauthorized access to any sensitive and proprietary information.

# Defining security controls

Securing a Linux server starts with the process of hardening the system, and to do this it's important to define a list of security controls. A security controls list (or security checklist) confirms that proper security controls have been implemented.

# How to do it...

Let's have a look at various security control checklists.

# Installation

Now we will look into each security control checklist:

*   Installation media such as CD-ROM/DVD/ISO should be checked by using checksum
*   A minimal base installation should be done when creating the server
*   It is good practice to create separate filesystems for `/home`, and `/tmp`
*   It is good practice to install minimum software on the server to minimize the chances of vulnerability
*   Always keep the Linux kernel and software up to date

# Boot and disk

Now we will look into each security control checklist:

*   Encrypt partitions using disk encryption methods such as LUKS.
*   Limit access to BIOS by configuring a BIOS password.
*   Limit bootable devices and allow only devices such as disk to be booted.
*   Configure a password to access the single user mode boot loader.

# Network and services

Now we will look into each security control checklist:

*   Determine the services running by checking the open network ports.
*   Use a firewall such as `iptables/nftables` to limit access to the services as per need.
*   Encrypt all data transmitted over the network.
*   Avoid using services such as FTP, Telnet, and Rlogin/Rsh.
*   Any unwanted services should be disabled.
*   A centralized authentication service should be used.

# Intrusion detection and Denial of Service (DoS)

Now we will look into each security control checklist:

*   File integrity tools such as AIDE, Samhain, and AFICK should be installed and configured for monitoring important files.
*   Use a malware scanner such as CalmAV to protect against malicious scripts.
*   Configure system logging to a remote machine for the purpose of detection, forensics, and archiving.
*   Deter brute-force attacks by using anti brute-force tools for authentication attempts.

# Auditing and availability

Now we will look into each security control checklist:

*   Read through logs to monitor for suspicious activity.
*   Configure auditd configuration to perform system accounting.
*   Ensure backup is working, and also check restores.

# How it works...

Implementing these security controls minimizes the security risk to your Linux server. This helps protect your data from the hands of hackers.

# Checking the integrity of installation medium by using checksum

Whenever you download an image file of any Linux distribution, it should always be checked for correctness and safety. This can be done by generating an MD5 hash after downloading the image file and then comparing the generated hash with the hash generated by the organization supplying the image file.

This helps in checking the integrity of the downloaded file. If the original file was tampered with it can be detected using the MD5 hash comparison. The larger the file size, the higher the possibility of changes in the file. It is always recommended you do an MD5 hash comparison for files such as the operating system installation CD.

# Getting ready

`md5sum` is normally installed in most Linux distributions, so installation is not required.

# How to do it…

Perform the following steps:

1.  Open the Linux Terminal and then change the directory to the folder containing the downloaded ISO file.

Because Linux is case sensitive, type the correct spelling for the folder name. Downloads are not the same as downloads in Linux.

2.  After changing to the download directory, type the following command:

```
md5sum ubuntu-filename.iso
```

md5sum will then print the calculated hash in a single line as shown here:

```
8044d756b7f00b695ab8dce07dce43e5 ubuntu-filename.iso
```

Now we can compare the hash calculated by this command with the hash on the UbuntuHashes page ([https://help.ubuntu.com/community/UbuntuHashes](https://help.ubuntu.com/community/UbuntuHashes)). After opening the UbuntuHashes page, we just need to copy this previously calculated hash, in the Find box of the browser (by pressing *Ctrl* + *F*).

# How it works…

If the calculated hash and the hash on the UbuntuHashes page match, then the downloaded file is not damaged. In case the hashes don't match, then there is a possibility that the file might be tampered or is damaged. Try downloading the file again. If the issue still persists, it is recommended you report the issue to the administrator of the server.

# See also

Here’s something extra in case you want to go the extra mile: the GUI checksum calculator available for Ubuntu.

Sometimes, it’s really inconvenient to use the Terminal for doing checksums. You need to know the right folder of the downloaded file and also the exact filename. This makes it difficult to remember the exact commands.

As a solution, there is the very small and simple software – **GtkHash**.

You can download the tool here: [http://gtkhash.sourceforge.net/](http://gtkhash.sourceforge.net/).

Or you can install it by using the following command:

```
sudo apt-get install gtkhash 
```

# Using LUKS disk encryption

In enterprises, small business, and government offices, the users may have to secure their systems in order to protect their private data, which includes customers details, important files, contact details, and so on. To help with this, Linux provides a good number of cryptographic techniques that can be used to protect data on physical devices such as hard disk or removable media. One such cryptographic technique is using **Linux Unified Key Setup** (**LUKS**)-on-disk-format. This technique allows the encryption of Linux partitions.

This is what LUKS does**:**

*   The entire block device can be encrypted using LUKS; it's well suited for protecting the data on removable storage media or the laptop disk drives
*   LUKS uses the existing device mapper kernel subsystem
*   It also provides passphrase strengthening, which helps protect against dictionary attacks

# Getting ready

For the following process to work, it is necessary that a separate partition is also created while installing Linux, which will be encrypted using LUKS.

Configuring LUKS using the steps given will remove all data on the partition being encrypted. So, before starting the process of using LUKS, make sure you take a backup of the data to some external source.

# How to do it...

To begin with manually encrypting directories, perform the following steps:

1.  Install `cryptsetup` as shown here, which is a utility used for setting up encrypted filesystems:

```
apt-get install cryptsetup
```

The preceding command generates the following output:

![](img/b167735b-b91c-4f98-a760-a9c7b363ed2f.png)

2.  Encrypt your `/dev/sdb1` partition, which is a removable device. To encrypt the partition, type the following command:

```
cryptsetup -y -v luksFormat /dev/sdb1
```

The preceding command generates the following output:

![](img/3439b7f1-428c-4d08-adb6-018e8b74c545.png)

This command initializes the partition and also sets a passphrase. Make sure you note the passphrase for further use.

3.  Now open the newly created encrypted device by creating a mapping:![](img/5bee70ce-7cbf-4f52-b45c-8e19eca573d0.png)
4.  Check to confirm that the device is present:

```
ls -l /dev/mapper/backup2
```

The preceding command generates the following output:

![](img/c5331396-f706-4d28-be1b-488f94acdb5a.png)

5.  Check the status of the mapping using the following command:![](img/b5ef027e-03c5-4fd3-af32-500fc11f1606.png)
6.  Dump LUKS headers using the following command:![](img/4b7ead46-bd0c-4a7a-9e96-2d7f076f8459.png)
7.  Next, write zeros to `/dev/mapper/backup2` encrypted device:![](img/9be3c11f-79b0-4bde-b9ff-314ce6f5b67a.png)

As the `dd` command may take hours to complete, we use the `pv` command to monitor the progress.

8.  Now create a filesystem:

```
mkfs.ext4 /dev/mapper/backup2
```

The preceding command generates the following output:

![](img/6a252565-418a-4f73-9840-20bca3a3c53e.png)

9.  Then mount the new filesystem and confirm the filesystem is visible:![](img/18c573e9-648b-4096-a4c8-d5bfc735a7bb.png)![](img/32a3072d-be95-4186-98f4-df53e0539ce3.png)

Congratulations! You have successfully created an encrypted partition. Now, you can keep all your data safe, even when the computer is off.

# There's more...

Perform the following commands to unmount and secure the data on the partition:

```
umount /backup2
cryptsetup luksClose backup
```

To remount the encrypted partition, perform the following steps:

```
cryptsetup luksOpen /dev/xvdc backup2
mount /dev/mapper/backup2 /backup2
df -H
mount
```

![](img/10b38d18-558f-45fc-9ab3-eb70f0f1dd66.png)

# Make use of sudoers – configuring sudo access

Sudoer is the functionality of the Linux system that can be used by an administrator to provide administrative access to a trusted regular user, without actually sharing the root user's password. The administrator simply needs to add the regular user in the sudoers list.

Once a user has been added to the sudoers list, they can execute any administrative command by preceding it with sudo. Then the user would be asked to enter their own password. After this, the administrative command would be executed the same way as by the root user.

# Getting ready

As the file for the configuration is pre-defined and the commands used are inbuilt, nothing extra is needed to be configured before starting the steps.

# How to do it…

Perform the following steps:

1.  You will first create a normal account and then give it sudo access. Once done, you will be able to use the `sudo` command from the new account and then execute the administrative commands. Follow the steps given to configure sudo access. First, use the root account to log in to the system then create a user account using the `useradd` command, as shown. Replace USERNAME in the command with any name of your choice:

![](img/6abf9ec2-b568-468b-bf57-ad9ccf6536f5.jpg)

2.  Now, using the `passwd` command set a password for the new user account, as shown:![](img/a918c81c-b939-4008-ac05-75a13ae8b01b.jpg)

3.  Now edit the `/etc/sudoers` file by running the `visudo` as shown. The policies applied when using the `sudo` command, are defined by the `/etc/sudoers` file:![](img/d0551b98-bb15-4f75-9b5d-4ca86eeda37c.jpg)

4.  Once the file is open in the editor, search for the following lines which allow sudo access to the users in the test group:![](img/aaf80c52-3f18-42d5-b8e9-486b96fe231b.jpg)

5.  You can enable the given configuration by deleting the comment character (`#`) at the beginning of the second line. Once the changes are done, save the file and exit from the editor. Now using the `usermod` command, add the previously created user to the test group:![](img/996f892a-30c6-4e92-9905-948079b84720.jpg)

6.  Now you need to check whether the configuration created now allows the new user account to run commands using `sudo`.

7.  To switch to the newly created user account, use the `su` option:![](img/ebbb55fa-77af-4c4c-910d-c294150c29e3.jpg)

8.  Now use the `groups` command to confirm the presence of the user account in the test group:![](img/305ed1ed-cbc3-4b28-94cb-572d20f86aa4.jpg)

Finally, run the `whoami` command with `sudo` from the new account. As you have executed a command using `sudo` for the first time using this new user account, the default banner message will be displayed for the `sudo` command. The screen will also ask for the user account password to be entered:

![](img/718f4426-3745-48de-83db-b0750f256cbf.jpg)

9.  The last line of the output shown is the username returned by the `whoami` command. If `sudo` is configured correctly this value will be root.

You have successfully configured a user with sudo access. You can now log in to this user account and use `sudo` to run commands the same way as you would from the root user.

# How it works…

When you create a new account, it does not have the permission to run administrator commands. However, after editing the `/etc/sudoers` file, and making appropriate entry to grant sudo access to the new user account, you can start using the new user account to run all administrator commands.

# There’s more…

Here are some extra measures that you can take to ensure total security.

# Vulnerability assessment

A vulnerability assessment is the process of auditing your network and system security, through which you can come to know about the confidentiality, integrity, and availability of your network. The first phase in vulnerability assessment is reconnaissance, and this further leads to the phase of system readiness, in which we mainly check for all known vulnerabilities in the target. Next follows the phase of reporting in which we group all the vulnerabilities found into categories of low, medium, and high risk.

# Scanning hosts with Nmap

Nmap, which can be used for scanning a network, is one of the most popular tools included in Linux. It has been in existence for many years, and is currently one of the preferred tools for gathering information about a network. Nmap can be used by administrators on their networks to find any open ports and the host systems. When performing vulnerability assessments, Nmap is surely a tool not to be missed.

# Getting ready

Most Linux versions come with Nmap installed. The first step is to check whether you already have it by using the following command:

```
    nmap --version
```

If Nmap exists, you should see output similar to this:

![](img/ace504d0-5e56-4838-9d43-486b78ee3b1f.png)

If Nmap is not already installed, you can download and install it from this link: [https://nmap.org/download.html.](https://nmap.org/download.html)

The following command will quickly install Nmap on your system:

```
sudo apt-get install nmap
```

# How to do it...

Follow these steps for scanning hosts with Nmap:

1.  The most common use of Nmap is to find all the hosts online within a given IP range. The default command used takes some time to scan the complete network, depending on the number of hosts in the network.
2.  The following screenshot shows an example:![](img/6d0f40a6-9684-4bf9-8dba-0ab67c72254c.png)

3.  To perform a SYN scan on a particular IP from a subnet, use the following command:![](img/bfcfa8dc-27e9-4c7e-857a-219e59cf9215.png)
4.  If SYN scan does not work properly, you can also use Stealth scan:![](img/e98d5af2-a766-4b63-8bf4-6277d0a4591c.png)

5.  To detect the version number of the services running on the remote host, you can perform Service Version Detection scan as follows:![](img/2edf882c-0432-4645-9d68-7aee019184c0.png)
6.  If you want to detect the operating system running on the remote host, run the following command:

```
nmap -O 192.168.1.102
```

![](img/538f3a50-00ef-404a-a8e2-4de873a3e642.png)

7.  The output here has been truncated:![](img/f6f73b30-6cc2-47b8-81dd-e857bc631921.png)
8.  If you wish to scan only for a particular port, such as `80`, run the command:![](img/9fdc5fa0-42c5-4377-81bf-e4f892b59866.png)

# How it works...

Nmap checks for the services that are listening by testing the most common network communication ports. This information helps the network administrator to close all unwanted or unused ports and services. The previous examples show how to use port scanning and Nmap as a powerful tool to study the network around us.

# See also

Nmap also has scripting features that we can use to write custom scripts. These scripts can be used with Nmap to automate and extend the scanning capabilities of Nmap.

You can find more information about using Nmap at its official homepage: [https://nmap.org/](https://nmap.org/). [](https://nmap.org/) 

# Gaining root on a vulnerable Linux system

When trying to learn how to scan and exploit a Linux machine, one major problem we encounter is where to try. For this purpose, the Metasploit team has developed and released a virtual machine called *Metasploitable*. This machine has been made vulnerable purposefully, having many services running unpatched. Due to this, it has become a great platform for practicing or developing penetration testing skills. In this section, we will learn how to scan a Linux system and then, using the scanning result, how to find a service that is vulnerable. Using that vulnerable service, we shall gain root access to the system.

# Getting ready

Kali Linux and the Metasploitable VMware system will be used in this section. The image file of Metasploitable can be downloaded from these links:

*   [http://sourceforge.net/projects/metasploitable/files/Metasploitable2/](http://sourceforge.net/projects/metasploitable/files/Metasploitable2/)
*   [https://images.offensive-security.com/virtual-images/kali-linux-2018.2-vm-i386.zip](https://images.offensive-security.com/virtual-images/kali-linux-2018.2-vm-i386.zip)

# How to do it...

The Metasploit Framework is an open source tool used by security professionals globally to perform penetration tests by executing exploit code on target systems from within the framework. It comes pre-installed with Kali Linux (the preferred choice of distribution for security professionals).

Follow these steps to gain root access to a vulnerable Linux system:

1.  First open the Metasploit console on the Kali system by running the following command:

```
service postgresql start
msfconsole
```

![](img/2dd9b486-0260-495a-93cd-b1df58aef53d.png)

2.  At the bottom of the screen, you should get the Metasploit framework prompt denoted by `msf>`.
3.  Next, we need to scan the target, which is `192.168.0.102` in this example, using Nmap:

The following screenshot shows the output of the command:

![](img/c45f87d1-d84e-4f21-8fcc-6e9cce8ae326.png)

4.  In the previous command, you can see there are many services running on different ports. Among them you can see FTP is also running on port `21`.
5.  We will focus on the FTP service for now. From the output shown, you can see that the FTP service is provided by the vsftpd application version 2.3.4.
6.  Now lets try to find an exploit for `vsftpd` within the Metasploit framework by simply executing the command `search` `vsftpd.` Here is the output:

![](img/da12ffaa-43b9-4a7a-9dbb-08dce8378529.png)

7.  The search results are showing a module, VSFTPD Backdoor Command Execution, with an excellent rating, which means that this exploit will work perfectly fine.

8.  Now run the following commands to use the exploit and check its options:![](img/00538b6b-cc3d-4fb6-bf02-5f2b8cd9e63e.png)

9.  As you can see from the screenshot, you need to set the value of `RHOST`, which is `192.168.1.102` in our case.
10.  Set the value for `RHOST` and then run the exploit as shown here:

![](img/cb65f99b-068b-4222-a564-394a8002ff95.png)

11.  Once the exploit runs successfully, you will get root access, as shown in the preceding screenshot.

# How it works...

We first did an Nmap scan to check for running services and open ports and found the FTP service running. Then we tried to find the version of the FTP service. Once we got the information, we searched for any exploit available for VSFTPD. The VSFTPD backdoor module that was found in the search result is actually a code that is being sent to the target machine by the Metasploit framework. The code gets executed on the target machine due to a module of the VSFTPD being improperly programmed. Once the code gets executed, we get a root shell access on our Kali machine

Using the exploit found for VSFTPD, we tried to attack the target system and got the root shell on it.

# There's more...

Let's learn about a few more exploits and attacks that are common in Linux.

# Missing backup plans

In this era of malicious attacks and dangerous cyberattacks, your data is never safe. Your data needs something more than just protection. Its needs insurance in the form of backups. At any point of time, if your data is lost, having data backups ensures that your business can be up and running in no time.

# Getting ready

When we talk about data backup in Linux, choosing the best backup tool that matches your business needs is essential. Everyone needs to have a data backup tool that is dependable, but it's not necessary to spend too much to get a tool that has features that meets your needs. The backup tool should allow you to have local backups, remote backups, one-time backups, scheduled backups, and many other features.

# How to do it...

Let's look at a few outstanding backup tools for Linux.

# fwbackups

This is the easiest of all Linux backup tools. fwbackups has a user-friendly interface and it can be used for single backups and also for recurring scheduled backups.

Local as well as remote backups can be done in various formats, such as `tar`, `tar.gz`, `tar.bz`, or `rsync` format. A single file or an entire computer can be backed up using this tool.

Using this tool, backup and restoring can be done easily. Incremental or differential backups can be done to speed the process.

# rsync

This is one of the most widely used backup solutions for Linux. It can be used for incremental backups, whether local or remote.

`rsync` can be used to update directory trees and filesystems while preserving links, ownerships, permissions, and privileges.

Being a command-line tool, `rsync` is perfect for creating simple scripts to use in conjunction with `cron`, so as to create automated backups.

# Amanda (Advanced Maryland Automatic Network Disk Archiver)

This is a free and open source tool developed for "*moderately sized computer centers*". It is designed for performing the backup of multiple machines over the network to tape drives, disks, or optical disks.

Amanda can be used to backup about everything on a diverse network, using a combination of a master backup server and Linux or Windows.

LVM snapshots and hardware snapshots can also be handled using this tool.

# Simple Backup Solution (SBS)

Primarily targeted at desktop backup, SBS can be used to backup files and directories. It also allows regular expressions to be used for exclusion purposes.

It includes pre-defined backup configurations that can be used to back up directories such as `/var/`, `/etc/`, `/usr/local`.

SBS can be used for custom backups, manual backups and scheduled backups, and is not limited to just pre-defined backups.

# Bacula

Bacula is a free and open source tool and requires client programs to be installed on each system targeted for backup. All these systems are controlled using a server that centrally handles the backup rules.

Bacula has its own file format, which is not proprietary as the tool is open source.

Routine full and incremental backups can be done using the tool and it offers better support for setups if multiple servers are being used with their own tape drives.

Encryption and RAID is supported by Bacula. Scripting language for customizing your backup jobs is also offered by Bacula, which can be used to incorporate encryption.

# How it works...

A backup tool is necessary for anyone in the IT industry or a computer power user. The backup tool should be capable of scheduled backups, one-time backups, local backups, remote backups, and many other features.