# Chapter 6. Configuring and Using an Automated Installer (AI) Server

In this chapter, we will cover the following topics:

*   Configuring an AI server and installing a system from it

# Introduction

Installing Oracle Solaris 11 from a DVD is a simple and straight forward task, and usually, only a few screens and inputs are required to accomplish the operation. However, when there are many hosts to be installed, this approach might not be enough anymore. In previous versions of Oracle Solaris, there was a nice feature named JumpStart that made this installation process on multiple machines very easy. As we already know, time passed and Oracle introduced a new method that installs any machine (SPARC or x86 platforms) named **Automated Installer** (**AI**).

Concisely, the AI configuration requirement is composed of the following:

*   Configuring the AI server that provides the install services; this is the system where all configurations are performed
*   Configuring a **DHCP** server that offers IP addresses and other network settings
*   Configuring an **IPS** repository that has all necessary packages that are required to install the Oracle Solaris 11 host
*   Having a client where Oracle Solaris 11 will be installed after leasing a DHCP IP address from the DHCP server

The installation of a client through AI is not complex. Initially, the client gets booted from the network and requires an IP address from the DHCP server. Then, it gets the boot archive from the AI server and loads its own kernel. With the kernel already loaded, the client downloads the installation program through the HTTP protocol, identifies the installation services, and downloads the installation manifest. Finally, the client is installed using the IPS repository, with the manifest as a guideline that configures the system in an appropriate way. When the installation is complete, the host gets rebooted and the **System Configuration** (**SC)** profile is applied in order to configure the entire machine identification, such as the time zone, DNS, keyboard, and so on.

If everything happens properly, Oracle Solaris 11 is installed and starts working.

# Configuring an AI server and installing a system from it

The procedure to install and configure an AI server is very interesting, a little complex, and long. Let's do this!

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) that runs Oracle Solaris 11 with 4 GB RAM, a static IP address configuration, an IPS repository configured on the same machine server, and a DHCP server that can also be installed on the same host. Briefly, the AI, DHCP, and IPS servers will be installed on this virtual machine.

Additionally, a second virtual machine with 2 GB RAM, a network interface, and a disk with 20 GB space will be required because it will be used as the client where Oracle Solaris 11 will be installed.

