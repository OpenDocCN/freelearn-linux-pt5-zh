# Chapter 1. Understanding Hosts, Services, and Contacts

In this chapter we will cover the following recipes:

*   Creating a new network host
*   Creating a new HTTP service
*   Creating a new e-mail contact
*   Verifying configuration
*   Creating a new hostgroup
*   Creating a new servicegroup
*   Creating a new contactgroup
*   Creating a new time period
*   Running a service on all hosts in a group

# Introduction

**Nagios Core** is appropriate for monitoring services and states on all sorts of hosts, and one of its primary advantages is that the configuration can be as simple or as complex as required. Many Nagios Core users will only ever use the software as a way to send PING requests to a few hosts on their local network or possibly the Internet, and to send e-mail or pager messages to the administrator if they don't get any replies. Nagios Core is capable of monitoring vastly more complex systems than this, scaling from simple LAN configurations to being the cornerstone for monitoring an entire network.

However, for both simple and complex configurations of Nagios Core, the most basic building blocks of configuration are **hosts**, **services**, and **contacts**. These are the three things that administrators of even very simple networking setups will end up editing and probably creating. If you're a beginner to Nagios Core, then you might have changed a hostname here and there or copied a stanza in a configuration to get it to do what you want. In this chapter, we're going to look at what these configurations do in a bit more depth than that.

In a Nagios Core configuration:

*   Hosts usually correspond to some sort of computer. This could be a physical or virtual machine accessible over the network, or the monitoring server itself. Conceptually, however, a host can monitor any kind of network entity, such as the endpoint of a VPN.
*   Services usually correspond to an arrangement for Nagios Core to check something about a host, whether that's something as simple as getting PING replies from it, or something more complicated such as checking that the value of an SNMP OID is within acceptable bounds.
*   Contacts define a means to notify someone when events happen to our services on our hosts, such as not being able to get a PING response, or being unable to send a test e-mail message.

In this chapter, we'll add all three of these, and we'll learn how to group their definitions together to make the configuration more readable, and to work with hosts in groups rather than having to edit each one individually. We'll also set up a custom time period for notifications, so that hardworking system administrators like us don't end up getting paged at midnight unnecessarily!

# Creating a new network host

In this recipe, we'll start with the default Nagios Core configuration, and set up a host definition for a server that responds to PING on our local network. The end result will be that Nagios Core will add our new host to its internal tables when it starts up, and will automatically check it (probably using PING) on a regular basis. In this example, I'll use my Nagios Core monitoring server with a **Domain Name System** (**DNS**) name of `olympus.naginet`, and add a host definition for a webserver with a DNS name of `sparta.naginet`. This is all on my local network – `10.128.0.0/24`.

## Getting ready

You'll need a working Nagios Core 3.0 or greater installation with a web interface, with all the Nagios Core Plugins installed. If you have not yet installed Nagios Core, then you should start with the QuickStart guide: [http://nagios.sourceforge.net/docs/3_0/quickstart.html](http://nagios.sourceforge.net/docs/3_0/quickstart.html).

We'll assume that the configuration file that Nagios Core reads on startup is located at `/usr/local/nagios/etc/nagios.cfg`, as is the case with the default install. It shouldn't matter where you include this new host definition in the configuration, as long as Nagios Core is going to read the file at some point, but it might be a good idea to give each host its own file in a separate objects directory, which we'll do here. You should have access to a shell on the server, and be able to write text files using an editor of your choice; I'll use `vi`. You will need root privileges on the server via `su` or `sudo`.

You should know how to restart Nagios Core on the server, so that the configuration you're going to add gets applied. It shouldn't be necessary to restart the whole server to do this! A common location for the startup/shutdown script on Unix-like hosts is `/etc/init.d/nagios`, which I'll use here.

You should also get the hostname or IP address of the server you'd like to monitor ready. It's good practice to use the IP address if you can, which will mean your checks keep working even if DNS is unavailable. You shouldn't need the subnet mask or anything like that; Nagios Core will only need whatever information the PING tool would need for its own `check_ping` command.

Finally, you should test things first; confirm that you're able to reach the host from the Nagios Core server via PING by checking directly from the shell, to make sure your network stack, routes, firewalls, and netmasks are all correct:

```
tom@olympus:~$ ping 10.128.0.21
PING sparta.naginet (10.128.0.21) 56(84) bytes of data.
64 bytes from sparta.naginet (10.128.0.21): icmp_req=1 ttl=64 time=0.149 ms

```

## How to do it...

We can create the new host definition for `sparta.naginet` as follows:

1.  Change directory to `/usr/local/nagios/etc/objects`, and create a new file called `sparta.naginet.cfg`:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi sparta.naginet.cfg

    ```

2.  Write the following into the file, changing the values in bold as appropriate for your own setup:

    ```
    define host {
     host_name              sparta.naginet
     alias                  sparta
     address                10.128.0.21
     max_check_attempts     3
        check_period           24x7
        check_command          check-host-alive
        contacts               nagiosadmin
        notification_interval  60
        notification_period    24x7
    }
    ```

3.  Change directory to `/usr/local/nagios/etc`, and edit the `nagios.cfg` file:

    ```
    # cd ..
    # vi nagios.cfg

    ```

4.  At the end of the file add the following line:

    ```
    cfg_file=/usr/local/nagios/etc/objects/sparta.naginet.cfg

    ```

5.  Restart the Nagios Core server:

    ```
    # /etc/init.d/nagios restart

    ```

If the server restarted successfully, the web interface should show a brand new host in the **Hosts** list, in **PENDING** state as it waits to run a check that the host is alive:

![How to do it...](img/5566_01_01.jpg)

In the next few minutes, it should change to green to show that the check passed and the host is **UP**, assuming that the check succeeded:

![How to do it...](img/5566_01_02.jpg)

If the test failed and Nagios Core was not able to get a PING response from the target machine after three tries, for whatever reason, then it would probably look similar to the following screenshot:

![How to do it...](img/5566_01_03.jpg)

## How it works...

The configuration we included in this section adds a host to Nagios Core's list of hosts. It will periodically check the host by sending a PING request, checking to see if it receives a reply, and updating the host's status as shown in the Nagios Core web interface accordingly. We haven't defined any other services to check for this host yet, nor have we specified what action it should take if the host is down. However, the host itself will be automatically checked at regular intervals by Nagios Core, and we can view its state in the web interface at any time.

The directives we defined in the preceding configuration are explained as follows:

*   `host_name`: This defines the hostname of the machine, used internally by Nagios Core to refer to its host. It will end up being used in other parts of the configuration.
*   `alias`: This defines a more recognizable human-readable name for the host; it appears in the web interface. It could also be used for a full-text description of the host.
*   `address`: This defines the IP address of the machine. This is the actual value that Nagios Core will use for contacting the server; using an IP address rather than a DNS name is generally best practice, so that the checks continue to work even if DNS is not functioning.
*   `max_check_attempts`: This defines the number of times Nagios Core should try to repeat the check if checks fail. Here, we've defined a value of `3`, meaning that Nagios Core will try two more times to PING the target host after first finding it down.
*   `check_period`: This references the time period that the host should be checked. `24x7` is a time period defined in the default configuration for Nagios Core. This is a sensible value for hosts, as it means the host will always be checked. This defines how often Nagios Core will check the host, and not how often it will notify anyone.
*   `check_command`: This references the command that will be used to check whether the host is `UP`, `DOWN`, or `UNREACHABLE`. In this case, a QuickStart Nagios Core configuration defines `check-host-alive` as a PING check, which is a good test of basic network connectivity, and a sensible default for most hosts. This directive is actually not required to make a valid host, but you will want to include it under most circumstances; without it, no checks will be run.
*   `contacts`: This references the contact or contacts that will be notified about state changes in the host. In this instance, we've used `nagiosadmin`, which is defined in the QuickStart Nagios Core configuration.
*   `notification_interval`: This defines how regularly the host should repeat its notifications if it is having problems. Here, we've used a value of `60`, which corresponds to 60 minutes or one hour.
*   `notification_period`: This references the time period during which Nagios Core should send out notifications, if there are problems. Here, we're again using the `24x7` time period; for other hosts, another time period such as `workhours` might be more appropriate.

Note that we added the definition in its own file called `sparta.naginet.cfg` , and then referred to it in the main `nagios.cfg` configuration file. This is simply a conventional way of laying out hosts, and it happens to be quite a tidy way to manage things to keep definitions in their own files.

## There's more...

There are a lot of other useful parameters for hosts, but the ones we've used include everything that's required.

While this is a perfectly valid way of specifying a host, it's more typical to define a host based on some template, with definitions of how often the host should be checked, who should be contacted when its state changes and on what basis, and similar properties. Nagios Core's QuickStart sample configuration defines a simple template host called `generic-host`, which could be used by extending the host definition with the `use` directive:

```
define host {
 use                 generic-host
    name                sparta
    host_name           sparta.naginet
    address             10.128.0.21
    max_check_attempts  3
    contacts            nagiosadmin
}
```

This uses all the parameters defined for `generic-host`, and then adds on the details of the specific host that needs to be checked. Note that if you use `generic-host`, then you will need to define `check_command` in your host definition. If you're curious to see what's defined in `generic-host`, then you can find its definition in `/usr/local/nagios/etc/objects/templates.cfg`.

## See also

*   The *Using an alternative check command for hosts* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Specifying how frequently to check a host* recipe in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*
*   The *Grouping configuration files in directories* and *Using inheritance to simplify configuration* recipes in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Managing Configuration*

# Creating a new HTTP service

In this recipe, we'll create a new service to check on an existing host. Specifically, we'll check our `sparta.naginet` server to see if it's responding to HTTP requests on the usual HTTP TCP port 80\. To do this, we'll be using a predefined command called `check_http` , which in turn uses one of the standard set of Nagios Core plugins, also called `check_http`. If you don't yet have a web server defined as a host in Nagios Core, then you may like to try the *Creating a new network host* recipe in this chapter.

After we've done this, not only will our host be checked for a PING response by `check_command`, but Nagios Core will also run a periodic check to ensure that an HTTP service on that machine is responding to requests on the same host.

## Getting ready

You'll need a working Nagios Core 3.0 or greater installation with a web interface, all the Nagios Plugins installed, and at least one host defined. If you need to set up a host definition for your web server first, then you might like to read the *Creating a new network host* recipe in this chapter, for which the requirements are the same.

It would be a good idea to test that the Nagios Core server is actually able to contact the web server first, to ensure that the check we're about to set up should succeed. The standard **telnet** tool is a fine way to test that a response comes back from TCP port 80 as we would expect from a web server:

```
tom@olympus:~$ telnet sparta.naginet 80
Trying 10.128.0.21...
Connected to sparta.naginet.
Escape character is '^]'.

