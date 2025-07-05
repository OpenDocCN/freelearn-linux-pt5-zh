# Scanning and Auditing Linux

In this chapter, we will discuss the following:

*   Installing an antivirus on Linux
*   Scanning with ClamAV
*   Finding rootkits
*   Using the auditd daemon
*   Using ausearch and aureport to read logs
*   Auditing system services with systemctl

# Installing an antivirus on Linux

In today's environment, viruses and malicious threats can be found in any system, including Linux. So, as a system administrator, we can use an antivirus on our Linux servers.

**ClamAV**, is one such open source antivirus software, for detecting and removing viruses, Trojans, malware, and other threats on Linux systems.

# Getting ready

ClamAV can be installed from the default repositories of Ubuntu. However, if we want to install it from the source, we can download the official source code from: [http://www.clamav.net/download.html.](http://www.clamav.net/download.html)

# How to do it...

In this section, we will see how to install ClamAV antivirus, on our Ubuntu server:

1.  Before beginning with the installation of the tool, we will update the repository, by running the following command:

![](img/c2b4aacd-d50d-4c20-adf7-762ab2f6ab83.png)

2.  Next, we will run the following command to install ClamAV antivirus:

![](img/0121c472-38b7-45c9-8ebf-cd5377bd54b6.png)

![](img/5fc8438b-d102-429d-bcd0-17a67f980363.png)

3.  We can also install the GUI for the same tool, using the following command:

![](img/777972f1-47f5-4ca0-909a-7498a7e18aea.png)

4.  Once the installation completes, we can check the version of the package installed by running the following command:

![](img/17c757fb-9e36-43cf-9765-86c28569ab48.png)

5.  The command-line version can be used by using `clamscan` followed by the appropriate options.
6.  To open the GUI version of ClamAV, go to the main menu, and search for the tool, as shown here:

![](img/cc03fc20-fb38-4c56-8f52-31d90b5031bb.png)

7.  When we open the GUI version, it will open as shown here:

![](img/6c69c9d2-0f42-4a84-9610-5946fb592d05.png)

# How it works...

ClamAV can be easily installed on a Linux system, either by using the default repositories of Linux, or by downloading the source code from its official website.

ClamAV can be used from the command line as well as from the GUI.

# Scanning with ClamAV

**ClamAV** is a cross-platform antivirus software that is capable of detecting different types of malware, including viruses. It includes various utilities such as a command-line scanner, a database updater, and a multi-threaded daemon, making it a powerful tool.

# Getting ready

We have to install either the command-line version or the GUI version of the tool, before we can run a scan on our system. The tool can be installed as discussed in the previous section.

# How to do it...

In this section, we will see how to use ClamAV to perform a scan; as per our requirements.

1.  As a first step, we will check the Help menu of the tool, to see the different options supported by ClamAV, as shown here:

![](img/994cc242-2d72-459e-9546-c6caba67bd8a.png)

2.  As seen in the following screenshot, ClamAV supports various options to be used during scanning:

![](img/b86e75dd-4d77-4460-8fb1-b4925aa1feca.png)

3.  We will now start the scan on the `/home` directory, as shown here:

![](img/2bcd0143-9511-4230-b33f-0fffc1624b73.png)

4.  Once the scan completes, it shows a scan summary as follows:

![](img/de95d551-0bbf-4a3d-b2a4-9218c666e5ff.png)

5.  We can also run the scan using the GUI version. After opening the tool, we change the scan settings, by clicking on Settings. This will open a window as shown here:

![](img/9c776481-b7fa-4f4f-96e8-6b53f5ced102.png)

In the preceding window, check or uncheck the options as per our requirement.

6.  Next, click on Scan a file or Scan a directory to begin the scan accordingly:

![](img/8b725f78-30fa-4830-97e1-ac613967bade.png)

7.  The scan will start running as shown here:

![](img/e4f75832-5b9f-41ac-abb8-0cf338398b34.png)

8.  When the scan completes, it will either display the findings or else show a the following message, if no threats are found:

![](img/01ace307-c67c-4052-8ba5-adbad03696b4.png)

9.  By clicking on Update, we can check for signature updates available for the software:

![](img/47dacb3d-cf88-4e48-b215-192b004d3dee.png)

# How it works...

**ClamAV** is a versatile tool and supports multiple file formats and multiple signature languages, which most viruses would use to exploit systems. It can perform multithreaded scans.

# Finding rootkits

Servers that are connected to the internet nowadays face a constant daily attacks. As a system administrator, it is recommended you keep a check regularly to ensure that no attacker has been able to get in.

By using different tools, we can keep a check on malware and rootkits, from getting installed on our servers.

# Getting ready

There are no specific requirements to use the scanning tools on our Linux system.

# How to do it...

In this section, we will see how to install and configure Linux rootkit scanning tools and use as per our requirements:

1.  To begin with, we will install `chkrootkit`, a classic rootkit scanner for Linux, as shown here:

![](img/cdfadde7-8f57-4f2a-8860-4bcd75546e23.png)

2.  Once the software has been installed, we can check the path where the software has been installed by running the following command:

![](img/c47c2f6c-1545-4538-be7f-ec0bacb7980d.png)

3.  Next, we check the Help menu to understand the options that can be used to run the tool:

![](img/69a3f2d4-714a-46c1-baf2-1fd0a8b90f71.png)

4.  If we want to see the list of available tests in chkrootkit, we can run the following command:

![](img/6085af77-e95f-4966-8189-3b57f7515b75.png)

5.  Now, let's start the scan as shown here:

![](img/817e7d19-d931-42a4-ace0-7830d7300f2b.png)

6.  As we can see in the scan output, the software is checking for all known rootkit signatures:

![](img/07a656cf-e224-43b3-9c84-756c54fd2283.png)

7.  Another well-know tool that can be used for scanning rootkits is `rkhunter`. Install the tool by running the following command:

![](img/2566bf41-5426-46ba-a127-0e81a08ca044.png)

8.  Next, check the Help menu to see the options that can be used when running the software:

![](img/3afcc33d-9bd9-470e-8bbd-e8b3a2d40bee.png)

9.  Now, start the scan as shown here:

![](img/750cfdbd-ff6e-4689-9261-2caccc64185a.png)

10.  As seen in the output, all known rootkit signatures have been checked and none were found:

![](img/fde92c07-f971-46a8-8d44-75c9f7b4946c.png)

11.  Finally, when the scan completes, the tool will show a scan summary as seen here:

![](img/0c11f4ea-83f5-43e5-8628-0f1b2e5a1a11.png)

# How it works...

Chkrootkit and rkhunter are both open source Linux-based rootkit scanner tools that help in scanning for rootkits, which may be present in the Linux machine.

Both the tools use signature based scanning to check for rootkits and any other malware on the Linux system.

# Using the auditd daemon

When we talk about securing a system, this it includes many procedures and auditing the system is one of them. The Linux system has a preinstalled tool named **auditd**, which is responsible for writing audit records on to the disk.

# Getting ready

There are no specific requirements to use auditd on a Linux system.

# How to do it...

In this section, we will see how to use auditd, for the purpose of auditing:

1.  If the tool is not already installed on our Linux distribution, we can install it by running the following command:

```
apt-get install auditd
```

2.  When the package is installed, it also installs a few other tools as part of the installation process. One of the tools installed is `auditctl` which helps in controlling the behavior of the software and also in adding rules.

3.  We can check the version of the tool by running the following command:

![](img/764dc05a-b374-4f20-b7f4-52439f17df93.png)

4.  When auditd is installed for the first time, it does not have any rules available yet. This can be checked by running the following command:

![](img/aa9b87a7-ae10-4789-b65f-87fe1aaaba5c.png)

5.  Now, let's see the Help menu to check for other options that can be used with the tool:

![](img/13e67e3d-9846-4fc8-9f12-620802d4bfe8.png)

6.  To start using the `auditd` tool, it is necessary to have rules. We can add rules for auditing a file as shown here:

![](img/3d9a2f51-01cf-48a6-acb2-a0446e4ef2a6.png)

In the previous command, the `-w` option will tell auditd to keep a watch on the file specified. The `p` option specifies the permissions for which auditd should trigger. And then `wxa` refers to read, write, execute, and attribute, respectively.

7.  We can also add rules for keeping a watch on directories, as shown here:

![](img/36b77be8-3610-4cca-8f99-483869db5c4a.png)

8.  If we now check the list of rules, we get the following output:

![](img/adf32b9c-ee8d-496f-9634-8c4dd623c4b1.png)

# How it works...

`auditd` helps in defining rules, based on which it will keep a watch on the files and directories specified. If any changes are made to those files and directories, then `auditd` will trigger based on the rules that have been defined.

# Using ausearch and aureport to read logs

In the previous section, we have seen how the auditd tool can be used to define rules and keep watch on particular files and directories.

To retrieve data from the auditd log files, we can use the `ausearch` tool and by using `aureport`, we can generate reports based on these logs.

`ausearch` is a command-line tool that is used to search the log files of the auditd daemon on the basis of events and other search criteria.

Similary, `aureport` is also a command-line tool that helps in creating useful summary reports from the log files of the audidt daemon.

# Getting ready

When we install the auditd daemon, it will also install the ausearch and aureport tool along with it. So no extra installation is needed to use these tools.

# How to do it...

In this section, we will see how to use ausearch and aureport tools to read the log files of the auditd daemon and create reports from them:

1.  The default location to find the logs of auditd is `/var/log/audit/audit.log`. If we view the content of this file, we get an output as shown here:

![](img/66f71006-0deb-484e-8a39-914dc407a01d.png)

As we can see in this output, the log contains lots of data, and us it is difficult to get a specific information from this file, just by viewing its content.

2.  Hence, we will use `ausearch` to search through the logs in a more powerful and efficient way. First, we check the help file of the tool to understand the options that can be used:

![](img/2307eb9d-8a64-4749-aaf8-1f672b04bb47.png)

3.  Suppose we want to check the logs related to a particular running process; we can do this by using the `-p` flag and passing the process ID to the `ausearch` command, as shown here:

![](img/f25213f0-8ff1-4d6e-ad8e-7aefe661c798.png)

As we can see in this output, now the information is displayed only for the particular process ID.

4.  If we want to check failed login attempts of the user account, we can do so by running the following command:

![](img/4ce998d3-b9a9-46a7-96b5-497915899be4.png)

5.  To find the user activity of any particular user account, we can run the following command:

![](img/15f515d7-21f5-41e7-887a-008b2b5e9870.png)

In the preceding command, `pentest` is the username we want to query for.

6.  We can also use `ausearch` to query for the actions performed by any user in a given period of time. In the following command, we use `-ts` for start date/time and `-te` for end date/time:

![](img/5ed92c4b-385a-47c8-ba73-2a833d5b3a46.png)

7.  If we want to create a report based on the audit rule keys, added by the auditd daemon, we can use the following command, using the `-k` flag:

![](img/1e2c139a-f761-4afc-bb08-79159a8e6a38.png)

9.  If we want to convert numeric entities into text (such as UID to account name), in the report created by using the preceding command, we can add the `-i` flag, as shown here:

![](img/581dd5aa-c190-458c-bc60-eb3eb325569c.png)

10.  To create a report regarding events related to user authentication, we can use the following command:

![](img/560574b8-aa7c-4fb9-bfe5-d077335141f0.png)

11.  To create a report of all logins, we use the `-l` flag as shown here:

![](img/21855408-41f4-4f6e-a433-867025a9d906.png)

12.  If we want to see a report of failed login events, we can use the following command:

![](img/2c6e9a29-ee6a-4b8d-8ac1-66939d1032ea.png)

13.  Similar to `ausearch`, we can use `aureport` to create a report for a specific period of time, as shown here:

![](img/bf39122d-bdc6-4fbf-8947-587aa036633a.png)

# How it works...

ausearch and aureport work along with the auditd daemon. Using the log files where auditd logs the event data, ausearch can help us read through the logs as per our requirements. Similarly, using aureport, we can create useful reports based on the log files of the auditd daemon.

# Auditing system services with systemctl

**Systemd** is an init system and also a system manager, and it has become the new standard for Linux systems. To control this init system, we have a central management tool, called systemctl. Using systemctl, we can check services status, manage the services, change their states, and work with their configuration files.

# Getting ready

Most of the Linux distributions have implemented systemctl, so it comes preinstalled.

If any particular Linux distribution does not have it preinstalled, this implies that the particular Linux distribution is not using the init system.

# How to do it...

In this section, we will discuss how to use the systemctl command to perform various actions on the services:

1.  To confirm if our Linux distribution supports systemctl, we can just run the command `systemctl`, as shown here:

![](img/a9305c09-0b3a-47fd-a230-c9dccd83de7d.png)

If we get output as shown here, it confirms that the command is working. If we receive an error, `bash: systemctl is not installed`, it implies the system does not support the command as it is using some other init system.

2.  If we want to check the status of any particular service, such as SSHD service, we can use `systemctl` as shown here:

![](img/1e26d107-46b6-41e7-b573-f9bf1d6edc5d.png)

The output shown clearly tells us that the SSHD service is running fine.

3.  To stop or start any service, we use the following commands:

![](img/2a5c0549-d27a-4f62-94a8-38329d29aa25.png)

4.  We can use `systemctl` to restart a running service. Also, if any particular service supports reloading its configuration files (without restarting), we can do so using the `reload` option with the `systemctl` command, as shown here:

![](img/982cd36a-4f87-4d48-a64b-f841cb8b25cd.png)

5.  We can use the `systemctl` command to see the list of all active units that systemd knows about, as shown here:

![](img/9902bd99-d69a-49c1-990e-68071c6f69e4.png)

6.  At times, we may want to see a particular service's dependency tree. This can be done by using the `systemctl` command as shown here:

![](img/6840785c-7d81-452a-8f9c-bc1a3dcbf026.png)

# How it works...

`systemctl` allows us to interact with and control the `systemd` instance. We use `systemctl` utility for any type of service and system state management.

Using different options with the `systemctl` command, we can perform different activities with the services.