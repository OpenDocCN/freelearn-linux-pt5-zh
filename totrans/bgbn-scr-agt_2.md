# Chapter 2. Circumventing Censorship with a Tor Bridge

In this chapter, you'll configure your **BeagleBone Black** (**BBB**) to run a bridge in the Tor network. This bridge will allow you and others to access the Internet more anonymously and provide an anti-censorship gateway. We'll add a simple hardware control interface to BBB so that we can see and adjust the bandwidth usage of the bridge in real time. We'll call this project BeagleBridge.

This chapter will discuss the following topics:

*   An introduction to Tor
*   The difference between a Tor relay and bridge
*   Obfuscated Tor proxies
*   How to download and install Tor on BBB
*   How to configure BBB as a Tor bridge running an obfuscated proxy
*   How to add hardware controls to adjust the bridge from a front panel

# Learning about Tor

In this project, you will learn how to use Tor, a tool and network designed to protect your anonymity online. Tor originally developed from research, sponsored by the U.S. Naval Research Laboratory, on **onion routing** (Dingledine, Mathewson, and Syverson, 2004). In onion routing, the client builds a circuit of nodes in an overlay network, which is a network built on top of an existing network. The Tor network is an overlay network that runs on the Internet, although it can run on separate networks. The client sends a message to each node, which is specifically encrypted for that node, asking the node to send it to the next node in the circuit. Each node peels back a layer of encryption and forwards the result to the next hop in the circuit, and hence, the **onion analogy**. The last node contains the client's actual message, which is forwarded to the destination server.

Onion routing provides anonymity because the destination server does not know the IP address of the client. Typically, when you use your browser to access the Internet, the browser creates a **Transmission Control Protocol** (**TCP**) connection that originates from your system and terminates at the website you are trying to visit. The address for TCP is provided by the **Internet Protocol** (**IP**). Each IP datagram contains a source and destination IP address. As datagrams arrive at the server, the server can read the source IP address. This is generally useful as the server needs this address to return your data. However, this also means that the server knows your IP address, which is where you live on the Internet. Your IP address alone reveals information about you, such as the country in which you live and your **Internet Service Provider** (**ISP**). Geolocating by an IP address can be accurate to the zip code level, when you use United States as an example.

### Note

**Tor**, originally an acronym for **The Onion Router**, is now simply referred to as *Tor*, not *TOR*. When you ask questions on the Tor mailing list or IRC channels, Tor developers will appreciate it if you make note of this subtlety.

The following diagram shows this routing. In this case, Alice is a client who connects to the first node of her circuit, which happens to be running on your BBB. This BBB unwraps a layer and forwards Alice's communication to the middle node. The middle node does the same to the final (exit) node. The exit node sends Alice's original message to the destination server named Bob. The green arrows show internal Tor connections that are encrypted. The connection from the exit node to Bob is shown as an unencrypted connection because this traffic is not part of the Tor network. From Bob's perspective, the IP originator of this connection is the exit node. However, the true originator is Alice whose IP is hidden by the Tor network.

![Learning about Tor](img/00006.jpeg)

The response to this information by many individuals is:

> *"So what? Why do I care if the website I'm visiting knows my IP address?"*

The website, which knows the data you requested, now knows your location. This information could be combined with other public information, for example, to conduct a **linkage attack**. A linkage attack is an attempt at deanonymization by combining information from multiple sources. For example, let's say you are searching for information on a rare medical condition. You post a question on a forum, under a pseudonym, asking for information. However, the website, and anybody passively monitoring your connection (if it wasn't encrypted), knows your general location as well as your question. Your medical condition is most prevalent in Hispanic females in their seventies. Using your IP address combined with knowledge about your medical condition, one can conduct a linkage attack possibly using public census data to reveal your identity.

Tor protects against this kind of attacks by encrypting your traffic through the Tor network and masking your IP address. To the remote website, your IP address will be that of the last node in the Tor network, known as an exit relay.

## Appreciating the various users of Tor

A wide range of people use Tor for daily Internet access. Individuals who don't want their ISP to collect information on the websites they visit, perhaps because they are seeking information on sensitive topics, use Tor. Government agents and the military use Tor. After all, it's difficult to go undercover online if the IP address you are connecting FROM resolves to `fbi.gov`. Tor is used by whistleblowers, such as Edward Snowden, to disclose information to the press. It is also used by normal citizens when a government blocks access to the Internet. In late March 2014, when the Turkish government attempted to ban access to Twitter, the number of Tor users in Turkey jumped from around 25,000 to just under 70,000 in about two weeks.

### Note

