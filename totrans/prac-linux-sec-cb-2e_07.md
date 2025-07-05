# Security Tools

In this chapter, we will discuss the following:

*   Linux sXID
*   Port Sentry
*   Using Squid Proxy
*   Open SSL Server
*   Trip Wire
*   Shorewall
*   OSSEC
*   Snort
*   Rsync and Grsync—backup tool

# Linux sXID

In Linux, normally a file has permissions to read, write, and execute. Apart from these permissions, it can also have special permissions such as SUID (Set owner User ID) and SGID. Due to these permissions, it is possible for a user to log in from their account and still run a particular file/program with the permissions of the actual file owner (which can be root also). sXid is the tool for monitoring SUID/SGID on a regular basis. Using this tool, we can track changes in the SUID/SGID of files and folders.

# Getting ready

To use this tool, we need to install the sXid package on our Linux system. We can either use the `apt-get` command to install the package, or we can download the package and manually configure and install it.To install the sXid package, we run the following command:

```
    apt-get install sxid
```

![](img/e1969764-cfc4-4196-9d5b-21fbc4e85c7b.png)

# How to do it...

To start monitoring the `suid/sgid` of files and folders, we configure the tool as follows:

1.  Once the installation completes, we start editing the `/etc/sxid.conf` file to use the tool as we require. Open the file in the editor of your choice:

```
    nano /etc/sxid.conf
```

2.  In the configuration file, look for the following line:

![](img/af76b953-182b-4376-99ab-629632580aa5.png)

Change the value for `EMAIL` to any other email ID, if you wish to have the output of changes whenever `sxid` is run sent to your email ID.

3.  Next, look for the line that reads `KEEP_LOGS` and change the value to a numerical value of your choice. This number defines how many log files to keep:

![](img/a2e6ae13-82da-40ad-b38a-1d0a0a2f89ee.png)

4.  If you wish to get the logs even when sXid finds no changes, then change the value for `ALWAYS_NOTIFY` to yes:

![](img/2e7653e9-2a1c-4cb1-98da-8222c23cf123.png)

5.  We can define a list of directories, separated with spaces, for the `SEARCH `option, for sXID to use as a starting point for its search. However, if we wish to exclude any directory from the search, we can specify it under the `EXCLUDE `option:

![](img/bdbbb258-53ed-40cf-9c05-1ba185e8fae1.png)

Suppose we have a directory, `/usr/local/share`, to be searched, and the `/usr/local` directory has been mentioned in the exclude list; it will still be searched. This becomes useful for excluding a main directory, and only specifying one.

7.  There are many more options in `/etc/sxid.conf`, which can be configured as per our requirements. Once we are done with editing the file, save and close the file.

8.  Now, if we want to run sxid manually for spot-checking, we use the following command:

```
    sxid -c /etc/sxid.conf -k
```

![](img/ecb966fd-fbcb-43a0-bfb2-981df97b0c63.png)

Here, the `-c` option helps to define the path of the config file, if it is not automatically picked up by the command. The `-k` option runs the tool.

# How it works...

We first install the sxid package and then we configure it by editing the `/etc/sxid.conf` file as per our requirements. Once the configuration has been done, we run sXid manually to perform spot-checking. We can even add an entry in `crontab` to run sXid automatically at a defined interval, if we wish to.

# Port Sentry

As a system administrator, one major concern would be to protect the system from network intrusions. This is where PortSentry comes into the picture. It has the ability to detect scans on a host system, and react to those scans in a way we choose.

# Getting ready

To demonstrate the implementation and use of PortSentry, we need two systems on the same network, which can ping each other. Also, we need the Nmap package on one system, which will be used as a client, and on the other system, we will install and configure the PortSentry package. To install the `nmap` package, use the `apt-get install nmap` command:

![](img/40ece4d2-39f1-4a42-b113-7f6d779e8fa2.png)

# How to do it...

1.  On the first system, we install the PortSentry package, using the following command:

```
    apt-get install portsentry
```

![](img/31d61286-3436-41d0-9181-cafea6ad4e79.png)

2.  During the installation process, a window will open containing some information about PortSentry. Just click `Ok `to continue.

3.  As soon as the installation completes, PortSentry starts monitoring on TCP and UDP ports. We can verify this by checking the `/var/log/syslog` file by using the following command:

```
    grep portsentry /var/log/syslog
```

![](img/d4b379fd-46c9-429b-a30c-a2227b4be695.png)

We can see messages related to `portsentry` in the log.

4.  Now on the second machine, which we are using as a client, run the `nmap` command as shown here:

![](img/b42e7bbf-9164-4d06-a8ae-65bf56456fe1.png)