Another important point is that we have to download the Oracle Solaris 11 Automated Installer (also known as the AI boot image) for x86 from the Oracle website at [http://www.oracle.com/technetwork/server-storage/solaris11/downloads/index.html?ssSourceSiteId=ocomen](http://www.oracle.com/technetwork/server-storage/solaris11/downloads/index.html?ssSourceSiteId=ocomen). This ISO image will be saved on the `/root` directory, and its version must be the same as the Oracle Solaris host that we want to install on the client (in this case, Version 11).

In this example, the AI server will be named `solaris11-1`, and the client machine will be named `solaris11-2ai`.

### Note

If you are using VirtualBox, I suggest that you download the latest version of VirtualBox and its respective **Extension Pack**, which enables the PXE support for Intel network interfaces. If you do not install the extension pack, this procedure will not work!

## How to do it…

Configuring the AI service is a two-stage procedure: we have to check the prerequisites and create its step-by-step configuration. As we have seen previously, we have to ensure that a static IP address is configured on an AI server by running the following command:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/zoneadmd.v4   static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
net0/zoneadmd.v4  static   ok           192.168.1.125/24
lo0/v6            static   ok           ::1/128
lo0/zoneadmd.v6   static   ok           ::1/128
```

As shown previously, the network interface (`net0`) is configured with a static IP address (`ipadm create-addr -T static -a 192.168.1.144/24 net0/v4`), and it is appropriate to verify that you have the Internet access and the DNS client configuration is working. By the way, the DNS client configuration will be changed in the next steps. So, to check the Internet access and current DNS client configuration, execute the following command:

```
root@solaris11-1:~# ping www.oracle.com
www.oracle.com is alive
root@solaris11-1:~# nslookup 
> server
Default server: 8.8.8.8
Address: 8.8.8.8#53
Default server: 8.8.4.4
Address: 8.8.4.4#53
> exit

```

A very important step is to edit the `/etc/netmask` file and insert the network mask that will be used:

```
root@solaris11-1:~# vi /etc/netmasks 
(truncated output)
# Both the network-number and the netmasks are specified in
# "decimal dot" notation, e.g:
#
#     128.32.0.0 255.255.255.0
#
192.168.1.0  255.255.255.0

```

To verify whether this configuration is being used and active, execute the following command:

```
root@solaris11-1:~# getent netmasks 192.168.1.0
192.168.1.0          255.255.255.0
```

During the installation, the client will receive packages from an IPS repository installed on the same system, so we have to confirm whether this IPS repository is online and is working by executing the following commands:

```
root@solaris11-1:~# pkg publisher
PUBLISHER               TYPE     STATUS P LOCATION
solaris                 origin   online F http://solaris11-1.example.com/
root@solaris11-1:~# svcs application/pkg/server
STATE          STIME    FMRI
online          1:09:30 svc:/application/pkg/server:default
root@solaris11-1:~# uname -a
SunOS solaris11-1 5.11 11.1 i86pc i386 i86pc
```

To test whether the IPS repository is really working, we can run a search for a package by running the following command:

```
root@solaris11-1:~# pkg search -p stunnel
PACKAGE                                            PUBLISHER
pkg:/service/security/stunnel@4.29-0.175.0.0.0.0.0 solaris
```

The next step requires your attention because there cannot be any existing DHCP configuration in the `/etc/inet` directory (`dhcp4.conf`), and the DHCP server must be disabled, as shown in the following command:

```
root@solaris11-1:~# svcs -a | grep dhcp
disabled       22:08:49 svc:/network/dhcp/server:ipv6
disabled       22:08:49 svc:/network/dhcp/relay:ipv4
disabled       22:08:49 svc:/network/dhcp/relay:ipv6
disabled       1:09:34 svc:/network/dhcp/server:ipv4

```

Additionally, when we are preparing an AI server, a DNS server must be configured and should be able to resolve the AI-installed server IP addresses. Therefore, let's configure both the DNS server and DNS client, but we are not going to delve into too much detail about the DNS server and client configuration here.

First, the client follows the DNS server, and we have to install the DNS server package by running the following command:

```
root@solaris11-1:~# pkg install service/network/dns/bind

```

In the next step, we have to configure the main DNS configuration file in order to make the DNS server resolve hostnames to the IP and vice versa:

```
root@solaris11-1:~# vi /etc/named.conf
options {
        directory       "/etc/dnsdb/config";
        pid-file        "/var/run/named/pid";
        dump-file       "/var/dump/dns_dump.db";
        statistics-file "/var/stats/named.stats";
        forwarders { 8.8.8.8; 8.8.4.4; };
};
zone "example.com" {
        type master;
        file "/etc/dnsdb/master/example.db";
};
zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/dnsdb/master/1.168.192.db";
};
```

According to the used directories from the `/etc/named.conf` file, it is time to create the same mentioned directories by executing the following command:

```
root@solaris11-1:~# mkdir /var/dump
root@solaris11-1:~# mkdir /var/stats
root@solaris11-1:~# mkdir -p /var/run/named
root@solaris11-1:~# mkdir -p /etc/dnsdb/master
root@solaris11-1:~# mkdir -p /etc/dnsdb/config

```

One of the most important steps in order to set the DNS server up is to create a database file for the straight name resolution (the hostname to the IP address) and another database file for the reverse resolution (the IP address to the hostname). Therefore, the first step is to create the straight database by executing the following commands:

```
root@solaris11-1:~# vi /etc/dnsdb/master/example.db
$TTL 3h
@  IN      SOA     solaris11-1.example.com. root.solaris11-1.example.com. (
        20140326 ;serial 
        3600 ;refresh (1 hour)
        3600 ;retry (1 hour)
        604800 ;expire (1 week)
        38400 ;minimum (1 day)
)
example.com.     IN      NS      solaris11-1.example.com.
gateway        IN      A       192.168.1.1   ; Router
solaris11-1            IN      A       192.168.1.144 ;
```

Now, it's time to create the reverse database file (the IP address to the hostname) using the following command:

```
root@solaris11-1:~# vi /etc/dnsdb/master/1.168.192.db
$TTL 3h
@       IN      SOA     solaris11-1.example.com. root.solaris11-1.example.com. (
        20140326 ;serial
        3600 ;refresh (1 hour)
        3600 ;retry (1 hour)
        604800 ;expire (1 week)
        38400 ;minimum (1 day)
)
        IN      NS      solaris11-1.example.com.
