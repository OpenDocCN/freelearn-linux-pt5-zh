# Preface

SELinux can be seen as a daunting beast to tame. For many, it is considered to be a complex security system on the already versatile environment that Linux can be. But as with most IT-related services, it is the unfamiliarity with the technology that is causing the notion of having a complicated system.

It is, however, nothing like that. SELinux is not all that difficult to understand. If it were, then Linux distributions such as Red Hat Enterprise Linux wouldn't enable it by default.

To support everyone in their daily operations with SELinux-enabled systems, this book came to life. It contains numerous chapters on the various aspects of SELinux handling and policy development in a recipe-based approach that allows every person to quickly dive into the details and challenges that making a system more secure brings forth.

What this will not present are administration-related commands and examples. For that, I have written another better-suited SELinux resource, *SELinux System Administration*, *Packt Publishing*, which covers the system administration tasks of SELinux-enabled systems, such as dealing with SELinux Booleans and file context changes as well as an introduction to the SELinux technology.

This book is also not a reference for the SELinux policy language in all its glory. Although the most common statements will be mentioned and used several times, it should be noted that the SELinux policy language and its internal architecture has a much wider scope. For a good language and component reference, *The SELinux Notebook – The Foundations*, *Richard Haines*, is recommended. This resource is available online at [http://www.freetechbooks.com/the-selinux-notebook-the-foundations-t785.html](http://www.freetechbooks.com/the-selinux-notebook-the-foundations-t785.html).

# What this book covers

[Chapter 1](ch01.html "Chapter 1. The SELinux Development Environment"), *The SELinux Development Environment*, tells us how to set up the SELinux policy development environment through which further policy development can occur. We will look into a structured, reusable method for SELinux policy development and will create our first set of SELinux policy modules that are nicely integrated with the existing SELinux policies.

[Chapter 2](ch02.html "Chapter 2. Dealing with File Labels"), *Dealing with File Labels*, focuses on how file labels are set and managed. We will learn how to configure the SELinux policy ourselves as well as how to use and declare file contexts and assign the right context to the right type of resource.

[Chapter 3](ch03.html "Chapter 3. Confining Web Applications"), *Confining Web Applications*, covers the default confinement of the web server SELinux domain and explains how to enhance the existing policy to suit our needs. Additional SELinux support through the mod_selinux Apache module is also covered.

[Chapter 4](ch04.html "Chapter 4. Creating a Desktop Application Policy"), *Creating a Desktop Application Policy*, is the first chapter where an entirely new application domain and policy is written. We will look at how the policy needs to be structured and the policy rules that are needed in order to successfully and securely run the application.

[Chapter 5](ch05.html "Chapter 5. Creating a Server Policy"), *Creating a Server Policy*, follows the previous chapter's momentum but now with a focus on server services. This chapter targets the differences between desktop application policies and server policies, and we will develop a fully functioning SELinux policy module together with the necessary administrative policy interfaces needed to integrate the policy in a larger SELinux environment.

[Chapter 6](ch06.html "Chapter 6. Setting Up Separate Roles"), *Setting Up Separate Roles*, looks into the role-based access controls that SELinux offers. We create our own set of roles with the least privilege principle in mind. After considering the definition of SELinux users and roles, we then practice the management of these roles in larger environments.

[Chapter 7](ch07.html "Chapter 7. Choosing the Confinement Level"), *Choosing the Confinement Level*, inspects the different confinement levels that policies can use and how these are implemented on the system. We learn about the pros and cons of each confinement level and create our own policy set that provides the different levels.

[Chapter 8](ch08.html "Chapter 8. Debugging SELinux"), *Debugging SELinux*, scrutinizes the various methods available to debug SELinux behavior and policies. We acquire the necessary skills to work with the Linux auditing subsystem to generate additional logging and perform advanced queries against the SELinux policy. In this chapter, we also uncover why certain popular Linux debugging tools do not (properly) work on an SELinux-enabled system.

[Chapter 9](ch09.html "Chapter 9. Aligning SELinux with DAC"), *Aligning SELinux with DAC*, examines how SELinux can be used to enhance the existing Linux DAC restrictions. We deal with the various technologies available and how the SELinux policy can be augmented to work properly with those technologies.

[Chapter 10](ch10.html "Chapter 10. Handling SELinux-aware Applications"), *Handling SELinux-aware Applications*, considers the SELinux-aware applications and the interaction (and debugging difficulties) they have with the system and SELinux in general. We learn how to configure these applications' SELinux integration and how to debug the applications when things go wrong. This chapter also describes how to create our own SELinux-aware application.

# What you need for this book

As the book focuses on hands-on experience, it is seriously recommended to have an SELinux-enabled system at your disposal. Many distributions offer live environments that can be used to perform initial investigations with, but ensure that you pick one that can persist the changes made to the system.

An SELinux-enabled system should be using a recent set of SELinux libraries and user space utilities. This book is written based on Gentoo Hardened, running the SELinux user space libraries and utilities released in October 2013 (such as libselinux-2.2.2) with the reference policy released in March 2014\. The distribution itself is not that important, as everything in this book is distribution-independent, so it is well usable for Fedora and Red Hat Enterprise Linux, although the latter—at the time of writing, Version 6—is still using older versions of the SELinux user space libraries and utilities.

From an experience point of view, you should be well-versed in Linux system administration as SELinux policy development and integration requires good knowledge of the components that we are about to confine and protect. This book assumes that you are familiar with the Git version control system as an end user. This book also assumes basic knowledge of how SELinux works on a system.

# Who this book is for

This book is meant for Linux system administrators and security administrators who want to perform the following tasks:

*   Fine-tune the SELinux subsystem on their Linux systems
*   Develop SELinux policies for applications and users
*   Tightly integrate SELinux within their current processes

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "Using the `auditallow` statement, we can track SELinux policy decisions and assist in the development of policies and debugging of application behavior."

A block of code is set as follows:

```
write_files_pattern(syslogd_t, named_conf_t, named_conf_t)
allow syslogd_t named_conf_t:file setattr_file_perms;
```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```
policy_module(mysysadm, 0.1)
gen_require(`
  type sysadm_t;
')
logging_exec_syslog(sysadm_t)
```

Any command-line input or output is written as follows:

```
~# setsebool cron_userdomain_transition on
~# grep crond_t /etc/selinux/mcs/contexts/users/user_u
system_r:crond_t  user_r:user_t

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "Capabilities are well explained on Chris Friedhoff's **POSIX Capabilities & File POSIX Capabilities** page."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or may have disliked. Reader feedback is important for us to develop titles that you really get the most out of.

To send us general feedback, simply send an e-mail to `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, and mention the book title through the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide on [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the example code

You can download the example code files for all Packt books you have purchased from your account at [http://www.packtpub.com](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/support](http://www.packtpub.com/support), selecting your book, clicking on the **errata** **submission** **form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website, or added to any list of existing errata, under the Errata section of that title.

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.