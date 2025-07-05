# 4

# Managing Users and Groups

Linux is a multiuser, multitasking operating system, which means multiple users can access the operating system at the same time while sharing platform resources, with the kernel performing tasks for each user concurrently and independently. Linux provides the required isolation and security mechanisms to avoid multiple users accessing or deleting each other’s files.

When multiple users are accessing the system, permissions come into play. We’ll learn how `root`) account, with complete access to the operating system resources.

Along the way, we’ll take a hands-on approach to the topics learned, further deepening the assimilation of key concepts through practical examples. This chapter covers the following topics:

*   Managing users
*   Managing groups
*   Managing permissions

We hope that by the end of the chapter, you will be comfortable with the command-line utilities for creating, modifying, and deleting users and groups, while proficiently handling file and directory permissions.

Let’s take a quick look at the technical requirements for this chapter.

# Technical requirements

You need a working Linux distribution installed on either a **virtual machine** (**VM**) or a desktop platform. In case you don’t have one already, [*Chapter 1*](B19682_01.xhtml#_idTextAnchor030), *Installing Linux*, will drive you through the related process. In this chapter, we’ll be using Ubuntu or Fedora, but most of the commands and examples used would pertain to any other Linux platform.

# Managing users

In this context, a **user** is anyone using a computer or a system resource. In its simplest form, a Linux *user* or *user account* is identified by a name and a **unique identifier**, known as a **UID**.

From a purely technical point of view, in Linux, we have the following types of users:

*   **Normal (or regular) users**: General-purpose, everyday user accounts, mostly suited for personal use and for common application and file management tasks, with limited access to system-wide resources. A regular user account usually has a *login* shell and a *home* directory.
*   `root` privileges. Consequently, possible vulnerabilities exposed through the web server would remain strictly isolated to the limited action realm of the associated system account.
*   `root` user is an example of a superuser.

In Linux, only the `root` user or users with `sudo` privileges (**sudoers**) can create, modify, or delete user accounts.

## Understanding sudo

The `root` user is the default superuser account in Linux, and it has the ability to do anything on a system. Ideally, acting as `root` on a system should generally be avoided due to safety and security reasons. With `sudo`, Linux provides a mechanism for *promoting* a regular user account to superuser privileges, using an additional layer of security. This way, a `sudo` user is generally used instead of `root`.

`sudo` is a command-line utility that allows a permitted user to execute commands with the security privileges of a superuser or another user (depending on the local system’s security policy). `sudo` originally stood for *superuser do* due to its initial implementation of acting exclusively as the superuser, but has since been expanded to support not only the superuser but also other (restricted) user impersonations. Thus, it is also referred to as *substitute user do*. Yet, more often than not, it is perceived as *superuser do* due to its frequent use in Linux administrative tasks.

Most of the command-line tools for managing users in Linux require `sudo` privileges unless the related tasks are carried out by the `root` user. If we want to avoid using the root context, we can’t genuinely proceed with the rest of this chapter—and create a user in particular—before we have a user account with superuser privileges. So, let’s take this chicken-and-egg scenario out of the way first.

Most Linux distributions create an additional user account with superuser privileges, besides `root`, during installation. The reason, as noted before, is to provide an extra layer of security and safety for elevated operations. The simplest way to check whether a user account has `sudo` privileges is to run the following command in a terminal, while logged in with the related user account:

```
sudo -v
```

According to the `sudo` manual (`man sudo`), the `-v` option causes `sudo` to update the user’s cached credentials and authenticate the user if the cached credentials expired.

If the user (for example, `julian`) doesn’t have superuser privileges on the local machine (for example, `neptune`), the preceding command yields the following (or a similar) error:

```
Sorry, user sudo command usually grants elevated permissions for a limited time. Ubuntu, for example, has a 15-minute sudo elevation span, after which time a sudo user would need to authenticate again. Subsequent invocations of sudo may not prompt for a password if done within the sudo cache credential timeout.
If we don’t have a default superuser account, we can always use the root context to create new users (see the next chapter) and elevate them to **sudoer** privileges. We’ll learn more about this in the *Creating a superuser* section, later in this chapter.
Now, let’s have a look at how to create, modify, and delete users.
Creating, modifying, and deleting users
In this section, we explore a few command-line tools and some common tasks for managing users. The example commands and procedures are shown for Ubuntu and Fedora, but the same principles apply to any other Linux distribution. Some user management `useradd` is not available on Alpine Linux, and `adduser` should be used instead). Please check the documentation of the Linux distribution of your choice for the equivalent commands.
Creating users
To create users, we can use either the `useradd` or the `adduser` command, although on some Linux distributions (for example, Debian or Ubuntu), the recommended way is to use the `adduser` command in favor of the low-level `useradd` utility. We’ll cover both in this section.
`adduser` is a Perl script using `useradd`—basically, a shim of the `useradd` command—with a user-friendly guided configuration. Both command-line tools are installed by default in Ubuntu and Fedora. Let’s take a brief look at each of these commands.
Creating users with useradd
The syntax for the `useradd` command is shown here:

```
useradd [OPTIONS] USER
```

 In its simplest invocation, the following command creates a user account (`julian`):

```
sudo useradd julian
```

 The user information is stored in a `/etc/passwd` file. Here’s the related user data for `julian`:

```
sudo cat /etc/passwd | grep julian
```

 In our case, this is the output:
![Figure 4.1 – The user record created with useradd](img/Figure_04_01_B19682.jpg)

Figure 4.1 – The user record created with useradd
Let’s analyze the related user record. Each entry is delimited by a colon (`:`) and is listed here:

*   `julian`: Username
*   `x`: Encrypted password (password hash is stored in `/etc/shadow`)
*   `1001`: The UID
*   `1001`: The user **group** **ID** (**GID**)
*   `::` The **General Electric Comprehensive Operating Supervisor** (**GECOS**) field—for example, display name (in our case, empty), explained later in this section
*   `/home/julian`: User home folder
*   `/bin/sh`: Default login shell for the user

Important note
The GECOS field is a string of comma-delimited attributes, reflecting general information about the user account (for example, real name, company, and phone number). In Linux, the GECOS field is the fifth field in a user record. See more information at [https://en.wikipedia.org/wiki/Gecos_field](https://en.wikipedia.org/wiki/Gecos_field).
We can also use the `getent` command to retrieve the preceding user information, as follows:

```
getent passwd julian
```

 To view the UID (`uid`), GID (`gid`), and group membership associated with a user, we can use the `id` command, as follows:

```
id julian
```

 This command gives us the following output:
![Figure 4.2 – The UID information](img/Figure_04_02_B19682.jpg)

Figure 4.2 – The UID information
With the simple invocation of `useradd`, the command creates the user (`julian`) with some immediate default values (as enumerated), while other user-related data is empty—for example, we have no full name or password specified for the user yet. Also, while the home directory has a default value (for example, `/home/julian`), the actual filesystem folder will not be created unless the `useradd` command is invoked with the `-m` or the `--create-home` option, as follows:

```
sudo useradd -m julian
```

 Without a home directory, regular users would not have the ability to save their files in a private location on the system. On the other hand, some system accounts may not need a home directory since they don’t have a login shell. For example, a database server (for example, PostgreSQL) may run with a non-root system account (for example, `postgres`) that only needs access to database resources in specific locations (for example, `/var/lib/pgsql`), controlled via other permission mechanisms (for example, **Security-Enhanced** **Linux** (**SELinux**)).
For our regular user, if we also wanted to specify a full name (display name), the command would change to this:

```
sudo useradd -m -c "Julian" julian
```

 The `-c, --comment` option parameter of `useradd` expects a *comment*, also known as the GECOS field (the fifth field in our user record), with multiple comma-separated values. In our case, we specify the full name (for example, `Julian`). For more information, check out the `useradd` manual (`man useradd`) or `useradd --help`.
The user still won’t have a password yet, and consequently, there would be no way for them to log in (for example, via a `julian`, we invoke the `passwd` command, like this:

```
sudo passwd julian
```

 You can see the following output:
![Figure 4.3 – Creating or changing the user password](img/Figure_04_03_B19682.jpg)

Figure 4.3 – Creating or changing the user password
The `passwd` command will prompt for the new user’s password. With the password set, there will be a new entry added to the `/etc/shadow` file. This file stores the secure password hashes (not the passwords!) for each user. Only superusers can access the content of this file. Here’s the command to retrieve the related information for the user `julian`:

```
sudo getent shadow julian
```

 You can also use the following command:

```
sudo cat /etc/shadow | grep julian
```

 The output of both commands is shown in the following screenshot:
![Figure 4.4 – Information about the user from the shadow file](img/Figure_04_04_B19682.jpg)

Figure 4.4 – Information about the user from the shadow file
Once the password has been set, in normal circumstances, the user can log in to the system (via SSH or GUI). If the Linux distribution has a GUI, the new user will show up on the login screen.
As noted, with the `useradd` command, we have low-level granular control over how we create user accounts, but sometimes we may prefer a more user-friendly approach. Enter the `adduser` command.
Creating users with adduser
The `adduser` command is a Perl wrapper for `useradd`. The syntax for this command is shown here:

```
adduser [OPTIONS] USER
```

 `sudo` may prompt for the superuser password. `adduser` will prompt for the new user’s password and other user-related information (as shown in *Figure 4**.5*).
Let’s create a new user account (`alex`) with `adduser`, as follows:

```
sudo adduser alex
```

 The preceding command yields the following output:
![Figure 4.5 – The adduser command](img/Figure_04_05_B19682.jpg)

Figure 4.5 – The adduser command
In Fedora, the preceding invocation of the `adduser` command will simply run without prompting the user for a password or any other information.
We can see the related user entry in `/etc/passwd` with `getent`, as follows:

```
getent passwd alex
```

 The following is the output:
![Figure 4.6 – Viewing user information with getent](img/Figure_04_06_B19682.jpg)

Figure 4.6 – Viewing user information with getent
In the preceding examples, we created a regular user account. Administrators or superusers can also elevate the privileges of a regular user to a superuser. Let’s see how in the following section.
Creating a superuser
When a regular user is given the power to run `sudo`, they become a superuser. Let’s assume we have a regular user created via any of the examples shown in the *Creating* *users* section.
Promoting the user to a superuser (or *sudoer*) requires a `sudo` group membership. In Linux, the `sudo` group is a reserved system group for users with elevated or `root` privileges. To make the user `julian` a sudoer, we simply need to add the user to the `sudo` group, like this (in Ubuntu):

```
sudo usermod -aG sudo julian
```

 The `-aG` options of `usermod` instruct the command to append (`-a, --append`) the user to the specified group (`-G, --group`)—in our case, `sudo`.
To verify our user is now a sudoer, first make sure the related user information reflects the `sudo` membership by running the following command:

```
id julian
```

 This gives us the following output:
![Figure 4.7 – Looking for the sudo membership of a user](img/Figure_04_07_B19682.jpg)

Figure 4.7 – Looking for the sudo membership of a user
The output shows that the `sudo` group membership (GID) in the `groups` tag is `27(sudo)`.
To verify the `sudo` access for the user `julian`, run the following command:

```
su - julian
```

 The preceding command prompts for the password of the user `julian`. A successful login would usually validate the superuser context. Alternatively, the user (`julian`) can run the `sudo -v` command in their terminal session to validate the `sudo` privileges. For more information on superuser privileges, see the *Understanding sudo* section earlier in the chapter.
With multiple users created, a system administrator may want to view or list all the users in the system. In the next section, we provide a few ways to accomplish this task.
Viewing users
There are a few ways for a superuser to view all users configured in the system. As previously noted, the user information is stored in the `/etc/passwd` and `/etc/shadow` files. Besides simply viewing these files, we can parse them and extract only the usernames with the following command:

```
cat /etc/passwd | cut -d: -f1 | less
```

 Alternatively, we can parse the `/etc/shadow` file, like this:

```
sudo cat /etc/shadow | cut -d: -f1 | less
```

 In the preceding commands, we read the content from the related files (with `cat`). Next, we piped the result to a delimiter-based parsing (with `cut`, on the `:` delimiter) and picked the first field `(-f1`). Finally, we chose a paginated display of the results, using the `less` command (to exit the command’s output, press *Q*).
Note the use of `sudo` for the `shadow` file since access is limited to superusers only, due to the sensitive nature of the password hash data. Alternatively, we can use the `getent` command to retrieve the user information.
The following command lists all the users configured in the system:

```
getent passwd
```

 The preceding command reads the `/etc/passwd` file. Alternatively, we can retrieve the same information from `/etc/shadow`, as follows:

```
sudo getent shadow
```

 For both commands, we can further pipe the `getent` output to `| cut -d: -f1` to list only the usernames, like this:

```
sudo getent shadow | cut -d: -f1 | less | column
```

 The output will be similar to this:
![Figure 4.8 – Viewing usernames](img/Figure_04_08_B19682.jpg)

Figure 4.8 – Viewing usernames
With new users created, administrators or superusers may want to change certain user-related information, such as password, password expiration, full name, or login shell. Next, we take a look at some of the most common ways to accomplish this task.
Modifying users
A superuser can run the `usermod` command to modify user settings, with the following syntax:

```
usermod [OPTIONS] USER
```

 The examples in this section apply to a user we previously created (`julian`) with the simplest invocation of the `useradd` command. As noted in the previous section, the related user record in `/etc/passwd` has no full name for the user, and the user has no password either.
Let’s change the following settings for our user (`julian`):

*   `Julian` (initially empty)
*   `/local/julian` (from default `/home/julian`)
*   `/bin/bash` (from default `/bin/sh`)

The command-line utility for changing all the preceding information is shown here:

```
sudo usermod -c "Julian" -d /local/julian -m -s /bin/bash julian
```

 Here are the command options, briefly explained:

*   `-c, --comment "Julian"`: The full username
*   `-d, --home local/julian`: The user’s new home directory
*   `-m, --move`: Move the content of the current home directory to the new location
*   `-s, --shell /bin/sh`: The user login shell

The related change, retrieved with the `getent` command, is shown here:

```
getent passwd julian
```

 We get the following output:
![Figure 4.9 – The user changes reflected with getent](img/Figure_04_09_B19682.jpg)

Figure 4.9 – The user changes reflected with getent
Here are a few more examples of changing user settings with the `usermod` command-line utility.
Changing the username
The `-l, --login` option parameter of `usermod` specifies a new login username. The following command changes the username from `julian` to `balog` (that is, first name to last name), as illustrated here:

```
sudo usermod -l "balog" julian
```

 In a production environment, we may have to add to the preceding command, as we may also want to change the display name and the home directory of the user (for consistency reasons). In a previous example in the *Creating users with useradd* section, we showcased the `-d, --home` and `-m, --move` option parameters, which would accommodate such changes.
Locking or unlocking a user
A superuser or administrator may choose to temporarily or permanently lock a specific user with the `-L, --lock` option parameter of `usermod`, as follows:

```
sudo usermod -L julian
```

 As a result of the preceding command, the login attempt for the user `julian` would be denied. Should the user try to SSH into the Linux machine, they would get a **Permission denied, please try again** error message. Also, the related username will be removed from the login screen if the Linux platform has a GUI.
To unlock the user, we invoke the `-U, --unlock` option parameter, as follows:

```
sudo usermod -U julian
```

 The preceding command restores system access for the user.
For more information on the `usermod` utility, please check out the related documentation (`man usermod`) or the command-line help (`usermod --help`).
Although the recommended way of modifying user settings is via the `usermod` command-line utility, some users may find it easier to manually edit the `/etc/passwd` file. The following section shows you how.
Modifying users via /etc/passwd
A superuser can also manually edit the `/etc/passwd` file to modify user data by updating the relevant line. Although the editing can be done with a text editor of your choice (for example, `nano`), we recommend the use of the `vipw` command-line utility for a safer approach. `vipw` enables the required locks to prevent possible data corruption—for example, in case a superuser performs a change at the same time regular users change their password.
The following command initiates the editing of the `/etc/passwd` file by also prompting for the preferred text editor (for example, `nano` or `vim`):

```
sudo vipw
```

 For example, we can change the settings for user `julian` by editing the following line:

```
julian:x:1001:1001:Julian,,,:/home/julian:/bin/bash
```

 The meaning of the colon (`:`)-separated fields was previously described in the *Creating users with useradd* section. Each of these fields can be manually altered in the `/etc/passwd` file, resulting in changes equivalent to the corresponding `usermod` invocation.
For more information on the `vipw` command-line utility, you can refer to the related system manual (`man vipw`).
Another relatively common administrative task for a user account is to change a password or set up a password expiration. Although `usermod` can change a user password via the `-p` or `--password` option, it requires an encrypted hash string (and not a cleartext password). Generating an encrypted password hash would be an extra step. An easier way is to use the `passwd` utility to change the password.
A superuser (administrator) can change the password of a user (for example, the user `julian`) with the following command:

```
sudo passwd julian
```

 The output will ask for the new password for the respective user. To change the expiration time of a password (the password age), the `chage` command is used. For example, to set a 30-day password age for the user `julian`, we will use the following command:

```
sudo chage -M 30 julian
```

 This will force the user `julian` to change their password every month. The password time availability is defined system-wide by the password policy. It is found inside the `/etc/login.defs` file, inside the `julian` as our example again):

```
sudo chage -d 0 julian
```

 This command will force the user `julian` to enter their own password the first time they log in to the system.
Sometimes, administrators are required to remove specific users from the system. The next section shows a couple of ways of accomplishing this task.
Deleting users
The most common way to remove users from the system is to use the `userdel` command-line tool. The general syntax of the `userdel` command is shown here:

```
userdel [OPTIONS] USER
```

 For example, to remove the user `julian`, a superuser would run the following command:

```
sudo userdel -f -r julian
```

 Here are the command options used:

*   `-f, --force`: Removes all files in the user’s home directory, even if not owned by the user
*   `-r, --remove`: Removes the user’s home directory and mail spool

The `userdel` command removes the related user data from the system, including the user’s home directory (when invoked with the `-f` or `--force` option) and the related entries in the `/etc/passwd` and `/``etc/shadow` files.
There is also an alternative way, which could be handy in some odd cleanup scenarios. The next section shows how.
Deleting users via /etc/passwd and /etc/shadow
A superuser can edit the `/etc/passwd` and `/etc/shadow` files and manually remove the corresponding lines for the user (for example, `julian`). Please note that both files have to be edited for consistency and complete removal of the related user account.
Edit the `/etc/passwd` file using the `vipw` command-line utility, as follows:

```
sudo vipw
```

 Remove the following line (for the user `julian`):

```
julian:x:1001:1001:Julian,,,:/home/julian:/bin/bash
```

 Next, edit the `/etc/shadow` file using the `-s` or `--shadow` option with `vipw`, as follows:

```
sudo vipw -s
```

 Remove the following line (for the user `julian`):

```
julian:$6$xDdd7Eay/RKYjeTm$Sf.../:18519:0:99999:7:::
```

 After editing the preceding files, a superuser may also need to remove the deleted user’s home directory, as follows:

```
sudo rm -rf /home/julian
```

 For more information on the `userdel` utility, please check out the related documentation (`man userdel`) or the command-line help (`userdel --help`).
The user management concepts and commands learned so far apply exclusively to individual users in the system. When multiple users in the system have a common access level or permission attribute, they are collectively referred to as a group. Groups can be regarded as standalone organizational units we can create, modify, or delete. We can also define and alter user memberships associated with groups. The next section focuses on group management internals.
Managing groups
Linux uses groups to organize users. Simply put, a group is a collection of users sharing a common attribute. Examples of such groups could be *employees*, *developers*, *managers*, and so on. In Linux, a group is uniquely identified by a GID. Users within the same group share the same GID.
From a user’s perspective, there are two types of groups, outlined here:

*   **Primary group**: The user’s initial (default) login group
*   **Supplementary groups**: A list of groups the user is also a member of; also known as **secondary groups**

Every Linux user is a member of a primary group. A user can belong to multiple supplementary groups or no supplementary groups at all. In other words, there is one mandatory primary group associated with each Linux user, and a user can have multiple or no supplementary group memberships.
From a practical point of view, we can look at groups as a permissive context of collaboration for a select number of users. Imagine a *developers* group having access to developer-specific resources. Each user in this group has access to these resources. Users outside the *developers* group may not have access unless they authenticate with a group password if the group has one.
In the following section, we provide detailed examples of how to manage groups and set up group memberships for users. Most related commands require *superuser* or `sudo` privileges.
Creating, modifying, and deleting groups
While our primary focus remains on group administrative tasks, some related operations still involve user-related commands. Command-line utilities such as `groupadd`, `groupmod`, and `groupdel` are targeted strictly at creating, modifying, and deleting groups, respectively. On the other hand, the `useradd` and `usermod` commands carry group-specific options when associating users with groups. We’ll also introduce you to `gpasswd`, a command-line tool specializing in group administration, combining user- and group-related operations.
With this aspect in mind, let’s take a look at how to create, modify, and delete groups and how to manipulate group memberships for users.
Creating groups
To create a new group, a superuser invokes the `groupadd` command-line utility. Here’s the basic syntax of the related command:

```
groupadd [OPTIONS] GROUP
```

 Let’s create a new group (`developers`), with default settings, as follows:

```
sudo groupadd developers
```

 The group information is stored in the `/etc/group` file. Here’s the related data for the `developers` group:

```
cat /etc/group | grep developers
```

 The command yields the following output:
![Figure 4.10 – The group with default attributes](img/Figure_04_10_B19682.jpg)

Figure 4.10 – The group with default attributes
Let’s analyze the related group record. Each entry is delimited by a colon (`:`) and is listed here:

*   `developers`: Group name
*   `x`: Encrypted password (password hash is stored in `/etc/gshadow`)
*   `1003`: GID

We can also use the `getent` command to retrieve the preceding group information, as follows:

```
getent group developers
```

 A superuser may choose to create a group with a specific GID, using the `-g, --gid` option parameter with `groupadd`. For example, the following command creates the `developers` group (if it doesn’t exist) with a GID of `1200`:

```
sudo groupadd -g 1200 developers
```

 For more information on the `groupadd` command-line utility, please refer to the related documentation (`man groupadd`).
Group-related data is stored in the `/etc/group` and `/etc/gshadow` files. The `/etc/group` file contains generic group membership information, while the `/etc/gshadow` file stores the encrypted password hashes for each group. Let’s take a brief look at group passwords.
Understanding group passwords
By default, a group doesn’t have a password when created with the simplest invocation of the `groupadd` command (for example, `groupadd developers`). Although `groupadd` supports an encrypted password (via the `-p, --password` option parameter), this would require an extra step to generate a secure password hash. There’s a better and simpler way to create a group password: by using the `gpasswd` command-line utility.
Important note
`gpasswd` is a command-line tool that helps with everyday group administration tasks.
The following command creates a password for the `developers` group:

```
sudo gpasswd developers
```

 We get prompted to enter and re-enter a password, as illustrated here:
![Figure 4.11 – Creating a password for the developers group](img/Figure_04_11_B19682.jpg)

Figure 4.11 – Creating a password for the developers group
The purpose of a group password is to protect access to group resources. A group password is inherently insecure when shared among group members, yet a Linux administrator may choose to keep the group password private while group members collaborate unhindered within the group’s security context.
Here’s a quick explanation of how it works. When a member of a specific group (for example, `developers`) logs in to that group (using the `newgrp` command), the user is not prompted for the group password. When users who don’t belong to the group attempt to log in, they will be prompted for the group password.
In general, a group can have administrators, members, and a password. Members of a group who are the group’s administrators may use `gpasswd` without being prompted for a password, as long as they’re logged in to the group. Also, group administrators don’t need superuser privileges to perform group administrative tasks for a group they are the administrator of.
We’ll take a closer look at `gpasswd` in the next sections, where we further focus on group management tasks, as well as adding users to a group and removing users from a group. But for now, let’s keep our attention strictly at the group level and see how we can modify a user group.
Modifying groups
The most common way to modify the definition of a group is via the `groupmod` command-line utility. Here’s the basic syntax for the command:

```
groupmod [OPTIONS] GROUP
```

 The most common operations when changing a group’s definition are related to the GID, group name, and group password. Let’s take a look at each of these changes. We assume our previously created group is named `developers`, with a GID of `1003`.
To change the GID to `1200`, a superuser invokes the `groupmod` command with the `-g, --gid` option parameter, as follows:

```
sudo groupmod -g developers to devops, we invoke the -n, --new-name option, like this:

```
sudo groupmod -n devops group with the following command:

```
getent group devops
```

 The command yields the following output:
![Figure 4.12 – Verifying the group changes](img/Figure_04_12_B19682.jpg)

Figure 4.12 – Verifying the group changes
To change the group password for `devops`, the simplest way is to use `gpasswd`, as follows:

```
sudo gpasswd devops
```

 We are prompted to enter and re-enter a password.
To remove the group password for `devops`, we invoke the `gpasswd` command with the `-r, --remove-password` option, as follows:

```
sudo gpasswd -r devops
```

 As the command has no visible outcome or message, we will be prompted back to the shell:
![Figure 4.13 – Setting a new group password and removing a group password](img/Figure_04_13_B19682.jpg)

Figure 4.13 – Setting a new group password and removing a group password
For more information on `groupmod` and `gpasswd`, refer to the system manuals of these utilities (`man groupmod` and `man gpasswd`), or simply invoke the `-h, --help` option for each.
Next, we look at how to delete groups.
Deleting groups
To delete groups, we use the `groupdel` command-line utility. The related syntax is shown here:

```
groupdel [OPTIONS] GROUP
```

 By default, Linux enforces referential integrity between a primary group and the users associated with that primary group. We cannot delete a group that has been assigned as a primary group for some users before deleting the users of that primary group. In other words, by default, Linux doesn’t want to leave the users with dangling primary GIDs.
For example, when we first added the user `julian`, they were assigned automatically to the `julian` primary group. We then added the user to the `sudoers` group.
Let’s attempt to add the user `julian` to the `devops` group. A superuser may run the `usermod` command with the `-g, --gid` option parameter to *change* the primary group of a user. The command should be invoked for each user. Here’s an example of removing the user `julian` from the `julian` primary group. First, let’s get the current data for the user, as follows:

```
id julian
```

 This is the output:
![Figure 4.14 – Retrieving the current primary group for the user](img/Figure_04_14_B19682.jpg)

Figure 4.14 – Retrieving the current primary group for the user
Now, let us add the user `julian` to the `devops` group. The `-g, --gid` option parameter of the `usermod` command accepts both a *GID* and a group *name*. The specified group name must already be present in the system; otherwise, the command will fail. If we want to change the primary group (for example, to `devops`), we simply specify the group name in the `-g, --gid` option parameter, as follows:

```
sudo usermod -g devops julian
```

 The output is shown in the following screenshot:
![Figure 4.15 – Changing the primary group of the user](img/Figure_04_15_B19682.jpg)

Figure 4.15 – Changing the primary group of the user
The result is that the user `julian` is now part of the `devops` group.
Now, let us attempt to delete the `devops` group, which is the primary group for the user `julian`. Attempting to delete the `devops` group results in an error, as can be seen in *Figure 4**.16* (the first command used). Therefore, we cannot delete a group that is not empty.
A superuser may choose to *force* the deletion of a primary group, invoking `groupdel` with the `-f, --force` option, but this would be ill advised. This is because the command would result in users with orphaned primary GIDs and a possible security hole in the system. The maintenance and removal of such users would also become problematic.
In order to be able to delete the `devops` group, we need to assign another group to the user `julian`. What we can do is assign it to the initial primary group called `julian`, and then attempt to delete the `devops` group, now that it is empty. First, let us assign the user `julian` to the `julian` group with the following command:

```
sudo usermod -g julian julian
```

 At this point, it’s safe to delete the group (`devops`), as follows:

```
sudo groupdel devops
```

 The outcome from the preceding commands is this:
![Figure 4.16 – Successful attempt to delete the group](img/Figure_04_16_B19682.jpg)

Figure 4.16 – Successful attempt to delete the group
For more information on the `groupdel` command-line utility, check out the related system manual (`man groupdel`), or simply invoke `groupdel --help`.
Modifying groups via /etc/group
An administrator can also manually edit the `/etc/group` file to modify group data by updating the related line. Although the editing can be done with a text editor of your choice (for example, `nano`), we recommend the use of the `vigr` command-line utility for a safer approach. `vigr` is similar to `vipr` (for modifying `/etc/passwd`) and sets safety locks to prevent possible data corruption during concurrent changes of group data.
The following command opens the `/etc/group` file for editing by also prompting for the preferred text editor (for example, `nano` or `vim`):

```
sudo vigr
```

 For example, we can change the settings for the `developers` group by editing the following line:

```
developers:x:1200:julian,alex
```

 When deleting groups using the `vigr` command, we’re also prompted to remove the corresponding entry in the group shadow file `(/etc/gshadow`). The related command invokes the `-s` or `--shadow` option, as illustrated here:

```
sudo vigr -s
```

 For more information on the `vigr` utility, please refer to the related system manual (`man vigr`).
As with most Linux tasks, all the preceding tasks could have been accomplished in different ways. The commands chosen are the most common ones, but there might be cases when a different approach may prove more appropriate.
In the next section, we’ll take a glance at how to add users to primary and secondary groups and how to remove users from these groups.
Managing users in groups
So far, we’ve only created groups that have no users associated. There is not much use for empty user groups, so let’s add some users to them.
Adding users to a group
Before we start adding users to a group, let’s create a few groups. In the following example, we create the groups by also specifying their GID (via the `-g, --gid` option parameter of the `groupadd` command):

```
sudo groupadd -g 1100 admin
sudo groupadd -g 1200 developers
sudo groupadd -g 1300 devops
```

 We can check the last groups created by using the following command:

```
cat /etc/group | tail -n 5
```

 It will show us the last five lines of the `/etc/group` file. We can see the last five groups created.
Next, we create a couple of new users (`alex2` and `julian2` as we already have users `alex` and `julian`) and add them to some of the groups we just created. We’ll have the `admin` group set as the *primary group* for both users, while the `developers` and `devops` groups are defined as *secondary* (or *supplementary*) *groups*. The code can be seen here:

```
sudo useradd -g admin -G developers,devops alex2
sudo useradd -g admin -G developers,devops julian2
```

 The `-g, --gid` option parameter of the `useradd` command specifies the (unique) primary group (`admin`). The `-G, --groups` option parameter provides a comma-separated list (without intervening spaces) of the secondary group names (`developers``,``devops`).
We can verify the group memberships for both users with the following commands:

```
id alex2
id julian2
```

 The output is shown in the following screenshot:
![Figure 4.17 – Assigning groups to new users](img/Figure_04_17_B19682.jpg)

Figure 4.17 – Assigning groups to new users
As we can see, the `gid` attribute shows the primary group membership: `gid=1100(admin)`. The `groups` attribute shows the supplementary (secondary) groups: `groups=1100(admin),1200(developers),1300(devops)`.
With users scattered across multiple groups, an administrator is sometimes confronted with the task of moving users between groups. The following section shows how to do this.
Moving and removing users across groups
Building upon the previous example, let’s assume the administrator wants to move (or add) the user `alex2` to a new secondary group called `managers`. Please note that, according to our previous examples, the user `alex2` has `admin` as the primary group and `developers`/`devops` as secondary groups (see the output of the `id alex2` command in *Figure 4**.18*).
Let’s create a `managers` group first, with GID `1400`. The code can be seen here:

```
sudo groupadd -g 1400 managers
```

 Next, add our existing user, `alex2`, to the `managers` group. We use the `usermod` command with the `-G, --groups` option parameter to specify the secondary groups the user is associated with.
The simplest way to *append* a secondary group to a user is by invocation of the `-a, --append` option of the `usermod` command, as illustrated here:

```
sudo usermod -a -G managers alex2
```

 The preceding command would preserve the existing secondary groups for the user `alex2` while adding the new `managers` group. Alternatively, we could run the following command:

```
sudo usermod -G developers,devops,managers alex2
```

 In the preceding command, we specified multiple groups (with no intervening whitespace!).
Important note
We preserved the existing secondary groups (`developers`/`devops`) and *appended* to the comma-separated list the `managers` additional secondary group. If we only had the `managers` group specified, the user `alex2` would have been *removed* from the `developers` and `devops` secondary groups.
To verify whether the user `alex2` is now part of the `managers` group, run the following command:

```
id alex2
```

 This is the output of the command:

```
uid=1004(alex2) gid=1100(admin) groups attribute (highlighted) includes the related entry for the managers group: 1400(managers).
Similarly, if we wanted to *remove* the user `alex2` from the `developers` and `devops` secondary groups, to only be associated with the `managers` secondary group, we would run the following command:

```
sudo usermod -G managers alex2
```

 This is the output:
![Figure 4.18 – Verifying the secondary groups for the user](img/Figure_04_18_B19682.jpg)

Figure 4.18 – Verifying the secondary groups for the user
The `groups` tag now shows the primary group `admin` (by default) and the `managers` secondary group.
The command to remove the user `alex2` from all secondary groups is shown here:

```
sudo usermod -G '' alex2
```

 The `usermod` command has an empty string (`''`) as the `-G, --groups` option parameter, to ensure no secondary groups are associated with the user. We can verify that the user `alex2` has no more secondary group memberships with the following command:

```
id alex
```

 This is the output:
![Figure 4.19 – Verifying the user has no secondary groups](img/Figure_04_19_B19682.jpg)

Figure 4.19 – Verifying the user has no secondary groups
As we can see, the `groups` tag only contains the `1100(admin)` primary GID, which by default is always shown for a user.
If an administrator chooses to remove the user `alex2` from a primary group or assign them to a different primary group, they must run the `usermod` command with the `-g, --gid` option parameter and specify the primary group name. A primary group is always mandatory for a user, and it must exist.
For example, to move the user `alex2` to the `managers` primary group, the administrator would run the following command:

```
sudo usermod -g managers alex2
```

 The related user data can be obtained using the following command:

```
id alex2
```

 The command yields the following output:
![Figure 4.20 – Verifying the user has been assigned to the new primary group](img/Figure_04_20_B19682.jpg)

Figure 4.20 – Verifying the user has been assigned to the new primary group
The `gid` attribute of the user record in *Figure 4**.21* reflects the new primary group: `gid=1400(managers)`.
If the administrator chooses to configure the user `alex2` without a specific primary group, they must first create an exclusive *group* (named `alex2`, for convenience), and have the GID matching the UID of the user `alex2` (`1004`), as follows:

```
sudo groupadd -g 1004 alex2
```

 And now, we can remove the user `alex2` from the current primary group (`managers`) by specifying the exclusive primary group we just created (`alex2`), like this:

```
sudo usermod -g alex2 alex2
```

 The related user record becomes this:

```
id alex2
```

 This is the output:
![Figure 4.21 – Verifying the user has been removed from primary groups](img/Figure_04_21_B19682.jpg)

Figure 4.21 – Verifying the user has been removed from primary groups
The `gid` attribute of the user record reflects the exclusive primary group (matching the user): `gid=1004(alex2)`. Our user doesn’t belong to any other primary groups anymore.
Adding, moving, and removing users across groups may become increasingly daunting tasks for a Linux administrator. Knowing at any time which users belong to which groups is valuable information, both for reporting purposes and user automation workflows. The following section provides a few commands for viewing user and group data.
Viewing users and groups
In this section, we will provide some potentially useful commands for retrieving group and group membership information. Before we get into any commands, we should keep in mind that group information is stored in the `/etc/group` and `/etc/gshadow` files. Among the two, the former has the information we’re most interested in.
We can parse the `/etc/group` file to retrieve all groups, as follows:

```
cat /etc/group | cut -d: -f1 | column | less
```

 The command yields the following output:
![Figure 4.22 – Retrieving all group names](img/Figure_04_22_B19682.jpg)

Figure 4.22 – Retrieving all group names
A similar command would use `getent`, which we can use like this:

```
getent group | cut -d: -f1 | column | less
```

 The output of the preceding command is identical to the output shown in *Figure 4**.22*. We can retrieve the information of an individual group (for example, `developers`) with the following command:

```
getent group developers
```

 This is the output:
![Figure 4.23 – Retrieving information for a single group](img/Figure_04_23_B19682.jpg)

Figure 4.23 – Retrieving information for a single group
The output of the preceding command also reveals the members of the `developers` group (`julian2`).
To list all groups a specific user is a member of, we can use the `groups` command. For example, the following command lists all groups the user `alex` is a member of:

```
groups alex
```

 This is the command output:
![Figure 4.24 – Retrieving group membership information of a user](img/Figure_04_24_B19682.jpg)

Figure 4.24 – Retrieving group membership information of a user
The output of the previous command shows the groups for the user `alex`, starting with the primary group (`alex`).
A user can retrieve their own group membership using the `groups` command-line utility without specifying a group name. The following command is executed in a terminal session of the user `packt`, who is also an administrator (superuser):

```
groups
```

 The command yields this output:
![Figure 4.25 – The current user’s groups](img/Figure_04_25_B19682.jpg)

Figure 4.25 – The current user’s groups
There are many other ways and commands to retrieve user- and group-related information. We hope that the preceding examples provide a basic idea about where and how to look for some of this information.
Next, let’s look at how a user can switch or log in to specific groups.
Group login sessions
When a user logs in to the system, the group membership context is automatically set to the user’s primary group. Once the user is logged in, any user-initiated task (such as creating a file or running a program) is associated with the user’s primary group membership permissions. A user may also choose to access resources in other groups where they are also a member (that is, supplementary or secondary groups). To switch the group context or log in with a new group membership, a user invokes the `newgrp` command-line utility.
The basic syntax for the `newgrp` command is this:

```
newgrp GROUP
```

 In the following example, we assume a user (`julian`) is a member of multiple groups—`admin` as the primary group, and `developers`/`devops` as secondary groups:

```
id julian
```

 This is the output:
![Figure 4.26 – A user with multiple group memberships](img/Figure_04_26_B19682.jpg)

Figure 4.26 – A user with multiple group memberships
Let’s impersonate the user `julian` for a while. We are currently logged in as the user `packt`. To change to the user `julian`, we will use the following command:

```
su julian
```

 Remember that the user `julian` needs to have their password set in order to authenticate.
When logged in as `julian`, the default login session has the following user and group context:

```
whoami
```

 In our case, this is the output:
![Figure 4.27 – Getting the current user](img/Figure_04_27_B19682.jpg)

Figure 4.27 – Getting the current user
The `whoami` command provides the current UID (see more details on the command with `man whoami` or `whoami --help`), as follows:

```
groups
```

 This is the output:
![Figure 4.28 – Getting the current user’s groups](img/Figure_04_28_B19682.jpg)

Figure 4.28 – Getting the current user’s groups
The `groups` command displays all groups that the current user is a member of (see more details on the command with `man groups` or `groups --help`).
The user can also view their IDs (user and GIDs) by invoking the `id` command, as follows:

```
id
```

 This is the output:
![Figure 4.29 – Viewing the current user and GID information](img/Figure_04_29_B19682.jpg)

Figure 4.29 – Viewing the current user and GID information
There are various invocations of the `id` command that provide information on the current user and group session. The following command (with the `-g, --group` option) retrieves the ID of the current group session for the user:

```
id -g
1100
```

 In our case, the preceding command shows `1100`—the GID corresponding to the user’s primary group, which is `admin` (see the `gid` attribute in *Figure 4**.30*). Upon login, the default group session is always the primary group corresponding to the user. If the user were to create a file, for example, the file permission attributes would reflect the primary group’s ID. We’ll look at the file permissions in more detail in the *Managing* *permissions* section.
Now, let’s switch the group session for the current user to `developers`, as follows:

```
newgrp developers
```

 The current group session yields this:

```
id -g
1200
```

 The GID corresponds to the `developers` secondary GID, as displayed by the `groups` attribute in *Figure 4**.30*: `1200(developers)`. If the user created any files now, the related file permission attributes would have the `developers` GID:
![Figure 4.30 – Switching the group session](img/Figure_04_30_B19682.jpg)

Figure 4.30 – Switching the group session
If the user attempts to log in to a group they are not a member of (for example, `managers`), the `newgrp` command prompts for the `managers` group’s password:

```
newgrp managers
```

 If our user had the `managers` group password, or if they were a superuser, the group login attempt would succeed. Otherwise, the user would be denied access to the `managers` group’s resources.
We conclude here our topic of managing users and groups. The examples of the related administrative tasks used throughout this section are certainly all-encompassing. In many of these cases, there are multiple ways to achieve the same result, using different commands or approaches.
By now, you should be relatively proficient in managing users and groups, and comfortable using the various command-line utilities for operating the related changes. Users and groups are managed in a relational fashion, where users belong to a group or groups are associated with users. We also learned that creating and managing users and groups requires superuser privileges. In Linux, user data is stored in the `/etc/passwd` and `/etc/shadow` files, while group information is found in `/etc/group` and `/etc/gshadow`. Besides using the dedicated command-line utilities, users and groups can also be altered by manually editing these files.
Next, we’ll turn to the security and isolation context of the multiuser group environment. In Linux, the related functionality is accomplished by a system-level access layer that controls the read, write, and execute permissions of files and directories, by specific users and groups.
The following section explores the management and administrative tasks related to these permissions.
Managing permissions
A key tenet of Linux is the ability to allow multiple users to access the system while performing independent tasks simultaneously. The smooth operation of this multiuser, multitasking environment is controlled via **permissions**. The Linux kernel provides a robust framework for the underlying security and isolation model. At the user level, dedicated tools and command-line utilities help Linux users and system administrators with related permission management tasks.
For some Linux users, especially beginners, Linux permissions may appear confusing at times. This section attempts to demystify some of the key concepts about file and directory permissions in Linux. You will learn about the basic permission *rights* of accessing files and directories—the *read*, *write*, and *execution* permissions. We explore some of the essential administrative tasks for viewing and changing permissions, using system-level command-line utilities.
Most of the topics discussed in this section should be regarded closely with users and groups. The related idioms can be as simple as *a user can read or update a file*, *a group has access to these files and directories*, or *a user can execute* *this program*.
Let’s start with the basics, introducing file and directory permissions.
File and directory permissions
In Linux, permissions can be regarded as the *rights* or *privileges* to act upon a file or a directory. The basic rights, or *permission attributes*, are outlined here:

*   **Read**: A *read* permission of a file allows users to view the content of the file. On a directory, the read permission allows users to list the content of the directory.
*   **Write**: A *write* permission of a file allows users to modify the content of the file. For a directory, the write permission allows users to modify the content of the directory by adding, deleting, or renaming files.
*   `cd` command).

First, let’s take a look at how to reveal the permissions for files and directories.
Viewing permissions
The most common way to view the permissions of a file or directory is by using the `ls` command-line utility. The basic syntax of this command is this:

```
ls [OPTIONS] FILE|DIRECTORY
```

 Here is an example use of the `ls` command to view the permissions of the `/``etc/passwd` file:

```
ls -l /etc/passwd
```

 The command yields the following output:

```
-rw-r--r-- 1 root root 2010 Mar  9 08:57 /etc/passwd
```

 The `-l` option of the `ls` command provides a detailed output by using the *long listing format*, according to the `ls` documentation (`man ls`).
Let’s analyze the output, as follows:

```
-rw-r--r-- 1 root root 2010 Mar  9 08:57 /etc/passwd
```

 We have nine segments, separated by single whitespace characters (delimiters). These are outlined here:

*   `-rw-r--r--`: The file access permissions
*   `1`: The number of hard links
*   `root`: The *user* who is the owner of the file
*   `root`: The *group* that is the owner of the file
*   `2010`: The size of the file
*   `Mar`: The month the file was created
*   `9`: The day of the month the file was created
*   `08:57`: The time of day the file was created
*   `/etc/passwd`: The filename

Let’s examine the file access permissions field (`-rw-r--r--`). File access permissions are defined as a 10-character field, grouped as follows:

*   The first character (attribute) is reserved for the file type (see the *File* *types* section).
*   The next 9 characters represent a 9-bit field, defining the effective permissions as 3 sequences of 3 attributes (bits) each: *user owner* permissions, *group owner* permissions, and *all other users’* permissions (see the *Permission* *attributes* section).

Let’s take a look at the file type attributes.
File type attributes
The file type attributes are listed here:

*   `d`: Directory
*   `-`: Regular file
*   `l`: Symbolic link
*   `p`: Named pipe—a special file that facilitates communication between programs
*   `s`: Socket—similar to a pipe but with bidirectional network communications
*   `b`: Block device—a file that corresponds to a hardware device
*   `c`: Character device—similar to a block device

Let’s have a closer look at the permission attributes.
Permission attributes
As previously noted, the access permissions are represented by a 9-bit field, a group of 3 sequences, each with 3 bits, defined as follows:

*   **Bits 1-3**: *User* owner permissions
*   **Bits 4-6**: *Group* owner permissions
*   **Bits 7-9**: *All* other users’ (or *world*) permissions

Each permission attribute is a bit flag in the binary representation of the related 3-bit sequence. They can be represented either as a character or as an equivalent numerical value, also known as the *octal* value, depending on the range of the bit they represent.
Here are the permission attributes with their respective octal values:

*   `r`: *Read* permission; 2 ^ 2 = `4` (bit 2 set)
*   `w`: *Write* permission: 2 ^ 1 = `2` (bit 1 set)
*   `x`: *Execute* permission: 2 ^ 0 = `1` (bit 0 set)
*   `-`: *No* permission: `0` (no bits set)

The resulting corroborated number is also known as the *octal value* of the file permissions (see the *File permission examples* section). Here’s an illustration of the file permission attributes:
![Figure 4.31 – The file permission attributes](img/Figure_04_31_B19682.jpg)

Figure 4.31 – The file permission attributes
Next, let’s consider some examples.
File permission examples
Now, let’s go back and evaluate the file access permissions for `/etc/passwd`: `-rw-r--r--`, as follows:

*   `-`: The first character (byte) denotes the file type (a regular file, in our case)
*   `rw-`: The next three-character sequence indicates the user owner permissions; (in our case, read (`r`); write (`w`); octal value = `4` (`r`) + `2` (`w`) = `6` (`rw`))
*   `r--`: The next 3-byte sequence defines the group owner permissions (in our case, read (`r`); octal value = `4` (`r`))
*   `r--`: The last three characters denote the permissions for all other users in the system (in our case, read (`r`); octal value = `4` (`r`))

According to the preceding information, the resulting octal value of the `/etc/passwd` file access permissions is `644`. Alternatively, we can query the octal value with the `stat` command, as follows:

```
stat --format '%a' /etc/passwd
```

 The command yields the following output:

```
644
```

 The `stat` command displays the file or filesystem status. The `--format` option parameter specifies the access rights in octal format (`'%a'`) for the output.
Here are a few examples of access permissions, with their corresponding octal values and descriptions. The three-character sequences are intentionally delimited with whitespace for clarity. The leading file type has been omitted:

*   `rwx` (`777`): Read, write, and execute for all users including owner, group, and world
*   `rwx r-x` (`755`): Read and execute for all users; the file owner has write permissions
*   `rwx r-x ---` (`750`): Read and execute for owner and group; the owner has write permissions while others have no access
*   `rwx --- ---` (`700`): Read, write, and execute for owner; everyone else has no permissions
*   `rw- rw- rw-` (`666`): Read and write for all users; there are no execute permissions
*   `rw- rw- r--` (`664`): Read and write for owner and group; read for others
*   `rw- rw- ---` (`660`): Read and write for owner and group; others have no permissions
*   `rw- r-- r--` (`644`): Read and write for owner; read for group and others
*   `rw- r-- ---` (`640`): Read and write for owner; read for group; no permissions for others
*   `rw- --- ---` (`600`): Read and write for owner; no permissions for group and others
*   `r-- --- ---` (`400`): Read for owner; no permissions for others

Read, write, and execute are the most common types of file access permissions. So far, we have mostly focused on permission types and their representation. In the next section, we will explore a few command-line tools used for altering permissions.
Changing permissions
Modifying file and directory access permissions is a common Linux administrative task. In this section, we will learn about a few command-line utilities that are handy when it comes to changing permissions and ownership of files and directories. These tools are installed with any modern-day Linux distribution, and their use is similar across most Linux platforms.
Using chmod
The `chmod` command is short for *change mode*, and it’s used to set access permissions on files and directories. The `chmod` command can be used by both the current user (owner) and a superuser.
Changing permissions can be done in two different modes: **relative** and **absolute**. Let’s take a look at each of them.
Using chmod in relative mode
Changing permissions in **relative** mode is probably the easiest of the two. It is important to remember the following:

*   *To whom* we change permissions: `u` = user (owner), `g` = group, `o` = others
*   *How* we change permissions: `+` = add, `-` = remove, `=` = exactly as is
*   *Which* permission we change: `r` = read, `w` = write, `x` = execute

Let’s explore a few examples of using `chmod` in relative mode.
In our first example, we want to add write (`w`) permissions for all *other* (`o`) users (*world*), to `myfile`, as follows:

```
chmod o+w myfile
```

 The related command-line output is shown here:
![Figure 4.32 – Setting write permissions to all other users](img/Figure_04_32_B19682.jpg)

Figure 4.32 – Setting write permissions to all other users
In the next example, we remove the read (`r`) and write (`w`) permissions for the current user owner (`u`) of `myfile`, as follows:

```
chmod u-rw myfile
```

 The command-line output is shown here:
![Figure 4.33 – Removing read-write permissions for owner](img/Figure_04_33_B19682.jpg)

Figure 4.33 – Removing read-write permissions for owner
We did not use `sudo` in either of the preceding examples since we carried out the operations as the current owner of the file (`packt`).
In the following example, we assume that `myfile` has read, write, and execute permissions for everyone. Then, we carry out the following changes:

*   Remove the read (`r`) permission for the owner (`u`)
*   Remove the write (`w`) permission for the owner (`u`) and group (`g`)
*   Remove the read (`r`), write (`w`), and execute (`x`) permissions for everyone else (`o`)

This is illustrated in the following code snippet:

```
chmod u-r,ug-w,o-rwx myfile
```

 The command-line output is shown here:
![Figure 4.34 – A relatively complex invocation of chmod in relative mode](img/Figure_04_34_B19682.jpg)

Figure 4.34 – A relatively complex invocation of chmod in relative mode
Next, let’s look at a second way of changing permissions: using the `chmod` command-line utility in absolute mode, by specifying the octal number corresponding to the access permissions.
Using chmod in absolute mode
The `chmod` changes all permission attributes at once, using an *octal* number. The *absolute* designation of this method is due to changing permissions without any reference to existing ones, by simply assigning the octal value corresponding to the access permissions.
Here’s a quick list of the octal values corresponding to effective permissions:

*   `7` `rwx`: Read, write, and execute
*   `6` `rw-`: Read and write
*   `5` `r-w`: Read and execute
*   `4` `r--`: Read
*   `3` `-wx`: Write and execute
*   `2` `-``w-`: Write
*   `1` `--``x`: Execute
*   `0` `---`: No permissions

In the following example, we change the permissions of `myfile` to read (`r`), write (`w`), and execute (`x`) for everybody:

```
chmod 777 myfile
```

 The related change is illustrated by the following command-line output:
![Figure 4.35 – The chmod invocation in absolute mode](img/Figure_04_35_B19682.jpg)

Figure 4.35 – The chmod invocation in absolute mode
For more information about the `chmod` command, please refer to the related documentation (`man chmod`).
Let’s now look at our next command-line utility, specializing in file and directory ownership changes.
Using chown
The `chown` command (short for *change owner*) is used to set the ownership of files and directories. Typically, the `chmod` command can only be run with *superuser* privileges (that is, by a *sudoer*). Regular users can only change the *group* ownership of their files, and only when they are a member of the target group.
The syntax of the `chown` command is shown here:

```
chown [OPTIONS] [OWNER][:[GROUP]] FILE
```

 Usually, we invoke the `chown` command with both user *and* group ownerships—for example, like this:

```
sudo chown julian:developers myfile
```

 The related command-line output is shown here:
![Figure 4.36 – A simple invocation of the chown command](img/Figure_04_36_B19682.jpg)

Figure 4.36 – A simple invocation of the chown command
One of the most common uses of `chown` is for *recursive mode* invocation, with the `-R, --recursive` option. The following example changes the ownership permissions of all files in `mydir` (directory), initially owned by `root`, to `julian`:

```
sudo chown -R julian:julian mydir/
```

 The related changes are shown in the following command-line output:
![Figure 4.37 – Invoking ls and chown in recursive mode](img/Figure_04_37_B19682.jpg)

Figure 4.37 – Invoking ls and chown in recursive mode
For more information about the `chown` command, please refer to the related documentation (`man chown`).
Next, let’s briefly look at a similar command-line utility that specializes exclusively in group ownership changes.
Using chgrp
The `chgrp` command (short for *change group*) is used to change the *group* ownership for files and directories. In Linux, files and directories typically belong to a user (owner) or a group. We can set user ownership by using the `chown` command-line utility, while group ownership can be set with `chgrp`.
The syntax for `chgrp` is shown here:

```
chgrp [OPTIONS] GROUP FILE
```

 The following example changes the group ownership of `myfile` to the `developers` group:

```
sudo chgrp developers myfile
```

 The changes are shown in the following output:
![Figure 4.38 - Using chgrp to change group ownership](img/Figure_04_38_B19682.jpg)

Figure 4.38 - Using chgrp to change group ownership
The preceding command has been invoked with superuser privileges (`sudo`) since the current user (`packt`) is not an admin for the `developers` group.
For more information about the `chgrp` utility, please refer to the tool’s command-line help (`chgrp --help`).
Using umask
The `umask` command is used to view or set the default *file mode mask* in the system. The file mode represents the default permissions for any new files and directories created by a user. For example, the default file mode masks in Ubuntu are given here:

*   `0002` for a regular user
*   `0022` for the `root` user

As a general rule in Linux, the *default permissions* for new files and directories are calculated with the following formulas:

*   `0666 – umask`: For a new file created by a regular user
*   `0777 – umask`: For a new directory created by a regular user

According to the preceding formula, on Ubuntu, we have the following default permissions:

*   File (regular user): `0666 – 0002 =` `0664`
*   File (`root`): `0666 – 0022 =` `0644`
*   Directory (regular user): `0777 – 0002 =` `0775`
*   Directory (`root`): `0777 – 0022 =` `0755`

In the following examples, run on Ubuntu, we create a file (`myfile`) and a directory (`mydir`), using the terminal session of a regular user (`packt`). Then, we query the `stat` command for each and verify that the default permissions match the values enumerated previously for regular users (file: `664`, directory: `775`).
Let’s start with the default file permissions first, as follows:

```
touch myfile2
stat --format '%a' myfile2
```

 The related output is shown here:

```
664
```

 Next, let’s verify the default directory permissions, as follows:

```
mkdir mydir2
stat --format '%a' mydir2
```

 The related output is shown here:

```
775
```

 Here’s a list with the typical `umask` values for files and directories on Linux systems:
![Figure 4.39 – Typical umask values on Linux](img/Figure_04_39_B19682.jpg)

Figure 4.39 – Typical umask values on Linux
For more information about the `umask` utility, please refer to the tool’s command-line help (`umask --help`).
File and directory permissions are critical for a secure environment. Users and processes should operate exclusively within the isolation and security constraints controlled by permissions, to avoid inadvertent or deliberate interference with the use and ownership of system resources. There are cases, particularly in user impersonation situations, when the access rights may involve some special permission attributes. Let’s have a look at them.
Special permissions
In Linux, the ownership of files and directories is usually determined by the UID and GID of the user—or group—who created them. The same principle applies to applications and processes—they are owned by the users who launch them. The special permissions are meant to change this default behavior when needed.
Here are the special permission flags, with their respective octal values:

*   `setuid`: 2 ^ 2 = `4` (bit 2 set)
*   `setgid`: 2 ^ 1 = `2` (bit 1 set)
*   `sticky`: 2 ^ 0 = `1` (bit 0 set)

When any of these special bits are set, the overall octal number of the access permissions will have an extra digit, with the leading (high-order) digit corresponding to the special permission’s octal value.
Let’s look at these special permission flags, with examples for each.
The setuid permission
With the `setuid` bit set, when an executable file is launched, it will run with the privileges of the file owner instead of the user who launched it. For example, if the executable is owned by `root` and launched by a *regular* user, it will run with `root` privileges. The `setuid` permission could pose a potential security risk when used inadequately, or when vulnerabilities of the underlying process could be exploited.
In the file access permission field, the `setuid` bit could have either of the following representations:

*   `s` *replacing* the corresponding executable bit (`x`) (when the executable bit is present)
*   `S` (the capital letter) for a non-executable file

The `setuid` permission can be set via the following `chmod` command (for example, for the `myscript.sh` executable file):

```
chmod u+s myscript.sh
```

 The resulting file permissions are shown here (including the octal value): `-``rwsrwxr-x` (`4775`).
Here is the related command-line output:
![Figure 4.40 – The setuid permission](img/Figure_04_40_B19682.jpg)

Figure 4.40 – The setuid permission
In the preceding screenshot, you can see the difference in permissions. Before applying the `chmod` command, the permissions are `-rwxrwxr-x`, and after applying the `setuid` permission with the `chmod` command, an `s` (referring to `setuid`) is included in the user’s permission, `-rwsrwxr-x`. For more information on `setuid`, please visit [https://docs.oracle.com/cd/E19683-01/816-4883/secfile-69/index.html](https://docs.oracle.com/cd/E19683-01/816-4883/secfile-69/index.html) or refer to the `chmod` command-line utility documentation (`man chmod`).
The setgid permission
While `setuid` controls user impersonation privileges, `setgid` has a similar effect on group impersonation permissions.
When an executable file has the `setgid` bit set, it runs using the permissions of the group that owns the file, rather than the group of the user who initiated it. In other words, the GID of the process is the same as the GID of the file.
When used on a directory, the `setgid` bit changes the default ownership behavior so that files created within the directory will have group ownership of the parent directory instead of the group associated with the user who created them. This behavior could be adequate in file-sharing situations when files can be changed by all users associated with the parent directory’s owner group.
The `setgid` permission can be set via the following `chmod` command (for example, for the `myscript.sh` executable file, the original file, before applying `setuid` to it):

```
chmod g+s myscript.shw
```

 The resulting file permissions are shown here (including the octal value): `-``rwxrwsr-x` (`2775`).
The command-line output is shown here:
![Figure 4.41 – The setgid permission](img/Figure_04_41_B19682.jpg)

Figure 4.41 – The setgid permission
For more information on `setgid`, please visit [https://en.wikipedia.org/wiki/Setuid](https://en.wikipedia.org/wiki/Setuid) or refer to the `chmod` command-line utility documentation (`man chmod`).
The sticky permission
The `sticky` bit has no effect on files. For a directory with the `sticky` permission, only the user owner or group owner of the directory can delete or rename files within the directory. Users or groups with write access to the directory, by way of user or group ownership, cannot delete or modify files in the directory. The `sticky` permission is useful when a directory is owned by a privileged group whose members share write access to files in that directory.
The `sticky` permission can be set via the following `chmod` command (for example, for the `mydir` directory):

```
chmod +t mydir
```

 The resulting directory permissions are shown here (including the octal value): `drwxrwxr-t` (`1775`).
The command-line output is shown here:
![Figure 4.42 – The sticky permission](img/Figure_04_42_B19682.jpg)

Figure 4.42 – The sticky permission
For more information on `sticky`, please visit [https://en.wikipedia.org/wiki/Setuid](https://en.wikipedia.org/wiki/Setuid) or refer to the `chmod` command-line utility documentation (`man chmod`).
Interpreting permissions can be a daunting task. This section aimed to demystify some of the related intricacies, and we hope that you will feel more comfortable handling file and directory permissions in everyday Linux administration tasks.
Summary
In this chapter, we explored some of the essential concepts related to managing users and groups in Linux. We learned about file and directory permissions and the different access levels of a multiuser environment. For each main topic, we focused on basic administrative tasks, providing various practical examples and using typical command-line tools for everyday user access and permission management operations.
Managing users and groups, and the related filesystem permissions that come into play, is an indispensable skill of a Linux administrator. The knowledge gained in this chapter will, we hope, put you on track to becoming a proficient superuser.
In the following chapter, we continue our journey of mastering Linux internals by exploring processes, daemons, and **inter-process communication** (**IPC**) mechanisms. An important aspect to keep in mind is that processes and daemons are also *owned* by users or groups. The skills learned in this chapter will help us navigate the related territory when we look at *who runs what* at any given time in the system.
Questions
Here are a few thoughts and questions that sum up the main ideas covered in this chapter:

1.  What is a superuser?

`sudo`

1.  Think of a command-line utility for creating users. Can you think of another one?

`adduser` and `useradd`

1.  What is the octal value of the `-rw-rw-r—` access permission?

`r`, `w`, and `x` are: `4`, `2`, and `1`

1.  What is the difference between a primary group and a secondary (supplementary) group?
2.  How do you change the ownership of a user’s home directory?
3.  Can you remove a user from the system without deleting their home directory? How?

Further reading
Here are a few Packt titles that can help you with the task of user management:

*   *Mastering Ubuntu Server – Fourth Edition*, *Jay LaCroix*
*   *Red Hat Enterprise Linux 9 Administration – Second Edition*, *Pablo Iranzo Gómez*, *Pedro Ibáñez Requena*, *Miguel Pérez Colino*, and *Scott McCarty*

```

```

```

```