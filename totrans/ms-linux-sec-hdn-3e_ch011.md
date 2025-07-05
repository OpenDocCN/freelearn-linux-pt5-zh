## Section 3: Advanced System Hardening Techniques

This section will teach you how to harden a Linux system with **Mandatory Access Control** (**MAC**), security profiles, and process isolation techniques. Audit a Linux system with auditd and logging services.

The section contains the following chapters:

*   *Chapter 10*, *Implementing Mandatory Access Control with SELinux and AppArmor*
*   *Chapter 11*, *Kernel Hardening and Process Isolation*
*   *Chapter 12*, *Scanning, Auditing, and Hardening*
*   *Chapter 13*, *Logging and Log Security*
*   *Chapter 14*, *Vulnerability Scanning and Intrusion Detection*
*   *Chapter 15*, *Blocking Applications with fapolicyd*
*   *Chapter 16*, *Security Tips and Tricks for the Busy Bee*

# 10 Implementing Mandatory Access Control with SELinux and AppArmor

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file60.png)

As we saw in previous chapters, **Discretionary Access Control** (**DAC**) allows users to control who can access their own files and directories. But what if your company needs to have more administrative control over who accesses what? For this, we need some sort of **Mandatory Access Control** (**MAC**).

The best way I know to explain the difference between DAC and MAC is to hearken back to my Navy days. I was riding submarines at the time, and I had to have a Top Secret clearance to do my job. With DAC, I had the physical ability to take one of my Top Secret books to the mess decks, and hand it to a cook who didn't have that level of clearance. With MAC, there were rules that prevented me from doing so. On operating systems, things work pretty much the same way.

There are several different MAC systems that are available for Linux. The two that we'll cover in this chapter are SELinux and AppArmor. We'll look at what both of them are, how to configure them, and how to troubleshoot them.

In this chapter, we'll cover the following topics:

*   What SELinux is and how it can benefit a systems administrator
*   How to set security contexts for files and directories
*   How to use setroubleshoot to troubleshoot SELinux problems
*   Looking at SELinux policies and how to create custom policies
*   What AppArmor is and how it can benefit a systems administrator
*   Looking at AppArmor policies
*   Working with AppArmor command-line utilities
*   Troubleshooting AppArmor problems
*   Exploiting a system with an evil Docker container

Let's start out by looking at SELinux and how you can benefit from it.

## How SELinux can benefit a systems administrator

SELinux is a free open source software project that was developed by the U.S. National Security Agency. While it can theoretically be installed on any Linux distribution, Red Hat-type distributions are the only ones that come with it already set up and enabled. It uses code in Linux kernel modules, along with extended filesystem attributes, to help ensure that only authorized users and processes can access either sensitive files or system resources. There are three ways in which SELinux can be used:

*   It can help prevent intruders from exploiting a system.
*   It can be used to ensure that only users with the proper security clearance can access files that are labeled with a security classification.
*   In addition to MAC, SELinux can also be used as a type of role-based access control.

In this chapter, I'll only be covering the first of these three uses because that is the most common way in which SELinux is used. There's also the fact that covering all three of these uses would require writing a whole book, which I don't have space to do here.

> **Tip**
> 
> > If you go through this introduction to SELinux and find that you still need more SELinux information, you'll find whole books and courses on just this subject on the Packt Publishing website.

So, how can SELinux benefit the busy systems administrator? Well, you might remember when, a few years ago, news about the Shellshock bug hit the world's headlines. Essentially, Shellshock was a bug in the Bash shell that allowed intruders to break into a system and to exploit it by gaining root privileges. For systems that were running SELinux, it was still possible for the bad guys to break in, but SELinux would have prevented them from successfully running their exploits.

SELinux is also yet another mechanism that can help protect data in users' home directories. If you have a machine that's set up as a Network File System server, a Samba server, or a web server, SELinux will prevent those daemons from accessing users' home directories, unless you explicitly configure SELinux to allow that behavior.

On web servers, you can use SELinux to prevent the execution of malicious CGI scripts or PHP scripts. If you don't need your server to run CGI or PHP scripts, you can disable them in SELinux.

With Docker and without SELinux, it's trivially easy for a normal user to break out of a Docker container and gain root-level access to the host machine. As we'll see at the end of this chapter, SELinux is a useful tool for hardening servers that run Docker containers.

So now, you're likely thinking that everyone would use such a great tool, right? Sadly, that's not the case. In its beginning, SELinux got a reputation for being difficult to work with, and many administrators would just disable it. In fact, a lot of tutorials you see on the web or on YouTube have disabling SELinux as the first step. In this section, I'd like to show you that things have improved and that SELinux no longer deserves its bad reputation.

## Setting security contexts for files and directories

Think of SELinux as a glorified labeling system. It adds labels, known as security contexts, to files and directories through extended file attributes. It also adds the same type of label, known as domains, to system processes. To see these contexts and domains on your CentOS or AlmaLinux machines, use the `-Z` option with either `ls` or `ps.` For example, files and directories in my own home directory would look like the following:

```
[donnie@localhost ~]$ ls -Z
drwxrwxr-x. donnie donnie unconfined_u:object_r:user_home_t:s0 acl_demo_dir
-rw-rw-r--. donnie donnie unconfined_u:object_r:user_home_t:s0 yum_list.txt
[donnie@localhost ~]$
```

Processes on my system would look something like the following:

```
[donnie@localhost ~]$ ps -Z
LABEL                             PID TTY          TIME CMD
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 1322 pts/0 00:00:00 bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 3978 pts/0 00:00:00 ps
[donnie@localhost ~]$
```

Now, let's break this down. In the outputs of both the `ls -Z` and `ps -Z` commands, we have the following parts:

*   **The SELinux user**: In both cases, the SELinux user is the generic `unconfined_u`.
*   **The SELinux role**: In the `ls -Z` example, we see that the role is `object_r`, and in the `ps -Z` example it's `unconfined_r`.
*   **The type**: It's `user_home_t` in the `ls -Z` output and `unconfined_t` in the `ps -Z` output.
*   **The sensitivity**: In the `ls -Z` output it's `s0`. In the `ps -Z` output, it's `s0-s0`.
*   **The category**: We don't see a category in the `ls -Z` output, but we do see `c0.c1023` in the `ps -Z` output.

Out of all of the preceding security context and security domain components, the only one that interests us now is the **type**. For the purposes of this chapter, we're only interested in covering what a normal Linux administrator would need to know to keep intruders from exploiting the system, and the **type** is the only one of these components that we need to use for that. All of the other components come into play when we set up advanced, security classification-based access control and role-based access control.

Okay, the following is a somewhat over-simplified explanation of how this helps a Linux administrator maintain security. What we want is for system processes to only access objects that we allow them to access. (System processes include things such as the web server daemon, the FTP daemon, the Samba daemon, and the Secure Shell daemon. Objects include things such as files, directories, and network ports.) To achieve this, we'll assign a **type** to all of our processes and all of our objects. We'll then create policies that define which process types can access which object types.

Fortunately, whenever you install any Red Hat-type distribution, pretty much all of the hard work has already been done for you. Red Hat-type distributions all come with SELinux already enabled and set up with the **targeted** policy. Think of this targeted policy as a somewhat relaxed policy, that allows a casual desktop user to sit down at the computer and actually conduct business without having to tweak any SELinux settings. But if you're a server administrator, you may find yourself having to tweak this policy in order to allow server daemons to do what you need them to do.

> The targeted policy, which comes installed by default, is what a normal Linux administrator will use in his or her day-to-day duties. If you look in the repositories of your CentOS virtual machine, you'll see that there are also several others, which we won't cover in this book.

### Installing the SELinux tools

For some bizarre reason that I'll never understand, the tools that you need to administer SELinux don't get installed by default, even though SELinux itself does. So, the first thing you'll need to do on your CentOS virtual machine is to install them.

On CentOS 7, run this command:

```
sudo yum install setools policycoreutils policycoreutils-python
```

On CentOS 8, run this command:

```
sudo dnf install setools policycoreutils policycoreutils-python-utils
```

Later on in this chapter, in the *Troubleshooting with setroubleshoot* section, we'll look at how to use `setroubleshoot` to help diagnose SELinux problems. In order to have some cool error messages to look at when we get there, go ahead and install `setroubleshoot` now, and activate it by restarting the `auditd` daemon. (There's no `setroubleshoot` daemon, because `setroubleshoot` is meant to be controlled by the `auditd` daemon.) Install `setroubleshoot` like so.

For CentOS 7, use these commands:

```
sudo yum install setroubleshoot
sudo service auditd restart
```

For AlmaLinux 8 and 9, use these commands:

```
sudo dnf install setroubleshoot
sudo service auditd restart
```

One of the little systemd quirks that we have to deal with on Red Hat-type systems is that you can't stop or restart the `auditd` daemon with the normal `systemctl` command. However, the old-fashioned `service` command works. For some reason that I don't understand, the Red Hat folk configured the `auditd` service file to disable the normal systemd way of doing things.

> Depending on the type of installation that you chose when installing CentOS or AlmaLinux, you might or might not already have `setroubleshoot` installed. To be sure, go ahead and run the command to install it. It won't hurt anything if `setroubleshoot` is already there.

You now have what you need to get started. Let's now look at what SELinux can do for a busy web server administrator.

### Creating web content files with SELinux enabled

Now, let's look at what can happen if you have web content files that are set with the wrong SELinux type. First, we'll install, enable, and start the Apache web server on our CentOS virtual machines. (Note that including the --now option allows us to enable and start a daemon all in one single step.) Do the following on CentOS 7:

```
sudo yum install httpd
sudo systemctl enable --now httpd
```

On CentOS 8, use the following command:

```
sudo dnf install httpd
sudo systemctl enable --now httpd
```

If you haven't done so already, configure the firewall to allow access to the web server:

```
[donnie@localhost ~]$ sudo firewall-cmd --permanent --add-service=http
success
[donnie@localhost ~]$ sudo firewall-cmd --reload
success
[donnie@localhost ~]$
```

When we look at the SELinux information for Apache processes, we'll see the following:

```
[donnie@localhost ~]$ ps ax -Z | grep httpd
system_u:system_r:httpd_t:s0     3689 ?        Ss     0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     3690 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     3691 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     3692 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     3693 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0     3694 ?        S      0:00 /usr/sbin/httpd -DFOREGROUND
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 3705 pts/0 R+   0:00 grep --color=auto httpd
```

As I said before, we're not interested in the user or the role. However, we are interested in the type, which in this case is `httpd_t`.

On Red Hat-type systems, we would normally place web content files in the `/var/www/html/` directory. Let's look at the SELinux context for that `html` directory:

```
[donnie@localhost www]$ pwd
/var/www
[donnie@localhost www]$ ls -Zd html/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 html/
[donnie@localhost www]$
```

The type is `httpd_sys_content`, so it stands to reason that the `httpd` daemon should be able to access this directory. It's currently empty, so let's `cd` into it and create a simple index file:

```
[donnie@localhost www]$ cd html
[donnie@localhost html]$ pwd
/var/www/html
[donnie@localhost html]$ sudo vim index.html
```

Here's what I'll put into the file:

```
<html>
<head>
<title>
Test of SELinux
</title>
</head>
<body>
Let's see if this SELinux stuff really works!
</body>
</html>
```

Okay, as I said, it's simple, since my HTML hand-coding skills aren't what they used to be. But still, it serves our present purposes.

Looking at the SELinux context, we see that the file has the same type as the `html` directory:

```
[donnie@localhost html]$ ls -Z
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 index.html
[donnie@localhost html]$
```

I can now navigate to this page from the web browser of my trusty OpenSUSE workstation:

![](img/file61.png)

Now, though, let's see what happens if I decide to create content files in my own home directory and then move them to the `html` directory. First, let's see what the SELinux context is for my new file:

```
[donnie@localhost ~]$ pwd
/home/donnie
[donnie@localhost ~]$ ls -Z index.html
-rw-rw-r--. donnie donnie unconfined_u:object_r:user_home_t:s0 index.html
[donnie@localhost ~]$
```

The context type is now `user_home_t`, which is a sure-fire indicator that I created this in my home directory. I'll now move the file to the `html` directory, overwriting the old file:

```
[donnie@localhost ~]$ sudo mv index.html /var/www/html/
[sudo] password for donnie:
[donnie@localhost ~]$ cd /var/www/html
[donnie@localhost html]$ ls -Z
-rw-rw-r--. donnie donnie unconfined_u:object_r:user_home_t:s0 index.html
[donnie@localhost html]$
```

Even though I moved the file over to the /var/www/html directory, the SELinux type is still associated with users' home directories. Now, I'll go to the browser of my host machine to refresh the page:

![](img/file62.png)

So, I have a slight problem. The type that's assigned to my file doesn't match the type of the httpd daemon processes, so SELinux doesn't allow the `httpd` processes to access the file.

> **Tip**
> 
> > Had I copied the file to the `html` directory instead of moving it, the SELinux context would have automatically changed to match that of the destination directory.

### Fixing an incorrect SELinux context

Okay, so I have this web content file that nobody can access, and I really don't feel up to creating a new one. So, what do I do? Actually, we have three different utilities for fixing this:

*   **chcon**
*   **restorecon**
*   **semanage**

Let's look at each of them.

#### Using chcon

There are two ways to use `chcon` to fix an incorrect SELinux type on a file or directory. The first is to just manually specify the proper type:

```
[donnie@localhost html]$ sudo chcon -t httpd_sys_content_t index.html
[sudo] password for donnie:
[donnie@localhost html]$ ls -Z
-rw-rw-r--. donnie donnie unconfined_u:object_r:httpd_sys_content_t:s0 index.html
[donnie@localhost html]$
```

We can use `chcon` to change any part of the context, but as I keep saying we're only interested in the type, which gets changed with the `-t` option. You can see in the `ls -Z` output that the command was successful.

The other way to use `chcon` is to reference a file that has the proper context. For demo purposes, I changed the `index.html` file back to the home directory type and created a new file within the `/var/www/html/` directory:

```
[donnie@localhost html]$ ls -Z
-rw-rw-r--. donnie donnie unconfined_u:object_r:user_home_t:s0 index.html
-rw-r--r--. root   root   unconfined_u:object_r:httpd_sys_content_t:s0 some_file.html
[donnie@localhost html]$
```

As you can see, any files that I create within this directory will automatically have the proper SELinux context settings. Now, let's use that new file as a reference in order to set the proper context on the `index.html` file:

```
[donnie@localhost html]$ sudo chcon --reference some_file.html index.html
[sudo] password for donnie:
[donnie@localhost html]$ ls -Z
-rw-rw-r--. donnie donnie unconfined_u:object_r:httpd_sys_content_t:s0 index.html
-rw-r--r--. root   root   unconfined_u:object_r:httpd_sys_content_t:s0 some_file.html
[donnie@localhost html]$
```

So, I used the `--reference` option and specified the file that I wanted to use as a reference. The file that I wanted to change is listed at the end of the command. Now, that's all to the good, but I want to find an easier way that doesn't require quite as much typing. After all, I am an old man, and I don't want to overexert myself. So, let's take a look at the `restorecon` utility.

#### Using restorecon

Using `restorecon` is easy. Just type `restorecon`, followed by the name of the file that you need to change. Once again, I've changed the context of the `index.html` file back to the home directory type. This time, though, I'm using `restorecon` to set the correct type:

```
[donnie@localhost html]$ ls -Z
-rw-rw-r--. donnie donnie unconfined_u:object_r:user_home_t:s0 index.html
[donnie@localhost html]$ sudo restorecon index.html
[donnie@localhost html]$ ls -Z
-rw-rw-r--. donnie donnie unconfined_u:object_r:httpd_sys_content_t:s0 index.html
[donnie@localhost html]$
```

And that's all there is to it.

> **Tip**
> 
> > You can also use `chcon` and `restorecon` to change the context of an entire directory and its contents. For either one, just use the `-R` option. The following is an example:

```
sudo chcon -R -t httpd_sys_content_t /var/www/html/
sudo restorecon -R /var/www/html/
```

> (Remember: `-R` stands for recursive.)

There's still one last thing to take care of, even though it isn't really affecting our ability to access this file. That is, I need to change ownership of the file to the Apache user:

```
[donnie@localhost html]$ sudo chown apache: index.html
[sudo] password for donnie:
[donnie@localhost html]$ ls -l
total 4
-rw-rw-r--. 1 apache apache 125 Nov 22 16:14 index.html
[donnie@localhost html]$
```

Let's now look at the final utility which is `semanage`.

#### Using semanage

In the scenario I've just presented, either `chcon` or `restorecon` will suit your needs just fine. The active SELinux policy mandates what security contexts in certain directories are supposed to look like. As long as you're using `chcon` or `restorecon` within directories that are defined in the active SELinux policy, you're good. But let's say that you've created a directory elsewhere that you want to use to serve out web content files. You would need to set the `httpd_sys_content_t` type on that directory and all of the files within it. However, if you use `chcon` or `restorecon` for that, the change won't survive a system reboot. To make the change permanent, you'll need to use `semanage`.

Let's say that, for some strange reason, I want to serve web content out of a directory that I've created in the `/home/` directory:

```
[donnie@localhost home]$ pwd
/home
[donnie@localhost home]$ sudo mkdir webdir
[sudo] password for donnie:
[donnie@localhost home]$ ls -Zd webdir
drwxr-xr-x. root root unconfined_u:object_r:home_root_t:s0 webdir
[donnie@localhost home]$
```

Because I had to use my `sudo` powers to create the directory here, it's associated with the root user's `home_root_t` type, instead of the normal `user_home_dir_t` type. Any files that I create within this directory will have the same type:

```
[donnie@localhost webdir]$ ls -Z
-rw-r--r--. root root unconfined_u:object_r:home_root_t:s0 index.html
[donnie@localhost webdir]$
```

The next step is to use `semanage` to add a permanent mapping of this directory and the `httpd_sys_content_t` type to the active policy's context list:

```
[donnie@localhost home]$ sudo semanage fcontext -a -t httpd_sys_content_t "/home/webdir(/.*)?"
[donnie@localhost home]$ ls -Zd /home/webdir
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 /home/webdir
[donnie@localhost home]$
```

Okay, here's the breakdown of the `semanage` command:

*   **fcontext**: Because `semanage` has many purposes, we have to specify that we want to work with a file context.
*   **-a**: This specifies that we're adding a new record to the context list for the active SELinux policy.
*   **-t**: This specifies the type that we want to map to the new directory. In this case, we're creating a new mapping with the `httpd_sys_content` type.
*   **/home/webdir(/.*)?**: This bit of gibberish is what's known as a **regular expression**. I can't go into the nitty-gritty details of regular expressions here, so suffice it to say that **Regular Expressions** is a language that we use to match text patterns. (And yes, I did mean to say *is* instead of *are*, since Regular Expressions is the name of the overall language.) In this case, I had to use this particular regular expression in order to make this `semanage` command recursive because `semanage` doesn't have the `-R` option switch. With this regular expression, I'm saying that I want anything that gets created in this directory to have the same SELinux type as the directory itself.

The final step is to do a `restorecon -R` on this directory to ensure that the proper labels have been set:

```
[donnie@localhost home]$ sudo restorecon -R webdir
[donnie@localhost home]$ ls -Zd /home/webdir
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 /home/webdir
[donnie@localhost home]$
```

Yeah, I know. You're looking at this and saying, "But this `ls -Zd` output looks the same as it did after you did the semanage command." And you're right. After running the `semanage` command, the type seems to be set correctly. But the `semanage-fcontext` man page says to run `restorecon` anyway, so I did.

> **Tip**
> 
> > For more information on how to use `semanage` to manage security contexts, refer to the relevant man page by entering `man semanage-fcontext`.

#### Hands-on lab – SELinux type enforcement

In this lab, you'll install the Apache web server and the appropriate SELinux tools. You'll then view the effects of having the wrong SELinux type assigned to a web content file if you're ready, let's go:

1.  Install Apache, along with all the required SELinux tools on CentOS 7:

```
sudo yum install httpd setroubleshoot setools policycoreutils policycoreutils-python
```

On CentOS 8, use the following command:

```
sudo dnf install httpd setroubleshoot setools policycoreutils policycoreutils-python-utils
```

1.  Activate `setroubleshoot` by restarting the `auditd` service:

```
sudo service auditd restart
```

1.  Enable and start the Apache service and open port `80` on the firewall:

```
sudo systemctl enable --now httpd
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
```

1.  In the `/var/www/html/` directory, create an `index.html` file with the following contents:

```
<html>
   <head>
      <title>SELinux Test Page</title>
   </head>
   <body>
      This is a test of SELinux.
   </body>
</html>
```

1.  View the information about the `index.html` file:

```
ls -Z index.html
```

1.  In your host machine's web browser, navigate to the IP address of the CentOS virtual machine. You should be able to view the page.
2.  Induce an SELinux violation by changing the type of the `index.html` file to something that's incorrect:

```
sudo chcon -t tmp_t index.html
ls -Z index.html
```

1.  Go back to your host machine's web browser and reload the document. You should now see a `Forbidden` message.
2.  Use `restorecon` to change the file back to its correct type:

```
sudo restorecon index.html
```

1.  Reload the page in your host machine's web browser. You should now be able to view the page.
2.  End of lab.

Now that we've seen how to use basic SELinux commands, let's look at a cool tool that makes troubleshooting much easier.

## Troubleshooting with setroubleshoot

So, you're now scratching your head and saying, *When I can't access something that I should be able to, how do I know that it's an SELinux problem?* Ah, I'm glad you asked.

### Viewing setroubleshoot messages

Whenever something happens that violates an SELinux rule, it gets logged in the `/var/log/audit/audit.log` file. Tools are available that can let you directly read that log, but to diagnose SELinux problems it's way better to use `setroubleshoot`. The beauty of `setroubleshoot` is that it takes cryptic, hard-to-interpret SELinux messages from the `audit.log` file and translates them into plain, natural language. The messages that it sends to the `/var/log/messages` file even contain suggestions about how to fix the problem. To show how this works, let's go back to our problem where a file in the `/var/www/html/` directory has been assigned the wrong SELinux type. Of course, we knew right away what the problem was because there was only one file in that directory and a simple `ls -Z` showed what was wrong with it. However, let's ignore that for the moment and say that we didn't know what the problem was. By opening the `/var/log/messages` file in `less` and searching for `sealert`, we'll find this message:

```
Nov 26 21:30:21 localhost python: SELinux is preventing httpd from open access on the file /var/www/html/index.html.#012#012*****  Plugin restorecon (92.2 confidence) suggests   ************************#012#012If you want to fix the label. #012/var/www/html/index.html default label should be httpd_sys_content_t.#012Then you can run restorecon.#012Do#012# /sbin/restorecon -v /var/www/html/index.html#012#012*****  Plugin catchall_boolean (7.83 confidence) suggests   ******************#012#012If you want to allow httpd to read user content#012Then you must tell SELinux about this by enabling the 'httpd_read_user_content' boolean.#012#012Do#012setsebool -P httpd_read_user_content 1#012#012*****  Plugin catchall (1.41 confidence) suggests   **************************#012#012If you believe that httpd should be allowed open access on the index.html file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -i my-httpd.pp#012
```

The first line of this message tells us what the problem is. It's saying that SELinux is preventing us from accessing the `/var/www/html/index.html` file because it's set with the wrong type. It then gives us several suggestions on how to fix the problem, with the first one being to run the `restorecon` command, as I've already shown you how to do.

> **Tip**
> 
> > A good rule-of-thumb to remember when reading these setroubleshoot messages is that the first suggestion in the message is normally the one that will fix the problem.

### Using the graphical setroubleshoot utility

So far, I've only talked about using setroubleshoot on text-mode servers. After all, it's very common to see Linux servers running in text-mode, so all of us Linux folk have to be text-mode warriors. But on desktop systems or on servers that have a desktop interface installed, there is a graphical utility that will automatically alert you when setroubleshoot detects a problem:

![](img/file63.png)

Click on that alert icon, and you'll see something like this:

![](img/file64.png)

Click the **Troubleshoot** button, and you'll see a list of suggestions on how to fix the problem:

![](img/file65.png)

> Note that these screenshots are from a CentOS 7 machine, but they would look the same on either an AlmaLinux 8 or an AlmaLinux 9 machine.

As is often the case with GUI thingies, this is mostly self-explanatory, so you shouldn't have any problem figuring it out.

### Troubleshooting in permissive mode

If you're dealing with a simple problem like the one I've just shown you, then you can probably assume that you can safely do what the first suggestion in the setroubleshoot message tells you to do. But there will be times when things get a bit more complex, where you might have more than one problem. For times like these, you need to use permissive mode.

When you first install your Red Hat or CentOS system, SELinux is in enforcing mode, which is the default. This means that SELinux will actually stop actions that are in violation of the active SELinux policy. This also means that, if you have multiple SELinux problems when you try to perform a certain action, SELinux will stop the action from taking place after the first violation occurs. When it happens, SELinux won't even see the remaining problems, and they won't show up in the `messages` log file. If you try to troubleshoot these types of problem while in enforcing mode, you'll be like the proverbial dog who chases its own tail. You'll go round and round and will accomplish nothing.

In permissive mode, SELinux allows actions that violate policy to occur, but it will log them. By switching to permissive mode and doing something to induce the problem that you were seeing, the prohibited actions will take place but `setroubleshoot` will log all of them in the `messages` file. This way, you'll get a better view of what you need to do to get things working properly.

First, let's use `getenforce` to verify what our current mode is:

```
[donnie@localhost ~]$ sudo getenforce
Enforcing
[donnie@localhost ~]$
```

Now, let's temporarily place the system into permissive mode:

```
[donnie@localhost ~]$ sudo setenforce 0
[donnie@localhost ~]$ sudo getenforce
Permissive
[donnie@localhost ~]$
```

When I say *temporarily*, I mean that this will only last until you do a system reboot. After a reboot, you'll be back in enforcing mode. Also, note that a `0` after `setenforce` denotes that I'm setting permissive mode. To get back to enforcing mode after you're done with troubleshooting, replace the `0` with a `1`:

```
[donnie@localhost ~]$ sudo setenforce 1
[donnie@localhost ~]$ sudo getenforce
Enforcing
[donnie@localhost ~]$
```

We're now back in enforcing mode.

At times, you may need to make permissive mode persist after a system reboot. An example of this would be if you ever have to deal with a system that has had SELinux disabled for a long period of time. In a case like that, you wouldn't want to just put SELinux into enforcing mode and reboot. If you try that, it will take forever for the system to properly create the file and directory labels that make SELinux work, and the system might lock up before it's done. By placing the system into permissive mode first, you'll avoid having the system lock up, although it will still take a long time for the relabeling process to complete.

To make permissive mode persistent across system reboots, you'll edit the `selinux` file in the `/etc/sysconfig/` directory. Here's what it looks like by default:

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

The two important things you see here are that SELinux is in enforcing mode, and that it's using the targeted policy. To switch to permissive mode, just change the SELINUX= line, and save the file:

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of three two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

The `sestatus` utility shows us lots of cool information about what's going on with SELinux:

```
[donnie@localhost ~]$ sudo sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          permissive
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28
[donnie@localhost ~]$
```

The two items that interest us here are the current mode and the mode from a configuration file. By changing the configuration file to permissive, we haven't changed the current running mode. So, we're still in enforcing mode. The switch to permissive won't happen until I either reboot this machine or until I manually switch by issuing a `sudo setenforce 0` command. And of course, you don't want to stay in permissive mode forever. As soon as you no longer need permissive mode, change the configuration file back to **enforcing** and do `sudo setenforce 1` to change the running mode.

## Working with SELinux policies

So far, all we've looked at is what happens when we have an incorrect SELinux type set on a file and what to do to set the correct type. Another problem we may have would comes about if we need to allow an action that is prohibited by the active SELinux policy.

### Viewing Booleans

Booleans are part of what makes up an SELinux policy, and each Boolean represents a binary choice. In SELinux policies, a Boolean either allows something or it prohibits something. To see all Booleans on your system, run the `getsebool -a` command. (It's a long list, so I'll only show partial output here.):

```
[donnie@localhost ~]$ getsebool -a
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
antivirus_can_scan_system --> off
antivirus_use_jit --> off
auditadm_exec_content --> on
. . .
. . .
zarafa_setrlimit --> off
zebra_write_config --> off
zoneminder_anon_write --> off
zoneminder_run_sudo --> off
[donnie@localhost ~]$
```

To view more than one Boolean, the `-a` switch is mandatory. If you just happen to know the name of the Boolean that you want to see, leave the `-a` out and list it. In keeping with the Apache web server theme that we've had going, let's see whether we're allowing Apache to access files in users' home directories:

```
[donnie@localhost html]$ getsebool httpd_enable_homedirs
httpd_enable_homedirs --> off
[donnie@localhost html]$
```

The fact that this Boolean is `off` means that the Apache server daemon isn't allowed to access any content within the users' home directories. This is an important protection, and you really don't want to change it. Instead, just put web content files elsewhere so that you don't have to change this Boolean.

You'll rarely want to look at the entire list, and you likely won't know the name of the specific Boolean that you want to see. Rather, you'll probably want to filter the output through `grep` in order to look at just certain things. For example, to see all of the Booleans that affect a web server, do this:

```
[donnie@localhost html]$ getsebool -a | grep 'http'
httpd_anon_write --> off
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off
httpd_can_connect_ldap --> off
. . .
. . .
httpd_use_nfs --> off
httpd_use_openstack --> off
httpd_use_sasl --> off
httpd_verify_dns --> off
named_tcp_bind_http_port --> off
prosody_bind_http_port --> off
[donnie@localhost html]$
```

It's also a rather long list, but scroll down a little and you'll find the Boolean that you seek.

### Configuring the Booleans

Realistically, you'll likely never have reason to allow users to serve web content out of their home directories. It's much more probable that you'll set up something like a Samba server, which would allow users on Windows machines to use their graphical Windows Explorer to access their home directories on Linux servers. But if you set up a Samba server and don't do anything with SELinux, users will complain about how they don't see any of their files in their home directories of the Samba server. Because you're the proactive type and you want to avoid the pain of listening to complaining users, you'll surely just go ahead and configure SELinux to allow the Samba daemon to access users' home directories. You might not know the exact name of the Boolean, but you can find it easily enough, as follows:

```
[donnie@localhost html]$ getsebool -a | grep 'home'
git_cgi_enable_homedirs --> off
git_system_enable_homedirs --> off
httpd_enable_homedirs --> off
mock_enable_homedirs --> off
mpd_enable_homedirs --> off
openvpn_enable_homedirs --> on
samba_create_home_dirs --> off
samba_enable_home_dirs --> off
. . .
use_samba_home_dirs --> off
xdm_write_home --> off
[donnie@localhost html]$
```

Okay, you knew that the Boolean name probably had the word `home` in it, so you filtered for that word. About half-way down the list, you see `samba_enable_home_dirs --> off`. You'll need to change this to `on` to let users access their home directories from their Windows machines:

```
[donnie@localhost html]$ sudo setsebool samba_enable_home_dirs on
[sudo] password for donnie:
[donnie@localhost html]$ getsebool samba_enable_home_dirs
samba_enable_home_dirs --> on
[donnie@localhost html]$
```

Users can now access their home directories as they should be able to, but only until you do a system reboot. Without the `-P` option, any changes you make with `setsebool` will only be temporary. So, let's make the change permanent with `-P`:

```
[donnie@localhost html]$ sudo setsebool -P samba_enable_home_dirs on
[donnie@localhost html]$ getsebool samba_enable_home_dirs
samba_enable_home_dirs --> on
[donnie@localhost html]$
```

Congratulations, you've just made your first change to an SELinux policy.

### Protecting your web server

Look at the output of the `getsebool -a | grep 'http'` command again, and you'll see that most httpd-related Booleans are turned off by default, with only a few turned on. There are two of them that you'll commonly need to turn on when setting up a web server.

If you ever need to set up a website with some sort of PHP-based content management system, such as Joomla or WordPress, you may have to turn on the `httpd_unified` Boolean. With this Boolean turned off, the Apache web server won't be able to interact properly with all of the components of the PHP engine:

```
[donnie@localhost ~]$ getsebool httpd_unified
httpd_unified --> off
[donnie@localhost ~]$ sudo setsebool -P httpd_unified on
[sudo] password for donnie:
[donnie@localhost ~]$ getsebool httpd_unified
httpd_unified --> on
[donnie@localhost ~]$
```

The other Boolean that you'll commonly need to turn on is the `httpd_can_sendmail` Boolean. If you ever need a website to send mail out through a form (or if you need to set up a mail server with a web-based frontend), you'll definitely need to set this to `on`:

```
[donnie@localhost ~]$ getsebool httpd_can_sendmail
httpd_can_sendmail --> off
[donnie@localhost ~]$ sudo setsebool -P httpd_can_sendmail on
[donnie@localhost ~]$ getsebool httpd_can_sendmail
httpd_can_sendmail --> on
[donnie@localhost ~]$
```

On the other hand, there are some Booleans that are turned on by default, and you might want to consider whether you really need them turned on. For example, allowing CGI scripts to run on a web server does represent a potential security risk. If an intruder were to somehow upload a malicious CGI script to the server and run it, much damage could occur as a result. Yet, for some bizarre reason, the default SELinux policy allows CGI scripts to run. If you're absolutely certain that nobody who hosts websites on your server will ever need to run CGI scripts, you might want to consider turning this Boolean off:

```
[donnie@localhost ~]$ getsebool httpd_enable_cgi
httpd_enable_cgi --> on
[donnie@localhost ~]$ sudo setsebool -P httpd_enable_cgi off
[donnie@localhost ~]$ getsebool httpd_enable_cgi
httpd_enable_cgi --> off
[donnie@localhost ~]$
```

### Protecting network ports

Each network daemon that's running on your system has a specific network port or set of network ports assigned to it, on which it will listen. The `/etc/services` file contains a list of common daemons and their associated network ports, but it doesn't prevent someone from configuring a daemon to listen on some non-standard port. So, without some mechanism to prevent it, some sneaky intruder could potentially plant some sort of malware that would cause a daemon to listen on a non-standard port, possibly listening for commands from its master.

SELinux protects against this sort of malicious activity by only allowing daemons to listen on certain ports. Use `semanage` to look at the list of allowed ports:

```
[donnie@localhost ~]$ sudo semanage port -l
SELinux Port Type              Proto    Port Number
afs3_callback_port_t           tcp      7001
afs3_callback_port_t           udp      7001
afs_bos_port_t                 udp      7007
. . .
. . .
zented_port_t                  udp      1229
zookeeper_client_port_t        tcp      2181
zookeeper_election_port_t      tcp      3888
zookeeper_leader_port_t        tcp      2888
zope_port_t                    tcp      8021
[donnie@localhost ~]$
```

This is yet another of those very long lists, so I'm only showing partial output. However, let's narrow things down a bit. Let's say that I only want to look at a list of ports on which the Apache web server can listen. For this, I'll use my good friend `grep`:

```
[donnie@localhost ~]$ sudo semanage port -l | grep 'http'
[sudo] password for donnie:
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[donnie@localhost ~]$
```

Several `http` items come up, but I'm only interested in the `http_port_t` item because it's the one that affects normal web server operation. We see here that SELinux will allow Apache to listen on ports `80`, `81`, `443`, `488`, `8008`, `8009`, `8443`, and `9000`. Since the Apache server is one of the few daemons for which you'd ever have a legitimate reason for adding a non-standard port, let's demo with it.

First, let's go into the `/etc/httpd/conf/httpd.conf` file and look at the ports on which Apache is currently listening. Search for `Listen`, and you'll see the following line:

```
Listen 80
```

I don't have the SSL module installed on this machine, but if I did I would have an `ssl.conf` file in the `/etc/httpd/conf.d/` directory with this line:

```
Listen 443
```

So for normal, non-encrypted website connections, the default configuration only has Apache listening on port `80`. For secure, encrypted website connections, Apache listens on port `443`. Now, let's go into the `httpd.conf` file and change `Listen 80` to a port number that SELinux doesn't allow, for example, port `82`:

```
Listen 82
```

After saving the file, I'll restart Apache to read in the new configuration:

```
[donnie@localhost ~]$ sudo systemctl restart httpd
Job for httpd.service failed because the control process exited with error code. See "systemctl status httpd.service" and "journalctl -xe" for details.
[donnie@localhost ~]$
```

Yes, I have a problem. I'll look in the `/var/log/messages` file to see if `setroubleshoot` gives me a clue:

```
Nov 29 16:39:21 localhost python: SELinux is preventing /usr/sbin/httpd from name_bind access on the tcp_socket port 82.#012#012***** Plugin bind_ports (99.5 confidence) suggests ************************#012#012If you want to allow /usr/sbin/httpd to bind to network port 82#012Then you need to modify the port type.#012Do#012# semanage port -a -t PORT_TYPE -p tcp 82#012 where PORT_TYPE is one of the following: http_cache_port_t, http_port_t, jboss_management_port_t, jboss_messaging_port_t, ntop_port_t, puppet_port_t.#012#012***** Plugin catchall (1.49 confidence) suggests **************************#012#012If you believe that httpd should be allowed name_bind access on the port 82 tcp_socket by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -i my-httpd.pp#012
```

The first line of the message details how SELinux is preventing `httpd` from binding to port `82`. The first suggestion we see for fixing this is to use `semanage` to add the port to the list of allowed ports. So, let's do that and then look at the list of Apache ports:

```
[donnie@localhost ~]$ sudo semanage port -a 82 -t http_port_t -p tcp
[donnie@localhost ~]$ sudo semanage port -l | grep 'http_port_t'
http_port_t                    tcp      82, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[donnie@localhost ~]$
```

It's not clear in the `setroubleshoot` message, but you need to specify the port number that you want to add after `port -a`. The `-t http_port_t` part specifies the **type** for which you want to add the port, and `-p tcp` specifies that you want to use the TCP protocol.

Now for the moment of truth. Will the Apache daemon start this time? Let's see:

```
[donnie@localhost ~]$ sudo systemctl restart httpd
[sudo] password for donnie:
[donnie@localhost ~]$ sudo systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-11-29 20:09:51 EST; 7s ago
     Docs: man:httpd(8)
. . .
. . .
```

It works, and we have achieved coolness. But now, I've decided that I no longer need this oddball port. Deleting it is just as easy as adding it:

```
[donnie@localhost ~]$ sudo semanage port -d 82 -t http_port_t -p tcp
[donnie@localhost ~]$ sudo semanage port -l | grep 'http_port_t'
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[donnie@localhost ~]$
```

All I had to do was to replace `port -a` with `port -d`. And of course, I still need to go into the `/etc/httpd/conf/httpd.conf` file to change `Listen 82` back to `Listen 80`.

### Creating custom policy modules

Sometimes, you'll run into a problem that you can't fix either by changing the type or by setting a Boolean. In times like these, you'll need to create a custom policy module, and you'll use the `audit2allow` utility to do that.

Here’s a screenshot of a problem I had several years ago, when I was helping a client set up a Postfix mail server on CentOS 7:

![](img/file66.png)

So, for some strange reason that I never understood, SELinux wouldn't allow Dovecot, the **Mail Delivery Agent** (**MDA**) component of the mail server, to read its own `dict` file. There's no Boolean to change and there wasn't a type problem, so `setroubleshoot` suggested that I create a custom policy module. It's easy enough to do, but you do need to be aware that this won't work with `sudo` on your normal user account. This is one of those rare times when you'll just have to go to the root user command prompt, and you'll also need to be in the root user's home directory:

```
sudo su -
```

Before you do it, be sure to put SELinux into permissive mode and then do something to induce the SELinux error. This way, you'll be sure that one problem isn't masking others.

When you run the command to create the new policy module, be sure to replace `mypol` with a custom policy name of your own choosing. In my case, I named the module `dovecot_dict`, and the command looked like this:

```
grep dict /var/log/audit/audit.log | audit2allow -M dovecot_dict
```

I’m using `grep` to search through the `audit.log` file for SELinux messages that contain the word `dict`. I then pipe the output of that into `audit2allow` and use the `-M` option to create a custom module with the name `dovecot_dict`.

