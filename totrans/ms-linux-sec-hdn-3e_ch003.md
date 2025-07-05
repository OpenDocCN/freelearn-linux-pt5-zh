# 2 Securing User Accounts

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file19.png)

Managing users is one of the more challenging aspects of IT administration. You need to make sure that users can always access their stuff and that they can perform the required tasks to do their jobs. You also need to ensure that users' stuff is always secure from unauthorized users and that users can't perform any tasks that don't fit their job description. It's a tall order, but we aim to show that it's doable. In this chapter, we'll look at how to lock down user accounts and user credentials to protect them from attackers and snoopers. We'll also look at how to prevent users from having any more privileges than they have to have in order to perform their jobs.

The specific topics covered in this chapter are as follows:

*   The dangers of logging in as the root user
*   The advantages of using `sudo`
*   Setting up `sudo` privileges for full administrative users and for users with only certain delegated privileges
*   Advanced tips and tricks to use `sudo`
*   Locking down users' home directories
*   Enforcing strong password criteria
*   Setting and enforcing password and account expiration
*   Preventing brute-force password attacks
*   Locking user accounts
*   Setting up security banners
*   Detecting compromised passwords
*   Understanding central user management systems

## The dangers of logging in as the root user

A huge advantage that Unix and Linux operating systems have over Windows is that Unix and Linux do a much better job of keeping privileged administrative accounts separated from normal user accounts. Indeed, one reason that older versions of Windows were so susceptible to security issues, such as drive-by virus infections, was the common practice of setting up user accounts with administrative privileges, without having the protection of the **User Access Control (UAC)** that's in newer versions of Windows. (Even with UAC, Windows systems still do get infected, just not quite as often.) With Unix and Linux, it's a lot harder to infect a properly configured system.

You probably already know that the all-powerful administrator account on a Unix or Linux system is the root account. If you're logged in as the root user, you can do anything you want to do to that system. So you may think, "Yeah, that's handy, so that's what I'll do." However, always logging in as the root user can present a whole load of security problems. Consider the following. Logging in as the root user can do the following:

*   Make it easier for you to accidentally perform an action that causes damage to the system
*   Make it easier for someone else to perform an action that causes damage to the system

So if you always log on as the root user, or even if you just make the root user account readily accessible, you could say that you're doing a big part of attackers' and intruders' work for them. Also, imagine if you were the head Linux administrator at a large corporation, and the only way to allow users to perform admin tasks was to give them all the root password. What would happen if one of those users were to leave the company? You wouldn't want that person to still have the ability to log in to the systems, so you'd have to change the password and distribute the new one to all of the other users. And what if you just want users to have admin privileges only for certain tasks, instead of having full root privileges?

What we need is a mechanism that allows users to perform administrative tasks without incurring the risk of having them always log on as the root user, and that would also allow users to have only the admin privileges they really need to perform a certain job. In Linux and Unix, we have that mechanism in the form of the `sudo` utility.

## The advantages of using sudo

Used properly, the `sudo` utility can greatly enhance the security of your systems, and it can make an administrator's job much easier. With `sudo`, you can do the following:

*   Assign certain users full administrative privileges, while assigning other users only the privileges they need to perform tasks that are directly related to their respective jobs.
*   Allow users to perform administrative tasks by entering their own normal user passwords so that you don't have to distribute the root password to everybody and his brother.
*   Make it harder for intruders to break into your systems. If you implement `sudo` and disable the root user account, would-be intruders won't know which account to attack because they won't know which one has admin privileges.
*   Create `sudo` policies that you can deploy across an entire enterprise network, even if that network has a mix of Unix, BSD, and Linux machines.
*   Improve your auditing capabilities because you'll be able to see what users are doing with their admin privileges.

With regard to that last bullet point, consider the following snippet from the `secure` log of my CentOS 7 virtual machine:

```
Sep 29 20:44:33 localhost sudo: donnie : TTY=pts/0 ; PWD=/home/donnie ; 
USER=root ; COMMAND=/bin/su - 
Sep 29 20:44:34 localhost su: pam_unix(su-l:session): session opened for 
user root by donnie(uid=0) 
Sep 29 20:50:39 localhost su: pam_unix(su-l:session): session closed for 
user root 
```

You can see that I used `su -` to log in to the root command prompt and that I then logged back out. While I was logged in, I did several things that require root privileges, but none of that got recorded. What did get recorded though is something that I did with `sudo`. That is, because the root account is disabled on this machine, I used my `sudo` privilege to get `su -` to work for me. Let's look at another snippet to show a bit more detail about how this works:

```
Sep 29 20:50:45 localhost sudo: donnie : TTY=pts/0 ; PWD=/home/donnie ; 
USER=root ; COMMAND=/bin/less /var/log/secure 
Sep 29 20:55:30 localhost sudo: donnie : TTY=pts/0 ; PWD=/home/donnie ; 
USER=root ; COMMAND=/sbin/fdisk -l 
Sep 29 20:55:40 localhost sudo: donnie : TTY=pts/0 ; PWD=/home/donnie ; 
USER=root ; COMMAND=/bin/yum upgrade 
Sep 29 20:59:35 localhost sudo: donnie : TTY=tty1 ; PWD=/home/donnie ;
USER=root ; COMMAND=/bin/systemctl status sshd 
Sep 29 21:01:11 localhost sudo: donnie : TTY=tty1 ; PWD=/home/donnie ; 
USER=root ; COMMAND=/bin/less /var/log/secure
```

This time, I used my `sudo` privilege to open a log file, to view my hard drive configuration, to perform a system update, to check the status of the Secure Shell daemon, and to once again view a log file. So, if you were the security administrator at my company, you'd be able to see whether or not I'm abusing my `sudo` power.

Now, you're asking, "What's to prevent a person from just doing a `sudo su -` to prevent his or her misdeeds from being detected?" That's easy. Just don't give people the power to go to the root command prompt.

## Setting up sudo privileges for full administrative users

Before we look at how to limit what users can do, let's first look at how to allow a user to do everything, including logging in to the root command prompt. There are a couple of methods for doing that.

### Adding users to a predefined admin group

The first method, which is the simplest, is to add users to a predefined administrators group and then, if it hasn't already been done, to configure the `sudo` policy to allow that group to do its job. It's simple enough to do except that different Linux distro families use different admin groups.

On Unix, BSD, and most Linux systems, you would add users to the wheel group. (Members of the Red Hat family, including CentOS and AlmaLinux, fall into this category.) When I do the `groups` command on any of my RHEL-type machines, I get this:

```
[donnie@localhost ~]$ groups
donnie wheel
[donnie@localhost ~]$
```

This shows that I'm a member of the `wheel` group. By doing `sudo visudo`, I'll open the `sudo` policy file. Scrolling down, we'll see the line that gives the `wheel` group its awesome power:

```
## Allows people in group wheel to run all commands
%wheel ALL=(ALL) ALL
```

The percent sign indicates that we're working with a group. The three appearances of `ALL` means that members of that group can perform any command, as any user, on any machine in the network on which this policy is deployed. The only slight catch is that group members will be prompted to enter their own normal user account passwords in order to perform a `sudo` task. Scroll down a bit more and you'll see the following:

```
## Same thing without a password
# %wheel ALL=(ALL) NOPASSWD: ALL
```

If we were to comment out the `%wheel` line in the former snippet and remove the comment symbol from in front of the `%wheel` line in this snippet, then members of the wheel group would be able to perform all of their `sudo` tasks without ever having to enter any password. That's something that I really don't recommend, even for home use. In a business setting, allowing people to have password-less `sudo` privileges is a definite no-no.

To add an existing user to the `wheel` group, use `usermod` with the `-G` option. You might also want to use the `-a` option, in order to prevent removing the user from other groups to which he or she belongs. For our example, let's add Maggie:

```
sudo usermod -a -G wheel maggie
```

You can also add a user account to the `wheel` group as you create it. Let's do that now for Frank:

```
sudo useradd -G wheel frank
```

> Tip
> 
> > Note that, with my usage of `useradd`, I'm assuming that we're working with a member of the Red Hat family, which comes with predefined default settings to create user accounts. For non-Red Hat-type distros that use the `wheel` group, you'd need to either reconfigure the default settings or use extra option switches in order to create the user's home directory and to assign the correct shell. Your command then would look something like this:

```
sudo useradd -G wheel -m -d /home/frank -s /bin/bash frank
```

For members of the Debian family, including Ubuntu, the procedure is the same, except that you would use the `sudo` group instead of the `wheel` group. (This kind of figures, considering that the Debian folk have pretty much always marched to the beat of a different drum.)

> Tip:
> 
> > One way in which this technique would come in handy is whenever you need to create a virtual private server on a cloud service, such as Rackspace, DigitalOcean, or Vultr. When you log in to one of those services and initially create your virtual machine, the cloud service will have you log in to that virtual machine as the root user. (This even happens with Ubuntu, even though the root user account is disabled whenever you do a local installation of Ubuntu.)
> 
> > The first thing that you'll want to do in this scenario is to create a normal user account for yourself and give it full `sudo` privileges. Then, log out of the root account and log back in with your normal user account. You'll then want to disable the root account with this command:

```
sudo passwd -l root
```

> You'll also want to do some additional configuration to lock down Secure Shell access, but we'll cover that in *Chapter 6*, *SSH Hardening.*

### Creating an entry in the sudo policy file

Okay, adding users to either the `wheel` group or the `sudo` group works great if you're either just working with a single machine or if you're deploying a `sudo` policy across a network that uses just one of these two admin groups. But what if you want to deploy a `sudo` policy across a network with a mixed group of both Red Hat and Ubuntu machines? Or what if you don't want to go around to each machine to add users to an admin group? Then, just create an entry in the `sudo` policy file. You can either create an entry for an individual user or create a user alias. If you do `sudo visudo` on either your CentOS or one of your AlmaLinux virtual machines, you'll see a commented-out example of a user alias:

```
# User_Alias ADMINS = jsmith, mikem
```

You can uncomment this line and add your own set of usernames, or you can just add a line with your own user alias. To give members of the user alias full `sudo` power, add another line that would look like this:

```
ADMINS ALL=(ALL) ALL 
```

It's also possible to add a `visudo` entry for just a single user, and you might need to do that under very special circumstances. Here's an example:

```
frank ALL=(ALL) ALL
```

But for ease of management, it's best to go with either a user group or a user alias.

> Tip:
> 
> > The `sudo` policy file is the `/etc/sudoers` file. I always hesitate to tell students that because, every once in a while, I have a student try to edit it in a regular text editor. That doesn't work though, so please don't try it. Always edit `sudoers` with the `sudo visudo` command.

## Setting up sudo for users with only certain delegated privileges

A basic tenet of IT security philosophy is to give network users enough privileges so that they can get their jobs done, but no privileges beyond that. So, you'll want as few people as possible to have full `sudo` privileges. (If you have the root user account enabled, you'll want even fewer people to know the root password.) You'll also want a way to delegate privileges to people according to what their specific jobs are. Backup admins will need to be able to perform backup tasks, help desk personnel will need to perform user management tasks, and so on. With `sudo`, you can delegate these privileges and disallow users from doing any other administrative jobs that don't fit their job description.

The best way to explain this is to have you open `visudo` on your any of the RHEL-type virtual machines. CentOS 7, AlmaLinux 8 and AlmaLinux 9 all work well for this. So, go ahead and start up one of them and enter:

```
sudo visudo
```

Unlike Ubuntu, the RHEL-type distros have a fully commented and well-documented `sudoers` file. I've already shown you the line that creates the `ADMIN` user alias, and you can create other user aliases for other purposes. You can, for example, create a `BACKUPADMINS` user alias for backup administrators, a `WEBADMINS` user alias for web server administrators, or whatever else you desire. So, you could add a line that looks something like this:

```
User_Alias SOFTWAREADMINS = vicky, cleopatra
```

That's good, except that Vicky and Cleopatra still can't do anything. You'll need to assign some duties to the user alias.

If you look at the example user alias mentioned later, you'll see a list of example command aliases. One of these examples just happens to be `SOFTWARE`, which contains the commands that an admin would need in order to either install or remove software or to update the system. It's commented out, as are all of the other example command aliases, so you'll need to remove the hash symbol from the beginning of the line before you can use it:

```
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
```

Now, it's just a simple matter of assigning the `SOFTWARE` command alias to the `SOFTWAREADMINS` user alias:

```
SOFTWAREADMINS ALL=(ALL) SOFTWARE
```

Vicky and Cleopatra, both members of the `SOFTWAREADMINS` user alias, can now run the `rpm`, `up2date`, and `yum` commands with root privileges, on all servers on which this policy is installed.