1       IN      PTR     gateway.example.com.   
144     IN      PTR     solaris11-1.example.com
```

Finally, the DNS server is ready and its service must be enabled by running the following commands:

```
root@solaris11-1:~# svcs -a | grep dns/server
disabled       18:46:05 svc:/network/dns/server:default
root@solaris11-1:~# svcadm enable svc:/network/dns/server:default
root@solaris11-1:~# svcs -a | grep dns/server
online          7:09:05 svc:/network/dns/server:default
```

The DNS client is a very important step for our recipe, and it can be configured by executing the following commands:

```
root@solaris11-1:~# svccfg -s svc:/network/dns/client setprop config/nameserver = net_address: "(192.168.1.144)"
root@solaris11-1:~# svccfg -s svc:/network/dns/client setprop config/domain = astring: '("example.com")'
root@solaris11-1:~# svccfg -s svc:/network/dns/client setprop config/search = astring: '("example.com")'
root@solaris11-1:~# svccfg -s svc:/system/name-service/switch setprop config/ipnodes = astring: '("files dns")'
root@solaris11-1:~# svccfg -s svc:/system/name-service/switch setprop config/host = astring: '("files dns")'
root@solaris11-1:~# svccfg -s svc:/network/dns/client listprop config
config                      application        
config/value_authorization astring     solaris.smf.value.name-service.dns.client
config/nameserver          net_address 192.168.1.144
config/domain              astring     example.com
config/search              astring     example.com
root@solaris11-1:~# svccfg -s svc:/system/name-service/switch listprop config
config                      application        
config/default             astring     files
config/value_authorization astring     solaris.smf.value.name-service.switch
config/printer             astring     "user files"
config/ipnodes             astring     "files dns"
config/host                astring     "files dns"
root@solaris11-1:~# svcadm refresh svc:/network/dns/client
root@solaris11-1:~# svcadm restart svc:/network/dns/client
root@solaris11-1:~# svcadm refresh svc:/system/name-service/switch:default
root@solaris11-1:~# svcadm restart svc:/system/name-service/switch:default

```

To test whether our DNS server configuration and DNS client configuration are working, we can use the `nslookup` tool to verify them, as shown in the following command:

```
root@solaris11-1:~# nslookup 
> server
Default server: 192.168.1.144
Address: 192.168.1.144#53
> solaris11-1.example.com
Server:    192.168.1.144
Address:  192.168.1.144#53
Name:  solaris11-1.example.com
Address: 192.168.1.144
> 192.168.1.144
Server:    192.168.1.144
Address:  192.168.1.144#53
144.1.168.192.in-addr.arpa  name = solaris11-1.example.com.
> exit

```

Perfect! Both the DNS server and the client are now configured on the AI install server.

From this point, we can start to configure the AI server itself, which requires the multicast service to be enabled, and this can be done by executing the following commands:

```
root@solaris11-1:~# svcs -a | grep  multicast
disabled       22:08:43 svc:/network/dns/multicast:default
root@solaris11-1:~# svcadm enable svc:/network/dns/multicast:default
root@solaris11-1:~# svcs -a | grep  multicast
online          2:38:35 svc:/network/dns/multicast:default
```

Additionally, the AI server also requires a series of tools to be configured, and we have to install the associated package by running the following command:

```
root@solaris11-1:~# pkg install installadm

```

Now the game begins! We have to configure an AI install service with a name that will be associated with an install image. Later, the install service name will be used by the client to access and deploy the install image. From this point, the install service name will be used as an index in order to find the correct install image. If we wanted to install both SPARC and x86 clients, we should have two install services: the first associated with a SPARC install image and a second one associated with an X86 install image.

To create an AI install service, execute the following command:

```
root@solaris11-1:~# installadm create-service -n borges_ai -s /root/sol-11_1-ai-x86.iso -i 192.168.1.20 -c 10 -d /export/borges_ai
Creating service from: /root/sol-11_1-ai-x86.iso
Setting up the image ...
Creating i386 service: borges_ai
Image path: /export/borges_ai
Starting DHCP server...
Adding IP range to local DHCP configuration
Refreshing install services
Creating default-i386 alias
Setting the default PXE bootfile(s) in the local DHCP configuration
to:
bios clients (arch 00:00):  default-i386/boot/grub/pxegrub2
uefi clients (arch 00:07):  default-i386/boot/grub/grub2netx64.efi
Refreshing install services
```

From the previous command, we have the following:

*   `-n`: This is the service name
*   `-s`: This is the path to the AI ISO image
*   `-i`: This will update the DHCP server starting from 192.168.1.20
*   `-c`: This install service will serve ten IP addresses
*   `-d`: This is the directory where the AI ISO image will be unpacked

After creating the `borges_ai` install service, the DHCP presents the following configuration file:

```
root@solaris11-1:~# more /etc/inet/dhcpd4.conf 
# dhcpd.conf
#
# Configuration file for ISC dhcpd
# (created by installadm(1M))
#
default-lease-time 900;
max-lease-time 86400;
# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# arch option for PXEClient
option arch code 93 = unsigned integer 16;

# Set logging facility (accompanies setting in syslog.conf)
log-facility local7;

