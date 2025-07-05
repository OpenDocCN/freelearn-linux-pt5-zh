# 容器、虚拟机和云

本章将涵盖以下主题：

+   使用 Linux 容器

+   使用 Docker

+   在 Linux 中使用虚拟机

+   云中的 Linux

# 引言

现代 Linux 应用程序可以部署在专用硬件、容器、虚拟机（VM）或云中。每种解决方案都有其优缺点，且每一种都可以通过脚本和图形用户界面（GUI）进行配置和维护。

如果你想部署多个相同应用程序的副本，其中每个实例都需要自己的数据副本，容器是理想的选择。例如，容器在数据库驱动的 Web 服务器中表现良好，每个服务器需要相同的 Web 基础设施，但拥有私有数据。

然而，容器的缺点是它依赖于宿主系统的内核。你可以在 Linux 宿主机上运行多个 Linux 发行版，但不能在容器中运行 Windows。

如果你需要一个与所有实例不相同的完整环境，使用虚拟机是最好的选择。通过虚拟机，你可以在单一宿主机上运行 Windows 和 Linux。这对于验证测试非常理想，当你不想在办公室里放置一堆计算机时，但又需要测试不同的发行版和操作系统。

虚拟机的缺点是它们非常庞大。每个虚拟机都实现了一个完整的计算机操作系统、设备驱动程序、所有应用程序和工具等。每个 Linux 虚拟机至少需要一个核心和 1 GB 的内存。Windows 虚拟机可能需要两个核心和 4 GB 的内存。如果你希望同时运行多个虚拟机，你需要足够的内存来支持每个虚拟机，否则宿主机会开始交换内存，性能会受到影响。

云就像是有很多计算机和大量带宽触手可及。你实际上可能是在云中运行虚拟机或容器，或者你可能拥有自己的专用系统。

云的最大优势是它可以扩展。如果你认为你的应用可能会突然火爆，或者你的使用是周期性的，能够快速扩展和收缩，而不需要购买或租赁新硬件或新的连接能力是必要的。例如，如果你的系统处理大学注册，它会在每年大约两周内超负荷运作，并且在其余时间几乎处于休眠状态。在那两周里，你可能需要一打硬件，但你并不希望它们在其余时间里空置。

云的缺点是它是你看不见的。所有的维护和配置必须远程完成。

# 使用 Linux 容器

**Linux 容器**（**lxc**）包提供了 Docker 和 LXD 容器部署系统使用的基本容器功能。

Linux 容器使用内核级支持的 **控制组**（**cgroups**）和 第十二章 中描述的 `systemd` 工具，*调整 Linux 系统*。cgroups 支持提供了控制一组程序可用资源的工具。这些工具告诉内核哪些资源可供容器中的进程使用。一个容器可能对设备、网络连接、内存等有有限访问权限。这种控制机制避免了容器之间的相互干扰，或可能对主机系统造成损害。

# 准备工作

默认发行版不提供容器支持。您需要单独安装容器支持。不同发行版的支持程度不一致。**lxc** 容器系统由 Canonical 开发，因此 Ubuntu 发行版提供完整的容器支持。相较于 Debian 8（Jessie），Debian 9（Stretch）在这方面有更好的支持。

Fedora 对 lxc 容器的支持有限。创建特权容器和桥接以太网连接非常简单，但从 Fedora 25 开始，所需的 `cgmanager` 服务已不可用，用于非特权容器的支持。

SuSE 支持有限的 lxc 使用。SuSE 的 `libvirt-lxc` 包类似但不完全等同于 lxc。本章未覆盖 SuSE 的 `libvirt-lxc` 包。创建一个没有以太网的特权容器在 SuSE 下非常简单，但它不支持非特权容器和桥接以太网。

以下是如何在主要发行版上安装 `lxc` 支持。

对于 Ubuntu，请使用以下代码：

```
    # apt-get install lxc1

```

接下来是 Debian。Debian 发行版可能只包含 `/etc/apt/sources.list` 中的安全仓库。如果是这样，您需要将 `deb http://ftp.us.debian.org/debian stretch main contrib` 添加到 `/etc/apt/sources.list` 中，然后执行 `apt-get update`，再加载 `lxc` 包：

