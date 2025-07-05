# Chapter 5. Setting Up a Home Server

Since you are now familiar with creating an entire root filesystem from scratch, it is a good time to create something a little more complex. Granted that a desktop can be quite useful by itself, but very often, a desktop OS on these kinds of ARM development boards might be a little too slow. However, these devices are very useful for low-power home servers.

The term home server is used here, as most services are useful to a person at home, but using a Cubieboard in a colocation center and getting it to function as a web server and mail server, for example, also works quite well. However, when doing that, security does come to mind. Security is not strongly addressed in this chapter, as it is a subject that requires a lot of attention and strong knowledge of security, both of which are out of the scope of this book. It has to be said that these instructions are not specific to a Cubieboard at all, and there are obviously many more services that could be thought of.

In this chapter, we will cover the following topics:

*   Accessing the Cubieboard remotely
*   Learning how to start, stop, and restart services
*   Adding and removing a service to be started at boot
*   Automatically running tasks at scheduled times
*   Set up various services (Squid, Apache, Samba, transmission, and ownCloud)

# Prerequisites for the home server board

In this chapter, several software packages will be installed upon the previous Debian or Ubuntu installation. If this installation has seen too much wear and tear due to experimentation, it is recommended to go over [Chapter 4](ch04.html "Chapter 4. Manually Installing an Alternative Operating System"), *Manually Installing an Alternative Operating System*, again.

It is wise to skip the final segment, where a graphical desktop environment is installed, as it will serve no purpose in this chapter. The initial Fedora installation on the SD card can also be reused, but it is up to you to identify the differences between these distributions.

# Accessing the server remotely

Most of the time, if not all the time, the Cubieboard has been accessed via a serial console, or directly via the keyboard and mouse while it is connected to a monitor. This works fine, but after the setting up and tinkering is done, it might be interesting to get it to sit in a different room where it can perform its task unattended. It could even be possible to have several boards running in a dataserver, for example, where remote access might become crucial. Thus, having remote access is very useful.

The most common and well-established way to connect to a Debian or Ubuntu machine is via an `ssh` server. Installing an `ssh` server is simple; just use the following command:

```
packt@PacktPublishing:~$ sudo apt-get install openssh-server

```

Just as with UART access, PuTTY can be used for `ssh` access. However, the important thing here is to set the connection type to `ssh`. The port should be set to `22` by default, and the correct IP or hostname needs to be used. See the following screenshot for an example of this:

![Accessing the server remotely](img/1572OS_05_01.jpg)

If you are using an existing Linux command line, `ssh` can be invoked quite easily, as shown in the following screenshot:

![Accessing the server remotely](img/1572OS_05_09.jpg)

The first time you connect to a host, the `ssh` fingerprint is displayed followed by a question as to whether you are sure you want to connect. In this case, it is safe to accept the proposal to continue.

### Tip

When connecting over the Internet, it is recommended that you always verify the key. If someone messed around with the server or tried to perform a man-in-the-middle attack, this key will no longer match, and you will know that something is wrong.

After entering the password, the familiar `packt@PacktPublishing:~$` command prompt will appear.

# Interacting with services

On a server, it is quite common to start, stop, or restart a service. This is not only while trying to fix a problem, but also sometimes to reload a configuration, for example. It might be that a service is used to run only occasionally but not every time during booting. For these purposes, the so-called startup scripts are available. When a service is installed, usually, a script to control its behavior is placed at `/etc/init.d`.

### Tip

For a server, it usually makes sense to configure networking either statically or via a DHCP server directly in the `interfaces` file. This to ensure networking is always available, as a graphical desktop is often not installed.

Let us take the networking script, for example. It is responsible for bringing up a network device, but only those configured at `/etc/networking/interfaces`. Refer to [Chapter 4](ch04.html "Chapter 4. Manually Installing an Alternative Operating System"), *Manually Installing an Alternative Operating System*, on how to properly set up a network configuration; it is assumed that `eth0` is configured in this way.

### Note

Stopping the networking service will deactivate all the networking interfaces! While this does not have to be a problem, be careful when using this command if you're using the connection remotely.

# Starting, stopping, restarting, or reloading a service

To stop the network interface, issue the following command. If you are logged in as root, which is not recommended, omit the `sudo` command.

```
packt@PacktPublishing:~$ sudo /etc/init.d/networking stop
Deconfiguring network interfaces...done.

```

Similarly, the network interface can be started again using the `start` parameter, as follows:

```
packt@PacktPublishing:~$ sudo /etc/init.d/networking start
Configuring network interfaces...

```

The same can be done using the `restart` or `reload` parameter, where `reload` forces the service to reload its configuration.

### Tip

Some distributions, such as Ubuntu, might use upstart or even SystemD. Sometimes, scripts are left behind in the `/etc/init.d` directory that can be used as described previously, but this is not always the case, and thus it might be required to get familiar with SystemD or upstart.

# Adding or removing a service from the boot up

Sometimes, it is required to always make a service start during boot time. For example, in the previous chapter, `lightdm` was installed to provide a graphical login service. This service was automatically added to be started during booting. Preventing `lightdm` from starting during booting any longer can be accomplished using the `update-rc.d` command. Passing the optional `-f` flag forces removal, as shown here:

```
packt@PacktPublishing:~$ sudo update-rc.d -f lightdm remove
update-rc.d: using dependency based boot sequencing

```

The graphical login manager will now no longer be started after a reboot using the preceding section to stop `lightdm` immediately; a reboot is then not even needed.

Adding a service is quite similar; instead of asking it to be removed from the boot time services, the default keyword asks that the service be started during booting, as shown here:

```
packt@PacktPublishing:~$ sudo update-rc.d lightdm defaults
update-rc.d: using dependency based boot sequencing

```

Defaults indicate that all the default runlevel services should be started or stopped. A Linux system has various runlevels, and you are encouraged to read more about it.

After each reboot, `lightdm` will be started. The preceding section was all specific to Debian. Ubuntu, however, has backwards-compatible scripts for most services in `/etc/init.d` and can be used as described previously.

### Note

As mentioned in the earlier chapter, both Debian's SystemV and Ubuntu's upstart will, in time, be replaced by SystemD or both. It is up to you to learn more about SystemD when the time comes.

# Running scheduled tasks automatically

It is often necessary to have certain tasks run at scheduled intervals. Think of downloading and updating a virus database, for example. For this purpose, Linux systems are equipped with a program called **Cron**. Cron normally runs in the background and checks whether it was given any jobs to run at a certain time. While it is up to you to get to know Cron in detail, by default on Debian, Cron makes running Cron-jobs very easy by supplying four directories in `/etc`, namely, `cron.hourly`, `cron.daily`, `cron.weekly`, and `cron.monthly`. From the names, it should be obvious what their intentions are. Placing an executable file here makes Cron run the command hourly, daily, weekly, or monthly.

A simple example is to ask `apt-get` to update its local database every day; a simple script can be created for this purpose. Using the editor as root, create a new file called `apt-update` in `/etc/cron.daily/`, as follows:

```
packt@PacktPublishing:~$ sudo nano /etc/cron.daily/apt-get
#!/bin/sh
apt-get update

```

The first line is important, as it tells the system that this is a shell script, or rather, that `/bin/sh` needs to be used to execute the lines in the rest of the file. Cron runs all its jobs in `/etc/cron.*` by root by default, so there is no need to prepend it with `sudo`.

With the file saved, it is important to give it the permission to execute. By default, it is just a text file. Cron will only execute files with the proper permissions. This is easily fixed using `chmod` to mark this newly created script executable, as shown in the following command:

```
packt@PacktPublishing:~$ sudo chmod u+x /etc/cron.daily/apt-update

```

It needs to be said, however, that while this was a very useful example, Debian and Ubuntu by default update the apt repository on a daily basis. So, while this serves well as an example, it is recommended that you clean it up afterwards, as shown in the following command:

```
packt@PacktPublishing:~$ sudo rm /etc/cron.daily/apt-update

```

# Setting up a proxy server

When bandwidth is scarce and has to be shared with multiple members of a household, and on top of that, a reduction of ads is desired, a proxy server can offer a solution. Additionally, it allows Internet access without complicated firewall rules; just configure the browser to use the proxy. In the following subsections, the steps to set up a proxy server are explained.

## Installing Squid

The proxy server used in this book is called Squid, and as you learned from the previous chapter, it can be installed easily with the following command:

```
packt@PacktPublishing:~$ sudo apt-get install squid

```

Squid in Debian comes with a reasonable default configuration file at `/etc/squid/squid.conf`. Using nano or any other editor, the file can be opened and examined. The squid configuration file is very heavy on comments, working as a guide or manual.

