# 第九章. 对齐 SELinux 和 DAC

在本章中，我们的重点将放在以下一组方案上：

+   为常规服务分配不同的根目录位置

+   为 SELinux 感知的应用程序使用不同的根目录位置

+   使用文件 ACL 分享用户内容

+   启用多实例化目录

+   配置能力而不是 setuid 二进制文件

+   使用组成员身份进行基于角色的访问控制

+   文件备份与恢复

+   管理应用程序网络访问

# 引言

SELinux 是一种访问控制机制，它与 Linux 提供的常规访问控制一起工作。确保这些不同的访问控制系统能够良好协同是非常重要的，因为它们各自都有优点和用途。

Linux 上的常规 DAC 安全服务已经相当强大，并且几乎每个 Linux 版本发布时都会进一步扩展。命名空间、扩展的访问控制、额外的 **chroot** 限制以及其他服务都被加入到 Linux 生态系统中，进一步支持 Linux 系统的加固。

在加固系统的过程中，SELinux 只是另一个防御层。仅仅将所有精力集中在 SELinux 上是一个重大的错误，因为 SELinux 也有其缺点。通过正确启用 Linux DAC 控制并调整 SELinux，使其与这些控制系统良好配合，可以使 Linux 系统在应对漏洞和攻击时更加韧性。

# 为常规服务分配不同的根目录位置

不同的根目录位置，也称为 chroot，是 Linux 系统的一项重要功能，旨在禁止直接访问指定目录位置之外的文件资源。从 chroot 中可以访问的环境称为 **jail** 或 **chroot jail**。在 chroot jail 中运行的应用程序会在不同的根目录下启动，其中仅托管应用程序正常运行所需的文件。

尽管通常被视为一种安全功能，但 chroot 本身并非出于这个目的。然而，通过正确的方法，chroot 可以增强应用程序的安全配置。

例如，在出现漏洞的情况下，成功的利用可能只能访问 chroot 中可用的文件。其他敏感文件，例如与身份验证相关的文件或其他服务配置文件，在 chroot 内是无法访问的（假设被利用的应用程序没有越狱 chroot 限制的权限）。

设置任何服务的 chroot 环境的步骤是类似的，但 chroot 的最终结果永远不会相同：根据被限制的应用程序，不同的文件需要在 chroot 中可用。

## 准备工作

查找需要被限制的应用程序。这些应用程序必须是最终服务，因为这些应用程序与其他应用程序或服务几乎没有交互。否则，所有这些其他应用程序和服务也需要在相同的 chroot 中可用。

通常，主要的目标是那些在互联网上使用非常广泛的服务。这些服务的漏洞通常会更积极地被搜索和开发，当发现漏洞并开发出利用代码时，恶意用户或团体会迅速扫描互联网寻找易受攻击的版本进行攻击。

## 如何操作……

下一组步骤展示了如何设置 chroot 环境并通知 SELinux 关于 chroot 的信息。我们以 BIND DNS 服务器作为示例服务，并将 `/var/chroot/` 作为 chroot 位置：

1.  创建 chroot 位置并添加必要的子目录：

    ```
    ~# mkdir -p /var/chroot/dev
    ~# mkdir -p /var/chroot/etc/bind
    ~# mkdir -p /var/chroot/var/bind/{sec,pri,dyn}
    ~# mkdir -p /var/chroot/var/{log,run}
    ~# chown root:named /var/chroot
    ~# chmod 750 /var/chroot
    ~# chown -R named:named /var/chroot/var/*

    ```

1.  复制应用程序所需的所有文件：

    ```
    ~# cp /etc/named.conf /var/chroot/etc/
    ~# cp /etc/localtime /var/chroot/etc/
    ~# cp -a /var/named/* /var/chroot/var/named/

    ```

1.  创建应用程序需要的设备文件：

    ```
    ~# mknod /var/chroot/dev/null c 1 3
    ~# mknod /var/chroot/dev/random c 1 8
    ~# chmod 666 /var/chroot/dev/*

    ```

1.  由于 BIND 服务支持 chroot，我们不需要将其二进制文件和库复制到 chroot 位置。但是，并非所有服务都原生支持 chroot。当遇到这种情况时，我们还需要复制二进制文件和库。

1.  现在，重新标记 chroot 中的文件，以便它们获得正确的 SELinux 标签：

    ```
    ~# setfiles -r /var/chroot/ /etc/selinux/mcs/contexts/files/file_contexts /var/chroot/

    ```

1.  使用适当的选项启动应用程序以启用 chroot 支持。一些 Linux 发行版已经为 BIND 服务支持 chroot 信息。通常，它要求通过 `-t /var/chroot/` 选项启动 `named` 应用程序。如果应用程序不原生支持 chroot，请使用 `chroot` 命令本身：

    ```
    ~# chroot /var/chroot/ su - named -c /usr/sbin/named

    ```

1.  如果应用程序原生支持 chroot，它可能需要 `chroot` 功能。这可以通过以下 SELinux 策略接口中的 `sys_chroot` 权限来支持：

    ```
    corecmd_exec_chroot(named_t)
    ```

