# Chapter 7. Using the Web Interface

In this chapter, we will cover the following recipes:

*   Using the Tactical Overview
*   Viewing and interpreting availability reports
*   Viewing and interpreting trends
*   Viewing and interpreting notification history
*   Adding comments on hosts or services in the web interface
*   Viewing configuration in the web interface
*   Scheduling checks from the web interface
*   Acknowledging a problem via the web interface

# Introduction

The Nagios Core web interface is an administrator's first port of call to see the current status of the network being monitored. By way of CGI and PHP scripts, it allows an overview of the general performance of all the hosts and services being monitored. It also provides information about their current states and how checks are performed. This allows considerably more detail than that which is normally contained in e-mailed notifications.

![Introduction](img/5566_07_01.jpg)

In most respects, the Nagios Core web interface (without any addons) is designed for the display of information rather than controlling or configuring the server, but there are some things that can be done from it to actually change the way Nagios Core is running. These include the following:

*   Disabling or enabling active and passive checks, event handlers, notifications, and flap detection, whether for all hosts and services or specific ones
*   Sending custom notifications about a host or service that don't necessarily correspond to an actual state change
*   Rescheduling checks, commonly done to prioritize a check when there have been problems that the administrator thinks are now resolved and wishes to recheck
*   Acknowledging problems, to allow administrators to suppress further notifications from problems that they're working on fixing
*   Scheduling downtime to suppress notifications for a certain time period; this was discussed in the *Scheduling downtime for a host or service* recipe in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*
*   Leaving comments on hosts and services for the information of any other administrator who inspects them; many of the other actions in this list will do this implicitly

We won't offer a comprehensive survey of the web interface here, as much of its use is explained in other chapters, or otherwise reasonably self-explanatory. Instead, we'll explore a few runtime behavior changes that we can arrange through the web interface listed in the preceding section, and on using and interpreting the reports available under the **Reports** heading of the left navigation menu.

Another important and extremely useful part of the web interface, the **Network Status Map**, is discussed in several recipes in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), *Managing Network Layout*. We won't discuss it here, as it's so useful that it merits a chapter in itself.

# Using the Tactical Overview

In this recipe, we'll take a look at the **Tactical** **Overview** screen of the Nagios Core web interface. As its name implies, this screen provides a one-page summary of the current operating status of both the monitored hosts and services and the Nagios Core server itself.

## Getting started

You will need access to the Nagios Core web interface. In the QuickStart install, the `nagiosadmin` user will have all of the necessary privileges.

## How to do it...

We can take a look at **Tactical Overview** as follows:

1.  Log in to the Nagios Core web interface.
2.  Click the **Tactical Overview** item in the left menu:![How to do it...](img/5566_07_02.jpg)

    You should see the **Tactical Monitoring Overview** screen appear in the right frame:

    ![How to do it...](img/5566_07_03.jpg)
3.  Try clicking on some of the items under **Hosts** and **Services**, for example the **Up** count under **Hosts**. Note that you're presented with a listing of all the hosts that comprise that section.
4.  Return to **Tactical Overview**, and try clicking on some of the items under the **Monitoring Features** heading, for example the **Enabled** item under **Flap Detection**. Note that clicking on these brings you to a screen where you can quickly turn the relevant feature on or off (provided your user has the appropriate privileges).

## How it works...

The Tactical Overview is a good first port of call whenever you become aware of problems with the monitored network, particularly if you've a reason to believe that the problems are on more than one host. It's also a good place to visit to adjust monitoring features. For example, if a very large segment of your network is going to be down and it's not practical to schedule downtime for all of the affected hosts, then you could simply suppress notifications completely for a while in order to fix the problem with no distractions.

Note that the screen includes an overall assessment of the monitoring performance of the server in the top-right corner; this can be a useful first place to look if you suspect that the monitoring server is struggling to keep up with its tasks, or want to get some idea of how many checks it's making.

Beneath that is a simple bar graph showing the general state of the network. It will change its color depending on the severity of the problems; in the preceding screenshot, there are serious problems with some of the hosts and services on this network, and as a result the bar is short and red.

## There's more...

