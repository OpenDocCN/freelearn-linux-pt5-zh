# Chapter 4. Securing OpenLDAP

In Chapter 2 we installed OpenLDAP and created a basic configuration file for the SLAPD server. Then, in the last chapter, we turned our attention to LDAP operations and LDAP clients. Now we will return to the SLAPD server, but with a specific focus: **security**. We will take a look at three major security considerations with OpenLDAP: securing connections between the server and client connections, authenticating users of the directory, and specifying what data particular users can access (and in what capacity they can access it). We will look at these security considerations on a practical level and, in doing so, we will cover the following:

*   Configuring SSL and TLS to protect network data
*   Using simple binding to authenticate DNS (Domain Name System) for using the directory
*   Using SASL to provide more robust authentication services
*   Integrating SASL and client SSL/TLS certificates for authentication
*   Configuring Access Control Lists (ACLs) to establish rules about what data users can access

# LDAP Security: The Three Aspects

As we have seen already, the directory contains sensitive information. One example of such sensitive information is the `userPassword` attribute. But other information that may be considered sensitive, such as personal information or confidential information about the organization, may exist in the directory. Such information needs to be protected.

We might ask what is meant by *protection* in this case. For it is certainly not the case that we want to prevent *all* clients from seeing *everything*. What we want rather, is to allow people to get at specific pieces of the directory information. But, on the other hand, there are cases where we want to deny certain users the ability to get at certain pieces of directory information. So protecting our data becomes a matter of providing information in some cases, while denying it in other cases.

While it is possible to draw finer-grained distinctions, here we are going to consider three broad aspects of security where we want to make sure that we are protecting the directory and its information. These three aspects are as follows:

*   **Connection Security**: This is the process of protecting directory information (and client information) as it is passed between a client and the directory server. We will talk about this in the context of network security with SSL and TLS.
*   **Authentication**: This is the process of ensuring that the user who tries to access the information in the directory is who he/she/it claims to be. In this chapter we will look at two types of authentication: simple and SASL Binding. SASL stands for **Simple Authentication and Security Layer****.**
*   **Authorization**: This is the process of ensuring that an identified or authenticated user is allowed to access pieces of information within the directory. OpenLDAP ACLs are used to specify rules for authorization.

In this chapter we will look at each of these three aspects of security. By combining all three we will be able to provide suitably fine-grained protection for our directory information.

# Securing Network-Based Directory Connections with SSL/TLS

The first element of security that we will examine is network security. Most clients connect to OpenLDAP over a network interface, and client requests, as well as the server's responses, are transferred over a network.

The LDAP protocol, by default, sends and receives messages in clear text. In this case no attempt is made to obscure the data as it is being transmitted across the network. Sending in clear text has a few advantages:

*   It is easier to configure and maintain.
*   LDAP services can function faster. The process of encrypting and decrypting messages can be processor-intensive, and eliminating that processing can serve to speed things up.

But these advantages come at the cost of security. Other devices on the network may be able to intercept these unencrypted transmissions and read their contents and in doing so, they may obtain sensitive information. On a small Local Area Network (LAN) the risks may be smaller (though still present). On a large scale network, such as the Internet, the dangers are much greater.

In this section we will walk through the process of configuring **Secure Sockets Layer** (**SSL**) and **Transport Layer Security (TLS)** encryption to protect data as it is transmitted over a network. SSL and TLS are very similar, to the point where the terms are often used (acceptably) as synonyms. TLS though, is a refinement of SSL, and has been implemented in ways that are more flexible than the typical SSL implementation. The StartTLS method of securing a connection is an example.

## The Basics of SSL and TLS

OpenLDAP provides two methods for encrypting network traffic. The first is to have OpenLDAP listen on a special port for requests (port 636, the LDAPS port, is used by default). Transmissions on this port will automatically be encrypted. This method is older, introduced as an addition to LDAP v2, but it is no longer the preferred method.

The second method, which is part of the LDAP v3 standard, is to allow clients connecting over the standard port (usually port 389) to request to switch from clear text transmissions to encrypted transmissions. I will cover both configurations here.

**Secure Sockets Layer** (**SSL**) is a security process, originally developed by Netscape Communications for their web browser, designed to provide a safe way of exchanging trusted information between a server and any client on the network. There are two major features of the SSL process: establishing authenticity and conducting securely encrypted transactions.

As SSL developed and evolved it was handed over to a standard body, the **Internet Engineering Task Force (IETF)**, for standardization and continued development. IETF renamed it **Transport Layer Security (TLS)** and released version 1.0 (as RFC 2246). SSL 3.0 and TLS 1.0 do not have any notable differences, and most servers that support one also support the other. Because of their similarity and shared heritage, I refer to them jointly as SSL/TLS.

### Authenticity

Proving authenticity and providing encryption are the two major features of SSL/TLS. In regards to the first, SSL/TLS provides a way to establish the authenticity of the server (and, if desired, the client too). What this means is that SSL/TLS makes it possible for the client to be reasonably sure that the server belongs to whom it claims to belong.

Consider the case of online banking. If I use my browser to log on to my bank's website and conduct a few transactions, I want to be sure that the website I am connected to really is my bank's website, and not some other website masquerading as my bank. SSL/TLS provides tools to establish the authenticity of the server using **X.509 certificates**. An X.509 certificate has three important pieces of information:

*   Information about the individual or organization that owns the certificate
*   A public encryption key (which we will discuss in the next section)
*   The **digital signature** of a certificate authority (CA)

A certificate is designed as a sort of assurance that a server is associated with a particular individual or organization. When I contact a server that I believe to be my bank's, I want some assurance that it is, in fact, my bank's server. So one piece of information contained in the certificate is information about who owns the certificate. We can inspect this information ourselves, but since the certificate has a digital signature, it is also possible for software to computationally verify this—in a way much more reliable than reading the certificate and simply trusting that the certificate is accurate.

The digital signature is an encrypted bit of information. It is encrypted with a special "private" key that is owned by a Certificate Authority. The CA can then issue a public key that client software can use to verify that the certificate was in fact signed by the CA. The CA then, plays a very important role in establishing trust. We will discuss public and private keys in the *Encryption* section.

Certificate Authorities are responsible for issuing certificates. Ideally, a CA is a trusted source that can verify the authenticity of the certificate, and provide assurance that the certificate is really owned by the organization or individual that claims to own it.

There are a number of commercial CAs that provide certificate generation services for a price. To obtain a certificate through these services, an organization or individual must provide a certain amount of information that can be used to verify that the person or organization signing up for the certificate is legitimate. Once investigation of this material has been done, and the person or organization has paid the requisite fee, the CA issues a digitally-signed certificate.

The certificates of large CAs are included by default in most SSL-aware applications, such as popular web browsers (like Mozilla Firefox) and SSL libraries (like OpenSSL). These certificates include the public keys necessary for verifying digital signatures. Thus, when a client gets an X.509 certificate that is signed by one of these CAs, it has all of the tools it needs to verify the certificate's authenticity.

But it is possible, and often useful, for an organization or individual to simply create a locally used CA, and then use that CA to generate certificates for in-house applications. This is what we will do when we create a certificate for OpenLDAP.

Of course, certificates generated this way may not be considered reliable to users outside of your organization, but hosting an individual or organization-wide CA can be an effective way to add security to your own network, without having to purchase certificates from a commercial vendor.

### Note

