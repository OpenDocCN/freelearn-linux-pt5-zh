# Chapter 2. Working with Commands and Plugins

In this chapter, we will cover the following recipes:

*   Finding a plugin
*   Installing a plugin
*   Removing a plugin
*   Customizing an existing command
*   Using an alternative check command for a host
*   Writing a new plugin from scratch

# Introduction

Nagios Core is perhaps best thought of less as a monitoring tool, and more as a monitoring framework. Its modular design can use any kind of program which returns appropriate values based on some kind of check, such as a `check_command` plugin for a host or service. This is where the concepts of commands and plugins come into play.

For Nagios Core, a **plugin** is any program that can be used to gather information about a host or service. To ensure that a host was responding to PING requests, we'd use a plugin, such as `check_ping`, which when run against a hostname or address—whether by Nagios Core or not—would return a status code to whatever called it, based on whether a response was received to the PING request within a certain period of time. This status code and any accompanying message is what Nagios Core uses to establish what state a host or service is in.

Plugins are generally just like any other program on a Unix-like system; they can be run from the command line, are subject to permissions and owner restrictions, can be written in any programming language, and can take parameters and options to modify how they work. Most importantly, they are entirely separate from Nagios Core itself (even if programmed by the same people), and the way that they're used by the application can be changed.

To allow for additional flexibility in how plugins are used, Nagios Core uses these programs according to the terms of a command definition. A command for a specific plugin defines the way in which that plugin is used, including its location in the filesystem, any parameters it should be passed, and any other options. In particular, parameters and options often include thresholds for `WARNING` and `CRITICAL` states.

