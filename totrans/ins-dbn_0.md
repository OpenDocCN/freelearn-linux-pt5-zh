# Preface

APT, one of the technologies that made Debian (and its derivatives, such as Ubuntu) highly popular, became mainstream for production deployments during the same time span that web applications broke through the barriers of enterprise fitness, reliability, and scalability.

The flexibility of Debian systems makes it very appealing for web DevOps and modern web apps. This book tries to simplify deployment and reduce time-to-market, while providing a solid foundation for you to grow your Debian sysadmin practices.

# What this book covers

In the first part of the book we’ll help you choose the right flavor of Debian, install it, and prepare for multiple and massive installations, should you need it. We will also guide you through the initial APT setup, installing the stack, and configuring storage and frameworks.

In the second part of the book, we harden the installation, analyze scalability paths, and learn how to effectively maintain your system, including backup/restore and performance. We also provide a future look at cloud and incident response, which will help you get the most out of your installation in the long run.

*Choosing the right flavor of Debian (Simple)* explains how Debian organizes software, the architectures, and installation methods, and indicates a set of criteria for system administrators to choose and get the right media.

*Installing Debian GNU/Linux (Simple)* goes through the installation process, including partitioning, networking, and using tasksel to install an initial set of packages to work with.

*Making Debian GNU/Linux installations scalable (Medium)* discusses methods to automate and scale Debian installations, such as debconf preseeding.

*Preparing the APT packaging system for your environment (Simple)* explains how to configure APT and use APT tools, such as apt-get for package management.

*Installing your application platform stack (Simple)* goes through the installation of an Apache or Nginx stack, as well as PHP and other components, and the databases—all in the Debian way.

*Setting up your storage, security, and permissions (Simple)* goes through the mount-level and user-level options to implement a security strategy for your application storage.

*Setting up your database/data storage (Medium)* explores how to configure the databases and their underlying storage.

*Configuring your programming language libraries (Medium)* discusses the Debian way to install a framework or a set of libraries, where it adds value, and where it needs administering.

*Setting up secure remote support options (Simple)* goes through the simple but necessary changes, necessary on a Debian system, in order to facilitate a more secure working environment.

*Keeping your system up-to-date (Simple)* teaches us that with thousands of developers maintaining the most exciting open source projects, it’s important to know how to and when to install updates on Debian.

*Backing up your environment (Medium)* describes a backup and restore strategy from an outcome-oriented vision using simple tools and dedicated backup software.

*Restoring your environment (Simple)* discusses the scenarios where fast restoring of data is critical to the application.

*Preparing for common security scenarios (Medium)* tackles several of the most common security scenarios and how to move from reactive to proactive security.

*Reading logs and troubleshooting your setup (Simple)* goes through the most important logfiles and essential skills to troubleshoot, by interpreting them.

*Using proxies, caches, and clusters to scale your architecture (Advanced)* discusses architectural scenarios for future growth of the application, including proxies, caches, and clusters, and when they can add value.

*Consuming Windows Azure Cloud Services (Medium)* provides a specific example of how to consume public cloud services for extending the application.

*Responding to security incidents (Advanced)* discusses several steps and outcomes in the face of a security incident.

*Monitoring your server’s operation (Medium)* goes through several of the most common data points for a web-based application, and how to collect, compare, and process them.

*Optimizing your solution performance (Advanced)* discusses not only the low-hanging fruit for web application performance improvement, but also the potentially bigger improvements of fundamental debugging and optimization.

# What you need for this book

You need hardware, either a server or hypervisor software that supports Linux-based operating systems to install Debian. You should plan for at least 2 GB of disk space, and additional space for your database and application, including static files. You also need a broadband Internet connection to download the installation media as well as any additional software packages.

# Who this book is for

Readers should be familiar with the essential concepts of the Linux system’s administration, such as how to work with files and permissions, command line instructions, and how service and processes work in Linux, although the chapters are not technically demanding. While the book does not require previous knowledge on the APT system, it does expect readers to understand how the particular configuration files for the services Apache, Nginx, MySQL, PostgreSQL, and so on, work.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text are shown as follows: “You can always delete those partitions and give space back to `/var` and `/tmp`.”

A block of code is set as follows:

```
<?php
  $mc = new Memcached();
  $mc->addServer(“localhost”, 11211);
  $value = file_get_contents(‘/var/www/icon.jpg’);
  $mc->set(“/icon.jpg”, $value);
?>
```

Any command-line input or output is written as follows:

```
chown –R www-data:www-data /var/www # resets owner and group to www-data

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: “. You might choose **Graphical Install**, which will run you through the same prompts but with mouse support, colors, buttons and scrollbars.”.

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or may have disliked. Reader feedback is important for us to develop titles that you really get the most out of.

To send us general feedback, simply send an e-mail to `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, and mention the book title via the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide on [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the example code

You can download the example code files for all Packt books you have purchased from your account at [http://www.packtpub.com](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **errata** **submission** **form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded on our website, or added to any list of existing errata, under the Errata section of that title. Any existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.