All but one of these predefined command aliases are ready to use after you uncomment them and assign them to either a user, group, or user alias. The one exception is the `SERVICES` command alias:

```
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start, /usr/bin/systemctl stop, /usr/bin/systemctl reload, /usr/bin/systemctl restart, /usr/bin/systemctl status, /usr/bin/systemctl enable, /usr/bin/systemctl disable
```

The problem with this `SERVICES` alias is that it also lists the different subcommands for the `systemctl` command. The way `sudo` works is that if a command is listed by itself, then the assigned user can use that command with any subcommands, options, or arguments. So, in the `SOFTWARE` example, members of the `SOFTWARE` user alias can run a command such as this:

```
sudo yum upgrade
```

But when a command is listed in the command alias with a subcommand, option, or argument, that's all anyone who's assigned to the command alias can run. With the `SERVICES` command alias in its current configuration, the `systemctl` commands just won't work. To see why, let's set Charlie and Lionel up in the `SERVICESADMINS` user alias and then uncomment the `SERVICES` command alias, as we did earlier:

```
User_Alias SERVICESADMINS = charlie, lionel
SERVICESADMINS ALL=(ALL) SERVICES
```

Now, watch what happens when Lionel tries to check the status of the Secure Shell service:

```
[lionel@centos-7 ~]$ sudo systemctl status sshd
 [sudo] password for lionel:
 Sorry, user lionel is not allowed to execute '/bin/systemctl status sshd' as root on centos-7.xyzwidgets.com.
 [lionel@centos-7 ~]$
```

Okay, so Lionel can run `sudo systemctl status`, which is pretty much useless, but he can't do anything meaningful, such as specifying the service that he wants to check. That's a bit of a problem. There are two ways to fix this, but there's only one way that you want to use. You could just eliminate all of the `systemctl` subcommands and make the `SERVICES` alias look like this:

```
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl
```

But if you do that, Lionel and Charlie will also be able to shut down or reboot the system, edit the services files, or change the machine from one systemd target to another. That's probably not what you want. Because the `systemctl` command covers a lot of different functions, you have to be careful not to allow delegated users to access too many of those functions. A better solution would be to add a wildcard to each of the `systemctl` subcommands:

```
Cmnd_Alias SERVICES = /sbin/service, /sbin/chkconfig, /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl reload *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *, /usr/bin/systemctl enable *, /usr/bin/systemctl disable * 
```

Now, Lionel and Charlie can perform any of the `systemctl` functions that are listed in this command alias, for any service:

```
 [lionel@centos-7 ~]$ sudo systemctl status sshd
 [sudo] password for lionel:
 ● sshd.service - OpenSSH server daemon
    Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
  Active: active (running) since Sat 2017-09-30 18:11:22 EDT; 23min ago
     Docs: man:sshd(8)
              man:sshd_config(5)
  Main PID: 13567 (sshd)
      CGroup: /system.slice/sshd.service
                   └─13567 /usr/sbin/sshd -D
 Sep 30 18:11:22 centos-7.xyzwidgets.com systemd[1]: Starting OpenSSH server daemon...
 Sep 30 18:11:22 centos-7.xyzwidgets.com sshd[13567]: Server listening on 0.0.0.0 port 22.
 Sep 30 18:11:22 centos-7.xyzwidgets.com sshd[13567]: Server listening on :: port 22.
 Sep 30 18:11:22 centos-7.xyzwidgets.com systemd[1]: Started OpenSSH server daemon.
 [lionel@centos-7 ~]$
```

Keep in mind that you're not limited to using user aliases and command aliases. You can also assign privileges to either a Linux group or an individual user. You can also assign individual commands to a user alias, Linux group, or individual user. Here's an example:

```
katelyn ALL=(ALL) STORAGE
gunther ALL=(ALL) /sbin/fdisk -l
%backup_admins ALL=(ALL) BACKUP
```

Katelyn can now do all of the commands in the `STORAGE` command alias, whereas Gunther can only use `fdisk` to look at the partition tables. The members of the `backup_admins` Linux group can do commands in the `BACKUP` command alias.

The last thing we'll look at in this topic is the host aliases examples that you see preceding the user alias example:

```
# Host_Alias FILESERVERS = fs1, fs2
# Host_Alias MAILSERVERS = smtp, smtp2
```

Each host alias consists of a list of server hostnames. This is what allows you to create one `sudoers` file on one machine and deploy it across the network. For example, you could create a `WEBSERVERS` host alias, a `WEBADMINS` user alias, and a `WEBCOMMANDS` command alias with the appropriate commands. Your configuration would look something like this:

```
Host_Alias WEBSERVERS = webserver1, webserver2
User_Alias WEBADMINS = junior, kayla
Cmnd_Alias WEBCOMMANDS = /usr/bin/systemctl status httpd, /usr/bin/systemctl start httpd, /usr/bin/systemctl stop httpd, /usr/bin/systemctl restart httpd
WEBADMINS WEBSERVERS=(ALL) WEBCOMMANDS 
```

Now, when a user types a command into a server on the network, `sudo` will first look at the hostname of that server. If the user is authorized to perform that command on that server, then `sudo` allows it. Otherwise, `sudo` denies it. In a small to medium-sized business, it would probably work just fine to manually copy the master `sudoers` file to all the servers on the network. But in a large enterprise, you'll want to streamline and automate the process. For this, you could use something like Puppet, Chef, or Ansible. (These three technologies are beyond the scope of this book, but you'll find plenty of books and video courses about all three of them on the Packt website.)

All of these techniques will work on your Ubuntu VM as well as on the CentOS VM. The only catch is that Ubuntu doesn't come with any predefined command aliases, so you'll have to type them in yourself.

Anyway, I know that you're tired of reading, so let's do some work.

### Hands-on lab for assigning limited sudo privileges

In this lab, you'll create some users and assign them different levels of privileges. To simplify things, let’s use the AlmaLinux 9 virtual machine.

1.  Log in to either the AlmaLinux virtual machine and create user accounts for Lionel, Katelyn, and Maggie:

```
 sudo useradd lionel
 sudo useradd katelyn
 sudo useradd maggie
 sudo passwd lionel
 sudo passwd katelyn
 sudo passwd maggie
```

1.  Open `visudo`:

```
sudo visudo
```

Find the `STORAGE` command alias and remove the comment symbol from in front of it.

1.  Add the following lines to the end of the file, using tabs to separate the columns:

```
lionel ALL=(ALL) ALL
katelyn ALL=(ALL) /usr/bin/systemctl status sshd
maggie ALL=(ALL) STORAGE 
```

Save the file and exit `visudo`.

1.  To save time, we'll use `su` to log in to the different user accounts. That way, you won't need to log out of your own account to perform these steps. First, log in to Lionel's account and verify that he has full `sudo` privileges by running several root-level commands:

```
 su - lionel
 sudo su -
 exit
 sudo systemctl status sshd
 sudo fdisk -l
 exit
```

1.  This time, log in as Katelyn and try to run some root-level commands. Don't be too disappointed if they don't all work, though:

```
 su - katelyn
 sudo su -
 sudo systemctl status sshd
 sudo systemctl restart sshd
 sudo fdisk -l
 exit
```

1.  Finally, log in as Maggie, and run the same set of commands that you ran for Katelyn.
2.  Keep in mind that, although we only had three individual users for this lab, you could just as easily have handled more users by setting them up in user aliases or Linux groups.

> Since `sudo` is such a great security tool, you would think that everyone would use it, right? Sadly, that's not the case. Pretty much any time you look at either a Linux tutorial website or a Linux tutorial YouTube channel, you'll see the person who's doing the demo logged in at the root user command prompt. In some cases, I've seen the person remotely logged in as the root user on a cloud-based virtual machine. Now, if logging in as the root user is already a bad idea, then logging in across the internet as the root user is an even worse idea. In any case, seeing everybody do these tutorial demos from the root user's shell drives me absolutely crazy.
> 
> > Having said all this, there are some things that don't work with `sudo`. Bash shell internal commands such as `cd` don't work with it, and using `echo` to inject kernel values into the `/proc` filesystem also doesn't work with it. For tasks such as these, a person would have to go to the root command prompt. Still, though, make sure that only users who absolutely have to use the root user command prompt have access to it.

Next, let’s look at some more advanced sudo usage.

## Advanced tips and tricks for using sudo

Now that we've looked at the basics of setting up a good `sudo` configuration, we're confronted with a bit of a paradox. That is, even though `sudo` is a security tool, certain things that you can do with it can make your system even more insecure than it was. Let's see how to avoid that.

### The sudo timer

By default, the `sudo` timer is set for five minutes. This means that once a user performs one `sudo` command and enters a password, he or she can perform another `sudo` command within five minutes without having to enter the password again. Although this is obviously handy, it can also be problematic if users were to walk away from their desks with a command terminal still open.

If the five-minute timer hasn't yet expired, someone else could come along and perform some root-level task. If your security needs require it, you can easily disable this timer by adding a line to the `Defaults` section of the `sudoers` file. This way, users will have to enter their passwords every time they run a `sudo` command. You can make this a global setting for all users, or you can just set it for certain individual users.

Let's also say that you're sitting in your nice, cozy cubicle, logged in to a remote Linux server that still has the five-minute timer enabled. If you need to leave your desk for a moment, your best action would be to log out of the server first. Short of that, you could just reset the `sudo` timer by running this command:

```
sudo -k
```

This is one of the few `sudo` actions you can do without entering a password. But the next time you do a `sudo` command, you will have to enter your password, even if it has been less than five minutes since you entered your password previously.

### View your sudo privileges

Are you unsure of what `sudo` privileges that you possess? Not to worry, you have a way to find out. Just run this command:

```
sudo -l
```

When I do this for myself, I first see some of the environmental variables for my account, and then I see that I have full `sudo` privileges:

```
donnie@packtpub1:~$ sudo -l
 [sudo] password for donnie:
 Matching Defaults entries for donnie on packtpub1:
 env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

 User donnie may run the following commands on packtpub1:
 (ALL : ALL) ALL
 donnie@packtpub1:~$
```

When Frank, my formerly feral flamepoint Siamese cat, does this for his account, he sees that he can only do the `fdisk -l` command:

```
frank@packtpub1:~$ sudo -l
 [sudo] password for frank:
 Matching Defaults entries for frank on packtpub1:
 env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

 User frank may run the following commands on packtpub1:
 (ALL) /sbin fdisk -l
 frank@packtpub1:~$
```

But since he's a cat, he doesn't complain. Instead, he'll just try to do something sneaky, as we'll see in just a bit.

#### Hands-on lab for disabling the sudo timer

For this lab, you'll disable the `sudo` timer on your AlmaLinux VM:

1.  Log in to the same AlmaLinux virtual machine that you used for the previous lab. We'll be using the user accounts that you've already created.
2.  At your own user account command prompt, enter the following commands:

```
 sudo fdisk -l
 sudo systemctl status sshd
 sudo iptables -L
```

You'll see that you only needed to enter the password once to do all three commands.

1.  At your own user account command prompt, run the following:

```
 sudo fdisk -l
 sudo -k
 sudo fdisk -l
```

Note how the `sudo -k` command resets your timer, so you'll have to enter your password again. Open `visudo` with the following command:

```
sudo visudo
```

In the `Defaults specification` section of the file, add the following line:

```
Defaults timestamp_timeout = 0 
```

Save the file and exit `visudo`.

1.  Perform the commands that you performed in *Step 2*. This time, you should see that you have to enter a password every time.
2.  Open `visudo` and modify the line that you added so that it looks like this:

```
Defaults:lionel timestamp_timeout = 0 
```

Save the file and exit `visudo`.

1.  From your own account shell, repeat the commands that you performed in *Step 2*. Then, log in as Lionel and perform the commands again.
2.  View your own `sudo` privileges by running the following:

```
sudo -l
```

> Note that this procedure also works for Ubuntu.

### Preventing users from having root shell access

Let's say that you want to set up a user with limited `sudo` privileges, but you did so by adding a line like this:

```
maggie ALL=(ALL) /bin/bash, /bin/zsh
```

I'm sorry to say that you haven't limited Maggie's access at all. You have effectively given her full `sudo` privileges with both the Bash shell and the ZSH shell. So, don't add lines like this to your `sudoers` because it will get you into trouble.

### Preventing users from using shell escapes

Certain programs, especially text editors and pagers, have a handy shell escape feature. This allows a user to run a shell command without having to exit the program first. For example, from the command mode of the Vi and Vim editors, someone could run the `ls` command by running `:!ls`. Executing the command would look like this:

```
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
~
~
:!ls
```

