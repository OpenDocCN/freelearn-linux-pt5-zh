# Chapter 8. Managing Network Layout

In this chapter, we will cover the following recipes:

*   Creating a network host hierarchy
*   Using the network map
*   Choosing icons for hosts
*   Establishing a host dependency
*   Establishing a service dependency
*   Monitoring individual nodes in a cluster
*   Using the network map as an overlay

# Introduction

While Nagios Core is still very useful when configured to monitor only a simple list of hosts and services, it includes some optional directives that allow defining some structural and functional properties of the monitored network; specifically, how the hosts and services interrelate. Describing this structure in the configuration allows some additional intelligent behavior in the monitoring and notification that Nagios Core performs.

There are two main approaches to working with network structure in Nagios Core:

*   Host parent definitions allow an administrator to define a hierarchy of connectivity to monitored hosts from the "point of view" of the Nagios Core server. An example might be a server with the monitored address in another subnet linked to the Nagios Core server by a router. If the router enters a `DOWN` state, it triggers Nagios Core's host reachability logic to automatically determine which hosts become inaccessible, and flags these as `UNREACHABLE` rather than `DOWN`, allowing refined notification behavior.
*   Host and service dependencies allow the formalization of relationships between hosts or services, usually for the purposes of suppressing unnecessary notifications. An example might be a service that tests a login to a mail service, that itself requires a database service to work properly. If Nagios Core finds that the database service and the login service are both down, a service dependency allows the suppressing of the notification about the login service; the administrator would therefore only be notified about the database service being down, which is more likely to be the actual problem.

There is some overlap of functionality here, but the general pattern is that host parent definitions describe the structure of your network from the vantage point of your monitoring server, and host and service dependencies describe the way it functions, independent of the monitoring server. We will define both parent definitions and dependencies in this chapter, with the primary goal of filtering and improving the notifications that Nagios Core sends in response to failed checks, which can assist greatly in diagnosing problems.

We'll also look at another, more subtle benefit of establishing host parent definitions in making the network map of the Nagios Core web interface useful, and once a basic hierarchy is set up, we'll show how to customize the map's appearance (including defining icons for hosts), to make it generally useful as a network weather map.

# Creating a network host hierarchy

In this recipe, we'll learn how to establish a parent-child relationship for two hosts in a very simple network, in order to take advantage of Nagios Core's reachability logic. Changing this configuration is very simple; it involves adding only one directive, and optionally changing some notification options.

## Getting ready

You will need to be running a Nagios Core 3.0 or newer server, and have at least two hosts, one of which is only reachable via the other. The host that allows communications with the other is the **parent host** . You should be reasonably confident that a loss of connectivity to the parent host necessarily implies that the child host becomes unreachable from the monitoring server.

Access to the web interface of Nagios Core would also be useful, as making this change will change the appearance of the network map, discussed in the *Using the network map* recipe in this chapter.

Our example will use a Nagios Core monitoring server, `olympus.naginet`, monitoring three hosts:

*   `calpe.naginet`, a router
*   `janus.naginet`, another router
*   `corsica.naginet`, a web server

The hosts are connected as shown in the following diagram:

![Getting ready](img/5566OS_08_01.jpg)

Note that the Nagios Core server `olympus.naginet` is only able to communicate with the `corsica.naginet` web server if the router `calpe.naginet` is working correctly. If **calpe.naginet** were to enter a **DOWN** state, we would see **corsica.naginet** enter a **DOWN** state too:

![Getting ready](img/5566_08_02.jpg)

This is a little misleading, as we don't actually know whether `corsica.naginet` is down. It might be, but with the router in between the hosts not working correctly, Nagios Core has no way of knowing. A more informative and accurate status for the host would be `UNREACHABLE`; this is what the configuration we're about to add will arrange.

## How to do it...

We can configure a parent-child relationship for our two hosts as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Edit the file containing the definition for the child host. In our example, the child host is `corsica.naginet`, the web server. The host definition might look something similar to the following code snippet:

    ```
    define host {
        use        linux-server
        host_name  corsica.naginet
        alias      corsica
        address    10.128.0.71
    }
    ```

