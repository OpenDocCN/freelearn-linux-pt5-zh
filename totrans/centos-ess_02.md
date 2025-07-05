# Chapter 2. Getting Started with etcd

In this chapter, we will cover `etcd`, CoreOS's central hub of services, which provides a reliable way of storing shared data across cluster machines and monitoring it.

For testing, we will use our already installed CoreOS VM from the previous chapter. In this chapter, we will cover the following topics:

*   Introducing `etcd`
*   Reading and writing to `etcd` from the host machine
*   Reading and writing from an application container
*   Watching changes in `etcd`
*   TTL (Time to Live) examples
*   Use cases of `etcd`

# Introducing etcd

The `etcd` function is an open source distributed key value store on a computer network where information is stored on more than one node and data is replicated using the Raft consensus algorithm. The `etcd` function is used to store the CoreOS cluster service discovery and the shared configuration.

The configuration is stored in the write-ahead log and includes the cluster member ID, cluster ID and cluster configuration, and is accessible by all cluster members.

The `etcd` function runs on each cluster's central services role machine, and gracefully handles master election during network partitions and in the event of a loss of the current master.

# Reading and writing to etcd from the host machine

You are going to learn how read and write to `ectd` from the host machine. We will use both the `etcdctl` and `curl` examples here.

## Logging in to the host

To log in to CoreOS VM, follow these steps:

1.  Boot the CoreOS VM installed in the first chapter. In your terminal, type this:

    ```
    $ cdcoreos-vagrant
    $ vagrant up

    ```

2.  We need to log in to the host via `ssh`:

    ```
    $ vagrant ssh

    ```

## Reading and writing to ectd

Let's read and write to `etcd` using `etcdctl`. So, perform these steps:

1.  Set a `message1` key with `etcdctl` with `Book1` as the value:

    ```
    $ etcdctl set /message1 Book1
    Book1 (we got respond for our successful write to etcd)

    ```

2.  Now, let's read the key value to double-check whether everything is fine there:

    ```
    $ etcdctl get /message1
    Book1
    Perfect!

    ```

3.  Next, let's try to do the same using `curl` via an HTTP-based API. The `curl` function is handy for accessing `etcd` from any place from where you have access to an `etcd` cluster but don't want/need to use the `etcdctl` client:

    ```
    $ curl -L -X PUT http://127.0.0.1:2379/v2/keys/message2 -d value="Book2"
    {"action":"set","key":"/message2","prevValue":"Book1","value":"Book2","index":13371}

    ```

    Let's read it:

    ```
    $ curl -L http://127.0.0.1:2379/v2/keys/message2
    {"action":"get","node":{"key":"/message2","value":"Book2","modifiedIndex":13371,"createdIndex":13371}}

    ```

    Using the HTTP-based `etcd` API means that `etcd` can be read from and written to by client applications without the need to interact with the command line.

4.  Now, if we want to delete the key-value pair, we type the following command:

    ```
    $ etcdctl rm /message1
    $ curl -L -X DELETE http://127.0.0.1:2379/v2/keys/message2

    ```

5.  Also, we can add a key value pair to a directory, as directories are created automatically when a key is placed inside. We only need one command to put a key inside a directory:

    ```
    $ etcdctl set /foo-directory/foo-key somekey

    ```

6.  Let's now check the directory's content:

    ```
    $ etcdctl ls /foo-directory –recursive
    /foo-directory/foo-key

    ```

7.  Finally, we get the key value from the directory by typing:

    ```
    $ etcdctl get /foo-directory/foo-key
    somekey

    ```

# Reading and writing from the application container

Usually, application containers (this is a general term for `docker`, `rkt`, and other types of containers) do not have `etcdctl` or even `curl` installed by default. Installing `curl` is much easier than installing `etcdctl`.

For our example, we will use the Alpine Linux docker image, which is very small in size and will not take much time to pull from the `docker` registry:

1.  Firstly, we need to check the `docker0` interface IP, which we will use with `curl`:

    ```
    $ echo"$(ifconfig docker0 | awk'/\<inet\>/ { print $2}'):2379"
    10.1.42.1:2379

    ```

2.  Let's run the `docker` container with a `bash` shell:

    ```
    $ docker run -it alpine ash

    ```

    We should see something like this in Command Prompt:`/ #`.

3.  As `curl` is not installed by default on Alpine Linux, we need to install it:

    ```
    $ apk update&&apk add curl
    $ curl -L http://10.1.42.1:2379/v2/keys/
    {"action":"get","node":{"key":"/","dir":true,"nodes":[{"key":"/coreos.com","dir":true,"modifiedIndex":3,"createdIndex":3}]}}

    ```

4.  Repeat steps 3 and 4 from the previous subtopic so that you understand that no matter where you are connecting to `etcd` from, `curl` still works in the same way.
5.  Press *Ctrl* +*D* to exit from the `docker` container.

# Watching changes in etcd

This time, let's watch the key changes in `etcd`. Watching key changes is useful when we have, for example, one `fleet` unit with `nginx` writing its port to `etcd`, and another reverse proxy application watching for changes and updating its configuration:

1.  We need to create a directory in `etcd` first:

    ```
    $ etcdctlmkdir /foo-data

    ```

2.  Next, we watch for changes in this directory:

    ```
    $ etcdctl watch /foo-data--recursive

    ```

3.  Now open another CoreOS shell in a new terminal window:

    ```
    $ cdcoreos-vagrant
    $ vagrantssh

    ```

4.  We add a new key to the `/foo-data` directory:

    ```
    $ etcdctl set /foo-data/Book is_cool

    ```

5.  In the first terminal, we should see a notification saying that the key was changed:

    ```
    is_cool

    ```

# TTL (time to live) examples

Sometimes, it is handy to put a **time to live** (**TTL**) for a key to expire in a certain amount of time. This is useful, for example, in the case of watching a key with a 60 second TTL, from a reverse proxy. So, if the `nginx fleet` service has not updated the key, it will expire in 60 seconds and will be removed from `etcd`. Then the reverse proxy checks for it and does not find it. Hence, it will remove the `nginx` service from `config`.

Let's set a TTL of 30 seconds in this example:

1.  Type this in a terminal:

    ```
    $ etcdctl set /foo "I'm Expiring in 30 sec" --ttl 30
    I'm Expiring in 30 sec

    ```

2.  Verify that the key is still there:

    ```
    $ etcdctl get /foo
    I'm Expiring in 30 sec

    ```

3.  Check again after 30 seconds :

    ```
    $ etcdctl get /foo

    ```

4.  If your requested key has already expired, you will be returned `Error`: `100`:

    ```
    Error: 100: Key not found (/foo) [17053]

    ```

This time the key got deleted by `etcd` because we put a TTL of 30 seconds for it.

### Note

TTL is very handy to use to communicate between the different services using `etcd` as the checking point.

# Use cases of etcd

Application containers running on worker nodes with `etcd` in proxy mode can read and write to an `etcd` cluster.

Very common `etcd` use cases are as follows: storing database connection settings, cache settings, and shared settings. For example, the Vulcand proxy server ([http://vulcanproxy.com/](http://vulcanproxy.com/)) uses `etcd` to store web host connection details, and it becomes available for all cluster-connected worker machines. Another example could be to store a database password for MySQL and retrieve it when running an application container.

We will cover more details about cluster setup, central services, and worker role machines in the upcoming chapters.

# Summary

In this short chapter, we covered the basics of `etcd` and how to read and write to `etcd`, watch for changes in `etcd`, and use TTL for `etcd` keys.

In the next chapter, you will learn how to use the `systemd` and `fleet` units.