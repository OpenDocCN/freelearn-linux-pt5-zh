# Network Security

In this chapter, we will discuss the following topics:

*   Managing TCP/IP networks
*   Using a packet sniffer to monitor network traffic
*   Using IP tables for configuring a firewall
*   Blocking spoofed addresses
*   Blocking incoming traffic
*   Configuring and using TCP Wrappers
*   Blocking country-specific traffic using `mod_security`
*   Securing network traffic using SSL

# Managing TCP/IP networks

As the size of a computer network grows, managing the network's information becomes an important task for the system administrator.

# Getting ready

Before we start with making any changes in the TCP/IP configuration, make sure to create a backup of the Network Manager configuration file by using the following command:

![](img/b0f31cd4-7c0e-462a-8adf-ebf05c674d88.png)

Also make a backup of the `/etc/network/interfaces` file in the same way.

# How to do it...

In this section, we will see how we can manually configure the network settings using the command line:

1.  Before starting with the manual configuration, first, let's check our current IP address, which has been assigned to the system automatically by DHCP. We can check the details graphically by right-clicking on the networking icon on the top-right panel and then selecting **Connection Information**, as shown in the following screenshot:

![](img/5ab25ae7-7a55-4fea-883e-5e2b9e956e33.png)

We can see that the current IP address of our system is `192.168.1.101`.

2.  Next, we check the same information using the command line by typing in the `ifconfig` command:

![](img/4fae4535-1083-4e3b-bf53-8b6983c4b871.png)

3.  If we want to just check the available Ethernet devices on the system, we can run the following command:

![](img/e6a46e22-c50a-4d71-8f8d-5e1b2eb21967.png)

The preceding command will list a one-line description of all the available Ethernet devices on the system.

4.  If we want to get more detailed information about the network interface, we can use a tool called `lshw`, like so:

![](img/d3764960-3c5f-422c-98a9-783fb4d65ea0.png)

This command also gives detailed information about other capabilities of the hardware.

5.  Now, we will disable the Network Manager and then set the IP address details manually. To disable the Network Manager, edit the `/etc/NetworkManager/NetworkManager.conf` file, like so:

![](img/93905e21-5d7f-4938-8257-a0aa21fe2afa.png)

Change the line `managed=false` to `managed=true` and save the file.

6.  Now, open the `/etc/network/interfaces` file in any editor of your choice. We can see that, by default, there is no information regarding the `eth0` interface:

![](img/7e90ef0c-5c2c-4073-8d65-258774991308.png)

7.  Edit the file and add the information shown in the following screenshot. Make sure to add the IP details according to your network settings:

![](img/b24926ca-eb9a-48bd-b342-7d1f60a80380.png)

When done, save the file and then reboot the computer to `disengage` the Network Manager.

8.  If you wish to create a virtual network adapter, you can add the following lines in the `/etc/network/interfaces` file, as follows:

![](img/3b0eb9b3-7837-4cc1-bfda-a26d278ebdcb.png)

By doing this, we have added two IP address to the single Ethernet card. We can do this to create multiple instances of the network card.

9.  Once you have completed editing, restart the networking service by using the following command:

```
    service network-manager restart
```

You can also use this command:

```
    /etc/init.d/networking restart
```

10.  Next, we can look at how to configure the appropriate `nameserver` to be used if the IP address is being configured manually.

To make changes, edit the `/etc/resolv.conf` file in any editor, and add the lines shown in the following screenshot:

![](img/45898733-b16f-4e7f-b2ae-e2d6425ecef3.png)

Following the preceding steps, we will be able to configure the IP details successfully.

# How it works...

The TCP/IP settings on a system can be either managed automatically or manually. Depending on the content in the `/etc/NetworkManager/NetworkManager.conf` file, the system understands whether the settings will be managed automatically or manually.

For manual configuration, we edit the `/etc/network/interfaces` file and enter the IP details shown in the preceding section. Once this is done, we restart the networking service or completely reboot the system to make the changes effective.

# Using a packet sniffer to monitor network traffic

