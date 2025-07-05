# Preface

The story of Tizen begins on February 11, 2011, when Nokia announced a partnership with Microsoft, which led to the end of MeeGo. Shortly after that, Intel and Samsung joined forces and started the Tizen project. Tizen is a trademark of the Linux Foundation, and as of today, more than 40 leading companies are directly involved in the project.

Nowadays, the mobile market is under the control of the duopoly between Android and iOS, who make up about 90 percent of all mobile devices. The control of the other 10 percent of devices is spread primarily between other corporations such as Microsoft and BlackBerry. Innovations are slowing down because of patent wars, proprietary APIs, closed application stores, and billing platforms. As an engineer, I have had to communicate many times that some requested features of a mobile application cannot be implemented because of restrictions of the operating system or just because of missing APIs.

Tizen is here, and it is bold enough to try to change the status quo. It is an innovative open source project that provides new opportunities on the mobile marketplace to application developers, hardware vendors, and telecommunication operators. To guarantee transparency and to prevent any single entity from taking control of the whole platform, Tizen is under open source governance. The objective is to establish a sustainable community of both companies and individuals, to put it at the center of the development process and to create a competitive alternative to the Android-iOS duopoly. Community members are encouraged to contribute and know what to expect from Tizen. Even smaller companies can get involved in the development of the platform, and their contributions can be accepted alongside the contributions of large corporations. Every contributor can participate in defining the future of Tizen. The main benefit for operators and OEMs is that they can customize the software and the services of the platform to match their own requirements and goals.

Tizen targets different computer architectures and form factors: from smartphones, tablets, personal computers, TVs, and photo cameras to home appliances and vehicles. The world and its technologies are moving forward, and the number of Internet-enabled devices is growing fast. New solutions are required to improve the communication between all these devices and to make our life easier. Tizen is here to fill the gap by providing flexibility and standardized solutions through open source software.

The main purpose of this book is to support you in your open source journey and to help you in your quest to develop user-friendly and innovative mobile applications and services.

# What this book covers

## Part 1 – Getting Started with Tizen

[Chapter 1](ch01.html "Chapter 1. The Tizen SDK"), *The Tizen SDK*, offers an introduction to the Tizen software development kit and its tools, a step-by-step installation guide for Windows, Mac OS, and Linux, as well as an SDB user guide.

[Chapter 2](ch02.html "Chapter 2. Introduction to the Tizen Ecosystem"), *Introduction to the Tizen Ecosystem*, gives an overview of the Tizen web and native application development process as well as guides for publishing and selling applications through the Tizen store.

## Part 2 – Creating Tizen Web Applications

[Chapter 3](ch03.html "Chapter 3. Building a UI"), *Building a UI*, is dedicated to graphical user interfaces. Tutorials for building applications with jQuery Mobile and other Tizen UI components are included. This chapter also contains guides for drawing 2D and 3D objects on an HTML5 canvas.

[Chapter 4](ch04.html "Chapter 4. Storing Data"), *Storing Data*, contains articles about storing data in files, local storage, and web SQL databases, as well as a tutorial on downloading files over the Internet.

[Chapter 5](ch05.html "Chapter 5. Creating Multimedia Apps"), *Creating Multimedia Apps*, contains programming examples for playing audio and video files, capturing images, streaming video, barcode generation, and scanning.

[Chapter 6](ch06.html "Chapter 6. Developing Social Networking Apps"), *Developing Social Networking Apps*, includes tutorials for developing a client web application for the most popular social networks (Facebook, Twitter, and LinkedIn).

[Chapter 7](ch07.html "Chapter 7. Managing the Address Book and Calendar"), *Managing the Address Book and Calendar*, includes articles about management of tasks, events, and alarms on the calendar, as well as management of the contacts from the address book.

[Chapter 8](ch08.html "Chapter 8. Communication"), *Communication*, is dedicated to the usage of different communication channels such as SMS, Bluetooth, NFC, and push notifications.

[Chapter 9](ch09.html "Chapter 9. Using Sensors"), *Using Sensors*, contains recipes related to hardware sensors such as the GPS, accelerometer, and gyroscope sensor. The main focus of this chapter is on location-based services, maps, and navigation.

## Part 3 – Porting and Debugging

[Chapter 10](ch10.html "Chapter 10. Porting Apps to Tizen"), *Porting Apps to Tizen*, includes options and hints for porting existing web, Android, or Qt applications to Tizen. Tutorials for running Android applications on Tizen using a compatibility layer as well as for complete porting of Android applications to HTML5 are included. Details about the community-driven port Qt for Tizen, which allows deployment of existing Qt mobile apps for Android, iOS, MeeGo, Symbian, SailfishOS, and BlackBerry 10 on Tizen devices, are also revealed.

[Chapter 11](ch11.html "Chapter 11. Debugging Apps in Tizen"), *Debugging Apps in Tizen*, contains tutorials for running and testing Tizen web applications as well as for JavaScript unit testing.

[Chapter 12](ch12.html "Chapter 12. Porting Tizen to Hardware Devices"), *Porting Tizen to Hardware Devices*, is a getting started guide for building embedded control systems powered by Tizen. Brief tutorials for building Tizen platform images and booting them on ARM and x86 devices are included. The information in this chapter is useful for the development of new systems or porting existing embedded control systems such as IVI or home automation to the Tizen software platform.

# What you need for this book

Basic knowledge of general programming in web development is required. To follow the tutorials and experiment with the examples, you will need a decent development computer with Windows, Mac OS X, or Ubuntu.

# Who this book is for

This book is suitable for:

*   Mobile application developers
*   Web developers who are interested in developing mobile applications
*   Developers interested in porting existing web, Android, and/or Qt applications to Tizen
*   Tizen developers who would like to improve their skills and knowledge

The main focus of the book is Tizen web application development. It does not contain any tutorials on basic HTML5, JavaScript, or CSS programming; so, a decent knowledge of these technologies is required. The book is also suitable for developers without any experience in Tizen or any other mobile platform but who have basic knowledge about web technologies.

The third part of this book is more advanced, and it might attract the attention of people developing embedded control systems, IVI (In-Vehicle Infotainment), or home automation systems and applications. This part of the book might also be useful to QA (Quality Assurance) engineers as it contains guidelines for both manual and automatic application testing.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "Our example inherits UiApp and overloads several methods including `OnAppInitializing()` and `OnAppInitialized()`."

A block of code is set as follows:

```
<link rel="stylesheet" href="tizen-web-ui-fw/latest/themes/tizen-white/tizen-web-ui-fw-theme.css" name="tizen-theme"/>
<script src="img/jquery.js"></script>
<script src="img/tizen-web-ui-fw-libs.js"></script>
<script src="img/tizen-web-ui-fw.js" data-framework-theme="tizen-white"></script>
<script type="text/javascript" src="img/main.js"></script>
```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```
helloWorldFrame* phelloWorldFrame = new (std::nothrow) helloWorldFrame;
TryReturn(phelloWorldFrame != null, false, "The memory is insufficient.");
phelloWorldFrame->Construct();
phelloWorldFrame->SetName(L"helloWorldNative");
AddFrame(*phelloWorldFrame);

```

Any command-line input or output is written as follows:

```
sdb devices

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "Launch the Tizen SDK Install Manager and click on **Next**."

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