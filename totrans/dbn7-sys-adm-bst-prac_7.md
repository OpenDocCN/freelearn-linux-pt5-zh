# Chapter 7. Advanced System Management

In this final chapter, we'll cover briefly several advanced administration subjects. Remote backups and configuration administration will be covered, and we will briefly look at cluster management. Finally, we will look at one of the most useful administration tools for any Linux system, Webmin.

# Remote backups

If you're in charge of one or more systems that are installed at a distant location, backing them up individually can be a large chore. Fortunately, there are a number of packages that can help. Most backup packages, even those intended to create a backup of a single machine, have options to send the data to a remote location for actual storage. Of course, any of the packages with this capability can be used on multiple machines. However, there are two popular packages that not only provide backup services to multiple hosts from a central location, but provide for management of backup cycles and automated runs as well. They are **Amanda** and **Bacula**. Of course, other such packages exist, and there are some excellent third-party backup solutions as well. However, both Amanda and Bacula are provided as part of Debian, so they are free as well as able to handle many systems and a variety of backup media.

## Amanda

For many years, the **University of Maryland** (**UoM**) Computer Science department was the source of quality, free software that rivaled or even surpassed proprietary solutions. The **Advanced Maryland Automatic Network Disk Archiver** (**AMANDA**), is one such solution. Although no longer supported by UoM, it is now hosted on SourceForge, where it remains in active development. In addition to the free Community edition, there is a paid Enterprise edition that includes additional features, such as a graphical configuration utility.

### Note

The Amanda site notes that some Linux distributions are far behind in the release they include. This is not the case with Debian. Version 3.3.1 is included in Debian 7 Wheezy.

Originally oriented heavily towards centralized tape backup of many networked systems, it now supports disk and even cloud-based storage of backup data as well. Amanda requires a software client running on the systems to be backed up. Clients are available for most Unix type systems, as well as Mac OS/X and various Windows releases. The server side will run on pretty much any Unix- or Linux-based system. This makes Amanda especially useful for large, heterogeneous sites.

Amanda uses standard tape and disk file formats, which allows standard tools, such as `mt` and `tar`, to be used to browse or even recover data if desired. Amanda provides for parallel backups of many systems at once, backup file management; restore utilities that are easy to use, and several layers of security (including encryption of the backup data over the network, and encryption of the backup files). Amanda is implemented as a single central server that communicates with multiple clients.

