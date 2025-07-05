# 5 Securing Your Server with a Firewall - Part 2

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file35.png)

In *Chapter 4*, *Securing Your Server with a Firewall - Part 1*, we covered iptables and nftables, which are management utilities that directly interface with netfilter. Although it’s helpful to be familiar with iptables and nftables commands in order to create advanced firewall configurations, having to use these commands all the time can become a bit unwieldy for performing normal day-to-day operations. In this chapter, we’ll look at ufw and firewalld, which are helper utilities that can simplify the process of working with either iptables or nftables.

First, we'll look at ufw. We'll look at its structure, its commands, and its configuration. Then, we'll do the same for firewalld. In both cases, you'll get plenty of hands-on practice.

We will cover the following topics in this chapter:

*   **ufw** for Ubuntu systems
*   **firewalld** for Red Hat systems

## Technical requirements

The code files for this chapter are available here: [https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-Second-Edition.](https://github.com/PacktPublishing/Mastering-Linux-Security-and-Hardening-Second-Edition.)

## Uncomplicated firewall for Ubuntu systems

The ufw is already installed on Ubuntu 20.04 and Ubuntu 22.04\. It still uses the iptables backend on Ubuntu 20.04, and the nftables backend on Ubuntu 22.04\. For normal operations, it offers a vastly simplified set of commands. Perform just one simple command to open the desired ports and another simple command to activate it, and you have a good, basic firewall. Whenever you perform a `ufw` command, it will automatically configure both the IPv4 and the IPv6 rules. This alone is a huge time-saver, and much of what we've had to configure by hand with with either iptables or nftables is already there by default. Although our two versions of Ubuntu use different backends, ufw configuration is identical for both of them.

> Tip:
> 
> > ufw is also available for Debian and other Debian-based distros, but it might not be installed. If that's the case, install it by issuing the `sudo apt install ufw` command.

### Configuring ufw

On both Ubuntu 20.04 and Ubuntu 22.04, the ufw service is already enabled by default, but the firewall itself isn't activated. In other words, the system's service is running, but it isn't enforcing any firewall rules yet. (I'll show you how to activate it in just a bit, after we go over how to open the ports that you need to open.) Check the ufw status with these two commands:

```
systemctl status ufw
sudo ufw status
```

The systemctl command should show you that the service is enabled, and the ufw command should show you that the firewall is inactive.

The first thing we want to do is open port `22` to allow it to connect to the machine via Secure Shell, like so:

```
sudo ufw allow 22/tcp
```

Okay, that looks good. Let’s now activate the firewall, like this:

```
sudo ufw enable
```

By using `sudo iptables -L` on Ubuntu 20.04, you'll see that the new Secure Shell rule shows up in the `ufw-user-input` chain:

```
Chain ufw-user-input (1 references)
 target prot opt source destination
 ACCEPT tcp -- anywhere anywhere tcp dpt:ssh
```

On Ubuntu 22.04, use the `sudo nft list ruleset` command to see the new rule in the `ufw-user-input` chain:

```
chain ufw-user-input {
        meta l4proto tcp tcp dport 22 counter packets 0 bytes 0 accept
    }
```

You'll also see that the total output of both of these commands is quite lengthy because so much of what we had to do with bare iptables or nftables has already been done for us with ufw. In fact, there's even more here than what we did with iptables and nftables. For example, with ufw, we already have rate limiting rules that help protect us against **Denial-of-Service** (**DoS**) attacks, and we also have rules that record log messages about packets that have been blocked. It's almost the no fuss, no muss way of setting up a firewall. (I'll get to that *almost* part in a bit.)

In the preceding `sudo ufw allow 22/tcp` command, we had to specify the TCP protocol because TCP is all we need for Secure Shell. We can also open a port for both TCP and UDP just by not specifying a protocol. For example, if you're setting up a DNS server, you'll want to have port `53` open for both protocols. (You'll see the entries for port `53` listed as domain ports). On either version of Ubuntu, do:

```
 sudo ufw allow 53
```

On Ubuntu 20.04, view the results by doing:

```
 sudo iptables -L
. . .
. . .
    Chain ufw-user-input (1 references)
    target prot opt source destination
    ACCEPT tcp -- anywhere anywhere tcp dpt:ssh
    ACCEPT tcp -- anywhere anywhere tcp dpt:domain
    ACCEPT udp -- anywhere anywhere udp dpt:domain
```

On Ubuntu 22.04, view the results by doing:

```
sudo nft list ruleset
chain ufw-user-input {
        meta l4proto tcp tcp dport 22 counter packets 0 bytes 0 accept
        meta l4proto tcp tcp dport 53 counter packets 0 bytes 0 accept
        meta l4proto udp udp dport 53 counter packets 0 bytes 0 accept
    }
```

If you do `sudo ip6tables -L` on the 20.04 machine,you'll see that a rule for IPv6 was also added for both of the two preceding examples. And, again, you'll see that most of what we had to do with the ip6tables commands has already been taken care of. (It's especially nice that we don't have to mess around with setting up all of those pesky ICMP rules.) On the 22.04 machine, the `sudo nft list ruleset` command that you did previously will show the IPv6 configuration in the `ufw6-user-input` stanza.

To see just a quick summary of your firewall configuration, use the `status` option. The output should look something like this:

```
donnie@ubuntu-ufw:~$ sudo ufw status
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     LIMIT       Anywhere                  
53                         LIMIT       Anywhere                 
22/tcp (v6)                LIMIT       Anywhere (v6)             
53 (v6)                    LIMIT       Anywhere (v6)             

donnie@ubuntu-ufw:~$
```

Next, we will look at the ufw configuration files.

### Working with the ufw configuration files

You can find the ufw firewall rules in the `/etc/ufw/` directory. As you can see, the rules are stored in several different files:

```
donnie@ubuntu-ufw:/etc/ufw$ ls -l
total 48
-rw-r----- 1 root root  915 Aug  7 15:23 after6.rules
-rw-r----- 1 root root 1126 Jul 31 14:31 after.init
-rw-r----- 1 root root 1004 Aug  7 15:23 after.rules
drwxr-xr-x 3 root root 4096 Aug  7 16:45 applications.d
-rw-r----- 1 root root 6700 Mar 25 17:14 before6.rules
-rw-r----- 1 root root 1130 Jul 31 14:31 before.init
-rw-r----- 1 root root 3467 Aug 11 11:36 before.rules
-rw-r--r-- 1 root root 1391 Aug 15  2017 sysctl.conf
-rw-r--r-- 1 root root  313 Aug 11 11:37 ufw.conf
-rw-r----- 1 root root 3014 Aug 11 11:37 user6.rules
-rw-r----- 1 root root 3012 Aug 11 11:37 user.rules
donnie@ubuntu-ufw:/etc/ufw$
```

At the bottom of the list, you'll see the `user6.rules` and `user.rules` files. You can't hand-edit either of these two files. You'll be able to save the files after you've made the edits, but when you use `sudo ufw reload` to load the new changes, you'll see that your edits have been deleted. Let's look into the `user.rules` file to see what we can see there.

> Tip:
> 
> > As you’ll soon see, all of the files for both Ubuntu 20.04 and 22.04 contain firewall rules that are in the iptables format, even though 22.04 uses nftables as its backend. That’s because Ubuntu 22.04 can automatically translate iptables rules into nftables rules. So, the files for both 20.04 and 22.04 are identical, which makes things very easy for us.

At the top of the file, you'll see the definition for the iptables filter table, as well as the list of its associated chains:

```
*filter
:ufw-user-input - [0:0]
:ufw-user-output - [0:0]
:ufw-user-forward - [0:0]
. . .
. . .
```

Next, in the `### RULES ###` section, we have the list of rules that we created with the `ufw` command. Here's what our rules for opening the DNS ports look like:

```
### tuple ### allow any 53 0.0.0.0/0 any 0.0.0.0/0 in
-A ufw-user-input -p tcp --dport 53 -j ACCEPT
-A ufw-user-input -p udp --dport 53 -j ACCEPT
```

As you can see, ufw use iptables syntax for its configuration files, even on Ubuntu 22.04.

Below the `### RULES ###` section, we see the rules for logging messages about any packets that the firewall has blocked:

```
### LOGGING ###
-A ufw-after-logging-input -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-A ufw-after-logging-forward -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-I ufw-logging-deny -m conntrack --ctstate INVALID -j RETURN -m limit --limit 3/min --limit-burst 10
-A ufw-logging-deny -j LOG --log-prefix "[UFW BLOCK] " -m limit --limit 3/min --limit-burst 10
-A ufw-logging-allow -j LOG --log-prefix "[UFW ALLOW] " -m limit --limit 3/min --limit-burst 10
### END LOGGING ###
```

These messages get sent to the `/var/log/kern.log` file. So that we don't overwhelm the logging system when lots of packets are getting blocked, we'll only send three messages per minute to the log file, with a burst rate of 10 messages per minute. Most of these rules will insert a `[UFW BLOCK]` tag in with the log message, which makes it easy for us to find them. The last rule creates messages with a `[UFW ALLOW]` tag, and curiously enough, the `INVALID` rule doesn't insert any kind of tag.

Lastly, we have the rate-limiting rules, which allow only three connections per user, per minute:

```
### RATE LIMITING ###
-A ufw-user-limit -m limit --limit 3/minute -j LOG --log-prefix "[UFW LIMIT BLOCK] "
-A ufw-user-limit -j REJECT
-A ufw-user-limit-accept -j ACCEPT
### END RATE LIMITING ###
```

Any packets that exceed that limit will be recorded in the `/var/log/kern.log` file with the `[UFW LIMIT BLOCK]` tag.

The `/etc/ufw user6.rules` file looks pretty much the same, except that it's for IPv6 rules. Any time you create or delete a rule with the `ufw` command, it will modify both the `user.rules` file and the `user6.rules` file.

To store rules that will run before the rules in the `user.rules` and `user6.rules` files, we have the `before.rules` file and the `before6.rules` file. To store rules that will run after the rules in the `user.rules` and `user6.rules` files, we have – you guessed it – the `after.rules` file and the `after6.rules` file. If you need to add custom rules that you can't add with the `ufw` command, just hand-edit one of these pairs of files. (We'll get to that in a moment.)

