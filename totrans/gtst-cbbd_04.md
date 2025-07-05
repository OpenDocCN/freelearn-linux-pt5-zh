# Chapter 4. Manually Installing an Alternative Operating System

Installing a full-blown desktop **operating system** (**OS**) onto an SD card via an image is certainly useful, but limitations arise quickly. What if an installation to an SSD is desired? Or what about having a very minimal installation for use as a server? Surely no heavy GUI is needed for this? All these questions will be covered in this chapter.

This chapter will cover the following topics:

*   Partitioning and formatting a destination medium
*   Creating rootfs
*   Allowing booting of the destination medium
*   Updating the OS
*   Installing additional software

# Prerequisites for this chapter

In this chapter, Debian (or even Ubuntu) will be installed to an alternative installation medium. A SATA SSD will be used, but a regular SATA disk can be equally used given that enough electrical power is available to power the drive; a power adapter with at least 2 amperes is required in such a case. Alternatively, a second microSD card can be used in a microSD-to-USB adapter.

When installing to a SATA drive, however, the Cubieboard still requires an SD card to boot, as the SoC cannot boot from a SATA drive directly. Technically, the onboard NAND flash or an onboard SPI flash could be used for this instead, but SPI flash-enabled Cubieboards are hard to find, and working with NAND requires a very old u-boot, which lacks a lot of new features. In this chapter, the microSD card created in the previous chapter will be repurposed and used to accomplish all these tasks.

### Tip

USB flash drives can, in theory, be used, but at the time of writing this book, the USB boot code has not landed in u-boot. At the moment, a USB flash drive can only be used after booting a kernel with USB support, which has been loaded from either a NAND or an SD card.

# Preparing the destination medium

To install Debian to a SATA drive, the destination drive will need some preparation. It needs to be partitioned and formatted.

Assuming the Cubieboard is booted up using the Fedora image created earlier and has either a SATA drive, USB flash drive, or a microSD card with a USB adapter connected, it is time to start `fdisk` on the destination medium, which is assumed to be `/dev/sda`. Please make sure that the correct device node is used; otherwise, the following actions will destroy anything present on the medium.

The most common tool of choice to partition a device is `fdisk`. While `fdisk` has a few parameters, starting it with a device node will start `fdisk` in an interactive mode where a disk can be prepared. Root access is required to use `fdisk` and thus may need to be prefixed with `sudo`, as shown in the following screenshot:

![Preparing the destination medium](img/1572OS_04_01.jpg)

There should not be any previous data present or at least be backed up, which will not be covered here. Pressing the *o* key should clear any previously created partitions, as shown here:

![Preparing the destination medium](img/1572OS_04_02.jpg)

Partitions are a useful thing, they allow a logical separation. There are many reasons and choices when dividing a disk, as but here, only the following four partitions are of interest:

*   **Boot**: This partition holds all the relevant boot files
*   **Root**: This partition holds all the relevant system files
*   **Home**: This partition holds all the user files
*   **Swap**: This partition expands the RAM memory

In this example, only three primary partitions will be created as the boot partition will be on the SD card. For the root partition, 6 GB is used, and for a swap partition, 512 MB is used. The remainder is used for the home partition.

Ultimately, however, it is up to the reader to define what is useful to you as this can differ greatly. With the `n` command, a new partition can be created. In the following screenshot, the three partitions explained earlier are created:

![Preparing the destination medium](img/1572OS_04_03.jpg)

When not entering a value for the last sector, `fdisk` will use the last sector available as default, thus using the remainder of the medium.

Partitions should not only be created, but also need to be categorized. By default, `fdisk` will turn all newly created partitions into regular Linux filesystem partitions, which is fine except for the swap partition. This partition needs a different type applied to it. The `t` command is used to categorize a partition, which in turn wants to know the exact type to use. For swap, this is type 82\. The `l` command can be used at any time to get an overview of the available types. The following screenshot shows how to turn partition `2` into a swap partition:

![Preparing the destination medium](img/1572OS_04_04.jpg)

