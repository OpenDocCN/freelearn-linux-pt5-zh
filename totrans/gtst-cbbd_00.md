# Preface

Over the last few years, ARM chips have become trendy and ubiquitous, ranging from the phone and tablet market to power-efficient server farms. The low cost associated with the chips in conjunction with their powerful features makes them an apt choice for hobbyists and enthusiasts. In addition to offering a ton of connectivity, these chips have been used by several manufacturers on their development boards. The Cubieboard is a type of board with built-in networking and various input and output ports, making it an awesome utility for myriad purposes, such as media centers, robotic projects, home automation, web servers, and home security systems, to mention a few. The Cubieboard is a microcontroller, which provides a whole new set of capabilities with the extensibility of desktop machines but without the bulk or noise.

Low cost, highly expandable, and high performing with a massive, diverse range of uses and applications, the Cubieboard will revolutionize the way we think about computing and programming. With its power-packed attributes and versatility, you can create fun things. There is absolutely no fixed way to develop complex projects; however, this book will give you enough basics of the Cubieboard in a few different realms so that you can dig deeper on your own.

# What this book covers

[Chapter 1](ch01.html "Chapter 1. Choosing the Right Board"), *Choosing the Right Board*, starts with an overview of various development boards and compares a few popular ones to help you choose a board tailored to your requirements. You will also take a look at the additional hardware and a few extra peripherals that will help you understand the stuff you require for your projects.

[Chapter 2](ch02.html "Chapter 2. Getting Started with the Hardware"), *Getting Started with the Hardware*, helps you with the initial settings before you can try out things with it. After unwrapping the Cubieboard, you will learn the procedure to connect a serial port to the development board and move on to booting up the preinstalled software.

[Chapter 3](ch03.html "Chapter 3. Installing an Operating System"), *Installing an Operating System*, explains the procedure of installing an operating system in addition to installing a fully-functional graphical desktop environment onto a microSD card. It also points out the difference between an OS image and a clean installation, thereafter moving on to installing Fedora in addition to writing the OS image to a microSD card.

[Chapter 4](ch04.html "Chapter 4. Manually Installing an Alternative Operating System"), *Manually Installing an Alternative Operating System*, helps you with the process of installing a customized OS on an alternative medium (SATA SSD) in addition to making the destination medium bootable using the command line.

[Chapter 5](ch05.html "Chapter 5. Setting Up a Home Server"), *Setting Up a Home Server*, explains how the Cubieboard can be used as a home server efficiently in addition to setting up different services to be used in a home environment. You can learn about the procedure of setting up a web server, file server, torrent server, and then summing it up with setting up a personal cloud.

[Chapter 6](ch06.html "Chapter 6. Updating the Bootloader and Kernel"), *Updating the Bootloader and Kernel*, helps you to understand the difference between the various bootloader and kernel types while also assisting you with the process of obtaining and installing a new bootloader or kernel onto an SD card, which will be used as a boot device. Kernels often get updated to newer versions with security fixes or support for new hardware, thereby making it mandatory to know about them when working with many ARM boards, such as the Cubieboard.

[Chapter 7](ch07.html "Chapter 7. Compiling the Bootloader and Kernel Using a BSP"), *Compiling the Bootloader and Kernel Using a BSP*, deals with the board support package (BSP), thereby helping you compile the bootloader and kernel from source when some changes are to be made to the source code of the bootloader and kernel. You will learn to use BSP in conjunction with Git and create an easy-to-use, device-specific hardware pack.

[Chapter 8](ch08.html "Chapter 8. Blinking Lights and Sensing the World"), *Blinking Lights and Sensing the World*, starts with explaining basic electronic concepts and moves on to toggling GPIO pins and then make LEDs blink, thereby encouraging you to try out new things as you make your foray into the world of possibilities with the Cubieboard.

[Appendix A](apa.html "Appendix A. Getting Help and Finding Other Helpful Online Resources"), *Getting Help and Finding Other Helpful Online Resources*, educates you on the online resources at your disposal due to the vibrant communities and also how to obtain these resources and get help from the community in general.

[Appendix B](apb.html "Appendix B. Basic Linux Commands Cheatsheet"), *Basic Linux Commands Cheatsheet*, is a collection of various Linux commands that form a major part of your workload while using the Cubieboard, thereby helping you get to grips with the technology.

[Appendix C](apc.html "Appendix C. The FEX Configuration File"), *The FEX Configuration File*, helps you understand the FEX files that are imperative due to the fact that they are used to configure the drivers.

[Appendix D](apd.html "Appendix D. Troubleshooting the Common Pitfalls"), *Troubleshooting the Common Pitfalls*, is a small guide that will be quite handy when faced with errors and hurdles, such as boot failures, stability issues, and errors that pop up while executing commands.

# What is needed for this book

Nearly everything covered in this book can be done be on the board. To communicate with the board, a working PC is required with a working USB port to connect a USB to serial 3.3 volt UART adapter. Depending on the operating system used, a terminal emulator such as PuTTY is required. There are two options to compile the sources used in this book, either natively on the development board or via a so-called cross compiler on a regular PC. If a regular PC is used, Linux is required, but this can be run from within a virtual machine. This book was written using only freely available open source software.

# Who this book is for

This book is intended for anyone who wants to start working with Allwinner A10, A13, or A20 ARM-based hardware. This can range from hobbyists and developers at home working on a cool project to professionals working on a new ARM-based product with little ARM or little Linux knowledge. No previous Linux knowledge is required but having it certainly makes things a lot easier. Android is not really addressed in this book, and while Android is a common operating system preinstalled on many of these development boards, this book will not cover Android or Android apps.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "The `l` command can be used at any time to get an overview of the available types."

Any command-line input or output is written as follows:

```
packt@PacktPublishing:~$ tar xJvf u-boot-sunxi-cubieboard.tar.xz
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/u-boot-sunxi-with-spl.bin
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/u-boot.bin
u-boot-sunxi-cubieboard-20140307T104232-b5bd4c9/sunxi-spl.bin

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, for example, in menus or dialog boxes, appear in the text like this: "Make sure to check **Serial**."

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

You can download the example code files from your account at [http://www.packtpub.com](http://www.packtpub.com) for all the Packt Publishing books you have purchased. If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

## Downloading the color images of this book

We also provide you a PDF file that has color images of the screenshots/diagrams used in this book. The color images will help you better understand the changes in the output. You can download this file from: [https://www.packtpub.com/sites/default/files/downloads/1572OS_ColoredImages.pdf](https://www.packtpub.com/sites/default/files/downloads/1572OS_ColoredImages.pdf)

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **Errata** **Submission** **Form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded on our website, or added to any list of existing errata, under the Errata section of that title. Any existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

To view the previously submitted errata, go to [https://www.packtpub.com/books/content/support](https://www.packtpub.com/books/content/support) and enter the name of the book in the search field. The required information will appear under the **Errata** section.

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.