# Global name services
option domain-name-servers 8.8.8.8, 8.8.4.4;
option domain-name "example.com";
option domain-search "example.com";
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.20 192.168.1.29;
  option broadcast-address 192.168.1.255;
  option routers 192.168.1.1;
  next-server 192.168.1.144;
}

class "PXEBoot" {
  match if (substring(option vendor-class-identifier, 0, 9) = "PXEClient");
  if option arch = 00:00 {
    filename "default-i386/boot/grub/pxegrub2";
  } else if option arch = 00:07 {
    filename "default-i386/boot/grub/grub2netx64.efi";
  }
}
```

We can face problems several times, and it would be nice if we could start the entire procedure from scratch and start over again. Therefore, if something goes wrong, it's feasible to undo the previous step, executing the `installadm install-service` command and executing the previous steps again:

```
root@solaris11-1:~# installadm delete-service default-i386
WARNING: The service you are deleting, or a dependent alias, is
the alias for the default i386 service. Without the 'default-i386'
service, i386 clients will fail to boot unless explicitly
assigned to a service using the create-client command.
Are you sure you want to delete this alias? [y/N]: y
Removing this service's bootfile(s) from local DHCP configuration
Stopping the service default-i386

root@solaris11-1:~# installadm delete-service -r borges_ai
WARNING: The service you are deleting, or a dependent alias, is
the alias for the default i386 service. Without the 'default-i386'
service, i386 clients will fail to boot unless explicitly
assigned to a service using the create-client command.

Are you sure you want to delete this alias? [y/N]: Y
Removing this service's bootfile(s) from local DHCP configuration
Stopping the service default-i386
Removing host entry '08:00:27:DF:15:A6' from local DHCP configuration.
Stopping the service borges_ai
The installadm SMF service is being taken offline.
The installadm SMF service is no longer online because the last
install service has been disabled or deleted.
```

After deleting the AI server configuration, it is also recommended that you remove the `/etc/inet/dhcpd4.conf` file and disable the DHCP server service by executing the following command:

```
root@solaris11-1:~# svcadm disable svc:/network/dhcp/server:ipv4

```

Returning to the configuration steps, an AI install server and its install services are represented by a service from SMF, as shown in the following command:

```
root@solaris11-1:~# svcs -a | grep install/server
online          4:53:41 svc:/system/install/server:default
root@solaris11-1:~# svcs -l svc:/system/install/server:default
fmri         svc:/system/install/server:default
name         Installadm Utility
enabled      true
state        online
next_state   none
state_time   March 23, 2014 04:53:41 AM BRT
logfile      /var/svc/log/system-install-server:default.log
restarter    svc:/system/svc/restarter:default
contract_id  472 
manifest     /lib/svc/manifest/system/install/server.xml
dependency   optional_all/restart svc:/network/dns/multicast:default (online)
dependency   optional_all/none svc:/network/tftp/udp6:default (online)
dependency   optional_all/none svc:/network/dhcp-server:default (uninitialized)
```

To list the existing AI install services, execute the following command:

```
root@solaris11-1:~# installadm list
Service Name Alias Of  Status  Arch   Image Path 
------------ --------  ------  ----   ---------- 
borges_ai    -         on      i386   /root/borges_ai
default-i386 borges_ai on      i386   /root/borges_ai
```

The command output shows us that Oracle Solaris 11 has created (by default) an AI install service named `default-i386`, which is an alias for our AI install service named `borges_ai`.

Until now, the system has created an AI install service (`borges_ai`), and then, we have had to associate it with one or more clients that will be installed through the AI server. Before accomplishing this task, the MAC address information from these clients must be collected. So, as we are using another virtual machine as the client (`solaris11-2ai`), it's easy to get the MAC information from the virtual machine properties (VirtualBox or VMware).

For example, when working with VirtualBox, you can select the Virtual Machine (Solaris11-1) by navigating to **Settings** | **Network** | **Advanced**.

The MAC address property from VirtualBox is shown in the following screenshot:

![How to do it…](img/00023.jpeg)

If we are working with VMware Workstation, it's possible to get the MAC address from a virtual machine by navigating to **Virtual Machine (Solaris11-1)** | **VM** | **Settings** | **Network Adapter** | **Advanced**, as shown in the following screenshot:

![How to do it…](img/00024.jpeg)

Once we have the MAC address, we use it to add the client (the host that will be installed using AI) by executing the following commands:

```
root@solaris11-1:~# installadm create-client -e 08:00:27:DF:15:A6 -n borges_ai
Adding host entry for 08:00:27:DF:15:A6 to local DHCP configuration.
root@solaris11-1:~# installadm list -c 
Service Name Client Address     Arch   Image Path 
------------ --------------     ----   ---------- 
borges_ai    08:00:27:DF:15:A6  i386   /export/borges_ai
```

The previous output shows us a client with the MAC address `08:00:27:DF:15:A6`, which was bound to an AI install service named `borges_ai`.

As the client (MAC `08:00:27:DF:15:A6`) is already assigned to an AI install service, the next step will be to create an AI manifest. What is that? An AI manifest is a file that contains instructions to install and configure AI clients that will be installed using the AI service. As this manifest is an XML file, it would be very hard to create a manifest for each client that needs to use the AI install service, and so a default manifest is provided by each boot image in order to use it for any client of any install service that will use this boot image.

In the AI framework, there are two types of manifests, as follows:

*   **Default**: This is valid for all clients that do not have any customized manifests. The default manifest is named `default.xml`.
*   **Custom**: This is a particular manifest that has an install image associated, and one or more clients can be assigned to it.

What is the decision factor to choose either a customized manifest or a default one? This is the role of a file named the `criteria` file, which associates clients to either a specific manifest or a default manifest using properties or attributes from these clients.

The following is an example of a default manifest (`default.xml`) that was installed in the `/export/borges_ai/auto_install` directory when we run the `installadm` create-service command:

```
root@solaris11-1:~# cat /export/borges_ai/auto_install/default.xml
<?xml version="1.0" encoding="UTF-8"?>
<!--

 Copyright (c) 2008, 2012, Oracle and/or its affiliates. All rights reserved.

