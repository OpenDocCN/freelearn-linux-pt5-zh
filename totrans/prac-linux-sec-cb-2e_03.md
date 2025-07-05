# Local Filesystem Security

In this chapter, we will discuss the following:

*   Viewing files and directory details using `ls`
*   Using `chmod` to set permissions on files and directories
*   Using `chown` to change ownership of files and directories
*   Using ACLs to access files
*   Implementing Mandatory Access Control with SELinux
*   Using extended file attributes to protect sensitive files
*   File handling using the `mv` command (moving and renaming)
*   Installing and configuring a basic LDAP server on Ubuntu

# Viewing files and directory details using ls

The `ls` command is used to list files in a directory, and it is similar to the `dir` command in DOS. This command can be used with various parameters to give different results.

# Getting ready

Since the `ls` command is a built-in command in Linux, we don't need to install anything else to use it.

# How to do it…

Now, let’s take a look at how we can use `ls` in different ways to get a variety of results by just following these steps:

1.  To take a look at the simple listing of files in a current directory, type `ls`:

![](img/1eaf670a-653b-45f1-8d70-51fbe6b05096.png)

2.  To get more information about the files and directories listed using the `ls` command, add a type identifier as shown here:

![](img/5855e3e6-8059-4cbc-8c07-57d3f0a25fa6.png)

3.  When the preceding identifier is used, the executable files have an asterisk at the end of the name, while the directories have a slash, and so on. To check out details of files, such as the creation dates, owners, and permissions, run a command with the `l` identifier, as shown here:

![](img/32345510-763f-4056-9dd9-e8b69833c823.png)

4.  To find a listing of all the hidden files in the current directory, use the `a` identifier, as shown here:

![](img/065e6007-4fb9-4a1e-8427-8d2b5b59fb12.png)

5.  Files that begin with a period (also called dot files) are hidden files, they are not shown if the `–an` option is not used.

To print the file size in readable form, such as MB, GB, TB, and so on, instead of printing in terms of bytes, we can use the `-h` identifier along with the `-l` identifier, as shown here:

![](img/54f34c79-25a1-4961-a3f9-feedbafbb064.png)

6.  If you wish to exclude all the files and display only their subdirectories, then you can use the `-d` option as follows:

![](img/0629a12f-7765-4e32-a39e-eddc42988b30.png)

7.  The `ls` command, when used with the`-R` option, will display the contents of the subdirectories too:

![](img/89c1aa34-8c3c-4050-87f9-7d35914493f9.png)

# How it works…

When we use different options with the `ls` command, it gives us different results in terms of listing a directory. We can use any option, as per our requirements.

It is recommended that you get into the habit of using `ls –lah` so that you can always find the listing in readable sizes.

# Using chmod to set permissions on files and directories

**chmod**, or **Change Mode**, is a Linux command used to modify the access permissions of files and directories. Everybody wants to keep their data secure and properly organized. For this reason, Linux has the concept of associating an owner and group with every file and directory. These owners and groups have different permissions to access a particular file.

# Getting ready

Before we see the use of the `chmod` command, we need to know that, for different types of users, the symbolic representations that are used are as follows:

*   `u` is used for user/owner
*   `g` is used for group
*   `o` is used for others

Now, let's create a file named `file1.txt` to check out the different commands of chmod.

# How to do it...

Now, we will see how to use chmod in different ways to set different permissions:

1.  We first check the current permission of the file and when it was created:

![](img/2ff419a0-d906-4059-ba18-c7e47235094b.png)

As we can see, currently only the user/owner has read and write permission, whereas groups and other users have only read permission.

2.  Now, let's change a single permission for the owner by using the `+` symbol, which is for adding permission:

```
chmod u+x file1.txt
```

![](img/58ffd21c-9947-4c11-b88d-69709a9158a8.png)

This command will add the execute permission for the owner of the file.

