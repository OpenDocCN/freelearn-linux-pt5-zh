# Appendix D. Troubleshooting the Common Pitfalls

When experimenting with new things, a lot can go wrong. In this appendix, a few of these common pitfalls are covered briefly. Anticipating everything that can go wrong is plainly impossible, but an overview of the most common problems is what this appendix is for.

Some general things that do tend to happen more often than one can imagine is the reading and entering of commands. A typo is easily made, or something is easily overlooked and read wrong. These things naturally happen and are all part of working with something exciting and new. So the first general tip is to always double-check your input. Despite the many eyes that went over all the pages in this book, there is always the possibility that a mistake has crawled into this book, so if something still goes wrong, even when following the book to the letter, check the errata.

# Stability issues

Quite often, users find the many communities surrounding these devices and complain about strange crashes, unstable systems or random reboots. While it is, of course, always possible that the actual device might be damaged, very often a flaky power supply is to blame, which either provides unstable power or is not powerful enough.

Testing the strength or stability of a power supply is very difficult without additional equipment. What tends to happen if the power supply cannot deliver enough current is that the voltage starts to drop. This can be measured with a standard multimeter.

If the voltage drops below 4.8 volt, things are likely to go wrong. A very noisy power supply is even harder to test and requires an oscilloscope. It is probably best to get a well-known and good power supply. Phone chargers, for example, come in different strengths. There are cheap phone power supplies that barely deliver 500 milliampere, which, when the device is heavily stressed, is not enough. Obviously, there are also decent chargers that come with high-end smartphones that can easily supply 2000 milliampere, but even here, in combination with the hard disk or solid-state disk, the power requirement can be too high. It is thus advised to disconnect as many devices as possible. No USB device and no SATA storage as they get their power from the board. No other components that might receive power directly from the board. Ideally, only a power connection and a serial connection should be made, and a multimeter or another power measuring device should be used to see that the voltage does not drop too low.

Should the board still remain unstable even after all that, where other people with the same board using the same bootloader and ideally the same root filesystem have no problem, then the board might be defective. It is probably best to ask for an exchange where the board was purchased. But please, always try to verify everything else, as a board exchange is no fun for all the parties involved.

# Boot failures when booting from SD cards

There is nothing more frustrating than spending hours on compiling and preparing an SD card to boot the board with, only to have it not work. While it is critical to have a serial console to see a debugging output during this stage, sometimes no output is generated at all due to an incorrect SD card or binary. The most important thing when experimenting with the bootloader is to always have a known working copy at hand. This is to make sure that modifications to u-boot are not the cause.