```
    # apt-get install lxc

```

对于 OpenSuSE，请使用以下代码：

```
    # zypper install lxc
 RedHat, Fedora:

```

对于基于 Red Hat/Fedora 的系统，添加以下 `Epel` 仓库：

```
    # yum install epel-release

```

完成此操作后，安装 lxc 支持之前，请先安装以下软件包：

```
    # yum install perl libvirt debootstrap

```

`libvirt` 包提供网络支持，`debootstrap` 是运行基于 Debian 的容器所需的：

```
    # yum install lxc lxc-templates tunctl bridge-utils

```

# 如何操作...

`lxc` 包为您的系统添加了多个命令。包括：

+   `lxc-create`：用于创建一个 lxc 容器

+   `lxc-ls`：用于列出可用的容器

+   `lxc-start`：用于启动一个容器

+   `lxc-stop`：用于停止一个容器

+   `lxc-attach`：用于连接到容器的 root shell

+   `lxc-console`：用于连接到容器的登录会话

在基于 Red Hat 的系统上，您可能需要在测试时禁用 SELinux。在 OpenSuSE 系统上，您可能需要禁用 **AppArmor**。禁用 AppArmor 后，您需要通过 `yast2` 重启系统。

Linux 容器有两种基本类型：特权容器和无特权容器。特权容器由 root 用户创建，底层系统具有 root 权限。无特权容器由普通用户创建，仅具有用户权限。

特权容器更易于创建且被更广泛支持，因为它们不需要 `uid` 和 `gid` 映射、设备权限等。然而，如果用户或应用程序成功逃逸容器，他们将拥有宿主机上的完全权限。

创建特权容器是确认系统上所有必要软件包已安装的好方法。创建特权容器后，使用无特权容器来运行应用程序。

# 创建特权容器

开始使用 Linux 容器的最简单方法是下载一个预构建的发行版到一个特权容器中。`lxc-create` 命令创建一个基础容器结构，并可以用预定义的 Linux 发行版填充它。`lxc-create` 命令的语法如下：

```
    lxc-create -n NAME -t TYPE

```

`-n` 选项为容器定义一个名称。此名称将在容器启动、停止或重新配置时用于标识该容器。

`-t` 选项定义用于创建此容器的模板。`download` 类型将你的系统连接到一个预构建容器的仓库，并提示你下载容器。

这是一个实验其他发行版或创建需要不同于主机 Linux 发行版的应用程序的简便方法：

```
    $ sudo lxc-create -t download -n ContainerName

```

下载模板会从互联网上获取可用的预定义容器列表，并从网络档案中填充容器。创建命令提供可用容器的列表，并提示输入**发行版**、**版本**和架构。只有当你的硬件支持此架构时，你才能运行该容器。如果你的系统使用的是 Intel CPU，则无法运行 Arm 容器，但你可以在 64 位 Intel CPU 的系统上运行 32 位 i386 容器：

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

你可以通过选择一个与当前安装匹配的模板，来基于当前的发行版创建容器。模板定义在 `/usr/share/lxc/templates` 中：

```
    # ls /usr/share/lxc/templates
 lxc-busybox   lxc-debian   lxc-download ...

```

要为当前发行版创建容器，选择适当的模板并运行 `lxc-create` 命令。下载过程和安装需要几分钟时间。以下示例跳过了大多数安装和配置消息：

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

前述命令从你的软件包管理器定义的仓库中填充新容器。在使用容器之前，你必须启动它。

# 启动容器

`lxc-start` 命令启动一个容器。与其他 lxc 命令一样，你必须提供要启动的容器名称：

```
    # lxc-start -n ubuntuContainer

```

启动顺序可能会挂起，您可能会看到类似以下的错误。这些错误是由于容器的启动顺序尝试执行图形操作，比如在客户端没有图形支持的情况下显示启动画面所导致的：

```
    <4>init: plymouth-upstart-bridge main process (5) terminated with   
    status 1
 ...

```

