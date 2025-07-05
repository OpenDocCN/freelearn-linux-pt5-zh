# Chapter 2. Cold Starts

In the Northern Hemisphere, I think we can all relate to the analogy of the cold start; those bleak January mornings where you are frantically trying to start your car. When it does finally splutter into some form of life, we then have to contend with a steering wheel too cold to hold. Thankfully, starting up a Linux system is not so unpleasant; perhaps air-conditioned server rooms have something to do with this, I am not sure…

Working through this chapter, we are going to build upon what you have already mastered—helping you understand your Linux systems. You will learn about the following topics:

*   **The GRUB and the MBR**: In this section, you will learn about the relationship that the **GRand Unified Bootloader** (**GRUB**) enjoys with the **Master Boot Record** (**MBR**), being able to slip its slender 466 bytes easily inside the 512-byte limit.
*   **When is the root filesystem not the root filesystem?**: In this section, we will understand the term *root* when used as a directive within a GRUB stanza, which is a little hurdle we shall overcome.
*   **Working on the** **GRUB console**: In this section, you will learn how to enable some powerful recovery tools.
*   **Protecting the GRUB menu with passwords**: In this section, you will learn how to enforce physical security of your systems: desktops or servers.
*   **Boot splashing with plymouth**: A little fun to finish the section with, we will look at the range of boot splash screens that we can use with CentOS. By the end of this chapter, your Linux system will never have been so well dressed.

# The GRUB and MBR

This is not just a competition to see how many acronyms we can fit into a chapter heading, although, out of four words, having used two already is not a bad start. The **GRUB** is the system-supplied bootloader that ships with CentOS and Red Hat Enterprise Linux 6\. This tiny piece of bootstrap code is used to load the kernel and allows us to dual boot different Linux versions or even with Microsoft Windows operating systems. The GRUB has been the bootloader of choice for many years, although other bootloaders do exist. These include:

*   **Lilo**: This is the original Linux loader
*   **EXTLinux**: This is part of the SYSLinux family that includes the following:

    *   EXTLinux to boot from fixed drives
    *   ISOLinux to boot from CDs and DVDs
    *   SYSLinux to boot from a USB device
    *   PXELinux to boot from the network

*   **GRUB2**: More recently, this is making its appearance as a replacement to GRUB, or what is now referred to as the legacy GRUB. GRUB2 is likely to debut in CentOS 7 in 2014.

The GRUB bootloader is most commonly stored in the MBR of the bootable drive.

### Tip

Although generally stored within the MBR, it is possible to install GRUB into the superblock, or the first 512 bytes, of a partition.

The MBR makes up the first 512 bytes of the disk, allowing up to 466 bytes of storage for the bootloader; the additional space will be used to store the partition table for that drive.

We can back up the MBR to a file using the `dd` command as follows:

```
# dd if=/dev/sda of=/tmp/sda.mbr count=1 bs=512

```

The `dd` command is used to duplicate a disk. In the previous command, we read from the first disk, `/dev/sda`, and backed it up to the `/tmp/sda.mbr` file. Rather than duplicating the entire disk, we limit the backup to a count of one block of 512 bytes.

Now that we have a backup for the MBR, we can investigate this fact a little more by running the following command:

### Tip

The following commands can be destructive, in that they will destroy the MBR, so please take care if you will be running commands on your own system, and I would recommend running only the following demonstration commands on a test system.

```
# dd if=/dev/zero of=/dev/sda count=1 bs=512

```

With the preceding command, we have wiped the data stored within the first 512 bytes of the disk `/dev/sda`. The MBR now is effectively cleared. We can verify this by using the following command:

```
$ lsblk /dev/sda

```

The output should display an empty partition table. The system remains usable as the partition table is resident to the RAM on the running system; however, until we are able to restore the MBR, a reboot will soon identify how much of a disaster we are in. Never fear, we can restore the MBR from the backup. What `dd` takes away, `dd` can return, simply by using the `dd` command as follows. Quickly, before someone notices!

