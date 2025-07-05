# 第七章：*第七章*：理解平台设备和驱动程序的概念

Linux 内核通过使用总线的概念来管理设备，也就是说，CPU 与这些设备之间的连接。一些总线足够智能，内嵌了发现逻辑来枚举连接在其上的设备。在这种总线下，Linux 内核在启动初期会请求这些总线返回它们枚举的设备以及它们所需的资源（如中断线和内存区域），以便设备能够正常工作。PCI、USB 和 SATA 总线都属于这类可发现总线。

不幸的是，现实并不总是如此美好。仍有许多设备是 CPU 无法检测到的。这些不可发现的设备大多数位于芯片上，虽然有一些设备位于速度较慢或没有设备发现支持的总线上。

因此，内核必须提供接收硬件信息的机制，用户必须向内核提供这些设备的位置信息。在 Linux 内核中，这些不可发现的设备被称为 **平台设备**。由于它们不位于 I2C、SPI 或任何不可发现总线等已知总线上，Linux 内核实现了 **平台总线**（也称为 **伪平台总线**）的概念，以维持设备始终通过总线连接到 CPU 的范式。

本章将学习如何以及在哪里实例化平台设备及其资源，并学习如何编写它们的驱动程序，即平台驱动程序。为此，本章将分为以下几个主题：

+   理解 Linux 内核中的平台核心抽象

+   处理平台设备

+   平台驱动程序抽象及架构

+   从零开始编写平台驱动程序的示例

# 理解 Linux 内核中的平台核心抽象

为了涵盖随着 **系统芯片**（**SoCs**）越来越流行，越来越多使用的不可发现设备，平台核心已被引入。在此框架下，最重要的三个数据结构如下：表示平台设备的一个，表示其资源的另一个，最后一个数据结构表示平台驱动程序。

平台设备在内核中表示为 `struct platform_device` 的一个实例，该结构在 `<linux/platform_device.h>` 中定义，如下所示：

```
struct platform_device {
     const char       *name;
     u32              id;
     struct device    dev;
     u32              num_resources;
     struct resource *resource;
     const struct platform_device_id  *id_entry;
     struct mfd_cell *mfd_cell;
};
```

在之前的数据结构中，`name` 是平台设备的名称。分配给平台设备的名称必须小心选择。平台设备在伪平台总线匹配函数中与相应的驱动程序进行匹配，即 `platform_match()`。在此函数中，在某些情况下（没有设备树或 ACPI 支持且没有 `id` 表匹配），匹配会回退到名称匹配，比较驱动程序的名称和平台设备的名称。

`dev`是 Linux 设备模型的底层设备结构，`id`用于扩展该设备名称。以下描述了`id`的使用方式：

+   当`id`为`-1`（对应`PLATFORM_DEVID_NONE`宏）时，底层设备名称将与平台设备名称相同。平台核心将执行以下操作：

    ```
    dev_set_name(&pdev->dev, "%s", pdev->name);
    ```

+   当`id`为`-2`（对应`PLATFORM_DEVID_AUTO`宏）时，内核将自动生成一个有效的 ID，并将底层设备命名如下：

    ```
    dev_set_name(&pdev->dev, "%s.%d.auto", pdev->name,
                 <auto_id>);
    ```

+   在其他情况下，`id`将按如下方式使用：

    ```
    dev_set_name(&pdev->dev, "%s.%d", pdev->name,
                 pdev->id);
    ```

`resource`是分配给平台设备的资源数组，`num_resources`是该数组中元素的数量。

在平台设备和驱动通过 ID 表匹配的情况下，`pdev->id_entry`（其类型为`struct platform_device_id`）将指向匹配的 ID 表条目，从而使平台驱动与该平台设备匹配。

无论平台设备如何注册，它们都需要由合适的驱动程序驱动，即平台驱动。这些驱动程序必须实现一组回调函数，平台核心将在设备出现在平台总线时调用这些回调函数。

平台驱动在 Linux 内核中以`struct platform_driver`的实例形式表示，定义如下：

```
struct platform_driver {
     int (*probe)(struct platform_device *);
     int (*remove)(struct platform_device *);
     void (*shutdown)(struct platform_device *);
     int (*suspend)(struct platform_device *,
                         pm_message_t state);
     int (*resume)(struct platform_device *);
     struct device_driver driver;
     const struct platform_device_id *id_table;
     bool prevent_deferred_probe;
}; 
```

以下描述了该数据结构中使用的元素：

+   `probe()`: 这是在设备与驱动匹配后，设备请求使用你的驱动时调用的函数。稍后我们将看到核心如何调用`probe`。其声明如下：

    ```
    int my_pdrv_probe(struct platform_device *pdev)
    ```

内核负责提供`platform_device`参数。当设备驱动程序在内核中注册时，`probe`由总线驱动程序调用。

