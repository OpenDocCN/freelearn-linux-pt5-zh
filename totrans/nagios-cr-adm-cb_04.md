# Chapter 4. Configuring Notifications

In this chapter, we will cover the following recipes:

*   Configuring notification periods
*   Configuring notification for groups
*   Specifying which states to be notified about
*   Tolerating a certain number of failed checks
*   Automating contact rotation
*   Defining an escalation for repeated notifications
*   Defining a custom notification method

# Introduction

**Notifications** in Nagios Core refer to the events fired and the messages generated when hosts or services change state, for the purposes of informing appropriate people or systems.

For example, when connectivity to a host is lost, it leaves the `UP` state and goes to the `DOWN` state on the next check. This eventually generates a notification event, and provided the appropriate flags are set and a contact is available, a message about the state change is generated and dealt with in some way as defined in the configuration.

The default text for such notifications goes into some detail about the problem, including the output from the appropriate plugin command:

```
***** Nagios *****
Notification Type: PROBLEM
Host: sparta.naginet
State: DOWN
Address: 10.128.0.21
Info: CRITICAL - Host Unreachable (10.128.0.21)
Date/Time: Sat May 19 16:27:39 NZST 2012

```

So, with this kind of text generated as a result of the event, the next question is what Nagios Core should do with it. A common method of notification that will be familiar to most Nagios Core administrators is sending this text as an e-mail message using the system mailer, but the same generated message can be used in a wide variety of ways. Just as commands can be flexibly defined in terms of the command lines they run, the action appropriate for a particular contact can be defined very flexibly to use the text of any notifications that a contact is configured to receive.

In this chapter, we'll learn how to refine notifications management in Nagios Core, to make sure that the appropriate people or systems are notified about appropriate network events, and not nagged about others which may not matter. We'll also learn how to set up notification methods beyond the simple e-mail message, and how to escalate notifications when problems remain unfixed after a certain period of time.

# Configuring notification periods

In this recipe, we'll adjust the configuration for a service that has been bugging us with notifications late at night. We'll arrange to keep checking this host, `sparta.naginet`, on a 24x7 basis, but we'll prevent it from sending notifications outside of work hours, using two of the predefined time periods in the default Nagios Core configuration.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file.

## How to do it...

We can define the `check_period` and `notification_period` plugins for our host as follows:

1.  Change to the objects configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing your host definition, and find the definition within the file:

    ```
    # vi sparta.naginet.cfg

    ```

    The host definition may look similar to the following code snippet:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21
    }
    ```

3.  Add or edit the value for the `check_period` directive to `24x7`:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21
     check_period         24x7
    }
    ```

