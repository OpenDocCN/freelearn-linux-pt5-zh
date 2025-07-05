# Chapter 3. Working with Checks and States

In this chapter, we will cover the following recipes:

*   Specifying how frequently to check a host or service
*   Changing thresholds for PING RTT and packet loss
*   Changing thresholds for disk usage
*   Scheduling downtime for a host or service
*   Managing brief outages with flapping
*   Adjusting flapping percentage thresholds for a service

# Introduction

Once hosts and services are configured in Nagios Core, its behavior is primarily dictated by the checks it makes to ensure that hosts and services are operating as expected, and the state it concludes these hosts and services must be in as a result of those checks.

How often it's appropriate to check hosts and services, and on what basis it's appropriate to flag a host or service as having problems, depends very much on the nature of the service and the importance of it running all the time. If a host on the other side of the world is being checked with PING, and during busy periods its round trip time is over 100ms, then this may not actually be a cause for concern at all, and perhaps not something to even flag a `WARNING` state over, let alone a `CRITICAL` one.

However, if the same host were on the local network where it would be appropriate to expect round trip times of less than 10ms, then a round trip time of more than 100ms could well be considered a grave cause for concern, signaling a packet storm or other problem with the local network, and we would want to notify the appropriate administrators immediately. Similarly, for hosts such as web servers, we may not be concerned by a response time of more than a second for a page on a busy budget shared web host for customers. But if the response time for the corporate website or a dedicated colocation customer was getting that bad, it might well be something to notify the web server administrator about.

Hosts and services are therefore not all created equal. Nagios Core provides several ways to define behaviors with more precision, as follows:

*   How often a host or service should be checked with its appropriate `check_command` plugin
*   How bad a check's results have to be before a `WARNING` or `CRITICAL` problem is flagged, if at all
*   Defining a downtime period for a host or service, so that Nagios Core knows not to expect it to operate during a specified period of time, often for upgrades or other maintenance
*   Whether to automatically tolerate flapping, or hosts and services seeming to go up and down a lot

This chapter will use some common instances of problems with the preceding behaviors to give examples showing how to configure them.

# Specifying how frequently to check a host or service

In this recipe, we'll configure a very important host to be checked every three minutes, and if Nagios Core finds it is `DOWN` as a result of the check failing, it will check again after a minute before it sends a notification about the state to its defined contact. We'll do this by customizing the definition of an existing host.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file.

You should also understand the basics of commands and plugins, in particular the meaning of the `check_command` directive. These are covered in the recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*.

## How to do it...