+   `remove()`: 当驱动程序不再被设备需要时，调用该函数以移除驱动，声明形式如下：

    ```
    static void my_pdrv_remove(struct platform_device *pdev)
    ```

+   `driver`是设备模型的底层驱动结构，必须提供名称（名称的选择必须谨慎）、所有者和一些其他字段（如设备树匹配表），稍后我们会看到。在平台驱动方面，在驱动和设备匹配之前，`platform_device.name`和`platform_driver.driver.name`字段必须相同。

+   `id_table`是平台驱动提供给总线代码的绑定实际设备到驱动的方式之一。另一种方式是通过设备树，这将在*在驱动中提供支持的设备*章节中讨论。

现在我们已经介绍了平台设备和平台驱动数据结构，让我们超越这些，尝试理解它们是如何创建和注册到系统中的。

# 处理平台设备

在开始编写平台驱动之前，本节将教你如何以及在哪里实例化平台设备。只有在此之后，我们才会深入研究平台驱动的实现。Linux 内核中的平台设备实例化已存在很长时间，并在各个内核版本中得到了改进，我们将在本节讨论每种实例化方法的具体细节。

## 分配和注册平台设备

由于平台设备无法使自己为系统所知，因此它们必须手动填充并与其资源和私有数据一起注册到系统中。在平台核心早期，平台设备在板文件`arch/arm/mach-*`中声明（对于 i.MX6 而言，是`arch/arm/mach-imx/mach-imx6q.c`），并通过`platform_device_add()`或`platform_device_register()`使内核知道，具体取决于每个平台设备的分配方式。

这使我们得出结论，分配和注册平台设备有两种方法。

一种静态方法，涉及在代码中列举平台设备，并对每个设备调用`platform_device_register()`进行注册。此 API 定义如下：

```
int platform_device_register(struct platform_device *pdev)
```

这样的使用示例如下：

```
static struct platform_device my_pdev = {
    .name           = "my_drv_name",
    .id             = 0,
    .ressource         = jz4740_udc_resources,
    .num_ressources    = ARRY_SIZE(jz4740_udc_resources),
};
int foo()
{
    [...]
    return platform_device_register(&my_pdev);
}
```

在静态注册中，平台设备被静态初始化并传递给`platform_device_register()`。此方法还允许批量添加平台设备。相应的代码可以使用`platform_add_devices()`，该函数接受指向平台设备的指针数组和数组中的元素数量。此函数定义如下：

```
int platform_add_devices(struct platform_device **devs,
                          int num);
```

另一方面，动态分配需要首先调用`platform_device_alloc()`来动态分配和初始化平台设备，定义如下：

```
struct platform_device *platform_device_alloc(
                          const char *name, int id);
```

在参数中，`name`是我们要添加的设备的基本名称，`id`是平台设备的实例 ID。如果成功，则此函数返回有效的平台设备对象，出错时返回`NULL`。

以这种方式分配的平台设备使用`platform_device_add()` API 进行注册，定义如下：

```
int platform_device_add(struct platform_device *pdev);
```

作为唯一参数，平台设备已使用`platform_device_alloc()`进行分配。如果成功，则此函数返回`0`，如果失败则返回负错误代码。

如果使用`platform_device_add()`进行平台设备注册失败，则应使用`platform_device_put()`释放平台设备结构占用的内存，定义如下：

```
void platform_device_put(struct platform_device *pdev);
```

以下是如何使用它的示例：

```
status = platform_device_add(evm_led_dev);
if (status < 0) {
     platform_device_put(evm_led_dev);
[...]
}
```

也就是说，无论平台设备以何种方式分配和注册，都必须使用`platform_device_unregister()`进行注销，定义如下：

```
void platform_device_unregister(struct platform_device *pdev)
```

请注意，`platform_device_unregister()`内部调用`platform_device_put()`。

现在我们知道了如何通过传统方式实例化平台设备，接下来让我们看看采用另一种方法——设备树，来实现相同目标会有多简洁。

## 如何不将平台设备分配给你的代码

正如我们之前所说，平台设备曾经是通过板文件或其他驱动程序进行填充的。使用这种方法时没有灵活性。添加/删除平台设备，最好的情况是重新编译一个模块，最坏的情况是重新编译整个内核。这个已弃用的方法也不可移植。

如今，设备树已经存在了相当长的一段时间。这个机制用于声明系统中存在的非可发现设备。作为一个独立的实体，它可以独立于内核或其他模块构建。事实证明，这可能是填充平台设备的最佳替代方案。

这是通过将这些平台设备声明为设备树中的节点来实现的，声明的位置是一个兼容字符串属性为 `simple-bus` 的节点。这个兼容字符串意味着该节点被视为一个总线，不需要特定的处理或驱动程序。而且，它还表示没有办法动态探测这个总线，唯一找到这个总线下子设备的方法是通过设备树中的地址信息。然后，在 Linux 的实现中，最终会为每个在 `compatible` 属性中包含 `simple-bus` 的节点下的第一层子节点创建一个平台设备。

