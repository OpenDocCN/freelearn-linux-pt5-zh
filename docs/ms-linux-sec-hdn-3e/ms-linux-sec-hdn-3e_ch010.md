# 第九章：9 访问控制列表与共享目录管理

## 加入我们的书籍社区，在 Discord 上与我们互动

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/file59.png)

在上一章中，我们回顾了**自主访问控制**（**DAC**）的基础知识。正常的 Linux 文件和目录权限设置不够细粒度。使用**访问控制列表**（**ACL**），我们可以精细调整权限，以获得我们真正需要的权限集。我们还可以利用这一能力来控制对共享目录中文件的访问。

本章的主题包括以下内容：

+   为用户或组创建 ACL

+   为目录创建继承的 ACL

+   使用 ACL 掩码移除特定权限

+   使用`tar --acls`选项来防止备份过程中丢失 ACL

+   创建一个用户组并添加成员

+   为组创建共享目录，并设置适当的权限

+   设置共享目录上的 SGID 位和粘滞位

+   使用 ACL 仅允许小组中的特定成员访问共享目录中的文件

## 为用户或组创建 ACL

正常的 Linux 文件和目录权限设置是可以的，但它们不够细粒度。使用 ACL，我们可以仅允许某个人访问文件或目录，或者允许多个不同的人访问文件或目录，并为每个人设置不同的权限。如果我们有一个对所有人开放的文件或目录，我们可以使用 ACL 为某个组或个人设置不同级别的访问权限。在本章的最后，我们将把所学的知识结合起来，以管理一个小组共享目录。

你可以使用`getfacl`查看文件或目录的 ACL。（注意，你不能用它一次性查看目录中的所有文件。）首先，我们使用`getfacl`查看是否已经在`acl_demo.txt`文件上设置了 ACL：

```
[donnie@localhost ~]$ touch acl_demo.txt
[donnie@localhost ~]$ getfacl acl_demo.txt
# file: acl_demo.txt
# owner: donnie
# group: donnie
user::rw-
group::rw-
other::r--
[donnie@localhost ~]$
```

在这里我们看到的只是正常的权限设置，因此没有 ACL。

设置 ACL 的第一步是将文件所有者以外的所有人权限移除。这是因为默认的权限设置允许组成员具有读/写权限，其他人具有读权限。所以，如果不移除这些权限，直接设置 ACL 是没有意义的：

```
[donnie@localhost ~]$ chmod 600 acl_demo.txt
[donnie@localhost ~]$ ls -l acl_demo.txt
-rw-------. 1 donnie donnie 0 Nov  9 14:37 acl_demo.txt
[donnie@localhost ~]$
```

使用`setfacl`设置 ACL 时，你可以为用户或组设置任意组合的读、写或执行权限。以我们的例子为例，假设我想让 Maggie 读取文件，但不允许她具有写或执行权限：

```
[donnie@localhost ~]$ setfacl -m u:maggie:r acl_demo.txt
[donnie@localhost ~]$ getfacl acl_demo.txt
# file: acl_demo.txt
# owner: donnie
# group: donnie
user::rw-
user:maggie:r--
group::---
mask::r--
other::---
[donnie@localhost ~]$ ls -l acl_demo.txt
-rw-r-----+ 1 donnie donnie 0 Nov  9 14:37 acl_demo.txt
[donnie@localhost ~]$
```

`setfacl`的`-m`选项意味着我们即将修改 ACL。（好吧，在这种情况下是*创建*一个，但没关系。）`u:`表示我们正在为用户设置 ACL。然后列出该用户的名字，后跟冒号，以及我们想要授予该用户的权限列表。在此案例中，我们只允许 Maggie 读取权限。通过列出我们要应用此 ACL 的文件来完成该命令。`getfacl`的输出显示 Maggie 确实具有读取权限。最后，我们在`ls -l`的输出中看到，尽管我们已为该文件设置了`600`权限，但组仍然被列为具有读权限。但也有一个`+`符号，表示该文件有 ACL。当我们设置 ACL 时，ACL 的权限会作为组权限出现在`ls -l`中。

为了进一步说明，假设我想让 Frank 对这个文件具有读/写访问权限：

