# Containers, Virtual Machines, and the Cloud

In this chapter, we will cover the following topics:

*   Using Linux Containers
*   Using Docker
*   Using Virtual Machines in Linux
*   Linux in the cloud

# Introduction

Modern Linux applications can be deployed on dedicated hardware, containers, Virtual Machines (VMs), or the cloud. Each solution has strengths and weaknesses, and each of them can be configured and maintained with scripts as well as GUIs.

A container is ideal if you want to deploy many copies of a single application where each instance needs its own copy of data. For example, containers work well with database-driven web servers where each server needs the same web infrastructure but has private data.

However, the downside of a container is that it relies on the host system's kernel. You can run multiple Linux distributions on a Linux host, but you can't run Windows in a container.

Using a VM is your best bet if you need a complete environment that is not the same for all instances. With VMs, you can run Windows and Linux on a single host. This is ideal for validation testing when you don't want a dozen boxes in your office but need to test against different distributions and operating systems.

The downside of VMs is that they are huge. Each VM implements an entire computer-operating system, device drivers, all the applications and utilities, and so on. Each Linux VM needs at least one core and 1 GB RAM. A Windows VM may need two cores and 4 GB RAM. If you wish to run multiple VMs simultaneously, you need enough RAM to support each one of the VMs; otherwise, the host will start swapping and performance will suffer.

The cloud is like having many computers and lots of bandwidth at your fingertips. You may actually be running on a VM or container in the cloud, or you might have your own dedicated system.

The biggest advantage of the cloud is that it can scale. If you think your application might go viral or your usage is cyclic, the ability to scale up and down quickly without needing to buy or lease new hardware new connectivity is necessary. For example, if your system processes college registrations, it will be overworked for about two weeks, twice a year, and almost dormant for the rest of the time. You may need a dozen sets of hardware for those two weeks, but you don't want to have them sitting idle for the rest of the year.

The downside of the cloud is that it's not something you can see. All of the maintenance and configuration has to be done remotely.

# Using Linux containers

**Linux Container** (**lxc**) packages provide the basic container functionality used by Docker and LXD container deployment systems.

A Linux container uses kernel level support for **Control Groups** (**cgroups**) and the `systemd` tools described in [Chapter 12](5c74c943-1155-4720-a3cb-f4740f691f8c.xhtml), *Tuning a Linux System*. The cgroups support provides tools to control the resources available to a group of programs. This informs kernel control about the resources that are available to the processes running in a container. A container may have limited access to devices, network connectivity, memory, and so on. This control keeps the containers from interfering with each other or potentially damaging the host system.

# Getting ready

Container support is not provided in stock distributions. You'll need to install it separately. The level of support across distributions is inconsistent. The **lxc** container system was developed by Canonical, so Ubuntu distributions have complete container support. Debian 9 (Stretch) is better than Debian 8 (Jessie) in this regard.

Fedora has limited support for lxc containers. It is easy to create privileged containers and a bridged Ethernet connection, but as of Fedora 25, the `cgmanager` service required for unprivileged containers is unavailable.

SuSE supports limited use of lxc. SuSE's `libvirt-lxc` package is similar but not identical to lxc. SuSE's `libvirt-lxc` package is not covered in this chapter. A privileged container with no Ethernet is easy to create under SuSE, but it does not support unprivileged containers and bridged Ethernet.

Here's how to install `lxc` support on major distributions.

For Ubuntu, use the following code:

```
    # apt-get install lxc1

```

Next we have Debian. Debian distributions may only include the security repositories in `/etc/apt/sources.list`. If so, you'll need to add `deb http://ftp.us.debian.org/debian stretch main contrib` to `/etc/apt/sources.list` and then perform `apt-get update before`, loading the `lxc` package:

```
    # apt-get install lxc

```

For OpenSuSE, use the following code:

```
    # zypper install lxc
 RedHat, Fedora:

```

For Red Hat/Fedora-based systems, add the following `Epel` repository:

```
    # yum install epel-release

```

Once you've done this, install the following packages before you install lxc support:

```
    # yum install perl libvirt debootstrap

```