Not all CAs use the same form of authoritative signing (and not all CAs charge for certificates). Some CAs, such as Cacert.org, use what is called a **web of trust** technique for establishing authenticity. In the web of trust the authenticity of a certificate is established by peers who can play the role of assuring that the certificate is owned by the person or organization that it claims to be owned by. For more information visit [http://www.cacert.org/](http://www.cacert.org/).

We have discussed the first role of SSL/TLS, establishing authenticity. Next we will turn to the second role of SSL/TLS; providing encryption services.

### Encryption

SSL/TLS provides the features required for sending encrypted messages back and forth between the client and the server. In a nutshell the process goes like this: the server sends the client its certificate, and inside the certificate (among other things) is the server's **public key**. The public key is the first half of a pair of keys. A public key can be used for encrypting a message, but not decrypting it. A second key, the **private key**, is then used for decrypting a message. The server keeps its private key to itself, but gives out its public key to any client that requests it. Clients can then send messages to the server that only the server can decrypt and interpret.

Depending on the configuration the client also sends the server its public key, which the server can use to send messages that only the client can decrypt. At this point, each can transmit encrypted messages to the other.

But there is a drawback to using public/private keys: they are slow and resource-intensive. Rather than trading all information through these public/private key combos, the client and server then negotiate a set of temporary symmetric encryption keys (which use the same key to encrypt and decrypt messages) that they will both use for the duration of the session. All traffic between the two clients is encrypted using these keys. Once the session is complete, both the client and server discard the temporary keys.

### Note

For a more detailed introduction to SSL and TLS, as well as pointers to further sources of information, see the Wikipedia entry for Transport Layer Security: [http://en.wikipedia.org/wiki/Transport_Layer_Security](http://en.wikipedia.org/wiki/ Transport_Layer_Security).

### StartTLS

As it is typically implemented, SSL requires that the server listen for encrypted traffic on a port separate from the one it uses for unencrypted traffic. All traffic that comes over the SSL port is assumed to be SSL-encrypted traffic. This means that every server that needs to provide both cleartext and encrypted services must listen on at least two different ports.

The multi-port requirement seemed to some to be unnecessary, inelegant, and wasteful. There is no reason why the client should not be able to request on a cleartext (non-SSL) connection that further communication between the client and server be encrypted. The client and server could then perform all of the SSL/TLS negotiation over the same connection, and not have to switch to another SSL/TLS-only port. This suggestion was standardized in RFC2487 as **StartTLS**.

### Tip

**Which to Pick: StartTLS or LDAPS?**

The standardized way of implementing SSL/TLS in LDAP v.3 is to use the StartTLS method. This method should be implemented whenever possible. However, external considerations (such as network firewalling or clients without StartTLS support) may require that you use LDAPS and a dedicated SSL/TLS-protected port. LDAPS support is now listed as deprecated, though it is not yet slated for removal from OpenLDAP. Both options can be used on the same server.

In a StartTLS-supporting server, if the client sends the server the command `STARTTLS` then the server will begin the TLS encryption process. Assuming the TLS negotiation is successful, the client and server will then continue their transactions using encrypted traffic.

StartTLS has the obvious advantage of requiring only one listening port per server. And, it makes it possible for clients and servers to communicate in cleartext for unimportant data, and then switch over to TLS when security becomes important. Since encryption is resource intensive, requiring extra processing power to encrypt and decrypt messages, streamlining services the StartTLS way can improve performance and free up resources for other tasks.

There is a drawback for StartTLS though. Since both encrypted and cleartext traffic are sent over the same port, the method of simply blocking a port to prevent insecure data transmissions (by using a firewall for instance) is not effective with StartTLS. Security measures must be capable of inspecting transmissions at the protocol level.

In order to improve security services in such cases, OpenLDAP provides methods of testing the **security strength factor (SSF)** of a connection to see if it is encrypted (and if so, if the encryption scheme is strong enough). We will look at SSF in more detail later in this chapter in the section on *Using* *Security* *Strength* *Factors*.

At this point, you should have a fairly good idea of how SSL and TLS function. Now we will move on to more practical matters. We will create our own CA, and our own certificate, and then configure OpenLDAP to support SSL/TLS and StartTLS.

## Creating an SSL/TLS CA

In order to create a Certificate Authority and generate certificates, you will need to have OpenSSL installed. Since many Ubuntu packages, including the OpenLDAP packages, require OpenSSL, it should be installed already.

If you build from source, as detailed in [Appendix A](apa.html "Appendix A. Building OpenLDAP from Source"), you may also enable support for SSL/TLS using the OpenSSL libraries.

### Tip

If you have a certificate already, you can skip this section and move to the *Configuring* *StartTLS* section. OpenLDAP uses certificates in the PEM format.

The first thing we will need to do is create our new CA.

While it is possible to manually configure your CA using the `openssl` command line tool, it is much simpler to use the `CA.pl` Perl script that is included with OpenSSL. This script streamlines many of the configuration options for OpenSSL, and the first thing that we will use it for is creating the environment for our new CA.

### Note

Ubuntu maintains documentation on creating a new Certificate Authority the "long way" (creating all of the files by hand). This documentation is detailed and well worth reading. While I will follow the conventions established there, I will be using the `CA.pl` script to do most of the heavy lifting ([https://help.ubuntu.com/community/OpenSSL](https://help.ubuntu.com/community/OpenSSL)).

You can put the CA environment anywhere on your system. Some prefer to keep CA files with the rest of the SSL configuration at `/etc/ssl/`. Others prefer keeping the certificate authority in a user directory so that it does not get overwritten during system upgrades (an unlikely, but possible, event). In keeping with the Ubuntu suggestion to keep CA info in a user's home directory, I will just put mine in my home directory, `/home/mbutcher/`:

```
 $ cd ~
 $ /usr/lib/ssl/misc/CA.pl -newca

```

Note that the `CA.pl` script is not in `$PATH`, so you will need to type in the entire path to the script.

### Tip

**Finding CA.pl**

Different operating system distributions will put `CA.pl` in different places. If running `which CA.pl` does not return any results, you may want to consult the man pages for SSL (`man config` or `man CA.pl`), or use the `find` or `slocate` utilities to find the `CA.pl` file.

The argument `-newca` instructs `CA.pl` to set up a new certificate authority environment. This will generate a directory structure along with a number of files.

The first thing that `CA.pl` will do is prompt you to enter a CA file:

```
$ /usr/lib/ssl/misc/CA.pl -newca
CA certificate filename (or enter to create)
```

Hit *Enter* to create a new CA certificate. `CA.pl` will then generate a new key and then prompt you for a password:

```
CA certificate filename (or enter to create)

Making CA certificate 
Generating a 1024 bit RSA private key
....++++++
...................................++++++
unable to write 'random state'
writing new private key to './demoCA/private/cakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
```

Once you have entered and re-entered your password, `CA.pl` will collect some information from you about your organization:

```
You are about to be asked to enter information that will be 
    incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or 
    a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Illinois
Locality Name (eg, city) []:Chicago
Organization Name (eg, company) [Internet Widgits]:Example.Com
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:Matt Butcher
Email Address []:matt@example.com 

Please enter the following 'extra' attributes
    to be sent with your certificate request
A challenge password []:mypassword
An optional company name []:Example.Com

```

`CA.pl` walks you through the process of creating a main certificate. The highlighted lines in the code listing are those where you will have to provide information at an interactive prompt. After setting the country, state, and city name for my locale, we set the **Organization Name** to **Example.Com**. While we left the **Organizational Unit** field blank, you can use that to further specify what part of the organization this CA is a member of.

### Note

You should consider using the same fields in your certificate that you used for your root DN when you set up your directory information tree in the previous two chapters.

Usually the **Common Name** and **Email Address** fields should contain information about the organization. Sometimes **Common Name** is used for the server name (as will be the case when we create our certificate). Sometimes, it is used for contact information. In the case that follows, we used my name and email. If the CA is to be the "official" CA for your organization, you should set this to the official contact person for certificate inquiries.

Next, `CA.pl` will begin the process of generating a certificate request for the CA certificate. In other words, `CA.pl` will create a new certificate that will be the CA's own certificate. The first step in doing this is to create a certificate request. We will need to set a challenging password for the certificate request. We can also set a company name too. With the above information, `CA.pl` will continue the process of generating a new certificate:

```
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
      Serial Number:
        bf:2f:58:47:b1:6d:31:4d
      Validity
        Not Before: Oct 10 21:34:28 2006 GMT
        Not After : Oct  9 21:34:28 2009 GMT
      Subject:
        countryName               = US
        stateOrProvinceName       = Illinois
        organizationName          = Example.Com
        commonName                = Matt Butcher
        emailAddress              = matt@example.com
      X509v3 extensions:
        X509v3 Basic Constraints: 
            CA:FALSE
          Netscape Comment: 
            OpenSSL Generated Certificate
          X509v3 Subject Key Identifier: 

07:92:9B:35:CB:B7:EE:92:A8:33:61:B0:DC:F7:88:E9:4F:06:9F:7F
    X509v3 Authority Key Identifier: 

keyid:07:92:9B:35:CB:B7:EE:92:A8:33:61:B0:DC:F7:88:E9:4F:06:9F:7F

Certificate is to be certified until 
    Oct 9 21:34:28 2009 GMT (1095 days)

Write out database with 1 new entries
Data Base Updated

```

We will be prompted to enter a pass phrase. This is the pass phrase we created first (when prompted to **Enter PEM pass phrase**). If we enter the pass phrase correctly, `CA.pl` will generate our new certificate and display its contents on the screen.

We have now created a Certificate Authority. Now we are ready to start generating a certificate to be used by SLAPD.

### Note

Due to a bug in some versions of `CA.pl`, you may have to `cd` into the `./demoCA` directory (the directory that `CA.pl -newca` created) and add a symbolic link to itself: `ln -s ./demoCA`. This is because `CA.pl` occasionally expects to find files in the current directory (`./`), which it assumes to be `demoCA/`, and sometimes it expects to find files in `./demoCA` (which, of course, is equivalent to `demoCA/demoCA/`). You can also fix this simply by editing the `dir=` line under `[CA_default]` in the `/etc/ssl/openssl.cnf` file, and setting it to an absolute path.

## Creating a Certificate

Creating a certificate is a two-step process:

1.  We need to generate the Certificate Request.
2.  We need to sign the request with the CA's signature.

Let's see these steps in detail.

### Creating a New Certificate Request

The first step in creating a valid SSL certificate is to create a Certificate Request. In the process, we will specify what information we want to show up on the certificate.

There are a few ways to generate Certificate Requests. For example, you can use the `openssl` command line tool and specifying a number of command line parameters. But, following our previous example, we will use `CA.pl` and let the application prompt us for information as is necessary.

To generate a new request we will run `CA.pl -newreq`. In the next example the highlighted lines are lines that require us to enter information:

```
$ /usr/lib/ssl/misc/CA.pl -newreq
Generating a 1024 bit RSA private key
.....++++++
.....................++++++
unable to write 'random state'
writing new private key to 'newkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be 
    incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or 
    a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Illinois
Locality Name (eg, city) []:Chicago
Organization Name (eg, company) [Internet Widgits]:Example.Com
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:example.com
Email Address []:matt@example.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```

This should look familiar. It is similar in most respects to the process of generating a Certificate Authority.

First, we will be prompted to enter a pass phrase. We will use this pass phrase in a few moments.

Next, we will be asked to supply information about the organization that this certificate will represent. As before the fields are Country Name, State/Province Name, Locality, Organization Name, Organizational Unit, Common Name (of the contact person), and the Email for the contact person. Again, as before, we entered the information for Example.Com.

This time, however, we set the Common Name field to be the domain name of the server that the certificate is for—`example.com`. It is very important that you use the correct domain name for the server. During the certificate negotiation process clients will check the Common Name field to see if it matches the domain name of the server. If the names do not match the user may get an error message, or the client application may simply terminate the connection.

The extra *password* and *optional* *company* *name* are sometimes used in the certificate request process. Since we are doing the requesting and the signing ourselves we don't need to complete either of these fields.

Now we should have two files in the CA directory:

*   One called `newreq.pem`, which contains a base-64 encoded representation of our certificate request
*   One called `newkey.pem`, which contains the base-64 encoded private key

We are now ready to move on to the second step.

### Signing the Certificate Request

The Certificate Request has all of the information required for a certificate, but it still lacks the digital signature of the CA. The next step, then, will be to use the CA we created previously to sign this new certificate. To do this, we will run `CA.pl -signreq`:

```
$ /usr/lib/ssl/misc/CA.pl -signreq
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
      Serial Number:
        ba:49:df:f5:8e:7e:77:c2
      Validity
        Not Before: Oct 12 21:23:49 2006 GMT
        Not After : Oct 12 21:23:49 2007 GMT
      Subject:
        countryName               = US
        stateOrProvinceName       = Illinois
        localityName              = Chicago
        organizationName          = Example.Com
        commonName                = example.com
        emailAddress              = matt@example.com
      X509v3 extensions:
        X509v3 Basic Constraints: 
          CA:FALSE
        Netscape Comment: 
          OpenSSL Generated Certificate
        X509v3 Subject Key Identifier: 

47:DD:90:8F:79:90:2E:C0:CC:B3:95:62:35:C4:D8:6C:5D:A2:EE:88
     X509v3 Authority Key Identifier: 
                keyid:6B:FB:66:33:5D:DB:CC:40:42:D7:71:F7:F0:D0:7C:94:3E:8F:CD:58

Certificate is to be certified until 
    Oct 12 21:23:49 2007 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
Signed certificate is in newcert.pem

```

The `CA.pl -signcert` command looks for `newreq.pem` and then begins the signing process. First, we need to enter the pass phrase for the CA. If that is correct, then `CA.pl` will display the certificate in `newreq.pem` and ask if we want to sign it. Finally, it will ask us to commit these changes.

Once the changes are committed a new file will be created, named `newcert.pem`.

There are two important files that we now have:

*   `newkey.pem`, which contains the private key
*   `newcert.pem`, which contains the signed certificate.

We've just got a few loose ends to tie up, and then we can move on to configuring SLAPD to use SSL/TLS.

### Configuring and Installing the Certificates

We have only three more steps to do, here. The first one has to do with the pass phrase we set on our certificate.

#### Remove the Pass Phrase from the Key

Be very careful here! When generating our certificate request, we set a pass phrase on the certificate. This encrypted the `newkey.pem` file with a pass phrase.

If you use a key file that is encrypted with a pass phrase, then every time you use this certificate, you will have to enter a password. This means, in our case, that every time we start OpenLDAP, we will have to enter a pass phrase. Unless we have stringent security requirements (and are willing to put up with the hassle of typing the pass phrase every time we start or restart the server), we probably do not want the key file to be encrypted.

So, we will need to create an unencrypted version of the key file using the `openssl` command:

```
 $ openssl rsa < newkey.pem > clearkey.pem

```

This is what we get:

```
Enter pass phrase:
writing RSA key
```

In this example the command `openssl rsa` executes the OpenSSL RSA tool, which will decrypt the key. Using `< newkey.pem`, we sent the file contents of `newkey.pem` into `openssl` to be decrypted. Then, using `> clearkey.pem` we directed `openssl` to write the cleartext key file to the `clearkey.pem` file. In order to complete this operation, `openssl` prompts for the pass phrase. Now `clearkey.pem` has the unencrypted private key for our certificate.

### Note

The `clearkey.pem` file now contains an unencrypted private key. This file should be protected from misuse. You should set strict permissions on this file so that other users of the system cannot access it.

### Tip

**The openssl Program**

The `openssl` program performs dozens of SSL-related functions, from generating certificates to emulating a network-based SSL client. Its syntax is notoriously difficult though. That is why we have been using the `CA.pl` wrapper script to perform common tasks. But some tasks can only be done with the `openssl` command. Should you need them though, `openssl` has excellent man pages: `man openssl`.

#### Relocate the Certificates

The second task is to move our new certificate and key to a useful location on the server, and give the PEM files useful names as well. If this certificate is to be used by lots of different services, it might make sense to locate it in the shared directory. But for our cases we will only be using the SSL certificate for LDAP, so we can put the files in `/etc/ldap/` (or `/usr/local/etc/openldap/` if you built from source).

The two files with which we are concerned are `newcert.pem` and `clearkey.pem`. We need to rename and move those two keys:

```
 $ sudo mv cacert.pem /etc/ldap/example.com.cert.pem
 $ sudo mv clearkey.pem /etc/ldap/example.com.key.pem

```

Now, we need to set permissions and ownership on the certificate files. Since we did not add a pass phrase to the key, we should also make sure that only the OpenLDAP user can read the key file:

```
 $ sudo chown root:root /etc/ldap/example.com.*.pem
 $ sudo chmod 400 /etc/ldap/example.com.key.pem

```

The first line changes the owner and group of the two PEM files to the `root` user and the `root` group. The second line sets the mode so that only the owner can read the file, and no one else has any access.

If you are running OpenLDAP as a user other than root (and it is a good idea to do so), then the files should be owned by that user instead of root; for example `chown oenldap example.com.*.pem`.

#### Install the CA Certificate

The third task is to install the CA's public certificate so that other applications on the system can use that certificate to verify the authenticity of the certificate we just generated. First, we need to copy the CA certificate to the local certificate database for Ubuntu. In the process we will give it a user-friendly name:

```
 $ sudo cp cacert.pem /usr/share/ca-certificates/Example.Com-CA.crt

```

Then, edit the `/etc/ca-certificates.conf` file, and add `Example.Com.crt` at the end of the file.

Finally, run `update-ca-certificates`:

```
$ sudo update-ca-certificates
Updating certificates in /etc/ssl/certs....done.
```

The CA certificate has now been installed. The `/etc/ssl/certs` directory is now the authoritative source for CA certificates.

### Note

UNIX and Linux systems other than Ubuntu and Debian may not have the `update-ca-certificates` script. Consult the system documentation to find out how to update the certificate database on such systems.

#### Optional: Clean Up

If you want, you can do a little clean-up in the CA directory. Delete the encrypted key file and the certificate request file, both of which are in the `demoCA/` directory:

```
 $ rm newkey.pem newreq.pem

```

Also, make sure `clearkey.pem` is no longer present in the `demoCA/` directory.

Now we are ready to configure OpenLDAP to use our new certificates. First, we will configure StartTLS support, which is the easiest, then we will configure SSL/TLS support on the LDAPS port, port 636.

## Configuring StartTLS

In the previous sections we created our new certificate and key, and placed the two files in the `/etc/ldap` directory. In this section we will set up StartTLS (which we introduced earlier in this chapter in the StartTLS section). Setting up StartTLS requires only a few extra lines in the `slapd.conf` file.

Again, StartTLS is the standard way (according to RFC 4511) of providing SSL/TLS security to OpenLDAP. For security reasons support for StartTLS should be provided whenever practical.

In the `slapd.conf` file, just before the `BDB Database Configuration` section, insert the SSL/TLS options:

```
###########
# SSL/TLS #
###########
TLSCACertificatePath    /etc/ssl/certs/
TLSCertificateFile      /etc/ldap/example.com.cert.pem
TLSCertificateKeyFile   /etc/ldap/example.com.key.pem
```

Basically, there are only three directives we need to specify to get StartTLS working:

*   The first directive, `TLSCACertificatePath`, tells SLAPD where to find all of the CA certificates that it will need for verifying certificates. The definitive location is, as we saw before, the `/etc/ssl/certs/` directory.
*   The second directive, `TLSCertificateFile`, specifies the location of the signed certificate.
*   The third directive, `TLSCertificateKeyFile`, specifies the location of the corresponding key file, which has the private encryption key for the certificate.

### Note

There are a handful of other TLS-specific directives that allow you to provide detailed constraints on TLS connections (such as which suites of ciphers can be used, and whether the client needs to provide a certificate to the server). Complete documentation on these can be found in the TLS section of the `slapd.conf` man page: `man slapd.conf`.

That's all we need to get SLAPD to perform StartTLS. Restart SLAPD so that the changes take effect.

## Configuring Client TLS

We do need to add a directive or two to `ldap.conf`—the configuration file that the OpenLDAP clients use. As with SLAPD, we need to direct the clients to the correct location of the new CA certificate so that they can verify the server certificate.

At the bottom of the `ldap.conf` file we can add the appropriate directive:

```
TLS_CACERTDIR /etc/ssl/certs
```

Clients will use this directive to locate the CA certificates for checking digital signatures on the certificates they get from servers. If you know that you are only going to use certificates signed by a specific CA, you can use the `TLS_CACERT` directive to point to a specific CA certificate file, instead of a directory containing one or more certificates.

By default, OpenLDAP clients always perform a check on the digital signatures. If a server sends a certificate that was signed by a CA other than those at `/etc/ssl/certs/` (or whatever directory `TLS_CACERTDIR` points to), then the client will close the connection and print an error message to the screen.

Sometimes though, the correct CA certificate is not available, and it is worthwhile to get the encryption support of TLS even if it is not possible to verify the identity of the server.

In such cases you may find it necessary to change the way OpenLDAP clients perform identification checks. For example, it might be desirable to try to verify the certificate, but to continue with the connection even if there is no appropriate CA locally. To accomplish this, use the following directive in `slapd.conf`:

```
TLS_REQCERT allow
```

In this case, if there is no CA certificate or if the certificate sent cannot be verified, the session will continue, rather than exiting with an error message. `TLS_REQCERT` has a few different levels of checking, ranging from `strict` (always verify) to `never` (do not even bother trying to verify certificates).

At this point, we can use `ldapsearch` to test a connection. To instruct a client to use StartTLS, we need to use the `-Z` flag. But if just `-Z` is specified, if the client fails TLS negotiation with the server, it will continue with the transaction in clear text. In other words, with `-Z`, TLS is preferred, but not required. To make TLS required, we will add an extra z to the flag, making it `-ZZ`:

```
 $ ldapsearch -LLL -x -W -D 'cn=Manager,dc=example,dc=com' -H \ 
 ldap://example.com -ZZ '(uid=manny)'

```

This should prompt for a password and then return one result:

```
Enter LDAP Password: 
dn: uid=manny,ou=Users,dc=example,dc=com
sn: Kant
uid: immanuel
uid: manny
ou: Users
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
givenName: Manny
cn: Manny Kant
```

If the result comes back like this, then TLS was successfully configured. But TLS can be difficult to get configured because it is strict by design. Small errors in configuration (like using a domain name that differs from the one in the CN field of the certificate) can prevent TLS from working. Consider this example:

```
$ ldapsearch -LL -x -W -D 'cn=Manager,dc=example,dc=com' -H \ 
    ldap://localhost -ZZ '(uid=manny)'

ldap_start_tls: Connect error (-11)
    additional info: TLS: hostname does not match CN in peer 
    certificate

```

In this case, the host name specified on the command line (`localhost`) differed from the one in the CN field of the certificate (`example.com`). Even though, in this case, the two domain names are hosted on the same system, TLS will not accept the mismatch.

Other common errors in TLS are:

*   Reversing the values of the `TLSCertificateFile` and the `TLSCertificateKeyFile` directives
*   Forgetting to install the CA certificate (which results in an error indicating that the server certificate cannot be verified)
*   Forgetting to set the client CA path correctly in `ldap.conf`
*   Setting the read/write permissions (or the ownership) on the key file (or the certificate file) in such a way that the SLAPD server cannot read it

While OpenLDAP can be forgiving in many areas, TLS configuration is not one of them. It pays to take extra care when configuring TLS and SSL.

## Configuring LDAPS

Now that we have configured TLS, we need to take only a few additional steps to enable SSL/TLS on its own port. The traditional port for running dedicated TLS/SSL-protected LDAP traffic is port 636, the LDAPS port.

Most of the time it is better to use StartTLS. However, network considerations (like clients that do not support StartTLS or policies dictating mandatory blocking on ports that allow non-encrypted text) might warrant using LDAPS.

Keep in mind that LDAPS and StartTLS can both be used for the same server. SLAPD can accept LDAPS traffic on a dedicated port, and continue to provide the StartTLS feature on an LDAP port.

### Note

Like the StartTLS configuration, this configuration requires that the `slapd.conf` file have the `TLSCertificateFile`, `TLSCertificateKeyFile`, and `TLSCACertificateDir` directives set.

Getting SLAPD to listen on this port requires passing an additional parameter when starting `slapd`. In Ubuntu, as with other Debian-based distributions, configuration parameters can be set in the `/etc/defaults/slapd` file. In that file we just need to set `SLAPD_SERVICES`. When the start script is executed, SLAPD will start all of the services listed here.

```
SLAPD_SERVICES="ldap:/// ldaps:///"
```

The given code tells SLAPD to listen on all available IP addresses on both the default LDAP (port 389) and the default LDAPS (port 636). If we wanted SLAPD to only listen on one address for LDAP traffic, but all addresses for LDAPS traffic, we could replace the above with:

```
SLAPD_SERVICES="ldap://127.0.0.1/ ldaps:///"
```

Here, the `ldap://127.0.0.1/` tells SLAPD to only listen on the loopback address for LDAP traffic, while `ldaps:///` indicates that SLAPD should listen for LDAPS traffic on all of the IP addresses configured for this host. You will need to restart SLAPD in order for these changes to take effect.

Similarly, if you built from source and want to start `slapd` directly, the `-h` command line flag lets you specify which services to start:

```
/usr/local/libexec/slapd -h "ldap:/// ldaps:///"
```

That is all there is to configuring LDAPS. We can now test it with `ldapsearch`:

```
 ldapsearch -LL -x -W -D 'cn=Manager,dc=example,dc=com' -H \ 
 ldaps://example.com '(uid=manny)'

```

There are two crucial differences between this `ldapsearch` and the ones we used when testing StartTLS:

*   The protocol for the URL specified after the `-H` flag is `ldaps://` rather than `ldap://`.
*   There is no `-Z` or `-ZZ` flag here. Those flags tell the client to send the StartTLS command, and SSL/TLS over a dedicated port do not recognize the StartTLS command.

If you get an error doing the given search, but StartTLS is working properly, the first place to look is at the firewall settings. Typically, firewalls allow traffic on port 389, but block 636\. It is also useful to make sure that the server is actually listening on port 636\. You can check this from a shell prompt using `netstat –tcp -l`, which will print out a list of what ports are being used. If LDAPS (636) does not show up, then check `/etc/defaults/slapd` again to make sure that the `SLAPD_SERVICES` directive is set correctly.

### Debugging with the OpenSSL Client

In some cases it is useful to be able to connect to SLAPD over LDAPS and watch the certificate processing. The `openssl` program can do this with its built-in `s_client` client application:

```
 $ openssl s_client -connect example.com:636

```

The `-connect` parameter takes a host name followed by a colon and a port number. When this command is run, `openssl` will connect to a remote server using SSL, and perform the certificate negotiation. The entire negotiation process is written to the screen. If certificate negotiation succeeds, then `openssl` leaves the connection open, and you can type in raw protocol commands at the command line. To exit, just hit *CTRL*+*C*.

Now we have both StartTLS and TLS/SSL working. We have one more short item to cover in this section, and then we will move on to authentication.

## Using Security Strength Factors

There are advantages to running StartTLS. It is simpler to configure, it is easier (in many respects) to debug, and complex transactions can switch back and forth from cleartext to encryption as needed.

But there is one clear drawback: we can use a standard firewall to block non-encrypted traffic when all clear text goes over one port and all encrypted traffic goes over another. But when both go over the same port, many firewalls can't do much to verify that the traffic is secure.

But OpenLDAP does provide some tools for implementing this sort of security in SLAPD, instead of in a firewall.

OpenLDAP can examine the integrity and encryption state of a connection and, based on those features, assign a **Security Strength Factor (SSF)** to that connection. An SSF is a numeric representation of the strength of the protective measures used.

Most of the SSF numbers simply reflect the key length of the encryption cipher. For example, since the maximum key length for **DES** is 56, when a connection is protected using DES, the SSF is 56\. **Triple-DES (3DES)**, which is the cipher used by default in Ubuntu's OpenSSL configuration, has a key length of 112\. Hence, its SSF is also 112\. The **AES** cipher, which is strong and can be computed quickly, can use different key sizes. AES-128 uses a 128-bit key, while AES-256 uses a 256-bit key. In the case of AES then, the SSF will reflect the key size.

There are two special SSF numbers: 0 and 1\. An SSF of 0 indicates (as might be expected) that no security measures have been implemented. An SSF of 1 indicates that only integrity checking on the connection is being done.

OpenLDAP can use SSF information to determine whether a client is allowed to connect to the directory. SSF information can also be used in ACLs and in SASL configuration, effectively allowing complex rules to be built as to what conditions a client connection must satisfy before getting access to perform certain operations on the directory.

We will look at SASL authentication and ACLs later in this chapter, but right now we will look at using SSFs in the `security` directive in `slapd.conf` as a way of specifying how secure a connection must be in order to access the database.

### The security Directive

The `security` directive can be used in two different contexts in `slapd.conf`. If it is put near the top of the file, before any backend databases are defined, then it is placed in the *global* context and will apply to all connections. On the other hand, if the security directive is placed within a backend definition, then it will only be applied to that particular database. For example, consider a case where there are two backends:

```
include /etc/ldap/schema/core.schema

modulepath /usr/local/libexec/openldap
moduleload back_hdb
# Other configuration directives ...

# DB 1:
database hdb
suffix "ou=Users,dc=example,dc=com"
# More directives for DB 1...
# DB 2:
database bdb
suffix "ou=System,dc=example,dc=com"
# More directives for DB 2...
```

This partial example of a `slapd.conf` file defines two directory backends. Now, if the `security` directive is used before the first database is defined (namely before the line that says `database hdb`), then it will be applied globally to all connections.

But if we wanted to allow non-encrypted connections to DB 2, but allow only well-encrypted connections to DB 1 (which houses all of our user entries), we could use separate `security` directives:

```
include /etc/ldap/schema/core.schema

modulepath /usr/local/libexec/openldap
moduleload back_hdb
loglevel stats
# Other configuration directives ...

# DB 1:
database hdb
suffix "ou=Users,dc=example,dc=com"
security ssf=112
# More directives for DB 1...

# DB 2:
database bdb
suffix "ou=System,dc=example,dc=com"
security ssf=0
# More directives for DB 2...
```

Note the addition of the two highlighted lines—two separate `security` directives, one for each database backend.

Now, restarting the directory (note that the `loglevel` is set to `stats`), we can test out the security parameters with `ldapsearch`. First, we will try to search the `Users` OU with a non-TLS connection:

```
 $ ldapsearch -x -W -D 'uid=matt,ou=Users,dc=example,dc=com' -b \ 
 'ou=Users,dc=example,dc=com' '(uid=david)' uid

```

In the log we see entries like this:

```
conn=0 fd=12 ACCEPT from IP=127.0.0.1:48758 (IP=0.0.0.0:389)
conn=0 op=0 BIND dn="uid=matt,ou=Users,dc=example,dc=com" method=128
conn=0 op=0 RESULT tag=97 err=13 text=confidentiality required
conn=0 fd=12 closed (connection lost)
connection_read(12): no connection!
```

The third line indicates that the server returned error number 13: `confidentiality required`. This is because we did not do anything to protect the connection. Using simple authentication (which is not encrypted) and failing to use TLS/SSL resulted in the client connection having an effective SSF of 0.

Next, let's do the same search with TLS turned on:

```
 $ ldapsearch -x -W -D 'uid=matt,ou=Users,dc=example,dc=com' -b \ 
 'ou=Users,dc=example,dc=com' -Z '(uid=david)' uid

```

Note that in this example, the `-Z` flag is included to send the StartTLS command. Now, the server log says:

```
conn=1 fd=12 ACCEPT from IP=127.0.0.1:44684 (IP=0.0.0.0:389)
conn=1 op=0 STARTTLS
conn=1 op=0 RESULT oid= err=0 text=
conn=1 fd=12 TLS established tls_ssf=256 ssf=256
conn=1 op=1 BIND dn="uid=matt,ou=Users,dc=example,dc=com" method=128
conn=1 op=1 BIND dn="uid=matt,ou=Users,dc=example,dc=com" mech=SIMPLE ssf=0
conn=1 op=1 RESULT tag=97 err=0 text=
```

There are a few things to note about this result. On the second line, OpenLDAP reports that it is doing StartTLS. Two lines later it reports: `TLS established tls_ssf=256 ssf=256`. This line indicates that the TLS connection has an SSF of 256 (since the connection is using AES-256), and that the total SSF of the connection is 256.

If you look a few lines lower, on the second line that begins `BIND`, you will notice that there another SSF is reported: `ssf=0`. Why?

OpenLDAP measures SSF on various aspects of the connection. First, as we can see above, it checks the SSF of the network connection. TLS/SSL connections are assigned an SSF based on their cipher strength.

But during the bind phase when the client authenticates to the directory, OpenLDAP also measures the SSF of the authentication mechanism. The simple (`mech=SIMPLE`) authentication mechanism does not encrypt the password, and so it is always given an SSF of 0.

The total SSF for the connection, however, remains at 256, with the TLS SSF being 256 and the SASL SSF at 0.

#### A Fine-Grained security Directive

The `security` directive that we have looked at so far is basic. It simply requires that the overall SSF be 112 (3DES encryption) or greater, but we can make it more specific.

For example, we can simply require that any TLS connection have at least a 128 bit key:

```
security tls=128
```

This will require that all incoming connections use TLS with a strong (128 bit or greater) cipher.

### Note

In some cases it is desirable to define which TLS/SSL ciphers or cipher families will be used. This cannot be done with the `security` directive. Instead, you will need to use the `TLSCipherSuite` directive, which will allow you to give a detailed specification for which ciphers are acceptable for TLS/SSL connections.

Or, if we only wanted to define a strong SSF for connections that try to perform a simple bind (as opposed to an SASL bind), then we can specify an SSF just for simple binding:

```
security simple_bind=128
```

This will require that some strong TLS cipher be used to protect the authentication information.

### Tip

If you plan to allow simple binding, and you are running on a non-secure network, you are strongly advised to configure TLS/SSL and require TLS encryption during the bind operation using the `security` directive.

You can also use the `update_ssf` keyword in the `security` directive to set the SSF necessary for updating operations. Thus you could specify that only low-grade encryption is needed for reading the directory, but high-grade encryption must be used for performing updates to directory information:

```
security ssf=56 update_ssf=256
```

In the coming section, we will look at SASL configuration. You can use the `security` directive to set SSF for SASL binding as well using the `sasl=` and `update_sasl=` phrases.

Finally, in rare cases where OpenLDAP is listening on a local socket (that is, `ldapi://`), you can use `security transport=112` (or whatever cipher strength you desire) to ensure that traffic coming over that socket is encrypted.

At this point, we have completed our examination of SSL and TLS. Next, we will move on to the second of our three aspects of security: authentication.

# Authenticating Users to the Directory

As we have seen earlier in the book, OpenLDAP supports two different methods of binding (or authenticating) to the directory. The first is to use simple binding. The second is to use SASL binding. In this part we will look at each of these two methods of authentication.

It is not necessary to choose one or the other. OpenLDAP can be configured to do both, at which point it is up to the client as to which method will be used. Simple binding is easier to configure (there is very little configuration that must be done). But SASL is more secure and more flexible, though these benefits come at the cost of added complexity.

The basics of the bind operation and the authentication process are covered early in Chapter 3\. While we will review some of that materials here, you may find it useful to glance back at that section.

## Simple Binding

The first form of authentication we will look at is simple binding. It is simple not necessarily from the user's perspective, but it is definitely easier to configure, and the process of binding is easier on the server, too, since less processing is needed.

To perform a simple bind the server requires two pieces of information: a DN and a password. If both the DN and the password fields are empty then the server attempts to bind as the Anonymous user.

During a simple bind the client connects to the server and sends the DN and password information to the server, without adding any additional security. The password, for example, is not specially encrypted.

If the client is communicating over TLS/SSL, then the whole transaction will be encrypted, and so the password will be safe. If the client is not using TLS/SSL then the password will be sent over the network in cleartext. This, of course, is a security issue, and should be avoided (perhaps by using the `security` directive discussed in the previous section, or by using an SASL bind instead of a simple bind).

There are two common ways in which client applications attempt to perform a simple bind. The first is sometimes called **Fast Bind**. In a Fast Bind, the client supplies a full DN (`uid=matt,ou=users,dc=example,dc=com`) and also a password (`myPassword`). It is faster than the common alternative (binding as anonymous and searching for the desired DN).

### Note

**Cyrus SASLAuthd**, which provides SASL authentication services to other applications, is the application in which the term "Fast Bind" was first used. SASLAuthd is a useful tool for providing SASL authentication services. We will look at it again in the next section. Nowhere in the OpenLDAP documentation, is the term "Fast Bind" used.

The directory first performs, as the anonymous user, an **auth** access on the `userPassword` attribute of the DN that the client supplies. In an auth access the server compares the value of the supplied password to the value of the `userPassword` stored in the directory. If the `userPassword` value is hashed (with, for example, SSHA or SMD5), then SLAPD hashes the password that the user supplies, and then compares the hashes. If the values match, OpenLDAP binds the user and allows it to perform other LDAP operations.

### Note

The OpenLDAP command-line clients, when used with the `-x` option, perform simple binding. The clients require that you specify the entire user DN and a password, and they then perform a Fast Bind.

That's a Fast Bind. But there is a second common method of doing a simple bind—a method designed to eliminate the requirement that the user know an entire DN.

In this second method (which is not, incidentally, called a "slow bind"), the client application requires that the user only know some particular unique identifier—usually the value of `uid` or `cn`. The client application then binds to the server as anonymous (or another pre-configured user) and performs a search for a DN that contains the matching attribute value. If it finds one (and only one) matching DN, then it re-binds, using the retrieved DN and the user-supplied password.

Usually, client applications that use simple bind will need a base DN. The second method of performing a simple bind requires one additional piece of information not required in a Fast Bind: a search filter. The filter is usually something like `(&(uid=?)(objectclass=inetOrgPerson))`, where the question mark (`?`) is replaced by the user-supplied value.

### Using an Authentication User for Simple Binding

While it is more convenient for the user when only a user ID or a CN is required, the second method we have seen may raise an additional concern: the Anonymous user, in order to perform the search, must have *read* access to all user records in the directory. This means that anyone can connect to the directory (remember, Anonymous has no password) and perform searches.

In many cases this isn't a problem. Allowing someone to see a list of all the users in the directory may not be a security concern at all. But in other cases, such access would not be acceptable.

One way to work around this problem is to use a different user (rather than Anonymous) to perform the search for the user's DN. In the last chapter, we created just such an account. Here is the LDIF record we used:

```
# Special Account for Authentication:
dn: uid=authenticate,ou=System,dc=example,dc=com
uid: authenticate
ou: System
description: Special account for authenticating users
userPassword: secret
objectClass: account
objectClass: simpleSecurityObject
```

The purpose of this account is to log into the server and perform searches for DNs. In other words, it conducts the same job as the Anonymous user, but it adds a little more security, since clients that use the `uid=authenticate` account will have to have the appropriate password, too.

To make this clear let's look at the case where a client, configured to use the Authenticate account, binds a user that identifies himself as `matt` with the password `myPassword`.

Here's a step-by-step breakdown of what happens when doing a bind operation this way:

1.  Client connects to the server and starts a bind operation with the DN `uid=autenticate,ou=system,dc=example,dc=com` and the password `secret`.
2.  The server, as Anonymous, compares the Authenticate password, `secret`, with the value of the `userPassword` attribute for the `uid=autenticate,ou=system,dc=example,dc=com` record.
3.  If the above succeeds, then the client (now logged in as the Authenticate user) performs a search with the filter: `(&(uid=matt)(objectclass=inetOrgPerson))`. Since `uid` is unique, the search should return either 0 or 1 record.
4.  If one matching DN is found (in our case, it would be `uid=matt,ou=user,dc=example,dc=com`), then the client tries to re-bind as this DN, and using the password the user initially supplies to the client (`myPassword`).
5.  The server, as Anonymous, compares the user-supplied password, `myPassword`, with the value of the `userPassword` attribute of `uid=matt,ou=user,dc=example,dc=com`.
6.  If the password comparison succeeds then the client application can continue performing LDAP operations as `uid=matt,ou=user,dc=example,dc=com`.

The process is lengthy and it requires that the client application be configured with bind DN and password information for the Authenticate user, but it adds an additional layer of security to an Anonymous bind and search.

In this section, we have looked at three different ways of performing a simple bind. Each of these methods is useful in particular circumstances, and when used in conjunction with SSL/TLS, simple binding does not pose a significant security threat when the password is transmitted across the network.

### Tip

**Simple Binding Directives in slapd.conf**

There are only a few directives in `slapd.conf` that have any bearing on simple binding. Simple binding is allowed by default. To prevent SLAPD from accepting simple bind operations, you can use the `require SASL` directive which will require that all bind operations are SASL bind operations. Additionally, the `security` directive provides the `simple_bind=` SSF check, which can be used to require a minimum SSF for simple bind operations. This is covered in more detail in the section entitled *The* *security* *Directive*.

Later in this book we will examine several third party applications that use simple binding when connecting to the directory.

But there are times when it is desirable to have an even more secure authentication process, or when the bind-search-rebind method of simple binding is too much for the client to do. In such cases using SASL binding may be even better.

## SASL Binding

SASL provides a second method of authenticating to the OpenLDAP directory. SASL works by supplanting the simple bind method outlined above with a more robust authentication process.

### Note

The SASL standard is defined in RFC 2222 ( [http://www.rfc-editor.org/rfc/rfc2222.txt](http://www.rfc-editor.org/rfc/rfc2222.txt)).

SASL supports a number of different kinds of underlying authentication mechanisms, ranging from login/password combinations to more complex configurations like **One-Time Passwords (OTP)** and even **Kerberos** ticket-based authentication.

While SASL provides dozens of different configuration options, we will cover only one. We will configure SASL for doing **DIGEST-MD5** authentication. It is slightly more difficult to set up than some SASL mechanisms, but does not require the detailed configuration involved in **GSSAPI** or Kerberos.

Later in this chapter, we will integrate our SASL work with our SSL/TLS work, and use the **SASL EXTERNAL mechanism** for authenticating to the directory with client SSL certificates.

### Note

The Cyrus SASL documentation (at `/usr/share/doc/libsasl2` or available online at [http://asg.web.cmu.edu/sasl/](http://asg.web.cmu.edu/sasl/)) provides information on implementing other mechanisms.

In DIGEST-MD5 authentication, the user's password will be encrypted by the SASL client, sent across the network in its encrypted form only, then decrypted by the server and compared to a cleartext version of the password.

The advantage to using DIGEST-MD5 is that the password is protected when transmitted over the network. The disadvantage, however, is that the passwords must be stored on the server in cleartext.

Contrast this with the way simple bind works. In a simple bind the password itself is not encrypted when crossing the network, but the copy of the password stored in the database is stored in an encrypted format (unless you configure OpenLDAP otherwise).

Keep in mind that when SSL/TLS is used, all data transmitted over the connection is encrypted, including passwords.

Configuring SASL is more complex than configuring simple bind operations. There are two parts to configuring SASL support:

*   Configuration of Cyrus SASL
*   Configuration of OpenLDAP

### Configuring Cyrus SASL

When we installed OpenLDAP in Chapter 2, one of the packages we installed was Cyrus SASL (the library was named `libsasl2`). We will also need the SASL command-line tools, which are included in the `sasl2-bin` package:

```
 $ sudo apt-get install sasl2-bin

```

Included in this package are the `saslpasswd2` program and the SASL testing client and server applications.

Now we are ready to start configuring.

#### The SASL Configuration File

The SASL library can be used by numerous applications, and each application can have its own SASL configuration file. SASL configuration files are stored in the `/usr/lib/sasl2` directory. In that directory, we will create a configuration file for OpenLDAP. The file, `slapd.conf`, looks like this:

```
# SASL Configuration
pwcheck_method: auxprop
sasldb_path: /etc/sasldb2
```

### Note

Do not confuse this `slapd.conf`, located at `/usr/lib/sasl2` with the main `slapd.conf` file at `/etc/ldap/`. These are two different files.

As usual, lines that begin with the `pound sign (#)` are comments. The second line determines how SASL will try to check passwords. For example, SASL comes with a stand-alone server, **saslauthd**, which will handle password checking. In our case though, we want to use the `auxprop` plugin, which does the password checking itself, rather than querying the `saslauthd` server.

The last line tells SASL where the password database (which stores a cleartext version of all of the passwords) is located. The standard location for this database is `/etc/sasldb2`.

#### Setting a User Password

As we get started, we will store the SASL password in the `/etc/sasldb2` database. To add a password to the database we use the `saslpasswd2` program:

```
 $ sudo saslpasswd2 -c -u example.com matt 

```

Note that we have to run the above using `sudo` because the password file is owned by root. Both `sudo` and `saslpasswd2` will prompt you to enter a password.

The `-c` argument for `saslpasswd2` indicates that we want the user ID to be created if it does not already exist. `-u example.com` sets the **SASL realm**. SASL uses realms as a way to partition the authentication name space. Client applications typically provide SASL with three pieces of information: the username, the password, and the realm. By default, clients will send their domain name as the realm.

Using realms, it is possible to give the same user name different passwords for different applications or application contexts. For example, `matt` in realm `example.com` can have one password, while `matt` in realm `testing.example.com` can have a different password.

For our purposes we need only one realm, and we will name it `example.com`. When the given command is run it will prompt for a password for user `matt`, and then prompt for a password confirmation. If the passwords match, it will store the password in clear text in the SASL password database.

Now we are ready to configure OpenLDAP.

### Configuring SLAPD for SASL Support

The OpenLDAP side of SASL configuration is done in the `slapd.conf` file for the server, and the `ldap.conf` file for the client. In this section, we will focus on the SLAPD server.

When OpenLDAP receives a SASL authentication request it receives four pieces of information from the client. The four fields of information it gets are:

*   Username: This field contains the ID that the user supplied when authenticating.
*   Realm: This field contains the SASL realm in which the user is authenticated.
*   SASL Mechanism: This field indicates which authentication system (mechanism) was used. Given our SASL configuration, this should be DIGEST-MD5.
*   Authentication Information: This field is always set to `auth` to indicate that the user needs authentication.

All of this information is compacted into one DN-like string that looks like this:

```
uid=matt,cn=example.com,cn=DIGEST-MD5,cn=auth
```

The order of the fields above is the same as the order of the bulleted list: User-name, realm, SASL mechanism, and authentication information. Note however, that the realm is not required and might not always be present. If SASL does not use any realm information, the realm field will be omitted.

Of course, we do not have any records in our LDAP with DNs like the SASL string above. So, in order to correlate the authenticated SASL user with a user in the LDAP, we need to set up some method of converting the above DN-like string into a DN that is structured like the DNs in the directory. So we want to make the given string into something like this:

```
uid=matt,ou=Users,dc=example,dc=com
```

There are two ways of doing this mapping. We can either configure a simple string replacement rule to convert the SASL information string to a DN like the last one, or we could perform a search of the directory for an entry with a `uid` that is `matt`, and then, if a match is found, use that matching entry's DN.

Each of these two methods has its advantages and disadvantages. Using string replacement is faster, but it is less flexible, and it may not be sufficient for complex directory information trees. Using string replacement it may be necessary to use several `authz-regexp` directives in a row, each one with a different regular expression and replacement string.

Searching for the user on the other hand, can be much more flexible in a directory with lots of subtrees. But it will incur the overhead of doing an additional search of the LDAP tree, and it may require tweaking ACLs to allow pre-authentication searches.

Both methods use the same directive in `slapd.conf`: the `authz-regexp` directive. Let's look at an example of each method, beginning with the string replacement method.

#### Using a Replacement String in authz-regexp

The `authz-regexp` directive takes two parameters: a regular expression for getting information out of the SASL DN-like string, and a replacement function (which is different depending on whether we do string replacement or a search).

For our regular expression we want to take the username from the SASL information and map it to the `uid` field in a DN. We don't really need any of the information in the other three SASL fields, so our regular expression is fairly simple:

```
"^uid=([^,]+).*,cn=auth$"
```

This rule starts at the beginning of the line (`^`) and looks for an entry that starts with `uid=`. The next part, `([^,]+)`, stores characters after `uid=` and before a comma (`,`) in a special variable called `$1`. The rule reads "match as many characters as possible (but at least one character) that are not commas and store them in the first variable (`$1`)."

After that, the rule (using `.*` to match anything) skips over the realm (if there is one) and the mechanism, and then looks for a match at the end of the line: `cn=auth$` (where the dollar sign (`$`) indicates a line ending).

Once the regular expression is run we should have a variable, `$1`, which contains the user's name. Now we can use that value in a replacement rule, setting the `uid` value to the value of `$1`. The entire `authz-regexp` line looks like this:

```
authz-regexp "^uid=([^,]+).*,cn=auth$"
             "uid=$1,ou=Users,dc=example,dc=com"
```

After the `authz-regexp` directive, I have inserted the regular expression we just looked at. After the regular expression comes the replacement rule, which instructs SLAPD to insert the value of `$1` in the `uid` field of this template DN.

The `authz-regexp` directive can go anywhere in the `slapd.conf` file before the first `database` directive.

Since `authz-regexp` is the only necessary directive for configuring SASL, we can now test SLAPD from the command line, without making any additional changes to `slapd.conf`:

```
$ ldapsearch -LLL -U matt@example.com -v '(uid=matt)' uid
ldap_initialize( <DEFAULT> )
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt@example.com
SASL SSF: 128
SASL installing layers
filter: (uid=matt)
requesting: uid 
dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
```

Previously, we have used the `-x` flag, combined with `-W` and `-D`, to do a simple bind with a full DN and a password.

With SASL however, we don't need the full DN. All we need is a shortened connection string. So, instead of using the `-x`, `-W`, and `-D` flags, we just use `-U matt@example.com`. The `-U` flag takes a SASL username and (optionally) a realm. The realm is appended to the username, separated by the *at* sign (`@`). So, in the given example, we are connecting with username `matt` and realm `example.com`.

Next, `ldapsearch` prompts for a password (see the highlighted line in the example). This is not our LDAP password, but our SASL password—the one in the account we created when we ran `saslpasswd2`.

To review, what is happening in the previous command is this:

*   The client is connecting to SLAPD requesting an SASL bind.
*   SLAPD uses the SASL subsystem (which checks the `/usr/lib/sasl/slapd.conf` file for settings) to tell the client how to authenticate. In this case, it tells the client to use DIGEST-MD5.
*   The client sends the authentication information to SLAPD.
*   SLAPD performs the translation specified in `authz-regexp`.
*   SLAPD then checks the client's response (using the SASL subsystem) against the information in `/etc/sasldb2`.
*   When the client authentication succeeds, OpenLDAP runs the search and returns the results to the client.

Now we are ready to look at using `authz-regexp` to search the directory with a specific filter.

#### Using a Search Filter in authz-regexp

In this case, we want to search the directory for an entry that matches the username (`uid`) received during the SASL bind. Recall that the SASL authentication information comes in a string that looks like this:

```
uid=matt,cn=example.com,cn=DIGEST-MD5,cn=auth
```

In the last case, we mapped the given directly on to a DN of the form:

```
uid=<username>,ou=users,dc=example,dc=com.
```

But what do we do if we don't know, for example, if the user `matt` is in the Users OU or the System OU? A simple mapping function will not work. We need to search the directory. We will do this by changing the last argument in our `authz-regexp` directive.

Our new `authz-regexp` directive looks like this:

```
authz-regexp "^uid=([^,]+).*,cn=auth$"
             "ldap:///dc=example,dc=com??sub?(uid=$1)"
```

This regular expression is the same as the one in the previous example. But the second argument to `authz-regexp` is an LDAP URL.

### Note

For an overview of writing and using LDAP URLs see [Appendix B](apb.html "Appendix B. LDAP URLs").

This LDAP URL instructs SLAPD to search in the base `dc=example,dc=com` (using a subtree (`sub`) search) for an entry whose `uid` equals the value of `$1`, which gets replaced by the value retrieved from the regular expression in the first argument to `authz-regexp`. If the user `matt` attempts to authenticate, for example, the URL will look like this:

```
ldap:///dc=example,dc=com??sub?(uid=matt)
```

When SLAPD performs that search against our directory information tree, it will get a single record back—the record with the DN `uid=matt,ou=Users,dc=example,dc=com`.

Here's an example using `ldapsearch`. It is the same example used in the previous section, and it should have the same results even though we are using the LDAP search method:

```
$ ldapsearch -LLL -U matt@example.com -v '(uid=matt)' uid
ldap_initialize( <DEFAULT> )
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt@example.com
SASL SSF: 128
SASL installing layers
filter: (uid=matt)
requesting: uid 
dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
```

#### A Note on ACLs and Search Filters

When SLAPD reads the search filter, it performs a search of the directory. But the search is done as the Anonymous user. What this means is that we will need to make sure that the Anonymous user will need to have the requisite permissions to search the directory using the filter.

Given our last example, the Anonymous user will need to be able to search the `dc=example,dc=com` subtree for `uid` values. The ACLs that we created in Chapter 2 do not grant the Anonymous user any such permission. We will need to add one rule to our ACLs in order to allow the search to operate successfully:

```
access to attrs=uid
       by anonymous read
       by users read
```

This rule, which should appear at the top of the list of ACLs, grants read access to the `uid` attribute to `anonymous` and to any authenticated users on the system. The important part, in this example, is that Anonymous gets read access.

Keep in mind that by adding this rule, we are making it possible for unauthenticated users to see what user IDs exist in the database. Depending on the nature of your directory data, this may be a security issue. If this is a problem you can either use the string replacement method (remember, you can use several `authz-regexp` expressions in a row to handle more complex pattern matching), or you can try to reduce exposure to the `uid` field by building more restrictive ACLs

Later in this chapter, we will take a more detailed look at ACLs.

#### Failure of Mapping

In some cases the mapping done by `authz-regexp` will fail. That is, SLAPD will search the directory (using the search filter) and not find any matches. The user, however, is authenticated, and SLAPD will not fail to bind.

Instead, what will happen is that the user will bind as the SASL DN. Thus, the effective DN may be something like:

```
uid=matt,cn=example.com,cn=digest-md5,cn=auth
```

It makes no difference that there is no actual record in the directory with that username. The client will still be able to access the directory.

But this DN is also subject to ACLs, so you can write access controls targeted at users who have authenticated through SASL but who do not have a DN corresponding to a record in the directory.

#### Removing the Need to Specify the Realm

In our configuration all of the users are in the same realm, `example.com`. Rather than typing that the username and the realm be typed in every time, we can configure a default realm in `slapd.conf` by adding the following directive:

```
sasl-realm  example.com
```

If we restart the server with this new modification, we can now run an `ldapsearch` without having to specify the realm:

```
$ ldapsearch -LLL -U matt -v '(uid=matt)' uid
ldap_initialize( <DEFAULT> )
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
filter: (uid=matt)
requesting: uid 
dn: uid=matt,ou=Users,dc=example,dc=com
uid: matt
```

This time, passing `-U matt` was sufficient for authentication. SLAPD automatically inserted the default realm into the SASL information.

#### Debugging the SASL Configuration

Getting the correct SASL configuration can be frustrating. One way of improving your ability to debug is to configure logging in such a way that you can see what is going on during a SASL transaction. The `trace` debugging level (`1`) can be used to watch what is happening in SASL. You can either set the debug level in `slapd.conf` to trace (or just the digit `1`), or you can run `slapd` in the foreground on the command line:

```
$ sudo slapd -d trace
# some of the voluminous output removed...
slap_sasl_getdn: u:id converted to uid=matt,cn=DIGEST-MD5,cn=auth
>>> dnNormalize: <uid=matt,cn=DIGEST-MD5,cn=auth>
<<< dnNormalize: <uid=matt,cn=digest-md5,cn=auth>
==>slap_sasl2dn: converting SASL name uid=matt,cn=digest-md5,cn=auth 
                 to a DN
slap_authz_regexp: converting SASL name 
                   uid=matt,cn=digest-md5,cn=auth
slap_authz_regexp: converted SASL name to 
                   uid=matt,ou=Users,dc=example,dc=com
slap_parseURI: parsing uid=matt,ou=Users,dc=example,dc=com
ldap_url_parse_ext(uid=matt,ou=Users,dc=example,dc=com)
>>> dnNormalize: <uid=matt,ou=Users,dc=example,dc=com>
<<< dnNormalize: <uid=matt,ou=users,dc=example,dc=com>
<==slap_sasl2dn: Converted SASL name to 
                 uid=matt,ou=users,dc=example,dc=com
slap_sasl_getdn: dn:id converted to 
                 uid=matt,ou=users,dc=example,dc=com
```

Following this log, we can see the initial SASL string, `uid=matt,cn=DIGEST-MD5,cn=auth`, and watch as it is normalized, run through the regular expression, and converted to `uid=matt,ou=users,dc=example,dc=com`.

The `ldapwhoami` client and the `slapauth` utility are also useful when attempting to debug SASL. An example of using `ldapwhoami` to evaluate the results of `authz-regexp` is given in the next section.

## Using Client SSL/TLS Certificates to Authenticate

SASL and SSL/TLS can be used in combination to perform **SASL EXTERNAL authentication**. In SASL EXTERNAL authentication the SASL module relies upon an external source, in this case a client's X.509 certificate, as a source of identity.

Using this configuration a client with an appropriately signed certificate can bind to SLAPD without having to enter a username and password, but in a way that is still secure.

How does this work? Just as it is possible to issue a server a certificate for SSL/TLS communication, it is also possible to issue one to a user or client. We have discussed already how a certificate provides, in a secure way, identity information about a server. A client certificate can serve the same purpose.

Authentication, using SASL EXTERNAL works like this:

*   The client and server communicate with SSL/TLS protection, either using LDAPS or using StartTLS
*   When the server sends its certificate, it requests that the client also provide a certificate
*   The client sends its own certificate, which includes the following

    *   Identity information
    *   A public key
    *   The signature of a certificate authority that the server will recognize

*   The server, after verifying the certificate, passes the identity information on to SLAPD through the SASL subsystem
*   SLAPD then uses that information to bind

Since the certificate sent by the client contains all of the information needed to verify the client's identity, no login/password combination is needed.

Configuring the SASL EXTERNAL mechanism requires the following steps:

1.  Create a new client certificate
2.  Configure the client to send the certificate
3.  Configure SLAPD to correctly handle client certificates
4.  Configure SLAPD to correctly translate the identity information provided in the client certificate

### Creating a New Client Certificate

Creating a new client certificate is not significantly different from creating a server certificate. We will use the same certificate authority that we created earlier in this chapter.

First, we need to create a new certificate request:

```
$ /usr/lib/ssl/misc/CA.pl -newreq
Generating a 1024 bit RSA private key
............++++++
..++++++
unable to write 'random state'
writing new private key to 'newkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be 
    incorporated into your certificate request.
What you are about to enter is what is called a Distinguished 
    Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Illinois
Locality Name (eg, city) []:Chicago
Organization Name (eg, company) 
    [Internet Widgits Pty Ltd]:Example.Com
Organizational Unit Name (eg, section) []: 
Common Name (eg, YOUR name) []:matt
Email Address []:matt@example.com

Please enter the following 'extra' attributes
    to be sent with your certificate request
A challenge password []:
An optional company name []:
Request is in newreq.pem, private key is in newkey.pem
```

This process is just like the one before, though the fields are completed specifically for the user who is represented by this certificate. For example, if we were generating this certificate for Barbara, we would complete the **Common Name** and **Email Address** fields with her information.

### Tip

**What should go in the Common Name field?**

Earlier we used the CN field to store a domain name. What should go in an individual's CN field? One option is to use the user's full name. A more pragmatic option is to use an identifier that is used in the user's LDAP DN (such as the value of the user's `uid` attribute). This makes mapping from a certificate to an LDAP record easier.

Now, we have the new request (`newreq.pem`) and key (`newkey.pem`). The next thing to do is sign the certificate with our CA's digital signature:

```
$ /usr/lib/ssl/misc/CA.pl -signreq
Using configuration from /usr/lib/ssl/openssl.cnf
Enter pass phrase for ./demoCA/private/cakey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
    Serial Number:
      ba:49:df:f5:8e:7e:77:c6
    Validity
      Not Before: Jul  4 03:28:28 2007 GMT
      Not After : Jul  3 03:28:28 2008 GMT
    Subject:
      countryName               = US
      stateOrProvinceName       = Illinois
      localityName              = Chicago
      organizationName          = Example.Com
      commonName                = matt
      emailAddress              = matt@example.com
    X509v3 extensions:
      X509v3 Basic Constraints: 
        CA:FALSE
      Netscape Comment: 
        OpenSSL Generated Certificate
      X509v3 Subject Key Identifier: 

9A:97:8F:8C:95:1F:E0:6E:50:BD:DF:F4:C5:71:68:92:3F:A0:30:DD
      X509v3 Authority Key Identifier: 

keyid:6B:FB:66:33:5D:DB:32:40:42:D7:71:F7:F0:D0:7C:94:3E:8F:CD:58

Certificate is to be certified until 
    Jul 3 03:28:28 2008 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
unable to write 'random state'
Signed certificate is in newcert.pem
```

Now, we have the signed certificate stored in the file `newcert.pem`.

The next thing to do is to move these files to a location that will be convenient for the user. In this case, we will make a new directory in the user's home directory and move the files into that directory:

```
 $ sudo mkdir /home/mbutcher/certs
 $ sudo mv new*.pem /home/mbutcher/certs
 $ sudo chown -R mbutcher:mbutcher /home/mbutcher/certs

```

In these three lines, we make a new directory for the certs. In this case, the new `certs/` directory is in the user's home directory.

Then we move the newly-created certificate files into the new directory. We could rename these files but for now the generic name will suffice.

Finally, we need to make sure that the user has access to his or her certificates. This is done with the `chown` command.

The certificates are ready to use.

### Configuring the Client

The next thing we need to do is configure the client to use the certificate and key. This is done by creating `.ldaprc` file in the user's home directory.

### Note

A **.ldaprc file** is a "personal" version of an `ldap.conf` file. It supports all of the directives normally included in `ldap.conf`, plus a couple of special directives, like the `TLS_CERT` and `TLS_KEY` directives.

Since I am the user `mbutcher`, I will create this file in my own home directory:

```
$ cd /home/mbutcher
$ touch .ldaprc
```

Now we can edit the `.ldaprc` file. This file needs to indicate that the client is using the SASL EXTERNAL mechanism. Also, it must contain directives about the certificate and key files that we want to use. Additionally, it is not a bad idea to specify the location of the CA certificates (or even to the specific certificate for the CA that signed the server's certificate), though this is usually done at a global level with the `ldap.conf` file.

The `.ldaprc` file then, looks like this:

```
SASL_MECH EXTERNAL
TLS_CERT /home/mbutcher/certs/newcert.pem
TLS_KEY /home/mbutcher/certs/newkey.pem
TLS_CACERT /etc/ssl/certs/Example.Com-CA.pem
```

The first directive, `SASL_MECH`, indicates what SASL mechanism the client is using. In our case the client is using the `EXTERNAL` SASL mechanism.

The `TLS_CERT` directive points to the location of the client's signed X.509 certificate, and the `TLS_KEY` directive indicates the location of the client's private key file.

The `TLS_CACERT` directive points to the specific certificate used for signing the server's certificate. This will be used by the client libraries to verify the identity of the server during SSL/TLS negotiation.

At this point the client is ready. Now we need to configure SLAPD.

### Configuring the Server

SLAPD needs to do a few things in order to make the SASL EXTERNAL mechanism work:

*   It must request a certificate from the client (otherwise the client will not present one)
*   It needs to translate the identity information given in the client certificate into a DN that is meaningful in our environment

To set the server to request a client certificate is a matter of adding one directive. In the global section of the `slapd.conf` file, before any database directive is specified, the `TLSVerifyClient` directive should be added:

```
TLSCACertificateFile    /etc/ssl/certs/Example.Com-CA.pem
TLSCertificateFile      /etc/ldap/example.com.cert.pem
TLSCertificateKeyFile   /etc/ldap/example.com.key.pem
TLSVerifyClient         try

```

Only the highlighted line is new. The other lines we added earlier in the chapter.

`TLSVerifyClient` determines whether SLAPD will take steps to request and verify client certificates. There are four possible values:

*   `never`: Never request a client certificate. This is the *default*. If no certificate is requested the client will not provide one. Hence SASL EXTERNAL authentication cannot be used when the `TLSVerifyClient` is set to `never`.
*   `allow`: This will cause SLAPD to request a certificate from the client but if the client does not provide one, or if the provided one is not good (for example if the signature cannot be verified), the session will continue.
*   `try`: In this case SLAPD will request a certificate from the client. If the client does not provide a certificate the session will continue. However, if the client provides a certificate that is bad, the session will terminate.
*   `demand`: This will cause SLAPD to require a certificate from the client. If the client does not provide one, or if the provided one is not good, the session will terminate.

In the last example we set `TLSVerifyClient` to `try`. This simply means that if the client submits a certificate, it must be a valid certificate (with a known CA signature) before SLAPD will allow the connection. But it will also allow clients to connect without supplying a certificate (though such clients will not be able to use SASL EXTERNAL authentication).

If we wanted to force clients to provide a certificate then we would use the `demand` keyword instead of `try`.

At this point, we have SSL/TLS configured correctly. Now, we need to add one additional step: we need to map the identity provided by the certificate (which happens to be a DN) onto a DN for a directory user.

### Note

Translating the certificate DN into another DN is not strictly necessary. A user can bind using a certificate DN even if it is not in the directory. ACLs can be written to target such DNs too.

The DN in the client certificate we create looks like this:

```
dn:email=matt@example.com,cn=matt,o=example.com,l=chicago,\
    st=illinois,c=us
```

Note that this is one long line.

The DN contains the information we entered when running `CA.pl -newreq`. What we want to do is translate this DN into the DN of the corresponding LDAP record: `uid=matt,ou=users,dc=example,dc=com`.

How is this translation done? Using the `authz-regexp` directive that we examined earlier in the section on SASL authentication.

There are two fields in the certificate's identity string that are particularly helpful in identifying the user: `email` and `cn`. Thus, a simple regular expression can capture these two fields:

```
^email=([^,]+),cn=([^,]+).*,c=us$
```

This will assign the email address to `$1`, and the CN to `$2`.

From here we could either specify an LDAP URL with a filter for looking up DNs by email address, or we could substitute the CN for the UID field used in the LDAP DN (since the CN maps cleanly onto UID).

We will use this second method, and create `authz-regexp` that looks like this:

```
authz-regexp "^email=([^,]+),cn=([^,]+).*,c=us$"
             "uid=$2,ou=Users,dc=example,dc=com"
```

This directive maps the CN value of the certificate DN to the UID attribute in the LDAP authorization DN. Thus, when a client connects with a certificate with the DN `dn:email=matt@example.com,cn=matt,o=example.com,l=chicago,st=illinois,c=us`, SLAPD will translate that into the DN `uid=matt,ou=users,dc=example,dc=com`.

Now we are ready to test things out.

### Testing with ldapwhoami

The ideal client for testing this process is `ldapwhoami`. This will allow us to connect and bind with SASL EXTERNAL. In addition it will indicate whether or not `authz-regexp` mapped the certificate DN to our LDAP DN.

After restarting SLAPD to load the changes, we can test the server:

```
$ ldapwhoami -ZZ -H 'ldap://example.com'
Enter PEM pass phrase:
SASL/EXTERNAL authentication started
SASL username: emailAddress=matt@example.com,CN=Matt, \O=Example.Com,L=Chicago,ST=Illinois,C=US
SASL SSF: 0
dn:uid=matt,ou=users,dc=example,dc=com
Result: Success (0)
```

First, let's take a closer look at the command entered:

```
ldapwhoami -ZZ -H 'ldap://example.com'
```

The `-ZZ` flag requires that StartTLS negotiation be done successfully. Using only one `Z` will attempt StartTLS, but not close the connect if the negotiations fail. Using `-ZZ` is always a good idea when attempting authentication with the SASL EXTERNAL mechanism.

Next, the `-H 'ldap://example.com'` parameter provides the URL of the SLAPD server. Remember that for StartTLS negotiation to work, here, the domain in the LDAP URL must match the domain in the server's certificate.

What happens when this command is executed? First, the user is prompted for a pass phrase:

```
Enter PEM pass phrase:
```

This prompt is actually generated by the SSL/TLS subsystem (OpenSSL). Recall that the key that we generated is protected by a pass phrase. In order to read the key file, the OpenSSL subsystem requires the pass phrase.

But didn't I say that the SASL EXTERNAL method can obviate the need for entering a password? Yes, it can—but to do so, we would need to remove the passphrase from the key as we did when generating the server certificate:

```
openssl rsa < newkey.pem > clearkey.pem
```

Then the `TLS_KEY` directive in `.ldaprc` would need to be adjusted to point to the `clearkey.pem` file.

Removing the pass phrase may be desirable in some circumstances, and undesirable in others. Keep in mind that removing the pass phrase from the key will make it easier for the certificate to be hijacked by someone else. A key without a pass phrase should be carefully protected by permissions and other means.

Once the user's pass phrase has been entered, SASL authentication begins:

```
SASL/EXTERNAL authentication started
SASL username: emailAddress=matt@example.com,CN=Matt, \O=Example.Com,L=Chicago,ST=Illinois,C=US
SASL SSF: 0
```

As can be seen here, the SASL EXTERNAL mechanism is used, and the SASL username is set to `emailAddress=matt@example.com,CN=Matt,O=Example.Com,L=Chicago,ST=Illinois,C=US`. Finally, the SASL security strength factor is set to 0 because no SASL security mechanism has been used. Instead, the security mechanisms are *external* to SASL. Since we are using SSL/TLS with an AES-256 encyrpted certificate, the overall SSF will still be 256.

One important detail to note is that SLAPD will normalize the DN. In normalized form the DN will look like this:

```
email=matt@example.com,cn=matt,o=example.com,l=chicago,st=illinois,\
    c=us
```

The `emailAddress` attribute has been converted to `email`, and all uppercase strings have been converted to lowercase. The `authz-regexp` that we looked at above operates on this normalized version of the DN.

Finally, the last few lines of output are the results of the LDAP Who Am I? operation:

```
dn:uid=matt,ou=users,dc=example,dc=com
Result: Success (0)
```

According to SLAPD, the client is currently performing directory operations with an effective DN of `uid=matt,ou=users,dc=example,dc=com`. This means that our mapping was successful.

What would the output look like if the `authz-regexp` mapping was not successful? It would look something like this:

```
$ ldapwhoami -ZZ -H 'ldap://example.com'
Enter PEM pass phrase:
SASL/EXTERNAL authentication started
SASL username: 
emailAddress=matt@example.com,CN=Matt,O=Example.Com,L=Chicago,
    ST=Illinois,C=US
SASL SSF: 0
dn:email=matt@example.com,cn=matt,o=example.com,l=chicago,st=illinois,c=us
Result: Success (0)

```

The highlighted portion shows the result of the Who Am I? operation. The DN returned is simply the normalized form of the certificate DN—not the desired LDAP DN.

### Going Further with SASL

SASL is a flexible tool for handling authentication. Here we have looked at only two SASL mechanisms: DIGEST-MD5 and EXTERNAL. But there are many other possibilities. It can be used in conjunction with robust network authentication systems like Kerberos. It can take advantage of secure One Time Password systems, like Opiekeys. And it can be used as an interface to more standard password storage systems, like PAM (Pluggable Authentication Modules).

While such configurations are outside of the scope of this book, there are many resources available. The SASL documentation (installed locally on Ubuntu in `/usr/local/doc/libsasl/index.html`), and the OpenLDAP Administrator's Guide ([http://openldap.org](http://openldap.org)), both provide more information about different SASL configurations.

Now we will move on from authentication to authorization, and turn our attention to ACLs.

# Controlling Authorization with ACLs

We've looked at connection security and authentication. Now we are ready to look at the last aspect of security: authorization. What we are specifically interested in is controlling access to information in our directory tree. Who should be able to access a record? Under what conditions? And how much of that record should they be able to see? These are the sorts of questions that we will address in this section.

## The Basics of ACLs

The primary way that OpenLDAP controls access to directory data is through Access Control Lists (ACLs). When the SLAPD server processes a request from a client, it evaluates whether the client has permissions to access the information it has requested. To do this evaluation SLAPD sequentially evaluates each of the ACLs in the configuration files, applying the appropriate rules to the incoming request.

### Note

Previously in this chapter, we have looked at *authentication* using simple and SASL binding. ACLs provide *authorization* services, which determine what information a given DN has access to.

ACLs were introduced in Chapter 2 in the section entitled *ACLs*. This section will develop the basic examples discussed there.

An ACL is just a fancy configuration directive (the `access` directive) for SLAPD. Like certain other directives, the `access` directive can be used multiple times. There are two different places in the SLAPD configuration where ACLs can be placed. Firstly, they can be placed in the global configuration outside of a database section (that is, near the top of the configuration file). Rules that are placed at this level will apply globally to all backends. In the next chapter we will look at the case where a single directory has multiple backends.

Secondly, ACLs may be placed within a backend section (somewhere beneath a `database` directive). In this case, the ACLs will only be used when handling requests for information within database. In Chapter 2, we put our ACLs within the backend section, and we did not create any global `access` directives.

How does all of this work out in practice? When are global rules used, and when are backend-specific rules used? If a backend has no specific ACLs, then the global rules will apply. If a backend does have ACLs, then the global rules will only be applied if none of the backend-specific rules apply. If the request is for a record which is not stored in any backend, such as the Root DSE or the `cn=subschema` entry, then only the global rules will be applied.

Within their context ACLs are evaluated top-down, from the first directive in the configuration file to the last. So, when backend-specific rules are tested, SLAPD begins testing with the first rule on the list and continues sequentially until either a stopping rule matches or SLAPD reaches the end of the list.

In Chapter 2 we put the ACLs directly in the `slapd.conf` configuration file. In this section we will put them in their own file and use the `include` directive in `slapd.conf` to direct SLAPD to load the ACL file. This will allow us to separate the potentially lengthy ACLs from the rest of the configuration file.

Let's take a quick look at the format of an ACL, and then we will move on to some examples which will help clarify the intricacies of the ACL method.

An access directive looks like this:

`access to` [*resources*]

`by` [*who*] [*type* *of* *access* *granted*] [*control*]

`by` [*who*] [*type* *of* *access* *granted*] [*control*]

```
# More 'by' clauses, if necessary....

```

An `access` directive can have one `to` phrase, and any number of `by` phrases. We will take a look at the `access to` phrase first, then the `by` phrase.

## Access to [resources]

In the `access to` part, an ACL specifies what is to be restricted in the directory tree by this rule. In the given rule we used `[resources]` as a placeholder for this section. An ACL can restrict by DN, by attribute, by filter, or by a combination of these. We will first look at restricting by DN.

### Access using DN

To restrict access to a particular DN, we would use something like this:

```
access to dn="uid=matt,ou=Users,dc=example,dc=com"
       by * none
```

### Note

The `by * none` phrase simply rejects access to anyone. We will cover this and other rules when we discuss the `by` phrase later in this chapter.

The rule would restrict access to that specific DN. Any time a request is received that needs access to the DN `uid=matt,ou=Users,dc=example,dc=com`, SLAPD would evaluate this rule to determine whether that request is authorized to access this record.

Restricting access to a specific DN can be useful at times, but there are several other supported options to the DN access specifier that come in useful for more general rule-making.

It is possible to restrict access to subtrees of a DN, or even by DN patterns. For example, if we wanted to write a rule that restricted access to entries beneath the Users OU, we could use an `access` clause like this:

```
access to dn.subtree="ou=Users,dc=example,dc=com"
       by * none

```

In this example the rule restricts access to the OU and any records subordinate to it. This is accomplished by using `dn.subtree` (or the synonym `dn.sub`). In our directory information tree there are a number of user records in the Users OU subtree. These records are children of the Users OU. The DN `uid=matt,ou=Users,dc=example,dc=com`, for example, is in the subtree, and an attempt to access the record would trigger this rule.

Along with `dn.subtree`, there are three other keywords for adding structural restrictions to the DN access specifier:

*   `dn.base`: Restrict access to this particular DN. This is the default, and `dn.exact` and `dn.baselevel` are synonyms of `dn.base`.
*   `dn.one`: Restrict access to any entries immediately below this DN. `dn.onelevel` is a synonym.
*   `dn.children`: Restrict access to the children (subordinate) entries of this DN. This is similar to subtree, except that the given DN itself is not restricted by the rule.

The `dn` clause accepts one other modifier that can be used to do sophisticated pattern matching: `dn.regex`. The `dn.regex` access specifier can process POSIX extended regular expressions. Here is an example of a simple regular expression in `dn.regex`:

```
access to dn.regex="uid=[^,]+,ou=Users,dc=example,dc=com"
       by * none
```

This example would restrict access to any DN with the pattern `uid=SOMETHING,ou=Users,dc=example,dc=com`, where `SOMETHING` can be any string that is at least one character long and has no commas (`,`) in it. Regular expressions are a powerful tool for writing ACLs. We will discuss them more in the section *Getting* *More* *from* *Regular* *Expressions* after we look at the `by` phrase.

### Access using attrs

In addition to restricting access to records by DN, we can also restrict access to one or more attributes within records. This is done using the `attrs` access specifier.

In the examples we've seen, when we restricted access we were restricting access at a record level. The `attrs` restriction works at a finer-grained level: it restricts access to particular attributes within records.

For example, consider a case where we want to limit access to the `homePhone` attribute of all records in our directory information tree. This can be done with the following access phrase:

```
access to attrs=homePhone
       by * none
```

The `attrs` specifier takes a list of one or more attributes. In the given example, we just restricted access to the `homePhone` attribute. If we wanted to block access to `homePostalAddress` as well, we could modify the `attrs` list accordingly:

```
access to attrs=homePhone,homePostalAddress
       by * none
```

Let's say that we wanted to restrict access to all of the attributes in the `organizationalPerson` object class. One way of doing this would be to create one long list: `attrs`=`title`, `x121Address`, `registeredAddress`, `destinationIndicator`,.... But such a method would be time-consuming, difficult to read, and clumsy.

Instead, there is a convenient shorthand notation for this:

```
access to attrs=@organizationalPerson
       by * none

```

This notation should be used carefully, however. This code does not just restrict access to the attributes explicitly defined in `organizationalPerson`, but also all of the attributes already defined in the `person` object class. Why? Because the `organizationalPerson` object class is a subclass of `person`. Therefore, all of the attributes of `person` are attributes of `organizationalPerson`.

Sometimes it useful to restrict access to all attributes *not* required or allowed by a particular object class. For example, consider the case where the only attributes we want to restrict are those that are not specified in the `organizationalPerson` object class. We can do that by replacing the *at* sign (`@`) with an exclamation point (`!`):

```
access to attrs=!organizationalPerson
       by * none
```

This will restrict access to any attributes unless they are allowed or required by the `organizationalPerson` object class.

There are two special names that can be specified in the attributes list but that do not actually match an attribute. These two names are `entry` and `children`. So we have two cases:

*   If `attrs=entry` is specified, then the record itself is restricted.
*   If `attrs=children`, then the records that are children of this record are restricted.

These two key words are not particularly useful in cases where only an `attrs` specifier is used, but they can be much more useful when `attrs` and `dn` specifiers are used in conjunction.

Sometimes it is useful to restrict by the value of an attribute (rather than by an attribute name). For example, we may want to restrict access to any `givenName` attribute that has the value `Matt`. This sort of thing can be accomplished using the `val` (value) specifier:

```
access to attrs=givenName val="Matt"
       by * none

```

Like the `dn` specifier, the `val` specifier has `regex`, `subtree`, `base`, `one`, `exact`, and `children` styles.

### Note

When using the `val` specifier you can have no more than one attribute in the `attrs` list. The `val` specifier will not work on object class lists either.

With `val.regex` you can use regular expressions for matching. We can modify the last example to restrict access to any `givenName` that starts with the letter `M`:

```
access to attrs=givenName val.regex="M.*"
       by * none
```

In cases where the attribute value is a DN (like the `member` attribute for a `groupOfNames` object), the `regex`, `subtree`, `base`, `one`, and `children` styles can be used to restrict access based on the DN in the attribute value.

```
access to attrs=member val.children="ou=Users,dc=example,dc=com"
       by * none
```

### Tip

**Specifying an Alternate Matching Rule**

By default, the `val` comparison uses the equality matching rule. You can select a different matching rule however, by inserting a slash (`/`) after `val`, and then entering the name or OID of the matching rule:`access to attrs=givenName val/caseIgnoreMatch="matt"`.

### Access using Filters

One of the lesser used but surprisingly powerful features of the `access` phrase is support for LDAP search filters as a means of restricting access to records. We looked at the LDAP filter syntax at the beginning of Chapter 3 when we discussed the search operation. Here we will use filters to restrict access to parts of a record.

Filters provide a way to support value matching for entire records (instead of just attribute values, as is done with `attrs`). For example, using filters we can restrict access to all records that contain the object class `simpleSecurityObject`:

```
access to filter="(objectClass=simpleSecurityObject)"
       by * none
```

This will restrict access to any record in the directory information tree that has the object class `simpleSecurityObject`. Any legal LDAP filter can be used in a filter specifier. For example, we could restrict access to all records that have the given name Matt, the given name Barbara, or the surname Kant:

```
access to 
    filter="(|(|(givenName=Matt)(givenName=Barbara))(sn=Kant))"
       by * none
```

This code uses the "or" (disjunction) operator to indicate that if the request needs access to records that have given names with the values of Matt or Barbara, or if the request needs access to a record with the surname Kant, this rule should be applied.

### Combining Access Specifiers

We have looked at three different access specifiers: `dn`, `attrs`, and `filter`. And in the previous sections we have used each. Now we will combine them to create even more specific access rules.

The order of combination is as follows:

`access to` [*dn*] [*filter*] [*attrs*] [*val*]

The `dn` and `filter` specifiers come first, as they both deal with records as a whole. Then `attrs` (and `val`), which function at the attribute level, come next. Let's say that we want to restrict access to records in the Users OU just in the cases where the record has an `employeeNumber` attribute. To do this we can use a combination of a DN specifier and a filter:

```
access to dn.subtree="ou=Users,dc=example,dc=com"
    filter="(employeeNumber=*)"
       by * none
```

This ACL will only restrict access when the request is for records in the `ou=Users,dc=example,dc=com` subtree and the `employeeNumber` field exists and has some value.

In a similar fashion, we can limit access to attributes for records in a certain subtree. For example, consider the case where we want to restrict access to the `description` attribute, but only for records that are in the the System OU. We can do this by combining the DN and attribute specifiers:

```
access to dn.subtree="ou=System,dc=example,dc=com" 
    attrs=description
       by * none
```

By this rule, a client could access the record with DN `uid=authenticate,ou=System,dc=example,dc=com`, but it would not be able to access the `description` attribute of that record.

By carefully combining these access specifiers it is possible to articulate exact access restrictions. We will see some more in action as we continue on to the `by` phrase.

## By [who] [type of access granted] [control]

The `by` phrase contains three parts:

*   The **who field** indicates what entities are allowed to access the resource identified in the access phrase
*   The **access field** (type of access granted) indicates what can be done with the resource
*   The third optional part, which is usually left off, is the **control field**

To get the gist of this distinction, consider the `by` phrase that we have been working with in the previous sections: `by * none`. In this `by` phrase, the `who` field is `*` (an asterisk character), and the access field is `none`. The control field is omitted in this example.

The `*` is the universal wildcard. It matches any entity, including anonymous and all DNs. The `none` access type indicates that no permissions at all should be granted to the entity identified in the `who` specifier. In other words, `by * none` means that no access should be granted to anyone.

### Note

The directory manager (`cn=Manager,dc=example,dc=com`), specified in the `slapd.conf` file with the `rootdn` directive, is an exception. It cannot be restricted by any access control. Thus, `by * none` does not apply to the manager.

We will explore the `who` field in detail, but before getting to that, let's examine the access field.

### The Access Field

There are six distinct privileges that a client can have, in regards to an entry or attribute. There is also a seventh privilege specifier that equates to the removal of all privileges:

1.  `w`: Writes access to a record or attribute.
2.  `r`: Reads access to a record or attribute.
3.  `s`: Searches access to a record or attribute.
4.  `c`: Accesses to run a comparison operation on a record or attribute.
5.  `x`: Accesses to perform a server-side authentication operation on a record or attribute.
6.  `d`: Accesses to information about whether or not a record or attribute exists ('d' stands for 'disclose').
7.  `0`: Does not allow access to the record or attribute. This is equivalent to `-wrscxd`.

These seven privileges can be specified in a `by` clause. To set one or more of these access privileges, use the `=` (equals) sign.

For example, to allow the server to compare a record's `givenName` field to a `givenName` specified by a client, we could use the following ACL:

```
access to attrs=givenName 
       by * =c
```

This will allow any client to attempt a compare operation. But that is the only operation it will allow. By this rule, no one can read from or write to this attribute. How does this work out in practice? When we use the `ldapsearch` client to attempt to read the value of the `givenName` attribute, we do not get any information about the `givenName`:

```
$ ldapsearch -LLL -U matt "(uid=matt)" givenName
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
dn: uid=matt,ou=Users,dc=example,dc=com

```

The only thing the server returns for our query is the DN of the record that matches the filter. No `givenName` attribute is returned.

However, if we use the `ldapcompare` client, we can ask the server to tell us whether or not the DN has a `givenName` field with the value 'Matt':

```
$ ldapcompare -U matt uid=matt,ou=Users,dc=example,dc=com \ 
 "givenName: Matt"
SASL/DIGEST-MD5 authentication started
Please enter your password: 
SASL username: matt
SASL SSF: 128
SASL installing layers
TRUE

```

The `ldapcompare` client sends a DN and an attribute/value pair to the server, and asks the server to compare the supplied attribute value with the server's copy of the attribute value for the record with the given DN.

Here the `ldapcompare` client will request that the SLAPD server look up the record for `uid=matt,ou=Users,dc=example,dc=com` and check to see if the `givenName` attribute has the value 'Matt'. The server will answer `TRUE`, `FALSE`, or (if there is an error) `UNDEFINED`.

In this case, the server responded `TRUE`. This indicates that the server performed the comparison, and the values matched. The combination of the `ldapsearch` and `ldapcompare` examples should illustrate how the ACL worked: while the server-side compare operation is permitted, the client does not have access to read the attribute value.

Multiple access privileges can be granted in one `by` phrase. To modify in order to allow reading (`r`), comparing (`c`), and disclosing (`d`) on the `givenName` attribute, we can use the following ACL:

```
access to attrs=givenName 
       by * =rcd
```

Now, both the `ldapsearch` and `ldapcompare` commands that we ran should succeed.

There are cases where permissions are inherited from other ACLs (we will look at some later). In such cases, we can selectively add or remove specific permissions by using `+` (plus sign) to add and `–` (minus sign) to remove.

For example, if we know that all the users already have compare (`c`) and disclose (`d`) on all the attributes, but we want to add *read* privileges just for the `givenName` attribute, we can use the following ACL:

```
access to attrs=givenName 
       by * +r
```

### Note

An access control that grants compare and disclose, and then continues processing might look something like this: `access to attrs=givenName,sn by * =cd break`. This uses the `break` control to instruct SLAPD to continue processing ACLs. If this rule appeared in the SLAPD configuration above the rule `access to attrs=giveName by * +r`, then a request to the `givenName` attribute would have the effective permissions `=rcd`.

Likewise, if we needed to remove the compare operation just for the `givenName` attribute, we could use a `by` clause like `by * -c`.

The `0` access privilege removes all privileges. It cannot be used with the `+` or `–` operators, it can only be used with the `=` operator. The following ACL removes all privileges for all users to the `givenName` attribute:

```
access to attrs=givenName 
       by * =0
```

This is the same as the `by` clause: `by * -wrscdx`.

These access controls are good for fine-grained control, but sometimes it is nice to have shortcuts. OpenLDAP has seven shortcuts that handle common configurations of access controls:

| Keyword | Privileges |
| --- | --- |
| `none` | `0` |
| `disclose` | `d` |
| `auth` | `xd` |
| `compare` | `cxd` |
| `search` | `scxd` |
| `read` | `rscxd` |
| `write` | `wrscxd` |

The `none` keyword we have seen before and it is the same as `=0`. Looking at the other keywords and their associated privilege, a pattern emerges: each keyword adds one new privilege, to the privileges of the previous keyword. Thus, `auth` has the `=d` privilege from `disclose`, plus the `x` privilege, and `compare` has `=xd` from `auth` and adds the `c` privilege. The `write` keyword at the bottom has all privileges.

Because this general accumulation of privileges captures the usual use cases while remaining more readable, keywords are used more frequently than privilege strings. In most of our examples from here on, we will use the keyword unless there is a specific reason to use the privilege string instead.

### Note

Of the seven keywords, `disclose`, `auth`, `compare`, `search`, `read`, and `write` can be prefixed with one of two prefixes: `self` and `realself`. The `self` prefix indicates that if the value in question refers to the user's DN, then the user may have certain privileges. Thus `selfwrite` indicates that the user has `=wrscxd` permissions if and only if the value of the attribute in question is the user's DN.

The `realself` prefix is similar, but it carries the additional stipulation that the DN not be proxied. These prefixes are particularly useful when dealing with groups and other membership-based records.

For example, the following ACL allows a user `write` access to the `uniqueMember` attribute only if the `uniqueMember` attribute contains that user's DN: `access to attrs=uniqueMember by users selfwrite`.

Now that we have covered the access field we will move on to the `who` field.

### The who Field

We have always used `*` in the `who` field. However, the `who` field is the richest of the ACL fields, providing twenty-three distinct forms, most of which can be used in combinations. In order to efficiently cover ground, we will cover the major forms on their own, and then group similar forms together and treat them as units.

The five most frequently used forms are `*`, `anonymous`, `self`, `users`, and `dn`.

#### The * and anonymous Specifiers

The `*` specifier, as we have already seen, is a global match. It matches any client, including the anonymous user.

The `anonymous` specifier matches only clients that bind to the directory as the Anonymous user (see Chapter 3 for details on the Anonymous user). This refers, then, to clients that have not authenticated to the directory. Since the process of authentication requires that the client connect Anonymously, and then attempt to bind as a DN with a specific password, the anonymous user almost always needs permissions to perform an `auth` operation, in which the client sends the DN and password to the directory and asks the directory to verify that the information is correct. For that reason, you will likely need an ACL that looks like this:

```
access to attrs=userPassword
       by anonymous auth
```

This grants the Anonymous user the ability to do an auth operation. Note that every ACL ends with an implicit phrase: `by * none`. In other words, if permissions are not explicitly specified none are granted.

Note that the ACL above does not allow users to modify their own passwords. That's where the `self` specifier comes in.

#### The self Specifier

The `self` specifier is used to specify access controls for a DN on its own record. Thus, we can use the `self` specifier to allow a user to modify her or his own `userPassword` value:

```
access to attrs=userPassword
       by anonymous auth
       by self write
```

If we log in as `uid=matt,ou=Users,dc=example,dc=com` and try to modify the `userPassword` value of our own record (`dn: uid=matt,ou=Users,dc=example,dc=com`), SLAPD will allow us to change the password. But it will not (according to the rule above) allow us to modify anyone else's `userPassword` value.

### Note

The `self` specifier can be further modified with a `level` style. The `level` style indicates whether (and how many) parent records or child records are to be treated as if they were part of `self`. The `level` style takes an integer index. Positive integers refer to parents, while negative integers refer to children.

Thus `access to` `ou` `by` `self.level{1}` `write` indicates that the current DN has write permissions to the `ou` of its parent. Likewise, `access` `to` `ou` `by` `self.level{-1}` `write` indicates that the current DN has write permission to the `ou` of any of its immediate children.

#### The users Specifier

The `users` specifier refers to any authenticated client. The anonymous user is not included in `users` because it represents a client that has not authenticated.

This specifier comes in very handy when you need to allow anyone who has authenticated access to some resources. For example, in an enterprise directory we would likely want to allow all users the ability to see each other's names, telephone numbers, and email addresses:

```
access to attrs=sn,givenName,displayName,telephoneNumber,mail
       by self write
       by users read
```

#### The dn Specifier

The `dn` specifier performs similarly in the `by` phrase to the role it plays in the `access` `to` phrase. It specifies one or more DNs. The `dn` has the `regex`, `base`, `one`, `subtree`, and `children` modifiers, all of which perform the same way here as they did in the `access` `to` phrase. Here's an example using a few different DN patterns:

```
access to dn.subtree="ou=System,dc=example,dc=com" attrs=description
       by dn="uid=barbara,ou=Users,dc=example,dc=com" write
       by dn.children="ou=System,dc=example,dc=com" read
       by dn.regex="uid=[^,]+,ou=Users,dc=example,dc=com" read
```

This rule restricts access to the description attributes of anything in the System OU subtree. The user `uid=barbara,ou=Users,dc=example,dc=com` has write permissions to the description, while any child users of the System OU have *read* permissions. Users with DNs of the form `uid=SOMETHING,ou=Users,dc=example,dc=com` also have *read* access to the description.

In addition to the regular DN modifiers, a `dn` in the `by` clause can also have a `level` modifier. Level allows the ACL author to specify exactly how many levels down a `by` phrase should go. Recall that the `dn.one` specifier indicates that any record directly below the specified DN is to be granted the specified permissions. For example `by` `dn.one="ou=Users,dc=example,dc=com"` `read` grants any direct descendant of the Users OU read permissions. So `uid=matt,ou=Users,dc=example,dc=com` would be granted read access, but `uid=jake,ou=Temp,ou=Users,dc=example,dc=com` would not be granted such access because he is two levels down. The `dn.level` specifier lets us arbitrarily specify how many levels to descend. For example, `by` `dn.level{2}="ou=Users,dc=example,dc=com"` `read` would allow both `matt` and `jake` read access.

### Note

**Proxy Authentication and Real DNs**

If SLAPD is set up to allow Proxy Authentication, in which case one DN is used for authentication, and then another DN is used for performing other directory operations, it is sometimes useful to write ACLs based on the DN used for authentication (the real DN). The `realdn` specifier can be used for this. It functions just like the `dn` specifier, except that it operates on the real DN. Also, `realanonymous`, `realusers`, `realdnattr`, and `realself` can be used to restrict based on the real DN. See the `slapd.access` man page for more: `man` `slapd.access`.

#### Groups and Members

Sometimes it is useful to grant group members the access to an object. For example, if you have an Administrators group, you may wish to grant any member of that group write access to all of the records in the System OU.

One might expect that the way to set permissions for group members is simply to use the group as the value of a `dn` specifier in an ACL. However, that is not the case since the `dn` specifier refers to the group record as a whole, and has nothing at all to do with the members of the group, each of which has its own record elsewhere in the directory.

Instead, what we need is a way to search the member attributes of a particular group record, and then grant access to the DNs listed in the record. The group specifier provides exactly this sort of capability.

Group evaluation can be done with the `group` specifier. In its simplest form it is used like this:

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by group="cn=Admins,ou=Groups,dc=example,dc=com" write
       by users read
```

This ACL will grant members of the `cn=Admins,ou=Groups,dc=example,dc=com` group write access to anything in the System OU, while giving all other users read-only permissions.

### Tip

**Order Matters**

ACL by phrase are evaluated sequentially, and by default SLAPD will stop processing `by` phrases when it hits a match. In other words, if the by phrases in the above rule were reversed, members of LDAP Admins would never be given write permission because they would always match the `by` `users` `read` phrase. Evaluation of the ACL would stop before group membership was checked.

But the ACL above will only work on groups whose object class is `groupOfNames`, and whose membership attribute is `member`. This is because groupOfNames is the default grouping object class, and member is the default membership attribute.

When we created our LDAP Admins group in Chapter 3, it was not `groupOfNames`, nor did it use the `member` attribute for membership. Our record looked like this:

```
dn: cn=LDAP Admins,ou=Groups,dc=example,dc=com
cn: LDAP Admins
ou: Groups
description: Users who are LDAP administrators
uniqueMember: uid=barbara,ou=Users,dc=example,dc=com
uniqueMember: uid=matt,ou=Users,dc=example,dc=com
objectClass: groupOfUniqueNames
```

We used the `groupOfUniqueNames` object class and the `uniqueMember` membership attribute. In order to get the ACL to match these constraints we will need to specify the object class and membership attribute in the `group` specifier:

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by group/groupOfUniqueNames/uniqueMember=
 "cn=LDAP Admins,ou=Groups,dc=example,dc=com" write
       by users read
```

Note the change in the highlighted line. Using slashes (`/`) we have specified first the object class then the membership attribute that should be used to determine who what entries represent members. When this `by` phrase is evaluated, SLAPD will find the DN `cn=LDAP` `Admins,ou=Groups,dc=example,dc=com`, check to see if it has object class `groupOfUniqueMembers`, and then grant write permissions to a DN if it is specified in a uniqueMember attribute.

Using this expanded notation, you can use other membership-based records as groups. For example, you can use the `organizationalRole` object class with the `roleOccupant` membership attribute.

Like many other specifiers, the group specifier also supports regular expressions with the `regex` style. Thus, we could create a rule that would allow members of any group in OU Groups write access to the System OU by expanding our last example:

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by group/groupOfUniqueNames/uniqueMember.regex=
 "cn=[^,]+,ou=Groups,dc=example,dc=com" write
       by users read
```

The second and third lines should be combined into one long line in `slapd.conf`. The regular expression in the group specifier would match any DN with a CN component at the beginning. For all such entries, if the object class is `groupOfUniqueMembers`, then the SLAPD will grant membership to a user who is a `uniqueMember` of one of those groups.

#### Member-Based Record Access

What if a group member needs to modify the record of the group to whom she or he belongs? One way to allow this is with the `dnattr` specifier. The `dnattr` specifier grants access to a record only if the client's DN appears in a certain attribute of the record. For example, the following example allows a group member (`uniqueMember`) of a group (which is a `groupOfUniqueNames` object) access to the group record:

```
access to dn.exact="cn=LDAP Admins,ou=Groups,dc=example,dc=com"
       by dnattr=uniqueMember write
       by users read
```

The second line specifies that if the client's DN is in the list of values for the `uniqueMember` attribute, then that client should be given write access to the entire group record. Other users, according to the third line, will have read access.

#### Network, Connections, and Security

SLAPD can use information about the client's connection (including network and security information) in access control lists. This feature provides an additional layer of network security that complements SSL/TLS and SASL.

The following are network or connection level specifiers:

*   `peername`: This is used to specify a range of IP addresses (for `ldap://` and `ldaps://`).
*   `sockname`: This is used to specify a socket file for an LDAPI listener (`ldapi://`).
*   `domain`: This is used to specify a domain name for `ldap://` and `ldaps://` listeners.
*   `sockurl`: This is used to specify a socket file in URL format (`ldapi://var/run/ldapi`) for an LDAPI listener.
*   `ssf`: The overall security strength factor (SSF) of the connection.
*   `transport_ssf`: The SSF for the underlying transport layer of the network.
*   `tls_ssf`: The SSF for the SSL/TLS connection. This works with SSL/TLS connections on LDAPS listeners and Start TLS on LDAP listeners.
*   `sasl_ssf`: The SSF of the SASL connection.

The SSF specifiers (`ssf`, `transport_ssf`, `tls_ssf`, and `sasl_ssf`) perform the same checks as the SSF parameters to the SLAPD `security` directive (discussed in the first part of this chapter). In this case, however, SSFs may be used to selectively restrict (or grant) access to portions of the directory information tree. SSF specifiers require an integer value for the level of security desired. For example, using `ssf=256` will require that the overall SSF of a connection be 256\. But `tls_ssf=56` will require that the SSF of the TLS/SSL layer be at least 56, regardless of what the SSF of the SASL configuration is. For more information on SSFs, see the section earlier in this chapter entitled *Using* *Security* *Strength* *Factors*.

For example, the following ACL will only grant *write* access to the specified DN when the client has connected with a strong SASL cipher:

```
access to dn.subtree="ou=users,dc=example,dc=com"
 by self sasl_ssf=128 write
       by users read
```

This rule allows users to modify their own records only if they have authenticated with SASL using a security mechanism with a strength of 128 (DIGEST-MD5) or more. All other users would only get read access.

### Tip

**Combining Specifiers in a by Phrase**

As the rule above illustrates, multiple specifiers can be used in a single by phrase. When this happens all specifiers must be matched before the indicated rights will be granted (or denied).

The `peername` specifier is used for setting restrictions based on information about the IP connection. It can be used to complement other components in network security, like SSL/TLS. The `peername` specifier can take an IP address or a range of IP addresses (using subnet masks) and can also specify a source port.

The following rule grants write access to local connections, read access to connections on the local LAN (address from 10.40.0.0 through 10.40.0.255), and denies access to all other clients. Remember, every rule ends with an implicit `by` `*` `none`.

```
access to *
       by peername.ip=127.0.0.1 write
       by peername.ip=10.40.0.0%255.255.255.0 read
```

Note that the `peername` specifier requires the ip style for specifying an IP address. It also supports the `regex` style (`access` `to` `*` `by` `peername.regex="^IP=10\`.`40\`.`0\`.`[0-9]+:[0-9]+$"` `write`) and the `path` specifier to replicate the behavior of `sockname`.

### Tip

**Regular Expressions for IP Addresses**

For an IP address, the format of the string used in regular expression evaluation is this: `IP=<address>:<port>`. If you are creating a precise regular expression make sure to deal with the `IP=` prefix and the port information. A regular expression like this will fail: `peername.regex="^10.40.12[0-9]$"`. Why? Because it is missing the `IP=` and port information.

A more useful version of the rule above would deny access to anything in the directory if it was not in the particular ranges, but would leave further access controls to rules appearing later in the ACL list. This can be done using the special `break` control described in the next section. We could also added SSF information, so connections coming over non-local connections must also use strong SSL/TLS encryption. Here is the rule:

```
access to * 
       by peername.ip=127.0.0.1 break
       by peername.ip=10.40.0.0%255.255.255.0 tls_ssf=128 break
```

The above rule might appear difficult to read, but here is what it does:

*   If the connection is local (coming over 127.0.0.1 or `localhost`), then SLAPD allows further processing of the ACL list (that's what `break` does). Whether or not the user then gets access to resources is dependent on other rules.
*   If the connection comes from an address on the LAN and it is using strong SSL/TLS encryption, then SLAPD will continue processing the ACL list.
*   Under any other connecting circumstances the connection is rejected. For example, if a connection comes from the LAN but does not use sufficiently strong SSL/TLS, the connection will be closed. This behavior is caused by the implicit `by` `*` `none` phrase.

For more on the `break` control, see the section called *The* *Control* *Field*.

Sometimes it is more useful to be able to specify which domain names (rather than which IP addresses) should be granted access. This can be done with the `domain` specifier:

```
access to * 
       by domain.exact="main.example.com" write
       by domain.sub="example.com" read
```

In the example above, the second line provides write access to any client connection coming from the domain name `main.example.com`. The third line grants read access to the domain `example.com`, and any subdomain of `example.com`. So, if a server with the domain name `test2.example.com` made a request, it would be granted access under the third rule. However, `testexample.com` would not match because it is not a subdomain of `example.com`—it is a different domain altogether.

When SLAPD encounters a domain specifier in an ACL, it takes the IP address of the client connection and does a reverse DNS lookup to get the host name. In light of this there are two things to keep in mind when using the domain specifier.

First, the name returned by a reverse DNS lookup may not be what you expect based on a forward DNS lookup. For example, doing a DNS lookup on `ldap.example.com` returns the address 10.40.0.23\. However, doing a reverse DNS lookup on 10.40.0.23 returns `mercury.example.com`. Why?

It is because `ldap.example.com` is in DNS parlance, a **CNAME record**, and `mercury.example.com` is an **A record**. Practically speaking, what this means is that `ldap.example.com` is an alias to the server's real (**canonical**) name, which is `mercury.example.com`. The practical consequence is this: when you write an ACL using the `domain` specifier, make sure you use the A record domain name, not the CNAME record name. Otherwise, SLAPD will apply the rule to the wrong domain name.

### Tip

**Looking up DNS Information**

There are many tools for looking up DNS information. Most Linux distributions, including Ubuntu Linux, provide the `host` and `dig` commands for command-line DNS lookups. The `host` command gives brief sentence-like information like this: `ldap.example.com` `is` `an` `alias` `for` `mercury.example.com`. The `dig` command, in contrast, gives detailed technical information.

The second thing to keep in mind when considering the domain specifier is that it is less reliable than using IP address information. DNS addresses can be spoofed, which means another server on the network can claim to be `ldap.example.com` and send traffic that looks, to SLAPD, like it is coming from the real `ldap.example.com`.

One way to diminish the risk of this is to use client-side SSL/TLS certificates and configure SLAPD to require that the client send a signed certificate to authenticate before it can perform any other directory operations. Unfortunately, client-side certificates cannot be selectively required through ACLs. Instead you will have to use the directive `TLSVerifyClient` `demand` in the `slapd.conf` file.

The `sockname` and `sockurl` specifiers are used for servers that run with UNIX local socket Inter Process Communication (IPC) instead of network sockets. These directives can be used to restrict local connections that use the IPC layer instead of connecting through the IP network.

### Note

It is uncommon to run LDAPI. Generally it is used only in situations where IP network connections cannot or should not be used. In typical cases, local clients connect to SLAPD over LDAP, using the URL `ldap://localhost/` rather than using LDAPI.

For example, we could use the following ACL to allow only local (LDAPI) connections to write to the record, while users who connected through a different mechanism could only read the record:

```
access to dn.exact="uid=matt,ou=Users,dc=example,dc=com"	
       by sockurl="ldapi://var/run/ldapi" write
       by users read
```

The second line indicates that only LDAPI connections that connect through a particular LDAPI socket file should gain write access to the DN. All other clients (`users`) will get read permissions.

#### Advanced Step: Using the set Specifier

In addition to the syntax we have examined just now, there is an experimental type of `by` phrase—the **set** syntax. The `set` syntax can be used to create a compact and powerful set of conditions for access. Since it allows Boolean operators, and has a method for accessing attribute values, a single rule in the `set` syntax can accomplish what would otherwise take tremendously complex ACLs.

The basic idea behind the `set` syntax is this. By using a rule composed of conditions joined by operators, SLAPD creates a set of objects which have access to the record in question. If the result of an evaluation of a `set` specifier is a set that contains one or more members, then the `by` phrase is considered a match and permissions are applied. If, on the other hand, the set is empty, then SLAPD will continue evaluating the `by` phrases for that rule to see if it can find another match.

### Tip

The `set` specifier uses operations of the sort used in set theory. When using the set specifier you may find it helpful to think in terms of set theory, with sets (lists of items) and set operations, such as union (`&`) and intersection (`|`).

Here is a simple ACL using a `set` specifier to replicate the behavior of the `group` specifier. It provides write access to records in the System OU only to clients in the LDAP Admins group. All others get read access only:

```
access to dn.subtree="ou=System,dc=example,dc=com"
 by set="[cn=ldap admins,ou=groups,dc=example,dc=com]/
 uniqueMember & user" write
       by users none
```

The second line, highlighted above, contains the `set` specifier, which contains a `set` statement. The text in the square brackets specifies a DN, which is the DN of the LDAP Admins group. To access the values of the `uniqueMember` attribute we append `/uniqueMember` to the DN. When SLAPD expands this, it will contain the set of all `uniqueMembers` in the LDAP `Admins` `group`. In set-theoretic notation (which is not used by OpenLDAP, but which is helpful to understand what is happening), the set of group members would look like this:

```
{ uid=matt,ou=users,dc=example,dc=com ; 
    uid=barbara,ou=users,dc=example,dc=com }
```

There are two members (the two `uniqueMembers`) for the LDAP Admins group.

The `&` (ampersand) operator performs a union operation on two sets. The **user keyword** expands to the set that contains one member: the DN of the current client. So, if I perform a search, binding as `uid=matt,ou=users,dc=example,dc=com`, then the user set will contain one record:

```
{ uid=matt,ou=users,dc=example,dc=com }
```

When the `&` operator is applied, it will generate the intersection of the two sets. That is, the resulting set will contain only members that are in both of the original sets. Since only the record for UID `matt` is in both, the resulting set will contain just the DN for `matt`:

```
{ uid=matt,ou=users,dc=example,dc=com }
```

The resulting set is not empty so it is considered a match. The result of the set evaluation, then, is that the `uid=matt,ou=users,dc=example,dc=com` will be granted access based on the `set` specifier.

### Note

Sets are case-sensitive, and always use the normalized DN form. What this means is that the DNs in sets should always be lowercase.

Consider a case though, when the user is not a member of the LDAP Admins group. If `uid=david,ou=users,dc=example,dc=com` binds, can he perform read and write operations? When the set specifier is run, the first of the two sets (group membership) will evaluate to the same thing it did above:

```
{ uid=matt,ou=users,dc=example,dc=com ; 
    uid=barbara,ou=users,dc=example,dc=com }
```

But the user keyword will expand to this:

```
{ uid=david,ou=users,dc=example,dc=com }
```

There are no items in the intersection of these two sets, so the resulting set, after the `&` operator is applied, is an empty set:

```
{ }
```

There are no matches, so this `by` phrase fails to apply. The last line in our ACL (`by` `users` `none`) will then apply, and the `uid=david` will be given no access permissions.

Let's look at another example. We will use the set specifier to implement a rule where, when a client DN tries to access a record DN, it is given write access only if the two DNs are the same, or else it is given read access if they are in the same OU. Otherwise, the client DN is denied access to the record DN. Here's the ACL:

```
access to dn.subtree="dc=example,dc=com"
       by set="this & user" write
       by set="this/ou & user/ou" read
```

The first line indicates that this rule will apply to the record `dc=example,dc=com` and everything under it.

The second line takes the intersection of the sets generated by two keywords: `this` and `user`. The `this` keyword expands to the set containing the DN of the requested record. The `user` keyword, as we saw, expands to the DN of the client.

So, if the client `uid=david,ou=users,dc=example,dc=com` requests access to its own record, the resulting set operation will be as follows:

```
{ uid=david,ou=users,dc=exampls,dc=com } & 
    { uid=david,ou=users,dc=example,dc=com }
```

Since both sets contain the same member, the resulting set (the intersection of the two) is `{` `uid=david,ou=users,dc=example,dc=com` `}`. The end set is not empty, so the user will be granted write access.

Now let's look at the third line of the given ACL. This rule will return a non-empty set whenever the requested DN and the client DN both have the same value for the `ou` attribute. If `uid=david,ou=users,dc=example,dc=com` requests the record for `uid=matt,ou=users,dc=example,dc=com`, then SLAPD will check the values of their respective OU attributes.

The set identified by `this/ou` will be expanded to contain the values of all of the OU attributes in the requested record (the record for `uid=matt,ou=users,dc=example,dc=com`). This set is:

```
{ 'Users' }
```

Note that in this case the value is not a DN, but a string. Sets can perform matching operations on strings as well as DNs.

The set identified by `user/ou` will be expanded to contain the values of all of the OU attributes in the client's record. The record for `uid=david,ou=users,dc=example,dc=com` contains one value for the `ou` attribute, and the resulting set will contain that one attribute value:

```
{ 'Users' }

```

SLAPD will compute the intersection of `{` `'Users'` `}` `&` `{` `'Users'` }, which is `{` `'Users'` `}`. Since the set is not empty, `uid=david,ou=users,dc=example,dc=com` will be granted access to read the record of `uid=matt,ou=users,dc=example,dc=com`.

The `set` specifier provides one way of granting access to a record *only* in the case that a record contains a certain attribute. If we only want to grant write access to records with the title attribute, we can use the following rule:

```
access to dn.child="ou=Users,dc=example,dc=com"
       by set="this/title" write
```

In this ACL, if the requested record has a single `title` attribute, then the result of the evaluation of the above rule will be a set containing one element. However, if the record attribute has no title attribute, then the resulting set will be empty, and write access will not be granted.

In our directory the record of `uid=matt,ou=users,dc=example,dc=com` has the following title attribute:

```
title: Systems Integrator
```

But the record `uid=barbara,ou=users,dc=example,dc=com` does not have a title attribute at all. So if the record for `uid=matt` was requested, then the resulting set, based on the above ACL, would be:

```
{ 'Systems Integrator' }
```

So if an authenticated user attempted to access the record for `uid=matt`, SLAPD would grant access. In contrast, the set for `uid=barbara` would be `{}`, the empty set. So a user trying to access the record having `uid=barbara` would be denied access.

Using a similar set specifier, we could grant access to a record depending not only on the existence of an attribute, but on its value too:

```
access to dn.child="ou=Users,dc=example,dc=com"
       by set="this/objectclass & [person]" write
```

According to the above rule, write access will be granted for anything in the Users OU only if the entry has an `objectclass` attribute with the value `person`. Note that in this case the square brackets are used to define a string literal.

If a client were to access the record `uid=barbara,ou=users,dc=example,dc=com`, the first part of our `set` statement would evaluate to the following set:

```
{ 'person' ; 'organizationalPerson' ; 'inetOrgPerson' }
```

Those are the three object classes for the `uid=barbara` record. The other part, `[person]`, would be expanded to this set:

```
{ 'person' }
```

When the union is computed, the result would be the set `{'person'}` and so write access would be granted.

These are just a few of the basic operations that can be done with the `set` specifier. Unfortunately, `set` is not documented in the `slapd.access` man page. However, there is a lengthy and informative article on using set in the OpenLDAP official FAQ-O-Matic: [http://www.openldap.org/faq/data/cache/1133.html](http://www.openldap.org/faq/data/cache/1133.html).

### The control Field

The last field in the `by` phrase is the control field. There are only three possible values for the control field: `stop`, `break`, and `continue`. If no control field is specified, `stop` is assumed. For example, `by` `*` `none` is the same as `by` `*` `none` `stop`.

The first value, `stop`, indicates that if that particular by clause matches, no further checking of ACLs for matching should occur. Consider the following (admittedly contrived) case:

```
access to attr=employeeNumber, employeeType, departmentNumber
       by users=cd
       by dn="uid=matt,ou=Users,dc=example,dc=com" +r

access to attr=employeeNumber
       by users +w
```

If I bind as `uid=matt,ou=Users,dc=example,dc=com` and try to modify my `employeeNumber`, will I be allowed to? No, I will not.

The reason I will not be able to modify the record is because I will only have the permissions granted by the first `by` phrase: `by` `users` `=cd` (remember, `by` `users` `=cd` is the same as `by` `users=cd` `stop`). As soon as SLAPD sees that I match the first `by` phrase of the first ACL, it will stop testing ACLs. Thus it will never reach the rule that grants my DN `+r` access, nor will it reach the rule that grants all users `+w` to the `employeeNumber` attribute.

This is an example of the `stop` control, which is used implicitly by all three rules.

Now, if I wanted to make sure that after the first `by` phrase SLAPD continues to evaluate phrases within the ACL, I could re-write the ACLs using the `continue` control:

```
access to attr=employeeNumber, employeeType, departmentNumber
       by users-=cd continue
       by dn="uid=matt,ou=Users,dc=example,dc=com" +r

access to attr=employeeNumber
       by users +w
```

After running the same test on these rules, the DN `uid=matt,ou=Users,dc=example,dc=com` would have the permissions `=cdr`.

The `continue` control instructs SLAPD to continue processing all of the `by` phrases in the *current* *ACL*. Once it is done evaluating that ACL though, it will not continue to look for matches in other ACLs.

In order to tell SLAPD to look at different rules for matches, we would have to use the `break` control. When SLAPD encounters an applicable clause that ends with a `break` control, it stops processing the current ACL but continues looking at other ACLs to see if they apply.

Thus, to get write permissions with our ACL we would want the following ACLs:

```
access to attr=employeeNumber, employeeType, departmentNumber
       by users=cd continue
       by dn="uid=matt,ou=Users,dc=example,dc=com" +r break

access to attr=employeeNumber
       by users +w stop
```

Now what will happen when the user with UID `matt` attempts accesses an `employeeNumber`?

First, the `by` phrase of the first ACL will be evaluated, and `matt` will be granted `=cd`. Because of the `continue` control, SLAPD will then examine the second `by` clause, which will also match for the user `matt`. Thus, `matt` will have `=rcd` when the processing of the first ACL completes.

Due to the `break` control the second ACL will also be evaluated, and `matt` will be granted `+w` as well, bringing his final permissions up to `=wrcd`.

Using the `continue` and `break` controls is one way to incrementally handle permissions. In complex configurations, judicious use of `continue` and `break` can make maintaining ACLs much easier, and can reduce the total number of ACLs.

## Getting More from Regular Expressions

In the previous sections we have looked at using regular expressions in both the `access` `to` phrase and the `by` phrase. But we can use both in conjunction. We can store information about the matches identified in the `access` `to` phrase, and use that information later in the `by` phrases.

To temporarily store matching information in an `access` `to` phrase we can surround the regular expression with parentheses. Here's an example:

```
access to dn.regex="ou=([^,]+),dc=example,dc=com"
       by dn.children,expand="ou=$1,dc=example,dc=com" read
```

This ACL grants a client the DN access to read a record DN only if both the client DN and the record DN are in the same part of the directory tree (that is, if both are in the same OU).

In the first line of the given ACL we used parentheses to capture the match from the regular expression `[^,]+`, which will be the value of the `ou=` component of the DN. Again, `[^,]+` says "match all charcters that are not '`,`'."

In the second line we used the `dn.children` specifier but supplemented it with an extra keyword: `expand`. The `expand` keyword tells SLAPD to substitute matches from the `access` `to` clause into this phrase.

Because of the `expand` keyword, the variable `$1` is substituted with the value of the match in the first line. Everything captured between '`(`' and '`)`' in the regular expression will be stored in `$1`.

Variable names are assigned in order. The first set of parenthesis in the regular `access` `to` phrase gets stored in `$1`. If a second set of parenthesis existed, the matching information inside of those would be stored in `$2` and so on for each additional set of parenthesis.

For example, we might want an ACL like this:

```
access to dn.regex="uid=([^,]+),ou=([^,]+),dc=example,dc=com"
       by dn.children,expand="uid=$1,ou=$2,dc=example,dc=com" write
```

This rule would grant a client DN access to read and write any entries subordinate to its own record but deny other uses the ability to even read those entries.

### Note

Address books are sometimes implemented in OpenLDAP by storing a user's addresses as subordinate entries to the user's own entry in the directory. There is an example of this in the OpenLDAP FAQ-O-Matic: [http://www.openldap.org/faq/data/cache/1005.html](http://www.openldap.org/faq/data/cache/1005.html)

Notice that the first line stores two variables. The UID goes in `$1` and the OU goes in `$2`. These are expanded in the second line.

It is also possible to use matches from the `access` `to` phrase in regular expressions in the `by` phrase:

```
access to dn.regex="uid=[^,]+,ou=([^,]+),dc=example,dc=com"
       by dn.regex="uid=[^,]+,ou=$1,dc=example,dc=com" write
```

In the first line only the results of the second regular expression are captured and stored in a variable. The second line also contains a regular expression, and it makes use of the `$1` variable to retrieve the value of the OU from the first line. Note that `dn.children,expand` was replaced with `dn.regex`. The `expand` keyword need not be added for regular expressions.

The rule grants write access to a client DN for any user record that is in the same OU of that directory tree.

We have looked at some simple, though useful, regular expressions in these ACLs. But much more complex regular expressions can be composed, making ACLs even more powerful. As you compose more advanced regular expressions you may find some other sources of information helpful. Along with the `slapd.access` man page, the POSIX extended regular expressions man page (`man` `regex`) may turn out to be useful as well.

## Debugging ACLs

Debugging ACLs can be frustrating. They are complex, security sensitive, and require detailed testing. But there are three tools that make the debugging and testing process easier.

The first is just the `ldapsearch` command-line client. It can be used to carefully craft filters designed to test the processing of ACLs. The `ldapcompare` tool also comes in handy when you need to test comparison operations.

But it is also useful to make the most of LDAP's logging directives. The `trace` and `acl` debugging levels each provide detailed information about ACL processing. The `acl` level, for example, records each ACL evaluation. This can be very useful in determining what rules are run and when. We find the `trace` debugging level to be useful as well, as it provides information about how each evaluation was performed, including how regular expressions were expanded.

### Tip

**Running SLAPD in the Foreground**

Sometimes it is easier to test ACLs by running SLAPD in the foreground, instead of as a daemon process, and printing debugging and logging information to standard out. For example, we can print ACL and trace debugging out this way: `slapd` `-d` `"acl,trace"`. Note that you will want to run this command as the appropriate user (such as `openldap`). To terminate the process use the *Ctrl*-*C* keyboard combination.

Finally, the `slapacl` command line utility provides a detail-oriented tool for evaluating ACLs directly. Since it does not connect to the SLAPD server over the LDAP protocol it allows more direct testing of just the ACLs.

For example, we can check whether or not a particular SASL user, `matt`, can access the record `cn=LDAP` `Admins,ou=Groups,dc=example,dc=com` and *read* the value of the `description` attribute:

```
 $ slapacl -U matt -b "cn=LDAP Admins,ou=Groups,dc=example,dc=com" \ 
 "description/read"

```

The `-U` `matt` param specifies the SASL user name. The `-b` `"cn=LDAP` `Admins,ou=Groups,` `dc=example,dc=com"` param indicates which record we want to test against, and the last field, `"description/read"` indicates the attribute and the access level. This will simply return `ALLOWED` if the ACLs allow read access, or `DENIED` otherwise.

Likewise, we can test other LDAP operations. For example, we can test whether a user has permissions to `compare`:

```
$ slapacl -U matt -b "uid=matt,ou=Users,dc=example,dc=com" 
    "uid/compare"
authcDN: "uid=matt,ou=users,dc=example,dc=com"
compare access to uid: ALLOWED
```

In this example we have included the response. The first response line indicates how the SASL DN was resolved, and the second line indicates that compare access on `uid` was allowed.

The `slapacl` program essentially runs its own SLAPD and as such, it can be set to print complete processing logs to the screen. For example, to turn on trace debugging we can just add the `-d` `trace` param to the given command:

```
$ slapacl -U matt -b "uid=matt,ou=Users,dc=example,dc=com" -d trace 
    "uid/compare"
slapacl init: initiated tool.
slap_sasl_init: initialized!
hdb_back_initialize: initialize HDB backend
hdb_back_initialize: Sleepycat Software: Berkeley DB 4.3.29: 
    (September  6, 2005)
bdb_db_init: Initializing HDB database
>>> dnPrettyNormal: <dc=example,dc=com>
# LOTS of lines deleted...
<<< dnPrettyNormal: <uid=matt,ou=Users,dc=example,dc=com>, 
    <uid=matt,ou=users,dc=example,dc=com>
entry_decode: ""
<= entry_decode()
compare access to uid: ALLOWED
slapacl shutdown: initiated
====> bdb_cache_release_all
slapacl destroy: freeing system resources.
```

As you can see`slapacl` provides detailed evaluation information in this case.

Using the LDAP command-line clients, detailed logging, and the `slapacl` command, debugging and testing ACLs can be done effectively.

## A Practical Example

In this part of the chapter, we have taken a low-level look at ACLs in OpenLDAP. We have covered many of the details of the ACL system. Now it is time to implement what we have covered so far to create a generic set of ACLs for our directory information tree.

In Chapter 2 we created a bare-bones set of ACLs in our `slapd.conf` file. Here's what we created then:

```
########
# ACLs #
########
access to attrs=userPassword
       by anonymous auth
       by self write
       by * none

access to *
       by self write
       by * none
```

Now, we will create a new, more practical list of ACLs.

The first thing we will do is move the ACLs out of `slapd.conf` and into a separate file: `acl.conf`. This will keep the lengthy list of ACLs separate from the rest of our configuration. To do this we will replace the ACLs above with an `include` directive:

```
########
# ACLs #
########
include /etc/ldap/acl.conf
```

When SLAPD is started it will include the contents of `/etc/ldap/acl.conf` at the location where the `include` statement appears. Recall that ACLs are backend-specific. Each different database can have its own ACLs (and multiple databases can be defined in the same `slapd.conf` file). So it is important to put the `include` directive after the database `directive` in `slapd.conf`.

Now we will begin editing the `acl.conf` file. The rules that we will write will be simple, and designed for a directory where most of the directory users are allowed to view most of the information in the directory. A higher-security directory may have a far more complex list of ACLs.

Since ACLs are evaluated in order from top to bottom we want to carefully craft our rules so that important restrictions are implemented right away.

If there are network-based access rules they should usually appear at the top of the ACL list so that they are evaluated first. For example, if we want to restrict access to the entire database if the host is not in our LAN, we would use the following rule:

```
access to * 
       by peername.ip=127.0.0.1 none break
       by peername.ip=10.40.0.0%255.255.255.0 none break
```

By this rule only access from the localhost (127.0.0.1) and from inside of our 10.40.0.0 subnet will be allowed to access the directory. Since the `break` control is specified, later rules may modify the `none` permission, granting clients more permissions. All other connections will be closed immediately.

Next, we want to grant members of the LDAP Admins group write access to everything in the `dc=example,dc=com` tree:

```
access to dn.subtree="dc=example,dc=com"
       by group/groupOfUniqueNames/uniqueMember=
           "cn=LDAP Admins,ou=Groups,dc=example,dc=com" write
       by * none break
```

This immediately grants write access to the members of the LDAP Admins group. For all other clients though, SLAPD will continue processing.

### Note

No ACLs need to be written for the directory manager, the DN specified in the `slapd.conf` directive `rootdn`. This DN always has full access to the directory information tree, and ACLs will have no effect on this user.

Next, we want to make sure that the `userPassword` field is available to the anonymous user for authentication purposes. We also want to allow users to be able to modify their own passwords, but otherwise we want `userPassword` unavailable for reading and writing by others. Note that by the previous rule the LDAP Admins will also be able to modify passwords for users.

```
access to attrs=userPassword
       by anonymous auth
       by self write
```

In some cases, other users may need `auth` access to the password as well, in which case you may need to add `by` `users` `auth` to the given list.

We also need to grant access to the `uid` attribute if we are using the `ldap://` URL form for SASL binding in the `authz-regexp` directive. This is because the filter in the LDAP URL is run as anonymous (see the discussion in the *Configuring* *SLAPD* *for* *SASL* *Support Subsection*).

Additionally, we don't want to let users try to modify their own `uid`, since `uid` is used in the DN:

```
access to attrs=uid
        by anonymous read
        by users read
```

Now Anonymous and all authenticated users will be able to access the `uid` attribute of any record in the directory to which they have access.

There are also a few other attributes we don't want users to be able to modify—even in their own records.

We don't want users to try to modify their OU attributes, since OU attributes are also used in DNs. We also don't want them to be able to modify their `employeeNumber` or their `employeeType`:

```
access to attrs=ou,employeeNumber,employeeType by users read
```

We have a special account, `uid=Authenticate,ou=System,dc=example,dc=com`, which will be used on occasion to help with bind requests. This user should not have access to anything else other than what we specified:

```
access to * 
       by dn.exact="uid=Authenticate,ou=System,dc=example,dc=com" 
           none
       by users none break
```

Again, the last line instructs SLAPD to continue processing ACLs for users who aren't having the authentication account. This line will also stop the anonymous user from browsing the rest of the tree since the implicit rule at the end, `by` `*` `none`, will catch the anonymous user.

### Note

The `uid=Authenitcate` user was already granted, in an earlier rule, access to the `uid` attribute, which is the attribute that this account will use to search for user information needed to bind.

Let's say that we don't want regular users (DNs in the Users OU) to be able to access records in the System OU of our directory (which is typically used for system accounts). We can implement this with the following rule:

```
access to dn.subtree="ou=System,dc=example,dc=com"
       by dn.subtree="ou=Users,dc=example,dc=com" none
       by users read
```

This denies access to users in the Users OU, but allows other users (like System accounts) access to these records.

We also want to give every user the ability to read and write records below its own, but restrict others from accessing those records. This makes it possible for users to store their own information (like address books) inside of the directory:

```
access to dn.regex="^.*,uid=([^,]+),ou=Users,dc=example,dc=com$"
       by dn.exact,expand="uid=$1,ou=Users,dc=example,dc=com write
```

Finally, the last rule we want is a default rule. This rule should answer the question, "What do we want to happen when no other rules are matched?" We want users to be able to modify their own records and see the records of others:

```
access to *
       by self write
       by users read
```

Now our list of ACLs is complete. Altogether, this is what they look like:

```
#################################################
# ACLs
# These are ACLs for the first database section
# of the slapd.conf file found in this directory
#################################################
##
## Restrict by IP address:
access to *
       by peername.ip=127.0.0.1 none break
       by peername.ip=10.40.0.0%255.255.255.0 none break

## Give Admins immediate write access:
access to dn.subtree="dc=example,dc=com"
       by group/groupOfUniqueNames/uniqueMember="cn=LDAP 
           Admins,ou=Groups,dc=example,dc=com" write
       by * none break

## Grant access to passwords for auth, but allow users to change 
## their own.
access to attrs=userPassword
       by anonymous auth
       by self write

## This rule is needed by authz-regexp
## (Note: Since uid is used in DN, user cannot change its own uid.)
access to attrs=uid
       by anonymous read
       by users read
## Don't let anyone modify OUs, employee num or employee type.
access to attrs=ou,employeeNumber,employeeType by users read

## Stop authentication account from reading anything else. This also 
## stops anonymous.
access to *
       by dn.exact="uid=Authenticate,ou=System,dc=example,dc=com" 
           none
       by users none break

## Prevent DNs in ou=Users from seeing system accounts
access to dn.subtree="ou=System,dc=example,dc=com"
       by dn.subtree="ou=Users,dc=example,dc=com" none
       by users read

## Allow user to add subentries beneath its own record.
access to dn.regex="^.*,uid=([^,]+),ou=Users,dc=example,dc=com$"
       by dn.exact,expand="uid=$1,ou=Users,dc=example,dc=com" write

## The default rule: Allow DNs to modify their own records. Give 
## read access to everyone else.
access to *
       by self write
       by users read
```

While they certainly won't meet all needs, these rules provide a good starting point for balancing security and usability of the directory. Furthermore, they set the stage for some of the things we will be doing later in this book.

In later chapters of this book, the mentioned ACLs will be revisited and fine-tuned to allow additional features, like directory replication.

# Summary

The focus of this chapter has been OpenLDAP security, and we have covered a lot of ground. We began with connection-level security, where we configured SSL/TLS encryption for our directory server. We used StartTLS over the standard LDAP port, and also configured the older (LDAP v2) LDAPS protocol on port 636\. Next, we looked at the process of authenticating to the LDAP. In that part we covered both simple binding and SASL binding. Finally, we took a detailed look at access control lists (ACLs), finishing the chapter with a basic set of ACLs.

In the next chapter we will look at advanced configuration of OpenLDAP's SLAPD server. We will configure our server to host multiple backend databases and we will use directory overlays to add powerful additional features to our SLAPD server.