The nightlies from linux-sunxi are an alternative, as those should always work. See [http://dl.linux-sunxi.org/nightly/u-boot-sunxi/u-boot-sunxi/u-boot-sunxi-latest/](http://dl.linux-sunxi.org/nightly/u-boot-sunxi/u-boot-sunxi/u-boot-sunxi-latest/) for a list of available bootloaders.

Sometimes, the microSD card might simply not be compatible or might even be broken. A different microSD card should be used to make sure the microSD card is not the cause. It is not unheard of that the first few bytes on the card are broken beyond repair. This is not generally a problem if the card is used as a simple storage medium. Because the bootloader gets written and read from the start of the SD card, this can thus result in a system that is unable to boot.

Now, going with either one of the nightlies or a self-compiled bootloader, quite often, users tend to choose the wrong file. This is not a big surprise, as there are many files and various instructions on the Internet that, sometimes, become outdated and are no longer accurate. A common example is that many older guides suggest writing `u-boot.bin` or `sunxi-spl.bin` using various `seek` offsets. Using the correct files and parameters, this can still work, whereas using the wrong parameters or wrong settings will cause failure. The recommended way, as described in [Chapter 4](ch04.html "Chapter 4. Manually Installing an Alternative Operating System"), *Manually Installing an Alternative Operating System*, onto the board is to use `u-boot-sunxi-with-spl.bin` using a `blocksize (bs)` of `1024` and `seek` of `8`. Applying this to, for example, `sdb`, the following command should be used:

```
packt@packt:~# dd if=u-boot-sunxi-with-spl.bin of=/dev/sdb bs=1024 seek=8

```

This is because both the previous mentioned files are combined in one big file, so fewer things can go wrong. Finally, after writing the bootloader, the intention is to format the first partition to store the kernel. The partition number gets omitted and not `/dev/sdb1`, but `/dev/sdb` gets formatted. This is completely legal to do; having partitions on a storage medium is optional, though often useful. Formatting the entire disk, rather than the partition, however, has a side effect that the earlier written bootloader is now overwritten and gone. When the board boots, the bootloader is no more to be found and hence no output is generated.

# No display output via a connected monitor

Quite often, when a monitor is connected via HDMI or VGA, there is no display output. This can be, understandably, very frustrating. Even more so, it probably worked fine with the preinstalled Android was working fine, but now when booting Linux from the SD card, nothing happens anymore. There are of course many possible reasons for not getting an image on the monitor, such as the cable might not be connected properly, the monitor might be set to the wrong input, and so on. Even though all these things seem very obvious and common, they do happen, and there are a few cases that are not so easily checked.

The Allwinner SoC is actually not very smart when it comes to displaying outputs. It needs to be told what is connected and what to use as an output. For this purpose, monitors and TVs, which really are just fancy monitors, have a communication channel in the cable called **Display Data Channel** (**DDC**). The purpose of the DDC is for the monitor to tell whoever asks what its capabilities and resolutions are. And this is where things get ugly.

For VGA, it was once agreed upon that there were two colors for the plugs: black if the port does not care about DDC and ignores it, and blue if it does read the DDC information and can respond accordingly. Unfortunately, many creators of development boards simply use a blue connector, thinking that is the standard VGA connector, but do not implement DDC. Without DDC, the board has no idea what is connected and how to send signals to it. And because of the ignorant usage of blue and black VGA plugs, there is no simple rule to follow anymore, which says that if your connector is black, you have to set up your VGA port manually.

For HDMI, the problem is very similar. The Allwinner HDMI display controller was created with television in mind for the most part. Thus, there are some quirks when connecting it to an HDMI monitor or when using a DVI to HDMI adapter. Android might, for example, have a purple hue over the image when being connected to a DVI monitor, which is a quirk caused by the driver in combination with the design of the display controller; running a linux-sunxi kernel usually fixes this.

The first thing that can be checked and changed is the `uEnv.txt` file that resides on the bootloader partition, possibly `/dev/sdb1`, when we use a microSD card in a USB adapter. Here, there should be a setting called `disp.screen0_output_mode=EDID:1280x720p60` or something similar for the bottom `extraargs` parameter.

This means that for the output, first try to probe the monitor via EDID, which is an extended form of DDC, and if that fails, fall back to the 1280 by 720 progressive resolution running at a refresh rate of 60 Hz. Some monitors might not properly announce their EDID information, so the first thing to try is removing the EDID bit from `output_mode` and thus forcing the output to `1280x720p60`. Additionally, it is possible that after EDID fails, the fixed resolution supplied is simply something the monitor does not accept. For valid resolutions and settings, check the second column in the table used in the following section. It should be noted that when we use a VGA monitor, EDID must be removed, as the driver will ignore the entire `output_mode` in that case. Additionally, the desired video output type needs to be appended to `extraargs` as `disp.screen0_output_type=4` for VGA or `3` for HDMI.

Overriding the kernel parameters via the `uEnv.txt` file might not be enough and cause things to not work. The same settings can and should also be supplied via the FEX file. The driver should, in theory, check both the kernel command line and the FEX file, but one might overrule the other, and thus, in case of trouble, both possibilities should be covered.

To force a video output mode, the FEX file needs to modified. The section that controls the display controller in the FEX file is the `[disp_init]` section. All the parameters for the display controller on the linux-sunxi wiki, [http://linux-sunxi.org/Fex_Guide](http://linux-sunxi.org/Fex_Guide), there are a few things that need to be looked at. When getting things going for the first time, `disp_mode` is probably best set to `0` , indicating one frame buffer on `screen0`. Depending how the monitor is connected, `screen0_output_type` should be set to either `3` or `4`, where `3` is an HDMI display and `4` is a VGA display. The `screen1_output_type` object is best disabled by setting it to `0`. For `screen0_output_mode`, the following table should be used, and any setting applied to `screen1_output_mode` will be ignored:

| `output_mode` | Used for the TV/HDMI output | Used for the VGA output |
| --- | --- | --- |
| 0 | 480i | 1680 * 1050 |
| 1 | 576i | 1440 * 900 |
| 2 | 480p | 1360 * 768 |
| 3 | 576p | 1280 * 1024 |
| 4 | 720p50 | 1024 * 768 |
| 5 | 720p60 | 800 * 600 |
| 6 | 1080i50 | 640 * 480 |
| 7 | 1080i60 |   |
| 8 | 1080p24 |   |
| 9 | 1080p50 |   |
| 10 | 1080p60 | 1920 * 1080 |
| 11 | pal | 1280 * 720 |
| 14 | ntsc |   |

The other settings should not be significant for the display output. The display should be able to output data now when booting, for example, the Fedora installation image, as was used in [Chapter 3](ch03.html "Chapter 3. Installing an Operating System"), *Installing an Operating System*.

# Summary

This appendix went over the three most common issues, which most often happen when starting to work with the various development boards. The things covered in this appendix by no means cover every possibly scenario but should give you a decent strategy on tackling these early problems.