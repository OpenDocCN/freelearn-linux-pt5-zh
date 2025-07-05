# Chapter 3. Networking

In this chapter, we will cover the following recipes:

*   Playing with Reactive Network Configuration
*   Internet Protocol Multipathing
*   Setting the link aggregation
*   Configuring network bridging
*   Configuring link protection and the DNS Client service
*   Configuring the DHCP server
*   Configuring Integrated Load Balancer

# Introduction

It's needless to say that a network card and its respective network configuration are crucial for an operating system such as Oracle Solaris 11\. I've been working with Oracle Solaris since version 7, and its network setup was always very simple, using files such as `/etc/hostname.<interface>`, `/etc/hosts`, `/etc/defaultrouter`, `/etc/resolv.conf`, and `/etc/hostname`. At that time, there wasn't anything else apart from these files, and this was very suitable because configuring a network takes only a few minutes. On the other hand, there wasn't any flexibility when the network configuration had to be changed. Moreover, at that time, there weren't any wireless interfaces on portable computers, and Oracle Solaris only worked with SPARC processors. That time has passed.

This network architecture was kept until Oracle Solaris 10 even when hundreds of modifications and new features were introduced on Oracle Solaris 10\. Now, in Oracle Solaris 11, there are new commands and different methods to set up your network. Furthermore, there are many interesting technologies that have improved since the previous version of Oracle Solaris, and some of them are included in Oracle Solaris 11.

In this chapter, we're going to learn about many materials related to Oracle Solaris 11 as well as advanced administration.

### Note

A fundamental point must be highlighted—during all examples shown here, I assume that there's a DHCP server on the network. In my case, my DHCP server is provided by a D-Link wireless router. Don't forget this warning!

# Playing with Reactive Network Configuration

This discussion is probably one of the more interesting topics from Oracle Solaris 11 and is also one of the most complex.

Some years ago, Oracle Solaris had only the SPARC version, and wireless networks were absent or rare. Starting with the release of Oracle Solaris 10, the use of Oracle Solaris on notebooks has been growing year after year. During the same time, wireless networks became popular and everything changed. However, this mobility brought with it a small problem with the network configuration. For example, imagine that we have a notebook with Oracle Solaris 11 installed and some day there's a need to connect to four different networks—home1, home2, work, and university—in order to read e-mails or access the Internet. This would be crazy because for each one of these environments, we would have to change the network configuration to be able to connect to the data network. Worse, if three out of the four networks require a manual network configuration (IP address, mask, gateway, name server, domain, and so on), we'd lose so much time in manual configuration.

Oracle Solaris 11 has an excellent feature that manages **Reactive Network Configuration** (**RNC**). Basically, using RNC, a user can create different network configurations, and from a user request or event (turning a wireless card on or off, leasing and renewing a DHCP setting, connecting or disconnecting a cable, and so on), it's possible to change the network configuration quickly. All of this is feasible only because RNC was implemented based on a key concept named profiles, which can be classified as fixed or reactive, and they have many properties that help us configure the network that is appropriated.

There are two types of profiles—**Network Configuration Profiles** (**NCP**) and **Location Profiles**—and both are complementary. An NCP (a kind of container) is composed of **Network Configuration Units** (**NCUs**) that are configuration objects, and they all have properties that are required to configure the network. Additionally, there's a third type of profile named **External Network Modifiers** (**ENMs**) that are used with VPNs, which require a special profile that is able to create its own configuration.

There are many terms or short concepts up to this point, so let's summarize them:

*   **RNC**: This stands for Reactive Network Configuration
*   **Profiles**: There are two classes: fixed or reactive
*   **NCP**: This stands for Network Configuration Profile
*   **Location Profile**: This is a profile that brings complementary information to NCP
*   **NCU**: This stands for Network Configuration Unit and are what makes up an NCP profile
*   **EMN**: This stands for External Network Modifier and is another kind of profile

Returning to the two main profiles (**NCP** and **Location**), the role of NCP is to provide the basic network configuration for interfaces, and the role of Location profiles is to complete the information and configuration provided by NCP.

Some useful configurations given by the Location profile are the **IP Filter** settings, domain, DNS configuration, and so on. The default Location profile named **NoNet** is applied to the system when there is no valid IP address. When one of the network interfaces gets a valid IP address, the **Automatic Location** profile is used.

There are two types of **NCP** profile. The first type is the `Automatic` profile that is read-only, has your configuration (more about this later) hanged when a network device is added or removed, uses the DHCP service, always gives preference to an Ethernet card instead of a wireless card, is composed of one **Link NCU** (offered in several flavors: physical link, aggregation, virtual NIC, vlans, and so on), and has an **Interface NCU** inside it.

The second type is the user-defined profile that must and can be set up manually (so it can be edited) according to the user goals.

## Getting ready

To follow this recipe, you need two virtual machines (VirtualBox or VMware) with Oracle Solaris 11 installed, each one with 4 GB RAM and four network interfaces.

## How to do it…

There are two key services related to RNC: `svc:/network/netcfg:default` and `svc:/network/location:default`. Both services must be enabled and working, and we have to pay attention to the `svc:/network/location:default` dependencies:

```
root@solaris11-1:~# svcs -a | grep netcfg
online         18:07:01 svc:/network/netcfg:default
root@solaris11-1:~# svcs -a | grep location:default
online         18:12:22 svc:/network/location:default
root@solaris11-1:~# svcs -l netcfg
fmri         svc:/network/netcfg:default
name         Network configuration data management
enabled      true
state        online
next_state   none
state_time   January  6, 2014 06:07:01 PM BRST
alt_logfile  /system/volatile/network-netcfg:default.log
restarter    svc:/system/svc/restarter:default
contract_id  7
manifest     /lib/svc/manifest/network/network-netcfg.xml
root@solaris11-1:~# svcs -l svc:/network/location:default
fmri         svc:/network/location:default
name         network interface configuration
enabled      true
state        online
next_state   none
state_time   January  6, 2014 06:12:22 PM BRST
logfile      /var/svc/log/network-location:default.log
restarter    svc:/system/svc/restarter:default
manifest     /lib/svc/manifest/network/network-location.xml
dependency   require_all/none svc:/network/location:upgrade (online)
dependency   require_all/none svc:/network/physical:default (online)
dependency   require_all/none svc:/system/manifest-import:default (online)
dependency   require_all/none svc:/network/netcfg:default (online)
dependency   require_all/none svc:/system/filesystem/usr (online)

```

All profiles are listed using the `netcfg` command:

```
root@solaris11-1:~# netcfg list
NCPs:
  Automatic
  DefaultFixed
Locations:
  Automatic
  NoNet
```

This is a confirmation of what we've seen in the introduction of this section. There's an NCP profile named `Automatic`, which is related to the DHCP service, and another NCP profile that's associated to a user-defined NCP profile named `DefaultFixed`. Moreover, there are two locations—`Automatic`, which is applied to the system when at least one network interface has a valid IP address, and `NoNet`, which is enforced when no network card has received a valid IP address.

Nonetheless, there is a lot of additional information that we can get from each of these profiles by executing the following command:

```
root@solaris11-1:~# netcfg list -a ncp Automatic
ncp:Automatic
  management-type   reactive
NCUs:
  phys  net0
  phys  net1
  phys  net2
  phys  net3
  ip    net0
  ip    net1
  ip    net3
  ip    net2
```

All of the network interfaces and their respective IP address objects are bound to the `Automatic` NCP profile, while nothing is assigned to the `DefaultFixed` NCP profile:

```
root@solaris11-1:~# netcfg list -a ncp DefaultFixed
ncp:DefaultFixed
  management-type   fixed
```

In the same way, tons of information can be taken from location profiles by running the following command:

```
root@solaris11-1:~# netcfg list -a loc Automatic
loc:Automatic
  activation-mode            system
  conditions                 
  enabled                    false
  nameservices               dns
  nameservices-config-file   "/etc/nsswitch.dns"
  dns-nameservice-configsrc  dhcp
  dns-nameservice-domain     
  dns-nameservice-servers    
  dns-nameservice-search     
  dns-nameservice-sortlist   
  dns-nameservice-options    
  nis-nameservice-configsrc  
  nis-nameservice-servers    
  ldap-nameservice-configsrc 
  ldap-nameservice-servers   
  default-domain             
  nfsv4-domain               
  ipfilter-config-file       
  ipfilter-v6-config-file    
  ipnat-config-file          
  ippool-config-file         
  ike-config-file            
  ipsecpolicy-config-file    

root@solaris11-1:~# netcfg list -a loc NoNet
loc:NoNet
  activation-mode            system
  conditions                 
  enabled                    false
  nameservices               files
  nameservices-config-file   "/etc/nsswitch.files"
  dns-nameservice-configsrc  dhcp
  dns-nameservice-domain     
  dns-nameservice-servers    
  dns-nameservice-search     
  dns-nameservice-sortlist   
  dns-nameservice-options    
  nis-nameservice-configsrc  
  nis-nameservice-servers    
  ldap-nameservice-configsrc  
  ldap-nameservice-servers   
  default-domain             
  nfsv4-domain               
  ipfilter-config-file       "/etc/nwam/loc/NoNet/ipf.conf"
  ipfilter-v6-config-file    "/etc/nwam/loc/NoNet/ipf6.conf"
  ipnat-config-file          
  ippool-config-file         
  ike-config-file            
  ipsecpolicy-config-file    
root@solaris11-1:~#
```

Nevertheless, it can be easier to do this interactively sometimes:

```
root@solaris11-1:~# netcfg
netcfg> select ncp Automatic
netcfg:ncp:Automatic> list
ncp:Automatic
  management-type   reactive
NCUs:
  phys  net0
  phys  net1
  phys  net2
  phys  net3
  ip  net0
  ip  net1
  ip  net3
  ip  net2
netcfg:ncp:Automatic> select ncu phys net0
netcfg:ncp:Automatic:ncu:net0> list
ncu:net0
  type              link
  class             phys
  parent            "Automatic"
  activation-mode   prioritized
  enabled           true
  priority-group    0
  priority-mode     shared
netcfg:ncp:Automatic:ncu:net0> end
netcfg:ncp:Automatic> select ncu ip net0
netcfg:ncp:Automatic:ncu:net0> list
ncu:net0
  type              interface
  class             ip
  parent            "Automatic"
  enabled           true
  ip-version        ipv4,ipv6
  ipv4-addrsrc      dhcp
  ipv6-addrsrc      dhcp,autoconf
netcfg:ncp:Automatic:ncu:net0> end
netcfg:ncp:Automatic> end
netcfg> select loc Automatic
netcfg:loc:Automatic> list
loc:Automatic
  activation-mode            system
  enabled                    false
  nameservices               dns
  nameservices-config-file   "/etc/nsswitch.dns"
  dns-nameservice-configsrc  dhcp
netcfg:loc:Automatic> end
netcfg> exit

```

As we can realize, many properties can be set to customize our system. Likewise, all NCP and NCU are listed by executing the following command:

```
root@solaris11-1:~# netadm list
TYPE        PROFILE        STATE
ncp         Automatic      online
ncu:phys    net0           online
ncu:phys    net1           online
ncu:phys    net2           online
ncu:phys    net3           online
ncu:ip      net0           online
ncu:ip      net1           online
ncu:ip      net3           online
ncu:ip      net2           online
ncp         DefaultFixed   disabled
loc         Automatic      online
loc         NoNet          offline
```

If there's a demand for more details, these can be obtained by running the following command:

```
root@solaris11-1:~# netadm list -x
TYPE      PROFILE       STATE     AUXILIARY STATE
ncp       Automatic     online    active
ncu:phys  net0          online    interface/link is up
ncu:phys  net1          online    interface/link is up
ncu:phys  net2          online    interface/link is up
ncu:phys  net3          online    interface/link is up
ncu:ip    net0          online    interface/link is up
ncu:ip    net1          online    interface/link is up
ncu:ip    net3          online    interface/link is up
ncu:ip    net2          online    interface/link is up
ncp       DefaultFixed  disabled  disabled by administrator
loc       Automatic     online    active
loc       NoNet         offline   conditions for activation are unmet
```

Instead of listing all profiles (NCP and Location), it is possible to list only a class of them by running the following command:

```
root@solaris11-1:~# netadm list -p ncp
TYPE        PROFILE        STATE
ncp         Automatic      online
ncu:phys    net0           online
ncu:phys    net1           online
ncu:phys    net2           online
ncu:phys    net3           online
ncu:ip      net0           online
ncu:ip      net1           online
ncu:ip      net3           online
ncu:ip      net2           online
ncp         DefaultFixed   disabled
root@solaris11-1:~# netadm list -p loc
TYPE        PROFILE        STATE
loc         Automatic      online
loc         NoNet          offline
```

Nice! All commands have worked very well up to now. Therefore, it's time to create a new profile using the `netcfg` command. To accomplish this task, we're going to create an NCP named `hacker_profile` with two NCUs inside it, followed by a loc profile named `work`. Therefore, execute the following command:

```
root@solaris11-1:~# netcfg
netcfg> create ncp hacker_profile
netcfg:ncp:hacker_profile> create ncu phys net2
Created ncu 'net2'.  Walking properties ...
activation-mode (manual) [manual|prioritized]> manual
mac-address> [ENTER]
autopush> [ENTER]
mtu> [ENTER]
netcfg:ncp:hacker_profile:ncu:net2> list
ncu:net2
  type              link
  class             phys
  parent            "hacker_profile"
  activation-mode   manual
  enabled           true
netcfg:ncp:hacker_profile:ncu:net2> end
Committed changes
netcfg:ncp:hacker_profile> list
ncp:hacker_profile
  management-type   reactive
NCUs:
  phys  net2
netcfg:ncp:hacker_profile> create ncu ip net2
Created ncu 'net2'.  Walking properties ...
ip-version (ipv4,ipv6) [ipv4|ipv6]> ipv4
ipv4-addrsrc [dhcp|static]> static
ipv4-addr> 192.168.1.99
ipv4-default-route> 192.168.1.1
netcfg:ncp:hacker_profile:ncu:net2> list
ncu:net2
  type              interface
  class             ip
  parent            "hacker_profile"
  enabled           true
  ip-version        ipv4
  ipv4-addrsrc      static
  ipv4-addr         "192.168.1.99"
  ipv4-default-route  "192.168.1.1"
netcfg:ncp:hacker_profile:ncu:net2> commit
Committed changes
netcfg:ncp:hacker_profile:ncu:net2> end
netcfg:ncp:hacker_profile> list ncu ip net2
ncu:net2
  type              interface
  class             ip
  parent            "hacker_profile"
  enabled           true
  ip-version        ipv4
  ipv4-addrsrc      static
  ipv4-addr         "192.168.1.99"
  ipv4-default-route  "192.168.1.1"
netcfg:ncp:hacker_profile> end
netcfg> create loc work
Created loc 'work'.  Walking properties ...
activation-mode (manual) [manual|conditional-any|conditional-all]> manual
nameservices (dns) [dns|files|nis|ldap]> dns
nameservices-config-file ("/etc/nsswitch.dns")> [ENTER]
dns-nameservice-configsrc (dhcp) [manual|dhcp]> manual
dns-nameservice-domain> alexandreborges.org
dns-nameservice-servers> 192.0.80.93
dns-nameservice-search> [ENTER]
dns-nameservice-sortlist> [ENTER]
dns-nameservice-options> [ENTER]
nfsv4-domain> [ENTER]
ipfilter-config-file> [ENTER]
ipfilter-v6-config-file> [ENTER]
ipnat-config-file> [ENTER]
ippool-config-file> [ENTER]
ike-config-file> [ENTER]
ipsecpolicy-config-file> [ENTER]
netcfg:loc:work> list
loc:work
  activation-mode            manual
  enabled                    false
  nameservices               dns
  nameservices-config-file   "/etc/nsswitch.dns"
  dns-nameservice-configsrc  manual
  dns-nameservice-domain     "alexandreborges.org"
  dns-nameservice-servers    "192.0.80.93"  
netcfg:loc:work> end
Committed changes
netcfg> exit
root@solaris11-1:~#
```

List current configurations by executing the following command:

```
root@solaris11-1:~# netadm list
TYPE        PROFILE        STATE
ncp         Automatic      online
ncu:phys    net0           online
ncu:phys    net1           online
ncu:phys    net2           online
ncu:phys    net3           online
ncu:ip      net0           online
ncu:ip      net1           online
ncu:ip      net3           online
ncu:ip      net2           online
ncp         DefaultFixed   disabled
ncp         hacker_profile     disabled
loc         Automatic      online
loc         NoNet          offline
loc         work           disabled
root@solaris11-1:~# netcfg list
NCPs:
  Automatic
  DefaultFixed
  hacker_profile
Locations:
  Automatic
  NoNet
  work
root@solaris11-1:~#
root@solaris11-1:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.106/24
net1/v4           dhcp     ok           192.168.1.107/24
net2/v4           dhcp     ok           192.168.1.105/24
net3/v4           static   ok           192.168.1.140/24
```

When the new NCP and LOC profiles are enabled, everything changes. Let's check this by executing the following command:

```
root@solaris11-1:~# netadm enable work
Enabling loc 'work'
root@solaris11-1:~# netadm enable hacker_profile
Enabling ncp 'hacker_profile'
root@solaris11-1:~# netadm list
TYPE        PROFILE        STATE
ncp         Automatic      disabled
ncp         DefaultFixed   disabled
ncp         hacker_profile     online
ncu:phys    net2           online
ncu:ip      net2           online
loc         Automatic      offline
loc         NoNet          offline
loc         work           online
root@solaris11-1:~# ipadm show-addr | grep v4
lo0/v4            static   ok           127.0.0.1/8
net2/v4           static   ok           192.168.1.99/24
```

The `Automatic` NCP profile has been disabled and the loc profile `Automatic` has gone offline. Then, the `hacker_profile` NCP profile has changed to the `online` status and the `work` Loc profile has also changed to the `online` status. Additionally, all network interfaces have disappeared except `net2`, because there's only one network interface NCU configured (`net2`) in the `hacker_profile` NCP profile. The other good fact is that this configuration is persistent, and we can reboot the machine (`init 6`) and everything will continue working according to what we've configured.

If we had committed any mistake by assigning a property with a wrong value, it would be easy to correct it. For example, the name servers (the `dns-nameservice-servers` property) can be altered by executing the following command:

```
root@solaris11-1:~# netcfg
netcfg> select loc work
netcfg:loc:work> set dns-nameservice-servers="8.8.8.8,8.8.4.4"
netcfg:loc:work> list
loc:work
  activation-mode            manual
  enabled                    true
  nameservices               dns
  nameservices-config-file   "/etc/nsswitch.dns"
  dns-nameservice-configsrc  manual
  dns-nameservice-domain     "alexandreborges.org"
  dns-nameservice-servers    "8.8.8.8","8.8.4.4"
netcfg:loc:work> commit
Committed changes
netcfg:loc:work> verify
All properties verified
netcfg:loc:work> end
netcfg> end
root@solaris11-1:~#
```

After all these long tasks, it's recommend that you save the new profiles, `hacker_profile` and `work`. Therefore, to make a backup of them, execute the following commands:

```
root@solaris11-1:~# mkdir /backup
root@solaris11-1:~# netcfg export -f /backup/hacker_profile_bkp ncp hacker_profile
root@solaris11-1:~# netcfg export -f /backup/work_bkp loc work
@solaris11-1:~# more /backup/hacker_profile_bkp
create ncp "hacker_profile"
create ncu phys "net2"
set activation-mode=manual
end
create ncu ip "net2"
set ip-version=ipv4
set ipv4-addrsrc=static
set ipv4-addr="192.168.1.99/24"
set ipv4-default-route="192.168.1.1"
end
end
root@solaris11-1:~# more /backup/work_bkp
create loc "work"
set activation-mode=manual
set nameservices=dns
set nameservices-config-file="/etc/nsswitch.dns"
set dns-nameservice-configsrc=manual
set dns-nameservice-domain="alexandreborges.org"
set dns-nameservice-servers="8.8.8.8","8.8.4.4"
end
root@solaris11-1:~#
```

Reverting the system to the old `Automatic` profiles (NCP and Loc) can be done by running the following command:

```
root@solaris11-1:~# netadm enable -p ncp Automatic
Enabling ncp 'Automatic'
root@solaris11-1:~# netadm enable -p loc Automatic
Enabling loc 'Automatic'
root@solaris11-1:~# netadm list | grep Automatic
ncp         Automatic      online
loc         Automatic      online
root@solaris11-1:~#
```

Finally, it would be appropriate to destroy the created NCP and loc profiles by executing the following commands:

```
root@solaris11-1:~# netcfg destroy loc work
root@solaris11-1:~# netcfg destroy ncp hacker_profile

```

Oracle Solaris 11 is terrific!

### An overview of the recipe

There is no doubt that RNC makes the life of an administrator easier. Administration, configuration, and monitoring are done through the command line and everything is configured using only two commands: `netadm` and `netcfg`. The `netadm` command role enables, disables, and lists profiles, while the `netcfg` command role creates profile configurations.

# Internet Protocol Multipathing

**Internet Protocol Multipathing** (**IPMP**) is a great technology that was introduced a long time ago (originally in Oracle Solaris 8), and since then, it has been improving a lot up to the current Oracle Solaris 11\. In a general way, IPMP offers fault-tolerance for the network interfaces scheme, thus eliminating any single point of failure. Moreover, it provides an increase in the network bandwidth for outbound traffic by spreading the load over all active interfaces in the same group. This is our start point; to play with IPMP, an IPMP group interface must be created and all of the data IP addresses should be assigned to this IPMP group interface. Therefore, at the end, all network interfaces that will be used with IPMP must have an IPMP group assigned.

To continue the explanation, the following is a quick example:

*   Group interface: `hacker_ipmp0`

    *   Interface 1: `net0`

        test IP (`test_net0`): `192.168.1.61`

    *   Interface 2: `net1`

        test IP (`test_net1`): `192.168.1.71`

In the previous example, we have two interfaces (`net0` and `net1`) that are used to send/receive the normal application data as usual. Nevertheless, the data IP addresses aren't assigned to the `net0` or `net1` interfaces, but they are assigned to the IPMP group interface that contains both physical network interfaces. The test IP addresses from the `net0` and `net1` interfaces (`192.168.1.61` and `192.168.1.71`, respectively) are used by the `in.mpathd` IPMP daemon to check whether the interface is healthy.

There are two possible configurations when deploying IPMP: active-active and active-passive. The former configuration works with all interfaces that transmit data, and the latter scheme works with at least one spare interface. Most of the time, you will see companies work with the active-active configuration.

What's the basic idea of IPMP? If one interface fails (or the cable is disconnected), the system continues transmitting and receiving data without any problems. Why? Because in the IPMP group, there is more than one interface that accomplishes the network job, and if any of them fails, any other interface resumes the work.

