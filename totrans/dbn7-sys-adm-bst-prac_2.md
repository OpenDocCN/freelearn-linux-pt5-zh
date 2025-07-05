# Chapter 2. Filesystem Layout

Some of the first decisions that must be made, even before installing Debian, involve deciding the best way to format the storage space for the installation. This includes what type of filesystem to use, how to partition it for the best effect, and whether and what to encrypt for security. The actual work of partitioning and boot code placement is handled by the Debian installer, and can be altered later using standard Linux bootloader and partitioning utilities. The installation process and the utilities are covered well by the Debian installation guide mentioned in the previous chapter, and the documentations for the GRUB2, fdisk, and GNU Parted included with the appropriate software packages.

This chapter serves as a basic introduction to the concepts of boot loading and disk partitioning, along with some guidelines to keep in mind when installing Debian or updating your boot or partitioning schemes. Do not worry if you are still uncertain what is best for your situation when first installing Debian. As we shall see, the defaults will work just fine for most cases, and the beginner can't really go wrong while using them when in doubt.

# Partition tables

Each architecture has its own characteristic method of partitioning disk drives and placing boot code in the appropriate place. For most, this is very straightforward. However, the Intel architecture is undergoing changes that require some understanding of the boot process and disk layout.

## Single or multiboot

One of the first choices to be made when installing any Linux distribution is whether the system will be single or multiboot. In general, many developers run both Windows and Linux on the same machine. In some cases, due to licensing restrictions or just personal preference, they wish to use the Windows installation that came with their computer and boot into one or the other as needed. This is perfectly fine, and most bootloaders will recognize both operating systems and provide menu items to boot the desired one. Another option is to use Xen or similar virtualization software to boot both simultaneously. A third choice is to run Windows under a Linux **virtual machine** (**VM**) using QEMU or KVM software. Creating VMs under QEMU, KVM, Xen, or any other virtualization software (such as VMware), would be a complete book in itself. For our purpose, we will consider a VM as essentially equivalent to an actual hardware system, since the issues outside VM creation are identical.

### Tip

Best practice, if this is a single operating system server environment, will be a single-boot system. If this is a developer system that may require booting into an alternative operating system, use dual boot. VM generally does not require dual boot.

## BIOS versus UEFI

Up until the late 1990s, the **Basic Input/Output System**, or **BIOS**, was the way all Intel-based systems were booted. Its disk partitioning information was held in a **Master Boot Record** (**MBR**) with additional code in the first sector of each bootable partition. With the advent of the Microsoft-sponsored Secure Boot feature, and its associated boot mechanism known as the **Universal Extensible Firmware Interface** (**UEFI**), there is a new partitioning layout, and additional considerations.

### Boot code under BIOS

BIOS is the traditional boot method, and is well-supported by Debian. There are several choices for the placement of the boot code. Common practice is for it to be placed in the MBR at the beginning of the boot disk. However, if there are multiple operating systems already installed (especially Microsoft Windows), this replaces the installed bootloader with the one common to Linux. This is generally not a problem, since the installation and update process searches for other operating systems and includes the ability to boot them as an alternative in the boot menu.

### Note

The current bootloader for Debian on Intel is called GRUB2, although other, older loaders exist and may be installed as an alternative.

However, there are occasions where the original, non-Linux bootloader is preferred. For example, some Windows installations won't update properly if a non-Windows bootloader is installed. In this case, the Linux boot code can be placed at the beginning of the Linux boot partition rather than the MBR at the beginning of the drive, where the non-Linux bootloader can usually find it and offer it as an alternative on its boot menu.

The problem of Windows updates when using the Linux GRUB2 bootloader is quite complex. The issue seems to occur primarily with major Internet Explorer version upgrades, and the reasons remain unclear, at least in any discussions and bug reports I've been able to find. Adding to the problem is the occasional report of inconsistent recognition of Linux boot partitions by the Windows loader. There seems to be no hard-and-fast guideline as to which Windows installations will experience problems and which will not. The only certain way to know is to try it, and that requires patience, good backups, and a willingness to start over if it doesn't work.