```
[donnie@localhost ~]$ setfacl -m u:frank:rw acl_demo.txt
[donnie@localhost ~]$ getfacl acl_demo.txt
# file: acl_demo.txt
# owner: donnie
# group: donnie
user::rw-
user:maggie:r--
user:frank:rw-
group::---
mask::rw-
other::---
[donnie@localhost ~]$ ls -l acl_demo.txt
-rw-rw----+ 1 donnie donnie 0 Nov  9 14:37 acl_demo.txt
[donnie@localhost ~]$
```

所以，我们可以将两个或更多不同的 ACL 分配给同一个文件。在`ls -l`输出中，我们看到为组设置了`rw`权限，这实际上只是我们在两个 ACL 中设置的权限的汇总。

我们可以通过将`u:`替换为`g:`来为组访问设置 ACL：

```
[donnie@localhost ~]$ getfacl new_file.txt
# file: new_file.txt
# owner: donnie
# group: donnie
user::rw-
group::rw-
other::r--
[donnie@localhost ~]$ chmod 600 new_file.txt
[donnie@localhost ~]$ setfacl -m g:accounting:r new_file.txt
[donnie@localhost ~]$ getfacl new_file.txt
# file: new_file.txt
# owner: donnie
# group: donnie
user::rw-
group::---
group:accounting:r--
mask::r--
other::---
[donnie@localhost ~]$ ls -l new_file.txt
-rw-r-----+ 1 donnie donnie 0 Nov  9 15:06 new_file.txt
[donnie@localhost ~]$
```

`accounting`组的成员现在可以读取这个文件。

## 为目录创建继承的 ACL

有时，你可能希望所有在共享目录中创建的文件都拥有相同的 ACL。我们可以通过对目录应用继承的 ACL 来实现这一点。不过，需要理解的是，尽管这听起来像个不错的主意，但以正常方式创建的文件将使文件对组设置读/写权限，并为其他人设置读权限。所以，如果你为一个用户通常创建文件的目录进行设置，最好的情况是创建一个 ACL，给某些人添加写或执行权限。或者，确保用户在创建文件时设置`600`权限，前提是用户确实需要限制对他们文件的访问。

另一方面，如果你正在创建一个在特定目录中创建文件的 shell 脚本，你可以包含`chmod`命令，以确保文件在创建时拥有必要的限制性权限，从而使你的 ACL 按预期工作。

为了演示，我们来创建`new_perm_dir`目录，并在其上设置继承的 ACL。我希望我的 shell 脚本在这个目录中创建的文件拥有读/写访问权限，并且 Frank 仅具有读权限。我不希望其他任何人能够读取这些文件：

```
[donnie@localhost ~]$ setfacl -m d:u:frank:r new_perm_dir
[donnie@localhost ~]$ ls -ld new_perm_dir
drwxrwxr-x+ 2 donnie donnie 26 Nov 12 13:16 new_perm_dir
[donnie@localhost ~]$ getfacl new_perm_dir
# file: new_perm_dir
# owner: donnie
# group: donnie
user::rwx
group::rwx
other::r-x
default:user::rwx
default:user:frank:r--
default:group::rwx
default:mask::rwx
default:other::r-x
[donnie@localhost ~]$
```

我所需要做的只是通过在`u:frank`前添加`d:`，使其成为继承的 ACL。我保留了目录的默认权限设置，这允许每个人读取目录。接下来，我将创建`donnie_script.sh` shell 脚本，该脚本将在该目录中创建一个文件，并为新文件的用户设置读/写权限：

```
#!/bin/bash
cd new_perm_dir
touch new_file.txt
chmod 600 new_file.txt
exit
```

使脚本可执行后，我将运行它并查看结果：

```
[donnie@localhost ~]$ ./donnie_script.sh
[donnie@localhost ~]$ cd new_perm_dir
[donnie@localhost new_perm_dir]$ ls -l
total 0
-rw-------+ 1 donnie donnie 0 Nov 12 13:16 new_file.txt
[donnie@localhost new_perm_dir]$ getfacl new_file.txt
# file: new_file.txt
# owner: donnie
# group: donnie
user::rw-
user:frank:r-- #effective:---
group::rwx #effective:---
mask::---
other::---
[donnie@localhost new_perm_dir]$
```

所以，`new_file.txt`已经创建，且权限设置正确，并且为 Frank 提供了读取权限。（我知道这是一个非常简化的例子，但你明白我的意思。）

## 通过使用 ACL 掩码删除特定权限