If you look at the `before` and `after` files, you'll see where so much has already been taken care of for us. This is all the stuff that we had to do by hand with either iptables/ip6tables or nftables.

However, as you might know, there is one slight caveat to all this ufw goodness. You can perform simple tasks with the ufw utility, but anything more complex requires you to hand-edit a file. (This is what I meant when I said that ufw is *almost* no fuss, no muss.)

> Tip:
> 
> > To see more examples of what you can do with the ufw command, view its man page by doing:

```
man ufw
```

For example, in the `before` files, you'll see that one of the rules for blocking invalid packets has already been implemented. Here's the code snippet from the `before.rules` file, which you'll find near the top of the file:

```
# drop INVALID packets (logs these in loglevel medium and higher)
-A ufw-before-input -m conntrack --ctstate INVALID -j ufw-logging-deny
-A ufw-before-input -m conntrack --ctstate INVALID -j DROP
```

The second of these two rules actually drops the invalid packets, and the first rule logs them. But as we've already seen in the *An overview of iptables* section of *Chapter 4*, *Securing your server with a firewall-Part 1*, this one particular `DROP` rule doesn't block all of the invalid packets. And, for performance reasons, we'd rather have this rule in the mangle table, instead of in the filter table where it is now. To fix that, we'll edit both of the `before` files. Open the `/etc/ufw/before.rules` file in your favorite text editor and look for the following pair of lines at the very bottom of the file:

```
# don't delete the 'COMMIT' line or these rules won't be processed
COMMIT
```

Just below the `COMMIT` line, add the following code snippet to create the mangle table rules:

```
# Mangle table added by Donnie
*mangle
:PREROUTING ACCEPT [0:0]
-A PREROUTING -m conntrack --ctstate INVALID -j DROP
-A PREROUTING -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j DROP
COMMIT
```

Now, we'll repeat this process for the `/etc/ufw/before6.rules` file. Then, we'll reload the rules by doing:

```
sudo ufw reload
```

By using the `iptables -L` and `ip6tables -L` commands on Ubuntu 20.04 or the `nft list ruleset` command on Ubuntu 22.04, you'll see the new rules show up in the mangle table, just where we want them to be.

#### Hands-on lab for basic ufw usage

You'll need to complete this lab on a clean snapshot of either an Ubuntu 20.04 or an Ubuntu 22.04 virtual machine. Let's get started:

1.  Shut down your Ubuntu virtual machine and restore the snapshot to get rid of all of the iptables or nftables stuff that you just did. (Or, if you prefer, just start with a fresh virtual machine.)
2.  When you've restarted the virtual machine, verify that the iptables rules are now gone. On Ubuntu 20.04 do:

```
sudo iptables -L
On Ubuntu 22.04 do:
sudo nft list ruleset
```

1.  View the status of `ufw`. Open port `22/TCP` and then enable `ufw`. Then, view the results:

```
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status
On Ubuntu 20.04 do:
sudo iptables -L
sudo ip6tables -L
On Ubuntu 22.04 do:
sudo nft list ruleset
```

1.  This time, open port `53` for both TCP and UDP:

```
sudo ufw allow 53
sudo ufw status
On Ubuntu 20.04 do:
sudo iptables -L
sudo ip6tables -L
On Ubuntu 22.04 do:
sudo nft list ruleset
```

1.  `cd` into the `/etc/ufw/` directory. Familiarize yourself with the contents of the files that are there.
2.  Open the `/etc/ufw/before.rules` file in your favorite text editor. At the bottom of the file, below the `COMMIT` line, add the following code snippet:

```
# Mangle table added by Donnie
*mangle
:PREROUTING ACCEPT [0:0]
-A PREROUTING -m conntrack --ctstate INVALID -j DROP
-A PREROUTING -p tcp -m tcp ! --tcp-flags FIN,SYN,RST,ACK SYN -m conntrack --ctstate NEW -j DROP
COMMIT
(Note that the second PREROUTING command wraps around on the printed page.)
```

1.  Repeat s*tep 6* for the `/etc/ufw/before6.rules` file.
2.  Reload the firewall with this command:

```
sudo ufw reload
```

1.  On Ubuntu 20.04, observe the rules by doing:

```
sudo iptables -L
sudo iptables -t mangle -L
sudo ip6tables -L
sudo ip6tables -t mangle -L
On Ubuntu 22.04, observe the rules by doing:
sudo nft list ruleset
```

1.  Take a quick look at the `ufw` status:

```
sudo ufw status
```

That's the end of the lab – congratulations!