3.  To add multiple permissions, we can do the same in a single command. We just need to separate different permissions using a comma, as shown here:

```
chmod g+x,o+x file1.txt
```

![](img/03134130-a54e-44e6-b319-b3436b67cecf.png)

This command will add the Execute permission for the group and other users of the file.

4.  If we wish to remove a permission, just use the `-` symbol instead of `+`, as shown here:

```
chmod o-x file1.txt
```

![](img/74c3e6ea-f854-42df-a4a9-b0c9ce35e396.png)

This will remove the Execute permission for other users of the particular file.

5.  If we wish to add or remove a permission for all users (owner, group, and others), this can be done with a single command by using the `a` option, which signifies all users.

To add read permission for all users, use this:

```
chmod a+r file1.txt
```

To remove read permission for all users, use this:

```
chmod a-r file1.txt
```

![](img/3ffcb4fc-07d5-4703-8dcc-874e17efd5bd.png)

6.  Suppose we want to add a particular permission to all the files in a directory. One way to do it is what we have seen already, by running the command for all the files individually. Instead, we can use the `-R` option, which signifies that the given operation is recursive. So, to add execute permission for other users, for all files in a directory, the command will be this:

```
chmod o+x -R /dir1
```

![](img/142056ae-55c5-4c9d-aaa3-117b6cd4d28b.png)

7.  We can also copy the permissions of a particular file to another file by using the `reference` option:

```
chmod --reference=file2 file1
```

![](img/13611a95-3019-465c-99ce-e78aa10d34ed.png)

Here, we have applied the permissions of `file2` to another file, called `file1`. The same command can be used to apply permissions of one directory to another directory.

# How it works...

When chmod is used with symbolic representation, we know the following:

*   `u` is used for user/owner
*   `g` is used for group
*   `o` is used for others

Also, different permissions are to referred as follows:

*   r: read
*   w: write
*   x: execute

So, using these commands, we change the permissions for the user, group, or others as per our requirements.

# There's more...

Permissions can be set with chmod using numbers also, known as Octal representation. We can edit permissions for the owner, group, and others, all at the same time using this method. The syntax of the command is as follows:

```
chmod xxx file/directory
```

Here, xxx refers to a three-digit number from 1-7\. The first digit refers to the owner's permission, the second represents group, and the third digit refers to the permissions of others.

When we use octal representation r, w, and x permissions have specific number values:

*   r = 4
*   w = 2
*   x = 1

So, if we want to add permission to read and execute, it will be calculated as follows:

*   r-x = 4+0+1 = 5

Similarly, permission to read, write, and execute is calculated as follows:

*   rwx = 4+2+1 = 7

And if we wish to give only read permission, it will be this:

*   r-- = 4+0+0 = 4

So, now, if we run the given command, it gives the permission as calculated:

```
chmod 754 file1.txt
```

![](img/4ad69d19-527c-48f6-8185-40f7fe3cb1fb.png)

# Using chown to change ownership of files and directories

File ownership is fundamental in Linux. As every file is associated with an owner and a group, we can change the owner of a file or directory using the `chown` command.

# How to do it...

To understand the use of `chown`, let's follow these steps:

1.  To understand the use of the `chown` command, let's create a file named `file1.txt` and a user named `user1`:

![](img/814962fe-0a6e-470e-b0b8-ab7b04bcd718.png)

The previous command for changing password information is optional. You can ignore it if you want to.

2.  Now, let's check the current owner of `file1.txt`.

We can see that the current owner for both the files is root and it belongs to the root group.

3.  Let's change the ownership of `file1.txt` to `user1`:

![](img/ef69031f-109a-410f-8f93-45ebd96cecc9.png)

As seen here, the owner of `file1.txt` has now changed to `user1`.

4.  If we want to change the group of a file, we can do that also using `chown`:

![](img/f3035b26-7ed1-4b31-839d-ef0d53c9c5c8.png)

