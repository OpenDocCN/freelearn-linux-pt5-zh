# Chapter 6. Users – Do We Really Want Them?

The question, I admit, is more than a little rhetorical but there are times when we dream of how great our lives would be without pesky users getting in the way and gumming up the cogs that make our systems run; however, the better we manage the user base on our Linux systems, the better CentOS will be to us. In reality, the more control we have over the users, the more life is better for them as they can continue their work uninterrupted by system downtime.

In this chapter, we will develop techniques to maintain unobtrusive control of our systems, keeping ourselves sane and the users happy. This will include:

*   **Managing public and private groups**: Understanding how we can use public groups and overriding the CentOS default of private groups can give us more scope in assigning permissions to users. We have to be aware of the potential pitfalls to each solution.
*   **Getent**: This can provide us with a global view of our users and groups and will open up to us in this chapter the understanding of the name services switch file: `nsswitch.conf`.
*   **Quotas**: Using quotas can allow us to both monitor and if required, restrict users' space to a partition, and is truly important where users' home directories are located.
*   **Scripting user creation**: When creating a user, we will need to set the password and possibly user's disk quota limits; it makes sense then to combine all these activities into a script so that nothing is forgotten.

# Managing public and private groups

The Red Hat and, therefore, the CentOS user management systems deploy a private user group system. Each user created will also belong to an eponymous primary group; in other words, creating a user bob will also create a group bob, to which the user will be the only member.

## Linux groups

Firstly, we have to understand a little about Linux groups. A user has both a primary group and secondary groups.

**User ID and group ID** (**UID/GID**) are used with the permission management structure in Linux. Every file in any filesystem will be owned by a user and a group by means of storing the UID and GID in the files metadata. Permissions can be assigned to the user, group, or others.

Each user has one UID and GID but belongs to just one group, which is a little restrictive, so users additionally have secondary groups. Users can change their current GID to one from their secondary groups using the `/usr/bin/newgrp` command, effectively switching their GID. In practice, this is not required and leads us to describing the differences between the users' primary group and secondary groups.

When creating a new file, the users UID and their current GID are used to create the ownership of the new file. If a user creates a new file, he/she will be the owner of that file and the file will be group owned by his/her own private group, creating an inherently secure system without the need of user intervention. Secondary groups are used in all other situations when accessing resources that *currently* exist. Users present all of their secondary groups when accessing a resource. In this way, a file that is readable by the `users` group but not to `others` will be accessible to a user whose GID is set to his/her own private group, but the list of secondary groups to which they belong includes the `users` group.

When assessing a user's ID, setting the `/usr/bin/id` command can be very useful. Run without any options or arguments and the output will display your own associated IDs. In the following screenshot, we can see that the user `andrew` belongs to only the private user group and has no additional secondary group memberships:

```
$ id

```

![Linux groups](img/5920OS_06_01.jpg)

We will use the same command but this time we will use the user, `u1`, as an argument. The output will show the associated IDs of that account; this command can be run as a standard user:

```
$ id u1

```

From the following screenshot, we can see that the user, `u1`, has the primary group or GID assigned to the private group `u1`; however, `u1` additionally belongs to the `users` group.

![Linux groups](img/5920OS_06_02.jpg)

With the current IDs in place for the user `u1`, any new file created will be group owned by GID 501 (`u1`), but `u1` can access any resource accessible to the `users` and `u1` groups without any additional action on `u1`'s part. From an administrative perspective, we need to make sure we assign the correct secondary IDs to our users.

The same cannot be said for the first example that we looked at. The user, `andrew`, currently belongs only to `andrew`'s private group, so he can only access resources where permissions are set to:

*   Their UID (`andrew`)
*   Their private GID (`andrew`)
*   Others

The user account `andrew` does not have access to permissions assigned to the `users` group in the same way that the user `u1` does.

## Adding users to groups

We can now see that the user `u1` has the desired access to resources shared with the `users` groups, but what about `andrew`? How can we help here?

If the user already exists and we need to add him/her to a public group, then we can use the `usermod` command to add the user to an additional group. When we add `andrew` to the `users` group, we will also want to maintain any existing secondary groups' memberships.

Run the following command:

```
# usermod -G users andrew

```

