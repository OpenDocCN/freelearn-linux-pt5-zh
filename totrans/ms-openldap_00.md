# Preface

The OpenLDAP directory server is a mature product that has been around (in one form or another) since 1995\. All of the major Linux distributions include the OpenLDAP server, and many major applications, both Open Source and proprietary, are directory aware, and can make use of the services provided by OpenLDAP. And yet the OpenLDAP server seems to be shrouded in mystery, known and understood only by the gurus and hackers. This book is meant not only to demystify OpenLDAP, but to give the system administrator and software developer a solid understanding of how to make use, in the practical realm, of OpenLDAP’s directory services.

OpenLDAP is an Open Source server that provides network clients with directory services. The directory server can be used to store organizational information in a centralized location, and make this information available to authorized applications. Client applications can connect to OpenLDAP using the Lightweight Directory Access Protocol (LDAP). They can then search the directory and (if they have appropriate access) modify and manipulate records in the directory. LDAP servers are most frequently used to provide network-based authentication services for users. But there are many other uses for an LDAP, including using the directory as an address book, a DNS database, an organizational tool, or even as a network object store for applications. We will look at some of these uses in this book.

The goal of this book is to prepare a system administrator or software developer for building a directory using OpenLDAP, and then employing this directory in the context of the network. To that end, this book will take a practical approach, emphasizing how to get things done. On occasion, we will delve into theoretical aspects of LDAP, but such discussions will only occur where understanding the theory helps us answer practical questions.

# What This Book Covers

In [Chapter 1](ch01.html "Chapter 1. Directory Servers and LDAP") we look at general concepts of directory servers and LDAP, cover the history of LDAP and the lineage of the OpenLDAP server, and finish up with a technical overview of OpenLDAP.

The next set of chapters focus on building directory services with OpenLDAP, and we take a close look at the OpenLDAP server in these chapters.

[Chapter 2](ch02.html "Chapter 2. Installation and Configuration") begins with the process of installing OpenLDAP on a GNU/Linux server. Once we have the server installed, we do the basic post-installation configuration necessary to have the server running.

[Chapter 3](ch03.html "Chapter 3. Using OpenLDAP") covers the basic use of the OpenLDAP server. We use the OpenLDAP command-line tools to add records to our new directory, search the directory, and modify records. This chapter introduces many of the key concepts involved in working with LDAP data.

[Chapter 4](ch04.html "Chapter 4. Securing OpenLDAP") covers security, including handling authentication to the directory, configuring Access Control Lists (ACLs), and securing network-based directory connections with Secure Sockets Layer (SSL) and Transport Layer Security (TLS).

[Chapter 5](ch05.html "Chapter 5. Advanced Configuration") deals with advanced configuration of the OpenLDAP server. Here, we take a close look at the various backend database options and also look at performance tuning settings, as well as the recently introduced technology of directory overlays.

[Chapter 6](ch06.html "Chapter 6. LDAP Schemas") focuses on extending the directory structure by creating and implementing LDAP schemas. Schemas provide a procedure for defining new attributes and structures to extend the directory and provide records tailor-made to your needs.

[Chapter 7](ch07.html "Chapter 7. Multiple Directories") focuses on directory replication and different ways of getting directory servers to interoperate over a network. OpenLDAP can replicate its directory contents from a master server to any number of subordinate servers. In this chapter, we set up a replication process between two servers.

[Chapter 8](ch08.html "Chapter 8. LDAP and the Web") deals with configuring other tools to interoperate with OpenLDAP. We begin with the Apache web server, using LDAP as a source of authentication and authorization. Next, we install phpLDAPadmin, a web-based program for managing directory servers. Then we look at the main features, and do some custom tuning.

The appendices include a step-by-step guide to building OpenLDAP from source ([Appendix A](apa.html "Appendix A. Building OpenLDAP from Source")), a guide to using LDAP URLs ([Appendix B](apb.html "Appendix B. LDAP URLs")), and a compendium of useful LDAP client commands ([Appendix C](apc.html "Appendix C. Useful LDAP Commands")).

# What You Need for This Book

To get the most from this book, you will need the OpenLDAP server software, as well as the client command-line utilities. These are all freely available (as Open Source software) in source code form from [http://openldap.org](http://openldap.org). However, you may prefer to use the version of OpenLDAP provided by your particular Linux or UNIX distribution.

While OpenLDAP will run on Linux, various versions of UNIX, MacOS X, and Windows 2000 and so on, the examples in this book use the Linux operating system.

Since the basic LDAP tools are command-line applications, you will need basic knowledge of getting around in a Linux/UNIX shell environment. The book does not cover the network protocol in detail, and it is assumed that the reader has a basic understanding of client-server network models. It is also assumed that the reader has a basic understanding of the structure of web and email services.

# Conventions

In this book you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

There are three styles for code. Code words in text are shown as follows: "The `telephoneNumber` attribute has two values, each representing a different phone number."

A block of code will be set as follows:

```
########
# ACLs #
########
access to attrs=userPassword
       by anonymous auth
       by self write
       by * none
```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items will be made bold:

```
directory /var/lib/ldap
# directory /usr/local/var/openldap-data
index objectClass sub,eq
index cn sub,eq
```

Any command-line input and output is written as follows:

```
 $ sudo slaptest -v -f /etc/ldap/slapd.conf

```

**New terms** and **important words** are introduced in a bold-type font. Words that you see on the screen, in menus or dialog boxes for example, appear in our text like this: "Clicking the **Advanced Search Form** link at the top of the simple search screen will load a search screen with more options".

### Note

Important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader Feedback

Feedback from our readers is always welcome. Let us know what you think about this book, what you liked or may have disliked. Reader feedback is important for us to develop titles that you really get the most out of.

To send us general feedback, simply drop an email to `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, making sure to mention the book title in the subject of your message.

If there is a book that you need and would like to see us publish, please send us a note in the **SUGGEST A TITLE** form on [www.packtpub.com](http://www.packtpub.com) or email `<[suggest@packtpub.com](mailto:suggest@packtpub.com)>`.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide on [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer Support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the Example Code for the Book

Visit [http://www.packtpub.com/support](http://www.packtpub.com/support), and select this book from the list of titles to download any example code or extra resources for this book. The files available for download will then be displayed.

## Errata

Although we have taken every care to ensure the accuracy of our contents, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in text or code—we would be grateful if you would report this to us. By doing this you can save other readers from frustration, and help to improve subsequent versions of this book. If you find any errata, report them by visiting [http://www.packtpub.com/support](http://www.packtpub.com/support), selecting your book, clicking on the **Submit Errata** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata are added to the list of existing errata. The existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with some aspect of the book, and we will do our best to address it.