If you find the Tactical Overview useful, it may be a good idea to make it the home page for the web interface, rather than the default page which mostly just shows the Nagios Core branding, version, and some links. We can arrange this by changing the value of `$corewindow` in `/usr/local/nagios/share/index.php` from `main.php` to `cgi-bin/tac.cgi`:

```
$corewindow=”cgi-bin/tac.cgi”;

```

With this done, when we visit the `/nagios/` URL on the monitoring host, the **Tactical Overview** should come up immediately. This can be done with any of the pages in the menu. The **Services** item is another good choice for an alternative homepage.

## See also

*   The *Viewing configuration in the web interface* recipe in this chapter

# Viewing and interpreting availability reports

In this recipe, we'll learn how to use the **Availability Report** to build a table showing uptime statistics for a host, hostgroup, service, or servicegroup. This is useful as a quick metric of overall availability, perhaps to meet the terms of a service-level agreement.

## Getting started

You will need access to the Nagios Core web interface, and permission to run commands from the CGIs. The sample configuration installed by following the Quick Start Guide grants all the necessary privileges to the `nagiosadmin` user when authenticated via `HTTP`.

If you find that you don't have this privilege, then check the `authorized_for_all_services` and `authorized_for_all_hosts` directives in `/usr/local/nagios/etc/cgi.cfg`, and include your username in both; for example, for the user `tom`, the directives might look similar to the following:

```
authorized_for_all_servicess=nagiosadmin,tom
authorized_for_all_hosts=nagiosadmin,tom
```

Alternatively, you should also be able to see a host or service's information if you are authenticating with the same username as the nominated contact for the host or service you want to check. This is explained in the *Using authenticated contacts* recipe in [Chapter 10](ch10.html "Chapter 10. Security and Performance"), *Security and Performance*.

In this example, we'll view a month's history for a `PING` service on `troy.naginet`, a Linux server.

## How to do it...

We can arrange an availability report for the last month for our `troy.naginet` server as follows:

1.  Log in to the Nagios Core web interface.
2.  Click on the **Availability** link in the left menu, beneath **Reports**:![How to do it...](img/5566_07_04.jpg)
3.  Select the type of report, either **Host(s)**, **Hostgroup(s)**, **Service(s)**, or **Servicegroup(s)**:![How to do it...](img/5566_07_05.jpg)
4.  Select a specific host or service on which to report, or optionally choose to run it for all the hosts, hostgroups, services, or servicegroups:![How to do it...](img/5566_07_06.jpg)
5.  Define some options for the report. The defaults are sensible, so we'll use them for now; we'll go over the effect of each of the other fields in the next section. Click on **Create Availability Report!** when done:![How to do it...](img/5566_07_07.jpg)

    You should be presented with a table showing the percentage of time the host or service has spent in each state. A healthy service might look similar to the following screenshot, with a few blips or none at all:

    ![How to do it...](img/5566_07_08.jpg)

    A more problematic service might have large percentages of time in a `WARNING` or `CRITICAL` state:

    ![How to do it...](img/5566_07_09.jpg)

    Above the table appears a quick visual summary of the time spent in each state, which will link to **Trends Report** with the same criteria if clicked. Additionally, below the table are any log entries for the service or host changing state.

## How it works...

Nagios Core assembles state changes from its log files for the specified time period, and constructs the table of state changes by percentage. The availability report, therefore, only works for times covered by your archived log files. The third step of building the report involves a lot of possible options:

*   **Report Period**: This dropdown allows choosing a fixed period for convenience, relative to the current date; alternatively, a custom time period may be used by selecting the final **CUSTOM TIME PERIOD** option and selecting dates in the two fields that follow:

    *   **Start Date (Inclusive)**: This field specifies the date on which the report should start, if the custom time period option has been set.
    *   **End Date (Inclusive)**: This field specifies the date on which the report should end, if the custom time period option has been set.