I think you’ll agree that `ufw` is pretty cool technology. Its commands for doing basic things are easier to remember than the equivalent iptables or nftables commands, and it takes care of both IPv4 and IPv6 with just a single command. On either of our Ubuntu versions, you can still do some fancy stuff just by hand-editing the ufw configuration files. But, ufw isn’t the only cool firewall manager that’s available. In the next section, we’ll take a look at what the Red Hat folk have given us.

## firewalld for Red Hat systems

For our next act, we turn our attention to **firewalld**, which is the default firewall manager on Red Hat Enterprise Linux 7 through 9 and all of their offspring.

As we just saw with ufw on Ubuntu, firewalld can be a frontend for either iptables or nftables. On RHEL/CentOS 7, firewalld uses the iptables engine as its backend. On the RHEL 8 and 9-type distros, firewalld uses nftables as its backend. Either way, you can't create rules with normal iptables or nftables commands while firewalld is enabled because firewalld stores the rules in an incompatible format.

> Tip:
> 
> > Until very recently, firewalld was only available for the newer RHEL versions and their offspring. Now, however, firewalld is also available in the Ubuntu repositories. So, if you want to run firewalld on Ubuntu, you finally have that choice. Also, the combination of firewalld and nftables now comes already installed and activated on the SUSE distros.

If you're running Red Hat, CentOS, or AlmaLinux on a desktop machine, you'll see that there is a GUI frontend for firewalld in the applications menu. On a text-mode server, though, all you have is the firewalld commands. For some reason, the Red Hat folk haven't created an ncurses-type program for text-mode servers as they did for iptables configuration on older versions of Red Hat.

A big advantage of firewalld is the fact that it's dynamically managed. That means that you can change the firewall configuration without restarting the firewall service, and without interrupting any existing connections to your server.

Before we look at the differences between the RHEL 7/CentOS 7 and the RHEL/AlmaLinux 8 and 9 versions of firewalld, let's look at the stuff that's the same for both.

### Verifying the status of firewalld

For this section, you can use a CentOS 7, AlmaLinux 8, or AlmaLinux 9 virtual machine. Let's start by verifying the status of firewalld. There are two ways to do this. The first way is to use the `--state` option of `firewall-cmd`:

```
[donnie@localhost ~]$ sudo firewall-cmd --state
 running
 [donnie@localhost ~]$
```

Alternatively, if we want a more detailed status, we can just check the daemon, the same as we would for any other daemon on a systemd machine:

```
[donnie@localhost ~]$ sudo systemctl status firewalld
 firewalld.service - firewalld - dynamic firewall daemon
  Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled;
 vendor preset: enabled)
  Active: active (running) since Fri 2017-10-13 13:42:54 EDT; 1h 56min ago
  Docs: man:firewalld(1)
  Main PID: 631 (firewalld)
  CGroup: /system.slice/firewalld.service
  └─631 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
. . .
 Oct 13 15:19:41 localhost.localdomain firewalld[631]: WARNING: reject-
 route: INVALID_ICMPTYPE: No supported ICMP type., ignoring for run-time.
 [donnie@localhost ~]$
```

Next, let's have a look at firewalld zones.

### Working with firewalld zones

`firewalld` is a rather unique animal, in that it comes with several pre-configured zones and services. If you look in the `/usr/lib/firewalld/zones/` directory of any of your CentOS or AlmaLinux machines, you'll see the zones files, all in `.xml` format:

```
[donnie@localhost ~]$ cd /usr/lib/firewalld/zones
[donnie@localhost zones]$ ls
block.xml dmz.xml drop.xml external.xml home.xml internal.xml public.xml
trusted.xml work.xml
[donnie@localhost zones]$
```

Each zone file specifies which ports are to be open and which ones are to be blocked for various given scenarios. Zones can also contain rules for ICMP messages, forwarded ports, masquerading information, and rich language rules. For example, the `.xml` file for the public zone, which is set as the default, looks like this:

```
<?xml version="1.0" encoding="utf-8"?> 
<zone> 
 <short>Public</short> 
 <description>For use in public areas. You do not trust the other 
computers on networks to not harm your computer. Only selected incoming 
connections are accepted.</description> 
 <service name="ssh"/> 
 <service name="dhcpv6-client"/> 
</zone> 
```

In the `service name` lines, you can see that the only open ports are for Secure Shell access and for DHCPv6 discovery. If you look at the `home.xml` file, you'll see that it also opens the ports for Multicast DNS, as well as the ports that allow this machine to access shared directories from either Samba servers or Windows servers:

```
<?xml version="1.0" encoding="utf-8"?> 
<zone> 
 <short>Home</short> 
 <description>For use in home areas. You mostly trust the other computers 
on networks to not harm your computer. Only selected incoming connections 
are accepted.</description> 
 <service name="ssh"/>
 <service name="mdns"/> 
 <service name="samba-client"/> 
 <service name="dhcpv6-client"/> 
</zone>
```

The `firewall-cmd` utility is what you would use to configure `firewalld.` You can use it to view the list of zone files on your system, without having to `cd` into the zone file directory:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-zones
[sudo] password for donnie:
block dmz drop external home internal public trusted work
[donnie@localhost ~]$
```

A quick way to see how each zone is configured is to use the `--list-all-zones` option:

```
[donnie@localhost ~]$ sudo firewall-cmd --list-all-zones
 block
  target: %%REJECT%%
  icmp-block-inversion: no
  interfaces:
  sources:
  services:
  ports:
  protocols:
  masquerade: no
  forward-ports:
. . .
. . .
```

Of course, this is only a portion of the output because the listing for all zones is more than we can display here. It's more likely that you'll only want to see information about one particular zone:

```
 [donnie@localhost ~]$ sudo firewall-cmd --info-zone=internal
 internal
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh mdns samba-client dhcpv6-client
  ports:
  protocols:
  masquerade: no
  forward-ports:
 source-ports:
  icmp-blocks:
  rich rules:
 [donnie@localhost ~]$
```

So, the `internal` zone allows the `ssh`, `mdns`, `samba-client`, and `dhcpv6-client` services. This is handy for setting up client machines on your internal LAN.

Any given server or client will have one or more installed network interface adapters. Each adapter in a machine can be assigned one, and only one, firewalld zone. To see the default zone, do this:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-default-zone
 public
[donnie@localhost ~]$
```

This is great, except that it doesn't tell you anything about which network interface is associated with this zone. To see that information, do this:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-active-zones
 public
  interfaces: enp0s3
[donnie@localhost ~]$
```

When you install Red Hat, CentOS, or AlmaLinux for the first time, the firewall will already be active with the public zone as the default. Now, let's say that you're setting up your server in the DMZ and you want to make sure that its firewall is locked down for that. You can change the default zone to the `dmz` zone. Let's take a look at the `dmz.xml` file to see what that does for us:

```
<?xml version="1.0" encoding="utf-8"?> 
<zone> 
 <short>DMZ</short> 
 <description>For computers in your demilitarized zone that are publicly- 
accessible with limited access to your internal network. Only selected 
incoming connections are accepted.</description> 
 <service name="ssh"/> 
</zone> 
```

So, the only thing that the DMZ allows through is Secure Shell. Okay; that's good enough for now, so let's set the `dmz` zone as the default:

```
[donnie@localhost ~]$ sudo firewall-cmd --set-default-zone=dmz
 [sudo] password for donnie:
 success
[donnie@localhost ~]$
```

Let's verify it:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-default-zone
 dmz
[donnie@localhost ~]$
```

And we're all good. However, an Internet-facing server in the DMZ probably needs to allow more than just SSH connections. This is where we'll use the firewalld services. But before we look at that, let's consider one more important point.

