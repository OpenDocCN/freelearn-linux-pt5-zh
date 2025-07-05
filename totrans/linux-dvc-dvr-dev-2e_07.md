# *Chapter 5*: Understanding and Leveraging the Device Tree

The device tree is an easy-to-read hardware description file, with a JSON-like formatting style. It is a simple tree structure where devices are represented by nodes and their properties. These properties can either be empty (that is, just the key to describe Boolean values) or key-value pairs, where the value can contain an arbitrary byte stream. This chapter is a simple introduction to device trees. Every kernel subsystem or framework has its own device tree binding, and we will talk about those specific bindings when we deal with the relevant topics.

The device tree originated from **Open Firmware** (**OF**), which is a standard endorsed by computer companies, and whose main purpose is to define interfaces for computer firmware systems. That said, you can find out more about device tree specification at [http://www.devicetree.org/](http://www.devicetree.org/). Therefore, this chapter will cover the basics of the device tree, including the following:

*   Understanding the basic concept of the device tree mechanism
*   Describing data types and their APIs
*   Representing and addressing devices
*   Handling resources

# Understanding the basic concept of the device tree mechanism

The support of the device tree is enabled in the kernel by setting the `CONFIG_OF` option to `Y`. To pull the device tree API from your driver, you must add the following headers:

```
#include <linux/of.h>
#include <linux/of_device.h>
```

The device tree supports a few data types and writing conventions that we can summarize with a sample node description:

```
/* This is a comment */
// This is another comment
node_label: nodename@reg{
   string-property = "a string";
   string-list = "red fish", "blue fish";
   one-int-property = <197>; /* One cell in the property */
   int-list-property = <0xbeef 123 0xabcd4>;
   mixed-list-property = "a string", <35>,[0x01 0x23 0x45];
   byte-array-property = [0x01 0x23 0x45 0x67];
   boolean-property;
};
```

In the preceding example, `int-list-property` is a property where each number (or cell) is a 32-bit integer (`uint32`), and there are three cells in this property. Here, `mixed-list-property` is, as its name suggests, a property with mixed element types.

The following are some definitions of the data types used in the device tree:

*   Text strings are represented with double quotes. You can use commas to create a list of the strings.
*   Cells are 32-bit unsigned integers delimited by angle brackets.
*   Boolean data is nothing more than an empty property. The true or false value depends on the property being there or not.

We have easily enumerated the types of data that can be found in the device tree. Before we start learning about the APIs that can be used to parse this data, first, let's understand how the device tree naming convention works.

## The device tree naming convention

Every node must have a name in the form of `<name>[@<address>]`, where `<name>` is a string that can be up to 31 characters in length, and `[@<address>]` is optional, depending on whether the node represents an addressable device or not. That said, `<address>` should be the primary address used to access the device. For example, for a memory-mapped device, it must correspond to the starting address of its memory region, the bus device address for an I2C device, and the chip-select index (relative to the controller) for an SPI device node.

The following presents some examples of device naming:

```
i2c@021a0000 {
    compatible = "fsl,imx6q-i2c", "fsl,imx21-i2c";
    reg = <0x021a0000 0x4000>;
    [...]
    expander@20 {
        compatible = "microchip,mcp23017";
        reg = <20>;
        [...]       
    };
};
```

In the preceding device tree excerpt, the I2C controller is a memory-mapped device. Therefore, the address part of the node name corresponds to the beginning of its memory region, relative to the **System on Chip** (**SoC**) memory map. However, the expander is an I2C device. Thus, the address part of its node name corresponds to its I2C address.

## An introduction to the concept of aliases, labels, phandles, and paths

Aliases, labels, phandles, and paths are keywords that you need to be familiar with when dealing with the device tree. It is likely that you will face at least one, if not all, of these terms as and when you deal with device drivers. To describe these terms, let's take the following device tree excerpt as an example:

```
aliases {
    ethernet0 = &fec;
    gpio0 = &gpio1;
    [...];
};
bus@2000000 { /* AIPS1 */
    gpio1: gpio@209c000 {
        compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
        reg = <0x0209c000 0x4000>;
        interrupts = <0 66 IRQ_TYPE_LEVEL_HIGH>,
                    <0 67 IRQ_TYPE_LEVEL_HIGH>;
        gpio-controller;
        #gpio-cells = <2>;
        interrupt-controller;
        #interrupt-cells = <2>;
    };
    [...];
};
bus@2100000 { /* AIPS2 */
    [...]
    i2c1: i2c@21a0000 {
        compatible = "fsl,imx6q-i2c", "fsl,imx21-i2c";
        reg = <0x021a0000 0x4000>;
        interrupts = <0 36 IRQ_TYPE_LEVEL_HIGH>;
        clocks = <&clks IMX6QDL_CLK_I2C1>;
    };
};
&i2c1 {
    eeprom-24c512@55 {
        compatible = "atmel,24c512";
        reg = <0x55>;
    };
    accelerometer@1d {
        compatible = "adi,adxl345";
        reg = <0x1d>;
        interrupt-parent = <&gpio1>;
        interrupts = <24 IRQ_TYPE_LEVEL_HIGH>,
        <25 IRQ_TYPE_LEVEL_HIGH>;
        [...]
    };
[...]
};
```

In the device tree, there are two ways in which a node can be referenced: by a path or by a `phandle` property and, sometimes, duplicated in a `linux,phandle` property for historical reasons.

However, the device tree source format allows labels to be attached to any node or property value. Given that a label must be unique all over the device tree source for a given board, it became obvious that it could be used to identify a node as well, and a decision was made to do so. Thus, logic has been added to the `&`) in a cell property, it is replaced with the phandle of the node to which this label is attached. Moreover, using the same logic, whenever a label is prefixed with an ampersand outside of a cell (a simple value assignment), it is replaced by the full path of the node to which the label is attached. This way, the phandle and path references can be automatically generated by referencing a label instead of explicitly specifying a phandle value or the full path to a node.

