# Chapter 10. Security Central

Linux security is not a chapter in a book, it is a way of life. For every task that you carry out in CentOS, you should consider the security impact and how your actions can be made more secure. Of course, security takes many forms, and much of it is quite simple (for example, the physical security of servers locked in server rooms and so on. In this section, we will visit the world of **pluggable authentication modules** (**PAM**) and **SELinux**. Do not be scared of either, especially, SELinux; they are your friends. In this chapter, we will cover the following topics:

*   **Understanding PAM configuration files**: At the heart of PAM are the files located in `/etc/pam.d`; I will lead you through their syntax and meaning
*   **Limits of PAM**: Through PAM's limits module, we can restrict limits on system resources that can be obtained in a user session
*   **SELinux**: This is a quick guide to SELinux, both through the filesystem and the user's perspective
*   **Hardening Linux**: This is a checklist on what and how to secure your Linux server

# Understanding PAM configuration files

Rather than building authentication into each and every application that needs it, most services in Linux will make use of modules such as PAM. These modules have the `.so` extension to identify them as standard modules, and are used by programs rather than the kernel directly. They make their home in `/lib/security` or `/lib64/security`, depending on whether your system is 32 bit or 64 bit. Each service or program that has PAM capabilities has its own configuration that dictates how authentication and session settings will be enforced; these files are located in `/etc/pam.d`. A quick look in this folder will reveal some recognizable names, such as `sshd`, `sudo`, `su`, and `login`, all representing services that have a level of authentication associated with them.

The `/etc/pam.d/login` file will be used by the console login program and `/etc/pamd/sshd` by the OpenSSH server. There will be multiple lines in each configuration file, and every line will have settings for type, control, module-path, and optional arguments to the module.

From the following figure, you can gain some understanding of the file syntax and see the four type settings: account, auth, session, and password.

![Understanding PAM configuration files](img/5920OS_10_01.jpg)

Take the following line as an example:

```
account  required  pam_nologin.so
```

This line has been taken from the `/etc/pam.d/login` file and has been set for the `account` type with the control set to `required` for the specified module: `pam_nologin.so`. This particular module checks for the existence of the `/etc/nologin` file; if it does exist, then even on successful authentication the user will not gain access to the system. He or she will be shown either the contents of the `/etc/nologin` file or a generic message explaining that login could not occur. This approach in itself becomes a simple way to temporarily disable logins to a server by just creating the `nologin` file; delete the file to re-enable logins.

Each of the elements of a PAM configuration file are listed next.

## Type

Type defines when the module will be invoked. There are 4 valid entries:

*   **Account**: This is typically used to restrict or permit access to a service based on the time of day, maximum users, concurrent logins, and so on. Think of these as account restrictions.
*   **Auth**: This is used to establish who the user is, most often, by prompting for a password. This module can also be used to grant group membership.
*   **Password**: This type is used when updating a user's authentication token; again, most commonly, the password. Typically, one module will need to be loaded for each challenge and response authentication method.
*   **Session**: This is associated with events that need to be complete before a user is given access to the server, such as creating a home directory, if required.

## Control

The control field indicates the behavior of the PAM-API based on success or failure returned by the module.

*   **required** (`success=ok new_authtok_reqd=ok ignore=ignore default=bad`): Failure will ultimately lead to the API returning `failure`, but only after the remaining modules have been invoked.
*   **requisite** (`success=ok new_authtok_reqd=ok ignore=ignore default=die`): Like required, except control and failure is immediately returned to the calling service.
*   **sufficient** (`success=done new_authtok_reqd=done default=ignore`): Success of a module with this type is enough to satisfy the authentication requirements unless a previously required module has failed. A failure of a module of this type is not fatal.
*   **optional** (`success=ok new_authtok_reqd=ok default=ignore`): Success or failure of this module is not important.

### Tip

Recently, the second notation (within the brackets) is referred to for the control value, which is accepted to be a better explanation of the meaning.

## The module path

The module path will consist of either the full filesystem path to the referenced module and the path would begin with the `/` filesystem or a relative to the `/lib/security` or `/lib64/security` directory.

## Module arguments

The module arguments are a space delimited list of tokens that can modify the behavior of a module. The following is an example:

```
silent umask=077 skel=/etc/skel
```

We could put arguments together and use an example whereby we create a user's home directory for them on first login. This can be useful where we create many accounts in one go and do not want to overload the server creating the home directories at the same time. This is common for universities and so on where many accounts are created in batches.

Users would most probably connect to the server using SSH, so we should consider adding the following line to the `/etc/pam.d/sshd` file:

```
session  required  pam_mkhomedir.so  umask=0077 skel=/etc/skel
```

This mechanism utilizes PAM to create the home directory at user login.

# Limits of PAM

Let's stick with using the SSH login at the moment. Many users will only access the server via SSH, perhaps using the PuTTY SSH client on Windows. If we want to control access to system resources, then we can implement restrictions using PAM and `pam_limits.so`. We should add the following line to the `/etc/pam.d/sshd` file:

```
session required pam_limits.so
```

This will implement the module, however, we still have to set the restrictions in the `/etc/security/limits.conf` file; the module reads from this file. The file's structure is set as follows with these elements making up a line in the limits file:

```
<domain> <type> <item> <value>
```

## Domain

Domain represents to whom the limit is intended. This, most often, is a username such as `user1` or a group entry such as `@users`; the `@` symbol differentiates between user and group names. To implement a default restriction to apply to all accounts that do not have their own entry is the wildcard `*`.

## Type

Type can be set to `soft`, `hard`, or both with `-`. Use the `hard` restriction for enforcing hard resource limits. These limits are set by the root user and enforced by the kernel. The user cannot raise their requirement of system resources above such values. Be cautious when enforcing restrictions before adequately testing, as it is possible to bring a system to a halt if enough processes cannot be spawned.

The `soft` restriction is used to enforce soft resource limits. These limits are the ones that the user can move up or down within the permitted range with any pre-existing hard limits. The values specified with this token can be thought of as default values for normal system usage. Use of the `-` symbol is to enforce both soft and hard resource limits together.

## Item

The item represents the system resource on which the restriction is being made. Command items that can be restricted include the following:

*   **nofile**: This sets the maximum number of open files
*   **maxlogins**: This is the maximum number of logins for this user except for the user with uid=0
*   **maxsyslogins**: This is the maximum number of all logins on a system
*   **priority**: This is the priority to run user process with (negative values boost process priority)
*   **nice**: This is the maximum nice priority allowed to raise to (Linux 2.6.12 and higher) values: [-20,19]

You can see from the preceding partial item list that it is possible to assign certain users a higher CPU priority than others, which is useful for call center groups whose access is more time sensitive than other accounts. The following example would give the telesales group a higher priority than standard users where the priority is normally 0:

```
*            -   priority    0
@telesales   -   priority   -5
```

It may also be useful to limit concurrent logins with an entry similar to the following:

```
@users  -   maxlogins   1
```

These limits might be very important to the running of your server and its security. So do take the time to investigate what is best in your environment, and more information can be found by referencing the manual page:

```
$ man 5 limits.conf

```

# SELinux

I am not really sure if I can quantify how many blogs I read on the Internet where "the solution" to an issue is to disable SELinux, or at least set it into permissive mode. While I do not disagree that the immediate problem may then be resolved, it is a little like setting the filesystem permissions to rwx for all users authenticated or otherwise. Similarly, we all joke about users sticking post-it notes with password to the screen; there is little difference in this to an administrator disabling SELinux inappropriately.

There are reasons that the **mandatory access control** (**MAC**) list exists, and we as administrators should use it to our advantage. Traditionally, we are accustomed to using **discretionary access control** (**DAC**) lists and these can be set by users as well as root. The MAC is said to be mandatory, as it can only be applied and revoked by root.

First the DAC list is applied, and then the MAC list. SELinux never gives additional rights that were not there in the first place. The real advantage that we have is that we can give the same process different rights depending on the SELinux context in which it was started or is accessing. So, a process started as a service through the `init` daemon can have different rights to the same process that was started outside of the normal `init` process by a user. SELinux is very powerful, and we just need to understand how we can channel its power.

## Reading the current SELinux mode

SELinux has two modes, but it can operate in three if you include the disabled mode. The operational modes are as follows:

*   Enforcing
*   Permissive

SELinux is enabled in both these modes, but only the `Enforcing` mode will apply and enforce SELinux policies. In `Permissive` mode, the policies are read but not acted upon; however, we can view any denials from the audit log. The log entries still appear as denials even though the denials are not enforced while in the `Permissive` mode. The `/usr/sbin/getenforce` command can be used to display the current SELinux mode:

```
$ getenforce

```

This setting can also be read from the `/selinux/enforce` file:

```
$ cat  /selinux/enforce

```

The integers 1 and 0 represent the SELinux modes: `Enforcing` and `Permissive` respectively. Note that the `/selinux` directory is the SELinux mountpoint in CentOS; in other systems, this may be `/sys/fs/selinux`, or you can check the output of `sestatus`, which is shown in the following screenshot:

![Reading the current SELinux mode](img/5920OS_10_02.jpg)

The mode can also be read from the `sestatus` (`/usr/bin/sestatus`) command; this command returns a little more information including the current mode as well as the mode configured in `/etc/selinux/config`.

## Setting the SELinux mode

As with most tasks in Linux, setting the SELinux mode is very flexible, in that, it can be set in many ways. Firstly we will look at the use of `setenforce` (`/usr/sbin/setenforce`):

```
# setenforce 1
# setenforce Enforcing
# setenforce 0
# setenforce Permissive

```

The first two options are used to set the mode to `Enforcing` and the bottom two, to `Permissive`, either method can be used, as the word and integer have the same meaning as the `setenforce` command.

We can also set the mode by writing directly to the control file within the SELinux mountpoint:

```
# echo 1 > /selinux/enforce
# echo 0 > /selinux/enforce

```

If we need more ways to set this, we can also use kernel options at boot time. Appending to the kernel entry in the GRUB configuration file will have the effect of overwriting the default mode in the `/etc/selinux/config` file:

```
enforcing=1
enforcing=0
```

Lastly, we can configure the mode in the `/etc/selinux/config` file.

## Preventing mode changes from the command line

I know many system administrators who will readily set the system to `Permissive` in order to provide a *quick fix*; if this is contrary to your administration policy, then you can enforce the mode at boot time by using the `setsebool` (`/usr/sbin/setsebool`) command; perhaps, as a script at system boot:

```
# setsebool secure_mode_policyload on

```

Once set, the mode cannot be changed from what was set during the startup process either from the file or boot parameters to the kernel. This can be seen from the following screenshot when the mode is attempted to be changed.

![Preventing mode changes from the command line](img/5920OS_10_03.jpg)

The setting made in this way will persist until the next boot. If we would prefer this setting to always be in place, then we can make it truly persistent with the `-P` option. Do take care while using this, as it does what it says on the tin. The setting is then persisted.

### Tip

Caution is advised in implementing the following command:

```
# setsebool -P secure_mode_policyload on

```

## Understanding SELinux contexts

SELinux will try to match the context of the process to the context of the resource being accessed; the SELinux policy in effect will specify what access is allowed to the resource from a given context. A SELinux context consists of four fields, note that the user is a SELinux user as opposed to a standard Linux user:

*   User
*   Role
*   Type
*   Sensitivity

### Tip

Technically, a file has a type and a user or process a domain, but in reality both the type and the domain are suffixed with `-t`.

We can view the context of a file with `ls -Z` as follows:

```
$ ls -Z /etc/hosts

```

See the following screenshot:

![Understanding SELinux contexts](img/5920OS_10_04.jpg)

The output shows the SELinux user, role, type, and sensitivity for the `/etc/hosts` file:

*   `system_u`
*   `object_r`
*   `net_conf_t`
*   `s0`

We can view the SELinux context of a user with the command ID:

```
$ id -Z

```

See the following screenshot:

![Understanding SELinux contexts](img/5920OS_10_05.jpg)

We can see from the output of the command that the SELinux context for this user (`user`) is currently:

*   `unconfined_u`
*   `unconfined_r`
*   `unconfined_t`
*   `s0` sensitivity (`s0` starts and stops at `s0`) and any category (`c0` through to `c1023`)

To view the SELinux context of a process, we can use the `ps` command. In this example, the process ID of `1629` represents the Nginx server:

```
# ps -Z 1629

```

Well over 90 percent of SELinux policies work with the type (`_t`), so most often this is where we will be troubleshooting.

## Troubleshooting SELinux

We will now look at what happens when the context of a file resource and process does not match the SELinux policy.

For the purpose of the demonstration, I will use CentOS 6.5 and the Apache httpd web server. To create a command SELinux issue the web server will be configured to access an aliased directory outside of the normal `/var/www` structure. This could mimic a typical process where you want to supply kickstart files via HTTP to aid the automatic installation of your workstations and servers.

Of course as soon as we work outside of the `/var/www/` directory, the file context or labels will not match the expectation of the web server and access will be denied even though we meet the requirements of the DAC (file permissions).

### Tip

The web server configuration is correct to allow the URL `/ks` to point to `/install/ks` as this has been added in by way of an alias.

Consider the following commands:

```
# mkdir  -m 755 -p /install/ks
# cp /root/anaconda-ks.cfg /install/ks
# chmod 644 /install/ks/anaconda-ks.cfg

```

With this in place, we can restart the web server and test access to the normal web root and to the URL `/ks`. For ease of demonstration, I have installed the command-line browser `w3m`:

```
$ w3m localhost

```

This works and displays the standard welcome page. Now use the following command:

```
$ w3m localhost/ks

```

We will see a rather ominous **Forbidden** page, as shown in the following screenshot:

![Troubleshooting SELinux](img/5920OS_10_06.jpg)

If we have the audit running, then SELinux denials are written to the audit log file, which is `/var/log/audit/audit.log`. We can use `ausearch` (available at `/sbin/ausearch`) to query this file:

```
# ausearch -m avc -ts recent

```

The options that we have used with the command are explained as follows:

*   `-m`: This is the message to search for. We look for `avc`, which is the SELinux denials.
*   `-ts`: This is the time start. If we use `recent`, it means from 10 minutes ago. The input `today` can also be commonly used.

The output of the `ausearch` command is shown in the following screenshot:

![Troubleshooting SELinux](img/5920OS_10_07.jpg)

As we can see from the output, we have been denied read access for the PID of `1731`. We were running the `httpd` command while accessing the `/ks` directory, which is the inode `22341` in the filesystem of `sda3`. I am already impressed with this detail! The subject context has the type `httpd_t`, and the target has the type `default_t`. The subject is process 1731, and the target is the file in this case, and they do not match contexts, and access is denied.

We can use the command, `chcon` (available at `/usr/bin/chcon`), to change the SELinux context of the target either directly or by referencing the context of a directory that we know works, `/var/www/html`:

```
# chcon -Rv --reference /var/www/html /install/ks

```

*   `-v`: This makes the file verbose
*   `-R`: This recurses all file and subdirectories
*   `--reference`: This means copy the context from this file

Alternatively, we can set the context directly; although for accuracy and simplicity, I would prefer to use the `--reference` method:

```
# chcon -Rv --type httpd_sys_content_t  /install.ks

```

A process of type `httpd_t` can access resources that include `httpd_sys_content_t`, so with this simple change, we should now have access to the `/ks` directory through the web server without disabling SELinux.

# Hardening Linux

We can really look at the previous example using SELinux to determine what we mean by hardening Linux, but this is often not the simple option. In the case of SELinux, the simple option is to set the `Permissive` mode but this does not go hand in hand with the best security for our systems.

Start with passwords and ask yourself how often are passwords changed on your system? When was the root password last changed? How many people have access to the root password? I come across many instances where the root password is never changed, and all administrators seem to have access to the root password. This is not a secure way of running your system even though it may help in the short term. Think of how many people who no longer work for your company have access to the root user password.

Of course, the system security has to work for you and the company, but the needs of a secure system should never be undervalued. For root access, consider using `sudo` instead of `su` and don't give the root password to each administrator.

Similarly, ensure strong password for accounts that have `sudo` rights as well as the root account.

## Password auditing

I will use the term password auditing, as I do feel the tool highlights the need for adequate monitoring of password policies. You may well ask users to use strong passwords and to a degree we can enforce this use PAM modules. However, how many users have the same password, perhaps managers whose passwords are set by an assistant. It may also be the case that you have not realized how important it is not to allow weak passwords, often to save issues with forgetful users. The package john from Openwall ([http://www.openwall.com](http://www.openwall.com)) is one not to be missed to help you understand the need for strong passwords and algorithms.

The RPM is not in the repository but can be obtained from the Openwall site.

## Preparing a password file

The password auditing tool john will be expecting users and passwords in one file, so we will use the `pwunconv` (`/usr/sbin/pwunconv`) command to add the passwords from the `/etc/shadow` file to the `/etc/passwd` file. Although this is not ideal, you have the password file; you can copy it to you own directory and use `pwconv` to switch back to the `/etc/shadow` file. The purpose of john is to show what is possible if password security isn't as it should be, and gaining rogue root privileges to the server is not unheard of. Even if john is run as a demonstration, it becomes a great tool to show what is possible with weak passwords, and a lab machine is fine for the demo.

The `pwunconv` command run as root is enough to read passwords from the shadow file into the `/etc/passwd` file.

## Cracking passwords

Now that we have the prepared password file, we can copy it our own home directory:

```
# cp /etc/passwd /root/passwd

```

This step is not strictly required, but will allow you to convert back to shadow files as quickly as possible if this is not a lab machine. Run the following `john` command with the path through to the password file you would like to crack. The password file should contain users and passwords.

```
# john /root/passwd

```

This will then start cracking the passwords that it finds. In my file, there is only one user with a password, and it uses the default SHA512 encryption.

### Tip

On my system, a simple password `Password1` took just 1 minute and 45 seconds to crack.

## Weakening the algorithm

The password is weak enough, but what if we weakened the algorithm too? MD5 is only 128-bit or 16-byte encryption; what effect would that have on the result? We can use the `chpasswd` (available at `/usr/sbin/chpasswd`) command to generate passwords with MD5 rather than SHA512 encryption:

```
# echo andrew:Password | chpasswd  -c MD5

```

This command writes to the `/etc/passwd` file; if you need to write to your own file, send the output to `stdout` with `-S` and add the displayed text to your own password file:

```
# echo andrew:Password | chpasswd -S  -c MD5

```

We can now try running john again. However, to ensure that the new password is cracked fairly, delete the `john.pol` file that will contain the original password:

```
# rm ~/.john/john.pot

```

The `john.pot` file contains previously cracked passwords; so to run a true comparison, we should ensure that this file does not exist. Now, running the same test on my system, the crack, with the same password set but only with 128-bit encryption, took just 0.8 seconds.

## Hardening the password

With a more secure password set, we can see the difference in time needed to crack the password, as a brute force attack will be needed. In this example, we use a password `27csg0TNWoUS`. This is more of a passphrase to me, as I would remember this as *the 27th scout group of the Northwestern United States*. Anyway, I am sure you have your own ways to memorize complex passwords that don't involve post-it notes.

With this password set and MD5 encryption, I left the program running for 20 minutes, and it had not cracked the password. In time, moving from 0.8 seconds with a simple password to 20 minutes is a great demonstration on the use of secure passwords and how much more effective a strong password is compared to relying on a strong algorithm.

Start with these few steps, and your Linux system will be more secure from the outset.

# Summary

In this chapter, we walked on the secure side of the road. Starting out with understanding a little of how PAM modules work and looking at ways in which we can use these modules to help us in administration and security. We saw that we could create user home directories, as they log in using PAM and a just-in-time method. We also looked at how we can restrict access and grant more access to users and groups using `/etc/security/limits.conf`. We then spent a little time to understand SELinux and securing our CentOS host.

The next chapter will discuss some general best practices guides to some of the elements that the book has taken us through.