# 12

# Managing Containers with Docker

In this chapter, you will learn about one of the most well-known tools for creating and managing containers – **Docker**. The topics in this chapter will prepare you for the future of Linux, as it is the foundation of every modern cloud technology. If you wish to remain up to date in a constantly changing landscape, this chapter will be the essential starting point for your journey.

In this chapter, we’re going to cover the following main topics:

*   Understanding Linux containers
*   Working with Docker
*   Working with **Dockerfiles**
*   Deploying a containerized application with Docker

# Technical requirements

No specific technical requirements are needed, just a working installation of Linux on your system. Both Ubuntu/Debian and Fedora/RHEL are equally suitable for this chapter’s exercises. We will use Debian GNU/Linux 12 for most of our examples, but where appropriate, we will discuss the specifics of Fedora Linux installation and use.

# Understanding Linux containers

As we have demonstrated in the previous chapter, there are two main types of virtualization: **virtual machine** (**VM**)**-based** and **container-based**. We discussed VM-based virtualization in the previous chapter, and now it is time to explain what containers are. At a very basic, conceptual level, containers are similar to VMs. They have similar purposes – allowing an isolated environment to run – but they are different in so many ways that they can hardly be called similar. Let’s compare these two concepts in more detail.

## Comparing containers and VMs

As you already know, a VM emulates the machine’s hardware and uses it as if there were several machines available. By comparison, containers do not replicate the physical machine’s hardware; they do not emulate anything.

A container shares the base OS kernel with shared libraries and binaries needed for certain applications to run. The applications are contained inside the container, isolated from the rest of the system. They also share a network interface with the host to offer similar connectivity to a VM.

Containers run on top of a **container engine**. Container engines offer OS-level virtualization, used to deploy and test applications by using only the requisite libraries and dependencies. This way, containers make sure that applications can run on any machine by providing the same expected behavior as intended by the developer. The following is a visual comparison between containers and VMs:

![Figure 12.1 – Containers versus VMs (general scheme)](img/B19682_12_1.jpg)

Figure 12.1 – Containers versus VMs (general scheme)

As you can see, the containers only use the **userspace**, sharing the underlying OS-level architecture.

Historically speaking, containerization has been around for some time now. With the Unix OS, **chroot** has been the tool used for containerization since 1982.

On Linux, some of the newest and most frequently used tools are **Linux containers** (**LXC**), with **LXD** as a newer and extended version of the former, introduced in 2008, and Docker, introduced in 2013\. Why this LXC/LXD nomenclature? Well, LXC was the first kid on the containers block, with LXD being a newer, redesigned version of it.

In the next section, we will dissect the underlying container technology.

## Understanding the underlying container technology

