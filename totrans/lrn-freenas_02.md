# Chapter 3. Exploring FreeNAS

It is time to install the FreeNAS server and start looking at the basic configuration. This chapter is divided into 5 main sections:

*   Quick start guide for the impatient

*   A detailed overview of installation

*   Basic configuration

*   Install to a hard disk

*   Upgrading FreeNAS from a previous version.

# Downloading FreeNAS

Before you can install the FreeNAS server, you will need to download the latest version from the FreeNAS website ([http://www.freenas.org](http://www.freenas.org)). Go to the download section and find the latest "LiveCD" version. The LiveCD version is what is known as an ISO image file and will have the `.iso` file extension. An ISO image is an exact copy of the structure and data for a CD or DVD disk. Using a CD burning program, you can create a FreeNAS bootable CD. We will look at this in more detail later on.

# What Hardware Do I Need?

In this chapter, we will start exploring FreeNAS, so you will need a machine on which to install the FreeNAS software. At this point in time, it doesn't have to be the final machine you are going to use as the FreeNAS server. You can use a "test" machine now and having learnt all about FreeNAS, you can build, install, and deploy a production machine (or machines) later.

So, what we need now is a PC with at least 96Mb of RAM (but 128Mb or more is recommended), a bootable CD-ROM drive, a network card, one or more hard disks, and either a floppy disk drive (and a blank formatted disk) or a USB flash disk (MS-DOS formatted and empty).

The hard disk will be for the data that you want to store and the floppy disk or USB flash disk will be for storing the configuration information.

For the installation and initialization stages, you will also need a monitor and keyboard (but not mouse) attached to the PC. You can remove the monitor later, once FreeNAS is up and running.

## Warning

FreeNAS boots as a LiveCD, which means that it does not use the disks on the host machine during boot up. However, when you start to configure storage on the FreeNAS server (specifically, when you format drives) all the data on the disk will be LOST. Do NOT use a machine that contains important data or an operating system that you will need afterwards.

### Note

**Virtualization & VMWare**

The average PC runs just one operating system and inside that operating system, you would run your applications like word processing and email. There is a technology (called virtualization), which allows PCs to run more than one operating system, or to be more precise, to allow a guest virtual PC to run inside your actual PC. This virtual PC is an independent software box that can run its own OS and applications as if it were a physical computer. A virtual PC behaves exactly like a physical PC and has its own virtual CPU, RAM, hard disk, and network interface card (NIC).

You can install FreeNAS on a virtual PC and FreeNAS can't tell the difference between the virtual PC and any other physical machine, also, it appears on the network just as a real PC would, running FreeNAS.

There are lots of virtualization products available for Windows, Linux, and Apple OS X today. You can learn more at Wikipedia [http://en.wikipedia.org/wiki/Virtualization](http://en.wikipedia.org/wiki/Virtualization)

A very popular virtualization solution is from VMWare ([http://www.vmware.com](http://www.vmware.com)). VMWare have both commercial and freeware offerings and there are pre-configured FreeNAS images available for the VMWare range of products. This makes it an ideal environment for testing the FreeNAS server.

# Quick Start Guide For the Impatient

If you are comfortable with burning ISO images to CDs, setting your computer's BIOS to boot from CDROM, disk partitions, and TCP/IP networking then this little guide should help you get a simple version of the FreeNAS server up and running in just a few minutes.

If, however, some of these things sound daunting, then skip this section and go on to the next one where we shall go through the installation process one step at a time.

For this example, we will use a USB flash disk to store the configuration information. You can use a floppy but be careful that during the boot process, the PC doesn't try to boot from the floppy before it boots from the CDROM.

## Burning and Booting

Once you have downloaded the ISO image file from the FreeNAS website, you need to burn it to a CD. Having done that, put the CD into the PC as well as the flash disk and switch it on. Make sure that the BIOS is set to boot from CD. If it isn't, you need to enter into the BIOS and configure it to boot from CD. On many modern PCs, it is possible to select the boot device at start-up by pressing a special key (which is often either F8 or F12) to show a boot device menu. You can then select the CD as the boot device.

The boot process is in four distinct parts:

1.  1\. First, the PC will go through its POST (Power On Self Test) sequence. Here, the PC will check the amount of memory installed (which you can often see being counted on the screen) and which devices are connected (like hard drives and CDROMs).

2.  2\. It should then start to boot from the CD. Here, FreeBSD (the underlying OS of FreeNAS) will start to boot, this is recognisable by the simple spinning wheel (made up of simple text characters like | - / and \, which are animated to give the appearance of spinning).

3.  3\. The third step is the FreeNAS boot menu. This will appear for just a few seconds and you should just let it boot normally, which is the default.

4.  4\. The final stage is when the FreeNAS logo appears and the system will boot as FreeNAS server. You can tell when the system is fully loaded because the PC speaker will make some short but melodious beeps.

To enable access to the web interface, the network of the FreeNAS server must be configured. Press the SPACE bar on the keyboard and the FreeNAS logo will disappear and a simple text menu will appear.

![Burning and Booting](img/4688_03_01.jpg)

There are two aspects to configuring the network, first, you need to choose which network card to use and second, you need to assign it an address. If you have only one network card in your machine, then the FreeNAS server should have found it and automatically assigned it to be the LAN (Local Area Network) interface.

### Note

**What If My Network Card Isn't Found?**

This probably means that the network card in your machine isn't supported by FreeNAS or more specifically, by FreeBSD. You will need to replace the card with one supported by FreeBSD. Check the FreeBSD hardware compatibility page for more information:[http://www.freebsd.org/releases/6.2R/hardware-i386.html](http://www.freebsd.org/releases/6.2R/hardware-i386.html)

If you see something like this:

![Burning and Booting](img/4688_03_02.jpg)

then the network has been recognised and assigned automatically by FreeNAS.

The default IPv4 address for FreeNAS is 192.168.1.250, if this is good for your network, then you can just leave it unchanged. However, if you need to change it then press 2 followed by *ENTER*. If you want the machine to get its address from DHCP (Dynamic Host Configuration Protocol), answer yes (y) to the IPv4 DHCP question, otherwise answer no (n). If you are not using DHCP, you can now enter the desired IP address. Next, you need to enter the subnet mask. For 255.255.255.0, enter 24, for 255.255.0.0 enter 16, and for 255.0.0.0, enter 8\. At this point, you can now skip the default gateway and DNS questions (by just pressing *ENTER*). If you do want to enter a default gateway and DNS server at this point, they will usually be the IP address of your Internet router. We won't be using IPv6 so the simplest thing to do now is just answer *yes* to the "Do you want to use AutoConfiguration for IPv6?" question. This will cause a small delay while FreeNAS tries (and probably fails) to get the IPv6 address but it is simpler than trying to enter the IPv6 address manually!

You are now ready to access the web interface. The FreeNAS web interface can be accessed from any machine on the network with a web browser (including Windows, Linux, and OS X machines). On this client machine, type the address of the FreeNAS server with `http://` in front of it into your web browser. For example:

[http://192.168.1.250](http://192.168.1.250).

## Configuring

The first time you access the FreeNAS web interface, you will be asked for the username and password. The default username is *admin* and the default password is *freenas*.

You should now be in the web interface. To configure some storage space, you need to work with "Disks". The logical order of working is that disks must be added, then formatted (if need be), then mounted. Finally, access is given to the various mounted disks by configuring different system services like CIFS and FTP.

![Configuring](img/4688_03_03.jpg)

So, to add a disk, go to **Disks: Management**.There is a **+** sign in a circle on the right-hand-side of the page (it can be easy to miss first time), click on it to add a disk. On the next page, select the disk you want to add. If you click on the drop-down menu, you should see the hard disks of the machines, the CDROM, and the USB flash disk.

### Note

**Disk Names in FreeBSD**

The disk naming convention in FreeBSD is:

**/dev/ad0:** *Is the IDE/ATA Primary Master*

**/dev/ad1** *Is the IDE/ATA Primary Slave*

**/dev/ad2** *Is the IDE/ATA Secondary Master*

**/dev/ad3:** *Is the IDE/ATA Secondary Slave*

**/dev/acd0:** *Is the first ATA CD/DVD drive detected*

**/dev/da0:** *Is the first SCSI hard drive, /dev/da1 the second and so on.*

USB flash disks are controlled using the SCSI driver, so they will appear as /dev/daN drives as well.

Make sure ad0 is selected (which it should be by default). The rest of the page you can leave alone. Click **Add** to add the disk to the system. You then need to click **Apply** in order for the changes to take effect. You will now have a table showing you the disk you have added, including its size and a description.

### Note

**Apply**

In FreeNAS, the majority of steps need to be applied (which saves the configuration file to disk) by clicking the *Apply* button. It is normally found near the top of the page before any tables or configuration information is given. If you do not apply the changes, the interface will, on the whole, remember your changes but they will not be enacted in the system. After a reboot, unapplied changes will disappear. It is possible on some pages to make multiple operations and apply them all at the end.

Next, the disk needs to be formatted. In **Disks: Format**, select the disk ad0 (which you just added above). Leave everything else unchanged and click **Format disk**. The disk will then be formatted. The low level output of the format command will be displayed in a box. It should end with **Done!**.

Now the disk needs to be mounted. Go to **Disks: Mount Point**. Click on the **+** in the circle (which I shall refer to as the "add circle" from now on). Leave the **Type** as *Disk* and select the disk *ad0* again. You need to type in a name, *store* is as good a name as any, but feel free to use which ever descriptive name you want to.

### Note

**Be Descriptive**

In setting up and configuring your FreeNAS server, you will be called upon to invent various names for mount points and share names etc. Try to be as descriptive as you can without being long winded. Temp, scratch, blob, and even zob are OK for testing, but try more meaningful names like storeage1, storage60gb or backupstorage etc. Don't use spaces in the names, instead use underline and in general, the names should be no longer than 15 characters.

Although filling-in the description isn't mandatory in the web interface, it is worth using. Once you have completed the form click **Add** and then apply the changes.

## Sharing with Windows Machines

Now that the disk has been added, formatted, and mounted, it is time to share it on the network and give other users the ability to read and write to it. FreeNAS supports many different types of access protocol, for this start guide, we will only look at Microsoft's CIFS protocol that primarily allows Windows machines (but also Apple OS X and Linux machines) to access the storage.

1.  1\. In Services: **CIFS/SMB**, tick the enable box (in the title of the configuration data table). At this point, you can just about leave everything else as is with the exception of the workgroup name. We will be leaving the authentication method as "Anonymous" here as this is the easiest to get working and provides unrestricted read/write access to everyone.

2.  2\. To make sure that the Windows machines are able to find the shared storage, we need to set the workgroup name, on the FreeNAS server, to be the same as the workgroup name of the Windows PC that will access the share. The default workgroup name for Windows Vista is WORKGROUP but note that the default for Window XP Home Edition was MSHOME.

3.  3\. Now click **Save and Restart**. This will save the changes you have made and restart the CIFS service.

4.  4\. Go to the **Shares** tab and click on add circle. Enter a name for the share. Repeating the name of the mount point is probably the safest policy, so in this case, store and also add a comment. Then click **..**. in the **Path** section. This will bring up a simple file system browser. The files you are seeing are on the FreeNAS server and NOT on your local PC. Click **store** and **/mnt/store/** will appear in the little edit box at the click. **OK** it and you will be taken back to the shares page. Now **/mnt/store/** has been added as the path.

5.  5\. Leave everything else as it is and click **Add** and then apply the changes.

So now the first hard disk of the computer is formatted, mounted, and shared to the rest of the network. Now, we will access the share from a Windows Vista machine.

## Testing the Share

You can perform this test from any machine that supports the CIFS protocol including Windows 95/98/ME, Windows 2000/XP, Apple OS X, and Linux. Here, we are going to use Windows Vista.

1.  1\. Open the Network and Sharing Center by clicking **Network** on the Start menu. When the window appears, Vista will automatically scan the network for any shared network resources. When it has finished, you will see the available machines on the network including **FREENAS**.

![Testing the Share](img/4688_03_04.jpg)

1.  2\. Open up the **FREENAS** computer and you will see store, the storage area that you configured. Double click on that and you now "inside" the FreeNAS server from within your Windows machine. Try dragging and dropping a few files in to the store area. Then try deleting them again.

2.  3\. To access the FreeNAS server without using the Network and Sharing Center, click **Start**, and type **\\freenas** and then press *Enter*. This will bring up the shares available on the FreeNAS server directly:

![Testing the Share](img/4688_03_05.jpg)

# Detailed Overview of Installation

It is time to get your hands on a working FreeNAS server and to do that, we need to boot it up onto a PC. There are several steps to this. First, you must burn a CD of the ISO image file you have downloaded. Then, you need to boot the PC from the CD; this may involve changing your computers BIOS to make it boot from the optical drive. Then, you can configure the FreeNAS server to make some storage space available on the network.

When using the LiveCD to boot FreeNAS, there are two types of storage on FreeNAS: data and configuration information. The data will be held on the hard drive of the PC, but the configuration needs to be held on a floppy disk or a USB flash disk. For this example, we will use a USB flash disk to store the configuration information.

## Making the FreeNAS CD

To boot the PC into FreeNAS, you need a CD. The ISO image file you have downloaded contains all the information needed for the CD, but it needs to be written onto a physical CD. This process is often known as *burning* the CD as the laser writes to the disk by heating it and marking or scorching the surface layer.

You need to use a PC with a CD-RW drive and a blank CD-R disk (I recommend using a good brand name CD-R for best results). Download the FreeNAS ISO image on to that machine. The PC with the CD writer should have some CD writing software on it (for example Roxio Easy CD or Nero). If you are familiar with the CD writing software, go ahead and burn the ISO file to the CD-R disk.

### Note

If you aren't familiar with the CD writing software or it doesn't have any CD writing software, then I recommend ISO Recorder. You can download it from [http://isorecorder.alexfeinman.com/isorecorder.htm](http://isorecorder.alexfeinman.com/isorecorder.htm).

## Booting from CD

Put your newly made FreeNAS CD into the CD drive of the machine on which you want to install FreeNAS, and also put the USB flash disk into a USB port. The flash disk will be used to store the configuration data. (You can also use a floppy disk. If you have both a USB flash disk and a floppy inserted, FreeNAS will save the configuration on the USB device). Now, you need to switch on the PC. When a PC starts, it goes through what is known as the Power On Self Test sequence. Here, the PC will check the amount of memory installed in the PC and find the installed hard drives. After the checks, the PC will try and boot from one of the hard drives, the CDROM, the floppy disk or even a USB flash disk. Which device the PC chooses first as its boot device can be changed by a built-in setup program. The setup program lets you modify basic system configuration settings. These settings are stored in a special battery-backed area of the computer's memory that retains the settings even when the power is switched off. During the POST sequence, there is normally a message telling you how to enter into the built-in setup program. It is normally either the *DEL* key or *F2*, on some systems it is also *F10*.

You need to enter into the setup to check and/or change the first boot device to be the CDROM so that the computer will boot into FreeNAS. Each PC has a slightly different setup program, so you will need to search around until you find what you need. The three most popular types of setup programs (also known as BIOS—Basic Input Output Program) are the Phoenix setup program, the Phoenix-Award setup program, and the AMI setup program.

### Note

There are many types of BIOS setup programs and each PC manufacturer modifies the setup program for their own use. The information below is really only a "rough guide" to help you feel your way around. Your BIOS setup program may be significantly different from the examples below. The best source of information is the manual that came with your PC or your motherboard. If you don't have one, most PC manufacturers have them available for download on their websites.

### Phoenix BIOS

If your machine has a Phoenix BIOS, then normally you need to press F2 to enter the setup program. The top of the setup program has a menu that you can navigate with the left and right arrow keys, you need to select the **Boot** menu.

![Phoenix BIOS](img/4688_03_06.jpg)

On the **Boot** menu page, you can move up and down the available boot devices using the up and down arrow keys. You can expand and collapse sections with the **+** or **-** signs using the *ENTER* key. To change the boot order, you use the **+** and **-**keys. You want to make sure that the CDROM is the first device in the list. After you have changed the boot order list, you need to go to the Exit menu (by pressing the right arrow key) and select **Exit Saving Changes**. The PC will then reboot and after the POST, it will start to boot from the FreeNAS CD.

![Phoenix BIOS](img/4688_03_07.jpg)

### Phoenix-Award BIOS

If your PC has a Phoenix-Award BIOS, then normally, you need to press DEL to enter the setup program. Once inside, you can the up, down, left, and right keys to navigate around the menus. Go in to *Advanced BIOS Features* and set the *First Boot Device* to be CDROM by using the **+** and keys. You now need to save your changes and exit. Pressing *ESC* will bring you back to the main menu, then select **Save & Exit Setup**. Often, pressing *F10* will have the same effect. The PC will then reboot and if you have made the changes correctly, it will boot from the FreeNAS CD.

### AMI BIOS

The American Megatrends, Inc (AMI) BIOS normally displays a message telling you to **Hit <DEL> if you want to run setup**. Once inside, it is quite different to that of the setup programs for Phoenix or Award. Here, the *Tab* key is used to navigate and the arrow keys are used to change values. To go from one page to the next, press the *ALT+P* keys. This information should also be printed at the bottom of the BIOS setup page. You need to find the variable *Boot Sequence* and make sure that it is set to boot from the CDROM first.

## First Look at FreeNAS

The boot process is in 4 distinct parts. First, the PC will go through its POST (Power On Self Test) sequence. Here, the PC will check the amount of memory installed (which you can often see being counted on the screen) and which devices are connected (like hard drives and CDROMs). It should then start to boot from the CD. Here, FreeBSD (the underlying OS of FreeNAS) will start to boot, this is recognisable by the simple spinning wheel (made up of simple text characters like | - / and \ which are animated to give the appearance of spinning). The third step is the FreeNAS boot menu. This will appear for just five seconds and you should just let it boot normally which is the default. The final stage is when the FreeNAS logo appears and the system will boot as a FreeNAS server. You can tell when the system is fully loaded because the PC speaker will make some short but melodious beeps.

## Configuring the Network

The majority of the configuration for FreeNAS is done via a web interface, but before you can use the web interface, the FreeNAS server needs to be configured for your network. This is done via a simple text menu system using the keyboard and monitor attached to the PC with FreeNAS running on it. You probably only need to do this once, and after that this new network information will be saved on the USB flash disk (or floppy disk) and the server will boot into this configuration every time.

If you press the SPACE bar on FreeNAS machine, the FreeNAS logo will disappear and a simple menu will appear.

![Configuring the Network](img/4688_03_01.jpg)

Here, you have a number of options including options to reboot or power off the system. The first two options are about configuring the network and they reflect the two parts to configuring the network, first you need to choose which network card to use (option 1) and second you need to assign it an address (option 2).

If you have only one network card in your machine then the FreeNAS server should have found it and automatically assigned it to be the LAN (Local Area Network) interface.

### Note

**What If My Network Card Isn't Found?**

This probably means that the network card in your machine isn't supported by FreeNAS or more specifically by FreeBSD. You will need to replace the card with one supported by FreeBSD. Check the FreeBSD hardware compatibility page for more information: [http://www.freebsd.org/releases/6.2R/hardware-i386.html](http://www.freebsd.org/releases/6.2R/hardware-i386.html)

If you see something like the following screenshot:

![Configuring the Network](img/4688_03_02.jpg)

then the network has been recognised and assigned automatically by FreeNAS.

### What is a LAN IP Address?

IP stands for Internet Protocol and it is the basic low level language that computers use to talk to each other on the Internet. It is also used on private networks (in the office or at home) to connect different PCs and even printers to each other. An IPv4 address is made up of 4 sets of number (0 to 255) and is expressed in what is known as dot notation (meaning that each number has a dot between it). So 192.168.1.250 is an IP address, it also happens to be the default IP address for the FreeNAS server. Like email, the postal service and telephone, each destination (email account, mailbox or handset) needs a unique way of being identified. This is what IP addresses do; they allow each piece of equipment on the network to have a unique identifier so that messages can be addressed to the right place on the network.

### Note

**Pronouncing IP Addresses**

If you need to speak to someone about an IP address, the simplest way is to speak about each digit separately, so 192.168.1.250 isn't "one hundred and ninety two dot" but rather "one nine two dot one six eight dot one dot two five zero".

There are two ways in which you can obtain an IP address for the FreeNAS server. The first is to have the address assigned automatically via the DHCP service (Dynamic Host Configuration Protocol), and the second is to assign it manually.

### Note

**What is DHCP?**

The Dynamic Host Configuration Protocol (DHCP) automates the assignment of IP addresses and other IP parameters (like subnet masks and default gateway). A computer that needs an IP address will send a request to the DHCP server and the server will reply with an IP address from a pool of addresses that have been set aside for this purpose. A DHCP server can be a PC or server (running Windows, OS X or Linux) as well as small devices like modern DSL modems and firewalls.

The advantage of the DHCP method is that the IP address assignment, all happens in the background and you don't need to worry about setting it yourself. The disadvantages are that first you need to have an already configured and running DHCP server on your network; and second, DHCP assigns addresses from a pool of available addresses. This means that every time the FreeNAS server boots, it is not guaranteed to have the same address as it had previously. This isn't a problem when using the CIFS protocol, however, for accessing the web interface or using protocols like FTP, it is desirable to have a stable IP address to refer to. However, for testing the FreeNAS server and learning about how it works using a DHCP assigned address could be acceptable for now.

### Note

It is actually possible to assign fixed, permanent IP address to certain pieces of hardware, including a FreeNAS server over DHCP, but that requires extra advanced configuration changes in the DHCP server that cannot be covered in this book.

So opting for the manual IP address, you now need to obtain two pieces of information. The first is the actual IP address for the FreeNAS and the second is what is known as the subnet mask. The subnet mask will also be expressed in the dot notation and is normally something like 255.255.255.0\. If you are in an office environment, you need to speak to the network administrator and he/she will be able to give you the information you need. If you are administering your own network, you need to choose an IP that isn't currently allocated to any other machine on your network (and also, isn't part of the address pool of any DHCP server on your network).

Having obtained the IP address and subnet mask, you can now configure the FreeNAS server for your network. Select option 2 on the console menu. If you have chosen to have DHCP assign the address, answer yes (y) to the first question about using DHCP for IPv4\. Otherwise answer no (n).

If you are setting the address manually, you can now enter the address in dot notation, i.e. 192.168.1.240\. Next, comes the subnet mask. If your subnet mask is 255.255.255.0: enter 24, for 255.255.0.0: enter 16, and for 255.0.0.0: enter 8\. At this point, you can now skip the default gateway and DNS questions (by just pressing *ENTER*).

We won't be using IPv6 so the simplest thing to do now is just answer yes to the "Do you want to use AutoConfiguration for IPv6?" question. This will cause a small delay while FreeNAS tries (and probably fails) to get the IPv6 address but it is simpler than trying to enter the IPv6 address manually!

After you have successful set the IP address, there will be a small message on the screen inviting you to access the web interface by opening the listed URL in your web browser. If you have used DHCP, note down the URL listed. If you set the IP address manually, check that the URL listed is the same as the IP address you set with [http://](http://) in front of it.

You are now ready to access the web interface.

### Note

**What is IPv4 and IPv6?**

The Internet Protocol has been around since the mid 1980's and when it was designed, the popularity of the Internet was not envisaged. The number of computers connected to the Internet is quickly growing beyond the addressing capabilities of the original protocol. As an answer to this, a new version of the IP protocol has been designed and has been given the name IP version 6 or IPv6 for short and the older version has taken the name IP version 4 or IPv4 for short. FreeNAS supports both versions of the Internet Protocol. In this book, we will concentrate just on IPv4 as it still remains the most popular of the two protocols.

# Basic Configuration

With your FreeNAS server now being up and running, it is time to access the web interface. Open a web browser on a computer on the same network as the FreeNAS server. Enter in the URL of the FreeNAS server. This should be the same as the IP address of the server with [http://](http://) in the front. The default URL is [http://192.168.1.250](http://192.168.1.250).

![Basic Configuration](img/4688_03_08.jpg)

The first time you access the FreeNAS web interface, you will be asked for the username and password. The default username is *admin* and the default password is *freenas*.

![Basic Configuration](img/4688_03_09.jpg)

## FreeNAS Web Interface

You should now have the web interface in your browser. The interface is split into two main sections. Down the left-hand-side are the menus, and the right-hand-side contains the pages for configuration. The menus are split into various sections: System, Interfaces, Disks, Services, Access, Status, Diagnostics, and Advanced.

![FreeNAS Web Interface](img/4688_03_03.jpg)

### Note

When talking about a particular menu item, we shall use the notation Subsection: Menu Item to help you find the right menu option easily. So, the Management option, which is in the Disks subsection, will be referred to as **Disks: Management**.

### System

This section is for system level configuration and operations, here for example you can change the username and password, backup and restore the configuration data, and shutdown or reboot the server.

### Interfaces

Here, you can configure the network of the FreeNAS server much like you did via the console menu. You can change the network card that is used for the web interface and assign permanent or automatic IP addresses.

Be careful when you change things here as some changes won't take effect until you reboot. If you have changed any of the addressing, you will need to access the web interface with the IP address.

### Disks

This section of the menu is for administering the disks on the server. Here, you can set up disk redundancy (RAID), control encryption, format disks, and mount the disks on the server.

### Services

The various access protocols like CIFS, NFS, and FTP are controlled from here. Each service is administered individually and by default NONE of the services are enabled, so before you can access files stored on the FreeNAS server, you need to enable at least one of these services.

### Access

Most of the services offered by FreeNAS use some form of list of users to control who has access and who does not. This section is for defining these users and the groups they belong to as well as connecting the FreeNAS server to other directory services.

### Status

The status menu has several reporting tools for you to see the current state of your FreeNAS server including a general overview, memory usage, disk usage, and network usage. You can also configure emails to be sent periodically about the status of the server.

### Diagnostics

The diagnostics menu contains different tools to help diagnose any problem with the FreeNAS server, including logs of all the important services and diagnostic information from the hard disks and other system modules.

### Advanced

The advanced section provides some simple tools for executing commands at the operating system level and should not be used by those unfamiliar with FreeBSD.

## Adding a Disk

To configure some storage space, you need to work with "Disks". The logical order of working with disks is:

1.  1\. First, FreeNAS needs to be told about the disks it can use. A disk can be a physical hard drive as well as a CD-ROM or an iSCSI target.

2.  2\. Next, the disks need to be formatted (if need be).

3.  3\. After that, the disks need to be mounted. This means that they are made available as working disk space, which can be used by the system. Before being mounted, the disks are not available to be used as storage space.

4.  4\. Finally, the storage space needs to be made available to the network via one of the access protocols, for example CIFS, NFS or FTP. To do this, the respective service needs to be enabled and configured to use the disk.

Here are the steps in detail:

1.  1\. Go to **Disks: Management**.

    *   There is a + sign in a circle on the right-hand-side of the page (which shall be referred to as the "add circle" from now on), click it to add a disk.

    *   On the next page, select the disk you want to add. If you click on the drop down menu, you should see the hard disks of the machines, the CDROM, and the USB flash disk.

        ![Adding a Disk](img/4688_03_10.jpg)

        ### Note

        **Disk Names in FreeBSD**

        The disk naming convention in FreeBSD is:

        **/dev/ad0:** *Is the IDE/ATA Primary Master*

        **/dev/ad1** : *Is the IDE/ATA Primary Slave*

        **/dev/ad2** : *Is the IDE/ATA Secondary Master*

        **/dev/ad3:** *Is the IDE/ATA Secondary Slave*

        **/dev/acd0:** *Is the first ATA CD/DVD drive detected*

        **/dev/da0:** *Is the first SCSI hard drive, /dev/da1 the second and so on.*

        USB flash disks are controlled using the SCSI driver so they will appear as **/dev/daN** drives as well.

    *   Make sure ad0 is selected (which it should be by default).

    *   The rest of the page you can leave alone.

    *   Click **Add** to add the disk to the system.

    *   You then need to click **Apply** in order for the changes to take effect. You will now have a table showing you the disk you have added including its size and a description.

        ![Adding a Disk](img/4688_03_11.jpg)

2.  2\. Next, the disk needs to be formatted. Beware all the data on this disk will be lost. In **Disks: Format**, select the disk ad0 (which you just added above). Leave everything else unchanged. By default, this means we are formatting the disk using UFS, which is the native file system for FreeBSD. You can enter a volume name if you desire. Click **Format disk**. The disk will then be formatted. The low level output of the format command will be displayed in a box. The time it takes to format the disk depends on the size of the disk. It should end with **Done!**

3.  3\. Now the disk needs to be mounted. Go to **Disks: Mount Point**. Click on the add circle. Leave the **Type** as *Disk* and select the disk *ad0* again. You need to type in a name, *store* is as good a name as any, but feel free to use which ever descriptive name you want to.

    ### Note

    **Be Descriptive**

    In setting up and configuring your FreeNAS server, you will be called upon to invent various names for mount points and share names etc. Try to be as descriptive as you can without being long winded. Temp, scratch, blob, and even zob are OK for testing but try more meaningful names like storeage1, storage60gb or backupstorage etc.

    Although filling-in the description isn't mandatory in the web interface, it is worth using. Once you have completed the form click **Add** and then apply the changes.

    ![Adding a Disk](img/4688_03_12.jpg)
4.  4\. Now that the disk has been added, formatted, and mounted, it is time to share it on the network and give other users the ability to read and write to it. FreeNAS supports many different types of access protocols. First, we shall have a look at Microsoft's CIFS protocol that primarily allows Windows machines (but also Apple OS X and Linux machines) to access the storage. Then, our second test will be to configure the FTP protocol.

## Accessing the Disk via CIFS

1.  1\. In Services: **CIFS/SMB**, tick the enable box (in the title of the configuration data table). At this point, you can just about leave everything else as is with the exception of the workgroup name.

2.  2\. To make sure that the Windows machines are able to find the shared storage, we need to set the workgroup name, on the FreeNAS server, to be the same as the workgroup name of the Windows PC that will access the share. The default workgroup name for Windows Vista is WORKGROUP, but note that the default for Window XP Home Edition was MSHOME.

### Note

**Checking Your Workgroup**

On Windows XP, you can discover your workgroup name by clicking **Start** and then right clicking on **My Computer**. Now click **Properties**. Click on the **Computer Name** tab in the **System Properties** dialog that has appeared. You can change the workgroup name by clicking the **Change…** button.

On Windows Vista, you can find out your workgroup name by clicking on **Start** and then right clicking on **Computer**. Now click **Properties**. The workgroup name is in the *Computer name, domain, and workgroup settings*. You can change the workgroup by clicking on **Change settings** and then the **Change…** button.

1.  3\. Leave the authentication method as **Anonymous** here as this is the easiest to get working and provides unrestricted read/write access to everyone. Now click **Save and Restart**. This will save the changes you have made and restart the CIFS service.

![Accessing the Disk via CIFS](img/4688_03_13.jpg)

1.  4\. Now, go to the **Shares** tab and click on add circle. Enter a name for the share. Repeating the name of the mount point is probably the safest policy, so in this case *store* and also add a comment. Then click **...** in the Path section. This will bring up a simple file system browser. The files you are seeing are on the FreeNAS server and NOT on your local PC. Click **store** and **/mnt/store/** will appear in the little edit box at the click. **OK** it and you will be taken back to the shares page. Now **/mnt/store/** has been added as the path.

2.  5\. Leave everything else as it is and click **Add** and then apply the changes.

So now the first hard disk of the computer is formatted, mounted, and shared to the rest of the network. Now we will access the share from a Windows Vista machine.

### Testing the Share

You can perform this test from any machine that supports the CIFS protocol including Windows, OS X, and Linux. Here we are going to use Windows Vista:

1.  1\. Open the Network and Sharing Center by clicking **Network** on the Start menu. When the window appears, Vista will automatically scan the network for any shared network resources. When it has finished, you will see the available machines on the network including FREENAS.

![Testing the Share](img/4688_03_04.jpg)

1.  2\. Open up the FREENAS computer and you will see *store*, the storage area that you configured. Double click on that and you now "inside" the FreeNAS server from within your Windows machine. Try dragging and dropping a few files into the *store* area. Then try deleting them again.

2.  3\. To access the FreeNAS server, without using the Network and Sharing Center, click **Start**, and type **\\freenas** and then press *Enter*. This will bring up the shares available on the FreeNAS server directly.

![Testing the Share](img/4688_03_05.jpg)

## Accessing via FTP

The File Transfer Protocol is a fast and stable protocol for transferring files over a network. FTP is a client/server protocol and is most often used to transfer files from one machine to another in a "one off" sense. A connection is made, the files are transferred, and the connection is closed. This is a protocol that is often used to allow people to download files from the Internet. In the NAS context, it is very useful for offering a repository of software on your network (like software, company templates, and antivirus software updates).

1.  1\. To enable the FTP service, go to **Services: FTP** and tick enable.

2.  2\. Now click **Save and Restart**.

This will save the changes you have made and restart the FTP service. By default, anonymous FTP logins are enabled. This means that any user can log in and access the storage.

### Testing FTP Access

All modern operating systems including Windows, Linux, and OS X contain command line FTP clients. These are simply enough to use, but can be a bit difficult to learn if you are used to using graphic user interfaces. A good free FTP client for Windows is Core FTP and you can download it from [http://www.coreftp.com](http://www.coreftp.com).

1.  1\. To test the FTP service of FreeNAS using Core FTP, first download it and install it.

2.  2\. The first window you see when you start Core FTP is the **Site Manager**. Here, you can enter the parameters for connecting to a server.

3.  3\. In the **Site Name** field, enter **FreeNAS_Server** as a way to remember this particular server connection.

4.  4\. In **Host/IP/URL** enter the IP address of your FreeNAS server.

5.  5\. Tick **Anonymous**, which enables you to connect without a username and password (actually the username is anonymous and Core FTP will fill that in for you).

6.  6\. Now press **Connect**.

![Testing FTP Access](img/4688_03_14.jpg)

Once you press *connect*, Core FTP will connect to the FreeNAS server. On the left, you will have the file on your Windows machine and on the right, the files on the FreeNAS machine. The FTP service puts you in at the top level so you will need to double click on *store* to enter into the *store* folder that represents the disk configured earlier. You can try dragging some files over from your Windows machines and see that they will be copied over to the FreeNAS server.

As a final test, you can try to access the FreeNAS server again via CIFS (just click **start** and enter **\\freenas** and then open the *store* share) and you will see the files you just copied over using FTP.

### Note

**Using Windows Built-In FTP Client**

Modern versions of Windows also come with a simple FTP client built-in to Windows Explorer. You can open an FTP connection to the FreeNAS server by typing **ftp://192.168.1.250** (if this isn't the IP address of your FreeNAS server, change accordingly) in the address bar of My Computer (or Internet Explorer). From here, you can drag and drop files and use Windows Explorer very much like you would with a local hard drive.

# Installing to Hard Disk

Up until now, FreeNAS has been running from the "LiveCD", which means it boots from the CD and runs from memory without needing the operating system to be copied on to the hard drive. FreeNAS has an option to install itself on a hard drive or USB flash disk and will boot from there without needing the CD. For a hard drive install, the configuration data will be saved on the hard drive rather than on a USB flash disk or floppy disk.

The preferred way to install FreeNAS is onto a USB flash disk as this frees up an IDE or SCSI channel for extra storage. FreeNAS is optimized for use with flash disks. To save wearing out the flash disk, (with too many write operations) the embedded version of FreeNAS runs from memory once the initial boot has been made from the flash disk. To use the USB flash disk version, you need to have a BIOS that can boot from a USB flash disk.

### Note

Before you proceed, you should note that installing FreeNAS on a hard drive will erase all the data on that hard drive.

When installing FreeNAS to a hard disk, you have the option to partition your hard drive into 2 parts. The first will be the boot partition which will contain the FreeNAS server. This is only a small partition and the rest of the disk will be the larger data partition.

To install FreeNAS to your hard disk, you need to go back to the console (as this can't be done from the web interface).

![Installing to Hard Disk](img/4688_03_15.jpg)

### Note

For more information on the difference between "embedded" and "full", see the *Embedded versus. Full* section below.

1.  1\. Select option **9** and then **2** to install the "embedded" version of FreeNAS onto a hard disk with a data partition.

2.  2\. You will be presented with a summary of what actions the installation will perform. Select **OK** to continue.

3.  3\. Next, you will be asked to confirm the device name of the CDROM (which has the FreeNAS files).

4.  4\. Then, you need to choose the hard drive on which you would like to install FreeNAS. You will be given a list of available hard drives. You need to install FreeNAS on ad0, which is the primary master IDE drive. If you have SCSI, you should choose da0\. Note that if you have a USB flash disk inserted, it may be listed as da0 and this shouldn't be confused with a genuine SCSI hard drive.

5.  5\. The system will now install.

Once the FreeNAS is installed, the console will show a success message. At this point, it is important to understand a few things:

1.  1\. The disk with FreeNAS installed has now been partitioned into two parts. These are referred to as ad0 partition 1 (or ad0p1 for short) and ad0 partition 2 (ad0p2). Partition 1 contains the FreeNAS software and partition 2 is for your data.

2.  2\. You need to revisit the **Disks: Mount Point** page as the underlying structure of the disk has changed and any previous mount point may be invalid.

3.  3\. You do not need to format ad0 again. It has been formatted using UFS for you.

4.  4\. You need to remove the disk from the CD drive and reboot the machine. To do this remove the disk, press *Enter* to go back to the installation submenu, the chose *EXIT* to return to the main menu and finally, use option **7** to reboot the machine.

To mount partition 2, you need to go to the **Disks: Mount Point** page. If you have existing mount points defined for the disk when you installed FreeNAS, you need to remove them first before continuing. To mount partition 2, click the add circle, chose the disk (probably ad0 or da0), then select partition 2\. This is different from when you used the whole disk. Partition 2 is the data partition while partition one contains the FreeNAS software. Fill out the name and description and click **add**.

After that, you can enable services like CIFS and FTP just like you did before.

![Installing to Hard Disk](img/4688_03_16.jpg)

## Embedded versus Full

When you were in the installation submenu, you would have seen that there is also what is known as a "full" install of FreeNAS. This doesn't mean that it has any more features or is somehow more complete than the "embedded" version. But rather, it is a way to install FreeNAS like a traditional operating system. The files are installed on the hard disk and if they are edited or deleted they are done so permanently. The embedded version runs from RAM (and is initially loaded from the hard disk) and any changes made to the operating system files will only be temporary and when the system is rebooted, it will return to its original state. This means that the full version can be tweaked and changed which is good for those developing plugins for FreeNAS, but it also means that the system can be broken accidentally by those unfamiliar with FreeBSD. The embedded version, however, adds a level of protection and if anything goes wrong, a reboot will restore the system files to their installed state.

### Note

This discussion is about the system files, not about the configuration file that is updated and controlled via the actions taken in the web interface. A reboot will NOT restore the configuration to a previous state unless the changes have not been applied.

The recommended way to install FreeNAS is to use the embedded version. You should only consider the full version if any of the following are true:

*   You need to install extra FreeBSD packages to your FreeNAS server.

*   You need to customize the size of the root file system.

*   You are low on system RAM and need to run FreeBSD from hard disk, rather than a RAM disk.

# Upgrading FreeNAS from a Previous Version

If you are running a LiveCD version of FreeNAS, then upgrading is simple. Just download the new version and burn it onto a CD and then reboot the FreeNAS server using the new CD. Always remember to check the FreeNAS website for any information about upgrading particularly with regards to internal changes like the format of the configuration file.

If you have installed FreeNAS on a hard drive (or USB flash disk), then FreeNAS can be upgraded via the web interface. On the FreeNAS website, you will find embedded versions of FreeNAS for download. These are not ISO files like the LiveCD version but rather files for upgrading an existing installation. The embedded download is a `.img` file. Download the correct file and save it on your hard disk. Go to the web interface and find the **System: Firmware** page, click **Enable Firmware Upload** and then locate the file you downloaded using the **Browse..**. button.

Finally, click on **Upgrade firmware**. Wait patiently and FreeNAS will reboot when the upgrade is completed, your configuration data will be preserved.

### Note

DO NOT abort the firmware upgrade once it has started. You need a minimum of 128 MB RAM to perform the firmware update. You should always backup the System configuration before doing a Firmware upgrade. You can do this from the System: Backup/Restore page.

# Summary

In this chapter, we have installed and configured the FreeNAS server. We booted the FreeNAS server from the LiveCD disk and configured a simple disk that was accessed by CIFS and FTP. We also looked at how to install FreeNAS to the hard drive and how to upgrade it.

In the next chapter, we shall take a detailed look at the different ways you can connect to your FreeNAS server including from Windows machines, from OS X and from Linux.