# Chapter 5. Herding Cats – Taking Control of Processes

All too often, Linux administrators without the insight that you have, will leave services running as shipped after a Vanilla install of CentOS. We know how important it is to be able to justify each running process and service on systems that we manage, and in this chapter, you will gain the insight to manage this. While we are here, we will take a look at the new Upstart services that are replacing the System V scripts, which we have become so accustomed to. Here is a list of sections we will go through in this chapter:

*   **Managing services with Upstart**: Investigate how to control services in CentOS 6.5 using Upstart and the `/etc/init` and `/etc/event.d` directories
*   **Creating your own Upstart script**: Learn how to create custom startup script for Upstart to manage your own needs at system boot
*   **Managing processes**: Gain practice with the p series of tools from the package procps to manage running processes: ps, pstree, pgrep, pmap, and pkill

# Managing services with Upstart

For many years now, we have become used to managing services with System V init scripts that date back many years. However, without the desire to improve and the dedicated meliorism of so many in the open source community, we would not move on and improve. In CentOS 6.5, we now see some services being managed from the configurations within the `/etc/init` directory. These services make use of Upstart. This may be short-lived, as the beta release of Red Hat Enterprise Linux 7 uses a similar service manager, systemd, which is likely to prevail over Upstart. That said, both Upstart and systemd are managed in a very similar way, so visiting Upstart here is not an issue.

Firstly, we can check to see that we are indeed using `upstart` using the following command:

```
# yum list upstart

```

The output from the preceding command should list the package as being installed. The service uses the `/etc./init` directory for its configuration and from here, we can see the services that utilize Upstart. Using the `rpm` command, we can check which package created this directory:

```
# rpm -qf /etc/init

```

The output shows the directory belonging to the `upstart` package. On CentOS, we have just a few services that are controlled via Upstart; however, we will see that it is very easy to configure our own services as required using this mechanism.

Within the `/etc/init` directory, we can see that the consoles screens are managed by `upstart` with the `tty.conf` file, and we can also see the `splash-manager.conf` file. This service is enabled for reboot and poweroff, runlevel 0 and 6, to request the Plymouth splash screens (remember Plymouth from [Chapter 2](ch02.html "Chapter 2. Cold Starts"), *Cold Starts*). The beauty of both Upstart and systemd is that we do not need to manage the symbolic links in the run control directories the way we required with older System V scripts. Taking a look inside the `/etc/init/spash-manager.conf` file, we can see when the service will run and that this is provided directly without reference to the old symbolic links. The following is an example of an Upstart directive:

```
start on starting rc RUNLEVEL=[06]
```

The preceding line states that Upstart will run the scripts contained in the configuration file on starting and on changing to runlevel 0 and 6\. It is as simple as that. The configuration may also contain everything that is required to run the service; this file becomes the autonomous service we have been looking for, self-contained in the single configuration file.

# Creating your own Upstart script

One of the best ways in which you can learn about these services is to create your own configuration file containing the Upstart script and all the associated conditions for our service. The configuration file will require the extension of `.conf` and has to be created in the `/etc/init` directory. For the purpose of this demonstration, we will create a simple service with the well-researched and inventive name: `sample`.

Using the text editor vi to create the `/etc/init/sample.conf` file, the service begins to take shape:

```
#/etc/init/sample.conf
description "Simple demonstration upstart script"
author "The Urban Penguin"
start on runlevel [35]
script
  logger -p local1.info "Starting upstart service"
end script
```

The service itself does nothing other than use the logger program to write to the syslog daemon; we can read the output from the `/var/log/messages` logfile. You, of course, could adjust the service to do more; however, this acts as a great start in demonstrating how compact the services can be.

We can test the service using the `/sbin/initctl` command:

```
# initctl start sample
# tail -n 1 /var/log/messages

```

The service can be manually started as we have shown here, if not manually started, it will start automatically, or at least send the message to the logfile when the system enters runlevel 3 or 5.

![Creating your own Upstart script](img/5920OS_05_01.jpg)

Along with the script option that we used here, we can additionally run pre or post scripts, especially useful where the service is a binary and we need to endure a certain environment in which it can run. There is an `exec` directive that can be used in place of the `script` directive where a single binary should run in place of the script. The full life cycle of an upstart service includes:

*   Pre-start
*   Post-start
*   Main (using exec or script)
*   Pre-stop
*   Post-stop

The advantages that we gain from using Upstart and/or systemd is that we are no longer restricted to only starting services for given runlevels; we can also start these services for events such as disks (or block devices) being added or other services being started. Many of these event driven services can be found in the `/etc/event.d` directory.

We can control Upstart services using the `/sbin/initctl` command. To view the options available, the following command can be used with the `help` option:

```
# initctl help

```

The output will show that the `version` option can be used to check the version of Upstart in use, as seen in the following screenshot:

![Creating your own Upstart script](img/5920OS_05_02.jpg)

# Managing processes

The bulk of this chapter will visit the `procps` package and the p series commands that we can use to manage our processes to make sure that we can fully appreciate the power that these tools can offer from the command line.

Many administrators are accustomed to using the `ps` command to determine the running process, and often the output is then piped to `grep` to search for a given process name. Although there is nothing incorrect with this as such, we may prefer to use tools that streamline these steps and are specifically designed tools for the purpose. For the moment, we will ignore the `/ps` command in preference of more specific tools with a real purpose to their binary lives.

## Using the pgrep command

