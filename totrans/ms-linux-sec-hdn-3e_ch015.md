# 14 Vulnerability Scanning and Intrusion Detection

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file83.png)

There are lots of threats out there, and some of them might even penetrate your network. You'll want to know when that happens, so you'll want to have a good **Network Intrusion Detection System** (**NIDS**) or **Network Intrusion Prevention System** (**NIPS**) in place. In this chapter, we'll look at Snort, which is probably the most famous one. Then, I'll show you a way to cheat so that you can have a good NIDS/NIPS up and running in no time at all. I'll also show you a quick and easy way to set up an edge firewall appliance, complete with a built-in NIPS.

We've already learned how to scan a machine for viruses and rootkits by installing scanning tools on the machines that we want to scan. However, there are a lot more vulnerabilities for which we can scan, and I'll show you some cool tools that you can use for that.

The following topics will be covered in this chapter:

*   Introduction to Snort and Security Onion
*   IPFire and its built-in Intrusion Prevention System (IPS)
*   Scanning and hardening with Lynis
*   Finding vulnerabilities with the Greenbone Security Assistant
*   Web server scanning with Nikto

So, if you're ready, let's begin by digging into the Snort Network Intrusion Detection System.

## Introduction to Snort and Security Onion

Snort is a **Network Intrusion Detection System** (**NIDS**), which is offered as a free open source software product. The program itself is free of charge, but you'll need to pay if you want to have a complete, up-to-date set of threat detection rules. Snort started out as a one-man project, but it's now owned by Cisco. Understand, though, this isn't something that you install on the machine that you want to protect. Rather, you'll have at least one dedicated Snort machine someplace on the network, just monitoring all network traffic, watching for anomalies. When it sees traffic that shouldn't be there – something that indicates the presence of a bot, for example – it can either just send an alert message to an administrator or it can even block the anomalous traffic, depending on how the rules have been configured. For a small network, you can have just one Snort machine that acts as both a control console and a sensor. For large networks, you could have one Snort machine set up as a control console and have it receive reports from other Snort machines that are set up as sensors.

Snort isn't too hard to deal with, but setting up a complete Snort solution from scratch can be a bit tedious. After we look at the basics of Snort usage, I'll show you how to vastly simplify things by setting up a pre-built NIPS appliance.

> Space doesn't permit me to present a comprehensive tutorial about Snort. Instead, I'll present a high-level overview and then present you with other resources if you want to learn about Snort in detail.
> 
> > Also, you might be wondering how a NIDS and a NIPS are different. Well, a NIDS is supposed to do nothing but alert network administrators about the suspicious network traffic that it detects. A NIPS will not only alert the administrator, but will also automatically block the suspicious traffic. However, the lines between the two types of systems are somewhat blurred, because some systems that are marketed as a NIDS can be configured to function as a NIPS.

First, let's download and install Snort.

### Obtaining and installing Snort

**Snort 3**, the newest version of Snort, isn't in the official repository of any Linux distro. So, you'll need to get it from the Snort website. It used to be available as installer packages for either Windows or Linux, but that’s no longer the case. Now, with the introduction of Snort 3, it’s available either as source code that you’ll need to compile yourself or as a pre-built Docker container. Oddly, there’s no mention of the container option on the Snort home page, and I only found it after doing a DuckDuckGo search.

