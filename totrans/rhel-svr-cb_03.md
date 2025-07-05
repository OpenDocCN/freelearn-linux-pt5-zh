# Chapter 3. Configuring Your Network

The recipes we'll be covering in this chapter are as follows:

*   Creating a VLAN interface
*   Creating a teamed interface
*   Creating a bridge
*   Configuring IPv4 settings
*   Configuring your DNS resolvers
*   Configuring static network routes

# Introduction

This chapter will attempt to explain how to use `NetworkManager`, which is the default network configuration tool and daemon in RHEL 7\. It is a set of tools that makes networking simple and straightforward.

Configuring your network can be hard at times, especially when using the more exotic configuration options in combination with well-known configuration scripts. The `NetworkManager` allows you to easily configure your network without needing to edit the configuration files manually.

### Tip

You can still edit the network configuration files located in `/etc/sysconfig/network-scripts` using your preferred editor; however, by default, `NetworkManager` does not notice any changes you make. You'll need to execute the following after editing the files located in the preceding location:

```
~]# nmcli connection reload

```

This is not enough to apply the changes immediately. You'll need to bring down and up the connection or reboot the system.

Alternatively, you can edit `/etc/NetworkManager/NetworkManager.conf` and add `monitor-connection-files=yes` to the `[main]` section. This will cause `NetworkManager` to pick up the changes and apply them immediately.

Within these recipes, you will get an overview on how to configure your network using the `NetworkManager` tools (`nmcli` and `nmtui`) and kickstart files.

# Creating a VLAN interface

VLANs are isolated broadcast domains that run over a single physical network. They allow you to segment a local network and also to "stretch" a LAN over multiple physical locations. Most enterprises implement this on their network switching environment, but in some cases, the tagged VLANs reach your server.

## Getting ready

In order to configure a VLAN, we need an established network connection on the local network interface.

## How to do it…

For the sake of ease, our physical network interface is called `eth0`. The VLAN's ID is 1, and the IPv4 address is `10.0.0.2`, with a subnet mask of `255.0.0.0` and a default gateway of `10.0.0.1`.

### Creating the VLAN connection with nmcli

With `nmcli`, we need to first create the connection and then activate it. Perform the following steps:

1.  Create a VLAN interface using the following command:

    ```
    ~]# nmcli connection add type vlan dev eth0 id 1 ip4 10.0.0.2/8 gw4 10.0.0.1
    Connection 'vlan' (4473572d-26c0-49b8-a1a4-c20b485dad0d) successfully added.
    ~]#

    ```

2.  Now, via this command, activate the connection:

    ```
    ~]# nmcli connection up vlan
    Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
    ~]#

    ```

3.  Check your network connection, as follows:

    ```
    ~]# nmcli connection show
    ~]# nmcli device status
    ~]# nmcli device show eth0.1

    ```

    Here is an example output of the preceding commands:

    ![Creating the VLAN connection with nmcli](img/00015.jpeg)

### Creating the VLAN connection with nmtui

The `nmtui` tool is a text user interface to `NetworkManager` and is launched by executing the following in a terminal:

```
~]# nmtui

```

This will bring up the following text-based interface:

![Creating the VLAN connection with nmtui](img/00016.jpeg)

Navigation is done using the *Tab* and arrow keys, and the selection is done by pressing the *Enter* key. Now, you need to do the following:

1.  Go to **Edit a connection** and select **<OK>**. The following screen will appear:![Creating the VLAN connection with nmtui](img/00017.jpeg)
2.  Next, select **<Add>** and the **VLAN** option. Confirm by Selecting **<Create>**:![Creating the VLAN connection with nmtui](img/00018.jpeg)
3.  Enter the requested information in the following form and commit by selecting **<OK>**:![Creating the VLAN connection with nmtui](img/00019.jpeg)

Your new **VLAN** interface will now be listed in the connections list:

![Creating the VLAN connection with nmtui](img/00020.jpeg)

### Creating the VLAN connection with kickstart

Let's explore what you need to add to your `kickstart` script in order to achieve the same result as in the preceding section:

1.  Look for the configuration parameters within your `kickstart` file with the following command:

    ```
    ...
    network --device=eth0
    ...
    ```

