# Chapter 10. Security and Performance

In this chapter, we will cover the following recipes:

*   Requiring authentication for the web interface
*   Using authenticated contacts
*   Writing debugging information to the Nagios log file
*   Monitoring Nagios performance with Nagiostats
*   Improving startup times with pre-cached object files
*   Setting up a redundant monitoring host

# Introduction

Most administrators of even mid-size networks will choose to dedicate an entire server to monitoring software, and sometimes a whole server just for Nagios Core. This is because of two main factors common to most comprehensive Nagios Core setups:

*   They have a lot of privileges, because in order to inspect the running state of so many different hosts and services, they need to be conferred appropriate network access. This often means that their IP addresses are whitelisted all over the network. A user who is able to assume that privilege could potentially do a lot of damage.
*   They have a lot of work to do, and hence ideally have dedicated software and hardware resources to run what can be thousands of host and service checks smoothly and to promptly notice problems and recoveries. If a Nagios Core server is not able to keep up with its check schedule, it could cause delays in notifications about very important services.

It's therefore very important to take into account the security and performance of your Nagios Core server when building your configuration.

General best network security practices apply to Nagios Core management and will not be discussed in depth here. The following general guidelines are just as relevant to securing a Nagios Core server as they are any other kind of server:

*   **Don't run the server as root**: This is probably not a concern unless you've changed it yourself, as if you've installed using the Quick Start Guides the server should be set up to run as the unprivileged `nagios` user. Nagios Core should not need `root` privileges under most circumstances.
*   **Use a firewall**: The protections offered by Nagios Core, NSCA, and NRPE's host checking are very basic and are not a replacement for a software or hardware firewall policy. Even a simple `iptables` or `pf` software firewall would be a very good idea to protect a monitoring server.
*   **Use the principle of least privilege**: Don't confer Nagios Core or any of its plugins or processes more privileges than they need, and lock down writable files, such as `logs`, and state information only to appropriate users. Similarly, only allow Nagios Core access through firewalls for what it needs to check, and nothing else.
*   **Encrypt sensitive information**: Don't put credentials in plain text in any of the configuration files if you can avoid it, and if you can't avoid it, define them in resource files that are only readable by the `nagios` user.

You should therefore not consider this chapter a complete guide to securing and optimizing your Nagios Core server. For a more comprehensive treatment of security and optimization procedures, be sure to take a look at these pages in the Nagios Core documentation:

*   **Security Considerations**: [http://nagios.sourceforge.net/docs/3_0/security.html](http://nagios.sourceforge.net/docs/3_0/security.html)
*   **Enhanced CGI Security and** **Authentication**: [http://nagios.sourceforge.net/docs/3_0/cgisecurity.html](http://nagios.sourceforge.net/docs/3_0/cgisecurity.html)
*   **Tuning Nagios for Maximum** **Performance**: [http://nagios.sourceforge.net/docs/3_0/tuning.html](http://nagios.sourceforge.net/docs/3_0/tuning.html)

In particular, this chapter will focus on securing the CGIs, as the web interface is readily exposing Nagios Core's information to the world, and is particularly vulnerable to abuse if misconfigured. It will also include methods to assess how Nagios Core is performing.

# Requiring authentication for the web interface

In this recipe, we'll explore the use of basic authentication for the Nagios Core web interface, probably the single most important configuration step in preventing abuse of the software by malicious users.

By default, the Nagios Core installation process takes the sensible step of locking down the authentication by default in its recommended Apache configuration file, with standard HTTP authentication for a default user named `nagiosadmin`, with full privileges.

Unfortunately, some administrators take the step of removing this authentication or never installing it, in spite of the recommendations in the installation guide. It's a good idea to install it and keep it in place even on private networks, and especially if the server running Nagios Core is open to the Internet in any way (generally not advised).

This is not just because of the security benefits, but also because it allows you to set up basic access control, allowing certain users the permission to read state or run commands on certain resources, but not on others. It also has other more subtle benefits, such as recording the names of users that carry out actions for logging purposes.

## Getting ready

You'll need access to the backend of a Nagios Core server with version 3.0 or greater, to change its configuration and restart it. You'll also need a functioning web interface. Much of this recipe will assume the web interface is running on the recommended **Apache HTTPD server**; you may also need to be able to edit this configuration. Some familiarity with Apache HTTPD is assumed here; the documentation available online is excellent if you need to consult it:

[https://httpd.apache.org/docs/](https://httpd.apache.org/docs/)

This recipe will be in two parts: we'll first ensure that all the recommended settings are in place to properly require authentication, and we'll then demonstrate adding a user named `helpdesk` with read-only permissions for the server's web interface. This user will be able to read the states of all the hosts and services, but will not be able to (for example) issue commands, or submit passive check results.

## How to do it...

We can ensure proper authentication is in place for the Nagios Core web interface as follows:

1.  Clear your browser of all cookies and saved authentication data, and try to visit the web interface. If your browser does not challenge you for a username or password as follows, then it's likely that your authentication is disabled or not working correctly:![How to do it...](img/5566_10_01.jpg)
2.  On the server, move to the Nagios Core configuration directory and open the `cgi.cfg` file. In the default installation, this is saved in `/usr/local/nagios/etc/cgi.cfg`.

    ```
    # cd /usr/local/nagios/etc
    # vi cgi.cfg

    ```

3.  Ensure that the value for `use_authentication` is uncommented and set to `1`:

    ```
    use_authentication=1
    ```

4.  Ensure that the `default_user_name` directive is commented out:

    ```
    #default_user_name=guest

    ```

5.  In the `nagios.conf` file in your Apache configuration, check that the following lines are included to refer to the `htpasswd.users` file:

    ```
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /usr/local/nagios/etc/htpasswd.users
    Require valid-user
    ```

    This file should be somewhere in the configuration directory for Apache, for example `/etc/apache/conf.d/nagios.conf`. If you need to make changes to Apache's configuration to fix this, you will need to restart Apache to have the changes take effect.

We can add a read-only user named `helpdesk` as follows:

1.  Add the user to the `htpasswd.users` file using `htpasswd`, the Apache HTTPD password manager. Its location will vary depending on your system; common locations are `/usr/bin` and `/usr/local/apache/bin`.

    ```
    # htpasswd /usr/local/nagios/etc/htpasswd.users helpdesk
    New password:
    Re-type new password:
    Adding password for user helpdesk

    ```

    You may like to make the `htpasswd.users` file only readable by the web server user if you are concerned about the hashes being stolen by users on the system.

2.  In `cgi.cfg`, uncomment the `authorized_for_read_only` directive and add the new `helpdesk` user to its value:

    ```
    authorized_for_read_only=helpdesk
    ```

3.  Add the user to the values for the `authorized_for_all_services` and `authorized_for_all_hosts` directives:

    ```
    authorized_for_all_services=nagiosadmin,helpdesk
    authorized_for_all_hosts=nagiosadmin,helpdesk
    ```

    Be careful not to confuse these with the `authorized_for_all_service_commands` and `authorized_for_all_host_commands` directives.

You should not need to restart the Nagios Core server for these changes to take effect as you normally would with other `.cfg` files.

With this done, you should only be able to access the Nagios Core web interface with a valid username and password. The default `nagiosadmin` user created on installation should have full privileges, and the `helpdesk` user added in this recipe should be able to view host and service states, but will be unable to issue any commands such as rescheduling checks or submitting passive check results.

## How it works...

It's important to note that it isn't actually Nagios Core itself prompting for the username and password and running the authentication check; this is a function performed by the web server, as specified in the `nagios.conf` file recommended for installation with Apache HTTPD.

After login, however, Nagios Core uses the permissions defined in the `cgi.cfg` file each time one of the CGI scripts in the web interface is accessed, to ensure that the user as authenticated with the web server has permissions to view the requested page, or execute the requested action.

We disable the `default_user_name` directive by commenting it out, because this specifies a user that Nagios Core will "assume" for users who access the CGIs without authenticating. This is a potentially dangerous setting and is best avoided in most circumstances, particularly with a server with publicly accessible addresses.

The following directives in `cgi.cfg` allow refining permissions for using the CGIs, to form a simple kind of access control list:

*   `authorized_for_configuration_information`: The specified users are allowed to view configuration information for hosts in the web interface
*   `authorized_for_system_information`: The specified users are allowed to view Nagios Core process and performance information
*   `authorized_for_system_commands`: The specified users are allowed to run commands affecting the Nagios Core process, such as shutdowns and restarts
*   `authorized_for_all_services`: The specified users are allowed to view status information and history for all services
*   `authorized_for_all_hosts`: The specified users are allowed to view status information and history for all hosts
*   `authorized_for_all_service_commands`: The specified users are allowed to run commands on all services, such as rescheduling checks or submitting passive check results
*   `authorized_for_all_host_commands`: The specified users are allowed to run commands on all hosts, such as rescheduling checks or submitting passive check results

Further refining access per-service and per-host is done with authenticated contacts, demonstrated in the *Using authenticated contacts* recipe in this chapter. This is highly recommended for teams with mixed responsibilities that all require access to the same Nagios Core web interface.

## There's more...

Besides authentication via Apache HTTPD, it's also often sensible to limit the IP addresses allowed access to the Nagios Core instance, using `Order` and `Allow` Apache directives. We could extend the `nagios.conf` file loaded by Apache as follows:

```
<Directory "/usr/local/nagios/sbin">
    Options ExecCGI
    AllowOverride None
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /usr/local/nagios/etc/htpasswd.users
    Require valid-user
 Order Allow,Deny
 Deny from all
 Allow from 127.0.0.0/16
 Allow from 10.128.0.1
</Directory>
```

This would only allow local addresses and `10.128.0.1` to access the CGIs, denying access with `403 Forbidden` to anyone else. Similarly, we could arrange to only allow connections over HTTPS, perhaps with an `SSLRequireSSL` directive; in general, we can configure Apache to carefully control even accessing the CGIs, let alone abusing them.

Note that none of this should take the place of an appropriate firewall solution and policy. A monitoring server should be protected as carefully as any other mission-critical server.

## See also

*   The *Using authenticated contacts* recipe in this chapter
*   The *Viewing configuration in the web interface* and *Scheduling checks from the web interface* recipes in [Chapter 7](ch07.html "Chapter 7. Using the Web Interface"), *Working With the Web Interface*

# Using authenticated contacts

In this recipe, we'll learn how to use authenticated contacts to refine our control of access to information in the Nagios Core web interface. This recipe is useful in situations where a particular user requires information on the status of certain hosts and services but should not be allowed to view others, a setup which can't be managed with the directives in `cgi.cfg`.

As a simple example, on a given monitoring server, we might have two hosts configured thus:

```
define host {
    use        linux-server
    host_name  sparta.naginet
    alias      sparta
    address    10.128.0.21
    contacts   nagiosadmin
}
define host {
    use        linux-server
    host_name  athens.naginet
    alias      athens
    address    10.128.0.22
    contacts   nagiosadmin
}
```

We might like to add a new user `athensadmin` with permissions to view the status of and run commands on the `athens.naginet` host and its services, but not the `sparta.naginet` host or its services.

## Getting ready

You'll need access to the backend of a Nagios Core server with version 3.0 or greater, to change its configuration and restart it. You'll also need a functioning web interface with authentication running, and be familiar with how it works; the *Requiring authentication for the web interface* recipe in this chapter explains how to do this.

## How to do it...

We can add an authenticated contact as follows:

1.  Create a new user in the `htpasswd.users` file named `athensadmin`, using the `htpasswd` tool:

    ```
    # htpasswd /usr/local/nagios/etc/htpasswd.users athensadmin
    New password:
    Re-type new password:
    Adding password for user athensadmin

    ```

2.  Log in to the Nagios Core web interface with the credentials you just added, and click on **Hosts** in the left menu to verify that you are not able to view any information yet:![How to do it...](img/5566_10_02.jpg)
3.  Back on the command line, change to the Nagios Core objects configuration directory. In the quick start guide installation, this is `/usr/local/nagios/etc/objects`.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

4.  Edit the `contacts.cfg` file to include a definition for a new contact object, `athensadmin`. Here, we've used the `generic-contact` template. You can use your own template or values if you prefer; the value for `contact_name` is the important part:

    ```
    define contact {
        use           generic-contact
     contact_name  athensadmin
        alias         Athens Administrator
        email         athens@example.com
    }
    ```

5.  Edit the hosts or services to which you want to allow the `athensadmin` user access, and add `athensadmin` to its list of contacts. For our example, this means the definition for `athens.naginet` looks similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  athens.naginet
        alias      athens
        address    10.128.0.22
     contacts   nagiosadmin,athensadmin
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, logging in as the `athensadmin` user should allow you to view the details of the `athens.naginet` host, but nothing else:

![How to do it...](img/5566_10_03.jpg)

You should also be able to issue commands related to that host, such as rescheduling checks and acknowledging issues.

## How it works...

When logging in as an authenticated user with Apache HTTPD, Nagios Core checks the username to see if it matches the `contact_name` directive of any configured contacts. If it does, then privileges to inspect the state of that contact's associated hosts and services are granted, along with the ability to run commands on only those hosts and services. The web interface otherwise works in just the same way. To the authenticated contact, it appears as if no other hosts or services are being monitored.

If your network includes co-located equipment or teams with mixed monitoring responsibilities, this will allow you to restrict the Nagios Core interface to certain hosts for certain users. This can be very useful for confidentiality, transparency, and delegation purposes.

## There's more...

If you want to allow an authenticated user read-only access to the details for their associated hosts or services, you can arrange that by adding their username to the values of the `authorized_for_read_only` directive in `cgi.cfg`:

```
authorized_for_read_only=athensadmin
```

The `athensadmin` user will then still be able to view the same host and service information, but will not be able to issue any commands:

![There's more...](img/5566_10_04.jpg)

## See also

*   The *Requiring authentication for the web interface* recipe in this chapter
*   The *Creating a new contact* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Writing debugging information to a Nagios log file

In this recipe, we'll learn how to use the debugging log file in Nagios Core to get various kinds of process information from the running program, considerably more than available in the file specified by the standard `log_file` directive. This is useful not just for debugging purposes when Nagios Core is doing something unexpected at runtime, but also to get a better idea of how the server is working in general and with your particular configuration.

## Getting ready

You will need a Nagios Core server with version 3.0 or greater. More debugging options are available in versions after 3.0, but these will be noted in the recipe. You will need access to change the `nagios.cfg` file and to restart the server.

In this example, we'll simply log everything we possibly can, and then explain how to refine the logging behavior if necessary in the *How it works…* section.

## How to do it...

We can enable very verbose debugging for our monitoring server as follows:

1.  Change to the configuration directory for Nagios Core. In the default installation, this is `/usr/local/nagios/etc`. Edit the file `nagios.cfg`:

    ```
    # cd /usr/local/nagios/etc
    # vi nagios.cfg

    ```

2.  Look for the `debug_level`, `debug_verbosity`, and `debug_file` directives. Ensure they are uncommented or add them to the end of the file if they don't exist, and define them as follows:

    ```
    debug_level=-1
    debug_verbosity=2
    debug_file=/usr/local/nagios/var/nagios.debug
    ```

3.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the `/usr/local/nagios/var/nagios.debug` file should start filling with information about the running process quite quickly. You may find it instructive to watch it for a while with `tail -f`, which will show you the contents of the file as it updates:

```
# tail -f /usr/local/nagios/var/nagios.debug
[1347529967.839794] [008.2] [pid=14621] No events to execute at the moment. Idling for a bit...
[1347529967.839799] [001.0] [pid=14621] check_for_external_commands()
[1347529967.839805] [064.1] [pid=14621] Making callbacks (type 8)...
[1347529968.089971] [008.1] [pid=14621] ** Event Check Loop
[1347529968.090027] [008.1] [pid=14621] Next High Priority Event Time: Thu Sep 13 21:52:52 2012
[1347529968.090038] [008.1] [pid=14621] Next Low Priority Event Time:  Thu Sep 13 21:53:00 2012
...

```

## How it works...

The `debug_level` directive specifies how much information (and of what kind) should be written to the debugging log. Here we've used the value `-1`, which is a shortcut for specifying that all debugging information should be written to the debugging log file.

In practice, however, we often only want to get information about particular kinds of Nagios Core tasks. In this case, we can use `OR` values for `debug_level` to specify which ones.

The different kinds of debugging information can be specified with the following numbers:

*   `1`: Function enter and exit debugging
*   `2`: Configuration debugging
*   `4`: Process debugging
*   `8`: Scheduled event debugging
*   `16`: Host and service check debugging
*   `32`: Notification debugging
*   `64`: Event broker debugging

In version 3.3 and later of Nagios Core, the following can also be specified. There may be even more added in subsequent versions:

*   `128`: External commands debugging
*   `256`: Commands debugging
*   `512`: Scheduled downtime debugging
*   `1024`: Comments debugging
*   `2048`: Macros debugging

Rather than comma-separating these values to specify more than one, they are added together. For example, if we wanted to save process and scheduled event debugging information and nothing else, we would use `4 + 8 = 12`:

```
debug_level=12
```

We can turn the debugging off completely by changing `debug_level` back to `0`, its default value.

## There's more...

Nagios Core generates a great deal of information at the highest level of debugging, with over 30 lines per second even on minimal configurations, so be careful not to leave this running permanently if you don't always need it, as it can slowly fill a disk. You can avoid this situation by using the `max_debug_file_size` directive to specify a maximum size in bytes for the file. For example, to restrict the file to one megabyte, we could define the following:

```
max_debug_file_size=1000000
```

Nagios Core will "roll" an existing debugging log by adding the extension `.old` when it exceeds this size, and will start a new one. It will also automatically delete any previous logs with the `.old` extension when it does this.

## See also

*   The *Monitoring Nagios performance with Nagiostats* recipe in this chapter
*   The *Viewing and interpreting notification history* recipe in [Chapter 7](ch07.html "Chapter 7. Using the Web Interface"), *Working With the Web Interface*

# Monitoring Nagios performance with Nagiostats

In this recipe, we'll learn how to use the `nagiostats` utility to get some statistics about the performance of a Nagios Core process, and the states of the hosts and services that it monitors.

Optionally, we'll also show how to use the `mrtg.cfg` file built in the Nagios Core source distribution at `./configure` time to set up graphs built by `mrtg` (**Multi-Router Traffic Grapher**), and link to the graphs in the menu of the web interface. The Nagios Core source distribution includes some files to assist with this, which we'll use here.

## Getting ready

You will need a Nagios Core 3.0 or newer server installed and running to invoke `nagiostats`. Older versions do include the utility, but there is not quite as much information returned.

If you would like to run the `mrtg` graphing as well, which is highly recommended, you should have `mrtg` and its helper program, `indexmaker`, installed on your system. If you are already graphing other things with `mrtg`, don't worry, this recipe should not interfere with that.

The recipe does not assume any familiarity with `mrtg`, but if you have any problems with it, you may like to consult its documentation online at [http://oss.oetiker.ch/mrtg/doc/index.en.html](http://oss.oetiker.ch/mrtg/doc/index.en.html).

You should also have access to the sources from which your installation of Nagios Core was compiled. If you need to retrieve the sources again, you can download them from the Nagios Core website ([http://www.nagios.org/](http://www.nagios.org/)).

In this case, you will need to run `./configure` again to generate the file required, `sample-config/mrtg.cfg`.

## How to do it...

We can invoke `nagiostats` itself in one step whenever we want to get some statistics about the server's performance:

1.  Run the following command:

    ```
    # /usr/local/nagios/bin/nagiostats -c /usr/local/nagios/etc/nagios.cfg

    ```

    This should give you output beginning with the following:

    ```
    Nagios Stats 3.4.1
    Copyright (c) 2003-2008 Ethan Galstad (www.nagios.org)
    Last Modified: 05-11-2012
    License: GPL
    CURRENT STATUS DATA
    ------------------------------------------------------
    Status File:                            /usr/local/nagios/var/status.dat
    Status File Age:                        0d 0h 0m 2s
    Status File Version:                    3.4.1
    Program Running Time:                   2d 2h 25m 23s
    Nagios PID:                             2487
    Used/High/Total Command Buffers:        0 / 0 / 4096
    Total Services:                         60
    Services Checked:                       60
    ...

    ```

2.  Run the following command:

    ```
    # /usr/local/nagios/bin/nagiostats -c /usr/local/nagios/etc/nagios.cfg --help

    ```

    This should give you a complete list of the names and meanings of all the fields returned by the output of `nagiostats`.

If you would like to include `mrtg` graphs of this data, a good starting point is using the sample configuration included in the Nagios Core source distribution, in `sample-config/mrtg.cfg`.

1.  Copy `sample-config/mrtg.cfg` into `/usr/local/nagios/etc`:

    ```
    # cp nagios-3.4.1/sample-config/mrtg.cfg /usr/local/nagios/etc

    ```

2.  Create a directory to store the `mrtg` pages and graphs, so that they can be viewed in the Nagios Core web interface:

    ```
    # mkdir /usr/local/nagios/share/stats

    ```

3.  Edit `/usr/local/nagios/etc/mrtg.cfg` to include a `WorkDir` declaration at the top of the file:

    ```
    WorkDir: /usr/local/nagios/share/stats
    ```

4.  Run `mrtg` to create the graphs:

    ```
    # mrtg /usr/local/nagios/etc/mrtg.cfg

    ```

    For this first run, we can safely ignore any errors about missing prior data, or backup log files:

    ```
    2012-09-16 17:01:04, Rateup WARNING: /usr/bin/rateup could not read the primary log file for nagios-n
    2012-09-16 17:01:04, Rateup WARNING: /usr/bin/rateup The backup log file for nagios-n was invalid as well
    2012-09-16 17:01:04, Rateup WARNING: /usr/bin/rateup Can't remove nagios-n.old updating log file
    2012-09-16 17:01:04, Rateup WARNING: /usr/bin/rateup Can't rename nagios-n.log to nagios-n.old updating log file

    ```

    If you are using a UTF-8 locale in your shell, `mrtg` may fail to run. You can run it in a standard C locale with an `env` prefix:

    ```
    # env LANG=C mrtg /usr/local/nagios/etc/mrtg.cfg

    ```

5.  Run the `indexmaker` helper installed with `mrtg` to create an index to the graphs:

    ```
    # indexmaker /usr/local/nagios/etc/mrtg.cfg --output=/usr/local/nagios/share/stats/index.html

    ```

    You should only need to do this once, unless you add or remove graph definitions from `mrtg.cfg` later on.

6.  Visit `http://olympus.naginet/nagios/stats`, substituting your own Nagios Core server's hostname for `olympus.naginet`. After authenticating (if necessary), we should be able to see some empty `mrtg` graphs:![How to do it...](img/5566_10_05_full.jpg)

    Don't worry that they're empty; we expect that, as we only have one data point for each graph at the moment.

7.  If everything has worked up to this point, we can probably add a `cron` task to run every five minutes to add new data points to the graph. Here, we are assuming the `mrtg` program is saved in `/usr/bin`:

    ```
    */5 * * * *  root  /usr/bin/mrtg /usr/local/nagios/etc/mrtg.cfg
    ```

    The best way to do this will vary between systems. You could put this in `/etc/crontab`, or in its own file in `/etc/cron.d/nagiostats` if you want to be a little tidier.

    It is probably safe to leave it running as `root`, but if you are concerned about this, you should also be able to run it as the `nagios` user by including a `--lock-file` option:

    ```
    */5 * * * *  nagios  /usr/bin/mrtg --lock-file=/usr/local/nagios/var/mrtg.cfg.lock /usr/local/nagios/etc/mrtg.cfg
    ```

    That might require correcting permissions on the graphs already generated:

    ```
    # chown -R nagios.nagios /usr/local/nagios/share/stats

    ```

With this done, if the `cron` task is installed correctly, we should start seeing data being plotted over the next few hours:

![How to do it...](img/5566_10_06_full.jpg)

## How it works...

The statistics provided by `nagiostats` provide both performance data about Nagios Core itself, how long it's taking to complete its round of checks of all objects and the average time it's taking per check, as well as data such as the number of hosts in various states. By default, running it will return the data in a terse but human-readable format; you can get a good idea of the meaning of each of the fields by running it with `--help` as suggested in the recipe.

The `mrtg.cfg` file included in the Nagios source distribution, which is tailored to your particular system at `./configure` time, contains example definitions of `mrtg` graphs that parse the data retrieved from `nagiostats`. These are not the only possible graphs using the information provided by `nagiostats`, but they are useful examples.

The data used is the same as the data you read if you invoked `nagiostats` from the shell, but the format is slightly different. If you want to see the data being passed to `mrtg` by `nagiostats`, you can run it with the `--mrtg` option and nominate fields to be included in the output with `--data`; for example:

```
# /usr/local/nagios/bin/nagiostats -c /usr/local/nagios/etc/nagios.cfg --mrtg --data=AVGACTSVCPSC,AVGPSVSVCPSC,PROGRUNTIME,NAGIOSVERPID
0
0
1d 2h 43m 43s
Nagios 3.4.1 (pid=1080)

```

The recipe's call to `indexmaker` is a separate program that builds an `index.html` file with links to all the graphs, just for convenience. Like the `mrtg` call, it refers to the configuration file `/usr/local/nagios/etc/mrtg.cfg` to figure out what it needs to do.

## There's more...

Once you're happy with the way your graph's web pages are being displayed, you might like to consider including them in your Nagios Core sidebar. This can be done by editing `/usr/local/nagios/share/side.php`, and adding a new link to the **System** section, perhaps named **Performance Reports**, below the **Performance Info** link. The new line might look similar to the following code snippet:

```
<li><a href="/nagios/stats/" target="<?php echo $link_target;?>">Performance Reports</a></li>
```

This would make a link to the graphs show up in the web interface as follows:

![There's more...](img/5566_10_07.jpg)

Note that customizations to the menu like this will be overwritten if you reinstall Nagios Core.

If you like what `mrtg` does with this data, you might like to look at **Cacti**, a very helpful frontend to `rrdtool`, which is similar to `mrtg`. It will allow you a lot of flexibility in defining graphs, although it takes a while to learn ([http://www.cacti.net/](http://www.cacti.net/)).

If you're interested in more graphing for Nagios Core performance and state data you may like the Nagiosgraph extension, which is discussed in the *Tracking host and service states with NagiosGraph* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*.

Finally, note that Nagios Core includes some built-in graphing of hosts and states in its reports, so be sure to check those out as well before you try to build a graph for a report that already exists! These are all also discussed in [Chapter 7](ch07.html "Chapter 7. Using the Web Interface"), *Working With the Web Interface*. Check out the references in the *See also* section for this recipe.

## See also

*   The *Using the tactical overview*, *Viewing and interpreting availability reports*, *Viewing and interpreting trends*, and *Viewing and interpreting notification history* recipes in [Chapter 7](ch07.html "Chapter 7. Using the Web Interface"), *Working With the Web Interface*
*   The *Tracking host and service states with Nagiosgraph* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*

# Improving startup times with pre-cached object files

In this recipe, we'll learn how to shorten startup times for large and/or complex Nagios Core configurations. This is done by pre-caching the Nagios Core objects from the configuration, applying all appropriate template and group expansions into a single file that Nagios Core can read much more quickly than a more modular and human-readable configuration.

This will likely only be of interest to you if you are monitoring more than a hundred hosts or services with a reasonably complex template and grouping layout, as suggested by some of the recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts") and [Chapter 9](ch09.html "Chapter 9. Managing Configuration"). It will still work on smaller installations, but the gains in startup speed are likely to be minimal.

If you are only running a small setup, then this recipe might be of interest if you want to better understand how Nagios Core expands a configuration that uses a lot of templates and other configuration tricks.

## Getting ready

You should be running a Nagios Core 3.0 or newer server, and have access to the server to change its configuration.

You should check that the `precached_object_file` directive in `/usr/local/nagios/etc/nagios.cfg` is uncommented and defined to an accessible file. The setting in the Quick Start configuration is sensible:

```
precached_object_file=/usr/local/nagios/var/objects.precache
```

Having this directive uncommented doesn't actually generate or use the pre-cached objects file; that needs to be done explicitly, as will be explained in the recipe.

Don't confuse this with the `object_cache_file` directive in the same file, which should be left untouched at its current setting.

For this example, we'll use a fairly large configuration with some 20,000 objects defined on a rather slow machine.

## How to do it...

We can get some idea of possible performance improvement from using pre-cached object files as follows:

1.  Run `nagios` with the `-s` option, and inspect the output. This will print a profile of the processes normally involved in building complete object definitions from the configuration files.

    ```
    # /usr/local/nagios/bin/nagios -s /usr/local/nagios/etc/nagios.cfg

    ```

    In this case, we're particularly interested in the sections of output marked with asterisks, in the section titled `OBJECT CONFIG PROCESSING TIMES`, which denote times that could be improved by pre-cached object files, including an estimate of the saved time at the end of the `TOTAL` field:

    ```
    OBJECT CONFIG PROCESSING TIMES      (* = Potential for precache savings with -u option)
    ----------------------------------
    Read:                 8.285613 sec
    Resolve:              0.001696 sec  *
    Recomb Contactgroups: 0.000124 sec  *
    Recomb Hostgroups:    1.972412 sec  *
    Dup Services:         0.309333 sec  *
    Recomb Servicegroups: 0.000029 sec  *
    Duplicate:            0.000168 sec  *
    Inherit:              0.102685 sec  *
    Recomb Contacts:      0.000001 sec  *
    Sort:                 0.000000 sec  *
    Register:             0.826046 sec
    Free:                 0.102805 sec
     ============
    TOTAL:                11.600912 sec  * = 2.386448 sec (20.57%) estimated savings

    ```

    We might decide that `2.3` seconds is a worthwhile saving from our restart time. Perhaps we really don't want to miss any important checks!

2.  Run `nagios` with the `-p` and `-v` options, to verify the configuration and also write a pre-cached object file:

    ```
    # /usr/local/nagios/bin/nagios -pv /usr/local/nagios/etc/nagios.cfg

    ```

3.  Run `nagios` with the `-u` and `-s` options to see how long the startup and scheduling test takes when instructed to use the pre-cached object file:

    ```
    # /usr/local/nagios/bin/nagios -us /usr/local/nagios/etc/nagios.cfg

    ```

4.  We may note that the `TOTAL` time taken is now significantly less (even more of an improvement than estimated), and that many of the times are now zero seconds, as Nagios Core did not have to do that step at all:

    ```
    OBJECT CONFIG PROCESSING TIMES      (* = Potential for precache savings with -u option)
    ----------------------------------
    Read:                 4.975257 sec
    Resolve:              0.000000 sec  *
    Recomb Contactgroups: 0.000000 sec  *
    Recomb Hostgroups:    0.000000 sec  *
    Dup Services:         0.000000 sec  *
    Recomb Servicegroups: 0.000000 sec  *
    Duplicate:            0.000000 sec  *
    Inherit:              0.000000 sec  *
    Recomb Contacts:      0.000000 sec  *
    Sort:                 0.000000 sec  *
    Register:             0.828953 sec
    Free:                 0.000758 sec
     ============
    TOTAL:                5.804968 sec

    ```

With this done, we should find that restarting Nagios Core with both the `-d` and the `-u` flags should be faster than before. This can be incorporated into any startup scripts (for example, `/etc/init.d/nagios`). It also means that if we make changes to the configuration, we must remember to run `nagios -pv` again to validate and regenerate the pre-cached object file.

## How it works...

When run with the `-p` option, Nagios Core parses the configuration as normal into objects with which it can work, expanding out the `hostgroups`, `templates`, and other configuration shortcuts. It writes this information into a single file as specified by the `precached_object_file` directive.

The configuration file is human-readable; if you view it in a text editor, you'll see the expanded definitions that have been built from your configuration, with all of the object inheritance, regular expression hostgroups, and multi-host service definitions expanded.

When next restarted, Nagios Core can be instructed to use this file instead of re-parsing the configuration all over again by including the `-u` option. You may need to incorporate this in any `init.d` scripts you are using.

## There's more...

If you don't really have speed problems on Nagios Core restarting, it's best to avoid making this permanent, as it adds another layer of complexity to building configuration; if you forget to rebuild the pre-cached object file after making configuration changes and before restarting Nagios Core, it will continue using the previous configuration, not realizing the difference. Use this with caution!

## See also

*   The *Running a service on all hosts in a group* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Configuring host roles using groups* and the *Using inheritance to simplify configuration* recipes in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Configuration Management*

# Setting up a redundant monitoring host

In this recipe, we'll learn how to implement a simple kind of redundancy for Nagios Core, by running a second Nagios Core instance with a near-identical configuration on another machine.

This may seem like it would not need a recipe to implement. It should be reasonably straightforward to simply copy over the configuration for a Nagios Core system and run it concurrently. There are two main problems with this:

*   Every problem detected on the network will fire notifications events twice. The administrator charged with looking after the pager might well find this unbearable!
*   Everything will be checked twice. On smaller networks with simple checks, this may not be too much of a concern, but it could be an issue on larger, busier networks.

This recipe will solve the first problem by configuring the slave monitoring server to suppress notifications until it detects an issue with the master server. In the *There's more…* section, we'll discuss extending this solution to solve the second problem as well, by preventing the slave server from making checks as well as sending notifications while the master server is active.

## Getting ready

This is the most complex recipe in this book, and one of the longest, tying in concepts from many other recipes and chapters. To follow it, you will likely need to have a good working knowledge of the following:

*   The building blocks of Nagios Core ­hosts, services, contacts, commands, plugins, and notifications ­explained in all recipes in Chapters 1 through 4.
*   Remote execution via `check_nrpe` ­explained in all recipes in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*. The recipe will at one point tell you to install NRPE on the master server to run a specific plugin, so you should learn how to do this first.
*   Event handlers and writing to the command file with them, ­explained in the *Setting up an event handler script* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core").

The event handler scripts, the most complex part of this setup, are fortunately already written for us; we'll show how to implement them by copying them out of the Nagios Core source package. You'll therefore need to have the sources available for your particular version of Nagios Core. If you need to retrieve the sources again, you can download them again from Nagios Core's website at [http://www.nagios.org/](http://www.nagios.org/).

The recipe will start by assuming we have two monitoring servers: `olympus.naginet` (`10.128.0.11`), which will be the master monitoring server, and `everest.naginet` (`10.128.0.12`), which will be the slave. The two servers are configured to monitor the same three hosts, with PING service checks:

*   `sparta.naginet`
*   `athens.naginet`
*   `ithaca.naginet`

The Nagios Core configuration of the two servers is completely identical to start with, and both are sending notifications to an appropriate contact group. However, note that the servers are not yet monitoring one another; this will be an important part of the recipe.

## How to do it...

We can arrange a simple redundancy setup for our two Nagios Core servers as follows:

1.  Confirm that the `check_nagios` plugin is available on the `master` server, and try running it:

    ```
    # cd /usr/local/nagios/libexec
    # ./check_nagios -e 5 -F /usr/local/nagios/var/status.dat -C /usr/local/nagios/bin/nagios
    NAGIOS OK: 1 process, status log updated 3 seconds ago

    ```

2.  Install the NRPE daemon on the master server, and define a command `check_nagios` in `nrpe.cfg` (see [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution")).

    ```
    command[check_nagios]=/usr/local/nagios/libexec/check_nagios -e 5 -F /usr/local/nagios/var/status.dat -C /usr/local/nagios/bin/nagios

    ```

3.  Include the slave server's address in the `allowed_hosts` directive for `nrpe`, in the file `/usr/local/nagios/etc/nrpe.cfg`:

    ```
    allowed_hosts=127.0.0.1,10.128.0.12
    ```

    Don't forget to restart NRPE to include this change to the configuration.

4.  On the slave server, verify that a call to `check_nrpe` can retrieve the results of `check_nagios` on the master server:

    ```
    # cd /usr/local/nagios/libexec
    # ./check_nrpe -H olympus.naginet
    NRPE v2.13
    # ./check_nrpe -H olympus.naginet -c check_nagios
    NAGIOS OK: 1 process, status log updated 2 seconds ago

    ```

    You will have to install the `check_nrpe` plugin on the slave server to do this. This is explained in the *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution").

5.  On the slave server, copy four files (two event handlers and two helper scripts) from the source distribution into the `/usr/local/nagios/libexec/eventhandlers` directory (which you may need to create first):

    ```
    # EHD=/usr/local/nagios/libexec/eventhandlers
    # mkdir -p $EHD
    # cd /usr/local/src/nagios
    # cp contrib/eventhandlers/enable_notifications $EHD
    # cp contrib/eventhandlers/disable_notifications $EHD
    # cp contrib/eventhandlers/redundancy-scenario1/handle-master-host-event $EHD
    # cp contrib/eventhandlers/redundancy-scenario1/handle-master-proc-event $EHD

    ```

    The preceding command assumes you are keeping the sources for your Nagios Core distribution in `/usr/local/src`. We define and use the shell variable `$EHD` to refer to the event handlers directory for convenience.

6.  In the installed `handle-master-proc-event` script, find and replace `active_service_checks` with `notifications`. The command line tool `sed` works well for this:

    ```
    # sed -i 's/active_service_checks/notifications/g' $EHD/handle-master-proc-event

    ```

    This is because the script as provided issues a command to toggle active checks, rather than notifications. At the time of writing, in Nagios 3.3.1 there is also a bug in `handle-master-proc-event` that may need to be corrected, on line 49:

    ```
    `eventhandlerdir/disable_active_service_checks`
    ```

    It should have a dollar sign added after the first backtick:

    ```
    `$eventhandlerdir/disable_active_service_checks`
    ```

7.  Ensure the event handlers are owned and executable by the `nagios` user:

    ```
    # chown nagios.nagios $EHD/*
    # chmod 0755 $EHD/*

    ```

8.  In `/usr/local/nagios/etc/objects/commands.cfg`, define two new event handler commands:

    ```
    define command {
        command_name  handle-master-host-event
        command_line  $USER1$/eventhandlers/handle-master-host-event $HOSTSTATE$ $HOSTSTATETYPE$ $HOSTATTEMPT$
    }
    define command {
        command_name  handle-master-proc-event
        command_line  $USER1$/eventhandlers/handle-master-proc-event $SERVICESTATE$ $SERVICESTATETYPE$ $SERVICEATTEMPT$
    }
    ```

9.  Make a host and service definition on the slave server to monitor the master server. It might look something similar to the following code snippet; change your `host_name`, `alias`, and `address` values as appropriate. The templates used are only examples; you will probably want to choose templates that are defined to run checks often, and during a 24x7 interval.

    ```
    define host {
        use            critical-host-template
        host_name      olympus.naginet
        alias          olympus
        address        10.128.0.11
        event_handler  handle-master-host-event
    }
        define service {
            use                  critical-service-template
            host_name            olympus.naginet
            service_description  NAGIOS
            check_command        check_nrpe!check_nagios
            event_handler        handle-master-proc-event
        }
    ```

    You can make the master server monitor the slave server as well if you wish.

10.  Note that you will need to have the `check_nrpe` command defined, which the *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution") explains. If you have followed that recipe then you have probably already done this. If not, the following definition works:

    ```
    define command {
        command_name  check_nrpe
        command_line  $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
    }
    ```

11.  Finally, in `nagios.cfg` on the slave server, change `enable_notifications` to `0`:

    ```
    enable_notifications=0
    ```

12.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the two Nagios Core servers should both be running, but importantly, notifications on the slave server start out as disabled, as visible in the **Tactical Overview**:

![How to do it...](img/5566_10_10.jpg)

However, all the systems are still being monitored, as visible in the **Services** screen, including the **NAGIOS** service on the host machine:

![How to do it...](img/5566_10_11.jpg)

This means that notifications will only be sent by the master server, since it still has its notifications enabled. However, if the master server goes down or its Nagios process stops working, the event handlers should be called, and notifications on the slave server will be automatically enabled. When the master server or its `NAGIOS` service comes back up, the notifications will be disabled again, with checks and state changes having continued uninterrupted throughout. We have therefore established a simple kind of redundancy. If you use this setup, you should test it thoroughly to make sure that the slave Nagios Core server will enable and disable its notifications for each contingency (host goes down, service goes down, service comes back, and so on.)

## How it works...

The event handlers included in the Nagios Core distribution, which we copied into the `eventhandlers` directory, are designed to handle toggling notifications and active checks based on the status of a given service or host. They are included for the purposes of demonstrating event handlers and redundancy situations like this one.

We start by setting up the slave server to monitor not just the host on which the master Nagios Core server is running, but also the Nagios Core service itself, using the `check_nagios` plugin. This plugin checks the age of the log file and the system's process tables to ensure that a Nagios Core service is actually running on the system. Because it's a local plugin that doesn't work for remote checks, we check it from the slave server via NRPE.

The slave server checks the status of the master server and its `NAGIOS` service as part of its normal routine of active checks. When the master server's host or its `NAGIOS` service change state, both call their respective event handlers, the two shell scripts `handle-master-host-event` and `handle-master-proc-event`, defined in the commands of the same name.

Each time the event handlers are called, they are passed three arguments, in macro form. For `handle-master-host-event`, these are:

*   `$HOSTSTATE$`: This is the new state of the master server
*   `$HOSTSTATETYPE$`: This specifies whether the state is `SOFT` or `HARD`
*   `$HOSTATTEMPT$`: This is the number of host checks attempted, up to the value of `max_check_attempts` for the host

`handle-master-proc-event` is passed three analogous arguments, the only difference being they refer to service states rather than host states:

*   `$SERVICESTATE$`: This is the new state of the `NAGIOS` service on the master server
*   `$SERVICESTATETYPE$`: This specifies whether the service state is `SOFT` or `HARD`
*   `$SERVICEATTEMPT$`: This is the number of host checks attempted, up to the value of `max_check_attempts` for the host

The event handlers are written in such a way that they only do anything if the new state is `HARD`; that is, if the number of `max_check_attempts` has been reached. It ignores `SOFT` state changes until enough consecutive checks have failed that it can be reasonably confident in concluding that the monitored host or service is suffering a problem.

If the host or service enters a `HARD CRITICAL` state, the event handlers call the helper script `enable-notifications` to write a command to the `commands` file at `/usr/local/nagios/var/rw/nagios.cmd` for the server to process. This command takes the following form, including the UNIX timestamp for when the command was written:

```
[1348129155] ENABLE_NOTIFICATIONS;1348129155

```

When Nagios Core processes this command, the effect is that the previously disabled notifications are enabled, and all subsequent notifications generated as a result of checks will be sent.

Similarly, when the host or service recovers from the `HARD CRITICAL` state, entering a `HARD UP` or `HARD OK` state, the `disable-notifications` helper script is called, writing a command in the same manner:

```
[1348129231] DISABLE_NOTIFICATIONS;1348129231

```

The effect is that when the master server is noted to be down, the slave server notices and assumes its notification behavior, and when it recovers, it stops its own notifications again, allowing the master server to resume its role.

## There's more...

If network bandwidth is a concern, we can arrange to leave the slave server more or less idle when not in use, by keeping not only notifications but also service checks off by default. Helper scripts for this are also included in the Nagios Core distribution, in the `disable_active_service_checks` and `enable_active_service_checks` scripts.

The primary issue with this change is the loss of state information as the slave server makes its initial round of checks; this can also be worked around, as explained in the Nagios Core documentation on redundancy at [http://nagios.sourceforge.net/docs/3_0/redundancy.html](http://nagios.sourceforge.net/docs/3_0/redundancy.html).

Once these steps are implemented, the main annoyance with this setup is having to keep the two configuration directories in sync. It's undesirable and error-prone to have to make changes on two servers each time the configuration needs to change, so you may like to consider using a snapshot tool such as `rsync` to keep the two directories the same. More information about `rsync` can be found at [http://en.wikipedia.org/wiki/Rsync](http://en.wikipedia.org/wiki/Rsync).

A configuration managed with version control is also help here, as recommended in the *Keeping configuration under version control* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"). That way you can use `git clone` or `svn checkout` to quickly copy and update configuration files on multiple machines.

## See also

*   The *Writing debugging information to Nagios log file* recipe in this chapter
*   The *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*
*   The *Keeping configuration under version control* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Configuration Management*
*   The *Setting up an event handler script* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*