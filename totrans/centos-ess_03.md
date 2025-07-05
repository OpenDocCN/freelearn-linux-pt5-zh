# Chapter 3. Getting Started with systemd and fleet

In this chapter, we will cover the basics of systemd and `fleet`, which includes system unit files. We will demonstrate how to use a `fleet` to launch Docker containers.

We will cover the following topics in this chapter:

*   Getting started with `systemd`
*   Getting started with `fleet`

# Getting started with systemd

You are going to learn what `systemd` is about and how to use `systemctl` to control `systemd` units.

## An overview of systemd

The `systemd` is an init system used by CoreOS for starting, stopping, and managing processes.

Basically, it is a system and service manager for CoreOS. On CoreOS, `systemd` will be used almost exclusively to manage the life cycle of Docker containers. The `systemd` records initialization instructions for each process in the unit file, which has many types, but we will mainly be covering the "service" unit file, as covering all of them is beyond the scope for this book.

## The systemd unit files

The `systemd` records initialization instructions/properties for each process in the "service" unit file we want to run. On CoreOS, unit files installed by the user manually or via cloud-init are placed at `/etc/systemd/system`, which is a read-write filesystem, as a large part of CoreOS has only read-only access. Units curated by the CoreOS team are placed in `/usr/lib64/system/system`, and ephemeral units, which exist for the runtime of a single boot, are located at `/run/system/system`. This is really good to know for debugging `fleet` services.

Okay, let's create a unit file to test `systemd`:

1.  Boot your CoreOS VM installed in the first chapter and log in to the host via `ssh`.
2.  Let's create a simple unit file, `hello.service`:

    ```
    $ sudo vi /etc/systemd/system/hello.service

    ```

    Press *I* and copy and paste the following text (or use the provided example file, `hello.service`):

    ```
    [Unit]
    Description=HelloWorld
    # this unit will only start after docker.service
    After=docker.service
    Requires=docker.service

    [Service]
    TimeoutStartSec=0
    # busybox image will be pulled from docker public registry
    ExecStartPre=/usr/bin/docker pull busybox
    # we use rm just in case the container with the name "busybox1" is left 
    ExecStartPre=-/usr/bin/docker rm busybox1
    # we start docker container
    ExecStart=/usr/bin/docker run --rm --name busybox1 busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"
    # we stop docker container when systemctl stop is used
    ExecStop=/usr/bin/docker stop busybox1

    [Install]
    WantedBy=multi-user.target
    ```

3.  Press *Esc* and then type :`wq` to save the file.
4.  To start the new unit, run this command:

    ```
    $ sudo systemctl enable /etc/systemd/system/hello.service

    ```

    Created a symlink from `/etc/systemd/system/multi-user.target.wants/hello.service` to `/etc/systemd/system/hello.service`.

    ```
    $ sudo systemctl start hello.service

    ```

5.  Let's verify that the `hello.service` unit got started:

    ```
    $ journalctl -f -u hello.service

    ```

    `#` You should see the unit's output similar to this:

    ![The systemd unit files](img/image00112.jpeg)

Also, you can check out the list of containers running with `docker ps`.

In the previous steps, we created the `hello.service` system unit, enabled and started it, and checked that unit's log file with `journalctl`.

### Note

To read about more advanced use of the `systemd` unit files, go to [https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd](https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd).

## An overview of systemctl

The `systemctl` is used to control and provide an introspection of the state of the `systemd` system and its units.

It is like your interface to a system (similar to `supervisord`/`supervisordctl` from other Linux distribution), as all processes on a single machine are started and managed by `systemd`, which includes `docker` containers too.

We have already used it in the preceding example to enable and start the `hello.service` unit.

The following are some useful `systemctl` commands, with their purposes:

1.  Checking the status of the unit:

    ```
    $ sudo systemctl status hello.service

    ```

    You should see a similar output as follows:

    ![An overview of systemctl](img/image00113.jpeg)
2.  Stopping the service:

    ```
    $ sudo systemctl stop hello.service

    ```

3.  You might need to kill the service, but that will not stop the `docker` container:

    ```
    $ sudo systemctl kill hello.service
    $ docker ps

    ```

    You should see a similar output as follows:

    ![An overview of systemctl](img/image00114.jpeg)
4.  As you can see, the `docker` container is still running. Hence, we need to stop it with the following command:

    ```
    $ docker stop busybox1

    ```

5.  Restarting the service:

    ```
    $ sudo systemctl restart hello.service

    ```

6.  If you have changed `hello.service`, then before restarting, you need to reload all the service files:

    ```
    $ sudo systemctl daemon-reload

    ```

7.  Disabling the service:

    ```
    $ sudo systemctl disable hello.service

    ```

The `systemd` service units can only run and be controlled on a single machine, and they should better be used for simpler tasks, for example, to download some files on reboot and so on.

You will continue learning about `systemd` in the next topic and in later chapters.

# Getting started with fleet

We use `fleet` to take advantage of `systemd` at the higher level. The `fleet` is a cluster manager that controls `systemd` at the cluster level. You can even use it on a single machine and get all the advantages of `fleet` there too.

It encourages users to write applications as small, ephemeral units that can be easily migrated around a cluster of self-updating CoreOS machines.

