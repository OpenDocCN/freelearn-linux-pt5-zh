# Chapter 4. Connecting to the FreeNAS

The strength of the FreeNAS server is that so many different operating systems can connect to it and use its services. In this chapter, we shall look at the different services and protocols supported by FreeNAS, and see examples of how various platforms like Windows, OS X, and Linux can use the FreeNAS server for file sharing, backup, and streaming multimedia.

# Introduction

The FreeNAS server is "multi lingual" in that it can talk to many different types of computers system using a variety of protocols. Before looking at each of these protocols individually, it is worth looking at them as a whole and seeing why each protocol exists and for which job is it best suited.

The table below lists each protocol along with which type of computer (Windows, Apple Mac etc) can connect to the FreeNAS server using that protocol. Some protocols are native to a particular platform, for examples CIFS is native protocol for Windows machines, but other operating systems like OS X can "talk" CIFS as well. Also listed, is the main use of that protocol.

|   | Windows | OS X | Linux/UNIX | Usage |
| CIFS | Yes. Native. | Yes. Built in. | Yes. Built in. | File sharing |
| NFS | Yes. Requires 3rd party software. | Yes. Native. | Yes. Native. | File sharing |
| FTP | Yes. Built in or you can use 3rd party software. | Yes. Built in and you can use 3rd party software. | Yes. Built in and you can use 3rd party software. | Uploading and downloading files |
| RSYNCD | Yes. Requires 3rd party software. | Yes. Built in and you can use 3rd party software. | Yes. Built in. | Backups and synchronization. |
| Unison | Yes. Requires 3rd party software. | Yes. Requires 3rd party software. | Yes. Requires 3rd party software. | Backups and synchronization. |
| AFP | No. | Native. | No. | File sharing for Apple |
| UPnP | Yes. Requires 3rd party software. | Yes. Requires 3rd party software. | Yes. Requires 3rd party software or a specialized distribution (see below). | Streaming media |
| iSCSI | Yes. Built in to some versions of Windows, e.g. Windows Vista. | Yes. Requires 3rd party software. | Yes. Built in. | Connecting to remote raw disks with the SCSI over IP protocol. |

In this chapter, we are going to look in detail at the various protocols. To help you in this, it will be best if you have a test FreeNAS server up and running and you have followed the quick installation and configuration guide in Chapter 2.

# Connecting via CIFS

The Common Internet File System (CIFS) is the standard way in which files are accessed on a remote Windows computer. Developed and maintained by Microsoft for use on their Windows platform, it has also been implemented on most major operating systems including OS X and Linux using 3rd party software, the most popular of which is called Samba. Samba is open source software that provides remote file access services to CIFS clients on a variety of platforms (including to Windows clients). Samba is included in the FreeNAS server.

The abilities of CIFS are actually larger than just accessing files. With it, other resources like printers can also be shared on the network, but for FreeNAS, CIFS is used to share disks on the server and make them available to other computers that understand the CIFS protocol. This means that Windows, Linux, and OS X machines are all capable of accessing files on the FreeNAS server via the CIFS protocol.

### Note

Sometimes, when reading about CIFS, you might read the term SMB or Server Message Block. SMB was the original name of the CIFS protocol and was the result of work done by IBM and then later Microsoft. In 1996, SMB was renamed CIFS.

## Configure CIFS on the FreeNAS Server

Before attempting to connect another computer to the FreeNAS server via CIFS, you need to be sure that CIFS is correctly configured on the FreeNAS server.

1.  1\. Go to Services: **CIFS/SMB**. This page contains two tabs, first, the **settings** page and then the **shares** page. First of all, make sure that the service is enabled by ticking the "Enable" box in the title bar of the configuration table.

