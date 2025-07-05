# Chapter 8. Advanced System Configuration

If you have followed everything in this book until now, you are almost an expert in all things FreeNAS. There are just a few more things to learn and your training will be complete. This chapter looks at Advanced System Configuration like disk encryption, adding a swap space, and tweaking FreeBSD.

# Disk Encryption

If the data you are storing on your FreeNAS server is of a sensitive nature (for example military, medical, financial or other confidential data) then it is worth considering using encryption to protect your data should the server fall into the wrong hands.

If security is a top priority for you, then encrypting the data on the disk is only one of several measures you should take to safe guard your data. For example if your building or the people in your building (for example employees) aren't subject to stringent security measures then having your hard disk encrypted is of minimal value. Someone (from within or without) could access your data over the network and copy the sensitive data and then email it to almost anywhere. Having your hard disk encrypted won't stop that from happening.

FreeNAS offers the ability to encrypt an entire disk with a strong level of encryption. If the server should be stolen, then the perpetrators will have a tough time accessing the data on the disk.

Normally, to add a new disk to your FreeNAS server you need to:

1.  1\. Add the disk in **Disks: Add**.

2.  2\. Format the disk in **Disks: Format**.

3.  3\. Add a mount point in **Disks: Mount Point**.

To add a new encrypted disk, almost the same procedure is used except that there is a new step to create an encrypted volume between step 1 and step 2\. The new sequence of events becomes:

1.  1\. Add the disk in **Disks: Add**.

2.  2\. Create an encrypted volume using the previously added disk in **Disks: Encryption**.

3.  3\. Format the disk in **Disks: Format**.

4.  4\. Add a mount point in **Disks: Mount Point**.

Notice that the creation of the encrypted volumes occurs before the disk is formatted. This is because the encryption used by FreeNAS is at a very low level. It is not file-based (meaning that each file is individually encrypted) but rather it is sector-based, which means that each piece of information that is written to the hard disk (including directory names etc) is encrypted.

### Note

**Do You Really Need Encryption?**

Encrypting your data actually increases the chances of data loss. Your data is actually more likely to be lost to encryption misconfiguration or lost keys than to theft. **There is no password recovery system for an encrypted hard drive**. If the password is lost, misplaced, forgotten or the holder becomes unavailable then the data is in actuality lost.

## Encrypting a Disk in FreeNAS

If you are encrypting a disk that has previously had data on it, it is important to completely erase all the old data before using the disk for encryption. This is because when the disk is initialized for encryption, the old data is not physically overwritten and as such, this old data would still be accessible at the sector level of the hard disk should the disk be stolen and analyzed. Unfortunately, FreeNAS doesn't provide a way to do this and you will need to remove the disk from the FreeNAS server and security wipe it in another machine.

To encrypt a disk, first of all, make sure it has been added to the system using the **Disks: Add page**. Then, go to **Disks: Encryption**.

Click on the add circle to add a new encrypted disk. There are four parameters for adding an encrypted disk:

*   **Disk—** Select the disk you want to encrypt. It has to be a whole disk and can not be a partition of a disk. This means that you can't encrypt the second partition of a disk where you have installed FreeNAS on the first partition. Also, the disk must have been previously added in **Disks: Add**.

*   **Encryption algorithm—** You can choose between three different encryption algorithms with which to encrypt your data. If you are unsure, stick with AES as it is the standard encryption method used by the U.S. government, it has been analyzed extensively and is now used worldwide.

*   **Passphrase—** The password. Whenever the disk is mounted on the FreeNAS server (generally after a reboot), then the password needs to be entered to unlock the encrypted drive. Using a good, solid password is essential for the disk encryption to be worthwhile. Don't use your birthday or your daughter's name!

*   **Initialize—** If this disk has never been used as an encrypted disk before, it needs to be initialized and made ready for the encryption process. You need to tick this box unless this is a previously a encrypted disk that you are adding back into the FreeNAS server.