## 它是如何工作的……

设置 chroot 环境通常是一个反复试验的过程；不过，对于更受欢迎的服务，互联网上有许多教程可以让设置 chroot 更加简单。

使用的基本方法是四个步骤：

1.  创建 chroot 位置和目录结构。

1.  安装必要的文件，并在必要时安装应用程序的二进制文件和库。

1.  更新资源的 SELinux 标签。

1.  调用 chroot 二进制文件或使用应用程序内置的 `chroot` 功能。

创建 chroot 位置时，我们需要确保其结构类似于真实的根目录位置（即`/`位置）；至于应用程序，它会将该 chroot 位置视为整个文件系统。

安装哪些文件是另一个问题，拥有在线资源来告知我们该怎么做是很大的帮助。但是，如果这些在线资源缺失，我们仍然可以找到哪些文件是必需的。

例如，我们可以使用 `ldd` 或 `scanelf` 应用程序：

```
~# ldd /usr/sbin/named
 linux-vdso.so.1
 liblwres.so.90 => /usr/lib64/liblwres.so.90
 libdns.so.100 => /usr/lib64/libdns.so.100
 libbind9.so.90 => /usr/lib64/libbind9.so.90
 libisccfg.so.90 => /usr/lib64/libisccfg.so.90
 libisccc.so.90 => /usr/lib64/libisccc.so.90
 libisc.so.95 => /usr/lib64/libisc.so.95
 libc.so.6 => /lib64/libc.so.6
 /lib64/ld-linux-x86-64.so.2

```

但通常，反复试验的方法最有效。只需在 chroot 中启动应用程序，记录其错误并解决它们。

对于 SELinux，这里需要注意的是 chroot 应该正确标记。例如，考虑 `/var/chroot/etc/named.conf`。SELinux 策略将假定该文件标记为 `named_conf_t`。然而，文件本身的路径（`/var/chroot/etc/named.conf`）暗示着它的标签为 `var_t`，因为 `/var/` 的标签是 `var_t`，并且没有针对我们所定义的任何子目录或文件的标签定义。

`setfiles` 命令允许我们通过不同的根位置重新标记位置，这样 `/var/chroot/etc/named.conf` 就会被标记为与 `/etc/named.conf` 相同的标签。然而，注意系统的重新标记操作之后，需要再次使用 `setfiles` 命令，因为 SELinux 配置并未意识到标签的改变。

最后，应用程序本身需要在 chroot 内部启动或通过其内建的 chroot 支持启动。支持 chroot 的应用程序可以通过其配置文件和启动选项进行调整，以确保它们在 chroot 环境中运行。如果这不可行，则应该通过一个 `init` 脚本启动该应用程序，该脚本调用 `chroot` 命令，并很可能与 `su` 应用程序一起使用，以允许切换到其他用户。

## 还有更多...

