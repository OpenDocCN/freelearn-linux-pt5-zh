# Appendix B. Basic Linux Commands Cheatsheet

A Linux-based system contains many commands. Each installed application is, in fact, a command. This appendix will give an overview of the most basic commands that should, in theory, be available in every basic Linux installation. Most of these commands will be used throughout this book, some of which are considered as the bare basics. This appendix is by no means an authoritative reference, but it should get you well on your way.

# Requesting the manual

Linux features an interesting command called `man`. It is special because, if installed, it opens a manual page about any command. Try it by requesting the manual page of `man`, as follows:

```
packt@PacktPublishing:~$ man man

```

Running `man` with the `man` parameter will open the manual page of `man`. With the *q* key, you can exit `man`, and with the arrow keys, you can navigate around. The *h* key opens a help screen where more keys are explained.

If `man` or manual pages are not installed, the Internet can be used instead. There are many sites that have the most common manual pages available. The website [http://www.die.net](http://www.die.net) is very popular and can be used to query various manual pages.

Something to note is that there are several sections of manual pages available—nine to be exact. The man-manual page will explain each of them briefly. The first section relates to commands and these pages are queried by default.

Finally, a lot of commands often have a short help screen, which can be activated by appending `-h` or `--help` to a command.

# Listing a directory

The `ls` command, which stands for list, is the command used to list the contents of a directory. Without a parameter, it will list the current active directory, and if supplied with a parameter, it will try to list that file or directory, as follows:

```
packt@PacktPublishing:~$ ls /home/
packt

```

# Changing through directories

To change to a different directory, the `cd` command can be used. Without a parameter, `cd` will always change to the current user's home directory; otherwise, the directory that is supplied via the first parameter is used, as follows:

```
packt@PacktPublishing:~$ cd /home
packt@PacktPublishing:/home$

```

# Getting the current working directory

The current active directory or current working directory can be printed using the `pwd` command. This can be useful when one wants to know where one is located in the current filesystem and can be done using the following command:

```
packt@PacktPublishing:~$ pwd
/home/packt

```

# Getting the current user

Finding out which user is currently logged in can be useful, especially when swapping between several users. The `whoami` command will print the current active logged-in user, as follows:

```
packt@PacktPublishing:~$ whoami
packt

```

# Running commands as root

Very often, when administering or setting up a system, certain commands need to be executed as root. The `sudo` command, when set up properly, can be used to allow certain users to execute certain commands as root. The *who* and *what* queries are controlled via the `sudoers` file at `/etc/sudoers` and should be edited with the `visudo` command. The `sudo` command is used as a prefix to the command to be executed as root, as follows:

```
packt@PacktPublishing:~$ sudo whoami
[sudo] password for packt:
root

```

It should be noted that while `sudo` is very often used to execute commands as root, it can also be used to have any user execute any command as any user.

# Changing the current user without logging out

To actually change to a different user as if one would log in with that user, the `su` command is used. With `su` followed by a different username, it is possible to change the identity of the said user. Unlike `sudo`, which requires the current user's password, here, the user to whom access is being requested is required, as shown in the following command:

```
packt@PacktPublishing:~$ su root
Password:

```

Logging out of a shell or from a different user, the following `exit` command is used. It takes no parameters. Alternatively, *Ctrl* + *d* can also be used to log out on nearly all shells.

```
packt@PacktPublishing:~$ exit

```

# Creating files or changing their dates

To create a new empty file, the `touch` command can be used. Additionally, to modify an existing file's access and modification date can be changed to reflect a new date and time, as follows:

```
packt@PacktPublishing:~$ touch /tmp/testfile

```

# Creating directories

To create a new empty directory, the following `mkdir` command can be used:

```
packt@PacktPublishing:~$ mkdir /tmp/testdir

```

# Removing files

To remove a file, the `rm` command can be used. The `rm` command removes the file that is passed along as a parameter. By default, `rm` will refuse to remove a directory; it only operates on files.

The two options that are very often passed to `rm` are `-r` and `-f`. First, a word of caution on the `-f` option, which stands for force; while the `rm` command should be used with extreme care, the `-f` option requires even more thought and attention. The `-f` option forces the removal of anything `rm` can delete, regardless of any permission.

The `-r` option also needs to be used with care, as it stands for recursively delete. Ironically, the `-r` option takes a directory as a parameter, so it can recursively delete every file and directory under the passed location. Recursively deleting a file does not seem to make sense anyway. The following `rm` command shows an example to remove a file:

```
packt@PacktPublishing:~$ rm /tmp/testfile

```

# Removing a directory

Removing a directory is done via the `rmdir` command; it, however, will only operate on an empty directory, as shown in the following command:

```
packt@PacktPublishing:~$ rmdir /tmp/testdir

```

# Copying files and directories

To copy a file, the `cp` command can be used. Copying a file, you need to supply the source file and the destination file as parameters to `cp` in that order. Optionally, a directory can be supplied instead of a file to copy a directory. While `copy` takes many options, which the manual page explains in detail, the `-r` option can be important when dealing with directories, as it tells `copy` to recursively copy a directory and everything underneath it. The following command shows the use of the copy command:

```
packt@PacktPublishing:~$ cp /tmp/testfile /tmp/testdir/copy_of_testfile

```

# Moving files and directories

To move a file, the `mv` command can be used. Supply the source file and destination file as parameters to `mv` in that order. Optionally, a directory can be supplied for the source and the destination or just the destination.

Renaming a file is actually nothing more than moving a file from one name to another. The following command is used to move a file:

```
packt@PacktPublishing:~$ mv /tmp/testfile /tmp/testdir/moved_testfile

```

# Changing file and directory access permissions

To grant or restrict access to certain files and directories, the `chmod` command can be used. This command stands for *change mode* and requires at least two parameters: the mode that needs to be applied and the file or directory on which this needs to be applied.

Managing permissions properly can be quite complex, though the manual page does help quite a bit. The basics are as follows. Under Linux, there are three standard access levels: user, group, and others. A fourth virtual-access level exists to cover the three others, all. Let us take a look at each of these in detail:

*   **User**: This access level relates to the user who owns a file or directory; usually, it refers to the user who created the file or directory
*   **Group**: This access level grants all the users who are also members of this group access to this level
*   **Others**: This access level gives access to everybody else
*   **All**: This is the fourth virtual-access level that incorporates the preceding three levels

The four access levels are often abbreviated with their first letters, `ugoa`. Next to the access levels, there are the access rights, and here, we will look at the three common ones. Technically, there are four, but more on that in a minute! The two primary access rights are read and write access to a file or directory, which are abbreviated with `r` and `w`. The third access right is execute, abbreviated with `x`, which grants execution permission on a file to a user, group, or anybody else. So, for example, `chmod` itself would require the execute access right to be set for anybody to actually execute that file. The fourth access right is `x` again, but this time it is applied to a directory. Since directories cannot be executed, the access right has a different meaning here and hence has four access rights. For directories, it allows users, groups, or anybody else to actually change into the directory and read the list of files in it.

Constructing a mode is done as follows. Firstly, the shorthand letter is used to designate the user, group, or others followed by `+` or `–` to grant or revoke access rights supplied immediately after. Users, groups, and other designators can be combined if separated by a comma. Refer to the following example to see the constructing mode:

```
packt@PacktPublishing:~$ chmod g+r-w,o+r-w-x /tmp/testfile

```

Very often, access permissions are applied by their numerical value, rather than through their letters. This has its roots mostly in history, where the actual mode bits were used. For more details on the numerical values, you can refer to the manual page.

# Changing file and directory ownership

To change the owner of a file or directory, the `chown` command can be used, which stands for change owner. For this, two parameters are required: the new owner and the file or directory that requires new ownership, as shown here:

```
packt@PacktPublishing:~$ chown packt /tmp/testdir

```

To change the group membership of a file, a similar command to `chown`, called `chgroup`, which stands for change group, exists and works identically.

# Changing passwords

To change passwords, the `passwd` command can be used. When executed without a parameter, the current user's password can be changed by supplying both the old and the new password. The root user can change any user's password by supplying that as the first parameter to `passwd`, as follows:

```
packt@PacktPublishing:~$ passwd
Changing password for packt.
(current) UNIX password:
Enter new UNIX password:
Retype new UNIX password:

```

# Displaying the content of a text file

There are many tools to output the content of a file; `less`, `more`, or `cat`, to name just a few. They all work similarly, pass a filename as their parameter, and they will start displaying the content. Both `less` and `more` allow search or scrolling through the file, with `less` being more advanced than `more`. The `cat` utility, which stands for concatenate, will just output whatever it finds in the file, be it text or not, as shown here:

```
packt@PacktPublishing:~$ cat /tmp/testfile

```

One common operation used with `cat` is redirecting the output content of a file to somewhere else, be it a new file where its functions mimic the copying of a file or appending to another file.

There are a few programs that function in a manner that is very similar to `cat`, but operate on compressed files, decompressing them on the fly. Such commands are `zcat`, for **gzip** cat or `xzcat`, for the **xz** compression. A useful purpose lies herein that when redirecting the output, a file could be decompressed and the output can be written elsewhere. [Chapter 3](ch03.html "Chapter 3. Installing an Operating System"), *Installing an Operating System*, makes use of this by taking a compressed binary file and redirecting the output directly onto a flash disk, as shown here:

```
packt@PacktPublishing:~$ xzcat /tmp/archive.xz > /dev/sdb

```

# Modifying the partitions on a disk

The `fdisk` command, which stands for fixed disk, is a command that can create and modify partitions on a hard or flash disk. It requires a disk device node to be supplied as a parameter. While it is quite menu-driven, `fdisk` has a lot of commands. The most important ones are briefly summarized, as follows:

*   `m`: This command shows a help menu
*   `p`: This command prints the current partition table
*   `o`: This command wipes out the entire partition table and creates a new empty partition table
*   `n`: This command creates a new partition by answering a few questions that `fdisk` asks
*   `d`: This command deletes a partition
*   `w`: This command writes the created partition table to the disk and exits

Always take great care when working with partitions.

### Note

The `fdisk` command does not actually write the changes to the disk unless explicitly requested. If there are errors, the *Ctrl* + *c* key can be used to quit `fdisk` without writing changes to the disk.

# Formatting partitions

To format a partition, the various `mkfs` commands can be used. It depends on whether the supporting utilities are installed. When creating an `ext4` partition, `mkfs.ext4` is used. Likewise, to create a `fat` or `vfat` partition, `mkfs.vfat` is used. Each filesystem partitioning tool has different options and parameters, so the manual page for these commands should be checked for details. Generally speaking, when using the default settings, supplying the device-specific partition node, such as `/dev/sdb1`, for the first partition `(1)` on the second hard or flash disk `(b)` is passed as a parameter to the `mkfs` commands. Creating filesystems is a destructive operation. Use it with care!

In the following example, an `ext4` filesystem is created on a previously partitioned USB flash stick. Note that `sudo` was used here to obtain permission to write directly to the flash drive.

```
packt@PacktPublishing:~$ sudo mkfs.ext4 /dev/sdb1

```

A special variant of `mkfs` is `mkswap`, which creates a filesystem specifically geared to swap space.

# Mounting partitions

Attaching storage to a system is called mounting. While many graphical desktop environments seem to just work, behind the scenes, they still mount and unmount disks and partitions. The `mount` command, which makes this happen, requires two parameters: the device node and the mount location.

Either of the two parameters might be omitted if either of them has been defined in the `fstab` file at `/etc/fstab`. The `fstab` file is parsed by `mount` to see what needs to be mounted, where, and how. Usage of the `mount` command is shown here:

```
packt@PacktPublishing:~$ mount /dev/sdb1 /tmp/testdir

```

# Unmounting partitions

To remove a partition from a running system, the `umount` command is used and stands for *unmount*. The *n* seems to be thought about as being redundant so the term has been abbreviated to `umount`. It is very important to know that `umount` requires no files or directories that are being accessed or in use when detaching a partition from the system. Either the device node or the mount point can be used to unmount a partition, as shown here:

```
packt@PacktPublishing:~$ umount /dev/sdb1

```

# Writing data

A somewhat unusual name is used for the program described in this section, `dd`. It is unknown what `dd` stands for, but its purpose is to copy data. There are many possible arguments to `dd`, but the most important ones used in this book will be covered here. The `if` parameter specifies the input file where data is to be read from. The `of` parameter is the output file parameter where the data is to be written. With these two parameters, it is already possible to copy data from the source to the destination. What makes `dd` so versatile is the plethora of other parameters. The `seek` parameter allows you to change the start position where to start writing. The `skip` parameter allows you to change the start position from where data is read. The `bs` parameter, which stands for block-size, determines the size of the data blocks involved in the transaction, and in combination with the `count` parameter, determines how much data is to be copied. As `dd` allows you to very specifically control a copy operation, it is often used to write full images, bootloaders at specific locations, and much more, as shown in the following command. See the manual page for more information.

```
packt@PacktPublishing:~$ dd if=inputfile of=outputfile seek=8 bs=1024

```

# Changing to a special root directory

Normally, the root directory is the main system directory and everything branches from there. Sometimes, we want to restrict access to only certain parts of the system or temporarily pretend a certain directory is this root. The `chroot` command, which stands for change root, ensures that the supplied directory is considered the new root until exited. As a second parameter, `chroot` can be told what command to run from within this restricted root. In the following example, the root directory is changed to `/tmp/testdir` and the requested command to be executed, `bash`, will reside at `/tmp/testdir/bin/bash`, as shown here:

```
packt@PacktPublishing:~$ chroot /tmp/testdir /bin/bash

```

# Forcing the system to write all content to disks

Modern systems buffer everything in memory and occasionally write that content to disk. The obvious reason for this is that the disks are very slow and the memory is fast. This does have a bad side effect, that is, sometimes the data that we expect to be on a disk is not actually written. The `sync` command causes all the data that is not yet written to disk to be synchronized to the disk, as follows:

```
packt@PacktPublishing:~$ sync

```

# Adding new users

To add a new user to the system, the `useradd` command can be used. While there are many options and parameters that can be supplied, as seen in [Chapter 4](ch04.html "Chapter 4. Manually Installing an Alternative Operating System"), *Manually Installing an Alternative Operating System*, the manual page does a great job of explaining all the options. However, just applying a new username is sufficient to create a bare user, as shown in the following command. Note that a new user does not have a password yet and needs one created via the previously mentioned `passwd` command.

```
packt@PacktPublishing:~$ useradd superpackt

```

# Additional commands

This chapter contained a short list of the most basic commands. There are many more commands and even more guides on the Internet that go over a lot of commands. The website [http://www.reallylinux.com/](http://www.reallylinux.com/) has a nice section called *Essential Commands* where these and more are briefly covered, but any site that covers the basic Linux commands can be used to learn more about commands.

# Summary

This appendix covered the most basic Linux commands as well as the ones used throughout this book. The next appendix will give you an overview of the FEX configuration file.