4.  Add or edit the value for the `notification_period` directive to `workhours`:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21
        check_period         24x7
     notification_period  workhours
    }
    ```

5.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core will run its check of the host all the time (24 hours a day, 7 days a week), including logging the `UP` and `DOWN` states and showing them in the web interface and reporting, but will only send notifications to the nominated contact for the host during work hours, by default from 9 AM to 5 PM, Monday to Friday.

## How it works...

The preceding configuration changed two properties of the host object type, to affect the changes we needed:

*   `check_period`: This defines the time periods during which the host should be checked; we chose the predefined period `24x7`, meaning checks against the host are always to be run, regardless of the time of day or night.
*   `notification_period`: This defines the time periods during which the host should send notifications to the appropriate contact; we chose the predefined period `workhours`, meaning that notifications will only be sent from 9 AM to 5 PM, Monday to Friday.

The same configuration could be applied exactly to a service, rather than a host:

```
define service {
    ...
 check_period         24x7
 notification_period  workhours
}
```

The two time periods to which this new configuration refers are themselves defined in the Nagios Core configuration. The ones we've used, `24x7` and `workhours`, are standard time periods included in the defaults, kept in the `/usr/local/nagios/etc/objects/timeperiods.cfg` file. The definition for `workhours`, for example, looks similar to the following code snippet:

```
define timeperiod {
    timeperiod_name  workhours
    alias            Normal Work Hours
    monday           09:00-17:00
    tuesday          09:00-17:00
    wednesday        09:00-17:00
    thursday         09:00-17:00
    friday           09:00-17:00
}
```

The ones we've used here are just examples of common time periods. We can define time periods ourselves with great precision; see the *Creating a new time period* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts* for some detail on how to do this.

## There's more...

The distinction between the two directives is very important. Under most circumstances, we will want a `check_period` directive that corresponds to all times for services that are up day and night, because it means we can retain information about long-term uptime for the service. Keeping a distinct `notification_period` directive simply allows us to control when notifications should be sent, perhaps because they would otherwise go to a pager and wake up a grumpy systems administrator for a relatively unimportant system!

It's valid for Nagios Core configuration to set a `check_period` plugin for a reduced time period, for example `workhours`. However, you shouldn't need to do this unless your host is actually going to be down outside these hours. In most cases, it's appropriate to use the `24x7` time period as a value for `check_period`.

## See also

*   The *Configuring notification for groups* recipe in this chapter
*   The *Creating a new time period* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*
*   The *Specifying how frequently to check a host or service* recipe in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*

# Configuring notification for groups

In this recipe, we'll learn how to define a contact group for a host. We'll demonstrate this as part of the general best practice of sending notifications to contact groups, rather than to individual contacts, which allows for more flexibility in assigning individual contacts to the appropriate groups for receiving appropriate notifications.

## Getting ready

You should have a Nagios Core 3.0 or newer server with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file, which we'll configure to send its notifications to an existing contact group called `noc`.

## How to do it...

We can configure notifications to be sent to our contact group as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Ensure that an appropriate `contact_group` definition exists for the group to which you intend to direct your notifications. A valid definition might look something similar to the following code snippet, perhaps kept in `contacts.cfg`:

    ```
    define contactgroup {
        contactgroup_name  noc
        alias              Network Operations
        members            john,jane
    }
    ```

3.  If any contacts are referenced for the group, then you should ensure those are also defined. They might look similar to the following code snippet:

    ```
    define contact {
        use            generic-contact
        contact_name   john
        alias          John Smith
        email          john@naginet
    }
    define contact {
        use            generic-contact
        contact_name   jane
        alias          Jane Doe
        email          jane@naginet
    }
    ```

4.  Edit the file containing your host definition, and find the definition within the file:

    ```
    # vi sparta.naginet.cfg

    ```

    The host definition may look similar to the following code snippet:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21notification_period  24x7
    }
    ```

    Add or edit the value for the `contact_groups` directive to `noc`:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21
        notification_period  24x7
     contact_groups       noc
    }
    ```

5.  If contacts for the host are already defined, then you may wish to remove them, though you don't have to.
6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, all notifications for this host should be sent to each member of the `noc` group; in our example, the contacts are `jane` and `john`.

## How it works...

Defining the contacts that should receive notifications for particular hosts can be done by referring to them individually, or as groups, or both. We could have managed the same result as the preceding one by defining the `contacts` directive for the host as `jane,john`, referring to the individual contacts directly, rather than by their group name.

Almost the same configuration process applies for setting the contact group for services, rather than hosts; we add a value for the `contact_groups` directive to the `service` definition:

```
define service {
    ...
 contact_groups  noc
}
```

When an event takes place for the host or service that prompts a notification, and the valid `notification_option` flags are set for that event, Nagios Core will check the values of both the `contacts` and the `contact_groups` directives for the host, to determine to whom to send the notification. In our case, the server finds the value of `contact_groups` to be `noc`, and so refers to that contact group to identify the individual contacts within it, and then sends the notification to each of them.

## There's more...

Which of the two methods to use for defining contacts for hosts or services, whether by individual contact references or groups, is up to you. For large numbers of hosts and services, it tends to be easier to direct notifications to appropriate groups rather than individuals, as it affords us more flexibility in choosing to whom notifications will be sent simply by changing the individuals in the group.

You need to define at least one contact or contact group for a host or service, or have it inherit one through a template, for it to be a valid configuration for Nagios Core.

## See also

*   The *Configuring notification periods*, *Specifying which states to be notified about*, and *Automating contact rotation* recipes in this chapter
*   The *Creating a new contact* and *Creating a new contact group* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Specifying which states to be notified about

In this recipe, we'll learn how to refine which notifications are sent by a host or service, and which notifications should be received by a particular contact. We'll do this by changing both the notification types that hosts and services should generate, and to complement that, the notification types that a contact should be configured to receive.

## Getting ready

You should have a Nagios Core 3.0 or newer server, with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file, which we'll configure to send only `DOWN` and `RECOVERY` notifications, ignoring other notifications such as `WARNING` and `UNKNOWN`. It will send notifications to an existing contact called `john`; we'll ensure that this contact is configured to receive those notifications.

## How to do it...

We can configure the notification types for our host as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for your host. It might look similar to the following code snippet:

    ```
    define host {
        use                    linux-server
        host_name              sparta.naginet
        alias                  sparta
        address                10.128.0.21
        contacts               john
        notification_period    24x7
    }
    ```

3.  Add a value for the `notification_options` directive. In our case, we'll use `d,r`, to correspond to notifications for `DOWN` and `RECOVERY` states:

    ```
    define host {
        use                    linux-server
        host_name              sparta.naginet
        alias                  sparta
        address                10.128.0.21
        contacts               john
        notification_period    24x7
     notification_options   d,r
    }
    ```

4.  We should also ensure that notifications for this host are enabled, specifically that `notifications_enabled` is set to `1`. An easy way to set this and other relevant directives is to have the host inherit from a template such as `linux-server`, as done previously, but we can set it explicitly if we prefer:

    ```
    define host {
        use                    linux-server
        host_name              sparta.naginet
        alias                  sparta
        address                10.128.0.21
        contacts               john
        notification_period    24x7
        notification_options   d,r
     notifications_enabled  1
    }
    ```

5.  Edit the file containing the definition for the host's nominated contacts, or the contacts within its nominated contact group. In this case, we're editing the host's nominated contact `john`. The definition might look similar to the following code snippet:

    ```
    define contact {
        use           generic-contact
        contact_name  john
        alias         John Smith
     email         john@naginet
    }
    ```

6.  Edit or set the directive for `host_notification_options`, for each contact, to include the same values that you set for the host. In this case, we set it to `d`, `u`, `r`, which means that `John Smith` can receive the `DOWN`, `UNREACHABLE`, and `RECOVERY` notifications about hosts:

    ```
    define contact {
        use                        generic-contact
        contact_name               john
        alias                      John Smith
        email                      john@naginet
     host_notification_options  d,u,r
    }
    ```

7.  Again, you should ensure that the contacts are configured to receive host notifications, using the `host_notifications_enabled` directive being set to `1`. We can do this via inheritance from a template such as `generic-contact`, which tends to be easier, or we can set it explicitly:

    ```
    define contact {
        use                         generic-contact
        contact_name                john
        alias                       John Smith
        email                       john@naginet
        host_notification_options   d,r
     host_notifications_enabled  1
    }
    ```

    The analogous directive for receiving service notifications is `service_notifications_enabled`.

8.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, notifications sent when the host enters the `DOWN` state should be sent to the contact, and also when it recovers to the `OK` state, but no other notifications about the host (such as the `UNREACHABLE` or `DOWNTIMESTART` notifications) should be sent about the `sparta.naginet` host to the contact `john`.

## How it works...

The kinds of notifications a host should produce is defined in its `notification_options` directive, and the kind of host notifications that a defined contact should receive is configured in its `host_notification_options` directive. Both take the form of comma-separated values.

The valid values for the `notification_options` directive for host objects are:

*   `d`: Notifications for a host being in the `DOWN` state
*   `u`: Notifications for a host being in the `UNREACHABLE` state
*   `r`: Notifications for a host recovering from problems (becoming `UP`)
*   `f`: Notifications for a host beginning or ending the flapping state
*   `s`: Notifications for a host beginning or ending scheduled downtime

For example, for a host that should only send notifications when entering the `DOWN` or `UP` states, but ignoring notification events for `UNREACHABLE` or flapping, we might define the following:

```
define host {
    ...
 notification_options d,r
}
```

The valid directives are slightly different for service objects:

*   `w`: Notifications for a service being in the `WARNING` state
*   `u`: Notifications for a service being in the `UNKNOWN` state
*   `c`: Notifications for a service being in the `CRITICAL` state
*   `r`: Notifications for a service recovering from problems (becoming `OK`)
*   `f`: Notifications for a service beginning or ending a flapping state
*   `s`: Notifications for a service beginning or ending scheduled downtime

For example, for a host that should send notifications for all of these events except for flapping and scheduled downtime states, we might define the following:

```
define service {
    ...
 notification_options  w,u,c,r
}
```

All of the preceding values can be used in contact object definitions to restrict the service notifications a particular contact should receive, through the `service_notification_options` directive:

```
define contact {
    ...
 service_notifications_enabled  1
 service_notification_options   w,u,c,r
}
```

## There's more...

Unless we have a specific reason for particular contacts not to accept notifications of a particular kind, it's a good practice to configure contacts to receive all notifications that they are sent, by including all the flags in both the directives for the contact:

```
define contact {
    ...
 host_notification_options     d,u,r,f,s
 service_notification_options  w,u,c,r,f,s
}
```

This means that the contact will process any notification dispatched to it, and we can instead restrict the kinds of notifications that are sent out by the hosts and services for which this is the delegated contact. With new configurations, it is generally better to start by sending too much information, and then removing unnecessary notification flags if appropriate, rather than potentially missing important messages because they're never sent.

If we intend to do this for all of our contacts, it may be appropriate to include these directives as part of a contact template, and inherit from it using the `use` directive. The `generic-contact` contact template included with the default configuration may be suitable.

If you want to completely disable notifications for a given host, service, or contact, the single value `n` for `notification_options`, `host_notification_options`, and `service_notification_options` will do this.

## See also

*   The *Configuring notification for groups*, *Automating contact rotation*, and *Defining a custom notification method* recipes in this chapter
*   The *Using inheritance to simplify configuration* recipe in [Chapter 9](ch09.html "Chapter 9. Managing Configuration"), *Configuration Management*

# Tolerating a certain number of failed checks

In this recipe, we'll learn how to arrange Nagios Core configuration to only send notifications about problems with a host or service after a check has been repeated a certain number of times and failed each time.

This can be an ideal arrangement for non-critical hosts that occasionally have "blips" or short outages for whatever reason, and only become problematic if they remain down after repeated checks.

## Getting ready

You should have a Nagios Core 3.0 or newer server, with at least one host configured already. We'll use the example of `sparta.naginet`, a host defined in its own file. We'll arrange for it to send us notifications only after a total of five failed host checks.

## How to do it...

We can configure the number of failed checks to tolerate before sending a notification as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing your host definition, and find the definition within it. It may look similar to the following code snippet:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21
        notification_period  24x7
    }
    ```

