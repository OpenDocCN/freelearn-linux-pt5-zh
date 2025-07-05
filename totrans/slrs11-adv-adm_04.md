# Chapter 4. Zones

In this chapter, we will cover the following recipes:

*   Creating, administering, and using a virtual network in a zone
*   Managing a zone using the resource manager
*   Implementing a flow control
*   Working with migrations from physical Oracle Solaris 10 hosts to Oracle Solaris 11 Zones

# Introduction

Oracle Solaris 11 Zones is a great framework that virtualizes and consolidates a system environment where there are many applications and physical machines running Oracle Solaris. Using a rough comparison, Oracle Solaris 11 zone is similar to other virtualization options offered by VMware ESX, Linux LXC, and FreeBSD Jails but presents some important differences such as not allowing either to perform a hardware emulation or run any other kind of operating system except Oracle Solaris 11 or prior Oracle Solaris versions.

In Oracle Solaris Zones, the fundamental idea is to create different small operating system installations (children) inside the main operating system (parent) by sharing or dividing (using the resource manager) the existing resources between these children installations. Each installation will have its own init files and processes, although it shares the kernel with the parent operating system, resulting in a lesser overhead than previously quoted solutions. Using the Oracle Solaris 11 terms, the parent is the global zone and children are non-global zones, as we'll see later.

Oracle Solaris zone offers application isolation, additional tiers of security, and reduced power requirements. This concern with security is necessary in order to prevent an application running inside a zone from crashing other applications in other zones. This is the reason why a non-global zone does not view other non-global zones, can contain additional software packages, and has a different product database that controls its own installed software.

Going into details of the previously mentioned features, zones make it possible for many applications to share host resources, therefore decreasing the cost of a deployment. This resource management allows us to assign specific resources to a non-global zone in order to create a limit of resource consumption (for example, CPU and memory) and to control how many resources will be used by a process, task, or project. Moreover, this resource control takes advantage of an available Oracle Solaris scheduler class **fair share scheduler** (**FSS**) in order to impose control over the CPU (using shares) and memory (using the `rcapd` daemon that limits the amount of physical memory) in a non-global zone.

Zone was introduced in Oracle Solaris Version 10, and it can be classified as the global zone (the physical machine installation that was presented as a parent previously) and non-global zone (informally named as *local zone* or just *zone*, which was presented as a child) where any application can be installed and administered and the right resource configuration can be performed.

The global zone (the parent zone) is a bootable zone that comes directly from the physical hardware, and it makes it possible to configure, install, administer, and remove non-global zones (children zones), given that it is also the only zone that is aware of all of the existing zones. Usually, non-global zones run the same operating system as the global zone, but Oracle Solaris 11 provides another zone type, named **branded zone**, which makes it feasible to create and install a non-global zone that runs Oracle Solaris 10, for example.

Briefly, during a non-global zone installation, it's requested to provide as input the directory where the zone will be installed, the network interface, and network information such as IP address and network mask. Additionally, it is also requested to provide the IP-type to be used with the network interface in the non-global zone. There are two options: shared-IP (used when the network interface is shared with the global zone) and exclusive-IP (used when the network interface is dedicated to the non-global zone).

Once the zone configuration is complete, the next step is to install the zone and administer it. It is advisable to know that non-global zones can have the following zone states:

*   **undefined**: This denotes whether the zone configuration is incomplete or deleted
*   **incomplete**: This denotes that the zone installation was aborted in between
*   **configured**: This denotes whether the zone configuration is complete
*   **installed**: This denotes that the zone packages and operating system were installed
*   **ready**: This denotes the almost-running zone with an associated zone ID
*   **running**: This denotes that everything is working and getting executed
*   **down**: This denotes that the zone is halted

Honestly, on a daily basis, the more typical states are `configured`, `installed`, `running`, and `down`. The remaining states are transient states and we rarely have to be concerned about them.

Therefore, the sequence of states is `undefined` | `configured` | `incomplete` | `installed` | `ready` | `running` | `down`.

There are professionals who usually ask me, "What are the differences between Oracle Solaris 11 and Oracle Solaris 10?" Truly, there are some relevant differences. Now, the `var` directory is a separated filesystem, the default zone brand is Solaris (previously, it was native), there is no concept of sparse zones anymore, and the default filesystem is ZFS and uses IPS as package manager. However, the most important zone difference in Oracle Solaris 11 is the introduction of network virtualization, which allows us to control the network zone resources using at least a network interface—**virtual network interfaces** (**VNICs**)—and virtual switch concepts. For example, a physical machine could have Oracle Solaris 11 running in a global zone and five non-global zones (z1, z2, z3, z4, and z5), each of them with a dedicated VNIC connected to a virtual switch (`etherstub`) with the last one connected to the real network interface card. Additionally, the network flow control can be enforced and specific link properties can be configured to increase the bandwidth control and efficiency as well, which makes it possible to share a network resource across different VNICs.

The possible network flow can be created on a per-VNIC basis with specific attributes, isolating and classifying similar packets and with associated bound resources. Possible flow attributes include `maxbw` (which defines the bandwidth of the flow) and priority (which defines the packet priority in a flow as low, medium, and high).

All resource controls mentioned so far (CPU, memory, and network) are disabled by default, and they are controlled by two resource services: the default resource pool service (`svc:/system/pools:default`) and dynamic resource pool service (`svc:/system/pools/dynamic:default`). A configuration file named `pooladm.conf` under `etc` helps us define the pool creation and the resource management behavior, as it is used by a daemon named `poold` that controls the entire allocation controls and limits after associating the created pool with a non-global zone.

Now, we are ready to learn about the next recipes on Oracle Solaris 11 Zones.

# Creating, administering, and using a virtual network in a zone

I love this recipe because here, we are going to use the main feature of zones in Oracle Solaris 11 virtual networks. Concisely, we are going to create and configure the following scenario:

*   `zone1` | `vnic1` (`192.168.1.51`) | `vswitch1` (`etherstub`) | `net0` (`192.168.1.144`)
*   `zone2` | `vnic2` (`192.168.1.52`) | `vswitch1` (`etherstub`) | `net0` (`192.168.1.144`)

Each zone connects to its respective **virtual network interface** (**VNIC**), and both VNICs go to the same `etherstub` (a kind of a virtual switch). Because of this, `etherstub` requires a virtual interface (`vnic0`). Finally, `etherstub` connects to a real interface (`net0`). The zonepath property for each zone and other properties are as follows:

*   zonepath zone1: `/myzones/zone1`
*   zonepath zone2: `/myzones/zone2`
*   IP type: exclusive-IP

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) that runs Oracle Solaris 11, with 4 GB (minimum) or 8 GB RAM (recommended), an extra disk with 80 GB, and a processor with two or more cores configured for this virtual machine, as shown in the following screenshot that was extracted from my VirtualBox environment:

![Getting ready](img/00022.jpeg)

## How to do it…

To start the procedure, we have to gather all current and relevant information about the system by running the following command:

```
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net1                phys      1500   up       --
net0                phys      1500   up       --
root@solaris11-1:~# ipadm show-if
IFNAME     CLASS    STATE    ACTIVE OVER
lo0        loopback ok       yes    --
net0       ip       ok       yes    --
net1       ip       ok       yes    --
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
net1/v4           static   ok           192.168.5.77/24
lo0/v6            static   ok           ::1/128
root@solaris11-1:~# zpool list
NAME      SIZE  ALLOC   FREE  CAP  DEDUP  HEALTH  ALTROOT
myzones  79.5G   544K  79.5G   0%  1.00x  ONLINE  -
rpool    79.5G  21.2G  58.3G  26%  1.00x  ONLINE  -
root@solaris11-1:~# zfs list | grep myzones
myzones                  494K  78.3G    31K  /myzones
root@solaris11-1:~# 
```

The system has two network interfaces (`net0` and `net1`), but only `net0` will be considered. Additionally, the pool (`myzones`) has almost 80 GB free space (you can create the myzones pool using `zpool create myzones <disk>`), and there is no filesystem under it. Then, the first step is to create the pool and one filesystem for each zone (`zone1` and `zone2`) by running the following commands:

```
root@solaris11-1:~# zpool create myzones c7t2d0
root@solaris11-1:~# zfs create myzones/zone1
root@solaris11-1:~# zfs list myzones/zone1
root@solaris11-1:/myzones# zfs create myzones/zone1
root@solaris11-1:/myzones# zfs create myzones/zone2
root@solaris11-1:/myzones# zfs list | grep zone
myzones                  314K  78.3G    33K  /myzones
myzones/zone1             31K  78.3G    31K  /myzones/zone1
myzones/zone2             31K  78.3G    31K  /myzones/zone2
```

The storage requirements have been met and now, the next important part of this recipe is to prepare all network infrastructures. To accomplish this task, it will be necessary to create `etherstub` (`vswitch1`) and three VNICs: `vnic0` (`etherstub`), `vnic1` (`zone1`), and `vnic2` (`zone2`). Moreover, we have to connect all VNICs into `etherstub` (`vswitch1`). All these tasks are accomplished by executing the following commands:

```
root@solaris11-1:~# dladm create-etherstub vswitch1
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net1                phys      1500   up       --
net0                phys      1500   up       --
vswitch1            etherstub 9000   unknown  --
root@solaris11-1:~# dladm create-vnic -l vswitch1 vnic0
root@solaris11-1:~# dladm create-vnic -l vswitch1 vnic1
root@solaris11-1:~# dladm create-vnic -l vswitch1 vnic2
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net1                phys      1500   up       --
net0                phys      1500   up       --
vswitch1            etherstub 9000   unknown  --
vnic0               vnic      9000   up       vswitch1
vnic1               vnic      9000   up       vswitch1
vnic2               vnic      9000   up       vswitch1
root@solaris11-1:~# dladm show-vnic
LINK          OVER         SPEED  MACADDRESS        MACADDRTYPE   VID
vnic0         vswitch1     40000  2:8:20:d:b:3b     random        0
vnic1         vswitch1     40000  2:8:20:ef:b6:63   random        0
vnic2         vswitch1     40000  2:8:20:ce:b0:da   random        0
```

Now, it's time to create the first zone (`zone1`) using `ip-type=exclusive` (this is the default value) and `vnic1` as a physical network interface:

```
root@solaris11-1:~# zonecfg -z zone1
Use 'create' to begin configuring a new zone.
zonecfg:zone1> create
create: Using system default template 'SYSdefault'
zonecfg:zone1> set autoboot=true
zonecfg:zone1> set zonepath=/myzones/zone1
zonecfg:zone1> add net
zonecfg:zone1:net> set physical=vnic1
zonecfg:zone1:net> end
zonecfg:zone1> info
zonename: zone1
zonepath: /myzones/zone1
brand: solaris
autoboot: true
bootargs: 
file-mac-profile: 
pool: 
limitpriv: 
scheduling-class: 
ip-type: exclusive
hostid: 
fs-allowed: 
net:
  address not specified
  allowed-address not specified
  configure-allowed-address: true
  physical: vnic1
  defrouter not specified
anet:
  linkname: net0
  lower-link: auto
 allowed-address not specified
  configure-allowed-address: true
  defrouter not specified
  allowed-dhcp-cids not specified
  link-protection: mac-nospoof
  mac-address: random
  mac-prefix not specified
  mac-slot not specified
  vlan-id not specified
  priority not specified
  rxrings not specified
  txrings not specified
  mtu not specified
  maxbw not specified
  rxfanout not specified
  vsi-typeid not specified
  vsi-vers not specified
  vsi-mgrid not specified
  etsbw-lcl not specified
  cos not specified
  pkey not specified
  linkmode not specified
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~#
```

To configure `zone2`, almost the same steps (the zone info details were omitted) need to be followed by running the following command:

```
root@solaris11-1:~# zonecfg -z zone2
Use 'create' to begin configuring a new zone.
zonecfg:zone2> create
create: Using system default template 'SYSdefault'
zonecfg:zone2> set autoboot=true
zonecfg:zone2> set zonepath=/myzones/zone2
zonecfg:zone2> add net
zonecfg:zone2:net> set physical=vnic2
zonecfg:zone2:net> end
zonecfg:zone2> verify
zonecfg:zone2> commit
zonecfg:zone2> exit

```

To list the recently configured zones, execute the following command:

```
root@solaris11-1:~# zoneadm list -cv
ID NAME           STATUS     PATH                     BRAND    IP    
 0 global         running    /                        solaris  shared
 - zone1          configured /myzones/zone1           solaris  excl  
 - zone2          configured /myzones/zone2           solaris  excl  
```

According to the previous recipe, during the first login that happens soon after installing the zone, it is required to provide interactively the system configuration information through eleven screens. To automate and make this simpler, it is feasible to create a system configuration file for each zone and provide it during each zone installation. To accomplish this task, some information will be asked from it:

For `zone1`, the information is as follows:

*   Computer name: `zone1`
*   Ethernet network configuration: `Manually`
*   Network interface: `vnic1`
*   IP address: `192.168.1.51`
*   DNS: `Do not configure DNS`
*   Alternate name server: `None`
*   Time zone: `(your time zone)`
*   Date and time: `(your current date and time)`
*   Root password: `(your choice)`
*   Your real name: `Alexandre Borges`
*   Username: `aborges1`
*   Password: `hacker123!`
*   E-mail: `anonymous@oracle.com`
*   Internet access method: `No proxy`

For `zone2`, the information is as follows:

*   Computer name: `zone2`
*   Ethernet network configuration: `Manually`
*   Network interface: `vnic2`
*   IP address: `192.168.1.52`
*   DNS: `Do not configure DNS`
*   Alternate name server: `None`
*   Time zone: `(your time zone)`
*   Date and time: `(your current date and time)`
*   Root password: `(your choice)`
*   Your real name: `Alexandre Borges`
*   Username: `aborges2`
*   Password: `hacker123!`
*   E-mail: `anonymous@oracle.com`
*   Internet access method: `No proxy`

Create a directory that will hold the zone profiles as follows:

```
root@solaris11-1:~# mkdir /zone_profiles

```

Create a profile to `zone1` by executing the following command:

```
root@solaris11-1:~# sysconfig create-profile -o /zone_profiles/zone1.xml

```

By using the almost the same command, create a profile to `zone2` by running the following command:

```
root@solaris11-1:~# sysconfig create-profile -o /zone_profiles/zone2.xml

```

To visualize the system configuration content, execute the following command:

```
root@solaris11-1:~# more /zone_profiles/zone1.xml 
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<service_bundle type="profile" name="sysconfig">
  <service version="1" type="service" name="system/config-user">
    <instance enabled="true" name="default">
      <property_group type="application" name="root_account">
        <propval type="astring" name="login" value="root"/>
        <propval type="astring" name="password" value="$5$Iabvrv4s$wAqPBNvP7QBZ12ocIdDp/TzNP8Gyv5PBvkTk1QTUEeA"/>
        <propval type="astring" name="type" value="role"/>
      </property_group>
      <property_group type="application" name="user_account">
        <propval type="astring" name="login" value="aborges1"/>
        <propval type="astring" name="password" value="$5$XfpOXWq9$1roklDSW7LW1Iq0pdpxq5Js16/d4DszHHlZB2AvYRL7"/>
        <propval type="astring" name="type" value="normal"/>
        <propval type="astring" name="description" value="Alexandre Borges"/>
        <propval type="count" name="gid" value="10"/>
        <propval type="astring" name="shell" value="/usr/bin/bash"/>
        <propval type="astring" name="roles" value="root"/>
        <propval type="astring" name="profiles" value="System Administrator"/>
        <propval type="astring" name="sudoers" value="ALL=(ALL) ALL"/>
(truncated output)

```

Now, it is time to install `zone1` and `zone2` using their respective system configuration files, as configured previously. Therefore, to perform this task, we'll be using our local repository (as learned in [Chapter 1](part0015_split_000.html#page "Chapter 1. IPS and Boot Environments"), *IPS and Boot Environments*) and executing the following command:

```
root@solaris11-1:~# pkg publisher
PUBLISHER           TYPE     STATUS P LOCATION
solaris             origin   online F http://solaris11-1.example.com/
root@solaris11-1:~# zoneadm -z zone1 install -c /zone_profiles/zone1.xml
root@solaris11-1:~# zoneadm -z zone2 install -c /zone_profiles/zone2.xml
root@solaris11-1:~# zoneadm list -iv
  ID NAME        STATUS     PATH                      BRAND    IP    
   0 global      running    /                         solaris  shared
   - zone1       installed  /myzones/zone1            solaris  excl  
   - zone2       installed  /myzones/zone2            solaris  excl  
root@solaris11-1:~#
```

Initiate both zones by running the following command:

```
root@solaris11-1:~# zoneadm list -iv
ID NAME         STATUS     PATH                       BRAND    IP    
 0 global       running    /                          solaris  shared
 - zone1        installed  /myzones/zone1             solaris  excl  
 - zone2        installed  /myzones/zone2             solaris  excl  
root@solaris11-1:~# zoneadm -z zone1 boot
root@solaris11-1:~# zoneadm -z zone2 boot

```

It is appropriate to check the network information before logging into zones by executing the following command:

```
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net1                phys      1500   up       --
net0                phys      1500   up       --
vswitch1            etherstub 9000   unknown  --
vnic0               vnic      9000   up       vswitch1
vnic1               vnic      9000   up       vswitch1
zone1/vnic1         vnic      9000   up       vswitch1
vnic2               vnic      9000   up       vswitch1
zone2/vnic2         vnic      9000   up       vswitch1
zone1/net0          vnic      1500   up       net0
zone2/net0          vnic      1500   up       net0
root@solaris11-1:~# dladm show-vnic
LINK          OVER        SPEED  MACADDRESS        MACADDRTYPE    VID
vnic0         vswitch1    40000  2:8:20:d:b:3b     rand           0
vnic1         vswitch1    40000  2:8:20:ef:b6:63   random         0
zone1/vnic1   vswitch1    40000  2:8:20:ef:b6:63   random         0
vnic2         vswitch1    40000  2:8:20:ce:b0:da   random         0
zone2/vnic2   vswitch1    40000  2:8:20:ce:b0:da   random         0
zone1/net0    net0        1000   2:8:20:ac:7d:b1   random         0
zone2/net0    net0        1000   2:8:20:f3:29:68   random         0
root@solaris11-1:~# ipadm show-addr
ADDROBJ           TYPE     STATE        ADDR
lo0/v4            static   ok           127.0.0.1/8
net0/v4           static   ok           192.168.1.144/24
net1/v4           static   ok           192.168.5.77/24
lo0/v6            static   ok           ::1/128
```

Now, we can log into the zones and test them by running the following command:

```
root@solaris11-1:~# zlogin zone1
[Connected to zone 'zone1' pts/5]
Oracle Corporation  SunOS 5.11  11.1  September 2012
root@zone1:~# ping 192.168.1.52
192.168.1.52 is alive
root@zone1:~# exit
logout
[Connection to zone 'zone1' pts/5 closed]

root@solaris11-1:~# zlogin zone2
[Connected to zone 'zone2' pts/5]
Oracle Corporation  SunOS 5.11  11.1  September 2012
root@zone2:~# ping 192.168.1.51
192.168.1.51 is alive
root@zone2:~# exit
logout
[Connection to zone 'zone2' pts/5 closed]
root@solaris11-1:~#
```

Everything is working. Zones are simply amazing!

### An overview of the recipe

The great news from this recipe was that we configured a virtual switch (`etherstub`) and three virtual network interfaces. Afterwards, we used these objects to create two zones using the virtual network concept.

# Managing a zone using the resource manager

Installing and configuring Oracle Solaris 11 non-global zones is great, and as we have mentioned previously, it is a great technique that isolates and runs applications without disturbing other applications if anything goes wrong. Nonetheless, there's still a problem. Each non-global zone runs in a global zone as it were running alone, but an inconvenient effect comes up if one of these zones takes all resources (the processor and memory) for itself, leaving little or nothing for the other zones. Based on this situation, a solution named resource manager can be deployed to control how many resources are consumed for each zone.

Focusing on the resource manager (without thinking about zones), there are many forms that enforce resource control in Oracle Solaris 11\. For example, we can use a project (`/etc/project`), which is composed by tasks and each one of these tasks contains one or more processes. A new project is created using the `projadd` command, and a new task can be created using the `newtask` command through a **Service Management Facility** (**SMF**) or even when a session is opened. Enabling the Resource Manager service and associating resources such as processors and memory to this project helps to create an upper limit of about how much of the resources (processors and memory) the processes bound to this project can use for themselves. Anyway, the existing project on Oracle Solaris 11 can be listed by running the `projects -l` command.

