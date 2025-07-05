# Preface

This book will show you how to get the most from your Linux computer. It describes how to perform common tasks such as finding and searching files, explains complex system administration activities such as monitoring and tuning a system, and discusses networks, security, distribution, and how to use the cloud.

Casual users will enjoy recipes for reformatting their photos, downloading videos and sound files from the Internet, and archiving their files.

Advanced users will find the recipes and explanations that solve complex issues, such as backups, revision control, and packet sniffing, useful.

Systems administrators and cluster managers will find recipes for using containers, virtual machines, and the cloud to make their job easier.

# What this book covers

[Chapter 1](195d920d-33c2-41d6-bd33-37d75f9c37f1.xhtml), *Shell Something Out*, explains how to use a command line, write and debug bash scripts, and use pipes and shell configuration.

[Chapter 2](36986eeb-141a-496a-a6b1-4f78f612c14e.xhtml), *Have a Good Command*, introduces common Linux commands that can be used from the command line or in bash scripts. It also explains how to read data from files; find files by name, type, or date; and compare files.

[Chapter 3](19219dcb-e034-408e-8fe2-db6daa9278c8.xhtml), *File In, File Out*, explains how to work with files, including finding and comparing files, searching for text, navigating directory hierarchy, and manipulating image and video files.

[Chapter  4](22424a9e-fea7-49de-9589-ea32aeb0b829.xhtml), *Texting and Driving*, explains how to use regular expressions with `awk`, `sed`, and `grep`.

[Chapter 5](4556e100-4227-4900-91d6-9a2acd334bf3.xhtml), *Tangled Web? Not At All!*, explains web interactions without a browser! It also explains how to script to check your website for broken links and download and parse HTML data.

[Chapter 6](534a28c5-20f8-442e-bef2-517864965f78.xhtml), *Repository Management*, introduces revision control with Git or Fossil. Keep track of the changes and maintain history.

[Chapter 7](3fc45121-c541-4c47-90ec-4db14dc7a60e.xhtml), *The Backup Plan,* discusses traditional and modern Linux backup tools. The bigger the disk, the more you need backups.

[Chapter 8](5ba784d5-fa8b-4840-b4c5-cac906e484f9.xhtml), *The Old-Boy Network*, explains how to configure and debug network issues, share a network, and create a VPN.

[Chapter 9](39e9cad3-701a-48c5-9b88-59e8b7c0ce41.xhtml), *Putting on the Monitor's Cap*, helps us know what your system is doing. It also explains how to track disk and memory usage, track logins, and examine log files.

Chapter 10, *Administration Calls*, explains how to manage tasks, send messages to users, schedule automated tasks, document your work, and use terminals effectively.

[Chapter 11](5bf336a4-5338-4a7a-b27e-1ebecac4704d.xhtml), *Tracing the Clues*, explains how to snoop your network to find network issues and track problems in libraries and system calls.

[Chapter 12](5c74c943-1155-4720-a3cb-f4740f691f8c.xhtml), *Tuning a Linux System*, helps us understand how to make your system perform better and use memory, disk, I/O, and CPU efficiently.

[Chapter 13](aebd8926-4121-4f45-9f45-d92d3f97ae91.xhtml), *Containers, Virtual Machines, and the Cloud*, explains when and how to use containers, virtual machines, and the cloud to distribute applications and share data.

# What you need for this book

The recipes in this book run on any Linux-based computer—from a Raspberry Pi to IBM Big Iron.

# Who this book is for

Everyone, from novice users to experienced admins, will find useful information in this book. It introduces and explains both the basic tools and advanced concepts, as well as the tricks of the trade.

# Sections

