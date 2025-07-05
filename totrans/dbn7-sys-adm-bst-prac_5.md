# Chapter 5. System Management

Aside from configuring individual software packages, an administrator is responsible for managing how the various services on his systems are started and stopped, managing network connections, maintaining the filesystem, managing system logs, and configuring the face the system shows to the users.

# Startup and shutdown

The proper startup and shutdown of services required for a system to function and fulfill its purpose is central to its management. While Unix `init` scripts (also known as System V or SysV scripts, due to their origin in Unix System V) have a long history and are in one form or another, common to all Unix and Linux systems, the way in which they are managed, sequenced, enabled, disabled, and the preferred script format often differs somewhat between distributions. The primary areas to be aware of for Debian startup and shutdown scripts include the purpose of run levels, dependency-based sequencing, and utilities available for administering the boot sequence.

## Debian run levels

In Debian, as in nearly all Unix/Linux operating systems, run levels from 0 through 6 are available, defined as follows:

*   0: System Halt
*   1: Single User (maintenance)
*   2 to 5: Multi-User Modes
*   6: System Reboot

Note that run levels 2 through 5 are identical in Debian. This is unlike some other distributions, such as RedHat, Fedora, SuSE, or OpenSuSE, which give specific purposes to some of these run levels. For example, run level 2 in these distributions is often defined as one without network support, 3 with networking, 4 with file sharing, and 5 includes a display manager which isn't active in the other run levels.

In most Debian systems, there is no difference between the multiuser run levels, and all of the `init` scripts default to active in levels 2 through 5\. This doesn't mean you can't define your own purposes for different run levels. However, if you do choose to do this, do not manually edit the various links to `init` scripts in the run level directories. The `update-rc.d` command should be used instead. The reason for this is that Debian now defaults to dependency based boot sequencing.

## Dependency-based boot sequence

As mentioned previously, this is now the default as of Debian 7 Wheezy. It was introduced in Debian 6 Squeeze, although it could be turned off. It is now always enabled, although provisions are made for legacy ordering (assigning specific numbers to start and stop scripts). Because of this, the administrator no longer needs to determine the order in which the `init` scripts are run. This is now handled by the `insserv` utility.

### Tip

The `insserv` utility should not be called directly. The `update-rc.d` utility provided by Debian, which calls the low-level `insserv` command, is the recommended interface to manage `init` scripts.

The `init` scripts must now have dependencies and defaults listed in a special set of headers, along with a description of the script, what service or services it provides, and what run levels the service should be active in. A good example is the beginning of the script for starting Apache:

```
#!/bin/sh
### BEGIN INIT INFO
# Provides:          apache2
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# X-Interactive:     true
# Short-Description: Start/stop apache2 web server
### END INIT INFO

```

The fields are fairly self-explanatory. This script provides the apache2 service. Other scripts can name this service as a pre-requisite. This script requires that local and remote filesystems be mounted, and that the network be up, and that `syslog` and named services be available prior to starting. Likewise, these services must not be stopped after this script has shut down the apache2 service. The default runlevels where apache2 should be active are 2 through 5, and of course, 0, 1, and 6 is where it is stopped. The X-Interactive field means that the script can require user input if it is run in such a way that a terminal is available to communicate with the script. There are other fields available as well, which are documented in the `insserv` manual page.

### Note

These headers are called **LSB** headers, since they are defined in the **Linux Standard Base** document, developed jointly by a number of Linux distributions under the organizational structure of the Linux Foundation.

While dependencies generally do not change, the administrator can modify what run levels a script is active in. This should not be done by editing the headers. Rather, `update-rc.d` should be used to modify the run levels. For example, suppose you want apache to run only in run levels 4 and 5, and not 2 or 3, the command `update-rc.d apache2 disable 2 3` will do this.

### Tip

Many scripts provide a switch in their `/etc/default` config file that defines whether the service should run at all. When you want to disable a service completely, this switch should be used in preference to disabling the script in all run levels via `update-rc.d`.

