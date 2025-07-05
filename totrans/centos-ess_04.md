# Chapter 4. Managing Clusters

In this chapter, we will cover how to setup and manage a local CoreOS cluster on a personal computer. You will learn how to bootstrap a three-peer cluster, customize it via the `cloud-config` file, and schedule a fleet unit in the cluster.

In this chapter, we will cover the following topics:

*   Bootstrapping a local cluster
*   Customizing a cluster via the`cloud-config` file
*   Scheduling a `fleet` unit in the cluster

You are going to learn how to setup a simple three-node cluster on your personal computer.

# Determining the optimal etcd cluster size

The most efficient cluster size is between three and nine peers. For larger clusters, `etcd` will select a subset of instances to participate in order to keep it efficient.

The bigger the cluster, the slower the writing to the cluster becomes, as all of the data needs to be replicated around the cluster peers. To have a cluster well-optimized, it needs to be based on an odd number of peers. It must have a quorum of at least of three peers and prevent a split-brain in the event of network partition.

In our case, we are going to set up a three-peer `etcd` cluster. To build a highly available cluster on the cloud (GCE, AWS, Azure, and so on), you should use multiple availability zones in order to decrease the effect of failure in a single domain.

In a general cluster, peers are not recommended to be used for anything except for running an `etcd` cluster. But for testing our cluster setup, it will be fine to deploy some `fleet` units there.

In later chapters, you will learn how to properly set up clusters to be used for production.

# Bootstrapping a local cluster

As discussed earlier, we will be installing a three-peer `etcd` cluster on our computer.

## Cloning the coreos-vagrant project

Let's clone the project and get it running. Follow these steps:

1.  In your terminal or command prompt, type this:

    ```
    $ mkdir cluster 
    $ cd cluster
    $ git clone https://github.com/coreos/coreos-vagrant.git
    $ cd coreos-vagrant
    $ cpconfig.rb.sampleconfig.rb
    $ cp user-data.sample user-data

    ```

2.  Now we need to adjust some settings. Edit `config.rb` and change the file's top part to this example:

    ```
    # Size of the CoreOS cluster created by Vagrant
    $num_instances=3

    # Used to fetch a new discovery token for a cluster of size $num_instances
    $new_discovery_url="https://discovery.etcd.io/new?size=#{$num_instances}"

    # To automatically replace the discovery token on 'vagrant up', uncomment
    # the lines below:
    #
    if File.exists?('user-data') &&ARGV[0].eql?('up')
      require 'open-uri'
      require 'yaml'

      token = open($new_discovery_url).read

      data = YAML.load(IO.readlines('user-data')[1..-1].join)
      if data['coreos'].key? 'etcd'
        data['coreos']['etcd']['discovery'] = token
      end
      if data['coreos'].key? 'etcd2'
        data['coreos']['etcd2']['discovery'] = token
      end

    yaml = YAML.dump(data)
    File.open('user-data', 'w') { |file| file.write("#cloud-config\n\n#{yaml}") }
    end
    #
    ```

    ### Note

    Alternatively, you can use the example code of this chapter, which will be kept up to date with changes in the coreos-vagrant GitHub repository.

    What we did here is as follows:

    *   We set the cluster to three instances
    *   Discovery token is automatically replaced on each `vagrant up` command

3.  Next, we need to edit the user data file:

    Change the `"#discovery: https://discovery.etcd.io/<token>"` line to this:

    ```
    "discovery: https://discovery.etcd.io/<token>"
    ```

    So, when we boot our vagrant-based cluster the next time, we will have three CoreOS `etcd` peers running and connected to the same cluster via the discovery token provided through "`https://discovery.etcd.io/<token>`".

4.  Let's now fire up our new cluster using the following command:

    ```
    $ vagrant up

    ```

    We should see something like this in our terminal:

    ![Cloning the coreos-vagrant project](img/image00121.jpeg)

    Hold on! There's more output!

    ![Cloning the coreos-vagrant project](img/image00122.jpeg)

    The cluster should be up and running now.

5.  To check the status of the cluster, type the following command:

    ```
    $ vagrant status

    ```

    You should see something like what is shown in the following screenshot:

    ![Cloning the coreos-vagrant project](img/image00123.jpeg)

Now it's time to test our new CoreOS cluster. We need to run `ssh` for one of our peers and check the `fleet` machines. This can be done by the following command:

```
$ vagrant ssh core-01 -- -A
$ fleetctl list-machines

```

We should see something like what is shown in the following screenshot:

![Cloning the coreos-vagrant project](img/image00124.jpeg)

Excellent! We have got our first CoreOS cluster set, as we see all the three machines up and running. Now, let's try to set a key in `etcd` with which we can check on another machine later on. Type in the following command:

```
$ etcdctl set etcd-cluster-key "Hello CoreOS"

```

You will see the following output:

```
Hello CoreOS

```

Press *Ctrl*+*D* to exit and type the following command to get to VM host's console:

```
$ vagrant ssh core-02 -- -A

```

Let's verify that we can see our new `etcd` key there too:

```
$ etcdctl get etcd-cluster-key
Hello CoreOS

```

Brilliant! Our `etcd` cluster is working just fine.

Exit from the `core-02` machine by pressing *Ctrl*+*D*.

## Customizing a cluster via the cloud-config file

Let's make some changes to the `cloud-config` file and push it into the cluster machines:

1.  In the user data file (`cloud-config` file for Vagrant-based CoreOS), below the text block `fleet` make changes:

    ```
    public-ip: $public_ipv4
    ```

    Add a new line:

    ```
    metadata: cluster=vagrant
    ```

    So, it will look like this:

    ```
    fleet:
          public-ip: $public_ipv4
          metadata: cluster=vagrant
    ```

2.  Let's add a `test.txt` file to the `/home/core` folder via `cloud-config` too. At the end of the user data file, add this code:

    ```
    write_files:
      - path: /home/core/test.txt
        permissions: 0644
        owner: core
        content: |
    Hello Cluster
    ```

    This will add a new file in the`/home/core` folder on each cluster machine.

3.  To get our changes implemented which we did previously, run the following commands:

    ```
    $ vagrant provision

    ```

    You will see the following result:

    ![Customizing a cluster via the cloud-config file](img/image00125.jpeg)

    Then, run this command:

    ```
    $ vagrant reload

    ```

    The first command provisionally updated user data file on all three VMs, and the second reloaded them.

4.  To `ssh` to one of the VMs, enter this code:

    ```
    $ vagrant ssh core-03 -- -A
    $ ls
    test.txt

    ```

5.  To check the content of the `test.txt` file, use this line:

    ```
    $ cat test.txt

    ```

    You should see output as follows:

    ```
    Hello Cluster

    ```

As you can see, we have added some files to all cluster machines via the `cloud-config` file.

Let's check one more change that we have done in that file using the following command:

```
$ fleetctl list-machines

```

You will see something like this:

![Customizing a cluster via the cloud-config file](img/image00126.jpeg)

Thus, you can see that we have some metadata assigned to cluster machines via the `cloud-init` file.

## Scheduling a fleet unit in the cluster

Now, for the fun part, we will schedule a `fleet` unit in the cluster.

1.  Let's log in to the `core-03` machine:

    ```
    $ vagrant ssh core-03 -- -A

    ```

2.  Create a new `fleet` unit called `hello-cluster.service` by copying and pasting this line:

    ```
    $ vi hello-cluster.service
    [Unit]
    [Service]
    ExecStart=/usr/bin/bash -c "while true; do echo 'Hello Cluster'; sleep 1; done"

    ```

3.  Let's schedule the `hello-cluster.service` job for the cluster:

    ```
    $ fleetctl start hello-cluster.service

    ```

    You should see output as follows:

    ```
    Unit hello-cluster.service launched on bb53c039.../172.17.8.103

    ```

    We can see that `hello-cluster.service` was scheduled to be run on the `172.17.8.103` machine because that machine first responded to the `fleetctl` command.

    In later chapters, you will learn how to specifically schedule jobs to a particular machine. Now let's check out the real-time `hello-cluster.service` log:

    ```
    $ journalctl -u hello-cluster.service–f

    ```

    You will see something like this:

    ![Scheduling a fleet unit in the cluster](img/image00127.jpeg)
4.  To exit from the VM and reload the cluster, type the following command:

    ```
    $ vagrant reload

    ```

5.  Now, `ssh` again back to any machine:

    ```
    $ vagrant ssh core-02 -- -A

    ```

6.  Then run this command:

    ```
    $ fleetctl list-units

    ```

    The following output will be seen:

    ![Scheduling a fleet unit in the cluster](img/image00128.jpeg)
7.  As you can see, `hello-cluster.service` got scheduled on another machine; in our case, it is `core-01`. Suppose we `ssh` to it:

    ```
    $ vagrant ssh core-01 -- -A

    ```

8.  Then, we run the following command there. As a result, we will see the real-time log again:

    ```
    $ journalctl -u hello-cluster.service–f

    ```

    ![Scheduling a fleet unit in the cluster](img/image00129.jpeg)

# References

You can read more about how to use cloud-config at [https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/](https://coreos.com/docs/cluster-management/setup/cloudinit-cloud-config/).You can find out more about Vagrant at [https://docs.vagrantup.com](https://docs.vagrantup.com).If you have any issues or questions about Vagrant, you can subscribe to the Vagrant Google group at [https://groups.google.com/forum/#!forum/vagrant-up](https://groups.google.com/forum/#!forum/vagrant-up).

# Summary

In this chapter,you learned how to set up aCoreOS cluster, customize it via cloud-config, schedule `fleet` service units to the cluster, and check the `fleet` unit in the cluster status and log.In the next chapter,you will learn how to perform local and cloud development setups.