以下是一个演示：

```
foo {
    compatible = "simple-bus";
    bar: bar@0 {
        compatible = "labcsmart,something";
        [...]
        baz: baz@0 {
            compatible = "labcsmart,anotherthing";
            [...]
        }
    }
    foz: foz@1 {
        compatible = "company,product";
        [...]
    };
}
```

在上面的示例中，只有 `bar` 和 `foz` 节点会被注册为平台设备。`baz` 节点不会（因为它的直接父节点没有在兼容字符串中包含 `simple-bus`）。因此，如果存在任何平台驱动程序，其兼容性匹配表中有 `company, product` 和/或 `labcsmart,something`，那么这些平台设备将会被探测到。

常见的用法是将 SoC 节点或片上内存映射总线下的设备声明为片上设备。另一个常见用法是声明稳压器设备，如下所示：

```
regulators {
    compatible = "simple-bus";
    #address-cells = <1>;
    #size-cells = <0>;

    reg_usb_h1_vbus: regulator@0 {
        compatible = "regulator-fixed";
        reg = <0>;
        regulator-name = "usb_h1_vbus";
        regulator-min-microvolt = <5000000>;
        regulator-max-microvolt = <5000000>;
        enable-active-high;
        startup-delay-us = <2>; 
        gpio = <&gpio7 12 0>;
    };
    reg_panel: regulator@1 {
        compatible = "regulator-fixed";
        reg = <1>;
        regulator-name = "lcd_panel";
        enable-active-high;
         gpio = <&gpio1 2 0>;
     };
};
```

在上面的示例中，将会注册两个平台设备，每个设备对应一个固定的稳压器。

## 与平台资源的交互

在与热插拔设备相对的端，热插拔设备会列举其所需的资源并进行广告宣传，而内核对系统中存在哪些平台设备、它们的功能或它们所需的资源以确保正常工作一无所知。没有自动协商过程，因此，任何有关给定平台设备所需资源的信息提供给内核都会受到欢迎。这些资源可能包括 IRQ 行、DMA 通道、内存区域、I/O 端口等等。

在平台代码中，资源被表示为一个 `struct resource` 实例，它在 `include/linux/ioport.h` 中定义，如下所示：

```
struct resource {
     resource_size_t start;
     resource_size_t end;
     const char *name;
     unsigned long flags;
};
```

在前述数据结构中，`start`/`end` 指向资源的起始/结束。它们指示 I/O 或内存区域的起始和终止位置。因为 IRQ 线、总线和 DMA 通道没有范围，所以通常将 `start`/`end` 赋予相同的值。

`flags` 是一个掩码，用于描述资源的类型，例如 `IORESOURCE_BUS`。可能的值如下：

+   `IORESOURCE_IO` 用于 PCI/ISA I/O 端口

+   `IORESOURCE_MEM` 用于内存区域

+   `IORESOURCE_REG` 用于寄存器偏移量

+   `IORESOURCE_IRQ` 用于 IRQ 线

+   `IORESOURCE_DMA` 用于 DMA 通道

+   `IORESOURCE_BUS` 用于总线

最后，`name` 元素标识或描述资源，因为可以通过名称提取资源。

将这些资源分配给平台设备可以通过两种方式进行，首先，在声明和注册平台设备的同一编译单元中进行，其次，从设备树中进行。

在本节中，我们描述了资源并展示了它们的使用方法。接下来让我们看看它们如何在下一节中提供给平台设备。

### 平台资源配置 - 旧的和已弃用的方法

此方法用于不支持设备树的内核或者不存在这种选择时。它主要用于**多功能设备**（**MFD**），其中主（外围）芯片与子设备共享其资源。

通过这种方法，资源的提供方式与平台设备相同。以下是一个示例：

```
static struct resource foo_resources[] = {
     [0] = { /* The first memory region */
          .start = 0x10000000,
          .end   = 0x10001000,
          .flags = IORESOURCE_MEM,
          .name  = "mem1",
     },
     [1] = {
          .start = JZ4740_UDC_BASE_ADDR2,
          .end   = JZ4740_UDC_BASE_ADDR2 + 0x10000 -1,
          .flags = IORESOURCE_MEM,
          .name  = "mem2",
     },
     [2] = {
          .start = 90,
          .end   = 90,
          .flags = IORESOURCE_IRQ,
          .name  = "mc-irq",
     },
};
```

前文摘录显示了三种资源（两种内存类型和一种 IRQ 类型），它们的类型可以用 `IORESOURCE_IRQ` 和 `IORESOURCE_MEM` 进行标识。第一个是 4 KB 的内存区域，而第二个也是一个内存区域，其范围由宏定义，最后是 IRQ 90。