Using the commands `w` to write and `q` to quit, the newly created partition table is saved to the disk and `fdisk` quits, as shown here:

![Preparing the destination medium](img/1572OS_04_05.jpg)

# Formatting the newly created partitions

With freshly created partitions available, they now need to be formatted. In this book, `ext4` will be used as the filesystem.

### Tip

While not yet supported by the 3.4.x kernel used in this book, `f2fs` is very interesting as it is optimized for SSD, USB, or microSD flash usage. This is something that could be interesting in future.

The command to format a partition is `mkfs.ext4`, and the parameters that are of interest are the device node being formatted and optionally `-L`, which is used to give the partition a name.

Formatting the root partition is shown here:

![Formatting the newly created partitions](img/1572OS_04_06.jpg)

Formatting the swap partition can be done using the command in the following screenshot:

![Formatting the newly created partitions](img/1572OS_04_07.jpg)

Formatting the remainder for the user files can be done as shown here:

![Formatting the newly created partitions](img/1572OS_04_08.jpg)

To ensure that all the data is written to the appropriate places, the partitions are mounted on the existing filesystem as follows:

```
[root@packt ~]# mount /dev/sda1 /mnt
[root@packt ~]# mkdir /mnt/home
[root@packt ~]# mount /dev/sda3 /mnt/home

```

# Creating a Debian or Ubuntu rootfs

The first thing that should be mentioned here is that Ubuntu is a Debian derivative. To cut a long story short, it is basically Debian, and as a result, it does not hugely matter which distribution is used for installation, be it Debian or Ubuntu. So when talking about installing Ubuntu or Debian, the same thing is really being said. The tool used for installation here is called **debootstrap** and is available in a lot of distributions.

## Installing debootstrap

Fedora has debootstrap available and can be installed via the Yum tool, as shown here:

![Installing debootstrap](img/1572OS_04_09.jpg)

## Running debootstrap

