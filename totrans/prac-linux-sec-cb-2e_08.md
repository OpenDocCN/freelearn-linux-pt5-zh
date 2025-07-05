# Linux Security Distros

In this chapter, we will discuss:

*   Kali Linux
*   pfSense
*   **Digital Evidence and Forensic Toolkit** (**DEFT**)
*   Network Security Toolkit (**NST**) 
*   Security Onion
*   Tails OS
*   Qubes OS

# Kali Linux

Kali is a Debian-based Linux distribution developed for the purpose of security testing. Having hundreds of penetration testing tools preinstalled, Kali is a ready-to-use OS. We can run it off a live CD, USB media, or in a virtual machine.

The latest versions of Kali have gone through some major changes, one of them being, since Kali 2.0, that the entire system has been shifted to a rolling release model. Now we can simply install Kali 2.0 or a higher version on our system and get the latest versions of the tools in it through normal updates. This means we don't have to remove the existing OS and install the latest version of Kali 2.2 to get the latest stuff.

To explore Kali 2.2, download the latest version of it from its official website at [https://www.kali.org/downloads/.](https://www.kali.org/downloads/)

We can download the ISO and then burn it to a CD/DVD or create a bootable USB device. We can even download **Kali Linux VMWare**, **VirtualBox** or **ARM** images from the same link.

The latest version of Kali includes major changes in terms of its updated development environment and tools. We will explore these changes to understand what the differences are.

To start using Kali, we can either install it or use it through the live option.

When we boot Kali, we notice that the GRUB screen has changed and made simple to use as shown:

![](img/b3582daa-ca21-442a-b0a0-8ab0b57860ba.png)

The desktop environment of Kali has moved to GNOME 3, with a new redesigned user interface. We can see these changes at the login screen here, which has also been redesigned:

![](img/7e11bda8-576e-41c6-bb25-49ace10bb5cf.png)

The entire desktop along with the panel as well as the Applications menu has been redesigned/reconstructed:

![](img/9ffd78d4-c6bc-41af-a7dd-57ecac7cdc7b.png)

We can also access the tools by clicking on the Menu icon, which is at the bottom of the sidebar, as shown. This way we can see all the tools at once:

![](img/c22ec97e-6fa0-4578-a853-53f152d8b1f8.png)

Kali includes a built-in screencasting option that is actually a part of GNOME 3\. On the top, right click on the Recorder icon and we get the option to Start recording. Now you can make videos of whatever you are doing on Kali with a single click:

![](img/1a94df18-9c05-4115-89d0-b9ab090096f0.png)

If we wish to access the Settings menu of Kali, we will see that it is missing under the Application menu. To access Settings, click on the Power icon on the top right and a menu pops out.

In this menu, we can see the Settings icon at the bottom left:

![](img/2d828dd8-f419-4921-bcc5-b46865e8f2e5.png)

We can also see an option for VPN in this menu. Using this option, we can configure the VPN settings.

When we click on the Settings icon in the previous step, we get our Settings menu as shown here. Now make changes in the system's settings as per the requirements:

![](img/0a93c275-5a2e-452b-8930-16672456fbef.png)

Scroll down and click on Details to see more information about Kali Linux.

We can see details about the system in the screen shown here. This includes information about the GNOME version:

![](img/efb417ad-53df-46d9-8b12-34ef851e5874.png)

Whenever we wish to update Kali, just click on the Check for updates button in the Details window.

If your system is already up to date, a message will appear as shown here, otherwise the available updates can be downloaded:

![](img/6586344c-206a-4217-a46e-2cb8b0e40fbf.png)

When we boot Kali, we will see that the desktop screen has changed. We now have a sidebar on the left side of the screen that helps us to access applications easily.

The Application menu on the top left contains all the tools under different categories. The applications can also be accessed by using the Menu icon on the sidebar at the bottom.

Next we see that Kali includes an inbuilt screen recording tool that can be accessed from the menu on the top right. In the same menu, we now have the option to access the menu for system settings.

Then we can see the option to check for system updates to keep Kali updated.

Kali has the updated tools included and is built to pull updates from Debian to ensure that the system is always up to date, and to also ensure that the security updates are implemented on a regular basis.

# pfSense

As a network administrator, having a firewall and router in place is essential. When we talk about setting up a firewall, we have the option to either simply install a preconfigured firewall from any vendor or set up our own firewall system.

pfSense is one amazing software distribution, if you wish to set up your own firewall from scratch. It is an open source distribution based on FreeBSD and is specially designed to be used as a firewall that can be managed easily through a web interface.

# Getting ready

1.  Download pfSense from this link: [https://www.pfsense.org/download/mirror.php?section=downloads.](https://www.pfsense.org/download/mirror.php?section=downloads)
2.  Choose the correct computer architecture and platform as per your requirement.
3.  After downloading pfSense, burn the ISO file to CD/DVD media, or you can even create a live bootable USB media.
4.  We also need a system with two network interface cards to install and configure pfSense.

This system will be dedicated to the firewall functionality and will not be usable for any other computing task such as web browsing or so. It is recommended you use an old computer such as a Pentium 4 machine, or even a virtual machine can be set up and used for this purpose.

# How to do it...

Follow the steps here to set up and configure the pfSense firewall:

1.  When we boot our system with the pfSense CD/DVD or USB device, the splash screen appears as shown here. Press *6* to Configure Boot Options:

![](img/98e5d159-42ff-43d8-b8b3-59a73167bc41.png)

2.  In the next screen, again press *6* to turn on Verbose then press *1* to return to the previous screen.

When back on the first screen, press *Enter* to boot pfSense.

3.  pfSense will start booting. During the booting process, we get a screen as shown here:

![](img/4f285430-7a89-4427-8c24-6fd8a5a5f855.png)

Press *I* to install pfSense. Choose the option quickly within the 20 seconds count.

4.  The next screen asks you to Configure Console. Choose the option Accept these settings and press *Enter* to continue.
5.  In the next screen, choose Quick/Easy Install, if new to pfSense, otherwise you can choose Custom Install for more advanced options during the installation process.
6.  Press OK to continue with the installation. The installation process will start now.
7.  During the installation, you will be asked to choose which kernel configuration to install. Select Standard Kernel as we are installing pfSense on a Desktop or PC. If installing on an embedded platform such as router boards, we can choose the option, Embedded kernel.
8.  After this, the installation will continue. Once complete, select Reboot  and press *Enter* to complete the installation.

9.  During the reboot process, the default username and password of pfSense will be displayed as shown:

![](img/d60c027d-cfc1-44ce-ad12-ff8705b0a68a.png)

10.  After rebooting, we now have to configure our interface cards according to the network configuration. The name of the two interfaces will be displayed as shown. These names may be different in your case:

![](img/1d8efd4a-bbe6-41d6-926d-055dd8d54068.png)

11.  Now you will be asked Do you want to set up VLANs now?  Enter `n` for NO at this moment.
12.  Now we need to enter the interface name to be used for WAN. In our case it is  `le0`. Enter the name as per your configuration.

13.  Next enter the name of the interface to be used for LAN. For our example, it is `le1`:

![](img/cb1363a0-f67a-40f7-b3f3-98a330708187.png)

Then press *Y* to proceed with the settings.

14.  Once the interface has been set, we will get the pfSense menu as shown here:

![](img/7bf83053-55f4-46cf-b741-d5d36b020306.png)

If the IP addresses for the WAN and LAN interface are not set properly up to this step, we can set the IP address manually by choosing option  *2*  from the preceding menu.

15.  Choose the interface to configure and then provide the IP address for the same:

![](img/9efc4145-ef94-4aae-a71e-5e154ed9cd47.png)

16.  Next, enter the subnet and the default gateway:

![](img/a15a6096-92ac-4945-bdf3-e09cc30b3a3f.png)

17.  Follow the same steps for the LAN interface. When done, a link will be shown on the screen, which can be used to access the `pfSensewebConfigurator`  interface:

![](img/c2f110d6-7874-410e-a6fb-745aec892561.png)

In our case it is `http://192.168.1.115.`

18.  Now access this link from any browser on a system on the same local network as the pfSense system. Once we access the link, we get a login screen as shown here:

![](img/bd4f7874-fb34-418e-b877-2b986abfba91.png)

Enter the default username `admin` and default password `pfsense` to log in. These details can be changed later after logging in.

Once logged in successfully, we get the main dashboard of pfSense:

![](img/983035db-2b83-412f-a17a-c8fd84d0f95f.png)

# How it works...

We boot from the pfSense CD/DVD and then choose the option to install the OS on our system.

To install pfSense, we use the option I during boot and then we use Quick/Easy Install. After the installation completes, we set up the two interface cards. The first card is configured according to the outside network, using the option Set interface IP address from the menu. Then we configure the IP address, subnet, and gateway address.

Next, we repeat the same process for the second card, which we configure according to the local network.

Once the configuration is done, we can use the IP address of the second card to access the web interface of pfSense from any browser on the same network system and customize our router/firewall as per our requirements.

# Digital Evidence and Forensic Toolkit  (DEFT)

While performing computer forensics, it is important that the software being used is able to ensure the integrity of file structures. It should also be able to analyze the system being investigated, without any alteration, deletion, or change of data.

DEFT is designed for forensics and is based on Lubuntu, which is itself based on Ubuntu.

DEFT can be downloaded from this link: [http://www.deftlinux.net/download/.](http://www.deftlinux.net/download/)

Once downloaded, we can burn the image file on CD/DVD media or create a live bootable USB media.

To use DEFT, we need to get an overview about what is included in the OS and we will do that next.

Once we boot the DEFT CD/DVD or USB media, we get the boot screen. Firstly, we need to select the language. Once done, we can choose to either run DEFT live or else we can install DEFT on our system.

In our example, we have chosen to boot DEFT live. We should be presented with the DEFT desktop after the boot process completes.

Now let's understand what different tools are available in DEFT.

In the start menu, the first submenu under DEFT, contains a list of various analysis tools:

![](img/5b989eb2-7dcc-4b2f-a010-ab2cf6105259.png)

The next submenu shows all the antimalware tools. Then we have the submenu of tools related to data recovery.

The next submenu contains a list of different hashing tools that can used to check and compare hashes of any file.

In the next submenu, we get tools for imaging. These can be used during forensics investigations for creating an image of a system disk that needs to be investigated. With the release of DEFT 7, tools for the analysis of mobile devices have also been added. These can be found under the, Mobile Forensics submenu.

The next submenu contains the network forensics tools. The next menu, OSINT, contains the open source intelligence tools.

DEFT also contains tools for password recovery which can be found in the next submenu.

Apart from these categories of tools, DEFT contains a few reporting tools, which can be useful while creating reports. DEFT uses **WINE** for executing Windows tools under Linux and the options for WINE can be found under the main menu.

We either install DEFT or use the live CD option to boot it on our system. Once booted, we go to the start menu and then we move to the DEFT menu. Here we find various tools under different categories. We can use tools for analysis, data recovery, mobile forensics, network forensics, and so on.

WINE is used in DEFT to execute Windows applications.

# Network Security Toolkit (NST)

Linux has many distributions developed mainly for the purpose of penetration testing. Among those, one is the **Network Security Toolkit **(**NST**), which was developed to provide easy access to open source network security applications at one place.

NST is based on Fedora Linux and contains tools for professionals and network administrators.

# Getting ready

NST can be downloaded from its webpage or directly from this link: [http://sourceforge.net/projects/nst/files/.](http://sourceforge.net/projects/nst/files/)

Once downloaded, we can either burn the ISO on a CD/DVD or create a live bootable USB media.

# How to do it...

Using NST for penetration testing becomes easy when we have an idea about how to use the OS and also what tools are included in the OS.

1.  To use NST, the first step is to boot the system with NST. We have the option to either boot using the live option, or directly install NST on the system. In our example, we have chosen the live boot option. You can choose any option as per your requirement.

2.  Once booting completes, we get the default desktop of NST as shown here:

![](img/900c5790-0e39-4ab8-b327-1613a4f0c7a8.png)

3.  NST comes with a web user interface, which is a kind of control panel to do anything with NST. However, this can be accessed only if the existing user account has a password set. To set the password, we click on the icon Set NST System Password which is on the desktop. This will open a terminal window and give the option to create a new password:

![](img/30c98fbc-d886-40e8-8124-48b3dc4dd2a9.png)

4.  Once the password has been set, we can access the NST web user interface from any browser of our choice. To access it on the local system we can use this address: `http://127.0.0.1:9980/nstwui.`

If accessing from any other system on the local network, then use the IP address of the system running NST:

![](img/1a22cd1a-46c5-46cb-8f66-3c11324cfb5d.png)

Once we open the link, we are prompted for the username and password. Enter the details and click OK.

5.  Now we see the landing page of **NSTWUI**. On the top left, we can see the details of the system running NST. Below this we have the NST menu:

![](img/dd8e4901-6306-4899-8a44-6c9b58cb0c16.png)

We can also see information about how long the system has been running on the top right:

![](img/c457ebf5-3331-4be1-8704-dac0f83f30b8.png)

6.  NST comes with various tools and amongst those one is **bandwidthd**. This tool shows an overview of network usage and we can enable it by going to the Network | Monitors | bandwidthd UI menu :

![](img/410f8c7a-8dec-4a43-9274-6615fd3ed562.png)

7.  Once we click on Start Bandwidthd the tool will start running.
8.  Another important feature that is available is the ability to do a remote activity via SSH using the web interface. Go to the System | Control Management | Run command menu.

A window will open as shown here in the screenshot. We can run any command here:

![](img/64a0826f-1226-440d-81e8-e0fac5371a76.png)

9.  NSTWUI also allows the administrator to remotely reboot or shutdown the server from the web interface. To do so, go to the System | Control Management | Reboot menu.
10.  Click on Proceed to reboot this NST system to confirm. Else click  Exit  to cancel.
11.  In the next screen, enter the text as shown and press OK:

![](img/ac0634fb-e149-492d-a670-38fa865d6d7b.png)

# How it works...

After installing or booting NST, the first step is to set the password for the existing user account. This is done by using the option Set NST System Password.

After setting the password, we access NST through the web user interface by accessing the IP address of the system through any browser.

After logging in to NSTWUI, we get a list of various tools related to network security.

We can explore a few tools such as  bandwidthd  and SSH.

# Security Onion

**Security Onion** is a free and open source distribution of Linux. It is useful for intrusion detection, enterprise-level security monitoring, and log management. Security Onion comes with a suite of tools preinstalled, such as Snort, Suricata, Kibana, OSSEC, and many more.

# Getting ready

Security Onion can be installed using the ISO image of Security Onion, the link for which is available on its official website. Another way to install Security Onion is to first install a standard Ubuntu 16.04 ISO image and then add the PPA and packages of Security Onion.

To download the ISO image of Security Onion, visit this link:

[https://github.com/Security-Onion-Solutions/security-onion/blob/master/Verify_ISO.md.](https://github.com/Security-Onion-Solutions/security-onion/blob/master/Verify_ISO.md)

# How to do it...

In this section, we will see how to install Security Onion using the ISO image. After the installation, we will configure it for further use:

1.  To start the installation, we boot our system using the ISO image. We will be presented with the following screen, where we select the first option to boot Security Onion:

![](img/a9f6afa7-91bb-44f7-b760-b67b9e3a2698.png)

2.  After the booting completes, the desktop appears. On the desktop, we can see the icon for Install Security Onion 16.04:

![](img/ff238874-77d8-4b3a-b846-6a6aabf17323.png)

3.  We click on the icon and the installation starts. The first screen will ask us to select the installation type. Choose any option as per the requirements, or else proceed with the default selection:

![](img/d69b2d07-6383-4d8e-a6c5-95ac99487c57.png)

8.  Once we click on Install Now, the installation process starts. This will take some time to complete.
9.  Once the initial installation finishes, it will prompt us to restart the system. Choose Restart Now to finish the installation.
10.  When the system reboots, we are presented with the boot menu. Select the default option and press *Enter* to boot Security Onion.
11.  After rebooting, we are presented with the login screen. Enter the username and password that was configured during the installation process.
12.  After getting logged in we can see a Setup icon on the screen. We will use this to complete the setup of security tools provided in Security Onion:

![](img/21ccc688-9dcb-4700-a460-6bccdebeeccc.png)

13.  We will be prompted to enter the password of the administrative account.
14.  After entering the password, in the next screen we are shown the list of services that will be configured. Press Yes, Continue! to proceed further:

![](img/44c7969d-d461-4d4c-85df-01f589efdcca.png)

15.  The setup will ask whether we wish to configure the interface now or later. Press Yes to configure the interfaces now:

![](img/a24dbf30-23a1-4207-92f6-7041b383793c.png)

16.  Setup will detect the interface present in the system and configure it. If there is more than one interface we can choose the interface to configure:

![](img/d4fb23b1-723d-4845-b0ef-05d213ee1142.png)

17.  Select Static or DHCP option as per requirement:

![](img/1731e555-fcb2-4656-a1b2-65b2504a1f37.png)

18.  Once done, click Yes, Make Changes to proceed further.

19.  Setup will ask us to restart the system. Restart to proceed with the setup.
20.  After the system restarts, click the Setup icon again to proceed with the setup. We will be asked if we want to reconfigure the interface or skip. Click  Yes, skip network configuration  to proceed:

![](img/04cbe2c3-642e-46ea-9ca3-7df4c03588f7.png)

21.  In the next step, setup will ask whether we are setting up the system for Evaluation Mode or Production Mode. At present we will choose Evaluation Mode and click OK:

![](img/a9bec8c6-c499-4d2d-963c-3c2b03f2363d.png)

22.  Now we will create a user account to be used by the services that the setup will configure. Enter the username in the window shown here:

![](img/9a17b5bf-1893-4271-8407-8c4c81099f38.png)

23.  Next, configure the password for the new user created:

![](img/67fd8235-1d42-4ba3-aa65-8df2192417fc.png)

24.  In the next screen, click Yes proceed with the changes.

25.  When the setup completes, we see the following window:

![](img/ac221e04-9eb1-4727-9bba-f8ce9240fbca.png)

26.  The setup also displays information, as shown here, for further use of the services:

![](img/0ceeda25-de38-4c37-850a-b43bc2bf4c7a.png)

27.  Setup also displays information about the location of rules being used by the services:

![](img/3ce64e25-a1cd-4ff9-82fa-44b80e71d89d.png)

28.  To start using the services configured by Security Onion during the setup, open the browser and visit [https://localhost](https://localhost). Accept the security warning regarding the SSL certificate and proceed further. We are presented with a webpage, as shown here:

![](img/2e66d8ef-189e-408e-9289-d2e2718004da.png)

Using this page, we can start accessing the services included in the Security Onion tool suite.

# How it works...

Security Onion comes with a suite of security tools. We install Security Onion on our system and then we set up all the tools in the suite. Once the setup is complete, we can start using the tools with the user account configured during the setup.

# Tails OS

**Tails** is a Linux distribution that runs as a live operating system and can be run on any system from a USB stick or a DVD.

It aims to preserve the privacy and anonymity of the user by helping the user use the internet anonymously and avoid restriction almost anywhere the user goes. Tails also helps the user perform activities without leaving any trace unless they ask it to explicitly.

# Getting ready

Tails is a free software built on Debian, and the ISO image can be downloaded from its website:  [https://tails.boum.org/index.en.html](https://tails.boum.org/index.en.html):

![](img/6b16f25e-3758-4a4d-bcbf-33c21434293f.png)

Use the button on the right to download the ISO of Tails.

# How to do it...

Tails is a complete operating system that can be used as a live OS from a USB stick or a DVD.

To use Tails, we boot our system using the live DVD or USB of Tails and when we boot our system using Tails, we get the following screen:

![](img/f8ee8ee9-2602-48e1-af5d-3afd24972a41.png)

Click Start Tails to start using the Tails operating system. After booting completes, we are presented with the desktop. Clicking on the Applications menu, we can see different categories of tools present, just like any other Debian-based OS. However, there is one category of Tails  in the menu:

![](img/018129b0-6e76-4d96-b00c-5678799341b4.png)

The Tails submenu contains the tools that help  it maintain privacy and anonymity. Tails contains the **Tor** browser, which helps in maintaining the privacy online. When we click on the Tor browser, it starts running in the background. We get a notification once Tor is ready:

![](img/b2f9795c-9f4d-485c-97bf-2b44e5c15ccb.png)

When we use Tails on any system, it does not alter or depend on the operating system currently running on the system. Tails has been configured to not use the hard disk of the system on which it is running, unless the user explicitly wants to save any data on the hard disk. Tails also comes with a set of tools to protect data, using strong encryption. This includes LUKS for encrypting USB sticks or external hard disks. Tails uses HTTPS Everywhere, to encrypt all communications to any major website, automatically.

We can install Tails on a USB stick, by using Tails Installer from the Application menu:

![](img/eafa331d-68df-461a-93f7-122be8a40866.png)

Tails can be used as a live OS and it comes with preinstalled applications that have been configured keeping security in mind. Once the user boots their computer using Tails, they can use any of these tools to maintain privacy and anonymity.

# Qubes OS

There are many Linux distributions available in the present time, and this includes several niche distributions. **Qubes** is one such Linux distribution which focuses on security. As its tagline says, it is a "*reasonably secure operating system*."

When a user is using any other operating system, if they unknowingly run a malware, which may have come through email attachments, it can affect everything on the system depending on the severity of the malware.

However, Qubes has been built with a different approach called *security by compartmentalization.* Its uses Qubes (virtual instances) to isolate programs run by the user.

# Getting ready

To use Qubes, we can download its ISO image from its official website:

[https://www.qubes-os.org/downloads/.](https://www.qubes-os.org/downloads/)

To install Qubes on a computer, it is recommended you have a minimum configuration as mentioned here:

*   64-bit Intel or AMD processor (x86_64 also known as x64 aka AMD64)
*   4 GB RAM
*   32 GB disk space

Qubes can also be installed on systems not meeting these recommended configuration, however the performance may be slow.

# How to do it...

Once we are done with downloading the ISO image of Qubes, we can start with the installation on a system meeting the recommended configuration.

1.  To install Qubes, we boot our system using the ISO image downloaded earlier.
2.  Once the ISO boots, it will present a menu. Select Test this media & install Qubes and press *Enter* to start the installation.
3.  The next screen will ask us to select the language. We will select English and click on Continue:

![](img/a5fdba78-d213-4445-9a52-50a1bbd70188.png)

4.  In the next window, click on INSTALLATION DESTINATION:

![](img/d1152243-6658-4e2e-a0bc-1113796eae03.png)

5.  This will open another window. Scroll down and uncheck the Encrypt my data option and click Done at the top left:

![](img/1b8bb145-191c-4116-81f1-51278615921c.png)

6.  Now, we come back to the previous window. Click on Begin Installation to proceed further:

![](img/b6460a13-4333-46fc-b7e2-acacd24b8aa0.png)

7.  As the installation continues, we get the option for USER SETTINGS. Here we can set the password for the root account and also create a new user for using Qubes, by clicking USER CREATION:

![](img/fd9035e4-4cd6-423d-905c-9c42a82622dc.png)

8.  When we click on USER CREATION, we get a new window to set a username and password for the new user. Once done, the installation continues:

![](img/efa58a3c-90f7-48cb-b25b-5926e63a2e98.png)

9.  Once the installation completes, we need to reboot the system to start using Qubes. Click on Reboot, as shown here, to use Qubes:

![](img/8f593f27-66cc-4404-90c6-44afef81de0c.png)

10.  When the system reboots, we are presented with the following screen, which shows an icon on QUBES OS. Click on QUBES OS to set up for use:

![](img/53169241-1e64-4d61-bdc0-ad932fe5d5a8.png)

11.  In the next window, select the options as per your requirements, and click on Done:

![](img/154e380b-4514-46a9-82c8-6aa34f7d1dd5.png)

12.  Once we click on Done, Qubes setup will start configuring the system using a default template, as shown here:

![](img/42d5c790-255d-47fa-8786-94e83dd3bc52.png)

13.  Once the template configuration completes, we get the following screen. Click on FINISH CONFIGURATION to proceed further:

![](img/53351e78-fd9a-4d3b-bea9-2602a895afa0.png)

14.  Now we can log in to Qubes using the user account created during the installation process:

![](img/edf12150-83ae-40dd-becb-3f602fdae476.png)

15.  Once logged in, we get the Qubes VM Manager window, which shows a list of configured Qubes. We can start any existing Qube from the list or create a new Qube as per our requirement using this VM Manager:

![](img/78cce552-fede-4e38-864c-2fcd0cd5bab9.png)

16.  Once we have created different VMs (also called **Qubes**) we can start using them. Any application accessed inside one Qube runs separately from another Qube, thus providing isolation.

# How it works...

Qubes OS uses **Xen hypervisor** for creating and isolating the virtual machines. We can work on these virtual machines to access the same application in isolation.

We can use two different instances of the same browser side by side, and they may be running on different security domains. If we visit a website using both the browsers, and are logged into the website on one if them, that login session will not be used by the other browser window, as it is running on a completely different virtual machine.