*   **Report Time Period**: This is the time period for which the availability should be assessed. The default is **None**, meaning that the service or host's state at any time will be used in the calculations. You could, for example, set this to `workhours` to see the percentage of uptime during a time that the server is expected to be busy.
*   **Assume State Retention**: If Nagios Core was restarted one or more times during the reporting period, checking this option would make the report assume that the state before any restart was retained by Nagios Core until it started again; this is enabled with the `retain_state_information` directive in `nagios.cfg`.
*   **Assume Initial States**: If Nagios Core can't figure out the initial state of the service at the time the report begins until the first check, it will assume it based on the value of the **First Assumed Host State** or **First Assumed Service State** fields.
*   **Assume States During Program Downtime**: If Nagios Core finds that it was down for a period in its log files, it will assume the final state it read from the host or service before it went down.
*   **Include Soft States**: Nagios Core will graph `SOFT` states, meaning that it will include state changes that occur, but that return to their previous state, before `max_check_attempts` is exhausted. Otherwise, it will only graph states that have endured right through the retry checks, or `HARD` states.
*   **First Assumed Host State** or **First Assumed Service State**: This is the value Nagios Core should assume for the host or service state, if it can't determine it from the log files.
*   **Backtracked Archives**: This specifies the number of archived log files back the Nagios Core process should check to try and find initial states for the host or service.

## There's more...

You can choose to run the report for a hostgroup or servicegroup as well, which will yield an indexed table showing both the per-host or per-service percentage state time, and also the average uptime for all the hosts or services in the group.

## See also

*   The *Viewing and interpreting trends* and *Viewing and interpreting notification history* recipes in this chapter
*   The *Using authenticated contacts* recipe in [Chapter 10](ch10.html "Chapter 10. Security and Performance"), *Security and Performance*

# Viewing and interpreting trends

In this recipe, we'll learn how to use the **Host** and **Service State Trends** reporting tool on a host or service to show a graph of states over some fixed period of time. This can be useful to determine not only overall availability, perhaps to meet the terms of a service-level agreement, but also to ascertain whether there are certain intervals or consistent times that the host enters a non-`OK` state. It's a good way to look for patterns in the downtime of your hosts.

## Getting started

You will need access to the Nagios Core web interface, and permission to run commands from the CGIs. The sample configuration installed by following the Quick Start Guide grants the `nagiosadmin` user all the necessary privileges when authenticated via `HTTP`.

If you find that you don't have this privilege, then check the `authorized_for_all_services` and `authorized_for_all_hosts` directives in `/usr/local/nagios/etc/cgi.cfg`, and include your username in both; for the user `tom`, the directives might look similar to the following:

```
authorized_for_all_services=nagiosadmin,tom
authorized_for_all_hosts=nagiosadmin,tom
```

Alternatively, you should also be able to see a host or service's information if you are authenticating with the same username as a nominated contact for the host or service you want to check.

In this example, we'll view a month's history for the HTTP service on `athens.naginet`, a web server for which we've been running checks.

## How to do it...

We can arrange a **Service State Trends** report for the last month for our `athens.naginet` server as follows:

1.  Log in to the Nagios Core web interface.
2.  Click on the **Trends** link in the left menu, beneath **Reports**:![How to do it...](img/5566_07_10.jpg)
3.  Select the type of report, either **Host** or **Service**:![How to do it...](img/5566_07_11.jpg)
4.  Select a specific host or service on which you would like to report:![How to do it...](img/5566_07_12.jpg)
5.  Define some options for the report. The defaults are sensible, so we'll use them for now; we'll go over the effect of each of the other fields in the next section. Click on **Create Report** when done:![How to do it...](img/5566_07_13.jpg)

You should be presented with a graph showing the state of the host or service over time, along with markings of time for when the state changed, and a percentage breakdown of the relative states on the right.

A healthy service might look similar to the following screenshot, with a few blips or none at all:

![How to do it...](img/5566_07_14.jpg)

A more problematic service might have long periods of time in a `WARNING` or `CRITICAL` state:

![How to do it...](img/5566_07_15.jpg)

You can click on sections of the graph to "zoom in” on it, provided that you did not select the **Suppress image map** option on the **Select Report Options** page.

## How it works...

Nagios Core assembles state changes from its log files for the specified time period and constructs the graph of state changes by color, delineating the dates on the horizontal axis and at regular intervals. The trends graph, therefore, only works for times covered by your archived log files. The third step of building the report involves a lot of possible options:

*   **Report period**: This dropdown allows choosing a fixed period for convenience, relative to the current date; alternatively, a custom time period may be used by selecting the final **CUSTOM TIME PERIOD** option and selecting dates in the two fields that follow:

    *   **Start Date (Inclusive)**: This field specifies the date on which the report should start, if the custom time period option has been set.
    *   **End Date (Inclusive)**: This field specifies the date on which the report should end, if the custom time period option has been set.

*   **Assume State Retention**: If Nagios Core was restarted one or more times during the reporting period, checking this option will make the report assume that the state before any restart was retained by Nagios Core until it started again; this is enabled with the `retain_state_information` directive in `nagios.cfg`.
*   **Assume Initial States**: If Nagios Core can't figure out the initial state of the service at the time the report begins until the first check, it will assume it based on the value of the **First Assumed Host State** or **First Assumed Service State** fields.
*   **Assume States During Program Downtime**: If Nagios Core finds that it was down for a period in its log files, it will assume the final state it read from the host or service while it was down.
*   **Include Soft States**: Nagios Core will graph `SOFT` states, meaning that it will include state changes that occur, but that return to their previous states, before `max_check_attempts` is exhausted. Otherwise, it will only graph states that have endured right through the retry checks, or `HARD` states.
*   **First Assumed Host State** or **First Assumed Service State**: This is the value Nagios Core should assume for the host or service state, if it can't determine it from the log files.
*   **Backtracked Archives**: This specifies the number of archived log files back the Nagios Core process should check to try and find initial states for the host or service.
*   **Suppress image map**: This prevents the graph from being clickable to zoom in on a particular region of it, perhaps for reasons of browser compatibility.
*   **Suppress popups**: This prevents the graph from showing popups when hovering over sections of it, perhaps for reasons of browser compatibility.

## There's more...

Note that it's important to ensure that checks have actually been running for the entire time for which you're running the report, as otherwise the **State Breakdowns** section will have distorted statistics. There is not much point running a yearly report for a host that has only existed for six months! In general, the more frequent and consistent your checks, the more accurate the trends graph will be.

## See also

*   The *Viewing and interpreting availability reports* *and Viewing and interpreting notification history* recipes in this chapter

# Viewing and interpreting notification history

In this recipe, we'll see how to get both complete listings and convenient summaries of the alerts and notifications being generated by Nagios Core in response to hosts and services changing state. These options are all available under the **Reports** section of the sidebar:

![Viewing and interpreting notification history](img/5566_07_16.jpg)

It's important to distinguish between alerts and notifications in this section. An **alert** is generated in response to an event such as a host or service changing state. A **notification**, in turn, may or may not be generated as a response to that alert, and be sent to the appropriate contacts. `SOFT` state changes constitute alerts; only `HARD` state changes generally make notifications.

It's likely that a production monitoring server will not be sending notifications for every alert, particularly if you're making good use of the `max_check_attempts`, scheduled downtime, and problem acknowledgement features. You should make sure you're checking the correct section.

## Getting started

You will need access to the Nagios Core web interface and permission to run commands from the CGIs. The sample configuration installed by following the Quick Start Guide grants all the necessary privileges to the `nagiosadmin` user when authenticated via `HTTP`.

## How to do it...

We can get an overview of the notifications, alerts, and other events being generated on the Nagios Core server as follows:

1.  Start with the **Notifications** report. The resulting screen should show you the day's notifications as read from the current log file.![How to do it...](img/5566_07_17.jpg)
2.  Note that if you're using the **log rotation** feature of Nagios Core, configured with the `log_rotation_method` directive, then you can use the arrows above the table to navigate to the previous period's monitoring (generally 24 hours worth). Additionally, note that the information includes the following:

    *   Links to the hosts and services that generate them
    *   The contacts to which they were sent
    *   The notification commands that were run
    *   The output of the commands that prompted the alert and notification

    Also note that you're able to filter for any particular kind of notification, and change the sorting order, using the form in the top-right corner:

    ![How to do it...](img/5566_07_18.jpg)
3.  Next, move to the **Alerts** or **Alerts | History report**. Rather than a tabular format, this shows you all of the alerts generated by hosts and services as a list, showing red exclamation icons for `CRITICAL` or `DOWN` states, green icons for `OK` states, and others. The list is broken into intervals of one hour:![How to do it...](img/5566_07_19.jpg)

    Note that these alerts also include the server being started or shut down. Also note that you're again able to filter for specific alert types with the form in the top-right corner.