```

## How to do it...

We can create the service definition for `sparta.naginet` as follows:

1.  Change to the directory containing the file in which the `sparta.naginet` host is defined, and edit it as follows:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi sparta.naginet.cfg

    ```

2.  Add the following code snippet to the end of the file, substituting in the value of the host's `host_name` directive:

    ```
    define service {
     host_name              sparta.naginet
        service_description    HTTP
        check_command          check_http
        max_check_attempts     3
        check_interval         5
        retry_interval         1
        check_period           24x7
        notification_interval  60
        notification_period    24x7
        contacts               nagiosadmin
    }
    ```

3.  Restart the Nagios Core server:

    ```
    # /etc/init.d/nagios restart
    ```

If the server restarted successfully, the web interface should show a new service under the **Services** section, in **PENDING** state as the service awaits its first check:

![How to do it...](img/5566_01_04.jpg)

Within a few minutes, the service's state should change to **OK** once the check has run and succeeded with an **HTTP/1.1 200 OK** response, or similar:

![How to do it...](img/5566_01_05.jpg)

If the check had problems, perhaps because the HTTP daemon isn't running on the target server, then the check may show **CRITICAL** instead. This probably doesn't mean the configuration is broken; it more likely means the network or web server isn't working:

![How to do it...](img/5566_01_06.jpg)

## How it works...

The configuration we've added adds a simple service check definition for an existing host, to check up to three times whether the HTTP daemon on that host is responding to a simple **HTTP/1.1** request. If Nagios Core can't get a response to its check, then it will flag the state of the service as **CRITICAL**, and will try again up to two more times before sending a notification. The service will be visible in the Nagios Core web interface and we can check its state at any time. Nagios Core will continue testing the server on a regular basis and flagging whether the checks were successful or not.

It's important to note that the service is like a property of a particular host; we define a service to check for a specific host, in this case, the `sparta.naginet` web server. That's why it's important to get the definition for `host_name` right.

The directives we defined in the preceding configuration are as follows:

*   `host_name`: This references the host definition for which this service should apply. This will be the same as the `host_name` directive for the appropriate host.
*   `service_description`: This is a name for the service itself, something human-recognizable that will appear in alerts and in the web interface for the service. In this case, we've used `HTTP`.
*   `check_command`: This references the command that should be used to check the service's state. Here, we're referring to a command defined in Nagios Core's default configuration called `check_http`, which refers to a plugin of the same name in the Nagios Core Plugins set.
*   `max_check_attempts`: This defines the number of times Nagios Core should attempt to re-check the service after finding it in a state other than `OK`.
*   `check_interval`: This defines how long Nagios Core should wait between checks when the service is `OK`, or after the number of checks given in `max_check_attempts` has been exceeded.
*   `retry_interval`: This defines how long Nagios Core should wait between retrying checks after first finding them in a state other than `OK`.
*   `check_period`: This references the time period during which Nagios Core should run checks of the service. Here we've used the sensible `24x7` time period, as defined in Nagios Core's default configuration. Note that this can be different from `notification_period`; we can check the service's status without necessarily notifying a contact.
*   `notification_interval`: This defines how long Nagios Core should wait between re-sending notifications when a service is in a state other than `OK`.
*   `notification_period`: This references the time period during which Nagios Core should send notifications if it finds a host in a problem state. Here we've again used `24x7`, but for some less critical services it might be appropriate to use a time period such as `workhours`.

Note that we added the service definition in the same file as defining the host, and directly after it. We can actually place the definition anywhere we like, but this happens to be a good way of keeping things organized.

## There's more...

