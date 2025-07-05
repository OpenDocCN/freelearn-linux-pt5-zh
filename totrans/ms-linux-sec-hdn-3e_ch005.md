# 4 Securing Your Server with a Firewall - Part 1

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file33.png)

Security is one of those things that's best done in layers. Security-in-depth, we call it. So, on any given corporate network, you will find a firewall appliance separating the Internet from the **demilitarized zone** (**DMZ**), where your Internet-facing servers are kept. You will also find a firewall appliance between the DMZ and the internal LAN, and firewall software installed on each individual server and client. We want to make it as tough as possible for intruders to reach their final destinations within our networks.

Interestingly, though, of all the major Linux distros, only the SUSE distros and the Red Hat type distros come with a firewall already set up and enabled. Newer versions of Ubuntu come with a pre-configured firewall, but you need to activate it by running a couple of simple commands.

Since the focus of this book is on hardening our Linux servers, we'll focus this chapter on that last level of defense: the firewalls on our servers and clients. We'll cover both of the command-line netfilter interfaces, which are **iptables** and **nftables**.

In this chapter, we'll cover the following topics:

*   An overview of the Linux firewall
*   An overview of iptables
*   An overview of nftables

In the next chapter, we'll cover **ufw** and **firewalld**, which are handy front-ends for iptables and nftables.

## Technical requirements

The code files for this chapter are available here: [https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-Second-Edition](https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-Second-Edition).

## An overview of the Linux firewall

In a typical business setting, especially in larger enterprises, you may encounter various types of firewalls in various places that can provide various types of functionality. Here are some examples:

*   Edge devices that separate the Internet from an internal network translate routable public IP addresses to non-routable private IP addresses. They can also provide various types of access control to keep out unauthorized people. By also providing various types of packet inspection services, they can help prevent attacks on the internal network, keep out malware, and prevent leakage of sensitive information from the internal network to the Internet.
*   Large enterprise networks are normally divided into subnetworks, or *subnets*, with each corporate department having a subnet to call its own. Best practice dictates separating the subnets with firewalls. This helps ensure that only authorized personnel can access any given subnet.
*   And, of course, you also have firewalls running on the individual servers and workstations. By providing a form of access control, they can help prevent an intruder who has compromised one machine from performing a lateral movement to another machine on the network. They can be also configured to prevent certain types of port scanning and **denial-of-service** (**DoS**) attacks.

For the first two items in the preceding list, you would likely see dedicated firewall appliances and teams of firewall administrators taking care of them. The third item in the list is where you, the Linux professional, come into the picture. In this chapter and the next, we'll look at the firewall technologies that come packaged with your Linux server and Linux workstation distros.

The name of the Linux firewall is **netfilter**. This netfilter code is compiled into the Linux kernel, and is what performs the actual packet filtering. There’s no way for human users to directly interface with netfilter, which means that we need some sort of helper program to interface with netfilter for us. There have been three helper programs, which are:

*   **ipchains**: This was the first one, and was part of the Linux kernel up through kernel version 2.4\. It’s now ancient history, so we won’t say any more about it.
*   **iptables**: This replaced ipchains in Linux kernel version 2.6\. It’s still used in a lot of Linux distros, but is rapidly disappearing.
*   **nftables**: This is the new kid on the block, and is rapidly replacing iptables. As we’ll see later, it has a lot of advantages over the older iptables.
*   All three of these helper programs do two things for us:
*   They provide a command-line interface for human users.
*   They take the commands that human users input and inject them into netfilter.
*   Making things even more interesting is that we also have helper programs for our helper programs. The **Uncomplicated Firewall** (**ufw**) was created by Ubuntu developers, and is a front-end for either iptables or nftables. Ubuntu comes with ufw already installed, and you can install ufw yourself on Debian and other Debian-type distros. On the Red Hat side, we have firewalld, which is also a front-end for either iptables or nftables. Note that firewalld is installed and active by default on all Red Hat-type distros, as well as the SUSE distros. It’s available as an option for Ubuntu and Debian. Both ufw and firewalld can vastly simplify the process of setting up a proper firewall. Still though, it’s sometimes helpful to know how to work with either bare iptables or bare nftables. So, let’s begin by looking at iptables.

## An overview of iptables

As I’ve mentioned, iptables is one of two command-line utilities that we can currently use to directly manage netfilter. It was originally introduced as a feature of Linux kernel version 2.6, so it's been around for a long time. With iptables, you do have a few advantages:

*   It's been around long enough that most Linux admins already know how to use it.
*   It's easy to use iptables commands in shell scripts to create your own custom firewall configuration.
*   It has great flexibility in that you can use it to set up a simple port filter, a router, or a virtual private network.
*   It still comes pre-installed on some Linux distros, although it’s rapidly getting replaced by nftables.
*   It's very well documented and has free of charge, book-length tutorials available on the Internet.

However, as you might know, there are also a few disadvantages:

*   IPv4 and IPv6 each require their own special implementation of iptables. So, if your organization still needs to run IPv4 while in the process of migrating to IPv6, you'll have to configure two firewalls on each server and run a separate daemon for each. (One for IPv4, the other for IPv6.)
*   If you need to do MAC bridging, that requires **ebtables**, which is the third component of iptables, with its own unique syntax.
*   **arptables**, the fourth component of iptables, also requires its own daemon and syntax.
*   Whenever you add a rule to a running iptables firewall, the entire iptables ruleset has to be reloaded, which can have an impact on performance.

Until recently, just plain iptables was the default firewall manager on every Linux distro. It still is on some distros, but Red Hat Enterprise Linux 7 and all of its offspring now use the new firewalld as an easier-to-use frontend for configuring iptables rules. Ubuntu comes with **Uncomplicated Firewall** (**ufw**), which is also an easy to use frontend for iptables on all Ubuntu versions up through 20.04.

In this chapter, we'll discuss setting up iptables firewall rules for both IPv4 and IPv6.

### Mastering the basics of iptables

iptables consists of five tables of rules, each with its own distinct purpose:

*   **Filter table**: For basic protection of our servers and clients, this might be the only table that we use.
*   **Network Address Translation (NAT) table**: NAT is used to connect the public Internet to private networks.
*   **Mangle table**: This is used to alter network packets as they go through the firewall.
*   **Raw table**: This is for packets that don't require connection tracking.
*   **Security table**: The security table is only used for systems that have SELinux installed.

Since we're currently only interested in basic host protection, we will only look at the filter table for the time being. (In a few moments, I'll show you a couple of fancy tricks that we can do with the mangle table.) Each table consists of chains of rules, and the filter table consists of the `INPUT`, `FORWARD`, and `OUTPUT` chains. Let’s look at this on an Ubuntu 20.04 virtual machine.

> Ubuntu 20.04 LTS comes with iptables, and Ubuntu 22.04 comes with nftables. However, even if you’re running Ubuntu 22.04 or newer, you’ll still want to learn how to work with iptables commands. The first reason is that Ubuntu 22.04 includes a cool feature that automatically translates iptables commands to nftables commands. This allows you to use any iptables scripts that you might already have without worrying about converting them to nftables format. The second reason is that once we start talking about Ubuntu’s **Uncomplicated Firewall** (**ufw**), you’ll see that you’ll still need to know iptables commands in order to configure it, regardless of which Ubuntu version that you’re using.

First, we'll look at our current configuration by using the `sudo iptables -L` command:

```
 donnie@ubuntu:~$ sudo iptables -L
 [sudo] password for donnie:
 Chain INPUT (policy ACCEPT)
 target prot opt source destination
 Chain FORWARD (policy ACCEPT)
 target prot opt source destination
 Chain OUTPUT (policy ACCEPT)
 target prot opt source destination
 donnie@ubuntu:~$
```

Remember that we said that you need a separate component of iptables to deal with IPv6\. Here, we'll use the `sudo ip6tables -L` command:

```
 donnie@ubuntu:~$ sudo ip6tables -L
 Chain INPUT (policy ACCEPT)
 target prot opt source destination
 Chain FORWARD (policy ACCEPT)
 target prot opt source destination
 Chain OUTPUT (policy ACCEPT)
 target prot opt source destination
 donnie@ubuntu:~$
```

In both cases, you can see that there are no rules and that the machine is wide open. (Understand that Ubuntu actually does come with a pre-configured Uncomplicated Firewall and you’ll also see some output that is specific to it, but we’ll ignore that for the time-being so that we can work directly with iptables.) We'll start by creating a rule that will allow us to pass incoming packets from servers that our host has requested a connection to:

```
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Here's the breakdown of this command:

*   **-A INPUT**: `-A` places a rule at the end of the specified chain, which in this case is the `INPUT` chain. We would have used `-I` had we wanted to place the rule at the beginning of the chain.
*   **-m**: This calls in an iptables module. In this case, we're calling in the `conntrack` module to track connection states. This module allows `iptables` to determine whether our client has made a connection to another machine, for example.
*   **--ctstate**: The `ctstate`, or connection state, portion of our rule is looking for two things. First, it's looking for a connection that the client established with a server. Then, it looks for the related connection that's coming back from the server in order to allow it to connect to the client. So, if a user were to use a web browser to connect to a website, this rule would allow packets from the web server to pass through the firewall to get to the user's browser.
*   **-j**: This stands for jump. Rules jump to a specific target, which in this case is `ACCEPT`. (Please don't ask me who came up with this terminology.) So, this rule will accept packets that have been returned from the server to which the client has requested a connection.

Our new ruleset looks like this:

```
 donnie@ubuntu:~$ sudo iptables -L
 Chain INPUT (policy ACCEPT)
 target prot opt source destination
 ACCEPT all -- anywhere anywhere ctstate RELATED,ESTABLISHED
 Chain FORWARD (policy ACCEPT)
 target prot opt source destination
 Chain OUTPUT (policy ACCEPT)
 target prot opt source destination
 donnie@ubuntu:~$
```

Next, we'll open up port `22` so that we can connect through Secure Shell:

```
sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
```

Here's the breakdown:

*   **-A INPUT**: As before, we want to place this rule at the end of the `INPUT` chain with `-A`.
*   **-p tcp**: `-p` indicates the protocol that this rule affects. This rule affects the TCP protocol, of which Secure Shell is a part.
*   **--dport ssh**: When an option name consists of more than one letter, we need to precede it with two dashes, instead of just one. The `--dport` option specifies the destination port on which we want this rule to operate. (Note that we could have also listed this portion of the rule as `--dport 22` since `22` is the number of the SSH port.)
*   **-j ACCEPT**: If we put this all together with `-j ACCEPT`, then we have a rule that allows other machines to connect to this one through Secure Shell.

Now, let's say that we want this machine to be a DNS server. For that, we need to open port `53` for both the TCP and the UDP protocols:

```
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
```

Finally, we have an almost complete, usable ruleset for our `INPUT` chain:

```
 donnie@ubuntu:~$ sudo iptables -L
 Chain INPUT (policy ACCEPT)
 target prot opt source destination
 ACCEPT all -- anywhere anywhere ctstate
 RELATED,ESTABLISHED
 ACCEPT tcp -- anywhere anywhere tcp dpt:ssh
 DROP all -- anywhere anywhere
 Chain FORWARD (policy ACCEPT)
 target prot opt source destination
 Chain OUTPUT (policy ACCEPT)
 target prot opt source destination
 donnie@ubuntu:~$
```

However, this is only *almost* complete, because there's still one little thing that we forgot. That is, we need to allow traffic for the loopback interface. This is okay because it gives us a good chance to see how to insert a rule where we want it if we don't want it at the end. In this case, we'll insert the rule at `INPUT 1`, which is the first position of the `INPUT` chain:

```
sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

Before you inserted the `ACCEPT` rule for the `lo` interface, you may have noticed that `sudo` commands were taking a long time to complete and that you were getting sudo: unable to resolve host. . .Resource temporarily unavailable messages. That's because `sudo` needs to know the machine's hostname so that it can know which rules are allowed to run on a particular machine. It uses the loopback interface to help resolve the hostname. If the `lo` interface is blocked, it takes longer for `sudo` to resolve the hostname.

Our ruleset now looks like this:

```
donnie@ubuntu:~$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
donnie@ubuntu:~$
```

Note how port `53` is listed as the domain port. To see port numbers instead of port names, we can use the `-n` switch:

```
donnie@ubuntu3:~$ sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:22
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:53
ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            udp dpt:53
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
donnie@ubuntu3:~$
```

Now, as things currently stand, we're still allowing *everything* to get through, because we still haven't created a rule that blocks what we haven't specifically allowed. Before we do that, though, let's look at a few more things that we might want to allow.

### Blocking ICMP with iptables

The conventional wisdom that you may have heard for most of your career is that we need to block all the packets from the **Internet Control Message Protocol** (**ICMP**). The idea you may have been told is to make your server invisible to hackers by blocking ping packets. Of course, there are some vulnerabilities that are associated with ICMP, such as the following:

By using a botnet, a hacker could inundate your server with ping packets from multiple sources at once, exhausting your server's ability to cope.

Certain vulnerabilities that are associated with the ICMP protocol can allow a hacker to either gain administrative privileges on your system, redirect your traffic to a malicious server, or crash your operating system.

By using some simple hacking tools, someone could embed sensitive data in the data field of an ICMP packet in order to secretly exfiltrate it from your organization.

However, while blocking certain types of ICMP packets is good, blocking all ICMP packets is bad. The harsh reality is that certain types of ICMP messages are necessary for the proper functionality of the network. Since the *drop all that's not allowed* rule that we'll eventually create also blocks ICMP packets, we'll need to create some rules that allow the types of ICMP messages that we have to have. So, here goes:

```
sudo iptables -A INPUT -m conntrack -p icmp --icmp-type 3 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack -p icmp --icmp-type 11 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack -p icmp --icmp-type 12 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
```

Here's the breakdown:

*   **-m conntrack**: As before, we're using the `conntrack` module to allow packets that are in a certain state. This time, though, instead of just allowing packets from a host to which our server has connected (`ESTABLISHED,RELATED`), we're also allowing `NEW` packets that other hosts are sending to our server.
*   **-p icmp**: This refers to the ICMP protocol.
*   **--icmp-type**: There are quite a few types of ICMP messages, which we'll outline next.

Here are the three types of ICMP messages that we want to allow:

*   **type 3**: These are the destination unreachable messages. Not only can they tell your server that it can't reach a certain host, but they can also tell it why. For example, if the server has sent out a packet that's too large for a network switch to handle, the switch will send back an ICMP message that tells the server to fragment that large packet. Without ICMP, the server would have connectivity problems every time it tries to send out a large packet that needs to be broken up into fragments.
*   **type 11**: Time exceeded messages let your server know that a packet that it has sent out has either exceeded its **Time-to-Live** (**TTL**) value before it could reach its destination, or that a fragmented packet couldn't be reassembled before the TTL expiration date.
*   **type 12**: Parameter problem messages indicate that the server had sent a packet with a bad IP header. In other words, the IP header is either missing an option flag, or it's of an invalid length.

Three common message types are conspicuously absent from our list:

*   **type 0** and **type 8**: These are the infamous ping packets. Actually, `type 8` is the echo request packet that you would send out to ping a host, while `type 0` is the echo reply that the host would return to let you know that it's alive. Of course, allowing ping packets to get through could be a big help with troubleshooting network problems. If that scenario ever comes up, you could just add a couple of `iptables` rules to temporarily allow pings.
*   **type 5**: Now, we have the infamous redirect messages. Allowing these could be handy if you have a router that can suggest more efficient paths for the server to use, but hackers can also use them to redirect you to someplace that you don't want to go. So, just block them.

There are lots more ICMP message types than I've shown here, but these are the only ones that we need to worry about for now.

When we use `sudo iptables -L`, we'll see our new ruleset, as things currently stand:

```
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
ACCEPT     icmp --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED icmp destination-unreachable
ACCEPT     icmp --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED icmp source-quench
ACCEPT     icmp --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED icmp time-exceeded
ACCEPT     icmp --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED icmp parameter-problem
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

Looks good, eh? Well, not really. We haven't blocked anything with this ruleset yet. So, let's take care of that.

### Blocking everything that isn't allowed with iptables

To start blocking stuff that we don't want, we have to do one of two things. We can set a default `DROP` or `REJECT` policy for the `INPUT` chain, or we can leave the policy set to `ACCEPT` and create a `DROP` or `REJECT` rule at the end of the `INPUT` chain. Which one you choose is really a matter of preference. (Of course, before you choose one over the other, you might want to check your organization's policy manual to see if your employer has a preference.)

The difference between `DROP` and `REJECT` is that `DROP` blocks packets without sending any message back to the sender. `REJECT` blocks packets, and then sends a message back to the sender about why the packets were blocked. For our present purposes, let's say that we just want to `DROP` packets that we don't want to get through.

> Tip:
> 
> > There are times when `DROP` is better, and times when `REJECT` is better. Use `DROP` if it's important to make your host invisible. (Although, even that isn't that effective, because there are other ways to discover hosts.) If you need your hosts to inform other hosts about why they can't make a connection, then use `REJECT`. The big advantage of `REJECT` is that it will let connecting hosts know that their packets are being blocked so that they will know to immediately quit trying to make a connection. With `DROP`, the host that's trying to make the connection will just keep trying to make the connection until it times out.

To create a `DROP` rule at the end of the `INPUT` chain, use this command:

```
donnie@ubuntu:~$ sudo iptables -A INPUT -j DROP
donnie@ubuntu:~$
```

To set a default `DROP` policy instead, we can use the this command:

```
donnie@ubuntu:~$ sudo iptables -P INPUT DROP
donnie@ubuntu:~$
```

The big advantage of setting up a default `DROP` or `REJECT` policy is that it makes it easier to add new `ACCEPT` rules if need be. This is because if we decide to keep the default `ACCEPT` policy and create a `DROP` or `REJECT` rule instead, that rule has to be at the bottom of the list.

Since `iptables` rules are processed in order, from top to bottom, any `ACCEPT` rules that come after that `DROP` or `REJECT` rule would have no effect. You would need to insert any new `ACCEPT` rules above that final `DROP` or `REJECT` rule, which is just a tiny bit less convenient than just being able to append them to the end of the list. For now, in order to illustrate my next point, I've just left the default `ACCEPT` policy and added the `DROP` rule.

When we look at our new ruleset, we'll see something that's rather strange:

```
donnie@ubuntu:~$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
. . .
. . .
ACCEPT     icmp --  anywhere             anywhere             ctstate NEW,RELATED,ESTABLISHED icmp parameter-problem
DROP       all  --  anywhere             anywhere            
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
. . .
. . .
```

The first rule and the last rule of the `INPUT` chain look the same, except that one is a `DROP` and the other is an `ACCEPT`. Let's look at it again with the `-v` (verbose) option:

```
donnie@ubuntu:~$ sudo iptables -L -v
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target prot opt in out source destination 
   67 4828 ACCEPT all -- lo any anywhere anywhere 
  828 52354 ACCEPT all -- any any anywhere anywhere ctstate RELATED,ESTABLISHED
. . .
. . .
    0 0 ACCEPT icmp -- any any anywhere anywhere ctstate NEW,RELATED,ESTABLISHED icmp parameter-problem
  251 40768 DROP all -- any any anywhere anywhere 
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target prot opt in out source destination 
. . .
. . .
```

Now, we can see that `lo`, for loopback, shows up under the `in` column of the first rule, and that `any` shows up under the `in` column of the last rule. We can also see that the `-v` switch shows the number of packets and bytes that have been counted by each rule. So, in the preceding example, we can see that the `ctstate RELATED,ESTABLISHED` rule has accepted 828 packets and 52,354 bytes. The `DROP all` rule has blocked 251 packets and 40,763 bytes.

This all looks great, except that if we were to reboot the machine right now, the rules would disappear. The final thing that we need to do is make them permanent. There are several ways to do this, but the simplest way to do this on an Ubuntu machine is to install the `iptables-persistent` package:

```
sudo apt install iptables-persistent
```

During the installation process, you'll be presented with two screens that ask you whether you want to save the current set of `iptables rules`. The first screen will be for IPv4 rules, while the second will be for IPv6 rules:

![19501_04_01.png](img/file34.png)

19501_04_01.png

You'll now see two new rules files in the `/etc/iptables/` directory:

```
 donnie@ubuntu:~$ ls -l /etc/iptables*
 total 8
 -rw-r--r-- 1 root root 336 Oct 10 10:29 rules.v4
 -rw-r--r-- 1 root root 183 Oct 10 10:29 rules.v6
 donnie@ubuntu:~$
```

If you were to reboot the machine now, you'd see that your iptables rules are still there and in effect. The only slight problem with `iptables-persistent` is that it won't save any subsequent changes that you make to the rules. That's okay, though. I'll show you how to deal with that in just a bit.

#### Hands-on lab for basic iptables usage

You'll complete this lab on your Ubuntu 20.04 virtual machine. Follow these steps to get started:

1.  Shut down your Ubuntu virtual machine and create a snapshot. After you boot it back up, look at your iptables rules, or lack thereof, by using this command:

```
sudo iptables -L
```

1.  With a default Ubuntu setup, the Uncomplicated Firewall (ufw) service is already running, albeit with an unactivated firewall configuration. We want to disable it to work directly with iptables. Do that now with this command:

```
sudo systemctl disable --now ufw
```

1.  Create the rules that you need for a basic firewall, allowing for Secure Shell access, DNS queries and zone transfers, and the proper types of ICMP. Deny everything else:

```
sudo iptables -A INPUT -m conntrack  --ctstate ESTABLISHED,RELATED  -j ACCEPT
sudo iptables -A INPUT -p tcp --dport ssh -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 53 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 53 -j ACCEPT
sudo iptables -A INPUT -m conntrack -p icmp --icmp-type 3 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack -p icmp --icmp-type 11 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack -p icmp --icmp-type 12 --ctstate NEW,ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -j DROP
```

1.  Use this command to view the results:

```
sudo iptables -L
```

1.  Oops – it looks like you forgot about that loopback interface. Add a rule for it at the top of the list:

```
sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

1.  View the results by using the following two commands. Note the difference between the output of each:

```
 sudo iptables -L
 sudo iptables -L -v
```

1.  Install the `iptables-persistent` package and choose to save the IPv4 and IPv6 rules when prompted:

```
sudo apt install iptables-persistent
```

1.  Reboot the virtual machine and verify that your rules are still active.
2.  Keep this virtual machine; you'll be adding more to it in the next hands-on lab.

That's the end of this lab—congratulations!

### Blocking invalid packets with iptables

If you've been in the IT business for any length of time, you're most likely familiar with the good old TCP three-way handshake. If you're not, no worries. Here's the simplified explanation.

Let's say that you're sitting at your workstation, and you pull up Firefox to visit a website. To access that website, your workstation and the web server have to set up the connection. Here's what happens:

Your workstation sends a packet with only the `SYN` flag set to the web server. This is your workstation's way of saying, "Hello, Mr. Server. I'd like to make a connection with you."

After receiving your workstation's `SYN` packet, the web server sends back a packet with the `SYN` and `ACK` flags set. With this, the server is saying, "Yeah, bro. I'm here, and I'm willing to connect with you."

Upon receipt of the `SYN-ACK` packet, the workstation sends back a packet with only the `ACK` flag set. With this, the workstation is saying, "Cool deal, dude. I'm glad to connect with you."

Upon receipt of the `ACK` packet, the server sets up the connection with the workstation so that they can exchange information.

This sequence works the same way for setting up any kind of TCP connection. This includes connections that involve Secure Shell, telnet, and the various mail server protocols, among other things.

However, clever people can use a variety of tools to craft TCP packets with some very weird combinations of flags. With these so-called *invalid* packets, a few things could happen:

The invalid packets could be used to elicit responses from the target machine in order to find out what operating system it's running, what services are running on it, and which versions of the services are running.

The invalid packets could be used to trigger certain sorts of security vulnerabilities on the target machine.

Some of these invalid packets require more processing power than what normal packets require, which could make them useful for performing Denial-of-Service (DoS) attacks.

Now, in truth, the `DROP all` rule at the end of the filter table's `INPUT` chain will block some of these invalid packets. However, there are some things that this rule might miss. And even if we could count on it to block all the invalid stuff, this still isn't the most efficient way of doing things. By depending on this `DROP all` rule, we're allowing these invalid packets to travel through the entire `INPUT` chain, looking for a rule that will let them through. When no `ALLOW` rule is found for them, they'll finally get blocked by the `DROP all` rule, which is the last rule in the chain. So, what if we could find a more efficient solution?

Ideally, we'd like to block these invalid packets before they travel through the entire `INPUT` chain. We could do that with a `PREROUTING` chain, but the filter table doesn't have a `PREROUTING` chain. Therefore, we need to use the `PREROUTING` chain of the mangle table instead. Let's start by adding these two rules:

```
 donnie@ubuntu:~$ sudo iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP

 donnie@ubuntu:~$ sudo iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
 donnie@ubuntu:~$
```

The first of these rules will block most of what we would consider *invalid*. However, there are still some things that it misses. Due to this, we added the second rule, which blocks all `NEW` packets that are not `SYN` packets. Now, let's see what we have:

```
 donnie@ubuntu:~$ sudo iptables -L
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  anywhere             anywhere            
 ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
 ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
 DROP       all  --  anywhere             anywhere            

 Chain FORWARD (policy ACCEPT)
 target     prot opt source               destination         

 Chain OUTPUT (policy ACCEPT)
 target     prot opt source               destination         
 donnie@ubuntu:~$
```

Hmm...