> You can get Snort and Snort training from the official Snort website: [https://www.snort.org](https://www.snort.org).

#### Hands-on lab – installing Snort via a Docker container

You’ll definitely want to go with the container option instead of the source code option. That’s because the directions for setting up the source code option aren’t as clear as they should be, and one particular library package doesn’t always compile properly. Instead of using the official Docker software, I’ll be showing you how to use Podman, which is Red Hat’s drop-in replacement for Docker. Podman’s security is better than that of Docker, and it’s available for pretty much every Linux distro. Podman is already installed on your AlmaLinux 8 and 9 virtual machines, but you’ll need to install it yourself on Ubuntu.

On Ubuntu only, install the `podman` package:

```
sudo apt update
sudo apt install podman
```

On Ubuntu only, open the `/etc/containers/registries.conf` file in your text editor. Find this line:

```
# unqualified-search-registries = ["example.com"]
```

Change it to this:

```
unqualified-search-registries = ["docker.io"]
```

Download and start the container:

```
podman run --name snort3 -h snort3 -u snorty -w /home/snorty -d -it ciscotalos/snort3 bash
```

Next, enter the container so that you can interact with the snort commands:

```
podman exec -it snort3 bash
```

If this command executes successfully, you’ll find yourself at the `snorty@snort3` command prompt.

Validate the Snort configuration with this single-word command:

```
snort
```

Snort requires a set of rules that define the potential problems that it should analyze. Paying customers will receive up-to-date rulesets, while non-paying users can download rulesets that are about one month behind. An old ruleset from 2018 comes with the Docker container, so you’ll want something that’s a bit more recent. You won’t be able to download the rulesets directly to your container, so you’ll need to download them to either your virtual machine or to your host machine, and then transfer them to the container. On either your host machine or in another terminal that’s connected to the virtual machine, download the latest community ruleset, like this:

```
wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz
```

You can’t use `scp` or `sftp` to connect to the container from the virtual machine or your host machine, but you can use them to connect to the virtual machine or host machine from the container. So, from within the container, use `sftp` to transfer in the new ruleset file. Your commands should look something like this:

```
sftp donnie@192.168.0.20
get snort3-community-rules.tar.gz
bye
```

While still within the container, unarchive the ruleset file and transfer the new ruleset to its proper location:

```
snort3-community-rules.tar.gz
cd snort3-community-rules
cp snort3-community.rules ~/snort3/etc/rules/
```

Test things out by examining a `.pcap` file that’s included with the example files:

```
snort -q --talos --rule-path snort3/etc/rules/ -r examples/intro/lab2/eternalblue.pcap
```

Take a look at the tutorial videos at the Snort website. You can see them here:

[https://www.snort.org/resources](https://www.snort.org/resources)

When you’re done, type exit to get out of the container. To shutdown the container, do this:

```
podman kill snort3
```

End of lab

Here are some of the significant differences between the new Snort 3 and the older Snort 2 that I covered in previous editions of this book.

*   There were several cool graphical front-ends that you could install for Snort 2, but there aren’t any for Snort 3\. So, the new Snort 3 is strictly a command-line mode program.
*   Snort 3 can save its output files in `.json` format, which makes it easy for centralized log aggregators to read and parse them.
*   The configuration files and rules files for Snort 3 are in `.yaml` format.
*   The Snort 3 rules syntax has been somewhat streamlined, making rules easier to write.

Next, let’s look at a cool appliance that has another Intrusion Detection System built into it.

## Using Security Onion

Security Onion consists of a set of **Free Open-source Software** (**FOSS**) tools that you can install on your own local Linux machine. It’s also offered as a pre-built Linux distro image, which is really the preferred method of installation. In the previous editions of this book, I showed you the original version of Security Onion, which was built on Xubuntu Linux. This version had a graphical desktop interface, used Snort 2 as the IDS, and included several graphical front-ends for Snort. The new Security Onion 2 is a completely different animal. It’s now built on a text-mode installation of CentOS 7, and offers way more functionality over the original version. In addition to using it as an IDS/IPS, you can now use it as a forensics analyzer, a log file aggregator, and a log file analyzer. For log file collection and analysis, it includes the ELK stack.

> ELK stands for **Elastic Search**, **Logstash**, and **Kibana**. Logstash, used with the appropriate collection agents on the end-points that you want to monitor, collects log files from the network end-points. Elastic Search stores the log messages in a searchable database. Kibana is the web-based graphical component that displays the collected log messages.

Instead of Snort, Security Onion 2 now uses Suricata, which is a Snort alternative. In place of the graphical front-ends for Snort, Security Onion 2 now uses the **Security Onion Console**, which is a web-based front-end.

For a couple of reasons, I’m not going to provide a hands-on lab for this. In the first place, there’s no real point, because you’ll find detailed tutorials on Security Onion’s YouTube channel. Also, there’s a good chance that any hands-on lab that I were to provide here would be outdated by the time you read this. That’s because the next version of Security Onion, which will be released some time in 2023, will be based on Rocky Linux 9 instead of CentOS 7\. So, I’m sure that the installation and usage procedures will be somewhat different from what they are now.

> You can download Security Onion 2 from here: [https://securityonionsolutions.com/](https://securityonionsolutions.com/)
> 
> > You can find the Security Onion YouTube channel here: [https://www.youtube.com/@security-onion](https://www.youtube.com/@security-onion)
> 
> > Fee-based support options, training options, and physical appliances with Security Onion pre-installed are also available.

In lieu of a lab, allow me to leave you with a screenshot of the Security Onion Console:

![Figure 14.1: The Security Onion Console](img/file84.png)

Figure 14.1: The Security Onion Console

Now, let’s turn our attention to a cool pre-built firewall appliance that also has its own Intrusion Prevention System.

## IPFire and its built-in Intrusion Prevention System (IPS)

When I wrote the original edition of this book, I included a discussion of IPFire in the Snort section. At that time, IPFire had Snort built into it. It was a neat idea because you had an edge firewall and an **Intrusion Detection System** (**IDS**) all in one handy package. But, in the summer of 2019, the IPFire folk replaced Snort with their own IPS. So, I've moved IPFire down here into its own section.

> The difference between an IDS and an IPS is that an IDS informs you of problems, but doesn't block them. An IPS also blocks them.

If you think back to our discussion of firewalls in *Chapter 4*, *Securing Your Server with a Firewall – Part 1* and *Chapter 5*, *Securing Your Server with a Firewall -- Part 2*, I completely glossed over any discussion of creating the **Network Address Translation** (**NAT**) rules that you would need in order to set up an edge or gateway type of firewall. That's because there are several Linux distros and BSD distros that have been created specifically for this purpose. One such distro is **IPFire**.

![Figure 14.2: IPFire installer](img/file85.png)

Figure 14.2: IPFire installer

IPFire is completely free of charge, and it only takes a few minutes to set up. You install it on a machine with at least two network interface adapters and configure it to match your network configuration. It's a proxy type of firewall, which means that in addition to doing normal firewall-type packet inspection, it also includes caching, content filtering, and NAT capabilities. You can set up IPFire in a number of different configurations:

*   On a computer with two network interface adapters, you can have one connected to the internet and the other connected to the internal LAN.
*   With three network adapters, you can have one connection to the internet, one to the internal LAN, and one to the **Demilitarized Zone** (**DMZ**), where you have your Internet-facing servers.
*   With a fourth network adapter, you can have all of what we just mentioned, plus protection for a wireless network.

With all that said, let's give it a try.

### Hands-on lab – Creating an IPFire virtual machine

You won't normally run IPFire in a virtual machine. Instead, you'll install it on a physical machine that has at least two network interfaces. But, just for the sake of letting you see what it looks like, setting it up in a virtual machine will do for now. Let's get started:

> You can download IPFire from their website: [https://www.ipfire.org/](https://www.ipfire.org/)

1.  Create a virtual machine with two network interfaces. Set one to Bridged mode and leave the other in NAT mode. Install IPFire into this virtual machine. During the setup portion, select the Bridged adapter as the Green interface and select the NAT adapter as the Red interface.
2.  After you install IPFire, you'll need to use the web browser of your normal workstation to navigate to the IPFire dashboard. Do this with this URL:

`https://192.168.0.190:444`

1.  (Of course, substitute your own IP address for your Green interface.)
2.  Under the **Firewall** menu, you'll see an entry for **Intrusion Prevention**. Click on that to get to this screen, where you can enable Intrusion Prevention. The first step for that is to click on the **Add provider** button that’s under the **Ruleset Settings** section.

    ![Figure 14.3: Click the Add provider button](img/file86.png)

    Figure 14.3: Click the Add provider button

3.  On the next page, select the ruleset that you want to use. Leave the **Enable automatic updates** checkbox enabled. Then, hit the **Add** button:

    ![Figure 14.4: Select the ruleset](img/file87.png)

    Figure 14.4: Select the ruleset

4.  You will then see this screen, where you'll select the interfaces for which you want to enable intrusion prevention. (Select both interfaces.) Then, select the **Enable Intrusion Prevention System** checkbox and click on Save:

![Figure 14.5: Enable the IPS](img/file88.png)

Figure 14.5: Enable the IPS

If all goes well, you'll see the following output:

![Figure 14.6: With the IPS enabled](img/file89.png)

Figure 14.6: With the IPS enabled

1.  In the **Ruleset Settings** section, click the **Customize ruleset** button. On the next page, click on the rules that you want to enable. Then, at the bottom of the screen, click the **Apply** button:

    ![Figure 14.7: Select the desired rules](img/file90.png)

    Figure 14.7: Select the desired rules

2.  View what's going on with the IPS by selecting **Log/IPS Logs**. (Note that what you see will depend upon which rules that you’ve chosen to enable. Even then, it might take a while for any entries to show up.)

    ![Figure 14.8: Look at the IPS logs](img/file91.png)

    Figure 14.8: Look at the IPS logs

3.  Click on the other menu items to view IPFire's other features.

You have completed this lab – congratulations!

You've just seen the easy way to set up an edge firewall with its own network IPS. Now, let's look at some scanning tools.

## Scanning and hardening with Lynis

Lynis is yet another FOSS tool that you can use to scan your systems for vulnerabilities and bad security configurations. It comes as a portable shell script that you can use not only on Linux, but also on a variety of different Unix and Unix-like systems. It's a multipurpose tool that you can use for compliance auditing, vulnerability scanning, or hardening. Unlike most vulnerability scanners, you install and run Lynis on the system that you want to scan. According to the creator of Lynis, this allows for more in-depth scanning.

The Lynis scanning tool is available as a free-of-charge version, but its scanning capabilities are somewhat limited. If you need all that Lynis has to offer, you'll need to purchase an enterprise license.

### Installing Lynis on Red Hat/CentOS

Red Hat, CentOS 7, and AlmaLinux 8/9 users will find an up-to-date version of Lynis in the EPEL repository. So, if you have EPEL installed, as I showed you in *Chapter 1*, *Running Linux in a Virtual Environment*, installation is just a simple matter of doing:

```
sudo yum install lynis
```

### Installing Lynis on Ubuntu

Ubuntu has Lynis in its own repository, but it's just a little bit behind what's current. If you're okay with using an older version, the command to install it is:

```
sudo apt install lynis
```

> If you want the newest version for Ubuntu or if you want to use Lynis on operating systems that don't have it in their repositories, you can download Lynis from [https://cisofy.com/downloads/lynis/](https://cisofy.com/downloads/lynis/).
> 
> > The cool thing about this is that once you download it, you can use it on any Linux, Unix, or Unix-like operating system. (This even includes macOS, which I confirmed by running it on my old Mac Pro that was running with macOS High Sierra.)

Since the executable file is nothing but a common shell script, there's no need to perform an actual installation. All you need to do is extract the archive file, `cd` into the resultant directory, and run Lynis from there:

```
tar xzvf lynis-3.0.8.tar.gz
cd lynis
sudo ./lynis -h
```

The `lynis -h` command shows you the help screen, along with all of the Lynis commands that you need to know about.

### Scanning with Lynis

Lynis commands work the same regardless of which operating system that you want to scan. The only difference is that if you're running it from the archive file that you downloaded from the website, you would `cd` into the `lynis` directory and precede the `lynis` commands with a `./`. (That's because, for security reasons, your own home directory isn't in the path setting that allows the shell to automatically find executable files.)

To scan your system that has Lynis installed, execute this command:

```
sudo lynis audit system
```

To scan a system that you just downloaded the archive file on, execute these commands:

```
cd lynis
sudo ./lynis audit system
```

Running Lynis from the shell script in your home directory presents you with this message:

```
donnie@ubuntu:~/lynis$ sudo ./lynis audit system
. . .
[X] Security check failed
    Why do I see this error?
    -------------------------------
    This is a protection mechanism to prevent the root user from executing user created files. The files may be altered, or including malicious pieces of script.
   . . .
[ Press ENTER to continue, or CTRL+C to cancel ]
```

This isn't hurting anything, so you can just hit Enter to continue. Or, if seeing this message really bothers you, you can change ownership of the Lynis files to the root user, as the message tells you. For now, I'll just press Enter.

Running a Lynis scan in this manner is similar to running an OpenSCAP scan against a generic security profile. The major difference is that OpenSCAP has an automatic remediation feature, while Lynis doesn't. Lynis tells you what it finds and suggests how to fix what it perceives to be a problem, but it doesn't fix anything for you.

Space doesn't permit me to show the entire scan output, but I can show you a couple of example snippets:

```
[+] Boot and services
------------------------------------
  - Service Manager                                           [ systemd ]
  - Checking UEFI boot                                        [ DISABLED ]
  - Checking presence GRUB                                    [ OK ]
  - Checking presence GRUB2                                   [ FOUND ]
    - Checking for password protection                        [ WARNING ]
  - Check running services (systemctl)                        [ DONE ]
        Result: found 21 running services
  - Check enabled services at boot (systemctl)                [ DONE ]
        Result: found 28 enabled services
  - Check startup files (permissions)                         [ OK ]
```

This warning message shows that I don't have password protection for my GRUB2 bootloader. That may or may not be a big deal because the only way someone can exploit it is to gain physical access to the machine. If it's a server that's locked away in a room that only a few trusted individuals can access, then I'm not going to worry about it, unless rules from an applicable regulatory agency dictate that I do. If it's a desktop machine that's out in an open cubicle, then I would definitely fix that. (We'll look at GRUB password protection in *Chapter 16*, *Security Tips and Tricks for the Busy Bee*.)

In the **File systems** section, we can see some items with the **SUGGESTION** flag next to them:

```
[+] File systems
------------------------------------
  - Checking mount points
    - Checking /home mount point                              [ SUGGESTION ]
    - Checking /tmp mount point                               [ SUGGESTION ]
    - Checking /var mount point                               [ SUGGESTION ]
  - Query swap partitions (fstab)                             [ OK ]
  - Testing swap partitions                                   [ OK ]
  - Testing /proc mount (hidepid)                             [ SUGGESTION ]
  - Checking for old files in /tmp                            [ OK ]
  - Checking /tmp sticky bit                                  [ OK ]
  - ACL support root file system                              [ ENABLED ]
  - Mount options of /                                        [ NON DEFAULT ]
```

Exactly what Lynis suggests comes near the end of the output:

```
. . .
. . .
  * To decrease the impact of a full /home file system, place /home on a separated partition [FILE-6310]
      https://cisofy.com/controls/FILE-6310/
  * To decrease the impact of a full /tmp file system, place /tmp on a separated partition [FILE-6310]
      https://cisofy.com/controls/FILE-6310/
  * To decrease the impact of a full /var file system, place /var on a separated partition [FILE-6310]
      https://cisofy.com/controls/FILE-6310/
. . .
. . .
```

The last thing we'll look at is the scan details section at the end of the output:

```
 Lynis security scan details:
  Hardening index : 67 [#############       ]
  Tests performed : 218
  Plugins enabled : 0
  Components:
  - Firewall               [V]
  - Malware scanner        [X]
  Lynis Modules:
  - Compliance Status      [?]
  - Security Audit         [V]
  - Vulnerability Scan     [V]
  Files:
  - Test and debug information      : /var/log/lynis.log
  - Report data                     : /var/log/lynis-report.dat
```

For **Components**, there's a red **X** by **Malware Scanner**. That's because I don't have ClamAV or maldet installed on this machine, so Lynis couldn't do a virus scan.

For **Lynis Modules**, we see a question mark by **Compliance Status**. That's because this feature is reserved for the Enterprise version of Lynis, which requires a paid subscription. As we saw in the previous chapter, you have OpenSCAP profiles to make a system compliant with several different security standards, and it doesn't cost you anything. With Lynis, you have to pay for the compliance profiles, but you have a wider range to choose from. For example, Lynis Enterprise can scan for Sarbanes-Oxley compliance issues, while OpenSCAP can’t.

The last thing I want to say about Lynis is in regard to the Enterprise version. In the following screenshot, which is from their website, you can see the current pricing and the differences between the different subscription plans:

![Figure 14.9: Pricing for Lynis Enterprise](img/file92.png)

Figure 14.9: Pricing for Lynis Enterprise

As you see, you have choices.

> You'll find information about pricing on the Cisofy website: [https://cisofy.com/pricing/](https://cisofy.com/pricing/)

That pretty much wraps things up as regards our discussion of Lynis. Next, we'll look at an external vulnerability scanner.

## Finding vulnerabilities with the Greenbone Security Assistant

In the previous versions of this book, I told you about OpenVAS, which stands for Open Vulnerability Assessment Scanner. It’s still with us, but its publisher has changed the name to **Greenbone Security Assistant** (**GSA**). Although it’s a commercial product, Greenbone also offers a Free Open-source Community Edition that’s free-of-charge.

The Greenbone Security Assistant is something that you would use to perform remote vulnerability scans. You can use it to scan a variety of network devices.

The big three security distros are Kali Linux, Parrot Linux, and Black Arch. They're aimed at security researchers and penetration testers, but they contain tools that would also be good for just a normal security administrator of either the Linux or Windows variety. GSA is one such tool. All three of these security distros have their unique advantages and disadvantages. Since Kali is the most popular, we'll go with it for the demos.

> You can download Kali Linux from [https://www.kali.org/get-kali/](https://www.kali.org/get-kali/).

When you go to the Kali download page, you'll see lots of choices. You can download a normal installer image for `x86`, `x86_64`, and Apple Silicon. Other options include:

*   Images for ARM devices, such as the Raspberry Pi
*   The Cloud
*   Pre-built virtual machine images for VMWare, VirtualBox, and QEMU
*   Pre-built Docker containers
*   Images for mobile devices
*   Windows Subsystem for Linux

Kali is built from Debian Linux, so installing it and keeping it updated is pretty much the same as installing and updating Debian.

> Greenbone Security Assistant is a rather memory-hungry program, so if you're installing Kali in a virtual machine, be sure to allocate at least 3 GB of memory.

The first thing you'll want to do after installing Kali is to update it, which is done in the same way that you'd update any Debian/Ubuntu-type of distro. Then, install GSA, like this:

```
sudo apt update
sudo apt dist-upgrade
sudo apt install openvas
```

Note that the `openvas` package is a **transitional package** that will automatically install all of the proper Greenbone packages.

After the GSA installation completes, you'll need to run a script that will create the security certificates and download the vulnerability database:

```
sudo gvm-setup
```

This will take a long time, so you might as well go grab a sandwich and a coffee while it's running. When it's finally done, you'll be presented with the password that you'll use to log in to GSA. Write it down and keep it in a safe place:

![Figure 14.10: Copy the password](img/file93.png)

Figure 14.10: Copy the password

Next, start the Greenbone services by doing:

```
sudo gvm-check-setup
```

To ensure that everything works properly, you’ll need to manually sync the data feeds, and then restart the GVA services:

```
sudo greenbone-feed-sync --type GVMD_DATA
sudo greenbone-feed-sync --type SCAP
sudo greenbone-feed-sync --type CERT
sudo gvm-stop
```

Wait for 30 seconds, and then restart the services:

```
sudo gvm-start
```

Once the service startup has completed, open Firefox and navigate to [https://localhost:9392](https://localhost:9392). You'll get a security alert because GVA uses a self-signed security certificate, but that's okay. Just click on the **Advanced** button, and then click on **Add Exception**.

On the login page, enter `admin` as the user and then enter the password that was generated by the `gvm-setup` script.

![Figure 14.11: The GVA login screen](img/file94.png)

Figure 14.11: The GVA login screen

There's all kinds of fancy stuff that you can do with GVA, but for now, we'll just look at how to do a basic vulnerability scan. To begin, select **Tasks** from the **Scans** menu on the GVA dashboard:

![Figure 14.12: Select Tasks](img/file95.png)

Figure 14.12: Select Tasks

When the Tasks page comes up, look for the little magic wand at the upper left-hand corner. Roll your mouse cursor over this wand, and you’ll see the various choices for the Task Wizard:

![Figure 14.13: Task Wizard choices](img/file96.png)

Figure 14.13: Task Wizard choices

For now, we'll just select the **Task Wizard** option, which will choose all of the default scan settings for us. The only thing you need to do here is enter the IP address of the machine that you want to scan, and then start the scan:

![Figure 14.14: Start a basic scan](img/file97.png)

Figure 14.14: Start a basic scan

The scan will take some time, so you might as well go grab a drink:

![Figure 14.15: Performing a basic scan](img/file98.png)

Figure 14.15: Performing a basic scan

The type of scan that you're doing is named **Full and Fast**, which is the most comprehensive type of scan that’s now offered. To select another type of scan and to configure other scan options, use the **Advanced Task Wizard**, as shown here:

![Figure 14.16: Selecting the scan options](img/file99.png)

Figure 14.16: Selecting the scan options

When the scan has completed, click on the **Scans/Results** menu item:

![Figure 14.17: View the results](img/file100.png)

Figure 14.17: View the results

For the sake of showing you some interesting stuff, I purposely chose a target machine that’s nearly 20 years old, with an outdated operating system and lots of vulnerabilities. Here, you see that the machine is using weak encryption algorithms for Secure Shell, which is classified as medium severity. Even worse is that it supports SSH version 1, which is classified as a high severity problem. Yikes!

![Figure 14.18: Scan results](img/file101.png)

Figure 14.18: Scan results

You also want to pay attention to the items that aren't flagged as vulnerabilities. For example, the **VNC security types** item shows that port `5900` is open. This means that the **Virtual Network Computing** (**VNC**) daemon is running, which allows users to remotely log in to this machine's desktop. If this machine were an Internet-facing machine, that would be a real problem because there's no real security with VNC like there is with Secure Shell.

By clicking on a vulnerability item, I can see an explanation of the vulnerability:

![Figure 14.19: An explanation of a vulnerability](img/file102.png)

Figure 14.19: An explanation of a vulnerability

Keep in mind that the target machine, in this case, is a desktop machine. If it were a server, there's a good chance that we'd see even more problems.

And that pretty much wraps things up for the Greenbone Security Assistant. As I mentioned previously, there's a lot of awesome stuff that you can do with it. However, what I've shown you here should be enough to get you started. Play around with it and try out the different scan options to see the difference in results.

> If you want to find out more about Kali Linux, you'll find a great selection of books about it on the Packt Publishing website.

Okay, you now know how to do a vulnerability scan with GSA. Now, let's look at a scanner that's specifically designed for web servers.

## Web server scanning with Nikto

The Greenbone Security Assistant, which we just looked at, is a general-purpose vulnerability scanner. It can find vulnerabilities for most any kind of operating system or for most any server daemon. However, as we've just seen, a GSA scan can take a while to run, and it might be more than what you need.

Nikto is a special-purpose tool with only one purpose. That is, it's meant to scan web servers, and only web servers. It's easy to install, easy to use, and capable of doing a comprehensive scan of a web server fairly quickly.

### Nikto in Kali Linux

If you have Kali Linux, you'll find that Nikto is already installed under the **Vulnerability Analysis** menu:

![Figure 14.20: Nikto on the Kali Linux menu](img/file103.png)

Figure 14.20: Nikto on the Kali Linux menu

However, your best bet is to ignore it, and instead use the more up-to-date version that you’ll download directly from GitHub. That’s because the Nikto signature database that’s installed on Kali hasn’t been updated since 2019, as you see here:

```
┌──(kali㉿kali)-[~]
└─$ cd /var/lib/nikto/databases 

┌──(kali㉿kali)-[/var/lib/nikto/databases]
└─$ ls -l
total 1652
-rw-r--r-- 1 root root    2093 Mar  9  2019 db_404_strings
-rw-r--r-- 1 root root    3147 Mar  9  2019 db_content_search
-rw-r--r-- 1 root root   15218 Mar  9  2019 db_dictionary
. . .
. . .
-rw-r--r-- 1 root root    4868 Mar  9  2019 db_variables

┌──(kali㉿kali)-[/var/lib/nikto/databases]
└─$ 
```

You used to be able to update the database with the `sudo nikto -update` command, but that no longer works because the author has deprecated the `-update` option. (I had hoped that doing a normal `sudo apt dist-upgrade` command would bring in some updates, but no such luck.) Now, the author recommends using the `git` commands to download and update Nikto. So, let’s look at how to do that.

#### Hands-on lab--Installing Nikto from Github

To make things easy, we’ll do this on Kali, because it already has all of the `perl` modules that Nikto needs to operate. If you do this on Debian or Ubuntu, it should work, but you’ll need to chase down the `perl` modules that it needs yourself. And, forget about doing this on AlmaLinux, because the necessary `perl` modules aren’t even in any of the AlmaLinux or EPEL repositories. (There’s an alternate way to install them, but that’s beyond the scope of this book.)

1.  In your normal user home directory, clone the Nikto repository. Then, `cd` into the `nikto` directory, and checkout the current branch:

```
git clone https://github.com/sullo/nikto.git
cd nikto
git checkout nikto-2.5.0
```

1.  To run Nikto, cd into the program subdirectory, and invoke Nikto from there. For example, to see the Nikto help screen, do this:

```
cd program
./nikto -help
```

1.  Periodically, you’ll want to update the Nikto signature databases. Just `cd` into the `nikto` directory and do:

```
git pull
```

1.  End of lab

Next, let’s do something useful with Nikto.

#### Scanning a web server with Nikto

1.  To do a simple scan, use the `-h` option to specify the target host, like this:

```
cd nikto/program
./nikto -h 192.168.0.9
./nikto -h www.example.com
```

1.  Let's look at some sample output. Here's the top part:

```
+ Allowed HTTP Methods: POST, OPTIONS, GET, HEAD
+ OSVDB-396: /_vti_bin/shtml.exe: Attackers may be able to crash FrontPage by requesting a DOS device, like shtml.exe/aux.htm -- a DoS was not attempted.
+ /cgi-bin/guestbook.pl: May allow attackers to execute commands as the web daemon.
+ /cgi-bin/wwwadmin.pl: Administration CGI?
+ /cgi-bin/Count.cgi: This may allow attackers to execute arbitrary commands on the server
```

At the top, we can see that there's an `shtml.exe` file present, which is supposedly for the FrontPage web authoring program. I have no idea why it's there, considering that this is a Linux server and that that's a Windows executable. Nikto is telling me that by having that file there, someone could possibly do a **Denial of Service** (**DOS**) attack against this site.

Next, we can see that there are various scripts in the `/cgi-bin/` directory. You can see from the explanatory messages that that's not a good thing because it could allow attackers to execute commands on my server.

Let's look at the second part:

```
+ OSVDB-28260: /_vti_bin/shtml.exe/_vti_rpc?method=server+version%3a4%2e0%2e2%2e2611: Gives info about server settings.
+ OSVDB-3092: /_vti_bin/_vti_aut/author.exe?method=list+documents%3a3%2e0%2e2%2e1706&service%5fname=&listHiddenDocs=true&listExplorerDocs=true&listRecurse=false&listFiles=true&listFolders=true&listLinkInfo=true&listIncludeParent=true&listDerivedT=false&listBorders=fals: We seem to have authoring access to the FrontPage web.
```

Here, we can see that there's an `author.exe` file in the `vti_bin` directory, which could theoretically allow someone to have authoring privileges.

And now, the final part:

```
+ OSVDB-250: /wwwboard/passwd.txt: The wwwboard password file is browsable. Change wwwboard to store this file elsewhere, or upgrade to the latest version.
+ OSVDB-3092: /stats/: This might be interesting...
+ OSVDB-3092: /test.html: This might be interesting...
+ OSVDB-3092: /webstats/: This might be interesting...
+ OSVDB-3092: /cgi-bin/wwwboard.pl: This might be interesting...
+ OSVDB-3233: /_vti_bin/shtml.exe/_vti_rpc: FrontPage may be installed.
+ 6545 items checked: 0 error(s) and 15 item(s) reported on remote host
+ End Time:           2017-12-24 10:54:21 (GMT-5) (678 seconds)
```

The final item of interest is the `passwd.txt` file that's in the `wwwboard` directory. Apparently, this password file is browsable, which is definitely not a good thing.

Now, before you accuse me of making these problems up, I will reveal that this is a scan of a real production website on a real hosting service. (And yes, I do have permission to scan it.) So, these problems are real and need to be fixed.

Here are a couple of other sample messages that I got from scanning a web server that's running WordPress:

```
HTTP TRACK method is active, suggesting the host is vulnerable to XST
Cookie wordpress_test_cookie created without the httponly flag
```

To cut a long story short, both of these two problems could potentially allow an attacker to steal user credentials. The fix, in this case, would be to see whether the WordPress folk have issued any updates that would fix the problem.

So, how can we protect a web server against these kinds of vulnerabilities? Let's see:

*   As we saw in the first example, you want to ensure that you don't have any risky executable files on your web server. In this case, we found two `.exe` files that might not hurt anything on our Linux server, since Windows executable files don't run on Linux. On the other hand, it could be a Linux executable that's disguised as a Windows executable. We also found some `perl` scripts that definitely would run on Linux and that could pose a problem.
*   In the event that someone were to plant some malicious script on your web server, you'll want to have some form of mandatory access control, such as SELinux or AppArmor, that would keep the malicious scripts from accessing things that they shouldn’t access. (See *Chapter 10*, *Implementing Mandatory Access Control with SELinux and AppArmor*, for details of that).
*   You may also consider installing a web application firewall, such as ModSecurity. Space doesn't permit me to cover the details of ModSecurity, but you'll find a book that covers it on the Packt Publishing website.
*   Keep your systems updated, especially if you're running a PHP-based content management system such as WordPress. If you keep up with the IT security news, you'll see stories about WordPress vulnerabilities more often than you'd like to.

> I can’t reveal the URL of the site that I scanned here, but you can download a vulnerable virtual machine from [https://www.vulnhub.com/](https://www.vulnhub.com/)
> 
> > Choose a virtual machine to download, and then import it into VirtualBox. To do that, choose **Import Appliance** under the **File** menu.

There are other scan options that you can see by just typing `./nikto` at the command line. For now though, this is enough to get you started with basic web server scanning.

## Summary

We've reached yet another milestone in our journey, and we saw some cool stuff. We started with a discussion about the basics of setting up Snort as a NIDS. Then, I showed you how to seriously cheat by deploying a specialty Linux distro that already has a NIDS set up and ready to go. As a bonus, I showed you a quick and easy edge firewall appliance that comes with a built-in Network Intrusion Prevention System.

Next, I introduced you to Lynis and how you can use it to scan your system for various vulnerabilities and compliance issues. Finally, we wrapped things up with working demos of the Greenbone Security Assistant and Nikto.

In the next chapter, we'll look at how to block certain applications from running. I'll see you there.

## Questions

1.  Which of the following best describes IPFire?
    1.  A host-based firewall appliance with a built-in Network Intrusion Detection System
    2.  An edge firewall appliance with a built-in Network Intrusion Detection System
2.  Which of the following utilities is best for scanning Sarbanes-Oxley compliance issues?
    1.  Lynis
    2.  Lynis Enterprise
    3.  Greenbone Security Assistant
    4.  OpenSCAP
3.  Which of the following best represents what Snort is?
    1.  HIDS
    2.  GIDS
    3.  NIDS
    4.  FIDS
4.  Which of the following would you use as a general-purpose, external vulnerability scanner?
    1.  Greenbone Security Assistant
    2.  Nikto
    3.  OpenSCAP
    4.  Lynis
5.  Which of these problems would you be most likely to find with a Nikto scan?
    1.  That the Samba service is running, although it shouldn't be
    2.  That the root user account is exposed to the Internet via SSH
    3.  That potentially malicious scripts reside in a CGI directory
    4.  That the root user account is configured with a weak password
6.  What is a unique characteristic about Lynis?
    1.  It's a proprietary, closed-source vulnerability scanner.
    2.  It's a shell script that can be used to scan any Linux, Unix, or Unix-like operating system for vulnerabilities.
    3.  It's an external vulnerability scanner.
    4.  It can only be installed on a specialty security distro, such as Kali Linux.
7.  Which of these problems would you most likely find with Snort?
    1.  A root user account with a weak password
    2.  Servers without active firewalls
    3.  Cryptocoin mining malware active on the network
    4.  Root user accounts exposed to the Internet via SSH

## Further reading

*   Lynis home page: [https://cisofy.com/lynis/](https://cisofy.com/lynis/)
*   How Lynis and auditd are different: [https://linux-audit.com/how-are-auditd-and-lynis-different/](https://linux-audit.com/how-are-auditd-and-lynis-different/)
*   Greenbone home page: [https://securityonionsolutions.com/](https://securityonionsolutions.com/)
*   Snort home page: [https://www.snort.org/](https://www.snort.org/)
*   Nikto home page: [https://cirt.net/nikto2](https://cirt.net/nikto2)
*   Security Onion home page: [https://securityonionsolutions.com/](https://securityonionsolutions.com/)
*   Tutorial for installing Greenbone Security Manager: [https://youtu.be/OUiRTv4Q80c](https://youtu.be/OUiRTv4Q80c)

## Answers

1.  b
2.  b
3.  c
4.  a
5.  c
6.  b
7.  c