你可以使用`-x`选项从文件或目录中删除 ACL。让我们回到之前创建的`acl_demo.txt`文件，并删除 Maggie 的 ACL：

```
[donnie@localhost ~]$ setfacl -x u:maggie acl_demo.txt
[donnie@localhost ~]$ getfacl acl_demo.txt
# file: acl_demo.txt
# owner: donnie
# group: donnie
user::rw-
user:frank:rw-
group::---
mask::rw-
other::---
[donnie@localhost ~]$
```

所以，Maggie 的 ACL 消失了。但`-x`选项会删除整个 ACL，即使这不是你真正想要的。如果你有一个具有多个权限设置的 ACL，你可能只想删除其中一个权限，保留其他权限。在这里，我们看到 Frank 仍然拥有他的 ACL，允许他读写访问。现在，假设我们想要删除写权限，同时仍然允许他保留读权限。为此，我们需要应用一个掩码：

```
[donnie@localhost ~]$ setfacl -m m::r acl_demo.txt
[donnie@localhost ~]$ ls -l acl_demo.txt
-rw-r-----+ 1 donnie donnie 0 Nov  9 14:37 acl_demo.txt
[donnie@localhost ~]$ getfacl acl_demo.txt
# file: acl_demo.txt
# owner: donnie
# group: donnie
user::rw-
user:frank:rw-            #effective:r--
group::---
mask::r--
other::---
[donnie@localhost ~]$
```

`m::r`在 ACL 上设置了只读掩码。运行`getfacl`显示，Frank 仍然拥有读写 ACL，但旁边的注释显示他的有效权限是只读。因此，Frank 的写权限现在已被删除。而且，如果我们为其他用户设置了 ACL，这个掩码也会以相同的方式影响他们。

## 使用 tar 的`--acls`选项来防止在备份过程中丢失 ACL

如果你需要使用`tar`来备份一个文件或最后两个文件：

```
[donnie@localhost ~]$ cd perm_demo_dir
[donnie@localhost perm_demo_dir]$ ls -l
total 0
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file1.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file2.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file3.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file4.txt
-rw-rw----+ 1 donnie donnie     0 Nov  9 15:19 frank_file.txt
-rw-rw----+ 1 donnie donnie     0 Nov 12 12:29 new_file.txt
[donnie@localhost perm_demo_dir]$
```

现在，我将不使用`--acls`进行备份：

```
[donnie@localhost perm_demo_dir]$ cd
[donnie@localhost ~]$ tar cJvf perm_demo_dir_backup.tar.xz perm_demo_dir/
perm_demo_dir/
perm_demo_dir/file1.txt
perm_demo_dir/file2.txt
perm_demo_dir/file3.txt
perm_demo_dir/file4.txt
perm_demo_dir/frank_file.txt
perm_demo_dir/new_file.txt
[donnie@localhost ~]$
```

看起来不错，对吧？啊，但外表可能会欺骗你。看看当我删除目录，然后从备份中恢复时会发生什么：

```
[donnie@localhost ~]$ rm -rf perm_demo_dir/
[donnie@localhost ~]$ tar xJvf perm_demo_dir_backup.tar.xz
perm_demo_dir/
. . .
[donnie@localhost ~]$ cd perm_demo_dir/
[donnie@localhost perm_demo_dir]$ ls -l
total 0
-rw-rw-r--. 1 donnie donnie 0 Nov 5 20:17 file1.txt
-rw-rw-r--. 1 donnie donnie 0 Nov 5 20:17 file2.txt
-rw-rw-r--. 1 donnie donnie 0 Nov 5 20:17 file3.txt
-rw-rw-r--. 1 donnie donnie 0 Nov 5 20:17 file4.txt
-rw-rw----. 1 donnie donnie 0 Nov 9 15:19 frank_file.txt
-rw-rw----. 1 donnie donnie 0 Nov 12 12:29 new_file.txt
[donnie@localhost perm_demo_dir]$
```

我甚至不需要使用`getfacl`来查看 ACL 已经从`perm_demo_dir`目录及其所有文件中消失，因为它们的`+`符号现在都没有了。现在，让我们看看当我使用`--acls`选项时会发生什么。首先，我将向你展示该目录及其唯一文件的 ACL 已设置：