将这些资源分配给平台设备是一个简单的操作。如果平台设备已静态分配，则应以以下方式分配资源：

```
static struct platform_device foo_pdev = {
     .name = "foo-device",
     .resource             = foo_resources,
     .num_ressources = ARRY_SIZE(foo_resources),
[...]
};
```

对于动态分配的平台设备，这是在函数内部完成的，应该类似以下内容：

```
struct platform_device *foo_pdev;
[...]
my_pdev = platform_device_alloc("foo-device", ...);
if (!my_pdev)
           return -ENOMEM;
my_pdev->resource = foo_resources;
my_pdev->num_ressources = ARRY_SIZE(foo_resources);
```

有几个辅助函数可从资源数组中获取数据；这些包括以下内容：

```
struct resource *platform_get_resource(
                       struct platform_device *pdev, 
                       unsigned int type, unsigned int n);
struct resource *platform_get_resource_byname(
                      struct platform_device *pdev,
                      unsigned int type, const char *name);
int platform_get_irq(struct platform_device *pdev,
                     unsigned int n);
```

`n` 参数表示所需类型的资源中的哪一个，其中零表示第一个。因此，例如，驱动程序可以使用以下内容找到其第二个 MMIO 区域：

```
r = platform_get_resource(pdev, IORESOURCE_MEM, 1);
```

此函数的使用方法在《第五章》的*处理资源*部分中有解释，*理解和利用设备树*。

### 理解平台数据的概念

到目前为止，我们使用的 `struct resource` 足以为简单的平台设备实例化资源，但许多设备比这复杂得多。这种数据结构只能编码有限类型的信息。作为扩展，`platform_device.device.platform_data` 用于为平台设备分配任何其他额外信息。

数据可以被包含在一个更大的结构体中，并分配到这个额外的字段中。这些数据可以是驱动程序能够理解的任何类型，但大多数情况下，它们是我们在前面章节中列举的资源类型之外的其他数据类型。例如，这些数据可能是调节器约束或甚至是指向每个设备函数的指针。

让我们在下面的代码块中描述与一组指向特定于平台设备类型的函数的指针对应的额外平台数据：

```
static struct foo_low_level foo_ops = {
     .owner          = THIS_MODULE,
     .hw_init         = trizeps_pcmcia_hw_init,
     .socket_state     = trizeps_pcmcia_socket_state,
     .configure_socket = trizeps_pcmcia_configure_socket,
     .socket_init     = trizeps_pcmcia_socket_init,
     .socket_suspend  = trizeps_pcmcia_socket_suspend,
[...]
};
```

对于静态分配的平台设备，我们将执行以下命令：

```
static struct platform_device my_device = {
     .name = "foo-pdev",
     .dev  = {
          .platform_data   = (void*)&trizeps_pcmcia_ops,
     },
     [...]
};
```

如果平台设备被发现是动态分配的，它将通过`platform_device_add_data()`助手进行分配，定义如下：

```
int platform_device_add_data(struct platform_device *pdev,
                        const void *data, size_t size);
```

上述函数在成功时返回`0`。`data`是用作平台数据的数据，`size`是该数据的大小。

回到我们之前举的函数集合的例子，我们将执行以下操作：

```
int ret;
[…]
ret = platform_device_add_data(foo_pdev,
                  &foo_ops, sizeof(foo_ops));
if (ret == 0)
    ret = platform_device_add(trizeps_pcmcia_device);
if (ret)
     platform_device_put(trizeps_pcmcia_device);
```

在平台驱动程序中，平台设备将其`pdev->dev.platform_dat`元素指向平台数据。虽然我们可以取消引用这个字段，但建议使用内核提供的函数`dev_get_platdata()`，定义如下：

```
void *dev_get_platdata(const struct device *dev)
```

然后，为了获取包含的函数结构集，驱动程序可以执行以下操作：

```
struct foo_low_level *my_ops =
          dev_get_platdata(&pdev->dev);
```

驱动程序无法检查传递的数据类型。驱动程序只能假设它们已被提供了一个预期类型的结构，因为平台数据接口缺乏任何类型检查。

### 平台资源配置 – 新的推荐方法

第一种资源配置方法有一些缺点，包括任何更改都需要重新构建内核或已更改的模块，这可能会增加内核的大小。

随着设备树的到来，事情变得更简单了。为了保持兼容性，设备树中指定的内存区域、中断和 DMA 资源会被平台核心转换为`struct resources`实例，以便`platform_get_resource()`、`platform_get_resource_by_name()`，甚至`platform_get_irq()`都可以返回适当的资源，无论该资源是通过传统方式填充的还是来自设备树的。这可以在*第五章*中的*处理资源*部分得到验证，*理解和利用设备树*。