您可以等待这些错误超时并忽略它们，或者禁用启动画面。禁用启动画面在不同的发行版和版本中有所不同。文件可能位于`/etc/init`，但这并不一定。

有两种方式可以在容器内工作：

+   `lxc-attach`：此命令直接附加到运行中的容器的 root 账户

+   `lxc-console`：这将打开一个登录会话的控制台，连接到正在运行的容器

容器的第一次使用是直接附加创建用户账户：

```
# lxc-attach -n containerName
root@containerName:/#
root@containerName:/# useradd -d /home/USERNAME -m  USERNAME
root@containerName:/# passwd USERNAME
Enter new UNIX password:
Retype new UNIX password:

```

创建用户账户后，请使用`lxc-console`应用程序以普通用户或 root 身份登录：

```
$ lxc-console -n containerName
Connected to tty 1
Type <Ctrl+a q> to exit the console, 
<Ctrl+a Ctrl+a> to enter Ctrl+a itself
Login:

```

# 停止容器

`lxc-stop`命令用于停止容器：

```
    # lxc-stop -n containerName

```

# 列出已知的容器

`lxc-ls`命令列出当前用户可用的容器名称。此命令不会列出系统中的所有容器，只会列出当前用户拥有的容器：

```
    $ lxc-ls
 container1Name container2Name...

```

# 显示容器信息

`lxc-info`命令显示容器的信息：

```
$ lxc-info -n containerName
Name:   testContainer
State:   STOPPED

```

但此命令仅显示单个容器的信息。通过使用 Shell 循环（如在第一章《Shell Something Out》中所描述的），我们可以显示所有容器的信息：

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

如果容器已停止，则无法获得状态信息。运行中的容器会记录其 CPU、内存、磁盘（块）、I/O 和网络使用情况。此工具允许您监视容器，查看哪些容器最活跃。

# 创建非特权容器

推荐使用非特权容器进行日常操作。如果容器配置不当或应用程序配置不当，可能会导致控制权限从容器中泄露出去。由于容器会调用主机内核中的系统调用，如果容器以 root 身份运行，那么系统调用也将以 root 身份运行。然而，非特权容器以普通用户权限运行，因此更为安全。

要创建非特权容器，主机必须支持 Linux 控制组和 uid 映射。这些支持包含在基础的 Ubuntu 发行版中，但其他发行版需要额外安装。`cgmanager`软件包并非所有发行版都提供。没有该软件包，无法启动非特权容器：

```
    # apt-get install cgmanager uidmap systemd-services

```

启动`cgmanager`：

```
    $ sudo service cgmanager start

```

Debian 系统可能需要启用克隆支持。如果在创建容器时收到`chown`错误，以下行可以解决此问题：

```
    # echo 1 > /sys/fs/cgroup/cpuset/cgroup.clone_children
 # echo 1 > /proc/sys/kernel/unprivileged_userns_clone 

```

允许创建容器的账户的用户名必须包含在`etc`映射表中：

```
    $ sudo usermod --add-subuids 100000-165536 $USER
 $ sudo usermod --add-subgids 100000-165536 $USER
 $ sudo chmod +x $HOME

```

这些命令将用户添加到用户 ID 和组 ID 映射表`(/etc/subuid`和`/etc/subgid`)中，并将 UID 从`100000 -> 165536`分配给用户。

接下来，设置容器的配置文件：

```
    $ mkdir ~/.config/lxc
 $ cp /etc/lxc/default.conf ~/.config/lxc

```

将以下行添加到 `~/.config/lxc/default.conf`：

```
    lxc.id_map = u 0 100000 65536
 lxc.id_map = g 0 100000 65536

```

如果容器支持网络访问，请向 `/etc/lxc/lxc-usernet` 添加一行，以定义可以访问网络桥接器的用户：

```
    USERNAME veth BRIDGENAME COUNT

```

这里，`USERNAME` 是拥有该容器的用户的名称。`veth` 是虚拟以太网设备的常用名称。`BRIDGENAME` 是 `ifconfig` 显示的名称，通常为 `br0` 或 `lxcbro`。`COUNT` 是允许的同时连接数：