### Note

Initializing the disk will cause all data to be lost on this disk.

Having selected the disk from the drop down box as well as picking an algorithm, you need to enter a good unguessable password and tick the **initialize** box. Now click **Add**. The disk will now be prepared and encrypted. The output will look something like this:

```
Encrypting '/dev/ad1'... Please wait!
Calculating number of iterations...
Done, using 38638 iterations.
Metadata value stored on /dev/ad1.
Done.
Attaching provider '/dev/ad1'.
Attached to /dev/ad1.
Done.

```

The reassuring **Done**. lets you know that all went well. It is always best to double check the output for any errors.

Now that the disk has been set up for encryption, it can be used just like any other disk. You can format it or mount it and also share it on the networking using CIFS, NFS, and AFP etc.

## Entering the Password When You Reboot

Because the volume is encrypted, it needs a password to unlock it and allow it to be accessed. Whenever the FreeNAS server is rebooted, the encrypted volume will not be accessible until you have entered the password.

Once the system is booted, go **to Disk: Mount Point**. You will see an error because FreeNAS can't mount the encrypted volume without the password.

![Entering the Password When You Reboot](img/4688_08_02.jpg)

Now go to **Disk: Encryption**. The encrypted volume has the status **Not attached**. To enter the password, click on the **Tools** tab. Choose the encrypted disk from *Encrypted disk name* drop down list and select the command **attach** (which should be the default). Now enter the password and click on **Send Command!** The output should be something like this:

```
Attached to /dev/ad0.
Done.
Mounting device.
Successful.

```

If you mistyped the password, then the output would be something like this:

```
Wrong key for ad0.

```

Click on the **Management** tab and the disk status will now be shown as **Attached**. Finally, go back to **Disk: Mount Point** and check that the mount point status is **OK**. If it isn't, click on **Retry** (which forces FreeNAS to mount the disk again) and then it should show **OK**.

## Encryption Tools

When the FreeNAS server is rebooted, the password needs to be entered to unlock the volume. To do this, you use **Tools** tab in **Disk: Encryption** (see *Entering the password when you reboot* above). Here is an overview of the other actions that can be performed on an encrypted disk.

### How to Unlock an Encrypted Disk—Attach and Detach

Attach and detach are technical FreeBSD words for unlock and lock. Attach means that the password supplied will be used to open up the disk and set up the necessary decryption parameters. Once successfully attached, the disk is able to be used like any other hard disk.Detach is the opposite. Here, the disk is locked and the data is inaccessible without the correct password. To detach an already attached disk, choose the attached, encrypted disk from *Encrypted disk name* drop down list and select the command **detach** and click on **Send Command!**

### How to Change the Password on an Encrypted Disk—setkey

Remaining on the **Tools** tab in the **Disk: Encryption** page, you can change the password of an encrypted disk by using the setkey command. Choose the encrypted disk from *Encrypted disk name* drop down list and select the command *setkey*. Now enter the old password along with the new password and click on **Send Command!** The output is just a simple **Done**.

### Note

You are not asked to confirm the new password. If you make a mistake in typing in the new password, all your data will be lost as you can not unlock the disk.

### Checking the Status of an Encrypted Disk—list and status

To get some simple status information about your encrypted drives, you can use the `status` and `list` commands.

`status` simply lists which drives are encrypted and in fact, might not even tell you their status! Here is an example output:

```
Name Status Components
ad0.eli N/A ad0

```

The list command is a bit more verbose. An example out would be:

```
Geom name: ad0.eli
EncryptionAlgorithm: AES-CBC
KeyLength: 128
Crypto: software
UsedKey: 0
Flags: NONE
Providers:
1\. Name: ad0.eli
Mediasize: 10262568448 (9.6G)
Sectorsize: 512
Mode: r1w1e2
Consumers:
1\. Name: ad0
Mediasize: 10262568960 (9.6G)
Sectorsize: 512
Mode: r1w1e1

```