5.  We can change both the owner and group of a file in single command as shown here:

![](img/5bd5d7b2-8712-4fff-9b0f-80663635a5fa.png)

We can see that the owner and group of `file2.txt` have changed.

6.  If we wish to recursively change ownership of a directory and its contents, we can do it as shown:

![](img/0f141356-75e2-4fdd-a8c7-66b4fabc5ea8.png)

7.  Chown can also be used to copy the owner/group permissions from one file to another:

![](img/59a26040-d415-4f1c-99c6-df12c5a1ca72.png)

# There's more...

If we wish to list the changes made by the `chown` command in a verbose manner, we can do so using this command:

![](img/699840b0-e58f-41dd-888d-27452c1abfcc.png)

# Using ACLs to access files

Implementing the basic file permissions using `chmod` is not enough, so we can use ACLs, or *Access Control Lists*. In addition to providing permissions for the owner and group for a particular file, we can set permissions for any user, any user group, or a group of all users who are not in the group of the particular user using ACLs.

# Getting ready

Before using ACLs, we check whether it is enabled or not:

1.  To do this, we try to view the ACLs for any file, as shown here:

![](img/7829684c-7691-4c4d-9041-9be4092d6785.png)

This command will show an output like this if ACLs are enabled. In our case, they are not enabled for `/dev/sda1`, as it is not listed in the mount options.

2.  To enable an ACL, we will add it to the filesystem, using the following command:

![](img/a17a5e94-8078-426b-86d6-d21bfd0c91dc.png)

3.  Now, run the `tune2fs` command again to confirm the ACL is enabled:

![](img/1a55be24-69f1-4da7-98dc-058ad9063382.png)

Now, we can see the ACLs option in the `/dev/sda1` partition.

# How to do it...

To understand the workings of ACLs, let's follow these steps:

1.  We will first check the default ACL values for any file or directory. To do this, we use the `getfacl` command:

![](img/7c59a5c8-ba8e-417b-90b6-7b70ab08b0bf.png)

2.  Now, to set an ACL on `file1.txt`, we will use the following command:

![](img/e8d79916-7432-49b7-8d88-846e6b4b9285.png)

In the preceding command, we have given `rwx` access to the root user for the `file1.txt` file.

When we check again using `getfacl`, we can see the new ACL values.

3.  Run the given command to confirm ACL setup on `file1.txt`:

![](img/7ba86c7e-c85d-40c2-b71a-6180433b9ddb.png)

The (`+`) sign after the file permissions confirms it has the ACL set up.

4.  We can also set an ACL on a folder recursively, by using the `setfacl` command. First, we will check the default ACL for our `dir1` directory, using `getfacl`:

![](img/b073351e-c63d-4117-9082-92de4864bcb0.png)

5.  Now, let's add `rwx` access to the `dir1` directory using `setfacl`:

![](img/82f8563b-98a6-4d4c-b921-a1c6a5aaddbd.png)

6.  We can now confirm that the ACL is set up on the directory using the given command:

![](img/739e3c85-4fc0-4cc7-b341-b90795cefb1c.png)

7.  Now, let's check the ACL for the `user1` group on `file1.txt`:

![](img/d19453dc-c648-4c44-ad71-40d60c1c31b0.png)

8.  Let's set `rwx` access to the group `user1` on `file1.txt`:

![](img/4d91341e-c770-4dab-8981-66ddb303a8b5.png)

# There's more...

Whenever we deal with file permissions, it is a good idea to take a backup of the permissions if your files are important.

Here, we suppose that we have a `dir1` directory that contains a few important files. Then, we first navigate to the `dir1` directory and take a backup of the permissions using this command:

```
getfacl -R * > permissions.acl
```

![](img/e1f95cd0-e827-4252-815a-2f947b7b6ee1.png)

This command creates a backup of the permissions and stores it in `permissions.acl`.

Now, if we want to restore the permissions, the command will be as follows:

```
setfacl --restore=permissions.acl
```

![](img/ab45ee42-99e4-423f-8683-83d7c3a08411.png)

This will restore all the permissions back to where they were while creating the backup.

# File handling using the mv command (moving and renaming)

The `mv` or `move` command is used to move files from one directory to another without creating any duplicates, unlike the case with the `cp` or copy command.

# Getting ready

Since `mv` is a built-in command on Linux, we don't have to configure anything else to understand its workings.

# How it works...

Let's take a look at how to use the `mv` command by taking different examples:

1.  To move `testfile1.txt` from the current directory to any other directory, such as `- home/practical`/`example`, the command will be as follows:

```
    mv testfile1.txt /home/practical/example
```

![](img/1a1fb844-c8bd-4478-8fb8-91d84ca24b8e.png)

This command will work only when the location of the source file is different from the destination location.

When we move the file using the preceding command, the file will get deleted from the original location.

2.  To move multiple files using a single command, we can use this command:

```
mv testfile2.txt testfile3.txt testfile4.txt /home/practical/example
```

When using the preceding command, all the files that we are moving should be in the same source location.

3.  To move a directory, the command is the same as that used for moving a file. Suppose we have a `directory1` directory in the present working directory and we want to move it to the location `/home/practical/example`; the command will be as follows:

```
    mv directory1 /home/practical/example
```

![](img/d68088f2-b8ac-49c5-8dc2-0c59a52598d9.png)

4.  The `mv` command is also used to rename files and directories. Let's say we have a file, `example_1.txt`, and we wish to rename it to `example_2.txt`. The command for doing this will be as follows:

```
    mv example_1.txtexample_2.txt 
```

This command will work only when the source and the destination are the same:

![](img/58d9bbe7-c96a-4bfa-a47c-e93a40c1409c.png)

5.  Renaming a directory works the same way as renaming a file. Suppose we have a directory, `test­­_directory_1`, and we want to rename it to `test­­_directory_2` ; the command will be as follows:

```
    mv test­­_directory_1/ test­­_directory_2/
```

![](img/82ec9116-5d4b-423a-b02d-12f01b645979.png)

6.  When we use the `mv` command to move or rename a large number of files or directories, we can check that the command works successfully by using the `-v` option while using the command.

Suppose we want to move all text files from the current directory to `/home/practical/example`, and we want to ensure it works correctly; the command will be as follows:

```
    mv -v *.txt /home/practical/example
```

![](img/59dc98e6-fdc6-4870-b144-0c6b71a93174.png)

The same works when moving or renaming a directory, as shown here:

![](img/c918ffce-71c6-4ec1-a8f5-942db34e1305.png)

7.  When we use the `mv` command to move a file to another location and a file with the same name already exists, then the existing file gets overwritten when using the default command. However, if we wish to show a pop-up notification before overwriting the file, then we have to use the `-i` option, as shown here:

```
mv -i testfile1.txt /home/practical/example
```

![](img/2d2f215f-732a-4af9-ae4c-0df22369c231.png)

When the preceding command is run, it will notify us that a file with the same name already exists in the destination location. Only when we press *Y* will the command complete, otherwise it will get cancelled.

8.  When using the `mv` command to move a file to another location where a file with the same name already exists, if we use the `-u` option, it will update the file at the destination location only if the source file is newer than the destination file.

We have two files, `file_1.txt` and `file_2.txt`, at the source location. First, check the details of the files using the `ls` command:

```
    ls -l *.txt
```

Now, let's check the details of the files at the destination location:

```
    ls -l /home/practical/example/*.txt
```

Now, move the files using the given command:

```
    mv -uv *.txt /home/practical/example/
```

![](img/f9b21aff-cfe3-4500-9676-f732d1c8570a.png)

We see that `file1.txt` and `file2.txt` have been moved to the destination location, updating the earlier files because of the newer timestamp of the source files.

