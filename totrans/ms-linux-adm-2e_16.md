# 16

# Deploying Applications with Kubernetes

Whether you are a seasoned system administrator managing containerized applications or a DevOps engineer automating app orchestration workflows, **Kubernetes** could be your platform of choice. This chapter will introduce you to Kubernetes and will guide you through the basic process of building and configuring a **Kubernetes cluster**. We’ll use Kubernetes to run and scale a simple application in a secure and highly available environment. You will also learn how to interact with Kubernetes using the **command-line** **interface** (**CLI**).

By the end of this chapter, you’ll learn how to install, configure, and manage a Kubernetes cluster on-premises. We’ll also show you how to deploy and scale an application using Kubernetes.

Here’s a brief outline of the topics we will cover in this chapter:

*   Introducing Kubernetes architecture and the API object model
*   Installing and configuring Kubernetes
*   Working with Kubernetes using the `kubectl` command-line tool and deploying applications

# Technical requirements

You should be familiar with Linux and the CLI in general. A good grasp of **TCP/IP networking** and **Docker** containers would go a long way in making your journey of learning Kubernetes easier.

You will also need the following:

*   A local desktop machine with a Linux distribution of your choice to install and experiment with the CLI tools used in this chapter. We will use both Debian and Ubuntu LTS.
*   A powerful desktop system with at least 8 CPU cores and at least 16 GB of RAM will allow you to replicate the necessary environment on your desktop as we’ll be devoting a relatively large section to building a Kubernetes cluster using VMs.
*   A desktop hypervisor.

Now, let’s start our journey together to discover Kubernetes.

# Introducing Kubernetes

Kubernetes is an open source **container orchestrator** initially developed by Google. A container orchestrator is a piece of software that automatically manages (including provisioning, deployment, and scaling) containerized applications. Assuming an application uses containerized microservices, a container orchestration system provides the following features:

*   **Elastic orchestration (autoscaling)**: This involves automatically starting and stopping application services (containers) based on specific requirements and conditions – for example, launching multiple web server instances with an increasing number of requests and eventually terminating servers when the number of requests drops below a certain threshold
*   **Workload management**: This involves optimally deploying and distributing application services across the underlying cluster to ensure mandatory dependencies and redundancy – for example, running a web server endpoint on each cluster node for high availability
*   **Infrastructure abstraction**: This involves providing container runtime, networking, and load-balancing capabilities – for example, distributing the load among multiple web server containers and autoconfiguring the underlying network connectivity with a database app container
*   **Declarative configuration**: This involves describing and ensuring the desired state of a multi-tiered application – for example, a web server should be ready for serving requests only when the database backend is up and running, and the underlying storage is available

A classic example of workload orchestration is a video-on-demand streaming service. With a popular new TV show in high demand, the number of streaming requests would significantly exceed the average during a regular season. With **Kubernetes**, we could scale out the number of web servers based on the volume of streaming sessions. We could also control the possible scale-out of some of the middle-tier components, such as database instances (serving the authentication requests) and storage cache (serving the streams). When the TV show goes out of fashion, and the number of requests drops significantly, Kubernetes terminates the surplus instances, automatically reducing the application deployment’s footprint and, consequently, the underlying costs.

Here are some key benefits of deploying applications with Kubernetes:

*   **Speedy deployment**: Application containers are created and launched relatively fast, using either a **declarative** or **imperative** configuration model (as we’ll see in the *Introducing the Kubernetes object model* section later in this chapter)
*   **Quick iterations**: Application upgrades are relatively straightforward, with the underlying infrastructure simply seamlessly replacing the related container
*   **Rapid recovery**: If an application crashes or becomes unavailable, Kubernetes automatically restores the application to the desired state by replacing the related container
*   **Reduced operation costs**: The containerized environment and infrastructure abstraction of Kubernetes yields minimal administration and maintenance efforts with relatively low resources for running applications

Now that we have introduced Kubernetes, let’s look at its basic operating principles next.

## Understanding the Kubernetes architecture

There are three major concepts at the core of the working model of Kubernetes:

*   **Declarative configuration or desired state**: This concept describes the overall application state and microservices, deploying the required containers and related resources, including network, storage, and load balancers, to achieve a running functional state of the application
*   **Controllers or controller loops**: This monitors the desired state of the system and takes corrective action when needed, such as replacing a failed application container or adding additional resources for scale-out workloads
*   **API object model**: This model represents the actual implementation of the desired state, using various configuration objects and the interaction – the **application programming interface** (**API**) – between these objects

For a better grasp of the internals of Kubernetes, we need to take a closer look at the Kubernetes object model and related API. Also, you can check out *Figure 16**.1* for a visual explanation of the Kubernetes (cluster) architecture.

## Introducing the Kubernetes object model

The Kubernetes architecture defines a collection of objects representing the desired state of a system. An **object**, in this context, is a programmatic term to describe the behavior of a subsystem. Multiple objects interact with one another via the API, shaping the desired state over time. In other words, the Kubernetes object model is the programmatic representation of the desired state.

So, what are these objects in Kubernetes? We’ll briefly enumerate some of the more important ones and further elaborate on each in the following sections: API server, pods, controllers, services, and storage.

We use these API objects to configure the system’s state, using either a declarative or imperative model:

*   With a **declarative configuration model**, we describe the state of the system, usually with a configuration file or manifest (in YAML or JSON format). Such a configuration may include and deploy multiple API objects and regard the system as a whole.
*   The **imperative configuration model** uses individual commands to configure and deploy specific API objects, typically acting on a single target or subsystem.

Let’s look at the API server first – the central piece in the Kubernetes object model.

### Introducing the API server

The API server is the core hub of the Kubernetes object model, acting as a management endpoint for the desired state of the system. The API server exposes an HTTP REST interface using JSON payloads. It is accessible in two ways:

*   **Internally**: It is accessed internally by other API objects
*   **Externally**: It is accessed externally by configuration and management workflows

The API server is essentially the gateway of interaction with the Kubernetes cluster, both from the outside and within. A Kubernetes cluster is a framework of different nodes that run containerized applications. The cluster is the basic running mode of Kubernetes itself. More on the Kubernetes cluster will be provided in the *Anatomy of a Kubernetes cluster* section. A system administrator connects to the API server endpoint to configure and manage a Kubernetes cluster, typically via a CLI. Internally, Kubernetes API objects connect to the API server to provide an update of their state. In return, the API server may further adjust the internal configuration of the API objects toward the desired state.

The API objects are the building blocks of the internal configuration or desired state of a Kubernetes cluster. Let’s look at a few of these API objects next.

### Introducing pods

A **pod** represents the basic working unit in Kubernetes, running as a single- or multi-container application. A pod is also known as the **unit of scheduling** in Kubernetes. In other words, containers within the same pod are guaranteed to be deployed together on the same cluster node.

A pod essentially represents a microservice (or a service) within the application’s service mesh. Considering the classic example of a web application, we may have the following pods running in the cluster:

*   Web server (Nginx)
*   Authentication (Vault)
*   Database (PostgreSQL)
*   Storage (NAS)

Each of these services (or applications) runs within their pod. Multiple pods of the same application (for example, a web server) make up a **ReplicaSet**. We’ll look at ReplicaSets closer in the *Introducing* *controllers* section.

Some essential features of pods are as follows:

*   They have an **ephemeral nature**. Once a pod is terminated, it is gone for good. No pod ever gets redeployed in Kubernetes. Consequently, pods don’t persist in any state unless they use persistent storage or a local volume to save their data.
*   Pods are an **atomic unit** – they are either deployed or not. For a single-container pod, atomicity is almost a given. For multi-container pods, atomicity means that a pod is deployed only when each of the constituent containers is deployed. If any of the containers fail to deploy, the pod will not be deployed, and hence there’s no pod. If one of the containers within a running multi-container pod fails, the whole pod is terminated.
*   Kubernetes uses **probes** (such as liveliness and readiness) to monitor the health of an application inside a pod. This is because a pod could be deployed and running, but that doesn’t necessarily mean the application or service within the pod is healthy. For example, a web server pod can have a probe that checks a specific URL and decides whether it’s healthy based on the response.

Kubernetes tracks the state of pods using controllers. Let’s look at controllers next.

### Introducing controllers

**Controllers** in Kubernetes are control loops responsible for keeping the system in the desired state or bringing the system closer to the desired state by constantly watching the state of the cluster. For example, a controller may detect that a pod is not responding and request the deployment of a new pod while terminating the old one. Let’s look at two key types of controllers:

*   A controller may also add or remove pods of a specific type to and from a collection of pod replicas. Such controllers are called **ReplicaSets**, and their responsibility is to accommodate a particular number of pod replicas based on the current state of the application. For example, suppose an application requires three web server pods, and one of them becomes unavailable (due to a failed probe). In that case, the ReplicaSet controller ensures that the failed pod is deleted, and a new one takes its place.
*   When deploying applications in Kubernetes, we usually don’t use ReplicaSets directly to create pods. We use the `v1`) with several pods, all running version 1 of our application, and we want to upgrade them to version 2\. Remember, pods cannot be regenerated or upgraded. Instead, we’ll define a second ReplicaSet (`v2`), creating the version 2 pods. The Deployment controller will tear down the `v1` ReplicaSet and bring up `v2`. Kubernetes performs the rollout seamlessly, with minimal to no disruption of service. The Deployment controller manages the transition between the `v1` and `v2` ReplicaSets and even rolls back the transition if needed.