The `Geom name:` tells you the name of the encrypted disk. It will be the name of the disk device (say ad0 for the first IDE hard disk) followed by `.eli`, in our example it was: `ad0.eli`. The `EncryptionAlgorithm:` tells you which algorithm is being used (which in this case was AES) and the `KeyLength:` tells you the strength of the encryption.

The provider and consumer are the two ends of the encryption process. In FreeBSD terms, this means the physical hard disk (ad0) and the pseudo device (ad0.eli) which is the hard disk after encryption. As FreeBSD writes to the pseudo version of the hard disk, (ad0.eli) the encryption software applies its algorithms and the encrypted data is written to the real hard disk (ad0). The opposite happens during a read; the encrypted data is read from the hard disk and passed to the decryption software before being passed on higher up.

# Advanced Hard Drive Parameters (S.M.A.R.T)

Self-Monitoring, Analysis, and Reporting Technology, or S.M.A.R.T, is a system for monitoring hard disks to report on a variety of characteristics that pertain to the reliability of the disk. Monitoring these characteristics should (in theory) help anticipate drive failures. According to Seagate, the hard disk manufacturer, mechanical failures (that are usually predictable failures) account for 60 percent of drive failure.

Not all hard drives have S.M.A.R.T capabilities and different manufacturers measure different characteristics and define different thresholds of failure. Essentially each hard disk model defines its health according to the rules set down by its manufacturer. Therefore one model of hard disk may have a different value for a certain characteristic than another model of hard drive and yet both be defined as acceptable by the manufacturer.

The characteristics or attributes of each drive has two values: one is the raw value whose meaning is defined by the drive manufacturer. The other is a normalized value that ranges from 1 to 253 (where 1 is the worst case and 253 the best).

## Enabling and using S.M.A.R.T of the FreeNAS

Before you can use S.M.A.R.T on your FreeNAS server, you need to check if your hard disk supports S.M.A.R.T. Go to **Diagnostics: Information** and click on the **S.M.A.R.T**. tab. The output will show a list of disks (IDE, SATA, SCSI, and even flash disks) along with any S.M.A.R.T information available.

If a particular device doesn't support S.M.A.R.T, the output will simply say:

```
Device does not support SMART

```

If the device supports S.M.A.R.T, it will report more information about the drive including the model number. Here is an example for an aging 10GB Quantum Fireball disk:

```
Device Model: QUANTUM FIREBALLlct10 10
Serial Number: 872001057089
Firmware Version: A03.0900
User Capacity: 10,262,568,960 bytes
Device is: Not in smartctl database [for details use: -P showall]
ATA Version is: 4
ATA Standard is: ATA/ATAPI-4 T13 1153D revision 15
Local Time is: Tue Mar 18 21:22:34 2008 UTC
SMART support is: Available - device has SMART capability.
SMART support is: Disabled

```

The key thing to note here is that *SMART support is: Available—device has SMART capability*. But that *SMART support is: Disabled*.

To enable S.M.A.R.T monitoring for this disk, go to **System: Advanced** and tick the **S.M.A.R.T Daemon** box. This will enable the S.M.A.R.T daemon (monitoring process) and log the status to the log file.

Now, if you return to the **Diagnostics: Information** page and again click on the **S.M.A.R.T**. tab, you will see that the output has changed significantly. The first difference is that *SMART support is: Enabled*. Below the initial summary is now a comprehensive list of different drive attributes pertaining to the reliability of the hard disk.

The first line is normally a report on the overall health of the disk. If it is healthy, it should read something like this:

```
SMART overall-health self-assessment test result: PASSED

```

If the overall health is listed as FAILED, then you need to back up the hard disk immediately and replace it with another one.

