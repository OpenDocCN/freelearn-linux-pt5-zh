# Chapter 5. Monitoring Methods

In this chapter, we will cover the following recipes:

*   Monitoring PING for any host
*   Monitoring SSH for any host
*   Checking an alternative SSH port
*   Monitoring mail services
*   Monitoring web services
*   Checking that a website returns a given string
*   Monitoring database services
*   Monitoring the output of an SNMP query
*   Monitoring a RAID or other hardware device
*   Creating an SNMP OID to monitor

# Introduction

Nagios Core is best thought of as a monitoring framework that uses plugins to perform appropriate checks on hosts and services, and returns results about their states in a format that it understands and can use for sending notifications and keeping track of states on a long-term basis.

The design is quite flexible. As explained in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*, Nagios Core can use as a plugin any command-line application that gives appropriate return values as defined in the Nagios Core header files, Perl library, or shell script. In turn, Nagios Core can be configured to use the same plugin in many different ways, taking advantage of any switch provided by the plugin to adjust its behavior, including providing metadata to it in the form of the values of Nagios Core macros, such as `$HOSTADDRESS$`.

The collection of plugins available on the Nagios Exchange website at [http://exchange.nagios.org/](http://exchange.nagios.org/) is fairly large, and documenting all of them is well out of the scope of this book. However, some of the most useful plugins are included as part of the Nagios Plugins set, and are installed as part of the recommended quick start guides for Nagios Core at [http://nagios.sourceforge.net/docs/3_0/quickstart.html](http://nagios.sourceforge.net/docs/3_0/quickstart.html). They include programs to monitor very common network features in typical ways, such as monitoring basic network connectivity, web services, mail servers, and many others. The plugin site itself is at [http://nagiosplugins.org/](http://nagiosplugins.org/).

This chapter will demonstrate the usage of some of the most useful components of this plugin set, which it assumes you have already installed. The focus will be on monitoring tasks that will be relevant to most or even all networks of various sizes, hopefully bringing the reader well past the point of thinking of Nagios Core as merely a process to send PING requests. The last few recipes will show how you can use the **Simple Network Management Protocol** (**SNMP**) as a method for checking any generic network service or system property that may not be covered by the standard Nagios Plugins set.

# Monitoring PING for any host

In this recipe, we'll learn how to set up PING monitoring for a host. We'll use the `check_ping` plugin, and its command of the same name, to send `ICMP` `ECHO` requests to a host. We'll use this as a simple diagnostic check to make sure that the host's network stack is responding in a consistent and timely fashion, in much the same way as an administrator might use the `ping` command interactively to check the same properties.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `corinth.naginet`, a host defined in its own file. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

## How to do it...

We can add a new PING service check to our existing host as follows:

1.  Change to the objects configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        windows-server
        host_name  corinth.naginet
        alias      corinth
        address    10.128.0.27
    }
    ```

3.  Beneath the definition for the host, place a service definition referring to `check_ping`. You may like to use the `generic-service` template, as follows:

    ```
    define service {
        use                  generic-service
        host_name            corinth.naginet
        service_description  PING
        check_command        check_ping!100,20%!200,40%
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service check will start taking place, with the appropriate contacts and contact groups notified when:

*   The **Round Trip Time** (**RTT**) of the request and its response exceeds 200ms, or more than 40 percent of the packets are lost during the check; a `CRITICAL` notification is fired for the service in either case.
*   If a `CRITICAL` notification was not fired, and the RTT of the request and its response exceeds 100ms, or more than 20 percent of the packets are lost during the check; in this case, a `WARNING` notification is fired for the service.

The information about the thresholds is given in the definition for the `check_command` directive, as arguments to the `check_ping` command.

More information about this service will also be visible in the web interface, under the **Services** section.

## How it works...

The configuration added in the preceding section defines a new service check on the existing `corinth.naginet` host to check that the RTT and the packet loss for an `ICMP` `ECHO` request and response are within acceptable limits.

For most network configurations, it may well be the case that the host itself is also being checked by `check_ping`, by way of the command `check-host-alive`. The difference is that the thresholds for the RTT and packet loss are intentionally set very high for this command, because it is intended to establish whether the host is up or down at all, not how responsive it is.

## There's more...

In networks where most of the hosts are configured to respond to `ICMP` `ECHO` requests, it could perhaps be worthwhile to configure service checks on all of the hosts in a configuration. This can be done using the `*` wildcard when defining `host_name` for the service:

```
define service {
    use                  generic-service
    host_name            *
    service_description  PING
    check_command        check_ping!100,20%!200,40%
}
```

This will apply the same check, with a `service_description` directive of `PING`, to all of the hosts configured in the database. This method will save the hassle of configuring a service separately for all the hosts.

If some of the hosts in a network do not respond to PING, it may be more appropriate to place the ones that do in a hostgroup, perhaps named something such as `icmp`:

```
define hostgroup {
    hostgroup_name  icmp
    alias           ICMP enabled hosts
    members         sparta.naginet,corinth.naginet
}
```

The single service can then be applied to all the hosts in that group, using the `hostgroup_name` directive in the `service` definition:

```
define service {
    use                  generic-service
    hostgroup_name       icmp
    service_description  PING
    check_command        check_ping!100,20%!200,40%
}
```

It's generally a good idea to have network hosts respond to ICMP messages wherever possible, in order to comply with the recommendations in RFC1122 and to ease debugging.

Finally, note that the thresholds for the RTT and the packet loss are not fixed; in fact, they're defined in the service definition, in the `check_command` line. For hosts that have higher latency, perhaps due to network load or topology, it may be appropriate to adjust these thresholds, which is covered in the *Changing thresholds for ping RTT and packet loss* recipe in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*.

## See also

*   The *Creating a new host*, *Creating a new service*, and *Creating a new hostgroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Using an alternative check command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Changing thresholds for ping RTT and packet loss* recipe in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*

# Monitoring SSH for any host

In this recipe, we'll learn how to check that the SSH daemon on a remote host responds to requests, using the `check_ssh` plugin, and the command of the same name. This will allow us to be notified as soon as there are problems connecting to the SSH service.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `troy.naginet`, a host defined in its own file. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

It may be a good idea to first verify that the host for which you want to add monitoring is presently running the SSH service that requires checking. This can be done by running the `ssh` client to make a connection to the host:

```
$ ssh troy.naginet

```

We should also check that the plugin itself will return the result required when run against the applicable host, as the `nagios` user:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_ssh troy.naginet

```

If you're unable to get a positive response from the SSH service on the target machine, even if you're sure it's running, then this could perhaps be a symptom of unrelated connectivity or filtering problems. We may, for example, need to add the monitoring server on which Nagios Core is running to the whitelist for SSH (normally TCP destination port `22`) on any applicable firewalls or routers.

## How to do it...

We can add a new SSH service check to our existing host as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  troy.naginet
        alias      troy
        address    10.128.0.25
    }
    ```

3.  Beneath the definition for the host, place a service definition referring to `check_ssh`. It may help to use the `generic-service` template or another suitable template, as follows:

    ```
    define service {
        use                  generic-service
        host_name            troy.naginet
        service_description  SSH
        check_command        check_ssh
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service check will start taking place, with the appropriate contacts and contact groups notified when an attempt to connect to the SSH server fails. The service check will be visible in the web interface, on the **Services** page.

## How it works...

The preceding configuration defines a new service with a `service_description` of `SSH` for the existing `troy.naginet` host, using the values in the `generic-service` template and additionally defining a `check_command` directive of `check_ssh`.

This means that in addition to checking whether the host itself is up with `check-host-alive`, as done previously, Nagios Core will also check that the SSH service running on the host is working by attempting to make a connection with it. It will also notify the applicable contacts if there are any problems found with the service after the appropriate number of tests.

For example, if the plugin finds that the host is accessible but not responding to client tests, then it might notify with the following text:

```
Subject: ** PROBLEM Service Alert: troy.naginet/SSH is CRITICAL **

***** Nagios *****

Notification Type: PROBLEM

Service: SSH
Host: troy.naginet
Address: troy.naginet
State: CRITICAL

Date/Time: Wed May 23 13:35:21 NZST 2012

Additional Info:

CRITICAL - Socket timeout after 10 seconds

```

Note that we don't need to actually supply credentials for the SSH check; the plugin simply ensures that the service is running and responding to connection attempts.

## There's more...

The definition for the `check_ssh` command warrants some inspection if we're curious as to how the plugin is actually applied as a command, as defined in the QuickStart configuration in `/usr/local/nagios/etc/objects/commands.cfg`:

```
define command {
    command_name  check_ssh
    command_line  $USER1$/check_ssh $ARG1$ $HOSTADDRESS$
}
```

This shows that the `check_ssh` command is configured to run the `check_ssh` binary file in `$USER1$`, a macro that normally expands to `/usr/local/nagios/libexec`, against the host address of the applicable server. It adds in any other arguments beforehand. We haven't used any arguments in this recipe, since we simply want to make a normal check of the SSH service on its default port.

This check should work with most SSH2 compliant servers, most notably including the popular **OpenSSH** server.

Checking SSH accessibility is a common enough thing for servers that you may wish to set up an SSH service check to apply to a hostgroup, rather than merely to an individual host. For example, if you had a group called `ssh-servers` containing several servers that should be checked with a `check_ssh` call, then you could configure them all to be checked with one service definition using the `hostgroup_name` directive:

```
define service {
    use                  generic-service
    hostgroup_name       ssh-servers
    service_description  SSH
    check_command        check_ssh
}
```

This would apply the same service check to each host in the group, which makes the definition easier to update if the check needs to be changed or removed in future.

Note that the `check_ssh` plugin is different from the `check_by_ssh` plugin, which is used to run checks on remote machines, much like NRPE.

## See also

*   The *Checking an alternative SSH port* recipe in this chapter
*   The *Creating a new host*, *Creating a new service*, and *Creating a new hostgroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Using check_by_ssh with key authentication instead of NRPE* and *Monitoring local services on a remote machine with NRPE* recipes in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*

# Checking an alternative SSH port

In this recipe, we'll learn how to deal with the common situation of a machine running an SSH daemon that is listening on an alternative port. So, a service definition that uses `check_ssh`, as used in the *Monitoring SSH for any host* recipe, fails because the plugin defaults to using the standard SSH TCP port number of 22.

This kind of setup is common in situations where an SSH server should not be open to the general public and is often employed as a "security by obscurity" method to reduce automated attacks against the server. The SSH daemon is therefore configured to listen on a different port, usually with a much higher number; administrators who need to use it are told what the port number is.

We'll deal with this situation and monitor the service in Nagios Core, even though it's running on a non-standard port. We'll do this by defining a new command that checks SSH on a specified port number, and creating a service definition that uses that command. The command will accept the port number to check as an argument.

The principles here should generalize well to any other situation where checking an alternative port is necessary, and the Nagios Core plugin being used to make the check supports doing so on an alternative port.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `troy.naginet`, a host defined in its own file, and listening on the non-standard SSH port of `5022`. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

A good first step may be to verify that we're able to access the SSH daemon from the monitoring server on the specified port. We can do this from the command line using the `ssh` client, specifying the port number with the `-p` option:

```
$ ssh troy.naginet -p 5022

```

Alternatively, you can run the `check_ssh` plugin directly from the command line:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_ssh -p 5022 troy.naginet

```

## How to do it...

We can set up a service check for SSH on a non-standard port as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit a suitable file containing command definitions, and find the definition for the `check_ssh` command. In the default installation, this file is `commands.cfg`. The `check_ssh` definition looks similar to the following code snippet:

    ```
    define command {
        command_name  check_ssh
        command_line  $USER1$/check_ssh $ARG1$ $HOSTADDRESS$
    }
    ```

3.  Beneath the `check_ssh` definition, add a new command definition as follows:

    ```
    define command {
        command_name  check_ssh_altport
     command_line  $USER1$/check_ssh -p $ARG1$ $HOSTADDRESS$
    }
    ```

4.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  troy.naginet
        alias      troy
        address    10.128.0.25
    }
    ```

5.  Beneath the definition for the host, place a new service definition using our new command:

    ```
    define service {
        use                  generic-service
        host_name            troy.naginet
        service_description  SSH_5022
        check_command        check_ssh_altport!5022
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core will begin running service checks using the `check_ssh` plugin, but will use the alternative destination port `5022` for its connection attempts for the service, which has a `service_description` of `SSH_5022`.

## How it works...

The configuration added in the preceding section has almost exactly the same end result as adding a default `check_ssh` service; the only difference is that a different port is checked in order to make the connection. We use the `check_ssh_altport` command to do this, which we also defined ourselves in a syntax very similar to the `check_ssh` definition.

The difference is that the command accepts an argument which is used as a value for the `-p` option to the `check_ssh` plugin, to check the specified port number; in this case, TCP port `5022`, rather than the default of port `22`.

## There's more...

Since arguments in Nagios Core can include spaces, we could also have defined the service check as follows, without having to define an extra command:

```
define service {
    use                  generic-service
    host_name            troy.naginet
    service_description  SSH_5022
    check_command        check_ssh!-p 5022
}
```

This is because the `$ARG1$` macro representing the argument is still used in the original `check_ssh` command, but it needs to have the option included as well as its value. The difference is mainly one of preference, depending on which we feel is clearer and more maintainable. It may help to consider whether a well-named command could assist someone else reading our configuration in understanding what is meant.

## See also

*   The *Monitoring SSH for any host* recipe in this chapter
*   The *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Monitoring mail services

In this recipe, we'll learn how to monitor three common mail services for a nominated host: **SMTP**, **POP**, and **IMAP**. We'll also see how to use the same structure to include additional checks for secure, encrypted versions of each of these services: **SMTPS**, **POPS**, and **IMAPS**.

For simplicity, we'll assume in this recipe that all three of these services are running on the same host, but the procedure will generalize easily for the common case where there are designated servers for one or more of the preceding functions.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `troy.naginet`, a host defined in its own file. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

Checking the connectivity for the required services on the target server is also a good idea, to make sure that the automated connections the monitoring server will be making on the appropriate protocols and ports will actually work as expected. For the plain unencrypted mail services, this could be done via **Telnet** to the appropriate ports. For SMTP:

```
$ telnet marathon.naginet 25
Trying 10.128.0.34...
Connected to marathon.naginet.
Escape character is '^]'.
220 marathon.naginet ESMTP Postfix

```

For POP:

```
$ telnet marathon.naginet 110
Trying 10.128.0.34...
Connected to marathon.naginet.
Escape character is '^]'.
+OK Hello there.

```

And for IMAP:

```
$ telnet marathon.naginet 143
Trying 10.128.0.34...
Connected to marathon.naginet.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE...

```

For secure services, one possibility for checking is using the `openssl` client. For SMTPS on its "classic" port number of 465:

```
$ openssl s_client -host marathon.naginet -port 465
CONNECTED(00000003)
...
220 marathon.naginet ESMTP Postfix

```

For POPS:

```
$ openssl s_client -host marathon.naginet -port 995
CONNECTED(00000003)
...
+OK Hello there.

```

And for IMAPS:

```
$ openssl s_client -host marathon.naginet -port 993
CONNECTED(00000003)
...
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE...

```

If you prefer, you could instead use a network scanner such as `nmap` to test whether the ports are open and responsive.

Once we've verified the connectivity for the mail services that we need, and also verified whether the host itself is being configured and checked in Nagios Core, we can add the appropriate service checks.

## How to do it...

We can add unencrypted mail service checks for SMTP, POP, and IMAP services on our host as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  marathon.naginet
        alias      marathon
        address    10.128.0.34
    }
    ```

3.  Beneath the definition for the host, place three new service definitions, one for each of the appropriate mail services:

    ```
    define service {
        use                  generic-service
        host_name            marathon.naginet
        service_description  SMTP
        check_command        check_smtp
    }
    define service {
        use                  generic-service
        host_name            marathon.naginet
        service_description  POP
        check_command        check_pop
    }
    define service {
        use                  generic-service
        host_name            marathon.naginet
        service_description  IMAP
        check_command        check_imap
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart
    ```

With this done, three new service checks will start taking place, with the appropriate contacts and contact groups notified when an attempt to connect to any of the services fails. Details for these services will also become available in the **Services** section of the web interface.

## How it works...

The configuration added in the preceding section adds three new service checks to the existing `marathon.naginet` host:

*   `SMTP`, which uses the `check_smtp` command to open an SMTP session
*   `POP`, which uses the `check_pop` command to open a POP session
*   `IMAP`, which uses the `check_imap` command to open an IMAP session

In all three cases, the connectivity and responsiveness of the service is checked, and determined to be `OK` if it returns appropriate values within an acceptable time frame.

It's important to note that the configuration defined here doesn't actually send or receive any e-mail messages; it merely checks the basic connectivity of the service, and whether it answers simple requests. Therefore, just because the status is `OK` does not necessarily mean that e-mail messages are being correctly delivered; it could just mean that the services are responding.

## There's more...

If it's necessary to check the secure SSL/TLS versions of each of these services, then the configuration is very similar but requires a little extra setup beforehand. This is because although plugins to check them are included in the Nagios Plugins setup, they are not configured to be used as commands. Note that this may well change in future versions of Nagios Core.

To add the appropriate commands, the following stanzas could be added to the commands configuration file, normally `/usr/local/nagios/etc/objects/commands.cfg`:

```
define command {
    command_name  check_ssmtp
    command_line  $USER1$/check_ssmtp $ARG1$ -H $HOSTADDRESS$
}
define command {
    command_name  check_spop
    command_line  $USER1$/check_spop $ARG1$ -H $HOSTADDRESS$
}
define command {
    command_name  check_simap
    command_line  $USER1$/check_simap $ARG1$ -H $HOSTADDRESS$
}
```

With this done, the following service definitions can be added to the appropriate host, either replacing or supplementing the checks for the unsecured services. They are just the same as the unsecured versions, except that an `s` is added to the `service_description` and to `check_command`:

```
define service {
    use                  generic-service
    host_name            marathon.naginet
    service_description  SSMTP
 check_command        check_ssmtp
}
define service {
    use                  generic-service
    host_name            marathon.naginet
    service_description  SPOP
 check_command        check_spop
}
define service {
    use                  generic-service
    host_name            marathon.naginet
    service_description  SIMAP
 check_command        check_simap
}
```

Finally, note that if you are managing more than one mail server running one or more of the preceding services, then it's a good practice to apply the service to a hostgroup containing all the applicable hosts, rather than creating new service definitions for each one. See the *Running a service on all hosts in a group* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts* to learn how to do this.

## See also

*   The *Creating a new host*, *Creating a new service*, *Running a service on all hosts in a group*, and *Creating a new hostgroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Monitoring web services

In this recipe, we'll set up a service check to monitor the responsiveness of an HTTP and HTTPS server. We'll use the `check_http` command and the plugin of the same name provided in the Nagios Plugins set to make HTTP and HTTPS requests of a web server, to ensure that it returns an appropriate and timely response. This is useful in situations where it's required to check whether a website is still functioning, particularly if there are times when it comes under heavy load or suffers denial of service attacks.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

An appropriate first step is making sure that the services we intend to check are accessible from the monitoring server running Nagios Core. This can be done from the command line, using an HTTP client such as `curl` or `wget`:

```
$ wget http://sparta.naginet/
$ curl http://sparta.naginet/

```

The `check_http` plugin binary could also be called directly to test this connectivity; we'd be hoping for an `HTTP` `OK` response, with a code of `200`:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_http -I sparta.naginet
HTTP OK: HTTP/1.1 200 OK - 453 bytes in 0.004 second response time |time=0.004264s;;;0.000000 size=453B;;;0

```

Optionally, we can check HTTPS the same way, adding the `-S` option for the plugin:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_http -S -I sparta.naginet
HTTP OK: HTTP/1.1 200 OK - 453 bytes in 0.058 second response time |time=0.057836s;;;0.000000 size=453B;;;0

```

Both may require the installation of a default page to be served by the host, probably something such as `index.html` or `default.asp`, depending on the web server software.

Once the HTTP connectivity to the host from the monitoring server is verified as working with appropriate responses, we can proceed to add our service check.

## How to do it...

We can add web service checks for our host as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  sparta.naginet
        alias      sparta
        address    10.128.0.21
    }
    ```

3.  Beneath the definition for the host, place a new service definition for the HTTP check:

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  HTTP
        check_command        check_http
    }
    ```

4.  If an HTTPS check is also needed, add an optional second service definition:

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  HTTPS
        check_command        check_http!-S
    }
    ```