3.  Add or edit the value for `max_check_attempts` to `5`:

    ```
    define host {
        use                  linux-server
        host_name            sparta.naginet
        alias                sparta
        address              10.128.0.21
        notification_period  24x7
     max_check_attempts   5
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core will not send a notification for the host entering the `DOWN` state until the check has been attempted a total of five times, and has failed each time. In the following screenshot, even though four checks have already taken place and failed, no notification has been sent, and the state is listed as **SOFT** :

![How to do it...](img/5566_04_01.jpg)

The `SOFT` state shows that while Nagios Core has flagged the host as `DOWN`, it will retry the check until it exhausts `max_check_attempts`. At this point it will flag the host as being in a `HARD` `DOWN` state and send a notification.

However, if the host were to come back up before the next check, then the state would change back to **UP**, without ever having sent a notification:

![How to do it...](img/5566_04_02.jpg)

## How it works...

The preceding configuration alters the `max_check_attempts` directive for the host to specify the number of checks that need to have failed before the host will flag a `HARD` `DOWN` or `UNREACHABLE` state, and generate a notification event.

The process for changing the maximum number of attempts before a notification for a service is identical; we add the same directive and value to the definition for the service:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  HTTP
    check_command        check_http
 max_check_attempts   5
}
```

## There's more...