```
[donnie@localhost ~]$ ls -ld new_perm_dir
drwxrwxr-x+ 2 donnie donnie 26 Nov 13 14:01 new_perm_dir
[donnie@localhost ~]$ ls -l new_perm_dir
total 0
-rw-------+ 1 donnie donnie 0 Nov 13 14:01 new_file.txt
[donnie@localhost ~]$
```

现在，我将使用带有`--acls`选项的`tar`命令：

```
[donnie@localhost ~]$ tar cJvf new_perm_dir_backup.tar.xz new_perm_dir/ --acls
new_perm_dir/
new_perm_dir/new_file.txt
[donnie@localhost ~]$
```

我现在将删除`new_perm_dir`目录并从备份中恢复它。就像我们之前做的那样，我们将使用`--acls`选项：

```
[donnie@localhost ~]$ rm -rf new_perm_dir/
[donnie@localhost ~]$ tar xJvf new_perm_dir_backup.tar.xz --acls
new_perm_dir/
new_perm_dir/new_file.txt
[donnie@localhost ~]$ ls -ld new_perm_dir
drwxrwxr-x+ 2 donnie donnie 26 Nov 13 14:01 new_perm_dir
[donnie@localhost ~]$ ls -l new_perm_dir
total 0
-rw-------+ 1 donnie donnie 0 Nov 13 14:01 new_file.txt
[donnie@localhost ~]$
```

`+`符号的存在表明 ACL 在备份和恢复过程中保留了下来。唯一稍微有点棘手的是，你必须在备份和恢复时都使用`--acls`选项。如果你在其中任何一个过程中省略了此选项，你将丢失 ACL。

## 创建用户组并向其中添加成员

到目前为止，我一直在自己的家目录中做演示，仅仅是为了展示基本概念。但最终的目标是向你展示如何利用这些知识做一些更实际的事情，比如控制文件

假设我们想为—你猜对了—市场部门的成员创建一个`marketing`组：

```
[donnie@localhost ~]$ sudo groupadd marketing
[sudo] password for donnie:
[donnie@localhost ~]$
```

现在让我们添加一些成员。我们可以通过三种不同的方式来实现：

+   在创建用户账户时添加成员。

+   使用`usermod`来添加已有用户账户的成员。

+   编辑`/etc/group`文件。

### 在创建用户账户时添加成员

首先，我们可以在创建用户帐户时通过 `useradd` 的 `-G` 选项将成员添加到组中。在 Red Hat、AlmaLinux 或 CentOS 上，命令应该是这样的：

```
[donnie@localhost ~]$ sudo useradd -G marketing cleopatra
[sudo] password for donnie:
[donnie@localhost ~]$ groups cleopatra
cleopatra : cleopatra marketing
[donnie@localhost ~]$
```

在 Debian/Ubuntu 上，命令应该是这样的：

```
donnie@ubuntu3:~$ sudo useradd -m -d /home/cleopatra -s /bin/bash -G marketing cleopatra
donnie@ubuntu3:~$ groups cleopatra
cleopatra : cleopatra marketing
donnie@ubuntu3:~$
```

当然，我还需要按正常方式为 Cleopatra 设置密码：

```
[donnie@localhost ~]$ sudo passwd cleopatra
```

### 使用 `usermod` 将现有用户添加到组中

好消息是，这在 Red Hat/CentOS/AlmaLinux 或 Debian/Ubuntu 上的效果是一样的：

```
[donnie@localhost ~]$ sudo usermod -a -G marketing maggie
[sudo] password for donnie:
[donnie@localhost ~]$ groups maggie
maggie : maggie marketing
[donnie@localhost ~]$
```

在这个例子中，`-a` 选项并不是必需的，因为 Maggie 还不是其他任何二级组的成员。不过，如果她已经属于其他组，`-a` 选项是必要的，否则会覆盖任何现有的组信息，从而把她从之前的组中移除。

这种方法在 Ubuntu 系统中尤其方便，因为在 Ubuntu 系统中创建加密的主目录时，必须使用 `adduser`。（正如我们在前一章节看到的，`adduser` 在创建帐户时并没有给你机会将用户添加到组中。）

### 通过编辑 `/etc/group` 文件将用户添加到组中

这个最终的方法是一个很好的“作弊”方式，可以加快将多个现有用户添加到组中的过程。首先，只需用你喜欢的文本编辑器打开 `/etc/group` 文件，并查找定义你想要添加成员的组的那一行：