5.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service called HTTP and optionally one called HTTPS will be added to the `sparta.naginet` host, and HTTP requests will be made from the server regularly, reporting if connectivity fails or a response comes back with an unexpected status. These services will both be visible in the **Services** section of the web interface.

## How it works...

The configuration added in the preceding section uses `check_http` as a plugin to make scheduled requests of the `sparta.naginet` server. By default, the index page is requested, so the request takes the following form:

```
GET / HTTP/1.0
User-Agent: check_http/v1.4.15 (nagios-plugins 1.4.15)
Connection: close

```

The plugin awaits a response, and then returns a status based on the following criteria:

*   Whether a well-formed HTTP response was received at all, within acceptable time bounds. If the response was too slow, it might raise a `CRITICAL` state when the plugin times out.
*   Whether the response code for the HTTP response was `200` `Found`, indicating that a document was identified and returned. A response code of `404` `Not` `Found` would prompt a `CRITICAL` state by default.

Inspecting the command definition for the default `check_http` command in `/usr/local/nagios/etc/objects/commands.cfg` gives some insight into how it uses the plugin of the same name:

```
define command {
    command_name  check_http
    command_line  $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
}
```

This uses three Nagios Core macros:

*   `$USER1$`: This expands to the directory in which the plugin scripts and binaries are kept; usually `/usr/local/nagios/libexec`.
*   `$HOSTADDRESS$`: This is the value for the `address` directive defined in the service's associated host; in this case `10.0.128.21`.
*   `$ARG1$`: This is one extra argument, if defined by the command; it allows us to add the `-S` option to the `check_http` call in order to run an HTTPS check.