-->
<!DOCTYPE auto_install SYSTEM "file:///usr/share/install/ai.dtd.1">
<auto_install>
  <ai_instance name="default">
    <target>
      <logical>
        <zpool name="rpool" is_root="true">
          <!--
            Subsequent <filesystem> entries instruct an installer to create
            following ZFS datasets:

                <root_pool>/export         (mounted on /export)
                <root_pool>/export/home    (mounted on /export/home)

            Those datasets are part of standard environment and should be
            always created.

            In rare cases, if there is a need to deploy an installed system
            without these datasets, either comment out or remove <filesystem>
            entries. In such scenario, it has to be also assured that
            in case of non-interactive post-install configuration, creation
            of initial user account is disabled in related system
            configuration profile. Otherwise the installed system would fail
            to boot.
          -->
          <filesystem name="export" mountpoint="/export"/>
          <filesystem name="export/home"/>
          <be name="solaris"/>
        </zpool>
      </logical>
    </target>
    <software type="IPS">
      <destination>
        <image>
          <!-- Specify locales to install -->
          <facet set="false">facet.locale.*</facet>
          <facet set="true">facet.locale.de</facet>
          <facet set="true">facet.locale.de_DE</facet>
          <facet set="true">facet.locale.en</facet>
          <facet set="true">facet.locale.en_US</facet>
          <facet set="true">facet.locale.es</facet>
          <facet set="true">facet.locale.es_ES</facet>
          <facet set="true">facet.locale.fr</facet>
          <facet set="true">facet.locale.fr_FR</facet>
          <facet set="true">facet.locale.it</facet>
          <facet set="true">facet.locale.it_IT</facet>
          <facet set="true">facet.locale.ja</facet>
          <facet set="true">facet.locale.ja_*</facet>
          <facet set="true">facet.locale.ko</facet>
          <facet set="true">facet.locale.ko_*</facet>
          <facet set="true">facet.locale.pt</facet>
          <facet set="true">facet.locale.pt_BR</facet>
          <facet set="true">facet.locale.zh</facet>
          <facet set="true">facet.locale.zh_CN</facet>
          <facet set="true">facet.locale.zh_TW</facet>
        </image>       </destination>
      <source>
        <publisher name="solaris">
          <origin name="http://pkg.oracle.com/solaris/release"/>
        </publisher>
      </source>
      <!--
        The version specified by the "entire" package below, is
        installed from the specified IPS repository.  If another build
        is required, the build number should be appended to the
        'entire' package in the following form:

            <name>pkg:/entire@0.5.11-0.build#</name>
      -->
      <software_data action="install">
        <name>pkg:/entire@0.5.11-0.175.1</name>
        <name>pkg:/group/system/solaris-large-server</name>
      </software_data>
    </software>
  </ai_instance>
</auto_install>
```

The `default.xml` file is very simple, and it has some good points that are worth mentioning, as shown:

*   `<ai_instance name="default">`: This element shows us the name of the AI instance
*   `<software type="IPS">`: All these packages come from an IPS server
*   `<publisher name="solaris">`: This is the IPS publisher name
*   `<origin name="http://pkg.oracle.com/solaris/release"/>`: This is the origin URI assigned to the repository that was made available by the publisher (Solaris)
*   `<name>pkg:/entire@0.5.11-0.build#</name>` and `<name>pkg:/entire@0.5.11-0.175.1</name>`: These are basically the entire IPS package and tell us about the version of the offered Oracle Solaris, and this information will be used to install patches or upgrades
*   `<name>pkg:/group/system/solaris-large-server</name>`: This is a package group that contains several tools and important files such as libraries, drivers, and Python, and they should be installed