The time between successive retry checks of a host before sending any notification can also be customized with the `retry_interval` directive. By default, the interval is in minutes, so if we wanted to configure a two-minute wait between retry checks, we could add this directive to the host or service:

```
define host {
    ...
 retry_interval  2
}
define service {
    ...
 retry_interval  2
}
```

## See also

*   The *Specifying how frequently to check a host or service* and *Managing brief outages with flapping* recipes in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*

# Automating contact rotation

In this recipe, we'll learn how to automatically arrange notifications so that they are only received by certain contacts in a group at certain times.

For companies with more than one person as dedicated network staff, it's a common practice to have an on-call roster, where one member of staff will be the dedicated on-call person for a certain period of time, perhaps a week or two. The kind of setup we'll configure here allows filtering notifications, so that they're only delivered to contacts at appropriate times; notifications are generated by hosts and services and sent to all the contacts in the nominated group, but only one (or perhaps more than one) of the contacts actually receives it.

We'll set this up using two properties of contacts that allow us to restrict the time period within which they should receive notifications: `host_notification_period` and `service_notification_period`.

## Getting ready

You should have a Nagios Core 3.0 or newer server, with at least one contact group configured already, with at least two contacts within it. You should be familiar with configuring time periods and understand the basics of notifications as discussed earlier in this chapter.