```
    $ cat /etc/lxc/lxc-usernet
 clif veth lxcbr0 10

```

# 创建以太网桥接器

容器无法直接访问你的以太网适配器。它需要一个虚拟以太网与实际以太网之间的桥接。最近的 Ubuntu 发行版在安装 lxc 包时会自动创建以太网桥接器。而 Debian 和 Fedora 可能需要你手动创建桥接器。在 Fedora 上创建桥接器时，首先使用 `libvirt` 包创建虚拟桥接器：

```
    # systemctl start libvirtd

```

然后，编辑 `/etc/lxc/default.conf`，将其引用 `virbr0` 而不是 `lxcbr0`：

```
    lxc.network_link = virbr0

```

如果你已经创建了一个容器，也可以编辑该容器的配置文件。

要在 Debian 系统上创建桥接器，你必须编辑网络配置和容器配置文件。

编辑 `/etc/lxc/default.conf`。注释掉默认的空网络，并为 lxc 桥接器添加定义：

```
    # lxc.network.type = empty
 lxc.network.type = veth
 lxc.network.link = lxcbr0
 lxc.network.flage = up`

```

接下来，创建网络桥接器：

```
    # systemctl enable lxc-net
 # systemctl start lxc-net

```

在执行这些步骤后创建的容器将启用网络功能。可以通过向容器的配置文件中添加 `lxc.network` 行来为现有容器添加网络支持。

# 它是如何工作的……

通过 `lxc-create` 命令创建的容器是一个目录树，包含容器的配置选项和根文件系统。特权容器构建在 `/var/lib/lxc` 下。非特权容器存储在 `$HOME/.local/lxc` 下：

```
    $ ls /var/lib/lxc/CONTAINERNAME
 config rootfs

```

你可以通过编辑容器顶层目录中的配置文件来检查或修改容器的配置：

```
    # vim /var/lib/lxc/CONTAINERNAME/config

```

`rootfs` 文件夹包含容器的根文件系统。这是运行中的容器的根（`/`）文件夹：

```
    # ls /var/lib/lxc/CONTAINERNAME/rootfs
 bin   boot cdrom dev  etc   home  lib   media mnt   proc
 root  run  sbin  sys  tmp   usr   var

```

你可以通过添加、删除或修改 `rootfs` 文件夹中的文件来填充容器。例如，要运行 Web 服务，容器可能通过包管理器安装了基本的 Web 服务，并通过将文件复制到 `rootfs` 来安装每个服务的实际数据。

# 使用 Docker

`lxc` 容器复杂，使用起来可能比较困难。这些问题促成了 Docker 包的出现。Docker 使用相同的底层 Linux 功能（如 `namespaces` 和 `cgroups`）来创建轻量级容器。

Docker 仅在 64 位系统上官方支持，因此对于旧系统来说，`lxc` 是更好的选择。

Docker 容器和 lxc 容器之间的主要区别是，Docker 容器通常只运行一个进程，而 lxc 容器运行多个进程。要部署一个数据库支持的 web 服务器，你至少需要两个 Docker 容器——一个用于 web 服务器，一个用于数据库服务器——但只需要一个 lxc 容器。

Docker 的理念使得从更小的构建模块构建系统变得简单，但它也可能使开发模块变得更加困难，因为许多 Linux 工具期望在一个完整的 Linux 系统中运行，并且需要有 `crontab` 条目来执行诸如清理、日志轮换等操作。

一旦 Docker 容器创建完成，它将在其他 Docker 服务器上按照预期运行。这使得在云集群或远程站点部署 Docker 容器变得非常简单。

# 准备就绪

Docker 并没有在大多数发行版中预安装。它是通过 Docker 的软件仓库进行分发的。使用这些仓库需要将新的仓库添加到你的软件包管理器中，并更新校验和。

Docker 为每个发行版和不同版本提供安装说明，相关信息可以在其主页 [`docs.docker.com`](http://docs.docker.com) 查找到。

# 如何操作...

当 Docker 首次安装时，它是没有运行的。你必须使用如下命令启动服务器：

```
    # service docker start

