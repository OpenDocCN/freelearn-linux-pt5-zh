# Preface

Welcome to *CentOS System Administration Essentials*. My name is Andrew Mallett, and I will be offering you expert guidance and tuition, enabling you with the skills to tame this powerful and popular Linux distribution. I have chosen to write about CentOS primarily as it will not cost you to use it, neither while learning nor during production. Additionally, CentOS closely follows the Red Hat Enterprise Linux distribution, so the skills that you learn and develop here can be put to good use across both CentOS and Red Hat. Should you be interested, your reading can act as an investment in your career by pursuing the Red Hat certification paths. Although not directly written to fit into any existing curricula, the Red Hat exams are all based on practical exercises, so the more you know and understand about the operation of Linux, the better.

CentOS stands for Community Enterprise Operating System, and even though community is such a small word, it encompasses so much. The support emanates from the community, via fora and the Linux community, to help develop the services and applications, and provide remedies to bugs that occur. The community has taken ownership of this distribution. The distribution collectively becomes stronger with the continued involvement of a growing community.

While we talk of community, I would like to thank Say Mistage (available on Twitter at `@sayomgwtf`) for her inspiration and doodles.

Writing about an Enterprise Linux distribution is important as we see the increase in the number of organizations deploying Linux and, as a result, require knowledgeable professionals to manage these systems. In 2013, the Linux Foundation with Dice, a specialist recruitment company, surveyed many large organizations and found the following results:

*   93 percent of the organizations polled were looking to employ Linux professionals
*   91 percent of hiring managers reported that they found it difficult to find skilled Linux administrators
*   As a side note to this, it was additionally noted that salaries for Linux professionals had increased by 9 percent during the previous 12 months

With such confidence in Linux within so many organizations, the focus of this book has to be commercially driven for both myself and you, the reader. I want you to be able to improve your career prospects as well as your Linux knowledge.

Enterprise Linux distributions such as CentOS, Red Hat, Debian, and SUSE Enterprise Linux generally do not deploy the latest and greatest bleeding edge technology that you might find in home or enthusiast-oriented distributions such as Fedora or openSUSE. Rather, they allow these to be development platforms to hone and perfect the software before migrating it to the enterprise platforms some months or even years later. Enterprise Linux has to be dependable, reliable, and resilient. On top of this, it must be well supported by both the organization deploying it, as well as the backend support coming from the community or paid support teams. The very latest in software development does not lend itself well to this by definition; as they are the most recent, the knowledge of these advancements, as well as their best practices, will without a doubt take time to evolve and develop.

# What this book covers

[Chapter 1](ch01.html "Chapter 1. Taming vi"), *Taming vi*, will make sure that you are fully versed in the shortcuts that exist to make your shell quickly navigable before entering into the realms of mastering vi. You may have some experience with vi but most often, I find that the experience has not been a good one. I am going to make sure that you are the master of vi and not vice versa.

[Chapter 2](ch02.html "Chapter 2. Cold Starts"), *Cold Starts*, is all about understanding the boot process in CentOS and learning how to not only modify the GRUB menu to make it more secure, but also how to use the GRUB command line to debug and repair boot issues. We will include a little boot splashing with Plymouth as well as explain when the root filesystem is not actually the root filesystem.

[Chapter 3](ch03.html "Chapter 3. CentOS Filesystems – A Deeper Look"), *CentOS Filesystems – A Deeper Look*, tells us that we have files and directories but they are all just different file types. However, when it comes to links, pipes, and sockets, we will discuss what they are and how they are used. Regarding links, we will discuss what is the difference between a hard and soft link. Let's also challenge the traditional filesystem design; you may have worked with logical volumes manager (LVM) in the past, but let me tell you just how last century that is. You are going to be blown away by the power and ease of your enterprise filesystem management using BTRFS, pronounced as Better FS.

[Chapter 4](ch04.html "Chapter 4. YUM – Software Never Looked So Good"), *YUM – Software Never Looked So Good*, gets you to grips with YUM repositories and software management; you are going to love this. You will learn how to download packages without installing them, thus allowing you to easily distribute packages in your enterprise. If this is not good enough, then you'll learn how to set up a local repository to share packages across your LAN and create your own RPMs.

[Chapter 5](ch05.html "Chapter 5. Herding Cats – Taking Control of Processes"), *Herding Cats – Taking Control of Processes*, tells us that too often, administrators, without the insight that you and I have, will leave services running that aren't required, and do not understand the tools they have to manage processes. You will learn here to control services and processes using upstart and traditional service scripts as well as become homicidal with the kill and pkill weapons of choice.

[Chapter 6](ch06.html "Chapter 6. Users – Do We Really Want Them?"), *Users – Do We Really Want Them?*, tells us, of course, that we do not want them (users) on our system, but it is often dictated, so we have little choice. Rather than be grumpy about this, you will learn how to manage users with a smile and keep them on a tight rein.