## There's more...

There are a many other great switches available for the `check_http` plugin; a list of them is available by entering the command with no arguments:

```
# ./check_http
check_http: Could not parse arguments
Usage:
 check_http -H <vhost> | -I <IP-address> [-u <uri>] [-p <port>]
 [-w <warn time>] [-c <critical time>] [-t <timeout>] [-L] [-a auth]
 [-b proxy_auth] [-f <ok|warning|critcal|follow|sticky|stickyport>]
 [-e <expect>] [-s string] [-l] [-r <regex> | -R <case-insensitive regex>]
 [-P string] [-m <min_pg_size>:<max_pg_size>] [-4|-6] [-N] [-M <age>]
 [-A string] [-k string] [-S] [--sni] [-C <age>] [-T <content-type>]
 [-j method]

```

One particularly useful option here is the `-u` option, which allows us to request specific URLs other than the default index document from the server. This can be useful if we're in a situation that requires setting up checks for more than one page on a site, which can be a nice supplement to code unit testing when a site is deployed or updated.

For example, if we wanted to check that three pages were returning `200` `Found` responses: `about.php`, `products.php`, and `contact.php`, then we could set up a new command similar to the following to check a specific page:

```
define command {
    command_name  check_http_page
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -u $ARG1$
}
```

This would allow us to make three service checks similar to the following, using the new command:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-about
 check_command        check_http_page!/about.php
}
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-products
 check_command        check_http_page!/products.php
}
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-contact
 check_command        check_http_page!/contact.php
}
```

These service checks would run the same way as the one demonstrated in the recipe, except they would each request a specific page. Note the leading slashes on the URLs are required.

Similarly, the `-H` option allows you to specify hostnames, which is helpful on servers hosting more than one site. This could be done by setting up a command as follows:

```
define command {
    command_name  check_http_host
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -H $ARG1$
}
```

This would allow you to check two sites on the same host, [http://www.naginet/](http://www.naginet/) and [http://dev.naginet/](http://dev.naginet/), in separate service checks:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-www
 check_command        check_http_host!www.naginet
}
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP-dev
 check_command        check_http_host!dev.naginet
}
```