Can IPMP monitor the interface using the assigned data IP address? No, it can't; because, if `in.mpathd` used the data IP address to monitor the interface, there could be a delay in the monitoring process. By the way, is the test IP address necessary? It isn't, really. The IPMP has two monitoring methods: probe-based detection (using a test IP address) and link-based (if it's supported by the interface). Personally, I like probe-based monitoring (using a test IP address) because I've already faced some problems with the link-based method, and I think probe-based monitoring is more reliable. However, if the interface supports the link-based method, then both methods will be used. Anyway, when using probed monitoring, the `in.mpathd` daemon continues to monitor the failed interface to check when it comes alive again.

Finishing the theory, the active-standby configuration is very similar to active-active, but the standby interface doesn't transmit any data packets while the active network interfaces are good and working. If any active network interfaces go to the `failed` status, the standby network interface will be activated, and it will start to send data packets.

## Getting ready

This recipe requires two virtual machines (VirtualBox or VMware Workstation) with Oracle Solaris 11 installed, 4 GB memory, and four network interfaces in the first virtual machine. For the second virtual machine, just one interface is enough.

## How to do it…

This recipe will be based on a similar scenario presented previously, but four interfaces will be used where all of them are active:

*   Group: `hacker_ipmp0`

    *   Data IP addresses: `192.168.1.50`, `192.168.1.60`, `192.168.1.70`, and `192.168.1.80`
    *   Interface 1: `net0`

        test IP (`test_net0`): `192.168.1.51`

    *   Interface 2: `net1`

        test IP (`test_net1`): `192.168.1.61`

    *   Interface 3: `net2`

        test IP (`test_net2`): `192.168.1.71`

    *   Interface 4: `net3`

        test IP (`test_net3`): `192.168.1.81`

Like every feature in Oracle Solaris 11, IPMP is based on a **Service Management Facility** (**SMF**) service that must be online (default) and can be verified by running the following command:

```
root@solaris11-1:~# svcs -a | grep ipmp
online         23:38:50 svc:/network/ipmp:default
root@solaris11-1:~# svcs -l ipmp
fmri         svc:/network/ipmp:default
name         IP Multipathing
enabled      true
state        online
next_state   none
state_time   January  9, 2014 11:38:50 PM BRST
alt_logfile  /system/volatile/network-ipmp:default.log
restarter    svc:/system/svc/restarter:default
contract_id  19
manifest     /lib/svc/manifest/network/network-ipmp.xml
dependency   require_all/none svc:/network/loopback (online)
```

Moreover, the behavior of the IPMP daemon is based on the `mpathd` configuration file that is in the `default` directory under `/etc/`. Additionally, this configuration file has default content that covers any usual environment that does not demand any special care with delay in responses. Execute the following command:

```
root@solaris11-1:~# more /etc/default/mpathd
#
# Copyright 2000 Sun Microsystems, Inc.  All rights reserved.
# Use is subject to license terms.
#
# ident  "%Z%%M%  %I%  %E% SMI"
#
# Time taken by mpathd to detect a NIC failure in ms. The minimum time
# that can be specified is 100 ms.
#
FAILURE_DETECTION_TIME=10000
#
# Failback is enabled by default. To disable failback turn off this option
#
FAILBACK=yes
#
# By default only interfaces configured as part of multipathing groups
# are tracked. Turn off this option to track all network interfaces
# on the system
#
TRACK_INTERFACES_ONLY_WITH_GROUPS=yes
root@solaris11-1:~#
```

Well, it's time to move forward. Initially, let's list what interfaces are available and their respective status by executing the following command:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.106/24
net1/v4           dhcp     ok           192.168.1.107/24
net2/v4           dhcp     ok           192.168.1.99/24
net3/v4           dhcp     ok           192.168.1.140/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
net1       ip       ok       yes    --
net2       ip       ok       yes    --
net3       ip       ok       yes    --
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
```

In the following step, all IP address objects will be deleted:

```
root@solaris11-1:~# ipadm delete-ip net0
root@solaris11-1:~# ipadm delete-ip net1
root@solaris11-1:~# ipadm delete-ip net2
root@solaris11-1:~# ipadm delete-ip net3

```

Returning to the monitoring commands, we shouldn't see all these IP address objects anymore:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
```

Everything is okay up to now. Thus, before starting to configure IPMP, it's appropriate to change the NCP profile from `Automatic` to `DefaultFixed` because the IPMP setup is going to use fixed IP addresses:

```
root@solaris11-1:~# netadm list
TYPE        PROFILE        STATE
ncp         Automatic      online
ncu:phys    net0           online
ncu:phys    net1           online
ncu:phys    net2           online
ncu:phys    net3           online
ncp         my_profile     disabled
ncp         DefaultFixed   disabled
loc         NoNet          online
loc         work           disabled
loc         Automatic      offline
root@solaris11-1:~# netadm enable -p ncp DefaultFixed
Enabling ncp 'DefaultFixed'
```

Great! It's interesting to realize that there is no IP address object on the system:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
```

The game begins. To make the administration more comfortable, all network links are going to be renamed for them to be more easily recognizable, and shortly thereafter, new IP address objects will be created too (for a while, without any IP address value):

```
root@solaris11-1:~# dladm rename-link net0 net0_myipmp0
root@solaris11-1:~# dladm rename-link net1 net1_myipmp1
root@solaris11-1:~# dladm rename-link net2 net2_myipmp2
root@solaris11-1:~# dladm rename-link net3 net3_myipmp3
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0_myipmp0        phys      1500   unknown  --
net1_myipmp1        phys      1500   unknown  --
net2_myipmp2        phys      1500   unknown  --
net3_myipmp3        phys      1500   unknown  --
root@solaris11-1:~# ipadm create-ip net0_myipmp0
root@solaris11-1:~# ipadm create-ip net1_myipmp1
root@solaris11-1:~# ipadm create-ip net2_myipmp2
root@solaris11-1:~# ipadm create-ip net3_myipmp3
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0_myipmp0 ip     down     no     --
net1_myipmp1 ip     down     no     --
net2_myipmp2 ip     down     no     --
net3_myipmp3 ip     down     no     --
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
```

Now, it's time to create the IPMP interface group (`hacker_ipmp0`) and assign all interfaces to this group. Pay attention to the fact that there are no IP addresses on any network interface yet:

```
root@solaris11-1:~# ipadm create-ipmp hacker_ipmp0
root@solaris11-1:~# ipadm add-ipmp -i net0_myipmp0 -i net1_myipmp1 -i net2_myipmp2 -i net3_myipmp3  hacker_ipmp0

```

```
down (see the ipadm show-if and ipmpstat -a commands in the following snippet) for now (wait for more steps):
```

```
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0_myipmp0 ip     ok       yes    --
net1_myipmp1 ip     ok       yes    --
net2_myipmp2 ip     ok       yes    --
net3_myipmp3 ip     ok       yes    --
hacker_ipmp0 ipmp   down     no     net0_myipmp0 net1_myipmp1 net2_myipmp2 net3_myipmp3
root@solaris11-1:~# ipmpstat –g
GROUP       GROUPNAME   STATE     FDT       INTERFACES
hacker_ipmp0 hacker_ipmp0 ok      --        net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
root@solaris11-1:~# ipmpstat -a
ADDRESS                  STATE  GROUP        INBOUND     OUTBOUND
::                       down   hacker_ipmp0  --         --
0.0.0.0                  down   hacker_ipmp0  --         --
```

Because there is no data or test IP address yet, all probe operations are disabled:

```
root@solaris11-1:~# ipmpstat -i
INTERFACE   ACTIVE  GROUP       FLAGS     LINK      PROBE     STATE
net3_myipmp3 yes    hacker_ipmp0 -------  up        disabled  ok
net2_myipmp2 yes    hacker_ipmp0 -------  up        disabled  ok
net1_myipmp1 yes    hacker_ipmp0 -------  up        disabled  ok
net0_myipmp0 yes    hacker_ipmp0 --mbM--  up        disabled  ok
root@solaris11-1:~# ipmpstat -p
ipmpstat: probe-based failure detection is disabled
```

Finally, all main data IP addresses and test IP addresses will be added to the IPMP configuration by executing the following commands:

```
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.50/24 hacker_ipmp0/v4addr1
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.60/24 hacker_ipmp0/v4addr2
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.70/24 hacker_ipmp0/v4addr3
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.80/24 hacker_ipmp0/v4addr4
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.51/24 net0_myipmp0/test
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.61/24 net1_myipmp1/test
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.71/24 net2_myipmp2/test
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.81/24 net3_myipmp3/test

```

To check whether our previous `ipadm` commands are working, execute the following command:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0_myipmp0/test static   ok           192.168.1.51/24
net1_myipmp1/test static   ok           192.168.1.61/24
net2_myipmp2/test static   ok           192.168.1.71/24
net3_myipmp3/test static   ok           192.168.1.81/24
hacker_ipmp0/v4addr1 static ok          192.168.1.50/24
hacker_ipmp0/v4addr2 static ok          192.168.1.60/24
hacker_ipmp0/v4addr3 static ok          192.168.1.70/24
hacker_ipmp0/v4addr4 static ok          192.168.1.80/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0_myipmp0 ip     ok       yes    --
net1_myipmp1 ip     ok       yes    --
net2_myipmp2 ip     ok       yes    --
net3_myipmp3 ip     ok       yes    --
hacker_ipmp0 ipmp   ok       yes    net0_myipmp0 net1_myipmp1 net2_myipmp2 net3_myipmp3
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0_myipmp0        phys      1500   up       --
net1_myipmp1        phys      1500   up       --
net2_myipmp2        phys      1500   up       --
net3_myipmp3        phys      1500   up       --
```

If everything went well, the IPMP interface group and all IP addresses should be `ok` and `up`:

```
root@solaris11-1:~# ipmpstat -g
GROUP       GROUPNAME   STATE     FDT       INTERFACES
hacker_ipmp0 hacker_ipmp0 ok      10.00s    net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
root@solaris11-1:~# ipmpstat -a
ADDRESS                   STATE  GROUP       INBOUND     OUTBOUND
::                        down   hacker_ipmp0 --         --
192.168.1.80              up     hacker_ipmp0 net0_myipmp0 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.70              up     hacker_ipmp0 net1_myipmp1 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.60              up     hacker_ipmp0 net2_myipmp2 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.50              up     hacker_ipmp0 net3_myipmp3 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
```

Thanks to each test IP address, all interfaces should be being monitored by the `in.mpathd` daemon (from the IPMP service), and this probe information is shown by executing the following command:

```
root@solaris11-1:~# ipmpstat -p
TIME     INTERFACE   PROBE  NETRTT    RTT      RTTAVG   TARGET
0.21s    net0_myipmp0 i1411 0.66ms    0.85ms   0.70ms   192.168.1.113
0.47s    net3_myipmp3 i1411 0.55ms    7.57ms   2.31ms   192.168.1.113
0.70s    net2_myipmp2 i1411 0.67ms    0.77ms   0.72ms   192.168.1.112
1.13s    net1_myipmp1 i1412 0.43ms    0.60ms   0.73ms   192.168.1.112
1.78s    net0_myipmp0 i1412 0.63ms    0.74ms   1.00ms   192.168.1.112
2.17s    net3_myipmp3 i1412 0.68ms    0.82ms   0.65ms   192.168.1.112
2.43s    net2_myipmp2 i1412 0.31ms    0.36ms   0.67ms   192.168.1.113
2.94s    net0_myipmp0 i1413 7.17ms    8.03ms   11.05ms  192.168.1.188
2.99s    net1_myipmp1 i1413 0.27ms    0.31ms   1.11ms   192.168.1.113
3.54s    net3_myipmp3 i1413 0.57ms    0.69ms   2.10ms   192.168.1.113
3.69s    net2_myipmp2 i1413 0.61ms    0.72ms   0.72ms   192.168.1.112
^C
```

You might notice some strange IPs: `192.168.1.112`, `192.168.1.113`, and `192.168.1.188`. Where do these addresses come from? The IPMP service makes tests and checks (probes) to assure that the data IPs are working as expected by using the multicast protocol, and it registers the RTT (round trip) for a packet to go and return from a discovered host. In this particular case, IPMP has reached some machines on my private local network and a printer.

Therefore, according to the previous command, it is possible to confirm whether all IPMP network interfaces are good by executing the following commands:

```
root@solaris11-1:~# ipmpstat -i
INTERFACE   ACTIVE  GROUP       FLAGS     LINK      PROBE     STATE
net3_myipmp3 yes    hacker_ipmp0 -------  up        ok        ok
net2_myipmp2 yes    hacker_ipmp0 -------  up        ok        ok
net1_myipmp1 yes    hacker_ipmp0 -------  up        ok        ok
net0_myipmp0 yes    hacker_ipmp0 --mbM--  up        ok        ok
```

These flags from the `ipmpstat –i` command deserve a quick explanation:

*   `m`: This is to send and/or receive IPv4 multicast packets
*   `M`: This is to send and/or receive IPv6 multicast packets
*   `b`: This is chosen to send and/or receive IPv4 broadcast packets
*   `i`: This means inactive
*   `s`: This means standby
*   `d`: This means down

Likewise, information about test IP addresses and hosts that were used to send multicast packets are presented in a simple way, as follows:

```
root@solaris11-1:~# ipmpstat -t
INTERFACE   MODE       TESTADDR            TARGETS
net3_myipmp3 multicast 192.168.1.81        192.168.1.113 192.168.1.112
net2_myipmp2 multicast 192.168.1.71        192.168.1.112 192.168.1.113
net1_myipmp1 multicast 192.168.1.61        192.168.1.113 192.168.1.112
net0_myipmp0 multicast 192.168.1.51        192.168.1.112 192.168.1.188 192.168.1.113
```

Excellent! Is it over? No. How can we know whether the IPMP configuration is working? The best way is to make a network fail. To simulate this scenario, we must first shut down Oracle Solaris 11 by executing the following command:

```
root@solaris11-1:~# shutdown –y –g0

```

In the next step, we must choose our virtual machine, click on the **Settings** button, and go to **Network**. There, for the **Attached to** option, change the first interface to **Not attached**.

![How to do it…](img/00011.jpeg)

This trick will simulate a failure on the interface and the interface won't be presented for Oracle Solaris 11\. Then, the virtual machine (`solaris11-1`) must be turned on again, and as expected, the system works very well. This can be confirmed by using all the previous network and IPMP commands:

```
root@solaris11-1:~# ipmpstat -pn
TIME     INTERFACE   PROBE  NETRTT   RTT      RTTAVG    TARGET
0.08s    net2_myipmp2 i761  0.22ms   0.31ms   0.56ms    192.168.1.113
1.31s    net1_myipmp1 i762  3.90ms   4.02ms   9.35ms    192.168.1.188
1.37s    net3_myipmp3 i761  0.48ms   0.57ms   0.83ms    192.168.1.113
1.57s    net2_myipmp2 i762  0.32ms   0.38ms   0.61ms    192.168.1.113
2.79s    net1_myipmp1 i763  0.63ms   0.73ms   0.78ms    192.168.1.113
2.85s    net3_myipmp3 i762  0.66ms   0.78ms   0.72ms    192.168.1.113
1.11s    net0_myipmp0 i763  --       --       --        192.168.1.113
-0.03s   net0_myipmp0 i762  --       --       --        192.168.1.188
3.08s    net2_myipmp2 i763  0.57ms   0.70ms   0.57ms    192.168.1.113
4.02s    net3_myipmp3 i763  0.58ms   0.69ms   0.82ms    192.168.1.113
```

As expected, the first interface (`net0_myipmp0`) fails during the probe test. Moving forward, the same failure will be shown in other commands:

```
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0_myipmp0 ip     failed   no     --
net1_myipmp1 ip     ok       yes    --
net2_myipmp2 ip     ok       yes    --
hacker_ipmp0 ipmp   ok       yes    net0_myipmp0 net1_myipmp1 net2_myipmp2 net3_myipmp3
net3_myipmp3 ip     ok       yes    --
root@solaris11-1:~# ipmpstat -g
GROUP       GROUPNAME   STATE     FDT       INTERFACES
hacker_ipmp0 hacker_ipmp0 degraded 10.00s   net3_myipmp3 net2_myipmp2 net1_myipmp1 [net0_myipmp0]

```

The IPMP group status is degraded because one of its interfaces (`net0_myipmp0`) is missing. Other IPMP commands can confirm this fact:

```
root@solaris11-1:~# ipmpstat -i
INTERFACE   ACTIVE  GROUP       FLAGS     LINK      PROBE     STATE
net3_myipmp3 yes    hacker_ipmp0 -------  up        ok        ok
net2_myipmp2 yes    hacker_ipmp0 -------  up        ok        ok
net1_myipmp1 yes    hacker_ipmp0 --mbM--  up        ok        ok
net0_myipmp0 no     hacker_ipmp0 -------  up        failed    failed
root@solaris11-1:~# ipmpstat -a
ADDRESS                   STATE  GROUP       INBOUND     OUTBOUND
::                        down   hacker_ipmp0 --         --
192.168.1.80              up     hacker_ipmp0 net1_myipmp1 net3_myipmp3 net2_myipmp2 net1_myipmp1
192.168.1.70              up     hacker_ipmp0 net3_myipmp3 net3_myipmp3 net2_myipmp2 net1_myipmp1
192.168.1.60              up     hacker_ipmp0 net2_myipmp2 net3_myipmp3 net2_myipmp2 net1_myipmp1
192.168.1.50              up     hacker_ipmp0 net1_myipmp1 net3_myipmp3 net2_myipmp2 net1_myipmp1
```

Take care—on the first view, it could seem that there's something wrong, but in fact, there isn't. It's usual for some people to guess that the IP address is bound to a specific interface, but this isn't true. All data IP addresses are assigned to the IPMP group interface, and IPMP will try to use the best interface for outbound connections. Nonetheless, the best and final test can be performed using another machine (`solaris11-2`), and from there, try to ping all data IP addresses from the first machine (`solaris11-1`):

```
root@solaris11-2:~# ping 192.168.1.50
192.168.1.50 is alive
root@solaris11-2:~# ping 192.168.1.60
192.168.1.60 is alive
root@solaris11-2:~# ping 192.168.1.70
192.168.1.70 is alive
root@solaris11-2:~# ping 192.168.1.80
192.168.1.80 is alive
```

Amazing! Oracle Solaris 11 wins again! If we shut down the first virtual machine once more (`shutdown –y –g0` or `poweroff`), return the interface to its old configuration (**Settings** | **Network** | **Adapter 1** | **Attached to: Bridged Network**) and turn on the `solaris11-1` virtual machine again; we're going to confirm that everything is `ok`:

```
root@solaris11-1:~# ipmpstat -i
INTERFACE   ACTIVE  GROUP       FLAGS     LINK      PROBE     STATE
net3_myipmp3 yes    hacker_ipmp0 -------  up        ok        ok
net2_myipmp2 yes    hacker_ipmp0 -------  up        ok        ok
net1_myipmp1 yes    hacker_ipmp0 -------  up        ok        ok
net0_myipmp0 yes    hacker_ipmp0 --mbM--  up        ok        ok
root@solaris11-1:~# ipmpstat -g
GROUP       GROUPNAME   STATE     FDT       INTERFACES
hacker_ipmp0 hacker_ipmp0 ok      10.00s    net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
root@solaris11-1:~# ipmpstat -a
ADDRESS                   STATE  GROUP       INBOUND     OUTBOUND
::                        down   hacker_ipmp0 --         --
192.168.1.80              up     hacker_ipmp0 net1_myipmp1 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.70              up     hacker_ipmp0 net3_myipmp3 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.60              up     hacker_ipmp0 net2_myipmp2 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.50              up     hacker_ipmp0 net0_myipmp0 net3_myipmp3 net2_myipmp2 net1_myipmp1 net0_myipmp0
```

Fantastic! However, let's execute another test. The goal is to convert an active interface into a standby interface (the active-passive configuration). Thus, to proceed, we should delete one of the IP addresses that carries data and is assigned to a standby network interface. If it's not deleted, it wouldn't make any difference. Relax! The following procedure is a piece of cake.

The first step is to change the `standby` property from the interface to `on` by running the following command:

```
root@solaris11-1:~# ipadm set-ifprop -p standby=on -m ip net3_myipmp3

```

Check whether the last command worked as expected by executing the following command:

```
root@solaris11-1:~# ipadm show-ifprop -p standby net3_myipmp3
IFNAME       PROPERTY  PROTO PERM CURRENT PERSISTENT DEFAULT POSSIBLE
net3_myipmp3 standby   ip    rw   on      on         off      on,off
```

As we've mentioned, a data IP address object (the forth) will be deleted by running the following command:

```
root@solaris11-1:~# ipadm delete-addr hacker_ipmp0/v4addr4

```

The `net3_myipmp3` interface is marked as deleted (its respective interface is put inside the parentheses):

```
root@solaris11-1:~# ipmpstat -g
GROUP       GROUPNAME   STATE     FDT       INTERFACES
hacker_ipmp0 hacker_ipmp0 ok      10.00s    net2_myipmp2 net1_myipmp1 net0_myipmp0 (net3_myipmp3)
```

Check whether the `net3_myipmp3` interface doesn't appear anymore by running the following three commands:

```
root@solaris11-1:~# ipmpstat -a
ADDRESS                   STATE  GROUP       INBOUND     OUTBOUND
::                        down   hacker_ipmp0 --         --
192.168.1.80              up     hacker_ipmp0 net1_myipmp1 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.70              up     hacker_ipmp0 net0_myipmp0 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.60              up     hacker_ipmp0 net2_myipmp2 net2_myipmp2 net1_myipmp1 net0_myipmp0
192.168.1.50              up     hacker_ipmp0 net0_myipmp0 net2_myipmp2 net1_myipmp1 net0_myipmp0
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0_myipmp0 ip     ok       yes    --
net1_myipmp1 ip     ok       yes    --
net2_myipmp2 ip     ok       yes    --
hacker_ipmp0 ipmp   ok       yes    net0_myipmp0 net1_myipmp1 net2_myipmp2 net3_myipmp3
net3_myipmp3 ip     ok       no     --
root@solaris11-1:~# ipmpstat -i

INTERFACE   ACTIVE  GROUP       FLAGS     LINK      PROBE     STATE
net3_myipmp3 no     hacker_ipmp0 is-----  up        ok        ok
net2_myipmp2 yes    hacker_ipmp0 -------  up        ok        ok
net1_myipmp1 yes    hacker_ipmp0 -------  up        ok        ok
net0_myipmp0 yes    hacker_ipmp0 --mbM--  up        ok        ok
```

Notice that the `is` flag on `net3_myipmp3` describes this interface as inactive and working in the standby mode. All tests can be performed in the same way using this active-passive scenario.

Last but not least, we need to return everything as it was before this section in order to prepare for the next section, which explains how to set up link aggregation:

```
root@solaris11-1:~# ipadm remove-ipmp hacker_ipmp0 -i net0_myipmp0 -i net1_myipmp1 -i net2_myipmp2 -i net3_myipmp3
root@solaris11-1:~# ipadm delete-ipmp hacker_ipmp0
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0_myipmp0/test static   ok           192.168.1.51/24
net1_myipmp1/test static   ok           192.168.1.61/24
net2_myipmp2/test static   ok           192.168.1.71/24
net3_myipmp3/test static   ok           192.168.1.81/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm delete-addr net0_myipmp0/test
root@solaris11-1:~# ipadm delete-addr net1_myipmp1/test
root@solaris11-1:~# ipadm delete-addr net2_myipmp2/test
root@solaris11-1:~# ipadm delete-addr net3_myipmp3/test
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0_myipmp0 ip     down     no     --
net1_myipmp1 ip     down     no     --
net2_myipmp2 ip     down     no     --
net3_myipmp3 ip     down     no     --
root@solaris11-1:~# ipadm delete-ip net0_myipmp0
root@solaris11-1:~# ipadm delete-ip net1_myipmp1
root@solaris11-1:~# ipadm delete-ip net2_myipmp2
root@solaris11-1:~# ipadm delete-ip net3_myipmp3
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0_myipmp0        phys      1500   unknown  --
net1_myipmp1        phys      1500   unknown  --
net2_myipmp2        phys      1500   unknown  --
net3_myipmp3        phys      1500   unknown  --
root@solaris11-1:~# dladm rename-link net0_myipmp0 net0
root@solaris11-1:~# dladm rename-link net1_myipmp1 net1
root@solaris11-1:~# dladm rename-link net2_myipmp2 net2
root@solaris11-1:~# dladm rename-link net3_myipmp3 net3
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   unknown  --
net1                phys      1500   unknown  --
net2                phys      1500   unknown  --
net3                phys      1500   unknown  --
root@solaris11-1:~# netadm enable -p ncp Automatic
Enabling ncp 'Automatic'
root@solaris11-1:~# ipadm show-addr | grep v4
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.108/24
net1/v4           dhcp     ok           192.168.1.106/24
net2/v4           dhcp     ok           192.168.1.109/24
net3/v4           dhcp     ok           192.168.1.107/24
```

We are done with IPMP! Oracle Solaris 11 is the best operating system in the world!

### An overview of the recipe

The main concept that must always be remembered is that the IPMP frame is suitable for eliminating a single point of failure. Although it is able to create the outbound load balance, the real goal is the high availability network.

# Setting the link aggregation

As a rough comparison, we could think about link aggregation (802.3ad LACP) as a network technology layer 2 (Datalink), which acts as the inverse of IPMP (network technology layer 3: IP). While IPMP is concerned with offering network interface fault tolerance—eliminating a single point of failure and offering a higher outbound throughput as a bonus—link aggregation works as the old "trunk" product from previous versions of Oracle Solaris and offers a high throughput for the network traffic and, as a bonus, also provides a fault tolerance feature so that if a network interface fails, the traffic isn't interrupted.

Summarizing the facts:

*   IPMP is recommended for fault tolerance, but it offers some output load balance
*   Link aggregation is recommended for increasing the throughput, but it also offers fault tolerance

The link aggregation feature puts two or more network interfaces together and administers all of them as a single unit. Basically, link aggregation presents performance advantages, but all links must have the same speed, working in full duplex and point-to-point modes. An example of aggregation is **Aggregation_1** | net0, net1, net2, and net3.

At the end, there's only one logic object (Aggregation_1) that was created on the underlying four network interfaces (net0, net1, net2, and net3). These are shown as a single interface, summing the strengths (high throughput, for example) and keeping them hidden. Nonetheless, a question remains: how are the outgoing packets delivered and balanced over the interfaces?

An answer to this question is named Aggregation and Load Balance Policies, which determine the outgoing link by hashing some values (properties) and are enumerated as follows:

*   **L2 (Networking)**: In this, the outgoing interface is chosen by hashing the MAC header of each packet.
*   **L3 (Addressing)**: In this, the outgoing interface is chosen by hashing the IP header of each packet.
*   **L4 (Communication)**: In this, the outgoing interface is chosen by hashing the UDP and TCP header of each packet. This is the default policy. A very important note is that this policy gives the best performance, but it isn't supported across all systems and it isn't fully 802.3ad-compliant in situations where the switch device can be a restrictive factor. Additionally, if the aggregation scheme is connected to a switch, then the **Link Aggregation Control Protocol** (**LACP**) must be supported by the physical switch and aggregation, given that the aggregation can be configured with the following values:

    *   **off**: This is the default mode for the aggregation
    *   **active**: This is the mode where the aggregation is configured and where it generates LACP Data Units at regular intervals
    *   **passive**: This is the mode where the aggregation is configured and only generates LACP Data Units when it receives one from the switch, obliging both sides (the aggregation and switch) to be set up using the passive mode

The only disadvantage of normal link aggregation (known as trunk link aggregation) is that it can't span across multiple switches and is limited to working with only one switch. To overcome this, there's another technique of aggregation that can span over multiple switches named **Data Link Multipathing** (**DLMP**) aggregation. To understand DLMP aggregation, imagine a scenario where we have the following in the same system:

*   Zone 1 with vnicA, vnicB, and vnicC virtual interfaces, which are connected to NIC1
*   Zone 2 with vnicD and vnicE virtual interfaces, where both of them are connected to NIC2
*   NIC1 is connected to **Switch1** (**SW1**)
*   NIC2 is connected to **Switch2** (**SW2**)

The following is another way of representing this:

*   Zone1 | vnicA,vnicB,vnicC | NIC1 | SW1
*   Zone 2 | vnicD,vnicE | NIC2 | SW2

Using trunk link aggregation, if the NIC1 network interface went to `down`, the system could still fail over all traffic to NIC2, and there wouldn't be any problem if both NIC1 and NIC2 were connected to the same switch (this isn't the case).

