# 16 Security Tips and Tricks for the Busy Bee

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file110.png)

In this final chapter, I'd like to do a roundup of some quick tips and tricks that don't necessarily fit in with the previous chapters. Think of these tips as time savers for the busy administrator. First, you will learn about some quick ways to see which system services are running, in order to ensure that nothing that isn't needed is running. Then, we'll look at how to password protect the GRUB 2 bootloader, how to securely configure BIOS/UEFI to help prevent attacks on a physically accessible machine, and how to use a checklist to perform a secure initial system setup.

In this chapter, we will cover the following topics:

*   Auditing system services
*   Password protecting the GRUB2 configuration
*   Securely configuring and then password protecting UEFI/BIOS
*   Using a security checklist when setting up your system

If you’re ready, let’s get going.

## Technical requirements

The code files for this chapter are available here: [https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-3E](https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-3E).

## Auditing system services

A basic tenet of server administration, regardless of which operating system we're talking about, is to never have anything that you don't absolutely need installed on a server. You especially don't want any unnecessary network services running because that would give the bad guys extra ways to get into your system. And, there's always a chance that some evil hacker might have planted something that acts as a network service, and you'd definitely want to know about that. In this section, we'll look at a few different ways to audit your system to ensure that no unnecessary network services are running on it.

### Auditing system services with systemctl

On Linux systems that come with `systemd`, the `systemctl` command is pretty much a universal command that does many things for you. In addition to controlling your system's services, it can also show you the status of those services, like so:

```
donnie@linux-0ro8:~> sudo systemctl -t service --state=active
```

Here's the breakdown of the preceding command:

*   `-t service`: We want to view information about the services – or, what used to be called daemons – on the system.
*   `--state=active`: This specifies that we want to view information about all the system services that are actually running.

A partial output of this command looks something like this:

```
UNIT                                                  LOAD   ACTIVE SUB     DESCRIPTION
accounts-daemon.service                               loaded active running Accounts Service
after-local.service                                   loaded active exited  /etc/init.d/after.local Compatibility
alsa-restore.service                                  loaded active exited  Save/Restore Sound Card State
apparmor.service                                      loaded active exited  Load AppArmor profiles
auditd.service                                        loaded active running Security Auditing Service
avahi-daemon.service                                  loaded active running Avahi mDNS/DNS-SD Stack
cron.service                                          loaded active running Command Scheduler
. . .
. . .
```

Generally, you won't want to see quite this much information, although you might at times. This command shows the status of every service that's running on your system. What really interests us now is the network services that can allow someone to connect to your system. So, let's look at how to narrow things down a bit.

### Auditing network services with netstat

Here are two reasons why you would want to keep track of what network services are running on your system:

*   To ensure that no legitimate network services that you don't need are running
*   To ensure that you don't have any malware that's listening for network connections from its master

The `netstat` command is both handy and easy to use. First, let's say that you want to see a list of network services that are listening, waiting for someone to connect to them. (Due to formatting restrictions, I can only show part of the output here. We'll look at some lines that I can't show here in just a moment. Also, you can download the text file with the full output from the Packt Publishing website.):

```
donnie@linux-0ro8:~> netstat -lp -A inet
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name
tcp 0 0 *:ideafarm-door *:* LISTEN -
tcp 0 0 localhost:40432 *:* LISTEN 3296/SpiderOakONE
tcp 0 0 *:ssh *:* LISTEN -
tcp 0 0 localhost:ipp *:* LISTEN -
tcp 0 0 localhost:smtp *:* LISTEN -
tcp 0 0 *:db-lsp *:* LISTEN 3246/dropbox
tcp 0 0 *:37468 *:* LISTEN 3296/SpiderOakONE
tcp 0 0 localhost:17600 *:* LISTEN 3246/dropbox
. . .
. . .
```

Here's the breakdown:

*   `-lp`: The `l` means that we want to see which network ports are listening. In other words, we want to see which network ports are waiting for someone to connect to them. The `p` means that we want to see the name and process ID number of the program or service that is listening on each port.
*   `-A inet`: This means that we only want to see information about the network protocols that are members of the `inet` family. In other words, we want to see information about the `raw`, `tcp`, and `udp` network sockets, but we don't want to see anything about the Unix sockets that only deal with interprocess communications within the operating system.

Since this output is from the openSUSE workstation that I used to write the original version of this chapter, you won't see any of the usual server-type services here. However, you will see a few things that you likely won't want to see on your servers. For example, let's look at the very first item:

```
Proto Recv-Q Send-Q Local Address      Foreign Address         State       PID/Program name
tcp        0      0 *:ideafarm-door    *:*                     LISTEN      -
```

The `Local Address` column specifies the local address and port of this listening socket. The asterisk means that this socket is on the local network, while `ideafarm-door` is the name of the network port that is listening. (By default, `netstat` will show you the names of ports whenever possible by pulling the port information out of the `/etc/services` file.)

Now, because I didn't know what the `ideafarm-door` service is, I used my favorite search engine to find out. By plugging the term `ideafarm-door` into DuckDuckGo, I found the answer:

![Figure 16.1: WhatPortIs](img/file111.png)

Figure 16.1: WhatPortIs

The top search result took me to a site named **WhatPortIs**. According to this, `ideafarm-door` is, in reality, port `902`, which belongs to the **VMware Server Console**. Okay, that makes sense because I do have VMware Player installed on this machine. So, that's all good.

> You can check out the `WhatPortIs` site here: [http://whatportis.com/](http://whatportis.com/).

Here’s the next item on the list:

```
tcp        0      0 localhost:40432    *:*       LISTEN      3296/SpiderOakONE
```

This item shows the local address as `localhost` and that the listening port is port `40432`. This time, the `PID/Program Name` column actually tells us what this is. `SpiderOak ONE` is a cloud-based backup service that you might or might not want to see running on your server.

Now, let's look at a few more items:

```
tcp 0      0 *:db-lsp                   *:*      LISTEN      3246/dropbox
tcp 0      0 *:37468                    *:*      LISTEN      3296/SpiderOakONE
tcp 0      0 localhost:17600            *:*      LISTEN      3246/dropbox
tcp 0      0 localhost:17603            *:*      LISTEN      3246/dropbox
```

Here, we can see that `dropbox` and `SpiderOakONE` are both listed with the asterisk for the local address. So, they're both using the local network address. The name of the port for `dropbox` is `db-lsp`, which stands for **Dropbox LAN Sync Protocol**. The `SpiderOakONE` port doesn't have an official name, so it's just listed as port `37468`. The bottom two lines show that `dropbox` also uses the local machine's address, on ports `17600` and `17603`.

So far, we've looked at nothing but TCP network sockets. Let's see how they differ from UDP sockets:

```
udp        0      0 192.168.204.1:ntp       *:*                                 -
udp        0      0 172.16.249.1:ntp        *:*                                 -
udp        0      0 linux-0ro8:ntp          *:*                                 -
```

The first thing to note is that there's nothing under the `State` column. That's because, with UDP, there are no states. They are actually listening for data packets to come in, and they're ready to send data packets out. But since that's about all that UDP sockets can do, there was really no sense in defining different states for them.

In the first two lines, we see some strange local addresses. That's because I have both VMware Player and VirtualBox installed on this workstation. The local addresses of these two sockets are for the VMware and VirtualBox virtual network adapters. The last line shows the hostname of my OpenSUSE workstation as the local address. In all three cases, the port is the **Network Time Protocol** port, for time synchronization.

Now, let's look at one last set of UDP items:

```
udp        0      0 *:58102         *:*                                 5598/chromium --pas
udp        0      0 *:db-lsp-disc   *:*                                 3246/dropbox
udp        0      0 *:43782         *:*                                 5598/chromium --pas
udp        0      0 *:36764         *:*                                 
udp        0      0 *:21327         *:*                                 3296/SpiderOakONE
udp        0      0 *:mdns          *:*                                 5598/chromium --pas
```

Here, we see that my Chromium web browser is ready to accept network packets on a few different ports. We also see that Dropbox uses UDP to accept discovery requests from other local machines that have Dropbox installed. I assume that port `21327` performs the same function for SpiderOak ONE.

Of course, since this machine is one of my workhorse workstations, Dropbox and SpiderOak ONE are almost indispensable to me. I installed them myself, so I've always known that they were there. However, if you see anything like this on a server, you'll want to investigate to see if the server admins know that these programs are installed, and then find out why they're installed. It could be that they're performing a legitimate function, and it could be that they're not.

> A difference between Dropbox and SpiderOak ONE is that with Dropbox, your files don't get encrypted until they've been uploaded to the Dropbox servers. So, the Dropbox folk have the encryption keys to your files. On the other hand, SpiderOak ONE encrypts your files on your local machine, and the encryption keys never leave your possession. So, if you really do need a cloud-based backup service and you're dealing with sensitive files, something such as SpiderOak ONE would definitely be better than Dropbox. (And no, the SpiderOak ONE folk aren't paying me to say that.)

