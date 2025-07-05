# Configuring a Secure and Optimized Kernel

The kernel is the core of any operating system, be it Windows or Linux. Linux is technically the kernel and not the complete operating system. Being the core of any operating system, the kernel is installed first and usually requires no manual configuration. Even if there are some kernel level updates to be installed, on a Linux system, it can be installed as a regular application. However, in some situations, compiling the kernel from source with some specific changes might be needed.

However, there might be a few situations where you need to compile the kernel yourself, from the source. These situations include:

*   Enabling experimental features in the kernel
*   Enabling new hardware support
*   Debugging the kernel
*   Exploring the kernel source code

Before you can start building the Linux kernel, you must ensure that a working boot media exists for the Linux system. This can be used to boot into the Linux system, if the boot loader is not configured properly. You will learn how to create USB boot media, kernel source retrieving, configure and build a kernel, and installing and booting from a kernel.

In this chapter, we will cover the following recipes:

*   Creating USB boot media
*   Retrieving the kernel source
*   Configuring and building the kernel
*   Installing and booting from a kernel
*   Kernel testing and debugging
*   Configuring the console for debugging using Netconsole
*   Debugging kernel boot
*   Kernel errors
*   Checking kernel parameters using Lynis

# Creating USB boot media

A USB boot media can be created on any USB media device that is formatted as ext2, ext3, or VFAT format. Also, ensure that enough free space is available on the device, varying from 4 GB for transferring a distribution DVD image, 700 MB in case of a distribution CD image, or just 10 MB to transfer a minimal boot media image. Learning how to create a boot media will be beneficial for readers who are not very experienced with Linux.

# Getting ready

Before starting the steps, you need to have an image file of a Linux installation disk, which you can name as `boot.iso`, and a USB storage device, as specified before.

# How to do it...

To start the procedure of creating the USB boot media, you need to perform the following commands as root:

1.  To install Syslinux on your system, simply execute the following command:

```
sudo apt-get install syslinux
```

2.  You need to install the Syslinux boot loader by executing the following command on the USB storage device:

```
       syslinux /dev/sdb1
```

3.  Now create mount points each for the `boot.iso` file and the USB storage device by executing this command:

```
    mkdir /mnt/isoboot /mnt/diskboot
```

4.  Next, mount the `boot.iso` file on the mount point created for it:

```
    mount -o loop boot.iso /mnt/isoboot
```

In the previous command, `-o loop` option is used for creating a pseudo device that acts as a block-based device. It treats a file as a block device.

5.  Next, mount the USB storage device on the mount point created for it:

```
    mount /dev/sdb1 /mnt/diskboot
```

6.  Once both `boot.iso` and USB storage device are mounted, copy the Isolinux files from `boot.iso` to the USB storage device:

```
    cp /mnt/isoboot/isolinux/* /mnt/diskboot
```

7.  Next, run the command to use the `isolinux.cfg` file from `boot.iso` as the `syslinux.cfg` file for the USB storage device:

```
    grep -v local /mnt/isoboot/isolinux/isolinux.cfg > /mnt/diskboot/syslinux.cfg
```

8.  Once done with the previous command, unmount `boot.iso` and the USB storage device:

```
    unmount /mnt/isoboot /mnt/diskboot
```

9.  Now reboot the system and then try to boot with the USB boot media to verify that you are able to boot with it.

# How it works...

When you copy the required files from the `boot.iso` file to the USB storage media and use the `isolinux.cfg` file from `boot.iso` in the USB storage media as the `syslinux.cfg` file, it converts the USB storage media into a bootable media device that can be used to boot into the Linux system.

# Retrieving the kernel source

Most of the Linux distributions include the kernel sources in them. However, these sources may tend to be a bit out of date. Because of this, you may need to get the latest sources when building or customizing the kernel.

# Getting ready

Most of the Linux kernel developer community uses the **Git** tool for source code management. Even Ubuntu has integrated Git for its own Linux kernel source code, hence enabling the kernel developers to interact better with the community. You can install the Git package using the following command:

```
sudo apt-get install git
```

# How to do it...

The Linux kernel source code can be downloaded from various sources, and here we will talk about the methods used to download from these sources:

1.  We can find the Linux source code as a complete tarball and also as an incremental patch at the official webpage of the Linux kernel:

[http://www.kernel.org](http://www.kernel.org/)

It is always recommended you use the latest version, unless you have a specific reason to work with an older version.

2.  Ubuntu's kernel source can be found under Git. Each release code of the kernel is separately maintained on `kernel.ubuntu.com` in its own Git repository, which is located at:

`git://kernel.ubuntu.com/ubuntu/ubuntu-<release>.git`

It's located here:

`http://kernel.ubuntu.com/git-repos/ubuntu/`

3.  You can clone the repository, using Git, to get a local copy. The command will get modified as per the Ubuntu release you are interested in.
4.  To obtain the precise tree, insert the following command:

![](img/c8d530ca-60e3-40cf-961b-bc07dfe1bfe4.png)

To download any other tree, the syntax of the command will be as follows:

```
    git clone git://kernel.ubuntu.com/ubuntu/ubuntu-<release>.git
```

5.  The downloaded file will be in either GNU ZIP (`gzip`) format or `bzip2` format. After downloading the source file, you need to uncompress it. If the tarball is in `bzip2`, use the following command:

```
tar xvjf linux-x.y.z.tar.bz2
```

If it is compressed GNU ZIP format, use this command:

```
tar xvzf linux-x.y.z.tar.gz
```

# How it works...

Using the different methods mentioned here, you are able to download the source code of Linux kernel. Using any option depends on the user's choice and preference.

# Configuring and building kernel

The need to configure the kernel could arise for many reasons. You may want to resize the kernel to run only the necessary services or you may have to patch it to support new hardware not supported earlier by the kernel. It could be a daunting task for any system administrator and in this section, you will see how you can configure and build the kernel.

# Getting ready

It is always recommended you have ample space for kernels in the boot partition in any system. You should either choose the whole disk install option or set aside a minimum of 3 GB disk space for boot partition. Once you are done with the installation of your Linux distribution and have configured the required development packages, enable sudo for your user account. Now update the system, before you start with installing any packages:

```
    sudo apt-get update && sudo apt-get upgrade
```

After this, you need to install a few packages before getting started. This includes the packages mentioned here:

*   Latest version of `gcc`
*   ncurses development package
*   Packages needed for cross-compiling Linux kernels
*   Package to run make menuconfig

To do so, use the command given here:

```
sudo apt-get install build-essential gcc libncurses5-dev ncurses-dev binutils-multiarch alien bc libelf-dev
```

These packages are used while configuring and compiling the Linux kernel on an `x86_64` system.

# How to do it...

Once you are done with the steps in the *Getting ready* section, you can move on to the process of configuring and building the kernel. This process will take a lot of time, so be prepared:

1.  Download the Linux kernel by visiting[ http://www.kernel.org](http://www.kernel.org) as shown in the screenshot here:

![](img/9777a2d3-8ff1-4bd2-b024-1e9fa1f90d4c.png)

2.  Or you can use the following command:

```
wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.14.47.tar.xz
```

![](img/9864d311-835c-49f0-974d-ea46271f55c6.png)

3.  When the download is completed, move to the folder where the download has been saved. The command to do this will be:

**![](img/1151a91e-8c8d-4cea-b134-0774a069b4be.png)**

4.  Now extract the downloaded tar file to `/usr/src/` using the following command:

![](img/284325cd-e7c7-4e2a-a933-e49453ef86f4.png)

5.  Next, change to the folder where the files have been extracted.:

![](img/8f80d916-2e50-4992-adfe-b8a0a39ae3da.png)

6.  Now run the command to configure the Linux kernel for compiling and installing on the system:

![](img/0628c2cd-007e-406e-bf4b-f850af828070.png)

You may have to use `sudo` before the command if your account doesn't have admin privileges.

7.  Once the previous command is executed, a pop-up window will appear containing lots of menus, as shown here. Select the items of the new configuration:

![](img/2a73f181-5f9b-4084-9ad2-3046ea5f6611.png)

8.  You need to check for the filesystems menu, as shown here:

![](img/c1d33636-aa1c-4ace-9d9e-5058649cdb2d.png)

9.  Under the menu, check whether `ext4` has been chosen or not, as shown in the screenshot. If it is not selected, you need to select it now:

![](img/23002b47-a21f-439a-82c2-0bd4ff72da3e.png)

10.  And then provide a name and save the configuration:

![](img/86d05651-8a37-4e5f-bc5e-9e4c605b4c15.png)

11.  Now compile the Linux kernel. The compile process will take around 40 to 50 minutes to complete, depending on the system configuration. Run the command as shown here:

```
    make -j 5 
```

The output for the preceding command is as shown in the following screenshot:

![](img/5070db44-4f22-466b-859f-4bc8de0e608a.png)

# How it works...

You first download the Linux kernel source and then, after extracting it to a particular location, you configure the kernel for the compilation process.

# Installing and booting from a kernel

After having spent a lot of time configuring and compiling the kernel, you can now start the process of installing the kernel to the local system.

# Getting ready

Before starting with the installation of the kernel, make sure you back up all your important data on the system. Also make a copy of `/boot/` onto external, storage which is formatted in the FAT32 filesystem. This will help repair the system if the installation process fails for any reason.

# How to do it...

Once the kernel is compiled, you can start following the commands here to proceed with installing the kernel:

1.  Install the drivers by running the command, if activated as modules. The command will copy the modules to a sub-directory of `/lib/modules`:

![](img/4317fad9-bc0b-4bc0-a3ee-48bb623fb200.png)

2.  Now run the following command to install the actual kernel:

```
    make install 
```

The preceding command will show the following output:

![](img/9ea1de45-8e6f-46c8-851a-5eb2af16b026.png)

This command executes `/sbin/installkernel`. The new kernel will be installed into `/boot/vmlinuz-{version}`. If a symbolic link already exists for `/boot/vmlinuz`, it will be refreshed by linking `/boot/vmlinuz` to the new kernel. The previously installed kernel will be available as `/boot/vmlinuz.old.` The same will happen for the config and `System.map` files.

3.  Next, copy the kernel to `/boot` directory by running the following command:

```
    cp -v arch/x86/boot/bzImage /boot/vmlinuz-4.1.6
```

The preceding command will show the following output:

![](img/deca58cc-a89d-44b4-b703-28f49ef2fef3.png)

4.  Now make an initial RAM disk:![](img/2ed5305c-9d31-4a41-b031-f2f1f5819135.png)

5.  Next, we need to copy `System.map`, which contains the list of kernel symbols and their corresponding address. Run the given command to do this appending the kernel's name in the destination file:![](img/57756179-2cb7-45d8-9be4-123c6a0612ab.png)

6.  Next, create a `symlink /boot/System.map` file, which will point to `/boot/System.map-YourKernelName`, if `/boot` is on a filesystem that supports `symlinks`:![](img/40c27f8a-0fcd-4836-9b4a-72be98718abe.png)

If `/boot` is on a filesystem that does not support symlinks, just run `cp /boot/System.map-YourKernelName /boot/System.map`

# How it works...

Once the kernel is configured and compiled, it can be installed on the system. The first command will copy the modules to a sub-directory of `/lib/modules`. The second command executes `/sbin/installkernel`. Also, the new kernel will be installed into `/boot/vmlinuz-{version}`. While doing this, if a symbolic link already exists for `/boot/vmlinuz`, it will get refreshed by linking `/boot/vmlinuz` to the new kernel. And the previously installed kernel will be available as `/boot/vmlinuz.old`. The same will happen for the config and `System.map` files. Once everything is done, we can reboot the system to boot from the new kernel:

![](img/8e74b13e-2b3f-4eb5-ae30-49d2ca3c6184.png)

# Kernel testing and debugging

An important part of any open or closed software development cycle is testing and debugging. And the same applies to the Linux kernel. The end goal of testing and debugging is to ensure that the kernel is working in the same way as earlier, even after installing a new kernel source code.

# Configuring console for debugging using netconsole

One of the biggest issues with the Linux kernel is kernel panic. It is similar to the *Blue Screen of Death* for Microsoft Windows operating systems. If the kernel panics, it will dump a lot of information on the screen and just stay there. It is very difficult to trace a kernel panic if the system is rebooted as no logs are created for it. To solve this issue, we can use Netconsole. It is a kernel module that helps by logging kernel `printk` messages over UDP. This is helpful with debugging problems when logging on disk fails.

# Getting ready

Before starting with the configuration of Netconsole, you need to know the MAC address of the system, where the UDP packets will be sent. This system can be called the receiver and it may be in the same subnet or a different one. These two cases are described here. Let's look at the first case, when receiver is in the same subnet:

1.  The IP address of the receiver in this example is `192.168.1.4`. We will send the UDP packets to this IP address using this command:![](img/e58fe079-3eec-4960-8007-f1a224a30f46.png)

2.  Now let's find the MAC address of the receiver system by executing the following command. Here, the IP address is of the receiver system:![](img/30f22507-aac8-4fc3-8696-295ec4590484.png)

As you can see in the previous example, `90:00:4e:2f:ac:ef` is the MAC address we need. Let's look at the second case, when receiver is not in the same subnet.

3.  In this case, we need to first find the default gateway. To do so, run the command shown here:![](img/d6d1bcfc-a459-4073-9cf5-f56a0f53e67e.png)

In this case, the default gateway is `192.168.1.1`.

4.  We need to find the MAC address of the default gateway. First, send a packet to the default gateway:

![](img/40ad101a-e4b0-482a-bf4d-8fe00df53d0d.png)

5.  Now let's find the MAC address:![](img/345dffb0-ced9-42e1-89dd-a97ffd388cdc.png)

Here, `c0:3f:0e:10:c6:be` is the MAC address of the default gateway that we need. Now that we have the MAC address of the receiver, we can start with the configuration process of Netconsole.

# How to do it...

To begin with, you need to change the kernel options at the boot time. If you are using GRUB as the bootloader, it will by default boot the kernel with the **quiet splash** option. However, you don't want that to happen, so you need to change the kernel options:

1.  First, create a backup of `/etc/default/grub`:![](img/5566afb3-6e7a-48dc-9ffd-7dc05e7d4e41.png)

2.  Now, open any editor of your choice to edit `/etc/default/grub`:![](img/5c483e39-1732-4a8d-bc64-881b0fc9a18a.png)

Find the line `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash` and replace it with `GRUB_CMDLINE_LINUX_DEFAULT="debug ignore_loglevel"`:

![](img/7acfcab7-3b60-4f79-932c-d606086f70fb.png)

3.  Now run the command to update GRUB accordingly:![](img/a68f99fe-9625-4ee6-9c1d-feb2241a1f6e.png)

4.  Once you are done with these commands, you need to initialize Netconsole at boot time. For this, we first need to know the IP address and the interface of the *sender* system. This can be done by using the following command:

![](img/bd1732a0-7104-4683-9428-80bce6e8925a.png)

You also need the IP address and MAC address of the receiver system, which we have already got in the *Getting ready* section.

5.  Now let's start initializing Netconsole. First, let's get Net­con­sole to load on boot by adding the mod­ule to `/etc/modules`:![](img/e6d3648c-b7ad-4054-b4c0-3dc4821d2170.png)

Next, make sure that it has the proper options configured as well. For this, add the module options to the `/etc/modprobe.d/netconsole.conf` file and run the following command:

![](img/c5735fb3-d161-4e38-a35e-038bde21e7b9.png)

In the previous command, the part that starts with `netconsole` has the following syntax:

```
netconsole=<LOCAL_PORT>@<SENDER_IP_ADDRESS>/<SENDER_INTERFACE>,<REMOTE_PORT>@<RECEIVER_IP_ADDRESS>/<STEP_1_MAC_ADDRESS>
```

We have used `6666` for both the `<LOCAL_PORT>` and `<REMOTE_PORT>`.

6.  Next, we need to set up the *receiver.*

Depending on which version of Linux is being used as receiver, the command to set up the receiver may vary:

```
netcat -l -u 192.168.1.4 6666 | tee ~/netconsole.log
```

Or try it without the IP address, if the previous command doesn't work:

```
netcat -l -u 6666 | tee ~/netconsole.log
```

7.  If you are using a different variant of Linux that has a different version of Netcat, the following error message will be printed when you try the previous commands:![](img/dd9ea211-32b0-49bd-8832-87a4db9cf93c.png)

If you get the error message, you can try this command:

![](img/5831298f-33eb-4113-8055-efc90227018f.png)

8.  Now let the previous command keep running.
9.  Next, you need to check whether everything is working properly. Reboot the sender system and then execute the following command:

![](img/47bcf300-b319-485c-926a-04435d1047ed.png)

10.  Now you need to check the receiver system to see whether the kernel messages have been received or not.
11.  Once everything is done, press *Ctrl* + *C.* Then you can check for the messages in `~/netconsole.log`.

# How it works...

To capture kernel panic messages, we configure Netconsole, which logs the messages over the network. To do this, we need one more system on the network, which serves as receiver. Firstly, we try to find the MAC address of the receiver system. Then we change the kernel boot options. After updating GRUB, we start initializing Netconsole on the sender system that we want to debug. Finally, we set up the receiver system to start receiving the kernel messages.

# There's more...

If you are using a Windows system as receiver, then we can use Netcat for Windows as well, which is available at `http://joncraton.org/files/nc111nt.zip`.

Download the file from the given link and extract it somewhere like `C:\Users\Tajinder\Downloads\nc`.

Now open Command Prompt (Start | `Run` | `cmd`). Then move to the folder where you have extracted Netcat:

![](img/324cadb2-8bae-45d8-8650-5931b0f8cef0.png)

Next, run the following command:

![](img/b9deec73-612a-46ec-ac0e-0b5973b0fc1f.png)

Here, `192.168.1.3` is the same as `<RECEIVER_IP_ADDRESS>`. Let the previous command run and continue with the commands in step 9\. Once it's done, press *Ctrl* + *C*. You will find the messages in `netconsole.txt.`

# Debugging kernel boot

Sometimes your system might fail to boot due to changes within the kernel. Hence it is important that when creating reports about these failures, all the appropriate information about debugging is included. This will be useful for the kernel team in resolving the issue.

# How to do it...

If you are trying to capture error messages that appear during boot, then it is better to boot the kernel with the `quiet` and `splash` options removed. This helps you see the messages, if any, that appear on the screen.

To edit boot option parameters, do the following:

1.  Boot the machine.
2.  During the BIOS screen, press the *Shift *key and hold it down. You should see the GRUB menu after the BIOS loads:![](img/ed56ecff-eea0-4099-aedf-026449a44811.png)

3.  Navigate to the kernel entry you want to boot and press *e*.
4.  Then remove the `quiet`and `splash`keywords (found in the line starting with `linux`).![](img/2e712f2b-4c42-48ef-a0df-2e59b3e94867.png)
5.  Press *Ctrl + X* to boot.

You can see the error messages, if any, on the screen. Depending on the type of error messages you encounter, there are other boot options you could try. For example, if you notice ACPI errors, try booting with the `acpi=off`boot option.

# Kernel errors

Kernel panic or kernel error is a term used when a Linux system has come to halt and seems unresponsive. When the kernel detects an abnormal situation, it voluntarily halts the system activity. When the Linux system detects an internal fatal error from which it cannot recover safely, it generates a kernel panic.

# Causes of kernel errors

In Linux, a kernel error can be caused due to various reasons. Here we will discuss a few of the reasons:

*   **Hardware – Machine Check Exceptions**: This type of kernel error is caused when a component failure is detected and reported by the hardware through an exception. This typically looks like this:

```
System hangs or kernel panics with MCE (Machine Check Exception) in /var/log/messages file.
System was not responding. Checked the messages in netdump server. Found the following messages ..."Kernel panic - not syncing: Machine check".
System crashes under load.
System crashed and rebooted.
Machine Check Exception panic
```

*   **Error Detection and Correction (EDAC):** If any memory chip and PCI transfer error is detected, the hardware mechanism reports it causing EDA errors. This error gets reported in `/sys/devices/system/edac/{mc/,pci}` and typically looks like this:

```
Northbridge Error, node 1, core: -1
K8 ECC error.
EDAC amd64 MC1: CE ERROR_ADDRESS= 0x101a793400
EDAC MC1: INTERNAL ERROR: row out of range (-22 >= 8)
EDAC MC1: CE - no information available: INTERNAL ERROR
EDAC MC1: CE - no information available: amd64_edacError Overflow
```

*   **Non-Maskable Interrupts (NMIs)**: When a standard operating system mechanism is unable to ignore or mask out an interrupt, it is called a **Non-Maskable Interrupt** (**NMI**). It is generally used for critical hardware errors. A sample NMI error appearing in `/var/log/messages` would look like this:

```
kernel: Dazed and confused, but trying to continue
kernel: Do you have a strange power saving mode enabled?
kernel: Uhhuh. NMI received for unknown reason 21 on CPU 0
kernel: Dazed and confused, but trying to continue
kernel: Do you have a strange power saving mode enabled?
kernel: Uhhuh. NMI received for unknown reason 31 on CPU 0.
```

*   **Software – The BUG() macro**: When any abnormal situation is seen indicating a programming error, kernel code causes this kind of kernel error. It typically looks like this:

```
NFS client kernel crash because async task already queued hitting BUG_ON(RPC_IS_QUEUED(task)); in __rpc_executekernel BUG at net/sunrpc/sched.c:616!invalid opcode: 0000 [#1] SMPlast sysfs file: /sys/devices/system/cpu/cpu15/cache/index2/shared_cpu_mapCPU 8Modules linked in: nfs lockd fscache nfs_acl auth_rpcgss pcc_cpufreq sunrpc power_meter hpilohpwdt igb mlx4_ib(U) mlx4_en(U) raid0 mlx4_core(U) sg microcode serio_raw iTCO_wdtiTCO_vendor_support ioatdma dca shpchp ext4 mbcache jbd2 raid1 sd_mod crc_t10dif mpt2sasscsi_transport_sas raid_class ahci dm_mirror dm_region_hash dm_log dm_mod[last unloaded: scsi_wait_scan]
```

*   **Software – Pseudo-hangs**: These type of errors are commonly encountered, when the system appears to be hung, and could have several reasons for this kind of behavior such as:
    *   **Livelock**: When running a real-time kernel, if application load is too high, it could lead the system to a situation where it becomes unresponsive. The system is not completely hung, but appears to be as it is moving so slowly.

A sample error message getting logged in `/var/log/messages`, when the system is frequently hung, looks like this:

```
INFO: task cmaperfd:5628 blocked for more than 120 seconds.
"echo 0 > /proc/sys/kernel/hung_task_timeout_secs" disables this message.
cmaperfd D ffff810009025e20 0 5628 1 5655 5577 (NOTLB)
ffff81081bdc9d18 0000000000000082 0000000000000000 0000000000000000
0000000000000000 0000000000000007 ffff81082250f040 ffff81043e100040
0000d75ba65246a4 0000000001f4db40 ffff81082250f228 0000000828e5ac68
Call Trace:
[<ffffffff8803bccc>]
:jbd2:start_this_handle+0x2ed/0x3b7
[<ffffffff800a3c28>] autoremove_wake_function+0x0/0x2e
[<ffffffff8002d0f4>] mntput_no_expire+0x19/0x89
[<ffffffff8803be39>]
:jbd2:jbd2_journal_start+0xa3/0xda
[<ffffffff8805e7b0>]
:ext4:ext4_dirty_inode+0x1a/0x46
[<ffffffff80013deb>] __mark_inode_dirty+0x29/0x16e
[<ffffffff80041bf5>] inode_setattr+0xfd/0x104
[<ffffffff8805e70c>] :ext4:ext4_setattr+0x2db/0x365
[<ffffffff88055abc>] :ext4:ext4_file_open+0x0/0xf5
[<ffffffff8002cf2b>] notify_change+0x145/0x2f5
[<ffffffff800e45fe>] sys_fchmod+0xb3/0xd7
```

*   **Software – Out-of-Memory killer**: This type of error or panic is triggered when some memory needs to be released by killing a few processes, when a case of memory starvation occurs. This error typically looks like this:

```
Kernel panic - not syncing: Out of memory and no killable processes...
```

Whenever a kernel panic or error occurs, you may have to analyze these errors to diagnose and troubleshoot them. This can be done using the Kdump utility. Kdump can be configured using the following steps:

1.  Install `kexec-tools`.
2.  Edit the `/etc/grub.conf` file, and insert `crashkernel=<reservered-memory-setting>` at the end of kernel line.
3.  Edit `/etc/kdump.conf` and specify the destination for sending the output of `kexec`, that is `vmcore`.
4.  Discard unnecessary memory pages and compress only the ones that are needed by configuring the Core collector.

# Checking kernel parameters using Lynis

Any operating system is as strong as its weakest link. In the case of Linux, any weakness in its kernel would imply a total compromise of the system. Hence it is necessary to check the security configuration of the Linux kernel.

In this topic, we will see how to use Lynis to check for kernel parameters automatically. Lynis has several predefined key pairs to look for in kernel configuration and accordingly provide advice.

# Getting ready

To view or edit any security related parameter of Linux kernel, there is the `/etc/sysctl.conf` file. All the parameters are stored in this file and this is read during boot time. If you wish to see the available kernel parameters in this file, you can do so by running the `command:sysctl -a`. This command will display an extensive list of configuration settings. The kernel security parameters are also in this list. Lynis helps check the kernel security parameters in this file automatically, thus avoiding the hassle of checking each parameter manually. To use Lynis, write access to `/tmp` is needed by the user account running the tool.

# How to do it...

Lynis is an open source security tool that helps with audits of systems running UNIX derivatives such as Linux, macOS, BSD, Solaris, AIX, and others.

Here is the list of following steps:

1.  The first step to using Lynis is to download its package. This can be done from this link:

[https://cisofy.com/downloads/lynis/](https://cisofy.com/downloads/lynis/)

![](img/0018eb17-54da-4764-9e45-3aae1f9de4b2.png)

2.  Once you click on download, you will get a tarball file to save. Save it in any folder on your system:![](img/d11f2c79-6fdf-4952-b8f2-3601b41dfd8e.png)

3.  The next step is to extract the content from the tar file. You do this by running this command:![](img/8781ba0c-d30c-430b-a75f-eb54292914d6.png)

Once the extraction is complete, you will get a directory named `lynis` in the same directory.

4.  Move inside the `lynis` directory and list the files present inside it:

![](img/52b1e1c5-4731-468c-8444-4ca5e3684091.png)

5.  Inside the `lynis` directory, among other files, you see an executable file again named `lynis`. Once you run this file, you will be able to see all the options available for using the lynis tool. So you run the tool as shown:![](img/d219f67e-c7f2-417a-9f91-85e1b48ceecc.png)

5.  To audit the kernel parameters you use the `audit system` parameter. Once you run Lynis using this parameter, it will start auditing the system:

![](img/643312f5-24d3-4016-a60e-1925bc5bff8b.png)

Among the results, you will also get the list of kernel parameters:

![](img/ba9b0b36-87b9-4504-a586-0a1e644efa3d.png)

7.  The kernel parameters will be displayed in the results, as shown here:

![](img/84f24478-2d19-4430-b4a3-3e31704e657d.png)

8.  Based on the OS and kernel version, the number of parameters may change. If any parameter is incorrectly configured, Lynis will inform you and provide suggestions.