> Tip:
> 
> > You don’t need to use the `--permanent` option when setting the default zone. In fact, you’ll get an error message if you do.

You never want to modify the files in the `/usr/lib/firewalld/` directory. Whenever you modify the firewalld configuration, you'll see the modified files show up in the `/etc/firewalld/` directory. So far, all we've modified is the default zone. So, we'll see the following files in `/etc/firewalld/`:

```
[donnie@localhost ~]$ sudo ls -l /etc/firewalld
 total 12
 -rw-------. 1 root root 2003 Oct 11 17:37 firewalld.conf
 -rw-r--r--. 1 root root 2006 Aug 4 17:14 firewalld.conf.old
 . . .
```

We can do a `diff` on those two files to see the difference between them:

```
[donnie@localhost ~]$ sudo diff /etc/firewalld/firewalld.conf /etc/firewalld/firewalld.conf.old
 6c6
 < DefaultZone=dmz
 ---
 > DefaultZone=public
 [donnie@localhost ~]$
```

So, the newer of the two files shows that the dmz zone is now the default.

> Tip:
> 
> > To find out more about firewalld zones, enter the `man firewalld.zones` command.

### Adding services to a firewalld zone

Each service file contains a list of ports that need to be opened for a particular service. Optionally, the service files may contain one or more destination addresses, or call in any needed modules, such as for connection tracking. For some services, all you need to do is open just one port. Other services, such as the Samba service, require that multiple ports be opened. Either way, it's sometimes handier to remember the service name that goes with each service, rather than the port numbers.

The services files are in the `/usr/lib/firewalld/services/` directory. You can look at them by using the `firewall-cmd` command, just as you could with the list of zones:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-services
 RH-Satellite-6 amanda-client amanda-k5-client bacula bacula-client bitcoin bitcoin-rpc bitcoin-testnet bitcoin-testnet-rpc ceph ceph-mon cfengine condor-collector ctdb dhcp dhcpv6 dhcpv6-client dns docker-registry dropbox-lansync elasticsearch freeipa-ldap freeipa-ldaps freeipa-replication freeipa-trust ftp ganglia-client ganglia-master high-availability http https imap imaps ipp ipp-client ipsec iscsi-target kadmin kerberos kibana klogin kpasswd kshell ldap ldaps libvirt libvirt-tls managesieve mdns mosh mountd ms-wbt mssql mysql nfs nrpe ntp openvpn ovirt-imageio ovirt-storageconsole ovirt-vmconsole pmcd pmproxy pmwebapi pmwebapis pop3 pop3s postgresql privoxy proxy-dhcp ptp pulseaudio puppetmaster quassel radius rpc-bind rsh rsyncd samba samba-client sane sip sips smtp smtp-submission smtps snmp snmptrap spideroak-lansync squid ssh synergy syslog syslog-tls telnet tftp tftp-client tinc tor-socks transmission-client vdsm vnc-server wbem-https xmpp-bosh xmpp-client xmpp-local xmpp-server
[donnie@localhost ~]$
```

Before we add any more services, let's check which ones are already enabled:

```
[donnie@localhost ~]$ sudo firewall-cmd --list-services
[sudo] password for donnie: 
ssh dhcpv6-client
[donnie@localhost ~]$
```

Here, ssh and dhcpv6-client are all we have.

The `dropbox-lansync` service would be very handy for us Dropbox users. Let's see which ports this opens:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-service=dropbox-lansync
 [sudo] password for donnie:
 dropbox-lansync
  ports: 17500/udp 17500/tcp
  protocols:
  source-ports:
  modules:
  destination:
[donnie@localhost ~]$
```

It looks like Dropbox uses port `17500` on UDP and TCP.

Now, let's say that we have our web server set up in the DMZ, with the `dmz` zone set as its default:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

As we saw previously, the Secure Shell port is the only one that's open. Let's fix that so that users can actually access our website:

```
[donnie@localhost ~]$ sudo firewall-cmd --add-service=http
 success
[donnie@localhost ~]$
```

When we look at the information for the `dmz` zone once more, we'll see the following:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh http
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

Here, we can see that the `http` service is now allowed through. But look what happens when we add the `--permanent` option to this `info` command:

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --info-zone=dmz
 dmz
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

Oops! The `http` service isn't here. What's going on?

For pretty much every command-line alteration of either zones or services, you need to add the `--permanent` option to make the change persistent across reboots. But without the `--permanent` option, the change takes effect immediately. With the `--permanent` option, you'll have to reload the firewall configuration for the change to take effect. To demonstrate this, I'm going to reboot the virtual machine to get rid of the `http` service.

Okay, I've rebooted, and the `http` service is now gone:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

This time, I'll add two services with just one command and specify that the change will be permanent:

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --add-service={http,https}
 [sudo] password for donnie:
 success
[donnie@localhost ~]$
```

You can add as many services as you need to with a single command, but you have to separate them with commas and enclose the whole list within a pair of curly brackets. Also, unlike what we just saw with nftables, we can't have blank spaces within the curly brackets. Let's look at the results:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=dmz
 dmz (active)
 target: default
 icmp-block-inversion: no
 interfaces: enp0s3
 sources:
 services: ssh
 ports:
 protocols:
 masquerade: no
 forward-ports:
 source-ports:
 icmp-blocks:
 rich rules:
[donnie@localhost ~]$
```

Since we decided to make this configuration permanent, it hasn't taken effect yet. However, if we add the `--permanent` option to the `--info-zone` command, we'll see that the configuration files have indeed been changed:

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --info-zone=dmz
 dmz
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: ssh http https
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
[donnie@localhost ~]$
```

Now, we need to reload the configuration so that it will take effect:

```
[donnie@localhost ~]$ sudo firewall-cmd --reload
 success
[donnie@localhost ~]$
```

Now, if you run the `sudo firewall-cmd --info-zone=dmz` command again, you'll see that the new configuration is in effect.

To remove a service from a zone, just replace `--add-service` with `--remove-service`.

> Tip:
> 
> > Note that we never specified which zone we're working with in any of these service commands. That's because if we don't specify a zone, firewalld just assumes that we're working with the default zone. If you want to add a service to something other than the default zone, just add the `--zone=` option to your commands.

### Adding ports to a firewalld zone

Having the service files is handy, except that not every service that you'll need to run has its own predefined service file. Let's say that you've installed Webmin on your server, which requires port `10000/tcp` to be open. A quick grep operation will show that port `10000` isn't in any of our predefined services:

```
donnie@localhost services]$ pwd
 /usr/lib/firewalld/services
[donnie@localhost services]$ grep '10000' *
[donnie@localhost services]$
```

So, let's just add that port to our default zone, which is still the `dmz` zone:

```
donnie@localhost ~]$ sudo firewall-cmd --add-port=10000/tcp
 [sudo] password for donnie:
 success
[donnie@localhost ~]$
```

Again, this isn't permanent, because we didn't include the `--permanent` option. Let's do this again and reload:

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --add-port=10000/tcp
 success
[donnie@localhost ~]$ sudo firewall-cmd --reload
 success
[donnie@localhost ~]$
```

