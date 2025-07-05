# Understanding Linux Service Security

In this chapter, we will discuss the following:

*   Web server – HTTPD
*   Remote service login – Telnet
*   Secure remote login – SSH
*   File transfer security – FTP
*   Securing mail transfer – SMTP

# Web server – HTTPD

HTTPD refers to the Apache2 web server, and is commonly used on Linux systems. Web servers commonly use the HTTP Protocol to transfer web pages. Apart from HTTP, protocols such as HTTPS and FTP are also supported.

# Getting ready

There are no specific requirements to configure Apache on a Linux system.

# How to do it...

In this section, we will see how to install and configure an Apache web server on an Ubuntu system:

1.  As Apache is available in Ubuntu's default software repositories, we can install it easily by using the `apt` installer. To install Apache and all its required dependencies, we run the following command:

![](img/17d182b9-50d0-4a71-9dc1-add8a4db3e65.png)

2.  During the installation process, Apache registers itself with Ubuntu's default firewall, UFW. This provides profiles that can be used to enable or disable access to Apache through the firewall. To list the profiles, type the following command:

![](img/00a26050-d721-4be9-b6ac-a0a0e80d0e36.png)

We can see three profiles are available for Apache. Apache refers to port `80` only, and Apache Full refers to both port `80` and `443`, whereas Apache Secure refers to only port `443`.

3.  As SSL is not configured, we will allow traffic on port `80` only for now. To do this, we run the following command:

![](img/91f6e0ce-d876-4b4a-bbf6-91eca48c49a4.png)

4.  Now, we can verify that access has been granted for HTTP traffic by checking the status of UFW, as shown here:

![](img/e68b0f68-40d0-4ef0-803e-21ed87532794.png)

5.  When the installation for Apache completes, Ubuntu automatically starts it. This can be confirmed by running the following command:

![](img/0393c8db-8b7f-4bff-95ce-8c0a26d197bf.png)

6.  Once Apache is up and running, we can enable additional modules to get extended features.
7.  To check the list of additional modules, see the `/etc/apache2/mods-available` directory.
8.  Suppose we want to install the MySQL Authentication module; this can be done by running the following command:

```
apt-get install libapache2-mod-auth-mysql
```

9.  Once installed, the module can be enabled by using the following command:

```
a2enmod auth_mysql
```

10.  Next, we need to restart Apache to make the changes effective:

```
systemctl restart apache2.service
```

11.  If we wish to encrypt the traffic sent and received by the Apache server, we can use the `mod_ssl` module. As this module is available in the `apache2-common` package, we can directly enable it by using the following command:

```
a2enmod ssl
```

12.  Once the SSL module has been enabled, manual configuration is needed to make SSL function properly. We have already discussed this in previous chapters.

# How it works...

Apache is the most commonly used web server on Ubuntu. We install it from Ubuntu's repository. Once installed, we allow access through the UFW firewall as per our requirements.

Once Apache is up and running, we can customize its configuration by installing additional modules and then enabling them.

# Remote service login – Telnet

**Telnet** is one the earliest remote login protocols still in use. It is older than most of the system administrators today, as it was developed in 1969\. Telnet allows users to make text-based connections between computers. Since Telnet provides no built-in security measures, it suffers from various security issues.

# Getting ready

To demonstrate the use of Telnet, we will use two systems. On the first system, a Telnet server will be running, and from the second system, we will check the security issues.

# How to do it...

In this section, we will see how Telnet can cause serious security issues:

1.  Using Telnet is very easy. Just open a terminal window and type the following command:

```
telnet <IP Address> <Port>
```

Here is an example: `telnet 192.168.43.100 23`

2.  When Telnet is running on a server, it can be used by attackers to perform the banner grabbing of other services. Let's use it to find the version of SSH running on the server. Type the following command to do this:

![](img/1da7e09a-7663-4bd9-a0f1-3964d3fb40b9.png)

As we can see from the previous screenshot, the version of SSH is clearly displayed.

3.  We can also perform SMTP banner grabbing through Telnet. For this, we run the following command:

![](img/12a4a2ed-40aa-445a-88b9-477c8fb95cdf.png)

In the previous output, we can see the remote server is using the PostFix mail server with a hostname of `metasploitable.localdomain`.

4.  Once the attacker has this information about the SMTP server, they can try guessing valid mail accounts. They use the `vrfy` command followed by a mail account to do this. Based on the result, they can understand whether the mail account is valid or not:

![](img/d8b4e8a2-0631-4549-b879-cd3f5ab7b646.png)

If the response code is `550`, it implies the guessed mail account is invalid, and when the response code is `250`, `251`, or `252`, it implies the mail account is valid.

5.  As Telnet by default does not encrypt any data being sent over the connection, attackers can easily eavesdrop on the communication and capture sensitive data, including passwords.
6.  Suppose a user is connecting to the Telnet server as shown here:

![](img/fd4105a5-f2c3-4006-b51b-8245b498252c.png)

They will be prompted for their login details. Once the correct details are entered, they get logged in as seen here:

![](img/f3ec1744-b6f7-4205-95f2-936969e1dc40.png)

7.  At the same time, the attacker, who is on the same network, captures the traffic by sniffing the network using the Wireshark tool. We can see here the Telnet traffic captured by Wireshark:

![](img/6c2c73ae-9a06-48cc-9558-7618d2dc7c69.png)

8.  When we analyze the captured packets, we can see the login details in clear text, as seen here:

![](img/06598d40-e817-4792-abf4-50967795cec5.png)

9.  This clearly tells us how insecure Telnet is.

# How it works...

Being one of the oldest protocols, Telnet has no inbuilt security measures. Attackers can use Telnet for banner grabbing.

Attackers can also sniff the traffic sent over a telnet connection and gain important information as telnet communicates in clear text.

# Secure remote login – SSH

As the internet grew, the security risks related to using SSH also became visible to users. To overcome these security risks, developers released a new tool called Secure Shell or SSH.

It provides the same functionality as Telnet, but in a secure encrypted tunnel.

# Getting ready

For the Linux systems, we can use OpenSSH for SSH connections. It is a free tool for Linux and can be installed using `apt`. We have discussed the installation and configuration of OpenSSH in previous chapters.

# How to do it...

In this section, we will see how using SSH instead of Telnet can secure our data:

1.  Once SSH is installed and configured, we can try to connect to the server using SSH as shown here. Enter the password when prompted:

![](img/447fe5b9-6790-4b28-801f-6c9c54e7369c.png)

2.  At the same time, if we try to capture the traffic using Wireshark, we get the following details:

![](img/a7bd929e-9532-418e-b30a-9a606b7a4ef7.png)

In the previous screenshot, we can see in the last lines that a Key Exchange was initiated between the client and server.

3.  When we go through the packets captured, we can see that Encrypted Packets are being exchanged between the client and the server shown as follows. This was not the case when using Telnet:

![](img/23f5a26d-c4f8-4d86-8a03-152af2c9d479.png)

4.  Even if we try to see the details of the packet, we can't see any clear text data. We get an encrypted output, as shown here:

![](img/c76a823a-2a7a-4a17-bb30-3644afcfbf38.png)

5.  So,we can see how SSH can help in securing data being communicated over the internet.

# File transfer security – FTP

File transfer security (FTP) has been the most common protocol for file transfers. When we talk about a file transfer protocol like FTP, it means the protocol is used to send streams of bits stored as a single unit in a particular filesystem. However, this process is not completely secure.

FTP has a lot of vulnerabilities and also it does not provide any encryption for data transfer.

Let's discuss a few security risks related to using FTP:

*   **FTP bounce attack**: When a file transfer happens using the FTP protocol, the source server sends the data to the client, and then the client transmits the data to the destination server. However, in the case of slow connections, users may use the FTP proxy and this makes the client transmit the data directly between the two servers.
*   In this kind of scenario, a hacker may use a `PORT` command to make a request to access ports by being the man-in-the-middle for that particular file transfer request. Then, the hacker can execute port scans on the host and gain access to the data transmitted over the network.
*   **FTP brute force attack**: A brute force attack can be tried against the FTP server to guess the password. Administrators tend to use weak passwords and also repeat the same password for multiple FTP servers. In such scenarios, if the attacker is able to perform a successful brute force attack, all the data will be exposed.
*   **Packet capture (or Sniffing)**: As FTP transfers data in clear text, any attacker can perform network packet sniffing to get access to sensitive information such as usernames and passwords.
*   **Spoof attack**: Suppose the administrator has restricted access to the FTP servers based on the network address. In such a scenario, an attacker may use an external computer and spoof its address to any computer on the enterprise network. Once this happens, the attacker will now have access to all the data being transferred.

We have seen different security risks associated to FTP. Now, let's discuss a few ways to perform secure data transfer:

*   **Disable standard FTP**: Many Linux servers have the standard FTP server preinstalled. As a best practice, it is recommended to disable the standard FTP as it lacks privacy and integrity, making it easy for an attacker to gain access to the data being transferred. Instead, use more secure alternatives such as SFTP.
*   **Use strong encryption and hashing**: When using SFTP or any other secure protocol, ensure that older and outdated ciphers such as Blowfish and DES are disabled, and only strong ciphers such as AES and TDES are being used.

FTP is a very old protocol, but still we can see the poor implementation of FTP in many networks. Being an old protocol, it has fewer security features, thus making it vulnerable to many security risks.

When we use secure protocols, such as SFTP, instead of FTP, we are less prone to attacks.