不用多说，设备树允许传递驱动程序可能需要了解的任何类型的信息。这些数据类型可用于传递任何设备/驱动程序特定的数据。这可以通过阅读*第五章*中的*提取特定应用数据*部分来实现，*理解和利用设备树*。

然而，设备树代码并不知道给定驱动程序为其平台数据使用的具体结构，因此无法以这种形式提供这些信息。为了传递额外的数据，驱动程序可以使用`of_device_id`条目中的`.data`字段，该条目促使平台设备和驱动程序匹配。该字段可以指向平台数据。请参见本章中的*驱动程序中支持设备的配置*部分。

传统方式下期望平台数据的驱动程序应检查`platform_device->dev.platform_data`指针。如果那里有非空值，这意味着设备是通过传统方式实例化的，并且没有使用设备树；在这种情况下，应继续使用平台数据。如果设备是从设备树代码实例化的，`platform_data`指针将为`NULL`，表明必须直接从设备树获取信息。在这种情况下，驱动程序将在平台设备的`dev.of_node`字段中找到`device_node`指针。然后，可以使用各种设备树访问例程（最显著的是`of_get_property()`）从设备树中提取所需的数据。

现在我们已经熟悉了平台资源配置，接下来让我们学习如何设计一个平台设备驱动程序。

# 平台驱动程序抽象和架构

在继续之前，让我们先提醒一下。并非所有平台设备都由平台驱动程序处理（或者，应该说，由伪平台驱动程序处理）。平台驱动程序专用于那些不依赖传统总线的设备。I2C 设备或 SPI 设备是平台设备，但分别依赖 I2C 或 SPI 总线，而不是平台总线。所有操作都需要通过平台驱动程序手动完成。

## 探测和释放平台设备

平台驱动程序的入口点是探测方法，该方法在与平台设备匹配后被调用。该探测方法的原型如下：

```
int pdrv_probe(struct platform_device *pdev)
```

`pdev`对应的是通过传统方式实例化的或由于相关设备树节点的直接父节点具有`simple-bus`兼容属性而由平台核心分配的新平台设备。如果有，平台数据和资源也将相应地设置。如果参数中的平台设备是驱动程序所期望的设备，则此方法必须返回`0`。否则，必须返回适当的负错误代码。

无论平台设备是通过传统方式实例化还是通过设备树实例化，都可以使用常规的平台核心 API（如`platform_get_resource()`、`platform_get_resource_by_name()`，甚至`platform_get_irq()`等）提取其资源。

探测方法必须请求驱动程序所需的任何资源（如 GPIO、时钟、IIO 通道等）和数据。如果需要进行映射，探测方法是执行此操作的合适位置。

`probe`方法中完成的所有操作必须在设备离开系统或取消注册平台驱动程序时撤消。`remove`方法是实现这一点的适当位置。它必须具有以下原型：

```
int pdrv_remove(struct platform_device *dev)
```

此函数必须仅在所有操作均已撤消并清理完成时返回`0`。否则，必须返回适当的错误代码，以便通知用户。参数中包括已传递给`probe`函数的相同平台设备。

在*从头开始编写平台驱动程序的示例*部分中，我们讨论了实现`probe`和`remove`方法的所有技巧和特殊情况。

现在驱动程序回调已准备就绪，我们可以填充由此驱动程序处理的设备。

## 驱动程序中支持的设备供应

我们的平台驱动程序本身是无用的。要使其对设备有用，必须告知内核它可以管理哪些设备。为实现此目的，必须提供 ID 表，分配给`platform_driver.id_table`字段。这将允许平台设备匹配。但是，为了将此功能扩展到模块自动加载，必须将相同的表提供给`MODULE_DEVICE_TABLE`，以便生成模块别名。

回到代码，该表中的每个条目均为`struct platform_device_id`类型，定义如下：

```
struct platform_device_id {
     char name[PLATFORM_NAME_SIZE];
     kernel_ulong_t driver_data;
};
```

在上述数据结构中，`name`是设备的描述性名称，`driver_data`是驱动程序状态值。它可以设置为指向每个设备数据结构的指针。以下是一个示例，摘自`drivers/mmc/host/mxs-mmc.c`：

```
static const struct platform_device_id mxs_ssp_ids[] = {
     {
          .name = "imx23-mmc",
          .driver_data = IMX23_SSP,
     }, {
          .name = "imx28-mmc",
          .driver_data = IMX28_SSP,
     }, {
          /* sentinel */
     }
};
MODULE_DEVICE_TABLE(platform, mxs_ssp_ids);
```

当将`mxs_ssp_ids`分配给`platform_driver.id_table`字段时，平台设备将能够根据它们的名称与任何`platform_device_id.name`条目匹配此驱动程序。`platform_device.id_entry`将指向触发匹配的此表中的条目。