However, in this case, everything is worse because there are two switches connected to the same system. What would happen if Switch1 had gone down? This could be a big problem because Zone1 would be isolated. Trunk link aggregation doesn't support spanning across switches; therefore, there wouldn't be any possibility of failing over to another switch (Switch2). Concisely, Zone1 would lose network access.

This is a perfect situation to use DLMP aggregation because it is able to span across multiple switches without requiring any special configuration performed in the switches (this is only necessary when both are in the same broadcast domain). Even if the **Switch1** (**SW1**) port goes to `down`, Oracle Solaris 11 is able to fail over all the vnicA, vnicB, and vnicC flow from Zone1 to NIC2, which uses a different switch (SW2) port. Briefly, Zone1 doesn't lose access to the network.

## Getting ready

To follow this recipe, you must have two virtual machines (VirtualBox or VMware) with Oracle Solaris 11 installed and have 4 GB RAM and four network interfaces in the first virtual machine. The second machine can have just one network interface.

## How to do it…

Let's see what we have in the system by executing the following command:

```
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
net1       ip       ok       yes    --
net2       ip       ok       yes    --
net3       ip       ok       yes    --
root@solaris11-1:~# ipadm show-addr| grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.108/24
net1/v4           dhcp     ok           192.168.1.106/24
net2/v4           dhcp     ok           192.168.1.109/24
net3/v4           dhcp     ok           192.168.1.107/24
```

There are four interfaces that get their IP address from a local DHCP service. Therefore, to configure the link aggregation, it's necessary to delete all IP object addresses from all interfaces and verify their status by running the following commands:

```
root@solaris11-1:~# ipadm delete-ip net0
root@solaris11-1:~# ipadm delete-ip net1
root@solaris11-1:~# ipadm delete-ip net2
root@solaris11-1:~# ipadm delete-ip net3
root@solaris11-1:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
```

Nice. Everything is working. This time, the link aggregation (the trunk link aggregation) can be set up. Let's take all of the interfaces to create the aggregation by running the following command:

root@solaris11-1:~# **dladm create-aggr -l net0 -l net1 -l net2 -l net3 super_aggr_0**

To check whether the aggregation was created, execute the following command:

```
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
super_aggr_0         aggr      1500   up       net0 net1 net2 net3

```

More details about the aggregation can be gathered by executing the following command:

```
root@solaris11-1:~# dladm show-aggr
LINK            MODE  POLICY   ADDRPOLICY      LACPACTIVITY LACPTIMER
super_aggr_0    trunk L4       auto            off          short
```

The `super_aggr_0` aggregation was created, and it works like a single network interface. As we mentioned previously, the default aggregation type is `trunk` and the default policy is L4 (Communication). For curiosity, if we wanted to create a DMLP link aggregation, the command would be as follows:

```
root@solaris11-1:~# dladm create-aggr –m dlmp -l net0 -l net1 -l net2 -l net3 super_aggr_0

```

Now, it's time to create an IP object on it:

```
root@solaris11-1:~# ipadm create-ip super_aggr_0
root@solaris11-1:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
```

The `super_aggr_0` aggregation is still down because no IP address is assigned to it:

```
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
super_aggr_0 ip     down     no     --

```

However, everything is `ok` at the layer 2 level (Datalink):

```
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
super_aggr_0         aggr      1500   up       net0 net1 net2 net3

```

Great! The definitive step is to assign an IP address to the aggregation object, which is `super_aggr_0`:

```
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.166/24 super_aggr_0/v4
root@solaris11-1:~# ipadm show-if
IFNAME       CLASS    STATE    ACTIVE OVER
lo0          loopback ok       yes    --
super_aggr_0 ip       ok       yes    --
```