The `libvirt` package provides networking support, and `debootstrap` is required to run Debian-based containers:

```
    # yum install lxc lxc-templates tunctl bridge-utils

```

# How to do it...

The `lxc` package adds several commands to your system. These include:

*   `lxc-create`: This is to create an lxc container
*   `lxc-ls`: This is a list of the available containers
*   `lxc-start`: This is to start a container
*   `lxc-stop`: This is to stop a container
*   `lxc-attach`: This is to connect to the root shell of a container
*   `lxc-console`: This is to connect to a login session in a container

On Red Hat-based systems, you may need to disable SELinux while testing. On OpenSuSE systems, you may need to disable **AppArmor**. You'll need to reboot after disabling AppArmor via `yast2`.

Linux containers come in two basic flavors: privileged and unprivileged. Privileged containers are created by the root and the underlying system has root privileges. An unprivileged container is created by a user and only has user privileges.

Privileged containers are easier to create and more widely supported since they don't require `uid` and `gid` mapping, device permissions, and so on. However, if a user or application manages to escape from the container, they'll have full privileges on the host.

Creating a privileged container is a good way to confirm that all the required packages are installed on your system. After you create a privileged container, use unprivileged containers for your applications.

# Creating a privileged container

The easiest way to get started with Linux containers is to download a prebuilt distribution in a privileged container. The `lxc-create` command creates a base container structure and can populate it with a predefined Linux distribution. The syntax of the `lxc-create` command is as follows:

```
    lxc-create -n NAME -t TYPE

```

The `-n` option defines a name for this container. This name will be used to identify this container when it is started, stopped, or reconfigured.

The `-t` option defines the template to be used to create this container. The type `download` connects your system to a repository of prebuilt containers and prompts you for the container to download.

This is an easy way to experiment with other distributions or create an application that needs a distribution other than the host's Linux distribution:

```
    $ sudo lxc-create -t download -n ContainerName

```

The download template retrieves a list of the available predefined containers from the Internet and populates the container from the network archive. The create command provides a list of the available containers and then prompts for the **Distribution**, **Release**, and Architecture. You can only run a container if your hardware supports this Architecture. You cannot run an Arm container if your system has an Intel CPU, but you can run a 32-bit i386 container on a system with a 64-bit Intel CPU:

```
$ sudo lxc-create -t download -n ubuntuContainer
...
ubuntu  zesty   armhf   default 20170225_03:49
ubuntu  zesty   i386    default 20170225_03:49
ubuntu  zesty   powerpc default 20170225_03:49
ubuntu  zesty   ppc64el default 20170225_03:49
ubuntu  zesty   s390x   default 20170225_03:49
---

Distribution: ubuntu
Release: trusty
Architecture: i386 

Downloading the image index
Downloading the rootfs
Downloading the metadata
The image cache is now ready
Unpacking the rootfs

---
You just created an Ubuntu container (release=trusty, arch=i386, variant=default)
To enable sshd, run: apt-get install openssh-server
For security reason, container images ship without user accounts and without a root password.
Use lxc-attach or chroot directly into the rootfs to set a root password or create user accounts.

```

You can create a container based on your current distribution by selecting a template that matches the current installation. The templates are defined in `/usr/share/lxc/templates`:

```
    # ls /usr/share/lxc/templates
 lxc-busybox   lxc-debian   lxc-download ...

```

To create a container for your current distribution, select the appropriate template and run the `lxc-create` command. The download process and installation takes several minutes. The following example skips most of the installation and configuration messages:

```
$ cat /etc/issue
Debian GNU/Linux 8
$ sudo lxc-create -t debian -n debianContainer
debootstrap is /usr/sbin/debootstrap
Checking cache download in /var/cache/lxc/debian/rootfs-jessie-i386 ... 
Downloading debian minimal ...
I: Retrieving Release 
I: Retrieving Release.gpg 
I: Checking Release signature
I: Valid Release signature (key id 75DDC3C4A499F1A18CB5F3C8CBF8D6FD518E17E1)
...
I: Retrieving Packages 
I: Validating Packages 
I: Checking component main on http://http.debian.net/debian...
I: Retrieving acl 2.2.52-2
I: Validating acl 2.2.52-2
I: Retrieving libacl1 2.2.52-2
I: Validating libacl1 2.2.52-2

I: Configuring libc-bin...
I: Configuring systemd...
I: Base system installed successfully.
Current default time zone: 'America/New_York'
Local time is now:      Sun Feb 26 11:38:38 EST 2017.
Universal Time is now:  Sun Feb 26 16:38:38 UTC 2017.

Root password is 'W+IkcKkk', please change !

```

