# Chapter 6. Enabling Remote Execution

In this chapter, we will cover the following recipes:

*   Monitoring local services on a remote machine with NRPE
*   Setting the listening address for NRPE
*   Setting allowed client hosts for NRPE
*   Creating new NRPE command definitions securely
*   Giving limited sudo privileges to NRPE
*   Using check_by_ssh with key authentication instead of NRPE

# Introduction

For a dedicated Nagios Core server with access to all the relevant parts of the network, making checks is relatively simple using commands and plugins that make ICMP, TCP, and UDP connections to network hosts and services, in order to determine their operating state. These can be used to check any sort of network service, without requiring anything to be installed on the target machine. As an example, when the `check_http` plugin is used to check a web server, it works in the same way as if a browser was making the request.

However, monitoring a network thoroughly usually has more to it than simply checking network connectivity and availability. It's also a good idea to check properties of the network that don't directly correspond to a network service, and hence can't be directly checked over a network connection.

These are often properties of hardware or the underlying system, such as disk space or system load average, or processes that are configured only to listen locally, commonly done for database servers.

We could install Nagios Core on all of the systems, perhaps, but this would make maintenance difficult. It would be much better to have some means of remote execution of diagnostic programs, so that they are run directly on the target host to retrieve the information they need, and the results are returned to a single Nagios Core server via a dedicated network service.

There are three general approaches to managing this problem:

*   Use `check_nrpe` to run a standard Nagios Core plugin on the target machine and return its results transparently to the monitoring server.
*   Use `check_by_ssh` to run an arbitrary command on the target machine from the monitoring server by first connecting to it with SSH.
*   Use `check_snmp` to check an SNMP OID that's configured to provide the return value and output of some command on the target host.

This chapter covers the first two solutions, focusing on the more commonly used **Nagios Remote Plugin Executor** (**NRPE**), and explaining how it differs from the `check_by_ssh` solution. For some information on configuring SNMP, see the *Monitoring the output of an SNMP query* and *Creating an SNMP OID to monitor* recipes in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*.

# Monitoring local services on a remote machine with NRPE

In this recipe, we'll learn how to install and run an NRPE server on a target host, `roma.naginet`. We'll use this to check the load average on that host with the `check_load` plugin.

The plugins for these checks will be executed on the target server by the NRPE daemon, but the results will be returned to our Nagios Core monitoring server `olympus.naginet`. This requires installing the `check_nrpe` plugin on the monitoring server, and the full Nagios Plugins set (but not Nagios Core itself) on the target server.

This is a reasonably long and in-depth recipe as it involves installing a total of three software packages on two servers.

## Getting ready

You will need a monitoring server with Nagios Core 3.0 or newer installed. You should also have a UNIX-like target host that you intend to monitor that can run the NRPE daemon. Most modern UNIX-like systems including Linux and BSD should be able to do this. Both the monitoring server and the target host will need internet connectivity, and you should already be monitoring the target host itself with a host definition, to which we'll be adding service checks.

If your servers don't have a direct gateway to the Internet, then you can work around this by uploading the relevant files after downloading them onto a workstation, or another machine with Internet access.

You should understand the basics of configuring, compiling, and installing software from source. In most cases, the usual `./configure`, `make`, and `make install` process will be all that's necessary, and the recipe will walk you through this. You will need to have `make` installed, along with any other tools needed for the configure and build processes, including a C compiler such as `gcc`.

You should also have a good grasp of how hosts and services interrelate in Nagios Core, which is discussed in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), and how Nagios Core uses commands and plugins, discussed in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"). You should not need an in-depth understanding of the use of any particular plugin; the recipe will demonstrate the usage of the plugins to which it refers.

Finally, you should be able to configure any firewalls to allow connectivity from the Nagios Core server to the server being monitored with TCP destination port `5666`.

## How to do it...

This first part of the recipe is done on the target server:

1.  Download and install the latest Nagios Plugins package. At the time of writing, the link is available at [http://nagiosplugins.org/download/](http://nagiosplugins.org/download/).

    ```
    $ wget http://downloads.sourceforge.net/project/nagiosplug/nagiosplug/1.4.16/nagios-plugins-1.4.16.tar.gz
    $ tar -xzf nagios-plugins-1.4.16.tar.gz

    ```

2.  Configure, compile, and install the plugins, the same way you would on a new monitoring server. You will need to have `root` privileges for the `make install` call.

    ```
    $ cd nagios-plugins-1.4.16
    $ ./configure
    $ make
    # make install

    ```

    You may need to install some shared libraries and headers on the system to do this for certain plugins, such as a `libssl` implementation. The output of the `./configure` script should alert you to any such problems.

3.  Download and install the latest version of NRPE from the Nagios Exchange website. At the time of writing, the link is available at [http://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details](http://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details).

    ```
    $ wget http://prdownloads.sourceforge.net/sourceforge/nagios/nrpe-2.13.tar.gz
    $ tar -xzf nrpe-2.13.tar.gz

    ```

4.  Enter the `nrpe-2.13` source directory, and configure, compile, and install the daemon and a stock configuration for it. You will need to have `root` privileges for both the `make install-daemon` and the `make install-daemon-config` calls:

    ```
    $ cd nrpe-2.13
    $ ./configure
    $ make all
    # make install-daemon
    # make install-daemon-config

    ```

    If you do not already have a `nagios` user on the target host, you may need to create one before the daemon will install properly:

    ```
    # groupadd nagios
    # useradd -r -g nagios nagios

    ```

5.  Edit the newly installed file at `/usr/local/nagios/etc/nrpe.cfg` and find the line beginning with `allowed_hosts`. Add a comma and the IP address of your monitoring server to this line. In this case, we've added the IP address `10.128.0.11`:

    ```
    allowed_hosts=127.0.0.1,10.128.0.11

    ```

6.  Start the `nrpe` daemon, and check that it is running by searching the process table with `pgrep` or `ps`:

    ```
    # /usr/local/nagios/bin/nrpe -c /usr/local/nagios/etc/nrpe.cfg -d
    # pgrep nrpe
    18593
    # ps -e | grep [n]rpe
    nagios 18593 1 0 21:55 ? 00:00:01 nrpe

    ```

7.  If you would like the `nrpe` daemon to start on boot, add an `init` script appropriate to your system. An example `init-script` is generated at `./configure` time in the source directory. Versions are also generated for Debian-derived systems and SUSE systems in `init-script.debian` and `init-script.suse`. Exactly how this should be done will depend on your particular system, for which you may need to consult its documentation.

This next part of the recipe is done on the monitoring server.

1.  Again, download the latest version of NRPE, the same way as done for the target server:

    ```
    $ wget http://prdownloads.sourceforge.net/sourceforge/nagios/nrpe-2.13.tar.gz
    $ tar -xzf nrpe-2.13.tar.gz

    ```

2.  Again, configure and build the software. However, note that this time the install line is different, as we're installing the `check_nrpe` plugin rather than the `nrpe` daemon:

    ```
    $ cd nrpe-2.13.tar.gz
    $ ./configure
    $ make all
    # make install-plugin

    ```

3.  Check that the plugin is installed correctly. It should be saved at `/usr/local/nagios/libexec/check_nrpe`:

    ```
    $ ls /usr/local/nagios/libexec/check_nrpe
    /usr/local/nagios/libexec/check_nrpe

    ```

4.  Move to the directory containing the Nagios Core object configuration. By default, this is `/usr/local/nagios/etc/objects`:

    ```
    $ cd /usr/local/nagios/etc/objects

    ```

5.  Edit an appropriate file for defining new commands. For the default installation, `/usr/local/nagios/etc/objects/commands.cfg` is a good choice. Add the following definition to the end of this file:

    ```
    define command {
        command_name  check_nrpe
        command_line  $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
    }
    ```

6.  Edit the file defining the target host as an object. The definition might look something similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  roma.naginet
        alias      roma
        address    10.128.0.61
    }
    ```

7.  Beneath the definition for the host, after any other services defined for it, add the following service definition:

    ```
    define service {
        use                  generic-service
        host_name            roma.naginet
        service_description  LOAD
        check_command        check_nrpe!check_load
    }
    ```

8.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service with the description `LOAD` will appear in the web interface ready to be checked, and will come up with an appropriate status, including the load average as read from the `nrpe` daemon on the target host:

![How to do it...](img/5566_06_01.jpg)

We can see more detail about how the check was performed and its results in the details page for the service:

![How to do it...](img/5566_06_02.jpg)

If the load average on `roma.naginet` exceeds the limits defined for the `check_load` command in `/usr/local/nagios/etc/nrpe.cfg` on the target host, the service will enter `WARNING` or `CRITICAL` states, and will send notifications if configured to do so, all in the same manner as a non-NRPE service.

## How it works...

The NRPE plugin and daemon are used to run Nagios Core plugins on the target host, rather than on the monitoring server itself. The results of the check are then passed back to the monitoring server, and recorded and analyzed by Nagios Core the same way as if the service was running a plugin on the monitoring server, for example `check_http` or `check_ssh`.

The recipe we followed does four main things:

*   We installed the latest Nagios Plugins package to the target host, including the `check_load` plugin. This is necessary because the plugin is actually run on the target host and not on the monitoring server, as is the case with the plugins that check network services.
*   We installed the `nrpe` daemon to the target host, along with a stock configuration file `nrpe.cfg`. This is the network service through which the `check_nrpe` plugin will request commands to be run on the target host. The plugins will be run by this process, typically as the `nagios` user.
*   We installed the `check_nrpe` plugin to the monitoring host, and defined a command of the same name to use it. The command accepts one argument in the `$ARG1$` macro; its value is the command that should be run on the target host. In this case, we supplied `check_load` for this argument.
*   We set up a service to monitor the output of the standard `check_load` plugin, via `check_nrpe`.

Like other Nagios Core plugins, the `check_nrpe` program can be run directly from the command line. If we wanted to test the response of the configuration that we arranged in the previous section, then we might run the following command:

```
$ /usr/local/nagios/libexec/check_nrpe -H roma.naginet -c check_load
OK - load average: 0.00, 0.00, 0.00|load1=0.000;15.000;30.000;0;
load5=0.000;10.000;25.000;0; load15=0.000;5.000;20.000;0;

```

In this case, the state of `OK` and the load average values, as retrieved by `check_load`, were returned by the `nrpe` daemon as the result of the `check_nrpe` call.

![How it works...](img/5566OS_06_03.jpg)

It's very important to note that this simple configuration of NRPE is not completely secure by default. The recipes listed under the *See also* section for this recipe provide some basic means to secure NRPE instances from abuse. These should be used in concert with a sensible firewall policy.

## There's more...

Of course, `check_load` is not the only plugin that can be run on the target server this way. If we inspect the `/usr/local/nagios/etc/nrpe.cfg` file `/usr/local/nagios/etc/nrpe.cfg` on the target host, near the end of the file, we find some other example definitions of commands that `check_nrpe` will run upon requests issued from the monitoring server:

```
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/hda1
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

```

We recognize `check_load` as the second of these. Note that it already includes some thresholds for `WARNING` and `CRITICAL` alerts in its `-w` and `-c` parameters.

If we also wanted to check the number of processes on this server, we could add a service check for `roma.naginet`, defined as follows:

```
define service {
    use                  generic-service
    host_name            roma.naginet
    service_description  PROCS
    check_command        check_nrpe!check_total_procs
}
```

This service will generate a `WARNING` alert if the number of processes exceeds 150, and a `CRITICAL` alert if it exceeds 200\. Again, the plugin is run on the target server, and not on the monitoring server.

Another useful and common application of `check_nrpe` is to make remote checks on database servers, with plugins such as `check_mysql` and `check_pgsql`, in the case where servers do not listen on network interfaces for security reasons. Instead, they listen only on `localhost` or UNIX sockets, and are hence inaccessible to the monitoring server. To work around this problem, we could add a new command definition to the end of `nrpe.cfg` on the target server as follows:

```
command[check_mysql]=/usr/local/nagios/libexec/check_mysql -u nagios -d nagios -p wGG7H233bq

```

A corresponding check that uses the `check_mysql` command can then be made on the monitoring server:

```
define service {
    use                  generic-service
    host_name            roma.naginet
    service_description  MYSQL
    check_command        check_nrpe!check_mysql
}
```

See the *Monitoring database services* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*, for some detail on how to use the `check_mysql` and `check_pgsql` plugins.

NRPE is thus useful not only for making checks of system properties or hardware, but also for any plugin that needs to be run on the target host rather than the monitoring host.

Finally, it's important to note that the command definitions included in the default `nrpe.cfg` file are intended as examples; you will probably want to fine-tune the parameters for some of them, and remove the ones you don't use, along with adding your own.

## See also

*   The *Setting the listening address for NRPE*, *Setting allowed client hosts for NRPE*, *Creating new NRPE command definitions securely*, and *Giving limited sudo privileges to NRPE* recipes in this chapter
*   The *Monitoring database services* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Setting the listening address for NRPE

In this recipe, we'll learn how to make NRPE listen on a specific IP address on a target host. This might be done on hosts with multiple interfaces in order to prevent spurious requests made to the `nrpe` daemon from untrusted interfaces, perhaps the public Internet. It could also be appropriate for configuring the daemon only to listen on a trusted VPN interface.

This setup can be particularly useful when the server has an interface into a dedicated management network to which the monitoring server also has access, preventing the `nrpe` daemon from responding to requests on other interfaces unnecessarily, and thereby closing a possible security hole.

## Getting ready

You should have a target host configured for checking in a Nagios Core 3.0 or later monitoring server. The target host should be running the `nrpe` daemon, and listening on all interfaces (which we'll fix). You can verify that `nrpe` is running with `pgrep` or `ps`:

```
# pgrep nrpe
29964
# ps -e | grep [n]rpe
nagios 29964 1 0 21:55 ? 00:00:01 nrpe

```

You can check whether the `nrpe` daemon is listening on all interfaces by checking the output of `netstat`:

```
# netstat -plnt | grep nrpe
tcp 0 0 0.0.0.0:5666 0.0.0.0:* LISTEN 29964/nrpe

```

The address of `0.0.0.0` shows that `nrpe` is listening on all interfaces, which is what we'd like to correct.

## How to do it...

We can configure the `nrpe` daemon only to listen on one address as follows:

1.  Edit the `nrpe` daemon's configuration file. The default location is `/usr/local/nagios/etc/nrpe.cfg`. Look for the line beginning with `server_address`, which is normally commented out by default:

    ```
    #server_address=127.0.0.1

    ```

    If you don't have such a line, then you can add it at the end of the file.

2.  Uncomment the line if it's commented by removing the leading `#` character, and change the `127.0.0.1` address to the address to which you want to restrict the `nrpe` process listening:

    ```
    server_address=10.128.0.61

    ```

3.  Restart the `nrpe` daemon. If you have installed an `init` script for it, you may be able to do this with the following:

    ```
    # /etc/init.d/nrpe restart

    ```

    If not, you can restart the process by sending it a `HUP` signal with the `kill` command, which will prompt it to re-read its configuration file and resume running:

    ```
    # pgrep nrpe
    29964
    # kill -HUP 29964

    ```

With this done, the `nrpe` daemon should now only be listening on the specified address. We can verify this using `netstat`:

```
# netstat -plnt | grep nrpe
tcp 0 0 10.128.0.61:5666 0.0.0.0:* LISTEN 29964/nrpe

```

## How it works...

The configuration we adjusted in the preceding section defines an address on which the `nrpe` daemon should listen, and implies that it should not respond to requests on any others.

Because the `nrpe` server is explicitly designed to run commands at the request of remote servers, it's very important to take steps like these wherever appropriate to prevent attackers from exploiting the service.

## See also

*   The *Monitoring local services on a remote machine with NRPE* recipe in this chapter.

# Setting allowed client hosts for NRPE

In this recipe, we'll learn how to configure the NRPE daemon to answer requests from a particular IP address, typically the designated Nagios Core server or servers monitoring your network. This means that `nrpe` will not run plugins or return results for any `check_nrpe` request made from IP addresses not in this list.

This is an elementary security step in running an NRPE server. This should be done in concert with a hardware or software firewall and security policy. If your target host has interfaces or routes into untrusted networks, there is a risk of attackers making spurious requests for information about the system, clogging up your disk with logs from excessive check requests, or even possibly exploiting the `nrpe` daemon or the Nagios Plugins.

## Getting ready

You should have a target host configured for checking in a Nagios Core 3.0 or later monitoring server. The target host should be running the `nrpe` daemon. You can verify that `nrpe` is running with `pgrep` or `ps`:

```
# pgrep nrpe
29964
# ps -e | grep [n]rpe
nagios 29964 1 0 21:55 ? 00:00:01 nrpe

```

We can verify that the target host is not configured to respond to a particular IP address by attempting to open a `telnet` or `netcat` connection to it. If we are not one of the allowed hosts, `nrpe` will close the session immediately without waiting for any input:

```
$ telnet roma.naginet 5666
Trying 10.128.0.61...
Connected to 10.128.0.61.
Escape character is '^]'.
Connection closed by foreign host.

```

This assumes that NRPE is listening on its default port number `5666`. In this example, we'll add the IP address `10.128.0.12` to the list of hosts allowed to request information from NRPE.

## How to do it...

We can configure the `nrpe` daemon to respond to a new address as follows:

1.  Edit the `nrpe` daemon's configuration file. The default location is `/usr/local/nagios/etc/nrpe.cfg`. Look for the line beginning with `allowed_hosts`. It may look similar to the following code snippet:

    ```
    allowed_hosts=127.0.0.1,10.128.0.11

    ```

2.  Add or remove IP addresses from the line, separating them with commas. For this example, we're adding one more address:

    ```
    allowed_hosts=127.0.0.1,10.128.0.11,10.128.0.12

    ```

3.  Restart the `nrpe` daemon. If you have installed an `init` script for it, you may be able to this with something similar to the following:

    ```
    # /etc/init.d/nrpe restart

    ```

    If not, you can restart the process by sending it a `HUP` signal with the `kill` command, which will prompt it to re-read its configuration file and resume running:

    ```
    # pgrep nrpe
    29964
    # kill -HUP 29964

    ```

With this done, the `nrpe` daemon should now respond only to the nominated hosts making `check_nrpe` requests, immediately closing the connection otherwise. We can verify whether our new host is allowed to talk to the `nrpe` service on `roma.naginet` with another `telnet` test:

```
$ telnet roma.naginet 5666

```

Note that the `nrpe` daemon is now waiting for input, rather than closing the connection immediately as it was doing before. This implies that we can now run `check_nrpe` checks from `10.128.0.12`, if we need to.

## How it works...

The configuration we adjusted above defines a set of addresses to which the `nrpe` daemon should respond if a request is made, and implies that it should refuse to answer requests made by any other address.

The `nrpe` daemon inspects the IP address of incoming connections, and if the `allowed_hosts` directive is defined, checks that the address features in that list. If it does not, it closes the connection and refuses to run any plugins, much less return any output from them.

## There's more...

The `allowed_hosts` directive is actually optional; if we wished, we could set the `nrpe` server up to respond to requests from any IP address. The default installation and example configuration, however, enables it by default, allowing both requests from the localhost IP `127.0.0.1`, and any network addresses the host had at the time `./configure` was run.

This is a sensible policy because Nagios Core plugins are designed by third parties in an open source community, for monitoring purposes in trusted networks, and may not necessarily be very secure. A plugin doesn't actually have to have a security hole to cause such problems; if the check it makes is very resource-intensive, for example opening a lot of TCP connections or querying a large database, an attacker could cause problems on the target host, if allowed, simply by making a large number of such `check_nrpe` requests in a short period of time.

## See also

*   The *Monitoring local services on a remote machine with NRPE* recipe in this chapter

# Creating new NRPE command definitions securely

In this recipe, we'll learn how to securely create new command definitions for `nrpe` to run upon request by a monitoring server. We need to do this because even if we have a huge set of plugins installed on our target host running `nrpe`, the daemon will only run commands defined in its configuration file.

We'll also learn how arguments can be passed to these commands if strictly necessary, and about the potentially negative security consequences of this.

## Getting ready

You should have a target host configured for checking in a Nagios Core 3.0 or later monitoring server. The target host should be running the `nrpe` daemon. You can verify that `nrpe` is running with `pgrep` or `ps`:

```
# pgrep nrpe
29964
# ps -e | grep [n]rpe
nagios 29964 1 0 21:55 ? 00:00:01 nrpe

```

We can inspect the list of commands that `nrpe` is already configured to run by looking for `command` directives in its configuration file. By default, this file is `/usr/local/nagios/etc/nrpe.cfg`, and the default command definitions are near the end of the file:

```
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/hda1
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

```

We'll add another command to this set to check whether the swap space available is above a specific threshold, using the standard Nagios Core plugin `check_swap`. We can test whether it is working first by running it on the target host:

```
$ /usr/local/nagios/libexec/check_swap -w 10% -c 5%
SWAP OK - 100% free (216 MB out of 217 MB) |swap=216MB;21;10;0;217

```

For completeness, we'll also show how to define a service check using this new plugin on the Nagios Core monitoring server.

## How to do it...

We can add a new command definition to an `nrpe` configuration as follows:

1.  Edit the `nrpe` daemon's configuration file. The default location is `/usr/local/nagios/etc/nrpe.cfg`. Look for lines beginning with `command`, which are near the end of the file by default.
2.  Add the following line to the end of the file:

    ```
    command[check_swap]=/usr/local/nagios/libexec/check_swap -w 10% -c 5%

    ```

3.  Restart the `nrpe` daemon. If you have installed an `init` script for it, you may be able to do this with something similar to the following command:

    ```
    # /etc/init.d/nrpe restart

    ```

    If not, you can restart the process by sending it a `HUP` signal with the `kill` command, which will prompt it to re-read its configuration file and resume running:

    ```
    # pgrep nrpe
    29964
    # kill -HUP 29964

    ```

With this done, assuming that our monitoring server is part of the `allowed_hosts` directive and can contact the target host, a call to `check_nrpe` on the monitoring host should return the status and output of the `check_swap` plugin on the target host:

```
$ /usr/local/nagios/libexec/check_nrpe -H roma.naginet -c check_swap
SWAP OK - 100% free (216 MB out of 217 MB) |swap=216MB;21;10;0;217

```

In turn, this allows us to use the check in a service definition on the monitoring server, with a `check_nrpe` command:

```
define service {
    use                  generic-service
    host_name            roma.naginet
    service_description  SWAP
    check_command        check_nrpe!check_swap
}
```

## How it works...

The configuration added in the preceding section defines a new command in `nrpe.cfg` called `check_swap`. The definition of new commands in `nrpe.cfg` takes the following general form:

```
command[command_name] = command_line

```

We defined a command `check_swap` for NRPE. It doesn't accept any arguments, where the actual `check_swap` plugin requires them; instead, the arguments are hard-coded into the command definition, with the two options `-w 10%` and `-c 5%` setting the thresholds for free swap space.

Besides checking system properties, such as the load average or the swap space, which might not otherwise be directly retrievable except via systems such as SNMP, another common use for NRPE is to report on the state of network services that only listen locally, or are otherwise unreachable by the monitoring host. Database server monitoring is a good example. We could define the following command in `nrpe.cfg`:

```
command[check_mysql] = /usr/local/nagios/libexec/check_mysql -u nagios -d nagios -p NFmxenQ5

```

Assuming that the `check_mysql` plugin is installed on this target host, which it ought to be if the MySQL client library and headers were available at compile time, this command would then enable this check to be run from the monitoring host:

```
$ /usr/local/nagios/libexec/check_nrpe -H roma.naginet -c check_mysql
Uptime: 420865  Threads: 1  Questions: 172  Slow queries: 0  Opens: 99
Flush tables: 1  Open tables: 23  Queries per second avg: 0.0

```

This can be configured as a service check using an appropriate command definition for `check_nrpe` as follows:

```
define service {
    use                  generic-service
    host_name            roma.naginet
    service_description  MYSQL
    check_command        check_nrpe!check_mysql
}
```

Thus we're able to get the status of the MySQL server on the remote host `roma.naginet` without actually connecting directly to the MySQL server itself; we arrange for the NRPE service on the target host to do it on the monitoring server's behalf.

This is not just useful for network services running on the target host. It can be used to delegate any kind of check that a remote host can perform where the monitoring server cannot. Using NRPE is thus also a way to work around the addressability problems of NAT, because a service running on an address such as `192.168.1.1` would not be addressable from outside the network. If NRPE were running on the NAT gateway, we could use that to address the appropriate systems by their local addresses.

## There's more...

Also, near the bottom of `nrpe.cfg`, you'll find some information on providing arguments to NRPE commands as part of the `check_nrpe` request, as opposed to hard-coding them. The comment included in the file makes it quite clear that this carries some risks:

> The following examples allow user-supplied arguments and can only be used if the NRPE daemon was compiled with support for command arguments AND the dont_blame_nrpe directive in this config file is set to '1'. This poses a potential security risk, so make sure you read the SECURITY file before doing this.

It's important to understand that, when running NRPE on a target host, you are running a service that is designed to allow network machines to run commands on the target machine with no strong authentication, which is why keeping NRPE secure is so important. If you allow passing arguments to commands, you need to be aware of the full ramifications and risks of doing so, and the recommended `SECURITY` file explains these well.

If you really want to use it, it requires reconfiguring and recompiling the `nrpe` daemon with the `--enable-command-args` switch:

```
$ ./configure --enable-command-args
$ make all
# make install-daemon

```

Then set the `dont_blame_nrpe` in `nrpe.cfg` parameter to `1`, where it otherwise defaults to `0`:

```
dont_blame_nrpe=1
```

After restarting `nrpe` (if you have rebuilt it, you will need to restart it completely this time, and not simply send the process a `HUP` signal), this allows us to use command definitions similar to the following:

```
command[check_mysql_args]=/usr/local/nagios/libexec/check_mysql -H localhost -u $ARG1$ -d $ARG2$ -p $ARG3$

```

This command in turn allows checks from the monitoring servers like this, using the `-a` option of `check_nrpe`:

```
# /usr/local/nagios/libexec/check_nrpe -H roma.naginet -c check_mysql_args -a nagios nagios NFmxenQ5

```

Because of the security concerns, I would recommend you avoid using command arguments if at all possible. If you do absolutely need to use them, it's also important to make sure that the traffic is encrypted, especially if it contains usernames and passwords. Advice on how to manage this is included in the `SECURITY` document for the `nrpe` daemon.

## See also

*   The *Monitoring local services on a remote machine with NRPE* and *Giving limited sudo privileges to NRPE* recipes in this chapter
*   The *Monitoring database services* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Giving limited sudo privileges to NRPE

In this recipe, we'll learn how to deal with the difficulty of execution permissions for NRPE. The majority of the standard Nagios plugins don't require special privileges to run, although this depends on how stringent your system's security restrictions are. However, some of the plugins require being run as `root`, or perhaps as a user other than `nrpe`. This is sometimes the case with plugins that need to make requests of system-level resources, such as checking the integrity of `RAID` arrays.

There are four general approaches to fixing this:

*   **Bad**: One method is to change the plugins to `setuid`, meaning that they will always be run as the user who owns them, no matter who executes them. The problem with this is that setting this bit allows anyone to run the program as `root`, not just `nrpe`, a very common vector for exploits.
*   **Worse**: Another method is to run `nrpe` as `root`, or as the appropriate user. This is done by changing the `nrpe_user` and `nrpe_group` properties in `nrpe.cfg`. This is even more dangerous, and completely inconsistent with the principle of least privilege; we should confer a user as little permission as possible to allow it to do its job. Never do this!
*   **Better**: A third method is to use `command_prefix` in `nrpe.cfg` to prepend `/usr/bin/sudo` to all commands, and gives `nrpe` full `sudo` privileges to run only the plugins in `/usr/local/nagios/libexec`. This is a bit better, but still quite risky as we probably don't need every single command to be run as `root`, only one or two.
*   **Best**: The best method is to use `sudo` to assign the `nrpe` user limited privileges for a subset of commands, only the ones it needs to run, and only as the user by which it needs to be run.

The last solution is the most likely to be secure, so we'll examine an example here. We'll run the `check_procs` plugin as `root`, to get a process count. In most cases, you wouldn't need `root` privileges to get a complete count of all processes, but it might be needed on a system with a very locked-down `grsecurity` patch installed.

## Getting ready

You should have a target host configured for checking in a Nagios Core 3.0 or later monitoring server. The target host should be running the `nrpe` daemon, and listening on all interfaces. You can verify that `nrpe` is running with `pgrep` or `ps`:

```
# pgrep nrpe
29964
# ps -e | grep [n]rpe
nagios 29964 1 0 21:55 ? 00:00:01 nrpe

```

You should also have `sudo` installed and working on the target system, and understand what it does. We'll be editing the `/etc/sudoers` file to confer `root` privileges to our `nrpe` user, for one program only. This recipe will assume that the `nrpe` daemon is running as the `nagios` user in the `nagios` group.

## How to do it...

We can confer limited `root` privileges for one command to our `nrpe` user as follows:

1.  Edit the `/etc/sudoers` file. The safest way to do this is generally with a call to `visudo`, which will make a temporary copy of the file and verify its syntax is correct before installing it:

    ```
    # visudo

    ```

2.  Add the following line to the file, and save it:

    ```
    nagios ALL=(ALL) NOPASSWD: /usr/local/nagios/libexec/check_procs
    ```

    Note that if the `requiretty` directive appears anywhere in your `/etc/sudoers` file, you may need to remove it to make this work.

3.  Become the `nagios` user with `sudo`, and test whether running the command runs as `root`, with no password prompt:

    ```
    # sudo -s -u nagios
    $ sudo /usr/local/nagios/libexec/check_procs
    PROCS OK: 89 processes

    ```

4.  Edit the `nrpe` daemon's configuration file. The default location is `/usr/local/nagios/etc/nrpe.cfg`. Look for the command definition for `check_total_procs`, and if there isn't one, create it. Note that `/usr/bin/sudo` has been added to the start of the command:

    ```
    command[check_total_procs]=/usr/bin/sudo /usr/local/nagios/libexec/check_procs -w 150 -c 200

    ```

5.  Restart the `nrpe` daemon. If you have installed an `init` script for it, you may be able to do this with something similar to the following:

    ```
    # /etc/init.d/nrpe restart

    ```

    If not, you can restart the process by sending it a `HUP` signal with the `kill` command, which will prompt it to re-read its configuration file and resume running:

    ```
    # pgrep nrpe
    29964
    # kill -HUP 29964

    ```

With this done, we should now be able to run a `check_nrpe` call from the monitoring server, and get a successful response:

```
$ /usr/local/nagios/libexec/check_nrpe -H roma.naginet -c check_total_procs
PROCS OK: 89 processes

```

## How it works...

The preceding configuration does not change the behavior of `nrpe` very much; most of the configuration is actually done on its host system. All we changed was the command definition for `check_total_procs` to run it from within `sudo`.

To make this work without a password, we defined it in the `/etc/sudoers` file so that no password was required to execute this particular program as `root` for the `nagios` user. Because `nrpe` runs as the `nagios` user, it is therefore able to use `sudo` with no password for this command only.

This means that when we call the `check_total_procs` command from the monitoring server, it returns us the full output of the plugin as it was run with `root` privileges, but the `nagios` user doesn't have `root` privileges to run anything else potentially dangerous, such as `rm` or `halt`.

## There's more...

While this is a much more secure way of allowing privileges as another user for `nrpe`, it still requires trusting that the plugin that is being run with `root` privileges is secure and can't easily be exploited. Be very careful before running this with custom code or with stray plugins you find on the Web!

If you intend to allow the `nagios` user to run more than a couple of distinct programs, it may look a little tidier to define them in `/etc/sudoers` with `Cmnd_Alias`:

```
Cmnd_Alias NAGIOS = /usr/local/nagios/libexec/check_procs, /usr/local/nagios/libexec/check_load
nagios ALL=(ALL) NOPASSWD: NAGIOS
```

## See also

*   The *Monitoring local services on a remote machine with NRPE* and *Using check_by_ssh with key authentication instead of NRPE* recipes in this chapter

# Using check_by_ssh with key authentication instead of NRPE

While all of the previous recipes in this chapter show that NRPE can be very effectively tied down and secured, it may be that we require some means of authentication to a target host in order to run the appropriate Nagios plugins on it. The `nrpe` daemon does not require any authentication to return information about the host's state; as long as the IP addresses all match, and the command is defined for running, it will return information.

If you already use SSH keys for a public key infrastructure in your network, then you may find it preferable to use the `check_by_ssh` plugin instead, which allows you to use public keys to authenticate with a target host before running any commands. This is only suitable if the target host runs an `ssh` daemon.

In this recipe, we'll repeat the setup for the `check_load` plugin as done in the first recipe in this chapter, *Monitoring local services on a remote machine with NRPE*, but we'll use the `check_by_ssh` plugin instead.

## Getting ready

You should have a Nagios Core 3.0 or newer server ready, with the `ssh` client software installed, and a target host running `sshd`. The OpenSSH implementations should work fine.

The target host should have all of the Nagios Plugins installed; in this case, we're using `check_load`.

You should be familiar with public key authentication and its advantages and disadvantages. Wikipedia has an excellent article about public key cryptography and authentication at [http://en.wikipedia.org/wiki/Public-key_cryptography](http://en.wikipedia.org/wiki/Public-key_cryptography).

There is a popular introduction to SSH authentication with a key infrastructure at [http://support.suso.com/supki/SSH_Tutorial_for_Linux](http://support.suso.com/supki/SSH_Tutorial_for_Linux).

We will be running through a sensible key infrastructure setup, but it would pay to have an understanding of how it works in case this setup does not suit your configuration.

## How to do it...

We can arrange to get the output of `check_load` from our remote host by way of `check_by_ssh` as follows:

1.  On the monitoring server, decide on a location for the private and public keys for the `nagios` user. I recommend placing them in `/usr/local/nagios/keys`. Create this directory and make it owned by the `nagios` user, and only readable by that user:

    ```
    # KEYSDIR=/usr/local/nagios/keys
    # mkdir -p $KEYSDIR
    # chown nagios.nagios $KEYSDIR
    # chmod 0700 $KEYSDIR

    ```

2.  Become the `nagios` user using `su` or `sudo`:

    ```
    # su nagios
    # sudo -s -u nagios

    ```

3.  Generate a pair of private and public SSH keys using `ssh-keygen`. Here we've used 2048-bit RSA; use whichever key length and cipher is appropriate for your network setup.

    ```
    $ ssh-keygen -b 2048 -t rsa -f /usr/local/nagios/keys/id_rsa

    ```

    When prompted for a passphrase, simply press *Enter* to signal that you don't want one.

    This should create two files in `/usr/local/nagios/keys`, called `id_rsa` and `id_rsa.pub`. The first one is the **private key**, and should be kept secret at all times. The second, the **public key**, is safe to distribute and to install on other machines.

4.  Decide on a location for the `authorized_keys` file on the target host. The easiest way to do this is probably to specify a `$HOME` directory for the `nagios` user, and to create it if appropriate:

    ```
    # NAGIOSHOME=/home/nagios
    # usermod -d $NAGIOSHOME nagios
    # mkdir -p $NAGIOSHOME/.ssh
    # chown -R nagios.nagios $NAGIOSHOME
    # chmod 0700 $NAGIOSHOME/.ssh

    ```

5.  Copy the `/usr/local/nagios/keys/id_rsa.pub` file from the monitoring server to `/home/nagios/.ssh/authorized_keys` on the target machine. The best way to do this varies. One possible method is to use `scp`:

    ```
    $ whoami
    nagios
    $ scp /usr/local/nagios/keys/id_rsa.pub roma.naginet:.ssh/authorized_keys

    ```

    You may have to set a password temporarily for the `nagios` user on the target host to do this:

    ```
    # passwd nagios
    Enter new UNIX password:
    Retype new UNIX password:
    passwd: password updated successfully.

    ```

6.  Check that you can now log in from the monitoring server to the target server as the `nagios` user with no password:

    ```
    $ whoami
    nagios
    $ ssh -i /usr/local/nagios/keys/id_rsa roma.naginet
    Linux roma 2.6.32-5-686 #1 SMP Mon Oct 3 04:15:24 UTC 2011 i686
    Lost login: Sun Jul 29 18:23:14 2012 from olympus.naginet

    ```

7.  Back on the monitoring server, check that you're able to run the `check_by_ssh` plugin to call one of the installed plugins on the remote server; in this example, we're calling `check_load`:

    ```
    $ whoami
    nagios
    $ /usr/local/nagios/libexec/check_by_ssh \
     -H roma.naginet \
     -i /usr/local/nagios/keys/id_rsa \
     -C '/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,24,20'
    OK - load average: 0.00, 0.00, 0.00|load1=0.000;15.000;30.000;0;
    load5=0.000;10.000;24.000;0; load15=0.000;5.000;20.000;0;

    ```

    Note that the value for the `-C` option needs to be surrounded with quotes.

8.  Now that we've verified that `check_by_ssh` works with our infrastructure, we can define a command to use it in `/usr/local/nagios/etc/objects/commands.cfg`:

    ```
    define command {
        command_name  check_by_ssh
        command_line  $USER1$/check_by_ssh -H $HOSTADDRESS$ -i /usr/local/nagios/keys/id_rsa -C '$ARG1$'
    }
    ```

9.  We can apply that command to a new service check for `roma.naginet` as follows, preferably placed beneath the host definition:

    ```
    define service {
        use                  generic-service
        host_name            roma.naginet
        service_description  LOAD_BY_SSH
        check_command        check_by_ssh!/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
    }
    ```

10.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service with the description `LOAD_BY_SSH` should appear in the web interface ready to be checked, and will come up with an appropriate status, including the load average as read via `SSH` to the target host. To Nagios Core, the results of the check are just the same as if they'd come via NRPE.

## How it works...

Checking services via NRPE and SSH is actually a reasonably similar process; the central idea is that we're using another network service shared by the monitoring server and the target host to instruct the target host to run a plugin, and return the results to the monitoring server.

In the case of `check_by_ssh`, this is done over an SSH connection. It's quite typical for administrators to run commands on remote hosts using the `ssh` client, simply by adding the command to be run to the command line:

```
$ hostname
olympus.naginet
$ ssh roma.naginet hostname
roma.naginet

```

All that the `check_by_ssh` plugin does is formalizes this process into a Nagios plugin context. The value for the `command_line` directive for the command we defined can be broken down as follows:

*   `$USER1$/check_by_ssh`: This is the path to the plugin, as is typical in command definitions, using the `$USER1$` macro to expand to `/usr/local/nagios/libexec`.
*   `-H $HOSTADDRESS$`: This specifies that the plugin should connect to the applicable host for any host or service that uses this command.
*   `-i /usr/local/nagios/keys/id_rsa`: This specifies the location of the private key to be used for identification with the remote host.
*   `-C '$ARG1$'`: This specifies that the command run by `check_by_ssh` on the target host is given in the first argument the service defines in its `check_command`; in this case, our command is:

    ```
    /usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20

    ```

Note that it's important that `$ARG1$` is in quotes, because we want to pass the whole command as one argument to `check_by_ssh`.

## There's more...

Using `check_by_ssh` instead of `check_nrpe` allows full authentication via public key, rather than merely by having the right IP address. It also encrypts all traffic implicitly, minimizing the risk of sensitive data, such as usernames or passwords, from being intercepted. Which of the two plugins you should use will depend very much on the nature of your network and your security policy. There is no reason you can't use both of them if you wish.

Be careful not to confuse this plugin with `check_ssh`, which checks that the SSH service itself is running, and is discussed in the *Monitoring SSH for any host* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*.

## See also

*   The *Monitoring local services on a remote machine with NRPE* recipe in this chapter
*   The *Monitoring SSH for any host* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*