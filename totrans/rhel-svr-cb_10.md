# Chapter 10. Monitoring and Performance Tuning

In this chapter, I'll explore the following topics:

*   Tuning your system's performance
*   Setting up PCP – Performance Co-Pilot
*   Monitoring basic system performance
*   Monitoring CPU performance
*   Monitoring RAM performance
*   Monitoring storage performance
*   Monitoring network performance

# Introduction

Monitoring your infrastructure is an important aspect of your environment as it teaches you much about its behavior. It will tell you where your bottlenecks are and where room for improvement is. In this chapter, we will monitor performance and not create triggers when certain metrics exceed specific values.

# Tuning your system's performance

Companies buy the best hardware their money can get, and they want to use everything optimally. However, it's not just the hardware that makes your applications run faster. Your OS will also behave differently under specific circumstances.

Tuned is a set of tools and a daemon that tunes your system's settings automatically depending on its usage. It periodically collects data from its components through plugins, which it uses to change system settings according to the current usage.

## How to do it…

In this recipe, we'll ask tuned which profile to use and apply it through the following steps:

1.  First, run the following command to install the required packages:

    ```
    ~]# yum install -y tuned

    ```

2.  Enable and start tuned by executing the following commands:

    ```
    ~]# systemctl enable tuned
    ~]# systemctl restart tuned

    ```

3.  Have tuned guess the profile to be used via the following:

    ```
    ~]# tuned-adm recommend
    virtual-guest

    ```

4.  Finally, apply the recommended profile to tuned, as follows:

    ```
    ~]# tuned-adm profile virtual-guest

    ```

## There's more…

You can find the system's tuned profiles used in `/lib/tuned/`. When you create your own, create them in `/etc/tuned` in the same way as they are organized in `/lib/tuned`. I do not recommend creating new profiles in `/etc/tuned` with the same name as in `/lib/tuned`, but if you do, the one in the `/etc/tuned` directory will be used. It is better to create a new one with a different name, including the one you want to modify, and then make the necessary changes in your new profile.

Every profile has a directory, which contains a set of files controlling the behavior of your system. If you explore the `tuned.conf` files in these directories, you will see that these files define the exact settings that other tools (such as **cpufreq**) need to be configured on and that some profiles include other profiles. For instance, if you create a profile for, say, a laptop that is a little better on the battery by applying the `powersave` CPU governor, you could create a new file located at `/etc/tuned/laptop/tuned.conf` containing the following:

```
#
# laptop tuned configuration
#

[main]
include=desktop

[cpu]
replace=1
governor=powersave
```

When you know the bottlenecks of your systems, you can find out how to mitigate them by configuring your system in a specific way. Tuned can come in handy to create and apply profiles based on the performance monitoring of your components.

## See also

For more information about tuning your system, refer to the Red Hat Performance Tuning guide at [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Performance_Tuning_Guide/index.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Performance_Tuning_Guide/index.html).

Check out the man pages of *tuned (8)*, *tuned-adm (8)*, *tuned-main.conf (5)*, and *tuned.conf (5)* for more information.

# Setting up PCP – Performance Co-Pilot

Over the years, a lot of tools have been created to troubleshoot performance issues on your systems, such as `top`, `sar`, `iotop`, `iostat`, `iftop`, `vmstat`, `dstat`, and others. However, none of these integrate with each other, some are extensions to others, and so on.

PCP seems to have a couple of things right: it monitors just about every aspect of your system, it allows the centralized storage of (important) performance data, and it allows you to use not only live data, but also saved data among others.

## How to do it…

In this recipe, we'll look at both the "default" setup and "collector" configuration, which allows you to pull in all the performance data you want.

### The default installation

This is the basic setup of PCP:

1.  Let's install the necessary packages; run the following command:

    ```
    ~]# yum install -y pcp

    ```

2.  Now, enable and start the necessary daemons, as follows:

    ```
    ~]# systemctl enable pmcd
    ~]# systemctl enable pmlogger
    ~]# systemctl start pmcd
    ~]# systemctl start pmlogger

    ```

3.  If you want to have the system monitored by a central collector, execute the following:

    ```
    ~]# firewall-cmd --add-service pmcd --permanent

    ```

### The central collector

Each host that is to act as a collector requires additional configuration. Here's how you can do this:

1.  Add a line per system to collect data from `/etc/pcp/pmlogger/control`, as follows:

    ```
    <hostname> n n PCP_LOG_DIR/pmlogger/<hostname> -r -T24h10m -c config.<hostname>
    ```

    Here, `<hostname>` is the FDQN to this host. Take a look at the following example:

    ```
    guest.example.com n n PCP_LOG_DIR/pmlogger/guest.example.com -r -T24h10m -c config.guest.example.com
    ```

2.  After adding a host in this way, you need to restart the `pmlogger` daemon. Execute the following command line:

    ```
    ~]# systemctl restart pmlogger

    ```

## There's more…

By default, PCP logs information every 60 seconds. If you want to increase this and want to gather performance statistics every 30 seconds, you need to change the line starting with `LOCALHOSTNAME` and add `-t 30s` at the end.