The preceding command populates the new container from the repositories defined in your package manager. Before you can use a container, you must start it.

# Starting a container

The `lxc-start` command starts a container. As with other lxc commands, you must provide the name of the container to start:

```
    # lxc-start -n ubuntuContainer

```

The boot sequence may hang and you may see errors similar to the following one. These are caused by the container's boot sequence trying to perform graphics operations, such as displaying a splash screen without graphics support in the client:

```
    <4>init: plymouth-upstart-bridge main process (5) terminated with   
    status 1
 ...

```

You can wait for these errors to time out and ignore them, or you can disable the splash screen. Disabling the splash screen varies between distributions and releases. The files may be in `/etc/init`, but that's not guaranteed. 

There are two ways to work within a container:

*   `lxc-attach`: This attaches directly to a root account on a running container
*   `lxc-console`: This opens a console for a login session on a running container

The first use of a container is to attach directly to create user accounts:

```
# lxc-attach -n containerName
root@containerName:/#
root@containerName:/# useradd -d /home/USERNAME -m  USERNAME
root@containerName:/# passwd USERNAME
Enter new UNIX password:
Retype new UNIX password:

```

After you've created a user account, log in as an unprivileged user or root with the `lxc-console` application:

```
$ lxc-console -n containerName
Connected to tty 1
Type <Ctrl+a q> to exit the console, 
<Ctrl+a Ctrl+a> to enter Ctrl+a itself
Login:

```

# Stopping a container

The `lxc-stop` command stops a container:

```
    # lxc-stop -n containerName

```

# Listing known containers

The `lxc-ls` command lists the container names that are available for the current user. This does not list all the containers in a system, only those that the current user owns:

```
    $ lxc-ls
 container1Name container2Name...

```

# Displaying container information

The `lxc-info` command displays information about a container:

```
$ lxc-info -n containerName
Name:   testContainer
State:   STOPPED

```

This command will only display information about a single container, though. Using a shell loop, as described in [Chapter 1](195d920d-33c2-41d6-bd33-37d75f9c37f1.xhtml), *Shell Something Out*, we can display information about all the containers:

```
$ for c in `lxc-ls` 
do
lxc-info -n $c
echo
done
Name:  name1
State:  STOPPED

Name:  name2
State:  RUNNING
PID:  1234
IP  10.0.3.225
CPU use:  4.48 seconds
BlkIO use:  728.00 KiB
Memory use:  15.07 MiB
KMem use:  2.40 MiB
Link:  vethMU5I00
 TX bytes:  20.48 KiB
 RX bytes:  30.01 KiB
 Total bytes:  50.49 KiB

```

If the container is stopped, there is no status information available. Running containers record their CPU, memory, disk (block), I/O, and network usage. This tool lets you monitor your containers to see which ones are most active.

# Creating an unprivileged container

Unprivileged containers are recommended for normal use. There is potential for a badly configured container or badly configured application to allow control to escape from the container. Since containers invoke system calls in the host kernel, if the container is running as the root, the system calls will also run as the root. However, unprivileged containers run with normal user privileges and are thus safer.

To create unprivileged containers, the host must support Linux Control Groups and uid mapping. This support is included in basic Ubuntu distributions, but it needs to be added to other distributions. The `cgmanager` package is not available in all distributions. You cannot start an unprivileged container without this package:

```
    # apt-get install cgmanager uidmap systemd-services

```

Start `cgmanager`:

```
    $ sudo service cgmanager start

```

Debian systems may require that clone support be enabled. If you receive a `chown` error when creating a container, these lines will fix it:

```
    # echo 1 > /sys/fs/cgroup/cpuset/cgroup.clone_children
 # echo 1 > /proc/sys/kernel/unprivileged_userns_clone 

```

The username of an account that's allowed to create containers must be included in the `etc` mapping tables:

```
    $ sudo usermod --add-subuids 100000-165536 $USER
 $ sudo usermod --add-subgids 100000-165536 $USER
 $ sudo chmod +x $HOME

```

These commands add the user to the User ID and Group ID mapping tables `(/etc/subuid` and `/etc/subgid`) and assign UIDs from `100000 -> 165536` to the user.

Next, set up the configuration file for your containers:

```
    $ mkdir ~/.config/lxc
 $ cp /etc/lxc/default.conf ~/.config/lxc

```

Add the following lines to `~/.config/lxc/default.conf`:

```
    lxc.id_map = u 0 100000 65536
 lxc.id_map = g 0 100000 65536

```

If the containers support network access, add a line to `/etc/lxc/lxc-usernet` to define the users who will have access to the network bridge:

```
    USERNAME veth BRIDGENAME COUNT

```

Here, `USERNAME` is the name of the user who owns the container. `veth` is the usual name for the virtual Ethernet device. `BRIDGENAME` is the name that's displayed by `ifconfig`. It is usually either `br0` or `lxcbro`. `COUNT` is the number of simultaneous connections that will be allowed:

```
    $ cat /etc/lxc/lxc-usernet
 clif veth lxcbr0 10

```

# Creating an Ethernet bridge

A container cannot access your Ethernet adapter directly. It requires a bridge between the Virtual Ethernet and the actual Ethernet. Recent Ubuntu distributions create an Ethernet bridge automatically when you install the lxc package. Debian and Fedora may require that you manually create the bridge. To create a bridge on Fedora, use the `libvirt` package to create a virtual bridge first:

```
    # systemctl start libvirtd

```

Then, edit `/etc/lxc/default.conf` to reference `virbr0` instead of `lxcbr0`:

```
    lxc.network_link = virbr0

```

If you've already created a container, edit the config file for that container as well.

To create a bridge on Debian systems, you must edit the network configuration and the container configuration files.

Edit `/etc/lxc/default.conf`. Comment out the default empty network and add a definition for the lxc bridge:

```
    # lxc.network.type = empty
 lxc.network.type = veth
 lxc.network.link = lxcbr0
 lxc.network.flage = up`

```

Next, create the networking bridge:

```
    # systemctl enable lxc-net
 # systemctl start lxc-net

```

Containers created after these steps are performed will have networking enabled. Network support can be added to the existing containers by adding the `lxc.network` lines to the container's config file.

# How it works...

The container created by the `lxc-create` command is a directory tree that includes the configuration options and root filesystem for the container. Privileged containers are constructed under `/var/lib/lxc`. Nonprivileged containers are stored under `$HOME/.local/lxc`:

```
    $ ls /var/lib/lxc/CONTAINERNAME
 config rootfs

```

You can examine or modify a container's configuration by editing the config file in the container's top directory:

```
    # vim /var/lib/lxc/CONTAINERNAME/config

```

The `rootfs` folder contains a root filesystem for the container. This is the root (`/`) folder of a running container:

```
    # ls /var/lib/lxc/CONTAINERNAME/rootfs
 bin   boot cdrom dev  etc   home  lib   media mnt   proc
 root  run  sbin  sys  tmp   usr   var