The service we've set up to monitor on `sparta.naginet` is an HTTP service, but that's just one of many possible services we could monitor on our network. Nagios Core defines many different commands for its core plugin set, such as `check_smtp`, `check_dns`, and so on. These commands, in turn, all point to programs that actually perform a check and return the results to the Nagios Core server to be dealt with. The important thing to take away from this is that a service can monitor pretty much anything, and there are hundreds of plugins available for common network monitoring checks available on the Nagios Exchange website: [http://exchange.nagios.org/](http://exchange.nagios.org/).

There are a great deal more possible directives for services, and in practice it's more likely for even simple setups that we'll want to extend a service template for our service. This allows us to define values that we might want for a number of services, such as how long they should be in a `CRITICAL` state before a notification event takes place and someone gets contacted to deal with the problem.

One such template that Nagios Core's default configuration defines is called `generic-service`, and we can use it as a basis for our new service by referring to it with the `use` keyword:

```
define service {
 use                    generic-service
    host_name              sparta.naginet
    service_description    HTTP
    check_command          check_http
}
```

This may work well for you, as there are a lot of very sensible default values set by the `generic-service` template, which makes things a lot easier. We can inspect these values by looking at the template's definition in `/usr/local/nagios/etc/objects/templates.cfg`. This is the same file that includes the `generic-host` definition that we may have used earlier.

## See also

*   The *Creating a new servicegroup* recipe in this chapter
*   The *Specifying how frequently to check a service* and *Scheduling downtime for a host or service* recipes in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*
*   The *Monitoring web services* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Creating a new e-mail contact

In this recipe, we'll create a new contact with which hosts and services can interact, chiefly to inform them of hosts or services changing states. We'll use the simplest example of setting up an e-mail contact, and configuring an existing host so that this person receives an e-mail message when Nagios Core's host checks fail and the host is apparently unreachable. In this instance, I'll make it e-mail me at `nagios@sanctum.geek.nz` whenever my host, `sparta.naginet`, goes from `DOWN` to `UP` state, or vice-versa.

## Getting ready

You should have a working Nagios Core 3.0 or greater server running, with a web interface and at least one host to check. If you need to do this first, see the *Creating a new network host* recipe in this chapter.

For this particular kind of contact, you'll also need to have a working SMTP daemon running on the monitoring server, such as **Exim** or **Postfix**. You should verify that you're able to send messages to the target address, and that they're successfully delivered to the mailserver you expect.

## How to do it...

We can add a simple new contact to the Nagios Core configuration as follows:

1.  Change to Nagios Core's object configuration directory; ideally it should contain a file that's devoted to contacts, such as `contacts.cfg` here, and edit that file:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi contacts.cfg

    ```

2.  Add the following contact definition to the end of the file, substituting your own values for the properties in bold as you need them:

    ```
    define contact {
     contact_name                   spartaadmin
     alias                          Administrator of sparta.naginet
     email                          nagios@sanctum.geek.nz
        host_notification_commands     notify-host-by-email
        host_notification_options      d,u,r
        host_notification_period       24x7
        service_notification_commands  notify-service-by-email
        service_notification_options   w,u,c,r
        service_notification_period    24x7
    }
    ```

3.  Edit the definition for the `sparta.naginet` host, and add or replace the definition for `contacts` for the appropriate host to our new `spartaadmin` contact:

    ```
    define host {
        host_name              sparta.naginet
        alias                  sparta
        address                10.128.0.21
        max_check_attempts     3
        check_period           24x7
        check_command          check-host-alive
     contacts               spartaadmin
        notification_interval  60
        notification_period    24x7
    }
    ```

4.  Restart the Nagios Core server:

    ```
    # /etc/init.d/nagios restart

    ```

With this done, the next time our host changes its state, we should receive messages similar to the following:

![How to do it...](img/5566_01_07.jpg)

When the host becomes available again, we should receive a recovery message similar to the following:

![How to do it...](img/5566_01_08.jpg)

If possible, it's worth testing this setup with a test host that we can safely bring down and then up again, to check that we receive the appropriate notifications.

## How it works...

This configuration adds a new contact to the Nagios Core configuration, and references it in one of the hosts as the appropriate contact to use when the host has problems.

We've defined the required directives for the contact, and a couple of others as follows:

*   `contact_name`: This defines a unique name for the contact, so that we can refer to it in host and service definitions, or anywhere else in the Nagios Core configuration.
*   `alias`: This defines a human-friendly name for the contact, perhaps a brief explanation of who the person or group is and/or for what they're responsible.
*   `email`: This defines the e-mail address of the contact, since we're going to be sending messages by e-mail.
*   `host_notification_commands`: This defines the command or commands to be run when a state change on a host prompts a notification for the contact. In this case, we're going to e-mail the contact the results with a predefined command called `notify-host-by-email`.
*   `host_notification_options`: This specifies the different kinds of host events for which this contact should be notified. Here, we're using `d,u,r`, which means that this contact will receive notifications for a host going `DOWN`, becoming `UNREACHABLE`, or coming back `UP`.
*   `host_notification_period`: This defines the time period during which this contact can be notified of any host events. If a host notification is generated and defined to be sent to this contact, but it falls outside this time period, then the notification will not be sent.
*   `service_notification_commands`: This defines the command or commands to be run when a state change on a service prompts a notification for this contact. In this case, we're going to e-mail the contact the results with a predefined command called `notify-service-by-email`.
*   `service_notification_options`: This specifies the different kinds of service events for which this contact should be notified. Here, we're using `w,u,c,r`, which means we want to receive notifications about the services entering the `WARNING`, `UNKNOWN`, or `CRITICAL` states, and also when they recover and go back to being in the `OK` state.
*   `service_notification_period`: This is the same as `host_notification_period`, except that this directive refers to notifications about services, and not hosts.

Note that we placed the definition for the contact in `contacts.cfg`, which is a reasonably sensible place. However, we can place the contact definition in any file that Nagios Core will read as part of its configuration; we can organize our hosts, services, and contacts any way we like. It helps to choose some sort of system, so we can easily identify where definitions are likely to be when we need to add, change, or remove them.

## There's more...

If we define a lot of contacts with similar options, it may be appropriate to have individual contacts extend contact templates, so that they can inherit those common settings. The QuickStart Nagios Core configuration includes such a template, called `generic-contact`. We can define our new contact as an extension of this template, as follows:

```
define contact {
 use    generic-contact
    alias  Administrator of sparta.naginet
    email  nagios@sanctum.geek.nz
}
```

To see the directives defined for `generic-contact`, you can inspect its definition in the `/usr/local/nagios/etc/objects/templates.cfg` file.

## See also

*   The *Creating a new contact group* recipe in this chapter
*   The *Automating contact rotation* and *Defining an escalation for repeated notifications* recipes in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Verifying configuration

In this recipe, we'll learn about the most basic step in debugging a Nagios Core configuration, which is to verify it. This is a very useful step to take before restarting the Nagios Core server to load an altered configuration, because it will warn us about possible problems. This is a good recipe to follow if you're not able to start the Nagios Core server at any point because of configuration problems, and instead get output similar to the following:

```
# /etc/init.d/nagios restart
Running configuration check... CONFIG ERROR!  Restart aborted.  Check your Nagios configuration.

```

## Getting ready

You should have a working Nagios Core 3.0 or better server running.

## How to do it...

We can verify the Nagios Core configuration as follows:

1.  Run the following command, substituting the path to the Nagios binary file and our primary `nagios.cfg` configuration file, if necessary:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

    ```

2.  If the output is very long, then it might be a good idea to pipe it through a pager program, such as `less`:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg | less

    ```

3.  Inspect the output and look for warnings and problems. Here's an example of part of the output we can expect, if our configuration is correct:![How to do it...](img/5566_01_09.jpg)

    If there's a problem of some sort, then we might see an output similar to the following, which is just an example of a possible error; my configuration is wrong because I tried to add a service for a host called `athens.naginet`, when I hadn't actually configured that host yet. So Nagios Core is quite right to yell at me:

    ![How to do it...](img/5566_01_10.jpg)

## How it works...

The configuration is parsed as though Nagios Core were about to start up, to check that the configuration all makes sense. It will run basic checks such as looking for syntax errors, and will also check things like having at least one host and service to monitor. Some of the things it reports are warnings, meaning that they're not necessarily problems; examples include hosts not having any services monitored, or not having any contacts defined.

This is the quickest way to get an idea of whether the Nagios Core configuration is sane and will work correctly. Whenever there's trouble restarting the Nagios Core server, it's a good idea to check the output of this command first. In fact, it's a good habit to check the configuration before restarting, particularly if we're unsure about the configuration changes, or if the monitoring server is checking something very important! This means if it turns out that our configuration is broken, then the Nagios Core daemon will keep running with the configuration from the point before we changed it, and we can fix things before we restart.

## There's more...

The program at `/usr/local/nagios/bin/nagios` is actually the same program that runs the Nagios Core server, but the `-v` part of the command is a switch for the program that verifies the configuration instead, and shows any problems with it. The second path is to the configuration file with which Nagios Core starts, which in turn imports the configuration files for objects, such as contact, host, and service definitions.

## See also

*   The *Writing debugging information to Nagios Core log file* recipe in [Chapter 10](ch10.html "Chapter 10. Security and Performance"), *Security and Performance*

# Creating a new hostgroup

In this recipe, we'll learn how to create a new hostgroup; in this case, we'll do this to group together two webservers. This is useful for having distinct groups of hosts that might have different properties, such as being monitored by different teams, or running different types of monitored services. It also allows us to view a group breakdown in the Nagios Core web interface, and to apply a single service to a whole group of hosts, rather than doing so individually. This means we can set up services for a new host simply by adding it to a group, rather than having to specify the configuration manually.

## Getting ready

You should have a working Nagios Core 3.0 or better server running, with a web interface.

You should also have at least two hosts that form a meaningful group; perhaps they're similar kinds of servers, such as webservers, or are monitored by the same team, or all at a physical location.

In this example, we have two webservers, `sparta.naginet` and `athens.naginet`, and we're going to add them to a group called `webservers`.

## How to do it...

We can add our new hostgroup `webservers` to the Nagios Core configuration as follows:

1.  Create a new file called `/usr/local/nagios/etc/objects/hostgroups.cfg`, if it doesn't already exist:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi hostgroups.cfg

    ```

2.  Add the following code into the new file, substituting the names in bold to suit your own layout:

    ```
    define hostgroup {
     hostgroup_name  webservers
     alias           Webservers with Greek names
    }
    ```

3.  Move a directory up, and then edit the `nagios.cfg` file:

    ```
    # cd ..
    # vi nagios.cfg

    ```

4.  Add the following line to the end of the file:

    ```
    cfg_file=/usr/local/nagios/etc/objects/hostgroups.cfg
    ```

5.  For each of the hosts we want to add to the group, find their definitions, and add a `hostgroups` directive to put them into the new hostgroup. In this case, our definitions for `sparta.naginet` and `athens.naginet` end up looking as follows:

    ```
    define host {
        use         linux-server
        host_name   sparta.naginet
        alias       sparta
        address     10.128.0.21
        hostgroups  webservers
    }
    define host {
        use         linux-server
        host_name   athens.naginet
        alias       athens
        address     10.128.0.22
        hostgroups  webservers
    }
    ```

6.  Restart Nagios:

    ```
    # /etc/init.d/nagios restart

    ```

We should now be able to visit the **Host Groups** section of the web interface, and see a new hostgroup with two members:

![How to do it...](img/5566_01_11.jpg)

## How it works...

The configuration we added includes a new file with a new hostgroup into the Nagios Core configuration, and inserts the appropriate hosts into the group. At the moment, all this is doing is creating a separate section in the web interface for us to get a quick overview of only the hosts in that particular group.

## There's more...

The way we've added hosts to groups is actually not the only way to do it. If we prefer, we can name the hosts for the group inside the group definition, using the `members` directive, so that we could have a code snippet similar to the following:

```
define hostgroup {
    hostgroup_name  webservers
    alias           Webservers with Greek names
    members         athens.naginet,sparta.naginet
}
```

This extends to allowing us to make a hostgroup that always includes every single host, if we find that useful:

```
define hostgroup {
    hostgroup_name  all
    alias           All hosts
 members         *
}
```

If we're going to use hostgroups extensively in our Nagios Core configuration, then we should use whichever method is going to be easiest for our configuration. We can use both, if necessary.

It's worth noting that a host can be in more than one group, and there is no limit on the number of groups we can declare, so we can afford to be quite liberal with how we group our hosts into useful categories. Examples could be organizing servers by function, manufacturer, or colocation customer, or routers by BGP or OSPF usage; it all depends on what kind of network we're monitoring.

## See also

*   The *Creating a new host* and *Running a service on all hosts in a group* recipes in this chapter
*   The *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Managing Configuration*

# Creating a new servicegroup

In this recipe, we'll create a new servicegroup. This allows us to make meaningful groups out of a set of arbitrary services, so that we can view the status of all those services in a separate part of the web administration area.

## Getting ready

You should have a working Nagios Core 3.0 or better server running, with web interface.

You should also have at least two services defined that form a meaningful group; perhaps they're similar kinds of services, such as mail services, or are monitored by the same team, or all on the same set of servers at a physical location.

In this example, we have three servers performing mail functions: `smtp.naginet`, `pop3.naginet`, and `imap.naginet`, running an SMTP, POP3, and IMAP daemon, respectively. All three of the hosts are set up in Nagios Core, and so are their services. We're going to add them into a new servicegroup called `mailservices`.

Here are the definitions of the hosts and services used in this example, so you can see how everything fits together:

```
define host {
    use                 linux-server
    host_name           smtp.naginet
    alias               smtp
    address             10.128.0.31
    hostgroups          webservers
}
    define service {
        use                  generic-service
        host_name            smtp.naginet
        service_description  SMTP
        check_command        check_smtp
    }
define host {
    use                 linux-server
    host_name           pop3.naginet
    alias               pop3
    address             10.128.0.32
    hostgroups          webservers
}
    define service {
        use                  generic-service
        host_name            pop3.naginet
        service_description  POP3
        check_command        check_pop
    }
define host {
    use                 linux-server
    host_name           imap.naginet
    alias               imap
    address             10.128.0.33
    hostgroups          webservers
}
    define service {
        use                  generic-service
        host_name            imap.naginet
        service_description  IMAP
        check_command        check_imap
    }
```

## How to do it...

We can add our new servicegroup with the following steps:

1.  Change to our Nagios Core configuration objects directory, and edit a new file called `servicegroups.cfg`:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi servicegroups.cfg

    ```

2.  Add the following definition to the new file, substituting the values in bold with your own values:

    ```
    define servicegroup {
     servicegroup_name  mailservices
     alias              Mail services
    }
    ```

3.  Move a directory up, and then edit the `nagios.cfg` file:

    ```
    # cd ..
    # vi nagios.cfg

    ```

4.  Add the following line to the end of the file:

    ```
    cfg_file=/usr/local/nagios/etc/objects/servicegroups.cfg
    ```

5.  For each of the services we want to add to the group, find their definitions and add a `servicegroups` directive to put them into the new servicegroup. The definitions may end up looking similar to the following code snippet:

    ```
    define service {
        use                  generic-service
        host_name            smtp.naginet
        service_description  SMTP
        check_command        check_smtp
     servicegroups        mailservices
    }
    define service {
        use                  generic-service
        host_name            pop3.naginet
        service_description  POP3
        check_command        check_pop
     servicegroups        mailservices
    }
    define service {
        use                  generic-service
        host_name            imap.naginet
        service_description  IMAP
        check_command        check_imap
     servicegroups        mailservices
    }
    ```

6.  Restart Nagios with the following command:

    ```
    # /etc/init.d/nagios restart

    ```

We should now be able to visit the **Service Groups** section of the web interface, and see a new servicegroup with three members:

![How to do it...](img/5566_01_12.jpg)

## How it works...

The configuration we added includes a new file with a new servicegroup into the Nagios Core configuration, and inserts the appropriate services into the group. This creates a separate section in the web interface for us to get a quick overview of only the services in that particular group.

## There's more...

The way we've added services to the groups is actually not the only way to do it. If we prefer, we can name the services (and their applicable hosts) for the group inside the group definition, using the `members` directive, so that we could have a code snippet similar to the following:

```
define servicegroup {
    servicegroup_name  mailservices
    alias              Mail services
 members            smtp.naginet,SMTP,pop3.naginet,POP3
}
```

Note that we need to specify both the host that the service is on, and then the services to monitor on it, comma-separated. The hostname comes first, and then the service.

This extends to allowing us to make a servicegroup that always includes every single service, if we find that useful:

```
define servicegroup {
    servicegroup_name  all
    alias              All services
 members            *
}
```

If we're going to be using servicegroup definitions extensively in our Nagios Core configuration, we should use whichever of the two methods to add services to groups that we think is going to be easiest for us to maintain.

It's worth noting that a service can be in more than one group, and there is no limit on the number of groups we can declare, so we can afford to be quite liberal with how we group our services into categories. Examples could be organising services by the appropriate contact for their notifications, for internal functions, or customer facing functions.

## See also

*   The *Creating a new service* and *Running a service on all hosts in a group* recipes in this chapter
*   The *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Managing Configuration*

# Creating a new contactgroup

In this recipe, we'll create a new contactgroup into which we can add our contacts. Like hostgroups and servicegroups, contactgroups mostly amount to convenient shortcuts. In this case, it allows us to define a contactgroup as the recipient of notifications for a host or service definition. This means that we could define a group `ops`, for example, and then even if people joined or left the group, we wouldn't need to change any definitions for the hosts or services.

## Getting ready

You should have a working Nagios Core 3.0 or better server running.

You should also have at least two contacts that form a meaningful group. In this case, we have two staff members, John Smith and Jane Doe, who are both a part of our network operations team. We want both of them to be notified for all the appropriate hosts and services, so we'll add them to a group called `ops`. Here are the definitions with which we're working:

```
define contact {
    use           generic-contact
    contact_name  john
    alias         John Smith
    email         john@naginet
}
define contact {
    use           generic-contact
    contact_name  jane
    alias         Jane Doe
    email         jane@naginet
}
```

## How to do it...

We can create our new `ops` contactgroup as follows:

1.  Change to our Nagios Core object configuration directory, and edit the `contacts.cfg` file:

    ```
    # cd /usr/local/nagios/etc
    # vi contacts.cfg

    ```

2.  Add the following definition to the file, substituting your own values in bold as appropriate:

    ```
    define contactgroup {
     contactgroup_name  ops
     alias              Network operators
    }
    ```

3.  For each of the contacts that we want to add to the group, find their definitions and add the `contactgroups` directive to them. The definitions will end up looking similar to the following code snippet:

    ```
    define contact {
        use            generic-contact
        contact_name   john
        alias          John Smith
        email          john@naginet
     contactgroups  ops
    }
    define contact {
        use            generic-contact
        contact_name   jane
        alias          Jane Doe
        email          jane@naginet
     contactgroups  ops
    }
    ```

4.  Restart the Nagios Core server:

    ```
    # /etc/init.d/nagios restart

    ```

## How it works...

With this group set up, we are now able to use it in the `contactgroups` directive for hosts and services, to define the contacts to which notifications should be sent. Notifications are sent to all the addresses in the group. This can replace the `contacts` directive where we individually name the contacts.

## There's more...

This means, for example, that instead of having a service definition similar to the following:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP
    check_command        check_http
 contacts             john,jane
}
```

We could use the following code snippet:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP
    check_command        check_http
 contact_groups       ops
}
```

If John Smith were to leave the operations team, then we could simply remove his contact definition, and nothing else would require changing; from then on, only Jane Doe would receive the service notifications. This method provides a layer of abstraction between contacts and the hosts and services for which they receive notifications.

## See also

*   The *Creating a new contact* recipe in this chapter
*   The *Automating contact rotation* recipe in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*
*   The *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Managing Configuration*

# Creating a new time period

In this recipe, we'll add a new time period definition to the Nagios Core configuration to allow us to set up monitoring for hosts and services only during weekdays. There's a default configuration defined as `workhours` that would almost suit us, except that it doesn't include the evenings. We'll make a new one from scratch, and we'll make another one to cover the weekends too.

## Getting ready

You should have a working Nagios Core 3.0 or better server running.

## How to do it...

We can set up our new time period, which we'll call `weekdays`, as follows:

1.  Change to our Nagios Core configuration objects directory, and edit the file called `timeperiods.cfg`:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi timeperiods.cfg

    ```