So, if you are planning to use a dual or multiboot layout that includes Windows, and you don't have the time, patience, and determination to actually try all the alternatives, the answer comes down to the following practical considerations:

*   Can you live without a major version upgrade to Internet Explorer?
*   Can you run Windows as a VM instead of as part of a dual or multiboot system?
*   Will your Windows bootloader recognize the Linux boot partition?

Many users never upgrade major versions of Internet Explorer, and are perfectly satisfied with security and feature updates to their current version. If this works for you, then proceed with the default placement in the MBR. If you absolutely must have the ability to upgrade major Internet Explorer versions, consider running Windows as a VM rather than as part of a dual or multiboot system. If you do not wish to do so (usually because of virtual hardware compatibility or licensing issues), then go with installing the Linux bootloader at the beginning of the Linux partition. Recent versions of Windows (since Vista) are pretty good about recognizing the Linux boot partition and adding it to the boot menu.

### Tip

Best practice is to use the default placement in the MBR. Only if you truly need the original bootloader should you place the Linux boot code at the beginning of the Linux boot partition and, if necessary, configure the non-Linux bootloader to include it in the boot menu, if it doesn't do so automatically.

### Boot code under UEFI

The UEFI is a recent development by Intel and Microsoft that supports what is called Secure Boot, which requires all the loaded firmware to be signed or it won't be loaded. This is a problem for Linux, since the keys required for signing must, under the current GPL, be made public. This, of course, defeats the purpose. There are several workarounds, including some being used by Red Hat, SuSE, and Ubuntu, which are being discussed by the Debian developers and will probably be included in an update at some future point. For now, the UEFI specification allows Secure Boot to be disabled, and that is the recommended way to install Debian so that it boots under UEFI. It is also possible to switch on Legacy mode in most UEFI implementations, which allows the old MBR method to work as well.

Under UEFI, boot code is placed in a subdirectory in a special partition. Generally, this will be a subdirectory of `/EFI` in the first partition on the disk (formatted with the FAT32 filesystem). Generally, the boot modules and configuration files are placed in the `/EFI/grub` directory in the UEFI partition. It is not a good idea to replace the default EFI module (usually `/EFI/Boot/bootx64.efi`) by copying the `grubx64.efi` module over it, as some have recommended in the past. Debian installation generally takes care of including the GRUB loader as one of the options when booting, and if it isn't the default option, the boot settings menu should be used to set it as the default. It can also be used to add it as an option if the installation doesn't do this for you.

### Note

Getting into the UEFI boot settings menu usually involves holding down certain keys while booting the computer, very similar to the way the old BIOS menus were invoked. It is different for each computer model.

UEFI is new to Debian 7.

# Filesystem types

Selecting a filesystem format is the next major choice before installing Debian. The supported formats that are appropriate for a Linux installation include ext2, ext3, ext4, JFS, XFS, ReiserFS, and Btrfs. The first three are actually progressive versions of the **extended filesystem** (**ext**) developed specifically for Linux.

## ext2, ext3, and ext4

The ext filesystem was originally developed to overcome the limitations of the MINIX filesystem.

### Note

MINIX was Linus Torvalds' inspiration for Linux.

The **second extended filesystem** (**ext2**) improved upon it, while the **third extended filesystem** (**ext3**) added journaling, as well as performance improvements. The **fourth extended filesystem** (**ext4**) added additional features and performance improvements.

### Note

The ability to disable journaling is one reason ext2 was sometimes used over ext3 for flash drives in order to reduce the write cycles.

## Journaled File System

Developed by IBM for its Unix-like AIX operating system, and offered as an alternative to the ext and ext2 filesystems via release under the GPL, **Journaled File System** (**JFS**) is one of the alternatives to the current ext4\. It uses fewer resources, while remaining quite stable and resilient. It includes many features of Btrfs, and is a good choice when CPU power is limited, or with database systems that require synchronous writes to survive hardware failures.