It is interesting to realize that my own system does not have the `solaris-large-server` package installed, as shown in the following command:

```
root@solaris11-1:~# pkg search solaris-large-server
INDEX      ACTION VALUE                                     PACKAGE
pkg.fmri   set    solaris/group/system/solaris-large-server pkg:/group/system/solaris-large-server@0.5.11-0.175.1.0.0.24.3
root@solaris11-1:~# pkg info -r solaris pkg:/group/system/solaris-large-server@0.5.11-0.175.1.0.0.24.3

          Name: group/system/solaris-large-server
       Summary: Oracle Solaris Large Server
   Description: Provides an Oracle Solaris large server environment
      Category: Meta Packages/Group Packages
         State: Not installed
     Publisher: solaris
       Version: 0.5.11
 Build Release: 5.11
        Branch: 0.175.1.0.0.24.3
Packaging Date: September 19, 2012 06:53:18 PM 
          Size: 5.46 kB
          FMRI: pkg://solaris/group/system/solaris-large-server@0.5.11,5.11-0.175.1.0.0.24.3:20120919T185318Z

          Name: system/zones/brand/solaris
       Summary: 
         State: Not installed (Renamed)
    Renamed to: pkg:/system/zones/brand/brand-solaris@0.5.11,5.11-0.173.0.0.0.0.0
                consolidation/osnet/osnet-incorporation
     Publisher: solaris
       Version: 0.5.11
 Build Release: 5.11
        Branch: 0.173.0.0.0.1.0
Packaging Date: August 26, 2011 07:00:28 PM 
          Size: 5.45 kB
          FMRI: pkg://solaris/system/zones/brand/solaris@0.5.11,5.11-0.173.0.0.0.1.0:20110826T190028Z
```

Therefore, according to the previous `default.xml` file (although it is not usually necessary), we have to install the missing package by executing the following command:

```
root@solaris11-1:~# pkg install pkg:/group/system/solaris-large-server@0.5.11-0.175.1.0.0.24.3

```

Returning to the default manifest (`default.xml`) explanation, we have to back up and modify it in order to adapting to our environment that has the following characteristics:

*   The AI instance name (`borges_ai`)
*   The IPS origin URI—`http://solaris11-1.example.com/`—(from the `pkg publisher` command)
*   Auto reboot (`auto_reboot`) is set to true

The code for the previous task is as follows:

```
root@solaris11-1:~# mkdir /backup
root@solaris11-1:~# cp /export/borges_ai/auto_install/manifest/default.xml /export/borges_ai/auto_install/borges_ai.xml
root@solaris11-1:~# vi /export/borges_ai/auto_install/borges_ai.xml
root@solaris11-1:~# grep borges_ai /export/borges_ai/auto_install/borges_ai.xml
 <ai_instance name="borges_ai" auto_reboot="true">
root@solaris11-1:~# grep solaris11-1 /export/borges_ai/auto_install/borges_ai.xml
<origin name="http://solaris11-1.example.com"/>
```

We have created a new manifest named `borges_ai.xml`, but we have to create a `criteria` file in order to associate the client (solaris11-2ai) with this manifest. Usually, there are some good attributes that can be used in a `criteria` file: MAC address, IPv4, platform, architecture (arch), memory (mem), hostname, and so on. Therefore, after a criteria file is created, the rule is that if the client matches any of these criteria files, the associated manifest will be used (in our case, the customized manifest is `borges_ai.xml`). If it does not match, the `default.xml` file manifest is used.

To create a criteria file with the MAC address of the client machine (solaris11-2ai), we can execute the following command:

```
root@solaris11-1:~# vi /export/borges_ai/auto_install/borges_criteria_ai.xml
<ai_criteria_manifest>
  <ai_criteria name="mac"> 
    <value>08:00:27:DF:15:A6</value>
  </ai_criteria>
</ai_criteria_manifest>
```

Finally, we're able to associate this criteria file (`borges_criteria_ai.xml`) and the customized manifest file (`borges_ai.xml`) with the AI install service (`borges_ai`):

```
root@solaris11-1:~# installadm create-manifest -n borges_ai -f /export/borges_ai/auto_install/borges_ai.xml -C /export/borges_ai/auto_install/borges_criteria_ai.xml

```

