# Preface

Sincerely, if someone had asked me to write a book a few years ago, I would have certainly answered that it was impossible for several personal and professional reasons. There have been many events since I taught my first course at Sun Microsystems at the beginning of 2001 (at that time, I worked on Sun Solaris 7). Nowadays, I am thankful to keep learning more about this outstanding operating system from many excellent professionals around the world who could have written this book.

I have to confess that I am a big fan of Oracle Solaris, and my practical experience of so many years has shown me that it is still the best operating system in the world and, for a while, it has also been incomparable. When anyone talks about performance, security, consistency, features, and usability, it always takes me to same point: Oracle Solaris.

It is likely that there will be people who disagree and I can try to explain my point of view, attacking other good operating systems such as Linux, AIX, HP-UX, and even Windows, but it will not be very effective or polite. Instead, I think it is more suitable to teach you the advanced features of Oracle Solaris and its use cases, and you can make your own conclusions.

Oracle has invested a lot of money in Oracle Solaris that has been improved a lot because many good and advanced features have been introduced since then and it is at this point that this book begins.

*Oracle Solaris 11 Advanced Administration Cookbook* aims to show and explain dedicated procedures about how to execute daily tasks on the Oracle Solaris 11 system on a step-by-step basis, where every single command is tested and its output is shown. Additionally, this book will be committed to reviewing a few key topics from Oracle Solaris 11 intermediate administration, and all the concepts from basic and advanced administration will be introduced according to need in order to help the reader understand obscure points.

While I was writing this book, I learned a lot and tested different scenarios and ways to bring you only the essential concepts and procedures, given that all commands and outputs came from my own lab. By the way, the entire book was written using an x64 machine because most people have difficulties in accessing SPARC-based systems.

Finally, I hope you have a great time reading this book as well, just like I had while I was writing it. I hope you enjoy it!

# What this book covers