## SGI's XFS File System

XFS is another alternative, developed by Silicon Graphics in 1993\. It is a high-speed JFS, with emphasis on parallel **input/output** (**I/O**). The NASA Advanced Supercomputing Division uses this format on their 300+ terabyte Altix storage servers. Metadata operations are somewhat slower than other formats, although this was improved somewhat with the changes made by Red Hat. This is a good choice where metadata changes very little (such as few file or directory creation, move, or delete operations) and I/O performance is of utmost importance.

## Reiser File System

**Reiser File System** (**ReiserFS**) was intended to supplant ext3 as the filesystem of choice for Linux, offering improved performance. At one point, ReiserFS version 3 was the default format choice for SuSE Linux. Version 4 was released, but development waned when the company went out of business, and SuSE eventually decided to go back to ext3 as its default.

ReiserFS offered some advantages over formats existing at the time, but it has fallen behind in some performance areas. It does support dynamic resizing, while other filesystems must be offline in order to be resized, or use a logical volume manager to provide virtual resizing support.

## B-Tree File System

**B-Tree File System** (**Btrfs**) is the next Linux filesystem format. It focuses on fault-tolerance, repair, and easy administration, with the ability to scale up to larger storage configurations. ext-based systems can be easily converted to Btrfs. For the moment, Btrfs is still under heavy development, although only forward-compatible format changes are anticipated. Debian 7 does allow it to be used, but it is not yet recommended for production systems.

## Clustered formats