We can't see our new rules, can we? That's because, by default, `iptables -L` only shows rules for the filter table. We need to see what we've just placed into the mangle table, so let's do this, instead:

```
 donnie@ubuntu:~$ sudo iptables -t mangle -L
 Chain PREROUTING (policy ACCEPT)
 target     prot opt source               destination         
 DROP       all  --  anywhere             anywhere             ctstate INVALID
 DROP       tcp  --  anywhere             anywhere             tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW

 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 Chain FORWARD (policy ACCEPT)
 target     prot opt source               destination         
 Chain OUTPUT (policy ACCEPT)
 target     prot opt source               destination         
 Chain POSTROUTING (policy ACCEPT)
 target     prot opt source               destination         
 donnie@ubuntu:~$
```

Here, we used the `-t mangle` option to indicate that we want to see the configuration for the mangle table. Something rather curious that you may have noticed is how iptables renders the rule that was created by the `sudo iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP` command. For some reason, it renders like this:

```
DROP     tcp  --  anywhere   anywhere    tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW
```

It looks strange, but don't let that throw you. It still means that it's blocking `NEW` packets that aren't `SYN` packets.

Previously, I mentioned that the `iptables-persistent` package won't save subsequent changes to your `iptables` rules. So, as things stand now, the mangle table rules that we just added will disappear once I reboot this virtual machine. To make these changes permanent, I'll use the `iptables-save` command to save a new file in my own home directory. Then, I'll copy the file over to the `/etc/iptables` directory, replacing the original one:

```
 donnie@ubuntu:~$ sudo iptables-save > rules.v4
 [sudo] password for donnie:
 donnie@ubuntu:~$ sudo cp rules.v4 /etc/iptables/
 donnie@ubuntu:~$
```

To test this, we'll use a handy utility called Nmap. It's a free utility that you can install on your Windows, Mac, or Linux workstation. Or, if you don't want to install it on your host machine, you can install it on one of your Linux virtual machines. It's in the normal repositories of Debian/Ubuntu, RHEL/CentOS 7, and RHEL/AlmaLinux 8\. So, just install the Nmap package with the appropriate install command for your distro. If you want to install Nmap on your Windows or Mac host machine, you'll need to download it from the Nmap website.

> You can download Nmap from the official website, which can be found here: [https://nmap.org/download.html](https://nmap.org/download.html).

With our new mangle table rules in place, let's perform an XMAS scan of our Ubuntu machine. I have Nmap installed here on the Fedora workstation that I'm currently using, so I'll just use this for now. I can do this like so:

```
[donnie@fedora-teaching ~]$ sudo nmap -sX 192.168.0.15
 [sudo] password for donnie:
 Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-26 21:20 EDT
 Nmap scan report for 192.168.0.15
 Host is up (0.00052s latency).
 All 1000 scanned ports on 192.168.0.15 are open|filtered
 MAC Address: 08:00:27:A4:95:1A (Oracle VirtualBox virtual NIC)

 Nmap done: 1 IP address (1 host up) scanned in 21.41 seconds
 [donnie@fedora-teaching ~]$
```

By default, Nmap only scans the most commonly used 1,000 ports. The XMAS scan sends invalid packets that consist of the FIN, PSH, and URG flags. The fact that all 1,000 scanned ports show as `open|filtered` means that the scan was blocked, and that Nmap can't determine the true state of the ports. (In reality, port `22` is open.) We can view the result to see which rule did the blocking. (To simplify things a bit, I'll only show the output for the `PREROUTING` chain since it's the only mangle table chain that's doing anything):

```
donnie@ubuntu:~$ sudo iptables -t mangle -L -v
 Chain PREROUTING (policy ACCEPT 2898 packets, 434K bytes)
  pkts bytes target     prot opt in     out     source               destination    

  2000 80000 DROP  all  --  any    any     anywhere             anywhere             ctstate INVALID

  0     0 DROP       tcp  --  any    any     anywhere             anywhere             tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW

 . . .
 . . .
 donnie@ubuntu:~$
```

Here, you can see that the first rule – the `INVALID` rule – blocked 2,000 packets and 80,000 bytes. Now, let's zero out the counter so that we can do another scan:

```
 donnie@ubuntu:~$ sudo iptables -t mangle -Z
 donnie@ubuntu:~$ sudo iptables -t mangle -L -v
 Chain PREROUTING (policy ACCEPT 22 packets, 2296 bytes)
 pkts bytes target prot opt in out source destination
 0 0 DROP all -- any any anywhere anywhere ctstate INVALID
 0 0 DROP tcp -- any any anywhere anywhere tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW

 . . .
 . . .
 donnie@ubuntu:~$
```

This time, let's do a Window scan, which bombards the target machine with `ACK` packets:

```
[donnie@fedora-teaching ~]$ sudo nmap -sW 192.168.0.15
 Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-26 21:39 EDT
 Nmap scan report for 192.168.0.15
 Host is up (0.00049s latency).
 All 1000 scanned ports on 192.168.0.15 are filtered
 MAC Address: 08:00:27:A4:95:1A (Oracle VirtualBox virtual NIC)

 Nmap done: 1 IP address (1 host up) scanned in 21.44 seconds
 [donnie@fedora-teaching ~]$
```

As before, the scan was blocked, as indicated by the message that all 1,000 scanned ports have been filtered. Now, let's view what we have on the target Ubuntu machine:

```
 donnie@ubuntu:~$ sudo iptables -t mangle -L -v
 Chain PREROUTING (policy ACCEPT 45 packets, 6398 bytes)
  pkts bytes target     prot opt in     out     source               destination         

  0     0 DROP       all  --  any    any     anywhere             anywhere             ctstate INVALID

  2000 80000 DROP       tcp  --  any    any     anywhere             anywhere             tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW

 . . .
 . . .      
 donnie@ubuntu:~$
```

This time, we can see that our invalid packets got past the first rule, but were blocked by the second rule.

Now, just for fun, let's clear out the mangle table rules and do our scans again. We'll use the `-D` option to delete the two rules from the mangle table:

```
 donnie@ubuntu:~$ sudo iptables -t mangle -D PREROUTING 1
 donnie@ubuntu:~$ sudo iptables -t mangle -D PREROUTING 1
 donnie@ubuntu:~$
```

When you delete a rule, you have to specify the rule number, just like you do when you insert a rule. Here, I specified rule 1 twice, because deleting the first rule moved the second rule up to first place. Now, let's verify that the rules are gone:

```
 donnie@ubuntu:~$ sudo iptables -t mangle -L
 Chain PREROUTING (policy ACCEPT)
 target prot opt source destination

 . . .
 . . .
 donnie@ubuntu:~$
```

Yes, they are. Cool. Now, let's see what we get when we perform another XMAS scan:

```
 [donnie@fedora-teaching ~]$ sudo nmap -sX 192.168.0.15
 [sudo] password for donnie:
 Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-26 21:48 EDT
 Nmap scan report for 192.168.0.15
 Host is up (0.00043s latency).
 All 1000 scanned ports on 192.168.0.15 are open|filtered
 MAC Address: 08:00:27:A4:95:1A (Oracle VirtualBox virtual NIC)

 Nmap done: 1 IP address (1 host up) scanned in 21.41 seconds
 [donnie@fedora-teaching ~]$
```

Even without the mangle table rules, it shows that my scan is still blocked. What's up with that? This is happening because I still have the `DROP all` rule at the end of the `INPUT` table. Let's disable that and see what we get with another scan.

First, I need to see what the rule number is:

```
donnie@ubuntu:~$ sudo iptables -L
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  anywhere             anywhere            
 ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
 ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
 DROP       all  --  anywhere             anywhere            

 Chain FORWARD (policy ACCEPT)
 target     prot opt source               destination         

 Chain OUTPUT (policy ACCEPT)
 target     prot opt source               destination         
 donnie@ubuntu:~$
```

Counting down, I can see that it's rule number 4, so I'll delete it:

```
donnie@ubuntu:~$ sudo iptables -D INPUT 4
donnie@ubuntu:~$donnie@ubuntu:~$ sudo iptables -L
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  anywhere             anywhere            
 ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
 ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh

 Chain FORWARD (policy ACCEPT)
 target     prot opt source               destination         

 Chain OUTPUT (policy ACCEPT)
 target     prot opt source               destination         
 donnie@ubuntu:~$
```

Now, for the XMAS scan:

```
[donnie@fedora-teaching ~]$ sudo nmap -sX 192.168.0.15
 Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-26 21:49 EDT
 Nmap scan report for 192.168.0.15
 Host is up (0.00047s latency).
 Not shown: 999 closed ports
 PORT STATE SERVICE
 22/tcp open|filtered ssh
 MAC Address: 08:00:27:A4:95:1A (Oracle VirtualBox virtual NIC)

 Nmap done: 1 IP address (1 host up) scanned in 98.76 seconds
 [donnie@fedora-teaching ~]$
```