要允许匹配通过其兼容字符串在设备树中声明的平台设备，平台驱动程序必须使用`struct of_device_id`元素的列表设置`platform_driver.driver.of_match_table`。然后，为了允许从设备树匹配进行模块自动加载，必须将此设备树匹配表提供给`MODULE_DEVICE_TABLE`。以下是一个示例：

```
static const struct of_device_id mxs_mmc_dt_ids[] = {
    {
        .compatible = "fsl,imx23-mmc",
        .data = (void *) IMX23_SSP,
    },{ 
        .compatible = "fsl,imx28-mmc",
        .data = (void *) IMX28_SSP,
    }, {
        /* sentinel */ 
    }
};
MODULE_DEVICE_TABLE(of, mxs_mmc_dt_ids);
```

如果在驱动程序中设置了`of_device_id`，则匹配是通过与设备节点中的兼容属性的任何`of_device_id.compatible`元素匹配来判断的。要获取引起匹配的`of_device_id`条目，驱动程序应调用`of_match_device()`，并传递设备树匹配表和底层设备结构`platform_device.dev`作为参数。

以下是一个示例：

```
static int mxs_mmc_probe(struct platform_device *pdev)
{
    const struct of_device_id *of_id =
        of_match_device(mxs_mmc_dt_ids, &pdev->dev);
    struct device_node *np = pdev->dev.of_node;
[...]
}
```

在定义这些匹配表后，它们可以按以下方式分配给平台驱动程序结构：

```
static struct platform_driver imx_uart_platform_driver = {
    .probe = imx_uart_probe,
    .remove = imx_uart_remove,
    .id_table = imx_uart_devtype,
    .driver = {
        .name = "imx-uart",
        .of_match_table = imx_uart_dt_ids,
    },
};
```

在前述最终平台驱动程序数据结构中，我们根据设备树匹配表或 ID 表，最后根据驱动程序名称，为设备提供匹配平台驱动程序的机会。

## 驱动程序初始化和注册

将平台驱动程序注册到内核中，就像在模块初始化函数中调用`platform_driver_register()`或`platform_driver_probe()`一样简单。然后，为了删除已注册的驱动程序，模块必须调用`platform_driver_unregister()`来取消注册该驱动程序。

以下是这些函数的各自原型：

```
int platform_driver_register(struct platform_driver *drv);
void platform_driver_unregister(struct platform_driver *);
int platform_driver_probe(struct platform_driver *drv,
                  int (*probe)(struct platform_device *))
```

这两个探测函数之间的区别如下：

+   `platform_driver_register()`函数将驱动程序注册并将其放入内核维护的驱动程序列表中，这意味着每当出现与平台设备匹配的情况时，内核可以按需调用其`probe()`函数。为了防止驱动程序被插入并注册到该列表中，可以使用下一个函数。也就是说，任何使用`platform_driver_register()`注册的平台驱动程序都必须通过`platform_driver_unregister()`取消注册。

+   使用`platform_driver_probe()`时，内核会立即运行匹配循环，检查是否有平台设备可以与此平台驱动程序匹配，并在每个匹配的设备上调用`probe`方法。如果此时没有找到设备，平台驱动程序将被简单地忽略。该方法防止了延迟探测，因为它不会将驱动程序注册到系统中。在这里，`probe`函数被放置在`__init`段中，该段在内核启动完成后会被释放（前提是驱动程序是静态编译的，即内置驱动程序），从而防止了延迟探测，并减少了驱动程序的内存占用。如果你确信设备在系统中存在，可以使用这种方法：

    ```
    ret = platform_driver_probe(&mypdrv, my_pdrv_probe);
    ```

如果设备已知不可热插拔，可以将`probe()`过程放置在`__init`段中。

注册平台驱动程序的合适时机是在模块加载后，这使我们可以说，合适的做法是在模块初始化函数中进行注册。同样，取消注册驱动程序也必须在模块的卸载路径中完成。由于这一点，我们可以说，`module_exit`函数是取消注册平台驱动程序的合适位置。

以下是一个典型的平台驱动程序注册到平台核心的示例：

```
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/platform_device.h>
static int my_pdrv_probe (struct platform_device *pdev)
{
     pr_info("Hello! device probed!\n");
     return 0;
}
static void my_pdrv_remove(struct platform_device *pdev)
{
     pr_info("good bye reader!\n");
}
static struct platform_driver mypdrv = {
     .probe     = my_pdrv_probe,
     .remove     = my_pdrv_remove,
[...]
};
static int __init foo_init(void)
{
     [...] /* My init code */
     return platform_driver_register(&mypdrv);
}
module_init(foo_init);

static void __exit foo_cleanup(void)
{
     [...] /* My clean up code */
     platform_driver_unregister(&my_driver);
}
module_exit(foo_cleanup);
```