Below the overall health check is a list of hard disk-specific information that culminates in a list of *Vendor Specific SMART Attributes with Thresholds*. This list shows each attribute along with its normalized value, the worst value that this attribute has had in the life time of the drive, and the thresholds of the value. When reading these values, you need to remember that 1 is the worst case and 253 the best. For example:

```
Vendor Specific SMART Attributes with Thresholds:
ATTRIBUTE_NAME FLAG VALUE WORST THRESH RAW_VALUE
Raw_Read_Error_Rate 0x0029 100 253 020 0
Spin_Up_Time 0x0027 083 081 020 2195
Start_Stop_Count 0x0032 093 093 008 5025
Reallocated_Sector_Ct 0x0033 100 100 020 0
Seek_Error_Rate 0x000b 100 100 023 0
Power_On_Hours 0x0012 090 090 001 6675
Calibration_Retry_Count 0x0013 100 100 020 0
Power_Cycle_Count 0x0032 093 093 008 4740
Read_Soft_Error_Rate 0x000b 100 100 023 0
UDMA_CRC_Error_Count 0x001a 116 116 000 84
Reallocated_Event_Count 0x0010 100 100 020 0
Current_Pending_Sector 0x0032 100 100 020 0
Offline_Uncorrectable 0x0010 100 253 000 0

```

Looking at the `Raw_Read_Error_Rate`, you can see that its normalized value is `100` but that the raw value is `0`. This means that there have been no read errors on this hard disk that the disk manufacturer has normalized to the value `100`. However, what is important is that the threshold is `20`. So IF read errors started to appear on this drive, then the normalized value would start to shrink until it reached `20`. At which point, the overall health of the drive would be reported as FAILED.

An attribute from a failing hard disk might look like this:

```
ATTRIBUTE_NAME VALUE WORST THRESH WHEN_FAILED
Reallocated_Sector_Ct 136 136 140 FAILING_NOW

```

Notice here that the `when_failing` column reports the drive in the process of failing. Looking at the normalized value, we read `136` and the threshold is `140`. Remembering that the lower the number the worse the situation, we see that this particular attribute has just recently gone passed its threshold and as such triggered the failure warnings.

On the FreeNAS server, when an attribute passes its threshold it will be reported in the SMARTD log on the **Diagnostics: Logs** page.

Here is a list of some key S.M.A.R.T attributes that if they pass their threshold, the disk is in a critical state:

| Attributes | Meaning |
| --- | --- |
| Read Error Rate | Measures the rate of read errors that occurred when reading data in the disk. |
| Reallocated Sectors Count | The number of reallocated sectors. When the hard drive finds a sector with an error it marks this sector as "reallocated" and transfers data to a special reserved area. If the number reallocated sectors increases too much, the disk is starting to fail. |
| Spin Retry Count | Number of retries of spin start attempts. This is the total number of the spin retries. An increase of this number is a sign of a mechanical problem. |
| Uncorrectable Sector Count | This is total number of uncorrectable errors when reading or writing to a sector. If this number starts to increase, it can mean that there is a problem with the hard disk's magnetic surface. |

# File System Consistency Check—FSCK

The FreeNAS server has a tool to verify if the file system on a disk is healthy. This is different than checking the S.M.A.R.T status of the disk in that S.M.A.R.T is at the hardware level and support for it is provided by the disk manufacturer. However, FreeBSD/FreeNAS writes data to the disk in a certain order and using a special structure that enables it to find files and folders. This special structure is called a file system and it is important to verify (from time to time) that the file system is intact and doesn't contain any errors. File system errors most often occur when the FreeNAS server is switched off without a proper shutdown. This can mean that the file system is left in a state where write operations were queued or cached and they were never actually completed. What remains therefore is a file system inconsistency. The FreeBSD tool for checking the file system consistency is called fsck (File System Consistency checK).

To run a file system consistency check, go to **Disks: Mount Point** and click on the **Fsck** tab. First, you need to select the disk to check from the drop down box. Next, you need to decide how you want to run the file system check.