4.  The other two items in the **Alerts** menu allow generating complex reports with many criteria. The first, **Summary**, presents its results in tabular format according to the criteria entered in a form. For now, try clicking on **Create Summary Report** at the bottom of the first page, with all of the fields on their defaults, to get a feel for the results it generates:![How to do it...](img/5566_07_20.jpg)

    The **Histogram** report generates something similar, showing a breakdown of the alerts generated for nominated hosts or services over a fixed period of time:

    ![How to do it...](img/5566_07_21.jpg)

## How it works...

Nagios Core saves the data for both alerts and notifications on a long-term basis, allowing for reports like this to be generated dynamically from its archives. Note that the data presented in these reports is for alerts and notifications, not the actual states of the services over time. To get useful statistics such as the percentage of uptime, you'll want to generate an availability report, as discussed in the *Viewing and interpreting availability reports* recipe in this chapter.

The same notifications data can also be translated into a MySQL database format via the NDOUtils extension, so that reading the alerts and notifications data programmatically to generate custom reports is possible. See the *Reading status into a* *MySQL database with NDOUtils* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*, for information on how to do this.

## There's more...

The remaining item in the menu is the **Event Log**, which presents a more comprehensive summary of Nagios Core's overall activity without filtering only for alerts or notifications. This screen can include information such as the application of external commands. It tends to be quite verbose, but it's a useful way to read the `nagios.log` file from the web interface. Your username will need to be included in the `authorized_for_system_information` directive in `/usr/local/nagios/etc/cgi.cfg` to use this:

```
authorized_for_system_information=nagiosadmin,tom
```

## See also

*   The *Viewing and interpreting availability reports* and *Viewing and interpreting trends* recipes in this chapter
*   The *Reading status into a MySQL database with NDOUtils* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*

# Adding comments on hosts or services in the web interface

In this recipe, we'll learn how to add comments to hosts or services in the Nagios Core web interface, to keep track of information about them for all the web interface users.

## Getting started

You will need access to the Nagios Core web interface, and permission to run commands from the CGIs. The sample configuration installed by following the Quick Start Guide grants all the necessary privileges to the `nagiosadmin` user when authenticated via `HTTP`. You will also need at least one host or service.

If you find that you don't have this privilege, then check the `authorized_for_all_service_commands` and `authorized_for_all_host_commands` directives in `/usr/local/nagios/etc/cgi.cfg`, and include your username in both; for example, for the user `tom`, the directives might look similar to the following:

```
authorized_for_all_service_commands=nagiosadmin,tom
authorized_for_all_host_commands=nagiosadmin,tom
```

## How to do it...

We can add a comment to a host or service as follows:

1.  Log in to the Nagios Core web interface, and click on the hostname or service description on which you wish to leave the comment. You can get to this via the **Hosts** or **Services** menu items. Here, I'm leaving a comment on my **sparta.naginet** host:![How to do it...](img/5566_07_22.jpg)
2.  Click on the **Add a new comment** link at the bottom of the page:![How to do it...](img/5566_07_23.jpg)
3.  Fill out the resulting form and include the following details:

    *   **Host Name**: This is the host name for the host or service on which the comment should be made. This should be filled in automatically.
    *   **Persistent**: Check this box if you would like the comment to remain on the host even after Nagios Core is restarted.
    *   **Author (Your Name)**: This is the name of the person acknowledging the fault. This should default to your name or username; it may be grayed out and unchangeable, depending on the value of `lock_author_names` in `/usr/local/nagios/etc/cgi.cfg`.
    *   **Comment**: This is the text of the comment itself.

    Note that explanatory notes also appear to the right of the command description. Click on **Commit** when done:

    ![How to do it...](img/5566_07_24.jpg)

With this done, a comment should be added to the host or service. It may take a little while for the command to be processed. You will find that there is an icon added to the host or service's link in its menu, and that a comment has been added:

![How to do it...](img/5566_07_25.jpg)

## How it works...

