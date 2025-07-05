# Chapter 7. LDAP – A Better Type of User

Having a local user account and password defined on each server and workstation for your access is not really practical. Imagine trying to enforce password changes when you may have to implement the change on 10 or 12 systems that you may use. You may also wish to consider what happens when a user leaves; do you really think the account will be deleted from every system each time? The reality is that where multiple systems are placed, some form of directory solution must be in place; this may be means of OpenLDAP, or even Active Directory. Yes, CentOS can join a Windows domain. We will look at 389-ds, the CentOS implementation of the Red Hat Directory Server. 389-ds is based on OpenLDAP, but with some pretty smart management tools. We are going to cover the following topics in this chapter:

*   **LDAP concepts**: In this section, you will understand terms that are used when talking about directory services
*   **Installing 389-ds**: This section involves installing the 389 Directory Server ensuring all the plumbing is in place
*   **LDAP user account management**: This section includes the management life cycle of users in the directory
*   **LDAP authentication**: This section will cover authenticating a second CentOS server to the shared directory

# LDAP concepts

LDAP stands for Lightweight Directory Access Protocol; as the name suggests, it began as a client-server protocol used to access a directory, but there was so little development in directories that it soon took on the entire role of a directory server. If we break a directory down, the **Directory Access Protocol** (**DAP**) is just one small part of the many pieces of an LDAP server:

*   **Directory Information Database** (**DIB**): This is the database where the directory is stored
*   **Directory Information Tree** (**DIT**): This is the hierarchical organization that represents entries in the DIB, organizations, organizational units, and so on
*   **Directory System Protocol** (**DSP**): This represents the server to server communication
*   **Directory Access Protocol** (**DAP**): This represents the client-to-server communication
*   **Directory Server Agent** (**DSA**): This is the server software
*   **Directory User Agent** (**DUA**): This is the client software
*   **Directory Information Shadowing Protocol** (**DISP**): This is the replication of the directory
*   **Schema**: These are entry data definitions

Each of these elements is represented in some way in all LDAP directories and common directories, including OpenLDAP, Red Hat Directory Server, and 389-ds. We will be looking at the 389-ds, which is based on the Red Hat Directory Server, which in turn is an implementation of OpenLDAP with some enhanced features. Ultimately, directories are used in the creation of what has commonly been dubbed as identity management, the central storage of user accounts to lessen the burden on account management, and improve security. A user is much more likely to recall a strong password if he/she only remembers a single credential set.

# Installing 389-ds

In the following sections, we will see the steps involved in installing 389-ds.

## Configuring DNS or hostname records

When installing OpenLDAP or 389-DS, it is imperative that you can resolve the hostname of the system on which you install the directory. My system is named `ldap1.tup.com`, and I have a local DNS record for this but it can also be maintained by an entry in the local `/etc/hosts` file on the host system. I can verify the name is correct by using the `host` command or something similar:

```
$ host ldap1.tup.com

```

You should see the IP address being returned. You can see the result of this command when executed on my system in the following screenshot:

![Configuring DNS or hostname records](img/5920OS_07_01.jpg)

## Setting TCP keepalives

The default timeout of TCP connections is 120 minutes. We will configure it for five minutes. In doing so, we will reduce the overhead caused by dropped TCP connections; they will be closed much more quickly. Edit the `/etc/sysctl` file to include the following line:

```
net.ipv4.tcp_keepalive_time = 300
```

Load the settings using `sysctl` to make them current as well as the default for the next reboot:

```
# sysctl -p

```

## Setting file descriptors

Editing the number of file descriptors on the Linux system can help a directory server access files more efficiently, so we will start by looking at the current setting:

```
# cat /proc/sys/fs/file-max

```

If the value is less than 64,000, then increase the limit in the `/etc/sysctl` file:

```
fs.file-max = 64000

```

Run the following command to read the new value we have just set, saving a restart:

```
# sysctl -p

```

For this setting to be effective, we must also allow all users to be allowed to have enough open files. Edit the `/etc/security/limits.conf` file to include the following lines:

```
*   soft   nofiles   8192
*   hard   nofiles   8192
```

This allows all users (`*`) to have a maximum of 8,192 open files (`nofiles`). We set both limits to the same value, but the soft limit can be exceeded with just a warning, while the hard limit cannot be exceeded.

## Creating the directory server user and group

When configuring the directory, we will need to assign the service a user and group ID; the default is the `nobody` account, but we should create a non-privileged user and group dedicated to the directory as follows:

```
# useradd -m ldap389

```

The `useradd` command will create the `ldap389` user and group.

At this stage, it is pertinent to reboot the system to ensure that the settings are in place before we start the installation of the directory server. With this in place, we can further prepare for the installation.

## The EPEL repository

We will need to implement the EPEL repository. Using the `wget` command, you can download the RPM file that will configure the repository for you:

```
$ wget http://epel.mirror.net.in/epel/6/i386/epel-release-6-8.noarch.rpm

```

Once you have downloaded, install the RPM as root. This will create the repository file for you. The output of the `yum repolist` command should show the EPEL repository. The following screenshot is from my demonstration system:

![The EPEL repository](img/5920OS_07_02.jpg)

## Installing and configuring 389-ds

With the repository configured on our system, we are now ready to use `yum` to install the directory server:

```
# yum install -y 389-ds openldap-clients

```

Among other things, you will find that Java will be installed as there is a Java management console that simplifies connecting to and managing entries in the directory.

The configuration is made simple with the implementation of a script that will guide us through the configuration of the administration and directory services on CentOS. The administration service represents the LDAP server; the directory service itself represents an individual directory hosted on the administration server. A single administration server can look after directories for company A and company B. The two directories can be administered separately.

The setup is achieved by running the script, `setup-ds-admin.pl`. The script is installed in your PATH statement. The default is for the script to run interactively but, especially where servers are setup frequently, you may use an answer file to accompany the script.

Running the following command will configure the directory and administration services interactively:

```
# setup-ds-admin.pl

```

This command, when run, will lead you into a menu, which is explained as follows:

1.  **Continue with setup**: Choose `yes` to continue with the script at the first prompt.
2.  **dsktune**: The tuning analysis will be run to check that you have RAM greater than 1024 MB and that the other preflight checks are in place.
3.  **Choose a setup type**:

    1.  **Express**
    2.  **Typical**
    3.  **Custom**

    We will choose `2` for **Typical**.

4.  **Set the computer name**: Next, we set the computer name. This should be default to your hostname; in my case `ldap1.tup.com`.
5.  **Set the user which the directory server will run**: This we set to `ldap389` in our case, the dedicated user we created.
6.  **Set the group which the directory server will run**: We will use the `ldap389` group.
7.  **Register the new directory server with an existing server**: We will choose `No` as this is the first server. If another server pre-existed, we could register this installation with that server.
8.  **Give the configuration directory server a user ID and password**: We can accept the preset name of the admin. We use this account to log in to the directory server.
9.  **The configuration directory server domain**: The domain I am using is `tup.com`. We will create an LDAP domain to reflect this.
10.  **Set the directory server port**: The default port for LDAP is `389`, and we will keep this as the default.
11.  **Set the directory server identifier**: We will then be prompted for a unique name for the server in the directory. This will default to the first part of the hostname, in my case, `ldap1`.
12.  **Set the directory suffix**: LDAP names are comma separated. The LDAP suffix will now be displayed similar to the following and will make up the top container in the LDAP tree:

    *   `dc=tup, dc=com`

13.  **Set the directory manager ID**: This is the LDAP directory manager and has a default name of `cn=Directory Manager`. We will need to set a secure password for this.
14.  **Set the administration server port**: The administration port can be used to manage the server through the Java console. We will leave the default port at `9830`.

