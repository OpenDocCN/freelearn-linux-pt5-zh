# Preface

Welcome to *Instant Varnish Cache How-to*.

**Varnish Cache** is an HTTP accelerator that stands between your customers and your application servers, acting as a front-end caching solution to your website. Varnish Cache is not a complete solution and will not serve your application content on its own; instead, it will proxy customers' requests to your web server and store their responses for later usage, avoiding an unnecessary (duplicated) workload.

Putting your website behind a Varnish Cache is quick and easy. In this book, you will learn how to make the most out of your infrastructure with minimal effort.

This book is the result of a year of collected information about scalability issues and caching in general, which led to the deployment of Varnish Cache on one of the major e-commerce/marketplace in Latin America.

# What this book covers

*Installing Varnish Cache (Must know)*, introduces you to how to install Varnish Cache using its own official repository and make sure everything is set before we start configuring our Varnish daemon.

*Varnish Cache server daemon options (Must know)*, explains all the daemon options which make your Varnish Cache server act as expected, and adjust memory and CPU usage.

*Connecting to backend servers (Should know)*, defines from which backend servers Varnish will fetch the data and also create mechanisms to make sure that Varnish does not proxy a request to any offline backend.

*Load balance requests (Should know)*, explores all the possibilities of load balancing requests through application servers and creating a dedicated pool of servers to critical parts of your application system.

*The Varnish Configuration Language (Should know)*, will get you started on writing VCL code to manage all your caching policies and manipulate requests and responses.

*Handling HTTP request vcl_recv (Should know)*, explains how to choose backend servers according to what was requested by the client, block restricted content, and avoid cache lookup when it should not be performed.

*Handling HTTP request vcl_hash (Should know)*, explains how Varnish generates the hash used to store the objects in memory and how to customize it to save memory space.

*Handling HTTP request vcl_pipe and vcl_pass (Should know)*, introduces how to handle requests that should not be cached, making the data flow directly to the client.

*Handling HTTP response vcl_fetch (Should know)*, provides the opportunity to rewrite portions of the originated response to suit your needs and stitch ESI parts.

*Handling HTTP response vcl_deliver (Should know)*, explains how to clean up extra headers for server obfuscation and add debug headers to responses.

*Handling HTTP response vcl_error (Should know)*, introduces how to deliver maintenance pages during scheduled or unexpected downtime or redirect users.

*Caching static content (Should know)*, explains how to identify portions of your website that can take advantage of a cache server and how to cache them.

*Cookies, sessions, and authorization (Become an expert)*, explains how to identify unique users and parts of the website that should never be cached.

*HTTP cache headers (Should know)*, discloses what headers are important to a cache server and how to deal with their absence.

*Invalidating cached content (Should know)*,explains how to invalidate content that should no longer be delivered and how to avoid it.

*Compressing the response (Become an expert)*, explains how to save bandwidth and improve the overall speed by compressing data.

*Monitoring hit ratio (Become an expert)*, discloses how well your cache is performing and how Varnish is connecting to your backend servers.

# What you need for this book

To use this book effectively, you need the following software and operating system:

*   Linux CentOS
*   Varnish Cache
*   Oracle VirtualBox

# Who this book is for

This book targets system administrators and web developers with previous knowledge of the HTTP protocol who need to scale websites without spending money on large and costly infrastructure.

# Conventions

In this book, you will find a number of styles of text that distinguish between different kinds of information. Here are some examples of these styles, and an explanation of their meaning.

Code words in text are shown as follows: "We will install Varnish Cache on a Linux CentOS box using the `varnish-cache.org` repository."

A block of code is set as follows:

```
set obj.http.Content-Type = "text/html; charset=utf-8";
set obj.http.Retry-After = "5";
synthetic {"
```

Any command-line input or output is written as follows:

```
# sudo yum install varnish
```

**New terms** and **important words** are shown in bold. Words that you see on the screen, in menus or dialog boxes for example, appear in the text like this: "In case of an unexpected failure, you can redirect costumers to an **Oops! We're sorry** friendly page."

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

Although we have taken every care to ensure the accuracy of our content, mistakes do happen. If you find a mistake in one of our books—maybe a mistake in the text or the code—we would be grateful if you would report this to us. By doing so, you can save other readers from frustration and help us improve subsequent versions of this book. If you find any errata, please report them by visiting [http://www.packtpub.com/support](http://www.packtpub.com/support), selecting your book, clicking on the **errata** **submission** **form** link, and entering the details of your errata. Once your errata are verified, your submission will be accepted and the errata will be uploaded on our website, or added to any list of existing errata, under the Errata section of that title. Any existing errata can be viewed by selecting your title from [http://www.packtpub.com/support](http://www.packtpub.com/support).

## Piracy

Piracy of copyright material on the Internet is an ongoing problem across all media. At Packt, we take the protection of our copyright and licenses very seriously. If you come across any illegal copies of our works, in any form, on the Internet, please provide us with the location address or website name immediately so that we can pursue a remedy.

Please contact us at `<[copyright@packtpub.com](mailto:copyright@packtpub.com)>` with a link to the suspected pirated material.

We appreciate your help in protecting our authors, and our ability to bring you valuable content.

## Questions

You can contact us at `<[questions@packtpub.com](mailto:questions@packtpub.com)>` if you are having a problem with any aspect of the book, and we will do our best to address it.