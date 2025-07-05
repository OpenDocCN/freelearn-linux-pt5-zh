# 14

# Short Introduction to Cloud Computing

In this chapter, you will learn the basics of **cloud computing** and will be presented with the core foundations of cloud infrastructure technologies. You will learn about the as-a-service solutions such as **Infrastructure as a Service** (**IaaS**), **Platform as a Service** (**PaaS**), **Software as a Service** (**SaaS**), and **Containers as a Service** (**CaaS**). You will be presented with the basics of cloud standards, **Development and Operations** (**DevOps**), **continuous integration/continuous deployment** (**CI/CD**), and **microservices**. A base knowledge of the cloud will offer at least a basic introduction to AWS, Azure, and other cloud solutions. By the end of this chapter, we will also introduce you to technologies such as **Ansible** and **Kubernetes**. This chapter will provide a concise theoretical introduction that will be the foundation for the following three cloud-related chapters, which will provide you with important practical knowledge on the many solutions presented here.

In this chapter, we’re going to cover the following main topics:

*   Introduction to cloud technologies
*   Introducing IaaS solutions
*   Introducing PaaS solutions
*   Introducing CaaS solutions
*   Introducing DevOps
*   Exploring cloud management tools

# Technical requirements

No special technical requirements are needed as this chapter is a purely theoretical one. All you need is a desire to learn about cloud technologies.

# Introduction to cloud technologies

