# Chapter 5. Building a Development Environment

In this chapter, we will cover how to set up a local CoreOS environment for development on a personal computer, and a test and staging environment cluster on the VM instances of Google Cloud's Compute Engine. These are the topics we will cover:

*   Setting up a local development environment
*   Bootstrapping a remote test/staging cluster on GCE

# Setting up the local development environment

We are going to learn how to set up a development environment on our personal computer with the help of VirtualBox and Vagrant, as we did in an earlier chapter. Building and testing `docker` images and coding locally makes you more productive, it saves time, and Docker repository can be pushed to the docker registry (private or not) when your docker images are ready. The same goes for the code; you just work on it and test it locally. When it is ready, you can merge it with the git test branch where your team/client can test it further.

## Setting up the development VM

In the previous chapters, you learned how to install CoreOS via Vagrant on your PC. Here, we have prepared installation scripts for Linux and OS X to go straight to the point. You can download the latest *CoreOS Essentials* book example files from GitHub repository:

```
$ git clone https://github.com/rimusz/coreos-essentials-book/

```

To install a local Vagrant-based development VM, type this:

```
$ cd coreos-essentials-book/chapter5/Local_Development_VM
$ ./install_local_dev.sh

```

You should see an output similar to this:

![Setting up the development VM](img/image00130.jpeg)

Hang on! There's more!

![Setting up the development VM](img/image00131.jpeg)