```
# dd if=/tmp/sda.mbr of=/dev/sda

```

We do not need to limit the amount of data to be read from the specified file. Remember, it only contains the 512 bytes that make up the MBR. With a little luck, using the `fdisk` command will now show the partition table correctly as it was before, and you can begin to breathe easy again:

```
$ fdisk /dev/sda

```

### Tip

Using the `dd` command to wipe a disk completely with the `/dev/zero` input file is useful should you wish to wipe a disk before selling a computer, ensuring that the operating system, applications, and most importantly, the data is not sold with the device. We use `fdisk` in the second example as `lsblk` reads from memory and not the disk.

Once we have booted into GRUB, a menu will be shown allowing the user to select the **operating system** (**OS**) to enter. In general, the default selection is loaded without user interaction. We can configure the menu choices using the `/boot/grub/menu.lst` file. You will learn more about this file later.

# When is the root filesystem not the root filesystem?

We now need to break down the menu entries within the file, identifying the core components so that we can understand how they relate to the system and, most importantly, how we can correct errors.

## Editing stanzas in GRUB

Each entry in the GRUB menu is known as a **stanza**, and each stanza will start with the `title` word, containing three directives as follows:

*   `root`
*   `kernel`
*   `initd`

The title of the stanza also becomes the displayed item in the menu. Let's consider a stanza that begins with the following title:

```
title CentOS 6.5 OS
```

The menu will display `CentOS 6.5 OS` as the selectable item, and it is important to note that we do not add quotes around the text as they will also be displayed to the user. This is unless, of course, you want or need to display these quotes; we are most certainly not quote unfriendly at Packt Publishing!

## Adding a root entry to a stanza

Directly following the stanza title will be a line that starts with the `root` directive. This identifies the root filesystem to GRUB and not the OS root; in simple terms, this should point to the partition that is marked as bootable in the partition table.

We can use the `fdisk` or `parted` command to display the bootable partition. If you are using the `fdisk` command to display the partition information, the command would be similar to the following where we want to list the partitions of the first hard drive within the system:

```
# fdisk -l /dev/sda

```

The partition marked as bootable will be identified with an asterisk mark. If you are using the `parted` command to display the partition table, you will be able to identify the bootable partition by the boot flag by executing the following command:

```
# parted /dev/sda print

```

### Tip

The `fdisk` shows the bootable partition with `*` and parted with the word `boot`.

The bootable partition can be `/boot` or the actual root filesystem itself `/`. This relates to how the system was configured as it was installed. It might often be the case that `/boot` will have its own partition to ease access by the bootloader. The legacy GRUB, for example, cannot access a filesystem built on **Logical Volume Management** (**LVM**); this is the default partitioning proposal in CentOS 6\. The same applies to software **Redundant Array of Inexpensive Disks** (**RAID**) arrays.

Consider the following stanza:

```
title CentOS 6.5 OS
  root (hd0,0)
```

From this, we can determine that GRUB should mount the first partition on the first drive (both the drive and partition numbering starts at 0) as the bootable partition.

To summarize, the `root` directive in a GRUB stanza indicates the partition that the MBR marks as bootable.

## Adding a kernel entry to a stanza

The directive, `kernel`, directs the bootloader to the target operating system kernel. The path to that kernel will be related to the GRUB root partition, or the bootable partition. If the path reads `/vmlinuz.version`, then this would be an indication that the kernel is located at the root of the bootable partition, whereas the path `/boot/vmlinuz.version` would indicate that the bootable partition is the Linux or OS root partition. The path has to include the `/boot` directory to be able to locate the kernel.

Following the filename of the kernel are the arguments used when loading the kernel, or more simply referred to as the kernel options. These options include, among others, the device name where the real root filesystem is located and the device name for the swap filesystem, which can be used to suspend the system, perhaps on a laptop build. An example of the OS root option would be `root=/dev/sda2`; this being the second partition on the first hard drive or `root=/dev/mapper/vg_centos-vg_root`. This indicates that the operating system root is built upon an LVM. The swap filesystem to be suspended is indicated by the `resume` option.