Modifying the statistics you gather is a bit more difficult. You can find the configuration for `pmlogger` in `/var/lib/pcp/config/pmlogconf/`. Every file in this directory contains information about which pointers to gather. The syntax is not very hard to understand, but it is complex to explain. The *pmlogconf (1)* man page contains everything you need to know.

If you want to visualize the data on a host, you need to install `pcp-gui`, as follows:

```
~]# yum install -y pcp-gui dejavu-sans-fonts

```

This package comes with a tool called `pmchart`, which allows you to create graphics with the data collected by PCP. The fonts are needed to properly display the characters.

## See also

For more information about PCP and its components, refer to their online manuals, which you can find at [http://www.pcp.io/documentation.html](http://www.pcp.io/documentation.html).

# Monitoring basic system performance

We need to keep an eye out on global system values. The ones that are particularly of interest are the following:

*   `kernel.all.pswitch`
*   `kernel.all.nprocs`
*   `kernel.all.load`

## How to do it…

I'll show you a way to display both text-based and graphical output. Here are the steps:

1.  Display live data for the metrics with a 1-second interval for the `guest.example.com` host by executing the following command:

    ```
    ~]# pmdumptext -H -t 1 -i -l kernel.all.pswitch kernel.all.nprocs kernel.all.load -h guest.example.com

    ```

    ![How to do it…](img/00070.jpeg)
2.  Create a configuration file for `pmchart` to display live data called `system.conf` with the following contents:

    ```
    #kmchart
    version 1

    chart style plot antialiasing off

    plot color #ffff00 metric kernel.all.pswitch
    chart style plot antialiasing off
     plot color #ffff00 metric kernel.all.nprocs
    chart style plot antialiasing off
     plot color #ffff00 metric kernel.all.load instance "1 minute"
     plot color #ff924a metric kernel.all.load instance "5 minute"
     plot color #ff0000 metric kernel.all.load instance "15 minute"

    ```

3.  Next, use `pmchart` to plot a live chart for `guest.example.com` via the following command:

    ```
    ~]# pmchart -h guest.example.com -c system.conf

    ```

    ![How to do it…](img/00071.jpeg)

## There's more…

The preceding examples are based on "live" data; however, you're not limited to live data. You could increase the interval of `pmlogger` in order to get more data about a troublesome system and then take a look at the generated data afterwards. With other tools, you'd have to use additional tools through cronjob and so on, while PCP allows you to do both.

Here's how you can do this:

1.  Show the the data of `guest.example.com` for November 1, 2015 between `15:30` and `16:30` with a 5-minute interval via the following command:

    ```
    ~]# pmdumptext -H -t 5m -i -l -S @15:30 -T @16:30 kernel.all.pswitch kernel.all.nprocs kernel.all.load -a /var/log/pcp/pmlogger/guest.example.com/20151101

    ```

    ![There's more…](img/00072.jpeg)
2.  You can do the same with `pmchart`, as follows:

    ```
    ~]# pmchart -a /var/log/pcp/pmlogger/guest.example.com/20151101 -c system.conf -S @15:30 -T @16:30 -W -o output.png

    ```

    ![There's more…](img/00073.jpeg)

# Monitoring CPU performance

This recipe will show you how to visualize using `pmchart` and command-line tools to monitor your CPU's performance. We will have a look at the following metrics:

*   `kernel.all.cpu.wait.total`
*   `kernel.all.cpu.irq.hard`
*   `kernel.all.cpu.irq.soft`
*   `kernel.all.cpu.steal`
*   `kernel.all.cpu.sys`
*   `kernel.all.cpu.user`
*   `kernel.all.cpu.nice`
*   `kernel.all.cpu.idle`

## How to do it…

This will show you how to create the text and graphical representation of performance data. Perform the following steps:

1.  Display live data for the preceding metrics with a 1-second interval for the host, `localhost`. Execute the following command:

    ```
    ~]# pmdumptext -H -t 1 -i -l kernel.all.cpu.wait.total kernel.all.cpu.irq.hard kernel.all.cpu.irq.soft kernel.all.cpu.steal kernel.all.cpu.sys kernel.all.cpu.user kernel.all.cpu.nice kernel.all.cpu.idle -h localhost

    ```

    ![How to do it…](img/00074.jpeg)
2.  Create a configuration file for `pmchart` to display live data called `cpu_stack.conf` with the following contents:

    ```
    #kmchart
    version 1

    chart style stacking antialiasing off
     plot color #aaaa7f metric kernel.all.cpu.wait.total

    plot color #008000 metric kernel.all.cpu.irq.hard
     plot color #ee82ee metric kernel.all.cpu.irq.soft
     plot color #666666 metric kernel.all.cpu.steal
     plot color #aa00ff metric kernel.all.cpu.user
     plot color #aaff00 metric kernel.all.cpu.sys
     plot color #aa5500 metric kernel.all.cpu.nice
     plot color #0000ff metric kernel.all.cpu.idle

    ```

    You will notice that I don't use all the metrics in the graph as some of the metrics are combined with one another.

3.  Use `pmchart` to plot a live chart for `guest.example.com`, as follows:

    ```
    ~]# pmchart -h guest.example.com -c cpu_stack.conf

    ```

    ![How to do it…](img/00075.jpeg)

# Monitoring RAM performance

To monitor RAM performance, I am only interested in a couple of metrics, not all the memory-related ones. Take a look at this list:

*   `mem.util.used`
*   `mem.util.free`
*   `mem.util.bufmem`
*   `mem.util.cached`
*   `swap.free`
*   `swap.used`
*   `swap.pagesin`
*   `swap.pagesout`

## How to do it…

This recipe will explain you how to create text-based and graphical outputs:

1.  First, display live data for the preceding metrics through this command:

    ```
    ~]# pmdumptext -H -t 1 -i -l mem.util.used mem.util.free mem.util.bufmem mem.util.cached swap.free swap.used swap.pagesin swap.pagesout -h guest.example.com

    ```

    ![How to do it…](img/00076.jpeg)
2.  Create a configuration file for `pmchart` to display live data called `memory.conf` with the following contents:

    ```
    #kmchart
    version 1

    chart style stacking
     plot color #ffff00 metric mem.util.used
     plot color #ee82ee metric mem.util.free
    chart style stacking
     plot color #ffff00 metric swap.used
     plot color #0000ff metric swap.free
    chart style plot antialiasing off
     plot color #19ff00 metric swap.pagesin
     plot color #ff0004 metric swap.pagesout

    ```

3.  Now, use `pmchart` to plot a live chart for `guest.example.com` by executing the following command line:

    ```
    ~]# pmchart -h guest.example.com -c memory.conf

    ```

    ![How to do it…](img/00077.jpeg)

I haven't included the buffer and cached memory in this graph as it's part of the memory-used metric.

# Monitoring storage performance

In this recipe, we'll look at the following metrics:

*   `disk.all.read`
*   `disk.all.write`
*   `disk.all.read_bytes`
*   `disk.all.write_bytes`

## How to do it…

Let's create a text and graphical representation of the performance data through the following steps:

1.  Display live data for the preceding metrics; you can use the following command for this:

    ```
    ~]# pmdumptext -H -t 1 -i -l disk.all.read disk.all.write disk.all.read_bytes disk.all.write_bytes -h guest.example.com

    ```

    ![How to do it…](img/00078.jpeg)
2.  Next, create a configuration file for `pmchart` to display live data called `disk.conf` with the following contents:

    ```
    #kmchart
    version 1

    chart style stacking
     plot color #ffff00 metric mem.util.used
     plot color #ee82ee metric mem.util.free
    chart style stacking

    plot color #ffff00 metric swap.used
     plot color #0000ff metric swap.free
    chart style plot antialiasing off
     plot color #19ff00 metric swap.pagesin
     plot color #ff0004 metric swap.pagesout

    ```

3.  Now, use `pmchart` to plot a live chart for `guest.example.com`, as follows:

    ```
    ~]# pmchart -h guest.example.com -c memory.conf

    ```

    ![How to do it…](img/00079.jpeg)

# Monitoring network performance

In this recipe, we'll look at the following network metrics:

*   `network.interface.collisions`
*   `network.interface.in.bytes`
*   `network.interface.in.packets`
*   `network.interface.in.errors`
*   `network.interface.in.drops`
*   `network.interface.out.bytes`
*   `network.interface.out.packets`
*   `network.interface.out.errors`
*   `network.interface.out.drops`

## How to do it…

Now, one last time, we'll look at how we can create a text and graphical representation of data. Perform the following steps:

1.  Display live data for the preceding metrics; run the following command:

    ```
    ~]# pmdumptext -H -t 1 -i -l network.interface.collisions network.interface.in.bytes network.interface.in.packets network.interface.in.errors network.interface.in.drops network.interface.out.bytes network.interface.out.packets network.interface.out.errors network.interface.out.drops -h guest.example.com

    ```

    ![How to do it…](img/00080.jpeg)
2.  Create a configuration file for `pmchart` to display live data called `network.conf` with the following contents:

    ```
    #kmchart
    version 1

    chart style plot antialiasing off
     plot color #ff0000 metric network.interface.collisions instance "eth0"
    chart style plot antialiasing off
     plot color #00ff00 metric network.interface.in.bytes instance "eth0"
     plot color #ff0000 metric network.interface.out.bytes instance "eth0"
    chart style plot antialiasing off

    plot color #00ff00 metric network.interface.in.packets instance "eth0"
     plot color #ff0000 metric network.interface.out.packets instance "eth0"
    chart style plot antialiasing off
     plot color #00ff00 metric network.interface.in.errors instance "eth0"
     plot color #ff0000 metric network.interface.out.errors instance "eth0"
    chart style plot antialiasing off
     plot color #00ff00 metric network.interface.in.drops instance "eth0"
     plot color #ff0000 metric network.interface.out.drops instance "eth0"

    ```

3.  Next, use `pmchart` to plot a live chart for `guest.example.com` via this command line:

    ```
    ~]# pmchart -h guest.example.com -c network.conf

    ```

    ![How to do it…](img/00081.jpeg)