# Chapter 5. User and System Administration

In this chapter, we will look at the different system administration tasks for the FreeNAS server as well as user administration. Areas covered include:

*   Adding new users

*   Using local user authentication with CIFS, FTP, AFP, and SSH

*   Rebooting and shutting down the server

*   Simple network management including configuring FreeNAS to use DNS and setting the default gateway

*   Getting status information about the server

# Introduction

In general, once the FreeNAS server is configured and running, it doesn't really need much attention, it should just work. However, there are lots of different features which can be configured. This chapter will look at some of the common administration tasks for setting up your FreeNAS server.

# Local User Management

Until now, we have only used the FreeNAS server in an "anonymous" mode, meaning that anyone can connect to the server and read, create, and delete files. This isn't always what you want, so FreeNAS has some user management features that change the way CIFS, FTP, and AFP allow users to connect to the server.

### Note

The 0.6 series of FreeNAS releases has rather blunt user management. The defined users have access or they don't have access, period. There is no granularity; for example being able to give some users read access while others have read/write access.

The roadmap for the 0.7 releases promises the ability to create a share (meaning a folder on a selected disk), with user/group/quota property on this share. This implies a greater level of control.

The first step to creating a user is in fact to create a group. Each user must belong to a group. Groups are sets of users who are associated with one another. So in your business, you might have a *sales* group and a *engineering* group. At home, you probably only want one group, for example *home*.

1.  1\. To create a group, go to **Access: Users and Groups** and click on the **Group** tab.

2.  2\. Now click on the add circle.

3.  3\. The form is very simple; you need to add a name and a description. For example sales and "The sales people".

4.  4\. Now click **Add** and then apply the changes.

### Note

Only a-z, A-Z, and 0-9 are supported in the group name. _ (underscores) and spaces are not supported, neither are punctuation characters like $%&* etc.

Now that you have a group created, you can create a user.

1.  1\. Click on the **Users** tab.

2.  2\. And then on the add circle.

    There are four mandatory fields:

    *   **Login:** This is the unique login name of user. If the user already has a login name on other servers or workstations, like a Windows user name or a Linux user name, it is best to keep it the same here. This way the user doesn't need to try an remember an extra username and also some programs (particularly Windows) try and log in with the Windows user name before asking which name it should use. Keeping them the same will ease integration.

    *   **Full Name:** The user's full name. Often, the login name is an abbreviation or short name for the user like john, gary. Here you need to enter the full name so that it is easy to tell which login name belongs to which person.

    *   **Password:** Their password (with confirmation). The colon ':' character isn't allowed in the password.

    *   **Primary Group:** The group to which they belong, for example sales.

    ![Local User Management](img/4688_05_01a.jpg)
3.  3\. To finish, you need to click **Add** and apply the changes.

You now have a user added to your FreeNAS server.

There are three more optional fields when adding a user: Home Directory, Full Shell, and Administrator, and we shall look at these in a moment, but first let's look at what effect adding a user has on the rest of the FreeNAS server.

## Using CIFS with Local Users

To use the users you have defined with Windows networking, you need to go to the **Services: CIFS/SMB** page and change the **Authentication** field to **Local User**. Then click **Save and Restart** to apply your changes.

What this means is that only authenticated users can now access the FreeNAS shares via CIFS.

### Note

In version 0.6, this user authentication is for all the shares, the user has access to everything or nothing. This should change with 0.7.

When trying to connect now from a Windows Vista machine, a window pops up asking for a user name and password.

![Using CIFS with Local Users](img/4688_05_04.jpg)

Once authenticated, the user has access to all the user shares on the FreeNAS server.

## FTP and User Login

On the **Services: FTP**, there are two fields that control how users log in to the FreeNAS server:

*   **Anonymous login:** This allows you to enable anonymous login. This means the user connects with the user name **anonymous** and any password.

*   **Local User:** This enables a local user login. Users log in using the user name and passwords defined in the Access: Users and Groups page.

The two can be used together; however, they do negate one another in terms of security. It is best to run the FTP with either anonymous logins enabled and local user logins disabled or vice versa. If you run with both enabled, then people can still log in using the anonymous method even if they don't have a user account and so, it diminishes the benefits of having the user accounts enabled.