We can customize the check frequency for a host as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`. If you've put the definition of your host in a different file, then move to its directory instead:

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing your host definition, and find the definition within the file:

    ```
    # vi sparta.naginet.cfg

    ```

    The host definition may look similar to the following:

    ```
    define host {
        use                 linux-server
        host_name           sparta.naginet
        alias               sparta
        address             10.128.0.21
    }
    ```

3.  Add or edit the value of the `check_interval` directive to `3`:

    ```
    define host {
        use                 linux-server
        host_name           sparta.naginet
        alias               sparta
        address             10.128.0.21
     check_interval      3
    }
    ```

4.  Add or edit the value of the `retry_interval` directive to `1`:

    ```
    define host {
        use                 linux-server
        host_name           sparta.naginet
        alias               sparta
        address             10.128.0.21
        check_interval      3
     retry_interval      1
    }
    ```

5.  Add or edit the value of `max_check_attempts` to `2`:

    ```
    define host {
        use                 linux-server
        host_name           sparta.naginet
        alias               sparta
        address             10.128.0.21
        check_interval      3
        retry_interval      1
     max_check_attempts  2
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core will run the relevant `check_command` plugin (probably something like `check-host-alive`) against this host every three minutes; if it fails, it will flag the host as down, check again one minute later, and only then send a notification to its defined contact if the second check fails too.

## How it works...

The preceding configuration changed three properties of the host object type to effect the changes we needed:

*   `check_interval`: This defines how long to wait between successive checks of the host under normal conditions. We set this to `3`, or three minutes.
*   `retry_interval`: This defines how long to wait between follow-up checks of the host after first finding problems with it. We set this to `1`, or one minute.
*   `max_check_attempts`: This defines how many checks in total should be run before a notification is sent. We set this to `2` for two checks. This means that after the first failed check is run, Nagios Core will run another check a minute later, and will only send a notification if this check fails as well. If two checks have been run and the host is still in a problem state, it will go from a `SOFT` state to a `HARD` state.

Note that setting these directives in a host that derives from a template, as is the case with our example, will override any of the same directives in the template.

## There's more...

It's important to note that we can also define the units used by the `check_interval` and `retry_interval` commands. They only use minutes by default, checking the `interval_length` setting that's normally defined in the root configuration file for Nagios Core, by default `/usr/local/nagios/etc/nagios.cfg`:

```
interval_length=60
```

If we wanted to specify these periods in seconds instead, we could set this value to `1` instead of `60`:

```
interval_length=1
```

This would allow us, for example, to set `check_interval` to `15`, to check a host every 15 seconds. Note that if we have a lot of hosts with such a tight checking schedule, it might overburden the Nagios Core process, particularly if the checks take a long time to complete.

Don't forget that changing these properties for a large number of hosts can be tedious, so if it's necessary to set these directives to some common value for more than a few hosts, then it may be appropriate to set the values in a host template and then have these hosts inherit from it. See the *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Configuration Management* for details on this. Note that the same three directives also work for service declarations, and have the same meaning. We could define the same notification behavior for a service on `sparta.naginet` with a declaration similar to the following:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP
    check_command        check_http
    address              10.128.0.21
 check_interval       3
 retry_interval       1
 max_check_attempts   2
}
```

## See also

*   The *Scheduling downtime for a host* recipe in this chapter
*   The *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Configuration Management*

# Changing thresholds for PING RTT and packet loss

In this recipe, we'll set up a service for a host that monitors PING, and take a look at how to adjust the thresholds for the `WARNING` and `CRITICAL` states, done using command arguments. We'll accomplish this by setting up a service for an existing host that's already being checked with a `check_command` plugin, such as `check-host-alive`. Our service will be used to monitor not whether the host is completely `DOWN`, but whether it's responding to PING requests within a reasonable period of time.

This could be useful to notify and assist in diagnosing problems with the actual connectivity of a service or host.

This recipe will therefore serve as a good demonstration of the concepts of supplying arguments to a command, and adjusting the `WARNING` and `CRITICAL` thresholds for a particular service.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already, and using a `check_command` plugin of `check-host-alive`. We'll use the example of `sparta.naginet`, a host defined in its own file.

You should also understand the basics of how hosts and services fit together in a Nagios Core configuration, and be familiar with the use of commands and plugins via the `check_command` directive.

## How to do it...

We can add our PING service to the existing host with custom round trip time and packet loss thresholds as follows:

1.  Change to the objects configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`. If you've put the definition of your host in a different file, then move to its directory instead:

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing your host definition:

    ```
    # vi sparta.naginet.cfg

    ```