There are various formats supported for clustered systems, including AFS and GFS2\. In general, they are not used for the basic system files required for booting, but are better suited for shared data. It is possible to set up such systems for booting, but this is beyond the scope of this discussion. If you are interested, there are a number of publications available on Linux clustering. A good starting point might be the Wikipedia article on clustered filesystems at [http://en.wikipedia.org/wiki/Clustered_file_system](http://en.wikipedia.org/wiki/Clustered_file_system).

## Non-Linux formats

The Linux kernel supports many additional formats, such as Microsoft's NTFS, the various FAT formats, the old OS/2 HPFS, and Apple's HFS. These formats do not support the attributes required by a Linux system, and are thus not appropriate for a root filesystem. They could be used for other data should it be necessary. Note that these formats lack the basic Linux security attributes, although there is some provision for translating the attributes that do exist into their approximate Linux equivalents.

## Other Unix formats

Many other formats are available, such as SCO's Unix BFS, QNX, and BSD's UFS. Although Unix-related, they are not considered appropriate for Linux root installations due to slight differences in attribute handling. They may work fine, but the Linux-specific formats generally have better performance and features.

## Choosing a format

Generally, the default ext4 format is the best choice. In specific cases, JFS or XFS may provide some advantages, and if the ability to resize dynamically is more important than performance or scalability, and you don't want to use logical volumes, ReiserFS (especially version 4) might be appropriate. Btrfs should not be used for critical data yet, but at some point soon it will become the preferred format. Non-Linux formats should not be used for the basic system.

# Partitioning

The next decision to be made is how to partition the available storage space. There are the following three main considerations when deciding how to partition storage for a Debian system:

*   Efficient backup and recovery
*   Limiting space
*   Disk management

## Partitioning for backup and recovery

In the past, backups were performed on full partitions. Large partitions could take a long time to back up, and the system could not write to the partition during the process. With the advent of incremental and live backups, this is no longer a primary consideration. Another problem was that when a disk got corrupted, recovery usually was limited to a single partition. There are partition repair utilities now that can fix most problems (though not all), and only those files that can't be fixed need to be recovered.

Still, limiting the damage and the focus of recovery can be useful and remains a valid consideration.

## Space-limiting partitions

Some administrators used partitions to limit the space available for certain directories. A good example is a mail spool directory. A massive spam attack can quickly consume large amounts of disk space. Using a separate partition for the spool directories will limit the total space that can be used by spool files, and the errors generated when no space remains alerts the administrator to the condition.

The availability of account quota systems for Linux can handle this situation without using partitions, but some administrators still prefer the hard limit of partitions.

## Disk management

Aside from backup, recovery, and damage limitation, there are administrative functions that may differ depending on how a disk is partitioned. In particular, using a single partition for an entire disk relieves an administrator from having to modify partition sizes if one partition fills up and more space is necessary. This is frequently why a single disk partition (plus swap space) is the recommendation for new users who are uncertain how they want to partition their drives.

### Note

Early BIOS systems could not boot from locations beyond the first 1024 cylinders of the disk. Thus, at one time, it was necessary to create a small/boot partition below that limit so that the system code (which could access larger areas) could be booted.

## Logical Volume Management

**Logical Volume Management**, or **LVM**, is a format pretty much exclusive to Linux. It is an alternative to partitions which makes space management much easier. Logical volumes can be resized at will, and can span multiple disks. They can be migrated to different disks without interrupting services (live migration). There are also striping and mirroring features that are similar to RAID 0 and RAID 1.

LVM is more complex than basic partitioning, and not commonly used except in large storage installations.

### Note

Technically, LVM is a structure that overlays the physical disk partitioning.

## The swap partition

If available, a swap file or partition is used by Linux when memory paging to disk is necessary. With the advent of cheap memory, such paging is often infrequent with one exception: system hibernation. This is where the system is paused and the memory contents are written to disk prior to power off in order to allow the system to resume from the saved state. While this is commonly associated with laptop systems, servers sometimes make use of it as well.

Swap files are single files created within an existing filesystem, while swap partitions are exactly that—a specially formatted disk partition. In general, swap files are only used when additional swap space is necessary for some reason, as it has all the additional overhead (metadata, journaling, allocation, and such) of the filesystem in which it resides.

Unless an administrator is absolutely certain a system will never need to swap to disk or require the ability to hibernate, a swap partition equal in size to the installed memory is recommended.

### Note

**Solid-state drives** (**SSD**), so-called flash drives, were once considered an exception. In that the limited write cycles were considered a problem if swap files were placed on such a drive. However, with modern flash technology, this is no longer an issue, especially since the swapping has been greatly reduced by the large amounts of memory in current systems.

## Selecting a partitioning scheme

The single partition (plus swap space) per disk scheme is the most common nowadays, as it is simple to create and manage. Multiple partitions may be used in the special cases mentioned previously, although, in general, the quality and speed of current backup utilities minimizes the need for separate partitions just for backup efficiency. If the system has multiple disks and may require resizing or live migration in the event of hardware changes, then LVM should be considered.

In general, the Debian defaults follow best practice. This usually means a single root partition and a single swap partition. If the administrator wants multiple partitions but isn't certain of the sizes required and doesn't want to use LVM, the defaults for the multipartition setup are a good starting point.

### Note

An exception to accepting the single root and single swap partitions default is the case of disk encryption discussed later. If implemented via the Linux kernel, an unencrypted/boot partition is required.

# Encryption

The final choice to be made prior to installation is whether to encrypt the disk contents. There are two main options, disk encryption and directory encryption.

### Tip

In some countries, encryption is subject to legal restrictions. Know the laws in your jurisdiction!

## Why encrypt?

One of the main reasons for encryption is to keep private and sensitive data secure from unauthorized access. Laptops, for example, are frequently stolen and their contents have, in some well-publicized cases, been made public or put to harmful or illegal uses. Servers, on the other hand, aren't usually stolen, but they do have multiple users, and while the Linux permissions system can prevent unauthorized access, there are ways for hackers to bypass it, and they are constantly trying. For example, if one can gain root access, either legally as a system administrator, or illicitly by exploiting unpatched software security vulnerabilities, read/write access to everything on the system is allowed. Or, if a user is not careful with setting permissions, access via other users may be allowed unintentionally. In all cases, access by anyone who does not have the proper keys can be prevented by encryption.

## Disk encryption

Disk encryption comes in several flavors. Full disk encryption, where the entire contents of the storage device are encrypted, is handled by hardware in the disk drive itself, or on the system's motherboard. This is because the code necessary to decrypt the disk can't really reside on the disk, since it will be in encrypted form and thus can't be loaded until decrypted. Since this method depends on the motherboard or disk software, which varies with manufacturer, it won't be covered here.

### Note

A non-hardware method does exist, using an unencrypted USB stick or other media to provide the boot code, but requires special steps for creating the boot media that won't be covered here either.

Partial disk encryption, where individual partitions are encrypted, can be handled by Linux directly. There must be some unencrypted area from which the decryption software and keys can be loaded. This usually means an unencrypted boot partition, or booting from an unencrypted USB stick, thus getting the initial decryption software loaded and then chain loading from the encrypted disk. The keys themselves are encrypted by a password or pass phrase that is required at boot time to keep the keys secure.

A special case would be encrypting the swap partition. If there is any page swapping, or if hibernation (suspend to disk) is used, information in memory can be exposed to anyone who can read the partition. For this reason, many administrators encrypt the swap partition.

Disk encryption is appropriate for laptops, or for separate partitions on servers that contain sensitive data.

## Directory encryption

An alternative to encrypting full partitions or disks is to encrypt portions of a filesystem, usually a directory and everything below it in the hierarchy, by using special features of Linux so that the encryption and decryption are handled automatically by the kernel or special software in the background. Thus, there are no implications for booting (as long as the boot directory isn't encrypted) and no installation issues, as it is configured after installation. Directory encryption is appropriate for servers containing sensitive information that resides in certain parts of the directory hierarchy.

Debian provides several packages for this type of encryption. The two most common are encfs and cryptsetup.

## Choosing encryption

There are two primary disadvantages to disk or directory encryption. The first is probably the most serious; if the password is forgotten, the data is permanently lost and completely unrecoverable unless unencrypted backups (or encrypted backups where the password has not been lost) are available. The second disadvantage is performance. Most software encryption modules perform well, but there is no avoiding some overhead, even if it is minimal. Hardware encryption, such as that provided by the motherboard or the disk controller or drive itself, generally does not have significant overhead.

### Tip

Best practice is to evaluate whether the disadvantages of encryption outweigh the potential damage from compromised or stolen systems.

In general, laptops with sensitive information and systems that contain sensitive information with many users, or that potentially can be accessed by hackers (such as web servers for example, which must be publicly accessible in order to serve their purpose) should use directory encryption at the very least. Disk encryption, especially if implemented in hardware, is even better.

Most corporate policies require partial or full (if available) disk encryption on laptops, and directory encryption as a minimum on public servers.

# Installing Debian

The actual installation is quite straightforward, and is considered one of the simplest distributions for base installs. Boot up the installation disk, and answer the necessary questions. A help button is frequently available to provide additional information during the installation. Also, standard and advanced installation subjects (including much of what is discussed in this chapter) are covered in detail in the Debian installation guide for the current release, available at [http://www.debian.org/releases/stable/installmanual](http://www.debian.org/releases/stable/installmanual).

### Tip

Recommended practice is to install only the base system and use the package manager to install additional software after the system is up and running. However, it is possible during installation to select tasks which will install additional classes of software (such as the desktop environment which installs KDE, GNOME, XFCE, or LXDE, "Laptop" which installs software commonly used on laptops, and so on, all described in the installation guide). Doing this installs a standard set of additional software that suits most users' needs.

# Summary

Prior to installing Debian, or any Linux distribution for that matter, an administrator should know whether he/she will be using single or dual/multiboot, and what his boot firmware is (BIOS or UEFI). It is also good to have some idea of where he/she will place the boot code, what filesystem types he/she will use, and some idea of his partitioning scheme. If unsure, the defaults offered by the Debian installer can be taken safely. If full disk encryption will be used, the setup depends on the hardware implementation and will probably need to be set up prior to installation. Directory encryption can be set up after the installation.

Once your system is set up, the next major issue to address is installing additional packages, which leads us to the next chapter on package management.