Nagios Core is usually downloaded and installed along a set of plugins called **Nagios Plugins**, available at [http://www.nagiosplugins.org/](http://www.nagiosplugins.org/), which this book assumes you have installed. These plugins were chosen because, as a set, they cover the most common needs for a monitoring infrastructure quite well, including checks for common services, such as web, mail services, and DNS services, as well as more generic checks, such as whether a TCP or UDP port is accessible and open on a server. It's likely that for most of our monitoring needs, we won't need any other plugins; but if we do, Nagios Core makes it possible to use existing plugins in novel ways using custom command definitions, adding third-party plugins written by contributors on the Nagios Exchange website, or even writing custom plugins ourselves from scratch in some special cases.

# Finding a plugin

In this recipe, we'll follow a good procedure for finding a plugin appropriate to a specific monitoring task. We'll start by checking to see if an existing plugin is already available to do just what we need. If we can't find one, we'll check to see if we can use another more generic plugin to solve the problem. If we still find that nothing suits, we'll visit Nagios Exchange and search for an appropriate plugin there.

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already, and you'll need to have a particular service on one of those hosts, which you're not sure how to monitor.

We'll use a simple problem as an example; we have a server named `troy.naginet` that runs an `rsync` process that listens on port `873`. We're already monitoring the host's network connectivity via `PING`, but we'd like to have Nagios Core check whether the `rsync` server is available and listening at all times, in case it crashes while running or doesn't start up when the system is rebooted.

## How to do it...

We can find a new plugin appropriate to any monitoring task as follows:

1.  Firstly, since we have the Nagios Core Plugins set installed, we'll check to see if any of the plugins available in it apply directly to our problem. We'll start by visiting `/usr/local/nagios/libexec` on our Nagios Core server, and getting a directory listing:

    ```
    # cd /usr/local/nagios/libexec
    # ls
    check_apt       check_ide_smart     check_nntp      check_simap
    check_breeze    check_ifoperstatus  check_nntps     check_smtp
    check_by_ssh    check_ifstatus      check_nt        check_spop
    ...

    ```

    There's a long list of plugins there, but none of them look like `check_rsync` or `check_backup`, so it doesn't quite seem like there's a plugin in the core to do exactly what we need.

2.  However, there is a plugin called `check_tcp`. A web search for its name pulls up its manual page on the Nagios Plugins website as the first result, and a description of what it does:

    *"This plugin tests TCP connections with the specified host (or unix socket)."*

    We need to do more than just check the port, so this doesn't quite suit us either.

3.  A web search for `check_rsync`, which would be an appropriate name for the plugin, turns up a page on the Nagios Exchange website with a plugin named exactly that. We've found an appropriate plugin now:![How to do it...](img/5566_02_01.jpg)

## How it works...

If all we needed to do was check that `rsync` was listening on port `873`, and we didn't really need to monitor any of its actual function, then the `check_tcp` plugin might actually suffice. However, in our case, we might need to find a way to not only check that a port is open, but also check that a specific directory or `rsync` module is accessible.

Reading the description for `check_rsync`, it looks like it has the exact functionality we need, checking that a certain `rsync` module is available on the server. At this point, we could download the plugin and follow its installation instructions.

## There's more...

This recipe is intended to highlight that in addition to having a capable set of plugins as a part of the Nagios Core Plugins set, the documentation available online on the Nagios Core Plugins website at [http://nagiosplugins.org/](http://nagiosplugins.org/) and the other plugins available on Nagios Exchange at [http://exchange.nagios.org/](http://exchange.nagios.org/) make it relatively straightforward to find an appropriate plugin for the particular monitoring problem we need to solve.

Note that when we download third-party plugins, it's important to check that we trust the plugin to do what we need it to. **Nagios Exchange** is a moderated community with a coding standard, but the plugins are provided at our own risk; if we don't understand what a plugin does, we should be wary of installing it or using it without reading its code, its documentation, and its reviews.

## See also

*   The *Installing a plugin*, *Removing a plugin*, and *Writing a new plugin from scratch* recipes in this chapter

# Installing a plugin

In this recipe, we'll install a custom plugin that we retrieved from Nagios Exchange onto a Nagios Core server, so that we can use it as a Nagios Core command and hence check a service with it.

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already, and have found an appropriate plugin to install, to solve some particular monitoring need. Your Nagios Core server should have internet connectivity to allow you to download the plugin directly from the website.

In this example we'll use `check_rsync`, which is available on the Web at [http://exchange.nagios.org/directory/Plugins/Network-Protocols/Rsync/check_rsync/details](http://exchange.nagios.org/directory/Plugins/Network-Protocols/Rsync/check_rsync/details).

This particular plugin is quite simple, consisting of a single Perl script with very basic dependencies. If you want to install this script as an example, then the server will also need to have a Perl interpreter installed; it's installed in `/usr/bin/perl` on many systems.

This example will also include directly testing a server running an `rsync` daemon called `troy.naginet`.

## How to do it...

We can download and install a new plugin as follows:

1.  Copy the URL for the download link for the most recent version of the `check_rsync` plugin:![How to do it...](img/5566_02_02.jpg)
2.  Navigate to the plugins directory for the Nagios Core server. The default location is `/usr/local/nagios/libexec`:

    ```
    # cd /usr/local/nagios/libexec

    ```

3.  Download the plugin using `wget` into a file called `check_rsync`. It's important to surround the URL in quotes:

    ```
    # wget 'http://exchange.nagios.org/components/com_mtree/attachment.php?link_id=307&cf_id=29' -O check_rsync

    ```

4.  Make the plugin executable using `chmod` and `chown`:

    ```
    # chown nagios.nagios check_rsync
    # chmod 0770 check_rsync

    ```

5.  Run the plugin directly with no arguments to check that it runs, and to get usage instructions. It's a good idea to test it as the `nagios` user using `su` or `sudo`:

    ```
    # sudo -s -u nagios
    $ ./check_rsync
    Usage: check_rsync -H <host> [-p <port>] [-m <module>[,<user>,<password>] [-m <module>[,<user>,<password>]...]]

    ```

6.  Try running the plugin directly against a host running `rsync`, to see if it works and reports a status:

    ```
    $ ./check_rsync -H troy.naginet
    Output normally starts with the status determined, with any extra information after a colon:
    OK: Rsync is up

    ```

If all of this works, then the plugin is now installed and working correctly.

## How it works...

Because Nagios Core plugins are programs in themselves, installing a plugin amounts to saving a program or script into an appropriate directory; in this case, `/usr/local/nagios/libexec`, where all the other plugins live. It's then available to be used the same way as any other plugin.

The next step once the plugin is working is defining a command in the Nagios Core configuration for it, so that it can be used to monitor hosts and/or services. This can be done with the *Creating a new command* recipe in this chapter.

## There's more...

If we inspect the Perl script, we can see a little bit of how it works. It works like any other Perl script, except for the fact that its return values are defined in a hash table called `%ERRORS`, and the return values it chooses depend on what happens when it tries to check the `rsync` process. This is the most important part of implementing a plugin for Nagios Core.

Installation procedures for different plugins vary. In particular, many plugins are written in languages such as C, and hence need be compiled. One such plugin is the popular `check_nrpe`. Rather than simply being saved into a directory and made executable, these sorts of plugins often follow the usual pattern of configuration, compilation, and installation:

```
$ ./configure
$ make
# make install

```

For many plugins that are built in this style, the last step in that process will often install the compiled plugin into the appropriate directory for us. In general, if instructions are included with the plugin, then it pays to read them so that we can ensure we install it correctly.

## See also

*   The *Finding a plugin*, *Removing a plugin*, and *Creating a new command* recipes in this chapter

# Removing a plugin

In this recipe, we'll remove a plugin that we no longer need as part of our Nagios Core installation. Perhaps it's not working correctly, the service it monitors is no longer available, or there are security or licensing concerns with its usage.

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already, and have a plugin that you would like to remove from the server. In this instance, we'll remove the now unneeded `check_rsync` plugin from our Nagios Core server.

## How to do it...

We can remove a plugin from our Nagios Core instance as follows:

1.  Remove any part of the configuration that uses the plugin, including hosts or services that use it for `check_command`, and command definitions that refer to the program. As an example, the following definition for a command would no longer work after we removed the `check_rsync` plugin:

    ```
    define command {
        command_name  check_rsync
        command_line  $USER1$/check_rsync -H $HOSTADDRESS$
    }
    ```

    Using a tool such as `grep` can be a good way to find mentions of the command and plugin:

    ```
    # grep -R check_rsync /usr/local/nagios/etc

    ```

2.  Change directory on the Nagios Core server to wherever the plugins are kept. The default location is `/usr/local/nagios/libexec`:

    ```
    # cd /usr/local/nagios/libexec

    ```

3.  Delete the plugin with the `rm` command:

    ```
    # rm check_rsync

    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

## How it works...

Nagios Core plugins are simply external programs that the server uses to perform checks of hosts and services. If a plugin is no longer wanted, all that needs to be done is to remove references to it in our configuration, if any, and then delete the plugin program from `/usr/local/nagios/libexec`.

## There's more...

Usually there's not any harm in leaving the plugin's program on the server even if Nagios Core isn't using it. It doesn't slow anything down or cause any other problems, and it may be needed later. Nagios Core plugins are generally quite small programs, and should not really cause disk space concerns on a modern server.

## See also

*   The *Finding a plugin*, *Installing a plugin*, and *Creating a new command* recipes in this chapter

# Creating a new command

In this recipe, we'll create a new command for a plugin that was just installed into the `/usr/local/nagios/libexec` directory on the Nagios Core server. This will define the way in which Nagios Core should use the plugin, and thereby allow it to be used as part of a service definition.

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already, and have a plugin installed for which you'd like to define a new command. This will allow you to use it as part of a service definition. In this instance, we'll define a command for an installed `check_rsync` plugin.

## How to do it...

We can define a new command in our configuration as follows:

1.  Change to the directory containing the objects configuration for Nagios Core. The default location is `/usr/local/nagios/etc/objects`:

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the `commands.cfg` file:

    ```
    # vi commands.cfg

    ```

3.  At the bottom of the file, add the following command definition:

    ```
    define command {
        command_name  check_rsync
     command_line  $USER1$/check_rsync -H $HOSTADDRESS$
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

If the validation passes and the server restarts successfully, we should be able to use the `check_rsync` command in a service definition.

## How it works...

The configuration we added to the `commands.cfg` file defines a new command called `check_rsync`, which defines a method for using the plugin of the same name to monitor a service. This enables us to use `check_rsync` as a value for the `check_command` directive in a service declaration, which might look similar to the following code snippet:

```
define service {
    use                  generic-service
    host_name            troy.naginet
    service_description  RSYNC
 check_command        check_rsync
}
```

Only two directives are required for command definitions, and we've defined both:

*   `command_name`: This defines the unique name with which we can reference the command when we use it in host or service definitions.
*   `command_line`: This defines the command line that should be executed by Nagios Core to make the appropriate check.

This particular command line also uses two macros:

*   `$USER1$`: This expands to `/usr/local/nagios/libexec`, the location of the plugin binaries, including `check_rsync`. It is defined in the sample configuration in the file `/usr/local/nagios/etc/resource.cfg`.
*   `$HOSTADDRESS$`: This expands to the address of any host for which this command is used as a host or service definition.

So if we used the command in a service checking the `rsync` server on `troy.naginet`, then the completed command might look similar to the following:

```
$ /usr/local/nagios/libexec/check_rsync -H troy.naginet

```

We could run this straight from the command line ourselves as the nagios user to see what kind of results it returns:

```
$ /usr/local/nagios/libexec/check_rsync -H troy.naginetOK: Rsync is up

```

## There's more...

A plugin can be used for more than one command. If we had a particular `rsync` module to check with the configured name of `backup`, we could write another command called `check_rsync_backup` , as follows, to check this module is available:

```
define command {
    command_name  check_rsync_backup
 command_line  $USER1$/check_rsync -H $HOSTADDRESS$ -m backup
}
```

Or if one or more of our `rsync` servers was running on an alternate port, say port `5873`, then we could define a separate command, `check_rsync_altport`, for that:

```
define command {
    command_name  check_rsync_altport
 command_line  $USER1$/check_rsync -H $HOSTADDRESS$ -p 5873
}
```

Commands can thus be defined as precisely as we need them to be. We explore this in more detail in the *Customizing an existing command* recipe in this chapter.

## See also

*   The *Installing a plugin* and *Customizing an existing command* recipes in this chapter

# Customizing an existing command

In this recipe, we'll customize an existing command definition. There are a number of reasons why you might want to do this, but a common one is if a check is "overzealous", sending notifications for `WARNING` or `CRITICAL` states, which aren't actually terribly worrisome. It can also be useful if a check is too "forgiving" and doesn't detect actual problems with hosts or services.

Another reason is to account for peculiarities in your own network. For example, if you run HTTP daemons on a large number of hosts on the alternative port `8080` that you need to check, it would be convenient to have a `check_http_altport` command available. We can do this by copying and altering the definition for the vanilla `check_http` command.

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already. You should also already be familiar with the relationship between services, commands, and plugins.

## How to do it...

We can customize an existing command definition as follows:

1.  Change to the directory containing the objects configuration for Nagios Core. The default location is `/usr/local/nagios/etc/objects`:

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the `commands.cfg` file, or any file which is at an appropriate location for the `check_http` command:

    ```
    # vi commands.cfg

    ```

3.  Find the definition for the `check_http` command. In a default Nagios Core configuration, it should look similar to the following:

    ```
    # 'check_http' command_definition
    define command {
     command_name  check_http
     command_line  $USER1$/check_http -H $HOSTADDRESS$ $ARG1$
    }

    ```

4.  Copy this definition into a new definition directly below it and alter it to look similar to the following code snippet, renaming the command and adding a new option to its command line:

    ```
    # 'check_http_altport' command_definition
    define command {
     command_name  check_http_altport
     command_line  $USER1$/check_http -H $HOSTADDRESS$ -p 8080 $ARG1$
    }

    ```

5.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

If the validation passed and the server restarted successfully, we should now be able to use the `check_http_altport` command, which is based on the original `check_http` command, in a service definition.

## How it works...

The configuration we added to the `commands.cfg` file reproduces the command definition for `check_http`, but changes it in two ways:

*   It renames the command from `check_http` to `check_http_alt`, which is necessary to distinguish the commands from one another. Command names in Nagios Core, just like host names, must be unique.
*   It adds the option `-p 8080` to the command-line call, specifying the time when the call to `check_http` is made. The check will be made using TCP port `8080`, rather than the default value for TCP port `80`.

The `check_http_alt` command can now be used as a check command in the same way as a `check_http` command. For example, a service definition that checks whether the `sparta.naginet` host is running an HTTP daemon on port `8080` might look similar to the following code snippet:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
 service_description  HTTP_8080
 check_command        check_http_alt
}
```

## There's more...

This recipe's title implies that we should customize the existing commands by editing them in place, and indeed, this works fine if we really do want to do things this way. Instead of copying the command definition, we could just add the `-p 8080` or another customization to the command line and change the original command.

However, this is bad practice in most cases, mostly because it could break the existing monitoring and be potentially confusing to other administrators of the Nagios Core server. If we have a special case for monitoring—in this case, checking a non-standard port for HTTP—then it's wise to create a whole new command based on the existing one with the customizations we need.

There is no limit to the number of commands you can define, so you can be very liberal in defining as many alternative commands as you need. It's a good idea to give them instructive names that say something about what they do, as well as to add explanatory comments to the configuration file. You can add a comment to the file by prefixing it with a `#` character:

```
#
# 'check_http_altport' command_definition. This is to keep track of
# servers that have panels running on alternative ports.
#
define command {
 command_name  check_http_altport
 command_line  $USER1$/check_http -H $HOSTADDRESS$ -p 8080 $ARG1$
}

```

## See also

*   The *Creating a new command* recipe in this chapter
*   The *Creating a new service* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Using an alternative check command for hosts

In this recipe, we'll learn how to deal with a slightly tricky case in network monitoring—monitoring a server that doesn't respond to PING, but still provides some network service that requires checking.

It's good practice to allow PING where you can, as it's one of the stipulations in **RFC 1122** and a very useful diagnostic tool not just for monitoring, but also for troubleshooting. However, sometimes servers that are accessed only by a few people might be configured not to respond to these messages, perhaps for reasons of secrecy. It's quite common for domestic routers to be configured this way.

Another very common reason for this problem, and the example we'll address here, is checking servers that are behind an **IPv4 NAT** firewall. It's not possible to address the host directly via an **RFC1918** address, such as `192.168.1.20`, from the public Internet. Pinging the public interface of the router therefore doesn't tell us whether the host for which it is translating addresses is actually working.

However, port `22` for SSH is forwarded from the outside to this server, and it's this service that we need to check for availability.

![Using an alternative check command for hosts](img/5566_02_03.jpg)

We'll do this by checking whether the host is up through an SSH check, since we can't PING it from the outside as we normally would.

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already. You should also already be familiar with the relationship between services, commands, and plugins.

## How to do it...

We can specify an alternative check method for a host as follows:

1.  Change to the directory containing the objects configuration for Nagios Core. The default location is `/usr/local/nagios/etc/objects`:

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Find the file that contains the host definition for the host that won't respond to PING, and edit it. In this example, our `crete.naginet` host is the one we want to edit:

    ```
    # vi crete.naginet.cfg

    ```

3.  Change or define the `check_command` parameter of the host to the command that we want to use for the check instead of the usual `check-host-alive` or `check_ping` plugin. In this case, we want to use `check_ssh`. The resulting host definition might look similar to the following code snippet:

    ```
    define host {
        use            linux-server
        host_name      crete.naginet
        alias          crete
        address        10.128.0.23
        check_command  check_ssh
    }
    ```

    Note that defining `check_command` still works even if we're using a host template, such as `generic-host` or `linux-server`. It's a good idea to check that the host will actually respond to our check as we expect it to:

    ```
    # sudo -s -u nagios
    $ /usr/local/nagios/libexec/check_ssh -H 10.128.0.23
    SSH OK - OpenSSH_5.5p1 Debian-6+squeeze1 (protocol 2.0)

    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the next scheduled host check for the `crete.naginet` server should show the host as `UP`, because it was checked with the `check_ssh` command and not the usual `check-host-alive` command.

## How it works

The configuration we added for the `crete.naginet` host uses `check_ssh` to check whether the host is `UP`, rather than a check that uses PING. This is appropriate because the only public service accessible from `crete.naginet` is its SSH service.

![How it works](img/5566_02_04.jpg)

The `check_ssh` command is normally used to check whether a service is available, rather than a host. However, Nagios Core allows us to use it as a host check command as well. Most service commands work this way; you could check a web server behind NAT in the same way with `check_http`.

## There's more...

Note that for completeness' sake, it would also be appropriate to monitor the NAT router via PING, or some other check appropriate to its public address. That way, if the host check for the SSH server fails, we can check to see if the NAT router in front of it is still available, which assists in troubleshooting whether the problem is with the server or with the NAT router in front of it. You can make this setup even more useful by making the NAT router a parent host for the SSH server behind it, explained in the *Creating a network host hierarchy* recipe in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), *Understanding the Network Layout*.

## See also

*   The *Monitoring SSH for any host* and *Checking an alternative SSH port* recipes in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*
*   The *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*
*   The *Creating a network host hierarchy* and *Establishing a host dependency* recipes in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), *Understanding the Network Layout*

# Writing a new plugin from scratch

Even given the very useful standard plugins in the Nagios Plugins set, and the large number of custom plugins available on Nagios Exchange, occasionally as our monitoring setup grows more refined, we may find that there is some service or property of a host that we would like to check, but for which there doesn't seem to be any suitable plugin available. Every network is different, and sometimes the plugins that others have generously donated their time to make for the community don't quite cover all your bases. Generally, the more specific your monitoring requirements get, the less likely it is that there's a plugin available that does exactly what you need.

In this example, we'll deal with a very particular problem that we'll assume can't be dealt with effectively by any known Nagios Core plugins, and we'll write one ourselves using Perl. Here's the example problem:

Our Linux security team wants to be able to automatically check whether any of our servers are running kernels that have known exploits. However, they're not worried about every vulnerable kernel, only specific versions. They have provided us with the version numbers of three kernels having small vulnerabilities that they're not particularly worried about but that do need patching, and one they're extremely worried about.

Let's say the minor vulnerabilities are in the kernels with version numbers `2.6.19`, `2.6.24`, and `3.0.1`. The serious vulnerability is in the kernel with version number `2.6.39`. Note that the version numbers in this case are arbitrary and don't necessarily reflect any real kernel vulnerabilities!

The team could log in to all of the servers individually to check them, but the servers are of varying ages and access methods, and managed by different people. They would also have to check manually more than once, because it's possible that a naive administrator could upgrade to a kernel that's known to be vulnerable in an older release, and they also might want to add other vulnerable kernel numbers for checking later on.

So, the team have asked us to solve the problem with Nagios Core monitoring, and we've decided the best way to do it is to write our own plugin, `check_vuln_kernel`, which checks the output of `uname` for a kernel version string, and then does the following:

*   If it's one of the slightly vulnerable kernels, then it will return a `WARNING` state, so that we can let the security team know that they should address it when they're next able to.
*   If it's the highly vulnerable kernel version, then it will return a `CRITICAL` state, so that the security team knows a patched kernel needs to be installed immediately.
*   If `uname` gives an error or output we don't understand, then it will return an `UNKNOWN` state, alerting the team to a bug in the plugin or possibly more serious problems with the server.
*   Otherwise, it returns an `OK` state, confirming that the kernel is not known to be a vulnerable one.
*   Finally, they want to be able to see at a glance in the Nagios Core monitoring what the kernel version is, and whether it's vulnerable or not.

For the purposes of this example, we'll only monitor the Nagios Core server itself, but via NRPE we'd be able to install this plugin on the other servers that require this monitoring, where they'll work just as well. You should see the *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution* to learn how to do this.

While this problem is very specific, we'll approach it in a very general way, which you'll be able to adapt to any solution where it's required for a Nagios plugin to:

1.  Run a command and pull its output into a variable.
2.  Check the output for the presence or absence of certain patterns.
3.  Return an appropriate status based on those tests.

All that means is that if you're able to do this, you'll be able to effectively monitor anything on a server from Nagios Core!

## Getting ready

You should have a Nagios Core 3.0 or newer server running with a few hosts and services configured already. You should also already be familiar with the relationship between services, commands, and plugins. You should also have Perl installed.

This will be a rather long recipe that ties in a lot of Nagios Core concepts. You should be familiar with all the following concepts:

*   Defining new hosts and services, and how they relate to one another
*   Defining new commands, and how they relate to the plugins they call
*   Installing, testing, and using Nagios Core plugins

Some familiarity with Perl would also be helpful, but is not required. We'll include comments to explain what each block of code is doing in the plugin.

## How to do it...

We can write, test, and implement our example plugin as follows:

1.  Change to the directory containing the plugin binaries for Nagios Core. The default location is `/usr/local/nagios/libexec`:

    ```
    # cd /usr/local/nagios/libexec

    ```

2.  Start editing a new file called `check_vuln_kernel`:

    ```
    # vi check_vuln_kernel

    ```

3.  Include the following code in it; take note of the comments, which explain what each block of code is doing:

    ```
    #!/usr/bin/env perl

    #
    # Use strict Perl style and report potential problems to help us write this
    # securely and portably.
    #
    use strict;
    use warnings;

    #
    # Include the Nagios utils.pm file, which includes definitions for the return
    # statuses that are appropriate for each level: OK, WARNING, CRITICAL, and
    # UNKNOWN. These will become available in the %ERRORS hash.
    #
    use lib "/usr/local/nagios/libexec";
    use utils "%ERRORS";
    #
    # Define a pattern that matches any kernel vulnerable enough so that if we find
    # it we should return a CRITICAL status.
    #
    my $critical_pattern = "^(2\.6\.39)[^\\d]";

    #
    # Same again, but for kernels that only need a WARNING status.
    #
    my $warning_pattern = "^(2\.6\.19|2\.6\.24|3\.0\.1)[^\\d]";

    #
    # Run the command uname with option -r to get the kernel release version, put
    # the output into a scalar $release, and trim any newlines or whitespace
    # around it.
    #
    chomp(my $release = qx|/bin/uname -r|);

    #
    # If uname -r exited with an error status, that is, anything greater than 1,
    # then there was a problem and we need to report that as the UNKNOWN status
    # defined by Nagios Core's utils.pm.
    #
    if ($? != 0) {
     exit $ERRORS{UNKNOWN};
    }

    #
    # Check to see if any of the CRITICAL patterns are matched by the release
    # number. If so, print the version number and exit, returning the appropriate
    # status.
    #
    if ($release =~ m/$critical_pattern/) {
     printf "CRITICAL: %s\n", $release;
     exit $ERRORS{CRITICAL};
    }

    #
    # Same again, but for WARNING patterns.
    #
    if ($release =~ m/$warning_pattern/) {
     printf "WARNING: %s\n", $release;
     exit $ERRORS{WARNING};
    }

    #
    # If we got this far, then uname -r worked and didn't match any of the
    # vulnerable patterns, so we'll print the kernel release and return an OK
    # status.
    #
    printf "OK: %s\n", $release;
    exit $ERRORS{OK};

    ```

4.  Make the plugin owned by the `nagios` user and executable with `chmod`:

    ```
    # chown nagios.nagios check_vuln_kernel# chmod 0770 check_vuln_kernel
    Run the plugin directly to test it:
    # sudo -s -u nagios
    $ ./check_vuln_kernel
    OK: 2.6.32-5-686

    ```

We should now be able to use the plugin in a command, and hence in a service check, just like any other command. Note that the code for this plugin is included in the code bundle of this book for your convenience.

## How it works...

The code we added in the new plugin file `check_vuln_kernel` is actually quite simple:

1.  It runs `uname -r` to get the version number of the kernel.
2.  If that didn't work, it exits with a status of `UNKNOWN`.
3.  If the version number matches anything in a pattern containing critical version numbers, it exits with a status of `CRITICAL`.
4.  If the version number matches anything in a pattern containing warning version numbers, it exits with a status of `WARNING`.
5.  Otherwise, it exits with a status of `OK`.

It also prints the status as a string, along with the kernel version number, if it was able to retrieve one.

We might set up a command definition for this plugin as follows:

```
define command {
    command_name  check_vuln_kernel
    command_line  $USER1$/check_vuln_kernel
}
```

In turn, we might set up a service definition for that command as follows:

```
define service {
    use                  local-service
    host_name            localhost
    service_description  VULN_KERNEL
 check_command        check_vuln_kernel
}
```

If the kernel was not vulnerable, the service's appearance in the web interface might look similar to the following screenshot:

![How it works...](img/5566_02_05.jpg)

However, if the monitoring server itself happened to be running a vulnerable kernel, then it might look more similar to the following screenshot (and send consequent notifications, if configured to do so):

![How it works...](img/5566_02_06.jpg)

## There's more...

This may be a simple plugin, but its structure can be generalized to all sorts of monitoring tasks. If we can figure out the correct logic to return the status we want in an appropriate programming language, then we can write a plugin to do basically anything.

A plugin like this could just as effectively be written in C for improved performance, but we'll assume for simplicity's sake that high performance for the plugin is not required. Instead, we can use a language that's better suited for quick ad hoc scripts like this one; in this case, we use Perl. The file `utils.sh`, also in `/usr/local/nagios/libexec`, allows us to write in shell script if we'd prefer that.

If you write a plugin that you think could be generally useful for the Nagios community at large, then please consider putting it under a free software license and submitting it to the Nagios Exchange, so that others can benefit from your work. Community contribution and support is what has made Nagios Core such a great monitoring platform in such wide use.

Any plugin you publish in this way should conform to the Nagios Plugin Development Guidelines. At the time of writing, these are available at [http://nagiosplug.sourceforge.net/developer-guidelines.html](http://nagiosplug.sourceforge.net/developer-guidelines.html).

Finally, you should note that the method of including `utils.pm`, used in this example, may be deprecated in future versions of Nagios Core. It is used here for simplicity's sake. The new method of including it in Perl is done with a CPAN module called `Nagios::Plugin`.

## See also

*   The *Creating a new command* and *Customizing an existing command* recipes in this chapter
*   The *Creating a new service recipe* in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, a**nd Contacts*
*   The *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*