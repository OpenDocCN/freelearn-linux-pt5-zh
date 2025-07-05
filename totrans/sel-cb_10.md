# Chapter 10. Handling SELinux-aware Applications

In this chapter, we will cover handling of SELinux-aware applications through the following recipes:

*   Controlling D-Bus message flows
*   Restricting service ownership
*   Understanding udev's SELinux integration
*   Using cron with SELinux
*   Checking the SELinux state programmatically
*   Querying SELinux userland configuration in C
*   Interrogating the SELinux subsystem code-wise
*   Running new processes in a new context
*   Reading the context of a resource

# Introduction

For most applications, the SELinux subsystem in the Linux kernel is capable of enforcing security controls without further interaction with other applications and components. However, there are actions that cannot be handled by the SELinux subsystem autonomously. Some applications execute commands for specific users, but the target domain cannot be deduced from the path of the application that is itself being executed, making type transitions based on the label impossible.

One solution for this problem is to make the application SELinux-aware, having the application interrogate the SELinux subsystem as to what should be the context of the newly executed application. Once the context is obtained, the application can then instruct the SELinux subsystem that this context can be assigned to the process that will be launched next.

Of course, it isn't only about deciding what context a process should be in. Applications can also check the SELinux policy and act on the policy themselves, rather than having the policies enforced through the Linux kernel. If applications use SELinux to get more information about a session and set contexts based on this information, then we call these applications SELinux-aware.

The easiest method to see whether an application is SELinux-aware is to check the documentation, or to check whether it is linked with the `libselinux.so` library:

```
~$  ldd /usr/sbin/crond | grep selinux
 libselinux.so.1 => /lib64/libselinux.so.1 (0x00007fa53299a000)

```

Some SELinux-aware applications not only query information, but also enforce decisions on objects that the SELinux subsystem in the Linux kernel cannot control. Examples of such objects are the database objects in the **Security Enhanced PostgreSQL** (**SEPostgreSQL**) application or the D-Bus services. Although represented in the SELinux policy, they are not part of the regular Linux operating system but are instead owned by the application itself. Such SELinux-aware applications are called **user space object managers**.

Regardless of how an application handles its SELinux-specific code, whenever such applications are used on a system, it is important to know how the SELinux code in the application works, as the standard approach (look at AVC denials and see whether a context needs to be changed or the policy tuned) might not work at all in these cases.

# Controlling D-Bus message flows

D-Bus implementation on Linux is an example of an SELinux-aware application, acting as a user space object manager. Applications can register themselves on a bus and can send messages between applications through D-Bus. These messages can be controlled through the SELinux policy as well.

## Getting ready

Before looking at the SELinux access controls related to message flows, it is important to focus on a D-Bus service and see how its authentication is done (and how messages are relayed in D-Bus) as this is reflected in the SELinux integration.

Go to `/etc/dbus-1/system.d/` (which hosts the configuration files for D-Bus services) and take a look at a configuration file. For instance, the service configuration file for `dnsmasq` looks like the following:

```
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN" "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <policy user="root">
    <allow own="uk.org.thekelleys.dnsmasq"/>
    <allow send_destination="uk.org.thekelleys.dnsmasq"/>
  </policy>
  <policy context="default">
    <deny own="uk.org.thekelleys.dnsmasq"/>
    <deny send_destination="uk.org.thekelleys.dnsmasq"/>
  </policy>
</busconfig>
```

This configuration tells D-Bus that only the root Linux user is allowed to have a service *own* the `uk.org.thekelleys.dnsmasq` service and send messages to this service. Others (as managed through the default policy) are denied these operations.

On a system with SELinux enabled, having root as the finest granularity doesn't cut it. So, let's look at how the SELinux policy can offer a fine-grained access control in D-Bus.

## How to do it…

To control D-Bus message flows with SELinux, perform the following steps:

1.  Identify the domain of the application that will (or does) own the D-Bus service we are interested in. For the `dnsmasq` application, this would be `dnsmasq_t`:

    ```
    ~# ps -eZ | grep dnsmasq | awk '{print $1}'
    system_u:system_r:dnsmasq_t:s0-s0:c0.c1023

    ```

2.  Identify the domain of the application that wants to send messages to the service. For instance, this could be the `sysadm_t` user domain.
3.  Allow the two domains to interact with each other through D-Bus messages as follows:

    ```
    gen_require(`
      class dbus send_msg;
    ')
    allow sysadm_t dnsmasq_t:dbus send_msg;
    allow dnsmasq_t sysadm_t:dbus send_msg;
    ```

## How it works…

When an application connects to D-Bus, the SELinux label of its connection is used as the label to check when sending messages. As there is no transition for such connections, the label of the connection is the context of the process itself (the domain); hence the selection of `dnsmasq_t` in the example.

When D-Bus receives a request to send a message to a service, D-Bus will check the SELinux policy for the `send_msg` permission. It does so by passing on the information about the session (source and target context and the permission that is requested) to the SELinux subsystem, which computes whether access should be allowed or not. The access control itself, however, is not enforced by SELinux (it only gives feedback), but by D-Bus itself as governing the message flows is solely D-Bus' responsibility.

This is also why, when developing D-Bus-related policies, both the class and permission need to be explicitly mentioned in the policy module. Without this, the development environment might error out, claiming that `dbus` is not a valid class.

D-Bus checks the context of the client that is sending a message as well as the context of the connection of the service (which are both domain labels) and see if there is a `send_msg` permission allowed. As most communication is two-fold (sending a message and then receiving a reply), the permission is checked in both directions. After all, sending a reply is just sending a message (policy-wise) in the reverse direction.

It is possible to verify this behavior with `dbus-send` if the rule is on a user domain. For instance, to look at the objects provided by the service, the D-Bus introspection can be invoked against the service:

```
~# dbus-send --system --dest=uk.org.thekelleys.dnsmasq --print-reply /uk/org/thekelleys/dnsmasq org.freedesktop.DBus.Introspectable.Introspect

```

When SELinux does not have the proper `send_msg` allow rules in place, the following error will be logged by D-Bus in its service logs (but no AVC denial will show up as it isn't the SELinux subsystem that denies the access):

```
Error org.freedesktop.DBus.Error.AccessDenied: An SELinux policy prevents this sender from sending this message to this recipient. 0 matched rules; type="method_call", sender=":1.17" (uid=0 pid=6738 comm="") interface="org.freedesktop.DBus.Introspectable" member="Introspect" error name="(unset)" requested_reply="0" destination="uk.org.thekelleys.dnsmasq" (uid=0 pid=6635 comm="")