我们的模块在初始化/退出函数中除了通过平台总线核心注册/注销平台驱动程序之外，不做其他操作。这是一个常见的情况，其中模块的初始化/退出方法非常简单。在这种情况下，我们可以去掉`module_init()`和`module_exit()`，而使用`module_platform_driver()`宏，代码如下：

```
[...]
static int my_pdrv_probe(struct platform_device *pdev)
{
     [...]
}
static void my_pdrv_remove(struct platform_device *pdev)
{
     [...]
}
static struct platform_driver mypdrv = {
     [...]
};
module_platform_driver(mypdrv);
```

尽管在前面的示例中我们仅插入了快照代码，但这已经足够演示如何减少代码并使其更简洁。

为确保我们在相同的知识基础上，下面将总结本章所学的所有概念，并将其应用于一个实际的平台驱动程序。

# 从头开始编写平台驱动程序的示例

本节将尽可能总结到目前为止在本章中获得的知识。现在，让我们设想一个内存映射的平台设备，其可以通过的内存范围从`0x02008000`开始，大小为`0x4000`。然后，假设这个平台设备在完成任务后能够中断 CPU，且该中断线路编号为`31`。为了简化问题，假设该设备不需要其他资源（我们本可以设想时钟、DMA、调节器等资源）。

首先，让我们从设备树实例化这个平台设备。如果你记得的话，为了让平台核心将一个节点注册为平台设备，这个节点的直接父节点必须在其兼容性字符串列表中包含`simple-bus`，而这正是这里实现的内容：

```
demo {
    compatible = "simple-bus";
    demo_pdev: demo_pdev@0 {
        compatible = "labcsmart,demo-pdev";
        reg = <0x02008000 0x4000>;
        interrupts = <0 31 IRQ_TYPE_LEVEL_HIGH>;
    };
};
```

如果我们需要以传统的方式实例化这个平台设备及其资源，我们将不得不在一个我们称之为`demo-pdev-init.c`的文件中执行类似以下的操作：

```
#include <linux/module.h>
#include <linux/init.h>
#include <linux/platform_device.h>
#define DEV_BASE 0x02008000
#define PDEV_IRQ 31
static struct resource pdev_resource[] = {
    [0] = {
        .start = DEV_BASE,
        .end   = DEV_BASE + 0x4000,
        .flags = IORESOURCE_MEM,
    },
    [1] = {
        .start = PDEV_IRQ,
        .end   = PDEV_IRQ,
        .flags = IORESOURCE_IRQ,
    },
};
```

在前面的内容中，我们首先定义了平台资源。现在我们可以实例化平台设备并分配这些资源，如下所示，仍然是在`demo-pdev-init.c`中：

```
struct platform_device demo_pdev = {
    .name        = "demo_pdev",
    .id          = 0,
    .num_resources = ARRAY_SIZE(pdev_resource),
    .resource      = pdev_resource,
    /*.dev  = {
     *   .platform_data  = (void*)&big_struct_1,
     *}*/,
};
```

在前面的内容中，我们注释掉了与`dev`相关的赋值操作，目的是为了展示我们本可以定义一个更大的结构体，包含驱动程序期望的任何其他信息，并将这个数据结构作为平台数据传递。

现在所有数据结构都已设置完毕，我们可以通过在`demo-pdev-init.c`的底部添加以下内容来注册平台设备：

```
static int demo_pdev_init()
{
    return platform_device_register(&demo_pdev);
}
module_init(demo_pdev_init);
static void demo_pdev_exit()
{
    platform_device_unregister(&demo_pdev);
}
module_exit(demo_pdev_exit);
```

前面的代码除了将平台设备注册到系统中，别无他用。在这个例子中，我们使用了静态初始化。也就是说，动态初始化和静态初始化并没有太大区别。

现在我们完成了平台设备的实例化，接下来让我们关注平台驱动程序本身，其编译单元名为`demo-pdriver.c`，并向其中添加以下头文件：

```
#include <linux/module.h>
#include <linux/init.h>
#include <linux/interrupt.h>
#include <linux/of_platform.h>
#include <linux/platform_device.h>
```

现在，必要的头文件已到位，允许我们支持所需的 API 并使用适当的数据结构，让我们从设备树中开始列举能够与此平台驱动程序匹配的设备：

```
static const struct of_device_id labcsmart_dt_ids[] = {
    {
        .compatible = " labcsmart,demo-pdev",
        /* .data = (void *)&big_struct_1, */
    },{ 
        .compatible = " labcsmart,other-pdev ",
        /* .data = (void *) &big_struct_2, */
    }, {
        /* sentinel */ 
    }
};
MODULE_DEVICE_TABLE(of, labcsmart_dt_ids);
```

在前面的内容中，设备树匹配表列出了通过`compatible`属性区分的支持设备。同样，`.data`赋值被注释掉，目的是展示如何根据与平台设备匹配的条目传递平台特定数据。这个平台特定的结构体本可以是`big_struct_1`或`big_struct_2`。当你需要传递平台数据并且仍然想使用设备树时，我推荐使用这种方法。

