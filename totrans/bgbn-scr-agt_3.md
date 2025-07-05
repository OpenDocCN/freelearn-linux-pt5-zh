# Chapter 3. Adding Hardware Security with the CryptoCape

This chapter continues our custom security hardware journey by using a **BeagleBone Black (BBB) cape**. In BeagleBone parlance, a **cape** is a daughterboard that attaches to the BBB. We'll briefly introduce hardware cryptographic devices and then explore the CryptoCape, which is a BBB cape containing numerous security features. We'll describe the process of creating your own cape using the CryptoCape as an example. This chapter introduces the crypto chips on the cape and shows you how to implement a biometric authentication device using the CryptoCape and a fingerprint scanner.

This chapter will discuss the following topics:

*   The pros and cons of hardware-based cryptography
*   An overview of the CryptoCape
*   How cape EEPROMs enable automatic hardware configuration on the BBB
*   How to use the CryptoCape's real-time clock
*   How to program an ATmega328p from BBB
*   How to implement a biometric authentication system

# Exploring the differences between hardware and software cryptography

In the following sections, we'll discuss the advantages and disadvantages of using hardware-based cryptography. The remaining projects in the book will use embedded hardware cryptographic devices, so it's important to know their capabilities and their limitations. Refer to [Chapter 1](part0015_split_000.html#page "Chapter 1. Creating Your BeagleBone Black Development Environment"), *Creating Your BeagleBone Black Development Environment*, for additional resources on cryptography and terms used in this section.

## Understanding the advantages of hardware-based cryptography

For the advantages of hardware cryptography, we'll focus on the embedded environment since that is the target use case of BBB. While chip manufacturers may provide a laundry list of advantages, there are two main categories: cryptographic acceleration and key isolation features.

### Offloading computation to a separate processor

One advantage of using a dedicated cryptographic co-processor is to offload computation to reduce CPU usage. A typical example is using hardware to perform the **Advanced Encryption Standard** (**AES**) encryption and decryption operations in a **Transport Layer Security** (**TLS**) session.

TLS is most commonly used in conjunction with the **Hypertext Transfer Protocol Secure** (**HTTPS**) protocol. You use HTTPS every time you buy something online to protect your credit card information. Depending on your browser, you may notice a lock icon or a green bar to indicate when a web page is served over HTTPS. In a TLS session, the client, your browser, and the server will negotiate to use the same symmetric key. While there are several symmetric ciphers that can be negotiated, AES is one of the preferred choices.

### Note

While some sites automatically redirect you to the HTTPS version of the site, often you must manually specify this. Remembering to type `https://` is often annoying but fortunately there is a cross-browser plugin that will automatically redirect you to the HTTPS site, if there is one. The plugin is called **HTTPS Everywhere** and it is maintained by the Electronic Frontier Foundation. Information and links to download the free software are located at [https://www.eff.org/https-everywhere](https://www.eff.org/https-everywhere).

In the crypto accelerator role, a cryptographic co-processor would perform the encryption and decryption of each TLS record. This offloads the main CPU to handle the processing of the network traffic and perform the intended application. The BBB actually has such a cryptographic co-processor. **Texas Instruments** (**TI**) crypto performance page for the AM335x, the processor on the BBB, shows the results of their benchmark tests with OpenSSL. Using AES with a 256 bit key size and operating on blocks of 8192 bytes, the measured throughput of data was 8129.19 kB/sec without using crypto acceleration. This test resulted in a CPU usage of 69 percent. However, using the crypto co-processor on the AM335x, they measured a throughput of 24376.66 kB/sec with a CPU usage of 41 percent. That's almost a 200 percent gain in throughput performance and a 40 percent drop in CPU usage!

### Note

More information on the crypto accelerators on the AM335x can be found on TI Crypto Performance page at [http://processors.wiki.ti.com/index.php/AM335x_Crypto_Performance](http://processors.wiki.ti.com/index.php/AM335x_Crypto_Performance).

If your embedded application's purpose is to perform a computationally intense calculation, using a cryptographic co-processor designed to offload the crypto processing can save your CPU cycles for your main program.

### Protecting keys through physical isolation

A major advantage of using hardware cryptography devices is that keys can be generated internal to the device and are designed to be difficult to remove. In the web server world, these devices are called **Hardware Security Modules** (**HSMs**). In April 2014, a major vulnerability was announced that affected the OpenSSL software, colloquially called **heartbleed**. The vulnerability was not a cryptographic one *per se*, but rather the result of a programming error called a **buffer overrun**. This vulnerability is unfortunately common in the C programming language, in which OpenSSL is written, because of the lack of automatic array bounds checking. It was possible for a client to exploit this error and view internal memory of the server, potentially discovering sensitive information. The code to fix this problem was shorter than this paragraph.

### Note

The following `xkcd` comic provides a succinct and amusing explanation of the heartbleed bug:

[https://xkcd.com/1354/](https://xkcd.com/1354/)

The scale and severity of this vulnerability cannot be overstated. The most damaging attack is if the server's private key was leaked. Knowing the private key, an attacker can impersonate the server, and clients could willingly disclose private information to the impostor. However, on a server with a HSM, the heartbleed vulnerability was more limited. Sensitive information, such as session cookies, would still have leaked via the software exploit. Yet the server's private key, which remains in the HSM, would not have leaked.

Since the hardware cryptographic co-processor runs as a physically separate machine, it is very difficult for software exploits running on the main processor to disclose secrets in the hardware module. In the embedded world, there are several chips that perform this key isolation feature.

## Understanding the disadvantages of hardware crypto devices

Adding hardware cryptographic devices doesn't automatically make your project *secure*. In the following sections, we'll discuss some of the downsides to using cryptographic hardware and some of the concerns you would need to resolve when using them in your project.

### Lacking cryptographic flexibility

A hardware cryptographic device is generally not configurable. This is usually by design but the implication is that it is difficult, if not impossible, to alter the cryptographic behavior of the device. If you select a chip that performs encryption with a limited key size, you will not be able to upgrade the device with a stronger key size later. It is generally easier to update software-based encryption systems.

### Exposing hardware-specific attack vectors

While an isolated crypto processor may reduce attacks or exploits from the software, it often allows more sophisticated hardware-based attacks. There are three categories of hardware-based attacks: noninvasive, invasive, and semi-invasive (Skoroboga, 2011). Noninvasive attacks treat the chip as a black box and attempt to manipulate the surrounding environment to perform an exploit. Successful non-invasive attacks include performing a **Differential Power Analysis** (**DPA**) to monitor the chip as it performs an encryption algorithm. By measuring the power usage during the encryption operation, it is possible to see the key through distinct power signatures. An invasive attack usually involves physically destroying the chip in some manner to gain access to its internals. Semi-invasive attacks may involve some sort of laser imaging to observe or interfere with the chip.

### Note

To perform a glitch attack, an attack attempts to manipulate the executing instruction by injecting a fault (Bar-El, 2004). Colin O'Flynn, a security researcher, has a concise and clear example of how a glitch attack can cause a password-checking microprocessor to fail: [https://www.youtube.com/watch?v=Ruphw9-8JWE&list=UUqc9MJwX_R1pQC6A353JmJg](https://www.youtube.com/watch?v=Ruphw9-8JWE&list=UUqc9MJwX_R1pQC6A353JmJg).

### Obfuscating implementation details

A final consideration is that while chip vendors may publish the interface to their device, the internals are often proprietary. This is similar to a software vendor who publishes the software programming interface, but provides only the compiled binary and not the source code. In this case, the chip is treated as a black box, and without a mechanism to verify the device, you have to trust that it is operating correctly. As cryptographic libraries are ported to unique microcontrollers by the open source community, this situation will hopefully improve.

## Summarizing the hardware versus software debate

So, which is the better route? As with most complex technologies the correct answer is: *it depends*. For a truly embedded system, one that can't spare even a few extra bytes, a hardware security chip may be your only option if you can't upgrade your microprocessor. Also, if there is a high threat of leaking your key due to a software vulnerability, then the separate crypto co-processor might help you. However, if an attacker can gain physical access to your device, then they might be able to extract the key with hardware-based attacks. Lastly, if transparency is paramount for you to verify the lack or existence of backdoors, then only fully open source software and hardware devices will satisfy you.

# Touring the CryptoCape

The CryptoCape is BeagleBone's first dedicated security daughterboard. As the BBB already has cryptographic accelerations, the chips on the CryptoCape provide the *key isolation* features discussed in the previous section. In the BeagleBone community, daughterboards are called *capes* which are analogous to Arduino *shields*. The CryptoCape contains several crypto ICs that you may use in your projects. The design is open source hardware, so you may also visit the SparkFun website to retrieve the design files. The CryptoCape contains the following major components:

| Component | Manufacturer | Features |
| --- | --- | --- |
| AT97SC3205T | Atmel | TPM—RSA 2048 encryption and SHA1 hashing |
| ATAES132 | Atmel | Encrypted EEPROM with AES-128-CCM |
| ATSHA204 | Atmel | SHA-2 hashing (SHA-256, HMAC-256) |
| ATECC108 | Atmel | ECDSA with NIST curves |
| ATmega328p | Atmel | Microcontroller |
| DS3231M | Maxim integrated | Real-time clock with battery |
| CAT24C256 | ON semiconductor | EEPROM |

If you imagine the BBB's Ethernet connector as a neck, then it appears that the cutouts of the attached daughterboard seem to wrap around it as if it were wearing a cape, hence the name. Also, Boris, the beagle mascot of [BeagleBoard.org](http://BeagleBoard.org), looks more adorable in a cape.

Each of these chips and the associated circuits are clearly labeled on the CryptoCape board:

![Touring the CryptoCape](img/00011.jpeg)

In the following sections, we'll briefly introduce each component and provide some example project ideas.

# Discovering the I2C protocol

Every chip on the CryptoCape uses the **Inter-Integrated Circuit** (**I2C**) Bus. I2C was developed by Phillips Semiconductor over 20 years ago, but it is still very prevalent in electronics design today. I2C requires two signal lines, one for a **Serial Clock** (**SCL**) and the other for **Serial Data** (**SDA**). Devices attached to the bus are either classified as master or slave. Each slave device has an address and there can be more than one master. The I2C protocol supports collision detection and is a true multimaster protocol.

The SDA and SCL lines are *pulled up* to the system voltage, VCC, typically with resistors connected from VCC to SDA and VCC to SCL. The processor on the BBB, the AM335x, contains internal pull-up resistors for the bus. Data is encoded to the bus by manipulating the logic levels of the SDA and SCL lines. For example, to start sending data, the master pulls SDA low while holding SCL at a logic high. The stop condition is sent with a low to high transition of SDA while holding SCL high.

### Note

Revision 6 of the I2C bus specification can be found on NXP semiconductors (previously Phillips) website at [http://www.nxp.com/documents/user_manual/UM10204.pdf](http://www.nxp.com/documents/user_manual/UM10204.pdf).

The following screenshot shows a normal I2C operation as captured by a logic analyzer. A **logic analyzer** is an instrument that can sample electrical connections and often decode the signals to produce a human readable format.

![Discovering the I2C protocol](img/00012.jpeg)

# Understanding the benefit of cape EEPROMs

At a glance, the CAT24C256 **Electrically Erasable Programmable Read Only Memory** (**EEPROM**) doesn't appear to add much value to the board. After all, the BeagleBone has a 2GB eMMC on the early revisions and a 4GB eMMC on revision C. An extra 256 kB of memory is hardly food scraps for the beagle. However, it serves a greater purpose; it's what enables automatic cape detection by the BBB.

The BBB has two 46 pin female expansion ports offering much more I/O capabilities than any other hobbyist board on the market. Certain pins can actually support eight different modes, mode 0 through mode 7\. The mapping of pin features to a mode is known as **pin muxing**, short for pin multiplexing. To use a pin in a certain mode, the software must enable and configure this pin through the kernel's interface. This can be manually performed or scripted, but the easiest method is to use a BeagleBone cape.

During the kernel startup, the software will probe the I2C bus looking for cape EEPROMs. There are four valid addresses for Cape EEPROMs, 0x54 through 0x57\. Therefore, the BBB supports up to four attached capes. The BBB will read the cape EEPROM, which must be programmed using the format in the BBB **System Reference Manual** (**SRM**). The BBB, using a software package called the **capemgr**, short for **Cape Manager**, will read the board name and revision from the Cape EEPROM. It will then try to match the name and revision to a compiled device tree fragment on your BBB. If there is a match, it will load that fragment.

### Note

The latest production files for the BBB including the schematics and the SRM are located on the BBB wiki at [http://elinux.org/Beagleboard:BeagleBoneBlack](http://elinux.org/Beagleboard:BeagleBoneBlack).

This automatic configuration provides two benefits. The first is that the pins on the BBB are automatically configured for a cape. The second is that the device tree can specify the kernel driver for the hardware on the cape, which means that the drivers for your hardware can be automatically loaded. The `capemgr` provides a plug-and-play like experience for embedded Linux.

If you are developing a BeagleBone cape, you should consider the process to have your cape supported in the BeagleBone images. This is a three-step process. First, you need to create a cape EEPROM file. This file should be written to your cape at manufacturing time. Second, you need to create a **Device Tree Source** (**DTS**) file, following the cape naming convention previously discussed, and submit a pull request on GitHub to BeagleBoard.org. Lastly, you need to create an eLinux wiki site discussing your cape. In the next sections, we'll briefly describe the software and hardware required to build a BeagleBone cape using the CryptoCape as an example.

## Creating a cape EEPROM

If you have any semi-complicated hardware for the BeagleBone, you will benefit by adding a cape-compatible EEPROM. The essential reference to populating the EEPROM is the BeagleBone SRM's section on *Cape Board Support*. This section contains the EEPROM format. To jumpstart your EEPROM file creation, you can use a tool called the **EEPROM cape generator** available at: [https://github.com/picoflamingo/BBCape_EEPROM](https://github.com/picoflamingo/BBCape_EEPROM). This tool with its simple command-line interface will provide the skeleton for your cape EEPROM. Currently, it does not completely implement the cape specification, so you must use a binary editor to set the remaining values of the EEPROM. The process to create the EEPROM binary involves reading through the SRM and writing the appropriate values, at the correct offsets, in a binary file.

You can view the CryptoCape's EEPROM by executing the following command as root:

```
cat /sys/bus/i2c/devices/1-0057/eeprom | hexdump -C

```

By default, the CryptoCape EEPROM is located at address 0x57 on the I2C bus. If you have multiple capes, you can change the address of the CryptoCape EEPROM by placing a solder jumper or solder *blob* on the A0 or A1 address pads next to the EEPROM. The results of reading the EEPROM with the previous command will produce the following:

```
00000000  aa 55 33 ee 41 31 42 42  2d 42 4f 4e 45 2d 43 52 
 |.U3.A1BB-BONE-CR|
00000010  59 50 54 4f 00 00 00 00  00 00 00 00 00 00 00 00 
 |YPTO............|
00000020  00 00 00 00 00 00 30 30  41 30 53 70 61 72 6b 46 
 |......00A0SparkF|
00000030  75 6e 00 00 00 00 00 00  00 00 42 42 2d 42 4f 4e 
 |un........BB-BON|
00000040  45 2d 43 52 59 50 54 4f  00 00 00 11 32 30 31 34 
 |E-CRYPTO....2014|
00000050  30 30 30 30 30 30 30 30  00 00 00 00 00 00 00 00 
 |00000000........|
00000060  00 00 00 00 00 00 00 00  00 00 e0 73 e0 73 00 00 
 |...........s.s..|
00000070  00 00 00 00 00 00 00 00  00 00 00 00 a0 26 c0 06 
 |.............&..|
00000080  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00 
 |................|
00000090  00 00 a0 2f 00 00 00 00  00 00 c0 17 00 00 00 00 
 |.../............|
000000a0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00 
 |................|
*
000000e0  00 00 00 00 00 00 00 00  00 00 00 00 01 f4 00 00 
 |................|
000000f0  00 00 00 00 47 50 47 20  46 69 6e 67 65 72 70 72 
 |....GPG Fingerpr|
00000100  69 6e 74 3a 20 30 78 42  35 39 31 39 42 31 41 43 
 |int: 0xB5919B1AC|
00000110  37 31 33 35 39 30 35 46  34 36 36 39 43 38 34 37 
 |7135905F4669C847|
00000120  42 46 41 35 30 33 31 42  44 32 45 44 45 41 36 0a 
 |BFA5031BD2EDEA6.|
00000130  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff 
 |................|
*
00008000

```

If you walk through the SRM EEPROM data format, you should be able to match the fields with those in the CryptoCape EEPROM. The two most important fields are the `Board Name`, which starts at offset 6 and is 32 bytes in length and the `Version`, which starts at byte 38 and is 4 bytes in length. From the previous example, the board name is `BB-BONE-CRYPTO` and the version is `00A0`. These two components are needed to name the DTS file in the next section. Starting at offset 244, the manufacturer can place any nonvolatile information. The CryptoCape contains the **GNU Privacy Guard** (**GPG**) fingerprint of the author's GPG public key, which is used to sign software packages and e-mail. In your cape, you could populate this with initial values for software or something similar.

## Creating the cape DTS file

Besides the EEPROM, you will also need to create a DTS file. This file defines attributes of your hardware to the Linux kernel. The DTS file for the CryptoCape Revision 00A0 is located on GitHub at: [https://github.com/beagleboard/linux/blob/3.8/firmware/capes/BB-BONE-CRYPTO-00A0.dts](https://github.com/beagleboard/linux/blob/3.8/firmware/capes/BB-BONE-CRYPTO-00A0.dts). When creating a new DTS for your hardware, it's best to review the existing BeagleBone DTS files. With the growing number of capes, there is a good chance that there exists an approved DTS file with the hardware configuration you seek.

The BBB device tree overlay system is one of the areas undergoing active development, and important technical nuances change rapidly. If you need to build your own DTS file, it's best to check in with the BeagleBoard.org mailing list available at [https://groups.google.com/forum/#!forum/beagleboard](https://groups.google.com/forum/#!forum/beagleboard).

### Note

A detailed introduction to the device tree system in the Linux kernel was presented by Thomas Petazzoni at the Embedded Linux Conference Europe in November 2013\. The presentation is available on YouTube at [https://www.youtube.com/watch?v=m_NyYEBxfn8](https://www.youtube.com/watch?v=m_NyYEBxfn8).

Let's briefly look at a portion of the CryptoCape's DTS file. The `capemgr` will load the compiled version of this file to configure the hardware. The drivers for some of the chips on the CryptoCape are also loaded automatically since they are specified in the DTS file, as shown in the following code:

```
fragment@2 {
        target = <&i2c2>;

        __overlay__ {
          #address-cells = <1>;
          #size-cells = <0>;

          /* Real Time Clock */
          ds1307@68 {
            compatible = "ds1307";
            reg = <0x68>;
          };

          /* TPM  */
          tpm_i2c_atmel@29 {
            compatible = "tpm_i2c_atmel";
            reg = <0x29>;
          };
        };
};
```

```
ds1307 driver. The second is the TPM driver, tpm_i2c_atmel, which is located at address 0x29\. The EEPROM driver is automatically loaded and the other chips currently do not have a native Linux kernel driver.
```

If you have your DTS file completed and want it included in the official BBB image, you can submit a pull request to the previously mentioned repository that contains the CryptoCape DTS file. Since this repository contains the existing DTS files for the BeagleBone firmware, it is well worth studying if you are building your own cape.

# Creating an eLinux wiki site

The last step is to create a Wiki site on [eLinux.org](http://eLinux.org) and e-mail `<[support@circuitco.com](mailto:support@circuitco.com)>` to let them know to link it to the main BeagleBone Capes page. The page is the main site for all BeagleBone capes and it is where the community expects to find cape information. The CryptoCape page, with links to all the supporting software and datasheet is located at [http://elinux.org/Cryptotronix:CryptoCape](http://elinux.org/Cryptotronix:CryptoCape).

The EEPROM is the defining cape component. Even if you don't manufacture a cape, you can still benefit from adding the EEPROM and creating the DTS files for your own design to take advantage of the BeagleBone's automatic hardware configuration.

# Keeping time with a real-time clock

Having all clocks synchronized throughout a system is often an assumption that doesn't hold for embedded devices. Specialized devices may not need to know the time to perform their function. However, in security protocols, accurate time keeping is often important. For example, in TLS, the X.509 certificates that are used to prove the identity of web servers contain a validity range. There is a *not before* and *not after* time that specifies when that certificate is valid. Without an accurate time keeping system, a device can't enforce this date range allowing it to possibly accept expired or not yet valid certificates.

If you use your BBB in an offline environment, but still need accurate time, then you can insert a coin cell battery into the battery compartment of the CryptoCape. When the BBB is disconnected from power, the RTC will receive enough power from the battery to maintain accurate time.

The BBB already contains an RTC; however, it lacks a dedicated battery. It is possible to power the entire BBB from a battery using the battery access pads located near the DC barrel plug adapter; however, a greater capacity battery would be needed since the entire board is powered from these pads.

The RTC driver is loaded automatically, which you can verify by running:

```
dmesg | grep rtc

```

This should result in the following:

```
[    0.729101] omap_rtc 44e3e000.rtc: rtc core: registered 44e3e000.rtc as rtc0
[    0.748605] rtc-ds1307 1-0068: rtc core: registered ds1307 as rtc1
[    0.748627] rtc-ds1307 1-0068: 56 bytes nvram
[    0.977426] [drm] Cannot find any crtc or sizes - going 1024x768
[    1.048851] omap_rtc 44e3e000.rtc: setting system clock to 2000-01-01 00:00:01 UTC (946684801)

```

The previous example shows the BBB's RTC, `omap_rtc`, registered as `rtc0` and the not-so-accurate-time of 2000-01-01 being set. The CryptoCape's RTC is `rtc1` and the time value is not manipulated.

You will have to set the RTC time initially after installing the CryptoCape. First, you'll need an accurate system time. Refer to the project from [Chapter 2](part0019_split_000.html#page "Chapter 2. Circumventing Censorship with a Tor Bridge"), *Circumventing Censorship with a Tor Bridge*, on how this is done. Set the RTC from the system time with the following command:

```
hwclock -w -f /dev/rtc1

```

Cross-check the time to ensure it was set properly:

```
hwclock -r -f /dev/rtc1

```

This should produce something like the following:

```
Tue 13 May 2014 07:29:27 PM UTC  -0.198319 seconds

```

If you use the coin cell battery from SparkFun Electronics, it has a stated capacity of 47mAh. The DS3231m draws 2 *micro* Amps when on the battery. Ideally, this would result in 23,500 hours of run time on the battery or about 2.7 *years*. In actuality, you see much less run time from your battery, but even if the battery dies in half of the ideal time, you should still see plenty of battery life from your RTC.

# Trusting computing devices with a Trusted Platform Module

The **Trusted Platform Module** (**TPM**) performs the RSA algorithm on the chip. However, it is much more capable than just an encryption chip. The TPM specification is developed and maintained by the **Trusted Computing Group** (**TCG**), an international industry standards body.

TPMs are included on several major vendor laptops including Dell, HP, and even Google Chromebooks. On laptops, TPMs are normally found in the **Low Pin Count** (**LPC**) package and are enabled via the BIOS. Embedded devices typically don't support the LPC bus; the TPM on the CryptoCape communicates over the I2C bus.

The software interface to the TPM is via the **Trusted Computing Group Software Stack** (**TSS**). In Linux, the TSS is provided by the TrouSerS package. In the next chapter, we'll be using the TPM and also take a closer look at the TPM on the CryptoCape.

# Providing hardware authentication with ATSHA204 and ATECC108

Both ATSHA204 and ATECC108 are authentication devices. **Authentication** is the process of guaranteeing the identity of communicating parties and ensuring data integrity. Each chip uses a different approach to authentication. The ATECC108 device uses elliptical curve cryptography to provide the **Elliptical Curve Digital Signature Algorithm** (**ECDSA**). The ATSHA024 device uses a hash algorithm, SHA-256, to provide **Hash Based Message Authentication Codes** (**HMAC**).

### Note

These two devices are not used in this book, but you can download the software from [https://github.com/cryptotronix/hashlet](https://github.com/cryptotronix/hashlet) and [https://github.com/cryptotronix/eclet](https://github.com/cryptotronix/eclet) and see the example usage for ATSHA204 and ATECC108 respectively.

# Encrypting EEPROM data with the ATAES132

The ATAES132 is a 32kb EEPROM that can be encrypted with AES using a 128-bit key using the **Counter with CBC-MAC** (**CCM**) mode. CCM provides an authenticated encryption mode. With an encrypted EEPROM, you can store small amounts of data-at-rest more securely. The ATAES132 also has the ability to encrypt and decrypt small packets of up to 32 bytes and return the result over the bus. The AES key remains in the device at all times.

At the time of writing this, there isn't a Linux driver for the ATAES132 device, but Atmel provides full documentation and an AVR-based library on their website at [http://www.atmel.com/devices/ataes132.aspx](http://www.atmel.com/devices/ataes132.aspx).

# Combining the BBB with an ATmega328p

Lastly, the CryptoCape contains an independent microcontroller. This microcontroller is the **ATmega328p**, which is the same microcontroller on the Arduino UNO. However, because the supply voltage is 3.3V and not 5V like the Arduino UNO, the processor runs at a clock speed of 8MHz versus 16Mhz. In this sense, it is more like the 3.3V Arduino Pro Mini. Shipped from SparkFun, the CryptoCape contains the 3.3V Arduino Pro Mini bootloader. Like any other Arduino-based board, you can reflash the bootloader by attaching an **In-System Programming** (**ISP**) programmer to the ISP headers next to the ATmega328p. SparkFun's pocket programmer is an inexpensive tool to perform this task. Just be sure to set the switch to *no power target* since the CryptoCape is powered from the BBB.

While you can use an ISP programmer, you can also use the BBB as a programmer. The BBB contains both serial UART and SPI; however, only the serial UART, UART 4, is connected to the onboard ATmega328p. Like the Arduino UNO, if the microprocessor is reset, it will accept uploaded programs over the serial line.

A simple script that toggles the BBB GPIO connected to ATmega's reset line, which is GPIO 49 for the CryptoCape, and then uploads the hex file using `avrdude` will do the trick. The script is part of a GitHub repository which can be cloned with the command:

```
git clone https://github.com/jbdatko/BBB_ATmega328P_flasher.git

```

In this repository, there is an `upload.sh` script, the main logic of which contains the following code snippet:

```
(echo 0 > /sys/class/gpio/gpio49/value \
    && sleep $tts \
    && echo 1 > \
    /sys/class/gpio/gpio49/value) &

avrdude -c arduino -p m328p -b 57600 -v \
    -P /dev/ttyO4 -U flash:w:$1
```

The first part toggles the ATmega reset line low and sleeps for `$tts`, which is currently defined to be .9 seconds. Then the script sets the reset line back to high to let the ATmega run. The next line is the `avrdude` command to upload your hex file. You'll need to install `avrdude` with the following command:

```
sudo apt-get install avrdude

```

Then, to run the script, perform the following:

```
sudo ./upload Blink.cpp.hex

```

Prior to flashing an Arduino sketch, you must have a sketch to flash. Depending on your development preference, there are several options. You can build your AVR program on the BBB with a makefile and `gcc-avr`. Or you can build your program with Atmel's free AVR Studio or the Arduino IDE. Whichever way you choose, you need to find the compiled `hex` file and download it to the BBB using `sftp` or a similar tool.

Before you upload sketches to the CryptoCape's ATmega, you'll need to perform one very important step: you need to attach jumpers to the two *Program Jumper* pins. Without the jumpers, the ATmega is not electrically connected to the BBB serial lines. This is a security feature. Using the Arduino bootloader, it is possible to upload `hex` files using serial UART. Thus, with the jumpers installed, the sketch on the CryptoCape's ATmega can always be modified. Now imagine if the BBB was inflicted with malware, but had the jumpers removed. This malware can't change the software on the ATmega. It can still do lots of other nasty things including resetting the ATmega, but it can't upload new firmware unless the jumpers are attached.

# Building a two-factor biometric system

With an independent processor on the CryptoCape, we can create some interesting applications. Since the ATmega cannot be flashed from the BeagleBone unless the physical jumpers are attached to the board, we can consider this to be a *trusted* processor. In this project, we'll implement a biometric authentication system with a fingerprint sensor and the CryptoCape. We'll use the ATmega to prevent access to the security ICs on the CryptoCape until you have authenticated yourself *to the ATmega* with your fingerprint.

A notable example of fingerprint biometrics in consumer devices is Apple's Touch ID technology on the iPhone 5s. The sensor used on the iPhone is much more sophisticated and expensive than the sensor we will use in this project. But by performing this project, you should appreciate the capabilities and challenges of using biometric technologies. On the Touch ID support page, Apple motivates the use of the technology with the argument that unlocking the phone with your fingerprint is more secure than having no passcode and easier than entering a code each time. In a later section, we'll discuss the weakness of biometric systems.

The major components needed for this project are listed in the following table. The SparkFun parts are listed as they were the ones used, but feel free to substitute equivalent components. The CryptoCape, which is only manufactured by SparkFun Electronics, is open source hardware and the board design files are licensed under a Creative Commons license, so you could also make your own CryptoCape if you wish. You will also need a basic soldering station and appropriate accessories.

| Component | SparkFun SKU |
| --- | --- |
| Fingerprint sensor | SEN-11792 |
| CryptoCape | DEV-12773 |
| JST jumper wire assembly | PRT-10359 |
| Female jumper wires | PRT-11710 |
| Male breakaway headers | PRT-00116 |
| 2-pin jumpers | PRT-09044 |

The following components are optional, but are nice to have:

| Component | SparkFun SKU |
| --- | --- |
| Heat shrink kit | PRT-09353 |
| Third hand | TOL-09317 |
| Heaterizer XL-3000 | TOL-10326 |

## The fingerprint sensor overview

The Fingerprint sensor is the GT-511C3 device from ADH Technology. This device contains an optical sensor for reading the fingerprints but also an ARM Cortex M3 processor for computing the image recognition. The interface to this device is via serial UART, using 3.3V level logic, and at a baud rate of 9600 bps. There are four wires to connect from the JST connector: transmit, receive, GND, and power.

The datasheet states that the false acceptance rate is less than .001 percent and the false rejection rate is less than .1 percent. In short, this is a very capable fingerprint sensor. The fingerprint data is analyzed and stored on this device. We will use an existing library to communicate with this hardware.

## Appreciating the limitations of fingerprint biometrics

Realize that this fingerprint sensor authentication mechanism is only as strong as your fingerprint. A common critic against using fingerprint sensors centers around the fact that it is difficult for you to change your fingerprint. Once your fingerprint is copied you can't revoke or change it as you can with a password. Using a fingerprint as a two-factor mechanism slightly reduces the risk of an authentication breach since a pin or password is still required. You can also mitigate the risk of a fake fingerprint attack on your sensor by stationing an armed guard to watch the sensor as Bruce Schneier, a security technologist, stated in a September 2013 opinion article in WIRED magazine. Such luxuries are often limited to deep-pocketed governments.

### Note

*Mythbusters*, a popular science and engineering program on the Discovery Channel, busted the myth that fingerprints could not be copied and showed how to defeat fingerprint sensors: [https://www.youtube.com/watch?v=3Hji3kp_i9k](https://www.youtube.com/watch?v=3Hji3kp_i9k). Also, days after the Apple iPhone 5s was released, the biometrics hacking team of the **Chaos Computer Club** (**CCC**) showed how to bypass the fingerprint sensor: [http://www.ccc.de/en/updates/2013/ccc-breaks-apple-touchid](http://www.ccc.de/en/updates/2013/ccc-breaks-apple-touchid).

Perhaps the greatest danger of biometric systems is the potential for a grave privacy breach. A database of fingerprints should be well protected since once the fingerprints are exposed they are no longer useful for any other biometric system, *ever*. In August 2014, Hold Security, an information security forensics company, reported to the *New York Times* that over 1.2 billion usernames and passwords were acquired by a Russian crime organization. While incredibly damaging, this breach would be irrecoverable if fingerprint biometrics were used. Hopefully, companies aren't storing fingerprints directly but representations of the fingerprint similar to a hash digest. However, if implemented poorly, the results can be as disastrous as storing the raw fingerprint.

With these warnings in mind, we will continue with using a fingerprint sensor for this project. You'll gain insight on how a fingerprint sensor works and how to it fits into an authentication system. When you are finished with this project, however, you should probably delete your fingerprint from the sensor's database.

## Preparing the CryptoCape

In order to make the CryptoCape more *hackable*, we need to populate the pads attached to the I/O signals with male headers. This will allow us to connect various external components. While you could directly solder to the pads, it's best if you solder a male 0.1" pin, which will allow you to easily connect a female terminated wire for your project, which can then be reused.

A fully populated CryptoCape will look like the following image. A strip of breakaway headers of 31 pins or more will be enough to populate each pad. Technically, you don't need to populate the EEPROM write protect pads unless you want to write to the EEPROM. However, if you overwrite the EEPROM cape information, the BBB might not load the correct drivers.

![Preparing the CryptoCape](img/00013.jpeg)

## Preparing the connections

Before attaching any connections, we physically need a method to attach the ends of the JST cable to the CryptoCape. The CryptoCape has 0.1" pads, which will fit 0.1" male headers and female-to-female jumper wires fit nicely to that connection. One end of the JST connector is simply four bare wires. If you solder each of these wires to a 4x1 0.1" male header, you can make a simple connector. Using a third hand, you can solder the JST wires to the headers and for a finishing touch, add heat-shrink around the wire and the male pin. Just remember to put your heat shrink on before you solder! The completed connector should look something like this:

![Preparing the connections](img/00014.jpeg)

## Connecting the Scanner to the CryptoCape

We'll attach the fingerprint scanner to the CryptoCape. The Arduino compatible library for this sensor was developed by Josh Hawley. This library is configured to use digital pin 4 for the receive pin on the ATmega and digital pin 5 as the transmit pin from the ATmega. These pins are appropriately labeled *D4* and *D5* on the CryptoCape, just under the ATmega. The remaining two pins are 3.3V power and ground which are also on the same row as the D4 and D5 pins. Attach the female ends of the jumper wires to these pins. If you are using the SparkFun JST connector, the pins from the fingerprint scanner are in the following order, starting with the black wire: D4, D5, GND, Power.

## Preparing the fingerprint sensor

The fingerprint sensor must be trained to recognize your fingerprint through a process called **enrollment**. There are several methods to enroll your fingerprint. The SparkFun website has instructions on how to use the ADH-Tech provided Windows-based software to program the sensor. However, you'll need an extra component, that is, the FTDI Basic Breakout board to convert USB from your computer to serial for the scanner. You could also use an Arduino directly, but you must also use logic level converters if you are using a 5V Arduino. Since the CryptoCape has an Arduino compatible processor, we will show you how to do this with your BBB and CryptoCape. The repository that contains the ATmega firmware can be cloned on your BBB with the following command:

```
git clone https://github.com/jbdatko/Fingerprint_Scanner-TTL.git

```

In this repository, the `FPS_Enroll.ino` file will enroll, or add, a fingerprint to the sensor's database. As previously mentioned, you'll need to compile this file out of band. Alternatively, you can use the pre-compiled version in the repository above.

Upload the enroll script:

```
sudo ./upload.sh FPS_Enroll.cpp.hex

```

You should see text scroll by that looks like this:

```
Reading | ################################################## | 100% 0.00s

avrdude: Device signature = 0x1e950f
avrdude: safemode: lfuse reads as 0
avrdude: safemode: hfuse reads as 0
avrdude: safemode: efuse reads as 0
avrdude: NOTE: FLASH memory has been specified, an erase cycle will be performed
 To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "FPS_Enroll.cpp.hex"
avrdude: input file FPS_Enroll.cpp.hex auto detected as Intel Hex
avrdude: writing flash (11458 bytes):

Writing | ################################################## | 100% 3.09s

avrdude: 11458 bytes of flash written
avrdude: verifying flash memory against FPS_Enroll.cpp.hex:
avrdude: load data flash data from input file FPS_Enroll.cpp.hex:
avrdude: input file FPS_Enroll.cpp.hex auto detected as Intel Hex
avrdude: input file FPS_Enroll.cpp.hex contains 11458 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 2.21s

avrdude: verifying ...
avrdude: 11458 bytes of flash verified

avrdude: safemode: lfuse reads as 0
avrdude: safemode: hfuse reads as 0
avrdude: safemode: efuse reads as 0
avrdude: safemode: Fuses OK

avrdude done.  Thank you.

```

If there is a problem, first check to see whether the jumpers are attached and then try again.

This script will output to the serial port at `9600` baud, so we need to change `/dev/ttyO4` to reflect this:

```
sudo stty -F /dev/ttyO4 9600

```

Finally, let's display the output of the serial port with:

```
cat /dev/ttyO4

```

The script is patiently waiting for you to enroll a finger. Place your finger on the reader and watch the output of the serial port and follow the instructions. A successful enrollment looks like this:

```
Remove finger
Press same finger again
Remove finger
Press same finger yet again
Remove finger
Enrolling Successful

```

If there is a problem, and the enrollment process can be a bit finicky, you'll need to reset the ATmega and re-attempt enrollment. In the same repository is a script, `reset.sh`, that simply toggles the ATmega reset line. If you have troubles, reset your ATmega and try again.

## Uploading the biometric detection sketch

With the sensor now trained to recognize your fingerprint, we'll upload a sketch that will lock out the CryptoCape until you present your fingerprint. Specifically, it will prevent access to the chips on the CryptoCape from the BBB. We'll accomplish this by *jamming* the SCL line on the I2C bus.

One weakness of I2C is that one misaligned device can disrupt the entire bus. On the CryptoCape, every IC is connected to the same I2C bus. We'll exploit this for this project. The ATmega will hold the SCL low until a successful fingerprint is received. This essentially jams the bus since the master, the BBB, can't generate the start condition.

The software running on the BBB will hang until the ATmega releases the lock. Typically, this is a very undesirable effect. For this project, however, it illustrates how two microprocessors can interact and even interfere with one another. In the following screenshot, you can see the effect of this jamming of Salee Logic Analyzer software. Once the SCL is released, the BBB is finally able to send a start condition.

![Uploading the biometric detection sketch](img/00015.jpeg)

In the `FPS_CryptoCape.ino` file, this is accomplished by setting digital output 2 as an output and then pulling the line low. When a fingerprint is recognized, the pin is configured as an input, which prevents the ATmega from pulling the line either high or low and allows normal I2C operation.

Add a jumper wire from *D2* on the ATmega breakout pads to the *SCL* pad near the TPM on the CryptoCape. This is that one extra wire that will allow the ATmega to lock out the BBB's access to the I2C bus. Once you add that wire, upload the `FPS_CryptoCape.cpp.hex`, which you can either compile yourself or use the pre-compiled version. The wires on your CryptoCape should look like the following image:

![Uploading the biometric detection sketch](img/00016.jpeg)

Upload the sketch as before and then listen to the serial port with the same `cat /dev/ttyO4` command. You will see the ATmega waiting for the sensor's fingerprint. Present your fingerprint to the sensor and it will then print a verification when complete:

```
Please press finger
Verified ID:0

```

You will also notice that the green LED on the CryptoCape will turn off. While the LED is on, the ATmega is locking out your BBB from accessing the CryptoCape.

## Security analysis of the biometric system

How secure is our biometric system? While it does prevent software on the BBB from using the CryptoCape until a valid fingerprint is accepted, the system is easily defeated by pulling (or cutting) the line from D2 to SCL. Without the electrical connection, the ATmega can't interfere with the I2C bus. However, depending on your installation, an attacker may have a difficult time physically accessing the hardware. The process of assessing vulnerabilities and mitigations to those vulnerabilities is known as **threat modeling**. In the previous chapter, the Tor design stated that it can't defend against a global passive adversary. In your implementation of our biometric system, maybe access to the jamming line is not a threat because you've placed your BBB in an adamantium box. There is no perfectly secure system so a threat model helps us understand the strengths, weaknesses, and assumptions of our system. We'll see more threat modeling in our final two chapters.

# Summary

This chapter provided a close look at BeagleBone capes and the CryptoCape. We introduced the idea of trusted computing and built a biometric authentication system. We've shown the normal use case for I2C devices and illustrated how one rogue device can corrupt the entire bus.

In the next chapter, we'll use the BBB to protect e-mail encryption and signing keys for **Pretty Good Privacy** (**PGP**) and its free software implementation: GPG.