You can also add multiple ports at once by enclosing the comma-separated list within a pair of curly brackets, just as we did with the services. (I purposely left out the `--permanent` option. You'll see why in a moment):

```
[donnie@localhost ~]$ sudo firewall-cmd --add-port={636/tcp,637/tcp,638/udp}
 success
[donnie@localhost ~]$
```

And of course, you can remove ports from a zone by substituting `--remove-port` for `--add-port`.

If you don't want to type `--permanent` every time you create a new permanent rule, just leave it out. Then, when you're done creating rules, make them all permanent at once by typing:

```
sudo firewall-cmd --runtime-to-permanent
```

Now's, let's turn our attention to controlling ICMP.

### Blocking ICMP

Let's take another look at the status of the default public zone:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: ssh dhcpv6-client
  ports: 53/tcp 53/udp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[donnie@localhost ~]$
```

Toward the bottom, we can see the `icmp-block` line, with nothing beside it. This means that our public zone allows all ICMP packets to come through. This isn't ideal, of course, because there are certain types of ICMP packets that we want to block. Before we block anything, let's look at all of the ICMP types that are available to us:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-icmptypes
[sudo] password for donnie: 
address-unreachable bad-header communication-prohibited destination-unreachable echo-reply echo-request fragmentation-needed host-precedence-violation host-prohibited host-redirect host-unknown host-unreachable ip-header-bad neighbour-advertisement neighbour-solicitation network-prohibited network-redirect network-unknown network-unreachable no-route packet-too-big parameter-problem port-unreachable precedence-cutoff protocol-unreachable redirect required-option-missing router-advertisement router-solicitation source-quench source-route-failed time-exceeded timestamp-reply timestamp-request tos-host-redirect tos-host-unreachable tos-network-redirect tos-network-unreachable ttl-zero-during-reassembly ttl-zero-during-transit unknown-header-type unknown-option
[donnie@localhost ~]$
```

As we did with zones and services, we can view information about the different ICMP types. In this example, we'll look at one ICMPv4 type and one ICMPv6 type:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-icmptype=network-redirectnetwork-redirect  destination: ipv4
[donnie@localhost ~]$ sudo firewall-cmd --info-icmptype=neighbour-advertisementneighbour-advertisement  
destination: ipv6
[donnie@localhost ~]$
```

We've already seen that we're not blocking any ICMP packets. We can also see if we're blocking any specific ICMP packets:

```
[donnie@localhost ~]$ sudo firewall-cmd --query-icmp-block=host-redirect
no
[donnie@localhost ~]$
```

We've already established that redirects can be a bad thing since they can be exploited. So, let's block host-redirect packets:

```
[donnie@localhost ~]$ sudo firewall-cmd --add-icmp-block=host-redirect
success
[donnie@localhost ~]$ sudo firewall-cmd --query-icmp-block=host-redirect
yes
[donnie@localhost ~]$
```

Now, let's check the status:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: ssh dhcpv6-client
  ports: 53/tcp 53/udp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: host-redirect
  rich rules: 
[donnie@localhost ~]$
```

Cool – it worked. Now, let's see if we can block two ICMP types with just one command:

```
[donnie@localhost ~]$ sudo firewall-cmd --add-icmp-block={host-redirect,network-redirect}
success
[donnie@localhost ~]$ 
```

As before, we'll check the status:

```
[donnie@localhost ~]$ sudo firewall-cmd --info-zone=public
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: host-redirect network-redirect
  rich rules: 
[donnie@localhost ~]$
```

This also worked, which means that we have achieved coolness. However, since we didn't include `--permanent` with these commands, these ICMP types will only be blocked until we reboot the computer. So, let's make them permanent:

```
[donnie@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
success
[donnie@localhost ~]$
```

And with this, we've achieved even more coolness. (Of course, all of my cats already think that I'm pretty cool.)

### Using panic mode

You've just seen evidence that bad people are trying to break into your system. What do you do? Well, one option is to activate `panic` mode, which cuts off all network communications.

> I can just see this now in the Saturday morning cartoons when some cartoon character yells, *Panic mode, activate!*

To activate `panic` mode, use this command:

```
[donnie@localhost ~]$ sudo firewall-cmd --panic-on
[sudo] password for donnie: 
success
[donnie@localhost ~]$
```

Of course, your access will be cut off if you're logged in remotely, and you'll have to go to the local terminal to get back in. To turn `panic` mode off, use this command:

```
[donnie@localhost ~]$ sudo firewall-cmd --panic-off
[sudo] password for donnie: 
success
[donnie@localhost ~]$
```

If you're logged in remotely, there's no need to check the status of `panic` mode. If it's on, you're not accessing the machine. But if you're sitting at the local console, you might want to check it. Just do:

```
[donnie@localhost ~]$ sudo firewall-cmd --query-panic
[sudo] password for donnie: 
no
[donnie@localhost ~]$
```

That's all there is to `panic` mode.

### Logging dropped packets

Here's another time-saver that you're sure to like. If you want to create log entries whenever packets get blocked, just use the `--set-log-denied` option. Before we do that, let's see if it's already enabled:

```
[donnie@localhost ~]$ sudo firewall-cmd --get-log-denied
[sudo] password for donnie: 
off
[donnie@localhost ~]$
```

It's not, so let's turn it on and check the status again:

```
[donnie@localhost ~]$ sudo firewall-cmd --set-log-denied=all
success
[donnie@localhost ~]$ sudo firewall-cmd --get-log-denied
all
[donnie@localhost ~]$
```

We've set it up to log all denied packets. However, you might not always want that. Your other choices are `unicast`, `broadcast`, and `multicast`.

So, for example, if all you want is to log blocked packets that are going to multicast addresses, do this:

```
[donnie@localhost ~]$ sudo firewall-cmd --set-log-denied=multicast
[sudo] password for donnie: 
success
[donnie@localhost ~]$ sudo firewall-cmd --get-log-denied
multicast
[donnie@localhost ~]$
```

So far, we've just set the runtime configuration, which will disappear once we reboot the machine. To make this permanent, we can use any of the methods that we've already used. For now, let's just do this:

```
[donnie@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
success
[donnie@localhost ~]$
```

Unlike what we saw with the Debian/Ubuntu distros, there's no separate `kern.log` file for our packet-denied messages. Instead, the RHEL-type distros log the packet-denied messages in the `/var/log/messages` file, which is the main log file in the RHEL world. Several different message tags are already defined, which will make it easier to audit the logs for dropped packets. For example, here's a message that tells us about blocked broadcast packets:

```
Aug 20 14:57:21 localhost kernel: FINAL_REJECT: IN=enp0s3 OUT= MAC=ff:ff:ff:ff:ff:ff:00:1f:29:02:0d:5f:08:00 SRC=192.168.0.225 DST=255.255.255.255 LEN=140 TOS=0x00 PREC=0x00
 TTL=64 ID=62867 DF PROTO=UDP SPT=21327 DPT=21327 LEN=120
```

The tag is `FINAL_REJECT`, which tells us that this message was created by the catch-all, final `REJECT` rule that's at the end of our input chain. The `DST=255.255.255.255` part tells us that this was a broadcast message.

Here's another example, where I did an Nmap NULL scan against this machine:

```
sudo nmap -sN 192.168.0.8
Aug 20 15:06:15 localhost kernel: STATE_INVALID_DROP: IN=enp0s3 OUT= MAC=08:00:27:10:66:1c:00:1f:29:02:0d:5f:08:00 SRC=192.168.0.225 DST=192.168.0.8 LEN=40 TOS=0x00 PREC=0x00 TTL=42 ID=27451 PROTO=TCP SPT=46294 DPT=23 WINDOW=1024 RES=0x00 URGP=0
```

In this case, I triggered the rule that blocks `INVALID` packets, as indicated by the `STATE_INVALID_DROP` tag.

