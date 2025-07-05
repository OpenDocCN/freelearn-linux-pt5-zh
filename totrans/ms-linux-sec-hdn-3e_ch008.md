# 7 SSH Hardening

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file49.png)

The **Secure Shell** (**SSH**) suite is one of those must-have tools for Linux administrators. It allows you to take care of Linux servers from the comfort of your cubicle, or even from the comfort of your own home. Either way, it's a lot better than having to don your parka and jump through security hoops to enter a cold server room. The *secure* in Secure Shell means that everything that you either type or transfer gets encrypted. That eliminates the possibility of someone obtaining sensitive data by plugging a sniffer into your network.

By this stage in your Linux career, you should already know how to use Secure Shell, or SSH, to do remote logins and remote file transfers. What you may not know is that the default configuration of SSH is actually quite insecure. In this chapter, we'll look at how to harden the default configuration in various ways. We'll look at how to use encryption algorithms that are stronger than the default, how to set up passwordless authentication, and how to set up a jail for users of the Secure File Transfer Protocol (SFTP). As a bonus, we'll look at how to scan SSH servers to find vulnerable configurations and how to share a remote directory via **Secure Shell Filesystem** (**SSHFS**).

In this chapter, we'll cover the following topics:

*   Ensuring that SSH protocol 1 is disabled
*   Creating and managing keys for passwordless logins
*   Disabling root user login
*   Disabling username/password logins.
*   Enabling two-factor authentication
*   Configuring Secure Shell with strong encryption algorithms
*   Setting system-wide encryption policies on RHEL 8/9-type systems
*   FIPS mode on RHEL 8/9-type systems
*   Configuring more detailed logging
*   Access control with whitelists and TCP Wrappers
*   Configuring automatic logouts and security banners
*   Other miscellaneous security settings
*   Setting up different configurations for different hosts
*   Setting up different configurations for different users and groups
*   Scanning an SSH server
*   Setting up a chroot environment for SFTP users
*   Setting up shared directories with SSHFS
*   Remotely connecting from Windows desktops

So, if you're ready, let's get started.

## Ensuring that SSH protocol 1 is disabled

In the two previous editions of this book, I told you about how Version 1 of the SSH protocol is severely flawed, and how you should always ensure that it’s not enabled in the `/etc/ssh/sshd_config` file. Nowadays you don’t have to worry about that, because SSH protocol 1 is now gone, and is nothing but a thing of the past. So, yee-haw! It’s time to celebrate.

## Creating and managing keys for passwordless logins

The Secure Shell suite is a great set of tools for communications with remote servers. You can use the `ssh` component to remotely log into the command line of a remote machine, and you can use either `scp` or `sftp` to securely transfer files. The default way to use any of these SSH components is to use the username of a person's normal Linux user account. So, logging into a remote machine from the terminal of my OpenSUSE workstation would look something like this:

```
donnie@linux-0ro8:~> ssh donnie@192.168.0.8
donnie@192.168.0.8's password:
```

While it's true that the username and password go across the network in an encrypted format, making it hard for malicious actors to intercept, it's still not the most secure way of doing business. The problem is that attackers have access to automated tools that can perform brute-force password attacks against an SSH server. Botnets, such as the Hail Mary Cloud, perform continuous scans across the Internet to find Internet-facing servers with SSH enabled.

If a botnet finds that the servers allow SSH access via username and password, it will launch a brute-force password attack. Sadly, such attacks have been successful quite a few times, especially when the server operators allow the root user to log in via SSH.