```

You can populate a container by adding, deleting, or modifying files in the `rootfs` folder. For instance, to run web services, a container might have basic web services installed via the package manager and the actual data of each service installed by copying files to the `rootfs`.

# Using Docker

The `lxc` containers are complex and can be difficult to work with. These issues led to the Docker package. Docker uses the same underlying Linux functionalities of `namespaces` and `cgroups` to create lightweight containers.

Docker is only officially supported on 64-bit systems, making `lxc` the better choice for legacy systems.

The major difference between a Docker container and an lxc container is that a Docker container commonly runs one process, while an lxc container runs many. To deploy a database-backed web server, you need at least two Docker containers–one for the web server and one for the database server–but only one lxc container.

The Docker philosophy makes it easy to construct systems from smaller building blocks, but it can make it harder to develop blocks since so many Linux utilities are expected to run inside a full Linux system with `crontab` entries to carry out operations such as cleanup, log rotation, and so on.

Once a Docker container is created, it will run exactly as expected on other Docker servers. This makes it very easy to deploy Docker containers on cloud clusters or remote sites.

# Getting ready

Docker is not installed with most distributions. It is distributed via Docker's repositories. Using these requires adding new repositories to your package manager with new checksums.

Docker has instructions for each distribution and different releases on their main page, which is available at [http://docs.docker.com](http://docs.docker.com).

# How to do it...

When Docker is first installed, it is not running. You must start the server with a command such as the following:

```
    # service docker start

```

The Docker command has many subcommands that provide functionality. These commands will find a Docker container and download and run it. Here's a bit about the subcommands:

*   `# docker search`: This searches Docker archives for containers with names that match a key
*   `# docker pull`: This pulls the named container to your system
*   `# docker run`: This runs an application in a container
*   `# docker ps`: This lists the running Docker containers
*   `# docker attach`: This attaches to a running container
*   `# docker stop`: This stops a container
*   `# docker rm`: This removes a container

The default Docker installation requires that the `docker` command be run either as a `root` or using `sudo`.

Each of these commands have a `man` page. This page is named by combining the command and subcommand with a dash. To view the `docker search` man page, use `man docker-search`.

The next recipe demonstrates how to download a Docker container and run it.

# Finding a container

The `docker search` command returns a list of Docker containers that match a search term:

```
    docker search TERM

```

Here TERM is an alphanumeric string (no wild cards). The search command will return up to 25 containers that include the string in their name:

```
# docker search apache
NAME            DESCRIPTION                STARS OFFICIAL   AUTOMATED
eboraas/apache  Apache (with SSL support)  70                   [OK]
bitnami/apache  Bitnami Apache Docker      25                   [OK]
apache/nutch    Apache Nutch               12                   [OK]
apache/marmotta Apache Marmotta             4                   [OK]
lephare/apache  Apache container            3                   [OK]

```

Here STARS represent a rating for the container. The containers are ordered with the highest rating first.

# Downloading a container

The `docker pull` command downloads a container from the Docker registry. By default, it pulls data from Docker's public registry at `registry-1.docker.io`. The downloaded container is added to your system. The containers are commonly stored under /`var/lib/docker`:

```
# docker pull lephare/apache
latest: Pulling from lephare/apache
425e28bb756f: Pull complete 
ce4a2c3907b1: Extracting [======================> ] 2.522 MB/2.522 MB
40e152766c6c: Downloading [==================>    ] 2.333 MB/5.416 MB
db2f8d577dce: Download complete 
Digest: sha256:e11a0f7e53b34584f6a714cc4dfa383cbd6aef1f542bacf69f5fccefa0108ff8
Status: Image is up to date for lephare/apache:latest

```

# Starting a Docker container

The `docker run` command starts a process in a container. Commonly, the process is a `bash` shell that allows you to attach to the container and start other processes. This command returns a hash value that defines this session.

When a Docker container starts, a network connection is created for it automatically.

The syntax for the run command is as follows:

```
    docker run [OPTIONS] CONTAINER COMMAND

```

The `docker run` command supports many options, including:

*   `-t`: Allocate a pseudo tty (by default, false)
*   `-i`: Keep an interactive session open while unattached
*   `-d`: Start the container detached (running in the background)
*   `--name`: The name to assign to this instance

This example starts the bash shell in the container that was previously pulled:

```
 # docker run -t -i -d --name leph1 lephare/apache  /bin/bash
 1d862d7552bcaadf5311c96d439378617d85593843131ad499...

```

# Listing the Docker sessions

The `docker p`s command lists the currently running Docker sessions:

```
# docker ps
CONTAINER ID  IMAGE           COMMAND   CREATED  STATUS  PORTS  NAMES
123456abc     lephare/apache  /bin/bash 10:05    up      80/tcp leph1

```

The `-a` option will list all the Docker containers on your system, whether they are running or not.