There are some methods that are available to associate resources with a project. The first way uses resource controls (the `rctladm` and `prctl` commands) to administer and view assigned controls to projects. The disadvantage of this method is that this approach restricts used resources by processes and prevents them from taking more processors or memory, if required. The other associated and possible problem is that the administrator has to know exactly how many resources are used by the application to make a good resource project, because if insufficient resources are assigned to a project or application, it can stop working.

The second good way to control how many resources can be taken by an application is to use the **fair share scheduler** (**FSS**) class that helps us moderate the resource allocation (the processor time) according to the resource requirement. A real advantage is that if an application is not using all assigned resources (the processor time), other applications can use the free resources from the first application. Therefore, this sharing of resources works like a dynamic resource control that spreads resources according to a plan (shares are assigned to applications) and changes its distribution based on demands. For example, when I personally use FSS, I normalize the available shares to 100 points in order to make a comparison with percentage easy. For project A, I grant 30 shares; for project B, I assign 50 shares; and for project C, I assign 20 shares. In the end, the distribution of the time processor is that app A gets 30 percent, app B gets 50 percent, and app C gets 20 percent. This is simple, isn't it?

The third way to deploy a resource manager is by using resource pools. Fundamentally, the idea is to assign resources to a resource group (or pool) and afterwards, to associate this pool with a project or application. Similar to what we have explained for FSS, the processor sets (group of processors) are normally assigned to resource pools and the latter is assigned to a project. Resource pools present a better flexibility because they permit us to set a minimum and maximum number of processors to be used by the application based on the demand. For example, it would be possible to assign a range from one to eight cores (or processors) to a project, and according to the resource demand, fewer or more processors would be used. Moreover, a specific processor (or core) could be dedicated to a processor set, if required. A small disadvantage of using the resource pool is that the processor is restricted to the pool, and even if there is a free resource (the processor), it cannot be used by another application. Personally, I prefer to manage and work with FSS because its flexibility and reusability offers you the opportunity to free up resources that can be used by other applications or projects. Nonetheless, it is feasible to mix resource pools with FSS and projects and have an advantage by implementing the controlled environment.

In the end, all of these techniques that control resources can be deployed in the zone context to limit the used resources by running applications, as we are going to learn in this recipe.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running on a processor with two or more cores, with 8 GB RAM and an 80 GB hard disk. To make the following procedure easier, we will take zones that were used in the previous recipe, and then the reader can assume that this recipe is a simple continuation.

## How to do it…

Basically, this recipe is composed of two parts. In the first part, the resource pools are configured, and in the second part, the existing resource pools are bound to zones.

To begin, we have to gather information about the existing zones by running the following command:

```
root@solaris11-1:~# zoneadm list -iv
ID NAME          STATUS     PATH                      BRAND    IP    
 0 global        running    /                         solaris  shared
 1 zone2         running    /myzones/zone2            solaris  excl  
 2 zone1         running    /myzones/zone1            solaris  excl  
root@solaris11-1:~#
```

The resource pool services have probably been stopped. We can verify them by executing the following command:

```
root@solaris11-1:~# svcs -a | grep pool
disabled       12:23:27 svc:/system/pools:default
disabled       12:23:35 svc:/system/pools/dynamic:default
```

Checking for dependencies from each service is done by executing the following command:

```
root@solaris11-1:~# svcs -d svc:/system/pools:default
STATE          STIME    FMRI
online         12:23:42 svc:/system/filesystem/minimal:default
root@solaris11-1:~# svcs -d svc:/system/pools/dynamic:default
STATE          STIME    FMRI
disabled       12:23:27 svc:/system/pools:default
online         12:24:08 svc:/system/filesystem/local:default
```

As the `svc:/system/pools/dynamic:default` service depends on `svc:/system/pools:default`, it is recommended that you enable both of them by running the following commands:

```
root@solaris11-1:~# svcadm enable -r svc:/system/pools/dynamic:default
root@solaris11-1:~# svcs -a | grep pool
online         14:30:31 svc:/system/pools:default
online         14:30:37 svc:/system/pools/dynamic:default
root@solaris11-1:~# svcs -p svc:/system/pools/dynamic:default
STATE          STIME    FMRI
online         14:30:37 svc:/system/pools/dynamic:default
               14:30:37     5443 poold
```

When a resource pool control is enabled, a default pool (`pool_default`) and a default processor set (`default_pset`) including all resources from the system are created, as verified by executing the following command:

```
root@solaris11-1:~# pooladm
system default
  string  system.comment 
  int  system.version 1
  boolean  system.bind-default true
  string  system.poold.objectives wt-load

  pool pool_default
    int  pool.sys_id 0
    boolean  pool.active true
    boolean  pool.default true
    int  pool.importance 1
    string  pool.comment 
    pset  pset_default

  pset pset_default
    int  pset.sys_id -1
    boolean  pset.default true
    uint  pset.min 1
    uint  pset.max 65536
    string  pset.units population
    uint  pset.load 211
    uint  pset.size 4
    string  pset.comment 

    cpu
      int  cpu.sys_id 1
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int  cpu.sys_id 3
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int  cpu.sys_id 0
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int  cpu.sys_id 2
      string  cpu.comment 
      string  cpu.status on-line
```

According to this output, there is a default pool (`pool_default`); the real processor has four cores (range 0 to 3), and all of them consist of a processor set (`pset`). However, this resource pool configuration is in the memory and is not persistent in the disk. Therefore, to save this into a configuration file, execute the following commands:

```
root@solaris11-1:~# pooladm -s
root@solaris11-1:~# more /etc/pooladm.conf 
<?xml version="1.0"?>
<!DOCTYPE system PUBLIC "-//Sun Microsystems Inc//DTD Resource Management All//EN" "file:///usr/share/lib/xml/dtd/rm_pool.dtd.1">
<!--
Configuration for pools facility. Do NOT edit this file by hand - use poolcfg(1) or libpool(3POOL) instead.
-->
<system ref_id="dummy" name="default" comment="" version="1" bind-default="true">
  <property name="system.poold.objectives" type="string">wt-load</property>
  <pool name="pool_default" active="true" default="true" importance="1" comment="" res="pset_-1" ref_id="pool_0">
    <property name="pool.sys_id" type="int">0</property>
  </pool>
  <res_comp type="pset" sys_id="-1" name="pset_default" default="true" min="1" max="65536" units="population" comment="" ref_id="pset_-1">
    <property name="pset.load" type="uint">176</property>
    <property name="pset.size" type="uint">4</property>
    <comp type="cpu" sys_id="1" comment="" ref_id="cpu_1">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
    <comp type="cpu" sys_id="3" comment="" ref_id="cpu_3">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
    <comp type="cpu" sys_id="0" comment="" ref_id="cpu_0">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
    <comp type="cpu" sys_id="2" comment="" ref_id="cpu_2">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
  </res_comp>
</system>
```

From this point, the following steps create a processor set (`pset`) with two cores, create a pool, and associate the processor set with this pool. Later, this pool will be assigned to the zone configuration, which can be shown as the processor set | pool | zone.

Thus, to create a processor set (`first_pset`) with one core at minimum (`pset.min=1`) and two cores (`pset.max=2`) at maximum, execute the following commands:

```
root@solaris11-1:~# poolcfg -c 'create pset first_pset (uint pset.min = 1; uint pset.max = 2)'
root@solaris11-1:~# poolcfg -c 'info pset first_pset'

pset first_pset
  int  pset.sys_id -2
  boolean  pset.default false
  uint  pset.min 1
  uint  pset.max 2
  string  pset.units population
  uint  pset.load 0
  uint  pset.size 0
  string  pset.comment
```

Now, we can create a pool named `first_pool`, which initially has all resources (four core processors) bound to it, by running the following commands:

```
root@solaris11-1:~# poolcfg -c 'create pool first_pool'
root@solaris11-1:~# poolcfg -c 'info pool first_pool'

pool first_pool
  boolean  pool.active true
  boolean  pool.default false
  int  pool.importance 1
  string  pool.comment 
  pset  pset_default

 pset pset_default
    int  pset.sys_id -1
 boolean  pset.default true
    uint  pset.min 1
    uint  pset.max 65536
    string  pset.units population
    uint  pset.load 176
    uint  pset.size 4
    string  pset.comment 

    cpu
      int  cpu.sys_id 1
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int  cpu.sys_id 3
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int  cpu.sys_id 0
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int  cpu.sys_id 2
      string  cpu.comment 
      string  cpu.status on-line

root@solaris11-1:~#
```

Then, assign the `first_pool` pool to the `first_pset` processor set by executing the following commands:

```
root@solaris11-1:~# poolcfg -c 'associate pool first_pool (pset first_pset)'
root@solaris11-1:~# poolcfg -c 'info pool first_pool'
pool first_pool
  boolean  pool.active true
  boolean  pool.default false
  int  pool.importance 1
  string  pool.comment 
   pset  first_pset

  pset first_pset
    int  pset.sys_id -2
    boolean  pset.default false
    uint  pset.min 1
    uint  pset.max 2
    string  pset.units population
    uint  pset.load 0
    uint  pset.size 0
    string  pset.comment 

root@solaris11-1:~#
```

So far, everything has been working well. Now, we have to check whether this new pool already appears in the resource memory configuration by executing the following command:

```
root@solaris11-1:~# poolcfg -c info
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool pool_default
    int      pool.sys_id 0
    boolean  pool.active true
    boolean  pool.default true
    int      pool.importance 1
    string   pool.comment 
    pset     pset_default

 pool first_pool
 boolean  pool.active true
 boolean  pool.default false
 int      pool.importance 1
 string   pool.comment 
 pset     first_pset

  pset pset_default
    int      pset.sys_id -1
    boolean  pset.default true
    uint     pset.min 1
    uint     pset.max 65536
    string   pset.units population
    uint     pset.load 176
    uint     pset.size 4
    string   pset.comment 

    cpu
      int     cpu.sys_id 1
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 3
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 0
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 2
      string  cpu.comment 
      string  cpu.status on-line

  pset first_pset
    int      pset.sys_id -2
    boolean  pset.default false
    uint     pset.min 1
    uint     pset.max 2
    string   pset.units population
    uint     pset.load 0
    uint     pset.size 0
    string   pset.comment 
```