```

Docker 命令有许多子命令提供不同的功能。这些命令将查找 Docker 容器并下载和运行它。以下是关于这些子命令的一些介绍：

+   `# docker search`：这会在 Docker 存档中搜索与关键字匹配的容器

+   `# docker pull`：这会将指定的容器拉取到你的系统中

+   `# docker run`：这会在容器中运行一个应用程序

+   `# docker ps`：这会列出正在运行的 Docker 容器

+   `# docker attach`：这会附加到一个正在运行的容器

+   `# docker stop`：这会停止一个容器

+   `# docker rm`：这会移除一个容器

默认的 Docker 安装要求 `docker` 命令必须以 `root` 用户身份运行，或者使用 `sudo`。

每个命令都有一个 `man` 页面。这个页面的名称由命令和子命令通过一个短横线连接组成。要查看 `docker search` 的 man 页面，使用 `man docker-search`。

下一条食谱展示了如何下载并运行一个 Docker 容器。

# 查找容器

`docker search` 命令返回一个符合搜索条件的 Docker 容器列表：

```
    docker search TERM

```

这里 TERM 是一个字母数字字符串（不允许使用通配符）。搜索命令将返回最多 25 个包含该字符串的容器：

```
# docker search apache
NAME            DESCRIPTION                STARS OFFICIAL   AUTOMATED
eboraas/apache  Apache (with SSL support)  70                   [OK]
bitnami/apache  Bitnami Apache Docker      25                   [OK]
apache/nutch    Apache Nutch               12                   [OK]
apache/marmotta Apache Marmotta             4                   [OK]
lephare/apache  Apache container            3                   [OK]

```

这里的 STARS 表示容器的评分。容器按评分从高到低排序。

# 下载容器

`docker pull` 命令从 Docker 注册表下载一个容器。默认情况下，它从 Docker 的公共注册表 `registry-1.docker.io` 拉取数据。下载的容器会被添加到你的系统中。容器通常存储在 /`var/lib/docker` 下：

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

# 启动 Docker 容器

`docker run` 命令在容器中启动一个进程。通常，这个进程是一个 `bash` shell，允许你附加到容器并启动其他进程。该命令返回一个哈希值，用于定义此会话。

当 Docker 容器启动时，会自动为它创建一个网络连接。

运行命令的语法如下：

```
    docker run [OPTIONS] CONTAINER COMMAND

```

`docker run` 命令支持许多选项，包括：

+   `-t`：分配一个伪 tty（默认值为 false）

+   `-i`：保持交互式会话在未附加的状态下开启

+   `-d`：启动容器时使其分离（在后台运行）

+   `--name`：为此实例指定名称

这个例子启动了之前拉取的容器中的 bash shell：

```
 # docker run -t -i -d --name leph1 lephare/apache  /bin/bash
 1d862d7552bcaadf5311c96d439378617d85593843131ad499...

```

# 列出 Docker 会话

`docker ps` 命令列出当前正在运行的 Docker 会话：

```
# docker ps
CONTAINER ID  IMAGE           COMMAND   CREATED  STATUS  PORTS  NAMES
123456abc     lephare/apache  /bin/bash 10:05    up      80/tcp leph1

```

`-a` 选项将列出系统上所有的 Docker 容器，无论它们是否正在运行。

# 将显示器附加到运行中的 Docker 容器

`docker attach` 命令将你的显示器附加到正在运行的容器中的 `tty` 会话。你需要以 root 用户身份在此容器内运行。

要退出附加的会话，输入`^P^Q`。

这个例子在容器中创建一个 HTML 页面并启动 Apache web 服务器：

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

浏览到 `172.17.0.5` 将显示 `It's Alive` 页面。

# 停止 Docker 会话

`docker stop` 命令终止一个正在运行的 Docker 会话：

```
    # docker stop leph1

```

# 删除 Docker 实例

`docker rm` 命令删除一个容器。删除容器之前，容器必须先停止。可以通过名称或标识符删除容器：

```
    # docker rm leph1

```

或者，你可以使用以下命令：