From the previous command, we note the following:

*   `-n`: This is the AI install service name
*   `-f`: This is the customized manifest file
*   `-C`: This is the criteria file

An alternative and easier approach to creating a `criteria` file is to associate the client with this `criteria` file and make the necessary customization, specifying the client MAC address as the criteria by running the following commands:

```
root@solaris11-1:~# installadm create-manifest -n borges_ai -f /export/borges_ai/auto_install/borges_ai.xml
root@solaris11-1:~#  installadm set-criteria –n borges_ai -m borges_ai –c mac="08:00:27:XX::YY:ZZ"

```

To verify the AI configuration up to this point, execute the following commands:

```
root@solaris11-1:/backup# installadm list 
Service Name Alias Of  Status  Arch   Image Path 
------------ --------  ------  ----   ---------- 
borges_ai    -         on      i386   /export/borges_ai
default-i386 borges_ai on      i386   /export/borges_ai
root@solaris11-1:~# installadm list -m
Service/Manifest Name  Status   Criteria
---------------------  ------   --------
borges_ai
   borges_ai                    mac  = 08:00:27:DF:15:A6
   orig_default        Default  None
default-i386
   orig_default        Default  None
```

That is good! The next step is interesting because usually, during Oracle Solaris 11 installation, we are prompted to enter many inputs, such as the initial user account, root password, time zone, keyboard, and so on. To answer all these questions once is easy, but when installing 100 machines, this would be a serious problem.

To automate this process, there's a configuration file named **System Configuration profile** (**SC**) that provides any necessary answer during the first boot after the Oracle Solaris 11 installation.

To help us with SC profile creation, Oracle Solaris 11 provides some templates of this profile in the `/export/borges_ai/auto_install/sc_profiles` directory. Before modifying it, we are going to copy a template from this directory and highlight some interesting lines, as shown in the following command:

```
root@solaris11-1:~# cp /export/borges_ai/auto_install/sc_profiles/sc_sample.xml /export/borges_ai/auto_install/sc_borges_ai.xml
root@solaris11-1:~# cat /export/borges_ai/auto_install/sc_borges_ai.xml
<?xml version="1.0"?>
<!--
Copyright (c) 2011, 2012, Oracle and/or its affiliates. All rights reserved.
-->
<!--
Sample system configuration profile for use with Automated Installer

Configures the following:
* User account name 'jack', password 'jack', GID 10, UID 101, root role, bash shell
* 'root' role with password 'solaris'
* Keyboard mappings set to US-English
* Timezone set to UTC
* Network configuration is automated with Network Auto-magic
* DNS name service client is enabled

See installadm(1M) for usage of 'create-profile' subcommand.
-->

<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<service_bundle type="profile" name="system configuration">
    <service name="system/config-user" version="1">
      <instance name="default" enabled="true">
        <property_group name="user_account">
          <propval name="login" value="jack"/>
          <propval name="password" value="9Nd/cwBcNWFZg"/>
          <propval name="description" value="default_user"/>
          <propval name="shell" value="/usr/bin/bash"/>
          <propval name="gid" value="10"/>
          <propval name="uid" value="101"/>
          <propval name="type" value="normal"/>
          <propval name="roles" value="root"/>
          <propval name="profiles" value="System Administrator"/>
        </property_group>
        <property_group name="root_account">
            <propval name="password" value="$5$dnRfcZse$Hx4aBQ161Uvn9ZxJFKMdRiy8tCf4gMT2s2rtkFba2y4"/>
            <propval name="type" value="role"/>
        </property_group>
      </instance>
    </service>

    <service version="1" name="system/identity">
      <instance enabled="true" name="node">
        <property_group name="config">
           <propval name="nodename" value="solaris"/>
        </property_group>
      </instance>
    </service>

    <service name="system/console-login" version="1">
      <instance name="default" enabled="true">
        <property_group name="ttymon">
          <propval name="terminal_type" value="sun"/>
        </property_group>
      </instance>
    </service>

    <service name="system/keymap" version="1">
      <instance name="default" enabled="true">
        <property_group name="keymap">
          <propval name="layout" value="US-English"/>
        </property_group>
      </instance>
    </service>

    <service name="system/timezone" version="1">
      <instance name="default" enabled="true">
        <property_group name="timezone">
          <propval name="localtime" value="UTC"/>
        </property_group>
      </instance>
    </service>

    <service name="system/environment" version="1">
      <instance name="init" enabled="true">
        <property_group name="environment">
          <propval name="LANG" value="en_US.UTF-8"/>
        </property_group>
      </instance>
    </service>

    <service name="network/physical" version="1">
      <instance name="default" enabled="true">
          <property_group name="netcfg" type="application">
              <propval name="active_ncp" type="astring" value="Automatic"/>
          </property_group>
      </instance>
    </service>
</service_bundle>
```

