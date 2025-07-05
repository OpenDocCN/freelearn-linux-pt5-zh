# Chapter 11. Automating and Extending Nagios Core

In this chapter, we will cover the following recipes:

*   Allowing and submitting passive checks
*   Submitting passive checks from a remote host with NSCA
*   Submitting passive checks in response to SNMP traps
*   Setting up an event handler script
*   Tracking host and service states with Nagiosgraph
*   Reading status into a MySQL database with NDOUtils
*   Writing customized Nagios Core reports
*   Getting extra visualizations with NagVis

# Introduction

In addition to being useful as a standalone monitoring framework, Nagios Core has a modular design that allows both interaction with and extension by other programs and tools, predominantly using its external command file for controlling the behavior of the server.

One of the most useful ways of interacting with the Nagios Core server in this way is through the use of passive checks: submitting check results to the server directly, rather than as the result of the server's own active checks.

The simplest application of the idea of passive checks is for monitoring some process that might take an indeterminate amount of time to run, and hence resists active checking; instead of the service making active checks of its own, it accepts a check result submitted by another application, perhaps something like a backup script after it has completed its run. These can be sent and accepted via an addon called the **Nagios Service Check Acceptor** (**NSCA**). Similarly, just as plugins and notifications are actually scripted calls to external commands, such as `check_http`, and `mail`, event handlers can be configured to run a specified command every time a host or service changes state. This can be used for the supplementary recording of change state data, or automated attempts at actively resolving the problem, such as restarting a remote server. We also saw event handlers used in the *Setting up a redundant monitoring host* recipe in [Chapter 10](ch10.html "Chapter 10. Security and Performance"), *Security and Performance*.

Finally, this chapter also includes installation procedures and discussion of a few of the more popular extensions to Nagios Core:

*   **Nagiosgraph**: This is an advanced web-based graphing solution for Nagios Core, graphing both server performance and host and service status metrics.
*   **NagVis**: This is an advanced web-based visualization extension for Nagios Core data, especially well-suited for administrators who need something more comprehensive than Nagios Core's built-in networking mapping.
*   **NDOUtils**: This applies the translations of Nagios Core data into a standard database system such as MySQL; very useful for performing advanced queries of the Nagios Core data for custom systems such as monitoring displays, or logging with change control systems.

We'll discuss NDOUtils, perhaps the most versatile Nagios Core extension of all, in two separate recipes; first by showing how to install it, and then some ideas on how to apply it to make custom Nagios Core reporting applications of our own, in the form of a **CLI report** written in **Perl**, and an **RSS feed** written in **PHP**.

# Allowing and submitting passive checks

In this recipe, we'll learn how to configure Nagios Core to accept passive checks for a service. This allows both users and external applications to directly submit the results of checks to Nagios Core, rather than having the application seek them out itself through polling with active checks, performed via plugins such as `check_http` or `check_ping`.

We'll show one simple example of a passive check, flagging a service called `BACKUP` for an existing host. We'll show how to do this via the web interface, which is very easy, and via the external commands file, which is slightly more complex but much more flexible and open to automation.

The idea is that when a user or process receives confirmation that the backup process on a host has completed correctly, they are able to supply a check result of `OK` directly to the service, without Nagios Core needing to poll anything itself.

## Getting ready

You should be running a Nagios Core 3.0 or newer server. You should also already have a host configured for which you want to define a service that will accept passive checks. In this example, we'll use the host `ithaca.naginet`, which might be defined as follows:

```
define host {
    use        linux-server
    host_name  ithaca.naginet
    alias      ithaca
    address    10.128.0.21
}
```

You will also need a working Nagios Core web interface to check that passive checks are enabled, and to try out the recipe's method of submitting passive checks.

The recipe will be in two parts: enabling and configuring the service for passive checks only, and actually submitting a passive check via the web interface. In the *There's more...* section, we'll show how to submit a check result via the external commands file, which is a little more complicated, but allows advanced automation behavior.

## How to do it...

We can define a new `BACKUP` service that accepts passive checks only, as follows:

1.  Log in to the web interface and ensure that passive checks are enabled. The **Tactical Overview** section shows a panel for it near the bottom. Check that it's green:![How to do it...](img/5566_11_01.jpg)

    If it's not green, you should be able to enable the checks again by clicking on the **Disabled** bar. In this case, you should also check the `/usr/local/nagios/etc/nagios.cfg` file to make sure that the `accept_passive_service_checks` option is set to `1` as well, so that Nagios Core allows passive checks on startup.

2.  Change to the Nagios Core `objects` configuration directory. If you're using the sample configuration, this will likely be `/usr/local/nagios/etc/objects`.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

3.  Edit the `commands.cfg` file, and add a definition for the `check_dummy` command:

    ```
    define command {
        command_name  check_dummy
        command_line  $USER1$/check_dummy $ARG1$ $ARG2$
    }
    ```

    If you already followed the *Monitoring individual nodes in a cluster* recipe in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), *Understanding the Network Layout*, then you may already have defined this command, in which case you can skip this step, as the definition is the same.

4.  Edit the file containing the definition for the existing host. In this example, the host is defined in a file called `ithaca.naginet.cfg`.

    ```
    # vi ithaca.naginet.cfg

    ```

5.  Add the following service definition to the end of the file, substituting the appropriate value for `host_name`.

    ```
    define service {
        use                     generic-service
        host_name               ithaca.naginet
        service_description     BACKUP
     active_checks_enabled   0
     passive_checks_enabled  1
     check_command           check_dummy!1!"Unwanted active check!"
    }
    ```

    This example uses the `generic-service` template. You can use any service template you like; the important directives are `active_checks_enabled`, `passive_checks_enabled`, and `check_command`.

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the Nagios Core web interface should show the service as accepting passive checks only in the **Services** listing:

![How to do it...](img/5566_11_02.jpg)

It will remain in the `PENDING` state until a passive check result is submitted for it.

We can submit a passive check result via the web interface as follows:

1.  Click on the service's name in the **Services** listing, and click on **Submit passive check result for this service** menu item:![How to do it...](img/5566_11_03.jpg)
2.  Complete the resulting form, with the following values:

    *   **Host Name**: This is the host name for which the passive check result should be submitted. This should already be filled out for us.
    *   **Service**: This is the service description for which the passive check result should be submitted. This should also already be filled out with `BACKUP`.
    *   **Check Result**: This is the particular result you would like to submit for the check. In this case, we choose `OK` to signal that the backup completed successfully. We could just as easily submit a `CRITICAL` result if we wished.
    *   **Check Output**: This is a message to attach to the status. In this case, we choose the simple message **Nightly backups were successful**.
    *   **Performance Data**: This is the optional extra detail about how the service being checked is performing. We can leave this blank.

3.  Click on **Commit** to submit the passive check result:![How to do it...](img/5566_11_04.jpg)

After a short delay, the detail for the service should show it as reflecting the result of the passive check, along with explicitly showing that active checks are disabled:

![How to do it...](img/5566_11_05.jpg)

## How it works...

The configuration added in the preceding section adds a new service called `BACKUP` to the existing `ithaca.naginet` host, and is designed to manage and report the status of backups for this host. In our example, this isn't something Nagios Core can check manually; no network service on `ithaca.naginet` can be checked to see if the backups have succeeded.

However, suppose as administrators we do receive a backup report in our inbox every morning, so we know whether the backups have succeeded or failed and would like to register that status in Nagios Core, perhaps for record-keeping purposes or to alert other administrators to problems.

To that end, we disable active checks for the service, and put in place a dummy check command, `check_dummy`, which we never expect to run. If for whatever reason an active check is run, it will always flag a `WARNING` state, with the message **Unwanted active check!**. The `check_dummy` command never actually checks anything; it is configured to always return the state and output defined in its two arguments.

Instead, we enable passive checks for the service and submit the results manually. If the backups failed, we could just as easily record that with a passive check result of `WARNING` or `CRITICAL`.

## There's more...

It's also possible (and often desirable) to submit active checks via the external commands file, which is useful for automation purposes. We write details for the check in a single line into the commands file in the following format:

```
[<timestamp>] PROCESS_SERVICE_CHECK_RESULT;<host_name>;<service_description>;<service_status>;<plugin_output>
```

For our example, the line would be similar to the following code snippet:

```
[1348806599] PROCESS_SERVICE_CHECK_RESULT;ithaca.naginet;BACKUP;0;Nightly backups were successful
```

We can write this directly to the external commands file as follows:

```
# CHECK="[`date +%s`] PROCESS_SERVICE_CHECK_RESULT;ithaca.naginet;BACKUP;0;Nightly backups were successful"
# echo $CHECK >>/usr/local/nagios/var/rw/nagios.cmd

```

