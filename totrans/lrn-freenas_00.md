# Chapter 1. All About NAS and FreeNAS

The first chapter is a high level look at Network Attached Storage (NAS), and more specifically, the FreeNAS software. We will cover the basic idea behind NAS and the philosophy of the FreeNAS server. This chapter is less hands-on than the others in this book, but it is important to understand the concepts of Network Attached Storage and where the FreeNAS server fits into your business. The main topics for this chapter include:

*   What is Network Attached Storage?

*   What is FreeNAS?

*   What are the features of FreeNAS?

*   What does FreeNAS do for me and my business?

# Network Attached Storage

In the mid 80s, two popular computer companies independently started to work on ways to access files, over the network, on another computer as if the hard drive of that remote computer was attached to the local machine. These two companies were Sun Microsystems and Microsoft. The Sun Microsystems method, which was for their UNIX operating system, is known as the Network File System (NFS) and was subsequently implemented in almost all versions of the UNIX operating system including Linux. The Microsoft solution (which they actually joint developed with IBM in the initial stages) became known as SMB (Server Message Block) but in later years was renamed as the Common Internet File System (CIFS).

The general functionality of NFS and CIFS are very similar, and with either installed on a networked computer, it can read and write to the file system on another networked computer. Windows users are most used to this concept via the "Network Neighborhood" (Windows 95/98) or "My Network Places" (Windows ME, 2000 and XP) or more recently "Network and Sharing Center" (Vista). Here, you can browse the local network for other PCs and read and write files on that machine as long as the owner shared it with you.

This ability to use a remote computer (a fileserver) to store files led to many companies deploying large centralized NFS Servers or Windows Servers that were accessed by hundreds and maybe thousands of UNIX workstations or PC clients. Users would then be encouraged to store all important files on these servers as the IT staff would back up the servers regularly and so back up the important user files.

Storage space has always been an important aspect of computer systems. Today, more than ever, hard disk space is in demand. Back in the 1960s, storage was measured in bytes (8 binary digits, taking a value of either 0 or 1) and kilobytes. Then, as computers advanced, storage (including hard disks) grew to the size of megabytes (1024 kilobytes) and then gigabytes (1024 megabytes) and today with the 21st century well underway, computer storage is into the realm of terabytes (1024 gigabytes).

With modern needs for video and audio, combined with high speed local networks and the access protocols of CIFS and NFS, a new kind of storage solution has appeared: Network Attached Storage or NAS for short. A NAS server is similar to a traditional file server in many ways, especially in respects to the hardware side of the server. But a NAS server is much more specialized than a traditional office or departmental server in that it only provides access to storage via the network. It is not designed to run other applications such as databases or email servers, which other types of server might. Normally, NAS servers don't require a keyboard, mouse or monitor permanently connected to them and for day-to-day administration, a web interface is used instead.

Here is an example of the FreeNAS web interface:

![Network Attached Storage](img/4688_01_01.jpg)

To access the data on the server, a typical NAS will support multiples access protocols and so allow Microsoft Windows clients, Apple OS X clients, and UNIX (including Linux) clients to connect and use the data on the server.

NAS servers normally contain one or more hard disks, and these hard disks can be combined to create large contiguous areas of storage or used in a way to create redundancy. In a redundancy set-up, if a hard disk fails then the system keeps working and your data isn't lost.

NAS servers come in all shapes and sizes. There are several companies that offer compact NAS servers with an embedded operating system and space for maybe two hard drives. These units are relatively cheap but offer limited room for expansion. At the other end of the scale, are dedicated NAS servers that look more like traditional file servers with good processing power and space for several hard disks (which can make the NAS capable of hosting several terabytes of data).

Here is how a NAS might fit into your network environment:

![Network Attached Storage](img/4688_01_02.jpg)

Network Attached Storage has several advantages over a traditional file server in that:

*   NAS offers better security. As the server is only running a dedicated operating system for providing the access to your data, there aren't other services running (like email servers and general purpose web servers) that can have potential security risks.

