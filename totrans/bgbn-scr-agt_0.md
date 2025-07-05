# Preface

BeagleBone Black inspires embedded hackers in different ways. Some see it as a controller board for a small robot, while others see it as a low-power, network server. In this book, we'll treat BeagleBone Black as a defender of personal security and privacy. Each chapter will discuss a technology and then provide a project to reinforce the concept.

Let's get started!

# What this book covers

[Chapter 1](part0015_split_000.html#page "Chapter 1. Creating Your BeagleBone Black Development Environment"), *Creating Your BeagleBone Black Development Environment*, starts the journey by showing you how to set up Emacs to maximize your embedded hacking.

[Chapter 2](part0019_split_000.html#page "Chapter 2. Circumventing Censorship with a Tor Bridge"), *Circumventing Censorship with a Tor Bridge*, walks through how to transform the BeagleBone Black into a Tor server with a front panel interface.

[Chapter 3](part0031_split_000.html#page "Chapter 3. Adding Hardware Security with the CryptoCape"), *Adding Hardware Security with the CryptoCape*, investigates biometric authentication and specialized security chips.

[Chapter 4](part0043_split_000.html#page "Chapter 4. Protecting GPG Keys with a Trusted Platform Module"), *Protecting GPG Keys with a Trusted Platform Module*, shows you how to use the BeagleBone Black to safeguard e-mail encryption keys.

[Chapter 5](part0053_split_000.html#page "Chapter 5. Chatting Off-the-Record"), *Chatting Off-the-Record*, details how to run an IRC gateway on the BeagleBone Black to encrypt instant messaging.

[Appendix](part0059_split_000.html#page "Appendix A. Selected Bibliography"), *Selected Bibliography*, lists the referenced works cited throughout the book.

# What you need for this book

This book contains several independent projects that use various software packages and hardware components. All the software is open source, and installation instructions are given throughout the chapter. A list of all the necessary hardware is maintained as a SparkFun Electronics wish list at [https://www.sparkfun.com/wish_lists/93119](https://www.sparkfun.com/wish_lists/93119). Each chapter will list the required components for that project, so it's best to first read the chapter and then collect the necessary hardware.

# Who this book is for

If you have an interest in individual security and privacy on the Internet, then you hopefully should enjoy this book. If you have a security background but are new to embedded computing, then you should find the projects challenging but rewarding. Conversely, if you are versed in electronics and are working with GNU/Linux distributions but haven't studied the security technologies mentioned in this book, then you should appreciate the discussions of the technologies in each chapter.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "The LCD, controlled by the `FrontPanelDisplay` class, writes to the serial port, `/dev/ttyO4`, on BBB's UART 4 at 9600 baud."

A block of code is set as follows:

```
self.clear_screen()
up_str = '{0:<16}'.format('Up:   ' + self.block_char * up)
dn_str = '{0:<16}'.format('Down: ' + self.block_char * down)

self.port.write(up_str)
self.port.write(dn_str)
```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```
import Adafruit_BBIO.UART as UART
import serial
class FrontPanelDisplay(object):

  def __init__(self):
    self.uart = 'UART4'
    UART.setup(self.uart)
 self.port = serial.Serial(port="/dev/ttyO4", baudrate=9600)
    self.port.open()
```

Any command-line input or output is written as follows:

```
Mar 25 21:37:43.000 [notice] Tor has successfully opened a circuit. Looks like client functionality is working.
Mar 25 21:37:43.000 [notice] Bootstrapped 100%: Done.

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "To connect to your bridge, launch the Tor browser and click on **Open Settings** as it starts up."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or may have disliked. Reader feedback is important for us to develop titles that you really get the most out of.

To send us general feedback, simply send an e-mail to `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, and mention the book title via the subject of your message. If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide on [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the example code

You can download the example code files for all Packt books you have purchased from your account at [http://www.packtpub.com](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata), selecting your book, clicking on the **errata submission form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded on our website, or added to any list of existing errata, under the Errata section of that title. Any existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.