As we've learned previously, all interfaces are hidden and only the link aggregation interface is shown and presented to an external network. To collect more information about the aggregation, run the following command:

```
root@solaris11-1:~# dladm show-aggr
LINK            MODE  POLICY   ADDRPOLICY      LACPACTIVITY LACPTIMER
super_aggr_0    trunk L4       auto            off          short
root@solaris11-1:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
super_aggr_0/v4   static   ok           192.168.1.166/24
```

A recommended way to verify whether everything is working is to try to send and receive packets:

```
root@solaris11-1:~# ping 192.168.1.1
192.168.1.1 is alive
```

We can also monitor the link aggregation activity by using the `netstat` command:

```
root@solaris11-1:~# netstat -i -I super_aggr_0 -f inet
Name  Mtu  Net/Dest      Address        Ipkts  Ierrs Opkts  Oerrs Collis Queue
super_aggr_0 1500 192.168.1.0   192.168.1.166  32745  0     243    0     0      0  
root@solaris11-1:~# netstat -rn -f inet
Routing Table: IPv4
Destination        Gateway           Flags  Ref     Use     Interface
------------------ ----------------- ------ ------- ------- ---------
127.0.0.1          127.0.0.1         UH     2       8066    lo0       
192.168.1.0        192.168.1.166     U      3       28   super_aggr_0

```

We have almost finished our learning (not yet!). To change the link aggregation policy (for example, from L4 to L2), we execute the following command:

```
root@solaris11-1:~# dladm show-aggr
LINK            MODE  POLICY  ADDRPOLICY       LACPACTIVITY LACPTIMER
super_aggr_0    trunk L4      auto             off          short
root@solaris11-1:~# dladm modify-aggr --policy=L2 super_aggr_0
root@solaris11-1:~# dladm show-aggr
LINK          MODE  POLICY   ADDRPOLICY        LACPACTIVITY LACPTIMER
super_aggr_0  trunk L2       auto              off          short
```

Our example of link aggregation was created using four interfaces. However, an interface can be either inserted or removed anytime. First, we have to know which interfaces are part of the aggregation by running the following command:

```
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
super_aggr_0        aggr      1500   up       net0 net1 net2 net3

```

Now, it's easy to remove an interface from aggregation by executing the following command:

```
root@solaris11-1:~# dladm remove-aggr -l net3 super_aggr_0

```

To confirm that the previous command worked, run the following command:

```
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
super_aggr_0        aggr      1500   up       net0 net1 net2

```

Adding an interface follows almost the same syntax, as follows:

```
root@solaris11-1:~# dladm add-aggr -l net3 super_aggr_0
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
super_aggr_0        aggr      1500   up       net0 net1 net2 net3
root@solaris11-1:~#
```

Finally, we can remove the aggregation in order to prepare our environment for the next section:

```
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
super_aggr_0 ip     ok       yes    --
root@solaris11-1:~# ipadm delete-ip super_aggr_0
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
super_aggr_0        aggr      1500   up       net0 net1 net2 net3
root@solaris11-1:~# dladm delete-aggr super_aggr_0
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
net2                phys      1500   up       --
net3                phys      1500   up       --
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-1:~#
root@solaris11-1:~# ipadm create-ip net0
root@solaris11-1:~# ipadm create-ip net1
root@solaris11-1:~# ipadm create-ip net2
root@solaris11-1:~# ipadm create-ip net3
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm create-addr -T dhcp net0
net0/v4
root@solaris11-1:~# ipadm create-addr -T dhcp net1
net1/v4
root@solaris11-1:~# ipadm create-addr -T dhcp net2
net2/v4
root@solaris11-1:~# ipadm create-addr -T dhcp net3
net3/v4
root@solaris11-1:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.108/24
net1/v4           dhcp     ok           192.168.1.106/24
net2/v4           dhcp     ok           192.168.1.109/24
net3/v4           dhcp     ok           192.168.1.107/24
root@solaris11-1:~#
```

Excellent! We've completed our study of link aggregation.

### An overview of the recipe

In this section, we learned about both types of link aggregation. The main advantage is the performance because it puts all interfaces together, hides them, and presents only the final logical object: the link aggregation object. For external hosts, this works as there was only a single interface on the system. Furthermore, we saw how to monitor, modify, and delete aggregations.

# Configuring network bridging

Oracle Solaris 11 provides a wonderful feature that offers the possibility to deploy network bridges (layer 2, Datalink) that connect separated network segments and share the broadcast domain without the requirement of a router using a packet-forwarding mechanism: Network 1 | Bridge | Network 2.

The real effect of configuring and using Network Bridging is that all machines are able to communicate with each other as if they were on the same network. However, as a bridge works in a promiscuous mode, it uses some techniques in order to prevent creating loops such as **Spanning Tree Protocol** (**STP**), which is used with switches, and **Transparent Interconnect of Lots of Links** (**TRILL**), which has a small advantage when compared to STP because it always uses the short path to forward packages without shutting down a physical link as STP does.

## Getting ready

To follow this recipe, it's necessary to create a complex setup. We must have three virtual machines (VirtualBox or VMware, but I'm showing you the steps for VirtualBox) with Oracle Solaris 11 and 2 GB each. The first machine must have two network interfaces and the other two must have only one interface. For the first virtual machine (`solaris11-1`), network adapters must have the following configuration:

*   **Adapter 1** should have **Attached to** set to **Bridged Adapter**
*   **Adapter 2** should have **Attached to** set to **Internal Network**

The second machine (`solaris11-2`) must have the following network configuration:

*   **Adapter 1** should have **Attached to** set to **Internal Network**

The third virtual machine must have the following network configuration:

*   **Adapter 1** should have **Attached to** set to **Bridged Adapter**

First, in the VirtualBox environment, select the `solaris11-1` virtual machine, go to the **Machine** menu, and select **Settings**. When the configuration screen appears, go to **Network**, and in the **Adapter 1** tab, change the **Attached to** configuration to **Bridged Adapter**.

![Getting ready](img/00012.jpeg)

On the same screen, go to **Adapter 2** and configure the **Attached to** property to **Internal Network**, as shown in the following screenshot:

![Getting ready](img/00013.jpeg)

Now, on VirtualBox's first screen, select the `solaris11-2` virtual machine, go to the **Machine** menu, and select **Settings**. When the configuration screen appears, go to **Network**, and in the **Adapter 1** tab, change the **Attached to** configuration to **Internal Network**, as shown in the following screenshot:

![Getting ready](img/00014.jpeg)

Repeat the same steps that were performed for the previous machine for the third system and change the **Attached to** value to **Bridge Adapter**, as shown in the following screenshot:

![Getting ready](img/00015.jpeg)

## How to do it…

The scheme for this recipe is `solaris11-2` | `solaris11-1` | `solaris11-3`. Let's configure the bridge (`solaris11-1`). On the `solaris11-1` virtual machine, list the current network configuration:

```
root@solaris11-1:~# netadm list | grep ncp
ncp         Automatic      online
ncp         my_profile     disabled
ncp         DefaultFixed   disabled
root@solaris11-1:~# dladm show-phys
LINK           MEDIA              STATE      SPEED  DUPLEX    DEVICE
net0           Ethernet           up         1000   full      e1000g0
net1           Ethernet           up         1000   full      e1000g1
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net0                phys      1500   up       --
net1                phys      1500   up       --
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.40/24
lo0/v6            static   ok           ::1/128
```

So far, we know that this machine has two network interfaces; both are `up` and one of them has an IP address. Since this IP address comes from the last recipe, the following commands are used to erase this existing IP address and create a new one for the `net0` and `net1` network interfaces:

```
root@solaris11-1:~# ipadm delete-ip net0
root@solaris11-1:~# ipadm create-ip net0
root@solaris11-1:~# ipadm create-ip net1
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       down     no     --
net1       ip       down     no     --
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
```

Assign an IP address for each network interface (`net0` and `net1`) by executing the following commands:

```
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.65/24 net0/v4
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.10.38/24 net1/v4

```

To verify that the IP assignment is working, run the following command:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.65/24
net1/v4           static   ok           192.168.10.38/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
net1       ip       ok       yes    --
root@solaris11-1:~#
```

Great! We assigned one IP address (`192.168.1.65/24`) for the `net0/24` network interface and another one (`192.168.10.38/24`) for the `net1` network interface. As we can see, both are in different networks so they aren't able to communicate with each other.

In the `solaris11-3` virtual machine, let's also list the current network configuration, delete it, and create a new one:

```
root@solaris11-3:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.103/24
root@solaris11-3:~# ipadm delete-ip net0
root@solaris11-3:~# ipadm create-ip net0
root@solaris11-3:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       down     no     --
root@solaris11-3:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-3:~# dladm show-phys
LINK            MEDIA             STATE      SPEED  DUPLEX    DEVICE
net0            Ethernet          up         1000   full      e1000g0
root@solaris11-3:~# ipadm create-addr -T static -a 192.168.1.77/24 net0/v4
root@solaris11-3:~# ipadm show-addr | grep v4
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.77/24
root@solaris11-3:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
root@solaris11-3:~# ping 192.168.1.65
192.168.1.65 is alive
root@solaris11-3:~#
```

Good! This virtual machine can reach the first one (`solaris11-1`) because both are on the same network.

On the `solaris11-2` virtual machine, the same steps are going to be executed, erasing the current network configuration and creating a new one:

```
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.113/24
lo0/v6            static   ok           ::1/128
root@solaris11-2:~# dladm show-phys
LINK            MEDIA             STATE      SPEED  DUPLEX    DEVICE
net0            Ethernet          up         1000   full      e1000g0

root@solaris11-2:~# ipadm delete-ip net0
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-2:~# ipadm create-ip net0
root@solaris11-2:~# ipadm create-addr -T static -a 192.168.1.55 net0/v4
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.55/24
lo0/v6            static   ok           ::1/128
root@solaris11-2:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
root@solaris11-2:~# ping 192.168.1.65
ping: sendto No route to host
root@solaris11-2:~# ping 192.168.1.77
ping: sendto No route to host
```

This is really good. This virtual machine (`solaris11-2`) is on a different network (**Internal Network**) than the other two virtual machines and there's a router that isn't able to reach them. We expected this exact behavior!

Now it's time! Returning to the `solaris11-1` virtual machine, make a bridge (layer 2) between the `net0` and `net1` network interfaces in the following steps. First, verify that there is a bridge on the system by executing the following two commands:

```
root@solaris11-1:~# dladm show-bridge
root@solaris11-1:~# dladm show-phys
LINK            MEDIA             STATE      SPEED  DUPLEX    DEVICE
net0            Ethernet          up         1000   full      e1000g0
net1            Ethernet          up         1000   full      e1000g1
```

There is no bridge, so it's time to create the bridge (between the `net0` and `net1` network interfaces) by executing the following command:

```
root@solaris11-1:~# dladm create-bridge -l net0 -l net1 baybridge

```

To verify that the bridge was created successfully, execute the following command:

```
root@solaris11-1:~# dladm show-bridge
BRIDGE      PROTECT ADDRESS            PRIORITY DESROOT
baybridge   stp     32768/8:0:27:32:85:80 32768 32768/8:0:27:32:85:80
```

Gathering some details from `baybridge` is done by executing the following command:

```
root@solaris11-1:~# dladm show-bridge baybridge -l
LINK        STATE       UPTIME  DESROOT
net0        forwarding  38      32768/8:0:27:32:85:80
net1        forwarding  38      32768/8:0:27:32:85:80
```

That sounds good. Both network interfaces from the `solaris11-1` virtual machine are forwarding and using STP to prevent loops. The next command confirms that they are using STP:

```
root@solaris11-1:~# dladm show-bridge baybridge -t
dladm: bridge baybridge is not running TRILL
```

To verify that the bridge configuration has worked, the execution of the most important step from this recipe from the `solaris11-2` virtual machine is as follows:

```
root@solaris11-2:~# ping 192.168.1.65
192.168.1.65 is alive
root@solaris11-2:~# ping 192.168.1.77
192.168.1.77 is alive
root@solaris11-2:~#
```

Incredible! Previously, we tried to reach the `192.168.1.0` network and we didn't achieve success. However, now this is different because the bridge (`baybridge`) configured on `solaris11-1` has made it possible. Moreover, there's a big detail—there is no router. There's only a bridge.

To undo the bridge and return the environment to the initial configuration, execute the following command:

```
root@solaris11-1:~# dladm show-bridge
BRIDGE      PROTECT ADDRESS            PRIORITY DESROOT
baybridge   stp     32768/8:0:27:32:85:80 32768 32768/8:0:27:32:85:80
root@solaris11-1:~# dladm show-bridge -l baybridge
LINK        STATE       UPTIME  DESROOT
net0        forwarding  325     32768/8:0:27:32:85:80
net1        forwarding  1262    32768/8:0:27:32:85:80
root@solaris11-1:~# dladm remove-bridge -l net0 baybridge
root@solaris11-1:~# dladm remove-bridge -l net1 baybridge
root@solaris11-1:~# dladm delete-bridge baybridge
root@solaris11-1:~# dladm show-bridge
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.65/24
net1/v4           static   ok           192.168.10.38/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm delete-ip net0
root@solaris11-1:~# ipadm delete-ip net1
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
root@solaris11-1:~#
```

Logically, we've undone everything, and now it's necessary to change the network configuration back from the `solaris11-2` virtual machine to **Network Bridged**.

### An overview of the recipe

In this section, we learned how to configure, monitor, and unconfigure a bridge, which is a layer 2 technology that makes it possible to transmit a packet from one network to another without using a router.

# Configuring link protection and the DNS Client service

Nowadays, virtualized systems are growing and spreading very fast, and usually, the virtual machines or virtual environments (zones, for example) have full physical network access. Unfortunately, this granted network access can compromise the system and the entire network if malicious packets originate from these virtual environments. It is at this point that Oracle Solaris 11 Link Protection can prevent any damage from being caused by these harmful packets that come from virtual environments.

Oracle Solaris 11 has introduced Link Protection to try and prevent several types of spoof attacks, such as IP spoofing (when someone masquerades the IP address from his/her system with a forged IP address in order to pretend being another system, which is very usual during a denial-of-service attack), DHCP spoofing (when a rogue DHCP server is attached in the network in order to provide false information such as the gateway address, causing all network data flow to go through the cracker machine in a classic man-in-the-middle attack), and MAC spoofing (a lethal attack in which the MAC address is manipulated, making it possible for a cracker to execute a man-in-the-middle attack or even gain access to system or network devices that control access using the MAC address). All these attacks have the potential to compromise a network or even the whole company.

For appropriate protection against all these attacks, the Link Protection feature offers a network interface property named protection, which has some possible values that determine the security level. For example, in the case of protection against MAC spoofing (the `protection` property value is equal to `mac-nospoof`), any MAC address outbound packets (packets that leave the system) must be equal to the MAC address from the source network; otherwise, the packet will certainly be dropped.

When applying the IP spoofing protection (`ip-nospoof`), any outgoing packet (for example, ARP or IP) must have a source address equal to the address offered by the DHCP service or equal to the IP list configured in the `allow-ips` property. Otherwise, Oracle Solaris 11 drops the packet.

The other two possible values for the `protection` property are `dhcp-nonspoof` and `restricted` (which restricts the outgoing packets to only IPv4, IPv6, and ARP).

Another relevant subject is how to set up a DNS client on Oracle Solaris 11\. Until Oracle Solaris 10, this procedure wasn't integrated with the **Service Management Facility** (**SMF**) framework. This has changed with Oracle Solaris 11.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) with Oracle Solaris 11 installed, 4 GB RAM, one network interface, and access to the Internet. Optionally, if the environment has some Oracle Solaris Zones configured, the tests can be more realistic.

## How to do it…

Link protection must be configured in the global zone. If the protection is applied to the physical network interface, all vnics connected to the physical network interface will be protected, but the following steps will be performed for one vnic only.

The link protection configuration is started through a reset (disabling and resetting the protection to its default):

```
root@solaris11-1:~# dladm reset-linkprop -p protection net0
root@solaris11-1:~# dladm reset-linkprop -p protection net1