We can also use any other `nmap` command to perform a TCP or UDP scan on the first system, which has `portsentry` running. To check Nmap commands, see [Chapter 1](f7bbe70d-3eb5-4946-b10a-46b36214b311.xhtml), *Linux Security Problem.* In the previous result, we can see that nmap is able to scan successfully even when PortSentry is running on the first system. We can even try to ping the server system from the client to see if it is working after installing Portsentry.

5.  Now, let's configure PortSentry by editing the `/etc/portsentry/portsentry.conf` file on the server system. After opening it in the editor of your choice, look for the lines shown here and change the value to `1`:

![](img/ea0d1ebc-1d88-4b6f-ba9f-c721920ddcf9.png)

6.  Scroll down and then find and uncomment this line:

![](img/e671fb24-32cc-4a46-b6f0-53715d5514a2.png)

7.  Next, uncomment the following line:

![](img/23ce2619-9031-48e5-84f0-ca1337cf0a47.png)

Once done, save and close the file.

8.  Next, edit the `/etc/default/portsentry` file:

![](img/c17a2340-2df6-47ff-9bde-d68d54b95535.png)

In the lines shown here, we need to mention for which protocol Portsentry should be working, TCP or ATCP.

9.  Now, edit the `/etc/portsentry/portsentry.ignore.static` file and add a line at the bottom, as shown here:

![](img/2affcb21-9bbb-448d-989b-17a4b96657c3.png)

Here, `192.168.1.104` is the IP address of the client machine that we are trying to block.

10.  Now, restart the Portsentry service by running this command:

![](img/b2a2faba-ed05-4de2-b5c6-172e73be3329.png)

11.  Once the previous steps are complete, we will again try to run `nmap` on the client machine and see if it still works properly:

![](img/a63c4e15-2e95-4680-9d03-b9ff100c767d.png)

We can see that nmap is now not able to scan the IP address.

12.  If we try to ping the server from the client, even that does not work:

![](img/31e73368-6515-485a-b2e6-9e0248fc04d7.png)

13.  If we check the `/etc/hosts.deny` file, we shall see the following line has automatically been added:

![](img/1f6d2cf3-fa4c-4dd4-8b1d-d08243d27721.png)

14.  Similarly, when we check the `/var/lib/portsentry/portsentry.history` file, we get the a result similar to the last line in this screenshot:

![](img/9c1e42cf-2c1c-4c7c-8112-dec4d3542e9d.png)

# How it works...

We use two systems. The first system acts as a Portsentry server while the other as a client. On the first system, we install the Portsentry package and on the second system we install nmap, which will be used to demonstrate the workings of Portsentry. Now we perform an Nmap scan from the client machine on the server. We can see that it works fine. After this, we configure Portsentry as per our requirements by editing various files. Once editing is complete, restart the portsentry service and then again try to perform an Nmap scan from the client on the server. We see that now the scan does not work properly.

# Using Squid proxy

Squid is a web proxy application with a variety of configurations and uses. Squid has a large number of access controls and supports different protocols, such as HTTP, HTTPS, FTP, and SSL. In this section, we will see how to use Squid as an HTTP proxy.

# Getting ready

To install and use Squid on a particular system and network, ensure that the system has enough physical memory, because Squid also works as a cache proxy server, and thus needs space for maintaining the cache. We are using an Ubuntu system for our example and Squid is available in Ubuntu repositories. So, we need to ensure that our system is up to date. To do this, we run this command:

```
    apt-get update
```

And then we run this command:

```
    apt-get upgrade
```

# How to do it...

To install and configure Squid on our system, we have to follow these steps:

1.  The first step is to install the Squid package using the following command:

![](img/8949a700-2ace-4b66-9c26-6c3a332a9b25.png)

2.  As soon as Squid is installed, it will start running with a default configuration, which is defined to block all the HTTP/HTTPS traffic on the network. To check this, we just need to configure the browser on any system on the network to use the IP address of the Proxy system, as shown here:

![](img/cdf910a6-e11d-4918-bd21-365af0fde6af.png)

3.  Once done, we can now try to access any website and we shall see an error screen, as shown in the following screenshot:

![](img/763ad532-8981-4468-9aaf-e5ade4633691.png)

4.  Now, we shall start configuring our proxy server to get it to work as per our requirements. For this, we will edit the `/etc/squid3/squid.conf` file in the editor of our choice. Once the file is open in the editor, search for the category that reads `TAG: visible_hostname`. Under this category, add a line, `visible_hostname ourProxyServer`:

![](img/7fd91c34-78bc-4488-bcc5-21f1591cff33.png)

Here, `ourProxyServer` is the name we have given to our Proxy server.

5.  Next, search for the category that reads `TAG: cache_mgr` and add a line, `cache_mgr email@yourdomainname`. Here, mention the email ID of the administrator who can be contacted instead of `email@yourdomainname`:

![](img/5f6f2e93-4386-497e-b268-c34557217e7f.png)