2.  Replace it with the following configuration parameters:

    ```
    network --device=eth0 --vlanid=1 --bootproto=static --ip=10.0.0.2 --netmask=255.0.0.0 --gateway=10.0.0.1
    ```

## There's more…

The command line to create a VLAN with `nmcli` is pretty basic as it uses default values for every piece of information that is missing. To make sure that everything is created to your wishes, it is wise to also use `con-name` and `ifname`. These will respectively name your connection and the device you're creating. Take a look at the following command:

```
~]# nmcli connection add type vlan con-name vlan1 ifname eth0.1 dev eth0 id 1 ip4 10.0.0.2/8 gw4 10.0.0.1

```

This will create the `vlan.1` connection with `eth0` as the parent and `eth0.1` as the target device.

As with `nmcli` and `nmtui`, you can name your VLAN connection in `kickstart`; you only need to specify the `--interfacename` option. If you cannot find any previous network configuration in your `kickstart` file, just add the code to your `kickstart` file.

## See also

The `nmcli` tool lacks a man page, but execute the following command for for more options to create VLAN connections:

```
~]# nmcli con add help

```

For more `kickstart` information on networks, check the following URL: [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html).

# Creating a teamed interface

Interface teaming, interface bonding, and link aggregation are all the same. It was already implemented in the kernel by way of the `bonding` driver. The team driver provides a different mechanism (from bonding) to team multiple network interfaces into a single logical one.

## Getting ready

To set up a teamed interface, we'll need more than one network interface.

## How to do it…

For the sake of ease, our physical network interfaces are called `eth1` and `eth2`. The IPv4 address for the team interface is `10.0.0.2`, with a subnet mask of `255.0.0.0` and a default gateway of `10.0.0.1`.

### Creating the teamed interface using nmcli

Using this approach, we'll need to create the team connection and two team slaves and activate the connection, as follows:

1.  Use the following command line to create the team connection:

    ```
    ~]# nmcli connection add type team ip4 10.0.0.2/8 gw4 10.0.0.1
    Connection 'team' (cfa46865-deb0-49f2-9156-4ca5461971b4) successfully added.
    ~]#

    ```

2.  Add `eth1` to the team by executing the following:

    ```
    ~]# nmcli connection add type team-slave ifname eth1 master team
    Connection 'team-slave-eth1' (01880e55-f9a5-477b-b194-73278ef3dce5) successfully added.
    ~]#

    ```

3.  Now, add `eth2` to the team by running the following command:

    ```
    ~]# nmcli connection add type team-slave ifname eth2 master team
    Connection 'team-slave-eth2' (f9efd19a-905f-4538-939c-3ea7516c3567) successfully added.
    ~]#

    ```

4.  Bring the team up, as follows:

    ```
    ~]# nmcli connection up team
    Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/12)
    ~]#

    ```

5.  Finally, check your network connections through the following commands:

    ```
    ~]# nmcli connection show
    ~]# nmcli device status
    ~]# nmcli device show nm-team

    ```

    Here's an example output of the preceding commands:

    ![Creating the teamed interface using nmcli](img/00021.jpeg)

### Creating the teamed interface using nmtui

Let's fire up `nmtui` and add a connection through the following steps:

1.  First, create a team connection by selecting **<Add>**:![Creating the teamed interface using nmtui](img/00022.jpeg)
2.  Enter the requested information in the following form and click on **<Add>** for every interface to add:![Creating the teamed interface using nmtui](img/00023.jpeg)
3.  Next, select **<Add>** within team slaves to add an interface by filling out the form and selecting **<OK>**. Repeat this for every physical interface:![Creating the teamed interface using nmtui](img/00024.jpeg)
4.  Now, select **<OK>** to create the team interface:![Creating the teamed interface using nmtui](img/00025.jpeg)

    Your new team interface will now be listed in the connections list, as shown in the following screenshot:

    ![Creating the teamed interface using nmtui](img/00026.jpeg)

### Creating the teamed interface with kickstart

Open your `kickstart` file with your favorite editor and perform the following steps:

1.  Look for the network configuration parameters within your `kickstart` file by running the following command:

    ```
    ...
    network --device=eth0
    ...
    ```