```

To list the link protection status, execute the following command:

```
root@solaris11-1:~# dladm show-linkprop -p protection,allowed-ips
LINK     PROPERTY            PERM VALUE        DEFAULT   POSSIBLE
net0     protection          rw   --           --        mac-nospoof,
                                                         restricted,
                                                         ip-nospoof,
                                                         dhcp-nospoof 
net0     allowed-ips         rw   --           --        -- 
vswitch1 protection          rw   --           --        mac-nospoof,
                                                         restricted,
                                                         ip-nospoof,
                                                         dhcp-nospoof 
vswitch1 allowed-ips         rw   --           --        -- 
vnic0    protection          rw   --           --        mac-nospoof,
                                                         restricted,
                                                         ip-nospoof,
                                                         dhcp-nospoof 
vnic0    allowed-ips         rw   --           --        -- 
vnic1    protection          rw   --           --        mac-nospoof,
                                                         restricted,
                                                         ip-nospoof,
                                                         dhcp-nospoof 
vnic1    allowed-ips         rw   --           --        -- 
vnic2    protection          rw   --           --        mac-nospoof,
                                                         restricted,
                                                         ip-nospoof,
                                                         dhcp-nospoof 
vnic2    allowed-ips         rw   --           --        -- 
```

The link protection is still not applied. Therefore, to enable link protection against IP spoofing for the network interface `net0`, execute the following:

```
root@solaris11-1:~# dladm set-linkprop -p protection=ip-nospoof net0
root@solaris11-1:~# dladm show-linkprop -p protection,allowed-ips
LINK     PROPERTY            PERM VALUE        DEFAULT   POSSIBLE
net0     protection          rw   ip-nospoof   --        mac-nospoof,
                                                         restricted,
                                                         ip-nospoof,
                                                         dhcp-nospoof 
net0     allowed-ips         rw   --           --        -- 
(truncated output)

```

Additionally, the two configured zones in the system have the IP addresses `192.168.1.55` and `192.168.1.66`, respectively, and both of them have virtual interfaces (`vnic0` and `vnic1`) connected to the `net0` interface. Then, to allow these zones to communicate over the physical network, execute the following command:

```
root@solaris11-1:~# dladm set-linkprop -p allowed-ips=192.168.1.55,192.168.1.66 net0

```

To verify and check the previous command, execute the following command:

```
root@solaris11-1:~# dladm show-linkprop -p protection,allowed-ips
LINK     PROPERTY           PERM VALUE         DEFAULT   POSSIBLE
net0     protection         rw   ip-nospoof    --        mac-nospoof,
                                                        restricted,
                                                        ip-nospoof,
                                                        dhcp-nospoof 
net0     allowed-ips        rw   192.168.1.55, --       -- 
                                 192.168.1.66              
(truncated output)

```

It's also possible to get some statistics about the link data protection for completeness, but we aren't going to delve into details here:

```
root@solaris11-1:~# dlstat -A | more
net0
  mac_rx_local
         ipackets                  0
           rbytes                  0
          rxlocal                  0
     rxlocalbytes                  0
            intrs                  0
        intrbytes                  0
            polls                  0
        pollbytes                  0
           idrops                  0
       idropbytes                  0
  mac_rx_other
         ipackets                  0
           rbytes                  0
(truncated)

```

To disable the link data protection, execute the following commands:

```
root@solaris11-1:~# dladm reset-linkprop -p protection net0
root@solaris11-1:~# dladm reset-linkprop -p protection net1

```

Approaching another subject, the DNS Client configuration has changed a lot since Oracle Solaris 10\. However, it isn't hard to configure it. It's only different.

Usually, this kind of task, which requires us to modify some configuration manually, is executed when working on an environment with the NCP profile `DefaultFixed` and loc profile `DefaultFixed` because when both profiles are set to `Automatic`, DHCP provides the name server configuration and other settings. Therefore, to make the next recipe more realistic, the NCP and loc profiles will be altered to `DefaultFixed` where every network configuration must be performed manually:

```
root@solaris11-1:~# dladm show-phys
LINK          MEDIA               STATE      SPEED  DUPLEX    DEVICE
net0          Ethernet            up         1000   full      e1000g0
root@solaris11-1:~# netadm list
TYPE        PROFILE        STATE
ncp         Automatic      online
ncu:phys    net0           online
ncu:ip      net0           online
ncp         my_profile     disabled
ncp         DefaultFixed   disabled
loc         NoNet          offline
loc         work           disabled
loc         Automatic      online
loc         DefaultFixed   offline
root@solaris11-1:~# netadm enable -p ncp DefaultFixed
Enabling ncp 'DefaultFixed'
root@solaris11-1:~# dladm show-phys
LINK            MEDIA             STATE      SPEED  DUPLEX    DEVICE
net0            Ethernet          unknown    1000   full      e1000g0
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
```

As we've enabled the `DefaultFixed` configuration, it's our task to create the IP object and assign an IP address to it:

```
root@solaris11-1:~# ipadm create-ip net0
root@solaris11-1:~# ipadm create-addr -T static -a 192.168.1.144/24 net0/v4

```

To confirm that the previous command is working, execute the following commands:

```
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
root@solaris11-1:~# netadm list
TYPE        PROFILE        STATE
ncp         Automatic      disabled
ncp         my_profile     disabled
ncp         DefaultFixed   online
loc         NoNet          offline
loc         work           offline
loc         Automatic      offline
loc         DefaultFixed   online
root@solaris11-1:~#
```

Great! Now, in order to change the DNS servers used by the system to look up hostnames and IP addresses, execute the following command:

```
root@solaris11-1:~# svccfg -s svc:/network/dns/client setprop config/nameserver=net_address:"(8.8.8.8 8.8.4.4)"

```

Setting the DNS domain (`example.com`) and domain search list (`example.com`) is done by running the following:

```
root@solaris11-1:~# svccfg -s svc:/network/dns/client setprop config/domain=astring:'("example.com")'
root@solaris11-1:~# svccfg -s svc:/network/dns/client setprop config/search=astring:'("example.com")'

```

Setting the IPv4 and IPv6 resolution order (first, try to resolve a hostname by looking at the `/etc/host` file, and if there is no success, try to resolve the hostname on the DNS service), respectively, is executed by the following commands:

```
root@solaris11-1:~# svccfg -s svc:/system/name-service/switch setprop config/host=astring:'("files dns")'
root@solaris11-1:~# svccfg -s svc:/system/name-service/switch setprop config/ipnodes=astring:'("files dns")'

```

Everything that was configured can be verified by executing the following commands:

```
root@solaris11-1:~# svccfg -s svc:/system/name-service/switch listprop config
config                     application
config/default             astring     files
config/value_authorization astring     solaris.smf.value.name-service.switch
config/printer             astring     "user files"
config/host                astring     "files dns"
config/ipnodes             astring     "files dns"
root@solaris11-1:~# svccfg -s svc:/network/dns/client listprop config
config                      application        
config/value_authorization astring     solaris.smf.value.name-service.dns.client
config/nameserver          net_address 8.8.8.8 8.8.4.4
config/domain              astring     example.com
config/search              astring     example.com
root@solaris11-2:~#
```

It's nice that the executed steps have worked; however, this isn't enough yet. All the DNS configuration up to this point isn't persistent and doesn't take effect now or till the next system boot. Therefore, the DNS Client service must be refreshed (to read its associated configuration file or service configuration again) for it to take effect immediately and restarted to make the configuration persistent (saved on the disk) and valid for the next system initializations. This task can be done by executing the following commands:

```
root@solaris11-1:~# svcadm refresh svc:/network/dns/client
root@solaris11-1:~# svcadm restart svc:/network/dns/client

```

Eventually, because of any prior random event, the `dns/client` service can be disabled, and in this case, we have to enable it by executing the following command:

```
root@solaris11-1:~# svcadm enable svc:/network/dns/client:default
root@solaris11-1:~# svcs dns/client
STATE          STIME    FMRI
online          5:34:07 svc:/network/dns/client:default
root@solaris11-1:~# svcs -l svc:/network/dns/client:default
fmri         svc:/network/dns/client:default
name         DNS resolver
enabled      true
state        online
next_state   none
state_time   January 12, 2014 05:34:07 AM BRST
logfile      /var/svc/log/network-dns-client:default.log
restarter    svc:/system/svc/restarter:default
manifest     /etc/svc/profile/generic.xml
manifest     /lib/svc/manifest/network/dns/client.xml
manifest     /lib/svc/manifest/milestone/config.xml
manifest     /lib/svc/manifest/network/network-location.xml
manifest     /lib/svc/manifest/system/name-service/upgrade.xml
dependency   optional_all/none svc:/milestone/config (online)
dependency   optional_all/none svc:/network/location:default (online)
dependency   require_all/none svc:/system/filesystem/root (online) svc:/system/filesystem/usr (online) svc:/system/filesystem/minimal (online)
dependency   require_any/error svc:/network/loopback (online)
dependency   optional_all/error svc:/milestone/network (online)
dependency   optional_all/none svc:/system/manifest-import (online)
dependency   require_all/none svc:/milestone/unconfig (online)
dependency   optional_all/none svc:/system/name-service/upgrade (online)
```

A very interesting point is that the `resolv.conf` file (the file that was the only point of configuration until Oracle Solaris 10) under `etc` is regenerated every time the DNS Client service is restarted. If the administrator modifies this file manually, the settings will take place immediately, but the file content will be restored from the service configuration in the next system reboot.

```
root@solaris11-1:~# more /etc/resolv.conf
#
# _AUTOGENERATED_FROM_SMF_V1_
#
# WARNING: THIS FILE GENERATED FROM SMF DATA.
#   DO NOT EDIT THIS FILE.  EDITS WILL BE LOST.
# See resolv.conf(4) for details.

domain  example.com
search  example.com
nameserver  8.8.8.8
nameserver  8.8.4.4
root@solaris11-2:~#
```

Finally, the name server resolution takes effect only if the following commands are executed:

```
root@solaris11-1:~# svcadm refresh svc:/system/name-service/switch:default
root@solaris11-1:~# svcadm restart svc:/system/name-service/switch:default

```

The same rule that is applied to the `resolv.conf` file under `etc` is also valid for the `nsswitch.conf` file (the file where the order of name resolution is configured) under `etc`, which is regenerated during each system boot as well:

```
root@solaris11-1:~# more /etc/nsswitch.conf
#
# _AUTOGENERATED_FROM_SMF_V1_
#
# WARNING: THIS FILE GENERATED FROM SMF DATA.
#   DO NOT EDIT THIS FILE.  EDITS WILL BE LOST.
# See nsswitch.conf(4) for details.

passwd:  files
group:  files
hosts:  files dns
ipnodes:  files dns
networks:  files
protocols:  files
rpc:  files
ethers:  files
netmasks:  files
bootparams:  files
publickey:  files
netgroup:  files
automount:  files
aliases:  files
services:  files
printers:  user files
project:  files
auth_attr:  files
prof_attr:  files
tnrhtp:  files
tnrhdb:  files
sudoers:  files
```

The final test is to ping a website as follows:

```
root@solaris11-1:~# ping www.oracle.com
www.oracle.com is alive
```

To configure the default gateway for the system (`192.168.1.1`) and prevent the same effect of persistence (settings that are only valid until the next reboot) such as that in the DNS client configuration case, execute the following command:

```
root@solaris11-1:~# route -p add default 192.168.1.1