2.  Add the following definitions to the end of the file:

    ```
    define timeperiod {
        timeperiod_name  weekdays
        alias            Weekdays
        monday           00:00-24:00
        tuesday          00:00-24:00
        wednesday        00:00-24:00
        thursday         00:00-24:00
        friday           00:00-24:00
    }
    define timeperiod {
        timeperiod_name  weekends
        alias            Weekends
        saturday         00:00-24:00
        sunday           00:00-24:00
    }
    ```

3.  Restart the Nagios Core server:

    ```
    # /etc/init.d/nagios restart

    ```

## How it works...

In our host and service definitions, there are two directives, `check_period` and `notification_period`. These directives are used to define the times during which a host or service should be checked, and the times when notifications about them should be sent. The `24x7` and `workhours` periods are defined in the `timeperiods.cfg` file that we just edited, and are used in several of the examples and templates.

We've just added two more of these time periods, which we can now use in our definitions for hosts and services. The first is called `weekdays`, and corresponds to any time during a weekday; the second is called `weekends`, and corresponds to any time that's not a weekday. Note that in both cases, we specified the dates and times by naming each individual day, and the times to which they corresponded.

## There's more...

The definitions for dates are quite clever, and can be defined in a variety of ways. The following are all valid definitions for days and time periods:

*   `june 1 - july 15 00:00-24:00`: Any time from June 1st to July 15th, inclusive
*   `thursday -1 00:00-24:00`: Any time on the last Thursday of every month
*   `day 1 - 10 13:00-21:00`: From 1 PM to 9 PM on any day from the 1st of any month to the 10th of any month, inclusive