2.  Next, add the following configuration parameters:

    ```
    network --device=team0 --teamslaves="eth1,eth2" --bootproto=static --ip=10.0.0.2 --netmask=255.0.0.0 --gateway=10.0.0.1
    ```

## There's more…

Teaming comes with runners—a way of load-sharing backup methods that you can assign to your team:

*   **active-backup**: In this, one physical interface is used, while the others are kept as backup
*   **broadcast**: In this, data is transmitted over all physical interfaces' selectors
*   **LACP**: This implements `802.3ad` Link Aggregation Control Protocol
*   **loadbalance**: This performs active Tx load balancing and uses a BPF-based Tx port
*   **round-robin**: The data is transmitted over all physical interfaces in turn

These can also be defined upon creation using either of the presented options here:

### nmcli

Add `team.config "{\"runner\":{\"name\": \"activebackup\"}}"` to your command to create your team interface, and substitute `activebackup` with the runner that you wish to use.

### nmtui

Fill out the JSON configuration field for the team interface with `{"runner": {"name": "activebackup"}}`, and substitute `activebackup` with the runner that you wish to use.

![nmtui](img/00027.jpeg)

### kickstart

Add `--teamconfig="{\"runner\":{\"name\": \"activebackup\"}}"` to your team device line, and substitute `activebackup` with the runner that you wish to use.

The options provided to create the team interface are bare bones using `nmcli`. If you wish to add a connection and interface name, use `con-name` and `ifname`, respectively, in this way:

```
~]# nmcli connection add type team con-name team0 ifname team0 ip4 10.0.0.2/8 gw4 10.0.0.1
Connection 'team0' (e1856313-ecd4-420e-96d5-c76bc00794aa) successfully added.
~]#

```

The same is true for adding the team slaves, except for `ifname`, which is required to specify the correct interface:

```
~# nmcli connection add type team-slave con-name team0-slave0 ifname eth1 master team0
Connection 'team0-slave0' (3cb2f603-1f73-41a0-b476-7a356d4b6274) successfully added.
~# nmcli connection add type team-slave con-name team0-slave1 ifname eth2 master team0
Connection 'team0-slave1' (074e4dd3-8a3a-4997-b444-a781114c58c9) successfully added.
~#

```

## See also

For more information on the networking team daemon and "runners", refer to the following URL:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Understanding_the_Network_Teaming_Daemon_and_the_Runners.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Understanding_the_Network_Teaming_Daemon_and_the_Runners.html)

For more information on using `nmcli` to create team interfaces, take a look at the following link:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Configure_a_Network_Team_Using-the_Command_Line.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Configure_a_Network_Team_Using-the_Command_Line.html)

For more information on using `nmtui` to create team interfaces, follow this link:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Configure_a_Network_Team_Using_the_Text_User_Interface_nmtui.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Configure_a_Network_Team_Using_the_Text_User_Interface_nmtui.html)

For more information on creating team interfaces in kickstart scripts, the following link will be useful:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html)

# Creating a bridge

A network bridge is a logical device that forwards traffic between connected physical interfaces based on MAC addresses. This kind of bridge can be used to emulate a hardware bridge in virtualization applications, such as KVM, to share the NIC with multiple virtual NICs.

## Getting ready

To bridge two physical networks, we need two network interfaces. Your physical interfaces should never be configured with any address as the bridge will be configured with the IP address(es).

## How to do it…

For the sake of ease, the physical network interfaces we will bridge are `eth1` and `eth2`. The IPv4 address will be `10.0.0.2` with a subnet mask of `255.0.0.0` and a default gateway of `10.0.0.1`.

### Creating a bridge using nmcli

Make sure that you activate the bridge after configuring the bridge and interfaces! Here are the steps that you need to perform for this:

1.  First, create the bridge connection via the following command:

    ```
    ~]# nmcli connection add type bridge ip4 10.0.0.2/8 gw4 10.0.0.1
    Connection 'bridge' (36e40910-cf6a-4a6c-ae28-c0d6fb90954d) successfully added.
    ~]#

    ```

2.  Add `eth1` to the bridge, as follows:

    ```
    ~]# nmcli connection add type bridge-slave ifname eth1 master bridge
    Connection 'bridge-slave-eth1' (6821a067-f25c-46f6-89d4-a318fc4db683) successfully added.
    ~]#

    ```

