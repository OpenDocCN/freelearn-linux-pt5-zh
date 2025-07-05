# 9 Access Control Lists and Shared Directory Management

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file59.png)

In the previous chapter, we reviewed the basics of **Discretionary Access Control** (**DAC**). Normal Linux file and directory permissions settings aren't very granular. With an **access control list** (**ACL**), we can fine-tune things to get the exact set of permissions that we really want. We can also use this capability to control access to files in shared directories.

The topics in this chapter include the following:

*   Creating an ACL for either a user or a group
*   Creating an inherited ACL for a directory
*   Removing a specific permission by using an ACL mask
*   Using the `tar --acls` option to prevent loss of ACLs during a backup
*   Creating a user group and adding members to it
*   Creating a shared directory for a group, and setting the proper permissions on it
*   Setting the SGID bit and the sticky bit on the shared directory
*   Using ACLs to allow only certain members of the group to access a file in the shared directory

## Creating an ACL for either a user or a group

The normal Linux file and directory permissions settings are okay, but they're not very granular. With an ACL, we can allow only a certain person to access a file or directory, or we can allow multiple people to access a file or directory with different permissions for each person. If we have a file or a directory that's wide open for everyone, we can use an ACL to allow different levels of access for either a group or an individual. Toward the end of the chapter, we'll put what we've learned all together in order to manage a shared directory for a group.

You would use `getfacl` to view an ACL for a file or directory. (Note that you can't use them to view all files in a directory at once.) To begin, let's use `getfacl` to see if we have any ACLs already set on the `acl_demo.txt` file:

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

All we see here are just the normal permissions settings, so there's no ACL.

The first step for setting an ACL is to remove all permissions from everyone except for the user of the file. That's because the default permissions settings allow members of the group to have read/write access, and others to have read access. So, setting an ACL without removing those permissions would be rather senseless:

```
[donnie@localhost ~]$ chmod 600 acl_demo.txt
[donnie@localhost ~]$ ls -l acl_demo.txt
-rw-------. 1 donnie donnie 0 Nov  9 14:37 acl_demo.txt
[donnie@localhost ~]$
```

When using `setfacl` to set an ACL, you can allow a user or a group to have any combination of read, write, or execute privileges. In our case, let's say that I want to let Maggie read the file and to prevent her from having write or execute privileges:

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

The `-m` option of `setfacl` means that we're about to modify the ACL. (Well, to *create* one in this case, but that's okay.) `u:` means that we're setting an ACL for a user. We then list the user's name, followed by another colon, and the list of permissions that we want to grant to this user. In this case, we're only allowing Maggie read access. We complete the command by listing the file to which we want to apply this ACL. The `getfacl` output shows that Maggie does indeed have read access. Finally, we see in the `ls -l` output that the group is listed as having read access, even though we've set the `600` permissions settings on this file. But, there's also a `+` sign, which tells us that the file has an ACL. When we set an ACL, the permissions for the ACL show up as group permissions in `ls -l`.

To take this a step further, let's say that I want Frank to have read/write access to this file:

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

So, we can have two or more different ACLs assigned to the same file. In the `ls -l` output, we see that we have `rw` permissions set for the group, which is really just a summary of permissions that we've set in the two ACLs.

We can set an ACL for group access by replacing `u:` with a `g:`:

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

Members of the `accounting` group now have read access to this file.

## Creating an inherited ACL for a directory

There may be times when you'll want all files that get created in a shared directory to have the same ACL. We can do that by applying an inherited ACL to the directory. Although, understand that even though this sounds like a cool idea, creating files in the normal way will cause files to have the read/write permissions set for the group, and the read permission set for others. So, if you're setting this up for a directory where users just create files normally, the best that you can hope to do is to create an ACL that adds either the write or execute permissions for someone. Either that or ensure that users set the `600` permissions settings on all files that they create, assuming that users really do need to restrict access to their files.