One of the most widely used command-line packet sniffer or packet analyzer tools for Linux is Tcpdump. It helps to capture or filter TCP/IP packets being transferred or received on a specific interface over the network.

# Getting ready

Tcpdump comes pre-installed in most Linux/Unix-based operating systems. If it is not available, we can install it by using the following command:

![](img/5a5957ea-402e-4804-a654-7a30ce437ac2.png)

# How to do it...

Once `tcpdump` has been installed, we can start using it by simply running the command `tcpdump`:

1.  When we simply run `tcpdump`, it will start capturing all the packets sent or received on any interface.

![](img/87c8ed39-ad3f-4f1a-9743-0a8a22a49898.png)

2.  If we want to capture the packets that are only on a specific interface, we can do the same as shown in the following screenshot:

![](img/60ff0cd1-5b13-4a18-9e74-03f0d85798e9.png)

3.  The preceding command will capture all the packets received on the defined interface, until manually stopped. If we wish to capture a specific count of packets, we can do so by using the `-c` option, as follows:

![](img/34a64f0b-d78f-4cb6-badd-363f997b3f7b.png)

4.  To display the captured packets in ASCII format, we can use the `-A` option:

![](img/44d3f1d5-dd2b-47a0-a49d-7ebddfcef19b.png)

5.  If we wish to list the number of available interfaces on the system, we can do the same using the `-D` option:

![](img/b6fe1c36-a7ce-49c1-87e6-85e74a9982a0.png)

6.  If we use the `-XX` option while capturing the packets, tcpdump will capture the packet's link level header in HEX and ASCII format, as follows:

![](img/f18c7bfa-cb69-4c6c-80b7-81b1a92ac092.png)

7.  We can save the captured packets in a file in `.pcap` format by using the `-w` option while executing `tcpdump`:

![](img/261db675-ebe5-4996-8ea8-8f54fa63bd86.png)

In the preceding command, we have the saved the data in the `capture.pcap` file.

8.  When we want to read and analyze the captured packet file, we use the command with the `-r` option, as follows:

![](img/5e84c7e1-444b-4c9e-8ee1-a3566d9ba335.png)

9.  By default, `tcpdump` captures packets for all ports. If we want to capture packets for any specific port, for example, port `80`, we can do so as follows:

![](img/f159ac37-125c-45b0-b967-b23fc61b4760.png)

# How it works...

TCPdump analyzes network behavior, performance, and applications that generate or receive network traffic. Tcpdump uses `libpacp/winpcap` to capture data and uses it extensive protocol definitions that are built inside it to analyze the captured packets.

# Using IP tables for configuring a firewall

One of the essential steps while securing a Linux system is setting up a good firewall. Most Linux distributions come pre-installed with different firewall tools. Iptables is one such default firewall of Linux distributions. For older versions of Linux kernel, Ipchains was the default firewall.

# Getting ready

Since **Iptables** ships with the Linux distribution, no extra tools need to be installed to use it. However, it is recommended that to use Iptables, we should not use the root account. Instead, use a normal account that has super user access to run the commands efficiently.

# How to do it...

We can define different rules using Iptables. These rules are then followed by the kernel when checking the incoming and outgoing traffic packets:

1.  The first thing we shall do on our system is check which version of `iptables` is installed by using the following command:

![](img/4d843be0-4a88-4cc3-95e2-d1429c4514f6.png)

2.  Now, we will check whether any rule already exists on the system for Iptables by using the `-L` option:

![](img/8746402f-62e7-4bd4-8bd7-714da47c9220.png)

3.  The preceding output can also be seen in a format that tells us about the commands that are necessary for each policy. To do this, use the `-S` option, as follows:

![](img/db6f89cf-8719-4f45-a7a5-c88d7360870f.png)

4.  Now, we will check which modules of `iptables` are loaded by default for its proper functionality by using the following command:

![](img/0252dbdb-858c-43b9-ba38-b31ea8da7181.png)

