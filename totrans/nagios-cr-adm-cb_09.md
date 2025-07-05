# Chapter 9. Managing Configuration

In this chapter, we will cover the following recipes:

*   Grouping configuration files in directories
*   Keeping configuration under version control
*   Configuring host roles using groups
*   Building groups using regular expressions
*   Using inheritance to simplify configuration
*   Defining macros in a resource file
*   Dynamically building host definitions

# Introduction

A major downside of Nagios Core's configuration being so flexible is that without proper management, a configuration can easily balloon out into hundreds of files with thousands of objects, all having unclear dependencies. This can be frustrating when attempting to make significant changes to a configuration, or even for something as simple as removing a host, sifting through dependencies to find what's causing errors in the configuration and prevents you from restarting Nagios Core.

It's therefore important to build your configuration carefully using as much abstraction as possible, to allow adding, changing, and removing hosts and service definitions from the configuration painlessly, and to avoid duplication of configuration. Nagios Core provides a few ways of dealing with this, most notably in the judicious use of groups and templates for the fundamental objects. Duplication of network-specific and volatile data, such as passwords, is also to be avoided; it's best done with the use of custom macros defined in a resource file.

This chapter's recipes will run through some examples of good practice for the configuration of a large network. The most important recipes are the first two, *Grouping configuration files in directories* and *Configuring host roles using groups*. If you're looking to untangle and revamp a messy configuration, then these two recipes would be the best place to start.

At the end of the chapter, the final recipe, *Dynamically building host definitions*, will show one of the primary advantages of a tidy configuration in being able to easily generate configuration according to a list of hosts and services kept in some other external information source, such as a **Configuration Management Database** (**CMDB**).

# Grouping configuration files in directories

In this recipe, we'll learn to group configuration files in directories to greatly ease the management of configuration. We'll do this by configuring Nagios Core to load every file it can find ending with a `.cfg` extension in a given directory, including recursing through subdirectories. The end result will be that to have Nagios Core load a file, we only need to include it somewhere in that directory with an appropriate extension; we don't need to define exactly which files are being loaded in `nagios.cfg`.

## Getting ready

You will need to have a server running Nagios Core 3.0 or later, and have access to the command line to change its configuration. You should be familiar with the loading of individual configuration files using the `cfg_file` directive in `/usr/local/nagios/etc/nagios.cfg`.

In particular, you should have a directory prepared that contains all of the configuration files you would like to be loaded by Nagios Core. In this example, we'll prepare a new directory called `/usr/local/nagios/etc/naginet`, which will contain three configuration files, each defining information for one host:

```
# ls -1 /usr/local/nagios/etc/naginet
athens.cfg
ithaca.cfg
sparta.cfg

```

You will need to ensure that the directory, files, and subdirectories within it are all readable (though not necessarily owned) by the `nagios` user.

## How to do it...

We can arrange for Nagios Core to include all the files in a directory as follows:

1.  Change to the directory containing the `nagios.cfg` file. In the default installation, it is located at `/usr/local/nagios/etc/nagios.cfg`.

    ```
    # vi nagios.cfg

    ```

2.  Edit the file, and below any other `cfg_file` or `cfg_dir` directives, add the following line referring to the absolute path of the directory containing your `.cfg` files:

    ```
    cfg_dir=/usr/local/nagios/etc/naginet
    ```

    Note that this directive is `cfg_dir`, rather than `cfg_file`.

3.  Remove any `cfg_file` definitions pointing to `.cfg` files in the new directory. This is to prevent loading the same objects twice. In our example, we would need to comment out our previous rules loading these files individually:

    ```
    #cfg_file=/usr/local/nagios/etc/naginet/sparta.cfg
    #cfg_file=/usr/local/nagios/etc/naginet/athens.cfg
    #cfg_file=/usr/local/nagios/etc/naginet/ithaca.cfg

    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, Nagios Core should have loaded all of the files with the extension `.cfg` found in the `naginet` directory or any of its subdirectories, saving us the burden of specifying them individually.

## How it works...

At load time, Nagios Core interprets the `cfg_dir` directive to mean that it should identify all of the `.cfg` files in a particular directory, including recursing through its subdirectories. It ignores the files that do not have that extension, allowing us to include metadata or other file types (such as version control information directories) without causing problems.

As a result, defining a new host or service becomes as simple as adding the file in an appropriate included directory, without needing to edit `nagios.cfg`.

## There's more...

It's important to note that this directive will include all the `.cfg` files in subdirectories as well. This might allow us to meaningfully group the configuration files:

```
$ find /usr/local/nagios/etc/naginet
/usr/local/nagios/etc/naginet
/usr/local/nagios/etc/naginet/hosts
/usr/local/nagios/etc/naginet/hosts/athens.cfg
/usr/local/nagios/etc/naginet/hosts/sparta.cfg
/usr/local/nagios/etc/naginet/hosts/ithaca.cfg
/usr/local/nagios/etc/naginet/commands
/usr/local/nagios/etc/naginet/commands/webserver-checks.cfg
/usr/local/nagios/etc/naginet/services
/usr/local/nagios/etc/naginet/services/webservers.cfg