After I created the new policy module, I inserted it into the SELinux policy like so:

```
semodule -i dovecot_dict.pp
```

There was a also a second problem that required another custom module, but I just repeated this procedure to produce another module of a different name. After I got all that done, I reloaded the SELinux policy, in order to get my new modules to take effect:

```
semodule -R
```

With `semodule`, the `-R` switch stands for reload, rather than recursive, as it does with most Linux commands.

With all that done, I put SELinux back into enforcing mode and exited back to my own user account. And I tested the setup to make sure that I had fixed the problem.

Of course, you also want to bear in mind that you don't want to just modify SELinux policy or contexts every time you see an `sealert` message in the log files. For example, consider this snippet from the `messages` file of my Oracle Linux 7 machine, which I set up mainly to run Docker and Docker containers:

```
Jun  8 19:32:17 docker-1 setroubleshoot: SELinux is preventing /usr/bin/docker from getattr access on the file /etc/exports. For complete SELinux messages. run sealert -l b267929a-d3ad-45d5-806e-907449fc2739
Jun  8 19:32:17 docker-1 python: SELinux is preventing /usr/bin/docker from getattr access on the file /etc/exports.#012#012*****  Plugin catchall (100\. confidence) suggests   **************************#012#012If you believe that docker should be allowed getattr access on the exports file by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# grep docker /var/log/audit/audit.log | audit2allow -M mypol#012# semodule -i mypol.pp#012
Jun  8 19:32:17 docker-1 setroubleshoot: SELinux is preventing /usr/bin/docker from getattr access on the file /etc/shadow.rpmnew. For complete SELinux messages. run sealert -l
. . .
```

These messages were caused by an early version of Docker trying to access resources on the host machine. As you can see, Docker is trying to access some rather sensitive files, and SELinux is preventing Docker from doing so. With Docker, and without some sort of MAC, it can be a trivial matter for a normal, unprivileged user to escape from the Docker container and have root user privileges on the host system. Naturally, when you see these sorts of messages, you don't want to automatically tell SELinux to allow the prohibited actions. It just may be that SELinux is preventing something truly bad from taking place.

