# Chapter 10. Introduction to Kubernetes

In this chapter, we will cover a short overview of Google Kubernetes, which manages containerized applications across multiple hosts in a cluster. As Kubernetes is a very large project, in this chapter, we will only overview its main concepts and some use cases, including these:

*   What is Kubernetes?
*   Primary components of Kubernetes
*   Kubernetes cluster setup
*   Tectonic—CoreOS and Kubernetes combined for a commercial implementation

# What is Kubernetes?

Google has been running everything in containers for more than decade. Internally, they use a system called Borg ([http://research.google.com/pubs/pub43438.html](http://research.google.com/pubs/pub43438.html)), the predecessor of Kubernetes, to scale and orchestrate containers across servers.

Lessons learned from Borg were used to build Kubernetes, an open source container orchestration system. It became popular very quickly when it was released in June 2014.

All of the best ideas from Borg were incorporated into Kubernetes. Many of Borg's developers now work on Kubernetes.

Kubernetes received thousands of stars at it's GitHub project ([https://github.com/GoogleCloudPlatform/kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)), and hundreds of supporters from the open source community and companies such as CoreOS, Red Hat, Microsoft, VMware, and so on.

## Primary components of Kubernetes

Kubernetes can be run on any modern Linux operating system.

Here are the main components of Kubernetes:

*   **Master**: This is the set of main Kubernetes control services, usually running on one server except the `etcd` cluster. However it can be spread around a few servers. The services of Kubernetes are as follows:

    *   `etcd` cluster
    *   API server
    *   Controller manager
    *   Scheduler

*   **Node**: This is a cluster worker. It can be a VM and/or bare-metal server. Nodes are managed from the master services and are dedicated to run pods. These two Kubernetes services must run on each node:

    *   Kubelet
    *   Network proxy

    Docker and rkt are used to run application containers. In future, we will see more support for application container systems there.

*   **Pod**: This is a group of application containers running with the shared context. Even a single application container must run in a Pod.
*   **Replication controllers**: These ensure that the specified numbers of pods are running. If there are too many pods, will be killed. If they are too less, then the required number of pods will be started. It is not recommended to run pods without replication controllers even if there is a single Pod.
*   **Services**: The same pod can be run only once. If it dies, the replication controller replaces it with a new pod. Every pod gets its own dedicated IP, which allows on the same node to run many containers on the port. But every time a pod is started from the template by replication controller gets a different IP, and this is where services come to help. Each service gets assigned a virtual IP, which stays with it until it dies.
*   **Labels**: These are the arbitrary key-value pairs that are used by every Kubernetes component; for example, the replication controller uses them for service discovery.
*   **Volumes**: A volume is a directory that is accessible from a container, and is used to store the container's stateful data.
*   **Kubectl**: This controls the Kubernetes cluster manager. For example, you can add/delete nodes, pods, or replication controllers; check their status; and so on. Kubernetes uses `manifest` files to set up pods, replication controllers, services, labels, and so on.

Kubernetes has a nice UI, which was built and contributed to by [http://kismatic.io/](http://kismatic.io/). It runs on an API server:

![Primary components of Kubernetes](img/image00180.jpeg)

This allows us to check the Kubernetes cluster's status and add/delete pods, replication controllers, and so on. It also allows us to manage a Kubernetes cluster from the UI in the same way as from `kubectl`.

[http://kismatic.io/](http://kismatic.io/) is also going to offer an enterprise/commercial version of Kubernetes in the near future.

# Kubernetes cluster setup

In the previous topic, we overviewed the main features of Kubernetes, so let's do some interesting stuff—installing small Kubernetes on Google Cloud.

Note, that if you are using a free/trial Google Cloud account, which has a limit of eight CPUs (eight VMs are allowed), you need to delete some of them. Let's replace our production cluster with a Kubernetes cluster. Select the VMs as per what is shown in the following screenshot. Then click on the **Delete** button in the top-right corner.

![Kubernetes cluster setup](img/image00181.jpeg)

Now we are ready to install a Kubernetes cluster:

1.  Type this in your terminal:

    ```
    $ cd coreos-essentials-book/Chapter10/Kubernetes_Cluster

    ```

    Note that as we have folders/files that are very similar to what we used to set up the Test/Staging/Production clusters, we are not going to review the scripts this time. You can always check out the setup files yourself and learn the differences there:

2.  Update the `settings` file there with your GC project ID and zone.
3.  Let's now run the first script, named `1-bootstrap_cluster.sh`:

    ```
    $ ./ 1-bootstrap_cluster.sh

    ```

    You should see an output similar to this:

    ![Kubernetes cluster setup](img/image00182.jpeg)

If you check out the Google Cloud console, you should see three new VMs there, namely **k8s-master**, **k8s-node1**, and **k8s-node2**:

![Kubernetes cluster setup](img/image00183.jpeg)

The `1-bootstrap_cluster.sh` script has installed a small CoreOS cluster, which is set up in the same way as our previous Test/Staging/Production cluster—one `etcd` server and two workers connected to it. And also create a new folder, `k8s-cluster`, in the user home folder where the `settings` file got copied and other binary files will be copied later on.

1.  Next, we need to install the `fleetctl`, `etcdctl`, and `kubectl` local clients on our computer to be able to communicate with the CoreOS cluster `etcd` and `fleet` services, and with the Kubernetes master service.

    Type the following line in your terminal:

    ```
    $ ./2-get_k8s_fleet_etcd.sh

    ```

    You should see an output similar to this:

    ![Kubernetes cluster setup](img/image00184.jpeg)
2.  Now let's install the Kubernetes cluster on top our new CoreOS cluster.

    Type this command in your terminal:

    ```
    $ ./3-install_k8s_fleet_units.sh

    ```

    You should see an output similar to what is shown here:

    ![Kubernetes cluster setup](img/image00185.jpeg)
3.  Let's try access our Kubernetes cluster via `""`, which was copied to `~/k8s-cluster/bin` by the `1-bootstrap_cluster.sh` script.

    Type this in your terminal:

    ```
    $ cd ~/k8s-cluster/bin
    $ ./set_k8s_access.sh

    ```

    You should get an output similar to the following:

    ![Kubernetes cluster setup](img/image00186.jpeg)

As you can see, our Kubernetes cluster is up and running.

What `set_k8s_access.sh` does is that it provides `fleetctl` and `kubectl` with access to the remote `k8s-master` server by forwarding the `localhost` ports 2379 (`fleet`) and 8080 (Kubernetes master) to it.

1.  Let's check out the Kubernetes cluster by typing this into the terminal:

    ```
    $ kubectl cluster-info

    ```

    You should see an output similar to this:

    ![Kubernetes cluster setup](img/image00187.jpeg)

    Perfect! Now we can access the remote Kubernetes cluster from our local computer.

2.  As we've got our Kubernetes cluster up and running, let's deploy the same `website1` Docker image that we used for our production cluster deployment.

    Type this into your terminal:

    ```
    $  kubectl run website1 --image=10.200.4.1:5000/website1 --replicas=2 --port=80

    ```

    You should see the following output:

    ![Kubernetes cluster setup](img/image00188.jpeg)

    The previous command has created two `website1` pods listening on `port 80`. It has also created a replication controller named `website1`, and this replication controller ensures that there are always two pods running.

    We can list created `pods` by typing the following into your terminal:

    ```
    $ kubectl get pods

    ```

    You should see an output like this:

    ![Kubernetes cluster setup](img/image00189.jpeg)

    To list the created replication controller, type this into your terminal:

    ```
    $ kubectl get rc

    ```

    You should see the following output:

    ![Kubernetes cluster setup](img/image00190.jpeg)
3.  Now, let's expose our pods to the Internet. The `Kubectl` command can integrate with the Google Compute Engine to add a public IP address for the `pods`. To do this, type the following line into your terminal:

    ```
    $ kubectl expose rc website1 --port=80 --type=LoadBalancer

    ```

    You should see an output like this:

    ![Kubernetes cluster setup](img/image00191.jpeg)

    The previous command created a service named `website1` and mapped an external IP address to the service. To find that IP address, type this into your terminal:

    ```
    $ kubectl get services

    ```

    You should see an output similar to the following:

    ![Kubernetes cluster setup](img/image00192.jpeg)

The IP in the bottom line is our IP, and it is of the load balancer. It is assigned to the `k8s-node-1` and `k8snode-2` servers and used by `website1` service.

Let's type this IP into our web browser. We should get an output similar to this:

![Kubernetes cluster setup](img/image00193.jpeg)

As you have seen previously, it shows exactly the same web page as we got on our production web servers. Also, it is exactly the same code as we had in the staging environment. We built the Docker image from it and used that Docker image for deployment on our production cluster and the Kubernetes cluster.

If you want, you can easily run more replicas of pods by using this simple command:

```
$ kubectl scale --replicas=4 rc website1

```

Let's check our replication controller by typing the following into our terminal:

```
$ kubectl get rc

```

You should see an output similar to this:

![Kubernetes cluster setup](img/image00194.jpeg)

The previous command scales the pods, and replication controller ensures that we always have four of them running.

### Note

You can find plenty of usage examples to play with at [https://github.com/GoogleCloudPlatform/kubernetes/tree/master/examples](https://github.com/GoogleCloudPlatform/kubernetes/tree/master/examples).

This book is too short to cover all the good things you can do with Kubernetes, but we should be seeing more Kubernetes books pop up soon.

### Note

Some other URLs to look at are given here:

If you are a Mac user, you can install one of the apps that will set your Kubernetes cluster on your Mac: 1 master x 2 nodes on [https://github.com/rimusz/coreos-osx-gui-kubernetes-cluster](https://github.com/rimusz/coreos-osx-gui-kubernetes-cluster), and standalone master/node on [https://github.com/rimusz/coreos-osx-gui-kubernetes-solo](https://github.com/rimusz/coreos-osx-gui-kubernetes-solo).

Other guides to Kubernetes on CoreOS are available at [https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/coreos.md](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/coreos.md).

# Tectonic – CoreOS and Kubernetes combined for a commercial implementation

Tectonic ([http://tectonic.com](http://tectonic.com)) is a commercial CoreOS distribution with a combined CoreOS and Kubernetes stack. It can be used by businesses of any size.

Tectonic is prepackaged with all the open source components of CoreOS and Kubernetes, and adds some more commercial features:

*   Management console/UI for workflows and dashboards
*   Corporate SSO integration
*   Quay-integrated container registry for building and sharing Linux containers
*   Tools for automation of container deployments
*   Customized rolling updates

It can run in public clouds or on-premise.

Its management console is simple and easy to use:

![Tectonic – CoreOS and Kubernetes combined for a commercial implementation](img/image00195.jpeg)

In the preceding screenshot, we have a visualization of our **Replication controllers** (**RC**). On the left-hand side, you can' see each RC with the labels will assign to pods as they're instantiated. Below the name of the RC, you'll see a list of all running pods that match the same label queries.

![Tectonic – CoreOS and Kubernetes combined for a commercial implementation](img/image00196.jpeg)

The preceding screenshot shows us the **elasticsearch** replication controller state, which labels are used there, and pod volumes.

Tectonic aims to provide an easily container deployment solution, and companies can begin seeing its benefits very quickly of using containers in enterprise.

# Summary

In this chapter, we overviewed Google Kubernetes and covered what is about, its main components, and its CoreOS commercial implementation.

We hope that this book will equip you with all the information you need to leverage the power of CoreOS and the related containers, and help you develop effective computing networks. Thank you for reading it!