The output would look like this:

```
[donnie@localhost default]$ sudo vim useradd
 [sudo] password for donnie:
 grub nss useradd
 Press ENTER or type command to continue
 grub nss useradd
 Press ENTER or type command to continue
```

Now, imagine that you want Frank to be able to edit the `sshd_config` file and only that file. You might be tempted to add a line to your `sudo` configuration that would look like this:

```
frank ALL=(ALL) /bin/vim /etc/ssh/sshd_config
```

This looks like it would work, right? Well, it doesn't, because once Frank has opened the `sshd_config` file with his `sudo` privilege, he can then use Vim's shell escape feature to perform other root-level commands, which includes being able to edit other configuration files. You can fix this problem by having Frank use `sudoedit` instead of `vim`:

```
frank ALL=(ALL) sudoedit /etc/ssh/sshd_config
```

`sudoedit` has no shell escape feature, so you can safely allow Frank to use it. Other programs that have a shell escape feature include the following:

*   emacs
*   less
*   view
*   more

### Preventing users from using other dangerous programs

Some programs that don't have shell escapes can still be dangerous if you give users unrestricted privileges to use them. These include the following:

*   cat
*   cut
*   awk
*   sed

If you must give someone `sudo` privileges to use one of these programs, it's best to limit their use to only specific files. And that brings us to our next tip.

### Limiting the user's actions with commands

Let's say that you create a `sudo` rule so that Sylvester can use the `systemctl` command:

```
sylvester ALL=(ALL) /usr/bin/systemctl
```

This allows Sylvester to have full use of the `systemctl` features. He can control daemons, edit service files, shut down or reboot, and carry out every other function that `systemctl` does. That's probably not what you want. It would be better to specify what `systemctl` functions that Sylvester is allowed to do. Let's say that you want him to be able to control just the Secure Shell service. You can make the line look like this:

```
sylvester ALL=(ALL) /usr/bin/systemctl * sshd 
```

Sylvester can now do everything he needs to do with the Secure Shell service, but he can't shut down or reboot the system, edit other service files, or change systemd targets. But what if you want Sylvester to do only certain specific actions with the Secure Shell service? Then you'll have to omit the wildcard and specify all of the actions that you want Sylvester to do:

```
sylvester ALL=(ALL) /usr/bin/systemctl status sshd, /usr/bin/systemctl restart sshd 
```

Now, Sylvester can only restart the Secure Shell service or check its status.

> Tip:
> 
> > When writing `sudo` policies, you'll want to be aware of the differences between the different Linux and Unix distributions on your network. For example, on Red Hat-type systems, the `systemctl` binary file is located in the `/usr/bin` directory. On Debian/Ubuntu systems, it's located in the `/bin` directory. If you have to roll out a `sudoers` file to a large enterprise network with mixed operating systems, you can use host aliases to ensure that servers will only allow the execution of commands that are appropriate for their operating systems.
> 
> > Also, be aware that some system services have different names on different Linux distributions. On Red Hat-type systems, the Secure Shell service is `sshd`. On Debian/Ubuntu systems, it's just plain `ssh`.

### Letting users run as other users

In the following line, `(ALL)` means that Sylvester can run the `systemctl` commands as any user:

```
sylvester ALL=(ALL) /usr/bin/systemctl status sshd, /usr/bin/systemctl restart sshd
```

This effectively gives Sylvester root privileges for these commands because the root user is definitely any user. You could, if desired, change that `(ALL)` to `(root)` in order to specify that Sylvester can only run these commands as the root user:

```
sylvester ALL=(root) /usr/bin/systemctl status sshd, /usr/bin/systemctl restart sshd 
```

Okay, there's probably not much point in that because nothing changes. Sylvester had root privileges for these `systemctl` commands before, and he still has them now. But there are more practical uses for this feature. Let's say that Vicky is a database admin, and you want her to run as the `database` user:

```
vicky ALL=(database) /usr/local/sbin/some_database_script.sh
```

Vicky could then run the command as the database user by entering the following command:

```
sudo -u database some_database_script.sh
```

This is one of those features that you might not use that often, but keep it in mind anyway. You never know when it might come in handy.

### Preventing abuse via user's shell scripts

So, what if a user has written a shell script that requires `sudo` privileges? To answer that, let's have Frank create the `frank_script.sh` shell script that looks like this:

```
#!/bin/bash
echo "This script belongs to Frank the Cat."
```

Okay, he wouldn't need `sudo` privileges for that, but let's pretend that he does. After he sets the executable permission and runs it with `sudo`, the output will look like this:

```
 frank@packtpub1:~$ sudo ./frank_script.sh
 [sudo] password for frank:
 Sorry, user frank is not allowed to execute './frank_script.sh' as root on packtpub1.tds.
 frank@packtpub1:~$
```

So, naturally frustrated, Frank requested that I create a `sudo` rule so that he can run the script. So, I open `visudo` and add this rule for Frank:

```
frank ALL=(ALL) /home/frank/frank_script.sh
```

Now when Frank runs the script with `sudo`, it works:

```
 frank@packtpub1:~$ sudo ./frank_script.sh
 [sudo] password for frank:
 This script belongs to Frank the Cat.
 frank@packtpub1:~$
```

But since this file is in Frank's own home directory and he is its owner, he can edit it any way he wants. So, being the sneaky type, he adds the `sudo -i` line to the end of the script so that it now looks like this:

```
#!/bin/bash

echo "This script belongs to Frank the Cat."
sudo -i
```

Be prepared for a shock as you watch what happens next:

```
 frank@packtpub1:~$ sudo ./frank_script.sh
 This script belongs to Frank the Cat.
 root@packtpub1:~#
```

As you can see, Frank is now logged in as the root user.

What `sudo -i` does is to log a person in to the root user's shell, the same way that `sudo su -` does. If Frank were to do `sudo -i` from his own command prompt, it would fail because Frank doesn't have the privilege to do that. But he does have the `sudo` privilege to run his own shell script. By leaving the shell script in his own home directory, Frank can put root-level commands into it. By running the script with `sudo`, the root-level commands in the script will execute with root-level privileges.

To remedy this, I'll use my awesome powers of `sudo` to move Frank's script to the `/usr/local/sbin/` directory and change the ownership to the root user so that Frank won't be able to edit it. And of course, before I do that, I'll make sure to delete that `sudo -i` line from it:

```
 donnie@packtpub1:~$ sudo -i
 root@packtpub1:~# cd /home/frank
 root@packtpub1:/home/frank# mv frank_script.sh /usr/local/sbin
 root@packtpub1:/home/frank# chown root: /usr/local/sbin/frank_script.sh
 root@packtpub1:/home/frank# exit
 logout
 donnie@packtpub1:~$
```

Finally, I'll open `visudo` and change his rule to reflect the new location of the script. The new rule looks like this:

```
frank ALL=(ALL) /usr/local/sbin/frank_script.sh
```

Frank can still run the script, but he can't edit it:

```
 frank@packtpub1:~$ sudo frank_script.sh
 This script belongs to Frank the Cat.
 frank@packtpub1:~$
```

### Detecting and deleting default user accounts

One challenge of dealing with **Internet of Things (IoT)** devices is that you don't do a normal operating system installation on them as you would when setting up a normal server. Instead, you download an image that has the operating system pre-installed, and burn that image to a microSD card. The installed operating system is set up with a default user account, and many times that user is set up with full `sudo` privileges and isn't required to enter a `sudo` password. Let's take, for example, the Raspex Linux distribution for the Raspberry Pi. (Raspex is built from Ubuntu source code.) On the documentation page of the Raspex download site, we see that the default user is `raspex`, and the default password for that user is also **raspex**. We also see that the default password for the `root` user is **root**:

![19501_02_01.png](img/file20.png)

19501_02_01.png

So, the default credentials are out there for all the world to see. Obviously, the first thing you want to do when setting up an IoT device is to set up your own user account, give it a good password, and give it `sudo` privileges. Then get rid of that default account, because leaving it in place, especially if you leave the default password, is just asking for trouble.

But let's dig deeper. Look in the `/etc/password` file on Raspex, and you'll see the default user there:

```
raspex:x:1000:1000:,,,:/home/raspex:/bin/bash
```

Then, look in the `/etc/sudoers` file, and you'll see this line, which allows the `raspex` user to do all `sudo` commands without having to enter a password:

```
raspex ALL=(ALL) NOPASSWD: ALL
```

Another thing to watch out for is that some Linux distributions for IoT devices have this rule in a separate file in the `/etc/sudoers.d` directory, instead of in the main `sudoers` file. Either way, you'll want to delete this rule, as well as the default user account, when you set up your IoT device. And of course, you'll also want to change the `root` user password, and then lock the `root` user account.

All right, let’s take a quick look at some new features that have been recently added to sudo.

## New sudo Features

I mentioned before that one of the beautiful things about sudo is that it allows you to see what users are doing with their sudo privileges. Beginning with sudo version 1.9.0, the sudo logging experience has been greatly enhanced. You can now save sudo log messages in .json format, which allows sudo to log much more information than it normally would, in a format that’s easier to parse. Beginning with sudo version 1.9.4, you can also have sudo send its log messages to a central log server, making it more difficult for bad actors to delete mention of their dirty deeds from the system log files.

Unfortunately, space constraints don’t allow me to do a full write-up about these new features here. That’s okay, though. Over at Opensource.com, Mr. Peter Czanik has written a great article that explains them very well. So, I’ll just refer you to him:

> 5 new sudo features sysadmins need to know in 2022-- [https://opensource.com/article/22/2/new-sudo-features-2022](https://opensource.com/article/22/2/new-sudo-features-2022)

I should also mention that in order to know which new sudo features your Linux distro supports, you’ll need to know which version of sudo that is includes. Find out by doing:

```
sudo --version
```

Next, let’s look at some SUSE quirkiness.

## Special sudo Considerations for SUSE and OpenSUSE

If you’ve ever worked with any kind of SUSE machine, you may have been puzzled by the fact that it asks for the root user’s password, rather than your own password, when you perform a sudo command. That’s because SUSE has a whole different way of doing business with sudo.

When you install a SUSE distro, you’ll see a user creation screen that looks similar to the one that you’ve seen on the RHEL-type distros.

![19501_02_02.png](img/file21.png)

19501_02_02.png

However, when you check the **Use this password for system administrator** box, it doesn’t add your user account to the wheel group as the RHEL-type distros do. Instead, it automatically assigns the same password that you created for yourself to the root user account. So, you and the root user will both have the same password.

When you do sudo visudo on a SUSE machine, you’ll see these two lines that you don’t see on any other Linux distro:

```
Defaults targetpw   # ask for the password of the target user i.e. root
ALL   ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
```

This means that any user who has the root user password can perform all sudo commands. To be fair though, the comments that precede these two lines inform us that this is only for the initial system setup, and that we should reconfigure sudo in the normal way before putting the machine into production use. To do that, first add your own user account to the wheel group, like so:

```
donnie@localhost:~> sudo usermod -a -G wheel donnie
[sudo] password for root: 
donnie@localhost:~>
```

Cause the group membership to take effect by logging out of the system and then logging back in. Use the groups command to verify wheel group membership, like so:

```
donnie@localhost:~> groups
users wheel
donnie@localhost:~>
```

Next, do sudo visudo, and comment out the two lines that we looked at before. They should now look like this:

```
#Defaults targetpw   # ask for the password of the target user i.e. root
#ALL   ALL=(ALL) ALL   # WARNING! Only use this together with 'Defaults targetpw'!
```

(Or, better yet, just delete the lines.)

Scroll down in the file until you see this line:

```
# %wheel ALL=(ALL:ALL) ALL
```

Remove the preceding comment symbol to make the line look like this:

```
%wheel ALL=(ALL:ALL) ALL
```

Save the file and exit visudo. Now, when you do a sudo command, you’ll be asked to enter your own password instead of the root user’s password.

```
donnie@localhost:~> sudo fdisk -l
[sudo] password for donnie:
. . .
. . .
```

Of course, the root user account is still enabled, so let’s disable it, like so:

```
donnie@localhost:~> sudo passwd -l root
passwd: password expiry information changed.
donnie@localhost:~>
```

All righty, that pretty much covers the sudo topic. Let’s look at how to secure normal user accounts.

## Locking down users' home directories the Red Hat way

This is another area where different Linux distro families do business differently from each other. As we shall see, each distro family comes with different default security settings for users’ home directories. A security administrator who oversees a mixed environment of different Linux distros will need to take this into account.

One beautiful thing about Red Hat Enterprise Linux and all of its offspring, such as CentOS and AlmaLinux, is that they have better out-of-the-box security than any other Linux distro. This makes it quicker and easier to harden Red Hat-type systems because much of the work has already been done. One thing that's already been done for us is locking down users' home directories:

```
 [donnie@localhost home]$ sudo useradd charlie
 [sudo] password for donnie:
 [donnie@localhost home]$
 [donnie@localhost home]$ ls -l
 total 0
 drwx------. 2 charlie charlie 59 Oct 1 15:25 charlie
 drwx------. 2 donnie donnie 79 Sep 27 00:24 donnie
 drwx------. 2 frank frank 59 Oct 1 15:25 frank
 [donnie@localhost home]$
```

By default, the `useradd` utility on Red Hat-type systems creates user home directories with a permissions setting of `700`. This means that only the user who owns the home directory can access it. All other normal users are locked out. We can see why by looking at the `/etc/login.defs` file. On your CentOS 7 VM, scroll down toward the bottom of the file, and you'll see this:

```
CREATE_HOME yes
UMASK 077
```

In the login.defs file of a RHEL 8 or RHEL 9-type distro, such as AlmaLinux, you’ll see that the UMASK is set for wide-open permissions, which seems a bit strange. Here’s what that looks like:

```
UMASK           022
```

But, a few lines below that, you’ll see a brand-new directive that we never had before, which looks like this:

```
HOME_MODE       0700
```

So, even though the UMASK is wide-open, new user home directories still get properly locked-down.

The `login.defs` file is one of two files where default settings for `useradd` are configured. Either the `UMASK` line or the HOME_MODE line is what determines the permissions values on home directories as they get created. Red Hat-type distros have it configured with the `077` value, which removes all permissions from the group and others. This `UMASK` line is in the `login.defs` file for all Linux distros, but until recently, Red Hat-type distros have been the only ones that have `UMASK` set to such a restrictive value by default. Most non-Red Hat distros usually have a `UMASK` value of `022`, which creates home directories with a permissions value of `755`. This allows everybody to enter everybody else's home directories and access each others' files.

## Locking down users' home directories the Debian/Ubuntu way

Debian and its offspring, such as Ubuntu, have two user creation utilities:

1.  `useradd`
2.  `adduser`

Let's have a look at both of them.

### useradd on Debian/Ubuntu

The `useradd` utility is there, but Debian and Ubuntu don't come with the handy preconfigured defaults as the Red Hat-type distros do. If you were to just do `sudo useradd frank` on a default Debian/Ubuntu machine, Frank would have no home directory and would be assigned the wrong default shell. So, to create a user account with `useradd` on a Debian or Ubuntu system, the command would look something like this:

```
sudo useradd -m -d /home/frank -s /bin/bash frank
```

In this command, we have the following:

1.  `-m` creates the home directory.
2.  `-d` specifies the home directory.
3.  `-s` specifies Frank's default shell. (Without the `-s`, Debian/Ubuntu would assign to Frank the `/bin/sh shell`.)

When you look at the home directories on either a Debian or an Ubuntu 20.04 machine, you'll see that they're wide open, with execute and read privileges for everybody:

```
 donnie@packt:/home$ ls -l
 total 8
 drwxr-xr-x 3 donnie donnie 4096 Oct 2 00:23 donnie
 drwxr-xr-x 2 frank frank 4096 Oct 1 23:58 frank
 donnie@packt:/home$
```

As you can see, Frank and I can get into each other's stuff. (And no, I don't want Frank getting into my stuff.) Each user could change the permissions on his or her own directory, but how many of your users would know how to do that? So, let's fix that ourselves:

```
 cd /home
 sudo chmod 700 *
```

Let's see what we have now:

```
 donnie@packt:/home$ ls -l
 total 8
 drwx------ 3 donnie donnie 4096 Oct 2 00:23 donnie
 drwx------ 2 frank frank 4096 Oct 1 23:58 frank
 donnie@packt:/home$
```

That looks much better.

To change the default permissions setting for home directories, open `/etc/login.defs` for editing. Look for this line:

```
UMASK 022
```

Change it to this:

```
UMASK 077
```

Now, new users' home directories will get locked down on creation, just as they do with Red Hat and its offspring.

On Ubuntu 22.04, things are different. Ubuntu developers have finally realized that users’ home directories should be locked down by default. So, the HOME_MODE setting in an Ubuntu 22.04 login.defs file now looks like this:

```
HOME_MODE       0750
```

This includes access permissions for a user’s own personal group, but that’s okay. It still effectively means that only the respective owners of the various home directories can get into them.

### adduser on Debian/Ubuntu

The `adduser` utility is an interactive way to create user accounts and passwords with a single command, which is unique to the Debian family of Linux distros. Most of the default settings that are missing from the Debian implementation of `useradd` are already set for `adduser`. On Debian and Ubuntu 20.04, it creates user home directories with the wide-open `755` permissions value. Fortunately, that's easy to change. (We'll see how in just a bit.) On Ubuntu 22.04, it creates properly locked-down home directories with the restrictive 750 permissions value.

Although `adduser` is handy for just casual creation of user accounts, it doesn't offer the flexibility of `useradd` and it isn't suitable for use in shell scripting. One thing that `adduser` can do that `useradd` can’t is to automatically encrypt a user's home directory as you create the account. To make it work, you'll first have to install the `ecryptfs-utils` package. So, to create an account with an encrypted home directory for Cleopatra, you do the following:

```
sudo apt install ecryptfs-utils
 donnie@ubuntu-steemnode:~$ sudo adduser --encrypt-home cleopatra
 [sudo] password for donnie:
 Adding user `cleopatra' ...
 Adding new group `cleopatra' (1004) ...
 Adding new user `cleopatra' (1004) with group `cleopatra' ...
 Creating home directory `/home/cleopatra' ...
 Setting up encryption ...
 ************************************************************************
 YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
  ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
 THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
 ********************************************************************
Done configuring.
 Copying files from `/etc/skel' ...
 Enter new UNIX password:
 Retype new UNIX password:
 passwd: password updated successfully
 Changing the user information for cleopatra
 Enter the new value, or press ENTER for the default
  Full Name []: Cleopatra Tabby Cat
  Room Number []: 1
  Work Phone []: 555-5556
  Home Phone []: 555-5555
  Other []:
 Is the information correct? [Y/n] Y
 donnie@ubuntu-steemnode:~$
```

The first time that Cleopatra logs in, she'll need to run the `ecryptfs-unwrap-passphrase` command that's mentioned in the preceding output. She'll then want to write her passphrase down and store it in a safe place:

```
 cleopatra@ubuntu-steemnode:~$ ecryptfs-unwrap-passphrase
 Passphrase:
 d2a6cf0c3e7e46fd856286c74ab7a412
 cleopatra@ubuntu-steemnode:~$
```

We'll look at the whole encryption thing in more detail when we get to the encryption chapter.

#### Hands-on lab for configuring adduser

For this lab, we'll be working with the `adduser` utility on an Ubuntu 22.04 VM.

1.  Install the `ecryptfs-utils` package:

```
sudo apt install ecryptfs-utils
```

1.  Create a user account with an encrypted home directory for Cleopatra and then view the results:

```
 sudo adduser --encrypt-home cleopatra
 ls -l /home
```

1.  Log in as Cleopatra and run the `ecryptfs-unwrap-passphrase` command:

```
su - cleopatra
ecryptfs-unwrap-passphrase
exit
```

1.  Note that some of the information that `adduser` asks for is optional, and you can just hit the *Enter* key for those items.

## Enforcing strong password criteria

You wouldn't think that a benign-sounding topic such as strong password criteria would be so controversial, but it is. The conventional wisdom that you've undoubtedly heard for your entire computer career says:

*   Make passwords of a certain minimum length.
*   Make passwords that consist of a combination of uppercase letters, lowercase letters, numbers, and special characters.
*   Ensure that passwords don't contain any words that are found in the dictionary or that are based on the users' own personal data.
*   Force users to change their passwords on a regular basis.

But, using your favorite search engine, you'll see that different experts disagree on the details of these criteria. For example, you'll see disagreements about whether passwords should be changed every 30, 60, or 90 days, disagreements about whether all four types of characters need to be in a password, and even disagreements on what the minimum length of a password should be.

The most interesting controversy of all comes from—of all places—the guy who invented the preceding criteria to begin with. He now says that it's all bunk and regrets having come up with it. He now says that we should be using passphrases that are long, yet easy to remember. He also says that they should be changed only if they've been breached.

> Bill Burr, the former National Institutes of Standards and Technology (NIST) engineer who created the strong password criteria that I outlined earlier, shares his thoughts about why he now disavows his own work. Refer to [https://www.pcmag.com/news/355496/you-might-not-need-complex-alphanumeric-passwords-after-all](https://www.pcmag.com/news/355496/you-might-not-need-complex-alphanumeric-passwords-after-all).
> 
> > And, since the original edition of this book was published, NIST has come to agree with Bill Burr. They have now changed their password implementation criteria to match Mr. Burr's recommendations. You can read about that at
> 
> > [https://www.riskcontrolstrategies.com/2018/01/08/new-nist-guidelines-wrong/](https://www.riskcontrolstrategies.com/2018/01/08/new-nist-guidelines-wrong/).

However, having said all that, there is the reality that many organizations are still wedded to the idea of using complex passwords that regularly expire, and you'll have to abide by their rules if you can't convince them otherwise. And besides, if you are using traditional passwords, you do want them to be strong enough to resist any sort of password attack. So now, we'll take a look at the mechanics of enforcing strong password criteria on a Linux system.

> Tip:
> 
> > I have to confess that I had never before thought to try creating a passphrase to use in place of a password on a Linux system. So, I just now tried it on my CentOS virtual machine to see if it would work.
> 
> > I created an account for Maggie, my black-and-white tuxedo kitty. For her password, I entered the passphrase `I like other kitty cats`. You may think, "Oh, that's terrible. This doesn't meet any complexity criteria*,* and it uses dictionary words. How is that secure?" But the fact that it's a phrase with distinct words separated by blank spaces does make it secure and very difficult to brute-force.
> 
> > Now, in real life, I would never create a passphrase that expresses my love for cats because it's not hard to find out that I really do love cats. Rather, I would choose a passphrase about some more obscure part of my life that nobody but me knows about. In any case, there are two advantages of passphrases over passwords. They're more difficult to crack than traditional passwords, yet they're easier for users to remember. For extra security, though, just don't create passphrases about a fact of your life that everybody knows about.

### Installing and configuring pwquality

We'll be using the `pwquality` module for the **Pluggable Authentication Module** (**PAM**). This is a newer technology that has replaced the old `cracklib` module. On any Red Hat 7 or newer type of system, and on SUSE and OpenSUSE, `pwquality` is installed by default, even if you do a minimal installation. If you `cd` into the `/etc/pam.d` directory, you can do a `grep` operation to see that the PAM configuration files are already set up. `retry=3` means that a user will only have three tries to get the password right when logging in to the system:

```
[donnie@localhost pam.d]$ grep 'pwquality' *
 password-auth:password requisite pam_pwquality.so try_first_pass
 local_users_only retry=3 authtok_type=
 password-auth-ac:password requisite pam_pwquality.so try_first_pass
 local_users_only retry=3 authtok_type=
 system-auth:password requisite pam_pwquality.so try_first_pass
 local_users_only retry=3 authtok_type=
 system-auth-ac:password requisite pam_pwquality.so try_first_pass
 local_users_only retry=3 authtok_type=
 [donnie@localhost pam.d]$
```

On Debian and Ubuntu, you’ll need to install pwquality yourself, like this:

```
sudo apt install libpam-pwquality
```

The rest of the procedure is the same for all of our operating systems and consists of just editing the `/etc/security/pwquality.conf` file. When you open this file in your text editor, you'll see that everything is commented out, which means that no password complexity criteria are in effect. You'll also see that it's very well documented because every setting has its own explanatory comment.

You can set password complexity criteria however you want just by uncommenting the appropriate lines and setting the appropriate values. Let's take a look at just one setting:

```
# Minimum acceptable size for the new password (plus one if 
# credits are not disabled which is the default). (See pam_cracklib manual.) 
# Cannot be set to lower value than 6\. 
# minlen = 8 
```

The minimum length setting works on a credit system. This means that for every different type of character class in the password, the minimum required password length will be reduced by one character. For example, let's set `minlen` to a value of `19` and try to assign Katelyn the password `turkeylips`:

```
minlen = 19
[donnie@localhost ~]$ sudo passwd katelyn
 Changing password for user katelyn.
 New password:
 BAD PASSWORD: The password is shorter than 18 characters
 Retype new password:
 [donnie@localhost ~]$