3.  Next, add `eth2` to the bridge using the following command:

    ```
    ~]# nmcli connection add type bridge-slave ifname eth2 master bridge
    Connection 'bridge-slave-eth2' (f20d0a7b-da03-4338-8060-07a3775772f4) successfully added.
    ~]#

    ```

4.  Activate the bridge by executing the following:

    ```
    ~# nmcli connection up bridge
    Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/30)
    ~]#

    ```

5.  Now, check your network connection by running the following commands:

    ```
    ~]# nmcli connection show
    ~]# nmcli device status
    ~]# nmcli device show bridge

    ```

    Here is an example output of the preceding commands:

    ![Creating a bridge using nmcli](img/00028.jpeg)

### Creating a bridge using nmtui

Launch `nmtui` and select **Edit a connection**. After this, follow these steps to create a bridge using `nmtui`:

1.  Create a bridge connection by selecting **<Add>** and **Bridge** from the connection list and then click on **<Create>**:![Creating a bridge using nmtui](img/00029.jpeg)
2.  Fill out the presented form with the required information:![Creating a bridge using nmtui](img/00030.jpeg)
3.  Next, add the two network interfaces by selecting **<Add>** and providing the requested information for each interface:![Creating a bridge using nmtui](img/00031.jpeg)
4.  Finally, select **<OK>** to create the bridge:![Creating a bridge using nmtui](img/00032.jpeg)

Your new bridge will now be listed in the connections list:

![Creating a bridge using nmtui](img/00033.jpeg)

### Creating a bridge with kickstart

Edit your `kickstart` file with your favorite editor through the following steps:

1.  Look for the configuration parameters within your `kickstart` file using this command line:

    ```
    ...
    network --device=eth0
    ...
    ```

2.  Now, add the following configuration parameters:

    ```
    network --device=bridge0 --bridgeslaves="eth1,eth2" --bootproto=static --ip=10.0.0.2 --netmask=255.0.0.0 --gateway=10.0.0.1
    ```

## There's more…

The options provided to create the bridge are bare bones using `nmcli`. If you wish to add a connection and interface name, use `con-name` and `ifname`, respectively, in this way:

```
~# nmcli connection add type bridge con-name bridge0 ifname bridge0 ip4 10.0.0.2/8 gw4 10.0.0.1
Connection 'bridge0' (d04180be-3e80-4bd4-a0fe-b26d79d71c7d) successfully added.
~#

```

The same is true for adding the bridge slaves, except for `ifname`, which is required to specify the correct interface:

```
~]# nmcli connection add type bridge-slave con-name bridge0-slave0 ifname eth1 master bridge0
Connection 'bridge0-slave0' (3a885ca5-6ffb-42a3-9044-83c6142f1967) successfully added.
~]# nmcli connection add type team-slave con-name team0-slave1 ifname eth2 master team0
Connection 'bridge0-slave1' (f79716f1-7b7f-4462-87d9-6801eee1952f) successfully added.
~]#

```

## See also

For more information on creating network bridges using `nmcli`, go to the following URL:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Network_Bridging_Using_the_NetworkManager_Command_Line_Tool_nmcli.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-Network_Bridging_Using_the_NetworkManager_Command_Line_Tool_nmcli.html)

For more information on creating network bridges using `nmtui`, go to this website:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/ch-Configure_Network_Bridging.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/ch-Configure_Network_Bridging.html)

For more information on kickstart and bridging, go to the following website:

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Installation_Guide/sect-kickstart-syntax.html)

# Configuring IPv4 settings

Changing your IP addresses is pretty straightforward in the old `ifcfg`-style files, and it's actually pretty simple using `NetworkManager` tools as well.

As kickstart is only used to set up a system, it is not relevant to go in depth into this matter in this recipe.

## How to do it…

Let's change our current IPv4 address and gateway for `eth1` to `10.0.0.3`/`8`, with `10.0.0.2` as the default gateway.

### Setting your IPv4 configuration using nmcli

Perform the following steps:

1.  Set the ipv4 information by executing the following command line:

    ```
    ~]# nmcli connection modify eth0 ipv4.addresses 10.0.0.3/8 ipv4.gateway 10.0.0.2

    ```