The following extract from a stanza indicates that the boot partition is `/dev/sda1 (hd0,0)` and the operating system root is `/dev/sda2`, with the swap located on `/dev/sda3`:

```
title CentOS 6.5 OS
  root (hd0,0)
  kernel /vmlinuz.version root=/dev/sda2 resume=/dev/sda3
```

If the OS root is also the bootable partition, the corresponding GRUB stanza would read similar to the following:

```
title CentOS 6.5 OS
  root (hd0,0)
  kernel /boot/vmlinuz.version root=/dev/sda1 resume=/dev/sda2
```

We can see that the path to the kernel is now the full operating system path and both the GRUB root and the OS root correspond to the same partition.

Given a running system where the boot process is completed and we are logged in, it is possible to view the version of the kernel with either of the following commands:

*   `$ cat /proc/version`
*   `$ uname –r`

You should look at both commands and see which one best suits your needs; the `/proc/version` file will give a little more information. However, the `uname -r` command summarizes the information well. This is your system and it is your choice.

Should we need to list the options with which the kernel was booted, we can display those options with the following command:

```
$ cat /proc/cmdline

```

By this stage, I am hoping you have a little more understanding of when the root filesystem may not actually be the root filesystem and when it can be the root filesystem. You are now ready to use this riddle anytime that you wish to confuse your colleagues. It really is a simple matter of knowing where the partition that holds the kernel is; this then becomes the root of the bootable partition. The OS root is what we normally think as of the root filesystem but this happens only once the system has completed the boot process. The kernel directive simply points to the kernel file with a path relative to the root of the boot partition along with any options that we may wish to pass through to the kernel when it is loaded.

### Tip

The `/proc` directory is a pseudo filesystem, meaning that it is transient and resides only in the RAM. It contains up-to-date information for the currently running system. This directory is worth becoming acquainted with.

## Adding an initrd entry to a stanza

Similar to the `kernel` directive, the `initrd` directive will point to the initialization RAM disk; a mini OS that is compiled with the drivers needed to access the OS root filesystem. The RAM disk loads prior to the kernel and mounts the OS root filesystem as read-only. Filesystem integrity checks are performed before handing it to the kernel to continue with the boot process and mounting as read/write. This means that the kernel does not have to have the drivers for the root filesystem internally compiled, allowing more flexibility in changes to the OS root and a more lean kernel. The RAM disk can be recompiled if the root filesystem changes or the drivers need to access the hardware change with the `mkinitrd` command.

Continuing with our example stanza, we can insert a line for the `initrd` directive to read as follows:

```
title CentOS 6.5 OS
  root (hd0,0)
  kernel /boot/vmlinuz.version root=/dev/sda1 resume=/dev/sda2
  initrd /boot/initramfs.version
```

Not wishing to be out performed by the preceding simple text, the following screenshot shows an extract from a real GRUB stanza on my CentOS 6.5 system.

![Adding an initrd entry to a stanza](img/5902OS_02_01.jpg)

# Working on the GRUB console

When presented with the GRUB menu, as well as selecting the entry we wish to boot, we can either edit existing entries or shell out to the GRUB console. Working on the GRUB console enables us to enter our own sets of commands. Remember the trilogy that should accompany each stanza:

*   root
*   kernel
*   initrd

We can enter these commands, but also reinstall GRUB if required. More simply, in the console, we can also edit or append to the exiting entries; using the *e* key, we can edit an entry, and *a* can be used to append an option to the kernel line. From the following screenshot, we can view these options:

![Working on the GRUB console](img/5902OS_02_02.jpg)

Editing the kernel arguments allows you to specify the runlevel target to boot to; using this method, it is possible to reset the password of the root user.

To recover a forgotten root password, we can boot the system to runlevel 1; by default, this will log you in directly as root.