3.  Add a new `parents` directive to the host's definition, and give it the same value as the `host_name` directive of the host on which it is dependent for connectivity. In our example, that host is `calpe.naginet`.

    ```
    define host {
        use        linux-server
        host_name  corsica.naginet
        alias      corsica
        address    10.128.0.71
     parents    calpe.naginet
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, if the parent host enters a **DOWN** state and the child host can't be contacted, then the child host will enter an **UNREACHABLE** state rather than also being flagged as **DOWN**:

![How to do it...](img/5566_08_03.jpg)

The child host's contacts will also receive `UNREACHABLE` notifications instead of `DOWN` notifications for the child host, provided the `u` flag is included in `notification_options` for the host, and `host_notification_options` for the contacts. See the *Specifying which states to be notified about* recipe in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*, for details on this.

## How it works...

This is a simple application of Nagios Core's reachability logic. When the check to `calpe.naginet` fails for the first time, Nagios Core notes that it is a parent host for one child host, `corsica.naginet`. If during checks for the child host it finds it cannot communicate with it, it flags an `UNREACHABLE` state instead of the `DOWN` state, firing a different notification event.

The primary advantages to this are twofold:

*   The `DOWN` notification is only sent for the nearest problem parent host. All other hosts beyond that host fire `UNREACHABLE` notifications. This means that Nagios Core's reachability logic automatically determines the point of failure from its perspective, which can be very handy in diagnosing which host is actually experiencing a problem.
*   If the host is a parent to a large number of other hosts, the configuration can be arranged not to send urgent notifications for `UNREACHABLE` hosts. There may not be much point sending a hundred pager or e-mail messages to an administrator when a very central router goes down; they know there are problems with the downstream hosts, so all we would be doing is distracting them with useless information.

With a little planning and some knowledge of the network, all we need to do is add a few `parents` directives to host definitions to build a simple network structure, and Nagios Core will behave much more intelligently as a result. This is one of the easiest ways to refine the notification behavior of Nagios Core; it can't be recommended enough!

## There's more...

Note that a child host can itself be a parent to other hosts in turn, allowing a nesting network structure. Perhaps in another situation, we might find that the `corsica.naginet` server is two routers away from the monitoring server:

![There's more...](img/5566OS_08_04.jpg)

In this case, not only is `corsica.naginet` the child host of `calpe.naginet`, but `calpe.naginet` is itself the child host of `janus.naginet`. We could specify this relationship in exactly the same way:

```
define host {
    use        linux-router
    host_name  calpe.naginet
    alias      calpe
    address    10.128.0.129
 parents    janus.naginet
}
```

It's also possible to set multiple parents for a host, if there are two possible paths to the same machine:

```
define host {
    use        linux-server
    host_name  corsica.naginet
    alias      corsica
    address    10.128.0.71
 parents    calpe.naginet,janus.naginet
}
```

With this configuration, `corsica.naginet` would only be deemed `UNREACHABLE` if both of its parent hosts were down. This kind of configuration is useful to account for redundant paths in a network; use cases could include spanning tree technologies, or dynamic routing failover.

After you've set up a good basic structure for your network using the `parents` directive, definitely check out the *Using the network map* recipe in this chapter to get some automatic visual feedback about your network's structure as generated from your new configuration.

## See also

*   The *Using the network map* and *Establishing a host dependency* recipes in this chapter
*   The *Specifying which states to be notified about* and *Configuring notification groups* recipes in [Chapter 4](ch04.html "Chapter 4. Configuring Notifications"), *Configuring Notifications*

# Using the network map

In this recipe, we'll examine our network hierarchy in the network map (or status map) in the Nagios Core web interface. The network map takes the form of a generated graphic showing the hierarchy of hosts and their current states. You can learn how to establish such a hierarchy in the recipe *Creating a network host hierarchy* in this chapter. The network map allows filtering to show specific hosts, and clicking on hosts to navigate through larger networks.

## Getting ready

You will need to be running a Nagios Core 3.0 or newer server, and have access to its web interface. You will also need permission to view the states of hosts, preferably all hosts. You can arrange this by adding your username in the `authorized_for_all_hosts` directive, normally in `/usr/local/nagios/etc/cgi.cfg`; for example, for the user `tom`, we might configure the directive to read as follows:

```
authorized_for_all_hosts=nagiosadmin,tom
```

By default, the `nagiosadmin` user should have all the necessary permissions to view the complete map.

The network map is not particularly useful without at least a few hosts configured and arranged in a hierarchy, so if you have not set any `parents` directives for your hosts, then you may wish to read the *Creating a network host hierarchy* recipe in this chapter first, and arrange your monitored hosts as it explains.

## How to do it...

We can inspect the network map for our newly configured host hierarchy like so:

1.  Log in to the Nagios Core web interface.
2.  Click on the **Map** item in the menu on the left:![How to do it...](img/5566_08_05.jpg)

    You should be presented with a generated graphic showing all the hosts in your network that your user has permissions to view:

    ![How to do it...](img/5566_08_06.jpg)
3.  Hover over any host with the mouse to see a panel breaking down the host's current state:![How to do it...](img/5566_08_07.jpg)
4.  By default, the network map is centered around the Nagios Process icon. Try clicking on one of your hosts to recenter the map; in this example, it's recentered on `calpe.naginet`:![How to do it...](img/5566_08_08.jpg)

## How it works...

The network map is automatically generated from your host configuration. By default, it arranges the hosts in sectors, radiating outward from the central Nagios Process icon, using lines to show dependencies, and adjusting background colors to green for `UP` states, and red for `DOWN` or `UNREACHABLE` states.

This map is generated via the **GD2 library** , written by Thomas Boutell. It takes the form of a linked image map. This means that you can simply right-click the image to save it while the network is in a particular state for later reference, and also that individual nodes can be clicked to recenter the map around the nominated host. This is particularly useful for networks with a large number of hosts and very many levels of parent/child host relationships.

## There's more...

Note that the form in the panel in the top-right allows customizing the appearance of the map directly:

![There's more...](img/5566_08_09.jpg)

*   **Layout Method**: This allows you to select the algorithm used to arrange and draw the hosts. It's worth trying each of these to see which you prefer for your particular network layout.
*   **Scaling factor**: Change the value here to reduce or increase the size of the map image; values between `0.0` and `1.0` will reduce the image's size, and values above `1.0` will increase it.
*   **Drawing Layers**: If your hosts are organized into hostgroups, you can filter the map to only display hosts belonging to particular groups.
*   **Layer mode**: If you selected any host groups in the **Drawing Layers** option, this allows you to select whether you want to include hosts in those groups in the map, or exclude them from it.
*   **Suppress popups**: If you find the yellow information popups that appear when hovering over hosts annoying, then you can turn them off by selecting this checkbox.

After selecting or changing any one of these options, you will need to click on **Update** to apply them.

The appearance of the status map can be configured well beyond this by changing directives in the Nagios Core configuration file, and adding some directives to your hosts; take a look at the recipes under the *See also* section of this recipe for some examples of how this is done.

## See also

*   The *Customizing appearance of the network map*, *Choosing icons for hosts*, *Specifying coordinates for a host on the network map*, and *Using the network map as an overlay* recipes in this chapter

# Choosing icons for hosts

In this recipe, we'll learn how to select graphics for hosts, to appear in various parts of the Nagios Core web interface. This is done by adding directives to a host to specify the paths to appropriate images to represent it.

Adding these definitions has no effect on Nagios Core's monitoring behavior; they are mostly cosmetic changes, although it's useful to see at a glance whether a particular node is a server or a workstation, particularly on the network map.

## Getting ready

You will need to be running a Nagios Core 3.0 or newer server, and have access to its web interface. You must also be able to edit the configuration files for the server.

It's a good idea to check that you actually have the required images installed. The default set of icons is included in `/usr/local/nagios/share/images/logos`. Don't confuse this with its parent directory, `images`, which contains images used as part of the Nagios Core web interface itself.

In the `logos` directory, you should find a number of images in various formats. In this example, we're interested in the `router` and `rack-server` icons:

```
$ ls /usr/local/nagios/share/images/logos/{router,rack-server}.*
/usr/local/nagios/share/images/logos/rack-server.gd2
/usr/local/nagios/share/images/logos/rack-server.gif
/usr/local/nagios/share/images/logos/router.gd2
/usr/local/nagios/share/images/logos/router.gif