```

To verify the previous command and confirm the gateway configuration, execute the following command:

```
root@solaris11-1:~# netstat -rn -f inet
Routing Table: IPv4
Destination         Gateway          Flags  Ref     Use     Interface
------------------- ---------------- ------ ------- ------- ---------
default             192.168.1.1      UG        2         46   
127.0.0.1           127.0.0.1        UH        2        500 lo0
192.168.1.0         192.168.1.144    U         3          4 net0
```

### An overview of the recipe

In this section, we learned about Link Protection to protect against DNS, DHCP, and IP spoofing. Additionally, the DNS Client service configuration was presented too.

# Configuring the DHCP server

Oracle Solaris 11 includes an open source version of DHCP named **Internet Systems Consortium Dynamic Host Configuration Protocol** (**ISC DHCP**), which is a well-known client-server service used by most IT administrators. This makes network and IP address configuration easier, mainly when there are many machines to be managed on a network. Without a DHCP server, the administrator would have to configure the IP address, mask, gateway, server name, and other settings on each network machine manually, making administration a time-consuming job. When using the DHCP service, most network settings are performed in a centralized point and there is the possibility of performing a particular configuration for chosen machines.

The DHCP server isn't already installed on Oracle Solaris 11, and it's available on the DVD or in the Oracle repository, whereas the DHCP client (`dhcpagent`) runs and is included on every default Oracle Solaris 11 and higher installations.

All DHCP operations are based on the broadcast service and are restricted to a local network, and each network segment should have its own DHCP server. When there are hosts on a network segment (for example, segment A) and there's only one DHCP server on another network segment (for example, segment B), it's possible to use the DHCP server from segment B through a router using a DHCP relay implementation. Oracle Solaris 11 offers the support to configure a DHCP relay as well. However, this won't be shown because using a DHCP relay with Oracle Solaris 11 is a rare configuration.

## Getting ready

This recipe requires three virtual machines (VirtualBox or VMware) running Oracle Solaris 11 with 4 GB RAM. It is recommended that all machines be on an isolated network to prevent any external DHCP server from disturbing our test.

## How to do it…

As we've mentioned, the DHCP server isn't installed by default; we have to install it on the first machine (`solaris11-1`):

```
root@solaris11-1:~# pkg publisher
PUBLISHER           TYPE     STATUS P LOCATION
solaris             origin   online F http://solaris11-1.example.com/
root@solaris11-1:~# pkg install dhcp/isc-dhcp
           Packages to install:  1
       Create boot environment: No
Create backup boot environment: No
            Services to change:  2
(truncated output)

```

As the appropriate packages have been installed, it's time to configure the DHCP server.

Our subnet is `192.168.1.0/24`, so the DHCP server needs to be configured to attend this network segment. Copy the `dhcpd.conf.example` template under `etc/inet` to `/etc/inet/dhcpd4.conf` and make some changes including the network segment, default lease time, domain server names, and default gateway line configuration, as follows:

```
root@solaris11-1:~# cp /etc/inet/dhcpd.conf.example /etc/inet/dhcpd4.conf
root@solaris11-1:~# more /etc/inet/dhcpd4.conf
option domain-name "example.com";
option domain-name-servers 8.8.8.8, 8.8.4.4;

default-lease-time 600;
max-lease-time 7200;

# This is a very basic subnet declaration.

subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.10 192.168.1.15 ;
  option routers 192.168.1.1 ;
}
root@solaris11-1:~#
```

To make the changes in `dhcp4.conf` under `/etc/inet/` take effect, execute the following commands:

```
root@solaris11-1:~# svcs -a | grep dhcp
disabled        7:23:22 svc:/network/dhcp/server:ipv6
disabled        7:23:22 svc:/network/dhcp/server:ipv4
disabled        7:23:24 svc:/network/dhcp/relay:ipv6
disabled        7:23:24 svc:/network/dhcp/relay:ipv4
root@solaris11-1:~# svcadm enable svc:/network/dhcp/server:ipv4
root@solaris11-1:~# svcs -a | grep dhcp
disabled        7:23:22 svc:/network/dhcp/server:ipv6
disabled        7:23:24 svc:/network/dhcp/relay:ipv6
disabled        7:23:24 svc:/network/dhcp/relay:ipv4
online          7:58:21 svc:/network/dhcp/server:ipv4
root@solaris11-1:~#
```

We've performed the configuration on the DHCP server; now, move to configure the DHCP client on the `solaris11-2` system. In order to set up the network interface to get the network configuration from our DHCP server, execute the following command:

```
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.55/24
lo0/v6            static   ok           ::1/128
root@solaris11-2:~# ipadm delete-ip net0
root@solaris11-2:~# ipadm create-ip net0
root@solaris11-2:~# ipadm create-addr -T dhcp net0/v4
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.10/24
lo0/v6            static   ok           ::1/128
```

Perfect! The client machine (`solaris11-2`) has received an IP address, which is in the range offered by the DHCP server (`192.168.1.10` to `192.168.1.15`). The most important command is `ipadm create-addr -T dhcp net0/v4`, which assigns an IP address from the DHCP server.

On the DHCP server machine, there's a file named `dhcp4.leases` that shows us the DHCP client lease information:

```
root@solaris11-1:~# more /var/db/isc-dhcp/dhcpd4.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.1-ESV-R6

lease 192.168.1.10 {
  starts 6 2014/01/18 20:16:07;
  ends 6 2014/01/18 22:16:07;
  cltt 6 2014/01/18 20:16:07;
  binding state active;
  next binding state free;
  hardware ethernet 08:00:27:96:46:f0;
}
server-duid "\000\001\000\001\032k\273=\010\000'2\205\200";
```

According to the preceding command, it was allocated an IP address (`192.168.1.10`) for the client that holds the MAC address `08:00:27:96:46:f0`. Retuning to the `solaris11-2` machine (the DHCP client machine), it's possible to confirm that we are talking about the same virtual machine:

```
root@solaris11-2:~# dladm show-linkprop net0 | grep mac-address
net0     mac-address         rw   8:0:27:96:46:f0 8:0:27:96:46:f0 –
```

At the DHCP client, execute the following command to renew the IP address:

```
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.10/24
lo0/v6            static   ok           ::1/128
root@solaris11-2:~# ipadm refresh-addr net0/v4

```

In the `solaris11-1` server, the renewing event is shown in `/var/db/isc-dhcp/dhcp4.leases`:

```
root@solaris11-1:~# more /var/db/isc-dhcp/dhcpd4.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.1-ESV-R6

lease 192.168.1.10 {
  starts 6 2014/01/18 20:16:07;
  ends 6 2014/01/18 22:16:07;
  cltt 6 2014/01/18 20:16:07;
  binding state active;
  next binding state free;
  hardware ethernet 08:00:27:96:46:f0;
}
server-duid "\000\001\000\001\032k\273=\010\000'2\205\200";

lease 192.168.1.10 {
  starts 6 2014/01/18 20:19:02;
  ends 6 2014/01/18 22:19:02;
  cltt 6 2014/01/18 20:19:02;
  binding state active;
  next binding state free;
  hardware ethernet 08:00:27:96:46:f0;
}
```

Let's test the renew process once more, releasing and leasing a new IP address by executing the following commands:

```
root@solaris11-2:~# ipadm delete-addr -r net0/v4
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
lo0/v6            static   ok           ::1/128
root@solaris11-2:~# ipadm create-addr -T dhcp net0
net0/v4
root@solaris11-2:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           dhcp     ok           192.168.1.10/24
lo0/v6            static   ok           ::1/128
root@solaris11-2:~#
```

Everything is working fine!

### An overview of the recipe

The DHCP server is a very common service and is easy to configure and maintain. This DHCP example will be used as a support service for the **Automated Installation** (**AI**) service in a later chapter.

# Configuring Integrated Load Balancer

Certainly, **Integrated Load Balancer** (**ILB**) is one of the most attractive features of Oracle Solaris 11 because it provides network layer 3 and 4 with the load balance service. Basically, when a client requires a resource from an application (for example, a web server), the ILB framework decides which backend host (for example, web server A or B) will attend the request. Therefore, the main role of ILB is to decide to which backend server (for example, the Apache web server) the request will be forwarded. ILB supports two work methods in Oracle Solaris 11: **Direct Server Return** (**DSR**) and **Network** **Address Translate** (**NAT**). In both cases, the ILB framework uses one of four algorithms:

*   **Round robin**: This tries to keep an equal statistic distribution over all backend servers
*   **Source IP hash**: In this, the choice of the destination backend server is made by hashing the source IP address of the client
*   **Source IP port hash**: In this, the choice of the destination backend server is made by hashing the source IP and port address of the client
*   **Source IP VIP hash**: In this, the choice of the destination backend server is made by hashing the source and destination IP address of the client

The DSR method allows ILB to receive the request in order to decide which backend server (for example, Apache servers) the request will be forwarded to and to make the answer from the backend server return directly to the client. Nevertheless, if the ILB server is configured as a router, then all answers from backend servers can be routed to the client through ILB.

When ILB is configured to use the DSR method, its performance is better than NAT and it also shows better transparency because only the destination MAC address is changed and the answer returning to the client can bypass the ILB server, as we've mentioned previously. Unfortunately, if we try to add a new backend server, the connection will be disrupted because the connection is stateless.

A scheme about what we've mentioned up to now can be viewed as follows:

*   **request**: client | ILB server | backend servers (A or B)
*   **answer**: backend server | client
*   **answer (ILB as router)**: backend server | ILB (router) | client

The following image (the IP addresses in the image are only an example) also describes the process:

![Configuring Integrated Load Balancer](img/00016.jpeg)

The NAT method (half or full) allows ILB to rewrite all requests by changing the destination IP address and—when ILB is working in the NAT full method—by also changing the source address by masking the real IP client with the ILB IP address. Backend servers think that the request is coming from the ILB server instead of coming from the client.

The following is a scheme that explains this process:

*   **request**: client | ILB server (NAT) | backend server (A or B)
*   **answer**: backend server (A or B) | ILB server (NAT) | client

To make this easier, the following diagram explains the process:

![Configuring Integrated Load Balancer](img/00017.jpeg)

Unlike DSR, the ILB NAT model requires the ILB server as a default gateway.

## Getting ready

To follow the recipe, we must have four virtual machines (VirtualBox or VMware) installed with Oracle Solaris 11 and 4 GB RAM.

Personally, I've installed all of these virtual machines in VirtualBox and their network adapters were configured as **Attached in: Internal Network**. The scenario was designed as `solaris11-2` | `solaris11-1` | `solaris11-3`/`solaris11-4`:

*   `solaris11-2` (`net0`): `192.168.1.155`
*   `solaris11-1` (`net0`): `192.168.1.144`
*   `solaris11-1` (`net1`): `192.168.5.77`
*   `solaris11-3` (`net0`): `192.168.5.88`
*   `solaris11-4` (`net0`): `192.168.5.99`

For example, `/etc/hosts` should be as follows:

```
root@solaris11-1:~# more /etc/hosts | grep -v '#'
::1 solaris11-1 localhost
127.0.0.1 solaris11-1 localhost loghost
192.168.1.144  solaris11-1  solaris11-1.example.com
192.168.1.155  solaris11-2  solaris11-2.example.com
192.168.5.77  solaris11-1b  solaris11-1b.example.com
192.168.5.88  solaris11-3  solaris11-3.example.com
192.168.5.99  solaris11-4  solaris11-4.example.com
```

## How to do it…

Before starting a NAT or DSR example, the infrastructure must be configured and all virtual machines must be set up according to the IP address configuration shown previously:

In `solaris11-1`, execute the following commands:

```
root@solaris11-1:~# ipadm delete-ip net0
root@solaris11-1:~# ipadm delete-ip net1
root@solaris11-1:~# ipadm create-ip net0
root@solaris11-1:~# ipadm create-ip net0
root@solaris11-1:~# ipadm create-addr –T static –a 192.168.1.144/24 net0/v4
root@solaris11-1:~# ipadm create-addr –T static –a 192.168.5.77/24  net1/v4
root@solaris11-1:~# ipadm show-addr | grep v4
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
net1/v4           static   ok           192.168.5.77/24
root@solaris11-1:~#
```

In `solaris11-2`, execute the following commands:

```
root@solaris11-2:~# ipadm delete-ip net0
root@solaris11-2:~# ipadm create-ip net0
root@solaris11-2:~# ipadm create-addr –T static –a 192.168.1.155/24 net0/v4
root@solaris11-2:~# ipadm show-addr | grep v4
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.155/24
root@solaris11-2:~#
```

In `solaris11-3`, execute the following commands:

```
root@solaris11-3:~# ipadm delete-ip net0
root@solaris11-3:~# ipadm create-ip net0
root@solaris11-3:~# ipadm create-addr –T static –a 192.168.5.88/24 net0/v4
root@solaris11-3:~# ipadm show-addr | grep v4
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.5.88/24
root@solaris11-3:~#
```

In `solaris11-4`, execute the following commands:

```
root@solaris11-4:~# ipadm delete-ip net0
root@solaris11-4:~# ipadm create-ip net0
root@solaris11-4:~# ipadm create-addr –T static –a 192.168.5.99/24 net0/v4
root@solaris11-4:~# ipadm show-addr | grep v4
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.5.99/24
root@solaris11-4:~#
```

The next stage is to configure both Apache servers (`solaris11-3` and `solaris11-4`) by executing the following commands:

```
root@solaris11-3:~# pkg install apache
root@solaris11-3:~# cd /var/apache2/2.2/htdocs
root@solaris11-3:/var/apache2/2.2/htdocs# cp index.html index.html.backup
root@solaris11-3:/var/apache2/2.2/htdocs# vi index.html
root@solaris11-3:/var/apache2/2.2/htdocs# more index.html