1.  Firstly, we must select the entry in the menu to boot to. If there is more than one, do not use the *Enter* key.
2.  With the menu entry highlighted, choose the letter *a*.
3.  This will take you directly to the end of the kernel line where you can add the number 1 to boot to runlevel 1.

    ### Note

    It is important to note that *CentOS System Administration Essentials* assumes that no prior runlevel has already been specified in the kernel arguments.

With the number added, just hit the *Enter* key, and the system will boot to the single user mode and logged in as root. Once the system has been booted, you can effectively change a password using the `passwd` command.

It is possible to prevent this behavior; we have to be cautious to avoiding the prevention genuine recovery mechanisms of our server. If there is enough physical protection of the server, then perhaps we do not need to make any changes. However, if we cannot ensure physical security of the server, we can edit the `/etc/sysconfig/init` file by changing the `SINGLE=/sbin/sushell` line to the following:

```
SINGLE=/sbin/sulogin
```

The `sulogin` command will prompt for the root user's password.

### Tip

If `sulogin` has been set and you still need emergency access as root, it is possible by specifying `init=/bin/bash` instead of 1 as the runlevel.

If our boot situation is a little more serious, or in human terms, it won't boot, then we can enter the GRUB command prompt using the option `c`. Using the command `help`, we can determine what commands are available from the minimal shell. To reinstall GRUB with the correct drivers to access the boot partition, execute the following command:

```
grub> setup(hd0)

```

The preceding command will check to see if `/boot/grub/stage1` or `/grub/stage1` exists on the bootable partition. This way, it determines which partition to use as root and copies the `stage1` file to the MBR complete with the drivers needed to access the bootable partition. We can then choose to restart the system with the `reboot` command.

Not only can we use the GRUB console to repair GRUB, we can use it to boot the system and verify the menu items. By specifying the root filesystem to be used for booting, we can check the path required to access the kernel and `initrd`. We can use the normal tab completion on the GRUB shell to see directories and filenames.

# Protecting the GRUB menu with passwords

Now I can imagine that all of this talk to gain root access from the physical server can be quite alarming; the truth is that it really shouldn't be, as securing physical access to the server is normally not difficult or onerous. However, where there is a desire or need to take the security further, it can easily be implemented through GRUB passwords. Any password settings will normally be added to the global section that precedes any stanza. Firstly, let's review some of the GRUB global options before setting some passwords.

On visiting the `/boot/grub/menu.lst` file on CentOS, we will see that the first lines are commented out and generated by the installer **anaconda**, and that the file is named as `grub.conf`.

The `menu.lst` file does exist in Red Hat and CentOS but is in the guise of a symbolic link to `/boot/grub/grub.conf`. From the legacy GRUB documentation, the file should be `menu.lst`; CentOS provides this with the link, but I feel that the file is more logically named `grub.conf`.

For ease of access, a symbolic link `/etc/grub.conf` links through the `/boot/grub/grub.conf` file. The single file can then be accessed as follows:

```
/boot/grub/grub.conf
/boot/grub/menu.lst
/etc/grub.conf
```

The `default` directive will direct GRUB to the default stanza or entry if no choice is made before the timeout.

```
default=0
timeout=5
hiddenmenu
```

Here, we will select the first stanza, stanza 0, if no selection has been made within 5 seconds of the menu loading. The directive, `hiddenmenu`, prevents the menu from showing unless the *Esc* key is pressed. This is especially helpful where there is only one entry in the menu, which is often the case, making good and practical sense.

If you need to prevent users from selecting anything other than what is provided via the menu, then we can add a password to the global settings. This will ensure that, unless the password is entered, only the menu items provided by the menu are available and the option to enter the GRUB shell or append or edit the entries is restricted. The following snippet of code illustrates how this is possible:

```
default=0
timeout=5
hiddenmenu
password=secret
```

If you do not like the idea of a clear text password in the GRUB menu file, then you can use the `grub-md5-crypt` command. You can add the encrypted password as follows:

```
default=0
timeout=5
hiddenmenu
password --md5 <password-hash>
```

You can also add a password directly to a stanza. Adding a password to a stanza ensures that users can only choose that selection from the menu if they know the password. In this way, should you want, you can always have a runlevel 1 entry in the menu but protected by the password as follows

```
title CentOS 6.5 Single User
  password --md5 <password-hash>
  root (hd0,0)
  kernel /boot/vmlinuz.version root=/dev/sda1 resume=/dev/sda2 1
initrd /boot/initramfs.version
```

# Boot splashing with plymouth

As soon as we have begun the boot process and just prior to handing control to the kernel, a boot splash screen can be displayed. This, as the name suggests, controls the splash screen you may see during the boot process. In CentOS, this defaults to the plymouth theme: rings. Plymouth is the boot splash manager; we can use other themes should we wish. Some of these are installed as standard, while others are included in the standard repositories. Yet, more themes can be found in third-party repositories.

You can, of course, build your own theme. Essentially, a minimal theme is just a wallpaper.

## Applying different themes

Most of the time during the boot process, you will not see the splash screen unless CentOS is your desktop machine. However, I would recommend still working with plymouth to change the default splash from rings to basic. With the basic theme, we can see the services loading during the boot process rather than the rings that merely show the boot progress. I would humbly suggest that if you are looking at the server during the boot process, then there are issues and you might want to see the services loading and messages they report back. If you want to be a little more relaxed in your approach, try the theme solar. This shows a planet and some asteroids whizzing around it to illustrate the boot process.

On the command line, we display the default theme as follows:

```
$ plymouth-set-default-theme

```

To display the available themes on the system, we can use the command as follows:

```
$ plymouth-set-default-theme --list

```

CentOS, by default, provides three themes as follows:

*   **details**: This shows us the services as they load
*   **rings**: This is the default theme and includes the CentOS logo with a spinning ring below the logo
*   **text**: This is a blank splash screen with just the horizontal progress bar at the base of the display

These themes are all located in sub directories under the path: `/usr/share/plymouth/themes`. Should we want to change the theme to `details`, we can do so by using the following command. Please note that the command does take a few minutes to run as the process rebuilds the RAM disk to include the new theme.

```
# plymouth-set-default-theme --rebuild-initrd details

```

With this done, you can reboot and even see the difference as the system shuts down. Instead of the infernal rings, we see meaningful messages from our services as they close down.

If we want to be a little more adventurous, then standard CentOS repositories include the additional themes:

*   fade-in
*   solar
*   spinfinity

In order to install and set the theme `spinfinity`, execute the following commands:

```
# yum -y install plymouth-theme-spinfinity
# plymouth-set-default-theme --rebuild-initrd spinfinity

```

A partial screenshot from the spinfinity theme is as shown in the following screenshot:

![Applying different themes](img/5902OS_02_03.jpg)

# Summary

Well, here we are, we have made it to the end of another glorious chapter! You, my dear reader, yes you (there is only one of you), are a little closer to stardom in the Linux Hall of Fame.

We should now have been able to understand that GRUB is the bootloader commonly used in Enterprise Linux, and it will consist of stanzas to boot operating systems. Each stanza consists of three commands. The triumvirate of commands being root, kernel, and initrd. We also made sure we could edit the GRUB menu and solidly protect the GRUB console using passwords that are encrypted and unencrypted.

Finally, we ended up in the paddling pool, the watery shallows of Linux on a summer evening, learning to boot splash with plymouth. This decorated the dawn and dusk of a Linux day with a little color, or a lot of red in the case of spinfinity.

In the next section, we are going to walk into the Linux filesystems within CentOS, gaining an understanding of their makeup and structure. Starting with traditional systems based on disks or logical volumes, we will investigate how filenames relate to inodes and inodes relate to data. We will then move through to links, pipes, and sockets, and finally, finish off by taking a look at the **Better FS** (**btrfs**).