chroot 是一种相对原始但强大的减少漏洞影响的方法。然而，确实存在逃逸 chroot 的方法。幸运的是，存在一些内核补丁，能显著提高 chroot 的安全性。一个流行的更新是由 **grsecurity** 团队维护的 ([`www.grsecurity.net`](http://www.grsecurity.net))。

使用 grsecurity 的 chroot 限制，内核可以通过以下选项进行配置：

+   禁止在 chroot 内部发起文件系统的挂载和重新挂载操作

+   禁止在 chroot 内部执行 chroot 操作

+   禁止在 chroot 内部执行 `pivot_root` 调用

+   强制 chroot 应用程序的当前工作目录为 chroot 根目录

+   禁止在 chroot 内部执行 `setuid` 和 `setgid chmod` 操作

+   禁止通过指向 chroot 外部的打开文件描述符来更改目录

+   禁止连接到 chroot 外部创建的共享内存

+   禁止访问 chroot 外部创建的 Unix 域套接字

+   禁止向 chroot 外部的进程发送信号

除了这些选项，还有许多其他选项。这些选项使得 chroot 监狱比最初预期的更加注重安全性，并且为防范漏洞提供了非常强大的缓解措施。

## 另见

关于 chroot 监狱和特别是 BIND chroot 的资源有很多：

+   在 [`www.unixwiz.net/techtips/bind9-chroot.html`](http://www.unixwiz.net/techtips/bind9-chroot.html) 上，关于在 chroot 监狱中构建和配置 BIND 9 的内容有很详细的讲解，并且提供了指向其他 BIND 相关资源的链接

+   在同一站点，可以找到 Unix `chroot()` 操作的最佳实践：[`www.unixwiz.net/techtips/chroot-practices.html`](http://www.unixwiz.net/techtips/chroot-practices.html)

+   Jailkit 项目（[`olivier.sessink.nl/jailkit/`](http://olivier.sessink.nl/jailkit/)）提供了一套用于管理 chroot 监狱的实用工具

# 为支持 SELinux 的应用程序使用不同的根位置

支持 SELinux 的应用程序在运行于 chroot 位置时有更多的需求。它们需要访问 SELinux 子系统（从 chroot 内部）以及可能需要的 SELinux 配置条目。这包括启用 PAM 的服务，因为这些服务的用户登录可能需要访问 SELinux 用户配置文件（如 `seusers` 文件和默认上下文）。

## 如何操作...

首先，像之前一样创建常规的 chroot 位置。为了更新系统以支持在 chroot 内运行的支持 SELinux 的应用程序，完成以下步骤：

1.  在 chroot 中挂载 SELinux 文件系统到 `/sys/fs/selinux/`，以便应用程序可以查询 SELinux 策略：

    ```
    ~# mkdir -p /var/chroot/sys/fs/selinux
    ~# mount -t selinuxfs none /var/chroot/sys/fs/selinux

    ```

1.  可选地，创建 `/var/chroot/etc/selinux/` 位置并将当前定义复制到其中：

    ```
    ~# cp -a /etc/selinux/ /var/chroot/etc/

    ```

1.  更新 `seusers` 文件（在 `/var/chroot/etc/selinux/mcs/`）以仅包含在 chroot 内所需的 SELinux 用户映射。

## 它是如何工作的...

支持 SELinux 的应用程序通常需要访问 SELinux 文件系统（`/sys/fs/selinux/`）和内核提供的伪文件系统，以便与 SELinux 子系统进行交互。这通常被视为一种更危险的情况，因为这通常会让应用程序以更高权限的用户身份运行，并访问不再受到 chroot 保护的系统资源。这会降低 chroot 监狱作为安全措施的有效性。

如果应用程序本身不支持 chroot，我们将不得不将 `/sys/fs/selinux/` 文件系统暴露给被 chroot 的应用程序。如果应用程序默认支持 chroot，它可能会在咨询 SELinux 后调用 chroot（即从非 chroot 父进程）并在 chroot 内运行工作进程或用户进程。这是 OpenSSH 支持的 chroot SFTP 用户的情况。

也可能只需将 SELinux 文件系统挂载到 `/selinux/`（一个已弃用但仍受支持的 SELinux 文件系统位置）内的 chroot 中。这样，就不需要创建虚假的 `/sys/fs/` 位置：

```
~# mount -t selinuxfs none /var/chroot/selinux

```

`/etc/selinux/` 位置并非总是需要的，因此默认情况下不应在 chroot 内部提供访问权限。但是，支持 SELinux 的应用程序，如果使用 SELinux 用户和角色转换，或者主动修改文件上下文，则需要能够读取 `/etc/selinux/` 内的文件。

根据 chroot 监狱的目的，可能也可以使用 `/etc/selinux/` 位置的只读绑定挂载：

```
~# mount -o bind /etc/selinux /var/chroot/etc/selinux
~# mount -o remount,ro /var/chroot/etc/selinux

```

之后需要重新挂载，以将其标记为只读。绑定挂载本身不允许传递额外的挂载选项，因此我们无法立即使用 `ro` 挂载选项进行挂载。当然，使用只读绑定挂载后，已经无法/不需要再修改 `seusers` 文件。

## 另见

+   关于 SFTP chroot 的详细指南可以在 [`wiki.archlinux.org/index.php/SFTP_chroot`](https://wiki.archlinux.org/index.php/SFTP_chroot) 和 [`en.wikibooks.org/wiki/OpenSSH/Cookbook/SFTP`](http://en.wikibooks.org/wiki/OpenSSH/Cookbook/SFTP) 找到

# 使用文件 ACL 共享用户内容

访问控制列表允许对文件进行更细粒度的访问控制。与使用公共组所有权不同，文件的访问可以单独授予用户或用户组。

然而，SELinux 启用的访问控制也应根据这种情况进行调整。SELinux 中的基于用户的访问控制限制等特性，可能会完全阻止共享用户内容的访问，无论文件上设置了什么 ACL。

## 如何操作……

假设某用户希望允许一组文件和目录的读取及读写访问，可以使用以下步骤：

1.  在用户的主目录外创建一个可访问的位置：

    ```
    ~# mkdir -p /home/share/
    ~# chmod 1777 /home/share/

    ```

1.  创建一个可以用于共享资源的 SELinux 文件类型：

    ```
    type user_share_t;
    files_type(user_share_t)
    ```

1.  创建一个允许用户管理资源的界面：

    ```
    interface(`userdom_admin_user_share','
      gen_require(`
        type user_share_t;
      ')
      admin_pattern($1, user_share_t)
     ')
    ```

1.  将该类型分配给新位置：

    ```
    ~# semanage fcontext -a -t user_share_t "/home/share(/.*)?"
    ~# restorecon -R /home/share/

    ```

1.  将该界面分配给将参与此资源共享开发的用户域：

    ```
    userdom_admin_user_share(user_t)
    ```

1.  将需要共享的文件移到用户的主目录外，因为主目录的 SELinux 上下文不允许在其中共享资源。

    ```
    ~$ cp -r sharedfiles/ /home/share && rm -r sharedfiles/

    ```

1.  分配允许（有限用户集）用户进行适当访问的 ACL：

    ```
    ~$ setfacl -R -m u:user1:rX /home/share/sharedfiles
    ~$ setfacl -R -m u:user2:rwX /home/share/sharedfiles
    ~$ setfacl -m "default:u:user2:rwX" /home/share/sharedfiles
    ~$ setfacl -m "default:u:user0:rwX" /home/share/sharedfiles
    ~$ setfacl -m "default:u:user1:rX" /home/share/sharedfiles

    ```

## 它是如何工作的……

文件级访问控制可以与 SELinux 访问控制完美配合使用。然而，需要特别小心，确保这两种控制机制（文件 ACL 和 SELinux 策略）不会相互干扰。SELinux 可能会禁止一些预期中的访问（例如，由于 SELinux 限制而不是类型强制设置），但同时也需要正确管理文件访问控制，以保持系统行为的一致性。

在这些操作中，共享的文件被移到用户的主目录之外。这主要是因为 SELinux 的 UBAC 特性，禁止不同 SELinux 用户访问彼此的常规资源（例如标记为 `user_home_t` 的资源，也包括 `user_home_dir_t`）。由于在 UBAC 限制下，`user_home_dir_t` 不允许其他 SELinux 用户访问，因此映射到不同 SELinux 用户的用户将无法进入并浏览共享用户的主目录，不论是否安装了 ACL。

不是所有系统都启用了 UBAC，或者共享可能仅限于单个 SELinux 用户，因此这种方法并不总是必要的。不过，使用不同的位置可以实现更好的管理。考虑以下情况：当第一个用户离开公司时，但他的团队希望继续访问和管理共享资源。如果删除用户的主目录，这些资源将消失。

将文件移动到不同的位置后，下一步是将文件标记为所有用户都可以访问的文件类型，但该类型不会受到 UBAC 特性限制。具有 `ubac_constrained_type` 属性的文件类型不能用于共享，因此创建一个新的文件类型，将其标记为常规文件。然后，用户域被授予此类型的管理权限（不仅可以管理文件，还可以将文件重新标记为 `user_share_t` 类型或从其标记为 `user_share_t` 类型）。这确保了 SELinux 不会阻止对共享资源的访问，同时仍然防止未经授权的域访问这些资源。

也可以选择一个已经被用户访问的文件类型，例如 `nfs_t` 类型（前提是 SELinux 布尔值 `use_nfs_home_dirs` 已设置）。然而，分配一个功能上用于不同目的的类型（`nfs_t` 用于 NFS 挂载的文件系统）可能会导致其他域也能访问这些资源。因此，管理员需要仔细考虑每个选择的理由和后果。

在 `/home/share/` 位置标记为 `user_share_t` 类型后，原始用户将资源复制到新位置，并从当前目录中删除它们。这种方法（复制并删除）用于确保资源继承目标位置的标签（`user_share_t`），而不是保持与原始文件位置（`user_home_t`）相关联的标签，这在使用 `mv` 命令移动文件时会发生。最近的 `coreutils` 包提供了对 `mv -Z` 的支持，可以直接移动资源，同时仍然为资源提供正确的上下文。

用户的第三种方法是先移动资源，然后重新标记它们：

```
~$ mv sharedfiles/ /home/share/
~$ chcon -R -t user_share_t /home/share/sharedfiles/

```

最后，在所有 SELinux 规则和支持到位后，文件访问控制将在共享资源上启用，并且启用默认的 ACL，以便其他用户的写操作会自动继承写入资源上的正确 ACL，确保所有在共享资源上合作的用户无需不断地设置文件的 ACL。

如果没有默认的 ACL，其他用户可能会在 `sharedfiles/` 中创建没有设置 ACL 的文件，从而阻止其他用户访问这些资源。

## 还有更多...

另一种可行的方法是使用 `setgid` 组所有权。例如，如果所有参与共享文件访问的用户都在 `shrgrp` 组中，则以下设置将自动使在该目录中创建的所有文件都具有 `shrgrp` 组所有权：

```
~$ chgrp -R shrgrp /home/share/sharedfiles/
~$ find /home/share/sharedfiles/ -type d -exec chmod g+s '{}' \;

```

这要求用户具有正确的 `umask` 设置（例如 `007` 或更低），以便新创建资源的组权限允许组成员读取和写入。

# 启用多实例目录

在 Linux 和 Unix 系统中，`/tmp/` 和 `/var/tmp/` 位置是所有用户可写的。它们用于提供一个公共的临时文件位置，并通过粘滞位进行保护，以防止用户删除他们不拥有的文件，尽管该目录是世界可写的。

尽管采取了这种措施，但 `/tmp/` 和 `/var/tmp/` 位置仍然存在攻击历史，例如符号链接的竞态条件以及通过（临时或非临时）世界或组可读文件泄露信息。

多实例目录为此问题提供了一个简洁的解决方案：用户获得自己的、私有的 `/tmp/` 和 `/var/tmp/` 实例。这些目录实例在登录时会在不同的位置创建，然后在该特定用户会话中将其显示（挂载）到 `/tmp/` 和 `/var/tmp/` 位置。通过使用 Linux 命名空间，这种挂载对用户会话是本地的——其他用户有他们自己对挂载的视图，对于管理员来说，多实例化未启用，因此他们保持系统的全局视图。

## 如何操作……

要启用 `/tmp/` 和 `/var/tmp/` 的多实例化，请遵循以下步骤：

1.  创建 `/tmp-inst/` 和 `/var/tmp/tmp-inst/` 位置：

    ```
    ~# mkdir /tmp-inst/ /var/tmp/tmp-inst/
    ~# chmod 000 /tmp-inst/ /var/tmp/tmp-inst/

    ```

1.  为这些位置设置标签为 `tmp_t`：

    ```
    ~# semanage fcontext -a -t tmp_t -f d /tmp-inst
    ~# semanage fcontext -a -t tmp_t -f d /var/tmp/tmp-inst

    ```

1.  编辑 `/etc/security/namespace.conf` 并添加以下定义：

    ```
    /tmp  /tmp-inst/    level  root,adm
    /var/tmp  /var/tmp/tmp-inst/  level  root,adm
    ```

1.  编辑用于登录的 PAM 配置文件，例如 `system-login`，并在 `pam_selinux.so` 后面的 session 组中添加以下行：

    ```
    session  required  pam_namespace.so
    ```

1.  启用 `allow_polyinstantiation` SELinux 布尔值：

    ```
    ~# setsebool -P allow_polyinstantiation on

    ```

## 工作原理……

多实例目录的系统准备工作要求这些目录本身可用并具有正确的权限设置。当父目录（如 `/tmp/`）是 tmpfs 挂载时，我们不能在其中创建多实例目录（例如 `/tmp/tmp-inst/`），因为该目录在重启后会消失（除非通过 `init` 脚本添加）；因此，将 `/tmp-inst/` 设置为单独的位置是必要的。当然，管理员仍然可以选择将该位置本身设置为 tmpfs 挂载——重要的是该目录必须存在并具有正确的权限（由 `000` 权限集表示）。

在示例中，假设 `/var/tmp/` 不是一个 tmpfs 挂载，因此我们可以在其中定义多实例目录。

多实例化目录的配置文件是位于`/etc/security/`下的`namespace.conf`文件。在其中，挂载点与创建多实例化目录的目录一起列出：

```
/tmp  /tmp-inst/  level  root,adm
```

第三列定义了多实例化的方法。在非 SELinux 系统中，最常用的方法是`user`方法，它基于用户名创建目录。在启用 SELinux 的系统中，方法必须是`level`或`context`。

在`level`方法的情况下，目录是基于用户名和用户会话的 MLS 级别创建的。`context`方法则是基于用户名和安全上下文创建目录。这使得可以根据用户的角色隐藏临时数据，从而减少意外数据泄露的可能性。

管理员可以访问多实例化目录，因为它们被排除在多实例化之外：排除的用户列表在`namespace.conf`文件的第四列中配置。管理员仍然可以看到动态创建的目录：

```
~# ls -l /tmp-inst/
drwxrwxrwt. 2 root root 4096 Jun 22 12:31 system_u:object_r:tmp_t:s0_user1
drwxrwxrwt. 2 root root 4096 Jun 22 12:30 system_u:object_r:tmp_t:s0_user2

```

接下来，修改 PAM 配置文件以启用`pam_namespace.so`库。要找到需要编辑的 PAM 配置文件，请查找调用`pam_selinux.so`的 PAM 配置文件：

```
~# cd /etc/pam.d
~# grep -l pam_selinux.so *
system-login

```

在此示例中，`system-login` PAM 配置文件是唯一调用`pam_selinux.so`的文件，因此在该文件中添加了`pam_namespace.so`行。该行必须添加在`pam_selinux.so`调用之后，因为`pam_namespace.so`文件使用用户的上下文来决定如何调用实例化的目录。如果`pam_selinux.so`尚未被调用，则无法获取此信息，导致登录失败。

最后，启用 SELinux 布尔值`allow_polyinstantiation`，使得适当的域具有创建（并更改上下文）适当目录、使用命名空间、检查用户上下文等的权限。

## 还有更多...

管理员可以进一步操作，不仅仅是在需要时创建目录。在多实例化目录的设置过程中，会调用名为`namespace.init`的脚本，该脚本位于`/etc/security/`目录下，用于进一步处理这些目录的创建和修改。

此脚本可以调整为将文件复制到实例化目录（该文件通常已经包含多实例化主目录的逻辑），或进行其他更改，进一步调整用户会话的设置。

`systemd init`系统也支持通过`PrivateTmp`指令为服务提供私有的`/tmp/`目录，而不是为最终用户提供。

# 配置能力而非 setuid 二进制文件

Linux 能力允许在用户和应用层面上进行粗粒度的内核安全授权。在能力机制出现之前，管理员只能通过 `setuid` 应用程序为用户授予额外的权限：这些应用程序在执行时会继承应用程序所有者的权限（通常是 `root`）。有了能力机制后，权限集合可以进一步限制。

例如，可以为 `ping` 应用程序授予 `cap_net_raw` 能力，这样它就不再需要 `setuid`。根据不同的配置，可能需要为用户授予使用该能力的权限（如果应用程序已设置了相应的标志），或者立即授予该能力（不考虑用户设置）。

## 如何操作……

要与 SELinux 一起使用能力，请执行以下步骤：

1.  启用应用程序所需的能力，作用于应用程序二进制文件：

    ```
    ~# setcap cap_net_raw+ei /bin/ping

    ```

1.  对于允许使用 `net_raw` 能力的用户，在 `/etc/security/capability.conf` 中添加适当的配置（每个用户一行）：

    ```
    cap_net_raw   user1
    ```

1.  将使用该能力的 SELinux 域需要获得使用该能力的权限。对于常见的应用程序，通常这些权限已经设置好。

    ```
    allow ping_t self:capability net_raw;
    ```

1.  被允许修改其进程能力集的 SELinux 域，必须具有 `setcap` 权限：

    ```
    allow local_login_t self:process setcap;
    ```

1.  编辑 PAM 配置文件，通过相应的服务来允许使用能力，并在 `auth` 配置块中添加以下行：

    ```
    auth  required  pam_cap.so
    ```

1.  如果需要追踪/审计能力，可以使用 SELinux 的 `auditallow` 语句：

    ```
    auditallow domain self:capability net_raw;
    ```

## 它是如何工作的……

当前进程允许使用的能力称为已许可的能力。处于活动状态的能力是有效的能力。第三种能力集合是可继承的能力。

在示例中，我们为 `ping` 应用程序启用了 `cap_net_raw` 能力，并标记为如果继承时生效。换句话说，默认情况下它并不会启用（允许）。如果我们希望立即启用 `cap_net_raw` 能力，则需要使用有效且已许可的权限集：

```
~# setcap cap_net_raw+ep /bin/ping

```

支持能力的应用程序不需要设置 `effective` 位。它们将在需要时通过适当的系统调用来启用（和丢弃）这些能力（这就是为什么这些领域需要 `setcap` 权限的原因）。如果 `ping` 支持能力，那么以下设置对我们的示例就足够了：

```
~# setcap cap_net_raw+i /bin/ping

```

接下来，允许使用 `cap_net_raw` 能力的用户（通过所选应用程序）需要在其继承的能力集中获得 `cap_net_raw` 能力。这可以通过 `/etc/security/` 中的 `capability.conf` 文件以及通过在正确的 PAM 配置文件中调用 `pam_cap.so` 模块来实现。使用 PAM 配置文件还允许我们根据用户登录的服务来区分能力。

要检查当前启用的能力，用户可以执行`capsh`应用：

```
~$ /sbin/capsh --print | grep ^Current
Current: cap_net_raw+i

```

要查看文件上的能力，可以使用`getcap`应用：

```
~$ getcap /bin/ping
/bin/ping = cap_net_raw+ei

```

最后，通过`auditallow`语句审计能力的使用情况，可以告诉我们何时（以及由谁）使用了某个能力，尽管同样的事情可以通过 Linux 审计子系统在没有 SELinux 策略的情况下完成，审计`setcap`系统调用。

## 另请参阅

+   能力在 Chris Friedhoff 的**POSIX 能力与文件 POSIX 能力**页面中得到了很好的解释（[`www.friedhoff.org/posixfilecaps.html`](http://www.friedhoff.org/posixfilecaps.html)）

# 使用组成员关系进行基于角色的访问控制

在更大的环境中，访问控制通常基于组成员关系授予。与单独的权限管理相比，组成员关系更容易管理：只需将用户添加或移出组即可自动授予或撤销权限，管理员可以轻松地根据组成员关系找出用户将拥有哪些权限。

## 如何操作…

为了使用组成员关系作为分配权限的高级方法，管理员需要注意以下方面：

1.  将用户添加到他们应属于的组中：

    ```
    ~# gpasswd -a user1 dba
    ~# gpasswd -a user1 dev

    ```

1.  将适当的 SELinux 用户分配给该组：

    ```
    ~# semanage login -s dbadm_u %dba

    ```

1.  限制仅应由特定组调用的二进制文件和库：

    ```
    ~# chgrp -R dev /usr/lib/gcc /usr/x86_64-pc-linux-gnu/gcc-bin
    ~# chmod -R o-rx /usr/lib/gcc /usr/x86_64-pc-linux-gnu/gcc-bin

    ```

1.  在`sudoers`文件中使用组表示法将特定权限授予组成员：

    ```
    %dba  ALL=(ALL)  TYPE=dbadm_t ROLE=dbadm_r NOPASSWD: initdb
    ```

## 它是如何工作的…

使用组使得权限管理变得更加简单。最终，这使得管理员只需管理用户的组成员关系，并根据组自动分配权限。

我们可以为组授予一个 SELinux 用户，并通过组成员关系决定普通用户登录的 SELinux 用户。当然，用户可以属于多个组。对于 SELinux 来说，是`seusers`文件的顺序决定以下映射的使用：

+   单个用户的 SELinux 用户映射优先于组映射

+   如果没有为该用户定义单独的 SELinux 用户映射，那么在`seusers`文件中第一个使用该 Linux 用户所属组的组映射将决定 SELinux 用户映射。

因此，如果一个用户是两个组的成员（例如，`dba`和`web`），并且这两个组分别有映射到`dbadm_u`（针对`dba`组）和`webadm_u`（针对`web`组），那么`seusers`文件中第一个映射将决定该用户的 SELinux 用户。

为了覆盖此行为，您可以单独添加用户，或创建另一个组（比如`dbaweb`），将该用户也加入该组，并将该组映射放在`seusers`文件中的最前面。

当只有特定用户组被允许访问某个应用程序，但该应用程序不使用任何特定的 SELinux 域时，管理员使用 Linux DAC 权限来限制对该应用程序的访问可能更灵活。通过仅允许特定组（在我们的例子中是 `dev`）对应用程序及其应用程序库具有读取和执行权限，我们可以轻松限制访问。

另一种方法是为文件贴上新的 SELinux 类型标签，并授予适当的域对这些类型的访问权限。然而，这可能导致需要访问这些类型的大量域（因此需要大量的策略开发工作），而 Linux DAC 方法则容易实现。

# 备份和恢复文件

系统可用性和服务安全性的一个重要方面是提供备份和恢复服务。对于许多人来说，拥有文件副本可能看起来足够作为备份方法。然而，备份内容应包含的不仅仅是文件的内容。

## 如何操作…

选择备份解决方案时，请确保检查以下内容：

1.  应该备份文件的扩展属性选择（而不仅仅是 `security.selinux` 属性）。

1.  当文件恢复到原始位置时，SELinux 上下文也应该一并恢复。如果备份解决方案不支持 SELinux 上下文，则应在恢复文件后，运行 `restorecon` 命令来恢复 SELinux 上下文。

1.  当文件恢复到临时区域时，SELinux 上下文不应恢复。相反，管理员应将文件放回原位，然后再恢复上下文。

1.  `/etc/selinux/` 目录中的 SELinux 配置应该进行备份，即使没有使用完整系统备份。每当策略或文件上下文定义发生更改时，备份文件时也应将这些内容一并备份。

## 它是如何工作的…

文件标签存储为 `security.selinux` 扩展属性。由于策略的运行基于所有相关对象的标签，若不备份和恢复文件标签，可能会影响系统在恢复操作后的正常运行。

当备份解决方案不支持扩展属性时，必须通过 `semanage fcontext` 命令正确设置所有标签。这是确保在恢复后，管理员能够运行 `restorecon` 命令来重置文件标签的唯一方法：

```
~# tar xvf /path/to/last_backup.tar.gz etc/named.conf
~# restorecon /etc/named.conf

```

然而，强烈建议选择支持扩展属性的备份解决方案，因为许多与 Linux 相关的设置都是以扩展属性的形式存储的。例如，文件 ACL 也是以扩展属性存储的：

```
~$ getfattr -m . -d named.conf
# file: named.conf
security.selinux="system_u:object_r:named_conf_t:s0"
system.posix_acl_access=0sAgAAAAEABgD/////AgAGAOo…

```

系统中可以使用的其他扩展属性包括 PaX 标记（`user.pax.flags`）、IMA 和 EVM 哈希（`security.ima`和`security.evm`）以及能力（`security.capability`）。但这里也存在问题：某些属性不应该（或不能）恢复。例如，IMA 和 EVM 属性由 Linux 内核处理，无法通过用户工具进行操作。

除了文件标签之外，SELinux 策略的备份和恢复也应该集成，尤其是在修改过 SELinux 策略的系统上。如果恢复后策略发生变化，可能会缺少类型或标签失效。

# 管理应用程序网络访问

在 Linux 系统上，`iptables`（以及最近的`nftables`）是事实上的主机级防火墙技术。管理员无疑会使用它来防止未授权的系统访问服务。我们还可以使用`iptables`来标识和标记网络数据包，只允许授权的应用程序（域）发送或接收这些网络数据包。

默认情况下，SELinux 策略支持客户端和服务器数据包，并允许常见的域访问其客户端和/或服务器数据包。例如，Web 服务器域（如`httpd_t`）将具有发送和接收`http_server_packet_t`数据包的权限：

```
allow httpd_t http_server_packet_t:packet { send recv };
```

这通过`corenet_sendrecv_http_server_packets`接口提供。启用数据包标签可以通过`iptables`轻松完成，如本食谱所示。但为了正确管理网络访问，需要创建自定义数据包类型，确保不使用任何默认的允许访问。

## 如何实现…

仅允许授权的域访问特定的网络数据包（数据报和数据流），请使用以下方法：

1.  确定需要允许的流量。例如，我们可能只希望来自`10.11.12.0/24`的 DNS 请求被`dnsmasq_t`域接受，而来自`10.13.14.0/24`的请求则被`named_t`域接受。

1.  创建两个新的数据包类型：

    ```
    type dnsmasq_server_packet_t;
    corenet_server_packet(dnsmasq_server_packet_t)

    type named_server_packet_t;
    corenet_server_packet(named_server_packet_t)
    ```

1.  允许这些数据包的域发送和接收权限：

    ```
    allow dnsmasq_t dnsmasq_server_packet_t:packet { send recv };
    allow named_t named_server_packet_t:packet { send recv };
    ```

1.  根据情况标记传入流量：

    ```
    ~# iptables -t mangle -A INPUT -p tcp -s 10.11.12.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:dnsmasq_server_packet_t:s0"
    ~# iptables -t mangle -A INPUT -p udp  -s 10.11.12.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:dnsmasq_server_packet_t:s0"
    ~# iptables -t mangle -A INPUT -p tcp -s 10.13.14.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:named_server_packet_t:s0"
    ~# iptables -t mangle -A INPUT -p udp -s 10.13.14.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:named_server_packet_t:s0"

    ```

## 它是如何工作的…

通过使用自定义的网络数据包标签，可以使用 SELinux 策略来管理特定应用程序的进出访问。尽管多个应用程序可以接受传入的 DNS 请求，本食谱展示了如何确保只有一个应用程序能够处理通过某个过滤器的请求。

每当使用`iptables`启用 SECMARK 标签时，Linux 内核将自动对所有数据包启用 SECMARK 标签。管理员未特别标记的数据包将被标记为`unlabeled_t`类型。某些域可以通过`corenet_sendrecv_unlabeled_packets`接口（或`kernel_sendrecv_unlabeled_packets`接口）处理`unlabeled_t`数据包。然而，如果不是这种情况，那么这些域将无法再处理网络流量。

因此，建议对其他传入（和传出）流量使用标准标签。为了识别哪些传入流量应该被标记，我们可以利用`netstat`输出的帮助：

```
~# netstat -naptZ | awk '/LISTEN/ {print $4,$6,$7,$8}'
0.0.0.0:13500 LISTEN 6489/mysqld system_u:system_r:mysqld_t:s0
0.0.0.0:80 LISTEN 23303/httpd system_u:system_r:httpd_t:s0
10.11.12.122:53 LISTEN 4432/dnsmasq system_u:system_r:dnsmasq_t:s0
10.13.14.42:53 LISTEN 5423/named system_u:system_r:named_t:s0

```

基于此输出，将适当的流量标记为`mysqld_server_packet_t`和`http_server_packet_t`，将允许这些域访问它们的传入网络流量。

通过为`dnsmasq_t`和`named_t`创建额外的类型，这些应用程序只能处理与这些数据包类型相关的请求。如果管理员更改了其中一个 DNS 服务器的配置，则网络数据包标签仍将确保来自先前标识的网络段的 DNS 请求无法被错误的 DNS 服务器使用，即使防火墙规则允许该流量通过。

使用`sesearch`，查询策略以查看哪些应用程序（域）能够发送和接收特定的数据包非常简单：

```
~# sesearch -t dns_server_packet_t -ACTS
Found 10 semantic av rules:
 allow nova_network_t dns_server_packet_t : packet { send recv } ;
 allow corenet_unconfined_type packet_type : packet { send recv relabelto flow_in flow_out forward_in forward_out } ;
 allow named_t dns_server_packet_t : packet { send recv } ;
 allow vmware_host_t server_packet_type : packet { send recv } ;
 allow dnsmasq_t dns_server_packet_t : packet { send recv } ;
 allow kernel_t packet_type : packet send ;
 allow iptables_t packet_type : packet relabelto ;
ET allow squid_t packet_type : packet { send recv } ; [ squid_connect_any ]
DT allow icecast_t packet_type : packet { send recv } ; [ icecast_connect_any ]
DT allow git_session_t server_packet_type : packet { send recv } ; [ git_session_bind_all_unreserved_ports ]

```

同样的方法也可以从客户端层面采用。邮件服务器可能需要连接到其他邮件服务器，这意味着传出的数据可以被标记为`mail_client_packet_t`（如果我们使用默认的流量标签）。然而，如果我们希望确保只有邮件服务器能够连接到其他邮件服务器（而没有其他域也有权限发送和接收`mail_client_packet_t`数据包），则可以使用新的数据包类型。

## 另见

有关 SECMARK 标签的更多信息，请参考以下资源：

+   [`www.selinuxproject.org/page/NB_Networking`](http://www.selinuxproject.org/page/NB_Networking)

+   Paul Moore 的**过渡到 Secmark**，详见 [`paulmoore.livejournal.com/4281.html`](http://paulmoore.livejournal.com/4281.html)

+   James Morris 的**基于 Secmark 的 SELinux 网络控制**，详见 [`james-morris.livejournal.com/11010.html`](http://james-morris.livejournal.com/11010.html)
