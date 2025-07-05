# Preface

The Debian Linux distribution is the most stable distribution available, and it is used on more Internet web servers than any other operating system. While there are many instructional web pages and cookbooks written about Linux, and Debian Linux in particular, it is too easy for new users and seasoned administrators to get lost in the details. This book provides a broad overview, more of a what to than a how to, of Debian Linux administration. The chapters are designed to cover the subjects an administrator must address, and include background information, tips and suggestions, and basic knowledge and administration techniques. References are included that cover the various topics in greater detail than can be included in a book of this length.

Although oriented towards the current Debian stable distribution, the subjects covered are useful for any Linux administrator to know. As for the lack of numerous, detailed examples, I apologize. It is impossible in a book of this length to go as far into details as I would have liked. Fortunately, the Debian Project provides excellent guides and references, as well as online web pages that are pointed out in the text.

# What this book covers

[Chapter 1](ch01.html "Chapter 1. Debian Basics for Administrators"), *Debian Basics for Administrators*, covers what distinguishes Debian from other Linux distributions, and delves into the background of the Debian Project and free software in general.

[Chapter 2](ch02.html "Chapter 2. Filesystem Layout"), *Filesystem Layout*, covers the two primary methods used to boot Intel 32- and 64-bit systems, the various Linux filesystem formats, disk partitioning, and data protection using disk, partition, and directory-based encryption.

[Chapter 3](ch03.html "Chapter 3. Package Management"), *Package Management*, covers the basics of Debian package management, including the management utilities and updating your system.

[Chapter 4](ch04.html "Chapter 4. Basic Package Configuration"), *Basic Package Configuration*, covers common software configuration techniques, including the location of files and documentations, and trends in Debian configuration.

[Chapter 5](ch05.html "Chapter 5. System Management"), *System Management*, covers important system management topics, including startup and shutdown, networking, filesystem maintenance, and display managers.

[Chapter 6](ch06.html "Chapter 6. Basic System Security"), *Basic System Security*, covers security issues important for system safety, including special packages available to assist in installing additional security software, firewall tools, and intrusion detection.

[Chapter 7](ch07.html "Chapter 7. Advanced System Management"), *Advanced System Management*, briefly covers advanced management topics including remote backups, distributed configuration management, and clustering. It also includes coverage of Webmin, a web-based administration tool that is compatible with nearly all Linux installations.

# What you need for this book

Although software is not required, this book covers the Debian 7 Linux distribution. All software referred to in this book, with the exception of Webmin, is available in the Debian stable release, available for download from the Debian Project web site ([http://www.debian.org/](http://www.debian.org/)). It is also available on CD, DVD, and Blu-ray Discs from vendors mentioned on that site. Webmin software is available from its own site ([http://www.webmin.com/](http://www.webmin.com/)).

Access to the Internet is required if you are going to download the software, or if you wish to follow up with the various reference material and other documents mentioned in the book. In particular, beginners are encouraged to become familiar with the Debian installation guide ([http://www.debian.org/releases/stable/installmanual](http://www.debian.org/releases/stable/installmanual)) and the reference manual ([http://www.debian.org/doc/manuals/debian-reference/](http://www.debian.org/doc/manuals/debian-reference/)), which are also available as documentation packages in the Debian distribution.

# Who this book is for

This book is for users and administrators who are new to Debian, or for seasoned administrators who are switching to Debian from another Linux distribution. A basic knowledge of Linux or Unix systems is assumed. Since the book is a high-level guide, more of a what to than a how to, the reader should be willing to go to the referenced material for further details and practical examples.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text are shown as follows: "Usually, this is added to a separate `webmin.list` file in `/etc/apt/sources.list.d`."

Any command-line input or output is written as follows:

```
# deb cdrom:[Debian GNU/Linux 7.0.0 "Wheezy" - Official amd64 \ NETINST Binary-1 20130504-14:43]/ stable main

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "Often, this is as simple as providing a standard configuration, such as Apache's simple **It works!** page."

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

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **errata submission form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded on our website, or added to any list of existing errata, under the Errata section of that title. Any existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.