# Local Authentication in Linux

In this chapter, we will discuss the following:

*   User authentication and logging
*   Limiting login capabilities of users
*   Disabling username/password logins
*   Monitoring user activity using acct
*   Login authentication using a USB device and PAM
*   Defining user authorization controls
*   Access management using IDAM

# User authentication and logging

One of the major parts of user authentication is to monitor the users of the system. There are various ways to keep a track of all the successful and failed login attempts made by a user in Linux.

# Getting ready

Linux system maintains a log of all login attempts by different accounts in the system. These logs are all located at `/var/log/`:

![](img/1c7049f0-1ff1-4296-b8d6-a235846c9c11.png)

# How to do it...

Linux has many ways to help an administrator to view the logs, both through graphical and command-line methods:

1.  If we want to check the incorrect login attempts for a particular user, like root for instance, we can do so by using this command:

```
    lastb root  
```

**![](img/de98a228-391c-45ed-a560-bd0904c5a103.png)**

2.  To see the log using the Terminal, we use the `dmesg` command. This command displays the buffer of the Linux kernel's message stored in memory, as shown here:

![](img/83101ba7-3b2c-406f-b643-b2a565459761.png)

3.  If we wish to filter the preceding output to show only the logs related to USB devices, we can do so by using `grep`:

![](img/6dd12dce-a46f-4a0a-ae66-a8438d401336.png)

4.  Instead of viewing all the logs, if we wish to view only the 10 most recent logs in a particular log file, the command will be as follows:

![](img/4d5d7bfa-0417-4dd0-ba3b-881471cf2c26.png)

In the preceding command, the `-n` option is used to specify the number of lines to be shown.

5.  If we wish to see the most recent login attempts for a user account, use `last`:

![](img/4a3dea49-0f04-466c-bcf3-a953c1701a55.png)

The `last` command displays `var/log/wtmp` in a formatted way.

6.  If we want to see the last time each user logged in to the system, we can use the `lastlog` command:

![](img/5bf77f57-19b7-45cc-8cc0-315b0db07065.png)

# How it works...

Linux has different files for logging different types of details. Using the commands shown here, we are able to view those logs and see the details. Every command gives us a different type of details.

# Limiting login capabilities of users

As a Linux system administrator, you may want to restrict access to a Linux system for specified users or groups. In this section, we will learn how to use two files, `/etc/securetty` and `/etc/security/access.conf`, to restrict user access.

# Getting ready

All the steps given here have been tried on an Ubuntu system; however, you can follow these on any other Linux distribution also.

# How to do it...

Let's learn how to edit the two files mentioned before to apply different restrictions to user access on a Linux system:

1.  First, we will check the content of the `/etc/securetty` file using the `more` command:

```
more /etc/securetty
```

![](img/b36bb05d-9d64-4752-8220-e3553130bac4.png)

As we can see in the preceding screenshot, the file specifies the Terminals on which root is allowed to log in.

2.  Now, edit the file using any editor of your choice and comment the `tty1` entry as shown here:

![](img/0fdb046e-c84f-4634-ac2b-02d67dea3b58.png)

Save and exit the editor after making the changes mentioned in the preceding step.

3.  Now, switch to Terminal `tty1` by running the command `chvt 1`. If we try to log in as root now, we will get the following result:

![](img/7f5d0adc-fad1-4e8c-9fdb-10916b3b17e1.png)

We can see that access was denied to the root account by the system. If we wish to still get root privileges, we have to first log in as a normal user and then use the `sudo` or `su` command.

4.  From the same Terminal, when we try to log in from a normal user account, we get logged in as seen here:

![](img/c8d32cf8-411d-48bc-80f2-6010176269e3.png)

5.  We have already seen how to use the `/etc/securetty` file to specify access for the root account from any Terminal. Now, let's see how to allow or deny access to specific users.
6.  The first thing to do is to modify the `/etc/pam.d/login` file and add the `pam_access.so` module. This will allow `pam` to scan the `/etc/security/access.conf` file and check for the rules defined by us.

So we open `etc/pam.d/login`, find the line that states `#account required pam_access.so`, and remove the `#` to un-comment the line:

![](img/b674ec31-6c4b-4b60-b51b-cbd4c81b578f.png)

7.  Next, we will define a rule in `/etc/security/access.conf`. Open the file in any editor and define a rule according to the following syntax:

```
permission : users : origins
```

8.  If we want to deny access to the root account from Terminal `tty1` we use the following rule:

