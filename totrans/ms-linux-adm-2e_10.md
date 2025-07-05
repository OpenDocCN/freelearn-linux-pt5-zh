# 10

# Disaster Recovery, Diagnostics, and Troubleshooting

In this chapter, you will learn how to do a system backup and restore in a disaster recovery scenario, and how to **diagnose** and **troubleshoot** a common array of problems. These are skills that each Linux system administrator needs to have if they wish to be prepared for worst-case scenarios such as power outages, theft, or hardware failure. The world’s IT backbone runs on Linux and we need to be prepared for anything that life throws at us.

In this chapter, we’re going to cover the following main topics:

*   Planning for disaster recovery
*   Backing up and restoring the system
*   Introducing common Linux diagnostic tools for troubleshooting

# Technical requirements

No special technical requirements are needed for this chapter, just a working installation of Linux on your system or even two different working systems on your local network for some of the examples used. Ubuntu and Fedora are equally suitable for this chapter’s exercises, but in this chapter, we’ll be using Ubuntu 22.04.2 LTS Server and Desktop editions.

# Planning for disaster recovery

Managing risks is an important asset for every business or individual. The responsibility of this is tremendous for everyone involved in system administration. For all businesses, managing risks should be part of a wider **risk management strategy**. There are various types of risks in IT, starting from natural hazards directly impacting data centers or business locations, all the way up to cyber security threats. IT’s footprint inside a company has exponentially grown in the last decade. Nowadays, there is no activity that does not involve some sort of IT operations being behind it, be it inside small businesses, big corporations, government agencies, or the health and education public sectors, just to give a few examples. Each activity is unique in its own way, so it needs a specific type of assessment. Unfortunately, with regard to the information security field, risk management has largely evolved into a one-size-fits-all practice, based on checklists that should be implemented by IT management. Let’s begin with a brief introduction to risk management before moving on to learning how we can assess risk.

## A brief introduction to risk management

What is **risk management**? In a nutshell, it is comprised of specific operations that are set to mitigate any possible threat that could impact the overall continuity of a business. The risk management process is crucial for every IT department.

Risk management frameworks initially arose in the United States due to the **Federal Information Systems Modernization Act** (**FISMA**) laws, which started in 2002\. This was the time when the United States **National Institute of Standards and Technology** (**NIST**) began to create new standards and methods for cyber security assessments among all US government agencies. Therefore, security certifications and compliances are of utmost importance for every Linux distribution provider that sees itself as a worthy competitor in the corporate and governmental space. Similar to the US certification bodies discussed previously, there are other agencies in the UK and Russia that develop specific security certifications. In this respect, all major Linux distributions from Red Hat, SUSE, and Canonical have certifications from NIST, the UK’s **National Cyber Security Centre** (**NCSC**), or Russia’s **Federal Service for Technic and Export** **Control** (**FSTEC**).