6.  Next, we search for the line that reads as shown in the following screenshot. The `http_port` variable defines the port on which Squid would listen. The default port is `3128`, but we can change it to any other port that is not being used. We can even define more than one port for Squid to listen to, as shown here:

![](img/61ae9caf-e434-4af2-8a46-8f24967480cf.png)

7.  Now, we need to add a rule for allowing traffic on the network computers as per our requirements. For this, we will search for the line that reads `acl localnet src 10.0.0.8`. Here, we add our rule, `acl localnetwork src 192.168.1.0/24`, as shown in the following screenshot:

![](img/1903c82f-a318-4831-9c76-7c6fd1214851.png)

In the previous rule that we added, `acl `is used to define a new rule and `localnetwork` is the name we have given to our rule. `src` defines the source of the traffic that will come to the Proxy server. We define the network IP address with the subnet in bits as shown here. We can add as many rules as we wish to, according to our requirements.

8.  Next, search for the line that reads `http_access allow localhost` and, below this, add the line `http_access allow localnetwork` to start using the rule we added in the previous step and allow traffic:

![](img/6bcc9ff1-1782-4c71-bf27-97724c92feb3.png)

9.  Once we are done with the previous configuration steps, we restart the Squid service using this command:

```
    service squid3 restart
```

10.  Now, our Squid proxy server is running. To check, we can try to access the IP address of the Proxy server from the browser of any system on the network:

![](img/e8e0fca8-51f1-46ad-b3ef-0804848fb68e.png)

The previous error screen tells us that the Squid proxy is working fine. Now, we can try to visit any other website and it should open as per the rule we have added in the configuration file of Squid.

# How it works...

We start with installing the Squid package. Once the package is installed, we edit its configuration file, `/etc/squid3/squid.conf`, and add the hostname, email id of the administrator, and port on which Squid should listen. Then, we create the rule to allow traffic for all systems on the same network. Once we save all the configurations, we restart the Squid service and our proxy server is now working.

# Open SSL server

SSL (Secure Sockets Layer) is a protocol used for transmitting sensitive information over the internet. This could include information such as account passwords and credit card details. SSL is most commonly used in conjunction with web browsing over the HTTP protocol. The OpenSSL library provides an implementation of the SSL and TLS (Transport Layer Security) protocols.

# Getting ready

To demonstrate the use of OpenSSL, we need two systems. One will be used as a server on which we shall install the OpenSSL package, and also Apache. The second system will be used as a client. To install Apache, we run the following command:

![](img/cd804bf7-4edd-4ae3-a6a2-c3f38832a6a8.png)

# How to do it...

We will now see how to create a self-signed certificate using OpenSSL for Apache. This will help encrypt traffic to the server:

1.  We start with installing the OpenSSL package on the first system using the following command:

![](img/6f757658-ac54-4f17-8907-cf00f96c2a9c.png)

2.  Once OpenSSL is installed, we need to enable SSL support, which comes as standard in the Apache package for Ubuntu. To do this, we run this command:

![](img/ce439d3f-c7b3-4d3c-84df-1e3a97583259.png)

After enabling SSL support, restart Apache using this command:

```
    service apache2 restart
```

3.  Now, create a directory inside Apache's configuration directory. This is the place where we shall keep the certificate files that we will be making in the next step:

```
    mkdir /etc/apache2/ssl
```

4.  Now, we will create the key and the certificate using the following command:

![](img/7f00292e-1745-43fb-a752-b3da2342a9cb.png)

In the previous command, `req -x509` specifies that we will be creating a self-signed certificate that will adhere to X.509 CSR (Certificate Signing Request) management. `-nodes` specifies that the key file will be created without being protected with any password. `-days 365` tells us that the certificate being created will be valid for one year. `-newkeyrsa:2048` tells us that the private key file and the certificate file will both be created at the same time, and the key generated will be 2048 bits long. The next parameter, `-keyout`, specifies the name for the private key that will be created. And the `-out` parameter mentions the name of the certificate file being created.

5.  When the key and certificate files are being created, you will be asked a few questions. Provide the details of your configuration. However, the option that reads `Common Name (e.g. server FQDN or YOUR name)` is important and we have to provide either the domain name or the server's public IP.

6.  Next, we need to edit the `/etc/apache2/sites-available/default` file to configure Apache to use the key file and the certificate file created in the previous steps. Find and edit the lines shown here. For `ServerName`, we have provided the IP address of the Apache server system:

![](img/2df135ef-08d3-44cb-8452-4c1139d16fba.png)

7.  In the same file, scroll to the end of the file, and before the `<VirtualHost>` block closes, add the lines given here. Mention the key file name and certificate file name that were used while creating these files:

![](img/6c6ff34a-8dd6-4db9-9adc-1134b2c5e68b.png)

8.  Now, on the client system, open any browser and visit the Apache server's public IP using the `https:// protocol`, as shown here:

![](img/af1df6b0-fe0f-4b9a-84ae-8cc2794236f9.png)

The browser will show a warning message regarding the connection not being secure, because the certificate is not signed by any trusted authorities.

9.  Click on `I Understand the Risks` and then click on the `Add Exception` button to add the certificate in the browser:

![](img/2b00b062-5bab-4774-9fd5-29e113630c85.png)

10.  The next windows will show some information about the server. To proceed further and add the certificate, click on `Confirm Security Exception`:

![](img/b16f9aab-8fb9-498a-8dd7-0440279f396c.png)

11.  If you wish to check out more details about the certificate, click on `View` in the previous screen and you will get a new window showing the complete details of the certificate.
12.  Once the certificate has been added successfully, web page loading will complete, as shown here:

![](img/40f8fda4-6fb9-4210-9ecd-c99dcee8e18f.png)

# How it works...

We use two systems in this setup. The first is the Apache server on which we install the OpenSSL package. The second system works as a client, from which we will try to connect to the Apache web server. After installing Apache and the OpenSSL package on the first system, we enable SSL support for Apache. Then, we create the server key and server certificate file using the OpenSSL tool and a few arguments. After this, we edit the `/etc/apache2/sites-available/default` file so that Apache can use the key and certificate that we have created. Once done, we try to access the Apache web server through the browser on the client machine. We see that it asks for the new certificate to be added to the browser and, after doing this, we are able to visit the web browser using the HTTPS protocol.

# There's more...

We have seen how OpenSSL can be used to create self-signed certificates. Apart from creating self-signed certificates, there are various other use cases for OpenSSL. Here, we will see a few of them:

1.  If we want to create a new **Certificate Signing Request** (**CSR**) and a new private key, we can do so by using the command shown here:

![](img/0a55a3a9-baec-4c76-85fd-531b91f8f89e.png)

2.  During the process, it will ask for a few details. Enter the details as shown here:

![](img/e1b99a72-b2e2-4adc-b3f3-52c01091bef3.png)

3.  We can see the two files created in the present directory:

![](img/cd22168d-8916-4b7a-9190-b0ffac6960bc.png)

4.  If we want to check the CSR before getting it signed by the CA, we can do so as shown here:

![](img/315a3f9c-0ac3-4218-ae3d-283d7ddb2ac7.png)

Likewise, there are other commands that can be used with OpenSSL.

# Tripwire

With the increasing numbers of attacks on servers nowadays, administering the server while ensuring security is becoming a complex problem. To be sure that every attack has been effectively blocked is difficult to know. Tripwire is a host-based Intrusion **Detection System** (**IDS**), which can be used to monitor different filesystem data points and then alert us if any file is modified or changed.

# Getting ready

We only need to install the Tripwire package on our Linux system to configure our IDS. In the next section, we will see how to install and configure the tool.

# How to do it...

We will discuss how to install and configure Tripwire on our Ubuntu system in the following steps:

1.  The first step will be to install the Tripwire package using `apt-get`, as shown here:

![](img/ccdfe7e0-4889-4d15-a2dc-8286296ea12a.png)

2.  During the installation process, it will show an information window. Click OK to continue.
3.  In the next window, select Internet Site for type of mail configuration and click Ok:

![](img/2e4c4a18-9095-4f70-a9b0-7de25f490228.png)

4.  In the next window, it will ask for `system mail name`. Enter the domain name of the system on which you are configuring Tripwire:

![](img/cc555bc2-8664-4f68-9485-540c2918a7a1.png)

5.  Press Ok on the next screen to continue.
6.  Now, we will be asked if we want to create a passphrase for Tripwire. Select Yes and continue.
7.  Now, we will be asked if we want to rebuild the configuration file. Select Yes and continue:

![](img/9eb1b6a5-30ac-43aa-8c8e-427886f7b58a.png)

8.  Next, select Yes to rebuild the policy file of Tripwire:

![](img/316fb681-17cf-4412-9a02-ed287d043bf0.png)

9.  Next, provide the passphrase you wish to configure for Tripwire:

![](img/f5a97217-907f-4445-8c89-891802bede57.png)

It will also ask you to re-confirm the passphrase in the next screen.

10.  Next, provide a passphrase for the local key and also re-confirm it in the next screen:

![](img/fae8d505-e8a8-4f14-b754-2ff9fabc7ec5.png)

11.  The next screen confirms that the installation process has completed successfully. Click Ok to complete the installation:

![](img/6e5de291-3b5d-43a3-998f-b8d1e7d48412.png)

12.  Once the installation has been completed successfully, our next step would be to initialize the Tripwire database. To do so, we run the command shown here:

![](img/3678c8b5-8bd1-4a58-bcb9-5562214cebc0.png)

In the output shown here, we can see that an error called `No such file or directory` is displayed for many filenames. This happens because Tripwire scans for every file mentioned in its configuration file, whether it exists on the system or not.

