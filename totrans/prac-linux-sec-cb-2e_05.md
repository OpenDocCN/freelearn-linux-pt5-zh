# Remote Authentication

In this chapter, we will discuss the following:

*   Remote server/host access using SSH
*   SSH root login disable or enable
*   Key-based login into SSH for restricting remote access
*   Copying files remotely
*   Setting up a Kerberos server with Ubuntu
*   Using LDAP for user authentication and management

# Remote server/host access using SSH

**Secure Shell** (**SSH**) is a protocol that is used to log onto remote systems securely and is the most commonly used method for accessing remote Linux systems.

# Getting ready

To see how to use SSH, you need two Ubuntu systems. One will be used as the server and the other as the client.

# How to do it...

To use SSH, you can use freely available software called OpenSSH. Once the software is installed, it can be used by the `ssh` command. We will look at how to use this tool in detail:

1.  If OpenSSH server is not already installed, it can be installed using the following command:

```
    sudo apt-get install openssh-server
```

![](img/a5148806-871d-4b0c-9cd7-2b2217734dd4.png)

2.  Next, we need to install the client version of the software:

```
    sudo apt-get install openssh-client
```

![](img/d94f5a00-5f54-443b-8512-c616b38a6346.png)

3.  For the latest versions, the SSH service starts running as soon as the software is installed. If it is not running by default, we can start the service by using the command:

```
    sudo service ssh start  
```

![](img/0d20de2a-32d7-4441-93a2-158cc4dad0ec.png)

4.  Now, to log in to the server from any other system using SSH, you can use the following command:

```
    ssh remote_ip_address
```

Here, `remote_ip_address` refers to the IP address of the server system. Also, this command assumes that the username on the client machine is the same as on the server machine:

![](img/3f76ed43-360c-415b-9e8b-d8d43bbb84e9.png)

If we want to log in for a different user, the command will be as follows:

```
    ssh username@remote_ip_address
```

![](img/54c76c67-f170-4d43-8d48-1b1c51ac4b27.png)

5.  Next, we need to configure SSH to use it as per our requirements. The main configuration file for `sshd` in Ubuntu is located at `/etc/ssh/sshd_config`. Before making any changes to the original version of this file, create a backup using the following command:

```
    sudo cp /etc/ssh/sshd_config{,.bak}
```

The configuration file defines the default settings for SSH on the server system.

6.  Opening the file in a text editor, we can see that the default port declaration on which the `sshd` server listens for the incoming connections is `22`. We can change this to any non-standard port to secure the server from random port scans, thus making it more secure. Suppose we change the port to `888`, then next time the client wants to connect to the SSH server, the command will be as follows:

```
    ssh -p port_numberremote_ip_address
```

![](img/0b88f5cb-b2b7-4620-86f2-06922c0a7b9d.png)

As you can see, when you run the command without specifying the port number, the connection is refused. When you mention the correct port number, the connection is established.

# How it works...

SSH is used to connect a client program to an SSH server. On one system we install the openssh-server package to make it the SSH server, and on the other system we will install the openssh-client package to use it as client.

Now, keeping the SSH service running on the server system, we try to connect to it through the client.

We use the configuration file of SSH to change settings such as the default port for connecting.

# Enabling and disabling root login over SSH

Linux systems have a root account that is enabled by default. Unauthorized users gaining root access to the system can be really dangerous.

We can disable or enable the root login for SSH as per our requirements to prevent the chances of an attacker getting access to the system.

# Getting ready

We need two Linux systems to be used as server and client. On the server system, install the openssh-server package, as shown in the previous recipe.

# How to do it...

First, we will see how to disable SSH root login and then we will also see how to enable it again:

1.  First, open the main configuration file of SSH, `/etc/ssh/sshd_config`, in any editor:

```
    sudo nano /etc/ssh/sshd_config
```

2.  Now look for the line that reads as follows:

```
    PermitRootLogin yes
```

3.  Change the value `yes` to `no`. Then save and close the file:

```
    PermitRootLogin no
```

![](img/881ddf81-a302-4956-aa44-3343a381d3f0.png)

4.  Once done, restart the SSH daemon service using the following command:

![](img/7d6bd4af-6b91-41a9-9a24-ba134444eb2e.png)

5.  Now let's try to log in as root. We should get an error:

```
"Permission Denied" 
```

This is because the root login has been disabled:

![](img/3e4859dd-52e4-45e5-81fa-3207d3086e27.png)

6.  Now whenever we want to log in as root, first we will have to log in as a normal user. And after that, we can use the `su` command and switch to the root user. So, the user accounts that are not listed in the `/etc/sudoers` file will not be able to switch to root user and the system will be more secure:

![](img/9825a30e-c951-4512-a736-5c8248a50691.png)

7.  Now if we want to enable SSH root login again, we just need to edit the `/etc/ssh/sshd_config` file again and change the `no` option to `yes`:

```
    PermitRootLogin yes
```

![](img/f8b5ab81-ce1b-4386-97e6-7f6ad3ddbd7b.png)

8.  Then restart the service again by using the following command:

![](img/1feee614-4e9c-450e-915a-5e0c65191744.png)

9.  Now if we try to log in as root again, it will work:

![](img/4e241d56-9841-4901-96ab-afb94784ba80.png)

# How it works...

When we try to connect to a remote system using SSH, the remote system checks its configuration file at `/etc/ssh/sshd_config` and, according to the details mentioned in this file, it decides whether the connection should be allowed or refused.

# There's more...

Suppose we have many user accounts on the systems. We need to edit the `/etc/ssh/sshd_config` file in such a way that remote access is allowed only for few mentioned users:

```
          sudo nano /etc/ssh/sshd_config
```

Add the following line:

```
          AllowUsers tajinder user1
```

Now restart the SSH service:

```
          sudo service ssh restart
```

Now when we try to log in with  `user1`, the login is successful. However, when we try to log in with `user2`, which has not been added in the `/etc/ssh/sshd_config` file, the login fails and we get  Permission denied error, as shown here:

![](img/645f50bd-8e03-474b-8e58-412c17107a6b.png)

# Key-based login into SSH for restricting remote access

Even though SSH login is protected by using passwords for the user account, we can make it more secure by using key-based authentication into SSH.

# Getting ready

To see how key-based authentication works, we need two Linux systems (in our example, both are Ubuntu systems). We should have the OpenSSH server package installed on them.

# How to do it...

To use key-based authentication, we need to create a pair of keys – a private key and a public key:

1.  On the client or local system,  execute the following command to generate the SSH key pairs:

```
    ssh-keygen -t rsa 
```

![](img/0b1e4303-998d-4fdb-adcb-55f4fd96c71c.png)

While creating the key, we can accept the default values or change them as we wish. It will also ask for a passphrase, for which you can choose anything or leave it blank.

2.  The key-pair will be created in the location - `~./ssh/`. Change to this directory and then use the  `ls -l` command to see the details of the key files:

![](img/5b50e052-01ea-4cee-91d5-37d752ebc309.png)

We can see that the `id_rsa` file can be read and written only by the owner. This permission ensures that the file is kept secure.

3.  Now we need to copy the public key file to the remote SSH server. To do so, we run the following command:

```
 ssh-copy-id 192.168.1.101  
```

![](img/1b169101-55dd-46e0-9625-614630024bc0.png)

An SSH session will be started and prompt for entering the password for the user account. Once the correct password has been entered, the key will be copied to the remote server.

4.  Once the public key has been successfully copied to the remote server, try to log in to the server again using the following command:

```
    ssh 192.168.1.101 
```

![](img/3e99f278-0f7f-490e-b86d-cb86a4ad32e0.png)

We can see that now we are not prompted for the user account's password. Since we had configured the passphrase for the SSH key, it has not been asked for. Otherwise, it would have asked us for the password.

# How it works...

When we create the SSH key pair and move the public key to the remote system, it works as an authentication method for connecting to the remote system. If the public key present in the remote system matches the public key generated by the local system, and the local system also has the private key to complete the key-pair, the login happens. Otherwise, if any key file is missing, login is not allowed.

# Copying files remotely

Managing a system remotely is great using SSH. However, many do not know that SSH can also help in uploading and downloading files remotely.

# Getting ready

To try the file transfer tools, we need two Linux systems that can ping each other. We also need the  OpenSSH package to be installed on one system and the SSH server should be running.

# How to do it...

Linux has a collection of tools that can help in transferring data between networked computers. We will see how a few of them work in this section:

1.  Suppose we have a `myfile.txt` file on the local system that we want to copy to the remote system. The command to do so as follows:

```
 scp myfile.txt tajinder@sshserver.com:~Desktop/  
```

![](img/fe8a6b50-c38b-4fc5-8a0b-c041088402d5.png)

Here, the remote location where the file will be copied is the `Desktop` directory of the user account being used to connect.

2.  When we check on the remote SSH system, we can see that the `myfile.txt` file has been copied successfully:

![](img/d3f27372-2e74-41f7-b032-339bb3bca82b.png)

3.  Now let's suppose we have a directory,  `mydata` in the local system, that we want to copy to the remote system. This can be done by using the  `-r`  option in the command:

```
    scp -r mydata/ tajinder@sshserver.com:~Desktop/ 
```

![](img/59cb1f9b-a48d-45d2-a9f0-dccb5faf4c78.png)

4.  Again, we check on the remote server and see that the `mydata` directory has been copied with all its files:

![](img/632e594f-e78c-4d04-9178-78073ce0ec30.png)

5.  Now we will see how to copy a file from the remote system back to the local system.

First, create a file on the remote server. Our file is `newfile.txt`:

![](img/30f93575-1d63-4da4-bdd1-927ef7691bcf.png)

6.  Now, on the local system, move to the directory where you wish to copy the file.

Then run the command as shown to copy the file from the remote system to the local system, in the current directory:

```
    scp -r tajinder@sshserver.com:/home/tajinder/Desktop/newfile.txt . 
```

![](img/4a077ce5-eefc-4bde-96a4-ac72fa1f0b85.png)

7.  You can also use `sftp` to interactively copy the files from the remote system, using `ftp` commands.
8.  To do this, you first start the connection using the following command:

```
    sftp tajinder@sshserver.com 
```

![](img/a7666d3b-5534-4ca1-91f6-32bcfe1b76ea.png)

9.  Next, you can run any `ftp` command. In our example, we try to get the file from the remote system using the `get `command, as shown:

```
    get sample.txt /home/tajinder/Desktop 
```

![](img/ce4a5bcc-8487-4693-b902-9007e972f84a.png)

10.  In the local system, you can now check whether the file has been copied successfully or not:

![](img/ebfa9908-0ad4-4d41-b650-bb42e949cc53.png)

11.  SSH also works through Nautilus (the default file manager for GNOME desktop). So, instead of using the command line, we can use the GNOME File Explorer to start an SSH connection with the remote system.
12.  In the GNOME File Explorer, go to File | Connect to Server.
13.  In the next window, enter the details as required and click on Connect:

![](img/30cd75b4-541f-498b-900f-de78e8089b4a.png)

14.  Now we can copy the files graphically from the remote system to the local system, or vice versa:

![](img/994edd3d-0432-424b-969d-d2c08d1a588c.png)

# How it works...

To copy files remotely over SSH, we use the `scp` tool. This helps copy a single file or a complete directory from the client system to a defined location on the server system. To copy a directory with all its content, we use the  `-r`  option with the command.

We use the same tool to copy files from the remote SSH server to the client machine. However, to do this we need to know the exact location of the file on the server.

Like `scp`, we also have the `sftp` tool, which is used to copy files over ftp from server to client. **Secure File Transfer Protocol** (**SFTP**) is better than FTP and ensures that data is transferred securely.

Lastly, we use the GNOME File Explorer to graphically connect and transfer files from server to client and vice versa.

# Setting up a Kerberos server with Ubuntu

Kerberos is an authentication protocol for allowing secure authentication over untrusted networks by using secret-key cryptography and trusted third parties.

# Getting started

To see Kerberos set up and running, we need three Linux systems (in our example we have used Ubuntu). They should be able to communicate with each other, and they should also have accurate system clocks.

We have set the hostname for each system is as follows:

*   Kerberos system – `mykerberos.com`
*   SSH Server system – `sshserver.com`
*   Client system – `sshclient.com`

After doing this, edit the `/etc/hosts` file in each system and add the following details:

![](img/91bc5316-eec3-47a8-a4b2-13ada980e1d3.png)

The IP address and the hostname can be different for your systems. Just make sure that after doing these changes they can still ping with each other.

# How to do it...

Now let's see how to do the setup of Kerberos server and the other systems for our example:

1.  The first step is to install the Kerberos server. To do this, we will run the following command on the `mykerberos.com` system:

```
 sudo apt-get install krb5-admin-server krb5-kdc 
```

![](img/e662fc60-e395-4d6f-b1c6-af53f6273a3a.png)

2.  During the installation process, a few questions will be asked. Enter the details as mentioned here.
3.  For the question `Default Kerberos version 5 realm`, the answer in our case is  `v=spf1 ip6:fd1d:f5c3:ee7c6::/48 -all`:

![](img/6571be59-6f32-4c0e-8196-caf6610bd827.png)

4.  For the next question, `Kerberos servers for your realm?`, the answer is `mykerberos.com`:

![](img/0726344d-a76f-4f79-aeb6-8cad71da60c4.png)

5.  In the next screen, the question is `Administrative server for your realm?` and its answer is  `mykerberos.com`:

![](img/00b34ddc-adf8-4241-a7c2-33d50ddb7847.png)

Once we have answered all the questions, the installation process will proceed.