This will perform a VM installation similar to the installation that we did in [Chapter 1](part0014.xhtml#aid-DB7S1 "Chapter 1. CoreOS – Overview and Installation"), *CoreOS – Overview and Installation*, but in a more automated way this time.

## What happened during the VM installation?

Let's check out what happened during the VM installation. To sum up:

*   A new CoreOS VM (VirtualBox/Vagrant-based) was installed
*   A new folder called `coreos-dev-env` was created in your `Home` folder

Run the following commands:

```
$ cd ~/coreos-dev-env
$ ls
bin 
fleet 
share 
vm 
vm_halt.sh 
vm_ssh.sh 
vm_up.sh

```

As a result, this is what we see:

*   Four folders, which consist of the following list:

    *   `bin`: `docker`, `etcdctl` and `fleetctl` files
    *   `fleet`: The `nginx.service fleet` unit is stored here
    *   `share`: This is shared folder between the host and VM
    *   `vm`: Vagrantfile, `config.rb` and `user-data` files

*   We also have three files:

    *   `vm_halt.sh`: This is used to shut down the CoreOS VM
    *   `vm_ssh.sh`: This is used to `ssh` to the CoreOS VM
    *   `vm_up.sh`: This is used to start the CoreOS VM, with the OS shell preset to the following:

        ```
        # Set the environment variable for the docker daemon
        export DOCKER_HOST=tcp://127.0.0.1:2375
        # path to the bin folder where we store our binary files
        export PATH=${HOME}/coreos-dev-env/bin:$PATH
        # set etcd endpoint
        export ETCDCTL_PEERS=http://172.19.20.99:2379
        # set fleetctl endpoint
        export FLEETCTL_ENDPOINT=http://172.19.20.99:2379
        export FLEETCTL_DRIVER=etcd
        export FLEETCTL_STRICT_HOST_KEY_CHECKING=false
        ```

Now that we have installed our CoreOS VM, let's run `vm_up.sh`. We should see this output in the **Terminal** window:

```
$ cd ~/coreos-dev-env
$ ./vm_up.sh

```

You should see output similar to this:

![What happened during the VM installation?](img/image00132.jpeg)

As we can see in the preceding screenshot, we do not have any errors. Only `fleetctl list-machines` shows our CoreOS VM machine, and we have no `docker` containers and `fleet` units running there yet.

## Deploying the fleet units

Let's deploy some fleet units to verify that our development environment works fine. Run the following commands:

```
$ cd fleet
$ fleetctl start nginx.service

```

### Note

It can take a bit of time for docker to download the `nginx` image.

You can check out the `nginx.service` unit's status:

```
$ fleetctl status nginx.service

```

You should see output similar to this:

![Deploying the fleet units](img/image00133.jpeg)

Once the `nginx fleet` unit is deployed, open in your browser `http://172.19.20.99`. You should see the following message:

![Deploying the fleet units](img/image00134.jpeg)

Let's check out what happened there. We scheduled this `nginx.service` unit with `fleetctl`:

```
$ cat ~/coreos-dev-env/fleet/nginx.service

[Unit]
Description=nginx

[Service]
User=core
TimeoutStartSec=0
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker rm nginx
ExecStart=/usr/bin/docker run --rm --name nginx -p 80:80 \
 -v /home/core/share/nginx/html:/usr/share/nginx/html \
 nginx:latest
#
ExecStop=/usr/bin/docker stop nginx
ExecStopPost=-/usr/bin/docker rm nginx

Restart=always
RestartSec=10s

[X-Fleet]
```

Then, we used the official `nginx` image from the docker registry, and shared our local `~/coreos-dev-env/share` folder with `/home/core/share`, which was mounted afterwards as a docker volume `/home/core/share/nginx/html:/usr/share/nginx/html`.

So, whatever `html` files we put into our local `~/coreos-dev-env/share/nginx/html` folder will be picked up automatically by `nginx`.

Let's overview what advantages such a setup gives us:

*   We can build and test docker containers locally, and then push them to the docker registry (private or public).
*   Test our code locally and push it to the git repository when we are done with it.
*   By having a local development setup, productivity really increases, as everything is done much faster. We do not have build new docker containers upon every code change, push them to the remote docker registry, pull them at some remote test servers, and so on.
*   It is very easy to clean up the setup and get it working from a clean start again, reusing the configured `fleet` units to start the all required docker containers.

Very good! So, now, we have a fully operational local development setup!

### Note

This setup is as per the CoreOS documentation at [https://coreos.com/docs/cluster-management/setup/cluster-architectures/](https://coreos.com/docs/cluster-management/setup/cluster-architectures/), in the *Docker Dev Environment on Laptop* section.

Go through the `coreos-dev-install.sh` bash script, which sets up your local development VM. It is a simple script and is well commented, so it should not be too hard to understand its logic.

If you are a Mac user, you can download from [https://github.com/rimusz/coreos-osx-gui](https://github.com/rimusz/coreos-osx-gui) and use my Mac App **CoreOS-Vagrant GUI for Mac OS X**, which has a nice UI to manage CoreOS VM. It will automatically set up the CoreOS VM environment.

![Deploying the fleet units](img/image00135.jpeg)

# Bootstrapping a remote test/staging cluster on GCE

So, we have successfully built our local development setup. Let's get to the next level, that is, building our test/staging environment on the cloud.

We are going to use Google Cloud's Compute Engine, so you need a Google Cloud account for this. If you do not have it, for the purpose of running the examples in the book, you can open a trial account at [https://cloud.google.com/compute/](https://cloud.google.com/compute/). A trial account lasts for 60 days and has $300 as credits, enough to run all of this book's examples. When you are done with opening the account, Google Cloud SDK needs to be installed from [https://cloud.google.com/sdk/](https://cloud.google.com/sdk/).

In this topic, we will follow the recommendations on how to set up CoreOS cluster by referring to *Easy Development/Testing Cluster* from [https://coreos.com/docs/cluster-management/setup/cluster-architectures/](https://coreos.com/docs/cluster-management/setup/cluster-architectures/).

## Test/staging cluster setup

Okay, let's get our cloud cluster installed, as you have already downloaded this book's code examples. Carry out these steps in the shown order:

1.  Run the following commands:

    ```
    $ cd coreos-essentials-book/chapter5/Test_Staging_Cluster
    $ ls
    cloud-config
    create_cluster_control.sh
    create_cluster_workers.sh
    files
    fleet
    install_fleetctl_and_scripts.sh
    settings

    Let's check "settings" file first:
    $ cat settings
    ### CoreOS Test/Staging Cluster on GCE settings

    ## change Google Cloud settings as per your requirements
    # GC settings

    # CoreOS RELEASE CHANNEL
    channel=beta

    # SET YOUR PROJECT AND ZONE !!!
    project=my-cloud-project
    zone=europe-west1-d

    # ETCD CONTROL AND NODES MACHINES TYPE
    #
    control_machine_type=g1-small
    #
    worker_machine_type=n1-standard-1
    ##

    ###
    ```

2.  Update the `settings` with your Google Cloud project ID and zone where you want the CoreOS instances to be deployed:

    ```
    # SET YOUR PROJECT AND ZONE !!!
    project=my-cloud-project
    zone=europe-west1-d
    ```

3.  Next, let's install our control server, which is our `etcd` cluster node:

    ```
    $ ./create_cluster_control.sh

    ```

    ![Test/staging cluster setup](img/image00136.jpeg)

We just created our new cluster `etcd` control node.

1.  Let's check out what we have in this script:

    ```
    #!/bin/bash
    # Create TS cluster control

    # Update required settings in "settings" file before running this script

    function pause(){
    read -p "$*"
    }

    ## Fetch GC settings
    # project and zone
    project=$(cat settings | grep project= | head -1 | cut -f2 -d"=")
    zone=$(cat settings | grep zone= | head -1 | cut -f2 -d"=")
    # CoreOS release channel
    channel=$(cat settings | grep channel= | head -1 | cut -f2 -d"=")
    # control instance type
    control_machine_type=$(cat settings | grep control_machine_type= | head -1 | cut -f2 -d"=")
    # get the latest full image name
    image=$(gcloud compute images list --project=$project | grep -v grep | grep coreos-$channel | awk {'print $1'})
    ##

    # create an instance
    gcloud compute instances create tsc-control1 --project=$project --image=$image --image-project=coreos-cloud \
     --boot-disk-size=10 --zone=$zone --machine-type=$control_machine_type \
     --metadata-from-file user-data=cloud-config/control1.yaml --can-ip-forward --tags tsc-control1 tsc

    # create a static IP for the new instance
    gcloud compute routes create ip-10-200-1-1-tsc-control1 --project=$project \
     --next-hop-instance tsc-control1 \
     --next-hop-instance-zone $zone \
     --destination-range 10.200.1.1/32

    echo " "
    echo "Setup has finished !!!"
    pause 'Press [Enter] key to continue...'
    # end of bash script

    ```

It fetches the settings needed for Google Cloud from the `settings` file. With the help of `gcloud` utility from the Google Cloud SDK, it sets up the `tsld-control1` instance and assigns to it a static internal IP `10.200.1.1`. This IP will be used by workers to connect the `etcd` cluster, which will run on `tsc-control1`.

In the `cloud-config` folder, we have the `cloud-config` files needed to create CoreOS instances on GCE.

Open `control1.yaml` and check out what is there in it:

```
$ cat control1.yaml
#cloud-config

coreos:

etcd2:
 name: control1
 initial-advertise-peer-urls: http://10.200.1.1:2380
 initial-cluster-token: control_etcd
 initial-cluster: control1=http://10.200.1.1:2380
 initial-cluster-state: new
 listen-peer-urls: http://10.200.1.1:2380,http://10.200.1.1:7001
 listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
 advertise-client-urls: http://10.200.1.1:2379,http://10.200.1.1:4001
 fleet:
 metadata: "role=services,cpeer=tsc-control1"
 units:
 - name: 00-ens4v1.network
 runtime: true
 content: |
 [Match]
 Name=ens4v1

 [Network]
 Address=10.200.1.1/24
 - name: etcd2.service
 command: start
 - name: fleet.service
 command: start
 - name: docker.service
 command: start
 drop-ins:
 - name: 50-insecure-registry.conf
 content: |
 [Unit]
 [Service]
 Environment=DOCKER_OPTS='--insecure-registry="0.0.0.0/0"'
write_files:
 - path: /etc/resolv.conf
 permissions: 0644
 owner: root
 content: |
 nameserver 169.254.169.254
 nameserver 10.240.0.1
#end of cloud-config

```

As you see, we have `cloud-config` file for the control machine, which does the following:

1.  It creates a node `etcd` cluster with a static IP of `10.200.1.1`, which will be used to connect to `etcd` cluster.
2.  It sets the `fleet` metadata to `role=services,cpeer=tsc-control1`.
3.  `Unit 00-ens4v1.network` assigns a static IP of `10.200.1.1`.
4.  The `docker.service` drop-in `50-insecure-registry.conf` sets `--insecure-registry="0.0.0.0/0"`, which allows you to connect to any privately hosted docker registry.
5.  In the `write_files` part, we update `/etc/resolv.conf` with Google Cloud DNS servers, which sometimes do not get automatically put there if the static IP is assigned to the instance.

### Creating our cluster workers

In order to create the cluster workers, the command to be used is as follows:

```
$ ./create_cluster_workers.sh

```

![Creating our cluster workers](img/image00137.jpeg)

Make a note of the workers' external IPs, as shown in the previous screenshot; we will need them later.

Of course, you can always check them at the Google Developers Console too.

![Creating our cluster workers](img/image00138.jpeg)

Let's check out what we have inside the `test1.yaml` and `staging1.yaml` files in the cloud-`config` folder. Run the following command:

```
$ cat test1.yaml
#cloud-config

coreos:
 etcd2:
 listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
 initial-cluster: control1=http://10.200.1.1:2380
 proxy: on
 fleet:
 public-ip: $public_ipv4
 metadata: "role=worker,cpeer=tsc-test1"
 units:
 - name: etcd2.service
 command: start
 - name: fleet.service
 command: start
 - name: docker.service
 command: start
 drop-ins:
 - name: 50-insecure-registry.conf
 content: |
 [Unit]
 [Service]
 Environment=DOCKER_OPTS='--insecure-registry="0.0.0.0/0"'
# end of cloud-config

```

As we can see, we have `cloud-config` file for the `test1` machine:

*   It connects to the `etcd` cluster machine `control1` and enables `etcd2` in proxy mode, which allows anything running on the host to access the `etcd` cluster via the `127.0.0.1` address
*   It sets the `fleet` metadata `role=services,cpeer=tsc-test1`
*   The `docker.service` drop-in `50-insecure-registry.conf` sets `--insecure-registry="0.0.0.0/0"`, which will allow you to connect to any privately hosted docker registry

That's it!

If you check out the `tsc-staging1.yaml` cloud-config file, you will see that it is almost identical to `test1.yaml`, except that the `fleet` metadata has `cpeer=tsc-staging1` in it. But we are not done yet!

Let's now install the OS X/Linux clients, which will allow us to manage the cloud development cluster from our local computer.

Let's run this installation script:

```
$ ./install_fleetctl_and_scripts.sh

```

You should see the following output:

![Creating our cluster workers](img/image00139.jpeg)

So, what has the last script done?

In your home folder, it created a new folder called `~/coreos-tsc-gce`, which has two folders:

*   `bin`

    *   `etcdctl`: This is the shell script used to access the `etcdctl` client on a remote cluster `control1` node
    *   `fleetctl`: The local `fleetctl` client is used to control the remote cluster
    *   `staging1.sh`: Make `ssh` connection to remote `staging1` worker
    *   `test1.sh`: Make `ssh` connection to remote `test1` worker
    *   `set_cluster_access.sh`: This sets up shell access to the remote cluster

*   `fleet`

    *   `test1_webserver.service`: Our `test1` server's `fleet` unit
    *   `staging1_webserver.service`: Our `staging1` server's `fleet` unit

Now, let's take a look at `set_cluster_access.sh`:

```
$ cd ~/coreos-tsc-gce/bin
$ cat set_cluster_access.sh
#!/bin/bash

# Setup Client SSH Tunnels
ssh-add ~/.ssh/google_compute_engine &>/dev/null

# SET
# path to the cluster folder where we store our binary files
export PATH=${HOME}/coreos-tsc-gce/bin:$PATH
# fleet tunnel
export FLEETCTL_TUNNEL=104.155.61.42 # our control1 external IP
export FLEETCTL_STRICT_HOST_KEY_CHECKING=false

echo "etcd cluster:"
etcdctl --no-sync ls /

echo "list fleet units:"
fleetctl list-units

/bin/bash

```

This script is preset by `./install_fleetctl_and_scripts.sh` with the remote `control1` external IP, and allows us to issue remote `fleet` control commands:

```
$ ./set_cluster_access.sh

```

![Creating our cluster workers](img/image00140.jpeg)

Very good! Our cluster is up and running, and the workers are connected to the `etcd` cluster.

Now we can run `fleetctl` commands on the remote cluster from our local computer.

### Running fleetctl commands on the remote cluster

Let's now install the `nginx` fleet units we have in the `~/coreos-tsc-gce/fleet` folder. Run the following command:

```
$ cd ~/coreos-tsc-gce/fleet

```

Let's first submit the `fleet` units to the cluster:

```
$ fleetctl submit *.service

```

Now, let's start them:

```
$ fleetctl start *.service

```

You should see something like what is shown in the following screenshot:

![Running fleetctl commands on the remote cluster](img/image00141.jpeg)

Give some time to docker to download the nginx image from the docker registry. We can then check the status of our newly deployed `fleet` units using the following command:

```
$ fleetctl status *.service

```

![Running fleetctl commands on the remote cluster](img/image00142.jpeg)

Then, run this command:

```
$ fleetctl list-units

```

![Running fleetctl commands on the remote cluster](img/image00143.jpeg)

Perfect!

Now, in your web browser, open the workers' external IPs, and you should see this:

![Running fleetctl commands on the remote cluster](img/image00144.jpeg)

The `nginx` servers are now working. The reason they are showing this error message is that we have not provided any `index.html` file yet. We will do that in the next chapter.

But, before we finish this chapter, let's check out our `test/staging nginx fleet` units:

```
$ cd ~/coreos-tsc-gce/fleet
$ cat test1_webserver.service

```

You should see something like the following code:

```
[Unit]
Description=nginx

[Service]
User=core
TimeoutStartSec=0 
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker rm nginx
ExecStart=/usr/bin/docker run --rm --name test1-webserver -p 80:80 \
-v /home/core/share/nginx/html:/usr/share/nginx/html \
nginx:latest
#
ExecStop=/usr/bin/docker stop nginx
ExecStopPost=-/usr/bin/docker rm nginx

Restart=always
RestartSec=10s
[X-Fleet]
MachineMetadata=cpeer=tsc-test1 # this where our fleet unit gets scheduled

```

There are a few things to note here:

*   `Staging1` has an almost identical unit; instead of `test1`, it has `staging1` there. So, we reused the same fleet unit as we used for our local development machine, with a few changes.
*   At `ExecStart`, we used `test1-webserver` and `staging1-webserver`, so by using `fleetctl list-units`, we can see which one is which.

    We added this bit:

    ```
    [X-Fleet]
    MachineMetadata=cpeer=tsc-test1

    ```

This will schedule the unit to the particular cluster worker.

If you are a Mac user, you can download from [https://github.com/rimusz/coreos-osx-gui-cluster](https://github.com/rimusz/coreos-osx-gui-cluster) and use my Mac App **CoreOS-Vagrant Cluster GUI for Mac OS X**, which has a nice UI for managing CoreOS VMs on your computer.

![Running fleetctl commands on the remote cluster](img/image00145.jpeg)

This app will set up a small `control+` two-node local cluster, which makes easier to test cluster things on local computer before pushing them to the cloud.

# References

You can read more about the CoreOS cluster architectures that we used for the local and cloud test/staging setup at [https://coreos.com/docs/cluster-management/setup/cluster-architectures/](https://coreos.com/docs/cluster-management/setup/cluster-architectures/).

# Summary

In this chapter, you learned how to set up a CoreOS local development environment and a remote test/staging cluster on GCE. We scheduled fleet units based on different metadata tags.

In the next chapter, we will see how to deploy code to our cloud servers.