If the **Unmount disk/partition** is not ticked then fsck is run in read-only mode. Here the file system is checked for errors but any errors are reported but **not** corrected. Running a file system check on a busy disk can reduce performance.

If **Unmount disk/partition** is ticked then the disk will first be unmounted and all errors will be fixed automatically. During the file system check, the data on the disk won't be available to the network users.

The output for fsck will look something like this:

```
** /dev/ad0p1 (NO WRITE)
** Last Mounted on /mnt/store
** Phase 1 - Check Blocks and Sizes
** Phase 2 - Check Pathnames
** Phase 3 - Check Connectivity
** Phase 4 - Check Reference Counts
** Phase 5 - Check Cyl groups
317 files, 17249 used, 4833873 free (121 frags, 604219 blocks, 0.0% fragmentation)
Successful

```

### Note

**Large Disks and fsck**

Depending on the hard disk size, running fsck can take from minutes to hours. Therefore, on large hard disks, running fsck from the web interface can give you a time-out. fsck will fix all errors on the file system but you will not see the output on web interface.

Also, note that fsck requires a large amount of RAM to run. For large volumes (2TB or greater), you should have a minimum 512MB RAM.

It is also possible to run the fsck tool from the command line. This might be useful for large hard disks if the web interface times out while waiting for the command to complete. Please see chapter 10 for more details.

# Advanced OS Tweaking

The underlying operating system of FreeNAS is FreeBSD. Like all complex systems, FreeBSD has a number of configuration parameters that can change its behavior. At the heart of the FreeBSD system, is its kernel and the kernel can be 'tweaked' to perform better under certain situations. The FreeNAS developers have defined a set of kernel parameters that can be tweaked at the click of the mouse. These parameters are as follows:

| Parameter | Normalvalue | Tweakedvalue | Meaning |
| --- | --- | --- | --- |
| `net.inet.tcp.delayed_ack:` | 1 | 0 | This tells FreeBSD to attempt to include TCP ACK information on a data packet instead of sending additional packets to signal the end of a connection. |
| `net.inet.tcp.sendspace:` | 32768 | 65536 | This along with `net.inet.udp.recvspace` define the maximum network packet size. Increasing the packet size can increase network performance, but it also increases memory usage. |
| `net.inet.udp.recvspace:` | 42080 | 65536 | See above. |
| `net.inet.udp.maxdgram:` | 9216 | 57344 | This is the maximum outgoing UDP datagram size. Increasing it can improve network performance but also increased memory usage. |
| `net.local.stream.recvspace:` | 8192 | 65535 | This and the .sendspace below are further buffer tweaks for networking. Performance can increase but so will memory usage. |
| `net.local.stream.sendspace :` | 8192 | 65535 | See above. |
| `kern.ipc.maxsockbuf:` | 262144 | 2097152 | This is the maximum combined buffer size for both sides of a TCP socket. It is increased along with the other network related parameters to boost network performance. |
| `kern.ipc.somaxconn:` | 128 | 8192 | This controls how many simultaneous connection attempts the system will try to handle. |
| `kern.ipc.maxsockets:` | 3072 | 16424 | This is the total number of sockets available on the system. You need one socket for every network connection. |
| `kern.ipc.nmbclusters:` | 3072 | 60000 | This controls the number of *mbufs* allocated by the system An *mbuf* is a chunk of kernel memory used for networking. This is increased in line with other network buffer increases. |
| `kern.maxfiles:` | 1064 | 65536 | The maximum number of files that the system can have open for reading or writing at any one time. |
| `kern.maxfilesperproc:` | 957 | 32768 | This is the maximum number of files a single process can open. |

Enabling Kernel Tuning will result in two things. First, a probable increase in the performance of the FreeNAS server and second, an increase in the amount of memory used by the FreeNAS server. If your system has sufficient memory (256MB or more) and your server experiences heavy network traffic, try enabling Kernel Tuning.

