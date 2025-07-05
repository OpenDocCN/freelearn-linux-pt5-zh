# Chapter 8. Yum and Repositories

In this chapter, we'll cover the following recipes:

*   Managing yum history
*   Creating a copy (mirror) of any (RHN) repository
*   Configuring additional repositories
*   Setting up yum to automatically update
*   Configuring `logrotate` for yum
*   Recovering from a corrupted RPM database

# Introduction

Originally, you needed to compile your GNU/Linux system manually from source, which used to be time consuming and could be problematic if you couldn't get your dependencies straight. Red Hat created **Red Hat Package Manager** (**RPM**) in 1998 to address the concerns of dependencies and reduce the time needed to install a system (among others). Since then, RPM has been improved by the Open Source community. One such improvement is yum.

**Yellowdog Updater, Modified** (**yum**) is a package management tool using RPM. It allows RPM to access remote repositories of RPM files and will automatically download the required RPM files based on the dependency information provided by RPM.

Without a Red Hat Network subscription, you will not get access to updates.

Besides Red Hat Network, you can purchase Red Hat Satellite if you want even more control of your Red Hat systems.

# Managing yum history

An often overlooked feature of yum is the history. It allows you to perform a load of additional features that can save your skin in an enterprise environment.

It allows you to turn back the proverbial clock to the last functioning state of an application should there be an issue with a package update, without having to worry about dependencies and so on.

## How to do it…

In this recipe, I'll show you a couple of the most used yum history features.

### Your yum history

Use the following command to show your yum history:

```
~]# yum history list

```

The preceding command will list the output, as follows:

![Your yum history](img/00053.jpeg)

### Information about a yum transaction or package

Show the details of a yum transaction by executing the following command:

```
~]# yum history info 1

```

This will show you all about this single transaction:

![Information about a yum transaction or package](img/00054.jpeg)

Show the details of a package installed with yum through the following:

```
~]# yum history info ntp

```

This will show information about all the transactions that have modified the `ntp` package in some way (installed/updated/removed):

![Information about a yum transaction or package](img/00055.jpeg)

### Undoing/redoing certain yum transactions

Undo a specific transaction through the following command:

```
~]# yum history undo 7

```

This command undoes a specific transaction (defined by the ID), as shown in the following screenshot:

![Undoing/redoing certain yum transactions](img/00056.jpeg)

Now, you can redo a specific transaction using the following:

```
~]# yum history redo 7

```

This command will reperform a specific transaction (as defined by the transaction ID), as follows:

![Undoing/redoing certain yum transactions](img/00057.jpeg)

### Roll back to a certain point in your transaction history

This allows you to undo all transactions up until the transaction ID that you specify. Run the following command:

```
~]# yum history rollback 6

```

Here, the transaction ID up to which you roll back is `6`. You will get the following output:

![Roll back to a certain point in your transaction history](img/00058.jpeg)

## There's more…

You have to be careful when you use history options such as undo and rollback. Yum does its best to comply, but it cannot restore configurations, and it will not restore previous versions of your configuration files if you have edited them. This is not a fail-safe option if you don't have any backups. Although both options are very useful, I recommend that you do not use them too often. When you do use them, try to keep the impact of the transactions as small as possible. The smaller the delta, the more chance of succeeding in undoing or rolling back!

## See also

Refer to the *yum(8)* man pages for more information about yum history options.

# Creating a copy of an RHN repository

In this recipe, I'll show you how you can set up a yum repository for Red Hat Network-based and "plain" yum repositories.

## Getting ready

Before you create a copy of an RHN repository, you need to ensure that you have a valid subscription to the repository that you want to duplicate. When this prerequisite is met, you can perform this recipe from the machine that uses the subscription.

## How to do it…

Before being able to create yum repositories, we need to install a couple of tools by performing the following steps:

1.  Install the `createrepo` and `yum-utils` packages using the following command:

    ```
    ~]# yum install -y yum-utils createrepo

    ```

2.  Now, install the Apache web server, as follows:

    ```
    ~]# yum install -y httpd

    ```

### Syncing RHN repositories

You can only sync RHN subscriptions that you have access to. Perform the following steps:

1.  Create a directory to hold the RHN `rhel7` repository, as follows:

    ```
    ~]# mkdir /var/www/html/repo/rhel/rhel-x86_64-server-7/packages

    ```

2.  Now, create `/mnt/iso` by executing the following command:

    ```
    ~]# mkdir -p /mnt/iso

    ```

3.  Mount the RHEL 7 Server DVD through the following:

    ```
    ~]# mount -o loop,ro /tmp/rhel-server-7.0-x86_64-dvd.iso /mnt/iso

    ```