On the other hand, if you're creating a shell script that creates files in a specific directory, you can include `chmod` commands to ensure that the files get created with the restrictive permissions that are necessary to make your ACL work as intended.

To demo, let's create the `new_perm_dir` directory, and set the inherited ACL on it. I want to have read/write access for files that my shell script creates in this directory, and for Frank to have only read access. I don't want anyone else to be able to read any of these files:

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

All I had to do to make this an inherited ACL was to add `d:` before `u:frank`. I left the default permissions settings on the directory, which allows everyone read access to the directory. Next, I'll create the `donnie_script.sh` shell script, which will create a file within that directory, and that will set read/write permissions for only the user of the new files:

```
#!/bin/bash
cd new_perm_dir
touch new_file.txt
chmod 600 new_file.txt
exit
```

After making the script executable, I'll run it and view the results:

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

So, `new_file.txt` got created with the correct permissions settings, and with an ACL that allows Frank to read it. (I know that this is a really simplified example, but you get the idea.)

## Removing a specific permission by using an ACL mask

You can remove an ACL from a file or directory with the `-x` option. Let's go back to the `acl_demo.txt` file that I created earlier, and remove the ACL for Maggie:

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

So, Maggie's ACL is gone. But, the `-x` option removes the entire ACL, even if that's not what you really want. If you have an ACL with multiple permissions set, you might just want to remove one permission, leaving the others. Here, we see that Frank still has his ACL that grants him read/write access. Let's now say that we want to remove the write permission, while still allowing him the read permission. For that, we'll need to apply a mask:

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

`m::r` sets a read-only mask on the ACL. Running `getfacl` shows that Frank still has a read/write ACL, but the comment to the side shows his effective permissions to be read-only. So, Frank's write permission for the file is now gone. And, if we had ACLs set for other users, this mask would affect them the same way.

## Using the tar --acls option to prevent the loss of ACLs during a backup

If you ever need to use `tar` to create a backup of either a file or a last two files:

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

Now, I'll do the backup without `--acls`:

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

It looks good, right? Ah, but looks can be deceiving. Watch what happens when I delete the directory, and then restore it from the backup:

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

I don't even have to use `getfacl` to see that the ACLs are gone from the `perm_demo_dir` directory and all of its files, because the `+` signs are now gone from them. Now, let's see what happens when I include the `--acls` option. First, I'll show you that an ACL is set for this directory and its only file:

```
[donnie@localhost ~]$ ls -ld new_perm_dir
drwxrwxr-x+ 2 donnie donnie 26 Nov 13 14:01 new_perm_dir
[donnie@localhost ~]$ ls -l new_perm_dir
total 0
-rw-------+ 1 donnie donnie 0 Nov 13 14:01 new_file.txt
[donnie@localhost ~]$
```

Now, I'll use the `tar` command with the `--acls` option:

```
[donnie@localhost ~]$ tar cJvf new_perm_dir_backup.tar.xz new_perm_dir/ --acls
new_perm_dir/
new_perm_dir/new_file.txt
[donnie@localhost ~]$
```

I'll now delete the `new_perm_dir` directory and restore it from backup. As we did before, we’ll use the `--acls` option:

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

The presence of the `+` signs indicates that the ACLs did survive the backup-and-restore procedure. The one slightly tricky part about this is that you must use `--acls` for both the backup and the restoration. If you omit the option either time, you will lose your ACLs.

## Creating a user group and adding members to it

So far, I've been doing all of the demos inside my own home directory, just for the sake of showing the basic concepts. But the eventual goal is to show you how to use this knowledge to do something more practical, such as controlling file

Let's say that we want to create a `marketing` group for members of—you guessed it—the marketing department:

```
[donnie@localhost ~]$ sudo groupadd marketing
[sudo] password for donnie:
[donnie@localhost ~]$
```

Let's now add some members. We can do that in three different ways:

*   Add members as we create their user accounts.
*   Use `usermod` to add members that already have user accounts.
*   Edit the `/etc/group` file.

### Adding members as we create their user accounts

First, we can add members to the group as we create their user accounts, using the `-G` option of `useradd`. On Red Hat, AlmaLinux, or CentOS, the command would look like this:

```
[donnie@localhost ~]$ sudo useradd -G marketing cleopatra
[sudo] password for donnie:
[donnie@localhost ~]$ groups cleopatra
cleopatra : cleopatra marketing
[donnie@localhost ~]$
```

On Debian/Ubuntu, the command would look like this:

```
donnie@ubuntu3:~$ sudo useradd -m -d /home/cleopatra -s /bin/bash -G marketing cleopatra
donnie@ubuntu3:~$ groups cleopatra
cleopatra : cleopatra marketing
donnie@ubuntu3:~$
```

And, of course, I'll need to assign Cleopatra a password in the normal manner:

```
[donnie@localhost ~]$ sudo passwd cleopatra
```

### Using usermod to add an existing user to a group

The good news is that this works the same on either Red Hat/CentOS/AlmaLinux or Debian/Ubuntu:

```
[donnie@localhost ~]$ sudo usermod -a -G marketing maggie
[sudo] password for donnie:
[donnie@localhost ~]$ groups maggie
maggie : maggie marketing
[donnie@localhost ~]$
```

In this case, the `-a` wasn't necessary, because Maggie wasn't a member of any other secondary group. But, if she had already belonged to another group, the `-a` would have been necessary to keep from overwriting any existing group information, thus removing her from the previous groups.

