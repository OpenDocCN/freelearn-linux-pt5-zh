# Preface

Gnu/Linux is the most important OS in the data center but how do you leverage it? How do you maintain and contain it? Many Gnu/Linux distributions try to answer these questions, but not all succeed. Red Hat Enterprise Linux is one that does answer these questions.

The next question is how do you, as a system administrator, manage a RHEL infrastructure? How do you deploy not just one system, but many? How do you make sure that it is secure and up to date? How can you monitor system components?

It may seem odd to you, but as a Red Hat Certified Engineer, I prefer the "lazy" approach—not as in "I can't be bothered," but as in "I like to do something once and do it good the first time and spend the rest of my time doing fun stuff."

In this book, I try to show you how to set up and configure systems, mainly by providing useful information to automate the setup, configuration, and management. This also explains the lack of the use of a GUI in this book. I'll be honest with you; I couldn't live without one on my laptop or desktop, but I do not believe servers should have a GUI. GUI-based applications tend not to have command-line counterparts, and I solemnly believe that if you cannot install, configure, manage, and maintain a piece of software through a script, it does not belong on a server.

This book does not pretend to be the de facto answer to all questions (that would be 42), but I do hope that you will learn something new and that, in turn, you will put this knowledge to good use. Remember, with great power, comes great responsibility!

# What this book covers

[Chapter 1](part0015_split_000.html#E9OE1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 1. Working with KVM Guests"), *Working with KVM Guests*, will not start by installing a basic RHEL system. It will start by introducing you to KVM if you don't already know it. You'll learn how to install and configure the KVM host and manage your KVM guests (the VMs). It will discuss the basics of adding resources on the fly, moving disks, and even moving the entire guest to another KVM host.

[Chapter 2](part0025_split_000.html#NQU21-501a83dd54944cb1bf060a2ce9fab11f "Chapter 2. Deploying RHEL "En Masse""), *Deploying RHEL "En Masse"*, will explore the ways of installing a RHEL system, introducing you to kickstart deployments, which are used to streamline automated system installs. If you want to orchestrate your environment, this chapter will lay out the basics for you to build on.

[Chapter 3](part0030_split_000.html#SJGS1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 3. Configuring Your Network"), *Configuring Your Network*, will explore `NetworkManager` tools to manage your network configuration, including advanced topics such as VLANs, link aggregation, and bridges. It will show you how to leverage its command-line tools to automate your system's network configuration during its deployment or afterwards, when all is installed.

[Chapter 4](part0037_split_000.html#1394Q1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 4. Configuring Your New System"), *Configuring Your New System*, will explain how to configure the basics, such as log retention, time, and your boot environment. It will also introduce you to the new systemd, which is SysVinit's replacement, and to monitoring and managing your services.

[Chapter 5](part0046_split_000.html#1BRPS1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 5. Using SELinux"), *Using SELinux*, will give you an overview, but a brief one, on how to manage and troubleshoot SELinux on your system. SELinux is becoming more and more important in today's world because of its security implementation, and it's better to know about it than to just turn it off because you can't handle it.

[Chapter 6](part0053_split_000.html#1IHDQ1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 6. Orchestrating with Ansible"), *Orchestrating with Ansible*, will tell you all about Ansible, which was recently bought by Red Hat. It will show you how to create simple playbooks that easily deploy new systems and how to manage your system's configuration.

[Chapter 7](part0060_split_000.html#1P71O1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 7. Puppet Configuration Management"), *Puppet Configuration Management*, will show you how to set up and configure Puppet. It will also give you a peek at its configuration management capacities.

[Chapter 8](part0066_split_000.html#1UU541-501a83dd54944cb1bf060a2ce9fab11f "Chapter 8. Yum and Repositories"), *Yum and Repositories*, will take a look at yum repositories, how you can create your own mirrors of the existing (Red Hat) repositories, and how to leverage it to keep your RHEL environment up to date without breaking a sweat.

[Chapter 9](part0073_split_000.html#25JP21-501a83dd54944cb1bf060a2ce9fab11f "Chapter 9. Securing RHEL 7"), *Securing RHEL 7*, will take security configuration and auditing problems a bit further. We'll explore how to configure setting up centralized secure authentication and privilege escalation. It will show you how you can operate a system that appears to be "hung" and trace the root cause of the event.

[Chapter 10](part0081_split_000.html#2D7TI1-501a83dd54944cb1bf060a2ce9fab11f "Chapter 10. Monitoring and Performance Tuning"), *Monitoring and Performance Tuning*, will show you the basics of easy performance tuning and how to monitor your system's resources.

# What you need for this book

The only thing you'll need for the recipes in this book is the Red Hat Enterprise Linux 7 Installation DVD, for which you can download an evaluation license from [https://access.redhat.com/downloads](https://access.redhat.com/downloads). All software used in this book is either available through the RHEL media or the yum repositories specified in the recipes.

# Who this book is for

This book is for the system administrators who want to learn about the new RHEL version and features that are included for management or certification purposes. Although this book provides a lot of information to get your Red Hat Certified System Administrator and/or Red Hat Certified Engineer certifications, it is by far a complete guide to get either!

To get the most of this book, you should have a working knowledge of the basic (RHEL) system administration and management tools.

# Sections

In this book, you will find several headings that appear frequently (Getting ready, How to do it, How it works, There's more, and See also).

To give clear instructions on how to complete a recipe, we use these sections as follows:

## Getting ready

This section tells you what to expect in the recipe, and describes how to set up any software or any preliminary settings required for the recipe.

## How to do it…

This section contains the steps required to follow the recipe.

## How it works…

This section usually consists of a detailed explanation of what happened in the previous section.

## There's more…

This section consists of additional information about the recipe in order to make the reader more knowledgeable about the recipe.

## See also

This section provides helpful links to other useful information for the recipe.

# Conventions

In this book, you will find a number of text styles that distinguish between different kinds of information. Here are some examples of these styles and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "We can include other contexts through the use of the `include` directive."

A block of code is set as follows:

```
node /^www[0-9]+\.critter\.be$/ {
}
node /^repo[0-9]+\.critter\.be$/ {
}
```

Any command-line input or output is written as follows:

```
~]# yum install -y /tmp/puppetlabs-release-el-7.noarch.rpm

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, for example, in menus or dialog boxes, appear in the text like this: "Clicking the **Next** button moves you to the next screen."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or disliked. Reader feedback is important for us as it helps us develop titles that you will really get the most out of.

To send us general feedback, simply e-mail `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, and mention the book's title in the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide at [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the color images of this book

We also provide you with a PDF file that has color images of the screenshots/diagrams used in this book. The color images will help you better understand the changes in the output. You can download this file from [https://www.packtpub.com/sites/default/files/downloads/RedHatEnterpriseLinuxServerCookbook_ColorImages.pdf](https://www.packtpub.com/sites/default/files/downloads/RedHatEnterpriseLinuxServerCookbook_ColorImages.pdf).

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you could report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **Errata Submission Form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website or added to any list of existing errata under the Errata section of that title.

To view the previously submitted errata, go to [https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support) and enter the name of the book in the search field. The required information will appear under the **Errata** section.

## Piracy

Piracy of copyrighted material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works in any form on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors and our ability to bring you valuable content.

## Questions

If you have a problem with any aspect of this book, you can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>`, and we will do our best to address the problem.