It's worth noting that the `check_http` request will show up in your server logs with its regular requests. If you're concerned about these distorted statistics or the appearance of unwanted values in reports, then it may be easiest to filter these out using its `User-Agent` header value, which includes the string `check_http`.

## See also

*   The *Checking a website returns a given string* in this chapter
*   The *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* and *Customizing an existing plugin* recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Checking that a website returns a given string

In this recipe, we'll build on the basic web service monitoring established in the *Monitoring web services* recipe in this chapter, and learn how to create a command that uses the `check_http` plugin to ensure that a particular string is included as part of an HTTP response.

By default, there's no Nagios Core command defined to use the plugin in this way, so the recipe will include defining a command before using it as part of a service check.

This may be necessary if we're monitoring a website on a server that may not necessarily return a `404` `Not` `Found` or similar error that will flag a `WARNING` or `CRITICAL` state in Nagios; rather than merely checking if a document was found, we can check if it matches a string, to see if it resembles the particular document we expected.

This kind of setup is a nice complement to a suite of code unit tests for a website or web application.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file, and we'll check that it's returning the simple string, `naginet`, in its responses. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

You should set up basic HTTP monitoring for the host first, as established in the *Checking web services* recipe in this chapter, to make sure that there is connectivity between the monitoring server and the host, and that requests and responses are both working correctly with appropriate error codes.

