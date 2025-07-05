# Chapter 9. Aligning SELinux with DAC

In this chapter, our focus will be on the following set of recipes:

*   Assigning a different root location to regular services
*   Using a different root location for SELinux-aware applications
*   Sharing user content with file ACLs
*   Enabling polyinstantiated directories
*   Configuring capabilities instead of setuid binaries
*   Using group membership for role-based access
*   Backing up and restoring files
*   Governing application network access

# Introduction

SELinux is an access control mechanism that works alongside the regular access controls that Linux provides. Making sure that these various access control systems play nicely together is important as both have their merits and uses.

Regular DAC security services on Linux are already quite powerful and are being extended with almost every Linux release. Namespaces, extended access controls, additional **chroot** restrictions, and other services are added to the Linux ecosystem to support the hardening of Linux systems further.

In this process of hardening systems, SELinux is just another layer of defense. Putting all efforts only on SELinux would be a major mistake to make, as SELinux has its downsides as well. By properly enabling the Linux DAC controls and tweaking SELinux so that it plays nicely together with these controls, a Linux system can be made much more resilient against vulnerabilities and attacks.

# Assigning a different root location to regular services

A different root location, also known as achroot, is an important feature of Linux systems meant to disallow direct access to file resources outside a specified directory location. The environment that is accessible from a chroot is called a **jail** or **chroot jail**. Applications in a chroot jail are launched with a different root, wherein only those files that are needed for the application to work are hosted.

Although it is commonly seen as a security feature, this was not the intention of a chroot. However, with the proper approach, chroots can enhance the secure setup of an application.

For instance, in case of a vulnerability, a successful exploit might only be able to access the files available in the chroot. Other sensitive files, such as authentication-related files or other service configurations, are not reachable from within the chroot (assuming the exploited application does not have the privileges to break out of a chroot jail).

The steps to set up a chroot environment for any service are similar, but the end result of a chroot is never the same: different files need to be available in a chroot depending on the application that is being restricted.

## Getting ready

Find the application that needs to be restricted. Such applications have to be end services, in the sense that there is little to no interaction between the application and other applications or services. Otherwise, all those other applications and services would need to be available in the same chroot as well.

Usually, the primary targets are those services that are very popular in use on the Internet. Exploits for these services are usually more actively searched and developed for, and when a vulnerability is found and an exploit has been developed, malicious users or groups quickly scan the Internet for vulnerable versions to attack.

## How to do it…

The next set of steps shows how to set up a chroot environment and inform SELinux about the chroot. We use the BIND DNS server as our example service and `/var/chroot/` as the chroot location:

1.  Create the chroot location and add in the necessary subdirectories:

    ```
    ~# mkdir -p /var/chroot/dev
    ~# mkdir -p /var/chroot/etc/bind
    ~# mkdir -p /var/chroot/var/bind/{sec,pri,dyn}
    ~# mkdir -p /var/chroot/var/{log,run}
    ~# chown root:named /var/chroot
    ~# chmod 750 /var/chroot
    ~# chown -R named:named /var/chroot/var/*

    ```

2.  Copy all the files that the application needs:

    ```
    ~# cp /etc/named.conf /var/chroot/etc/
    ~# cp /etc/localtime /var/chroot/etc/
    ~# cp -a /var/named/* /var/chroot/var/named/

    ```

3.  Create the device files that the application needs:

    ```
    ~# mknod /var/chroot/dev/null c 1 3
    ~# mknod /var/chroot/dev/random c 1 8
    ~# chmod 666 /var/chroot/dev/*

    ```

4.  As the BIND service knows about chroots, we do not need to copy its binaries and libraries to the chroot location. However, not all services support chroots out of the box. When this is the case, we need to copy the binaries and libraries as well.
5.  Now, relabel the files in the chroot so that they get the proper SELinux labels:

    ```
    ~# setfiles -r /var/chroot/ /etc/selinux/mcs/contexts/files/file_contexts /var/chroot/

    ```

6.  Launch the application with the proper options to enable the chroot support. Some Linux distributions already support chroot information for the BIND service. In general, it requires the `named` application to be launched with the `-t /var/chroot/` option. If the application does not support chroots out of the box, use the `chroot` command itself:

    ```
    ~# chroot /var/chroot/ su - named -c /usr/sbin/named

    ```

7.  If the application supports chroots out of the box, it might require the `chroot` capability. This is supported through the `sys_chroot` permission, granted through the following SELinux policy interface:

    ```
    corecmd_exec_chroot(named_t)
    ```

## How it works…

Setting up a chroot environment is usually a trial-and-error approach; although, for more popular services, many tutorials exist on the Internet that make setting up chroots a lot easier.

