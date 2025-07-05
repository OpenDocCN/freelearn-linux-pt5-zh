## Section 2: Mastering File and Directory Access Control (DAC)

This section will talk about protecting sensitive files and directories by setting proper permissions and ownership, and by using **Extended Attributes** (**xattr**). Avoid security related problems with **Set User ID** (**SUID**) and **Set Group ID** (**SGID**).

The section contains the following chapters:

*   *Chapter 8*, *Mastering Discretionary Access Control*
*   *Chapter 9*, *Access Control Lists and Shared Directory Management*

# 8 Mastering Discretionary Access Control

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file57.png)

**Discretionary Access Control** (**DAC**) really just means that each user has the ability to control who can get into their stuff. If I wanted to open my home directory so that every other user on the system can get into it, I could do that. Having done so, I could then control who can access each specific file. In the next chapter, we'll use our DAC skills to manage shared directories, where members of a group might need different levels of access to the files within.

At this point in your Linux career, you likely know the basics of controlling access by setting file and directory permissions. In this chapter, we'll review the basics, and then we'll look at some more advanced concepts.

In this chapter, we'll cover the following topics:

*   Using `chown` to change the ownership of files and directories
*   Using `chmod` to set permissions on files and directories
*   What SUID and SGID settings can do for us on regular files
*   The security implications of having the SUID and SGID permissions set on files that don't need them
*   How to use extended file attributes to protect sensitive files
*   Securing system configuration files

### Using chown to change ownership of files and directories

Controlling access to files and directories really just boils down to ensuring that the proper users can access their own files and directories and that each file and directory has permissions set in such a way that only authorized users can access them. The `chown` utility covers the first part of this equation.

One unique thing about `chown` is that you must have `sudo` privileges to use it, even if you're working with your own files in your own directory. You can use it to change the user of a file or directory, the group that's associated with a file or directory, or both at the same time.

First, let's say that you own the `perm_demo.txt` file and that you want to change both the user and group association to that of another user. In this case, I'll change the file ownership from me to `maggie`:

```
[donnie@localhost ~]$ ls -l perm_demo.txt
-rw-rw-r--. 1 donnie donnie 0 Nov  5 20:02 perm_demo.txt
[donnie@localhost ~]$ sudo chown maggie:maggie perm_demo.txt
[donnie@localhost ~]$ ls -l perm_demo.txt
-rw-rw-r--. 1 maggie maggie 0 Nov  5 20:02 perm_demo.txt
[donnie@localhost ~]$
```

The first `maggie` in `maggie:maggie` is the user to whom you want to grant ownership. The second `maggie`, after the colon, represents the group that you want the file to be associated with. Since I was changing both the user and the group to `maggie`, I could have left off the second `maggie`, with the first `maggie` followed by a colon, and I would have achieved the same result:

```
sudo chown maggie: perm_demo.txt
```

To just change the group association without changing the user, just list the group name, preceded by a colon:

```
[donnie@localhost ~]$ sudo chown :accounting perm_demo.txt
[donnie@localhost ~]$ ls -l perm_demo.txt
-rw-rw-r--. 1 maggie accounting 0 Nov  5 20:02 perm_demo.txt
[donnie@localhost ~]$
```

Finally, to just change the user without changing the group, list the username without the trailing colon:

```
[donnie@localhost ~]$ sudo chown donnie perm_demo.txt
[donnie@localhost ~]$ ls -l perm_demo.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:02 perm_demo.txt
[donnie@localhost ~]$
```

These commands work the same way on a directory as they do on a file. However, if you also want to change the ownership and/or the group association of the contents of a directory, while also making the change on the directory itself, use the `-R` option, which stands for *recursive*. In this case, I just want to change the group for the `perm_demo_dir` directory to `accounting`. Let's see what we have to begin with:

```
[donnie@localhost ~]$ ls -ld perm_demo_dir
drwxrwxr-x. 2 donnie donnie 74 Nov  5 20:17 perm_demo_dir
[donnie@localhost ~]$ ls -l perm_demo_dir
total 0
-rw-rw-r--. 1 donnie donnie 0 Nov  5 20:17 file1.txt
-rw-rw-r--. 1 donnie donnie 0 Nov  5 20:17 file2.txt
-rw-rw-r--. 1 donnie donnie 0 Nov  5 20:17 file3.txt
-rw-rw-r--. 1 donnie donnie 0 Nov  5 20:17 file4.txt
```

Now, let's run the command and look at the results:

```
[donnie@localhost ~]$ sudo chown -R :accounting perm_demo_dir
[donnie@localhost ~]$ ls -ld perm_demo_dir
drwxrwxr-x. 2 donnie accounting 74 Nov  5 20:17 perm_demo_dir
[donnie@localhost ~]$ ls -l perm_demo_dir
total 0
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file1.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file2.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file3.txt
-rw-rw-r--. 1 donnie accounting 0 Nov  5 20:17 file4.txt
[donnie@localhost ~]$
```

That's all there is to `chown`. Next, let’s change some permissions.

### Using chmod to set permissions on files and directories

On Unix and Linux systems, you would use the `chmod` utility to set permissions values on files and directories. You can set permissions for the user of the file or directory, the group that's associated with the file or directory, and more. The three basic permissions are as follows:

*   **r**: This indicates a read permission.
*   **w**: This indicates a write permission.
*   **x**: This is the executable permission. You can apply it to any type of program file, or to directories. If you apply an executable permission to a directory, authorized people will be able to `cd` into it.

If you perform `ls -l` on a file, you'll see something like this:

```
-rw-rw-r--. 1 donnie donnie     804692 Oct 28 18:44 yum_list.txt
```

The first character of this line indicates the type of file. In this case, we can see a dash, which indicates a regular file. (A regular file is pretty much every type of file that a normal user would be able to access in his or her daily routine.) The next three characters, `rw-`, indicate that the file has read and write permissions for the user, which is the person who owns the file. Then, we can see the `rw-` permissions for the group and the `r--` permissions for others. A program file would also have the executable permissions set:

```
-rwxr-xr-x. 1 root root     62288 Nov 20  2015 xargs
```

Here, we can see that the `xargs` program file has executable permissions set for everybody.

There are two ways that you can use `chmod` to change permissions settings:

*   The symbolic method
*   The numerical method

We'll cover these methods next.

#### Setting permissions with the symbolic method

Whenever you create a file as a normal user, by default, it will have read/write permissions for the user and group, and read permission for others.

```
chmod u+x donnie_script.sh
chmod g+x donnie_script.sh
chmod o+x donnie_script.sh
chmod u+x,g+x donnie_script.sh
chmod a+x donnie_script.sh
```

The first three commands add the executable permission for the user, the group, and others. The fourth command adds executable permissions for both the user and the group, while the last command adds executable permissions for everybody (`a` for all). You can also remove the executable permissions by replacing `+` with `-`. Finally, you can also add or remove the read or write permissions, as appropriate.

While this method can be handy at times, it also has a bit of a flaw; that is, it can only add permissions to what's already there, or remove permissions from what's already there. If you need to ensure that all of the permissions for a particular file get set to a certain value, the symbolic method can get a bit unwieldy. And for shell scripting, forget about it. In a shell script, you'd need to add all kinds of extra code just to determine which permissions have already been set. The numerical method can vastly simplify things for us.

#### Setting permissions with the numerical method

With the numerical method, you'll use an octal value to represent the permissions settings on a file or directory. For the `r`, `w`, and `x` permissions, you assign the numerical values `4`, `2`, and `1`, respectively. You would do this for the user, group, and others positions, and then add them all up to get the permissions value for the file or directory:

| **User** | **Group** | **Others** |
| rwx | rwx | rwx |
| 421 | 421 | 421 |
| 7 | 7 | 7 |

So, if you have all the permissions set for everybody, the file or directory will have a value of `777`. If I were to create a shell script file, by default, it would have the standard `664` permissions, meaning read and write for the user and group, and read-only for others:

```
-rw-rw-r--. 1 donnie donnie 0 Nov  6 19:18 donnie_script.sh
```

> **Tip**
> 
> > If you create a file with root privileges, either with `sudo` or from the root user command prompt, you'll see that the default permissions setting is the more restrictive `644`.

Let's say that I want to make this script executable, but I want to be the only person in the whole world who can do anything with it. To do this, I could do the following:

```
[donnie@localhost ~]$ chmod 700 donnie_script.sh
[donnie@localhost ~]$ ls -l donnie_script.sh
-rwx------. 1 donnie donnie 0 Nov  6 19:18 donnie_script.sh
[donnie@localhost ~]$
```

With this one simple command, I've removed all permissions from the group and from others, and set the executable permission for myself. This is the sort of thing that makes the numerical method so handy for writing shell scripts.

Once you've been working with the numerical method for a while, looking at a file and figuring out its numerical permissions value will become second nature. In the meantime, you can use `stat` with the `-c %a` option to show you the values. This can be done like so:

```
[donnie@localhost ~]$ stat -c %a yum_list.txt
664
[donnie@localhost ~]$
[donnie@localhost ~]$ stat -c %a donnie_script.sh
700
[donnie@localhost ~]$
[donnie@localhost ~]$ stat -c %a /etc/fstab
644
[donnie@localhost ~]$
```