```

For large networks, it's worth deciding on some suitable organizing principle for directories. One common approach is having a separate directory for hosts, services, and command definitions, relevant to a particular **Domain Name System** (**DNS**) zone or larger subnet.

If you want to prevent a file from being included at any point, all you need to do is either move it out of the directory, or rename it so that it no longer has a `.cfg` extension. One possibility is adding the suffix `.exclude`:

```
# mv unwanted-file.cfg unwanted-file.cfg.exclude

```

This will prevent Nagios Core from picking it up as part of its `cfg_dir` searching algorithm.

## See also

*   The *Using inheritance to simplify configuration* and *Keeping configuration under version control* recipes in this chapter

# Keeping configuration under version control

In this recipe, we'll place a Nagios Core configuration directory under version control, in an attempt to keep track of changes made to it, and to enable us to reverse changes if there are problems.

## Getting ready

You should choose an appropriate version control system. The recipe will vary considerably depending on which system you use; there are too many options to demonstrate here, so we'll use the popular open source content tracker **Git**, the basics of which are very easy to use for this kind of version control and do not require an external server. However, there's no reason you can't use **Subversion** or **Mercurial**, if you'd prefer. You should have the client for your chosen system (`git`, `hg`, `svn`, and so on) installed on your server.

This will all work with any version of Nagios Core. It does not involve directly changing any part of the Nagios Core configuration, only keeping track of the files in it.

## How to do it...

We can place our configuration directory under version control as follows:

1.  Change to the root of the configuration directory. In the default installation, this is `/usr/local/nagios/etc`:

    ```
    # cd /usr/local/nagios/etc

    ```

2.  Run `git init` to start a repository in the current directory:

    ```
    # git init
    Initialized empty Git repository in /usr/local/nagios/etc/.git/.

    ```

3.  Add all the files in the configuration directory with `git add`:

    ```
    # git add .

    ```

4.  Commit the files with `git commit`:

    ```
    # git commit -m "First commit of configuration"
    [master (root-commit) 6a2c605] First commit of configuration.
     39 files changed, 3339 insertions(+), 0 deletions(-)
     create mode 100644 naginet/athens.naginet.cfg
     create mode 100644 naginet/ithaca.naginet.cfg
     create mode 100644 naginet/sparta.naginet.cfg
     ...

    ```

With this done, we should now have a `.git` repository in `/usr/local/nagios/etc`, tracking all the changes made to the configuration files.

## How it works...

Changes to the configuration files in this directory can now be reviewed for commit with `git status`. For example, if we changed the IP addresses of one of our hosts, we might check the changes by typing the following:

```
# git status -s
M naginet/hosts/sparta.cfg

```

We could then commit this change with an explanatory message:

```
# git commit -am "Changed IP address of host"
[master d5116a6] Changed IP address of host
1 files changed, 1 insertions(+), 1 deletions(-)

```

We can review the change with `git log`:

```
# git log --oneline
d5116a6 Changed IP address of host
6a2c605 First commit of configuration

```

If we want to inspect exactly what was changed later on, we can use `git diff` followed by the short commit ID given in the first column of the preceding output:

```
# git diff 6a2c605
diff --git a/naginet/hosts/sparta.cfg b/naginet/hosts/sparta.cfg
index 0bb3b0b..fb7c2a9 100644
--- a/naginet/hosts/sparta.cfg
+++ b/naginet/hosts/sparta.cfg
@@ -2,7 +2,7 @@ define host {
 use                 linux-server
 host_name           sparta.naginet
 alias               sparta
-    address             10.128.0.101
+    address             10.128.0.102
}