This time, the scan shows that 999 ports are closed and that port `22`, the SSH port, is either open or filtered. This shows that the scan is no longer being blocked by anything.

### Restoring the deleted rules

When I used the `iptables -D` commands, I only deleted the rules from the runtime configuration, and not from the `rules.v4` configuration file. To restore the rules that I deleted, I can either reboot the machine or restart the `netfilter-persistent` service. The latter choice is quicker, so I'll activate it like this:

```
donnie@ubuntu:~$ sudo systemctl restart netfilter-persistent
[sudo] password for donnie:
donnie@ubuntu:~$
```

The `iptables -L` and `iptables -t mangle -L` commands will show that all the rules are now back in effect.

#### Hands-on lab for blocking invalid IPv4 packets

For this lab, you'll use the same virtual machine that you used for the previous lab. You won't replace any of the rules that you already have. Rather, you'll just add a couple. Let's get started:

1.  Look at the rules for the filter and the mangle tables. (Note that the `-v` option shows you statistics about packets that were blocked by `DROP` and `REJECT` rules.) Then, zero out the blocked packets counter:

```
sudo iptables -L -v
sudo iptables -t mangle -L -v
sudo iptables -Z
sudo iptables -t mangle -Z
```

1.  From either your host machine or another virtual machine, perform the NULL and Windows Nmap scans against the virtual machine:

```
sudo nmap -sN ip_address_of_your_VM
sudo nmap -sW ip_address_of_your_VM
```

1.  Repeat *Step 1*. You should see a large jump in the number of packets that were blocked by the final `DROP` rule in the `INPUT` chain of the filter table:

```
sudo iptables -L -v
sudo iptables -t mangle -L -v
```

1.  Make the firewall work more efficiently by using the `PREROUTING` chain of the mangle table to drop invalid packets, such as those that are produced by the two Nmap scans that we just performed. Add the two required rules with the following two commands:

```
sudo iptables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP
sudo iptables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
```

1.  Save the new configuration to your own home directory. Then, copy the file to its proper location and zero out the blocked packet counters:

```
sudo iptables-save > rules.v4
sudo cp rules.v4 /etc/iptables
sudo iptables -Z
sudo iptables -t mangle -Z
```

1.  Perform only the NULL scan against the virtual machine:

```
sudo nmap -sN ip_address_of_your_VM
```

1.  Look at the `iptables` ruleset and observe which rule was triggered by the Nmap scan:

```
sudo iptables -L -v
sudo iptables -t mangle -L -v
```

1.  This time, perform just the Windows scan against the virtual machine:

```
sudo nmap -sW ip_address_of_your_VM
```

1.  Observe which rule was triggered by this scan:

```
sudo iptables -L -v
sudo iptables -t mangle -L -v
```

That's the end of this lab—congratulations!

### Protecting IPv6

I know, you're used to having all networking based on IPv4, with its nice, short, easy to use IP addresses. However, that can't last forever, considering that the world is now out of new IPv4 addresses. IPv6 offers a much larger address space that will last for a long time to come. Some organizations, especially wireless carriers, are either in the process of switching over to IPv6 or have already switched to it.

So far, all we've covered is how to set up an IPv4 firewall with iptables. But remember what we said before. With iptables, you need one daemon and one set of rules for the IPv4 network, and another daemon and set of rules for IPv6\. This means that when using iptables to set up a firewall, protecting IPv6 means doing everything twice. Most Linux distros come with IPv6 networking enabled by default, so you either need to protect it with a firewall or disable it. Otherwise, your IPv6 address will still be open for attack since the IPv4 firewall that you've just configured won't protect it. This is true even if your server or device is facing the IPv4 Internet because there are ways to tunnel IPv6 packets through an IPv4 network. Fortunately, the commands for setting up an IPv6 firewall are mostly the same as what we've just covered. The biggest difference is that instead of using the `iptables` command, you'll use the `ip6tables` command. Let's start with our basic setup, just like what we did for IPv4:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -i lo -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p tcp --dport ssh -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p tcp --dport 53 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p udp --dport 53 -j ACCEPT
```

The other big difference between IPv4 and IPv6 is that with IPv6, you must allow more types of ICMP messages than you need to for IPv4\. This is due to the following reasons:

*   With IPv6, new types of ICMP messages have replaced the **Address Resolution Protocol** (**ARP**).
*   With IPv6, dynamic IP address assignments are normally done by exchanging ICMP discovery messages with other hosts, rather than by DHCP.
*   With IPv6, echo requests and echo replies, the infamous ping packets, are required when you need to tunnel IPv6 packets through an IPv4 network.

And of course, we still need the same types of ICMP messages that we need for IPv4\. So, let's start with them:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 1 -j ACCEPT
[sudo] password for donnie: 
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 2 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 3 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 4 -j ACCEPT
donnie@ubuntu3:~$
```

These message types are as follows, in order of appearance:

*   Destination unreachable
*   Packet too big
*   Time exceeded
*   Parameter problem with the packet header

Next, we'll enable echo requests (type 128) and echo responses (type 129) so that IPv6 over IPv4 tunneling will work:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 128 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 129 -j ACCEPT
donnie@ubuntu3:~$
```

> The Teredo protocol is one of a few different ways to tunnel IPv6 packets across an IPv4 network. This protocol is what requires echo requests and echo replies, the infamous ping packets, to be allowed through a firewall. However, if you search through your distro repositories for a Teredo package, you won't find it. That's because the Linux implementation of the Teredo protocol is called miredo. So, when installing the Teredo protocol on a Linux machine, you'll need to install the `miredo` and `miredo-server` packages.

The next four ICMP message types that we need are for the Link-local Multicast Receiver Notification messages:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT --protocol icmpv6 --icmpv6-type 130
donnie@ubuntu3:~$ sudo ip6tables -A INPUT --protocol icmpv6 --icmpv6-type 131
donnie@ubuntu3:~$ sudo ip6tables -A INPUT --protocol icmpv6 --icmpv6-type 132
donnie@ubuntu3:~$ sudo ip6tables -A INPUT --protocol icmpv6 --icmpv6-type 143
donnie@ubuntu3:~$
```

These are as follows, in order of appearance:

*   Listener query
*   Listener report
*   Listener done
*   Listener report v2

Next up is our neighbor and router discovery message types:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 134 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 135 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 136 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 141 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 142 -j ACCEPT
donnie@ubuntu3:~$
```

These are as follows, in order of appearance:

*   Router solicitation
*   Router advertisement
*   Neighbor solicitation
*   Neighbor advertisement
*   Inverse neighbor discovery solicitation
*   Inverse neighbor discovery advertisement

Space doesn't permit me to go into the details of these message types. So, for now, let's just say that they're required in order for IPv6 hosts to dynamically assign themselves an IPv6 address.

For times when you're using security certificates to authenticate the routers that are attached to your network, you'll also need to allow **Secure Neighbor Discovery** (**SEND**) messages:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 148 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 149 -j ACCEPT
donnie@ubuntu3:~$
```