If you want to view the numerical permissions of all the files at once, do this:

```
[donnie@donnie-ca ~]$ stat -c '%n %a ' *
dropbear 755 
internal.txt 664 
password.txt 664 
pki-server.crt 664 
pki-server.p12 644 
yum_list.txt 664 
[donnie@donnie-ca ~]$
```

Here, you can see the wildcard (`*`) at the end of the command, indicating that you want to view the settings for all the files. `%n` indicates that you want to view the filenames, along with the permissions settings. Since we're using two `-c` options, we have to enclose both of the options within a pair of single quotes. The only slight catch here is that this output doesn't show which of these items are files, and which are directories. However, since directories require executable permissions so that people can `cd` into them, we can guess that `dropbear` is probably a directory. To be sure though, just use `ls -l`, like so:

```
[donnie@donnie-ca ~]$ ls -l
total 2180
-rwxr-xr-x. 1 donnie donnie  277144 Apr 22  2018 dropbear
-rw-rw-r--. 1 donnie donnie      13 Sep 19 13:32 internal.txt
-rw-rw-r--. 1 donnie donnie      11 Sep 19 13:42 password.txt
-rw-rw-r--. 1 donnie donnie    1708 Sep 19 14:41 pki-server.crt
-rw-r--r--. 1 root   root      1320 Sep 20 21:08 pki-server.p12
-rw-rw-r--. 1 donnie donnie 1933891 Sep 19 18:04 yum_list.txt
[donnie@donnie-ca ~]$
```

Now, let's move on to a couple of very special permissions settings.

### Using SUID and SGID on regular files

When a regular file has its SUID permission set, whoever accesses the file will have the same privileges as the user of the file.

To demo this, let's say that Maggie, a regular, unprivileged user, wants to change her own password. Since it's her own password, she would just use the one-word `passwd` command, without using `sudo`:

```
[maggie@localhost ~]$ passwd
Changing password for user maggie.
Changing password for maggie.
(current) UNIX password:
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[maggie@localhost ~]$
```

To change a password, a person has to make changes to the `/etc/shadow` file. On my CentOS and AlmaLinux machines, the shadow file's permissions look like this:

```
[donnie@localhost etc]$ ls -l shadow
----------. 1 root root 840 Nov  6 19:37 shadow
[donnie@localhost etc]$
```

On an Ubuntu machine, they look like this:

```
donnie@ubuntu:/etc$ ls -l shadow
-rw-r----- 1 root shadow 1316 Nov  4 18:38 shadow
donnie@ubuntu:/etc$
```

Either way, the permissions settings don't allow Maggie to directly modify the shadow file. However, by changing her password, she is able to modify the shadow file. So, what's going on? To answer this, let's go into the `/usr/bin/` directory and look at the permissions settings for the `passwd` executable file:

```
[donnie@localhost etc]$ cd /usr/bin
[donnie@localhost bin]$ ls -l passwd
-rwsr-xr-x. 1 root root 27832 Jun 10 2014 passwd
[donnie@localhost bin]$
```

For the user permissions, you will see `rws` instead of `rwx`. The `s` indicates that this file has the SUID permission set. Since the file belongs to the root user, anyone who accesses this file has the same privileges as the root user. The fact that we see a lowercase `s` means that the file also has the executable permission set for the root user. Since the root user is allowed to modify the shadow file, whoever uses this `passwd` utility to change his or her own password can also modify the shadow file.

A file with the SGID permission set has an `s` in the executable position for the group:

```
[donnie@localhost bin]$ ls -l write
-rwxr-sr-x. 1 root tty 19536 Aug  4 07:18 write
[donnie@localhost bin]$
```

The `write` utility, which is associated with the `tty` group, allows users to send messages to other users via their command-line consoles. Having `tty` group privileges allows users to do this.

### The security implications of the SUID and SGID permissions

As useful as it may be to have SUID or SGID permissions on your executable files, we should consider it as just a necessary evil. While having SUID or SGID set on certain operating system files is essential for the operation of your Linux system, it becomes a security risk when users set SUID or SGID on other files. The problem is that, if intruders find an executable file that belongs to the root user and has the SUID bit set, they can use that to exploit the system. Before they leave, they might leave behind their own root-owned file with an SUID set, which will allow them to easily gain entry to the system the next time they encounter it. If the intruder's SUID file isn't found, the intruder will still have access, even if the original problem has been fixed.

The numerical value for SUID is `4000`, and for SGID, it's `2000`. To set SUID on a file, you'd just add `4000` to whichever permissions value that you would set otherwise. For example, if you have a file with a permissions value of `755`, you'd set SUID by changing the permissions value to `4755`. (This would give you read/write/execute access for the user, read/execute for the group, and read/execute for others, with the SUID bit added on.)