4.  Now, copy the `*-comps-Server.x86_64.xml` file from the RHEL Server DVD to your `repo` directory. The following command will help in this:

    ```
    ~]# cp /mnt/iso/repodata/*-comps-Server.x86_64.xml /var/www/html/repo/rhel/comps-Server.x86_64.xml

    ```

5.  Unmount the RHEL Server DVD, as follows:

    ```
    ~]# umount /mnt/iso

    ```

6.  Synchronize the RHEL 7 OS repository by running the following command: (This may take a while… I suggest you kill time drinking a cup of freshly ground Arabica coffee!)

    ```
    ~]# reposync --repoid=rhel-7-server-rpms --norepopath –download_path=/var/www/html/repo/rhel/rhel-x86_64-server-7/packages

    ```

7.  Next, create the local repository (depending on your hardware, this may take a long time), as follows:

    ```
    ~]# cd /var/www/html/repo/rhel/rhel-x86_64-server-7/
    ~]# createrepo --groupfile=/var/www/html/repo/rhel/comps-Server.x86_64.xml .

    ```

8.  Finally, test your repository through the following:

    ```
    ~]# curl http://localhost/repo/rhel/rhel-x86_64-server-7/repodata/repomd.xml

    ```

Let's create a copy of the EPEL repository through the following steps:

1.  First, install the EPEL repository, as follows:

    ```
    ~]# yum install -y epel-release

    ```

2.  Create a directory to hold the EPEL repository by executing the following command:

    ```
    ~]# mkdir -p /var/www/html/repo/epel/7/x86_64

    ```

3.  Now, download the `*-comps-epel7.xml` file to `/repo` as `comps-epel7.xml`, as follows:

    ```
    ~]# curl -o /var/www/html/repo/epel/comps-epel7.xml http://mirror.kinamo.be/epel/7/x86_64/repodata/xxxxxxxxxxxxxxxxxxxx-comps-epel7.xml

    ```

You will need to replace the multiple `x`'s with the correct MD5 hash, as found in the `repodata` folder.

1.  Next, synchronize the EPEL repository by executing the following (this may take a very long time, depending on your hardware and internet speed):

    ```
    ~]# reposync --repoid=epel --norepopath –download_path=/var/www/html/repo/epel/7/x86_64

    ```

2.  Create the local repository (again, depending on your hardware, this may take a long time), as follows:

    ```
    ~]# cd /var/www/html/repo/epel/7/x86_64
    ~]# createrepo --groupfile=/var/www/html/repo/epel/comps-epel7.xml .

    ```

3.  Finally, test your repository by executing the following command:

    ```
    ~]# curl http://localhost/repo/epel/7/x86_64/repodata/repomd.xml

    ```

## There's more…

When synchronizing RHEL 7 repositories, you will only be able to sync those you have entitlement to. To find out what entitlements you have on a given system connected to RHN, execute the following:

```
~]# cd /etc/yum/pluginconf.d/ && echo *.conf | sed "s/rhnplugin.conf//"|sed 's/\([0-9a-zA-Z\-]*\).conf/--disableplugin=\1/g'|xargs yum repolist && cd - >/dev/null

```

Whenever you synchronize a repository, try to keep the same directory structure as the original. I have found that it makes life easier when you want to rewrite your `/etc/yum.repos.d` files.

In an enterprise, it is useful to have a point in time when you "freeze" your yum repositories to ensure that all your systems are at the same RPM level. By default, any repository is "live" and gets updated whenever a new package is added. The advantage of this is that you always have the latest version of all packages available; the downside is that your environment is not uniform and you can end up troubleshooting for different versions of the same package.

The easiest way to achieve a "frozen" repository is to create a central location that holds all the RPMs as you would a normal yum mirror or copy.

Every `x` time, which you predefine, create a new directory with a timestamp, in which you hard link all the RPMs you mirror. Then finally, create a hard link to the directory, which you will later use in your repo configuration.

Here's an example:

| Directories | Description |
| --- | --- |
| `/rhel7/x86_64.all` | This directory contains a mirror which is synced nightly. RPMs are added, never deleted. |
| `/rhel7/x86_64.20150701` | This directory contains hard links to the RPMs in `/rhel7/x86_64`, all of which were synced on 01/07/2015, along with monthly iterations of the `/rhel6/x86_64.20150701` directory. |
| `/rhel7/x86_64` | This directory contains a hard link to the monthly iteration, which is deemed in production. |

Of course, you need to ensure that you create a repository for each new sync!

## See also

Refer to the *createrepo(8)* man pages for more information about creating a repository.

Also, refer to the *reposync(1)* man pages for more information on keeping your repository up-to-date.

# Configuring additional repositories

