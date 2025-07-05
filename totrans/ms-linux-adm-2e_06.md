# 6

# Working with Disks and Filesystems

In this chapter, you will learn how to manage disks and filesystems, how to use the **Logical Volume Management** (**LVM**) system, and how to mount and partition the hard drive, as well as gain an understanding of storage in Linux. You will also learn how to partition and format a disk, as well as how to create logical volumes, and you will gain a deeper understanding of filesystem types. In this chapter, we’re going to cover the following main topics:

*   Understanding devices in Linux
*   Understanding filesystem types in Linux
*   Understanding disks and partitions
*   Introducing LVM in Linux

# Technical requirements

A basic knowledge of disks, partitions, and filesystems is preferred. No other special technical requirements are needed, just a working installation of Linux on your system. We will mainly use Ubuntu or Debian for this chapter’s exercises. All the commands used in this chapter can be replicated on any Linux distribution, even if you don’t use Debian or Ubuntu.

# Understanding devices in Linux

As already stated on several occasions in this book, everything in Linux is a file. This also includes devices. **Device files** are special files in Unix and Linux operating systems. Those special files are interfaces to device drivers, and they are present in the filesystem as a regular file.

With no further ado, let’s see how Linux abstraction layers work. This will give you an overview of how hardware and software are related and interconnected.

## Linux abstraction layers

Now is as good a time as any to discuss Linux system abstraction layers and how devices fit into the overall picture. Any computer is generally organized into two layers (or levels) – the hardware and the software levels:

*   **Hardware level**: This level contains the hardware components of your machine, such as the memory (RAM), **central processing unit** (**CPU**), and devices, including disks, network interfaces, ports, and controllers.
*   **Software level**: For all these hardware components to work, the operating system (Linux, in our case) uses **abstraction layers**. Those layers exist in the **kernel**, which is the main software component of Linux. Without diving into more information, it is sufficient for you to know that Linux has these layers that are responsible for accessing low-level resources and providing the specific drivers for different hardware components. When the computer is booted up, the Linux kernel is loaded from the disk into the system’s memory (RAM). Thus, inside the memory, there will be two separate regions, called **kernel space** and **user space**, and this would be the **software level**:
    *   The kernel is the beating heart of the Linux operating system. The kernel resides inside the memory (RAM) and manages all the hardware components. It is the *interface* between the software and hardware on your Linux system.
    *   The user space level is the level where user processes are executed. As presented in [*Chapter 5*](B19682_05.xhtml#_idTextAnchor104), *Working with Processes, Daemons, and Signals*, a process is a running instance of a program.

Where are devices in this grand scheme of things? Devices are managed by the *kernel*. To sum up, the kernel is in charge of managing processes, system calls, memory, and devices. When dealing with devices, the kernel manages **device drivers**, which are the interface between hardware components and software. All devices are accessible only in kernel mode, for a more secure and streamlined operation.

How does all this work? Well, the memory, known as RAM, consists of cells that are used to store information temporarily. Those cells are accessed by different programs that are executed and function as an intermediary between the CPU and the storage. The speeds of accessing memory are very high to secure a seamless process of execution. The management of user processes inside the user space is the kernel’s job. The kernel makes sure that none of the processes will interfere with each other. The kernel space is usually accessed only by the kernel, but there are times when user processes need to access this space. This is done through **system calls**. A system call is the way a user process requests a kernel service through an active process inside the kernel space, for anything such as **input/output** (**I/O**) requests to internal or external devices. All those requests transfer data to and from the CPU, through RAM, to get the job done.

In the following section, we will introduce you to the naming convention in Linux and how device files are managed.

## Device files and naming conventions

After seeing how the abstraction layers work, you may be wondering how Linux manages devices. Well, it does that with the help of **userspace /dev** (**udev**), which is a device manager for the kernel. It works with **device nodes**, which are special files (also called **device files**) that are used as an interface to the driver.

### Device files in Linux

`udev` runs as a daemon that listens to the user space calls that the kernel is sending, so it is aware of what kinds of devices are used and how they are used. The daemon is called `udevd` and its configurations are currently available under `/etc/udev/udev.conf`. You can concatenate the `/etc/udev/udev.conf` file to see its contents by running the following command:

```
cat /etc/udev/udev.conf
```

Each Linux distribution has a default set of rules that governs `udevd`. Those rules are normally stored under the `/etc/udev/rules.d/` directory, as shown in the following screenshot:

![Figure 6.1 – udevd rules location](img/B19682_06_01.jpg)

Figure 6.1 – udevd rules location

Note

The kernel sends calls for events using the **Netlink** socket. The netlink socket is an interface for inter-process communication that’s used for both userspace and kernel space processes alike.

The `/dev` directory is the interface between user processes and devices managed by the kernel. If you were to use the `ls -la /dev` command, you would see a lot of files inside, each with different names. If you were to do a long listing, you would see different file types. Some of the files will start with the letters *b* and *c*, but the letters *p* and *s* may also be present, depending on your system. Files starting with those letters are device files. The ones starting with *b* are **block devices**, and those starting with the letter *c* are **character devices**, as shown in the following screenshot:

![Figure 6.2 – Device files inside the /dev directory](img/B19682_06_02.jpg)

Figure 6.2 – Device files inside the /dev directory

Let’s see how disk devices are presented inside the `/dev` directory. But first, we’ve provided a few words about our working setup in the following note.

Important note

For most of the exercises in this book, we will be using virtual machines with planet names as hostnames, running different Linux-based operating systems. For example, `neptune` is running on Ubuntu 22.04.2 LTS Server, so when you see the `neptune` hostname on the shell’s prompt, you will know we are on an Ubuntu-based system. We also have `jupiter`, running on the openSUSE 15.4 Leap server, `saturn` running on Fedora 37 Workstation, `venus` running on AlmaLinux, and `mars` running on a Debian 11.6 server. Inside a virtual machine, device drivers are presented with different names than on bare-metal systems. We will provide details when we discuss device naming conventions in the following section. For some examples, though, which are marked accordingly, we will also use our primary workstation, which is running on Debian 12 GNU/Linux.

As shown in *Figure 6**.3*, the disk device, `sda`, is represented as a block device. Block devices have a fixed size that can easily be indexed. Character devices, on the other hand, can be accessed using data streams, as they don’t have a size like block devices. For example, printers are represented as character devices. In the following screenshot, `sg0` is an SCSI generic device, and not assigned to any disks in our case. We used our primary workstation running on Debian GNU/Linux and the device presented as `sda` is an external USB device:

![Figure 6.3 – Disk drives inside the /dev directory](img/B19682_06_03.jpg)

Figure 6.3 – Disk drives inside the /dev directory

In comparison, when listing devices from our `neptune` virtual machine, we will have the output presented in the following screenshot:

![Figure 6.4 – Virtual devices inside a virtual machine](img/B19682_06_04.jpg)

Figure 6.4 – Virtual devices inside a virtual machine

Device blocks presented with `vdaX` are virtual devices inside the virtual machine. You will learn more about virtual machines in [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231), *Working with* *Virtual Machines*.

But for now, let’s find out more about device naming conventions in Linux.

### Understanding device naming conventions

Linux uses a device naming convention that makes device management easier and more consistent throughout the Linux ecosystem. `udev` uses several specific naming schemes that, by default, assign fixed names to devices. Those names are standardized for device categories. For example, when naming network devices, the kernel uses information compiled from sources such as firmware, topology, and location. On a Red Hat-based system, five schemes are used for naming a network interface, and we encourage you to look at these on the Red Hat customer portal official documentation website: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9).