#### Finding spurious SUID or SGID files

One quick security trick is to run the `find` command to take inventory of the SUID and SGID files on your system. You can also save the output to a text file so that you can verify whether anything has been added since you ran the command. Your command will look something like this:

```
sudo find / -type f \( -perm -4000 -o -perm -2000 \) > suid_sgid_files.txt
```

Here's the breakdown:

*   **/**: We're searching through the entire filesystem. Since some directories are only accessible to someone with root privileges, we need to use `sudo`.
*   **-type f**: This means that we're searching for regular files, which includes executable program files and shell scripts.
*   **-perm 4000**: We're searching for files with the `4000`, or SUID, permission bit set.
*   **-o**: The or operator.
*   **-perm 2000**: We're searching for files with the `2000`, or SGID, permission bit set.
*   **>**: Here, we're redirecting the output into the `suid_sgid_files.txt` text file with the `>` operator.

Note that the two `-perm` items need to be combined into a term that's enclosed in a pair of parentheses. To prevent the Bash shell from interpreting the parenthesis characters incorrectly, we need to escape each one with a backslash. We also need to place a blank space between the first parenthesis character and the first `-perm`, and another between `2000` and the last backslash. Also, the **and** operator between `-type f` and the `-perm` term is understood to be there, even without inserting `-a`. The text file that you'll create should look something like this:

```
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/su
/usr/bin/umount
/usr/bin/sudo
. . .
. . .
/usr/lib64/dbus-1/dbus-daemon-launch-helper
```

Optionally, if you want to see details about which files are SUID and which are SGID, you can add the `-ls` option:

```
sudo find / -type f \( -perm -4000 -o -perm -2000 \) -ls > suid_sgid_files.txt
```

Okay, you're now saying, *Hey, Donnie, this is just too much to type*. And, I hear you. Fortunately, there's a shorthand equivalent of this. Since `4000 + 2000 = 6000`, we can create a single expression that will match either the SUID (`4000`) or the SGID (`2000`) value, like this:

```
sudo find / -type f -perm /6000 -ls > suid_sgid_files.txt
```

The `/6000` in this command means that we're looking for either the `4000` or the `2000` value. For our purposes, these are the only two addends that can combine to make `6000`.

> **Tip**
> 
> > In some older references, you might see `+6000` instead of `/6000`. Using the `+` sign for this has been deprecated, and no longer works.

Now, let's say that Maggie, for whatever reason, decides to set the SUID bit on a shell script file in her home directory:

```
[maggie@localhost ~]$ chmod 4755 bad_script.sh
[maggie@localhost ~]$ ls -l
total 0
-rwsr-xr-x. 1 maggie maggie 0 Nov  7 13:06 bad_script.sh
[maggie@localhost ~]$
```

Run the `find` command again, saving the output to a different text file. Then, perform a `diff` operation on the two files to see what changed:

```
[donnie@localhost ~]$ diff suid_sgid_files.txt suid_sgid_files2.txt
17a18
> /home/maggie/bad_script.sh
[donnie@localhost ~]$
```

The only difference is the addition of Maggie's shell script file.

##### Hands-on lab – searching for SUID and SGID files

You can perform this lab on either of your virtual machines. You'll save the output of the `find` command to a text file. Let's get started:

1.  Search through the entire filesystem for all the files that have either SUID or SGID set before saving the output to a text file:

```
 sudo find / -type f -perm /6000 -ls > suid_sgid_files.txt
```

1.  Log into any other user account that you have on the system and create a dummy shell script file. Then, set the SUID permission on that file and log back out and into your own user account:

```
su - desired_user_account
touch some_shell_script.sh
chmod 4755 some_shell_script.sh
ls -l some_shell_script.sh
exit
```

1.  Run the `find` command again, saving the output to a different text file:

```
sudo find / -type f -perm /6000 -ls > suid_sgid_files_2.txt
```

1.  View the difference between the two files:

```
diff suid_sgid_files.txt suid_sgid_files_2.txt
```

That's the end of the lab – congratulations!

#### Preventing SUID and SGID usage on a partition

As we mentioned previously, you don't want users to assign SUID and SGID to files that they create, because of the security risk that it presents. You can prevent SUID and SGID usage on a partition by mounting it with the `nosuid` option. So, the `/etc/fstab` file entry for the `luks` partition that I created in the previous chapter would look like this:

```
/dev/mapper/luks-6cbdce17-48d4-41a1-8f8e-793c0fa7c389 /secrets   xfs  nosuid  0 0
```

Different Linux distributions have different ways of setting up default partition schemes during an operating system's installation. Mostly, the default way of doing business is to have all the directories, except for the `/boot/` directory, under the `/` partition. If you were to set up a custom partition scheme instead, you could have the `/home/` directory in its own partition, where you could set the `nosuid` option. Keep in mind that you don't want to set `nosuid` for the `/` partition; otherwise, you'll have an operating system that doesn't function properly.

### Using extended file attributes to protect sensitive files

Extended file attributes are another tool that can help you protect sensitive files. They won't keep intruders from accessing your files, but they can help you prevent sensitive files from being altered or deleted. There are quite a few extended attributes, but we only need to look at the ones that deal with file security.

First, let's use the `lsattr` command to see which extended attributes we already have set. On a CentOS or AlmaLinux machine, your output would look something like this:

```
[donnie@localhost ~]$ lsattr
---------------- ./yum_list.txt
---------------- ./perm_demo.txt
---------------- ./perm_demo_dir
---------------- ./donnie_script.sh
---------------- ./suid_sgid_files.txt
---------------- ./suid_sgid_files2.txt
[donnie@localhost ~]$
```

So far, I don't have any extended attributes set on any of my files.

On an Ubuntu machine, the output would look more like this:

```
donnie@ubuntu:~$ lsattr
-------------e-- ./file2.txt
-------------e-- ./secret_stuff_dir
-------------e-- ./secret_stuff_for_frank.txt.gpg
-------------e-- ./good_stuff
-------------e-- ./secret_stuff
-------------e-- ./not_secret_for_frank.txt.gpg
-------------e-- ./file4.txt
-------------e-- ./good_stuff_dir
donnie@ubuntu:~$
```

We won't worry about the `e` attribute because that only means that the partition is formatted with the **ext4** filesystem. CentOS and AlmaLinux don’t have that attribute set because their partitions are formatted with the **XFS** filesystem.

The two attributes that we'll look at in this section are as follows:

*   **a**: You can append text to the end of a file that has this attribute, but you can't overwrite it. Only someone with proper `sudo` privileges can set or delete this attribute.
*   **i**: This makes a file immutable, and only someone with proper `sudo` privileges can set or delete it. Files with this attribute can't be deleted or changed in any way. It's also not possible to create hard links to files that have this attribute.

To set or delete attributes, you need to use the `chattr` command. You can set more than one attribute on a file, but only when it makes sense. For example, you wouldn't set both the `a` and the `i` attributes on the same file because the `i` will override the `a`.

Let's start by creating the `perm_demo.txt` file, which contains the following text:

```
This is Donnie's sensitive file that he doesn't want to have overwritten.
```

Now, let's go ahead and set the attributes.

#### Setting the a attribute

Now, I'll set the `a` attribute:

```
[donnie@localhost ~]$ sudo chattr +a perm_demo.txt
[sudo] password for donnie:
[donnie@localhost ~]$
```

You use `+` to add an attribute and `-` to delete it. Also, it doesn't matter that the file belongs to me and is in my own home directory. I still need `sudo` privileges to add or delete this attribute.

Now, let's see what happens when I try to overwrite this file:

```
[donnie@localhost ~]$ echo "I want to overwrite this file." > perm_demo.txt
-bash: perm_demo.txt: Operation not permitted
[donnie@localhost ~]$ sudo echo "I want to overwrite this file." > perm_demo.txt
-bash: perm_demo.txt: Operation not permitted
[donnie@localhost ~]$
```

With or without `sudo` privileges, I can't overwrite it. So, how about if I try to append something to it?

```
[donnie@localhost ~]$ echo "I want to append this to the end of the file." >> perm_demo.txt
[donnie@localhost ~]$
```

There's no error message this time. Let's see what's in the file:

```
This is Donnie's sensitive file that he doesn't want to have overwritten.
I want to append this to the end of the file.
```

In addition to not being able to overwrite the file, I'm also unable to delete it:

```
[donnie@localhost ~]$ rm perm_demo.txt
rm: cannot remove ‘perm_demo.txt’: Operation not permitted
[donnie@localhost ~]$ sudo rm perm_demo.txt
[sudo] password for donnie:
rm: cannot remove ‘perm_demo.txt’: Operation not permitted
[donnie@localhost ~]$
```

So, the `a` works. However, I've decided that I no longer want this attribute to be set, so I'll remove it:

```
[donnie@localhost ~]$ sudo chattr -a perm_demo.txt
[donnie@localhost ~]$ lsattr perm_demo.txt
---------------- perm_demo.txt
[donnie@localhost ~]$
```

#### Setting the i attribute

When a file has the `i` attribute set, the only thing you can do with it is view its contents. You can't change it, move it, delete it, rename it, or create hard links to it. Let's test this with the `perm_demo.txt` file:

```
[donnie@localhost ~]$ sudo chattr +i perm_demo.txt
[donnie@localhost ~]$ lsattr perm_demo.txt
----i----------- perm_demo.txt
[donnie@localhost ~]$
```

Now for the fun part:

```
[donnie@localhost ~]$ sudo echo "I want to overwrite this file." > perm_demo.txt
-bash: perm_demo.txt: Permission denied
[donnie@localhost ~]$ echo "I want to append this to the end of the file." >> perm_demo.txt
-bash: perm_demo.txt: Permission denied
[donnie@localhost ~]$ sudo echo "I want to append this to the end of the file." >> perm_demo.txt
-bash: perm_demo.txt: Permission denied
[donnie@localhost ~]$ rm -f perm_demo.txt
rm: cannot remove ‘perm_demo.txt’: Operation not permitted
[donnie@localhost ~]$ sudo rm -f perm_demo.txt
rm: cannot remove ‘perm_demo.txt’: Operation not permitted
[donnie@localhost ~]$ sudo rm -f perm_demo.txt
```

There are a few more commands that I could try, but you get the idea. To remove the `i` attribute, do this:

```
[donnie@localhost ~]$ sudo chattr -i perm_demo.txt
[donnie@localhost ~]$ lsattr perm_demo.txt
---------------- perm_demo.txt
[donnie@localhost ~]$
```

##### Hands-on lab – setting security-related extended file attributes

For this lab, you'll need to create a `perm_demo.txt` file with some text of your choice. You'll set the `i` and `a` attributes and view the results. Let's get started:

1.  Using your preferred text editor, create the `perm_demo.txt` file with a line of text.
2.  View the extended attributes of the file:

```
lsattr perm_demo.txt
```

1.  Add the `a` attribute:

```
sudo chattr +a perm_demo.txt
lsattr perm_demo.txt
```

1.  Try to overwrite and delete the file:

```
echo "I want to overwrite this file." > perm_demo.txt
sudo echo "I want to overwrite this file." > perm_demo.txt
rm perm_demo.txt
sudo rm perm_demo.txt
```

1.  Now, append something to the file:

```
echo "I want to append this line to the end of the file." >> perm_demo.txt
```

1.  Remove the `a` attribute and add the `i` attribute:

```
sudo chattr -a perm_demo.txt
lsattr perm_demo.txt
sudo chattr +i perm_demo.txt
lsattr perm_demo.txt
```

1.  Repeat *Step 4*.
2.  Additionally, try to change the filename and create a hard link to the file:

```
mv perm_demo.txt some_file.txt
sudo mv perm_demo.txt some_file.txt
ln ~/perm_demo.txt ~/some_file.txt
sudo ln ~/perm_demo.txt ~/some_file.txt
```

1.  Now, try to create a symbolic link to the file:

```
ln -s ~/perm_demo.txt ~/some_file.txt
```

> Note that the `i` attribute won't let you create hard links to a file, but it will let you create symbolic links.

That's the end of the lab – congratulations!

### Securing system configuration files

If you look at the configuration files for any given Linux distro, you'll see that most of them belong to either the root user or to a specified system user. You'll also see that most of these files have read and write privileges for their respective owners, and read privileges for everyone else. This means that everybody and his brother can read most Linux system configuration files. Take, for example, this Apache web server configuration file:

```
[donnie@donnie-ca ~]$ cd /etc/httpd/conf
[donnie@donnie-ca conf]$ pwd
/etc/httpd/conf
[donnie@donnie-ca conf]$ ls -l httpd.conf 
-rw-r--r--. 1 root root 11753 Aug  6 09:44 httpd.conf
[donnie@donnie-ca conf]$
```

With that `r` in the "others" position, everybody who logs in, regardless of their privilege level, can view the Apache configuration.

So, is this a big deal? It really depends upon your circumstances. Some configuration files, especially ones for certain PHP-based **Content Management Systems** (**CMS**) on a web server, can contain plain text passwords that the CMS must be able to access. In these cases, it's quite obvious that you need to restrict access to these configuration files. But what about other configuration files that don't contain sensitive passwords?

For servers that only a chosen few administrators can access, this isn't such a big deal. But what about servers that normal, non-administrative users can access remotely via Secure Shell? If they don't have any `sudo` privileges, they can't edit any configuration files, but they can view them to see how your server has been configured. If they see how things are configured, would that help them in their efforts to compromise the system, should they choose to do so?

I have to confess, this is something that I hadn't given much thought about until recently, when I became a Linux consultant for a company that specializes in the security of **Internet of Things** (**IoT**) devices. With IoT devices, you have a bit more to worry about than you do with normal servers. Normal servers are protected with a high degree of physical security, while IoT devices often have little to no physical security. You could go your entire IT career without actually seeing a server, unless you're one of the few who have been authorized to enter the inner sanctum of the server room. Conversely, IoT devices are generally out in the open.

The IoT security company that I work with has a set of guidelines that help harden IoT devices against compromise and attack. One of them is to ensure that all the configuration files on the devices are set with the `600` permissions setting. This would mean that only the owner of the files – generally either the root user or a system account – can read them. However, there are a lot of configuration files, and you need an easy way to change the settings. You can do that with our trusty friend, the `find` utility. Here's how you can do this:

```
sudo find / -iname '*.conf' -exec chmod 600 {} \;
```

Here's the breakdown:

*   **sudo find / -iname '*.conf'**: This does exactly what you would expect it to do. It performs a case-insensitive (`-iname`) search throughout the entire root filesystem (`/`) for all the files with the `.conf` filename extension. Other filename extensions you might look for include `.ini` and `.cfg`. Also, because `find` is inherently recursive, you don't have to provide an option switch to get it to search through all the lower-level directories.
*   **-exec**: This is what performs the magic. It automatically executes the following command on each file that `find` finds, without prompting the user. If you'd rather answer *yes* or *no* for each file that `find` finds, use `-ok` instead of `-exec`.
*   **chmod 600 {} \;**: `chmod 600` is the command that we want to perform. As `find` finds each file, its filename is placed within the pair of curly brackets (`{}`). Every `-exec` clause has to end with a semicolon. To prevent the Bash shell from interpreting the semicolon incorrectly, we have to escape it with a backslash.

If you decide to do this, test things thoroughly to ensure that you haven't broken anything. Most things work just fine with their configuration files set to a `600` permissions setting, but some don't. I've just performed this command on one of my virtual machines. Let's see what happens when I try to ping an internet site:

```
[donnie@donnie-ca ~]$ ping www.civicsandpolitics.com
ping: www.civicsandpolitics.com: Name or service not known
[donnie@donnie-ca ~]$
```

This looks bad, but the explanation is simple. It's just that in order to have internet access, the machine has to be able to find a DNS server. DNS server information can be found in the `/etc/resolv.conf` file, from which I've just removed read permissions for others. Without the read permissions for others, only someone with root user privileges can access the internet. So, unless you want to restrict internet access to users with root or `sudo` privileges, you'll need to change the `resolv.conf` permission setting back to `644`:

```
[donnie@donnie-ca etc]$ ls -l resolv.conf 
-rw-------. 1 root root 66 Sep 23 14:22 resolv.conf
[donnie@donnie-ca etc]$ sudo chmod 644 resolv.conf 
[donnie@donnie-ca etc]$
```

Okay, let's try this again:

```
[donnie@donnie-ca etc]$ ping www.civicsandpolitics.com
PING www.civicsandpolitics.com (64.71.34.94) 56(84) bytes of data.
64 bytes from 64.71.34.94: icmp_seq=1 ttl=51 time=52.1 ms
64 bytes from 64.71.34.94: icmp_seq=2 ttl=51 time=51.8 ms
64 bytes from 64.71.34.94: icmp_seq=3 ttl=51 time=51.2 ms
^C
--- www.civicsandpolitics.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 51.256/51.751/52.176/0.421 ms
[donnie@donnie-ca etc]$
```

That looks much better. Now, let's reboot the machine. When you do, you'll get this output:

![](img/file58.png)

So, I also need to set the `/etc/locale.conf` file back to the `644` permission setting for the machine to boot properly. As I mentioned previously, be sure to test everything if you choose to set more restrictive permissions on your configuration files.

As I've already stated, you might not always find it necessary to change the permissions of your configuration files from their default settings. But if you ever do find it necessary, you now know how to do it.

> **Tip**
> 
> > You definitely want to make friends with the `find` utility. It's useful both on the command line and within shell scripts, and it's extremely flexible. The man page for it is very well-written, and you can learn just about everything you need to know about `find` from it. To see it, just use the `man find` command.
> 
> > Once you get used to `find`, you'll never want to use any of those fancy GUI-type search utilities again.

Okay, I think that this wraps things up for this chapter.

## Summary

In this chapter, we reviewed the basics of setting ownership and permissions for files and directories. Then, we covered what SUID and SGID can do for us when they're used properly, as well as the risk of setting them on our own executable files. After looking at the two extended file attributes that deal with file security, we wrapped things up with a handy, time-saving trick for removing world-readable permissions from your system configuration files.

In the next chapter, we'll extend what we've learned here to more advanced file and directory access techniques. I'll see you there.

## Questions

1.  Which of the following partition mount options would prevent setting the SUID and SGID permissions on files?

A. `nosgid`

B. `noexec`

C. `nosuid`

D. `nouser`

1.  Which of the following represents a file with read and write permissions for the user and the group, and read-only permissions for others?

A. `775`

B. `554`

C. `660`

D. `664`

1.  You want to change the ownership and group association of the `somefile.txt` file to Maggie. Which of the following commands would do that?

A. **sudo chown maggie somefile.txt**

B. **sudo chown :maggie somefile.txt**

C. **sudo chown maggie: somefile.txt**

D. **sudo chown :maggie: somefile.txt**

1.  Which of the following is the numerical value for the SGID permission?

A. `6000`

B. `2000`

C. `4000`

D. `1000`

1.  Which command would you use to view the extended attributes of a file?

A. `lsattr`

B. `ls -a`

C. `ls -l`

D. `chattr`

1.  Which of the following commands would search through the entire filesystem for regular files that have either the SUID or SGID permission set?

A. **sudo find / -type f -perm \6000**

B. **sudo find / \( -perm -4000 -o -perm -2000 \)**

C. **sudo find / -type f -perm -6000**

D. **sudo find / -type r -perm \6000**

1.  Which of the following statements is true?

A. Using the symbolic method to set permissions is the best method for all cases.

B. Using the symbolic method to set permissions is the best method to use in shell scripting.

C. Using the numeric method to set permissions is the best method to use in shell scripting.

D. It doesn't matter which method you use to set permissions.

1.  Which of the following commands would set the SUID permission on a file that has read/write/execute permissions for the user and group, and read/execute permissions for others?

A. **sudo chmod 2775 somefile**

B. **sudo chown 2775 somefile**

C. **sudo chmod 1775 somefile**

D. **sudo chmod 4775 somefile**

1.  Which of the following functions is served by setting the SUID permission on an executable file?

A. It allows any user to use that file.

B. It prevents accidental erasure of the file.

C. It allows "others" to have the same privileges as the "user" of the file.

D. It allows "others" to have the same privileges as the group that's associated with the file.

1.  Why shouldn't users set the SUID or SGID permissions on their own regular files?

A. It unnecessarily uses more hard drive space.

B. It could prevent someone from deleting the files if needed.

C. It could allow someone to alter the files.

D. It could allow an intruder to compromise the system.

1.  Which of the following `find` command options allows you to automatically perform a command on each file that `find` finds, without being prompted?

A. `-exec`

B. `-ok`

C. `-xargs`

D. `-do`

1.  True/False: For the best security, always use the `600` permission setting for every `.conf` file on the system.

A. True

B. False

1.  Which of the following is a true statement?

A. Prevent users from setting SUID on files by mounting the `/` partition with the `nosuid` option.

B. You must have the SUID permission set on certain system files for the operating system to function properly.

C. Executable files must never have the SUID permissions set.

D. Executable files should always have the SUID permission set.

1.  Which two of the following are security concerns for configuration files?

A. With a default configuration, any normal user with command-line access can edit configuration files.

B. Certain configuration files may contain sensitive information.

C. With a default configuration, any normal user with command-line access can view configuration files.

D. The configuration files on servers require more protection than the configuration files on IoT devices.

## Further reading

*   How to find files with SUID and SGID permissions in Linux: [https://www.tecmint.com/how-to-find-files-with-suid-and-sgid-permissions-in-linux/](https://www.tecmint.com/how-to-find-files-with-suid-and-sgid-permissions-in-linux/)
*   The Linux `find` command: [https://youtu.be/tCemsQ_ZjQ0](https://youtu.be/tCemsQ_ZjQ0)
*   Linux and Unix file permissions: [https://youtu.be/K9FEz20Zhmc](https://youtu.be/K9FEz20Zhmc)
*   Linux file permissions: [https://www.linux.com/tutorials/understanding-linux-file-permissions/](https://www.linux.com/tutorials/understanding-linux-file-permissions/)
*   25 simple examples of the Linux `find` command: [https://www.binarytides.com/linux-find-command-examples/](https://www.binarytides.com/linux-find-command-examples/)
*   35 practical examples of the Linux `find` command: [https://www.tecmint.com/35-practical-examples-of-linux-find-command/](https://www.tecmint.com/35-practical-examples-of-linux-find-command/)

## Answers

1.  C
2.  D
3.  C
4.  B
5.  A
6.  A
7.  C
8.  D
9.  C
10.  D
11.  A
12.  B
13.  B
14.  B, C