6.  The next step is to create a new realm. To do so, we use the following command:

```
 sudo krb5_realm 
```

![](img/64669e6b-3ede-4015-9888-413e4ead24a6.png)

During this process, we will be asked to create a password for the Kerberos database. We can choose any password we want.

7.  Next, we need to edit the `/etc/krb5.conf` file and modify the details as shown here. If any line does not already exist in the file, we need to enter them as well. Go to the `libdefaults` section in the file and modify the value as shown:

![](img/80b7cd73-91fa-4b11-96be-d61c8ca59e45.png)

Move down to the `realms` section and modify the details as shown:

![](img/d0acc26c-9878-436b-81e4-03d0fca1037f.png)

Next, go to `domain_realm` section and enter the lines as shown:

```
 mykerberos.com = MYKERBEROS.COM .mykerberos.com = MYKERBEROS.COM
```

![](img/5edec1b8-1d09-450b-8464-183415a85e81.png)

8.  Next, we need to add principles or entries into the Kerberos database that would represent users or services on the network, and to do this we will use the `kadmin.local` tool. The principle must be defined for every user that participates in Kerberos authentication.

Run the tool by typing the following:

```
 sudo kadmin.local
```

This will start the `kadmin.local` prompt, as shown:

![](img/7ec9b300-674a-45c9-8e97-fdaf0214194f.png)

To see the existing principles, we can type the following command:

```
 list princs 
```

9.  Now to add a principle for a user we use the `addprinc` command, as shown:

To add the  `tajinder ` account we have used the following command:

![](img/d70103b1-dc10-42d3-846f-6a89d03382fd.png)

To add an admin role to the account being added, the command will be as follows:

![](img/354af7f6-1fe7-4d57-bfe4-09c5470f8d0e.png)

If we give an admin role to any user, then uncomment `*/admin " linein /etc/krb5kdc/kadm.acl file`.

10.  To check whether the principle has been applied correctly or not, use the following command:

```
 kinit 
```

11.  Once you're done with the setup of the Kerberos system, we now move to the client system. First, we need to install the client package for Kerberos by using the following command:

![](img/aa759eb3-cc58-4c92-ba40-ea4e356a58fd.png)

During the installation process, it will ask the same questions that were asked during the installation of the Kerberos server. Enter the same details here that we entered earlier.

12.  After completing the installation, check whether we are still able to ping mykerberos.com from the sshclient.com system.
13.  Now, to get the ticket for the client machine, depending on the principle that we had created in mykerberos.com, the command to be used will be as follows:

![](img/d1b81a08-b4eb-4e7c-bd7e-b36cd8e07aa2.png)

If the command runs perfectly, it means it's working properly.

14.  Once you're done with the previous command, we move to the third system, which we are using as SSH server. We need to install SSH server and the `krb5-config` package on this system. To do so, we run the command as shown:

![](img/667c069c-a23e-4eea-be64-87a52b14df1c.png)

Again, we will be asked the same questions that were asked during the installation of the Kerberos server. Enter the same details here as previously.

15.  Now edit the `file/etc/ssh/sshd_config` file to enable the following lines:

![](img/a0fc8df4-0fa7-4b62-a907-329cbace59c4.png)

Remove the `#` and also change the value to `yes` if it is not changed already.

After making the changes, restart the SSH server, using the following command:

```
 sudo service ssh restart 
```

16.  Next, we will configure the Kerberos server so that it works with the SSH server. To do so, run the `kadmin.local` tool and then run the commands shown:

```
 kadmin.local 
```

![](img/1f684d7f-cb56-49e7-abcc-c49d2aec9e3a.png)

The previous command in the screenshot adds the principle for the SSH server.

Next, we run the following command to create the key file:

![](img/e1e60422-bb8f-4c85-8a68-bb7f1394bc47.png)

17.  Now we shall copy the key file from the Kerberos server system to the SSH server system, using the following command:

![](img/3b9821ab-3ae7-4535-a675-d6dff63ab1b1.png)

We have copied the file to `/tmp/` directory of the SSH server system. Once the copy completes, move the file to the `/etc/` directory.

18.  Now on the client system, edit the file  `/etc/ssh/ssh_config`, and modify the lines as shown:

```
    GSSAPIAuthentication yes
    GSSAPIDelegateCredentialsyes
```

19.  Now on the client system, get the ticket by running the command:

```
 kinit tajinder 
```

20.  Once this command works fine, try to log in into the SSH server system from the client system, using `ssh`.

![](img/e788eb57-705a-431b-9ac7-6ce6fe116e4c.png)

We should get authenticated without being asked for the password.

# How it works...