```
. . .
marketing:x:1005:cleopatra,maggie
. . .
```

所以，我已经将 Cleopatra 和 Maggie 添加到这个组中了。接下来，我们编辑这个文件，添加几个新的成员：

```
. . .
marketing:x:1005:cleopatra,maggie,vicky,charlie
. . .
```

完成后，保存文件并退出编辑器。

对每个用户运行 `groups` 命令将显示我们的“作弊”方式效果很好：

```
[donnie@localhost etc]$ sudo vim group
[donnie@localhost etc]$ groups vicky
vicky : vicky marketing
[donnie@localhost etc]$ groups charlie
charlie : charlie marketing
[donnie@localhost etc]$
```

这种方法在你需要一次性将许多成员添加到组时非常方便。

## 创建共享目录

在我们的场景中，下一步是创建一个共享目录，供我们市场部门的所有成员使用。这个问题实际上会引发一些争议。有些人喜欢把共享目录放在文件系统的根目录下，而另一些人则喜欢把它放在 `/home/` 目录下。还有一些人有其他的偏好。但实际上，这主要是个人喜好和/或公司政策的问题。除此之外，目录放在哪儿并不重要。为了简化问题，我会直接在文件系统的根目录下创建这个目录：

```
[donnie@localhost ~]$ cd /
[donnie@localhost /]$ sudo mkdir marketing
[sudo] password for donnie:
[donnie@localhost /]$ ls -ld marketing
drwxr-xr-x. 2 root root 6 Nov 13 15:32 marketing
[donnie@localhost /]$
```

新的目录属于 root 用户。它的权限设置为 `755`，允许所有人读取和执行，只有 root 用户拥有写入权限。我们真正想要的是仅允许市场部门的成员访问这个目录。我们将首先更改所有权和组关联，然后设置正确的权限：

```
[donnie@localhost /]$ sudo chown nobody:marketing marketing
[donnie@localhost /]$ sudo chmod 770 marketing
[donnie@localhost /]$ ls -ld marketing
drwxrwx---. 2 nobody marketing 6 Nov 13 15:32 marketing
[donnie@localhost /]$
```

在这种情况下，我们并没有一个特定的用户希望拥有该目录，我们也不希望 root 用户拥有它。所以，将所有权分配给 `nobody` 伪用户账户为我们提供了解决方案。然后，我将 `770` 权限值分配给该目录，这样所有 `marketing` 组成员可以读/写/执行，而其他人无法访问。现在，让我们让我们的一位组员登录，看看她能否在这个目录中创建文件：

```
[donnie@localhost /]$ su - vicky
Password:
[vicky@localhost ~]$ cd /marketing
[vicky@localhost marketing]$ touch vicky_file.txt
[vicky@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 vicky vicky 0 Nov 13 15:41 vicky_file.txt
[vicky@localhost marketing]$
```

好的，这样可以正常工作，除了一个小问题。文件属于 Vicky，这是应该的。但它也和 Vicky 的个人组关联。为了对这些共享文件进行最佳的访问控制，我们需要将它们与 `marketing` 组关联。接下来我们来解决这个问题。

## 在共享目录上设置 SGID 位和粘滞位

我之前告诉过你，将 SUID 或 SGID 权限设置在文件上，特别是可执行文件上，是有一定的安全风险的。但在共享目录上设置 SGID 是完全安全且非常有用的。

目录上的 SGID 行为与文件上的 SGID 行为完全不同。在目录上，SGID 会使任何人创建的文件都与目录关联的组相关联。因此，考虑到 SGID 权限值是 `2000`，让我们在我们的 `marketing` 目录上设置 SGID：

```
[donnie@localhost /]$ sudo chmod 2770 marketing
[sudo] password for donnie:
[donnie@localhost /]$ ls -ld marketing
drwxrws---. 2 nobody marketing 28 Nov 13 15:41 marketing
[donnie@localhost /]$
```

组的可执行位置上的 `s` 表示命令执行成功。现在让我们让 Vicky 重新登录以创建另一个文件：

```
[donnie@localhost /]$ su - vicky
Password:
Last login: Mon Nov 13 15:41:19 EST 2017 on pts/0
[vicky@localhost ~]$ cd /marketing
[vicky@localhost marketing]$ touch vicky_file_2.txt
[vicky@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 vicky marketing 0 Nov 13 15:57 vicky_file_2.txt
-rw-rw-r--. 1 vicky vicky     0 Nov 13 15:41 vicky_file.txt
[vicky@localhost marketing]$
```

