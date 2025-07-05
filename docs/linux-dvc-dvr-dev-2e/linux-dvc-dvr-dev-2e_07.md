# 第五章：*第五章*：理解和利用设备树

设备树是一个易于阅读的硬件描述文件，具有类似 JSON 的格式风格。它是一个简单的树形结构，其中设备通过节点及其属性表示。这些属性可以为空（即仅通过键描述布尔值），也可以是键值对，其中值可以包含任意字节流。本章是设备树的简要介绍。每个内核子系统或框架都有其自己的设备树绑定，我们将在处理相关主题时讨论这些特定的绑定。

设备树起源于**Open Firmware**（**OF**），这是一个计算机公司支持的标准，其主要目的是定义计算机固件系统的接口。也就是说，你可以在[`www.devicetree.org/`](http://www.devicetree.org/)了解更多关于设备树规范的信息。因此，本章将介绍设备树的基础知识，包括以下内容：

+   理解设备树机制的基本概念

+   描述数据类型及其 API

+   表示和访问设备

+   处理资源

# 理解设备树机制的基本概念

设备树的支持在内核中通过将`CONFIG_OF`选项设置为`Y`来启用。要从驱动程序中调用设备树 API，必须添加以下头文件：

```
#include <linux/of.h>
#include <linux/of_device.h>
```

设备树支持一些数据类型和写作约定，我们可以通过一个示例节点描述来总结：

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

在前面的示例中，`int-list-property`是一个属性，其中每个数字（或单元）是一个 32 位整数（`uint32`），该属性包含三个单元。这里，`mixed-list-property`顾名思义，是一个具有混合元素类型的属性。

以下是设备树中使用的一些数据类型的定义：

+   文本字符串用双引号表示。你可以使用逗号来创建字符串的列表。

+   单元是由尖括号括起来的 32 位无符号整数。

+   布尔数据不过是一个空属性。其真假值取决于该属性是否存在。

我们已经轻松列举了可以在设备树中找到的数据类型。在开始学习可以用来解析这些数据的 API 之前，首先让我们了解一下设备树的命名约定是如何工作的。

## 设备树命名约定

每个节点必须有一个形式为`<name>[@<address>]`的名称，其中`<name>`是一个最长为 31 个字符的字符串，`[@<address>]`是可选的，取决于该节点是否表示一个可寻址的设备。也就是说，`<address>`应该是用来访问设备的主要地址。例如，对于一个内存映射设备，它必须对应其内存区域的起始地址，I2C 设备的总线设备地址，和 SPI 设备节点的芯片选择索引（相对于控制器）。

以下是一些设备命名的示例：

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

在前面的设备树摘录中，I2C 控制器是一个内存映射设备。因此，节点名称中的地址部分对应于其内存区域的起始位置，相对于**片上系统**（**SoC**）的内存映射。然而，扩展器是一个 I2C 设备。因此，其节点名称中的地址部分对应于其 I2C 地址。

## 别名、标签、phandle 和路径概念简介

别名、标签、phandle 和路径是处理设备树时需要熟悉的关键字。处理设备驱动时，你很可能会遇到这些术语中的至少一个，甚至是所有术语。为了描述这些术语，我们以以下设备树摘录为例：

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

在设备树中，节点可以通过两种方式引用：通过路径或通过`phandle`属性，有时为了历史原因，可能会在`linux,phandle`属性中重复出现。

然而，设备树源格式允许将标签附加到任何节点或属性值上。由于标签必须在特定板卡的整个设备树源中是唯一的，因此显然可以使用标签来标识节点，并且已做出决定这么做。因此，已经在逻辑中加入了处理：当标签作为单元格属性中的`&`前缀时，它会被替换为该标签所附加节点的 phandle。此外，使用相同的逻辑，当标签在单元格外（简单的值赋值）以`&`符号为前缀时，它会被替换为该标签所附加节点的完整路径。通过这种方式，phandle 和路径引用可以通过引用标签来自动生成，而不必显式指定 phandle 值或节点的完整路径。

注意

标签仅用于设备树源格式，并不会被编码在`dtc`工具中，该工具会从节点中移除该标签并为该节点添加一个`phandle`属性，生成并分配一个唯一的 32 位值。然后，`dtc`工具将在每个引用该节点的单元格中使用该`phandle`（以&符号为前缀）。

回到前面的摘录，`gpio@0209c000`节点被标记为`gpio1`，这个标签也被作为引用使用。这将指示 DTC 为此节点生成一个 phandle。因此，在`accelerometer@1d`节点中，`interrupt-parent`属性内的单元格值（`&gpio1`）将被替换为`gpio1`所附加的节点的 phandle（单元格内部的赋值）。以同样的方式，在`aliases`节点中，`&gpio1`将被替换为`gpio1`所附加的节点的完整路径（单元格外部的赋值）。

在编译和反编译原始设备树摘录后，我们得到以下内容，其中标签不再存在，标签引用已被 phandle 或完整节点路径所替代：

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

在前面的代码片段中，加速度计节点的`interrupt-parent`属性单元被赋值为`0x40`。查看`gpio@209c000`节点，我们可以看到该值对应其`phandle`属性的值，该值在编译时由 DTC 生成。`aliases`节点也是如此，其中节点引用已经被替换为其完整路径。

这引出了别名的定义；别名只是通过它们的绝对路径进行引用的节点，用于快速查找。`aliases`节点可以看作是一个快速查找表。与标签不同，别名确实会出现在输出的设备树中，尽管路径是通过引用标签生成的。使用别名时，可以通过在别名部分查找来获取它所引用节点的句柄，而不是像通过`phandle`查找时那样在整个设备树中查找。别名可以看作是一种快捷方式，类似于我们在 Unix shell 中设置的别名，用来引用完整的/长的/重复的路径/命令。

Linux 内核会解除引用别名，而不是直接在设备树源中使用它们。当使用`of_find_node_by_path()`或`of_find_node_opts_by_path()`根据路径查找节点时，如果提供的路径没有以`/`开头，则路径的第一个元素必须是`/aliases`节点中的属性名称。该元素将被该别名的完整路径所替代。

注意

给节点打标签只有在该节点打算从另一个节点的属性中引用时才有意义。你可以将标签视为指向节点的指针，可以通过路径或引用来进行。

## 理解节点和属性的覆盖

在仔细查看解编译片段的结果后，你应该注意到另一件事：在原始源文件中，`i2c@21a0000`节点是通过其标签作为外部节点（`&i2c1 { [...] }`）引用的，并且有一些内容。然而，奇怪的是，在解编译后，`i2c@21a0000`节点的最终内容已经与外部引用的内容合并，而外部引用节点不再存在。

这是标签的第三种用途：允许你覆盖节点和属性。在外部引用中，任何新内容（如节点或属性）都将在编译时附加到原始节点内容上。然而，在出现重复时（无论是节点还是属性），外部引用的内容将优先于原始内容。

参考以下示例：

```
bus@2100000 { /* AIPS2 */
    [...]
    i2c1: i2c@21a0000 {
        [...]
        status = "disabled";
    };
};
```

让我们通过其`i2c1`标签引用`i2c@21a0000`节点，如下所示：

```
&i2c1 {
    [...]
    status = "okay";
};
```

我们将看到编译结果是`status`属性的值为`"okay"`。这是因为外部引用的内容优先于原始内容。

总结来说，后面的定义总是会覆盖前面的定义。为了完全覆盖一个节点，你只需要像重新定义属性一样重新定义它们。

## 设备树源和编译器

`.dts`扩展名，而二进制形式则有`.dtb`或`.dtbo`扩展名。`.dtbo`是一个特殊的扩展名，用于编译的设备树覆盖（`.dtsi`文本文件（其中末尾的`i`表示“包含”））。这些文件承载 SoC 级别的定义，并打算被包含在`.dts`文件中，后者则承载板级别的定义。

设备树的语法允许你使用`/include/`或`#include`来包含其他文件。这个包含机制使得使用`#define`成为可能，但最重要的是，它允许你在共享文件中提取多个平台的共同部分。

这种提取使得你能够将源文件拆分为树形结构，其中最常见的是 SoC 级别，由 SoC 供应商（例如 NXP）提供，其次是**系统模块**（**SoM**）级别（例如 Engicam），最后是承载板或客户板级别。

因此，所有使用相同 SoC 的电子板不需要从头开始重新定义 SoC 的所有外设：这些描述被提取到一个公共文件中。根据约定，这种*公共*文件使用`.dtsi`扩展名，而最终的设备树使用`.dts`扩展名。

在 Linux 内核源码中，ARM 设备树源文件可以在`arch/arm/boot/dts/`和`arch/arm64/boot/dts/<vendor>/`目录下找到，分别用于 32 位和 64 位 ARM SoC/板卡。在这两个目录中，都有一个`Makefile`文件，列出了可以编译的设备树源文件。

用于将 DTS 文件编译为 DTB 文件的工具叫做 DTC。DTC 的源代码存在于两个地方：

+   **作为一个独立的上游项目**：DTC 上游项目托管在[`git.kernel.org/cgit/utils/dtc/dtc.git`](https://git.kernel.org/cgit/utils/dtc/dtc.git)。它会定期拉取到 Linux 内核源树中。

+   `scripts/dtc/`。新版本会定期从上游项目拉取。DTC 在 Linux 内核构建过程中作为依赖项构建（例如，在编译设备树之前）。如果你希望在 Linux 内核源树中显式构建它，可以使用`make scripts`命令。

从内核源代码的主目录中，你可以选择编译特定的设备树或为特定 SoC 编译所有设备树。在这两种情况下，必须启用适当的配置选项以启用这些设备树文件。对于单个设备树的编译，make 目标是将`.dts`文件的名称中的`.dts`更改为`.dtb`。对于所有启用的设备树进行编译，应该使用的 make 目标是`dtbs`。在这两种情况下，你应确保已设置启用`dtb`的配置选项。

考虑以下来自`arch/arm/boot/dts/Makefile`的摘录：

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

通过启用 `CONFIG_SOC_IMX6Q`，你可以编译列表中所有的设备树文件，或者针对特定的设备树进行编译。通过运行 `make dtbs`，内核 DTC 将编译所有启用配置选项中列出的设备树文件。

首先，确保已经设置了适当的配置选项：

```
$ grep CONFIG_SOC_IMX6Q .config
CONFIG_SOC_IMX6Q =y
```

然后，我们来编译所有的设备树文件：

```
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make dtbs
```

假设我们的平台是 ARM64 平台，我们将使用以下命令：

```
ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- make dtbs
```

再次提醒，必须设置正确的内核配置选项。

你可以使用以下命令来针对特定的设备树构建（例如 `imx6q-hummingboard2.dts`）：

```
ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make imx6q-hummingboard2.dtb
```

必须注意，给定一个已编译的设备树（`.dtb`）文件，你可以进行反向操作，提取源代码（`.dts`）文件：

```
$ dtc -I dtb -O dts arch/arm/boot/dts/imx6q-hummingboard2.dtb > path/to/my_devicetree.dts
```

出于调试目的，可能会将运行系统的当前设备树暴露给用户空间，也就是所谓的实时设备树。为此，必须启用内核 `CONFIG_PROC_DEVICETREE` 配置选项。然后，你可以在 `/proc/device-tree` 目录中浏览和查看设备树。

如果在运行的系统上安装了 DTC，可以使用以下命令将文件系统树转换为更易读的形式：

```
# dtc -I fs -O dts /sys/firmware/devicetree/base > MySBC.dts
```

在此命令返回后，`MySBC.dts` 文件将包含与当前设备树相对应的源代码。

### 设备树覆盖

设备树覆盖是一种机制，允许你对实时设备树进行修补，即在运行时修改当前的设备树。它通过更新现有的节点和属性或创建新的节点和属性，允许你在运行时更新当前的设备树。然而，它不允许删除节点或属性。

设备树覆盖具有以下格式：

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

从前面的摘录中，我们可以注意到，每个需要覆盖的基础设备树节点都必须被包含在覆盖设备树中的 `fragment` 节点内。

然后，每个片段有两个元素：

+   这两个属性中的一个可能如下所示：

    +   `target-path`: 这指定了片段将要修改的节点的绝对路径。

    +   `target`: 这指定了片段将要修改的节点别名的相对路径（以 & 符号为前缀）。

+   一个名为 `__overlay__` 的节点，包含应该应用到被引用节点的更改。这些更改可以是新的节点（被添加的）、新的属性（被添加的），或者现有的属性（被新值覆盖）。不允许进行删除操作，因为无法删除节点或属性。

现在我们已经掌握了设备树覆盖的基础知识，接下来我们可以学习它们是如何被编译并转化为二进制 blob，随时加载的。

#### 构建设备树覆盖

除非设备树覆盖仅在根节点下添加新节点（在这种情况下，它可以在片段中指定 `/` 作为 `target-path` 属性），否则指定目标节点时通过其 phandle（`<&label_name>`）会更容易，因为这可以避免我们手动计算节点的完整路径（尤其是当它是嵌套的）。

问题在于，基础设备树和覆盖之间没有直接的关联或链接。它们各自独立构建。因此，从设备树覆盖中引用远程节点（即基础设备树中的节点）会引发错误，覆盖的构建将因为未定义的引用或标签而失败。这就像在没有符号解析空间的情况下构建一个动态链接的应用程序。

为了解决这个问题，DTC 已添加对 `-@` 命令行标志的支持。必须为基础设备树和所有覆盖指定该标志，才能编译。它会指示 DTC 在根节点中生成额外的节点（如 `__symbols__`、`__fixups__` 和 `__local_fixups__`），这些节点包含用于解析 phandle 名称的翻译数据。这些额外的节点分布如下：

+   当 `-@` 选项添加到构建覆盖时，它会识别标记设备树片段/对象的 `/plugin/;` 行。该行控制 `__fixups__` 和 `__local_fixups__` 节点的生成。

+   当 `-@` 选项添加到构建基础设备树时，`/plugin/;` 不存在，因此源被识别为基础设备树，这会导致仅生成 `__symbols__` 节点。

这些额外的节点为符号解析提供了空间。

注意

`-@` 选项的支持仅在 `dtc` 版本 1.4.4 或更高版本中找到。只有 Linux 内核版本 v4.14 或更高版本包括一个符合此要求的内置 `dtc` 版本。如果设备树覆盖中仅使用 `target-path` 属性（即非 phandle 基于属性），则不需要此选项。

构建二进制设备树覆盖的过程与构建传统的二进制设备树相同。例如，考虑以下基础设备树，我们将其命名为 `base.dts`：

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

接下来，让我们使用以下命令构建基础设备树：

```
dtc -@ -I dts -O dtb -o base.dtb base.dts
```

在下一步中，让我们考虑以下设备树覆盖，我们将其命名为 `foo-verlay.dts`：

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

在前面的设备树覆盖中，基础设备树中 `foo` 节点的 `status` 属性已从 `disabled` 修改为 `okay`，这将激活该节点。随后，添加了 `overlay-1-property` 布尔属性，最后，添加了一个 `bar` 子节点，并包含一个布尔属性。此设备树覆盖可以通过以下命令进行编译：

```
dtc -@ -I dts -O dtb -o foo-overlay.dtbo foo-overlay.dts
```

如你所见，`-@` 标志已添加到两侧，提供了符号解析的空间。

注意

在 Yocto 构建系统中，你可以将此标志添加到机器配置中，或者在开发过程中，将其添加到`local.conf`文件中，如下所示：`DEVICETREE_FLAGS += "-@"`。

为了让 Linux 内核构建设备树覆盖文件，你应当将其添加到你所使用的 SoC 架构的`Makefile`设备树中，例如`arch/arm64/boot/dts/freescale/Makefile`或`arm/arm/boot/dts/Makefile`，并使用`dtbo`扩展名，如下所示：

```
dtb-y += foo-overlay.dtbo
```

既然我们已经能够构建自己的设备树覆盖文件，接下来我们可以考虑一个逻辑上的下一步，即将这些覆盖文件加载到系统中。

#### 通过 configfs 加载设备树覆盖文件

本节的名称表明了设备树覆盖文件将被加载的方式（configfs），因为加载设备树覆盖文件的方式不止一种。在这一节中，我们将专注于在一个已经启动并且根文件系统已经挂载的系统中加载覆盖文件。

为了做到这一点，你的内核必须在编译时启用了`CONFIG_OF_OVERLAY`和`CONFIG_CONFIGFS`，才能使接下来的步骤有效。以下是一个检查示例，假设目标系统上已经有了内核配置：

```
~# zcat /proc/config.gz | grep CONFIGFS
CONFIG_CONFIGFS_FS=y
~# zcat /proc/config.gz | grep OF_OVERLAY
CONFIG_OF_OVERLAY=y
~# 
```

现在是时候使用`configfs`将 DTB 插入到正在运行的内核中了。首先，如果`configfs`文件系统尚未挂载，你需要先将其挂载：

```
# mount -t configfs none /sys/kernel/config
```

当`configfs`正确挂载后，目录应该包含基本的子目录（`device-tree/overlays`），根据我们的挂载路径，这将会是`/sys/kernel/config/device-tree/overlays`，如下所示：

```
# mkdir -p /sys/kernel/config/device-tree/overlays/
```

然后，必须从`overlays`目录中添加每个覆盖文件条目。需要注意的是，覆盖条目的创建和操作是通过标准的文件系统 I/O 进行的。

为了加载一个覆盖文件，必须在`overlays`目录下创建一个与该覆盖文件对应的目录。以我们的例子为例，我们使用名称`foo`：

```
# mkdir /sys/kernel/config/device-tree/overlays/foo
```

接下来，为了有效加载覆盖文件，你可以通过`echo`命令将覆盖固件文件路径写入到`path`属性文件中，如下所示：

```
# echo /path/to/foo-overlay.dtbo > /sys/kernel/config/device-tree/overlays/foo/path
```

另外，你可以将覆盖文件的内容通过`cat`命令写入到`dtbo`文件中：

```
# cat foo.dtbo > /sys/kernel/config/device-tree/overlays/foo/dtbo
```

之后，覆盖文件将会应用，设备将会根据需要被创建或销毁。

要移除覆盖文件并撤销其更改，你只需执行`rmdir`命令删除相应的覆盖文件目录。在我们的例子中，应该如下所示：

```
# rmdir /sys/kernel/config/device-tree/overlays/foo
```

尽管你已经动态加载了设备树覆盖文件，但这还不够；为了让新添加的设备节点工作，设备驱动程序也需要加载，除非这个驱动程序是内建的并且已经启用（即在`make menuconfig`期间选择了`y`）。

到此为止，我们已经完成了设备树编译相关的工作。现在，我们可以开始学习如何编写我们自己的设备树，从设备寻址和表示开始。

# 表示和寻址设备

在设备树中，节点是设备的表示单元。换句话说，一个设备至少由一个节点表示。接下来，设备节点可以包含其他节点（从而创建父子关系）或包含属性（这些属性描述与它们所填充节点对应的设备）。

尽管每个设备都可以独立操作，但也有一些情况，其中一个设备可能希望被其父设备访问，或者父设备可能希望访问其子设备。例如，当总线控制器（父节点）想要访问其总线上一个或多个设备（作为子节点声明）时，就会发生这种情况。典型的例子包括 I2C 控制器与 I2C 设备，SPI 控制器与 SPI 设备，CPU 与内存映射设备等。因此，设备地址分配的概念应运而生。设备地址分配通过 `reg` 属性引入，该属性用于每个可寻址设备，但其含义或解释依赖于父设备（通常是总线控制器）。子设备中 `reg` 的含义和解释取决于其父设备的 `#address-cells` 和 `#size-cells` 属性。`#`（井号）字符作为前缀出现在 `size-cells` 和 `address-cells` 中，可以理解为“长度”。

每个可寻址设备都会有一个 `reg` 属性，它是一个元组列表，形式为 `reg = <address0 size0 [address1 size1] [address2 size2] ... >`，其中每个元组代表设备使用的地址范围。`#size-cells` 表示用于表示大小的 32 位单元数量，如果大小不相关，可能为 0。另一方面，`#address-cells` 表示用于表示地址的 32 位单元数量。换句话说，每个元组的地址元素是根据 `#address-cells` 来解释的，它使用与大小元素相同的大小，而大小元素则根据 `#size-cells` 来解释。

总结来说，可寻址设备继承其父设备的 `#size-cell` 和 `#address-cell` 属性，而父设备通常是表示总线控制器的节点。某一设备中是否存在 `#size-cell` 和 `#address-cell` 不会影响设备本身，但会影响它的子设备（如果它们是可寻址的）。

现在我们已经看到了地址分配的一般原理，让我们开始讨论针对不可发现设备的具体地址分配，从 SPI 和 I2C 开始。

## 处理 SPI 和 I2C 设备地址分配

SPI 和 I2C 设备都属于非内存映射设备，因为它们的地址无法被 CPU 访问。相反，父设备的驱动程序（总线控制器驱动程序）将代表 CPU 进行间接访问。每个 I2C/SPI 设备节点总是表示为设备所在的 I2C/SPI 控制器节点的子节点。对于非内存映射设备，`#size-cells`属性为 0，寻址元组中的大小元素为空。这意味着这种设备的`reg`属性始终只有一个单元。以下是一个示例：

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

如果你查看`arch/arm/boot/dts/imx6qdl.dtsi`中的 SoC 级文件，你会注意到，在 I2C 和 SPI 控制器节点（标记为`i2c3`和`ecspi1`）中，`#size-cells`和`#address-cells`分别被设置为`0`和`1`。这有助于你理解它们的`reg`属性，其中地址值只需要一个单元，而大小值不需要单元。

I2C 设备的`reg`属性用于指定设备在总线上的地址。对于 SPI 设备，`reg`表示在控制器节点所拥有的芯片选择线列表中分配给设备的芯片选择索引。例如，对于`ad7606r8` ADC，芯片选择索引为 1，对应于`<&gpio5 17 0>`中的`cs-gpios`。这就是控制器节点的芯片选择列表。其他控制器的绑定方式可能不同，你应参考它们的文档，位于`Documentation/devicetree/bindings/spi`。

## 内存映射设备与设备寻址

本节讨论简单的内存映射设备，其中内存区域可以被 CPU 访问。对于这种设备节点，`reg`属性仍然定义设备的地址，并采用`reg = <address0 size0 [address1 size1] [address2 size2] ... >`的模式。每个区域都用一个单元组表示，第一个单元是内存区域的基地址，第二个单元是该区域的大小。这可以转化为以下模式：`reg = <base0 length0 [base1 length1] [address2 length2] ... >`。在这里，每个元组表示设备使用的地址范围。

让我们考虑以下示例：

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

在前面的摘录中，`compatible`属性中包含`simple-bus`的设备节点是 SoC 内部内存映射总线控制器，用于将 IP 核心（如 I2C、SPI、USB、以太网及其他内部 SoC IP）连接到 CPU。它们的子节点，可能是 IP 核心或其他内部总线，继承自`#address-cells`和`#size-cells`属性。我们可以看到，`i2c@021a0000` I2C 控制器（标记为`i2c1`）连接到`spba-bus@02000000`总线。它们都会在运行时作为平台设备出现在系统中。另一方面，这个 I2C 控制器通过定义自己的`#address-cells`和`#size-cells`属性来更改其寻址方案，连接到它的 I2C 设备将继承这些属性。SPI 控制器也是如此。

总结来说，在实际应用中，如果不清楚父节点的`#size-cells`和`#address-cells`属性，你不应当解读节点中的`reg`属性。内存映射设备必须在其`reg`属性的`size`字段中设置设备内存区域的大小，同时还需要定义`address`字段，该字段必须与 SoC 内存映射中设备内存区域的起始位置对应，如 SoC 的数据手册所示。

注意

simple-bus-compatible 字符串还表明该总线没有特殊的驱动程序，无法动态探测该总线，且直接子节点（仅限一级子节点）将作为平台设备注册。有时，这在板级设备树中用于实例化基于 GPIO 的固定电压调节器。

# 资源处理

设备驱动的主要目的是为给定设备提供一组驱动功能，并向用户暴露其能力。在这里，目标是收集设备的配置参数，特别是资源（如内存区域、中断线、DMA 通道等），这些资源将帮助驱动程序执行其工作。

## `struct resource`

一旦探测完成，分配给设备的资源（无论是在设备中还是在板/机器文件中）都会由`of_platform`或`platform`核心使用`struct resource`进行收集和分配，具体如下：

```
struct resource {
    resource_size_t start;
    resource_size_t end;
    const char *name;
    unsigned long flags;
[...]
};
```

以下列出了数据结构中各个元素的含义：

+   `start`：根据资源标志，这可以是内存区域的起始地址、IRQ 线编号、DMA 通道编号或寄存器偏移量。

+   `end`：这是内存区域的结束位置或寄存器偏移的结束位置。在 IRQ 或 DMA 通道的情况下，大多数时候，`start`和`end`的值是相同的。

+   `name`：这是资源的名称（如果有的话）。我们将在下一节中详细讨论它。

+   `flags`：指示资源的类型。可能的值包括以下几种：

    +   `IORESOURCE_IO`：指示 PCI/ISA I/O 端口区域。`start`是该区域的第一个端口，`end`是最后一个端口。

    +   `IORESOURCE_MEM`：用于 I/O 内存区域。`start`表示区域的起始地址，`end`表示区域的结束地址。

    +   `IORESOURCE_REG`：这指的是寄存器偏移量。它主要用于 MFD 设备。`start`表示相对于父设备寄存器的偏移量，而`end`表示寄存器段的结束位置。

    +   `IORESOURCE_IRQ`：该资源是 IRQ 线编号。在这种情况下，`start`和`end`的值要么相同，要么`end`无关紧要。

    +   `IORESOURCE_DMA`：表示该资源是一个 DMA 通道编号。你应当像处理 IRQ 一样处理`end`。然而，当 DMA 通道标识符有多个单元，或者当有多个 DMA 控制器时，情况并没有很好地定义。`IORESOURCE_DMA`在多控制器系统中不可扩展。

每个资源分配一个数据结构实例。这意味着，对于一个被分配了两个内存区域和一个 IRQ 线的设备，将分配三个数据结构。此外，同一类型的资源将按照它们在设备树（或板文件）中声明的顺序进行分配和索引（从 0 开始）。这意味着第一个分配的内存区域将具有索引 0，依此类推。

为了获取适当的资源，我们将使用一个通用的 API，`platform_get_resource()`，并传入资源类型以及该类型下的资源索引。该函数的定义如下：

```
struct resource *platform_get_resource(
                      struct platform_device *dev,
                      unsigned int type, unsigned int num)
```

在前面的原型中，`dev`是我们为其编写驱动程序的平台设备，`type`是资源类型，`num`是该资源在同一类型中的索引。成功时，该函数返回指向`struct resource`的有效指针，失败则返回`NULL`。

当同一类型下有多个资源时，使用索引可能会导致误解。你可能决定依赖资源名称作为替代方案，这时将引入`platform_get_resource()`的命名版本，即`platform_get_resource_byname()`。该函数传入一个资源标志（或类型）及其名称，返回相应的资源，无论它们在设备树中的声明顺序如何。

它的定义如下：

```
struct resource *platform_get_resource_byname(
                             struct platform_device *dev,
                             unsigned int type,
                           const char *name)
```

为了理解如何使用这个功能，首先让我们介绍一下命名资源的概念。我们将接下来讨论这个内容。

### 命名资源的概念

当驱动程序期望某种类型的资源列表时（假设是两个 IRQ 线，第一个用于 Tx，第二个用于 Rx），列表的顺序并不能得到保证，驱动程序不能做任何假设。如果驱动程序的逻辑硬编码假设先是 Rx IRQ，但设备树中是先配置了 Tx，会发生什么情况呢？为了避免这种不匹配，提出了命名资源的概念（如时钟、IRQ、DMA 通道和内存区域）。这包括定义资源列表并为其命名。无论它们的索引是什么，给定的名称总是能匹配到相应的资源。命名资源的概念还使得设备树的资源分配更加易于阅读和理解。

用于命名资源的对应属性如下：

+   `reg-names`：这是`reg`属性中内存区域的名称列表。

+   `interrupt-names`：这是为`interrupts`属性中的每个中断线命名的。

+   `dma-names`：这是用于`dma`属性的。

+   `clock-names`：这是用来给`clocks`属性中的时钟命名的。请注意，书中不会讨论时钟的相关内容。

为了演示这个概念，假设我们有如下的虚拟设备节点条目：

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

在前面的示例中，设备被分配了三个内存区域和两条中断线。资源名称列表和资源之间存在一一映射关系。这意味着，例如，索引为`0`的名称将被分配给相同索引的资源。驱动程序中提取每个命名资源的代码如下：

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

正如你所看到的，以命名方式请求资源更不容易出错。也就是说，`platform_get_resource()`和`platform_get_resource_byname()`是通用 API，用于处理资源。然而，也有专门的 API 可以帮助你减少开发工作量（例如`platform_get_irq_byname()`、`platform_get_irq()`或`platform_get_and_ioremap_resource()`），我们将在后面的章节中学习这些内容。

## 提取特定应用数据

特定应用数据是指超出通用资源的数据（既不是 IRQ 号、内存区域、调节器，也不是时钟）。这些是可以分配给设备的任意属性和子节点。通常，这些属性使用厂商前缀。它们可以是任何类型的字符串、布尔值或整数值，以及它们在 Linux 源代码中`drivers/of/base.c`中定义的 API。请注意，我们在这里讨论的示例并不详尽。现在，让我们重用本章早些时候定义的节点：

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

在接下来的章节中，我们将学习如何获取前面设备树节点摘录中的每个属性。

### 提取字符串属性

以下是前面示例的摘录，展示了一个单独的字符串和多个字符串属性：

```
string-property = "a string";
string-list = "red fish", "blue fish";
```

在驱动程序中，根据需求可以使用不同的 API。你应该使用`of_property_read_string()`来读取单个字符串值属性。其原型定义如下：

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

在前面的函数中，`np`是需要读取字符串属性的节点，`propname`是包含字符串或字符串列表的属性名称，`sz`是要读取的数组元素数量，`out_string`是一个输出参数，其指针值将被更改为指向字符串值。

如果属性没有值，`of_property_read_string()`和`of_property_read_string_index()`将返回`-ENODATA`。如果属性根本不存在，它们将返回`-EINVAL`。此外，如果字符串在属性数据的长度范围内没有以`NULL`结尾，则会返回`-EILSEQ`。最后，如果成功，它们将返回`0`。

`of_property_read_string_array()` 在给定的设备树节点中搜索指定的属性，检索该属性中的一组 NULL 终止的字符串值（实际上是指向这些字符串的指针，而不是副本），并将其分配给 `out_strs`。除了 `of_property_read_string()` 和 `of_property_read_string_index()` 返回的值外，这个函数有个特别之处，即如果目标指针数组不是 `NULL`，它会返回已读取的字符串数量。如果您只是想计算属性中的字符串数量，可以省略 `out_strs` 参数（设置为 `NULL`）。

以下代码展示了如何使用它们：

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

在上面的示例中，我们学习了如何提取字符串属性，无论是来自单一值属性还是来自列表。需要注意的是，返回的是字符串（或字符串列表）的指针，而不是副本。此外，在最后一行中，我们看到如何提取数组中给定数量的 NULL 终止字符串元素。同样，这里返回的是指向这些元素的指针，而不是它们的副本。

### 读取单元格和无符号 32 位整数

这是我们的 `int` 属性：

```
one-int-property = <197>;
int-list-property = <1350000 0x54dae47 1250000 1200000>;
```

在驱动程序中，与字符串属性一样，您可以根据需要选择一组 API。您应该使用 `of_property_read_u32()` 来读取单元格值。其原型定义如下：

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

上述 API 的行为与它们的 `_string`、`_string_index` 和 `_string_array` 对应函数相同。这是因为它们在成功时都返回 `0`，如果属性根本不存在，则返回 `-EINVAL`，如果属性没有值，则返回 `-ENODATA`。不同之处在于，读取的值类型在此情况下为 `u32`，如果属性数据不够大，则会返回 `-EOVERFLOW` 错误，并且 `out_values` 必须先分配，因为返回的值是已复制的，而不是原始数据。

以下是我们在示例中使用这些 API 的方法，涉及 `int` 类型和我们的 `int` 属性列表：

```
unsigned int number;
of_property_read_u32(pdev->dev.of_node,
                    "one-cell-property", &number);
```

您可以使用 `of_property_read_u32_array` 来读取一个单元格列表。其原型如下：

```
int of_property_read_u32_array(
                           const struct device_node *np,
                           const char *propname,
                           u32 *out_values, size_t sz);
```

这里，`sz` 是要读取的数组元素数量。请查看 `drivers/of/base.c` 以了解如何解释其返回值：

```
unsigned int cells_array[4]; /* return value by copy */
if (!of_property_read_u32_array(pdev->dev.of_node,
    "int-list-property", cells_array, 4))
    dev_info(&pdev->dev, "u32 list read successfully\n");
/* can now process values in cells_array */
[...]
```

在这里，我们演示了如何轻松处理一系列单元格，即 32 位整数值的数组。在下一节中，我们将看到如何使用布尔属性来处理这些。

### 处理布尔属性

您应该使用 `of_property_read_bool()` 来读取函数第二个参数中给定名称的布尔属性：

```
bool my_bool = of_property_read_bool(pdev->dev.of_node,
                                  "boolean-property");
if(my_bool){
    /* boolean is true */
} else
    /* boolean is false */
}
```

上面的示例演示了如何处理布尔属性。现在我们可以学习更复杂的 API，从提取和解析子节点开始。

### 提取和解析子节点

请注意，你可以将任何你想要的子节点添加到设备节点中。这种用法在许多应用场景中很常见，例如在 MTD 设备节点中表示分区，或在电源管理芯片节点中描述调节器约束。例如，给定一个表示闪存设备的节点，分区可以作为嵌套的子节点来表示。以下示例展示了如何实现这一点：

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

你可以使用`for_each_child_of_node()`来遍历给定节点的子节点：

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

在前面的示例中，我们学习了如何遍历给定设备树节点的子节点。

# 总结

从硬编码的设备配置切换到设备树的时机已经到来。本章为你提供了处理设备树的所有基础知识。现在你已经具备了自定义或添加任何节点和属性到设备树中的必要技能，并能从驱动中提取它们。

在下一章中，我们将讨论 I2C 驱动，并使用设备树 API 来枚举和配置我们的 I2C 设备。
