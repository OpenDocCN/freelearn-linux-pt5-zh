# 15

# Deploying to the Cloud with AWS and Azure

Recent years have seen a significant shift from on-premises computing platforms to private and public clouds. In an ever-changing and accelerating world, deploying and running applications in a highly scalable, efficient, and secure infrastructure is critical for businesses and organizations everywhere. On the other hand, the cost and expertise required to maintain the equivalent level of security and performance with on-premises computing resources become barely justifiable compared to current public cloud offerings. Businesses and teams, small and large, adopt public cloud services in increasing numbers, albeit large enterprises are relatively slow to make a move.

One of the best metaphors for cloud computing is application services *on tap*. Do you need more resources for your apps? Just *turn on the tap* and provision the virtual machines or instances in any number you require (scale horizontally). Or perhaps, for some instances, you require more CPUs or memory (scale vertically). When you no longer need resources, just *turn off* *the tap*.

Public cloud services provide all these functions at relatively low rates, taking away the operations overhead that you might otherwise have with maintaining the on-premises infrastructure accommodating such features.

This chapter will introduce you to **Amazon Web Service** (**AWS**) and **Microsoft Azure** – two major public cloud providers – and offer some practical guidance for deploying your applications in the cloud. In particular, we’ll focus on typical cloud management workloads, using both the web administration console and the command-line interface.

By the end of this chapter, you’ll know how to use the AWS web console and the AWS CLI, as well as the Azure web portal and the Azure CLI, to manage your cloud resources with the two most popular cloud providers of our time. You’ll also learn how to make a prudent decision about creating and launching your resources in the cloud, striking a sensible balance between performance and cost.

We hope that Linux administrators – novice and experienced alike – will find the content in this chapter relevant and refreshing. Our focus is purely practical as we explore the AWS and Azure cloud workloads. We will refrain from comparing the two since such an endeavor would go beyond this chapter’s scope. To make the journey less boring, we’ll also steer away from keeping a perfect symmetry between describing AWS and Azure management tasks. We all know AWS blazed the trail into the public cloud realm first. Other major cloud providers followed, adopted, and occasionally improved the underlying paradigms and workflows. Since we’ll be introducing AWS first, we’ll cover more ground on some cloud provisioning concepts (such as Regions and **Availability Zones** (**AZs**)), which in many ways are very similar to what’s available in Azure.

Finally, we’ll leave the ultimate choice between using AWS or Azure up to you. We’re giving you the map. The road is yours to take.

Here are some of the key topics you’ll learn about:

*   Working with AWS EC2
*   Working with Microsoft Azure

# Technical requirements

To complete the tasks in this chapter, you will require the following:

*   AWS and Azure accounts if you want to follow along with the practical examples. Both cloud providers provide free subscriptions:
    *   AWS free tier: [https://aws.amazon.com/free](https://aws.amazon.com/free)
    *   Microsoft Azure free account: [https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/)
*   A local machine with a Linux distribution of your choice to install and experiment with the AWS CLI and Azure CLI utilities.
*   A modern web browser (such as Google Chrome or Mozilla Firefox) for the web-console-driven management tasks for both AWS and Azure. You can access the related portals on any platform.
*   A Linux command-line terminal and intermediate-level proficiency using the shell to run the AWS and Azure CLI commands.

Important note

As you proceed, please make sure to shut down your EC2 instances or Azure virtual machines after you finish experimenting with them. Failure to do so may result in relatively high bills being generated for you.

With this, let’s start with our first contender, AWS EC2.

# Working with AWS EC2

AWS **Elastic Compute Cloud** (**EC2**) is a scalable computing infrastructure that allows users to lease virtual computing platforms and services to run their cloud applications. AWS EC2 has gained extreme popularity in recent years due to its outstanding performance and scalability combined with relatively cost-effective service plans. This section provides some basic functional knowledge to get you started with deploying and managing AWS EC2 instances running your applications. In particular, we’ll introduce you to EC2 instance types, particularly how you can differentiate between various provisioning and related pricing tiers, how to use SSH to connect and SCP to transfer files to and from your EC2 instances, and how to work with the AWS CLI.

By the end of this section, you’ll have a basic understanding of AWS EC2 and how to choose, deploy, and manage your EC2 instances.

## Introducing and creating AWS EC2 instances

AWS EC2 provides various instance types, each with its provisioning, capacity, pricing, and use case models. Choosing between different EC2 instance types is not always trivial. This section will briefly describe each EC2 instance type, some pros and cons of using them, and how to choose the most cost-effective solution. With each instance type, we’ll show you how to launch one using the AWS console.

We can look at EC2 instance types from two perspectives. When you decide on an EC2 instance, you’ll have to consider both these factors. Let’s look at each of these options briefly:

*   **Provisioning**: The capacity and computing power of your EC2 instance. The main differentiating feature of each EC2 instance provisioning type is the computing power, which is expressed by vCPU (or CPU), memory (RAM), and storage (disk capacity).

    Some EC2 instance types also provide **graphical processing unit** (**GPU**) or **field programmable gate array** (**FPGA**) computing capabilities. A detailed view of EC2 instance provisioning types is beyond the scope of this chapter. You can explore the related information at [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html) or [https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/).

*   `type` and `region`
*   **Spot instances**: These EC2 instances are available for reuse at a lower price than on-demand instances
*   **Dedicated instances**: These EC2 instances run in a **virtual private cloud** (**VPC**) assigned to a single-payer account

We’ll cover each of these EC2 instance types in the following sections. For each of these types, we’ll show an example of how to launch a corresponding instance.

However, before creating an instance, we’ll look at another key concept regarding EC2 instances – **AZs**.

### EC2 AZs

The AWS EC2 service is available in multiple locations around the globe, known as **Regions**. Here are some examples of Regions:

*   `us-west-2`)
*   `ap-south-1`)

EC2 defines multiple AZs in a Region, which are essentially one or more data centers. Regions are entirely isolated from each other in terms of the underlying infrastructure to provide fault tolerance and high availability. If a Region becomes unavailable, only the EC2 instances within the affected Region are unreachable. Other Regions with their EC2 instances will continue to work uninterrupted.

Similarly, AZs are connected while providing highly available and fault-tolerant EC2 services within a Region. Launching an EC2 instance creates it in the current Region selected in the AWS console. An AWS EC2 administrator may switch between different Regions when managing EC2 instances. Only instances within the selected Region are visible in the EC2 administration console. An EC2 administrator will usually choose a Region based on the geographical location of the users accessing the EC2 instance.

Now that we have preliminary knowledge of various EC2 instance types, let’s look at on-demand instances.

### EC2 on-demand instances

AWS EC2 **on-demand instances** use a *pay-as-you-go* pricing model for resource usage per second, without a long-time contract. On-demand instances are best suited for experimenting with uncertain workloads where the resource usage is not fully known (such as during development). The flexibility of these on-demand instances comes with a higher price than reserved instances, for example.

Let’s launch an on-demand instance:

1.  First, we must log into our AWS account console at [https://console.aws.amazon.com](https://console.aws.amazon.com). The following screenshot shows the default console view, where you will have information about the user (**1**), the Region (**2**), and the available – or recently visited – services (**3**):

![Figure 15.1 – The AWS management console](img/B19682_15_01.jpg)

Figure 15.1 – The AWS management console

1.  Next, we must select the EC2 service from the list on the left-hand side, as shown in the preceding screenshot. This will lead us to the EC2 dashboard interface, as follows:

![Figure 15.2 – The EC2 dashboard](img/B19682_15_02.jpg)

Figure 15.2 – The EC2 dashboard

The **Launch instance** button, as shown in the preceding screenshot, will begin the step-by-step process of creating our on-demand instance. This will lead us to a new interface where we can provide the main configuration options on our new instance, such as its name, operating system to use, instance type, login key pair, network, and storage settings.

1.  Next, we need to choose a name and a tag. First, we will choose a name for our instance. In our case, we set the name to `aws_packt_testing_1`:

![Figure 15.3 – Giving a name to the new EC2 instance](img/B19682_15_03.jpg)

Figure 15.3 – Giving a name to the new EC2 instance

Alternatively, we can add a new tag. To do this, we must click `env` and a value of `packt`. We can add multiple tags (key-value pairs) to a given instance if we need to:

![Figure 15.4 – Adding tags to an EC2 instance](img/B19682_15_04.jpg)

Figure 15.4 – Adding tags to an EC2 instance

1.  Next, we must choose an operating system. We’ll select Ubuntu 22.04 LTS as our Linux distribution for the new EC2 instance, as shown in the following screenshot:

![Figure 15.5 – Choosing the operating system for the EC2 instance](img/B19682_15_05.jpg)

Figure 15.5 – Choosing the operating system for the EC2 instance

There are other operating systems to choose from, including macOS, Microsoft Windows, and Linux distributions such as Red Hat Enterprise Linux, SUSE Linux Enterprise, or Debian Linux, alongside Amazon Linux and Ubuntu.

1.  Now, we can choose an instance type. Here, we’ll select the instance type based on our provisioning needs. In our case, we will select the **t2.micro** type, with 1 vCPU and 1 GB of memory:

![Figure 15.6 – Choosing an instance type – the t2.micro](img/B19682_15_06.jpg)

Figure 15.6 – Choosing an instance type – the t2.micro

1.  Under **Network settings**, we will leave the default settings unchanged for now:

![Figure 15.7 – Setting up the network for the new EC2 instance](img/B19682_15_07.jpg)

Figure 15.7 – Setting up the network for the new EC2 instance

1.  The next step is to configure storage. By default, the EC2 instance provides an 8 GB SSD volume, but you can set it up to 30 GB while using the free tier. We will use it with the default 8 GB:

![Figure 15.8 – Setting up storage for the EC2 instance](img/B19682_15_08.jpg)

Figure 15.8 – Setting up storage for the EC2 instance

1.  The last step before launching the new EC2 instance is to review the detailed summary provided and decide if there are any other changes to be made. Once the new instance has been tailored to our needs, we can create it by clicking on the **Launch instance** button, as shown in the following screenshot:

![Figure 15.9 – Summary of the new EC2 instance and the Launch instance button](img/B19682_15_09.jpg)

Figure 15.9 – Summary of the new EC2 instance and the Launch instance button

At this point, we are ready to launch our instance by pressing the **Launch instance** button. Alternatively, we could follow further configuration steps by revisiting the previous steps; otherwise, EC2 will assign some default values.

1.  When launching the EC2 instance, we will be asked to create or select a certificate key pair for remote SSH access into our instance. This will highlight the key pair section on the screen, where we can click on the `packt_aws_key`, select the **RSA** type and the **.pem** file format, and then click on the **Create key pair** button in the lower right corner:

![Figure 15.10 – Selecting or creating a certificate key pair for SSH access](img/B19682_15_10.jpg)

Figure 15.10 – Selecting or creating a certificate key pair for SSH access

You will be automatically asked to download the new `.pem` file to a location on your local machine.

1.  Download the related file (`packt-ec2.pem`) to a secure location on your local machine, where you can use it with the `ssh` command to access your EC2 instance:

    ```
    env: packt tag, we’ll get a view of the EC2 instance we just created:
    ```

![Figure 15.11 – An EC2 instance in the running state](img/B19682_15_11.jpg)

Figure 15.11 – An EC2 instance in the running state

For more information about on-demand instances, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-on-demand-instances.html).

Now that we have learned the basics of launching an on-demand instance, let’s look at reserved instances.

### EC2 reserved instances

With **reserved instances**, we lease the EC2 computing capacity of a specific type for a specific amount of time. This length of time is called a **term** and can either be a 1-year or a 3-year commitment. Here are the main characteristics that need to be set upfront when purchasing reserved instances, as shown in *Figure 15**.12*:

*   **Platform**: An example of this is **Linux**.
*   **Tenancy**: Running on **Default** (shared) or **Dedicated** hardware.
*   **Offering Class**: These are the types of reserved instances that are available. The options are as follows:
    *   **Standard**: A plain reserved instance with a well-defined set of options
    *   **Convertible**: It allows specific changes, such as modifying the instance type (for example, from **t2.large** to **t2.xlarge**)
*   **Instance Type**: An example of this is **t2.large**.
*   **Term**: For example, **1 year.**
*   **Payment option**: **All upfront**, **Partial upfront**, **No upfront**.

With each of these options and the different tiers within, your costs depend on the cloud computing resources involved and the duration of the service. For example, if you choose to pay all upfront, you’ll get a better discount than otherwise. Choosing from among the options previously mentioned is ultimately an exercise in cost-saving and flexibility.

A close analogy to purchasing reserved instances is a *mobile telephone plan* – you decide on all the options you want, and then you commit a certain amount of time. With reserved instances, you get less flexibility in terms of making changes, but with significant savings in cost – sometimes up to 75%, compared to on-demand instances.

To launch a reserved instance, go to your EC2 dashboard in the AWS console and choose **Reserved Instances** under **Instances** in the left panel, then click on the **Purchase Reserved Instances** button. Here is an example of purchasing a reserved EC2 instance:

![Figure 15.12 – Purchasing a reserved EC2 instance](img/B19682_15_12.jpg)

Figure 15.12 – Purchasing a reserved EC2 instance

For more information about EC2 reserved instances, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-reserved-instances.html).

We have learned that reserved instances are a cost-effective alternative to on-demand EC2 instances. Now, let’s take our journey further and look at yet another way to reduce the costs by using spot instances.

### EC2 spot instances

A **spot instance** is an unused instance waiting to be leased. The amount of time a spot instance is vacant and at your disposal depends on the general availability of the requested capacity in EC2, given that the associated costs are not higher than the amount you are willing to pay for your spot instance. AWS advertises spot instances with an up to 90% discount compared to on-demand EC2 pricing.

The major caveat of using spot instances is the potential *no-vacancy* situation when the required capacity is no longer available at the initially agreed-upon rate. In such circumstances, the spot instance will shut down (and perhaps be leased elsewhere). AWS EC2 is kind enough to give a 2-minute warning before stopping the spot instance. This time should be used to properly tear down the application workflows running within the instance.

Spot instances are best suited for non-critical tasks, where application processing could be inadvertently interrupted at any moment and resumed later, without considerable damage or data loss. Such jobs may include data analysis, batch processing, and optional tasks.

To launch a spot instance, go to your **EC2** dashboard and choose **Spot Requests** in the left-hand menu. Under **Instances**, click on **Request Spot Instances** and follow the steps described in the *EC2 on-demand* *instances* section.

A detailed explanation of launching a spot instance is beyond the scope of this chapter. The AWS EC2 console does a great job of describing and assisting with the related options. For more information about spot instances, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-spot-instances.html).

We know that by default, EC2 instances run on shared hardware, meaning that instances owned by multiple AWS customers share the same machine (or virtual machine). But what if you wanted to have a dedicated platform to run your EC2 instances? We’ll look at dedicated instances next.

### EC2 dedicated instances