为了提供使用 ID 表进行匹配的机会，我们如下所示填充了这样一个表：

```
static const struct platform_device_id labcsmart_ids[] = {
     {
          .name = "demo-pdev",
          /*.driver_data = &big_struct_1,*/
     }, {
           .name = "other-pdev",
           /*.driver_data = &big_struct_2,*/
     }, {
           /* sentinel */
     }
};
MODULE_DEVICE_TABLE(platform, labcsmart_ids);
```

前述内容不需要特别的注释，除了被注释掉的部分，它展示了如何根据所处理的设备使用额外的数据。

在这个阶段，我们可以继续实现探测方法。仍然在我们的演示平台驱动编译单元中，我们添加了如下内容：

```
static u32 *reg_base;
static struct resource *res_irq;
static int demo_pdrv_probe(struct platform_device *pdev)
{
    struct resource *regs;
    regs = platform_get_resource(pdev, IORESOURCE_MEM, 0);
    if (!regs) {
        dev_err(&pdev->dev, "could not get IO memory\n");
        return -ENXIO;
    }
    /* map the base register */ 
    reg_base = devm_ioremap(&pdev->dev, regs->start,
                              resource_size(regs));
    if (!reg_base) {
        dev_err(&pdev->dev, "could not remap memory\n");
        return 0;
    }
    res_irq = platform_get_resource(pdev,
                                   IORESOURCE_IRQ, 0);
    /* we could have used
     * irqnum = platform_get_irq(pdev, 0); */
    devm_request_irq(&pdev->dev, res_irq->start,
                   top_half, IRQF_TRIGGER_FALLING,
                   "demo-pdev", NULL);
[...]
     return 0;
}
```

在前述的探测方法中，可以添加很多东西或者以不同的方式处理。第一个情况是如果我们的驱动程序仅符合设备树，并且相关的平台注册设备也从设备树实例化。我们可以这样做：

```
struct device_node *np = pdev->dev.of_node;
const struct of_device_id *of_id =
        of_match_device(labcsmart_dt_ids, &pdev->dev);
struct big_struct *pdata_struct = (big_struct*)of_id->data;
```

在前述内容中，我们获取了与平台注册设备关联的设备树节点的引用，在此基础上我们可以使用任何与设备树相关的 API（如`of_get_property()`）从中提取数据。接下来，在通过匹配表提供受支持设备时，如果传递了平台特定的数据结构，我们使用`of_match_device()`来指向对应匹配项的条目，并提取平台特定数据。

如果匹配是通过 ID 表发生的，`pdev->id_entry`将指向发生匹配的条目，`pdev->id_entry->driver_data`将指向适当的更大结构。

然而，如果平台数据是以传统方式声明的，我们将使用如下内容：

```
struct big_struct *pdata_struct =
                 (big_struct*)dev_get_platdata(&pdev->dev);
```

现在我们已经完成了探测方法，我想提醒你一个特别的点：在该方法中，我们仅使用了带有`devm_`前缀的函数来处理资源。这些函数会在适当的时机自动释放资源。

这意味着我们不需要一个`remove`方法。然而，为了教学目的，以下是如果我们不使用资源管理帮助函数时，`remove`方法的实现方式：

```
int demo_pdrv_remove(struct platform_device *dev)
{
    free_irq(res_irq->start, NULL);
    iounmap(reg_base);
    return 0;
}
```

在前述内容中，IRQ 资源被释放，内存映射被销毁。

好了，现在一切都就绪。我们可以使用以下内容来初始化并注册平台驱动：

```
static struct platform_driver demo_driver = {
    .probe     = demo_pdrv_probe,
    /* .remove = demo_pdrv_remove, */
    .driver    = {
        .owner    = THIS_MODULE,
        .name    = "demo_pdev",
    },
};
module_platform_driver(demo_driver);
```

还有一个重要点需要考虑：平台设备、平台驱动以及设备树中的设备节点名称都相同，即`demo_pdev`。这提供了一种即使在设备树、ID 表和 ACPI 匹配失败时，通过平台设备和平台驱动名称进行匹配的机会，因为名称匹配会作为所有其他匹配失败后的后备方案。

# 总结

内核伪平台总线不再对你保留任何秘密。通过总线匹配机制，你可以理解你的驱动程序是如何、何时、为何加载的，以及它是为哪个设备编写的。我们可以根据所需的匹配机制实现任何探测功能。由于设备驱动程序的主要目的是处理设备，我们现在能够在系统中填充设备，无论是通过传统方式还是通过设备树。为了完美收尾，我们从零开始实现了一个功能性平台驱动示例。

在下一章中，我们将继续学习如何为不可发现的设备编写驱动程序，不过这次是在 I2C 总线上进行。