There are many other controller types in Kubernetes, and we encourage you to explore them at [https://kubernetes.io/docs/concepts/workloads/controllers/](https://kubernetes.io/docs/concepts/workloads/controllers/).

As applications scale-out or terminate, the related pods are deployed or removed. Services provide access to the dynamic and transient world of pods. We’ll look at Services next.

### Introducing Services

**Services** provide persistent access to the applications running in pods. It is the Services’ responsibility to ensure that the pods are accessible by routing the traffic to the corresponding application endpoints. In other words, Services provide network abstraction for communicating with pods through IP addresses, routing, and DNS resolution. As pods are deployed or terminated based on the system’s desired state, Kubernetes dynamically updates the Service endpoint of the pods, with minimal to no disruption in terms of accessing the related applications. As users and applications access the Service endpoint’s persistent IP address, the Service will ensure that the routing information is up to date and traffic is exclusively routed to the running and healthy pods. Services can also be leveraged to load-balance the application traffic between pods and scale pods up or down based on demand.

So far, we have looked at Kubernetes API objects controlling the deployment, access, and life cycle of application Services. What about the persistent data that applications require? We’ll look at the Kubernetes storage next.

### Introducing storage

Kubernetes provides various storage types for applications running within the cluster. The most common are **volumes** and **persistent volumes**. Due to the ephemeral nature of pods, application data stored within a pod using volumes is lost when the pod is terminated. Persistent volumes are defined and managed at the Kubernetes cluster level, and they are independent of pods. Applications (pods) requiring a persistent state would reserve a persistent volume (of a specific size), using a **persistent volume claim**. When a pod using a persistent volume terminates, the new pod replacing the old one retrieves the current state from the persistent volume and will continue using the underlying storage.

For more information on Kubernetes storage types, please refer to [https://kubernetes.io/docs/concepts/storage/](https://kubernetes.io/docs/concepts/storage/).

Now that we are familiar with the Kubernetes API object model, let’s quickly go through the architecture of a Kubernetes cluster.

## The anatomy of a Kubernetes cluster

A Kubernetes cluster consists of one **Control Plane** (**CP**) **node** and one or more **worker nodes**. The following diagram presents a high-level view of the Kubernetes architecture:

![Figure 16.1 – Kubernetes cluster architecture](img/B19682_16_01.jpg)

Figure 16.1 – Kubernetes cluster architecture

The components of a Kubernetes cluster that are shown in the preceding figure are divided into the worker node and CP node as the two major components. The Worker nodes have different components, such as the container runtime, kubelet, and kube-proxy, and the CP node has the API server, the controller manager, and the scheduler. All of these components will be discussed in detail in the following sections.

First, let’s look at the Kubernetes cluster nodes shown in the preceding image in some detail next, starting with the CP node.

### Introducing the Kubernetes CP

The **Kubernetes CP** provides essential Services for deploying and orchestrating application workloads, and it runs on a dedicated node in the Kubernetes cluster – the CP node. This node, also known as the **master node**, implements the core components of a Kubernetes cluster, such as resource scheduling and monitoring. It’s also the primary access point for cluster administration. Here are the key subsystems of a CP node:

*   **API server**: The central communication hub between Kubernetes API objects; it also provides the cluster’s management endpoint accessible either via CLI or the Kubernetes web administration console (dashboard)
*   **Scheduler**: This decides when and which nodes to deploy the pods on, depending on resource allocation and administrative policies
*   **Controller manager**: This maintains the control loops, monitoring and shaping the desired state of the system
*   **etcd**: Also known as the **cluster store**, this is a highly available persisted database, maintaining the state of the Kubernetes cluster and related API objects; the information in etcd is stored as key-value pairs
*   `kubectl`: The primary administrative CLI for managing and interacting with the Kubernetes cluster; `kubectl` communicates directly with the API server, and it may connect remotely to a cluster

A detailed architectural overview of the Kubernetes CP is beyond the scope of this chapter. You may explore the related concepts in more detail at [https://kubernetes.io/docs/concepts/architecture/](https://kubernetes.io/docs/concepts/architecture/).

Next, let’s take a brief look at the Kubernetes node – the workhorse of a Kubernetes cluster.

### Introducing the Kubernetes nodes

In a Kubernetes cluster, the **nodes** – also referred to as **worker nodes** – run the actual application pods and maintain their full life cycle. Nodes provide the compute capacity of Kubernetes and ensure that the workloads are uniformly distributed across the cluster when deploying and running pods. Nodes can be configured either as physical (bare metal) or VMs.

Let’s enumerate the key elements of a Kubernetes node:

*   **Kubelet**: This processes CP requests (from the scheduler) to deploy and start application pods; the kubelet also monitors the node and pod state, reporting the related changes to the API server
*   **Kube-Proxy**: Dynamically configures the virtual networking environment for the applications running in the pods; it routes the network traffic, provides load balancing, and maintains the IP addresses of Services and pods
*   `containerd` and Docker)

All the preceding Services run on *each* node in the Kubernetes cluster, including the CP node. These components in the CP are required by special-purpose pods, providing specific CP Services, such as DNS, ingress (load balancing), and dashboard (web console).

For more information on Kubernetes nodes and related architectural concepts, please visit [https://kubernetes.io/docs/concepts/architecture/nodes/](https://kubernetes.io/docs/concepts/architecture/nodes/).

Now that we have become familiar with some of the key concepts and cluster components, let’s get ready to install and configure Kubernetes.

# Installing and configuring Kubernetes

Before installing or using Kubernetes, you have to decide on the infrastructure you’ll use, whether that be on-premises or public cloud. Second, you’ll have to choose between an **Infrastructure-as-a-Service** (**IaaS**) or a **Platform-as-a-Service** (**PaaS**) model. With IaaS, you’ll have to install, configure, manage, and maintain the Kubernetes cluster yourself, either on physical (bare metal) or VMs. The related operation efforts are not straightforward and should be considered carefully. If you choose a PaaS solution, available from all major public cloud providers, you’ll be limited to only administrative tasks but saved from the burden of maintaining the underlying infrastructure.

In this chapter, we’ll cover only IaaS deployments of Kubernetes. For IaaS, we’ll use a local desktop environment running Ubuntu VMs.

For the on-premises installation, we may also choose between a lightweight desktop version of Kubernetes or a full-blown cluster with multiple nodes. Let’s look at some of the most common desktop versions of Kubernetes next.

## Installing Kubernetes on a desktop

If you’re looking only to experiment with Kubernetes, a desktop version may fit the bill. Desktop versions of Kubernetes usually deploy a single-node cluster on your local machine. Depending on your platform of choice, whether it be Windows, macOS, or Linux, you have plenty of Kubernetes engines to select from. Here are just a few:

*   **Docker Desktop (macOS,** **Windows)**: [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
*   **minikube (Linux, macOS,** **Windows)**: [https://minikube.sigs.k8s.io/docs/](https://minikube.sigs.k8s.io/docs/)
*   **Microk8s (Linux, macOS,** **Windows)**: [https://microk8s.io/](https://microk8s.io/)
*   **k3s (****Linux)**: [https://k3s.io/](https://k3s.io/)

In this section, we’ll show you how to install Microk8s, one of the trending Kubernetes desktop engines at the time of writing. Microk8s is available to install via the Snap Store. We will use Debian 12 as the base operating system on our test computer, so we can install Microk8s from the Snap Store. If you don’t have `snapd` installed, you will have to first proceed and install it. You will have to use the following command to install the Snap daemon:

```
sudo apt install snapd
```

If you already have `snapd` installed, you can skip this first step. The following step will be to run the command to install the `snapd` core runtime environment needed to run the Snap Store:

```
sudo snap install core
```

Only after `snap` is installed, you can install `microk8s` by using the following command:

```
sudo snap install microk8s --classic
```

A successful installation of Microk8s should yield the following result:

![Figure 16.2 – Installing Microk8s on Linux](img/B19682_16_02.jpg)

Figure 16.2 – Installing Microk8s on Linux

As we already had `snapd` installed, we did not run the first of the commands listed previously.

To access the Microk8s CLI without `sudo` permissions, you’ll have to add the local user account to the `microk8s` group and also fix the permissions on the `~/.kube` directory with the following commands:

```
sudo usermod -aG microk8s $USER
sudo chown -f -R $USER ~/.kube
```

The changes will take effect on the next login, and you can use the `microk8s` command-line utility with invocations that are not `sudo`. For example, the following command displays the help for the tool:

```
microk8s help
```

To get the status of the local single-node Microk8s Kubernetes cluster, we run the following command:

```
microk8s status
```

As you can see, the installation steps of Microk8s on Debian are straightforward and similar to the ones used in Ubuntu.

In the next section, we will show you how to install Microk8s on a VM. This time, we will use Ubuntu 22.04 LTS as the base operating system for the VM.

## Installing Kubernetes on VMs

In this section, we’ll get closer to a real-world Kubernetes environment – though at a much smaller scale – by deploying a Kubernetes cluster on Ubuntu VMs. You can use any hypervisor, such as KVM, Oracle VirtualBox, or VMware Fusion. We will use KVM as our preferred hypervisor.

We will create four VMs, and we’ll provision each VM with 2 vCPU cores, 2 GB RAM, and 20 GB disk capacity. You may follow the steps described in the *Installing Ubuntu* section of [*Chapter 1*](B19682_01.xhtml#_idTextAnchor030), *Installing Linux*, using your hypervisor of choice.

Before we dive into the Kubernetes cluster installation details, let’s take a quick look at our lab environment.

### Preparing the lab environment

Here are the specs of our VM environment:

*   **Hypervisor**: VMware Fusion
*   **Kubernetes cluster**: One CP node and three worker nodes
*   `k8s-cp1`: `192.168.122.104`

*   `k8s-n1`: `192.168.122.146`*   `k8s-n2`: `192.168.122.233`*   `k8s-n3`: `192.168.122.163`*   **VMs**: Ubuntu Server 22.04.3 LTS, 2 vCPUs, 2 GB RAM, 20 GB disk*   `packt` (on all nodes), with SSH access enabled

We set the username and hostname settings on each VM node on the Ubuntu Server installation wizard. Also, make sure to enable the OpenSSH server when prompted. Your VM IP addresses would most probably be different from those in the specs, but that shouldn’t matter. You may also choose to use static IP addresses for your VMs.

To make hostname resolution simple within the cluster, edit the `/etc/hosts` file on each node and add the related records. For example, we have the following `/etc/hosts` file on the CP node (`k8s-cp1`):

![Figure 16.3 – The /etc/hosts file on the CP node (k8s-cp1)](img/B19682_16_03.jpg)

Figure 16.3 – The /etc/hosts file on the CP node (k8s-cp1)

In production environments, with the firewall enabled on the cluster nodes, we have to make sure that the following rules are configured for accepting network traffic within the cluster (according to [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/):](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/):)

![Figure 16.4 – The ports used by the Kubernetes cluster nodes](img/B19682_16_04.jpg)

Figure 16.4 – The ports used by the Kubernetes cluster nodes

The following sections assume that you have the VMs provisioned and running according to the preceding specs. You may take some initial snapshots of your VMs before proceeding with the next steps. If anything goes wrong with the installation, you can revert to the initial state and start again.

Here are the steps we’ll follow to install the Kubernetes cluster:

1.  Disable swapping.
2.  Install `containerd.`
3.  Install the `kubelet`, `kubeadm`, and Kubernetes packages.

We’ll have to perform these steps on each cluster node. The related commands are also captured in the accompanying chapter source code on GitHub.

Let’s start with the first step and disable the memory swap on each node.

### Disable swapping

`swap` is a disk space used when the memory is full (refer to [https://github.com/kubernetes/kubernetes/issues/53533](https://github.com/kubernetes/kubernetes/issues/53533) for more details). The Kubernetes kubelet package doesn’t work with `swap` enabled on Linux platforms. This means that we will have to disable `swap` on all the nodes.

To disable `swap` immediately, we run the following command on each VM:

```
sudo swapoff -a
```

To persist the disabled `swap` with system reboots, we need to comment out the `swap`-related entries in `/etc/fstab`. You can do this either manually, by editing `/etc/fstab`, or with the following command:

```
sudo sed -i '/\s*swap\s*/s/^\(.*\)$/# \1/g' /etc/fstab
```

You may want to double-check that all `swap` entries in `/etc/fstab` are disabled:

```
cat /etc/fstab
```

We can see the `swap` mount point commented out in our `/``etc/fstab` file:

![Figure 16.5 – Disabling swap entries in /etc/fstab](img/B19682_16_05.jpg)

Figure 16.5 – Disabling swap entries in /etc/fstab

Remember to run the preceding commands on each node in the cluster. Next, we’ll look at installing the Kubernetes container runtime.

### Installing containerd

`containerd` is the default container runtime in recent versions of Kubernetes. `containerd` implements the CRI required by the Kubernetes container engine abstraction layer. The related installation procedure is not straightforward, and we’ll follow the steps described in the official Kubernetes documentation in the following link at the time of this writing: [https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/). These steps may change at any time, so please make sure to check the latest procedure. The container runtime needs to be installed on each node of the cluster. We will proceed by installing the needed components on the CP node, the VM called `k8s-cp1`, and then on the other nodes as well.

We’ll start by installing some `containerd` prerequisites:

1.  First, we enable the `br_netfilter` and `overlay` kernel modules using `modprobe`:

    ```
    sudo modprobe br_netfilter
    sudo modprobe overlay
    ```

2.  We also ensure that these modules are loaded upon system reboots:

    ```
    cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
    br_netfilter
    overlay
    sysctl parameters, also persisted across system reboots:

    ```
    cat <<EOF | sudo tee /etc/sysctl.d/containerd.conf
    net.bridge.bridge-nf-call-iptables = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward = 1
    EOF
    ```

    ```

3.  We want the preceding changes to take effect immediately, without a system reboot:

    ```
    sudo sysctl --system
    ```

    Here is a screenshot showing the preceding commands:

![Figure 16.6 – Setting containerd prerequisites](img/B19682_16_06.jpg)

Figure 16.6 – Setting containerd prerequisites

1.  Next, we will verify if specific system variables are enabled in the `sysctl` configuration by running the following command:

    ```
    sudo sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
    ```

    The output of the command should be as follows:

![Figure 16.7 – Verifying system variables in sysctl configuration](img/B19682_16_07.jpg)

Figure 16.7 – Verifying system variables in sysctl configuration

Each variable should have a value of `1` in the output, just as shown in the previous screenshot.

1.  Now, let’s make sure the `apt` repository is up to date before installing any new packages:

    ```
    containerd:

    ```
    containerd configuration:

    ```
    sudo mkdir -p /etc/containerd
    config.toml inside it. The output of the command is too large to show, but it will show you on the screen the automatically generated contents of the new file we created.
    ```

    ```

    ```

2.  We need to slightly alter the default `containerd` configuration to use the `systemd` `cgroup` driver with the container runtime (`runc`). This change is required because the underlying platform (Ubuntu in our case) uses `systemd` as the Service manager. Open the `/etc/containerd/config.toml` file with your editor of choice, such as the following:

    ```
    [plugins] section of the file):

    ```
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    ```

    ```

3.  Then, add the highlighted lines, adjusting the appropriate indentation (this is *very* important):

    ```
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      ...
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
      SystemdCgroup = true
    ```

    Here’s the resulting configuration stub:

![Figure 16.8 – Modifying the containerd configuration](img/B19682_16_08.jpg)

Figure 16.8 – Modifying the containerd configuration

1.  Save the `/etc/containerd/config.toml` file and restart `containerd`:

    ```
    containerd Service, by using the following command:

    ```
    sudo systemctl status containerd
    ```

    The output should show that the system is running and there are no issues.
    ```

With `containerd` installed and configured, we can proceed with the installation of the Kubernetes packages next.

### Installing Kubernetes packages

To install the Kubernetes packages, we’ll follow the steps described at [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/). This procedure may also change over time, so please make sure to check out the latest instructions. The steps presented next are applicable to Debian 12 and Ubuntu 22.04\. Let’s begin:

1.  We first install the packages required by the Kubernetes `apt` repository:

    ```
    apt repository GNU Privacy Guard (GPG) public signing key (for the latest Kubernetes 1.28 at the time of writing):

    ```
    apt repository to our system:

    ```
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

    ```

    ```

2.  Let’s read the packages available in the new repository we just added:

    ```
    sudo apt update -y
    ```

3.  We’re now ready to install the Kubernetes packages:

    ```
    apt-mark hold command to pin the version of the Kubernetes packages, including containerd:

    ```
    containerd and kubelet Services are enabled upon system startup (reboot):

    ```
    sudo systemctl enable containerd
    containerd Service first:

    ```
    containerd should be active and running. The following is a screenshot showing the output of the preceding commands:
    ```

    ```

    ```

    ```

![Figure 16.9 – Pinning the Kubernetes packages and the running status of containerd](img/B19682_16_09.jpg)

Figure 16.9 – Pinning the Kubernetes packages and the running status of containerd

1.  Next, let’s check the status of the `kubelet` Service:

    ```
    exited:
    ```

![Figure 16.10 – The kubelet crashing without cluster configuration](img/B19682_16_10.jpg)

Figure 16.10 – The kubelet crashing without cluster configuration

As shown in the preceding screenshot, `kubelet` is looking for the Kubernetes cluster, which is not set up yet. We can see that `kubelet` attempts to start and activate itself but keeps crashing, as it cannot locate the required configuration.

Important note

Please install the required Kubernetes packages on *all* cluster nodes following the previous steps before proceeding with the next section.

Next, we’ll bootstrap (initialize) the Kubernetes cluster using `kubeadm`.

### Introducing kubeadm

`kubeadm` is a helper tool for creating a Kubernetes cluster and essentially has two invocations:

*   `kubeadm init`: This bootstraps or initializes a Kubernetes cluster
*   `kubeadm join`: This adds a node to a Kubernetes cluster

The default invocation of `kubadm init [flags]` – with no flags – performs the following tasks:

1.  `kubeadm init` ensures that we have the minimum system resources in terms of CPU and memory, the required user permissions, and a supported CRI-compliant container runtime. If any of these checks fail, `kubeadm init` stops the execution of creating the cluster. If the checks succeed, `kubeadm` proceeds to the next step.
2.  `kubeadm init` creates a self-signed CA used by Kubernetes to generate the certificates required to authenticate and run trusted workloads within the cluster. The CA files are stored in the `/etc/kubernetes/pki`/ directory and are distributed on each node upon joining the cluster.
3.  `kubeadm init` creates a default set of kubeconfig files required to bootstrap the cluster. The kubeconfig files are stored in the `/``etc/kubernetes/` directory.
4.  `kubelet` daemon. Examples of static pods are the API server, the controller manager, scheduler, and etcd. Static pod manifests are configuration files describing the CP pods. `kubeadm init` generates the static pod manifests during the cluster bootstrapping process. The manifest files are stored in the `/etc/kubernetes/manifests/` directory. The `kubelet` Service monitors this location and, when it finds a manifest, deploys the corresponding static pod.
5.  `kubelet` daemon deploys the static pods, `kubeadm` queries `kubelet` for the static pods’ state. When the static pods are up and running, `kubeadm init` proceeds with the next stage.
6.  `kubeadm init` follows the Kubernetes best practice of tainting the CP to avoid user pods running on the CP node. The obvious reason is to preserve CP resources exclusively for system-specific workloads.
7.  `kubeadm init` generates a bootstrap token that can be shared with a trusted node to join the cluster.
8.  `kubeadm init` creates and deploys the *DNS* and *kube-proxy* add-on pods.

The stages of a Kubernetes cluster’s bootstrapping process are highly customizable. `kubeadm init`, when invoked without additional parameters, runs all the tasks in the preceding order. Alternatively, a system administrator may invoke the `kubeadm` command with different option parameters to control and run any of the stages mentioned.

For more information about `kubeadm`, please refer to the utility’s help with the following command:

```
kubeadm help
```

For more information about bootstrapping a Kubernetes cluster using `kubeadm`, including installing, troubleshooting, and customizing components, you may refer to the official Kubernetes documentation at [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/).

In the next section, we’ll bootstrap a Kubernetes cluster using `kubeadm` to generate a cluster configuration file and then invoke `kubeadm init` to use this configuration. We’ll bootstrap our cluster by creating the Kubernetes CP node next.

### Creating a Kubernetes CP node

In order to create the CP node, we will use a networking and security solution named `kubeadm`, and afterwards, we will use different tools to apply the configurations. The choice of Calico is purely subjective, but it is needed for managing communications between workloads and components. For more information about Calico, please visit the following link: [https://docs.tigera.io/calico/latest/about/](https://docs.tigera.io/calico/latest/about/).

The commands are performed on the `k8s-cp1` host in our VM environment. As the hostname suggests, we choose `k8s-cp1` as the CP node of our Kubernetes cluster. Now, let’s get to work and configure our Kubernetes CP node:

1.  We’ll start by downloading the Calico manifest for **overlay networking**. The overlay network – also known as **software-defined network** (**SDN**) – is a logical networking layer that accommodates a secure and seamless network communication between the pods over a physical network that may not be accessible for configuration. Exploring the internals of cluster networking is beyond the scope of this chapter, but we encourage you to read more at [https://kubernetes.io/docs/concepts/cluster-administration/networking/](https://kubernetes.io/docs/concepts/cluster-administration/networking/). You’ll also find references to the Calico networking add-on. To download the related manifest, we run the following command:

    ```
    calico.yaml file in the current directory (/home/packt/) that we’ll use with kubectl to configure pod networking later in the process.
    ```

2.  Next, let’s open the `calico.yaml` file using a text editor and look for the following lines (starting with *line 3672*):

    ```
    # - name: CALICO_IPV4POOL_CIDR
    CALICO_IPV4POOL_CIDR points to the network range associated with the pods. If the related subnet conflicts in any way with your local environment, you’ll have to change it here. We’ll leave the setting as is.
    ```

3.  Next, we’ll create a default cluster configuration file using `kubeadm`. The cluster configuration file describes the settings of the Kubernetes cluster we’re building. Let’s name this file `k8s-config.yaml`:

    ```
    k8s-config.yaml file we just generated and mention a few changes that we’ll have to make. We will open it using the localAPIEndpoint.advertiseAddress configuration parameter – the IP address of the API server endpoint. The default value is 1.2.3.4, and we need to change it to the IP address of the VM running the CP node (k8s-cp1), in our case, 192.168.122.104. Refer to the *Preparing the lab environment* section earlier in this chapter. You’ll have to enter the IP address matching your environment:
    ```

![Figure 16.11 – Modifying the advertiseAddress configuration parameter](img/B19682_16_11.jpg)

Figure 16.11 – Modifying the advertiseAddress configuration parameter

1.  The next change we need to make is pointing the `nodeRegistration.criSocket` configuration parameter to the `containerd` socket `(/run/containerd/containerd.sock`) and the name (`k8s-cp1`):

![Figure 16.12 – Changing the criSocket configuration parameter](img/B19682_16_12.jpg)

Figure 16.12 – Changing the criSocket configuration parameter

1.  Next, we change the `kubernetesVersion` parameter to match the version of our Kubernetes environment:

![Figure 16.13 – Changing the kubernetesVersion parameter](img/B19682_16_13.jpg)

Figure 16.13 – Changing the kubernetesVersion parameter

The default value is `1.28.0`, but our Kubernetes version, using the following command, is `1.28.2`:

```
kubeadm version
```

The output is as follows:

![Figure 16.14 – Retrieving the current version of Kubernetes](img/B19682_16_14.jpg)

Figure 16.14 – Retrieving the current version of Kubernetes

1.  Our final modification of the cluster configuration file sets the `cgroup` driver of `kubelet` to `systemd`, matching the `cgroup` driver of `containerd`. Please note that `systemd` is the underlying platform’s Service manager (in Ubuntu), hence the need to yield related Service control to the Kubernetes daemons. The corresponding configuration block is not yet present in `k8s-config.yaml`. We can add it manually to the end of the file or with the following command:

    ```
    cat <<EOF | cat >> k8s-config.yaml
    ---
    apiVersion: kubelet.config.k8s.io/v1beta1
    kind: KubeletConfiguration
    cgroupDriver: systemd
    kubeadm init command with the --config option pointing to the cluster configuration file (k8s-config.yaml), and with the --cri-socket option parameter pointing to the containerd socket:

    ```
    sudo kubeadm init --config=k8s-config.yaml
    ```

    The preceding command takes a couple of minutes to run. A successful bootstrap of the Kubernetes cluster completes with the following output:
    ```

![Figure 16.15 – Successfully bootstrapping the Kubernetes cluster](img/B19682_16_15.jpg)

Figure 16.15 – Successfully bootstrapping the Kubernetes cluster

At this point, our Kubernetes CP node is up and running. In the output, we highlighted the relevant excerpts for the following commands:

*   A successful message (**1**)
*   Configuring the current user as the Kubernetes cluster administrator (**2**)
*   Joining new nodes to the Kubernetes cluster (**3**)

We recommend taking the time to go over the complete output and identify the related information for each of the `kubeadm init` tasks, as captured in the *Introducing kubeadm* section earlier in this chapter.

1.  Next, to configure the current user as the Kubernetes cluster administrator, we run the following commands:

    ```
    mkdir -p ~/.kube
    sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
    sudo chown $(id -u):$(id -g) ~/.kube/config
    ```

2.  With our cluster up and running, let’s deploy the Calico networking manifest to create the pod network:

    ```
    kubectl command to list all the pods in the system:

    ```
    kubectl get pods --all-namespaces
    ```

    The command yields the following output:
    ```

![Figure 16.16 – Retrieving the pods in the Kubernetes cluster](img/B19682_16_16.jpg)

Figure 16.16 – Retrieving the pods in the Kubernetes cluster

The `--all-namespaces` option retrieves the pods across all resource groups in the cluster. Kubernetes uses **namespaces** to organize resources. For now, the only pods running in our cluster are **system pods**, as we haven’t deployed any **user** **pods** yet.

1.  The following command retrieves the current nodes in the cluster:

    ```
    k8s-cp1 as the only node configured in the Kubernetes cluster, running as a CP node:
    ```

![Figure 16.17 – Listing the current nodes in the Kubernetes cluster](img/B19682_16_17.jpg)

Figure 16.17 – Listing the current nodes in the Kubernetes cluster

1.  You may recall that prior to bootstrapping the Kubernetes cluster, the `kubelet` Service was continually crashing (and attempting to restart). With the cluster up and running, the status of the `kubelet` daemon should be `active` and `running`:

    ```
    sudo systemctl status kubelet
    ```

    The output shows the following:

![Figure 16.18 – A healthy kubelet in the cluster](img/B19682_16_18.jpg)

Figure 16.18 – A healthy kubelet in the cluster

1.  We encourage you to check out the manifests created in the `/etc/kubernetes/manifests/` directory for each cluster component using the following command:

    ```
    ls /etc/kubernetes/manifests/
    ```

    The output shows the configuration files describing the static (system) pods, corresponding to the API server, controller manager, scheduler, and etcd:

![Figure 16.19 – The static pod configuration files in /etc/kubernetes/manifests/](img/B19682_16_19.jpg)

Figure 16.19 – The static pod configuration files in /etc/kubernetes/manifests/

1.  You may also look at the kubeconfig files in `/etc/kubernetes`:

    ```
    ls /etc/kubernetes/
    ```

    As you may recall from the *Introducing kubeadm* section earlier in this chapter, the kubeconfig files are used by the cluster components to communicate and authenticate with the API server.

As we have used the `kubectl` utility quite extensively in this section, you can visit the official documentation to find out more about the commands and options available for it at the following link: [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands).

Next, let’s add the worker nodes to our Kubernetes cluster.

### Joining a node to a Kubernetes cluster

As previously noted, before adding a node to the Kubernetes cluster, you’ll need to run the preliminary steps described in the *Preparing the lab environment* section earlier in this chapter.

To join a node to the cluster, we’ll need both the `kubeadm join` command were provided in the output at the end of the bootstrapping process with `kubeadm init`. Refer to the *Creating a Kubernetes CP node* section earlier in this chapter. Keep in mind that the bootstrap token expires in 24 hours. If you forget to copy the command, you can retrieve the related information by running the following commands in the CP node’s terminal (on `k8s-cp1`).

To proceed, follow these steps:

1.  Retrieve the current bootstrap tokens:

    ```
    abcdef.0123456789abcdef):
    ```

![Figure 16.20 – Getting the current bootstrap tokens](img/B19682_16_20.jpg)

Figure 16.20 – Getting the current bootstrap tokens

1.  Get the CA certificate hash:

    ```
    openssl x509 -pubkey \
        -in /etc/kubernetes/pki/ca.crt | \
        openssl rsa -pubin -outform der 2>/dev/null | \
        openssl dgst -sha256 -hex | sed 's/^.* //'
    ```

    The output is as follows:

![Figure 16.21 – Getting the CA certificate hash](img/B19682_16_21.jpg)

Figure 16.21 – Getting the CA certificate hash

1.  You may also generate a new bootstrap token with the following command:

    ```
    kubeadm join command with the required parameters:

    ```
    kubeadm token create --print-join-command
    ```

    ```

Now that the token has been created, we can proceed to the next bootstrapping steps.

In the following steps, we’ll use our initial tokens as displayed in the output at the end of the bootstrapping process. So, let’s switch to the node’s command-line terminal (on `k8s-n1`) and run the following command:

1.  Make sure to invoke `sudo`, or the command will fail with insufficient permissions:

    ```
    sudo kubeadm join 192.168.122.104:6443 \
        --token abcdef.0123456789abcdef \
    k8s-cp1):

    ```
    k8s-n1) added to the cluster:
    ```

    ```

![Figure 16.22 – The new node (k8s-n1) added to the cluster](img/B19682_16_22.jpg)

Figure 16.22 – The new node (k8s-n1) added to the cluster

1.  We encourage you to repeat the process of joining the other two cluster nodes (`k8s-n2` and `k8s-n3`). During the join, while the CP pods are being deployed on the new node, you may temporarily see a `NotReady` status for the new node if you query the nodes on the CP node (`k8s-cp1`) too fast. The process should take a while. In the end, we should have all three nodes showing `Ready` in the output of the `kubectl get nodes` command (on `k8s-cp1`):

![Figure 16.23 – The Kubernetes cluster with all nodes running](img/B19682_16_23.jpg)

Figure 16.23 – The Kubernetes cluster with all nodes running

We have now completed the installation of our Kubernetes cluster, with a CP node and three worker nodes. We used a local (on-premises) VM environment, but the same process would also apply to a hosted IaaS solution running in a private or public cloud.

In the next section, we’ll explore the `kubectl` CLI to a certain extent and use it to create and manage Kubernetes resources. Then, we’ll look at deploying and scaling applications using the imperative and declarative deployment models in Kubernetes.

# Working with Kubernetes

In this section, we’ll use real-world examples of interacting with a Kubernetes cluster. Since we’ll be using the `kubectl` CLI to a considerable extent, we’re going to take a deep dive into some of its more common usage patterns. Then, we will turn our focus to deploying applications to a Kubernetes cluster. We’ll be using the on-premises environment we built in the *Installing Kubernetes on* *VMs* section.

Let’s start by taking a closer look at `kubectl` and its usage.

## Using kubectl

`kubectl` is the primary tool for managing a Kubernetes cluster and its resources. `kubectl` communicates with the cluster’s API server endpoint using the Kubernetes REST API. The general syntax of a `kubectl` command is as follows:

```
kubectl [command] [TYPE] [NAME] [flags]
```

In general, `kubectl` commands execute **CRUD operations** – CRUD stands for **Create**, **Read**, **Update**, and **Delete** – against Kubernetes resources, such as pods, Deployments, and Services.

One of the essential features of `kubectl` is the command output format, either in YAML, JSON, or plain text. The output format is handy when creating or editing application Deployment manifests. We can capture the YAML output of a `kubectl` command (such as create a resource) to a file. Later, we can reuse the manifest file to perform the same operation (or sequence of operations) in a declarative way. This brings us to the two basic Deployment paradigms of Kubernetes:

*   `kubectl` commands to operate on specific resources
*   `kubectl apply` command, usually targeting a set of resources with a single invocation

We’ll look at these two Deployment models more closely in the *Deploying applications* section later in this chapter. For now, let’s get back to exploring the `kubectl` command further. Here’s a short list of some of the most common `kubectl` commands:

*   `create`, `apply`: These create resources imperatively/declaratively
*   `get`: This reads resources
*   `edit`, `set`: These update resources or specific features of objects
*   `delete`: This deletes resources
*   `run`: This starts a pod
*   `exec`: This executes a command in a pod container
*   `describe`: This displays detailed information about resources
*   `explain`: This provides resource-related documentation
*   `logs`: This shows the logs in pod containers

A couple of frequently used parameters of the `kubectl` command are also worth mentioning:

*   `--dry-run`: This runs the command without modifying the system state while still providing the output as if it executed normally
*   `--output`: This specifies various formats for the command output: `yaml`, `json`, and `wide` (additional information in plain text)

In the following sections, we’ll look at multiple examples of using the `kubectl` command. Always keep in mind the general pattern of the command:

![Figure 16.24 – The general usage pattern of kubectl](img/B19682_16_24.jpg)

Figure 16.24 – The general usage pattern of kubectl

We recommend that you check out the complete `kubectl` command reference at [https://kubernetes.io/docs/reference/kubectl/overview/](https://kubernetes.io/docs/reference/kubectl/overview/). While you are becoming proficient with `kubectl`, you may also want to keep the related cheat sheet at hand, which you can find at the following link: [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

Now, let’s prepare our `kubectl` environment to interact with the Kubernetes cluster we built earlier with VMs. You may skip the next section if you prefer to use `kubectl` on the CP node.

### Connecting to a Kubernetes cluster from a local machine

In this section, we configure the `kubectl` CLI running locally on our Linux desktop to control a remote Kubernetes cluster. We are running Debian 12 on our local machine.

First, we will need to install `kubectl` on our system. We will observe the installation instructions at [https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/). We will download the latest `kubectl` release with the following command:

```
curl -LO "kubectl with the following command:

```
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

 Now that the package is installed, we can test the installation using the following command:

```
kubectl version --client
```

 The output of the preceding commands is shown in the following screenshot:
![Figure 16.25 – Installing kubectl locally on a Debian system](img/B19682_16_25.jpg)

Figure 16.25 – Installing kubectl locally on a Debian system
We want to add (merge) yet another cluster configuration to our environment. This time, we connect to an on-premises Kubernetes CP, and we’ll use `kubectl` to update kubeconfig. Here are the steps we’ll be taking:

1.  We first copy kubeconfig from the CP node (`k8s-cp1`, `192.168.122.104`) to a temporary location `(/tmp/config.cp`):

    ```
    scp packt@192.168.122.104:~/.kube/config /tmp/config.cp
    ```

     2.  Finally, we can move the new kubeconfig file to the new location:

    ```
    mv /tmp/config.cp ~/.kube/config
    ```

     3.  Optionally, we can clean up the temporary files created in the process:

    ```
    rm ~/.kube/config.old /tmp/config.cp
    ```

     4.  Let’s get a view of the current kubeconfig contexts:

    ```
    kubernetes-admin@kubernetes) and cluster name (kubernetes):
    ```

![Figure 16.26 – The new kubeconfig contexts](img/B19682_16_26.jpg)

Figure 16.26 – The new kubeconfig contexts

1.  For consistency, let’s change the on-premises cluster’s context name to `k8s-local` and make it the default context in our `kubectl` environment:

    ```
    kubectl config rename-context \
        kubernetes-admin@kubernetes \
        k8s-local
    kubectl context becomes k8s-local, and we’re now interacting with our on-premises Kubernetes cluster (kubernetes). The output is shown in the following screenshot:
    ```

![Figure 16.27 – The current context set to the on-premises Kubernetes cluster](img/B19682_16_27.jpg)

Figure 16.27 – The current context set to the on-premises Kubernetes cluster
Next, we look at some of the most common `kubectl` commands used with everyday Kubernetes administration tasks.
Working with kubectl
One of the first commands we run when connected to a Kubernetes cluster is the following:

```
kubectl cluster-info
```

 The command shows the IP address and port of the API server listening on the CP node, among other information:
![Figure 16.28 – The Kubernetes cluster information shown](img/B19682_16_28.jpg)

Figure 16.28 – The Kubernetes cluster information shown
The `cluster-info` command can also help to debug and diagnose cluster-related issues:

```
kubectl cluster-info dump
```

 To get a detailed view of the cluster nodes, we run the following command:

```
kubectl get nodes --output=wide
```

 The `--output=wide` (or `-o wide`) flag yields detailed information about cluster nodes. The output in the following illustration has been cropped to show it more clearly:
![Figure 16.29 – Getting detailed information about cluster nodes](img/B19682_16_29.jpg)

Figure 16.29 – Getting detailed information about cluster nodes
The following command retrieves the pods running in the default namespace:

```
kubectl get pods
```

 As of now, we don’t have any user pods running, and the command returns the following:

```
No resources found in default namespace.
```

 To list all the pods, we append the `--all-namespaces` flag to the preceding command:

```
kubectl get pods --all-namespace
```

 The output shows all pods running in the system. Since these are exclusively system pods, they are associated with the `kube-system` namespace:
![Figure 16.30 – Getting all pods in the system](img/B19682_16_30.jpg)

Figure 16.30 – Getting all pods in the system
We would get the same output if we specified `kube-system` with the `--``namespace` flag:

```
kubectl get pods --namespace kube-system
```

 For a comprehensive view of all resources running in the system, we run the following command:

```
kubectl get all --all-namespaces
```

 So far, we have only mentioned some of the more common object types, such as nodes, pods, and Services. There are many others, and we can view them with the following command:

```
kubectl api-resources
```

 The output includes the name of the API object types (such as `nodes`), their short name or alias (such as `no`), and whether they can be organized in namespaces (such as `false`):
![Figure 16.31 – Getting all API object types (cropped)](img/B19682_16_31.jpg)

Figure 16.31 – Getting all API object types (cropped)
Suppose you want to find out more about specific API objects, such as `nodes`. Here’s where the `explain` command comes in handy:

```
kubectl explain nodes
```

 The output is as follows:
![Figure 16.32 – Showing nodes’ detailed information (cropped)](img/B19682_16_32.jpg)

Figure 16.32 – Showing nodes’ detailed information (cropped)
The output provides detailed documentation about the `nodes` API object type, including the related API fields. One of the API fields is `apiVersion`, describing the versioning schema of an object. You may view the related documentation with the following command:

```
kubectl explain nodes.apiVersion
```

 The output is as follows:
![Figure 16.33 – Details about the apiVersion field (cropped)](img/B19682_16_33.jpg)

Figure 16.33 – Details about the apiVersion field (cropped)
We encourage you to use the `explain` command to learn about the various Kubernetes API object types in a cluster. Please note that the `explain` command provides documentation about *resource types*. It should not be confused with the `describe` command, which shows detailed information about the *resources* in the system.
The following commands display cluster node-related information about *all* nodes, and then node `k8s-n1` in particular:

```
kubectl describe nodes
kubectl describe nodes k8s-n1
```

 For every `kubectl` command, you can invoke `--help` (or `-h`) to get context-specific help. Here are a few examples:

```
kubectl --help
kubectl config -h
kubectl get pods -h
```

 The `kubectl` CLI is relatively rich in commands, and becoming proficient with it may take some time. Occasionally, you may find yourself looking for a specific command or remembering its correct spelling or use. The `auto-complete` bash for `kubectl` comes to the rescue. We’ll show you how to enable this next.
Enabling kubectl autocompletion
With `kubectl` autocompletion, you’ll get context-sensitive suggestions when you hit the *Tab* key twice while typing the `kubectl` commands.
The `kubectl` autocompletion feature depends on `bash-completion`. Most Linux platforms have `bash-completion` enabled by default. Otherwise, you’ll have to install the related package manually. On Ubuntu, for example, you install it with the following command:

```
sudo apt-get install -y bash-completion
```

 Next, you need to source the `kubectl` autocompletion in your shell (or similar) profile:

```
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

 The changes will take effect on your next login to the terminal or immediately if you source the `bash` profile:

```
source ~/.bashrc
```

 With the `kubectl` autocomplete active, you’ll get context-sensitive suggestions when you hit the *Tab* key twice while typing the command. For example, the following sequence provides all the available resources when you try to create one:

```
kubectl create [Tab][Tab]
```

 When typing the `kubectl create` command and pressing the *Tab* key twice, the result will be a list of resources available for the command:
![Figure 16.34 – Autocompletion use with kubectl](img/B19682_16_34.jpg)

Figure 16.34 – Autocompletion use with kubectl
The `kubectl` autocompletion reaches every part of the syntax: command, resource (type and name), and flags.
Now that we know more about using the `kubectl` command, it’s time to turn our attention to deploying applications in Kubernetes.
Deploying applications
When we introduced the `kubectl` command and its usage pattern at the beginning of the *Using kubectl* section, we touched upon the two ways of creating application resources in Kubernetes: imperative and declarative.
We’ll look at both of these models closely in this section while deploying a simple web application. Let’s start with the imperative model first.
Working with imperative Deployments
As a quick refresher, with imperative Deployments, we follow a sequence of `kubectl` commands to create the required resources and get to the cluster’s desired state, such as running the application. Declarative Deployments accomplish the same, usually with a single `kubectl` `apply` command using a manifest file describing multiple resources.
Creating a Deployment
Let’s begin by creating a Deployment first. We’ll name our Deployment `packt`, based on a demo Nginx container we’re pulling from the public Docker registry (`docker.io/nginxdemos/hello`):

```
kubectl create deployment packt --image=nginxdemos/hello
```

 The command output shows that our Deployment was created successfully:

```
deployment.apps/packt created
```

 We just created a Deployment with a ReplicaSet containing a single pod running a web server application. We should note that our application is managed by the controller manager within an app Deployment stack (`deployment.apps`). Alternatively, we could just deploy a simple application pod (`packt-web`) with the following command:

```
kubectl run pod/packt-web), is not part of a Deployment:

```
pod/packt-web created
```

 We’ll see later in this section that this pod is not part of a ReplicaSet and so is not managed by the controller manager.
Let’s look at the state of our system by querying the pods for detailed information:

```
kubectl get pods -o wide
```

 Let’s analyze the output:
![Figure 16.35 – Getting the application pods with detailed information](img/B19682_16_35.jpg)

Figure 16.35 – Getting the application pods with detailed information
In the preceding output, you can see the series of commands described, and we can also see that our pods are up and running and that Kubernetes deployed them on separate nodes:

*   `packt-579bb9c999-rtvzr`: On cluster node `k8s-n1`
*   `packt-web`: On cluster node `k8s-n3`

Running the pods on different nodes is due to internal load balancing and resource distribution in the Kubernetes cluster.
The application pod managed by the controller is `packt-579bb9c999-rtvzr`. Kubernetes generates a unique name for our managed pod by appending a `579bb9c999`) and a `rtvzr`) to the name of the Deployment (`packt`). The pod template hash and pod ID are unique within a ReplicaSet.
In contrast, the standalone pod (`packt-web`) is left as is since it’s not part of an application Deployment. Let’s describe both pods to obtain more information about them. We’ll start with the managed pod first. Don’t forget to use the `kubectl` autocompletion (by pressing the *Tab* key twice):

```
kubectl describe pod packt-5dc77bb9bf-bnzsc
```

 The related output is relatively large. Here are some relevant snippets:
![Figure 16.36 – Pod information](img/B19682_16_36.jpg)

Figure 16.36 – Pod information
In contrast, the same command for the standalone pod (`packt-web`) would be slightly different without featuring the `Controlled` `By` field:

```
kubectl describe pod packt-web
```

 Here are the corresponding excerpts:
![Figure 16.37 – Relevant pod information](img/B19682_16_37.jpg)

Figure 16.37 – Relevant pod information
You can also venture out to any of the cluster nodes where our pods are running and take a closer look at the related containers. Let’s take node `k8s-n3` (`192.168.122.163`), for example, where our standalone pod (`packt-web`) is running. We’ll SSH into the node’s terminal first:

```
ssh packt@192.168.122.163
```

 Then we’ll use the `containerd` runtime to query the containers in the system:

```
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps
```

 The output shows the following:
![Figure 16.38 – Getting the containers running on a cluster node](img/B19682_16_38.jpg)

Figure 16.38 – Getting the containers running on a cluster node
Next, we’ll show you how to access processes running inside the pods.
Accessing processes in pods
Let’s switch back to our local (on our local machine, not the VM) `kubectl` environment and run the following command to access the shell in the container running the `packt-web` pod:

```
kubectl exec -it packt-web -- /bin/sh
```

 The command takes us inside the container to an interactive shell prompt. Here, we can run commands as if we were logged in to the `packt-web` host using the terminal. The interactive session is produced using the `-it` option – interactive terminal – or `--``interactive --tty`.
Let’s run a few commands, starting with the process explorer:

```
ps aux
```

 Here’s a relevant excerpt from the output, showing the processes running inside the `packt-web` container, and some commands running inside it:
![Figure 16.39 – The processes running inside the packt-web container](img/B19682_16_39.jpg)

Figure 16.39 – The processes running inside the packt-web container
We can also retrieve the IP address with the following command:

```
ifconfig | grep 'inet addr:' | cut -d: -f2 | awk '{print $1}' | grep -v '127.0.0.1'
```

 The output shows the pod’s IP address (as seen in *Figure 16**.39*):

```
172.16.57.193
```

 We can also retrieve the hostname with the following command:

```
hostname
```

 The output shows the pod name (as seen in *Figure 16**.39*):

```
packt-web
```

 Let’s leave the container shell with the `exit` command (as shown in *Figure 16**.39*) or by pressing *Ctrl* + *D*. With the `kubectl` `exec` command, we can run any process inside a pod, assuming that the related process exists.
We’ll experiment next by testing the `packt-web` application pod using `curl`. We should note that, at this time, the only way to access the web server endpoint of `packt-web` is via its internal IP address. Previously, we used the `kubectl` `get pods -o wide` and `describe` commands to retrieve detailed information regarding pods, including the pod’s IP address. You can also use the following one-liner to retrieve the pod’s IP:

```
kubectl get pods packt-web -o jsonpath='{.status.podIP}{"\n"}'
```

 In our case, the command returns `172.16.57.193`. We used the `-o jsonpath` output option to specify the JSON query for a specific field, `{.status.podIP}`. Remember that the pod’s IP is only accessible within the pod network (`172.16.0.0/16`) inside the cluster. Following is a screenshot showing the output:
![Figure 16.40 – Testing the application pod](img/B19682_16_40.jpg)

Figure 16.40 – Testing the application pod
Consequently, we need to probe the `packt-web` endpoint using a `curl` command that has originated within the pod network. An easy way to accomplish such a task is to run a test pod with the `curl` utility installed:

1.  The following command runs a pod named `test`, based on the `curlimages/curl` Docker image:

    ```
    sleep command due to the Docker entry point of the corresponding image, which simply runs a curl command and then exits. Without sleep, the pod would keep coming up and crashing. With the sleep command, we delay the execution of the curl entry point to prevent the exit. The output is shown in the following screenshot:
    ```

![Figure 16.41 – Running a test with curl on the pod](img/B19682_16_41.jpg)

Figure 16.41 – Running a test with curl on the pod

1.  Now, we can run a simple `curl` command using the `test` pod targeting the `packt-web` web server endpoint:

    ```
    kubectl exec test -- curl http://172.16.57.193
    ```

     2.  We’ll get an HTTP response and a corresponding **access log trace** (from the Nginx server running in the pod) accounting for the request. A snippet of the output is as follows:

![Figure 16.42 – The response of running the curl test](img/B19682_16_42.jpg)

Figure 16.42 – The response of running the curl test

1.  To view the logs on the `packt-web` pod, we run the following command:

    ```
    kubectl logs packt-web
    ```

    The output is as follows:

![Figure 16.43 – Logs for packt-web pod](img/B19682_16_43.jpg)

Figure 16.43 – Logs for packt-web pod

1.  The logs in the `packt-web` pod are produced by Nginx and redirected to `stdout` and `stderr`. We can easily verify this with the following command:

    ```
    kubectl exec packt-web -- ls -la /var/log/nginx
    ```

    The output shows the related symlinks:

![Figure 16.44 – Related symlinks](img/B19682_16_44.jpg)

Figure 16.44 – Related symlinks

1.  When you’re done using the `test` pod, you can delete it with the following command:

    ```
    kubectl delete pods test
    ```

Now that we have deployed our first application inside a Kubernetes cluster, let’s look at how to expose the related endpoint to the world. Next, we will expose Deployments via a Service.
Exposing Deployments as Services
Now, let’s rewind to the command we used previously to create the `packt` Deployment. Don’t run it. Here it is just as a refresher:

```
kubectl create deployment packt --image=nginxdemos/hello
```

 The command carried out the following sequence:

1.  It created a Deployment (`packt`).
2.  The Deployment created a ReplicaSet (`packt-579bb9c999`).
3.  The ReplicaSet created the pod (`packt-579bb9c999-rtvzr`).

We can verify that with the following commands:

```
kubectl get deployments -l app=packt
kubectl get replicasets -l app=packt
kubectl get pods -l app=packt
```

 In the preceding commands, we used the `--label-columns (-l)` flag to filter results by the `app=packt` label, denoting the `packt` Deployment’s resources.
We encourage you to take a closer look at each of these resources using the `kubectl describe` command. Don’t forget to use the `kubectl` autocomplete feature when typing in the commands:

```
kubectl describe deployment packt | more
kubectl describe replicaset packt | more
kubectl describe pod packt-5dc77bb9bf-bnzsc | more
```

 The `kubectl` `describe` command could be very resourceful when troubleshooting applications or pod Deployments. Look inside the *Events* section in the related output for clues on pods failing to start, errors, if any, and possibly understand what went wrong.
Now that we have deployed our first application inside a Kubernetes cluster, let’s look at how to expose the related endpoint to the world.
So far, we have deployed an application (`packt`) with a single pod (`packt-579bb9c999-rtvzr`) running an Nginx web server listening on port `80`. As we explained earlier, at this time, we can only access the pod within the pod network, which is internal to the cluster. In this section, we’ll show you how to expose the application (or Deployment) to be accessible from the outside world. Kubernetes uses the Service API object, consisting of a **proxy** and a **selector** routing the network traffic to application pods in a Deployment. To proceed, you can follow these steps:

1.  The following command creates a Service for our Deployment (`packt`):

    ```
    kubectl expose deployment packt \
        --port=80 \
        --target-port=80 \
    --type=NodePort flag, the Service type would be ClusterIP by default, and the Service endpoint would only be accessible within the cluster.
    ```

     2.  Let’s take a closer look at our Service (`packt`):

    ```
    10.105.111.243) and the ports the Service is listening on for TCP traffic (80:32664/TCP):*   port `80`: Within the cluster*   port `32664`: Outside the cluster, on any of the nodesWe should note that the cluster IP is only accessible within the cluster and not from the outside:
    ```

![Figure 16.45 – The service exposing the packt deployment](img/B19682_16_45.jpg)

Figure 16.45 – The Service exposing the packt Deployment
Also, `EXTERNAL-IP` (`<none>`) should not be mistaken for the cluster node’s IP address where our Service is accessible. The external IP is usually a load balancer IP address configured by a cloud provider hosting the Kubernetes cluster (configurable via the `--``external-ip` flag).

1.  We should now be able to access our application outside the cluster by pointing a browser to any of the cluster nodes on port `32664`. To get a list of our cluster nodes with their respective IP addresses and hostnames, we can run the following command:

    ```
    kubectl get nodes -o jsonpath='{range .items[*]}{.status.addresses[*].address}{"\n"}'
    ```

    The output is as follows:

![Figure 16.46 – List of cluster nodes](img/B19682_16_46.jpg)

Figure 16.46 – List of cluster nodes

1.  Let’s choose the CP node (`192.168.122.104/k8s-cp1`) and enter the following address in a browser: http://192.168.122.104:32664.

    The web request from the browser is directed to the Service endpoint (`packt`), which routes the related network packets to the application pod (`packt-579bb9c999-rtvzr`). The `packt` web application responds with a simple Nginx `172.16.215.65`) and name (`packt-579bb9c999-rtvzr`):

![Figure 16.47 – Accessing the packt application service](img/B19682_16_47.jpg)

Figure 16.47 – Accessing the packt application Service

1.  To verify that the information on the web page is accurate, you may run the following `kubectl` command, retrieving similar information:

    ```
    kubectl get pod packt-579bb9c999-rtvzr -o jsonpath='{.status.podIP}{"\n"}{.metadata.name}{"\n"}'
    ```

    The output of the command will be the internal IP address and the name of the pod, as shown in the following:

![Figure 16.48 – Verify the information with the kubectl command](img/B19682_16_48.jpg)

Figure 16.48 – Verify the information with the kubectl command
Suppose we have high traffic targeting our application, and we’d like to scale out the ReplicaSet controlling our pods. We’ll show you how to accomplish this task in the next section.
Scaling application Deployments
Currently, we have a single pod in the `packt` Deployment. In order to scale the application Deployments, we have to obtain information about the running replicas, to scale them to the desired number and test it. Here are the steps to take:

1.  To retrieve the relevant details about the number of running replicas, we run the following command:

    ```
    kubectl describe deployment packt
    ```

    The relevant excerpt in the output is as follows:

![Figure 16.49 – Pod details](img/B19682_16_49.jpg)

Figure 16.49 – Pod details

1.  Let’s scale up our `packt` Deployment to 10 replicas with the following command:

    ```
    packt Deployment, we’ll see 10 pods running:

    ```
    kubectl get pods -l app=packt
    ```

    The output is as follows:
    ```

![Figure 16.50 – Scaling up the deployment replicas](img/B19682_16_50.jpg)

Figure 16.50 – Scaling up the Deployment replicas

1.  Incoming requests to our application Service endpoint (`http://192.168.122.104:32664`) will be load balanced between the pods. To illustrate this behavior, we can either use `curl` or a text-based browser at the command line to avoid the caching-related optimizations of a modern desktop browser. For a better illustration, we’ll use **Lynx**, a simple text-based browser. On our Debian 12 desktop, the package is already installed. You can install it with the following command:

    ```
    sudo apt-get install -y lynx
    ```

     2.  Next, we point Lynx to our application endpoint:

    ```
    lynx 172.16.191.6:32081
    ```

    If we refresh the page with *Ctrl* + *R* every few seconds, we observe that the server address and name change based on the current pod processing the request:

![Figure 16.51 – Load balancing requests across pods](img/B19682_16_51.jpg)

Figure 16.51 – Load balancing requests across pods
You can exit the Lynx browser by typing `Q` and then pressing *Enter*.

1.  We can scale back our Deployment (`packt`) to three replicas (or any other non-zero positive number) with the following command:

    ```
    packt application pods, we can see the surplus pods terminating until only three pods are remaining:

    ```
    kubectl get pods -l app=packt
    ```

    The output is as follows:
    ```

![Figure 16.52 – Scaling back to three pods](img/B19682_16_52.jpg)

Figure 16.52 – Scaling back to three pods

1.  Before concluding our imperative Deployments, let’s clean up all the resources we have created thus far:

    ```
    kubectl delete service packt
    kubectl delete deployment packt
    kubectl delete pod packt-web
    ```

     2.  The following command should reflect a clean slate:

    ```
    kubectl get all
    ```

    The output is as follows:

![Figure 16.53 – The cluster in a default state](img/B19682_16_53.jpg)

Figure 16.53 – The cluster in a default state
In the next section, we’ll look at how to deploy resources and applications declaratively in the Kubernetes cluster.
Working with declarative Deployments
At the heart of a declarative Deployment is a manifest file. Manifest files are generally in YAML format and authoring them usually involves a mix of autogenerated code and manual editing. The manifest is then deployed using the `kubectl` `apply` command:

```
kubectl apply -f MANIFEST
```

 Deploying resources declaratively in Kubernetes involves the following stages:

*   Creating a manifest file
*   Updating the manifest
*   Validating the manifest
*   Deploying the manifest
*   Iterating between the preceding stages

To illustrate the declarative model, we follow the example of deploying a simple Hello World web application to the cluster. The result will be similar to our previous approach of using the imperative method.
So, let’s start by creating a manifest for our Deployment.
Creating a manifest
When we created our `packt` Deployment imperatively, we used the following command (don’t run it just yet!):

```
kubectl create deployment packt --image=nginxdemos/hello
```

 The following command will simulate the same process without changing the system state:

```
kubectl create deployment packt --image=nginxdemos/hello \
    --dry-run=client --output=yaml
```

 We used the following additional options (flags):

*   `--dry-run=client`: This runs the command in the local `kubectl` environment (*client*) without modifying the system state
*   `--output=yaml`: This formats the command output as YAML

The output of the command is as follows:
![Figure 16.54 – Simulating a manifest creation](img/B19682_16_54.jpg)

Figure 16.54 – Simulating a manifest creation
We can use the previous command’s output to analyze the changes to be made to the system. Then we can redirect it to a file (`packt.yaml`) serving as a draft of our Deployment manifest:

```
kubectl create deployment packt --image=nginxdemos/hello \
    --dry-run=client --output=yaml > packt.yaml. From here, we can edit the file to accommodate more complex configurations. For now, we’ll leave the manifest as is and proceed with the next stage in our declarative Deployment workflow.
Validating a manifest
Before deploying a manifest, we recommend validating the Deployment, especially if you edited the file manually. Editing mistakes can happen, particularly when working with complex YAML files with multiple indentation levels.
The following command validates the `packt.yaml` Deployment manifest:

```
kubectl apply -f packt.yaml --dry-run=client
```

 A successful validation yields the following output:

```
deployment.apps/packt created (dry run)
```

 If there are any errors, we should edit the manifest file and correct them prior to deployment. Our manifest looks good, so let’s go ahead and deploy it.
Deploying a manifest
To deploy the `packt.yaml` manifest, we use the following command:

```
kubectl apply -f packt.yaml
```

 A successful Deployment shows the following message:

```
deployment.apps/packt created
```

 We can check the deployed resources with the following command:

```
kubectl get all -l app=packt
```

 The output shows that the `packt` Deployment resources created declaratively are up and running:
![Figure 16.55 – The deployment resources created declaratively](img/B19682_16_55.jpg)

Figure 16.55 – The Deployment resources created declaratively
Next, we want to expose our Deployment using a Service.
Exposing the Deployment with a Service
We’ll repeat the preceding workflow by creating, validating, and deploying the Service manifest (`packt-svc.yaml`). For brevity, we simply enumerate the related commands:

1.  Create the manifest file (`packt-svc.yaml`) for the Service exposing our Deployment (`packt`):

    ```
    kubectl expose deployment packt \
        --port=80 \
        --target-port=80 \
        --type=NodePort \
        --dry-run=client --output=yaml > packt-svc.yaml
    ```

    We explained the preceding command previously in the *Exposing Deployments as* *Services* section.

     2.  Next, we’ll validate the Service Deployment manifest:

    ```
    kubectl apply -f packt-svc.yaml --dry-run=client
    ```

     3.  If the validation is successful, we deploy the Service manifest:

    ```
    packt resources:

    ```
    packt application resources, including the Service endpoint (service/packt) listening on port 31380:
    ```

    ```

![Figure 16.56 – The packt application resources deployed](img/B19682_16_56.jpg)

Figure 16.56 – The packt application resources deployed

1.  Using a browser, `curl`, or Lynx, we can access our application by targeting any of the cluster nodes on port `31380`. Let’s use the CP node (`k8s-cp1`, `192.168.122.104`) by pointing our browser to `http://192.168.122.104:31380`:

![Figure 16.57 – Accessing the packt application endpoint](img/B19682_16_57.jpg)

Figure 16.57 – Accessing the packt application endpoint
If we want to change the existing configuration of a resource in our application Deployment, we can update the related manifest and redeploy it. In the next section, we’ll modify the Deployment to accommodate a scale-out scenario.
Updating a manifest
Suppose our application is taking a high number of requests, and we’d like to add more pods to our Deployment to handle the traffic. We need to change the `spec.replicas` configuration setting in the `pack.yaml` manifest:

1.  Using your editor of choice, edit the `packt.yaml` file and locate the following configuration section:

    ```
    spec:
      replicas: 1
    ```

    Change the value from `1` to `10` for additional application pods in the ReplicaSet controlled by the `packt` Deployment. The configuration becomes the following:

    ```
    spec:
      replicas: 10
    ```

     2.  Save the manifest file and redeploy with the following command:

    ```
    packt Deployment has been reconfigured:

    ```
    packt resources in the cluster, we should see the new pods up and running:

    ```
    packt Deployment, including the additional pods deployed in the cluster:
    ```

    ```

    ```

![Figure 16.58 – The additional pods added for application scale-out](img/B19682_16_58.jpg)

Figure 16.58 – The additional pods added for application scale-out
We encourage you to test with the scale-out environment and verify the load balancing workload described in the *Scaling application Deployments* section earlier in this chapter.

1.  Let’s scale back our Deployment to three pods, but this time by updating the related manifest on the fly with the following command:

    ```
    kubectl edit deployment packt
    ```

    The command will open our default editor in the system (**vi**) to make the desired change:

![Figure 16.59 – Making deployment changes on the fly](img/B19682_16_59.jpg)

Figure 16.59 – Making Deployment changes on the fly

1.  After saving and exiting the editor, we’ll get a message suggesting that our Deployment (`packt`) has been updated:

    ```
    kubectl edit will not be reflected in the Deployment manifest (packt.yaml). Nevertheless, the related configuration changes are persisted in the cluster (etcd).
    ```

     2.  We can verify our updated Deployment with the help of the following command:

    ```
    kubectl get deployment packt
    ```

    The output now shows only three pods running in our Deployment:

![Figure 16.60 – Showing the number of deployments](img/B19682_16_60.jpg)

Figure 16.60 – Showing the number of Deployments

1.  Before wrapping up, let’s clean up our resources once again with the following commands to bring the cluster back to the default state:

    ```
    kubectl delete service packt
    kubectl delete deployment packt
    ```

We have shown you how to use Kubernetes on bare metal, and in the next section, we will briefly point you to some useful resources for using Kubernetes in the cloud.
Running Kubernetes in the cloud
Managed Kubernetes Services are fairly common among public cloud providers. Amazon **Elastic Kubernetes Service** (**EKS**), **Azure Kubernetes Services** (**AKS**), and **Google Kubernetes Engine** (**GKE**) are the major cloud offerings of Kubernetes at the time of this writing. In this section, we’ll not focus on any of these solutions, but we will provide you with solid resources on how to use Kubernetes in the cloud. For more advanced titles on this subject, please check the *Further reading* section of this chapter.
We should note that we just scratched the surface of deploying and managing Kubernetes clusters. Yet, here we are, at a significant milestone, where we deployed our first Kubernetes clusters on-premises. We have reached the end of this journey here, but we trust that you’ll take it to the next level and further explore the exciting domain of application Deployment and scaling with Kubernetes. Let’s now summarize what we have learned in this chapter.
Summary
We began this chapter with a high-level overview of the Kubernetes architecture and API object model, introducing the most common cluster resources, such as pods, Deployments, and Services. Next, we took on the relatively challenging task of building an on-premises Kubernetes cluster from scratch using VMs. We explored various CLI tools for managing Kubernetes cluster resources on-premises. At the high point of our journey, we focused on deploying and scaling applications in Kubernetes using imperative and declarative Deployment scenarios.
We believe that novice Linux administrators will benefit greatly from the material covered in this chapter and become more knowledgeable in managing resources across hybrid clouds and on-premises distributed environments, deploying applications at scale, and working with CLI tools. We believe that the structured information in this chapter will also help seasoned system administrators refresh some of their knowledge and skills in the areas covered.
It’s been a relatively long chapter, and we barely skimmed the surface of the related field. We encourage you to explore some resources captured in the *Further reading* section and strengthen your knowledge regarding some key areas of Kubernetes environments, such as networking, security, and scale.
In the next chapter, we’ll stay within the application deployment realm and look at **Ansible**, a platform for accelerating application delivery on-premises and in the cloud.
Questions
Here are a few questions for refreshing or pondering upon some of the concepts you’ve learned in this chapter:

1.  Enumerate some of the essential Services of a Kubernetes CP node. How do the worker nodes differ?
2.  What command did we use to bootstrap a Kubernetes cluster?
3.  What is the difference between imperative and declarative Deployments in Kubernetes?
4.  What is the `kubectl` command for deploying a pod? How about the command for creating a Deployment?
5.  What is the `kubectl` command to access the shell within a pod container?
6.  What is the `kubectl` command to query all resources related to a Deployment?
7.  How do you scale out a Deployment in Kubernetes? Can you think of the different ways (commands) in which to accomplish the task?
8.  How do you delete all resources related to a Deployment in Kubernetes?

Further reading
The following resources may help you to consolidate your knowledge of Kubernetes further:

*   Kubernetes documentation online: [https://kubernetes.io/docs/home/](https://kubernetes.io/docs/home/)
*   The `kubectl` cheat sheet: [https://kubernetes.io/docs/reference/kubectl/cheatsheet/](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
*   *Kubernetes and Docker: The Container Masterclass [Video]*, *Cerulean Canvas*, Packt Publishing
*   *Mastering Kubernetes – Third Edition*, Gigi Sayfan, Packt Publishing

The following is a short list of useful links for deploying Kubernetes on Azure, Amazon, and Google:

*   Amazon EKS:
    *   [https://docs.aws.amazon.com/eks/index.html](https://docs.aws.amazon.com/eks/index.html)
    *   [https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html](https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html)
*   AKS:
    *   [https://azure.microsoft.com/en-us/services/kubernetes-service/](https://azure.microsoft.com/en-us/services/kubernetes-service/)
    *   [https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster?tabs=azure-cli](https://learn.microsoft.com/en-us/azure/aks/tutorial-kubernetes-deploy-cluster?tabs=azure-cli)
    *   [https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal?tabs=azure-cli](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-portal?tabs=azure-cli)
*   GKE:
    *   [https://cloud.google.com/kubernetes-engine](https://cloud.google.com/kubernetes-engine)
    *   [https://cloud.google.com/build/docs/deploying-builds/deploy-gke](https://cloud.google.com/build/docs/deploying-builds/deploy-gke)
    *   [https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster](https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster)

```

```

```