The basic approach to use is four-fold:

1.  Create the chroot location and directory structure.
2.  Install the necessary files and, if necessary, application binaries and libraries.
3.  Update the SELinux labels of the resources.
4.  Call the chroot binary or use the built-in `chroot` capabilities of the application.

When creating a chroot location, we need to make sure that the structure is similar to a real root location (that is, the `/` location); as for the application, it will see the filesystem as if this chroot location is the entire filesystem.

Which files to install is a different matter though, and having online resources to inform us what to do is a great help. But if these online resources are missing, then we can still find out which files are needed.

For instance, we can use the `ldd` or `scanelf` application:

```
~# ldd /usr/sbin/named
 linux-vdso.so.1
 liblwres.so.90 => /usr/lib64/liblwres.so.90
 libdns.so.100 => /usr/lib64/libdns.so.100
 libbind9.so.90 => /usr/lib64/libbind9.so.90
 libisccfg.so.90 => /usr/lib64/libisccfg.so.90
 libisccc.so.90 => /usr/lib64/libisccc.so.90
 libisc.so.95 => /usr/lib64/libisc.so.95
 libc.so.6 => /lib64/libc.so.6
 /lib64/ld-linux-x86-64.so.2

```

But in general, it is the trial-and-error approach that works the easiest. Just launch the application in the chroot, register its errors, and resolve them.

For SELinux, the important bit here is that the chroot should be labeled correctly. Consider `/var/chroot/etc/named.conf`, for instance. The SELinux policy will assume that this file is labeled `named_conf_t`. However, the location itself (`/var/chroot/etc/named.conf`) implies `var_t`, as `/var/` is `var_t` and there are no definitions for any of our defined location's subdirectories or files within.

The `setfiles` command allows us to relabel a location with a different root location, resulting in `/var/chroot/etc/named.conf` being labeled as if it was `/etc/named.conf`. However, take care that a system relabeling operation is followed by the `setfiles` command again as the SELinux configuration is not aware of this change in labeling.

Finally, the application itself needs to be launched inside the chroot or through its built-in chroot support. Applications that support chroots themselves can be tuned through their configuration files and start up options to make sure that they run in a chroot environment. If that isn't possible, then the application should be started using an `init` script that calls the `chroot` command, most likely together with the `su` application to allow switching to a different user.

## There's more...