Specific businesses require applications to run on dedicated hardware without them sharing the platform with anyone. AWS EC2 provides **dedicated hosts** and **dedicated instances** to accommodate this use case. As you may expect, dedicated instances would cost more than other instance types. So, why should we care about leasing such instances?

There are businesses – especially among financial, health, and governmental institutions – required by law to meet strict regulatory requirements for processing sensitive data or to acquire hardware-based licenses for running their applications.

With dedicated instances *without* a dedicated host, EC2 would guarantee that your applications run on a hypervisor exclusively dedicated to you, yet it would not enforce a fixed set of machines or hardware. In other words, some of your instances may run on different physical hosts. Choosing a dedicated host in addition to dedicated instances would always warrant a fully dedicated environment – hypervisors and hosts – for running your applications exclusively, without sharing the underlying platforms with other AWS customers.

To launch a dedicated instance, you can follow these steps:

1.  Start with the same steps that you performed for launching an on-demand EC2 instance that were described earlier in this chapter, in the *EC2 on-demand* *instances* section.
2.  You can add another step to the launching process. To do this, you must open the **Advanced details** drop-down section and scroll to the **Tenancy** option, where you must choose **Dedicated – run a dedicated instance**, as shown in the following screenshot:

![Figure 15.13 – Launching a dedicated EC2 instance](img/B19682_15_13.jpg)

Figure 15.13 – Launching a dedicated EC2 instance

1.  If you want to run your dedicated instance on a dedicated host, you must create a dedicated host first. To do this, on the **EC2** dashboard, navigate to **Instances** | **Dedicated Hosts**:

![Figure 15.14 – Creating a dedicated EC2 host](img/B19682_15_14.jpg)

Figure 15.14 – Creating a dedicated EC2 host

1.  Follow the EC2 wizard to allocate your dedicated host according to your preferences. After creating your host, you may launch your dedicated instance as described previously and choose the **Dedicated host – launch this instance on a dedicated Host** option for **Tenancy**, as shown in *step 2*.

For more information on dedicated hosts, please visit [https://aws.amazon.com/ec2/dedicated-hosts/](https://aws.amazon.com/ec2/dedicated-hosts/). For dedicated instances, see [https://aws.amazon.com/ec2/pricing/dedicated-instances/](https://aws.amazon.com/ec2/pricing/dedicated-instances/).

We’ll conclude our journey through AWS EC2 instance types here. For more information, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Instances.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Instances.html).

Next, we’ll look at one of the two essential EC2 deployment features that allow you to be proficient and resourceful when deploying and scaling your EC2 instances – **Amazon Machine Images** (**AMIs**) and **placement groups**. We will only discuss placement groups in this chapter. For more information about AMIs, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html). Placement groups control how your instances are spread across the EC2 infrastructure for high availability and optimized workloads. We’ll discuss this next.

## Introducing AWS EC2 placement groups

**Placement groups** allow you to specify how your EC2 instances are placed across the underlying EC2 hardware or hypervisors, providing strategies to group or separate instances depending on your requirements. Placement groups are offered free of charge.

There are three types of placement groups to choose from. Let’s quickly go through each of these types and look at their use cases:

*   **Cluster placement groups**: With cluster placement groups, instances are placed within a single AZ (data center). They are best suited for low-latency, high-throughput communication between instances, but not with the outside world. Applications with high-performance computing or data replication would greatly benefit from cluster placement, but web servers, for example, not so much.
*   **Spread placement groups**: When you launch multiple EC2 instances, there’s always a possibility that they may end up running on the same physical machine or hypervisor. This may not be desirable when a single point of failure (such as hardware) would be critical for your applications. Spread placement groups provide hardware isolation between instances. In other words, if you launch multiple instances in a spread placement group, there’s a guarantee that they will run on separate physical machines. In the rare case of an EC2 hardware failure, only one of your instances would be affected.
*   **Partition placement groups**: Partition placement groups will group your instances in logical formations (partitions) with hardware isolation between the partitions, but not at the instance level. We can view this model as a hybrid between the cluster and spread placement groups. When you launch multiple instances within a partition placement group, EC2 will do its best to distribute the instances between partitions evenly. For example, if you had four partitions and 12 instances, EC2 would place three instances in each node (partition). We can look at a partition as a computing unit made of multiple instances. In the case of a hardware failure, the isolated partition instances can still communicate with each other but not across partitions. Partition placement groups support up to seven instances in a single logical partition.

To create a placement group, navigate to **Network & Security** | **Placement Groups** in your **EC2** dashboard’s left menu and click on the **Create Placement Group** button. On the next screen, you must specify a name for the **Placement Group** option, as well as a **Placement Strategy** value. Optionally, you can add tags (key-value pairs) for organizing or identifying your placement group. When you’re done, click the **Create** **group** button:

![Figure 15.15 – Creating a placement group](img/B19682_15_15.jpg)

Figure 15.15 – Creating a placement group

For more information on EC2 placement groups, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html).

Now that we are familiar with various EC2 instance types, let’s look at how to use our instances.

## Using AWS EC2 instances

In this section, we’ll briefly go through some essential operations and management concepts regarding your instances. First, we’ll look at the life cycle of an EC2 instance.

### The life cycle of an EC2 instance

When using or managing EC2 instances, it is important to understand the transitional stages, from launch to running to hibernation, shutdown, or termination. Each of these states affects billing and the way we access our instances:

![Figure 15.16 – The life cycle of an EC2 instance](img/B19682_15_16.jpg)

Figure 15.16 – The life cycle of an EC2 instance

Let’s break this down:

*   The **PENDING** state corresponds to the boot-up and initialization phase of our instance.
*   Transitioning from **PENDING** to **RUNNING** is not always immediate, and it may take a while for the applications running within the instance to become responsive. EC2 starts billing our instance in the **RUNNING** state until transitioning to the **STOPPED** state.
*   In the **RUNNING** state, we can reboot our instance if needed. During the **REBOOTING** state, EC2 always brings back our instance on the same host, whereas stopping and restarting doesn’t always guarantee the same host for the instance.
*   In the **STOPPED** state, we’ll no longer be charged for the instance, but there will be costs related to any additional storage (other than the root volume) attached to the instance.
*   When the instance is no longer required, we may choose between the **STOPPING** or **HIBERNATING** states. With **HIBERNATING**, we avoid the **PENDING** state’s potential latency upon startup. If we no longer use the instance, we may decide to terminate it. Upon termination, there are no more charges related to the instance. When terminating an instance, it may still show up for a little while in the EC2 dashboard before it gets permanently removed.

We can connect to an EC2 instance in a running state using SSH. In the next section, we’ll show you how.

### Connecting to AWS EC2 instances

EC2 instances, in general, serve the purpose of running a specific application or a group of applications. The related platform’s administration and maintenance usually require terminal access. The way to access these instances (or any other service) is determined by how AWS differentiates the available services into two different concepts, similar to the ones used in networking – control plane and data plane. These are concepts that represent how EC2 instances can be accessed:

*   **Control plane (or management plane) access**: Using the AWS EC2 console and the SSH terminal, we perform administrative tasks on EC2 instances.
*   **Data plane access**: Applications running on EC2 instances may also expose their specific endpoints (ports) for communicating with the outside world. EC2 uses security groups to control the related network traffic.

In this section, we’ll briefly look at both control plane and data plane access. In particular, we’ll discuss how to connect to an EC2 instance using SSH and how to use SCP for file transfer with EC2 instances.

#### Connecting via SSH to an EC2 instance

In control plane access mode, using SSH with our EC2 instance allows us to manage it like any on-premises machine on a network. The related SSH command is as follows:

```
ssh -i SSH_KEY ec2-user@EC2_INSTANCE
```

Let’s break this down to understand it better:

*   `SSH_KEY` represents the private key file on our local system that we created and downloaded when launching our instance. See the *EC2 on-demand instances* section for more details.
*   `ec2-user` is the default user assigned by EC2 to our AMI Linux instance. Different AMIs may have different usernames to connect with. You should check with the AMI vendor of your choice about the default username to use with SSH. For example, when using `ec2-user`, when using `ubuntu`, and when using `admin`.
*   `EC2_INSTANCE` represents the public IP address or DNS name of our EC2 instance. You can find this in the EC2 dashboard for your instance:

![Figure 15.17 – The public IP address and DNS name of an EC2 instance](img/B19682_15_17.jpg)

Figure 15.17 – The public IP address and DNS name of an EC2 instance

In our case, the SSH command is as follows:

```
ssh -i packt_aws_key.pem ubuntu@3.68.98.173
```

However, before we connect, we need to set the right permissions for our `private key` file so that it’s not publicly viewable:

```
chmod 400 packt_aws_key.pem
```

Failing to do this results in an unprotected key file error while attempting to connect.

A successful SSH connection to our EC2 instance yields the following output:

![Figure 15.18 – Connecting with SSH to an EC2 instance](img/B19682_15_18.jpg)

Figure 15.18 – Connecting with SSH to an EC2 instance

At this point, we can interact with our EC2 instance as if it were a standard machine.

Next, let’s look at how to transfer files from an EC2 instance.

#### Using SCP for file transfer

To transfer files to and from an EC2 instance in data plane access mode, we must use the `scp` utility. `scp` uses the **Secure Copy Protocol** (**SCP**) to securely transfer files between network hosts.

The following command copies a local file (`README.md`) to our remote EC2 instance (so it must be run from our machine, not the EC2 instance):

```
scp -i packt_aws_key.pem README.md ubuntu@3.68.98.173:~/
```

The file is copied to the `ubuntu` user’s home folder (`/home/ubuntu`) on the EC2 instance. The reverse operation of transferring the `README.md` file from the remote instance to our local directory is as follows:

```
scp -i packt_aws_key.pem ubuntu@3.68.98.173:~/README.md .
```

Note that the `scp` command’s invocation is similar to `ssh`, where we specify the private key file (`aws/packt-ec2.pem`) via the `-i` (identity file) parameter.

Next, we’ll look at yet another critical aspect of managing and scaling EC2 instances – storage volumes.

### Using EC2 storage volumes

**Storage volumes** are device mounts within an EC2 instance that provide additional disk capacity (at extra cost). You may need additional storage for large file caches or extensive logging, for example, or you may choose to mount network-attached storage for critical data shared between your EC2 instances. You can think of EC2 storage volumes as *modular hard drives*. You mount or unmount them based on your needs.

Two types of storage volumes are provided by EC2:

*   **Instance store**
*   **Elastic Block** **Store** (**EBS**)

Knowing how to use storage volumes allows you to make better decisions when scaling your applications as they grow. Let’s look at instance store volumes first.

#### Introducing instance store volumes

Instance store volumes are disks that are directly (physically) attached to your EC2 instance. Consequently, the maximum size and number of instance store volumes you can connect to your instance are limited by the instance type. For example, a storage-optimized *i3* instance can have up to 8 x 1.9 TB SSD disks attached, while a general-purpose *m5d* instance may only grow up to four drives of 900 GB each. See [https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/) for more information on instance capacity.

An instance store volume comes at no additional cost if it’s the root volume – the volume with the operating system platform booting the instance.

Not all EC2 instance types support instance store volumes. For example, the general-purpose *t2* instance types only support EBS storage volumes. On the other hand, if you want to grow your storage beyond the maximum capacity allowed by the instance store, you’ll have to use EBS volumes. We’ll look at EBS volumes next.

#### Introducing EBS volumes

The data on instance store volumes only persists with your EC2 instance. If your instance stops or terminates, or there is a failure with it, all of your data is lost. To store and persist critical data with your EC2 instances, you’ll have to choose EBS.

EBS volumes are flexible and high-performing network-attached storage devices that serve both the root volume system and additional volume mounts on your EC2 instance. An EBS root volume can only be attached to a single EC2 instance at a time while an EC2 instance can have multiple EBS volumes attached at any time. An EBS volume can also be attached to multiple EC2 instances at a time via *multi-attach*. For more information on EBS multi-attach, see [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes-multi.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes-multi.html).

When you create an EBS volume, it will be automatically replicated within the AZ of your instance to minimize latency and data loss. With EBS, you get live monitoring of drive health and stats via Amazon CloudWatch free of charge. EBS also supports encrypted data storage to meet the latest regulatory standards for data encryption.

EC2 storage volumes are backed by Amazon’s **Simple Storage Service** (**S3**) or **Elastic File System** (**EFS**) infrastructure. For more information on the different EC2 storage types, please visit [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Storage.html).

Now, let’s create and configure an EBS storage volume and attach it to our EC2 instance.

#### Configuring an EBS storage volume

Here are the steps we’ll follow, starting with creating the EBS volume:

1.  In the **EC2** dashboard, go to **Volumes** under **Elastic Block Store** in the left navigation pane, and click on the **Create Volume** button at the top:

![Figure 15.19 – Creating an EBS volume](img/B19682_15_19.jpg)

Figure 15.19 – Creating an EBS volume

1.  Enter the values of your choice for **Volume Type**, **Size**, and **Availability Zone**. Make sure you choose the AZ where your EC2 instances are. You could also include a **Snapshot ID** value if you want to restore the volume from a previous EC2 instance backup (snapshot). We’ll look at backup/restore using EBS snapshots later in this chapter.
2.  Press the **Create Volume** button when you’re done. You’ll get a **Volume created successfully** message that specifies your new EBS volume ID if all goes well.
3.  Click on **Volume ID** or select the volume from the left navigation pane, under **Elastic Block Store** and **Volumes**. Click on the **Actions** button and choose **Attach Volume**:

![Figure 15.20 – Attaching the EBS volume to an EC2 instance](img/B19682_15_20.jpg)

Figure 15.20 – Attaching the EBS volume to an EC2 instance

1.  On the next screen, enter your EC2 instance ID (or name tag to search for it) in the **Instance** field:

![Figure 15.21 – Entering the EC2 instance ID to attach the volume](img/B19682_15_21.jpg)

Figure 15.21 – Entering the EC2 instance ID to attach the volume

1.  Press the **Attach volume** button when you’re done. After a few moments, EC2 will initialize your EBS volume, and **State** will change to **in-use**. The volume device is now ready, but we need to format it with a filesystem so that it can be used.
2.  Let’s SSH into the EC2 instance where we attached the volume:

    ```
    lsblk command-line utility to list the block devices:

    ```
    lsblk
    ```

    The output is as follows:
    ```

![Figure 15.22 – The local volumes in our EC2 instance](img/B19682_15_22.jpg)

Figure 15.22 – The local volumes in our EC2 instance

Looking at the size of the volumes, we can immediately tell the one we just added – `xvdf` with `1G`. The other volume (`xvda`) is the original root volume of our `t3.micro` instance.

1.  Now, let’s check if our new EBS volume (`xvdf`) has a filesystem on it:

    ```
    /dev/xvdf: data, meaning that the volume doesn’t have a filesystem yet.
    ```

2.  Let’s build a filesystem on our volume using the `mkfs` (*make filesystem*) command-line utility:

    ```
    -t (--type) parameter with an xfs filesystem type. XFS is a high-performance journaled filesystem that’s supported by most Linux distributions and is installed by default in some of them.
    ```