9.  We have two files, `file_1.txt` and `file_2.txt`, at the source location. First, check the details about the file using this command:

```
    ls -l *.txt  
```

Now, move the files using this command:

```
    mv -vn *.txt /home/practical/example/
```

Now, let's check the details of the files at the destination location:

```
    ls -l /home/practical/example/*.txt
```

We see that the files with the same name have not been moved, which can be checked by observing the time stamp:

![](img/c6e7038e-5b33-4aa7-b2c9-5b6ddc445777.png)

10.  When moving files, if the destination location already has a file with the same name, then we can also create a backup of the destination file before it is overwritten by the new one. To do this, we use the `-b` option:

```
    mv -bv *.txt /home/practical/example
```

Now, let's check the details of the files at the destination location.

We see in the details that we have files named `file1.txt~` and `file2.txt~`. These files are the backup files, which can be verified by the timestamp, which are older than those of `file1.txt` and `file2.txt`:

![](img/5ab6454a-a1c1-4845-ae90-4cb65cf7a60e.png)

# Implementing Mandatory Access Control with SELinux

**SELinux** (short for **Security-Enhanced Linux**), is a flexible Mandatory Access Control (MAC) devised to overcome the limitations of standard ugo/rwx permissions and ACLs.

# Getting ready

In most Linux distributions, such as CentOS and Redhat, SELinux is by default incorporated in the kernel. However, if we are working on any other distribution, such as Debian, we may have to install and configure SELinux on the system:

1.  First, we have to get the basic set of SELinux utilities and default policies by running the following command:

![](img/0c1ab45b-2d7c-4582-816b-dcda933fc4e6.png)

2.  Once the installation has completed, run the following command to configure GRUB and PAM, and to create `/autorelabel`:

![](img/d4e98939-f819-4207-9f1c-790d5039209b.png)

After this, you have to reboot the system to label the filesystems on boot.

3.  After reboot, when the system starts, you may get the following warning:

![](img/986440c4-666a-402e-b7fb-c4f217985eb1.png)

Now, we have a working SELinux system.

# How to do it...

Once we have a working SELinux system, we can choose how to use it:

1.  SELinux can operate in two different ways: Enforcing and Permissive.
2.  Let's check the current mode by using the `getenforce` command:

![](img/7a0dd6ac-ad1f-4ad8-ae10-76bd5f46ce09.png)

As we can see, SELinux is currently working in Permissive mode.

3.  Now, if we wish to toggle the mode, we can use the `setenforce` command:

![](img/3d5b6919-defa-436f-9680-30a8fa681ebf.png)

When we run the preceding command with option 1, the mode changes to Enforcing.

4.  If we want to toggle back to Permissive mode, we can again use the `setenforce` command with option 0:

![](img/a9a61390-a532-482c-8e1b-cef70e98f2c8.png)

Now, the mode has changed back to Permissive.

5.  This change will not survive system reboot. If we want to change the mode permanently, we have to edit the `/etc/selinux/config` file and change the SELinux variable to what we want: enforcing, permissive or disabled.
6.  Whenever an application is misbehaving or not working as expected, as part of the troubleshooting, we should try toggling between SELinux modes (from Enforcing to Permissive, or vice versa). This will help us understand if the application works after changing to Permissive mode or not. If it does, then we know that it's an SElinux permission issue.

# How it works...

SELinux operates in two modes, Enforcing and permissive. When working in Enforcing mode, SELinux denies access based on how SELinux policy rules have been defined. These rules are a part of the guidelines that control the security engine.

When SELinux is working in Permissive mode, it does not deny access, but any actions that would have been denied in enforcing mode get logged.

SELinux rules can be modified to allow any application or service that would otherwise be denied in Enforcing mode.

# There's more...

You can learn more about the `mv` command by typing `man mv` or `mv --help`. This will display its manual page, where we can explore more details about the command.