The term “cloud computing,” or the simple alternative “cloud,” is not missing from any tech enthusiast’s or **Information Technology** (**IT**) professional’s vocabulary these days. You don’t even have to be involved in IT at all to hear (or even use) the term “cloud” relatively often. Today’s computing landscape is changing at a rapid pace, and the pinnacle of this change is the cloud and the technologies behind it. According to the literature, the term “cloud computing” was used for the first time in 1996, in a business plan from Compaq ([https://www.technologyreview.com/2011/10/31/257406/who-coined-cloud-computing/](https://www.technologyreview.com/2011/10/31/257406/who-coined-cloud-computing/)).

Cloud computing is a relatively old concept, even though it was not referred to using this term right from the beginning. It is a computing model that was used from the early days of computing. Back in the 1950s, for example, there were mainframe computers that were accessed from different terminals. This model is similar to modern cloud computing, where services are hosted and delivered over the internet to different terminals, from desktop computers to smartphones, tablets, or laptops. This model is based on technologies that are extremely complex and essential for anyone who wants to master them to know.

You might be wondering why we have an entire section dedicated to cloud computing and technologies inside a *Mastering Linux Administration* title. This is simply because Linux has taken over the cloud in the last decade, in the same way that Linux took over the internet and the high-performance computing landscape. According to the TOP500 association, the world’s top 500 supercomputers all run on Linux ([https://top500.org/lists/top500/2020/11/](https://top500.org/lists/top500/2020/11/)). Clouds need to have an operating system to operate on, but it doesn’t have to be Linux. Nevertheless, Linux runs on almost 90% of public clouds ([https://www.redhat.com/en/resources/state-of-linux-in-public-cloud-for-enterprises](https://www.redhat.com/en/resources/state-of-linux-in-public-cloud-for-enterprises)), mostly because its open source nature appeals to IT professionals inside the public and private sectors alike.

In the following section, we will tackle the subject of cloud standards and why it is good to know about them when planning to deploy or manage a cloud instance.

## Exploring the cloud computing standards

Before going into more details about cloud computing, let’s give you a short introduction to what cloud standards are and their importance in the overall contemporary cloud landscape. You may know that almost every activity in the wider **Information and Communications Technology** (**ICT**) spectrum is governed by some kind of standard or regulation.

Cloud computing is no wild land, and you will be surprised at how many associations, regulatory boards, and organizations are involved in developing standards and regulations for it. Covering all these institutions and standards is out of the scope of this book and chapter, but in the following sections, we will describe some of the most important and relevant ones (in our opinion) so that you can have an idea of their importance in keeping clouds together and web applications running.

### International Organization for Standardization/International Electrotechnical Commission

Two of the most widely known standards entities are the **International Organization for Standardization** (**ISO**) and the **International Electrotechnical Commission** (**IEC**), and they currently have 28 published and under-development standards on cloud computing and distributed platforms. They have a joint task group to develop standards for specific cloud core infrastructure, consumer application platforms, and services. Those standards are found under the responsibility of the **Joint Technical Committee 1** (**JTC 1**) **subcommittee 38** (**SC38**), or **ISO/IEC JTC 1/SC 38** for short.

Examples of standards from ISO/IEC include the following:

*   Cloud computing **service-level agreement** (**SLA**) frameworks:
    *   ISO/IEC 19086-1:2016
    *   ISO/IEC 19086-2:2018
    *   ISO/IEC 19086-3:2017
*   Cloud computing **service-oriented architecture** (**SOA**) frameworks:
    *   ISO/IEC 18384-1:2016
    *   ISO/IEC 18384-2:2016
    *   ISO/IEC 18384-3:2016
*   **Open Virtualization Format** (**OVF**) specifications:
    *   ISO/IEC 17203:2017
*   Cloud computing **data sharing agreement** (**DSA**) frameworks:
    *   ISO/IEC CD 23751
*   **Distributed application platforms and services** (**DAPS**) technical principles:
    *   ISO/IEC TR 30102:2012

To take a closer look at those standards, please go to [https://www.iso.org/committee/601355/x/catalogue/](https://www.iso.org/committee/601355/x/catalogue/).

### The Cloud Standards Coordination initiative

Next on our list of standards development entities is an initiative called the **Cloud Standards Coordination** (**CSC**), created by the **European Commission** (**EC**), together with specialized bodies.

Back in 2012, the EC, together with the **European Telecommunications Standards Institute** (**ETSI**), launched the CSC to develop standards and policies for cloud security, interoperability, and portability. The initiative had two phases, with Phase 1 starting in 2012 and Phase 2 starting in 2015\. The final reports of Phase 2 (version 2.1.1), were made public, as follows:

*   Cloud computing users’ needs (ETSI SR 003 381)
*   Standards and Open Source (ETSI SR 003 382)
*   Interoperability and Security (ETSI SR 003 391)
*   Standards Maturity Assessment (ETSI SR 003 392)

For more details on each of those standards, access the following link: [http://csc.etsi.org/](http://csc.etsi.org/).

### National Institute of Standards and Technology

The list continues with one of the most widely known entities in standards development: the **United States** (**US**) **National Institute of Standards and Technology** (**NIST**). This will not be the first time you will read about NIST in this book. It is the standards development body in the US Department of Commerce. The main objective of NIST is the standardization of security and interoperability inside US government agencies, so anyone interested in developing for those entities should take a look at the NIST cloud documentation. The NIST document that standardizes cloud computing is called NIST SP 500-291r2 and can be found at [http://csrc.nist.gov/publications/nistpubs/800-145/SP800-145.pdf](http://csrc.nist.gov/publications/nistpubs/800-145/SP800-145.pdf).

### The International Telecommunication Union

We will close our shortlisting with one of the oldest—if not the oldest—standards development bodies, part of the **United Nations** (**UN**) organization: the **International Telecommunication Union** (**ITU**). The ITU is a body inside the UN, and its main focus is to develop standards for communications, networking, and development. This agency was founded in 1865 and, among other things, it is responsible for global radio frequency spectrum and satellite orbit allocation. It is also responsible for the use of Morse code as a standard means of communication. When it comes to global information infrastructure, internet protocols, next-generation networks, **Internet of Things** (**IoT**), and smart cities, the ITU has a lot of standards and recommendations available. To check them all out, have a look at the following link: [https://www.itu.int/rec/T-REC-Y/en](https://www.itu.int/rec/T-REC-Y/en). To narrow down the document list from the aforementioned link, some specific cloud computing documents can be found using document codes, from Y.3505 up to Y.3531\. The cloud computing standards were developed by the **Study Group 13** (**SG13**) **Joint Coordination Activity on Cloud Computing** (**JCA-Cloud**) within the ITU.

Besides the entities described in this section, there are many others, including the following:

*   The **Cloud Standards Customer** **Council** (**CSCC**)
*   The **Distributed Management Task** **Force** (**DMTF**)
*   The **Organization for the Advancement of Structured Information** **Standards** (**OASIS**)

The main reason for adopting standards for cloud computing is ease of use when it comes to either the **Cloud Service Provider** (**CSP**) or the client. Both categories need to have easy access to data, more so for CSPs and application developers, as easy access to data is translated into agility and interoperability. However, standards, besides being technically correct, need to be consistent and persistent. According to the literature, there are two main standards groups: ones that are established from practice, and ones that are being regulated. An important part of the cloud standards, from the second category, is the **application programming interfaces** (**APIs**). Standardization of application frameworks, network protocols, and APIs guarantees success for everyone involved.

### Understanding the cloud through API standards

APIs are sets of protocols, procedures, and functions: all the bricks needed to build a web-distributed application. Modern APIs emerged at the beginning of the 21st century, firstly as a theory in Roy Fielding’s doctoral dissertation. Before the modern APIs, there were SOA standards and the **Simple Object Access Protocol** (**SOAP**), based on **Extensible Markup Language** (**XML**). Modern APIs are based on a new application architectural style, called **REpresentational State** **Transfer** (**REST**).

REST APIs are based on a series of architectural styles, elements, connectors, and views that are clearly described by Roy Fielding in his thesis. There are six guiding constraints for an API to be RESTful, and they are as follows:

*   Uniform user interfaces
*   Client-server clear delineation
*   Stateless operations
*   Cacheable resources
*   Layered system of servers
*   Code on-demand execution

Following these guiding principles is far from following a standard, but REST provides them for developers as a high-level abstraction layer. Unless they are standardized, they will always remain great principles that generate confusion and frustration among developers.

The only organization that managed to standardize REST APIs for the cloud is the DMTF, through the **Cloud Infrastructure Management Interface** (**CIMI**) model and the RESTful **HyperText Transfer Protocol** (**HTTP**)-based protocol, in a document coded DSP0263 version 2.0.0, which can be downloaded from the following link: [https://www.dmtf.org/standards/cloud](https://www.dmtf.org/standards/cloud).

There are other specifications that emerge as possible future standards for developers to use when designing REST APIs. Among those, there is the **OpenAPI Specification** (**OAS**), an industry standard that provides a language-agnostic description for API development (document available at [http://spec.openapis.org/oas/v3.0.3](http://spec.openapis.org/oas/v3.0.3)), and **GraphQL**, as a query language and server-side runtime, with support for several programming languages such as Python, JavaScript, Scala, Ruby, and **PHP: Hypertext** **Preprocessor** (**PHP**).

REST managed to become the preferred API because it is easier to understand, more lightweight, and simple to write. It is more efficient, uses less bandwidth, supports many data formats, and uses **JavaScript Object Notation** (**JSON**) as the preferred data format. JSON is easy to read and write and offers better interoperability between applications written in different languages, such as JavaScript, Ruby, Python, and Java. By using JSON as the default data format for the API, it makes it friendly, scalable, and platform-agnostic.

APIs are everywhere on the web and in the cloud and are the base for SOA and microservices. For example, microservices use RESTful APIs to communicate between services by offering an optimized architecture for cloud-distributed resources.

Therefore, if you want to master cloud computing technologies, you should be open to embracing cloud standards. In the next section, we will discuss the cloud types and architecture.

## Understanding the architecture of the cloud

The cloud’s architectural design is similar to a building’s architectural design. There is one design paradigm that governs the cloud—the one in which the design starts from a blank, clean drawing board where the architects put together different standardized components in order to achieve an architectural design. The final result is based on a certain architectural style. The same happens when designing the cloud’s architecture.

The cloud is based on a client-server, layered, stateless, network-based architectural style. The REST APIs, SOA, microservices, and web technologies all are the base components that form the foundation of the cloud. The architecture of the cloud has been defined by NIST ([https://www.nist.gov/publications/nist-cloud-computing-reference-architecture](https://www.nist.gov/publications/nist-cloud-computing-reference-architecture)).

Some of the technologies behind the cloud have been discussed in [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231), *Working with Virtual Machines*, and [*Chapter 12*](B19682_12.xhtml#_idTextAnchor257), *Managing Containers with Docker*. Indeed, both virtualization and containers are the foundation technologies of cloud computing.

Let’s imagine a situation where you would like to have several Linux systems to deploy your apps. What you do first is go to a CSP and request the systems you need. The CSP will create the **virtual machines** (**VMs**) on its infrastructure, according to your needs, and will put all of them in the same network and share the credentials to access them with you. This way, you will have access to the systems you wanted, in exchange for a subscription fee that is billed either on a daily, monthly, or yearly basis or based on a resource-consumption basis. Most of the time, those CSP requests are done through a specific web interface, developed by the provider to best suit the needs of their users.

Everything that the cloud uses as technology is based on VMs and containers. Inside the cloud, everything is abstracted and automated. In the following section, we will provide you with information about the types of services that are available in the cloud.

### Describing the types of infrastructure and services

No matter the type, each cloud has a specific architecture, just as we showed you in the previous section. It provides the blueprints for the foundation of cloud computing. The cloud architecture is the base for the cloud infrastructure, and the infrastructure is the base for cloud services. See how everything is connected? Let’s now see what the infrastructure and services are with regard to the cloud.

There are four main **cloud infrastructure types**, as follows:

*   **Public clouds**: These run on infrastructure owned by the provider and are available mostly off-premises; the largest public cloud providers are AWS, Microsoft Azure, and Google Cloud.
*   **Private clouds**: These run specifically for individuals and groups with isolated access; they are available on both on-premises and off-premises hardware infrastructure. There are managed private clouds available, or dedicated private clouds.
*   **Hybrid clouds**: These are both private and public clouds running inside connected environments, with resources available for potential on-demand scaling.
*   **Multi-clouds**: These are more than one cloud running from more than one provider.

Besides the cloud infrastructure types, there are also four main **cloud** **service types**:

*   **IaaS**: With an IaaS cloud service type, the cloud provider manages all hardware infrastructure such as servers and networking, plus virtualization and storage of data. The infrastructure is owned by the provider and rented by the user; in this case, the user needs to manage the operating system, the runtimes, automation, management solutions, and containers, together with the data and applications. IaaS is the backbone of every cloud computing service, as it provides all the resources.
*   **CaaS**: This is considered to be a subset of IaaS; it has the same advantages as IaaS, only that the base consists of containers, not VMs, and it is better suited to deploying distributed systems and microservices architectures.
*   **PaaS**: With a PaaS cloud service type, the hardware infrastructure, networking, and software platform are managed by the cloud provider; the user manages and owns the data and applications.
*   **SaaS**: With a SaaS cloud service type, the cloud provider manages and owns the hardware, networking, software platform, management, and software applications. This type of service is also known for delivering web apps or mobile apps.

Besides these types of services, there is another one that we should bring into the discussion: **serverless computing** services. In contrast to what the name might suggest, serverless computing still implies the use of servers, but the infrastructure running them is not visible to users, who in most cases are developers. Serverless can also be referred to as **Function as a Service** (**FaaS**), and it provides an on-demand way to execute modular code, by letting developers update their code on the fly. Examples of this kind of service would be AWS Lambda, Azure Functions, and Google Cloud Functions. It is similar to SaaS; actually, it would fit right between PaaS and SaaS. It has no infrastructure management, is scalable, offers a faster way to market for app developers, and is efficient when it comes to the use of resources.

Now that you know the types of cloud infrastructure and services, you might wonder why you, your business, or anyone you know should migrate to cloud services. First of all, cloud computing is based on on-demand access to various resources that are hosted and managed by a CSP. This means that the infrastructure is owned or managed by the CSP and the user will be able to access the resources based on a subscription fee. Should you migrate to the cloud? We will discuss the advantages and disadvantages of migrating to the cloud in the next section.

## Knowing the key features of cloud computing

Before deciding whether migrating to the cloud would be a good decision, you need to know the advantages and disadvantages of doing this. Cloud computing does provide some essential features, such as the following ones listed:

*   **Cost savings**: There are reduced costs generated by the infrastructure setup, which is now managed by the CSP; this puts the user’s focus on application development and running the business.
*   **Speed**, **agility**, and **resource access**: All the resources are available from any place, just a few clicks away, at any time (dependent on internet connectivity and speed).
*   **Reliability**: Resources are hosted in different locations, by providing good quality control, **disaster recovery** (**DR**) policies, and loss prevention; maintenance is done by the CSP, meaning that end users don’t need to waste time and money doing this.

Besides the advantages (key features) listed previously, there are possible disadvantages too, such as the following:

*   **Performance variations**: Performance may vary depending on the CSP you choose, but none of the big names out there (such as AWS, Azure, or GCP) have any significant performance issues. In these cases, performance is dictated by the local internet speed of the user, so it isn’t a CSP problem after all. This might not be the case for other CSPs though.
*   **Downtime**: Downtime could be an issue, but all major providers strive to offer 99.9% uptime. If disaster strikes, issues are solved in a matter of minutes—or in worst-case scenarios, in a matter of hours.
*   **Lack of predictability**: There is a lack of predictability with regard to the CSP and its presence on the market, but rest assured that none of the big players will go away anytime soon.

Therefore, these are not game stoppers for anyone wanting to migrate to the cloud. In the next section, we will introduce you to some IaaS solutions.

# Introducing IaaS solutions

IaaS is the backbone of cloud computing. It offers on-demand access to resources, such as compute, storage, network, and so on. The CSP uses hypervisors to provide IaaS solutions. In this section, we provide you with information about some of the most widely used IaaS solutions available. We will give you details about providers such as **Amazon Elastic Compute Cloud** (**Amazon EC2**), and Microsoft Azure Virtual Machines as the big players, and **DigitalOcean** as a viable solution. We will tackle **OpenStack** too, for those interested in what it could offer.

## Amazon EC2

The IaaS solution provided by AWS is called Amazon EC2\. It provides a good infrastructure solution for anyone, from low-cost compute instances to a high-power **graphics processing unit** (**GPU**) for machine learning. AWS was the first provider of IaaS solutions 12 years ago and it is doing better than ever, even after the COVID-19 pandemic ([https://www.statista.com/chart/18819/worldwide-market-share-of-leading-cloud-infrastructure-service-providers/](https://www.statista.com/chart/18819/worldwide-market-share-of-leading-cloud-infrastructure-service-providers/)).

When starting with Amazon EC2, you have several steps to fulfill:

*   The first is the option to choose your **Amazon Machine Image** (**AMI**), which is basically a preconfigured image of either Linux or Windows. When it comes to Linux, you can choose between the following:
    *   Amazon Linux 2 (based on CentOS/**Red Hat Enterprise** **Linux** (**RHEL**))
    *   RHEL 8/9
    *   **SUSE Linux Enterprise** **Server** (**SLES**)
    *   Ubuntu Server 20.04/22.04 **Long-Term** **Support** (**LTS**)
*   You will need to choose your **instance type** from a really wide variety. To learn more about EC2 instances, visit [https://aws.amazon.com/ec2/instance-types/](https://aws.amazon.com/ec2/instance-types/) and find out details about each one. EC2, for example, is the only provider that offers Mac instances, based on Mac mini. To use Linux, you can choose from low-end instances up to high-performance instances, depending on your needs.
*   Amazon provides an **Elastic Block Store** (**EBS**) option with **solid-state drive** (**SSD**) and magnetic mediums available. You can select a custom value depending on your needs. EC2 is a flexible solution compared to other options. It has an easy-to-use and straightforward interface, and you will only pay for the time and resources you use. An example of how to deploy on EC2 will be provided to you in [*Chapter 15*](B19682_15.xhtml#_idTextAnchor326), *Deploying to the Cloud with AWS* *and Azure*.

## Microsoft Azure Virtual Machines

Microsoft is the second biggest player in the cloud market, right after Amazon. Azure is the name of their cloud computing offering. Even though it is provided by Microsoft, Linux is the most widely used operating system on Azure ([https://www.zdnet.com/article/microsoft-developer-reveals-linux-is-now-more-used-on-azure-than-windows-server/](https://www.zdnet.com/article/microsoft-developer-reveals-linux-is-now-more-used-on-azure-than-windows-server/)).

Azure’s IaaS offering is called **Virtual Machines** and is similar to Amazon’s offering; you can choose between many tiers. What is different about Microsoft’s offering is the pricing model. They have a pay-as-you-go model, or a reservation-based instance, for one to three years. Microsoft’s interface is totally different from Amazon’s, and in our opinion might not be as straightforward as its competitor’s, but you will get used to it eventually.

Microsoft offers several types of VM instances, from economical burstable VMs to powerful memory-optimized instances. The pay-as-you-go model offers a per-hour cost, and this could add to the final bill for those services, so choose with care based on your needs. When it comes to Linux distributions, you can choose from the following: CentOS, Debian, RHEL, SLES for SAP, openSUSE Leap, and Ubuntu Server.

Azure has a very powerful SaaS offering too, and this will make it a good option if you use other Azure services. An example of how to deploy to Azure will be provided to you in [*Chapter 15*](B19682_15.xhtml#_idTextAnchor326), *Deploying to the Cloud with AWS* *and Azure*.

## Other strong IaaS offerings

DigitalOcean is another important player in the cloud market and it offers a strong IaaS solution. It has a straightforward interface and it helps you to create a cloud in a very short time. They call their VMs **droplets** and you create one in a matter of seconds. All you have to do is the following:

*   Choose the image (Linux distribution)
*   Select the plan, based on your **virtual CPU** (**vCPU**), memory, and disk space needs
*   Add storage blocks
*   Choose your data center region, authentication method (password or **Secure Shell** (**SSH**) key), and hostname
*   You can also assign droplets to certain projects you manage

DigitalOcean’s interface is better looking and much more user friendly than its competitors. Following the example of DigitalOcean, other IaaS providers—such as **Linode** and **Hetzner**—provide a slim and friendly interface for creating virtual servers.

Linode is another strong competitor in the cloud market, offering powerful solutions. Their VMs are called Linodes. The interface is somewhere between DigitalOcean and Azure, with regard to their ease of use and appearance.

Another strong player, at least in the European market, is Hetzner, a Germany-based cloud provider. They offer a great balance between resources and cost, and similar solutions to the others mentioned in this section. They provide an interface similar to DigitalOcean that is really easy to explore, and the cloud instance will be deployed in a matter of seconds.

Similar to the offerings of DigitalOcean, Linode, and Hetzner, there is a relatively new offering from Amazon (starting in 2017), called **Lightsail**. This service was introduced in order to offer clients an easy way to deploy **virtual private servers** (**VPSs**) or VMs in the cloud. The interface is similar to that seen from the competition, but it comes with the full Amazon infrastructure reliability on top. Lightsail provides several distributions, together with application bundles. Deploying on AWS, using Lightsail, becomes more straightforward. It is a useful tool to lure in new users wanting a quick and secure solution for delivering their web apps.

There are other solutions available, such as Google’s solution, called GCE, which is the IaaS solution from **Google Cloud Platform** (**GCP**). The GCP interface is very similar to the one on the Azure platform.

Important note

One interesting aspect of using GCP is that when you want to delete a project, the operation is not immediate, and the deletion is scheduled in one month’s time. This could be seen as a safety net if the deletion was not intentional, and you need to roll back the project.

In the next section, we will detail some of the PaaS solutions.

# Introducing PaaS solutions

PaaS is another form of cloud computing. Compared to IaaS, PaaS provides the hardware layer together with an application layer. The hardware and software are hosted by the CSP, with no need to manage them from the client side. Clients of PaaS solutions are, in the majority of cases, application developers. The CSPs that offer PaaS solutions are mostly the same as those that offer IaaS solutions. We have Amazon, Microsoft, and Google as the major PaaS providers. In the following subsections, we will discuss some PaaS solutions.

## Amazon Elastic Beanstalk

Amazon offers the **Elastic Beanstalk** service, whose interface is straightforward. You can create a sample application or upload your own, and Beanstalk takes care of the rest, from deployment details to load balancing, scaling, and monitoring. You select the AWS EC2 hardware instances to deploy on.

Next, we will discuss another major player’s offering: **Google** **App Engine**.

## Google App Engine

Google’s PaaS solution is Google App Engine, a fully managed serverless environment that is relatively easy to use, with support for a large number of programming languages. Google App Engine is a scalable solution with automatic security updates and managed infrastructure and monitoring. It offers solutions to connect to Google Cloud storage solutions and support for all major web programming languages such as Go, Node.js, Python, .NET, or Java. Google offers competitive pricing and an interface similar to the one we saw in their IaaS offering. Another major player with solid offerings is DigitalOcean, and we will discuss this next.

## DigitalOcean App Platform

DigitalOcean offers a PaaS solution in the form of **App Platform**. It offers a straightforward interface, a direct connection with your GitHub or GitLab repository, and a fully managed infrastructure. DigitalOcean is on the same level as big players such as Amazon and Google, and with App Platform, it manages infrastructure, provisioning, databases, application runtimes and dependencies, and the underlying operating system. It offers support for popular programming languages and frameworks such as Python, Node.js, Django, Go, React, and Ruby. DigitalOcean App Platform uses open cloud-native standards, with automatic code analysis, container creation, and orchestration. A distinctive competence of this solution is the free *starter tier*, for deploying up to three static websites. For prototyping dynamic web apps, there is a *basic tier*, and for deploying professional apps on the market, there is a *professional tier* available. DigitalOcean’s interface is pleasant and could be attractive to newcomers. Their pricing acts as an advantage too.

## Open source PaaS solutions

Besides ready-to-use solutions from providers listed previously, there are open source PaaS solutions provided by **Cloud Foundry**, **Red Hat OpenShift**, **Heroku**, and others, all of which we will not detail in this section. Nevertheless, the three mentioned previously are worth at least a short introduction, so let’s discuss them briefly:

*   **Red Hat OpenShift**: This is a container platform for application deployments. Its base is a Linux distribution (RHEL) paired with a container runtime and solutions for networking, registry, authentication, and monitoring. OpenShift was designed to be a viable, hybrid PaaS solution with total Kubernetes integration (Kubernetes will be covered briefly in the following section, *Introducing CaaS solutions*, and in more detail in [*Chapter 16*](B19682_16.xhtml#_idTextAnchor342), *Deploying Applications* *with Kubernetes*). OpenShift took advantage of the CoreOS acquisition (discussed later on) by providing some unique solutions. The new CoreOS Tectonic container platform is merging with OpenShift to bring to the user the best of both worlds.
*   **Cloud Foundry**: This is a cloud platform designed as an enterprise-ready PaaS solution. It is open source and can be deployed on different infrastructures, from on-premises to IaaS providers such as Google GCP, Amazon AWS, Azure, or OpenStack. It offers various developer frameworks and a choice of Cloud Foundry certified platforms, such as Atos Cloud Foundry, IBM Cloud Foundry, SAP Cloud Platform, SUSE Cloud Application Platform, and VMware Tanzu.
*   **Heroku**: Heroku is a Salesforce company, and the platform was developed as an innovative PaaS. It is based on a container system called Dynos, which uses Linux-based containers run by a container management system, designed for scalability and agility. It offers fully managed data services with support for Postgres, Redis, Apache Kafka, and Heroku Runtime, a component responsible for container orchestration, scaling, and configuration management. Heroku also supports a plethora of programming languages, such as Node.js, Ruby, Python, Go, Scala, Clojure, and Java.

PaaS has many solutions for developers, helping them create and deploy an application by taking away the burden of managing the infrastructure. As you might have learned by now, many of the solutions described in this section rely on the use of containers. This is why, in the next section, we will detail a subset of IaaS called CaaS, where we will introduce you to container orchestration and container-specialized operating systems.

# Introducing CaaS solutions

CaaS is a subset of the IaaS cloud service model. It lets customers use individual containers, clusters, and applications on top of a provider-managed infrastructure. CaaS can be used either on-premises or in the cloud, depending on the customer’s needs. In a CaaS model, the container engines and orchestration are provided and managed by the CSP. A user’s interaction with containers can be done either through an API or a web interface. The container orchestration platform used by the provider—mainly **Kubernetes** and **Docker**—is important and is a key differentiator between different solutions.

We covered containers (and VMs) in [*Chapter 11*](B19682_11.xhtml#_idTextAnchor231), *Working with Virtual Machines*, and[*Chapter 12*](B19682_12.xhtml#_idTextAnchor257)*, Managing Containers with Docker*, without giving any detailed information about orchestration or container-specialized micro operating systems. We will now provide you with some more details on those subjects.

## Introducing the Kubernetes container orchestration solution

Kubernetes is an open source project developed by Google to be used for the automatic deployment and scaling of containerized applications. It was written in the **Go** programming language. The name “Kubernetes” comes from Greek, and it refers to a ship’s helmsman or captain. Kubernetes is a tool for automating container management together with infrastructure abstraction and service monitoring.

Many newcomers confuse Kubernetes with Docker, or vice versa. They are complementary tools, each used for a specific purpose. Docker creates a container (like a box) in which you want to deploy your application, and Kubernetes takes care of the containers (or boxes) once the applications are packed inside and deployed. Kubernetes provides a series of services that are essential to running containers, such as service discovery and load balancing, storage orchestration, automated backups and self-healing, and privacy. The Kubernetes architecture consists of several components that are crucial for any administrator to know. We will break them down for you in the next section.

### Introducing the Kubernetes components

When you run Kubernetes, you mainly manage clusters of hosts, which are usually containers running Linux. In short, this means that when you run Kubernetes, you run clusters. Here is a list of the basic components found in Kubernetes:

*   A **cluster** is the core of Kubernetes, as its sole purpose is to manage lots of clusters. Each cluster consists of at least a control plane and one or more nodes, each node running containers inside pods.
*   A **control plane** consists of processes that control nodes. The components of a control plane are as follows:
    *   **kube-apiserver**: This is the API server at the frontend of the control plane
    *   **etcd**: This is the key-value store for all the data inside the cluster
    *   **kube-scheduler**: This looks for pods that have no assigned node and connects them to a node to run
    *   **kube-controller-manager**: This runs the controller processes, including the node controller, the replication controller, the endpoints controller, and the token controller
    *   **cloud-controller-manager**: This is a tool that allows you to link your cluster to your cloud provider’s API; it includes the node controller, the route controller, and the service controller
*   **Nodes** are either a VM or a physical machine running services needed for pods. The node components run on every node and are responsible for maintaining the running pods. The components are as follows:
    *   **kube-proxy**: This is responsible for network rules on each node
    *   **kubelet**: This makes sure that each container is running inside a pod
*   **Pods** are a collection of different containers running in the cluster. They are the components of the workload.

Kubernetes clusters are extremely complicated to master. Understanding the concepts around it needs a lot of practice and dedication. No matter how complex it is, Kubernetes does not do everything for you. You still have to choose the container runtime (supported runtimes are **Docker**, **containerd**, and **Container Runtime Interface** (**CRI**)-O), CI/CD tools, the storage solution, access control, and app services.

Managing Kubernetes clusters is out of the scope of this chapter, but you will learn about this in [*Chapter 16*](B19682_16.xhtml#_idTextAnchor342), *Deploying Applications* *with Kubernetes*. This short introduction was needed for you to understand the concepts and tools that Kubernetes uses.

Besides Kubernetes, there are several other container orchestration tools, such as **Docker Swarm**, **Apache Mesos**, and **Nomad** from HashiCorp. They are extremely powerful tools, used by many people around the world. We will not cover these in detail here, but we thought it would be useful to at least enumerate them at the end of this container orchestration section. In the next section, we will provide you with some information about container solutions in the cloud.

## Deploying containers in the cloud

You can use container orchestration solutions in the cloud, and the following offerings are essential for this:

*   **Amazon Elastic Container Service** (**ECS**): Amazon ECS is a fully managed service for orchestrating containers. It offers an optional, serverless solution (**AWS Fargate**) and is run inside by some of Amazon’s key services, which ensures that the tool is tested and is secure enough for anyone to use.
*   **Amazon Amazon Elastic Kubernetes Service** (**EKS**): Amazon also offers an EKS service for orchestrating Kubernetes applications. It is based on **Amazon EKS Distro** (**EKS-D**), which is a Kubernetes distribution developed by Amazon, based on the original open source Kubernetes. By using EKS-D, you can run Kubernetes either on-premises, on Amazon’s own EC2 instances, or on VMware vSphere VMs.
*   **Google Kubernetes Engine** (**GKE**): GKE offers pre-built deployment templates, with pod auto-scaling based on the CPU and memory usage. Scaling can be done across multiple pools, with enhanced security provided by GKE Sandbox. GKE Sandbox provides an extra layer of security by protecting the host kernel and running applications. Besides Google and Amazon, Microsoft offers a strong solution for container orchestration with AKS.
*   **Microsoft Azure Kubernetes Service** (**AKS**): AKS is a managed service for deploying clusters of containerized applications. As with the other providers, Microsoft offers a fully managed solution by handling resource maintenance and health monitoring. The AKS nodes use Azure VMs to run and support different operating systems, such as Microsoft Windows Server images. It also offers free upgrades to the newest available Kubernetes images. Among other solutions, AKS offers GPU-enabled nodes, storage volume support, and special development tool integration with Microsoft’s own Visual Studio Code.

After seeing some of the solutions available to deploy Kubernetes in the cloud and learning about the main components of Kubernetes and how they work, in the following section, we will discuss the importance of microservices in cloud computing.

## Introducing microservices

A **microservice** is an architectural style used in application delivery. Over time, application delivery evolved from a monolithic model toward a decentralized one, all thanks to the evolution of cloud technologies. Starting with the historical launch of AWS in 2006, followed by the launch of Heroku in 2007 and Vagrant in 2010, application deployment started to change too, in order to take advantage of the new cloud offerings. Applications moved from having a single, large, and monolithic code base to a model where each application would benefit from different sets of services. This would make the code base more lightweight and dependent on different services.

Let us explain how a **monolithic application** differs from a **microservices** **architecture** application:

*   A monolithic application model has all the functionalities inside a single process and it is deployed by simple replication on multiple servers. In comparison, a microservice architecture assumes that the application has its functionalities separated into different services. Those services are then distributed and scaled across different servers, depending on the user’s needs.
*   A microservices architecture has a modular-based approach. Each module will correspond to a specific service. Services work independently of one another and are connected through REST APIs based on the HTTP protocol. This means that each application functionality can be developed in different languages, depending on which one is better suited. This modular base can also take advantage of new container technologies.
*   A microservices architecture is known for rapidly delivering complex applications. It has no technology or language lock-in; it offers independent scaling and updates for each service and component, with no disturbance to other running services; and it has a fail-proof architecture. The microservices model can be adapted to existing monolithic applications by breaking them down into individual, modular services. There is no need to rewrite the entire application, only splitting the entire code base into smaller parts. Microservices are optimized for DevOps and CI/CD practices, thanks to their modular approach.

In the next section, we will introduce you to DevOps practices and tools.

# Introducing DevOps

**DevOps** is a culture. Its name comes from a combination of development and operations, and it envisions the practices and tools that are used to deliver rapidly. DevOps is about speed, agility, and time. We all know the phrase “time is money,” and this applies very well to the IT sector. The ability to deliver services and applications at a high speed can make the difference between being successful as a business and being irrelevant in the market.

DevOps is a model of cooperation between different teams involved in delivering services and applications. This means that the entire life cycle, from development and testing, up to deployment and management, is done by teams that are equally involved at every stage. The DevOps model assumes that no team is operating in a closed environment, but rather operates transparently in order to achieve the agility they need to succeed. There is also a different DevOps model whereby security and quality assurance teams are equally involved in the development cycle. It is called **DevSecOps**.

Crucial for the DevOps model are the automated processes that are created using specific tools. This mindset of agility and speed determined the rise of a new name associated with DevOps, and that is CI/CD. The CI/CD mindset assures that every development step is continuous, with no interruptions. To support this mindset, new automation tools have emerged. Perhaps the most widely known is the open source automation tool, **Jenkins**. This is a modular tool and can be extended with the use of plugins. The ecosystem around the application is quite large, with hundreds of plugins available to choose from. Jenkins is written in Java and was designed to automate software development processes, from building and testing, up to delivery. One of the assets of Jenkins is the ability to create a pipeline through the use of specialized plugins. A pipeline is a tool that adds support for CD as an automated process to the application life cycle. Jenkins can be used either on-premises or in the cloud. It is also a viable solution for use as a SaaS offering.

The DevOps philosophy is not only related to application deployment; healthy CD and CI are closely tied to the state of infrastructure. In this respect, tools for cloud management such as Ansible, Puppet, and Chef are extremely useful for managing the infrastructure that supports application deployments. This is why configuration and management at the infrastructure level is extremely important. In the next section, you will learn about cloud infrastructure management.

# Exploring cloud management tools

Today’s software development and deployment relies on a plethora of physical systems and VMs. Managing all the related environments for development, testing, and production is a tedious task and involves the use of automated tools. The most widely used solutions for cloud infrastructure management are tools such as **Ansible**, **Puppet**, and **Chef Infra**. All these configuration management tools are powerful and reliable, and we will reserve [*Chapter 17*](B19682_17.xhtml#_idTextAnchor359), *Infrastructure and Automation with Ansible*, to teach you how to use only one of them: Ansible. Nevertheless, we will briefly introduce you to all of them in this section.

## Ansible

Ansible is an open source project currently owned by Red Hat. It is considered a simple automation tool, used for diverse actions such as application deployment, configuration management, cloud provisioning, and service orchestration. It was developed in Python and uses the concept of nodes to define categories of systems, with a **control node** as the master machine running Ansible, and different **managed nodes** as other machines that are controlled by the master. All the nodes are connected over SSH and controlled through an application called an **Ansible module**. Each module has a specific task to do on the managed nodes, and when the task is completed, it will be removed from that node.

The way modules are used is determined by an Ansible playbook. The playbook is written in **YAML** (a recursive acronym for **YAML Ain’t Markup Language**), a language mostly used for configuration files. Ansible also uses the concept of **inventory**, where lists of the managed nodes are kept. When running commands on nodes, you can apply them based on the lists inside your inventory, based on **patterns**. Ansible will apply the commands on every node or group of nodes available in a certain pattern. Ansible is considered one of the easiest automation tools available. It supports Linux/Unix and Windows for client machines, but the master machine must be Linux/Unix.

## Puppet

Puppet is one of the oldest automation tools available. Puppet’s architecture is different from that of Ansible. It uses the concepts of **primary servers** and **agents**. Puppet works with infrastructure code written using **domain-specific language** (**DSL**) code specific to Puppet, based on the Ruby programming language. The code is written on the primary server, transferred to the agent, and then translated into commands that are executed on the system you want to manage. Puppet also has an inventory tool called **Facter**, which stores data about the agents, such as hostname, IP address, and operating system. Information stored is sent back to the primary server in the form of a **manifest**, which will then be transformed into a JSON document called a **catalog**. All the manifests are kept inside **modules**, which are tools that are used for specific tasks. Each module contains information in the form of code and data. This data is centralized and managed by a tool called **Hiera**. All the data that Puppet generates is stored inside databases and managed through APIs by every app that needs to manage it. Compared to Ansible, Puppet seems a lot more complex. Puppet’s primary server supports only Linux/Unix.

## Chef Infra

Chef Infra is another automation tool. It uses a client-server architecture. It uses the concepts of **cookbooks** and **recipes**. The main components are as follows:

*   **Chef Server**: This is similar to a hub that handles all the configuration data. It is mainly used to upload cookbooks to the Chef Client.
*   **Chef Client**: This is an application that is installed on every node from the infrastructure that you manage.
*   **Chef Workstation**: The Chef Workstation manages cookbooks that are used for infrastructure administration.

Chef Infra uses the same Ruby-based code similar to Puppet called DSL. The server needs to be installed on Linux/Unix, and the client supports Windows too.

All the automation and configuration tools presented in this section use different architectures but do the same thing, which is to provide an abstraction layer that defines the desired state of the infrastructure. Each tool is a different beast, having its own strengths and weaknesses. Chef Infra and Puppet might have a steeper learning curve with their Ruby/DSL-based code, while Ansible could be easier to approach due to its simpler architecture and use of the Python programming language. Nevertheless, you can’t go wrong with either one.

# Summary

In this chapter, we introduced you to cloud computing by showing you some of the most important concepts, tools, and solutions used. This should be enough for you to start learning about cloud technologies, which is a very vast and complex subject. For more details, please refer to the *Further* *reading* section.

We talked about cloud standards, a significant and largely overlooked subject, and about the main cloud types and services. You now have an idea of what each as-a-service solution means and what the main differences are between them. You know what the most important solutions are and how they are provided by the main players in this field: Amazon, Google, and Microsoft. We introduced you to container orchestration with Kubernetes and how it works. You learned about APIs and minimal container-specialized operating systems and the DevOps culture, microservices, and infrastructure automation tools. You learned a lot in this chapter, but keep in mind that all these subjects have only scratched the surface of cloud computing.

In the next chapter, we will introduce you to the more practical side of cloud deployments. You will learn how to deploy Linux on major clouds such as AWS, Azure, and GCP.

# Further reading

If you want to learn more about cloud technologies, please check out the following titles:

*   *OpenStack for Architects – Second Edition* by Ben Silverman and Michael Solberg, Packt Publishing
*   *Learning DevOps – Second Edition* by Mikael Krief, Packt Publishing
*   *Design Microservices Architecture with Patterns and Principles [Video]* by Mehmet Ozkaya, Packt Publishing
*   *Multi-Cloud Strategy for Cloud Architects – Second Edition* by Jeroen Mulder, Packt Publishing
*   *Architecting Cloud-Native Serverless Solutions*, Safeer CM, Packt Publishing