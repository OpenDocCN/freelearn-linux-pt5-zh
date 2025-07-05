# 6 Encryption Technologies

## Join our book community on Discord

[https://packt.link/SecNet](https://packt.link/SecNet)

![](img/file36.png)

You may work for a super-secret government agency, or you may be just a regular Joe or Jane citizen. Either way, you still have sensitive data that you need to protect from prying eyes. Business secrets, government secrets, personal secrets—it doesn't matter; it all needs protection. Locking down users home directories with restrictive permissions settings, as we saw in *Chapter 3*, *Securing Normal User Accounts*, is only part of the puzzle; we also need encryption. This encryption will provide three things for us:

*   **Confidentiality**: This ensures that only people who are authorized to see the data can see it.
*   **Integrity**: This ensures that the original data hasn't been altered by unauthorized people.
*   **Availability**: This ensures that sensitive data is always available, and can't be deleted by unauthorized people.

The two general types of data encryption that we'll look at in this chapter are meant to protect data at rest and data in transit. We'll begin with using file, partition, and directory encryption to protect data at rest. We'll wrap up with a look at using OpenSSL to protect data in transit.

In this chapter, we'll cover the following topics:

```
GNU Privacy Guard (GPG)
Encrypting partitions with Linux Unified Key Setup (LUKS)
```

*   Encrypting directories with eCryptfs
*   Using VeraCrypt for the cross-platform sharing of encrypted containers
*   OpenSSL and the Public Key Infrastructure
*   Commercial certificate authorities
*   Creating keys, certificate requests, and certificates
*   Creating an on-premises certificate authority
*   Adding a certificate authority to an operating system
*   OpenSSL and the Apache web server
*   Setting up mutual authentication

If you’re ready to get cryptic, let’s get started.

## GNU Privacy Guard (GPG)

We'll begin with **GNU Privacy Guard** (**GPG**). This is a free open source implementation of Phil Zimmermann's Pretty Good Privacy, which he created back in 1991\. You can use either one of them to either encrypt or cryptographically sign files or messages. In this section, we'll focus strictly on GPG.

There are some advantages of using GPG:

*   It uses strong, hard-to-crack encryption algorithms.
*   It uses the private/public key scheme, which eliminates the need to transfer a password to a message or file recipient in a secure manner. Instead, just send along your public key, which is useless to anyone other than the intended recipient.
*   You can use GPG to just encrypt your own files for your own use, the same as you'd use any other encryption utility.
*   It can be used to encrypt email messages, allowing you to have true end-to-end encryption for sensitive emails.
*   There are a few GUI-type frontends available to make it somewhat easier to use.

But, as you might know, there are also some disadvantages:

*   Using public keys instead of passwords is great when you work directly only with people who you implicitly trust. But for anything beyond that, such as distributing a public key to the general population so that everyone can verify your signed messages, you're dependent upon a web-of-trust model that can be very hard to set up.
*   For the end-to-end encryption of email, the recipients of your email must also have GPG set up on their systems and know how to use it. That might work in a corporate environment, but lots of luck getting your friends to set that up. (I've never once succeeded in getting someone else to set up email encryption.)
*   If you use a standalone email client, such as Mozilla Thunderbird, you can install a plugin that will encrypt and decrypt messages automatically. But every time a new Thunderbird update is released, the plugin breaks, and it always takes a while before a new working version gets released.
*   Even if you could get other people to set up their email clients with GPG, it’s still not the perfect privacy solution. That’s because the email **metadata**--the email addresses of the sender and the recipient--can’t be encrypted. So, hackers, advertisers, or government agencies can still see who you’re exchanging email messages with, and use that information to build a profile that tells them a lot about your activities, your beliefs, and what kind of a person you are. If you really need complete privacy, your best bet is to go with a private messenger solution, such as the **Session** messenger. (That however, is beyond the scope of this book.)

Even with its numerous weaknesses, GPG is still one of the best ways to share encrypted files and emails. GPG comes preinstalled on most Linux distros. So, you can use any of your *newer* virtual machines for these demos. (I say *newer*, because the procedure will differ slightly on older distros, such as CentOS 7.)

### Hands-on lab – creating your GPG keys

1.  On a text-mode AlmaLinux machine, the first thing you need to do is to install the `pinentry` package. Do that with:

```
sudo dnf install pinentry
```

(Note that you won’t have to do this with either a GUI-mode AlmaLinux machine or with Ubuntu Server.)

1.  Next, create your pair of GPG keys:

```
gpg --full-generate-key
```

Note that since you're setting this up for yourself, you don't need `sudo` privileges.

The first thing that this command does is to create a populated `.gnupg` directory in your home directory:

```
gpg: /home/donnie/.gnupg/trustdb.gpg: trustdb created
gpg: key 56B59F39019107DF marked as ultimately trusted
gpg: directory '/home/donnie/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/donnie/.gnupg/openpgp-revocs.d/BD057E0E01E664424E8B812E56B59F39019107DF.rev'
public and secret key created and signed.
```

You'll then be asked to select which kinds of keys you want. We'll just go with the default `RSA and RSA`. RSA keys are stronger and harder to crack than the older DSA keys. Elgamal keys are good, but they may not be supported by older versions of GPG:

```
Please select what kind of key you want:
 (1) RSA and RSA (default)
 (2) DSA and Elgamal
 (3) DSA (sign only)
 (4) RSA (sign only)
(14) Existing key from card
Your selection?
```

For decent encryption, you'll want to go with a key of at least 3,072 bits, because anything smaller is now considered vulnerable. (This is according to the newest guidance from the U.S. National Institute of Standards and Technology, or NIST.) That’s now the default on our newest Linux distros, so you’re already good there. On older distros, such as CentOS 7, the default is only 2048 bits, so you’ll need to change it.

Next, select how long you want the keys to remain valid before they automatically expire. For our purposes, we'll go with the default `key does not expire`:

```
Please specify how long the key should be valid. 
 0 = key does not expire 
 <n> = key expires in n days 
 <n>w = key expires in n weeks 
 <n>m = key expires in n months 
 <n>y = key expires in n years 
Key is valid for? (0) 
```

Provide your personal information:

```
GnuPG needs to construct a user ID to identify your key. 
Real name: Donald A. Tevault 
Email address: donniet@something.net 
Comment: No comment 
You selected this USER-ID: 
 "Donald A. Tevault (No comment) <donniet@something.net>" 
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
Create a passphrase for your private key:
You need a Passphrase to protect your secret key. 
We need to generate a lot of random bytes. It is a good idea to perform some other action (type on the keyboard, move the mouse, utilize the disks) during the prime generation; this gives the random number generator a better chance to gain enough entropy. 
```

On older Linux distros, this could take a while, even when you're doing all of the recommended things to create entropy. On newer Linux distros, the random number generator works more efficiently, so you can disregard the notice about how the key generation could take a long time. Here’s what you’ll see when the process has finished:

```
gpg: /home/donnie/.gnupg/trustdb.gpg: trustdb created 
gpg: key 19CAEC5B marked as ultimately trusted 
public and secret key created and signed. 
gpg: checking the trustdb 
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model 
gpg: depth: 0 valid: 1 signed: 0 trust: 0-, 0q, 0n, 0m, 0f, 1u 
pub 2048R/19CAEC5B 2017-10-26 
 Key fingerprint = 8DE5 8894 2E37 08C4 5B26 9164 C77C 6944 19CA EC5B 
uid Donald A. Tevault (No comment) <donniet@something.net> 
sub 2048R/37582F29 2017-10-26
```

1.  Verify that the keys did get created:

```
[donnie@localhost ~]$ gpg --list-keys
 /home/donnie/.gnupg/pubring.gpg
 -------------------------------
 pub 2048R/19CAEC5B 2017-10-26
 uid Donald A. Tevault (No comment) <donniet@something.net>
 sub 2048R/37582F29 2017-10-26
 [donnie@localhost ~]$
```

1.  While you're at it, take a look at the files that you created:

```
[donnie@localhost ~]$ ls -l .gnupg/
total 12
drwx------. 2 donnie donnie   58 Oct 26 14:53 openpgp-revocs.d
drwx------. 2 donnie donnie  110 Oct 26 14:53 private-keys-v1.d
-rw-r--r--. 1 donnie donnie 1970 Oct 26 14:53 pubring.kbx
-rw-------. 1 donnie donnie   32 Oct 26 14:43 pubring.kbx~
-rw-------. 1 donnie donnie 1280 Oct 26 15:51 trustdb.gpg
[donnie@localhost ~]$
```

These files are your public and private keyrings, a revocation database, and a trusted users database.

### Hands-on lab – symmetrically encrypting your own files

You may find GPG useful for encrypting your own files, even when you never plan to share them with anyone else. For this, you'll use symmetric encryption, which involves using your own private key for encryption. Before you try this, you'll need to generate your keys, as I outlined in the previous section:

> Symmetric key encryption is, well, just that, symmetric. It's symmetric in the sense that the key that you would use to encrypt a file is the same key that you would use to decrypt the file. That's great for if you're just encrypting files for your own use. But if you need to share an encrypted file with someone else, you'll need to figure out a secure way to give that person the password. I mean, it's not like you'd want to just send the password in a plain-text email.

1.  In addition to your own user account, you'll also need a user account for Maggie. On AlmaLinux, create her account like this:

```
sudo useradd maggie
sudo passwd maggie
```

For Ubuntu, create Maggie’s account like this:

```
sudo adduser maggie
```

1.  Let's encrypt a super-secret file that we just can't allow to fall into the wrong hands:

```
[donnie@localhost ~]$ gpg -c secret_squirrel_stuff.txt
[donnie@localhost ~]$
```

Note that the `-c` option indicates that I chose to use symmetric encryption with a passphrase for the file. The passphrase that you enter will be for the file, not for your private key.

1.  Look at your new set of files. One slight flaw with this is that GPG makes an encrypted copy of the file, but it also leaves the original, unencrypted file intact:

```
[donnie@localhost ~]$ ls -l
 total 1748
 -rw-rw-r--. 1 donnie donnie 37 Oct 26 14:22 secret_squirrel_stuff.txt
 -rw-rw-r--. 1 donnie donnie 94 Oct 26 14:22
 secret_squirrel_stuff.txt.gpg
[donnie@localhost ~]$
```

1.  Let's get rid of that unencrypted file with `shred`. We'll use the `-u` option to delete the file, and the `-z` option to overwrite the deleted file with zeros:

```
[donnie@localhost ~]$ shred -u -z secret_squirrel_stuff.txt
[donnie@localhost ~]$
```

It doesn't look like anything happened, because `shred` doesn't give you any output. But `ls -l` will prove that the file is gone.

1.  Now, if I were to look at the encrypted file with `less secret_squirrel_stuff.txt.gpg`, I would be able to see its contents after being asked to enter my private key passphrase. Try this for yourself:

```
less secret_squirrel_stuff.txt.gpg
Shhh!!!! This file is super-secret.
secret_squirrel_stuff.txt.gpg (END)
```

1.  As long as my private key remains loaded into my keyring, I'll be able to view my encrypted file again without having to reenter the passphrase. Now, just to prove to you that the file really is encrypted, I'll create a shared directory, and move the file there for others to access. Again, go ahead and give it a try:

```
sudo mkdir /shared
sudo chown donnie: /shared
sudo chmod 755 /shared
mv secret_squirrel_stuff.txt.gpg /shared
```

When I go into that directory to view the file with `less`, I can still see its contents without having to reenter my passphrase.

1.  But now, let's see what happens when Maggie tries to view the file. Use `su - maggie` to switch to her account, and have her try:

```
su - maggie
cd /shared
[maggie@localhost shared]$ less secret_squirrel_stuff.txt.gpg
"secret_squirrel_stuff.txt.gpg" may be a binary file. See it anyway?
```

And when she hits the *Y* key to see it anyway, she gets this:

```
<8C>^M^D^C^C^B<BD>2=<D3>͈u<93><CE><C9>MОOy<B6>^O<A2><AD>}Rg9<94><EB><C4>^W^E
 <A6><8D><B9><B8><D3>(<98><C4>æF^_8Q2b<B8>C<B5><DB>^]<F1><CD>#<90>H<EB><90><
 C5>^S%X [<E9><EF><C7>
 ^@y+<FC><F2><BA><U+058C>H'+<D4>v<84>Y<98>G<D7>֊
secret_squirrel_stuff.txt.gpg (END)
```

Poor Maggie really wants to see my file, but all she can see is encrypted gibberish.

What I've just demonstrated is another advantage of GPG. After entering your private key passphrase once, you can view any of your encrypted files without having to manually decrypt them, and without having to reenter your passphrase. With other symmetric file encryption tools, such as `bcrypt`, you wouldn't be able to view your files without manually decrypting them first.

1.  But let's now say that you no longer need to have this file encrypted, and you want to decrypt it in order to let other people see it. Exit Maggie's account by typing `exit`. Then, just use `gpg` with the `-d` option:

```
[maggie@localhost shared]$ exit
[donnie@localhost shared]$ gpg -o secret_squirrel_stuff.txt -d secret_squirrel_stuff.txt.gpg
 gpg: AES256.CFB encrypted data
 gpg: encrypted with 1 passphrase
 Shhh!!!! This file is super-secret.
 [donnie@localhost shared]$
```

This works differently from how it worked on older Linux distros. On our newer distros, we now have to use the `-o` option along with the filename of the decrypted file that we want to create. Also, note that the `-o` option has to come before the `-d` option, or else you’ll get an error message.

### Hands-on lab – encrypting files with public keys

In this lab, you'll learn about how to encrypt and share a file with GPG public key encryption:

1.  To begin, create a user account for Frank, as you did for Maggie in the previous lab.
2.  Create a key set for both yourself and for Frank, as I've already shown you. Next, extract your own public keys into an `ASCII` text file:

```
cd .gnupg
gpg --export -a -o donnie_public-key.txt
```

Log in as Frank, and repeat this command for him.

1.  Normally, the participants in this would send their keys to each other either through an email attachment or by placing the keys in a shared directory. In this case, you and Frank will receive each other's public key files and place them into your respective `.gnupg` directories. Once that's done, import each other's keys:

```
donnie@ubuntu:~/.gnupg$ gpg --import frank_public-key.txt
gpg: key 4CFC6990: public key "Frank Siamese (I am a cat.) <frank@any.net>" imported
gpg: Total number processed: 1
gpg: imported: 1 (RSA: 1)
donnie@ubuntu:~/.gnupg$
frank@ubuntu:~/.gnupg$ gpg --import donnie_public-key.txt
gpg: key 9FD7014B: public key "Donald A. Tevault <donniet@something.net>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
frank@ubuntu:~/.gnupg$
```

1.  Now for the good stuff. Create a super-secret message for Frank, asymmetrically encrypt it (`-e`), and sign it (`-s`). Signing the message is the verification that the message really is from you, rather than from an impostor:

```
donnie@ubuntu:~$ gpg -s -e secret_stuff_for_frank.txt
. . .
. . .
It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.
Use this key anyway? (y/N) y
Current recipients:
2048R/CD8104F7 2017-10-27 "Frank Siamese (I am a cat.) <frank@any.net>"
Enter the user ID.  End with an empty line:
donnie@ubuntu:~$
```

So, the first thing you have to do is to enter the passphrase for your private key. Where it says to enter the user ID, enter `frank`, since he's the intended recipient of your message. But look at the line after that, where it says `There is no assurance this key belongs to the named user`. That's because you still haven't trusted Frank's public key. We'll get to that in a bit. The last line of the output again says to enter a user ID so that we can designate multiple recipients. But Frank is the only one you care about right now, so just hit the *Enter* key to break out of the routine. This results in a `.gpg` version of your message to Frank:

```
donnie@ubuntu:~$ ls -l
total 8
. . .
-rw-rw-r-- 1 donnie donnie 143 Oct 27 18:37 secret_stuff_for_frank.txt
-rw-rw-r-- 1 donnie donnie 790 Oct 27 18:39 secret_stuff_for_frank.txt.gpg
donnie@ubuntu:~$
```

1.  The final step on your end is to send Frank his encrypted message file by whatever means available.
2.  When Frank receives his message, he'll use the `-d` option to view it:

```
frank@ubuntu:~$ gpg -d secret_stuff_for_frank.txt.gpg
. . .
. . .
gpg: gpg-agent is not available in this session
gpg: encrypted with 2048-bit RSA key, ID CD8104F7, created 2017-10-27
      "Frank Siamese (I am a cat.) <frank@any.net>"
This is TOP SECRET stuff that only Frank can see!!!!!
If anyone else see it, it's the end of the world as we know it.
(With apologies to REM.)
gpg: Signature made Fri 27 Oct 2017 06:39:15 PM EDT using RSA key ID 9FD7014B
gpg: Good signature from "Donald A. Tevault <donniet@something.net>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: DB0B 31B8 876D 9B2C 7F12  9FC3 886F 3357 9FD7 014B
frank@ubuntu:~$
```

1.  Frank enters the passphrase for his private key, and he sees the message. At the bottom, he sees the warning about how your public key isn't trusted, and that `There is no indication that the signature belongs to the owner`. Let's say that you and Frank know each other personally, and he knows for a fact that the public key really is yours. He then adds your public key to the trusted list:

```
frank@ubuntu:~$ cd .gnupg
frank@ubuntu:~/.gnupg$ gpg --edit-key donnie
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
pub  2048R/9FD7014B  created: 2017-10-27  expires: never       usage: SC
                     trust: ultimate      validity: ultimate
sub  2048R/9625E7E9  created: 2017-10-27  expires: never       usage: E
[ultimate] (1). Donald A. Tevault <donniet@something.net>
gpg>
```

1.  The last line of this output is the command prompt for the `gpg` shell. Frank is concerned with trust, so he'll enter the `trust` command:

```
gpg> trust
pub  2048R/9FD7014B  created: 2017-10-27  expires: never       usage: SC
                     trust: unknown       validity: unknown
sub  2048R/9625E7E9  created: 2017-10-27  expires: never       usage: E
[ unknown] (1). Donald A. Tevault <donniet@something.net>
Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)
  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu
Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y
```

1.  Frank has known you for quite a while, and he knows for a fact that you're the one who sent the key. So, he chooses option `5` for ultimate trust. Once Frank logs out and logs back in, that trust will take effect:

```
frank@ubuntu:~$ gpg -d secret_stuff_for_frank.txt.gpg
You need a passphrase to unlock the secret key for
user: "Frank Siamese (I am a cat.) <frank@any.net>"
2048-bit RSA key, ID CD8104F7, created 2017-10-27 (main key ID 4CFC6990)
gpg: gpg-agent is not available in this session
gpg: encrypted with 2048-bit RSA key, ID CD8104F7, created 2017-10-27
      "Frank Siamese (I am a cat.) <frank@any.net>"
This is TOP SECRET stuff that only Frank can see!!!!!
If anyone else see it, it's the end of the world as we know it.
(With apologies to REM.)
gpg: Signature made Fri 27 Oct 2017 06:39:15 PM EDT using RSA key ID 9FD7014B
gpg: Good signature from "Donald A. Tevault <donniet@something.net>"
frank@ubuntu:~$
```

1.  With no more warning messages, this looks much better. At your end, do the same thing with Frank's public key.

> As you can see in the screen output in *step 8*, you can assign the marginal, full, or ultimate trust level to someone else's public key. Space doesn't permit me to provide a full explanation of the trust levels, but you can read a rather colorful explanation here: PGP Web of Trust: Core Concepts Behind Trusted Communication — [https://www.linux.com/tutorials/pgp-web-trust-core-concepts-behind-trusted-communication/.](https://www.linux.com/tutorials/pgp-web-trust-core-concepts-behind-trusted-communication/.)

What's so very cool about this is that even though the whole world may have your public key, it's useless to anyone who isn't a designated recipient of your message.

Now, let's look at how to sign a file *without* encrypting it.

### Hands-on lab – signing a file without encryption

If a file isn't secret but you still need to ensure authenticity and integrity, you can just sign it without encrypting it:

1.  Create an unencrypted message for Frank and then sign it:

```
donnie@ubuntu:~$ gpg -s not_secret_for_frank.txt
You need a passphrase to unlock the secret key for
user: "Donald A. Tevault <donniet@something.net>"
2048-bit RSA key, ID 9FD7014B, created 2017-10-27
gpg: gpg-agent is not available in this session
donnie@ubuntu:~$ ls -l
. . .
-rw-rw-r-- 1 donnie donnie  40 Oct 27 19:30 not_secret_for_frank.txt
-rw-rw-r-- 1 donnie donnie 381 Oct 27 19:31 not_secret_for_frank.txt.gpg
```

Just as before, this creates a `.gpg` version of the file.

1.  Send the message to Frank.
2.  Log in as Frank. Have him try to open it with `less`:

```
frank@ubuntu:~$ less not_secret_for_frank.txt.gpg
```

On older Linux distros, you’ll see a lot of gibberish because of the signature, but you’ll also see the plain-text message. On newer Linux distros, you’ll only see the plain-text message, without the gibberish.

1.  Have Frank use `gpg` with the `--verify` option to verify that the signature really does belong to you:

```
frank@ubuntu:~$ gpg --verify not_secret_for_frank.txt.gpg
gpg: Signature made Fri 27 Oct 2017 07:31:12 PM EDT using RSA key ID 9FD7014B
gpg: Good signature from "Donald A. Tevault <donniet@something.net>"
frank@ubuntu:~$
```

This wraps it up for our discussion of encrypting individual files. Let's now take a look at encrypting block devices and directories.

## Encrypting partitions with Linux Unified Key Setup (LUKS)

Being able to encrypt individual files can be handy, especially if you want to share sensitive files with other users. But, other types of encryption are also available:

*   **Block encryption**: We can use this for either whole-disk encryption or to encrypt individual partitions.
*   **File-level encryption**: We'd use this to encrypt individual directories without having to encrypt the underlying partitions.
*   **Containerized Encryption**: Using third-party software that doesn't come with any Linux distribution, we can create encrypted, cross-platform containers that can be opened on either Linux, macOS, or Windows machines.

**Linux Unified Key Setup** (**LUKS**) falls into the first category. It's built into pretty much every Linux distribution, and directions for use are the same for each. LUKS is now the default encryption mechanism for pretty much all of the newest Linux distros.

> You might be wondering if there's any performance impact with all of this disk encryption business. Well, with today's fast CPUs, not really. I run Fedora with full-disk encryption on a low-spec, Core i5 laptop, and other than having to enter the disk-encryption password when I first boot up, I don't even notice that encryption is taking place.

Okay, let's look at encrypting a disk while installing the operating system.

### Disk encryption during operating system installation

When you install most any Linux-based operating system, you have the option of encrypting the drive during the installation. Just click the **Encryption** option on the drive setup screen:

![19501_06_01.png](img/file37.png)

19501_06_01.png

Other than that, I just let the installer create the default partitioning scheme. On this AlmaLinux 9 machine, that means that the `/` filesystem and the `swap` partition will both be encrypted logical volumes. (I'll cover that in a moment.)

Before the installation can continue, I have to create a passphrase to mount the encrypted disk:

![19501_06_02.png](img/file38.png)

19501_06_02.png

Now, whenever I reboot the system, I need to enter this passphrase:

![19501_06_03.png](img/file39.png)

19501_06_03.png

Rather than actually encrypting a normal disk partition, the installer will set up encrypted **logical volumes**. Once the machine is up and running, I can look at the list of logical volumes. Here, I see both the `/` logical volume and the `swap` logical volume:

```
[donnie@localhost ~]$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/almalinux/swap
  LV Name                swap
  VG Name                almalinux
. . .
. . .
   --- Logical volume ---
  LV Path                /dev/almalinux/root
  LV Name                root
  VG Name                almalinux
. . .
. . .
[donnie@localhost ~]$
```

Now, let’s look at the list of **physical volumes**. Actually, there's only one physical volume in the list, and it's listed as a `luks` physical volume:

```
[donnie@localhost ~]$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/mapper/luks-b0acc532-5347-417e-a86e-a3ee8431fba7
  VG Name               almalinux
  PV Size               <19.30 GiB / not usable 2.00 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              4940
  Free PE               0
  Allocated PE          4940
  PV UUID               mRI75u-aVJI-uRjC-GY1O-ih7N-T3co-vssRRX

[donnie@localhost ~]$
```

In the `/etc/` directory, you’ll find the `crypttab` file, which contains an entry for this physical volume.

```
[donnie@localhost ~]$ sudo cat /etc/crypttab 
luks-b0acc532-5347-417e-a86e-a3ee8431fba7 UUID=b0acc532-5347-417e-a86e-a3ee8431fba7 none discard
[donnie@localhost ~]$
```

This shows that the underlying physical volume is encrypted, which means that both the `/` and the `swap` logical volumes are also encrypted. That's a good thing because leaving the swap space unencrypted—a common mistake when setting up disk encryption manually—can lead to data leakage. (We’ll talk more about this `crypttab` file in just a bit.)

#### Hands-on lab – adding an encrypted partition with LUKS

There may be times when you'll need to either add another encrypted drive to an existing machine or encrypt a portable device, such as a USB memory stick. This procedure works for both scenarios. Also, the procedure is the same for all of the Linux distros that we’re using, so it doesn’t matter which virtual machine you use. Follow these steps to add an encrypted partition:

Bump the size up to 20 GB:

![19501_06_05.png](img/file40.png)

19501_06_05.png

1.  After rebooting the machine, you'll now have a `/dev/sdb` drive to play with. We can see that here:

```
 donnie@ubuntu2204-packt:~$ ls -l /dev/sd*
brw-rw---- 1 root disk 8,  0 Oct 27 19:33 /dev/sda
brw-rw---- 1 root disk 8,  1 Oct 27 19:33 /dev/sda1
brw-rw---- 1 root disk 8,  2 Oct 27 19:33 /dev/sda2
brw-rw---- 1 root disk 8,  3 Oct 27 19:33 /dev/sda3
brw-rw---- 1 root disk 8, 16 Oct 27 19:33 /dev/sdb
donnie@ubuntu2204-packt:~$
```

1.  Open the drive in `gdisk`. Use the entire drive for the partition, and leave the partition type set at the default type `8300`:

```
sudo gdisk /dev/sdb
```

1.  View the details about your new `/dev/sdb1` partition:

```
donnie@ubuntu2204-packt:~$ sudo gdisk -l /dev/sdb
GPT fdisk (gdisk) version 1.0.8
Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present
. . .
. . .
Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        41943006   20.0 GiB    8300  Linux filesystem
donnie@ubuntu2204-packt:~$
```

1.  Next, use `cryptsetup` to convert the partition to LUKS format. In this command, the `-v` signifies verbose mode, and the `-y` signifies that you'll have to enter your passphrase twice in order to properly verify it. Note that when it says to type `yes` all in uppercase, it really does mean to type it in uppercase:

```
donnie@ubuntu2204-packt:~$ sudo cryptsetup -v -y luksFormat /dev/sdb1
WARNING!
========
This will overwrite data on /dev/sdb1 irrevocably.
Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/sdb1: 
Verify passphrase: 
Key slot 0 created.
Command successful.
donnie@ubuntu2204-packt:~
```

1.  Look at the information about your new encrypted partition:

```
donnie@ubuntu2204-packt:~$ sudo cryptsetup luksDump /dev/sdb1
LUKS header information
Version:        2
Epoch:          3
Metadata area:  16384 [bytes]
Keyslots area:  16744448 [bytes]
UUID:           e38e087a-205c-4aeb-81d5-03f03b8e8020
Label:          (no label)
Subsystem:      (no subsystem)
Flags:          (no flags)
. . .
. . .
```

There's a lot more to the output than I can show here, but you get the idea.

1.  Map the partition to a device name. You can name the device pretty much whatever you want. For now, just name this one `secrets`. I know, it's a corny name. In real life, you won't want to make it so obvious where you're storing your secrets:

```
donnie@ubuntu2204-packt:~$ sudo cryptsetup luksOpen /dev/sdb1 secrets
Enter passphrase for /dev/sdb1: 
donnie@ubuntu2204-packt:~$
```

1.  Look in the `/dev/mapper/` directory. You'll see your new `secrets` device listed as a symbolic link to some sort of `dm` device. (In this case, it’s `dm-1`.):

```
donnie@ubuntu2204-packt:~$ cd /dev/mapper
donnie@ubuntu2204-packt:/dev/mapper$ ls -l secrets 
lrwxrwxrwx 1 root root 7 Oct 27 19:50 secrets -> ../dm-1
donnie@ubuntu2204-packt:/dev/mapper$
```

1.  Use `dmsetup` to look at the information about your new device:

```
donnie@ubuntu2204-packt:~$ sudo dmsetup info secrets
Name:              secrets
State:             ACTIVE
Read Ahead:        256
Tables present:    LIVE
Open count:        0
Event number:      0
Major, minor:      253, 1
Number of targets: 1
UUID: CRYPT-LUKS2-e38e087a205c4aeb81d503f03b8e8020-secrets
donnie@ubuntu2204-packt:~$
```

1.  Format the partition in the usual manner. You can use any filesystem that's supported by your Linux distro. On a production server, that will generally mean either XFS or EXT4\. Just for fun, let’s go with XFS:

```
donnie@ubuntu2204-packt:~$ sudo mkfs.xfs /dev/mapper/secrets
meta-data=/dev/mapper/secrets    isize=512    agcount=4, agsize=1309631 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=5238523, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
donnie@ubuntu2204-packt:~$
```

1.  Create a mount point and mount the encrypted partition:

```
donnie@ubuntu2204-packt:~$ sudo mkdir /secrets
donnie@ubuntu2204-packt:~$ sudo mount /dev/mapper/secrets /secrets/
donnie@ubuntu2204-packt:~$
```

1.  Use the `mount` command to verify that the partition is mounted properly:

```
donnie@ubuntu2204-packt:~$ mount | grep 'secrets'
/dev/mapper/secrets on /secrets type xfs (rw,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota)
donnie@ubuntu2204-packt:~$
```

### Configuring the LUKS partition to mount automatically

The only missing piece of the puzzle is to configure the system to automatically mount the LUKS partition upon boot-up. To do that, configure two different files:

*   **/etc/crypttab**
*   **/etc/fstab**

If you encrypted the `sda` drive while installing the operating system, you'll already have a `crypttab` file that contains information about that drive. It would look something like this:

```
luks-b0acc532-5347-417e-a86e-a3ee8431fba7 UUID=b0acc532-5347-417e-a86e-a3ee8431fba7 none discard
```

The first two fields describe the name and location of the encrypted partition. The third field is for the encryption passphrase. If it's set to `none`, as it is here, then the passphrase will have to be manually entered upon boot-up.

In the `fstab` file, we have the entry that actually mounts the partition:

```
/dev/mapper/almalinux-root /           xfs     defaults,x-systemd.device-timeout=0 0 0
UUID=28218289-34cb-4c57-9755-379c65d580af /boot       xfs     defaults        0 0
/dev/mapper/almalinux-swap none      swap    defaults,x-systemd.device-timeout=0 0 0
```

Well, there are actually two entries in this case, because we have two logical volumes, `/` and `swap`, on top of the encrypted physical volume. The `UUID` line is the `/boot/` partition, which is the only part of the drive that isn't encrypted. Now, let's add our new encrypted partition so that it will mount automatically, as well.

### Hands-on lab – configuring the LUKS partition to mount automatically

In this lab, you'll set up the encrypted partition that you created in the previous lab to automatically mount when you reboot the machine:

> Tip:
> 
> > This is where it would be extremely helpful to remotely log in to your virtual machine from your desktop host machine. By using a GUI-type terminal, be it Terminal from a Linux or macOS machine or Cygwin from a Windows machine, you'll have the ability to perform copy-and-paste operations, which you won't have if you work directly from the virtual machine terminal. (Trust me, you don't want to be typing in those long UUIDs.)

1.  The first step is to obtain the UUID of the encrypted partition:

```
donnie@ubuntu2204-packt:~$ sudo cryptsetup luksUUID /dev/sdb1
e38e087a-205c-4aeb-81d5-03f03b8e8020
donnie@ubuntu2204-packt:~$
```

1.  Copy that UUID and paste it into the `/etc/crypttab` file. (If a `cryptab` file isn’t already there, just create a new one.) Also, note that you'll paste the UUID in twice. The first time, you'll prepend it with `luks-`, and the second time you'll append it with `UUID=`:

```
luks-e38e087a-205c-4aeb-81d5-03f03b8e8020 UUID=e38e087a-205c-4aeb-81d5-03f03b8e8020 none
```

1.  Edit the `/etc/fstab` file, adding the last line in the file for your new encrypted partition. Note that you again have to use `luks-`, followed by the UUID number:

```
/dev/mapper/luks-e38e087a-205c-4aeb-81d5-03f03b8e8020 /secrets xfs defaults 0 0
```

> When editing the `fstab` file for adding normal, unencrypted partitions, I always like to do `sudo mount -a` to check the `fstab` file for typos. That won't work with LUKS partitions though, because `mount` won't recognize the partition until the system reads in the `crypttab` file, and that won't happen until I reboot the machine. So, just be extra careful with editing `fstab` when adding LUKS partitions.

1.  Now for the moment of truth. Reboot the machine to see if everything works. Use the `mount` command to verify that your endeavors have been successful:

```
donnie@ubuntu2204-packt:~$ mount | grep 'secrets'
/dev/mapper/luks-e38e087a-205c-4aeb-81d5-03f03b8e8020 on /secrets type xfs (rw,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota)
donnie@ubuntu2204-packt:~$
```

1.  End of lab.

> Tip:
> 
> > Although it's possible to include passwords or keys in the `/etc/crypttab` file, my own preference is to not do so. If you must do so, be sure that the passwords or keys are stored on an encrypted `/` partition, for which you'll always have to enter a password upon boot-up. You can read more about that here: Store the passphrase of encrypted disk in `/etc/crypttab` encrypted: [https://askubuntu.com/questions/181518/store-the-passphrase-of-encrypted-disk-in-etc-crypttab-encrypted](https://askubuntu.com/questions/181518/store-the-passphrase-of-encrypted-disk-in-etc-crypttab-encrypted)

Now that we've seen LUKS, let's move on to eCryptfs.

## Encrypting directories with eCryptfs

Encrypting entire partitions is cool, but you might, at times, just need to use file-level encryption to encrypt an individual directory. For that, we can use eCryptfs. We'll need to use our Ubuntu machines for this, because Red Hat and its offspring no longer include eCryptfs. (It was in Red Hat 6 and CentOS 6, but it's no longer even available for installation in any newer versions.)

> Tip:
> 
> > It’s possible to use eCryptfs on a LUKS-encrypted disk. But, it’s not at all necessary, and I really don’t recommend it.

### Hands-on lab – encrypting a home directory for a new user account

In *Chapter* 3, Securing Normal User Accounts, I showed you how Ubuntu allows you to encrypt a user's home directory as you create his or her user account. To review, let's see the command for creating Goldie's account:

1.  If it hasn't already been done, install the `ecryptfs-utils` package:

```
sudo apt install ecryptfs-utils
```

1.  On an Ubuntu VM, create Goldie's account with an encrypted directory:

```
sudo adduser --encrypt-home goldie
```

1.  Have Goldie log in. Have her unwrap her mount passphrase, write it down, and store it in a secure place. She'll need it if she ever needs to recover a corrupted directory:

```
ecryptfs-unwrap-passphrase .ecryptfs/wrapped-passphrase
```

When you use `adduser --encrypt-home`, home directories for new users will automatically be set to a restrictive permissions value that will keep everyone out except for the owner of the directory. This happens even on Ubuntu 20.04 when you leave the `adduser.conf` file set with its default settings.

### Creating a private directory within an existing home directory

Let's say that you have users on your Ubuntu servers who, for whatever strange reason, don't want to encrypt their entire home directories, and want to keep the `755` permissions settings on their home directories so that other people can access their files. But they also want a private directory that nobody but them can access.

Instead of encrypting an entire home directory, any user can create an encrypted private directory within his or her own home directory. Let's check it out:

1.  If it hasn't already been done, install the `ecryptfs-utils` package:

```
sudo apt install ecryptfs-utils
```

To create this private directory, use the interactive `ecryptfs-setup-private` utility. If you have admin privileges, you can do this for other users. Users without admin privileges can do it for themselves. For our demo, let's say that Charlie, my big Siamese/Gray Tabby guy, needs his own encrypted private space. (Who knew that cats had secrets, right?)

1.  Create Charlie's account in the normal manner, *without* the encrypted home directory option.
2.  Then, log in as Charlie and have him create his own private directory:

```
charlie@ubuntu2:~$ ecryptfs-setup-private
Enter your login passphrase [charlie]:
Enter your mount passphrase [leave blank to generate one]:
Enter your mount passphrase (again):
************************************************************************
YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
  ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
************************************************************************
. . .
. . .
charlie@ubuntu2:~$
```

1.  For the login passphrase, Charlie enters his normal password or passphrase for logging in to his user account. He could have let the system generate its own mount passphrase, but he decided to enter his own. Since he did enter his own mount passphrase, he didn't need to do the `ecryptfs-unwrap-passphrase` command to find out what the passphrase is. But, just to show how that command works, let's say that Charlie entered `TurkeyLips` as his mount passphrase:

```
charlie@ubuntu2:~$ ecryptfs-unwrap-passphrase .ecryptfs/wrapped-passphrase
Passphrase:
TurkeyLips
charlie@ubuntu2:~$
```

Yes, it's a horribly weak passphrase, but for our demo purposes, it works.

1.  Have Charlie log out, and then log back in. After this, he can start using his new private directory. Also, you can see that he has three new hidden directories within his home directory. All three of these new directories are only accessible by Charlie, even if he set his top-level home directory so that it’s open to everybody:

```
charlie@ubuntu2:~$ ls -la
total 40
drwxr-xr-x 6 charlie charlie 4096 Oct 30 17:00 .
drwxr-xr-x 4 root root 4096 Oct 30 16:38 ..
-rw------- 1 charlie charlie 270 Oct 30 17:00 .bash_history
-rw-r--r-- 1 charlie charlie 220 Aug 31 2015 .bash_logout
-rw-r--r-- 1 charlie charlie 3771 Aug 31 2015 .bashrc
drwx------ 2 charlie charlie 4096 Oct 30 16:39 .cache
drwx------ 2 charlie charlie 4096 Oct 30 16:57 .ecryptfs
drwx------ 2 charlie charlie 4096 Oct 30 16:57 Private
drwx------ 2 charlie charlie 4096 Oct 30 16:57 .Private
-rw-r--r-- 1 charlie charlie 655 May 16 08:49 .profile
charlie@ubuntu2:~$
```

1.  Run the `grep 'ecryptfs' *` command in the `/etc/pam.d` directory. You'll see that PAM is configured to automatically mount users' encrypted directories whenever they log in to the system:

```
donnie@ubuntu2:/etc/pam.d$ grep 'ecryptfs' *
common-auth:auth    optional    pam_ecryptfs.so unwrap
common-password:password    optional    pam_ecryptfs.so
common-session:session    optional    pam_ecryptfs.so unwrap
common-session-noninteractive:session    optional    pam_ecryptfs.so unwrap
donnie@ubuntu2:/etc/pam.d$
```

1.  End of lab.

All righty, then. We now know how to encrypt users' home directories. Now, let's find out how to encrypt other directories.

### Hands-on lab – encrypting other directories with eCryptfs

Encrypting other directories is a simple matter of mounting them with the `ecryptfs` filesystem:

1.  Create a `secrets2` directory in the top level of the filesystem:

```
donnie@ubuntu2204-packt:~$ sudo mkdir /secrets2
[sudo] password for donnie: 
donnie@ubuntu2204-packt:~$
```

1.  Use `mount` with the `-t ecryptfs` option to encrypt the directory. Note that you’ll list the directory name twice, because the it will be used as its own mount point. From the menu, choose `1` to enter your desired passphrase, and choose the encryption algorithm and the key length:

```
donnie@ubuntu2204-packt:~$ sudo mount -t ecryptfs /secrets2/ /secrets2/
Select key type to use for newly created files: 
 1) passphrase
 2) tspi
Selection: 1
Passphrase: 
Select cipher: 
 1) aes: blocksize = 16; min keysize = 16; max keysize = 32
 2) blowfish: blocksize = 8; min keysize = 16; max keysize = 56
 3) des3_ede: blocksize = 8; min keysize = 24; max keysize = 24
 4) twofish: blocksize = 16; min keysize = 16; max keysize = 32
 5) cast6: blocksize = 16; min keysize = 16; max keysize = 32
 6) cast5: blocksize = 8; min keysize = 5; max keysize = 16
Selection [aes]:
```

Go with the default of `aes`, and `16` bytes for the key.

1.  Go with the default of `no` for `plaintext passthrough`, and with `yes` for filename encryption:

```
Enable plaintext passthrough (y/n) [n]:
Enable filename encryption (y/n) [n]: y
```

1.  Go with the default `Filename Encryption Key` and verify the mounting options:

```
Filename Encryption Key (FNEK) Signature [e339e1ebf3d58c36]:
Attempting to mount with the following options:
  ecryptfs_unlink_sigs
  ecryptfs_fnek_sig=e339e1ebf3d58c36
  ecryptfs_key_bytes=16
  ecryptfs_cipher=aes
  ecryptfs_sig=e339e1ebf3d58c36
```

1.  This warning only comes up when you mount the directory for the first time. For the final two questions, type `yes` in order to prevent that warning from coming up again:

```
WARNING: Based on the contents of [/root/.ecryptfs/sig-cache.txt],
it looks like you have never mounted with this key
before. This could mean that you have typed your
passphrase wrong.
Would you like to proceed with the mount (yes/no)? : yes
Would you like to append sig [e339e1ebf3d58c36] to
[/root/.ecryptfs/sig-cache.txt]
in order to avoid this warning in the future (yes/no)? : yes
Successfully appended new sig to user sig cache file
Mounted eCryptfs
```

1.  Just for fun, create a file within your new encrypted `secrets2` directory, and then unmount the directory. Then, try to do a directory listing:

```
cd /secrets2
sudo vim secret_stuff.txt
cd
sudo umount /secrets2
donnie@ubuntu2204-packt:~$ ls -l /secrets2/
total 12
-rw-rw-r-- 1 donnie donnie 12288 Oct 28 19:04 ECRYPTFS_FNEK_ENCRYPTED.FXbXCS5fwxKABUQtEPlumGPaN-RGvqd13yybkpTr1eCVWVHdr-lrmi1X9Vu-mLM-A-VeqIdN6KNZGcs-
donnie@ubuntu2204-packt:~$
```

By choosing to encrypt filenames, nobody can even tell what files you have when the directory is unmounted. When you're ready to access your encrypted files again, just remount the directory the same as you did before.

## Encrypting the swap partition with eCryptfs

If you're just encrypting individual directories with eCryptfs instead of using LUKS whole-disk encryption, you'll need to encrypt your swap partition in order to prevent accidental data leakage. Fixing that problem requires just one simple command:

```
donnie@ubuntu:~$ sudo ecryptfs-setup-swap
WARNING:
An encrypted swap is required to help ensure that encrypted files are not leaked to disk in an unencrypted format.
HOWEVER, THE SWAP ENCRYPTION CONFIGURATION PRODUCED BY THIS PROGRAM WILL BREAK HIBERNATE/RESUME ON THIS SYSTEM!
NOTE: Your suspend/resume capabilities will not be affected.
Do you want to proceed with encrypting your swap? [y/N]: y
INFO: Setting up swap: [/dev/sda5]
WARNING: Commented out your unencrypted swap from /etc/fstab
swapon: stat of /dev/mapper/cryptswap1 failed: No such file or directory
donnie@ubuntu:~$
```

Don't mind the warning about the missing `/dev/mapper/cryptswap1` file. It will get created the next time you reboot the machine.

## Using VeraCrypt for cross-platform sharing of encrypted containers

Once upon a time, there was TrueCrypt, a cross-platform program that allowed the sharing of encrypted containers across different operating systems. But the project was always shrouded in mystery because its developers would never reveal their identities. And then, right out of the blue, the developers released a cryptic message about how TrueCrypt was no longer secure, and shut down the project.

VeraCrypt is the successor to TrueCrypt, and it allows the sharing of encrypted containers across Linux, Windows, macOS, and FreeBSD machines. Although LUKS and eCryptfs are good, VeraCrypt offers more flexibility in certain ways:

*   As mentioned, VeraCrypt offers cross-platform sharing, whereas LUKS and eCryptfs don't.
*   VeraCrypt allows you to encrypt either whole partitions or whole storage devices, or to create virtual encrypted disks.
*   Not only can you create encrypted volumes with VeraCrypt, you can also hide them, giving you plausible deniability.
*   VeraCrypt comes in both command-line and GUI variants, so it's appropriate for either server use or for the casual desktop user.
*   Like LUKS and eCryptfs, VeraCrypt is free open source software, which means that it's free to use, and that the source code can be audited for either bugs or backdoors.

### Hands-on lab – getting and installing VeraCrypt

Follow these steps to install VeraCrypt:

1.  Download VeraCrypt from here: [https://www.veracrypt.fr/en/Downloads.html](https://www.veracrypt.fr/en/Downloads.html)
2.  The Linux version of VeraCrypt comes two ways. First, there’s `.tar.bz2` file, which contains a set of universal installer scripts that should work on any Linux distribution. Once you extract the `.tar.bz2` archive file, you'll see three scripts for GUI installation and two for console-mode installation. There are scripts for both 32-bit and 64-bit versions of Linux:

```
donnie@donnie-VirtualBox:~$ tar xjvf veracrypt-1.25.9-setup.tar.bz2 
veracrypt-1.25.9-setup-console-x64
veracrypt-1.25.9-setup-console-x86
veracrypt-1.25.9-setup-gtk3-console-x64
veracrypt-1.25.9-setup-gtk3-gui-x64
veracrypt-1.25.9-setup-gui-x64
veracrypt-1.25.9-setup-gui-x86
donnie@donnie-VirtualBox:~$
```

1.  The executable permission is already set, so all you have to do to install is this:

```
donnie@donnie-VirtualBox:~$ ./veracrypt-1.25.9-setup-gui-x64
```

You'll need sudo privileges, but the installer will prompt you for your sudo password. After reading and agreeing to a rather lengthy license agreement, the installation only takes a few seconds.

End of lab

> More recently, the VeraCrypt developers have also begun supplying `.deb` and `.rpm` installer packages for specific Linux distros. For Debian/Ubuntu-type systems, use `sudo dpkg -i` to install the `.deb` file. On RHEL/CentOS/AlmaLinux/SUSE systems, use `sudo rpm -Uvh` to install the `.rpm` file. Note that you might receive an error message telling you to install other packages as dependencies. Also, note that there’s no `.rpm` package for the RHEL/AlmaLinux 9 distros. Not to worry though, because I’ve just verified that the CentOS 8 package works just fine on AlmaLinux 9.

#### Hands-on lab – creating and mounting a VeraCrypt volume in console mode

I haven't been able to find any documentation for the console-mode variant of VeraCrypt, but you can see a list of the available commands just by typing `veracrypt`. For this demo, you'll create a 2 GB encrypted directory. But you can just as easily do it elsewhere, such as on a USB memory stick.

1.  To create a new encrypted volume, type the following:

```
veracrypt -c
```

1.  This will take you into an easy-to-use interactive utility. For the most part, you'll be fine just accepting the default options:

```
donnie@ubuntu:~$ veracrypt -c
Volume type:
 1) Normal
 2) Hidden
Select [1]:
Enter volume path: /home/donnie/good_stuff
Enter volume size (sizeK/size[M]/sizeG): 2G
Encryption Algorithm:
 1) AES
 2) Serpent
. . .
. . .
Select [1]:
. . .
. . .
```

1.  For the filesystem, the default option of `FAT` gives you the best cross-platform compatibility between Linux, macOS, and Windows:

```
Filesystem:
 1) None
 2) FAT
 3) Linux Ext2
 4) Linux Ext3
 5) Linux Ext4
 6) NTFS
 7) exFAT
Select [2]:
```

1.  Select your password and a **PIM** (short for **Personal Iterations Multiplier**). For my PIM, I entered `8891`. (High PIM values give better security, but they will also cause the volume to take longer to mount.) Then, type at least 320 random characters in order to generate the encryption key. (This is where it would be handy to have my cats walking across my keyboard):

```
Enter password:
Re-enter password:
Enter PIM: 8891
Enter keyfile path [none]:
Please type at least 320 randomly chosen characters and then press Enter:
```

1.  After you hit the **Enter** key, be patient, because the final generation of your encrypted volume will take a few moments. Here, you see that my 2 GB `good_stuff` container has been successfully created:

```
donnie@ubuntu:~$ ls -l good_stuff
-rw------- 1 donnie donnie 2147483648 Nov  1 17:02 good_stuff
donnie@ubuntu:~$
```

1.  Mount this container in order to use it. Begin by creating a mount point directory:

```
donnie@ubuntu:~$ mkdir good_stuff_dir
donnie@ubuntu:~$
```

1.  Use the `veracrypt` utility to mount your container on this mount point:

```
donnie@ubuntu:~$ veracrypt good_stuff good_stuff_dir
Enter password for /home/donnie/good_stuff:
Enter PIM for /home/donnie/good_stuff: 8891
Enter keyfile [none]:
Protect hidden volume (if any)? (y=Yes/n=No) [No]:
Enter your user password or administrator password:
donnie@ubuntu:~$
```

1.  To see what VeraCrypt volumes you have mounted, use `veracrypt -l`:

```
donnie@ubuntu:~$ veracrypt -l
1: /home/donnie/secret_stuff /dev/mapper/veracrypt1 /home/donnie/secret_stuff_dir
2: /home/donnie/good_stuff /dev/mapper/veracrypt2 /home/donnie/good_stuff_dir
donnie@ubuntu:~$
```

1.  End of lab. That's all there is to it.

### Using VeraCrypt in GUI mode

Desktop users of any of the supported operating systems can install the GUI variant of VeraCrypt. Be aware, though, that you can't install both the console-mode variant and the GUI variant on the same machine, because one will overwrite the other. Here’s what that looks like:

![19501_06_06.png](img/file41.png)

19501_06_06.png

Since the main focus of this book is server security, I won't go into the details of the GUI version here. But it's fairly self-explanatory, and you can view the full VeraCrypt documentation on their website.

> You can get VeraCrypt from here: [https://www.veracrypt.fr/en/Home.html](https://www.veracrypt.fr/en/Home.html).

## OpenSSL and the public key infrastructure

With OpenSSL, we can encrypt information on the fly as it goes across the network. There's no need to manually encrypt our data before we send it across the network because OpenSSL encryption happens automatically. This is important because online commerce and banking couldn't exist without it.

The **Secure Sockets Layer** (**SSL**) is the original in-transit encryption protocol. Ironically, even though we're using the OpenSSL suite of programs and libraries, we no longer want to use SSL. Instead, we now want to use the **Transport Layer Security** (**TLS**) protocol . SSL is full of legacy code and a lot of vulnerabilities that go along with that legacy code. TLS is newer, and is much more secure. But, even when working with TLS, we can still use the OpenSSL suite.

One reason that the older SSL protocol is so bad is because of past government regulations, especially here in the U.S., that prohibited the use of strong encryption. For the first few years of the public Internet, U.S. website operators couldn't legally implement encryption keys that were longer than a measly 40 bits. Even back then, a 40-bit key didn't provide a whole lot of security. But the U.S. government considered strong encryption as a type of munition, and tried to control it so that the governments of other countries couldn't use it. Meanwhile, an Australian outfit named Fortify started producing a strong encryption plugin that people could install in their Netscape web browsers. This plugin allowed the use of 128-bit encryption, and my geek buddies and I all eagerly installed it on our own machines. Looking back, I'm not sure that it did a lot of good, because website operators in the U.S. were still prohibited from using strong encryption keys on their web servers.

Amazingly, the Fortify outfit still has their website up. You can still download the Fortify plugin, even though it's now completely useless. Here’s a screenshot of the Fortify website:

![19501_06_07.png](img/file42.png)

19501_06_07.png

An encrypted SSL/TLS session uses both symmetric and asymmetric mechanisms. For acceptable performance, it uses symmetric encryption to encrypt the data in transit. But symmetric encryption requires a private key to be exchanged between the two communication partners. To do that, SSL/TLS first negotiates an asymmetric session using the same public key exchange mechanism that we looked at in the GPG section.

Once that asymmetric session is set up, the two communication partners can safely exchange the private key that they'll use for the symmetric session.

### Commercial certificate authorities

To make this magic work, you need to install a security certificate onto your web server. The certificate serves two purposes:

*   It contains the public key that's needed to set up an asymmetric key-exchange session.
*   Optionally, it can verify the identity of, or authenticate, your website. So, for example, users can theoretically be sure that they're connected to their real bank, instead of to Joe Hacker's Bank of Crooks and Criminals that's disguised as their bank.

When you shop for a certificate, you'll find quite a few vendors, which are all referred to as **certificate authorities**, or **CAs**. Most CAs, including vendors such as Thawte, Symantec, GoDaddy, and Let's Encrypt, among others, offer several different grades of certificates. To help explain the differences between the grades of certificates, here's a screenshot from the GoDaddy site:

At the left-hand side of the list, at the cheapest price, is the **standard** **Domain Verification** **(DV)** offering. Vendors advertise this type of certificate as for use where all you really care about is encryption. Identity verification is limited to domain verification, which means that yeah, records for your site have been found on a publicly accessible DNS server.

At the right, we see the **premium** **Extended Verification** (**EV**) offering. This is the top-of-the-line, highest-grade certificate that certificate vendors offer. With this extended verification grade of certificate, you have to jump through some hoops to prove that you are who you really are and that your website and your business are both legit. It used to be that both Firefox and Chrome would show a green High-Assurance bar in the URL of any site with an EV certificate, but they no longer do, for reasons that I’ll explain in a moment.

So, just how good is this **Premium SSL EV** certificate with rigorous identity testing? Well, not quite as good as I thought. Two days after I wrote this explanation about the different types of certificates for the previous edition of this book, I received the latest edition of the *Bulletproof TLS Newsletter* from Feisty Duck Publishing. The big news was that Google and Mozilla decided to remove the green high assurance bar from future editions of Chrome and Firefox. Their reasons are as follows:

*   The green high assurance bar is meant to help users avoid phishing attacks. But for that to be useful, users have to notice that the high assurance bar is even there. Studies have shown that most people don't even notice it.
*   Ian Carrol, a security researcher, questions the value of extended validation certificates. As an experiment, he was able to register a bogus certificate for Stripe, Inc., which is a legitimate company. The certificate vendor finally did notice their mistake and revoked the certificate, but it's something that shouldn't have happened in the first place.
*   On top of everything else, it's also possible to register extended validation certificates with incorrect information. This indicates that the verification process isn't quite as thorough as the certificate vendors would have us believe.

But in spite of these occasional problems, I still believe that extended validation certificates are useful. When I access my bank account, I like to believe that extra identity verification is never a bad thing.

> Something else that's rather curious is that certificate vendors still market their certificates as SSL certificates. Don't be fooled, though. As long as the website owners configure their servers correctly, they'll be using the more secure TLS protocol, rather than SSL.

**Let's Encrypt** is a fairly new organization that has the goal of ensuring that all websites everywhere are set up with encryption. It's a worthy goal, but it has also introduced a new problem. Here’s what the Let's Encrypt website looks like:

![19501_06_09.png](img/file43.png)

19501_06_09.png

To obtain a certificate from one of the traditional vendors, you have to use the OpenSSL utility to create your keys and a certificate request. Then, you'll submit the certificate request, proof of identity if applicable, and your payment to the certificate authority. Depending upon which grade of certificate you purchase, you'll have to wait anywhere from one to several days before you get the certificate.

Let's Encrypt is totally free of charge, and you don't have to jump through hoops to get the certificate. Instead, you configure your web server to automatically obtain a new Let's Encrypt certificate each time you set up a new website. If Let's Encrypt sees that your new site has a valid record on a publicly accessible DNS server, it will automatically create and install the certificate on your server. Other than having to configure your web server to use Let's Encrypt, it's no fuss, no muss.

The problem with Let's Encrypt is that it's even easier to abuse than the extended validation certificates. Shortly after Let's Encrypt began operation, criminals began setting up domains that appeared to be subdomains of legitimate business websites. So, people see that the website is encrypted and that the domain name seems to be legit, and they merrily enter their credentials without giving things a second thought. Let's Encrypt is handy and useful for legitimate purposes, but be aware of its downside, too.

> Tip:
> 
> > Before you choose a certificate vendor, do some research. Sometimes, even the big name vendors have problems. A few years ago, Google removed Symantec from Chrome's list of trusted certificate authorities because Symantec had allegedly violated industry best practices several times. That's rather ironic, considering that Symantec has had a long history of being a trusted vendor of security products.

Now that we've covered the basics of SSL/TLS encryption, let's see how to implement it with the OpenSSL suite.

### Creating keys, certificate signing requests, and certificates

The good news is that, regardless of which *newer* Linux distribution we're on, this procedure is the same. (I say *newer*, because the newest versions of Ubuntu and RHEL/AlmaLinux use OpenSSL version 3\. Some of the Version 3 commands are different from what you’ll see on the older versions.) The not-so-good news is that OpenSSL can be a bit tricky to learn because it has loads of sub-commands, each with its own set of options and arguments. Bear with me, and I'll break it down the best I can.

#### Creating a self-signed certificate with an RSA key

A self-signed certificate is useful when all you need is encryption, or for testing purposes. There's no identity verification involved with self-signed certificates, so you never want to use them on servers that your users need to trust. Let's say that I need to test my new website setup before putting it into production, and I don't want to do my testing with a for-real key and certificate. I'll create the key and the self-signed certificate with one single command:

```
openssl req -newkey rsa:2048 -nodes -keyout donnie-domain.key-x509 -days 365 -out donnie-domain.crt
```

Here's the breakdown:

*   **openssl**: I'm using OpenSSL with just my normal user privileges. For now, I'm doing everything in my own home directory, so there's no need for root or sudo privileges.
*   **req**: This is the sub-command for managing certificate signing requests (CSRs). When creating self-signed certificates, OpenSSL will create a temporary CSR.
*   **-newkey rsa:2048**: I'm creating an RSA keypair that's 2,048 bits in length. I'd actually like to use something a bit longer, but that might impact server performance when setting up the TLS handshake. (Again, this is preceded by only a single dash.)
*   **-nodes**: This means that I'm not encrypting the private key that I'm about to create. If I were to encrypt the private key, I would have to enter the private key passphrase every time I restart the web server.
*   **-keyout donnie-domain.key-x509**: I'm creating the private key with the name `donnie-domain.key-x509`. The `x509` part indicates that this will be used for a self-signed certificate.
*   **-days 365**: The certificate will expire in one year.
*   **-out donnie-domain.crt**: Finally, I'm creating the `donnie-domain.crt` certificate.

When you run this command, you'll be prompted to enter information about your business and your server. (We'll look at that in just a moment.) After creating this key and certificate, I'll need to move them to their proper locations and configure my web server to find them. (We'll also touch on that in a bit.)

Encrypting the private key is an optional step, which I didn't do. If I were to encrypt the private key, I would have to enter the passphrase every time that I restart the web server. That could be problematic if there are any web server admins who don't have the passphrase. And, even though this sounds counter-intuitive, encrypting the private key that's on the web server doesn't really help that much with security. Any malicious person who can get physical access to the web server can use memory forensics tools to get the private key from system memory, even if the key is encrypted. But if you plan to make a backup of the key to store elsewhere, definitely encrypt that copy. So now, let's make an encrypted backup copy of my private key that I can safely store somewhere other than on the web server:

```
[donnie@localhost ~]$ openssl rsa -aes256 -in donnie-domain.key-x509 -out donnie-domain-encrypted.key-x509 
writing RSA key
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
[donnie@localhost ~]$
```

There are two things to look at here:

*   **rsa -aes256** means that I'm using the AES256 encryption algorithm to encrypt an RSA key.
*   To ensure that I made a copy instead of overwriting the original unencrypted key, I specified `donnie-domain-encrypted.key-x509` as the name for the copy.

#### Creating a self-signed certificate with an Elliptic Curve key

```
RSA keys were okay in their day, but they do have their disadvantages. (I'll cover this more in just a bit.) Elliptic Curve (EC) keys are superior in pretty much every way. So, let's now create a self-signed certificate with an EC key, instead of with an RSA key, like so:
openssl req -new -x509 -nodes -newkey ec:<(openssl ecparam -name secp384r1) -keyout cert.key.x509 -out cert.crt -days 3650
```

The only part of this that's different is the `ec:<(openssl ecparam -name secp384r1)` part. It looks strange, but it's really quite logical. When creating an EC key, you have to specify a parameter with the `ecparam` command. You'll normally see this as two separate `openssl` commands, but it's handier to combine the two commands together as one command within another command. The inner `openssl` command is feeding its output back to the outer `openssl` command via the input redirection symbol (`<`). The `-name secp384r1` part means that we're creating a 384-bit EC key with the `secp384` named curve algorithm.

#### Creating an RSA key and a Certificate Signing Request

Normally, we won't use a self-signed certificate for anything that's meant for the general public to interface with. Instead, we want to obtain a certificate from a commercial CA because we want users to know that they're connecting to a server for which the identity of its owners has been verified. To obtain a certificate from a trusted CA, you'll first need to create a key and a **Certificate Signing Request** (**CSR**). Let's do that now:

```
openssl req --out CSR.csr -new -newkey rsa:2048 -nodes -keyout server-privatekey.key
```

Here's the breakdown:

*   **openssl**: I'm using OpenSSL with just my normal user privileges. For now, I'm doing everything in my own home directory, so there's no need for root or sudo privileges.
*   **req**: This is the sub-command for managing CSRs.
*   **--out CSR.csr**: The `--out` means that I'm creating something. In this case, I'm creating the CSR with the name `CSR.csr`. All CSRs will have the `.csr` filename extension.
*   **-new**: This is a new request. (And yes, this is preceded by a single dash, unlike the `out` in the previous line that's preceded by two dashes.)
*   **-newkey rsa:2048**: I'm creating an RSA key pair that's 2,048 bits in length. I'd actually like to use something a bit longer, but that will impact server performance when setting up the TLS handshake. (Again, this is preceded by only a single dash.)
*   **-nodes**: This means that I'm not encrypting the private key that I'm about to create. If I were to encrypt the private key, I would have to enter the private key passphrase every time I restart the web server.
*   **-keyout server-privatekey.key**: Finally, I'm creating the private key with the name `server-privatekey.key`. Since this key isn't for a self-signed certificate, I didn't put the `-x509` at the end of the key's filename.

Let's now look at a snippet from the command output:

```
[donnie@localhost ~]$ openssl req --out CSR.csr -new -newkey rsa:2048 -nodes -keyout server-privatekey.key
Generating a RSA private key
. . .
. . .
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:GA
Locality Name (eg, city) [Default City]:Saint Marys
Organization Name (eg, company) [Default Company Ltd]:Tevault Enterprises
Organizational Unit Name (eg, section) []:Education
Common Name (eg, your name or your server's hostname) []:www.tevaultenterprises.com
Email Address []:any@any.net
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:TurkeyLips
An optional company name []:
```

So, I've entered my information about my company location, name, and website name. Note the bottom where it asks me for a **challenge password**. This password doesn't encrypt either the key or the certificate. Rather, it's just a shared secret between the certificate authority and me that's embedded into the certificate. I'll need to keep it in a safe place in case I ever need to reinstall the certificate. (And, for goodness' sake, when you do this for real, pick a better password than `TurkeyLips`.)

As before, I didn't encrypt the private key. But if you need to make a backup copy, just follow the procedure that you saw in the previous section.

To obtain a certificate from a commercial CA, go to their website and follow their directions. When you receive your certificate, install it in the proper place in your web server and configure the web server to find it.

#### Creating an EC key and a CSR

Up until a few years ago, you would have wanted to use RSA keys on your web servers. They don't have the security weaknesses that certain other key types have, and they're widely supported by pretty much every web browser. But RSA keys do have two weaknesses:

*   Even at the standard 2,048-bit length, they require more computational power than other key types. Increasing the key length for better security could degrade web server performance.
*   RSA doesn't offer **Perfect Forward Secrecy** (**PFS**). In other words, if someone were to capture a session key that's produced by the RSA algorithm, they would be able to decrypt material from the past. If the same person were to capture a session key that was produced by a PFS algorithm, they would only be able to decrypt the current communication stream.

Using the new-fangled EC algorithms instead of the creaky old RSA solves both of these problems. But if you pick up a book from even a couple of years ago, you'll see that it recommends using RSA keys for backward compatibility with older web browsers. That's partly because certain operating systems, along with their associated proprietary web browsers, lingered on for far longer than they should have. (*I'm looking at you, Windows XP.*) Now though, as I sit here writing this in October 2022, I think it's safe to start ignoring the needs of anyone who refuses to move on from these antiquated platforms. I mean, Windows XP and Windows 7 both reached end-of-life several years ago. So, let's get with the times, people.

Unlike what we just saw with the RSA keys, we can't create the EC private key and the CSR all with one simple command. With EC, we need to do this in two separate steps.

First, I'll create the private key:

```
openssl genpkey -algorithm EC -out eckey.pem -pkeyopt ec_paramgen_curve:P-384 -pkeyopt ec_param_enc:named_curve
```

Here's the breakdown:

*   **genpkey -algorithm EC**: The `genpkey` command is a fairly recent addition to OpenSSL and is now the recommended way to create private keys. Here, I'm telling it to create a key with the EC algorithm.
*   **-out eckey.pem**: I'm creating the `eckey.pem` key, which is in the **Privacy Enhanced Mail** (**PEM**) format. The RSA keys that I created in the previous section were also PEM keys, but I used the `.key` filename extension on them. You can use either the `.key` or the `.pem` filename extension, and they'll both work. But if you use the `.pem` extension, everyone who looks at them can tell at a glance that they are PEM keys.
*   **-pkeyopt ec_paramgen_curve:P-384**: This tells OpenSSL to create an EC key that's 384 bits in length. A beautiful thing about EC is that its shorter-length keys provide the same encryption strength as the longer RSA keys. In this case, we have a 384-bit key that's actually stronger than a 2,048-bit RSA key. And, it requires less computational power. (I call that a total win!)
*   **-pkeyopt ec_param_enc:named_curve**: This is the encoding method that I'm using for the EC parameters. It has to be set to either `named_curve or explicit`.

Now, I'll create a CSR and sign it with my new private key, like so:

```
[donnie@localhost ~]$ openssl req -new -key eckey.pem -out eckey.csr
. . .
. . .
[donnie@localhost ~]$
```

The output that I didn't include is the same as what you saw in the RSA key section.

The final steps are the same as before. Choose a CA and let them tell you how to submit the CSR. When they issue the certificate, install it on your web server.

### Creating an on-premises CA

Buying a certificate from a commercial CA is good when you're dealing with the general public on a website that they need to trust. But for an organization's own internal use, it's not always necessary or feasible to buy commercial certificates. Let's say that your organization has a group of developers who need their own client certificates to access the development server. Buying a commercial certificate for each developer would be costly, and it would require the development server to have a publicly accessible domain name so that the commercial CA can do domain verification. Even going with the free-of-charge Let's Encrypt certificates isn't a good option, because that would also require that the development server have a publicly accessible domain name. Option 2 is to go with self-signed certificates. But that won't work because client authentication doesn't work with self-signed certificates. That leaves Option 3, setting up a private, on-premises CA.

If you search around on the web, you'll find lots of guides for setting up your own private CA. But almost all of them are woefully outdated, and most of them are for setting up a CA with OpenSSL. There's nothing wrong with using OpenSSL for a CA, except that setting it up is a rather convoluted, multi-stage process. Then, when you finally do have it set up, you have to use complex commands from the command line in order to do anything. What we want is something a bit more user-friendly for both you and your users.

### Hands-on lab – setting up a Dogtag CA

Dogtag PKI is much simpler to set up, and it has a nice web interface that OpenSSL doesn't have. It's available in the normal repositories of Debian/Ubuntu and RHEL/AlmaLinux, but under different package names. In the Debian/Ubuntu repositories, the package name is `dogtag-pki`. In the RHEL/AlmaLinux repositories, the name is `pki-ca`. (For some reason that I don't understand, you'll never see Red Hat folk use the "Dogtag" name.)

Before we install the Dogtag packages, we need to do a couple of simple chores:

```
Set a Fully Qualified Domain Name (FQDN) on the server
```

Either create a record in a local DNS server for the Dogtag server, or create an entry for it in its own `/etc/hosts` file

You can do this on either your AlmaLinux 9 or your Ubuntu 22.04 VM, and I’ll give directions for both. To access the Dogtag dashboard, we'll use a second Linux VM with a desktop environment installed. With all this out of the way, let's get started:

1.  On your server virtual machine, set an FQDN, substituting your own for the one that I'm using:

```
sudo hostnamectl set-hostname donnie-ca.local
```

1.  Edit the `/etc/hosts` file to add a line like the following:

```
192.168.0.53 donnie-ca.local
```

Use your virtual machine's own IP address and FQDN.

1.  Next, increase the number of file descriptors that your system can have open at one time. (Otherwise, you'll get a warning message when you run the directory server installer.) Do that by editing the `/etc/security/limits.conf` file. At the end of the file, add these two lines:

```
root            hard    nofile          4096
root            soft    nofile          4096
```

1.  Reboot the machine so that the new hostname and file descriptor limits can take effect.
2.  Dogtag stores its certificate and user information in an LDAP database. In this step, we'll install the LDAP server package, along with the Dogtag package. For AlmaLinux 9, do this:

```
sudo dnf install 389-ds-base pki-ca
```

For Ubuntu 22.04, do this:

```
 sudo apt install 389-ds-base dogtag-pki
```

Next, create an LDAP Directory Server (DS) instance by first creating an `instance.inf` file in the root user’s home directory:

```
sudo vim /root/instance.inf
```

```
Make its contents look something like this, using your own suffix and root_password:
# /root/instance.inf
[general]
config_version = 2
[slapd]
root_password = TurkeyLips
[backend-userroot]
sample_entries = yes
suffix = dc=donnie-ca,dc=local
```

1.  (Yes, I know that it’s bad practice to put passwords into plain-text configuration files. That’s okay, though. We’ll take care of that in just a bit.)
2.  We can now use this `instance.inf` file, along with the `dscreate` utility, to create the Directory Server instance:

```
sudo dscreate from-file /root/instance.inf
```

1.  Finally, it's time to create the CA:

```
sudo pkispawn
```

Accept all the defaults until you get to the very end. When it asks **Begin Installation?**, type `Yes`. When you get to the Directory Server part, enter the password that you used to create the DS instance in the previous step. Note that you'll be offered the choice to access the LDAP DS instance via a secure port. But since we're setting up LDAP and Dogtag on the same machine, this isn't necessary.

1.  Ensure that the Dogtag service will automatically start by enabling the `pki-tomcatd.target`. Do that with:

```
sudo systemctl enable pki-tomcatd.target
sudo shutdown -r now
```

1.  After everything is set up, you’ll no longer need the `instance.inf` file that holds your password in plain-text. Get rid of it by doing:

```
sudo shred -u -z /root/instance.inf
```

1.  You'll access the Dogtag web interface via port `8443/tcp`. On the AlmaLinux machine, open that port like this:

```
sudo firewall-cmd --permanent --add-port=8443/tcp
sudo firewall-cmd --reload
```

On the Ubuntu machine, assuming that you’re using the Uncomplicated Firewall, open the port like this:

```
sudo ufw allow 8443/tcp
```

1.  On another Linux virtual machine that has a desktop interface, edit the `/etc/hosts` file to add the same line that you added to the server `hosts` file in *step 2*. Then, open the Firefox web browser on that machine and navigate to the Dogtag dashboard. In keeping with the example in this scenario, the URL would look like this:

```
https://donnie-ca.local:8443
```

You'll receive a warning about the certificate being invalid because it's self-signed. That's normal, because every CA has to start with a self-signed certificate, and you haven't yet imported this certificate into your trust store. Temporarily add the exception and continue. (In other words, clear the checkmark from the **Add permanently** box. You'll see why in the next lab.) Click through the links until you reach this screen:

![19501_06_10.png](img/file44.png)

19501_06_10.png

1.  Click the **SSL End Users Services** link. This is where end users can request the various types of certificates. Click the back button to return to the previous screen. This time, click on the **Agent Services** link. You won't be able to go there because it requires you to install a certificate into your web browser for authentication.
2.  The certificate that you need to install is in the `/root/.dogtag/pki-tomcat/` directory of your Dogtag VM. Copy this file to the VM on which you're using Firefox to access the Dogtag dashboard. Do the following:

```
sudo su -
cd /root/.dogtag/pki-tomcat
scp ca_admin_cert.p12 donnie@192.168.0.14:
exit
```

Of course, substitute your own username and IP address. Note that the file will automatically land in your own home directory, and that its ownership will change from root to your own username.

1.  On the VM with Firefox, import the certificate into Firefox. From the Firefox menu, choose **Settings**, then **Privacy and Security**. At the very bottom of the screen, click on **View Certificates**. Click the **Your Certificates** tab at the top and the **Import** button at the bottom. Navigate to your home directory and choose the certificate that you just sent over from the Dogtag server VM. Once the import operation is complete, you should see the **PKI Administrator** certificate in the list of imported certificates:

    ![19501_06_11.png](img/file45.png)

    19501_06_11.png

2.  Now try to access the **Agent Services** page. You'll be allowed access once you confirm that you want to use the certificate that you just imported.
3.  End of lab.

When a users need to request a certificate for their own use, they'll use `openssl` to create a key and a CSR, as I've already shown you earlier in this chapter. They'll then go to the SSL End User Services page and paste the contents of their CSR into the box for the certificate that they're requesting. An administrator will then go to the Agent Services page to approve the request and issue the certificate. (To help familiarize yourself with Dogtag, I encourage you to click around on the web interface, exploring all the options.)

### Adding a CA to an operating system

Most of the major web browsers, such as Firefox, Chrome, and Chromium, come with their own pre-defined database of trusted CAs and their associated certificates. When you create a private CA, you'll need to import the CA certificate into your browser's trust store. Otherwise, your users will keep receiving messages about how the sites that they're viewing are using untrusted certificates. Indeed, that's the case with our Dogtag server. Any user who accesses it to request a certificate will receive a warning about how the CA is using a non-trusted certificate. We'll fix that by exporting the CA certificate from the Dogtag server and importing it into all of your users' browsers. Let's dig in, shall we?

#### Hands-on lab – exporting and importing the Dogtag CA certificate

The Dogtag web portal doesn't have an option for this, so we'll have to use the command line:

1.  In your home directory of the Dogtag server, create the `password.txt` file. On the first line of the file, insert the password for the server's certificate. (It's the password that you set when you ran the `pkispawn` command.)
2.  Extract the server key and certificate like so:

```
sudo pki-server ca-cert-chain-export --pkcs12-file pki-server.p12 --pkcs12-password-file password.txt
```

Run an `ls -l` command to verify that the `pki-server.p12` file was created.

1.  The problem with the `p12` file is that it contains both the server's private key and its certificate. But to add a certificate to the CA section of your browser's trusted store, you have to have just the certificate without the key. Extract the certificate like so:

```
openssl pkcs12 -info -in pki-server.p12 -out pki-server.crt -nokeys
```

1.  Transfer this new `pki-server.crt` file to a machine with a graphical desktop. In Firefox, open **Settings/Privacy & Security**. Click the **View Certificates** button at the bottom. Click the **Authorities** tab and import the new certificate. Select **Trust this CA to identify websites** and to **Trust this CA to identify email users**:

    ![19501_06_12.png](img/file46.png)

    19501_06_12.png

2.  Close Firefox and then open it again to ensure that the certificate takes effect. Navigate to the Dogtag portal. This time, you shouldn't receive any warning messages about using an untrusted certificate.
3.  End of lab.

#### Importing the CA into Windows

With either Firefox or Chrome, you'll import the CA certificate directly into the browser's trust store, regardless of which operating system you're running. But if you're stuck running one of Microsoft's own proprietary browsers on that off-brand operating system that's known as Windows, then you'll need to import the certificate into the Windows trust store instead of into the browser. Fortunately, that's incredibly easy to do. After you copy the certificate to the Windows machine, just open up Windows File Explorer and double-click on the certificate file. Then, click the **Install Certificate** button on the pop-up dialog box. If your organization is running an Active Directory domain, just ask one of the AD administrators to import it into Active Directory for you.

### OpenSSL and the Apache web server

A default installation of any web server isn't all that secure, so you'll need to harden it up a bit. One way to do that is by disabling the weaker SSL/TLS encryption algorithms. The general principles apply to all web servers, but for our examples, we'll just look at Apache. (The topic of web server hardening is quite extensive. For the present, I'll confine the discussion to hardening the SSL/TLS configuration.) You can use either Ubuntu 22.04 or AlmaLinux 9 for this section, but the package names and configuration files are different between the two distros. The configurations also differ between CentOS 7 and AlmaLinux 9, so we'll look at them as well. But, before I can explain the configuration options, I need to say a word or two about the history of the SSL/TLS protocol.

In the 1990s, engineers at Netscape invented the SSL protocol. Version 1 never saw the light of day, so the first released version was SSL version 2 (SSLv2). SSLv2 had its share of weaknesses, many of which were addressed in SSLv3\. At the insistence of Microsoft, the next version was renamed Transport Layer Security (TLS) version 1 (TLSv1). (I have no idea why Microsoft objected to the SSL name.) The current version is TLSv1.3, which is finally now supported by most Linux distros. By default, Apache still supports some of the older protocols. Our goal is to disable those older protocols. Only a couple of years ago, that would have meant disabling SSLv2 and SSLv3 and leaving TLSv1 through TLSv1.2, due to questionable browser support for anything newer. Now, though, I think it's safe to disable support for anything older than TLSv1.3\. When I wrote the Second Edition of this book back in 2019, Apple Safari was the only major browser that didn’t support TLSv1.3\. Fortunately, even Apple is now on board with the newest TLS.

#### Hardening Apache SSL/TLS on Ubuntu

For this demo, we'll use two Ubuntu 22.04 virtual machines. We'll install Apache on the first one and `sslscan` on the second one. (This `sslscan` package isn’t available in the AlmaLinux repository.):

1.  To install Apache on your Ubuntu machine, just do the following:

```
sudo apt install apache2
```

This also installs the `mod_ssl` package, which contains the libraries and configuration files for SSL/TLS implementation.

And, of course, if you have a firewall enabled, be sure that port `443/tcp` is open.

1.  The Apache service is already enabled and running, so you don't have to mess with that. But you do need to enable the default SSL site and the SSL module with these three commands:

```
sudo a2ensite default-ssl.conf
sudo a2enmod ssl
sudo systemctl restart apache2
```

1.  Before we look at the SSL/TLS configuration, let's set up a scanner machine to externally test our configuration. On the second Ubuntu VM, install the `sslscan` package:

```
sudo apt install sslscan
```

On the scanner machine, scan the Ubuntu machine on which you installed Apache, substituting the IP address of your own machine:

```
sslscan 192.168.0.3
```

Note the algorithms and the protocol versions that are supported. You should see that SSLv2, SSLv3, TLSv1.0, and TLSv1.1 are all disabled. TLSv1.2 and TLSv1.3 are the only ones that are enabled.

1.  On the Ubuntu VM with Apache, edit the `/etc/apache2/mods-enabled/ssl.conf` file. Look for the line that says this:

```
SSLProtocol all -SSLv3
```

Change it to this:

```
SSLProtocol all -SSLv3 -TLSv1.2
```

1.  Restart the Apache daemon to make this change take effect:

```
sudo systemctl restart apache2
```

1.  Scan this machine again, and note the output. You should see that the older TLSv1.2 protocol has also now been disabled. So, congratulations! You've just made a quick and easy security upgrade to your web server.
2.  End of lab.

Now, let's take a look at RHEL 9/AlmaLinux 9.

#### Hardening Apache SSL/TLS on RHEL 9/AlmaLinux 9

For this demo, you'll install Apache and `mod_ssl` on an AlmaLinux 9 VM. (Unlike on Ubuntu, you have to install these as two separate packages.) Use the same scanner VM that you used in the previous lab. A new feature of the RHEL 8/9 distros is that you can now set system-wide crypto policies for most of your services and applications that require cryptography. We'll take a quick look at it here, and again in *Chapter 7*, *SSH Hardening*:

1.  Before doing anything, shut down your AlmaLinux 9 VM and create a snapshot from the VirtualBox console. That’s because in just a bit, you’ll need to go back to a clean snapshot in order to test the crypto policies feature.
2.  On your AlmaLinux 9 VM, install Apache and `mod_ssl`, and start the service:

```
sudo dnf install httpd mod_ssl
sudo systemctl enable --now httpd
```

1.  Open port `443` on the firewall:

```
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

1.  From the scanner VM, scan the Apache VM, substituting your own IP address:

```
sslscan 192.168.0.160
```

As you just saw on the Ubuntu server, nothing older than TLSv1.2 is supported.

1.  Next, on the Apache VM, view the status of the system-wide crypto configuration:

```
sudo update-crypto-policies --show
```

You should see `DEFAULT` as the output. With `DEFAULT`, you get TLSv1.2 as the minimum protocol version along with the goodness of TLSv1.3\. But you'll also see some TLSv1.2 algorithms that we can do without.

1.  Shut down the Apache VM. Go to the VirtualBox console and restore the snapshot that you created in Step 1, in order to get rid of the Apache installation. Then, restart the virtual machine and set the crypto policy to `FUTURE`, like this:

```
sudo update-crypto-policies --set FUTURE
```

> I had a good reason for having you create and restore the snapshot before setting `FUTURE` mode. It’s just that if you install Apache before setting `FUTURE` mode, you’ll no longer be able to start Apache. So, if you want to run your Apache webserver with `FUTURE` mode, you’ll need to set `FUTURE` mode first, then install Apache.

1.  Reboot the Apache VM so that the `FUTURE` mode will take effect. Verify that `FUTURE` mode has taken effect by doing:

```
sudo update-crypto-policies --show
```

1.  Install the `mod_ssl` and Apache packages, and start Apache as you did in Step 2.
2.  Scan the webserver VM as you did in Step 4\. You'll see that TLSv1.2 is still enabled, but with a much smaller list of enabled algorithms.
3.  End of lab.

There are two other crypto policy modes besides the two that I've shown here. `LEGACY` mode enables some really old algorithms that we don't want to use unless it's absolutely necessary to support older clients. But, as I keep saying, anyone who's using a client that's that old needs to upgrade. There's also the `FIPS` mode, which you might need to use if you’re doing business with the U.S. government. Even though the `update-crypto-policies` utility appears to work with `FIPS` mode, Red Hat recommends against doing that. Instead, they recommend setting `FIPS` mode as you install the operating system. We’ll look at that next.

#### Setting FIPS mode on RHEL 9/AlmaLinux 9

FIPS stands for Federal Information Processing Standards, and is a set of cybersecurity requirements for people and companies who want to do business with the United States government. Setting your server to run in `FIPS` mode involves more than just disabling some weak encryption algorithms. It also involves installing a set of modules that help harden other aspects of the operating system.

Even though the `update-crypto-policies` utility has an option for setting `FIPS` mode, you’ll never use it. To set `FIPS` mode on a machine on which the operating system has already been installed, you’d instead use the `sudo fips-mode-setup --enable` command. But, Red Hat even recommends against doing that. Instead, they recommend setting `FIPS` mode as you install the operating system. Their concern is that setting `FIPS` mode after installing the operating system might leave behind encryption keys that were created with non-`FIPS` algorithms. Instead, they recommend setting `FIPS` mode as you install the operating system. Fortunately, that’s easy. All you have to do is interrupt the installer’s boot process and make a quick edit to the kernel configuration. Here’s how to do it as you create a new AlmaLinux VM:

1.  Create a new AlmaLinux VM and boot up the AlmaLinux installer. Hit the **Up arrow** key to highlight the **Install AlmaLinux** option. Instead of hitting the **Enter** key to continue, hit the **Tab** key to bring up the kernel options. Here’s what you should see:

    ![](img/file47.png)
2.  At the bottom of the screen, add `fips=1` to the end of the kernel option line. It should now look like this:

    ![](img/file48.png)
3.  Hit the Enter key and continue installation as you normally would.
4.  Once the installation is complete and the VM has rebooted, check the status of `FIPS` mode, like this:

```
[donnie@localhost ~]$ fips-mode-setup --check
FIPS mode is enabled.
[donnie@localhost ~]$
```

1.  Finally, install `mod_ssl` and Apache. Open the the firewall port and scan the VM with `sslscan`, as you did in the previous exercise.
2.  End of lab.

There is one caveat that you should know about when setting `FIPS` mode on any RHEL 9-type distro. It’s that the current version of the FIPS standard is version 140-3\. However, at the time of this writing in October 2022, RHEL 9 and its offspring still only meet the standards for FIPS 140-2\. The Red Hat documentation gives no insight into when that might change.

> If you’re wondering why I’m not covering `FIPS` mode on Ubuntu, it’s just that it’s not possible to set `FIPS` mode on the free-of-charge version of Ubuntu. If you want to run Ubuntu in `FIPS` mode, you’ll have to purchase a support contract.

Now, let’s take a quick look at the legacy CentOS 7.

#### Hardening Apache SSL/TLS on RHEL 7/CentOS 7

Okay, I did say that we'd look at doing this on a CentOS 7 machine. But, I'll make it brief.

You'll install Apache and `mod_ssl` on CentOS 7 the same way that you did on AlmaLinux 9, except that you'll use the `yum` command instead of the `dnf` command. As with AlmaLinux, you'll need to enable and start Apache with `systemctl`, but you won't need to enable the ssl site or the ssl module. And, of course, make sure that port `443` is open on the firewall.

When you do an sslscan of a CentOS 7 machine, you'll see a very long list of supported algorithms, from TLSv1 through TLSv1.2\. Even with TLSv1.2, you'll see a few really bad things, like this:

```
Accepted  TLSv1.2  112 bits  ECDHE-RSA-DES-CBC3-SHA        Curve P-256 DHE 256
Accepted  TLSv1.2  112 bits  EDH-RSA-DES-CBC3-SHA          DHE 2048 bits
Accepted  TLSv1.2  112 bits  DES-CBC3-SHA 
```

The `DES` and `SHA` in these lines indicate that we're supporting use of the antiquated **Data Encryption Standard** (**DES**) and version 1 of the **Secure Hash Algorithm** (**SHA**). That is not good. Get rid of them by editing the `/etc/httpd/conf.d/ssl.conf` file. Look for these two lines:

```
SSLProtocol all -SSLv2 -SSLv3
SSLCipherSuite HIGH:3DES:!aNULL:!MD5:!SEED:!IDEA
```

Change them to this:

```
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite HIGH:!3DES:!aNULL:!MD5:!SEED:!IDEA:!SHA
```

Reload Apache with this command:

```
sudo systemctl reload httpd
```

Scan the machine again, and you'll see a lot fewer supported algorithms. (And by the way, one advantage of the new TLSv1.3 is that it completely gets rid of these legacy algorithms.)

Next, let's look at how users can identify themselves to a server.

### Setting up mutual authentication

When you access your bank's secure website, your web browser requires that the web server authenticate itself to the browser. In other words, the browser demands to see the server's certificate for the website so that it can verify if it's valid. This way, you have some assurance that you're logging in to the bank's real, genuine website instead of a counterfeit site. You then have to authenticate yourself to the web server, but you'll normally do that with a username and password.

If a web server is set up to allow it, users can instead authenticate themselves with a certificate. This way, there's no password for the bad guys to either steal or crack. You already saw how this is done when you imported Dogtag's `ca_admin_cert.p12` certificate into your web browser. This certificate gave you the awesome power to access Dogtag's administrator page. Your normal end users won't have this certificate, so all they can access is just the end-user page where they can request certificates.

The major web servers—Apache, Nginx, lighttpd, and some others—support mutual authentication. Space doesn't permit me to go into the details of setting this up on a server, but the documentation for whichever server you use will cover it.

Next, let’s get back to the future!

## Introducing quantum-resistant encryption algorithms

You’ve likely heard of quantum computers, which have nothing to do with the old *Quantum Leap* show on television. This new type of computer is still in the experimental stage, and likely will remain there for some time to come. Still, there’s a lot of hype about what they’ll be like when they finally are ready for production use. Supposedly, they’ll be way more powerful than the current generation of computers, and they’ll supposedly be able to easily crack even the strongest of the current encryption algorithms. Indeed, that’s a rather scary prediction. (Perhaps it’s fitting that I’m typing this on Halloween, the scariest day of the year.)

Even though there’s some skepticism about whether this dire prediction will come true, or of whether production-grade quantum computers will even see the light of day, the U.S. federal government is taking this seriously. Here’s the list of quantum-resistant algorithms that the National Institutes of Standards and Technology (NIST) currently recommends:

*   **CRYSTALS-Kyber**: This one is for general encryption. Cloudflare, Amazon, and IBM already use it.
*   **CRYSTALS-Dilithium**: This is for encrypted digital signatures. NIST recommends this one as the primary signature algorithm.
*   **FALCON**: This is also a signature algorithm. NIST recommends it for whenever you need a signature that’s smaller than what CRYSTALS-Dilithium can provide.
*   **SPHINCS+**: This is the third signature algorithm, which is slower and larger than the first two. It uses a different approach than what the first two use, which is why NIST recommends it as a backup, in case the first two get hacked.

So, how do we know that a particular encryption algorithm is resistant to quantum computer hacking if viable quantum computers don’t yet exist? Well, I wish that I could tell you, but I can’t. At any rate, you might not have to worry too much abut this just yet, but it’s still worthwhile to learn.

Okay, let’s wrap this baby up and move on to the next chapter.

## Summary

As always, we've covered a lot of ground in this chapter. We began by using GPG to encrypt, sign, and share encrypted files. We then looked at various methods of encrypting drives, partitions, directories, and sharable containers. After that, we looked at how to use OpenSSL to create keys, CSRs, and certificates. But since we don't want to use self-signed certificates all the time and commercial certificates aren't always necessary, we looked at how to set up a private CA with Dogtag. We then looked at simple ways to harden the TLS configuration on the Apache web server, and we touched on the subject of mutual authentication. Finally, we saw an introduction to quantum-resistant encryption algorithms.

Along the way, we had plenty of hands-on labs. That's good, because after all, idle hands are the devil's workshop, and we certainly don't want any of that.

In the next chapter, we'll look at ways to harden Secure Shell. I'll see you there.

## Questions

1.  Which of the following is not an advantage of GPG?
    1.  It uses strong, hard-to-crack algorithms.
    2.  It works well for sharing secrets with people you don't know.
    3.  Its public/private key scheme eliminates the need to share passwords.
    4.  You can use it to encrypt files that you don’t intend to share, for your own personal use.
2.  You need to send an encrypted message to Frank. What must you do before you can encrypt his message with GPG so that you don't have to share a password?
    1.  Nothing. Just encrypt the message with your own private key.
    2.  Import Frank's private key into your keyring and send Frank your private key.
    3.  Import Frank's public key into your keyring and send Frank your public key.
    4.  Just import Frank's public key into your keyring.
    5.  Just import Frank's private key into your keyring.
3.  Which of the following would be the proper choice for whole-disk encryption on a Linux system?
    1.  Bitlocker
    2.  VeraCrypt
    3.  eCryptfs
    4.  LUKS
4.  If you use eCryptfs to encrypt users' home directories and you're not using whole-disk encryption, what other action must you take in order to prevent leakage of sensitive data?
    1.  None.
    2.  Ensure that users use strong private keys.
    3.  Encrypt the swap partition.
    4.  You must use eCryptfs in whole-disk mode.
5.  In which of the following scenarios would you use VeraCrypt?
    1.  Whenever you want to implement whole-disk encryption.
    2.  Whenever you just want to encrypt users' home directories.
    3.  Whenever you'd prefer to use a proprietary, closed source encryption system.
    4.  Whenever you need to create encrypted containers that you can share with Windows, macOS, and BSD users.
6.  You need to ensure that your web browser trusts certificates from the Dogtag CA. How do you do it?
    1.  You use `pki-server` to export the CA certificate and key, and then use `openssl pkcs12` to extract just the certificate. Then, import the certificate into your browser.
    2.  You import the `ca_admin.cert` certificate into your browser.
    3.  You import the `ca_admin_cert.p12` certificate into your browser.
    4.  You import the `snakeoil.pem` certificate into your browser.

## Further reading

Explanations about TLS and OpenSSL:

*   OpenSSL Tutorial-How do SSL certificate, private keys, and CSRs work?: [https://phoenixnap.com/kb/openssl-tutorial-ssl-certificates-private-keys-csrs](https://phoenixnap.com/kb/openssl-tutorial-ssl-certificates-private-keys-csrs)
*   Transport Layer Security version 1.3 in Red Hat 8: [https://www.redhat.com/en/blog/transport-layer-security-version-13-red-hat-enterprise-linux-8](https://www.redhat.com/en/blog/transport-layer-security-version-13-red-hat-enterprise-linux-8)
*   The OpenSSL website: [https://www.openssl.org/](https://www.openssl.org/)
*   Feisty Duck Publishing, who offer books, training, and newsletters about OpenSSL: [https://www.feistyduck.com/](https://www.feistyduck.com/)

Problems with EV certificates:

*   Chrome browser moving EV UI to Page Info: [https://chromium.googlesource.com/chromium/src/+/HEAD/docs/security/ev-to-page-info.md](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/security/ev-to-page-info.md)
*   Extended Validation is Broken: [https://www.cyberscoop.com/easy-fake-extended-validation-certificates-research-shows/](https://www.cyberscoop.com/easy-fake-extended-validation-certificates-research-shows/)
*   EV Certificates issued with "Default City" as the location: [https://groups.google.com/forum/#!topic/mozilla.dev.security.policy/1oReSOPCNy0](https://groups.google.com/forum/#!topic/mozilla.dev.security.policy/1oReSOPCNy0)
*   EV certificates issued with erroneous information: [https://twitter.com/Scott_Helme/status/1163546360328740864](https://twitter.com/Scott_Helme/status/1163546360328740864)

Problems with free Let's Encrypt certificates:

*   CyberCriminals abusing free Let's Encrypt certificates: [https://www.infoworld.com/article/3019926/cyber-criminals-abusing-free-lets-encrypt-certificates.html](https://www.infoworld.com/article/3019926/cyber-criminals-abusing-free-lets-encrypt-certificates.html)

Dogtag CA:

*   How to increase the number of file descriptors in Linux: [https://www.tecmint.com/increase-set-open-file-limits-in-linux/](https://www.tecmint.com/increase-set-open-file-limits-in-linux/)
*   Dogtag PKI Wiki: [https://www.dogtagpki.org/wiki/PKI_Main_Page](https://www.dogtagpki.org/wiki/PKI_Main_Page)
*   Import CA into Linux and Windows: [https://thomas-leister.de/en/how-to-import-ca-root-certificate/](https://thomas-leister.de/en/how-to-import-ca-root-certificate/)
*   Red Hat (Dogtag) Certificate Authority Documentation: [https://access.redhat.com/documentation/en-us/red_hat_certificate_system/9/](https://access.redhat.com/documentation/en-us/red_hat_certificate_system/9/)

RHEL 9/AlmaLinux 9:

*   Setting system-wide cryptographic policies: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/using-the-system-wide-cryptographic-policies_security-hardening)

FIPS

*   FIPS home page: [https://www.nist.gov/itl/fips-general-information](https://www.nist.gov/itl/fips-general-information)
*   Installing Red Hat in FIPS mode: [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_installing-the-system-in-fips-mode_security-hardening](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/assembly_installing-the-system-in-fips-mode_security-hardening)

Quantum-resistant encryption

*   NIST announces quantum-resistant algorithms: [https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms](https://www.nist.gov/news-events/news/2022/07/nist-announces-first-four-quantum-resistant-cryptographic-algorithms)
*   Some people are skeptical about quantum-computing: [https://www.fudzilla.com/news/55434-quantum-computing-is-neither-dead-or-alive](https://www.fudzilla.com/news/55434-quantum-computing-is-neither-dead-or-alive)

## Answers

1.  B
2.  C
3.  D
4.  C
5.  D
6.  A