```

![How to do it…](img/00018.jpeg)

```
root@solaris11-3:/var/apache2/2.2/htdocs# svcs -a | grep apache22
disabled        1:21:53 svc:/network/http:apache22
root@solaris11-3:/var/apache2/2.2/htdocs# svcadm enable svc:/network/http:apache22
root@solaris11-3:~# svcs -a | grep apache22
online          4:31:59 svc:/network/http:apache22
root@solaris11-4:~# cd /var/apache2/2.2/htdocs
root@solaris11-4:/var/apache2/2.2/htdocs# cp index.html index.html.backup
root@solaris11-4:/var/apache2/2.2/htdocs# vi index.html
root@solaris11-4:/var/apache2/2.2/htdocs# more index.html

```

![How to do it…](img/00019.jpeg)

```
root@solaris11-4:/var/apache2/2.2/htdocs# svcs -a | grep apache22
disabled        1:21:53 svc:/network/http:apache22
root@solaris11-4:/var/apache2/2.2/htdocs# svcadm enable svc:/network/http:apache22
root@solaris11-4:/var/apache2/2.2/htdocs# svcs -a | grep apache22
online          4:43:58 svc:/network/http:apache22
```

The required infrastructure is ready, and so the ILB setup is going to be executed in the `solaris11-1` virtual machine that is configuring a half-NAT scenario:

```
root@solaris11-1:~# ping solaris11-2
solaris11-2 is alive
root@solaris11-1:~# ping solaris11-3
solaris11-3 is alive
root@solaris11-1:~# ping solaris11-4
solaris11-4 is alive
```

To verify the routing and forwarding configuration, run the following command:

```
root@solaris11-1:~# routeadm
             Configuration   Current              Current
                    Option   Configuration        System State
---------------------------------------------------------------
              IPv4 routing   disabled             disabled
              IPv6 routing   disabled             disabled
           IPv4 forwarding   disabled             disabled
           IPv6 forwarding   disabled             disabled

          Routing services   "route:default ripng:default"

Routing daemons:

                     STATE   FMRI
                  disabled   svc:/network/routing/rdisc:default
                  disabled   svc:/network/routing/route:default
                  disabled   svc:/network/routing/ripng:default
                    online   svc:/network/routing/ndp:default
                  disabled   svc:/network/routing/legacy-routing:ipv4
                  disabled   svc:/network/routing/legacy-routing:ipv6
```

To enable the IPv4 forwarding between network interface cards in the system, execute the following commands:

```
root@solaris11-1:~# routeadm -e ipv4-forwarding
root@solaris11-1:~# ipadm set-prop -p forwarding=on ipv4
root@solaris11-1:~# routeadm
             Configuration   Current              Current
                    Option   Configuration        System State
---------------------------------------------------------------
              IPv4 routing   disabled             disabled
              IPv6 routing   disabled             disabled
            IPv4 forwarding   enabled              enabled
           IPv6 forwarding   disabled             disabled

          Routing services   "route:default ripng:default"

Routing daemons:

                     STATE   FMRI
                  disabled   svc:/network/routing/rdisc:default
                  disabled   svc:/network/routing/route:default
                  disabled   svc:/network/routing/ripng:default
                    online   svc:/network/routing/ndp:default
                  disabled   svc:/network/routing/legacy-routing:ipv4
                  disabled   svc:/network/routing/legacy-routing:ipv6
root@solaris11-1:~#
root@solaris11-1:~# svcs -a | grep ilb
disabled        5:03:26 svc:/network/loadbalancer/ilb:default
```

At this time, we have to enable the ILB service by executing the following commands:

```
root@solaris11-1:~# svcadm enable svc:/network/loadbalancer/ilb:default
root@solaris11-1:~# svcs -a | grep ilb
online          5:08:42 svc:/network/loadbalancer/ilb:default
```

When working with ILB, we must create a server group that points to the application running in the backend servers (in our case, Apache):

```
root@solaris11-1:~# ilbadm create-servergroup -s servers=solaris11-3:80,solaris11-4:80 apachegroup
root@solaris11-1:~# ilbadm show-servergroup
SGNAME         SERVERID            MINPORT MAXPORT IP_ADDRESS
apachegroup    _apachegroup.0      80      80      192.168.5.88
apachegroup    _apachegroup.1      80      80      192.168.5.99
```

The next step creates a **virtual IP address** (**VIP address**), which makes the load balance possible and application to be accessed by the client through any network interface:

```
root@solaris11-1:~# ipadm create-addr -d -a 192.168.1.220/24 net0
net0/v4a
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
net0/v4a          static   down         192.168.1.220/24
net1/v4           static   ok           192.168.5.77/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# ipadm up-addr net0/v4a
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
net0/v4a          static   ok           192.168.1.220/24
net1/v4           static   ok           192.168.5.77/24
lo0/v6            static   ok           ::1/128
```

Finally, we're going to configure ILB using the round-robin algorithm by running the following command:

```
root@solaris11-1:~# ilbadm create-rule -ep -i vip=192.168.1.220,port=8080 -m lbalg=roundrobin,type=HALF-NAT,pmask=24 -o servergroup=apachegroup rule_one

```

Some options of this command are as follows:

*   `-e`: This enables a rule
*   `-p`: This makes the rule persistent across a reboot
*   `-i`: This specifies an incoming package
*   `vip`: This is the virtual IP address (the connection point)
*   `port`: This is the virtual IP address port
*   `-m`: This specifies the keys that describe how to handle a packet
*   `lbalg`: This is the load-balance algorithm
*   `type`: This is the ILB topology

This recipe doesn't use dynamic routing; hence, it's necessary to include a static route in each backend server manually in order to return all answers to the ILB server:

```
root@solaris11-3:~# route add net 192.168.1.0/24 192.168.5.77
add net 192.168.1.0/24: gateway 192.168.5.77
root@solaris11-3:~# ping 192.168.1.144
192.168.1.144 is alive
root@solaris11-3:~#
root@solaris11-4:~# route add net 192.168.1.0/24 192.168.5.77
add net 192.168.1.0/24: gateway 192.168.5.77
root@solaris11-4:~# ping 192.168.1.144
192.168.1.144 is alive
root@solaris11-4:~#
```

The test of the ILB setup is performed through a browser pointing to the ILB server (`http://192.168.1.220:8080`), and it confirms that the result of the recipe is the following screenshot:

![How to do it…](img/00020.jpeg)

After a short time (60 seconds), we try to access the same address again:

![How to do it…](img/00021.jpeg)

Wonderful! The ILB recipe works perfectly!

There are other educational details here. For example, it's possible to gather the rules' details in the command line by executing the following command:

```
root@solaris11-1:~# ilbadm show-rule
RULENAME         STATUS LBALG       TYPE    PROTOCOL VIP        PORT
rule_one         E      roundrobin  HALF-NAT TCP 192.168.1.220   8080
root@solaris11-1:~# ilbadm show-rule -f
       RULENAME: rule_one
         STATUS: E
           PORT: 8080
       PROTOCOL: TCP
          LBALG: roundrobin
           TYPE: HALF-NAT
      PROXY-SRC: --
          PMASK: /24
        HC-NAME: --
        HC-PORT: --
     CONN-DRAIN: 0
    NAT-TIMEOUT: 120
PERSIST-TIMEOUT: 60
    SERVERGROUP: apachegroup
            VIP: 192.168.1.220
        SERVERS: _apachegroup.0,_apachegroup.1
```

The statistics (sampled every two seconds) can be presented by executing the following command:

```
root@solaris11-1:~# ilbadm show-statistics 2
PKT_P   BYTES_P   PKT_U   BYTES_U   PKT_D   BYTES_D
189     33813     0       0         0       0
0       0         0       0         0       0
0       0         0       0         0       0
0       0         0       0         0       0
0       0         0       0         0       0
^C
root@solaris11-1:~#
```

Here, note the following:

*   `PKT_P`: These are processed packets
*   `BYTES_P`: These are processed bytes
*   `PKT_U`: These are unprocessed packets
*   `BYTES_U`: These are unprocessed bytes
*   `PKT_D`: These are dropped packets
*   `BYTES_D`: These are dropped bytes

Great! Although the ILB configuration is complete, we can add or remove new backend servers anytime without having to stop ILB or disrupt any connection using the `ilbadm add-server` and `ilbadm remove-server` commands. This feature is possible only when configuring NAT ILB. Moreover, another alternative is to stick the connection from the same client to the same server (session persistence) using the `–p` option and by specifying the `pmask` suboption.

The half-NAT ILB setup provides you with the capacity to prevent new connections from being completed on a disabled server when there's a plan to execute the maintenance of this disabled server. A very good detail is that we deployed a single port (`8080`) to receive a new connection to the VIP address. Nevertheless, it would be possible to use several ports (`8080`-`8089`, for example) in order to balance connections among them using TCP or UDP.

There are other alternatives that are worth mentioning:

*   `conn-drain`: This is used in the NAT ILB scenario; it's a kind of timeout. After this time, the server's connection state is removed as well as the respective rule. The default behavior for TCP is that connections remain until they are terminated, whereas the UDP connection is kept until the idle timeout time.
*   `nat-timeout`: This value establishes the upper time limit for a connection (60 seconds for UDP and 120 seconds for TCP) to be killed and removed.
*   `persist-timeout`: This is only used when persistent mapping is enabled, and it works like a time limit (the default is 60 seconds) in order to remove the mapping. At the end, the persistent mapping will be lost after the time limit.

To show how these options can be used, disable and remove the existing rule and afterwards, create a new rule with some additional parameters:

```
root@solaris11-1:~# ilbadm disable-rule rule_one
root@solaris11-1:~# ilbadm delete-rule rule_one
root@solaris11-1:~# ilbadm show-rule
root@solaris11-1:~# ilbadm create-rule -ep -i vip=192.168.1.220,port=8080-8099,protocol=tcp -m lbalg=roundrobin,type=HALF-NAT,pmask=24 -t conn-drain=30,nat-timeout=30,persist-timeout=30 -o servergroup=apachegroup rule_two
root@solaris11-1:~# ilbadm show-rule
RULENAME    STATUS LBALG       TYPE    PROTOCOL VIP         PORT
rule_two    E      roundrobin  HALF-NAT TCP 192.168.1.220   8080-8099
root@solaris11-1:~# ilbadm show-rule -f
       RULENAME: rule_two
         STATUS: E
           PORT: 8080-8099
       PROTOCOL: TCP
          LBALG: roundrobin
           TYPE: HALF-NAT
      PROXY-SRC: --
          PMASK: /24
        HC-NAME: --
        HC-PORT: --
     CONN-DRAIN: 30
    NAT-TIMEOUT: 30
PERSIST-TIMEOUT: 30
    SERVERGROUP: apachegroup
            VIP: 192.168.1.220
        SERVERS: _apachegroup.0,_apachegroup.1
```

This example uses a port range (`8080` to `8099`) by permitting any client using TCP to connect to any port in this range and specific parameters that control the timeout values explained previously. Any setup should be performed according to the applications that run in the backend servers.

Erasing all ILB configuration can be done by executing the following commands:

```
root@solaris11-1:~# ilbadm disable-rule rule_two
root@solaris11-1:~# ilbadm show-rule -f
       RULENAME: rule_two
         STATUS: D
           PORT: 8080-8099
       PROTOCOL: TCP
          LBALG: roundrobin
           TYPE: HALF-NAT
      PROXY-SRC: --
          PMASK: /24
        HC-NAME: --
        HC-PORT: --
     CONN-DRAIN: 30
    NAT-TIMEOUT: 30
PERSIST-TIMEOUT: 30
    SERVERGROUP: apachegroup
            VIP: 192.168.1.220
        SERVERS: _apachegroup.0,_apachegroup.1
root@solaris11-1:~# ilbadm delete-rule rule_two
root@solaris11-1:~# ilbadm show-servergroup
SGNAME         SERVERID            MINPORT MAXPORT IP_ADDRESS
apachegroup    _apachegroup.0      80      80      192.168.5.88
apachegroup    _apachegroup.1      80      80      192.168.5.99
root@solaris11-1:~# ilbadm delete-servergroup apachegroup
root@solaris11-1:~# ilbadm show-servergroup
root@solaris11-1:~#
```

### An overview of the recipe

ILB is a fantastic feature of Oracle Solaris 11 that creates the load balance for layer 3 and 4 and helps distribute the client requests over backend servers.

# References

*   *Managing Oracle Solaris 11.1 Network Performance* at [http://docs.oracle.com/cd/E26502_01/html/E28993/preface-1.html#scrolltoc](http://docs.oracle.com/cd/E26502_01/html/E28993/preface-1.html#scrolltoc)
*   *Oracle Solaris Administration: Network Interfaces and Network Virtualization* at [http://docs.oracle.com/cd/E23824_01/html/821-1458/docinfo.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1458/docinfo.html#scrolltoc)
*   *Working With DHCP in* *Oracle Solaris 11.1* at [http://docs.oracle.com/cd/E26502_01/html/E28991/dhcp-overview-1.html#scrolltoc](http://docs.oracle.com/cd/E26502_01/html/E28991/dhcp-overview-1.html#scrolltoc)
*   *Oracle Solaris Administration: IP Services* at [http://docs.oracle.com/cd/E23824_01/html/821-1453/toc.html](http://docs.oracle.com/cd/E23824_01/html/821-1453/toc.html)
*   *Integrated Load Balancer Overview* at [http://docs.oracle.com/cd/E23824_01/html/821-1453/gijjm.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1453/gijjm.html#scrolltoc)
*   *System Administration Commands* at [http://docs.oracle.com/cd/E26502_01/html/E29031/ilbadm-1m.html#REFMAN1Milbadm-1m](http://docs.oracle.com/cd/E26502_01/html/E29031/ilbadm-1m.html#REFMAN1Milbadm-1m)
*   *Configuration of Integrated Load Balancer* at [http://docs.oracle.com/cd/E23824_01/html/821-1453/gijgr.html#scrolltoc](http://docs.oracle.com/cd/E23824_01/html/821-1453/gijgr.html#scrolltoc)