*   A NAS server is designed to offer higher availability (less downtime). A NAS server is designed to offer redundancy models for the hard disks and so allowing for hardware failure without losing valuable data.

*   A NAS server is easier to use and administer as most of the configuration is done via a web interface and that interface is designed to perform the specific tasks need to run the NAS.

*   The system requirements for a NAS are modest and bleeding edge processing power isn't need.

*   A NAS server works in a heterogeneous network environment and so allows diverse types of computers to connect and use its storage.

Therefore, NAS has an overall lower cost than a traditional server while allowing for expansion and increasing availability and security.

This book focuses on one implementation of NAS called FreeNAS (Free Network Attached Storage), which will turn a normal PC or server into a NAS.

# What is FreeNAS?

FreeNAS is free piece of software that turns a PC into a NAS server. It supports connections from Microsoft Windows, Apple OS X, Linux, and FreeBSD. It supports hard disk redundancy, has a simple web administration interface, and modest system requirements.

FreeNAS is what is known as an embedded operating system. This means it is compact, efficient, and dedicated to just one task, in this case, NAS. Once FreeNAS is installed on a PC, the PC becomes a dedicated NAS, it can't do other general tasks at the same time.

To use FreeNAS, you need to download a copy of the software from [http://www.freenas.org](http://www.freenas.org) and boot it on the computer you want to make a NAS. We shall look at this in more detail in the next chapter.

FreeNAS comes in two variations; the live CD and an installable kit. The live CD boots the machine as a NAS and uses a floppy disk or USB flash drive to store the configuration information. The installable version installs itself on to the server (much like a traditional operating system would) and uses the system hard drive to store the configuration data.

So why is FreeNAS free? FreeNAS is what is known as Open Source Software. It was originally written by Olivier Cochard-Labbé and is now maintained by a small international team with Oliver as the project leader. Being open source means that the FreeNAS team have licensed the software in such a way that they give unrestricted access to the software and to its source code. You are free to use and deploy FreeNAS without any restrictions. You can also obtain the source code and build or modify the software for yourself. The only restriction is that when redistributing FreeNAS, with or without modifications, the original copyright notices must remain intact. Olivier Cochard-Labbé is the copyright holder. He also holds the trademark for the name FreeNAS.

FreeNAS is made up of several different components. At the lowest level, there is the operating system (FreeBSD, see below). Then, there are various server components that provide the network services and finally, a web administration interface.

### Note

Several times throughout this book, we will refer to the operating system FreeBSD. FreeBSD is a UNIX like operating system with lineage back to the original AT&T version of UNIX through the Berkeley Software Distribution (BSD) branch. FreeNAS is built on top of and relies on FreeBSD. Because of the high level of synergy between FreeNAS and FreeBSD, chapters 9 and 10 have been written to help you in troubleshooting problems on your FreeNAS and will deal with low level commands for the FreeBSD operating system.

# Features

The capabilities of the FreeNAS software are quite impressive and the feature list is growing with every release while maintaining the goal of providing a simple NAS server. So what can the FreeNAS do?

FreeNAS installs on either a hard drive or USB flash drive and takes less than 32MB of disk space once installed.

There is support for Microsoft Windows machines using the Common Internet File System (CIFS) protocol. This is Microsoft's protocol for accessing files over the network. CIFS is also supported in the Linux operating system as well as in Apple's OS X. This means that Linux and Macintosh computers will also be able to access the NAS. With CIFS, areas of the NAS can be permanently mounted on the client machine as if they were local hard drives.

FreeNAS includes support for the Network File System (NFS). NFS is a mature network file access protocol that is most often used in UNIX-type environments. With NFS, storage areas on the NAS server can be used as if they were local disks on the client.

The File Transfer Protocol (FTP) is supported. FTP is a mature protocol for transferring files over a network. FTP is a client/server protocol and is most often used to transfer files from one machine to another in a "one off" sense. A connection is made, the files are transferred, and the connection is closed. This is a protocol that is often used to allow people to download files from the Internet. In the NAS context, it is very useful for offering a repository of software on your network (like software, company templates, and anti-virus software updates).

The FreeNAS server can be used as a backup server via different utilities like Unison and RSYNC (Remote Synchronization). With RSYNC, an entire disk or folder (and its sub-folders) can be synchronized with the backup server in an efficient manner. The advantage of RSYNC over a straight copy of the files over the network is that RSYNC only copies the portions of the files that have been changed.

FreeNAS also supports Secure Shell (SSH) for encrypted connections and data exchange, the Apple Filing Protocol (AFP) that offers file access services for Mac OS X and Classic Mac OS, and UNISON (another file synchronization protocol).

For offering both access to storage on the NAS, and also for extending the storage capabilities of your server, FreeNAS supports the Internet Small Computer System Interface (iSCSI). iSCSI simulates the presence of a local SCSI hard drive over your IP network. FreeNAS can act as an iSCSI server (exporting RAW local storage via the SCSI over IP protocol), which is technically known as being an iSCSI target. FreeNAS can also act as a client and connect to other iSCSI targets and mount iSCSI disks. iSCSI disks that are mounted locally become part of the general storage resources for the server; and the FreeNAS server can act as a gateway or head to those disks allowing other machines (like a Windows PC) to use the iSCSI disk over CIFS or FTP etc. This type of setup is very popular in the Storage Area Network (SAN) model.

FreeNAS supports several different file systems. A file system is the way files are stored and organized on the disk. Different methods of organizing files have different characteristics like speed, maximum file size, and recovery after a system crash. FreeNAS can use the UNIX File System (UFS) that is the FreeNAS default. It can use the Linux file systems ext2 and ext3 as well as the NT File System (NTFS) that is the native file system for Microsoft Windows NT/2000/XP, Windows Server 2003/2008, and Vista.

Many types of hard drives are supported, including all the popular hard drives of today (SATA/PATA, SCSI, iSCSI, USB, and Firewire). FreeNAS also handles hard drives larger than 2 Terabytes where the file system permits.

All of the popular network cards (both wired and wireless), which are supported by FreeBSD, work with FreeNAS without needing to download and install additional drivers.

FreeNAS includes hard disk fail-over and mirroring technology. Using a system called RAID (Redundant Array of Inexpensive Disks), you can configure disks into sets which work in combination to spread the data over 2 or more disks so that if a disk fails data integrity remains and your NAS continues to work. FreeNAS can use hardware RAID (where the controller card is responsible for controlling all the disks) or software RAID where the FreeBSD operating system runs the RAID sets. FreeNAS supports many popular hardware RAID cards via the drivers supplied with FreeBSD.

# What Does FreeNAS Do for Me and My Business?

Network Attached Storage is a solution to a problem. To discover what a NAS and more specifically FreeNAS can do for you, we need to first look at the nature of the problem.

The problem, put simply, is the need for highly accessible storage space. Storage space demands are increasing for three distinct but yet important reasons. First, the volume of digital information created in a business is increasing. Secondly, the size of this digital information is growing, and finally, the need for comprehensive archiving and management of older data is growing, especially in countries where there are legal requirements to store data for long periods of time.

The volume of digital information created is increasing because more and more business data is being stored on computers, and businesses have become less reliant on paper. Take email for an example. 10 to 15 years ago email probably wasn't a critical part of your business. It probably existed and was used but it was not yet critical. Today, on the other hand, email has become mission critical. Online email services like Yahoo! ,Gmail, and Hotmail started by offering mailboxes with 5 or 10 megabytes, today they are offering mailboxes in gigabytes. An organization of 100 people will generate gigabytes of email data each year. If your business grows and you employ more people, then the amount of email will increase with it. As the number of your customers increases, your email data will increase and so on.

Then, if you factor in documents, contracts, accounting, inventories, presentations, sales material, and so on, you can see that the volume of data is growing. Each new sales lead, each new customer, and each support contract increases the data generated.

Secondly, the nature of digital information is changing and the size of the data is changing with it. In the years gone by, data was more text orientated but today data has gone multimedia. Photos, video, and music are normal data types today. The big difference is of course that text type data (including simple word processor documents) is small but multimedia files are much larger. This is reflected in the type of optical disks available. The humble CD-ROM was 650 megabytes which was enough for 72 minutes of music. Then came the DVD which was six times bigger at 4.7 Gigabytes (for a single layer disk). Now, there are high density disks like Blu-ray, a dual layer Blu-ray disc can store 50 GB, almost six times the size of a dual layer DVD at 8.5 GB. In parallel with this, hard disks are growing to terabyte sizes.

Thirdly, how your business manages old data is becoming more and more important. Governments around the world have passed legislation requiring businesses to keep data for longer periods of time. The value of data constantly fluctuates. What may seem to be just an old collection of files from an ex-employee can overnight become the hottest files in your system as you discover that important information is in those files. The value of old emails also fluctuates. What you consider an old email today might become important tomorrow if the lawyers want to see it due to some legal matters. Keeping your old data safe and accessible (by copying it to a FreeNAS server in conjunction with other backup methods) is critical. But by its very nature, the volume of old data will increase as every day passes and so will your need for storage.

### Note

**When FreeNAS Isn't the Right Solution**

FreeNAS won't be the best solution in all circumstances. The most obvious of which is if you only have a very small number of users on your network, say less than 3\. In such scenarios, direct attached storage (i.e. an external USB hard drive or adding another hard drive to an existing PC on the network) is an alternative solution.

Along with the increasing demands for storage space, there is the need to have your data accessible by all different types of computer (Windows, UNIX, OS X) on your network and also for it to be available in a consistent manner without long periods of down time.

Traditionally, Windows desktop machines worked with Windows servers and UNIX desktops worked with UNIX servers, but less often did Windows and UNIX work with each other. Although many of the network protocols are available for both Windows and UNIX, the two different systems tended to be used in isolation. The Windows servers had their system managers and the UNIX machines had theirs.

To lower costs and increase availability, Network Attached Storage needs to provide access to the data from both Windows PCs and from UNIX clients. Having a single type of server that services all the clients on your network is an important aspect of managing your data.

## How FreeNAS Meet These Needs

To solve this need for more storage space, you can use a NAS. But to be of value to your business, the NAS needs to be easy to use, easy to install and deploy, easy to manage, scalable, and without prohibitive licensing costs.

**Ease of use—** Since FreeNAS is designed to do one thing and one thing only, namely convert a PC or server into Network Attached Storage, it is simple to use and manage. From the user's points of view, it will just appear as storage. If used on the Windows platform, it will just be an extra hard drive or for the Intranet it will be an FTP repository. Most of the system administration is done via a web interface and the machine can even be rebooted via the web interface.

**Easy to install and deploy—** Getting FreeNAS up and running is very simple, especially when using the live CD with a USB flash drive. Just pop the CD in the drive and boot up. In the next chapter, we will do a quick install to help you become familiar with FreeNAS and its web interface.

**Easy to manage—** As mentioned in the 'ease of use' paragraph above, FreeNAS is dedicated NAS software. Unlike a full installation of a tradition server operating system, there is nothing to worry about except for the storage configuration. There are no overly complicated system services to configure with 101 different options. But don't be misled, FreeNAS is feature-rich and a very clever piece of software, but the management interface is simple to use while yet remaining comprehensive.

**Scalability—** Scalability and robustness are built-into FreeNAS as the core of its functionality comes from the FreeBSD operating system. FreeBSD has proved to be a mature and production-ready operating system. There have been many successful deployments of FreeBSD in small, medium, and large businesses and even companies like Yahoo! have relied on FreeBSD for their servers at one time or another. FreeBSD is known for its network performance and reliability which is why it was chosen for FreeNAS. The real limit to scalability will be the physical number of hard drives that can be fitted to the server. But beyond the physical drive limit, FreeNAS can use iSCSI devices on your network and so the server can scale even further.

### Note

If you installed FreeNAS on a USB flash drive or use the live CD version, the majority of your disk space will be used for storage, which is of course the aim of a NAS. If you used a traditional operating system, several gigabytes of hard disk space could well be used up just in the initial installation.

**Licensing costs—With** a traditional server operating system, there are normally licensing fees to pay to the operating system manufacturer. Sometimes, this includes a "per seat" license fee that is based on the number of users in your organization. Since FreeNAS is open source, there are **no licensing fees**. This means you are free to use and deploy FreeNAS as much as you want.

## Practical Uses for the FreeNAS Server

So what practical uses are there for the FreeNAS server? Here is a brief summary of some of the uses of a FreeNAS server:

*   As a common storage area for resources in your business including sales material, presentations, leaflets, brochures, templates, software kits, software updates, popular office software, and popular Internet software.

*   To provide each user with a folder on the network which only he/she has access to, so they can save all their work on the server and not on their local PC. This is sometimes referred to as "the network drive" meaning a hard drive accessible via the network. This has two clear advantages:

    *   Backups are much easier as all the data is stored in a central place (i.e. the FreeNAS server).

    *   Users aren't reliant on the PC that they use as all their data is in a central place and also the hard drives of each user's PC doesn't need to be too large and can also be wiped (for an operating system re-install perhaps) without fear at any time.

*   As a central storage for large files that would quickly fill a desktop PC hard disk. If your business involves the use of large files like multimedia files (video, audio, and photographs) then a FreeNAS server can provide a convenient central storage point for these files that would otherwise overwhelm the hard drive on a desktop PC.

*   As an FTP repository for your Intranet (a private, internal network which uses the same protocols as the public Internet). Assuming you have an Intranet, there are most likely common resources like presentation templates that you make available to all your employees. If your web server doesn't have enough disk space (as many don't because they are tuned for performance rather than storage) then linking to the FreeNAS server via the FTP protocol would take the burden off your web server.