```

To get the full benefit of the icons, you'll likely want to be familiar with using the network map, and have access to view it with the appropriate hosts in your own Nagios Core instance. The network map is introduced in the *Using the network map* recipe in this chapter.

## How to do it...

We can define images to be used in displaying our host as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Add three new directives to each of the hosts to which you want to apply the icons. In this example, the `rack-server` icon is assigned to `corsica.naginet`, and the `router` icon to both `calpe.naginet` and `corsica.naginet`:

    ```
    define host {
        use              linux-server
        host_name        corsica.naginet
    	alias            corsica
    	address          10.128.0.71
     icon_image       rack-server.gif
     icon_image_alt   Rack Server
     statusmap_image  rack-server.gd2
    }
    define host {
        use              linux-router
        host_name        janus.naginet
        alias            janus
    	address          10.128.0.128
     icon_image       router.gif
     icon_image_alt   Router
     statusmap_image  router.gd2
    }
    define host {
        use              linux-router
        host_name        calpe.naginet
        alias            calpe
    	address          10.128.0.129
     icon_image       router.gif
     icon_image_alt   Router
     statusmap_image  router.gd2
    }
    ```

3.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, a visit to the status map should display the appropriate hosts with icons rather than question marks:

![How to do it...](img/5566_08_10.jpg)

The **Hosts** list should also include a scaled-down version of the image:

![How to do it...](img/5566_08_11.jpg)

## How it works...

When a host list, service list, or network status map is generated, it checks for the presence of `icon_image` or `statusmap_image` values for each host object, reads the appropriate image if defined, and includes that as part of its processing. The network status map defaults to displaying only a question mark in the absence of a value for the `statusmap_image` directive.

Note that for the `statusmap_image` directive, we chose the `.gd2` version of the icon rather than the `.gif` version. This is for performance reasons; the status map is generated with the GD2 library, which deals more efficiently with its native `.gd2` format.

The `icon_image_alt` directive defines the value for the `alt` attribute when the image is displaying in an `<img>` HTML tag. Most web browsers will show the contents of this tag after briefly hovering over the icon.

Nagios Core 3.0 allows you to put these directives in a separate `hostextinfo` object, but this object type is officially deprecated as of Nagios Core 4.0, so it's recommended to avoid it.

## There's more

If you have a number of hosts that need to share the same image, it's a good practice to inherit from a common host template with the appropriate directives set. For our example, we might define a template as follows:

```
define host {
    name             router-icon
    icon_image       router.gif
    icon_image_alt   Router
    statusmap_image  router.gd2
    register         0
}
```

We could then apply the image settings directly to both our routers simply by inheriting from that template, by adding it to the `use` directive:

```
define host {
 use        linux-router,router-icon
    host_name  janus.naginet
    alias      janus
    address    10.128.0.128
}
define host {
 use        linux-router,router-icon
    host_name  calpe.naginet
    alias      calpe
    address    10.128.0.129
}
```

If you don't like the included icon set, there are many icon sets available online on the Nagios Exchange site at [http://exchange.nagios.org/](http://exchange.nagios.org/). If you want, you could even make your own, out of pictures of your physical hardware, saved in standard PNG or GD2 format.

## See also

*   The *Using the network map* and *Specifying coordinates for a host on the network map* recipes in this chapter

# Establishing a host dependency

In this recipe, we'll learn how to establish a host dependency between two hosts. This feature can be used to control how Nagios Core checks hosts and notifies us about problems in situations where if one host is `DOWN`, it implies that at least one other host is necessarily `DOWN`.

## Getting ready

First of all, it's very important to note that this is not quite the same thing as a host being `UNREACHABLE`, which is what the `parents` directive is for, as discussed in the *Creating a network host hierarchy* recipe in this chapter. Most of the time, a host actually being `DOWN` does not mean that other hosts actually go `DOWN` by definition. It's more typical for a child host to simply be `UNREACHABLE`; it might be working fine, but Nagios Core can't check it because of the `DOWN` host in its path.

However, there's one particularly broad category where host dependencies are definitely useful: the host/guest relationship of virtual machines. If you are monitoring both a host physical machine and one or more guest virtual machines, then the virtual machines are definitely dependent on the host; if the host machine is actually in a `DOWN` state and has no redundant failover, then it would imply that the guests were `DOWN` as well, and not simply `UNREACHABLE`.

We'll use **virtualization** as an example, with two virtual machines `zeus.naginet` and `athena.naginet` running on a host machine, `ephesus.naginet`. All three are already monitored, but we'll establish a host dependency so that Nagios Core doesn't notify anyone about the guests' state if it determines that the host is `DOWN`.

You will need a Nagios Core 3.0 or newer server, and have shell access to change its backend configuration.

## How to do it...

We can establish our host dependencies as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Create or edit an appropriate file that will be included by the configuration in `/usr/local/nagios/etc/nagios.cfg`. A sensible choice could be `/usr/local/nagios/etc/objects/dependencies.cfg`:

    ```
    # vi dependencies.cfg

    ```

3.  Add a `hostdependency` definition. In our case, the definition looks similar to the following code snippet. Note that you can include multiple dependent hosts by separating their names with commas:

    ```
    define hostdependency {
        host_name                      ephesus.naginet
        dependent_host_name            zeus.naginet,athena.naginet
        execution_failure_criteria     n
        notification_failure_criteria  d,u
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, if the `ephesus.naginet` host goes down and takes both the `zeus.naginet` and `athena.naginet` hosts down with it, then checks to all three hosts will continue but notifications will be suppressed for the two guest hosts.

## How it works...

The host dependency object's four directives are as follows:

*   `host_name`: This is the name of the host on which at least one other host is dependent. We'll refer to this as the dependency host. This can also be a comma-separated list of host names.
*   `dependent_host_name`: This is the name of the dependent host. Again, this can be a comma-separated list.
*   `execution_failure_criteria`: This defines a list of states for the dependency host. If that host is in any of these states, then Nagios Core will skip checks for the dependent hosts. This can be a comma-separated list of any of the following flags:

    *   `o`: Dependency host is `UP`
    *   `d`: Dependency host is `DOWN`
    *   `u`: Dependency host is `UNREACHABLE`
    *   `p`: Dependency host is `PENDING` (not checked yet)

    Alternatively, the single flag `n` can be used (as it is in this example), to specify that the checks should take place regardless of the dependency host's state.

*   `notification_failure_criteria`: This defines a list of states for the dependency host. If that host is in any of these states, then notifications for the dependent host will not be sent. The flags are the same as for `execution_failure_criteria`; in this example, we've chosen to suppress the notifications if the dependency host is `DOWN` or `UNREACHABLE`.

When Nagios Core notices that the `zeus.naginet` or `athena.naginet` hosts have apparently gone `DOWN` as a result of a failed host check, it refers to its configuration to check if there are any dependencies for the host, and finds that they depend on `ephesus.naginet`.

It then checks the status of `ephesus.naginet` and finds it to be `DOWN`. Referring to the `execution_failure_criteria` directive and finding `n`, it continues to run checks for both of the dependent hosts as `normal`. However, referring to the `notification_failure_criteria` directive and finding `d,u`, it determines that notifications should be suppressed until the host returns to an `UP` state.

## There's more...

We can specify groups rather than host names for dependencies using the `hostgroup_name` and `dependent_hostgroup_name` directives:

```
define hostdependency {
 hostgroup_name                 vm-hosts
 dependent_hostgroup_name       vm-guests
    execution_failure_criteria     n
    notification_failure_criteria  d,u
}
```

We can also provide comma-separated lists of dependency hosts:

```
define hostdependency {
 host_name            ephesus.naginet,alexandria.naginet
    dependent_host_name  zeus.naginet,athena.naginet    
    execution_failure_criteria     n
    notification_failure_criteria  d,u
}
```

If a host depends on more than one host, the check or notification rules apply if any of its dependencies are not met, rather than all of them. For the previous example, this means that if `ephesus.naginet` was `DOWN`, but `alexandria.naginet` was `UP`, then the dependency would still suppress checks or notifications for all dependent hosts.

This means that host dependencies are not really suitable in redundant scenarios where the loss of one of the depended-upon hosts does not necessarily imply the loss of all its dependent hosts. You are likely to find that monitoring nodes as a cluster is a better fit for this situation; this is discussed in the *Monitoring individual nodes as a cluster* recipe, also in this chapter.

## See also

*   The *Establishing a service dependency*, *Creating a network host hierarchy*, and *Monitoring individual nodes as a cluster* recipes in this chapter

# Establishing a service dependency

In this recipe, we'll learn how to establish a service dependency between two services. This feature can be used to control how Nagios Core checks services and notifies us about problems in situations where if one service is in a `PROBLEM` state, it implies that at least one other service is necessarily also in a `PROBLEM` state.

## Getting ready

You will need a Nagios Core 3.0 or newer server, and have shell access to change its backend configuration. You will also need to have at least two services defined, one of which is by definition dependent on the other; this means that if the dependency service were to enter `CRITICAL` state, then it would imply that the dependent service would also be `CRITICAL`.

We'll use a simple example: suppose we are testing authentication to a mail server `marathon.naginet` with a service `MAIL_LOGIN`, and also checking a database service `MAIL_DB` on the same host, which stores the login usernames and password hashes.

In this situation, it might well be the case that if `MAIL_DB` is not working, then `MAIL_LOGIN` will almost certainly not be working either. If so, then we can configure Nagios Core to be aware that the `MAIL_LOGIN` service is dependent on the `MAIL_DB` service.

## How to do it...

We can establish our service dependency as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Create or edit an appropriate file that will be included by the configuration in `/usr/local/nagios/etc/nagios.cfg`. A sensible choice could be `/usr/local/nagios/etc/objects/dependencies.cfg`:

    ```
    # vi dependencies.cfg

    ```

3.  Add a `servicedependency` definition. In our case, the definition looks similar to the following code snippet:

    ```
    define servicedependency {
        host_name                      marathon.naginet
        service_description            MAIL_DB
        dependent_host_name            marathon.naginet
    	dependent_service_description  MAIL_LOGIN
        execution_failure_criteria     c
        notification_failure_criteria  c
    }
    ```

4.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, if the `MAIL_DB` service fails for whatever reason and enters a `CRITICAL` state, the `MAIL_LOGIN` service will skip its checks of that service, and also skip any notifications that it would normally send about its own problems, if any. Note that the web interface may still show Nagios Core is scheduling checks, but it won't actually run them.

## How it works...

The service dependency object's five directives are as follows:

*   `host_name`: This is the name of the host with which these services are associated. We'll refer to this as the dependency host.
*   `service_description`: This is the description of the service being depended upon. It can be a comma-separated list. We'll refer to this as the dependency service.
*   `dependent_service_description`: This is the description of the dependent service. It can also be a comma-separated list.
*   `execution_failure_criteria`: This defines a list of states for the dependency service. If that service is in any of these states, then Nagios Core will skip the checks for the dependent services. It can be a comma-separated list of any of the following flags:
*   `o`: Dependency service is `OK`

    *   `w`: Dependency service is `WARNING`
    *   `c`: Dependency service is `CRITICAL` (as in this example)
    *   `u`: Dependency service is `UNKNOWN`
    *   `p`: Dependency service is `PENDING` (not checked yet)

    Alternatively, the single flag `n` can be used to specify that the checks should take place regardless of the dependency service's state. In this example, we've chosen the value `c` to suppress service checks only if the dependency service is `CRITICAL`.

*   `notification_failure_criteria`: This defines a list of states for the dependency service. If that service is in any of these states, then notifications for the dependent service will not be sent. The flags are the same as for `execution_failure_criteria`; in this example, we've again chosen the value `c` to suppress the notifications only if the dependency service is `CRITICAL`.

When Nagios Core notices the `MAIL_DB` service has gone `CRITICAL` as a result of a failed service check, it refers to its configuration to check if there are any dependencies for the service, and finds that they depend on `MAIL_LOGIN`.

It then checks the status of `MAIL_DB` and finds it to be `CRITICAL`. Referring to the `execution_failure_criteria` directive and finding `c`, it prevents checks for both of the dependent services. Referring to the `notification_failure_criteria` directive and also finding `c`, it also decides that notifications should be suppressed until the service returns to any other state.

## There's more...

Note that services do not have to be on the same host to depend upon one another. We can add `dependent_host_name` or `dependent_hostgroup_name` directives to specify other hosts:

```
define servicedependency {
 host_name                      marathon.naginet
    service_description            MAIL_DB
 dependent_host_name            sparta.naginet
    dependent_service_description  WEBMAIL_LOGIN
    execution_failure_criteria     c
    notification_failure_criteria  c
}
```

In this example, the `WEBMAIL_LOGIN` service on `sparta.naginet` is defined as dependent on the `MAIL_DB` service on `marathon.naginet`. Note that the values for `host_name` and `dependent_host_name` are different.

In versions of Nagios Core before 3.3.1, the `dependent_host_name` directive is required, even if it is the same as the `host_name`.

## See also

*   The *Establishing a host dependency* and *Monitoring individual nodes as a cluster* recipes in this chapter

# Monitoring individual nodes in a cluster

In this recipe, we'll learn how to monitor a collection of hosts in a cluster, using the `check_cluster` plugin included in the standard Nagios Plugins. Being able to monitor more than one host collectively is useful in situations with redundancy; one of a set of hosts being `DOWN`, perhaps for power conservation or maintenance reasons, is not necessarily a cause for notification. However, if a larger number or all of the hosts were down, we would definitely want to be notified. Using `check_cluster` allows us to arrange this.

## Getting ready

You will need a Nagios Core 3.0 or newer server, and have shell access to change its backend configuration. You will also need to have at least two monitored hosts in a redundant setup for some function, such as database replication, DNS servers, or load-balanced web servers.

You should also be familiar with the way hosts and services are defined, and in particular defining commands; these concepts are discussed in [Chapter 1](ch01.html "Chapter 1. Understanding Hosts, Services, and Contacts"), *Understanding Hosts, Services, and Contacts*.

For this example, we'll work with three blade servers with hostnames `achilles.naginet`, `odysseus.naginet`, and `agamemnon.naginet`, running in a redundant cluster to support a virtual hosting environment. All three hosts are already being monitored to send an e-mail message if one of them goes down. We will arrange a `check_cluster` service on a "dummy host" in such a way that:

*   If none of the blades is down, the service is `OK`
*   If one of the blades is down, the service enters `WARNING` state, again notifying us as appropriate
*   If two or all three of the blades are down, the service enters `CRITICAL` state, again notifying us as appropriate

## How to do it...

We can arrange a cluster check for our hosts as follows:

1.  Change to the `objects` configuration directory for Nagios Core. The default path is `/usr/local/nagios/etc/objects`. If you've put the definition for your host in a different file, then move to its directory instead.

    ```
    # cd /usr/local/nagios/etc/objects

    ```

2.  Create or edit an appropriate file for defining a new command. A sensible choice might be `/usr/local/nagios/etc/objects/commands.cfg`.
3.  Define two new commands in this file, `check_dummy` and `check_host_cluster`:

    ```
    define command {
        command_name  check_dummy
        command_line  $USER1$/check_dummy $ARG1$ $ARG2$
    }
    define command {
        command_name  check_host_cluster
        command_line  $USER1$/check_cluster -h -d $ARG1$ -w $ARG2$ -c $ARG3$
    }
    ```

4.  Create or edit an appropriate file that will be included by the configuration in `/usr/local/nagios/etc/nagios.cfg`. A sensible choice might be `/usr/local/nagios/etc/objects/clusters.cfg`.
5.  Define a dummy host for the cluster with the following values:

    ```
    define host {
        use                 generic-host
        host_name           naginet-blade-cluster
        alias               Naginet Blade Cluster
        address             127.0.0.1
        max_check_attempts  1
        contact_groups      admins
        check_command       check_dummy!0!"Dummy host only"
    }
    ```

    Note that the `address` directive has the value `127.0.0.1`; this is deliberate, as the dummy host itself does not really need to be actively checked or send any notifications.

6.  Add a service for the dummy host:

    ```
    define service {
        use                   generic-service
        host_name             naginet-blade-cluster
        service_description   CLUSTER
     check_command         check_host_cluster!$HOSTSTATEID:achilles.naginet$,$HOSTSTATEID:odysseus.naginet$,$HOSTSTATEID:agamemnon.naginet$!@1:!@2:
        notification_options  c,w,r
    }
    ```

    The suggestion of inheriting from `generic-service` is only an example; you will likely want to use your own template or values. Note that the value for `check_command` is all on one line, with no spaces. You should substitute the hostnames of your own machines.

7.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, the `CLUSTER` service on the `naginet-blade-cluster` dummy host should be available for viewing. It will change state and send notifications the same way as any other service if the hosts in the cluster come up or go down:

![How to do it...](img/5566_08_12.jpg)

## How it works...

The host added in this recipe is just a "hook" for the `CLUSTER` service, which performs the actual check. This is why we used a command using the `check_dummy` plugin to always return an `OK` state:

```
# /usr/local/nagios/libexec/check_dummy 0 "Dummy host only"
OK: Dummy host only

```

The `check_cluster` command is actually quite simple. It performs no actual checks of its own. Instead, it determines states based on the current state of other hosts or services.

This is why the `$HOSTSTATEID:hostname$` macros are used in the `check_command` directive for the service; they evaluate to a number indicating the state of the host, with the hostname specified after the colon.

For example, if `achilles.naginet` and `odysseus.naginet` were `UP`, but `agamemnon.naginet` was `DOWN`, then the command run by Nagios Core for the check would look similar to the following code snippet after the macros were expanded:

```
/usr/local/nagios/libexec/check_cluster -h -d 0,0,1 -w @1: -c @2:

```

We can run this ourselves as the `nagios` user to check the output:

```
# sudo -s -u nagios
$ /usr/local/nagios/libexec/check_cluster -h -d 0,0,1 -w @1: -c @2:
CLUSTER WARNING: Service cluster: 2 ok, 1 warning, 0 unknown, 0 critical

```

The comma-separated states given in the `-d` option to the plugin correspond to two hosts being `UP` (state ID of `0`), and one host being `DOWN` (state ID of `1`). The `-w` option's value of `@1:` means that a `WARNING` state will be entered if one or more of the hosts is down. Similarly, the `-c` option's value of `@2:` means that a `CRITICAL` state will be entered if two or more of the hosts go down.

This allows you to customize notifications to be sent based on the number of hosts in the `DOWN` state, rather than merely monitoring the hosts individually. Once you're confident this is working correctly, you may even choose to prevent the individual hosts from sending notifications to your pager, and have the `CLUSTER` service notify you instead.

## There's more...

If you have a cluster of services rather than hosts to monitor, this can be done by using the `-s` option to `check_cluster`, rather than the `-h` option. In this case, instead of the `$HOSTSTATEID:<host_name>$` macro, you would use the `$SERVICESTATEID:<host_name>:<service_description>$` macro. An example configuration might look similar to the following code snippet for a cluster of two web servers, `sparta.naginet` and `athens.naginet`, given a dummy host named `naginet-http-cluster`:

```
define command {
    command_name  check_service_cluster
 command_line  $USER1$/check_cluster -s -d $ARG1$ -w $ARG2$ -c $ARG3$
} 
define service {
    use                  generic-service
	host_name            naginet-http-cluster
	service_description  CLUSTER_HTTP
 check_command       check_service_cluster!$SERVICESTATEID:sparta.naginet:HTTP$,$SERVICESTATEID:athens.naginet:HTTP$!@1:!@2:
}
```

## See also

*   The *Establishing a host dependency* recipe in this chapter

# Using the network map as an overlay

In this recipe, we'll learn how to use a background for the network map and deliberate placement of hosts in specific points on it, to make a kind of network status weather map to see host statuses at a glance in a geographical context.

## Getting ready

You will need a Nagios Core 3.0 or newer server, and have shell access to change its backend configuration. You should also have at least a couple of hosts configured to place on the map, and understand the basics of using the Nagios network map and icons for hosts. These are discussed in the *Using the network map* and *Choosing icons for hosts* recipes, in this chapter.

You should also select a background image on which you can meaningfully place hosts. If you are monitoring an office network, this could be a floor plan of the building or server room. If you're monitoring a nationwide Internet service provider, then you could use a map of your state or country. Some administrators even like to use pictures of physical equipment, and place the Nagios Core hosts over their physical analogues. In this example, we'll use a map of Australia, 640 by 509 pixels in size, a public domain image retrieved from the Natural Earth website at [http://www.naturalearthdata.com/](http://www.naturalearthdata.com/):

![Getting ready](img/5566_08_13.jpg)

The background image can be anything you like, and several graphics formats including PNG can be used. However, for the sake of quick map rendering, it's recommended to use an image in the GD2 file format, with extension .`gd2`. If you have your image in a PNG format, you can generate a GD2 image from it using the free tool `pngtogd2`:

```
$ pngtogd2 australia.png australia.gd2 0 1

```

This tool is available on Debian-derived systems in the `libgd-tools` package. Its source code is also available online at [http://www.libgd.org/](http://www.libgd.org/).

## How to do it...

We can set up a background for our network map as follows:

1.  Copy your GD2 format image into the `images` subdirectory of the `physical_html_path` directory. In the default installation, this is `/usr/local/nagios/share/images`; if it is different, you can find the definition of `physical_html_path` in `/usr/local/nagios/etc/cgi.cfg`.

    ```
    # cp /home/tom/australia.gd2 /usr/local/nagios/share/images

    ```

2.  Change to the configuration directory for Nagios Core. In the default installation, this is `/usr/local/nagios/etc`. Edit the file `cgi.cfg`.

    ```
    # cd /usr/local/nagios/etc
    # vi cgi.cfg

    ```

3.  Look for the directive `statusmap_background_image` in this file. Uncomment it and make its value the name of your image:

    ```
    statusmap_background_image=australia.gd2
    ```

4.  Look for the directive `default_statusmap_layout` in the same file. Change it to `0`, which corresponds to the **User-defined coordinates** layout.

    ```
    default_statusmap_layout=0
    ```

5.  Change to the `objects` configuration directory for Nagios Core. In the default installation, this is `/usr/local/nagios/etc/objects`. Add `2d_coords` directives to each of the hosts you want to display on the map. You might like to include definitions for `statusmap_image` here too, which is done as follows:

    ```
    # vi australia.cfg
    define host {
        use              linux-server
        host_name        adelaide.naginet
    	address          10.128.0.140
        2d_coords        390,360
     statusmap_image  rack-server.gd2
    }
    define host {
        use              linux-server
        host_name        cairns.naginet
        address          10.128.0.141
        2d_coords        495,100
     statusmap_image  rack-server.gd2
    }
    ... etc ...
    ```

6.  For the directive `2d_coords`, supply two comma-separated values describing the coordinates for the placement of the host. For example, `adelaide.naginet` is 390 pixels from the left, and 360 pixels from the top. A convenient way to get the coordinates is by using GIMP, the open source imaging tool, or even a simple tool such as MS Paint; load the image and hover over the point you wish to use to find its pixel coordinates.
7.  Validate the configuration and restart the Nagios Core server:

    ```
    # /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
    # /etc/init.d/nagios restart

    ```

With this done, on visiting the network map in the Nagios Core web interface by clicking on **Map** on the left-hand side menu, your hosts will be placed in their corresponding positions on the map, including the normal lines and colors to specify child-parent relationships and reachability:

![How to do it...](img/5566_08_14.jpg)

## How it works...

The `default_statusmap_layout` directive fixes the network map into the **User-supplied coords** mode by default. In this mode, only hosts with values for `2d_coords` are shown, and they are displayed at fixed points on the map, rather than being dynamically placed.

It's possible to use this display mode without a background if we wish, but we can give a lot of useful context to the picture of the network generated by taking the extra step of using an actual background image.

Note that if you don't have any hosts with coordinates defined, you'll receive an error that looks similar to the following screenshot:

![How it works...](img/5566_08_15.jpg)

## There's more...

Using the network map with an image background can be particularly helpful for seeing not only the statuses of individual hosts at a glance, but in the case of outages from multiple hosts, looking for possible geographical causes. If all of the nodes in one part of the city or country went down at once, we would be able to see that at a glance. This makes the network map an excellent choice for a network monitoring display, or as one of the first ports of call in diagnosing large-scale problems.

The network map is very useful in this way, and graphically it is probably the most impressive part of the Nagios Core web interface. If you would like even more options and an impressive range of visualizations for host statuses, you may like to consider looking at the excellent **NagVis** extension, which could fill a whole book in itself. There is a brief introduction to its usage in the *Getting extra visualizations with NagVis* recipe, in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*.

## See also

*   The *Creating a network host hierarchy*, *Using the network map*, and *Choosing icons for hosts* recipes in this chapter
*   The *Getting extra visualizations with NagVis* recipe in [Chapter 11](ch11.html "Chapter 11. Automating and Extending Nagios Core"), *Automating and Extending Nagios*