5.  Now, let's add our first rule, which will make sure that all the online connections at present will stay online, even after we have made the rules to block the unwanted services:

```
    iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Here, the `-A` option appends a rule to the existing table. `INPUT` tells us that this rule will be appended in the input chain of Iptables. The next arguments of the command, `-m conntrack --ctstateESTABLISHED,RELATED`, makes sure that the rule applies only to the connections which are online currently. Then `-j ACCEPT` tells the `iptables` to accept and allow the packets that match the criteria specified previously.

6.  Now, if we check the list of rules in `iptables` again, we can see that our rule has been added:

**![](img/82c3dc12-58b7-44aa-b269-b25bb3251d5f.png)**

7.  Now, let's assume that we want to keep our SSH connection allowed through Iptables. To do so, we add the rule shown in the following screenshot:

**![](img/0de7e69d-6bc6-4541-9887-3a660acc28bb.png)**

We have used port `22` as it is the default port for SSH. If you have changed the port for SSH in your server, use the appropriate port in the preceding screenshot.

8.  We also need to make sure that our server continues to function properly by letting the services on the server communicate with each other without being blocked by the rules of the Iptables. To do this, we want to allow all the packets being sent to the loopback interface.

We add the following rule to allow loopback access:

```
 iptables -I INPUT 1 -i lo -j ACCEPT
```

Here, the `-I` option tells the iptables to insert a new rule rather than append it. It takes the chain and the position where the new rule needs to be added. In the preceding command, we are adding this rule as the first rule in the `INPUT` chain, so that this is the first rule that's applied.

9.  Now, if we see the list of rules in Iptables by using the `-v` option, we can see the rule for loopback interface, `lo`, as our first rule:

**![](img/78a17dae-e0f7-421a-982a-bb4ae61b55fa.png)**

10.  Now, assuming that we have added the rules for all the packets to be allowed as per our requirements, we have to make sure that any other packet that enters the `INPUT` chain should be blocked.

To do so, we will modify the `INPUT` chain by running the `iptables -A INPUT -j DROP` command:

**![](img/33c3d3dd-b51e-4c49-9f5a-a325b6e7a7b4.png)**

We can see from the preceding code that the rule to drop all packets has been added to the bottom of the list in the `INPUT` chain. This makes sure that whenever a packet comes in, the `itpables` rules are checked in the order specified. If none of the rules match for the packet, it will get dropped, thus preventing a packet from being accepted by default.

11.  Until now, whatever rules we have added in Iptables are non-persistent. This means that as soon the system is restarted, all the rules of `iptables` will be gone. In order to save the rules that we have created and then automatically load them when the server reboots, we can use the `iptables-persistent` package.

12.  Install the package by using the following command:

```
    apt-get install iptables-persistent
```

**![](img/cb298f2a-0ef0-4c82-9864-698be033cc22.png)**

13.  During the installation process, you will be asked if you want to save the current iptables rules and automatically load them. Select `Yes` or `No` as per your choice.

14.  Once the installation is complete, we can start the package by running the following command:

**![](img/364a9691-d26e-4c94-9e16-51829fed92c8.png)**

# How it works...

In the preceding example, we used Iptables in Linux to configure firewalls on our system.

First, we went through the basic options of the `iptables` command, and then we saw how to add different rules in `iptables`. We added rules to allow localhost access and outgoing active connections. Then, we added a rule to allow for SSH connection.

Then, we added the rule to deny every other incoming packet that does not match the aforementioned applied rules.

Lastly, we used the `iptables-persistent` package to save the rules of iptables even after system reboot.

# Blocking spoofed addresses

IP spoofing is a very common technique used by attackers to send malicious packets to a server computer. It is the process of creating IP packets with a forged IP address. This is mainly used for performing attacks like **Denial of Service** (**DoS**) attacks.

# Getting ready

If we wish to block spoofed IP addresses, we need to have a list of those IP address or the domain names from where these spoofed connections are trying to connect.

# How to do it...

We will try to create a basic rule set of iptables, using which we will restrict all the incoming packets, except for those that are necessary for us:

1.  The first step will be to create a rule to allow access to the loopback interface so that the services on the system can communicate properly with each other locally. The command to do so is as follows:

```
    iptables -A INPUT -i lo -j ACCEPT  
