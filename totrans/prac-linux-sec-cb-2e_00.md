# Preface

When setting up a Linux system, security is supposed to be an important part of all stages. A good knowledge of the fundamentals of Linux is essential to implementing a good security policy on the machine.

Linux, as it ships, is not completely secure, and it is the responsibility of the administrator to configure the machine in a way such that it becomes more secure. *Practical Linux Security Cookbook* will work as a practical guide for administrators and help them configure a more secure machine.

If you want to learn about Kernel configuration, filesystem security, secure authentication, network security, and various security tools for Linux, this book is for you.

Linux security is a massive subject and not everything can be covered in just one book. Still, *Practical Linux Security Cookbook* will give you a lot of recipes to help you secure your machine.

# Who this book is for

*Practical Linux Security Cookbook* is intended for all those Linux users who already have knowledge of Linux filesystems and administration. You should be familiar with basic Linux commands. Understanding information security and its risks to a Linux system is also help you in understand the recipes more easily.

However, even if you are unfamiliar with information security, you will be able to easily follow and understand the recipes discussed.

Since *Practical Linux Security Cookbook* follows a practical approach, following the steps is very easy.

# What this book covers

[Chapter 1](f7bbe70d-3eb5-4946-b10a-46b36214b311.xhtml), *Linux Security Problem*, discusses the kinds of security that can be implemented for these exploits. Topics include preparing security policies and security controls for password protection and server security and performing vulnerability assessments of the Linux system. It also covers the configuration of sudo access.

[Chapter 2](1d1bfb75-2d27-457d-8fcc-4c24c4208b36.xhtml), *Configuring a Secure and Optimized Kernel*, focuses on the process of configuring and building the Linux kernel and testing it. Topics covered include requirements for building a kernel, configuring a kernel, kernel installation, customization, and kernel debugging. The chapter also discusses configuring a console using Netconsole.

[Chapter 3](dd9ad20f-3479-47df-8819-358a6d9354ca.xhtml), *Local Filesystem Security*, looks at Linux file structures and permissions. It covers topics such as viewing file and directory details, handling files and file permissions using chmod, and the implementation of an access control list. The chapter also gives readers an introduction to the configuration of LDAP.

[Chapter 4](4c086a0c-339e-45e3-bdbe-e2312cefb1b8.xhtml), *Local Authentication in Linux*, explores user authentication on a local system while maintaining security. Topics covered in this chapter include user authentication logging, limiting user login capabilities, monitoring user activity, authentication control definition, and also how to use PAM.

[Chapter 5](f12cf2b4-4121-48ed-9ff1-f4167a824547.xhtml), *Remote Authentication*, talks about authenticating users remotely on a Linux system. The topics included in this chapter are remote server access using SSH, disabling and enabling root login, restricting remote access when using SSH, copying files remotely over SSH, and setting up Kerberos.

[Chapter 6](65f508c6-0f3f-4727-bc3d-7de21ab9ce26.xhtml), *Network Security*, provides information about network attacks and security. It covers managing the TCP/IP network, configuring a firewall using IPtables, blocking spoofed addresses, and unwanted incoming traffic. The chapter also gives readers an introduction to configuring and using TCP Wrapper.

[Chapter 7](75ebf47a-39ee-4764-8f27-172d0c5718e7.xhtml), *Security Tools*, targets various security tools or software that can be used for security on a Linux system. Tools covered in this chapter include sXID, Portsentry, Squid proxy, OpenSSL server, Tripwire, Shorewall, OSSEC, Snort, and Rsync/Grsync.

[Chapter 8](e09be5f5-4e40-4c0a-a863-5fd5eb8f6172.xhtml), *Linux Security Distros*, introduces the readers to some of the famous Linux/Unix distributions of that have been developed in relation to security and penetration testing. The distros covered in this chapter include Kali Linux, PfSense, DEFT, NST, Security Onion, Tails, and Qubes.

[Chapter 9](5bb312f2-826f-4b39-aaac-4662fa569a67.xhtml), *Bash Vulnerability Patching*, explores the most famous vulnerability of the Bash shell, which is known as Shellshock. It gives readers an understanding of Shellshock's vulnerability and the security issues that can arise with its presence. The chapter also tells the reader how to use the Linux Patch Management system to secure their machine and also gives them an understanding of how patches are applied in a Linux system. It also gives an insight into other known Linux vulnerabilities.