So, now you're saying, *But wait. These two rules that we just tested aren't anywhere to be found in the firewalld configuration files that we've looked at so far. What gives?* And you're right. The location of these default, pre-configured rules is something that the Red Hat folk apparently want to keep hidden from us. However, in the following sections that are specific to RHEL/CentOS 7 and RHEL/AlmaLinux 8 and 9, we'll spoil their fun, because I can show you where these rules are.

### Using firewalld rich language rules

What we've looked at so far might be all you'll ever need for general use scenarios, but for more granular control, you'll want to know about **rich language rules**. (Yes, that really is what they're called.)

Compared to iptables rules, rich language rules are a bit less cryptic and are closer to plain English. So, if you're new to the business of writing firewall rules, you might find rich language a bit easier to learn. On the other hand, if you're already used to writing iptables rules, you might find some elements of the rich language a bit quirky. Let's look at one example:

```
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="200.192.0.0/24" service name="http" drop'
```

Here, we're adding a rich rule that blocks website access from an entire geographic block of IPv4 addresses. Note that the entire rule is surrounded by a pair of single quotes, and the assigned value for each parameter is surrounded by a pair of double quotes. With this rule, we're saying that we're working with IPv4 and that we want to silently block the `http` port from accepting packets from the `200.192.0.0/24` network. I used the `--permanent` option here, because AlmaLinux 9 is a bit quirky if I don’t use it. Let's see what our zone looks like with this new rule:

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --info-zone=dmz
  dmz (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3
  sources:
  services: ssh http https
  ports: 10000/tcp 636/tcp 637/tcp 638/udp
. . .
. . .
  rich rules:
  rule family="ipv4" source address="200.192.0.0/24" service name="http"
 drop
 [donnie@localhost ~]$
```

The rich rule shows up at the bottom.

You could just as easily write a rule for IPv6 by replacing `family="ipv4"` with `family="ipv6"` and supplying the appropriate IPv6 address range.

Some rules are generic and apply to either IPv4 or IPv6\. Let's say that we want to log messages about **Network Time Protocol** (**NTP**) packets for both IPv4 and IPv6 and that we want to log no more than one message per minute. The command to create that rule would look like this:

```
sudo firewall-cmd --add-rich-rule='rule service name="ntp" audit limit value="1/m" accept'
```

There is, of course, a lot more to firewalld rich language rules than we can present here. But for now, you know the basics. For more information, consult the man page:

```
man firewalld.richlanguage
```

> If you go to the official documentation page for Red Hat Enterprise Linux 8, you'll see no mention of rich rules. However, I've just tested them on an RHEL 8-type machine and a RHEL 9-type machine, and they work fine.
> 
> > To read about rich rules, you'll need to go to the documentation page for Red Hat Enterprise Linux 7\. What's there also applies to RHEL 8/9\. But even there, there's not much detail. To find out more, see the man page on either RHEL/CentOS 7 or RHEL/CentOS 8, or RHEL/AlmaLinux 9.

To make the rule permanent, just use any of the methods that we've already discussed. When you do, the rule will show up in the `.xml` file for the default zone. In my case, the default zone is still set to public. So, let's look in the `/etc/firewalld/zones/public.xml` file:

```
<?xml version="1.0" encoding="utf-8"?>
<zone>
  <short>Public</short>
  <description>For use in public areas. You do not trust the other computers on networks to not harm your computer. Only selected incoming connections are accepted.</description>
  <service name="ssh"/>
  <service name="dhcpv6-client"/>
  <service name="cockpit"/>
  <rule family="ipv4">
    <source address="192.168.0.225"/>
    <service name="http"/>
    <drop/>
  </rule>
</zone>
```

Our rich rule shows up in the `rule family` block at the bottom of the file.

Now that we've covered what's common between the RHEL/CentOS 7 and the RHEL/CentOS/AlmaLinux 8/9 versions of firewalld, let's look at what's particular to each different version.

### Looking at iptables rules in RHEL/CentOS 7 firewalld

RHEL 7 and its offspring use the iptables engine as the firewalld backend. You can't create rules with the normal iptables commands as long as firewalld is enabled. However, every time you create a rule with a `firewall-cmd` command, the iptables backend creates the appropriate iptables rule and inserts it into its proper place. You can view the active rules with `iptables -L`. Here's the first part of a very long output:

```
[donnie@localhost ~]$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
ACCEPT     all  --  anywhere             anywhere            
INPUT_direct  all  --  anywhere             anywhere            
INPUT_ZONES_SOURCE  all  --  anywhere             anywhere            
INPUT_ZONES  all  --  anywhere             anywhere            
DROP       all  --  anywhere             anywhere             ctstate INVALID
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited
```

As was the case with ufw on Ubuntu, a lot has already been configured for us. At the top, in the `INPUT` chain, we see that the connection state rule and the rule to block invalid packets are already there. The default policy for the chain is `ACCEPT`, but the final rule of the chain is set to `REJECT` what isn't specifically allowed. In between these, we can see rules that direct other packets to other chains for processing. Now, let's look at the next portion:

```
Chain IN_public_allow (1 references)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh ctstate NEW
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain ctstate NEW
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain ctstate NEW
Chain IN_public_deny (1 references)
target     prot opt source               destination         
REJECT     icmp --  anywhere             anywhere             icmp host-redirect reject-with icmp-host-prohibited
```

Toward the bottom of the very long output, we can see the `IN_public_allow` chain, which contains the rules that we created for opening firewall ports. Just below that is the `IN_public_deny` chain, which contains the `REJECT` rule for blocking unwanted ICMP types. In both the `INPUT` chain and the `IN_public_deny` chain, the `REJECT` rules return an ICMP message to inform the sender that the packets were blocked.

Now, keep in mind that there's a lot of this `IPTABLES -L` output that we haven't shown. So, look at it for yourself to see what's there. When you do, you may ask yourself, *Where are these default rules stored? Why am I not seeing them in the* `/etc/firewalld/` *directory?*

To answer that question, I had to do some rather extensive investigation. For some truly bizarre reason, the Red Hat folk have left this completely undocumented. I finally found the answer in the `/usr/lib/python2.7/site-packages/firewall/core/` directory. Here, there's a set of Python scripts that set up the initial default firewall:

```
[donnie@localhost core]$ ls
base.py fw_config.pyc fw_helper.pyo fw_ipset.py fw_policies.pyc fw_service.pyo fw_zone.py icmp.pyc ipset.pyc logger.pyo rich.py base.pyc fw_config.pyo fw_icmptype.py fw_ipset.pyc fw_policies.pyo fw_test.py fw_zone.pyc icmp.pyo ipset.pyo modules.py rich.pyc base.pyo fw_direct.py fw_icmptype.pyc fw_ipset.pyo fw.py fw_test.pyc fw_zone.pyo __init__.py ipXtables.py modules.pyc rich.pyo ebtables.py fw_direct.pyc fw_icmptype.pyo fw_nm.py fw.pyc fw_test.pyo helper.py __init__.pyc ipXtables.pyc modules.pyo watcher.py ebtables.pyc fw_direct.pyo fw_ifcfg.py fw_nm.pyc fw.pyo fw_transaction.py helper.pyc __init__.pyo ipXtables.pyo prog.py watcher.pyc ebtables.pyo fw_helper.py fw_ifcfg.pyc fw_nm.pyo fw_service.py fw_transaction.pyc helper.pyo io logger.py prog.pyc watcher.pyo fw_config.py fw_helper.pyc fw_ifcfg.pyo fw_policies.py fw_service.pyc fw_transaction.pyo icmp.py ipset.py logger.pyc prog.pyo
[donnie@localhost core]$
```

The script that does most of the work is the `ipXtables.py` script. If you look in it, you'll see that its list of iptables commands matches up with the `iptables -L` output.

### Creating direct rules in RHEL/CentOS 7 firewalld

As we've seen, any time we do anything with the normal `firewall-cmd` commands on RHEL/CentOS 7, firewalld automatically translates those commands into iptables rules and inserts them into the proper place. (Or, it deletes the rules, if you've issued some sort of delete command.) However, there are some things that we can't do with the normal `firewalld-cmd` commands. For example, we can't use normal `firewall-cmd` commands to place rules in a specific iptables chain or table. To do things like that, we need to use the direct configuration commands.

The `firewalld.direct` man page and the documentation at the Red Hat site both warn you to only use direct configuration as an absolute last resort when nothing else will work. That's because, unlike the normal `firewall-cmd` commands, the direct commands won't automatically place your new rules into the proper places so that everything works correctly. With the direct commands, you can break the whole firewall by placing a rule into the wrong spot.

In the example output of the previous section, in the default ruleset, you saw that there's a rule in the filter table's `INPUT` chain that blocks invalid packets. In the *Blocking invalid packets with iptables* section of *Chapter 4*, *Securing your server with a firewall-Part 1*, you saw that this rule misses certain types of invalid packets. So, we'd like to add a second rule to block what the first rule misses. We'd also like to place these rules into the `PREROUTING` chain of the mangle table in order to enhance firewall performance. To do this, we need to create a couple of direct rules. (This isn't hard if you're familiar with normal iptables syntax.) So, let's get to it.

First, let's verify that we don't have any effective direct rules, like so:

```
sudo firewall-cmd --direct --get-rules ipv4 mangle PREROUTING
sudo firewall-cmd --direct --get-rules ipv6 mangle PREROUTING
```

You should get no output for either command. Now, let's add our two new rules, for both IPv4 and IPv6, with the following four commands:

```
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
```

The `direct` command syntax is very similar to that of normal iptables commands. So, I won't repeat the explanations that I've already presented in the iptables section. However, I do want to point out the `0` and the `1` that come after `PREROUTING` in each of the commands. Those represent the priority of the rule. The lower the number, the higher the priority, and the higher up the rule is in the chain. So, the rules with the `0` priority are the first rules in their respective chains, while the rules with the `1` priority are the second rules in their respective chains. If you give the same priority to each rule you create, there's no guarantee that the order will remain the same upon each reboot. So, be sure to assign a different priority to each rule.

Now, let's verify that our rules are in effect:

```
[donnie@localhost ~]$ sudo firewall-cmd --direct --get-rules ipv4 mangle PREROUTING
0 -m conntrack --ctstate INVALID -j DROP
1 -p tcp '!' --syn -m conntrack --ctstate NEW -j DROP
[donnie@localhost ~]$ sudo firewall-cmd --direct --get-rules ipv6 mangle PREROUTING
0 -m conntrack --ctstate INVALID -j DROP
1 -p tcp '!' --syn -m conntrack --ctstate NEW -j DROP
[donnie@localhost ~]$
```

We can see that they are. When you use the `iptables -t mangle -L` command and the `ip6tables -t mangle -L` command, you'll see that the rules show up in the `PREROUTING_direct` chain. (I'm only showing the output once since it's the same for both commands.):

```
. . .
. . .
Chain PREROUTING_direct (1 references)
target prot opt source destination 
DROP all -- anywhere anywhere ctstate INVALID
DROP tcp -- anywhere anywhere tcp flags:!FIN,SYN,RST,ACK/SYN ctstate NEW
. . .
. . .
```

To show that it works, we can perform some Nmap scans against the virtual machine, just like how I showed you to in the *Blocking invalid packets with iptables* section of *Chapter 4*, *Securing your server with a firewall-Part 1*. (Don't fret if you don't remember how to do it. You'll see the procedure in the upcoming hands-on lab.) Then, we can use `sudo iptables -t mangle -L -v` and `sudo ip6tables -t mangle -L -v` to see the packets and bytes that these two rules blocked.

We didn't use the `--permanent` option with these commands, so they're not permanent yet. Let's make them permanent now:

```
[donnie@localhost ~]$ sudo firewall-cmd --runtime-to-permanent
[sudo] password for donnie: 
success
[donnie@localhost ~]$
```

Now, let's take a look in the `/etc/firewalld/` directory. Here, you'll see a `direct.xml` file that wasn't there before:

```
[donnie@localhost ~]$ sudo ls -l /etc/firewalld
total 20
-rw-r--r--. 1 root root  532 Aug 26 13:17 direct.xml
. . .
. . .
[donnie@localhost ~]$
```

Look inside the file; you'll see the new rules:

```
<?xml version="1.0" encoding="utf-8"?>
<direct>
  <rule priority="0" table="mangle" ipv="ipv4" chain="PREROUTING">-m conntrack --ctstate INVALID -j DROP</rule>
  <rule priority="1" table="mangle" ipv="ipv4" chain="PREROUTING">-p tcp '!' --syn -m conntrack --ctstate NEW -j DROP</rule>
  <rule priority="0" table="mangle" ipv="ipv6" chain="PREROUTING">-m conntrack --ctstate INVALID -j DROP</rule>
  <rule priority="1" table="mangle" ipv="ipv6" chain="PREROUTING">-p tcp '!' --syn -m conntrack --ctstate NEW -j DROP</rule>
</direct>
```

The official Red Hat 7 documentation page does cover direct rules, but only briefly. For more detailed information, see the `firewalld.direct` man page.

### Looking at nftables rules in RHEL/AlmaLinux 8 and 9 firewalld

RHEL 8/9 and their offspring use nftables as the default firewalld backend. Every time you create a rule with a `firewall-cmd` command, the appropriate nftables rule is created and inserted into its proper place. To look at the ruleset that's currently in effect, we'll use the same nft command that we used with nftables on Ubuntu:

```
[donnie@localhost ~]$ sudo nft list ruleset
. . .
. . .
table ip firewalld {
    chain nat_PREROUTING {
        type nat hook prerouting priority -90; policy accept;
        jump nat_PREROUTING_ZONES_SOURCE
        jump nat_PREROUTING_ZONES
    }
    chain nat_PREROUTING_ZONES_SOURCE {
    }
. . .
. . .
[donnie@localhost ~]$
```

Again, we can see a very lengthy list of default, pre-configured firewall rules. (To see the whole list, run the command for yourself.) You'll find these default rules in the `/usr/lib/python3.6/site-packages/firewall/core/nftables.py` script on RHEL 8-type machines, and in the `/usr/lib/python3.9/site-packages/firewall/core/nftables.py` script on RHEL 9-type machines. This script runs every time you boot up the machine.

### Creating direct rules in RHEL/AlmaLinux firewalld

Okay, here's where things get downright weird. Even though the direct rule commands create iptables rules and the RHEL 8/9 distros use nftables for the firewalld backend, you can still create direct rules. Just create and verify them the same way that you did in the *Creating direct rules in RHEL/CentOS 7 firewalld* section. Apparently, firewalld allows these iptables rules to peacefully coexist with the nftables rules. However, if you need to do this on a production system, be sure to thoroughly test your setup before putting it into production.

There's nothing about this in the Red Hat 8/9 documentation, but there is the `firewalld.direct` man page if you want to find out more.

#### Hands-on lab for firewalld commands

By completing this lab, you'll get some practice with basic firewalld commands:

1.  Log into your CentOS 7 virtual machine or either of the AlmaLinux virtual machines and run the following commands. Observe the output after each one:

```
 sudo firewall-cmd --get-zones
 sudo firewall-cmd --get-default-zone
 sudo firewall-cmd --get-active-zones
```

1.  Briefly view the man pages that deal with `firewalld.zones`:

```
 man firewalld.zones
 man firewalld.zone
```

(Yes, there are two of them. One explains the zone configuration files, while the other explains the zones themselves.)

1.  Look at the configuration information for all of the available zones:

```
sudo firewall-cmd --list-all-zones
```

1.  Look at the list of predefined services. Then, look at the information about the `dropbox-lansync` service:

```
 sudo firewall-cmd --get-services
 sudo firewall-cmd --info-service=dropbox-lansync
```

1.  Set the default zone to `dmz`. Look at the information concerning the `zone`, add the `http` and `https` services, and then look at the `zone` information again:

```
 sudo firewall-cmd --set-default-zone=dmz
 sudo firewall-cmd --permanent --add-service={http,https}
 sudo firewall-cmd --info-zone=dmz
 sudo firewall-cmd --permanent --info-zone=dmz
```

1.  Reload the **firewall** configuration and look at `zone` information again. Also, look at the list of services that are being allowed:

```
 sudo firewall-cmd --reload
 sudo firewall-cmd --info-zone=dmz
 sudo firewall-cmd --list-services
```

1.  Permanently open port `10000/tcp` and view the results:

```
 sudo firewall-cmd --permanent --add-port=10000/tcp
 sudo firewall-cmd --list-ports
 sudo firewall-cmd --reload
 sudo firewall-cmd --list-ports
 sudo firewall-cmd --info-zone=dmz
```

1.  Remove the port that you just added:

```
 sudo firewall-cmd --permanent --remove-port=10000/tcp
 sudo firewall-cmd --reload
 sudo firewall-cmd --list-ports
 sudo firewall-cmd --info-zone=dmz
```

1.  Add a rich language rule to block a geographic range of IPv4 addresses:

```
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="200.192.0.0/24" service name="http" drop'
```

1.  Block the `host-redirect` and `network-redirect` ICMP types:

```
sudo firewall-cmd --permanent --add-icmp-block={host-redirect,network-redirect}
```

1.  Add the directive to log all dropped packets:

```
sudo firewall-cmd --set-log-denied=all
```

1.  View both the `runtime` and `permanent` configurations and note the differences between them:

```
sudo firewall-cmd --info-zone=dmz
sudo firewall-cmd --info-zone=dmz --permanent
```

1.  Make the `runtime` configuration `permanent` and verify that it took effect:

```
sudo firewall-cmd --runtime-to-permanent
sudo firewall-cmd --info-zone=dmz --permanent
```

1.  On CentOS 7, view the complete list of effective firewall rules by doing:

```
sudo iptables -L
```

1.  On AlmaLinux 8 or 9, view the complete list of effective firewall rules by doing:

```
sudo nft list ruleset
```

1.  Create the `direct` rules in order to block invalid packets from the mangle table's `PREROUTING` chain:

```
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv4 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 0 -m conntrack --ctstate INVALID -j DROP
sudo firewall-cmd --direct --add-rule ipv6 mangle PREROUTING 1 -p tcp ! --syn -m conntrack --ctstate NEW -j DROP
```

1.  Verify that the **rules** took effect and make them **permanent**:

```
sudo firewall-cmd --direct --get-rules ipv4 mangle PREROUTING
sudo firewall-cmd --direct --get-rules ipv6 mangle PREROUTING
sudo firewall-cmd --runtime-to-permanent
```

1.  View the contents of the `direct.xml` file that you've just created:

```
sudo less /etc/firewalld/direct.xml
```

1.  Perform XMAS Nmap scans for both IPv4 and IPv6 against the virtual machine. Then, observe which rule was triggered by the scan:

```
sudo nmap -sX ipv4_address_of_Test-VM
sudo nmap -6 -sX ipv6_address_of_Test-VM
sudo iptables -t mangle -L -v
sudo ip6tables -t mangle -L -v
```

1.  Repeat *step 19*, but this time with a Windows scan:

```
sudo nmap -sW ipv4_address_of_Test-VM
sudo nmap -6 -sW ipv6_address_of_Test-VM
sudo iptables -t mangle -L -v
sudo ip6tables -t mangle -L -v
```

1.  View the list of main pages for firewalld:

```
apropos firewall
```

That's the end of the lab – congratulations!

## Summary

In this chapter, we looked at two helper utilities that can simplify using either iptables or nftables. We started with ufw, which is available for the Debian and Ubuntu families. Then, we looked at firewalld, which used to be specific to Red Hat-type distros, but is now also available in Ubuntu repositories and comes already installed and activated on the SUSE distros.

In the space that I've been allotted, I've presented the basics of using these technologies to set up single-host protection. I've also presented some details about the innards of firewalld that you won't find documented anywhere else, including in the official Red Hat documentation.

In the next chapter, we'll look at the various encryption technologies that can help keep your data private. I'll see you there.

## Questions

1.  What is the major difference between firewalld on RHEL 7-type distros and firewalld on RHEL 8/9-type distros?
2.  In which of the following formats does firewalld store its rules?
    1.  `.txt`
    2.  `.config`
    3.  `.html`
    4.  `.xml`
3.  Which of the following commands would you use to list all of the firewalld zones on your system?
    1.  sudo firewalld --get-zones
    2.  sudo firewall-cmd --list-zones
    3.  sudo firewall-cmd --get-zones
    4.  sudo firewalld --list-zones
4.  With ufw, everything you'll ever need to do can be done with the ufw utility.
    1.  True
    2.  False
5.  Your system is set up with firewalld and you need to open port `10000/tcp`. Which of the following commands would you use?
    1.  sudo firewall-cmd --add-port=10000/tcp
    2.  sudo firewall-cmd --add-port=10000
    3.  sudo firewalld --add-port=10000
    4.  sudo firewalld --add-port=10000/tcp
6.  Which of the following ufw commands would you use to open the default Secure Shell port?
    1.  sudo ufw allow 22
    2.  sudo ufw permit 22
    3.  sudo ufw allow 22/tcp
    4.  sudo ufw permit 22/tcp

## Further reading

*   Rate-limiting with ufw: [https://45squared.com/rate-limiting-with-ufw/](https://45squared.com/rate-limiting-with-ufw/)
*   firewalld documentation for RHEL 7: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls)
*   firewalld documentation for RHEL 8: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/using-and-configuring-firewalls_securing-networks](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/securing_networks/using-and-configuring-firewalls_securing-networks)
*   The firewalld home page: [https://firewalld.org/](https://firewalld.org/)
*   UFW Community Help Wiki: [https://help.ubuntu.com/community/UFW](https://help.ubuntu.com/community/UFW)
*   How to set up a Linux firewall with UFW on Ubuntu 18.04: [https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-ubuntu-18-04/](https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-ubuntu-18-04/)

## Answers

1.  RHEL 7 distros use iptables as the firewalld backend, and RHEL 8/9 distros use nftables as the firewalld backend.
2.  D
3.  C
4.  B
5.  A
6.  C