# Chapter 1. Working with KVM Guests

In this chapter, we will cover the following recipes:

*   Installing and configuring a KVM
*   Configuring resources
*   Building VMs
*   Adding CPUs on the fly
*   Adding RAM on the fly
*   Adding disks on the fly
*   Moving disks to another storage
*   Moving VMs
*   Backing up your VM metadata

# Introduction

This book will attempt to show you how to deploy RHEL 7 systems without too much of a hassle. As this book is written with automation in mind, I will emphasize on command-line utilities rather than elaborating on its GUI counterparts, which are useless for automation.

This chapter explains how to build and manage KVM guests using the libvirt interface and various tools built around it. It will provide a brief overview on how to set up a KVM on RHEL and manage its resources. The setup provided in this overview is far from the ready enterprise as it doesn't provide any redundancy, which is generally required in enterprises. However, the recipes provided are relevant in enterprise setups as the interface stays the same. Most of the time, you will probably use a management layer (such as RHEV or oVirt), which will make your life easier in managing redundancy.

### Note

Libvirt is the API between the user and the various virtualization and container layers that are available, such as KVM, VMware, Hyper-V, and Linux Containers. Check [https://libvirt.org/drivers.html](https://libvirt.org/drivers.html) for a complete list of supported hypervisors and container solutions.

As most tasks performed need to be automated in the end, I tend not to use any graphical interfaces as these do not allow an easy conversion into script. Hence, you will not find any recipes in this chapter involving a graphical interface. These recipes will primarily focus on `virsh`, the libvirt management user interface that is used to manage various aspects of your KVM host and guests. While a lot of people rely on the edit option of `virsh`, it doesn't allow you to edit a guest's configuration in real time. Editing your guest's XML configuration in this way will require you to shut down and boot your guest for the changes to take effect. A reboot of your guest doesn't do the trick as the XML configuration needs to be completely reread by the guest's instance in order for it to apply the changes. Only a fresh boot of the guest will do this.

The `virsh` interface is also a shell, so by launching `virsh` without any commands, you will enter the libvirt management shell. A very interesting command is `help`. This will output all the available commands grouped by keyword. Each command accepts the `--help` argument to show a detailed list of the possible arguments, and their explanation, which you can use.

# Installing and configuring a KVM

This recipe covers the installing of virtualization tools and packages on RHEL 7.

By default, a RHEL 7 system doesn't come with a KVM or libvirt preinstalled. This can be installed in three ways:

*   Through the graphical setup during the system's setup
*   Via a kickstart installation
*   Through a manual installation from the command line

For this recipe, you should know how to install packages using yum, and your system should be configured to have access to the default RHEL 7 repository (refer to [Chapter 8](part0066_split_000.html#1UU541-501a83dd54944cb1bf060a2ce9fab11f "Chapter 8. Yum and Repositories"), *Yum and Repositories*, for more information), which is required for the packages that we will use.

Alternatively, you could install packages from the installation media using `rpm`, but you'll need to figure out the dependencies yourself.

Check the dependencies of an `rpm` using the following command:

```
~]# rpm -qpR <rpm file>

```

This will output a list of binaries, libraries, and files that you need installed prior to installing this package.

Check which package contains these files through this command:

```
~]# rpm -qlp <rpm package>

```

As you can imagine, this is a tedious job and can take quite some time as you need to figure out every dependency for every package that you want to install in this way.

## Getting ready

To install a KVM, you will require at least 6 GB of free disk space, 2 GB of RAM, and an additional core or thread per guest.

Check whether your CPU supports a virtualization flag (such as SVM or VMX). Some hardware vendors disable this in the BIOS, so you may want to check your BIOS as well. Run the following command:

```
~]# grep -E 'svm|vmx' /proc/cpuinfo
flags    : ... vmx ...

```

Alternatively, you can run the following command:

```
~]# grep -E 'svm|vmx' /proc/cpuinfo
flags    : ... svm ...

```

Check whether the hardware virtualization modules (such as `kvm_intel` and `kvm`) are loaded in the kernel using the following command:

```
~]# lsmod | grep kvm
kvm_intel             155648  0
kvm                   495616  1 kvm_intel

```

## How to do it…

We'll look at the three ways of installing a KVM onto your system.

### Manual installation

This way of installing a KVM is generally done once the base system is installed by some other means. You need to perform the following steps:

1.  Install the software needed to provide an environment to host virtualized guests with the following command:

    ```
    ~]# yum -y install qemu-kvm qemu-img libvirt

    ```

    The installation of these packages will include quite a lot of dependencies.

2.  Install additional utilities required to configure `libvirt` and install virtual machines by running this command:

    ```
    ~]# yum -y install virt-install libvirt-python python-virthost libvirt-client

    ```

3.  By default, the `libvirt` daemon is marked to `autostart` on each boot. Check whether it is enabled by executing the following command:

    ```
    ~]# systemctl status libvirtd
    libvirtd.service - Virtualization daemon
     Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled)
     Active: inactive
     Docs: man:libvirtd(8)
     http://libvirt.org

    ```

4.  If for some reason this is not the case, mark it for autostart by executing the following:

    ```
    ~]# systemctl enable libvirtd

    ```

5.  To manually stop/start/restart the `libvirt` daemon, this is what you'll need to execute:

    ```
    ~]# systemctl stop libvirtd
    ~]# systemctl start libvirtd
    ~]# systemctl restart libvirtd

    ```

### Kickstart installation

Installing a KVM during kickstart offers you an easy way to automate the installation of KVM instances. Perform the following steps:

1.  Add the following package groups to your kickstarted file in the `%packages` section:

    ```
    @virtualization-hypervisor
    @virtualization-client
    @virtualization-platform
    @virtualization-tools
    ```

2.  Start the installation of your host with this kickstart file.

### Graphical setup during the system's setup

This is probably the least common way of installing a KVM. The only time I used this was during the course of writing this recipe. Here's how you can do this:

1.  Boot from the RHEL 7 Installation media.
2.  Complete all steps besides the **Software selection** step.![Graphical setup during the system's setup](img/00002.jpeg)
3.  Go to **Software Selection** to complete the KVM software selection.
4.  Select the **Virtualization host** radio button in **Base Environment**, and check the **Virtualization Platform** checkbox in **Add-Ons for Selected Environment**:![Graphical setup during the system's setup](img/00003.jpeg)
5.  Finalize the installation.
6.  On the **Installation Summary** screen, complete any other steps and click on **Begin Installation**.

## See also

To set up your repositories, check out [Chapter 8](part0066_split_000.html#1UU541-501a83dd54944cb1bf060a2ce9fab11f "Chapter 8. Yum and Repositories"), *Yum and Repositories*.

To deploy a system using kickstart, refer to [Chapter 2](part0025_split_000.html#NQU21-501a83dd54944cb1bf060a2ce9fab11f "Chapter 2. Deploying RHEL "En Masse""), *Deploying RHEL "En Masse"*.

For more in-depth information about using libvirt, go to [http://www.libvirt.org/](http://www.libvirt.org/).

RHEL 7 has certain support limits, which are listed at these locations:

[https://access.redhat.com/articles/rhel-kvm-limits](https://access.redhat.com/articles/rhel-kvm-limits)

[https://access.redhat.com/articles/rhel-limits](https://access.redhat.com/articles/rhel-limits)

# Configuring resources

Virtual machines require CPUs, memory, storage, and network access, similar to physical machines. This recipe will show you how to set up a basic KVM environment for easy resource management through libvirt.

A storage pool is a virtual container limited by two factors:

*   The maximum size allowed by `qemu-kvm`
*   The size of the disk on the physical machine

Storage pools may not exceed the size of the disk on the host. The maximum sizes are as follows:

*   virtio-blk = 2^63 bytes or 8 exabytes (raw files or disk)
*   EXT4 = ~ 16 TB (using 4 KB block size)
*   XFS = ~8 exabytes

## Getting ready

For this recipe, you will need a volume of at least 2 GB mounted on `/vm` and access to an NFS server and export.

We'll use `NetworkManager` to create a bridge, so ensure that you don't disable `NetworkManager` and have `bridge-utils` installed.

## How to do it…

Let's have a look into managing storage pools and networks.

### Creating storage pools

In order to create storage pools, we need to provide the necessary details to the KVM for it to be able to create it. You can do this as follows:

1.  Create a `localfs` storage pool using `virsh` on `/vm`, as follows:

    ```
    ~]# virsh pool-define-as --name localfs-vm --type 
    dir --target /vm

    ```

2.  Create the target for the storage pool through the following command:

    ```
    ~# mkdir -p /nfs/vm

    ```

3.  Create an NFS storage pool using `virsh` on NFS server:`/export/vm`, as follows:

    ```
    ~]# virsh pool-define-as --name nfs-vm --type network --source-host nfsserver --source-path /export/vm –target /nfs/vm

    ```

4.  Make the storage pools persistent across reboots through the following commands:

    ```
    ~]# virsh pool-autostart localfs-vm
    ~]# virsh pool-autostart nfs-vm

    ```

5.  Start the storage pool, as follows:

    ```
    ~]# virsh pool-start localfs-vm
    ~]# virsh pool-start nfs-vm

    ```

6.  Verify that the storage pools are created, started, and persistent across reboots. Run the following for this:

    ```
    ~]# virsh pool-list
     Name                 State      Autostart
    -------------------------------------------
     localfs-vm           active     yes
     nfs-vm               active     yes

    ```

### Querying storage pools

At some point in time, you will need to know how much space you have left in your storage pool.

Get the information of the storage pool by executing the following:

```
~]# virsh pool-info --pool <pool name>
Name:           nfs-vm
UUID:           some UUID
State:          running
Persistent:     yes
Autostart:      yes
Capacity:       499.99 GiB
Allocation:     307.33 GiB
Available:      192.66 GiB

```

As you can see, this command easily shows you its disk space allocation and availability.

### Tip

Be careful though; if you use a filesystem that supports sparse files, these numbers will most likely be incorrect. You will have to manually calculate the sizes yourself!

To detect whether a file is sparse, run `ls -lhs` against the file. The `-s` command will show an additional column (the first), showing the exact space that the file is occupying, as follows:

```
~]# ls -lhs myfile
121M -rw-------. 1 root root  30G Jun 10 10:27 myfile

```

### Removing storage pools

Sometimes, storage is phased out. So, it needs to be removed from the host.

You have to ensure that no guest is using volumes on the storage pool before proceeding, and you need to remove all the remaining volumes from the storage pool. Here's how to do this:

1.  Remove the storage volume, as follows:

    ```
    ~]# virsh vol-delete --pool <pool name> --vol <volume name>

    ```

2.  Stop the storage pool through the following command:

    ```
    ~]# virsh pool-destroy --pool <pool name>

    ```

3.  Delete the storage pool using the following command:

    ```
    ~]# virsh pool-delete --pool <pool name>

    ```

### Creating a virtual network

Before creating the virtual networks, we need to build a bridge over our existing network interface. For the sake of convenience, this NIC will be called `eth0`. Ensure that you record your current network configuration as we'll destroy it and recreate it on the bridge.

Unlike the storage pool, we need to create an XML configuration file to define the networks. There is no command similar to `pool-create-as` for networks. Perform the following steps:

1.  Create a bridge interface on your network's interface, as follows:

    ```
    ~]# nmcli connection add type bridge autoconnect yes con-name bridge-eth0 ifname bridge-eth0

    ```

2.  Remove your NIC's configuration using the following command:

    ```
    ~]# nmcli connection delete eth0

    ```

3.  Configure your bridge, as follows:

    ```
    ~]# nmcli connection modify bridge-eth0 ipv4.addresses <ip address/cidr> ipv4.method manual
    ~# nmcli connection modify bridge-eth0 ipv4.gateway <gateway ip address>
    ~]# nmcli connection modify bridge-eth0 ipv4.dns <dns servers>

    ```

4.  Finally, add your NIC to the bridge by executing the following:

    ```
    ~]# nmcli connection add type bridge-slave autoconnect yes con-name slave-eth0 ifname eth0 master bridge-eth0

    ```

For starters, we'll take a look at how we can create a NATed network similar to the one that is configured by default and called the default:

1.  Create the network XML configuration file, `/tmp/net-nat.xml`, as follows:

    ```
    <network>
      <name>NATted</name>
      <forward mode='nat'>
        <nat>
          <port start='1024' end='65535'/>
        </nat>
      </forward>
      <bridge name='virbr0' stp='on' delay='0'/>
      <ip address='192.168.0.1' netmask='255.255.255.0'>
        <dhcp>
          <range start='192.168.0.2' end='192.168.0.254'/>
        </dhcp>
      </ip>
    </network>
    ```

2.  Define the network in the KVM using the preceding XML configuration file. Execute the following command:

    ```
    ~]# virsh net-define /tmp/net-nat.xml

    ```

Now, let's create a bridged network that can use the network bound to this bridge through the following steps:

1.  Create the network XML configuration file, `/tmp/net-bridge-eth0.xml`, by running the following:

    ```
    <network>
        <name>bridge-eth0</name>
        <forward mode="bridge" />
        <bridge name="bridge-eth0" />
    </network>
    ```

2.  Create the network in the KVM using the preceding file, as follows:

    ```
    ~]# virsh net-define /tmp/net-bridge-eth0.xml

    ```

There's one more type of network that is worth mentioning: the isolated network. This network is only accessible to guests defined in this network as there is no connection to the "real" world.

1.  Create the network XML configuration file, `/tmp/net-local.xml`, by using the following code:

    ```
    <network>
      <name>isolated</name>
      <bridge name='virbr1' stp='on' delay='0'/>
      <domain name='isolated'/>
    </network>
    ```

2.  Create the network in KVM by using the above file:

    ```
    ~]# virsh net-define /tmp/net-local.xml

    ```

Creating networks in this way will register them with the KVM but will not activate them or make them persistent through reboots. So, this is an additional step that you need to perform for each network. Now, perform the following steps:

1.  Make the network persistent across reboots using the following command:

    ```
    ~]# virsh net-autostart <network name>

    ```

2.  Activate the network, as follows:

    ```
    ~]# virsh net-start <network name>

    ```

3.  Verify the existence of the KVM network by executing the following:

    ```
    ~]# virsh net-list --all
     Name                 State      Autostart     Persistent
    ----------------------------------------------------------
     bridge-eth0          active     yes           yes
     default              inactive   no            yes
     isolated             active     yes           yes
     NATted               active     yes           yes

    ```

### Removing networks

On some occasions, the networks are phased out; in this case, we need to remove the network from our setup.

Prior to executing this, you need to ensure that no guest is using the network that you want to remove. Perform the following steps to remove the networks:

1.  Stop the network with the following command:

    ```
    ~# virsh net-destroy --network <network name>

    ```

2.  Then, delete the network using this command:

    ```
    ~]# virsh net-undefine --network <network name>

    ```

## How it works…

It's easy to create multiple storage pools using the define-pool-as command, as you can see. Every type of storage pool needs more, or fewer, arguments. In the case of the NFS storage pool, we need to specify the NFS server and export. This is done by specifying--source-host and--source-path respectively.

Creating networks is a bit more complex as it requires you to create a XML configuration file. When you want a network connected transparently to your physical networks, you can only use bridged networks as it is impossible to bind a network straight to your network's interface.

## There's more…

The storage backend created in this recipe is not the limit. Libvirt also supports the following backend pools:

### Local storage pools

Local storage pools are directly connected to the physical machine. They include local directories, disks, partitions, and LVM volume groups. Local storage pools are not suitable for enterprises as these do not support live migration.

### Networked or shared storage pools

Network storage pools include storage shared through standard protocols over a network. This is required when we migrate virtual machines between physical hosts. The supported network storage protocols are Fibre Channel-based LUNs, iSCSI, NFS, GFS2, and SCSI RDMA.

By defining the storage pools and networks in libvirt, you ensure the availability of the resources for your guest. If, for some reason, the resource is unavailable, the KVM will not attempt to start the guests that use these resources.

When checking out the man page for *virsh (1)*, you will find a similar command to `net-define`, `pool-define`: `net-create`, and `pool-create` (and `pool-create-as`). The `net-create` command, similar to `pool-create` and `pool-create-as`, creates transient (or temporary) resources, which will be gone when libvirt is restarted. On the other hand, `net-define` and `pool-define` (as also `pool-define-as`) create persistent (or permanent) resources, which will still be there after you restart libvirt.

## See also

You can find out more on libvirt storage backend pools at [https://libvirt.org/storage.html](https://libvirt.org/storage.html)

More information on libvirt networking can be found at [http://wiki.libvirt.org/page/Networking](http://wiki.libvirt.org/page/Networking)

# Building guests

After you install and configure a KVM on the host system, you can create guest operating systems. Every guest is defined by a set of resources and parameters stored in the XML format. When you want to create a new guest, creating such an XML file is quite cumbersome. There are two ways to create a guest:

*   Using `virt-manager`
*   Using `virt-install`

This recipe will employ the latter as it is perfect for scripting, while `virt-manager` is a GUI and not very well suited to automate things.

## Getting ready

In this recipe, we will cover a generic approach to create a new virtual machine using the `bridge-eth0` network bridge and create a virtual disk on the `localfs-vm` storage pool, which is formatted as QCOW2\. The QCOW2 format is a popular virtual disk format as it allows thin provisioning and snapshotting. We will boot the RHEL 7 installation media located on the `localfs-iso` storage pool (`rhel7-install.iso`) to start installing a new RHEL 7 system.

## How to do it…

Let's create some guests and delete them.

### Create a guest

Let's first create a disk for the guest and then create the guest on this disk, as follows:

1.  Create a 10 GB QCOW2 format disk in the `localfs-vm` pool, as follows:

    ```
    ~]# virsh vol-create-as --pool localfs-vm --name rhel7_guest-vda.qcows2 --format qcows2 –capacity 10G

    ```

2.  Create the virtual machine and start it through the following command:

    ```
    ~]# virt-install \
    --hvm \
    --name rhel7_guest \
    –-memory=2048,maxmemory=4096 \
    --vcpus=2,maxvcpus=4 \
    --os-type linux \
    --os-variant rhel7 \
    --boot hd,cdrom,network,menu=on \
    --controller type=scsi,model=virtio-scsi \
    --disk device=cdrom,vol=localfs-iso/rhel7-install.iso,readonly=on,bus=scsi \
    --disk device=disk,vol=localfs-vm/rhel7_guest-vda.qcow2,cache=none,bus=scsi \
    --network network=bridge-eth0,model=virtio \
    --graphics vnc \
    --graphics spice \
    --noautoconsole \
    --memballoon virtio

    ```

### Deleting a guest

At some point, you'll need to remove the guests. You can do this as follows:

1.  First, ensure that the guest is down by running the following:

    ```
    ~]# virsh list –all
     Id    Name                           State
    ----------------------------------------------------
    -     rhel7_guest                     shut off

    ```

    If the state is not `shut off`, you can forcefully shut it down:

    ```
    ~]# virsh destroy --domain <guest name>

    ```

2.  List the storage volumes in use by your guest and copy this somewhere:

    ```
    ~]# virsh domblklist <guest name>
    Type       Device     Target     Source
    ------------------------------------------------
    file       disk       vda        /vm/rhel7_guest-vda.qcow2
    file       cdrom      hda        /iso/rhel7-install.iso

    ```

3.  Delete the guest through the following command:

    ```
    ~]# virsh undefine --domain <guest name> --storage vda

    ```

    Adding `--remove-all-storage` to the command will wipe off the data on the storage volumes dedicated to this guest prior to deleting the volume from the pool.

## How it works…

The `virt-install` command supports creating storage volumes (disks) by specifying the pool, size, and format. However, if this storage volume already exists, the application will fail. Depending on the speed of your KVM host disks (local or network) and the size of the guest's disks, the process of creating a new disk may take some time to be completed. By specifying an existing disk with `virt-install`, you can reuse the disk should you need to reinstall the guest. It would be possible to only create the disk on the first pass and change your command line appropriately after this. However, the fact remains that using `virsh vol-create-as` gives you more granular control of what you want to do.

We're using the QCOW2 format to contain the guest's disk as it is a popular format when it comes to storing KVM guest disks. This is because it supports thin provisioning and snapshotting.

When creating the guest, we specify both the `maxmemory` option for memory configuration and the `maxvcpus` option for vcpus configuration. This will allow us to add CPUs and RAM to the guest while it is running. If we do not assign these, we'll have to shut down the system before being able to change the XML configuration using the following command:

```
~# virsh edit <hostname>

```

As you can see, we're using the `virtio` driver for any hardware (network, disks, or balloon) that supports it as it is native to the KVM and is included in the RHEL 7 kernel.

### Note

If, for some reason, your guest OS doesn't support `virtio` drivers, you should remove the `--controller` option of the command line and the bus specification from the `--disk` option.

For more information on `virtio` support, go to [http://wiki.libvirt.org/page/Virtio](http://wiki.libvirt.org/page/Virtio).

The `--memballoon` option will ensure that we do not run into problems when we overcommit our memory. When specific guests require more memory, the ballooning driver will ensure that the "idle" guests' memory can be evenly redistributed.

The `graphics` option will allow you to connect to the guest through the host using either VNC (which is a popular client to control remote computers) or spice (which is the default client for `virt-manager`). The configuration for both VNC and spice is insecure, though. You can either set this up by specifying a password—by adding `password=<password>` to each graphics stanza—or by editing the `/etc/libvirt/qemu.conf` file on the KVM host, which will be applied to all guests.

## There's more…

In this recipe, we used "local" install media in the form of an ISO image to install the system. However, it is also possible to install a guest without a CD, DVD, or an ISO image. The `--location` installation method option allows you to specify a URI that contains your kernel/initrd pair, which is required to start the installation.

Using `--location` in combination with `--extra-args` will allow you to specify kernel command-line arguments to pass to the installer. This can be used, for instance, to pass on the location of an Anaconda kickstart file for automated installs and/or specifying your IP configuration during the installer.

## See also

Check the man page of *virt-install (1)* for more information on how to use it to your advantage.

# Adding CPUs on the fly

Imagine an enterprise having to correctly add dimension to all their systems right from the start. In my experience, this is very difficult. You will either underdimension it, and your customers will complain about performance at some point, or you will overdimension it, and then the machine will sit there, idling about, which is not optimal either. This is the reason hardware vendors have come up with `hot-add` resources. This allows a system to have its CPUs, memory, and/or disks to be upgraded/increased without the need for a shutdown. A KVM implements a similar functionality for its guests. It allows you to increase the CPUs, memory, and disks on the fly.

The actual recipe is very simple to execute, but there are some prerequisites to be met.

## Getting ready

In order to be able to add CPUs on the fly to a guest, the guest's configuration must support them.

There are two ways to achieve this:

*   It must be created with the max option, as follows:

    ```
    --vcpus 2,maxvcpus=4
    ```

*   You can set the maximum using `virsh` (which will be applied at the next boot) through the following command:

    ```
    ~]# virsh setvcpus --domain <guestname> --count <max cpu count> --config --maximum

    ```

*   You can edit the guests' XML files, as follows:

    ```
    ~]# virsh edit <guestname>

    ```

The last two options will require you to shut down and boot (not reboot) your guest as these commands cannot change the "live" configuration.

The guest's XML file must contain the following element with the subsequent attributes:

```
<domain type='kvm'>
...
<vcpu current='2'>4</vcpu>
...
</domain>
```

Here, `current` indicates the number of CPUs in use, and the number within the node indicates the maximum number of vCPUs that can be assigned. This number can be increased but should never exceed the number of cores or threads in your host.

## How to do it…

Let's add some CPUs to the guest.

### On the KVM host, perform the following steps:

1.  Get the maximum number vCPUs that you can assign, as follows:

    ```
    ~]# virsh dumpxml <guestname> |grep vcpu
    <vcpu placement='static' current='4'>8</vcpu>

    ```

2.  Now, set the new number of vCPUs through this command:

    ```
    ~]# virsh setvcpus --domai
    n <guestname> --count <# of CPUs> --live

    ```

### On the KVM guest, perform the following:

1.  Tell your guest OS there are more CPUs available by executing the following command:

    ```
    ~]# for i in $(grep -H 0 /sys/devices/system/cpu/cpu*/online | awk -F: '{print $1}'); do echo 1 > $i; done

    ```

# Adding RAM on the fly

As with CPUs, the possibility to add memory on the fly is an added value in mission-critical environments where downtime can literally cost a company millions of Euros.

The recipe presented here is quite simple, similar to the one on CPUs. Here, your guest needs to be prepared to use this functionality as well.

## Getting ready

If you want to be able to add memory on the fly to a guest, it must be configured to support it. As with the CPU, this has to be activated. There are three ways to do this:

*   The guest must be created with the `maxmem` option, as follows:

    ```
    --memory 2G,maxmemory=4G
    ```

*   You can set the maximum memory using the `virsh` command, as follows:

    ```
    ~]# virsh setmaxmem --domain <guestname> --size <max mem> --live

    ```

*   You can edit the guests' XML files:

    ```
    ~]# virsh edit <guestname>

    ```

Of course, the latter 2 option requires you to shut down the guest, which is not always possible in production environments.

Ensure that the guests' XML configuration files contain the following elements with the subsequent attributes:

```
<domain type='kvm'>
...
    <memory unit='KiB'>4194304</memory>
    <currentMemory unit='KiB'>2097152</currentMemory>
...
</domain>
```

## How to do it…

Let's increase the guest's memory.

On the KVM host, perform the following steps:

1.  Get the current and maximum memory allocation for a guest, as follows:

    ```
    ~]# virsh dumpxml srv00002 |grep -i memory
     <memory unit='KiB'>4194304</memory>
     <currentMemory unit='KiB'>4194304</currentMemory>

    ```

2.  Set the new amount of memory for the guest by executing the following command:

    ```
    ~]# virsh setmem --domain <guestname> --size <memory> --live

    ```

On the KVM guest, perform the following:

1.  Tell your guest OS about the memory increase through this command:

    ```
    ~]# for i in $(grep -H offline /sys/devices/system/memory/memory*/state | awk -F: '{print $1}'); do echo online > $i; done

    ```

# Adding disks on the fly

This recipe includes instructions on how to create different types of storage volumes. Storage volumes are dedicated storage sets aside for use by guests.

## Getting ready

There is not a lot of preparation to be done in order to add disks to your guest, which is in contrast to adding CPUs and RAM.

You only need to ensure that the storage pool has enough free disk space to accommodate the new disk.

## How to do it…

Similar to the recipe for creating guests, you'll need to create a disk first. This can be done as follows:

1.  Let's create a raw disk in the `localfs-vm` pool that is `30` GB big through the following command:

    ```
    ~]# virsh vol-create-as --pool localfs-vm --name rhel7_guest-vdb.raw --format raw --capacity 30G

    ```

2.  Look up the path of the newly created volume, as follows:

    ```
    ~]# virsh vol-list --pool localfs-vm |awk '$1 ~ /^rhel7_guest-vdb.raw$/ {print $2}'

    ```

    This will result in the path of your volume; here's an example:

    ```
    /vm/rhel7_guest-vdb.raw

    ```

3.  Attach the disk to the guest, as follows:

    ```
    ~]# virsh attach-disk --domain <guestname> --source <the above path> --target vdb --cache none --persistent –live

    ```

## How it works…

Creating a disk using `vol-create-as` may take some time depending on the speed of your host's disks and the size of the guest's disks.

We will look up the path of the newly created volume as it is a required argument for the command that attaches the disk to the guest. In most cases, you won't need to do this as you'll know how your host is configured, but when you script this kind of functionality, you will require this step.

Adding a disk in this way will attach a disk using the `virtio` driver, which, as specified earlier, is optimized for use with KVMs.

## There's more…

If, for some reason, the original guest doesn't support `virtio` drivers or you do not have the `virtio` controller, you can create this yourself. Store the XML configuration file as `/tmp/controller.xml` with the following contents:

```
<controller type='scsi' model='virtio' />
```

You can find this out by checking the host's XML file for the preceding statement.

Then, import the XML configuration file, as follows:

```
~]# virsh attach-device –domain <guestname> /tmp/controller.xml

```

This will allow you to create disks using `virtio`.

# Moving disks to another storage

Moving disks around is part of the life cycle of a guest. Disks in the storage pools (local or network) may fail or fill up due to bad capacity management. Another reason may be the cost or speed of the disks involved. Sooner or later, one of these things will happen, and then you will need to move the storage somewhere else.

Ordinarily, one would have to shut down the guest, copy the storage volume file elsewhere (if it is a file), wait, update the machine's XML configuration, and launch it again. However, in today's mission-critical enterprises, this may not always be possible.

## Getting ready

In order to perform this copy, you need the source and destination paths of the disk. You can get the source path by checking the XML configuration file or, even better, by querying the storage volume itself. This does require you to know which storage pool it is located on.

Execute the following command:

```
~]# virsh vol-list --pool <storage pool> |awk '$1 ~ /^<volume name>$/ {print $2}'

```

Ensure that your destination is an existing storage pool; if not, go ahead and create it.

Check out the *Configuring resources* recipe in this chapter to create storage pools.

If you can't remember the path to your pool's location, run the following:

```
~]# virsh pool-dumpxml <poolname> |awk '/<path>.*<\/path>/ {print $1}'

```

## How to do it…

Moving disks can take some time, so ensure that you have plenty of time available. Perform the following steps:

1.  Dump the inactive XML configuration file for the guest, as follows:

    ```
    ~]# virsh dumpxml --inactive <guestname> > /tmp/<guestname>.xml

    ```

    The `–-inactive` file will ensure that it doesn't copy any temporary information that is irrelevant to the guest.

2.  Undefine the guest through the following command:

    ```
    ~]# virsh undefine <guestname>

    ```

3.  Copy the virtual disk to another location by executing the following:

    ```
    ~]# virsh blockcopy --domain <guestname> --path <original path> --dest <destination path> --wait --verbose –-pivot

    ```

4.  Now, edit the guest's XML configuration file and change the path of the disk to the new location.
5.  Redefine the guest, as follows:

    ```
    ~]# virsh define /tmp/<guestname>.xml

    ```

6.  Remove the source disk after you are happy with the results. Run the following command:

    ```
    ~]# virsh vol-delete --pool <poolname> --vol <volname>

    ```

## How it works…

The moving of disks can only be performed on transient domains, which is the reason we execute the `virsh undefine` command. In order to be able to make it persistent again after the transfer, we also need to dump the XML configuration file and modify the storage volume path.

Moving the disk does two things, which are:

*   Firstly, it copies all the data of the source to the destination
*   Secondly, when the copying is complete, both source and destination remain mirrored until it is either canceled with `blockjob --abort` or actually switched over to the new target by executing the `blockjob --pivot` command

The preceding `blockcopy` command does everything at the same time. The `--wait` command will not give control back to the user until the command fails or succeeds. It is essentially the same as the following:

```
~]# virsh blockcopy --domain <guestname> --path <source path> --dest <destination path>

```

Monitor the progress of the copy by executing the following:

```
~]# watch -n10 "virsh blockjob –domain <guestname> --path <source path> --info"

```

When it's done, execute this:

```
~]# virsh blockjob –domain <guestname> --path <source path> --pivot

```

## There's more…

It is also possible to change the disk format on the fly, by specifying the `--format` argument with the format that you want to convert your disk into. If you want to copy it to a block device, specify `--blockdev`.

# Moving VMs

Moving disks will mitigate the risk of failing disks. When your CPUs, memory, and other non-disk-related components start failing, you have no other option but to move the guests to other host(s).

The recipe for this task is rather simple, but it's the prerequisites that can make it succeed or fail miserably.

## Getting ready

The prerequisites for this recipe are quite extended.

For the host, the following are the requirements:

*   You'll need to have access to shared data. Both the source and destination KVM machine will need to be able to access the same storage—for example, iSCSI, NFS, and so on.
*   Both hosts need the same type of CPU—that is, Intel or AMD (one cannot live migrate a guest from a host with Intel CPUs to a host with AMD CPUs).
*   Both hosts need to be installed with the same version and updates of libvirt.
*   Both hosts need to have the same network ports open.
*   Both hosts must have identical KVM network configurations or at least the same network configurations for the interfaces used by the guest.
*   Both hosts must be accessible through the network.
*   It's a good idea to have a management network set up and connected to the two hosts, which can be used for data transfer. This will cause less network traffic on your "production" network and increase the overall speed.
*   The `No execution` bit must be the same on both hosts.

The requirement for the guest is:

*   The `cache=none` must be specified for all block devices that are opened in write mode.

## How to do it…

There are multiple ways to migrate hosts, but we will only highlight the two most common ways.

### Live native migration over the default network

This process to migrate a host is luckily very simple and can be summarized in one command.

On the source host, execute the following:

```
~]# virsh migrate --domain <guestname> --live –-persistent --undefinesource --verbose --desturl qemu+ssh://<host 2>/system

```

### Live native migration over a dedicated network

It is possible to perform the migration over a dedicated network. By default, this will use the first network it finds that suits it needs. You'll need to specify the listening address (on the host) and the protocol. This requires the same command as before, but we'll need to specify the local listening IP address and protocol, such as TCP.

On the source host, execute the following:

```
~]# virsh migrate --domain <guestname> --live –-persistent --undefinesource --verbose --desturl qemu+ssh://<host 2>/system tcp://<local ip address on dedicated network>/

```

## How it works…

This type of migration is called a "hypervisor native" transport. The biggest advantage of this type of migration is that it incurs the lowest computational cost by minimizing the number of data copies involved.

When we migrate a host, it performs a copy of the memory of the guest to the new host. When the copying is successful, it kills the guest on the source host and starts it on the new host. As the memory is copied, the interruption will be very short-lived.

## There's more…

Communication between the two hosts is over SSH, which is already pretty secure. However, it's also possible to tunnel the data over an even more strongly encrypted channel by specifying the `--tunnelled` option. This will impose more traffic on your network as there will be extra data communication between the two hosts.

The `--compress` option can help you out if you wish to reduce the traffic over your network, but this will increase the load on both your hosts as they need to compress/decompress the data, which, in turn, may impact your guests performance. If time is not of the essence but traffic is, this is a good solution.

## See also

There's very good and in-depth documentation about this process at [https://libvirt.org/migration.html](https://libvirt.org/migration.html).

# Backing up your VM metadata

While a KVM stores some of the resources' configuration on the disk in a human readable format, it is a good idea to query libvirt for the configuration of your resources.

## How to do it…

In this recipe we'll back up all relevant KVM metadata by performing the following steps:

Here's the network configuration:

```
~]# for i in $(virsh net-list --all | sed -e '1,2d' |awk '{print $1}'); do \
 virsh net-dumpxml --network $i --inactive > /tmp/net-$i.xml; \
done

```

Here's the storage configuration:

```
~]# for i in $(virsh pool-list --all | sed -e '1,2d' |awk '{print $1}'); do \
 for j in $(virsh vol-list --pool $i |sed -e '1,2d') | awk '{print $1}'; do \
 virsh vol-dumpxml --pool $i --vol $j > /tmp/vol-$j.xml; \
 done \
 virsh pool-dumpxml --pool $i --inactive > /tmp/pool-$i.xml; \
done

```

Here's the guest configuration:

```
~]# for i in $(virsh list --all | sed -e '1,2d' |awk '{print $1}'); do \
 virsh dumpxml --domain $i --inactive > /tmp/domain-$i.xml; \
done

```

## How it works…

The `virsh net-dumpxml` command allows you to dump the precise configuration of the specified network. In combination with `virsh net-list`, you can create a loop that enumerates all networks and dumps them on the file. By specifying `–-all`, you will export all networks, even those that are not active. If you do not wish to back up the configuration for nonactive networks, substitute `virsh net-list --all` with `virsh net-list`.

Storage pools can be enumerated, similarly to networks, using `virsh net-list`. However, besides the individual storage pool configuration, we are also interested in the configuration of individual storage volumes. Luckily, both implement a `list` and `dumpxml` command! If you're not interested in nonactive pools, you can omit the `--all` option with `virsh pool-list`.

Guests can similarly be enumerated and their XML configuration dumped using `dumpxml`. Again, if you're not interested in nonactive guests, you can omit the `--all` option with `virsh list`.

## See also

The man page for *virsh (1)* lists all the possible options for the commands used in the preceding section.