This method is especially handy for use on Ubuntu systems, where it is necessary to use `adduser` in order to create encrypted home directories. (As we saw in a previous chapter, `adduser` doesn't give you the chance to add a user to a group as you create the account.)

### Adding users to a group by editing the /etc/group file

This final method is a good way to cheat, to speed up the process of adding multiple existing users to a group. First, just open the `/etc/group` file in your favorite text editor, and look for the line that defines the group to which you want to add members:

```
. . .
marketing:x:1005:cleopatra,maggie
. . .
```

So, I've already added Cleopatra and Maggie to this group. Let's edit this to add a couple more members:

```
. . .
marketing:x:1005:cleopatra,maggie,vicky,charlie
. . .
```

When you're done, save the file and exit the editor.

A `groups` command for each of them will show that our wee bit of cheating works just fine:

```
[donnie@localhost etc]$ sudo vim group
[donnie@localhost etc]$ groups vicky
vicky : vicky marketing
[donnie@localhost etc]$ groups charlie
charlie : charlie marketing
[donnie@localhost etc]$
```

This method is extremely handy for whenever you need to add lots of members to a group at the same time.

## Creating a shared directory

The next act in our scenario involves creating a shared directory that all the members of our marketing department can use. Now, this is another one of those areas that engenders a bit of controversy. Some people like to put shared directories in the root level of the filesystem, while others like to put shared directories in the `/home/` directory. Some people even have other preferences. But really, it's a matter of personal preference and/or company policy. Other than that, it really doesn't much matter where you put them. For our purposes, to make things simple, I'll just create the directory in the root level of the filesystem:

```
[donnie@localhost ~]$ cd /
[donnie@localhost /]$ sudo mkdir marketing
[sudo] password for donnie:
[donnie@localhost /]$ ls -ld marketing
drwxr-xr-x. 2 root root 6 Nov 13 15:32 marketing
[donnie@localhost /]$
```

The new directory belongs to the root user. It has a permissions setting of `755`, which permits read and execute access to everybody, and write access only to the root user. What we really want is to allow only members of the marketing department to access this directory. We'll first change ownership and group association, and then we'll set the proper permissions:

```
[donnie@localhost /]$ sudo chown nobody:marketing marketing
[donnie@localhost /]$ sudo chmod 770 marketing
[donnie@localhost /]$ ls -ld marketing
drwxrwx---. 2 nobody marketing 6 Nov 13 15:32 marketing
[donnie@localhost /]$
```

In this case, we don't have any one particular user that we want to own the directory, and we don't really want the root user to own it. So, assigning ownership to the `nobody` pseudo-user account gives us a way to deal with that. I then assigned the `770` permissions value to the directory, which allows read/write/execute access to all `marketing` group members, while keeping everyone else out. Now, let's let one of our group members log in to see if she can create a file in this directory:

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

Okay, it works, except for one minor problem. The file belongs to Vicky, as it should. But, it's also associated with Vicky's personal group. For the best access control of these shared files, we need them to be associated with the `marketing` group. Let’s take care of that next.

## Setting the SGID bit and the sticky bit on the shared directory

I've told you before that it's a bit of a security risk to set either the SUID or SGID permissions on files, especially on executable files. But it is both completely safe and very useful to set SGID on a shared directory.

SGID behavior on a directory is completely different from SGID behavior on a file. On a directory, SGID will cause any files that anybody creates to be associated with the same group with which the directory is associated. So, bearing in mind that the SGID permission value is `2000`, let's set SGID on our `marketing` directory:

```
[donnie@localhost /]$ sudo chmod 2770 marketing
[sudo] password for donnie:
[donnie@localhost /]$ ls -ld marketing
drwxrws---. 2 nobody marketing 28 Nov 13 15:41 marketing
[donnie@localhost /]$
```

The `s` in the executable position for the group indicates that the command was successful. Let's now let Vicky log back in to create another file:

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

Vicky's second file is associated with the `marketing` group, which is just what we want. Just for fun, let's let Charlie do the same:

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

Again, Charlie's file is associated with the `marketing` group. But, for some strange reason that nobody understands, Charlie really doesn't like Vicky and decides to delete her files, just out of pure spite:

```
[charlie@localhost marketing]$ rm vicky*
rm: remove write-protected regular empty file ‘vicky_file.txt’? y
[charlie@localhost marketing]$ ls -l
total 0
-rw-rw-r--. 1 charlie marketing 0 Nov 13 15:59 charlie_file.txt
[charlie@localhost marketing]$
```

The system complains that Vicky's original file is write-protected since it's still associated with her personal group. But the system does still allow Charlie to delete it, even without `sudo` privileges. And, since Charlie has write access to the second file, due to its association with the `marketing` group, the system allows him to delete it without question.

Okay. So, Vicky complains about this and tries to get Charlie fired. But our intrepid administrator has a better idea. He'll just set the sticky bit in order to keep this from happening again. Since the SGID bit has a value of `2000`, and the sticky bit has a value of `1000`, we can just add the two together to get a value of `3000`:

```
[donnie@localhost /]$ sudo chmod 3770 marketing
[sudo] password for donnie:
[donnie@localhost /]$ ls -ld marketing
drwxrws--T. 2 nobody marketing 30 Nov 13 16:03 marketing
[donnie@localhost /]$
```

The `T` in the executable position for others indicates that the sticky bit has been set. Since the `T` is uppercase, we know that the executable permission for others has not been set. Having the sticky bit set will prevent group members from deleting anybody else's files. Let's let Vicky show us what happens when she tries to retaliate against Charlie:

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

Even with the `-f` option, Vicky still can't delete Charlie's file. Vicky doesn't have `sudo` privileges on this system, so it would be useless for her to try that.

## Using ACLs to access files in the shared directory

As things currently stand, all members of the `marketing` group have read/write access to all other group members' files. Restricting access to a file to only specific group members is the same two-step process that we've already covered.

### Setting the permissions and creating the ACL

First, Vicky sets the normal permissions to only allow herself to have read/write permissions on the file. Then, she’ll create an ACL that will allow Cleopatra to read the file:

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

There's nothing here that you haven't already seen. Vicky just removed all permissions from the group and from others and set an ACL that only allows Cleopatra to read the file. Let's see if Cleopatra actually can read it:

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

So far, so good. But, can Cleopatra write to it? Let's take a look:

```
[cleopatra@localhost marketing]$ echo "You are my friend too, Vicky." >> vicky_file.txt
-bash: vicky_file.txt: Permission denied
[cleopatra@localhost marketing]$
```

Cleopatra can't do that, since Vicky only allowed her the read privilege in the ACL.

Now though, what about that sneaky Charlie, who wants to go snooping in other users' files? Let's see if Charlie can do it:

```
[donnie@localhost /]$ su - charlie
Password:
Last login: Mon Nov 13 15:58:56 EST 2017 on pts/0
[charlie@localhost ~]$ cd /marketing
[charlie@localhost marketing]$ cat vicky_file.txt
cat: vicky_file.txt: Permission denied
[charlie@localhost marketing]$
```

So yes, it's really true that only Cleopatra can access Vicky's file, and even then only for reading.

#### Hands-on lab – creating a shared group directory

For this lab, you'll just put together everything that you've learned in this chapter to create a shared directory for a group. You can do this on any of your virtual machines:

1.  On any virtual machine, create the `sales` group:

```
sudo groupadd sales
```

1.  Create the users `mimi`, `mrgray`, and `mommy`, adding them to the `sales` group as you create the accounts.

On CentOS or AlamaLinux, do this:

```
sudo useradd -G sales mimi
sudo useradd -G sales mrgray
sudo useradd -G sales mommy
```

On Ubuntu, do this:

```
sudo useradd -m -d /home/mimi -s /bin/bash -G sales mimi
sudo useradd -m -d /home/mrgray -s /bin/bash -G sales mrgray
sudo useradd -m -d /home/mommy -s /bin/bash -G sales mommy
```

1.  Assign each user a password.
2.  Create the `sales` directory in the root level of the filesystem. Set proper ownership and permissions, including the SGID and sticky bits:

```
sudo mkdir /sales
sudo chown nobody:sales /sales
sudo chmod 3770 /sales
ls -ld /sales
```

1.  Log in as Mimi, and have her create a file:

```
su - mimi
cd /sales
echo "This file belongs to Mimi." > mimi_file.txt
ls -l
```

1.  Have Mimi set an ACL on her file, allowing only Mr. Gray to read it. Then, have Mimi log back out:

```
chmod 600 mimi_file.txt
setfacl -m u:mrgray:r mimi_file.txt
getfacl mimi_file.txt
ls -l
exit
```

1.  Have Mr. Gray log in to see what he can do with Mimi's file. Then, have Mr. Gray create his own file and log back out:

```
su - mrgray
cd /sales
cat mimi_file.txt
echo "I want to add something to this file." >> mimi_file.txt    
echo "Mr. Gray will now create his own file." > mr_gray_file.txt        
ls -l
exit
```

1.  Mommy will now log in and try to wreak havoc by snooping in other users' files and by trying to delete them:

```
su - mommy
cat mimi_file.txt
cat mr_gray_file.txt
rm -f mimi_file.txt
rm -f mr_gray_file.txt
exit
```

1.  End of lab.

## Summary

In this chapter, we saw how to take DAC to the proverbial next level. We first saw how to create and manage ACLs to provide more fine-grained access control over files and directories. We then saw how to create a user group for a specific purpose, and how to add members to it. Then, we saw how we can use the SGID bit, the sticky bit, and ACLs to manage a shared group directory.

But sometimes, DAC might not be enough to do the job. For those times, we also have mandatory access control, which we'll cover in the next chapter. I'll see you there.

## Questions

1.  When creating an ACL for a file in a shared directory, what must you first do to make the ACL effective?

A. Remove all normal permissions from the file for everyone except for the user.

B. Ensure that the file has the permissions value of `644` set.

C. Ensure that everyone in the group has read/write permissions for the file.

D. Ensure that the SUID permission is set for the file.

1.  What is the benefit of setting the SGID permission on a shared group directory?

A. None. It's a security risk and should never be done.

B. It prevents members of the group from deleting each others' files.

C. It makes it so that each file that gets created within the directory will be associated with the group that's also associated with the directory.

D. It gives anyone who accesses the directory the same privileges as the user of the directory.

1.  Which of the following commands would set the proper permissions for the `marketing` shared group directory, with the SGID and sticky bit set?

A. **sudo chmod 6770 marketing**

B. **sudo chmod 3770 marketing**

C. **sudo chmod 2770 marketing**

D. **sudo chmod 1770 marketing**

1.  Which of the following `setfacl` options would you use to just remove one specific permission from an ACL?

A. `-xB. -r`

C. `-w`

D. `m: :`

E. `-m`

F. `x: :`

1.  Which of the following statements is true?

A. When using `tar`, you must use the `--acls` option for both archive creation and extraction, in order to preserve the ACLs on the archived files.

B. When using `tar`, you need to use the `--acls` option only for archive creation in order to preserve the ACLs on the archived files.

C. When using `tar`, ACLs are automatically preserved on archived files.

D. When using `tar`, it's not possible to preserve ACLs on archived files.

1.  Which two of the following are *not* a valid method for adding the user Lionel to the `sales` group?

A. **sudo useradd -g sales lionel**

B. **sudo useradd -G sales lionel**

C. **sudo usermod -g sales lionel**

D. **sudo usermod -G sales lionel**

E. By hand-editing the `/etc/group` file.

1.  What happens when you create an inherited ACL?

A. Every file that gets created in the directory with that inherited ACL will be associated with the group that's associated with that directory.

B. Every file that gets created in the directory with that inherited ACL will inherit that ACL.

C. Every file that gets created in that directory with that inherited ACL will have the same permissions settings as the directory.

D. Every file that gets created in that directory will have the sticky bit set.

1.  Which of the following commands would you use to grant read-only privilege on a file to the user Frank?

A. **chattr -m u:frank:r somefile.txt**

B. **aclmod -m u:frank:r somefile.txt**

C. **getfacl -m u:frank:r somefile.txt**

D. **setfacl -m u:frank:r somefile.txt**

1.  You've just done an `ls -l` command in a shared group directory. How can you tell from that whether an ACL has been set for any of the files?

A. Files with an ACL set will have `+` at the beginning of the permissions settings.

B. Files with an ACL set will have `-` at the beginning of the permissions settings.

C. Files with an ACL set will have `+` at the end of the permissions settings.

D. Files with an ACL set will have `-` at the end of the permissions settings.

E. The `ls -l` command will show the ACL for that file.

1.  Which of the following would you use to view the ACL on the `somefile.txt` file?

A. **getfacl somefile.txt**

B. **ls -l somefile.txt**

C. **ls -a somefile.txt**

D. **viewacl somefile.txt**

## Further reading

*   How to create users and groups from the Linux command line: [https://www.techrepublic.com/article/how-to-create-users-and-groups-in-linux-from-the-command-line/](https://www.techrepublic.com/article/how-to-create-users-and-groups-in-linux-from-the-command-line/)
*   Add a user to a group: [https://www.howtogeek.com/50787/add-a-user-to-a-group-or-second-group-on-linux/](https://www.howtogeek.com/50787/add-a-user-to-a-group-or-second-group-on-linux/)
*   SGID on directories: [https://www.toptip.ca/2010/03/linux-setgid-on-directory.html](https://www.toptip.ca/2010/03/linux-setgid-on-directory.html)
*   What a sticky bit is and how to set it in Linux: [https://www.linuxnix.com/sticky-bit-set-linux/](https://www.linuxnix.com/sticky-bit-set-linux/)

## Answers

1.  A
2.  C
3.  B
4.  D
5.  A
6.  A, C
7.  B
8.  D
9.  C
10.  A