3.  Add the following definition to the end of the file. Of most interest here is the value for the `check_command` directive:

    ```
    define service {
        use                  generic-service
        host_name            sparta.naginet
        service_description  PING
     check_command        check_ping!100,20%!200,40%
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core will not only run a host check of `check-host-alive` against your original host to ensure that it's up, but it will also run a more stringent check of the PING responses from the machine as a service to check that it's adequately responsive:

*   If the **Round Trip Time** (**RTT**) of the PING response is greater than 100ms (but less than 200ms), Nagios Core will flag a `WARNING` state.
*   If the RTT of the PING response is greater than 200ms, Nagios Core will flag a `CRITICAL` state.
*   If more than 20 percent (but less than 40 percent) of the PING requests receive no response, Nagios Core will flag a `WARNING` state.
*   If more than 40 percent of the PING requests receive no response, Nagios Core will flag a `CRITICAL` state.

In both cases, a notification will be sent to the service's defined contacts if configured to do so.

Otherwise, this service works the same way as any other service, and appears in the web interface.

## How it works...

The configuration we added for our existing host creates a new service with a `service_description` of PING. For `check_command`, we use the `check_ping` command, which uses the plugin of the same name. The interesting part here is what follows the `check_command` definition: the string `!100,20%!200,40%`.

In Nagios Core, a `!` character is used as a separator for arguments that should be passed to the command. In the case of `check_ping`, the first argument defines thresholds, or conditions that, if met, should make Nagios Core flag a `WARNING` state for the service. Similarly, the second argument defines the thresholds for a `CRITICAL` state.

Each of the two arguments are comprised of two comma-separated terms: the first number is the threshold for the RTT of the PING request and its response that should trigger a state, and the second number is the percentage of packet loss that should be tolerated before raising the same state.

This pattern of arguments is specific to `check_ping`; they would not work for other commands such as `check_http`.

## There's more...

If we want to look in a bit more detail at how these arguments are applied, we can inspect the command definition for `check_ping`. By default, this is in the `/usr/local/nagios/etc/objects/commands.cfg` file, and looks similar to the following:

```
define command {
    command_name  check_ping
    command_line  $USER1$/check_ping -H $HOSTADDRESS$ -w $ARG1$ -c $ARG2$ -p 5
}
```

In the value for `command_line`, four macros are used:

*   `$USER1$`: This expands to `/usr/local/nagios/libexec`, or the directory in which the Nagios Core plugins are normally kept, including `check_ping`.
*   `$HOSTADDRESS$`: This expands to the hostname for the host or service definition in which the command is used. In this case, it expands to `10.128.0.21`, the value of the `address` directive for the `sparta.naginet` host.
*   `$ARG1$`: This expands to the value given for the first argument of the command, in our recipe's case, the string `100,20%`.
*   `$ARG2$`: This expands to the value given for the second argument of the command; in our recipe's case, the string `200,40%`.

The complete command-line call for our specific check with all these substitutions made would therefore look similar to the following:

```
/usr/local/nagios/libexec/check_ping -H 10.128.0.21 -w 100,20% -c 200,40% -p 5

```

This command line makes use of four parameters of the `check_ping` program:

*   `-H`: This specifies the address of the host to check
*   `-w`: This specifies the thresholds for raising a `WARNING` state
*   `-c`: This specifies the thresholds for raising a `CRITICAL` state
*   `-p`: This specifies the number of PING requests to send

We can run this directly from the command line on the Nagios Core server to see what the results of the check might be:

```
# /usr/local/nagios/libexec/check_ping -H 10.128.0.21 -w 100,20% -c 200,40% -p 5

```

The preceding command yields an output including the `OK` result of the check, and also some performance data, as follows:

```
PING OK - Packet loss = 0%, RTA = 0.17 ms|rta=0.174000ms;100.000000;200.000000;0.000000 pl=0%;5;10;0