With debootstrap installed, it is almost time to get started. A few things need to be mentioned first; debootstrap, which stands for Debian bootstrap, can be used to install any Debian variant for any architecture for at least Debian and Ubuntu. It does require a mirror to be supplied to its list of arguments. The list of mirrors can be obtained for Debian at [http://www.debian.org/mirror/list-full](http://www.debian.org/mirror/list-full), but for Ubuntu, there is no official list of mirrors. However, using the country code in the URL can result in a mirror, for example, in the Netherlands, [http://nl.ports.ubuntu.com](http://nl.ports.ubuntu.com) is a valid mirror. Using a mirror has the obvious advantage that the download will proceed much faster.

Since Allwinner SoCs are based on the ARMv7 architecture, for the architecture, **armhf** will be used for that.

The **suite**, as it is called, depends on what is desired. For Debian, there are the stable, testing, and unstable suites, where **Wheezy** is the name of the stable branch, **Jessie** is testing, and **sid** is the unstable branch. It should be noted that in future, the names Wheezy and Jessie will change to new suite names, but sid will always remain the unstable development version.

Finally, debootstrap is prefixed with the `PATH` variable to ensure debootstrap uses the correct path. This is due to a bug currently in debootstrap in combination with the newer distributions.

### Tip

The `–foreign` parameter can be used to bootstrap any architecture, even on an x86-based system, as no code is executed. Bootstrapping will require some additional work using the `–second-stage` parameter. It is up to the reader to learn more about this when cross bootstrapping.

The following command will start the installation of Debian Wheezy for arm-hard-float into `/mnt` using the [http://ftp.nl.debian.org/debian/](http://ftp.nl.debian.org/debian/) mirror, as shown here:

![Running debootstrap](img/1572OS_04_10.jpg)

As debootstrap is written in Perl. It may be possible that **Perl** is not currently installed on the system where debootstrap is being used. Installing Perl may follow a long list of packages that need to be downloaded and installed.

### Tip

Similarly, debootstrap `–foreign --arch=armhf trusty /mnt http://nl.ports.ubuntu.com` can be used to install **Ubuntu Trusty Tahr** with a note that this will only be the base system. Also, debootstrap might not come with all the suites as expected. All Ubuntu suites are symlinks to the gutsy suite at `/usr/share/debootstrap/scripts` as long as it exists on [http://ports.ubuntu.com/dists/](http://ports.ubuntu.com/dists/). Use `ln -s gutsy utopic` in the scripts directory, for example, to add utopic as a valid suite.

## Configuring the base system

The so-called `fstab` file in Linux is responsible for mounting partitions in their designated positions. Using any editor, the following changes need to be added to the `fstab` file. While the UUID-based mount points can be used, only standard entries are used here. However, you are welcome and even encouraged to use the UUID-based mount points. A common editor that is relatively easy to use is **nano**. After modifying the file, exit nano with the *Ctrl* key in combination with the *x* key and answer the question to save the modified buffer with the *y* key. The filename should remain the same, thus answering with the *Enter* key. The `fstab` file in the editor is shown in the following screenshot:

![Configuring the base system](img/1572OS_04_11.jpg)

With `/boot` mounted read-only, it makes sure that not only no accidental writes happen, but also no intended writes occur. Also, as mentioned earlier in this chapter, the boot partition of the microSD card, which was created in the previous chapter, is reused here.

## Configuring the networking

Debian and Ubuntu use the `interfaces` file at `/etc/network/interfaces` to configure networking. Note that this is a more permanent configuration used when not using the graphical utilities, such as network manager. If the final goal of this setup is a graphical desktop, it is probably wise to skip setting up the `interfaces` file.

### Tip

Using the network interface `eth0` as the parameter for `dhclient` should result in a working network connection, as shown in the following command. However, this will be lost after a reboot.

```
dhclient eth0

```

Use nano to open the `interfaces` file at `/mnt/etc/network/interfaces` and make the following addition at the bottom:

```
auto eth0
iface eth0 inet dhcp

```

Generally, a similar section exists for the loopback device.

If a static IP configuration is required, the following can be used as an example (do replace the proper values with all numbers for the desired network).

```
auto eth0
iface eth0 inet static
 address 192.168.0.10
 network 192.168.0.0
 netmask 255.255.255.0
 broadcast 192.168.0.255
 gateway 192.168.0.1

```

When using a static IP address, the system also needs to be told how to resolve things; the `resolv.conf` file at `/etc/resolv.conf` is responsible for this. Note that this file will get overwritten if the network is configured either via `dhcp` or via a network manager. Using nano as an example editor, open the `resolv.conf` file at `/mnt/etc/resolv.conf` to add the following lines to it:

```
search homedomain.local
nameserver 192.168.0.1

```

Also here, we need to properly replace the values with whatever is appropriate for the network used. Remember to save the file using *Ctrl* + *x*.

If there is more than one nameserver on the local network, a new line prefixed with the word nameserver should be used for each additional nameserver.

### Tip

In the unlikely event that there is no nameserver available on the local network, Google's or OpenDNS's nameservers can be used. For Google, they are `8.8.8.8` and `8.8.4.4`, and for OpenDNS, they are `208.67.222.222` and `208.67.220.220`.

Finally, the system needs to be named on the network—the so-called `hostname`. Simply write a name that is unique on the network in the `hostname` file at `/mnt/etc/hostname`, as shown here:

```
[root@packt ~]# echo "PacktPublishing" > /mnt/etc/hostname

```

Another thing that needs to be set up is the so-called hosts file. It serves as the most basic way to look up a hostname, for example, when there is no DNS server available. The hostname needs to be in here in addition to any other hostnames that are required to be available from the network; for example, there is a time server on the network from where all the computers get their current time. Every system queries this server via `time.example.com`. Even if there's no Internet connectivity and no DNS service available, to ensure the time server is always able to be looked up, an entry to the hosts file can be added. With `192.168.0.15` being the local time server, the following command can be used as an example for the hostname and a guide to add additional hosts. Note that quite often, there is little need to add additional hosts, as DNS is nearly always used for this purpose. Remember to save the file using *Ctrl* + *x*.

```
[root@packt ~]# nano /mnt/etc/hosts
127.0.0.1       localhost
127.0.0.1       PacktPublishing
192.168.0.15    time.example.com
::1             localhost ip6-localhost ip6-loopback
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

```

# Making the destination medium bootable

Unfortunately, as mentioned before, the SoC cannot boot from SATA drives or USB flash drives; it requires a helper component. In this book, a small microSD card is used for this purpose. As such, it will hold the bootloader, the kernel, and a little bit of configuration to glue it all together. In the previous chapter, the installation script used for the Fedora installation was done automatically. The kernel that will be installed onto the microSD card will, by default, continue loading the rootfs from the microSD card. This will obviously need to be adjusted so that the microSD card will boot the newly created medium. To do this, the boot partition needs to be mounted first, as shown in the following command:

```
[root@packt ~]# mount /dev/mmcblk0p1 /mnt/boot

```

Using nano, edit the `uEnv.txt` file and modify the line that starts with root `/dev/sda1`, as shown in the following screenshot. Remember to save the file using *Ctrl* + *x*.

![Making the destination medium bootable](img/1572OS_04_12.jpg)

With the microSD card set up to boot from the newly created medium, it is safe to unmount it again with the following command:

```
[root@packt ~]# umount /mnt/foo

```

If things go wrong and the Cubieboard refuses to boot, the microSD card can be used in a microSD-to-USB adapter and the `uEnv.txt` file can be opened with a locally installed text editor. The OS used to make this modification, however, will need to be able to read and write `ext4` or at least `ext2` filesystems.

# The root user

While a root user exists with any default Debian or Ubuntu installation, the question arises as to how to log in as the root user. The following are the two options:

*   Precreate a regular user that has administrative rights via `sudo` and don't allow the root user to log in
*   Or the easier way, set up a root password for the root user and use it

Security-wise, the first option is safer. Both options will be briefly covered here. It is up to the reader to decide which option is better suited and how important the security aspect of it all is. This book is also not about properly securing a system. This is left to the reader as an exercise and is far beyond the scope of this book.

## Preparing the chroot command

To set up a root password, a few steps are required, as this has to be done actually from within the system. The `chroot` command makes it possible to actually enter the system as if it was booted as such. But there is a prerequisite, that is, certain dynamic directories need to be populated, namely, `/dev` and `/proc`, as shown here:

```
[root@packt ~]# mount --rbind /dev /mnt/dev
[root@packt ~]# mount none -t proc /mnt/proc

```

Here, the existing `/dev` mount is reused whereas the `proc` filesystem is mounted normally. Now, it is possible to use `chroot` into the filesystem, as shown here:

```
[root@packt ~]# PATH=/bin:/sbin:/usr/bin:/usr/sbin chroot /mnt /bin/bash
root@packt:/#

```

### Changing the root password

The `passwd` command will be used to start the password change for the root user, where a new password is entered twice. The system will not echo anything back to the user, as shown here:

```
root@packt:/# passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
root@packt:/#

```

### Creating a new super user

To create a new user, the `useradd` command is used. There are a lot of options to this command and it is up to you to get familiarized with them. The options used in this example are, however, the ones generally used. Besides a new user, a new group matching the user's username will also be automatically created; the `-U` flag is responsible for that. Additionally, the `-s` flag is used to supply an alternative login shell. This is completely optional but recommended as Debian defaults to the default `sh` shell, which is rather limited. The `-m` flag creates a directory for the user and copies some basic files. In the following example, the username used will be `packt`:

```
root@packt:/# useradd -m -G sudo -s /bin/bash -U packt
root@packt:/#

```

To give the new user administrative powers, it will need rights to the `sudo` command. The user has already been made a part of the `sudo` user group via the `-G` parameter. But the command needs to be actually available to be usable. While the usage of `apt` will be discussed later in this chapter, use the following command to install `sudo`:

```
root@packt:/# apt-get install sudo

```

Finally, the user will also require a password to actually log in. The following `passwd` command can be used for this:

```
root@packt:/# passwd packt
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
root@packt:/#

```

## Exiting chroot

The `chroot` environment can now be exited using the `exit` command, as follows:

```
root@packt:/# exit
exit
[root@packt ~]#

```

# Adding the serial console

If the system were to be rebooted at this point, the login console would show up on `tty0`, which is the normal console when a monitor and keyboard are connected. There might, however, be a case where the connected monitor is not immediately compatible or where for some reason the USB keyboard or monitor cannot be used. The serial console has served us very well until now, and thus, in order to enable it on this, the Debian or Ubuntu installation is strongly recommended. Here, the first difference between Debian and Ubuntu, however, becomes apparent. Debian still uses the older `sysvinit`, whereas Ubuntu still uses `upstart`, and while both are slowly in the progress of migrating to `systemd`, this is not applicable at this moment. Ironically, with `systemd`, or at least with the one that is installed and currently running, the Fedora image has the serial console set up by default.

## Adding the serial console to Debian

The file responsible for spawning the various `init` services is `inittab` at `/etc/inittab`, and uses nano; this can be edited to add the serial console. Find the following section, uncomment the line starting with `#T0`, and remove the hashtag. Also note that by default the baud speed is `9600`, and our entire setup assumes a baud speed of `115200`, so make sure that this change has been performed, as shown in the following example. Remember to save the file using *Ctrl* + *x*.

```
[root@packt ~]# nano /mnt/etc/inittab
# Example how to put a getty on a serial line (for a terminal)
#
T0:23:respawn:/sbin/getty -L ttyS0 115200 vt100
#T1:23:respawn:/sbin/getty -L ttyS1 9600 vt100

```

## Adding the serial console to Ubuntu

For `upstart`, the situation is completely different. The files in the `/etc/init/` directory are parsed by `upstart`. First, copy the file `tty1.conf` to `ttyS0.conf` as this makes editing much easier; the files are similar after all, as shown here:

```
[root@packt ~]# cd /mnt/etc/init/
[root@packt init]# cp tty1.conf ttyS0.conf

```

However, a few changes need to be made to this file. First, replace all occurrences of `tty1` with `ttyS0`. Next, in addition to `2345`, add `1` to the number of run levels. Also, remove the `and` `(…)` section. Finally, the `getty` line needs to be adjusted to listen to a serial port at `115200` bps. The following command shows you how the file eventually should look like. Alternatively, a new file can be created with the following content. Remember to save the file using *Ctrl* + *x*.

```
# ttyS0 - getty
#
# This service maintains a getty on ttyS0 from the point the system 
# is started until it is shut down again.

start on stopped rc RUNLEVEL=[12345]
stop on runlevel [!12345]

respawn
exec /sbin/getty -L 115200 ttyS0 vt102

```

# Rebooting the new OS

With all the required changes in place, it is time to reboot into the new operating system. First, unmount all partitions that have been mounted for this installation. Make sure that none of the mounted directories are in use. The `umount -l` command is used, where the `-l` parameter stands for lazy, which means that `umount` will try to unmount all subdirectories first and finish with unmounting the requested directory. In case of errors, manual unmounting is probably required. The `reboot` command will then reboot the system, as shown here:

```
[root@packt etc]# cd
[root@packt ~]# umount -l /mnt
[root@packt ~]# reboot

```

After the reboot, you will be greeted with a login prompt, as shown in the following screenshot. Obviously, this differs slightly between Debian and Ubuntu. After logging in with either the created user or the root, the installation part of this chapter is completed, congratulations!

![Rebooting the new OS](img/1572OS_04_13.jpg)

# Getting around the new OS via the command line

If you are new to GNU/Linux in general, using the cheatsheet as shown in [Appendix B](apb.html "Appendix B. Basic Linux Commands Cheatsheet"), *Basic Linux Commands Cheatsheet*, will be helpful, as a few common Linux commands can be explored, and it can be considered as a beginner's guide to GNU/Linux. This section will focus a little more on common tasks, keeping Debian or Ubuntu up to date and installing new software.

## Introducing apt

Both Debian and Ubuntu rely heavily on apt for their software needs. Apt is a suite of commands that allows the installation of new software packages or keeping the existing ones up to date. Apt works closely together with dpkg although a regular user will probably never invoke dpkg directly. Apt is responsible for downloading a requested piece of software, checking what its dependencies are, and telling dpkg to install them. It is the Swiss army knife of package management under Debian and Ubuntu. It was the command-line AppStore before AppStores even existed.

## Configuring apt

Apt does not require a whole lot of configuring. What apt does need is a list of places where it can check and download software from. The Debian and Ubuntu defaults might not offer everything of interest. The file responsible for apt's configuration is `sources.list` at `/etc/apt/sources.list`, and if you are logged in as a regular user, it requires `sudo` to grant the user additional privileges to edit the file. In the following example, the non-free component and the security repository will be added to the main component. Note that the various sources in the `sources.list` file will vary between various suites or derivatives, such as Ubuntu.

```
packt@PacktPublishing:/etc/apt$ sudo nano sources.list
[sudo] password for packt:
deb http://ftp.nl.debian.org/debian wheezy main non-free

deb http://security.debian.org/ wheezy/updates main non-free
deb-src http://security.debian.org/ wheezy/updates main non-free

```

With these changes in place, apt will need to be updated, but that is handled in the next subsection.

Debian has a good tutorial for more in-depth reading at [https://wiki.debian.org/SourcesList](https://wiki.debian.org/SourcesList), and an additional components and repositories are listed at [https://wiki.debian.org/UnofficialRepositories](https://wiki.debian.org/UnofficialRepositories). For Ubuntu, the source list can be found at [https://help.ubuntu.com/community/SourcesList](https://help.ubuntu.com/community/SourcesList) and additional repositories at [https://help.ubuntu.com/community/Repositories/Ubuntu](https://help.ubuntu.com/community/Repositories/Ubuntu). Further configuration beyond what is listed in this subsection is left as an exercise to the reader. Note that while in theory Ubuntu and Debian repositories can be mixed, it is not recommended and will likely cause issues.

## Keeping the OS up to date

Regularly checking and installing updates can be critical to security. Also, new versions of existing software packages potentially yielding new features or fixing bugs are also obtained in this way. The steps are identical for Ubuntu and Debian.

To download and update apt with a new list of the software, the first apt command `apt-get` is introduced. The `apt-get` command without parameters, however, will not do a lot more than print a help screen. As the intended action is to update the list of applications that will be the `update` parameter passed. Again, if run as a regular user, it should be prefixed with `sudo`. Also, if the network has not been configured yet and the intention is to let the graphical user interface configure the network, remember to run `dhclient eth0` to obtain a temporary network configuration. The following command in the screenshot runs an update of apt:

![Keeping the OS up to date](img/1572OS_04_14.jpg)

If sections and components were not added in the previous subsection, the list would obviously be shorter. At this point, the apt has an up-to-date list of the available software. To upgrade all installed packages, most importantly because of security updates, the `upgrade` parameter to `apt-get` is used, as shown in the following screenshot:

![Keeping the OS up to date](img/1572OS_04_15.jpg)

In the preceding example, there was only one update available, which was an update to `libgnutls`. This list can vary depending on the amount of updates, naturally.

### Tip

The `apt` file at `/etc/cron.daily/apt` is responsible for the daily update of apt and installing security critical packages, thus running the `apt-get` update is not always required.

## Installing additional software

The most exciting thing about any OS probably is the software. However, the default created Debian or Ubuntu installation lacks most of the software. It is, after all, just the bare minimum.

### Finding packages

While installing packages, a problem can occur when a name is roughly remembered but not to exactly. In that case, there is an apt tool called `apt-cache`. Using the `search` parameter, the internal cached apt database can be searched for available packages. Especially in combination with the `grep` command, these two can be helpful to find what is needed, as shown in the following screenshot. The `grep –` command is explained in [Appendix B](apb.html "Appendix B. Basic Linux Commands Cheatsheet"), *Basic Linux Commands Cheatsheet*.

![Finding packages](img/1572OS_04_16.jpg)

Additionally, you might want to search for a filename of an application. There is an apt tool for that as well. The `apt-file` tool using the `search` parameter will allow you to search for files inside packages. Unfortunately, at the time of writing this book, the `apt-file` tool is not yet included in the current stable release of Debian or Ubuntu, but should be hopefully added soon.

### Installing the software package using apt-get

The most basic way to install a software package is also via `apt-get`, the parameter not surprisingly being install. Nano has been used often as an example editor. This is because it is very easy to use and nearly always preinstalled as it is so small. **Vi** is another small editor that is nearly always preinstalled but is far from easy to use. Vi has a bigger brother called Vi improved or vim. Let us install vim via `apt-get`, as follows:

![Installing the software package using apt-get](img/1572OS_04_17.jpg)

After downloading and installing a few packages, vim is now installed.

### Installing the software package using tasksel

More than often, a collection of packages is required to have the system perform certain functions. The `tasksel` command can be used to install a collection of packages to perform a certain task. Running `tasksel` with elevated privileges yields the menu, as shown in the following screenshot. Ubuntu does not have `tasksel` preinstalled and requires it to be installed via `apt-get`, as shown in the previous example for vim.

![Installing the software package using tasksel](img/1572OS_04_18.jpg)

Installing the Debian desktop environment task will install a graphical desktop environment based on **GNOME** and some additional packages marked as standard by Debian, such as LibreOffice.

The downloading and installation might take a while depending on the target medium and Internet connection speed. Installation on an SSD with a very fast Internet connection can take about thirty minutes.

Under Ubuntu, `tasksel` will look slightly different, but even here there is an Ubuntu desktop option, as shown in the following screenshot. This will take about the same time thirty minutes or more.

![Installing the software package using tasksel](img/1572OS_04_19.jpg)

### Installing packages via metapackages

The tasks available via `tasksel` appear to be rather limited. Instead, it is probably easier to use metapackages. Metapackages are, in effect, not that much different from tasks; in fact, they very well might be the same in the background. A metapackage is not really a package that installs anything, rather it is a list of packages or a collection of packages that get installed from it. For example, `xfce4` is a metapackage for Debian, which will install all the packages required for the `xfce4` desktop environment and will also install software that the package maintainers thought would make sense to have for a full-fledged desktop environment, such as a file manager.

### Note

For Ubuntu, this metapackage is just called xfce, though the xubuntu-desktop metapackage should be more interesting here.

Another interesting metapackage to match the `xfce4` metapackage is `xfce4-goodies`. In fact, the following command will show you how to install multiple packages in one go. Running the command will result in a huge list of packages to be installed, but will yield a usable `xfcef4` desktop, as if it were installed from a CD, for example. One could even argue that an installation CD would do just that, as shown here:

```
packt@PacktPublishing:~$ sudo apt-get install xfce4 xfce4-goodies

```

After waiting about 30 minutes for the xfce4 desktop to get installed, xfce4 can be started using the `startx` command. At this point, however, the monitor and keyboard will have to be used. Xfce4 cannot be used or started over the serial console. This yields an almost usable desktop environment. Almost means here that certain things with permissions are missing. A user is not allowed to shut down the machine by default, for example.

There are several ways to allow these things to work properly, and one is the use of a login manager; xfce4, however, does not come with one, but that is okay. There are plenty of login managers to choose from. GNOME comes with `gdm`, but going with a lightweight login manager that matches Xfce4 as a lightweight desktop manager, LightDM should be a good candidate. Installing the `lightdm` package should require no extra instructions. Rebooting the Cubieboard now will yield an active login window, which upon login, wraps up this chapter.

# Summary

Having worked through this chapter, admittedly a big one, the result should technically be the same as the previous chapter; a working desktop environment based on either Debian or Ubuntu. The installation of additional software via the command line is now a breeze, and keeping the system up to date is not an issue at all.

The next chapter will take this installation and turn it into a server for various tasks, optionally retaining the desktop functionality.