13.  If we wish to remove the error shown previously, we have to edit the `/etc/tripwire/tw.pol` file and comment the lines for the file/directory that is not present in our system. We can even leave it as it is if we wish to, as it does not hamper Tripwire.

14.  In case we get any error related to "Segmentation fault", we may have to edit /etc/tripwire/twpol.txt file to disable the devices/files for which the error appears, as shown below -

![](img/ba1435ce-82be-4045-b3d9-b8216bc60a9e.jpg)

13.  We shall now test how Tripwire is working. To do so, we will create a new file by running this command:

```
    touch tripwire_testing
```

You can choose any name for the file.

15.  Now, run the Tripwire interactive command to test it's working. To do so, the command is as follows:

```
    tripwire --check --interactive
```

![](img/943a76c0-a64e-4bd9-ac21-29c5b082a1fe.png)

We will get the output shown previously. Tripwire checks all the files/directories and if there are any modifications, it will be shown in the result:

![](img/1902779c-25bb-45ee-8bb4-2d17d2c0f3e0.png)

In our case, it displays the line shown previously, which tells us that a file, `tripwire_testing`, has been added in the `/root` directory. If we wish to keep the changes shown, just save the resulting file that was automatically opened in your editor. While saving the result, you will be prompted for the local passphrase. Enter the passphrase that you configured during the installation of Tripwire.

16.  Finally, we add an entry in `crontab` to run Tripwire automatically to check for the changes in the file/directory. Open the `/etc/crontab` file in the editor of your choice and add this line:

![](img/1fadbe8c-94d7-4748-b692-430794e5bad2.png)

Here, `00 6` tells us that Tripwire will check daily at 6 o'clock.

# How it works...

We first install the Tripwire package and during the installation, we fill in the details as asked. Once the installation completes, we initialize the tripwire database. After this, we check whether tripwire is working properly or not. For this, we first create a new file at any location, and then we run the tripwire interactive command. Once the command completes, we see in the output that it shows the new file has been added. This confirms that Tripwire is working perfectly. We then edit the Crontab configuration to run Tripwire automatically at a particular interval.

# Shorewall

Want to set up a Linux system as a firewall for a small network? Shorewall helps us to configure an enterprise-level firewall via standard Shorewall tools. Shorewall is actually built on Iptables, but it makes it easier to configure things.

# Getting ready

A Linux system with two network cards installed and working is needed for configuring Shorewall. One card will be used as an external network interface and the second will be used as an internal network interface. In our example, we are using `eth0` as external and `eth1` as internal. Configure both cards as per the network configuration. Make sure that you are able to ping another system on the local network and also something on the external network, the internet. On this system, we will be installing the Shorewall package and then configuring it as per our requirements.

# How to do it...

1.  We begin with installing Shorewall on our system using the `apt-get` command:

![](img/5395b3fc-45a0-4c50-b2ef-49f9227cb132.png)

2.  Once the installation is complete, try to start Shorewall. You will get an error message as shown here:

![](img/a3f65b1f-39c5-405e-a704-371e9ec9fa21.png)

This means we need to first configure Shorewall before it can start running.

3.  To configure Shorewall, edit the `/etc/default/shorewall` file in the editor of your choice. Look for the line that reads `startup=0` and change its value to `1`:

![](img/d11844d9-044e-4d80-9694-19be5b39389f.png)

4.  Next, edit the `/etc/shorewall/shorewall.conf` file and find the line that reads `IP_FORWARDING`. Verify that its value is set to `On`:

![](img/4c889d15-97e9-40b4-bd0c-5307d991bd30.png)

5.  The configuration files of Shorewall are located in the `/etc/shorewall` directory. The minimum files that are essential for it to work are interfaces, policy, rules, and zones. If any of these files are not found in the `/etc/shorewall` directory after its installation, we can find the same files in the `/usr/share/doc/shorewall/default-config/` directory. Copy the required files from this location to the `/etc/shorewall` directory.
6.  Now, edit the `/etc/shorewall/interfaces` file and add the lines shown in the following screenshot:

![](img/d6f9d931-b50b-42e3-b182-5fe4d999129f.png)

We are referring to `eth0` as `net` in our configuration and `eth1` as `local`. You can choose any other name, as long as it is alphanumeric and `5` characters or less. 

7.  Next, edit the `/etc/shorewall/zones` file. Zone is mainly used to set whether to use `ipv4` or `ipv6`:

![](img/bcf0a55f-7b45-4f20-a0ce-46e6136f502b.png)

In the previous configuration, `fw` refers to `me` or the shorewall firewall itself. The next two lines define `ipv4` for both the network interfaces.

8.  Now, edit the `/etc/shorewall/policy` policy file. This file is mainly used to set the overall policy of who is allowed to go where. Each line in this file is processed from top to bottom and each is read in the following format: if a packet is sent from the ____ to the __, then ______ it:

![](img/92628c47-c372-4647-b459-ba8ca2a1466f.png)

In our example, if we read the first policy, it will be read as follows: if a packet is sent from the local to the net, then accept it. You can add as many policies as you want in the same way, and the Shorewall firewall will work accordingly.

9.  Finally, we edit the `/etc/shorewall/rules` file. This file is used to create exceptions to the policy. It is mainly used if you wish to allow people from the external network into the internal network. A sample rules files is shown here:

![](img/a4fd086a-c6ed-4dde-b3be-56e1fcac9b45.png)

We have added a rule that says: `accept` a packet if it is sent from the `net` to the `fw` using the protocol of `tcp` on port number `80`.

10.  Once we are done with configuring the previous files as per our requirements, we can test the settings by running this command:

```
    shorewall check
```

11.  In the output shown, scroll to the bottom, and if it says `Shorewall configuration verified`, it means the settings have been done properly and now shorewall can be used as a firewall:

![](img/e9628bf5-0dbf-400a-adb2-e90edaba435e.png)

12.  Now, restart the `shorewall` service to apply the settings:

```
    serviceshorewall restart
```

# How it works...

We begin with installing shorewall on the system, which has two network interface cards. Once the installation is done, we edit the `/etc/default/shorewall` file and also the `/etc/shorewall/shorewall.conf` file. Then, we edit or create these files in the `/etc/shorewall` location: interfaces, policy, rules, and zones. And, we add the lines in each file as per the requirements given. Once the editing is done, we check that everything is fine and then we start the `shorewall` service to start our firewall.

# OSSEC

As a system administrator, we may want to keep track of authorized and unauthorized activity on your server. OSSEC may be the solution for this. It's an open source host-based intrusion detection system, which can be used for tracking server activity. When properly configured, OSSEC can perform log analysis, integrity checking, rootkit detection, time-based alerting, and many other things.

# Getting ready

To install and configure OSSEC, we will use an Ubuntu server. Additional packages such as gcc, libc, Apache, and PHP may be needed for compiling and running OSSEC. Also, if we want real-time alerting to work, a separate package would be needed for this. To install all the essential packages, run the command shown here:

![](img/fe91c673-3f00-4ab2-a8ff-27ed4224ddfb.png)

# How to do it...

In this section, we will learn how OSSEC can be installed and configured to monitor a local Ubuntu server. We will also test OSSEC against any file modifications:

1.  Our first step will be to download the latest version of OSSEC from its GitHub repository using the following command:

![](img/f8157bf9-d15d-4a3c-b34d-3ea0b328b5d6.png)

2.  Depending on where the download has been saved after completion, extract the downloaded file with the following command:

![](img/2c73cdb9-ae4a-4578-a5e9-47dda2db335d.png)

3.  Move inside the extracted directory and list its contents. We will see an install.sh script, which will be used to install OSSEC:

![](img/717b361a-c0ac-4d76-bae2-3a556b9b785b.png)

4.  Run install.sh as shown here to install OSSEC:

![](img/78b70848-419d-4210-909c-28e41c471ecb.png)

When prompted, we will select our language. So, if our language is English, then we will type `en` and press *Enter*.

5.  Once we press *Enter*, the following output will be seen:

![](img/e8837813-e6da-4c5d-b9a3-bcae440e222d.png)

6.  Press *Enter* again to continue. In the next screen, it will ask you to choose the kind of installation we want. Type `local` to monitor the server on which OSSEC is being installed, and then press *Enter*:

![](img/205e71ff-c5bb-4d19-8312-1b9d95984598.png)

7.  Next, we will choose the install location for OSSEC. The default install location is `/var/ossec`. Press Enter to continue:

![](img/b5640e9f-91ec-4199-9c01-f69467e10d06.png)

8.  We can configure OSSEC to get email notifications to our local email address. Type `y` and press Enter to do this:

![](img/d501403c-f4ce-4ceb-a78c-62439372e343.png)

9.  In the next step, we will be asked if we want to run the integrity check daemon and rootkit detection engine. Enter `Y` for both and press *Enter* to continue:

![](img/aadffc18-4dfe-494d-9243-132acef2e235.png)

10.  Next, we will enable active response:

![](img/d69baec1-dfde-4e8e-8f47-b94c56b789b9.png)

11.  Proceed further to enable the firewall-drop response:

![](img/44bba6c0-a756-44b5-90c3-ec51a3785393.png)

12.  We can add IPs to the white list, if we want. Otherwise, type `n` and press *Enter* to continue:

![](img/731933b9-edba-4dd0-8b0d-4d9dee8e8f26.png)

13.  Next, press *Enter* to enable remote Syslog.
14.  Once all the configuration is done, press *Enter* to start installation. Once the installation starts, the output shown here will appear:

![](img/1545caf2-9d5f-483d-bf4e-bdd010d83934.png)

15.  When the installation is complete, the following output will be seen:

![](img/fab2d89f-e599-40d3-aec7-84ba6876a7c1.png)

16.  After completing the installation, we can check the status of OSSEC with the following command:

![](img/e48c870d-aba7-48b8-8405-6f3e8578f550.png)

17.  To start OSSEC, run the following command:

![](img/e3e8d3ed-ba3a-4799-b2bb-14e3e81325a8.png)

18.  As soon as OSSEC starts, we will get an email alert. Type `mail` to check the mail, which will look like the following:

![](img/4b203e4b-da74-46f6-a6a5-2c560042d705.png)

19.  Our next step is to edit the main configuration file of OSSEC, which is the `/var/ossec/etc/ossec.conf` file. Open the `ossec.conf` configuration file using an editor like nano.
20.  When we open the file, it will show us the email configurations we specified during installation. We can change this setting at any time:

![](img/74a321a9-a3f6-4b10-b2a6-c9d1d27164df.png)

21.  According to the default configuration, OSSEC does not alert us when a new file is added to the server. We can change this setting by adding a new line just under the section, as shown here:

![](img/9846d0f7-045b-40bd-b054-0071373ba150.png)

22.  If we want OSSEC to send real-time alerts, we will have to make changes in the list of directories that OSSEC should check. To do this, we need to modify the following two lines to make OSSEC report changes in real time. Make the changes as shown here:

![](img/b0feaf19-eb7c-4220-9cca-3bec0f195ae1.png)

23.  Next, modify the `local_rules.xml` rules file, which is located inside the `/var/ossec/rul``es` directory, to include the rules for new files added to the system:

![](img/5ca07289-2832-459b-85a4-5cbe8fb098ce.png)

24.  When the previously mentioned changes are done, save and close the file. Then, restart OSSEC:

![](img/1df94d18-fe6a-4100-8d32-36cd595ea86b.png)

25.  Now, we will check whether OSSEC is working or not. Let's try to make few changes in `/etc/network/interfaces`. If OSSEC is working fine, we should receive an email alert mentioning that something has changed in the system. An alert such as the following will be seen:

![](img/918c655c-f547-4168-b9bf-652fc5e1713d.png)

# How it works...

We first install OSSEC on our Ubuntu server and during the installation, we provide the details of where we would like to receive the alerts being generated by OSSEC. We also enable the daemons that we want to use for monitoring also during the installation process. After the installation completes, we make changes in the configuration file to receive an alert every time a new file is added to the server. Other necessary changes are also done in the respective configuration files to get alerts from OSSEC.

# Snort

In today's enterprise environments, where security is a major issue, there are lots of tools available for securing network infrastructure and communication over the internet. Snort is one of those tools, available for free as it is open source. It is a lightweight network intrusion detection and prevention system. Snort works in three different modes: sniffer mode, packet logging mode, and network intrusion detection system mode.

# Getting ready

Before getting started with the installation of Snort, ensure that our system is up to date and install the required dependencies on it. To install the required dependencies, we run the following command:

![](img/ac079b4d-6734-437c-8797-1e1076eb638e.png)

# How to do it...

Snort can be installed on Ubuntu, either from its source code or through the deb package. In this section, we will install Snort using the deb package:

1.  To get started, we install on our Ubuntu system, using the `apt-get` command, as shown here:

![](img/8c2bec16-47c3-466f-a88a-fed29b775f5b.png)

2.  During the installation, we will be asked to select the interface on which Snort should listen for packets. The default interface selected is `eth0`, as shown here:

![](img/d290f619-a6a7-42d9-9aa0-c5ccce7dfdbf.png)

3.  Select the interface according to our system configuration:

![](img/e1e7e21e-473a-4bcf-9ccc-4fe1be246737.png)

4.  Now, let's get started with the sniffer mode of Snort. In sniffer mode, Snort reads the network's traffic and displays the human-readable translation. To test Snort in Sniffer mode, type the following command:

![](img/64f8a927-1f02-4c3a-863b-ab3a6236a4c3.png)

5.  In the output shown here, we can see the headers of traffic detected by Snort between the system, the router, and the internet:

![](img/3670fbe1-1219-4602-b7d2-c8b78aaf2250.png)

6.  The following output displays a summary of the traffic analyzed by Snort:

![](img/3a0e48f0-d1f2-4bff-a11f-501c89b1b713.png)

7.  If we want Snort to show the data too, we can run the following command:

```
-snort -vd
```

![](img/0ccf8ad6-3f30-4648-8dee-6b47b50166c2.png)

This will give the output shown previously.

8.  Now, let's get started with using the packet logger mode of Snort. If we want Snort to show just the traffic headers and log the complete traffic details on disk, we need to first specify a directory where Snort can save its reports. For this, we move inside `/var/log/snort` and create a directory with any name, as shown here:

![](img/0b18e411-c318-408f-b2b5-b16883573d9f.png)

9.  Now, run the command shown here and Snort's log will be saved inside the `logs_snort` directory:

![](img/7468f218-cec1-4876-8608-a629a0834a66.png)

10.  Once we have logged enough packets, we stop the command. Now, we can check inside our `logs_snort` directory and see that a file has been created:

![](img/371cef63-1a5e-4b28-957a-f0a6622afef9.png)

11.  If we want to read the content of this log file, which was created in the previous step, we run this command:

![](img/25d7d717-2127-4da9-bb92-86cfe2c3fadd.png)

We can see the complete output, as shown previously.

# How it works...

Snort works in three different modes: sniffer mode, packet logging mode, and network intrusion detection system mode. Based on the arguments used when running Snort, the respective mode gets initiated and we can capture and monitor logs accordingly.

# Rsync and Grsync – backup tool

**Remote sync** (**Rsync**) is a local and remote file synchronization tool. Using its algorithm, it can efficiently copy and sync files, which allows us to transfer only the differences between two sets of files. Grsync is a GUI frontend for the Rsync tool. Being cross-platform, it works on Linux, Windows, and macOS.

# Getting ready

Due to its popularity on Linux and Unix-like systems, Rsync comes pre-installed in most Linux distributions by default. However, if it is not installed, we can install it by running the following command:

![](img/ec231faa-5f7b-4816-be69-b027dfaa048e.png)

Unlike Rsync, Grsync does not come pre-installed in Linux distributions. To install Grsync on Ubuntu, run the following command:

![](img/713973d3-7e55-4864-bb56-98269fc3e62f.png)

To use Rsync and Grsync for remote file syncing, it's essential to have SSH access enabled on both the systems, and rsync and grsync should be installed on both systems.

# How to do it...

In this section, we will see how to use rsync and grsync to synchronize files/directories locally, as well as remotely from one system to another.

1.  Let's start by creating two test directories on one system and also create some test files inside one of the directories. To do this, we run the following commands:

![](img/0f87db92-a2f7-49d4-90f1-1e4514483423.png)

Here, we have created two directories, `dir1` and `dir2`, and `dir1` has five empty files created inside it.

2.  If we want to sync the contents of `dir1` to `dir2` locally, we can do so by using the following command:

![](img/27d498d1-92d6-43cd-8b3d-c89b3b3db82e.png)

The `-r` option refers to a recursive method, and the trailing slash (`/`) at the end of `dir1` refers to the contents of `dir1`.

3.  If we want to sync the `dir1` directory to another system remotely, we can do so by using the following command:

![](img/9428ce22-a665-43ba-acb2-0ced91114b13.png)

Here, we mention the destination address preceded by the username of the destination system. When we run the command, it will ask for the password of the remote user. Once the password is entered, the syncing will be done.

4.  Once the previous command completes its working, we can check on the remote system and see that the `dir1` directory has been synced on the remote system, as seen here:

![](img/cc68de14-d8c8-470c-9d9f-cc08a35826b7.png)

5.  Now, let's see how to use Grsync for syncing files using a GUI. We can launch Grsync either through the Application Menu or through the command line, using the `grsync` command. The default interface of Grsync looks like this:

![](img/648c2edd-303d-48fa-8a14-40181c70c94d.png)

6.  To back up a directory (`/root/backup`) from the local system to the remote system, enter the source and destination details as shown here:

![](img/458e5a42-6531-4407-9670-21d8aede0688.png)

7.  After entering the previous details, go to the File menu and click on Simulation to verify that the details entered are collected:

![](img/ed23a94b-7009-48e5-be72-0258cebb3b26.png)

Once you click on Simulation, it will prompt you to enter the password of the remote user.

8.  If the details entered are correct and everything is OK, a Completed Successfully message will appear, as shown here:

![](img/8b37b156-9280-4ea4-97ab-c19bee1a24c0.png)

9.  Now, we can start the file transfer by clicking on the Execute option in the File menu:

![](img/ebd422f5-2840-437d-b7bf-ed8546c66c4b.png)

Again, it will prompt for the remote user password. Provide the password to proceed.

10.  Depending on the contents of the directory, the process may take some time. Once it completes, a Completed Successfully message will appear as shown here:

![](img/6d78c542-fff7-464d-b174-b4b5d4c7b163.png)

11.  We can verify that the transfer was successful by checking for the backup files on the remote system:

![](img/96e0ae31-32e7-40d0-ad24-f53093036597.png)

# How it works...

Rsync and Grsync are synchronization tools that work locally, as well as remotely. While Rsync is a command-line tool, Grsync provides a GUI to Rsync. Using different options available in these tools, we can manage backup synchronization between two systems.