It's likely that the standard `24x7` and `workhours` definitions will be fine for day-to-day monitoring, maybe with a `weekdays` and `weekends` definition added. However, there may well come a time when we need a specific host or service monitored on an unusual schedule, particularly if we're debugging a specific problem that only manifests around a certain time, or have a lot of contacts to manage, or a complex on-call roster.

Note that Nagios Core can behave in unusual ways, particularly with uptime reporting, if the time periods for our monitoring of hosts and services don't add up to 24 hours. Ideally, we should check and notify all our hosts and services in at least some way around the clock, but dealing with the notifications in different ways depending on schedule; for example, we could page the systems administrators about a non-critical system during work hours, but just e-mail them when they're asleep!

## See also

*   The *Automating contact rotation*, *Configuring notification periods*, and *Configuring notification groups* recipes in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*

# Running a service on all hosts in a group

In this recipe, we'll create a new service, but instead of applying it to an existing host, we'll apply it to an existing hostgroup. In this case, we'll create a group called `webservers`. The steps for this are very similar to adding a service for just one host; only one directive is different.

## Getting ready

You should have a working Nagios Core 3.0 or better server running, with a web interface. You should be familiar with adding services to individual hosts.

You should also have at least one hostgroup defined, with at least one host in it; we'll use a group called `webservers`, with the hosts `sparta.naginet` and `athens.naginet` defined in it.