We'll use the very simple example of three network operators, named as contacts `alan`, `brenden`, and `charlotte`, taking turns with monitoring duty every week. All three are members of the group `ops`, which is specified in the `contact_groups` directive for all of the hosts and services on the network.

## How to do it...

We can set up a basic alternating schedule for receiving notifications for our operators as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Check the relevant file or files to ensure your contacts are defined, and that they are members of the appropriate contact group. A good location for such information is in the `contacts.cfg` file. The configuration might look similar to the following code snippet:

    ```
    define contactgroup {
        contactgroup_name  ops
        alias              Network Operations
        members            alan,brenden,charlotte
    }
        define contact {
            use           generic-contact
            contact_name  alan
            alias         Alan Jones
            email         alan@pager.naginet
        }
        define contact {
            use           generic-contact
            contact_name  brenden
            alias         Brenden Peters
            email         brenden@pager.naginet
        }
        define contact {
            use           generic-contact
            contact_name  charlotte
            alias         Charlotte Franks
            email         charlotte@pager.naginet
        }
    ```

3.  In this case, all our hosts and services are configured to send messages to the `ops` contact group:

    ```
        define host {
            ...
     contact_groups  ops
        }
        define service {
            ...
     contact_groups  ops
        }
    ```

4.  Define time periods to correspond to the times during which each of your contacts should be receiving messages. We'll use the special syntax of time period definitions to set up the definitions for this rotation:

    ```
    define timeperiod {
        timeperiod_name  alan-pageralias            Alan pager schedule
        2012-06-05 / 21  00:00-24:00
        2012-06-06 / 21  00:00-24:00
        2012-06-07 / 21  00:00-24:00
        2012-06-08 / 21  00:00-24:00
        2012-06-09 / 21  00:00-24:00
        2012-06-10 / 21  00:00-24:00
        2012-06-11 / 21  00:00-24:00
    }
    define timeperiod {
        timeperiod_name  brenden-pageralias            Brenden pager schedule
        2012-06-12 / 21  00:00-24:00
        2012-06-13 / 21  00:00-24:00
        2012-06-14 / 21  00:00-24:00
        2012-06-15 / 21  00:00-24:00
        2012-06-16 / 21  00:00-24:00
        2012-06-17 / 21  00:00-24:00
        2012-06-18 / 21  00:00-24:00
    }
    define timeperiod {
        timeperiod_name  charlotte-pager
        alias            Charlotte pager schedule
        2012-06-19 / 21  00:00-24:00
        2012-06-20 / 21  00:00-24:00
        2012-06-21 / 21  00:00-24:00
        2012-06-22 / 21  00:00-24:00
        2012-06-23 / 21  00:00-24:00
        2012-06-24 / 21  00:00-24:00
        2012-06-25 / 21  00:00-24:00
    }
    ```

