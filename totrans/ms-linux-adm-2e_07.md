# 7

# Networking with Linux

**Linux networking** is a vast domain. The last few decades have seen countless volumes and references written about Linux network administration internals. Sometimes, the mere assimilation of essential concepts can be overwhelming for both novice and advanced users. This chapter provides a relatively concise overview of Linux networking, focusing on network communication layers, sockets and ports, network services and protocols, and network security.

We hope that the content presented in this chapter is both a comfortable introduction to basic Linux networking principles for a novice user and a good refresher for an advanced Linux administrator.

In this chapter, we’ll cover the following topics:

*   Exploring basic networking – focusing on computer networks, networking models, protocols, network addresses, and ports. We’ll also cover some practical aspects of configuring Linux network settings using the command-line Terminal.
*   Working with network services – introducing common networking servers that run on Linux.
*   Understanding network security.

# Technical requirements

Throughout this chapter, we’ll be using the Linux command line to some extent. A working Linux distribution, installed on either a **virtual machine** (**VM**) or a desktop platform, is highly recommended. If you don’t have one already, go back to [*Chapter 1*](B19682_01.xhtml#_idTextAnchor030), *Installing Linux*, which will guide you through the installation process. Most of the commands and examples illustrated in this chapter use Ubuntu and Fedora, but the same would apply to any other Linux platform.

# Exploring basic networking

Today, it’s almost inconceivable to imagine a computer not connected to some sort of network or the internet. Our ever-increasing online presence, cloud computing, mobile communications, and **Internet of Things** (**IoT**) would not be possible without the highly distributed, high-speed, and scalable networks serving the underlying data traffic; yet the basic networking principles behind the driving force of the modern-day internet are decades old. Networking and communication paradigms will continue to evolve, but some of the original primitives and concepts will still have a long-lasting effect in shaping the building blocks of future communications.

This section will introduce you to a few of these networking essentials and, hopefully, spark your curiosity for further exploration. Let’s start with computer networks.

## Computer networks

A **computer network** is a group of two or more computers (or nodes) connected via a physical medium (cable, wireless, optical) and communicating with each other using a standard set of agreed-upon communication protocols. At a very high level, a **network communication infrastructure** includes computers, devices, switches, routers, Ethernet or optical cables, wireless environments, and all sorts of network equipment.

Beyond the *physical* connectivity and arrangement, networks are also defined by a *logical* layout via network topologies, tiers, and the related data flow. An example of a logical networking hierarchy is the three-tiered layering of the **demilitarized zone** (**DMZ**), *firewall*, and *internal* networks. The DMZ is an organization’s outward-facing network, with an extra security layer against the public internet. A firewall controls the network traffic between the DMZ and the internal network.

Network devices are identified by the following aspects:

*   **Network addresses**: These assist with locating nodes on the network using **communication protocols**, such as the **Internet Protocol** (**IP**) (see more on IP in the *TCP/IP protocols* section, later in this chapter)
*   **Hostnames**: These are user-friendly labels associated with devices that are easier to remember than network addresses

A common classification criterion looks at the *scale* and *expansion* of computer networks. Let’s introduce you to **local area networks** (**LANs**) and **wide area** **networks** (**WANs**):

*   **LANs**: A LAN represents a group of devices connected and located in a single physical location, such as a private residence, school, or office. A LAN can be of any size, ranging from a home network with only a few devices to large-scale enterprise networks with thousands of users and computers.

    Regardless of the network’s size, a LAN’s essential characteristic is that it connects devices in a single, limited area. Examples of LANs include the home network of a single-family residence or your local coffee shop’s free wireless service.

    For more information about LANs, you can refer to [https://www.cisco.com/c/en/us/products/switches/what-is-a-lan-local-area-network.html](https://www.cisco.com/c/en/us/products/switches/what-is-a-lan-local-area-network.html).

    When a computer network spans multiple regions or multiple interconnected LANs, WANs come into play.

*   **WANs**: A WAN is usually a network of networks, with multiple or distributed LANs communicating with each other. In this sense, we regard the internet as the world’s largest WAN. An example of a WAN is the computer network of a multinational company’s geographically distributed offices worldwide. Some WANs are built by service providers, to be leased to various businesses and institutions around the world.

    WANs have several variations, depending on their type, range, and use. Typical examples of WANs include **personal area networks** (**PANs**), **metropolitan area networks** (**MANs**), and **cloud** or **internet area** **networks** (**IANs**).

    For more information about WANs, you can refer to [https://www.cisco.com/c/en/us/products/switches/what-is-a-wan-wide-area-network.html](https://www.cisco.com/c/en/us/products/switches/what-is-a-wan-wide-area-network.html).

We think that an adequate introduction to basic networking principles should always include a brief presentation of the theoretical model governing network communications in general. We’ll look at this next.

## The OSI model

The **Open Systems Interconnection** (**OSI**) model is a theoretical representation of a multilayer communication mechanism between computer systems interacting over a network. The OSI model was introduced in 1983 by the **International Organization for Standardization** (**ISO**) to provide a standard for different computer systems to communicate with each other.

We can regard the OSI model as a universal framework for network communications. As the following figure shows, the OSI model defines a stack of seven layers, directing the communication flow:

![Figure 7.1 – The OSI model](img/B19682_07_01.jpg)

Figure 7.1 – The OSI model

In the layered view shown in the preceding figure, the communication flow moves from top to bottom (on the transmitting end) or bottom to top (on the receiving end). Before we look at each layer in more detail, let’s briefly explain how the OSI model and data encapsulation and decapsulation work.

### Data encapsulation and decapsulation in the OSI model

When using the network to transfer data from one computer to another, some specific rules are followed. Inside the OSI model, the network data flows in two different ways. One way is down, from Layer 7 to Layer 1, and this is known as **data encapsulation**. The other way, up from Layer 1 to Layer 7, is known as **data decapsulation**. Data encapsulation represents the process of sending data from one computer to another, while data decapsulation represents the process of receiving data. Let’s look at these in greater detail:

*   **Encapsulation**: When sending data, data from one computer is converted to be sent through the network, and it receives extra information as it is being sent though all the layers of the stack. The application layer (Layer 7) is the place where the user directly interacts with the application. Then, data is sent through the presentation (Layer 6) and session (Layer 5) layers, where data is transformed into a usable format. In the transport layer (Layer 4), data is broken into smaller chunks (segments) and receives a new TCP header. Inside the network layer (Layer 3), the data is called a packet, receives an IP header, and is sent to the data link layer (Layer 2), where it is called a frame and contains both TCP and IP headers. At Layer 2, each frame receives information about the hardware addresses of the source and destination (**media access control** (**MAC**) addresses) and information about the protocols to be used in the network layer (created by the **logical link control** (**LLC**) data communication protocol). At this point, a new field is added, called **Frame Check Sequence** (**FCS**), which is used for checking errors. Then, the frames are passed through the physical layer (Layer 1).
*   **Decapsulation**: When data is received, the process is identical, but in reverse order. This starts from the physical layer (Layer 1), where the first synchronization happens, after which the frame is sent through the data link layer (Layer 2), where an error check is done, by verifying the FCS field. This process is called a **Cyclic Redundancy Check** (**CRC**). The data, which is now a packet, is sent through all the other layers. Here, the headers that were added during the encapsulation process are stripped off until they reach the upper layers and become ready to be used on the target computer. A graphic explanation is provided in the following figure:

![Figure 7.2 – Encapsulation and decapsulation in the OSI model](img/B19682_07_02.jpg)

Figure 7.2 – Encapsulation and decapsulation in the OSI model

Let’s look at each of these layers in detail and describe their functionality in shaping network communication.

### The physical layer

The **physical layer** (or *Layer 1*) consists of the networking equipment or infrastructure connecting the devices and serving the communication, such as cables, wireless or optical environments, connectors, and switches. This layer handles the conversion between raw bit streams and the communication medium (which includes electrical, radio, or optical signals) while regulating the corresponding bit-rate control.

Examples of protocols operating at the physical layer include Ethernet, **Universal Serial Bus** (**USB**), and **Digital Subscriber** **Line** (**DSL**).

### The data link layer

The **data link layer** (or *Layer 2*) establishes a reliable data flow between two directly connected devices on a network, either as adjacent nodes in a WAN or as devices within a LAN. One of the data link layer’s responsibilities is flow control, adapting to the physical layer’s communication speed. On the receiving device, the data link layer corrects communication errors that originated in the physical layer. The data link layer consists of the following subsystems:

*   **Media access control** (**MAC**): This subsystem uses MAC addresses to identify and connect devices on the network. It also controls the device access permissions to transmit and receive data on the network.
*   **Logical link control** (**LLC**): This subsystem identifies and encapsulates network layer protocols and performs error checking and frame synchronization while transmitting or receiving data.

The protocol data units controlled by the data link layer are also known as **frames**. A frame is a data transmission unit that acts as a container for a single network packet. Network packets are processed at the next OSI level (*network layer*). When multiple devices access the same physical layer simultaneously, frame collisions may occur. Data link layer protocols can detect and recover from such collisions and further reduce or prevent their occurrence.

There are also Ethernet frames, for example, which are encapsulated data that is defined for MAC implementations. The original IEEE 802.3 Ethernet format, the 802.3 **SubNetwork Access Protocol** (**SNAP**), and the Ethernet II (extended) frame formats are also available.

One more example of the data link protocol is the **Point-to-Point Protocol** (**PPP**), a binary networking protocol that’s used in high-speed broadband communication networks.

### The network layer

The **network layer** (or *Layer 3*) discovers the optimal communication path (or route) between devices on a network. This layer uses a routing mechanism based on the IPaddresses of the devices involved in the data exchange to move data packets from source to destination.

On the transmitting end, the network layer disassembles the data segments that originated in the *transport layer* into network packets. On the receiving end, the data frames are reassembled from the layer below (*data link layer*) into packets.

A protocol that operates at the network layer is the **Internet Control Message Protocol** (**ICMP**). ICMP is used by network devices to diagnose network communication issues. ICMP reports an error when a requested endpoint is not available by sending messages such as *destination network unreachable*, *timer expired*, *source route failed*, and others.

### The transport layer

The **transport layer** (or *Layer 4*) operates with **data segments** or **datagrams**. This layer is mainly responsible for transferring data from a source to a destination and guaranteeing a specific **quality of service** (**QoS**). On the transmitting end, data that originated from the layer above (*session layer*) is disassembled into segments. On the receiving end, the transport layer reassembles the data packets received from the layer below (*network layer*) into segments.

The transport layer maintains the reliability of the data transfer through flow-control and error-control functions. The flow-control function adjusts the data transfer rate between endpoints with different connection speeds, to avoid a sender overwhelming the receiver. When the data received is incorrect, the error-control function may request the retransmission of data.

Examples of transport layer protocols include the **Transmission Control Protocol** (**TCP**) and the **User Datagram** **Protocol** (**UDP**).

### The session layer

The **session layer** (or *Layer 5*) controls the lifetime of the connection channels (or sessions) between devices communicating on a network. At this layer, sessions or network connections are usually defined by network addresses, sockets, and ports. We’ll explain each of these concepts in the *Sockets and ports* and *IP addresses* sections. The session layer is responsible for the integrity of the data transfer within a communication channel or session. For example, if a session is interrupted, the data transfer resumes from a previous checkpoint.

Some typical session layer protocols are the **Remote Procedure Call** (**RPC**) protocol, which is used by interprocess communications, and **Network Basic Input/Output System** (**NetBIOS**), which is a file-sharing and name-resolution protocol.

### The presentation layer

The **presentation layer** (or *Layer 6*) acts as a data translation tier between the *application layer* above and the *session layer* below. On the transmitting end, this layer formats the data into a system-independent representation before sending it across the network. On the receiving end, the presentation layer transforms the data into an application-friendly format. Examples of such transformations are encryption and decryption, compression and decompression, encoding and decoding, and serialization and deserialization.

Usually, there is no substantial distinction between the presentation and application layers, mainly due to the relatively tight coupling of the various data formats with the applications consuming them. Standard data representation formats include the **American Standard Code for Information Interchange** (**ASCII**), **Extensible Markup Language** (**XML**), **JavaScript Object Notation** (**JSON**), **Joint Photographic Experts Group** (**JPEG**), ZIP, and others.

### The application layer

The **application layer** (or *Layer 7*) is the closest to the end user in the OSI model. This layer collects or provides the input or output of application data in some meaningful way. This layer does not contain or run the applications themselves. Instead, it acts as an abstraction between applications, implementing a communication component and the underlying network. Typical examples of applications that interact with the application layer are web browsers and email clients.

A few examples of Layer 7 protocols are the DNS protocol, the **HyperText Transfer Protocol** (**HTTP**), the **File Transfer Protocol** (**FTP**), and email messaging protocols, such as the **Post Office Protocol** (**POP**), **Internet Message Access Protocol** (**IMAP**), and **Simple Mail Transfer** **Protocol** (**SMTP**).

Before wrapping up, we should note that the OSI model is a generic representation of networking communication layers and provides the theoretical guidelines for how network communication works. A similar – but more practical – illustration of the networking stack is the TCP/IP model. Both these models are useful when it comes to network design, implementation, troubleshooting, and diagnostics. The OSI model gives network operators a good understanding of the full networking stack, from the physical medium to the application layer, and each level has **Protocol Data Units** (**PDUs**) and communication internals. However, the TCP/IP model is somewhat simplified, with a few of the OSI model layers collapsed into one, and it takes a rather protocol-centric approach to network communications. We’ll explore this in more detail in the next section.

## The TCP/IP network stack model

The **TCP/IP model** is a four-layer interpretation of the OSI networking stack, where some of the equivalent OSI layers appear consolidated, as shown in the following figure:

![Figure 7.3 – The OSI and TCP/IP models](img/B19682_07_03.jpg)

Figure 7.3 – The OSI and TCP/IP models

Chronologically, the TCP/IP model is older than the OSI model. It was first suggested by the US **Department of Defense** (**DoD**) as part of an internetwork project developed by the **Defense Advanced Research Projects Agency** (**DARPA**). This project eventually became the modern-day internet.

The TCP/IP model layers encapsulate similar functions to their counterpart OSI layers. Here’s a summary of each layer in the TCP/IP model.

### The network interface layer

The **network interface layer** is responsible for data delivery over a physical medium (such as wire, wireless, or optical). Networking protocols operating at this layer include Ethernet, Token Ring, and Frame Relay. This layer maps to the composition of the *physical and data link layers* in the OSI model.

### The internet layer

The **internet layer** provides *connectionless* data delivery between nodes on a network. Connectionless protocols describe a network communication pattern where a sender transmits data to a receiver without a prior arrangement between the two. This layer is responsible for disassembling data into network packets at the transmitting end and reassembling them on the receiving end. The internet layer uses routing functions to identify the optimal path between the network nodes. This layer maps to the *network layer* in the OSI model.

### The transport layer

The **transport layer** (also known as the **transmission layer** or the **host-to-host layer**) is responsible for maintaining the communication sessions between connected network nodes. The transport layer implements error-detection and correction mechanisms for reliable data delivery between endpoints. This layer maps to the *transport layer* in the OSI model.

### The application layer

The **application layer** provides the data communication abstraction between software applications and the underlying network. This layer maps to the composition of the *session, presentation, and application layers* in the OSI model.

As discussed earlier in this chapter, the TCP/IP model is a protocol-centric representation of the networking stack. This model served as the foundation of the internet by gradually defining and developing networking protocols required for internet communications. These protocols are collectively referred to as the *IP suite*. The following section describes some of the most common networking protocols.

## TCP/IP protocols

In this section, we’ll describe some widely used networking protocols. The reference provided here should not be regarded as an all-encompassing guide. There are a vast number of TCP/IP protocols, and a comprehensive study is beyond the scope of this chapter. Nevertheless, there are a handful of protocols worth exploring, frequently at work in everyday network communication and administration workflows.

The following list briefly describes each TCP/IP protocol and its related **Request for Comments** (**RFC**) identifier. The RFC represents the detailed technical documentation – of a protocol, in our case – that’s usually authored by the **Internet Engineering Task Force** (**IETF**). For more information about RFC, please refer to [https://www.ietf.org/standards/rfcs/](https://www.ietf.org/standards/rfcs/). Here are the most widely used protocols:

*   **IP**: IP (*RFC 791*) identifies network nodes based on fixed-length addresses, also known as IP addresses. IP addresses will be described in more detail in the next section. The IP protocol uses datagrams as the data transmission unit and provides fragmentation and reassembly capabilities of large datagrams to accommodate small-packet networks (and avoid transmission delays). The IP protocol also provides routing functions to find the optimal data path between network nodes. IP operates at the network layer (*Layer 3*) in the OSI model.
*   **ARP**: The **Address Resolution Protocol** (**ARP**) (*RFC 826*) is used by the IP protocol to map IP network addresses (specifically, **IP version 4** or **IPv4**) to device MAC addresses used by a data link protocol. ARP operates at the data link layer (*Layer 2*) in the OSI model.
*   **NDP**: The **Neighbor Discovery Protocol** (**NDP**) (*RFC 4861*) is like the ARP protocol, and it also controls **IP version 6** (**IPv6**) address mapping. NDP operates within the data link layer (*Layer 2*) in the OSI model.
*   **ICMP**: ICMP (*RFC 792*) is a supporting protocol for checking transmission issues. When a device or node is not reachable within a given timeout, ICMP reports an error. ICMP operates at the network layer (*Layer 3*) in the OSI model.
*   **TCP**: TCP (*RFC 793*) is a connection-oriented, highly reliable communication protocol. TCP requires a logical connection (such as a *handshake*) between the nodes before initiating the data exchange. TCP operates at the transport layer (*Layer 4*) in the OSI model.
*   **UDP**: UDP (*RFC 768*) is a connectionless communication protocol. UDP has no handshake mechanism (compared to TCP). Consequently, with UDP, there’s no guarantee of data delivery. It is also known as a *best-effort protocol*. UDP uses datagrams as the data transmission unit, and it’s suitable for network communications where error checking is not critical. UDP operates at the transport layer (*Layer 4*) in the OSI model.
*   **Dynamic Host Configuration Protocol** (**DHCP**): The DHCP (*RFC 2131*) provides a framework for requesting and passing host configuration information required by devices on a TCP/IP network. DHCP enables the automatic (dynamic) allocation of reusable IP addresses and other configuration options. DHCP is considered an application layer (*Layer 7*) protocol in the OSI model, but the initial DHCP discovery mechanism operates at the data link layer (*Layer 2*).
*   `dns.google.com`) into an IP address (such as `8.8.8.8`). The DNS protocol operates at the application layer (*Layer 7*) in the OSI model.
*   **HTTP**: HTTP (*RFC 2616*) is the vehicular language of the internet. HTTP is a stateless application-level protocol based on the request and response between a client application (for example, a browser) and a server endpoint (for example, a web server). HTTP supports a wide variety of data formats, ranging from text to images and video streams. HTTP operates at the application layer (*Layer 7*) in the OSI model.
*   **FTP**: FTP (*RFC 959*) is a standard protocol for transferring files requested by an FTP client from an FTP server. FTP operates at the application layer (*Layer 7*) in the OSI model.
*   **TELNET**: The **Terminal Network protocol** (**TELNET**) (*RFC 854*) is an application-layer protocol that provides a bidirectional text-oriented network communication between a client and a server machine, using a virtual terminal connection. TELNET operates at the application layer (*Layer 7*) in the OSI model.
*   **SSH**: **Secure Shell** (**SSH**) (*RFC 4253*) is a secure application-layer protocol that encapsulates strong encryption and cryptographic host authentication. SSH uses a virtual terminal connection between a client and a server machine. SSH operates at the application layer (*Layer 7*) in the OSI model.
*   **SMTP**: SMTP (*RFC 5321*) is an application-layer protocol for sending and receiving emails between an email client (for example, Outlook) and an email server (such as Exchange Server). SMTP supports strong encryption and host authentication. SMTP acts at the application layer (*Layer 7*) in the OSI model.
*   **SNMP**: The **Simple Network Management Protocol** (**SNMP**) (*RFC 1157*) is used for remote device management and monitoring. SNMP operates at the application layer (*Layer 7*) in the OSI model.
*   **NTP**: The **Network Time Protocol** (**NTP**) (*RFC 5905)* is an internet protocol that’s used for synchronizing the system clock of multiple machines across a network. NTP operates at the application layer (*Layer 7*) in the OSI model.

Most of the internet protocols enumerated previously use the IP protocol to identify devices participating in the communication. Devices on a network are uniquely identified by an IP address. Let’s examine these network addresses more closely.

## IP addresses

An **IP address** is a fixed-length **unique identifier** (**UID**) of a device in a network. Devices locate and communicate with each other based on IP addresses. The concept of an IP address is very similar to a postal address of a residence, whereby mail or a package would be sent to that destination based on its address.

Initially, IP defined the IP address as a 32-bit number known as an **IPv4 address**. With the growth of the internet, the total number of IP addresses in a network has been exhausted. To address this issue, a new version of the IP protocol devised a 128-bit numbering scheme for IP addresses. A 128-bit IP address is also known as an **IPv6 address**.

In the next few sections, we’ll take a closer look at the networking constructs that play an important role in IP addresses, such as IPv4 and IPv6 address formats, network classes, subnetworks, and broadcast addresses.

### IPv4 addresses

An `.`). Each number in these four groups is an integer between `0` and `255`. An example of an IPv4 address is `192.168.1.53`.

The following figure shows a binary representation of an IPv4 address:

![Figure 7.4 – Network classes](img/B19682_07_04.jpg)

Figure 7.4 – Network classes

The IPv4 address space is limited to 4,294,967,296 (2^32) addresses (roughly 4 billion). Of these, approximately 18 million are reserved for special purposes (for example, private networks), and about 270 million are multicast addresses.

A **multicast address** is a logical identifier of a group of IP addresses. For more information on multicast addresses, please refer to *RFC* *6308* ([https://tools.ietf.org/html/rfc6308](https://tools.ietf.org/html/rfc6308)).

### Network classes

In the early stages of the internet, the highest-order byte (first group) in the IPv4 address indicated the **network number**. The subsequent bytes further express the network hierarchy and subnetworks, with the lowest-order byte identifying the device itself. This scheme soon proved insufficient for network hierarchies and segregations as it only allowed for 256 (2^8) networks, denoted by the leading byte of the IPv4 address. As additional networks were added, each with its own identity, the IP address specification needed a special revision to accommodate a standard model. The *Classful Network* specification, which was introduced in 1981, addressed this problem by dividing the IPv4 address space into five classes based on the leading 4 bits of the address, as illustrated in the following figure:

![Figure 7.5 – Network classes](img/B19682_07_05.jpg)

Figure 7.5 – Network classes

For more information on network classes, please refer to *RFC 870* ([https://tools.ietf.org/html/rfc870](https://tools.ietf.org/html/rfc870)). In the preceding figure, the last column specifies the default subnet mask for each of these network classes. We’ll look at subnets (or subnetworks) next.

### Subnetworks

**Subnetworks** (or **subnets**) are logical subdivisions of an IP network. Subnets were introduced to identify devices that belong to the same network. The IP addresses of devices in the same network have an identical most-significant group. The subnet definition yields a logical division of an IP address into two fields: the **network identifier** and the **host identifier**. The numerical representation of the subnet is called a **subnet mask** or **netmask**. The following figure provides an example of a network identifier and a host identifier:

![Figure 7.6 – Subnet with network and host identifiers](img/B19682_07_06.jpg)

Figure 7.6 – Subnet with network and host identifiers

With our IPv4 address (`192.168.1.53`), we could devise a network identifier of `192.168.1`, where the host identifier is `53`. The resulting subnet mask would be as follows:

```
192.168.1.0
```

We dropped the least significant group in the subnet mask, representing the host identifier (`53`), and replaced it with `0`. Here, `0` indicates the starting address in the subnet. In other words, any host identifier value in the range of `0` to `255` is allowed in the subnetwork. For example, `192.168.1.92` is a valid (and accepted) IP address in the `192.168.1.0` network.

An alternative representation of subnets uses so-called `/`) and the *bit-length* of the prefix. In our case, the CIDR notation of the `192.168.1.0` subnet is this:

```
192.168.1.0/24
```

The first three groups in the network address make up *3 x 8 = 24* bits, hence the `/``24` notation.

Usually, `100` and end with `125`. Let’s look at how we can achieve this.

First, let’s see the binary representation of `192.168.1.100`:

```
11000000.10101000.00000001.100). Remember that we want the network to start with 100 and end with 125\. This means that the closest binary value to the reserved 99 addresses that would not be permitted in our subnet is *96 = 64 + 32*. The equivalent binary value for it is as follows:

```
1. These bits would be added to the 24 already reserved bits of the network address (192.168.1), accounting in total for *27 = 24 + 3* bits. Here’s the equivalent representation:

```
11111111.11111111.11111111.11100000
```

 Consequently, the resulting netmask is as follows:

```
255.255.255.224
```

 The CIDR notation of the corresponding subnet is shown here:

```
192.168.1.96/27
```

 The remaining five bits in the host identifier’s group account for *2^5 = 32* possible addresses in the subnet, starting with `97`. This would limit the maximum host identifier value to *127 = 96 + 32 – 1* (we subtract 1 to account for the starting number of 97 included in the total of 32). In this range of 32 addresses, the last IP address is reserved as a **broadcast address**, as shown here:

```
192.168.1.127
```

 A broadcast address is reserved as the highest number in a network or subnet, when applicable. Back to our example, excluding the broadcast address, the maximum host IP address in the subnet is as follows:

```
192.168.1.126
```

 You can learn more about subnets in *RFC 1918* ([https://tools.ietf.org/html/rfc1918](https://tools.ietf.org/html/rfc1918)). Since we mentioned the broadcast address, let’s have a quick look at it.
Broadcast addresses
A **broadcast address** is a reserved IP address in a network or subnetwork that’s used to transmit a collective message (data) to all devices belonging to the network. The broadcast address is the last IP address in the network or subnet, when applicable.
For example, the broadcast address of the `192.168.1.0/24` network is `192.168.1.255`. In our example in the previous section, the broadcast address of the `192.168.1.96/27` subnet is `192.168.1.127` (*127 = 96 + 32 –* *1*).
For more information on broadcast addresses, you can refer to [https://www.sciencedirect.com/topics/computer-science/broadcast-address](https://www.sciencedirect.com/topics/computer-science/broadcast-address).
IPv6 addresses
An IPv6 address is a 128-bit number (16 bytes) that’s usually expressed as up to eight groups of 2-byte (16 bits) numbers, separated by a column (`:`). Each number in these eight groups is a hexadecimal number, with values between `0000` and `FFFF`. Here’s an example of an IPv6 address:

```
2001:0b8d:8a52:0000:0000:8b2d:0240:7235
```

 An equivalent representation of the preceding IPv6 address is shown here:

```
2001:b8d:8a52::8b2d:240:7235/64
```

 In the second representation, the leading zeros are omitted, and the all-zero groups (`0000:0000`) are collapsed into an empty group (`::`). The `/64` notation at the end represents the `1` and `128`.
In our case, with a prefix length of 64 (*4 x 16*) bits, the subnet looks like this:

```
2001:b8d:8a52::
```

 The subnet represents the leading four groups (`2001`, `0b8d`, `8a52`, and `0000`), which results in a total of *4 x 16 = 64* bits. In the shortened representation of the IPv6 subnet, the leading zeros are omitted and the all-zero group is collapsed to `::`.
Subnetting with IPv6 is very similar to IPv4\. We won’t go into the details here since the related concepts were presented in the *IPv4 addresses* section. For more information about IPv6, please refer to *RFC* *2460* ([https://tools.ietf.org/html/rfc2460](https://tools.ietf.org/html/rfc2460)).
Now that you’ve become familiar with IP addresses, it is fitting to introduce some of the related network constructs that serve the software implementation of IP addresses – that is, sockets and ports.
Sockets and ports
A **socket** is a software data structure representing a network node for communication purposes. Although a programming concept, in Linux, a **network socket** is ultimately a file descriptor that’s controlled via a network **application programming interface** (**API**). A socket is used by an application process for transmitting and receiving data. An application can create and delete sockets. A socket cannot be active (send or receive data) beyond the lifetime of the process that created the socket.
Network sockets operate at the *transport-layer* level in the OSI model. There are two endpoints to a socket connection – a sender and a receiver. Both the sender and receiver have an IP address. Consequently, a critical piece of information in the socket data structure is the *IP address* of the endpoint that owns the socket.
Both endpoints create and manage their sockets via the network processes using these sockets. The sender and receiver may agree upon using multiple connections to exchange data. Some of these connections may even run in parallel. How do we differentiate between these socket connections? The IP address by itself is not sufficient, and this is where **ports** come into play.
A `0` and `65535`. Usually, ports in the range of `0` and `1024` are assigned to the most used services on a system. These ports are also called **well-known ports**. Here are a few examples of well-known ports and the related network service for each:

*   `25`: SMTP
*   `21`: FTP
*   `22`: SSH
*   `53`: DNS
*   `67`, `68`: DHCP (client = `68`, server = `67`)
*   `80`: HTTP
*   `443`: **HTTP** **Secure** (**HTTPS**)

Port numbers beyond `1024` are for general use and are also known as **ephemeral ports**.
A port is always associated with an IP address. Ultimately, a socket is a combination of an IP address and a port. For more information on network sockets, you can refer to *RFC 147* ([https://tools.ietf.org/html/rfc147](https://tools.ietf.org/html/rfc147)). For well-known ports, see *RFC* *1340* ([https://tools.ietf.org/html/rfc1340](https://tools.ietf.org/html/rfc1340)).
Now, let’s put the knowledge we’ve gained so far to work by looking at how to configure the local networking stack in Linux.
Linux network configuration
This section describes the TCP/IP **network configuration** for Ubuntu and Fedora platforms, using their latest released versions to date. The same concepts would apply to most Linux distributions, albeit some of the network configuration utilities and files involved could be different.
We can use the `ip` command-line utility to retrieve the system’s current IP addresses, as follows:

```
ip addr show
```

 An example of the output is shown here:
![Figure 7.7 – Retrieving the current IP addresses with the ip command](img/B19682_07_07.jpg)

Figure 7.7 – Retrieving the current IP addresses with the ip command
We’ve highlighted some relevant information here, such as the network interface ID (`2: enp1s0`) and the IP address with the subnet prefix (`192.168.122.117/24`).
We’ll look at Ubuntu’s network configuration next. At the time of writing this book, the released version of Ubuntu is 22.04.2 LTS.
Ubuntu network configuration
Ubuntu 22.04 provides the `netplan` command-line utility for easy network configuration. `netplan` uses a `netplan` configuration file(s) is in the `/etc/netplan/` directory, and we can access it by using the following command:

```
ls /etc/netplan/
```

 In our case, the configuration file is `00-installer-config.yaml`.
Changing the network configuration involves editing the `netplan` YAML configuration file. As good practice, we should always make a backup of the current configuration file before making changes. Changing a network configuration would most commonly involve setting up either a dynamic or a static IP address. We will show you how to configure both types in the next few sections. We’ll look at dynamic IP addressing first.
Dynamic IP configuration
To enable a dynamic (DHCP) IP address, we must edit the `netplan` configuration file and set the `dhcp4` attribute to `true` (as shown in *Figure 7**.8*) for the network interface of our choice (`ens33`, in our case). Open the `00-installer-config.yaml` file with your text editor of choice (nano, in our case):

```
sudo nano /etc/netplan/00-installer-config.yaml
```

 Here’s the related configuration excerpt, with the relevant points highlighted:
![Figure 7.8 – Enabling DHCP in the netplan configuration](img/B19682_07_08.jpg)

Figure 7.8 – Enabling DHCP in the netplan configuration
After saving the configuration file, we can test the related changes with the following command:

```
sudo netplan try
```

 We’ll get the following response:
![Figure 7.9 – Testing and accepting the netplan configuration changes](img/B19682_07_09.jpg)

Figure 7.9 – Testing and accepting the netplan configuration changes
The `netplan` keyword validates the new configuration and prompts the user to accept the changes. The following command applies the current changes to the system:

```
sudo netplan apply
```

 Next, we will configure a static IP address using `netplan`.
Static IP configuration
To set the static IP address of a network interface, we start by editing the `netplan` configuration YAML file, as follows:

```
sudo nano /etc/netplan/00-installer-config.yaml
```

 Here’s a configuration example with a static IP address of `192.168.122.22/24`:
![Figure 7.10 – Static IP configuration example with netplan](img/B19682_07_10.jpg)

Figure 7.10 – Static IP configuration example with netplan
After saving the configuration, we can test and accept it, and then apply changes, as we did in the *Dynamic IP* section, with the following commands:

```
sudo netplan try
sudo netplan apply
```

 For more information on the `netplan` command-line utility, see `netplan --help` or the related system manual (`man netplan`).
We’ll look at the Fedora network configuration next. At the time of writing, the current released version of Fedora is 37.
Fedora/RHEL network configuration
Starting with Fedora 33 and RHEL 9, network configuration files are *no longer* kept in the `/etc/sysconfig/network-scripts/` directory. To learn more about the new configuration options, read the following file:

```
cat /etc/sysconfig/network-scripts/readme-ifcfg-rh.txt
```

 The preferred method to configure the network in Fedora/RHEL is to use the `nmcli` utility. This location is deprecated and no longer used by NetworkManager in Fedora; it can still be used, but we do not recommend it. The new NetworkManager keyfiles are stored inside the `/``etc/NetworkManager/system-connections/` directory.
Let’s use some basic `nmcli` commands to view information about our connections. To learn about `nmcli`, read the related manual pages. First, let’s find information about our active connection using the following command:

```
nmcli connection show
```

 The output will show basic information about the name of the connection, UUID, type, and the device used. The following screenshot shows the relevant information on our Fedora 37 VM:
![Figure 7.11 – Using nmcli to view connection information](img/B19682_07_11.jpg)

Figure 7.11 – Using nmcli to view connection information
Similar to Ubuntu, when using Fedora/RHEL, changing a network configuration would most commonly involve setting up either a dynamic or a static IP address. We will show you how to configure both types in the following sections. Let’s look at dynamic IP addressing first.
Dynamic IP configuration
To configure a dynamic IP address using `ncmli`, we can run the following command:

```
sudo nmcli connection modify 'Wired connection 1' ipv4.method auto
```

 The `ipv4.method auto` directive enables DHCP. There is no output from the command; after execution, you will be returned to the prompt again. You can check if the command worked by viewing the `/etc/NetworkManager/system-connections/` directory. In our case, there is a new keyfile inside. It has the same name as our connection. The following is an excerpt:
![Figure 7.12 – New configuration keyfile](img/B19682_07_12.jpg)

Figure 7.12 – New configuration keyfile
Next, we’ll configure a static IP address.
Static IP configuration
To perform the equivalent static IP address changes using `ncmli`, we need to run multiple commands. First, we must set the static IP address, as follows:

```
sudo nmcli connection modify 'Wired connection 1' ipv4.address 192.168.122.3/24
```

 If no previous static IP address has been configured, we recommend saving the preceding change before proceeding with the next steps. The changes can be saved with the following code:

```
sudo nmcli connection down 'Wired connection 1'
sudo nmcli connection up 'Wired connection 1'
```

 Next, we must set the gateway and DNS IP addresses, as follows:

```
sudo nmcli connection modify 'Wired connection 1' ipv4.gateway 192.168.122.1
sudo nmcli connection modify 'Wired connection 1' ipv4.dns 8.8.8.8
```

 Finally, we must disable DHCP with the following code:

```
sudo nmcli connection modify 'Wired connection 1' ipv4.method manual
```

 After making these changes, we need to restart the `'Wired connection 1'` network interface with the following code:

```
sudo nmcli connection down 'Wired connection 1'
sudo nmcli connection up 'Wired connection 1'
```

 Now, let’s see the results of all the commands we’ve performed. After bringing the connection up again, let’s check the new IP address and the contents of the network keyfile. The following figure shows the new IP we assigned to the system (`192.168.122.3`) by using the `ip addr` `show` command:
![Figure 7.13 – Checking the new IP address](img/B19682_07_13.jpg)

Figure 7.13 – Checking the new IP address
Now, let’s view the contents of the network keyfile to see the changes that were made for static IP configuration. Remember that the location of the file is `/etc/NetworkManager/system-connections/`:
![Figure 7.14 – New keyfile configuration for the static IP address](img/B19682_07_14.jpg)

Figure 7.14 – New keyfile configuration for the static IP address
The `nmcli` utility is a powerful and useful one. At the end of this chapter, we will provide you with some useful links for learning more about it. Next, we’ll take a look at how to configure network services on openSUSE.
openSUSE network configuration
openSUSE provides several tools for network configuration: **Wicked** and NetworkManager. According to the official SUSE documentation, Wicked is used for all types of machines, from servers to laptops and workstations, whereas NetworkManager is used only for laptop and workstation setup and is not used for server setup. However, in openSUSE Leap, Wicked is set up by default on both desktop or server configurations, and NetworkManager is set up by default on laptop configurations.
For example, on our main workstation (which is a laptop), if we want to see which service is running by default on openSUSE Leap, we can use the following command:

```
sudo systemctl status network
```

 The output will show us which service is running, and in our case, it is NetworkManager:
![Figure 7.15 – Checking which network service is running in openSUSE](img/B19682_07_15.jpg)

Figure 7.15 – Checking which network service is running in openSUSE
When running the same command inside an openSUSE Leap server VM, the result is different. The output shows that Wicked is running by default. The following screenshot shows an example:
![Figure 7.16 – Wicked is running inside the openSUSE server](img/B19682_07_16.jpg)

Figure 7.16 – Wicked is running inside the openSUSE server
As a result, we will perform all the examples in this section on an openSUSE Leap server VM, thus using Wicked as the default network configuration tool. In the next section, we will configure dynamic IP on an openSUSE machine.
Dynamic IP configuration
Before setting anything up, let’s check for active connections and devices. We can do this by using the following command:

```
wicked show all
```

 The output will show all the active devices. The following screenshot shows an excerpt from the output on our machine:
![Figure 7.17 – Information about active devices](img/B19682_07_17.jpg)

Figure 7.17 – Information about active devices
There are two active connections, one is loopback (`lo`) and the other is on the ethernet port (`eth0`). We will only show information related to `eth0`. The location where Wicked stores configuration files in openSUSE is `/etc/sysconfig/network`. If we do a listing of that directory, we will see that it is already populated with configuration files for existing connections:
![Figure 7.18 – Location of the Wicked configuration files](img/B19682_07_18.jpg)

Figure 7.18 – Location of the Wicked configuration files
In our case, and maybe it will be the same for you, the file we are interested in is called `ifcfg-eth0`; we can open it with a text editor or concatenate it. Let’s take a look at the contents of the file:
![Figure 7.19 – Information provided by the configuration file](img/B19682_07_19.jpg)

Figure 7.19 – Information provided by the configuration file
As shown in the preceding screenshot, the information provided is rather scarce but relevant. For a more detailed output, we can use the following command:

```
sudo wicked show-config
```

 This will provide much more relevant information directly to the monitor. The following screenshot provides an excerpt from our output, with detailed IPv4 DHCP information:
![Figure 7.20 – Detailed information provided by Wicked](img/B19682_07_20.jpg)

Figure 7.20 – Detailed information provided by Wicked
To better understand this output, we recommend reading the `config` file and, if available, the `dhcp` files inside the `/etc/sysconfig/network` directory. These files provide information about specific variables and default parameters needed for configuring the network devices.
Dynamic IP addresses are usually set up by default when installing the operating system. If yours is not configured, all you need to do is create a configuration file inside `/etc/sysconfig/network` and give it a relevant name based on the device you are using to connect to the network, such as `ifcfg-eth0`. Inside that file, you will have to provide just three lines, as seen in *Figure 7**.19*. The following commands are needed for the actions described in this paragraph:

*   Check your device name using the `ip` `addr` command:

    ```
    /etc/sysconfig/network directory:

    ```
    sudo nano /etc/sysconfig/network/ifcfg-eth0
    ```

    ```

     *   Provide relevant information for the DHCP configuration:

    ```
    BOOTPROTO='dhcp'
    STARTMODE='auto'
    ZONE=public
    ```

     *   Restart the Wicked service:

    ```
    ping command. Here’s an example:

    ```
    ping google.com
    ```

    ```

In the following section, we will show you how to set up a static IP configuration.
Static IP configuration
To set up a static IP configuration, you would need to manually provide variables for the configuration files. These files are the same ones that were presented in the previous section. The location of the files is `/etc/sysconfig/network`. For example, you can create a new file for the `eth0` device connection and provide the information you want. Let’s look at an example of using our openSUSE Leap server VM. However, before doing this, we advise you to open the manual pages for the `ifcfg` utility as they provide valuable information on the variables used for this exercise.
Therefore, we will set the `ifcfg` configuration file as follows:

*   First, we will check for the IP address and the network device name; in our case, the dynamically allocated IP is `192.168.122.146` and the device’s name is `eth0`:

![Figure 7.21 – IP and device information](img/B19682_07_21.jpg)

Figure 7.21 – IP and device information

*   Go to the `/etc/sysconfig/network` directory and edit the `ifcfg-eth0` file; we will use the following variables for static IP configuration:
    *   `BOOTPROTO='static'`: This allows us to use a fixed IP address that will be provided by using the IPADDR variable
    *   `STARTMODE='auto'`: The interface will be automatically enabled on boot
    *   `IPADDR='192.168.122.144'`: The IP address we choose for the machine
    *   `ZONE='public'`: The zone used by the `firewalld` utility
    *   `PREFIXLEN='24'`: The number of bits in the `IPADDR` variable

    This will look as follows:

![Figure 7.22 – New variables for static IP configuration](img/B19682_07_22.jpg)

Figure 7.22 – New variables for static IP configuration

*   Save the changes to the new file and restart the Wicked daemon:

    ```
    sudo systemctl restart wickedd.service
    ```

     *   Enable the interface so that the changes appear:

    ```
    ip addr show command again to check for the new IP address:
    ```

![Figure 7.23 – New IP address assigned to eth0](img/B19682_07_23.jpg)

Figure 7.23 – New IP address assigned to eth0
At this point, you know how to configure networking devices in all major Linux distributions, using their preferred utilities. We’ve merely scratched the surface of this matter, but we’ve provided sufficient information for you to start working with network interfaces in Linux. For more information, feel free to read the manual pages freely provided with your operating system. In the next section, we will approach the matter of hostname configuration.
Hostname configuration
To retrieve the current hostname on a Linux machine, we can use either the `hostname` or `hostnamectl` command, as follows:

```
hostname
```

 The most convenient way to change the hostname is with the `hostnamectl` command. We can change the hostname to `earth` using the `set-hostname` parameter of the command:

```
sudo hostnamectl set-hostname earth
```

 Let’s verify the hostname change with the `hostname` command again. You could use the `hostnamectl` command to verify the hostname. The output of the `hostnamectl` command provides more detailed information compared to the `hostname` command, as shown in the following screenshot:
![Figure 7.24 – Retrieving the current hostname with the different commands](img/B19682_07_24.jpg)

Figure 7.24 – Retrieving the current hostname with the different commands
Alternatively, we can use the `hostname` command to change the hostname *temporarily*, as follows:

```
sudo hostname jupiter
```

 However, this change will not survive a reboot unless we also change the hostname in the `/etc/hostname` and `/etc/hosts` files. When editing these two files, change your hostname accordingly. The following screenshot shows the succession of commands:
![Figure 7.25 – The /etc/hostname and /etc/hosts files](img/B19682_07_25.jpg)

Figure 7.25 – The /etc/hostname and /etc/hosts files
After the hostname reconfiguration, a logout followed by a login would usually reflect the changes. Hostnames are important for coherent network management, where each system on the network should have relevant hostnames set up. In the following section, you’ll learn about network services in Linux.
Working with network services
In this section, we’ll enumerate some of the most common network services running on Linux. Not all the services mentioned here are installed or enabled by default on your Linux platform of choice. [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing* *Linux*, and [*Chapter 10*](B19682_10.xhtml#_idTextAnchor212), *Disaster Recovery, Diagnostics, and Troubleshooting*, will dive deeper into how to install and configure some of them. Our focus in this section remains on what these network services are, how they work, and the networking protocols they use for communication.
A **network service** is typically a system process that implements application layer (OSI *Layer 7*) functionality for data communication purposes. Network services are usually designed as peer-to-peer or client-server architectures.
In peer-to-peer networking, multiple network nodes each run their own equally privileged instance of a network service while sharing and exchanging a common set of data. Take, for example, a network of DNS servers, all sharing and updating their domain name records.
Client-server networking usually involves one or more server nodes on a network and multiple clients communicating with any of these servers. An example of a client-server network service is SSH. An SSH client connects to a remote SSH server via a secure Terminal session, perhaps for remote administration purposes.
Each of the following subsections briefly describes a network service, and we encourage you to explore topics related to these network services in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276) or other relevant titles recommended at the end of this chapter. Let’s start with DHCP servers.
DHCP servers
A **DHCP server** uses the DHCP protocol to enable devices on a network to request an IP address that’s been assigned dynamically. The DHCP protocol was briefly described in the *TCP/IP protocols* section earlier in this chapter.
A computer or device requesting a DHCP service sends out a broadcast message (or query) on the network to locate a DHCP server, which, in turn, provides the requested IP address and other information. Communication between the DHCP client (device) and the server uses the DHCP protocol.
The DHCP protocol’s initial *discovery* workflow between a client and a server operates at the data link layer (*Layer 2*) in the OSI model. Since Layer 2 uses network frames as PDUs, the DHCP discovery packets cannot transcend the local network boundary. In other words, a DHCP client can only initiate communication with a *local* DHCP server.
After the initial *handshake* (on Layer 2), DHCP turns to UDP as its transport protocol, using datagram sockets (*Layer 4*). Since UDP is a connectionless protocol, a DHCP client and server exchange messages without a prior arrangement. Consequently, both endpoints (client and server) require a well-known DHCP communication port for the back-and-forth data exchange. These are the well-known ports `68` (for a DHCP server) and `67` (for a DHCP client).
A DHCP server maintains a collection of IP addresses and other client configuration data (such as MAC addresses and domain server addresses) for each device on the network requesting a DHCP service.
DHCP servers use a **leasing mechanism** to assign IP addresses dynamically. Leasing an IP address is subject to a **lease time**, either finite or infinite. When the lease of an IP address expires, the DHCP server may reassign it to a different client upon request. A device would hold on to its dynamic IP address by regularly requesting a **lease renewal** from the DHCP server. Failing to do so would result in the potential loss of the device’s dynamic IP address. A late (or post-lease) DHCP request would possibly result in a new IP address being acquired if the previous address had already been allocated by the DHCP server.
A simple way to query the DHCP server from a Linux machine is by invoking the following command:

```
ip route
```

 This is the output of the preceding command:
![Figure 7.26 – Querying the IP route for DHCP information](img/B19682_07_26.jpg)

Figure 7.26 – Querying the IP route for DHCP information
The first line of the output provides the DHCP server (`192.168.122.1`).
[*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*, will further go into the practical details of installing and configuring a DHCP server.
For more information on DHCP, please refer to *RFC* *2131* ([https://tools.ietf.org/html/rfc2131](https://tools.ietf.org/html/rfc2131)).
DNS servers
A `wikipedia.org`) into an IP address (such as `208.80.154.224`). The name-resolution protocol is DNS, briefly described in the *TCP/IP protocols* section earlier in this chapter. In a DNS-managed TCP/IP network, computers and devices can also identify and communicate with each other via hostnames, not just IP addresses.
As a reasonable analogy, DNS very much resembles an address book. Hostnames are relatively easier to remember than IP addresses. Even in a local network, with only a few computers and devices connected, it would be rather difficult to identify (or memorize) any of the hosts by simply using their IP address. The internet relies on a globally distributed network of DNS servers.
There are four different types of DNS servers: **recursive servers**, **root servers**, **top-level domain** (**TLD**) **servers**, and **authoritative servers**. All these DNS server types work together to bring you the internet as you experience it in your browser.
A **recursive DNS server** is a resolver that helps you find the destination (IP) of a website you search for. When you perform a lookup operation, a recursive DNS server is connected to different DNS servers to find the IP address that you are looking for and return it to you in the form of a website. Recursive DNS lookups are faster as they cache every query that they perform. In a recursive type of query, the DNS server calls itself and does the recursion while still sending the request to another DNS server to find the answer.
In contrast, an **iterative DNS** lookup is done by every DNS server directly, without using caching. For example, in an iterative query, each DNS server responds with the address of another DNS server, until one of them has the matching IP address for the hostname in question and responds to the client. For more details on DNS server types, please check out the following Cloudflare learning solution: [https://www.cloudflare.com/learning/dns/what-is-dns/](https://www.cloudflare.com/learning/dns/what-is-dns/).
DNS servers maintain (and possibly share) a collection of `/etc/resolv.conf`.
To query the DNS server managing the local machine, we can query the `/etc/resolv.conf` file by running the following code:

```
cat /etc/resolv.conf | grep nameserver
```

 The preceding code yields the following output:
![Figure 7.27 – Querying DNS server using /etc/resolv.conf](img/B19682_07_27.jpg)

Figure 7.27 – Querying DNS server using /etc/resolv.conf
A simple way to query name-server data for an arbitrary host on a network is by using the `nslookup` tool. If you don’t have the `nslookup` utility installed on your system, you may do so with the commands outlined here.
On Ubuntu/Debian, run the following command:

```
sudo apt install dnsutils
```

 On Fedora, run this command:

```
sudo dnf install bind-utils
```

 For example, to query the name-server information for a computer named `neptune.local` in our local network, we can run the following command:

```
nslookup neptune.local
```

 The output is shown here:
![Figure 7.28 – Querying name-server information with nslookup](img/B19682_07_28.jpg)

Figure 7.28 – Querying name-server information with nslookup
We can also use the `nslookup` tool interactively. For example, to query the name-server information for `wikipedia.org`, we can simply run the following command:

```
nslookup
```

 Then, in the interactive prompt, we must enter `wikipedia.org`, as illustrated here:
![Figure 7.29 – Using the nslookup tool interactively](img/B19682_07_29.jpg)

Figure 7.29 – Using the nslookup tool interactively
To exit the interactive shell mode, press *Ctrl* + *C*. Here’s a brief explanation of the information shown in the preceding output:

*   `127.0.0.53`) and port (`53`) of the DNS server running locally
*   `wikipedia.org`)
*   `91.198.174.192`) and IPv6 (`2620:0:862:ed1a::1`) addresses that correspond to the lookup domain (`wikipedia.org`)

`nslookup` is also capable of reverse DNS search when providing an IP address. The following command retrieves the name server (`dns.google`) corresponding to the IP address `8.8.8.8`:

```
nslookup 8.8.8.8
```

 The preceding command yields the following output:
![Figure 7.30 – Reverse DNS search with nslookup](img/B19682_07_30.jpg)

Figure 7.30 – Reverse DNS search with nslookup
For more information on the `nslookup` tool, you can refer to the `nslookup` system reference manual (`man nslookup`).
Alternatively, we can use the `dig` command-line utility. If you don’t have the `dig` utility installed on your system, you can do so by installing the `dnsutils` package on Ubuntu/Debian or `bind-utils` on Fedora platforms. The related commands for installing the packages were shown previously with `nslookup`.
For example, the following command retrieves the name-server information for the `google.com` domain:

```
dig google.com
```

 This is the result (see the highlighted `ANSWER SECTION`):
![Figure 7.31 – DNS lookup with dig](img/B19682_07_31.jpg)

Figure 7.31 – DNS lookup with dig
To perform a reverse DNS lookup with `dig`, we must specify the `-x` option, followed by an IP address (for example, `8.8.4.4`), as follows:

```
dig -x 8.8.4.4
```

 This command yields the following output (see the highlighted `ANSWER SECTION`):
![Figure 7.32 – Reverse DNS lookup with dig](img/B19682_07_32.jpg)

Figure 7.32 – Reverse DNS lookup with dig
For more information about the `dig` command-line utility, please refer to the related system manual (`man dig`).
The DNS protocol operates at the application layer (*Layer 7*) in the OSI model. The standard DNS service’s well-known port is `53`.
[*Chapter 8*](B19682_08.xhtml#_idTextAnchor164), *Linux* *Shell Scripting*, will cover the practical details of installing and configuring a DNS server in more detail. For more information on DNS, you can refer to *RFC* *1035* ([https://www.ietf.org/rfc/rfc1035.txt](https://www.ietf.org/rfc/rfc1035.txt)).
The DHCP and DNS network services are arguably the closest to the TCP/IP networking stack while playing a crucial role when computers or devices are attached to a network. After all, without proper IP addressing and name resolution, there’s no network communication.
There’s a lot more to distributed networking and related application servers than just strictly the pure network management stack performed by DNS and DHCP servers. In the following sections, we’ll take a quick tour of some of the most relevant application servers running across distributed Linux systems.
Authentication servers
Standalone Linux systems typically use the default authentication mechanism, where user credentials are stored in the local filesystem (such as `/etc/passwd` and `/etc/shadow`). We explored the related user authentication internals in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing* *Users and Groups*. However, as we extend the authentication boundary beyond the local machine – for example, accessing a file or email server – having the user credentials shared between the remote and localhosts would become a serious security issue.
Ideally, we should have a centralized authentication endpoint across the network that’s handled by a secure authentication server. User credentials should be validated using robust encryption mechanisms before users can access remote system resources.
Let’s consider the secure access to a network share on an arbitrary file server. Suppose the access requires **Active Directory** (**AD**) user authentication. Creating the related mount (share) locally on a user’s client machine will prompt for user credentials. The authentication request is made by the file server (on behalf of the client) to an authentication server. If the authentication succeeds, the server share becomes available to the client. The following diagram represents a simple remote authentication flow between a client and a server, using a **Lightweight Directory Access Protocol** (**LDAP**) authentication endpoint:
![Figure 7.33 – Authentication workflow with LDAP](img/B19682_07_33.jpg)

Figure 7.33 – Authentication workflow with LDAP
Here are some examples of standard secure authentication platforms (available for Linux):

*   **Kerberos** ([https://web.mit.edu/kerberos/](https://web.mit.edu/kerberos/))
*   **LDAP** ([https://www.redhat.com/en/topics/security/what-is-ldap-authentication](https://www.redhat.com/en/topics/security/what-is-ldap-authentication))
*   **Remote Authentication Dial-In User Service** (**RADIUS**) ([https://freeradius.org/documentation/](https://freeradius.org/documentation/))
*   **Diameter** ([https://www.f5.com/glossary/diameter-protocol](https://www.f5.com/glossary/diameter-protocol))
*   **Terminal Access Controller Access-Control System** (**TACACS+**) ([https://datatracker.ietf.org/doc/rfc8907/](https://datatracker.ietf.org/doc/rfc8907/))

A Linux LDAP authentication server can be configured using OpenLDAP, which was covered in the first edition of this book.
In this section, we illustrated the authentication workflow with an example of using a file server. To remain on topic, we’ll look at network file-sharing services next.
File sharing
In common networking terms, **file sharing** represents a client machine’s ability to *mount* and access a remote filesystem belonging to a server, as if it were local. Applications running on the client machine would access the shared files directly on the server. For example, a text editor can load and modify a remote file, and then save it back to the same remote location, all in a seamless and transparent operation. The underlying remoting process – the appearance of a remote filesystem acting as local – is made possible by file-sharing services and protocols.
For every file-sharing network protocol, there is a corresponding client-server file-sharing platform. Although most network file servers (and clients) have cross-platform implementations, some operating system platforms are better suited for specific file-sharing protocols, as we’ll see in the following subsections. Choosing between different file-server implementations and protocols is ultimately a matter of compatibility, security, and performance.
Here are some of the most common file-sharing protocols, with some brief descriptions for each:

*   **Server Message Block** (**SMB**): The SMB protocol provides network discovery and file- and printer-sharing services. SMB also supports interprocess communication over a network. SMB is a relatively old protocol, developed by **International Business Machines Corporation** (**IBM**) in the 1980s. Eventually, Microsoft took over and made some considerable alterations to what became the current version through multiple revisions (SMB 1.0, 2.0, 2.1, 3.0, 3.0.2, and 3.1.1).
*   **Common Internet File System** (**CIFS**): This protocol is a particular implementation of the SMB protocol. Due to the underlying protocol similarity, SMB clients can communicate with CIFS servers and vice versa. Though SMB and CIFS are idiomatically the same, their internal implementation of file locking, batch processing, and – ultimately – performance is quite different. Apart from legacy systems, CIFS is rarely used these days. SMB should always be preferred over CIFS, especially with the more recent revisions of SMB 2 or SMB 3.
*   **Samba**: As with CIFS, Samba is another implementation of the SMB protocol. Samba provides file- and print-sharing services for Windows clients on a variety of server platforms. In other words, Windows clients can seamlessly access directories, files, and printers on a Linux Samba server, just as if they were communicating with a Windows server.

    As of version 4, Samba natively supports Microsoft AD and Windows NT domains. Essentially, a Linux Samba server can act as a domain controller on a Windows AD network. Consequently, user credentials on the Windows domain can transparently be used on the Linux server without being recreated, and then manually kept in sync with the AD users.

*   **Network File System** (**NFS**): This protocol was developed by Sun Microsystems and essentially operates on the same premise as SMB – accessing files over a network as if they were local. NFS is not compatible with CIFS or SMB, meaning that NFS clients cannot communicate directly with SMB servers or vice versa.
*   **Apple Filing Protocol** (**AFP**): The AFP is a proprietary file-sharing protocol designed by Apple and exclusively operates in macOS network environments. We should note that besides AFP, macOS systems also support standard file-sharing protocols, such as SMB and NFS.

Most of the time, NFS is the file-sharing protocol of choice within Linux networks. For mixed networking environments – such as Windows, Linux, and macOS interoperability – Samba and SMB are best suited for file sharing.
Some file-sharing protocols (such as SMB) also support print sharing and are used by print servers. We’ll take a closer look at print sharing next.
Printer servers
A **printer server** (or **print server**) connects a printer to client machines (computers or mobile devices) on a network using a printing protocol. Printing protocols are responsible for the following remote printing tasks over a network:

*   Discovering printers or print servers
*   Querying printer status
*   Sending, receiving, queueing, or canceling print jobs
*   Querying print job status

Common printing protocols include the following:

*   **Line Printer Daemon** (**LPD**) protocol
*   **Generic protocols**, such as SMB and TELNET
*   **Wireless printing**, such as AirPrint by Apple
*   **Internet printing protocols**, such as Google Cloud Print

Among the generic printing protocols, SMB (also a file-sharing protocol) was previously described in the *File sharing* section. The TELNET communication protocol was described in the *Remote* *access* section.
File- and printer-sharing services are mostly about *sharing* documents, digital or printed, between computers on a network. When it comes to *exchanging* documents, additional network services come into play, such as *file transfer* and *email* services. We’ll look at file transfer next.
File transfer
FTP is a standard network protocol for transferring files between computers on a network. FTP operates in a client-server environment, where an FTP client initiates a remote connection to an FTP server, and files are transferred in either direction. FTP maintains a `21`, and it’s used for exchanging commands between the client and the server. Data connections are exclusively used for data transfer and are negotiated between the client and the server (through the control connection). Data connections usually involve ephemeral ports for inbound traffic, and they only stay open during the actual data transfer, closing immediately after the transfer completes.
FTP negotiates data connections in one of the following two modes:

*   `PORT` command to the FTP server, signaling that the client *actively* provides the inbound port number for data connections
*   `PASV` command to the FTP server, indicating that the client *passively* awaits the server to supply the port number for inbound data connections

FTP is a relatively *messy* protocol when it comes to firewall configurations due to the dynamic nature of the data connections involved. The control connection port is usually well known (such as port `21` for insecure FTP) but data connections originate on a different port (usually `20`) on either side, while on the receiving end, the inbound sockets are opened within a preconfigured ephemeral range (`1024` to `65535`).
FTP is most often implemented securely through either of the following approaches:

*   `990`.
*   `22`. For more information on the SSH protocol and client-server connectivity, refer to *SSH* in the *Remote access* section, later in this chapter.

Next, we’ll look at mail servers and the underlying email exchange protocols.
Mail servers
A **mail server** (or **email server**) is responsible for email delivery over a network. A mail server can either exchange emails between clients (users) on the same network (domain) – within a company or organization – or deliver emails to other mail servers, possibly beyond the local network, such as the internet.
An email exchange usually involves the following actors:

*   An **email client** application (such as Outlook or Gmail)
*   One or more **mail servers** (Exchange or Gmail server)
*   The **recipients** involved in the email exchange – a *sender* and one or more *receivers*
*   An **email protocol** that controls the communication between the email client and the mail servers

The most used email protocols are **POP3**, **IMAP**, and **SMTP**. Let’s take a closer look at each of these protocols.
POP3
**POP version 3** (**POP3**) is a standard email protocol for receiving and downloading emails from a remote mail server to a local email client. With POP3, emails are available for reading offline. After being downloaded, emails are usually removed from the POP3 server, thus saving up space. Modern-day POP3 mail client-server implementations (Gmail, Outlook, and others) also have the option of keeping email copies on the server. Persisting emails on the POP3 server becomes very important when users access emails from multiple locations (client applications).
The default POP3 ports are outlined here:

*   `110`: For insecure (non-encrypted) POP3 connections
*   `995`: For secure POP3 using SSL/TLS encryption

POP3 is a relatively old email protocol that’s not always suitable for modern-day email communications. When users access their emails from multiple devices, IMAP is a better choice. We’ll look at the IMAP email protocol next.
IMAP
IMAP is a standard email protocol for accessing emails on a remote IMAP mail server. With IMAP, emails are always retained on the mail server, while a copy of the emails is available for IMAP clients. A user can access emails on multiple devices, each with their IMAP client application.
The default IMAP ports are outlined here:

*   `143`: For insecure (non-encrypted) IMAP connections
*   `993`: For secure IMAP using SSL/TLS encryption

Both POP3 and IMAP are standard protocols for receiving emails. To send emails, SMTP comes into play. We’ll take a look at the SMTP email protocol next.
SMTP
SMTP is a standard email protocol for sending emails over a network or the internet.
The default SMTP ports are outlined here:

*   `25`: For insecure (non-encrypted) SMTP connections
*   `465` or `587`: For secure SMTP using SSL/TLS encryption

When using or implementing any of the standard email protocols described in this section, it is always recommended to use the corresponding secure implementation with the most up-to-date TLS encryption, if possible. POP3, IMAP, and SMTP also support user authentication, an added layer of security – this is also recommended in commercial or enterprise-grade environments.
To get an idea of how the SMTP protocol operates, let’s go through some of the initial steps for initiating an SMTP handshake with Google’s Gmail SMTP server.
We will start by connecting to the Gmail SMTP server, using a secure (TLS) connection via the `openssl` command, as follows:

```
openssl s_client -starttls smtp -connect smtp.gmail.com:587
```

 Here, we invoked the `openssl` command, simulated a client (`s_client`), started a TLS SMTP connection (`-starttls smtp`), and connected to the remote Gmail SMTP server on port `587` `(-``connect smtp.gmail.com:587`).
The Gmail SMTP server responds with a relatively long TLS handshake block that ends with the following code:
![Figure 7.34 – Initial TLS handshake with a Gmail SMTP server](img/B19682_07_34.jpg)

Figure 7.34 – Initial TLS handshake with a Gmail SMTP server
While still inside the `openssl` command’s interactive prompt, we initiate the SMTP communication with a `HELO` command (spelled precisely as such). The `HELO` command *greets* the server. It is a specific SMTP command that starts the SMTP connection between a client and a server. There is also an `EHLO` variant, which is used for ESMTP service extensions. Google expects the following `HELO` greeting:

```
HELO hellogoogle
```

 Another handshake follows, ending with `250 smtp.gmail.com at your service`, as illustrated here:
![Figure 7.35 – The Gmail SMTP server is ready for communication](img/B19682_07_35.jpg)

Figure 7.35 – The Gmail SMTP server is ready for communication
Next, the Gmail SMTP server requires authentication via the `AUTH LOGIN` SMTP command. We won’t go into further details, but the key point to be made here is that the SMTP protocol follows a plaintext command sequence between the client and the server. It’s very important to adopt a secure (encrypted) SMTP communication channel using TLS. The same applies to any of the other email protocols (POP3 and IMAP).
So far, we’ve covered several network services, some of them spanning multiple networks or even the internet. Network packets carry data and destination addresses within the payload, but there are also synchronization signals between the communication endpoints, mostly to discern between sending and receiving workflows. The synchronization of network packets is based on timestamps. Reliable network communications would not be possible without a highly accurate time-synchronization between network nodes. We’ll look at network timekeepers next.
NTP servers
NTP is a standard networking protocol for clock synchronization between computers on a network. NTP attempts to synchronize the system clock on participating computers within a few milliseconds of **Coordinated Universal Time** (**UTC**) – the world’s time reference.
The NTP protocol’s implementation usually assumes a client-server model. An NTP server acts as a time source on the network by either broadcasting or sending updated **timestamp datagrams** to clients. An NTP server continually adjusts its system clock according to well-known accurate time servers worldwide, using specialized algorithms to mitigate network latency.
A relatively easy way to check the NTP synchronization status on our Linux platform of choice is by using the `ntpstat` utility. `ntpstat` may not be installed by default on our system. On Ubuntu, we can install it with the following command:

```
sudo apt install ntpstat
```

 On Fedora, we can install `ntpstat` with the following command:

```
sudo dnf install ntpstat
```

 `ntpstat` requires an NTP server to be running locally. To set up a local NTP server, you will need to do the following (all examples shown here are for Ubuntu 22.04.2 LTS):

*   Install the `ntp` package with the following command:

    ```
    ntp service’s status:

    ```
    ntp service:

    ```
    sudo systemctl enable ntp
    ```

    ```

    ```

     *   Modify the firewall settings:

    ```
    ntpdate package:

    ```
    ntp service:

    ```
    sudo systemctl restart ntp
    ```

    ```

    ```

Before installing the `ntp` utility, take into account that Ubuntu is using another tool instead of `ntpd` by default, named `timesyncd`. When installing `ntpd`, the default utility will be disabled.
To query the NTP synchronization status, we can run the following command:

```
ntpstat
```

 This is the output:
![Figure 7.36 – Querying the NTP synchronization status with ntpstat](img/B19682_07_36.jpg)

Figure 7.36 – Querying the NTP synchronization status with ntpstat
`ntpstat` provides the IP address of the NTP server the system is synchronized with (`31.209.85.242`), the synchronization margin (`29` milliseconds), and the time-update polling interval (`64` seconds). To find out more about the NTP server, we can `dig` its IP address with the following command:

```
dig -x 31.209.85.242
```

 It looks like it’s one of the `lwlcom` time servers (`ntp1.lwlcom.net`), as shown here:
![Figure 7.37 – Querying the NTP synchronization status with ntpstat](img/B19682_07_37.jpg)

Figure 7.37 – Querying the NTP synchr[onization status with ntpstat](https://en.wikipedia.org/wiki/Network_Time_Protocol)
[The NTP client-server](https://en.wikipedia.org/wiki/Network_Time_Protocol) communication uses UDP as the transport protocol on port `123`. [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing* *Linux*, has a section dedicated to installing and configuring an NTP server. For more information on NTP, you can refer to [https://en.wikipedia.org/wiki/Network_Time_Protocol](https://en.wikipedia.org/wiki/Network_Time_Protocol).
With that, our brief journey into networking servers and protocols has come to an end. Everyday Linux administration tasks often require some sort of remote access to a system. There are many ways to access and manage computers remotely. The next section describes some of the most common remote-access facilities and related network protocols.
Remote access
Most Linux network services provide a relatively limited **remote management** interface, with their management **command-line interface** (**CLI**) utilities predominantly operating locally on the same system where the service runs. Consequently, the related administrative tasks assume local Terminal access. Direct console access to the system is sometimes not possible. This is when remote-access servers come into play to enable a virtual Terminal login session with the remote machine.
Let’s look at some of the most common remote-access services and applications.
SSH
SSH is perhaps the most popular secure login protocol for remote access. SSH uses strong encryption, combined with user authentication mechanisms, for secure communication between a client and a server machine. SSH servers are relatively easy to install and configure, and the *Setting up an SSH server* section in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*, is dedicated to describing the related steps. The default network port for SSH is `22`.
SSH supports the following authentication types:

*   Public-key authentication
*   Password authentication
*   Keyboard-interactive authentication

The following sections provide brief descriptions of these forms of SSH authentication.
Public-key authentication
**Public-key authentication** (or **SSH-key authentication**) is arguably the most common type of SSH authentication.
Important note
This section will use the terms *public-key* and *SSH-key* interchangeably, mostly to reflect the related SSH authentication nomenclature in the Linux community.
The SSH-key authentication mechanism uses a *certificate/key* pair – a `ssh-keygen` tool, using standard encryption algorithms such as the **Rivest–Shamir–Adleman** algorithm (**RSA**) or the **Digital Signature** **Algorithm** (**DSA**).
The SSH public-key authentication supports either **user-based authentication** or **host-based authentication** models. The two models differ in the ownership of the certificate/key pairs involved. With client authentication, each user has a certificate/key pair for SSH access. On the other hand, host authentication involves a single certificate/key pair per system (host).
Both SSH-key authentication models are illustrated and explained in the following sections. The basic SSH handshake and authentication workflows are the same for both models:

*   First, the SSH client generates a secure certificate/key pair and shares its public key with the SSH server. This is a one-time operation for enabling public-key authentication.
*   When a client initiates the SSH handshake, the server asks for the client’s public key and verifies it against its allowed public keys. If there’s a match, the SSH handshake succeeds, the server shares its public key with the client, and the SSH session is established.
*   Further client-server communication follows standard encryption/decryption workflows. The client encrypts the data with its private key, while the server decrypts the data with the client’s public key. When responding to the client, the server encrypts the data with its own private key, and the client decrypts the data with the server’s public key.

SSH public-key authentication is also known as **passwordless authentication**, and it’s frequently used in automation scripts where commands are executed over multiple remote SSH connections without prompting for a password.
Let’s take a closer look at the user-based and host-based public-key authentication mechanisms:

*   **User-based authentication**: This is the most common SSH public-key authentication mechanism. According to this model, every user connecting to a remote SSH server has its own SSH key. Multiple user accounts on the same host (or domain) would have different SSH keys, each with its own access to the remote SSH server, as suggested in the following figure:

![Figure 7.38 – User-based key authentication](img/B19682_07_38.jpg)

Figure 7.38 – User-based key authentication

*   **Host-based authentication**: This is another form of SSH public-key authentication and involves a single SSH key per system (host) to connect to a remote SSH server, as illustrated in the following figure:

![Figure 7.39 – Host-based key authentication](img/B19682_07_39.jpg)

Figure 7.39 – Host-based key authentication
With host-based authentication, the underlying SSH key can only authenticate SSH sessions that originated from a single client host. Host-based authentication allows multiple users to connect from the same host to a remote SSH server. If a user attempts to use a host-based SSH key from a different machine than the one allowed by the SSH server, access will be denied.
Sometimes, a mix of the two public-key authentications is used – user- and host-based authentication –an approach that provides an increased level of security to SSH access.
When security is not critical, simpler SSH authentication mechanisms could be more suitable. Password authentication is one such mechanism.
Password authentication
`/etc/passwd`) or select user accounts defined in the SSH server configuration (`/etc/ssh/sshd_config`). The SSH server configuration described in [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing* *Linux*, further elaborates on this subject.
Besides local authentication, SSH can also leverage remote authentication methods such as Kerberos, LDAP, RADIUS, and others. In such cases, the SSH server delegates the user authentication to a remote authentication server, as described in the *Authentication servers* section earlier in this chapter.
Password authentication requires either user interaction or some automated way to provide the required credentials. Another similar authentication mechanism is keyboard-interactive authentication, described next.
Keyboard-interactive authentication
**Keyboard-interactive authentication** is based on a dialogue of multiple challenge-response sequences between the SSH client (user) and the SSH server. This dialogue is a plaintext exchange of questions and answers, where the server may prompt the user for any number of challenges. In some respect, password authentication is a **single-challenge interactive** **authentication** mechanism.
The *interactive* connotation of this authentication method could lead us to think that user interaction would be mandatory for the related implementation. Not really. Keyboard-interactive authentication could also serve implementations of authentication mechanisms based on custom protocols, where the underlying message exchange would be modeled as an authentication protocol.
Before moving on to other remote access protocols, we should call out the wide use of SSH due to its security, versatility, and performance. However, SSH connectivity may not always be possible or adequate in specific scenarios. In such cases, *TELNET* may come to the rescue. We’ll take a look at it next.
TELNET
**TELNET** is an application-layer protocol for bidirectional network communication that uses a plaintext CLI with a remote host. Historically, TELNET was among the first remote-connection protocols, but it always lacked secure implementation. SSH eventually became the standard way to log in from one computer to another, yet TELNET has its advantages over SSH when it comes to troubleshooting various application-layer protocols, such as web- or email-server communication. You will learn more about how to use TELNET in [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing* *Linux*.
TELNET and SSH are command-line-driven remote-access interfaces. There are cases when a direct desktop connection is needed to a remote machine through a **graphical user interface** (**GUI**). We’ll look at desktop sharing next.
VNC
**Virtual Network Computing** (**VNC**) is a desktop-sharing platform that allows users to access and control a remote computer’s GUI. VNC is a cross-platform client-server application. A VNC server running on a Linux machine, for example, allows desktop access to multiple VNC clients running on Windows or macOS systems. The VNC network communication uses the **Remote Framebuffer** (**RFB**) protocol, defined by *RFC 6143*. Setting up a VNC server is relatively simple. VNC assumes the presence of a graphical desktop system. More details on this will be provided in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring* *Linux Servers*.
This concludes our section about network services and protocols. We tried to cover the most common concepts about general-purpose network servers and applications, mostly operating in a client-server or distributed fashion. With each network server, we described the related network protocols and some of the internal aspects involved. [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing Linux*, and [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*, will showcase practical implementations for some of these network servers.
In the next section, our focus will turn to network security internals.
Understanding network security
**Network security** represents the processes, actions, and policies to prevent, monitor, and protect unauthorized access to computer networks. Network security paradigms span a vast array of technologies, tools, and practices. Here are a few important ones:

*   **Access control**: Selectively restricting access based on user authentication and authorization mechanisms. Examples of access control include users, groups, and permissions. Some of the related concepts were covered in [*Chapter 4*](B19682_04.xhtml#_idTextAnchor090), *Managing* *Users* *and Groups*.
*   **Application security**: Securing and protecting server and end user applications (email, web, and mobile apps). Examples of application security include **Security-Enhanced Linux** (**SELinux**), strongly encrypted connections, antivirus, and anti-malware programs. We’ll cover **SELinux** in [*Chapter 10*](B19682_10.xhtml#_idTextAnchor212), *Disaster Recovery, Diagnostics,* *and Troubleshooting*.
*   **Endpoint security**: Securing and protecting servers and end user devices (smartphones, laptops, and desktop PCs) on the network. Examples of endpoint security include *firewalls* and various intrusion-detection mechanisms. We’ll look at firewalls in [*Chapter 10*](B19682_10.xhtml#_idTextAnchor212), *Disaster Recovery, Diagnostics,* *and Troubleshooting*.
*   **Network segmentation**: Partitioning computer networks into smaller segments or **virtual LANs** (**VLANs**). This is not to be confused with subnetting, which is a logical division of networks through addressing.
*   **VPNs**: Accessing corporate networks using a secure encrypted tunnel from public networks or the internet. We’ll look at VPNs in more detail in [*Chapter 9*](B19682_09.xhtml#_idTextAnchor194), *Securing Linux*, and [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring* *Linux Servers*.

In everyday Linux administration, setting up a network security perimeter should always follow the paradigms enumerated previously, roughly in the order listed. Starting with access-control mechanisms and ending with VPNs, securing a network takes an *inside-out* approach, from local systems and networks to firewalls, VLANs, and VPNs.
Summary
This chapter provided a relatively condensed view of basic Linux networking principles. We learned about network communication layers and protocols, IP addressing schemes, TCP/IP configurations, well-known network application servers, and VPN. A good grasp of networking paradigms will give Linux administrators a more comprehensive view of the distributed systems and underlying communication between the application endpoints involved.
Some of the theoretical aspects covered in this chapter will be taken for a practical spin in [*Chapter 13*](B19682_13.xhtml#_idTextAnchor276), *Configuring Linux Servers*, where we’ll focus on real-world implementations of network servers. [*Chapter 10*](B19682_10.xhtml#_idTextAnchor212), *Disaster Recovery, Diagnostics, and Troubleshooting*, will further explore network security internals and practical Linux firewalls. Everything we have learned so far will serve as a good foundation for the assimilation of these upcoming chapters.
The following chapter will introduce you to Linux shell scripting, where you will learn about the most common shell features and how to use decisions, loops, variables, arrays, and functions.
Questions
Here’s a quick quiz to outline and test some of the essential concepts covered in this chapter:

1.  How does the OSI model compare to the TCP/IP model?

    **Hint**: *Figure 7**.2* could be of help.

2.  Think of a couple of TCP/IP protocols and try to see where and how they operate in some of the network administration tasks or applications you are familiar with.
3.  At what networking layer does the HTTP protocol operate? How about DNS?

    **Hint**: Both operate on the same layer.

4.  What is the network class for IP address `192.168.0.1`?

    **Hint**: Refer to *Figure 7**.5*.

5.  What is the network prefix that corresponds to network mask `255.255.0.0`?

    **Hint**: Check out *Figure* *7**.5* again.

6.  How do you configure a static IP address using the `nmcli` utility?

    `connection modify`.

7.  How do you change the hostname of a Linux machine?

    `hostnamectl` utility.

8.  What is the difference between the POP3 and IMAP email protocols?
9.  How does SSH host-based authentication differ from user-based SSH key authentication?
10.  What is the difference between SSH and TELNET?

Further reading
For more information about what was covered in this chapter, please refer to the following Packt titles:

*   *Linux Administration Best Practices*, by Scott Alan Miller
*   *Linux for Networking Professionals*, by Rob VandenBrink

```

```