To enable it, go to **System: Advanced** and tick the **Tuning** box. Click **Save** to apply the changes.

# Tweaking the Network Settings

The FreeNAS server has a couple of advanced sections for controlling the network. The first is the global network configurations for each network installed in the machine and the second is the ability to add static routes to the network routing table.

## MTU, Device Polling, Speed, and Duplex

On the **Interfaces: LAN** page (where LAN represents the default network card), there are four parameters that can be changed to increase the performance of your network.

| Parameters | Meaning |
| --- | --- |
| MTU | The Maximum Transmission Unit (MTU) is the size (in bytes) of the largest packet that can be sent on your network. A higher MTU means higher bandwidth efficiency. For an 10/100 Ethernet network, 1500 is the largest allowed MTU. For a Gigabit Ethernet network (with Jumbo Frame support), 9000 is best (and maximum) option. |
| Device polling | Device polling is a technique that lets the system periodically poll network devices for new data instead of relying on interrupts. This can reduce CPU load and therefore increase throughput, at the expense of a slightly higher forwarding delay (the devices are polled 1000 times per second). Not all network cards support polling. |
| Speed | The speed of your network card should be automatically selected (autoselect) but if you find that it is not selected, then you can manually select from: 10baseT/UTP, 100baseTX, 1000baseTX, and 1000baseSX. |
| Duplex | If your network card and the switch to which it is connected are capable of supporting full duplex communications (meaning the card and switch can send while receiving and visa-versa) then you can set it here. |

### Note

Avoid Duplex Mismatch

Duplex mismatch occurs when two connected devices operate in different duplex modes (one in half duplex while the other is in full duplex). The result of a duplex mismatch is a network that is not completely 'broken' but is incredibly slow. Duplex mismatch may occur from wrongly manually setting two connected network interfaces at different duplex modes, and also from connecting a device that performs auto-negotiation to one that is manually set to a full duplex mode. The auto-negotiating device will assume half duplex if it fails to negotiate.

## Adding a Static Route

If you are using two or more network cards, it can sometimes be of benefit to add a static route to the networking routing table. The network routing table defines to who each network connection is routed, or in other words which path it takes across the network. A default route is defined when you configure the default gateway in either the console menu or the Interfaces section of the web interface. By defining the default gateway, you are creating a default path for all traffic that is not destined for the local network. It is assumed that the router (which can be another PC or a DSL modem as well as a network router) can correctly direct the traffic to its destination. If you have more than one network card or you have a second router that handles traffic for a particular part of your network, it can be beneficial to add a static route telling the FreeNAS server to direct all traffic for that particular network to the second router rather than to the default gateway.

To add a static route, go to **System: Static routes** and click on the add circle. There are three mandatory fields to complete:

| Parameter | Meaning |
| --- | --- |
| Interface | Specify which interface this route applies to. |
| Destination network | Destination network for this static route. |
| Gateway | Gateway to be used to reach the destination network. |

The destination network takes the form of an IP address in dot notation (or more specifically a subnet address in that the ending digits will be 0 depending on the network mask) followed by a network mask specified as with 8, 16 or 24 for 255.0.0.0, 255.255.0.0 or 255.255.255.0, respectively.

For example, if the IP address of my FreeNAS server is 192.168.1.250 and there is a router other than the default gateway that can router traffic to the network 192.168.99.0 (meaning all machines with the IP address from 192.168.99.1 to 192.168.99.254), and that this router has an IP address of 192.168.1.123 then the settings would be:

Destination network: 192.168.99.0/24

Gateway: 192.168.1.123

Click **Add** and apply the changes to add the static route.

![Adding a Static Route](img/4688_08_01.jpg)

# Using Wireless

If you have a wireless network card that is supported by FreeNAS, then you can configure it to offer wireless access to the FreeNAS server.