Note

Labels are only used in the device tree source format and are not encoded within the `dtc` tool will remove that label from the node and add a `phandle` property to that node, generating and assigning a unique 32-bit value. The `dtc` tool will then use this phandle in every cell where the node has been referenced by the label (prefixed with an ampersand).

Back to our preceding excerpt, the `gpio@0209c000` node is labeled `gpio1`, and this label is also used as a reference. This will instruct the DTC to generate a phandle for this node. Therefore, in the `accelerometer@1d` node, inside the `interrupt-parent` property, the cell value (`&gpio1`) will be replaced by the phandle of the node to which `gpio1` is attached (the assignment inside a cell). In the same way, inside the `aliases` node, `&gpio1` will be replaced with the full path of the node to which `gpio1` is attached (the assignment outside of a cell).

After compiling and decompiling our original device tree excerpt, we obtain the following, where labels no longer exist and label references have been replaced either by a phandle or by the full node path:

```
aliases {
    gpio0 = "/soc/aips-bus@2000000/gpio@209c000";
    ethernet0 = "/soc/aips-bus@2100000/ethernet@2188000";
    [...]
};
aips-bus@2000000 {
    gpio@209c000 {
        compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
        gpio-controller;
        #interrupt-cells = <0x2>;
        interrupts = <0x0 0x42 0x4 0x0 0x43 0x4>;
        phandle = <0x40>;
        reg = <0x209c000 0x4000>;
        #gpio-cells = <0x2>;
        interrupt-controller;
    };
};
aips-bus@2100000 {
    i2c@21a8000 {
        compatible = "fsl,imx6q-i2c", "fsl,imx21-i2c";
        clocks = <0x4 0x7f>;
        interrupts = <0x0 0x26 0x4>;
        reg = <0x21a8000 0x4000>;
        eeprom-24c512@55 {
            compatible = "atmel,24c512";
            reg = <0x55>;
        };
        accelerometer@1d {
            compatible = "adi,adxl345";
            interrupt-parent = <0x40>;
            interrupts = <0x18 0x4 0x19 0x4>;
            reg = <0x1d>;
        };
    };
};
```

In the preceding snippet, the accelerometer node has its `interrupt-parent` property cell assigned the value of `0x40`. Looking at the `gpio@209c000` node, we can see that this value corresponds to the value of its `phandle` property, which has been generated at compile time by the DTC. It's the same for the `aliases` node, where the node references have been replaced by their full paths.

This leads us to the definition of aliases; aliases are simply nodes that are referenced via their absolute paths for a quick lookup. The `aliases` node can be seen as a fast lookup table. Unlike labels, aliases do appear in the output device tree, although paths are generated by referencing labels. With an alias, a handle to the node it is referring to is obtained by simply searching for it in the aliases section rather than searching for it in the entire device tree as it is done while looking up by phandle. Aliases can be seen as a shortcut or similar to the aliases we set in our Unix shell to refer to a complete/long/repetitive path/command.

The Linux kernel dereferences the aliases rather than using them directly in the device tree source. When using `of_find_node_by_path()` or `of_find_node_opts_by_path()` to find a node given its path, if the supplied path does not start with `/`, then the first element of the path must be a property name in the `/aliases` node. That element is replaced with the full path from the alias.

Note

Labeling a node is only useful if the node is intended to be referenced from the property of another node. You can consider a label as a pointer to a node, either by the path or by the reference.

## Understanding overwriting nodes and properties

After taking a closer look at the result of the decompiled excerpt, you should notice one more thing: in the original sources, the `i2c@21a0000` node has been referenced through its label as an external node (`&i2c1 { [...] }`) with some content inside. However, oddly, after decompiling, the final content of the `i2c@21a0000` node has been merged with the content of the external reference, and the external reference node no longer exists.

This is the third usage of labels: allowing you to overwrite nodes and properties. In the external reference, any new content (such as nodes or properties) will be appended to the original node content at compile time. However, in the case of duplication (either nodes or properties), the content of the external reference will take precedence over the original content.

Consider the following example:

```
bus@2100000 { /* AIPS2 */
    [...]
    i2c1: i2c@21a0000 {
        [...]
        status = "disabled";
    };
};
```

Let's reference the `i2c@21a0000` node through its `i2c1` label, as follows:

```
&i2c1 {
    [...]
    status = "okay";
};
```

We will see that the result from the compilation will be the `status` property having the value of `"okay"`. This is because the content of the external reference will have taken precedence over the original content.

To summarize, latter definitions always overwrite earlier definitions. For entire nodes to be overwritten, you simply have to redefine them as you would do so for properties.

## Device tree sources and compilers