First, we install the required packages on the first system to create a Kerberos server. After installation, a realm is created for the server configuration. To complete the configuration, we do the changes as mentioned in the `/etc/krb5.conf` file.

Then we add principle in the Kerberos database to add the user account to be used.

Once this is done, we move to the next system and install the Kerberos user package to create the client system. Then we get a ticket from the Kerberos server system for the user account to be used on the client.

Next, we proceed to the third system where we install the Openssh server package to create an SSH server. Then we edit the configuration file of SSH to enable authentication.

We now come back to the Kerberos server system and add a principle for the SSH server. We create a key for the SSH server and then transfer this key file from the Kerberos server to the SSH server using  the `scp`  command.

Now if we try to log in into the SSH server system from the client system, we get logged in without being asked for the password, as the key we generated earlier is being used for authentication.

# Using LDAP for user authentication and management

**Lightweight Directory Access Protocol** (**LDAP**) helps to keep authentication information in a centralized location. In this topic, we shall discuss the configuration of any client machine for remote authentication with the LDAP server.

# Getting started

To proceed with the configuration of the client machine, we need a Linux machine configured as a LDAP server. This has already been covered in [Chapter 3](dd9ad20f-3479-47df-8819-358a6d9354ca.xhtml), *Local Filesystem Security*.

After configuring the LDAP server, we have to add organizational units, groups and users. Once we log in to the LDAP server, we can use the left menu to create groups and users.

After completing the process, we should have an LDAP server set with a few users and groups.

# How to do it...

After completing the setup of LDAP server on Ubuntu, and creating a few users and groups, we shall now try to configure our client machines, to remotely authenticate with the server:

1.  The first step  is to install a few packages on the client machine so that authentication functions properly with the LDAP server. To install the packages, we run the following command:

```
apt-get install libpam-ldap nscd
```

![](img/e9a88ec1-7bc8-42cd-8bdc-854bba938aa8.png)

2.  During the installation, various questions will be asked, the same way as they were asked during the installation of the server components.
3.  The first information to be asked for will be the LDAP server's Uniform Resource Identifier, as shown here:

![](img/13178fdc-6312-428c-a6bd-2d75e918c1a5.png)

Change the string from `ldapi:///` to  `ldap://` and enter the server's information as entered here:

![](img/5d70778b-cb28-4102-9942-1a42bc07ba1d.png)

4.  Next enter the recognized name of the LDAP server, the same as the value entered in the LDAP server, when configuring the `/etc/phpldapadmin/config.php` file:

![](img/302e3f0b-dd05-49db-bd63-127cbae61ac9.png)

5.  Next, select the LDAP version to use as  `3`:

![](img/1dd5f534-de39-4ab4-aa32-2f8f029f3b4b.png)

6.  Next, select `Yes` to allow the LDAP admin account to behave as the local root:

![](img/13b7b284-468c-4a27-b79b-619501ec1e22.png)

7.  Then select `No` for `Does the LDAP database require login`:

![](img/aeb56e5e-7751-454f-98ba-c139a1ec6341.png)

8.  Next, enter the details of the LDAP account for the root as configured in the`/etc/phpldapadmin/config.php` of the LDAP server:

![](img/faffab56-c6f4-4e6b-94c8-8e88debbf933.png)

9.  Enter the LDAP root account password:

![](img/23482c0f-a779-4748-9621-642572af6852.png)

10.  Once all the questions have been answered, the installation of the packages will complete.
11.  Our next step will be to configure the client so that its authentication files know that they have to look to our LDAP server for the authentication information. To do  this, we edit the `/etc/nsswitch.conf` file and update the three lines with the  `passwd` , `group`, and `shadow` definitions, as shown here:

![](img/c64f298c-86c8-4d79-8d07-59d3ba0985e9.png)

12.  After this, we add a value in the PAM configuration file, `etc/pam.d/common-session`. PAM, or Pluggable Authentication Modules, help connect authentication, providing applications to the application requiring authentication.

13.  Edit the `/etc/pam.d/common-session` file and add a line at the bottom, as shown here:

```
 session required    pam_mkhomedir.so skel=/etc/skel umask=0022 
```

![](img/b03cedc6-a9ab-4d76-8713-d9809b42bc54.png)

14.  Now we restart the service for implementing the previous changes:

![](img/cab3516a-6717-4e17-ba36-ac39edc4a1ad.png)

15.  We are done with all the configurations on the client machine. Now we shall try to log in with our LDAP user. Use a new terminal window to SSH  into the client machine using the LDAP user's details:

![](img/b849bf7f-381b-4892-ad95-2d1d20f6b66b.png)

16.  We should log in successfully, just like a local user.