```

The arguments specified in the command are therefore used to customize the behavior of `check_command` for the particular host or service being edited.

## See also

*   The *Changing thresholds for disk usage* recipe in this chapter
*   The *Creating a new service* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Customizing an existing command* and *Creating a new command* recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Monitoring PING for any host* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Changing thresholds for disk usage

In this recipe, we'll configure the Nagios Core server to check its own disk usage, and to flag a `WARNING` or `CRITICAL` state depending on how little free space is left on the disk. We'll accomplish this by adding a new service to the already defined `localhost` called `DISK`, which will run the `check_local_disk` command to examine the state of mounted volumes on the server.

Because burgeoning disk usage can creep up on any system administrator, and because of the dire effect it can have when a disk suddenly fills completely without any warning, this is amongst the more important things to monitor in any given network.

For simplicity, we'll demonstrate this only for the monitoring server itself, as a host called `localhost` on `127.0.0.1`. This is because the `check_disk` plugin can't directly check the disk usage of a remote server over a network. However, the principles discussed here could be adapted to running the check on a remote server using `check_nrpe`. The use of NRPE is discussed in all the recipes in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution"), *Enabling Remote Execution*.

## Getting ready

You should have a Nagios Core 3.0 or newer server with a definition for `localhost`, so that the monitoring host is able to check itself. A host definition for `localhost` is included in the sample configuration in `/usr/local/nagios/etc/objects/localhost.cfg`. You should also understand the basics of how hosts and services fit together in a Nagios Core configuration, and be familiar with the use of commands and plugins via the `check_command` directive.

We'll use the example of `olympus.naginet` as our Nagios Core server checking itself, with one block device on one disk, with its device file at `/dev/sda1`.

## How to do it...

We can add our `DISK` service to the existing host with custom usage thresholds as follows:

1.  Change to the objects configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, move to its directory instead:

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing your host definition:

    ```
    # vi localhost.cfg

    ```

3.  Add the following definition to the end of the file. Of most interest here is the value for the `check_command` directive:

    ```
    define service {
        use                  local-service
        host_name            localhost
        service_description  DISK
     check_command        check_local_disk!10%!5%!/dev/sda1
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a new service is created for `localhost` that checks the disk usage on `/dev/sda1`, and flags a `WARNING` state for the service if the free space is below 10 percent, and a `CRITICAL` state if it is below 5 percent.

In both cases, a notification will be sent to the service's defined contacts, if configured to do so.

## How it works

The configuration we added for our existing host creates a new service with a `service_description` of `DISK`. For `check_command`, we use the `check_local_disk` command, which in turn uses the `check_disk` plugin to check the local machine's disks. The interesting part here is what follows the `check_local_disk` definition: the string `!10%!5%!/dev/sda1`.

In Nagios Core, a `!` character is used as a separator for arguments that should be passed to the command. In the case of `check_local_disk`, the first two arguments define thresholds, or conditions that, if met, should make Nagios Core flag a `WARNING` state (first argument, 10 percent) or a `CRITICAL` state (second argument, 5 percent) for the service. The third argument defines the device name of the disk to check, `/dev/sda1`.

## There's more...

If we want to look in a bit more detail at how these arguments are applied, we can inspect the command definition for `check_local_disk`. By default, this is in the file `/usr/local/nagios/etc/objects/commands.cfg`, and looks similar to the following:

```
define command {
    command_name  check_local_disk
    command_line  $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
}
```

In this case, `command_name` and the name of the plugin used in the `command_line` are not the same.

In the value for `command_line`, four macros are used:

*   `$USER1$`: This expands to `/usr/local/nagios/libexec`, or the directory in which the Nagios Core plugins are normally kept, including `check_disk`.
*   `$ARG1$`: This expands to the value given for the first argument of the command; in this case, the string `10%`.
*   `$ARG2$`: This expands to the value given for the second argument of the command; in this case, the string `5%`.
*   `$ARG3$`: This expands to the value given for the third argument of the command; in this case, the string `/dev/sda1`.

The complete command-line call for our specific check with all these substitutions made would therefore be as follows:

```
/usr/local/nagios/libexec/check_disk -w 10% -c 5% -p /dev/sda1

```

This command line makes use of three parameters of the `check_disk` program:

*   `-w`: This specifies the thresholds for raising a `WARNING` state
*   `-c`: This specifies the thresholds for raising a `CRITICAL` state
*   `-p`: This specifies the device file for the disk to check

We can run this directly from the command line on the Nagios Core server to see what the results of the check might be:

```
# /usr/local/nagios/libexec/check_disk -w 10% -c 5% -p /dev/sda1

```

The output includes both the `OK` result of the check, and also some performance data:

```
DISK OK - free space: / 2575 MB (71% inode=78%);| /=1044MB;3432;3051;0;3814

```

## See also

*   The *Changing thresholds for disk usage* recipe in this chapter
*   The *Creating a new service* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Customizing an existing command* and *Creating a new command* recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Monitoring PING for any host* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Scheduling downtime for a host or service

In this recipe, we'll learn how to schedule downtime for a host or service in Nagios Core. This is useful for elegantly suppressing notifications for some predictable period of time; a very good example is when servers require downtime to be upgraded, or to have their hardware checked.

In this example, we'll demonstrate scheduling downtime for a host named `sparta.naginet`, and we'll examine the changes it makes in the web interface.

## Getting ready

You should have a Nagios Core 3.0 or newer server with a definition for at least one host and at least one service, and some idea of when you would like your downtime to be scheduled. You should also have a working web interface, per the QuickStart installation of Nagios Core 3.0.

You should also have Nagios Core configured to process external commands, and have given your web interface user the permissions to apply them. If you are logging in as the `nagiosadmin` user per the recommended quick start guide, then you can check this is the case with the following directive in `/usr/local/nagios/etc/nagios.cfg`:

```
check_external_commands=1
```

Permissions to submit external commands from the web interface are defined in `/usr/local/nagios/etc/cgi.cfg`; check that your username is included in these directives:

```
authorized_for_all_service_commands=nagiosadmin
authorized_for_all_host_commands=nagiosadmin
```

If you have followed the Nagios Core QuickStart guides, then you will probably find this is already working: [http://nagios.sourceforge.net/docs/3_0/quickstart.html](http://nagios.sourceforge.net/docs/3_0/quickstart.html).

## How to do it...

We can set up a fixed period of scheduled downtime for our host and service as follows:

1.  Log into the web interface for Nagios Core.
2.  Click **Hosts** in the left menu:![How to do it...](img/5566_03_01.jpg)
3.  Click on the host's name in the table that comes up, to view the details for that host:![How to do it...](img/5566_03_02.jpg)
4.  Click on **Schedule downtime for this host** in the **Host Commands** menu:![How to do it...](img/5566_03_03.jpg)
5.  Fill out the fields in the resulting form, including the following details:

    *   **Host Name**: The name of the host for which you're scheduling downtime. This should have been filled out for you.
    *   **Author**: Your name, for records of who scheduled the downtime. This may be greyed out and just say **Nagios Admin**; that's fine.
    *   **Comment**: Some comment explaining the reason for the downtime.
    *   **Start Time**: The time at which the scheduled downtime should begin, and when the state notifications end.
    *   **End Time**: The time at which the scheduled downtime should end, and when the state notifications resume.

    ![How to do it...](img/5566_03_04.jpg)

    In this case, our downtime will be from 8:00 PM to 9:00 PM on the 15th of June, 2012\.

6.  Click on **Commit** to submit the downtime definition, and then **Done** in the screen that follows.

With this done, we can safely bring the `sparta.naginet` host down between the nominated times, and any notifications for the host and any of its services will be suppressed until the downtime is over.

Note that restarting Nagios Core is not required for this step, as it usually would be for changes made to Nagios Core's configuration files. The change is done "on the fly".

Note also that comments now appear in the detailed information for both the host and service, defining the downtime and including the reason specified for it.

## How it works...

The preceding steps nominate a period of downtime for both the `sparta.naginet` server and all of its services. This accomplishes two things:

*   It suppresses all notifications (whether e-mails or anything else) for the host or service for the appropriate time period, including `RECOVERY` notifications. The only exceptions are the `DOWNTIMESTART` and `DOWNTIMEEND` notifications.
*   It adds a comment to the host or service showing the scheduled downtime, for the benefit of anyone else who might be using the web interface.

Nagios Core keeps track of any downtime defined for all the hosts and services, and prevents the notifications it would normally send out during that time. Note that it will still run its checks and record the state of both the hosts and services even during downtime. All that is suppressed are the notifications, and not the actual checks.

## There's more...

Note that the downtime for individual services can be applied in much the same way, by clicking on **Schedule downtime for this service** in the web interface, under **Service Commands**.

What was defined in this recipe was a method for defining fixed downtime, where we know ahead of time when the host or its services are likely to be unavailable. If we don't actually know what time the unavailability will start, but we do know how long it's likely to last, then we can define a period of flexible downtime. This means that the downtime can start any time within the nominated period, and will last for the length of time we specify from that point.

A notification event is also fired when the host or service begins the downtime, called `DOWNTIMESTART`, and another when the downtime ends, called `DOWNTIMEEND`. This may be a useful notification to send to the relevant contact or contact group if they'd like to be notified when this happens. This can be arranged by ensuring that the host or service is configured to send these messages, by including the `s` flag in the `notification_options` directive for both hosts and services, and correspondingly in the contact definition:

```
notification_options  d,u,r,f,s
```

## See also

*   The *Managing brief outages with flapping* and *Adjusting flapping percentage thresholds for a service* recipes in this chapter
*   The *Specifying which states to be notified about* and *Tolerating a certain number of failed checks* recipes in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*
*   The *Adding comments on hosts or services in web interface* recipe in [Chapter 7](ch07.html "Chapter 7. Using the Web Interface"), *Working with the Web Interface*

# Managing brief outages with flapping

In this recipe, we'll learn how to use Nagios Core's state flapping detection and handling to avoid sending excessive notifications when a host or service changes its state too frequently. This is useful in circumstances where a host or service is changing between `OK` to `WARNING` to `CRITICAL` states too frequently within the last 21 checks. If the percentage of state changes is too high, Nagios Core will suppress further notifications and add an icon and comment to the host or service showing that it is flapping.

Flap detection is normally enabled in the QuickStart configuration for Nagios Core, and is part of the sample `generic-host` host template and the `generic-service` service template. It's therefore likely that it's already enabled on most servers, and we only need to check that it's still working.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host and one service configured already. You should also have access to a working web interface for the Nagios Core server. It would be helpful if you are monitoring a test service that you can bring up and down to trigger the flap detection to test it; an unused webserver might be good for this.

You should be familiar with the way hosts and services change state as a result of their checks and the different states corresponding to hosts and services in order to understand the basics of how flap detection works.

## How to do it...

We can check whether or not flap detection is enabled for our Nagios Core server, our hosts, and our services as follows:

1.  Change to the configuration directory for Nagios Core. The default is `/usr/local/nagios/etc`.

    ```
    # cd /usr/local/nagios/etc

    ```

2.  Edit the `nagios.cfg` file.

    ```
    # vi nagios.cfg

    ```

3.  Look for an existing definition for the `enable_flap_detection` directive, and check that it is set to `1`:

    ```
    enable_flap_detection=1
    ```

4.  If this was not set to `1`, then after we've changed it, we will probably also need to at least temporarily disable the `use_retained_program_state` directive in the same file:

    ```
    use_retained_program_state=0
    ```

5.  Edit the file for our particular hosts and or services. We should check that at least one of the following is the case:

    *   The host or service inherits from a template that has the `enable_flap_detection` directive set to `1`. For example, both the `generic-host` and `generic-service` templates defined by default in `/usr/local/nagios/etc/objects/templates.cfg` do this.
    *   The host or service itself has the `enable_flap_detection` directive set to `1` in its own definition.

        In the latter case, the configuration for the host or service might look similar to the following code snippet:

        ```
        define host {
            ...
         flap_detection_enabled 1
        }
        define service {
            ...
         flap_detection_enabled 1
        }
        ```

6.  If any of the preceding configuration was changed, validate the new configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

7.  Check that for the hosts or services for which flap detection is wanted, the word **ENABLED** appears in the details for that host or service:![How to do it...](img/5566_03_09.jpg)

With this done, if a host or service changes its state too frequently within 21 checks, then it will be flagged as flapping, and will appear with a custom icon in views of that host or service:

![How to do it...](img/5566_03_10.jpg)

A comment is also placed on the host or service explaining what has happened for the benefit of anyone viewing the host or service in the web interface, who might perhaps be wondering why notifications have stopped:

![How to do it...](img/5566_03_11.jpg)

There will also be an indicator on the details for the host or service defining whether the host or service is flapping:

![How to do it...](img/5566_03_12.jpg)

For hosts or services that don't have flap detection enabled, this particular field simply reads **N/A**, and the **Flap Detection** field below it will show as **DISABLED**.

## How it works...

The logic behind determining flap detection is actually quite complex. For our purposes, it suffices to explain flap detection as being based on whether a host or service has changed state within its last 21 checks too often—with the thresholds usually expressed as a percentage.

This is discussed in great detail including the formulae that are used to determine flapping state in the Nagios Core 3.0 documentation, available online at the following URL:

[http://nagios.sourceforge.net/docs/3_0/flapping.html](http://nagios.sourceforge.net/docs/3_0/flapping.html)

## There's more...

A common cause of flapping is that checks are too stringent. As an example, if you are checking a shared web server's response time is less than 50ms while the server is busy, checks might pass and fail without actually giving an accurate reflection of whether the service is doing its job. In this case, it would be appropriate to loosen the thresholds of the service by increasing its percentage thresholds, so that it isn't quite so ready to flag a `WARNING` or `CRITICAL` state over things that aren't actually very worrisome. Flap detection can help diagnose these sorts of cases.

We can also enable or disable flap detection for a host via the web interface; in the details screen for both hosts and services, a menu item is available under **Host Commands** labeled **Enable/Disable flap detection for this host**, and under **Service Commands** there's another labeled **Enable/Disable flap detection for this service**.

These may be useful when we want to turn flap detection on or off for a particular host or service temporarily, perhaps because under certain circumstances it is or is not appropriate to use the feature. For permanent setup and for clarity, it would be best to include it explicitly in the configuration as shown in the recipe.

## See also

*   The *Adjusting flapping percentage thresholds for a service* recipe in this chapter
*   The *Tolerating a certain number of failed checks* recipe in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring* *Notifications*
*   The *Adding comments on hosts or services in web interface* recipe in [Chapter 7](ch07.html "Chapter 7. Using the Web Interface"), *Working with the Web Interface*

# Adjusting flapping percentage thresholds for a service

In this recipe, we'll learn how to adjust the percentage thresholds for host or service flap detection. This means that we can adjust how frequently a host or service has to change state within its last 21 checks, before Nagios Core will conclude that it is flapping, and suppress notifications until its state becomes stable again.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host and one service configured already. You should also have access to a working web interface for the Nagios Core server.

You should be familiar with the way hosts and services change state as a result of their checks and the different states corresponding to hosts and services, to understand the basics of how flap detection works. Flap detection should also already be enabled and working for the appropriate hosts and services.

## How to do it...

We can adjust the thresholds for flap detection for a specific host or service as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the host or service for which we want to set the thresholds:

    ```
    # vi sparta.naginet.cfg

    ```

3.  Within the host or service definition, set the `low_flap_threshold` and/or `high_flap_threshold` values to appropriate percentages:

    ```
    define host {
        ...
     high_flap_threshold 50.0
     low_flap_threshold 25.0
    }
    define service {
        ...
     high_flap_threshold 45.0
     low_flap_threshold 20.0
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the flapping thresholds for the host or service should be changed appropriately for future checks.

## How it works...

The preceding configuration changes include the following directives:

*   `high_flap_threshold`: A host or service that is changing state by a certain percentage of the time must exceed this percentage threshold, before it is determined to be flapping.
*   `low_flap_threshold`: If a host or service is already in the flapping state, then its state change percentage must fall below this threshold before the flapping state will end.

For a detailed breakdown of how the state change percentage is calculated, see the Nagios Core 3.0 documentation online at the following URL:

[http://nagios.sourceforge.net/docs/3_0/flapping.html](http://nagios.sourceforge.net/docs/3_0/flapping.html)

## There's more...

If appropriate, we can also set a global default for hosts and services' flap thresholds with the following directives in `/usr/local/nagios/etc/nagios.cfg`. The following values are examples:

```
high_host_flap_threshold=50.0
low_host_flap_threshold=25.0
high_service_flap_threshold=45.0
low_service_flap_threshold=20.0
```

These values correspond to the percentages of state change, in just the same way that the per-host and per-service configurations do.

Note that there are separate directives for hosts and services in this case. These values are also overridden if you specify thresholds for a particular service or host, as we did in the recipe.

## See also

*   The *Managing brief outages with flapping* recipe in this chapter