The manual page for `update-rc.d` also documents the options `start` and `stop`, as well as a means of specifying the start or stop order of a script (using the legacy method of assigning numbers to the start and stop links). However, these are deprecated and it appears they will be removed in Jessie (Debian 8).

### Note

If you have locally developed or third-party `init` scripts that do not include the necessary headers, Debian 7 will still boot using the old method of script ordering, but will notify you of the reasons it can't migrate to a dependency-based boot sequence.

## Managing SysV scripts

The `update-rc.d` utility has already been mentioned, and is one of the primary command line interfaces for managing `init` scripts. However, there are several other utilities that are essentially a frontend for `update-rc.d` that make the administrator's job a little easier. The primary ones are `bum` and `sysv-rc-conf`.

The Boot Up Manager, or `bum`, is a graphical application for managing `init` scripts. It requires a window manager to run, and provides a nice interface showing what services are running and what scripts are enabled. In advanced mode, it will also allow you to adjust individual run levels and run order.

### Tip

As mentioned for the `update-rc.d` start and stop commands, adjusting script ordering is not recommended.

There is also a utility called `sys-rc-conf`. It uses the curses library to provide a full-screen text interface. In its default mode, it will not modify script order, although special options on the command line will allow this if you really need this functionality.

Both utilities are pretty much self-explanatory. Check the services you want, uncheck those you don't, and you can set or unset check marks for different run levels on the same script.

### Tip

Frequently, you will need to execute a SysV script manually, either to check the status, or to restart a service that requires it. Although the script in `/etc/init.d` may be executed manually, the recommended method is to use the `invoke-rc.d` command, which ensures that system policy and run level constraints are satisfied.

## Third-party and local scripts

Non-Debian third-party packages often do not provide SysV scripts to start and stop their software's background processes, and you may need to write your own. Even if such scripts are provided, they may need to be modified to follow the Debian standards, particularly if they use prepackaged functions available in other distributions that differ from those in Debian.