To configure the wireless card, go to the **Interfaces: Management** page. Here, you will be able to configure the card and set the various wireless parameters like the Service Set Identifier (SSID) and the channel number. You can also configure the wireless security with Wired Equivalent Privacy (WEP).

# Adding a Swap File

Some operations on the FreeNAS server, most notably using the iSCSI target and running fsck for big hard disks, require a minimum of 256MB of RAM (Random Access Memory). If your system does not have 256MB of RAM, you can use a swap file to temporarily extend the system's memory.

FreeBSD divides its physical RAM into chucks of memory called pages. Swapping is the process whereby a page of memory is copied to a swap file, to free up that page of memory. The combined sizes of the physical memory and the swap file is the amount of virtual memory available.

Swapping is necessary when the system requires more memory than is physically available; the kernel (the core of FreeBSD) swaps out less used pages and gives memory to the current application (process) that needs the memory immediately.

However, swapping does have a downside. Compared to memory, disks are very slow. Accessing the disk can be tens of thousands times slower than accessing physical memory. The more swapping that occurs, the slower your system will be.

Before adding a swap file, it is best to consider the option of adding more memory to your system.

To add a swap file go to **System: Advanced** and click on the **Swap** tab. To enable the use of a swap file, tick the **Enable** box in the title bar. Select the disk you wish to use to host the swap file, it will be listed by mount point name in the **Mount to use for swap** drop down box. Now, enter the amount of swap space you require. 256MB will be more than enough (note that you don't need to enter the MB part, just 256).

Once you click on **save**, a file called `swap_file` will be created on the specified disk and it will be used for swapping.

To double check that your swap file is configured correctly go to **Diagnostics: Information** and click on the **Swap** tab. The output should look something this:

```
Swap Status:
Device 512-blocks Used Avail Capacity
/dev/md0 524288 0 524288 0%

```

This shows that there is a 256MB swap file in use (524288 divided by 2 as it is shown in 512 byte blocks not in Kilobytes). Currently, all the swap space is available as none of it is being used.

# Enabling Secure Shell Connections (SSH)

In chapters 9 and 10, it may be necessary to make a connection to the FreeNAS server and use the command line of FreeBSD. The command line is often referred to as a terminal or a shell (name after the command line interpreter of the same name). Secure SHell (SSH) is a way to use the command line on the FreeNAS via an encrypted connection and so allows you to use the FreeNAS server without the danger of others, who are snooping around on your network, discovering your passwords.

By default, SSH access is disabled to enable it go to **Services: SSHD** and enable the **SSH Daemon** (server) by ticking **Enable** in the title of the configuration data table. Click **Save and Restart** to finish start the SSH server.

By default, SSH only allows local FreeNAS users to log in if they have been granted **Full Shell** access. Go to **Access: Users and Groups**. If you don't have any users created, then create one (after first creating a group). When you create the user, make sure the **Full Shell** box is ticked. This tells FreeNAS that this user is allowed to connect to the server via SSH and have access to the command line. For more details on user management, see chapter 5.

If you already have a user created who you wish to grant **Full Shell** access, then click the **edit** button next to the user name (an 'e' in a circle) and tick the **Full Shell** attribute. Click **Save** to store the new settings and apply the changes.

In FreeBSD, there are two categories of users, one is the normal user who has limited privileges (for example they can't stop or start services) and the other is the administrator or in FreeBSD terms is know as **root**. Root is a superuser who can do anything on the server.

## Allow Root Login

For troubleshooting and using FreeBSD, the most useful user is root. By default, root is not allowed to log in to the FreeNAS server via SSH as it can pose a security risk. To allow root to log in go to **Services: SSHD** and tick **Permit root login**. Then click **Save and Restart** to finish start the SSH server.

### Note

**Root Password**

The root password is the same as the web interface password, which by default is *freenas*. If the web interface password is changed, so does the root password.

## Types of SSH Authentication

SSH has two types of authentication method. The first is what it calls *keyboard-interactive authentication*, which in normal English—means you type in a username and password to log in. The second type is known as *public key authentication* Here, a system known as public-key cryptography is used to enable a remote SSH client to log in to the FreeNAS without a password.

In public key cryptography, there are two keys called a private key and a public key. The magic behind public key cryptography is that you are free (in fact, encouraged) to give out your public key to anyone who wants it. Then that person can encrypt some information using the public key and once encrypted, only the private key can unlock it. Someone with your public key cannot unlock a message created with your public key.

What this means for SSH is that if the FreeNAS has a copy of your public key, it can correctly authenticate you as the holder of the private key. Thus, a secure connection is made between you and the FreeNAS server and you can use the command line knowing that your passwords and commands cannot be seen.

To get this working with the FreeNAS server, a few simple steps need to occur:

1.  1\. A public and private key need to be generated.

2.  2\. The public key needs to be copied to the FreeNAS server.

3.  3\. You connect to the FreeNAS server and by exchanging of data encrypted using your public key and decrypted using your private key, secure communications are established.

The following example is for Apple OS X and Linux. If you are using Linux, you need to be sure that the OpenSSH package is installed (which it will be by default on most Linux distributions).

First, a public and private key pair need to be generated. This is done with the *ssh-keygen* command:

```
ssh-keygen

```

The output will look something like this:

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/gary/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/gary/.ssh/id_rsa.
Your public key has been saved in /Users/gary/.ssh/id_rsa.pub.
The key fingerprint is:
18:fa:6b:68:c7:9f:49:80:bb:0a:1a:2e:15:86:99:e6 gary@apple-mac.local

```

Here, on my Apple Mac, two files are created `id_rsa` and `id_rsa.pub`. The `.pub` file is the public key and can be freely distributed. The `id_rsa` file is the private key and needs to be protected. In this example I have not used a password. This means that to start the decryption process, no password is needed. The benefit of this is that the client SSH program can connect to the FreeNAS server and it will be authenticated without any input from the user. The disadvantage is that if someone else takes your private key (`id_rsa`) they can also do the same thing, completely unhindered. It is therefore recommended that if you need extra security, you should include a password for your private key.

The next step is to tell the FreeNAS server your public key. The best way to do this is to copy it to the server using the SCP program. SCP stands for Secure Copy and it is a way to copy files in a secure manner from one machine to another. The `id_rsa.pub` file needs to be copied to the `/root/.ssh/authorized_keys2` file on the FreeNAS server.

The main problem is that the `/root/.ssh` directory doesn't exist and SCP can't copy a file to a non-existent directory. There are several solutions to this, of which the simplest is to go to the FreeNAS console (and if needed press *ENTER* to remove the logo and see the menu). Now choose **6) Shell**. At the # prompt type:

```
mkdir .ssh

```

Now to copy `id_rsa.pub` file to `/root/.ssh/authorized_keys2` file on the FreeNAS server with SCP you need to enter:

```
scp ~/.ssh/id_rsa.pub root@192.168.1.250:.ssh/authorized_keys2

```

where 192.168.1.250 is the address of your FreeNAS server. You will be asked to enter the root password.

The final step is to connect to the FreeNAS server using SSH:

```
ssh -l root 192.168.1.250

```

The `-l` option specifies who to log in as (in this case root) and 192.168.1.250 is the IP address of the FreeNAS server.

When you are connected, you will see something like this:

```
Last login: Wed Mar 19 14:20:52 2008 from 192.168.1.249
Copyright (c) 1980, 1983, 1986, 1988, 1990, 1991, 1993, 1994
The Regents of the University of California. All rights reserved.
freenas:~#

```

# Summary

In this chapter, we have looked at advanced system configuration including disk encryption, S.M.A.R.T and SSH access.

The next chapter is about troubleshooting and helping you solve the most common FreeNAS problems.