If you want to see port numbers and IP addresses instead of network names, add the `n` option. As before, here's the partial output:

```
donnie@linux-0ro8:~> netstat -lpn -A inet
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address      Foreign Address     State       PID/Program name
tcp        0      0 0.0.0.0:902        0.0.0.0:*           LISTEN      -
tcp        0      0 127.0.0.1:40432    0.0.0.0:*           LISTEN      3296/SpiderOakONE
tcp        0      0 0.0.0.0:22         0.0.0.0:*           LISTEN      -
tcp        0      0 127.0.0.1:631      0.0.0.0:*           LISTEN      -
tcp        0      0 127.0.0.1:25       0.0.0.0:*           LISTEN      -
tcp        0      0 0.0.0.0:17500      0.0.0.0:*           LISTEN      3246/dropbox
tcp        0      0 0.0.0.0:37468      0.0.0.0:*           LISTEN      3296/SpiderOakONE
tcp        0      0 127.0.0.1:17600    0.0.0.0:*           LISTEN      3246/dropbox
. . .
. . .
```

All you have to do to view the established TCP connections is to leave out the `l` option. On my workstation, this makes for a very long list, so I'll only show a few items:

```
donnie@linux-0ro8:~> netstat -p -A inet
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address      Foreign Address         State       PID/Program name
tcp        1      0 linux-0ro8:41670   ec2-54-88-208-223:https CLOSE_WAIT  3246/dropbox
tcp        0      0 linux-0ro8:59810   74-126-144-106.wa:https ESTABLISHED 3296/SpiderOakONE
tcp        0      0 linux-0ro8:58712   74-126-144-105.wa:https ESTABLISHED 3296/SpiderOakONE
tcp        0      0 linux-0ro8:caerpc  atl14s78-in-f2.1e:https ESTABLISHED 10098/firefox
. . .
. . .
```

The `Foreign Address` column shows the address and port number of the machine at the remote end of the connection. The first item shows that the connection with a Dropbox server is in a `CLOSE_WAIT` state. This means that the Dropbox server has closed the connection, and we're now waiting on the local machine to close the socket.

Because the names of those foreign addresses don't make much sense, let's add the `n` option to see the IP addresses instead:

```
donnie@linux-0ro8:~> netstat -np -A inet
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address         Foreign Address      State        PID/Program name
tcp        0      1 192.168.0.222:59594   37.187.24.170:443    SYN_SENT     10098/firefox
tcp        0      0 192.168.0.222:59810   74.126.144.106:443   ESTABLISHED  3296/SpiderOakONE
tcp        0      0 192.168.0.222:58712   74.126.144.105:443   ESTABLISHED  3296/SpiderOakONE
tcp        0      0 192.168.0.222:38606   34.240.121.144:443   ESTABLISHED  10098/firefox
. . .
. . .
```

This time, we see something new. The first item shows a `SYN_SENT` state for the Firefox connection. This means that the local machine is trying to establish a connection to the foreign IP address. Also, under `Local Address`, we can see the static IP address for my OpenSUSE workstation.

If I had space to display the entire `netstat` output here, you'd see nothing but `tcp` under the `Proto` column. That's because the UDP protocol doesn't establish connections in the same way that the TCP protocol does.

> Something to keep in mind is that rootkits can replace legitimate Linux utilities with their own trojaned versions. For example, a rootkit could have its own trojaned version of `netstat` that would show all network processes except for those that are associated with the rootkit. That's why you want to do everything you can to prevent unauthorized users from gaining root privileges, to prevent them from being able to install rootkits.

If you need more information about `netstat`, see the `netstat` man page.

#### Hands-on lab – viewing network services with netstat

In this lab, you'll practice what you've just learned about `netstat`. Do this on a virtual machine that has a desktop interface so that you can use Firefox to visit websites. Follow these steps:

1.  View the list of network services that are listening for a connection:

```
netstat -lp -A inet
netstat -lpn -A inet
```

1.  View the list of established connections:

```
netstat -p -A inet
netstat -pn -A inet
```

1.  Open Firefox and navigate to any website. Then, repeat *Step 2*.
2.  Repeat *Step 2* again, but preface each command with `sudo`. Note how the output is different from that of *Step 2*.
3.  From your host machine, log into the virtual machine via SSH. Then, repeat *Step 2*.

You've reached the end of the lab – congratulations!

You've just seen how to audit network services with `netstat`. Now, let's learn how to do this with Nmap.

### Auditing network services with Nmap

The `netstat` tool is very good, and it can give you lots of good information about what's going on with your network services. The slight downside is that you have to log in to every individual host on your network in order to use it.

If you'd like to remotely audit your network to see what services are running on each computer without having to log in to each and every one, then you need a tool such as Nmap. It's available for all the major operating systems, so even if you're stuck having to use Windows on your workstation, you're in luck. An up-to-date version is already installed on Kali Linux, if that's what you're using. It's also in the repositories of every major Linux distro, so installing it is quite simple. If you’re running Windows or macOS, you can download a version for either of them directly from the Nmap website.

