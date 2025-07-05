# Chapter 3. CentOS Filesystems – A Deeper Look

So we know that our filesystems are comprised of files and directories; both of which are files, just different types. However, what about links, pipes, and sockets? What are they and how are they used? And why do we talk of links? What is the difference between a hard link and soft link? I think I need to sit down. I can feel one of my turns coming on.

Let's also challenge the traditional filesystem design; you may well have worked with a **logical volume manager** (**LVM**) in the past, but let me tell you just how last century that is. You are going to be blown away by the power and ease of your enterprise filesystem management using BTRFS, pronounced Better FS. We will cover the following sections in this chapter:

*   **A magician's secret**: We reveal how to count subdirectories without actually counting them.
*   **Special permissions**: This will cover the tail of the `wall` command and how it met the GUID bit.
*   **Naming your pipes**: I am sure that you would not care to be unnamed, and your pipes feel this way too. We investigate how to use named pipes to enable **inter-process communication** (**IPC**).
*   **Understanding the command stat**: This will cover all you ever needed to know about an inode, the files' metadata.
*   **Enterprise filesystem shootout**: In BTRFS versus LVM, BTRFS wins hands down. We look at what is new in the BTRFS and see how we can make use of snapshots and extend volumes.

# A magician's secret

We know that there are many groups of people in this world that can and often do annoy us; magicians perhaps being just one of those groups of people. They annoy us because we do not know how they do what they do; quite simply we know that we are being tricked, but we don't quite know how. Well let me be the one to break the honor of the magician's circle and disclose a little trick within CentOS Linux that you can use to trick your colleagues; believe me when I say that this is one trick worth knowing.

Let me show you that if I run the following command on my CentOS 6.5 system, I will be shown a long listing of the specified `doc` directory:

```
$ ls -ld /usr/share/doc

```

The output on my system is as follows:

```
drwxr-xr-x. 758 root root 36864 May 1 09:09 /usr/share/doc

```

The first number that is displayed, `758`, is the *link* count. This shows the number of filenames that are hard linked to the file's metadata; in simple terms, this directory has 758 separate names.

Immediately from this value, I can categorically state that this directory has 756 subdirectories!

> *"It's not rocket science, it is subtracting 2 from a number"*

The formula is simple! For a given directory, the number of subdirectories is equal to the hard link count of two.

I think it is time that we investigate this a little further. When a new directory is created, it is initiated with the link count being equal to two; in other words, each new directory has to have two names that point to it. This will consist of the directory name and the file named `.` (just the period by itself).

In fact, in a new directory, there are always two new files that are created along with the directory: the `.` and `..` files.

*   The `.` file represents the directory itself
*   The `..` file represents the parent directory

Try this yourself; it is easier to understand and might prevent you from going too dotty over this:

```
$ cd
$ mkdir newdir 
$ ls -a newdir #Note the two files
$ ls -ld newdir # Note the hard link count of 2

```

The following screenshot shows the new directory and the listing of the two hidden files therein. The color coding, natural to BASH, highlights both files in blue, which indicate that they represent directories:

![A magician's secret](img/5902OS_03_01.jpg)

I am sure, when you take time to think, that we use the dot notation as a form of shorthand all the time. Consider the following code using the copy command `cp`:

```
$ cp /etc/hosts .

```

Here, we copy the `hosts` file from the `/etc` directory to the current directory using the notation of a single dot. In the following example, we change to the parent directory using the `cd` command:

```
$ cd ..

```

We can now begin understanding how a directory's link count relates to the subdirectory count. If the filename consists of two dots representing the parent directory, then for each subdirectory we create in the given directory, we will have a new file pointing to the parent directory. Each of these double-dot files is hard linked to the subdirectory's parent directory.

## Hard links

In Linux filesystems, we have two types of links: hard links and soft or symbolic links. Hard links are, as we have seen, the name or names of the file. A regular file will have just a single name when it is first created. We can add additional names to the file using the `ln` command:

```
$ cd
$ echo "Hello" > my_newfile
$ ln my_newfile the_samefile
$ ls -li my_newfile the_samefile

```

We will walk through the steps we executed on our system as follows:

1.  We move to our home directory.
2.  Then, we create a new file containing the word `Hello`.
3.  The `ln` command links the original file to a new name, `the_samefile`. We now have two filenames that point to the same metadata. The hard link count of both files will be two; the names point to the same metadata.
4.  Using the `ls` command with the option for the long listing, `-l`, will display the hard link count. The option, `-i`, will display the inode number of the file. The inode number of both files will be the same. As hard links share the same inode number, the source and target file must be on the same filesystem. An inode is an entry within a single filesystem.

## Symbolic links

Symbolic links, or soft links as they are sometimes referred to, are completely separate files whose data points to another filename; as such, they can cross the filesystem boundaries and be a little more useful than hard links. Symbolic links have a file type of `l` indicating that they are a special type of file. Hard links are regular files and can only be identified as links via the hard link count. Symbolic links do not affect a file's hard link count; they are completely independent files with their own name, inode, and data. The data of a symbolic link is the pointer to the target files. In the following code segment, we can see the creation and display of a symbolic link:

```
$ cd
$ ln -s my_newfile the_linkedfile
$ ls -l the_linkedfile

```

Let's walk through the following steps that create a symbolic link in our home directory:

1.  We first move to our home directory.
2.  The `ln` command links the original file to a new name, `the_linkedfile`. The option `-s` will create a soft link.
3.  Using the `ls` command, which is the option for the long listing, `-l`, we can see from the output that the first character, which indicates the file type, shows an `l` indicating that this file is a symbolic link. The extended output also shows where the target file is.

# Special permissions

The permissions or *mode* of a file you we will be familiar with is **Read, Write, and eXecute** (**RWX**). These permissions can be set to the three objects:

*   User
*   Group
*   Others

The standard permissions are shown with their octal notation, should you want a quick revision exercise, as follows:

![Special permissions](img/5902OS_03_02.jpg)

There is a fourth block of permissions that precedes user, group, and others. This block is for the special permissions; however, rather than representing RWX, the permissions comprise of:

*   The set user ID (SUID) bit
*   The set group ID (SGID) bit
*   The sticky bit

Using symbolic notations, these permissions can be added to `file1`, which acts as our axiom for the filename during the following demonstration:

```
$ chmod u+s file1 #adding the SUID Bit
$ chmod g+s file1 #adding the SGID Bit
$ chmod o+t file1 #adding the Sticky Bit

```

## The SUID bit

The set user ID bit is used when a program needs to run using another user ID other than the user running the program. When set, the program runs with the permissions of the file's owner and not the user ID of the current user. This is set on some simple programs; for example, the password program `/usr/bin/passwd` has this permission set. This is required as standard users can change their own password program but they do not have the permissions to write to the `/etc/shadow` file, where the passwords are stored. The program will always execute as the root user no matter who initiates it.

If you are curious and want to see how many files on your system have this permission included, then the `find` command may come to your aid:

```
$ find / -perm +4000

```

The command, as typed, will search the OS root directory, `/`, down for any file that includes the SUID bit, `-perm +4000`.

## The SGID bit

Similar to the set UID bit, if the SGID permission is set on an executable, then the program will run with the group ID of the file's group rather than the group ID of the current user. This is set by default in the `/usr/bin/wall` file, and we will again take a closer look by executing the following command:

```
$ ls -l /usr/bin/wall

```

From the output, we see that the permissions read `r-xr-sr-x`. The lowercase `s` indicates that the SGID and execute permissions are set. If it were an uppercase `S`, then the execute permission would not be set for the group.

When looking at the `/usr/bin/wall` program, we should understand that this program is used to send messages to user consoles; it is a group owned by the `tty` group. With the SGID bit set, when any user executes this program, he or she run with the privileges of the `tty` group.

A user logged on to a console has some control over these messages using the `y` or `n` option with the `mesg` command:

```
$ mesg   #without options displays the current messaging state
$ mesg y #enables messages to be received in the console
$ mesg n #disables messages from being received in the console

```

We could leave the matter at this, understanding that we are simply enabling and disabling messages to our console. We could, but we would not learn the relationship with the SGID bit on the wall program, and we would not unlock the fountain of knowledge that understanding brings. Remaining within our console, we will determine which console we are currently connected to; the `tty` command will help us here. The output on my system shows `/dev/pts/1`. Obtaining a long listing of the device file using the following command will show the file type and permissions. The file type is `c` indicating a character device:

```
$ ls -l /dev/pts/1

```

From the output, we can see the permissions of the file; the group owner is `tty` and, if messaging is enabled, the group will have the write permission. If messaging is disabled, the group will have no permission. We can combine the two commands together using the bracket expansion we first saw in [Chapter 1](ch01.html "Chapter 1. Taming vi"), *Taming vi*:

```
$ ls -l $(tty)

```

The contents of the brackets are evaluated first; the output from which is in turn processed using the `ls -l` command.

In the following screenshot, messaging is disabled as the group does not have the write permission to the console:

![The SGID bit](img/5902OS_03_03.jpg)

When we enable messaging and review the output in the following screenshot, we can see that, miraculously, the write permission now shows for the group:

![The SGID bit](img/5902OS_03_04.jpg)

We have now seen a little more of that initial Linux magic in which we indulged in earlier in this chapter. Moreover, we have seen how to apply it to real-world Linux issues by controlling the use of the `/usr/bin/wall` command via the `/usr/bin/mesg` command.

However, the giving from the SGID does not stop with executable files. The SGID bit can also be set on directories. When set on a directory, the SGID bit ensures that all new files created within the directory are group owned by the group owner from the directory. To put this in context, let's say that our web server's document root, (where the web pages go), is set to the `/var/www/html/` directory. If we set the group ownership of the directory to the `apache` group, we can then use the SGID bit to maintain the correct ownership of all files created. The following commands demonstrate this intricate procedure:

```
# chgrp apache /var/www/html
# chmod g+s /var/www/html
# ls -ld /var/www/html

```

Now, each new web page created `in /var/www/html` will automatically be group owned by the `apache` group. We have now seen that the SGID bit can be effective on executable files and directories.

## The sticky bit

The final special permission is the sticky bit. This is used on directories and set on the `/tmp` directory during the default installation of CentOS. The ability to delete a file is controlled by directory permissions and not by, as some people think, the file's permissions. When you create or delete a file, you are writing to the directory. This means that within a central shared directory, such as `/tmp`, where all users can write to the directory, it would be possible for users to delete any file. To limit the deletions to files owned by the user, the sticky bit is applied to the directory. To add the sticky bit permission to the `/data` directory, we can use the following command:

```
# chmod +t /data

```

# Naming your pipes

I am sure that we all have come across the vertical bar or pipe character `|`; we can use this to create command pipelines, where the output of one command is piped to the input of another. As a simple demonstration, we can use the following commands as an illustration of how often we may use unnamed pipes:

```
$ yum list installed | grep plymouth

```

The first command, `yum list installed`, lists all the installed packages which will be a considerable size; in order to reduce the content, we search for the string `plymouth` with the second command `grep`. The two lines of code are conjoined with an unnamed pipe. It is said to be unnamed as it is transient and only exists for the instance that the two commands run, which, incidentally, is much shorter than the life of a mayfly.

This transient nature may not be useful to us in every situation, in which case we can create named pipes, which are files with the pipe type. Files can be one of the following types:

*   Regular file
*   Directory
*   Symbolic link
*   Socket
*   Named pipe
*   Character device
*   Block device

You should be quite familiar with the first three types, but we tend to see the others less, although we saw a character device file `/dev/pts/1` in the previous section where we were looking at the SGID bit. Character devices are simply terminals that we can access. Here, we want to keep our focus on the file type of pipe, some of which may exist in your filesystem already. We can hunt them using the `find` command, searching for a file type `p`:

```
$ find / -type p 2>/dev/null

```

Running this command as a standard user, you can expect errors related to directories to which we do not have rights; in such a case, it is often easier to redirect errors to `/dev/null`, as we have done here. Having the `autofs` service running on your system will create named pipes: `/var/run/autofs.fifo-misc` and `/var/run/autofs.fifo-net`.

Named pipes allow for different processes to talk with each other or interprocess communication. With unnamed pipes, the processes are always running in the same parent hierarchy, the same BASH shell in other words. As such, they are useful but only to us and our own parochial worlds. Named pipes, however, open up the input of one command to any process running on that system irrelevant of the process hierarchy. This is perhaps similar to the realization that you had when you first realized that there were more places in the world to holiday than the Isle of Wight. A service process such as the `autofs` service may connect to the output of the named pipe waiting for the input from clients on the system, releasing us from the inward facing coterie, which is the unnamed pipe, into a wider expanse of communication.

The easiest way to explain how named pipes operate is to demonstrate them. So why don't we open two terminal windows? These can be graphical terminals running on the desktop if this is easier. I will stay logged in through my own standard account into both windows.

In the first terminal window, we can type the following groups of commands:

```
$ cd  #move to your home directory
$ mkfifo my-pipe #create a named pipe called "my-pipe"
$ ls -l my-pipe #will list the file as type p
$ wc -l < my-pipe #We read in from the pipe and count the lines. As nothing is connected to the input we wait for something to process.

```

While the first window waits for input, we can go to the second terminal window and feed data to the input of the pipe:

```
$ cd #move to your home directory where the pipe is located
$ ls > my-pipe #send the output of the ls command to the pipe

```

Immediately, we will see the result in the first window as we are now able to count the lines of output from the `ls` command that we input in the second terminal. By doing this, we have allowed two separate processes to talk with each other.

# Understanding the command stat

The CentOS command line is full of tools, and trying to learn them all is perhaps a lifetime's work. As with all tasks, reaching the finish line begins with the first step. Our first step will be to delve into the world of the `/usr/bin/stat` command. By using this command, we can query a file's metadata. A file in CentOS consists of:

*   A filename (hard link)
*   File metadata (inode)
*   Data

Using `stat` and the filename alone, we can view the complete inode metadata. This is demonstrated with the following group of commands:

```
$ cd #move to your home directory
$ ls > my_newfile #list the contents and redirect the output to the new file
$ stat my_newfile #display the inode metadata

```

The following screenshot displays the output of `stat`:

![Understanding the command stat](img/5902OS_03_05.jpg)

We can see that the complete metadata is displayed, but if we choose, we can display just elements of the metadata; for example, to display the file permissions in the octal format, run the following command:

```
$ stat -c%a my_newfile

```

To display the permission in human-readable format, run the following command:

```
$ stat -c%A my_newfile

```

The output will show `664` and `-rw-rw-r`, respectively. The inode will always store the permissions in the octal format, but many commands, such as `ls` and `stat` can convert to a friendlier format.

There are three timestamps that are stored in the inode:

*   The last access time
*   The last modified time
*   The last changed time

## The last access time

The last access time for a file lists the time that the file was last read. This is dependent on the filesystem maintaining the last access time; there is a mount option noatime that prevents the last access time from being updated. To list the last access time for the file, run the following command:

```
$ stat -c%x my_newfile

```

The time shown for me is 10:12\. If I now read the file and run the command again, the time will change:

```
$ less my_newfile
$ stat -c%x my_newfile

```

The time now shows as 10:28\. This is useful to find out if files are being read on a system. If they are not, it indicates that perhaps they are not needed and can be archived onto another device.

## The last modified time

The last modified time for a file indicates when the file itself was changed, that is, the file's data. If we edit the file and then check the last modified time, it will have changed.

```
$ ls >> my_newfile #we now append another listing to the file
$ stat -c%y my_newfile #displays the last modified time

```

This is now 10:36 as opposed to 10:12 when the original content was created.

### Tip

The output from `ls -l` also shows the file's last modified time.

## The last changed time

The last changed time of a file relates to when the metadata was changed, as opposed to the file's data. Changing the file permissions, for example, will alter the last changed time:

```
$ chmod 640 my_newfile
$ stat -c%z my_newfile

```

The time for my system now shows that the file's metadata was changed at 10:41.

# Enterprise filesystem shootout

The LVM has been for many years the way to manage disk growth, and allowing logical volumes to span over multiple disks and support backing up through the use of snapshots. LVMs, although very good, still require a filesystem to sit on top of the logical volume and hence, incur an extra level of management; bearing in mind that the LVM system itself has three levels of management:

*   **Physical volumes**: These are the disk space made available to the LVM system
*   **Volume groups**: These organize the physical volumes to be made available to the consumer
*   **Logical volumes**: These consume the disk space made available via the volume groups and are presented to the filesystem tools to be formatted

Now just because we have used such software for the last 10 years or so does not give it the right to continue unchallenged, even within the enterprise. We now see **B-tree filesystem** (**BTRFS**) pronounced as **Better FS** making inroads in Linux. BTRFS is available on version 0.20 to install and can be used on CentOS 6.5, although caution should be taken, as it is marked as experimental.

## What BTRFS has to offer

With over 55 kernel-based filesystems in the Linux kernel tree currently, do we really need another one? The first issue here is that many filesystems have limited or very specific usage; only the extN systems such as ext2, ext3, and ext4 are truly general purpose but even with the latest incarnation of these, ext4, the size limit is 16 TB. BTRFS scales to 16 **exabytes** (**EB**) and brings reliability features previously not found, as follows:

*   Very fast filesystem creation
*   Data and metadata checksums
*   Snapshotting
*   Online scrub to fix issues

## Installing BTRFS

On the CentOS 6.5 demonstration system I am using, we will first need to install BTRFS:

```
# yum install -y btrfs-progs

```

Now that we have the utilities installed, we can begin to experience the power and simplicity of BTRFS. My lab machine currently has four additional free partitions on the second drive; each one consists of 1 GB to use in the following demonstrations.

## Creating a BTRFS filesystem

To kick off the show today, we will first create a BTRFS filesystem on a single 1 GB partition, mount it to the `/data` directory, and copy some data to it as follows:

```
# mkfs.btrfs /dev/sdb5 
# mount /dev/sdb5 /data 
# find /usr/share/doc -name '*.pdf' -exec cp {} /data \; 
# btrfs filesystem show /dev/sdb5 

```

From these commands, you will see that we copy some of the existing PDF files to give us some real data to use in the demonstration, ensuring that we will see no loss of data during the exercises. The final command line shows the filesystem and confirms it size of 1 GB.

## Expanding a BTRFS filesystem

We may well be running out of space within the `/data` structure; we are not but we can imagine. If we were using an LVM structure, we would have to run several commands to expand the existing filesystem across a new partition or disk. This would be the process in LVMs:

Volume management in the old way requires us to execute the following commands:

```
# pvcreate /dev/sdb6
# vgextend vg1 /dev/sdb6
# lvextend -L+1000M /dev/vg1/data_lv
# resize2fs /dev/vg1/data

```

## Volume management with BTRFS

As we can see, there are four commands to be executed, all with a generous sprinkling of syntax that will try to trip us up. We can now see how to do this using BTRFS:

```
# btrfs add device /dev/sdb6 /data

```

That's it! That is all that we needed to do, and we now have a 2 GB volume. We can confirm this by using the following command:

```
# df -h /data
# btrfs filesystem show /dev/sdb5

```

Both commands will confirm that we now have 2 GB of disk space available in the volume and the data is still there and accessible. The volume metadata is copied to both partitions. In this way, we can view the volume information from either device:

```
# btrfs filesystem show /dev/sdb5
# btrfs filesystem show /dev/sdb6

```

Both commands will show the same data, as their metadata is stored on both devices.

## Balancing the filesystem

If we had genuinely added the extra partition because we were running out of disk space within the original volume, then we can balance the data across the complete volume now as follows:

```
# btrfs balance start -d -m /data

```

The `-m` argument represents the metadata and `-d` represents the data. In this way, the disks are equally used.

## Adding an entry to /etc/fstab

One would assume that we would like the `/data` directory mounted at boot time and we will add an entry to the `/etc/fstab` file. When mounting from this file, we must reference all the devices:

```
/dev/sdb5  /data  btrfs  device=/dev/sdb5,device=/dev/sdb6  0 0

```

In this way, we instruct the early mount process of the device construction when a BTRFS scan is not available.

## Creating an RAID1 mirror

Software **redundant array of inexpensive disks** (**RAID**) is also support by BTRFS. The following are the currently supported RAID levels:

*   RAID 0: Striping without redundancy
*   RAID 1: Disk mirroring
*   RAID 10: Striped mirror

We can create a mirrored device using BTRFS software mirroring, should we need it. This does not give us extra disk space, but does provide fault tolerance in the case of a disk failure. We can emulate this in our setup, but as all of our partitions are on one disk, it will not help against disk failure, but the idea holds true.

```
# mkfs.btrfs -m raid1 -d raid1 /dev/sdb7 /dev/sdb8
# mount /dev/sdb7 /mirror

```

Creating the mirror, we use RAID1 for the metadata and data `-m` and `–d`, respectively. The disk space available is 1 GB. Whatever we write to `/dev/sdb7` is mirrored to `/dev/sdb8`; with mirroring, we lose 50 percent of the data storage but have a high level of redundancy.

We will again need to add an entry to the `/etc/fstab` file, as seen earlier to ensure the system mounts correctly during boot time:

```
/dev/sdb7  /mirror  btrfs  device=/dev/sdb7,device=/dev/sdb8  0 0

```

# Using BTRFS snapshots

Analyzing what you have so far, BTRFS is quite cool, don't you think? However, we have not yet exhausted the wealth of goodness that it has to offer. Snapshots can be used as read only or read/write copies of data. The reality is that there is no need to copy data as it is effectively linked until it changes in one of the locations. In this way, a snapshot of a large filesystem can be taken instantly. You can use snapshots in the following ways:

*   As part of a backup solution where you may be concerned with open files affecting the backup, the snapshot will be created as read only and subsequently you will implement a backup of the snapshot. In this way, the backup will be of the host filesystem at the point in time that the snapshot was created.
*   Snapshots can be useful where you know many files will change in a structure and you may want to restore the original files quickly. Perhaps where you are working with scripts to modify many files, you can easily revert to the snapshot copies if the scripts prove not to be as robust as you had imagined, thought, or hoped.

The snapshot *must* be created in the same filesystem as the target data; as we mentioned before, the rapid creation of the snapshot is affected by a form of internal linking within the filesystem. Within a BTRFS filesystem, we can create subvolumes. Subvolumes allow discrete management identities within the BTRFS filesystem. We will take a snapshot of a BTRFS subvolume storing it in another subvolume on the same filesystem.

To achieve this, we shall define two subvolumes within the `/data` BTRFS filesystem. Defining the subvolumes will create both the directories in the filesystem as well as the BTRFS subvolume entities. We will create a snapshot of the first subvolume, storing it in the second subvolume on the same `/data` filesystem. We cannot create a snapshot of the complete filesystem as changes to the snapshot will need to be written back to itself casing infinite recursion; believe me infinite recursion is not a good thing, not a good thing, not a good thing,…

Let's begin by creating the two subvolumes:

```
# btrfs subvolume create /data/working
# btrfs subvolume create /data/backup

```

We can list subvolumes easily using the following command:

```
# btrfs subvolume list /data

```

With the subvolumes in place, we can now move our existing data to the `/data/working` directory, allowing some data to be ready for snapshotting. The working directory, as the name suggests, should be where our real data is stored and the lifeblood of our organization. If this data fails, then so does our organization and we lose our jobs. It makes sense that this data is managed carefully.

```
# mv /data/*.pdf  /data/working

```

Our scenario is that we test scripts that will delete files based perhaps on the last accessed time; I do realize that we should not be working with live data but living on the edge does liven up our otherwise mundane life. That said, we have not entirely lost all sense of the importance of this data. Before we run the scripts, we create a read-only snapshot of the data.

To create a read-only snapshot of the working subvolume, execute the following command:

```
# btrfs subvolume snapshot -r /data/working /data/backup/first-run

```

We can list the available subvolumes as shown earlier with the following command:

```
# btrfs subvolume list /data

```

From the output, we can see that the snapshot appears as a new subvolume. Listing the contents of both directories should indicate that the contents are the same:

```
# ls /data/working
# ls /data/backup/first-run

```

The name `first-run` is not important, but perhaps we can create multiple snapshots based on the data before the first run of the scripts, before the second run of the scripts, and so on. At this stage, the snapshot really does not take up any space as the data is the same in both the source and destination. Should we delete all the files from `/data/working`, the **copy-on-write** (**COW**) technology in BTRFS will then create the files in `/data/backup/first-run`. This would also be the case if the files were modified in any way rather than deleted; the snapshot holds the files as they were at the time the snapshot was created. We can simply copy the files back to the original location in the event of a catastrophe.

# Summary

This chapter has seen us disseminate the filesystem structure that we find in CentOS Linux and opens our comfort zone to entertain new technologies such as BTRFS. We began with a little trickery or understanding of the hard link count that we can see with the `ls` or `stat` command. This count shows how many filenames are linked to the one inode or file metadata. Understanding the metadata of the file led us to look more at `/usr/bin/stat` and the options that it supplies to us including the three timestamps, not of the apocalypse but of the file itself: last access, last modified, and last changed.

A little foray into special permissions released the knowledge of how users can enable and disable console messaging, the console files being group owned by the `tty` group, and the write permission being added and removed.

Finally, we basked in the glory that is the BTRFS filesystem. This is truly something to start working with now as this will be the enterprise filesystem of choice for years to come. Providing both filesystem and volume management in a single task is simplified and improved beyond measure.

You now need to prepare yourself for the banquet that is YUM and of software repository management tool, Yellowdog Update Manager, ensuring that we know a little more than simply `yum install`.