# Attaching your display to a running Docker container

The `docker attach` command attaches your display to the `tty` session in a running container. You need to run as the root within this container.

To exit an attached session, type `^P^Q`.

This example creates an HTML page and starts the Apache web server in the container:

```
$ docker attach leph1
root@131aaaeeac79:/# cd /var/www
root@131aaaeeac79:/var/www# mkdir symfony
root@131aaaeeac79:/var/www# mkdir symfony/web
root@131aaaeeac79:/var/www# cd  symfony/web
root@131aaaeeac79:/var/www/symfony/web# echo "<html><body><h1>It's Alive</h1></body></html>"   
    >index.html
root@131aaaeeac79:/# cd /etc/init.d
root@131aaaeeac79:/etc/init.d# ./apache2 start
[....] Starting web server: apache2/usr/sbin/apache2ctl: 87: ulimit: error setting limit (Operation 
    not permitted)
Setting ulimit failed. See README.Debian for more information.
AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 
    172.17.0.5\. Set the 'ServerName' directive globally to suppress this message
. ok 

```

Browsing to `172.17.0.5` will show the `It's Alive` page.

# Stopping a Docker session

The `docker stop` command terminates a running Docker session:

```
    # docker stop leph1

```

# Removing a Docker instance

The `docker rm` command removes a container. The container must be stopped before removing it. A container can be removed either by name or identifier:

```
    # docker rm leph1

```

Alternatively, you can use this:

```
    # docker rm 131aaaeeac79

```

# How it works

The Docker containers use the same `namespace` and `cgroup` kernel support as that of the `lxc` containers. Initially, Docker was a layer over `lxc`, but it has since evolved into a unique system.

The main configuration files for the server are stored at /`var/lib/docker` and `/etc/docker`.

# Using Virtual Machines in Linux

There are four options for using VMs in Linux. The three open source options are KVM, XEN, and VirtualBox. Commercially, VMware supplies a virtual engine that can be hosted in Linux and an executive that can run VMs.

VMware has been supporting VMs longer than anyone else. They support Unix, Linux, Mac OS X, and Windows as hosts and Unix, Linux, and Windows as guest systems. For commercial use, VMware Player or VMWare Workstation are the two best choices you have.

KVM and VirtualBox are the two most popular VM engines for Linux. KVM delivers better performance, but it requires a CPU that supports virtualization (Intel VT-x). Most modern Intel and AMD CPUs support these features. VirtualBox has the advantage of being ported to Windows and Mac OS X, allowing you to move a virtual machine to another platform easily. VirtualBox does not require VT-x support, making it suitable for legacy systems as well as modern systems.

# Getting ready

VirtualBox is supported by most distributions, but it may not be part of these distributions' default package repositories.

To install VirtualBox on Debian 9, you need to add the virtualbox.org repository to the sites that apt-get will accept packages from:

```
# vi /etc/apt/sources.list
## ADD:
deb http://download.virtualbox.org/virtualbox/debian stretch contrib

```

The `curl` package is required to install the proper keys. If this is not already present, install it before adding the key and updating the repository information:

```
# apt-get install curl
# curl -O https://www.virtualbox.org/download/oracle_vbox_2016.asc
# apt-key add oracle_vbox_2016.asc
# apt-get update

```

Once the repository is updated, you can install VirtualBox with `apt-get`:

```
# apt-get install virtualbox-5.1

OpenSuSE 
# zypper install gcc make kernel-devel
Open yast2, select Software Management, search for virtualbox.
Select virtualbox, virtualbox-host-kmp-default, and virtualbox-qt.

```

# How to do it...

When VirtualBox is installed, it creates an item in the start menu. It may be under System or Applications/System Tools. The GUI can be started from a terminal session as `virtualbox` or as `VirtualBox`.

The VirtualBox GUI makes it easy to create and run VMs. The GUI has a button named New in the upper-left corner; this is used to create a new, empty VM. The wizard prompts you for information such as memory and disk limits for the new VM.

Once the VM is created, the Start button is activated. The default settings connect the virtual machine's CD-ROM to the host's CD-ROM. You can put an installation disk in the CD-ROM and click on Start to install the operating system on a new VM.

