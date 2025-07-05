# Chapter 1. Creating Your BeagleBone Black Development Environment

This book is for secret agents. James Bond is the secret agent who most likely comes to mind, but in today's highly connected world, the real secret agents are to be found in Q branch. Spycraft techniques such as camouflage, listening devices, and palm-print weapons are useful for field agents, but online we need other tools. These tools, like their field agent counterparts, enable you to hide in a crowd, protect secret communication, and prove the identity of other agents. The software and hardware projects in this book use tools relied upon by whistleblowers, journalists protecting their sources, and everyday citizens attempting to access unfiltered information in a strongly censored country.

**BeagleBone Black** (**BBB**) is a complete computer that fits inside an Altoid's tin. Its small form factor, low power consumption, and capable performance empower the device to help you secure your privacy and protect your communication online. Before we can build upon these tools, we first need to know how to interact with the BBB. This chapter will introduce you to BBB and suggest a development environment in which you can build the later projects.

In this chapter, you will:

*   Learn about BBB and the open source principles behind the project
*   Install and use the Emacs editor
*   Configure Emacs as your embedded development environment
*   Tailor your SSH configuration for usability
*   Investigate resources for background information on cryptography, electronics, and Linux

# Introducing the BBB

From the BeagleBoard website ([BeagleBoard.org](http://BeagleBoard.org) ), which is the nonprofit foundation behind BeagleBoard-xM, BeagleBone, and BBB, the BBB is *a low-cost, community-supported development platform for developers and hobbyists*. The Rev C, which is the latest revision, has impressive specifications including the TI Sitara AM3358 1GHz ARM Cortex-A8 processor, 512 MB of DDR3 RAM, and 4 GB **Embedded Multi-Media Card** (**eMMC**) for on-board flash. When you take a look at the board, you'll see two 46-pin expansions headers. If you compare it to other hobbyist boards around the same price point, you'll come to the conclusion that the other boards do not contain nearly as much expansion capability. The BBB supports many more **Input/Output** (**IO**) options, including three I2C buses, multiple serial ports, 65 **General Purpose IO** (**GPIO**), multiple **Pulse Width Modulators** (**PWM**), and seven analog inputs with built-in **Analog-to-Digital Converters** (**ADCs**). If you don't know what all of these are, that's not a problem, as we'll explain the systems that we use throughout the projects.

The quality of the documentation from [BeagleBoard.org](http://BeagleBoard.org) is outstanding. You should read the BBB **System Reference Manual** (**SRM**), the official manual for BBB, which is located at [https://github.com/CircuitCo/BeagleBone-Black/blob/master/BBB_SRM.pdf?raw=true](https://github.com/CircuitCo/BeagleBone-Black/blob/master/BBB_SRM.pdf?raw=true). This is a complete manual that covers connecting the BBB, power options, and boot sequences. Many of the questions asked on the BeagleBoard mailing list and IRC channel can be quickly answered in this manual. The author of this book assumes that you've at least skimmed sections 3, 4, and 5 of the SRM, which means you are aware of the basic capabilities of BBB and are familiar with the physical connectors. There is simply no better reference for the BBB than this document.

## Appreciating BBB's commitment to open source hardware

BBB has another very important quality: it is **Open Source Hardware** (**OSHW**). OSHW is a relatively new concept, the exact definition of which may confuse people. However, there is a group, the **OSHW Association** (**OSHWA**), whose mission is to educate and promote OSHW. Their definition is maintained on their website: [http://www.oshwa.org/definition/](http://www.oshwa.org/definition/). As with most organizations, a consensus for a definition can be difficult to obtain. The definition of OSHW is well over a page. The complication with defining OSHW from open source software is that hardware is a *physical thing*. There are design files to manufacture hardware, but there are physical components also. To make a software analogy, the compiler for the hardware is the manufacturer. Therefore, the definition of OSHW is carefully constructed and it generally applies to the design files of the hardware.

### Note

Alicia Gibb, the Executive Director of OSHWA, gave a TEDx talk called *The Death of Patents and What Comes After* ([https://www.youtube.com/watch?v=z__Sbw1Ax4o](https://www.youtube.com/watch?v=z__Sbw1Ax4o)). The talk illustrates how hardware design straddles both copyrights and patent law. Alicia also provides some interesting insights on the incentives behind patents and OSHW.

BBB is OSHW, in that it releases documentation, schematics, **Computer-Aided Design** (**CAD**) files, **Bill Of Materials** (**BOM**), and production files (Gerbers), all under a Creative Commons license. This means that you can not only study the complete design but you are also free to make your own derivative BBB.

## Unboxing the BBB and providing power

Unlike Raspberry Pi, BBB is ready out of the box. A recent BBB will come with the Debian distribution of GNU/Linux, henceforth referred to as Debian, installed on the eMMC. eMMC is the on-board flash memory for the BBB. As soon as power is applied to the board, BBB will start to boot from the eMMC. As shown in section three of the SRM, you can connect the BBB directly to your PC with the supplied USB cable. The BBB can also be powered from a 5V barrel jack, where you'll want a wall adapter that can supply up to 1A. If you plan on connecting a mini **liquid-crystal display** (**LCD**) to the BBB, you may want to use a 2A adapter. You can typically find the output voltage and amperage specifications written on the adapter. You can tell whether the board is powered and running if the blue user **light-emitting diodes** (**LEDs**) are flashing. Specifically, the LED USER0 will flash in a heartbeat pattern.

### Note

Each user LED has a default meaning that corresponds to a specific BBB activity. To further motivate you to read the SRM, the meanings are defined in section 3.4.3 under step 6, *Booting the Board*.

### Tip

Be very careful when choosing your power supply. The BBB needs 5VDC +/-.25V. Connecting a higher voltage power supply will damage the board.

The BBB can support multiple peripherals such as a keyboard, mouse, and monitor. However, in this book, we'll be using the BBB in a *headless* configuration, which means *without* the monitor. Section 3 of the SRM details the various connection scenarios.

# Creating an embedded development environment with Emacs

With the BBB powered, we need to find a way to interact with it. We need a set of tools that will connect to the BBB, send shell commands, and transfer files. This set of tools are your **development environment**. Development environments are a personal choice and there is no shortage of choices. Finding a suitable environment is well worth the time since it will be the main tool, or tools, with which you interface. Your environment needs to be technically capable of performing the tasks you need, but it also needs to be configurable and extensible. The environment that is described here is fully contained within the Emacs editor.

### Note

If you would rather use a specific **Integrated Development Environment** (**IDE**) such as Eclipse, you can take a look at some useful tutorials at [http://derekmolloy.ie/beaglebone/setting-up-eclipse-on-the-beaglebone-for-c-development/](http://derekmolloy.ie/beaglebone/setting-up-eclipse-on-the-beaglebone-for-c-development/) and [http://janaxelson.com/eclipse1.htm](http://janaxelson.com/eclipse1.htm).

## Understanding the complications of embedded development

The history of Emacs is about as long as the history of modern computing. As an editor, Emacs is often overlooked because of its reputation of being outdated and difficult to learn. Emacs can be overwhelming since the interface is different from most modern interfaces. Also, the keybindings were created prior to the invention of modern **Graphical User Interfaces** (**GUIs**), so the keybindings don't correspond to the shortcuts that you are typically accustomed to. However, there is active development on various Emacs *starter packs*, which provide a smoother Emacs experience. If you keep in mind that Emacs predates your operating system, you may find it easier to accept the Emacs way.

### Note

Many early notable programmers have worked on Emacs, including Guy Steele, Richard Stallman, James Gosling, and Jamie Zawinski. The design of Emacs was presented by Richard Stallman in 1981 to the ACM Conference on Text Processing; the full text is available at [https://www.gnu.org/software/emacs/emacs-paper.html](https://www.gnu.org/software/emacs/emacs-paper.html).

Embedded development is slightly more complicated than web or desktop development because there are typically two machines involved: the host (your main computer) and the target (your embedded platform). Embedded systems range in capabilities and some run without an operating system, which certainly can't support running a compiler. In this case, the user cross-compiles the code on the host for the target. This can be performed for the BBB, but compiling small programs natively on the BBB does not take too long.

A common recommendation for development on the BBB is to connect to the device over **Secure Shell** (**SSH**) and to use command-line tools. This can be an effective technique, but this limits you to the command line and your terminal emulator. These tools are very powerful and in this context, limited does not mean limited in functionality but limited in the interface.

The one helpful feature of Emacs for embedded development is **Transparent Remote Access Multiple Protocol** or the **TRAMP** mode. In short, the TRAMP mode lets you run Emacs on your host machine, which is most likely more powerful than the BBB, but access files and run commands on the BBB as if they were local. This is transparent in TRAMP; working in this environment feels like you are working on a host, not on an embedded platform. The following screenshot shows the TRAMP feature of Emacs:

![Understanding the complications of embedded development](img/00002.jpeg)

This screenshot was captured with Emacs running on Mac OS X, but connected to a BBB with the hostname `slartibartfast`. Everything on this screen is running on the BBB. In the top-left corner, the frame shows a live debugging session with the **GNU debugger** (**GDB**) on a simple C program. In the bottom-left corner, the frame shows the compilation result obtained from running `gcc` on the BBB. In the top-right corner, the frame shows the interactive `gdb` session. Lastly, in the bottom-right corner, the frames show the current directory and a shell, respectively. In the following sections, we'll provide details on how to install Emacs and set up this development environment.

### Tip

To name your BBB, you'll have to change two files. The first file is `/etc/hostname` and then you'll have to make the same change in `/etc/hosts`. If you struggle to choose a proper name, you will find some comfort in the following XKCD comic:

[https://xkcd.com/910/](https://xkcd.com/910/)

## Installing Emacs 24

We'll need Emacs 24.x to take advantage of the **prelude** starter pack by Bozhidar Batsov. Prelude is under active maintenance by Batsov and dedicated prelude users. The code is maintained on GitHub ([https://github.com/bbatsov/prelude](https://github.com/bbatsov/prelude)), where there is a detailed README. This software is optional; however, it adds many convenient features and reasonable default configuration settings. Installation procedures for Emacs vary greatly by operating system, but you can consult the following table for the software locations:

| OS | Link |
| --- | --- |
| Windows | [http://ftp.gnu.org/gnu/emacs/windows/](http://ftp.gnu.org/gnu/emacs/windows/) |
| Mac OS X | [http://emacsformacosx.com/builds](http://emacsformacosx.com/builds) |
| Source | [http://ftp.gnu.org/gnu/emacs/emacs-24.3.tar.gz](http://ftp.gnu.org/gnu/emacs/emacs-24.3.tar.gz) |

If you are a Windows user and are having trouble installing Emacs, consult the guide at [https://www.gnu.org/software/emacs/manual/html_mono/efaq-w32.html](https://www.gnu.org/software/emacs/manual/html_mono/efaq-w32.html). Mac users should find the binaries listed in the previous section sufficient. If you are using a *nix machine, you can either use your package manager to install Emacs or build from the source. If you use the package manager, be sure to install Emacs' major version 24\. In a Debian-based system, the command is `apt-get install emacs24`.

## Installing the prelude

The Emacs prelude software can be installed by typing the following command:

```
curl -L http://git.io/epre | sh

```

### Tip

**Downloading the example code**

You can download the example code files for all Packt books you have purchased from your account at [http://www.packtpub.com](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

If you are wary of shortened URLs, take a look at the prelude README for alternate installation instructions, which is available at [https://github.com/bbatsov/prelude](https://github.com/bbatsov/prelude). The installer will complain about any missing packages that you need to install. Once complete, you should run Emacs. If you have installed a binary version for your OS, double-clicking on the icon should suffice. On a command line, just type `emacs` to launch the program. On the initial start, Emacs will attempt to go to the Emacs package managers and download additional software. It will then byte-compile the source files, so it takes a minute or two. Once this is complete, close Emacs and open it again; you should see something similar to what is shown in the following screen:

![Installing the prelude](img/00003.jpeg)

### Note

Yes, Emacs has its own package manager and Emacs-specific repositories that provide the Emacs Lisp Package. See the following blog post by Batsov for a deeper look at Emacs packaging:

[http://batsov.com/articles/2012/02/19/package-management-in-emacs-the-good-the-bad-and-the-ugly/](http://batsov.com/articles/2012/02/19/package-management-in-emacs-the-good-the-bad-and-the-ugly/)

## Learning how to learn about Emacs

Emacs has too many features. Here's a tongue-in-cheek joke: *Emacs is an operating system that is in need of a good editor*. This section will illustrate the basic skills that one needs to use Emacs and more importantly, to learn how to use Emacs. One of the best features of Emacs is the immense built-in help. If you have never run Emacs before, press *Ctrl-h t*, abbreviated *C-h t*. This means that while holding the *Ctrl* key, press *h*, then release it, and then press *t*. Also, note that the Emacs commands are case sensitive. *C-H T* means to press *Ctrl + Shift + H*, then hit *Shift + T*, whereas *C-h t* is performed without the shift modifier. You'll be presented with the interactive Emacs tutorial. This tutorial will teach you the fundamentals of Emacs. In the tutorial, you'll see references to a meta key, which you will be hard pressed to find on your keyboard. Command sequences, such as *M-x*, mean to hold down the meta key while pressing *x*. Depending on your system, the meta key may be *Alt* or *Option*. If neither of these work, you can press the *Esc* key and release it to provide meta.

The tutorial is very good and for the rest of this chapter, it is assumed that you have completed this tutorial or have an equivalent level of knowledge to understand the more advanced features.

### Note

Some other helpful resources to learn Emacs are Sacha Chua's *How to Learn Emacs* diagram ([http://sachachua.com/blog/wp-content/uploads/2013/05/How-to-Learn-Emacs-v2-Large.png](http://sachachua.com/blog/wp-content/uploads/2013/05/How-to-Learn-Emacs-v2-Large.png)) and the official *Emacs 24 Reference Card* ([https://www.gnu.org/software/emacs/refcards/pdf/refcard.pdf](https://www.gnu.org/software/emacs/refcards/pdf/refcard.pdf)).

### Tip

If you are a vi user, there is a mode that emulates most vi keybindings, called the viper mode ([http://www.emacswiki.org/emacs/ViperMode](http://www.emacswiki.org/emacs/ViperMode)).

## Streamlining the SSH connections

Even if you don't use Emacs, you'll find yourself frequently connecting to the BBB with SSH. If you run the BBB from the USB cable attached to your PC, then the default Internet Protocol (IP) address of the BBB will be `192.168.7.2`. However, if you want to download software directly to the BBB, as you will need to do in order to complete these projects, you'll have to forward your Internet connection from the PC to the BBB. Also, when powered from the USB directly, there is typically a 500mA current limit. When the BBB is powering expansion boards or other peripherals, it'll need to be powered from a 5V DC plug that can supply up to 1A. As a result of these two issues, you may find it much more convenient to run the BBB from a 5V power supply and connect the Ethernet cable into your home network directly.

When you do this, the IP address will most likely be assigned by your router and will only be known to your router. This will leaving you staring at the BBB pontificating over its IP address.

### Discovering the IP address of your networked BBB

There are a few ways to discover the IP address of your device. The first way is to log in to your router and look up the IP address. This is a quick method but as there are numerous routers, you'll have to consult your router's documentation on how to accomplish this.

You can also use the `nmap` utility to conduct a quick **ping scan** on your network. Assuming that you are familiar with the IP address of your own network, you should be able to see a new device. If you are running an apt-based distribution, you can install `nmap` with `sudo apt-get install nmap`. If you are on a Mac, you can use the homebrew package manager ([http://brew.sh/](http://brew.sh/)) and install it with `brew install nmap`. Windows users should download the binary from [http://nmap.org/download.html#windows](http://nmap.org/download.html#windows). Once it is installed, you'll want to run a command similar to the following:

```
nmap -sn 192.168.1.0/24

```

Replace `192.168.1.0` with the appropriate address for your network. `/24` is the **Classless Inter-Domain Routing** (**CIDR**) notation for the subnet mask. In this case, `/24` means use a subnet mask of 24 bits of 1's, or `255.255.255.0`. The `-sn` option directs `nmap` to conduct a ping scan.

If you are still unable to determine the IP address, you can buy a USB-to-serial adapter cable. BBB has a serial debug header located inboard of the P9 header. You can attach the USB-to-serial adapter to BBB's serial debug header, as shown in the following image:

![Discovering the IP address of your networked BBB](img/00004.jpeg)

In this figure, pin 1 (which is the black wire) is the ground, pin 4 is receive, and pin 5 is transmit. Once physically attached, you'll need a terminal emulator to connect over the serial connection. GNU Screen is a widely available, free, and robust software that will connect with the following command; just replace `/dev/tty.XXXX` with the file of your connection:

```
screen /dev/tty.PL2303-004014FA 115200

```

Lastly, you can set a static IP for your BBB. Your router may support a static lease feature, which will assign the IP of your choosing to the device when it performs a **Dynamic Host Configuration Protocol** (**DHCP**) request. The lease on this request can be indefinite and, therefore, effectively provides a *static* IP. You can also set the IP statically on your device, but then you will have to manage conflicts manually. Therefore, the preferred method is to let the router handle this situation.

### Tip

To find a serial debugging cable, see the eLinux page at [http://elinux.org/Beagleboard:BeagleBone_Black_Serial](http://elinux.org/Beagleboard:BeagleBone_Black_Serial).

To take full advantage of the SSH configuration options in the following section, you'll want to have a static IP set. Otherwise, you will have to repeat the previous steps to determine the IP address of the device when your router power cycles or if the BBB is disconnected for a few days.

### Tip

If you prefer to use a WiFi connection instead of Ethernet, be aware that there are issues with some dongles. Refer to this eLinux page for supported adapters: [http://elinux.org/Beagleboard:BeagleBoneBlack#WIFI_Adapters](http://elinux.org/Beagleboard:BeagleBoneBlack#WIFI_Adapters).

### Editing the SSH configuration file

In order to connect to BBB, you need to type something similar to the following, but use the BBB's IP address from the previous section:

```
ssh debian@192.168.1.10

```

### Tip

If you are a Windows user, you should consider using more Unixy tools on your host since the BBB is running Debian anyway. You can install Cygwin ([https://www.cygwin.com/](https://www.cygwin.com/)), which will give you a nice shell with the associated command-line tools. Or, consider downloading VirtualBox ([https://www.virtualbox.org/](https://www.virtualbox.org/)) and installing a GNU/Linux distribution in a virtual machine.

The default username is `debian` and the default password is `temppwd`. If you only have one device, typing this command may be manageable. Once you start collecting BBBs, Raspberry Pis, pcDuinos, Cubieboards, Arduinos with a WiFly Shield, and so on, you will have to remember each IP, username, and password. This can become annoying. Fortunately, there are a few tricks that you can use to improve this situation.

First, you'll need a SSH private key. If you don't already have a key, you can generate one on your host with the following command:

```
ssh-keygen -t rsa -b 4096

```

Now edit the `~/.ssh/config` file. If you are doing this in Emacs, you can press *C-x C-f* and then type `~/.ssh/config`. Emacs keybindings are all bound to Emacs Lisp commands (technically, there are a few basic commands implemented in C). The keybinding *C-x C-f* is bound to the `find-file` function. An alternate, albeit longer, way to open a file is to press *M-x*, type `find-file`, then open the file of your choice.

Add the following section to your `~/.ssh/config` file, replacing HostName with the IP address of your BBB:

```
Host bbb
 HostName 192.168.1.10
 User debian

```

```
http://www.cyberciti.biz/faq/howto-regenerate-openssh-host-keys/:
```

```
rm /etc/ssh/ssh_host_*
dpkg-reconfigure openssh-server

```

This will delete your existing SSH host key and regenerate it from scratch. Since the host key has changed, we need to update your SSH client on you main computer. If you log in to the BBB again, you will probably receive a warning that the fingerprint doesn't match because you changed it. You can remove the old entry from your client by typing the following (you may have to use the IP address instead of `bbb`):

```
ssh-keygen -R bbb

```

Alternatively, you can edit `~/.ssh/known_hosts` directly and remove the entry.

With the new SSH key on the BBB and the old entry in `known_hosts` removed, log back in to the BBB just to make sure everything is working. Log back out and return to the terminal on your host. After executing the following command, you will no longer be prompted for a password:

```
cat .ssh/id_rsa.pub | ssh bbb 'cat >> .ssh/authorized_keys'

```

This will copy your public SSH key to the BBB, which will be recognized in the SSH handshake as an authorized key, and therefore, it will not require you to enter the `debian` user's password.

Most likely, you will need to perform this operation again and you may not want to refer to this book to find this command. You can add this function to your `.bashrc` file or an equivalent for further ease of use:

```
ssh_upload(){
  if [[ $# -ne 1 ]]; then
    echo "usage: host\nNOTE: host should be set in ~/.ssh/config"
    else
    cat ~/.ssh/id_rsa.pub | ssh $1 'cat >> ~/.ssh/authorized_keys'
    fi
}
```

Then, you can add your key to servers with:

```
ssh_upload bbb

```

Don't forget to reload your `.bashrc` file to gain this function with:

```
source ~/.bashrc
```

Logging out and in again will also work.

### Running an SSH agent to control access to your SSH keys

Now you don't have to enter the `debian` account password each time you log in to the BBB, but now you do have to enter the password for your SSH private key. This is equally annoying. However, we can fix this by running `ssh-agent`. Installing and configuring `ssh-agent` is OS-dependent, and instructions are easy to find online. Once `ssh-agent` is running, you'll typically have to enter your SSH private key password when you log in and if you make an SSH connection within a certain time period, you'll instantly connect to the remote server without any additional password entry.

## Connecting to BBB with TRAMP

Now that we've configured SSH, we will now have an easier time connecting to BBB with TRAMP and Emacs. Emacs has a built-in directory editor called `dired`, which can be invoked by pressing *C-x d*. We want to open `dired` on the BBB to view the remote files. Press *C-x d* and then enter the following for the directory: `/ssh:bbb:/home/debian`. A screen similar to the following should immediately open, which will show you the contents of the home directory:

![Connecting to BBB with TRAMP](img/00005.jpeg)

All the basic Emacs navigation commands work in `dired`, so you can navigate the directory and press enter to open a file or another directory. The *^* key will navigate you to the parent directory. You can also click on the directories or files to open them.

### Tip

If the `tramp-default-method` variable is set to `ssh`, then you can shorten your URLs to `/bbb:/home/debian`. You can view the state of the variable by evaluating it. To evaluate Emacs Lisp, press *M-:* and then type `tramp-default-method` and the mini buffer should reply with `ssh`.

For more information on `dired` or anything about Emacs, you can view the built-in documentation by pressing *C-h r*. If you scroll or search (by pressing *C-s*), you'll find the `dired` manual on that page. The manual uses the info viewer, and you can read about this by pressing *C-h i* to get the main information page and then pressing *h*.

One of the features of Emacs is its extensibility. Presumably, you will be connecting to the BBB often, pressing *C-x d*, and typing `/ssh:bbb:/home/debian` can be tiresome. Let's define a new key sequence to make this easier. Switch to the scratch buffer in Emacs. This can be done by pressing *C-x b*, then typing `*scratch`, and then hitting *Tab* + *Enter*. Paste the following code:

```
(global-set-key
 [(control x) (control y)]
 (lambda ()
   "Open a dired buffer on the BBB"
   (interactive)
   (dired "/ssh:bbb:/home/debian")))
```

```
 press *C-x C-y* and you should see the home directory of BBB. Congratulations! You've just customized Emacs! If you restart Emacs, however, you'll lose this convenient function, so you need to save it. If you are using the Emacs prelude, customizations should go in ~/.emacs.d/personal/foo.el. You can save this buffer with *C-x C-s* and specify the filename.
```

If you enjoy using Emacs, you will probably enjoy changing Emacs, which can be done with Emacs Lisp. Of course, Emacs contains a great manual to learn Emacs Lisp, *An Introduction to Programming in Emacs Lisp*. It can be accessed by pressing *C-h i d* and then you can either scroll down to the manual or press m, type `Emacs Lisp`, and then press *Tab* for tabcompletion to *Emacs Lisp Intro*.

### Running commands from Emacs

If you are currently looking at the `dired` buffer in Emacs, then Emacs is aware that you are on a remote machine. Certain commands are TRAMP-aware and will execute remotely. For example, *M-!* will execute a shell command. When used over TRAMP, it will execute the shell command remotely. Try using *M-!* with `touch` `test.txt`. Now press *g* to refresh `dired`. You should now see `test.txt`. If you edit this file by navigating to it in `dired` and then pressing *Enter*, you will now be editing the file remotely. Technically, you are editing a local copy of this buffer but when you save it with *C-x C-s*, it will send the changes over to the BBB.

There is a benefit to this system. If the SSH connection drops, for whatever reason, the buffer is still in RAM on your local machine. Once the connection is restored, Emacs will save the buffer. Emacs acts as a session manager for your remote connections and will transparently reconnect when needed.

If you open a C file in Emacs, Emacs will apply C-aware syntax highlighting, but it can also compile and debug over TRAMP. To compile, invoke *M-x*, type `compile`, and supply the compile command. If there is a makefile, then `make` should suffice; otherwise, provide the full `gcc` compilation string. The benefit of using the compile function is that Emacs will be aware of compilation errors and you can jump to each one with *C-x*`. Lastly, if you install `gdb` on BBB, you debug with `gdb` from your host computer. Assuming that you compiled your program with debug symbols, if you are looking at the source file in the current buffer, you can press *M-x*, and then type `gdb`.

You'll have to provide the `gdb` command, which is something such as `gdb nameofexec -i=mi`. Emacs will remotely launch `gdb` and open an interactive session. If you set breakpoints, Emacs will switch over to the code buffer and show you where the instruction pointer has stopped in the code.

Lastly, you can run a shell inside Emacs on the remote system as well. If you are in a TRAMP-aware buffer, pressing *M-x* and then typing `shell` will open a remote shell. You can perform all your development actions while never leaving Emacs. Emacs can also act as your mail reader, calculator, IRC client, and even play Tetris (by pressing *M-x*, and then typing `Tetris`). Emacs is a cross platform editor, so if you learn how to use Emacs once, you can use it as your development environment on any machine.

### Tip

XKCD fans of comic 378, should press *M-x* and then type `butterfly`. Be sure to respond `yes` to the question *Do you really want to unleash the powers of the butterfly?* For background on *M-x* butterfly, see [https://xkcd.com/378/](https://xkcd.com/378/)

### Using Emacs dired to copy files to and from BBB

Another routine and tedious task is to copy files from the host to the target or vice versa. Most likely, you can use the command line and a tool such as `scp` to send files back and forth. Emacs `dired` using TRAMP will make this task much simpler. In a `dired` buffer, press *C* to invoke the `dired-do-copy` function. This will prompt you for the destination. If you specify a remote address such as `/ssh:bbb:/home/debian`, then `dired` will copy the files over SSH. You can copy multiple files by marking them with *m* and then pressing *C*. The reverse, copying files from BBB to the host, is the same process. This prevents you from having to drop to a shell and use `scp`.

There are many resources online to learn Emacs. If you are an IRC user, the `#emacs` channel on `Freenode` is the watering hole of many experienced Emacs users. The Emacs wiki is also a good starting point for Emacs research, which has a Emacs newbie entry ([http://www.emacswiki.org/emacs/EmacsNewbie](http://www.emacswiki.org/emacs/EmacsNewbie)). The built-in help system is quite extensive. To receive help on the help system, press *C-h ?*.

# Finding additional background information

The projects in the book are, for the most part, self-contained. However, they do assume that you have some background knowledge in working with a GNU/Linux system, electronics, and some cryptographic concepts. The following sections list some resources in each area for those who want to seek further information.

## Finding additional cryptography resources

The projects in this book all use some sort of cryptography. If you are familiar with terms such as asymmetric cryptography, digital signatures, and message authentication codes, then you should be OK. If not, you will still be able to complete the projects but you may not appreciate the theory behind them.

As this is not a book on cryptography, a primer here will not provide enough detail for a beginner and will be woefully inadequate for someone familiar with the material. If cryptography is new to you, you can still proceed with the book. In the beginning of each chapter, there is some tailored background to explain the material, which should be enough for you to understand what the project is trying to accomplish. Once complete, hopefully, you'll be interested in cryptography and you can get more information from the following resources.

For a gentle introduction to the topic, *Cryptography: A Very Short Introduction* by *Fred Piper and Sean Murphy*, *Oxford University Press*, *2002*, is good starting point. However, as the title suggests, it lacks technical depth. *Understanding Cryptography: A Textbook for Students and Practitioners* by *Christof Paar et. al.*, *Springer, 2010*, is a more detailed introduction. For the more inquisitive readers, *Introduction to Modern Cryptography* by *Jonathan Katz and Yehuda Lindell*, *Chapman and Hall/CRC*, *2007*, is a good up-to-date reference.

If lectures better suit you, you are in luck. Khan Academy has some interesting and free mini-lectures covering ancient cryptography up to RSA ([https://www.khanacademy.org/computing/computer-science/cryptography](https://www.khanacademy.org/computing/computer-science/cryptography)). Another free resource is Coursera, which has three cryptography classes, *Cryptography I and II* taught by Standford Professor Dan Boneh, and a Cryptography class on modern cryptography taught by Jonathan Katz. The links for these classes are [https://www.coursera.org/course/crypto](https://www.coursera.org/course/crypto), [https://www.coursera.org/course/crypto2](https://www.coursera.org/course/crypto2), and [https://www.coursera.org/course/cryptography](https://www.coursera.org/course/cryptography), respectively.

## Finding additional electronics resources

Similarly, this book assumes some basic electronics knowledge. If you are looking for a book, *Practical Electronics for Inventors*, *Third Edition* by *Paul Scherz and Simon Monk*, *Tab Books*, *2013*, is a solid and approachable reference. Khan Academy has a series on basic concepts in electricity and magnetism as well ([https://www.khanacademy.org/science/physics/electricity-and-magnetism](https://www.khanacademy.org/science/physics/electricity-and-magnetism)). Dave Jones EEVBlog is an entertaining and informative video blog that covers many areas of electronics and should be of interest to hobbyists in electronics ([http://www.eevblog.com/](http://www.eevblog.com/)).

## Finding additional Debian resources

For those new to Debian, there is an comprehensive and free handbook available online called *The Debian Administrator's Handbook* by *Raphaël Hertzog and Roland Mas*, *Freexian SARL*, *2014*, at [http://debian-handbook.info/browse/stable/](http://debian-handbook.info/browse/stable/). At a minimum, you should read *Chapter 6, Maintenance and Updates: The APT Tools*, of this book, which details how to use the apt set of tools for package management. There is also a free class offered by the Linux Foundation and edX called *Introduction to Linux* ([https://www.edx.org/course/linuxfoundationx/linuxfoundationx-lfs101x-introduction-1621](https://www.edx.org/course/linuxfoundationx/linuxfoundationx-lfs101x-introduction-1621)). The class is fairly recent and it does not appear to use the Debian distribution, but it does seem like a representative course on generic Linux information. Lastly, there are several Debian IRC channels on the OFTC IRC network. Most of the channels start with `#debian` and you should be able to ask specific questions there. The BeagleBoard channel is `#beagle` on `Freenode`, and there will be people there who can help answer your questions as well.

# Summary

In this chapter, you learned how to connect to your BBB. [BeagleBoard.org](http://BeagleBoard.org) has streamlined the out-of-box experience, so there is a minimal setup on the device itself. You learned that the definitive reference manual for the BBB is the *System Reference Manual*, which should be your first stop for hardware-related questions about BBB. You learned about Emacs and how to create an embedded development environment with Emacs. Lastly, we identified several resources to find additional information on cryptography, electronics, and Linux.

In the rest of this book, you will build projects around the most popular and trusted software security packages: Tor, GNU Privacy Guide, and Off-the-Record messaging, all of which are actively used and maintained. In the next chapter, we'll use Tor and the BBB to help strengthen a censorship-resistant network.