3.  If we check the filesystem using the following command, we should see the filesystem details displayed instead of empty data:

    ```
    sudo file -s /dev/xvdf
    ```

    The output of all the commands used to check and create a new filesystem is shown in the following screenshot:

![Figure 15.23 – Creating a new filesystem](img/B19682_15_23.jpg)

Figure 15.23 – Creating a new filesystem

The volume drive is now formatted.

1.  Now, let’s make it accessible to our local filesystem. We’ll name our mount point `packt_drive` and create it in the root directory:

    ```
    sudo mkdir /packt_drive
    /packt directory, we’re accessing the EBS volume:
    ```

![Figure 15.24 – Accessing the EBS volume](img/B19682_15_24.jpg)

Figure 15.24 – Accessing the EBS volume

Working with EBS volumes is similar to working with any volumes on Linux, so all the knowledge you’ve received from this chapter will be very useful when you’re working with Amazon EC2 instances. Now, let’s look at how to detach an EBS volume.

#### Detaching an EBS volume

Before detaching any EBS volume, you need to unmount it. In our case, as we are already in the terminal and using the volume, we will type the following command to unmount the volume we use:

```
sudo umount -d /dev/xvdf
```

Now that the volume has been unmounted, we can head to the EC2 dashboard and go to **Volumes**, select the corresponding **in-use** volume we want to detach, go to **Actions** in the top-right corner, and choose **Detach Volume**. Acknowledge the operation and wait for the volume to be detached.

We’re now at the final step of our backup-restore procedure – that is, attaching the new volume containing the snapshot to our EC2 instance.