# Linux in the cloud

There are two primary reasons to use a cloud server. Service providers use a commercial cloud service, such as Amazon's AWS, because it lets them easily ramp up their resources when demand is higher and ramp down their costs when demand is lower. Cloud storage providers, such as Google Docs, allow users to access their data from any device and share data with others.

The OwnCloud package transforms your Linux server into a private cloud storage system. You can use an OwnCloud server as a private corporate file sharing system to share files with friends or as a remote backup for your phone or tablet.

The OwnCloud project forked in 2016\. The NextCloud server and applications are expected to use the same protocol as that of OwnCloud and to be interchangeable.

# Getting ready

Running the OwnCloud package requires a **LAMP** (**Linux, Apache, MySQL, PHP**) installation. These packages are supported by all Linux distributions, though they may not be installed by default. Administering and installing MySQL is discussed in [Chapter 10](20129291-0a5b-43a8-ad0c-54c74992d0e3.xhtml), *Administration Calls*.

Most distributions do not include the OwnCloud server in their repositories. Instead, the OwnCloud project maintains repositories to support the distributions. You'll need to attach OwnCloud to your RPM or apt repository before you download.

# Ubuntu 16.10

The following steps will install the LAMP stack on a Ubuntu 16.10 system. Similar commands will work for any Debian-based system. Unfortunately, package names sometimes vary between releases:

```
 apt-get install apache2
 apt-get install mysql-server php-mysql

```

OwnCloud requires security beyond default settings. The `mysql_secure_installation` script will configure MySQL properly:

```
    /usr/bin/mysql_secure_installation

```

Configure the `OwnCloud` repository:

```
curl \ https://download.owncloud.org/download/repositories/stable/ \ Ubuntu_16.10/Release.key/'| sudo tee \ /etc/apt/sources.list.d/owncloud.list

apt-get update

```

Once the repository is in place, apt will install and start the server:

```
    apt-get install owncloud

```

# OpenSuSE Tumbleweed

Install the **LAMP** stack with **Yast2**. Open `yast2`, select Software Management, and install `apache2`, `mysql`, and `owncloud-client`.

Next, select the `System` tab, and from this tab, select the `Services Manager` tab. Confirm that the `mysql` and `apache2` services are enabled and active.

These steps install the OwnCloud client that will let you synchronize your workspace to an OwnCloud server and the system requirements for a server.

OwnCloud requires security beyond default settings. The `mysql_secure_installation` script will configure MySQL properly:

```
    /usr/bin/mysql_secure_installation

```

The following commands will install and start the OwnCloud server. The first three commands configure `zypper` to include the OwnCloud repository. Once these repositories are added, the Owncloud package is installed like any other package:

```
rpm --import https://download.owncloud.org/download/repositories/stable/openSUSE_Leap_42.2/repodata/repomd.xml.key

zypper addrepo http://download.owncloud.org/download/repositories/stable/openSUSE_Leap_42.2/ce:stable.repo

zypper refresh

zypper install owncloud 

```

# How to do it...

Once OwnCloud is installed, you can configure an admin account, and from there, add user accounts. The NextCloud Android app will communicate with the OwnCloud server as well as the NextCloud server.

# Configuring OwnCloud

Once `owncloud` is installed, you can configure it by browsing to your local address:

```
    $ konqueror http://127.0.0.1/owncloud

```

The initial screen will prompt you for an admin username and password. You can log in as the user to create backups and copy files between phones, tablets, and computers.

# There's more...

The bare installation process we just discussed is suitable for testing. OwnCloud and NextCloud will use HTTPS sessions if HTTPS support is available. Enabling HTTPS support requires an X.509 security certificate.

You can purchase a security certificate from one of the dozens of commercial providers, self-sign a certificate for your own use, or create a free certificate with **Let's Encrypt** (http://letsencrypt.org).

A self-signed certificate is adequate for testing, but most browsers and phone apps will flag this as an untrusted site. Let's Encrypt is a service of the Internet Security Research Group (ISRG). The certificates they generate are fully registered and all applications can accept them.