For reference, here is the hostgroup definition and the definitions for the two hosts in it:

```
define hostgroup {
    hostgroup_name  webservers
    alias           Webservers
}
define host {
    use         linux-server
    host_name   athens.naginet
    alias       athens
    address     10.128.0.22
    hostgroups  webservers
}
define host {
    use         linux-server
    host_name   sparta.naginet
    alias       sparta
    address     10.128.0.21
    hostgroups  webservers
}
```

## How to do it...

We can create the service definition for the webservers group as follows:

1.  Change to the directory containing the file in which the webservers hostgroup is defined, and edit it:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi hostgroups.cfg

    ```

2.  Add the following code snippet just after the hostgroup definition. Change the lines in bold to suit your own template and hostgroup names:

    ```
    define service {
     use                    generic-service
     hostgroup_name         webservers
        service_description    HTTP
        check_command          check_http
    }
    ```

3.  Restart the Nagios Core server:

    ```
    # /etc/init.d/nagios restart

    ```

It's important to note that if we are already monitoring those hosts with a per-host service of the same name, then we will need to remove those definitions as well; Nagios Core may not start if a service of the same description is already defined on the same host.

## How it works...

Adding a service to a hostgroup works in exactly the same way as adding it to an individual host, except that it only requires one definition, which is then individually applied to all the hosts in the group. This means it's a very good way to keep a Nagios Core configuration tidier. If we have a group of 50 different web servers in it and we need to monitor their HTTP services on the same basis for each one of them, then we don't need to create 50 service definitions; we can just create one for their hostgroup, which amounts to a smaller and more easily updated configuration.

## There's more...

Like the `host_name` directive for services, the `hostgroup_name` directive can actually have several hostgroups defined, separated by commas. This means that we can apply the same service to not just one group, but several. For services that we would want to run on several different groups (for example, basic PING monitoring) this can amount to a much more flexible configuration.

## See also

*   The *Creating a new service and Creating a new hostgroup* recipe in this chapter
*   The *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Managing Configuration*