A chroot is a relatively primitive yet powerful method for reducing the impact of an exploit. However, methods exist to escape a chroot. Luckily, there are some kernel patches that improve the security of chroots tremendously. A popular update is the one maintained by the **grsecurity** team ([http://www.grsecurity.net](http://www.grsecurity.net)).

With grsecurity's chroot restrictions, the kernel can be configured with the following options:

*   Disallow mounts and remounts of filesystems initiated from within the chroot
*   Disallow chrooting from within the chroot
*   Disallow the `pivot_root` call from within the chroot
*   Force the current working directory of chrooted applications to be the root directory of the chroot
*   Disallow the `setuid` and `setgid chmod` operations from within the chroot
*   Disallow changing directories through open file descriptors pointing outside the chroot
*   Disallow attaching to shared memory created outside the chroot
*   Disallow access to Unix domain sockets created outside the chroot
*   Disallow sending signals to processes outside the chroot

Besides these options, there are many, many more options. Such options make chroot jails much more security-oriented than originally intended and make for a very powerful mitigation against exploits.

## See also

There are many resources available about chroot jails and BIND chroots in particular:

*   Building and configuring BIND 9 in a chroot jail available at [http://www.unixwiz.net/techtips/bind9-chroot.html](http://www.unixwiz.net/techtips/bind9-chroot.html) goes in great detail and has pointers to various other BIND-related resources
*   On the same site, best practices for Unix `chroot()` operations can be found: [http://www.unixwiz.net/techtips/chroot-practices.html](http://www.unixwiz.net/techtips/chroot-practices.html)
*   The Jailkit project ([http://olivier.sessink.nl/jailkit/](http://olivier.sessink.nl/jailkit/)) provides a set of utilities to manage chroot jails

# Using a different root location for SELinux-aware applications

SELinux-aware applications have more requirements when they run inside a chroot location. They require access to the SELinux subsystem (from within the chroot) and possibly SELinux configuration entries. This includes PAM-enabled services, as user logins on these services might require access to the SELinux user configuration files (such as the `seusers` file and default contexts).

## How to do it…

First, create the regular chroot location as we saw earlier. To update the system to support SELinux-aware applications inside the chroot, complete the following steps:

1.  Mount the SELinux filesystem inside the chroot at `/sys/fs/selinux/` so that the application can query the SELinux policy:

    ```
    ~# mkdir -p /var/chroot/sys/fs/selinux
    ~# mount -t selinuxfs none /var/chroot/sys/fs/selinux

    ```

2.  Optionally, create the `/var/chroot/etc/selinux/` location and copy the current definition inside it:

    ```
    ~# cp -a /etc/selinux/ /var/chroot/etc/

    ```

3.  Update the `seusers` file (in `/var/chroot/etc/selinux/mcs/`) to only contain the SELinux user mapping(s) needed inside the chroot.

## How it works…

Applications that are SELinux-aware usually require access to the SELinux filesystem (`/sys/fs/selinux/`) and a kernel-provided pseudo filesystem needed in order to interact with the SELinux subsystem. This should be seen as a more dangerous situation, as this usually has the application run as a more privileged user and with access to a system resource that is not protected by the chroot anymore. This reduces the effectiveness of a chroot jail as a security measure.

If applications do not support chroots themselves internally, then we will have to expose the `/sys/fs/selinux/` filesystems to the application that is chrooted. If the application supports chroot out of the box, it might only call the chroot after consulting SELinux (that is, from the nonchrooted parent) and run the worker or user processes inside a chroot. This is the case with chrooted SFTP users supported through OpenSSH.

It might also be sufficient to mount the SELinux filesystem on `/selinux/` (a deprecated but still a supported location for the SELinux filesystem) inside the chroot. That way, no fake `/sys/fs/` location needs to be created:

```
~# mount -t selinuxfs none /var/chroot/selinux

```

The `/etc/selinux/` location is not always needed, so it shouldn't be made accessible inside the chroot by default. SELinux-aware applications that use SELinux user and role transitions or that actively modify file contexts will need to be able to read the files inside `/etc/selinux/` though.

Depending on the reason of the chroot jail, it might be possible as well to use a read-only bind-mount of the `/etc/selinux/` location:

```
~# mount -o bind /etc/selinux /var/chroot/etc/selinux
~# mount -o remount,ro /var/chroot/etc/selinux

```

The remount afterwards is needed to mark it as read-only. A bind-mount, by itself, doesn't allow additional mount options to be passed, so we cannot immediately mount with the `ro` mount option. Of course, it is no longer possible/needed to modify the `seusers` file with a read-only bind-mount.

## See also

*   Detailed guides on SFTP chroots can be found at [https://wiki.archlinux.org/index.php/SFTP_chroot](https://wiki.archlinux.org/index.php/SFTP_chroot) and [http://en.wikibooks.org/wiki/OpenSSH/Cookbook/SFTP](http://en.wikibooks.org/wiki/OpenSSH/Cookbook/SFTP)

# Sharing user content with file ACLs

Access control lists allow for more fine-grained access controls on files. Instead of using a common group ownership, access to files can be individually granted to users or groups.

However, the access controls that SELinux enables should also be tailored to this situation. Features such as the user-based access control constraints in SELinux might prevent sharing user content altogether, regardless of the ACLs set on the file.

## How to do it…

Assuming that a user wants to allow read and read-write accesses to a set of files and directories, the following set of steps can be used:

1.  Create an accessible location outside the user's home directory:

    ```
    ~# mkdir -p /home/share/
    ~# chmod 1777 /home/share/

    ```

2.  Create an SELinux file type that can be used for sharing resources:

    ```
    type user_share_t;
    files_type(user_share_t)
    ```

3.  Create an interface allowing users to administer the resource:

    ```
    interface(`userdom_admin_user_share','
      gen_require(`
        type user_share_t;
      ')
      admin_pattern($1, user_share_t)
     ')
    ```

4.  Assign this type to the new location:

    ```
    ~# semanage fcontext -a -t user_share_t "/home/share(/.*)?"
    ~# restorecon -R /home/share/

    ```

5.  Assign the interface to the user domain(s) that will participate in the shared development of this resource:

    ```
    userdom_admin_user_share(user_t)
    ```

6.  Move the files that need to be shared outside the user's home directory, as the SELinux context of the home directory will not allow sharing resources within.

    ```
    ~$ cp -r sharedfiles/ /home/share && rm -r sharedfiles/

    ```

7.  Assign the ACL that allows the (limited set of) users proper access:

    ```
    ~$ setfacl -R -m u:user1:rX /home/share/sharedfiles
    ~$ setfacl -R -m u:user2:rwX /home/share/sharedfiles
    ~$ setfacl -m "default:u:user2:rwX" /home/share/sharedfiles
    ~$ setfacl -m "default:u:user0:rwX" /home/share/sharedfiles
    ~$ setfacl -m "default:u:user1:rX" /home/share/sharedfiles

    ```

## How it works…

The file-level access controls can be perfectly used together with the SELinux access controls. However, special care needs to be taken that both control mechanisms (file ACLs and the SELinux policy) don't interfere with each other. SELinux might disallow accesses expected to work (for instance, due to SELinux constraints rather than type enforcement settings), but also file access controls need to be properly managed in order to keep the behavior on the system consistent.

In the recipes, the files that are shared are moved outside the user's home directory. This is mostly because of SELinux' UBAC feature, which disallows different SELinux users to access each others' regular resources (such as those labeled as `user_home_t` but also `user_home_dir_t`). As `user_home_dir_t` isn't accessible by other SELinux users under the UBAC constraints, users mapped to a different SELinux user will not be able to enter and search through the sharing user's home directory, regardless of ACLs being installed.

Not all systems have UBAC enabled, or the sharing might be within a single SELinux user, so this approach is not always necessary. Still, using a different location allows for better management. Consider the case where the first user exits the company, but his team wants to continue accessing and managing the shared resources. They would disappear if the user home directory is removed.

With the files moved to a different location, the next step is to label the files with a file type that all users can access, but which isn't restricted by the UBAC feature. File types that have the `ubac_constrained_type` attribute set cannot be used for sharing, so a new file type is created that is labeled as a regular file. The user domains are then granted administrative rights on this type (allowing them not only to manage the files, but also to relabel files to or from the `user_share_t` type). This ensures that SELinux doesn't prevent access to the shared resources, while still preventing unauthorized domains to access the resources.

It might also be sufficient to pick a file type that is already accessible by users, such as the `nfs_t` type (if the SELinux Boolean, `use_nfs_home_dirs`, is set). However, assigning a type that is functionally used for different reasons (`nfs_t` is for NFS-mounted filesystems) might open up access to these resources from other domains as well. As such, administrators need to carefully consider the reasons for and the consequences of each choice.

After labeling the `/home/share/` location with the `user_share_t` type, the original user copies the resources to the new location and removes them from the current one. This approach (copy and remove) is used to ensure that resources inherit the label of the target location (`user_share_t`) instead of keeping the labels associated with the original file location (`user_home_t`), as would be the case with a move (`mv`) command. In more recent `coreutils` packages, support for `mv -Z` is made available, which allows you to move the resources directly while still giving the resources a proper context.

A third approach for the user would be to move the resources first and then relabel them:

```
~$ mv sharedfiles/ /home/share/
~$ chcon -R -t user_share_t /home/share/sharedfiles/

```

Finally, with all SELinux rules and support in place, the file access controls are enabled on the shared resources, and a default ACL is enabled so that write operations by other users will automatically inherit the proper ACL on the written resource as well, making sure that all users cooperating on the shared resource don't need to continuously set ACLs on the files.

Without the default ACLs, other users might create files inside `sharedfiles/` that have no ACLs set, disallowing other users to access the resources.

## There's more...

Another approach that could be taken is to use the `setgid` group ownership. For instance, if all users that participate in the shared files access are in a `shrgrp` group, then the following will automatically have all files created inside the mentioned directory have the `shrgrp` group ownership defined as well:

```
~$ chgrp -R shrgrp /home/share/sharedfiles/
~$ find /home/share/sharedfiles/ -type d -exec chmod g+s '{}' \;

```

This does require the users to have a proper `umask` setting (such as `007` or less) so that the group permission on the newly created resource allows read and write accesses for group members.

# Enabling polyinstantiated directories

On Linux and Unix systems, the `/tmp/` and `/var/tmp/` locations are world writable. They are used to provide a common location for temporary files and are protected through the sticky bit so that users cannot remove files they don't own from the directory, even though the directory is world writable.

But despite this measure, there is a history of attacks against the `/tmp/` and `/var/tmp/` locations, such as race conditions with symbolic links and information leakage through (temporary or not) world or group-readable files generated within.

Polyinstantiated directories provide a neat solution to this problem: users get their own, private `/tmp/` and `/var/tmp/` instance. These directory instances are created upon login on a different location, but then made visible (mounted) on the `/tmp/` and `/var/tmp/` locations for that specific user session. This mount is local to the user session through the use of Linux namespaces—other users have their own view on the mounts, and for administrators, polyinstantiation is not enabled, so they keep a global view on the system.

## How to do it…

To enable polyinstantiation of `/tmp/` and `/var/tmp/`, the following steps should be followed:

1.  Create the `/tmp-inst/` and `/var/tmp/tmp-inst/` locations:

    ```
    ~# mkdir /tmp-inst/ /var/tmp/tmp-inst/
    ~# chmod 000 /tmp-inst/ /var/tmp/tmp-inst/

    ```

2.  Set the label for these locations as `tmp_t`:

    ```
    ~# semanage fcontext -a -t tmp_t -f d /tmp-inst
    ~# semanage fcontext -a -t tmp_t -f d /var/tmp/tmp-inst

    ```

3.  Edit `/etc/security/namespace.conf` and add in the following definitions:

    ```
    /tmp  /tmp-inst/    level  root,adm
    /var/tmp  /var/tmp/tmp-inst/  level  root,adm
    ```

4.  Edit the PAM configuration file used by logins, such as `system-login`, and add the following line to the session group after the `pam_selinux.so` one:

    ```
    session  required  pam_namespace.so
    ```

5.  Enable the `allow_polyinstantiation` SELinux Boolean:

    ```
    ~# setsebool -P allow_polyinstantiation on

    ```

## How it works…

The system preparation for polyinstantiated directories requires that the directories themselves are available and have the proper permissions set. When the parent directory, such as `/tmp/`, is a tmpfs mount, then we cannot have the polyinstantiated directories made available inside of it (such as `/tmp/tmp-inst/`), as that directory would be missing after a reboot (unless it is added through the `init` scripts); hence the setup of `/tmp-inst/` as a separate location. Of course, administrators can still opt to have this location itself as a tmpfs mount—the important thing is that the directory must exist and have the proper permissions (which is represented by the `000` permission set).

In the example, `/var/tmp/` is assumed not to be a tmpfs mount, so we can define the polyinstantiated directories inside of it.

The configuration file for polyinstantiated directories is the `namespace.conf` file under `/etc/security/`. In it, the mount-point is mentioned together with the directory in which the polyinstantiated directories are created:

```
/tmp  /tmp-inst/  level  root,adm
```

The third column defines the method for polyinstantiation. On non-SELinux systems, the most common method used is the `user` method, which creates directories based on the username. On SELinux-enabled systems, the method must be either `level` or `context`.

In case of the `level` method, the directories are created based on the username and MLS level of the user session. The `context` method has directories created based on the username and security context. This allows for hiding temporary data based on the role of the user, so accidental data leakage is less likely to occur.

Administrators can access the polyinstantiated directories as they are excluded from the polyinstantiation: the excluded list of users is configured as the fourth column in the `namespace.conf` file. Administrators can still see the directories that are created dynamically:

```
~# ls -l /tmp-inst/
drwxrwxrwt. 2 root root 4096 Jun 22 12:31 system_u:object_r:tmp_t:s0_user1
drwxrwxrwt. 2 root root 4096 Jun 22 12:30 system_u:object_r:tmp_t:s0_user2

```

Next, the PAM configuration file(s) are modified to enable the `pam_namespace.so` library. To find the PAM configuration files that need to be edited, look for the PAM configuration files that call `pam_selinux.so`:

```
~# cd /etc/pam.d
~# grep -l pam_selinux.so *
system-login

```

In this example, the `system-login` PAM configuration file is the only file calling `pam_selinux.so`, so the `pam_namespace.so` line is added to this file. The line must be added after the `pam_selinux.so` call as the `pam_namespace.so` file uses the context of the user to decide how to call the instantiated directory. If `pam_selinux.so` has not been called yet, then this information is not available and the logon will fail.

Finally, the SELinux Boolean, `allow_polyinstantiation`, is enabled so that the proper domains have the privilege to create (and change the contexts of) the proper directories, to use namespaces, to check user context, and more.

## There's more...

Administrators can go further than just having the directories created when needed. During the setup of polyinstantiated directories, a script called `namespace.init`, which is available at `/etc/security/` is called to further handle the creation and modification of those directories.

This script can be adjusted to copy files towards the instantiated directory (the file usually contains this logic already for polyinstantiated home directories) or do other changes, allowing to further tune the setup for a user session.

The `systemd init` system also has support for polyinstantiated `/tmp/` directories through the `PrivateTmp` directive, which provides a private `/tmp/` directory for a service rather than end users.

# Configuring capabilities instead of setuid binaries

Linux capabilities allow for course-grained kernel security authorizations on the user and application levels. Before capabilities existed, administrators could only grant additional privileges to users through `setuid` applications: applications which, when executed, inherit the privileges of the owner of the application (usually, `root`). With capabilities, the set of privileges can be restricted further.

For instance, the `ping` application can be granted the `cap_net_raw` capability, so it does not need to be `setuid` anymore. Depending on the setup, either users need to be granted the possible use of the capability (if the application has the proper flag set) or the capability is granted immediately (regardless of user settings).

## How to do it…

To use capabilities with SELinux, execute the following steps:

1.  Enable the capabilities that are needed for an application on the application binary:

    ```
    ~# setcap cap_net_raw+ei /bin/ping

    ```

2.  For the users that are allowed to use the `net_raw` capability, add the proper configurations in `/etc/security/capability.conf` (one line per user):

    ```
    cap_net_raw   user1
    ```

3.  SELinux domains that will use the capability need to be granted the use of it. For common applications, this is usually already in place.

    ```
    allow ping_t self:capability net_raw;
    ```

4.  SELinux domains that are allowed to modify the capability set assigned to their process(es) must have the `setcap` privilege set:

    ```
    allow local_login_t self:process setcap;
    ```

5.  Edit the PAM configuration file(s) for the services through which the capabilities are allowed, and add the following line to the `auth` configuration block:

    ```
    auth  required  pam_cap.so
    ```

6.  If capabilities need to be tracked/audited, SELinux's `auditallow` statement can be used:

    ```
    auditallow domain self:capability net_raw;
    ```

## How it works…

The capabilities that a process is currently allowed to use are called the permitted capabilities. The capabilities that are active are the effective capabilities. A third set of capabilities are inheritable capabilities.

In the example, we enabled the `cap_net_raw` capability for the `ping` application and marked the capability as effective if it is inherited. In other words, it is not enabled (permitted) by default. If we want to enable the `cap_net_raw` capability immediately, we would use the effective and permitted set:

```
~# setcap cap_net_raw+ep /bin/ping

```

Applications that are capability-aware do not need to have the `effective` bit set. They will enable (and drop) the capabilities as they are needed through the proper system calls (which is why the `setcap` permission is needed for these domains). If `ping` was capability-aware, then the following would be sufficient for our example:

```
~# setcap cap_net_raw+i /bin/ping

```

Next, the users that are allowed the `cap_net_raw` capability (through the selected set of applications) need to be granted the `cap_net_raw` capability in their inherited capability set. This is done through the `capability.conf` file in `/etc/security/` and by calling the `pam_cap.so` module from within the proper PAM configuration files. The use of PAM configuration files also allows us to differentiate capabilities based on the service through which a user logs on.

To check the currently enabled capabilities, users can execute the `capsh` application:

```
~$ /sbin/capsh --print | grep ^Current
Current: cap_net_raw+i

```

To see the capabilities on a file, the `getcap` application can be used:

```
~$ getcap /bin/ping
/bin/ping = cap_net_raw+ei

```

Finally, auditing the use of capabilities through the `auditallow` statement tells us when (and by whom) a capability was used, although the same can be accomplished without an SELinux policy using the Linux audit subsystem, auditing for the `setcap` system call.

## See also

*   Capabilities are well explained in Chris Friedhoff's **POSIX Capabilities & File POSIX Capabilities** page ([http://www.friedhoff.org/posixfilecaps.html](http://www.friedhoff.org/posixfilecaps.html))

# Using group membership for role-based access

In larger environments, access controls are usually granted based on group membership. Group membership is easier to manage than individual permissions: just adding or removing users from a group automatically grants or revokes permissions, and administrators can easily find out which permission(s) a user will have based on the group membership.

## How to do it…

In order to use group membership as a high-level method for assigning permissions, administrators need to take care of the following aspects:

1.  Add user(s) to the groups they should belong to:

    ```
    ~# gpasswd -a user1 dba
    ~# gpasswd -a user1 dev

    ```

2.  Assign the proper SELinux user to the group:

    ```
    ~# semanage login -s dbadm_u %dba

    ```

3.  Restrict binaries and libraries that should only be called by a specific group:

    ```
    ~# chgrp -R dev /usr/lib/gcc /usr/x86_64-pc-linux-gnu/gcc-bin
    ~# chmod -R o-rx /usr/lib/gcc /usr/x86_64-pc-linux-gnu/gcc-bin

    ```

4.  Use group notation inside the `sudoers` file to grant specific privileges to group members:

    ```
    %dba  ALL=(ALL)  TYPE=dbadm_t ROLE=dbadm_r NOPASSWD: initdb
    ```

## How it works…

Using groups makes permission handling much easier. In the end, this allows administrators to just handle group membership for users and automatically assign privileges based on the groups.

We can grant groups an SELinux user, and through the group membership decide which SELinux user a regular user is logged into. Of course, users can belong to multiple groups. For SELinux, it is the order of the `seusers` file that decides which of the following mappings are used:

*   SELinux user mappings for individual users take precedence over group mappings
*   The first group mapping in the `seusers` file that uses a group that the Linux user is a member of decides the SELinux user mapping if no individual SELinux user mappings exist for this user

As such, if a user is a member of two groups (say, `dba` and `web`) and there are mappings to both `dbadm_u` (for the `dba` group) and `webadm_u` (for the `web` group), then the first mapping in the `seusers` file will decide what the user's SELinux user will be.

In order to override this, either add the user individually or create another group (say, `dbaweb`), grant the user this group as well, and put that group mapping at the beginning of the list in the `seusers` file.

When only a specific user group is allowed access to an application, but that application does not use any specific SELinux domains, then it might be more flexible for administrators to use the Linux DAC permissions to restrict access to the application. By only allowing a specific group (`dev`, in our example), read and execute rights on the application and application libraries, we can restrict access easily.

Another approach is to label the files with new SELinux types and grant the proper domains access to those types. However, this might lead to a large set of domains needing access to the types (and so requires massive policy development effort), whereas the Linux DAC approach is easily implemented.

# Backing up and restoring files

An important aspect to the availability of a system and the security of a service is to provide backup and restore services. For many, having a copy of the files available might seem sufficient as a backup approach. However, backups should contain more than just the content of a file.

## How to do it…

When selecting a backup solution, make sure to check for the following:

1.  A selection of the extended attributes of the files should be backed up as well (and not only the `security.selinux` one).
2.  When files are restored onto their original location, the SELinux context should be restored with it as well. If the backup solution doesn't support SELinux contexts, the `restorecon` command should be invoked afterwards against the restored file(s).
3.  When files are restored into a temporary area, the SELinux context should not be restored. Instead, the administrator should put the file back in place and restore the context afterwards.
4.  The SELinux configuration in `/etc/selinux/` should definitely be backed up, even if no full system backups are used. Whenever the policy or file context definitions are altered, these should be backed up as well whenever files are backed up.

## How it works…

File labels are stored as the `security.selinux` extended attribute. As the functioning of a policy is based on the labels of all objects involved, not backing up and restoring the file labels might jeopardize the functioning of the system after a restore operation.

When the backup solution does not support extended attributes, it is important that all labels are properly set through the `semanage fcontext` command. This is the only way to make sure that, after a restore, the admin can run `restorecon` against the restored files in order to reset the file labels:

```
~# tar xvf /path/to/last_backup.tar.gz etc/named.conf
~# restorecon /etc/named.conf

```

However, it is seriously recommended to select a backup solution that supports extended attributes as many other Linux-related settings are stored as extended attributes. The file ACLs, for instance, are stored as extended attributes as well:

```
~$ getfattr -m . -d named.conf
# file: named.conf
security.selinux="system_u:object_r:named_conf_t:s0"
system.posix_acl_access=0sAgAAAAEABgD/////AgAGAOo…

```

Other examples of extended attributes that can be used on a system are PaX markings (`user.pax.flags`), IMA and EVM hashes (`security.ima` and `security.evm`), and capabilities (`security.capability`). But herein lies the problem as well: some attributes shouldn't (or cannot) be restored. The IMA and EVM attributes, for instance, are handled by the Linux kernel and cannot be manipulated by user utilities.

Alongside the file labels, backing up and restoring the SELinux policy should be integrated as well, especially on a system with a modified SELinux policy. If a policy is different after a restore, then types might be missing and labels might become invalid.

# Governing application network access

On Linux systems, `iptables` (and more recently, `nftables`) is the de facto host-based firewall technology. Administrators will undoubtedly use it to prevent access to a service from unauthorized systems. We can also use `iptables` to identify and label network packets, allowing only authorized applications (domains) to send or receive those network packets.

By default, the SELinux policy supports client and server packets and allows the usual domains access to their client and/or server packets. For instance, the web server domains (such as `httpd_t`) will have the privileges to send and receive `http_server_packet_t` packets:

```
allow httpd_t http_server_packet_t:packet { send recv };
```

This is provided through the `corenet_sendrecv_http_server_packets` interface. Enabling packet labeling is simply done using `iptables` as will be shown through this recipe. But to properly govern network access, custom packet types will need to be created to ensure that no default allowed access is used.

## How to do it…

To only allow authorized domains access to particular network packets (datagrams and data streams), use the following approach:

1.  Identify the flow that needs to be allowed. For instance, we might only want DNS requests from `10.11.12.0/24` to be accepted by the `dnsmasq_t` domain, and requests from `10.13.14.0/24` to be accepted by the `named_t` domain.
2.  Create two new packet types:

    ```
    type dnsmasq_server_packet_t;
    corenet_server_packet(dnsmasq_server_packet_t)

    type named_server_packet_t;
    corenet_server_packet(named_server_packet_t)
    ```

3.  Allow the domains send and receive privileges for these packets:

    ```
    allow dnsmasq_t dnsmasq_server_packet_t:packet { send recv };
    allow named_t named_server_packet_t:packet { send recv };
    ```

4.  Label the incoming traffic accordingly:

    ```
    ~# iptables -t mangle -A INPUT -p tcp -s 10.11.12.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:dnsmasq_server_packet_t:s0"
    ~# iptables -t mangle -A INPUT -p udp  -s 10.11.12.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:dnsmasq_server_packet_t:s0"
    ~# iptables -t mangle -A INPUT -p tcp -s 10.13.14.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:named_server_packet_t:s0"
    ~# iptables -t mangle -A INPUT -p udp -s 10.13.14.0/24 --dport 53 -j SECMARK --selctx "system_u:object_r:named_server_packet_t:s0"

    ```

## How it works…

By using custom network packet labels, access from or to specific applications can be governed using an SELinux policy. Even though multiple applications can accept incoming DNS requests, this recipe shows how to ensure that only one application can deal with requests that have passed a certain filter.

Whenever a SECMARK label is enabled with `iptables`, the Linux kernel will automatically enable SECMARK labeling on all packets. Packets that are not marked specifically by the administrator will be marked with the `unlabeled_t` type. Some domains are allowed to handle the `unlabeled_t` packets through the `corenet_sendrecv_unlabeled_packets` interface (or the `kernel_sendrecv_unlabeled_packets` interface). However, if that is not the case, then those domains will not be able to handle network traffic anymore.

As such, it is advised to use the standard labeling for other incoming (and outgoing) traffic. To identify which incoming traffic should be labeled, we can leverage assistance from the `netstat` output:

```
~# netstat -naptZ | awk '/LISTEN/ {print $4,$6,$7,$8}'
0.0.0.0:13500 LISTEN 6489/mysqld system_u:system_r:mysqld_t:s0
0.0.0.0:80 LISTEN 23303/httpd system_u:system_r:httpd_t:s0
10.11.12.122:53 LISTEN 4432/dnsmasq system_u:system_r:dnsmasq_t:s0
10.13.14.42:53 LISTEN 5423/named system_u:system_r:named_t:s0

```

Based on this output, labeling the appropriate traffic as `mysqld_server_packet_t` and `http_server_packet_t` will allow those domains to access their incoming network traffic.

By creating additional types for `dnsmasq_t` and `named_t`, those applications can only handle requests associated with those packet types. If an administrator changes the configuration of one of these DNS servers, then the network packet labeling will still ensure that DNS requests from the previously identified network segments cannot be used by the wrong DNS server, even though the flow is allowed firewall-wise.

With `sesearch`, interrogating the policy to see which applications (domains) are able to send and receive certain packets is easy:

```
~# sesearch -t dns_server_packet_t -ACTS
Found 10 semantic av rules:
 allow nova_network_t dns_server_packet_t : packet { send recv } ;
 allow corenet_unconfined_type packet_type : packet { send recv relabelto flow_in flow_out forward_in forward_out } ;
 allow named_t dns_server_packet_t : packet { send recv } ;
 allow vmware_host_t server_packet_type : packet { send recv } ;
 allow dnsmasq_t dns_server_packet_t : packet { send recv } ;
 allow kernel_t packet_type : packet send ;
 allow iptables_t packet_type : packet relabelto ;
ET allow squid_t packet_type : packet { send recv } ; [ squid_connect_any ]
DT allow icecast_t packet_type : packet { send recv } ; [ icecast_connect_any ]
DT allow git_session_t server_packet_type : packet { send recv } ; [ git_session_bind_all_unreserved_ports ]

```

The same approach can be taken from a client level. A mail server might need to connect to other mail servers, which means that the outgoing data can be labeled as `mail_client_packet_t` (if we use the default traffic). However, if we want to make sure only the mail server can connect to other mail servers (and no other domains that also have privileges to send and receive the `mail_client_packet_t` packets), then a new packet type can be used.

## See also

For more information about SECMARK labeling, read up on the following resources:

*   [http://www.selinuxproject.org/page/NB_Networking](http://www.selinuxproject.org/page/NB_Networking)
*   Paul Moore's **Transitioning to Secmark** at [http://paulmoore.livejournal.com/4281.html](http://paulmoore.livejournal.com/4281.html)
*   James Morris's **New Secmark-based network controls for SELinux** at [http://james-morris.livejournal.com/11010.html](http://james-morris.livejournal.com/11010.html)