Vicky 的第二个文件与 `marketing` 组关联，这正是我们希望的。为了好玩，让我们让 Charlie 也这么做：

```
[donnie@localhost /]$ su - charlie
Password:
[charlie@localhost ~]$ cd /marketing
[charlie@localhost marketing]$ touch charlie_file.txt
[charlie@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
-rw-rw-r--. 1 vicky   marketing 0 Nov 13 15:57 vicky_file_2.txt
-rw-rw-r--. 1 vicky   vicky     0 Nov 13 15:41 vicky_file.txt
[charlie@localhost marketing]$
```

再次强调，Charlie 的文件与 `marketing` 组关联。但由于某种原因，大家都不理解，Charlie 非常不喜欢 Vicky，出于纯粹的恶意决定删除她的文件：

```
[charlie@localhost marketing]$ rm vicky*
rm: remove write-protected regular empty file ‘vicky_file.txt’? y
[charlie@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
[charlie@localhost marketing]$
```

系统抱怨 Vicky 的原始文件是写保护的，因为它仍然与她的个人组关联。但系统仍然允许 Charlie 删除它，即使没有 `sudo` 权限。而且，由于第二个文件与 `marketing` 组关联，Charlie 拥有写访问权限，因此系统允许他毫无阻碍地删除它。

好的。所以，Vicky 抱怨这件事并试图让 Charlie 被解雇。但我们的勇敢管理员有了一个更好的主意。他决定设置粘滞位，防止这种情况再次发生。由于 SGID 位的值为 `2000`，而粘滞位的值为 `1000`，我们可以将两者相加得到 `3000`：

```
[donnie@localhost /]$ sudo chmod 3770 marketing
[sudo] password for donnie:
[donnie@localhost /]$ ls -ld marketing
drwxrws--T. 2 nobody marketing 30 Nov 13 16:03 marketing
[donnie@localhost /]$
```

可执行文件的其他用户位置上的 `T` 表示粘滞位已设置。由于 `T` 是大写的，我们知道其他用户的执行权限没有被设置。设置粘滞位会阻止组成员删除其他人的文件。让我们让 Vicky 来展示她尝试报复 Charlie 时会发生什么：

```
[donnie@localhost /]$ su - vicky
Password:
Last login: Mon Nov 13 15:57:41 EST 2017 on pts/0
[vicky@localhost ~]$ cd /marketing
[vicky@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
[vicky@localhost marketing]$ rm charlie_file.txt
rm: cannot remove ‘charlie_file.txt’: Operation not permitted
[vicky@localhost marketing]$ rm -f charlie_file.txt
rm: cannot remove ‘charlie_file.txt’: Operation not permitted
[vicky@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
[vicky@localhost marketing]$
```

即使使用 `-f` 选项，Vicky 仍然无法删除 Charlie 的文件。Vicky 在此系统上没有 `sudo` 权限，因此尝试删除是无效的。

## 使用 ACL 访问共享目录中的文件

目前所有 `marketing` 组的成员都可以读/写其他组成员的文件。限制文件访问权限仅限于特定的组成员，是我们已经讨论过的两步过程。

### 设置权限并创建 ACL

首先，Vicky 设置正常权限，只允许她自己对文件具有读/写权限。然后，她将创建一个 ACL，允许 Cleopatra 阅读该文件：

```
[vicky@localhost marketing]$ echo "This file is only for my good friend, Cleopatra." > vicky_file.txt
[vicky@localhost marketing]$ chmod 600 vicky_file.txt
[vicky@localhost marketing]$ setfacl -m u:cleopatra:r vicky_file.txt
[vicky@localhost marketing]$ ls -l
total 4
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
-rw-r-----+ 1 vicky marketing 49 Nov 13 16:24 vicky_file.txt
[vicky@localhost marketing]$ getfacl vicky_file.txt
# file: vicky_file.txt
# owner: vicky
# group: marketing
user::rw-
user:cleopatra:r--
group::---
mask::r--
other::---
[vicky@localhost marketing]$
```

这里没有什么是你没有见过的。Vicky 只是撤销了组和其他人的所有权限，并设置了一个只允许 Cleopatra 阅读文件的 ACL。让我们看看 Cleopatra 是否真的能读取该文件：