We’ll conclude our exploration of the AWS EC2 console and related management operations here. For a comprehensive reference on EC2, please refer to the Amazon EC2 documentation at [https://docs.aws.amazon.com/ec2](https://docs.aws.amazon.com/ec2).

The EC2 management tasks we’ve presented so far exclusively used the AWS web console. If you’re looking to automate your EC2 workloads, you may want to adopt the AWS CLI, which we’ll check out next.

## Working with the AWS CLI

The AWS CLI is a unified tool for managing AWS resources. This tool was created so that we can manage all the AWS services from a terminal session on our local machine. Compared to the AWS web console, which offers visual tools through the web browser, the AWS CLI (as its name states) offers all the functionalities inside your terminal program on your machine. The AWS CLI is available for all major operating systems, such as Linux, macOS, and Windows. In the next section, we will show you how to install it on your Linux local machine.

### Installing the AWS CLI

To install the AWS CLI, please visit [https://aws.amazon.com/cli/](https://aws.amazon.com/cli/). At the time of writing, the latest release of the AWS CLI is version 2\. For the examples in this chapter, we’ll be using a Debian machine to install the AWS CLI while following the instructions at [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#cliv2-linux-install](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#cliv2-linux-install). This resource link offers instructions for installing the AWS CLI on all major operating systems, not only Linux.

Follow these steps to install the AWS CLI on Linux:

1.  We’ll start by downloading the AWS CLI v2 package (`awscliv2.zip`):

    ```
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    ```

2.  Next, we’ll unzip and install the AWS CLI:

    ```
    unzip awscliv2.zip
    aws command-line utility installed on our system. Let’s check the version:

    ```
    help:

    ```
    aws help
    ```

    ```

    ```

To manage your AWS EC2 resources using the `aws` utility, first, you need to configure your local environment to establish the required trust with the AWS endpoint. We’ll do that next.

### Configuring the AWS CLI

To configure the local AWS environment on your local machine, you will have to set up your AWS access key. If you already have it, you can skip this part:

1.  The AWS CLI configuration asks for your **AWS Access Key ID** and **AWS Secret Access Key** values. You can generate or retrieve these keys by logging into your AWS account.
2.  Select the dropdown next to your account name in the top-right corner of the AWS web console and choose **Security Credentials**. If you haven’t generated your access key yet, go to the **Access keys** tab and click on the **Create access key** button. You’ll have to store your AWS key ID and secret in a safe place for later reuse.
3.  Now that you have those keys, you can proceed to setting up AWS on your local machine. To configure the AWS environment, run the following command:

    ```
    aws configure
    ```

    The preceding command will prompt you for a few pieces of information, as suggested by the following output:

![Figure 15.25 – Configuring the local AWS CLI environment](img/B19682_15_25.jpg)

Figure 15.25 – Configuring the local AWS CLI environment

1.  In the AWS CLI configuration wizard, set `eu-central-1` (Frankfurt, in our case). You may want to enter the region of your choice or leave it as the default (`None`). If you don’t have a default Region specified, you’ll have to enter it every time you invoke the `aws` command.

At this point, we’re ready to use the AWS CLI. Let’s start by listing our EC2 instances.

### Querying EC2 instances

The following command provides detailed information about EC2 instances:

```
aws ec2 describe-instances
```

The preceding command provides sizeable JSON output (too long to present in a screenshot), with the details of all the EC2 instances we own in our default Region (`eu-central-1`). Alternatively, we can specify the Region with the `--``region` parameter:

```
aws ec2 describe-instances --region eu-central-1
```

We can get more creative and list only the EC2 instances matching a specific key-value tag, such as `env: packt`, that we previously tagged our instances with, using the `--``filters` parameter:

```
aws ec2 describe-instances \
  --filters "Name=tag-key,Values=env" \
  --filters "Name=tag-value,Values=packt"
```

The first `--filters` parameter specifies the key (`tag-key=env`), while the second points to the value (`tag-value=packt`).

Combining the `aws` and `jq` (*JSON query*) commands, we can extract the JSON fields we want. For example, the following command lists the `InstanceId`, `ImageId`, and `BlockDeviceMappings` fields of the EC2 instances tagged with `env:packt`:

```
aws ec2 describe-instances \
  --filters "Name=tag-key,Values=env" \
  --filters "Name=tag-value,Values=packt" | \
  jq '.Reservations[].Instances[] | { InstanceId, ImageId, BlockDeviceMappings }'
```

If the `jq` utility is not present on your Linux machine, install it with the following command:

```
sudo apt install -y jq # on Ubuntu/Debian
```

The output of the preceding `aws` command is as follows:

![Figure 15.26 – Querying EC2 instances](img/B19682_15_26.jpg)

Figure 15.26 – Querying EC2 instances

Note the `DeviceName` properties in the output JSON, which reflect only one block device (this is `/dev/sdf` since we deleted the EBS volume we attached earlier).

We can filter the output of the `aws ec2 describe-instances` command by any property. For example, the following command filters our EC2 instances by the `image-id` AMI:

```
aws ec2 describe-instances \
  --filters "Name=image-id versus ImageId. You have to keep this rule in mind when you write your filter queries.
Next, let’s plan to launch a new EC2 instance of the same AMI type with our current machine and the same security group.
Creating an EC2 instance
Let’s look at how we can create a new EC2 instance. To do this, we will need some prior information regarding the existing security group inside which we would like to create the new instance. The following command retrieves the security groups of the current instance (`i-0f8fe6aced634e71c`):

```
aws ec2 describe-instances \
  --filters "Name=instance-id,Values=i-0f8fe6aced634e71c" \
  --query "Reservations[].Instances[].SecurityGroups[]"
```

 The output is as follows:
![Figure 15.27 – Retrieving the security groups of an EC2 instance](img/B19682_15_27.jpg)

Figure 15.27 – Retrieving the security groups of an EC2 instance
To retrieve `GroupId` directly, we could run the following:

```
aws ec2 describe-instances \
  --filters "Name=instance-id,Values=i-0f8fe6aced634e71c" \
  --query "Reservations[].Instances[].SecurityGroups[].GroupId"
```

 In this case, the output would only show `GroupId`. Here, we used the `--query` parameter to specify the exact JSON path for the field we’re looking for (`GroupId`):

```
Reservations[].Instances[].SecurityGroups[].GroupId
```

 The use of the `--query` parameter somewhat resembles piping the output to the `jq` command, but it’s less versatile.
We also need the AMI ID. To obtain this, navigate to your AWS EC2 console and then, from the left-hand side pane, go to `ami-0faab6bdbac9486fb`.
To launch a new instance with the AMI type of our choice and the security group ID, we must use the `aws ec2` `run-instances` command:

```
aws ec2 run-instances --image-id ami-0faab6bdbac9486fb --count 1 --instance-type t3.micro --key-name packt_aws_key --security-group-ids sg-0aa7c8ef75503a9aa –placement AvailabilityZone=eu-central-1b
```

 Here’s a brief explanation of the parameters:

*   `image-id`: The AMI image ID (`ami-0faab6bdbac9486fb`); we’re using the same AMI type (*Ubuntu Linux*) as with the previous instance we created in the AWS EC2 web console
*   `count`: The number of instances to launch (`1`)
*   `instance-type`: The EC2 instance type (`t3.micro`)
*   `key-name`: The name of the SSH private key file (`packt_aws_key`) to use when connecting to our new instance; we’re reusing the SSH key file we created with our first EC2 instance in the AWS web console
*   `security-group-ids`: The security groups attached to our instance; we’re reusing the security group attached to our current instance (`sg-0aa7c8ef75503a9aa`)
*   `--placement`: The AZ to place our instance in (`AvailabilityZone=eu-central-1b`)

The command should run successfully and you will see the new EC2 instance running in your console. Here’s a screenshot of our EC2 console and the two instances running:
![Figure 15.28 – The new EC2 instance shown in the console](img/B19682_15_28.jpg)

Figure 15.28 – The new EC2 instance shown in the console
As you can see, our newly created EC2 instance has no name. Let’s add a name and some tags to it using the command line.
Naming and tagging an EC2 instance
The following command names our new instance as `aws_packt_testing_2`:

```
aws ec2 create-tags --resources i-091e2f515d15c3b0b --tags Key=Name,Value=aws_packt_testing_2
```

 Now, we can add a tag to the new instance. Adding a name and a tag can be done in a single command, but for a better understanding, we’ve decided to use two different commands. Here’s how to add a tag:

```
aws ec2 create-tags \
  --resources i-0e1692c9dfdf07a8d \
  --tags Key=env,Value=packt
```

 Now, let’s query the instance with the following command:

```
aws ec2 describe-instances \
  --filters "Name=tag-key,Values=env" \
  --filters "Name=tag-value,Values=packt" \
  --query "Reservations[].Instances[].InstanceId"
```

 The output shows that our two EC2 instances have the same tag (`packt`):
![Figure 15.29 – Querying EC2 instance IDs by tag](img/B19682_15_29.jpg)

Figure 15.29 – Querying EC2 instance IDs by tag
If you check the EC2 console inside your browser, you will see the two instances. Now, the second instance has its new name shown (`aws_packt_testing_2`) and the **Instance ID** value for each one:
![Figure 15.30 – Showing all instances in the EC2 console](img/B19682_15_30.jpg)

Figure 15.30 – Showing all instances in the EC2 console
Next, we’ll show you how to terminate an EC2 instance from the command line.
Terminating an EC2 instance
To terminate an EC2 instance, we can use the `aws ec2 terminate-instance` command. Note that terminating an instance results in the instance being deleted. We cannot restart a terminated instance. We could use the `aws ec2 stop-instances` command to stop our instance until later use.
The following command will terminate the instance with the ID of `i-091e2f515d15c3b0b`:

```
aws ec2 terminate-instances --instance-ids shutting-down (from a previously running state):
![Figure 15.31 – Terminating an instance](img/B19682_15_31.jpg)

Figure 15.31 – Terminating an instance
The instance will eventually transition to the `terminated` state and will no longer be visible in the AWS EC2 console. The AWS CLI will still list it among the instances until EC2 finally disposes of it. According to AWS, terminated instances can still be visible up to an hour after termination. It’s always a good practice to discard the instances in the `terminated` or `shutting-down` state when performing queries and management operations via the AWS CLI. The following screenshot of the EC2 console shows that the second EC2 instance is in a `terminated` state:
![Figure 15.32 – Showing the terminated instance in the EC2 console](img/B19682_15_32.jpg)

Figure 15.32 – Showing the terminated instance in the EC2 console
This is where we’ll wrap up our journey regarding AWS EC2\. Note that we’ve only scratched the surface of cloud management workloads in AWS.
The topics that were covered in this section provide a basic understanding of AWS EC2 cloud resources and help system administrators make better decisions when managing the related workloads. Power users may find the AWS CLI examples a good starting point for automating their cloud management workflows in EC2.
Now, let’s turn our focus to our next public cloud services contender, Microsoft Azure.
Working with Microsoft Azure
**Microsoft Azure**, also known as **Azure**, is a public cloud service by Microsoft for building and deploying application services in the cloud. Azure provides a full offering of a highly scalable **Infrastructure-as-a-Service** (**IaaS**) at relatively low costs, accommodating a wide range of users and business requirements, from small teams to large commercial enterprises, including financial, health, and governmental institutions.
In this section, we’ll explore some very basic deployment workflows using Azure, such as the following:

*   Creating a Linux virtual machine
*   Managing virtual machine sizes
*   Adding additional storage to a virtual machine
*   Working with the Azure CLI

Once you’ve created your free Azure account, go to [https://portal.azure.com](https://portal.azure.com) to access the Azure portal. You may want to enable the docked view of the portal navigation menu on the left for quick and easy access to your resources. Throughout this chapter, we’ll use the docked view for our screen captures. Go to the **Portal settings** cog in the top-right corner and choose **Docked** for the default mode of the portal menu.
Now, let’s create our first resource in Azure – an Ubuntu virtual machine.
Creating and deploying a virtual machine
We’ll follow a step-by-step procedure, guided by the resource wizard in the Azure portal. Let’s start with the first step, which is creating a compute resource for the virtual machine.

1.  **Creating a compute resource**: Start by clicking on the **Create a resource** option in the left navigation menu or under **Azure services** in the main window. This will take us to the Azure Marketplace. Here, we can search for our resource of choice. You can either search for a relevant keyword or narrow down your selection based on the resource type you’re looking for. Let’s narrow down our selection by choosing **Compute**, then selecting **Ubuntu Server 22.04 LTS** from the available **Popular Marketplace products** options. You may click on **Learn more** for a detailed description of the image or click **Create**.

    When we select **Ubuntu Server 22.04 LTS**, we’ll be guided through the process of configuring and creating our new virtual machine. The setup page looks like this:

![Figure 15.33 – Creating a virtual machine in Azure](img/B19682_15_33.jpg)

Figure 15.33 – Creating a virtual machine in Azure
In the **Basics** tab, information such as the subscription type, resource group, region, image, and architecture is provided. The other tabs include **Disks**, **Networking**, **Management**, **Monitoring**, **Advanced**, and **Tags**.

1.  `packt-demo`. If we had a previously created resource group, we could specify it here.
2.  `packt-ubuntu-demo` and place it in the `(Europe) France Central` Region, the closest to the geographical location where our instance will operate. The size of our machine will directly impact the associated costs. The following screenshot shows the details that have been specified so far regarding the **Resource group** set up, **Virtual machine name**, **Region**, **Availability options**, **Security type**, **Image**, **VM architecture**, and **Size**:

![Figure 15.34 – Setting up the Azure virtual machine](img/B19682_15_34.jpg)

Figure 15.34 – Setting up the Azure virtual machine
Alternatively, we could browse different options for **Image** and **Size** by choosing **See all images** or **See all sizes**, respectively. Azure also provides a *pricing calculator* online tool for various resources at [https://azure.microsoft.com/en-us/pricing/calculator/](https://azure.microsoft.com/en-us/pricing/calculator/).

1.  `packt` and `packt-ubuntu-demo_key`. Next, we must set **Inbound port rules** for our instance to allow SSH access. If, for example, our machine will run a web server application, we can also enable HTTP and HTTPS access:

![Figure 15.35 – Enabling SSH authentication and access to the virtual machine](img/B19682_15_35.jpg)

Figure 15.35 – Enabling SSH authentication and access to the virtual machine
At this point, we are ready to create our virtual machine. The wizard can take us further to the additional steps of specifying the *disks* and *networking* configuration associated with our instance. For now, we’ll leave them as-is.

1.  **Validating and deploying the virtual machine**: We can click the **Review + create** button to initiate the validation process. Next, the deployment wizard will validate our virtual machine configuration. In a few moments, if everything goes well, we’ll get a **Validation passed** message specifying the product details and our instance’s hourly rate. By clicking **Create**, we agree to the relevant terms of use, and our virtual machine will be deployed shortly:

![Figure 15.36 – Creating the virtual machine](img/B19682_15_36.jpg)

Figure 15.36 – Creating the virtual machine
In the process, we’ll be prompted to download the SSH private key for accessing our instance:
![Figure 15.37 – Downloading the SSH private key for accessing the virtual machine](img/B19682_15_37.jpg)

Figure 15.37 – Downloading the SSH private key for accessing the virtual machine

1.  **Deployment completion**: If the deployment completes successfully, we’ll get a brief pop-up message stating **Deployment succeeded** and a **Go to resource** button that will take us to our new virtual machine:

![Figure 15.38 – Successfully deploying a virtual machine](img/B19682_15_38.jpg)

Figure 15.38 – Successfully deploying a virtual machine
We’ll also get a brief report about the deployment details if we click on the relevant drop-down button:
![Figure 15.39 – Deployment details](img/B19682_15_39.jpg)

Figure 15.39 – Deployment details
Let’s take a quick look at each of the resources that were created with our virtual machine deployment:

*   `packt-ubuntu-demo`: The virtual machine host
*   `packt-ubuntu-demo348`: The network interface (or network interface card) of the virtual machine
*   `packt-ubuntu-demo-ip`: The IP address of the virtual machine
*   `packt-ubuntu-demo-vnet`: The virtual network associated with the resource group (`packt-demo`)
*   `packt-ubuntu-demo-nsg`: The **Network Security Group** (**NSG**) controlling inbound and outbound access to and from our instance

Azure will create a new set of the resource types mentioned previously with every virtual machine, except the virtual network corresponding to the resource group, when the instance is placed in an existing resource group. Don’t forget that we also created a new resource group (`packt-demo`), which is not shown in the deployment report.
Let’s try to connect to our newly created instance (`packt-ubuntu-demo`). Go to `packt-ubuntu-demo`). In the `20.19.173.232`).
Now that we’ve deployed our virtual machine, we want to make sure we can access it via SSH. We’ll do that next.
Connecting with SSH to a virtual machine
Before we connect to our virtual machine via SSH, we need to set the permissions to our SSH private key file so that it’s not publicly viewable:

```
chmod 400 packt-ubuntu-demo_key.pem
```

 Next, we must connect to our Azure Ubuntu instance with the following command:

```
ssh -i packt-rhel.pem packt@20.19.173.232
```

 We must use the SSH key (`packt-ubuntu-demo_key.pem`) and administrator account (`packt`) that were specified when we created the instance. Alternatively, you can click on the **Connect** button in the virtual machine’s **Overview** tab and then click **SSH**. This action will bring up a view where you can see the preceding commands and copy and paste them into your terminal.
A successful connection to our Ubuntu instance should yield the following output:
![Figure 15.40 – Connecting to the Azure virtual machine](img/B19682_15_40.jpg)

Figure 15.40 – Connecting to the Azure virtual machine
Now that we’ve created our first virtual machine in Azure, let’s look at some of the most common management operations that are performed during a virtual machine’s lifetime.
Managing virtual machines
As our applications evolve, so does the computing power and capacity required by the virtual machines hosting the applications. As system administrators, we should know how cloud resources are being utilized. Azure provides the necessary tools for monitoring the health and performance of virtual machines. These tools are available on the **Monitoring** tab of the virtual machine’s management page.
A small virtual machine with a relatively low number of virtual CPUs and reduced memory may negatively impact application performance. On the other hand, an oversized instance would yield unnecessary costs. Resizing a virtual machine is a common operation in Azure. Let’s see how we can do it.
Changing the size of a virtual machine
Azure makes it relatively easy to resize virtual machines. In the portal, go to **Virtual Machines**, select your instance, and click **Size** under **Availability +** **Scale**:
![Figure 15.41 – Changing the size of a virtual machine](img/B19682_15_41.jpg)

Figure 15.41 – Changing the size of a virtual machine
Our virtual machine (`packt-ubuntu-demo`) is of *B1s* size (1 vCPUs, 1 GB RAM). We can choose to size it up or down. For demo purposes, we could resize to the lower *B1ls* capacity (1 vCPU, 0.5 GB RAM). We could select the **B1ls** (1 vCPU, 0.5 GB RAM) option and click the **Resize** button. Lowering the size of our instance will also result in cost savings. Azure will stop and restart our virtual machine while resizing. It’s always good to stop the machine before changing the size to avoid possible data inconsistencies within the instance.
One of the remarkable features of virtualized workloads in Azure is the ability to scale – including the storage capacity – by adding additional data disks to virtual machines. We can add existing data disks or create new ones.
Let’s look at how to add a secondary data disk to our virtual machine.
Adding additional storage
Azure can add disks to our instance on the fly, without stopping the machine. We can add two types of disks to a virtual machine: **data disks** and **managed disks**. In this chapter, we will only provide information on how to add data disks. For more information on disk types on Azure, please visit [https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types](https://docs.microsoft.com/en-us/azure/virtual-machines/disks-types).
Let’s learn how to add a data disk:

1.  Navigate to **Virtual machines** in the left navigation menu and select your instance, click **Disks** under **Settings**, and then click on **Create and attach a** **new disk**.
2.  In the disk properties, leave the `packt-disk`), **Storage Type**, and **Size** (such as **4 GB**) values. Click **Apply** when you’re done:

![Figure 15.42 – Adding a new data disk](img/B19682_15_42.jpg)

Figure 15.42 – Adding a new data disk
At this point, the new disk is attached to our virtual machine, but the disk hasn’t been initialized with a filesystem yet.

1.  Connect to our virtual machine via SSH:

    ```
    ssh -i packt-rhel.pem packt@20.19.173.232
    ```

     2.  List the current block devices:

    ```
    sdc:
    ```

![Figure 15.43 – Identifying the block device for the new data disk](img/B19682_15_43.jpg)

Figure 15.43 – Identifying the block device for the new data disk

1.  Verify that the block device is empty:

    ```
    /dev/sdc: data, meaning that the data disk doesn’t contain a filesystem yet.
    ```

     2.  Next, initialize the volume with an XFS filesystem:

    ```
    /packt) and mount the new volume:

    ```
    sudo mkdir /packt
    sudo mount /dev/sdc /packt
    ```

    ```

Now, we can use the new data disk for regular file storage.
Note that data disks will only be persisted during the lifetime of the virtual machine. When the virtual machine is paused, stopped, or terminated, the data disk becomes unavailable. When the machine is terminated, the data disk is permanently lost. For persistent storage, we need to use *managed disks*, similar in behavior to network-attached storage. See the link at the beginning of this section for more details.
So far, we have performed all these management operations in the Azure portal. But what if you want to automate your workloads in the cloud using scripting? This can be done using Azure CLI.
Working with the Azure CLI
Azure CLI is a specialized command-line interface for managing your resources in the cloud. First, let’s install the Azure CLI on our platform of choice. Follow the instructions at [https://docs.microsoft.com/en-us/cli/azure/install-azure-cli](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli). We’ll choose the Azure CLI for Linux and, for demo purposes, install it on a Debian machine. The related instructions can be found at [https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux). From the multiple installation options available, we’ll use the following command:

```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

 After the installation is complete, we can invoke the Azure CLI with the `az` command:

```
az help
```

 The preceding command displays detailed help information about using the `az` utility. However, before performing any management operations, we need to authenticate the CLI with our Azure credentials. The following command will set up the local Azure CLI environment accordingly:

```
az login
```

 We’ll be prompted with a message containing an authentication code and the URL ([https://microsoft.com/devicelogin](https://microsoft.com/devicelogin)) to visit and enter the code:
![Figure 15.44 – Initializing the Azure CLI environment](img/B19682_15_44.jpg)

Figure 15.44 – Initializing the Azure CLI environment
At this point, we are ready to use the Azure CLI for management operations. Let’s create a new resource group called `packt-dev` in the `West` `US` Region:

```
az group create --name packt-dev --location francecentral
```

 The preceding command yields the following output upon successfully creating the resource group:
![Figure 15.45 – Creating a new resource group](img/B19682_15_45.jpg)

Figure 15.45 – Creating a new resource group
Next, we must launch an Ubuntu virtual machine named `packt-ubuntu-dev` in the Region we just created:

```
az vm create --resource-group packt-dev --name packt-ubuntu-dev --image Ubuntu2204 --admin-username packt --generate-ssh-keys
```

 Let’s quickly go through each of the preceding command-line options:

*   `resource-group`: The name of the resource group (`packt-dev`) where we create our virtual machine
*   `name`: The name of the virtual machine (`packt-ubuntu-dev`)
*   `image`: The Linux distribution to use (`Ubuntu2204`)
*   `admin-username`: The username of the machine’s administrator account (`packt`)
*   `generate-ssh-keys`: This generates a new SSH key pair to access our virtual machine

The preceding code produces the following output:
![Figure 15.46 – Creating a new virtual machine](img/B19682_15_46.jpg)

Figure 15.46 – Creating a new virtual machine
As the output suggests, the SSH key files have been automatically generated and placed in the local machine’s `~/.ssh` directory to allow SSH access to the newly created virtual machine. The JSON output also provides the public IP address of the machine:

```
"publicIpAddress": "51.103.100.132"
```

 The following command lists all virtual machines:

```
az vm list
```

 You can also check the Azure portal in your browser to see all your virtual machines, including the newly created one:
![Figure 15.47 – Viewing the virtual machines in the Azure portal](img/B19682_15_47.jpg)

Figure 15.47 – Viewing the virtual machines in the Azure portal
To get information about a specific virtual machine (`packt-ubuntu-dev`), run the following command:

```
az vm show \
  --resource-group packt-dev \
  --name packt-ubuntu-dev), run the following command:

```
az vm redeploy \
  --resource-group packt-dev \
  --name packt-ubuntu-dev):

```
az vm delete \
  --resource-group packt-dev \
  --name az vm), we also need to specify the resource group the machine belongs to.
A comprehensive study of the Azure CLI is beyond the scope of this chapter. For detailed information, please visit the Azure CLI’s online documentation portal: [https://docs.microsoft.com/en-us/cli/azure/](https://docs.microsoft.com/en-us/cli/azure/).
This concludes our coverage of public cloud deployments with AWS and Azure. We have covered a vast domain and have merely skimmed the surface of cloud management workloads. We encourage you to build upon this preliminary knowledge and explore more, starting with the AWS and Azure cloud docs. The relevant links are mentioned in the *Further reading* section, together with other valuable resources.
Now, let’s summarize what you have learned so far about AWS and Azure.
Summary
AWS and Azure provide a roughly similar set of features for flexible compute capacity, storage, and networking, with pay-as-you-go pricing. They share the essential elements of a public cloud – elasticity, autoscaling, provisioning with self-service, security, and identity and access management. This chapter explored both cloud providers strictly from a practical vantage point, focusing on typical deployment and management aspects of everyday cloud administration tasks.
We covered topics such as launching and terminating a new instance or virtual machine. We also looked at resizing an instance to accommodate a higher or lower compute capacity and scaling the storage by creating and attaching additional block devices (volumes). Finally, we used CLI tools for scripting various cloud management workloads.
When working with AWS, we learned a few basic concepts about EC2 resources. Next, we looked at typical cloud management tasks, such as launching and managing instances, adding and configuring additional storage, and using EBS snapshots for disaster recovery. Finally, we explored the AWS CLI with hands-on examples of standard operations, including querying and launching EC2 instances, creating and adding additional storage to an instance, and terminating an instance.
At this point, you should be familiar with the AWS and Azure web administration consoles and CLI tools. You have learned the basics of some typical cloud management tasks and a few essential concepts about provisioning cloud resources. Overall, you’ve enabled a special skillset of modern-day Linux administrators by engaging in cloud-native administration workflows. Combined with the knowledge you’ve built so far, you are assembling a valuable Linux administration toolbelt for on-premises, public, and hybrid cloud systems management.
In the next chapter, we’ll take this further and introduce you to managing application deployments using containerized workflows and services with Kubernetes.
Questions
Let’s recap some of the concepts you’ve learned about in this chapter as a quiz:

1.  What is an AZ?
2.  Between a `t3.small` and a `t3.micro` AWS EC2 instance type, which one yields better performance?
3.  You have launched an AWS EC2 instance in the `us-west-1a` AZ and plan to attach an EBS volume created in `us-west-1b`. Would this work?
4.  What is the SSH command to connect to your AWS EC2 instance or Azure virtual machine?
5.  What is the Azure CLI command for listing your virtual machines? How about the equivalent AWS CLI command?
6.  What is the AWS CLI command for launching a new EC2 instance?
7.  What is the Azure CLI command for deleting a virtual machine?

Further reading
Here are a few resources to further explore AWS and Azure cloud topics:

*   AWS EC2: [https://docs.aws.amazon.com/ec2/index.html](https://docs.aws.amazon.com/ec2/index.html)
*   Azure: [https://docs.microsoft.com/en-us/azure](https://docs.microsoft.com/en-us/azure)
*   *AWS for System Administrators*, by Prashant Lakhera, Packt Publishing
*   *Learning AWS – Second Edition*, by Aurobindo Sarkar and Amit Shah, Packt Publishing
*   *Learning Microsoft Azure*, by Geoff Webber-Cross, Packt Publishing
*   *Learning Microsoft Azure: A Hands-On Training [Video]*, by Vijay Saini, Packt Publishing

```

```

```

```

```