```

![](img/95a43f69-a7f8-447c-9bd0-add1d9ffdc2a.png)

This is necessary for the system to function properly.

2.  Next, we will create the rule for the outbound connections that have been initiated by our system:

```
    iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

This will accept all the outbound traffic, including the responses from the remote servers that we have tried to connect ourselves (such as any website we're visiting):

![](img/eee92b3c-b58b-4996-9b22-5cbfb7848d6f.png)

3.  Now, let's create a table to be used in `iptables`. We have called it `blocked_ip`. You can choose any name you want:

```
    iptables -N blocked_ip  
```

This is the table where we will add the spoofed IP addresses that we want to block.

4.  Now, we will insert this table into the `INPUT` table of iptables by using the following command:

```
    iptables -I INPUT 2 -j blocked_ip  
```

Note that we have used the number `2` to make sure that this rule will be the second from the top in the `iptables`.

5.  Now, let's add the bad IPs into the `blocked_ip` table that we have created:

```
    iptables -A blocked_ip -s 192.168.1.115 -j DROP  
```

We have used the IP address `192.168.1.115` as an example here. You can replace it with the IP address that you want to block. If you have more than one IP address to block, add them one by one to `iptables`.

6.  Now, we can see the list of rules in `iptables` by using the following command:

```
    iptables -L
```

In the details shown in the following screenshot, we can see that, at the bottom, we have the IP address that we are trying to block. You can specify a single IP address or a range, as per your choice:

![](img/269b745d-8bed-476c-91d1-662d6ff4981f.png)

7.  After making rules in the Iptables, we can edit the `/etc/host.conf` file as well. Open the file in any editor of your choice. I am using nano:

```
    nano /etc/host.conf
```

Now, add or edit the following lines in the file, as follows:

```
    order hosts,bind
    nospoof on
```

**![](img/8553560a-146e-4d9d-95b6-270a28219cf3.png)**

In the preceding example, the nospoof on option does a comparison of IP address returned by hostname lookup with the hostname returned by IP address lookup. If the comparison fails, this option generates a spoof warning.

Once done, save and close the file. This will also help protect the system from IP spoofing.

# How it works...

To block a spoofed IP address or any other IP address, we again use Iptables as it is the default firewall, unless we don't want to use any other tool available for Linux.

We again create rules to allow localhost access in the system and also to keep the outbound active connections alive. Then, we create a table in Iptables that we use to maintain a list of the spoofed IP addresses that we want to block. After, we add this table to the input chain of Iptables. Now, we can add any IP address to the table whenever required and it will automatically get blocked.

We can also use the `/etc/host.conf` file to protect the system from IP spoofing.

# Blocking incoming traffic

One of the most important tasks for a Linux system administrator is to control access to the network services. At times, it may be better to block all incoming traffic on the server and only allow required services to connect.

# Getting ready

As we will be using `iptables` here as well, no extra packages are needed to perform these steps. We just need a user account with super user access. However, this account should preferably not be a `root` account.

# How to do it...

We will configure Iptables to deny everything, except the traffic that has been initiated from inside our system (for example, the web browsers have web traffic, or some downloading has already been initiated earlier for updating the package or any other software):

1.  As in the previous examples, our first rule in Iptables will be to allow access to localhost data. Run the following command to do this:

```
    iptables -A INPUT -i lo -j ACCEPT  
```

![](img/d118c30f-ec03-4cc5-a361-c6e209565291.png)

2.  Our next rule will be for accepting all traffic related to outbound connections. This also includes the responses from the remote server to which our system is connecting:

![](img/51b60836-439b-49c1-93e7-7a12e1584e95.png)

3.  Next, we will add the rule to accept Time Exceeded ICMP packets. This is important for time-restricted connection setups:

```
    iptables -A INPUT -p icmp -m icmp --icmp-type 11 -j ACCEPT   
```

4.  After this, we will add the rule to accept Destination Unreachable ICMP packets coming from remote servers:

```
    iptables -A INPUT -p icmp -m icmp --icmp-type 3/4 -j ACCEPT
```

5.  Next, add the rule to accept PING requests/responses (Echo ICMP) to keep our system's connections alive to those web services that may require PING:

```
    iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT  
```

6.  Once the preceding rules have been added, we check the list in Iptables by running the following command:

```
    iptables -L 
```

![](img/c28d582e-73a6-4626-8f31-b8b89a239233.png)

7.  Now, we will create a table of iptables, which will contain a list of acceptable rules and services:

```
    iptables -N allowed_ip  
```

We will then add this table to the `INPUT` chain of Iptables:

```
    iptables -A INPUT -j allowed_ip  
```

8.  Now, let's add a rule so that access to SSH is allowed on the system. To do so, we can run the following command:

```
    iptables -A allowed_ip -p tcp --dport 22 -j ACCEPT
```

9.  Now, if we check the list of rules in iptables, we get the following result:

```
 iptables -L 
```

![](img/43f27643-b7e1-4ab6-bf6d-3614cf6e1768.png)

10.  Once we have added the rules to accept the traffic we want to, we now want to reject all other traffic for which no rules have been set. To do so, we add the following rule:

```
    iptables -A INPUT -j REJECT --reject-with icmp-host-unreachable

```

By doing this, whenever anyone tries to connect to the server, a **Host Unreachable** ICMP packet will be sent to them that would then terminate the connection attempt.

11.  After adding all the aforementioned given rules, our iptables will now look like what's shown in the following screenshot:

![](img/3132968a-b873-4261-a18c-acc574499721.png)

# How it works...

To block all the incoming traffic on the server and allow only the outbound connections, we again use Iptables as it is the default firewall of Linux.

To allow the proper functioning of the server internally, we allow access to localhost.

Next, to keep the outbound connections active, we add rules to accept the **Time Exceeded**, **Destination Unreachable**, and **Echo** ICMP packets.

Once these rules have been added, we can define whether we want to allow any incoming traffic for some particular services, such as SSH or a particular client address. For this, we create a table to add the list of IP address for the clients we want to allow. After, we add a rule to allow access to SSH service, or any other service as per our requirements.

Lastly, we add the rule to reject all the traffic for which no rule has been added.

# Configuring and using TCP Wrappers

Securing a server by restricting access is a critical measure that should never be omitted while setting up a server. Using TCP Wrappers, we can allow only those networks to have access to our server's services that we have been configured and support TCP Wrappers.

# Getting ready

For demonstrating the following steps, we are using two systems that are on the same network and can ping to each other successfully. One system will be used as a server and the other will be used as a client.

# How to do it...

Linux provides several tools for controlling access to the network services. TCP Wrappers is one among those and adds an additional layer of protection. In the following steps, we will see how to configure TCP Wrappers to define the access for the different hosts:

1.  First, we need to check whether a program supports TCP Wrappers or not. To do so, first, find the path of the program executable by using the `which` command:

```
    which sshd 
```

![](img/3212b0fd-357b-4987-8a71-f22c0ae9c919.png)

Here, we have used the `SSH` program as example.

2.  Next, we use the `ldd` program to check the compatibility of the `SSH` program with TCP Wrappers:

```
    ldd /usr/sbin/sshd  
```

![](img/b986fb5e-ed5c-40a2-8e87-f9e244190de7.png)

If the output of the preceding command includes `libwrap.so`, it means that the program is supported by TCP Wrappers.

3.  Now, whenever the SSH program tries to connect to the server using TCP Wrappers, two files are checked in the following order:

*   *   `/etc/hosts.allow`: If a matching rule is found in this file for the program, access will be given
    *   `/etc/hosts.deny`: If a matching rule is found in this file for the program, access will be denied

4.  If no matching rule is found in either of the two files for the specific program, access will be given.

5.  If we try to connect to the SSH server, before adding any rule, we will see that it connects successfully:

![](img/e245501f-e197-4518-b2ca-d8460cf310c9.png)

6.  Now, let's suppose we want to deny access to the SSH program for a particular system that has the given IP address. To do so, we will edit the `/etc/hosts.deny` file, as follows:

![](img/e8d6bf51-bced-49da-ad3a-456ba4ecf3b9.png)

7.  Now, if we try to connect to the SSH server from this particular system for which we have denied access, it shows the following error:

![](img/47fe79d3-0e89-458e-a072-e50c12e92927.png)

8.  If we want to allow access for all the programs and all the clients, either add no rules in either of the two files or add the following line in the `/etc/hosts.allow` file:

![](img/fd9b510c-0d7c-4bb4-a961-57f4aa8af4ee.png)

9.  If we want to allow access to all the services from a particular client that has the IP address `192.168.1.106`, then we add the following line in the `/etc/hosts.allow` file:

![](img/e7be05ec-a52d-48c9-b939-7b915e263817.png)

10.  If we want to allow all the clients on a particular network to access SSH, except for a particular client that has the IP address `192.168.1.100`, we perform the following changes in the `/etc/hosts.allow` file:

![](img/92849ffa-d71e-46d3-8034-96a4e0f16e88.png)

11.  After making the aforementioned changes, when we try to connect through SSH, we will see the following error:

![](img/615241c2-28df-4a6e-9976-0bdc280abf47.png)

We can see that once the IP address is changed for the client, SSH access is now allowed, which means that all the clients on the particular network can access SSH, except for the IP address, which has been denied.

12.  The preceding steps block the services for which we define the rule in the `/etc/hosts.allow` file. However, on the server end, we don't get to find out which client has tried to access the server and when. So, if we want to maintain a log of all connection attempts by the client, we can edit the `/etc/hosts.allow` file, as follows:

![](img/f6f740e0-b32d-4dac-8b7f-03b57c449018.png)

In the preceding line, the `spawn` keyword defines that whenever a connection request is made by the client, it will echo the details, as specified by the `%h` option, and save it in the log file, `conn.log`.

13.  Now, when we read the contents of the `conn.log` file, we see its details, as shown here:

![](img/cb6ca3d8-bddf-4cf3-aaa3-a4463f09e4af.png)

The file contains a log of when the client has tried to connect and from which IP address. More details can be captured by using different arguments of the `spawn` command.

# How it works...

We use TCP Wrappers to restrict access to programs that are supported by the TCP wrapper package. We first check if the program we want to restrict is supported by TCP Wrapper or not by using the `ldd` tool. We then add a rule in the `/etc/hosts.allow` or `/etc/hosts.deny` file as per our requirements.

Afterwards, we add rules to restrict the program from a particular client or the complete network, as per our choice. Using the `spawn` option in the TCP Wrapper, we even maintain a log for the connection attempts made by the client or program that we have restricted.

# Blocking country-specific traffic using mod_security

ModSecurity is a web application firewall that can be used for Apache web servers. It provides logging capabilities and can monitor HTTP traffic in order to detect attacks. ModSecurity can also be used as an intrusion detection tool, where we can use it to block country-specific traffic as per our requirements.

# Getting ready

Before we start with the installation and configuration of `mod_security`, we require Apache server installed on our Ubuntu system.

To install Apache on Ubuntu, run the following command:

```
apt-get update
apt-get install apache2
```

# How to do it...

In this section, we will see how to install and configure the ModSecurity **Web Application Firewall** (**WAF**) to block country-specific traffic:

1.  Once Apache has been installed on Ubuntu, the next step is to install ModSecurity by running the following command:

![](img/d8fb711a-4653-4d3c-a45d-461afaa09ee6.png)

2.  After installing ModSecurity, restart Apache:

![](img/dbbf452b-1a1b-45fc-a0d7-4dd50effec81.png)

3.  To confirm that ModSecurity has been installed successfully, run the following command:

![](img/8ea629dd-c7d9-4604-ae40-71f7377f2581.png)

If the installation is successful, we should see something like this: `security2_module (shared)`, as shown in the preceding screenshot.

4.  After completing the installation, we start configuring ModSecurity. For this, we use the pre-included and recommended configuration file—`modsecurity.conf-recommended`—which is located in the `/etc/modsecurity` directory.

5.  Rename the `modsecurity.conf-recommended` file, as follows:

![](img/9c00eb34-cb47-4d0e-b0fd-c4af3b720a7d.png)

6.  After renaming the file, we edit the `modsecurity.conf` file and change the value for `SecRuleEngine detectiononly` to `SecRuleEngine` on:

![](img/8cd4b0c2-c911-423e-a5f2-46d32a95a5a9.png)

7.  After saving these changes, restart Apache:

![](img/a983c38d-9619-4277-aa72-fd944e7296db.png)

8.  ModSecurity comes with many **Core Set Rules** (**CSR**). However, we can download the latest OWASP ModSecurity CRS from GitHub by using the following command:

```
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
```

9.  Once downloaded, move into the downloaded directory. Next, move and rename the `crs-setup.conf.example` file to `/etc/modsecurity/`, as follows. Move the `rules/` directory to `/etc/modsecurity/` as well:

![](img/84bb3db3-5429-4bdf-95c0-e9cecfe170d2.png)

10.  Now, edit the Apache configuration file, `/etc/apache2/mods-available/security2.conf`, and add the `Include` directive to point to the rule set, as follows:

![](img/9709f983-4688-4286-84d8-8014648c672e.png)

11.  Restart Apache again to reflect the changes:

![](img/ba421a46-f927-4bd5-b573-702e6df05107.png)

12.  ModSecurity supports the use of geolocation data by integrating with the free Maxmind database.
13.  To block country-specific traffic, we first have to download the geolocation database on the same server where we have configured ModSecurity. To download the database, use the following command:

![](img/bceea268-9dca-4c58-8e4d-fa8b5fc394c6.png)

14.  After completing the download, extract and move the file to `/usr/share/GeoIP/`.

15.  The next step is to edit the `/etc/modsecurity/crs-setup.conf` file to enable the use of the geolocation database. For this, we enable the `SecGeoLookupDb` directive and also define the path to the downloaded GeoIP database file:

![](img/9de28e17-b0db-4e64-8ccd-6a25f9f962c1.png)

16.  Next, we need rule for blocking traffic from any country. The configuration file has an example rule for reference, which can be used by uncommenting the rule:

![](img/efc07ca3-3779-43bc-9201-7359ffdd76e4.png)

17.  After uncommenting the rule and adding the country code of the country we want to block, as shown in the following screenshot, we can save the file:

![](img/bd045faa-ffa9-4dd1-a1ed-b93698c7ec4d.png)

18.  If we want our server to be accessible only from a specific country and block the traffic from all other countries, we can create a rule such as the following:

```
SecRule *GEO:COUNTRY_CODE3* "***!@streq USA***" "phase:1,t:none,log,deny,msg:'Client IP not from USA'"
```

In this way, we can use ModSecurity to block or allow country-specific traffic.

# Securing network traffic using SSL

**TLS** and **SSL** are secure protocols and have been developed to put normal traffic in a protected, encrypted wrapper. With the help of these protocols, traffic can be sent between remote users in an encrypted format, thus protecting the traffic from being intercepted and read by anyone else. These certificates form an essential component of the data encryption process and help in making internet transactions secure.

# Getting ready

Before we start with the creation and configuration of the self-signed certificate, we require Apache server to be installed on our Ubuntu system.

To install Apache on Ubuntu, run the following command:

```
apt-get update
apt-get install apache2
```

# How to do it...

In this section, we will learn how to create a self-signed certificate to encrypt traffic to our Apache server:

1.  Once our Apache server has been installed, we can check the default web page by visiting the `http://localhost` link on our browser:

![](img/7010cac6-c5d8-4c49-ba6e-3817889c14e5.png)

2.  However, when we try to access the same page using HTTPS, we get the following error:

![](img/fd2149db-9c21-4f4f-8b06-611169e3d725.png)

3.  To start using SSL, we have to enable the SSL support module on our Ubuntu server. To do this, we must run the following command:

![](img/0f93ca00-595d-4d54-b854-2d21be87ba87.png)

4.  After enabling the SSL module, restart the Apache server so that the changes can be recognized:

![](img/ef364f1a-9dac-4143-a031-32eb1f964258.png)

5.  Now, we will proceed to create our self-signed certificate. First, create a subdirectory within Apache's configuration hierarchy. The certificate file that we will be creating will be placed here. Run the following command:

![](img/e1607766-d3b0-4874-a245-f071314c7db3.png)

6.  Now, we will create our key and the certificate file and place them in the directory that we created in the previous step. To create the key and certificate file, we can use the `openssl` command, as follows:

![](img/267b87d4-96a1-43d7-81fe-0d852c82277b.png)

For more details about the options used with the `openssl` command, it is recommended to check the manual of the command.

7.  When we execute the previous command, we will be asked a few questions. Enter the required details to complete the process of certificate creation:

![](img/b7d1842c-ff68-43e7-8a54-813564321c8e.png)

Among these questions, the most important is the `Common Name` (the server's `FQDN or YOUR name`). Here, we have to enter the domain name that should be associated with the certificate. The server's public IP can also be used if we don't have a domain name.

8.  Once the command is complete, we can check the `/etc/apache2/ssl` directory that we created earlier. We will see that the key and the certificate file have been placed here:

![](img/f33e8fb6-c41e-4aeb-930d-b038b7f20a85.png)

9.  After creating the key and certificate file, the next step is to configure Apache to use these files. We will base our configuration on the `etc/apache2/sites-available/default-ssl` file as it contains the default SSL configuration.

10.  Edit the file and set the values that we would configure for the virtual host. This includes `SeverAdmin`, `ServerName`, `ServerAlias`, and so on. Enter the details as follows:

![](img/50dba488-853a-41d9-a350-5f6a7acfd7d1.png)

11.  In the same file, we have to define the location where Apache can find the SSL certificate and key, as shown in the following screenshot:

![](img/922c1f90-8445-4985-8826-9d4735bb0a3d.png)

12.  Once the preceding configuration has been done for the SSL-enabled virtual host, we have to enable it. To do this, we run the following command:

![](img/1ff4fe0a-25fc-4c78-98d9-07da318569b1.png)

13.  To load the new virtual host file, restart Apache for the changes to take effect.

14.  After completing all the setup, we can now test our configuration by visiting our server's domain name by using the HTTPS protocol. Once we enter the domain name in the browser with `https` and press *Enter*, we will get the following screen:

![](img/8b9badd9-7b88-4ee7-b945-008d630cbebc.png)

This confirms that the certificate is being loaded. Since it is a self-signed certificate, we get a warning message that states This Connection is Untrusted. If the certificate had been signed by one of the certificate authorities that the browser trusts, this warning would have not appeared.

15.  Once we click on I Understand the Risks and proceed, we will be asked to add a security exception, as shown in the following screenshot. Confirm the security exception to add our self-signed certificate in the browser:

![](img/d3be1679-8b1f-4b37-842a-f0bf7948fe01.png)

We now have SSL enabled on our web server. This will help secure the traffic between the users and the server.

# How it works...

SSL certificates are digital passports that provide authentication to protect the confidentiality and integrity of communication between the server and the browsers. SSL certificates have a key pair: a public and a private key, which work together to establish an encrypted connection.

When a browser attempts to connect to a web server secured with SSL, it asks the web server to identify itself. Then, the server sends a copy of its SSL certificate. After receiving the certificate, the browser sends a message to the server. After this, the server sends back a digitally signed acknowledgement that tells the server to start an SSL encrypted connection. Now, all the data being shared between the server and the browser is encrypted.