As mentioned earlier, LXC was one of the earliest forms of containers, introduced 12 years ago. The newer form of containers, and the ones that changed the entire container landscape and started all the DevOps hype (more on this in [*Chapter 14*](B19682_14.xhtml#_idTextAnchor299)), is called Docker. Containers do not abstract the hardware level as hypervisors do. They use a specific userspace interface that benefits from the kernel’s techniques to isolate specific resources. By using Linux containers, you can replicate a default Linux system without using a different kernel, as you would do by using a VM.

What made LXC appealing when it first appeared were the APIs it uses for multiple programming languages, including Python 3, Go, Ruby, and Haskell. So, even though LXC is no longer that popular, it is still worth knowing. Docker has taken the crown and center stage in container engine usage. We will not use LXC/LXD in our examples, but we will still discuss it for backward compatibility purposes. As of the time of writing this book, there are two supported versions of LXC, version 4.0, with support until June 2025, and version 5.0, with support until June 2027.

According to its developers, LXC uses features to create an isolated environment that is as close as possible to a default Linux installation. Among the kernel technologies that it uses, we could bring up the most important one, which is the backbone of any container inside Linux: kernel **namespaces** and **cgroups**. Besides those, there are still chroots and security profiles for both AppArmor and SELinux.

Let’s now explain these basic features that Linux containers use.

### Linux namespaces

What are **Linux namespaces**? In a nutshell, namespaces are kernel global system resources responsible for the isolation that containers provide. Namespaces wrap a global system resource inside an abstraction layer. This process fools any app process that is running inside the namespace into believing that the resource it is using is its own. A namespace provides isolation at a logical level inside the kernel and also provides visibility for any running processes.

To better understand how namespaces work, think of any user on a Linux system and how it can view different system resources and processes. As a user, you can see the global system resources, the running processes, other users, and kernel modules, for example. This amount of transparency could be harmful when wanting to use containers as virtualized environments at the OS level. As it cannot provide the encapsulation and emulation level of a VM, the container engine must overcome this somehow, and the kernel’s low-level mechanisms of virtualization of the environment come in the form of namespaces and cgroups.

There are several types of namespaces inside the Linux kernel, and we will describe them briefly:

*   **Mount**: They restrict visibility for available filesystem mount points within a single namespace so that processes from that namespace have visibility of the filesystem list; processes can have their own root filesystem and different private or shared mounts
*   **Unix Time Sharing** (**UTS**): This isolates the system’s hostname and domain name
*   **Interprocess Communication** (**IPC**): This allows processes to have their own IPC shared memory, queues, and semaphores
*   **Process Identification**: This allows mapping of **process IDs** (**PIDs**) with the possibility of a process with PID 1 (the root of the process tree) to spin off a new tree with its own root process; processes inside a PID namespace only see the processes inside the same PID namespace
*   **Network**: Abstraction at the network protocol level; processes inside a network namespace have a private network stack with private network interfaces, routing tables, sockets, and iptables rules
*   **User**: This allows mapping of UID and GID, including root UID 0 as a non-privileged user
*   **cgroup**: A cgroup namespace process can see filesystem paths relative to the root of the namespace

The namespaces can be viewed by using the `lsns` command in Linux. The following is an excerpt from the command’s output:

![Figure 12.2 – Using lsns to view the available namespaces](img/B19682_12_2.jpg)

Figure 12.2 – Using lsns to view the available namespaces

In the following section, we will break down cgroups as the second major building block of containers.

### Linux cgroups

What are cgroups? Their name comes from **control groups**, and they are kernel features that restrict and manage resource allocation to processes. Cgroups control how memory, CPU, I/O, and network are used. They provide a mechanism that determines specific sets of tasks that limit how many resources a process can use. They are based on the concept of **hierarchies**. Every child group will inherit the attributes of its parent group, and multiple cgroups hierarchies can exist at the same time in one system.

Cgroups and namespaces combined are creating the isolation that containers are built upon. By using cgroups and namespaces, resources are allocated and managed for each container separately. Compared to VMs, containers are lightweight and run as isolated entities.

As stated earlier, there are two types of containers used, LXC and Docker. As we have already discussed LXC, let us see in the following section what Docker is.

## Understanding Docker

Docker, similar to LXC/LXD, is based, among other technologies, on kernel namespaces and cgroups. Docker is a platform that is used for developing and shipping applications. The Docker platform provides the underlying infrastructure for containers to operate securely. Docker containers are lightweight entities that run directly on the host’s kernel. The platform offers features such as tools to create and manage isolated, containerized applications. Thus, the container is the base unit used for application development, testing, and distribution. When apps are production-ready and fit for deployment, they can be shipped as containers or as orchestrated services (we will discuss orchestration in [*Chapter 16*](B19682_16.xhtml#_idTextAnchor342), *Deploying Applications**with Kubernetes*).

In the following diagram, we will show you how the Docker architecture works:

![Figure 12.3 – Docker architecture](img/B19682_12_3.jpg)

Figure 12.3 – Docker architecture

Let’s explain the preceding diagram. Docker uses both namespaces and cgroups available in the Linux kernel, and is split into two major components:

*   `runc` and `containerd` to *Cloud Native Computing Foundation* so that more organizations would be able to contribute to both. The following is a diagram showing the details of the Docker architecture, with the core components, the Docker engine and the container runtime, being shown in detail:

![Figure 12.4 – Docker architecture details](img/B19682_12_4.jpg)

Figure 12.4 – Docker architecture details

*   The **Docker engine**: This engine is what is split into the **dockerd** daemon, the **API** interface, and the **command-line interface** (**CLI**). The Docker engine comprises the API interface and the dockerd daemon, while the container runtime has two main components – the **containerd** daemon and **runc** for namespaces and cgroups management. Besides the components listed in the previous point, a number of other components are used to run and deploy Docker containers. Docker has a client-server architecture and the workflow involves a **host**, or server daemon, a **client**, and a **registry**. The host consists of images and containers (downloaded from the registry), and the client provides the commands needed to manage containers.

    The workflow of these components is as follows:

    *   The dockerd daemon listens for API requests to manage services and objects (such as images, containers, networks, and volumes).
    *   The client is the way for users to interact with the daemon through the API.
    *   The registries store images, and Docker Hub is a public registry for anyone to use freely. In addition to this, there are private registries that can be used.

    The following is a graphical representation of Docker’s workflow, showing the client component, the API, and the daemon:

![Figure 12.5 – Docker workflow](img/B19682_12_5.jpg)

Figure 12.5 – Docker workflow

Docker may seem difficult, even disarming, to a beginner. All the different components that work together, all those new typologies, and specific workflows are complicated. Do you feel like you know how Docker works just after reading this section? Of course not. The process of learning Docker has just begun. Having a strong foundation on which to build your Docker knowledge is extremely important. This is why, in the next section, we will show you how to use Docker.

# Working with Docker

We will use Debian GNU/Linux 12 for this section’s exercises, installed on a VM with 2 vCPUs and 2 GB of RAM as a host. But before we start installing Docker, let’s go into a little detail about how Docker, as an entity, operates, to help us identify which version to choose for our use case.

## Which Docker version to choose?

In order for the business to be viable, the corporation behind Docker (the Docker corporation) offers a series of products, all revolving around their primary product, Docker. In the past, it had two different products available, the Docker **Community Edition** (**CE**) and the Docker **Enterprise Edition** (**EE**). Out of these two, only the EE version was responsible for the revenue of Docker.

Recently, the portfolio evolved to different products and offerings, such as **Docker Personal**, **Docker Pro**, **Docker Team**, and **Docker Business**. Among those, only Docker Personal is free to use; the other three products are subscription-based. Docker Personal is suitable for individual developers, education, and open source communities, but has some limitations (which can be seen here: [https://www.docker.com/products/personal/](https://www.docker.com/products/personal/)). Using Docker Personal implies the existence of a Docker user account, and it includes Docker Desktop for Linux (and all other major platforms) and Docker Engine for servers.

On the server side, Docker Engine is available for `.deb` and `.rpm` package formats and is available for Ubuntu, Debian, Fedora, and CentOS. For our examples, we will use Docker Personal. We will show you how to install Docker in the next section.

## Installing Docker

Depending on the version of your preferred Linux distribution that you choose, the package available inside the official repository may be out of date. Nevertheless, you have two options: one is to use the official package from our Linux distribution’s own repository, and the other is to download the latest available version from the official Docker website.

As we are using a fresh system, with no prior Docker installation, we will not need to worry about older versions of the software and possible incompatibilities with the new versions. We will use Docker’s `apt` repository, which will ensure that our package versions will always be up to date. Remember that we are using a Debian host.

The procedure to install Docker is as follows:

1.  First, add the requisite certificates for the Docker repository using the following commands:

    ```
    sudo apt update -y && sudo apt install ca-certificates curl gnupg
    ```

2.  In order to use the official Docker repository, you will need to add the Docker GPG key. For this, use the following commands:

    ```
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    ```

3.  Set up the repository needed to install the version of Docker we want:

    ```
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

4.  The next logical step is to update the repository list. When you do this, you should see the official Docker repository. Use the following command:

    ```
    sudo apt update -y
    ```

5.  Install the Docker packages. In our case, we will install the latest packages available using the following command:

    ```
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

6.  To verify that you installed the packages from the official Docker repository and not the ones from the Debian repositories, run the following command:

    ```
    apt-cache policy docker-ce
    ```

    If the output shows the source from the [docker.com](http://docker.com) website, this means that the source repository is the official Docker one:

![Figure 12.6 – Verifying the source repository](img/B19682_12_6.jpg)

Figure 12.6 – Verifying the source repository

1.  Check the status of the Docker daemon. It should be started right after installation:

    ```
    docker is created. In order to be able to use Docker, your user should be added to the docker group. The existing groups in Linux are inside the /etc/group file. You can list the last lines (new groups are appended at the end of the file) to see the docker group as the last one created:

    ```
    tail /etc/group
    ```

    You can either add your existing user or create a new one. We will add our already existing user. Add the user with the following command:

    ```
    sudo usermod -aG docker ${USER}
    ```

    ```

2.  After you add the user, log out and back in again and check whether you were added to the new group with the following command:

    ```
    groups
    ```

3.  You have completed the installation of Docker. Now you can enable the Docker daemon to begin at system startup:

    ```
    sudo systemctl enable docker
    ```

Installing Docker is only the first step. Now let’s explore what we can do with it. In the following section, you will learn about the commands available in Docker.

## Using some Docker commands

Working with Docker means using its CLI. It has a significant number of sub-commands available. If you want to see them all, you should run the `docker -–help` command. There are two main command groups shown:

*   The first group shows the management commands
*   The second group shows the regular commands

We will not discuss all the commands in this section. We will only focus on the ones that you will need to get started with Docker.

Before learning anything about the commands, let’s first perform a test to see whether the installation is working. We will use the `docker run` command to check whether we can access Docker Hub and run containers. Our testing command will be the following:

```
sudo docker run hello-world
```

This command downloads an image from Docker Hub and runs it as a container. The following is a screenshot of the output:

![Figure 12.7 – Running the first docker run command](img/B19682_12_7.jpg)

Figure 12.7 – Running the first docker run command

The preceding screenshot is self-explanatory and a nice touch from the Docker team. It explains what the command did in the background using clear and easy-to-understand language. By running the `docker run` command, you both learn about the workflow and the success of the installation. Also, it is one of the basic Docker commands that you will use relatively often.

Let’s now dig deeper and search for other images available on Docker Hub. Let’s search for an Ubuntu image to run containers on. To search for the image, we will use the `docker` `search` command:

```
docker search ubuntu
```

The output of the command should list all the Ubuntu images available inside Docker Hub:

![Figure 12.8 – Searching for the Ubuntu image](img/B19682_12_8.jpg)

Figure 12.8 – Searching for the Ubuntu image

As you can see, the output has five columns:

*   **NAME**: The first column shows the image’s name
*   **DESCRIPTION**: The second column shows the description, which is a short text providing information about a specific image
*   **STARS**: The third column shows the number of stars it has (representing popularity based on user opinion)
*   **OFFICIAL**: The fourth column shows whether that image is an official one supported by the company behind the distribution/software
*   **AUTOMATED**: The fifth column shows whether the image has automated scripts

Once you find the image you are looking for, you can download it onto your system using the `docker pull` command. Let us download the first image from the list shown in the preceding screenshot, the one called `ubuntu`. We will use the following command:

```
docker pull ubuntu
```

With this command, the `ubuntu` image is downloaded locally onto your computer. Now, containers can be run using this image. To list the images that are already available on your computer, run the `docker` `images` command:

![Figure 12.9 – Running the docker images command](img/B19682_12_9.jpg)

Figure 12.9 – Running the docker images command

Please note the small size of the Ubuntu Docker image. You may be wondering why it is so small. This is because Docker images contain only the base and minimum packages needed to run. This makes the container running on the image extremely efficient in resource usage.

The few commands we showed you in this section are the most basic ones needed to start using Docker. Now that you know how to download an image, let’s show you how to manage Docker containers.

## Managing Docker containers

In this section, we will learn how to run, list, start, stop, and remove Docker containers, and also how to manage networking.

### Running containers

We will use the Ubuntu image that we just downloaded. To run it, we will use the `docker run` command with two arguments, `-i` for interactive output and `-t` for starting a pseudo TTY, which will give us interactive access to the shell:

```
docker run -it ubuntu
```

You will notice that your command prompt will change. Now it will contain the container ID. The user, by default, is the root user. Basically, you are now inside an Ubuntu image, so you can use it exactly as you would use any Ubuntu command line. You can update the repository, install the requisite applications, remove unnecessary apps, and so on. Any changes that you make to the container image stay inside the container. To exit the container, simply type `exit`. Now, we will show you how to list containers, but before we do that, we should ask you not to close the terminal in which the Ubuntu-based container is currently running.

### Listing containers

You can open a new terminal on your system and check to see how many Docker containers are actively running, using the `docker` `ps` command.

In the command’s output, you will see the ID of the container that is running in the other terminal. There are also details about the command that runs inside the container and the creation time.

There are a couple of arguments that you can use with the `docker` `ps` command:

*   If you want to see all active and inactive containers, use the `docker ps -``a` command
*   If you want to see the latest created container, use the `docker ps -``l` command

The following is the output of all three variants of the `docker` `ps` command:

![Figure 12.10 – Listing containers with the docker ps command](img/B19682_12_10.jpg)

Figure 12.10 – Listing containers with the docker ps command

In the output, you will also see names assigned to containers, such as `amazing_hopper` or `recursing_murdock`. Those are random names automatically given to containers by the daemon. Now, we will learn how to start, stop, and remove running containers.

### Starting, stopping, and removing running containers

When managing containers, such as starting and stopping, you can refer to them by using the container ID or the name assigned by Docker. Let’s now show you how to start, stop, and remove a container.

To start a Docker container, use the `docker start` command, followed by the name or ID of the container. Here is an example:

```
docker start amazing_hopper
```

To stop a container, use the `docker stop` command, followed by the name of the ID of the container. Here is an example:

```
docker stop amazing_hopper
```

In our case, the Ubuntu container, named `amazing_hopper`, is already running, so the `start` command will not do anything. But the `stop` command will stop the container. After stopping, if you run the `docker ps` command, there will be no more containers in the list.

Let’s take a look at the output of both these commands:

![Figure 12.11 – Starting and stopping containers](img/B19682_12_11.jpg)

Figure 12.11 – Starting and stopping containers

To remove a container, you can use the `docker rm` command. For example, if we would like to remove the initial `hello-world` container (also called `recursing_murdock` in our case), we will use the following command:

```
docker rm recursing_murdock
```

Once you remove the container, any changes that you made and you did not save (commit) will be lost. Let us show you how to commit changes you made in a container to the Docker image. This means that you will save a specific state of a container as a new Docker image.

Important note

Please take into consideration that removing a container will not erase the existing image downloaded from Docker Hub.

Let’s say that you would like to develop, test, and deploy a Python application on Ubuntu. The default Docker image of Ubuntu doesn’t have Python installed.

Next, we will show you how to troubleshoot Docker networking and how to commit a new image.

### Docker networking and committing a new image

The scenario for this section’s exercise is that you would like to modify the existing Ubuntu image by installing the Python packages that you need for your application. To do this, we follow these steps:

1.  First, we start the container and check to see whether Python is installed or not:

![Figure 12.12 – Checking for Python inside the container](img/B19682_12_12.jpg)

Figure 12.12 – Checking for Python inside the container

1.  We check for both Python 2 and Python 3, but neither version is installed on the image. As we want to use the latest version of the programming language, we will use the following command to install Python 3 support (running as root):

    ```
    apt install python3
    ```

    In doing so, you will get in contact with Docker networking for the first time, as the container needs to reach out to the official Ubuntu repositories in order to download and install the packages you need. It might be the case for you, too, as it was in ours, that when trying to install Python, you will be greeted with an error, such as the one shown in the following screenshot:

![Figure 12.13 – Error while trying to install Python](img/B19682_12_13.jpg)

Figure 12.13 – Error while trying to install Python

This error shows that the package named `python3` cannot be located, meaning that our container does not have access to the repositories. A quick thought would be that there is something wrong with Docker’s networking. In order to troubleshoot this, we have a useful command called `docker network`. It is used to manage network connections for the Docker container. In our case, the fault for the error message could be a missing connection between the container and the network. In this case, we can investigate first with the `docker network` `ls` command:

```
docker network ls
```

This command will show all the active networks that are used by Docker. In our case, when running the preceding command, we can see that there are three available networks for Docker, each with a network ID and a name assigned to it:

![Figure 12.14 – Showing the available networks](img/B19682_12_14.jpg)

Figure 12.14 – Showing the available networks

Important note

The issue described in this example may not appear in your case. However, it is a good exercise to show you how to use the `docker network` command in action.

1.  Before starting to solve our issue, let us see once again what the containers are that are running and what their given names are. As we started the new container once again, it should have another ID and name. We will use the following command:

    ```
    docker ps command) to the bridge network. This will be done with the following command:

    ```
    docker network connect bridge [container_name]
    ```

    ```

2.  Running the command will not show any kind of output, but we will test the result by running the command to update the repositories inside the container to see that it works:

    ```
    apt update -y
    ```

    The output of the command shows that the repositories are accessible from our container, meaning that we can now install Python. The following screenshot shows an excerpt of the output:

![Figure 12.15 – Proof that the network connection is working](img/B19682_12_15.jpg)

Figure 12.15 – Proof that the network connection is working

By connecting a container to a network, it will be able to communicate with other containers on the same network too.

1.  Now, before going further with our Python installation, we would like to show you that there is a command that we can use to automatically connect a container to a network when starting the container:

    ```
    bridge) and the container’s name. We used the -i option for interactive output, the -t option for opening a pseudo TTY, and the -d option for detaching the container and running in the background.
    ```

2.  Now, let us proceed with our initial Python installation. We can once again run the following command:

    ```
    apt install python3
    ```

    This time, the command will not give any errors and it will proceed with the installation. The following is an excerpt from the command’s output:

![Figure 12.16 – Installing Python packages inside a Docker container](img/B19682_12_16.jpg)

Figure 12.16 – Installing Python packages inside a Docker container

1.  Now, with Python 3 installed and the necessary modifications made to the image used inside the container, we can save the instance of the container to a new Docker image. For this, we will use the following command:

    ```
    -m option to add a comment that details our commit process and the -a option to specify the account user and the ID of the base image we used, in our case, the ID of the running container. The following screenshot shows a series of commands to help you to understand the process better:
    ```

![Figure 12.17 – A new image committed locally](img/B19682_12_17.jpg)

Figure 12.17 – A new image committed locally

As shown in the preceding screenshot, we first used the `docker ps` command to see the ID of the container we are running, then we used the `docker commit` command (with the options described earlier) to save the new image locally. Notice the increased size of the image we just saved. Installing Python 3 more than doubled the size of the initial Ubuntu image. The last command used was `docker images`, to see the existing images, including the one we just created (`ubuntu-python3`).

By now, you have learned how to use extremely basic Docker commands for opening, running, and saving containers. In the next section, we will introduce you to Dockerfiles and the process of building container images.

# Working with Dockerfiles

Before starting to work with Dockerfiles, let’s see what a Dockerfile is. It is a text file that consists of instructions defined by the user for Docker to execute, and respecting some basic structure, such as the following:

```
INSTRUCTION arguments
```

The Dockerfile is mainly used for creating new container images. This file is used by Docker to automatically build images based on the information the user provides inside the file. There are some keywords that define a Dockerfile. Those keywords, which are referred to as instructions, are as follows:

*   `FROM`: This must be the first instruction inside a Dockerfile as it tells Docker what the image is that you build upon
*   `LABEL`: This instruction adds some more information, such as a description, or anything that could help describe the new image you are creating; the use of such instructions needs to be limited
*   `RUN`: This is the instruction that offers direct interaction with the image, the place where commands or scripts to be run inside the image are written
*   `ADD`: This instruction is used to transfer files inside the image; it copies files or directories to the filesystem of the image
*   `COPY`: This instruction is similar to `ADD`, as it is also used to copy files or directories from one source to the image’s filesystem
*   `CMD`: This instruction can occur only once in a Dockerfile, as it provides defaults for an image that is executed
*   `USER`: This instruction is used to set a username that will be used when a command is executed; it can be used on the `RUN` or `CMD` instructions
*   `WORKDIR`: This instruction will set the default working directory for other instructions inside a Dockerfile, such as `RUN`, `CMD`, `COPY`, `ADD`, or `ENTRYPOINT`
*   `ENTRYPOINT`: This instruction is used to configure containers that will run as executables

The instructions listed here are just the ones that are usually used in a Dockerfile, but they do not represent all the instructions available. You can visit [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/) for a complete listing of all the instructions available for a Dockerfile. In the next section, we will show you how to build a container image using a Dockerfile.

## Building container images from Dockerfiles

In this section, we will create a Dockerfile that will be used for building a new Docker container image. Let us present the scenario on which our exercise is built. Similar to the exercise used in the *Docker networking and committing a new image* section, we will prepare an image for the Python programming environment. In order to create the new Docker image, we will first need to create the Dockerfile. The following are the steps to take:

1.  Create a new directory inside your home directory using the following command:

    ```
    mkdir ~/my_docker_images && cd ~/my_docker_images
    ```

2.  Create a new file inside the new directory:

    ```
    docker search debian command, the second image name on the output list will be the official Debian Linux image, called debian. We will use that. The contents of the Dockerfile are shown in the following screenshot:
    ```

![Figure 12.18 – Creating the Dockerfile](img/B19682_12_18.jpg)

Figure 12.18 – Creating the Dockerfile

1.  Now that the Dockerfile is created, we will run the `docker build` command to create the new Docker image. The command we used is as follows:

    ```
    -f option to specify the Dockerfile name and the -t option to specify the name of the image we want to create, in our case pydeb. In the next screenshot, you will see the output of the docker build command, showing all the steps needed to build the image, as specified in the Dockerfile. The build was successful:
    ```

![Figure 12.19 – Building a new custom image from a Dockerfile](img/B19682_12_19.jpg)

Figure 12.19 – Building a new custom image from a Dockerfile

We can verify if the image was created by using the `docker images` command. As shown in the preceding screenshot, the new `pydeb` image was successfully created.

1.  We can use the new image and create a new container with the following command:

    ```
    pydeb images we just created.
    ```

2.  To verify the container running, open a new terminal window and run the `docker ps` command, as shown in the following screenshot:

![Figure 12.20 – New container based on our custom image](img/B19682_12_20.jpg)

Figure 12.20 – New container based on our custom image

By now, you already know enough about Docker to feel comfortable using it in production. In the next section, we will show you how to use Docker to deploy a very basic application. We will make it so simple that the app to deploy will be a basic static presentation website.

# Deploying a containerized application with Docker

So far, we have shown you how to use Docker and how to manage containers. Docker is so much more than that, but this is enough to get you started and make you want to learn more. Docker is a great tool for developers as it offers a streamlined way to deploy applications by removing the necessity to replicate development environments. In the next section, we will show you how to deploy a simple website using Docker.

## Deploying a website using Docker

To deploy a website using Docker, follow these steps:

1.  We will use a free website template randomly downloaded from the internet (the download link is [https://www.free-css.com/free-css-templates/page262/focus)](https://www.free-css.com/free-css-templates/page262/focus)). We will copy the download location from the website and download the file inside our home directory using the `wget` utility:

    ```
    unzip command:

    ```
    docker_webapp inside the ~/my_docker_images directory created in the previous section, and move the extracted file inside of it. Therefore, the new location in our case will be the following:

    ```
    webapp_dockerfile inside our present working directory. The contents of the Dockerfile are as follows:
    ```

    ```

    ```

![Figure 12.21 – Contents of a new Dockerfile](img/B19682_12_21.jpg)

Figure 12.21 – Contents of a new Dockerfile

The file is simple and has only two lines:

*   The first line, using the `FROM` keyword, specifies the base image that we will use, which will be the official NGINX image available on Docker Hub. As you will see in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), NGINX is a widely used type of web server.
*   The second line, using the `COPY` keyword, specifies the location where the contents of our present working directory will be copied inside the new container.

The following action builds the Docker image using the `docker` `build` command:

```
docker build ~/my_docker_images/docker_webapp/focus -f webapp_dockerfile -t webapp
```

1.  The new image was created, so we can now check for it using the `docker images` command. In our case, the output is as follows:

![Figure 12.22 – The new webapp Docker image was created](img/B19682_12_22.jpg)

Figure 12.22 – The new webapp Docker image was created

1.  As the output shows, the new image called `webapp` was created, and we can start a new container using it. As we will need to access the container from the outside, we will need to open specific ports, and we will do that using the `-p` parameter inside the `docker run` command. We can either specify a single port or a range of ports. When specifying ports, we will give the ports for the container and for the host, too. We will use the `-d` parameter to detach the container and run it in the background. The command is as follows:

    ```
    docker run -it -d -p 8080:80 webapp
    ```

    The output is as follows:

![Figure 12.23 – The output of the docker run command](img/B19682_12_23.jpg)

Figure 12.23 – The output of the docker run command

We are exposing host port `8080` to port `80` on the container. We could have used both ports `80`, but on the host, it might be occupied by other services.

1.  You can now access the new containerized application by going to your web browser and typing the local IP address and port `8080` into the address bar. As we are using a VM and not the host, we will point to the VM’s IP address, in our case `192.168.122.48`, followed by the `8080` port. In the next screenshot, you will see our Docker-deployed website:

![Figure 12.24 – Running the ﻿web app in our web browser](img/B19682_12_24.jpg)

Figure 12.24 – Running the web app in our web browser

As you can see in the preceding image, the website is accessible from localhost. For deploying a website on a virtual private server, please visit [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276).

# Summary

In this chapter, we emphasized the importance of containerization. We showed you what containers are, how they work, and why they are so important. Containers are the foundation of the modern DevOps revolution, and you are now ready to use them. We also taught you about Docker, the basic commands for sleek use. You are now ready to start the cloud journey. Virtualization and container technologies are at the heart of cloud and server technologies.

In the next chapter, we will show you how to install and configure different Linux-based servers, such as web servers, DNS servers, DHCP servers, and mail servers.

# Questions

Here’s a brief quiz about some of the essential concepts that were covered in this chapter:

1.  What is the major difference between containers and VMs?

    **Hint**: Revisit *Figure 12**.1*.

2.  How does container technology work?
3.  What are the two major components of Docker architecture?
4.  Which Docker command shows the running containers?
5.  Which command is used for container network management?

    `docker network` command’s help.

# Further reading

For more information on the topics covered in this chapter, you can refer to the following Packt titles:

*   *Docker Quick Start Guide*, Earl Waud
*   *Mastering Docker – Fourth Edition*, Russ McKendrick
*   *Containerization with LXC*, Konstantin Ivanov
*   *A Developer’s Essential Guide to Docker Compose*, Emmanouil Gkatziouras