The `/usr/bin/pgrep` command really does become a snap-in replacement for the `ps` and `grep` pipelines we use all too often. For example, if I start my Apache web server, I can easily check the **Process IDs** (**PIDs**) in use by the service:

```
# service httpd start
# pgrep httpd

```

The output of the command follows:

![Using the pgrep command](img/5920OS_05_03.jpg)

From the output, I can see the lowest PID, in my case **3950**. This will be the main daemon process that will spawn the child processes. Knowing how much we enjoy the easy life, we can see how effortlessly we can the child processes to the parent using the `pstree` command with the parent process as the argument. There are eight child processes in my case, no need to use my thumbs to count these.

```
# pstree 3950

```

Here's the output to the preceding command:

![Using the pgrep command](img/5920OS_05_04.jpg)

## Using the pstree command

This utility is so under used; I am sure that many administrators just know to execute the command without arguments or options. We have seen with the previous example how powerful it can be, but of course there is much more.

The `pstree` command, when run without arguments or options, will list the process tree from PID 1 through all running processes showing each in a hierarchical form linked by **parent process ID** (**PPID**) and PID.

To take it a step further, we can use the `-h` option, which will highlight the process tree in which we run `pstree`:

```
$ pstree -h

```

The following screenshot shows the highlighted extract from `pstree`:

![Using the pstree command](img/5920OS_05_05.jpg)

We can see from the previous screenshot that we have run `pstree` from a `gnome-terminal` in which we had been running the Bash shell in a **substituted user** (**SU**) environment; it is all laid out clearly for us to see.

Running something similar, we can highlight the process tree for the `httpd` process running at PID `3950` using the following command:

```
$ pstree -h 3950

```

On my system, this can be seen in the following extract from the screenshot:

![Using the pstree command](img/5920OS_05_06.jpg)

Using `pstree` with the `-a` option will show the process and any arguments used when it was started. This can be useful to see that a service is taking the correct configuration options:

```
$ pstree -a

```

## Using the pkill command

When we reconsider from where we started this chapter, with the understanding that many administrators are accustomed to using the `ps` command and piped to `grep`, we should question the need to run the `ps` command in the first place. Perhaps we do this in order to kill a process. We did show earlier in this chapter how we can simplify and streamline the `ps`/`grep` pipeline with `pgrep`; the reality though, is that, we can streamline it further with `pkill`. Let's assume that we are running CentOS as a desktop, and while running the Firefox web browser we notice that is has become unresponsive, we can simply use the `pkill` to resolve the issue:

```
$ pkill firefox

```

Of course, the process names in Linux are case sensitive, but you do get used to the names of those processes that you may often need to whip into shape with a little `pkill`.

We can also kill all processes owned by a particular user. They may have done something really bad, and their session is totally unresponsive. We could try:

```
# pkill -9 -U bob

```

The preceding command will send the kill signal (`-9`) to all processes owned by the user `bob`.

If it is just processes in one terminal that we need to concern ourselves with, then we could use the following command:

```
# pkill -9 -t tty2

```

This will kill all processes running in the `tty2` console. If we own the running processes, we can execute the `pkill` command as our own account. If we don't own those processes, then the commands need to be run as root. We have been using the `-9` signal in the last few examples, as we want to be sure to remove all process. It may be better to run the commands in the following way for best practice:

```
# pkill -15 -t tty2
# pkill -9 -t tty2

```

In this way, we issue the terminate signal first (`-15`) and then the kill (`-9`). In this way, processes are shutdown gracefully if they respond to the initial terminate request; those that are unresponsive to the terminate signal are then cleaned up with the final kill.

Just because a process does not respond to the terminate request, it does not mean that the process has hung; the bash shell, for example, does not respond to a terminate request, it has to be killed outright. This is just how the program is written.

To see a list of signals that can be issued, we should use the `/usr/bin/kill` command with the `-l` (list) option as follows:

```
$ kill -l

```

An extract from the output can be seen in the following screen capture:

![Using the pkill command](img/5920OS_05_07.jpg)

Although many possible signals exist, they must be written into the program if it is to respond to the signal. This is where the `-9` or `-sigkill` option is useful, although abrupt, the signal works directly with `init` to remove the process rather than relying on the application to respond.

## Using the pmap command

Another useful tool from the `procps` package that we can use to gain information on running processes is the `pmap` command. This can be used to print a memory map for a running process. In simple terms, it will show you how much memory is being used by the process and any library modules that it uses.

To gain information on the PID for the current shell, we can use the special variable `$$`; thus, `pmap $$` will show me the process map for my current shell:

```
$ pmap $$

```

The output looks most impressive; firstly we see that `$$` on this system resolves to the PID 7259 and `/bin/bash`. As the process map continues, we see the address column followed by the size of memory in use by each component of `/bin/bash`.

From the following partial screenshot, we can see the output that you can expect from the `pmap` command:

![Using the pmap command](img/5920OS_05_08.jpg)

# Summary

In this chapter, we have been able to while away a little boondoggle time, learning about the `procps` package in CentOS Linux and all the treasures that it can reveal. The tools therein are a gold mine to administrators and can save us all valuable time where we elect the most effective tool to be used in the examining processes. Putting this into practice, we have seen the effortless use that can be made from the `pgrep` and `pkill` commands in streamlining our process management, while tools such as `pmap` are more useful for diagnosing system resource usage.

As we get ready to venture into the next chapter, we will look at some of the valuable shortcuts that we can use when managing users on a CentOS 6.5 system.