In this context, the `<service_status>` field needs to be an integer corresponding to the appropriate state. If you use the text value `OK` or `WARNING`, the command will not work as expected.

*   `0` for `OK`
*   `1` for `WARNING`
*   `2` for `CRITICAL`
*   `3` for `UNKNOWN`

If the syntax is correct, then the passive check will be registered just the same way as if it were submitted via the web interface. Writing to the command file thus allows us to submit passive check results with scripts and automated systems, with a little knowledge of an appropriate shell scripting language such as Bash or Perl.

We go into a little more detail about using external commands for passive check results, including a common application with the NSCA add-on, in the *Submitting passive checks from a remote host with NSCA* recipe in this chapter. If you don't want to input your passive checks manually, then you will most likely find this recipe of interest, along with its accompanying explanation of freshness checks.

## See also

*   The *Submitting passive checks from a remote host with NSCA*, *Submitting passive checks in response to SNMP traps*, and *Setting up an event handler script* recipes in this chapter

# Submitting passive checks from a remote host with NSCA

In this recipe, we'll show how to automate the submission of passive checks by a remote host, using the example of a monitored host, `ithaca.naginet`, submitting a passive check to a Nagios Core server with information about how its `BACKUP` service is performing.

For example, if the backup process completed successfully, we configure the monitored host to submit a passive check result specifying that the `BACKUP` service should have the status `OK`. However, if there were a problem with the backup, the monitored host could send a passive check result with a `WARNING` or `CRITICAL` status.

In both cases, Nagios Core does no checking of its own; it trusts the results submitted by its target host.

To do this, we'll use the NSCA add-on. We'll install the NSCA server on the Nagios Core server, and the NSCA client program `send_nsca` on the monitored host.

## Getting ready

You should already have followed the *Allowing and submitting passive checks* recipe in this chapter. In this recipe, we will be building on the configuration established in that recipe; specifically, we will assume that you already have a host with a service configured only to accept passive checks.

You will need to be able to install the software on both the monitoring server (the NSCA server) and on the server that will submit passive checks (the NSCA client), and ideally be generally familiar with the `./configure`, `make`, and `make install` process for installing software from source on UNIX-like systems.

You should also be able to define any necessary firewall configuration to allow the NSCA client to send information to TCP port `5667` on the NSCA server. A firewall is absolutely necessary to protect the `nsca` daemon from abuse.

## How to do it...

We can set up the NSCA server on the monitoring server (in this example, `olympus.naginet`) as follows:

1.  Download the latest version of NSCA using `wget` or a similar tool. You can find download links on the Nagios Exchange page for NSCA at [http://exchange.nagios.org/directory/Addons/Passive-Checks/NSCA--2D-Nagios-Service-Check-Acceptor/details](http://exchange.nagios.org/directory/Addons/Passive-Checks/NSCA--2D-Nagios-Service-Check-Acceptor/details).

    In this example, we're downloading and compiling it in our home directory on the monitoring server.

    ```
    $ cd
    $ wget http://downloads.sourceforge.net/project/nagios/nsca-2.x/nsca-2.7.2/nsca-2.7.2.tar.gz

    ```

2.  Inflate the `.tar.gz` file:

    ```
    $ tar -xzf nsca-2.7.2.tar.gz

    ```

3.  Move into the new `nsca-2.7.2` directory, and configure and compile the `nsca` daemon. Note that this process may prompt you to install the `libmcrypt` library and its headers, perhaps in `libmcrypt` and `libmcrypt-dev` packages in your system's package manager:

    ```
    $ cd nsca-2.7.2
    $ ./configure
    $ make

    ```

4.  Install the NSCA server files manually; you will likely need `root` privileges for this:

    ```
    # cp src/nsca /usr/local/nagios/bin
    # cp sample-config/nsca.cfg /usr/local/nagios/etc

    ```

5.  Edit the new file `/usr/local/nagios/etc/nsca.cfg`:

    ```
    # vi /usr/local/nagios/etc/nsca.cfg

    ```

6.  Uncomment the `password` directive, and define it. A random password generated by a tool such as `pwgen` or `makepasswd` will work fine. Don't use the one below; it's just an example!

    ```
    password=yV3aa6S2o
    ```

7.  Check that the NSCA daemon runs with no errors:

    ```
    # /usr/local/nagios/bin/nsca -c /usr/local/nagios/etc/nsca.cfg --single

    ```

    If it does, you should add this command to an appropriate startup script, perhaps in `/etc/rc.local`, so that the daemon starts when the monitoring server boots. You should consult your system's documentation to find out the best place to add this.

We can set up the NSCA client on the monitored server (in this example, `ithaca.naginet`) as follows:

1.  Again, download and expand the latest version of NSCA, and configure and compile it:

    ```
    $ cd
    $ wget http://downloads.sourceforge.net/project/nagios/nsca-2.x/nsca-2.7.2/nsca-2.7.2.tar.gz
    $ tar -xzf nsca-2.7.2.tar.gz
    $ cd nsca-2.7.2
    $ ./configure
    $ make

    ```

2.  Install the NSCA client files manually; you will likely need `root` privileges for this, and may need to create the `/usr/local/bin` and `/usr/local/etc` directories beforehand:

    ```
    # mkdir -p /usr/local/bin /usr/local/etc
    # cp src/send_nsca /usr/local/bin
    # cp sample-config/send_nsca.cfg /usr/local/etc

    ```

3.  Edit the new file `/usr/local/etc/send_nsca.cfg`:

    ```
    # vi /usr/local/etc/send_nsca.cfg

    ```

4.  Uncomment the `password` directive, and define it to be the same as the password given in `nsca.cfg` on the monitoring server:

    ```
    password=yV3aa6S2o
    ```

5.  Run the `send_nsca` program to try and submit a passive check result:

    ```
    # CHECK="ithaca.naginet\tBACKUP\t0\tBackup was successful, this check submitted by NSCA\n"
    # echo -en $CHECK | send_nsca -c /usr/local/etc/send_nsca.cfg -H olympus.naginet
    1 data packet(s) sent to host successfully.

    ```

    Substitute the appropriate host names for the monitoring server (`olympus.naginet`), the monitored server (`ithaca.naginet`), and the service description `BACKUP`.

    Note that the fields are separated by `\t` characters, which expand to literal *Tab* characters with `echo -en`.

If this worked correctly, you should see that the passive check result in the web interface was successfully read by the monitoring server and applied appropriately:

![How to do it...](img/5566_11_06.jpg)

## How it works...

The `nsca` daemon installed on the monitoring server is designed to listen for submitted service checks from the `send_nsca` client, provided that the password is correct and the data is in the appropriate format:

```
<host_name>\t<service_description>\t<check_result>\t<check_output>\n
```

Our example passive check took this form:

```
ithaca.naginet\tBACKUP\t0\tBackup was successful, this check submitted by NSCA\n
```

Here, as with locally submitted passive checks, `check_result` corresponds to a numeric value, to one of the following:

*   `0` for `OK`
*   `1` for `WARNING`
*   `2` for `CRITICAL`
*   `3` for `UNKNOWN`

Once received by the `nsca` daemon on the monitoring server, this is translated into a passive check result command, written to the Nagios Core external commands file at `/usr/local/nagios/var/rw/nagios.cmd`, and processed in the same way as a locally submitted passive check would be.

This allows us to include calls to `send_nsca` at the end of scripts such as those managing backups, to immediately and automatically send a passive check result corresponding to whether the backup script succeeded.

Because of the NSCA daemon's very simple design and very basic security checks, it's important to apply a firewall policy to ensure that only the appropriate hosts can write to the NSCA port on the host monitoring system. A password as implemented here is a good first step, but is not sufficient to keep things secure. Make sure you read the `SECURITY` file included in the NSCA sources to ensure your configuration for the daemon is secure. Similar security guidelines apply to the installation of NRPE as discussed in [Chapter 6](ch06.html "Chapter 6. Enabling Remote Execution").

## There's more...

To supplement this setup, it's often a good idea to also have Nagios Core check the freshness of its services. If we have a process that needs to run regularly, such as backups, we will likely want to be notified if we haven't received any passive checks from the host in a given period of time.

This can be managed by configuring the service to run an active check after a certain period of time has elapsed with no passive checks. The configuration might look similar to the following code snippet, adding values for `check_freshness` and `freshness_threshold`:

```
define service {
    use                     generic-service
    host_name               ithaca.naginet
    service_description     BACKUP
    active_checks_enabled   0
    passive_checks_enabled  1
 check_freshness         1
 freshness_threshold     86400
    check_command           check_dummy!1!"No backups have run for 24 hours"
}
```

In this case, `freshness_threshold` is 86400 seconds, or 24 hours; if there have been no passive checks submitted for 24 hours, `check_command` will be run, even though active checks are disabled. `check_command` is defined to flag a `WARNING` state for the service with an appropriate explanatory message using the `check_dummy` command and plugin, whenever it is actually run.

Check freshness is discussed in more detail in the Nagios Core documentation, in the section entitled *Service and Host Freshness Checks* at [http://nagios.sourceforge.net/docs/3_0/freshness.html](http://nagios.sourceforge.net/docs/3_0/freshness.html).

Note that there's no reason that the status of a service has to come from the same host. You can send a passive check from one host to submit information about another. In fact, this is the basis of a distributed monitoring setup; one host can submit check results for any number of other hosts.

This can be particularly useful for working around network connectivity or routing problems; if Nagios Core has no connectivity at all to a host it needs to monitor, but does have connectivity to an intermediate host, that host can be configured to submit checks on behalf of the unreachable host, a slightly complex but often necessary setup.

## See also

*   The *Allowing and submitting passive checks*, *Submitting passive checks in response to SNMP traps*, and *Setting up an event handler script* recipes in this chapter
*   The *Using an alternative check command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*

# Submitting passive checks in response to SNMP traps

In this recipe, we'll learn how to configure Nagios Core to process **Simple Network Management Protocol** (**SNMP**) traps, information sent by monitored network devices to a central monitoring server.

Because SNMP traps often contain useful or urgent information about how a host is working, processing them in at least some way can be very helpful, particularly for firmware network devices that can't use `send_nsca` to submit a passive check result in a standard form, as explained in the *Submitting passive checks from a remote host with NSCA* recipe.

As an example, most SNMP-capable hosts can be configured to send SNMP traps when one of their network interfaces changes state, perhaps due to a pulled network cable. These are known as `linkUp` and `linkDown` traps. Monitoring this particular kind of trap is especially useful for devices with a large number of interfaces, such as switches or routers.

Keeping track of these events in Nagios Core is valuable for keeping a unified monitoring interface, rather than having to monitor SNMP traps with a separate application.

## Getting ready

There are quite a few prerequisites for getting this recipe to work. It is among the most powerful but also most complex methods of Nagios Core monitoring.

First of all, the recipe assumes some knowledge of SNMP. Unfortunately, SNMP is not very simple, despite its name! You should be familiar with the concepts of **SNMP checks** and **SNMP traps**. The documentation for **Net-SNMP** (the implementation of SNMP used for this example) may help ([http://www.net-snmp.org/docs/readmefiles.html](http://www.net-snmp.org/docs/readmefiles.html)).

On the same host as your Nagios Core server with version 3.0 or greater, you should have `snmptrapd` installed to collect `trap` information, and `snmptt`, the **SNMP Trap Translator** , to filter useful information from the traps and submit the information to Nagios Core in a workable format. Documentation for SNMPTT is available at [http://snmptt.sourceforge.net/docs/snmptt.shtml](http://snmptt.sourceforge.net/docs/snmptt.shtml).

Both systems are free software and relatively popular, so check to see if there are packages available for your particular system to save the hassle of compiling them from source. On Debian-derived systems such as Ubuntu, for example, they are available in the `snmpd` and `snmptt` packages.

We will use an event handler called `submit_check_result`, available in the Nagios Core distribution. You will therefore need to have access to the original sources handy. If you have misplaced them, you can download them again from Nagios' website: [http://www.nagios.org/download](http://www.nagios.org/download).

It's also necessary to use values of `host_name` for your hosts that actually correspond to host names resolvable by DNS from the monitoring server. This is because when the SNMP trap is received by SNMPTT, the only way it can translate it to a host name is with DNS. `host_name` for your host might be `crete.naginet`, but the trap will arrive from an IP address such as `10.128.0.27`. The system will therefore need to be able to resolve this with reverse DNS lookup. An easy way to test this is working is to use `host` or `dig`:

```
$ host 10.128.0.27
27.0.128.10.in-addr.arpa domain name pointer crete.naginet.
$ dig -x 10.128.0.27 +short
crete.naginet.

```

Finally, you should, of course, actually have a device configured to send SNMP traps to your monitoring server, which in turn is configured to listen for SNMP traps with the `snmpd` daemon. I don't really want to encourage you to unplug one of your core switches to test this, so we'll generate a trap manually with `snmptrap` on the monitored server to demonstrate the principle.

## How to do it...

We can configure a new service to receive SNMP traps for an existing host as follows:

1.  Copy the event handler script `contrib/eventhandlers/submit_check_result` from the Nagios Core source files into `/usr/local/nagios/libexec/eventhandlers`. You may need to create the target directory first. Your source files need not be in `/usr/local/src/nagios`; this is just an example.

    ```
    # mkdir -p /usr/local/nagios/libexec/eventhandlers
    # cp /usr/local/src/nagios/contrib/eventhandlers/submit_check_result /usr/local/nagios/libexec/eventhandlers

    ```

    This script should be made executable as whatever user the `snmptrapd` user runs as.

2.  Change to the Nagios Core `objects` configuration directory on the monitoring server. For the default configuration, this is `/usr/local/nagios/etc/objects`.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

3.  Edit the file containing the definition for the SNMP-enabled monitored host. In this example, the definition for `crete.naginet` is in its own file, `crete.naginet.cfg`:

    ```
    # vi crete.naginet.cfg

    ```

    The host definition might look similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  crete.naginet
        alias      crete
        address    10.128.0.21
    }
    ```

4.  Add a service definition for your existing host that accepts only passive checks, and is flagged as `volatile`. Here we have used the `generic-service` template included in the sample configuration. You may prefer to use a different template, but all of the values defined here are important.

    ```
    define service {
        use                     generic-service
        host_name               crete.naginet
        service_description     TRAP
     is_volatile             1
        check_command           check-host-alive
     active_checks_enabled   0
     passive_checks_enabled  1
        max_check_attempts      1
        contact_groups          admins
    }
    ```

5.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

    In the web interface, this service should now be visible in the **Services** section:

    ![How to do it...](img/5566_11_07.jpg)
6.  Check that the `submit_check_result` script actually works, by invoking it with a test string on the monitoring server:

    ```
    # /usr/local/nagios/libexec/eventhandlers/submit_check_result crete.naginet TRAP 0 "Everything working"

    ```

    After a short delay, if this has worked correctly, we should see the service change state in the web interface to reflect the test:

    ![How to do it...](img/5566_11_08.jpg)

We now need to configure `snmptrapd` and `snmpd` to receive traps, and call the `submit_check_result` script for us:

1.  Configure `snmpd` to pass received traps to `snmptt` by changing its configuration file `/etc/snmp/snmptrapd.conf`. The following configuration may work:

    ```
    traphandle default /usr/sbin/snmptt
    disableAuthorization yes
    donotlogtraps yes
    ```

2.  Restart `snmpd` to apply this change:

    ```
    # /etc/init.d/snmpd restart

    ```

3.  Configure `snmptt` to convert the IP addresses to hostnames, by changing the value for `dns-enable` to `1` in `/etc/snmp/snmptt.ini`:

    ```
    dns_enable = 1
    ```

4.  Configure `snmptt` to use Net-SNMP at startup in `/etc/snmp/snmptt.ini`:

    ```
    net_snmp_perl_enable = 1
    ```

5.  Configure `snmptt` to respond to an SNMP event by defining it in `/etc/snmp/snmptt.conf`. Here we've used the generic `linkDown` event defined by the OID `.1.3.t6.1.6.3.1.1.5.3`:

    ```
    EVENT linkDown .1.3.6.1.6.3.1.1.5.3 "Status Events" Normal
    FORMAT Link down on interface $1\.  Admin state: $2\.  Operational state: $3
    EXEC /usr/local/nagios/libexec/eventhandlers/submit_check_result $r TRAP 1 "linkDown for interface $1"
    SDESC
    A linkDown trap signifies that the SNMP entity, acting in
    an agent role, has detected that the ifOperStatus object for
    one of its communication links is about to enter the down
    state from some other state (but not from the notPresent
    state).  This other state is indicated by the included value
    of ifOperStatus.
    EDESC
    ```

    Depending on your distribution, there may already be a definition for a `linkDown` event, in which case you may only need to change the `EXEC` field.

6.  From the monitored host, fire a test trap for a `linkDown` event. Substitute `olympus.naginet` for the name or IP address of your monitoring host. This will require the `snmptrap` utility to be installed on that host, and may require `root` privileges.

    ```
    # snmptrap -v 1 -c public olympus.naginet .1.3.6.1.6.3.1.1.5.3 localhost 2 0 '' .1.3.6.1.2.1.2.2.1.1.1 i 1

    ```

    Note that we use the `public` community string here; your own will likely differ.

7.  Check the Nagios Core log file located at `/usr/local/nagios/var/nagios.log` to see if there's new output from the event handler:

    ```
    [1348914096] EXTERNAL COMMAND: PROCESS_SERVICE_CHECK_RESULT;crete.naginet;TRAP;1;linkDown for interface 1
    [1348914100] PASSIVE SERVICE CHECK: crete.naginet;TRAP;1;linkDown for interface 1
    [1348914100] SERVICE ALERT: crete.naginet;TRAP;WARNING;HARD;3;linkDown for interface 1
    [1348914100] SERVICE NOTIFICATION: nagiosadmin;crete.naginet;TRAP;WARNING;notify-service-by-email;linkDown for interface 1

    ```

    If so, the same state should be reflected for the `TRAP` service in the web interface:

    ![How to do it...](img/5566_11_09.jpg)

With this done, we've confirmed that SNMP traps from the `crete.naginet` host can be received and processed by the `olympus.naginet` server. We can apply the same setup for other hosts that generate SNMP traps in our network by configuring them to send their traps to the Nagios Core monitoring server, and adding appropriate handlers for the expected traps.

If this didn't work, the first thing to check should be that your monitoring server is actually listening for checks on the relevant IP address, as `snmptrap` does not throw errors when it can not deliver traps. On Debian-derived systems, you should check that the `snmptrapd` process is actually running; it may require a change to `/etc/defaults/snmp` and a restart of `snmpd`.

## How it works...

When an SNMP trap is generated and delivered to the monitoring server by whatever means, the `snmpd` daemon will pass it to the `snmptt` program for processing.

The `snmptt` handler checks if the event OID matches any of the traps for which it has defined events in `snmptt.conf`. In our example, it finds a handler defined for the OID `.1.3.6.1.6.3.1.1.5.3`, which corresponds to the `linkDown` event, with the number of the relevant interface as an additional argument in `$1`.

Using this information, it fires the `submit_check_result` handler that we installed in the first part of the recipe, setting the state of the `TRAP` service to `WARNING`, and including the information `linkDown` for interface `1`, as specified by the final argument to `submit_check_result` in the `EXEC` handler. The service can be set to notify the appropriate contacts or contact groups, just as it would for an actively monitored service.

If a trap arrives on the Nagios Core server for a host that Nagios Core doesn't know about, even if it has an event handler defined for `snmptt`, it will simply ignore it.

## There's more...

In order to "clear" the state of the service and return it to `OK`, we can simply schedule an active check for it from the web interface, with **Force Check** selected:

![There's more...](img/5566_11_10.jpg)

Because `check_command` is defined as `check-host-alive`, as long as the monitoring host is actually responding to `PING`, the service should assume an `OK` state:

![There's more...](img/5566_11_11.jpg)

## See also

*   The *Submitting passive checks from a remote host with NSCA*, *Allowing and submitting passive checks*, and *Setting up an event handler scripts* recipes in this chapter
*   The *Monitoring the output of an SNMP query* and *Creating an SNMP OID to monitor* recipes in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*

# Setting up an event handler script

In this recipe, we'll learn how to set up an event handler script for Nagios Core. Event handlers are commands that are run on every state change for a host or service (whether for all hosts or services, or just particular ones). They are defined in a similar way to notification commands and check commands for plugins.

In this example, we'll implement a simple event handler that writes the date, the host state, and the number of check attempts to a separate file for a single host. This is a trivial example to demonstrate the concept; a more practical and complex application for the use of event handlers is given in the *Setting up a redundant monitoring host* recipe, in [Chapter 10](ch10.html "Chapter 10. Security and Performance").

## Getting ready

You will need a server running Nagios Core 3.0 or higher. You should be familiar with defining new commands, as per the *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins") and the *Writing low-priority notifications to an MOTD* recipe in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*.

## How to do it...

We can set up a new event handler for the Nagios Core server as follows:

1.  Change to the `objects` configuration directory for Nagios Core. In the quick start guide installation, this is `/usr/local/nagios/etc/objects`. Edit the file `commands.cfg`:

    ```
    # cd /usr/local/nagios/etc/objects
    # vi commands.cfg

    ```

2.  Add the following command definition:

    ```
    define command {
        command_name    record_host_data
        command_line    /usr/bin/printf "%b" "$LONGDATETIME$: $HOSTSTATE$ (attempt $HOSTATTEMPT$)\n" >>/usr/local/nagios/var/states-$HOSTNAME$.log
    }
    ```

3.  Edit the file containing the definition for an existing host, in this example `delphi.naginet`. Add the `event_handler` directive with the value `record_host_data` to your host definition:

    ```
    define host {
        use            linux-server
        host_name      delphi.naginet
        alias          delphi
        address        10.128.0.26
     event_handler  record_host_data
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the next time the host changes state (whether to a `SOFT` or `HARD` state) it should log the information in the `/usr/local/nagios/var/states-delphi.naginet.log` file:

```
Sat Sept 29 23:53:54 NZST 2012: DOWN (attempt 1)
Sat Sept 29 23:54:04 NZST 2012: DOWN (attempt 2)
Sat Sept 29 23:54:14 NZST 2012: DOWN (attempt 3)
Sat Sept 29 23:57:14 NZST 2012: UP (attempt 1)

```

## How it works...

The `event_handler` command we defined is configured to use `printf` to write a line of text to a file named after the host. Its definition is built out of four macros:

*   `$LONGDATETIME$`: This specifies the date and time, in a human-readable format
*   `$HOSTSTATE$`: This specifies the state of the host (`UP`, `DOWN`, or `UNREACHABLE`)
*   `$HOSTATTEMPT$`: This specifies the number of check attempts made so far for a host in a problem state
*   `$HOSTNAME$`: This is the hostname itself (used to build the name of the file)

Note that this behavior is slightly different from notifications. Notification commands are only run when the number of `max_check_attempts` for a host or service has been exceeded, to alert somebody to the problem. Event handlers are run on `SOFT` changes as well as `HARD` changes, and hence can be used to keep more information about host performance that might be missed by the routine notifications.

Service event handlers can be defined in just the same way, by adding the `event_handler` directive to their definitions:

```
define service {
    use                  generic-service
    host_name            crete.naginet
    service_description  PING
    check_command        check_ping!100,10%!200,20%
 event_handler        record_service_data
}
```

In this case, we would probably want to use the macros for service states instead:

```
define command {
    command_name    record_service_data
    command_line    /usr/bin/printf "%b" "$LONGDATETIME$: $SERVICESTATE$ (attempt $SERVICEATTEMPT$)\n" >>/usr/local/nagios/var/states-$HOSTNAME$-$SERVICEDESC$.log
}
```

## There's more...

As a shortcut, if there's an event handler we want to run on all hosts or all services, we can use the `global_host_event_handler` and `global_service_event_handler` directives in `nagios.cfg`:

```
global_host_event_handler=record_host_data
global_service_event_handler=record_service_data
```

This will apply the appropriate event handlers to all hosts and services, therefore running whenever a host or service changes state.

A specialized case of event handler for recording detailed performance data of plugins and checks is also possible using Nagios Core's Performance Data feature, as documented in the manual at [http://nagios.sourceforge.net/docs/3_0/perfdata.html](http://nagios.sourceforge.net/docs/3_0/perfdata.html).

Performance data is written on every check rather than every state change, and is hence useful for assessing the performance of plugins and checks. Performance data is used by the **Nagiosgraph** utility, for example, discussed in the *Tracking host and service states with Nagiosgraph* recipe in this chapter.

## See also

*   The *Tracking host and service states with Nagiosgraph* recipe in this chapter
*   The *Creating a new command* recipe in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*
*   The *Writing low-priority notifications to an MOTD* recipe in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*

# Tracking host and service states with Nagiosgraph

In this recipe, we'll learn how to install and configure Nagiosgraph, a program that integrates with Nagios Core's performance data tools to produce graphs showing long-term information about how checks for hosts and services are performing.

## Getting ready

You will need to be running a Nagios Core 3.0 or later server. Nagiosgraph will probably still work with older versions of Nagios Core, but the configuration may be slightly different. The `INSTALL` document included in the source for Nagiosgraph explains the differences in detail.

You should have a thorough understanding of defining hosts, services, and commands, and be able to install new software as the `root` user on the monitoring server. You should also be at least familiar with the layout of your Apache HTTPD server on the monitoring system; this recipe will assume it is installed in `/usr/local/apache`.

Because Nagiosgraph has many Perl dependencies, you will need to have Perl installed on your server, and you will likely also need to install a few Perl modules as dependencies. The package manager for your system may include them, or you may need to download them using the **Comprehensive Perl Archive Network** (**CPAN**): [http://www.cpan.org/modules/INSTALL.html](http://www.cpan.org/modules/INSTALL.html).

The server will need to already be monitoring at least one host with at least one service for the graphs to be any use. Nagiosgraph includes rule sets that translate known performance data strings into usable statistics. This means that graphing will work well for familiar plugins with a predictable output format such as `check_ping` or `check_http`, but might not graph data for less commonly used plugins without a little custom configuration.

This recipe is not a comprehensive survey of everything you can do with Nagiosgraph; if you like what this does, make sure to check out Nagiosgraph's documentation online at [http://nagiosgraph.sourceforge.net/](http://nagiosgraph.sourceforge.net/).

## How to do it...

We can get some basic Nagiosgraph functionality for our monitoring server as follows:

1.  Download the latest version of Nagiosgraph from its website at [http://nagiosgraph.sourceforge.net/](http://nagiosgraph.sourceforge.net/), directly onto your monitoring server, using a tool such as `wget`:

    ```
    $ cd
    $ wget http://downloads.sourceforge.net/project/nagiosgraph/nagiosgraph/1.4.4/nagiosgraph-1.4.4.tar.gz

    ```

2.  Inflate the `.tar.gz` file and change to the directory within it:

    ```
    $ tar -xzf nagiosgraph-1.4.4.tar.gz
    $ cd nagiosgraph-1.4.4

    ```

3.  As the `root` user, run the `install.pl` script with the `--check-prereq` option. This will give you a survey of any dependencies you may need to install via packages or CPAN. When you have installed all the prerequisites, the output should look similar to the following code snippet:

    ```
    # ./install.pl --check-prereq
    checking required PERL modules
     Carp...1.11
     CGI...3.43
     Data::Dumper...2.124
     File::Basename...2.77
     File::Find...1.14
     MIME::Base64...3.08
     POSIX...1.17
     RRDs...1.4003
     Time::HiRes...1.9719
    checking optional PERL modules
     GD...2.39
    checking nagios installation
     found nagios at /usr/local/nagios/bin/nagios
    checking web server installation
     found apache at /usr/sbin/apache2

    ```

    These are all reasonably standard Perl libraries, so don't forget to check if packages are available for them before you resort to using CPAN. For example, I was able to install the RRDs and GD modules on my Debian system as follows:

    ```
    # apt-get install librrds-perl libgd-gd2-perl

    ```

    If you are having trouble getting `install.pl` to find your Nagios Core or Apache HTTPD instances, then take a look at the output of `install.pl --help` to run an installation specific to your kind of system. This is documented in more detail in the `INSTALL` file.

4.  As the `root` user, run the `install.pl` script with the `--install` argument. You will be prompted many times for directory layout options. The default is shown in square brackets and should be correct for a typical Nagios Core installation, so to start with, simply press *Enter* on each option.

    ```
    # ./install.pl --install
    ...Destination directory (prefix)? [/usr/local/nagiosgraph]
    Location of configuration files (etc-dir)? [/usr/local/nagiosgraph/etc]
    Location of executables? [/usr/local/nagiosgraph/bin]
    Location of CGI scripts? [/usr/local/nagiosgraph/cgi]
    Location of documentation (doc-dir)? [/usr/local/nagiosgraph/doc]
    Location of examples? [/usr/local/nagiosgraph/examples]
    Location of CSS and JavaScript files? [/usr/local/nagiosgraph/share]
    Location of utilities? [/usr/local/nagiosgraph/util]
    Location of state files (var-dir)? [/usr/local/nagiosgraph/var]
    Location of RRD files? [/usr/local/nagiosgraph/var/rrd]
    Location of log files (log-dir)? [/usr/local/nagiosgraph/var]
    Path of log file? [/usr/local/nagiosgraph/var/nagiosgraph.log]
    Path of CGI log file? [/usr/local/nagiosgraph/var/nagiosgraph-cgi.log]
    URL of CGI scripts? [/nagiosgraph/cgi-bin]
    URL of CSS file? [/nagiosgraph/nagiosgraph.css]
    URL of JavaScript file? [/nagiosgraph/nagiosgraph.js]
    Path of Nagios performance data file? [/tmp/perfdata.log]
    URL of Nagios CGI scripts? [/nagios/cgi-bin]
    username or userid of Nagios user? [nagios]
    username or userid of web server user? [www-data]
    Modify the Nagios configuration? [n]
    Modify the Apache configuration? [n]
    ...

    ```

    After the preceding selections are all made, the files should be installed with appropriate permissions set. The final part of the output gives instructions for adding configuration to Nagios Core and Apache HTTPD, which we'll do next.

5.  Change to the Nagios Core configuration directory. In the quick start guide installation, this is `/usr/local/nagios/etc`.

    ```
    # cd /usr/local/nagios/etc

    ```

6.  Edit the core configuration file `nagios.cfg`, and add the following directives at the end of the file:

    ```
    # process nagios performance data using nagiosgraph
    process_performance_data=1
    service_perfdata_file=/tmp/perfdata.log
    service_perfdata_file_template=$LASTSERVICECHECK$||$HOSTNAME$||$SERVICEDESC$||$SERVICEOUTPUT$||$SERVICEPERFDATA$
    service_perfdata_file_mode=a
    service_perfdata_file_processing_interval=30
    service_perfdata_file_processing_command=process-service-perfdata-for-nagiosgraph

    ```

7.  Change to the Nagios Core `objects` configuration directory. In the quick start guide installation, this is `/usr/local/nagios/etc/objects`.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

8.  Edit the `commands.cfg` file, and add the following command definition:

    ```
    # command to process nagios performance data for nagiosgraph
    define command {
     command_name process-service-perfdata-for-nagiosgraph
     command_line /usr/local/nagiosgraph/bin/insert.pl
    }

    ```

9.  Edit the `httpd.conf` file for your Apache HTTPD server to include the following line at the end:

    ```
    Include /usr/local/nagiosgraph/etc/nagiosgraph-apache.conf
    ```

    In a local install of Apache HTTPD, this file is normally in `/usr/local/apache/conf/httpd.conf`, but its location varies by system. On Debian-derived systems it may be `/etc/apache2/apache2.conf`.

10.  Validate the configuration of both the Apache HTTPD server and the Nagios Core server, and restart them both:

    ```
    # /usr/local/apache/bin/apachectl configtest
    # /usr/local/apache/bin/apachectl restart
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

11.  Visit [http://olympus.naginet/nagiosgraph/cgi-bin/showconfig.cgi](http://olympus.naginet/nagiosgraph/cgi-bin/showconfig.cgi) in your browser, substituting your own Nagios Core server's hostname, to test that everything's working. You should see a long page with configuration information for Nagiosgraph:![How to do it...](img/5566_11_12.jpg)
12.  If everything is working up to this point, the only thing left to do is to define an action URL for the services that you want to graph, so that you can click to go directly to the graphs for that service from the Nagios Core web interface.

    The tidiest and most straightforward way to do this is to define a service template:

    ```
    define service {
        name        nagiosgraph
        action_url  /nagiosgraph/cgi-bin/show.cgi?host=$HOSTNAME$&service=$SERVICEDESC$
        register    0
    }
    ```

    Then, you can have the services you want graphed inherit from it, as well as from any other templates they use, by adding `nagiosgraph` to the value for the `use` directive:

    ```
    define service {
     use                   generic-service,nagiosgraph
        host_name             corinth.naginet
        service_description   PING
        check_command         check_ping!100,10%!200,20%
    }
    ```

    You should do this for all the services for which you want graphing.

13.  Validate the configuration and restart the Nagios Core server again:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, visiting the **Service** section of the web interface should include action icons after each graphed service:

![How to do it...](img/5566_11_13.jpg)

Clicking one of these should bring up a graph interface; for example, a service using `check_ping` might show something similar to the following screenshot:

![How to do it...](img/5566_11_14.jpg)

Note that it includes two line bars to show the thresholds for `CRITICAL` and `WARNING` state as well as the actual response time. Also note that the preceding graph is several days old; it will take a while to build up enough data to see a perceptible line, and you may not see any graphs until performance data has actually been received from Nagios by Nagiosgraph.

Don't forget that graphs won't work out of the box for every service. If Nagiosgraph doesn't know how to parse the performance data for a check, it will show a red error text instead of graphs. We'll mention an approach to fixing this in the *There's more...* section.

## How it works...

The configuration changes in the preceding section prompt Nagios Core to log performance data for every check on every service, using the `service_perfdata_file_processing_command` directive. This command, named `process-service-perfdata-for-nagiosgraph`, is defined to pass data to the `bin/insert.pl` script included in the new `/usr/local/nagiosgraph` directory.

This script in turn parses performance output, such as the following output from a typical service using `check_ping`:

```
PING OK - Packet loss = 0%, RTA = 174.19 ms
```

Nagiosgraph extracts numeric information from the performance data, according to the templates defined in `/usr/local/nagiosgraph/etc/map`, using Perl regular expressions. This data is recorded using Perl's bindings for the RRD library, and graphed using the GD2 library, with an appearance similar to graphs produced by MRTG.

The `action_url` directive uses macros for each service to define a URL for each service that shows its graphs. In our example, for a service PING on host `corinth.naginet`, `action_url` would expand to the following:

```
action_url  /nagiosgraph/cgi-bin/show.cgi?host=corinth.naginet&service=PING
```

This isn't the only possible use of `action_url`, of course; it just happens to be a useful one in our case. You can make `action_url` go anywhere you'd like for a given host or service.

## There's more...

If you don't intend to define any other kind of action for services, you may like to change the `action.gif` image to something more descriptive than the default red splotch. The Nagiosgraph sources include a possible alternative icon, but you can use any GIF image you wish.

```
# cp nagiosgraph-1.4.4/share/graph.gif /usr/local/nagios/share/images/action.gif

```

You may well be running some kind of check that Nagiosgraph isn't able to graph, because it doesn't understand the format of the performance output, and can't extract numeric information from it. The default mapping rules cover output from quite a few standard plugins, but if you know a little about Perl, then you may be able to add more rules to `/usr/local/nagiosgraph/etc/map` to process other kinds of plugin output.

Besides the examples in the map file, which include instructions for writing new output checks, there are more examples of such definitions included in the `/usr/local/nagiosgraph/examples/map_examples` file.

If you're comparing Nagios graphing solutions, another popular solution to try could be **PNP4Nagios**, available at [http://docs.pnp4nagios.org/pnp-0.6/start](http://docs.pnp4nagios.org/pnp-0.6/start).

## See also

*   The *Getting extra visualizations with NagVis* recipe in this chapter
*   The *Monitoring Nagios performance with Nagiostats* recipe in [Chapter 10](ch10.html "Chapter 10. Security and Performance"), *Security and Performance*

# Reading status into a MySQL database with NDOUtils

In this recipe, we'll learn how to install the **NDOUtils** extension to Nagios Core, in order to have all of Nagios Core's configuration and data written into a MySQL database. This allows easy development of custom reports and interfaces for Nagios Core data with languages, such as Perl and PHP, and their standard interfaces to the popular MySQL server, rather than needing to interact with Nagios Core's own logs or its data format. Some plugins, such as NagVis, use this format to read information about Nagios Core configuration and objects.

## Getting ready

You will need a Nagios Core server version 3.0 or later. NDOUtils will probably still install and work on older versions of Nagios Core, but the installation process is slightly different; see the `INSTALL` file included in the NDO source for information on this.

Nagios Core uses its event broker functionality to write information to the socket for the MySQL database to pick up. You will need to have compiled Nagios Core with the `--enable-event-broker` flag:

```
$ ./configure --enable-event-broker
$ make
# make install

```

If you are unsure whether you compiled with this flag, it is probably a good idea to recompile and reinstall Nagios Core from your original sources with this installed. Don't forget to back up your previous installation in case of problems.

In order to compile the `ndomod` part of NDOUtils, you will need to have the MySQL client libraries and headers installed on the Nagios Core server. You will also need to have a MySQL server ready to store the data. The MySQL server does not have to be running on the same host as Nagios Core, but the Nagios Core server should be able to connect to it.

Finally, you should take note of the opening paragraph of `README`, which at the time of writing points out that NDOUtils for Nagios Core 3.0 is still officially in beta; you should read the note and be aware of the risks in installing it. In my own experience, however, the code is very stable. There are also no guarantees at the time of writing that this procedure will work correctly with the unreleased Nagios 4.0.

## How to do it...

We can install the NDOUtils package for Nagios Core as follows:

1.  Download the latest NDOUtils `.tar.gz` from its Sourceforge site at [http://sourceforge.net/projects/nagios/files/ndoutils-1.x/](http://sourceforge.net/projects/nagios/files/ndoutils-1.x/).

    ```
    $ wget http://downloads.sourceforge.net/project/nagios/ndoutils-1.x/ndoutils-1.5.2/ndoutils-1.5.2.tar.gz

    ```

2.  Unpack the `.tar.gz` file and change directory into it.

    ```
    $ tar -xzf ndoutils-1.5.2.tar.gz
    $ cd ndoutils-1.5.2

    ```

3.  Run `./configure` and build the software. Note that there is no install target; we will be performing the installation manually.

    ```
    $ ./configure
    $ make

    ```

4.  Read the output of `./configure` carefully if the build fails, to determine if you are missing any dependencies on your system. The output of `./configure` should end with something similar to the following code snippet:

    ```
    *** Configuration summary for ndoutils 1.5.2 06-08-2012 ***:
    General Options:
    -------------------------
    NDO2DB user:    nagios
    NDO2DB group:   nagios
    Review the options above for accuracy.  If they look okay,
    type 'make' to compile the NDO utilities.

    ```

5.  On the MySQL server, create a database to store the Nagios Core information, and a user to access it. In this example, the MySQL server is running on the same host as the Nagios Core server (`olympus.naginet`), so the access will be done from `localhost`.

    ```
    mysql> CREATE DATABASE nagios;
    Query OK, 1 row affected (0.10 sec)
    mysql> CREATE USER 'ndo'@'localhost' IDENTIFIED BY 'mPYxbAYqa';
    Query OK, 0 rows affected (0.62 sec)
    mysql> GRANT ALL ON nagios.* TO 'ndo'@'localhost';
    Query OK, 0 rows affected (0.02 sec)

    ```

    We have used a random password after `IDENTIFIED BY`. You should generate your own secure password.

6.  Run the `installdb` script in the source to create the various tables Nagios Core will use. Use the database details established in the previous step:

    ```
    $ cd db
    $ ./installdb -u ndo -p mPYxbAYqa -h localhost -d nagios
    ** Creating tables for version 1.5.2
     Using mysql.sql for installation...
    ** Updating table nagios_dbversion
    Done!

    ```

    Don't be concerned about the following error message; it is because you are installing the extension for the first time:

    ```
    DBD::mysql::db do failed: Table 'nagios.nagios_dbversion' doesn't exist at ./installdb line 51.
    ```

7.  Copy the compiled `ndomod` module into the `/usr/local/nagios/bin` directory:

    ```
    # cd ..
    # cp src/ndomod-3x.o /usr/local/nagios/bin/ndomod.o

    ```

8.  Copy the sample configuration for the module into `/usr/local/nagios/etc`:

    ```
    # cp config/ndomod.cfg-sample /usr/local/nagios/etc/ndomod.cfg

    ```

9.  Make sure it is readable only by the `nagios` user:

    ```
    # chown nagios.nagios /usr/local/nagios/etc/ndomod.cfg
    # chmod 0600 /usr/local/nagios/etc/ndomod.cfg

    ```

10.  Edit your `nagios.cfg` file:

    ```
    # vi /usr/local/nagios/etc/nagios.cfg

    ```

11.  Add a `broker_module` definition to the file, and check that the `event_broker_options` directive is set to `-1`:

    ```
    broker_module=/usr/local/nagios/bin/ndomod.o config_file=/usr/local/nagios/etc/ndomod.cfg
    event_broker_options=-1
    ```

    Note that the `broker_module` and `config_file` definitions should be on the same line, but `event_broker_options` should be on its own line.

    With this done, the broker module ought to be successfully installed, and we can move on to installing the `ndo2db` daemon.

12.  Copy the `ndo2db` binary into the `/usr/local/nagios/bin` directory:

    ```
    # cp src/ndo2db-3x /usr/local/nagios/bin/ndo2db

    ```

13.  Copy the sample configuration for the daemon into `/usr/local/nagios/etc`:

    ```
    # cp config/ndo2db.cfg-sample /usr/local/nagios/etc/ndo2db.cfg

    ```

14.  Make sure it is readable only by the `nagios` user:

    ```
    # chown nagios.nagios /usr/local/nagios/etc/ndo2db.cfg
    # chmod 0600 /usr/local/nagios/etc/ndo2db.cfg

    ```

15.  Edit the configuration file as installed:

    ```
    # vi /usr/local/nagios/etc/ndo2db.cfg

    ```

16.  Change the values in `ndo2db.cfg` to reflect the database details:

    ```
    db_host=localhost
    db_port=3306
    db_name=nagios
    db_user=ndo
    db_pass=mPYxbAYqa
    ```

17.  Test the `ndo2db` daemon by starting it and verifying it is running with `ps -e` or `pgrep`:

    ```
    # /usr/local/nagios/bin/ndo2db -c /usr/local/nagios/etc/ndo2db.cfg
    # ps -e | grep '[n]do2db'
    32285 ? 00:00:00 ndo2db
    # pgrep ndo2db
    32285

    ```

    If it works, you should add this command into your system's `init` scripts, so that the daemon is started at boot time.

18.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, inspecting the database tables in MySQL should show they have been filled with information from Nagios Core, for example the `nagios_services` table:

```
$ mysql --user=ndo --password --database=nagios
mysql> select count(1) from nagios_services;
+----------+
| count(1) |
+----------+
|       54 |
+----------+
1 row in set (0.00 sec)

```

## How it works...

NDOUtils is in fact a collection of components, two of which we installed in the previous sections:

*   `ndomod` is used as a broker module, for writing events and data from Nagios Core to a UNIX socket in `/usr/local/nagios/var/ndo.sock`. It runs as a module of the Nagios Core server.
*   `ndo2db` is used as a database backend, for reading the events and data from the UNIX socket to which `ndomod` writes, and applying them as MySQL database operations. It runs independently as a daemon on the system, and performs the MySQL connection and transactions.

`broker_module` updates these tables as plugins are run, hosts and services change state, notifications are issued, and other Nagios Core behavior takes place. It covers most data of interest quite comprehensively. Note that it includes:

*   Configuration directives
*   Details for types of Nagios Core objects
*   Properties and current states of hosts
*   Acknowledgement and scheduled downtime information
*   Notification history and complete logging

The main reason to install NDOUtils is to put Nagios Core's data into a standardized format, so that it can be read and processed by external applications, whether in simple table-style reports or completely new application interfaces to the Nagios Core data. This tends to be much easier than custom building a Nagios Core CGI of your own!

## There's more...

Another recipe in this chapter, *Writing customized Nagios Core reports*, applies NDOUtils after its installation by demonstrating some example MySQL queries for retrieving useful data and summaries from its tables, including an example of writing a report in Perl, and a simple RSS feed in PHP.

To get the most out of NDOUtils, it's a good idea to take a look at its documentation, which includes a complete breakdown of the contents of the MySQL tables: [http://nagios.sourceforge.net/docs/ndoutils/NDOUtils.pdf](http://nagios.sourceforge.net/docs/ndoutils/NDOUtils.pdf).

## See also

*   The *Writing customized Nagios Core reports* and *Getting extra visualizations with NagVis* recipes in this chapter

# Writing customized Nagios Core reports

In this recipe, we'll explore some simple applications of the NDOUtils database by trying out some queries, and change one of them into both a simple report in Perl, and also into a PHP-based RSS feed.

## Getting ready

This recipe assumes you have NDOUtils already installed, and that your Nagios Core 3.0 (or later) server is monitoring at least a few hosts and services, so that the queries we try actually return some data. You should also have some means of executing MySQL queries on the database server. The `mysql` command-line client will work just fine; a tool such as phpMyAdmin might make the data a little easier to explore.

## How to do it...

We can explore some queries against the NDOUtils databases as follows:

1.  Retrieve the content and date/time of the latest ten notifications:

    ```
    mysql> SELECT start_time, long_output FROM nagios_notifications ORDER BY start_time DESC LIMIT 10;

    ```

2.  Retrieve the content and date/time of the latest ten host or service comments:

    ```
    mysql> SELECT entry_time, comment_data FROM nagios_commenthistory ORDER BY entry_time DESC LIMIT 10;

    ```

3.  Count the number of hosts currently in the `OK` state:

    ```
    mysql> SELECT COUNT(1) FROM nagios_hoststatus WHERE current_state = 0;

    ```

4.  Retrieve the names of all hosts currently in scheduled downtime:

    ```
    mysql> SELECT display_name FROM nagios_hosts JOIN nagios_hoststatus USING (host_object_id) WHERE scheduled_downtime_depth > 0;

    ```

    Note that the syntax of this query assumes a MySQL version of at least 5.0.12.

5.  Return a list of all host names and the number of services associated with them:

    ```
    mysql> SELECT nagios_hosts.display_name, COUNT(service_id) FROM nagios_hosts LEFT JOIN nagios_services ON nagios_hosts.host_object_id = nagios_services.host_object_id GROUP BY nagios_hosts.host_object_id;

    ```

We could implement a Perl script to print the latest ten notifications using the DBI module as follows:

```
#!/usr/bin/env perl

# Enforce Perl best practices
use strict;
use warnings;

# Import database modules
use DBI;
use DBD::mysql;

# Get connection to database
my $nagios = DBI->connect(
    'dbi:mysql:dbname=nagios;host=localhost',
    'ndo',
    'mPYxbAYqa'
) or die "Could not connect to database";

# Define an SQL query to run
my $query = q{
    SELECT
        notification_id, start_time, name1, long_output
    FROM
        nagios_notifications
    JOIN
        nagios_objects USING (object_id)
    ORDER BY
        start_time DESC
    LIMIT
        10
};

# Execute query and retrieve notifications
my $notifications = $nagios->selectall_hashref(
    $query,
    'notification_id'
) or die 'Could not retrieve notifications';

# Print each notification
foreach my $id (keys %{$notifications}) {
    my $notification = $notifications->{$id};
    printf {*STDOUT} "%s - %s: %s\n",
        $notification->{start_time},
        $notification->{name1},
        $notification->{long_output};
}

# Exit successfully
exit 0;
```

Saved into a file `latest-notifications.pl`, we could run it as follows:

```
$ chmod +x latest-notifications.pl
$ ./latest-notifications.pl
2012-10-06 10:11:27 - blog: PING OK - Packet loss = 0%, RTA = 160.31 ms
2012-10-06 10:48:17 - athens: PING OK - Packet loss = 0%, RTA = 155.82 ms
2012-10-06 14:08:17 - athens: PING WARNING - Packet loss = 16%, RTA = 171.79 ms
2012-10-06 14:13:17 - athens: PING OK - Packet loss = 0%, RTA = 164.39 ms
2012-10-06 10:56:17 - athens: PING CRITICAL - Packet loss = 28%, RTA = 164.65 ms
2012-10-06 10:38:17 - athens: PING CRITICAL - Packet loss = 28%, RTA = 166.10 ms
2012-10-06 10:06:27 - blog: PING WARNING - Packet loss = 16%, RTA = 163.72 ms
2012-10-06 13:39:27 - blog: PING WARNING - Packet loss = 16%, RTA = 163.78 ms
2012-10-06 13:44:27 - blog: PING OK - Packet loss = 0%, RTA = 167.10 ms
2012-10-06 13:29:27 - blog: PING CRITICAL - Packet loss = 28%, RTA = 159.55 ms

```

Similarly, we could implement a crude RSS feed for notifications using PHP5 with PDO MySQL as follows:

```
<?php
// Get connection to database
$nagios = new PDO(
    'mysql:host=localhost;dbname=nagios',
    'ndo',
    'mPYxbAYqa'
);
// Define an SQL query to run
$query = '
    SELECT
        start_time, name1, long_output
    FROM
        nagios_notifications
    JOIN
        nagios_objects
    USING
        (object_id)
    ORDER BY
        start_time DESC
    LIMIT
        10
';
// Retrieve all the notifications as objects
$statement = $nagios->prepare($query);
$statement->setFetchMode(PDO::FETCH_OBJ);
$statement->execute();
// Read the notifications into an array
for ($notifications = array(); $notification = $statement->fetch(); ) {
    $notifications[] = $notification;
}
// Send an RSS header rather than an HTML one
header("Content-Type: application/rss+xml; charset=utf-8");
echo '<?xml version="1.0" encoding="UTF-8"?>';
?>
<rss version="2.0">
    <channel>
        <title>
            Latest Nagios Core Notifications
        </title>
        <link>
            http://olympus.naginet/nagios/
        </link>
        <description>
            The ten most recent notifications from Nagios Core
        </description>
<? foreach ($notifications as $notification): ?>
        <item>
            <description>
                <?= htmlspecialchars($notification->name1); ?>: <?= htmlspecialchars($notification->long_output) ?>
            </description>
            <pubDate>
                <?= htmlspecialchars(date('r', strtotime($notification->start_time))) ?>
            </pubDate>
        </item>
<? endforeach; ?>
    </channel>
</rss>
```

Saved in a file named `latest-notifications.php`, we could subscribe to this in our favorite RSS reader, such as **Life** **rea** ([http://liferea.sourceforge.net/](http://liferea.sourceforge.net/)):

![How to do it...](img/5566_11_15.jpg)

## How it works...

The examples given in the preceding section are just to get you started with very simple reports; there is a wealth of data available in the NDOUtils database to explore. Here are some other possibilities:

*   A breakdown of all the hosts in your system and their states, ordered by name, presented in an HTML table
*   A list of all the hosts that have been down more than once in a month
*   The percentage of uptime for all hosts

## See also

*   The *Reading status into a MySQL database with NDOUtils* and *Getting extra visualizations with NagVis* recipes in this chapter

# Getting extra visualizations with NagVis

In this recipe, we'll explore how to go beyond the default network map discussed in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), to get a lot of visualization power with the extension NagVis. NagVis can use the NDOUtils backend to build custom maps in various styles.

**NagVis** is most likely of interest to you if you're interested in visualizing Nagios data more extensively, particularly if you're having problems with the scalability of the included Nagios Core status map. The default status map works well for smaller networks, but can struggle with rendering larger ones in a timely fashion.

A complete survey of NagVis' functions would not be possible in one recipe, but this one will walk you through downloading, installing, and configuring the extension to give you a simple **automap**, in order to get you started.

## Getting ready

You should have a running Nagios Core server with version 3.0 or later, and have the NDOUtils backend successfully installed and populating a MySQL database to which you have administrative access. This is discussed in the *Reading status into a MySQL database with NDOUtils* recipe in this chapter.

In order for the automap to be much use, you will need a network with at least a few parent-child relationships—see the *Creating a network host hierarchy* recipe in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout") for details on how this is done.

NagVis includes an installation script that deals quite well with different systems' installations of Nagios Core. However, it still requires certain dependencies, specifically:

*   Apache with `mod_php` on the same server as Nagios Core
*   PHP 5.3 or newer, with the following modules: `gd`, `gettext`, `mbstring`, `mysql`, `pdo`, `session`, `sqlite`, and `xml`
*   The Graphviz graph visualization software, with the following modules: `circo`, `dot`, `fdp`, `neato`, `twopi`
*   SQLite 3

You may need to consult your system's documentation to install all these dependencies; check your system's package manager, if there is one. For Ubuntu and other Debian-derived systems, the following packages generally suffice:

```
# apt-get install apache2 libapache2-mod-php5 libgd2-xpm libgd2-xpm-dev php5 php5-gd php5-mysql php5-sqlite graphviz sqlite3

```

On systems such as CentOS and Fedora, the following packages may work:

```
# yum install php php-gd php-gettext php-mbstring php-mysql php-pdo php-sqlite php-xml graphviz graphviz-gd graphviz-php

```

It is difficult to anticipate the exact packages needed for all systems; searching your package manager for keywords (for example `php sqlite`) may help.

## How to do it...

We can install NagVis with an NDO backend as follows:

1.  Download the latest sources for NagVis from [http://www.nagvis.org/downloads](http://www.nagvis.org/downloads):

    ```
    $ wget http://downloads.sourceforge.net/project/nagvis/NagVis1.7/nagvis-1.7.1.tar.gz

    ```

2.  Inflate `.tar.gz` and change directory to within it:

    ```
    $ tar -xzf nagvis-1.7.1.tar.gz
    $ cd nagvis-1.7.1

    ```

3.  Run the `install.sh` script as `root`:

    ```
    # ./install.sh

    ```

4.  The script will attempt to find your Nagios Core installation, and will ask you to specify a location for the new NagVis files. In our case, the defaults are correct and acceptable, so we can simply press *Enter*.

    ```
    -- Checking paths --
    Please enter the path to the nagios base directory [/usr/local/nagios]:
     nagios path /usr/local/nagios  found
    Please enter the path to NagVis base [/usr/local/nagvis]:

    ```

5.  The script will attempt to find all of the prerequisites needed and will alert you if any are not found. If this is the case, you should abort the installation with *Ctrl* + *C* and install them before trying again.

    ```
    -- Checking prerequisites --
    PHP 5.3                                  found
     PHP Module: gd 5.3                     found
     PHP Module: mbstring compiled_in       found
     PHP Module: gettext compiled_in        found
     PHP Module: session compiled_in        found
     PHP Module: xml compiled_in            found
     PHP Module: pdo compiled_in            found
     Apache mod_php                         found
    Graphviz 2.26                            found
     Graphviz Module dot 2.26.3             found
     Graphviz Module neato 2.26.3           found
     Graphviz Module twopi 2.26.3           found
     Graphviz Module circo 2.26.3           found
     Graphviz Module fdp 2.26.3             found
    SQLite 3.7                               found

    ```

6.  The script will prompt you for an appropriate backend to configure as NagVis' data source. In this example, the only one we want is the `ndo2db` backend. Press *n* for all the others.

    ```
    -- Checking Backends. (Available: mklivestatus,ndo2db,ido2db,merlinmy) --
    Do you want to use backend mklivestatus? [y]: n
    Do you want to use backend ndo2db? [n]: y
    Do you want to use backend ido2db? [n]: n
    Do you want to use backend merlinmy? [n]: n
     /usr/local/nagios/bin/ndo2db (ndo2db)  found

    ```

7.  The script will attempt to detect your Apache HTTPD settings. It does a good job with most systems, but you should check the results are correct before you press *Enter*. It should also be safe to allow it to create an Apache configuration file for you.

    ```
    -- Trying to detect Apache settings --
    Please enter the web path to NagVis [/nagvis]:
    Please enter the name of the web-server user [www-data]:
    Please enter the name of the web-server group [www-data]:
    create Apache config file [y]:

    ```

8.  The script will give you a summary of its intentions for installing the software, and will ask you to confirm. Do so by pressing *Enter*:

    ```
    -- Summary --
    NagVis home will be:           /usr/local/nagvis
    Owner of NagVis files will be: www-data
    Group of NagVis files will be: www-data
    Path to Apache config dir is:  /etc/apache2/conf.d
    Apache config will be created: yes
    Installation mode:             install
    Do you really want to continue? [y]:

    ```

9.  Restart the Apache HTTPD server:

    ```
    # apache2ctl configtest
    # apache2ctl restart

    ```

If all of the above goes correctly, once the installation is finished, you should be able to visit the NagVis configuration page on [http://olympus.naginet/nagvis/](http://olympus.naginet/nagvis/), substituting the hostname for your own Nagios Core server:

![How to do it...](img/5566_11_16.jpg)

You can log in with the default username `admin` and the password `admin` to take a look at some of the demo maps.

There is a little more to go yet before we can get our automap working:

1.  On the server, edit the `/usr/local/nagvis/etc/nagvis.ini.php` file:

    ```
    # vi /usr/local/nagvis/etc/nagvis.ini.php

    ```

2.  Find and change the following directives under the `backend_ndomy_1` section, adding the values you used for your `ndo2db` installation:

    ```
    ; hostname for NDO-db
    dbhost="localhost"
    ; portname for NDO-db
    dbport=3306
    ; database name for NDO-db
    dbname="nagios"
    ; username for NDO-db
    dbuser="ndo"
    ; password for NDO-db
    dbpass="mPYxbAYqa"
    ; prefix for tables in NDO-db
    dbprefix="nagios_"
    ; instance name for tables in NDO-db
    dbinstancename="default"
    ```

    Make sure that all the preceding values are uncommented (they should not be preceded by a semicolon).

3.  Log in to the NagVis web interface, and click on **Manage Maps** under the **Options** menu:![How to do it...](img/5566_11_17.jpg)
4.  Under **Create Map**:

    1.  For **Map name**, enter the value `Automap`.
    2.  For **Map iconset**, choose **std_small**.
    3.  Leave **Background** blank.
    4.  Click on **Create**.

    ![How to do it...](img/5566_11_18.jpg)

    The page should refresh to a blank screen, because we have not yet elected a data source for the map.

5.  Click on **Map Options** under the **Edit Map** menu.![How to do it...](img/5566_11_19.jpg)
6.  In the resulting dialog:

    1.  Check the **sources** checkbox, and change the value to `automap`.
    2.  Check the **backend_id** checkbox, and choose the value **ndomy_1**.
    3.  Scroll down to the bottom and click on **Save**.

    ![How to do it...](img/5566_11_20.jpg)

With this done, the page should refresh and show you a map of your network, automatically generated from your configuration, in a similar style to the Nagios Core web interface status map. You should also be able to hover over individual nodes to see their details.

![How to do it...](img/5566_11_21.jpg)

## How it works...

NagVis' automap is generated from the data in the database that we established in the NDOUtils recipe. It generates the map in much the same way that the default status map does, but is more scalable for larger networks. The parent and child relationships defined in the configuration are included, to make a tree-style map.

## There's more...

The use of NagVis could fill an entire book in itself, and the automap is only one of many possible maps, including defining one's own backgrounds, icons, labels, and hover behavior. For more detail on how to make customized maps as well as other styles of automaps, consult the NagVis documentation at [http://www.nagvis.org/doc](http://www.nagvis.org/doc).

## See also

*   The *Reading status into a MySQL database with NDOUtils* and *Writing customized Nagios Core reports* recipes in this chapter
*   The *Creating a network host hierarchy* and *Using the network map* recipes in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), *Understanding the Network Layout*