We have realized that the `first_pset` configuration is still not persistent in the pool configuration file. To validate (the -`n -c` option) and commit (the `-c` option) the new configuration, execute the following commands:

```
root@solaris11-1:~# pooladm -n -c
root@solaris11-1:~# pooladm -c
root@solaris11-1:~# more /etc/pooladm.conf
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE system PUBLIC "-//Sun Microsystems Inc//DTD Resource Management All//EN" "file:///usr/share/lib/xml/dtd/rm_pool.dtd.1">
<!--
Configuration for pools facility. Do NOT edit this file by hand - use poolcfg(1) or libpool(3POOL) instead.
-->
<system ref_id="dummy" name="default" comment="" version="1" bind-default="true">
  <property name="system.poold.objectives" type="string">wt-load</property>
  <pool name="pool_default" active="true" default="true" importance="1" comment="" res="pset_-1" ref_id="pool_0">
    <property name="pool.sys_id" type="int">0</property>
  </pool>
  <res_comp type="pset" sys_id="-1" name="pset_default" default="true" min="1" max="65536" units="population" comment="" ref_id="pset_-1">
    <property name="pset.load" type="uint">176</property>
    <property name="pset.size" type="uint">4</property>
    <comp type="cpu" sys_id="1" comment="" ref_id="cpu_1">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
    <comp type="cpu" sys_id="3" comment="" ref_id="cpu_3">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
    <comp type="cpu" sys_id="0" comment="" ref_id="cpu_0">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
    <comp type="cpu" sys_id="2" comment="" ref_id="cpu_2">
      <property name="cpu.status" type="string">on-line</property>
    </comp>
  </res_comp>
  <res_comp ref_id="id_0" sys_id="-2" type="pset" name="first_pset" min="1" max="2" units="population" comment="">
    <property name="pset.load" type="uint">0</property>
    <property name="pset.size" type="uint">0</property>
  </res_comp>
  <property name="system._next_id" type="uint">2</property>
  <pool ref_id="id_1" res="id_0" name="first_pool" active="true" importance="1" comment=""/>
</system>
root@solaris11-1:~#
```

Everything is ready. Nevertheless, it's easy to verify that the configuration is active only in the memory (the kernel state) using the `-dc` option, but it isn't saved in the resource pool configuration file (option `-c`) as follows:

```
root@solaris11-1:~# poolcfg -dc info
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool first_pool
    int      pool.sys_id 1
    boolean  pool.active true
    boolean  pool.default false
    int      pool.importance 1
    string   pool.comment 
 pset     first_pset

  pool pool_default
    int      pool.sys_id 0
    boolean  pool.active true
    boolean  pool.default true
    int      pool.importance 1
    string   pool.comment 
    pset     pset_default

 pset first_pset
 int      pset.sys_id 1
 boolean  pset.default false
 uint     pset.min 1
 uint     pset.max 2
 string   pset.units population
 uint     pset.load 0
 uint     pset.size 2
 string   pset.comment 

 cpu
 int     cpu.sys_id 1
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int  cpu.sys_id 0
 string  cpu.comment 
 string  cpu.status on-line

 pset pset_default
 int      pset.sys_id -1
 boolean  pset.default true
 uint     pset.min 1
 uint     pset.max 65536
 string   pset.units population
 uint     pset.load 151
 uint     pset.size 2
 string   pset.comment 

 cpu
 int     cpu.sys_id 3
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int     cpu.sys_id 2
 string  cpu.comment 
 string  cpu.status on-line

root@solaris11-1:~# poolcfg -c info
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool pool_default
    int  pool.sys_id 0
    boolean  pool.active true
    boolean  pool.default true
    int      pool.importance 1
    string   pool.comment 
    pset     pset_default

  pool first_pool
    boolean  pool.active true
    boolean  pool.default false
    int      pool.importance 1
    string   pool.comment 
 pset     first_pset

 pset pset_default
    int  pset.sys_id -1
    boolean  pset.default true
    uint     pset.min 1
    uint     pset.max 65536
    string   pset.units population
    uint     pset.load 176
    uint     pset.size 4
    string   pset.comment 

 cpu
 int     cpu.sys_id 1
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int     cpu.sys_id 3
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int     cpu.sys_id 0
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int     cpu.sys_id 2
 string  cpu.comment 
 string  cpu.status on-line

 pset first_pset
 int      pset.sys_id -2
 boolean  pset.default false
 uint     pset.min 1
 uint     pset.max 2
 string   pset.units population
 uint     pset.load 0
 uint     pset.size 0
    string   pset.comment 
```

To solve the problem of saving the resource pool configuration from the memory to disk, we can use the `-s` option by running the following command:

```
root@solaris11-1:~# pooladm -s
root@solaris11-1:~# poolcfg -c info
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool first_pool
    int      pool.sys_id 1
    boolean  pool.active true
    boolean  pool.default false
    int      pool.importance 1
    string   pool.comment 
    pset     first_pset

  pool pool_default
    int      pool.sys_id 0
    boolean  pool.active true
    boolean  pool.default true
    int      pool.importance 1
    string   pool.comment 
    pset     pset_default

 pset first_pset
    int      pset.sys_id 1
    boolean  pset.default false
    uint     pset.min 1
    uint     pset.max 2
    string   pset.units population
    uint     pset.load 0
    uint     pset.size 2
    string   pset.comment 

 cpu
 int     cpu.sys_id 1
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int     cpu.sys_id 0
 string  cpu.comment 
 string  cpu.status on-line

 pset pset_default
    int      pset.sys_id -1
    boolean  pset.default true
    uint     pset.min 1
    uint     pset.max 65536
    string   pset.units population
    uint     pset.load 201
    uint     pset.size 2
    string   pset.comment 

 cpu
 int     cpu.sys_id 3
 string  cpu.comment 
 string  cpu.status on-line

 cpu
 int     cpu.sys_id 2
 string  cpu.comment 
      string  cpu.status on-line
```

That is great! Listing the active resource pools is done by executing the `poolstat` command as follows:

```
root@solaris11-1:~# poolstat
                              pset
 id pool                 size used load
  1 first_pool              2 0.00 0.00
  0 pool_default            2 0.00 0.17
root@solaris11-1:~# poolstat -r all
id pool              type rid rset           min  max size used load
  1 first_pool        pset   1 first_pset     1    2    2   0.00 0.00
  0 pool_default      pset  -1 pset_default   1    66K  2   0.00 0.17
```

Associating the recently created pool (`first_pool`) to non-global `zone1` is done by executing the following command:

```
root@solaris11-1:~# zonecfg -z zone1 info | grep pool
pool: 

root@solaris11-1:~# zonecfg -z zone1 set pool=first_pool
root@solaris11-1:~# zonecfg -z zone1 info | grep pool
pool: first_pool
```

It is impossible to activate the bound resource pool without rebooting `zone1`, so execute the following commands:

```
root@solaris11-1:~# zoneadm -z zone1 shutdown -r
root@solaris11-1:~# zoneadm list -iv
ID NAME          STATUS     PATH                      BRAND    IP    
 0 global        running    /                         solaris  shared
 1 zone2         running    /myzones/zone2            solaris  excl  
 3 zone1         running    /myzones/zone1            solaris  excl  
```

Now, it is time to log in to `zone1` and check whether the `first_pool` pool is active by running the following command:

```
root@solaris11-1:~# zlogin zone1
[Connected to zone 'zone1' pts/3]
Oracle Corporation  SunOS 5.11  11.1  September 2012
root@zone1:~# poolcfg -dc info
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool first_pool
    int      pool.sys_id 1
    boolean  pool.active true
    boolean  pool.default false
    int      pool.importance 1
    string   pool.comment 
    pset     first_pset

  pset first_pset
    int      pset.sys_id 1
    boolean  pset.default false
    uint     pset.min 1
    uint     pset.max 2
    string   pset.units population
    uint     pset.load 540
    uint     pset.size 2
    string   pset.comment 

    cpu
      int     cpu.sys_id 1
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 0
      string  cpu.comment 
      string  cpu.status on-line
root@zone1:~# psrinfo
0  on-line   since 02/01/2014 12:23:05
1  on-line   since 02/01/2014 12:23:07
root@zone1:~# psrinfo -v
Status of virtual processor 0 as of: 02/01/2014 15:52:47
  on-line since 02/01/2014 12:23:05.
  The i386 processor operates at 2470 MHz,
    and has an i387 compatible floating point processor.
Status of virtual processor 1 as of: 02/01/2014 15:52:47
  on-line since 02/01/2014 12:23:07.
  The i386 processor operates at 2470 MHz,
    and has an i387 compatible floating point processor.
```

Perfect! Two cores were associated with `zone1`, and any application running inside this zone can use these core processors.

To change the resource type focus, a very interesting method that limits the used memory is resource capping, which helps us limit the physical, swap, and locked memory.

For example, using the same `zone1`, let's change its configuration by executing the following commands:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> add capped-memory
zonecfg:zone1:capped-memory> set physical=1G
zonecfg:zone1:capped-memory> set swap=500M
zonecfg:zone1:capped-memory> end
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~# zonecfg -z zone1 info
zonename: zone1
zonepath: /myzones/zone1
brand: solaris
autoboot: true
(truncated)

capped-memory:
  physical: 1G
  [swap: 500M]
rctl:
  name: zone.max-swap
  value: (priv=privileged,limit=524288000,action=deny)
root@solaris11-1:~#
```

According to the previous output, the physical memory from `zone1` is limited to 1 GB, and the used swap space is restricted to 500 MB. Furthermore, there is a strange line for maximum swap:

```
value: (priv=privileged,limit=524288000,action=deny)

```

The interpretation for this line is as follows:

*   `privileged`: This can be modified only by privileged users (root). Another possible value is `basic` (only the owner can modify it).
*   `deny`: This can deny any requested resource for an amount above the limit value (500 MB). The other possibilities would be `none` (no action is taken even if the requested resource is above the limit) and `signal`, in which a signal is sent when the threshold value is exceeded.

Resource capping is a service implemented by the `rcapd` daemon, and this service can be enabled by the following command:

```
root@solaris11-1:~# svcs -a | grep rcap
disabled       21:56:20 svc:/system/rcap:default