```
[donnie@localhost /]$ su - cleopatra
Password:
[cleopatra@localhost ~]$ cd /marketing
[cleopatra@localhost marketing]$ ls -l
total 4
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
-rw-r-----+ 1 vicky marketing 49 Nov 13 16:24 vicky_file.txt
[cleopatra@localhost marketing]$ cat vicky_file.txt
This file is only for my good friend, Cleopatra.
[cleopatra@localhost marketing]$
```

到目前为止，一切顺利。但 Cleopatra 能写入该文件吗？让我们看看：

```
[cleopatra@localhost marketing]$ echo "You are my friend too, Vicky." >> vicky_file.txt
-bash: vicky_file.txt: Permission denied
[cleopatra@localhost marketing]$
```

Cleopatra 无法做到这一点，因为 Vicky 在 ACL 中只授予她读取权限。

但是，现在那个狡猾的 Charlie 怎么样？他想要窥探其他用户的文件。让我们看看 Charlie 能否做到：

```
[donnie@localhost /]$ su - charlie
Password:
Last login: Mon Nov 13 15:58:56 EST 2017 on pts/0
[charlie@localhost ~]$ cd /marketing
[charlie@localhost marketing]$ cat vicky_file.txt
cat: vicky_file.txt: Permission denied
[charlie@localhost marketing]$
```

所以，确实只有 Cleopatra 能访问 Vicky 的文件，而且只能读取。

#### 实践实验 – 创建一个共享组目录

对于这个实验，你将整合本章所学的内容，创建一个用于小组的共享目录。你可以在任何虚拟机上执行此操作：

1.  在任何虚拟机上，创建 `sales` 组：

```
sudo groupadd sales
```

1.  创建用户 `mimi`、`mrgray` 和 `mommy`，并在创建帐户时将他们添加到 `sales` 组中。

在 CentOS 或 AlamaLinux 上，执行以下操作：

```
sudo useradd -G sales mimi
sudo useradd -G sales mrgray
sudo useradd -G sales mommy
```

在 Ubuntu 上，执行以下操作：

```
sudo useradd -m -d /home/mimi -s /bin/bash -G sales mimi
sudo useradd -m -d /home/mrgray -s /bin/bash -G sales mrgray
sudo useradd -m -d /home/mommy -s /bin/bash -G sales mommy
```

1.  为每个用户分配一个密码。

1.  在文件系统的根目录下创建 `sales` 目录。设置适当的所有权和权限，包括 SGID 和粘滞位：

```
sudo mkdir /sales
sudo chown nobody:sales /sales
sudo chmod 3770 /sales
ls -ld /sales
```

1.  以 Mimi 的身份登录，并让她创建一个文件：

```
su - mimi
cd /sales
echo "This file belongs to Mimi." > mimi_file.txt
ls -l
```

1.  让 Mimi 为她的文件设置 ACL，只允许 Gray 先生读取它。然后，让 Mimi 注销：

```
chmod 600 mimi_file.txt
setfacl -m u:mrgray:r mimi_file.txt
getfacl mimi_file.txt
ls -l
exit
```

1.  让 Gray 先生登录，看看他能对 Mimi 的文件做什么。然后，让 Gray 先生创建自己的文件并注销：

```
su - mrgray
cd /sales
cat mimi_file.txt
echo "I want to add something to this file." >> mimi_file.txt    
echo "Mr. Gray will now create his own file." > mr_gray_file.txt        
ls -l
exit
```

1.  现在，Mommy 将登录并试图通过窥探其他用户的文件以及尝试删除它们来制造混乱：

```
su - mommy
cat mimi_file.txt
cat mr_gray_file.txt
rm -f mimi_file.txt
rm -f mr_gray_file.txt
exit
```

1.  实验结束。

## 总结

在这一章中，我们看到如何将 DAC 提升到更高的水平。我们首先了解了如何创建和管理 ACL，以对文件和目录提供更细粒度的访问控制。接着，我们了解了如何为特定目的创建用户组，并将成员添加到其中。然后，我们学习了如何使用 SGID 位、粘滞位和 ACL 来管理共享组目录。

但有时，DAC 可能不足以完成任务。对于这种情况，我们还可以使用强制访问控制（MAC），这一部分内容将在下一章讨论。到时候见。

## 问题