> **Tip**
> 
> > Be sure to get your copy of The SELinux Coloring Book from [https://opensource.com/business/13/11/selinux-policy-guide](https://opensource.com/business/13/11/selinux-policy-guide).

#### Hands-on lab – SELinux Booleans and ports

In this lab, you'll view the effects of having Apache try to listen on an unauthorized port:

1.  View the ports that SELinux allows the Apache web server daemon to use:

```
sudo semanage port -l | grep 'http'
```

1.  Open the `/etc/httpd/conf/httpd.conf` file in your favorite text editor. Find the line that says `Listen 80` and change it to `Listen 82`. Restart Apache by entering the following:

```
sudo systemctl restart httpd
```

1.  View the error message you receive by entering:

```
sudo tail -20 /var/log/messages
```

1.  Add port `82` to the list of authorized ports and restart Apache:

```
sudo semanage port -a 82 -t http_port_t -p tcp
sudo semanage port -l
sudo systemctl restart httpd
```

1.  Delete the port that you just added:

```
sudo semanage -d 82 -t http_port_t -p tcp
```

1.  Go back into the `/etc/httpd/conf/httpd.conf` file and change `Listen 82` back to `Listen 80`. Restart the Apache daemon to return to normal operation.
2.  End of lab.

Okay, you've seen how SELinux can protect you against various bad things, and how to troubleshoot things that go wrong. Let's turn our attention to AppArmor.

## How AppArmor can benefit a systems administrator

AppArmor is the MAC system that comes installed with the SUSE and the Ubuntu families of Linux. Although it's designed to do pretty much the same job as SELinux, its mode of operation is substantially different:

*   SELinux labels all system processes and all objects such as files, directories, or network ports. For files and directories, SELinux stores the labels in their respective inodes as extended attributes. (An inode is the basic filesystem component that contains all information about a file, except for the filename.)
*   AppArmor uses pathname enforcement, which means that you specify the path to the executable file that you want AppArmor to control. This way, there's no need to insert labels into the extended attributes of files or directories.
*   With SELinux, you have system-wide protection out of the box.
*   With AppArmor, you have a profile for each individual application.
*   With either SELinux or AppArmor, you might occasionally find yourself having to create custom policy modules from scratch, especially if you're dealing with either third-party applications or home-grown software. With AppArmor, this is easier, because the syntax for writing AppArmor profiles is much easier than the syntax for writing SELinux policies. And AppArmor comes with utilities that can help you automate the process.
*   Just as SELinux can, AppArmor can help prevent malicious actors from ruining your day and can help protect user data.

So, you see that there are advantages and disadvantages to both SELinux and AppArmor, and a lot of Linux administrators have strong feelings about which one they prefer. (To avoid being subjected to a flame-war, I'll refrain from stating my own preference.) Also, note that even though we're working with an Ubuntu virtual machine, the information I present here, other than the Ubuntu-specific package installation commands, also works with the SUSE Linux distros.

### Looking at AppArmor profiles

To begin, we’ll install the lxc package so that we can have more to look at:

```
sudo apt install lxc
```

In the `/etc/apparmor.d/` directory, you'll see the AppArmor profiles for your system. (SELinux folk say *policies*, but AppArmor folk say *profiles*.):

```
donnie@ubuntu3:/etc/apparmor.d$ ls -l
total 72
drwxr-xr-x 5 root root  4096 Oct 29 15:21 abstractions
drwxr-xr-x 2 root root  4096 Nov 15 09:34 cache
drwxr-xr-x 2 root root  4096 Oct 29 14:43 disable
. . .
. . .
-rw-r--r-- 1 root root   125 Jun 14 16:15 usr.bin.lxc-start
-rw-r--r-- 1 root root   281 May 23  2017 usr.lib.lxd.lxd-bridge-proxy
-rw-r--r-- 1 root root 17667 Oct 18 05:04 usr.lib.snapd.snap-confine.real
-rw-r--r-- 1 root root  1527 Jan  5  2016 usr.sbin.rsyslogd
-rw-r--r-- 1 root root  1469 Sep  8 15:27 usr.sbin.tcpdump
donnie@ubuntu3:/etc/apparmor.d$
```

All of the text files you see in this directory are AppArmor profiles. If you’ve installed the `lxc` package, you'll find a few other profiles in the `lxc` and `lxc-containers` subdirectories. Still, though, there's not a whole lot there in the way of application profiles.

> **Tip**
> 
> > For some reason, a default installation of OpenSUSE comes with more installed profiles than Ubuntu Server does. To install more profiles on Ubuntu, just run this command:
> 
> > **sudo apt install apparmor-profiles apparmor-profiles-extra**

In the `abstractions` subdirectory, you'll find files that aren't complete profiles but that can be included in complete profiles. Any one of these abstraction files can be included in any number of profiles. This way, you don't have to write the same code over and over every time you create a profile. Just include an abstraction file instead.

> **Tip**
> 
> > If you're familiar with programming concepts, just think of abstraction files as `include` files by another name.

Here’s a partial listing of the abstraction files:

```
donnie@ubuntu3:/etc/apparmor.d/abstractions$ ls -l
total 320
-rw-r--r-- 1 root root  695 Mar 15  2017 apache2-common
drwxr-xr-x 2 root root 4096 Oct 29 15:21 apparmor_api
-rw-r--r-- 1 root root  308 Mar 15  2017 aspell
-rw-r--r-- 1 root root 1582 Mar 15  2017 audio
. . .
. . .
-rw-r--r-- 1 root root  705 Mar 15  2017 web-data
-rw-r--r-- 1 root root  739 Mar 15  2017 winbind
-rw-r--r-- 1 root root  585 Mar 15  2017 wutmp
-rw-r--r-- 1 root root 1819 Mar 15  2017 X
-rw-r--r-- 1 root root  883 Mar 15  2017 xad
-rw-r--r-- 1 root root  673 Mar 15  2017 xdg-desktop
donnie@ubuntu3:/etc/apparmor.d/abstractions$
```

To get a feel for how AppArmor rules work, let's peek inside the `web-data` abstraction file:

```
 /srv/www/htdocs/ r,
  /srv/www/htdocs/** r,
  # virtual hosting
  /srv/www/vhosts/ r,
  /srv/www/vhosts/** r,
  # mod_userdir
  @{HOME}/public_html/ r,
  @{HOME}/public_html/** r,
  /srv/www/rails/*/public/ r,
  /srv/www/rails/*/public/** r,
  /var/www/html/ r,
  /var/www/html/** r,
```

This file is just a list of directories from which the Apache daemon is allowed to read files. Let's break it down:

*   Note that each rule ends with `r,` . This denotes that we want Apache to have read access to each listed directory. Also note that each rule has to end with a comma.
*   `/srv/www/htdocs/ r,` : This means that the listed directory itself has read access for Apache.
*   `/srv/www.htdocs/* * r,` : The `* *` wildcards make this rule recursive. In other words, Apache can read all files in all subdirectories of this specified directory.
*   `# mod_userdir` : If installed, this Apache module allows Apache to read web content files from a subdirectory that's within a user's home directory. The next two lines go along with that.
*   `@{HOME}/public_html/ r,` and `@{HOME}/public_html/ r,` : The `@{HOME}` variable allows this rule to work with any user's home directory. (You'll see this variable defined in the `/etc/apparmor.d/tunables/home` file.)
*   Note that there's no specific rule that prohibits Apache from reading from other locations. It's just understood that anything that's not listed here is off-limits to the Apache web server daemon.

The `tunables` subdirectory contains files that have predefined variables. You can also use this directory to either define new variables or make profile tweaks:

```
donnie@ubuntu3:/etc/apparmor.d/tunables$ ls -l
total 56
-rw-r--r-- 1 root root  624 Mar 15  2017 alias
-rw-r--r-- 1 root root  376 Mar 15  2017 apparmorfs
-rw-r--r-- 1 root root  804 Mar 15  2017 dovecot
-rw-r--r-- 1 root root  694 Mar 15  2017 global
-rw-r--r-- 1 root root  983 Mar 15  2017 home
. . .
. . .
-rw-r--r-- 1 root root  440 Mar 15  2017 proc
-rw-r--r-- 1 root root  430 Mar 15  2017 securityfs
-rw-r--r-- 1 root root  368 Mar 15  2017 sys
-rw-r--r-- 1 root root  868 Mar 15  2017 xdg-user-dirs
drwxr-xr-x 2 root root 4096 Oct 29 15:02 xdg-user-dirs.d
donnie@ubuntu3:/etc/apparmor.d/tunables$
```

Space doesn't permit me to show you the details of how to write your own profiles from scratch. Thanks to the suite of utilities that we'll look at in the next section, you might never need to do that. Still, just to give you a better understanding about how AppArmor does what it does, the following is a chart of some example rules that you might find in any given profile:

| **Rule** | **Explanation** |
| `/var/run/some_program.pid rw,` | The process will have read and write privileges for this process ID file. |
| `/etc/ld.so.cache r,` | The process will have read privileges for this file. |
| `/tmp/some_program.* l,` | The process will be able to create and delete links with the `some_program` name. |
| `/bin/mount ux` | The process has executable privileges for the `mount` utility, which will then run unconstrained. (Unconstrained means without an AppArmor profile.) |

Now that you know about AppArmor profiles, let's look at some basic AppArmor utilities.

### Working with AppArmor command-line utilities

Whether or not you have all the AppArmor utilities you need will depend on which Linux distribution you have. On my OpenSUSE Leap workstation, the utilities were there out of the box. On my Ubuntu Server virtual machine, I had to install them myself:

```
sudo apt install apparmor-utils
```

First, let's look at the status of AppArmor on the Ubuntu machine. Since it's a rather long output, we'll look at it in sections. Here's the first section:

```
donnie@ubuntu2204-packt:~$ sudo aa-status
apparmor module is loaded.
61 profiles are loaded.
43 profiles are in enforce mode.
   /snap/snapd/17029/usr/lib/snapd/snap-confine
   /snap/snapd/17029/usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /snap/snapd/17336/usr/lib/snapd/snap-confine
   /snap/snapd/17336/usr/lib/snapd/snap-confine//mount-namespace-capture-helper
   /usr/bin/lxc-start
   /usr/bin/man
   /usr/bin/pidgin
   /usr/bin/pidgin//sanitized_helper
. . .
. . .
```

The first thing to note here is that AppArmor has an **enforce** mode and a **complain** mode. The enforce mode that's shown here does the same job as its enforcing mode counterpart in SELinux. It prevents system processes from doing things that the active policy doesn't allow, and it logs any violations.

Now, here's the second section:

```
. . .
. . .
0 processes are in enforce mode.
1 processes are in complain mode.
   /usr/sbin/dnsmasq (2485) dnsmasq
0 processes are unconfined but have a profile defined.
0 processes are in mixed mode.
0 processes are in kill mode.
donnie@ubuntu2204-packt:~$
```

Complain mode is the same as permissive mode in SELinux. It allows processes to perform actions that are prohibited by the active policy, but it records those actions in either the `/var/log/audit/audit.log` file, or the system log file, depending on whether you have `auditd` installed. (Unlike Red Hat-type distributions, `auditd` doesn't come installed by default on Ubuntu.) You would use complain mode either to help with troubleshooting or to test new profiles.

Most enforce mode profiles we see here have to do with either network management or with `lxc` container management. Two exceptions we see are the two profiles for `snapd`, which is the daemon that makes the snap packaging technology work. The third exception is the `mysqld` profile.

> Snap packages are universal binary files that are designed to work on multiple distributions. Snap technology is currently available for most major Linux distros.

Curiously, when you install a daemon package on Ubuntu, you'll sometimes get a predefined profile for that daemon and sometimes you won't. Even when a profile does come with the package that you've installed, it's sometimes already in enforce mode and sometimes isn't. For example, if you're setting up a **Domain Name Service** (**DNS**) server and you install the `bind9` package for it, you'll get an AppArmor profile that's already in enforce mode. If you're setting up a database server and install the `mysql-server` package, you'll also get a working profile that's already in enforce mode.

But, if you're setting up a database server and you prefer to install the `mariadb-server` instead of `mysql-server`, you'll get an AppArmor profile that's completely disabled and that can't be enabled. When you look in the `usr.sbin.mysqld` profile file that gets installed with the `mariadb-server` package, you'll see this:

```
# This file is intentionally empty to disable apparmor by default for newer
# versions of MariaDB, while providing seamless upgrade from older versions
# and from mysql, where apparmor is used.
#
# By default, we do not want to have any apparmor profile for the MariaDB
# server. It does not provide much useful functionality/security, and causes
# several problems for users who often are not even aware that apparmor
# exists and runs on their system.
#
# Users can modify and maintain their own profile, and in this case it will
# be used.
#
# When upgrading from previous version, users who modified the profile
# will be promptet to keep or discard it, while for default installs
# we will automatically disable the profile.
```

Okay, so apparently AppArmor isn't good for everything. (And whoever wrote this needs to take spelling lessons.)

And then there's Samba, which is a special case in more ways than one. When you install the `samba` package to set up a Samba server, you don't get any AppArmor profiles at all. For Samba and several other different applications as well, you'll need to install AppArmor profiles separately:

```
sudo apt install apparmor-profiles apparmor-profiles-extra
```

When you install these two profile packages, the profiles will all be in **complain** mode. That's okay, because we have a handy utility to put them into **enforce** mode. Since Samba has two different daemons that we need to protect, there are two different profiles that we'll need to place into **enforce** mode:

```
donnie@ubuntu5:/etc/apparmor.d$ ls *mbd
usr.sbin.nmbd  usr.sbin.smbd
donnie@ubuntu5:/etc/apparmor.d$
```

We'll use `aa-enforce` to activate **enforce** mode for both of these profiles:

```
donnie@ubuntu5:/etc/apparmor.d$ sudo aa-enforce /usr/sbin/nmbd usr.sbin.nmbd
Setting /usr/sbin/nmbd to enforce mode.
Setting /etc/apparmor.d/usr.sbin.nmbd to enforce mode.
donnie@ubuntu5:/etc/apparmor.d$ sudo aa-enforce /usr/sbin/smbd usr.sbin.smbd
Setting /usr/sbin/smbd to enforce mode.
Setting /etc/apparmor.d/usr.sbin.smbd to enforce mode.
donnie@ubuntu5:/etc/apparmor.d$
```

To use `aa-enforce`, you first need to specify the path to the executable file of the process that you want to protect. (Fortunately, you normally won't even have to look that up, since the path name is normally part of the profile filename.) The last part of the command is the name of the profile. Note that you'll need to restart the Samba daemon to get this AppArmor protection to take effect.

Placing a profile into other modes is just as easy. All you have to do is to replace the `aa-enforce` utility with the utility for the mode that you need to use. The following is a chart of utilities for the other modes:

| **Command** | **Explanation** |
| `aa-audit` | Audit mode is the same as enforce mode, except that allowed actions get logged, as well as actions that have been blocked. (Enforce mode only logs actions that have been blocked.) |
| `aa-disable` | This completely disables a profile. |
| `aa-complain` | This places a profile into complain mode. |

Okay, we're moving right along. You now know about basic AppArmor commands. Next up, we'll look at troubleshooting AppArmor problems.

### Troubleshooting AppArmor problems

When I wrote the first edition of this book back in 2017, I sat here racking my brains for several days, trying to come up with a good troubleshooting scenario. It turns out that I didn't need to. The Ubuntu folk handed me a good scenario on a silver platter in the form of a buggy Samba profile. At the time, I was working with Ubuntu 16.04, which had the original version of the bug. The original bug got fixed for Ubuntu 18.04, but was replaced by two others. In Ubuntu 22.04, the profile is finally bug-free. I still want to show you how to troubleshoot AppArmor problems though, so I’ve left the 16.04 and 18.04 write-ups intact. (It’s still possible to download and install Ubuntu 16.04 and 18.04, so you can create some virtual machines to follow along if you’d like. I’ll leave that decision up to you.)

### Troubleshooting an AppArmor profile – Ubuntu 16.04

As you've just seen, I used `aa-enforce` to put the two Samba-related profiles into enforce mode. But watch what happens now when I try to restart Samba in order to get the profiles to take effect:

```
donnie@ubuntu3:/etc/apparmor.d$ sudo systemctl restart smbd
Job for smbd.service failed because the control process exited with error code. See "systemctl status smbd.service" and "journalctl -xe" for details.
donnie@ubuntu3:/etc/apparmor.d$
```

Okay, that's not good. Looking at the status for the `smbd` service, I see the following:

```
donnie@ubuntu3:/etc/apparmor.d$ sudo systemctl status smbd
● smbd.service - LSB: start Samba SMB/CIFS daemon (smbd)
   Loaded: loaded (/etc/init.d/smbd; bad; vendor preset: enabled)
   Active: failed (Result: exit-code) since Tue 2017-12-05 14:56:35 EST; 13s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 31160 ExecStop=/etc/init.d/smbd stop (code=exited, status=0/SUCCESS)
  Process: 31171 ExecStart=/etc/init.d/smbd start (code=exited, status=1/FAILURE)
Dec 05 14:56:35 ubuntu3 systemd[1]: Starting LSB: start Samba SMB/CIFS daemon (smbd)...
Dec 05 14:56:35 ubuntu3 smbd[31171]:  * Starting SMB/CIFS daemon smbd
Dec 05 14:56:35 ubuntu3 smbd[31171]:    ...fail!
. . .
```

The important things to note here are all the places where some form of the word `fail` shows up.

The original error message said to use `journalctl -xe` to view the log message. You can do that if you like, or you can just use either `less` or `tail` to look in the `/var/log/syslog` log file:

```
Dec  5 20:09:10 ubuntu3 smbd[14599]:  * Starting SMB/CIFS daemon smbd
Dec  5 20:09:10 ubuntu3 kernel: [174226.392671] audit: type=1400 audit(1512522550.765:510): apparmor="DENIED" operation="mknod" profile="/usr/sbin/smbd" name="/run/samba/msg.
lock/14612" pid=14612 comm="smbd" requested_mask="c" denied_mask="c" fsuid=0 ouid=0
Dec  5 20:09:10 ubuntu3 smbd[14599]:    ...fail!
Dec  5 20:09:10 ubuntu3 systemd[1]: smbd.service: Control process exited, code=exited status=1
Dec  5 20:09:10 ubuntu3 systemd[1]: Failed to start LSB: start Samba SMB/CIFS daemon (smbd).
Dec  5 20:09:10 ubuntu3 systemd[1]: smbd.service: Unit entered failed state.
Dec  5 20:09:10 ubuntu3 systemd[1]: smbd.service: Failed with result 'exit-code'.
```

So, we see `apparmor=DENIED`. Obviously, Samba is trying to do something that the profile doesn't allow. Samba needs to write temporary files to the `/run/samba/msg.lock` directory, but it isn't allowed to. I'm guessing that the profile lacks a rule that allows that to happen.

But even if this log file entry gave me no clue at all, I could just cheat, using a troubleshooting technique that has served me well for many years. That is, I could just copy and paste the error message from the log file into my favorite search engine. Pretty much every time I've ever done that, I've found that other people before me have already had the same problem:

![](img/file67.png)

Okay, I didn't paste in the entire error message, but I did paste in enough for DuckDuckGo to work with. And lo and behold, it worked:

![](img/file68.png)

Hmmm, it looks like my profile file might be missing an important line. So, I'll open the `usr.sbin.smbd` file and place this line at the end of the rule set:

```
/run/samba/** rw,
```

This line will allow read and write access to everything in the `/run/samba/` directory. After making the edit, I'll need to reload this profile because it's already been loaded with `aa-enforce`. For this, I'll use the `apparmor_parser` utility:

```
donnie@ubuntu3:/etc/apparmor.d$ sudo apparmor_parser -r usr.sbin.smbd
donnie@ubuntu3:/etc/apparmor.d$
```

All you need to do is use the `-r` option for reloading and list the name of the profile file. Now, let's try to restart Samba:

```
donnie@ubuntu3:/etc/apparmor.d$ sudo systemctl restart smbd
donnie@ubuntu3:/etc/apparmor.d$ sudo systemctl status smbd
● smbd.service - LSB: start Samba SMB/CIFS daemon (smbd)
   Loaded: loaded (/etc/init.d/smbd; bad; vendor preset: enabled)
   Active: active (running) since Wed 2017-12-06 13:31:32 EST; 3min 6s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 17317 ExecStop=/etc/init.d/smbd stop (code=exited, status=0/SUCCESS)
  Process: 16474 ExecReload=/etc/init.d/smbd reload (code=exited, status=0/SUCCESS)
  Process: 17326 ExecStart=/etc/init.d/smbd start (code=exited, status=0/SUCCESS)
    Tasks: 3
   Memory: 9.3M
      CPU: 594ms
   CGroup: /system.slice/smbd.service
           ├─17342 /usr/sbin/smbd -D
           ├─17343 /usr/sbin/smbd -D
           └─17345 /usr/sbin/smbd -D
Dec 06 13:31:28 ubuntu3 systemd[1]: Stopped LSB: start Samba SMB/CIFS daemon (smbd).
Dec 06 13:31:28 ubuntu3 systemd[1]: Starting LSB: start Samba SMB/CIFS daemon (smbd)...
Dec 06 13:31:32 ubuntu3 smbd[17326]:  * Starting SMB/CIFS daemon smbd
Dec 06 13:31:32 ubuntu3 smbd[17326]:    ...done.
Dec 06 13:31:32 ubuntu3 systemd[1]: Started LSB: start Samba SMB/CIFS daemon (smbd).
donnie@ubuntu3:/etc/apparmor.d$
```

And it works! The two Samba profiles are in enforce mode, and Samba finally starts up just fine.

The odd part about this is that I had this same problem with both Ubuntu 16.04 and Ubuntu 17.10\. So, the bug has been there for a long time.

### Troubleshooting an AppArmor profile – Ubuntu 18.04

As I said before, the bug in Ubuntu 16.04 got replaced by two others in Ubuntu 18.04\. So, let’s look at that.

I installed Samba and the additional AppArmor profiles on my Ubuntu 18.04 VM, and then set the two Samba profiles into **enforce** mode, the same way that I've already shown you for Ubuntu 16.04\. When I tried to restart Samba, the restart failed. So, I looked in the `/var/log/syslog` file and found these two messages:

```
Oct 15 19:22:05 ubuntu-ufw kernel: [ 2297.955842] audit: type=1400 audit(1571181725.419:74): apparmor="DENIED" operation="capable" profile="/usr/sbin/smbd" pid=15561 comm="smbd" capability=12  capname="net_admin"
Oct 15 19:22:05 ubuntu-ufw kernel: [ 2297.960193] audit: type=1400 audit(1571181725.427:75): apparmor="DENIED" operation="sendmsg" profile="/usr/sbin/smbd" name="/run/systemd/notify" pid=15561 comm="smbd" requested_mask="w" denied_mask="w" fsuid=0 ouid=0
```

Now that we know how to read the AppArmor error messages, this is easy to figure out. It looks like we need to allow the SMBD service to have `net_admin` capabilities so that it can properly access the network. And, it looks like we also need to add a rule to allow SMBD to write to the `/run/systemd/notify` socket file. So, let's edit the `/etc/apparmor.d/usr.sbin.smbd` file and add the two missing lines.

First, in the stanza with all of the `capability` lines, I'll add this line to the bottom:

```
capability net_admin,
```

Then, at the bottom of the rules list, just under the `/var/spool/samba/** rw,` line, I'll add this line:

```
/run/systemd/notify rw,
```

It's now just a matter of reloading the policy and restarting the SMBD service, the same as we did for Ubuntu 16.04.

#### Hands-on lab – Troubleshooting an AppArmor profile

Perform this lab on an Ubuntu 18.04 VM. Carry out the following steps for troubleshooting:

1.  Install the AppArmor utilities and the extra profiles:

```
sudo apt install apparmor-utils apparmor-profiles apparmor-profiles-extra
```

1.  Install Samba and verify that it's running:

```
sudo apt install samba
sudo systemctl status smbd
sudo systemctl status nmbd
```

1.  Set the two aforementioned Samba policies to enforce mode and try to restart Samba:

```
cd /etc/apparmor.d
sudo aa-enforce /usr/sbin/smbd usr.sbin.smbd
sudo aa-enforce /usr/sbin/nmbd usr.sbin.nmbd
sudo systemctl restart smbd
```

1.  Note that Samba should fail to restart. (It will take quite a while before it finally errors out, so be patient.)
2.  Look in the `/var/log/syslog` file to see if you can spot the problem.
3.  Edit the `/etc/apparmor.d/usr.sbin.smbd` file. In the `capability` stanza, add this line:

```
capability net_admin,
```

1.  At the bottom of the rules sections, under the `/var/spool/samba/** rw,` line, add this line:

```
/run/systemd/notify rw,
```

1.  Save the file and reload the policy:

```
sudo apparmor_parser -r usr.sbin.smbd
```

1.  As before, try to restart the Samba service, and verify that it started properly:

```
sudo systemctl restart smbd
sudo systemctl status smbd
```

1.  End of lab.

Okay, you've just explored the basics of troubleshooting buggy AppArmor profiles. This is good knowledge to have, especially when your organization needs to deploy its own custom profiles that could end up being just as buggy.

### Troubleshooting Samba problems in Ubuntu 22.04

I told you before that the buggy AppArmor profile for Samba has been fixed in Ubuntu 22.04\. So, hallelujah for that, right? Well, not so fast. As I’m writing this in November, 2022, there’s now a bug in certain versions of Samba that prevents Samba from starting if the AppArmor profile is enabled.

> This bug was also in SUSE 15.3, but has been fixed in SUSE 15.4\. It has also been fixed in Ubuntu 22.10\. By the time you read this, it might also have gotten fixed in Ubuntu 22.04.

First, install the `samba`, `apparmor-profiles`, `apparmor-profiles-extra`, and `apparmor-util` packages, as I described in the preceding sections. Doing a `systemctl status smbd` command should show that the Samba service is running normally. Next, we’ll set the two Samba profiles to **enforce** mode:

```
cd /etc/apparmor.d
sudo aa-enforce /usr/sbin/smbd usr.sbin.smbd
sudo aa-enforce /usr/sbin/nmbd usr.sbin.nmbd
sudo systemctl restart smbd
```

This time, you won’t see any error message until you do `systemctl status smbd`:

```
donnie@ubuntu2204-packt:~$ systemctl status smbd
× smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Mon 2022-11-14 21:01:28 UTC; 7s ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 1966 ExecStartPre=/usr/share/samba/update-apparmor-samba-profile (code=exited, status=0/SUCCESS)
    Process: 1976 ExecStart=/usr/sbin/smbd --foreground --no-process-group $SMBDOPTIONS (code=exited, status=1/FAILURE)
   Main PID: 1976 (code=exited, status=1/FAILURE)
     Status: "daemon failed to start: Samba failed to init printing subsystem"
      Error: 13 (Permission denied)
        CPU: 133ms
```

Unlike before, searching through the system log file won’t tell you anything. If you search through the `usr.sbin.smbd` file, you’ll see no problems at all. Instead, the problem this time is with the print service function of the Samba service. Fortunately, it’s an easy fix, as long as you don’t mind doing without Samba print sharing. Just open the `/etc/samba/smb.conf` file in your text editor, and add the following line to the `[global]` section:

```
disable spoolss = yes
```

The Samba service should now start up without issues.

Now, I have to confess that I don’t know what the exact impact of this directive will be. The `smb.conf` man page says that printing might be impacted for Windows NT/2000 systems, but it doesn’t say anything about how it affects newer versions of Windows. At any rate, I’ll leave this experimentation up to you if you really need Samba print server support.

All right, enough of this Samba business. Let’s move on to something that’s evil, but also fun.

## Exploiting a system with an evil Docker container

You might think that containers are somewhat like virtual machines, and you'd be partly correct. The difference is that a virtual machine runs an entire self-contained operating system, and a container doesn't. Instead, a container comes with the guest's operating system's package management and libraries, but it uses the kernel resources of the host operating system. That makes containers much more lightweight. So, you can pack more containers on a server than you can virtual machines, which helps cut down on hardware and energy costs. Containers have been around for quite a few years, but they didn't become all that popular until Docker came on the scene.

But, the very thing that makes containers so lightweight—the fact that they use the host machine's kernel resources—can also make for some interesting security problems. Using some form of MAC is one thing you can do to help mitigate these problems.

One problem is that, to run Docker, a person needs either to have the proper `sudo` privileges, or be a member of the `docker` group. Either way, anyone who logs into a container will be at the root command prompt for that container. By creating a container that mounts the root filesystem of the host machine, a non-privileged user can take complete control of the host system.

### Hands-on lab – Creating an evil Docker container

To demonstrate, I'll use a CentOS 7 VM in order to show how SELinux can help protect you. (I'm using CentOS 7 because the RHEL 8/9-type distros use `podman` instead of `docker`, which won’t allow this exploit to happen.) Also, you'll need to do this from the local console of the VM because the root user will be locked out from logging in via SSH (you'll see what I mean in just a bit):

1.  On your CentOS 7 VM, install Docker and enable the daemon:

```
sudo yum install docker
sudo systemctl enable --now docker
```

1.  Create the `docker` group.

```
sudo groupadd docker
```

1.  Create a user account for Katelyn, my teenage calico kitty, adding her to the `docker` group at the same time:

```
sudo useradd -G docker katelyn
sudo passwd katelyn
```

1.  Log out of your own user account and log back in as Katelyn.
2.  Have Katelyn create a Debian container that mounts the `/` partition of the host machine in the `/homeroot` mountpoint, and that opens a Bash shell session for the root user:

```
docker run -v /:/homeroot -it debian bash
```

> Note how Katelyn has done this without having to use any `sudo` privileges. Also note that there are no blank spaces in the `/:/homeroot` part.

1.  The goal is for Katelyn to make herself the root user on the host machine. In order to do that, she'll need to edit the `/etc/passwd` file to change her own user ID to `0`. To do that, she'll need to install a text editor. (Katelyn prefers vim, but you can use nano if you really want to.) While still within the Debian container, run the following:

```
apt update
apt install vim
```

1.  Have Katelyn `cd` into the host machine's `/etc/` directory and attempt to open the `passwd` file in the text editor:

```
cd /homeroot/etc
vim passwd
```

1.  She won't be able to do it, because SELinux prevents it.
2.  Type `exit` to exit the container.
3.  Log out from Katelyn's account and then log back into your own account.
4.  Place SELinux into permissive mode:

```
sudo setenforce 0
```

1.  Log out from your own account and log back in as Katelyn.
2.  Repeat *steps* *5* through *7*. This time, Katelyn will be able to open the `/etc/passwd` file in her text editor.
3.  In the `passwd` file, have Katelyn find the line for her own user account. Have her change her user ID number to `0`. The line should now look something like this:

```
katelyn:x:0:1002::/home/katelyn:/bin/bash
```

1.  Save the file, and type `exit` to have Katelyn exit the container. Have Katelyn log out of the virtual machine, and then log back in. This time, you'll see that she has successfully logged into the root user shell.
2.  End of lab

Okay, you've just seen one of Docker's security weaknesses and how SELinux can protect you from it. Since Katelyn doesn't have `sudo` privileges, she can't put SELinux into permissive mode, which prevents her from doing any Docker mischief. On the RHEL 8/9-type distros, things are even better. Even with SELinux in either permissive mode or completely disabled, you still won’t be able to edit the `passwd` file from within a container on either of your AlmaLinux machines. That’s because RHEL 8/9 and their offspring use `podman`, which is Red Hat’s drop-in replacement for `docker`. `podman` has several advantages over docker, mainly in the area of host machine security.

So now you're wondering if AppArmor on Ubuntu helps us with this. Well, not by default, because there's no pre-built profile for the Docker daemon. When you run Docker on an Ubuntu machine, it automatically creates a Docker profile for the container in the `/tmpfs/` directory, but it really doesn't do much. I tested this procedure on an Ubuntu 18.04 VM with AppArmor enabled, and Katelyn was able to do her evil deed just fine. (Note that `podman` is now also available for non-Red Hat Linux distros, including Ubuntu.)

Earlier in this chapter, I said that I would refrain from stating which of these two MAC systems I prefer. But, I lied. If you haven't figured it out by now, I'm definitely a big fan of SELinux, because it provides better out-of-the-box protection than AppArmor does. If you choose to use Ubuntu, then plan on writing a new AppArmor policy any time that your development team deploys a new application.

> **Tip**
> 
> > If you’d rather not deal with the complexities of creating AppArmor profiles, you can instead place security directives in the systemd unit files for your services. You might find that this is a bit easier, and it can give you much of the same protection that AppArmor would. To read all about it, check out my other book, *Linux Service Management Made Easy with systemd*.

I do believe that this wraps things up for our discussion of MAC.

## Summary

In this chapter, we looked at the basic principles of MAC and compared two different MAC systems. We saw what SELinux and AppArmor are and how they can help safeguard your systems against malicious actors. We then looked at the basics of how to use them and the basics of how to troubleshoot them. We also saw that, even though they're both meant to do the same job, they work in vastly different ways. We wrapped things up by showing you a practical example of how SELinux can protect you from evil Docker containers.

Whether you're working with AppArmor or with SELinux, you'll always want to thoroughly test a new system in either **complain** or **permissive** mode before you put it into production. Make sure that what you want to protect gets protected, while at the same time what you want to allow gets allowed. After you place the machine into production, don't just assume that you can automatically change a policy setting every time you see a policy violation occur. It could be that nothing is wrong with your MAC setup and that MAC is just doing its job, protecting you from the bad guys.

There's a lot more to both of these topics than we can cover here. Hopefully, though, I've given you enough to whet your appetite and help you out in your day-to-day duties.

In the next chapter, we'll look at more techniques for hardening the kernel and isolating processes. I'll see you there.

## Questions

1.  Which of the following would represent a MAC principle?

A. You can set permissions on your own files and directories however you need to.

B. You can allow any system process to access whatever you need it to access.

C. System processes can only access whichever resources MAC policies allow them to access.

D. MAC will allow access, even if DAC doesn't.

1.  How does SELinux work?

A. It places a label on each system object and allows or denies access according to what SELinux policies say about the labels.

B. It simply consults a profile for each system process to see what the process is allowed to do.

C. It uses extended attributes that an administrator would set with the `chattr` utility.

D. It allows each user to set his or her own MACs.

1.  Which of these utilities would you use to fix an incorrect SELinux security context?

A. `chattr`

B. `chcontext`

C. `restorecon`

D. `setsebool`

1.  For normal day-to-day administration of a Red Hat-type server, which of the following aspects of a security context would an administrator be most concerned about?

A. user

B. role

C. type

D. sensitivity

1.  You’ve set up a new directory that a particular daemon wouldn’t normally access, and you want to permanently allow that daemon to access that directory. Which of the following utilities would you use to do that?

A. `chcon`

B. `restorecon`

C. `setsebool`

D. `semanage`

1.  Which of the following constitutes one difference between SELinux and AppArmor?

A. With SELinux, you have to install or create a policy profile for each system process that you need to control.

B. With AppArmor, you have to install or create a policy profile for each system process that you need to control.

C. AppArmor works by applying a label to each system object, while SELinux works by simply consulting a profile for each system object.

D. It's much easier to write a policy profile for SELinux, because the language is easier to understand.

1.  Which `/etc/apparmor.d/` subdirectory contains files with pre-defined variables?

A. `tunables`

B. `variables`

C. `var`

D. `functions`

1.  Which of the following utilities would you use to enable an AppArmor policy?

A. `aa-enforce`

B. `aa-enable`

C. `set-enforce`

D. `set-enable`

1.  You've already enabled an AppArmor policy for a daemon, but you now need to change the policy. Which utility would you use to reload the modified policy?

A. `aa-reload`

B. `apparmor_reload`

C. `aa-restart`

D. `apparmor_parser`

1.  You're testing a new AppArmor profile and you want to find any possible problems before you place the server into production. Which AppArmor mode would you use to test that profile?

A. permissive

B. enforce

C. testing

D. complain

## Further reading

SELinux:

*   Accessing SELinux policy documentation: [https://www.redhat.com/sysadmin/accessing-selinux-policy-documentation](https://www.redhat.com/sysadmin/accessing-selinux-policy-documentation)
*   Using SELinux: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/index](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/index)
*   SELinux System Administration-Third Edition: [https://www.packtpub.com/product/selinux-system-administration-third-edition/9781800201477](https://www.packtpub.com/product/selinux-system-administration-third-edition/9781800201477)
*   SELinux Coloring book: [https://opensource.com/business/13/11/selinux-policy-guide](https://opensource.com/business/13/11/selinux-policy-guide)
*   Linux Service Management Made Easy with systemd: [https://www.packtpub.com/product/linux-service-management-made-easy-with-systemd/9781801811644?_ga=2.122984843.1038813545.1668463819-58585121.1668463819](https://www.packtpub.com/product/linux-service-management-made-easy-with-systemd/9781801811644?_ga=2.122984843.1038813545.1668463819-58585121.1668463819)

AppArmor:

*   Ubuntu AppArmor wiki: [https://wiki.ubuntu.com/AppArmor](https://wiki.ubuntu.com/AppArmor)
*   How to create an AppArmor profile: [https://tutorials.ubuntu.com/tutorial/beginning-apparmor-profile-development#0](https://tutorials.ubuntu.com/tutorial/beginning-apparmor-profile-development#0)
*   The comprehensive guide to AppArmor: [https://medium.com/information-and-technology/so-what-is-apparmor-64d7ae211ed](https://medium.com/information-and-technology/so-what-is-apparmor-64d7ae211ed)
*   Samba print server bug: [https://www.reddit.com/r/openSUSE/comments/q9cpcc/samba_share_doesnt_work_since_snapshot_20211012/](https://www.reddit.com/r/openSUSE/comments/q9cpcc/samba_share_doesnt_work_since_snapshot_20211012/)

## Answers

1.  C
2.  A
3.  C
4.  C
5.  D
6.  B
7.  A
8.  A
9.  D
10.  D