## How to do it...

We can set up a service check that includes an HTTP response content check as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit a suitable file containing command definitions, and find the definition for the `check_http` command. In the QuickStart installation, this file is `commands.cfg`. The `check_http` definition looks similar to the following code snippet:

    ```
    define command {
        command_name  check_http
        command_line  $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
    }
    ```

3.  Beneath the `check_http` definition, add a new command definition as follows:

    ```
    define command {
        command_name  check_http_content
     command_line  $USER1$/check_http -I $HOSTADDRESS$ -s $ARG1$
    }
    ```

4.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  sparta.naginet
        alias      sparta
        address    10.128.0.21
    }
    ```

5.  Beneath the definition for the host and beneath any other checks that might use `check_http`, place a new service definition using our new command:

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  HTTP-content
        check_command        check_http_content!naginet
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core should begin making HTTP requests to monitor the service as `check_http` normally does, except that it will only return an `OK` state if the content of the website includes the string `naginet`. Otherwise, it will generate an alert and flag the service as `CRITICAL`, with a message similar to the following:

```
HTTP CRITICAL: HTTP/1.1 200 OK - string 'naginet' not found on 'http://10.128.0.21:80/' - 453 bytes in 0.006 second response time

```

## How it works...

One of the many options for `check_http` is `-s`, short for `--string` , which takes a single argument specifying a string that must occur in the content for the service check to return an `OK` state.

When the HTTP response is received, `check_http` examines the text in the response to see if it matches the string specified, on top of its usual behavior of flagging `WARNING` or `CRITICAL` states for connectivity or timeout problems.

Note that in order to make this work, it was necessary to define a new command that uses the first argument (in this case the string `naginet`) as the value for the `-s` option to `check_http`. The full command line executed would look similar to the following command:

```
$ /usr/local/nagios/libexec/check_http -H sparta.naginet -s naginet

```

## There's more...

The `check_http` plugin allows considerably more than single string checks, if it's necessary to test for the presence of a regular expression in the content. This can be done using the `-r` or `--regex` options. We could define a command to check for regular expressions as follows:

```
define command {
    command_name  check_http_regex
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -r $ARG1$
}
```

If it's necessary to check that a particular regular expression doesn't match the content, this is possible by adding the `--invert-regex` flag:

```
define command {
    command_name  check_http_noregex
    command_line  $USER1$/check_http -I $HOSTADDRESS$ -r $ARG1$ --invert-regex
}
```

A service check using this command would return `CRITICAL` if the response was found to match the pattern provided as the first argument to a `check_command` directive.

Other similar options include `-e` or `--expect` , which allows specifying a comma-separated set of strings, at least one of which must match the first line of the header for the check to pass.

## See also

*   The *Monitoring web services* recipe in this chapter
*   The *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* and *Customizing an existing plugin* recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Monitoring database services

In this recipe, we'll learn how Nagios Core can be used to monitor the status of a database server. We'll demonstrate this with the popular MySQL as an example, using the `check_mysql` plugin, and we'll discuss running an actual test query and specifying a similar check for PostgreSQL in the *There's more* section of this recipe.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `delphi.naginet`, a host defined in its own file. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

For a check on a remote host to work from the monitoring server, the database server will need to be listening on an appropriate network interface. It's also necessary to make sure that an appropriate database user account exists with which the `check_mysql` plugin may authenticate. It's a good idea to make this into a dedicated user with no privileges on any database, because the credentials need to be stored in plain text, which could be a security risk if more sensitive credentials were used.

For MySQL, we could create a new user with no privileges with the following command, assuming that the monitoring server `olympus.naginet` has `10.128.0.11` as an IPv4 address. I've used a randomly generated password here:

```
mysql> CREATE USER 'nagios'@'10.128.0.11' IDENTIFIED BY 'UVocjPHoH0';