The risk management framework, according to NIST SP 800-37r2 (see the official NIST website at [https://csrc.nist.gov/publications/detail/sp/800-37/rev-2/final](https://csrc.nist.gov/publications/detail/sp/800-37/rev-2/final)), has seven steps, starting with preparing for the framework’s execution, up to monitoring the organization’s systems on a daily basis. We will not discuss those steps in detail; instead, we will provide a link at the end of this chapter for NIST’s official documentation. In a nutshell, the risk management framework is comprised of several important branches, such as the following:

*   **Inventory**: A thorough inventory of all available systems that are on-premises, and a list of all software solutions
*   **System categorization**: Assesses the impact level for each data type that’s used with regard to availability, integrity, and confidentiality
*   **Security control**: Subject to detailed procedures with regard to hundreds of computer systems’ security – a compendium of NIST security controls can be found under SP800-53r4 (the following is a link to the official NIST website: [https://csrc.nist.gov/publications/detail/sp/800-53/rev-4/final](https://csrc.nist.gov/publications/detail/sp/800-53/rev-4/final))
*   **Risk assessment**: A series of steps that cover threat source identification, vulnerability identification, impact determination, information sharing, risk monitoring, and periodical updates
*   **System security plan**: A report based on every security control and how future actions are assessed, including their implementation and effectiveness
*   **Certification, accreditation, assessment, and authorization**: The process of reviewing security assessments and highlighting security issues and effective resolutions that are detailed in a future plan of action
*   **Plan of action**: A tool that’s used to track security weaknesses and apply the correct response procedures

There are many types of risks when it comes to information technology, including hardware failure, software errors, spam and viruses, human error, and natural disasters (fires, floods, earthquakes, hurricanes, and so on). There are also risks of a more criminal nature, including security breaches, employee dishonesty, corporate espionage, or anything else that could be considered a cybercrime. These risks can be addressed by implementing an appropriate risk management strategy. As a basis, such a strategy should have five distinct steps:

1.  **Identifying risk**: Identifying possible threats and vulnerabilities that could impact your ongoing IT operations.
2.  **Analyzing risk**: Deciding how big or small it is, based on thorough studies.
3.  **Evaluating risk**: Evaluating the impact that it could have on your operations; the immediate action is to respond to the risk based on the impact it has. This calls for real actions to be performed at every level of your operations.
4.  **Responding to risk**: Activating your **disaster recovery plans** (**DRPs**), combined with strategies for prevention and mitigation.
5.  **Monitoring and reviewing risk**: Triggering a drastic monitoring and reviewing strategy will ensure that all the IT teams know how to respond to the risk and have the tools and abilities to isolate it and enforce the company’s infrastructure.

Risk assessment is extremely important to any business and should be taken very seriously by IT management. Now that we’ve tackled some concepts of risk management, it is time to explain what it really is.

## Risk calculation

**Risk assessment**, also known as **risk calculation** or **risk analysis**, refers to the action of finding and calculating solutions to possible threats and vulnerabilities. The following are some basic terms you should know when you talk about risk impact:

*   The **annual loss expectancy** (**ALE**) defines the loss that’s expected in 1 year.
*   The **single loss expectancy** (**SLE**) represents how much loss is expected at any given time.
*   The **annual rate of occurrence** (**ARO**) is the likeliness of a risky event occurring within 1 year.
*   The **risk calculation formula** is *SLE x ARO = ALE*. There is a monetary value that each element of the formula will provide, so the final result is also expressed as a monetary value. This is a formula that is useful to know.
*   The **mean time between failures** (**MTBF**) is used to measure the time between anticipated and reparable failures.
*   The **mean time to failure** (**MTTF**) is the average time the system can operate before experiencing an irreparable failure.
*   The **mean time to restore** (**MTTR**) measures the time needed to repair an affected system.
*   The **recovery time objective** (**RTO**) represents the maximum time that’s allocated for downtime.
*   The **recovery point objective** (**RPO**) defines the time when a system needs to be restored.

Knowing those terms will help you understand risk assessments so that you can perform a well-documented assessment if or when needed. Risk assessment is based on two major types of actions (or better said, strategies):

*   **Proactive actions**:
    *   **Risk avoidance**: Based on risk identification and finding a quick solution to avoid its occurrence
    *   **Risk mitigation**: Based on actions taken to reduce the occurrence of a possible risk
    *   **Risk transference**: Transferring the risk’s possible outcome with an external entity
    *   **Risk deterrence**: Based on specific systems and policies that should discourage any attacker from exploiting the system
*   **Non-active actions**:
    *   **Risk acceptance**: Accepting the risk if the other proactive actions could exceed the cost of the harm that’s done by the risk

The strategies described here can be applied to the risk associated with generic, on-premises computing, but nowadays, cloud computing is slowly and surely taking over the world. So, how could these risk strategies apply to cloud computing? In cloud computing, you use the infrastructure of a third party, but with your own data. Even though we will start discussing Linux in the cloud in [*Chapter 14*](B19682_14.xhtml#_idTextAnchor299), *Short Introduction to* *Computing*, there are some concepts that we will introduce now. As we mentioned earlier, the cloud is taking the infrastructure operations from your on-premises environment to a larger player, such as Amazon, Microsoft, or Google. This could generally be seen as outsourcing. This means that some risks that were a threat when you were running services on-premises are now transferred to third parties.

There are three major cloud paradigms that are now buzzwords all over technology media:

*   **Software as a service** (**SaaS**): This is a software solution for companies looking to reduce IT costs and rely on software subscriptions. Some examples of SaaS solutions are **Slack**, **Microsoft 365**, **Google Apps**, and **Dropbox**, among others.
*   **Platform as a service** (**PaaS**): The way you get software applications to your clients using another’s infrastructure, runtimes, and dependencies is also known as an application platform. This can be on a public cloud, on a private cloud, or on a hybrid solution. Some examples of PaaS are **Microsoft Azure**, **AWS Lambda**, **Google App Engine**, **SAP Cloud Platform**, **Heroku**, and **Red** **Hat OpenShift**.
*   **Infrastructure as a service** (**IaaS**): These are services that are run online and provide high-level **application programming interfaces** (**APIs**). A notable example is **OpenStack**.

Details about all these technologies will be provided in [*Chapter 14*](B19682_14.xhtml#_idTextAnchor299), *Short Introduction to* *Computing*, but for this chapter’s purpose, we have provided enough information. Major risks regarding cloud computing are concerned with data integration and compatibility. Those are among the risks that you must still overcome since most of the other risks are no longer your concern as they are transferred to the third party managing the infrastructure.

Risk calculation can be managed in different ways, depending on the IT scenario a company uses. When you’re using the on-premises scenario and you’re managing all the components in-house, risk assessments become quite challenging. When you’re using the IaaS, PaaS, and SaaS scenarios, risk assessment becomes less challenging as responsibilities are gradually transferred to an external entity.

Risk assessment should always be taken seriously by any individual concerned with the safety of their network and systems and by any IT manager. This is when a DRP comes into action. The foundation of a good DRP and strategy is having an effective risk assessment.

## Designing a DRP

A DRP is structured around the steps that should be taken when an incident occurs. In most cases, the DRP is part of a **business continuity plan**. This determines how a company should continue to operate based on a functioning infrastructure.

Every DRP needs to start from an accurate **hardware inventory**, followed by a **software applications inventory** and a separate **data inventory**. The most important part of this is the strategy that’s designed to back up all the information that’s used.

In terms of the hardware that’s used, there must be a clear policy for standardized hardware. This will ensure that faulty hardware can easily be replaced. This kind of policy ensures that everything works and is optimized. Standardized hardware surely has good driver support, and this is very important in the Linux world. Nevertheless, using standardized hardware will tremendously limit practices such as **bring your own device** (**BYOD**), since employees only need to use the hardware provided by their employer. Using standardized hardware comes with using specific software applications that have been set up and configured by the company’s IT department, with limited input available from the user.

The IT department’s responsibility is huge, and it plays an important role in designing the **IT recovery strategies** as part of a DRP. Key tolerances for downtime and loss of data should be defined based on the minimal acceptable RPO and RTO.

Deciding on the **roles** regarding who is responsible for what is another key step of a good DRP. This way, the response time for implementing the plan will be dramatically reduced and everyone will know their own responsibilities in case any risks arise. In this case, having a good **communication strategy** is critical. Enforcing clear procedures for every level of the organizational pyramid will provide clear communication, centralized decisions, and a succession plan for backup personnel.

DRPs need to be thoroughly tested at least twice a year to prove their efficiency. Unplanned downtime and outages can negatively impact a business, both on-premises and in any multi-cloud environment. Being prepared for worst-case scenarios is important. Therefore, in the following sections, we will show you some of the best tools and practices for backing up and restoring a Linux system.

# Backing up and restoring the system

Disasters can occur at any time. Risk is everywhere. In this respect, backing up your system is of utmost importance and needs to be done regularly. It is always better to practice good prevention than to recover from data loss and learn this the hard way.

**Backup** and **recovery** need to be done based on a well-thought-out strategy and need to take the RTO and RPO factors into consideration. The RTO should answer basic questions such as how fast to recover lost data and how this will affect the business operations, while the RPO should answer questions such as how much of your data you can afford to lose.

There are different types and methods of backup. The following are some examples:

![Figure 10.1 – Backup methods and types](img/B19682_10_01.jpg)

Figure 10.1 – Backup methods and types

When doing a backup, keep the following rules in mind:

*   The **321 rule** means that you should always have at least three copies of your data, with two copies on two different media at separate locations and one backup always being kept off-site (at a different geographical location). This is also known as the **rule of three**; it can be adapted to anything, such as 312, 322, 311, or 323.
*   **Backup checking** is extremely relevant and is overlooked most of the time. It checks the data’s integrity and usefulness.
*   **Clear and documented backup strategy and procedures** are beneficial to everyone in the IT team who is using the same practices.

In the next section, we will look at some well-known tools for full Linux system backups, starting with the ones that are integrated inside the operating system to third-party solutions that are equally suited for both home and enterprise use.

## Disk cloning solutions

A good option for a backup is to clone the entire hard drive or several partitions that hold sensitive data. Linux offers a plethora of versatile tools for this job. Among those are the `dd` command, the `ddrescue` command, and the **Relax-and-Recover** (**ReaR**) software tool. Let’s look at these in detail in the following sections.

### The dd command

One of the most well-known disk backup commands is the `dd` command. We discussed this previously in [*Chapter 6*](B19682_06.xhtml#_idTextAnchor124), *Working with Disks and Filesystems*. Let’s recap how it is used in a backup and restore scenario. The `dd` command is used to copy block by block, regardless of the filesystem type, from a source to a destination.

Let’s learn how to clone an entire disk. We have a virtual machine on our system that has a 20 GB drive that we want to back up on a 128 GB USB pen drive. The procedures we are going to show you will work on bare metal too.

First, we will run the `sudo fdisk -l` command to verify that the disk sizes are correct. The output will show us information about both our local drive and the USB pen drive, among other information, depending on your system.

Now that we know what the sizes are and that the source can fit into the destination, we will proceed to cloning the entire virtual disk. We will clone the source disk, `/dev/vda`, to the destination disk, `/dev/sda` (the operation could take a while):

![Figure 10.2 – Using dd to clone an entire hard drive](img/B19682_10_02.jpg)

Figure 10.2 – Using dd to clone an entire hard drive

The options shown in the preceding command are as follows:

*   `if=/dev/vda` represents the input file, which, in our case, is the source hard drive
*   `of=/dev/sda` represents the output file, which is the destination USB drive
*   `conv=noerror` represents the instruction that allows the command to continue ignoring errors
*   `sync` represents the instruction to fill the input error blocks with zeros so that the data offset will always be synced
*   `status=progress` shows statistics about the transfer process

Please keep in mind that this operation could take a while to finish. On our system, it took 200 minutes to complete. We took the preceding screenshot while the operation was at the beginning. In the following section, we will show you how to use `ddrescue`.

### The ddrescue command

The `ddrescue` command is yet another tool you can use to clone your disk. This tool copies from one device or file to another one, trying to copy only the good and healthy parts the first time. If your disk is failing, you might want to use `ddrescue` twice since, the first time, it will copy only the good sectors and map the errors to a destination file. The second time, it will copy only the bad sectors, so it is better to add an option for several read attempts just to be sure.

On Ubuntu, the `ddrescue` utility is not installed by default. To install it, use the following `apt` command:

```
sudo apt install gddrescue
```

We will use `ddrescue` on the same system we used previously and clone the same drive. The command for this is as follows:

```
sudo ddrescue -n /dev/vda /dev/sda rescue.map --force
```

The output is as follows:

![Figure 10.3 – Using ddrescue to clone the hard drive](img/B19682_10_03.jpg)

Figure 10.3 – Using ddrescue to clone the hard drive

We used the `ddrescue` command with the `--force` option to make sure that everything on the destination will be overwritten. This operation is time-consuming too, so be prepared for a lengthy wait. In our case, it took almost 1 hour to finish. Next, we will show you how to use another useful tool, the ReaR utility.

### Using ReaR

ReaR is a powerful disaster recovery and system migration tool written in Bash. It is used by enterprise-ready distributions such as RHEL and SLES, and can also be installed on Ubuntu. It was designed to be easy to use and set up. It is integrated with the local bootloader, with the `cron` scheduler, or monitoring tools such as **Nagios**. For more details on this tool, visit the official website at [http://relax-and-recover.org/about/](http://relax-and-recover.org/about/).

To install it on Ubuntu, use the following command:

```
sudo apt install rear
```

Once the packages have been installed, you will need to know the location of the main configuration file, which is `/etc/rear/local.conf`, and all the configuration options should be written inside it. ReaR makes ISO files by default, but it also supports Samba (CIFS), USB, and NFS as backup destinations. Next, we will show you how to use ReaR to back up to a local NFS server.

#### Backing up to a local NFS server using ReaR

As an example, we will show you how to back up to an NFS server. As specified in the *Technical requirements* section, you would need to have at least two systems available on your network for this exercise: an NFS server set up on one of the machines (as a backup server) and a second system as the production machine to be backed up. ReaR should be installed on both of them. Perform the following steps:

1.  First, we must configure the NFS server accordingly (the operation is covered in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*). For extensive information on setting up an NFS server, please refer to [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276); here, we only cover it briefly. The configuration file for NFS is `/etc/exports` and it stores information about the share’s location. Before you add any new information about the ReaR backup share’s location, add a new directory. We will consider the `/home/export/` directory for our NFS setup. Inside that directory, we will create a new one for our ReaR backups. The command to create the new directory is as follows:

    ```
    root, ReaR will not have permission to write the backup to this location. Change the ownership using the following command:

    ```
    /etc/exports file with your favorite editor and add a new line for the backup directory. The line should look like the following:

    ```
    /home/export/rear 192.168.124.0/24(rw,sync,no_subtree_check)
    ```

    Use your local network’s IP range, not the one we used.
    ```

    ```

2.  Once the new line has been introduced, restart the NFS service and run the `exportfs` command using the `-``s` option:

    ```
    /etc/rear/local.conf file and add the lines shown in the following output. Use your own system’s IP address, not the one we used. The code should look like the following:

    ```
    OUTPUT=ISO
    OUTPUT_URL=nfs://192.168.124.112/home/export/rear
    BACKUP=NETFS
    OUTPUT: The bootable image type, which, in our case, is ISO
    ```

    ```

3.  `OUTPUT_URL`: The backup target, which can represent NFS, CIFS, FTP, RSYNC, or file
4.  `BACKUP`: The backup method used, which, in our case, is `NETFS`, the default ReaR method
5.  `BACKUP_URL`: The backup target’s location
6.  Now, run the `mkbackup` command with the `-v` and `-``d` options:

    ```
    sudo rear -v -d mkbackup
    ```

    The output will be large, so we will not show it to you here. The command will take a significant time to finish. Once it has finished, you can check the NFS directory to view its output. The backup should be in there.

There are several files written on the NFS server. Among those, the one called `rear-neptune.iso` is the actual backup and the one that will be used in case a system restore is needed. There is also a file called `backup.tar.gz`, which contains all the files from our local machine.

Important note

The naming convention of ReaR is as follows. The name will consist of the term `rear-`, followed by the system’s hostname and the `.iso` extension. Our system’s hostname is `neptune`, which is why the backup file is called `rear-neptune.iso` in our case.

Once the backup has been written on the NFS server, you will be able to restore the system by using a USB disk or DVD with the ISO image that was written on the NFS server.

#### Backing up to USB using ReaR

There is also the option of directly backing up on the USB disk. Here are the steps to be followed:

1.  Insert a disk into the USB port and format it by using the following command:

    ```
    sudo rear format /dev/sda
    ```

    The command will take a significant time to complete. The output is as follows:

![Figure 10.4 – Formatting the USB disk with ReaR](img/B19682_10_04.jpg)

Figure 10.4 – Formatting the USB disk with ReaR

1.  Now, we need to modify the `/etc/rear/local.conf` file and adapt it so that it uses the USB as the backup destination. The new lines we will add should look as follows:

    ```
    OUTPUT=USB
    BACKUP_URL="usb:///dev/disk/by-label/REAR-000"
    ```

2.  To understand the last line of code, you can run the following command:

    ```
    /dev/disk/by-label/REAR-000 is a link to /dev/sda1 (in our case):
    ```

![Figure 10.5 – Checking the URL location from the ReaR configuration file](img/B19682_10_05.jpg)

Figure 10.5 – Checking the URL location from the ReaR configuration file

1.  To back up the system on the USB disk, run the following command:

    ```
    tar.gz files will be on the USB drive.
    ```

2.  To recover the system, you will need to boot from the USB drive and select the first option, which says `Recover "hostname"`, where `"hostname"` is the hostname of the computer you backed up.

System backup and recovery are two very important tasks that should be indispensable to any Linux system administrator. Knowing how to execute those tasks can save data, time, and money for both the company and the client. Minimal downtime and having a quick, effective response should be the most important assets on every **Chief Technology Officer’s** (**CTO’s**) table. Backup and recovery strategies should always have a strong foundation in terms of good mitigation practices. In this respect, a strong diagnostics toolset and troubleshooting knowledge will always come in handy for every system administrator. This is why, in the next section, we will show you some of the best diagnostic tools in Linux.

# Introducing common Linux diagnostic tools for troubleshooting

The openness of Linux is one of its best assets. This opened the door to an extensive number of solutions that can be used for any task at hand. Hence, many diagnostic tools are available to Linux system administrators. Depending on which part of your system you would like to diagnose, there are several tools available. Troubleshooting is essentially problem-solving based on diagnostics generated by specific tools. To reduce the number of diagnostic tools to cover, we will narrow down the issues to the following categories for this section:

*   Boot issues
*   General system issues
*   Network issues
*   Hardware issues

There are specific diagnostic tools for each of these categories. We will start by showing you some of the most widely used ones.

## Tools for troubleshooting boot issues

To understand the issues that may affect the boot process, it is important to know how the boot process works. We have not covered this in detail yet, so pay attention to everything that we will tell you.

### The boot process

All the major Linux distributions, such as Ubuntu, OpenSUSE, Debian, Fedora, and RHEL, use `systemd` as their default init system. Until GRUB2 initialization and the `systemd` startup were put in place, the Linux boot process had several more stages.

The boot order is as follows:

1.  The **Basic Input Output System** (**BIOS**) **Power-On** **Self-Test** (**POST**)
2.  GRUB2 bootloader initialization
3.  GNU/Linux kernel initialization
4.  `systemd init` system initialization

BIOS POST is a process specific to hardware initialization and testing, and it is similar for every PC, regardless of whether it is using Linux or Windows. The BIOS makes sure that every hardware component inside the PC is working properly. When the BIOS fails to start, there is usually a hardware problem or incompatibility issue. The BIOS searches for the disk’s boot record, such as the **master boot record** (**MBR**) or **GUID Partition Table** (**GPT**), and loads it into memory.

GRUB2 initialization is where Linux starts to kick in. This is the stage when the system loads the kernel into memory. It can choose between several different kernels in case there’s more than one operating system available. Once the kernel has been loaded into memory, it takes control of the boot process.

The kernel is a self-extracting archive. Once extracted, it runs into the memory and loads the `init` system, the parent of all the other processes on Linux.

The `init` system, called `systemd`, starts by mounting the filesystems and accessing all the available configuration files.

During the boot process, issues may appear. In the next section, we will discuss what to do if disaster strikes and your bootloader won’t start.

### Repairing GRUB2

If GRUB2 breaks, you will not be able to access your system. This calls for a GRUB repair. At this stage, a live bootable USB drive will save you. The following steps are an exercise that could count as an experiment. In our case, we have an Ubuntu 22.04 LTS Desktop edition live disk, and we will use it for this example. However, you can use any Linux distribution you like, not necessarily Ubuntu. The point is that you would need a live bootable USB drive with Linux on it. Here are the steps you should follow:

1.  Plug in the Ubuntu 22.04 live disk drive and boot the system.
2.  Open the BIOS, select the bootable disk as the main boot device, and restart it.
3.  Select the **Try** **Ubuntu** option.
4.  Once inside the Ubuntu instance, open Terminal, enter the `sudo fdisk -l` command, and check your disks and partitions.
5.  Select the one that GRUB2 is installed on and use the following command (use the disk names as provided by your system; don’t copy/paste our example):

    ```
    sudo mount -t ext4 /dev/sda1 /mnt
    ```

6.  Install GRUB2 using the following commands:

    ```
    sudo chroot /mnt
    grub-install /dev/sda
    grub-install –recheck /dev/sda
    update-grub
    ```

7.  Unmount the partition using the following commands:

    ```
    exit
    sudo unmount /mnt
    ```

8.  Reboot the computer.

Dealing with bootloaders is extremely sensitive. Pay attention to all the details and take care of all the commands you type in. If not, everything could go sideways. In the next section, we will show you some diagnostic tools for general system issues.

## Tools for troubleshooting general system issues

System issues can be of different types and complexities. Knowing the tools to deal with them is of utmost importance. In this section, we will cover the default tools provided by the Linux distribution. Basic troubleshooting knowledge is necessary for any Linux system administrator as issues can – and will – occur during regular operations.

What could general system issues mean? Well, basically, these are issues regarding disk space, memory usage, system load, and running processes.

### Commands for disk-related issues

Disks, be they HDDs or SSDs, are an important part of the system. They provide the necessary space for your data, files, and software of any type, including the operating system. We will not discuss hardware-related issues as this will be the subject of a future section in this chapter called *Tools for troubleshooting hardware issues*. Instead, we will cover issues related to **disk space**. The most common diagnostic tools for this are already installed on any Linux system, and they are represented by the following commands:

*   `du`: A utility that shows disk space utilization for files and directories
*   `df`: A utility that shows the disk usage for directories

The following is an example of using the `df` utility with the `-h` (human-readable) option:

```
df -h
```

If one of the disks runs out of space, it will be shown in the output. This is not an issue in our case, but the tool is still relevant for finding out which of the available disks is having issues with the free available space.

When a disk is full, or almost full, there are several fixes that can be applied. If you have to delete some of the files, we advise you to delete them from your `/home` directory. Try not to delete important system files. The following are some ideas for troubleshooting available space issues:

*   Delete the unnecessary files using the `rm` command (optionally using, with caution, the `-rf` option) or the `rmdir` command
*   Move files to an external drive (or to the cloud) using the `rsync` command
*   Find which directories use the most space in your `/``home` directory

The following is an example of using the `du` utility to find the largest directories inside the `/home` directory. We are using two pipes to transfer the output of the `du` command to the `sort` command and finally to the `head` command with the option of `5` (because we want to show the five largest directories, not all of them):

![Figure 10.6 – Finding the largest directories in your /home directory](img/B19682_10_06.jpg)

Figure 10.6 – Finding the largest directories in your /home directory

Another troubleshooting scenario is with regard to the number of inodes being used, not with the space on a disk. In this case, you can use the `df -i` command to see whether you’ve run out of inodes:

![Figure 10.7 – inode usage statistics](img/B19682_10_07.jpg)

Figure 10.7 – inode usage statistics

The output in the preceding screenshot shows basic information about the inode usage on the system. You will see the total numbers of inodes on different filesystems, how many inodes are in use (as a number and as a percentage), and how many are free.

Besides the commands shown here, which are the defaults for every Linux distribution, there are many other open source tools for disk space issues, such as **pydf**, **parted**, **sfdisk**, **iostat**, and the GUI-based **GParted** application.

In the next section, we will show you how to use commands to verify possible memory issues.

### Commands for memory usage issues

`free` and it can be accessed in any major distribution. In the following example, we will use the `-h` option for a human-readable output:

![Figure 10.8 – Using the free command in Linux](img/B19682_10_08.jpg)

Figure 10.8 – Using the free command in Linux

As shown in the preceding screenshot, using the `free` command (with the `-h` option for human-readable output) shows the following:

*   `total`: The total amount of memory
*   `used`: The used memory, which is calculated as the total memory minus the buffered, cache, and free memory
*   `free`: The free or unused memory
*   `shared`: The memory used by `tmpfs`
*   `buff/cache`: The memory used by kernel buffers and the page cache
*   `available`: The amount of memory available for new applications

This way, you can find specific issues related to higher memory usage. Constantly checking memory usage on servers is important to see whether resources are being used efficiently.

Another way to check for memory usage is to use the `top` command, as shown in the following screenshot:

![Figure 10.9 – Using the top command to check memory usage](img/B19682_10_09.jpg)

Figure 10.9 – Using the top command to check memory usage

While using the `top` command, there are several sections available on screen. The output is dynamic, in the sense that it constantly changes, showing real-time information about the processes running on the system. The `memory` section shows information about total memory used, as well as free and buffered memory. All the information is shown in megabytes by default so that it’s easier to read and understand.

Another command that shows information about memory (and other valuable system information) is `vmstat`:

![Figure 10.10 – Using vmstat with no options](img/B19682_10_10.jpg)

Figure 10.10 – Using vmstat with no options

By default, `vmstat` shows information about processes, memory, swap, disk, and CPU usage. The memory information is shown starting from the second column and contains the following details:

*   `swpd`: How much virtual memory is being used
*   `free`: How much memory is free
*   `buff`: How much memory is being used for buffering
*   `cache`: How much memory is being used for caching

The `vmstat` command has several options available. To learn about all the options and what all the columns from the output represent, visit the respective pages in the manual using the following command:

```
man vmstat
```

The options that can be used with `vmstat` to show different information about memory are `-a` and `-s`. By using `vmstat -a`, the output will show the active and inactive memory:

![Figure 10.11 – Using vmstat -a to show the active and inactive memory](img/B19682_10_11.jpg)

Figure 10.11 – Using vmstat -a to show the active and inactive memory

Using `vmstat -s` will show detailed memory, CPU, and disk statistics.

All the commands discussed in this section are essential for troubleshooting any memory issues. There might be others you can use, but these are the ones you will find by default on any Linux distribution.

Nevertheless, there is one more that deserves to be mentioned in this section: the `sar` command. This can be installed in Ubuntu through the `sysstat` package. Therefore, install the package using the following command:

```
sudo apt install sysstat
```

Once the package has been installed, to be able to use the `sar` command to show detailed statistics about the system’s memory usage, you will need to enable the `sysstat` service. It needs to be active to collect data. By default, the service runs every 10 minutes and saves the logs inside the `/var/log/sysstat/saXX` directory. Every directory is named after the day the service runs on. For example, if we were to run the `sar` command on April 25, the service would look for data inside `/var/log/sysstat/sa25`. We ran the `sar` command on April 25 before starting the service, and an error occurred. Thus, to enable data collection, first, we will start and enable the `sysstat` service, then we will run the application. With the `sar` command, you can generate different reports in real time. For example, if we want to generate a memory report times every two seconds, we will use the `-``r` option:

![Figure 10.12 – Starting and enabling the service and running sar](img/B19682_10_12.jpg)

Figure 10.12 – Starting and enabling the service and running sar

The service’s name is `sadc`) and it uses the `sysstat` name for the package and service.

Important note

In the event of a system reboot, the service might not restart by default, even though the preceding commands were executed. To overcome this, on Ubuntu, you should edit the /`etc/default/sysstat` file and change the `ENABLED` status from `false` to `true`.

The output shown in *Figure 10**.12* will write one line every two seconds, five times in a row, and an average line at the end. It is a powerful tool that can be used for more than just memory statistics. There are options for CPU and disk statistics as well.

Overall, in this section, we covered the most important tools to use for troubleshooting memory issues. In the next section, we will cover tools to use for general system load issues.

### Commands for system load issues

Similar to what we covered in the previous sections, in this section, we will discuss `top` command is one of the most widely used when we’re trying to determine the sluggishness of a system. All the other tools, such as `vmstat` and `sar`, can also be used for CPU and system load troubleshooting.

A basic command for troubleshooting system load is `uptime`:

![Figure 10.13 – Using uptime for checking system load](img/B19682_10_13.jpg)

Figure 10.13 – Using uptime for checking system load

The `uptime` output shows three values at the end. Those values represent the load averages for 1, 5, and 15 minutes. The load average can give you a fair image of what happens with the system’s processes.

If you have a single CPU system, a load average of `1` means that that CPU is under full load. If the number is higher, this means that the load is much higher than the CPU can handle, and this will probably put a lot of stress on your system. Because of this, processes will take longer to execute and the system’s overall performance will be affected.

A high load average means that there are applications that run multiple threads simultaneously at once. Nevertheless, some load issues are not only the result of an overcrowded CPU – they can be the combined effect of CPU load, disk I/O load, and memory load. In this case, the Swiss Army knife for troubleshooting system load issues is the `top` command. The output of the `top` command constantly changes in real time, based on the system’s load.

By default, `top` sorts processes by how much CPU they use. It runs in interactive mode and, sometimes, the output is difficult to see on the screen. You can redirect the output to a file and use the command in batch mode using the `-b` option. This mode only updates the command a specified number of times. To run `top` in batch mode, run the following command:

```
top -b -n 1 | tee top-command-output
```

The `top` command could be a little intimidating for inexperienced Linux users. The output is highlighted in the following screenshot:

![Figure 10.14 – The output of the top command](img/B19682_10_14.jpg)

Figure 10.14 – The output of the top command

Let’s look at what the output means:

*   `us`: User CPU time
*   `sy`: System CPU time
*   `ni`: Nice CPU time
*   `id`: Idle CPU time
*   `wa`: Input/output wait time
*   `hi`: CPU hardware interrupts time
*   `si`: CPU software interrupts time
*   `st`: CPU steal time

Another useful tool for troubleshooting CPU usage and hard drive input/output time is `iostat`:

![Figure 10.15 – The output of iostat](img/B19682_10_15.jpg)

Figure 10.15 – The output of iostat

The CPU statistics are similar to the ones from the output of the `top` command shown earlier. The I/O statistics are shown below the CPU statistic and here is what each column represents:

*   `tps`: Transfers per second to the device (I/O requests)
*   `kB_read/s`: Amount of data read from the device (in terms of the number of blocks – kilobytes)
*   `kB_wrtn/s`: Amount of data written to the device (in terms of the number of blocks – kilobytes)
*   `kB_dscd/s`: Amount of data discarded for the device (in kilobytes)
*   `kB_read`: Total number of blocks read
*   `kB_wrtn`: Total number of blocks written
*   `kB_dscd`: Total number of blocks discarded

For more details about the `iostat` command, read the respective manual pages by using the following command:

```
man iostat
```

Besides the `iostat` command, there is another one that you could use, called `iotop`. It is not installed by default on Ubuntu, but you can install it with the following command:

```
sudo apt install iotop
```

Once the package has been installed, you will need `sudo` privileges to run it:

```
sudo iotop
```

The output is as shown in the following screenshot:

![Figure 10.16 – Running the iotop command](img/B19682_10_16.jpg)

Figure 10.16 – Running the iotop command

You can also run the `sysstat` service to troubleshoot system load issues, similar to how we used it for troubleshooting memory issues.

By default, `sar` will output the CPU statistics for the current day:

![Figure 10.17 – Running sar for CPU load troubleshooting](img/B19682_10_17.jpg)

Figure 10.17 – Running sar for CPU load troubleshooting

In the preceding screenshot, `sar` ran five times, every two seconds. Our local network servers are not under heavy load at this time, but you can imagine that the output would be different when the command is run on a heavily used server. As we pointed out in the previous section, the `sar` command has several options that could prove useful in finding solutions to potential problems. Run the `man sar` command to view the manual page containing all the available options.

There are many other tools that could be used for general system troubleshooting. We barely scratched the surface of this subject with the tools shown in this section. We advise you to search for more tools designed for general system troubleshooting if you feel the need to do so. Otherwise, the ones presented here are sufficient for you to generate a viable report about possible system issues.

Network-specific issues will be covered in the next section.

## Tools for troubleshooting network issues

Quite often, due to the complexities of a network, issues tend to appear. Networks are essential for everyday living. We use them everywhere, from our wireless smartwatch to our smartphone, to our computer, and up to the cloud. Everything is connected worldwide to make our lives better and a systems administrator’s life a little bit harder. In this interconnected world, things can go sideways quite easily, and network issues need troubleshooting.

Troubleshooting **network issues** is almost 80% of a system administrator’s job – probably even more. This number is not backed up by any official studies but more of a hands-on experience insight. Since most of the server and cloud issues are related to networking, an optimal working network means reduced downtime and happy clients and system administrators.

The tools we will cover in this section are the defaults on all major Linux distributions. All these tools were discussed in [*Chapter 7*](B19682_07.xhtml#_idTextAnchor139), *Networking with Linux*, and [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing Linux*, or will be discussed in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*, so we will only name them again from a problem-solving standpoint. Let’s break down the tools we should use on specific TCP/IP layers. Remember how many layers there are in the TCP/IP model? There are five layers available, and we will start from layer 1\. As a good practice, troubleshooting a network is best done through the stack, starting from the application layer all the way to the physical layer.

### Diagnosing the physical layer (layer 1)

One of the basic testing tools and one of the first to be used by most system administrators is the `ping` command. The name comes from `-c` option of the `ping` command. The output is as follows:

![Figure 10.18 – Running a basic test using the ping command](img/B19682_10_18.jpg)

Figure 10.18 – Running a basic test using the ping command

Ping is sending simple ICMP packets to the destination (in our case, it was [google.com](http://google.com)) and is waiting for a response. Once it is received and no packets are lost, this means that everything is working fine. The `ping` command can be used to test connections to local network systems, as well as remote networks. It is the first tool that’s used to test and isolate possible problems.

There are times when a simple test with the `ping` command is not enough. In this case, another versatile command is the `ip` command. You can use it to check whether there are any issues with the physical layer:

```
ip link show
```

You will see the output as follows:

![Figure 10.19 – Showing the state of the physical interfaces with the ip command](img/B19682_10_19.jpg)

Figure 10.19 – Showing the state of the physical interfaces with the ip command

In the preceding screenshot, you can see that the Ethernet interface is running well (`state UP`). As we are running on a virtual machine, we do not have a wireless connection. If we were to use a laptop, for example, the wireless connection would have been visible, showing something such as `wlp0s20f3`.

If any of the interfaces is not working, for example, the wireless one, the output of the preceding command would show `state DOWN`. In our case, we will bring the wireless interface up using the following command:

```
ip link set wlp0s20f3 up
```

Once executed, you can check the state of the interface by running the `ip` command once again:

```
ip link show
```

If you have direct access to a bare metal system, maybe a server, you can directly check whether the wires are connected. If, by any chance, you are using a wireless connection (not recommended), you will need to use the `ip` command.

Another useful tool for layer 1 is `ethtool`. On Ubuntu 22.04.2 LTS, it is installed by default. To check the Ethernet interface, run the following command by using the connection’s name (you’ve seen it while using the `ip` command):

```
ethtool enp1s0
```

By using `ethtool`, we can check whether a connection has negotiated the correct speed. In the output of the command (which we will not show here), you would see that the system has correctly negotiated a full 1,000 Mbps full-duplex connection, for example (it may differ in your case). In the next section, we will show you how to diagnose the layer 2 stack.

### Diagnosing the data link layer (layer 2)

The second layer in the TCP/IP stack is called the `ip` command and the `arp` command. The `arp` command, which comes from the `arp` command is available through the `net-tools` package. First, proceed and install it using the following command:

```
sudo apt install net-tools
```

To check the entries inside the ARP table, you can use the `arp` command with the `-a` (all) option. As we are running on a virtual machine, we will have only one entry. As a comparison, we run the same command on our host system, this time having three entries, one for the wireless connection, one for the wired connection, and another for the virtual bridge connection used by KVM. The following is an example:

![Figure 10.20 – Using the arp command to map the ARP entries](img/B19682_10_20.jpg)

Figure 10.20 – Using the arp command to map the ARP entries

The output of the `arp` command will show all the connected devices, with details about their IP and MAC addresses. Note that the MAC addresses have been blurred for privacy reasons.

Similar to the `arp` command, you can use the `ip neighbor show` command, as shown in the following screenshot. We will use our host system, not the virtual machine:

![Figure 10.21 – Listing the ARP entries using the ip command](img/B19682_10_21.jpg)

Figure 10.21 – Listing the ARP entries using the ip command

The `ip` command can be used to delete entries from the ARP list, like so:

```
ip neighbor delete 192.168.124.112 dev virbr0
```

We used the command to delete the virtual bridge connection, using its IP address and name from the list as shown in *Figure 10**.21*.

Both the `arp` and `ip` commands have a similar output. They are powerful commands and very useful for troubleshooting possible layer 2 issues. In the next section, we will show you how to diagnose the layer 3 stack.

### Diagnosing the internet layer (layer 3)

On the internet layer, layer 3, we are working with IP addresses only. We already know about the tools to use here, such as the `ip` command, the `ping` command, the `traceroute` command, and the `nslookup` command. Since we’ve already covered the `ip` and `ping` commands, we will only discuss how to use `traceroute` and `nslookup` here. The `traceroute` command is not installed by default in Ubuntu. You will have to install it using the following command:

```
sudo apt install traceroute
```

The `nslookup` package is already available in Ubuntu by default. First, to check for the routing table to see the list of gateways for different routes, we can use the `ip route` `show` command:

![Figure 10.22 – Showing the routing table using the “ip route show” command](img/B19682_10_22.jpg)

Figure 10.22 – Showing the routing table using the “ip route show” command

The `ip route show` command is showing the default gateway. An issue would be if it is missing or incorrectly configured.

The `traceroute` tool is used to check the path of traffic from the source to the destination. The following output shows the path of packets traveling from our local gateway to Google’s servers:

![Figure 10.23 – Using traceroute for path tracing](img/B19682_10_23.jpg)

Figure 10.23 – Using traceroute for path tracing

Packets don’t usually have the same route when they’re sending and coming back to the source. Packets are sent to gateways to be processed and sent to the destination on a certain route. When packets exceed the local network, their route can be inaccurately represented by the `traceroute` tool, since the packets it relies on could be filtered by many of the gateways on the path (*ICMP TTL Exceeded* packets are generally filtered).

Similar to `traceroute`, there is a newer tool called `tracepath`. It is installed by default on Ubuntu and is a replacement for `traceroute`. Both `traceroute` and `tracepath` use the UDP ports for tracking by default. The `tracepath` command can also be used with the `-n` option to show the IP address instead of the hostname. The following is an example of this:

![Figure 10.24 – Using the tracepath command](img/B19682_10_24.jpg)

Figure 10.24 – Using the tracepath command

Checking further network issues could lead to faulty DNS resolution, where a host can only be accessed by the IP address and not by the hostname. To troubleshoot this, even if it is not a layer 3 protocol, you could use the `nslookup` command, combined with the `ping` command.

If the `nslookup` command has the same IP output as the `ping` command, it means that everything is fine. If a different IP address shows up in the output, then you have an issue with your host’s configuration. In the next section, we will show you how to diagnose both the layer 4 and layer 5 stacks.

### Diagnosing the transport and application layers (layers 4 and 5)

The last two layers, layer 4 (`ss` command (where `ss` is short for **socket statistics**).

The `ss` command is a recent replacement for `netstat` and is used to see the list of all network sockets. As such, a list can have a significant size, so you could use several command options to reduce it.

For example, you could use the `-t` option to see only the TCP sockets, the `-u` option for UDP sockets, and `-x` for Unix sockets. Thus, to see TCP and UDP socket information, we will use the `ss` command with the `-t` option. Furthermore, to see all the listening sockets on your system, you can use the `-l` option. This one, combined with the `-u` and `-t` options, will show you all the UDP and TCP listening sockets on your system. The following is an excerpt from a much longer list:

![Figure 10.25 – Using the ss command to list TCP and UDP sockets and ﻿all listening sockets](img/B19682_10_25.jpg)

Figure 10.25 – Using the ss command to list TCP and UDP sockets and all listening sockets

The `ss` command is important for network troubleshooting when you want to verify the available sockets and the ones that are in the `LISTEN` state, for example. Another use is for the `TIME_WAIT` state, and in this case, you can use the command as shown in the following screenshot:

![Figure 10.26 – Showing the TIME_WAIT ports with the ss command](img/B19682_10_26.jpg)

Figure 10.26 – Showing the TIME_WAIT ports with the ss command

We used the `ss` command with the `-o` option and the `state time-wait` argument. The `TIME_WAIT` state is achieved when a socket closes for a short time.

For more information on this state, you can visit the following link: [https://vincent.bernat.ch/en/blog/2014-tcp-time-wait-state-linux](https://vincent.bernat.ch/en/blog/2014-tcp-time-wait-state-linux).

The `ss` tool should not be missing from a system administrator’s toolbox. Layer 5, the application layer, comprises protocols used by applications, and we will remember protocols such as the **Dynamic Host Configuration Protocol** (**DHCP**), the **Hypertext Transfer Protocol** (**HTTP**), and the **File Transfer Protocol** (**FTP**). Since diagnosing layer 5 is mainly an application troubleshooting process, this will not be covered in this section.

In the next section, we will discuss troubleshooting hardware issues.

## Tools for troubleshooting hardware issues

The first step in troubleshooting hardware issues is to check your hardware. A very good tool to see details about the system’s hardware is the `dmidecode` command. This command is used to read details about each hardware component in a human-readable format. Each piece of hardware has a specific DMI code, depending on its type. This code is specific to the **System Management Basic Input/Output System** (**SMBIOS**). There are 45 codes that are used by the SMBIOS. More information about the codes can be obtained at [https://www.thegeekstuff.com/2008/11/how-to-get-hardware-information-on-linux-using-dmidecode-command/](https://www.thegeekstuff.com/2008/11/how-to-get-hardware-information-on-linux-using-dmidecode-command/).

To view details about the system’s memory, you can use the `dmidecode` command with the `-t` option (from `TYPE`) and code `17`, which corresponds to the memory device code from SMBIOS. An example from our virtual machine is as follows:

![Figure 10.27 – Using dmidecode to view information about memory](img/B19682_10_27.jpg)

Figure 10.27 – Using dmidecode to view information about memory

To see details about other hardware components, use the command with the specific code.

Other quick troubleshooting tools include commands such as `lspci`, `lsblk`, and `lscpu`. The output of the `lsblk` command shows information about the disks and partitions being used on the system. The `lscpu` command will show details about the CPU.

Also, when you’re troubleshooting hardware issues, taking a quick look at the kernel’s logs could prove useful. To do this, use the `dmesg` command. You can use the `dmesg | more` command to have more control over the output.

As you’ve seen in this section, hardware troubleshooting is just as important and challenging as all other types of troubleshooting. Solving hardware-related issues is an integral part of any system administrator’s job. This involves constantly checking hardware components, replacing the faulty parts with new ones, and making sure that they run smoothly.

# Summary

In this chapter, we emphasized the importance of disaster recovery planning, backup and restore strategies, and troubleshooting various system issues. Every system administrator should be able to put their knowledge into practice when disaster strikes. Different types of failures will eventually hit the running servers, so solutions should be found as soon as possible to ensure minimal downtime and to prevent data loss.

This chapter represented the culmination of the *Advanced Linux Administration* section of this book. In the next chapter, we will introduce you to **server administration**, with emphasis on KVM virtual machine management, Docker containers, and different types of Linux server configuration.

# Questions

Troubleshooting is problem-solving at its best. Before we dive into the server section, let’s test your troubleshooting knowledge:

1.  Try to draft a DRP for your private network or small business.
2.  Back up your entire system using the 321 rule.
3.  Find out what your system’s top 10 processes are that use CPU the most.
4.  Find out what your system’s top 10 processes are that use RAM the most.

# Further reading

*   Ubuntu LTS official documentation: [https://ubuntu.com/server/docs](https://ubuntu.com/server/docs)
*   RHEL official documentation: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/)
*   SUSE official documentation: [https://documentation.suse.com/](https://documentation.suse.com/)

# Part 3:Server Administration

In this third part, you will learn about advanced Linux server administration tasks by setting up different types of servers, as well as working with and managing virtual machines and Docker containers.

This part has the following chapters:

*   [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231), *Working with Virtual Machines*
*   [*Chapter 12*](B19682_12.xhtml#_idTextAnchor257), *Managing Containers with Docker*
*   [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*