```

The full functionality of Git for managing these changes, including reverting to older revisions, is out of scope here. You can read more about how to use Git in general in Scott Chacon's excellent book entitled *Pro Git*, free to read online at [http://git-scm.com/book](http://git-scm.com/book).

## There's more...

Version control is particularly useful in this way when more than one person is editing a configuration, because it allows us to determine who made a change and when. It also allows us to see the exact changeset to review why it was changed, and to undo or edit it if it is causing problems.

If you're going to use this method, it's a good idea to keep your configuration reasonably granular, using at least several files rather than just one or two. It will still work if you have two big files such as `hosts.cfg` and `services.cfg` for your network, but the differences between each commit will not be as clear. This is therefore a very good recipe to combine with the *Grouping configuration files in directories* recipe, also in this chapter.

Rather than merely the configuration directory, you may prefer to keep the entire Nagios Core directory under version control, including the plugins and other scripts and binaries. This could be particularly handy if you upgrade your installation with new releases and want to see what's changed in your files, in case it breaks anything. In this case, be careful to use your chosen version control system's "ignore" functionality to prevent tracking temporary files or log files. For Git, take a look at the output of `git help ignore`.

## See also

*   The *Grouping configuration files in directories* recipe in this chapter

# Configuring host roles using groups

In this recipe, we'll learn how to use the abstraction of host and service groups to our advantage in order to build a configuration where hosts and services can be added or removed more easily. We'll do this by defining roles for hosts by using a hostgroup structure, and then assigning relevant services to the hostgroup, rather than to the hosts individually.

## Getting ready

You will need to have a server running Nagios Core 3.0 or later, have access to the command line to change its directories, and understand the basics of how host and service groups work. These are covered in the *Creating a new hostgroup* and *Creating a new servicegroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts").

In this example, we'll create two simple hostgroups; one called `servers`, for which a `PING` check should be made for its member hosts, and another called `webservers`, which should include `HTTP` checks for its member hosts. Once this is set up, we'll then add an example host `sparta.naginet` to both groups, thereby easily assigning all the appropriate services to the host in one definition, which we can cleanly remove simply by deleting the host.

## How to do it...

We can create our group-based roles for hosts as follows:

1.  Change to the `objects` configuration directory. In the default installation, this is `/usr/local/nagios/etc/objects`. If you have already followed the *Grouping configuration files in directories* recipe in this chapter, then your own directory may differ.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  In an appropriate file, perhaps `hostgroups.cfg`, define new hostgroups with names corresponding to the roles for the hosts. Do not assign them any members just yet.

    ```
    define hostgroup {
        hostgroup_name  servers
        alias           Servers
    }
    define hostgroup {
        hostgroup_name  web-servers
        alias           Web Servers
    }
    ```

3.  In a second appropriate file, perhaps `services.cfg`, define new services and assign them `hostgroup_name` values, corresponding to the hostgroups added in the previous step:

    ```
    define service {
        use                  generic-service
     hostgroup_name       servers
        service_description  PING
        check_command        check_ping!100,10%!200,20%
    }
    define service {
        use                  generic-service
     hostgroup_name       web-servers
        service_description  HTTP
        check_command        check_http
    }
    ```

    Note that the use of the `generic-service` template is an example only; you will probably want to inherit from your own particular service template.

4.  Add or edit your existing hosts to make them a part of the appropriate hostgroups, using the `hostgroups` directive:

    ```
    define host {
        use         linux-server
        host_name   sparta.naginet
        alias       sparta
        address     10.128.0.21
     hostgroups  servers,web-servers
    }
    define host {
        use         linux-server
        host_name   athens.naginet
        alias       athens
        address     10.128.0.22
     hostgroups  servers,web-servers
    }
    ```

5.  If you already had services with the same value for their `service_description` directives as the ones you're adding in this recipe, you will need to remove them, as this may cause a conflict with the services added in the previous step.
6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, you should now find that all of the services you defined for the hostgroups you created have been attached to the appropriate hosts:

![How to do it...](img/5566_09_01.jpg)

## How it works...

The configuration added in the preceding section avoids assigning services directly to hosts, instead assigning them to hostgroups, thereby creating roles for services that hosts can adopt or discard simply by becoming part of or leaving the group.

Apart from making the configuration shorter, another advantage to this approach is that if services are added this way, adding or deleting a host from the Nagios Core configuration requires nothing but the adding or removing of the host definition. Similarly, if a host takes on another role (for example, a web server adding some database functionality), then we can modify the services being checked on it simply by modifying its hostgroups. This ends up being much easier than adding dependencies in other files.

Another advantage is that having hostgroups organized by host function is helpful for applying batch operations such as scheduled downtime in one easy action, or for checking all the services for one particular type of host. Once a hostgroup is defined, we can run operations on all the hosts within it by clicking on its name in the brackets, in any of the hostgroup views:

![How it works...](img/5566_09_02.jpg)

If we had twenty web servers that we knew were going to be down, for example, this would be much easier than scheduling downtime for each of them individually!

## There's more...

It's worth noting that hostgroups can have subgroups, meaning that all of the hosts added to any of their subgroups are implicitly added to the parent group.

For example, we could define the hostgroup web servers as a subgroup of the `servers` hostgroup, using the `hostgroup_members` directive:

```
define hostgroup {
    hostgroup_name     servers
    alias              Servers
 hostgroup_members  web-servers
}
```

This would allow us to implicitly add hosts to both groups, without needing to refer to the parent group, and with all the services assigned to both groups assigned to the host:

```
define host {
    use         linux-server
    host_name   athens.naginet
    alias       athens
    address     10.128.0.22
 hostgroups  web-servers
}
```

This can be very useful for sorting "subcategories" of services. Other examples might include a `dns-servers` group with subgroups `dns-authoritative-servers` and `dns-recursive-servers`, or a `database-servers` group with subgroups `oracle-servers` and `mysql-servers`.

## See also

*   The *Building groups using regular expressions* and *Using inheritance to simplify configuration* recipes in this chapter
*   The *Creating a new hostgroup*, *Creating a new servicegroup*, and *Running a service on all hosts in a hostgroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Building groups using regular expressions

In this recipe, we'll learn a shortcut for building groups of hosts using regular expressions tested against their hostnames.

This recipe is likely only of use to you if you use a naming convention for your hosts that allows them to be reasonably grouped by location, function, or some other useful metric by a common string in their hostnames.

## Getting ready

You will need to have a server running Nagios Core 3.0 or later, have access to the command line to change its configuration, and understand the basics of how hostgroups and servicegroups work. These are covered in the *Creating a new hostgroup* and *Creating a new servicegroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts").

In this example, we'll group three existing hosts named `web-server-01`, `web-server-02`, and `web-server-03` into a new hostgroup, `web-servers`, based only on their hostnames.

It would help to have some familiarity with regular expressions, but the recipe includes a simple example, which should meet many use cases for this trick. An excellent site about regular expressions including tutorials can be found at [http://www.regular-expressions.info/](http://www.regular-expressions.info/).

## How to do it...

We can build a hostgroup by matching regular expressions to hostnames as follows:

1.  Change to the Nagios configuration directory. In the default installation, this is `/usr/local/nagios/etc`. Edit the `nagios.cfg` file in this directory.

    ```
    # cd /usr/local/nagios/etc
    # vi nagios.cfg

    ```

2.  Search for the `use_regexp_matching` directive. Uncomment it if necessary and set it to `1`.

    ```
    use_regexp_matching=1
    ```

3.  Search for the `use_true_regexp_matching` directive. Uncomment it if necessary, and ensure it's set to `0`, which it should be by default.

    ```
    use_true_regexp_matching=0
    ```

4.  Change to the `objects` configuration directory. In the default installation, this is `/usr/local/nagios/etc/objects`. If you have already followed the *Grouping configuration files in directories* recipe in this chapter, then your own directory may differ.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

5.  In an appropriate file, perhaps `hostgroups.cfg`, add a definition similar to the following. In this case, `.+` means "any string at least one character in length"; you will probably want a pattern of your own, devising appropriate to your own host names.

    ```
    define hostgroup {
        hostgroup_name  web-servers
     members         web-server-.+
        alias           Web Servers
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, if your regular expression was correctly formed to match all the appropriate hostnames, then you should find that the hosts become part of the group:

![How to do it...](img/5566_09_03.jpg)

## How it works...

With the `use_regexp_matching` directive set to `1`, Nagios Core will attempt to use hostname strings containing the strings `*`, `?`, `+`, or `\.` as regular expressions to match against hostnames. Because `web-server-01`, `web-server-02`, and `web-server-03` all match the regular expression `web-server-.+` given in the `members` directive for the `web-servers` hostgroup, all three hosts are added to the group.

We keep `use_true_regexp_matching` off. If it were on, it would use every hostname pattern as a regular expression, whether or not it had any special regular expression characters. This is probably not what you want for most configurations.

## There's more...

This matching works in other places aside from hostgroup definitions; for example, you can also use it in the `host_name` definitions for services:

```
define service {
    use                  generic-service
 host_name            web-server-.+
    service_description  HTTP
    check_command        check_http
}
```

This is one of a number of very good suggestions for simplifying object definitions suggested in the Nagios Core manual: [http://nagios.sourceforge.net/docs/3_0/objecttricks.html](http://nagios.sourceforge.net/docs/3_0/objecttricks.html).

## See also

*   The *Building groups using regular expressions* and *Using inheritance to simplify configuration* recipes in this chapter
*   The *Creating a new hostgroup* and *Creating a new servicegroup* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Using inheritance to simplify configuration

In this recipe, we'll learn how to use inheritance to handle the situation where hosts and services share a lot of values in common, amounting to a large amount of undesirable redundancy in configuration.

Some Nagios Core objects, particularly hosts and services, have a rather long list of possible directives, and the default values for these are not always suitable. It's therefore worthwhile to be able to declare the values you want for these directives once, and then spend only a few lines on the actual host definition by copying those values from a template, making the configuration shorter and easier to read.

Previous examples in this book have already demonstrated the use of this in suggesting you inherit from the `linux-server` host template or the `generic-service` service template, for the sake of brevity; in this example, we'll define our own templates, and show how these can be used to streamline a configuration.

## Getting ready

You will need to have a server running Nagios Core 3.0 or later, have access to the command line to change its configuration, and be familiar with how to define hosts and services. These are covered in the *Creating a new host* and *Creating a new service* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts").

In this example, we'll define a template, `critical-host`, which we'll use as the basis for any host that needs to be checked and notified around the clock, with a very stringent value for its `check_command` directive, and with all notification types enabled. We'll also define two hosts named `phobos.naginet` and `deimos.naginet` that inherit from this template.

## How to do it...

We can define a host template and then define hosts to inherit from it as follows:

1.  Change to the `objects` configuration directory. In the default installation, this `is /usr/local/nagios/etc/objects`. If you have already followed the *Grouping configuration files in directories* recipe in this chapter, then your own directory may differ.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  In an appropriate file, perhaps `templates.cfg`, add the following definition. Note the use of the special directives `name` and `register`:

    ```
    define host {
     name                   critical-host
     register               0
        check_command          check_ping!25,10%!50,20%
        max_check_attempts     3
        check_interval         1
        notification_interval  1
        notification_period    24x7
        notification_options   d,u,r,f,s
        check_period           24x7
        contact_groups         admins
    }
    ```

3.  In another file, or separate files if you prefer to keep your hosts to one per file, define hosts to inherit from this template. Add in the remaining required directives for hosts, and include a `use` directive referring to the established template:

    ```
    define host {
     use        critical-host
        host_name  phobos.naginet
        alias      phobos
        address    10.128.0.151
    }
    define host {
     use        critical-host
        host_name  deimos.naginet
        alias      deimos
        address    10.128.0.152
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

    You might receive warnings about no services being defined for the hosts, but you can ignore those for now.

With this done, two new hosts should be registered in your configuration, **phobos.naginet** and **deimos.naginet**:

![How to do it...](img/5566_09_04.jpg)

No other new hosts should be added, as the template itself was explicitly not registered as an object with the `register` directive.

## How it works...

The configuration added in the preceding section defines a new template with the `name` directive of `critical-host`. Because the value for the host's register directive is `0`, where it normally defaults to `1`, the host is not registered as an object with Nagios Core. Instead, it becomes a snippet of configuration that can be referenced by real objects of the same type, by referring to its name.

Note that in the template, the normally required values of `host_name`, `alias`, and `address` are missing; this means that the host is not complete and wouldn't work if we tried to register it as an actual host anyway.

Instead, we use its values as the basis for other hosts, `phobos.naginet` and `deimos.naginet`. Both of these hosts inherit from `critical-host`, and fill in the rest of the missing values in their own definitions. This frees us from having to repeat the same directives in two different hosts.

If an object inherits a value for a directive from its parent, it's possible to override that directive by redefining it in the inheriting object definition. For example, if we wanted `phobos.naginet` to have a different value for `max_check_attempts`, we can add that to its definition:

```
define host {
    use                 critical-host
    host_name           deimos.naginet
    alias               deimos
    address             10.128.0.152
 max_check_attempts  5
}
```

## There's more...

The most important thing to note about templates is that they work for a variety of Nagios Core objects, most importantly including hosts, services, and contacts. You can therefore set up and inherit from a service template in the same way:

```
define service {
    name      critical-service
    register  0
    ...etc...
}
define service {
    use       critical-service
    ...etc...
}
```

Or a contact template:

```
define contact {
    name      critical-contact
    register  0
    ...etc...
}
define contact {
    use       critical-contact
    ...etc...
}
```

Note that inheritance can stack. For example, `critical-host` could itself inherit from a template, perhaps `generic-host`, by adding its own `use` directive:

```
define host {
 use       generic-host
    name      critical-host
    register  0
    ...etc...
}
```

This allows the setting up of an inheritance structure of arbitrary complexity, but you should avoid too much depth to prevent confusing yourself or anyone else trying to read your configuration. Two levels is probably a sensible limit.

The rules around how inheritance is handled are discussed in more depth in the Nagios Core manual, including a treatment of multiple inheritance. It's useful to know about this, but in the interests of keeping configuration clear, it's best used sparingly: [http://nagios.sourceforge.net/docs/3_0/objectinheritance.html](http://nagios.sourceforge.net/docs/3_0/objectinheritance.html).

## See also

*   The *Configuring host roles using groups* recipe in this chapter
*   The *Creating a new host*, *Creating a new service*, and *Creating a new contact* recipes in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*

# Defining macros in a resource file

In this recipe, we'll learn how to define custom user macros in resource files. This is good practice for strings used in `check_command` definitions or other directives that are shared by more than one host or service. For example, in lieu of writing the full path in a `command_name` directive as follows:

```
command_name=/usr/local/nagios/libexec/check_ssh $HOSTADDRESS$
```

We could instead write:

```
command_name=$USER1$/check_ssh $HOSTADDRESS$
```

As a result, if the location of the `check_ssh` script changes, we only need to change the value of `$USER1$` in the appropriate resource file to update all of its uses throughout the configuration.

Most of the macros in Nagios Core are defined automatically by the monitoring server, but up to 32 user-defined macros can be used as well, in the form `$USERn$`.

## Getting ready

You will need to have a server running Nagios Core 3.0 or later, and have access to the command line to change its configuration, in particular the `resource.cfg` file.

In this example, we'll add a new macro, `$USER2$`, to contain the SNMP community name `snagmp`, as used for various `check_snmp` requests.

## How to do it...

We can define our user macro as follows:

1.  Change to the Nagios configuration directory. In the default installation, this is `/usr/local/nagios/etc`. Edit the `resource.cfg` file in this directory.

    ```
    # cd /usr/local/nagios/etc
    # vi resource.cfg

    ```

2.  Ensure that the `$USER2$` macro is not already defined in the file. If it is, we could define `$USER3$` instead, and so on.
3.  Add the following definition to the end of the file:

    ```
    $USER2$=snagmp
    ```

4.  Change to the `objects` configuration directory for Nagios Core. In the default installation, this is `/usr/local/nagios/etc/objects`.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

5.  Edit any of the `object` configuration files in which you wish to use the value of the macro, and replace the inline values with `$USER2$`. In our example, we might find various uses of the literal `checksnmp` community string:

    ```
    define command {
        ...
        command_line  $USER1$/check_snmp -H $HOSTADDRESS$ -C snagmp -o .1.3.6.1.2.1.1.5.0 -r $ARG1$
    }
    define service {
        ...
        check_command  check_snmp!snagmp
    }
    ```

    We could swap these out to use the macro instead:

    ```
    define command {
        ...
     command_line  $USER1$/check_snmp -H $HOSTADDRESS$ -C $USER2$ -o .1.3.6.1.2.1.1.5.0 -r $ARG1$
    }
    define service {
        ...
     check_command  check_snmp!$USER2$
    }
    ```

6.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, all the monitoring should be working the same as it was before, but we're now using macro expansion to centralize configuration.

## How it works...

Before Nagios Core processes directives such as `command_line` and `check_command`, it will first expand all the macros referenced, including the user-defined macros we added in the `resource.cfg` resource file.

One common use for the `$USERn$` macros is for defining the full path of directories where Nagios Core resources such as plugins or event handler scripts are located— in fact, the sample configuration included in Nagios Core defines `$USER1$` in `resource.cfg` as `/usr/local/nagios/libexec`, the default location for plugin scripts and binaries.

It's worth noting that you can load more than one resource file by adding more `resource_file` directives in `/usr/local/nagios/etc/nagios.cfg`. For example, to load another file called `resource-extra.cfg`, we could add a second line as follows:

```
resource_file=/usr/local/nagios/etc/resource.cfg
resource_file=/usr/local/nagios/etc/resource-extra.cfg

```

## There's more...

There's also a security benefit to using resource files—for sensitive information, we can prevent users other than `nagios` from reading them by making them readable only to the `nagios` user:

```
# cd /usr/local/nagios/etc
# chown nagios.nagios resource.cfg
# chmod 0600 resource.cfg

```

This makes it a decent way to store credentials, such as usernames and passwords, for example to be used in `check_command` for MySQL:

```
define command {
    command_name  check_mysql_secure
 command_line  check_mysql -H $HOSTADDRESS$ -u naguser -d nagdb -p $USER3$
}
```

You can also define per-host macros, by using custom directives preceded with an underscore in the host definition:

```
define host {
    use                 critical-host
    host_name           sparta.naginet
    alias               sparta
    address             10.128.0.21
 _mac_address        08:00:27:7e:7c:d2
}
```

In the preceding example, we're able to include the host's MAC address in a custom directive; this can be referenced in services as `$_HOSTMAC_ADDRESS$`:

```
define service {
    use                  generic-service
    host_name            sparta.naginet
    service_description  ARP
    check_command        check_arp!$_HOSTMAC_ADDRESS$
}
```

The same trick can also apply for contacts and services. This special use of custom macros is discussed in the *Custom Object Variables* chapter of the Nagios documentation at [http://nagios.sourceforge.net/docs/3_0/cu](http://nagios.sourceforge.net/docs/3_0/customobjectvars.html).

## See also

*   The *Monitoring the output of an SNMP query* recipe in [Chapter 5](ch05.html "Chapter 5. Monitoring Methods"), *Monitoring Methods*
*   The *Monitoring individual nodes in a cluster* recipe in [Chapter 8](ch08.html "Chapter 8. Managing Network Layout"), *Understanding the Network Layout*

# Dynamically building host definitions

In this recipe, we'll learn one possible method of building Nagios configuration dynamically, to avoid having to compose or copy-paste a lot of directives for new hosts or services. In other words, this recipe is about generating configuration using templates.

To demonstrate how this is useful, we'll use the `m4` macro language utility, which should be available on virtually any UNIX-like system, including GNU/Linux and BSD. As a tool designed for macro expansion, `m4` is particularly well-suited to creating verbose plain text configuration files such as the ones used by Nagios Core.

The principles here should apply just as easily to your favored programming or templating language, perhaps Python or Perl, or shell scripts.

## Getting ready

You will need to have the `m4` macro language tool available to you, preferably but not necessarily on the same system as the one running Nagios Core. It is a very standard tool and should be already installed, or available as part of a package. The version used in this example is **GNU m4**, documented at [http://www.gnu.org/software/m4/manual/m4.html](http://www.gnu.org/software/m4/manual/m4.html). This recipe does not assume any familiarity with `m4`, and will show you the basics.

You may like to work in a new subdirectory in your home directory:

```
$ mkdir $HOME/nagios-dynamic-hosts
$ cd $HOME/nagios-dynamic-hosts

```

## How to do it...

We can create and apply an example Nagios Core configuration template as follows:

1.  Create a new file `host-service-template.m4` with the following contents:

    ```
    define(`NAGHOST', `
        define host {
            host_name              $1
            alias                  $2
            address                $3
            contact_groups         ifelse(`$#', `4', `$4', `admins')
            check_command          check-host-alive
            check_interval         5
            check_period           24x7
            max_check_attempts     3
            notification_interval  30
            notification_options   d,r
            notification_period    24x7
        }
        define service {
            host_name              $1
            contact_groups         ifelse(`$#', `4', `$4', `admins')
            check_command          check_ping!100,10%!200,20%
            check_interval         5
            check_period           24x7
            max_check_attempts     3
            notification_interval  30
            notification_period    24x7
            retry_interval         1
            service_description    PING
        }
    ')
    ```

2.  Create a second file in the same directory called `sparta.host.m4` with the following contents:

    ```
    include(`host-service-template.m4')
    NAGHOST(`sparta', `Sparta Webserver', `10.0.128.21')
    ```

3.  Create a third file in the same directory called `athens.host.m4` with the following contents:

    ```
    include(`host-service-template.m4')
    NAGHOST(`athens', `Athens Webserver', `10.0.128.22', `ops')
    ```

4.  Run the following commands and note the output:

    ```
    $ m4 sparta.host.m4
    define host {
     host_name              sparta
     alias                  Sparta Webserver
     address                10.0.128.21
     contact_groups         admins
     check_command          check-host-alive
     check_interval         5
     check_period           24x7
     max_check_attempts     3
     notification_interval  30
     notification_options   d,r
     notification_period    24x7
    }
    define service {
     host_name              sparta
     contact_groups         admins
     check_command          check_ping!100,10%!200,20%
     check_interval         5
     check_period           24x7
     max_check_attempts     3
     notification_interval  30
     notification_period    24x7
     retry_interval         1
     service_description    PING
    }
    $ m4 athens.host.m4
    define host {
     host_name              athens
     alias                  Athens Webserver
     address                10.0.128.22
     contact_groups         ops
     check_command          check-host-alive
     check_interval         5
     check_period           24x7
     max_check_attempts     3
     notification_interval  30
     notification_options   d,r
     notification_period    24x7
    }
    define service {
     host_name              athens
     contact_groups         ops
     check_command          check_ping!100,10%!200,20%
     check_interval         5
     check_period           24x7
     max_check_attempts     3
     notification_interval  30
     notification_period    24x7
     retry_interval         1
     service_description    PING
    }

    ```

As seen in the preceding output, we can now generate a basic host and service configuration with a two-line `m4` script referring to a template, simply by writing the output to a `.cfg` file:

```
$ m4 sparta.host.m4 > sparta.cfg

```

## How it works...

The files `sparta.host.m4` and `athens.host.m4` both called an `m4` macro with arguments, after including the template for the host and service in the `host-service-template.m4` file. This was expanded into the full definition, and the arguments given were substituted as follows:

*   `$1` was replaced with the first argument, `host_name`
*   `$2` was replaced with the second argument, `alias`
*   `$3` was replaced with the third argument, `address`
*   `$4` was replaced with the fourth argument, `contact_group`

Note that two of these values, `$1` and `$4`, were used in both the host and the `PING` service definitions.

Also note that the argument `$4` is optional; the `if-else` construct tests the number of arguments, and if it finds there are four, it uses the value of the fourth argument; for `athens.naginet`, this is the contact group `ops`. If there is no fourth argument, it defaults instead to the value `admins`. This allows us to set default values for arguments if we so choose.

The rest of the directives are all written directly into the template. The configuration made by this process is valid for Nagios Core, assuming that the `check_command` and `contact_groups` used are defined.

## There's more...

To automate things even further, we could use `make` to automatically generate `.cfg` files from anything with the extension `.host.m4` with the following `Makefile`:

```
%.cfg : %.host.m4
    m4 $< > $*.cfg
```

Note that correct `Makefile` syntax usually requires a literal *Tab* character to indent the second line, not four spaces.

With this in the same directory as all the preceding files, in order to build the configuration for the `sparta.naginet` host, we would only need to use a `make` call to generate the file:

```
$ make sparta.cfg
m4 sparta.host.m4 > sparta.cfg
$ make athens.cfg
m4 athens.host.m4 > athens.cfg

```

Note that it's better practice to avoid repeating directives, and instead to use hostgroups and host and service templates to define "roles" for new hosts. This makes adding and removing the hosts much easier, and both processes are explained in this chapter, in the *Configuring host roles using groups* and *Using inheritance to simplify configuration* recipes.

David Douthitt goes into considerably more depth about the possibilities of using `m4` for Nagios configuration at [http://administratosphere.wordpress.com/2009/02/19/configuring-nagios-with-m4/](http://administratosphere.wordpress.com/2009/02/19/configuring-nagios-with-m4/).

## See also

*   The *Configuring host roles using groups* and *Using inheritance to simplify configuration* recipes in this chapter