After carefully reading this file, we have the following conclusions:

*   The initial default username is `jack`, with the password `jack`
*   The root is a role (this is not a normal account), and its password is `solaris`
*   The machine name is `solaris`
*   The active NCP is `Automatic`

To adapt this file for our purpose, change the initial default username to `borges` and its password to `oracle123!` (`$5$VPcyGvgl$bt4cybd8cpZdHKWF2tvBn.SPFeJ8YdgvQUqHzWkNLl1`). Additionally, the hostname will be changed to `solaris11-2ai`. Every change can be verified by running the following command:

```
root@solaris11-1:/export/borges_ai/auto_install# cat sc_borges_ai.xml 
(truncated output)
See installadm(1M) for usage of 'create-profile' subcommand.
-->
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<service_bundle type="profile" name="system configuration">
    <service name="system/config-user" version="1">
      <instance name="default" enabled="true">
        <property_group name="user_account">
          <propval name="login" value="borges"/>
          <propval name="password" value="$5$VPcyGvgl$bt4cybd8cpZdHKWF2tvBn.SPFeJ8YdgvQUqHzWkNLl1"/>
          <propval name="description" value="default_user"/>
          <propval name="shell" value="/usr/bin/bash"/>
          <propval name="gid" value="10"/>
          <propval name="uid" value="101"/>
          <propval name="type" value="normal"/>
          <propval name="roles" value="root"/>
          <propval name="profiles" value="System Administrator"/>
        </property_group>
        <property_group name="root_account">
            <propval name="password" value="$5$dnRfcZse$Hx4aBQ161Uvn9ZxJFKMdRiy8tCf4gMT2s2rtkFba2y4"/>
            <propval name="type" value="role"/>
        </property_group>
      </instance>
    </service>

    <service version="1" name="system/identity">
      <instance enabled="true" name="node">
        <property_group name="config">
           <propval name="nodename" value="solaris11-2ai"/>
        </property_group>
      </instance>
    </service>

(truncated output)

```

Now that the SC profile `sc_borges_ai.xml` has been modified, it is time to create it in the AI service database, to validate its syntax, and to list the result, as done in the following commands:

```
root@solaris11-1:~# installadm create-profile -n borges_ai -f /export/borges_ai/auto_install/sc_borges_ai.xml -c mac=08:00:27:DF:15:A6
Profile sc_borges_ai.xml added to database.

root@solaris11-1:~# installadm validate -n borges_ai -p sc_borges_ai.xml 
Validating static profile sc_borges_ai.xml...
 Passed
root@solaris11-1:~# installadm list -p 
Service/Profile Name  Criteria
--------------------  --------
borges_ai
   sc_borges_ai.xml   mac = 08:00:27:DF:15:A6
```

This is wonderful! We have configured the AI server. The `sc_borges_ai.xml` SC profile will be used by our client (solaris11-2ai) according to the established criteria (MAC = `08:00:27:DF:15:A6`).

Finally, it is show time! To test whether the entire AI server configuration is working, we have to turn on the client (the solaris11-2ai virtual machine) and just wait for the whole installation. If everything is working, we will see the following screenshot:

![How to do it…](img/00025.jpeg)

After selecting **Oracle Solaris 11.1 Automated Install**, the Oracle Solaris 11 installation should begin.

![How to do it…](img/00026.jpeg)

This is simply outstanding!

### An overview of the recipe

This section was impressive! We learned how to configure an AI install server in order to remotely install a client without any interaction. In the middle of the chapter, we also saw how to configure a DNS server and client.

# References

*   *Installing Oracle Solaris 11 Systems* at [http://docs.oracle.com/cd/E23824_01/html/E21798/docinfo.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/E21798/docinfo.html#scrolltoc)
*   *Booting and Shutting Down* *Oracle Solaris 11.1 Systems* at [http://docs.oracle.com/cd/E26502_01/html/E28983/docinfo.html#scrolltoc](http://docs.oracle.com/cd/E26502_01/html/E28983/docinfo.html#scrolltoc)
*   *Configuring a Basic DNS Server + Client in Solaris 11*, *Paul Johnson*, at [http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris11-net-svcs-ips-2086656.html](http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris11-net-svcs-ips-2086656.html)
*   *Exploring Networking, Services, and the New Image Packaging System In Oracle Solaris 11*, *Alexandre Borges*, at [http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris11-net-svcs-ips-2086656.html](http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris11-net-svcs-ips-2086656.html)