The first step in acquiring a certificate is verifying that your site is what you claim it is. Let's Encrypt certificates are validated using a system called Automated Certificate Management Environment (ACME). The ACME system creates a hidden file on your web server, tells the **Certificate Authority** (**CA**) where that file is, and the CA confirms that the expected file is there. This proves that you have access to the web server and that DNS records point to the proper hardware.

If you are using a common web server, such as Nginx or Apache, the simplest way to set up your certificates is with the `certbot` created by EFF:

```
    # wget https://dl.eff.org/certbot-auto
 # chmod a+x certbot-auto
 # ./certbot-auto

```

This robot will add new packages and install your new certificate in the proper place.

If you are using a less common server or have a non-standard installation, the `getssl` package is more configurable. The `getssl` package is a bash script that reads two configuration files to automate the creation of the certificate. Download the package from here and unzip from `https://github.com/srvrco/getssl`.

Unzipping `getssl.zip` creates a folder named `getssl_master`.

Generating and installing the certificates requires three steps:

1.  Create the default configuration files with `getssl -c DOMAIN.com`.
2.  Edit the configuration files.
3.  Create the certificates.

Start by `cd-ing` to the `getssl_master` folder and creating the configuration files:

```
    # cd getssl_master
 # getssl -c DOMAIN.com

```

Replace `DOMAIN` with the name of your domain.

This step creates the `$HOME/.getssl` and `$HOME/.getssl/DOMAIN.com` folders and creates a file named `getssl.cfg` in both of these. Each of these files must be edited.

Edit `~/.getssl/getssl.cfg` and add your email address:

```
    ACCOUNT_EMAIL='myName@mySite.com'

```

The default values in the rest of the fields are suitable for most sites.

Next, edit `~/.getssl/DOMAIN.com/getssl.cfg`. There are several fields to modify in this file.

The main change is to set the Acme Challenge Location (ACL) field. The ACME protocol will try to find a file in [http://www.DOMAIN.com/.well-known/acme-challenge](http://www.DOMAIN.com/.well-known/acme-challenge). The ACL value is the physical location of that folder on your system. You must create the `.well-known` and .`well-known/acme-challenge` folders and set ownership if they don't exist.

If your web pages are kept in `/var/web/DOMAIN`, you could create new folders as follows:

```
# mkdir /var/web/DOMAIN/.well-known
# mkdir /var/web/DOMAIN/.well-known/acme-challenge
# chown webUser.webGroup /var/web/DOMAIN/.well-known
# chown webUser.webGroup /var/web/DOMAIN/.well-known/acme-challenge

```

The ACL lines would resemble the following:

```
ACL="/var/web/DOMAIN/.well-known/acme-challenge"
USE_SINGLE_ACL="true"

```

You must also define where the certificates are to be placed. This location must match the configuration option in your web server. For instance, if certificates are kept in `/var/web/certs`, the definitions will resemble this:

```
DOMAIN_CERT_LOCATION="/var/web/certs/DOMAIN.crt"
DOMAIN_KEY_LOCATION="/var/web/certs/DOMAIN.key"
CA_CERT_LOCATION="/var/web/certs/DOMAIN.com.bundle"

```

You must set the type of test that the ACME protocol will use. These are commented out at the bottom of the configuration file. Using the default values are usually best:

```
SERVER_TYPE="https"
CHECK_REMOTE="true"

```

After these edits are complete, test them by running this:

```
./getssl DOMAIN.com

```

This command resembles the first one, but it does not include the `-c` (create) option. You can repeat this command until you've corrected any errors and are happy with the results.

The default behavior of the `getssl` script is to generate a test certificate that's not really valid. This is done because Let's Encrypt limits the number of actual certificates it will generate for a site to avoid abuse.

Once the configuration files are correct, edit them again and change the server–from the Staging server to the actual Let's Encrypt server:

```
CA="https://acme-v01.api.letsencrypt.org"

```

Then, rerun the `getssl` script one last time with the `-f` option to force it to rebuild and replace the previous files:

```
./getssl -f DOMAIN.com

```

You may need to restart your web server or reboot your system before the new files are recognized.