## The fleet unit files

The `fleet` unit files are regular `systemd` units combined with specific `fleet` properties.

![The fleet unit files](img/image00115.jpeg)

They are the primary interaction with `fleet`. As in the `systemd` units, the `fleet` units define what you want to do and how `fleet` should do it. The `fleet` will schedule a valid unit file to the single machine or a machine in a cluster, taking in mind the `fleet` special properties from the `[X-Fleet]` section, which replaces the `systemd` unit's `[Install]` section. The rest of `systemd` sections are same in `fleet` units.

Let's overview the specific options of `fleet` for the `[X-Fleet]` section:

*   `MachineID`: This unit will be scheduled on the machine identified by a given string.
*   `MachineOf`: This limits eligible machines to the one that hosts a specific unit.
*   `MachineMetadata`: This limits eligible machines to those hosts with this specific metadata.
*   `Conflicts`: This prevents a unit from being collocated with other units using glob-matching on other unit names.
*   `Global`: Schedule this unit on all machines in the cluster. A unit is considered invalid if options other than MachineMetadata are provided alongside `Global=true`.

An example of how a `fleet` unit file can be written with the [X-Fleet] section is as follows:

```
[Unit]
Description=Ping google

[Service]
ExecStart=/usr/bin/ping google.com

[X-Fleet]
MachineMetadata=role=check
Conflicts=ping.*
```

So, let's see how `Conflicts=ping*` works. For instance, we have two identical `ping.1.service` and `ping.2.service` files, and we run on our cluster using the following code:

```
fleetctl start ping.* 
```

This will schedule two `fleet` units on two separate cluster machines. So, let's convert the `systemd` unit called `hello.service` that we previously used to `fleet` unit.

1.  As usual, you need to log in to the host via `ssh` with `vagrant ssh`.
2.  Now let's create a simple unit file with the new name `hello1.service`:

    ```
    $ sudo vi hello1.service

    ```

    Press *I* and copy and paste the text as follows:

    ```
    [Unit]
    Description=HelloWorld
    # this unit will only start after docker.service
    After=docker.service
    Requires=docker.service

    [Service]
    TimeoutStartSec=0
    # busybox image will be pulled from docker public registry
    ExecStartPre=/usr/bin/docker pull busybox
    # we use rm just in case the container with the name "busybox2" is left 
    ExecStartPre=-/usr/bin/docker rm busybox2
    # we start docker container
    ExecStart=/usr/bin/docker run --rm --name busybox2 busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"
    # we stop docker container when systemctl stop is used
    ExecStop=/usr/bin/docker stop busybox1

    [X-Fleet]
    ```

3.  Press *Esc* and then type :`wq` to save the file.

    As you can see, we have the `[X-Fleet]` section empty for now because we have nothing to use there yet. We will cover that part in more detail in the upcoming chapters.

4.  First, we need to submit our `fleet` unit :

    ```
    $ fleetctl submit hello1.service

    ```

5.  Let's verify that our `fleet` unit files:

    ```
    $ fleetctl list-unit-files

    ```

    ![The fleet unit files](img/image00116.jpeg)
6.  To start the new unit, run this command:

    ```
    $ fleetctl start hello1.service

    ```

    ![The fleet unit files](img/image00117.jpeg)

    The preceding commands have submitted and started `hello1.service`.

    Let's verify that our new `fleet` unit is running:

    ```
    $ fleetctl list-units

    ```

    ![The fleet unit files](img/image00118.jpeg)

Okay, it's now time to overview the `fleetctl` commands.

## An overview of fleetctl

The `fleetctl` commands are very similar to `systemctl` commands— you can see this as follows—and we do not have to use `sudo` with `fleetctl`. Here are some tasks you can perform, listed with the required commands:

1.  Checking the status of the unit:

    ```
    $ fleetctl status hello1.service

    ```

2.  Stopping the service:

    ```
    $ fleetctl stop hello1.service

    ```

3.  Viewing the service file:

    ```
    $ fleetctl cat hello1.service

    ```

4.  If you want to just upload the unit file:

    ```
    $ fleetctl submit hello1.service

    ```

5.  Listing all running fleet units:

    ```
    $ fleetctl list-units

    ```

6.  Listing fleet cluster machines:

    ```
    $ fleetctl list-machines

    ```

    ![An overview of fleetctl](img/image00119.jpeg)

We see just one machine, as in our case, as we have only one machine running there.

Of course, if we want to see the `hello1.service` log output, we still use the same systemd `journalctl` command, as follows:

```
$ journalctl -f -u hello1.service

```

You should see the unit's output similar to this:

![An overview of fleetctl](img/image00120.jpeg)

# References

You can read more about these topics at the given URLs:

*   **systemd unit files**: [https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd/](https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd/)
*   **fleet unit files**: [https://coreos.com/docs/launching-containers/launching/fleet-unit-files/](https://coreos.com/docs/launching-containers/launching/fleet-unit-files/)

# Summary

In this chapter, you learned about CoreOS's `systemd` init system. You also learned how to create and control system and `fleet` service units with `systemctl` and `fleetctl`.

In the next chapter, you will learn how to set up and manage CoreOS clusters.