![](img/3ab12ffd-eeef-427c-8a69-0f1afc46580c.png)

9.  To deny access to `user1` we use the following rule:

![](img/9c027ee8-22eb-4bbe-80e4-e1a1a12b32d6.png)

10.  If we want to specify multiple usernames in the same rule, we can do it as shown in this rule:

![](img/a1afaef3-efab-4ae4-9c96-3b79185305d7.png)

# How it works...

Linux uses `/etc/securetty` to specify from which Terminal root access is possible. So, when we make changes in this file, root access from a specific Terminal also gets affected.

Similarly, the `/etc/security/access.conf` file is used by `pam` to check if Terminal access is allowed for a particular user or not. The rules defined in this file follow this syntax:

```
permission : users: origins
```

Here, `permission` refers to denying or allowing a rule and is denoted by the `-` or `+` sign.

Users refers to a list of login names.

Origins refers to the source from which access is being allowed or denied.

# Disabling username/password logins

One major role of a system administrator is to configure and manage users and groups on a Linux system. It also involves the task of checking the login capabilities for all users and disabling them if required.

# Getting ready

All the steps given here have been tried on an Ubuntu system; however, you can follow these on any other Linux distribution also.

# How to do it...

Here, we will discuss how the login capabilities of users can be restricted on a Linux system:

1.  We can restrict the access of a user account by changing the login shell of the account to a special value in the `/etc/passwd` file. Let's check the details of an account, `user1` as an example, in the `/etc/passwd` file, as shown here:

![](img/0a7031f2-9218-4f60-b6ae-b03e7c7e29f8.png)

2.  In these details, the final value for the `user1` account is set to `/bin/bash`. At present, we can log in from the `user1` account. Now, if we want to change the shell of the user account we wish to restrict, we can do so as shown here:

![](img/12951079-090c-4d03-8ecf-4b4111ffd749.png)

3.  If we try to log in from user 1 now, we get the following error:

**![](img/946c4bf0-402c-4362-8343-a4e5d7415daf.png)**

4.  Another way of restricting access to users is by using the `/etc/shadow` file. If we check the details of this file using the `cat` command, we get the result shown here:

**![](img/a752604c-69a8-4f9d-be48-b50421db82cf.png)**

5.  The details show the hashed password for the `user3` account (the one starting with ... $6$wI1akgI4...).

6.  Now to lock the account `user3` the command will be as follows:

```
 passwd -l user3 
```

![](img/62aafc31-ffa0-4b3d-bcc4-6f5a667e445c.png)

Let's check the details in the `/etc/shadow` file again for the `user3` account. We see that the hashed password has been made invalid by preceding it with a `!`.

```
cat /etc/shadow | grep user3
```

**![](img/65e83945-ff4d-42c2-94b5-f249764630b0.png)**

7.  To unlock the account again, the command is as follows:

```
    passwd -u user3
```

8.  If we wish to check whether the account was already locked or not, we can do so by using this command:

**![](img/7455f7c6-70ab-4e6b-9284-15562b112e22.png)**

As we can see in the output, the `user1` account is locked, which is denoted by `L` in the second field of the details, while `user2` is not locked, as it shows `P` in the details.

9.  The process to lock or unlock an account can also be done using the `usermod` command. To lock the account using `usermod`, the command will be as follows:

```
    usermod -L user2
```

**![](img/69598742-5811-449f-89dd-445fd03b1b66.png)**

10.  Once locked, if we try to log in from that account, we get the following error:

**![](img/4cf74ec9-8847-4e3a-9676-8d1749221fac.png)**

11.  And to unlock the account using `usermod`, the command will be as follows:

```
    usermod -U user2
```

**![](img/e3b09297-33fe-42b3-bfe4-4f0639737f7e.png)**

# How it works...

For every account in Linux, the user account details are stored in the `/etc/passwd` and `/etc/shadow` files. These details specify how the user account will act. When we are able to change the details for any user account in these files, we are able to change the behavior of the user account.

In the preceding section, we have seen how to modify these files to "lock" or "unlock" the user account.

# Monitoring user activity using acct

**Acct** is an open source application which helps in monitoring user activity on a Linux system. It runs in the background and tracks all the activities of the users, and also maintains a track of the resources being used.

# Getting ready

To use the `acct` commands, we first need to install the package on our Linux system by using this command:

```
    apt-get install acct
```

**![](img/90f5bf3d-0655-4b40-ab10-5977261028b4.png)**

In case the preceding method doesn't work properly, we can download the package manually by visiting this link:

[http://packages.ubuntu.com/precise/admin/acct](http://packages.ubuntu.com/precise/admin/acct)

After downloading the package, we need to extract it into a directory somewhere, like we did on the desktop:

![](img/0a52a7cd-e853-485d-8f54-8ab5a6cd22a0.png)

Then, move into the directory:

![](img/1712e431-5fe1-404c-b1c9-74413c2ed50b.png)

Next run the script to configure the package:

![](img/cc3b79a1-ed6c-43e8-914c-8ff1f0485e37.png)

Once the configuration is complete, next we run the `make` command:

![](img/56a7cc00-99f7-4194-a118-30798147c8cf.png)

Then, run the `make install` command:

![](img/365db1da-ef10-4117-8283-86dfdd12e28a.png)

Once successfully done, it will install the package on your Linux system.

# How to do it...

The acct package has different commands for monitoring process activities:

1.  Based on a particular user login and logout from a wtmp file, if we wish to check the total connect time, we can use the `ac` command:

![](img/aa2a8e2e-7fe4-4ecc-81aa-87c0119a7c17.png)

2.  If we wish to print the total login time day-wise, we will use the `-d` option with the "ac" command:

![](img/1cb7cb36-1355-4159-883a-abbeca8f5cf4.png)

3.  To print the total login time user-wise, we use this command:

![](img/45c4b839-36a4-46e2-ae8f-3d0998faa416.png)

4.  If we wish to check the login time only for a particular user, we use this command:

![](img/2f5ec9f6-c1f6-40b9-b68c-7c69b4be25e1.png)

5.  We can also see the previously executed commands for all users, or a particular user, by using the `lastcomm` command:

![](img/78a25715-1803-4452-a36a-9a9e5f3c59f1.png)

# How it works...

To monitor the system, we first install the acct package on the system. For a few other Linux distributions, the package to be used would be `psacct`, if `acct` is not compatible.

Once the tool is installed and running, it starts maintaining a log of the activities on the system. We can then watch these logs using the commands discussed in the preceding section.

# Login authentication using a USB device and PAM

When a Linux user wants to secure the system, the most common method is always using their login password. However, we know this method is not very reliable as there are many methods to hack a traditional password. To increase the security level, we can use a USB device like an authentication token, which will be used to log in to the system.

# Getting ready

To follow these steps, we need to have a USB storage device and **Pluggable Authentication Modules** (**PAM**) downloaded on the Linux system. Most Linux systems have PAM in the form of precompiled packages, which can be accessed from the relevant repository.

# How to do it...

By using any type of USB storage device and PAM, we can create the authentication token:

1.  To start with, we first need to install the packages required for PAM USB authentication. To do so, we run this command:

```
 $ sudo apt-get install pamusb-tools libpam-usb
```

**![](img/8900ebd5-14b1-4eb9-9219-6f9e861f41a7.png)**

2.  Once the packages are installed, we have to configure the USB device to use PAM authentication. To do so, we can either use a command, or else we can edit the `/etc/pamusb.conf` file.

To use the command method, first connect the USB device, and after that execute this command:

```
    $ sudopamusb-conf --add-device usb-device  
```

![](img/6d19d713-34fb-412e-80e0-1bacc5b1540a.png)

In the preceding command, `usb-device` is the name given to the USB device we are using. This name can be anything you choose.

When the `pamusb-conf` command is used, it automatically discovers the USB device, which also includes multiple partitions. When the command completes its execution, it adds an XML code block to the `/etc/pamusb.conf` file, defining our USB device:

![](img/dd645efe-3898-4405-96dd-cb9850bed3cd.png)

3.  Next, we define our USB device:

```
    $ sudopamusb-conf --add-user user1
```

**![](img/a5fefc96-57ce-405f-a41c-0bb62c9c2de7.png)**

If the user already exists, it will be added to the PAM configuration.

The previous command adds the definition of the `pam_usb` user to the `/etc/pamusb.conf` file:

![](img/a5bb09ee-0418-40b4-a97d-87bbf8f93483.png)

4.  Now, we will configure PAM to add the `pam_usb` module in the system authentication process. For this, we will edit the `/etc/pam.d/common-auth` file and add this line:

**![](img/81034e61-4054-4a7c-aa26-20b451d9b71b.png)**

This will make the system-wide PAM library aware of the `pam_usb` module.

The `required` option specifies that the correct password is necessary, while the `sufficient` option means that this can also authenticate the user. In the preceding configuration, we have used `sufficient` for the usb-device authentication, but `required` for the default password.

In case the USB device defined for `user1` is not present in the system, the user will need to enter the correct password. To force the user to have both authentication routines in place before granting access to the system, change `sufficient` to `required`.

5.  Now, we will try to switch to `user1`:

![](img/90d9486f-c9b7-4662-ad5d-04fa27ac846c.png)

When asked for, connect usb-device. If the correct USB token device is connected, the login will complete, otherwise it will give an error.

6.  If any errors appear, such as the one shown here, it could be possible that the path of the USB device was not added properly:

```
    Error: device /dev/sdb1 is not removable
    * Mount failed
```

In such a situation, add the USB device's full path to `/etc/pmount.allow`.

7.  Now, run the command to check how the USB device partition has been listed in the filesystem:

```
    $ sudo fdisk -l
```

**![](img/7a8020bc-9339-42fe-b14a-8d496e951c5c.png)**

In our case, the partition has been listed as `/dev/sdb1`.

8.  Now, add a line into the `/etc/pmount.allow` file to solve the error.
9.  The configuration that we have done in `/etc/pam.d/common-auth` up to now means that if the USB device is not connected, the user will still be able to log in with the correct password. If we wish to force the user to also use the USB device for login, then change `sufficient` to `required`, as shown here:

![](img/1b9019e4-1dde-4e22-94eb-74ff4acf9dc9.png)

10.  If the user now tries to log in, they will have to enter the correct password as well as insert the USB device:

![](img/fa64e912-955e-4127-94e3-650238ac112f.png)

11.  Now, remove the USB device and try to log in again with the correct password:

![](img/9f09639a-50ba-4341-aa79-0a53f3f46e3e.png)

# How it works...

Once we install the required `pam-usb` package, we edit the configuration file to add our USB device, which we want to use as an authentication token. After that, we add the user account to be used. And then, we make the changes in the `/etc/pam.d/common-auth` file to specify how the USB authentication should work, and whether it is always required to log in.

# There's more...

We have seen how to use a USB device to authenticate user login. Apart from this, we can also use the USB device to trigger an event, every time it is disconnected or connected from/to the system.

Let modify the XML code in `/etc/pamusb.conf` to add event code for the user definition:

**![](img/88992947-410f-4e95-83fc-31d1262c5411.png)**

Due to the preceding modification, whenever the user disconnects the USB device, the screen will get locked. Similarly, when the user again connects the USB device, the screen will get unlocked.

# Defining user authorization controls

Defining user authorization on a computer mainly deals with deciding the activities that a user may or may not be allowed to do. This could include activities such as executing a program or reading a file.

Since the `root` account has all privileges, authorization controls mainly deal with allowing or disallowing root access to user accounts.

# Getting ready

To see how user authorization works, we need a user account to try the commands on. So, we create few user accounts, `user1` and `user2`, to try the commands.

# How to do it...

In this section, we will go through various controls that can be applied on user accounts:

1.  Suppose we have two user accounts, `user1` and `user2`. We log in from `user2` and then try to run a command, `ps`, as `user1`. In a normal scenario, we get this result:

![](img/31598865-2f78-43c8-bac4-259d88cfb5a2.png)

2.  Now, edit the `/etc/sudoers` file and add this line:

```
    User2 ALL = (user1) /bin/ps
```

2.  After saving the changes in `/etc/sudoers`, again try to run the `ps` command from `user2` as `user1`:

![](img/4016a147-99df-414a-896b-7d550093e70e.png)

3.  Now, if we want to run the same command again from `user2` as `user1`, but without being asked for the password, we can do this by editing the `/etc/sudoers` file as shown here:

![](img/6bc486cc-080b-4642-b906-6078cdd1f065.png)

4.  Now, when we run the `ps` command from `user2` as `user1`, we see that it does not ask for a password anymore:

![](img/d6d60eb7-8f3c-4950-9e9f-5eea2a690544.png)

5.  Now that we have seen how to run a command without being asked for the password, the major concern of the system administrator will be that `sudo` should always prompt for a password.

6.  To make `sudo` always prompt for a password for the `user1` user account on the system, edit the `/etc/sudoers` file and add this line:

```
    Defaults:user1    timestamp_timeout = 0
```

![](img/b1474717-f0a3-4c1b-83f4-7ffa26345518.png)

7.  Now, if `user1` tries to run any command, they will be always prompted for the password:

![](img/b8e002a8-f79e-4d58-8157-557309a47cd1.png)

8.  Now, let's suppose we want to give the `user1` account permission to change the password of `user2` and `user3`. Edit the `/etc/sudoers` file and add this line:

![](img/e703e55d-d309-465e-a622-2b869c5cf631.png)

9.  Now, log in from `user1` and let's try to change the passwords of the `user2` and `user3` accounts:

![](img/54a92e48-fdd8-4861-a135-dd51ddbe5de3.png)

# How it works...

Using the `sudo` command and the `/etc/sudoers` file, we make the required changes to execute the tasks as required.

We edit the file to allow permission to execute a program as another user. We also add the `NOPASSWD` option to execute the program without being asked for a password. We then add the required line so that `sudo` always prompts for a password.

Next, we will see how to authorize a user account to change the password for other user accounts.

# Access Management using IDAM

In today's world, a single Linux system may be used by various users locally or remotely. It becomes essential to manage the access of these users to protect sensitive and confidential information that should be accessible to only a few authenticated users.

IDAM, or Identity and Access Management, tools can help a system administrator to manage the identity and access of various users easily.

# Getting ready

To get going with the installation and configuration of WSO2 Identity Server, we need any Linux distribution on which the Java environment is setup.

Here, we will see how to set up the Java environment:

1.  Before installing the JDK, we shall install a package related to Python as part of the dependency. The command to do this is as follows:

![](img/9fc871bf-6954-4f0b-9380-4e3a3a8e49f8.png)

2.  Now, to install Oracle JDK, the official version distributed by Oracle, we will have to update the system's package repository and add Oracle's PPA. To do this, we run the following command:

![](img/9d4c51b0-a263-4fb9-991e-0fbf09e1b638.png)

3.  Now, install the stable version of Java by running the following command:

![](img/d0ff03fa-ecd0-49eb-97f6-ce157a795de6.png)

4.  Once the installation completes, the next step is to set the `JAVA_HOME` environment variable. To do this, edit the `/etc/environment` file using any editor and add the following lines:

![](img/0eab836c-596e-498c-9faa-4fa9e5782003.png)

5.  To test if the environment variable has been set properly or not, execute the following command:

![](img/2df13433-9315-4415-9b0d-0b8a6a5aa27d.png)

We can see the path that has been set in the previous steps.

# How to do it...

Once we are done with the installation and configuration of the JDK on our system, we can proceed with the installation and configuration of WSO Identity and Access Management Server:

1.  To begin with, download the WSO2 package from the link given here: [https://wso2.com/identity-and-access-management/install/download/?type=ubuntu](https://wso2.com/identity-and-access-management/install/download/?type=ubuntu)

2.  Next, create a directory, `/var/wso2`, and unpack the downloaded package into this directory:

![](img/b8d28b0e-d2af-463d-ab3c-585058393356.png)

3.  To extract the package, run this command:

```
unzip ~/wso2is-5.6.0.zip /var/wso2
```

4.  Once the extraction process is complete, we can check the files inside the directory:

![](img/c4ff6390-daf8-48bc-9806-daccb0978130.png)

5.  Next, we can change the configuration in the `carbon.xml` file if we wish to configure our server to launch using FQDN instead of `localhost`. To do this, edit the `carbon.xml` file located at `[INSTALL_DIR]/repository/conf/carbon.xml`:

![](img/55c1c105-ab33-4966-b409-570d8da78dc6.png)

Make the changes to `<HostName>` to replace localhost with your system's FQDN:

![](img/7725e6fb-bbe5-4b52-bb7d-b964c9635675.png)

6.  Now, we can launch WSO2 Identity Server. To do so, we run the following command:

![](img/15e94b56-b57a-479d-85b0-9228d7a53efc.png)

7.  Once the server starts running successfully, it will display a line similar to `WSO2 Carbon started in 463 sec`, as shown in the following output:

![](img/5589b794-1147-44e2-8b81-29b880a4c50a.png)

8.  Once the server is up and running, we can access it through the browser. The default configuration to access the server is always via HTTPS ad on port `9443`:

![](img/0a153856-b2cb-4bf4-90a8-0a0ae8922f89.png)

On the sign-in page, use the default username `admin` and the default password `admin` to log in.

9.  Once logged in, we can use it to add users and roles for those users.

The Linux administrator can now use WSO2 IS to manage identities and perform access management.

# How it works...

WSO2 Identity Server is an open source IAM product. It specializes in access management, access control, identity governance administration, API security, and many other features.