> You can download Nmap for all of the major operating systems from [https://nmap.org/download.html](https://nmap.org/download.html).
> 
> > In all cases, you'll also find instructions for installation.

You'll use Nmap the same way on all operating systems, with only one exception. On Linux and macOS machines, you'll preface certain Nmap commands with `sudo`, while on Windows machines, you won't. (Although, on Windows 10/11, you might have to open the `command.exe` terminal as an administrator.) Since I just happen to be working on my trusty OpenSUSE workstation, I'll show you how it works on Linux. Let's start by doing a SYN packet scan:

```
donnie@linux-0ro8:~> sudo nmap -sS 192.168.0.37
Starting Nmap 6.47 ( http://nmap.org ) at 2017-12-24 19:32 EST
Nmap scan report for 192.168.0.37
Host is up (0.00016s latency).
Not shown: 996 closed ports
PORT STATE SERVICE
22/tcp open ssh
515/tcp open printer
631/tcp open ipp
5900/tcp open vnc
MAC Address: 00:0A:95:8B:E0:C0 (Apple)
Nmap done: 1 IP address (1 host up) scanned in 57.41 seconds
donnie@linux-0ro8:~>
```

Here's the breakdown:

*   `-sS`: The lowercase `s` denotes the type of scan that we want to perform. The uppercase `S` denotes that we're doing a SYN packet scan. (More on that in a moment.)
*   `192.168.0.37`: In this case, I'm only scanning a single machine. However, I could also scan either a group of machines or an entire network.
*   `Not shown: 996 closed ports`: The fact that it's showing all of these closed ports instead of `filtered` ports tells me that there's no firewall on this machine. (Again, more on that in a moment.)

Next, we see a list of ports that are open. (More on that in a moment.)

The MAC address of this machine indicates that it's an Apple product of some sort. In a moment, I'll show you how to get more details about what kind of Apple product that it might be.

Now, let's look at this more in detail.

#### Port states

An Nmap scan will show the target machine's ports in one of three **port states**:

*   `filtered`: This means that the port is blocked by a firewall.
*   `open`: This means that the port is not blocked by a firewall and that the service that's associated with that port is running.
*   `closed`: This means that the port is not blocked by a firewall, and that the service that's associated with that port is not running.

So, in our scan of the Apple machine, we see that the Secure Shell service is ready to accept connections on port `22`, that the print service is ready to accept connections on ports `515` and `631`, and that the **Virtual Network Computing** (**VNC**) service is ready to accept connections on port `5900`. All of these ports would be of interest to a security-minded administrator. If Secure Shell is running, it would be interesting to know if it's configured securely. The fact that the print service is running means that this is set up to use the **Internet Printing Protocol** (**IPP**). It would be interesting to know why we're using IPP instead of just regular network printing, and it would also be interesting to know if there are any security concerns with this version of IPP. And of course, we already know that VNC isn't a secure protocol, so we would want to know why it's even running at all. We also saw that no ports are listed as `filtered`, so we would also want to know why there's no firewall on this machine.

One little secret that I'll finally reveal is that this machine is the same one that I used for the Greenbone Security Assistant scan demos. So, we already have some of the needed information. The Greenbone scan told us that Secure Shell on this machine uses weak encryption algorithms and that there's a security vulnerability with the print service. In just a bit, I'll show you how to get some of that information with Nmap.

#### Scan types

There are lots of different scanning options, each with its own purpose. The SYN packet scan that we're using here is considered a stealthy type of scan because it generates less network traffic and fewer system log entries than certain other types of scans. With this type of scan, Nmap sends a SYN packet to a port on the target machine, as if it were trying to create a TCP connection to that machine. If the target machine responds with a SYN/ACK packet, it means that the port is in an `open` state and is ready to create the TCP connection. If the target machine responds with an RST packet, it means that the port is in a `closed` state. If there's no response at all, it means that the port is `filtered`, blocked by a firewall. As a normal Linux administrator, this is one of the types of scans that you would do most of the time.

The `-sS` scan shows you the state of TCP ports, but it doesn't show you the state of UDP ports. To see the UDP ports, use the `-sU` option:

```
donnie@linux-0ro8:~> sudo nmap -sU 192.168.0.37
Starting Nmap 6.47 ( http://nmap.org ) at 2017-12-28 12:41 EST
Nmap scan report for 192.168.0.37
Host is up (0.00018s latency).
Not shown: 996 closed ports
PORT     STATE         SERVICE
123/udp  open          ntp
631/udp  open|filtered ipp
3283/udp open|filtered netassistant
5353/udp open          zeroconf
MAC Address: 00:0A:95:8B:E0:C0 (Apple)
Nmap done: 1 IP address (1 host up) scanned in 119.91 seconds
donnie@linux-0ro8:~>
```

Here, you see something a bit different: two ports are listed as `open|filtered`. That's because, due to the way that UDP ports respond to Nmap scans, Nmap can't always tell whether a UDP port is `open` or `filtered`. In this case, we know that these two ports are probably open because we've already seen that their corresponding TCP ports are open.

ACK packet scans can also be useful, but not to see the state of the target machine's network services. Rather, it's a good option for when you need to see if there might be a firewall blocking the way between you and the target machine. An ACK scan command looks like this:

```
sudo nmap -sA 192.168.0.37
```

You're not limited to scanning just a single machine at a time. You can scan either a group of machines or an entire subnet at once:

```
sudo nmap -sS 192.168.0.1-128
sudo nmap -sS 192.168.0.0/24
```

The first command scans only the first 128 hosts on this network segment. The second command scans all 254 hosts on a subnet that's using a 24-bit netmask.

A discovery scan is useful for when you need to just see what devices are on the network:

```
sudo nmap -sn 192.168.0.0/24
```

With the `-sn` option, Nmap will detect whether you're scanning the local subnet or a remote subnet. If the subnet is local, Nmap will send out an **Address Resolution Protocol** (**ARP**) broadcast that requests the IPv4 addresses of every device on the subnet. That's a reliable way of discovering devices because ARP isn't something that will ever be blocked by a device's firewall. (I mean, without ARP, the network would cease to function.) However, ARP broadcasts can't go across a router, which means that you can't use ARP to discover hosts on a remote subnet. So, if Nmap detects that you're doing a discovery scan on a remote subnet, it will send out ping packets instead of ARP broadcasts. Using ping packets for discovery isn't as reliable as using ARP because some network devices can be configured to ignore ping packets. Anyway, here's an example from my own home network:

```
donnie@linux-0ro8:~> sudo nmap -sn 192.168.0.0/24
Starting Nmap 6.47 ( http://nmap.org ) at 2017-12-25 14:48 EST
Nmap scan report for 192.168.0.1
Host is up (0.00043s latency).
MAC Address: 00:18:01:02:3A:57 (Actiontec Electronics)
Nmap scan report for 192.168.0.3
Host is up (0.0044s latency).
MAC Address: 44:E4:D9:34:34:80 (Cisco Systems)
Nmap scan report for 192.168.0.5
Host is up (0.00026s latency).
MAC Address: 1C:1B:0D:0A:2A:76 (Unknown)
. . .
. . .
```

We see three hosts in this snippet, and there are three lines of output for each host. The first line shows the IP address, the second shows whether the host is up, and the third shows the MAC address of the host's network adapter. The first three pairs of characters in each MAC address denote the manufacturer of that network adapter. (For the record, that unknown network adapter is on a recent model Gigabyte motherboard. I have no idea why it's not in the Nmap database.)

The final scan that we'll look at does four things for us:

*   It identifies `open`, `closed`, and `filtered` TCP ports.
*   It identifies the versions of the running services.
*   It runs a set of vulnerability scanning scripts that come with Nmap.
*   It attempts to identify the operating system of the target host.