root@solaris11-1:~# svcs  -d svc:/system/rcap:default
STATE          STIME    FMRI
online         21:56:31 svc:/system/filesystem/minimal:default
online         21:56:33 svc:/system/resource-mgmt:default
online         21:56:35 svc:/system/manifest-import:default
root@solaris11-1:~# svcadm enable svc:/system/rcap:default
root@solaris11-1:~# svcs -	a | grep rcap
online         22:52:06 svc:/system/rcap:default
root@solaris11-1:~# svcs -p svc:/system/rcap:default
STATE          STIME    FMRI
online         22:52:06 svc:/system/rcap:default
               22:52:06     5849 rcapd
```

Reboot `zone1` for memory capping to take effect. It would be feasible to enable the resource capping daemon without rebooting and starting the daemon now by running the following command:

```
root@solaris11-1:~# rcapadm -E -n

```

To monitor the action of the `rcap` daemon (`rcapd`), execute the following commands:

```
root@solaris11-1:~# zoneadm -z zone1 shutdown -r
root@solaris11-1:~# zoneadm list -iv
ID NAME             STATUS     PATH                 BRAND    IP    
 0 global           running    /                    solaris  shared
 1 zone2            running    /myzones/zone2       solaris  excl  
 3 zone1            running    /myzones/zone1       solaris  excl  
root@solaris11-1:~# rcapstat -z
id zone            nproc    vm   rss   cap    at avgat    pg avgpg
 3 zone1               -   26M   38M 1024M    0K    0K    0K    0K
 3 zone1               -   31M   44M 1024M    0K    0K    0K    0K
 3 zone1               -   31M   44M 1024M    0K    0K    0K    0K
```

The used physical memory (RSS) is below the memory capping limit (1024 MB). If the physical memory is increased, its limit is 1024 MB. Nice!

To make this example more attractive, some changes can be made. Let's remove the `first_pool` resource pool (and any other existing pool) from `zone1`. Additionally, the `first_pool` pool will be deleted by the `pooladm -x` command. Obviously, the new pool configuration must be saved by the `pooladm -s` command. The following is the sequence:

```
root@solaris11-1:~# zonecfg -z zone1 clear pool
root@solaris11-1:~# zoneadm -z zone1 shutdown -r
root@solaris11-1:~# pooladm -x
root@solaris11-1:~# pooladm -s
root@solaris11-1:~# pooladm 
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool pool_default
    int      pool.sys_id 0
    boolean  pool.active true
    boolean  pool.default true
    int      pool.importance 1
    string   pool.comment 
    pset     pset_default

  pset pset_default
    int      pset.sys_id -1
    boolean  pset.default true
    uint     pset.min 1
    uint     pset.max 65536
    string   pset.units population
    uint     pset.load 15511
    uint     pset.size 4
    string   pset.comment 

    cpu
      int     cpu.sys_id 1
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 3
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 0
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 2
      string  cpu.comment 
      string  cpu.status on-line
```

Everything has returned to the default status, and from this point, `zone1` doesn't have a special associated pool. This permits us to focus on FSS from now on.

The following command checks what the current default kernel scheduling class is:

```
root@solaris11-1:~# dispadmin -d
dispadmin: Default scheduling class is not set
```

There is no default scheduling class. If we want to use FSS, then it would be appropriate to configure it on the global zone because this setting will be inherited by all non-global zones. To configure the FSS as explained, execute the following command:

```
root@solaris11-1:~# dispadmin -d FSS
root@solaris11-1:~# dispadmin -d
FSS  (Fair Share)
```

This setup only takes effect after a system is rebooted. After the system has been reinitiated, all processes will be classified as FSS. Nonetheless, to enforce it now without a reboot, execute the following command:

```
root@solaris11-1:~# priocntl -s -c FSS

```

Unfortunately, all current processes are still running under other scheduling classes and only new processes will take the FSS setting. This can be verified by running the following command:

```
root@solaris11-1:~# ps -efcZ | more
  ZONE     UID   PID  PPID  CLS PRI    STIME TTY    TIME CMD
global    root     0     0  SYS  96 00:04:41 ?      0:01 sched
global    root     5     0  SDC  99 00:04:38 ?      0:07 zpool-rpool
global    root     6     0  SDC  99 00:04:42 ?      0:01 kmem_task
global    root     1     0   TS  59 00:04:42 ?      0:00 /usr/sbin/init
global    root     2     0  SYS  98 00:04:42 ?      0:00 pageout
global    root     3     0  SYS  60 00:04:42 ?      0:00 fsflush
global    root     7     0  SYS  60 00:04:42 ?      0:00 intrd
global    root     8     0  SYS  60 00:04:42 ?      0:00 vmtasks
global    root   115     1   TS  59 00:05:09 ?      0:00 /usr/lib/pfexecd
global    root    11     1   TS  59 00:04:48 ?      0:13 /lib/svc/bin/svc.startd
global    root    13     1   TS  59 00:04:48 ?      0:33 /lib/svc/bin/svc.configd
global    root   911     1   TS  59 02:05:55 ?      0:00 
(truncated output)

```

Again, it's unnecessary to wait for the next reboot. Therefore, all processes can be moved from their current scheduling classes to FSS by executing the following commands:

```
root@solaris11-1:~# priocntl -s -c FSS -i all
root@solaris11-1:~# ps -efcZ | more
  ZONE      UID   PID  PPID  CLS PRI    STIME TTY    TIME CMD
global     root     0     0  SYS  96 00:04:41 ?      0:01 sched
global     root     5     0  SDC  99 00:04:38 ?      0:12 zpool-rpool
global     root     6     0  SDC  99 00:04:42 ?      0:02 kmem_task
global     root     1     0  FSS  29 00:04:42 ?      0:00 /usr/sbin/init
global     root     2     0  SYS  98 00:04:42 ?      0:00 pageout
global     root     3     0  SYS  60 00:04:42 ?      0:01 fsflush
global     root     7     0  SYS  60 00:04:42 ?      0:00 intrd
global     root     8     0  SYS  60 00:04:42 ?      0:00 vmtasks
global     root   115     1  FSS  29 00:05:09 ?      0:00 /usr/lib/pfexecd
global     root    11     1  FSS  29 00:04:48 ?      0:13 /lib/svc/bin/svc.startd
global     root    13     1  FSS  29 00:04:48 ?      0:33 /lib/svc/bin/svc.configd
(truncated output)

```

When FSS is set up as the default scheduling class in the global zone, all non-global zones automatically take this configuration. To verify this, run the following command:

```
root@solaris11-1:~# zlogin zone1
 [Connected to zone 'zone1' pts/4]
Oracle Corporation  SunOS 5.11  11.1  September 2012
root@zone1:~# ps -efcZ | more
 ZONE      UID   PID  PPID  CLS PRI    STIME TTY         TIME CMD
zone1     root  3944  2454  FSS  29 02:06:47 ?           0:00 /usr/sbin/init
zone1     root  4284  2454  FSS  29 02:06:58 ?           0:06 /lib/svc/bin/svc.startd
zone1     root  2454  2454  SYS  60 02:06:29 ?           0:00 zsched
zone1     root  5479  2454  FSS  59 02:48:52 pts/4       0:00 /usr/bin/login -z global -f root
zone1     root  4287  2454  FSS  29 02:07:00 ?           0:21 /lib/svc/bin/svc.configd
zone1   netcfg  4448  2454  FSS  29 02:07:27 ?           0:00 /lib/inet/netcfgd
zone1     root  4922  2454  FSS  29 02:08:21 ?           0:00 
(truncated output)

```

We can realize that all main processes from `zone1` are under the FSS class. Anyway, it is recommended that the FSS class be explicitly configured in the non-global settings in order to prevent possible mistakes in the future. Therefore, execute the following command:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> set scheduling-class=FSS
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~#
root@solaris11-1:~# zonecfg -z zone2
zonecfg:zone2> set scheduling-class=FSS
zonecfg:zone2> verify
zonecfg:zone2> commit
zonecfg:zone2> exit
root@solaris11-1:~# 
```

Finally, it is the right moment to use the FSS class to configure some shares for each zone (`zone1` and `zone2`). This way, it is possible to share an amount (70 percent) from the CPU processing for `zone1` and the other amount (30 percent) from the CPU processing for `zone2`. The following is the procedure:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> add rctl 
zonecfg:zone1:rctl> set name=zone.cpu-shares
zonecfg:zone1:rctl> add value (priv=privileged, limit=70,action=none)
zonecfg:zone1:rctl> end
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~# zonecfg -z zone2
zonecfg:zone2> add rctl
zonecfg:zone2:rctl> set name=zone.cpu-shares
zonecfg:zone2:rctl> add value (priv=privileged,limit=30,action=none)
zonecfg:zone2:rctl> end
zonecfg:zone2> verify
zonecfg:zone2> commit
zonecfg:zone2> exit

```

This is excellent! Shares were assigned to `zone1` (70 shares) and `zone2` (30 shares) using the `zonecfg` command in a persistent way. For both the zones to take effect, execute the following commands:

```
root@solaris11-1:~# zoneadm -z zone1 shutdown -r
root@solaris11-1:~# zoneadm -z zone2 shutdown -r

```

The processor time can be followed and monitored using the following command:

```
root@solaris11-1:~# prstat -Z
  PID USERNAME  SIZE   RSS STATE   PRI NICE      TIME  CPU PROCESS/NLWP      
  4466 root      216M   98M sleep    59    0   0:00:41 0.7% java/25
  4702 root      129M   19M sleep    59    0   0:00:06 0.5% gnome-terminal/2
rcapd/1
     5 root        0K    0K sleep    99  -20   0:00:19 0.2% zpool-rpool/138
   898 root       53M   18M sleep    53    0   0:00:06 0.1% poold/9
  (omitted output)
 automountd/2
   198 root     1780K  788K sleep    29    0   0:00:00 0.0% utmpd/1
   945 root     2392K 1552K sleep    59    0   0:00:00 0.0% ttymon/1
ZONEID    NPROC  SWAP   RSS MEMORY      TIME  CPU ZONE      
     0      117 2885M  794M   9.5%   0:03:28 2.5% global    
     2       28  230M   62M   0.7%   0:00:30 0.1% zone1     
     1       28  230M   64M   0.7%   0:00:29 0.1% zone2      