*   As a backup server. You can use your FreeNAS server just as a backup server. Either with the Microsoft Windows file sharing protocol (CIFS) or with RSYNC, each PC on your network can be configured to copy user data from the local PC to the FreeNAS server. Alternatively, if you are already using a general purpose server on your network, the FreeNAS server can act as a backup store for that server.

*   As an iSCSI head/gateway. If you have invested in SCSI over IP technology, then FreeNAS is a good way to offer access to your iSCSI devices over multiple protocols. iSCSI devices only use one protocol, SCSI over IP, and they don't understand other protocols like the Windows file sharing protocol (CIFS) or FTP or NFS and so on. But FreeNAS understands these protocols as well as SCSI over IP. Therefore, it can act as an intermediary (a gateway) between your iSCSI devices and the computers on your network.

## Consolidation

There are two aspects to consolidation (the act of reducing many servers into one) that FreeNAS can achieve. The first is consolidation based on access protocol. As mentioned previously, Windows servers and UNIX servers use different protocols for accessing their stored data. FreeNAS supports multiple access protocols including CIFS (the Windows protocol) and NFS (the UNIX protocol). A UNIX file server and a Windows file server can be consolidated into a single FreeNAS server that supports clients from both types of system. The second type of consolidation is data consolidation. If you create a FreeNAS server with high initial storage space, you can replace existing general purpose servers into one FreeNAS server. Previously, each general purpose server needed its own management (often specialized) but with just one FreeNAS server, your management workload is reduced.

# Summary

This chapter has been an introduction to the concepts of Network Attached Storage and FreeNAS server. We looked at the features of the FreeNAS server and what FreeNAS can do for your business. Finally, we looked at some of the practical uses of the FreeNAS server. In the next chapter, we will look at how to plan for FreeNAS in your environment including looking at the network implications and planning for disk redundancy.