Are your fingers tired yet? If so, have no fear. This next group of ICMP rules is the last one. This time, we need to allow Multicast Router Discovery messages:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 151 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 152 -j ACCEPT
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -p icmpv6 --icmpv6-type 153 -j ACCEPT
donnie@ubuntu3:~$
```

Finally, we'll add our `DROP` rule to block everything else:

```
donnie@ubuntu3:~$ sudo ip6tables -A INPUT -j DROP
donnie@ubuntu3:~$
```

I know you're thinking, “Wow, that's a lot of hoops to jump through just to set up a basic firewall”. And yeah, you're right, especially when you also need to configure rules for IPv6\. Soon, I'll show you what the Ubuntu folk came up with to make things simpler.

> You can get the whole scoop on how to use iptables on Ubuntu here: [https://help.ubuntu.com/community/IptablesHowTo](https://help.ubuntu.com/community/IptablesHowTo).

#### Hands-on lab for ip6tables

For this lab, you'll use the same Ubuntu virtual machine that you used in the previous iptables labs. You'll leave the IPv4 firewall setup that's already there as-is and create a new firewall for IPv6\. Let's get started:

1.  View your IPv6 rules, or lack thereof, with this command:

```
sudo ip6tables -L
```

1.  Create the IPv6 firewall. Due to formatting constraints, I can't list the entire code block of commands here. You can find the respective commands in this chapter's directory, in the code file that you can download from the Packt Publishing website.
2.  View the new ruleset by using the following command:

```
sudo ip6tables -L
```

1.  Next, set up the mangle table rules for blocking invalid packets:

```
sudo ip6tables -t mangle -A PREROUTING -m conntrack --ctstate INVALID -j DROP
sudo ip6tables -t mangle -A PREROUTING -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
```

1.  Save the new ruleset to a file in your own home directory, and then transfer the rules file to the proper location:

```
sudo ip6tables-save > rules.v6
sudo cp rules.v6 /etc/iptables/
```

1.  Obtain the IPv6 address of the virtual machine with this command:

```
ip a
```

1.  On the machine on which you installed Nmap, perform a Windows scan of the virtual machine's IPv6 address. The command will look like this, except with your own IP address:

```
sudo nmap -6 -sW fe80::a00:27ff:fe9f:d923
```

1.  On the virtual machine, observe which rule was triggered with this command:

```
sudo ip6tables -t mangle -L -v
```

You should see non-zero numbers for the packet counters for one of the rules.

1.  On the machine on which you installed Nmap, perform an XMAS scan of the virtual machine's IPv6 address. The command will look like this, except with your own IP address:

```
sudo nmap -6 -sX fe80::a00:27ff:fe9f:d923
```

1.  As before, on the virtual machine, observe which rule was triggered by this scan:

```
sudo ip6tables -t mangle -L -v
```

1.  Shut down the virtual machine, and restore it from the snapshot that you created at the beginning of the *Hands-on lab for basic iptables usage*.

That's the end of this lab – congratulations!

So far, you've seen the good, the bad, and the ugly of `iptables`. It's very flexible, and there's a lot of power in the `iptables` commands. If you're clever at shell scripting, you can create some rather complex shell scripts that you can use to deploy firewalls on the machines all across your network.

On the other hand, getting everything right can be quite complex, especially if you need to consider that your machines have to run both IPv4 and IPv6, and that everything you do for IPv4 has to be done again for IPv6\. (If you're a masochist, you might actually enjoy it.)

## nftables – a more universal type of firewall system

Now, let's turn our attention to nftables, the new kid on the block. So, what does nftables bring to the table? (Yes, the pun was intended.):

*   You can forget about needing separate daemons and utilities for all of the different networking components. The functionality of iptables, ip6tables, ebtables, and arptables is now all combined in one neat package. The nft utility is now the only firewall utility that you'll need.
*   With nftables, you can create multi-dimensional trees to display your rulesets. This makes troubleshooting vastly easier because it's now easier to trace a packet all the way through all of the rules.
*   With iptables, you have the filter, NAT, mangle, and security tables installed by default, whether or not you use each one.
*   With nftables, you only create the tables that you intend to use, resulting in enhanced performance.
*   Unlike iptables, you can specify multiple actions in one rule, instead of having to create multiple rules for each action.
*   Unlike iptables, new rules get added atomically. (That's a fancy way of saying that there's no longer a need to reload the entire ruleset in order to just add one rule.)
*   nftables has its own built-in scripting engine, allowing you to write scripts that are more efficient and more human-readable.
*   If you already have lots of iptables scripts that you still need to use, you can install a set of utilities that will help you convert them into nftables format. (That is, unless you’re running Ubuntu 22.04, which can automatically translate iptables commands for you.)

Although nftables was created by Red Hat, Ubuntu was the first enterprise-grade Linux distro to offer it as an option, beginning with Ubuntu 16.04\. It’s now the default option for Ubuntu 22.04, SUSE, OpenSUSE, and the RHEL 8/9-type distros. Let’s begin by looking at some basic nftables concepts.

### Learning about nftables tables and chains

If you're used to iptables, you might recognize some of the nftables terminology. The only problem is that some of the terms are used in different ways, with different meanings. Let's go over some examples so that you’ll know what I'm talking about:

*   **Tables**: Tables in nftables refer to a particular protocol family. The table types are ip, ip6, inet, arp, bridge, and netdev.
*   **Chains**: Chains in nftables roughly equate to tables in iptables. For example, in nftables, you could have filter, route, or NAT chains.

#### Getting started with nftables

Let's start with a clean snapshot of our Ubuntu 22.04 virtual machine, since it comes with nftables already installed.

> Tip:
> 
> > You can use Ubuntu 20.04 if you really want to, but you’ll first have to install nftables by doing:

```
sudo apt install nftables
```

Now, let's take a look at the list of installed tables:

```
sudo nft list tables
```

You didn't see any tables, did you? So, let's load some up.

### Configuring nftables on Ubuntu

On the Ubuntu virtual machine that we'll be using, the default `nftables.conf` file is nothing more than a meaningless placeholder. The file you need, which you'll copy over to replace the default `nftables.conf` file, is elsewhere. Let's check it out.

First, we'll go into the directory where the sample configurations are stored and list the sample configuration files:

```
cd /usr/share/doc/nftables/examples/
ls -l
```

You should see something similar to this:

```
donnie@ubuntu2204-packt:/usr/share/doc/nftables/examples$ ls -l
total 124
-rw-r--r-- 1 root root  1016 Mar 23  2022 all-in-one.nft
-rw-r--r-- 1 root root   129 Mar 23  2022 arp-filter.nft
. . .
. . .
-rwxr-xr-x 1 root root   817 Mar 23  2022 workstation.nft
donnie@ubuntu2204-packt:/usr/share/doc/nftables/examples$
```

If you view the contents of the `workstation.nft` file, you'll see that it's the one we need.

Next, we'll copy the workstation file over to the `/etc` directory, changing its name to `nftables.conf`. (Note that this will overwrite the old `nftables.conf` file, which is what we want.):

```
sudo cp workstation.nft /etc/nftables.conf
```

Here's the breakdown of what you'll see in the `/etc/nftables.conf` file that you'll be using:

*   **#!/usr/sbin/nft -f**: Although you can create normal Bash shell scripts with nftables commands, it's better to use the built-in scripting engine that's included with nftables. That way, we can make our scripts more human-readable, and we don't have to type `nft` in front of everything we want to execute.
*   **flush ruleset**: We want to start with a clean slate, so we'll flush out any rules that may have already been loaded.
*   **table inet filter**: This creates an inet family filter, which works for both IPv4 and IPv6\. The name of this table is `filter`, but it could just as well have been something a bit more descriptive.
*   **chain input**: Within the first pair of curly brackets, we have a chain called `input`. (Again, the name could have been something more descriptive.)
*   **type filter hook input priority 0;**: Within the next pair of curly brackets, we define our chain and list the rules. This chain is defined as a `filter` type. `hook input` indicates that this chain is meant to process incoming packets. Because this chain has both a `hook` and a `priority`, it will accept packets directly from the network stack.

Finally, we have the standard rules for a very basic host firewall, starting with the **Input Interface** (**iif**) rule, which allows the loopback interface to accept packets.

Next is the standard connection tracking (`ct`) rule, which accepts traffic that's in response to a connection request from this host.

Then, there's a commented-out rule to accept Secure Shell and both secure and nonsecure web traffic. `ct state` new indicates that the firewall will allow other hosts to initiate connections to our server on these ports.

The `meta nfproto ipv6` rule accepts neighbor discovery packets, allowing IPv6 functionality.

The `counter drop` rule at the end silently blocks all other traffic and counts both the number of packets and the number of bytes that it blocks. (This is an example of how you can have one nftables rule perform multiple different actions.)

If all you need on your Ubuntu server is a basic, no-frills firewall, your best bet is to just edit this `/etc/nftables.conf` file so that it suits your own needs. For starters, let's set this up to match the setup that we created for the iptables section. In other words, let's say that this is a DNS server, and we need to allow connections to port `22` and port `53`. Remove the comment symbol from in front of the `tcp dport` line, get rid of ports `80` and `443`, and add port `53`. The line should now look like this:

```
tcp dport { 22, 53 } ct state new accept
```

Note how you can use one nftables rule to open multiple ports.

DNS also uses port `53/udp`, so let's add a line for it:

```
udp dport 53 ct state new accept
```

When you're only opening one port, you don't need to enclose that port number within curly brackets. When opening multiple ports, just include the comma-separated list within curly brackets, with a blank space after each comma, before the first element, and after the last element.

Load the configuration file and view the results:

```
donnie@ubuntu2204-packt:/etc$ sudo systemctl reload nftables
donnie@ubuntu2204-packt:/etc$ sudo nft list ruleset
table inet filter {
    chain input {
        type filter hook input priority 0; policy accept;
        iif "lo" accept
        ct state established,related accept
        tcp dport { ssh, domain } ct state new accept
        udp dport domain ct state new accept
        icmpv6 type { nd-router-advert, nd-neighbor-solicit, nd-neighbor-advert } accept
        counter packets 1 bytes 32 drop
    }
}
donnie@ubuntu2204-packt:/etc$
```

The `counter drop` rule is another example of how an nftables rule can do multiple things. In this case, the rule drops and counts unwanted packets. So far, the rule has blocked one packet and 32 bytes. To demonstrate how this works, let's say that we want to make a log entry when packets are dropped. Just add the `log` keyword to the `drop` rule, like so:

```
counter log drop
```

To make these messages easier to find, add a tag to each log message, like this:

```
counter log prefix "Dropped packet: " drop
```

Now, when you need to peruse the `/var/log/kern.log` file to see how many dropped packets you've had, just search for the `Dropped packet` text string.

Now, let's say that we want to block certain IP addresses from reaching the Secure Shell port of this machine. To do this, we can edit the file, placing a `drop` rule above the rule that opens port `22`. The relevant section of the file would look like this:

```
tcp dport 22 ip saddr { 192.168.0.7, 192.168.0.10 } log prefix "Blocked SSH packets: " drop
tcp dport { 22, 53 } ct state new accept
```

After we reload the file, we'll be blocking SSH access from two different IPv4 addresses. Any attempts to log in from either of those two addresses will create a `/var/log/kern.log` message with the `Blocked SSH packets` tag. Note that we've placed the `drop` rule ahead of the `accept` rule because if the `accept` rule gets read first, the `drop` rule won't have an effect.

Next, we need to allow the desired types of ICMP packets, like so:

```
ct state new,related,established icmp type { destination-unreachable, time-exceeded, parameter-problem } accept
ct state established,related,new icmpv6 type { destination-unreachable, time-exceeded, parameter-problem } accept
```

In this case, you need separate rules for ICMPv4 and ICMPv6.

Finally, we'll block invalid packets by adding a new prerouting chain to the filter table, like so:

```
chain prerouting {
                type filter hook prerouting priority 0;
                ct state invalid counter log prefix "Invalid Packets:  " drop
                tcp flags & (fin|syn|rst|ack) != syn ct state new counter log drop
        }