If we choose to run the preceding command, then `andrew` would be added to the `users` groups but, along with his primary group, this would be his only secondary group membership. In other words, if `andrew` belongs to multiple secondary groups, the `-G` option overwrites this group list, which is not a good thing.

The command ID can display current secondary groups with the `-G` option:

```
# id -G andrew

```

If we combine the two commands together, then we can effectively append the `users` groups to the current group list of `andrew`. To do this, additionally, we have to translate the spaces in the group list supplied by the command ID into commas:

```
# usermod -G$(id -G andrew | tr ' ' ','),users

```

The commands in the parenthesis are evaluated first. The `id` command creates a space-separated list of secondary groups, and the `tr` command will translate the spaces to commas (in this case). The group list we supply to `usermod` needs to be comma delimited but can use group names or IDs. More simply though, we can use the append option to usermod as shown in the following code example:

```
# usermod -a -G users andrew

```

When creating new users, we can simply specify the secondary groups the user should belong to. We don't need to concern ourselves with the existing group membership:

```
# useradd -G users u4
# id u4

```

From the following output, we can see that the new user, `u4`, is created and added to the secondary group users.

![Adding users to groups](img/5920OS_06_03.jpg)

## Evaluating private group usage

You do not need to use private `groups` schemes. They are the default, but as with all defaults, we can specify options to modify this. Using the `-N` option with `useradd` will not create the private groups and, if not specified, the user's primary group or GID will be the `users` groups. Let's execute the following commands:

```
# useradd -N u5
# id u5

```

The output is shown in the following screenshot, and we see that the users' primary group is the `users` group:

![Evaluating private group usage](img/5920OS_06_04.jpg)

The only security issue that we may need to contend with is that now, by default, any file created by the user `u5` will be group owned by a shared group. Depending on the circumstances, this may be not desirable; however, having all files private to the user by default is no more desirable either. This is up to the administration team deciding which model suits the organizational needs best.

# Getent

The `/usr/bin/getent` command will display a list of entries, *Get Entries*. The entries are resolved by *Name Service Switch Libraries*, which are configured in the `/etc/nsswitch.conf` file. This file has a list of databases and libraries that will be used to access those databases.

For example, we could use the `getent passwd` command to display all users, or `getent group` to display all groups. We could extend this though to commands such as `getent hosts` to display host file entries and `getent aliases` to display user aliases on the system.

The `nsswitch.conf` file will define the libraries used to access the `passwd` database. On a standard CentOS system, `/etc/passwd` is often the only local file, but an enterprise system could include **Lightweight Directory Access Protocol** (**LDAP**) modules. In the next chapter, we will learn more using directory services.

We search the `/etc/nsswitch` file for the `passwd` database using `grep`:

```
# grep passwd /etc/nsswitch.conf

```

We can see that on my system, we just use the local files to resolve user names:

![Getent](img/5920OS_06_05.jpg)

The `getent` command is a very useful way to quickly list users or groups on your system, and the output can be filtered or sorted as required with the `grep` and `sort` commands. For example, if we want to see all configured groups on our system that start with the letter `u` and have only one additional character in their names, we can use the following command:

```
# getent group | grep 'u.:' | sort

```

The following screenshot shows this command:

![Getent](img/5920OS_06_06.jpg)

# Quotas

In almost all areas of user management, we have to assign disk space quotas of some description in order to give the responsibility of disk space management back to the user. If we do not, then the user would never know the struggles that we have to face in providing them with unlimited disk space. Allowing the user to see that their space is filling up then may prompt them to carry out a little housekeeping.

In Linux, disk quotas are applied to the mount points; if you want to limit a user's space in their home directory, then the `/home` directory will need to be in its own partition. If it is part of the root filesystem, then a user's space will be restricted to all directories within that partition.

Quota restrictions are implemented using tools from the `quota` package. You can use the `yum` command to verify that it is installed:

```
$ yum list quota

```

If the output of the command indicates that it is available rather than installed, then install the quota with:

```
# yum install quota

```

## Setting quotas

My system includes a partition for `/home` and has the `quota` package installed. We now need to set the correct mount options for the `/home` partition. Currently, it does not include quotas.

To enable this, we will edit the `/etc/fstab` file and mount options for the `/home` partition. The following two mount options should be added to enable journal quotas for a selected partition:

```
usrjquota=aquota.user,jqfmt=vfsv0

```

The `usrjquota=aquota.user` part specifies the quota file, and `jqfmt=vfsv0` specifies the quota format.

The line in question is shown in the following screenshot:

![Setting quotas](img/5920OS_06_07.jpg)

We have enabled journal-based user quotas as we are using `ext4`, a journal-based filesystem. User space restriction is checked when writing the journal rather than waiting until the changes are flushed to disk. We also set the format of the journal quotas.

To make these settings effective, we can remount the `/home` partition using the following command:

```
# mount -o remount /home

```

We will now need to initialize the quota database; this was referenced in the mount options as `aquota.user` and will reside at the root of the partition where quotas are enabled. Enabling quotas on a filesystem may take some time, depending on the amount of data in the filesystem:

```
#quotacheck -muv /home

```

Using these options with the `/sbin/quotacheck` command, we can set the following options:

*   `-m`: This indicates not to remount as read-only during an operation
*   `-u`: This is for user quotas
*   `-v`: This is the verbose output
*   `/home`: This is the partition to work with, or use `-a` for all quota-enabled partitions

It may be worth adding the `quotacheck` commands and options into your `crontab` to ensure that `quotacheck` is run perhaps once a day. Even though journal quotas are more reliable than traditional quotas, there is no harm in re-evaluating file space used to ensure that the data maintained is accurate.

Quotas can be set with the `edquota` or `setquota` command; I prefer the `setquota` command, but traditionally `edquota` is taught to new administrators. The `/usr/sbin/edquota` command takes you into your editor to make the changes, whereas `/usr/sbin/setquota` sets the quota directly from the command line:

```
# setquota -u u1 20000 25000 0 0 /home

```

The preceding command will set the quota for the user `u1`. Giving the user a soft limit, just a warning when they exceed 20 M (20 x 1k blocks) and implementing a hard limit of 25 M, where the user cannot save any more data in `/home`. I have not limited the user `u1` with either soft or hard limits to the number of files they may have, just the space they use.

The `/usr/sbin/repquota` command can be used to display disk usage:

```
# repquota -uv /home

```

The output from my system is shown in the following screenshot:

![Setting quotas](img/5920OS_06_08.jpg)

# Scripting user creation

User creation will now consist of three steps:

*   `useradd`: This creates the user
*   `passwd`: This sets the password
*   `setquota`: This sets the disk limits

We can ensure that all this happens correctly and uniformly using scripts to ensure the procedural integrity of the user creation process. It is also going to save you time. As a very quick solution, the following script provides all that we need:

```
#!/bin/bash
useradd -m -G users $1 
echo Password123 | passwd --stdin $1
passwd -e $1
setquota -u $1 20000 25000 0 0 /home

```

We will need to run the script with the new username as the argument, as shown in the following example:

```
# userscript.sh bob

```

Reading the script though line by line can explain the script contents as follows:

*   `#!/bin/bash`: This is the script interpreter to use
*   `useradd -m -G users $1`: This creates the user supplied as the first argument to the script, `$1`. The user's home directory will be created, and it will be added to the `users` group.
*   `echo Password123 | passwd --stdin $1`: This sets the user's password to a standard password.
*   `passwd -e $1`: The password is expired so the user will need to set their own password when they first login.
*   `setquota -u $1 20000 25000 0 0 /home`: Finally, the quotas are implemented for the user.

We can, of course, allow more functionality in the script to set different groups and quotas; however, as an example of procedural integrity and a functional script, this is a great start.

# Summary

As we close another chapter, we can take stock of all that we have acquainted ourselves with in the process. The big task for this section was to become more accustomed to the vagaries of CentOS group management and being able to properly differentiate between the primary group and secondary groups of a user. During this process, we took the time to evaluate the use of public and private group schemes and the use of the `-N` option to disable the user's private group during user creation.

It was not long before we found ourselves in the depths of `/etc/nsswitch.conf` and the `getent` command (get entries). From here, we got down straight to business implementing user disk limits or quotas before seeing how to link all of this together with scripts.

In the next chapter, we stick to the theme of users, but look at centralizing our accounts in a central LDAP directory, using the open source code from Red Hat's directory server by implementing the 389 Directory Server on CentOS 6.5.