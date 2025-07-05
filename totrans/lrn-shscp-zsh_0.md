# Preface

If I had to take a wild guess, I'd say that you are reading these lines because, like me, you spend quite some time dealing with Unix systems. Be it because your job requires you to, or you simply love to poke around an operating system's internals, the shell is arguably how you deal with most of your activities.

Historically, shells were conceived for speeding up our work, but we all know that at some point, what was supposed to be a leaner way to get things done turned into a slugfest of arcane symbols and impossibly long-to-remember lines of code.

Wouldn't it be great then, if we could squeeze just a bit more out of our system? Imagine the things you are currently doing and being able to do them in a more efficient, elegant way, even the things that you thought were some sort of magic that only Linux's wizards with centuries' worth of experience were able to perform.

What if I told you that feats such as knowing which option flags are available to a program no longer require you to scan endless screens of manpages? Imagine not having to deal with journeys along infinite horizontal lines of characters anymore. And what about relying on automatic completion instead of typing the same lines again? What if knowing which directory you are currently working on merely required you to stare at your command prompt? Now imagine that all it takes for getting started with all of this only demands you to switch to a new shell.

# What this book covers

[Chapter 1](ch01.html "Chapter 1. Getting Started"), *Getting Started*, starts from scratch by explaining how to install and set up zsh. Learn about startup files and customizing the shell prompt.

[Chapter 2](ch02.html "Chapter 2. Alias and History"), *Alias and History*, explains how aliasing works, how to define aliases in your startup files, and teaches you how to work with the shell's history log.

[Chapter 3](ch03.html "Chapter 3. Advanced Editing"), *Advanced Editing*, introduces zsh's Line Editor and working with the various shortcuts and key bindings on the command line.

[Chapter 4](ch04.html "Chapter 4. Globbing"), *Globbing*, introduces the new ways of working with the system's files and directories by applying parameter substitution and modifiers to deal with all kinds of tasks.

[Chapter 5](ch05.html "Chapter 5. Completion"), *Completion*, introduces you to one of zsh's greatest features and shows you how to start tweaking "the new" completion system by defining your own styles and functions.

[Chapter 6](ch06.html "Chapter 6. Tips and Tricks"), *Tips and Tricks*, explains miscellaneous settings and configuration options that are definitely worth trying, together with some cool community projects that should be on your radar.

# What you need for this book

Before getting started, you should be comfortable in handling a terminal emulator. Most operating systems bundle such software within their stock set of applications, but as is the case with any application, there are other offerings out there waiting to be discovered. Such alternatives are probably even better suited for the task at hand, so please make sure you get to know the ins and outs of your weapon of choice and its quirks before jumping into this book.

Also required for following this text and the provided examples is the Git source code management system. It can be easily obtained and installed by following the instructions provided at [http://git-scm.com](http://git-scm.com), and it's an indispensable tool when attempting to use some of the various software projects and sources mentioned throughout this book.

# Who this book is for

This book is great for system administrators, developers, and other computer professionals involved with Unix, who are looking to improve on their daily tasks involving the Unix shell. It's assumed that you have some familiarity with a Unix command-line interface and feel comfortable with editors such as Emacs or vi. The usage of web browsers is optionally required for reading some online documentation.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text are shown as follows: "This alias changes the behavior of `ls` by calling it with the `color` flag every time you type it, instead of using its more vanilla version."

A block of code is set as follows:

```
zstyle ':completion:*:descriptions' format '%B%d%b'
zstyle ':completion:*:messages' format %d
zstyle ':completion:*:warnings' format 'No matches for: %d'
```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```
autoload -Uz compinit
compinit

```

Any command-line input or output is written as follows:

```
$ zsh --version
zsh 5.0.2 (x86_64-apple-darwin12.3.0)

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "Should your operating system greet you with a polite **zsh not found** message, that's ok though; otherwise, you won't be reading these lines."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.

# Reader feedback

Feedback from our readers is always welcome. Let us know what you think about this book—what you liked or may have disliked. Reader feedback is important for us to develop titles that you really get the most out of.

To send us general feedback, simply send an e-mail to `<[feedback@packtpub.com](mailto:feedback@packtpub.com)>`, and mention the book title through the subject of your message.

If there is a topic that you have expertise in and you are interested in either writing or contributing to a book, see our author guide on [www.packtpub.com/authors](http://www.packtpub.com/authors).

# Customer support

Now that you are the proud owner of a Packt book, we have a number of things to help you to get the most from your purchase.

## Downloading the example code

You can download the example code files for all Packt books you have purchased from your account at [http://www.packtpub.com](http://www.packtpub.com). If you purchased this book elsewhere, you can visit [http://www.packtpub.com/support](http://www.packtpub.com/support) and register to have the files e-mailed directly to you.

## Errata

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/support](http://www.packtpub.com/support), selecting your book, clicking on the **errata** **submission** **form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded to our website, or added to any list of existing errata, under the Errata section of that title.

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.