# Using extended file attributes to protect sensitive files

Extended attributes are an extensible way to store metadata in files. These are name-value pairs associated with files and directories. Several filesystems support extended file attributes that enable further customization of allowable file operations.

# Getting ready

At present, in Linux there are four different namespaces for extended file attributes:

*   `user`
*   `trusted`
*   `security`
*   `system`

Many tools are available for manipulating extended attributes, and these are normally included in the `attr` package, which comes with most Linux distributions.

If `attr` is not installed in your system, simply execute the following command to install it:

```
sudo apt-get install attr
```

To check if this package is installed on our system, just run `attr` in the Terminal. If an output appears as shown here, it will confirm that the package is installed:

![](img/f8d07d86-6ec1-4938-a97a-84d5bff2f236.png)

The next step is to check if the kernel has support for the attribute. It can be checked using the following command:

![](img/9dbf665e-68de-4b5c-a0b5-7247b50b807c.png)

The output confirms that the kernel has attribute support.

We also need to confirm that the filesystem is mounted with the `user_xattr` option. To confirm this, we run this command:

![](img/8266c8ab-4daa-4be0-8fd0-7c3911471488.png)

# How to do it...

Once we have confirmed all the requirements mentioned in the previous section, we can move on to see how to use extended attributes:

1.  First, we will create a test file with some dummy data in it:

![](img/c742473c-b857-47ea-acfb-041c98cf0f0b.png)

2.  Now, we will add an extended attribute to this file using the `setfattr` command:

![](img/06ea09dd-fa24-4d49-b418-b86ebafd1f8a.png)

3.  Now, let's determine the attributes on the file, using the `getfattr` command:

![](img/7bf1b227-c31a-4ecf-bbf2-d26ef0e842bb.png)

4.  If we want to check the values of the attributes, we can do that by using the following command:

![](img/4f8782dd-d136-4810-a670-496ecef33ff8.png)

5.  At any time, if we want to remove any attribute, we can do the same using the `setfattr` command, but with the `-x` option.

# Installing and configuring a basic LDAP server on Ubuntu

**Lightweight Directory Access Protocol** (**LDAP**) is a protocol for managing access to a file and directory hierarchy from some centralized location. LDAP is mainly used for centralized authentication.

An LDAP server helps in controlling who has access to read and update information in the directory.

# Getting ready