Whether you create your own mirror repository or organizations provide software for you in repositories, setting up additional repositories on your RHEL system is quite simple. This recipe will show you how to set them up. Many repositories have their own repo files or even an RPM that automatically installs the repository. When these are available, don't hesitate to use them!

## Getting ready

For this to work, you will need to have a repository set up, which can be accessed through the following URL: `http://repo.example.com/myrepo/7/x86_64`.

## How to do it…

In order to create an additional repository, create a file in `/etc/yum.repos.d` called `myrepo.repo`, which contains the following information:

```
[myrepo]
name=My Personal Repository
baseurl=http://repo.example.com/myrepo/$releasever/$basearch
gpgcheck=0
enabled=1
```

## There's more…

The `gpgcheck=1` option only functions if you or the provider of a repo has signed all the RPMs in the repo. This is generally a good practice and provides extra security to your repositories.

The `$releasever` and `$basearch` variables allow you to create a single repository file that can work on multiple systems as long as you have a repository for the URLs. The `$releasever` variable expands to the major version of the OS (7 in our case), and the `$basearch` will expands to x86_64\. On an i386 system (RHEL 7 only comes in the x86_64 architecture), `$basearch` expands to i386.

You can find many repositories on the Internet, such as `epel` and `elrepo`, but it may not always be a good idea to use them. Any software provided by the Red Hat standard repositories are also supported by Red Hat, and they will no longer support you if you start using the same software provided through another repository. So, you better ensure that you don't care about support or have another party that is willing to support you.

## See also

Although I do not condone the use of these in production without taking the appropriate support actions, here is a list of some popular repositories that you can use:

The ELRepo repository can be found at:

[http://elrepo.org/tiki/tiki-index.php](http://elrepo.org/tiki/tiki-index.php)

The EPEL repository is at:

[https://fedoraproject.org/wiki/EPEL](https://fedoraproject.org/wiki/EPEL)

The Puppetlabs repositories can be found at:

[https://docs.puppetlabs.com/guides/puppetlabs_package_repositories.html](https://docs.puppetlabs.com/guides/puppetlabs_package_repositories.html)

The Zabbix repositories are at the following link:

[https://www.zabbix.com/documentation/2.0/manual/installation/install_from_packages](https://www.zabbix.com/documentation/2.0/manual/installation/install_from_packages)

For the RepoForge repositories, refer to the following website:

[http://repoforge.org/use/](http://repoforge.org/use/)

Remi's repositories can be found at:

[http://rpms.famillecollet.com/](http://rpms.famillecollet.com/)

The Webtatic repositories are at:

[https://webtatic.com/projects/yum-repository/](https://webtatic.com/projects/yum-repository/)

# Setting up yum to automatically update

In enterprises, automating the systematic updating of your RHEL systems is very important. You want to stay ahead of hackers or, in general, people trying to hurt you by exploiting the weaknesses in your environment.

Although I do not recommend applying this recipe to all systems in an enterprise, this is quite useful to ensure that certain systems are kept up to date as the patches and bugfixes are applied to the RPMs in Red Hat's (and other) repositories.

## Getting ready

In order for this recipe to work, you'll need to be sure that the repositories you are using are set up correctly and you have valid mail setup (using Postfix or Sendmail, for example).

## How to do it…

We'll set up yum to autoupdate your system once a week (at 03:00 ) and reboot if necessary through the following steps:

1.  Install the yum cron plugin, as follows:

    ```
    ~]# yum install -y yum-cron

    ```

2.  Then, disable the hourly and daily yum cron jobs through the following commands:

    ```
    ~]# echo > /etc/cron.dhourly/0yum-hourly.cron
    ~]# echo > /etc/cron.daily/0yum-daily.cron

    ```

3.  Create the configuration file for the weekly yum update cron job via the following:

    ```
    ~]# cp /etc/yum/yum-cron.conf /etc/yum/yum-cron-weekly.conf

    ```

4.  Modify the created configuration file to apply updates and send a notification through e-mail by setting the following values:

    ```
    apply_updates = yes
    emit_via = email
    email_to = <your email address>

    ```

5.  Next, create a weekly cron job by adding the following contents to `/etc/cron.weekly/yum-weekly.cron`:

    ```
    #!/bin/bash 

    # Only run if this flag is set. The flag is created by the yum-cron init
    # script when the service is started -- this allows one to use chkconfig and
    # the standard "service stop|start" commands to enable or disable yum-cron.
    if [[ ! -f /var/lock/subsys/yum-cron ]]; then
     exit 0
    fi

    # Action!
    exec /usr/sbin/yum-cron /etc/yum/yum-cron-weekly.conf
    if test "$(yum history info |egrep '\skernel'|wc -l)" != "0"; then

    /sbin/shutdown --reboot +5 "Kernel has been upgraded, rebooting the server in 5 minutes. Please save your work."
    fi

    ```

6.  Finally, make the cron job executable by executing the following command:

    ```
    ~]# chmod +x /etc/cron.weekly/yum-weekly.cron

    ```

## How it works…

By default, `yum-cron` sets up a cron job that is run every hour (`/etc/cron.dhourly/0yum-hourly.cron`) and every day (`/etc/cron.daily/0yum-daily.cron`).

## There's more…

This recipe will upgrade all your packages when there's an update available. If you just want to apply security fixes, modify the `update_cmd` value of your yum cron configuration file in the following way:

```
update_cmd = security
```

Alternatively, you can even use the following configuration if you only want critical fixes:

```
update_cmd = security-severity:Critical
```

## See also

Check the *yum cron(8)* man page or the default `yum-cron.conf` file located at `/etc/yum/yum-cron.conf` for more information.

# Configuring logrotate for yum

Every time you use yum to install and/or update packages, it logs to `/var/log/yum.log`. A lot of people don't want to rotate the file a lot as they believe (incorrectly) that it is their only source to the history of their yum tasks. They may even believe that it provides a way to restore your rpm database if it gets corrupted - it does not.

I do recommend keeping your complete yum history as it doesn't grow a lot, unless you reinstall packages a lot.

For a rich interface to your yum history, I suggest you use yum history.

By default, your yum log file is rotated yearly, and even then, it only rotates if the size of your log file exceeds 30 KB, and your logs are only kept for 4 years. Usually, this is enough in the physical world as physical servers tend to be replaced every 3-4 years. However, virtual servers have the potential to stay "alive" beyond these 3-4 years.

## How to do it…

Modify `/etc/logrotate.d/yum` to the following:

```
/var/log/yum.log {
    missingok
    notifempty
    size 30k
    rotate 1000
    yearly
    create 0600 root root
}
```

## How it works…

This configuration will only rotate the yum log when it exceeds 30 KB in size on a yearly basis, and it will keep 1000 rotated logs, which is basically log files for 1000 years!

## See also

For more information on how to use and configure logrotate, refer to the *logrotate(8)* man page.

# Recovering from a corrupted RPM database

Although everything is done to ensure that your RPM databases are intact, your RPM database may become corrupt and unuseable. This happens mainly if the filesystem on which the `rpm db` resides is suddenly inaccessible (full, read-only, reboot, or so on).

This recipe will show you the two ways in which you can attempt to restore your RPM database.

## Getting ready

Verify that your system is backed up in some way.

## How to do it…

We'll start with the easiest option and the one with the highest success rate in these steps:

1.  Start by creating a backup of your corrupt `rpm db`, as follows:

    ```
    ~]# cd; tar zcvf rpm-db.tar.gz /var/lib/rpm/*

    ```

2.  Remove stale lock files if they exist through the following command:

    ```
    ~]# rm -f /var/lib/rpm/__db*

    ```

3.  Now, verify the integrity of the `Packages` database via the following:

    ```
    ~]# /usr/lib/rpm/rpmdb_verify /var/lib/rpm/Packages; echo $?

    ```

    If the previous step prints `0`, proceed to Step 7.

4.  Rename the `Packages` file (don't delete it, we'll need it!), as follows:

    ```
    ~]# mv /var/lib/rpm/Packages  /var/lib/rpm/Packages.org

    ```

5.  Now, dump the `Packages db` from the original `Packages db` by executing the following command:

    ```
    ~]# cd /usr/lib/rpm/rpmdb_dump Packages.org | /usr/lib/rpm/rpmdb_load Packages

    ```

6.  Verify the integrity of the newly created `Packages` database. Run the following:

    ```
    ~]# /usr/lib/rpm/rpmdb_verify /var/lib/rpm/Packages; echo $?

    ```

    If the exit code is not `0`, you will need to restore the database from backup.

7.  Rebuild the `rpm` indexes, as follows:

    ```
    ~]# rpm -vv --rebuilddb

    ```

8.  Next, use the following command to check the `rpm db` with yum for any other issues (this may take a long time):

    ```
    ~]# yum check

    ```

9.  Restore the SELinux context of the `rpm` database through the following command:

    ```
    ~]# restorecon -R -v /var/lib/rpm

    ```

    ![How to do it…](img/00059.jpeg)

## There's more…

If, for some reason, you are unable to recover your RPM database, there is one final option left. Enterprises tend to have standardized builds, and many servers are installed with the same packages, so copy the healthy `/var/lib/rpm` directory from another server with the exact same package set to the corrupted one, and perform the preceding recipe's steps to ensure that everything is okay.

Although you'll find additional tools that can save your skin (such as RPM cron), it's usually more practical to have a decent backup.