2.  Now, run the following to verify the information:

    ```
    ~]# nmcli connection show eth0

    ```

    Here is an example output of the preceding commands:

    ![Setting your IPv4 configuration using nmcli](img/00034.jpeg)

### Setting your IPv4 configuration using nmtui

The `nmtui` tool takes a bit more work, but the end result remains the same. Perform the following steps:

1.  Start `nmtui`, select the interface that you wish to modify, and click on **<Edit...>**:![Setting your IPv4 configuration using nmtui](img/00035.jpeg)
2.  Now, modify the IPv4 configuration to your liking and click on **<OK>**.

## There's more…

Managing IPv6 ip addresses is as straightforward as configuring your IPv4 counterparts.

The options you need to use in kickstart to set your ip address and gateway are:

*   `--ip`: This is used to set the system's IPv4 address
*   `--netmask`: This is used for the subnet mask
*   `--gateway`: This is used to set the IPv4 gateway

# Configuring your DNS resolvers

DNS servers are stored in `/etc/resolv.conf`. You can also manage this file using `NetworkManager`.

As with the previous recipe, and for the same reasons, this recipe won't go into the `kickstart` options.

## How to do it…

Let's set the DNS resolvers for `eth1` to point to Google's public DNS servers: `8.8.8.8` and `8.8.4.4`.

### Setting your DNS resolvers using nmcli

Perform the following steps:

1.  Set the DNS servers via the following command:

    ```
    ~]# nmcli connection modify System\ eth1 ipv4.dns "8.8.8.8,8.8.4.4"

    ```

2.  Now, use the following command to check your configuration:

    ```
    ~]# nmcli connection show System\ eth1

    ```

    Here is an example output of the preceding commands:

    ![Setting your DNS resolvers using nmcli](img/00036.jpeg)

### Setting your DNS resolvers using nmtui

The `nmtui` tool requires a bit more work to set the DNS resolvers, as follows:

1.  Start `nmtui`, select the interface that you wish to modify, and click on **<Edit...>**:![Setting your DNS resolvers using nmtui](img/00037.jpeg)

## There's more…

The `nmcli` tool supports adding multiple DNS servers by separating them with a semicolon. Using a blank value (`""`) will remove all the DNS servers for this connection.

Similarly, you can set the DNS search domains for your environment. When using `nmcli`, you'll need to specify the `ipv4.dns-search` property.

Kickstart will allow you to specify the DNS servers using the `--nameserver` option for each DNS server. If you do not wish to specify any DNS servers, use `--nodns`. Unfortunately, there is no native way to set the DNS domain search using `kickstart`. You will have to use `nmcli`, for example, in the `%post` section of your `kickstart` script.

### Tip

Be careful when setting DNS configurations for multiple network interfaces. `NetworkManager` adds all your nameservers to your `resolv.conf` file, but libc may not support more than six nameservers.

# Configuring static network routes

In some cases, it is required to set static routes on your system. As static routes are not natively supported in `kickstart`, this is not covered in this recipe.

## How to do it…

Add static routes to both the `192.168.0.0`/`24` and `192.168.1.0`/`24` networks via `10.0.0.1`.

### Configuring static network routes using nmcli

Here's what you need to do:

1.  Set the route using the following command:

    ```
    ~]# nmcli connection modify eth0 ipv4.routes "192.168.0.0/24 10.0.0.1,192.168.1.0/24 10.0.0.1"

    ```

2.  Now, execute the following command line to verify the configuration:

    ```
    ~]# nmcli connection show eth0

    ```

    Here is an example output of the preceding commands:

    ![Configuring static network routes using nmcli](img/00038.jpeg)

### Configuring network routes using nmtui

Here are the steps for this recipe:

1.  Launch `nmtui`, select the interface that you wish to modify the static routes for, and click on **<Edit...>**:![Configuring network routes using nmtui](img/00039.jpeg)
2.  Now, select **<Edit...>** next to the **IPv4 Configuration – Routing** entry and enter your routes. Select **<OK>** to confirm:![Configuring network routes using nmtui](img/00040.jpeg)
3.  Finally, click on **<OK>** to confirm the changes and save them.