```
    # docker rm 131aaaeeac79

```

# 工作原理

Docker 容器使用与 `lxc` 容器相同的 `namespace` 和 `cgroup` 内核支持。最初，Docker 是 `lxc` 的一层，但它后来发展成了一个独立的系统。

服务器的主要配置文件存储在 `/var/lib/docker` 和 `/etc/docker` 中。

# 在 Linux 中使用虚拟机

在 Linux 中使用虚拟机有四种选择。三种开源选项是 KVM、XEN 和 VirtualBox。商业上，VMware 提供了一个可以托管在 Linux 上并运行虚拟机的虚拟引擎和执行程序。

VMware 支持虚拟机的时间比任何人都长。它们支持 Unix、Linux、Mac OS X 和 Windows 作为主机系统，并支持 Unix、Linux 和 Windows 作为客体系统。对于商业用途，VMware Player 或 VMware Workstation 是你最好的选择。

KVM 和 VirtualBox 是 Linux 中最流行的虚拟机引擎。KVM 提供更好的性能，但它需要支持虚拟化的 CPU（Intel VT-x）。大多数现代的 Intel 和 AMD CPU 都支持这些功能。VirtualBox 的优势在于它被移植到 Windows 和 Mac OS X，使得你可以轻松地将虚拟机迁移到其他平台。VirtualBox 不需要 VT-x 支持，这使得它适用于旧系统和现代系统。

# 准备就绪

虚拟盒子（VirtualBox）被大多数发行版支持，但可能不是这些发行版的默认软件包仓库的一部分。

要在 Debian 9 上安装 VirtualBox，您需要将 virtualbox.org 仓库添加到 apt-get 将接受的软件包来源中：

```
# vi /etc/apt/sources.list
## ADD:
deb http://download.virtualbox.org/virtualbox/debian stretch contrib

```

安装正确的密钥需要`curl`软件包。如果它还没有安装，请在添加密钥和更新仓库信息之前先安装它：

```
# apt-get install curl
# curl -O https://www.virtualbox.org/download/oracle_vbox_2016.asc
# apt-key add oracle_vbox_2016.asc
# apt-get update

```

一旦仓库更新完毕，您可以通过`apt-get`安装 VirtualBox：

```
# apt-get install virtualbox-5.1

OpenSuSE 
# zypper install gcc make kernel-devel
Open yast2, select Software Management, search for virtualbox.
Select virtualbox, virtualbox-host-kmp-default, and virtualbox-qt.

```

# 如何操作...

安装 VirtualBox 后，它会在开始菜单中创建一个项目。它可能位于系统或应用程序/系统工具下。您可以从终端会话启动 GUI，命令是`virtualbox`或`VirtualBox`。

VirtualBox 的 GUI 使得创建和运行虚拟机变得容易。GUI 的左上角有一个名为“新建”的按钮；它用于创建一个新的空虚拟机。向导会提示您输入诸如内存和磁盘限制等信息。

一旦虚拟机创建完成，启动按钮将被激活。默认设置将虚拟机的 CD-ROM 连接到主机的 CD-ROM。您可以将安装盘放入 CD-ROM 中，然后点击启动以在新虚拟机上安装操作系统。

# 云中的 Linux

使用云服务器的主要原因有两个。服务提供商使用商业云服务，如亚马逊的 AWS，因为它可以在需求增加时轻松扩展资源，在需求减少时降低成本。云存储服务提供商，如 Google Docs，允许用户从任何设备访问他们的数据并与他人共享数据。

OwnCloud 软件包将您的 Linux 服务器转变为一个私有云存储系统。您可以将 OwnCloud 服务器作为私人企业文件共享系统，与朋友共享文件，或者作为手机或平板的远程备份。

OwnCloud 项目于 2016 年分叉。NextCloud 服务器和应用程序预计将使用与 OwnCloud 相同的协议，并且是可以互换的。

# 准备工作

运行 OwnCloud 软件包需要安装**LAMP**（**Linux, Apache, MySQL, PHP**）。这些软件包被所有 Linux 发行版所支持，尽管它们可能不是默认安装的。第十章中讨论了 MySQL 的管理和安装，*管理调用*。