```

When the policy does allow the `send_msg` permission, the introspection returns an XML output showing the provided methods and interfaces for this service.

## There's more...

The current D-Bus implementation is a pure user space implementation. Because more applications become dependent on D-Bus, work is being done to create a kernel-based D-Bus implementation called **kdbus**. The exact implementation details of this project are not finished yet, so it is unknown whether the SELinux access controls that are currently applicable to D-Bus will still be valid on kdbus.

# Restricting service ownership

Applications that register themselves on the bus own a service name. The `uk.org.thekelleys.dnsmasq` service name is an example of this. The D-Bus policy, declared in the `busconfig` XML file at `/etc/dbus-1/system.d/` (or `session.d/` if the service is for the session bus instead of system bus) provides information for D-Bus to decide when taking ownership of a particular service is allowed.

Thanks to D-Bus' SELinux integration, additional constraints can be added to ensure that only authorized applications can take ownership of a particular service.

## How to do it…

To restrict service ownership through the SELinux policy, follow the ensuing set of steps:

1.  Inside the D-Bus configuration file of the service, make sure that the `own` permission is properly protected. For instance, make sure only the `root` Linux user can own the service:

    ```
    <policy user="root">
      <allow own="uk.org.thekelleys.dnsmasq" />
    </policy>
    ```

2.  If the runtime service account can differ, it is possible to declare a `group=` parameter instead of a `user=` parameter as well.
3.  Next, declare which label to associate to the service:

    ```
    <selinux>
      <associate own="uk.org.thekelleys.dnsmasq" context="dnsmasq_t" />
    </selinux>
    ```

4.  In the SELinux policy, declare which domain(s) are allowed to acquire this service:

    ```
    gen_require(`
      class dbus acquire_svc;
    ')
    allow dnsmasq_t self:dbus acquire_svc;
    ```

## How it works…

The D-Bus configuration allows administrators to define when service ownership for a particular service can be taken. Most services define the user (or group) that is allowed to own a service, as shown in the example. But for system services, only declaring that the Linux root user can own a particular service is definitely not sufficiently fine-grained.

Enter SELinux. With the association definition in the `busconfig` XML file, D-Bus is told that any application domain that tries to own that particular service must have the `acquire_svc` privilege (in the `dbus` class) against the mentioned context.

With this approach, administrators can ensure that other domains, even though they run as the Linux root user, are not allowed to own the service.

Although the usual approach, for the target label, is to require the context of the application itself, it is also possible to use a different context. For instance, a new type can be declared such as `dnsmasq_dbus_t` and then the SELinux policy is set to the following:

```
allow dnsmasq_t dnsmasq_dbus_t:dbus acquire_svc;
```

## There's more...

The D-Bus application has a configuration file inside `/etc/selinux/mcs/contexts/`, which follows the same structure, called `dbus_contexts`. This is a default context definition for D-Bus ownership (what context should be used by default if it cannot be deduced by other means). By default, no SELinux-specific settings are provided anymore as D-Bus is now fully aware of the contexts to use, and it is not recommended to modify this file anymore.

However, it is useful to know that the file exists and is used, especially when D-Bus would be executed in a container, chroot, or other environment as D-Bus will complain if the file is missing:

```
Failed to start message bus: Failed to open "/etc/selinux/mcs/contexts/dbus_contexts": No such file or directory

```

If the SELinux support in D-Bus needs to be disabled (but without rebuilding D-Bus), then edit `/etc/dbus-1/system.conf` and `session.conf` and remove the following line:

```
<include if_selinux_enabled="yes" selinux_root_relative="yes">contexts/dbus_contexts</include>
```

# Understanding udev's SELinux integration

The udev device manager is responsible for handling device files inside the `/dev/` structure whenever changes occur. As many device files have different contexts, without any SELinux awareness, the udev policy would need to be enhanced with many, many named file transitions. Such a named file transition, for a device `/dev/mydevice` towards the `mydevice_t` type, would look like the following code:

```
dev_filetrans(udev_t, mydevice_t, chr_file, "mydevice")
```

However, when `/dev/mydevice1`, `/dev/mydevice2`, and so on need to be labeled as well, then each possible name would need to be iterated in the policy (named file transitions do not support regular expressions). Luckily, udev is SELinux-aware, making it unnecessary to create policy enhancements for every device file.

This recipe shows us when additional policy enhancements are needed and when not.

## How to do it…

To understand how udev's SELinux integration works, the following decision criteria can be followed:

1.  Whenever a device file is created by udev inside a directory with the `device_t` label, then udev will automatically label the device file with the label known to the SELinux subsystem through its `file_contexts` definitions if the target type is assigned the `device_node` attribute.
2.  If the parent directory does not use the `device_t` type, then make sure that udev holds manage rights on that target type.
3.  If the target file context is not associated with the `device_node` attribute, grant udev the proper `relabelto` privileges.
4.  If udev's rules are configured to create symbolic links, then assert that the label of the links remains the generic `device_t` type.

## How it works…

The udev application is a standard SELinux-aware application that interacts with the SELinux user space by querying the context definitions and either creating the new device files with the queried context or by relabeling the device files afterwards.

By querying the context definitions (instead of relying on the SELinux policy), administrators can easily modify the rules for different device names or include support for new device types, without the need to enhance the `udev_t` related policies. All that an administrator has to do is to configure the proper file context definition:

```
~# semanage fcontext -a -t mydevice_t -f -c /dev/mydevice[0-9]*

```

However, if the target device type (`mydevice_t`) is not associated with the `device_node` attribute, then `udev_t` will not have the privileges to relabel this device type. This attribute is vital for the support of `udev_t`, as it has relabel (and manage) rights on all device nodes through this attribute.

If a udev rule would request the creation of a device file that is not associated with the `device_node` attribute (or a different file—the requested file does not need to be a device), then an update on the SELinux policy is needed if the default context association (that is, through inheritance of the type through the parent directory) is not sufficient.

For the same reason, it is necessary to have symbolic links remain as `device_t` as the SELinux policy does not handle different types for symbolic links.

Of course, this SELinux support inside udev also has its consequences when device files are created outside of udev's handling. If that is the case, then the administrator has to make sure that the label of the files is corrected, as wrong device types can result in a system malfunction.

A popular approach for that is to relabel the entire `/dev/` structure (which is often done by a distribution `init` script to counter the default device file creation—and its default `device_t` type—from within the initial RAM filesystem or the `devtmpfs` mount):

```
~# restorecon -R /dev

```

# Using cron with SELinux

Another example of an SELinux-aware application is cron. Well, actually a set of cron implementations, as there is not a single cron application. Examples of cron implementations are vixie-cron, cronie, and fcron.

The cron implementations invoke commands for (and as) a particular Linux user. As these commands are not set in stone (the main purpose of cron is to allow any command to be run for a particular user or even for the system itself), it is not possible to easily create a policy that is sufficiently fine-grained to accommodate all features provided by cron. After all, for SELinux itself, there is no difference between cron calling a command for one user or another: all that is involved is the cron domain (`crond_t`) and the target type of the command (such as `bin_t`).

For this reason, many cron implementations are made SELinux-aware, allowing the cron implementation to select the proper target context.

## How to do it…

To properly interact with an SELinux-aware cron, the following steps need to be followed:

1.  Make sure that the crontab files are properly labeled: `user_cron_spool_t` for the user crontabs, and `system_cron_spool_t` for the system crontab.
2.  Check `/etc/selinux/mcs/contexts/default_contexts` or `/etc/selinux/mcs/contexts/users/*` for the target context of the `system_r:crond_t` domain.
3.  Have the crontab file context be an entrypoint for the target domain. For instance, if the target domain for a user is its own user domain (such as `user_t`), then `user_cron_spool_t` has to be known as an entrypoint for `user_t`.
4.  Set the `cron_userdomain_transition` Boolean to `on` if the target domain for user jobs is the user domain, or `off` if the target domain should be the `cronjob_t` domain.

## How it works…

When cron is SELinux-aware, it is vital that it is running in the `crond_t` domain. Its internal SELinux code will query the SELinux policy to see what the target domain is for a user through the application, and if cron isn't running in the `crond_t` domain, then this query will not result in the correct set of domains:

```
~# ps -efZ | grep fcron | awk '{print $1}'
system_u:system_r:crond_t:s0-s0:c0.c1023

```

Before launching user jobs from cron, the cron application will check the file context of the user crontab file. This file context is then used to see whether the target domain for the user jobs has the user crontab file context as an entrypoint.

To know what the current target domain will be, we can use the `getseuser` helper application:

```
~# getseuser hannah system_u:system_r:crond_t:s0
seuser: user_u
Context 0    user_u:user_r:cronjob_t:s0

```

In this case, the target domain is `cronjob_t`. This should be confirmed by the `default_contexts` (or user-specific context) file:

```
~# grep crond_t /etc/selinux/mcs/contexts/users/user_u
system_r:crond_t  user_r:cronjob_t

```

If the target domain should be the user domain, then we need to toggle the right Boolean and adjust the context file accordingly:

```
~# setsebool cron_userdomain_transition on
~# grep crond_t /etc/selinux/mcs/contexts/users/user_u
system_r:crond_t  user_r:user_t

```

With the target domain known, the last thing that is needed is that the user cronjob file context is known as an entrypoint for the domain, which most cron implementations will check as a sort-of access control:

```
~# sesearch -s user_t -t user_cron_spool_t -c file -p entrypoint -A
Found 1 semantic av rules:
 allow user_t user_cron_spool_t : file entrypoint ;

```

## There's more…

Not all cron implementations are SELinux-aware. If the implementation is not SELinux-aware, then the cron jobs will all run inside a single cron job container (`cronjob_t` for user cron jobs and `system_cronjob_t` for system cron jobs) with the `system_u` SELinux user and the `system_r` SELinux role.

# Checking the SELinux state programmatically

If the need arises to make an SELinux-aware application, then several languages can be used. The `libselinux` package usually provides bindings for multiple programming and scripting languages. In the next set of recipes, the C programming language will be used as an example implementation.

The first step to support SELinux in an application is to check the SELinux state. In this recipe, we will show how to create an application that links with the `libselinux` library and checks the state of SELinux.

## Getting ready

As we are going to update a C application, this set of recipes will assume basic knowledge of C programming. An example C application that uses all the input from this (and other) recipes can be found in the download pack of this book.

## How to do it…

In order to link with `libselinux` and to check the current SELinux state, the following set of steps can be used:

1.  Create a C application code file and refer to the SELinux header files through a compiler directive:

    ```
    #ifdef SELINUX
    #include <selinux/selinux.h>
    #include <selinux/av_permissions.h>
    #include <selinux/get_context_list.h>
    #endif
    ```

2.  In the application, have the SELinux-related function call return `success` if SELinux support should not be built-in (that is, when the compiler directive isn't set):

    ```
    int selinux_prepare_fork(char * name) {
    #ifndef SELINUX
      return 0;
    #else
      …
    #endif
    };
    ```

3.  Inside the SELinux function, check whether SELinux is enabled using the `is_selinux_enabled()` function call:

    ```
    int rc;
    rc = is_selinux_enabled();
    if (rc == 0) {
      … // SELinux is not enabled
    } else if (rc == -1) {
      … // Could not check SELinux state (call failed)
    } else {
      … // SELinux is enabled
    };
    ```

4.  Add a check to see whether SELinux is in permissive or enforcing mode. Of course, this check is only needed if SELinux is enabled:

    ```
    rc = security_getenforce();
    if (rc == 0) {
      … // SELinux is in permissive mode
    } else if (rc == 1) {
      … // SELinux is in enforcing mode
    } else {
      … // Failed to query state
    };
    ```

5.  Build the application while linking with `libselinux`:

    ```
    ~# gcc -o test -DSELINUX -lselinux test.c

    ```

## How it works…

The `libselinux` library provides all needed functions for applications to query SELinux and interact with the SELinux subsystem. Of course, when developing applications, it remains important that SELinux support is a compile-time optional choice: not all Linux systems have SELinux enabled, so if the application is by default linked with `libselinux`, then all target systems would need to install the necessary dependencies.

But even applications that are linked with `libselinux` must be able to support systems where SELinux has been disabled; hence, the need to check the state of SELinux using `is_selinux_enabled()`.

However, this `is_selinux_enabled()` function does not return any other information (such as which policy is loaded). To check if SELinux is running in permissive mode, the call to `security_getenforce()` can be used.

A well-defined application should use this state as well to adjust its behavior: if the application is running in permissive mode, then it should try not to enforce SELinux policy-related decisions in its application logic.

To refer to the cron example from an earlier recipe: if the crontab file context is not known as an entrypoint for the selected domain, then the application should log that this is not the case, but still continue working (as the mode is set in permissive mode). Sadly, most SELinux-aware applications do not change their behavior based on the permissive state of SELinux and can still fail (or follow a different logic) as if SELinux is in the enforcing state.

## There's more...

There are other similar methods available that can be used to query the SELinux state.

The `is_selinux_mls_enabled()` method, for instance, returns a value indicating whether SELinux is running with MLS or not. This is useful as some context-related methods require level information if MLS is enabled, so querying the state and changing the method calls depending on the MLS state might be necessary.

A similar function to `security_getenforce()` is `security_setenforce()`. As can be deduced from the name, this allows applications to toggle the enforcing mode of SELinux. Of course, this is only possible if the domain in which the application runs has the proper SELinux permissions.

# Querying SELinux userland configuration in C

In this recipe, we will be querying the SELinux userland to obtain the default context for a given user based on the context of the current process. The process is responsible for gathering the Linux username of the user upfront.

## How to do it…

Query the SELinux configuration as follows:

1.  Get the current context of the process:

    ```
    char * curcon = 0;
    rc = getcon(&curcon);
    if (rc) {
      … // Getting context failed
      if (permissive) {
        … // Continue with the application logic, ignoring SELinux stuff
      } else {
        … // Log failure and stop application logic
      };
    };
    ```

2.  Take the Linux username (assumed to be in the `name` variable) and get the SELinux user:

    ```
    char * sename = 0;
    char * selevel = 0;
    rc = getseuserbyname(name, &sename, &selevel);
    if (rc) {
      … // Call failed. Again check permissive state
      … // and take appropriate action.
      freecon(curcon);
    };
    ```

3.  Now, get the default context based on the obtained SELinux user (`sename`) and current context (which is handled by the method itself through the `NULL` variable):

    ```
    char * newcon = 0;
    rc = get_default_context(sename, NULL, &newcon);
    if (rc) {
      … // Call failed. Again check permissive state
      … // and take appropriate action.
      freecon(curcon);
    };
    ```

## How it works…

In the first block, the current process context is obtained using the `getcon()` method. For the end result of this recipe, getting the current context explicitly isn't necessary—the `get_default_context()` method that is invoked later will base its decision on the current context anyway (through the second parameter, which is `NULL` in this recipe). However, having the current context known is important for logging purposes as well as to query the SELinux policy itself (as we will do in the next recipe).

The next step is to obtain the SELinux user given a Linux user. The `sename` (SELinux user) and `selevel` (SELinux sensitivity) variables are filled in by the `getseuserbyname()` method, given the Linux username (which is a regular `char *` variable).

Finally, with the SELinux user now available, `get_default_context()` is invoked to get the default context stored into the third parameter (`newcon`). If we would need to get the default context from a different context than the current one, then instead of `NULL`, the second parameter should be the context to query for:

```
rc = get_default_context(sename, curcon, &newcon);
```

## There's more...

Some other methods might be interesting to use in SELinux-aware applications.

The `getprevcon()` method, for instance, returns the previous context rather than the current context of the process. This previous context is usually the context of the parent process, although with applications that can perform dynamic transitions, this can be the previous context of the current process as well.

This information can also be obtained from the `/proc/` filesystem, in the process's `attr/` subdirectory in which the `current` and `prev` files can be checked:

```
~$ id -Z
staff_u:staff_r:staff_t:s0
~$ newrole -r sysadm_r
Password: 
~$ id -Z
staff_u:sysadm_r:sysadm_t:s0
~$ cat /proc/$$/attr/current
staff_u:sysadm_r:sysadm_t:s0
~$ cat /proc/$$/attr/prev
staff_u:staff_r:newrole_t:s0

```

As can be seen, after running `newrole` to switch roles, the last domain that the process was in was the `newrole_t` domain (which then performed a domain and role transition to the current context).

Applications that are allowed to perform dynamic transitions (that is, without launching new commands) can use the `setcon()` method to switch from the current context to a new context.

The `get_default_context()` method is also part of a larger family of methods. For instance, when the user has multiple roles assigned, there can be multiple contexts allowed for a particular transition. The `get_ordered_context_list()` method returns the list of contexts that are supported (whereas the `get_default_context()` method only returns the first). One can filter out specific contexts by providing the role with the `get_ordered_context_list_with_role()` method.

On MLS-enabled systems, `get_default_context_with_level()` or `get_default_context_with_rolelevel()` will apply a specified level to the resulting context as well.

Another method that is available is the `get_default_type()` method, which returns the default type for a given role. As with the other methods, this results in the SELinux code to query configuration files inside `/etc/selinux/`; in this particular case, the `default_type` file inside `/etc/selinux/mcs/contexts/`.

# Interrogating the SELinux subsystem code-wise

In order to query the SELinux policy, we have seen the use of the `sesearch` command and other SELinux utilities. Code-wise, SELinux policies can be queried using the `security_compute_av_flags` method.

## Getting ready

The `curcon` and `newcon` variables can be filled in through methods such as `getcon()` (for the current context) or `get_default_context()` as we have seen in the previous recipe.

## How to do it…

As an example, we want to query the transition permission between two process domains. To accomplish this, the following method is used:

1.  First of all, call the `security_compute_av_flags()` method:

    ```
    struct av_decision avd;
    rc = security_compute_av_flags(curcon, newcon, SECCLASS_PROCESS, PROCESS__TRANSITION, &avd);
    if (rc) {
      … // Method failed.
      freecon(curcon);
      freecon(newcon);
    };
    ```

2.  Now read the response:

    ```
    if (!(avd.allowed & PROCESS__TRANSITION)) {
      … // Transition is denied
    };
    ```

3.  Check whether the current context is a permissive domain or not:

    ```
    if (avd.flags & SELINUX_AVD_FLAGS_PERMISSIVE) {
      … // Domain is permissive
    };
    ```

## How it works…

The `security_compute_av_flags()` method is the C method equivalent of `sesearch` (roughly speaking). It takes the source and target context, class, and permission and stores the result of the query in a specific structure (`struct av_decision`).

The class and permission entries can be obtained from the `flask.h` (for the class declarations) and the `av_permissions.h` (for the permission declarations) header files that are located inside `/usr/include/selinux/`.

The result of the query is obtained by checking whether the permission is in the decision result.

Next to the permission query, an important aspect to validate (and which is often forgotten by SELinux-aware applications) is to check whether the domain itself is marked as permissive. After all, even on an SELinux-enabled system, where SELinux is in enforcing mode, some domains can still be marked as permissive.

The `SELINUX_AVD_FLAGS_PERMISSIVE` flag is a flag added to the query response (`struct av_decision`), which allows developers to query the permissive state of domains. With this information at hand, the SELinux-aware application can still decide to continue even if the policy denies a certain activity, just as the user has requested.

## There's more...

There are other methods available as well to query the SELinux policy that might be used by SELinux-aware applications.

With `selinux_check_access()`, for instance, applications can query the SELinux policy to see if a given source context has the access permission for a given class and permission on the target context. This is not the same as `security_compute_av_flags()`, as this method uses strings for the class and permission, and also has a different return based on the enforcing state of SELinux or the permissive nature of a particular domain.

# Running new processes in a new context

Sometimes, it isn't possible to force a particular domain upon invocation of a new task or process. The default transition rules that can be enabled through the SELinux policy are only applicable if the source domain and file context (of the application or task to execute) are unambiguously decisive for the target context.

In applications that can run the same command (or execute commands with the same context) for different target domains, SELinux-awareness is a must.

This recipe will show how to force a particular domain for a new process.

## Getting ready

The `newcon` variable that is used in this recipe can be filled in through methods such as `get_default_context()` as we have seen in a previous recipe.

## How to do it…

To launch a process in a specific context, go through the following steps:

1.  Tell SELinux what the new context should be:

    ```
    int rc = setexeccon(newcon);
    if (rc) {
      … // Call failed
      freecon(newcon);
    };
    ```

2.  Fork and execute the command. For instance, to execute `id -Z`, the following code is used:

    ```
    pid_t child;
    child = fork();
    if (child < 0) {
      … // Fork failed } else if (child == 0) {
      int pidrc;
      pidrc = execl("/usr/bin/id", "id", "-Z", NULL);
      if (pidrc != 0) {
        … // Command failed
      };
    } else {
      … // Parent process
      int status;
      wait(&status);
    };
    ```

## How it works…

Applications that want newly executed tasks to run in a particular context need to tell the SELinux subsystem that the next `execve`, `execl`, or other `exec*` method should result in the child process running in the new domain.

Of course, the SELinux policy must still allow the transition policy-wise, even though there is no more need for an automatic domain transition in the policy (as this would require an unambiguous decision, which is exactly what isn't possible if the source domain and file context are the same for different target contexts):

```
allow crond_t self : process setexec;
allow crond_t staff_t : process transition;
```

The `setexec` permission allows the source domain to explicitly tell the SELinux subsystem what context the task should run in. Without this permission, the call to `setexeccon()` would fail.

## There's more...

The `setexeccon()` method has a sibling method called `getexeccon()`. This method returns the context that would be assigned when executing a new process (which would provide a validation of the last `setexeccon()` call).

Another similar method is the `setexecfilecon()` method. This method allows SELinux-aware applications to take the SELinux policy decisions into account in case of file-based transition information. So, if there is a domain transition known when executing a particular file, then this domain transition is honored. If not, the fallback type provided through the `setexecfilecon()` method is used:

```
char * fallbackcon = "system_u:object_r:openscap_helper_script_t:s0";
char * filename = "/usr/libexec/openscap/probe_process";
…rc = setexecfilecon(filename, fallbackcon);
```

In this example, if the context of the `probe_process` file is used in the SELinux policy to create an automatic domain transition upon invocation by the current application, then that target domain is used for the application execution. However, if the context of the `probe_process` file is the one that does not trigger any automatic domain transition, then the `fallbackcon` context is used for the next application execution.

# Reading the context of a resource

It is, of course, also important to obtain the context of a resource if the application is SELinux-aware. This could be for logging purposes or to decide which domain to transition to (based on the resource context, current context, username, and so on).

## How to do it…

To read the context of a resource, the following methods are available:

1.  Given a file path, the following call to `getfilecon()` will provide the context of the file:

    ```
    security_context_t filecon = 0;
    char * path = "/etc/passwd";
    rc = getfilecon(path, &filecon);
    if (rc < 0) {
      … // Call failed
    };
    … // Do stuff with the context
    freecon(filecon);
    ```

2.  To get the context of a process, assuming the `pid` variable (of the `pid_t` type) has the proper process ID in it, the following code is used:

    ```
    security_context_t pidcon = 0;
    rc = getpidcon(pid, &pidcon);
    if (rc < 0) {
      … // Call failed
    };
    … // Do stuff with the context
    freecon(pidcon);
    ```

## How it works…

The SELinux library has various methods for obtaining the contexts of resources. File and process types are shown in the recipe, but other methods exist as well. For instance, with the `fgetfilecon()` method, the context of a file descriptor can be obtained. All these methods provide the context in a standard string (`char *`) format.

After getting the context of a resource, it is important to free the context when it is no longer used. Otherwise, a memory leak will occur in the application as there are no other methods that will clean up the contexts.

## There's more...

When labeled networking is used (for instance, with CIPSO/NetLabel support or labeled IPSec), then the `getpeercon()` method can be used to obtain the context of the peer that participates in the communication session.

Alongside querying the context, it is also possible to tell the SELinux subsystem that file creation should result in that file being created immediately with a particular context. For this, the `setfscreatecon()` method can be used—this is also the method that recent udev versions use when creating new device files in `/dev/`.