By default, Squid will only listen to the traffic on connections that it thinks are from the internal network. The lines starting with `acl localnet` followed by the internal network, IP-range, indicates this. It should be noted that an additional `localnet` line can be added, but be careful—adding an IP-range internal network that is not local can mean that the proxy is accessible worldwide, so do be careful when experimenting here. This, however, only defines the `localnet`; it has not been granted access yet. Searching through `squid.conf`, a commented section reading the following command will be found:

```
#http_access allow localnet

```

This line allows HTTP access where the source originates from `localnet`, which we just learned about, and which was defined as the internal network range. You can uncomment it by removing the hashtag, or the number sign; this grants access to the so-called `localnet` line, and after restarting Squid, the proxy should now allow usage from the internal network.

## Setting up a caching proxy

A proxy server can temporarily store frequently accessed data so that it only needs to be downloaded over an Internet connection once. This can, in certain cases, greatly reduce bandwidth usage; for example, imagine that a household of five computers with all of them requiring to download a certain update file. If all the five computers download this update via the proxy, the proxy only downloads this the first time the file is requested. The moment a second computer requests that same file, the proxy would offer its internally stored file making the download much faster and not burden the Internet connection.

### Note

By default, Squid uses a cache of 100 MB only. Take a look at the `cache_dir` parameter to learn more about this.

Squid does not require any additional configuration to function as a caching proxy. After it has been installed, it is automatically started and configured to be started upon a reboot. To change any of these behaviors, refer to the earlier two subsections.

### Configuring a browser to use the proxy

Without covering every browser or every operating system out there, at least one example will be shown here. In this case, we choose Mozilla Firefox. To configure the browser to use the proxy, follow these steps:

1.  First, open the **Preferences** option, which, depending on the operating system used, is under **Edit** or under **Tools**.
2.  In the preference screen, navigate to the **Advanced** tab, and from there, to the **Network** tab. The first button on the right reads **Settings**, and it holds the network configuration, as shown in the following screenshot:![Configuring a browser to use the proxy](img/1572OS_05_02.jpg)
3.  In the **Settings** dialog, select **Manual proxy configuration**, which will enable the edit boxes.
4.  For the **HTTP Proxy**, the hostname or the IP address needs to be entered. The default port for Squid is **3128**.
5.  Finally, the **Use this proxy server for all protocols** option can be safely enabled, as Squid covers them all, as shown in the following screenshot:![Configuring a browser to use the proxy](img/1572OS_05_03.jpg)
6.  If the IP address is unknown, it can be obtained using the `ifconfig` command on the Cubieboard where Squid is running. In this case, the IP address is `192.168.0.10`, as shown here:![Configuring a browser to use the proxy](img/1572OS_05_04.jpg)

It is not strictly required to use `sudo` to access the `ifconfig` information. Most of the time, it can be accessed via its full path, `/sbin/ifconfig` in this case, without requiring elevated privileges. Additionally, `ifconfig` might be replaced by the `ip addr` command on certain distributions.