The interactive setup is now complete, and the configuration will be created along with the default entries. The service should start in the final stage and a success message will be shown. We should ensure that the services start correctly on boot using `chkconfig`:

```
# chkconfig dirsrv on
# chkconfig dirsrv-admin on

```

### Tip

Remember that `dirsrv` runs on port `389` and is the LDAP directory. The `dirsrv-admin` is the administration server that listens on port `9830`

## Testing the installation

Congratulations! You now have an LDAP server, and we can test the configuration in a couple of ways. Firstly, we can use the `ldapsearch` command to display entries from the directory, and then we will use the 389-console, the GUI tool that we can manage LDAP with. In the following command, the `-x` option is for simple authentication and the `-b` option is the search base:

```
$ ldapsearch -x -b dc=tup,dc=com

```

The output from this will list, in **Lightweight Directory Interchange Format** (**LDIF**), some containers and groups that were created during the configuration. The following screenshot shows one such group from the output on my system:

![Testing the installation](img/5920OS_07_03.jpg)

The big advantage that we have with the 389-ds is the GUI. We can run it from the server or we could install the console on a client machine. We will run it from the server using the following command:

```
$ 389-console -a http://ldap1.tup.com:9830

```

We will be presented with a graphical console, as shown in the following screenshot, in which we can log in as an admin:

![Testing the installation](img/5920OS_07_04.jpg)

# LDAP user account management

The purpose of the directory is to house user accounts. These accounts do not have to be used for authentication but we can use them for authentication. This may be for other CentOS or Linux systems as well as services such as Apache. We will now look at creating user accounts in the directory both from the command line and from the GUI console. If we start with the GUI console, we can create and export the user and reimport it from the command line.

## Adding users using the GUI console

We can log in to the console using the admin account as before. From the main welcome page, we should choose the **Users and Groups** tab and then, select the **Search** button. With nothing in the search dialog, the search will return all users, groups and containers. The container objects that we can see are organizational units; an **organizational unit** (**OU**) is a little like a folder within a filesystem. In the directory, OUs are used to organize objects. Navigating to the OU named "people", we can use the menu to **Create** | **User**, (found at the bottom of the screen). This will open a form for use to complete the user's details. On the user form, we will add the following data:

| **First Name** | `Bob` |
| **Last Name** | `Bloggs` |
| **Common Name** | `Bob Bloggs` |
| **User ID** | `bbloggs` (it is suggested to keep it all lowercase) |
| **Password** | `Password1` |
| **Confirm Password** | `Password1` |

If we only needed a user to support an application login, such as from an Apache web server, this is all we would need. As we ultimately would like to log in from Linux, we will also complete the **Posix User** form as follows:

*   **Enable Posix User Attributes**: `Selected`
*   **UID Number**: `5000`
*   **GID Number**: `100`
*   **Home Directory**: `/home/bbloggs`
*   **Login Shell**: `/bin/bash`

These attributes then combine to create our user. This will be displayed in the 389-console as well as in `ldapsearch`. If we rerun the earlier search, this time, we will add a filter to display only those objects that match `posixAccount`:

```
$ ldapsearch -x -b dc=tup,dc=com objectClass=posixAccount

```

This should just show a single entry.

## Adding users from the command line

To create users from the command line, we have a similar tool to `ldapsearch`. The `ldapadd` command will create users in the directory. The user account details will be defined within an LDIF file to use as an argument to the `ldapadd` command. We will additionally need to be authenticated to perform this operation as anonymous connections, because as you will imagine, it cannot create entries within the directory. We will begin by ensuring the authentication is correct by using `ldapsearch`; this will also allow us to view all attributes for the new user `bbloggs`. We will use this output, once edited, to create a new user when imported with `ldapadd`

```
$ ldapsearch -x -D "cn=directory manager" -w Password1 -b dc=tup,dc=com objectClass=posixAccount > /tmp/user.ldif

```