Other than the security benefits, another advantage of local user login with FTP is that you can define a home directory for the user and when the user logs in, they will be taken to that directory and only they have access to that directory and those below it. This effectively offers each user their own space on the server and other users cannot interfere with their files.

To get this working, you need to create a directory on your shared disk. You can do this with any of the access protocols CIFS, NFS, FTP, and AFS. You need to connect to the shared disk and create a new folder.

Then, in **Access: Users**, either create a new user or edit an existing one (by clicking on the 'e' in a circle). In the **Home directory**, you need to enter the directory for that user. For example for the user john, you might create a directory cunningly named *john*. Assuming the disk is named *store* (as per the quick start guide) then the path for the home directory would be: /mnt/store/john.

Click **Save** and apply the changes. Now when John logs in using the user name *john* he will be taken directly to the *john* directory. He doesn't have access to other files or folders on the store disk, only those in *john* and any sub folder.

### Note

**chroot() Everyone, but Root**

In the advanced settings section of the Services: FTP page, there is a field called **chroot() everyone, but root**. What this means is that when a user logs in via FTP, the root directory (top or start directory) for them will be the directory set in the **Home directory** field. Without this set, the user will log in to the server at the physical / and will see the server in its entirety including the FreeNAS and FreeBSD system files. It is much safer to have this box checked. The exception to this is the user root (which in FreeBSD terms is the system administer account). If **Permit root login** is enabled, then the user root can log in and they will be taken to the root of the actual server. This can be useful if you ever need to alter any of the system files on the FreeNAS, but this isn't recommend unless you absolutely know what you are doing!

## Authenticating AFP Users

Like CIFS and FTP, the Apple Filing Protocol (AFP) can also use the local user authentication features of FreeNAS.

In the **Services: AFP** page, there are two options for controlling access to the server via AFP:

*   **Enable guest access**, meaning that anyone can connect without giving a username or password. The users have full read and write access.

*   **Enable local user authentication**, meaning that only users defined on the FreeNAS server (on the **Access: Users** page) can access the server. The user name and password set in the FreeNAS server need to be given to authenticate.

Like FTP, the two can be used together, however, they do negate one another in terms of security. It is best to run the AFP service with either guest logins enabled and local user logins disabled or vice versa. If you run with both enabled then people can still log in using the guest account even if they don't have a user account and so it reduces the benefits of having the user accounts enabled.

With just local user authentication enabled, initial connections from an Apple Macintosh will fail. In the top right-hand corner of the Finder window, there is a button labeled **Connect As..**. Use that to enter a user name and password.

![Authenticating AFP Users](img/4688_05_05.jpg)

## Connect to the FreeNAS Server via SSH

One of the services that hasn't been mentioned much in this book so far is Secure Shell access or SSH for short. It is really for advanced users and it will be used to connect to the server in Chapter 10, when we look at FreeBSD and command line tools available.

However, SSH depends heavily on the local users defined on the server and as such it is worth looking at now.

SSH is a network protocol that allows data to be exchanged over an encrypted (secure) channel between two computers. It is commonly used as a secure command line interface to a remote computer. This means that you can access the command line interface of the FreeNAS server from a remote computer without having to access the keyboard the and monitor of the FreeNAS server. On the FreeNAS server, it is also used in conjunction with the Unison suite of programs. Unison uses SSH to log in to the server and start the synchronization process.

On the **Access: Users: Add** page, there is a field called **Full Shell**, which when enabled, gives that user access to the FreeNAS server via SSH.

To test SSH connectivity:

1.  1\. Create a user and make sure that **Full Shell** is enabled.

2.  2\. Go to the **Services: SSHD** and enable the service.

3.  3\. Make sure that Password authentication is ticked.

4.  4\. Click **Save and Restart**.

5.  5\. Connect to the FreeNAS server using a SSH client (see below).

### Note

**Password Authentication**

It is possible to connect to the FreeNAS server without giving a user name and password but by relying on an exchange of encryption keys that verify that you are who you claim to be. With Password Authentication enabled, you are able to log in just using a username and password.