# Securing Mail Transfer – SMTP

**SMTP** or **Simple Mail Transfer Protocol** is used by email servers. Every email is sent and received by the SMTP server over the SMTP protocol. An MTA, or Mail Transfer Agent, like Postfix can be configured as an email server.

Postfix can be used on a Linux system to route and deliver emails.

# Getting ready

To install and configure Postfix, we will be using an Ubuntu server. We will also need a **Fully Qualified Domain Name** (**FQDN**) on our server.

# How to do it...

In this section, we will see how to install and configure Postfix on an Ubuntu sever and use as per our requirements:

1.  As Postfix is included in Ubuntu's default repositories, installing it becomes easy. To begin the installation, we will run the following command, along with the `DEBIAN_PRIORITY=low` environmental variable to answer some additional prompts during the installation:

![](img/c2f6420d-fae3-440a-8b05-f32337f59fbc.png)

2.  Once the installation starts, the first window will ask for the type of mail configuration. We will select `Internet Site` for our needs, as shown as follows:

![](img/08312560-3326-41de-a084-c025d4841083.png)

3.  In the next window, enter the hostname to be used for `System mail name`, as shown here:

![](img/57091be2-15f9-454c-a6b9-ff3841c6b562.png)

4.  Next, enter the Linux user account that will be used to forward the mails addressed to root and postmaster, shown as follows:

![](img/57a7d1a0-05d6-4223-828b-648becf442dd.png)

5.  The next window defines the mail destinations that will be accepted by Postfix. Confirm the existing entries and add any other domains if needed:

![](img/4d21b5b3-78a8-46ce-94b2-c60c96bdb6f2.png)

6.  In the next window, select `No` and proceed.
7.  The next window specifies the list of networks for which the mail server is configured to relay messages:

![](img/7eb6e91b-6d3c-44c8-86e0-6002047350b3.png)

8.  In the next window, we can limit the size of messages. We will set `0` to disable any size restrictions:

![](img/65aca31c-57a2-424d-90f9-1f3a403201dd.png)

9.  In the next step, choose which IP version Postfix should support. In our case, we will choose `all`:

![](img/d87adace-0cd3-4f8c-89bb-3c57b5bdf8ab.png)

10.  Once we are done with the previous steps, the setup will complete the installation, as shown here:

![](img/683f1fba-d7b9-40cd-b60f-bce89c5e759b.png)

11.  Now, we will begin to set the mailbox. For this, we will set the `home_mailbox` variable to `Maildir/`, as shown here:

![](img/ebcfa41a-3d57-46f8-bbe6-17db6628c719.png)

This step will create a directory structure within the user's home directory.

12.  Next, set the location of the `virtual_alias_maps` table. This table is used to map the Linux system accounts with the email accounts. We will run the following command to do this:

![](img/2696d9f8-d910-4045-8949-37bd9bb75d47.png)

13.  Now, let's edit `etc/postfix/virtual` to map the mail addresses to the Linux account, as shown here:

![](img/326732bb-763f-4a6c-af3e-df8d4e8052a2.png)

14.  Once done, apply the mapping by running the following command:

![](img/9a779521-f0de-4b58-be86-f7bdf17e7cd9.png)

15.  Now, we will restart the postfix service:

![](img/0ccff554-072f-4377-ba1d-09e3f0ccf2bf.png)

16.  Our next step will be to allow Postfix through the UFW firewall:

![](img/cd0d2b9a-4ec7-4688-98a1-2c9cbcf0cebc.png)

17.  Postfix should configured now to send mails. We can test this by sending a test mail from any user account to the root email account, as shown here:

![](img/9eb11974-af71-4dc3-9894-22b9c405ec2b.png)

18.  Next, we check the mails for the root account by typing `mail`. We will see a new mail waiting. When we press *Enter*, we can see the content of the mail, as shown here:

![](img/9127c4c6-544d-43cb-a63e-3575240bb042.png)

19.  Before finishing, we will perform Postfix hardening.
20.  We saw in the previous recipe, *Remote service login - Telnet*, how an attacker can use the `vrfy` command to guess email accounts, as seen here:

![](img/c2efad17-398f-46eb-86dd-54fa722f980d.png)

21.  To secure Postfix against this, we need to disable the `vrfy` command. To do this, we run the following command:

```
postconf -e disable_vrfy_command=yes
```

After this, we restart the Postfix service to make the changes effective.

22.  Now, if the attacker tries the same steps, they will get the output shown here:

![](img/f38dc635-bbfc-4321-b23a-8d4c57b29fb5.png)

# How it works...

Postfix is used to configure our SMTP server. During configuration, we create a mapping of email accounts to Linux system accounts.

To increase the security of Postfix, we disable the `vrfy` command, thus preventing attackers from guessing the email accounts configured on the server.