Using this search, we now authenticate as the directory manager and redirect the output through to the `/tmp/user.ldif` file. Using the `-W` option in place of `-w` will allow the password to be requested during the operation rather than being placed on the command line.

We should edit the file using your preferred text editor so that it looks similar to the following:

```
dn: uid=ssmith,ou=People,dc=tup,dc=com
givenName: Sally
sn: Smith
loginShell: /bin/bash
uidNumber: 5001
gidNumber: 100
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: inetorgperson
objectClass: posixAccount
uid: ssmith
cn: Sally Smith
homeDirectory: /home/ssmith
userPassword: Password1
```

### Tip

We are setting the `uidNumber` manually, and it must be incremented along with changing the other name-based fields.

Once the file is edited, we can create the user from the command line using the `ldapadd` command:

```
$ ldapadd -x -D "cn=directory manager" -w Password1 -f /tmp/user.ldif

```

If you typed correctly, then you should see a success message similar to the following screenshot taken from my system:

![Adding users from the command line](img/5920OS_07_05.jpg)

The password for the new user will be stored in an encrypted form, although we add it in clear text from the LDIF file.

We have now seen how we can use the standard OpenLDAP tool to search and add entries to our directory as well as the comfort of being able to use the GUI tools if we prefer. We have two users in the system now that can be provisioned to client Linux systems.

# LDAP authentication

We will use an additional CentOS 6.5 server on which we will configure the OpenLDAP client for authentication so that we make use of the central account database that we established on the 389-ds server.

From the client machine, we will need to install the following packages:

*   `openldap`
*   `openldap-client`
*   `nss-pam-ldapd`

This will be managed through the standard `yum` repositories:

```
# yum install openldap openldap-clients  nss-pam-ldapd

```

Once this is installed, we will make one change to the `/etc/sysconfig/authconfig` file. We will edit the line that reads `FORCELEGACY=no` to `read FORCELEGACY=yes`. This change will allow us to use LDAP rather than LDAPS. Although it would be more secure to use LDAPS as the information encrypts data exchange, using LDAP, we alleviate the need to create certificates for the server, which is adequate on a local network.

To configure the authentication, we can use the `authconfig` command:

```
# authconfig --enableldap --enableldapauth --enablemkhomedir \
--enablemkhomedir --ldapserver=ldap://192.168.0.76:389/ \ 
--ldapbasedn="dc=tup,dc=com" \
--enablecache -- disablefingerprint --update

```

This configures authentication and will write the configuration to the correct files. You will notice that we include the option to create user home directories, `--enablemkhomedir`; on login, if a user's home directory does not exist, it will be created. We added the home directory path when creating the users, but this will not have created the home directory. These home directories are not shared unless we remap the `/home` directory on client machines to a central location.

We can verify the configuration now using the `getent` command we touched upon in the previous chapter:

```
# getent passwd

```

This should list the `bbloggs` and `ssmith` account from the central LDAP server. This can be seen in the output on my system:

![LDAP authentication](img/5920OS_07_06.jpg)

We can now log out and log in again as one of our users. Choosing the `ssmith` account, we log in to the graphical gnome desktop, navigate to the **System** | **Preferences**, and select the **About Me** button. We can see that we are logged in as **Sally**. The output from my system is shown in the following screenshot:

![LDAP authentication](img/5920OS_07_07.jpg)

# Summary

In this chapter, we introduced you to the concept of identity management and showed the use of the 389-ds, the enriched OpenLDAP server on CentOS. It is enriched with a simplified setup script and graphical tools; however, we also saw how we can use traditional OpenLDAP tools to create and search entries in a directory. We finished the chapter by allowing a second CentOS server to use the account database shared by 389-ds supplying us with a single logon across many systems.

In the next chapter, we will discuss the Nginx web server, the new kid on the block, but a refreshing alternative to Apache.