5.  Go back and configure each of your contacts with the `host_notification_period` and `service_notification_period` directives, to filter notifications to only be received during their dedicated time period:

    ```
    define contact {
        use                          generic-contact
        contact_name                 alan
        alias                        Alan Jones
        email                        alan@pager.naginet
     host_notification_period     alan-pager
     service_notification_period  alan-pager
    }
    define contact {
        use                          generic-contact
        contact_name                 brenden
        alias                        Brenden Peters
        email                        brenden@pager.naginet
     host_notification_period     brenden-pager
     service_notification_period  brenden-pager
    }
    define contact {
        use                          generic-contact
        contact_name                 charlotte
        alias                        Charlotte Franks
        email                        charlotte@pager.naginet
     host_notification_period     charlotte-pager
     service_notification_period  charlotte-pager
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the contacts configured should only receive notifications during their dedicated pager time. If these are our only contacts, then all the notifications generated will go to only one person at any given time.

## How it works...

The `host_notification_period` and `service_notification_period` directives in the definitions for our contacts are used to limit the times during which these contacts should receive any notifications. Each of our contacts in this example has their own time period defined to correspond to their portion of the pager schedule.

Let's take a look at Alan's example:

```
    define timeperiod {
        timeperiod_name  alan-pager
        2012-06-05 / 21  00:00-24:00
        2012-06-06 / 21  00:00-24:00
        2012-06-07 / 21  00:00-24:00
        2012-06-08 / 21  00:00-24:00
        2012-06-09 / 21  00:00-24:00
        2012-06-10 / 21  00:00-24:00
        2012-06-11 / 21  00:00-24:00
    }