Tor publishes, via a wide range of interactive graphs, numerous metrics on who is using Tor and where they're using it from. The users spike in Turkey can be seen in the graph available at [https://metrics.torproject.org/users.html?graph=userstats-relay-country&start=2014-01-04&end=2014-04-04&country=tr&events=off#userstats-relay-country](https://metrics.torproject.org/users.html?graph=userstats-relay-country&start=2014-01-04&end=2014-04-04&country=tr&events=off#userstats-relay-country).

## Understanding Tor relays

**Tor relays** are the individual nodes in the Tor network. A node is some type of computer that is running the Tor software. These relays route messages throughout the network. Exit nodes, which are the last nodes in the circuit, send the clients' messages to the destination service. One of the interesting aspects of Tor is that most of the nodes are run by volunteers. These relay operators volunteer their bandwidth, time, and energy bill to increase the capacity of the Tor network. The Tor Project encourages individuals to run relays in order to add to the diversity of the network. The motivation behind this recommendation is that it is easier to hide in a crowd—it's difficult to hide in a crowd of one person.

## Understanding Tor bridges

In 2006, in response to an increased Internet censorship, the Tor project added an anticensorship feature called a bridge relay or simply, a bridge (Dingledine, 2006). A bridge is a special form of a relay—it's not listed in the public relay directory. Strong Internet censors were able to block access to Tor through several methods. One method that was effective was to look up all the public relays, which were only a thousand or so at the time, and deny access to those IP addresses. Therefore, bridge relays were created and distributed by less public means in order to allow users to access the Tor network. If a client could access the bridge, which was already connected to Tor, then the client could access the Internet. In this context, less public means a distribution mechanism that does not work via the public relay list.

## Using obfuscated proxies and pluggable transports

Censors were still able to block access to Tor; the technique was to identify the traffic patterns generated by Tor and block the connection. In 2010, Jacob Appelbaum and Nick Mathewson of the Tor Project suggested a method to obfuscate the Tor traffic between the client and the bridge in order to thwart deep packet inspection. As the obfuscation mechanism might need to change, and has changed in fact, the Tor Project wanted a generic protocol to allow different obfuscated proxies to run. This abstraction was known as **pluggable transport**.

In this chapter, we will set up your BBB to run as a Tor bridge, running an obfuscated proxy, using the obfs3 pluggable transport.

## Realizing the limitations of Tor

Tor is one of the best tools currently available to protect anonymity. However, like all the security tools discussed in this book, there are limits to its protection. First of all, Tor's threat model does not hold up to a global passive adversary. Such an adversary can passively monitor the entire Internet. If an adversary can monitor the entire Internet, then it can correlate the traffic entering the Tor network with the traffic leaving the network and can possibly deanonymize Tor clients. One of the trade-offs of this design is that Tor is a low-latency system, meaning that you can access the Internet using normal protocols such as HTTP without experiencing much delay. This is one of the reasons why diversity in the Tor network is important. If all the relays were in a particular country, it might be easier for that country to monitor the traffic. However, currently it is thought to be difficult, in spite of the recent leaks provided by Edward Snowden, for such an adversary to monitor the entire Internet.

### Note

Roger Dingledine of the Tor Project commented on NSA's exploits of Tor with the following:

*"The good news is that they [the NSA] went for a browser exploit, meaning there's no indication that they can break the Tor protocol or do traffic analysis on the Tor network. Infecting the laptop, phone, or desktop is still the easiest way to learn about the human behind the keyboard."*

