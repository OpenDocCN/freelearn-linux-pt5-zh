# 13

# Configuring Linux Servers

In this chapter, you will learn how to set up different types of Linux servers. This will include **Domain Name System** (**DNS**) servers, **Domain Host Configuration Protocol** (**DHCP**) servers, **Samba** or **Server Message Block**/**Common Internet File System** (**SMB**/**CIFS**) file servers, and **Network File System** (**NFS**) servers. All these servers, in one way or another, are powering the backbone of the World Wide Web. Even though we will not cover it here, you should know that the reason your computer is showing the exact time is because of a well-implemented **Network Time Protocol** (**NTP**) server. You can shop online and transfer files between your friends and colleagues thanks to effective DHCP, web, and file servers. Configuring the different types of Linux services that power all these servers represents the knowledge base for any Linux system administrator. In this edition of the book, we will only cover a select few of these Linux servers, those we consider the most important and major Linux servers currently. For further information, please refer to the *Further reading* section at the end of this chapter.

In this chapter, we’re going to cover the following main topics:

*   Introduction to Linux services
*   Setting up an SSH server
*   Setting up a DNS server
*   Setting up a DHCP server
*   Setting up an NFS server
*   Setting up a Samba file server

# Technical requirements

Basic knowledge of networking and Linux commands is required. You will need access to multiple working systems, preferably on premise or in the cloud. If this is not possible, you can use local virtual machines on your system. Furthermore, it would be useful to have a domain available for you to use too. We will use Ubuntu Server 22.04.2 LTS as the distribution of choice for this chapter’s exercises and examples. Nevertheless, any other major Linux distribution—such as Fedora, RHEL, openSUSE, or Debian—is equally suitable for the tasks detailed in this chapter.

# Introducing Linux services

Everything you have learned up until now will easily apply to any workstation or desktop/laptop running Linux. We have delved into advanced networking subjects that are meant to ease your learning path to becoming a seasoned Linux system administrator. Now we will enter the *server* territory as a natural path to the *cloud*, which will be discussed in detail in the last four chapters of this book.

A **Linux server**, compared to a Linux workstation, is a system that serves content over a network. While doing so, a server provides its hardware and software resources to different clients that are accessing it. For example, every time you enter a website address into your browser, a server is accessed. That particular type of server is a **web server**. When you print over the network in your workplace, you access a **print server**, and when you read your email, you access a **mail server**. All these are specialized systems that run a specific piece of software (sometimes called a service) that provides you, the client, with the data you requested. Usually, servers are very powerful systems that have lots of resources available for a client’s use.

In contrast, a **workstation** (which is yet another powerful piece of hardware) is generally used for personal work, not for client access over a network. A workstation is used for intensive work, similar to any regular desktop or laptop system. In light of everything we have exposed up to now, the contents of this chapter and the following chapters are best suited for server use, but not limited to it.