You can connect to the FreeNAS server via the command line program ssh using Linux and Mac OS X. For Windows, you will need a SSH client, the best one is called Putty ( [http://www.chiark.greenend.org.uk/~sgtatham/putty/](http://www.chiark.greenend.org.uk/~sgtatham/putty/)). We will look in more detail at Putty in Chapter 10.

From a Linux or OS X command line type:

```
ssh -l john 192.168.1.250 

```

### Note

Don't forget to change the address to that of your FreeNAS if you aren't using the default.

The `-l` tells the SSH program which user you want to use as the login name, in this case, I have chosen `john`.

The first time you log in, you may be asked if you trust the remote machine as you are about to enter into encrypted communications with it. It should read some thing like this:

```
The authenticity of host '192.168.1.250 (192.168.1.250)' can't be established.
DSA key fingerprint is b2:d0:99:cb:6e:b2:53:95:4d:f6:b3:02:1d:bc:36:db.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.250' (DSA) to the list of known hosts 

```

Answer yes and then type in the password for the user. You are now connected to the FreeNAS server via SSH. From here, you can access the command line tools of the FreeNAS server. See chapter 10 for more details.

## Services that Don't Use Local User Accounts

Not all services provided by FreeNAS use local accounts for authentication, most notably NFS. This requires a note of caution. If you have NFS enabled, and are using local user authentication for CIFS, FTP, and AFP be aware that users can still connect to the FreeNAS server using NFS without any username and password. This is an easy way for people to circumvent the local user authentication process.

# Using FreeNAS with the Microsoft Active Directory

Until now, we have defined all the user information locally on the FreeNAS server. This is fine for small networks but if you have a large business network, you may already have Microsoft's Active Directory deployed. FreeNAS can use the user database of a Microsoft Active Directory (Windows 2000/2003) to authenticate user names and passwords and therefore, remove the need to define users locally.

When Active Directory is being used, the FreeNAS server will authenticate users using the directory for the following services: CIFS, FTP, SSH, and Unison.

### Note

**Pre-Windows 2000**

FreeNAS is considered as a pre-Windows 2000 client and as such the Active Directory must be configured with pre-Windows 2000 compatibility.

Assuming the Active Directory is installed and running:

1.  1\. Go to **Access: Active Directory**.

2.  2\. Tick the Enable check box in the title bar of the table.

3.  3\. Enter the Active Directory server name in the **AD server name** field. For example the Windows Server 2003 server on my test network is called WS2003, so I entered WS2003.

4.  4\. Enter the IP address of the Active Directory server in the **AD server IP** field.

5.  5\. Enter the domain name for Active Directory. This is in pre-Windows 2000 format.

6.  6\. Enter the domain administrator account user name (probably Administrator) and the password.

7.  7\. Finally, click **Save**.

To check if the FreeNAS is able to communicate with the Active Directory correctly:

1.  1\. Go to **Diagnostics: Information**

2.  2\. Click the **MS Domain** tab.

This will test the connecting to the Active Directory.

A successful test will look like this:

`Accessibility test to MS domain:`

`Results for net rpc testjoin:`

`Join to 'FREENAS' is OK`

`Ping winbindd to see if it is alive:`

`Ping to winbindd succeeded on fd 4`

`Check shared secret:`

`Checking the trust secret via RPC calls succeeded`

After the Active Directory is configured, CIFS, FTP, SSH, and Unison authentication will rely **only** on account information in the Active Directory.

The authentication method for CIFS/SMB is automatically changed to *Domain* when the Active Directory is configured for use.

To check this, go to **Services: CIFS/SMB** and notice that **Authentication** is now set to **Domain**.

To test the use of Active Directory, try connecting to the FreeNAS server via CIFS, FTP or SSH and use account information from the Active Directory.

# System Admin

Some of the common administration tasks for the system admin:

## How to Change the Web GUI User Name and Password

When you first install or boot-up the FreeNAS server it has a default username and password for accessing the web GUI. These are *admin* and *freenas* respectively. If your FreeNAS is in an environment where others could potentially access the FreeNAS server and change settings, either maliciously or by accident/curiosity, it is advisable that you change the password and possibly even the username for access to the web GUI.

To change the password, go to **System: General** and click on the **Password** tab. Enter the current web GUI password (which is probably *freenas*) and then enter the new desired password twice, the second time for confirmation to make sure you entered the right characters. Now click on **Save**.

Once saved, you will automatically asked to log in again. So enter the username (probably *admin*) and then the new password you entered.

### Note

Choosing a Good Password

### Note

You need to choose a strong password, which means one that is difficult to guess.

It is best to choose a password that you will remember, that way you don't have to write it down or leave it in the open.

But avoid using a complete word from a dictionary or a name.

The more characters your password contains, the harder it is for someone to guess it. Often a longer but simple password can be safer than a short, complex one; it also has the benefit that it is easier to remember.

Use a combination of capital and lowercase letters, numbers, and standard symbols (! @ # $ % ^ & *). The FreeNAS web GUI password is case-sensitive, which means that a capital letter Z is different from a lowercase z.

Don't use your birthday, your child or pet's name, your phone number etc. Personal information can be easy for someone to figure out.

Avoid the obvious password like "123456", "test" or "password."

It is also possible to change the default username from *admin* to another name of your choosing. To change it, go to **System: General** and enter a new username in **username** field. Click Save to apply the changes. You will be asked to log in again, this time you will need to enter the new username.

## Rebooting and Shutting Down

It is possible to shutdown and reboot the FreeNAS server from the web interface. In the web interface, rebooting and shutting down work in very similar fashions. To reboot, you need to go to **System: Reboot** and to shutdown go to **System: Shutdown**.

There are two types of reboot/shutdown, immediate (now) or scheduled. To reboot or shutdown immediately, go to the respective page and click on **Yes** in answer to the question **Are you sure?** Once you have clicked on **Yes**, the system will start to shutdown/reboot.

For scheduled reboots or shutdowns, click on the **Scheduled** tab.

![Rebooting and Shutting Down](img/4688_05_01.jpg)

Reboots and shutdowns are scheduled by selecting which minute, hour, day, and month you want the reboot or shutdown or occur. This is a re-occurring event and as such you can also choose a day of the week rather than day of the month for the reboot.

To enable the scheduled reboot or shutdown, tick the enable tick box and then select the time you want for the reboot or shutdown. For example, to reboot the server every Sunday at 4:35PM you would select:

*   35 from the minutes section

*   16 from the hours (remember it is a 24 hour clock)

*   Sunday from the week days

*   Days and months would remain empty

## How to Set the Hostname of the Server

Your FreeNAS server has a name, known as the hostname. This name uniquely identifies the server. If you have more than one FreeNAS server on your network, you need to change the hostnames so that each one is unique.

To change the hostname, go to **System: General** and enter a new hostname in the **Hostname** field. If you are deploying your FreeNAS in a network environment, which also uses Internet domains names for identification, then you can also enter a domain name (e.g. yourcompany.com) in the **Domain** field.

## Configuring the Web Interface to use HTTPS

HTTPS (Hypertext Transfer Protocol over Secure Socket Layer) is a way for your web browser to connect to the FreeNAS server over a secure encrypted connection. If your FreeNAS server is in an environment where people can "snoop" on the network to discover the passwords being used, it is best to use HTTPS. This is also true if you have configured your network in such a way that your FreeNAS can be administered from the Internet (probably through a firewall). This can be useful when the FreeNAS server is in a different location than the person responsible for administering it.

To enable HTTPS, go to **System: General** and check the HTTPS checkbox in the **WebGUI protocol** field. After you have clicked **Save**, you will need to reboot the server for the changes to be applied.

Once rebooted, you now connect to the web GUI with:

```
https://192.168.1.250 

```

Notice that the URL starts with `https` and not `http` as before. Once you connect to the server, your browser will almost certainly tell you that the certificate for this secure connection is signed by an unknown authority. It will question if you can trust server or not. Don't be alarmed, this isn't a problem. Accept the certificate and proceed. Depending on your browser, you may be able to accept this certificate permanently so as to not receive the warning in future. You can now log in as usual.

### Note

When a web browser makes a connection to a secure site using HTTPS, the web server presents the browser with a certificate identifying itself. This certificate contains unique, authenticated information about the certificate owner. To be sure that this information is correct, a 3^(rd) party needs to vouch for the certificate and say it is trustworthy. This 3^(rd) party is called a Certificate Authority (CA) and they verify the identity of the certificate owner. They establish a level of trust. If *they* say this certificate is valid then I *trust* them.

Web browsers are preprogrammed with an accepted list of Certificate Authorities and when they see a CA that isn't on that list, the web browser will question if this is a trustworthy certificate.

FreeNAS uses a self-signed certificate, which means it vouches for itself. This would be a problem for a major website that accepts online payments but for FreeNAS it is acceptable as it isn't the trust of the server that is required but rather the ability to talk securely. Once you accept the certificate, the secure communications will begin with the FreeNAS server based on some of the information in the certificate.

## Changing the Web Interface Port

Sometimes, it can be useful to change the web server port from the default value of 80 (443 for HTTPS) to something else. For example if you want to administer your FreeNAS server from the Internet and you need to configure your firewall with a forwarding rule, then it can be helpful to put the web server on a different port.

To change the port number, go to **System: General** and enter the new port number in **WebGUI port** field. When you change the port number, you need to reboot the server for the settings to take effect. Once rebooted, you need to use a new URL:

```
http ://192.168.1.250:8080

```

Where 8080 is the new port number you choose and of course 192.168.1.250 is the IP address of your FreeNAS server, you will need to change this accordingly.

## How to Set a DNS Server

Several of the services on the FreeNAS server like the Network Time Protocol service and the sending of email status reports require the FreeNAS to use DNS. DNS is an Internet service that translates domain names into IP addresses. Each time you use a domain name (in your web browser for example), DNS is used to translate the name into the corresponding IP address. To find different servers on the Internet, like an email server, FreeNAS also needs to use DNS.

Your can ascertain the DNS servers you need to use from one of several sources:

*   If you are using the FreeNAS on your company network, you may well have DNS servers on your LAN. You need to speak to your network administrator to get the correct information.

*   If you are using the FreeNAS server at home, then your DNS could be the address of your ADSL/DSL modem.

*   Alternatively, for the home user, the DNS servers could be those of your ISP. You need to contact your ISP (or search their website) to find out what DNS servers you should use.

If you can't find which DNS servers to use, then copy the settings from an existing machine on your network. For example, to find out what your DNS settings are in Windows XP, Windows Server 2003, and Vista, click on **Start** then **Run..**. and type **cmd** and press *ENTER*. A command prompt will appear. Now type:

```
C:\>ipconfig /all 

```

The result will list various bits of information about your network connection. In the **Ethernet adapter Local Area Connection:** section, there will be information about the DNS servers.

![How to Set a DNS Server](img/4688_05_02.jpg)

## How to Set the Language for the Web Interface

Along with English, the FreeNAS web interface comes in several different languages including: Bulgarian, Chinese, Dutch, French, German, Greek, Hungarian, Italian. Japanese, Norwegian, Polish, Portuguese, Romanian, Russian, Slovenian, Spanish, and Swedish.

To change the language of the web interface, go to **System: General Setup** and select the desired language from the **Language** drop down box. Once you click **Save**, the language should change straight away, however, sometimes because of browser caching you might need to click to another section of the web interface before you see the language change.

## Date and Time Configuration

All PCs have a clock and can keep a track of the date and time. Keeping the date and time correct is important for several reasons including:

*   Files will be marked with the correct creation and modification time stamps.

*   The scheduled reboots and shutdowns will occur at the right time.

*   The FreeNAS server can act as a time server to Windows machines.

*   Scheduled status reports sent by email will be sent at the right time and will have the correct time and date on them.

*   Log files will have the right time stamps, which aids any diagnostics.

The date and time can be set in one of three ways:

*   Set it in the computers BIOS

*   Set it via the web GUI

*   Configure automatic time adjustment via NTP

The time and date configurations are made on the **System: General** page.

The first thing to set is your time zone, this is done by selecting it from the **time zone** drop down box. You need to select the nearest location to you (normally, the capital city of the country you are in or a state capital), for example **America/New York**. You can also select the time zone in a more manual fashion, for example **Etc/GMT-1**. Use the **Save** button to apply the changes.

If you need to set the date and time, you can choose to set the date and time by entering them into the **System time** field. The format is mm/dd/yyyy hh:mm. You can also use the icon to select the date and time from a simple calendar widget. It is worth noting that the seconds cannot be set using this method. Use the **Save** button to apply the changes.

You can choose to configure the date and time automatically using the Network Time Protocol (NTP). NTP is a way of synchronizing computer clocks over the Internet. The time server normally has a very accurate atomic clock attached to it, and other computers ask it the time and set their clocks accordingly. The protocol has special algorithms to allow for the delays in sending and receiving requests over the Internet. With this, protocol clocks can be accurate within a fraction of a second.

To enable NTP, tick the **Enable NTP** field. Next, you need to enter which time server you want to use, `pool.ntp.org` is the default and should be just fine for your usage. If your FreeNAS doesn't have access to the Internet, then you will need to find an NTP server on your local network. Although using one NTP server is sufficient, the recommendation from the NTP people is to specify 3 servers:

*   `0.pool.ntp.org`

*   `1.pool.ntp.org`

*   `2.pool.ntp.org`

Use a space to separate the hosts.

### Note

The `pool.ntp.org` project is a big virtual cluster of timeservers striving to provide reliably easy to use NTP service for millions of clients without putting a strain on the big popular timeservers.

To spread the load and handle the occasional down server, the NTP project have created random sets of servers using the 0, 1, and 2.pool.ntp.org names. The servers in each set randomly change every hour.

The last thing to set is the interval, in minutes, between network time syncs. The default is 300, which is every 5 hours. This should be sufficient for most needs.

For the NTP protocol to work correctly, you need to make sure you have defined at least one DNS server for name resolution. See the *How to set a DNS server* section for more details.

## How to Disable Console Menu

Beyond initial network settings, the FreeNAS server uses the web interface for the majority of its configuration. However, the console menu remains active even when you don't need it anymore. This can be a security risk as anyone with access to the keyboard and monitor attached to the server can change settings either deliberately or by accident.

To disable the console menu, go to **System: Advanced** and check the **Disable console menu** checkbox. Once you click **Save**, you will need to reboot for the console to be disabled.

## How to Stop the Startup and Shutdown Beeps

When FreeNAS starts up and shuts down, it plays a few melodious beeps to indicate that it has started. If you want to disable them, then go to the **System: Advanced Setup** page and tick **the System Beep** box, this will disable the speaker beep on startup and shutdown.

## Adding Predefined Network Hosts

In the rare case that you don't have access to a DNS server but yet you still want to use the NTP protocol or email status reports, you will need to define the NTP server or the email (SMTP) server manually. You can do that on the System: Hosts page.

To add, say an NTP server, with the address 86.125.34.112, click the add circle and then enter the host name of the server and its IP address. You can also add a description.

![Adding Predefined Network Hosts](img/4688_05_06.jpg)

## Reset the Server to the Factory Defaults

If you want to reset the FreeNAS server back to its original installation state (or Live CD boot state), which is often known as the factory settings (metaphorically: the settings it had when it left the factory) then you can do this on the **System: Factory** defaults.

If you click **Yes,**, the FreeNAS server will be reset to factory defaults and will reboot immediately. The entire system configuration will be overwritten. The LAN IP address will be reset to 192.168.1.250 and the password will be set to 'freenas'.

### Note

It is worth noting that when you reset the server to the factory defaults, the data on the disks won't be touched. This means that you can reset the server and then reconfigure it to find your data. But be VERY CAREFUL, if during the re-configuration phase you reformat a disk or apply a different configuration than before, you are likely to lose data.

## Simple Network Administration

On the **Interfaces: LAN** page, you can perform some limited network administration tasks such as defining a new IP address for the FreeNAS server and defining a default gateway for the server.

The default gateway is the IP address of another device on the network that will accept network traffic for forwarding on to another network. The most common default type of default gateway in the home situation is the ADSL/DSL modem/router that is provided by your Internet Service Provider (ISP). The modem's job is to route traffic from your home network on to the Internet and the reverse.

In a business environment, the setup may be same or if you have a comprehensive network infrastructure, the default gateway will be a network device that connects your segment of the network to the rest of the corporate network.

You will almost certainly need to set your default gateway to give the FreeNAS server access to the Internet ,which you will especially need if you are using an Internet NTP service.

You need to ask your network administer or ISP for the details of the default gateway. Another way to ascertain the default gateway settings is to see what they are on another machine on your network.

For example, to find out what your default gateway is in Windows XP, Windows Server 2003, and Vista, click on **Start** then **Run..**. and type **cmd** and press ENTER. A command prompt will appear. Now type:

```
C:\>ipconfig /all 

```

The result will list various bits of information about your network connection. In the **Ethernet adapter Local Area Connection:** section, there will be information about the Default Gateway.

![Simple Network Administration](img/4688_05_07.jpg)

## Disabling Bonjour/ZeroConf

By default, FreeNAS will announce which services it provides (CIFS, AFP etc) with the Zeroconf protocol. This will permit other computers on the LAN to detect that there is a server that provides network services. Zeroconf/Bonjour is used by Apple OS X operating system and recent Linux distributions.

If you wish to disable this feature, then remove the tick from the **Zeroconf** field on the **System: Advanced Setup** page.

## Getting Status Information About the Server

FreeNAS comes with some excellent tools to monitor the status of your server. These are all grouped together under the heading **Status** in the left-hand menu column.

The different status categories are:

*   **System:** This is the summary page that is displayed when you first connect to the web interface. It will tell you, among other things, which version of FreeNAS you are running, how long the server has been up and running, the amount of the memory being used, and the disk space usage.

*   **Process:** This will show you some summary information about the processes running on your FreeNAS server and a list of the top processes.

*   **Interfaces:** This displays a summary of the network interfaces in the server and some simple statics about their traffic loads.

*   **Disks:** Here you can see a list of disks currently configured on your system including each disks size, description, and status.

*   **Wireless:** This displays a summary of the wireless network interfaces in the server and some simple statics about their traffic loads.

*   **Graph:** This page is divided into 2 sections, Traffic graph and CPU load. The first is a real time graph showing the amount of traffic on the network card and the second the amount of CPU usage. Both work in a 2 minute window.

### Note

FreeBSD is able to run many programs at the same time and FreeNAS uses this multitasking ability of FreeBSD to run the web server, the CIFS server, the FTP etc simultaneously.

Each of these programs or services runs as a separate **process**. Each process uses time on a system's CPU, as well as other system resources such as memory and disk space.

The various status pages help you keep a eye on those process and the resources being used by the FreeNAS server.

![Getting Status Information About the Server](img/4688_05_08.jpg)

### Note

The output from the processes status page can seem a bit strange if you are not used to UNIX-type systems, here is a quick guide:

**PID:** Each process has a unique process ID called the PID.

**SIZE:** SIZE is the total amount of memory used, in kilobytes, by the process.

**RES:** Is the actual amount of physical memory being used by the process. This can different from SIZE as FreeBSD shares regions of memory (called pages) between processes that have the same values and have not been changed.

**STATE:** This is the current state of the process (one of sleep, wait, run, idl, zomb or stop).

**TIME:** This is the number of system and user CPU seconds that the process has used.

**WCPU:** This is percentage of the CPU being used at that moment.

**COMMAND:** Is the command line which started the process.

## Sending Status Report by Email

FreeNAS has the ability to send status reports by email at pre-programmed times. To configure these reports, go to **Status: Email Report**. FreeNAS doesn't include an email server (SMTP server), so you need to tell it which server to use. If you are a home user then this will be the SMTP server provided by your ISP. For business environments, you will need to ask your network administrator where is the address of the SMTP server on your network. If you are addressing the SMTP server by its domain name (e.g. mail.myisp.com) then you need to be sure that DNS is configured so FreeNAS can look up the IP address of the mail server. You could also predefine the server in System: Hosts.

If your SMTP server requites authentication (which is almost certainly found to be true if you are using the SMTP server provided by your ISP) then tick the **Authentication** box and enter the appropriate user name and password.

Next, you need to set who the email is from and where you want it to go. The **To email** needs to be the address where you want to receive the email and the **From email** the sender of the email. For the **From email**, you need to enter a valid address as any notifications of failure to deliver the email will be sent there. Also, you want the **From email** address to be valid so as not to antagonize any spam filtering that your email system may have.

You can fill in the desired subject line of the email and also fine tune what is included in the email report.

Finally, you need to define the time when these status messages will be sent. The table is exactly the same as the one used for scheduled reboots/shutdowns. Messages scheduled by selecting which minute, hour, day, and month you want the messages to be sent. This is a re-occurring event and as such you can also choose a day of the week rather than day of the month for the messages. For example, to send a status message on the 1st of every month at 9:00AM you would select:

*   0 from the minutes section.

*   9 from the hours.

*   1 from the days.

*   Select *All* for the months.

*   The week days would remain unused.

# Summary

In this chapter, we have looked at the different administration tasks for setting up your FreeNAS server including user management and its effects on the access protocols like CIFS and FTP. We have also looked at some of the system administration options as well as simple administration tasks like shutting down and rebooting.

In the next chapter, we shall take a more in depth look at configuring disks on the FreeNAS server including setting up software RAID.