Starting a web-surfing session should now go via the proxy. Manual configuration of a proxy is not strictly required. As seen here, there is the **Use system proxy settings** or **Auto-detect proxy settings for this network** option. When using the first option, whatever is configured by and for the operating system is used as the default proxy by the browser. The latter option is used if the proxy settings can be detected via the so-called WPAD, which is up to the reader to learn more about and can be found at [http://en.wikipedia.org/wiki/Web_Proxy_Autodiscovery_Protocol](http://en.wikipedia.org/wiki/Web_Proxy_Autodiscovery_Protocol). Both of these options, however, are not covered here.

## Setting up a blocking proxy

One of the advantages of using a proxy is that certain sites can be easily blocked. Let us say the sites `hotmail.com`, `ebay.com`, and `live.com` need to be blocked. This list of domain names will have to be stored somewhere, for example, `blocked.domains.acl` in the `/etc/squid` directory. To do this, follow these steps:

1.  Start editing the file and add those names, as follows:

    ```
    packt@PacktPublishing:~$ sudo nano /etc/squid/blocked.domains.acl
    hotmail.com
    ebay.com
    live.com

    ```

2.  Next, go to a very specific section in the configuration file, as the order does matter. Find the section called `# TAG: acl`, and scroll beyond the big commented section where the `acl` tag gets documented. The `acl localnet` definitions will show up as seen in the earlier *Installing Squid* section. Just before the next section starts, find `# TAG: http_access` and add the following line:

    ```
    acl blockeddomain dstdomain "/etc/squid/blocked.domains.acl"

    ```

    In short, under the section called `acl`, a new item will be added, just before the `http_access` section. Here, Squid is instructed to create an `acl` called *blocked domain* and mark the domains as destination domains from the file just created. So now there should be a list of `acls` with the blocked domain one being last.

3.  Immediately following the `acl` section, the `http_access` section starts. Here, the order is crucial. Just above the sections where the localhost and `localnet` were allowed, the following command needs to be added:

    ```
    http_access deny blockeddomain

    ```

    The reason is that, first, things get denied, such as the blocked domains, and then localhost and `localnet` are granted access to what is left. Finally, everything else gets blocked for those that have not been allowed anything.

4.  Using the `restart` command you learned earlier in this chapter, Squid can be restarted and will now refuse to load all the content from the mentioned sites.

Taking this example a step further, it would be nice to block all domains that do nothing but serve ads. While creating a file to list all these domains is of course a valid solution, it becomes quite tedious to list all domains and all of its permutations. Squid can instead do regular expressions on a list, making things a lot easier. To do this, follow these steps:

1.  As before, create a file named `squid.adservers`, and enter the following regular expression. It's okay if you don't understand what any of it means; it's only an example for now.

    ```
    (^|\.)wikia-ads\.wikia\.com$

    ```

2.  After saving the file, open `squid.conf`, and just like before, add the following lines to the appropriate sections:

    ```
    acl ads dstdom_regex -i "/etc/squid/squid.adservers"
    http_access deny ads

    ```

3.  Squid is instructed to create a list following the file, but this time using a regular expression for the destination domain list. Again, Squid will then deny all HTTP access to the domains in the list ads, which in this case is `wikia-ads.wikia.com`.

Manually, adding regular expressions for each and every ad-network can be tiresome, especially when trying to maintain them. Luckily, a group of people maintains a list of known ad-servers that can be used without too much work; the list can be found at [http://pgl.yoyo.org/adservers/](http://pgl.yoyo.org/adservers/). Someone was even kind enough to provide a script that can be used to download the list and prepare it to be used with Squid. Download the following file using `wget`, as seen in the following example:

```
packt@PacktPublishing:~$ wget http://pgl.yoyo.org/adservers/scripts/squid/update-squid-adservers.txt

```

Before being able to use this script, it does require minor modification. To do this, follow these steps:

1.  Open the file, find `listurl`, and replace it as shown here:

    ```
    listurl='http://pgl.yoyo.org/adservers/serverlist.php?hostformat=squid-dstdom-regex&mimetype=plaintext'

    ```

2.  Also, double-check the target file parameter, for it will have to match what was used on the ad's `acl` rule that was defined as shown here:

    ```
    targetfile='/etc/squid/squid.adservers'

    ```

3.  After saving the file, it can be run as root, and it will download an up-to-date list of ad-servers and reload this data into Squid. Make use of `sh` here, as `sh` is run as root and it is told to run the script as if it were a shell script:

    ```
    packt@PacktPublishing:~$ sudo sh update-squid-adservers.txt
    Reloading Squid configuration files.
    done.
    packt@PacktPublishing:~$

    ```

4.  To double-check whether all this worked accordingly, open the target file, probably `/etc/squid.adservers` by default, unless adjusted as suggested earlier.
5.  One final step is to have this script update on a monthly basis. Using the earlier section on how to set up a Cron-job should make this task easy.

    ### Tip

    Using the `cp` command, the file can be copied to the appropriate directory. It is probably a wise idea to drop the `.txt` extension when copying.

# Setting up a web server

A personal web server is something that a lot of people involved in Linux have. Setting up one properly is, however, a topic worth of an entire book. In this section, only the most basic of steps will be covered for you to get started. By no means is it complete or secure, as we will rely on the default settings that Debian or Ubuntu ship with in their packages.

There are several well-known web servers, but without a doubt, the most well-known is the following Apache web server:

```
packt@PacktPublishing:~$ sudo apt-get install apache2

```

As before, Debian will automatically start Apache, and it can be used right away. Using a web browser by typing the IP or hostname, such as `http://192.168.0.10` which was used earlier, into the URL bar will open the following window:

![Setting up a web server](img/1572OS_05_10.jpg)

The file being served is from `/var/www/index.html`. Feel free to edit this document or even entirely replace it.

### Tip

It might be useful to give the web developer access rights to this directory and its files. Quite often, there is a group associated with the `webroot` directory. Admins often even create several groups for the various webroots they might have. For simplicity's sake, the user and group is set to our example user, as follows:

```
sudo chown -R packt:packt /var/www

```

# Setting up a file server

Editing the files directly on the Cubieboard can be tiresome for sure. Even when using the previously installed Xfce desktop, once it is placed at a remote location, accessing the files remotely will be quite useful. It might be the case that the media files are desired to be shared throughout the household.

There are two approaches to this situation. With the `ssh` server installed earlier in this chapter, any modern Linux desktop environment has the ability to access files via `ssh`. While this works well, it is left to the reader as an exercise. What this section will focus on is installing and setting up Samba as a file server. To do this, follow these steps:

1.  The files shared via Samba are accessible via many operating systems and in many devices it can be easily installed, as shown here:

    ```
    packt@PacktPublishing:~$ sudo apt-get install samba

    ```

2.  After Samba has been installed, it is probably a good idea to create a directory that will be shared using the `mkdir` command. It can be called `mediafiles` and lives in the root of the filesystem, as shown here:

    ```
    packt@PacktPublishing:~$ mkdir /mediafiles

    ```

3.  Using an editor, open the Samba configuration file at `/etc/samba/smb.conf`, as shown in the following command:

    ```
    packt@PacktPublishing:~$ sudo nano /etc/samba/smb.conf

    ```

4.  One variable that probably should be changed is the so-called workgroup; this is the name of the network that Samba will try to join:

    ```
    # Change this to the workgroup/NT-domain name your Samba server 
    # will part of
     workgroup = PacktNet

    ```

5.  The other thing that needs to be added at the bottom of the file is a section where the previously created directory is being shared, as shown here:

    ```
    [mediafiles]
     comment = Shared media files
     read only = no
     path = /mediafiles
     guest ok = yes

    ```

Security-wise, this is not the best protection, as everybody is allowed to read and write to the media files and no valid account is required to access these files. There is one final barrier, however, and that is the permissions left on the directory. Samba will run on the media files as the user tries to access this directory.

Since this share is being used as a guest user, or no user, the permissions will be applied when reading the directory media files.

1.  Using `chmod` to change the permission to allow everybody to read and write makes it possible that this share can be used by everybody, as shown in the following command:

    ```
    packt@PacktPublishing:~$ sudo chmod o+rw /mediafiles/

    ```

    Within a home network, this is not a problem. Protecting things better, however, is a good exercise left to the reader. Samba is a great suite offering a whole lot more than just file sharing. Learn more at [http://www.samba.org/](http://www.samba.org/).

2.  After saving these changes, Samba needs to be restarted.
3.  Using a file browser on a desktop, there should be a network workgroup called PacktNet, as was defined for the workgroup, and in that network, amongst other possible hosts, there should be `PacktPublishing`, the name of the host, as shown in the following screenshot:![Setting up a file server](img/1572OS_05_05.jpg)
4.  Finally, within `PacktPublishing`, the freshly shared directory `mediafiles` will become visible.![Setting up a file server](img/1572OS_05_06.jpg)

# Setting up a torrent server

Torrents are a common way of sharing data. Many Linux distribution ISO files are shared via torrents. Having a dedicated server dealing with torrents without having the main PC turned on is also very useful. It has to be noted, however, that some storage for that torrent data is required. Having all data on a SD card is probably not wise. Have a look at the following command to install transmission, a torrent server:

```
packt@PacktPublishing:~$ sudo apt-get install transmission-daemon

```

After the transmission is installed, it will need to be configured if the remote administration is desired. By default, the remove administration feature will only listen on the localhost. Using `sudo`, the configuration file should be opened, and two items, `rpc-password` and `rpc-whitelist`, need to be changed. The `rpc-whitelist` item determines which hosts can access the server, and the `rpc-password` item determines which password was used by any clients connecting to the server, as shown here:

```
packt@PacktPublishing:~$ sudo nano /var/lib/transmission-daemon/info/settings.json
 "rpc-password": "mysecretpassword",
 "rpc-whitelist": "127.0.0.1,192.168.*.*",

```

The default password set is an encrypted hash, but when modifying it, a regular plain-text password can be used. This seems like a security problem, and it is, but only very briefly. Upon being told to reload its configuration file, the transmission will read the password, encrypt it, and rewrite the configuration file.

Transmission needs to be reloaded because if the transmission is restarted, it will not re-read the configuration file but will rewrite configuration file on exit. The rest of the defaults should be satisfactory though it might be wise to double-check that `rpc-authentication-required` is set to `true` and that `rpc-enabled` is also `true`. Additionally, the `rpc-username` variable can be changed to something memorable, but do remember to use the new username in the remainder of this chapter, as shown here:

```
packt@PacktPublishing:~$ sudo /etc/init.d/transmission-daemon reload
Reloading bittorrent daemon: transmission-daemon.

```

Once the transmission is reloaded, a web browser can be used to browse to the IP address or the hostname on port `9091` and can be further configured from there, as shown in the following screenshot:

![Setting up a torrent server](img/1572OS_05_07.jpg)

The web interface is really nice and very usable, and there are various transmission clients that can remotely connect. They work as if the client ran locally but talk to the transmission-daemon running on the network. One such program is called `transmission-remote-gtk`. For Android, for example, there exists an application called **Transdroid**.

# Setting up a personal cloud

The cloud is where everything is these days. However, a lot of people are not happy with storing everything in the cloud on a random server. Luckily, there is something called ownCloud. This is a web service that lets you have your own personal cloud.

For this book, ownCloud will be installed using the SQLite database. For very light, single user workloads, SQLite will be fine.

### Tip

A more serious, multiuser installation would strongly benefit from using PostgreSQL or even MySQL; however, those systems are far more complex to set up and work with. After getting more comfortable with Linux in general, this can be an exciting exercise to the reader.

Installing **Mail Transport Agent** (**MTA**) is suggested by ownCloud; However, installing MTA is also beyond the scope here. To bypass the installation of any MTA, a fake package called `lsb-invalid-mta` exists, which can be installed as shown here:

```
packt@PacktPublishing:~$ sudo apt-get install lsb-invalid-mta

```

Unfortunately, as of the time of writing this book, the ownCloud package is not yet available for Debian Wheezy, but it is for later versions. Also, Ubuntu has had it available in its repositories for a while now. In some cases, such as the ownCloud package, it is available in a different repository—the backports repository. This was covered already in the previous chapter, where the apt repository was mentioned. Adding the backports repository can be done by adding the following line. Again, use an appropriate mirror if you can. In the following example, the `nl` mirror is used:

```
deb http://ftp.nl.debian.org/debian wheezy-backports main

```

Remember to update the apt database after adding the backports repository. There are a few dependencies of the SQLite variant of ownCloud that were put in a metapackage called `owncloud-sqlite`, as follows:

```
packt@PacktPublishing:~$ sudo apt-get install owncloud-sqlite

```

With the ownCloud SQLite dependencies installed, it is time to install ownCloud itself. Unfortunately, the ownCloud package, by default, wants to install a few suggested packages for a database. In the case of Debian Wheezy, it will always want to try to install the MySQL database even though a different database will be used.

To remedy this, we introduce a new options to `apt-get`, `–no-install-suggests`, and `–no-install-recommends`, which, as the names suggest, do not install any suggested or recommended packages. But even that will not prevent the full installation of MySQL. By adding `mysql-server-` (notice the dash at the end of the list of packages to be installed), apt will not install MySQL Server, as shown here:

```
packt@PacktPublishing:~$ sudo apt-get install –no-install-suggests – no-install-recommends owncloud mysql-server-

```

At the time of writing this book, the ownCloud package in the `wheezy-backports` repository is broken. At least a certain version of `php-getid3`, which is not available in the standard repositories, is required by ownCloud, as shown here:

```
owncloud : Depends: php-getid3 (>= 1.9.5~) but 1.9.3-1+deb7u1 is to be installed

```

This package, however, is also available in the `wheezy-backports` repository, but apt needs to be instructed to install it specifically from there. The `-t` parameter tells apt to install a package from a specific repository, as shown here:

```
root@PacktPublishing:~$ sudo apt-get -t wheezy-backports install php-getid3

```

It might be required to install this or other packages from the `backports` repository as a dependency of ownCloud. Using a web browser, navigate to the IP or hostname of the server, and the ownCloud setup wizard will display the following screenshot:

![Setting up a personal cloud](img/1572OS_05_08.jpg)

By logging in for the first time, an administrative account will be created. Initial setup might take a little while, after which ownCloud is ready for use.

# Summary

While this chapter had nothing specific for the Cubieboard, it did teach some basic administration tasks and used them to set up some basic but useful services for a home server. While there are many more interesting services to think of, such as a DHCP server (`ics`), a printer server (`cups`), or a DNS server (`bind`), on top of that, one can build a device and incorporate it with the web server and control a light switch via a web page.

The next chapter will work on upgrading the bootloader and the kernel, two reasonably easy tasks.