[Chapter 1](part0015_split_000.html#page "Chapter 1. IPS and Boot Environments"), *IPS and Boot Environments*, covers all aspects from IPS and boot environment administration, where it is explained how to administer packages, configure IP repositories, and create your own packages. Additionally, this chapter also discuss BE administration and its associated operations.

[Chapter 2](part0038_split_000.html#page "Chapter 2. ZFS"), *ZFS*, explains the outstanding world of ZFS. This chapter focuses on ZFS pool and filesystem administration as well as how to handle snapshots, clones, and backups. Moreover, it will include a discussion on using ZFS shadow, ZFS sharing with SMB shares, and logs. Finally, it will provide a good explanation on how to mirror the root pool and how to play with ZFS swap.

[Chapter 3](part0054_split_000.html#page "Chapter 3. Networking"), *Networking*, takes you through the reactive network configuration, link aggregation setup, and IPMP administration. Other complex topics such as network bridging, link protection, and Integrated Load Balancer will be explained and fully demonstrated.

[Chapter 4](part0063_split_000.html#page "Chapter 4. Zones"), *Zones*, shows us how to administer a virtual network and deploy the resource manager on a zone. Complementary and interesting topics such as flow control and zone migration will be also discussed.

[Chapter 5](part0069_split_000.html#page "Chapter 5. Playing with Oracle Solaris 11 Services"), *Playing with Oracle Solaris 11 Services*, helps you to understand all SMF operations and to review the basic concepts about how to administer a service. Furthermore, this chapter explains and shows you step-by-step recipes to create SMF services, handle manifests and profiles, administer network services, and troubleshoot Oracle Solaris 11 services.

[Chapter 6](part0076_split_000.html#page "Chapter 6. Configuring and Using an Automated Installer (AI) Server"), *Configuring and Using an Automated Installer (AI) Server*, takes you through an end-to-end Automated Installer (AI) configuration recipe and provides all the information about how to install an x86 client from an AI server.

[Chapter 7](part0079_split_000.html#page "Chapter 7. Configuring and Administering RBAC and Least Privileges"), *Configuring and Administering RBAC and Least Privileges*, explains how to configure and administer RBAC and least privileges. The focus is to keep the Oracle Solaris installation safe.

[Chapter 8](part0083_split_000.html#page "Chapter 8. Administering and Monitoring Processes"), *Administering and Monitoring Processes*, provides an interesting approach on how to handle processes and their respective priorities.

[Chapter 9](part0088_split_000.html#page "Chapter 9. Configuring the Syslog and Monitoring Performance"), *Configuring the Syslog and Monitoring Performance*, provides step-by-step recipes to configure the Syslog service and offers a nice introduction on performance monitoring in Oracle Solaris 11.

# What you need for this book

I am sure you know how to install Oracle Solaris 11 very well. Nevertheless, it is pertinent to show you how to configure a simple environment to execute each procedure of this book. A well-done environment will help us to draw every concept and understanding from this book by executing all the commands, examples, and procedures. In the end, you should remember that this a practical book!

To follow this recipe, it is necessary to have a physical machine with at least 8 GB RAM and about 80 GB of free space on the hard disk. Additionally, this host should be running operating system that is compatible with and supported by the VMware or VirtualBox hypervisor software, including processors such as Intel or AMD, which support hardware virtualization. You are also required to have a working Solaris 11 that will be installed and configured as a virtual machine (VMware or VirtualBox).

To get your environment ready, you have to execute the following steps:

1.  First, you should download Oracle Solaris 11 from the Oracle website ([http://www.oracle.com/technetwork/server-storage/solaris11/downloads/index.html](http://www.oracle.com/technetwork/server-storage/solaris11/downloads/index.html)). It is appropriate to pick the *Oracle Solaris 11 Live Media for x86* method because it is easier than the *Text Installer* method, and it allows us to bring up the Oracle Solaris 11 from DVD before starting the installation itself. For example, if we are using a physical machine (not a virtual one as is usually used), it provides us with a utility named *Device Driver Utility* that checks whether Oracle Solaris 11 has every driver software for the physical hardware. Nonetheless, if we want to install Oracle Solaris 11 on a SPARC machine, then the *Text Installer* method should be chosen.
2.  We should download all the pieces from the Oracle Solaris repository images and concatenate them into a single file (`# cat part1 part2 part3 … > sol-11-repo-full.iso`). This final image will be used in [Chapter 1](part0015_split_000.html#page "Chapter 1. IPS and Boot Environments"), *IPS and Boot Environments*, when we talk about how to configure an IPS local repository.
3.  Later in this book, how to configure *Oracle Solaris 11 Automatic Installation* will be explained, so it is recommended that you take out time to download *Oracle Solaris 11 Automated* Installer image for DVD for x86 from [http://www.oracle.com/technetwork/server-storage/solaris11/downloads/install-2245079.html](http://www.oracle.com/technetwork/server-storage/solaris11/downloads/install-2245079.html).
4.  It is necessary to get some virtualization tool to create virtual machines for Oracle Solaris 11 installation, such as VMware Workstation ([http://www.vmware.com/products/workstation/workstation-evaluation](http://www.vmware.com/products/workstation/workstation-evaluation)) or Oracle VirtualBox that can be downloaded from [https://www.virtualbox.org/](https://www.virtualbox.org/).
5.  Unfortunately, it is not possible to give details about how to install Oracle Solaris 11 in this book. However, there is a good article that explains and shows a step-by-step procedure at [http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris-install-borges-1989211.html](http://www.oracle.com/technetwork/articles/servers-storage-admin/solaris-install-borges-1989211.html) from Oracle Technical Network (OTN).
6.  It is helpful to remember that during the LiveCD GUI installation method, the root user is always configured as a role, and this action is different from the *Text Installer* method that allows us to choose whether the root user will or will not be configured as a role.
7.  Just in case the reader does not remember how to change the root role back to work as a user again, we can execute the following command:

    ```
    root@solaris11:/#  su - root
    root@solaris11:/#  rolemod  -K  type=normal  root

    ```

    Afterwards, it is necessary to log out and log on to the system again for using the root user.

8.  Finally, we recommend you verify that Oracle Solaris 11 is working well by running the following commands:

    ```
    root@solaris11:/# svcs  network/physical
    STATE          STIME    FMRI
    online         13:43:02 svc:/network/physical:upgrade
    online         13:43:18 svc:/network/physical:default

    root@solaris11:~# ipadm show-addr
    ADDROBJ       TYPE       STATE          ADDR
    lo0/v4      static        ok       127.0.0.1/8
    net0/v4     dhcp          ok       192.168.1.111/24
    lo0/v6      static        ok       ::1/128
    net0/v6     addrconf      ok       fe80::a00:27ff:fe56:85b8/10
    ```

We have finished setting up our environment. Thus, it is time to learn!

# Who this book is for

If you are an IT professional, IT analyst, or anyone with a basic knowledge of Oracle Solaris 11 intermediate administration and you wish to learn and deploy advanced features from Oracle Solaris 11, this book is for you. Furthermore, this is a practical book that requires a system running Oracle Solaris 11 virtual machines.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "The command used to detect the `nmap` package corruption detected the exact problem."

Any command-line input or output is written as follows:

```
root@solaris11:~# beadm list
BE                Active Mountpoint Space    Policy  Created  
-------------     ------ ---------- -------  ------  ---------- 
solaris           NR     /          4.99G    static  2013-10-05 20:44
solaris-backup-1  -      -          163.0K   static  2013-10-10 19:57
solaris-backup-b  -      -          173.0K   static  2013-10-12 22:47
```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "To launch the Package Manager interface, go to **System** | **Administrator** | **Package Manager**."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or may have disliked. Reader feedback is important for us to develop titles that you really get the most out of.

To send us general feedback, simply send an e-mail to `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, and mention the book title via the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide on [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **errata submission form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded on our website, or added to any list of existing errata, under the Errata section of that title. Any existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.