In this book, you will find several headings that appear frequently (*Getting ready*, *How to do it...*, *How it works...*, T*here's more...*, and *See also*).

To give clear instructions on how to complete a recipe, we use these sections as follows:

# Getting ready

This section tells you what to expect in the recipe, and it describes how to set up any software or any preliminary settings required for the recipe.

# How to do it…

This section contains the steps required to follow the recipe.

# How it works…

This section usually consists of a detailed explanation of what happened in the previous section.

# There's more…

This section consists of additional information about the recipe in order to make the reader more knowledgeable about the recipe.

# See also

This section provides helpful links to other useful information for the recipe.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, path names, dummy URLs, user input, and Twitter handles are shown as follows: "Shebang is a line on which `#!` is prefixed to the interpreter path."

A block of code is set as follows:

```
$> env 
PWD=/home/clif/ShellCookBook 
HOME=/home/clif 
SHELL=/bin/bash 
# ... And many more lines

```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```
$> env 
PWD=/home/clif/ShellCookBook 
HOME=/home/clif 
SHELL=/bin/bash 
# ... And many more lines

```

Any command-line input or output is written as follows:

```
$ chmod a+x sample.sh

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, for example, in menus or dialog boxes, appear in the text like this: "Select System info from the Administration panel."

Warnings or important notes appear in a box like this.

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or disliked. Reader feedback is important for us as it helps us develop titles that you will really get the most out of.

To send us general feedback, simply e-mail `feedback@packtpub.com`, and mention the book's title in the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide at [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

# Downloading the example code

You can download the example code files for this book from your account at [h t t p ://w w w . p a c k t p u b . c o m](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [h t t p ://w w w . p a c k t p u b . c o m /s u p p o r t](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

You can download the code files by following these steps:

1.  Log in or register to our website using your e-mail address and password.
2.  Hover the mouse pointer on the SUPPORT tab at the top.
3.  Click on Code Downloads & Errata.
4.  Enter the name of the book in the Search box.
5.  Select the book for which you're looking to download the code files.
6.  Choose from the drop-down menu where you purchased this book from.
7.  Click on Code Download.

You can also download the code files by clicking on the Code Files button on the book's webpage at the Packt Publishing website. This page can be accessed by entering the book's name in the Search box. Please note that you need to be logged in to your Packt account.

Once the file is downloaded, please make sure that you unzip or extract the folder using the latest version of:

*   WinRAR / 7-Zip for Windows
*   Zipeg / iZip / UnRarX for Mac
*   7-Zip / PeaZip for Linux

The code bundle for the book is also hosted on GitHub at

[https://github.com/PacktPublishing/Linux-Shell-Scripting-Cookbook-Third-Edition](https://github.com/PacktPublishing/Linux-Shell-Scripting-Cookbook-Third-Edition).

We also have other code bundles from our rich catalog of books and videos available at

**[h t t p s ://g i t h u b . c o m /P a c k t P u b l i s h i n g /](https://github.com/PacktPublishing/)**. Check them out!

# Downloading the color images of this book

We also provide you with a PDF file that has color images of the screenshots/diagrams used in this book. The color images will help you better understand the changes in the output. You can download this file from the following link:

[https://www.packtpub.com/sites/default/files/downloads/LinuxShellScriptingCookbookThirdEdition_ColorImages.pdf](https://www.packtpub.com/sites/default/files/downloads/LinuxShellScriptingCookbookThirdEdition_ColorImages.pdf)

# Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you could report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the Errata Submission Form link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website or added to any list of existing errata under the Errata section of that title.

To view the previously submitted errata, go to [h t t p s ://w w w . p a c k t p u b . c o m /b o o k s /c o n t e n t /s u p p o r t](https://www.packtpub.com/books/content/support) and enter the name of the book in the search field. The required information will appear under the Errata section.

# Piracy

Piracy of copyrighted material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works in any form on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `copyright@packtpub.com` with a link to the suspected pirated material.

We appreciate your help in protecting our authors and our ability to bring you valuable content.

# Questions

If you have a problem with any aspect of this book, you can contact us at `questions@packtpub.com`, and we will do our best to address the problem.