大多数发行版不将 OwnCloud 服务器包含在其仓库中。相反，OwnCloud 项目维护仓库以支持这些发行版。在下载之前，您需要将 OwnCloud 附加到您的 RPM 或 apt 仓库中。

# Ubuntu 16.10

以下步骤将在 Ubuntu 16.10 系统上安装 LAMP 堆栈。类似的命令适用于任何基于 Debian 的系统。不幸的是，软件包名称在不同版本之间有时会有所不同：

```
 apt-get install apache2
 apt-get install mysql-server php-mysql

```

OwnCloud 需要超出默认设置的安全性。`mysql_secure_installation`脚本将正确配置 MySQL：

```
    /usr/bin/mysql_secure_installation

```

配置`OwnCloud`仓库：

```
curl \ https://download.owncloud.org/download/repositories/stable/ \ Ubuntu_16.10/Release.key/'| sudo tee \ /etc/apt/sources.list.d/owncloud.list

apt-get update

```

一旦仓库配置好，apt 将安装并启动服务器：

```
    apt-get install owncloud

```

# OpenSuSE Tumbleweed

使用 **Yast2** 安装 **LAMP** 堆栈。打开 `yast2`，选择软件管理，安装 `apache2`、`mysql` 和 `owncloud-client`。

接下来，选择 `System` 选项卡，并从该选项卡选择 `Services Manager` 选项卡。确认 `mysql` 和 `apache2` 服务已启用并处于活动状态。

这些步骤将安装 OwnCloud 客户端，允许你将工作区与 OwnCloud 服务器同步，并提供服务器的系统要求。

OwnCloud 需要比默认设置更高的安全性。`mysql_secure_installation` 脚本将正确配置 MySQL：

```
    /usr/bin/mysql_secure_installation

```

以下命令将安装并启动 OwnCloud 服务器。前三个命令将配置 `zypper` 以包括 OwnCloud 仓库。一旦这些仓库被添加，OwnCloud 包将像安装其他软件包一样安装：

```
rpm --import https://download.owncloud.org/download/repositories/stable/openSUSE_Leap_42.2/repodata/repomd.xml.key

zypper addrepo http://download.owncloud.org/download/repositories/stable/openSUSE_Leap_42.2/ce:stable.repo

zypper refresh

zypper install owncloud 

```

# 如何操作...

一旦安装了 OwnCloud，你可以配置一个管理员账户，并从那里添加用户账户。NextCloud Android 应用将与 OwnCloud 服务器以及 NextCloud 服务器进行通信。

# 配置 OwnCloud

一旦 `owncloud` 安装完成，你可以通过浏览器访问本地地址来进行配置：

```
    $ konqueror http://127.0.0.1/owncloud

```

初始界面将提示你输入管理员用户名和密码。你可以以用户身份登录，以创建备份并在手机、平板电脑和电脑之间复制文件。

# 还有更多…

我们刚刚讨论的基本安装过程适合测试。如果支持 HTTPS，OwnCloud 和 NextCloud 将使用 HTTPS 会话。启用 HTTPS 支持需要一个 X.509 安全证书。

你可以从众多商业提供商中购买安全证书，或者为自己的使用自签证书，或者通过 **Let's Encrypt**（http://letsencrypt.org）创建一个免费的证书。

自签证书足以用于测试，但大多数浏览器和手机应用会标记此为不受信任的网站。Let's Encrypt 是互联网安全研究小组（ISRG）提供的服务。他们生成的证书是完全注册的，所有应用程序都可以接受它们。

获取证书的第一步是验证你的网站是否符合声明的身份。Let's Encrypt 证书使用名为自动证书管理环境（ACME）的系统进行验证。ACME 系统会在你的网页服务器上创建一个隐藏文件，并告知 **证书颁发机构**（**CA**）该文件的位置，CA 会确认该文件是否存在。这证明你可以访问网页服务器，并且 DNS 记录指向正确的硬件。

如果你使用的是常见的网页服务器，如 Nginx 或 Apache，设置证书的最简单方法是使用 EFF 创建的 `certbot`：