```

Surprisingly, it is feasible to change the `zone.cpu-shares` attribute dynamically without rebooting zones but in a non-persistent way (all the changes are lost after a reboot) by running the following commands:

```
root@solaris11-1:~# prctl -n zone.cpu-shares -v 60 -r -i zone zone1
root@solaris11-1:~# prctl -n zone.cpu-shares -P -i zone zone1
zone: 3: zone1
zone.cpu-shares usage 60 - - -
zone.cpu-shares privileged 60 - none -
zone.cpu-shares system 65535 max none -
root@solaris11-1:~# prctl -n zone.cpu-shares -v 40 -r -i zone zone2
root@solaris11-1:~# prctl -n zone.cpu-shares -P -i zone zone2
zone: 4: zone2
zone.cpu-shares usage 40 - - -
zone.cpu-shares privileged 40 - none -
zone.cpu-shares system 65535 max none -
root@solaris11-1:~#
```

To collect information about the memory and CPU from both zones in an interval of 5 seconds, execute the following command:

```
root@solaris11-1:~#  zonestat -z zone1,zone2 -r physical-memory 5
Collecting data for first interval...
Interval: 1, Duration: 0:00:05
PHYSICAL-MEMORY              SYSTEM MEMORY
mem_default                          8191M
                                ZONE  USED %USED   CAP  %CAP
                             [total] 1464M 17.8%     -     -
                            [system]  624M 7.62%     -     -
                               zone2 63.9M 0.78%     -     -
                               zone1 3561K 0.04% 1024M 0.33%

Interval: 2, Duration: 0:00:10
PHYSICAL-MEMORY              SYSTEM MEMORY
mem_default                          8191M
                                ZONE  USED %USED   CAP  %CAP
                             [total] 1464M 17.8%     -     -
                            [system]  624M 7.62%     -     -
                               zone2 63.9M 0.78%     -     -
                               zone1 3485K 0.04% 1024M 0.33%
Removing all configured shares is quickly executed by running:
root@solaris11-1:~# zonecfg -z zone1 clear cpu-shares
root@solaris11-1:~# zonecfg -z zone2 clear cpu-shares
root@solaris11-1:~# zoneadm -z zone1 shutdown -r
root@solaris11-1:~# zoneadm -z zone2 shutdown -r

```

Keeping up with our approach about the resource manager, there's a zone resource, named `dedicated-cpu`, where it is possible to specify a subset of processors (or cores) to a non-global zone. For example, the following example shows us that `zone1` can use one to four processors (`ncpus=1-4`) according to the demand, and this setting has an `importance` value equal to `8` when competing for resources against other zones or configurations. This smart setup creates a temporary pool including any necessary processor inside it. The following is the sequence:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> add dedicated-cpu
zonecfg:zone1:dedicated-cpu> set ncpus=1-4
zonecfg:zone1:dedicated-cpu> set importance=8
zonecfg:zone1:dedicated-cpu> end
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~# zoneadm -z zone1 shutdown -r
root@solaris11-1:~# zlogin zone1
[Connected to zone 'zone1' pts/2]
Oracle Corporation  SunOS 5.11  11.1  September 2012
root@zone1:~# pooladm
system default
  string   system.comment 
  int      system.version 1
  boolean  system.bind-default true
  string   system.poold.objectives wt-load

  pool SUNWtmp_zone1
    int  pool.sys_id 1
    boolean  pool.active true
    boolean  pool.default false
    int      pool.importance 8
    string   pool.comment 
    boolean  pool.temporary true
    pset     SUNWtmp_zone1

  pset SUNWtmp_zone1
    int      pset.sys_id 1
    boolean  pset.default false
    uint     pset.min 1
    uint     pset.max 4
    string   pset.units population
    uint     pset.load 4
    uint     pset.size 2
    string   pset.comment 
    boolean  pset.temporary true

    cpu
      int     cpu.sys_id 1
      string  cpu.comment 
      string  cpu.status on-line

    cpu
      int     cpu.sys_id 0
      string  cpu.comment 
      string  cpu.status on-line
```

Amazing! To remove the `dedicated-cpu` resource from `zone1`, execute the following command:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> remove dedicated-cpu
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit

```

Before continuing, we must reboot the zone by running the following command:

```
root@solaris11-1:~# zoneadm -z zone1 shutdown -r

```

Another good technique to control zone resources is using the `capped-cpu` resource, which permits us to specify how big a percentage of a CPU the zone can use. The value to be specified means a percentage of CPUs, and this procedure can be performed by executing the following sequence:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> add capped-cpu
zonecfg:zone1:capped-cpu> set ncpus=2.5
zonecfg:zone1:capped-cpu> end
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~# zoneadm -z zone1 shutdown -r

```

According to the previous configuration, the `ncpus=2.5` attribute means 250 percent of CPUs or 2.5 CPUs. To remove the recently added resource, execute the following command:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> remove capped-cpu
zonecfg:zone1:capped-cpu> end
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit

```

After all the changes, we have to reboot the zone by executing the following command:

```
root@solaris11-1:~# zoneadm -z zone1 shutdown -r

```

This is outstanding! We have executed many trials with resource management, and all of them have worked! As `zone1` still has a resource capping (memory), it is time to remove it:

```
root@solaris11-1:~# zonecfg -z zone1
zonecfg:zone1> remove capped-memory
zonecfg:zone1> verify
zonecfg:zone1> commit
zonecfg:zone1> exit
root@solaris11-1:~# zoneadm -z zone1 shutdown -r

```

Finally, the resource capping feature can be disabled by executing the following command:

```
root@solaris11-1:~# svcs -a | grep rcap
online         18:49:28 svc:/system/rcap:default
root@solaris11-1:~# rcapadm -D
                                      state: disabled
           memory cap enforcement threshold: 0%
                    process scan rate (sec): 15
                 reconfiguration rate (sec): 60
                          report rate (sec): 5
                    RSS sampling rate (sec): 5
root@solaris11-1:~# svcs -a | grep rcap
disabled       19:28:33 svc:/system/rcap:default
```

Another way of disabling the resource capping feature would be to execute the following command:

```
root@solaris11-1:~# svcadm disable svc:/system/rcap:default

```

Perfect! Everything has returned to the initial setup.

### An overview of the recipe

This section was very long, and we could learn lots of details about resource management controls and how to limit processors and the memory. In the next chapter, we are going to handle the network resource control.

# Implementing a flow control

In the last subsection, we handled resource control on processors and memory. In Oracle Solaris 11, the network control has acquired importance and relevance, allowing us to set a network flow control based on TCP/IP services and ports. Read the next pages to learn a bit more.

## Getting ready

This recipe requires a virtual machine (VMware or VirtualBox) that runs Oracle Solaris 11 on one processor, with 4 GB RAM and one physical network interface. To make our life simpler, we are going to reuse the same environment as the one in the previous recipes.

## How to do it…

To be able to follow the steps in this section, you need to check the current environment setup. Therefore, it is possible to gather information about existing virtual interfaces, virtual switches, and network interfaces by running the following commands:

```
root@solaris11-1:~# dladm show-vnic
LINK           OVER        SPEED  MACADDRESS        MACADDRTYPE  VID
vnic0          vswitch1    40000  2:8:20:d:b:3b     random       0
vnic1          vswitch1    40000  2:8:20:ef:b6:63   random       0
zone1/vnic1    vswitch1    40000  2:8:20:ef:b6:63   random       0
vnic2          vswitch1    40000  2:8:20:ce:b0:da   random       0
zone2/vnic2    vswitch1    40000  2:8:20:ce:b0:da   random       0
zone2/net0     net0        1000   2:8:20:f3:29:68   random       0
zone1/net0     net0        1000   2:8:20:ac:7d:b1   random       0
root@solaris11-1:~# dladm show-link
LINK                CLASS     MTU    STATE    OVER
net1                phys      1500   up       --
net0                phys      1500   up       --
vswitch1            etherstub 9000   unknown  --
vnic0               vnic      9000   up       vswitch1
vnic1               vnic      9000   up       vswitch1
zone1/vnic1         vnic      9000   up       vswitch1
vnic2               vnic      9000   up       vswitch1
zone2/vnic2         vnic      9000   up       vswitch1
zone2/net0          vnic      1500   up       net0
zone1/net0          vnic      1500   up       net0
```

As the existing virtual interfaces are currently assigned to non-global zones, create a new **virtual interface** (**VNIC**) and associate it with the `vswitch` virtual switch by executing the following commands:

```
root@solaris11-1:~# dladm create-vnic -l vswitch1 vnic5
root@solaris11-1:~# dladm show-vnic
LINK          OVER         SPEED  MACADDRESS        MACADDRTYPE   VID
vnic0         vswitch1     40000  2:8:20:d:b:3b     random        0
vnic1         vswitch1     40000  2:8:20:ef:b6:63   random        0
zone1/vnic    vswitch1     40000  2:8:20:ef:b6:63   random        0
vnic2         vswitch1     40000  2:8:20:ce:b0:da   random        0
zone2/vnic2   vswitch1     40000  2:8:20:ce:b0:da   random        0
zone2/net0    net0         1000   2:8:20:f3:29:68   random        0
zone1/net0    net0         1000   2:8:20:ac:7d:b1   random        0
vnic5         vswitch1     40000  2:8:20:c0:9a:f7   random        0
```

Create two flow controls on `vnic5`: the first one controls the TCP flow in the port `80` and the second one controls UDP in the same port `80` by executing the following commands:

```
root@solaris11-1:~# flowadm show-flow
root@solaris11-1:~# flowadm add-flow -l vnic5 -a transport=tcp,local_port=80 http_tcp_1
root@solaris11-1:~# flowadm add-flow -l vnic5 -a transport=udp,local_port=80 http_udp_1
root@solaris11-1:~# flowadm show-flow
FLOW        LINK          IPADDR         PROTO  LPORT   RPORT   DSFLD
http_tcp_1  vnic5         --             tcp    80      --      --
http_udp_1  vnic5         --             udp    80      --      --
```

According to the previous output, we named the flow controls `http_tcp_1` and `http_udp_1`; both control the HTTP data and use TCP and UDP as the transport protocol, respectively. Therefore, it is appropriate to bind a new property to this HTTP flow to control the maximum possible bandwidth and limit it to 50 MBps. Thus, run the following commands:

```
root@solaris11-1:~# flowadm set-flowprop -p maxbw=50M http_tcp_1
root@solaris11-1:~# flowadm set-flowprop -p maxbw=50M http_udp_1
root@solaris11-1:~# flowadm show-flowprop 
FLOW         PROPERTY        VALUE          DEFAULT        POSSIBLE
http_tcp_1   maxbw              50          --             -- 
http_udp_1   maxbw              50          --             -- 
root@solaris11-1:~#
```

We have set the bandwidth limit for port `80` (TCP and UDP) to 50 MBps at maximum. A specific flow can be monitored in a two-second interval for the received packages (illustrated in our recipe) by executing the following command:

```
root@solaris11-1:~# flowstat -r http_tcp_1 -i 2
           FLOW    IPKTS   RBYTES   IDROPS
     http_tcp_1        0        0        0
     http_tcp_1        0        0        0
     http_tcp_1        0        0        0
     http_tcp_1        0        0        0