Writing `init` scripts is a whole subject in itself. However, the Debian `initscripts` package includes a `/etc/init.d/skeleton` script that can be copied and modified according to your needs. The requirements for such scripts are defined in *Chapter 9*, *The Operating System* of the *Debian Policy Manual* (available as a Debian package and at [http://www.debian.org/doc/debian-policy/](http://www.debian.org/doc/debian-policy/)) and *Chapter 20*, *System Initialization* of the *Core Linux System Base standard* (available at [http://refspecs.linuxfoundation.org/lsb.shtml](http://refspecs.linuxfoundation.org/lsb.shtml)). The latter also provides for some standard functions in `/lib/lsb/init-functions` to assist in script coding.

## Network administration

Basic to any system is network access, either to allow others to use its services, or to allow users to access services on other systems. There are two main ways of setting up and controlling networking, the static `/etc/network/interfaces` file, and the more dynamic Network Manager.

## The interfaces file

This is the traditional method for setting up networking on a Debian system. It involves a series of files in `/etc/network`. RPM-based systems such as RedHat Fedora and SuSE Linux use a different layout in `/etc/sysconfig/network` that is managed by their own utilities.

The `/etc/network/interfaces` file is probably the simplest way to get a network up and running. While it must be edited manually, it is easy to understand and a minimum number of configuration lines are needed to handle most situations. In fact, the Debian installation process will set up this file for you. If you use the same networking configuration as you used for installation, the network will work immediately and require minimal tweaking. In fact, a minimal interfaces file is usually sufficient for most servers that aren't part of a cluster. Even if you do use an alternate method for network configuration, such as Network Manager, the local loopback interface is usually left configured in the interfaces file to keep the alternate configuration uncluttered, since it requires only basic configuration and doesn't normally need to be modified.

The interfaces file, while it can be very simple, also offers many options for more complex setups. You can configure wired, wireless, **VLAN** (**Virtual Local Area Network**) and Bridge interfaces, IP tunnels, and **Point to Point** (**PPP**) interfaces. Each interface can be configured to use DHCP or a static IP address, with both IPv4 and IPv6 supported as well as Novell's IPX protocol. A simple interfaces file is shown as follows:

```
# This file describes the network interfaces available on your
# system and how to activate them. For more information, see
# interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug eth0
iface eth0 inet static
 address 192.168.3.52
 netmask 255.255.255.0
 gateway 192.168.3.1

```

Lines beginning with `#` are comments, of course. The purpose of other lines is as follows:

*   `auto lo`: The `lo` interface is brought up whenever `ifup` is run with the `-a` option, as it is during system initialization
*   `iface lo inet loopback`: This defines the basic loopback interface ; the address is always `127.0.0.1` or `::1` for IPv6
*   `allow-hotplug eth0`: This brings up Ethernet 0 if it is available and plugged in
*   `iface eth0 inet static`: This defines Ethernet 0 as an interface with a static IP address
*   `address`, `netmask`, and `gateway`: These define Ethernet 0's address, net mask, and the default IP gateway, respectively.

Simple and to the point. This file was actually set up during Debian installation and works as is. Of course, there are many other options available which are documented in the interfaces manual page that allow you to handle much more complex configurations. The details on setting up the interfaces file may be found in the interfaces manual page. It includes information on setting up IPv6 and many other protocols.

In addition to the interfaces file, there are a number of scripts in the `/etc/network` hierarchy that are related. In particular, the subdirectories `if-pre-up.d`, `if-up.d`, `if-down.d`, and `if-post-down.d` contain scripts that are run automatically when interfaces are brought up or down. In addition, specific scripts can be identified in the interfaces file to be executed when a particular interface is brought up or down (this is especially useful in configuring Bridges). The major disadvantage to using the `/etc/network/interfaces` file is that there is no provision to configure **Virtual Private Networks** (**VPN**). However, in general, these can be configured using command line tools.

### Note

Additional details and examples may be found in [Chapter 5](ch05.html "Chapter 5. System Management"), *Network Setup* of the Debian Reference Manual ([http://www.debian.org/doc/manuals/debian-reference/ch05.en.html](http://www.debian.org/doc/manuals/debian-reference/ch05.en.html)).

## Network Manager

As flexible as the `/etc/network/interfaces` configuration is, many administrators prefer a more graphical interface to network configuration that may be used in a more dynamic network environment. Network Manager is most often used to manage wireless connections. It consists of a background process that does the actual connection management and has both a command line and a graphical utility that allows you to configure and control the managed connections.

The graphical utility displays available access points and provides a menu and an easy way to configure the connection. It can also be used for wired connections and can manage VPN connections to a private network as well. The main disadvantage of Network Manager is that it does not handle bridging, VLANs, or the IPX protocol. Of course, command line tools can be used to supplement Network Manager in order to configure these options, or the interfaces file can be used to manually configure them while Network Manager handles the rest.

There are other packages that provide a GUI interface for network configuration. One of the main ones is `wicd`. Some users prefer it as it handles wireless connections in a different manner that may allow certain wireless cards to work better, but it has fewer features.

## Combining methods

It is possible to combine both the interfaces file and Network Manager methods of network configuration, with each responsible for a portion of the configuration. This technique might be used when certain features are required that only one of the methods support. For example, Network Manager doesn't handle network bridging, and the interfaces file generally can't be used to configure VPN connections. If you needed both, you would set up bridging using the interfaces file and configure your VPN connection using Network Manager.

For those unfamiliar with network bridging, a good example would be a development system that runs one or more virtual machines that require direct access to the network (meaning its connection must behave as if it is an actual interface with a direct network connection). The basic setup looks like the following diagram:

![Combining methods](img/3118OS_05_01.jpg)

Note that the host machine now uses **br0** instead of **eth0** as its primary interface. The bridge interface uses eth0 to connect to the actual network. The **VM** (**Virtual Machine**) will set up its own network interface (or you will set it up). These are generally set up as a tap or tunnel interface (hence the name **tap0**).

To set it up, the basic interfaces file might look like the following listing (assuming the hardware Ethernet card is eth0):

```
# TAP setup
auto tap0
iface tap0 inet manual
 pre-up /usr/sbin/tunctl -t tap0

# Bridge setup
auto br0
iface br0 inet dhcp
 bridge_ports eth0 tap0

```

This sets up a TAP device using the `tunctl` command to create the device. It must appear before the bridge or it won't be there when the bridge device is initialized and attempts to connect it to the bridge. The bridge setup creates the bridge interface and uses DHCP to obtain the IP address. The `bridge_ports` line indicates that it bridges traffic to the real interface (eth0) and the tap0 interface, which will be used by the VM. I've left out the loopback entry for clarity. In this example, VPN and wireless connections are left for Network Manager to handle.

More about network bridging may be found in the `bridge-utils` package, and network tap and tunnel interfaces are covered by the `uml-utilities` package.

## Which method?

Network Manager is automatically installed with the GNOME Window Manager. It is especially useful in a laptop environment. For servers, the interfaces file is probably the best and most flexible option. Of course, if you require certain features available only in one or the other, use whatever provides them, combining the two techniques if necessary.

### Tip

Don't attempt to control the same interface using both methods. While Network Manager will refuse to manage a connection defined in the interfaces file, and the `ifup` command will generate an error if it attempts to set up an interface already controlled by Network Manager, it is possible to circumvent these protections.

# Filesystem maintenance

There are two types of filesystem maintenance: partition and content maintenance. The former includes periodic checking of the filesystem's underlying structure and metadata, modifying the partition layout, and low-level backups. The latter involves monitoring and controlling the space used by files as well as file backup and recovery.

## Partition maintenance

Although modern journaling filesystems are quite resilient, they will, on occasion, suffer an error in the underlying infrastructure. The reasons are many and include power fluctuations, hardware failures, and certain types of kernel failures. While the last is extremely rare in Debian stable releases, it does happen, especially if third-party kernel modules that are not part of Debian are installed or if the kernel has been modified locally for some reason.

### Filesystem Check (FSCK)

Maintenance involves running the **Filesystem Check program** (**FSCK**). If the system is rebooted frequently, as is the case with laptops or workstations, or the partition is unmounted and mounted frequently, this will normally happen automatically. EXT3 and EXT4 filesystems default to every 39 mounts when originally created unless otherwise requested. This may be changed using the `tune2fs` utility which can also set a time-dependent check interval instead of mount count dependent.

### Tip

Although it is possible, periodic checking should not be disabled. Journaling filesystems in general are always marked clean and so Linux will not recognize when such a system may have problems.

If your system is always up and the partition in question is never unmounted, you may want to arrange for a periodic reboot of the system, either via the `shutdown -rF` command that forces a filesystem check, or by using `tune2fs` to set a time-dependent check interval that is less than the reboot interval. Of course, if problems are found, you must have access to the physical console in order to answer the questions about how to fix them.

An alternative is to never check the filesystem except when problems are obvious or likely. Normally, after a failure that could potentially cause filesystem infrastructure errors, the system will need to be rebooted anyway, if it isn't forced. This is the time to run the FSCK manually in maintenance (single user) mode, or to force the check by having a short, time-dependent check interval set. Again, if problems arise, someone will need access to the physical console in order to answer questions.

### Note

To speed up periodic filesystem checks, use EXT4 instead of EXT3\. In most circumstances, checks are anywhere from 2 to 20 times faster for EXT4.

### Partition resizing

Partition resizing may be required if you need more space. To get a quick overview of your disk space usage, use the `df` command. There is also a disk usage (`du`) command that summarizes file and directory space. Refer to the manual pages for details.

If you must resize a disk partition, the procedures are fairly straightforward but there are some considerations.

### Tip

There is always a risk in modifying your partition layout. Make sure you have current backups prior to modifying the layout.

If the partition is being expanded, there must be room to expand it. If you are using **Logical Volume Manager** (**LVM**), this can be as simple as adding a physical volume to the volume group if there is not enough space left in it. If you are not using LVM, or if you have multiple partitions configured on the same logical volume, then there must be free space available on the physical disk or logical volume following the end of the partition to be expanded. If there isn't, it must be created somehow. If the partition is to be shrunk, space is not an issue, although you may want to move or expand other partitions to use the space freed up by shrinking. Also, it is sometimes necessary to move partitions around. This is not necessary if you use LVM and assign one partition per logical volume.

Moving or shrinking a partition may only be done with unmounted (offline) partitions, since it requires moving files and risks corruption if any files are in use. Some filesystems, EXT3 and EXT4 in particular, may be expanded while the partition is still mounted, however, since this doesn't require moving any files, just modifying the filesystem metadata.

While each filesystem architecture has appropriate tools for resizing and there are simple tools for managing the partition table, the primary tool for managing partitions is `parted`. There are other, mostly commercial tools, but `parted` does pretty much everything that is necessary, is available as an easy to use graphical utility (`gparted`), and it is free. There is even a live CD version you can boot to manage your partitions regardless of what operating system is installed. Many Windows administrators use it quite successfully. It handles both enlarging and shrinking partitions as well as moving them, and will handle many different filesystem architectures, and ensures that the partition table matches the filesystem sizes.

### Tip

Unfortunately, neither `parted` nor `gparted` will not allow you to expand a mounted partition. If you must expand a partition while it is in use, you must delete and recreate the partition with the same, identical beginning position, force a reread of the partition table, and then use the appropriate filesystem resizing commands. This is extremely dangerous and should never be used for resizing the root partition.

### Note

The procedures for modifying partition layout are straightforward. Note that if you aren't booted from a `gparted` live CD or similar utility disk, you cannot unmount partitions required to run the necessary utilities and operating system services.

To manually modify your partition layout:

1.  Ensure there is adequate space for the expanded partition, or to hold the partition being moved. If necessary, shrink or move other partitions or expand the logical volume.
2.  You must ensure the partition table matches the resized or moved partition exactly. Failing to do this can result in corrupted or lost data. `gparted` will handle this for you automatically.
3.  To shrink a partition, it must be offline (unmounted). Resize the filesystem first, then modify the partition table to match the new size.
4.  To expand a partition, modify the partition table first, then expand the filesystem. If you are doing this with a mounted partition, you must use the procedure outlined previously.
5.  To move a partition, you must first create a new partition of the same size as the original. Then you must unmount the current partition, copy the data to the new partition exactly, and delete the old partition.

In all cases, the partition must be clean prior to modifying or moving it, and should be checked afterwards prior to mounting it. The `gparted` utility or similar will handle the steps above automatically, including the necessary filesystem checks, data movement, and partition table modifications.

Note that either `gparted` or the manual procedures may be used on a live system, provided the partitions being modified can be unmounted, or if a mounted partition must be modified, it is only being expanded and `gparted` is not being used.

### Tip

Unless you must expand a partition while it remains mounted and in use, the recommended procedure is to boot from a `gparted` live CD. This greatly simplifies the whole process, allows modification of all partitions, not just those that aren't required for operating system services and necessary utilities, and reduces the risk of corruption.

## Backups

In general, there are two types of backups: low-level image backups, and file-level backups.

### Low-level backups

Low-level disk image or so-called Bare Metal backups, are byte-level copies of a full partition or even a complete logical or physical disk. Some utilities will only include used sectors in the copy, thus reducing the size and speed of the backup and restore operations, but this is not always the case. When restoring such a backup, some sort of media, such as a live CD, is required in order to boot up the software to restore the image.

Full disk byte-level backups are useful if you need to duplicate or restore a system quickly, as long as the disk being restored is the same or larger. Partition backups of this type may also be done, but in order to restore them you will need some way to restore the partition information to the disk partition table.

Low-level backups are useful in two situations:

*   When a system must be replaced and restored quickly with identical hardware
*   When a system needs to be duplicated many times on identical hardware

A good example of the latter is when a company provides workers with preconfigured systems all with identical or nearly identical initial configurations. In this case, it is easy just to restore the disk from a standardized copy whenever a new system is required or an old one needs to be returned to its original condition for a new employee.

### File-level backups

File-level backups can be as simple as a file-by-file copy of everything in your directory hierarchy, to a backup that takes into account the filesystem metadata structures and that doesn't store duplicate data (commonly called **data de-duplication**). Restoration generally requires a minimal installed system, or a live CD. File-level backups do not have the ability to restore boot sectors or partition tables, so these must already exist or be created prior to restoring your files. Many file-level backup utilities provide the ability to back up only files that have changed since a previous backup.

File-level backups are most useful when individual files or directories are lost or corrupted, or need to be reverted to an earlier version, as only the files necessary need to be restored. This is not possible with low-level image backups.

### Backup utilities

There are many backup utilities available and they vary from simple ones appropriate for individual systems, to complex backup suites appropriate for managing backups for multiple production systems and clusters.

The simpler utilities generally provide file-level backups that are written to external media or even a remote network location. They frequently provide options for incremental backups, where only changed files are copied after the first, full backup. Mostly, these are command line utilities which can be scripted and executed periodically via a CRON job, although there are graphical front ends available. The disadvantage of most of these are that in the event you need to restore after a complete disk failure, you must have some other means of restoring the partition information correctly, and of recreating the boot sector information properly so the system will be bootable.

Among the more common of the simple utility commands are `rsync` and `tar`. Frequently, these utilities are used in the more complex backup software to actually store the data. If you use one of the EXT filesystems, the `dump` and `restore` commands are of particular interest, because they understand and take into account the filesystem's metadata and are thus faster and more efficient than the simpler copy utilities. The disadvantage, of course, is that restores may only be done to an equivalent EXT filesystem. Other utilities such as `rsync` can restore to a completely different filesystem type, although certain metadata, such as file ownership and permissions, may be lost if the type is too different, such as backing up from an EXT4 filesystem and restoring to an NTFS partition.

More complex backup software is sometimes capable of so-called Bare Metal backups. These combine small, low-level backups of (or at least the ability to recreate) the non-filesystem structures such as the partition tables and boot sectors, with file-level backups of the filesystem contents. Many are multisystem backup solutions that can be administered from a central location and which can store the backups on various media in various locations. Usually, these multisystem suites include the ability to define and control backup schedules, contents, and locations as well as provide for off-site archiving.

### Choosing your solution

Which backup software you choose is somewhat a matter of taste, but there are some guidelines. For quick and dirty backups of individual files, the standard `rsync`, `tar`, and `cpio` utilities are usually sufficient. For more routine backup of individual systems, one of the software packages that provide for configuring and scheduling automatic backups, both full and incremental, is your best bet. If you have multiple systems to back up, one of the major backup suites that provide central control, scheduling, and storage is best. We will discuss this further in [Chapter 7](ch07.html "Chapter 7. Advanced System Management"), *Advanced System Management*.

In all cases, if your backup solution doesn't automatically take care of it, be sure to include a rescue or live CD that includes the software necessary to format and partition the disk drive, restore the files, and recreate the boot information. Some backup packages, such as `Mondo`, include the creation of such bootable media as part of their software.

### Tip

Don't neglect backups. Even the most resilient VM environment with multipath **Network Addressable Storage** (**NAS**) can fail in unexpected ways that will corrupt your data. I am aware of one such environment where the UPS was disconnected in a way that removed power from both the NAS system and the VM host. Although the Debian system used a non-cached journal for its filesystem, the NAS was buffering the journal writes. When power was lost, the journal was incomplete, and the disk ended up quite corrupted. Lesson learned.

## System logging

Another one of the system administrator's responsibilities is to manage the system logs. Debian systems by default log information that can tell an administrator how the system is being used, provide warnings and error messages that can indicate problems with software or hardware, and even provide early signs the system is being attacked or misused.

System logs are handled by the `rsyslog` package, and normally reside in `/var/log` and its subdirectories. Various software packages and especially those that provide important services, such as DNS, FTP, E-Mail, and HTTP (Apache), often do extensive logging that may include entries that allow activity to be tracked and warn of potential problems or misconfiguration. The package documentation includes information on how to configure what is logged and where.

### The logging facility

The `rsyslog` system logging facility in Linux provides for various options, facilities, and log levels. The options control what happens when a message is logged, such as whether it is sent to the console if it can't be logged in a file, whether the **process ID** (**PID**) is included in the message. The facilities provide a means to divide messages into various areas according to what subsystem is involved, such as AUTH for authorization messages, CRON for scheduled jobs, KERN for kernel messages. Most software that uses the system log provides configuration items to control the options, files, and facilities it uses for its messages. In addition, and most important, are the levels of log messages, which determine how important a message is.

Log levels detail which messages, of the many that might be sent to the system log, actually get logged. This varies from the EMERG level, which only involves messages that essentially mean the system is unusable, down through ALERT (immediate action required), CRIT (critical), and ERR (errors) to WARN (warning conditions), NOTICE (normal but significant), INFO, and DEBUG. Setting the log level to any of the latter three can generate a significant number of entries and use a lot of disk space as well as requiring significant system overhead.

For that reason, there are some guidelines as to what should be logged:

*   Production systems should only log EMERG, CRIT, ALERT, ERR, and WARN levels. NOTICE, INFO, and DEBUG should never be logged on a production system unless absolutely necessary.
*   Development systems should log those levels mentioned in the previous point, along with NOTICE and perhaps INFO levels to provide information necessary to the software developers. DEBUG may also be used when necessary.

The idea is to provide the necessary information without unduly burdening a system. In particular, NOTICE, INFO and especially DEBUG levels can produce massive amounts of data that are generally unnecessary in a stable production system.

Log data is frequently used to analyze how the system is being used, who accesses it, what activities are being performed, as well as to notify the administrator of things that need attention.

### Controlling the logs

Even production system logs will eventually grow to the point where they take up a significant amount of disk space, and this is even more of a problem with development systems that follow the preceding guidelines. One of the administrator's duties is to determine how much log data should be kept, and manage the files appropriately.

All distributions, and Debian is no exception, provide by default a job or jobs that run periodically to close the current log files, mark them with a cycle number, and open new, clean files for logging. In Debian, this is provided by the `logrotate` package. While this package is primarily concerned with log files, it can be used for any other files that grow constantly and need to be cycled. Detailed documentation is provided with the package, but the basic idea is that each log file is checked and if it is over a certain size, it is closed. Then all cycles of the log file are renamed, and if necessary, those over a certain age or cycle number are deleted. A new log file is then created to continue logging.

You can configure how many cycles are kept, or even how old the cycles can be before they are deleted. The primary choice here is how long you wish to keep log entries. The defaults installed with each package reflect the experience of many administrators and are usually appropriate in most circumstances.

### Monitoring the logs

So, does an administrator need to wade through all of the log entries daily? That would be an extremely tedious task, and is really not necessary. A number of packages exist in Debian that will scan the logs for certain conditions and email the results to the administrator for further checking and action. The most common is `logcheck`, which checks the latest log cycles against a database of entries that an administrator might be interested in. It then emails the important ones to the administrator for further checking. There are also packages that perform various analysis tasks, such as email statistics or HTTP access statistics, using the system logs often combined with other information sources. They may be found using the Debian package management search facilities.

### Tip

What package you use to check the logs, or even whether you use your own scripts based on string searches, is not important. The critical thing is to check the logs regularly. They can give you timely warning of impending hardware issues, software instabilities, programming problems, and attacks on your system.

# Display managers

Straight servers, as opposed to development servers, generally do not require a display manager. Of course, development servers and those servers that do provide the ability to log in to a managed display environment do require both a display manager and a window manager. The former handles the creation and security of the X-Windows display environment and resources required, while the latter handles the actual desktop environment presented to the user.

There are several major environments, each with its associated Display and Window Managers. The two major environments are GNOME and KDE. Both support a variety of graphic toolkit libraries, so that most applications that run in one will run in the other, provided the necessary services are available. These services are usually installed as a dependency when the application is installed if you don't install the basic meta package for either GNOME or KDE. So if, for example, you like the `KcacheGrind` tool for browsing program profiling data, but are using GNOME, you can go ahead and install and use it under GNOME. It will work fine. Likewise, if you prefer the **K Display Manager** (**KDM**) that comes with KDE to the **GNOME Display Manager** (**GDM**) provided by GNOME, it will work just as well.

Given this interoperability, why choose one over the other? It is mostly a matter of personal preference. As mentioned in the first chapter, KDE is usually preferred by European users, while GNOME is more of an American preference. Our main concern in this section will be where to find configuration files or applications for each.

## Where did my desktop go?

Both GNOME and KDE, like the new Windows 8, have abandoned the old desktop metaphor for a more activity or task oriented look. It takes some getting used to and can be disconcerting at first. Covering the changes from the old look to the new would take a book for each Desktop Manager! However, the home sites for both KDE and GNOME provide tutorials and documentation to help you get started. If you prefer the older desktop metaphor, both GNOME and KDE provide ways to set this up as well.

## GNOME

The current GNOME release in Debian 7 is 3.4\. If you install the `gnome` meta package, all major GNOME applications as well as the window and display managers, will be installed. Configuration is pretty straightforward, as the user menu provides a System Settings application to access the major settings of your workspace in order to alter how it looks and feels.

Modifying the GNOME login screen, GDM3, if you don't want the default behavior, requires manual editing of `/etc/gdm3/greeter.gsettings` and `/etc/gdm3/daemon.conf` as the root user. Then execute the `dpkg-reconfigure gdm3` command.

### Note

Part of the reason for this is that GDM is being integrated more fully into GNOME, and is being moved to the `dconf` settings framework from the old GNOME 2 `gconf` based settings. At some point, the GNOME control center should provide the appropriate utility but, at this time, there is no official GDM3 setup utility. An unofficial utility called `gdm3`setup exists, but has not found its way into Debian, yet.

## KDE

The current KDE release for Debian 7 is 4.8.4\. The `kde-full` meta package will provide all KDE applications as well as the window and display managers. The KDE user environment provides a System Settings application just as GNOME does, but it is a bit more comprehensive as it includes the ability to configure KDM, the display manager responsible for the login screen. In fact, because of this, some administrators use KDM as the display manager, even though the default session may be GNOME.

## Other desktops

XFCE and LXDE are the most common alternatives to KDE or GNOME, although there are others. Neither one provides a display manager for login screens, although LXDE recommends LightDM, a lightweight display manager. XFCE and LXDE (as well as LightDM) retain the old desktop metaphor, and are designed to place a minimal load on the system, making them appropriate for older hardware.

### Note

During the time *Linus Torvalds* abandoned GNOME3, he ran XFCE.

Like both GNOME and KDE, both XFCE and LXDE provide a system settings application to control the look and feel. LightDM also provides a graphical settings utility.

## Showing your best face

Take time to at least look into the settings for your chosen display manager. Unless you are running a server that never sees a graphical login, this is the first and last thing your users see. You may also want to look into appropriate backgrounds for your users' desktops. They don't need to be fancy but they do make an impression.

# Summary

The tasks of an administrator are many and include the responsibility for what services the system provides (especially how they are started and shut down), network configuration, system backup, filesystem space management, system operation (system logs), and providing the face the system shows to the world. We've covered some of the issues in each of these areas, although comprehensive coverage of any of the subjects could take several books. One subject not covered here that must be covered in depth is basic system security. We will cover this in the next chapter.