You’ve probably already heard about setting up different Linux servers—such as a web server, a file server, or an email server—and have probably wondered why they are called that. They do not represent the hardware boxes that are the actual servers—they are basically services running on top of Linux. So what are Linux services? These are programs that run in the background. Inside the Linux world, those services are known as `init` process in [*Chapter 5*](B19682_05.xhtml#_idTextAnchor104), *Working with Processes, Daemons, and Signals*, when we discussed what processes, daemons, and signals are and how to manage them on Linux. The mother of all processes is the `init` process, which is among the first processes when Linux boots up. Currently, the latest version of Ubuntu (and also CentOS, Fedora, openSUSE, and others) uses `systemd` as the default `init` process.

We will refresh your memory by using some basic commands for working with services on Linux. If you want more information, please refer to [*Chapter 5*](B19682_05.xhtml#_idTextAnchor104), *Working with Processes, Daemons,* *and Signals*.

The first command we will remind you of is the `ps` command. We will use it to show the `init` process running, as follows:

```
ps -ef | less
```

The output on our Ubuntu 22.04.2 system is shown in the following screenshot:

![Figure 13.1 – Showing the init process by using the ps command](img/B19682_13_01.jpg)

Figure 13.1 – Showing the init process by using the ps command

By using the `-e` flag, we can generate information about all processes, except kernel-related ones, while the `-f` flag is used to generate a full listing.

In the process list shown, the first process is either the `init` process or `systemd`. Sometimes, on older operating systems (such as Ubuntu 20.04 or 18.04), it uses the `init` name for backward-compatibility issues, but to make sure that it really is `systemd`, you can use the manual pages of `init` for more details. When you type `man init` in the command line, the manual page shown is for `systemd`. As the parent of all services, `systemd` starts all the running processes in parallel as a way to make the boot process and service-time response more efficient. To see how efficient those processes are, you can run the `systemd-analyze` command.

A command we have frequently used, and one you should already know, is the `systemctl` command, which is the main command-line utility for working with `systemd` services (or daemons) on Linux.

As you know `systemctl` invokes `systemd`. Those units have several types, such as service, mount, socket, and others. To see those units listed by the time they take to start up, you can use the `systemd-analyze blame` command, as shown in the following screenshot:

![Figure 13.2 – The systemd-analyze blame command](img/B19682_13_02.jpg)

Figure 13.2 – The systemd-analyze blame command

The output shows different types of units, such as *service*, *mount*, and *device*. The preceding screenshot is only an excerpt of the running units. To learn more about the `systemctl` command, feel free to use the manual pages, or go back to [*Chapter 5*](B19682_05.xhtml#_idTextAnchor104) to refresh your memory.

This brief introduction to Linux services is only a refresher of the respective sections from [*Chapter 5*](B19682_05.xhtml#_idTextAnchor104), enough for you to start delving into setting up and configuring specific Linux services. In the following sections, we will show you how to manage some of the most important services on Linux, such as **SSH**, **DNS**, **DHCP**, **NTP**, **Samba**, **NFS**, **web**, **File Transfer Protocol** (**FTP**), and **printing services**. Now, it’s time to roll up your sleeves and configure them yourself. First, we will show you how to set up an SSH server on Ubuntu Linux.

# Setting up SSH

We will configure SSH on a computer running Ubuntu Server 22.04.2 LTS as the host operating system. Throughout the entire book, we used SSH connections several times, and showed you how to create an SSH key pair in [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231), *Working with Virtual Machines*, when we worked with cloud-init. This time, we will show you how to install **OpenSSH**, how to enable SSH, and how to modify some of its configuration defaults.

## Installing and configuring OpenSSH on Ubuntu

In order to use SSH, the first thing we need to do is to install the `openssh` package. On Ubuntu, this can be done by using the following command:

```
sudo apt install openssh-server
```

Chances are that it is already installed on your system. If that is the case, you can go ahead and skip this step and go to the configuration files.

After installation, we can start and enable the `openssh` service with the following commands:

```
sudo systemctl enable ssh && sudo systemctl start ssh
```

For openSSH, the configuration file that we can work with is located under `/etc/ssh/sshd_config`. By default, this file already contains a lot of information, all we need to do is to open it with our text editor and start modifying the available options. Depending on what we want to achieve, the bare minimum configuration for SSH involves the following:

*   Modify the remote root login options; this can be done by changing the line with the following code and setting the option accordingly:

    ```
    PermitRootLogin no
    ```

    In our case, we modified the default `prohibit-password` option to `no`, so that the root user will not be able to connect through SSH at all.

*   Disable the SSH password authentication. This can be done by changing the following lines to `no`:

    ```
    PasswordAuthentication no
    PermitEmptyPasswords no
    ```

    Do this only after you copy your key pair on the remote machine and make sure you can use it. Otherwise, you won’t be able to access your server.

*   To allow public key authentication, you will need to uncomment the following line:

    ```
    PubkeyAuthentication yes
    ```

*   We will quickly show the commands used in [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231) to enable public keys authentication, as a reminder:

    ```
    ssh-keygen
    user, use the username from the remote server, and instead of host_IP, use the IP address of the remote server. Also, ssh-keygen can be used with several options, such as -b to specify the number of bits and -t to specify the algorithm type. After this, you can connect with the ssh user@IP command, which in our case is the following:

    ```
    ssh packt@192.168.0.113
    ```

    ```

The configuration options presented here are the base minimum to start using SSH when working with a remote system or a virtual machine. However, OpenSSH is a very powerful tool that offers lots of options that you could explore. Here are two links that could help you in your endeavors: [https://ubuntu.com/server/docs/service-openssh](https://ubuntu.com/server/docs/service-openssh) and [https://www.openssh.com/](https://www.openssh.com/)manual.html. In the following section, we will show you how to set up a DNS server.

# Setting up a DNS server

One of the most widely used DNS services is **Berkeley Internet Name Domain 9** (**BIND 9**). You can visit its official website at the following address: [https://www.isc.org/bind/](https://www.isc.org/bind/). Before continuing, let’s underline the system configuration and goals. For this section, we will use a computer running on Ubuntu Server 22.04.2 LTS. On this system, we will create two types of servers, a **caching name server** and a **primary name server**, which you can use on your local network to manage hostnames and private IP addresses.

Important note

There are different DNS servers, such as **authoritative**, **caching**, or **forwarding** types; those are also called functional types. Among those, a caching DNS server is the one that always answers recursive requests from clients. Also, there are **relational server types**, such as primary and secondary DNS servers. Those are authoritative types, and are almost identical, with the only difference between the primary and the secondary being the location from which they get the zones’ information. For more information on DNS, you can consult the following links: [https://www.digitalocean.com/community/tutorials/a-comparison-of-dns-server-types-how-to-choose-the-right-dns-configuration](https://www.digitalocean.com/community/tutorials/a-comparison-of-dns-server-types-how-to-choose-the-right-dns-configuration) and [https://www.digitalocean.com/community/tutorials/an-introduction-to-dns-terminology-components-and-concepts](https://www.digitalocean.com/community/tutorials/an-introduction-to-dns-terminology-components-and-concepts).

There are other ways to do this, but for the purpose of showing you the basics of DNS setup, this configuration will suffice. If you would like a secondary server, you will need to have another spare system, or if you use **virtual private servers** (**VPSs**), they would have to be in the same data center and be using the same private network. In our case, however, we will use a local system on our small private network.

First, we will install the `bind9` package in Ubuntu by using the following command:

```
sudo apt install bind9 bind9utils bind9-doc
```

The preceding command will install all the packages needed for BIND9 to run.

Once the packages are installed, you can test them to see whether BIND works as expected. For this, we will use the `nslookup` command, as shown in the following screenshot, by using the local address (or *loopback* address):

![Figure 13.3 – Checking to see whether BIND is working using nslookup](img/B19682_13_03.jpg)

Figure 13.3 – Checking to see whether BIND is working using nslookup

You can now start setting up the service, as you can see it is working. First, we will configure a caching DNS service. But before we start, we advise you to back up the following configuration files: `/etc/bind/named.conf`, `/etc/bind/named.conf.options`, `/etc/hosts`, and `/etc/resolv.conf`. Let’s see how to create a caching server.

## Caching a DNS service

The default behavior of BIND9 is as a caching server. This means that setting it up is quite straightforward. We will tweak the configuration file just a little, in order to make it work according to our requests:

1.  After seeing that the installed packages work as intended, you can also configure the firewall to allow BIND9, using the following command:

    ```
    /etc/bind/named.conf.options file to add or delete different options. We will do that by opening it with the text editor. Inside the file, by default, are a few settings that have been set up, and a lot of commented lines with details on how to use the file. The // double slashes indicate that the respective lines are commented out. All the modifications will be done inside the options directive, between curly brackets, as it is the only existing directive by default.The first thing to do, as we will only use **IP version 4** (**IPv4**), is to comment out the following line by adding two slashes:

    ```
    // listen-on-v6 { any; };
    ```

    ```

2.  You will have to add a list of IP addresses inside the `forwarders` directive. This line tells the server where to look in order to find addresses not cached locally. For simplicity, we will add the Google public DNS servers, but feel free to use your `forwarders` directive to look like this:

    ```
    forwarders {
          8.8.8.8;
          8.8.4.4;
            };
    ```

3.  You can also add a directive defining the `allow-query` spectrum. This line tells the server which networks can be accepted for DNS queries. You can add your local network address. In our case, it will be the following:

    ```
    allow-query {
          localhost;
          192.168.0.0/24;
            };
    ```

4.  There is also a `listen-on` directive, where you can specify the networks the DNS server will work for. This applies for IPv4 addresses and is shown in the following code snippet:

    ```
    listen-on {
            192.168.0.0/24;
            }
    ```

    After the added options, the code you added should appear as in the following screenshot:

![Figure 13.4 – The final form of /etc/bind/named.conf.options after adding new options](img/B19682_13_04.jpg)

Figure 13.4 – The final form of /etc/bind/named.conf.options after adding new options

1.  Save the file and exit the editor. You can check the BIND9 configuration with the `named-checkconf` command. If there is no output, it means that the configuration of the file is correct. Restart the BIND9 service and optionally check its status.
2.  Knowing that the BIND9 service is working fine, you can test the service from any other computer on the network with the `nslookup` command, as follows:

![Figure 13.5 – Testing the BIND9 implementation](img/B19682_13_05.jpg)

Figure 13.5 – Testing the BIND9 implementation

We used the command with the host’s IP address, as seen in the previous screenshot. The output shows that the DNS service on our test machine is working fine. The test was conducted from a local ThinkPad on the same network.

You now have a working caching DNS server on your private network. In the next section, we will show you how to create a primary DNS server.

## Creating a primary DNS server

In order to configure a primary DNS server, we will need a domain name that it will serve. For this section’s purpose, we will use the `calcatinge.ro` domain name (when you try on your system, please use a domain name you own). We will have to create new zones for the BIND9 configuration, and we will add information to the `/etc/bind/named.conf.local` file about the ones we create. Right now, we will create a new zone for our `calcatinge.ro` domain.

Important note

What is a **DNS zone**? The short answer is that it is a part of a domain namespace associated with an entity responsible for maintaining it. Zones also offer a granular take on administrating different components. For more information on DNS zones, please refer to the following link: [https://ns1.com/resources/dns-zones-explained](https://ns1.com/resources/dns-zones-explained).

In the following screenshot, you can see the contents of our configuration file, followed by details on each of the lines:

![Figure 13.6 – New zone for our domain in /etc/bind/named.conf.local file](img/B19682_13_06.jpg)

Figure 13.6 – New zone for our domain in /etc/bind/named.conf.local file

Now, let’s explain the contents of a zone directive:

*   First, we had to add the name of the domain that the zone will serve
*   The `type` of the zone is set as `master`, but there are other types to use, such as `slave`, `forward`, or `hint`
*   The `file` represents the path to the actual zone file that will be created
*   In the `allow-transfer` list, the IPs of DNS servers that handle the zone are set
*   In the `also-notify` list, the IPs of servers that will be notified about zone changes are indicated

Now, let’s create a new zone for our chosen domain:

1.  The following screenshot shows how we copied the `db.local` file under another name and used it for a new zone:

![Figure 13.7 – Creating the zone file for our domain](img/B19682_13_07.jpg)

Figure 13.7 – Creating the zone file for our domain

1.  The next step is to create the zone file, as indicated in the zone directive about `calcatinge.ro`. The details are shown in the preceding screenshot and as you can see, its location is `/etc/bind/`. Once the file is created using `db.local` as a template, you can open it with your favorite text editor and add information about your server IP and domain name. In the following screenshot, you can see the zone file for `calcatinge.ro` created on our machine:

![Figure 13.8 – Zone file information](img/B19682_13_08.jpg)

Figure 13.8 – Zone file information

1.  The DNS records are introduced at the end of the file. Here are some details about the contents of the file:
    *   The table has a specific format that contains details about hostname (first column), class (second column), DNS record type (third column), and value (the last column).
    *   For the hostname, we entered `@`, which means that the entry of the record refers to the zone name from the file.
    *   The class is `IN`, which indicates that the network is the internet.
    *   DNS records types are `A`, `NS`, `MX`, `CNAME`, `TXT`, and `SOA`. `A` indicates the IP address of the domain name; `NS` indicates the IP address of the DNS server; `MX` is the address of the email server; `CNAME` is an alias (canonical name); and `TXT` has a custom entry, `SOA`, which indicates the authoritative name server for the zone, with details on the administrator, serial number, and refresh rates.
    *   The value in the last column most often comprises of the IP address or the hostname.
2.  The next step is to restart the **Remote Name Daemon Control** (**RNDC**), which is a control utility inside BIND that controls the name server. The command to do that is shown here:

    ```
    sudo rndc reload
    ```

3.  Now, you can check to see if the primary DNS server works. Try the `nslookup` command from another system on the network, as follows:

    ```
    nslookup calcatinge.ro 192.168.0.113
    ```

    The output of the preceding command will most certainly show that the local DNS server on the system with the indicated IP address has a working zone file. Your primary DNS server works as expected on your local network. Don’t forget to use your IP inside the command.

It is always a good idea to create a secondary DNS server in case the first one stops working, which is why we will show you how to set up a second one in the following section.

## Setting up a secondary DNS server

It comes as no surprise that the secondary DNS server should be set up on different hardware from the primary one, but on the same network. If you do it inside a data center, use a VPS in the same network as the first one. If you plan to experiment with it on your home private network, make sure you have another system at your disposal.

We will start another NUC system available that is also running Ubuntu Server 22.04.2 LTS. We will need to know its IP address in order to use it in our configuration. In our case, the IP of the new system is `192.168.0.140`. This second machine needs to have BIND9 installed and configured too. Before setting up the secondary server, you will need to modify the configuration of the primary DNS server first.

### Modifying the primary server configuration files

To modify the configuration of the primary DNS server and allow it to send the zone details to the secondary server, follow these steps:

1.  You will have to open the `/etc/bind/named.conf.local` configuration file and add some new lines to it. We add the second server’s IP address inside the `allow-transfer` and `also-notify` directives, as shown in the following screenshot:

![Figure 13.9 – Adding the IP of the secondary DNS server](img/B19682_13_09.jpg)

Figure 13.9 – Adding the IP of the secondary DNS server

1.  Save the file and restart the BIND9 service.
2.  You will also need to open the `/etc/bind/named.conf.options` configuration file and add an access list parameter (`acl "trusted"`) with all the accepted IP addresses on the network. In our case, the primary server has the address `192.168.0.113`, and the secondary server has the address `192.168.0.140`. Add this before the already existing `options` directive.
3.  Inside the `options` directive block, below the comments, we also add the following directives:

![Figure 13.10 – Adding new directive inside /etc/bind/named.conf.options file](img/B19682_13_10.jpg)

Figure 13.10 – Adding new directive inside /etc/bind/named.conf.options file

1.  The configuration is finished, and we will restart the BIND9 service using the `systemctl` command, as follows:

    ```
    sudo systemctl restart bind9.service
    ```

Before proceeding further, let’s understand the directives used. The `recursion` directive has a boolean value (`yes` | `no`) and defines whether recursion and caching are allowed or not on the server. The default is `yes`, and thus the server will require DNS query recursion by solving all the attempts.

Important note

**Recursion** is also known as **recursive query**, which in the case of DNS refers to the way a name resolution is solved. The recursive mode represents the way a computer looks for a FQDN by inquiring first at the local cache data and the local DNS server. The request is clear and demands a precise answer, and the solution for that answer is the responsibility of the DNS server. Thus, the query initiated by the computer, which is a DNS client, to the DNS server is a recursive query.

The `allow-recursion` directive is referenced to a list of matching addresses of clients, in our case specified by a `listen-on` directive specifies the IP on which the server listens. And we also used the `allow-transfer` directive that provides a list of hosts that are allowed to transfer zone information (in our case, none). For more information about configuration options, please refer to the following link: [https://bind9.readthedocs.io/en/latest/reference.html#](https://bind9.readthedocs.io/en/latest/reference.html#).

Next, let’s learn how to configure the secondary server.

### Setting up the secondary server

On the secondary server, as stated earlier, you will need to install BIND9 too. Once it is installed, you will need to follow these steps:

1.  Go to the `/etc/bind/named.conf.options` file and add the following lines in an `acl` directive, before the already existing `options` directive:

![Figure 13.11 – Adding an acl directive on the secondary server](img/B19682_13_11.jpg)

Figure 13.11 – Adding an acl directive on the secondary server

1.  Also, add the following lines inside the `options` directive:

![Figure 13.12 – Adding new directives inside options](img/B19682_13_12.jpg)

Figure 13.12 – Adding new directives inside options

1.  Now edit the `/etc/bind/named.conf.local` file and add the zones you want, but this time use the `secondary` type, as opposed to the `master` one used on the primary DNS server.

    The following is a comparison between the two `/etc/bind/named.conf.local` files. On the left is the file on the primary DNS server, and on the right is the file on the secondary DNS server:

![Figure 13.13 – Configuration files from primary (left) and secondary (right) servers](img/B19682_13_13.jpg)

Figure 13.13 – Configuration files from primary (left) and secondary (right) servers

1.  Now we can restart the BIND9 service and make sure that the firewall is allowing DNS connections on the second server with the following commands:

    ```
    sudo ufw allow Bind9 && sudo systemctl restart bind9
    ```

Now you have two DNS servers set up and working, one primary and one secondary. In the following section, we will show you how to set up a local DHCP server.

# Setting up a DHCP server

DHCP is a network service that is used to assign IP addresses to hosts on a network. The settings are enabled by the server, without any control from the host. Most commonly, the DHCP server provides the IP addresses and netmasks for clients, the default gateway IP, and the DNS server’s IP address.

To install the DHCP service on Ubuntu, use the following command:

```
sudo apt install isc-dhcp-server
```

As a test system, we will use the same system on which we installed the DNS services in the previous section. After installation, we will configure two specific files. On an Ubuntu system, like ours, the default configuration will be set inside the `/etc/dhcp/dhcpd.conf` file, while the interfaces will be configured inside the `/``etc/default/isc-dhcp-server` file:

1.  We will first show you how to set up a basic local DHCP server. In this respect, we will alter the `/etc/dhcp/dhcpd.conf` file by adding the IP address pool. You can either uncomment one of the `subnet` directives already available inside the file, or you can add a new one, which is what we will do. Our existing subnet is `192.168.0.0/24`, and we will add a new one for this new DHCP server, as shown in the following screenshot (for a refresher on networking, please refer to [*Chapter 7*](B19682_07.xhtml#_idTextAnchor139), *Networking* *with Linux*):

![Figure 13.14 – Defining a new subnet in /etc/dhcp/dhcpd.conf](img/B19682_13_14.jpg)

Figure 13.14 – Defining a new subnet in /etc/dhcp/dhcpd.conf

1.  Inside the same `/etc/dhcp/dhcpd.conf` file, you can uncomment the line that says `authoritative;`. The `authoritative` DHCP clause ensures that the server will automatically resolve any invalid IP numbers on the network and assign a new and valid IP to each new device registered without requiring the user’s manual interaction.
2.  Once the options are modified, you must specify the network interface name in the `/etc/default/isc-dhcp-server` file. This is needed for the server to know which network device to use. To do this, open the file with your preferred editor and add the interface’s name. If you don’t remember your interface name, run the `ip addr show` command and select the appropriate interface. In our case, the system we use has both Ethernet and wireless interfaces, and we will choose the Ethernet interface for the DHCP server, which is `enp0s25`. Inside the `/etc/default/isc-dhcp-server` file, add the interface as follows:

    ```
    INTERFACESv4="enp0s25"
    ```

3.  Then, save the changes to the file and restart the DHCP service with the following command:

    ```
    sudo systemctl restart isc-dhcp-server.service
    ```

Now, you have a working DHCP server on your system of choice. A DHCP server gives you some advantages in managing your local network, but there are times when you might not need to create a new one, as all the network routers provide a fully working DHCP service right out of the box.

Important note

To avoid conflicts, you might want to isolate the new DHCP server from your local network router. Your network router already has a fully functional DHCP server that will most likely conflict with the new one. In most cases, `isc-dhcp-server.service` will give you an error as an indication that it was not able to connect to any interface.

Taking the preceding note into consideration, when we check to see whether the DHCP service is running with the following command:

```
sudo systemctl status isc-dhcp-server.service
```

We received an error, as shown in the following screenshot:

![Figure 13.15 – Error running the DHCP service](img/B19682_13_15.jpg)

Figure 13.15 – Error running the DHCP service

Once you isolate the machine from your local network and use it as a single DHCP server, the service will be running as intended.

In the next section, we will show you how to set up an NFS server on your local network.

# Setting up an NFS server

NFS is a distributed filesystem used to share files over a network. To show you how it works, we will set up the NFS server on one of our machines on the network. We will use Ubuntu 22.04.2 LTS as the base for the NFS server. For more in-depth theoretical information about NFS, please refer to [*Chapter 7*](B19682_07.xhtml#_idTextAnchor139), *Networking* *with Linux*.

The NFS filesystem type is supported by any Linux and/or Unix environment and also by Windows, but with some limitations. For mostly-Windows client environments, we recommend using the Samba/**Common Internet File System** (**CIFS**) protocol instead. Also, for those of you concerned about privacy and security, please keep in mind that the NFS protocol is not encrypted, thus any transfer of data is not protected by default.

## Installing and configuring the NFS server

On our network, we will use an Ubuntu machine as a server and we will show you how to access the files from another Linux client. First, let’s install and configure the server, as follows:

1.  We will install the `nfs-kernel-server` package using the following command:

    ```
    systemctl command, and then we will check its status. The commands are the following:

    ```
    sudo systemctl start nfs-kernel-server.service
    sudo systemctl enable nfs-kernel-server.service
    /home directory for all clients on the network, or you could make a dedicated shared directory from the start. You could create a new directory starting from the root, but you could also create your shared directory inside specific directories such as /var, /mnt, or /srv—it’s your choice.We will create a new directory called `/home/export/shares` inside our `/home` directory using the following commands (make sure that you are already inside your `/home` directory if you want to use the command as is):

    ```
    777 as we will not use Lightweight Directory Access Protocol (LDAP) authentication on the following example:

    ```
    /etc/exports file. There are three configuration files for NFS (/etc/default/nfs-kernel-server, /etc/default/nfs-common, and /etc/exports) but we will only alter one of them. In the following screenshot, you will see the two files inside /etc/default and the default contents of the /etc/exports configuration file:
    ```

    ```

    ```

    ```

![Figure 13.16 – The configuration files inside /etc/default and the contents of the /etc/exports configuration file](img/B19682_13_16.jpg)

Figure 13.16 – The configuration files inside /etc/default and the contents of the /etc/exports configuration file

Open the `/etc/exports` file with your preferred text editor and edit it according to your configuration. Inside the file, we will add lines for each shared directory. Before doing that, you might have noticed that inside the file there are two types of directives: one for NFS versions 2 and 3, and one for version 4\. For more details about the differences between those versions, we encourage you to consult the following document: [https://archive.fosdem.org/2018/schedule/event/nfs3_to_nfs4/attachments/slides/2702/export/events/attachments/nfs3_to_nfs4/slides/2702/FOSDEM_Presentation_Final_pdf.pdf](https://archive.fosdem.org/2018/schedule/event/nfs3_to_nfs4/attachments/slides/2702/export/events/attachments/nfs3_to_nfs4/slides/2702/FOSDEM_Presentation_Final_pdf.pdf).

1.  Now let’s start editing the configuration file. We will add a new line containing the directory, the IP address of the client, and configuration options. The general syntax is as follows:

    ```
    /first/path/to/files  IP(options)[ IP(options) IP(options)]
    /second/path/to/files IP(options)[ IP(options) IP(options)]
    ```

    We share only one directory, hence the single line added. If you want all clients on the network to access the shares, you can add the subnet class (for example, `192.168.0.0/24`). Or, if you want only specific clients to access the shares, you should add their IPs. More clients will be added on the same line, separated by spaces. There are many options that you can add, and for a full list, we advise that you consult [https://linux.die.net/man/5/exports](https://linux.die.net/man/5/exports) or the local manual file with the `man exports` command. In our file, we added the following options:

    *   `rw` for both read and write access
    *   `sync` to force write changes to disk (this reduces the speed, though)
    *   `no_subtree_check` to prevent subtree checking, which is mainly a check to see whether the file is still available before the request

    In a nutshell, here is the line we added to the file (please take into consideration that this is only a single line, with a space between `/home/export/shares` and the IP, and there is no space between the IP and the parentheses):

    ```
    /home/export/shares   192.168.0.0/24(rw,sync,no_subtree_check)
    ```

2.  After saving and closing the file, restart the service with the following command:

    ```
    sudo systemctl restart nfs-kernel-server.service
    ```

3.  Then, apply the configuration with the following command:

    ```
    -a options export all directories without specifying a path.
    ```

4.  Once the service is restarted and running, you can set up a firewall to allow NFS access. For this, it is extremely useful to know that the port NFS is using port `2049` by default. As we allow all the systems from our network to access the shares, we will add the following new rule to the firewall:

    ```
    sudo ufw allow nfs
    ```

5.  Now that the new firewall rule has been added, we can run the following command to make sure that it is running according to our needs:

    ```
    sudo ufw status
    ```

6.  If your firewall is not actively running, you can use the following command to activate it:

    ```
    ufw status command is shown in the following screenshot:
    ```

![Figure 13.17 – Showing the new firewall rule to allow NFS shares](img/B19682_13_17.jpg)

Figure 13.17 – Showing the new firewall rule to allow NFS shares

In the preceding screenshot, you can see that after adding the new rule using the `sudo ufw allow nfs` command, port `2049` was added to the list of allowed rules.

The basic configuration of the server is now complete. In order to access the files, you will need to configure the clients too. We will show you how to do this in the next section.

## Configuring the NFS client

As a client, we will use another system, a laptop running Debian GNU/Linux 12 Bookworm. First, we will have to install NFS on the client by using the following command:

```
sudo apt install nfs-common
```

Now that the needed packages are installed on the client too, we can create directories on the client to mount the shares to. We will create a new directory on the client, where the shares from the server will be mounted. This new directory will be `/home/shares`. We create it with the following command:

```
sudo mkdir /home/shares
```

Now that the new directory is created, we can mount the location from the server:

```
sudo mount 192.168.0.113:/home/export/shares /home/shares
```

With the preceding command, we mounted the shares from the server on the client using the `mount` command. We gave the location on the server as the first argument, followed by the location on the client as the second argument. We can also check to see whether everything went well by using the `df -h` command. The new mount is shown last in the `df` command’s output. The following is a screenshot showing the commands used to create, mount, and check the new shares directory:

![Figure 13.18 – Mounting the new shares directory on the client](img/B19682_13_18.jpg)

Figure 13.18 – Mounting the new shares directory on the client

At this point, we have finished the setup for the NFS shares. We now need to test the configuration to prove that it works.

## Testing the NFS setup

Once the setup is finished on both server and client, you can test to see if everything works according to your expectations. In our test, we created several files named `testing_files` using the `packt` user on the server, and a file called `file` using the regular user `alexandru` on the client machine. The following is the output showing the contents of the `/home/shares` directory on our Debian local system:

![Figure 13.19 – Testing the NFS on our local client](img/B19682_13_19.jpg)

Figure 13.19 – Testing the NFS on our local client

There you have it, the NFS server and client are working just fine. You can use the **graphical user interface** (**GUI**) on the client (as shown in the previous screenshot), and also the **command-line interface** (**CLI**) to access the NFS shares.

In the next section, we will show you how to configure a Samba/CIFS share that can be accessed by Windows clients on the network.

# Setting up a Samba file server

The Samba server allows you to share files over a network where clients use different operating systems, such as Windows, macOS, and Linux. In this section, we will set up a Samba server on Ubuntu 22.04.2 LTS and access shares from different operating systems on the network. The SMB/CIFS protocol is developed by Microsoft; for more details, you can visit their developer pages at [https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview](https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview). Some information about the SMB/CIFS protocol can be found in [*Chapter 7*](B19682_07.xhtml#_idTextAnchor139), *Networking with Linux*, too. In the following sub-sections we will show you how to install and configure it on your local network.

## Installing and configuring Samba

The installation procedure has the following steps:

1.  First, we will install Samba on the system using the following command:

    ```
    sudo apt install samba
    ```

2.  Once Samba is installed, we can check whether the service is running as expected. We will run the following command for this:

    ```
    /home directory using the mkdir command, as follows:

    ```
    /etc/samba/smb.conf. The configuration file has two major sections: a [global] section with general configuration settings, and a [shares] section that configures the shares’ behavior. A safe practice is to back up the original configuration file before starting to modify it. We will do this with the following command:

    ```
    smb.conf. Inside the configuration file, we will first create the [global] section with directives, as shown in the following screenshot:
    ```

    ```

    ```

![Figure 13.20 – The smb.conf global directive](img/B19682_13_20.jpg)

Figure 13.20 – The smb.conf global directive

Let us briefly explain the content shown in the preceding screenshot:

*   The first line sets a server name, in our case Local Samba File Server
*   The second line sets the `workgroup` name, in our case the Windows default name, `WORKGROUP`
*   The third line determines that it will not be necessary to have a Samba user account in order to access the shares (mapping the guest to `Bad User`)
*   Guest users are allowed on the fourth line
*   Then we set the interfaces used by Samba, which in our case will be the Ethernet connection (`eno1`) and the loopback (`lo`) interfaces (check your exact interface name with the `ip addr show` or `ip` `link` commands)
*   We will set the `server role` and the hosts allowed (from the local pool)
*   The last line sets the order the hostnames are checked, using the broadcast (`bcast`) method

1.  Now we will create the `[shares]` directive and add the necessary configuration options for the shares. We will add details about the shared directory on our server, user details, and permissions. See the following screenshot for details:

![Figure 13.21 – Adding information inside the shares.conf file](img/B19682_13_21.jpg)

Figure 13.21 – Adding information inside the shares.conf file

Let us briefly explain the contents of this directive:

*   We first set a name for the shares
*   Then we set the path of our local shared directory
*   Next, we set the default permissions for the directory and files
*   We also set the default mask values
*   Finally, we set the shares as public, guest-user friendly, and writable

1.  Once the modifications are done, we restart the service with the following command:

    ```
    sudo systemctl restart smbd.service
    ```

2.  Furthermore, we adjust the firewall rules to allow Samba using the following command:

    ```
    testparm command to test that the configuration has been done correctly, with no errors, as follows:
    ```

![Figure 13.22 – Testing the Samba configuration](img/B19682_13_22.jpg)

Figure 13.22 – Testing the Samba configuration

The output shows `Loaded services file OK`, which means that the configuration files have no syntax errors.

After the system is restarted and the firewall configured, we can proceed to setting up a Samba password for users who can access the shares. In the next section, we will create new Samba users and groups.

## Creating Samba users

Each Samba server needs to have specific users that can access the shared directories and files. Those users need to be both Samba and system users, as this is necessary for users to be able to authenticate and to read and write system files. Let’s assume that you need to create a local share for your small business or family group.

By creating local users specifically for using the Samba shares, you don’t need them to act like actual users as they only need to be able to access the shares. However, local Samba users need to be local system users. In our case, we will use our local `packt` user to create a new user for Samba:

1.  We use the following command to add the `packt` local user to Samba:

    ```
    packt is our primary user, it is also the owner of the samba_shares directory created at the beginning of this section. We can change that with the following command:

    ```
    alex using the following command:

    ```
    sudo adduser alex
    ```

    ```

    ```

2.  Then we add it to Samba with the following command:

    ```
    smb.conf file and add the users we would like to give permissions to Samba, such as packt and alex users. Let us open the /etc/samba/smb.conf file and add the following line inside the [samba shares] directive:

    ```
    valid users = @packt @alex
    ```

    ```

Before testing the new user’s access to Samba, we will need to give the user `alex` access to the shared directory. To do this, we follow these steps:

1.  We add the `acl` package in Ubuntu with the following command:

    ```
    setfacl command to set read, write, and execute permissions for the /home/packt/samba_shares directory. For this, we use the following command:

    ```
    sudo setfacl -R -m "u:alex:rwx" /home/packt/samba_shares
    ```

    ```

In the next section, we will show you how to access the Samba shares from different systems on the network.

## Accessing the Samba shares

On your network, you can access the shares from Linux, macOS, or Windows. To make everything a little bit challenging—worthy of a Linux master!—we will show you how to access the Samba shares in Linux using the CLI only. We will let you find out for yourselves how to access them from the GUI or from a Windows or macOS client. To access the shares from the CLI, use the `smbclient` tool.

On a Linux system, you need to install the Samba client `smbclient` first. We will assume that you will have an Ubuntu or Debian Linux client, but the steps are similar for other Linux distributions too. On Ubuntu/Debian, first install the Samba client with the following command, but make sure that your repositories are updated before you do this:

```
sudo apt install smbclient
```

If you are running a Fedora/RHEL client, install the Samba client with the following command:

```
sudo dnf install samba-client
```

For example, let’s access the shares of user `packt` from one of our local machines. Remember that the shares are on a local server running Ubuntu. We will use our local IP for the server and run the following code:

![Figure 13.23 – Accessing shares from a Linux CLI client](img/B19682_13_23.jpg)

Figure 13.23 – Accessing shares from a Linux CLI client

In the preceding screenshot, you can see that the Samba access was successful, and the user `packt` managed to access the Samba shares on the server. We used the `-U` option followed by the username to specify the name of the user we are connecting with. The location was given using the server’s local IP, followed by the Samba `samba shares` name. Furthermore, if we would like to see the Samba services available on the server, we could use the `-L` option with the `smbclient` command, as follows:

![Figure 13.24 – Listing the available services on the Samba server](img/B19682_13_24.jpg)

Figure 13.24 – Listing the available services on the Samba server

Here we are, at the end of this chapter on configuring Linux servers. The servers showcased were considered important and relevant for any Linux sysadmin to know. However, there are many other types of Linux servers not covered due to page-count constraints. Nevertheless, there are plenty of resources you can find online. As a base, you can start with the official documentation for RHEL, Ubuntu, or Debian, which will cover most of the Linux server types that you should know. One server type that will prove useful to know how to configure is the web server. Feel free to explore any other resources that you find relevant and might not be included in the *Further* *reading* list.

# Summary

In this chapter, we covered the installation and configuration processes for the most well-known services available for Linux. Knowing how to configure all the servers described in this chapter—from DNS to DHCP, Apache, and a printing server—is a minimum requirement for any Linux administrator.

By going through this chapter, you learned how to provide essential services for any Linux server. You learned how to set up and configure a web server using the Apache package; how to provide networked printing services to a small office or home office; how to run an FTP server and share files over TCP; how to share files with Windows clients on your network using the Samba/CIFS protocol; how to share files over Unix and Linux systems using the NFS file-sharing protocol; how to set up NTP to show an accurate time; and how to configure DNS and local DHCP servers. In a nutshell, you learned a lot in this chapter, and yet we have barely scratched the surface of Linux server administration.

In the next chapter, we will introduce you to cloud technologies.

# Questions

Now that you have a clear view of how to manage some of the most widely used services in Linux, here are some exercises that will further contribute to your learning:

1.  Try using a VPS for all the services detailed in this chapter, not on your local network.
2.  Try setting up a LEMP stack on Ubuntu.
3.  Test all the services described in this chapter using Fedora or RHEL-based distributions.

# Further reading

For more information about the topics covered in the chapter, you can refer to the following links:

*   Official Ubuntu documentation: [https://ubuntu.com/server/docs](https://ubuntu.com/server/docs)
*   RHEL official docs: [https://www.redhat.com/sysadmin/install-apache-web-server](https://www.redhat.com/sysadmin/install-apache-web-server)
*   NGINX official docs: [https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/)
*   DigitalOcean official docs: [https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)

# Part 4:Cloud Administration

In this fourth part, you will learn about advanced concepts related to cloud computing. By the end of this part, you will be proficient in using specific tools such as Kubernetes and Ansible and deploying Linux to the AWS and Azure clouds.

This part has the following chapters:

*   [*Chapter 14*](B19682_14.xhtml#_idTextAnchor299), *Short Introduction to* *Computing*
*   [*Chapter 15*](B19682_15.xhtml#_idTextAnchor326), *Deploying to the Cloud with AWS and Azure*
*   [*Chapter 16*](B19682_16.xhtml#_idTextAnchor342), *Deploying Applications with Kubernetes*
*   [*Chapter 17*](B19682_17.xhtml#_idTextAnchor359), *Infrastructure and Automation with Ansible*