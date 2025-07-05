# Preface

This book covers the use of vSphere Replication and VMware Site Recovery Manager for making your vSphere environment recoverable in the event of a disaster. All the concepts and tasks covered in this book are for vSphere Replication 5.5 and VMware vCenter Site Recovery Manager 5.5.

# What this book covers

[Chapter 1](ch01.html "Chapter 1. Installing and Configuring vCenter Site Recovery Manager (SRM) 5.5"), *Installing and Configuring vCenter Site Recovery Manager (SRM) 5.5*, introduces you to the architecture of SRM and also guides you through the process of installing and configuring SRM to leverage array-based replication.

[Chapter 2](ch02.html "Chapter 2. Creating Protection Groups and Recovery Plans"), *Creating Protection Groups and Recovery Plans*, teaches you how to configure protection for virtual machines by creating Protection Groups and creating an orchestrated runbook with the help of Recovery Plans.

[Chapter 3](ch03.html "Chapter 3. Testing and Performing a Failover and Failback"), *Testing and Performing a Failover and Failback*, teaches you how to test the recovery plans that were created and also perform a Planned Migration, a Failover, and a Failback using them.

[Chapter 4](ch04.html "Chapter 4. Deploying vSphere Replication 5.5"), *Deploying vSphere Replication 5.5*, guides you through the steps required in deploying vSphere Replication Appliances and vSphere Replication Servers.

[Chapter 5](ch05.html "Chapter 5. Configuring and Using vSphere Replication 5.5"), *Configuring and Using vSphere Replication 5.5*, teaches you how to add target sites and enable replication on virtual machines and recover them. It will also teach you to configure vCenter SRM to leverage vSphere Replication engine.

# What you need for this book

If you were to follow along with each chapter by practicing the tasks in a lab, then you would need two ESXi hosts, two vCenter Servers, two SRM instances, and two storage array nodes with replication configured between them. This might sound like a lot of hardware, but all you need is VMware Workstation 9.x or 10.x and a Virtual Storage Appliance such as HP Store Virtual 9500 (LeftHand networks). You could get a trial license for HP Store Virtual by registering for one at HP's website. The ESXi hosts, vCenter Servers, vSphere Replication Appliances, SRM Servers, and the storage nodes would be virtual machines that are hosted using VMware Workstation.

# Who this book is for

This book is a guide for anyone who is keen on using vSphere Replication or vCenter Site Recovery Manager as a disaster recovery solution. This is an excellent handbook for solution architects, administrators, on-field engineers, and support professionals. Although the book assumes that the reader has some basic knowledge of data center virtualization using VMware vSphere, it can still be a very good reference for anyone who is new to virtualization.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "For instance, if you were protecting the SQL Server VMs, then you might name the protection group as `SQL Server Protection Group`."

Any command-line input or output is written as follows:

For instance, to run a batch script in `D:\demoscript.bat`, include the following command:

```
c:\windows\system32\cmd.exe /c d:\demoscript.bat

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "Click on **Recovery Plans** on the left pane."

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