To install and configure LDAP, we need to first create an Ubuntu server. The current version for Ubuntu server installation media can be found here: [http://www.ubuntu.com/download/server](http://www.ubuntu.com/download/server).

After downloading, follow the steps provided to install an Ubuntu server.

We need a second system with a desktop version of Ubuntu installed. This will be used to access your LDAP server through a web interface.

Once this is done, we can proceed with the installation of LDAP.

# How to do it...

We shall now start to install and configure LDAP on the Ubuntu server:

1.  We will first update the package list on the server from Ubuntu's repositories to get information about the latest versions of all the packages and their dependencies:

```
sudo apt-get update
```

2.  Now, run the command to install the `slapd` package and some associated utilities, and `ldap-utils` to install the LDAP server:

![](img/c7947761-3708-4b04-bc8c-65cfab4ba829.png)

3.  During the installation process, when prompted, enter and confirm an administrator password, which will be used for the administrator account of LDAP. Configure a password of your choice and proceed with the installation process.
4.  Once the package is installed, we will reconfigure the LDAP package as per our requirements. To do so, type this command:

```
Sudo dpkg-reconfigure slapd
```

This will start a series of questions regarding configuring the software. We need to choose the options one by one as per our requirements.

5.  The first question asked is Omit OpenLDAP server configuration? Select No and continue:

![](img/f5c3a78d-5865-45d9-ba5f-3a96845741da.png)

6.  Next, enter the domain name. An already existing domain name on the server can be used, or a new one can be created. We will use `example.com` here:

![](img/e6fff28c-2373-4748-a587-f157b719c7bf.png)

7.  Next, enter the `Organization name`:

![](img/2e09a4df-f3d5-4d4a-9ae0-c5ec7f328f28.png)-

8.  Next, configure the administrator password for LDAP. Use the same as configured during the installation process, or change it to something else in this step.
9.  Next, set Database backend to use. Select `MDB` and continue:

![](img/e612be60-1c9d-468c-a5c8-6e8411b92197.png)

10.  Now, you will be asked if you wish to remove the database when slapd is purged. Select **No** here.
11.  Next, select Yes to move the old database and allow the configuration process to create a new database:

![](img/c2e74dfc-97cf-4d11-a0cd-eba83e31d1ad.png)

12.  When asked `Allow LDAPv2 protocol?` choose `No`, as the latest version is LDAPv3, and LDAPv2 is obsolete now:

![](img/8b37381d-0e75-42cf-bf7d-79a542a97c39.png)

13.  At this point, the configuration process is done and LDAP is running:

![](img/15b5c75f-7edb-4ff5-9d70-72c0ba66c778.png)

14.  Let's now open the firewall port for LDAP so that external users can use it:

![](img/5e5ffd54-7219-4012-89cf-a55d40157f2e.png)

15.  We will now install the PHPldapadmin package, which will help in administering LDAP through the web interface:

```
sudo apt-get install phpldapadmin
```

![](img/fe6225ff-64d3-47fd-b102-7eaa8b2518dc.png)

Once the installation completes, edit the configuration file of PHPldapadmin to configure a few values:

```
Sudo nano /etc/phpldapadmin/config.php
```

16.  Now, search for the given section and modify it to reflect the domain name or the IP address of the Ubuntu server:

```
$servers->setValue('server','host','domain_name_or_IP_address');
```

17.  Next, edit the following entry and insert the domain name that we gave when we reconfigured `slapd`:

```
$servers->setValue('server','base',array('dc=example,dc=com'));
```

Give the domain name as values to the `dc` attribute in the previous line. Since our domain name was `example.com`, the value in the previous line will be entered as `dc=example, dc=com`.

18.  Next, find the following line and again enter the domain name as the `dc` attribute. For the `cn` attribute, the value will be `admin` only:

```
$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
```

19.  Now, search for the section that reads as follows, and first uncomment the line, then set the value to `true`:

```
$config->custom->appearance['hide_template_warning'] = true;
```

After making all these changes, save and close the file.

20.  When the configuration of PHPldapadmin is complete, open a browser in the other system, which has the desktop version of Ubuntu. In the address bar of the browser, type the domain name or the IP address of the server, followed by `/phpldapadmin`:

```
domain_name_or_IP_address/phpldapadmin
```

21.  Once the PHPldapadmin page opens, on the left-hand side we find the login link. Click on it and you will get a login prompt:

![](img/1010e3d9-a24a-4e50-8a3e-34521a27068d.png)

The login screen will have the correct Login DN details if PHPldapadmin was configured correctly:

![](img/995949ad-6294-4c58-800c-0250856c5d7b.png)

This is `cn=admin,dc=example,dc=com` in our case.

22.  Once you enter the administrator password correctly, the admin interface will be shown:

![](img/d51ceec9-28dd-4085-80f8-aa0c8baa0d1e.png)

23.  On the admin interface, on the left-hand side where you see the domain components (`dc=example,dc=co`), click on the plus sign next to it. It will show the admin login being used:

![](img/e4220f48-1d92-4616-9fc7-60e0f0956757.png)

24.  Our basic LDAP server is now set up and running.

# How it works...

We first create an Ubuntu server, and then on top of it, we install the `slapd` package for installing LDAP. Once it is completely installed, we install the additional package required. Then, we reconfigure LDAP as per our requirements.

Once reconfiguration is complete, we install the PHPldapadmin package, which will help us in managing the LDPA server through the web interface using a browser.