The full text and additional commentary is available on the Tor website ([https://blog.torproject.org/blog/yes-we-know-about-guardian-article](https://blog.torproject.org/blog/yes-we-know-about-guardian-article)).

Also, Tor will not automatically encrypt all your traffic. If you request information over unencrypted HTTP, the Tor exit node you use will relay this unencrypted information to its destination. A malicious exit node can monitor or manipulate your traffic; therefore, it's always best to use encrypted sessions, such as HTTPS, over Tor. Tor does not protect all your traffic just because you are running Tor. Applications generally need to be configured to use Tor. Even if you set up a transparent proxy on your home network and route all of your traffic through that proxy, the malware or exploits in your browser may leak your identity. This is why the Tor Project recommends that you use the Tor Browser, which essentially is a forked version of Mozilla Firefox that has been patched especially to not leak your identity. Lastly, Tor can't protect your identity if you choose to reveal it. If you decide to log in to Facebook over Tor, you've just told Facebook who you are and that you are using Tor since all the Tor exit nodes are public.

### Tip

An easy way to use Tor more securely is to use it only from Tails ([https://tails.boum.org/](https://tails.boum.org/)), which is a custom Linux distribution that will boot from various media. Tails is a collection of free software and includes numerous correctly configured tools to help protect your confidentiality and anonymity.

## The impact and benefits of running a Tor bridge

So, why run a Tor bridge on BBB? The impact and benefits of Tor is in the network. The more the Tor servers, the more the resources in the network. Many users in developed nations have high-speed Internet connections that are orders of magnitude faster than in countries where access to censor-free Internet is restricted. A bridge is likely to receive less traffic than a relay, as there are fewer bridge users than normal Tor users. Most likely, the limiting performance factor for your bridge will be your home network's upload speed. This can be a constraining factor if you are running a relay, but as a bridge, you are most likely helping those in precarious Internet situations and any donated bandwidth is appreciated. Lastly, running a bridge on BBB has the extra advantage of a low impact on your electric bill, as BBB will only draw about 460mA when loading a web page according to the BeagleBone Black System Reference Manual.

# Installing Tor on BBB

The instructions provided in the following sections are geared towards the user running BeagleBridge on a home network. The bridge will consume some otherwise unused bandwidth and donate it to the Tor network. You should check your ISP's Terms of Service before running a server to see whether it's permitted. Also, you'll need to configure port forwarding from your home router. As there are numerous devices, each with their own configuration mechanism, you should consult your router's manual on how to enable port forwarding.

## Installing Tor from the development repository

The Tor images in the official Debian repository are not as up to date as those from the Tor Project. We'll use the Tor Project's development repository to retrieve the latest software. This is especially important when you are running a bridge, as the bridge and the pluggable transport software are updated frequently.

### Note

The latest instructions as well as the latest GPG fingerprint can be found on the Tor Project's website ([https://www.torproject.org/docs/debian](https://www.torproject.org/docs/debian)). The following steps explain the installation procedure, but you should cross-reference them with the published instructions.

Edit `/etc/apt/sources.list` by adding the following lines:

```
deb http://deb.torproject.org/torproject.org wheezy main
deb http://deb.torproject.org/torproject.org tor-experimental-0.2.5.x-wheezy main

```

Next, add the GPG key used to sign these Tor packages:

```
gpg --keyserver keys.gnupg.net --recv 886DDD89
gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -

```

Then, issue the following command:

```
sudo apt-get update

```

The Tor Project recommends that you add the GPG key ring with the following command:

```
sudo apt-get install deb.torproject.org-keyring

```

Tor needs an up-to-date time as it enforces the time validity on certificates. In a later chapter, we'll show you how to keep time with a dedicated **Real Time Clock** (**RTC**). For now, update your clock from the **Network Time Protocol** (**NTP**) as follows:

```
sudo ntpdate -b -u pool.ntp.org

```

Install Tor:

```
sudo apt-get install tor

```

Then, install `obfsproxy`. Obfsproxy is the software that implements the obfuscated proxy and allows various pluggable transports. Obfsproxy uses Python Twisted library, an event-driven networking engine, which will install around 17 MB of packages in total:

```
sudo apt-get install obfsproxy

```

While we are installing software, let's install the Stem Python package. Stem is a Python controller library for Tor, and we'll be using it later to interact with our bridge. The easiest method is to install it with `pip`:

```
sudo pip install stem

```

## Configuring Tor for BBB

Under Debian, the configuration file for Tor is `/etc/tor/torrc`. Before editing `/etc/tor/torrc`, you should first take a backup. This `torrc` file is available for download at [https://github.com/jbdatko/beagle-bone-for-secret-agents/blob/master/ch2/torrc](https://github.com/jbdatko/beagle-bone-for-secret-agents/blob/master/ch2/torrc). We will discuss more interesting aspects of this configuration file in the following sections. When you are ready, replace `/etc/tor/torrc` with the following:

```
# We are running a relay, no need for the SocksPort
SocksPort 0
# Extra logging is nice
Log notice file /var/log/tor/notices.log
# Run in the background
RunAsDaemon 1
# The following two lines are so we can connect with the
## Tor Stem library over the control port
ControlPort 9051
CookieAuthentication 1
# The is the Onion Router (OR) Port for normal relay operation
ORPort 9001
# Your bridge's nickname, change!
Nickname changeme
# Bandwidth settings
RelayBandwidthRate 520 KB # Throttle traffic to 520 KB/s
RelayBandwidthBurst 640 KB # But allow burts up to 640 KB/s
# You put a real email here, but consider making a new account
## or alias address
ContactInfo Random Person <nobody AT example dot com>
# Do not exit traffic
ExitPolicy reject *:* # no exits allowed
# Yes, we want to be a bridge
BridgeRelay 1
# Use the obfs3 pluggable transport
ServerTransportPlugin obfs3 exec /usr/bin/obfsproxy managed
# Enable extra bridge statistics collection
ExtORPort auto
# A nice option for embedded platforms to minimize writes
# to eMMC or SD card
AvoidDiskWrites 1
```

### Adding contact details to the torrc file

At minimum, you should change the `Nickname` field and `ContactInfo`. The `Nickname` field is a shorter way to refer to your bridge; however, your bridge's fingerprint is always the best method as it is unique. The `ContactInfo` field allows the Tor project to send you an e-mail if there is a problem with your bridge. You can create an e-mail alias if you are concerned about receiving spam. Just be sure to monitor this account for infrequent e-mails from the Tor project.

### Tuning the bandwidth usage of your bridge

Tor's man page will describe most of these settings in detail, but some warrant extra explanation. The bandwidth settings, `RelayBandwidthRate` and `RelayBandwidthBurst`, are tunable bandwidth settings, and in a later section, we will connect our hardware controls to manipulate these settings. The rate and the burst are in kilobytes per second, not in the more common kilo or *megabits* per second, so watch your units.

# Understanding Tor exit policies

A bridge, by definition, is the entry point to the Tor network. As such, the exit policy, which will allow traffic to exit the Tor network from the server, should be the following:

```
ExitPolicy reject *:*

```

This prevents your server from running as an exit node. If you do decide to run an exit node, be prepared to receive some complaints from your ISP if you are running it on a home network. This is why the Tor Project and the Electronic Frontier Foundation recommend that you *don't* run an exit relay on a home network. A thorough, legal FAQ prepared by the Electronic Frontier Foundation can be found at [https://www.torproject.org/eff/tor-legal-faq.html.en](https://www.torproject.org/eff/tor-legal-faq.html.en).

# Setting bridge-specific settings

There are three bridge specific settings: `BridgeRelay`, `ServerTransportPlugin`, and `ExtORPort`. The `BridgeRelay` setting is the key setting that defines your relay as a bridge. Your bridge's meta information is published in the bridge database instead of the public directory server, which keeps your bridge's IP less public than a Tor relay's IP address. `ServerTransportPlugin` defines which pluggable transport proxy your bridge supports. Currently, ScrambleSuit is the latest promising pluggable transport technology. However, obfs3, which is the transport enabled in our bridge configuration example, is slightly more mature and it is the more conservative recommendation. Lastly, `ExtORPort` allows the gathering and reporting of bridge statistics to the Tor Project.

### Note

For those who are interested in running the ScrambleSuit obfsproxy, take a look at the following link on how to configure your bridge: [https://lists.torproject.org/pipermail/torrelays/ 2014-February/003886.html](https://lists.torproject.org/pipermail/torrelays/%202014-February/003886.html).

# Starting your new Tor bridge

With the time updated and the configuration set, it's time to turn on the bridge. At the moment, the bridge should be able to make a connection to the Tor network, but it will not be able to accept incoming connections as we have not yet configured port forwarding from your router. However, the `obfsproxy` port is randomly assigned, so we need to run the bridge first to find the port. Restart the Tor service with the following command:

```
sudo service tor restart

```

Next, let's check the log to see whether Tor has started correctly:

```
tail -n 20 /var/log/tor/notices.log

```

If you see something like the following, then your Tor client's behavior is working:

```
Mar 25 21:37:43.000 [notice] Tor has successfully opened a circuit. Looks like client functionality is working.
Mar 25 21:37:43.000 [notice] Bootstrapped 100%: Done.

```

# Enabling port forwarding

We know that we need to forward port `9001`, as it is the ORPort, but we need to know which port the obfsproxy software runs on. This will be logged in the same file and will be discovered by searching the Tor log with the following command:

```
grep obfs3 /var/log/tor/notices.log

```

The previous command should yield the following search result:

```
Mar 05 01:56:04.000 [notice] Registered server transport 'obfs3' at '0.0.0.0:59519'

```

The `obfsproxy` port for our obfs3 service is on `59519`. From your home router, configure port forwarding from `9001`, and configure port forwarding from `59519` from your external IP to BBB. It will also help if you give your BBB a static internal IP. Consult your router's manual for directions. Alternatively, you can specify the port with the following line in the `/etc/tor/torrc` file:

```
ServerTransportListenAddr obfs3 0.0.0.0:xxxx
```

Replace the x's with the desired port address. However, it's best to let obfsproxy pick a random address; otherwise, the Tor Project might end up with an uneven distribution of bridges running on certain ports, which will make it easier to block access to bridges.

Once you've forwarded the necessary ports, restart the router again. You should see the following messages, indicating success:

```
Mar 25 21:37:43.000 [notice] Now checking whether ORPort xxx.xxx.xxx.xxx:9001 is reachable... (this may take up to 20 minutes -- look for log messages indicating success)
Mar 25 21:37:44.000 [notice] Self-testing indicates your ORPort is reachable from the outside. Excellent. Publishing server descriptor.

```

Congratulations! You are now running a Tor bridge on BBB and are helping to improve Internet freedom. You are also enabling agents, both secret and otherwise, to access the unfiltered Internet.

# Adding physical interfaces to the bridge

Now you have a Tor bridge running and you can stop here. If you do, you'd be missing out on the ability to combine software with custom hardware on BBB. Our BBB Tor bridge currently has no visual feedback, so it's not obvious that it's working. Also, the only means to control the bridge is to log in to BBB over SSH and manipulate the configuration options. The Tor bridge is an appliance and it needs appliance controls. In this section, we'll add a front panel, which will give us an easy method to control the bridge's bandwidth and a quick indicator to know that the software hasn't crashed. In the following section, we'll add the software to interface with our bridge and control the hardware.

### Note

If you decide to run a Tor relay, there are websites such as Tor atlas ([https://atlas.torproject.org/](https://atlas.torproject.org/)) that will produce bandwidth graphs and display other information about your relay. Another tool that will also display information about your bridge is Globe ([https://globe.torproject.org/](https://globe.torproject.org/)).

## Gathering the front panel components

As this is our first project, we are going to use some basic components: a **light-emitting diode** (**LED**), a rotary potentiometer, and a **liquid-crystal display** (**LCD**), all shown in the following circuit diagram:

![Gathering the front panel components](img/00007.jpeg)

LCDs can been a bit tricky to work with, but SparkFun Electronics has combined a 16x2 LCD with a microcontroller so that it can use a serial interface instead of a more complicated parallel one. The serial interface is simpler because it only requires one data line to the device, whereas a parallel interface will require more wiring.

## Using an LCD to display status information

Our Tor bridge generates a lot of metadata, such as the bandwidth usage, the number of connections, and some Tor-specific statistics. For a home network, it's useful to know how much bandwidth your bridge is using, and it would be nice to know this without logging in to BBB all the time. In the serial LCDs, we will draw a graphical representation of the bandwidth usage. We'll use ten bars, each representing a tenth of the available bandwidth. If you see five bars, then the bridge is using half of the available bandwidth. The LCD selected for this project was LCD-09067 (SparkFun Electronics).

### Note

SparkFun Electronics is an international distributor that ships products from Boulder, Colorado, in the United States. Some of the more common components recommended in this book can be replaced with equivalent ones in your local electronic store if you are trying to save on shipping. You don't need to type in each component; they are all listed in one consolidated *wish list* at [https://www.sparkfun.com/wish_lists/93119](https://www.sparkfun.com/wish_lists/93119).

## Controlling the bandwidth with a potentiometer

A potentiometer is like a variable resistor. If you've ever adjusted the volume of a radio or speaker with a knob, you've probably used a potentiometer. By adjusting the knob, you adjust the resistance in the potentiometer, which in turns adjusts the voltage sensed at the output. The potentiometer has three leads: one for voltage (in), one for ground, and the final is the output.

This knob, connected to BeagleBridge, will throttle the bandwidth available to Tor. With the dial turned up to max, the bridge will report that all of your bandwidth is available to route traffic to the Tor network. With the dial at its midpoint, the bridge will report that half of your bandwidth is available for use. If you notice that the bridge is consuming more bandwidth than you'd like, as indicated by the LCD, you can turn down the *volume* of your bridge. The potentiometer and knob for this project are COM-09939 and COM-10001 (SparkFun Electronics).

Your bridge will not immediately attract users; it will take some time. Start with your bridge set at the max bandwidth; if you discover that it is consuming more bandwidth than you like, then turn it down. To receive an indication whether the Tor bridge is working, we will use an LED controlled by a BBB's GPIO, which will flash periodically. This way you can look over at the panel and tell whether the software is still running. The LED by SparkFun is COM-10633, and the LED holder is COM-11148.

## Designing the BeagleBridge circuit

The following Fritzing diagram shows the schematic for this project.

![Designing the BeagleBridge circuit](img/00008.jpeg)

The potentiometer is connected via BBB's **Analog-to-Digital Converter** (**ADC**) pins. BBB's analog inputs can only tolerate a max of 1.8V; so, it's very important to use the dedicated analog voltage pin, pin P9_32 (pin number 32 of the connector P9), which provides this voltage. Do *not* connect the potentiometer to the normal 3.3V or 5V power rails, as this will damage the processor. We'll arbitrarily pick the analog input AIN5 on pin P9_36 as our input pin, and connect that to the middle lead or output of the potentiometer. Lastly, connect the last lead of the potentiometer to the dedicated analog ground pin, P9_34.

The LED and LCD both use 3.3V and the common ground. We need a serial transmit pin for the LCD, so we'll arbitrarily pick P9_13, which is the transmit for UART4\. UART1 transmit on P9_24 or UART2 on P9_21 will also work fine.

Lastly, we need a GPIO to control the LED. Again, any will do, but I've picked pin P9_15\. You'll need a resistor to limit the current to the LED. Sizing a resistor is straightforward using Ohm's Law; we just need to know the forward voltage drop and the max current of the LED. This information is found in the datasheet. If you are using SparkFun's 5mm Green LED, its max current is 20mA and has a forward voltage drop of 2.2V.

Ohm's Law states that voltage is equal to the current multiplied by the resistance, or *V = IR*. Subtracting 2.2V from 3.3V of our supply voltage, and dividing the resulting value by .020 Amps gives 55 Ohms. 55 Ohms isn't a standard value, but 56 is, so you can use this. But, you can always use a higher resistance value, and the LED will just be less bright. The resistor used in this project had a value of 100 Ohms.

## Wiring the hardware with a proto cape

There are some special considerations when wiring this project as it is meant to go inside an enclosure to provide the front panel. A stranded wire will bend and flex better compared to a solid core. Once settled in the enclosure, we wouldn't expect the wires to strain or move around, but they certainly will when you try to install the panel inside the enclosure! We also have a few options to wire our project to BBB. We could use jumper wires with male pins to insert into the female expansion headers, but these pins can come out easily, especially when you put your project into the enclosure.

We'll use SparkFun's BeagleBone Black Proto Cape, DEV-12774, to easily combine our circuit with BBB. This board breaks out power, ground, and other pin signals for expansion. It includes an EEPROM, which is not only useful if you want to have persistent data stored on your board but is also required if you are building a BeagleBone Cape, which will be discussed in detail in the next chapter. The advantage of a protoboard, especially the Beagle proto capes, is that we can solder male headers on the board and then use female terminated wires, which tend to connect a little better. Plus, by using the male headers, we can easily reuse the protoboard for a later project. However, these protoboards are not required, and if you are trying to save some money, with some creative wiring, you don't need one.

If you are using the protoboard approach, you don't have to populate all the headers either. Only the male pins that we need are soldered to the corresponding pads on the protoboard's P9 header. The way these protoboards work is that each pin is duplicated on the pads next to them. Solder the 100 Ohm resistor from P9_15 to the bottom of one of the male pins in the middle of the board. Then, connect a wire from the male pin to the positive terminal of the LED.

When complete, your project should look something like the following circuit diagram. We don't have the control software running yet, but you can see the LCD displaying the bandwidth usage. The following diagram doesn't have soldered connections to the potentiometer for the ease of testing, but instead uses *IC Hooks*. In a finished installation, you will want to solder the wires for a better mechanical and electrical connection.

![Wiring the hardware with a proto cape](img/00009.jpeg)

## Developing the software using Python libraries

The software for this project is freely available for download at [https://github.com/jbdatko/beagle-bone-for-secret-agents/](https://github.com/jbdatko/beagle-bone-for-secret-agents/). In this section, we'll highlight the finer details of working with the existing libraries and outline the architecture of the project. The code uses many asynchronous callbacks from these libraries since there are several concurrent activities: the Tor bridge, the bandwidth knob, writing to the LCD, and blinking of the LED. To interact with the bridge, we'll use the Tor `Stem` library ([https://stem.torproject.org/](https://stem.torproject.org/)), and to interact with the hardware, we'll use Adafruit's BeagleBone Python library ([https://github.com/adafruit/adafruit-beaglebone-io-python](https://github.com/adafruit/adafruit-beaglebone-io-python)).

## Controlling the hardware with pyBBIO

We installed the `Stem` library when we installed Tor; so, now we need to install the `Adafruit` BBIO library by performing the following commands:

```
sudo apt-get install build-essential python-dev python-setuptools python-pip python-smbus -y
sudo pip install Adafruit_BBIO
sudo pip install pyserial

```

The `Adafruit` library conveniently enables the device tree overlays, which are the files that describe the hardware configuration of BBB to the kernel, at runtime. Therefore, there is no need to manipulate the configuration via the `sysfs` as everything is handled in the Python library. The code models each hardware component as its own Python class. The LED modeled by the `TorFreedomLED` class is the most straightforward. We need the LED to blink; this is accomplished by toggling the output high, sleeping the executing thread briefly, and then toggling the output low to turn it off. To set up the GPIO, we only need to call `GPIO.setup`, by passing in the pin and the direction:

```
import Adafruit_BBIO.GPIO as GPIO
class TorFreedomLED(object):
  def __init__(self):
    self.pin = 'P9_15'
    GPIO.setup(self.pin, GPIO.OUT)

  def on(self):
    GPIO.output(self.pin, GPIO.HIGH)

  def off(self):
    GPIO.output(self.pin, GPIO.LOW)

  def blink(self):
    self.on()
    sleep(.5)
    self.off()
```

The LCD, controlled by the `FrontPanelDisplay` class, writes to the serial port, `/dev/ttyO4`, on BBB's UART 4 at 9600 baud. The LCD only receives information; therefore, we are only using the transmit lines from BBB. Commands are written to the attached microcontroller, which in turn drives the display. The datasheet, available on the SparkFun website, describes all of the commands. In general, the procedure is to move the virtual cursor on the display and then send the text.

### Note

SparkFun has a more complete quick start guide on the serial LCD at [https://www.sparkfun.com/tutorials/246](https://www.sparkfun.com/tutorials/246). The example code is for an Arduino, but porting it to Python is straightforward if you follow the example of the bridge LCD in this chapter.

Similar to the GPIO example, the serial port in the `Adafruit` library can be used as follows:

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

The display will automatically wrap lines if the text you are trying to display is longer than sixteen characters. Therefore, you have to manage the LCD to ensure that it displays correctly. For example, let's take a look at the `display_graph` method that is responsible for producing the bandwidth graph:

```
self.clear_screen()
up_str = '{0:<16}'.format('Up:   ' + self.block_char * up)
dn_str = '{0:<16}'.format('Down: ' + self.block_char * down)

self.port.write(up_str)
self.port.write(dn_str)
```

The first line clears the screen and resets the cursor to the top-left corner of the 16x2 display. Next, we format the two lines to display the graph using the special block character on the LCD. This line is left justified and is filled with whitespace up to sixteen characters. Finally, the lines are written to the LCD with the serial `write` methods. The cursor does not have to be reset between each line because after writing the first, it will be ready in the second line.

Lastly, the bandwidth adjust knob is modeled in the `BandwidthKnob` class. This class uses the `ADC` module and is also implemented as a thread. The analog input will always produce a value when read, and we will ratio this value to the available bandwidth for the bridge. The `Adafruit` library normalizes the values from `0.0` to `1.0`; so, when the knob is at its midpoint, the call to `ADC.read(pin)` should return `0.5`. There is some jitter in the analog signal, and we don't need a fine resolution on the bandwidth. Sampling once every second should suffice since we don't expect the knob to change frequently. We'll round up to the next whole number, which will give the knob ten discrete settings. The `run` method will report back to the caller, via a message queue, if the *volume* setting has changed. The calling thread can then update the Tor bridge bandwidth limits accordingly.

This code sample shows you how to set up the ADC on BBB with the `Adafruit` library and how to pass that result to a waiting thread with a message queue:

```
import Adafruit_BBIO.ADC as ADC
import threading
from time import sleep
from math import ceil, floor
import Queue
class BandwidthKnob(threading.Thread):

  def __init__(self, pin, *args, **kwargs):

    threading.Thread.__init__(self, *args, **kwargs)
    self.pin = pin
    self.setup_adc()
    self.kill = False
    self.prev_value = -1
    self.q = Queue.Queue()

  def setup_adc(self):
    'Load the Adafruit device tree fragment for ADC pins'
 ADC.setup()

  def read_value(self):
    return ceil(10 * ADC.read(self.pin))

  def stop(self):
    self.kill = True

  def run(self):
    knob = self.prev_value

    while knob != 0 and not self.kill:
      sleep(1)
      knob = self.read_value()
      if knob != self.prev_value:
 self.q.put(knob)
        self.prev_value = knob
```

# Determining your bandwidth with speedtest-cli

In order to adjust the bandwidth rate, we first need to know how much bandwidth our bridge has. Fortunately, there is a nice script to run a speed test from your command line that is appropriately called `speedtest-cli`. This is installed with the following command:

```
sudo pip install git+https://github.com/sivel/speedtest-cli.git

```

Run the test with the following command:

```
speedtest-cli --simple > speedtest.txt

```

If you inspect the output file, you should see something like the following:

```
Ping: 107.686 ms
Download: 28.23 Mbit/s
Upload: 5.37 Mbit/s

```

We'll use the results in this file as the basis for our bandwidth adjustment. At the moment, we only need to remember its location for later use.

# Controlling the bridge with the Stem library

The bridge is controlled using the `Stem` library, which communicates with the Tor process over the Tor control protocol. The setup is managed in the `BeagleBridge` class. After establishing a connection with the Tor process, this class registers two event listeners for the `Bandwidth` and `Configuration` changed event. The bandwidth event is triggered each second and reports, via the `print_bw` callback, the bytes used in the last second. This information is used to draw the bandwidth graph. The following callback function shows how the callback interacts with the LCD:

```
def make_bw_callback(test,lcd):
  '''Returns a callback function for the bandwidth event'''
  def print_bw(event):
    '''Obtains the bandwidth used from the last second from the
       bridge, normalizes it to the total bandwidth, and draw 
       that information to the display'''
    up = int(test.get_up_ratio(event.written))
    down = int(test.get_down_ratio(event.read))
    lcd.display_graph(up, down)

  return print_bw
```

### Note

For those who want to dive deeper into the `Stem` library and the Tor control protocol, the `Stem` library has thorough online documentation and examples ([https://stem.torproject.org/tutorials.html](https://stem.torproject.org/tutorials.html)). The control protocol, for those who want an even more in-depth look, is located at [https://gitweb.torproject.org/torspec.git?a=blob_plain;hb=HEAD;f=control-spec.txt](https://gitweb.torproject.org/torspec.git?a=blob_plain;hb=HEAD;f=control-spec.txt).

The `Configuration Changed` event is the callback to inform the process that the bridge's configuration was changed. This occurs when the bandwidth knob is adjusted, which causes the `update_rate` method to be called, that sends a command to the bridge to update the configuration. The end result is that by adjusting the knob, you directly affect the bandwidth limits on your bridge. When the callback occurs, it will display the new bandwidth rates on the LCD so that you know that your limit has changed. This callback is shown as follows:

```
def make_conf_callback(lcd):
  '''Returns a callback function for the configuration changed
     event'''
  def conf_changed(event):
    '''Reads the new bandwidth rates from the bridge and draws
       that information to the display'''
    rate = str(int(event.config['RelayBandwidthRate']) / 1024)
    burst = str(int(event.config['RelayBandwidthBurst']) / 1024)
    lcd.display_rates(rate, burst)

  return conf_changed
```

The main section of the Python script performs the class instantiation and also displays a one-time splash screen to the LCD. It will show the total bytes transmitted by the bridge, the number of established circuits, and the last 24 bytes of the bridge's fingerprint. The bridge controller will run forever, while the bandwidth knob is at a nonzero value. When you dial the knob to zero, the program will exit and the LCD will be filled with blocks.

Finally, to run the bridge controller, execute the following command, with the first parameter being the location of the speed test results:

```
sudo python beaglebridge.py ~/speedtest.txt &

```

To avoid running as root, you'll have to manipulate user groups and permissions. The Tor process runs as the `debian-tor` user, and the `Adafruit` library, which enables device tree overlays on your behalf, needs to run at a user level that has the permission to enable these features. You can create a custom group and a user that is in the `debian-tor` group and then give that group permission to modify the device tree files to not run as root.

# Connecting to your obfuscated bridge

With BeagleBridge, you have your own entry point in the Tor network. You can download and install the Tor browser and configure it to use your bridge. This is useful anytime you find yourself on a restricted or hostile network and want to access the Internet more anonymously. However, if you use *your* BeagleBridge, passive attackers could learn the IP address to which you are connecting, which happens to be your home network. The traffic is obfuscated, but it may look suspicious over time. It might be better to use a random bridge address obtained via Tor. Even if you don't directly connect to your own bridge, your bridge is helping to contribute resources to the Tor network, which helps everybody access a censor-free Internet. To connect to your bridge, launch the Tor browser and click on **Open Settings** as it starts up. Then, answer with a `Yes` to questions about whether your connection is censored. Select **Enter Custom Bridges** and enter your bridge as follows, but replace the x's with your IP address and the port number of your bridge:

`bridge obfs3 XXX.XXX.XXX.XXX:59519`

# Continuing with Tor-related projects

This project is just the first step in learning Tor. If you want to route more traffic with BBB, you can switch from a bridge to a public relay. Relays receive more traffic but are also more public. Your IP address will be listed as a public bridge, and some websites will refuse to serve you content even if you are running a nonexit relay.

You can also run a Tor **hidden service** on the BeagleBone. Using Tor with a client helps to anonymize your IP address to a server, but a Tor hidden service hides the server's IP address from a client. It would be an interesting project for a BeagleBone if you had a service that you want accessible only through Tor.

Hardware-wise, consider placing the BeagleBone and front panel in a dedicated enclosure. See whether there is a local hackerspace around you, and they may be able to help you make a nice laser-cut enclosure! If you don't have a laser cutter, you can use a SparkFun Electronics box like the one in the following screenshot:

![Continuing with Tor-related projects](img/00010.jpeg)

### Note

The onion logo is a registered trademark of the Tor Project, used with permission. The BeagleBridge project described in this chapter is not affiliated to the Tor Project.

# Summary

In this chapter, you learned about Tor and how to circumvent Internet censorship by running a Tor bridge on BBB. We've also shown how to add some basic hardware controls to BBB in order to create a front panel interface. Lastly, through some Python code, we were able to tie the hardware controls and the Tor bridge together.

In the next chapter, we'll take a closer look at specialized cryptographic hardware available for BBB and show you how to use each of these devices.