The scan command that does all of these things looks like this:

```
sudo nmap -A 192.168.0.37
```

I guess that you could think of the `-A` option as the *all* option since it really does do it all. (Well, almost all, since it doesn't scan UDP ports.) First, here's the command that I ran to do the scan:

```
donnie@linux-0ro8:~> sudo nmap -A 192.168.0.37
```

Here are the results, broken down into sections for formatting purposes:

```
Starting Nmap 6.47 ( http://nmap.org ) at 2017-12-24 19:33 EST
Nmap scan report for 192.168.0.37
Host is up (0.00016s latency).
Not shown: 996 closed ports
```

Right away, we see that there's no active firewall on this machine because no ports are in the `filtered` state. By default, Nmap scans only the most 1,000 most popular ports. Since 996 ports are in the `closed` state, we obviously only have four active network services that would listen on any of these 1,000 ports:

```
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 5.1 (protocol 1.99)
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
|_sshv1: Server supports SSHv1
515/tcp open printer?
```

Port `22` is open for Secure Shell access, which we would normally expect. However, look at the SSH version. Version 5.1 is a really old version of OpenSSH. (At the time of writing, the current version is version 9.1.) What's worse is that this OpenSSH server supports version 1 of the Secure Shell protocol. Version 1 is seriously flawed and is easily exploitable, so you never want to see this on your network.

Next, we have amplifying information on the print service vulnerability that we found with the Greenbone Security Assistant scan:

```
631/tcp open ipp CUPS 1.1
| http-methods: Potentially risky methods: PUT
|_See http://nmap.org/nsedoc/scripts/http-methods.html
| http-robots.txt: 1 disallowed entry
|_/
|_http-title: Common UNIX Printing System
```

In the `631/tcp` line, we see that the associated service is `ipp`. This protocol is based on the same HTTP that we use to look at web pages. The two methods that HTTP uses to send data from a client to a server are `POST` and `PUT`. What we really want is for every HTTP server to use the `POST` method because the `PUT` method makes it very easy for someone to compromise a server by manipulating a URL. So, if you scan a server and find that it allows using the `PUT` method for any kind of HTTP communications, you have a potential problem. In this case, the solution would be to update the operating system and hope that the updates fix the problem. If this were a web server, you'd want to have a chat with the web server administrators to let them know what you found.

Next, we see that the VNC service is running on this machine:

```
5900/tcp open vnc Apple remote desktop vnc
| vnc-info:
| Protocol version: 3.889
| Security types:
|_ Mac OS X security type (30)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at http://www.insecure.org/cgi-bin/servicefp-submit.cgi :
SF-Port515-TCP:V=6.47%I=7%D=12/24%Time=5A40479E%P=x86_64-suse-linux-gnu%r(
SF:GetRequest,1,"\x01");
MAC Address: 00:0A:95:8B:E0:C0 (Apple)
Device type: general purpose
```

VNC can be handy at times. It's like Microsoft's Remote Desktop service for Windows, except that it's free, open source software. But it's also a security problem because it's an unencrypted protocol. So, all your information goes across the network in plain text. If you must use VNC, run it through an SSH tunnel.

Next, let's see what Nmap found out about the operating system of our target machine:

```
Running: Apple Mac OS X 10.4.X
OS CPE: cpe:/o:apple:mac_os_x:10.4.10
OS details: Apple Mac OS X 10.4.10 - 10.4.11 (Tiger) (Darwin 8.10.0 - 8.11.1)
Network Distance: 1 hop
Service Info: OS: Mac OS X; CPE: cpe:/o:apple:mac_os_x
```

Wait, what? Mac OS X 10.4? Isn't that really, really ancient? Well, yeah, it is. The secret that I've been guarding for the past couple of chapters is that the target machine for my Greenbone Security Assistant and Nmap scan demos has been my ancient, collectible Apple eMac from the year 2003\. I figured that scanning it would give us some interesting results to look at, and it would appear that I was right. (And yes, that is *eMac*, not *iMac*.)

The final thing we see is the `TRACEROUTE` information. It's not very interesting, though, because the target machine was sitting right next to me, with only one Cisco switch between us:

```
TRACEROUTE
HOP RTT ADDRESS
1 0.16 ms 192.168.0.37
OS and Service detection performed. Please report any incorrect results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 213.92 seconds
donnie@linux-0ro8:~>
```

> Let's say that the target machine has had its SSH service changed to some alternate port, instead of having it run on the default port, `22`. If you scan the machine with a normal `-sS` or `-sT` scan, Nmap won't correctly identify the SSH service on that alternate port. However, a `-A` scan will correctly identify the SSH service, regardless of which port it's using.

Okay, let’s do a lab.

#### Hands-on lab – scanning with Nmap

In this lab, you'll see the results of scanning a machine with various services either enabled or disabled. You'll start with a virtual machine that has its firewall disabled. Let's get started:

1.  Briefly peruse the Nmap help screen by using the following command:

```
nmap
```

1.  From either your host machine or from another virtual machine, perform these scans against a virtual machine that has its firewall disabled (substitute your own IP address for the one I'm using here):

```
sudo nmap -sS 192.168.0.252
sudo nmap -sT 192.168.0.252
sudo nmap -SU 192.168.0.252
sudo nmap -A 192.168.0.252
sudo nmap -sA 192.168.0.252
```

1.  Stop the SSH service on the target machine on Ubuntu:

```
sudo systemctl stop ssh
```

On either CentOS or AlmaLinux, use this command:

```
sudo systemctl stop sshd
```

1.  Repeat *step 2*.

You've reached the end of this lab – congratulations!

Now that you've seen how to scan a system, let's look at the GRUB2 bootloader.

## Password protecting the GRUB 2 bootloader

People sometimes forget passwords, even if they're administrators. And sometimes, people buy used computers but forget to ask the seller what the password is. (Yes, I've done that.) That's okay, though, because all of the major operating systems have ways to let you either reset or recover a lost administrator password. That's handy, except that it does kind of make the whole idea of having login passwords a rather moot point when someone has physical access to the machine. Let's say that your laptop has just been stolen. If you haven't encrypted the hard drive, it would only take a few minutes for the thief to reset the password and steal your data. If you have encrypted the drive, the level of protection would depend on which operating system you're running. With standard Windows folder encryption, the thief would be able to access the encrypted folders just by resetting the password. With LUKS whole-disk encryption on a Linux machine, the thief wouldn't be able to get past the point of having to enter the encryption passphrase.

With Linux, we have a way to safeguard against unauthorized password resets, even if we're not using whole-disk encryption. All we have to do is to password protect the **Grand Unified Bootloader** (**GRUB**), which would prevent a thief from booting into emergency mode to do the password reset.

> Whether or not you need the advice in this section depends upon your organization's physical security setup. That's because booting a Linux machine into emergency mode requires physical access to the machine. It's not something that you can do remotely. In an organization with proper physical security, servers – especially ones that hold sensitive data – are locked away in a room that's locked within another room. Only a very few trusted personnel are allowed to enter, and they have to present their credentials at both access points. So, setting a password on the bootloader of those servers would be rather pointless, unless you're dealing with a regulatory agency that dictates otherwise.
> 
> > On the other hand, password protecting the bootloaders of workstations and laptops that are out in the open could be quite useful. However, that alone won't protect your data. Someone could still boot the machine from a live disk or a USB memory stick, mount the machine's hard drive, and obtain the sensitive data. That's why you also want to encrypt your sensitive data, as I showed you in *Chapter 6*, *Encryption Technologies*.

To reset a password, all you have to do is interrupt the boot process when the boot menu comes up and either change a couple of kernel parameters, or select the **Recovery** mode option if it’s available. Either way, the machine will boot into emergency mode without asking for a password. However, resetting passwords isn't the only thing you can do from emergency mode. Once you’ve booted into emergency mode, you have full root user control over the whole system.

Now, just so you'll know what I'm talking about when I say that you can edit kernel parameters from the GRUB 2 boot menu, let me show you how to perform a password reset on a Red Hat-type system.

### Hands-on lab – resetting the password for Red Hat/CentOS/AlmaLinux

With only one very minor exception, this procedure works exactly the same on CentOS 7, AlmaLinux 8, and AlmaLinux 9\. Let's get started:

1.  Boot the virtual machine. When the boot menu comes up, interrupt the boot process by hitting the down arrow key once. Then, hit the up arrow key once to select the default boot option:

    ![Figure 16.2: Selecting the boot option](img/file112.png)

    Figure 16.2: Selecting the boot option

2.  Hit the e key to edit the kernel parameters. When the GRUB 2 configuration comes up, cursor down until you see this line:

    ![Figure 16.3: Edit the kernel options](img/file113.png)

    Figure 16.3: Edit the kernel options

    > Note that on CentOS 7, the line begins with `linux16`, as shown here. On AlmaLinux 8/9, the line begins with `linux`.

3.  Delete the words `rhgb quiet` from this line and then add `rd.break enforcing=0` to the end of the line. Here's what these two new options do for you:
4.  `rd.break`: This will cause the machine to boot into emergency mode, which gives you root user privileges without you having to enter a root user password. Even if the root user password hasn't been set, this still works.
5.  `enforcing=0`: When you do a password reset on an SELinux-enabled system, the security context for the `/etc/shadow` file will change to the wrong type. If the system is in enforcing mode when you do this, SELinux will prevent you from logging in until the `shadow` file is relabeled. However, relabeling during the boot process can take a very long time, especially with a large drive. By setting SELinux to permissive mode, you can wait until after you've rebooted to restore the proper security context on just the `shadow` file.
6.  When you've finished editing the kernel parameters, hit Ctrl + X to continue the boot process. This will take you to emergency mode with the `switch_root` command prompt:

    ![Figure 16.4: In emergency mode](img/file114.png)

    Figure 16.4: In emergency mode

7.  In emergency mode, the filesystem is mounted as read-only. You'll need to remount it as read-write and enter `chroot` mode before you can reset the password, using these two commands:

```
mount -o remount,rw /sysroot
chroot /sysroot
```

After you enter these two commands, the command prompt will change to that of a normal bash shell:

![Figure 16.5: Entering the chroot](img/file115.png)

Figure 16.5: Entering the chroot

Now that you've reached this stage, you're finally ready to reset the password.

1.  If you want to reset the root user password, or even if you want to create a root password where none previously existed, just enter:

```
passwd
```

Then, enter the new desired password.

1.  If the system has never had a root user password and you still don't want it to have one, you can reset the password for an account that has full sudo privileges. For example, on my system, the command would look like this:

```
passwd donnie
```

1.  Next, remount the filesystem as read-only. Then, enter `exit` twice to resume rebooting:

```
mount -o remount,ro /
exit
exit
```

1.  The first thing you need to do after rebooting is to restore the proper SELinux security context on the `/etc/shadow` file. Then, put SELinux back into enforcing mode:

```
sudo restorecon /etc/shadow
sudo setenforce 1
```

Here's a before and after screenshot of the context settings for my `shadow` file:

![Figure 16.6: SELinux context settings for the shadow file](img/file116.png)

Figure 16.6: SELinux context settings for the shadow file

Here, you see that resetting the password changed the type of the file to `unlabeled_t`. Running the `restorecon` command changed the type back to `shadow_t`.

You've reached the end of this lab – congratulations!

Now, we'll look at the same procedure for Ubuntu.

### Hands-on lab – resetting the password for Ubuntu

The procedure for resetting a password on an Ubuntu system is quite a bit different and quite a bit simpler. However, there is one slight difference between doing this on Ubuntu 16.04 and Ubuntu 18.04 or newer. That is, to see the boot menu on Ubuntu 16.04, you don't have to do anything. On Ubuntu 18.04, you have to press either the Shift key (on BIOS-based systems) or the Esc key (on UEFI-based systems) in order to see the boot menu. On the current Ubuntu 22.04, you’ll press the Esc key for either BIOS-based or UEFI-based systems. Other than that, the procedure is identical for everything from Ubuntu 16.04 through the current Ubuntu 22.04\. So now, let’s get started:

1.  Boot the virtual machine. Press the Esc key to bring up the boot menu.
2.  Press the down arrow key to highlight the Advanced Options for Ubuntu menu item, and press the Enter key:

    ![Figure 16.7: Ubuntu Advanced Options submenu](img/file117.png)

    Figure 16.7: Ubuntu Advanced Options submenu

3.  From the **Advanced Options for Ubuntu** submenu, select the **recovery mode** option, and press Enter:

    ![Figure 16.8: Select the recovery mode option](img/file118.png)

    Figure 16.8: Select the recovery mode option

4.  When the **Recovery Menu** comes up, select the **root** option, and press the Enter key:

    ![Figure 16.9: Select the root option](img/file119.png)

    Figure 16.9: Select the root option

5.  Press the Enter key again. This will take you to a root shell:

    ![Figure 16.10: In recovery mode](img/file120.png)

    Figure 16.10: In recovery mode

6.  Since Ubuntu doesn't normally have a password assigned to the root user, you would most likely just reset the password of whoever had full sudo privileges, like so:

```
passwd donnie
```

1.  When you've finished, reboot as you normally would:

```
shutdown -r now
```

The machine will now boot up for normal operation.

You've reached the end of this lab – congratulations!

Of course, we don't want everybody and his brother to be able to edit kernel parameters or enter **Recovery** mode when booting a machine. So, let's fix that.

### Preventing kernel parameter edits on Red Hat/CentOS/AlmaLinux

Ever since the introduction of Red Hat/CentOS 7.2, setting a GRUB 2 password to prevent kernel parameter edits is easy. Fortunately, this trick still works on the newest iterations of Red Hat and AlmaLinux. All you have to do is to run one command and choose a password:

```
[donnie@localhost ~]$ sudo grub2-setpassword
[sudo] password for donnie:
Enter password:
Confirm password:
[donnie@localhost ~]$
```

That's all there is to it. The password hash will be stored in the `/boot/grub2/user.cfg` file.

Now, when you reboot the machine and try to do a kernel parameter edit, you'll be prompted to enter a username and password:

![Figure 16.11: Password protection for RHEL 7.2 and newer](img/file121.png)

Figure 16.11: Password protection for RHEL 7.2 and newer

Note that you'll enter `root` as the username, even if the `root` user's password hasn't been set on the system. The `root` user, in this case, is just the superuser for GRUB 2.

When you boot your Red Hat, CentOS, or AlmaLinux machine, you’ll see a **0-rescue** option come up at the bottom of the boot menu. (You can see it above in Figure 16.2.) If you select it, you’ll find that it does nothing but take you to a normal login prompt that will require you to enter your username and password. (Red Hat-type distros really do have a Rescue mode, but you have to boot the machine from the installation media to get to it.)

### Preventing kernel parameter edits or Recovery mode access on Ubuntu

Ubuntu doesn't have that cool utility that Red Hat, CentOS, and AlmaLinux have, so you'll have to set a GRUB 2 password by hand-editing a configuration file.

In the `/etc/grub.d/` directory, you'll see the files that make up the GRUB 2 configuration:

```
donnie@ubuntu3:/etc/grub.d$ ls -l
total 76
-rwxr-xr-x 1 root root  9791 Oct 12 16:48 00_header
-rwxr-xr-x 1 root root  6258 Mar 15  2016 05_debian_theme
-rwxr-xr-x 1 root root 12512 Oct 12 16:48 10_linux
-rwxr-xr-x 1 root root 11082 Oct 12 16:48 20_linux_xen
-rwxr-xr-x 1 root root 11692 Oct 12 16:48 30_os-prober
-rwxr-xr-x 1 root root  1418 Oct 12 16:48 30_uefi-firmware
-rwxr-xr-x 1 root root   214 Oct 12 16:48 40_custom
-rwxr-xr-x 1 root root   216 Oct 12 16:48 41_custom
-rw-r--r-- 1 root root   483 Oct 12 16:48 README
donnie@ubuntu3:/etc/grub.d$
```

The file you want to edit is the `40_custom` file. However, before you edit the file, you'll need to create the password hash. Do that with the `grub-mkpasswd-pbkdf2` utility:

```
donnie@ubuntu3:/etc/grub.d$ grub-mkpasswd-pbkdf2
Enter password:
Reenter password:
PBKDF2 hash of your password is grub.pbkdf2.sha512.10000.F1BA16B2799CBF6A6DFBA537D43222A0D5006124ECFEB29F5C81C9769C6C3A66BF53C2B3AB71BEA784D4386E86C991F7B5D33CB6C29EB6AA12C8D11E0FFA0D40.371648A84CC4131C3CFFB53604ECCBA46DA75AF196E970C98483385B0BE026590C63A1BAC23691517BC4A5D3EDF89D026B599A0D3C49F2FB666F9C12B56DB35D
donnie@ubuntu3:/etc/grub.d$
```

Open the `40_custom` file in your favorite text editor and add a line that defines who the superuser(s) will be. Add another line for the password hash. In my case, the file now looks like this:

```
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries. Simply type the
# menu entries you want to add after this comment. Be careful not to change
# the 'exec tail' line above.
set superusers="donnie"
password_pbkdf2 donnie grub.pbkdf2.sha512.10000.F1BA16B2799CBF6A6DFBA537D43222A0D5006124ECFEB29F5C81C9769C6C3A66BF53C2B3AB71BEA784D4386E86C991F7B5D33CB6C29EB6AA12C8D11E0FFA0D40.371648A84CC4131C3CFFB53604ECCBA46DA75AF196E970C98483385B0BE026590C63A1BAC23691517BC4A5D3EDF89D026B599A0D3C49F2FB666F9C12B56DB35D
```

> The string of text that begins with `password_pbkdf2` is all one line that wraps around on the printed page.

After you save the file, the last step is to generate a new `grub.cfg` file:

```
donnie@ubuntu3:/etc/grub.d$ sudo update-grub
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-4.4.0-104-generic
Found initrd image: /boot/initrd.img-4.4.0-104-generic
Found linux image: /boot/vmlinuz-4.4.0-101-generic
Found initrd image: /boot/initrd.img-4.4.0-101-generic
Found linux image: /boot/vmlinuz-4.4.0-98-generic
Found initrd image: /boot/initrd.img-4.4.0-98-generic
done
donnie@ubuntu3:/etc/grub.d$
```

Now, when I reboot this machine, I have to enter my password before I can either edit kernel parameters or access the **Advanced options for Ubuntu** submenu:

![Figure 16.12: Password protection for Ubuntu](img/file122.png)

Figure 16.12: Password protection for Ubuntu

There's only one problem with this. Not only does this prevent anyone except the superuser from editing the kernel parameters, but it also prevents anyone except for the superuser from booting normally. Yes, that's right. Even for normal booting, Ubuntu will now require you to enter the username and password of the authorized superuser. Fortunately, this is an easy fix.

The fix requires inserting a single word into the `/boot/grub/grub.cfg` file. Easy enough, right? However, it's not an elegant solution because you're not really supposed to hand-edit the `grub.cfg` file. At the top of the file, we see this:

```
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#
```

This means that every time we do something that will update the `grub.cfg` file, any hand-edits that we've made to the file will be lost. This includes when we do a system update that installs a new kernel, or when we do a `sudo apt autoremove` that removes any old kernels that we no longer need. The supreme irony though, is that the official GRUB 2 documentation tells us to hand-edit the `grub.cfg` file to deal with these sorts of problems. A much better way is to modify the shell script that the `update-grub` utility uses to build the `grub.cfg` file. This will prevent you from accidentally overwriting any changes that you need to preserve.

In the `/etc/grub.d/` directory, you’ll see several scripts that are used to build `grub.cfg`. The one we want is in the `10_linux` file. Open it in your text editor, and navigate down to the vicinity of line number 197\. Look for these two lines:

```
echo "menuentry '$(echo "$title" | grub_quote)' ${CLASS} \$menuentry_id_option 'gnulinux-$version-$type-$boot_device_id' {" | sed "s/^/$submenu_indentation/"
. . .
. . .
echo "menuentry '$(echo "$os" | grub_quote)' ${CLASS} \$menuentry_id_option 'gnulinux-simple-$boot_device_id' {" | sed "s/^/$submenu_indentation/"
```

(Note that each of these is one line that wraps around on the printed page.)

In each line, add `--unrestricted` after `{CLASS}`, so that the lines now look like this:

```
echo "menuentry '$(echo "$title" | grub_quote)' ${CLASS} --unrestricted \$menuentry_id_option 'gnulinux-$version-$type-$boot_device_id' {" | sed "s/^/$submenu_indentation/"
. . .
. . .
echo "menuentry '$(echo "$os" | grub_quote)' ${CLASS} --unrestricted \$menuentry_id_option 'gnulinux-simple-$boot_device_id' {" | sed "s/^/$submenu_indentation/"
```

Finally, run the `sudo update-grub` command, and you’ll be able to boot the machine normally on the default option. But, it’s a different story if you want to enter the **Advanced options for Ubuntu** submenu. With a superuser password set, you’ll always need to enter the superuser password in order to enter the **Advanced options for Ubuntu** submenu. This is true even with the `--unrestricted` option that you added to `10_linux script.` Effectively, this prevents anyone without the password from accessing the **Recovery** option.

### Disabling the submenu for Ubuntu

On Ubuntu systems, you can easily disable the Ubuntu submenu so that you’ll see all boot options by default, which will look something like this:

![Figure 16.13: The Ubuntu boot menu without the submenu](img/file123.png)

Figure 16.13: The Ubuntu boot menu without the submenu

If desired, you can also make it so that you don’t have to press the Shift or Esc key in order to see the boot menu.

First, open the `/etc/default/grub` file in your text editor. Disable the submenu by adding by adding the `GRUB_DISABLE_SUBMENU=y` line. To make the boot menu visible by default, look for these two lines:

```
GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=0
```

Comment out the first line, and change the value for the second line to a non-zero number. The lines should now look something like this:

```
# GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=10
```

Finally, run the `sudo update-grub` command. Now, when you reboot the machine, you'll see the boot menu come up by itself, and you’ll see the whole list of boot options instead of just the default boot option and a submenu option. After a ten-second timeout, the system will automatically boot on the default option.

The major flaw with disabling the Ubuntu submenu is that if you’ve configured GRUB with the `--unrestricted` option as I’ve just shown you, users will be able to boot into **Recovery** mode without entering a password. So, it’s now just as if you never password-protected GRUB in the first place. If you do disable the Ubuntu submenu, remember to also disable the **Recovery** mode option. Open the `/etc/default/grub` file in your editor, and look for this line:

```
# GRUB_DISABLE_RECOVERY="true"
```

Remove the `#` sign from in front of the line so that it now looks like this:

```
GRUB_DISABLE_RECOVERY="true"
```

Update the GRUB configuration as you’ve done before:

```
sudo update-grub
```

Finally, reboot the machine and verify that the **Recovery** mode option is gone. If you disable the **Recovery** boot menu option and still need to boot into **Recovery** mode, you can still do that by editing the kernel parameters at the beginning of the boot process. The procedure is somewhat different from what you’ve just seen with AlmaLinux, since you don’t have to worry about SELinux on Ubuntu. Rather than duplicate the procedure here, I’ll leave a link to a tutorial for it in the *Further reading* section. (The linked article is for Ubuntu 18.04, but the procedure still works for the current Ubuntu 22.04.)

So, you’re now asking, *Why would I ever need to disable the Ubuntu submenu?* Well, you’ll never actually *need* to. For me, it’s just a matter of preference. Unlike the Red Hat distros, Ubuntu doesn’t automatically delete old Linux kernels if a new one gets installed during an update operation. If you don’t remember to do a `sudo apt autoremove` command after you update in order to get rid of them, you could fill up your `/boot/` partition, which could prevent future updates from installing a new kernel. By disabling the submenu and making the boot menu visible by default, I can see how many Linux kernels are installed as soon as I boot the machine. (But hey, that’s just me, and I’m kind of weird. Just ask anyone who knows me.) On a production machine, it would make more sense to leave both the submenu and the **Recovery** option enabled, and set a GRUB 2 password.

> You'll find the security section of the official GRUB 2 documentation at [http://www.gnu.org/software/grub/manual/grub/grub.html#Security](http://www.gnu.org/software/grub/manual/grub/grub.html#Security).

## Securely configuring BIOS/UEFI

This topic is different from anything we've looked at thus far because it has nothing to do with the operating system. Rather, we're now going to talk about the computer hardware.

Every computer motherboard has either a BIOS or a UEFI chip, which stores both the hardware configuration for the computer and the bootstrap instructions that are needed to start the boot process after the power is turned on. UEFI has replaced the old-style BIOS on newer motherboards, and it has more security features than the old BIOS had.

I can't give you any specific information about BIOS/UEFI setup because every model motherboard has a different way of doing things. What I can give you is some more generalized information.

When you think about BIOS/UEFI security, you might be thinking about disabling the ability to boot from anything other than the normal system drive. In the following screenshot, you see that I've disabled all SATA drive ports except for the one to which the system drive is connected:

![Figure 16.14: Disabling drive ports on my Hewlett-Packard Envy](img/file124.png)

Figure 16.14: Disabling drive ports on my Hewlett-Packard Envy

When computers are out in the open where the general public can have easy physical access to them, this might be a consideration. For servers that are locked away in their own secure room with limited access, there's no real reason to worry about this, unless the security requirements of some regulatory body dictate otherwise. For machines that are out in the open, having the whole disk encrypted would prevent someone from stealing data after booting from either an optical disk or a USB device. However, you may still have other reasons to prevent anyone from booting the machine from these alternate boot devices.

Another consideration might be if you work in a secure environment where super-sensitive data are handled. If you're worried about unauthorized exfiltration of sensitive data, you might consider disabling the ability to write to USB devices. This will also prevent people from booting the machine from USB devices:

![Figure 16.15: Disabling USB devices](img/file125.png)

Figure 16.15: Disabling USB devices

> At times, you might not want to completely disable a machine’s USB ports. Instead, you can leave them enabled and use USBGuard to allow only certain USB devices to be connected. Rather than do my own write-up about it, I’ll refer you to this excellently-written tutorial that I found:
> 
> > [https://www.cyberciti.biz/security/how-to-protect-linux-against-rogue-usb-devices-using-usbguard/](https://www.cyberciti.biz/security/how-to-protect-linux-against-rogue-usb-devices-using-usbguard/)
> 
> > The main catch with USBGuard is that it still won’t prevent someone from booting from a USB device.

However, there's more than just this to BIOS/UEFI security. Today's modern server CPUs come with a variety of security features to help prevent data breaches. For example, let's look at a list of security features that are implemented in Intel Xeon CPUs:

*   Identity-protection technology
*   Advanced Encryption Standard New Instructions
*   Trusted Execution Technology
*   Hardware-assisted virtualization technology

AMD, that plucky underdog in the CPU market, have their own new security features in their line of EPYC server CPUs. These features include:

*   Secure Memory Encryption
*   Secure Encrypted Virtualization

In any case, you would configure these CPU security options in your server's UEFI setup utility.

> You can read about Intel Xeon security features at [https://www.intel.com/content/www/us/en/newsroom/news/xeon-scalable-platform-built-sensitive-workloads.html](https://www.intel.com/content/www/us/en/newsroom/news/xeon-scalable-platform-built-sensitive-workloads.html).
> 
> > You can read about AMD EPYC security features at [https://semiaccurate.com/2017/06/22/amds-epyc-major-advance-security/](https://semiaccurate.com/2017/06/22/amds-epyc-major-advance-security/) and at [https://www.servethehome.com/amd-psb-vendor-locks-epyc-cpus-for-enhanced-security-at-a-cost/](https://www.servethehome.com/amd-psb-vendor-locks-epyc-cpus-for-enhanced-security-at-a-cost/)

And of course, for any machines that are out in the open, it's a good idea to password-protect the BIOS or UEFI:

![Figure 16.16: Password protect the BIOS/UEFI](img/file126.png)

Figure 16.16: Password protect the BIOS/UEFI

If for no other reason, do it to keep people from monkeying around with your settings.

Now that you know a bit about locking down BIOS/UEFI, let's talk about security checklists.

## Using a security checklist for system setup

Previously, I told you about OpenSCAP, which is a really useful tool for locking down your system with just a minimal amount of effort. OpenSCAP comes with various profiles that you can apply to help bring your systems into compliance with the standards of different regulatory agencies. However, there are certain things that OpenSCAP can't do for you. For example, certain regulatory agencies require that your server's hard drive be partitioned in a certain way, with certain directories separated out into their own partitions. If you've already set up your server with everything under one big partition, you can't fix that just by doing a remediation procedure with OpenSCAP. The process of locking down your server to ensure that it's compliant with any applicable security regulations has to begin before you even install the operating system. For this, you need the appropriate checklist.

There are a few different places where you can obtain a generic security checklist if that's all you need. The University of Texas at Austin published a generic checklist for Red Hat Enterprise Linux 7, which you can adjust if you need to use it with CentOS 7, Oracle Linux 7, or Scientific Linux 7\. (Sadly, they don’t offer anything that’s more up-to-date.)

You might find that some checklist items don't apply to your situation, and you can adjust them as required:

![Figure 16.17: University of Texas checklist](img/file127.png)

Figure 16.17: University of Texas checklist

For specific business fields, you'll need to get a checklist from the applicable regulatory body. If you work in the financial sector or with a business that accepts credit card payments, you'll need a checklist from the Payment Card Industry Security Standards Council:

![Figure 16.18: The PCI-DSS website](img/file128.png)

Figure 16.18: The PCI-DSS website

For healthcare organizations here in the US, there's HIPAA with its requirements. For publicly-traded companies here in the US, there's Sarbanes-Oxley with its requirements.

> You can get the University of Texas checklist at [https://wikis.utexas.edu/display/ISO/Operating+System+Hardening+Checklists](https://wikis.utexas.edu/display/ISO/Operating+System+Hardening+Checklists).
> 
> > You can get a PCI-DSS checklist at [https://www.pcisecuritystandards.org/](https://www.pcisecuritystandards.org/).
> 
> > You can get a HIPAA checklist at [https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html](https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html)
> 
> > You can get a Sarbanes-Oxley checklist at [https://www.sarbanes-oxley-101.com/sarbanes-oxley-checklist.htm](https://www.sarbanes-oxley-101.com/sarbanes-oxley-checklist.htm).

Other regulatory bodies may also have their own checklists. If you know that you have to deal with any of them, be sure to get the appropriate checklist.

## Summary

Once again, we've come to the conclusion of another chapter, and we covered a lot of cool topics. We started by looking at various ways to audit which services are running on your systems, and we saw some examples of what you probably don't want to see. We then saw how to use the password protection features of GRUB 2, and we saw the little quirks that we have to deal with when using those features. Next, we had a change of pace by looking at how to further lock down a system by properly setting up a system's BIOS/UEFI. Finally, we looked at why we need to begin preparations to set up a hardened system by obtaining and following the proper checklist.

Not only does this conclude another chapter, but it also concludes this book. However, this doesn't conclude your journey into the land of *Mastering Linux Security and Hardening*. Oh, no. As you continue this journey, you'll find that there's still more to learn, and still more that won't fit into the confines of just one book. Where you go from here mainly depends on the particular area of IT administration in which you work. Different types of Linux servers, whether they be web servers, DNS servers, or whatever else, have their own special security requirements, and you'll want to follow the learning path that best fits your needs.

I've enjoyed the part of the journey on which I've been able to accompany you. I hope that you've enjoyed it just as much as I have.

## Questions

1.  You need to see a list of network services that are listening for incoming connections. Which of the following commands would you use?
    1.  `sudo systemctl -t service --state=active`
    2.  `netstat -i`
    3.  `netstat -lp -A inet`
    4.  `sudo systemctl -t service --state=listening`
2.  Which of the following commands would you use to see only a list of established TCP connections?
    1.  `netstat -p -A inet`
    2.  `netstat -lp -A inet`
    3.  `sudo systemctl -t service --state=connected`
    4.  `sudo systemctl -t service --state=active`
3.  When Nmap tells you that a port is in an open state, what does that mean?
    1.  That the port is open on the firewall.
    2.  That the port is open on the firewall and that the service that's associated with that port is running.
    3.  That the port is accessible via the internet.
    4.  That the port's Access Control List is set to open.
4.  Which of these Nmap scan options would you most likely use to scan for open TCP ports?
    1.  `-sn`
    2.  `-sU`
    3.  `-sS`
    4.  `-sA`
5.  What do you want to do when resetting the root user password on a Red Hat/CentOS/AlmaLinux machine?
    1.  Ensure that AppArmor is in enforcing mode.
    2.  Ensure that SELinux is in enforcing mode.
    3.  Ensure that AppArmor is in complain mode.
    4.  Ensure that SELinux is in permissive mode.
6.  How does discovery mode work in Nmap?
    1.  It discovers network devices by sending ping packets to the network's broadcast address.
    2.  It discovers network devices by sending SYN packets to the network's broadcast address.
    3.  It sends out ARP packets for a local network and ping packets for a remote network.
    4.  It sends out ping packets for a local network and ARP packets for a remote network.
7.  You want to use Nmap to perform a UDP port scan of an entire subnet. Which of the following commands would you use?
    1.  `sudo nmap -sU 192.168.0.0`
    2.  `sudo nmap -U 192.168.0.0`
    3.  `sudo nmap -U 192.168.0.0/24`
    4.  `sudo nmap -sU 192.168.0.0/24`
8.  How would you begin the process of hardening a new computer system?
    1.  Apply an OpenSCAP profile when installing the operating system.
    2.  Begin the initial setup by following a checklist.
    3.  Install the operating system, then apply an OpenSCAP profile.
    4.  Install the operating system, then follow a hardening checklist.
9.  On a Red Hat/CentOS/AlmaLinux server, what would you most likely do to force users to enter a password before editing kernel parameters during bootup?
    1.  Enter the `sudo grub2-password` command.
    2.  Hand-edit the grub configuration file.
    3.  Enter the `sudo grub2-setpassword` command.
    4.  Enter the `sudo grub-setpassword` command.
    5.  Enter the `sudo grub-password` command.

## Further reading

*   netstat – The easy tutorial: [https://openmaniak.com/netstat.php](https://openmaniak.com/netstat.php)
*   Four ways to find which process is listening on a specific port: [https://www.putorius.net/process-listening-on-port.html](https://www.putorius.net/process-listening-on-port.html)
*   netstat versus ss User Guide: [https://computingforgeeks.com/netstat-vs-ss-usage-guide-linux/](https://computingforgeeks.com/netstat-vs-ss-usage-guide-linux/)
*   The official Nmap website: [https://nmap.org/](https://nmap.org/)
*   The GNU GRUB manual: [https://www.gnu.org/software/grub/manual/grub/grub.html](https://www.gnu.org/software/grub/manual/grub/grub.html)
*   How to boot Ubuntu 18.04 into emergency and rescue mode (An alternate method that still works on Ubuntu 22.04.): [https://linuxconfig.org/how-to-boot-ubuntu-18-04-into-emergency-and-rescue-mode](https://linuxconfig.org/how-to-boot-ubuntu-18-04-into-emergency-and-rescue-mode)
*   How to see the GRUB boot menu on Ubuntu 18.04: [https://askubuntu.com/questions/16042/how-to-get-to-the-grub-menu-at-boot-time](https://askubuntu.com/questions/16042/how-to-get-to-the-grub-menu-at-boot-time)

## Answers

1.  c
2.  a
3.  b
4.  c
5.  d
6.  c
7.  d
8.  b
9.  c