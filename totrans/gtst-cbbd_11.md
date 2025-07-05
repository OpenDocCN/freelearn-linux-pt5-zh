# Appendix C. The FEX Configuration File

Many systems have a way of configuring themselves, be it software or, in this case, hardware. Allwinner-based hardware is no different; there are some bits and pieces that do require configuring, such as GPIO pins. A chip cannot configure itself; that is, it cannot parse a configuration file and configure itself. The actual configuration of the chip is done by the various drivers. The procedure is illustrated in the following sections.

# Initial boot up

The chip starts in a hardwired way, where certain components are preprogrammed to be active on specific pins. Because of this, the chip can boot from various boot media, as mentioned in [Chapter 3](ch03.html "Chapter 3. Installing an Operating System"), *Installing an Operating System*, and load the bootloader. The bootloader is also preconfigured to a certain hardware setup. Hence, as mentioned earlier, each board has its own bootloader.

Besides bringing up certain components, the bootloader has the following two important tasks:

*   Loading the kernel into memory and later executing it
*   Loading a configuration file into the memory for the kernel to use

However, the bootloader itself does not parse the configuration file.

### Tip

It has to be mentioned that when using a mainline kernel, the principle is the same: the bootloader still loads a configuration file named device tree binary, which the kernel uses for configuration.

The FEX configuration concept is interesting in itself, where one file is changed to configure an entire device. As mentioned earlier, the FEX file is loaded into the memory by the bootloader, and the bootloader only checks one specific location. While u-boot is more flexible and could be configured to allow reading the configuration file from any location or any filename, the bootloader that is preprogrammed into the onboard NAND flash storage will only check the first partition on the device, and that has to be FAT formatted. As such, this is a common convention that we will follow for the remainder of this chapter. Additionally, this file has to be called by a specific name, `script.bin`, and the duplicate backup by the name `script0.bin` file. When booting a kernel that supports the onboard `nand` flash, the device node where this file will be stored is `/dev/nanda`. Otherwise, normal device nodes will be used, most commonly the SD card at `/dev/mmcblk0p1`.

# Compiling and decompiling the FEX file

The `script.bin` file is, as its file extension suggests, a binary file. However, it is not possible to directly modify this file. The linux-sunxi community has created a set of tools to convert this binary file into a text file and vice versa. They can be found in their GitHub repository at [https://github.com/linux-sunxi/sunxi-tools](https://github.com/linux-sunxi/sunxi-tools).

After cloning this repository, the `make fex2bin` command is run to build the **fexc**, the fex decompiler. Compiling and running this tool is probably best on a system that has a comfortable text editor available.

Running fexc to decompile the binary file will look like this:

```
[packt@packt:~]$ fexc -I bin -O fex script.bin script.fex

```

There are two shorthands in the form of symlinks to fexc, namely `fex2bin` and `bin2fex`. Using these makes the `-I` and `-O` parameters unnecessary.

# Understanding the FEX file format

Using any text editor, a FEX file will show that it is divided in various sections, prepended by a header within brackets, `[]`. In the following example, the UART `0` component is explained:

```
[uart_para0]
uart_used = 1
uart_port = 0
uart_tx = port:PB22<2><1><default><default>
uart_rx = port:PB23<2><1><default><default>

```

Here, the component called `uart_para0`, the first serial port or UART, has four fields that a serial driver would read. Each field is set up as a key-value pair, where the key is on the left-hand side and the value is on the right-hand side of the equal sign. In this case, the key `uart_used` is set to the number `1` indicating that this definition should be parsed and activated.

Following that is the `uart_port` key, which is set to the number `0`, indicating that this configuration is about UART 0—the first UART port. Some of the settings are very straightforward key-value pairs, that is, a key on the left and a string or a number on the right. There is, however, a key-value pair that needs some special attention, and that is the pin configuration, where the value is a port definition. Every component might require certain pins to function.

A basic UART requires two pins: a transmit pin and a receive pin. The SoC can provide several UARTs on several pins. In the preceding example, two pins are defined, the transmit pin, `uart_tx`, and the receive pin, `uart_rx`. The pins on a SoC are very often grouped, usually by a related function.

In the case of the A10, it has nine groups called ports. Each port can consist of a varying amount of pins. Port B, for example, has 24 pins. The last two pins of the set are the UART transmit and UART receive pins. The numbering starts at 0, following from which, it should be no surprise that `PB22` and `PB23` in the preceding example are Port B: pin 22 and pin 23.

As mentioned before, each pin has multiple features, or as they say, many functions are multiplexed onto a pin. These multiplexes, or muxes, are enumerated, where MUX 0 always configures a pin as GPIO input, and MUX 1 always configures a pin as GPIO output. Depending on the port and pin, MUX 2 and beyond can have various meanings—in the case of `PB22` and `PB23` MUX 2 are the UART pins. The first parameter surrounded by angle brackets, `<>`, is thus defined as MUX 2.

## Pin configurations

[Chapter 8](ch08.html "Chapter 8. Blinking Lights and Sensing the World"), *Blinking Lights and Sensing the World*, talks about the purpose of a pull-up resistor and a pull-down resistor. Allwinner-based SoCs actually have pull-up or pull-down resistors attached internally to the pins. The second angle-bracket surrounded parameter, `<1>`, in this case, enables the internal pull-up, a `<0>` here disables the pull-up/pull-down feature, and a `<2>` enables the pull-down feature. The pull-down feature is, however, only valid when the port is configured as an input; the SoC does not support pull-down on outputs. One more valid option is to use the keyword `<default>`, which tells the driver to use a safe default value.

The third angle-bracket surrounded parameter defines the current strength that the pin should output. There are four valid values: `0`, `1`, `2`, and `3`, where `0` corresponds to 10 mA, `1` to 20 mA, `2` to 30 mA, and `3` to 40 mA. The default value can be used to let the driver choose a safe default.

The fourth position defines the initial output level of the pin, which can be either low, `<0>`, or high, `<1>`. Naturally, this is only valid when configuring the pin as output. Also here, default means that the driver uses a safe default value.

### Tip

While Port B: pins 22 and 23 were discussed and explained here, the rest of the pins and their muxes can be found on the linux-sunxi community wiki at [http://linux-sunxi.org/PIO](http://linux-sunxi.org/PIO).

# Further reading

The FEX file contains many other such components that can be set up, varying from configuring which pins are used for the SD card reader to what color format is used for the LCD. All options that have been discovered are also documented at the linux-sunxi wiki page, [http://linux-sunxi.org/Fex_Guide](http://linux-sunxi.org/Fex_Guide). However, since each driver is written to read a key-value pair, things easily and often do change depending on the progression of the driver and kernel as a whole. When in doubt, the kernel source code can always be checked.

# Installing the configured FEX file

Compiling the FEX file back into a bin file is nearly identical.

```
[packt@packt:~]$ fexc -I fex -O bin script.fex script.bin

```

Depending what boot medium is being used, `script.bin` has to be copied back so that the device can use these new, changed values. This can be the `/dev/nanda` partition on the onboard NAND flash or the first partition that is FAT formatted on a microSD card.

After having put the `script.bin` into place, a reboot is required for the system to read these changes.

# Summary

This appendix gave a small introduction to the FEX file and showed how to modify it. The next appendix will try to cover the most basic troubleshooting for the most common pitfalls.