```

Because the lowercase characters in `turkeylips` count as credit for one type of character class, we're only required to have 18 characters instead of 19\. If we try this again with `TurkeyLips`, we'll get:

```
[donnie@localhost ~]$ sudo passwd katelyn
 Changing password for user katelyn.
 New password:
 BAD PASSWORD: The password is shorter than 17 characters
 Retype new password:
 [donnie@localhost ~]$
```

This time, the uppercase `T` and uppercase `L` count as a second character class, so we only need to have 17 characters in the password.

Just below the `minlen` line, you'll see the credit lines. Let's say that you don't want lowercase letters to count toward your credits. You would find this line:

```
# lcredit = 1 
```

Uncomment it, and change the `1` to a `0`:

```
lcredit = 0
```

Then, try assigning Katelyn `turkeylips` as a password:

```
[donnie@localhost ~]$ sudo passwd katelyn
 Changing password for user katelyn.
 New password:
 BAD PASSWORD: The password is shorter than 19 characters
 Retype new password:
 [donnie@localhost ~]$
```

This time, the `pwquality` really does want 19 characters. If we set a credit value to something higher than `1`, we would get credit for multiple characters of the same class type up to that value.

We can also set the credit values to negative numbers in order to require a certain number of characters types in a password. For example, we could have this:

```
dcredit = -3
```

This would require at least three digits in a password. However, it's a really bad idea to use this feature, because someone who's doing a password attack would soon find the patterns that you require, which would help the attacker to more precisely direct the attack. If you need to require that a password has multiple character types, it would be better to use the `minclass` parameter:

```
# minclass = 3
```

It's already set to a value of `3`, which would require characters from three different classes. To use this value, all you have to do is to remove the comment symbol.

The rest of the parameters in `pwquality.conf` work pretty much the same way, and each one has a well-written comment to explain what it does.

> Tip:
> 
> > If you use your `sudo` privilege to set someone else's password, the system will complain if you create a password that doesn't meet complexity criteria, but it will let you do it. If a normal user were to try to change his or her own password without `sudo` privileges, the system would not allow a password that doesn't meet complexity criteria.

#### Hands-on lab for setting password complexity criteria

For this lab, you can use either a CentOS, AlmaLinux, or Ubuntu virtual machine, as desired. The only difference is that you won't perform Step 1 for either CentOS or AlmaLinux.

1.  For Ubuntu only, install the `libpam-pwquality` package:

```
sudo apt install libpam-pwquality
```

1.  Open the `/etc/security/pwquality.conf` file in your preferred text editor. Remove the comment symbol from in front of the `minlen` line and change the value to `19`. It should now look like this:

```
 minlen = 19
```

Save the file and exit the editor.

1.  Create a user account for Goldie and attempt to assign her the passwords `turkeylips`, `TurkeyLips`, and `Turkey93Lips`. Note the change in each warning message.
2.  In the `pwquality.conf` file, comment out the `minlen` line. Uncomment the `minclass` line and the `maxclassrepeat` line. Change the `maxclassrepeat` value to `5`. The lines should now look like this:

```
minclass = 3 
maxclassrepeat = 5 
```

Save the file and exit the text editor.

1.  Try assigning various passwords that don't meet the complexity criteria that you've set to Goldie's account and view the results.

> In the `/etc/login.defs` file on your CentOS 7 machine, you'll see the line `PASS_MIN_LEN 5`.
> 
> > Supposedly, this is to set the minimum password length, but in reality, `pwquality` overrides it. So, you could set this value to anything at all, and it would have no effect.

## Setting and enforcing password and account expiration

Something you never want is to have unused user accounts remain active. There have been incidents where an administrator set up user accounts for temporary usage, such as for a conference, and then just forgot about them after the accounts were no longer needed.

Another example would be if your company were to hire contract workers whose contract expires on a specific date. Allowing those accounts to remain active and accessible after the temporary employees leave the company would be a huge security problem. In cases like these, you want a way to ensure that temporary user accounts aren't forgotten about when they're no longer needed. If your employer subscribes to the conventional wisdom that users should change their passwords on a regular basis, then you'll also want to ensure that it gets done.

Password expiration data and account expiration data are two different things. They can be set either separately or together. When someone's password expires, he or she can change it, and everything will be all good. If somebody's account expires, only someone with the proper admin privileges can unlock it.

To get started, take a look at the expiry data for your own account. Note that you won't need `sudo` privileges to look at your own data, but you will still need to specify your own username:

```
donnie@packt:~$ chage -l donnie
 [sudo] password for donnie:
 Last password change : Oct 03, 2017
 Password expires : never
 Password inactive : never
 Account expires : never
 Minimum number of days between password change : 0
 Maximum number of days between password change : 99999
 Number of days of warning before password expires : 7
 donnie@packt:~$
```

You can see here that no expiration data have been set. Everything here is set according to the out-of-the-box system default values. Other than the obvious items, here's a breakdown of what you see:

*   `Password inactive`: If this were set to a positive number, I would have that many days to change an expired password before the system would lock out my account.
*   `Minimum number of days between password change`: Because this is set to `0`, I can change my password as often as I like. If it were set to a positive number, I would have to wait that number of days after changing my password before I could change it again.
*   `Maximum number of days between password change`: This is set to the default value of `99999`, meaning that my password will never expire.
*   `Number of days of warning before password expires`: The default value is `7`, but that's rather meaningless when the password is set to never expire.

> With the `chage` utility, you can either set password and account expiration data for other users or use the `-l` option to view expiration data. Any unprivileged user can use `chage -l` without `sudo` to view his or her own data. To either set data or view someone else's data, you need `sudo`. We'll take a closer look at `chage` a bit later.

Before we look at how to change expiration data, let's first look at where the default settings are stored. We'll first look at the `/etc/login.defs` file. Here are the three relevant lines:

```
PASS_MAX_DAYS 99999 
PASS_MIN_DAYS 0 
PASS_WARN_AGE 7
```

You can edit these values to fit your organization's needs. For example, changing `PASS_MAX_DAYS` to a value of `30` would cause all new user passwords from that point on to have a 30-day expiration data. (By the way, setting the default password expiry data in `login.defs` works for all of the Linux distros that we’re using.)

## Configuring default expiry data for useradd for Red Hat-type systems only

The `/etc/default/useradd` file has the rest of the default settings. In this case, we'll look at the one from the AlmaLinux 9 machine:

> Ubuntu also has the `useradd` configuration file, but it doesn't work. No matter how you configure it, the Ubuntu version of `useradd` just won't read it. So, the write-up about this file only applies to Red Hat-type systems.

```
# useradd defaults file
GROUP=100
HOME=/home 
INACTIVE=-1 
EXPIRE= 
SHELL=/bin/bash 
SKEL=/etc/skel 
CREATE_MAIL_SPOOL=yes
```

The `EXPIRE=` line sets the default expiration date for new user accounts. By default, there is no default expiration date. `INACTIVE=-1` means that user accounts won't be automatically locked out after the users' passwords expire. If we set this to a positive number, then any new users will have that many days to change an expired password before the account gets locked. To change the defaults in the `useradd` file, you can either hand-edit the file or use `useradd -D` with the appropriate option switch for the item that you want to change. For example, to set a default expiration date of December 31, 2023, the command would be as follows:

```
sudo useradd -D -e 2023-12-31
```

To see the new configuration, you can either open the `useradd` file or just do `sudo useradd -D`:

```
[donnie@localhost ~]$ sudo useradd -D
 GROUP=100
 HOME=/home
 INACTIVE=-1
 EXPIRE=2023-12-31
 SHELL=/bin/bash
 SKEL=/etc/skel
 CREATE_MAIL_SPOOL=yes
 [donnie@localhost ~]$
```

You've now set it so that any new user accounts that get created will have the same expiration date. You can do the same thing with either the `INACTIVE` setting or the `SHELL` setting:

```
sudo useradd -D -f 5
 sudo useradd -D -s /bin/zsh
 [donnie@localhost ~]$ sudo useradd -D
 GROUP=100
 HOME=/home
 INACTIVE=5
 EXPIRE=2019-12-31
 SHELL=/bin/zsh
 SKEL=/etc/skel
 CREATE_MAIL_SPOOL=yes
 [donnie@localhost ~]$
```

Now, any new user accounts that get created will have the Zsh shell set as the default shell and will have to have expired passwords changed within five days to prevent having the account automatically locked out.

> `useradd` doesn't do any safety checks to ensure that the default shell that you've assigned is installed on the system. In our case, Zsh isn't installed, but `useradd` will still allow you to create accounts with Zsh as the default shell.

So, just how useful is this `useradd` configuration feature in real life? Probably not that much, unless you need to create a whole bunch of user accounts at once with the same settings. Even so, a savvy admin would just automate the process with a shell script, rather than messing around with this configuration file.

## Setting expiry data on a per-account basis with useradd and usermod

You might find it useful to set the default password expiry data in `login.defs`, but you probably won't find it too useful to configure the `useradd` configuration file. Really, what are the chances that you'll want to create all user accounts with the same account expiration date? Setting password expiry data in `login.defs` is more useful because you'll just be saying that you want new passwords to expire within a certain number of days, rather than to have them all expire on a specific date.

Most likely, you'll want to set account expiry data on a per-account basis, depending on whether you know that the accounts will no longer be needed as of a specific date. There are three ways that you can do this:

*   Use `useradd` with the appropriate option switches to set expiry data as you create the accounts. (If you need to create a whole bunch of accounts at once with the same expiry data, you can automate the process with a shell script.)
*   Use `usermod` to modify expiry data on existing accounts. (The beautiful thing about `usermod` is that it uses the same option switches as `useradd`.)
*   Use `chage` to modify expiry data on existing accounts. (This one uses a whole different set of option switches.)

You can use `useradd` and `usermod` to set account expiry data, but not to set password expiry data. The only two option switches that affect account expiry data are as follows:

*   `-e`: Use this to set an expiration date for the account, in the form YYYY-MM-DD.
*   `-f`: Use this to set the number of days after the user's password expires that you want for his or her account to get locked out.

Let's say that you want to create an account for Charlie that will expire at the end of 2025\. On a Red Hat-type machine, you could enter the following:

```
sudo useradd -e 2025-12-31 charlie
```

On a non-Red Hat-type machine, you'd have to add the option switches that create the home directory and assign the correct default shell:

```
sudo useradd -m -d /home/charlie -s /bin/bash -e 2025-12-31 charlie
```

Use `chage -l` to verify what you've entered:

```
donnie@ubuntu-steemnode:~$ sudo chage -l charlie
 Last password change : Oct 06, 2017
 Password expires : never
 Password inactive : never
 Account expires : Dec 31, 2025
 Minimum number of days between password change : 0
 Maximum number of days between password change : 99999
 Number of days of warning before password expires : 7
 donnie@ubuntu-steemnode:~$
```

Now, let's say that Charlie's contract has been extended, and you need to change his account expiration to the end of January 2026\. You'll use `usermod` the same way on any Linux distro:

```
sudo usermod -e 2026-01-31 charlie
```

Again, verify that everything is correct with `chage -l`:

```
donnie@ubuntu-steemnode:~$ sudo chage -l charlie
 Last password change : Oct 06, 2017
 Password expires : never
 Password inactive : never
Account expires : Jan 31, 2026
 Minimum number of days between password change : 0
 Maximum number of days between password change : 99999
 Number of days of warning before password expires : 7
 donnie@ubuntu-steemnode:~$
```

Optionally, you can set the number of days before an account with an expired password will get locked out:

```
sudo usermod -f 5 charlie
```

But if you were to do that now, you wouldn't see any difference in the `chage -l` output because we still haven't set expiration data for Charlie's password.

## Setting expiry data on a per-account basis with chage

You would only use `chage` to modify existing accounts, and you would use it for setting either an account expiration or a password expiration. Here are the relevant option switches:

| **Option** | **Explanation** |
| `-d` | If you use the `-d 0` option on someone's account, you'll force the user to change his or her password on their next login. |
| `-E` | This is equivalent to the lowercase `-e` for `useradd` or `usermod` . It sets the expiration date for the user account. |
| `-I` | This is equivalent to `-f` for `useradd` or `usermod` . It sets the number of days before an account with an expired password will be locked out. |
| `-m` | This sets the minimum number of days between password changes. In other words, if Charlie changes his password today, the `-m 5` option will force him to wait five days before he can change his password again. |
| `-M` | This sets the maximum number of days before a password expires. (Be aware, though, that if Charlie last set his password 89 days ago, using a `-M 90` option on his account will cause his password to expire tomorrow, not 90 days from now.) |
| `-W` | This will set the number of warning days for passwords that are about to expire. |

1.  You can set just one of these data items at a time or you can set them all at once. In fact, to avoid frustrating you with a different demo for each individual item, let's set them all at once, except for `-d 0`, and then we'll see what we've got:

```
sudo chage -E 2026-02-28 -I 4 -m 3 -M 90 -W 4 charlie
 donnie@ubuntu-steemnode:~$ sudo chage -l charlie
 Last password change : Oct 06, 2019
 Password expires : Jan 04, 2026
 Password inactive : Jan 08, 2026
 Account expires : Feb 28, 2026
 Minimum number of days between password change : 3
 Maximum number of days between password change : 90
 Number of days of warning before password expires : 4
 donnie@ubuntu-steemnode:~$