```

Now, we can save the file and close the text editor.

> Due to formatting constraints, I can't show the entire completed file here. To see the whole file, download the code file from the Packt website, and look in the `Chapter 4` directory. The example file you seek is the `nftables_example_1.conf` file.

Now, let's load up the new rules:

```
sudo systemctl reload nftables
```

Another really cool thing to note is how we've mixed IPv4 (ip) rules with IPv6 (ip6) rules in the same configuration file. Also, unless we specify otherwise, all the rules that we create will apply to both IPv4 and IPv6\. That's the beauty of using an inet-type table. For simplicity and flexibility, you'll want to use inet tables as much as possible, rather than separate tables for IPv4 and IPv6.

Most of the time, when all you need is just a simple host firewall, your best bet would be to just use this `nftables.conf` file as your starting point, and edit the file to suit your own needs. However, there's also a command-line component that you may find useful.

### Using nft commands

My preferred method of working with nftables is to just start with a template and hand-edit it to my liking, as we did in the previous section. But for those who'd rather do everything from the command line, there's the nft utility.

> Tip:
> 
> > Even if you know that you'll always create firewalls by hand-editing `nftables.conf`, there are still a couple of practical reasons to know about the nft utility.
> 
> > Let's say that you've observed an attack in progress, and you need to stop it quickly without bringing down the system. With an `nft` command, you can create a custom rule on the fly that will block the attack. Creating nftables rules on the fly also allows you to test the firewall as you configure it, before making any permanent changes.
> 
> > And, if you decide to take a Linux security certification exam, you might see questions about `nft` commands. (I happen to know.)

There are two ways to use the `nft` utility. For one, you could just do everything directly from the Bash shell, prefacing every action you want to perform with `nft`, followed by the `nft` subcommands. The other way is to use `nft` in interactive mode. For our present purposes, we'll just go with the Bash shell.

First, let's delete our previous configuration and create an `inet` table since we want something that works for both IPv4 and IPv6\. We'll want to give it a somewhat descriptive name, so let's call it `ubuntu_filter`:

```
sudo nft delete table inet filter
sudo nft list tables
sudo nft add table inet ubuntu_filter
sudo nft list tables
```

Next, we'll add an input filter chain to the table that we just created (note that since we're doing this from the Bash shell, we need to escape the semicolon with a backslash):

```
sudo nft add chain inet ubuntu_filter input { type filter hook input priority 0\; policy drop\; }
```

We could have given it a more descriptive name, but for now, `input` works. Within the pair of curly brackets, we're setting the parameters for this chain.

Each nftables protocol family has its own set of hooks, which define how packets will be processed. For now, we're only concerned with the ip/ip6/inet families, which have the following hooks:

*   Prerouting
*   Input
*   Forward
*   Output
*   Postrouting

Of these, we're only concerned with the input and output hooks, which apply to filter-type chains. By specifying a hook and a priority for our input chain, we're saying that we want this chain to be a base chain that will accept packets directly from the network stack. You will also see that certain parameters must be terminated by a semicolon, which in turn would need to be escaped with a backslash if you're running the commands from the Bash shell. Finally, we're specifying a default policy of `drop`. If we had not specified `drop` as the default policy, then the policy would have been `accept` by default.

> Tip:
> 
> > Every `nft` command that you enter takes effect immediately. So, if you're doing this remotely, you'll drop your Secure Shell connection as soon as you create a filter chain with a default `drop` policy.
> 
> > Some people like to create chains with a default `accept` policy, and then add a `drop` rule as the final rule. Other people like to create chains with a default `drop` policy, and then leave off the `drop` rule at the end. Be sure to check your local procedures to see what your organization prefers.

Verify that the chain has been added. You should see something like this:

```
donnie@ubuntu2004-packt:~$ sudo nft list table inet ubuntu_filter
 [sudo] password for donnie:
 table inet filter {
       chain input {
             type filter hook input priority 0; policy drop;
       }
 }
 donnie@ubuntu2004-packt:~$
```

That's great, but we still need some rules. Let's start with a connection tracking rule and a rule to open the Secure Shell port. Then, we'll verify that they were added:

```
sudo nft add rule inet ubuntu_filter input ct state established accept
sudo nft add rule inet ubuntu_filter input tcp dport 22 ct state new accept
sudo nft list table inet ubuntu_filter
 table inet ubuntu_filter {
     chain input {
         type filter hook input priority 0; policy drop;
         ct state established accept
         tcp dport ssh ct state new accept
     }
 }
```

Okay, that looks good. You now have a basic, working firewall that allows Secure Shell connections. Well, except that just as we did in the in the iptables section of this chapter, we forgot to create a rule to allow the loopback adapter to accept packets. Since we want this rule to be at the top of the rules list, we'll use `insert` instead of `add`:

```
sudo nft insert rule inet ubuntu_filter input iif lo accept
sudo nft list table inet ubuntu_filter
 table inet ubuntu_filter {
       chain input {
              type filter hook input priority 0; policy drop;
              iif lo accept
              ct state established accept
              tcp dport ssh ct state new accept
      }
 }
```

Now, we're all set. But what if we want to insert a rule at a specific location? For that, you'll need to use list with the `-a` option to see the rule handles:

```
sudo nft list table inet ubuntu_filter -a
 table inet ubuntu_filter {
        chain input {
                 type filter hook input priority 0; policy drop;
                 iif lo accept # handle 4
                 ct state established accept # handle 2
                 tcp dport ssh ct state new accept # handle 3
       }
 }
```

As you can see, there's no real rhyme or reason to the way the handles are numbered. Let's say that we want to insert the rule about blocking certain IP addresses from accessing the Secure Shell port. We can see that the SSH `accept` rule is `handle 3`, so we'll need to insert our `drop` rule before it. This command will look like this:

```
sudo nft insert rule inet ubuntu_filter input position 3 tcp dport 22 ip saddr { 192.168.0.7, 192.168.0.10 } drop
sudo nft list table inet ubuntu_filter -a
 table inet ubuntu_filter {
         chain input {
                  type filter hook input priority 0; policy drop;
                  iif lo accept # handle 4
                  ct state established accept # handle 2
                  tcp dport ssh ip saddr { 192.168.0.10, 192.168.0.7} drop # handle 6
                  tcp dport ssh ct state new accept # handle 3
         }
 }
```

So, to place the rule before the rule with the `handle 3` label, we have to insert it at position `3`. The new rule that we just inserted has the label `handle 6.` To delete a rule, we have to specify the rule's handle number:

```
sudo nft delete rule inet ubuntu_filter input handle 6
sudo nft list table inet ubuntu_filter -a
 table inet ubuntu_filter {
       chain input {
              type filter hook input priority 0; policy drop;
              iif lo accept # handle 4
              ct state established accept # handle 2
              tcp dport ssh ct state new accept # handle 3
      }
 }