2.  2\. To get CIFS working with the default settings, you need to set as a minimum the Authentication system, the NetBios Name, and the Workgroup. Authentication should be left at Anonymous for the moment. We will look more at authentication and user management in FreeNAS in the next chapter. The NetBios Name is the name that the FreeNAS server will have on the Windows network. When you want to access the server, Windows lets you use a friendly name rather than the IP. For this example, we will use the name FreeNAS. Finally, you need to enter the workgroup. All Windows machines belong either to a workgroup or a domain.

    ### Note

    **Workgroups and Domains**

    Home networks and small office LANs use workgroups, which are essentially collections of computers that share a similar tag or designation. On a home network, there is normally only 1 workgroup called "WORKGROUP" (or MSHOME if you mainly have Windows XP machines. For the small office environment, there might be 2 or 3 workgroups, maybe "SALES", "ENGINEERING", and "ACCOUNTS". The workgroup links the machines together so that when the network is viewed, these machines will be grouped together. There are typically no more than ten to twenty computers in a workgroup and all the computers must be on the same local network.

    Larger business networks use domains rather than workgroups. In a domain, there is a server (called the domain controller) that controls the resources and security in that domain. Once a user has a domain account, they can log on to any computer in the domain without needing a local account on any particular machine. Domains can have thousands of computers spread across several networks.

3.  3\. Enter in the workgroup name for the FreeNAS server. It will probably be WORKGROUP or maybe XPHOME if you have many Windows XP machines on your network.

4.  4\. The next step is to configure what disks are shared on the network. Click on **Save and Restart** at the bottom of the page and then click the **Shares** tab at the top. Here is where you add disks to be shared on the network via the CIFS protocol. You can only share previously configured disks. If you have disks that need to be added to the FreeNAS, go to the Disk: menu. For more information, see the quick start guide in Chapter 2 or for advanced information, go to Chapter 6.

5.  5\. Click the add circle. You are now in **Services: CIFS/SMB: Share: Add**. The minimum data here is the **Name** you want to give the share, a Comment or description about the share and where the share is, the Path.

6.  6\. Name and comment should be easy enough to fill in. For this example, we will use store and a storage place respectively. Now, for the path. In FreeNAS, all disks are mounted under **/mnt**. This means that the path will be something like **/mnt/diskname**. To find the right path name, click on **...** This will bring up a new window.

![Configure CIFS on the FreeNAS Server](img/4688_04_01.jpg)

At the top, you will see **/mnt** and then further down **store**. This is the name of the disk that was configured in **Disks: Mount Point: Add**. Click on store and the path will change to **/mnt/path**.

![Configure CIFS on the FreeNAS Server](img/4688_04_02.jpg)

1.  7\. Click on **OK**, then on **Add**, and finally, apply the changes.

At this point, CIFS is configured and ready to go. Before looking at how the different versions of Windows, as well as OS X and Linux, can connect to FreeNAS using CIFS, let us look in details at the other parameters.

### CIFS Settings Explained

| Parameter | Meaning |
| --- | --- |
| Description | Server description. This can be left blank but you may find including a description useful. |
| Dos charset | This is the charset FreeNAS uses when communicating with Windows 9x/Me clients. It will talk Unicode to all newer clients. The default is CP850, which is perfect for English and other Western European Languages. |
| Unix charset | This is the charset used internally by FreeNAS. The default is UTF-8, which is fine for most systems and covers all characters in all languages. |
| Log Level | Sets the amount of log/debug messages that are sent to the log file. These can be read in Diagnostics: Logs: System. You should leave this on Minimum unless you are trying to solve a CIFS connection problem. |
| Local Master Browser | Allows FreeNAS to try and become a local master browser.Inside the My Network Places (or the equivalent in Vista), icons represent machines. The local master browser collates information to create this browse list.You can safely leave this at Yes, unless you are running the FreeNAS inside a domain, when it is best to set it to No. |
| Time server | If your FreeNAS server has an accurate clock, you can instruct it to advertise itself as an SMB time server to Windows clients. |
| WINS server | WINS (Windows Internetworking Name Server) allows the FreeNAS server to discover the friendly CIFS name of machines on other networks. You can set up FreeNAS to use a WINS server somewhere else on the network by simply pointing it to the IP address of the WINS server.If you are using FreeNAS just for machines on the local LAN, then you can leave this empty. |

### CIFS Advanced Settings

These are advanced parameters and you should only change them if you know what you are doing and why you would want to change them. For 99.9% of all cases, these parameters can be left alone.

| Parameter | Meaning |
| Create mask | Every data file created on the FreeNAS has some default read and write permissions. Use this option to override the file creation permissions. (0666 by default).See the tip below for more information. |
| Directory mask | All directories that are created on the FreeNAS server have default read and write permissions. Use this option to override the directory creation permissions (0777 by default).See the tip below for more information. |
| Send Buffer Size | Size of send buffer (16384 by default). For Windows 95/98, you have to use a 8192 buffer size. |
| Receive Buffer Size | Size of receive buffer (16384 by default). For Windows 95/98, you have to use a 8192 buffer size. |
| Large read/write | Use the new 64k streaming read and write variant SMB requests introduced with Windows 2000. |
| EA support | Extended attribute support. Allow clients to attempt to store OS/2 style extended attributes on a share. |

### Note

**Create Mask**

In FreeBSD (and most UNIX operating systems), each file or directory has 3 levels of read/write permissions. User, Group, and World. The permission to read, write or execute a file can be set at each level.

When FreeNAS creates files that have been copied over the network, a default set of file permissions is set for that file. This is determined by the default create and directory masks (see above).

Using a numbering scheme, the create or directory mask has four number places starting always with 0, for example 0744, representing the three levels. The first number on the left side is for "user", the middle one is for "group", and the right-hand one for "world." Here is what each number means:

0 = no access, 1 = execute only, 2 = write only, 3 = write and execute, 4 = read only, 5 = read and execute, 6 = read and write, 7 = read, write and execute (full access).

### Options when Adding Shares

Like the general CIF settings, there are several different options available when you add a share on the **Services: CIFS/SMB: Share: Add page**. Here is a brief look at those options and what they do.

| Parameter | Meaning |
| --- | --- |
| Browseable | This controls whether this share is seen in the list of available shares in a net view and in the browse list. If it is not browseable, the share is still accessible but only when the address is specified directly. |
| Inherit permissions | The permissions on new files and directories are normally governed by create mask and directory mask, but the inherit permissions parameter overrides this. This can be particularly useful on systems with many users to allow a single share to be used flexibly by each user. |
| Recycle bin | This will create a recycle bin on the share. If you create the recycle bin, you have to empty it manually. |
| Hosts allow | This parameter is a comma, space, or tab delimited set of hosts that are permitted to access this share. Use the keyword ALL to permit access for everyone. Leave this field empty to disable this setting. |
| Hosts deny | This parameter is a comma, space, or tab delimited set of host that are NOT permitted to access this share. Where the lists conflict, the allow list takes precedence. In the event that it is necessary to deny all by default, use the keyword ALL (or the netmask 0.0.0.0/0) and then explicitly specify to the host's allow parameter those hosts that should be permitted access. Leave this field empty to disable this setting. |

## What does It Mean to Map a Network Drive?

When accessing files over the network, there are two ways to find the data. One is to browse to it using the *My Network Places* (or similar depending on your version of Windows). Here, you can find a machine and then drill down until you find the folder you want. The other option is to mount the remote shared folder as a drive on your machine. This *network drive* always takes you to the right place where the data is on the remote server. You can also configure it to be mounted every time you start windows. This is particularly useful for something like the FreeNAS server as the storage space made available is always available on your PC.

## Connecting with CIFS via Windows Millennium

Double click on **My Network Places**. Here, you can browse the local network and see recently accessed network resources.

1.  1\. Double clicking on **Entire Network** will take you to a list of the workgroups and domains on your local network. On a small network, this may well just be one workgroup, mostly likely **WORKGROUP** or **MSHOME**.

![Connecting with CIFS via Windows Millennium](img/4688_04_03.jpg)

1.  2\. Double click on **Workgroup** and you will see a list of machines in that workgroup.

2.  3\. Find the FreeNAS server, it is probably called **Freenas**. Now double click on it and you will see a list of available shares on the FreeNAS server.

3.  4\. If you are using a test FreeNAS server setup according to the quick installation guide in chapter 2 then you will only see one share called *store*. If you double click on *store*, you will have access to that storage space on the FreeNAS server. You can try copying some files to *store* and see the FreeNAS server in action.

    ### Note

    To save time finding the FreeNAS server, you can just enter its address directly in the Address bar in My Network Places. The address is in the form **\\server\share** so in our example we would use **\\freenas\store**.

    Now just hit *Enter* and you will have access to the FreeNAS server.

4.  5\. There are a couple of ways to map a network drive in Windows Millennium. One is that when you have the FreeNAS machine open in My Network Places. Rather than double clicking on store, you can right-hand click on it instead. This will produce a small menu. Now click on **Map Network Drive...**

![Connecting with CIFS via Windows Millennium](img/4688_04_04.jpg)

*   Now, choose the drive letter you wish to map to this network share and tick **Reconnect at logon** if you want the drive to be automatically mounted when you start Windows.

Once you have done this, the network drive will appear in *My Computer* and you can use it much like you would any other hard drive on your machine.

## Using CIFS with Windows XP

Assuming your FreeNAS and your Windows XP machine are in the same workgroup, when you open *My Network Places*, the *store* share on the *Freenas* server should appear automatically. If it doesn't, wait a few moments (as sometimes it takes Windows a few minutes to collate the list of all available resources in the workgroup) and press *F5* to refresh the display.

Another way to get to the FreeNAS server is to click on **View workgroup computers** in the **Network Task** panel which is on the left side of the window. This will show you all the computers in your workgroup that should include the FreeNAS server. From here, you can drill down and find the network share *store*.

Like Windows Millennium and Windows Vista, you can enter the address of the FreeNAS server directly in the address bar of the *My Network Places* window.

To map a drive from *My Network Places* to the FreeNAS server, find the *Freenas* machine either through the workgroup or by entering its name in the address bar.

1.  1\. Right-click the shared folder that you want to map, and then click **Map Network Drive**.

2.  2\. Click the drive letter that you want to use, and then specify whether you want to reconnect every time that you log on to your computer.

### Note

Network drives are mapped by using letters starting from the letter Z. This is the default drive letter for the first mapped drive you create. However, you can select another letter if you want to use a letter other than Z.

1.  3\. Click **Finish**.

2.  4\. A window opens that displays the contents of the store share that you have mapped and the drive letter (for example Z) will be available in **My Computer**.

![Using CIFS with Windows XP](img/4688_04_05.jpg)

## FreeNAS, CIFS, and Windows Vista

Accessing the FreeNAS server via CIFS in Windows Vista is very similar to the previous versions of Windows and was also covered in chapter 2\. In summary, open the Network and Sharing Center by clicking **Network** on the Start menu. When the window appears, Vista will automatically scan the network for any shared network resources. When it has finished, you will see the available machines on the network including FREENAS.

Also, to access the FreeNAS server without using the Network and Sharing Center, click **Start**, and type **\\freenas** and then press *Enter*. This will bring up the shares available on the FreeNAS server directly.

To map a network drive, click **Start**, and type **\\freenas**, and then press *Enter*. Right-hand click the shares you wish to map (e.g. store) and click on the **Map Network Drive..**. item that appears in the menu. Like XP, network drives are mapped by using letters starting from the letter Z. This is the default drive letter for the first mapped drive you create. However, you can select another letter if you want to use a letter other than Z. Click **Finish** and the drive will be mapped.

## Accessing the FreeNAS via CIFS from Linux

All the popular Linux distributions support the CIFS protocol using a piece of software called Samba. Samba is an open source suite that allows users of non-Windows operating systems, like Linux, to interoperate with Windows machines and servers. The goal behind the Samba project is one of removing barriers to interoperability.

Using the client aspects of Samba (as Samba is also a server) in Linux is easy. For example in Kubuntu, to get a listing similar to My Network Places in Windows, open Konqueror and enter **smb:/** in the address bar. This will then show you the workgroups available. From here, you can drill down to the FreeNAS server and the store share. This should work for all KDE-based distributions like Mandriva and SuSE. Alternatively, you use the **Remote Places** icon that you will find on the System Menu (which isn't to be confused with the K Menu). Here, you can click on **Samba Shares**, which will take you to the smb:/ listing.

![Accessing the FreeNAS via CIFS from Linux](img/4688_04_06.jpg)

## A CIFS Connection from OS X

Apple's OS X operating system has full Windows connectivity. With the release of OS X 10.5 Leopard, it's easy of use has also been improved. Leopard will automatically detect the available CIFS shared resources available on the local area network. They appear in the Finder window on the left-hand-side under SHARED. Click on the resource you want to use (in case of FreeNAS, it will say "freenas SMB Service") and from there, you just access them as you would any other folder.

![A CIFS Connection from OS X](img/4688_04_07.jpg)

# FTP

While CIFS and NFS are file system protocols, which means that whole file systems can be shared on the network and other computers can uses those file systems as if they were attached locally, FTP is more limited in that it is designed just to for the transfer of files from one computer to another. In CIFS and NFS, when you read or write to files, they remain on the remote server, whereas with FTP, files are transferred from the server to the local machine or vice-versa. Any changes you make to the files once downloaded to your computer, will not be reflected in the files on the server.

The main use of the FTP protocol is for downloading files from the server onto the local computer. Files can be uploaded once to the server (also by FTP) and then downloaded many times by those who need the files. This can be useful in one of two ways:

*   For the small office or home environment, it is perfect as a repository for downloaded software and drivers. As all web browsers have support for FTP, it means that files can be downloaded on to the local machine with just a standard web browser, or more complex FTP clients can be used.

*   In the business environment, if there is a company Intranet, then links from the various internal websites can link to files on the FreeNAS server via FTP. This is particularly useful if there are large files to be downloaded that don't need to reside on the web server.

FTP also supports the uploading of files to the server. Here, those responsible for the Intranet or file repository upload the files to the FreeNAS server and then those needing the files can download them as described above.

To configure the FTP service, go to **Services: FTP** and tick the **Enable** box. Click **Save and Restart** and FTP is configured. The rest of the settings can be left as they are. But should you consider changing any of them, below is a table of what each parameter means.

| Parameter | Meaning |
| --- | --- |
| TCP port | If, for networking reasons, you need to change the port for FTP, you can change it here. The default is port 21. |
| Number of clients | Maximum number of simultaneous clients. |
| Max. conn. per IP | Maximum number of connections per IP address (0 = unlimited). |
| Timeout | Maximum idle time in minutes. |
| Permit root login | Specifies whether connecting user is allowed to log in as superuser (root) directly. Use this option with care as enabling it can be a security risk. |
| Anonymous login | Enable anonymous login. |
| Local User | Enable local user login. |
| Banner | Greeting banner displayed by FTP when a connection first comes in. |

There are also several advanced FTP options. The table below describes them:

| Parameter | Meaning |
| --- | --- |
| Create mask | Use this option to set a new default file creation mask (077 is the initial default). |
| Directory mask | Use this option to set a new default directory creation mask (022 is the initial default). |
| FXP | Enable FXP protocol. FXP allows transfers between two remote servers without any file data going to the client asking for the transfer (insecure!). |
| NAT mode | Force NAT mode. Enable this if your FTP server is behind a NAT box that doesn't support applicative FTP proxying. |
| Keep all files | Allow users to resume and upload files, but NOT to delete or rename them. Directories can be removed, but only if they are empty. However, overwriting existing files is still allowed. |
| chroot everyone | chroot() everyone, but root. |
| Passive IP address | Force the specified IP address in reply to a PASV/EPSV/SPSV command. If the server is behind a masquerading (NAT) box that doesn't properly handle stateful FTP masquerading, put the IP address of that box here. If you have a dynamic IP address, you can put the public host name of your gateway, which will be resolved every time a new client will connect. |
| pasv_min_port | The minimum port to allocate for PASV style data connections (0 = use any port). |
| pasv_max_port | The maximum port to allocate for PASV style data connections (0 = use any port). |

## Using the Command Line FTP Client

In chapter 2, we saw how to use the free FTP client, CoreFTP, to connect from a Windows machine to the FreeNAS server, and we also mentioned the built-in FTP client that comes with Windows Explorer. All the major operating systems (including Windows also) include a built in command line FTP program. They are all essentially the same regardless of operating system so if you know how to use the command line FTP program on OS X, you will be able to manage on Linux and so on. For this example we will use Apple's OS X.

To start, you need to open a terminal window (on OS X or Linux) or a command prompt (on Windows). To connect to the FreeNAS server you type:

```
ftp 192.168.1.250

```

Where `192.168.1.250` is the IP address of the FreeNAS server, you will need to enter the IP address of the FreeNAS if yours is different.

The first thing you need to enter is a user name, for now we will use anonymous FTP, which means that anyone can access the FTP. The next chapter we will look at user management and authentication. So enter:

```
anonymous

```

Now, you will need to enter a password. As we are doing an anonymous login, any password will do, including just pressing *ENTER*.

You will then be in the FTP program and you will see the prompt **ftp>**.

To see a listing of what is in the directory, we use the `ls` command. So type `ls` and then press *ENTER*. The result will be something similar to this:

```
ftp> ls
227 Entering Passive Mode (192,168,1,250,112,201)
150 Accepted data connection
drwxrwxrwx 3 0 0 512 Jan 22 11:59 store
226-Options: -l
226 1 matches total
ftp> 

```

The important line amongst all this information is the line starting with `drwxrwxrwx`. You will see on the right-hand-side the word store. This is the disk called store that was added, formated, and mounted in the quick start guide of chapter 2\. The `drwxrwxrwx` shows us that it is a directory (because of the leading `d`) and that everyone has read, write, and execute permissions.

We now want to change directory to *store*, to do this we use the `cd` command:

```
ftp> cd store
250 OK. Current directory is /store
ftp> 

```

Doing a directory listing of the store folder reveals:

```
-rwx------ 1 21 0 135477136 Jan 22 12:45 W2KSP4_EN.EXE 

```

This is Service Pack 4 for Windows 2000\. I have copied to my FreeNAS server as an example. If I have one or more Windows 2000 machines on my network, which need to be upgraded to service pack 4, I can download the file from the FreeNAS server and run it on each machine.

To get the file and download it on to my machine, I use the `get` command:

```
ftp> get W2KSP4_EN.EXE 

```

The transfer will start and will finish with a summary like this:

```
135477136 bytes received in 00:15 (8.50 MB/s) 

```

To exit the FTP program, just type `quit`.

Here are a few more FTP commands to help you on your way:

| Command | Description |
| Delete | Deletes a file in the current remote directory. |
| lcd | Changes directory on your local machine. |
| mkdir | Makes a new directory within the current remote directory. |
| mget | Copies multiple files from the remote machine to the local machine; you are prompted for a y/n answer before transferring each file unless you use the prompt off command before hand. |
| mput | Copies multiple files from the local machine to the remote machine; you are prompted for a y/n answer before transferring each file, unless you use the prompt off command before hand. |
| put | Copies a file from the local machine to the remote machine. |
| pwd | Displays the current working directory on the remote machine. |
| rmdir | Deletes a directory in the current remote directory. |

## Using a Web Browser for FTP

All web browsers contain a simple FTP client and it is possible to browse the FTP areas and download files. Start your web browser and then in the address bar, enter:

```
ftp://192.168.1.250/ 

```

Notice that the address starts with `ftp://` and NOT `http://`.

The web browser will show something like this:

![Using a Web Browser for FTP](img/4688_04_08.jpg)

Here, the web browser shows *store*, the disk that was previously configured. Unlike CIFS, you don't need to add each disk individually; all the disks mounted are available.

If you click **store**, you will be taken into the **store** folder and you will see something like this:

![Using a Web Browser for FTP](img/4688_04_09.jpg)

From here, you can click on the Service Pack and the web browser will start to download it for you.

# NFS

The Network Filing System is the UNIX equivalent to the CIFS protocol. Although there are 3rd party packages to allow Windows machines to use NFS resources, NFS has remained most dominant on the UNIX/Linux operating systems.

To configure the NFS service, go to the **Services: NFS** page. As with all the other services, to enable it you need to tick the Enable box.

There are few parameters for NFS, but they are important.

### Note

The following instructions are for the 0.68 series of FreeNAS releases. The 0.69 and 0.7 versions of FreeNAS treat NFS slightly differently but essentially the ideas are the same.

In 0.69 and 0.7 please note the following differences:

The Services: NFS settings page now has two tabs; *Settings* and *Shares*.

The Settings tab has a new parameter: *Number of servers* that specifies how many NFS processes to create. There needs to be enough processes to handle the maximum level of concurrency from NFS clients on the network. A typical figure would be four to six.

The NFS set-up now has the concept of shares (very similar to that of CIFS), to make a folder or disk available on the network a 'share' must be created for it. To do this you click on the *Shares* tab and click the Add circle.

The rest of the parameters described below are now set on a per share basis and not for the whole server.

The first parameter is *Map all users to root*. This means that on the FreeNAS machine remote users will have root privilege (or administrator right in Windows terminology) while accessing the disks over NFS. This is generally a bad idea and while the default is *yes*, it is best to change it to *no*. Also, any OS X clients connecting to the FreeNAS via NFS will issue a few warnings about needing usernames and passwords. When *Map all users to root* is set to *no*, this doesn't happen.

The second parameter is the list of authorized networks that are allowed to connect to the FreeNAS server via NFS. This list of authorized networks is defined by setting which group of addresses can access the server.

If your FreeNAS server has a netmask of 255.255.255.0 (which is most probably the case) then:

Click the add circle and enter the first 3 numbers of the IP address of the FreeNAS server (with . in between) and then finish it with dot zero (.0). For example, if the IP address of the FreeNAS server is 192.168.1.250, then you would enter 192.168.1.0 for the authorized network. Finally, select 24 (in other words 255.255.255.0) in the drop down box.

If you are using a netmask of 255.255.0.0, then you need to enter the first 2 numbers of the IP address and end with dot zero dot zero (.0.0), for example 192.168.0.0\. Set the drop down box to 16.

If you are using a netmask of 255.0.0.0, then you need to enter just the first number of the IP address and end with dot zero dot zero dot zero (.0.0.0), e.g. 192.0.0.0\. The drop down box needs to be set to 8.

If you are unsure what the value of the netmask is, then go to Interfaces: LAN and see the IP Address field. You need to set the drop down box in the Services: NFS page to that of the value of the drop down box in the IP Address field.

## Using NFS from OS X

At the core of Apple's OS X, is the open source UNIX operating system called Darwin. As a result, OS X natively understands the NFS protocol. To connect to the FreeNAS server from OS X, open the Finder and the Go->Connect to Server... (Or Apple-K as a shortcut)

In the Server Address field enter:

```
 nfs://192.168.1.250/mnt/store 

```

The address is made up of 3 parts. First is `nfs://`, which tells OS X that we want to talk via NFS. Then comes the address of the FreeNAS server (192.168.1.250, change this accordingly) and finally, the resource to which we want to connect. All the disks are exported as /mnt/name. In this case, the name is store as we previously defined *store* in Disks: Mount Point: Management. Once mounted, it will appear under SHARED in the Finder on the left-hand side.

## Mount FreeNAS via NFS on Linux

The particular Linux distribution you are using may have a graphical to mount NFS shares, but the lowest common denominator that works for all Linux versions is to use the command line.

To mount the FreeNAS server via NFS is easy and also mostly the same as in OS X, but rather than entering the command in the GUI, it is entered at the command line. In Linux, to mount any type of device, remote or local, you need to have a mount point, a place to peg the mounted resource. So open a terminal window and become root (usually with the *su* command, or alternatively you can prefix *sudo* in front of the following commands).

First, you need to change directory to the /mnt directory:

```
# cd /mnt 

```

Now create a directory to peg the FreeNAS server to:

```
# mkdir freenas 

```

And now mount the FreeNAS server:

```
# mount 192.168.1.250:/mnt/store /mnt/freenas 

```

Once completed, all the files and folders on the disk *store* on the FreeNAS server will appear under `/mnt/freenas`. So using earlier examples, you will find the Windows 2000 Service Pack 4 at `/mnt/freenas/W2KSP4_EN.EXE`, not that it would be much good here!

# RSYNCD, Unison, AFP, and UPnP

CIFS, NFS, and FTP are the 3 main file access protocols for FreeNAS, however, FreeNAS doesn't stop there, it has another 4 protocols for different specialized tasks including backups and streaming multimedia.

## Using RSYNC for Backups

RSYNC copies files between machines in an efficient manner and is therefore excellent for making backups. The RSYNC protocol allows RSYNC to transfer just the differences between two sets of files across the network connection, using an efficient algorithm. RSYNC is a client/server system meaning that a server is running to receive the backup file and a client sends the files from the local machine. For FreeNAS, this means that the RSYNC service can be configured and computers on your network can be backed up to it using an RSYNC client.

To enable the RSYNC service, tick the Enable box on the Services: **RSYNCD: Server** page. The other parameters you can leave at their defaults.

RSYNC is essentially a command line tool with a Linux/UNIX background, however, there are some Windows programs for handling backups via RSYNC. For more information, visit the websites:

*   DeltaCopy ([http://www.aboutmyip.com/AboutMyXApp/DeltaCopy.jsp](http://www.aboutmyip.com/AboutMyXApp/DeltaCopy.jsp))

*   NasBackup ([http://www.nasbackup.com](http://www.nasbackup.com))

Both of these tools are open source and free to download and use without restriction.

From the Linux (or OS X) command line, you can start a backup of files on the local machine like this:

```
rsync -a /home/user/backup rsync://192.168.1.250/store 

```

This will backup the directory `/home/user/backup` to the FreeNAS onto the store disk. The `-a` parameter put RSYNC into archive mode, which means that any subdirectories will also be copied while preserving time stamps.

As before, you need to change the IP address (192.168.1.250) to the IP address of your FreeNAS server.

### Note

FreeNAS can also act as a RSYNC client, which means it send files from the server to another server and so, allowing the FreeNAS server to be backed up. We will look at this in more detail in chapter 7.

## Using Unison for Backups and Synchronization

Unison is a file-synchronization tool for UNIX and Windows. It allows two replicas of a collection of files and directories to be stored on different hosts (or different disks on the same host), modified separately, and then brought up-to-date by propagating the changes in each replica to the other. It is similar to RSYNC in that it can perform mirroring/synchronization of two directories, but unlike RSYNC, Unison can deal with updates to both copies of a distributed directory structure. Updates that do not conflict are propagated automatically. Conflicting updates are detected and displayed. Unison is open source and is published under the GNU Public License.

To enable the Unison service, go to **Services: Unison** and tick the Enable box. You need to choose which share the unison service will use and you can probably leave the other settings to their defaults.

For Unison to work, the Secure Shell (SSH) must be enabled (Services: SSH) and the user must have shell access enabled (Access: Users).

SSH and user administration are covered in detail in the next chapter.

## Connecting to FreeNAS via AFP

Like CIFS for Windows and NFS for UNIX, Apple also have a file system protocol called AFP (Apple Filing Protocol). It allows Apple Macs to connect natively to the FreeNAS server. As with CIFS, the Mac will automatically discover any AFP services offered on the local area network. The AFP service will appear in the **SHARED** section in the Finder.

![Connecting to FreeNAS via AFP](img/4688_04_11.jpg)

To enable AFP, go to the **Services: AFP** page, and tick the Enable box. Now tick the **Enable guest access** box and click **Save and Restart**. The local authentication method will be looked at in the next chapter along with all other user management issues. AFP will now be running. Go to your Mac and you will find the server as described above.

There are a few options available when configuring AFP, here is a brief look at them:

| Parameter | Meaning |
| --- | --- |
| Server Name | Name of the server. If this field is left empty, the default server is specified. |
| Authentication | Guest access or local user authentication. You must select one of these, select guest access for the anonymous access. |

## Streaming Media with UPnP

UPnP (Universal Plug and Play) is a set of network protocols that allow devices to connect seamlessly and to simplify the implementation of networks in the home. It is popular for easily enabling the streaming of multimedia data from a server to a client. In the FreeNAS context, UPnP allows a previously configured directory to be made available to a UPnP multimedia client such as a wireless media center that attaches to your TV. The UPnP client will find the FreeNAS server via the UPnP protocols and play (via a selection menu) the multimedia files stored in the FreeNAS server.

To enable UPnP, go to the **Services: UPnP** page and tick the Enable box.

You can leave the name and the network interface at their defaults.

Now, you need to add which disks you would like to share on the FreeNAS server. Clicking the add circle will bring up the familiar window for selecting which disks to share. In our test setup, we need to share **/mnt/store**. Enter this and the click **Add**.

Finally, click **Save and Restart**.

There are also a few parameters that can be changed:

| Parameter | Meaning |
| --- | --- |
| Port | Enter a custom port number for the HTTP server if you want to override the default (49152). Only dynamic or private ports can be used (from 49152 through 65535). |
| Profile | Compliant profile to be used. UPnP has several profiles and you need to adjust this according to which device you are streaming to. If you are using a Microsoft's Xbox 360 then select Xbox 360, if you are using a Digital Living Network Alliance (formerly Digital Home Working Group) compliant device then select DLNA. |
| Control web page | Enable control web page. Accessible through [http://ip_address:port/web/ushare.html](http://ip_address:port/web/ushare.html). This is a rudimentary web page that allows you to add more shares. It is also the page Windows will show as you double click on the FreeNAS UPnP icon. |

As a test client, we can use the GeexBox Linux distribution ( [http://geexbox.org](http://geexbox.org)). GeeXboX is a free Linux distribution that turns a computer into a Home Theater PC or Media Center. It is a standalone LiveCD-based distribution, meaning it boots from the CD and runs out of the box on any modern PC. It can even be used on a diskless computer, as the whole system is loaded in RAM. Much like FreeNAS, you need to download the `.ISO` image file and burn it onto a CD and then boot the PC from the CD.

Once booted, you can select **Open** from the main menu followed by **Open File...** and then choose the UPNP option and finally, the FreeNAS server. If you have copied any multimedia files on to the FreeNAS server, you can play them.

![Streaming Media with UPnP](img/4688_04_10.jpg)

# iSCSI Target

iSCSI is an evolution of the SCSI protocol, which allows SCSI commands to be sent over a network. It allows two hosts to negotiate and then exchange SCSI commands using IP networks. The result is that a remote device with iSCSI capabilities can be seen to be a local disk drive but the commands and data for that device are being sent over the network rather than down a cable in the machine.

FreeNAS can be configured to be an iSCSI target, this means it acts as a remote SCSI disk and it can receive SCSI commands and apply them to a local disk. Once configured, an iSCSI initiator can connect to the FreeNAS iSCSI target and use the designated disk as a local disk.

### Note

To use the iSCSI functionality of FreeNAS, your server needs to have a minimum of 256MB of RAM.

iSCSI configuration starts on the **Services: iSCSI Target** page. An iSCSI target is formed from one of two things: an extent (the actual storage file or hard disk) or a device (a combination of other extents or devices in a RAID 0 or RAID 1 configuration). Therefore, there are a few steps to configuring a iSCSI target:

1.  1\. First, an extent needs to be created or defined.

2.  2\. Optionally, a device can be configured.

3.  3\. Finally, the target needs to be defined using information about the extent or device.

Extent in this context means the device or file that will ultimately act as the iSCSI device. From the iSCSI Target page, click the add circle for extent section. Here, you need to enter two things: the name which by default is **extent0**, and then the path to the extent. Here, you can choose a device like **/dev/ad0** or a file. For this example I will choose a file as this offers the greatest flexibility in that the same physical disk can be used to host several iSCSI targets (note however, that you get greater performance when using a whole device). The name for the file is in the format **/mnt/sharename/extentname** so in this example it will be **/mnt/store/extent0**. Finally, put in a size, in megabytes, for example 2048 for 2 gigabytes.

If you wish to combine several extents or devices, then next, you can create a device. This is an optional step and isn't required if you are using a single extent (whether a file or hard disk). Click the add circle for the device section. You need to enter a device name which by default is **device0**. Next, select how you want to combine the extents (either with RAID 0 or RAID 1, see chapter 3 for more information on RAID).

Finally, you need to define the iSCSI target itself. Click on the add circle for the target section. Enter a target name, the default is **target0**. Then tick the extent that makes up the target, in this case **extent0**. You also need to configure the authorized networks that can access this iSCSI target. This is similar to the way in which NFS networks are authorized.

iSCSI disks behave in exactly the same way as local disks in that you can only have one computer controlling that disk. Therefore, it is important to ensure that only one iSCSI initiator uses any particular iSCSI target. If two (or more) iSCSI initiators use an iSCSI target, there will be data corruption.

One way to ensure that only one iSCSI initiator is using the iSCSI target is to limit the authorized network to just one IP address. If my iSCSI initiator has an IP address of 192.168.1.100, then entering that address along with 32 from the drop down box will limit iSCSI connections to just that IP address.

However, if your iSCSI initiator doesn't have a fixed IP address (and is using DHCP), then you won't be able to permanently set the IP address. In this case, you can only limit the IP address to a range of addresses on your network. Assuming your FreeNAS server has a netmask of 255.255.255.0, enter the first 3 numbers of the IP address of the FreeNAS server (with . in between) and then finish it with dot zero (.0). For example, if the IP address of the FreeNAS server is 192.168.1.250, then you would enter 192.168.1.0 for the authorized network. Finally, select 24 (in other words 255.255.255.0) in the drop down box.

Once you have done this, apply the changes. The FreeNAS server will now create the 2GB file **extent0 in /mnt/store** and make the iSCSI target available on the network.

You will need to note the target name that is displayed as you will need this for the iSCSI initiator.

![iSCSI Target](img/4688_04_12.jpg)

## Testing the iSCSI Target with Another FreeNAS Server

To test the iSCSI target, we can get another FreeNAS server and connect it to the iSCSI disk on the iSCSI target FreeNAS server and see how the disk is used. To avoid confusion, I shall refer to the FreeNAS server where the iSCSI target is defined as the FreeNAS target server, and the server that will mount the remote iSCSI disk as the FreeNAS initiator server.

### Note

**Only One FreeNAS Server Available**

If you only have access to one FreeNAS server, you can still perform this test by using the same FreeNAS server. Here, the iSCSI initiator will loop back to the same FreeNAS server and use the defined iSCSI target. This isn't very practical but it proves that you have iSCSI working.

1.  1\. On the FreeNAS initiator server, go to the **Disks: Management** page and click on the iSCSI Initiator tab. Now click the add circle.

2.  2\. Enter a name for the iSCSI disk, say iSCSI0\. It isn't too important as it for information only (it is not using during iSCSI negotiation).

3.  3\. For the initiator name, enter: **iqn.1994-04.org.netbsd.iscsi-initiator:freenas**.

    These names do have a special format (see below) but the most important aspect is that they are unique. For uniqueness, you can vary the part after the last colon, **freenas2** etc.

4.  4\. For the target name, enter: **iqn.1994-04.org.netbsd.iscsi-target:target0**, which should be the target name you noted down on the FreeNAS target server.

5.  5\. Finally, enter the IP address of the FreeNAS target server. And then click **Add**.

### Note

iSCSI names may seem long and complicated but it is possible to understand them. They are technically called iSCSI Qualified Names or IQNs for short and they all start with the letters iqn.

Here is a reminder of the target name:

**iqn.1994-04.org.netbsd.iscsi-initiator:freenas**

After **iqn**. comes a date code specifying the year and month in which the organization registered the domain, here it is April 1994.

Then comes the domain name which is backwards: **iscsi-initiator.netbsd.org**

NetBSD is a sister project to FreeBSD, and the iSCSI parts of FreeBSD come from NetBSD, so their domain name is used.

Finally, there is a **:** and a local defined string to make sure the address is unique, in this case, **freenas**, which was the name of the target server.

If you have many iSCSI targets and initiators, you might consider using the last number of the IP address along with the word freenas to make unique names, e.g. freenas-250.

If you get any errors, try looking at the logs pages for clues to what went wrong:

*   **Diagnostics: Logs: System**

*   **Diagnostics: Information: Disks**

*   **Diagnostics: Information: iSCSI Initiator**

Also, there is more help in tracking down iSCSI problems in Chapter 9.

1.  1\. Now back to the **Disks: Management** page. Click the add circle and select the iSCSI device from the Disk drop down menu. It should read something like:

```
da0: 2048MB (NetBSD NetBSD iSCSI 0)

```

1.  2\. Add the disk in the normal way and apply the changes.

2.  3\. Now the disk can be formatted from the **Disks: Format** page and mounted on the **Disks: Mount Point: Management** page exactly as we have done previously in the quick start guide in chapter 2.

After that, you will be able to use the disk via any of the protocols of your choosing including CIFS, NFS, AFP, and FTP.

### Note

Did you notice that the iSCSI disk needs to be formatted and mounted just like a physical disk? This is because iSCSI disks are low level devices and the operating system that initiated the connection sees the iSCSI as if it were a normal hard disk in the system. This means it can be partitioned, formatted, and used like any local other disk.

![Testing the iSCSI Target with Another FreeNAS Server](img/4688_04_Graphics.jpg)

## Testing the iSCSI Target with Windows Vista

Microsoft Windows Vista comes with some built-in iSCSI initiator software (and it is also available as a separate download for Windows XP from the Microsoft website). This allows Windows to connect to an iSCSI device and use it as a local hard disk.

1.  1\. Having defined an iSCSI target on your FreeNAS server, go to the Windows machine and launch the iSCSI Initiator tool. It can be found in the Administrative Tools area of the Control Panel. Specifically: **Control Panel | System and Maintenance | Administrative Tools | iSCSI Initiator**.

2.  2\. Click on the **Discovery** tab and then click **Add Portal…** Enter the IP address of your FreeNAS sever in the **IP address or DNS name** field and leave the **Port** at the default of 3260\. Click **OK**. If the iSCSI component of the FreeNAS server is running correctly, you will be returned quickly to the **Discovery** tab. However, if a connection cannot be made to the iSCSI target, then an error dialog will appear. In the case of an error, go back and verify the FreeNAS iSCSI target.

3.  3\. Now click the **Targets** tab. The FreeNAS iSCSI target will be listed as something like: **iqn.1994-04.org.netbsd.iscsi-target:target0**. Click it and then click on **Log on…**

4.  4\. In the next dialog box, you have the chance to configure Windows to **automatically restore this connection when the computer starts**, which is useful if this is going to be a permanent storage option for this PC. Click **OK**. Windows will now connect to the iSCSI target and its **Status** will change to **Connected**. You can now click on **OK** to close the iSCSI initiator tool.

5.  5\. If you now open **Computer** you will see that nothing has changed. But don't be alarmed, this is as expected. The iSCSI connection is at the lowest level and it is similar to adding a new hard drive to your machine. Before that hard drive can be used, it needs to be formatted.

6.  6\. To initialize and format the drive, you need to use the Computer Management tool. To start it click the **Start** button, click **Control Panel, click System and Maintenance**, click **Administrative Tools**, and then double-click **Computer Management**. (If you are prompted for an administrator password or confirmation, type the password or provide confirmation). Finally, in the Navigation pane, under Storage, click **Disk Management**.

7.  7\. The iSCSI disk will be listed as a disk and it will be marked as **Not Initialized**. To initialize the disk, right-click on the disk label (where it says **Not Initialized**) and click **Initialize Disk**. Accept the defaults on the dialog that appears and click **OK**. The status of the disk will now change from **Not Initialized** to **Online**.

8.  8\. To format the drive, you need to create a partition and then format it. To do this, right-click on an unallocated region of the hard disk and then click **New Simple Volume**. In the **New Simple Volume** Wizard, click **Next**. Type the size of the volume you want to create in megabytes (the default is the maximum size) and then click **Next**. Choose a drive letter to identify the partition and then click **Next**. To format the volume with the default settings, click **Next**. Review your choices, and then click **Finish**.

9.  9\. You will now have a new hard disk on your computer. Whenever you use this hard disk, the commands to control, read and write to the disk will be sent using iSCSI to the FreeNAS server.

# Accessing Your Files Using HTTP and the Built-In Web Server

The web browser has become a ubiquitous tool on the desktop, and as such the ability to access files on the FreeNAS server while using a web browser has become an important file access method. Also, the protocol behind the web, HTTP (Hyper Text Transfer Protocol) is well understood by firewall devices. As such, using HTTP can mean that firewalls can be passed through, (of course, in a controlled manner and when desired) allowing you access to your firewalls when you find yourself on the other side of the firewall.

### Note

To use the built-in web server, you will need to use FreeNAS v0.69 or greater.

Having enabled a general-purpose web server in the FreeNAS server, the developers also added the ability to use PHP as well as HTML. However, the FreeNAS web server isn't intended to be a comprehensive web server. It does not support any type of database connectivity nor does it have advanced features such as URL re-writing or bandwidth quotas.

To enable the built-in web server, go to **Services: Webserver** and tick the enable box. The web server has 5 parameters:

| Parameter | Meaning |
| --- | --- |
| Protocol | Choose HTTP or HTTPS as the connection protocol. If you choose HTTPS you will need to paste an X.509 certificate in the Certificate field. |
| Port | TCP port for the server. Port 80 is the default for HTTP but it is used for the web administration interface. You need to choose another port here or move the web interface to another port (on the System: General Setup page) and use 80 here. Standard alternative ports include 81 and 8080. |
| Document root | Document root of the web server. This is where the web server will look for the web pages and files it is asked to serve. |
| Authentication | Give only local users access to the web server. You can define which directories need authorization to access them. Any locally defined user can then have access to that page. |
| Directory listing | A directory listing is generated if a directory is requested and no index-file (index.php, index.html, index.htm or default.htm) was found in that directory. This is useful if you just want to give access to some files without having to generate HTML or PHP files. |

Once you have entered the parameters, you need to click **Save and Restart**. Some parameter changes will require the server to be rebooted before they take effect.

The simplest parameter set to test the web server is to use HTTP on port 8080 with a document root of /mnt and with the directory listings enabled. To test the web server, enter the following URL in your web browser:

```
http://192.168.1.250:8080 

```

Where 192.168.1.250 is the IP address of your FreeNAS.

You will see a simple listing:

![Accessing Your Files Using HTTP and the Built-In Web Server](img/4688_04_14.jpg)

# Summary

In this chapter, we have looked at the different protocols that can be used to connect to the FreeNAS server from a variety of operating systems including Windows, Linux, and OS X.

We have seen that CIFS is available across the board and NFS is useful for Linux or OS X networks. Also, we looked at specialized protocols like RSYNC and UPnP, which allow the FreeNAS server to be used as a backup server or as a multimedia streaming server respectively.

In the next chapter, we shall look at the various user and system administration tasks.