```

We can then check the connectivity using the `mysql` client on the monitoring server:

```
$ mysql --host=delphi.naginet --user=nagios --password=UVocjPHoH0
mysql>

```

Or alternatively, by running the plugin directly from the command line as the `nagios` user:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_mysql -H delphi.naginet -u nagios -p UVocjPHoH0
Uptime: 1631  Threads: 1  Questions: 102  Slow queries: 0  Opens: 99 
Flush tables: 1  Open tables: 23  Queries per second avg: 0.62

```

If we did not have the MySQL libraries installed when we built the Nagios plugins, we may find that we do not have the `check_mysql` and `check_mysql_query` binaries in `/usr/local/nagios/libexec`. We can fix this by installing the MySQL shared libraries on the monitoring system, and rebuilding and reinstalling the Nagios Plugins package.

By default, it's also necessary to define new commands to actually use these plugins as well, which we'll do in this recipe.

## How to do it...

We can set up some basic database monitoring for our MySQL server as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit a suitable file containing command definitions, perhaps `commands.cfg`, and add the following definition:

    ```
    define command {
        command_name  check_mysql
     command_line  $USER1$/check_mysql -H $HOSTADDRESS$ -u $ARG1$ -p $ARG2$
    }
    ```

3.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  delphi.naginet
        alias      delphi
        address    10.128.0.51
    }
    ```

4.  Beneath the definition for the host, place a new service definition for the MySQL check, including the username and password chosen earlier for arguments:

    ```
    define service {
        use                  generic-service
        host_name            delphi.naginet
        service_description  MYSQL
     check_command        check_mysql!nagios!UVocjPHoH0
    }
    ```

5.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service check with description `MYSQL` will be added for the `delphi.naginet` host, which will employ the `check_mysql` plugin to report the status of the MySQL server. The output will also include statistics about its uptime, open tables, and query averages, and like all service output will be visible in the web interface under **Services**.

## How it works...

This configuration defines a new command named `check_mysql` to use the plugin of the same name, accepting two arguments; the first is the username of the test Nagios Core user, in this case `nagios`, and the second is the password for that user. The `check_mysql` plugin acts as a MySQL client using the credentials provided to it, and requests diagnostic information from the database, which it returns as part of its check.

If it has problems connecting to or using the MySQL server, it will flag a status of `CRITICAL`, and generate appropriate notifications.

## There's more...

We can optionally check access to a specific database using the plugin by supplying a value to the `-d` parameter. This should be a database to which the `nagios` user has been given access, otherwise the check would fail.

If we want to check whether we can actually run a query after connecting, we could extend this even further to use the `check_mysql_query` plugin:

```
define command {
    command_name  check_mysql_query
 command_line  $USER1$/check_mysql_query -H $HOSTADDRESS$ -u $ARG1$ -p $ARG2$ -d $ARG3$ -q $ARG4$
}
define service {
    use                  generic-service
    host_name            delphi.naginet
    service_description  MYSQL_QUERY
 check_command        check_mysql_query!nagios!UVocjPHoH0!exampledb!"SELECT COUNT(1) FROM exampletbl"
}
```

The preceding code snippet would attempt to run the `SELECT` `COUNT(1)` `FROM` `exampletbl` query on the `exampledb` database. Note that it is important to wrap the query in quotes so that it gets processed as one argument, rather than several.

A similar service check to the one specified in this recipe could be configured for PostgreSQL database servers, using the `check_pgsql` plugin, also part of the standard Nagios Plugins set. The command and service check definitions might look similar to the following code snippet:

```
define command {
    command_name  check_pgsql
    command_line  $USER1$/check_pgsql -H $HOSTADDRESS$ -p $ARG1$
}
define service {
    use                  generic-service
    host_name            delphi.naginet
    service_description  PGSQL
    check_command        check_pgsql!N4Nw8o8X
}
```

In the preceding example, an access would need to be granted on the PostgreSQL server for the monitoring server's IP address in `pg_hba.conf`, with access to the default standard `template1` database.

In production environments, it's often the case that for security or programming policy reasons, database servers are not actually configured to accept direct connections over network interfaces, even on secure interfaces. Packaged MySQL and PostgreSQL servers on many systems will in fact default to listening only on the `localhost` interface on `127.0.0.1`.

This can complicate the monitoring setup a little, but it can usually be addressed by installing a remote Nagios plugin execution agent on the database server, such as NRPE or NSclient++. NRPE usage is addressed in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*, and uses a MySQL server configured in this way as its demonstration of the concept.

## See also

*   The *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Monitoring local services on a remote machine with NRPE* recipe in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*

# Monitoring the output of an SNMP query

In this recipe, we'll learn how to use the `check_snmp` plugin to monitor the output given by **Simple Network Management Protocol** (**SNMP**) requests.

Despite its name, SNMP is not really a very simple protocol, but it's a very common method for accessing information on many kinds of networked devices, including monitoring boards, usage meters, and storage appliances, as well as workstations, servers, and routing equipment.

Because SNMP is so widely supported and typically able to produce such a large volume of information to trusted hosts, it's an excellent way to gather information from hosts that's not otherwise retrievable from network services. For example, while checking for a PING response from a large router is simple enough, there may not be an easy way to check properties, such as the state of each of its interfaces, or the presence of a certain route in its routing tables.

Using `check_snmp` in Nagios Core allows automated retrieval of this information from the devices, and generating alerts appropriately. While its setup is somewhat complex, it is worth learning how to use it, as it is among the most powerful plugins in Nagios Core for network administrators, and it is quite typical to see dozens of commands defined for its use in a typical configuration for a large network. It can often be used to complement or even replace remote plugin execution daemons such as NRPE or NSclient++.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. You should also understand the basics of how hosts and services relate, which is covered in the recipes of [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

This recipe assumes a basic knowledge of SNMP, including its general intended purpose, the concept of an SNMP community, and what SNMP MIBs and OIDs are. In particular, if you're looking to monitor some property of a network device that's available to you via SNMP, you should know what the OID for that data is. This information is often available in the documentation for network devices, or can be deduced by running an appropriate `snmpwalk` command against the host to view the output for all its OIDs.

You should check that an SNMP daemon is running on the target host, and also that the `check_snmp` plugin is available on the monitoring host. It is included as part of the standard Nagios Plugins, so provided the Net-SNMP libraries were available on the system when these were compiled, the plugin should be available. If it is not, you may need to install the Net-SNMP libraries on your monitoring system and recompile the plugins.

We'll use the example of retrieving the total process count from a Linux server with hostname `ithaca.naginet`, and flagging `WARNING` and `CRITICAL` states at appropriate high ranges. We'll also discuss how to test for the presence or absence of strings, rather than numeric thresholds.

It's a good idea to test that the host will respond to SNMP queries in the expected form. We can test this with `snmpget`. Assuming a community name of `public`, we could write:

```
$ snmpget -v1 -c public ithaca.naginet .1.3.6.1.2.1.25.1.6.0
iso.3.6.1.2.1.25.1.6.0 = Gauge32: 81