On a Debian-based system, the naming convention is similar in that it’s based on hardware buses’ names for predictability. This is similar to all modern Linux-based operating systems.

You could also check what `udev` rules are active on your system. On Debian and Red Hat-based distributions, they are stored in the `/``lib/udev/rules.d/` directory.

When it comes to hard drives or external drives, the conventions are more streamlined. Here are some examples:

*   `hda` (the master device), `hdb` (the slave device on the first channel), `hdc` (the master device on the second channel), and `hdd` (the slave device on the second channel)
*   `nvme0` (the first device controller – character device), `nvme0n1` (first namespace – block device), and `nvme0n1p1` (first namespace, first partition – block device)
*   `mmcblk` (for SD cards using eMMC chips), `mmcblk0` (first device), and `mmcblk0p1` (first device, first partition)
*   `sd` (for mass storage devices), `sda` (for the first registered device), `sdb` (for the second registered device), `sdc` (for the third registered device), and so on, and `sg` (for generic SCSI layers – character device)

The devices that we are most interested in regarding this chapter are the mass storage devices. Those devices are usually **hard disk drives** (**HDDs**) or **solid-state drives** (**SSDs**), which are used inside your computer to store data. These drives are most likely divided into partitions with a specific structure provided by the filesystem. We talked a little bit about filesystems earlier in this book in [*Chapter 2*](B19682_02.xhtml#_idTextAnchor053), *The* *Linux Shell and* *Filesystem*, when we referred to the Linux directory structure, but now, it is time to get into more details about filesystem types in Linux.

# Understanding filesystem types in Linux

When talking about physical media, such as hard drives or external drives, we are *not* referring to the directory structure. Here, we are talking about the structures that are created on the physical drive when formatting and/or partitioning it. These structures, depending on their type, are known as filesystems, and they determine how the files are managed when stored on the drive.

There are several types of filesystems, some being native to the Linux ecosystem, while others are not, such as specific Windows or macOS filesystems. In this section, we will describe only the Linux-native filesystems.

The most widely used filesystems in Linux are the `Ext`, `Ext2`, `Ext3`, and `Ext4`, the `XFS` filesystem, `ZFS`, and `btrfs` (short for `Ext4`, the latest iteration, is similar to `Ext3`, but better, with improved support for larger files, fragmentation, and performance. The `Ext3` filesystem uses 32-bit addressing, while `Ext4` uses 48-bit addressing, thus supporting files up to 16 TB in size. It also offers support for unlimited subdirectories as `Ext3` only supports 32k subdirectories. Also, support for extended timestamps was added in `Ext4`, offering two more bits for up to the year 2446 AD, and online defragmentation at the kernel level.

Nonetheless, `Ext4` is not a truly next-gen filesystem; rather, it is an improved, trustworthy, robust, and stable *workhorse* that failed the data protection and integrity test. Its journaling system is not suitable for detecting and repairing data corruption and degradation. That is why other filesystems, such as `XFS` and `ZFS`, started to resurface by being used in Red Hat Enterprise Linux, starting from version 7 (`XFS`) and in Ubuntu since version 16.04 (`ZFS`).

The case of `btrfs` is somewhat controversial. It is considered a modern filesystem, but it is still used as a single-disk filesystem and not used in multiple disk volume managers due to several performance issues compared to other filesystems. It is used in SUSE Linux Enterprise and openSUSE, is no longer supported by Red Hat, and has been voted as the future default filesystem in Fedora, starting with version 33.

Here are some more details on the major filesystem features:

*   `Ext4` filesystem was designed for Linux right from the outset. Even though it is slowly being replaced with other filesystems, this one still has powerful features. It offers block size selection, with values between 512 and 4,096 bytes. There is also a feature called inode reservation, which saves a couple of inodes when you create a directory, for improved performance when creating new files.

    The layout is simple, written in `Ext4` takes advantage of. Among them, we will bring the following into the discussion: a maximum filesystem size of 1 `fsck` command for speedy filesystem checks, the use of checksums for journaling and better reliability, and the use of improved timestamps.

*   **ZFS**: This filesystem was created at Sun Microsystems and combines a file system and a logical volume manager into one solution. It was announced in 2004, with development starting in 2001, and was first integrated into the Solaris operating system, then used (not the default though) by Debian, FreeBSD, and others. ZFS is a highly scalable 128-bit system that offers simple administration, data integrity, scalability, and performance. Development of this filesystem is done through the **OpenZFS** open source project. ZFS offers a complex structure by using a copy-on-write mechanism, different from traditional filesystems. For more detailed information about ZFS, we recommend the following link: [https://openzfs.github.io/openzfs-docs/Getting%20Started/index.html](https://openzfs.github.io/openzfs-docs/Getting%20Started/index.html).
*   `Ext4` to other competent filesystem types. Among those is `XFS`. This filesystem was first created by Silicon Graphics, Inc and used in the IRIX operating system. Its most important key design element is performance as it is capable of dealing with large datasets. Furthermore, it is designed to handle parallel I/O tasks with a guaranteed high I/O rate. The filesystem supports up to 16 EB with support for individual files up to 8 EB. `XFS` has a feature to journal quota information, together with online maintenance tasks such as defragmenting, enlarging, and restoring. There are also specific tools for backup and restore, including `xfsdump` and `xfsrestore`.
*   `btrfs`) is still under development, but it addresses issues associated with existing filesystems, including the lack of snapshots, pooling, checksums, and multi-device spanning. These are features that are required in an enterprise Linux environment. The ability to take snapshots of the filesystem and maintain its internal framework for managing new partitions makes `btrfs` a viable newcomer in terms of the critical enterprise ecosystem.

There are other filesystems that we did not discuss here, including `cat /``proc/filesystems` command.

Linux implements a special software system that is designed to run specific functions of the filesystems. It is known as the **virtual file system** and acts as a bridge between the kernel and the filesystem types and hardware. Therefore, when an application wants to open a file, the action is delivered through the Virtual File System as an abstraction layer:

![Figure 6.5 – The Linux Virtual File System abstraction layer](img/B19682_06_05.jpg)

Figure 6.5 – The Linux Virtual File System abstraction layer

Basic filesystem functions include provisioning namespaces, metadata structures as a logical foundation for hierarchical directory structures, disk block usage, file size and access information, and high-level data for logical volumes and partitions. There is also an **application programming interface** (**API**) available for every filesystem. Thus, developers can access system function calls for filesystem object manipulation with specific algorithms for creating, moving, and deleting files, or for indexing, searching, and finding files. Furthermore, every modern filesystem provides a special access rights scheme that’s used to determine the rules governing a user’s access to files.

At this point, we have already covered the principal Linux filesystems, including `EXT4`, `btrfs`, and `XFS`. In the next section, we will teach you the basics of disks and partition management in Linux.

# Understanding disks and partitions

Understanding disks and partitions is a key asset for any system administrator. Formatting and partitioning disks is critical, starting with system installation. Knowing the type of hardware available on your system is important, and it is therefore imperative to know how to work with it. One of these is the disk; let’s look at this in further detail.

## Common disk types

A **disk** is a hardware component that stores your data. It comes in various types and uses different interfaces. The main disk types are the well-known **spinning HDD**, the SSD, and the **non-volatile memory express** (**NVMe**). SSDs and NVMes use RAM-like technologies, with better energy consumption and higher transfer rates than original spinning hard drives. The following interfaces are used:

*   **Integrated Drive Electronics** (**IDE**): This is an old standard that’s used on consumer hardware with small transfer rates. It’s now deprecated.
*   **Serial Advanced Technology Attachment** (**SATA**): This replaced IDEs and has transfer rates of up to 16 GB/s.
*   **Small Computer Systems Interface** (**SCSI**): This is used mostly in enterprise servers with RAID configurations with sophisticated hardware components.
*   **Serial Attached SCSI** (**SAS**): This is a point-to-point serial protocol interface with transfer rates similar to SATA. It is mostly used in enterprise environments for their reliability.
*   **Universal Serial Bus** (**USB**): This is used for external hard drives and memory drives.

Each disk has a specific geometry that consists of heads, cylinders, tracks, and sectors. On a Linux system, to see the information regarding a disk’s geometry, you can use the `fdisk -``l` command.

On our primary workstation, we have a single SSD running Debian 12 GNU/Linux and a USB device inserted in one of the ports. We will run the following command to obtain information about the drives on our machine:

```
sudo fdisk -l
```

The following screenshot shows excerpts of the `fdisk` command’s output for both drives:

![Figure 6.6 – The output of the fdisk -l command showing disk information](img/B19682_06_06.jpg)

Figure 6.6 – The output of the fdisk -l command showing disk information

The output of the `fdisk` utility may look intimidating at first, but rest assured that we will explain it to you so that it will look friendlier from now on. By using the `fdisk` utility without a specific partition as an argument, all the partition information available inside `/proc/partitions` will be shown. In the example shown in the preceding screenshot, you have details on two disks that are available on our system: a 1 TB Lexar NM620 SSD and an 8 GB USB flash drive attached. Let’s explain how the 1 TB drive is shown:

*   First, you have `Disk model` with the name of the drive, `Units` as sectors, each of which has a size of 512 bytes, `Disklabel type` as GPT, and `Disk identifier`, which is unique for each drive.
*   Next is a table of the partitions available on the disk. This table has six columns (sometimes seven columns, as in the case of the USB flash drive, shown on the lower side of the screenshot). The first column has the `Device` header and shows the partition naming scheme. The second and third columns (in our example) show the starting and ending sectors. The fourth column shows the total number of sectors on the partition. The fifth column shows the size of the partition in human-readable format and the last column shows the type of the filesystem.

Knowing basic information about disk devices on your system is merely the starting point for working with disks and partitions on Linux. Disks are just a big chunk of metal if we don’t format and partition them so that the system can use them. This is why, in the next section, we will teach you what partitions are.

## Partitioning disks

Commonly, disks use **partitions**. To understand partitions, knowing a disk’s geometry is essential. This legacy knowledge base is useful even when dealing with SSDs. Partitions are contiguous sets of sectors and/or cylinders, and they can be of several types: **primary**, **extended**, and **logical** partitions. A maximum number of 15 partitions can exist on a disk. The first four will be either primary or extended, and the remaining are logical partitions. Furthermore, there can only be a single extended partition, but they can be divided into several logical partitions until the maximum number is reached.

### Partition types

There are two major partition types – the `0x0c` for FAT, `0x07` for NTFS, `0x83` for a Linux filesystem type, and `0x82` for swap. GPT became a part of the **Unified Extensible Firmware Interface** (**UEFI**) standard as a solution to some issues with MBR, including partition limitations, addressing methods, using only one copy of the partition table, and so on. It supports up to 128 partitions and disk sizes of up to 75.6 **Zettabytes** (**ZB**).

### The partition table

The **partition table** of a disk is stored inside the disk’s MBR. MBR is the first 512 bytes of a drive. Out of these, the partition table is 64 bytes and is stored after the first 446 bytes of records. At the end of MBR, there are 2 bytes known as the end of sector marker. The first 446 bytes are reserved for code that usually belongs to a bootloader program. In the case of Linux, the bootloader is called **GRand Unified** **Bootloader** (**GRUB**).

When you boot up a Linux system, the bootloader looks for the active partition. There can only be one active partition on a single disk. When the active partition is located, the bootloader loads items. The partition table has 4 entries, each of which is 16 bytes in size, with each belonging to a possible primary partition on the system. Furthermore, each entry contains information regarding the beginning address of `cylinder/head/sectors`, the partition type code, the end address of `cylinder/head/sectors`, the starting sector, and the number of sectors inside one partition.

### Naming partitions

The kernel interacts with the disk at a low level. This is done through device nodes that are stored inside the `/dev` directory. Device nodes use a simple naming convention that tells you which disk is the one that requires your attention. Looking at the contents of the `/dev` directory, you can see all the available disk nodes, also referred to as disk drives, in *Figure 6**.2* and *Figure 6**.3* earlier in this section. A short explanation is always useful, so disks and partitions are recognized as follows:

*   The first hard drive is always `/dev/sda` (for an SCSI or SATA device)
*   The second hard drive is `/dev/sdb`, the third is `/dev/sdc`, and so on
*   The first partition of the first disk is `/dev/sda1`
*   The first partition of the second disk is `/dev/sdb1`
*   The second partition of the second disk is `/dev/sdb2`, and so on

We specified that this is true in the case of an SCSI and SATA, and we need to explain this in a little more detail. The kernel gives the letter designation, such as *a*, *b*, and *c*, based on the ID number of the SCSI device, and not based on the position of the hardware bus.

### Partition attributes

To learn about your partition’s attributes, you can use the `lsblk` command. We will run it on our Debian system, as shown in the following screenshot:

![Figure 6.7 – The output of lsblk](img/B19682_06_07.jpg)

Figure 6.7 – The output of lsblk

The `lsblk` command shows the device’s name (the node’s name from `sysfs` and the `udev` database), the major and minor device number, the removable state of the device (`0` for a non-removable device and `1` for a removable device), the size in human-readable format, the read-only state (again, using `0` for the ones that are not read-only and `1` for the read-only ones), the type of device, and the device’s mount point (where available).

Now that we know more about the drive, let’s learn how to alter a disk’s partition table.

### Partition table editors

In Linux, there are several tools we can use when managing partition tables. Among the most commonly used ones are the following:

*   `fdisk`: A command-line partition editor, perhaps the most widely used one
*   `Sfdisk`: A non-interactive partition editor, used mostly in scripting
*   `parted`: The GNU (the recursive acronym for GNU is *GNU's Not Unix*) partition manipulation software
*   `gparted`: The graphical interface for `parted`

Of these, we will only detail how to use `fdisk` as this is the most widely used command-line partition editor in Linux. It is found in both Ubuntu/Debian and RHEL/Fedora or openSUSE and many other distributions too.

But before we use `fdisk`, we would like to see the partitions that the operating system knows about. If you are not sure about the operations you just completed, you can always visualize the contents of the `/proc/partitions` file with the `cat` command:

![Figure 6.8 – Listing the /proc/partitions file](img/B19682_06_08.jpg)

Figure 6.8 – Listing the /proc/partitions file

To use `fdisk`, you must be the root user. We advise you to use caution when using `fdisk` as it can damage your existing partitions and disks. `fdisk` can be used on a particular disk as follows:

![Figure 6.9 – Using fdisk for the first time](img/B19682_06_09.jpg)

Figure 6.9 – Using fdisk for the first time

You will notice that when using `fdisk` for the first time, you are warned that changes will be done to the disk only when you decide to write them to it. You will also be prompted to introduce a command, and you will be shown the `m` option for help. We advise you to always use the help menu, even if you already know the most used commands.

When you type `m`, you will be shown the entire list of commands available for `fdisk`. You will see options to manage partitions, create new boot records, save changes, and others. Partition table editors are important tools for managing disks in Linux. Their use is incomplete if you do not know how to format a partition. In the next section, we will show you how to partition a disk drive.

### Creating and formatting partitions

We will use the `fdisk` utility to create a new partition table on a USB memory stick plugged into our primary workstation running Debian GNU/Linux. We will create an MBR partition table using the following command:

```
sudo fdisk /dev/sda
```

We will use the `o` option to create an empty MBR partition table and then the `w` option to save the changes to disk. The output of the command is shown in the following screenshot:

![Figure 6.10 – Creating a new MBR partition table with fdisk](img/B19682_06_10.jpg)

Figure 6.10 – Creating a new MBR partition table with fdisk

With that, the partition table has been created, but there is no partition defined on the disk. While still inside the `fdisk` command-line interface, you can use the `v` option to verify the newly created partition table and the `I` option to see information about existing partitions. You will see some output saying that no partitions have been defined yet. So, it is time to set up a new partition.

To create a new partition, we will use the following series of options:

*   The `n` option to start the creation processes
*   The `p` option when asked to create either a primary (`p`) or extended (`e`) partition type
*   Enter the partition number (use the default of `1`)
*   Enter the first sector (use the default of `2048`)
*   Enter the last sector – if you want a specific size for the partition, you can use size values in KB, MB, GB, and so on, or sector values (the default is the maximum size of the disk)
*   If asked to remove any signatures, type `Y` to remove them
*   `w` to write changes to disk

The output of the previous series of actions is shown in the following screenshot:

![Figure 6.11 – Creating a new partition with fdisk](img/B19682_06_11.jpg)

Figure 6.11 – Creating a new partition with fdisk

With that, the partition has been created, but it hasn’t been formatted. Before we learn how to format partitions, let’s learn how to back up a partition table.

There are situations when you will need to back up and restore your `dd` utility. The command to use is as follows:

```
sudo dd if=/dev/sda of=mbr-backup bs=512 count=1
```

This program is very useful and powerful as it can clone disks or wipe data. Here is an example showing the output of the command:

![Figure 6.12 – Backing up MBR with the dd command](img/B19682_06_12.jpg)

Figure 6.12 – Backing up MBR with the dd command

The `dd` command has a clear syntax. By default, it uses the standard input and standard output, but you can change those by specifying new input files with the `if` option, and output files with the `of` option. We specified the input file as the device file for the disk we wanted to back up and gave a name for the backup output file. We also specified the block size using the `bs` option, and the `count` option to specify the number of blocks to read.

To restore the bootloader, we can use the `dd` command, as follows:

```
sudo dd if=~/mbr-backup of=/dev/sda bs=512 count=1
```

Now that you have learned how to use `dd` to back up a partition table, let’s format the partition we created earlier. The most commonly used program for formatting a filesystem on a partition is `mkfs`. Formatting a partition is also known as *making* a filesystem, hence the name of the utility. It has specific tools for different filesystems, all using the same frontend utility. The following is a list of all filesystems supported by `mkfs`:

![Figure 6.13 – Details regarding the mkfs utility](img/B19682_06_13.jpg)

Figure 6.13 – Details regarding the mkfs utility

To format the target disk as having the `Ext4` filesystem, we will use the `mkfs` utility. The commands to execute are shown here:

1.  First, we will run the `fdisk` utility to make sure that we select the largest disk correctly. Run the following command:

    ```
    sudo fdisk -l
    ```

2.  Then, check the output with extreme caution and select the correct disk name.
3.  Now that we know which disk to work with, we will use `mkfs` to format it as an `Ext4` filesystem. The output is shown here:

![Figure 6.14 – Formatting an Ext4 partition using mkfs](img/B19682_06_14.jpg)

Figure 6.14 – Formatting an Ext4 partition using mkfs

When using `mkfs`, there are several options available. To create an `Ext4` type partition, you can either use the command shown in *Figure 6**.14* or you can use the `-t` option followed by the filesystem type. You can also use the `-v` option for a more verbose output, and the `-c` option for bad sector scanning while creating the filesystem. You can also use the `-L` option if you want to add a label for the partition right from the command. The following is an example of creating an `Ext4` filesystem partition with the name `newpartition`:

```
sudo mkfs -t ext4 -v -c -L newpartition /dev/sda
```

Once a partition is formatted, it’s advised that you check it for errors. Similar to `mkfs`, there is a tool called `fsck`. This is a utility that sometimes runs automatically following an abnormal shutdown or on set intervals. It has specific programs for the most commonly used filesystems, just like `mkfs`. The following is the output of running `fsck` on one of our partitions. After running, it will show whether there are any problems. In the following screenshot, the output shows that checking the partition resulted in no errors:

![Figure 6.15 – Using fsck to check a partition](img/B19682_06_15.jpg)

Figure 6.15 – Using fsck to check a partition

After partitions are created, they need to be mounted; otherwise, they cannot be used.

Important note

**Mounting** is an important action in Linux, and any other operating system for that matter. By mounting, you give the operating system access to the disk resource in such a way that it looks like it is using a local disk. On Linux, the external disk that is mounted is linked to a mount point, which is a directory on the local filesystem. Mount points are essential to POSIX-compatible operating systems, such as Linux. Mounting a disk makes it accessible to the entire operating system through mount points. For more information on mounting, visit [https://docs.oracle.com/cd/E19455-01/805-7228/6j6q7ueup/index.html](https://docs.oracle.com/cd/E19455-01/805-7228/6j6q7ueup/index.html).

Each partition will be mounted inside the existing filesystem structure. Mounting is allowed at any point in the tree structure. Each filesystem is mounted under certain directories, created inside the directory structure. We will explore mounting and unmounting partitions in the next section.

### Mounting and unmounting partitions

The `mount`, and the `umount`. To see whether a certain partition is mounted, you can simply type `mount` and see the output, which will be of a significant size. You can use `grep` to filter it:

```
mount | grep /dev/sda
```

We are looking for `/dev/sda` in the output, but it is not shown. This means that the drive is not mounted.

To mount it, we need to make a new directory. For simplicity, we will show all the steps required until you mount and use the partition:

1.  Create a new directory to mount the partition. In our case, we created a new directory called `USB` inside the `/``home/alexandru` directory:

    ```
    mkdir USB
    ```

2.  Mount the partition using the following command:

    ```
    mbr-backup file we created a few steps back to the newly mounted USB memory stick using the following command:

    ```
    sudo cp mbr-backup USB/
    ```

    ```

The following is the output for all the commands from the preceding list:

![Figure 6.16 – Mounting an external memory stick](img/B19682_06_16.jpg)

Figure 6.16 – Mounting an external memory stick

The `mount` command needs to be used with superuser permission. If you try to mount an external USB device without `sudo`, you will get the following message:

![Figure 6.17 – Error for not using sudo with mount](img/B19682_06_17.jpg)

Figure 6.17 – Error for not using sudo with mount

The `mount` utility has many options available. Use the help menu to see everything that it has under the hood. Now that the partition has been mounted, you can start using it. If you want to unmount it, you can use the `umount` utility. You can use it as follows:

```
sudo umount /dev/sda
```

When unmounting a filesystem, you may receive errors if that partition is still in use. Being in use means that certain programs from that filesystem are still running in memory, using files from that partition. Therefore, you first have to close all running applications, and if other processes are using that filesystem, you will have to kill them, too. Sometimes, the reason a filesystem is busy is not clear at first, and to know which files are open and running, you can use the `lsof` command:

```
sudo lsof | grep /dev/sda
```

Mounting a filesystem only makes it available until the system is shut down or rebooted. If you want the changes to be persistent, you will have to edit the `/etc/fstab` file accordingly. First, open the file with your favorite text editor:

```
sudo nano /etc/fstab
```

Add a new line similar to the one that follows:

```
/dev/sda /mnt/sdb ext4 defaults 0 0
```

The `/etc/fstab` file is a configuration file for the filesystem table. It consists of a set of rules needed to control how the filesystems are used. This simplifies the need to manually mount and unmount each disk when used, by drastically reducing possible errors. The table has a six-column structure, with each column designated with a specific parameter. There is only one correct order for the parameters to work:

*   **Device name**: Either by using UUID or the mounted device name
*   **Mount point**: The directory where the device is, or will be, mounted
*   **Filesystem type**: The filesystem type used
*   **Options**: The options shown, with multiple ones separated by commas
*   `0` = no backup, `1` = dump utility backup
*   `0` = no `fsck` filesystem check, with `1` for the root filesystem, and `2` for other partitions

By updating the `/etc/fstab` file, the mounting is permanent and is not affected by any shutdown or system reboot. Usually, the `/etc/fstab` file only stores information about the internal hard drive partitions and filesystems. The external hard drives or USB drives are automatically mounted under `/media` by the kernel’s **hardware abstraction** **layer** (**HAL**).

By now, you should be comfortable with managing partitions in Linux, but there is still one type of partition we have not discussed: the **swap partition**. In the next section, we will introduce you to how swap works on Linux.

### Swap partition

Linux uses a robust swap implementation. The virtual memory uses hard drive space when physical memory is full through swap. This additional space is made available either for the programs that do not use all the memory they are given, or when memory pressure is high. Swapping is usually done using one or more dedicated partitions as Linux permits multiple swap areas. The recommended swap size is at least the total RAM on the system. To check the actual swap used on the system, you can concatenate the `/proc/swaps` file or use the `free` command to see swap utilization, as shown in the following screenshot:

![Figure 6.18 – Checking the currently used swap](img/B19682_06_18.jpg)

Figure 6.18 – Checking the currently used swap

If swap is not set up on your system, you can format a partition as swap and activate it. The commands to do that are as follows:

```
mkswap /dev/sda1
swapon /dev/sda1
```

The operating system is caching file contents inside the memory to prevent the use of swap as much as possible. This happens because memory is working at much higher speeds compared to hard drives or hard disk drives. Only when available memory is limited will swap be used. However, the memory that the kernel uses is never swapped; only the memory that the user space is using gets to be swapped. This assures data integrity for the kernel. Refer to the utilities we applied in [*Chapter 5*](B19682_05.xhtml#_idTextAnchor104) to show memory usage in Linux.

Filesystems and partitions are the bare bones of any disk management task, but there are still several hiccups that an administrator needs to overcome, and this can be solved by using logical volumes. This is why, in the next section, we will introduce you to LVM.

# Introducing LVM in Linux

Some of you may have already heard of **LVM**. For those who do not know what it is, we will explain it briefly in this section. Imagine a situation where your disks run out of space. You can always move it to a larger disk and then replace the smaller one, but this implies system restarts and unwanted downtimes. As a solution, you can consider LVM, which offers more flexibility and efficiency. By using LVM, you can add more physical disks to your existing volume groups while they’re still in use. This still offers the possibility to move data to a new hard drive but with no downtime – everything is done while filesystems are online.

The utilities used in Linux for LVM management are called `pvcreate`, `vgcreate`, `vgdisplay`, `lvcreate`, `lvextend`, and `lvdisplay`. Let’s learn how to use them.

As we don’t have a system with LVM set up just yet, we will show you the steps that are necessary to create new LVM volumes by using another system with two internal drives: one with the operating system installed on it, and a second, internal one that’s available. We’ll be using Debian GNU/Linux, but the commands are the same for any other Linux-based operating system.

Follow these steps to create an LVM volume:

1.  Use the `fdisk` command to verify the names of the available disks (you can also use `lsblk` for this step):

    ```
    /dev/sda.
    ```

2.  Create the LVM physical volume with the `pvcreate` command:

    ```
    wipefs utility. The output is shown in the following screenshot:
    ```

![Figure 6.19 – Using pvcreate to create an LVM physical volume](img/B19682_06_19.jpg)

Figure 6.19 – Using pvcreate to create an LVM physical volume

1.  Create a new volume group to add the new physical volume to using the `vgcreate` command:

    ```
    vgdisplay command:
    ```

![Figure 6.20 – Creating and viewing details of the new volume](img/B19682_06_20.jpg)

Figure 6.20 – Creating and viewing details of the new volume

1.  Now, let’s create a logical volume using some of the space available from the volume group, using `lvcreate`. Use the `-n` option to add a name for the logical volume and `-L` to set the size in a human-readable manner (we created a 5 GB logical volume named `projects`):

![Figure 6.21 – Creating a logical volume using lvcreate](img/B19682_06_21.jpg)

Figure 6.21 – Creating a logical volume using lvcreate

1.  Check whether the logical volume exists:

    ```
    sudo ls /dev/mapper/newvolume-projects
    ```

2.  The newly created device can only be used if it’s formatted using a known filesystem and mounted afterward, in the same way as a regular partition. First, let’s format the new volume:

![Figure 6.22 – Formatting the new logical volume as an Ext4 filesystem](img/B19682_06_22.jpg)

Figure 6.22 – Formatting the new logical volume as an Ext4 filesystem

1.  Now, it’s time to mount the logical volume. First, create a new directory and mount the logical volume there. Then, check its size using the `df` command:

![Figure 6.23 – Mounting the logical volume](img/B19682_06_23.jpg)

Figure 6.23 – Mounting the logical volume

1.  All changes implemented hitherto are not permanent. To make them permanent, you will have to edit the `/etc/fstab` file by adding the following within the file:

    ```
    vgdisplay command to see the following details:

    ```
    lvextend command. We will extend the initial size by 5 GB, for a total of 10 GB. The following is an example:
    ```

    ```

![Figure 6.24 – Extending the logical volume using lvextend](img/B19682_06_24.jpg)

Figure 6.24 – Extending the logical volume using lvextend

1.  Now, resize the filesystem so that it fits the new size of the logical volume using `resize2fs` and check for the size with `df`:

![Figure 6.25 – Resizing the logical volume with resize2fs and checking for the size with df](img/B19682_06_25.jpg)

Figure 6.25 – Resizing the logical volume with resize2fs and checking for the size with df

LVM is an advanced topic that will prove essential for any Linux system administrator to have. The brief hands-on examples we provided in this section only show the basic operations that you need to work with LVM. Feel free to dig deeper into this topic if you need to.

In the following section, we will discuss several more advanced LVM topics, including how to take full filesystem snapshots.

## LVM snapshots

What is an LVM snapshot? It is a frozen instance of an LVM logical volume. More specifically, it uses a **copy-on-write** technology. This technology monitors each block of the existing volume, and when blocks change, due to new writings, that block’s value is copied to the snapshot volume.

The snapshots are created constantly and instantly and persist until they are deleted. This way, you can create backups from any snapshot. As snapshots are constantly changing due to the copy-on-write technology, initial thoughts on the size of the snapshot should be given when creating one. Take into consideration, if possible, how much data is going to change during the existence of the snapshot. Once the snapshot is full, it will be automatically disabled.

### Creating a new snapshot

To create a new snapshot, you can use the `lvcreate` command, with the `-s` option. You can also specify the size with the `-L` option and add a name for the snapshot with the `-n` option, as follows:

![Figure 6.26 – Creating an LVM snapshot with the lvcreate command](img/B19682_06_26.jpg)

Figure 6.26 – Creating an LVM snapshot with the lvcreate command

In the preceding command, we set a size of 5 GB and used the name `linux-snapshot-01`. The last part of the command contains the destination of the volume for which we created the snapshot. To list the new snapshot, use the `lvs` command:

![Figure 6.27 – Listing the available volume and the newly created snapshot](img/B19682_06_27.jpg)

Figure 6.27 – Listing the available volume and the newly created snapshot

For more information on the logical volumes, run the `lvdisplay` command. The output will show information about all the volumes, and among them, you will see the snapshot we just created.

When we created the snapshot, we gave it a size of 5 GB. Now, we would like to extend it to the size of the source, which was 10 GB. We will do this with the `lvextend` command:

![Figure 6.28 – Extending the snapshot from 5 to 10 GB](img/B19682_06_28.jpg)

Figure 6.28 – Extending the snapshot from 5 to 10 GB

As shown in the preceding screenshot, the name that the snapshot volume is using is different from the one we used. Even though we used the name `linux-snapshot-01` for the snapshot volume, if we do a listing of the `/dev/mapper/` directory, we will see that the name uses two more dashes instead. This is a convention that’s used to represent logical volume files.

Now that you know how to create snapshots, let’s learn how to restore a snapshot.

### Restoring a snapshot

To restore a snapshot, first, you would need to unmount the filesystem. To unmount, we will use the `umount` command:

```
sudo umount /home/alexandru/LVM
```

Then, we can proceed to restore the snapshot with the `lvconvert` command. After the snapshot is merged into the source, we can check this by using the `lvs` command. The output of the two commands is shown in the following screenshot:

![Figure 6.29 – Restoring and checking the snapshot](img/B19682_06_29.jpg)

Figure 6.29 – Restoring and checking the snapshot

Following the merge, the snapshot is automatically removed.

We have now covered all the basics of LVM in Linux. LVM is more complicated than normal disk partitioning. It might be intimidating to many, but it can show its strengths when needed. Nevertheless, it also comes with several drawbacks – for example, it can add unwanted complexity in a disaster recovery scenario or when a hardware failure occurs. But all this aside, it is still worth learning about.

# Summary

Managing filesystems and disks is an important task for any Linux system administrator. Understanding how devices are managed in Linux, and how to format and partition disks, is essential. Furthermore, it is important to learn about LVM as it offers a flexible way to manage partitions.

Mastering those skills will give you a strong foundation for any basic administration task. In the following chapter, we will introduce you to the vast domain of **networking** in Linux.

# Questions

If you managed to skim through some parts of this chapter, you might want to recap a few essential details about Linux filesystem and disk management:

1.  Think of another tool to use for working with disks and install it.

    `parted` and use it from the command line. You can also use GParted from the GUI.

2.  Experiment with using Disks (in GNOME) and KDE Partition Manager (in KDE) and use the command-line interface side by side.

    **Hint**: Keep both applications open and use the command-line utilities side by side. Try to format and mount a disk from the command line while keeping the GUI apps open.

3.  Format new partitions using different filesystems.

    `btrfs` instead of `ext4`.

4.  Explore your filesystem and disks.

    `lsblk`, `df`, and `fdisk`.

# Further reading

For more information about what was covered in this chapter, please refer to the following Packt titles:

*   *Linux Administration Best Practices*, by Scott Alan Miller
*   *Mastering Ubuntu Server – Fourth Edition*, by Jay LaCroix