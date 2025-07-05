# Preface

CoreOS is a new breed of the Linux operating system and is optimized to run Linux containers, such as Docker and rkt. It has a fully automated update system, no package manager, and a fully clustered architecture.

Whether you are a Linux expert or just a beginner with some knowledge of Linux, this book will provide you with step-by-step instructions on installing and configuring CoreOS servers as well as building development and production environments. You will be introduced to the new CoreOS rkt Application Containers runtime engine and Google's Kubernetes system, which allows you to manage a cluster of Linux containers as a single system.

# What this book covers

[Chapter 1](part0014.xhtml#aid-DB7S1 "Chapter 1. CoreOS – Overview and Installation"), *CoreOS – Overview and Installation*, contains a brief CoreOS overview what CoreOS is about.

[Chapter 2](part0018.xhtml#aid-H5A41 "Chapter 2. Getting Started with etcd"), *Getting Started with etcd*, explains what etcd is and what it can be used for.

[Chapter 3](part0025.xhtml#aid-NQU21 "Chapter 3. Getting Started with systemd and fleet"), *Getting Started with systemd and fleet*, covers an overview of systemd. This chapter tells you what fleet is and how to use it to deploy Docker containers.

[Chapter 4](part0029.xhtml#aid-RL0A1 "Chapter 4. Managing Clusters"), *Managing Clusters*, is a guide to setting up and managing a cluster.

[Chapter 5](part0033.xhtml#aid-VF2I2 "Chapter 5. Building a Development Environment"), *Building a Development Environment*, shows you how to set up the CoreOS development environment to test your Application Containers.

[Chapter 6](part0037.xhtml#aid-1394Q1 "Chapter 6. Building a Deployment Setup"), *Building a Deployment Setup*, helps you set up code deployment, the Docker image builder, and the private Docker registry.

[Chapter 7](part0040.xhtml#aid-164MG2 "Chapter 7. Building a Production Cluster"), *Building a Production Cluster*, explains the setup of the CoreOS production cluster on the cloud.

[Chapter 8](part0046.xhtml#aid-1BRPS1 "Chapter 8. Introducing CoreUpdate and Container/Enterprise Registry"), *Introducing CoreUpdate and Container/Enterprise Registry*, has an overview of free and paid CoreOS services.

[Chapter 9](part0051.xhtml#aid-1GKCM2 "Chapter 9. Introduction to CoreOS rkt"), *Introduction to CoreOS rkt*, tells you what rkt is and how to use it.

[Chapter 10](part0055.xhtml#aid-1KEEU1 "Chapter 10. Introduction to Kubernetes"), *Introduction to Kubernetes*, teaches you how to set up and use Kubernetes.

# What you need for this book

For this book, you will need a Linux-powered system or an Apple Mac, and a Google Cloud account to run the examples covered. You will also require the latest versions of VirtualBox and Vagrant to run the scripts.

# Who this book is for

This book will benefit any Linux/Unix system administrator. Any person with even a basic knowledge of Linux/Unix will have an advantage when using this book.

This book is also for system engineers and system administrators who are already experienced with network virtualization and want to understand how CoreOS can be used to develop computing networks for the deployment of applications and servers. They must have a proper knowledge of the Linux operating system and Application Containers, and it is better if they have used a Linux distribution for the purpose of development or administration before.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text are shown as follows: "We can include other contexts through the use of the `include` directive."

A block of code is set as follows:

```
  etcd2:
    name: core-01
    initial-advertise-peer-urls: http://$private_ipv4:2380
    listen-peer-urls: http://$private_ipv4:2380,http://$private_ipv4:7001
    initial-cluster-token: core-01_etcd
    initial-cluster: core-01=http://$private_ipv4:2380
    initial-cluster-state: new
    advertise-client-urls: http://$public_ipv4:2379,http://$public_ipv4:4001
    listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
  fleet:
```

Any command-line input or output is written as follows:

```
$ git clone https://github.com/coreos/coreos-vagrant/

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "We should see this output in the **Terminal** window."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or disliked. Reader feedback is important for us as it helps us develop titles that you will really get the most out of.

To send us general feedback, simply e-mail `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>` and mention the book's title in the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide at [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the example code

You can download the example code files from your account at [http://www.packtpub.com](http://www.packtpub.com) for all the Packt Publishing books you have purchased. If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you could report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **Errata Submission Form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website or added to any list of existing errata under the Errata section of that title.

To view the previously submitted errata, go to [https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support) and enter the name of the book in the search field. The required information will appear under the **Errata** section.

## Piracy

Piracy of copyrighted material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works in any form on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.