```
    # wget https://dl.eff.org/certbot-auto
 # chmod a+x certbot-auto
 # ./certbot-auto

```

这个机器人将添加新的软件包并将你的新证书安装到正确的位置。

如果你使用的是不太常见的服务器或有非标准安装，`getssl` 包是更具可配置性的。`getssl` 包是一个 bash 脚本，它读取两个配置文件以自动创建证书。可以从此处下载并解压该包：`https://github.com/srvrco/getssl`。

解压 `getssl.zip` 会创建一个名为 `getssl_master` 的文件夹。

生成和安装证书需要三个步骤：

1.  使用 `getssl -c DOMAIN.com` 创建默认配置文件。

1.  编辑配置文件。

1.  创建证书。

首先，通过 `cd` 命令进入 `getssl_master` 文件夹并创建配置文件：

```
    # cd getssl_master
 # getssl -c DOMAIN.com

```

将 `DOMAIN` 替换为你的域名。

这一步会创建 `$HOME/.getssl` 和 `$HOME/.getssl/DOMAIN.com` 文件夹，并在这两个文件夹中创建名为 `getssl.cfg` 的文件。每个文件都必须进行编辑。

编辑 `~/.getssl/getssl.cfg` 文件，并添加你的电子邮件地址：

```
    ACCOUNT_EMAIL='myName@mySite.com'

```

其余字段中的默认值适用于大多数站点。

接下来，编辑 `~/.getssl/DOMAIN.com/getssl.cfg` 文件。此文件中有多个字段需要修改。

主要的变化是设置 Acme Challenge Location (ACL) 字段。ACME 协议将尝试在 [`www.DOMAIN.com/.well-known/acme-challenge`](http://www.DOMAIN.com/.well-known/acme-challenge) 中查找一个文件。ACL 值是该文件夹在系统中的物理位置。如果这些文件夹不存在，你必须创建 `.well-known` 和 `.well-known/acme-challenge` 文件夹并设置其所有权。

如果你的网页保存在 `/var/web/DOMAIN` 中，你可以按以下方式创建新的文件夹：

```
# mkdir /var/web/DOMAIN/.well-known
# mkdir /var/web/DOMAIN/.well-known/acme-challenge
# chown webUser.webGroup /var/web/DOMAIN/.well-known
# chown webUser.webGroup /var/web/DOMAIN/.well-known/acme-challenge

```

ACL 行应类似如下：

```
ACL="/var/web/DOMAIN/.well-known/acme-challenge"
USE_SINGLE_ACL="true"

```

你还必须定义证书将被放置的位置。这个位置必须与 web 服务器中的配置选项匹配。例如，如果证书保存在 `/var/web/certs` 中，定义应类似如下：

```
DOMAIN_CERT_LOCATION="/var/web/certs/DOMAIN.crt"
DOMAIN_KEY_LOCATION="/var/web/certs/DOMAIN.key"
CA_CERT_LOCATION="/var/web/certs/DOMAIN.com.bundle"

```

你必须设置 ACME 协议将使用的测试类型。这些设置已在配置文件的底部注释掉。通常使用默认值是最好的：

```
SERVER_TYPE="https"
CHECK_REMOTE="true"

```

完成这些编辑后，运行以下命令进行测试：

```
./getssl DOMAIN.com

```

这个命令类似于第一个，但不包括 `-c`（创建）选项。你可以重复运行这个命令，直到修正错误并对结果满意为止。

`getssl` 脚本的默认行为是生成一个无效的测试证书。这是因为 Let's Encrypt 限制了每个站点实际证书的生成数量，以防止滥用。

一旦配置文件正确，再次编辑它们并将服务器从 Staging 服务器更改为实际的 Let's Encrypt 服务器：

```
CA="https://acme-v01.api.letsencrypt.org"

```

然后，再次运行 `getssl` 脚本，并使用 `-f` 选项强制它重新构建并替换之前的文件：

```
./getssl -f DOMAIN.com

```

在新的文件被识别之前，你可能需要重启 web 服务器或重启系统。