Like most changes issued from the Nagios Core web interface, adding a comment to a host or service is a command issued for processing by the server, along with its primary task of executing plugins and recording states. This is why it can sometimes take a few seconds to apply even on an idle server. This is all written to the command file, by default stored at `/usr/local/nagios/var/rw/nagios.cmd`, which the Nagios Core system regularly reads.

When the command is executed, a comment is added to the host or service that can be viewed by anyone with appropriate permissions in the web interface.

## There's more...

You can view a list of all comments with the **Comments** link in the sidebar:

![There's more...](img/5566_07_26.jpg)

This will bring up a complete list of comments for all hosts and services, including both automated and manual ones:

![There's more...](img/5566_07_27.jpg)

## See also

*   The *Acknowledging a problem via the web interface* recipe in this chapter

# Viewing configuration in the web interface

In this recipe, we'll learn how to view a table of all the objects currently configured for the running Nagios Core instance. This is a very convenient way to view how the system has understood your configuration. If you use a lot of configuration tricks, such as object inheritance and patterns for hostnames, this can really help you make sense of things.

Some administrators aren't aware of this feature, because it's tucked away at the bottom of the Nagios Core web interface's navigation menu. It's very simple to use as there are only really two steps involved.

## Getting started

You will need access to the Nagios Core web interface, and permission to run commands from the CGIs. The sample configuration installed by following the Quick Start Guide grants all the necessary privileges to the `nagiosadmin` user when authenticated via `HTTP`.

If you find that you don't have this privilege, then check the `authorized_for_configuration_information` directive in `/usr/local/nagios/etc/cgi.cfg`, and include your username in it; for example, for the user `tom`, the directives might look like the following:

```
authorized_for_configuration_information=nagiosadmin,tom
```

## How to do it...

We can view the configuration of different object types in the web interface as follows:

1.  Log in to the Nagios Core web interface, and click on the **Configuration** item in the navigation menu:![How to do it...](img/5566_07_28.jpg)
2.  Select the object type you wish to view, and click on **Continue**. In this example, I want to view all the hosts I have defined:![How to do it...](img/5566_07_29.jpg)

    You should be presented with a complete table showing the directive values of all the hosts defined in your system. It will likely be wider than the screen, and will hence require horizontal scrolling.

    ![How to do it...](img/5566_07_30.jpg)

## How it works...

This is simply a convenient shortcut for viewing the configuration of your hosts as Nagios Core understands them. The following object types can be viewed:

*   Hosts
*   Host dependencies
*   Host escalations
*   Host groups
*   Services
*   Service groups
*   Service dependencies
*   Service escalations
*   Contacts
*   Contact groups
*   Time periods
*   Commands
*   Command expansion

This shortcut is particularly convenient for definitions that can sometimes be quite complex, such as hosts and services that inherit from templates, or time period definitions. It's a good way to check that Nagios Core has interpreted your configuration the way you expected.

## There's more...

It's important to note that the output doesn't actually tell you anything about the state of hosts or services; it only shows how they're configured. If you need to get both the configuration and state information in an accessible format for processing by a custom script, an excellent way to arrange this is with the NDOUtils addon, which allows you to translate this information into a MySQL database format. See the *Reading status into a MySQL database with NDOUtils* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios Core* for information on how to do this.

## See also

*   The *Reading status into a MySQL database with NDOUtils* and *Writing customized Nagios Core reports* recipes in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios Core*

# Scheduling checks from the web interface

In this recipe, we'll learn how to manually schedule checks of hosts and services from the web interface, overriding the automatic scheduling normally done by Nagios Core. This can be convenient to hurry along checks for hosts or services that have just been added, or which have just had problems, or to force a check to be made even when active checks are otherwise disabled on a host for whatever reason.

In this example, we'll schedule a check for a service, but checks for hosts can be established in just the same way.

## Getting started

You will need access to the Nagios Core web interface, and permission to run commands from the CGIs. The sample configuration installed by following the Quick Start Guide grants all the necessary privileges to the `nagiosadmin` user when authenticated via `HTTP`.

If you find that you don't have this privilege, then check the `authorized_for_all_service_commands` and `authorized_for_all_host_commands` directives in `/usr/local/nagios/etc/cgi.cfg`, and include your username in both; for example, for the user `tom`, the directives might look like the following:

```
authorized_for_all_service_commands=nagiosadmin,tom
authorized_for_all_host_commands=nagiosadmin,tom
```

## How to do it...

We can schedule a check for an existing service as follows:

1.  Log in to the Nagios Core web interface, and find the link to the host or service for which you want to schedule a check. In this example, we're forcing a check of the **LOAD** service on the **sparta.naginet** host:![How to do it...](img/5566_07_31.jpg)
2.  Click on the **Re-schedule the next check of this service** or **Re-schedule the next check of this host** link in the menu on the right:![How to do it...](img/5566_07_32.jpg)
3.  Complete the resulting form and include the following details:

    *   **Host Name**: This is the host name for the host or service having the problem. This should be filled in automatically.
    *   **Service**: This is the service description for the service having problems, if applicable. This will also be filled in automatically.
    *   **Check Time**: This is the time at which Nagios Core should reschedule the check.
    *   **Force Check**: This specifies whether it's required to force a check regardless of active checks being disabled for this host or service.

    Note that explanatory notes also appear to the right of the command description. Click on **Commit** when done:

    ![How to do it...](img/5566_07_33.jpg)

With this done, after Nagios Core processes the new command, viewing the service should show the new time for its next scheduled check, and the check will be run at the nominated time and its return values processed normally.

## How it works...

Scheduling checks in this manner is a means of "jumping the queue” for a particular host or service. Check scheduling is otherwise an automated process that Nagios Core figures out based on host and service directives such as `check_interval` and `retry_interval`.

Like all external commands issued from the web interface, a command is written to the commands file, usually at `/usr/local/nagios/var/rw/nagios.cmd`, for processing by the system.

## There's more...

We can inspect the enqueued checks by clicking on the **Scheduling Queue** link under the **System** section of the sidebar:

![There's more...](img/5566_07_34.jpg)

This will bring up a list of the pending checks, and the times at which they'll take place:

![There's more...](img/5566_07_35.jpg)

We can also get more detailed information about the queuing process and performance by using the `-s` flag with the `nagios` command:

```
# /usr/local/nagios/bin/nagios -s /usr/local/nagios/etc/nagios.cfg

```

This will print a lot of information to the terminal, too much to reproduce here! It is probably helpful to pipe it through a paging tool like `less` to read it:

```
# /usr/local/nagios/bin/nagios -s /usr/local/nagios/etc/nagios.cfg | less

```

## See also

*   The *Viewing and interpreting notification history* recipe in this chapter
*   The *Checking Nagios Core performance with Nagiostats* recipe in [Chapter 10](ch10.html "Chapter 10. Security and Performance"), *Security and Performance*
*   The *Allowing passive checks*, and *Running passive checks from a remote host with NSCA* recipes in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios Core*

# Acknowledging a problem via the web interface

In this recipe, we'll learn how to acknowledge problems with a host or service, in order to prevent further notifications coming from it and to signal that the problem is being worked on. This is useful if more than one administrator has access to the Nagios Core web interface, to prevent more than one administrator trying to fix a problem, and to prevent unnecessary notifications for a longer-term problem after the operations team has been made aware of it. The exact changes in behavior that the acknowledgement causes are defined at the time it's submitted.

## Getting started

To acknowledge a notification, there needs to be at least one host or service suffering problems. Any host or service in a `WARNING`, `CRITICAL`, or `UNKNOWN` state can be acknowledged.

You will need access to the Nagios Core web interface, and permission to run commands from the CGIs. The Quick Start configuration grants all the necessary privileges to the `nagiosadmin` user when authenticated via `HTTP`.

If you find that you don't have this privilege, check the `authorized_for_all_service_commands` and `authorized_for_all_host_commands` directives in `/usr/local/nagios/etc/cgi.cfg`, and include your username in both; for example, for the user `tom`, the directives might look like the following:

```
authorized_for_all_service_commands=nagiosadmin,tom
authorized_for_all_host_commands=nagiosadmin,tom
```

## How to do it...

We can acknowledge a host or service problem as follows:

1.  Log in to the Nagios Core web interface, and click on the **Hosts** or **Services** section in the left menu, to visit the host or service having problems. In this example, I'm acknowledging a problem with the **APT** service on my machine **athens**; Nagios Core is reporting upgrades available, but they're not critical and I can't run them right now. Hence I'm going to acknowledge them, so that other administrators know I've got a plan for the problem:![How to do it...](img/5566_07_36.jpg)
2.  Click on the **Acknowledge this service problem** link under the **Service Commands** menu:![How to do it...](img/5566_07_37.jpg)
3.  Complete the resulting form and include the following details:

    *   **Host Name**: This is the host name for the host or service having the problem. This should be filled in automatically.
    *   **Service**: This is the service description for the service having problems, if applicable. This will also be filled in automatically.
    *   **Sticky Acknowledgement**: If checked, notifications will be suppressed until the problem is resolved; the acknowledgement will not go away if Nagios Core is restarted.
    *   **Send Notification**: If checked, an `ACKNOWLEDGEMENT` notification will be sent to all the contacts and contact groups for the host or service.
    *   **Persistent Comment**: Acknowledgements always leave comments on the host or service in question. By default, these are removed when the host or service recovers, but if you wish you can check this to arrange to have them take the form of a permanent comment.
    *   **Author (Your Name)**: This is the name of the person acknowledging the fault. This should default to your name or username; it may be grayed out and unchangeable, depending on the value of `lock_author_names` in `/usr/local/nagios/etc/cgi.cfg`.
    *   **Comment**: This is an explanation of the acknowledgement; this is generally a good place to put expected times to resolution, and what's being done about the problem.

    Note that explanatory notes also appear to the right of the command description. Click on **Commit** when done:

    ![How to do it...](img/5566_07_38.jpg)

With this done, the effects you chose for the downtime should take effect shortly. It may take a little while for the command to be processed. You will find that there is an acknowledgement icon added to the host or service's link in its menu, and that a comment has been added:

![How to do it...](img/5566_07_39.jpg)

Your contacts may also receive an **ACKNOWLEDGEMENT** notification:

![How to do it...](img/5566_07_40.jpg)

## How it works...

Like most changes issued from the Nagios Core web interface, acknowledging a host or service is a command issued for processing by the server, along with its primary task of executing plugins and recording states. This is why it can sometimes take a few seconds to apply even on an idle server. This is all written to the command file, which is by default stored at `/usr/local/nagios/var/rw/nagios.cmd`. The Nagios Core system regularly reads this file, provided external command processing is enabled in `/usr/local/nagios/etc/nagios.cfg`.

When the command is executed and the **Sticky Acknowledgement** option is selected, the effect is that the filters normally applied by Nagios Core will notice that the host or service has had its problems acknowledged before dispatching a notification, and will therefore prevent the notification from being sent.

When the host recovers (a `RECOVERY` event), a notification to that effect will be sent, and the acknowledgement will be removed as it's no longer needed. Future problems with the host or service, even if they're exactly the same problem, will need to be themselves acknowledged.

## There's more...

Acknowledging host and service problems is a really good habit to get into for administrators, particularly if working in a team, as it may prevent a lot of confusion or duplication of work. It's therefore helpful in the same way comments are, in keeping your team informed about what's going on with the network.

The distinction between scheduled downtime and acknowledgements is important. **Downtime** is intended to be for planned outages to a host or service; an **acknowledgement** is intended as a means of notification that an unanticipated problem has occurred, and is being worked on.

If we're getting nagged repeatedly by notifications for problems that we already know about, as well as using acknowledgements, then it may be appropriate to review the `notification_interval` directive for the applicable hosts or services, to limit the frequency with which notifications are sent. For example, to only send repeat notifications every four hours, we could write the following code snippet:

```
define host {
    ...
    notification_interval  240
}
```

For low-priority hosts, for example, it's perhaps not necessary to send notifications every 10 minutes!

Do not confuse this directive with `notification_period`, which is used to define the times during which notifications may be sent, another directive that may require review:

```
define host {
    ...
    notification_period  24x7
}
```

## See also

*   The *Adding comments on hosts or services in the web interface* recipe in this chapter
*   The *Scheduling downtime for a host or service* recipe in [Chapter 3](ch03.html "Chapter 3. Working with Checks and States"), *Working with Checks and States*