A discussion of Amanda configuration could take up a whole book and is very dependent on the type of backup media you use and the systems you are backing up. However, briefly, the community (free) version of Amanda must be configured manually, by adding subdirectories and configuration files to the `/etc/amanda/` configuration directory. Refer to the documentation that comes with Amanda, or visit the [www.amanda.org](http://www.amanda.org) website for further information. Also, several commercially published books on backup and recovery have chapters on Amanda configuration.

## Bacula

Bacula is another popular free backup package. It is designed to be more modular than Amanda. Like Amanda, it requires a client on the system to be backed up. In addition to the client, however, there is an administrative console service, a status monitor service, a backup director which controls the actual backup operations, a storage service that keeps the actual backup data, and a database service where the backup information and catalogs are maintained. Of course, except for the client (which must reside on the systems being backed up), these services may be spread among different systems or consolidated on a single server.

Bacula configuration is object-oriented, in that you define clients, jobs, schedules jobs, file sets (to be backed up), storage pools to hold the backup data, messages (to handle emailing of reports), the catalog database, and the director which coordinates the whole thing. There are many useful functions, including some that allow restoration of a system without access to the catalog, creation of boot CDs which will allow a full, bare metal restore.

One thing to note is that the Bacula rescue CD is set up to restore disk partitions exactly as they existed at the time of the disk creation. If you need to run a bare metal restore to a system with a different disk configuration, the rescue CD also provides the fdisk utility, and you can add other utilities to it if you wish.

The Bacula director and storage components run on Linux, FreeBSD, or Solaris. It has also been reported to work on some Windows versions, Mac OS/X, and other BSD variants, although this is not officially supported. The client is available for many different systems, including various Linux, Windows, Mac, and BSD systems. Bacula is also reported to work on AIX, BSDI, and HPUX systems, although this is not officially supported.

Installing Bacula on Debian is straightforward. There are packages for each of the various parts, as well as, several meta packages. The bacula meta package installs both the `bacula-client` and `bacula-server` meta packages. The client package installs the Bacula console and file daemon (client). The server package installs the Bacula director and storage packages. There are several choices for the Bacula director, depending on what database you wish to use for your catalogs. The packages may be installed either via the meta packages or individually, as desired.

As with Amanda, Bacula is a comprehensive and complex solution. Aside from the comprehensive documentation available on the Bacula web site, [http://www.bacula.org/](http://www.bacula.org/), there are several books available that cover it well, including one available from [www.packtpub.com](http://www.packtpub.com) (*Network Backup with Bacula [How-to]*, by *Yauheni V. Pankov*, *[PACKT] Publishing*). Briefly, though, Bacula uses text files for configuration, in directories under `/etc/bacula`. The Bacula console package provides a graphical console application, although in practice the interface is actually a command line utility.

## Other backup systems

Of course, Debian offers other backup packages as well. They are less complex than Amanda or Bacula, but more suited to smaller environments. Most use standard file archiving utilities, and offer remote storage options (either via standard remote file specification or by using a client and server approach). Some offer backup cycle management utilities, backup encryption, communications encryption, and even de-duplication when using special backup storage formats. Synaptic or apt-cache can be used to search for these packages using the search term backup.

Some administrators prefer to keep it even simpler and write their own short scripts which use the basic archiving commands (such as `rsync`, `tar`, or the `EXT dump/restore` commands) to perform backups as a scheduled CRON job.

## Beyond backups

Of course, backups are not the only issue with managing multiple, remote systems. In particular, managing such multiple configurations using a centralized application is often desirable.

# Configuration management

One of the issues frequently faced by administrators is that of having multiple, remote systems all with similar software for the most part, but with minor differences in what is installed or running. Debian provides several packages that can help manage such an environment in a unified manner. Two of the more popular packages, both available in Debian, are FAI and Puppet. While we don't have the space to go into details, both applications are described briefly here.

## Fully Automated Installation

**Fully Automated Installation** (**FAI**) focuses on managing Linux installations, and is developed using Debian, although it works with many different distributions, not just Debian. FAI uses a class concept for categorizing similar systems, and provides a good deal of flexibility and customization via hooks. FAI provides for unattended, automatic installation as well as tools for monitoring and updating groups of systems. FAI is frequently used for creating and maintaining clusters. More information is available at [http://fai-project.org/](http://fai-project.org/).

## Puppet

Probably the best known application for distributed management is Puppet, developed by Puppet Labs. Unlike FAI, only the Open Source edition is free, the Enterprise edition, which has many additional features, is not. Puppet does include support for environments other than Linux. The desired configuration is described in a custom, high-level definition language, and distributed to systems with installed clients. Unlike FAI, Puppet does not provide its own bare metal remote installation method, but does use existing methods (such as `kickstart`) to provide this function. A number of companies that make heavy use of distributed and clustered systems use Puppet to manage their environments. More information is available at [http://puppetlabs.com/](http://puppetlabs.com/).

## Other packages

There are other packages that can be used to manage a distributed environment, such as Chef and BCFG2\. While simpler than Puppet or FAI, they support similar functions and have been used in some distributed and clustered environments.

The use of FAI, Puppet, and others in cluster management warrants a brief look at clustering next, and what packages in Debian support clustering.

# Clusters

A cluster is a group of systems that work together in such a way that the whole functions as a single unit. Such clusters can be loosely coupled or tightly coupled. A loosely coupled environment, each system is complete in itself, and can handle all of the tasks any of the other systems can handle. The environment provides mechanisms for redundancy, load sharing, and fail-over between systems, and is often called a **High Availability** (**HA**) cluster. In a tightly coupled environment, the systems involved are highly dependent on one another, often sharing memory and disk storage, and all work on the same task together. The environment provides mechanisms for data sharing, avoiding storage conflicts, keeping the systems in synchronization, and splitting up tasks appropriately. This design is often used in super-computing environments.

### Note

Clustering is an advanced technique that involves more than just installing and configuring software. It also involves hardware integration, and systems and network design, and implementation. Along with the URLs mentioned below, a good text on the subject is *Building Clustered Linux Systems*, by *Robert W. Lucke*, *Prentice Hall*. Here we will only touch the very basics, along with what tools Debian provides.

Let's take a brief look at each environment, and some of the tools used to create them.

## High Availability clusters

Two primary functions are required to implement a high availability cluster:

1.  A way to handle load balancing and individual host fail-over.
2.  A way to synchronize storage so that all servers provide the same view of the data they serve.

Debian includes meta packages that bring together software from the Linux High Availability project, including `cluster-agents` and `resource-agents`, two of the higher-level meta packages. These packages install various agents that are useful in coordinating and managing load balancing and fail-over. In some cases, a master server is designated to distribute the processing load among other servers.

Data synchronization is handled by using shared storage and any of the filesystems that provide for multiple accesses and shared files, such as NFS or AFS.

High Availability clusters generally use standard software, along with software that is readily available to manage the dynamics of such environments.

## Beowulf clusters

In addition to the considerations for High Availability clusters, more tightly coupled environments such as Beowulf clusters also require an infrastructure to manage and distribute computing tasks. There are several web pages devoted to creating a Beowulf cluster using Debian as well as packages that aid in creating such a cluster. One such page is [https://wiki.debian.org/StartaBeowulf](https://wiki.debian.org/StartaBeowulf), a Debian Wiki page on Beowulf basics. The manual for FAI, mentioned previously in configuration management, also has a section on creating a Beowulf cluster. Books are available as well. Debian provides several packages that are helpful in building such a cluster, such as the OpenMPI libraries for message passing, and various utilities that run commands on multiple systems, such as those in the `kadif` package. There are even projects that have released scripts and live CDs that allow you to set up a cluster quickly (one such project is the PelicanHPC project, developed for Debian Lenny, hosted at [http://www.pelicanhpc.org/](http://www.pelicanhpc.org/).

This type of cluster is not something that you can set up and go. Beowulf and other tightly coupled clusters are intended for highly parallel computing, and the programs that do the actual computing must be designed specifically for such an environment. That said, some packages for specific parallel computations do exist in Debian, such as `nwchem`, which provides several applications for computational chemistry that take advantage of parallelism.

## Common tools

Some common components of clusters have already been mentioned, such as the OpenMPI libraries. Aside from the meta-packages already mentioned, the *redhat-cluster* suite of tools is available in Debian, as well as many useful libraries, scheduling tools, and failover tools such as *booth*. All of these can be found using apt-cache or Synaptic by searching for "cluster".

# Webmin

Many administrators will never have to administer a cluster, and many won't be responsible for a large number of systems requiring central backup solutions. However, even administering a single system using command line tools and text editors can be a chore. Even clusters sometimes require administrative tasks on individual systems. Fortunately, there is an application that can ease many administrative tasks, is easy to use, and can handle many aspects of Linux administration. It is called **Webmin**.

Up until Debian Sarge, Webmin was a part of Debian distributions. However, the Debian developer in charge of packaging it had difficulty keeping up with the frequent releases, and it was eventually dropped from Debian. However, the upstream Webmin developers maintain current packages that install cleanly. Some users have reported issues because Webmin does not always handle configuration files exactly as Debian intends, but it most certainly attempts to handle them in a compatible manner, and while some users have experienced problems with upgrades, many administrators are quite happy with Webmin.

As long as you are willing to deal with conflicts during upgrades, or restrict use of modules that have major configuration impacts, you will find Webmin quite useful.

## Installing Webmin

Webmin may be installed by adding the following lines to your apt sources file:

```
deb http://download.webmin.com/download/repository sarge contrib
deb http://webmin.mirror.somersettechsolutions.co.uk/repository sarge contrib
```

Usually, this is added to a separate `webmin.list` file in `/etc/apt/sources.list.d`.

### Note

The use of 'sarge' for the release name in the configuration is not a mistake. Since Webmin was dropped after the Sarge release (Debian 3.1), the developers update the repository as it is and haven't bothered changing it to keep up with the Debian code names. However, the versions available in the repository are compatible with any Debian release since 3.1.

After updating your cache file, Webmin can be installed and maintained using apt-get, aptitude, or Synaptic. Also, if you request a Webmin upgrade from within Webmin itself on a Debian system, it will use the proper Debian package to upgrade.

## Using Webmin

Webmin runs in the background, and provides an HTTP or HTTPS server on localhost port 10,000\. You can use any web browser to connect to `http://localhost:10000/` to access Webmin. Upon first installation, only the root user or those in a group allowed to use `sudo` to access the root account, may log in but Webmin users can be managed separately or in conjunction with local users.

Webmin provides extensive and easy to understand menus and icons for various configuration tasks. Webmin is also highly modular and extensible, and an extensive list of standard modules is included with the base package. It is not possible to cover Webmin as fully here as it deserves, but a short list of some of its capabilities includes:

*   Configuration of Webmin itself (the server, users, modules, and security)
*   Local system user and password management
*   Filesystem management
*   Bootup and service management
*   CRON job management
*   Software updates
*   Basic filesystem backups
*   Authentication and security configuration
*   APACHE, DNS, SSH, and FTP (if you're using ProFTP) configuration
*   User mail management
*   Qmail or sendmail configuration
*   Network and Firewall configuration and management
*   Bandwidth monitoring
*   Printer management

There are even modules that apply to clusters. Also, Webmin can search and allow access to other Webmin servers on the local network or you can define remote servers manually. This allows a central Webmin server, installed on a particular system, to be the gateway to all of the other servers in your environment, essentially providing a single point of access to manage all Webmin enabled servers.

## Webmin and Debian

Webmin understands the configuration file layout of many distributions. The main problem is when a particular module does not handle certain types of configuration in the way the Debian developers prefer, which can make package upgrades somewhat difficult.

This can be handled in a couple of ways. Most modules provide a means to edit configuration files directly, so if you have read the Debian documentation you can modify the configuration appropriately to use Debian specific configuration techniques. Or, you may choose to allow Webmin to modify files as it sees fit, and handle any conflicts manually when you upgrade the software involved. Finally, you can avoid those modules involved with specific software that are more likely to cause problems.

### Note

One such module is Apache, which doesn't use links from `sites-enabled` to `sites-available`. Rather, it configures directly in the `sites-enabled` directory. Some administrators create the configuration in Webmin, and then move and link the files. Others prefer to manually configure Apache outside of Webmin.

Webmin modules are constantly changing, and some actually recognize the Debian file layouts well, so it is not possible to give a comprehensive list of modules to avoid at this time.

### Note

Best practice when using Webmin is to read the documentation and check the configuration files for specific software prior to using Webmin. Then, after configuring with Webmin, check the files again to determine whether changes may be required to work within the particular package's Debian configuration framework. Based upon this, you can decide whether to continue to configure using Webmin or switch back to manual configuration of that particular software.

## Webmin security

Security is always a concern when remote access to a system is involved. Webmin handles this by requiring authentication and providing for detailed access restrictions that provide a layer of control beyond the firewall. Webmin users can be defined separately, or certain local users can be designated. Access to the various modules in Webmin can be restricted to certain users or groups of users, and detailed logs of Webmin actions are kept.

## Usermin

In addition to Webmin, there is a server called Usermin which may be installed from the same repository as Webmin. It allows individual users to perform a number of functions more easily, such as changing their password, accessing their files, read and manage their email, and managing some aspects of their user profile. It is also modular and has the same security features as Webmin.

# Summary

Several powerful and flexible central backup solutions exist that help manage backups for multiple remote servers and sites. Debian provides packages that assist in building High Availability and Beowulf style multiprocessing clusters as well. And, whether you are involved in managing clusters or not, or even a single system, Webmin can ease an administrator's tasks.