```

Additionally, it is recommended that you analyze a more complete view, including sent and received packets, by running the following command:

```
root@solaris11-1:~# flowstat -i 2
           FLOW    IPKTS   RBYTES   IDROPS    OPKTS   OBYTES   ODROPS
     http_tcp_1        0        0        0        0        0        0
     http_udp_1        0        0        0        0        0        0
     http_tcp_1        0        0        0        0        0        0
     http_udp_1        0        0        0        0        0        0
     http_tcp_1        0        0        0        0        0        0
     http_udp_1        0        0        0        0        0        0
```

Finally, to remove both flow controls from the system and the `vnic5` interface, execute the following command:

```
root@solaris11-1:~# flowadm
FLOW        LINK          IPADDR         PROTO  LPORT   RPORT   DSFLD
http_tcp_1  vnic5         --             tcp    80      --      --
http_udp_1  vnic5         --             udp    80      --      --
root@solaris11-1:~# flowadm remove-flow http_tcp_1
root@solaris11-1:~# flowadm remove-flow http_udp_1
root@solaris11-1:~# flowadm show-flow
root@solaris11-1:~# dladm delete-vnic vnic5
root@solaris11-1:~# dladm show-vnic
LINK           OVER       SPEED  MACADDRESS        MACADDRTYPE    VID
vnic0          vswitch1   40000  2:8:20:d:b:3b     random         0
vnic1          vswitch1   40000  2:8:20:ef:b6:63   random         0
zone1/vnic1    vswitch1   40000  2:8:20:ef:b6:63   random         0
vnic2          vswitch1   40000  2:8:20:ce:b0:da   random         0
zone2/vnic2    vswitch1   40000  2:8:20:ce:b0:da   random         0
zone2/net0     net0       1000   2:8:20:f3:29:68   random         0
zone1/net0     net0       1000   2:8:20:ac:7d:b1   random         0
```

### An overview of the recipe

This recipe showed you how to implement, monitor, and unconfigure the flow over **virtual network interfaces** (**VNICs**), limiting the bandwidth to 50 MBps in port `80` for the TCP and UDP protocols.

# Working with migrations from physical Oracle Solaris 10 hosts to Oracle Solaris 11 Zones

Two common questions arise when considering how to deploy Oracle Solaris 11\. First, what can we do with the previous Oracle Solaris 10 installation? Second (and worse), what is possible with Oracle Solaris 10 Zones?

Happily, Oracle Solaris 11 provides an optimal solution for both cases: the **physical to virtual** (**P2V**) migration where a physical Oracle Solaris 10 installation is migrated to Oracle Solaris 11 Zone and the **virtual to virtual** (**V2V**) migration where an Oracle Solaris 10 native zone is migrated to a Solaris 10 branded zone on Oracle Solaris 11.

## Getting ready

This recipe requires one virtual machine (VirtualBox or VMware) with Oracle Solaris 11 installed, 8 GB RAM, and enough free space on disk (about 10 GB). To make things easier, the pool myzone (from the previous recipe) will be used, and if you have deleted it, you should create it again using the `zpool create myzone <disks>` command. Furthermore, there must be an Oracle Solaris 10 virtual machine (2 GB RAM and a virtual disk with 15 GB at least) that should be used in this migration example. The installation of this Oracle Solaris 10 virtual machine will not be shown here. The Oracle Solaris 10 DVD for its installation and deployment can be downloaded from [http://www.oracle.com/technetwork/server-storage/solaris10/downloads/index.html?ssSourceSiteId=ocomau](http://www.oracle.com/technetwork/server-storage/solaris10/downloads/index.html?ssSourceSiteId=ocomau).

Our task is to migrate a physical (global zone) Oracle Solaris 10 host (without any non-global zones inside) to an Oracle Solaris 11 zone. The steps to migrate an Oracle Solaris 10 native zone to an Oracle Solaris 11 brand10 zone are very similar, and they will not be shown.

## How to do it…

To migrate a physical Oracle Solaris 10 (global zone) to Oracle Solaris 11 Solaris 10 branded zone, it's advisable to collect any information (the hostname, host ID, amount of memory, operating system version, available disks, and so on) about Oracle Solaris 10 before executing the migration steps. From now, every time we see the `bash-3.2#` prompt, it will mean that we are working on Oracle Solaris 10\. The information can be collected by executing the following simple commands:

```
# bash
bash-3.2# uname -a
SunOS solaris10 5.10 Generic_147148-26 i86pc i386 i86pc
bash-3.2# hostname
solaris10
bash-3.2# ping 192.168.1.1
192.168.1.1 is alive
bash-3.2# hostid
37e12f92
bash-3.2# prtconf | grep -i memory
Memory size: 2048 Megabytes
bash-3.2# more /etc/release
                    Oracle Solaris 10 1/13 s10x_u11wos_24a X86
  Copyright (c) 1983, 2013, Oracle and/or its affiliates. All rights reserved.
                            Assembled 17 January 2013
bash-3.2# ifconfig -a
lo0: flags=2001000849<UP,LOOPBACK,RUNNING,MULTICAST,IPv4,VIRTUAL> mtu 8232 index 1
        inet 127.0.0.1 netmask ff000000
e1000g0: flags=1004843<UP,BROADCAST,RUNNING,MULTICAST,DHCP,IPv4> mtu 1500 index 2
        inet 192.168.1.108 netmask ffffff00 broadcast 192.168.1.255
        ether 8:0:27:49:c4:39
bash-3.2#
bash-3.2# zpool list
no pools available
bash-3.2# df -h
Filesystem             size   used  avail capacity  Mounted on
/dev/dsk/c0t0d0s0       37G   4.2G    33G    12%    /
/devices                 0K     0K     0K     0%    /devices
ctfs                     0K     0K     0K     0%    /system/contract
proc                     0K     0K     0K     0%    /proc
mnttab                   0K     0K     0K     0%    /etc/mnttab
swap                   3.1G   992K   3.1G     1%    /etc/svc/volatile
objfs                    0K     0K     0K     0%    /system/object
sharefs                  0K     0K     0K     0%    /etc/dfs/sharetab
/usr/lib/libc/libc_hwcap1.so.1
                        37G   4.2G    33G    12%    /lib/libc.so.1
fd                       0K     0K     0K     0%    /dev/fd
swap                   3.1G    72K   3.1G     1%    /tmp
swap                   3.1G    32K   3.1G     1%    /var/run
bash-3.2# format
Searching for disks...done

AVAILABLE DISK SELECTIONS:
       0\. c0t0d0 <ATA    -VBOX HARDDISK  -1.0  cyl 5218 alt 2 hd 255 sec 63>
          /pci@0,0/pci8086,2829@d/disk@0,0
Specify disk (enter its number): ^D
bash-3.2#
```

Now that we have already collected all the necessary information from the Oracle Solaris 10 virtual machine, the `zonep2vchk` command is executed to verify the P2V migration compatibility and whether this procedure is possible:

```
bash-3.2# zonep2vchk -b
--Executing Version: 5.10.1.1

  - Source System: solaris10
      Solaris Version: Oracle Solaris 10 1/13 s10x_u11wos_24a X86
      Solaris Kernel:  5.10 Generic_147148-26
      Platform:        i86pc i86pc

  - Target System:
      Solaris Version: Solaris 10
      Zone Brand:      native (default)
      IP type:         shared

--Executing basic checks

  - The following SMF services will not work in a zone:

        svc:/network/iscsi/initiator:default
        svc:/system/iscsitgt:default

  - The following SMF services require ip-type "exclusive" to work in
    a zone. If they are needed to support communication after migrating
    to a shared-IP zone, configure them in the destination system's global
    zone instead:

        svc:/network/ipsec/ipsecalgs:default
        svc:/network/ipsec/policy:default
        svc:/network/routing-setup:default

  - When migrating to an exclusive-IP zone, the target system must have an
    available physical interface for each of the following source system
    interfaces:

        e1000g0

  - When migrating to an exclusive-IP zone, interface name changes may
    impact the following configuration files:

        /etc/hostname.e1000g0
        /etc/dhcp.e1000g0

  - Dynamically assigned IP addresses are configured on the following
    interfaces. These addresses are not supported with shared-IP zones.
    Use an exclusive-IP zone or replace any dynamically assigned addresses
    with statically assigned addresses. These IP addresses could change
    as a result of MAC address changes. You may need to modify this
    system's address information on the DHCP server and on the DNS,
    LDAP, or NIS name servers:

        DHCP assigned address on: e1000g0

  Basic checks complete. Issue(s) detected: 9

--Total issue(s) detected: 9
```

There are no critical issues (it is recommended that you examine this report line by line) so we are able to proceed with the migration in order to create a zone configuration file by executing the following sequence of commands:

```
bash-3.2# mkdir /migration
bash-3.2# zonep2vchk -c > /migration/solaris10.cfg
bash-3.2# vi /migration/solaris10.cfg
bash-3.2# more /migration/solaris10.cfg
create -b
set zonepath=/zones/solaris10
add attr
        set name="zonep2vchk-info"
        set type=string
        set value="p2v of host solaris10"
        end
set ip-type=shared
# Uncomment the following to retain original host hostid:
# set hostid=37e12f92
# maximum lwps based on max_uproc/v_proc
set max-lwps=57140
add attr
        set name=num-cpus
        set type=string
        set value="original system had 1 cpus"
        end
# Only one of dedicated or capped CPU can be used.
# Uncomment the following to use capped CPU:
# add capped-cpu
#       set ncpus=1.0
#       end
# Uncomment the following to use dedicated CPU:
# add dedicated-cpu
#       set ncpus=1
#       end
# Uncomment the following to use memory caps.
# Values based on physical memory plus swap devices:
# add capped-memory
#       set physical=2048M
#       set swap=6142M
#       end
# Original configuration for interface: e1000g0:
#    Statically defined ip address: 192.168.1.108 (solaris10)
#  * DHCP assigned ip address: 192.168.1.108/24 (solaris10)
#    MAC address: Factory assigned: 8:0:27:49:c4:39
#    Unable to migrate addresses marked with "*".
#    Shared IP zones require statically assigned addresses.
add net
        set address=solaris10
        set physical=change-me
        end
exit
bash-3.2#
```

From this previous file, some changes were made as shown in the following command lines (in bold and self-explanatory). The new migrating configuration file looks like the following output:

```
bash-3.2# vi /migration/solaris10.cfg

#create -b
create -t SYSsolaris10

#set zonepath=/zones/solaris10
set zonepath=/myzones/solaris10
add attr
        set name="zonep2vchk-info"
        set type=string
        set value="p2v of host solaris10"
        end
set ip-type=shared
remove anet
# Uncomment the following to retain original host hostid:
set hostid=37e12f92
# maximum lwps based on max_uproc/v_proc
set max-lwps=57140
add attr
        set name=num-cpus
        set type=string
        set value="original system had 1 cpus"
        end
# Only one of dedicated or capped CPU can be used.
# Uncomment the following to use capped CPU:
# add capped-cpu
#       set ncpus=1.0
#       end
# Uncomment the following to use dedicated CPU:
# add dedicated-cpu
#       set ncpus=1
#       end
# Uncomment the following to use memory caps.
# Values based on physical memory plus swap devices:
# add capped-memory
#       set physical=2048M
#       set swap=1024M
#       end
# Original configuration for interface: e1000g0:
#    Statically defined ip address: 192.168.1.108 (solaris10)
#  * DHCP assigned ip address: 192.168.1.108/24 (solaris10)
#    MAC address: Factory assigned: 8:0:27:49:c4:39
#    Unable to migrate addresses marked with "*".
#    Shared IP zones require statically assigned addresses.
add net
        set address=192.168.1.124
        set physical=net0
        end
exit
```

Before continuing the procedure, we have to verify that there is only a global zone (our initial purpose is to migrate an Oracle Solaris 10 host without containing inside zones) by running the following command:

```
bash-3.2# zoneadm list -iv
ID NAME           STATUS     PATH                     BRAND    IP
 0 global         running    /                        native   shared
```

This is great! Now, it is time to create an image (`solaris10.flar`) from the original Oracle Solaris 10 global zone, excluding the directory where the image will be saved (`-x /migration`) in order to prevent a recursion effect by executing the following command:

```
bash-3.2# flarcreate -S -n solaris10 -x /migration /migration/solaris10.flar
Full Flash
Checking integrity...
Integrity OK.
Running precreation scripts...
Precreation scripts done.
Creating the archive...
8417435 blocks
Archive creation complete.
Running postcreation scripts...
Postcreation scripts done.

Running pre-exit scripts...
Pre-exit scripts done.
```

After some time, check the created file by running the following command:

```
bash-3.2# ls -lh /migration/solaris10.flar
-rw-r--r--   1 root     root        4.0G Feb 11 17:32 /migration/solaris10.flar
```

This FLAR image will be used in the following steps from the Oracle Solaris 11 machine, and it is important to share its directory by running the following commands:

```
bash-3.2# share /migration
bash-3.2# share
-               /migration   rw   ""
```

Switching to another machine (`solaris11-1`), which is running Oracle Solaris 11, it is necessary to create a ZFS filesystem to migrate the Oracle Solaris 10 installation into this filesystem as a non-global zone. Therefore, execute the following commands:

```
root@solaris11-1:~# zfs create myzones/solaris10
root@solaris11-1:~# zfs list myzones/solaris10
NAME               USED  AVAIL  REFER  MOUNTPOINT
myzones/solaris10   31K  77.4G    31K  /myzones/solaris10
```

As the `solaris10.flar` image is going to be accessed in order to transfer the Oracle Solaris 10 content from the Oracle Solaris 10 physical host, the connection to the NFS share (`/migration`) from the Oracle Solaris 11 host (`solaris11-1`) has to be verified by running the following command:

```
root@solaris11-1:~# showmount -e 192.168.1.108
export list for 192.168.1.108:
/migration (everyone)

root@solaris11-1:~#
```

It is time to execute the migration steps. Mount the NFS share in `/mnt` by running the following commands:

```
root@solaris11-1:~# mount -F nfs 192.168.1.108:/migration /mnt
root@solaris11-1:~# df -h | grep migration
192.168.1.108:/migration    37G   8.2G        29G    23%    /mnt
```

Create the non-global zone in the Oracle Solaris 11 host (`solaris11-1`) using the saved Solaris 10 configuration file (`solaris10.cfg`) created in a previous step by running the following command:

```
root@solaris11-1:~# zonecfg -z solaris10 -f /mnt/solaris10.cfg 
root@solaris11-1:~# zonecfg -z solaris10 info
zonename: solaris10
zonepath: /myzones/solaris10
brand: solaris10
autoboot: false
bootargs: 
pool: 
limitpriv: 
scheduling-class: 
ip-type: shared
hostid: 37e12f92
fs-allowed: 
[max-lwps: 57140]
net:
  address: 192.168.1.124
  allowed-address not specified
  configure-allowed-address: true
  physical: net0
  defrouter not specified
attr:
  name: zonep2vchk-info
  type: string
  value: "p2v of host solaris10"
attr:
  name: num-cpus
  type: string
  value: "original system had 1 cpus"
rctl:
  name: zone.max-lwps
  value: (priv=privileged,limit=57140,action=deny)
```

Finally, we install the zone using the `solaris10.flar` image by running the following command:

```
root@solaris11-1:~# zoneadm -z solaris10 install -a /mnt/solaris10.flar -u
/myzones/solaris10 must not be group readable.
/myzones/solaris10 must not be group executable.
/myzones/solaris10 must not be world readable.
/myzones/solaris10 must not be world executable.
changing zonepath permissions to 0700.
Progress being logged to /var/log/zones/zoneadm.20140212T033711Z.solaris10.install
    Installing: This may take several minutes...
Postprocessing: This may take a while...
   Postprocess: Updating the image to run within a zone

        Result: Installation completed successfully.
Log saved in non-global zone as /myzones/solaris10/root/var/log/zones/zoneadm.20140212T033711Z.solaris10.install
```

After the previous step, it is recommended that you verify whether the `solaris10` zone is installed and configured correctly by executing the following command:

```
root@solaris11-1:~# zoneadm list -cv
ID NAME        STATUS     PATH                      BRAND     IP    
 0 global      running    /                         solaris   shared
 1 zone1       running    /myzones/zone1            solaris   excl  
 2 zone2       running    /myzones/zone2            solaris   excl  
 - solaris10   installed  /myzones/solaris10        solaris10 shared
root@solaris11-1:~# zoneadm -z solaris10 boot
zone 'solaris10': WARNING: net0: no matching subnet found in netmasks(4): 192.168.1.124; using default of 255.255.255.0.
zone 'solaris10': Warning: "/usr/lib/netsvc/rstat/rpc.rstatd" is not installed in the global zone
```

After booting the zone, check its status again by running the following command:

```
root@solaris11-1:~# zoneadm list -cv
 ID NAME         STATUS     PATH                     BRAND    IP    
 0 global        running    /                        solaris  shared
 1 zone1         running    /myzones/zone1           solaris  excl  
 2 zone2         running    /myzones/zone2           solaris  excl  
 4 solaris10     running    /myzones/solaris10       solaris10 shared
```

Log in to the new zone and verify that it is an Oracle Solaris 10 installation, as follows:

```
root@solaris11-1:~# zlogin solaris10
[Connected to zone 'solaris10' pts/2]
Last login: Tue Feb 11 16:04:11 on console
Oracle Corporation  SunOS 5.10  Generic Patch  January 2005

# bash
bash-3.2# uname -a
SunOS solaris10 5.10 Generic_Virtual i86pc i386 i86pc

bash-3.2# more /etc/release
                    Oracle Solaris 10 1/13 s10x_u11wos_24a X86
  Copyright (c) 1983, 2013, Oracle and/or its affiliates. All rights reserved.
                            Assembled 17 January 2013

bash-3.2# ping 192.168.1.1
192.168.1.1 is alive
bash-3.2#
```

This is amazing! We have migrated the Oracle Solaris 10 host to a solaris10 branded zone in the Oracle Solaris 11 host.

### An overview of the recipe

Using no extra or external tools, we've learned how to migrate an Oracle Solaris 10 physical host to a Oracle Solaris 11 non-global zone using the `zonep2vchk`, `flarcreate`, and `zonecfg` commands.

# References

*   *Oracle Solaris SDN and* *Network Virtualization* at [http://www.oracle.com/technetwork/server-storage/solaris11/technologies/networkvirtualization-312278.html](http://www.oracle.com/technetwork/server-storage/solaris11/technologies/networkvirtualization-312278.html)
*   *Oracle Solaris 11.1 Administration: Oracle Solaris Zones, Oracle Solaris 10 Zones, and Resource Management* ([http://docs.oracle.com/cd/E26502_01/html/E29024/toc.html](http://docs.oracle.com/cd/E26502_01/html/E29024/toc.html)) at [http://docs.oracle.com/cd/E26502_01/html/E29024/z.conf.start-2.html#scrolltoc](http://docs.oracle.com/cd/E26502_01/html/E29024/z.conf.start-2.html#scrolltoc)
*   *Using Virtual Networks in Oracle Solaris 11.1* ([http://docs.oracle.com/cd/E26502_01/html/E28992/toc.html](http://docs.oracle.com/cd/E26502_01/html/E28992/toc.html)) at [http://docs.oracle.com/cd/E26502_01/html/E28992/gdyss.html#scrolltoc](http://docs.oracle.com/cd/E26502_01/html/E28992/gdyss.html#scrolltoc)