```

We can also test the plugin by running it directly as the `nagios` user:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_snmp -H ithaca.naginet -C public -o .1.3.6.1.2.1.25.1.6.0
SNMP OK - 81 | iso.3.6.1.2.1.25.1.6.0=81

```

## How to do it...

We can define a command and service check for the Linux process count OID as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit a suitable file containing command definitions, perhaps `commands.cfg`, and add the following definition to the end of the file.

    ```
    define command {
        command_name  check_snmp_linux_procs
     command_line  $USER1$/check_snmp -H $HOSTADDRESS$ -C $ARG1$ -o .1.3.6.1.2.1.25.1.6.0 -w 100 -c 200
    }
    ```

3.  Edit the file containing the definition for the host. The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  ithaca.naginet
        alias      ithaca
        address    10.128.0.61
    }
    ```

4.  Beneath the definition for the host, place a new service definition using our new command. Replace `public` with the name of your SNMP community if it differs:

    ```
    define service {
        use                  generic-service
        host_name            ithaca.naginet
        service_description  SNMP_PROCS
     check_command        check_snmp_linux_procs!public
    }
    ```

5.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service check with a description of `SNMP_PROCS` will be added to the `ithaca.naginet` host, and the `check_snmp` plugin will issue a request for the value of the specified OID as its regular check. It will flag a `WARNING` state if the count is greater than `100`, and a `CRITICAL` state if greater than `200`, notifying accordingly. All this appears in the web interface the same way as any other service, under the **Services** menu item.

## How it works...

The preceding configuration defines both a new command based around the `check_snmp` plugin, and in turn, a new service check using that command for the `ithaca.naginet` server. The community name for the SNMP request, `public`, is passed into the command as an argument; everything else, including the OID to be requested, is fixed into the `check_snmp_linux_procs` command definition.

A part of the command line defined includes the `-w` and `-c` options. For numeric outputs like ours, these are used to define the limits for the value beyond which a `WARNING` or `CRITICAL` state is raised, respectively. In this case, we define a `WARNING` threshold of `100` processes, and a `CRITICAL` threshold of `200` processes.

Similarly, if the SNMP check fails completely due to connectivity problems or syntax errors, an `UNKNOWN` state will be reported.

## There's more...

It's also possible to test the output of SNMP checks to see if they match a particular string or pattern to determine whether the check succeeded. If we needed to check that the system's hostname was `ithaca.naginet`, for example (perhaps as a simple test SNMP query that should always succeed), then we might set up a command definition as follows:

```
define command {
    command_name  check_snmp_hostname
 command_line  $USER1$/check_snmp -H $HOSTADDRESS$ -C $ARG1$ -o .1.3.6.1.2.1.1.5.0 -r $ARG2$
}
```

With a corresponding service check as follows:

```
define service {
    use                  generic-service    host_name            ithaca.naginet
    service_description  SNMP_HOSTNAME
 check_command        check_snmp_hostname!public!ithaca
}
```

This particular check would only succeed if the SNMP query succeeds and returns a string matching the string `ithaca`, as specified in the second argument.

## See also

*   The *Creating an SNMP OID to monitor* in this chapter
*   The *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Monitoring a RAID or other hardware device

In this recipe, we'll learn a general strategy for monitoring the properties of hardware devices. Because of the different ways that vendors implement their hardware, this tends to be less straightforward than monitoring standard network services.

There are at least four general approaches to this problem.

## Getting ready

You will need to know some specifics about the hardware that you want to monitor, including the model number. You should preferably also have a Nagios Core 3.0 server that was compiled with Net-SNMP libraries available to build the `check_snmp` plugin, part of the Nagios Plugins set.

## How to do it...

We can find a way to monitor an arbitrary hardware device on a local or remote machine as follows:

1.  Check if official or unofficial Nagios Core plugins already exist for polling the particular device. The best place to start is with Nagios Exchange at [*http://exchange.nagios.org/*](http://exchange.nagios.org/); just search for the make of hardware, and see if a plugin already exists, per the *Finding a plugin* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*. You can then install it by following the *Installing a plugin* recipe in the same chapter.
2.  Check if any of the values you need from the hardware are or can be exported as SNMP OIDs, to be checked with the *Monitoring the output of an SNMP query* recipe in this chapter.
3.  If they aren't, but there's a command-line diagnostic tool with output, or a return value that can be used as the check, you could consider exporting it as a custom OID in an SNMP server, by using the *Creating a new SNMP OID to monitor* recipe in this chapter.
4.  Finally, we may have to resort to writing our own plugin. This is not actually as difficult as it may seem; it is discussed in the *Writing a new plugin from scratch* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*. This may be the only option for custom or very uncommon hardware.

## How it works...

If we find an appropriate plugin for the hardware online, the main snag here is that we will need to not only be sure that the plugin works, by testing it against the hardware in both an `OK` and a `CRITICAL` state (which might be hard to do), but we will also need to make sure that the plugin is safe to run. The plugins on Nagios Exchange are reviewed before they are added, but the code for plugins that you find on any other website might not be safe to run.

Using SNMP for these kinds of checks wherever possible has two advantages, in that the values can be checked using a standard Nagios plugin, `check_snmp`, and the values can also be read over the network, meaning that we may not need to rely on remote execution daemons such as NRPE or NSclient++ to get this information.

## See also

*   The *Monitoring the output of an SNMP query* and *Creating a new SNMP OID to monitor* recipes in this chapter
*   The *Finding a plugin*, *Installing a plugin*, and *Writing a new plugin from scratch* recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Creating an SNMP OID to monitor

In this recipe, we'll learn how to configure a Net-SNMP `snmpd` server on a Linux server to return the output of a command in an SNMP OID. This can be useful as an alternative to NRPE monitoring for information that is not otherwise available in a checkable network service, so that Nagios Core can check it via its standard `check_snmp` method.

As an example, this can be a very good way of monitoring hardware devices, such as **RAID** arrays on remote servers, where command-line diagnostic tools are available to report a status as a number or string, but they only work locally and don't otherwise include any information in an SNMP MIB tree.

## Getting ready

The host we intend to check should be running a Net-SNMP `snmpd` server that allows full read access to the MIB tree for a specified community string, such as `public`. This SNMP server should be capable of using the `exec` directive in its configuration to return the output of a command as the value of an SNMP OID when requested by an SNMP client. As such, you will need to know the basics of SNMP.

You will also need a Nagios Core 3.0 or newer server with SNMP enabled in order to actually monitor the OID, which is explained in the *Monitoring the output of an SNMP query* recipe.

In this example, we'll deal with a simple script running on a target host `ithaca.naginet` called `/usr/local/bin/raidstat` that returns an integer: zero for the RAID being in a good state, and non-zero to signal any problems. We'll assume this tool can be run as any user and does not require root privileges.

## How to do it...

We can set up our custom SNMP OID as follows:

1.  Add the following line to the `snmpd.conf` file on the target host, substituting in the path of the command that generates the needed output:

    ```
    exec raidstat /usr/local/bin/raidstat

    ```

2.  Restart the `snmpd` server on the target host, which might be done as follows, depending on the system:

    ```
    # /etc/init.d/snmpd restart

    ```

3.  Back on the monitoring server, we can walk the `.1.3.6.1.4.1.2021` OID, and find our `raidstat` OID and its value in the output:

    ```
    $ snmpwalk -v1 -c public ithaca.naginet .1.3.6.1.4.1.2021 | less
    ...
    iso.3.6.1.4.1.2021.8.1.2.1 = STRING: "raidstat"
    iso.3.6.1.4.1.2021.8.1.3.1 = STRING: "/usr/local/bin/raidstat"
    iso.3.6.1.4.1.2021.8.1.100.1 = INTEGER: 0
    iso.3.6.1.4.1.2021.8.1.101.1 = STRING: "GOOD"
    ...

    ```

4.  We now know which OID we can query for a check using the *Monitoring the output of an SNMP query* recipe in this chapter, and can test retrieving it directly with `snmpget`:

    ```
    $ snmpget -v1 -c public ithaca.naginet .1.3.6.1.4.1.2021.8.1.100.1

    ```

## How it works...

The `exec` call in the `snmpd` configuration defines a program that should be run to return a new value in the OID tree. The four OIDs we found in the output are as follows:

*   `1.3.6.1.4.1.2021.8.1.2.1`: This is the name of the OID we've assigned, `raidstat`
*   `1.3.6.1.4.1.2021.8.1.3.1`: This is the full path to the script that was called, `/usr/local/bin/raidstat`
*   `1.3.6.1.4.1.2021.8.1.100.1`: This is the return value from the call, expressed as an integer
*   `1.3.6.1.4.1.2021.8.1.101.1`: This is any output from the call, expressed as a string

This setup can therefore be used to check the return value and character output of any command that the `snmpd` user is able to execute, a good method for ad hoc monitoring for very specific cases like this.

## There's more...

More than one `exec` command can be added to the server configuration if required. For example, if we needed to check the state of the CPU temperature with a command `tempstat`, then we might define:

```
exec raidstat /usr/local/bin/raidstat
exec tempstat /usr/local/bin/tempstat

```

The return values and output for both would then show up in the SNMP output, in separate OIDs.

If necessary, the command definitions can also be followed by arguments:

```
exec tempstat /usr/local/bin/tempstat --cpu=0

```

A full discussion of how `exec` and the similar `pass` configuration for Net-SNMP works is outside the scope of this book, but is discussed extensively in its documentation available at the time of writing at [http://www.net-snmp.org/docs/readmefiles.html](http://www.net-snmp.org/docs/readmefiles.html).

Note that if exporting values via SNMP is unsuitable, an alternative for remote monitoring is to use **Nagios Remote Plugin Executor** (**NRPE**) to run a check on the target server and return the result to the monitoring server. This is discussed in the recipes of [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*.

## See also

*   The *Creating an SNMP OID to monitor* recipe in this chapter
*   The *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Monitoring local services on a remote machine with NRPE* recipes in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*