```

All expiration data have now been set.

For our final example, let's say that you've just created a new account for Samson, and you want to force him to change his password the first time he logs in. There are two ways to do that. Either way, you would do it after you've set his password initially. For example, let's do this:

```
sudo chage -d 0 samson
 or
 sudo passwd -e samson

 donnie@ubuntu-steemnode:~$ sudo chage -l samson
 Last password change : password must be changed
 Password expires : password must be changed
 Password inactive : password must be changed
 Account expires : never
 Minimum number of days between password change : 0
 Maximum number of days between password change : 99999
 Number of days of warning before password expires : 7
 donnie@ubuntu-steemnode:~$
```

Next, we will go through a hands-on lab.

### Hands-on lab for setting account and password expiry data

In this lab, you'll create a couple of new user accounts, set expiration data, and view the results. You can do this lab on any of your virtual machines. The only difference will be with the `useradd` commands:

1.  On your CentOS or AlmaLinux VM, create a user account for Samson with the expiration date of June 30, 2025, and view the results:

```
sudo useradd -e 2025-06-30 samson
sudo chage -l samson
```

For Ubuntu, run this command:

```
sudo useradd -m -d /home/samson -s /bin/bash -e 2025-06-30
sudo chage -l samson
```

1.  Use `usermod` to change Samson's account expiration date to July 31, 2025:

```
sudo usermod -e 2025-07-31
sudo chage -l samson
```

1.  Assign a password to Samson's account, then force him to change his password on his first login. Log in as Samson, change his password, then login to your own account:

```
sudo passwd samson
sudo passwd -e samson
sudo chage -l samson
su - samson
exit
```

1.  Use `chage` to set a five-day waiting period for changing passwords, a password expiration period of 90 days, an inactivity period of two days, and a warning period of five days:

```
sudo chage -m 5 -M 90 -I 2 -W 5 samson
sudo chage -l samson
```

1.  Keep this account, because you'll be using it for the lab in the next section.

Next, let's see how to prevent brute-force attacks.

## Preventing brute-force password attacks

Amazingly enough, this is another topic that engenders a bit of controversy. I mean, nobody denies the wisdom of automatically locking out user accounts that are under attack. The controversial part concerns the number of failed login attempts that we should allow before locking the account.

Back in the stone age of computing, so long ago that I still had a full head of hair, the early Unix operating systems only allowed users to create a password with a maximum of eight lowercase letters. So in those days, it was possible for early man to brute-force someone else's password just by sitting down at the keyboard and typing in random passwords. That's when the philosophy started of having user accounts get locked out after only three failed login attempts. Nowadays, with strong passwords, or better yet, a strong passphrase, setting a lockout value of three failed login attempts will do three things:

1.  It will unnecessarily frustrate users.
2.  It will cause extra work for help desk personnel.
3.  If an account really is under attack, it will lock the account before you've had a chance to gather information about the attacker.

Setting the lockout value to something more realistic, such as 100 failed login attempts, will still provide good security, while still giving you enough time to gather information about the attackers. Just as importantly, you won't cause undue frustration to users and help desk personnel.

Anyway, regardless of how many failed login attempts your employer allows you to allow, you'll still need to know how to set it all up. On RHEL 7-type systems and Ubuntu 18.04, you’ll do this by configuring the pam_tally2 Pluggable Authentication Module (PAM). On RHEL 8/9-type systems and Ubuntu 20.04/22.04, you’ll instead configure the pam_faillock PAM module. Let’s dig in and see how it’s done.

### Configuring the pam_tally2 PAM module on CentOS 7

To make this magic work, we'll rely on our good friend, PAM. The `pam_tally2` module comes already installed on CentOS 7, but it isn't configured. We'll begin by editing the `/etc/pam.d/login` file. Figuring out how to configure it is easy because there's an example at the bottom of the `pam_tally2` man page:

```
EXAMPLES
 Add the following line to /etc/pam.d/login to lock the account after
4 failed logins. Root account will be locked as well. The accounts will be
automatically unlocked after 20 minutes. The module does not have to be
called in the account phase because the login calls pam_setcred(3)
correctly.
 auth required pam_securetty.so
 auth required pam_tally2.so deny=4 even_deny_root
unlock_time=1200
 auth required pam_env.so
 auth required pam_unix.so
 auth required pam_nologin.so
 account required pam_unix.so
 password required pam_unix.so
 session required pam_limits.so
 session required pam_unix.so
 session required pam_lastlog.so nowtmp
 session optional pam_mail.so standard
```

In the second line of the example, we see that `pam_tally2` is set with the following parameters:

*   `deny=4`: This means that the user account under attack will get locked out after only four failed login attempts.
*   `even_deny_root`: This means that even the root user account will get locked if it's under attack.
*   `unlock_time=1200`: The account will get automatically unlocked after 1,200 seconds, or 20 minutes.

Now, if you look at the actual `login` file on either of your virtual machines, you'll see that they don't look exactly like this example `login` file that's in both of their man pages. That's okay, we'll still make it work.

Once you've configured the `login` file and have had a failed login, you'll see a new file created in the `/var/log` directory. You'll view information from that file with the `pam_tally2` utility. You can also use `pam_tally2` to manually unlock a locked account if you don't want to wait for the timeout period:

```
donnie@ubuntu-steemnode:~$ sudo pam_tally2
 Login Failures Latest failure From
 charlie 5 10/07/17 16:38:19
 donnie@ubuntu-steemnode:~$ sudo pam_tally2 --user=charlie --reset
 Login Failures Latest failure From
 charlie 5 10/07/17 16:38:19
 donnie@ubuntu-steemnode:~$ sudo pam_tally2
 donnie@ubuntu-steemnode:~$
```

Note that, after I did the reset on Charlie's account, I received no output from doing another query.

#### Hands-on lab for configuring pam_tally2 on CentOS 7

Configuring `pam_tally2` is super easy because it only requires adding one line to the `/etc/pam.d/login` file. To make things even easier, you can just copy and paste that line from the example in the `pam_tally2` man page. In spite of what I said earlier about bumping the number of failed logins up to 100, we'll keep that number at 4 for now, because I know that you don't want to have to do 100 failed logins in order to demo this:

1.  On the CentOS 7 virtual machine, open the `/etc/pam.d/login` file for editing. Look for the line that invokes the `pam_securetty` module. (That should be around line 2.) Beneath that line, insert this line:

```
auth required pam_tally2.so deny=4 even_deny_root unlock_time=1200
```

Save the file and exit the editor.

1.  For this step, you'll need to log out of your own account, because `pam_tally2` doesn't work with `su`. So log out and, while purposely using the wrong password, attempt to log in to the `samson` account that you created in the previous lab. Keep doing that until you see the message that the account is locked. Note that when the `deny` value is set to `4`, it will actually take five failed login attempts to lock Samson out.
2.  Log back in to your own user account. Run this command and note the output:

```
sudo pam_tally2
```

1.  For this step, you'll simulate that you're a help desk worker, and Samson has just called to request that you unlock his account. After verifying that you really are talking to the real Samson, enter the following two commands:

```
sudo pam_tally2 --user=samson --reset
sudo pam_tally2
```

1.  Now that you've seen how this works, open the `/etc/pam.d/login` file for editing, and change the `deny=` parameter from `4` to `100` and save the file. (This will make your configuration a bit more realistic in terms of modern security philosophy.)
2.  Next, let’s look at configuring pam_faillock on our AlmaLinux machines.

### Configuring pam_faillock on AlmaLinux 8/9

1.  The pam_faillock module is already installed on any RHEL 8 or RHEL 9-type of Linux distro. Since the basic concepts of pam_faillock are pretty much the same as they are for pam_tally2, we’ll dispense with the preliminary explanations and jump right to the hands-on procedure.

#### Hands-on lab for configuring pam_faillock on AlmaLinux 8 or AlmaLinux 9

Although you can enable and configure pam_faillock by hand-editing the PAM configuration files, the RHEL distros provide an easier method, which is called *authselect*.

1.  On either an AlmaLinux 8 or an AlmaLinux 9 VM, view the available authselect profiles by doing:

```
[donnie@localhost ~]$ sudo authselect list
[sudo] password for donnie: 
- minimal    Local users only for minimal installations
- sssd       Enable SSSD for system authentication (also for local users only)
- winbind    Enable winbind for system authentication
[donnie@localhost ~]$
```

1.  For now, at least, we’re only dealing with local users. So, we’ll use the minimal profile. View the features of this profile like this:

```
[donnie@localhost ~]$ sudo authselect list-features minimal
. . .
. . .
with-faillock
. . .
. . .
[donnie@localhost ~]$
```

Note that there are a lot of included features, but we’re only interested in the with-faillock feature.

1.  Enable the minimal profile, like this:

```
[donnie@localhost ~]$ sudo authselect select minimal --force
```

1.  After enabling a profile, we can now enable the pam_faillock module, like this:

```
[donnie@localhost ~]$ sudo authselect enable-feature with-faillock
```

1.  In the /etc/security/ directory, open the faillock.conf file in your favorite text editor. Look for these four lines:

```
# silent
# deny = 3
# unlock_time = 600
# even_deny_root
```

Remove the preceding comment symbols from all four lines, and save the file.

1.  Create a user account, and deliberately have the user make three failed login attempts. View the results, like this:

```
[donnie@localhost ~]$ sudo faillock
donnie:
When                Type  Source                                           Valid
vicky:
When                Type  Source                                           Valid
2022-10-12 15:54:35 RHOST 192.168.0.16                                         V
2022-10-12 15:54:42 RHOST 192.168.0.16                                         V
2022-10-12 15:54:46 RHOST 192.168.0.16                                         V
[donnie@localhost ~]$
```

1.  Wait until the ten-minute timer expires, and then have your user try to log in with the correct password.
2.  Have the user log out. Then, have the user again deliberately make three failed login attempts. This time, reset the user’s account before the timer expires, like this:

```
[donnie@localhost ~]$ sudo faillock --reset --user vicky
```

Doing this on Ubuntu is a bit different, so let’s now look at that.

### Configuring pam_faillock on Ubuntu 20.04 and Ubuntu 22.04

Sadly, the authselect utility isn’t available for Ubuntu, so we’ll just have to hand-edit the PAM configuration files. Here’s the procedure.

#### Hands-on lab for configuring pam_faillock on Ubuntu 20.04 and Ubuntu 22.04

1.  Open the /etc/pam.d/common-auth file in your favorite text editor. At the top of the file, insert these two lines:

```
auth        required                                     pam_faillock.so preauth silent
       auth        required                                     pam_faillock.so authfail
```

1.  Open the /etc/pam.d/common-account file in your text editor. At the bottom of the file, add this line:

```
account     required                                     pam_faillock.so
```

1.  Configure the /etc/security/faillock.conf file the same way that I’ve shown in Step 5 of the preceding lab for AlmaLinux.
2.  Test the setup as outlined in Steps 6 through 8 of the preceding AlmaLinux lab.
3.  And, that’s all there is to it. Next, let’s look at how to manually lock a user’s account.

### Locking user accounts

Okay, you've just seen how to have Linux automatically lock user accounts that are under attack. There will also be times when you'll want to be able to manually lock out user accounts. Let's look at a few examples:

*   When a user goes on vacation and you want to ensure that nobody monkeys around with that user's account while he or she is gone
*   When a user is under investigation for questionable activities
*   When a user leaves the company

With regard to the last point, you may be asking yourself, *Why can't we just delete the accounts of people who are no longer working here?* And, you certainly can, easily enough. However, before you do so, you'll need to check with your local laws to make sure that you don't get yourself into deep trouble. Here in the United States, for example, we have the Sarbanes-Oxley law, which restricts what files that publicly traded companies can delete from their computers. If you were to delete a user account, along with that user's home directory and mail spool, you just might be running afoul of Sarbanes-Oxley or whatever you may have as the equivalent law in your own home country.

Anyway, there are two utilities that you can use to temporarily lock a user account:

*   `usermod`
*   `passwd`

> In apparent contradiction to what I just said, at some point you will need to remove inactive user accounts. That's because malicious actors can use an inactive account to perform their dirty deeds, especially if that inactive account had any sort of administrative privileges. But when you do remove the accounts, make sure that you do so in accordance with local laws and with company policy. In fact, your best bet is to ensure that your organization has written guidelines for removing inactive user accounts in its change management procedures.

### Using usermod to lock a user account

Let's say that Katelyn has gone on maternity leave and will be gone for several weeks. We can lock her account by doing:

```
 sudo usermod -L katelyn