The `.dts` extension, while the binary forms have either a `.dtb` or `.dtbo` extension. .`dtbo` is a particular extension that is used for compiled device tree overlays (`.dtsi` text files (where the `i` at the end means "include"). These host SoC-level definitions and are intended to be included in `.dts` files, hosting the board-level definitions.

The syntax of the device tree allows you to use `/include/` or `#include` to include other files. This inclusion mechanism makes it possible to use `#define`, but above all, it allows you to factorize the common aspects of several platforms in the shared files.

This factorization allows you to split the source files into tree levels, with the most common being the SoC level, which is provided by the SoC vendor (for example, NXP), the **System on Module** (**SoM**) level (for example, Engicam), and, finally, the carrier board or customer board level.

Therefore, all electronic boards using the same SoC do not redefine all of the peripherals of the SoC from scratch: this description is factored into a common file. By convention, such *common* files use the `.dtsi` extension, while final device trees use the `.dts` extension.

In the Linux kernel sources, ARM device tree source files can be found under the `arch/arm/boot/dts/` and `arch/arm64/boot/dts/<vendor>/` directories for the 32-bit and 64-bit ARM SoCs/boards, respectively. In either directory, there is a `Makefile` file that lists the device tree source files that can be compiled.

The utility used to compile the DTS files into DTB files is called DTC. The DTC sources exist in two places:

*   **As a standalone upstream project**: The DTC upstream project is maintained in [https://git.kernel.org/cgit/utils/dtc/dtc.git](https://git.kernel.org/cgit/utils/dtc/dtc.git). It is pulled into the Linux kernel source tree on a regular basis.
*   `scripts/dtc/`. New versions are pulled from the upstream project on a regular basis. The DTC is built by the Linux kernel build process as a dependency when needed (for example, before compiling the device tree). You can use the `make scripts` command if you wish to build it explicitly in the Linux kernel source tree.

From the main directory in the kernel sources, you can either compile a specific device tree or all device trees for a specific SoC. In either case, the appropriate config option to enable this or these device tree files must be enabled. For a single device tree compilation, the make target is the name of the `.dts` file with `.dts` changed to `.dtb`. For all of the enabled device trees to be compiled, the make target that you should use is `dtbs`. In both cases, you should make sure that the config option that enables the `dtb` has been set.

Consider the following excerpt from `arch/arm/boot/dts/Makefile`:

```
dtb-$(CONFIG_SOC_IMX6Q) += \
    imx6dl-alti6p.dtb \
    imx6dl-aristainetos_7.dtb \
[...]
    imx6q-hummingboard.dtb \
    imx6q-hummingboard2.dtb \
    imx6q-hummingboard2-emmc-som-v15.dtb \
    imx6q-hummingboard2-som-v15.dtb \
    imx6q-icore.dtb \
[...]
```

By enabling `CONFIG_SOC_IMX6Q`, you can either compile all of the device tree files listed in there or target a specific device tree. By running `make dtbs`, the kernel DTC will compile all the device tree files listed in the enabled config options.

First, let's make sure the appropriate config option has been set:

```
$ grep CONFIG_SOC_IMX6Q .config
CONFIG_SOC_IMX6Q =y
```

Then, let's compile all of the device tree files:

```
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make dtbs
```

Assuming our platform is an ARM64 platform, we would use the following:

```
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make dtbs
```

Once again, the right kernel config options must be set.

You could target a particular device tree build (let's say `imx6q-hummingboard2.dts`), using a command such as the following:

```
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make imx6q-hummingboard2.dtb
```

It must be noted that, given a compiled device tree (`.dtb`) file, you can do the reverse operation and extract the source (`.dts`) file:

```
$ dtc -I dtb -O dts arch/arm/boot/dts/imx6q-hummingboard2.dtb > path/to/my_devicetree.dts
```

For debug purposes, it might be useful to expose to user space the current device tree of a running system, that is, the so-called live device tree. To do so, the kernel `CONFIG_PROC_DEVICETREE` config option must be enabled. Then, you can explore and walk through the device tree in the `/proc/device-tree` directory.

If installed on the running system, the DTC can be used to convert the filesystem tree into a more readable form using the following command:

```
# dtc -I fs -O dts /sys/firmware/devicetree/base > MySBC.dts
```

After this command returns, the `MySBC.dts` file will contain the sources corresponding to the current device tree.

### The device tree overlay

Device tree overlaying is a mechanism that allows you to patch a live device tree, that is, modify the current device tree at runtime. It allows you to update the current device tree at runtime by updating existing nodes and properties or creating new ones. However, it does not allow you to delete a node or a property.

A device tree overlay has the following format:

```
/dts-v1/;
/plugin/; /* allow undefined label references and record them */
/{
    fragment@0 { /* first child node */
        target=<phandle>; /* phandle of the target node to extend */
    or
        target-path="/path"; /* full path of the node to extend */
        __overlay__ {
            property-a;  /* add property-a to the target */
            property-b = <0x80>; /* update property-b value */
            node-a { /* add to an existing, or create a node-a */
                    ...
            };
        };
    };
    fragment@1 {
         /* second fragment overlay ... */
    };
    /* more fragments follow */
}
```

From the preceding excerpt, we can note that each node from the base device tree that needs to be overlayed must be enclosed inside a `fragment` node in the overlay device tree.

Then, each fragment has two elements:

*   One of these two properties could be as follows:
    *   `target-path`: This specifies the absolute path to the node that the fragment will modify.
    *   `target`: This specifies the relative path to the node alias (prefixed with an ampersand symbol) that the fragment will modify.
*   A node, named `__overlay__`, that contains the changes that should be applied to the referred node. Such changes can be new nodes (which are added), new properties (which are added), or existing properties (which are overridden with the new value). There is no removal operation possible since a property or a node cannot be removed.

Now that we are comfortable with the basics of device tree overlaying, we can learn how they are compiled and turned into a binary blob that can be loaded on demand.

#### Building device tree overlays

Unless a device tree overlay adds new nodes under the root node only (in which case, it could specify `/` in the `target-path` property in the fragment), it would be much easier to specify the target node via its phandle (`<&label_name>`) as it would save us from manually computing the node's full path (especially if it is nested).

The thing is, there is no direct correlation or link between the base device tree and the overlay. They are each built, standalone, on their sides. Therefore, referencing a remote node (that is, a node in the base device tree) from the device tree overlay will raise errors, and the build of the overlay will fail because of undefined references or labels. It would be like building a dynamically linked application without room for symbol resolution.

To address this issue, `-@` command-line flag support has been added to the DTC. This flag must be specified for both the base device tree and all of the overlays to be compiled. It will instruct the DTC to generate extra nodes in the root (such as `__symbols__`, `__fixups__`, and `__local_fixups__`) that contain resolution data for the translation of phandle names. These extra nodes are spread as follows:

*   When the `-@` option is added to build an overlay, it recognizes the `/plugin/;` line that marks a device tree fragment/object. That line controls the generation of `__fixups__` and `__local_fixups__` nodes.
*   When the `-@` option is added to build the base device tree, `/plugin/;` is not present, so the source is recognized as being the base device tree, which causes the generation of `__symbols__` nodes only.

These extra nodes add room for symbol resolution.

Note

Support for the `-@` option can only be found in `dtc` version 1.4.4 or later. Only Linux kernel versions v4.14 or higher includes a built-in version of `dtc` that meets this requirement. This option is not needed if only `target-path` properties (that is, non-phandle-based) are used in the device tree overlay.

Building a binary device tree overlay follows the same process as building a traditional binary device tree. For example, let's consider the following base device tree. Let's call it `base.dts`:

```
/dts-v1/;
/ {
        foo: foonode {
                foo-bool-property;
                foo-int-property = <0x80>;
                status = "disabled";
        };
};
```

Then, let's build this base device tree with the following command:

```
dtc -@ -I dts -O dtb -o base.dtb base.dts
```

In the next step, let's consider the following device tree overlay. Let's call it `foo-verlay.dts`:

```
/dts-v1/;
/plugin/;
/ {
        fragment@1 {
                target = <&foo>;
                __overlay__ {
                        overlay-1-property;
                        status = "okay";
                        bar: barnode {
                                bar-property;
                        };
                };
        };
};
```

In the preceding device tree overlay, the `status` property of the `foo` node in the base device tree has been modified from `disabled` to `okay`, which will activate this node. Following this, the `overlay-1-property` Boolean property has been added, and finally, a `bar` sub-node has been added with a single Boolean property. This device tree overlay can be compiled with the following command:

```
dtc -@ -I dts -O dtb -o foo-overlay.dtbo foo-overlay.dts
```

As you can see, the `-@` flag has been added on both sides, enabling room for symbol resolution.

Note

In the Yocto build system, you could add this flag to the machine configuration or, during development, to the `local.conf` file, as follows: `DEVICETREE_FLAGS += "-@"`.

For the Linux kernel to build the device tree overlay, you should add it to the `Makefile` device tree of your SoC architecture, for example, `arch/arm64/boot/dts/freescale/Makefile` or `arm/arm/boot/dts/Makefile`, along with the `dtbo` extension, as follows:

```
dtb-y += foo-overlay.dtbo
```

Now that we can manage to build our own device tree overlays, let's consider the logical next step, which consists of loading these overlays into the system.

#### Loading device tree overlays via configfs

This section is named such that it mentions the way the device tree overlay is going to be loaded (configfs) since there is not only one way to load device tree overlays. In this section, we will focus on doing this on a running system whose kernel has already booted and the root filesystem is already mounted.

In order to do this, your kernel must have been compiled with `CONFIG_OF_OVERLAY` and `CONFIG_CONFIGFS` for the following steps to work. The following is a check, assuming the kernel config is available in the target:

```
~# zcat /proc/config.gz | grep CONFIGFS
CONFIG_CONFIGFS_FS=y
~# zcat /proc/config.gz | grep OF_OVERLAY
CONFIG_OF_OVERLAY=y
~# 
```

Now it's time to insert the DTBs into a running kernel using `configfs`. First, we mount the `configfs` filesystem if it has not already been mounted on your system:

```
# mount -t configfs none /sys/kernel/config
```

When `configfs` has been mounted properly, the directory should be populated with base subdirectories (`device-tree/overlays`), which, according to our mount path, will result in `/sys/kernel/config/device-tree/overlays`, as demonstrated in the following:

```
# mkdir -p /sys/kernel/config/device-tree/overlays/
```

Then, each overlay entry must be added from within the `overlays` directory. It has to be noted that overlay entries are created and manipulated using a standard filesystem I/O.

To load an overlay, a directory corresponding to this overlay must be created under the `overlays` directory. For our example, let's use the name `foo` :

```
# mkdir /sys/kernel/config/device-tree/overlays/foo
```

Next, to effectively load the overlay, you can `echo` the overlay firmware file path to the `path` property file, as follows:

```
# echo /path/to/foo-overlay.dtbo > /sys/kernel/config/device-tree/overlays/foo/path
```

Alternatively, you can `cat` the contents of the overlay to the `dtbo` file:

```
# cat foo.dtbo > /sys/kernel/config/device-tree/overlays/foo/dtbo
```

After that, the overlay file will be applied, and devices will be created/destroyed as required.

To remove the overlay and undo its changes, you should simply `rmdir` the corresponding overlay directory. In our example, it should be as follows:

```
# rmdir /sys/kernel/config/device-tree/overlays/foo
```

Although you have loaded the device tree overlay dynamically, it won't be sufficient; the device driver for the added device node needs to be loaded for the device to work unless this driver is built-in and enabled (that is, selected with `y` during `make menuconfig`).

At this stage, we are done with our device tree compilation-related stuff. Now, we can learn how to write our own device trees, starting with device addressing and representation.

# Representing and addressing devices

In the device tree, a node is the representational unit of a device. In other words, a device is represented by at least one node. Following this, device nodes can either be populated with other nodes (therefore, creating a parent-child relationship) or with properties (which would describe the device corresponding to the node they populate).

While each device can operate standalone, there are situations where a device might want to be accessed by its parent or where a parent might want to access one of its children. For example, such situations occur when a bus controller (the parent node) wants to access one or more of the devices (declared as a sub-node) sitting on its bus. Typical examples include I2C controllers and I2C devices, SPI controllers and SPI devices, CPUs and memory-mapped devices, and more. Thus, the concept of device addressing has emerged. Device addressing has been introduced with a `reg` property, which is used in each addressable device but whose meaning or interpretation depends on the parent (most of the time, they are bus controllers). The meaning and interpretation of `reg` in a child device depends on the `#address-cells` and `#size-cells` properties of its parent. The `#` (sharp) character that prefixes `size-cells` and `address-cells` can be considered to mean "length of."

Each addressable device gets a `reg` property that is a list of tuples in the form of `reg = <address0 size0 [address1 size1] [address2 size2] ... >`, where each tuple represents an address range used by the device. `#size-cells` indicates how many 32-bit cells are used to represent the size, which might be 0 if the size is not relevant. On the other hand, `#address-cells` indicates how many 32-bit cells are used to represent the address. In other words, the address element of each tuple is interpreted according to `#address-cells`; this uses the same size as the size element, which is interpreted according to `#size-cells`.

To sum up, addressable devices inherit from the `#size-cell` and `#address-cell` properties of their parent, which is the node that represents the bus controller most of the time. The presence of `#size-cell` and `#address-cell` in a given device does not affect the device itself but its children if they are addressable.

Now that we have seen how addressing works in a general manner, let's address specific addressing for non-discoverable devices, starting with SPI and I2C.

## Handling SPI and I2C device addressing

SPI and I2C devices both belong to non-memory-mapped devices because their addresses are not accessible to the CPU. Instead, the parent device's driver (the bus controller driver) will perform indirect access on behalf of the CPU. Each I2C/SPI device node is always represented as a sub-node of the I2C/SPI controller node that the device sits on. For a non-memory-mapped device, the `#size-cells` property is 0, and the size element in the addressing tuple is empty. This means that the `reg` property for this kind of device is always one cell. The following is an example:

```
&i2c3 {
    [...]
    status = "okay";
    temperature-sensor@49 {
        compatible = "national,lm73";
        reg = <0x49>;
    };
    pcf8523: rtc@68 {
        compatible = "nxp,pcf8523";
        reg = <0x68>;
    };
};
&ecspi1 {
    fsl,spi-num-chipselects = <3>;
    cs-gpios = <&gpio5 17 0>, <&gpio5 17 0>, <&gpio5 17 0>;
    status = "okay";
    [...]
    ad7606r8_0: ad7606r8@1 {
        compatible = "ad7606-8";
        reg = <1>;
        spi-max-frequency = <1000000>;
        interrupt-parent = <&gpio4>;
        interrupts = <30 0x0>;
        convst-gpio = <&gpio6 18 0>;
    };
};
```

If you look at the SoC-level file in `arch/arm/boot/dts/imx6qdl.dtsi`, you will notice that `#size-cells` and `#address-cells` are set to `0`, for the former, and `1`, for the latter, respectively, in both the I2C and SPI controller nodes (labeled `i2c3` and `ecspi1`). This helps you to understand their `reg` property, which is only one cell for the address value and none for the size value.

The I2C device's `reg` property is used to specify the device's address on the bus. For SPI devices, `reg` represents the index of the chip-select line assigned to the device among the list of chip selects the controller node has. For example, for the `ad7606r8` ADC, the chip-select index is 1, which corresponds to `<&gpio5 17 0>` in `cs-gpios.` This is the list of chip selects of the controller node. The binding of other controllers might differ, and you should refer to their documentation in`Documentation/devicetree/bindings/spi`.

## Memory-mapped devices and device addressing

This section addresses simple memory-mapped devices where the memory region is accessible by the CPU. With such device nodes, the `reg` property still defines the device's address, and the `reg = <address0 size0 [address1 size1] [address2 size2] ... >` pattern takes place. Each region is represented with a tuple of cells, where the first cell is the base address of the memory region, and the second cell is the size of the region. It could be translated into the following pattern: `reg = <base0 length0 [base1 length1] [address2 length2] ... >`. Here, each tuple represents an address range used by the device.

Let's consider the following example:

```
soc {
    #address-cells = <1>;
    #size-cells = <1>;
    compatible = "simple-bus";
    aips-bus@02000000 { /* AIPS1 */
        compatible = "fsl,aips-bus", "simple-bus";
        #address-cells = <1>;
        #size-cells = <1>;
        reg = <0x02000000 0x100000>;
        [...];
        spba-bus@02000000 {
            compatible = "fsl,spba-bus", "simple-bus";
            #address-cells = <1>;
            #size-cells = <1>;
             reg = <0x02000000 0x40000>;
             [...]

            ecspi1: ecspi@02008000 {
                #address-cells = <1>;
                #size-cells = <0>;
                compatible = "fsl,imx6q-ecspi", "fsl,imx51-ecspi";
                reg = <0x02008000 0x4000>;
                [...]
            };

            i2c1: i2c@021a0000 {
                #address-cells = <1>;
                #size-cells = <0>;
                compatible = "fsl,imx6q-i2c", "fsl,imx21-i2c";
                reg = <0x021a0000 0x4000>;
              [...]
            };
        };
    };
};
```

In the preceding excerpt, device nodes that have `simple-bus` in their `compatible` properties are SoC internal memory-mapped bus controllers connecting IP cores (such as I2C, SPI, USB, Ethernet, and other internal SoC IPs) to the CPU. Their sub-nodes, which are either IP core or other internal buses, inherit from the `#address-cells` and `#size-cells` properties. We can see that the `i2c@021a0000` I2C controller (labeled `i2c1`) is connected to the `spba-bus@02000000` bus. All of them will appear as platform devices on the system at runtime. On the other hand, this I2C controller changes its addressing scheme by defining its own `#address-cells` and `#size-cells` properties, which the I2C devices connected to it will inherit. This is the same for the SPI controller.

To summarize, in the real world, you should not interpret a `reg` property in a node without knowing the `#size-cells` and `#address-cells` properties of its parent. Memory-mapped devices must have the `size` field of their `reg` property set with the size of the memory regions of the devices, but also the `address` field, which must be defined such that it corresponds to the beginning of the device memory regions in the SoC memory map, as shown in the SoC datasheet.

Note

The simple-bus-compatible string also indicates that the bus has no special driver, that there's no way to dynamically probe the bus, and that direct child nodes (exclusively, level-1 children) will be registered as platform devices. Sometimes, this is used in board-level device trees to instantiate GPIO-based fixed regulators.

# Handling resources

The main purpose of a device driver is to provide a set of driving functions for a given device and expose its capabilities to users. Here, the objective is to gather the device's configuration parameters, especially resources (such as the memory region, interrupt line, DMA channel, and more) that will help the driver to perform its job.

## The struct resource

Once probed, device resources assigned to the device (either in the device or the board/machine file) are gathered and allocated either by `of_platform` or by the `platform` cores using `struct resource`, as follows:

```
struct resource {
    resource_size_t start;
    resource_size_t end;
    const char *name;
    unsigned long flags;
[...]
};
```

The following lists the meanings of the elements in the data structure:

*   `start`: Depending on the resource flag, this can be the starting address of a memory region, an IRQ line number, a DMA channel number, or a register offset.
*   `end`: This is the end of the memory region or the end of the register offset. In the case of an IRQ or DMA channel, most of the time, the `start` and `end` values are the same.
*   `name`: This is the name of the resource if any. We discuss it in more detail in the next section.
*   `flags`: This indicates the type of resource. Possible values include the following:
    *   `IORESOURCE_IO`: This indicates the PCI/ISA I/O port region. `start` is the first port of the region, and `end` is the last one.
    *   `IORESOURCE_MEM`: This is used for the I/O memory regions. `start` indicates the starting address of the region, and `end` indicates where it ends.
    *   `IORESOURCE_REG`: This refers to the register offsets. It is mostly used with MFD devices. `start` indicates the offset relative to a parent device register, and `end` indicates where the register section ends.
    *   `IORESOURCE_IRQ`: The resource is an IRQ line number. In this case, either both `start` and `end` have the same value or `end` is irrelevant.
    *   `IORESOURCE_DMA`: This indicates that the resource is a DMA channel number. You should consider `end` in the same way as an IRQ. However, what happens when you have more than one cell for the DMA channel identifier or when you have multiple DMA controllers is not very well defined. `IORESOURCE_DMA` is not scalable for multiple controller systems.

There is one instance of this data structure allocated per resource. That means, for a device that is assigned two memory regions and one IRQ line, there will be three data structures allocated. Moreover, resources of the same type will be allocated and indexed (starting from 0) in the order they are declared in the device tree (or the board file). This means that the first memory region assigned will have index 0, and so on.

To get the appropriate resource, we will use a generic API, `platform_get_resource()`, given the resource type and its index in this type. This function is defined as follows:

```
struct resource *platform_get_resource(
                      struct platform_device *dev,
                      unsigned int type, unsigned int num)
```

In the preceding prototype, `dev` is the platform device that we write the driver for, `type` is the resource type, and `num` is the index of this resource in the same type. On success, the function returns a valid pointer to `struct resource`, or `NULL` otherwise.

When there is more than one resource in the same type, using indexes could be misleading. You might decide to rely on the resource name as an alternative, and the named variant of `platform_get_resource()` will be introduced, which is `platform_get_resource_byname()`. This function is given a resource flag (or type) and its name and returns the appropriate resource, whatever the order they are declared in.

It is defined as follows:

```
struct resource *platform_get_resource_byname(
                             struct platform_device *dev,
                             unsigned int type,
                           const char *name)
```

To understand how to use this function, first, let's introduce the concept of named resources. We will discuss this next.

### The concept of named resources

When the driver expects a list of resources of a certain type (let's say two IRQ lines, where the first one is for Tx and the second for Rx), there is no guarantee of the way the list will be ordered, and the driver must make no assumption. What happens if the driver logic is hardcoded so that it expects Rx IRQ first, but the device tree has been populated with Tx first? To avoid such mismatches, the concept of named resources (such as clocks, IRQs, DMA channels, and memory regions) has been introduced. This consists of defining the resource list and naming them. This is so that, whatever their indexes are, a given name will always match the resource. The concept of a named resource also makes it easy to read and understand the device tree resource assignment.

The corresponding properties to name the resources are as follows:

*   `reg-names`: This is the list of names for memory regions in the `reg` property.
*   `interrupt-names`: This gives a name to each interrupt line in the `interrupts` property.
*   `dma-names`: This is for the `dma` property.
*   `clock-names`: This is to name the clocks inside the `clocks` property. Note that clocks won't be discussed in this book.

To demonstrate the concept, let's consider the following fake device node entry:

```
fake_device {
    compatible = "packt,fake-device";
    reg = <0x4a064000 0x800>,
          <0x4a064800 0x200>,
          <0x4a064c00 0x200>;
    reg-names = "ohci", "ehci", "config";
    interrupts = <0 66 IRQ_TYPE_LEVEL_HIGH>,
                 <0 67 IRQ_TYPE_LEVEL_HIGH>;
    interrupt-names = "ohci", "ehci";
};
```

In the preceding example, the device is assigned three memory regions and two interrupt lines. The resource name list and resources respect a one-to-one mapping. This means, for example, that the name at index `0` will be assigned to the resource at the same index. The code in the driver to extract each named resource is as follows:

```
struct resource *res_mem_config, resirq, *res_mem1;
int txirq, rxirq;
/*let's grab region <0x4a064000 0x800>*/
res_mem1 = platform_get_resource_byname(pdev,
                   IORESOURCE_MEM, "ohci");
/*let's grab region <0x4a064c00 0x200>*/
res_mem_config = platform_get_resource_byname(pdev,
                  IORESOURCE_MEM, "config");
txirq = platform_get_resource_byname(pdev, 
                          IORESOURCE_IRQ, "ohci");
rxirq = platform_get_resource_byname(pdev,
                          IORESOURCE_MEM, "ehci");
```

As you can see, requesting resources in the named manner is less error-prone. That said, both `platform_get_resource()` and `platform_get_resource_byname()` are generic APIs used to deal with resources. However, there are dedicated APIs that allow you to reduce the development effort (such as `platform_get_irq_byname()`, `platform_get_irq()`, or `platform_get_and_ioremap_resource()`), as we will learn in later chapters.

## Extracting application-specific data

Application-specific data is data that is beyond the common resources (neither IRQ numbers, memory regions, regulators, nor clocks). These are arbitrary properties and child nodes that can be assigned to a device. Usually, such properties use a manufacture prefix. These can be any kind of string, Boolean, or integer value, along with their API defined in `drivers/of/base.c` in the Linux sources. Note that the examples we discuss here are not exhaustive. Now, let's reuse the node that was defined earlier in this chapter:

```
node_label: nodename@reg{
    string-property = "a string";
    string-list = "red fish", "blue fish";
    one-int-property = <197>; /* One cell property */
   /* in the following line, each number (cell) is a
    * 32-bit integer(uint32). There are 3 cells in
    * this property */
    int-list-property = <0xbeef 123 0xabcd4>;
    mixed-list-property = "a string", <0xadbcd45>, 
                          <35>, [0x01 0x23 0x45];
    byte-array-property = [0x01 0x23 0x45 0x67];
    one-cell-property = <197>;
    boolean-property;
};
```

In the sections that follow, we will learn how to obtain each property in the preceding device tree node excerpt.

### Extracting string properties

The following is an excerpt of the previous example, showing a single string and multiple string properties:

```
string-property = "a string";
string-list = "red fish", "blue fish";
```

Back in the driver, there are different APIs that can be used depending on the need. You should use `of_property_read_string()` to read a single string value property. Its prototype is defined as follows:

```
int of_property_read_string(const struct device_node *np,
                           const char *propname,
                           const char **out_string)
int of_property_read_string_index(const struct 
                                 device_node *np,
                                 const char *propname,
                                 int index, 
                                 const char **output)
int of_property_read_string_array(
                  const struct device_node *np,
                  const char *propname, 
                  const char **out_strs,
                  size_t sz)
```

In the preceding functions, `np` is the node from which the string property needs to be read. `propname` is the name of the property hosting the string or string list, and `sz` is the number of array elements to read. `out_string` is an output parameter whose pointer value will be changed to point to the string value(s).

`of_property_read_string()` and `of_property_read_string_index()` return `-ENODATA` if the property does not have a value. Alternatively, they return `-EINVAL` if the property does not exist at all. Additionally, `-EILSEQ` is returned if the string is not `NULL`-terminated within the length of the property data. Finally, on success, they return `0`.

`of_property_read_string_array()` searches for the specified property in the given device tree node, retrieves a list of `NULL`-terminated string values (actually a pointer to these strings, not a copy) inside that property, and assigns it to `out_strs`. In addition to the values returned by `of_property_read_string()` and `of_property_read_string_index()`, this function is special in the way that it returns the number of strings that have been read if the target array of pointers is not `NULL`. The `out_strs` parameter can be omitted (`NULL`) if you simply wish to count the number of strings in the property.

The following code shows how you can use them:

```
size_t count;
const char **res;
const char *my_string = NULL;
const char *blue_fish = NULL;
of_property_read_string(pdev->dev.of_node,
                      "string-property", &my_string);
of_property_read_string_index(pdev->dev.of_node,
                      "string-list", 1, &blue_fish);
count = of_property_read_string_array(dp, 
                      "string-list", res, count);
```

In the preceding example, we learned how to extract string properties, either from a single-value property or from a list. It must be noted that a pointer to the string (or string list) is returned, not a copy. Additionally, in the last line, we see how to extract a given number of NULL-terminated string elements in the array. Here again, a pointer to these elements are returned, not copies of these.

### Reading cells and unsigned 32-bit integers

Here are our `int` properties:

```
one-int-property = <197>;
int-list-property = <1350000 0x54dae47 1250000 1200000>;
```

Back in the driver, as with string properties, there is a set of APIs that you can choose according to your need. You should use `of_property_read_u32()` to read a cell value. Its prototype is defined as follows:

```
int of_property_read_u32(const struct device_node *np,
                        const char *propname,
                        u32 *out_value)
int of_property_read_u32_index(
                      const struct device_node *np,
                      const char *propname,
                      u32 index, u32 *out_value)
int of_property_read_u32_array(
                      const struct device_node *np,
                      const char *propname,
                      u32 *out_values, size_t sz)
```

The preceding APIs behave in the same way as their `_string`, `_string_index`, and `_string_array` counterparts. This is because they all return `0` on success, `-EINVAL` if the property does not exist at all, or `-ENODATA` when the property does not have a value. The differences are that the type of value to read, in this case, is `u32`, that we have an `-EOVERFLOW` error if the property data isn't large enough, and that `out_values` must be allocated first because the values to be returned have been copied instead.

The following is the usage of these APIs in our example, with `int` and our list of `int` properties:

```
unsigned int number;
of_property_read_u32(pdev->dev.of_node,
                    "one-cell-property", &number);
```

You can use `of_property_read_u32_array` to read a list of cells. Its prototype is as follows:

```
int of_property_read_u32_array(
                           const struct device_node *np,
                           const char *propname,
                           u32 *out_values, size_t sz);
```

Here, `sz` is the number of array elements to read. Take a look at `drivers/of/base.c` to see how to interpret its return value:

```
unsigned int cells_array[4]; /* return value by copy */
if (!of_property_read_u32_array(pdev->dev.of_node,
    "int-list-property", cells_array, 4))
    dev_info(&pdev->dev, "u32 list read successfully\n");
/* can now process values in cells_array */
[...]
```

Here, we have demonstrated how easy it would be to deal with an array of cells, that is, arrays of 32-bit integer values. In the next section, we will see how to do that with Boolean properties.

### Handling Boolean properties

You should use `of_property_read_bool()` to read the Boolean property whose name is given in the second argument of the function:

```
bool my_bool = of_property_read_bool(pdev->dev.of_node,
                                  "boolean-property");
if(my_bool){
    /* boolean is true */
} else
    /* boolean is false */
}
```

The preceding example demonstrates how to deal with Boolean properties. Now we can learn about far more complex APIs, starting with extracting and parsing sub-nodes.

### Extracting and parsing sub-nodes

Note that you are allowed to add whatever sub-node you want to a device node. This usage is common in numerous use cases, such as population partitions in an MTD device node or describing regulator constraints in a power management chip node. For example, given a node representing a flash memory device, partitions can be represented as nested sub-nodes. The following excerpt shows how this is achieved:

```
eeprom: ee24lc512@55 {
    compatible = "microchip,24xx512";
    reg = <0x55>;
    partition@1 {
        read-only;
        part-name = "private";
        offset = <0>;
        size = <1024>;
    };
    config@2 {
        part-name = "data";
        offset = <1024>;
        size = <64512>;
    };
};
```

You can use `for_each_child_of_node()` to walk through sub-nodes of the given node:

```
structdevice_node *np = pdev->dev.of_node;
structdevice_node *sub_np;
for_each_child_of_node(np, sub_np) {
    /* sub_np will point successively to each sub-node */
    [...]
    int size;
    of_property_read_u32(client->dev.of_node,
                        "size", &size);
    ...
}
```

In the preceding excerpt, we learned how to iterate over the sub-nodes of a given device tree node.

# Summary

The time to switch from hardcoded device configurations to device trees has come. This chapter gave you all the bases to handle device trees. Now you have the necessary skills to customize or add whatever node and property you want to the device tree and extract them from your driver.

In the next chapter, we will discuss the I2C driver, and use the device tree API to enumerate and configure our I2C devices.