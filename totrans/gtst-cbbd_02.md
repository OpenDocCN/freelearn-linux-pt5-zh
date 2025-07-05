# Chapter 2. Getting Started with the Hardware

When you start to play with a new device, there are a few questions that might come to your mind. How do I know it is working properly? How do I get an output from or an input to the device, such as video on a monitor or key strokes from a keyboard? These are probably the first basic questions that are asked after a board is unpacked or started for the first time.

This chapter will cover the following topics:

*   Connecting a serial port to the development board
*   Booting up the preinstalled software

# Connecting a serial port

A serial port might seem like something outdated, but it is in fact still very common on certain devices. One of the main reasons is that it is very simple and reliable. It is very simple in both hardware and software implementations, and because of its simplicity, it is often very reliable and pretty much always works—which is why it has been on devices since the sixties until today. A serial connection is, however, rather slow, but for text-based input and output, it is perfectly adequate. However, why would one want a serial port to begin with? As Murphy can attest, things just go wrong, and with these development boards, the serial port is very often the only thing providing any output.

There are two ways to connect the serial port or UART to the device, either with a USB to UART adapter or with a real serial port and a level shifter to convert the voltage to the appropriate voltage.

Since most PCs actually lack a serial port, only the USB to serial approach will be discussed here. The first step is to connect the USB to UART adapter to your Cubieboard, which can be a challenge in itself as there are some USB to UART adapters with three wires, where others have four or even more. Furthermore, the colors of the cables might differ from product to product. Finally, there are 3.3-volt or 5-volt adapters. Check the user manual of the cable to find out which color corresponds to which signal and whether the voltage is 3.3 volts. In the case of the USB to serial adapter that is shipped with the Cubieboards, for example, the following rules apply:

*   Black is GND or ground and connects to the GND pin
*   Green is TX or transmit and connects to the RX or receive pin on the board
*   White is RX or receive and connects to the TX or transmit pin on the board
*   Red is VCC or power and should never be connected, or the device might get damaged

Refer to the following image to see the various connections:

![Connecting a serial port](img/1572OS_02_01.jpg)

A UART connection on Cubieboard1

After connecting the UART side of things, the other end can simply be plugged into a USB port. Depending on the operating system used, a driver might need to be installed; furthermore, a program to connect to this serial port is required, and this creates something referred to as a serial terminal. PuTTY is such a program, and it is available on most operating systems. It can be downloaded for the Windows platform at [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Other operating systems might have it available via a built-in software store, and it can be installed via this way. Other usable types of software to view the serial console are programs such as GNU Screen or minicom and can be used equally well.

Some parameters are needed to get the serial communication working; a baud rate of 115,200 bits per second is needed. Additionally, eight data bits, no parity, and one stop bit might be needed (which is often abbreviated as 8n1), but both in the aforementioned case of PuTTY or screen (which is the default) might be omitted. For screen, `screen /dev/ttyUSB0 115200` command line can be used, assuming `/dev/ttyUSB0` is the serial port that is being used.

### Tip

Finding the correct device name or number might be tricky; this not only varies between operating systems, but obtaining the correct number can also be tricky. Under Linux and OSX, for example, using `dmesg` after plugging the USB converter in or using autocomplete on `/dev/ttyUSB` might help. On Windows, the device manager can be used.

For PuTTY, the following screenshot portrays the required settings, assuming `COM5` is the serial port. Make sure to check **Serial**:

![Connecting a serial port](img/1572OS_02_02.jpg)

If the serial connection was established properly, applying power to the Cubieboard should now yield text on the serial console. An example of this text is shown in the following screenshot. Here, the bootloader that is usually installed on a microSD card is displayed. First, the SPL is loaded, which probes the memory and prints the current CPU configuration, followed by U-Boot, which prints the current configuration. Have a look at the following screenshot:

![Connecting a serial port](img/1572OS_02_03.jpg)

The preceding screenshot is just an example. This varies between the bootloaders used, the bootmedium, and the board used. It only illustrates what is possibly seen on the first boot.

# Booting up the preinstalled software

When a Cubieboard is first powered up, a few things happen. First, the SoC checks various devices to see whether it can boot from them. If available, the onboard NAND flash is very likely to be preprogrammed by the manufacturer. Booting up the Cubieboard using whatever is preinstalled does serve a purpose. It allows you to check whether the Cubieboard is functioning properly.

If the Cubieboard doesn't have a preinstalled operating system or a NAND flash, a specially prepared microSD card can be used. It should yield similar results because the microSD card is actually the first device that the SoC tries to boot. It can be very useful to have one around. The following chapters will give you an idea about how to prepare such an SD card. The preinstalled operating system will very likely require a monitor, keyboard, and mouse connected so that it can be interacted with. While most variants have the Android OS preinstalled, there have been cases where a command-line version of Linux was installed. With an Android preinstallation, various components can easily be tested. Use the following checklist to test the most obvious things:

*   Does the display work?
*   Does the mouse work?
*   Does the keyboard work?
*   Does the networking connection work?
*   Does the audio playback work?
*   Does the MMC card work?

When the display is being tested to see whether it works, it is quite possible that the image is configured to be used with other peripherals than the ones that were expected. For example, an HDMI monitor might be expected, when, in fact, a VGA monitor is connected. The same goes for the audio; it might be routed over to HDMI when a regular headphone is connected via the audio jack.

Going over the aforementioned checklist is harder with a command-line installation but not entirely impossible. The display and keyboard can be tested quite easily. Even the networking feature should be detected. With some basic Linux knowledge, all of the earlier-mentioned components can be easily testable.

### Tip

For networking to be set up almost automatically, a DHCP server on the network is recommended. Most modems/routers or wireless access points supply this functionality by default.

# Summary

Having learned how to connect and use a serial port, you should now be able to watch a Cubieboard boot and use the preinstalled software to check whether things are working normally.

In the next chapter, you will finally start to get some real work done that is, you will set up a full-blown desktop system.