[Chapter 7](ch07.html "Chapter 7. LDAP – A Better Type of User"), *LDAP – A Better Type of User*, tells us that rather than having silos of users and groups on each machine, it is better to get back on the golf course by spending more time improving the system and less time managing users. Adding users to a central directory and sharing them across all systems as required is your gateway to freedom.

[Chapter 8](ch08.html "Chapter 8. Nginx – Deploying a Performance-centric Web Server"), *Nginx – Deploying a Performance-centric Web Server*, tells us that commonly, Linux administrators and publications concentrate on the Apache web server; I will introduce you to the new kid on the block, Nginx (pronounced Engine X). Introduced in 2004, Nginx is rapidly taking market share from Apache and has already surpassed IIS in a number of deployed web servers worldwide. We will deploy Nginx and PHP.

[Chapter 9](ch09.html "Chapter 9. Puppet – Now You Are the Puppet Master"), *Puppet – Now You Are the Puppet Master*, shifts our focus from Linux in the enterprise to taking control of your enterprise systems with the renowned Puppet software from Puppet Labs. Central configuration control is as good as centralized user management in giving you more time to spend on the golf course, not that I want you to think that golf dominates my life.

[Chapter 10](ch10.html "Chapter 10. Security Central"), *Security Central*, introduces you to Pluggable Authentication Modules (PAM). It is your friend and will help you manage when and how users connect. SELinux, again, is a friend, albeit a temperamental one. When treated well, it will help you ensure correct use of your system. You will learn how to harden your Linux system and gain a set of best practices!

[Chapter 11](ch11.html "Chapter 11. Graduation Day"), *Graduation Day*, tells us that as we prepare to leave with our newfound skills, we will remind ourselves the need for security and adhere to the best practices. We can revisit some of the products that we have seen before, such as Puppet and Nginx, and outline some industry-recognized guidelines for the deployment of these services, along with some of the new features of CentOS 7.

# What you need for this book

You will be expected to have knowledge about working with Linux and look to fast-track that knowledge to an expert level. Working along with this book and the exercises therein is recommended and encouraged. Although this book can be used as a "read and learn", I would recommend "read, try, and learn for life". The try bit in the middle is essential to any real understanding and knowledge; this is a pedagogy that has been tried and tested across ages.

At the time of writing this book, CentOS version 6.5 is released, although any version of CentOS is acceptable for most of the exercises, including later versions. Versions of CentOS can be downloaded from [http://wiki.centos.org/Download](http://wiki.centos.org/Download). It is free and open to use, as you will see, under the terms of the GPL license. CentOS 6.5 supports updates free of charge up to November 30, 2020.

# Who this book is for

I think it is fair to say that I know Linux, and more importantly, how to keep you engaged. I will deliver my knowledge to you in a way that is designed to help you understand and remember, by breaking down complex ideas into easy-to-consume nuggets of wisdom, enabling you to grow in knowledge and confidence with the turn of every page. We will concentrate on the power and ease of use of the command line. For instance, let me ask you this question:

What was the date 73 days ago?

I am surprised that you do not know. The Linux command line knows, simply by executing the following command:

```
$ date --date "73 days ago"

```

This book has been written to target those Linux administrators with some level of knowledge and who wish to gain further experience and are not frightened of getting their hands dirty using the command-line shell.

Understanding the power of the Linux command line and being able to master it with little enhancements like these will be your key to success as a Linux administrator. This is where I will differentiate this book from others that you may see. You may also want to view my YouTube channel at [http://www.youtube.com/theurbanpenguin](http://www.youtube.com/theurbanpenguin), where I have created over 700 tutorials on various products that interest mostly Linux with a lot of scripting and programming too.

Alternatively, you can visit my own site at [http://theurbanpenguin.com](http://theurbanpenguin.com), where the content is better organized.

# Conventions

In this book, you will find a number of text styles that distinguish among different kinds of information. Here are some examples of these styles and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "Getting the `.vimrc` setup the way you like."

A block of code is set as follows:

```
default=0
timeout=5
hiddenmenu
password --md5 <password-hash>
```

Any command-line input or output is written as follows:

```
# vi /etc/httpd/conf/httpd.conf
# service httpd restart
w3m localhost

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, for example, in menus or dialog boxes, appear in the text like this: "From the main welcome page, we should choose the **Users and Groups** tab and then select the **Search** button."

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

We also provide you with a PDF file that has color images of the screenshots/diagrams used in this book. The color images will help you better understand the changes in the output. You can download this file from: [https://www.packtpub.com/sites/default/files/downloads/5920OS_coloredimages.pdf](https://www.packtpub.com/sites/default/files/downloads/5920OS_coloredimages.pdf).

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you could report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **Errata Submission Form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website or added to any list of existing errata under the Errata section of that title.

To view the previously submitted errata, go to [https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support) and enter the name of the book in the search field. The required information will appear under the **Errata** section.

## Piracy

Piracy of copyrighted material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works in any form on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors and our ability to bring you valuable content.

## Questions

If you have a problem with any aspect of this book, you can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>`, and we will do our best to address the problem.