```

When you look at Katelyn's entry in the `/etc/shadow` file, you'll now see an exclamation point in front of her password hash, like this:

```
katelyn:!$6$uA5ecH1A$MZ6q5U.cyY2SRSJezV000AudP.ckXXndBNsXUdMI1vPO8aFmlLXcbGV25K5HSSaCv4RlDilwzlXq/hKvXRkpB/:17446:0:99999:7:::
```

This exclamation point prevents the system from reading her password hash, which effectively locks her out of the system.

To unlock her account, just do this:

```
sudo usermod -U katelyn
```

You'll see that the exclamation point has been removed so that she can now log in to her account.

### Using passwd to lock user accounts

You could also lock Katelyn's account with this:

```
sudo passwd -l katelyn
```

This does the same job as `usermod -L`, but in a slightly different manner. For one thing, `passwd -l` will give you some feedback about what's going on, whereas `usermod -L` gives you no feedback at all. On Ubuntu, the feedback looks like this:

```
donnie@ubuntu-steemnode:~$ sudo passwd -l katelyn
 [sudo] password for donnie:
 passwd: password expiry information changed.
 donnie@ubuntu-steemnode:~$
```

On CentOS or AlmaLinux, the feedback looks like this:

```
[donnie@localhost ~]$ sudo passwd -l katelyn
 Locking password for user katelyn.
 passwd: Success
 [donnie@localhost ~]$
```

Also, on a CentOS or AlmaLinux machine, you'll see that `passwd -l` places two exclamation points in front of the password hash, instead of just one. Either way, the effect is the same.

To unlock Katelyn's account, just do this:

```
sudo passwd -u katelyn
```

> In versions of Red Hat or CentOS prior to version 7, `usermod -U` would remove only one of the exclamation points that `passwd -l` places in front of the `shadow` file password hash, thereby leaving the account still locked. No big deal, though, because running `usermod -U` again would remove the second exclamation point.
> 
> > Ever since the introduction of the RHEL 7-type distros, this has been fixed. The `passwd -l` command still places two exclamation points in the `shadow` file, but `usermod -U` now removes both of them. (That's a shame, really, because it ruined a perfectly good demo that I liked to do for my students.)

## Locking the root user account

The cloud is big business nowadays, and it's now quite common to rent a virtual private server from companies such as Rackspace, DigitalOcean, or Microsoft Azure. These can serve a variety of purposes:

*   You can run your own website, where you install your own server software instead of letting a hosting service do it.
*   You can set up a web-based app for other people to access.
*   Recently, I saw a YouTube demo on a crypto-mining channel that showed how to set up a Proof of Stake master node on a rented virtual private server.

One thing that these cloud services have in common is that when you first set up your account and the provider sets up a virtual machine for you, they'll have you log in to the root user account. (It even happens with Ubuntu, even though the root account is disabled on a local installation of Ubuntu.)

I know that there are some folk who just keep logging in to the root account of these cloud-based servers and think nothing of it, but that's really a horrible idea. There are botnets, such as the Hail Mary botnet, that continuously scan the internet for servers that have their Secure Shell port exposed to the internet. When the botnets find one, they'll do a brute-force password attack against the root user account of that server. And yes, the botnets sometimes are successful in breaking in, especially if the root account is set with a weak password.

So, the first thing that you want to do when you set up a cloud-based server is to create a normal user account for yourself and set it up with full `sudo` privileges. Then, log out of the root user account, log in to your new account, and do this:

```
sudo passwd -l root
```

I mean, really, why take the chance of getting your root account compromised?

## Setting up security banners

Something that you really, really don't want is to have a login banner that says something to the effect of *Welcome to our network*. I say that because, quite a few years ago, I attended a mentored SANS course on incident handling. Our instructor told us a story about how a company took a suspected network intruder to court, only to get the case thrown out. The reason? The alleged intruder said, "*Well, I saw the message that said Welcome to the network, so I thought that I really was welcome there."* Yeah, supposedly, that was enough to get the case thrown out.

A few years later, I related that story to the students in one of my Linux admin classes. One student said, "*That makes no sense.* *We all have welcome mats at our front doors, but that doesn't mean that burglars are welcome to come in.*" I have to confess that he had a good point, and I now have to wonder about the veracity of the story.

At any rate, just to be on the safe side, you do want to set up login messages that make clear that only authorized users are allowed to access the system.

### Using the motd file

The `/etc/motd` file will present a message banner to anyone who logs in to a system through Secure Shell. On your CentOS or AlmaLinux machine, an empty `motd` file is already there. On your Ubuntu machine, the `motd` file isn't there, but it's a simple matter to create one. Either way, open the file in your text editor and create your message. Save the file and test it by remotely logging in through Secure Shell. You should see something like this:

```
 maggie@192.168.0.100's password:
 Last login: Sat Oct 7 20:51:09 2017
 Warning: Authorized Users Only!
 All others will be prosecuted.
 [maggie@localhost ~]$
```

> `motd` stands for **Message of the Day**.

Ubuntu comes with a dynamic MOTD system that displays messages from Ubuntu's parent company and messages about the operating system. When you create a new `motd` file in the `/etc` directory, whatever message you put in it will show up at the end of the dynamic output, like so:

```
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-48-generic x86_64)
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
  System information as of Thu Oct 13 06:20:54 PM UTC 2022
  System load:  0.0               Processes:               103
  Usage of /:   47.8% of 9.75GB   Users logged in:         1
  Memory usage: 12%               IPv4 address for enp0s3: 192.168.0.11
  Swap usage:   0%
39 updates can be applied immediately.
To see these additional updates run: apt list --upgradable
Warning!!! Authorized users only!
Last login: Thu Oct 13 17:14:52 2022 from 192.168.0.16
```

The `Warning!!! Authorized users only!` line is what I placed into the `/etc/motd` file.

### Using the issue file

The `issue` file, also found in the `/etc` directory, shows a message on the local terminal, just above the login prompt. A default `issue` file would just contain macro code that would show information about the machine. Here's an example from an Ubuntu machine:

```
Ubuntu 22.04.1 LTS \n \l
```

Or, on a Red Hat-type machine, it would look like this:

```
\S
Kernel \r on an \m
```

On an Ubuntu machine, the banner would look something like this:

![19501_02_03.png](img/file22.png)

19501_02_03.png

On a a Red Hat-type machine, it would look something like this:

![19501_02_04.png](img/file23.png)

19501_02_04.png

You could put a security message in the issue file, and it would show up after a reboot:

![19501_02_05.png](img/file24.png)

19501_02_05.png

In reality, is there really any point in placing a security message in the issue file? If your servers are properly locked away in a server room with controlled access, then probably not. For desktop machines that are out in the open, this would be more useful.

### Using the issue.net file

Just don't. It's for `telnet` logins, and anyone who has `telnet` enabled on their servers is seriously screwing up. However, for some strange reason, the `issue.net` file still hangs around in the `/etc` directory.

## Detecting compromised passwords

Yes, dear hearts, the bad guys do have extensive dictionaries of passwords that either are commonly used or have been compromised. One of the most effective ways of brute-forcing passwords is to use these dictionaries to perform a dictionary attack. This is when the password-cracking tool reads in passwords from a specified dictionary and tries each one until either the list has been exhausted, or until the attack is successful. So, how do you know if your password is on one of those lists? Easy. Just use one of the online services that will check your password for you. One popular site is *Have I Been Pwned?*, which you can see here:

![19501_02_06.png](img/file25.png)

19501_02_06.png

> You can get to *Have I Been Pwned?* here:
> 
> > [https://haveibeenpwned.com](https://haveibeenpwned.com)

All you really have to do is to type in your password, and the service will show if it's on any lists of compromised passwords. But think about it. Do you really want to send your production password to somebody's website? Yeah, I thought not. Instead, let's just send a hash value of the password. Better yet, let's just send enough of the hash to allow the site to find the password in its database, but not so much that they can figure out what your exact password is. We'll do that by using the *Have I Been Pwned?* **Application Programming Interface** (**API**).

To demonstrate the basic principle, let's use `curl`, along with the API, to see a list of password hashes that have `21BD1` as part of their values. (You can do this on any of your virtual machines. I'll just do it on the Fedora workstation that I'm currently using to type this.) Just run this:

```
curl https://api.pwnedpasswords.com/range/21BD1
```

You're going to get a lot of output like this, so I'll just show the first few lines:

```
 [donnie@fedora-teaching ~]$ curl https://api.pwnedpasswords.com/range/21BD1
 0018A45C4D1DEF81644B54AB7F969B88D65:1
 00D4F6E8FA6EECAD2A3AA415EEC418D38EC:2
 011053FD0102E94D6AE2F8B83D76FAF94F6:1
 012A7CA357541F0AC487871FEEC1891C49C:2
 0136E006E24E7D152139815FB0FC6A50B15:3
 01A85766CD276B17DE6DA022AA3CADAC3CE:3
 024067E46835A540D6454DF5D1764F6AA63:3
 02551CADE5DDB7F0819C22BFBAAC6705182:1
 025B243055753383B479EF34B44B562701D:2
 02A56D549B5929D7CD58EEFA97BFA3DDDB3:8
 02F1C470B30D5DDFF9E914B90D35AB7A38F:3
 03052B53A891BDEA802D11691B9748C12DC:6
. . .
. . .
```

Let's pipe this into `wc -l`, a handy counting utility, to see how many matching results we've found:

```
 [donnie@fedora-teaching ~]$ curl https://api.pwnedpasswords.com/range/21BD1 | wc -l
 % Total % Received % Xferd Average Speed Time Time Time Current
 Dload Upload Total Spent Left Speed
 100 20592 0 20592 0 0 197k 0 --:--:-- --:--:-- --:--:-- 199k
 526
 [donnie@fedora-teaching ~]$
```

According to this, we've found 526 matches. But that's not very useful, so let's fancy things up just a bit. We'll do that by creating the `pwnedpasswords.sh` shell script, which looks like this:

```
#!/bin/bash
candidate_password=$1
echo "Candidate password: $candidate_password"
full_hash=$(echo -n $candidate_password | sha1sum | awk '{print substr($1, 0, 32)}')
prefix=$(echo $full_hash | awk '{print substr($1, 0, 5)}')
suffix=$(echo $full_hash | awk '{print substr($1, 6, 26)}')
if curl https://api.pwnedpasswords.com/range/$prefix | grep -i $suffix;
 then echo "Candidate password is compromised";
 else echo "Candidate password is OK for use";
fi
```

Okay, I can't try to turn you into a shell scripting guru at the moment, but here's the simplified explanation:

*   `candidate_password=$1`: This requires you to enter the password that you want to check when you invoke the script.
*   `full_hash=` , `prefix=`, `suffix=`: These lines calculate the SHA1 hash value of the password, and then extract just the portions of the hash that we want to send to the password-checking service.
*   `if curl`: We wrap up with an `if..then..else` structure that sends the selected portions of the password hash to the checking service, and then tells us whether or not the password has been compromised.

After saving the file, add the executable privilege for the user, like so:

```
chmod u+x pwnedpasswords.sh
```

Now, let's see if `TurkeyLips`, my all-time favorite password, has been compromised:

```
 [donnie@fedora-teaching ~]$ ./pwnedpasswords.sh TurkeyLips
 Candidate password: TurkeyLips
 % Total % Received % Xferd Average Speed Time Time Time Current
 Dload Upload Total Spent Left Speed
 0 0 0 0 0 0 0 0 --:--:-- --:--:-- --:--:-- 09FDEDF4CA44D6B432645D6C1D3A8D4A16BD:2
 100 21483 0 21483 0 0 107k 0 --:--:-- --:--:-- --:--:-- 107k
 Candidate password is compromised
 [donnie@fedora-teaching ~]$