[Chapter 10](d23eec36-90d8-498e-9e7b-22dee69eb713.xhtml), *Security Monitoring and Logging*, provides information on monitoring logs in Linux, on a local system as well as a network. Topics discussed in this chapter include monitoring logs using Logcheck, using Nmap for network monitoring, system monitoring using Glances, and using MultiTail to monitor logs. A few other tools are also discussed, which include Whowatch, stat, lsof, and strace. Readers also learn about network monitoring using IPTraf, Suricata and OpenNMS.

[Chapter 11](50841baf-717c-46e1-b177-ef5cdc195684.xhtml), *Understanding Linux Service Security*, helps the reader understand the commonly used services on Linux systems and the security concern related to each of these services. Services such as HTTPD, Telnet, and FTP, have been in use since long time and still, many administrators are not aware of the security concerns that each of them can cause, if not configured properly.

[Chapter 12](69dc587e-a6d3-4d64-959b-bb10408a7dfa.xhtml), *Scanning and Auditing Linux*, provides information about performing malware scan on Linux systems so as to find all malwares including rootkits. It also gives an insight into auditing using system services such as auditd and tools like ausearch and aureport. This chapter will help readers understand how to read through logs to learn what the system services are doing.

[Chapter 13](b872d5bc-dc7f-443d-be1e-5fb4229c5a9f.xhtml), *Vulnerability Scanning and Intrusion Detection,* will help readers perform vulnerability assessment on Linux machine using various tools and Linux distros like Security Onion, OpenVAS, and Nikto. Learn about network and server category vulnerabilities and also web based vulnerabilities. The chapter also helps readers to harden Linux systems using Lynis.

# To get the most out of this book

To get the most out of this book, readers should have a basic understanding of the Linux filesystem and administration. They should be aware of the basic commands of Linux, and knowledge about information security would be an added advantage.

This book will include practical examples on Linux security using inbuilt Linux tools as well as other available open source tools. As per the recipe, readers will have to install these tools if they are not already installed in Linux.

# Conventions used

There are a number of text conventions used throughout this book.

`CodeInText`: Indicates code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles. Here is an example: "Mount the downloaded `WebStorm-10*.dmg` disk image file as another disk in your system."

Any command-line input or output is written as follows:

```
$ mkdir css
$ cd css
```

**Bold**: Indicates a new term, an important word, or words that you see onscreen. For example, words in menus or dialog boxes appear in the text like this. Here is an example: "Select System info from the Administration panel."

Warnings or important notes appear like this.

Tips and tricks appear like this.

# Sections

In this book, you will find several headings that appear frequently (*Getting ready*, *How to do it...*, *How it works...*, *There's more...*, and *See also*).

To give clear instructions on how to complete a recipe, use these sections as follows:

# Getting ready

This section tells you what to expect in the recipe and describes how to set up any software or any preliminary settings required for the recipe.

# How to do it...

This section contains the steps required to follow the recipe.

# How it works...

This section usually consists of a detailed explanation of what happened in the previous section.

# There's more...

This section consists of additional information about the recipe in order to make you more knowledgeable about the recipe.

# See also

This section provides helpful links to other useful information for the recipe.

# Get in touch

Feedback from our readers is always welcome.

**General feedback**: Email `feedback@packtpub.com` and mention the book title in the subject of your message. If you have questions about any aspect of this book, please email us at `questions@packtpub.com`.

**Errata**: Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you have found a mistake in this book, we would be grateful if you would report this to us. Please visit [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the Errata Submission Form link, and entering the details.

**Piracy**: If you come across any illegal copies of our works in any form on the internet, we would be grateful if you would provide us with the location address or website name. Please contact us at `copyright@packtpub.com` with a link to the material.

**If you are interested in becoming an author**: If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, please visit [authors.packtpub.com](http://authors.packtpub.com/).

# Reviews

Please leave a review. Once you have read and used this book, why not leave a review on the site that you purchased it from? Potential readers can then see and use your unbiased opinion to make purchase decisions, we at Packt can understand what you think about our products, and our authors can see your feedback on their book. Thank you!

For more information about Packt, please visit [packtpub.com](https://www.packtpub.com/).