```

As is the case with iptables, everything you do from the command line will disappear once you reboot the machine. To make it permanent, let's redirect the output of the `list` subcommand to the `nftables.conf` configuration file (of course, we'll want to have made a backup copy of the already-existing file, in case we want to revert back to it):

```
sudo sh -c "nft list table inet ubuntu_filter > /etc/nftables.conf"
```

Due to a quirk in the Bash shell, we can't just redirect output to a file in the `/etc/` directory in the normal manner, even when we use `sudo`. That's why I had to add the `sh -c` command, with the `nft list` command surrounded by double quotes. Also, note that the file has to be named `nftables.conf` because that's what the nftables systemd service looks for. Now, when we look at the file, we'll see that there are a couple of things that are missing:

```
table inet ubuntu_filter { 
    chain input { 
        type filter hook input priority 0; policy drop; 
        iif lo accept 
        ct state established accept 
        tcp dport ssh ct state new accept 
    } 
}
```

Those of you who are sharp-eyed will see that we're missing the `flush` rule and the shebang line to specify the shell that we want to interpret this script. Let's add them:

```
#!/usr/sbin/nft -f 
flush ruleset 
table inet ubuntu_filter { 
    chain input { 
         type filter hook input priority 0; policy drop; 
         iif lo accept 
         ct state established accept 
         tcp dport ssh ct state new accept 
   } 
} 
```

Much better. Let's test this by loading the new configuration and observing the `list` output:

```
sudo systemctl reload nftables
sudo nft list table inet ubuntu_filter
 table inet ubuntu_filter {
        chain input {
                 type filter hook input priority 0; policy drop;
                 iif lo accept
                 ct state established accept
                 tcp dport ssh ct state new accept
       }
 }
```

That's all there is to creating your own simple host firewall. Of course, running commands from the command line, rather than just creating a script file in your text editor, does make for a lot more typing. However, it does allow you to test your rules on the fly, as you create them. And creating your configuration in this manner and then redirecting the `list` output to your new configuration file relieves you of the burden of having to keep track of all of those curly brackets as you try to hand-edit the file.

It's also possible to take all of the `nft` commands that we just created and place them into a regular, old-fashioned Bash shell script. Trust me, though, you really don't want to do that. Just use the nft-native scripting format, as we've done here, and you'll have a script that performs better and is much more human-readable.

#### Hands-on lab for nftables on Ubuntu

For this lab, you'll need a clean snapshot of your Ubuntu 22.04 virtual machine. Let’s get started.

Restore your Ubuntu virtual machine to a clean snapshot to clear out any firewall configurations that you created previously. (Or, if you refer, start with a new virtual machine.) Disable ufw and verify that no firewall rules are present:

```
sudo systemctl disable --now ufw
sudo iptables -L
```

You should see no rules listed for nftables.

Copy the `workstation.nft` template over to the `/etc/` directory and rename it `nftables.conf`:

```
sudo cp /usr/share/doc/nftables/examples/syntax/workstation /etc/nftables.conf
```

Edit the `/etc/nftables.conf` file to create your new configuration. (Note that due to formatting constraints, I have to break this into three different code blocks.) Make the top portion of the file look like this:

```
#!/usr/sbin/nft -f flush ruleset
table inet filter {
    chain prerouting {
        type filter hook prerouting priority 0;
        ct state invalid counter log prefix "Invalid Packets:  " drop
        tcp flags & (fin|syn|rst|ack) != syn ct state new counter log prefix "Invalid Packets 2: " drop
}
```

Make the second portion of the file look like this:

```
chain input {
    type filter hook input priority 0;
    # accept any localhost traffic
    iif lo accept
    # accept traffic originated from us
    ct state established,related accept
        # activate the following line to accept common local services
        tcp dport 22 ip saddr { 192.168.0.7, 192.168.0.10 } log prefix "Blocked SSH packets: " drop

        tcp dport { 22, 53 } ct state new accept
        udp dport 53 ct state new accept
        ct state new,related,established icmp type { destination-unreachable, time-exceeded, parameter-problem } accept
```

Make the final portion of the file look like this:

```
 ct state new,related,established icmpv6 type { destination-unreachable, time-exceeded, parameter-problem } accept
        # accept neighbour discovery otherwise IPv6 connectivity breaks.
        ip6 nexthdr icmpv6 icmpv6 type { nd-neighbor-solicit,  nd-router-advert, nd-neighbor-advert } accept
# count and drop any other traffic
    counter log prefix "Dropped packet: " drop
     }
}
```

Save the file and reload nftables:

```
 sudo systemctl reload nftables
```

View the results:

```
sudo nft list tables
sudo nft list tables
sudo nft list table inet filter
sudo nft list ruleset
```

From either your host computer or from another virtual machine, do a Windows scan against the Ubuntu virtual machine:

```
sudo nmap -sW ip_address_of_UbuntuVM
```

Look at the packet counters to see which blocking rule was triggered. (Hint: It's in the prerouting chain.):

```
sudo nft list ruleset
```

This time, do a null scan of the virtual machine:

```
sudo nmap -sN ip_address_of_UbuntuVM
```

Finally, look at which rule was triggered this time. (Hint: It's the other one in the prerouting chain.):

```
sudo nft list ruleset
```

In the `/var/log/kern.log` file, search for the `Invalid Packets` text string to view the messages about the dropped invalid packets.

That's the end of this lab – congratulations!

In this section, we looked at the ins and outs of nftables, and looked at ways to configure it to help prevent certain types of attacks. In the next chapter, we’ll look at the helper programs for our helper programs.

## Summary

In this chapter, we looked at both of the helper programs that directly interface with the netfilter firewall. First, we looked at our trusty old friend, iptables. We saw that even though it's been around forever and still works, it does have some shortcomings. Then, we worked with nftables, and saw that it has certain advantages over the old iptables.

In the space that's been allotted for this chapter, I've only been able to present the essentials that you need in order to set up basic host protection. However, this should be enough to get you started.

In the next chapter, we'll look at ufw and firewalld, which are helper programs for the two helper programs that we discussed in this chapter. I’ll see you there.

## Questions

1.  Which of the following statements is true?
    1.  iptables is the easiest firewall system to work with.
    2.  With iptables, any rule that you create applies to both IPv4 and IPv6.
    3.  With iptables, you have to create IPv6 rules separately from IPv4 rules.
    4.  With nftables, you have to create IPv6 rules separately from IPv4 rules.
2.  What is the official name of the Linux firewall?
    1.  iptables
    2.  ufw
    3.  nftables
    4.  netfilter
3.  Which of the following statements about nftables is false?
    1.  With nftables, rules are added atomically.
    2.  With nftables, a table refers to a particular protocol family.
    3.  With nftables, ports and their associated rules are bundled into zones.
    4.  With nftables, you can write scripts in either normal bash shell scripting, or with the scripting engine that's built into nftables.
4.  Which iptables command would show you how many packets have been dropped by a particular rule?
5.  Which nftables command would you use to see how many packets have been dropped by a particular rule?
6.  In iptables, which of the following targets would cause packets to be blocked without sending a notification back to the source?
    1.  `STOP`
    2.  `DROP`
    3.  `REJECT`
    4.  `BLOCK`
7.  Which of the following six choices are tables in iptables?
    1.  netfilter
    2.  filter
    3.  mangle
    4.  security
    5.  ip6table
    6.  NAT
8.  Which firewall system loads its rules atomically?

## Further reading

*   25 iptables netfilter firewall examples: [https://www.cyberciti.biz/tips/linux-iptables-examples.html](https://www.cyberciti.biz/tips/linux-iptables-examples.html)
*   Linux IPv6 how-to: [http://tldp.org/HOWTO/html_single/Linux+IPv6-HOWTO/](http://tldp.org/HOWTO/html_single/Linux+IPv6-HOWTO/)
*   Recommendations for Filtering ICMPv6 Messages in Firewalls: [https://www.ietf.org/rfc/rfc4890.txt](https://www.ietf.org/rfc/rfc4890.txt)
*   nftables wiki: [https://wiki.nftables.org/wiki-nftables/index.php/Main_Page](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page)
*   nftables examples: [https://wiki.gentoo.org/wiki/Nftables/Examples](https://wiki.gentoo.org/wiki/Nftables/Examples)

## Answers

1.  C
2.  D
3.  C
4.  sudo iptables -L -v
5.  sudo nft list ruleset
6.  B
7.  B, C, D, F
8.  nftables