```

Yeah, it's been compromised, all right. So, I reckon that I don't want to use that for a production password.

Now, let's try it again, except with a random two-digit number tacked on at the end:

```
 [donnie@fedora-teaching ~]$ ./pwnedpasswords.sh TurkeyLips98
 Candidate password: TurkeyLips98
 % Total % Received % Xferd Average Speed Time Time Time Current
 Dload Upload Total Spent Left Speed
 100 20790 0 20790 0 0 110k 0 --:--:-- --:--:-- --:--:-- 110k
 Candidate password is OK for use
 [donnie@fedora-teaching ~]$
```

Well, it says that this one is okay. Still though, you probably don't want to use such a simple permutation of a password that's known to have been compromised.

> I'd like to take credit for the shell script that I've presented here, but I can't. That was a creation of my buddy, Leo Dorrendorf of the former VDOO Internet of Things security company, which has since been acquired by JFrog. (I've reproduced the script here with his kind permission.)
> 
> > If you're interested in security solutions for your Internet of Things devices, you can check them out here:
> 
> > [https://jfrog.com/security-and-compliance/?vr=1/](https://jfrog.com/security-and-compliance/?vr=1/)
> 
> > Full disclosure: the VDOO/JFrog company has been one of my clients.

Now, having said all of this, I still need to remind you that a passphrase is still better than a password. Not only is a passphrase harder to crack, it's also much less likely to be on anyone's list of compromised credentials.

### Hands-on lab for detecting compromised passwords

In this lab, you'll use the `pwnedpasswords` API in order to check your own passwords:

1.  Use `curl` to see how many passwords there are with the `21BD1` string in their password hashes:

```
curl https://api.pwnedpasswords.com/range/21BD1
```

1.  In the home directory of any of your Linux virtual machines, create the `pwnpassword.sh` script with the following content:

```
#!/bin/bash
candidate_password=$1
echo "Candidate password: $candidate_password"
full_hash=$(echo -n $candidate_password | sha1sum | awk '{print substr($1, 0, 32)}')
prefix=$(echo $full_hash | awk '{print substr($1, 0, 5)}')
suffix=$(echo $full_hash | awk '{print substr($1, 6, 26)}')
if curl https://api.pwnedpasswords.com/range/$prefix | grep -i $suffix;
        then echo "Candidate password is compromised";
        else echo "Candidate password is OK for use";
fi
```

1.  Add the executable permission to the script:

```
chmod u+x pwnedpasswords.sh
```

1.  Run the script, specifying `TurkeyLips` as a password:

```
./pwnedpasswords.sh TurkeyLips
```

1.  Repeat *Step 4* as many times as you like, using a different password each time.

The user management techniques that we've looked at so far work great on a small number of computers. But what if you're working in a large enterprise? We'll look at that next.

## Understanding centralized user management

In an enterprise setting, you'll often have hundreds or thousands of users and computers that you need to manage. So, logging in to each network server or each user's workstation to perform the procedures that we've just outlined would be quite unworkable. (But do bear in mind that you still need those skills.) What we need is a way to manage computers and users from one central location. Space doesn't permit me to give the complete details about the various methods for doing this. So for now, we'll just have to settle for a high-level overview.

### Microsoft Active Directory

I'm not exactly a huge fan of either Windows or Microsoft. But when it comes to Active Directory, I'll have to give credit where it's due. It's a pretty slick product that vastly simplifies the management of very large enterprise networks. And yes, it is possible to add Unix/Linux computers and their users to an Active Directory domain.

> I've been keeping a dark secret, and I hope that you won't hate me for it. Before I got into Linux, I obtained my MCSE certification for Windows Server 2003\. Mostly, my clients work with nothing but Linux computers, but I occasionally do need to use my MCSE skills. Several years ago, a former client needed me to set up a Linux-based Nagios server as part of a Windows Server 2008 domain, so that its users would be authenticated by Active Directory. It took me a while to get it figured out, but I finally did, and my client was happy.

Unless you wear many hats, as I sometimes have to do, you—as a Linux administrator—probably won't need to learn how to use Active Directory. Most likely, you'll just tell the Windows Server administrators what you need, and let them take care of it.

I know, you've been chomping at the bit to see what we can do with a Linux server. So, here goes.

## Samba on Linux

Samba is a Unix/Linux daemon that can serve three purposes:

*   Its primary purpose is to share directories from a Unix/Linux server with Windows workstations. The directories show up in the Windows File Explorer as if they were being shared from other Windows machines.
*   It can also be set up as a network print server.
*   It can also be set up as a Windows domain controller.

You can install Samba version 3 on a Linux server, and set it up to act as an old-style Windows NT domain controller. It's a rather complex procedure, and it takes a while. Once it's done, you can join both Linux and Windows machines to the domain and use the normal Windows user management utilities to manage users and groups.

One of the Linux community's Holy Grails was to figure out how to emulate Active Directory on a Linux server. That became something of a reality just a few years ago, with the introduction of Samba version 4\. But setting it up is a very complex procedure, and isn't something that you'll likely enjoy doing. So, perhaps we should keep searching for something even better.

### FreeIPA/Identity Management on RHEL-type distros

Several years ago, the Red Hat company introduced FreeIPA as a set of packages for Fedora. Why Fedora? It's because they wanted to give it a thorough test on Fedora before making it available for actual production networks. It's now available for RHEL 7 through RHEL 9 and all of their offspring, including CentOS and AlmaLinux. This is what IPA stands for:

*   Identity
*   Policy
*   Audit

It's something of an answer to Microsoft's Active Directory, but it still isn't a complete one. It does some cool stuff, but it's still very much a work in progress. The coolest part about it is how simple it is to install and set up. All it really takes is to install the packages from the normal repositories, open the proper firewall ports, and then run a setup script. Then, you're all set to start adding users and computers to the new domain via FreeIPA's web interface. Here, I'm adding Cleopatra, my gray-and-white tabby kitty:

![19501_02_07.png](img/file26.png)

19501_02_07.png

Although you can add Windows machines to a FreeIPA domain, it's not recommended. But, starting with RHEL/CentOS 7.1, you can use FreeIPA to create cross-domain trusts with an Active Directory domain.

> The official name of this program is FreeIPA. But, for some strange reason, the Red Hat folk refuse to mention that name in their documentation. They always just refer to it as either Identity Management or IdM.

That's pretty much it for the user management topic. Let's summarize, and then move on to the next chapter.

## Summary

We covered a lot of ground in this chapter, and hopefully you found some suggestions that you can actually use. We started out by showing you the dangers of always logging in as the root user and how you should use `sudo` instead. In addition to showing you the basics of `sudo` usage, we also looked at some good `sudo` tips and tricks.

We moved on to user management by looking at how to lock down users' home directories, how to enforce strong password policies, and how to enforce account and password expiration policies. Then, we talked about a way to prevent brute-force password attacks, how to manually lock out user accounts, how to set up security banners, and how to check for compromised passwords. We wrapped things up with a brief overview of central user management systems.

In the next chapter, we'll look at how to work with various firewall utilities. I'll see you there.

## Questions

1.  What is the best way to grant administrative privilege to users?
    1.  Give every administrative user the root user password.
    2.  Add each administrative user to either the `sudo` group or the `wheel` group.
    3.  Create `sudo` rules that only allow administrative users to do the tasks that are directly related to their jobs.
    4.  Add each administrative user to the `sudoers` file and grant them full administrative privileges.
2.  Which of the following is true?
    1.  When users log in as the root user, all the actions that they perform will be recorded in the `auth.log` or the `secure` log file.
    2.  When users use `sudo`, all the actions that they perform will be recorded in the `messages` or the `syslog` file.
    3.  When users log in as the root user, all the actions that they perform will be recorded in the `messages` or the `syslog` file.
    4.  When users use `sudo`, all the actions that they perform will be recorded in the `auth.log` or the `secure` log file.
3.  In which file would you configure complex password criteria?
4.  When using the `useradd` utility, what should the `UMASK` setting be in the `/etc/login.defs` file?
5.  When using the `adduser` utility, how would you configure the `/etc/adduser.conf` file so that new users' home directories will prevent other users from accessing them?
6.  What change did the National Institute for Standards and Technology recently make to their recommended password policy?
7.  Which of the following methods would you use to create `sudo` rules for other users?
    1.  Open the `/etc/sudoers` file in your favorite text editor.
    2.  Open the `/etc/sudoers` file with `visudo`.
    3.  Add a `sudoers` file to each user's home directory.
    4.  Open the `/var/spool/sudoers` file with `visudo`.
8.  Which three of the following utilities can you use to set user account expiry data?
    1.  `useradd`
    2.  `adduser`
    3.  `usermod`
    4.  `chage`
9.  Why might you want to lock out the user account of a former employee, rather than to delete it?
    1.  It's easier to lock an account than it is to delete it.
    2.  It takes too long to delete an account.
    3.  It's not possible to delete a user account.
    4.  Deleting a user account, along with the users' files and mail spool, might get you into trouble with the law.
10.  You've just created a user account for Samson, and you now want to force him to change his password the first time that he logs in. Which two of the following commands will do that?
    1.  `sudo chage -d 0 samson`
    2.  `sudo passwd -d 0 samson`
    3.  `sudo chage -e samson`
    4.  `sudo passwd -e samson`
11.  Which one of the following represents best security practice?
    1.  Always give the root user password to all users who need to perform administrative tasks.
    2.  Always give full `sudo` privileges to all users who need to perform administrative tasks.
    3.  Always just give specific, limited `sudo` privileges to all users who need to perform administrative tasks.
    4.  Always edit the `sudoers` file in a normal text editor, such as nano, vim, or emacs.
12.  Which of the following statements is true?
    1.  `sudo` can only be used on Linux.
    2.  `sudo` can be used on Linux, Unix, and BSD operating systems.
    3.  When a user performs a task using `sudo`, the task does not get recorded in a security log.
    4.  When using `sudo`, users must enter the root user password.
13.  You want specific users to edit a specific system configuration file, but you don't want them to use a shell escape that would allow them to perform other administrative tasks. Which of the following would you do?
    1.  In the `sudoers` file, specify that the users can only use vim to open a specific configuration file.
    2.  In the `sudoers` file, specify that the users can use `sudoedit` to edit a specific configuration file.
    3.  In the `sudoers` file, specify the `no shell escape` option for these users.
    4.  In the `sudoers` file, place these users into a group that does not have shell escape privileges.
14.  Which one of the following is an advantage that the `adduser` utility has over the traditional `useradd` utility?
    1.  `adduser` can be used in shell scripts.
    2.  `adduser` is available for all Linux distributions.
    3.  `adduser` has an option that allows you to encrypt a user's home directory as you create the user account.
    4.  `adduser` is also available for Unix and BSD.
15.  In the newest Linux distributions, what is the name of the PAM that you would use to enforce strong passwords?
    1.  cracklib
    2.  passwords
    3.  secure
    4.  pwquality

## Further reading

*   You might not need complex, alphanumeric passwords after all: [https://www.pcmag.com/news/355496/you-might-not-need-complex-alphanumeric-passwords-after-all](https://www.pcmag.com/news/355496/you-might-not-need-complex-alphanumeric-passwords-after-all)
*   The new NIST Guidelines-We had it all wrong before: [https://www.riskcontrolstrategies.com/2018/01/08/new-nist-guidelines-wrong/](https://www.riskcontrolstrategies.com/2018/01/08/new-nist-guidelines-wrong/)
*   Linux Privilege Escalation exploiting sudo rights: [https://medium.com/schkn/linux-privilege-escalation-using-text-editors-and-files-part-1-a8373396708d](https://medium.com/schkn/linux-privilege-escalation-using-text-editors-and-files-part-1-a8373396708d)
*   sudo Main Page: [https://www.sudo.ws/](https://www.sudo.ws/)
*   Granting sudo access: [https://www.golinuxhub.com/2013/12/how-to-give-permission-to-user-to-run.html](https://www.golinuxhub.com/2013/12/how-to-give-permission-to-user-to-run.html)
*   Linux user management: [https://www.youtube.com/playlist?list=PL6IQ3nFZzWfpy2gISpCppFk3UQVGf_x7G](https://www.youtube.com/playlist?list=PL6IQ3nFZzWfpy2gISpCppFk3UQVGf_x7G)
*   The FreeIPA Project home page: [https://www.freeipa.org/page/Main_Page](https://www.freeipa.org/page/Main_Page)
*   RHEL 9 Documentation (Scroll down to the Identity Management section) [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9)
*   RHEL 8 Documentation (Scroll down to the Identity Management section) : [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/)
*   RHEL 7 Documentation (Scroll down to the Identity Management section): [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/)
*   pam_faillock: Lock user account after X failed attempts: [https://www.golinuxcloud.com/pam-faillock-lock-user-account-linux/](https://www.golinuxcloud.com/pam-faillock-lock-user-account-linux/)