> This older article provides more details about the Hail Mary Cloud botnet: [http://futurismic.com/2009/11/16/the-hail-mary-cloud-slow-but-steady-brute-force-password-guessing-botnet/](http://futurismic.com/2009/11/16/the-hail-mary-cloud-slow-but-steady-brute-force-password-guessing-botnet/).

In the next section, we'll look at two ways to help prevent these types of attacks:

*   Enabling SSH logins through an exchange of public keys
*   Disabling the root user login through SSH

Now, let's create some keys.

### Creating a user's SSH key set

Each user has the ability to create his or her own set of private and public keys. It doesn't matter whether the user's client machine is running Linux, macOS, Cygwin on Windows, or Bash Shell for Windows. In all cases, the procedure is exactly the same.

There are several different types of keys that you can create, and 3072-bit RSA keys are normally the default. Until very recently, 2,048-bit RSA keys were considered strong enough for the foreseeable future. But now, the most recent guidance from the US **National Institute of Standards and Technology** (**NIST**) says to use either an RSA key of at least 3,072 bits or an **Elliptic Curve Digital Signature Algorithm** (**ECDSA**) key of at least 384 bits. (You'll sometimes see these ECDSA keys referred to as *P-384.*) Their reasoning is that they want to get us ready for quantum computing, which will be so powerful that it will render any weaker encryption algorithms obsolete. Of course, quantum computing isn't practical yet, and so far, it seems to be one of those things that's always just ten years off in the future, regardless of what year it is. But even if we discount the whole quantum thing, we still have to acknowledge that even our current, non-quantum computers keep getting more and more powerful. So, it's still not a bad idea to start going with stronger encryption standards.

> To see the NIST list of recommended encryption algorithms and the recommended key lengths, go to [https://cryptome.org/2016/01/CNSA-Suite-and-Quantum-Computing-FAQ.pdf](https://cryptome.org/2016/01/CNSA-Suite-and-Quantum-Computing-FAQ.pdf).

For these next few demos, let's switch over to an Ubuntu 22.04 client. To create a 3072 RSA key pair, just do this:

```
donnie@ubuntu2204-packt:~$ ssh-keygen
```

We didn’t have to use any option switches, because the command will already create a 3072-bit RSA pair by default. When prompted for the location and name of the keys, I'll just hit the **Enter** key to accept the defaults. You could just leave the private key with a blank passphrase, but that's not a recommended practice.

Note that if you choose an alternative name for your key files, you'll need to type in the entire path to make things work properly. For example, in my case, I could specify the path for `donnie_rsa` keys as `/home/donnie/.ssh/donnie_rsa`.

You'll see your new keys in the `.ssh` directory:

```
donnie@ubuntu2204-packt:~$ ls -l .ssh
total 16
-rw------- 1 donnie donnie    0 Oct  6 22:09 authorized_keys
-rw------- 1 donnie donnie 2655 Nov  1 19:49 id_rsa
-rw-r--r-- 1 donnie donnie  577 Nov  1 19:49 id_rsa.pub
-rw------- 1 donnie donnie  978 Oct 26 20:41 known_hosts
-rw-r--r-- 1 donnie donnie  142 Oct 26 20:41 known_hosts.old
donnie@ubuntu2204-packt:~$
```

The `id_rsa` key is the private key, with read and write permissions only for me. The `id_rsa.pub` public key has to be world-readable. For ECDSA keys, the default length is 256 bits. If you choose to use ECDSA instead of RSA, do the following to create a strong 384-bit key:

```
donnie@ubuntu2204-packt:~$ ssh-keygen -t ecdsa -b 384
```

Either way, when you look in the `.ssh` directory, you'll see that the ECDSA keys are named differently from the RSA keys:

```
donnie@ubuntu2204-packt:~$ ls -l .ssh/id*
-rw------- 1 donnie donnie  667 Nov  1 19:55 .ssh/id_ecdsa
-rw-r--r-- 1 donnie donnie  229 Nov  1 19:55 .ssh/id_ecdsa.pub
-rw------- 1 donnie donnie 2655 Nov  1 19:49 .ssh/id_rsa
-rw-r--r-- 1 donnie donnie  577 Nov  1 19:49 .ssh/id_rsa.pub
donnie@ubuntu2204-packt:~$
```

The beauty of elliptic curve algorithms is that their seemingly short key lengths are just as secure as RSA keys with longer key lengths. And, even the largest ECDSA keys require less computing power than RSA keys. The maximum key length you can do with ECDSA is 521 bits. (Yes, you read that correctly. It's 521 bits, not 524 bits.) So, you may be thinking, *Why don't we just go for the gusto with 521-bit keys?* Well, it's mainly because 521-bit keys aren't recommended by NIST. There's some fear that they may be subject to **padding attacks**, which could allow the bad guys to break your encryption and steal your data.

If you take a gander at the man page for `ssh-keygen`, you'll see that you can also create an `Ed25519` type of key, which you'll sometimes see referred to as `curve25519`. This one isn't included in the NIST list of recommended algorithms and also isn’t allowed by the FIPS regulations, but there are a couple of reasons why some people like to use it.

RSA and DSA can leak private key data when creating signatures if the random number generator of the operating system is flawed. `Ed25519` doesn't require a random number generator when creating signatures, so it's immune to this problem. Also, `Ed25519` is coded in a way that makes it much less vulnerable to side-channel attacks. (A side-channel attack is when someone tries to exploit weaknesses in the underlying operating system, rather than in the encryption algorithm.)

The second reason why some folk like `Ed25519` is precisely because it's *not* on the NIST list. These are the folk who, rightly or wrongly, don't trust the recommendations of government agencies.

> Quite a few years ago, in the early part of this century, there was a bit of a scandal that involved the **Dual Elliptic Curve Deterministic Random Bit Generator** (**Dual_EC_DRBG**). This was a random number generator that was meant for use in elliptic curve cryptography. The problem was that, early on, some independent researchers found that it had the capability to have *back doors* inserted by anyone who knew about this capability. And, it just so happened that the only people who were supposed to know about this capability were the folk who work at the U.S. **National Security Agency** (**NSA**). At the NSA's insistence, NIST included Dual_EC_DRBG in their NIST list of recommended algorithms, and it stayed there until they finally removed it in April 2014\. You can get more details about this at the following links:
> 
> > [https://www.pcworld.com/article/2454380/overreliance-on-the-nsa-led-to-weak-crypto-standard-nist-advisers-find.html](https://www.pcworld.com/article/2454380/overreliance-on-the-nsa-led-to-weak-crypto-standard-nist-advisers-find.html)
> 
> > [http://www.math.columbia.edu/~woit/wordpress/?p=7045](http://www.math.columbia.edu/~woit/wordpress/?p=7045)
> 
> > You can read the details about Ed25519 here: [https://ed25519.cr.yp.to/](https://ed25519.cr.yp.to/).

There's only one key size for `Ed25519`, which is 256 bits. So, to create a `curve25519` key, just do this:

```
donnie@ubuntu2204-packt:~$ ssh-keygen -t ed25519
```

Here are the keys that I've created:

```
donnie@ubuntu2204-packt:~$ ls -l .ssh/*25519*
-rw------- 1 donnie donnie 464 Nov  1 20:35 .ssh/id_ed25519
-rw-r--r-- 1 donnie donnie 105 Nov  1 20:35 .ssh/id_ed25519.pub
donnie@ubuntu2204-packt:~$
```

There are, however, some potential downsides to `ed25519`:

*   First, it isn't supported by older SSH clients. However, if everyone on your team is using current operating systems that use current SSH clients, this shouldn't be a problem.
*   The second is that it only supports one certain set key length, which is the equivalent of either 256-bit elliptic curve algorithms or 3,000-bit RSA algorithms. So, it might not be quite as future-proof as the other algorithms that we've covered.
*   Lastly, you can't use it if your organization is required to remain compliant with either NIST recommendations or FIPS requirements.

Okay, there is one other type of key that we haven't covered. That's the old-fashioned DSA key, which `ssh-keygen` will still create if you tell it to. But, don't do it. The DSA algorithm is old, creaky, and very insecure by modern standards. So, when it comes to DSA, just say *No*.

### Transferring the public key to the remote server

Transferring my public key to a remote server allows the server to readily identify both me and my client machine. Before I can transfer the public key to the remote server, I need to add the private key to my session keyring. This requires two commands. (One command invokes `ssh-agent`, while the other actually adds the private key to the keyring):

```
donnie@ubuntu2204-packt:~$ exec /usr/bin/ssh-agent $SHELL
donnie@ubuntu2204-packt:~$ ssh-add
Enter passphrase for /home/donnie/.ssh/id_rsa: 
Identity added: /home/donnie/.ssh/id_rsa (donnie@ubuntu2204-packt)
Identity added: /home/donnie/.ssh/id_ecdsa (donnie@ubuntu2204-packt)
Identity added: /home/donnie/.ssh/id_ed25519 (donnie@ubuntu2204-packt)
donnie@ubuntu2204-packt:~$
```

Finally, I can transfer my public key(s) to my AlmaLinux 9 server, which is at address `192.168.0.17`:

```
donnie@ubuntu2204-packt:~$ ssh-copy-id donnie@192.168.0.17
The authenticity of host '192.168.0.17 (192.168.0.17)' can't be established.
ED25519 key fingerprint is SHA256:GkpFwJdpWRQ5GawgFEz9bgDSny//E1I5aLGkjU9DWWY.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 3 key(s) remain to be installed -- if you are prompted now it is to install the new keys
donnie@192.168.0.17's password: 
Number of key(s) added: 3
Now try logging into the machine, with:   "ssh 'donnie@192.168.0.17'"
and check to make sure that only the key(s) you wanted were added.
donnie@ubuntu2204-packt:~$
```

Normally, you would only create one pair of keys of whichever type you choose. As you can see here, I've created three key pairs, one pair of each type. All three private keys were added to my session keyring, and all three public keys were transferred to the remote server.

The next time that I log in, I'll use the key exchange, and I won't have to enter a password:

```
donnie@ubuntu2204-packt:~$ ssh donnie@192.168.0.17
Last login: Tue Nov  1 16:52:27 2022
[donnie@donnie-ca ~]$ 
```

As I mentioned previously, you would normally only create one key pair per machine. However, there are exceptions to this rule. Some administrators prefer to use a different key pair for each server that they administer, rather than using the same key pair for all servers. A handy way to do this is to create keys with filenames that match the hostnames of the respective servers. Then, you can use the `-i` option to specify which key pair that you want to use.

In this example, I only have one server, but I have multiple keys for it. Let's say that I prefer to use the `Ed25519` keys:

```
donnie@ubuntu2204-packt:~$ ssh -i ~/.ssh/id_ed25519 donnie@192.168.0.17
Last login: Tue Nov  1 16:56:43 2022 from 192.168.0.14
[donnie@donnie-ca ~]$ 
```

So, now you're wondering, *How is that secure if I can log in without entering my password?* The answer is that once you close the client machine's terminal window that you used for logging in, the private key will be removed from your session keyring. When you open a new terminal and try to log in to the remote server, you'll see this:

```
donnie@ubuntu2204-packt:~$ ssh donnie@192.168.0.17
Enter passphrase for key '/home/donnie/.ssh/id_rsa': 
Last login: Tue Nov  1 16:58:22 2022 from 192.168.0.14
[donnie@donnie-ca ~]$
```

Now, every time I log into this server, I'll need to enter the passphrase for my private key, until I add it back to the session keyring with the two commands that I showed you in the preceding section.

#### Hands-on lab – Creating and transferring SSH keys

In this lab, you'll use one virtual machine (VM) as your client, and one VM as the server. Alternatively, if you're using a Windows host machine, you can use Cygwin, PowerShell, or the built-in Windows Bash shell for the client. (Be aware, though, that PowerShell and the Windows Bash shell store the key files in alternate locations.) If you're on either a Mac or a Linux host machine, you can use the host machine's native command-line terminal as the client. In any case, the procedure will be the same.

For the server VM, use either Ubuntu 22.04 or CentOS 7.

> This procedure does work the same on AlmaLinux 8 and 9\. However, we'll be using this same VM for the next few labs, and AlmaLinux has some special considerations that we'll look at later.

Let's get started:

1.  On the client machine, create a pair of 384-bit elliptic curve keys. Accept the default filename and location and create a passphrase:

```
ssh-keygen -t ecdsa -b 384
```

1.  Observe the keys, taking note of the permissions settings:

```
ls -l ./ssh
```

1.  Add your private key to your session keyring. Enter your passphrase when prompted:

```
exec /usr/bin/ssh-agent $SHELL
ssh-add
```

1.  Transfer the public key to the server VM. When prompted, enter the password for your user account on the server VM. (Substitute your own username and IP address in the following command.):

```
ssh-copy-id donnie@192.168.0.7
```

1.  Log into the server VM as you normally would:

```
ssh donnie@192.168.0.7
```

1.  Observe the `authorized_keys` file that was created on the server VM:

```
ls -l .ssh
cat .ssh/authorized_keys
```

1.  Log out of the server VM and close the terminal window on the client. Open another terminal window and try to log into the server again. This time, you should be prompted to enter the passphrase for your private key.
2.  Log back out of the server VM and add your private key back to the session keyring of your client. Enter the passphrase for your private key when prompted:

```
exec /usr/bin/ssh-agent $SHELL
ssh-add
```

As long as you keep this terminal window open on your client, you'll be able to log into the server VM as many times as you want without having to enter a password. However, when you close the terminal window, your private key will be removed from your session keyring.

1.  Keep your server VM, because we'll do more with it in a bit.

You've reached the end of the lab – congratulations!

What we've done here is good, but it's still not quite enough. One flaw is that if you go to another client machine, you'll still be able to use the normal username/password authentication to log in. That's okay; we'll fix that in a few moments.

### Disabling root user login

A few years ago, there was a somewhat celebrated case where malicious actors had managed to plant malware on quite a few Linux servers somewhere in southeast Asia. There were three reasons that the bad guys found this so easy to do:

*   The Internet-facing servers involved were set up to use username/password authentication for SSH.
*   The root user was allowed to log in through SSH.
*   User passwords, including the root user's password, were incredibly weak.

All this meant that it was easy for the Hail Mary botnet to brute-force its way in.

Different distros have different default settings for root user login. In the `/etc/ssh/sshd_config` file of your CentOS 7 or AlmaLinux 8 machine, you'll see this line:

```
#PermitRootLogin yes
```

Unlike what you have in most configuration files, the commented-out lines in `sshd_config` define the default settings for the Secure Shell daemon. So, this line indicates that the root user is indeed allowed to log in through SSH. To change that, I'll remove the comment symbol and change the setting to `no`:

```
PermitRootLogin no
```

To make the new setting take effect, I'll reload the SSH daemon, which is named `sshd` on CentOS and AlmaLinux, and is named `ssh` on Ubuntu:

```
sudo systemctl reload sshd
```

On the Ubuntu machine, the default setting looks a bit different:

```
PermitRootLogin prohibit-password
```

This means that the root user is allowed to log in, but only via a public key exchange. This is probably secure enough if you really need to allow the root user to log in. But in most cases, you'll want to force admin users to log in with their normal user accounts and use `sudo` for their admin needs. So, in most cases, you still want to change this setting to `no`.

On your AlmaLinux 9 machine, you’ll see that it also has `PermitRootLogin` set to `prohibit-password` by default.

> Be aware that if you deploy a Linux instance on a cloud service, such as Rackspace or Vultr, the service owners will have you log into the VM with the root user account. The first thing you'll want to do is create your own normal user account, log back in with that account, disable the root user account, and disable the root user login in `sshd_config`. Microsoft Azure is one exception to this rule because it automatically creates a non-privileged user account for you.

You'll be able to practice this in just a few moments, in the next section.

### Disabling username/password logins

This is something that you'll only want to do after you've set up the key exchange with your clients. Otherwise, clients will be locked out of doing remote logins.

#### Hands-on lab – disabling root login and password authentication

For this lab, use the same server VM that you used for the previous lab. Let's get started:

1.  On either an Ubuntu, CentOS, or AlmaLinux 8 server VM, look for this line in the `sshd_config` file:

```
#PasswordAuthentication yes
```

1.  Remove the comment symbol, change the parameter value to `no`, and reload the SSH daemon. The line should now look like this:

```
PasswordAuthentication no
```

Now, when the botnets scan your system, they'll see that doing a brute-force password attack would be useless. They'll then just go away and leave you alone.

1.  Look for either of these two lines, depending on whether the server is an Ubuntu or a CentOS 7/AlmaLinux 8 VM:

```
#PermitRootLogin yes
#PermitRootLogin prohibit-password
```

1.  Uncomment the line and change it to the following:

```
PermitRootLogin no
```

1.  Reload the SSH daemon so that it will read in the new changes. On Ubuntu, do this:

```
sudo systemctl reload ssh
```

On CentOS/AlmaLinux, do this:

```
sudo systemctl reload sshd
```

1.  Attempt to log into the server VM from the client that you used in the previous lab.
2.  Attempt to log into the server VM from another client on which you haven't created a key pair. (You shouldn't be able to.)
3.  As before, keep the server VM, because we'll do more with it in a bit.

You've reached the end of the lab – congratulations!

Now that we've covered how to create a private/public key pair on the client side and how to transfer the public key to the server, let’s talk about setting up two-factor authentication.

### Enabling two-factor authentication

Two-factor authentication can give an extra layer of protection. If you own a smart phone, you can set this up with **Google Authenticator**, which will present you with a **one-time password** for logging in at the local terminal, invoking a `sudo` command, or logging in remotely via SSH. Before we get started though, there are a few caveats that you need to understand:

*   To make this work on a Linux machine, you’ll need to install a PAM module that isn’t supplied by Google. It’s in the repositories for some, but not all, Linux distros. (Of course, you could download the source code from the Github repository and compile it yourself, but that’s beyond the scope of this book.)
*   The creator of this PAM module has created some semblance of documentation, but it’s not very useful. If you search for documentation, you’ll find some blog posts with procedures that are worse than useless, because they *will* break your system if you follow them.
*   You can set up your machine to require Google Authentication for either global usage, or for just logging in via SSH. (Global usage means that an Authenticator code will be needed for logging in at the local terminal, using `sudo`, *and* logging in remotely via SSH.)
*   If you’re dealing with multiple users, each user will need to set up Google Authenticator for his or her own user account, with his or her own smart phone.

Now, with that out of the way, let’s set up our Ubuntu 22.04 server with Google Authenticator for local log-ins and sudo:

> Note that this PAM module is in the normal Ubuntu repository and in the EPEL repository for RHEL 8-type distros. It’s not available at all for RHEL 9-type distros.

#### Hands-on lab--Setting up two-factor authentication on Ubuntu 22.04

For this lab, start with a fresh Ubuntu 22.04 virtual machine that’s not set up for public key authentication. (That will save a lot of confusion when going through this procedure.)

1.  Install Google Authenticator on your smart phone. (It’s in the normal app stores for both Android and iPhone.)
2.  On your Ubuntu VM, install the `libpam-google-authenticator` package, like this:

```
sudo apt install libpam-google-authenticator
```

1.  For this step, if you haven’t already, use SSH to remotely log into the Ubuntu VM from the GUI-type terminal of your host machine. (That’s because you might need to resize things to make the next step work.) Now, from this GUI-type terminal, run the `google-authenticator` app, like so:

```
google-authenticator
```

1.  A big QR code will now show up on your screen. If the whole code graphic isn’t visible, use your GUI terminal controls to zoom out until the whole graphic is visible.
2.  Bring up the Google Authenticator app on your smart phone, and touch the `+` sign in the lower right-hand corner of the screen. Choose the **Scan a QR code** option, and then take a picture of your QR code.
3.  On your smart phone, note that a new entry for your Ubuntu VM has been added to the list. On the Ubuntu VM, enter the verification code that shows up with that entry.
4.  The next thing you’ll see on the Ubuntu terminal is your emergency scratch codes. Copy them down and store them in a safe location. (If you lose your mobile phone, you’ll use these scratch codes to log in.)
5.  Next, you’ll be asked series of questions. Just enter `y` for everything.
6.  In this step, you’ll set up two-factor authentication for logging in at the local terminal and for using `sudo`. Open the `/etc/pam.d/common-auth` file in your favorite text editor. Add the `auth required pam_google_authenticator.so` line as the first parameter. The top portion of the file should now look something like this:

```
#
# /etc/pam.d/common-auth - authentication settings common to all services
#
# This file is included from other service-specific PAM config files,
. . .
. . .
auth required pam_google_authenticator.so
# here are the per-package modules (the "Primary" block)
. . .
. . .
```

> Certain blog posts that you’ll find tell you to add this line to the *end* of the file. Be aware that if you do that, you *will* get locked out of your machine, and you’ll need to perform an emergency procedure to get back in to fix it. Any time you edit a PAM file, it’s vitally important that you place the directives in the proper order. (In case you’re wondering, I’ll show you the emergency procedure in *Chapter 16*, *Security Tips & Tricks for the Busy Bee*.)

1.  At the local terminal of the Ubuntu VM, log out and then log back in. When prompted, enter the verification code from your smart phone app.
2.  Perform a command that requires `sudo` privileges. You should see something like this:

```
donnie@ubuntu2204-packt:~$ sudo nft list ruleset
Verification code: 
[sudo] password for donnie:
. . .
. . .
Enter the verification code at the prompt.
```

> Note that you won’t be required to enter a verification code again until the `sudo` timer times out.

1.  From either your host machine or another virtual machine, remotely log into the Ubuntu VM via SSH. You should still be able to do that, because we haven’t yet configured the `/etc/ssh/sshd_config` file. Open the `sshd_config` file in your text editor, and change the `KbdInteractiveAuthentication no` line to `KbdInteractiveAuthentication yes`.
2.  Reload the Secure Shell configuration:

```
sudo systemctl reload ssh
```

1.  Try logging in again from either your host machine or another virtual machine. This time, you should be prompted to enter your verification code.
2.  Now, let’s say that your organization needs two-factor authentication for remote SSH logins, but doesn’t need it for either local logins or `sudo` operations. Let’s change the configuration so that only remote users will need to enter a code. Open the `/etc/pam.d/common-auth` file in your text editor, and remove the line that you inserted in Step 9.
3.  Open the `/etc/pam.d/sshd` file in your text editor, and add that line just under the `@include common-auth` line at the top of the file. The top portion of the file should now look like this:

```
PAM configuration for the Secure Shell service
      # Standard Un*x authentication.
     @include common-auth
     auth required pam_google_authenticator.so
```

You should now be able to log in to the local terminal and perform `sudo` actions without having to enter a verification code. Instead, you should only have to enter a verification code when logging in remotely.

1.  End of lab.

Next, let’s look at using Google Authenticator together with key exchange on our Ubuntu machine.

#### Hands-on lab--Using Google Authenticator with key exchange on Ubuntu

For this lab, use the same Ubuntu virtual machine that you used for the previous lab.

1.  On either your host machine or another virtual machine, create a pair of keys and transfer them to the Ubuntu virtual machine, as you did in the *Creating and transferring SSH keys* lab. This time, you should be prompted to enter a verification code when you execute the `ssh-copy-id` command.
2.  On the Ubuntu virtual machine, open the `/etc/ssh/sshd_config` file in your text editor. This time, instead of changing the `#PasswordAuthentication yes` line, add this line below the `KbdInteractiveAuthentication yes` line:

```
AuthenticationMethods publickey keyboard-interactive:pam
```

After reloading the SSH configuration, you’ll see that you’ll be able to remotely log in by using key exchange if you’re logging in from a machine that has that set up. If you’re logging in from a machine that doesn’t have key exchange set up, you’ll still be able to log in with a password and a verification code. So, we don’t have true two-factor authentication just yet.

1.  To require both key-based authentication and Google Authenticator verification, change the above line to look like this:

```
AuthenticationMethods publickey,keyboard-interactive:pam
```

1.  After reloading the SSH configuration, you’ll only be allowed to log in from machines for which you’ve set up key exchange. You now effectively have three-factor authentication, because you’ll still be prompted to enter your normal login password.
2.  To disable the password login so that you’ll only be using key exchange and a verification code, open the `/etc/pam.d/sshd` file in your text editor. At the very top of the file, find the `@include common-auth` line and change it to `#@include common-auth`.
3.  Verify that the key exchange works by trying to log in from a virtual machine on which you haven’t performed the key exchange setup. (You shouldn’t be allowed to.)
4.  That’s it. End of lab.

Now, let’s see what we can do with AlmaLinux 8.

#### Hands-on lab--Setting up two-factor authentication on AlmaLinux 8

For this lab, I assume that you’ve already installed the Google Authenticator app on your smart phone.

The Authenticator PAM module isn’t in any of the repositories for the RHEL 9 distros, but it is in the EPEL repository for the RHEL 8 distros. (That might change by the time you read this, so it won’t hurt to check if you want to try this on AlmaLinux 9.) So, fire up a fresh AlmaLinux 8 VM, and let’s get started.

1.  Install the PAM module like this:

```
sudo dnf install epel-release
sudo dnf install google-authenticator qrencode-libs
```

Note that you need the `qrencode-libs` package in order to produce a QR code.

1.  From the GUI terminal of your host machine, use SSH to remotely log into the AlmaLinux 8 VM. This will allow you to resize the QR code image so that you can take a picture of it with your smart phone. Then, run the `google-authenticator` app, like this:

```
google-authenticator -s ~/.ssh/google_authenticator
```

1.  This time, we’re creating the `google_authenticator` file within the `.ssh` directory, because AlmaLinux is set up to use SELinux. When you try to log in remotely with Authenticator enabled, the SSH daemon will try to write to the `google_authenticator` file. SELinux prevents SSH from writing to files that are outside of the `.ssh` directory. (We’ll talk more about SELinux in *Chapter 10*, *Implementing Mandatory Access Control with SELinux and AppArmor*.)
2.  Follow through on the Authenticator setup, the same as you did in Steps 4 through 8 of the *Setting up two-factor authentication on Ubuntu 22.04* lab.
3.  Open the `/etc/pam.d/sshd` file in your text editor. Add this line to the very bottom of the file:

```
auth required pam_google_authenticator.so secret=/home/${USER}/.ssh/google_authenticator
```

(Note that the line wraps around on the printed page.)

1.  Open the `/etc/ssh/sshd_config` file in your text editor. Find the line that says `#ChallengeResponseAuthentication no` and change it to `ChallengeResponseAuthentication yes`.
2.  Reload or restart the `sshd` service:

```
sudo systemctl reload sshd
```

1.  Set the proper SELinux security context on the google_authenticator file that you created:

```
cd
sudo restorecon .ssh/google_authenticator
```

1.  Log out of the remote session, and try logging back in. This time, you should be prompted to enter a verification code.

Next, let’s set AlmaLinux up for using key exchange.

#### Hand-on lab--Using Google Authenticator with key exchange on AlmaLinux 8

This will mostly be the same as it was for Ubuntu, with only a few differences.

1.  Transfer the public key from your host machine to the AlmaLinux 8 machine as you did in the *Creating and transferring SSH keys* lab.
2.  In the `/etc/ssh/sshd_config` file, change the `#PasswordAuthentication yes` line to `PasswordAuthentication no`, and reload the SSH configuration. Now, you’ll only be using key exchange to log in, which will completely bypass the Authenticator. Let’s fix things so that you’ll be using both.
3.  In the `/etc/ssh/sshd_config` file, add the following line just beneath the `PasswordAuthentication no` line:

```
AuthenticationMethods publickey,password publickey,keyboard-interactive
```

1.  After reloading the SSH configuration, you’ll have three-factor authentication, because you’ll need to enter both your password and a verification code along with the key exchange.
2.  If desired, you can easily disable the password prompt and just use the key exchange and verification code. In the `/etc/pam.d/sshd` file, find the `auth substack password-auth` line at the top of the file, and change it to `#auth substack password-auth`.
3.  That’s all there is to Google Authenticator.

In the next section, let’s make sure that we’re only using the strongest encryption algorithms.

### Configuring Secure Shell with strong encryption algorithms

As I mentioned previously, the current set of NIST recommendations, the **Commercial National Security Algorithm Suite** (**CNSA Suite**), involves using stronger algorithms and longer keys than what we needed to use previously. I'll summarize the new recommendations here in this table:

| **Algorithm** | **Usage** |
| RSA, 3,072 bits or larger | Key establishment and digital signatures |
| Diffie-Hellman (DH), 3,072 bits or larger | Key establishment |
| ECDH with NIST P-384 | Key establishment |
| ECDSA with NIST P-384 | Digital signatures |
| SHA-384 | Integrity |
| AES-256 | Confidentiality |

In other publications, you might see that NIST Suite B is the recommended standard for encryption algorithms. Suite B is an older standard that has been replaced by the CNSA Suite.

Another cryptographic standard that you might have to work with is the **Federal Information Processing Standard** (**FIPS**), which is also promulgated by the U.S. government. The current version is FIPS 140-3, which gained its final approval on September 22, 2019.

#### Understanding SSH encryption algorithms

SSH works with a combination of symmetric and asymmetric cryptography, similar to how Transport Layer Security works. The SSH client starts the process by using the public key method to set up an asymmetric session with an SSH server. Once this session has been set up, the two machines can agree on and exchange a secret code, which they'll use to set up a symmetric session. (As we saw previously with TLS, we want to use symmetric cryptography for performance reasons, but we need an asymmetric session to perform the secret key exchange.) To perform this magic, we need four classes of encryption algorithms, which we'll configure on the server side. These are:

*   **Ciphers**: These are the symmetric algorithms that encrypt the data that the client and server exchange with each other.
*   **HostKeyAlgorithms**: This is the list of host key types that the server can use.
*   **KexAlgorithms**: These are the algorithms that the server can use to perform the symmetric key exchange.
*   **MAC**: Message Authentication Codes are hashing algorithms that cryptographically sign the encrypted data in transit. This ensures data integrity and will let you know if someone has tampered with your data.

The best way to get a feel for this is to look at the `sshd_config` man page, like this:

```
man sshd_conf
```

I could use any VM to demo this. For now, though, I'm going with CentOS 7, unless I state otherwise. (The lists of default and available algorithms will be different for different Linux distributions and versions.) Also, note that to demo this, we want to look at in the `sshd_config` man page to see the lists of algorithms that are **available** and **enabled**. The **enabled** list is in the man pages for CentOS 7 and AlmaLinux 8, but not in the man page for AlmaLinux 9.

First, let's look at the list of supported ciphers. Scroll down the man page until you see them:

```
3des-cbc
aes128-cbc
aes192-cbc
aes256-cbc
aes128-ctr
aes192-ctr
aes256-ctr
aes128-gcm@openssh.com
aes256-gcm@openssh.com
arcfour
arcfour128
arcfour256
blowfish-cbc
cast128-cbc
chacha20-poly1305@openssh.com
```

However, not all of these supported ciphers are enabled. Just below this list, we can see the list of ciphers that are enabled by default:

```
chacha20-poly1305@openssh.com,
aes128-ctr,aes192-ctr,aes256-ctr,
aes128-gcm@openssh.com,aes256-gcm@openssh.com,
aes128-cbc,aes192-cbc,aes256-cbc,
blowfish-cbc,cast128-cbc,3des-cbc
```

Next, in alphabetical order, are the **HostKeyAlgorithms**. The list on CentOS 7 looks like this:

```
ecdsa-sha2-nistp256-cert-v01@openssh.com,
ecdsa-sha2-nistp384-cert-v01@openssh.com,
ecdsa-sha2-nistp521-cert-v01@openssh.com,
ssh-ed25519-cert-v01@openssh.com,
ssh-rsa-cert-v01@openssh.com,
ssh-dss-cert-v01@openssh.com,
ecdsa-sha2-nistp256,ecdsa-sha2-nistp384,ecdsa-sha2-nistp521,
ssh-ed25519,ssh-rsa,ssh-dss
```

Next, scroll down to the **KexAlgorithms** (short for **Key Exchange Algorithms**) section. You'll see a list of supported algorithms, which looks like this:

```
curve25519-sha256
curve25519-sha256@libssh.org
diffie-hellman-group1-sha1
diffie-hellman-group14-sha1
diffie-hellman-group-exchange-sha1
diffie-hellman-group-exchange-sha256
ecdh-sha2-nistp256
ecdh-sha2-nistp384
ecdh-sha2-nistp521
```

Be aware that this list can vary from one distribution to the next. For example, RHEL 8/AlmaLinux 8 supports three additional algorithms that are newer and stronger. Its list looks like this:

```
curve25519-sha256
curve25519-sha256@libssh.org
diffie-hellman-group1-sha1
diffie-hellman-group14-sha1
diffie-hellman-group14-sha256
diffie-hellman-group16-sha512
diffie-hellman-group18-sha512
diffie-hellman-group-exchange-sha1
diffie-hellman-group-exchange-sha256
ecdh-sha2-nistp256
ecdh-sha2-nistp384
ecdh-sha2-nistp521
```

You’ll see the same list on an AlmaLinux 9 machine, except that the `sntrup761x25519-sha512@openssh.com` algorithm has been added.

Next, you'll see the list of algorithms that are enabled by default:

```
curve25519-sha256,curve25519-sha256@libssh.org,
ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,
diffie-hellman-group-exchange-sha256,
diffie-hellman-group14-sha1,
diffie-hellman-group1-sha1
```

This list can also vary from one Linux distribution to another. (In this case, though, there's no difference between CentOS 7 and AlmaLinux 8.)

Finally, we have the MAC algorithms. The default list of enabled algorithms looks like this on CentOS 7:

```
umac-64-etm@openssh.com,umac-128-etm@openssh.com,
hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,
hmac-sha1-etm@openssh.com,
umac-64@openssh.com,umac-128@openssh.com,
hmac-sha2-256,hmac-sha2-512,hmac-sha1,
hmac-sha1-etm@openssh.com
```

To see the list of algorithms that your particular system supports, either look at the `sshd_config` man page for that machine or perform the following `ssh -Q` commands:

```
ssh -Q cipher
ssh -Q key
ssh -Q kex
ssh -Q mac
```

When you look in the `/etc/ssh/sshd_config` file, you won't see any lines that configure any of these algorithms. That's because the default list of algorithms is hard coded into the SSH daemon. The only time you'll configure any of these is if you want to either enable an algorithm that isn't enabled or disable one that is. Before we do that, let's scan our system to see what is enabled and see if the scanner can make any recommendations.

### Scanning for enabled SSH algorithms

We have two good ways to scan an SSH server. If your server is accessible via the Internet, you can go to the SSHCheck site at [https://sshcheck.com/](https://sshcheck.com/).

Then, just type in either the IP address or hostname of your server. If you've changed the port from the default port `22`, enter the port number as well. When the scan completes, you'll see the list of enabled algorithms, along with recommendations on which ones to either enable or disable.

If the machine that you want to scan isn't accessible from the Internet, or even if it is, you can use a local scanning tool. In the previous edition of this book, we used the **ssh_scan** tool. Sadly, this tool is no longer supported, and it doesn’t work on newer Linux distros that come with OpenSSL version 3\. So instead, let’s try this with the Nmap scripting engine.

#### Hands-on lab – Scanning with Nmap

For this lab, you can use any of your virtual machines. Let's get started:

1.  First, install the `nmap` package from your normal distro repository. On Ubuntu do:

```
sudo apt update
sudo apt install nmap
```

On CentOS 7, do this:

```
sudo yum install nmap
```

On AlmaLinux 8 or 9, do this:

```
sudo dnf install nmap
```

1.  Use `nmap` with the `ssh2-enum-algos.nse` script to scan the server VM that you created and configured in the previous labs. Substitute your own IP address for the one I'm using here. Note that even if you haven't created a key pair on the scanner machine, the scan still works against machines that have had username/password authentication disabled. (But, of course, you won't be able to log in from the scanner machine):

```
nmap --script=ssh2-enum-algos.nse 192.168.0.14
```

Note that if you’re scanning a machine with an enabled firewall, you might get an error message about how the scan has been blocked. If that happens, try adding the `-Pn` switch, so that the command will look like this:

```
nmap -Pn --script=ssh2-enum-algos.nse 192.168.0.14
```

1.  Repeat the scan, but this time, save the output to a normal text file, like so:

```
nmap --script=ssh2-enum-algos.nse 192.168.0.14 -oN ubuntuscan.txt
```

1.  Open the text file in a normal text editor or pager. You'll see a complete list of all of the algorithms that are enabled. Compare your results with the standards that are applicable to your circumstances, such as NIST's CNSA standard, to be sure you enable or disable the right things.
2.  On either your host machine or a VM with a desktop interface, visit the Shodan website at [https://www.shodan.io](https://www.shodan.io). Type `ssh` into the search window and observe the list of Internet-facing SSH servers that comes up. Click on different IP addresses until you find an SSH server that's *not* running on the default port `22`. Observe the list of enabled algorithms for that device.
3.  Scan the device, using the `-p` switch to scan the different port, like so:

```
nmap -p 2222 --script=ssh2-enum-algos.nse 192.168.0.14
```

1.  Note that in addition to the list of enabled algorithms that you saw on Shodan, you now have a list of weak ones that the owner of this device needs to disable.
2.  Keep both this scanner and this server VM handy, because we'll use them again after we disable some algorithms.

You've reached the end of the lab – congratulations!

Okay, let's disable some of the creaky, old, and weak stuff.

### Disabling weak SSH encryption algorithms

As I said before, we want to compare our scan results against the NIST recommendations, and configure things accordingly. Understand though, that the list of available algorithms differs from one Linux distro to the next. To make things less confusing, I'll present two hands-on procedures in this section. One is for Ubuntu 22.04, while the other is for CentOS 7\. AlmaLinux 8 and 9 have their own unique way of doing business, so I'm saving that for the next section.

#### Hands-on lab – disabling weak SSH encryption algorithms – Ubuntu 22.04

For this lab, you'll need the VM that you've been using as a scanner, and another Ubuntu 22.04 VM to scan and configure. Let's get started:

1.  If you haven't done so already, scan the Ubuntu 22.04 VM and save the output to a file:

```
nmap --script=ssh2-enum-algos.nse 192.168.0.14 -oN ubuntuscan.txt
```

1.  Count the number of lines in the file by doing:

```
wc -l ubuntuscan.txt
```

1.  On the target Ubuntu 22.04 VM, open the `/etc/ssh/sshd_config` file in your preferred text editor. Toward the top of the file, find these two lines:

```
# Ciphers and keying
#RekeyLimit default none
```

1.  Beneath those two lines, insert these three lines:

```
Ciphers -aes128-ctr,aes192-ctr,aes128-gcm@openssh.com
KexAlgorithms ecdh-sha2-nistp384
MACs -hmac-sha1-etm@openssh.com,hmac-sha1,umac-64-etm@openssh.com,umac-64@openssh.com,umac-128-etm@openssh.com,umac-128@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-256
```

1.  In the `Ciphers` and `MACs` lines, you see a comma-separated list of algorithms that were disabled by the preceding `-` sign. (You only need one `-` to disable all the algorithms in the list.) In the `KexAlgorithms` line, there's no `-` sign. This means that the algorithm that's listed on that line is the only one that is enabled.
2.  Save the file and restart the SSH daemon. Verify that it started correctly:

```
sudo systemctl restart ssh
sudo systemctl status ssh
```

1.  Scan the Ubuntu 22.04 VM again, saving the output to a different file:

```
nmap --script=ssh2-enum-algos.nse 192.168.0.14 -oN ubuntuscan_modified.txt
```

1.  Count the number of lines in the new file:

```
wc -l ubuntuscan_modified.txt
```

1.  On the scanner VM, use `diff` to compare the two files. You should see fewer algorithms than you saw previously:

```
diff -y ubuntuscan.txt ubuntuscan_modified.txt
```

> The sharp-eyed among you will notice that we left one Cipher that isn't on the NIST CNSA list. `chacha20-poly1305@openssh.com` is a lightweight algorithm that's good for use with low-powered, hand-held devices. It's a good, strong algorithm that can replace the venerable **Advanced Encryption Standard** (**AES**) algorithm, but with higher performance. However, if you have to remain 100% compliant with the NIST CNSA standard, then you might have to disable it.

You've reached the end of the lab – congratulations!

Next, let's work with CentOS 7.

#### Hands-on lab – disabling weak SSH encryption algorithms – CentOS 7

You'll notice two things when you start working with CentOS 7:

*   **More algorithms enabled**: A default SSH configuration on CentOS 7 has a lot more enabled algorithms than what Ubuntu 22.04 has. This includes some really ancient stuff that you really don't want to see anymore. I'm talking about things such as Blowfish and 3DES, which should have been retired years ago.
*   **A different configuration technique**: On CentOS, placing a `-` sign in front of a list of algorithms that you want to disable doesn't work. Instead, you'll need to list all of the algorithms that you want to enable.

For this lab, you'll need a CentOS 7 VM and the same scanner VM that you've been using. With that in mind, let's get to work:

1.  Scan the CentOS 7 VM and save the output to a file. Note that due to the CentOS 7 firewall, you’ll need to add the `-Pn` option:

```
nmap -Pn --script=ssh2-enum-algos.nse 192.168.0.12 -oN centos7scan.txt
```

1.  Count the number of lines in the output file:

```
wc -l centos7scan.txt
```

1.  On the target CentOS 7 VM, open the `/etc/ssh/sshd_config` file in your preferred text editor. Toward the top of the file, find these two lines:

```
# Ciphers and keying
#RekeyLimit default none
```

1.  Beneath those two lines, insert these three lines:

```
Ciphers aes256-gcm@openssh.com,aes256-ctr,chacha20-poly1305@openssh.com
KexAlgorithms ecdh-sha2-nistp384
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-256
```

1.  As I mentioned previously, with CentOS, using `-` to disable algorithms doesn't work. Instead, we have to list all of the algorithms that we do want to enable.
2.  Save the file and reload the SSH daemon. Verify that it started correctly:

```
sudo systemctl reload sshd
sudo systemctl status sshd
```

1.  Scan the CentOS 7 VM again, saving the output to a different file:

```
nmap -Pn --script=ssh2-enum-algos.nse 192.168.0.12 -oN centos7scan_modified.txt
```

1.  Count the number of lines in the new output file:

```
wc -l centos7scan_modified.txt
```

1.  On the scanner VM, use `diff` to compare the two files. You should see fewer algorithms than you saw previously:

```
diff -y centos7scan.txt centos7scan_modified.txt
```

> As before, I left the `chacha20-poly1305@openssh.com` algorithm enabled. If you have to remain 100% compliant with the NIST CNSA standard, then you might have to disable it.

You've reached the end of the lab – congratulations!

Next, let's look at a handy new feature that comes with the RHEL 8 and 9 families.

### Setting system-wide encryption policies on RHEL 8/9 and AlmaLinux 8/9

In *Chapter 6*, *Encryption Technologies*, we briefly looked at how to set system-wide encryption policies on AlmaLinux 8 and 9\. With this cool feature, you no longer have to configure crypto policies for each individual daemon. Instead, you just run a couple of simple commands, and the policy is instantly changed for multiple daemons. To see which daemons are covered, look in the `/etc/crypto-policies/back-ends/` directory. Here's a partial view of what's there:

```
[donnie@localhost back-ends]$ ls -l
total 0
. . .
. . .
lrwxrwxrwx. 1 root root 46 Sep 24 18:17 openssh.config -> /usr/share/crypto-policies/DEFAULT/openssh.txt
lrwxrwxrwx. 1 root root 52 Sep 24 18:17 opensshserver.config -> /usr/share/crypto-policies/DEFAULT/opensshserver.txt
lrwxrwxrwx. 1 root root 49 Sep 24 18:17 opensslcnf.config -> /usr/share/crypto-policies/DEFAULT/opensslcnf.txt
lrwxrwxrwx. 1 root root 46 Sep 24 18:17 openssl.config -> /usr/share/crypto-policies/DEFAULT/openssl.txt
[donnie@localhost back-ends]$
```

As you see, this directory contains symbolic links to text files that contain directives about which algorithms to either enable or disable for the `DEFAULT` configuration. One level up, in the `/etc/crypto-policies/` directory, there's the `config` file. Open it, and you'll see that this is where the system-wide configuration is set:

```
DEFAULT
```

Scanning this VM with its `DEFAULT` configuration shows that quite a few older algorithms are still enabled. To get rid of them, we can change to either `FUTURE` mode or to `FIPS` mode.

> **Tip**
> 
> > At the time of this writing, the EPEL repository uses a security certificate that’s not compatible with `FUTURE` mode. This will prevent you from updating or installing any software packages from the EPEL repository. If you need to set your machine with both `FUTURE` mode and the EPEL repository, be aware that you’ll need to set the machine back to `DEFAULT` mode before you can either fully update your system or install packages from EPEL. (Of course, this problem could be fixed by the time you read this.)

To show you how this works, let's get our hands dirty with another lab.

#### Hands-on lab – setting encryption policies on AlmaLinux 9

Start with a fresh AlmaLinux 9 VM and the scanner VM that you've been using. Now, follow these steps:

1.  On a AlmaLinux 9 VM, use the `update-crypto-policies` utility to verify that it's running in `DEFAULT` mode:

```
sudo update-crypto-policies --show
```

1.  Scan the AlmaLinux 9 VM in its `DEFAULT` configuration and save the output to a file:

```
nmap -Pn --script=ssh2-enum-algos.nse 192.168.0.17 -oN alma9_default.txt
```

1.  On the AlmaLinux 9 VM, set the system-wide crypto policy to `FUTURE`:

```
sudo update-crypto-policies --set FUTURE
```

1.  In the `/etc/ssh/` directory, remove the current host machine keys:

```
sudo rm /etc/ssh/*key*
```

1.  (Don’t worry. New keys will get created when you reboot the machine.)
2.  Reboot the VM:

```
sudo shutdown -r now
```

1.  On the scanner VM, open the `~/.ssh/known_hosts` file in your text editor. Delete the entry that was previously made for the AlmaLinux VM and save the file. (We have to do this because the public key fingerprint on the AlmaLinux VM will have changed because of the new policy.)
2.  Scan the AlmaLinux VM again, saving the output to a different file:

```
nmap -Pn --script=ssh2-enum-algos.nse 192.168.0.17 -oN alma9_future.txt
```

1.  Compare the two output files. You should now see fewer enabled algorithms than you did previously.

```
diff -y alma9_default.txt alma9_future.txt
```

1.  Look at the files in the `/etc/crypto-policies/back-ends/` directory:

```
ls -l /etc/crypto-policies/back-ends/
```

1.  You'll now see that the symbolic links point to files in the `FUTURE` directories.
2.  Look at the host keys in the `/etc/ssh/` directory, and see if they differ from what you had before:

```
ls -l /etc/ssh/*key*
```

1.  Scan the AlmaLinux 9 VM that you set up in `FIPS` mode for the lab in *Chapter 6*, *Encryption Technologies*. Compare the results with the `DEFAULT` and `FUTURE` mode scans.
2.  End of lab

> **Tip**
> 
> > If `FUTURE` mode hasn’t disabled enough algorithms for you, you can always just create your own custom policy. See the details here:
> 
> > [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening#customizing-system-wide-cryptographic-policies-with-subpolicies_using-the-system-wide-cryptographic-policies](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening#customizing-system-wide-cryptographic-policies-with-subpolicies_using-the-system-wide-cryptographic-policies)

You now know how to configure SSH to use only the most modern, most secure algorithms. Next, let's look at logging.

### Configuring more detailed logging

In its default configuration, SSH already creates log entries whenever someone logs in via SSH, SCP, or SFTP. On Debian/Ubuntu systems, the entry is made in the `/var/log/auth.log` file. On Red Hat/CentOS/AlmaLinux systems, the entry is made in the `/var/log/secure` file. Either way, the log entry looks something like this:

```
Oct  1 15:03:23 donnie-ca sshd[1141]: Accepted password for donnie from 192.168.0.225 port 54422 ssh2
Oct  1 15:03:24 donnie-ca sshd[1141]: pam_unix(sshd:session): session opened for user donnie by (uid=0)
```

Open the `sshd_config` man page and scroll down to the `LogLevel` item. There, you'll see the various settings that provide different levels of detail for logging SSH messages. The levels are as follows:

*   **QUIET**
*   **FATAL**
*   **ERROR**
*   **INFO**
*   **VERBOSE**
*   **DEBUG** or **DEBUG1**
*   **DEBUG2**
*   **DEBUG3**

Normally, the only two of these we would care about are `INFO` and `VERBOSE`. `INFO` is the default setting, while `VERBOSE` is the only other one that we would use under normal circumstances. The various `DEBUG` levels might come in handy for troubleshooting, but the man page warns us that using `DEBUG` in production settings would violate users' privacy.

Let's go ahead and get our hands dirty, just to get a feel for what gets logged with the various levels.

#### Hands-on lab – configuring more verbose SSH logging

For this lab, use the same VM that you've been using for the previous labs. That way, you'll get a better picture of what a complete `sshd_config` file should look like when it's fully locked down. Remotely log into the target VM via SSH and follow these steps:

1.  Open the main log file and scroll down to where you see the entry that was made due to your login and observe what it says. For Ubuntu, do:

```
sudo less /var/log/auth.log
```

For CentOS or AlmaLinux, do:

```
sudo less /var/log/secure
```

1.  As I mentioned previously, you never want to run a production machine with the SSH log level set to any of the `DEBUG` levels. But, just so you can see what it does log, set your machine to `DEBUG` now. Open the `/etc/ssh/sshd_config` file in your favorite text editor. Find the line that says the following:

```
#LogLevel INFO
```

Change it to the following:

```
LogLevel DEBUG3
```

1.  After saving the file, reload SSH. On Ubuntu, do:

```
sudo systemctl reload ssh
```

On CentOS or AlmaLinux, do this:

```
sudo systemctl reload sshd
```

1.  Log out of the SSH session, and then log back in. View the system log file to see the new entries from this new login.
2.  Open the `/etc/ssh/sshd_config` file for editing. Change the `LogLevel DEBUG3` line to the following:

```
LogLevel VERBOSE
```

1.  After saving the file, reload or restart the SSH daemon. Log out of the SSH session, log back in, and look at the entries in the system log file.

> **Tip**
> 
> > The main benefit of `VERBOSE` mode is that it will log the fingerprints of any key that was used to log in. This can be a big help with key management.

You've reached the end of the lab – congratulations!

So, you've just seen how to get more information about SSH logins in your system logs. Next, let's talk a bit about access control.

## Configuring access control with whitelists and TCP Wrappers

We've already locked things down pretty well just by requiring that clients authenticate via key exchange, rather than by username and password. When we prohibit password authentication, the bad guys can do brute-force password attacks against us until the cows come home, and it won't do them any good. (Although, in truth, they'll just give up as soon as they find that password authentication has been disabled.) For an extra measure of security, we can also set up a couple of access control mechanisms that will allow only certain users, groups, or client machines to log in to an SSH server. These two mechanisms are:

*   Whitelists within the `sshd_config` file
*   TCP Wrappers, via the `/etc/hosts.allow` and `/etc/hosts.deny` files

Okay, you're now saying, *But what about firewalls? Isn't that a third mechanism that we can use?* And yeah, you're right. But, we already covered firewalls in *Chapter 4*, *Securing Your Server with a Firewall - Part 1*, and *Chapter 5*, *Securing Your Server with a Firewall - Part 2*, so I won't repeat any of that here. You can also place access control directives in your systemd unit files for SSH. For our present discussion though, I’d rather avoid the complexities of explaining how to edit a systemd unit file. At any rate, these are the ways of controlling access to your SSH server. You can use all of them together if you really want to, or you can just use one of them at a time. (It really depends on just how paranoid you really are.)

> There are two competing philosophies about how to do access control. With blacklists, you specifically prohibit access by certain people or machines. That's difficult to do because the list could get very long, and you still won't block everybody that you need to block. The preferred and easier method is to use whitelists, which specifically allow access by certain people or machines.

First, let's look at creating whitelists within `sshd_config` with a hands-on lab.

### Configuring whitelists within sshd_config

The four access control directives that you can set within `sshd_config` are as follows:

*   **DenyUsers**
*   **AllowUsers**
*   **DenyGroups**
*   **AllowGroups**

For each directive, you can specify more than one username or group name, separating them with a blank space. Also, these four directives are processed in the order that I've listed them here. In other words, if a user is listed with both the `DenyUsers` and the `AllowUsers` directives, `DenyUsers` takes precedence. If a user is listed with `DenyUsers` and is a member of a group that's listed with `AllowGroups`, `DenyUsers` again takes precedence. To demonstrate this, let's do a lab.

#### Hands-on lab – configuring whitelists within sshd_config

This lab will work on any of your VMs. Follow these steps:

1.  On the VM that you wish to configure, create user accounts for Frank, Charlie, and Maggie. On Ubuntu, do it like this:

```
sudo adduser frank
```

On CentOS or AlmaLinux, do it like this:

```
sudo useradd frank
sudo passwd frank
```

1.  Create the `webadmins` group and add Frank to it:

```
sudo groupadd webadmins
sudo usermod -a -G webadmins frank
```

1.  From either your host machine or from another VM, have the three users log in. Then, log them back out.
2.  Open the `/etc/ssh/sshd_config` file in your favorite text editor. At the bottom of the file, add an `AllowUsers` line with your own username, like so:

```
AllowUsers donnie
```

1.  Then, restart or reload the SSH service and verify that it has started correctly:

```
For Ubuntu:
sudo systemctl restart ssh
sudo systemctl status ssh
For CentOS:
sudo systemctl restart sshd
sudo systemctl status sshd
```

1.  Repeat *step 3*. This time, these three kitties shouldn't be able to log in. Open the `/etc/ssh/sshd_config` file in your text editor. This time, add an `AllowGroups` line to the bottom of the file for the `webadmins` group, like so:

```
AllowGroups webadmins
```

1.  Restart the SSH service and verify that it started properly.

From either your host machine or another VM, have Frank try to log in. You'll see that even though he's a member of the `webadmins` group, he'll still be denied. That's because the `AllowUsers` line with your own username takes precedence.

1.  Open `sshd_config` in your text editor and remove the `AllowUsers` line that you inserted in *step 4*. Restart the SSH service and verify that it started properly.
2.  Try to log into your own account, and then try to log into the accounts of all the other users. You should now see that Frank is the only one who is allowed to log in. The only way that any of the other users can now log into the VM is from the VM's local console.
3.  Log into your own account at the VM's local console. Delete the `AllowGroups` line from `sshd_config` and restart the SSH service.

You've reached the end of the lab – congratulations!

You've just seen how to configure a whitelist on the daemon level, using the SSH daemon's own configuration file. Next, we'll look at configuring whitelists at the network level.

### Configuring whitelists with TCP Wrappers

It's a strange name, but a simple concept. TCP Wrappers – singular, not plural – listens to incoming network connections and either allows or denies connection requests. Whitelists and blacklists are configured in the `/etc/hosts.allow` file and the `/etc/hosts.deny` file. Both of these files work together. If you create a whitelist in `hosts.allow` without adding anything to `hosts.deny`, nothing will be blocked. That's because TCP Wrappers consults `hosts.allow` first, and if it finds a whitelisted item there, it will just skip over looking in `hosts.deny`. If a connection request comes in for something that isn't whitelisted, TCP Wrappers will consult `hosts.allow`, find that there's nothing there for the source of this connection request, and then will consult `hosts.deny`. If nothing is in `hosts.deny`, the connection request will still go through. So, after you configure `hosts.allow`, you have to also configure `hosts.deny` in order to block anything.

> You’ll want to note that the Red Hat folk have stripped TCP Wrappers from RHEL 8/9 and their offspring. So, if you decide to practice with the techniques that I present here, you can do so with either your Ubuntu or CentOS 7 VMs, but not on your AlmaLinux 8/9 VMs. (The Red Hat folk now recommend doing access control via firewalld, rather than TCP Wrappers.)
> 
> > You can read about it here: [https://access.redhat.com/solutions/3906701](https://access.redhat.com/solutions/3906701).
> 
> > (You'll need a Red Hat account to read the whole article. If you don't need to pay for Red Hat support, you can open a free-of-charge developers' account.)

Now, here's something that's extremely important. Always, *always*, configure `hosts.allow` before you configure `hosts.deny`. That's because as soon as you save either one of these files, the new configuration immediately takes effect. So, if you configure the blocking rule in `hosts.deny` while logged in remotely, your SSH connection will break just as soon as you save the file. The only way to get back in will be to enter the server room and reconfigure things from the local console. Your best bet is to get used to the idea of always configuring `hosts.allow` first, even when you're working from the local console. That way, you'll always be sure. (Amazingly, though, there are other TCP Wrappers tutorials out there that tell you to configure `hosts.deny` first. What *are* these guys thinking?)

You can do some rather fancy tricks with TCP Wrappers, but for now, I just want to keep things simple. So, let's look at some of the most-used configurations.

To whitelist a single IP address, place a line like this into the `/etc/hosts.allow` file:

```
SSHD: 192.168.0.225
```

Then, place this line into the `/etc/hosts.deny` file:

```
SSHD: ALL
```

Now, if you try to log in from anywhere else besides the IP address that's listed in `hosts.allow`, you will be denied access.

You can also list either multiple IP addresses or network addresses in `hosts.allow`. For details on how to do this, see the `hosts.allow` man page.

As I mentioned previously, you can do some fancy things with TCP Wrappers. But, now that the Red Hat folk have deprecated it, you should probably get used to the idea of either setting up firewall rules or configuring the `sshd_config` file. On the other hand, TCP Wrappers could come in handy whenever you need to configure an access control rule very quickly, provided that you’re on a machine that supports it.

## Configuring automatic logouts and security banners

Best security practice dictates that people log out of their computers before they walk away from their desks. This is especially important when an administrator uses his or her cubicle computer to remotely log into a sensitive server. By default, SSH allows a person to remain logged in forever without complaining. However, you can set it up to automatically log out idle users. We'll look at two quick methods for doing that.

### Configuring automatic logout for both local and remote users

This first method will automatically log out idle users who are logged on either at the local console or remotely via SSH. Go into the `/etc/profile.d/` directory and create the `autologout.sh` file with the following contents:

```
TMOUT=100
readonly TMOUT
export TMOUT
```

This sets a timeout value of 100 seconds. (`TMOUT` is a Linux environmental variable that sets timeout values.)

Set the executable permission for everybody:

```
sudo chmod +x autologout.sh
```

Log out and then log back in. Then, let the VM sit idle. After 100 seconds, you should see that the VM is back at the login prompt. Note, though, that if any users are already logged in at the time you create this file, the new configuration won't take effect for them until they log out and then log back in.

### Configuring automatic logout in sshd_config

The second method only logs out users who are logged in remotely via SSH. Instead of creating the `/etc/profile.d/autologout.sh` file, look for these two lines in the `/etc/ssh/sshd_config` file:

```
#ClientAliveInterval 0
#ClientAliveCountMax 3
```

Change them to the following:

```
ClientAliveInterval 100
ClientAliveCountMax 0
```

Then, restart or reload the SSH service to make the change take effect.

> I've been using 100 seconds for the timeout value in both of these examples. However, you can set the timeout value to suit your own needs.

You now know how to automatically log out your users. Now, let's look at setting up security banners.

### Creating a pre-login security banner

In *Chapter 3*, *Securing Normal User Accounts*, I showed you how to create a security message that shows up *after* a user has logged in. You do this by inserting a message into the `/etc/motd` file. But, when you think about it, wouldn't it be better for people to see a security banner *before* they log in? You can do that with `sshd_config`.

First, let's create the `/etc/ssh/sshd-banner` file, with the following contents:

```
Warning!!  Authorized users only.  All others will be prosecuted.
```

In the `/etc/ssh/sshd_config` file, look for this line:

```
#Banner none
```

Change it to this:

```
Banner /etc/ssh/sshd-banner
```

As always, restart or reload the SSH service. Now, whoever logs in remotely will see something like this:

```
[donnie@fedora-teaching ~]$ ssh donnie@192.168.0.3
Warning!!  Authorized users only.  All others will be prosecuted.
donnie@192.168.0.3's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-64-generic x86_64)
. . .
. . .
```

So, will this banner keep your system safe and secure from the bad guys? No, but it could be useful if you ever have to take a case to court. Sometimes, it's important to show a judge and jury that the intruders knew that they were going where they don't belong.

Now that you know how to set up security banners and automatic logouts, let's look at a few miscellaneous settings that don't fit neatly into any one category.

## Configuring other miscellaneous security settings

Our SSH configuration is a lot more secure than it used to be, but we can still make it better. Here are a few little tricks that you might not have seen elsewhere.

### Disabling X11 forwarding

When you SSH into a server in the normal manner, as we've been doing, you can only run text-mode programs. If you try to remotely run any GUI-based program, such as Firefox, you'll get an error message. But, when you open the `sshd_config` file of pretty much any Linux distribution, you'll see this line:

```
X11Forwarding yes
```

This means that with the right option switch, you can remotely run GUI-based programs. Assuming that you're logging into a machine that has a graphical desktop environment installed, you can use either the `-Y` or the `-X` option when logging in, like so:

```
ssh -X donnie@192.168.0.12
```

or

```
ssh -Y donnie@192.168.0.12
```

The problem here is that the X11 protocol, which powers graphical desktop environments on most Linux and Unix systems, has a few security weaknesses that make it somewhat dangerous to use remotely. The bad guys have ways of using it to compromise an entire system. Your best bet is to disable it by changing the `X11Forwarding` line to look like this:

```
X11Forwarding no
```

As usual, restart or reload the SSH service to make it read in the new configuration.

Now that you know about X11 forwarding, let's dig some tunnels.

### Disabling SSH tunneling

SSH tunneling, or as it's sometimes called, SSH port forwarding, is a handy way to protect non-secure protocols. For example, by tunneling normal HTTP through an SSH tunnel, you can access a non-secure website in a secure fashion. Here’s what that looks like:

```
sudo ssh -L 80:localhost:80 donnie@192.168.0.12
```

I had to use `sudo` here because all network ports below port `1024` are **privileged ports**. If I were to change the web server configuration to listen on a non-privileged high-number port, I wouldn't need `sudo`.

Now, to connect to this site in a secure manner, I can just open the web browser on my local machine and type in the following URL:

`http://localhost`

Yeah, it seems strange to access a remote machine by typing in `localhost`, but that's the designator I used when I logged in with SSH. I could have used another name, but `localhost` is the name you traditionally see in SSH tutorials, so I'm following suit here. Now, as soon as I log out of the SSH session, my connection to the web server will break.

Even though this sounds like a good idea, it actually creates a security problem. Let's say that your corporate firewalls are set up to prevent people from going home and remotely logging into their company workstations. That's a good thing, right? Now, let's say that the company firewall has to allow outbound SSH connections. A user could create an SSH tunnel from his or her company workstation to a computer at another location, then go to that location and create a reverse tunnel back to the company workstation. So, if it isn't possible to block outgoing SSH traffic at the firewall, then your best bet is to disable SSH tunneling. In your `sshd_config` file, ensure that you have lines that look like this:

```
AllowTcpForwarding no
AllowStreamLocalForwarding no
GatewayPorts no
PermitTunnel no
```

Restart or reload the SSH service, as always. Now, port tunneling will be disabled.

Now that you know how to disable SSH tunneling, let's talk about changing the default port.

### Changing the default SSH port

By default, SSH listens on port `22/TCP`. If you've been around for a while, you've surely seen plenty of documentation about how important it is to use some other port in order to make it harder for the bad guys to find your SSH server. But, I must say, this notion is a bit controversial.

In the first place, if you enable key authentication and disable password authentication, then changing the port has limited value. When a scanner bot finds your server and sees that password authentication is disabled, it will just go away and won't bother you anymore. In the second place, if you were to change the port, the bad guys' scanning tools can still find it. If you don't believe me, just go to Shodan.io and search for `ssh`. In this example, someone thought they were smart by changing to port `2211`:

![](img/file50.png)

*Yeah, smarty-pants. That didn't hide things so well, now, did it?*

On the other hand, security expert Daniel Miessler says that it's still useful to change the port, in case someone tries to leverage a zero-day exploit against SSH. He recently published the results of an informal experiment that he did, in which he set up a public server that listens for SSH connections to both port `22` and port `24`, and observed the number of connection attempts to each port. He said that over a single weekend, there were 18,000 connections to port `22` and only five to port `24`. But, although he doesn't explicitly say, it appears that he left password authentication enabled. To have truly scientifically accurate results, he needs to conduct the same study with password authentication disabled. He also needs to conduct the study on separate servers that have SSH enabled for either port `22` or port `24`, instead of having both ports enabled on a single machine. My hunch is that when the scanner bots found that port `22` was open, they didn't bother to scan for any other open SSH ports.

> You can read about his experiment here: [https://danielmiessler.com/study/security-by-obscurity/](https://danielmiessler.com/study/security-by-obscurity/).

Anyway, if you do want to change ports, just uncomment the `#Port 22` line in `sshd_config`, and change the port number to whatever you want.

Next, let's talk about key management.

### Managing SSH keys

Earlier, I showed you how to create a pair of keys on your local workstation, and then transfer the public key to a remote server. This allows you to disable username/password authentication on the server, making it much harder for the bad guys to break in. The only problem with this that we didn't address is that the public key goes into an `authorized_keys` file that's in the user's own home directory. So, the user can manually add extra keys to the file, which would allow the user to log in from other locations besides the one that's been authorized. And, there's also the problem of having `authorized_keys` files all over the place, in every user's home directory. That makes it a bit hard to keep track of everyone's keys.

One way to handle this is to move everyone's `authorized_keys` file to one central location. Let's take Vicky, my 15-year old solid gray kitty. The administrator created an account on the server that she needs to access and allowed her to create and transfer her key to it before disabling password authentication. So, Vicky now has her `authorized_keys` file in her home directory on that server, as we see here:

```
vicky@ubuntu-nftables:~$ cd .ssh
vicky@ubuntu-nftables:~/.ssh$ ls -l
total 4
-rw------- 1 vicky vicky 233 Oct  3 18:24 authorized_keys
vicky@ubuntu-nftables:~/.ssh$
```

Vicky owns the file, and she has both read and write permissions on it. So, even though she can't transfer other keys to it the normal way once the administrator has disabled password authentication, she can still transfer key files manually, and manually edit the `authorized_keys` file to include them. To thwart her efforts, our intrepid administrator will create a directory within the `/etc/ssh/` directory to hold everyone's `authorized_keys` files, like so:

```
sudo mkdir /etc/ssh/authorized-keys
```

Our intrepid administrator's full admin privileges allow him to log into the root user's shell, which allows him to go into the directories of all other users:

```
donnie@ubuntu-nftables:~$ sudo su -
[sudo] password for donnie: 
root@ubuntu-nftables:~# cd /home/vicky/.ssh
root@ubuntu-nftables:/home/vicky/.ssh# ls -l
total 4
-rw------- 1 vicky vicky 233 Oct 3 18:24 authorized_keys
root@ubuntu-nftables:/home/vicky/.ssh#
```

The next step is to move Vicky's `authorized_keys` file to the new location, changing its name to `vicky`, like so:

```
root@ubuntu-nftables:/home/vicky/.ssh# mv authorized_keys /etc/ssh/authorized-keys/vicky
root@ubuntu-nftables:/home/vicky/.ssh# exit
donnie@ubuntu-nftables:~$
```

Now, we have a bit of a conundrum. As you see here, the file still belongs to Vicky, and she has both read and write privileges. So, she can still edit the file without any administrator privileges. Removing the write privilege won't work, because since the file belongs to her, she could just add the write privilege back. Changing ownership to the root user is part of the answer, but that will prevent Vicky from being able to read the file, which will prevent her from logging in. To see the whole solution, let's see what I've already done with my own `authorized_keys` file:

```
donnie@ubuntu-nftables:~$ cd /etc/ssh/authorized-keys/
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$ ls -l
total 8
-rw------- 1 vicky vicky 233 Oct 3 18:24 vicky
-rw-r-----+ 1 root root 406 Oct 3 16:24 donnie
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$
```

The eagle-eyed among you have surely noticed what's going on with the `donnie` file. You have seen that I changed ownership to the root user and then added an access control list, as indicated by the `+` sign. Let's do the same for Vicky:

```
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$ sudo chown root: vicky
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$ sudo setfacl -m u:vicky:r vicky 
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$
```

Looking at the permissions settings, we see that Vicky has read access to the `vicky` file:

```
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$ ls -l 
total 8
-rw-r-----+ 1 root root 406 Oct 3 16:24 donnie
-rw-r-----+ 1 root root 233 Oct 3 18:53 vicky
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$
```

While we're at it, let's look at her access control list:

```
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$ getfacl vicky
# file: vicky
# owner: root
# group: root
user::rw-
user:vicky:r--
group::---
mask::r--
other::---
donnie@ubuntu-nftables:/etc/ssh/authorized-keys$
```

Vicky can now read the file so that she can log in, but she can't change it.

The final step is to reconfigure the `sshd_config` file, and then restart or reload the SSH service. Open the file in your text editor and look for this line:

```
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
```

Change it to this:

```
AuthorizedKeysFile      /etc/ssh/authorized-keys/%u
```

The `%u` at the end of the line is a mini-macro that tells the SSH service to look for a keys file that has the same name as the user who's logging in. Now, even if the users were to manually create their own `authorized_keys` files in their own home directories, the SSH service would just ignore them. Another benefit is that having the keys all in one place makes it a bit easier for an administrator to revoke someone's access, should the need arise.

Be aware that there's a lot more to managing SSH keys than what I've been able to present here. One problem is that while there are a few different free open source software solutions for managing public keys, there aren't any for managing private keys. A large corporation could have thousands or perhaps even millions of private and public keys in different places. Those keys never expire, so they'll be around forever unless they get deleted. If the wrong people get hold of a private key, your whole system could become compromised. As much as I hate to say it, your best bet for managing SSH keys is to go with a commercial solution, such as ones from SSH.com and CyberArk.

> Check out the key management solutions from SSH.com here: [https://www.ssh.com/academy/iam/ssh-key-management](https://www.ssh.com/academy/iam/ssh-key-management).
> 
> > Head here for CyberArk's key management solutions: [https://www.cyberark.com/resources/blog/ssh-keys-the-powerful-unprotected-privileged-credentials](https://www.cyberark.com/resources/blog/ssh-keys-the-powerful-unprotected-privileged-credentials).
> 
> > Full disclosure: I have no connection with either SSH.com or CyberArk, and receive no payment for telling you about them.

You've learned several cool tricks here for beefing up your server security. Now, let's look at how to create different configurations for different users and groups.

### Setting different configurations for different users and groups

On the server side, you can use the `Match User` or `Match Group` directive to set up custom configurations for certain users or groups. To see how it's done, look at the example at the very bottom of the `/etc/ssh/sshd_config` file. There, you'll see the following:

```
# Match User anoncvs
# X11Forwarding no
# AllowTcpForwarding no
# PermitTTY no
# ForceCommand cvs server
```

Of course, this has no effect since it's commented out, but that's okay. Here's what we see for user `anoncvs`:

*   He can't do **X11 Forwarding**.
*   He can't do **TCP Forwarding**.
*   He won't have the use of a command terminal.

As soon as he logs in, he'll be starting the **Concurrent Versioning Service** (**CVS**) server. By not having use of the terminal, `anoncvs` can start the CVS server, but can't do anything else.

You can set up different configurations for as many users as you need to. Anything that you put in the custom configurations will override the global settings. To set up a custom configuration for a group, just replace `Match User` with `Match Group`, and supply a group name instead of a user name.

### Creating different configurations for different hosts

For a change of pace, let's look at the client's end now. This time, we'll look at a handy trick to help ease the pain of logging into different servers that require different keys or SSH options. All you have to do is go into the `.ssh` directory in your own home directory and create a `config` file. To demonstrate this, let's say that we've created either a DNS record or an `/etc/hosts` file entry for our servers so that we don't have to remember so many IP addresses. Let's also say that we've created a separate pair of keys for each server that we need to access. In the `~/.ssh/config` file, we can add a stanza that looks something like this:

```
Host ubuntu-nftables
 IdentityFile ~/.ssh/unft_id_rsa
 IdentitiesOnly yes
 ForwardX11 yes
 Cipher aes256-gcm@openssh.com
```

Here's the breakdown:

*   **IdentityFile**: This specifies the key that goes with this server.
*   **IdentitiesOnly yes**: If you happen to have more than one key loaded into your session keyring, this forces your client to only use the key that's specified here.
*   **ForwardX11 yes**: We want this client to use *X11* forwarding. (Of course, this will only be effective if the server has been configured to allow it.)
*   **Cipher aes256-gcm@openssh.com**: We want to use this algorithm, and *only* this algorithm, to perform our encryption.

To create custom configurations for other hosts, just add a stanza for each one to this file.

After you save the file, you have to change its permissions settings to a value of `600`. If you don't, you'll get an error when you try to log into any of the servers that are configured in the file.

Now that you know about custom configurations, let's talk about SFTP, where we'll make good use of the `Match Group` directive that we just looked at.

## Setting up a chroot environment for SFTP users

**Secure File Transfer Protocol** (**SFTP**) is a great tool for performing secure file transfers. There is a command-line client, but users will most likely use a graphical client, such as FileZilla. With a default SSH setup, anyone who has a user account on a Linux machine can log in through either SSH or SFTP and can navigate through the server's entire filesystem. What we really want for SFTP users is to prevent them from logging into a command prompt via SSH, and to confine them to their own designated directories.

> **Tip**
> 
> > One good use for this trick would be to set up SFTP configurations for web site creators. Instead of allowing these users to transfer files to and from only their own home directories, just allow them to transfer files to and from the web site content directories.

### Creating a group and configuring the sshd_config file

With the exception of the slight difference in user-creation commands, this procedure works the same on any of your VMs.

We'll begin by creating an `sftpusers` group:

```
sudo groupadd sftpusers
```

Create the user accounts and add them to the `sftpusers` group. We'll do both operations in one step. On your CentOS or AlmaLinux machine, the commands for creating Max's account would look like this:

```
sudo useradd -G sftpusers max
sudo passwd max
```

On your Ubuntu machine, it would look like this:

```
sudo useradd -m -d /home/max -s /bin/bash -G sftpusers max
```

Open the `/etc/ssh/sshd_config` file in your favorite text editor. Find the line that says this:

```
Subsystem sftp /usr/lib/openssh/sftp-server
```

Change it to this:

```
Subsystem sftp internal-sftp
```

This setting allows you to disable normal SSH login for certain users.

At the bottom of the `sshd_config` file, add a `Match Group` stanza:

```
Match Group sftpusers
        ChrootDirectory /home
        AllowTCPForwarding no
        AllowAgentForwarding no
        X11Forwarding no
        ForceCommand internal-sftp
```

An important consideration here is that the `ChrootDirectory` has to be owned by the root user, and it can't be writable by anyone other than the root user. When Max logs in, he'll be in the `/home/` directory, and will then have to `cd` into his own directory. This also means that you want all your users' home directories to have the restrictive `700` permissions settings, in order to keep everyone out of everyone else's stuff.

Save the file and restart the SSH daemon. Then, try to log on as Max through normal SSH, just to see what happens:

```
donnie@linux-0ro8:~> ssh max@192.168.0.8
max@192.168.0.8's password:
This service allows sftp connections only.
Connection to 192.168.0.8 closed.
donnie@linux-0ro8:~>
```

Okay, so he can't do that. Now, let's have Max try to log in through SFTP and verify that he is in the `/home/` directory:

```
donnie@linux-0ro8:~> sftp max@192.168.0.8
max@192.168.0.8's password:
Connected to 192.168.0.8.
drwx------    7 1000     1000         4096 Nov  4 22:53 donnie
drwx------    5 1001     1001         4096 Oct 27 23:34 frank
drwx------    3 1003     1004         4096 Nov  4 22:43 katelyn
drwx------    2 1002     1003         4096 Nov  4 22:37 max
sftp>
```

Now, let's see him try to `cd` out of the `/home/` directory:

```
sftp> cd /etc
Couldn't stat remote file: No such file or directory
sftp>
```

So, our chroot jail does indeed work.

#### Hands-on lab – setting up a chroot directory for the sftpusers group

For this lab, you can use either the CentOS VM or the Ubuntu VM. You'll add a group, then configure the `sshd_config` file to allow group members to only be able to log in via SFTP, and then confine them to their own directories. For the simulated client machine, you can use the terminal of your macOS or Linux desktop machine, or any of the available Bash shells from your Windows machine. Let's get started:

1.  Create the `sftpusers` group:

```
sudo groupadd sftpusers
```

1.  Create a user account for Max and add him to the `sftpusers` group. On CentOS or AlmaLinux, do this:

```
sudo useradd -G sftpusers max
sudo passwd max
```

On Ubuntu, do this:

```
sudo useradd -m -d /home/max -s /bin/bash -G sftpusers max
```

1.  For Ubuntu, ensure that the users' home directories are all set with read, write, and execute permissions for only the directory's user. If that's not the case, do this:

```
sudo chmod 700 /home/*
```

1.  Open the `/etc/ssh/sshd_config` file in your preferred text editor. Find the line that says the following:

```
Subsystem sftp /usr/lib/openssh/sftp-server
```

Change it to the following:

```
Subsystem sftp internal-sftp
```

1.  At the end of the `sshd_config` file, add this stanza:

```
Match Group sftpusers
     ChrootDirectory /home
     AllowTCPForwarding no
     AllowAgentForwarding no
     X11Forwarding no
     ForceCommand internal-sftp
```

1.  Reload the SSH configuration. On CentOS or AlmaLinux, do this:

```
sudo systemctl reload sshd
```

On Ubuntu, do this:

```
sudo systemctl reload ssh
```

1.  Have Max try to log in through normal SSH, to see what happens:

```
ssh max@IP_Address_of_your_vm
```

1.  Now, have Max log in through SFTP. Once he's in, have him try to `cd` out of the `/home/` directory:

```
sftp max@IP_Address_of_your_vm
```

You've reached the end of the lab – congratulations!

Now that you know how to securely configure SFTP, let's look at how to securely share a directory.

## Sharing a directory with SSHFS

There are several ways to share a directory across a network. In enterprise settings, you'll find the **Network Filesystem** (**NFS**), **Samba**, and various distributed filesystems. **SSHFS** isn't used in enterprises quite as much, but it can still come in handy. The beauty of it is that all of its network traffic is encrypted by default, unlike with NFS or Samba. And, other than installing the SSHFS client program and creating a local mount-point directory, it doesn't require any configuration beyond what you've already done. It's especially handy for accessing a directory on a cloud-based **Virtual Private Server** (**VPS**) because it allows you to just create files in the shared directory rather than using `scp` or `sftp` commands to transfer the files. So, if you're ready, let's jump in.

### Hands-on lab – sharing a directory with SSHFS

For this lab, we'll use two VMs. For the server, you can use any of your virtual machines. The same is true of the client, except that each distro has the the SSHFS client in a different repository. Here’s what I’m talking about:

*   The client is in the normal Ubuntu repositories, so you don’t have to do anything special to get it.
*   For CentOS 7 and AlmaLinux 9, you’ll need to install the `epel-release` package with the normal `yum install` or `dnf install` command.
*   AlmaLinux 8 has the SSHFS client in its own PowerTools repository, which isn’t enabled by default. To enable it, open the `/etc/yum.repos.d/almalinux-powertools.repo` file in your favorite text editor. In the `[powertools]` section, find the line that says `enabled=0`, and change it to `enabled=1`.

Now that we have all that straight, let's get started:

1.  Boot up one VM for a server. (That's all you need to do for the server end.)
2.  On the other VM that you'll use as a client, create a mount-point directory within your own home directory, like so:

```
mkdir remote
```

1.  On the client VM, install the SSHFS client. On Ubuntu, do this:

```
sudo apt install sshfs
```

On CentOS 7, do this:

```
sudo yum install fuse-sshfs
```

On AlmaLinux 8 or 9, do this:

```
sudo dnf install fuse-sshfs
```

1.  From the client machine, mount your own home directory that's on the server:

```
sshfs donnie@192.168.0.10: /home/donnie/remote
```

> Note that if you don't specify a directory to share, the default is to share the home directory of the user account that's being used for logging in.

1.  Verify that the directory was mounted properly with the `mount` command. You should see your new shared mount at the bottom of the output:

```
donnie@ubuntu-nftables:~$ mount
. . .
. . .
donnie@192.168.0.10: on /home/donnie/remote type fuse.sshfs (rw,nosuid,nodev,relatime,user_id=1000,group_id=1004)
```

1.  `cd` into the `remote` directory and create some files. Verify that they actually do show up on the server.
2.  At the local console of the server VM, create some files in your own home directory. Verify that they show up in the `remote/` directory of your client VM.

You've reached the end of the lab – congratulations!

With this lab, I just showed you how to mount your own home directory from a remote server. You can also mount other server directories by specifying them in the `sshfs` command. For example, let's say that I want to mount the `/maggie_files/` directory, with the `~/remote3/` directory as my local mount-point. (I chose that name because Maggie cat is sitting here in front of me where my keyboard should be.) Just do it like this:

```
sshfs donnie@192.168.0.53:/maggie_files /home/donnie/remote3
```

You can also make the remote directory automatically mount every time you boot your client machine by adding an entry to the `/etc/fstab` file. But, that's generally not a good idea. If the server isn't available when you boot the client machine, it could cause the boot process to hang up.

Okay, so you've seen how to use SSHFS to create an encrypted connection with a shared remote directory. Let's now log into the server from a Windows desktop machine.

## Remotely connecting from Windows desktops

I know, all of us Penguinistas would like to use Linux, and nothing but Linux. But, in an enterprise environment, things just don't always work that way. There, you'll most likely have to administer your Linux servers from a Windows 10/11 desktop machine that's sitting on your cubicle desk. In *Chapter 1*, *Running Linux in a Virtual Environment*, I showed you how to use either Cygwin or the new Windows 10/11 shell to remotely connect to your Linux VMs. You can also use these techniques to connect to actual Linux servers.

But, some shops require that admins use a terminal program, rather than a full-blown Bash Shell such as Cygwin. Normally, these shops will require that you use **PuTTY** on your Windows machine.

> PuTTY is a free program that you can download from here: [https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

Installation is simple. Just double-click the installer file and follow through the installer screens:

![](img/file51.png)

You can open the PuTTY user manual from your Windows 10/11 Start menu:

![](img/file52.png)

Connecting to a remote Linux machine is easy. Just enter the machine's IP address and click on **Open**:

![](img/file53.png)

Note that this also gives you the option to save your sessions. So, if you have to administer multiple servers, you can open PuTTY and just click on the name of the server that you want to connect to, and then click on **Open**:

![](img/file54.png)

As you can see, this is a lot handier than having to manually type in the `ssh` command every time you need to log into a server, and it prevents you from having to remember a whole list of IP addresses for multiple servers. (But of course, you can accomplish the same thing with either Cygwin or a Windows 10 shell by creating a login shell script for each Linux machine that you need to administer.)

Either way, you'll end up at the remote machine's Bash Shell:

![](img/file55.png)

To set up key-exchange authentication, use PuTTYgen to create the key pair. The only slight catch is that you'll have to transfer the public key to the server by manually copying and pasting the key into the server's `authorized_keys` file:

![](img/file56.png)

I've given you the basics about PuTTY. You can read the PuTTY manual to get the nitty-gritty details.

Okay, I think that that about wraps things up for our discussion of the Secure Shell suite.

## Summary

In this chapter, we've seen that a default configuration of Secure Shell isn't as secure as we'd like it to be, and we've seen what to do about it. We've looked at how to set up key-based authentication and two-factor authentication, and we've looked at lots of different options that can lock down the SSH server. We also looked at how to disable weak encryption algorithms, and at how the new system-wide crypto policies on RHEL 8/CentOS 8 and RHEL 9/AlmaLinux 9 make doing that really easy. Along the way, we looked at setting up access controls, and at creating different configurations for different users, groups, and hosts. After demoing how to confine SFTP users to their own home directories, we used SSHFS to share a remote directory. We wrapped up this chapter by presenting a handy way to log into our Linux servers from a Windows desktop machine.

Conspicuous by their absence are a couple of technologies that you may have seen recommended elsewhere. Port knocking and Fail2Ban are two popular technologies that can help control access to an SSH server. However, they're only needed if you allow password-based authentication to your SSH server. If you set up key-based authentication, as I've shown you here, you won't need the added complexity of those other solutions.

In the next chapter, we'll take an in-depth look at the subject of discretionary access control. I'll see you there.

## Questions

1.  Which of the following statements is true?

A. Secure Shell is completely secure in its default configuration.

B. It's safe to allow the root user to use Secure Shell to log in across the Internet.

C. Secure Shell is insecure in its default configuration.

D. The most secure way to use Secure Shell is to log in with a username and password.

1.  Which three of the following things would you do to conform with the best security practices for Secure Shell?

A. Make sure that all users are using strong passwords to log in via Secure Shell.

B. Have all users create a public/private key pair, and transfer their public keys to the server to which they want to log in.

C. Disable the ability to log in via username/password.

D. Ensure that the root user is using a strong password.

E. Disable the root user's ability to log in.

1.  Which one of the following lines in the `sshd_config` file will cause botnets to not scan your system for login vulnerabilities?

A. `PasswordAuthentication no`

B. `PasswordAuthentication yes`

C. `PermitRootLogin yes`

D. `PermitRootLogin no`

1.  How would you confine a user of SFTP to his or her own specified directory?

A. Ensure that proper ownership and permissions are set on that user's directory.

B. In the `sshd_config` file, disable that user's ability to log in via normal SSH and define a `chroot` directory for that user.

C. Define the user's limitations with TCP Wrappers.

D. Use whole-disk encryption on the server so that SFTP users will only be able to access their own directories.

1.  Which two of the following commands would you use to add your private SSH key to your session keyring?

A. `ssh-copy-id`

B. `exec /usr/bin/ssh-agent`

C. `exec /usr/bin/ssh-agent $SHELL`

D. `ssh-agent`

E. `ssh-agent $SHELL`

F. `ssh-add`

1.  Which of the following is *not* on NIST's list of recommended algorithms?

A. `RSA`

B. `ECDSA`

C. `Ed25519`

1.  Which of the following is the correct directive for creating a custom configuration for Katelyn?

A. `User Match katelyn`

B. `Match katelyn`

C. `Match Account katelyn`

D. `Match User katelyn`

1.  When creating a `~/.ssh/config` file, what should the permissions value on that file be?

A. `600`

B. `640`

C. `644`

D. `700`

1.  Which of the following crypto policies provides the strongest encryption on RHEL 8/9-type distros?

A. `LEGACY`

B. `FIPS`

C. `DEFAULT`

D. `FUTURE`

1.  Which of the following standards defines NIST's current recommendations for encryption algorithms?

A. FIPS 140-2

B. FIPS 140-3

C. CNSA

D. Suite B

## Further reading

*   How to set up SSH keys on Debian 10 Buster: [https://devconnected.com/how-to-set-up-ssh-keys-on-debian-10-buster/](https://devconnected.com/how-to-set-up-ssh-keys-on-debian-10-buster/)
*   How to configure the OpenSSH Server: [https://www.ssh.com/academy/ssh/sshd_config](https://www.ssh.com/academy/ssh/sshd_config)
*   Setting up passwordless SSH: [https://www.redhat.com/sysadmin/passwordless-ssh](https://www.redhat.com/sysadmin/passwordless-ssh)
*   OpenSSH best practices for Unix, Linux, and BSD: [https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html](https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html)
*   Different SSH configurations for different hosts: [https://www.putorius.net/how-to-save-per-user-per-host-ssh-client-settings.html](https://www.putorius.net/how-to-save-per-user-per-host-ssh-client-settings.html)
*   SSH Query at Shodan: [https://www.shodan.io/search?query=ssh](https://www.shodan.io/search?query=ssh)
*   Mozilla OpenSSH Security Guide: [https://infosec.mozilla.org/guidelines/openssh](https://infosec.mozilla.org/guidelines/openssh)
*   Execute commands on a remote system over SSH: [https://www.2daygeek.com/execute-run-linux-commands-remote-system-over-ssh/](https://www.2daygeek.com/execute-run-linux-commands-remote-system-over-ssh/)
*   CNSA Suite and Quantum Cryptography: [https://cryptome.org/2016/01/CNSA-Suite-and-Quantum-Computing-FAQ.pdf](https://cryptome.org/2016/01/CNSA-Suite-and-Quantum-Computing-FAQ.pdf)
*   FIPS 140-3: [https://csrc.nist.gov/projects/fips-140-3-transition-effort](https://csrc.nist.gov/projects/fips-140-3-transition-effort)
*   ChaCha20 and Poly1305: [https://tools.ietf.org/html/rfc7539](https://tools.ietf.org/html/rfc7539)
*   System-wide cryptographic policies on Red Hat Enterprise Linux 8: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening#system-wide-crypto-policies_using-the-system-wide-cryptographic-policies](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening#system-wide-crypto-policies_using-the-system-wide-cryptographic-policies)
*   How to log out inactive users in Linux: [https://www.ostechnix.com/auto-logout-inactive-users-period-time-linux/](https://www.ostechnix.com/auto-logout-inactive-users-period-time-linux/)
*   Configure host-specific SSH settings: [https://www.putorius.net/how-to-save-per-user-per-host-ssh-client-settings.html](https://www.putorius.net/how-to-save-per-user-per-host-ssh-client-settings.html)
*   How to use SSHFS to mount remote directories over SSH: [https://linuxize.com/post/how-to-use-sshfs-to-mount-remote-directories-over-ssh/](https://linuxize.com/post/how-to-use-sshfs-to-mount-remote-directories-over-ssh/)

## Answers

1.  C
2.  B, C, E
3.  A
4.  B
5.  C, F
6.  C
7.  D
8.  A
9.  D
10.  C