1.  在为共享目录中的文件创建 ACL 时，必须首先做什么才能使 ACL 生效？

A. 删除文件对所有人（除了用户）的所有普通权限。

B. 确保文件的权限值为`644`。

C. 确保组内的每个人对文件具有读/写权限。

D. 确保为文件设置 SUID 权限。

1.  设置 SGID 权限在共享组目录中的好处是什么？

A. 无。它是一个安全风险，永远不应执行。

B. 它防止组成员删除彼此的文件。

C. 这样，每个在该目录中创建的文件都会与目录关联的组相关联。

D. 它赋予任何访问该目录的人与目录用户相同的权限。

1.  以下哪条命令会为`marketing`共享组目录设置正确的权限，且设置了 SGID 和粘滞位？

A. **sudo chmod 6770 marketing**

B. **sudo chmod 3770 marketing**

C. **sudo chmod 2770 marketing**

D. **sudo chmod 1770 marketing**

1.  以下哪种`setfacl`选项可以用来从 ACL 中仅删除一个特定权限？

A. `-xB. -r`

C. `-w`

D. `m: :`

E. `-m`

F. `x: :`

1.  以下哪项陈述是正确的？

A. 使用`tar`时，必须在归档创建和提取时都使用`--acls`选项，以便保留归档文件中的 ACL。

B. 使用`tar`时，只需在归档创建时使用`--acls`选项即可保留归档文件中的 ACL。

C. 使用`tar`时，ACL 会在归档文件中自动保留。

D. 使用`tar`时，无法保留归档文件中的 ACL。

1.  以下哪两种方法*不是*将用户 Lionel 添加到`sales`组的有效方法？

A. **sudo useradd -g sales lionel**

B. **sudo useradd -G sales lionel**

C. **sudo usermod -g sales lionel**

D. **sudo usermod -G sales lionel**

E. 通过手动编辑`/etc/group`文件。

1.  创建继承的 ACL 时会发生什么？

A. 使用该继承的 ACL 创建的目录中的每个文件都会与该目录相关联的组相同。

B. 使用该继承的 ACL 创建的目录中的每个文件都会继承该 ACL。

C. 使用该继承 ACL 创建的每个文件都会与目录具有相同的权限设置。

D. 在该目录中创建的每个文件都会设置粘滞位。

1.  以下哪条命令可用于授予用户 Frank 只读权限？

A. **chattr -m u:frank:r somefile.txt**

B. **aclmod -m u:frank:r somefile.txt**

C. **getfacl -m u:frank:r somefile.txt**

D. **setfacl -m u:frank:r somefile.txt**

1.  你刚刚在共享组目录中执行了`ls -l`命令。如何从中判断是否为文件设置了 ACL？

A. 设置 ACL 的文件会在权限设置的开头显示`+`。

B. 设置了 ACL 的文件，其权限设置的开头会有`-`符号。

C. 设置了 ACL 的文件，其权限设置的末尾会有`+`符号。

设置了 ACL 的文件，其权限设置的末尾会有`-`符号。

E. `ls -l` 命令会显示该文件的 ACL。

1.  以下哪一项可以用来查看`somefile.txt`文件的 ACL？

A. **getfacl somefile.txt**

B. **ls -l somefile.txt**

C. **ls -a somefile.txt**

D. **viewacl somefile.txt**

## 进一步阅读

+   如何从 Linux 命令行创建用户和组：[`www.techrepublic.com/article/how-to-create-users-and-groups-in-linux-from-the-command-line/`](https://www.techrepublic.com/article/how-to-create-users-and-groups-in-linux-from-the-command-line/)

+   将用户添加到组：[`www.howtogeek.com/50787/add-a-user-to-a-group-or-second-group-on-linux/`](https://www.howtogeek.com/50787/add-a-user-to-a-group-or-second-group-on-linux/)

+   目录上的 SGID：[`www.toptip.ca/2010/03/linux-setgid-on-directory.html`](https://www.toptip.ca/2010/03/linux-setgid-on-directory.html)

+   什么是粘滞位，以及如何在 Linux 中设置：[`www.linuxnix.com/sticky-bit-set-linux/`](https://www.linuxnix.com/sticky-bit-set-linux/)

## 答案

1.  A

1.  C

1.  B

1.  D

1.  A

1.  A, C

1.  B

1.  D

1.  C

1.  A