```

The second line of the directive, `2012-06-05 / 21 00:00-24:00`, can be broken down as follows:

*   Starting from the 5th of June,
*   Every 21 days,
*   From midnight to the following midnight (i.e., the entire day).

The configuration then goes on to do the same for the remaining days of the first week Alan will be on call, and specifies that he will be on call the same day 21 days (3 weeks) later. A similar configuration, but starting one week and two weeks later, is set up for the contacts `brenden` and `charlotte` respectively.

## There's more...

Note that there's no reason time periods can't overlap. If it happens to suit us to arrange for more than one contact to receive a notification, we can do that. Similarly, we can leave gaps in the schedule if notifications aren't needed during a certain time period.

Time periods in Nagios Core are highly flexible; to see some examples of the various syntaxes you can use for defining them, take a look at the *Creating a new time period* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

## See also

*   The *Configuring notification for groups* and *Specifying which states to be notified about* recipes in this chapter
*   The *Creating a new contact*, *Creating a new contact group*, and *Creating a new time period* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Defining an escalation for repeated notifications

In this recipe, we'll learn how to arrange a Nagios Core configuration such that after a certain number of repetitions, notifications for problems on hosts or services are escalated to another contact, instead of (or in addition to) the normally defined contact. This is done by defining a separate object type called a host or service escalation.

This kind of setup could be useful to alert more senior networking staff to an unresolved problem that a less experienced person is struggling to fix, and can also function as a "safety valve" to ensure that problem notifications for hosts eventually do reach someone else if they remain unfixed.

## Getting ready

You should have a Nagios Core 3.0 or newer server, with at least one host or service configured already, and at least two contact groups—one for the first few notifications, and one for the escalations. You should understand how notifications are generated and sent to the `contacts` and `contact_groups` for hosts or services.

We'll use the example of a host called `sparta.naginet` which normally sends notifications to a group called `ops`. We'll arrange for all of the notifications after the fourth one to also be sent to a contact group called `emergency`.

## How to do it...

We can configure an escalation for our host or service as follows:

1.  Change to the objects configuration directory for Nagios Core. The default is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the host. The definition might look similar to the following code snippet:

    ```
    define host {
        use                    linux-server
        host_name              sparta.naginet
        alias                  sparta
        address                10.128.0.21
        contact_groups         ops
        notification_period    24x7
        notification_interval  10
    }
    ```

3.  Underneath the host definition, add the configuration for a new `hostescalation` object:

    ```
    define hostescalation {
        host_name              sparta.naginet
        contact_groups         ops,emergency
        first_notification     5
        last_notification      0
        notification_interval  10
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, when problems are encountered with the host that generate notifications, all the notifications beyond the fifth one will be sent to both the `ops` contact group and also to the `emergency` contact group, expanding the number of people or systems contacted, to make it more likely the problem will actually be addressed or fixed. Perhaps someone on the ops team has misplaced their pager.

## How it works...

The configuration added in the preceding section is best thought of as a special case or an override for a particular host, in that it specifies a range of notifications that should be sent to a different set of contact groups. It can be broken down as follows:

*   `host_name`: This is the same value of `host_name` given for the host in its definition; we specified `sparta.naginet`.
*   `contact_groups`: This is the contact groups to which notifications meeting this special case should be sent. We specify both the `emergency` group and the `ops` group, so that matching notifications go to both. Note that they are comma-separated.
*   `first_notification`: This is the count of the first notification that should match this escalation; we chose the fifth notification.
*   `last_notification`: This is the count of the last notification that should match this escalation. We set this to zero, which means that all notifications after `first_notification` should be sent to the nominated contact or contact groups. The notifications will not stop until they are manually turned off, or the problem is fixed.
*   `notification_interval`: Like the host and service directives of the same name, this specifies how long Nagios Core should wait before sending new notifications if the host remains in a problematic state. Here, we've chosen ten minutes.

Individual contacts can also (or instead) be specified with the `contacts` directive, rather than `contact_groups`.

Service escalations work in much the same way; the difference is that you need to specify the service by its `service_description` directive as well as its host name. Everything else works the same way. A similar escalation for a service check with a `service_description` of `HTTP` running on `sparta.naginet` might look similar to the following code snippet:

```
define serviceescalation {
    host_name              sparta.naginet
    service_description    HTTP
    contact_groups         ops,emergency
    first_notification     5
    last_notification      0
    notification_interval  10
}
```

## There's more...

The preceding escalation continues sending notifications both to the original `ops` group and also to the members of the `emergency` group. It's generally a good idea to do it this way rather than sending notifications only to the escalated group, because the point of escalations is to increase the reach of the notifications when a problem is not being dealt with, rather than merely to try contacting a different group of people instead.

This principle applies to stacking escalations, as well; if we had a host group with all of our contacts in it, perhaps named `everyone`, we could define a second escalation so that the tenth notification onwards goes to every single contact:

```
define hostescalation {
    host_name              sparta.naginet
    contact_groups         everyone
    first_notification     10
    last_notification      0
    notification_interval  10
}
```

Just as we can specify multiple host escalations, it's also fine for the ranges of notifications to overlap so that more than one escalation applies.

With a little arithmetic, you can arrange escalations such that they work after a host or service has been in a problematic state for a certain period of time. For example, the escalation we specified in the recipe will apply after the host has been in a problem state for 40 minutes, because `notification_interval` specifies that Nagios Core should wait for 10 minutes between re-sending notifications.

## See also

*   The *Configuring notifications for groups* recipe in this chapter
*   The *Creating a new contact group* recipe in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Defining a custom notification method

In this recipe, we'll learn how to specify an alternative method for a contact to receive notifications about a service. A very typical method for a contact to receive notifications is by sending an e-mail to their contact address; e-mail messages could be sent to an inbox or a paging device.

However, notifications are just text; we can arrange to deal with them via any command we wish, in much the same way as we can configure host or service checks. In this recipe, we'll set up a new contact called `motd`, which when it receives notifications will write them into the server's `/etc/motd` directory to be displayed on login.

## Getting ready

You should have a Nagios Core 3.0 or newer server, with at least one host or service configured already. You should understand how notifications are generated and their default behavior in being sent to the `contacts` and `contact_groups` for hosts or services.

We'll use the example of a host called `troy.naginet`, configured to send notifications to the `ops` contact group. We'll add a new contact within this group called `motd`.

## How to do it...

We can arrange our custom notification method as follows:

1.  Ensure that your `/etc/motd` file can be written to by Nagios Core, and that it's a static file that your system will not overwrite on boot. If you run your Nagios Core server as the recommended default user and group of `nagios`, you could do something similar to the following:

    ```
    # chgrp nagios /etc/motd
    # chmod g+w /etc/motd

    ```

2.  It would be a good idea to test that the user can write to the file using `su` or `sudo`:

    ```
    # sudo -s -u nagios
    $ echo 'test' >>/etc/motd

    ```

3.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

4.  Edit the file containing the definitions for your notification commands. By default, this is in the file `commands.cfg`. Append the following definitions to the file:

    ```
    define command {
        command_name    notify-host-motd
        command_line    /usr/bin/printf "%s\n" "$SHORTDATETIME$: $HOSTALIAS$ is $HOSTSTATE$" >>/etc/motd
    }
    define command {
        command_name    notify-service-motd
        command_line    /usr/bin/printf "%s\n" "$SHORTDATETIME$: $SERVICEDESC$ on $HOSTALIAS$ is $SERVICESTATE$" >>/etc/motd
    }
    ```

5.  Edit the file containing the definitions for your contacts. In the QuickStart configuration, this is in the file `contacts.cfg`. Append the following definitions to the file:

    ```
    define contact {
        use                            generic-contact
        contact_name                   motd
        contact_groups                 ops
        alias                          MOTD
        host_notification_commands     notify-host-motd
        service_notification_commands  notify-service-motd
    }
    ```

6.  Edit the file containing the definitions for your host and/or services, to ensure that they specify `ops` in their `contact_groups` directives:

    ```
    define host {
        host_name       troy.naginet
        ...
     contact_groups  ops
    }
        define service {
            host_name       troy.naginet
            ...
     contact_groups  ops
        }
    ```

7.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the next notifications that go out for the host and services should not only be e-mailed to any contacts in the `ops` group, but should also be written by the `motd` contact to `/etc/motd`, to be presented on login:

```
06-30-2012 17:24:05: troy is DOWN
06-30-2012 17:24:05: HTTP on troy is CRITICAL
06-30-2012 17:25:07: troy is DOWN
06-30-2012 17:25:07: HTTP on troy is CRITICAL
```

## How it works...

The first part of the configuration that we added is defining new notification commands. These are actually command definitions, just the same as those used with the `check_command` definitions for hosts and services; the difference is that they're used by contacts to send notifications to people (or in this case, write them to files) in the appropriate way.

If you're using the default Nagios Core configuration, the commands already defined in `commands.cfg` should include `notify-host-by-email` and `notify-service-by-email`. I've truncated the complete command line as it's very long:

```
define command {
    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification...
}
define command {
    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification...
}
```

These commands use the system `printf` implementation along with many Nagios Core macros to build an appropriate notification string, which is e-mailed to the appropriate contact using a shell pipe into the system mailer.

If we inspect the `templates.cfg` file where the `generic-contact` template is defined, we can see these methods are referred to in two directives:

```
define contact {
    ...
    service_notification_commands notify-service-by-email
    host_notification_commands notify-host-by-email
}
```

This configuration thus defines a particular notification command to be used for hosts that derive from the `generic-host` template.

Our own command definition does something similar in building a string with a call to `printf`. But instead of writing the output into a pipe to the system mailer, it appends it to the `/etc/motd` file.

The `command_line` directive that we specified for the `motd` contact to process host notifications is as follows:

```
/usr/bin/printf "%s\n" "$SHORTDATETIME$: $HOSTALIAS$ is $HOSTSTATE$" >>/etc/motd

```

The first thing Nagios Core does when a notification event for this contact is successfully fired is substitute values for its macros, `$SHORTDATETIME$`, `$HOSTALIAS$`, and `$HOSTSTATE$`:

```
/usr/bin/printf "%s\n" "06-30-2012 17:24:05: troy is DOWN" >>/etc/motd

```

The resulting string is then executed as a command by the running Nagios Core user, by default `nagios`, and providing that user has the appropriate permissions, it's appended to the end of the MOTD.

## There's more...

Adding messages to the MOTD may not be useful for all network administrators, particularly for larger or more sensitive networks, for which hundreds of notifications could be generated daily. The main thing to take away from this is that Nagios Core can be configured to process notification events in pretty much any way, provided:

*   We can define a command line that will consistently do what is needed with what the results of the macro substitution Nagios Core will perform for us
*   The `nagios` user, or whichever user as which the Nagios Core server is running, has all the relevant permissions it needs to run the command

E-mail notifications, particularly to a mobile paging device, are a very sensible basis for notifications for most administrators. However, if some other means of notification is needed, such as inserting into a database or piping into a Perl script, it is possible to do this as well.

Note that it's safest to specify the full paths to any binaries or file we use in these commands, hence `/usr/bin/printf` rather than merely `printf`. This is to ensure that the program we actually intend to run is run, and avoids having to mess about with the `$PATH` specifications for the `nagios` user.

## See also

*   The *Configuring notifications for groups* recipe in this chapter
*   The *Creating a new command* and *Customizing an existing command* recipes in [Chapter 2](ch02.html "Chapter 2. Working with Commands and Plugins"), *Working with Commands and Plugins*