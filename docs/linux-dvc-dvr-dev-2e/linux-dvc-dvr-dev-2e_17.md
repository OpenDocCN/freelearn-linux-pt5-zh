# 第十四章：*第十四章*：Linux 设备模型简介

直到 2.5 版本之前，Linux 内核没有办法描述和管理对象，代码的可重用性也不像现在这样强大。换句话说，当时没有设备拓扑，也没有像现在在 sysfs 中那样的组织结构。没有关于子系统关系的信息，也没有关于系统是如何组合在一起的信息。然后出现了**Linux 设备模型**（**LDM**），它引入了以下功能：

+   类的概念。它们用于将相同类型或暴露相同功能的设备分组（例如，鼠标和键盘都是输入设备）。

+   通过虚拟文件系统与用户空间进行通信，允许你管理和枚举设备以及它们从用户空间暴露的属性。

+   使用引用计数进行对象生命周期管理。

+   电源管理功能，允许你管理设备关闭的顺序。

+   代码的可重用性。类和框架暴露接口，表现得像一种契约，任何注册它们的驱动程序都必须遵守。

+   **面向对象**（**OO**）风格的编程和内核中的封装。

在本章中，我们将利用 LDM，通过 sysfs 文件系统将一些属性导出到用户空间。为此，我们将覆盖以下主题：

+   LDM 数据结构简介

+   深入了解 LDM

+   从 sysfs 看设备模型概述

# LDM 数据结构简介

Linux 设备模型引入了设备层次结构。它建立在一些数据结构之上。其中包括总线，它在内核中表示为`struct bus_type`的一个实例；设备驱动程序，它由`struct device_driver`结构表示；以及设备，它是最后一个元素，表示为`struct device`结构的一个实例。在本节中，我们将介绍所有这些结构，并学习它们如何相互作用。

## 总线数据结构

总线是设备与处理器之间的通道链接。管理总线并向设备暴露协议的硬件实体被称为总线控制器。例如，USB 控制器提供 USB 支持，而 I2C 控制器提供 I2C 总线支持。然而，总线控制器作为一个设备本身，必须像任何设备一样进行注册。它将是需要连接到此总线的设备的父设备。换句话说，坐在总线上的每个设备的父设备字段必须指向总线设备。总线在内核中由`struct bus_type`结构表示，其定义如下：

```
struct bus_type {
    const char        *name;
    const char        *dev_name;
    struct device     *dev_root;
    const struct attribute_group **bus_groups;
    const struct attribute_group **dev_groups;
    const struct attribute_group **drv_groups;
    int (*match)(struct device *dev,
                 struct device_driver *drv);
    int (*probe)(struct device *dev);
    void (*sync_state)(struct device *dev);
    int (*remove)(struct device *dev);
    void (*shutdown)(struct device *dev);
    int (*suspend)(struct device *dev, pm_message_t state);
    int (*resume)(struct device *dev);
    const struct dev_pm_ops *pm;
[...]
};
```

这里只列出了与本书相关的元素；让我们更详细地了解它们：

+   `match` 是一个回调函数，每当新设备或驱动程序被添加到此总线时都会调用该回调。回调函数必须足够智能，当设备与驱动程序之间有匹配时，应返回一个非零值。`match` 回调的主要目的是让总线判断一个特定设备是否可以由给定的驱动程序处理，或者如果该驱动程序支持该设备的其他逻辑。大多数情况下，这种验证是通过简单的字符串比较完成的（设备和驱动程序名称，是否是表格和设备树兼容属性，等等）。对于枚举设备（如 PCI、USB），验证是通过比较驱动程序支持的设备 ID 与给定设备的设备 ID 来完成的，而不会牺牲总线特定的功能。

+   `probe` 是一个回调函数，当新设备或驱动程序在匹配后被添加到该总线时调用。此函数负责分配特定的总线设备结构，并调用给定驱动程序的 `probe` 函数，该函数负责管理设备（先前分配的）。

+   `remove` 在设备离开该总线时被调用。

+   `suspend` 是在总线上的设备需要进入睡眠模式时调用的方法。

+   `resume` 在总线上的设备必须从睡眠模式中恢复时调用。

+   `pm` 是此总线的电源管理操作集，也可能调用驱动程序特定的电源管理操作。

+   `drv_groups` 是指向 `struct attribute_group` 元素列表（数组）的指针，每个元素都有一个指向 `struct attribute` 元素列表（数组）的指针。它表示总线上设备驱动程序的默认属性。传递到此字段的属性将会赋予每个注册到总线上的驱动程序。这些属性可以在驱动程序的目录 `/sys/bus/<bus-name>/drivers/<driver-name>` 中找到。

+   `dev_groups` 表示总线上设备的默认属性。任何通过 `struct attribute_group` 元素的列表/数组传递到此字段的属性，将会赋予每个注册到总线上的设备。这些属性可以在设备的目录 `/sys/bus/<bus-name>/devices/<device-name>` 中找到。

+   `bus_group` 保持总线注册到核心时自动添加的默认属性集（组）。

除了定义`bus_type`外，总线驱动程序还必须定义一个特定于总线的驱动程序结构，它扩展了通用的`struct device_driver`结构，以及一个特定于总线的设备结构，它扩展了通用的`struct device`结构。这两者都是设备模型核心的一部分。总线驱动程序还必须为每个在探测时发现的物理设备分配一个特定于总线的设备结构。它还负责设置设备的总线和父字段，并将其注册到 LDM 核心。这些字段必须指向总线驱动程序中定义的`bus_type`和`bus_device`结构，LDM 核心将使用它们来构建设备层级结构并初始化其他字段。

每个总线内部管理着两个重要的列表：一个是已添加并挂载在其上的设备列表，另一个是已注册到该总线的驱动程序列表。每当你添加/注册或删除/注销设备/驱动程序时，相应的列表都会更新，加入新的条目。总线驱动程序必须提供辅助函数，以便注册/注销可以处理该总线上设备的驱动程序，以及注册/注销挂载在总线上的设备。这些辅助函数总是封装了 LDM 核心提供的通用函数，即`driver_register()`、`device_register()`、`driver_unregister()`和`device_unregister()`。

现在，让我们开始编写一个新的总线基础设施，叫做`PACKT`。`PACKT`将成为我们的总线；挂载在这条总线上的设备将是`PACKT`设备，而它们的驱动程序将是`PACKT`驱动程序。让我们开始编写允许我们注册`PACKT`设备和驱动程序的助手函数：

```
/*
 * Let's write and export symbols that people
 * writing drivers for packt devices must use.
 */
int packt_register_driver(struct packt_driver *driver)
{    
    driver->driver.bus = &packt_bus_type;
    return driver_register(&driver->driver);
}
EXPORT_SYMBOL(packt_register_driver);
void packt_unregister_driver(struct packt_driver *driver)
{
    driver_unregister(&driver->driver);
}
EXPORT_SYMBOL(packt_unregister_driver);
int packt_register_device(struct packt_device *packt)
{
    packt->dev.bus = &packt_bus_type;
    return device_register(&packt->dev);
}
EXPORT_SYMBOL(packt_device_register);
void packt_unregister_device(struct packt_device *packt)
{
    device_unregister(&packt->dev);
}
EXPORT_SYMBOL(packt_unregister_device);
```

现在我们已经创建了注册助手函数，接下来编写唯一的函数，允许我们分配一个新的`PACKT`设备并将其注册到`PACKT`核心：

```
struct packt_device * packt_device_alloc(const char *name,
                                          int id)
{
    struct packt_device    *packt_dev;
    int                    status;
    packt_dev = kzalloc(sizeof(*packt_dev), GFP_KERNEL);
    if (!packt_dev)
        return NULL;
    /* devices on the bus are children of the bus device */
    strcpy(packt_dev->name, name);
    packt_dev->dev.id = id;
    dev_dbg(&packt_dev->dev,
      "device [%s] registered with PACKT bus\n",
       packt_dev->name);
    return packt_dev;
}
EXPORT_SYMBOL_GPL(packt_device_alloc);
```

`packt_device_alloc()`函数分配了一个与总线相关的设备结构，必须使用它来将`PACKT`设备注册到总线。此时，我们应该暴露一些助手函数，允许我们分配一个`PACKT`控制器设备并注册它——也就是说，注册一个新的`PACKT`总线。为此，我们必须定义`PACKT`控制器数据结构，如下所示：

```
struct packt_controller {
    char name[48];
    struct device dev;    /* the controller device */
    struct list_head  list;
    int (*send_msg) (stuct packt_device *pdev,
                       const char *msg, int count);
    int (*recv_msg) (stuct packt_device *pdev,
                        char *dest, int count);
};
```

在前面的数据结构中，`name`表示控制器的名称，`dev`表示与该控制器关联的底层`struct device`，`list`用于将该控制器插入到系统的`PACKT`控制器全局列表中，而`send_msg`和`recv_msg`是控制器必须提供的钩子函数，用于访问挂载在其上的`PACKT`设备：

```
/* system global list of controllers */
static LIST_HEAD(packt_controller_list);
struct packt_controller
    *packt_alloc_controller(struct device *dev)
{
    struct packt_controller *ctlr;
    if (!dev)
        return NULL;
    ctlr = kzalloc(sizeof(packt_controller), GFP_KERNEL);
    if (!ctlr)
        return NULL;
    device_initialize(&ctlr->dev);
    [...]
    return ctlr;
}
EXPORT_SYMBOL_GPL(packt_alloc_controller);
int packt_register_controller(
                           struct packt_controller *ctlr)
{
    /* must provide at least on hook */
if (!ctlr->send_msg && !ctlr->recv_msg){
        pr_err("Registering PACKT controller failure\n");
    }
    device_add(&ctlr->dev);
    [...] /* other sanity check */
    list_add_tail(&ctlr->list, &packt_controller_list);
}
EXPORT_SYMBOL_GPL(packt_register_controller);
```

在这两个函数中，我们演示了控制器分配和注册操作的实现。分配方法分配内存并进行一些基本的初始化，为驱动程序留出空间以完成其余操作。请注意，在注册控制器之后，它会出现在 sysfs 中的`/sys/devices`下。任何添加到该总线的设备将会出现在`/sys/devices/packt-0/`下。

### 总线注册

总线控制器本身就是一个设备，在大多数情况下，总线是内存映射的平台设备（即使是总线也支持设备枚举）。例如，PCI 控制器就是一个平台设备，它的相应驱动程序也是。我们应该使用`bus_register(struct *bus_type)`函数将总线注册到内核中。`PACKT`总线结构如下所示：

```
/* This is our bus structure */
struct bus_type packt_bus_type = {
    .name     = "packt",
    .match    = packt_device_match,
    .probe    = packt_device_probe,
    .remove   = packt_device_remove,
    .shutdown = packt_device_shutdown,
};
```

现在基本的总线操作已经定义，我们需要注册`PACKT`总线框架并使其可供控制器和从属驱动程序使用。总线控制器本身就是一个设备；它必须注册到内核，并作为总线上设备的父设备。这个过程在总线控制器的探测或`init`函数中完成。对于`PACKT`总线，代码如下：

```
static int __init packt_init(void)
{
    int status;
    status = bus_register(&packt_bus_type);
    if (status < 0)
        goto err0;
    status = class_register(&packt_master_class);
    if (status < 0)
        goto err1;
    return 0;
err1:
    bus_unregister(&packt_bus_type);
err0:
    return status;
}
postcore_initcall(packt_init);
```

当设备由总线控制器驱动程序注册时，设备的`parent`成员必须指向总线控制器设备（这应由设备驱动程序完成），并且其`bus`属性必须指向`PACKT`总线类型（这是由核心完成的），以构建物理设备树。要注册一个`PACKT`设备，必须调用`packt_device_register()`，该函数应接收一个使用`packt_device_alloc()`分配的`PACKT`设备作为参数：

```
int packt_device_register(struct packt_device *packt)
{
    packt->dev.bus = &packt_bus_type;
    return device_register(&packt->dev);
}
EXPORT_SYMBOL(packt_device_register);
```

现在我们完成了总线注册，让我们看看驱动程序的架构，看看它是如何设计的。

## 驱动程序数据结构

驱动程序是一组允许我们驱动给定设备的方法。全局设备层次结构允许系统中所有设备以相同的方式表示。这使得核心能够简单地浏览设备树并执行诸如正确顺序的电源管理过渡等任务。

每个设备驱动程序都表示为`struct device_driver`的一个实例，其定义如下：

```
struct device_driver {
    const char        *name;
    struct bus_type   *bus;
    struct module     *owner;
    const struct of_device_id   *of_match_table;
    const struct acpi_device_id  *acpi_match_table;
    int (*probe) (struct device *dev);
    int (*remove) (struct device *dev);
    void (*shutdown) (struct device *dev);
    int (*suspend) (struct device *dev,
                    pm_message_t state);
    int (*resume) (struct device *dev);
    const struct attribute_group **groups;
    const struct dev_pm_ops *pm;
};
```

让我们看看这个数据结构中的元素：

+   `name`表示驱动程序的名称。它可以用于匹配，通过将其与设备的名称进行比较。

+   `bus`表示该驱动程序所处的总线。`bus`驱动程序必须填写此字段。

+   `module`表示拥有此驱动程序的模块。在 99%的情况下，您必须将此字段设置为`THIS_MODULE`。

+   `of_match_table`是指向`struct of_device_id`数组的指针。`struct of_device_id`结构用于通过一个特殊的文件（称为设备树）执行开放固件匹配，该文件在启动过程中传递给内核：

    ```
    struct of_device_id {
        char       compatible[128];
        const void *data;
    };
    ```

+   `suspend` 和 `resume` 是电源管理回调，分别在设备进入睡眠状态或从睡眠状态唤醒时调用。`remove` 回调在设备被物理移除系统时，或者当其引用计数达到 `0` 时被调用。系统重启时也会调用 `remove` 回调。

+   `probe` 是在尝试将驱动程序绑定到设备时运行的探测回调。总线驱动程序负责调用设备驱动程序的探测函数。

+   `group` 是指向 `struct attribute_group` 列表（数组）的指针，用作驱动程序的默认属性。优先使用此方法，而不是单独创建属性。

现在我们已经熟悉了驱动程序数据结构及其所有元素，让我们了解内核提供了哪些 API 供我们注册它。

### 驱动程序注册

低级 `driver_register()` 函数用于将设备驱动程序与总线注册，并将其添加到总线的驱动程序列表中。当设备驱动程序与总线注册时，核心会遍历该总线上所有设备的列表，并为每个没有驱动程序的设备调用总线的匹配回调。这样做是为了查找是否有设备是该驱动程序可以处理的。

以下是我们驱动程序基础设施的声明：

```
/*
 * Bus specific driver structure
 * You should provide your device's probe
 * and remove functions.
 */
struct packt_driver {
    int     (*probe)(struct packt_device *packt);
    int     (*remove)(struct packt_device *packt);
    void    (*shutdown)(struct packt_device *packt);
    struct device_driver driver;
    const struct i2c_device_id *id_table;
};
#define to_packt_driver(d) \
        container_of(d, struct packt_driver, driver)
#define to_packt_device(d) container_of(d, \
                             struct packt_device, dev)
```

在我们的示例中，有两个辅助宏，用于根据通用的 `struct device` 或 `struct driver` 获取 `PACKT` 设备和 `PACKT` 驱动程序。

然后是用于标识 `PACKT` 设备的结构，定义如下：

```
struct packt_device_id {
    char name[PACKT_NAME_SIZE];
    kernel_ulong_t driver_data;   /* Data private to the driver */
};
```

设备和设备驱动程序在匹配时绑定在一起。绑定是将设备与设备驱动程序关联的过程，由总线框架执行。

现在，让我们回到将驱动程序与 `PACKT` 总线注册的问题。驱动程序必须使用 `packt_register_driver(struct packt_driver *driver)`，它是 `driver_register()` 的封装。`driver` 参数必须在注册 `PACKT` 驱动程序之前填写好。LDM 核心提供了辅助函数，用于遍历已注册到总线的驱动程序列表：

```
int bus_for_each_drv(struct bus_type * bus,
                struct device_driver * start, void * data,
                int (*fn)(struct device_driver *, void *));
```

这个辅助函数遍历总线的驱动程序列表，并为列表中的每个驱动程序调用 `fn` 回调。

## 设备数据结构

`struct device` 是通用数据结构，用于描述和表征系统中的每个设备，无论其是否为物理设备。它包含关于设备物理属性的详细信息，并提供适当的链接信息，以帮助构建合适的设备树和引用计数：

```
struct device {
    struct device *parent;
    struct kobject kobj;
    const struct device_type *type;
    struct bus_type      *bus;
    struct device_driver *driver;
    void    *platform_data;
    void    *driver_data;
    struct device_node      *of_node;
    struct class *class;
    const struct attribute_group **groups;
    void    (*release)(struct device *dev);
[...]
};
```

为了便于阅读，前面的数据结构已被简化。尽管如此，我们来看一下已提供的元素：

+   `parent` 表示设备的父设备，用于构建设备树层次结构。当与总线注册时，总线驱动程序负责用总线设备设置此字段。

+   `bus` 表示设备所在的总线。总线驱动程序必须填写此字段。

+   `type`标识设备的类型。

+   `kobj`是 kobject，用于处理引用计数和设备模型支持。

+   `of_node`是指向与设备相关的开放固件（设备树）节点的指针。由总线驱动来设置此字段。

+   `platform_data`是指向特定于设备的平臺数据的指针。它通常在设备配置期间在板级特定文件中声明。

+   `driver_data`是指向驱动程序私有数据的指针。

+   `class`是指向该设备所属类的指针。

+   `group`是指向`struct attribute_group`类型的列表（数组）的指针，并用作设备的默认属性。你应该使用这个，而不是单独创建属性。

+   `release`是一个回调函数，当设备的引用计数降到零时被调用。总线有责任设置这个字段。`PACKT`总线驱动展示了如何执行此操作。

现在我们已经描述了设备的结构，接下来让我们学习如何通过注册将它纳入系统。

### 设备注册

`device_register()`是 LDM 核心提供的一个函数，用于将设备注册到总线。在这个调用之后，总线的驱动程序列表会被遍历，寻找支持该设备的驱动程序；然后，这个设备会被添加到总线的设备列表中。`device_register()`内部调用`device_add()`：

```
int device_add(struct device *dev)
{
    [...]
    bus_probe_device(dev);
        if (parent)
                klist_add_tail(&dev->p->knode_parent,
                            &parent->p->klist_children);
    [...]
}
```

内核提供的帮助函数，用于遍历总线的设备列表的是`bus_for_each_dev`。它的定义如下：

```
int bus_for_each_dev(struct bus_type * bus,
                    struct device * start, void * data,
                    int (*fn)(struct device *, void *));
```

每当一个设备被添加时，核心会调用总线驱动的匹配方法（`bus_type->match`）。如果匹配函数成功，核心会调用总线驱动的探测函数（`bus_type->probe`），前提是设备和驱动作为参数匹配。然后，由总线驱动来调用设备驱动的探测方法（即`driver->probe`）。对于我们的`PACKT`总线驱动，用于注册设备的函数是`packt_device_register(struct packt_device *packt)`，该函数内部调用`device_register()`。这里的参数是一个通过`packt_device_alloc()`分配的`PACKT`设备。

总线特定的设备数据结构随后定义如下：

```
/*
 * Bus specific device structure
 * This is what a PACKT device structure looks like
 */
struct packt_device {
    struct module        *owner;
    unsigned char        name[30];
    unsigned long        price;
    struct device        dev;
};
```

在前面的代码中，`dev`是设备模型的底层`struct device`结构，而`name`是设备的名称。

现在我们已经定义了数据结构，让我们了解一下 LDM 的底层机制。

# 更深入了解 LDM

到目前为止，我们讨论了总线、驱动程序和设备，它们被用来构建系统的设备拓扑。虽然这是真的，但前面讨论的只是冰山一角。在内部，LDM 依赖于三个最低级的数据结构，它们分别是`kobject`、`kobj_type`和`kset`。这些结构用于链接对象。

在我们进一步讨论之前，让我们定义一下本章中将要使用的一些术语：

+   `struct kobject`。

+   **属性**：一个属性（或 sysfs 属性）在 sysfs 中表现为一个文件。在内核中，它可以映射到任何事物：一个变量、一个设备属性、一个缓冲区，或者任何可能需要导出到外部的对驱动程序有用的东西。

在本节中，我们将了解这些结构如何参与设备模型。

## 理解 kobject 结构

`struct kobject`，即内核对象（也简称为 `struct kobject`，在内核中的某个地方徘徊）。此外，一个 kobject 可以导出一个或多个属性，这些属性在该 Kobject 的 sysfs 目录中表现为文件。现在，让我们回到代码中 —— `struct kobject` 在内核中的定义如下：

```
struct kobject {
    const char              *name;
    struct list_head        entry;
    struct kobject          *parent;
    struct kset             *kset;
    struct kobj_type        *ktype;
    struct sysfs_dirent     *sd;
    struct kref             kref;
[...]
};
```

在前面的数据结构中，仅列出了主要元素。让我们更详细地查看它们：

+   `name` 是此 kobject 的名称。可以通过 `kobject_set_name(struct kobject *kobj, const char *name)` 函数修改该名称。它用作此 `kobject` 目录的名称。

+   `parent` 是指向另一个 kobject 的指针，该 kobject 被视为此 kobject 的父对象。它用于构建拓扑并描述对象之间的关系。

+   `sd` 指向一个 `struct sysfs_dirent` 结构体，表示此 kobject 在 sysfs 中的目录。`name` 将用作此目录的名称。如果 `parent` 被设置，则该目录将成为父目录中的子目录。

+   `kref` 为 kobject 提供引用计数。它帮助跟踪对象是否仍在使用中，并在不再使用时可能释放它。或者，它可以防止对象在仍被使用时被删除。当它在内核对象中使用时，初始值为 `1`。

+   `ktype` 描述了 kobject。每个 kobject 在创建时都会赋予一组默认属性。`ktype` 元素属于 `struct kobj_type` 结构体，用于指定这些默认属性。这样的结构允许内核对象共享公共操作（`sysfs_ops`），无论这些对象是否在功能上相关。

+   `kset` 告诉我们该对象属于哪个对象集（组）。

在 kobject 被使用之前，必须对其进行（独占）动态分配并初始化。为此，驱动程序可以使用 `kzalloc()`（或 `kmalloc()`）或 `kobject_create()` 函数。使用 `kzalloc()` 时，对象会被分配并且为空，需要使用另一个 API `kobject_init()` 来初始化。使用 `kobject_create()` 时，分配和初始化是隐式的。这些 API 定义如下：

```
void kobject_init(struct kobject *kobj,
                  struct kobj_type *ktype)
struct kobject *kobject_create(void)
```

在前面的代码中，`kobject_init()`的第一个参数期望一个已通过`kzalloc()`等方法分配的 kobject。第二个参数`ktype`是必需的，不能为`NULL`；否则，内核将报错（`dump_stack()`）。另一方面，`kobject_create()`不期望任何参数；它隐式地完成了分配和初始化（它内部调用了`kzalloc()`和`kobject_init()`）。如果成功，它将返回一个刚刚初始化的 kobject 对象。

一旦一个 kobject 被初始化，驱动程序可以使用`kobject_add()`将这个对象与系统连接，创建这个 kobject 的 sysfs 目录项。这个目录会创建在哪里取决于 kobject 的父元素是否已设置。也就是说，如果父元素没有设置，可以在使用`kobject_add()`将 kobject 添加到系统时指定，`kobject_add()`的定义如下：

```
int kobject_add(struct kobject *kobj, struct kobject *parent,
                const char *fmt, ...);
```

在前面的函数中，`kobj`是要添加到系统中的内核对象，`parent`是它的父对象。`kobject`目录将作为父目录的子目录创建。如果`parent`为`NULL`，目录将直接在`/sys/`下创建。

与其单独使用`kobject_init()`或`kobject_create()`，然后再使用`kobject_add()`，不如使用`kobject_init_and_add()`，它将这些操作组合在一起。它的定义如下：

```
int kobject_init_and_add(struct kobject *kobj,
                         struct kobj_type *ktype,
                         struct kobject *parent,
                         const char *fmt, ...);
```

在前面的接口中，首先需要分配对象。还有另一个辅助函数会隐式地分配、初始化并将 kobject 添加到系统中。这个函数是`kobject_create_and_add()`，其定义如下：

```
struct kobject * kobject_create_and_add(const char *name,
                                     struct kobject *parent);
```

前面的函数接受一个 kobject 名称，用作目录名，以及一个父 kobject，其创建的目录将作为子目录。如果将`NULL`作为第二个参数传递，则目录将直接创建在`/sys/`下。

也就是说，内核中一些预定义的 kobjects 已经代表了`/sys/`下的一些目录。让我们看看其中的几个：

+   `kernel_kobj`：这个 kobject 负责`/sys/kernel`目录。

+   `mm_kobj`：这个对象负责`/sys/kernel/mm`。

+   `fs_kobj`：这是文件系统 kobject，负责`/sys/fs`。

+   `hypervisor_kobj`：这个对象负责`/sys/hypervisor`。

+   `power_kobj`：这是一个电源管理 kobject，用于`/sys/power`的起始位置。

+   `firmware_kobj`：这是拥有`/sys/firmware`目录的固件 kobject。

一旦完成 kobject 的使用，驱动程序应该释放它。你可以使用的低级函数是`kobject_release()`。然而，这个 API 没有考虑 kobject 的其他潜在用户。它是原始且简单的。建议使用`kobject_put()`代替，它会递减 kobject 的引用计数，并在新的引用计数值为`0`时释放该 kobject。记住，当 kobject 被初始化时，引用计数值被设置为`1`。此外，建议用户将 kobject 的使用包装到`kobject_get()`和`kobject_put()`函数中，其中`kobject_get()`只会增加引用计数值。

这些 API 具有以下原型：

```
void kobject_put(struct kobject * kobj);
struct kobject *kobject_get(struct kobject *kobj);
```

在前面的代码中，`kobject_get()`将 kobject 作为参数来增加引用计数，并在初始化参数后返回相同的 kobject。`kobject_put()`将从引用计数中减去 1，如果新的值为`0`，则会自动调用`kobject_release()`来释放该对象。

以下代码展示了如何结合`kobject_create()`、`kobject_init()`和`kobject_add()`来创建并将一个内核对象添加到系统中：

```
/* Somewhere */
static struct kobject *mykobj;
[...]
mykobj = kobject_create();
if (!mykobj)
    return -ENOMEM;
kobject_init(mykobj, &my_ktype);
if (kobject_add(mykobj, NULL, "%s", "hello")) {
    pr_info("ldm: kobject_add() failed\n");
    kobject_put(mykobj);
    mykobj = NULL;
    return -1;
}
```

正如我们所看到的，我们可以使用一体化函数，如`kobject_create_and_add()`，它内部调用`kobject_create()`和`kobject_add()`。以下是`drivers/base/core.c`中的摘录，展示了如何使用它：

```
static struct kobject * class_kobj   = NULL;
static struct kobject * devices_kobj = NULL;
/* Create /sys/class */
class_kobj = kobject_create_and_add("class", NULL);
if (!class_kobj)
    return -ENOMEM;
[...]
/* Create /sys/devices */
devices_kobj = kobject_create_and_add("devices", NULL);
if (!devices_kobj)
    return -ENOMEM;
```

请记住，对于每个`struct kobject`，相应的 kobject 目录可以在`/sys/`中找到，父目录由`kobj->parent`指向。

注意

由于在初始化 kobject 时必须提供一个`kobj_type`，因此一体化的辅助函数使用默认的、由内核提供的`kobj_type`；即`dynamic_kobj_ktype`。因此，除非你有充分的理由初始化你的`kobj_type`（通常是在你希望填充一些默认属性时），否则你应该使用`kobject_create*()`变体，它们使用内核提供的 kobject 类型，而不是`kobject_init*()`，后者需要提供你自己的初始化`kobj_type`。

## 理解 kobj_type 结构

`struct kobj_type`结构，即内核对象类型，是一个定义 kobject 元素行为并控制该 kobject 在创建或销毁时如何处理的数据结构。此外，`kobj_type`还包含 kobject 的默认属性，以及允许它操作这些属性的钩子函数。

因为同一类型的大多数设备具有相同的属性，这些属性被隔离并存储在`ktype`元素中。这样可以灵活地管理它们。每个 kobject 都必须有一个关联的`kobj_type`结构。其数据结构定义如下：

```
struct kobj_type {
    void (*release)(struct kobject *);
    const struct sysfs_ops sysfs_ops;
    struct attribute **default_attrs;
};
```

在前面的数据结构中，`release` 是一个回调函数，在 kobject 的释放路径上被调用，允许驱动程序有机会释放为该 kobject 分配的资源。当 `kobject_put()` 即将释放 kobject 时，这个回调函数会隐式运行。`default_attrs` 是一个指向 `attribute` 结构体的指针数组。该字段列出了每个该类型 kobject 创建的属性，而 `sysfs_ops` 提供了一组方法，允许你访问这些属性。

以下代码展示了内核中 `struct sysfs_ops` 数据结构的定义：

```
struct sysfs_ops {
    ssize_t (*show)(struct kobject *kobj,
                   struct attribute *attr, char *buf);
    ssize_t (*store)(struct kobject *kobj,
                   struct attribute *attr, const char *buf,
                   size_t size);
};
```

在前面的代码中，`show` 是一个回调函数，在读取由此 `kobj_type` 显示的属性时被调用——即每当从用户空间读取某个属性时。`buf` 是输出缓冲区。缓冲区的大小是固定的，长度为 `PAGE_SIZE`。必须暴露的数据需要放入 `buf` 中，最好使用 `scnprintf()`。最后，如果回调成功，它必须返回写入缓冲区的数据的大小（以字节为单位），如果失败则返回负值错误。每个属性应根据 sysfs 规则包含/提供一个单一的、可读的人类值或属性；如果你有大量数据要返回，应该考虑将其分解为多个属性。

`store` 用于写入操作——即当用户向属性写入数据时。它的 `buf` 参数最大为 `PAGE_SIZE`，但可以更小。成功时，必须返回从缓冲区读取的数据大小（以字节为单位），如果失败（或收到不期望的值）则返回负值错误。

`attr` 指针作为参数传递给这两个方法，可以用来确定正在访问的属性是哪一个。`show`/`store` 方法经常会对属性名称进行一系列测试来判断这一点。另一方面，其他实现则将属性的结构体封装在一个包含数据的结构体中，该结构体提供了返回属性值所需的数据（例如 `struct kobject_attribute`、`struct device_attribute`、`struct driver_attribute` 和 `struct class_attribute` 等）；在这种情况下，`container_of` 宏用于获取指向封装结构体的指针。这是 `kobj_sysfs_ops` 使用的方法，`kobj_sysfs_ops` 代表由 `dynamic_kobj_ktype` 提供的操作。两个方法在本书的示例中都有演示。

## 理解 `kset` 结构

`struct kset` 的目的是将相关的内核对象组合在一起。`kset` 代表内核对象集合，可以理解为一组 kobject。换句话说，`kset` 将相关的 kobject 聚集在一个地方，例如所有的 *块设备*。

`kset` 数据结构在内核中定义如下：

```
struct kset {
    struct list_head list; 
    spinlock_t list_lock;
    struct kobject kobj;
 };
```

数据结构中的所有元素都非常直观。简而言之，`list` 是 `kset` 中所有 kobjects 的链表，`list_lock` 是一个自旋锁，用于保护链表访问（在向 `kset` 中添加或移除 kobject 元素时），而 `kobj` 代表该集合的基础类 kobject。这个 kobject 将作为添加到集合中的 kobjects 的默认父级，前提是其父级为 `NULL`。

每个注册的 `kset` 对应一个由其 `kobj` 元素代表创建的 sysfs 目录。可以使用 `kset_create_and_add()` 函数创建并添加一个 `kset`，并通过 `kset_unregister()` 将其移除。以下代码显示了两者的定义：

```
struct kset * kset_create_and_add(const char *name,
                      const struct kset_uevent_ops *u,
                      struct kobject *parent_kobj);
void kset_unregister (struct kset * k);
```

在前面的 API 中，`name` 是 `kset` 的名称，也用作为 `kset` 创建的目录名称。`u` 参数是指向 `struct uevent_ops` 的指针，表示一组 `kset`，例如，它可以添加新的环境变量或在需要时过滤掉 uevents。这个参数可以是（而且大多数时候是）`NULL`。最后，`parent_kobj` 是 `kset` 的父级 kobject。

将 kobject 添加到集合中非常简单，只需为正确的 `kset` 指定它的 `.kset` 字段：

```
static struct kobject foo_kobj, bar_kobj;
[...]
example_kset = kset_create_and_add("kset_example",
                           NULL, kernel_kobj);
 /* since we have a kset for this kobject,
  * we need to set it before calling into the kobject core.
  */
foo_kobj.kset = example_kset;
bar_kobj.kset = example_kset;

retval = kobject_init_and_add(&foo_kobj, &foo_ktype,
                              NULL, "foo_name");
retval = kobject_init_and_add(&bar_kobj, &bar_ktype,
                              NULL, "bar_name");
```

一旦完成对 `kset` 的操作，可以通过 `kset_unregister()` 释放它，之后当不再使用时，它将被动态地解除分配。以下代码将释放我们示例中的 `kset`：

```
kset_unregister(example_kset);
```

现在我们已经熟悉了 kobjects 和类型结构，让我们来学习如何处理非默认的 sysfs 属性。

## 处理非默认属性

属性是通过 kobjects 导出到用户空间的 sysfs 文件。虽然默认属性大多数时候已经足够，但你可以添加其他属性。一个属性可以是可读的、可写的，或者两者都能从用户空间进行操作。

一个属性定义如下所示：

```
struct attribute {
        char              *name;
        struct module     *owner;
        umode_t           mode;
};
```

在属性数据结构中，`name` 是属性的名称，也就是对应文件条目的名称。`owner` 是属性的拥有者——大多数情况下是 `THIS_MODULE`——而 `mode` 以 **user-group-other**（**ugo**）格式指定此属性的读写权限。

默认属性使用起来非常方便，但灵活性不足。而且，除非通过它们的 `kobj_type` sysfs ops，否则简单的属性不能被读写，这意味着如果属性太多，`show/store` 函数中的分支会变得混乱。为了解决这个问题，kobject 核心提供了一种机制，将每个属性嵌入到一个封装的特殊数据结构中：`struct kobj_attribute`。这个数据结构提供了读取和写入的包装例程。

`struct kobj_attribute`（在 `include/linux/kobject.h` 中定义）如下所示：

```
struct kobj_attribute {
 struct attribute attr;
 ssize_t (*show)(struct kobject *kobj,
                 struct kobj_attribute *attr, char *buf);
 ssize_t (*store)(struct kobject *kobj,
                 struct kobj_attribute *attr,
                 const char *buf, size_t count);
};
```

在这个数据结构中，`attr` 是表示要创建的文件的属性，`show` 是一个指向函数的指针，该函数将在从用户空间读取文件时被调用，`store` 是一个指向函数的指针，该函数将在从用户空间写入文件时被调用。

使用封装的 `kobj_attribute` 结构使开发变得更加通用，并扩展了属性的灵活性。这样，指向 `attr` 的指针会被传递给 `store` 或 `show` 函数，且不仅可以用来确定哪个属性正在被访问，还可以用来获取封装的结构体（即 `kobj_attribute`），从而能够调用该属性的 `show`/`store` 方法。为了做到这一点，您可以使用 `container_of` 宏来获取指向嵌套结构体的指针。

以下是来自 `lib/kobject.c` 的一个摘录，演示了在内核提供的 sysfs 操作元素 `kobj_sysfs_ops` 的 `show` 和 `store` 方法中这种通用机制的应用。这个元素也是 sysfs 操作的数据结构（`kobj_type->sysfs_ops` 元素），它被 `dynamic_kobj_ktype` 使用：

```
static ssize_t kobj_attr_show(struct kobject *kobj,
                     struct attribute *attr, char *buf)
{
   struct kobj_attribute *kattr;
   ssize_t ret = -EIO;
   kattr = container_of(attr, struct kobj_attribute, attr);
   if (kattr->show)
       ret = kattr->show(kobj, kattr, buf);
   return ret;
}
static ssize_t kobj_attr_store(struct kobject *kobj,
                          struct attribute *attr, 
                          const char *buf, size_t count)
{
   struct kobj_attribute *kattr;
   ssize_t ret = -EIO;
   kattr = container_of(attr, struct kobj_attribute, attr);
   if (kattr->store)
       ret = kattr->store(kobj, kattr, buf, count);
   return ret;
}
const struct sysfs_ops kobj_sysfs_ops = {
   .show  = kobj_attr_show,
   .store = kobj_attr_store,
};
```

在前面的代码中，`container_of` 宏完成了所有工作。这也让我们放心，使用这种方法，我们与所有总线、设备、类和驱动相关的 kobject 实现保持兼容，正如我们将在下一节中看到的那样。

让我们回到这些 API。您可能总是会预先知道希望暴露哪些属性；因此，这些属性几乎总是会静态声明。为了帮助这一点，内核提供了 `__ATTR` 宏来初始化 `kobj_attribute`。该宏的定义如下：

```
#define __ATTR(_name, _mode, _show, _store) {         \
    .attr = {.name = __stringify(_name),              \
           .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },\
     .show   = _show,                              \
     .store = _store,                               \
}
```

在前面的宏定义中，`_name` 将被转换为字符串并用作属性名，`_mode` 表示属性的权限，`_show` 和 `_store` 分别是指向属性的 `show` 和 `store` 方法的指针。

以下是两个属性声明的示例，`bar` 和 `foo`（此示例将在本节后面作为基础使用）：

```
static struct kobj_attribute foo_attr =
    __ATTR(foo, 0660, attr_show, attr_store);
static struct kobj_attribute bar_attr =
    __ATTR(bar, 0660, attr_show, attr_store);
```

在前面的示例中，我们有两个权限为 `0660` 的属性。第一个属性名为 `foo`，第二个属性名为 `bar`，这两个属性都使用相同的 `show` 和 `store` 方法。

现在，我们必须创建底层文件。用于在 sysfs 文件系统中添加/删除属性的低级内核 API 分别是 `sysfs_create_file()` 和 `sysfs_remove_file()`，它们的定义如下：

```
int sysfs_create_file(struct kobject * kobj,
                      const struct attribute * attr);
void sysfs_remove_file(struct kobject * kobj,
                        const struct attribute * attr);
```

`sysfs_create_file()` 在成功时返回 0，在失败时返回负错误。`sysfs_remove_file()` 必须给定相同的参数来删除文件属性。

让我们使用这些 API 将 `bar` 和 `foo` 属性添加到系统中：

```
struct kobject *demo_kobj;
int err;
demo_kobj = kobject_create_and_add("demo", kernel_kobj);
if (!demo_kobj) {
    pr_err("demo: demo_kobj registration failed.\n");
    return -ENOMEM;
}
err = sysfs_create_file(demo_kobj, &foo_attr.attr);
if (err)
    pr_err("unable to create foo attribute\n");
err = sysfs_create_file(demo_kobj, &bar_attr.attr);
if (err){
    sysfs_remove_file(demo_kobj, &foo_attr.attr);
    pr_err("unable to create bar attribute\n");
}
```

一旦执行了前面的代码，`bar` 和 `foo` 文件将在 sysfs 中显示，并位于 `/sys/demo` 目录下。在我们的示例中，我们使用了 `__ATTR` 宏来定义我们的属性。我们需要指定名称、模式和 `show`/`store` 方法。内核提供了便捷的宏，用于最常见的情况，以简化属性的指定和代码的编写，使其更加简洁、可读和易于理解。这些宏如下：

+   `__ATTR_RO(name)`：假设 `name_show` 是 `show` 回调函数的名称，并将模式设置为 `0444`。

+   `__ATTR_WO(name)`：假设 `name_store` 是 `store` 函数的名称，并将模式限制为 `0200`，这意味着仅有 root 用户可以写入。

+   `__ATTR_RW(name)`：假设 `name_show` 和 `name_store` 分别是 `show` 和 `store` 回调函数的名称，并将模式设置为 `0644`。

+   `__ATTR_NULL`：它作为列表的结束符使用。它将两个名称都设置为 `NULL`，并作为列表结束的指示符（见 `kernel/workqueue.c`）。

所有这些宏只期望属性名作为参数。这些宏的不同之处在于，与 `__ATTR` 不同，后者的 `store`/`show` 函数名可以是任意的，而这里的属性是在假定 `show` 和 `store` 方法分别命名为 `<attribute_name>_show` 和 `<attribute_name>_store` 的前提下构建的。以下代码通过 `__ATTR_RW_MODE` 宏展示了这一点，该宏的定义如下：

```
#define __ATTR_RW_MODE(_name, _mode) {                 \
    .attr = { .name = __stringify(_name),              \
          .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },\
    .show   = _name##_show,                            \
    .store  = _name##_store,                          \
}
```

如我们所见，`.show` 和 `.store` 字段分别设置为属性名称，后缀分别为 `_show` 和 `_store`。我们来看以下示例：

```
static struct kobj_attribute attr_foo = __ATTR_RW(foo);
```

前面的属性声明假定 `show` 和 `store` 方法分别被定义为 `foo_show` 和 `foo_store`。

注意

如果你需要为所有属性提供一对 `store/show` 操作，你应该使用 `__ATTR` 来定义这些属性。然而，如果处理这些属性需要为每个属性提供一对 `show/store` 操作，你可以使用其他属性定义宏。

以下代码展示了我们之前定义的 `foo` 和 `bar` 属性的 `show/store` 函数的实现：

```
static ssize_t attr_store(struct kobject *kobj,
                      struct kobj_attribute *attr,
                      const char *buf, size_t count)
{
    int value, ret;
    ret = kstrtoint(buf, 10, &value);
    if (ret < 0)
        return ret;
    if (strcmp(attr->attr.name, "foo") == 0)
        foo = value;
    else /* if (strcmp(attr->attr.name, "bar") == 0) */
        bar = value;
    return count;
}
static ssize_t attr_show(struct kobject *kobj,
                      struct kobj_attribute *attr,
                    char *buf)
{
    int value;
    if (strcmp(attr->attr.name, "foo") == 0)
        value = foo;
    else
        value = bar;
     return sprintf(buf, "%d\n", value);
}
```

在前面的代码中，我们没有为每个属性提供一对 `show/store` 操作，而是为所有属性使用了相同的函数对，并通过各自的名称区分这些属性。这是在使用通用的 `kobject_attribute` 而不是框架特定的属性时的常见做法。这是因为它们有时会为每个属性强制要求不同的 `show/store` 函数名称，因为它们不依赖于 `__ATTR` 宏来定义属性。

### 与二进制属性的配合

到目前为止，我们已经熟悉了 sysfs 语句，并提到过属性必须以人类可读的文本格式存储单一的属性/值，并且该属性有一个 `PAGE_SIZE` 限制。然而，可能会出现一些情况，尽管很少见，但需要交换更大的数据并以二进制格式传输，例如，所有数据都支持随机访问。一个典型的例子是设备固件传输，其中用户空间会上传一些二进制数据，这些数据会被推送到硬件或 PCI 设备中，暴露部分或全部的配置地址空间。

为了涵盖这些情况，sysfs 框架提供了二进制属性。请注意，这些属性用于发送/接收内核完全不进行解释或操作的二进制数据。它应*仅*作为与硬件之间的传递通道，内核不会对其进行任何解释。你可以执行的唯一操作是对魔数和大小等进行一些检查。

现在，让我们回到代码中。二进制属性使用 `struct bin_attribute` 来表示，定义如下：

```
struct bin_attribute {
    struct attribute attr;
    size_t     size;
    void             *private;
    ssize_t (*read)(struct file *filp,
              struct kobject *kobj,
              struct bin_attribute *attr,
              char *buffer, loff_t off, size_t count);
    ssize_t (*write)(struct file *filp,
             struct kobject *kobj,
             struct bin_attribute *attr,
             const char *buffer, 
             loff_t off, size_t count);
    int (*mmap)(struct file *filp, struct kobject *kobj,
                  struct bin_attribute *attr,
                  struct vm_area_struct *vma);
};
```

在前面的代码中，`attr` 是该二进制属性的底层经典属性，包含该二进制属性的名称、所有者和权限。`size` 表示二进制属性的最大大小（如果没有最大限制，则为零）。`private` 是一个可以用作任何方便的字段。大多数时候，它被分配为二进制属性的缓冲区。`read()`、`write()` 和 `mmap()` 函数是可选的，其工作方式与正常的 `char` 驱动程序等效。它们的参数中，`filp` 是与该属性相关联的已打开文件指针实例，`kobj` 是与此二进制属性相关联的底层 `kobject`。`buffer` 是分别用于读取或写入操作的输入或输出缓冲区。`off` 是所有类型文件的所有读取或写入方法中都会出现的偏移量参数，它表示从文件开始的偏移量——也就是二进制数据中的偏移量。最后，`count` 是读取或写入的字节数。

注意

尽管二进制属性可能没有大小限制，但较大的数据始终按 `PAGE_SIZE` 块请求/发送。这意味着，例如，对于单次加载，`write()` 函数可能会被多次调用。然而，这种拆分是由内核处理的，因此对驱动程序是透明的。缺点是，sysfs 没有办法标记一系列写操作的结束，因此实现二进制属性的代码必须通过其他方式来解决这个问题。

要创建二进制属性，必须先分配并初始化。与经典属性一样，二进制属性的分配有两种方式——静态分配和动态分配。对于静态分配，框架提供了底层的 `__BIN_ATTR` 宏，其定义如下：

```
#define __BIN_ATTR(_name, _mode, _read, _write, _size) {  \
   .attr = { .name = __stringify(_name), .mode = _mode }, \
   .read    = _read,                        \
   .write   = _write,                       \
    .size      = _size,                          \
```

它的工作原理类似于`__ATTR`宏。在参数方面，`_name`是二进制属性的名称，`_mode`表示其权限，`_read`和`_write`分别是读取和写入函数，`_size`是二进制属性的大小。

与经典属性一样，二进制属性也有自己的高级辅助宏，以简化定义过程。以下是一些此类宏：

```
BIN_ATTR_RO(name, size)
BIN_ATTR_WO(name, size)
BIN_ATTR_RW(name, size)
```

这些宏声明了一个`struct bin_attribute`的实例，其对应的变量命名为`bin_attribute_<name>`，如下所示的`BIN_ATTR`定义：

```
#define BIN_ATTR_RW(_name, _size)         \
struct bin_attribute bin_attr_##_name =   \
                __BIN_ATTR_RW(_name, _size)
```

此外，与经典属性一样，这些高级宏期望读写方法分别命名为`<attribute_name>_read`和`<attribute_name>_write`，如下所示的`__BIN_ATTR_RW`定义：

```
#define __BIN_ATTR_RW(_name, _size) \
 __BIN_ATTR(_name, 0644, _name##_read, _name##_write, \
             _size)
```

对于动态分配，简单的`kzalloc()`就足够了。然而，动态分配的二进制属性必须使用`sysfs_bin_attr_init()`进行初始化，如下所示：

```
void sysfs_bin_attr_init(strict bin_attribute *bin_attr)
```

在此之后，驱动程序必须设置其他属性，如基础属性的模式、名称和权限，此外还可以选择设置读写/映射函数。

与经典属性不同，经典属性可以作为默认属性设置，而二进制属性必须显式创建。可以使用`sysfs_create_bin_file()`来实现，如下所示：

```
int sysfs_create_bin_file(struct kobject *kobj, 
                          struct bin_attribute *attr);
```

该函数在成功时返回`0`，失败时返回负错误。当你完成对二进制属性的操作后，可以使用`sysfs_remove_bin_file()`将其删除，定义如下：

```
int sysfs_remove_bin_file(struct kobject *kobj, 
                          struct bin_attribute *attr);
```

以下是一个摘录（完整版本可以在`drivers/i2c/i2c-slave-eeprom.c`中找到），展示了一个已分配并初始化的动态二进制属性的具体使用示例：

```
struct eeprom_data {
[...]
    struct bin_attribute bin;
    u8 buffer[];
};
static int i2c_slave_eeprom_probe(
                             struct i2c_client *client)
{
    struct eeprom_data *eeprom;
    int ret;
    unsigned int size = FIELD_GET(I2C_SLAVE_BYTELEN,
                                  id->driver_data) + 1;
    eeprom = devm_kzalloc(&client->dev,
                sizeof(struct eeprom_data) + size,
                GFP_KERNEL);
    if (!eeprom)
        return -ENOMEM;
    [...]
    sysfs_bin_attr_init(&eeprom->bin);
    eeprom->bin.attr.name = "slave-eeprom";
    eeprom->bin.attr.mode = S_IRUSR | S_IWUSR;
    eeprom->bin.read = i2c_slave_eeprom_bin_read;
    eeprom->bin.write = i2c_slave_eeprom_bin_write;
    eeprom->bin.size = size;
    ret = sysfs_create_bin_file(&client->dev.kobj,
                                  &eeprom->bin);
    if (ret)
        return ret;
    [...]
    return 0;
};
```

当卸载模块路径或设备断开时，相关的二进制文件会被删除，如下所示：

```
static int i2c_slave_eeprom_remove(struct i2c_client *client)
{
   struct eeprom_data *eeprom = i2c_get_clientdata(client);
   sysfs_remove_bin_file(&client->dev.kobj, &eeprom->bin);
[...]
   return 0;
}
```

然后，在实现读写功能时，可以使用`memcpy()`在数据之间来回移动，如下所示：

```
static ssize_t i2c_slave_eeprom_bin_read(struct file *filp,
          struct kobject *kobj, struct bin_attribute *attr,
          char *buf, loff_t off, size_t count)
{
    struct eeprom_data *eeprom;
    eeprom = dev_get_drvdata(kobj_to_dev(kobj));
[...]
    memcpy(buf, &eeprom->buffer[off], count);
[...]
    return count;
}
static ssize_t i2c_slave_eeprom_bin_write(
         struct file *filp, struct kobject *kobj,
         struct bin_attribute *attr,
         char *buf, loff_t off, size_t count)
{
     struct eeprom_data *eeprom;
     eeprom = dev_get_drvdata(kobj_to_dev(kobj));
[...]
     memcpy(&eeprom->buffer[off], buf, count);
[...]
    return count;
}
```

在前面的摘录中，偏移量（`off` 参数）指示数据应读取/写入的位置，`count` 决定了数据的大小。

### 属性组的概念

到目前为止，我们已经学习了如何通过调用`sysfs_create_file()`或`sysfs_create_bin_file()`函数单独添加（二进制）属性。如果我们需要添加的属性不多，这已经足够，但当属性数量增加时，在添加或删除时可能会变得非常麻烦。驱动程序需要遍历所有属性，逐个创建它们，或者调用`sysfs_create_file()`与属性数量相同的次数。这时，属性组就显得尤为重要。它依赖于`struct attribute_group`结构，定义如下：

```
struct attribute_group {
    const char        *name;
    umode_t           (*is_visible)(struct kobject *,
                         struct attribute *, int);
    umode_t           (*is_bin_visible)(struct kobject *,
                         struct bin_attribute *, int);
    struct attribute  **attrs;
    struct bin_attribute   **bin_attrs;
};
```

如果没有命名，属性组将在定义属性组时将所有属性直接放入 kobject 的目录中。然而，如果提供了 `name`，则会为属性创建一个子目录，并且该目录的名称将是属性组的名称。`is_visible()` 是一个可选的回调函数，用于返回与属性组中特定属性相关的权限。它会为组中的每个（非二进制）属性反复调用。这个回调函数必须返回属性的读/写权限，或者如果该属性根本不应该被访问，则返回 `0`。`is_bin_visible()` 是 `is_visible()` 针对二进制属性的对应函数。返回的值/权限将替代在 `struct attribute` 中定义的静态权限。`attrs` 元素是指向以 `NULL` 结尾的属性列表的指针，而 `bin_attrs` 是二进制属性的对应元素。

用于向文件系统添加/删除属性组的内核函数如下：

```
int sysfs_create_group(struct kobject *kobj,
                       const struct attribute_group *grp)
void sysfs_remove_group(struct kobject * kobj,
                        const struct attribute_group * grp)
```

回到我们的示例，使用标准属性，`bar` 和 `foo` 两个属性可以嵌入到一个 `struct attribute_group` 中。这将允许我们通过一次函数调用将它们添加到系统中，如下所示：

```
static struct kobj_attribute foo_attr =
    __ATTR(foo, 0660, attr_show, attr_store);
static struct kobj_attribute bar_attr =
    __ATTR(bar, 0660, attr_show, attr_store);
/* attrs is aa array of pointers to attributes */
static struct attribute *demo_attrs[] = {
    &bar_foo_attr.attr,
    &bar_attr.attr,
    NULL,
};
static struct attribute_group my_attr_group = {
    .attrs = demo_attrs,
    /*.bin_attrs = demo_bin_attrs,*/
};
```

最后，要一次性创建属性，我们需要使用 `sysfs_create_group()`，如下代码所示：

```
struct kobject *demo_kobj;
int err;
demo_kobj = kobject_create_and_add("demo", kernel_kobj);
if (!demo_kobj) {
    pr_err("demo: demo_kobj registration failed.\n");
    return -ENOMEM;
}
err = sysfs_create_group(demo_kobj, &foo_attr.attr);
```

在这里，我们展示了创建属性组的重要性，以及如何轻松使用它们的 API。虽然到目前为止我们一直在讨论通用内容，但在下一节中，我们将学习如何创建特定框架的属性。

### 创建符号链接

驱动程序可以使用 `sysfs_{create|remove}_link()` 函数在现有的 kobject（目录）上创建/删除符号链接，如下所示：

```
int sysfs_create_link(struct kobject * kobj,
                     struct kobject * target, char * name); 
void sysfs_remove_link(struct kobject * kobj, char * name);
```

这允许一个对象存在于多个位置，甚至创建一个快捷方式。`create` 函数将创建一个名为 `name` 的符号链接，指向远程 `target` kobject 的 sysfs 入口。该链接将被创建在 `kobj` kobject 目录下。一个著名的例子是设备同时出现在 `/sys/bus` 和 `/sys/devices` 中，因为总线控制器首先是一个设备，之后才暴露出一个总线。然而，请注意，创建的任何符号链接都会是持久性的（除非系统重启），即使目标被移除后也是如此。因此，驱动程序必须考虑在关联设备离开系统或模块卸载时的情况。

# sysfs 设备模型概览

sysfs 是一个非持久性的虚拟文件系统，提供系统的全局视图，并通过其 kobjects 显示内核对象层次结构（拓扑）。每个 kobject 都表现为一个目录。这些目录中的文件表示由相关 kobject 导出的内核变量。这些文件称为属性，可以进行读取或写入。

如果任何已注册的 kobject 在 sysfs 中创建了一个目录，目录的创建位置取决于该 kobject 的父对象（它也是一个 kobject，因此突显了内部对象的层次结构）。在 sysfs 中，顶级目录表示对象层次结构的共同祖先或对象所属的子系统。

这些顶级 sysfs 目录可以在`/sys/`目录中找到，如下所示：

```
/sys$ tree -L 1
├── block
├── bus
├── class
├── dev
├── devices
├── firmware
├── fs
├── hypervisor
├── kernel
├── module
└── power
```

`block`包含系统中每个块设备的一个目录。每个目录包含该设备上分区的子目录。`bus`包含系统中已注册的总线。`dev`以原始方式（没有层次结构）包含已注册的设备节点，每个节点是指向`/sys/devices`目录中真实设备的符号链接。`devices`目录展示了系统中设备拓扑的真实视图。`firmware`展示了系统特定的低级子系统树，如 ACPI、EFI 和 OF（设备树）。`fs`列出了系统上使用的文件系统。`kernel`包含内核配置选项和状态信息。最后，`module`是已加载模块的列表，`power`是来自用户空间的系统电源管理控制接口。

每个这些目录都对应一个 kobject，其中一些作为内核符号被导出。它们如下所示：

+   `kernel_kobj`，对应于`/sys/kernel`。

+   `power_kobj`，对应于`/sys/power`。

+   `firmware_kobj`，对应于`/sys/firmware`。它在`drivers/base/firmware.c`源文件中被导出。

+   `hypervisor_kobj`，对应于`/sys/hypervisor`。它在`drivers/base/hypervisor.c`源文件中被导出。

+   `fs_kobj`，对应于`/sys/fs`。它在`fs/namespace.c`源文件中被导出。

其余的`class/`、`dev/`和`devices/`在启动时由内核源中的`devices_init()`函数在`drivers/base/core.c`中创建，`block/`在`block/genhd.c`中创建，`bus/`作为`kset`在`drivers/base/bus.c`中创建。

## 创建与设备、驱动、总线和类相关的属性

到目前为止，我们已经学会了如何创建专用的 kobject 以填充其中的属性。然而，设备、驱动、总线和类框架提供了属性抽象和文件创建，其中创建的属性直接与适当 kobject 目录中的各自框架相关联。

为此，每个框架提供了一个特定于框架的属性数据结构，封装了默认属性并允许我们提供自定义的 show/store 回调。这些数据结构分别是`struct device_attribute`、`struct driver_attribute`、`struct bus_attribute`和`struct class_attribute`，它们分别对应设备、驱动、总线和类框架。它们的定义类似于`kobj_attribute`，但使用不同的名称。让我们看看它们各自的数据结构：

+   设备有以下的属性数据结构：

    ```
    struct driver_attribute {
        struct attribute attr;
        ssize_t (*show)(struct device_driver *driver,
                    char *buf);
        ssize_t (*store)(struct device_driver *driver,
                    const char *buf, size_t count);
    };
    ```

+   类有以下的属性数据结构：

    ```
    struct class_attribute {
        struct attribute attr;
        ssize_t (*show)(struct class *class,
                 struct class_attribute *attr, char *buf);
        ssize_t (*store)(struct class *class,
                 struct class_attribute *attr,
                 const char *buf, size_t count);
    };
    ```

+   总线框架具有以下属性数据结构：

    ```
    struct bus_attribute {
        struct attribute  attr;
        ssize_t (*show)(struct bus_type *bus, char *buf);
        ssize_t (*store)(struct bus_type *bus,
                     const char *buf, size_t count);
    };
    ```

+   设备具有以下属性数据结构：

    ```
    struct device_attribute {
        struct attribute  attr;
        ssize_t (*show)(struct device *dev,
                     struct device_attribute *attr,
                     char *buf);
        ssize_t (*store)(struct device *dev,
                           struct device_attribute *attr,
                           const char *buf, size_t count);
    };
    ```

前述设备特定数据结构的`show`函数需要一个额外的`count`参数，而其他函数则没有。

它们可以通过`kzalloc()`动态分配，并通过设置其内部属性元素的字段并提供适当的回调函数来初始化。然而，每个框架都提供了一组宏，用于静态分配、初始化并分配单个实例的各自属性数据结构。让我们看看这些宏：

+   总线基础设施提供了以下宏：

    ```
    BUS_ATTR_RW(_name)
    BUS_ATTR_RO(_name)
    BUS_ATTR_WO(_name)
    ```

使用这些特定于总线框架的宏，生成的总线属性变量将命名为`bus_attr_<_name>`。例如，`BUS_ATTR_RW(foo)`生成的变量名将是`bus_attr_foo`，类型为`struct bus_attribute`。

+   对于驱动程序，提供了以下宏：

    ```
    DRIVER_ATTR_RW(_name)
    DRIVER_ATTR_RO(_name)
    DRIVER_ATTR_WO(_name)
    ```

这些特定于驱动程序的属性定义宏将使用`driver_attr_<_name>`模式命名生成的变量。因此，`DRIVER_ATTR_RW(foo)`生成的变量将是`struct driver_attribute`类型，命名为`driver_attr_foo`。

+   类框架使用以下宏：

    ```
    CLASS_ATTR_RW(_name)
    CLASS_ATTR_RO(_name)
    CLASS_ATTR_WO(_name)
    ```

使用这些特定于类的宏，生成的变量将是`struct class_atribute`类型，并且将基于`class_attr_<_name>`模式命名。因此，`CLASS_ATTR_RW(foo)`生成的变量名将是`class_attr_foo`。

+   最后，设备特定属性可以通过以下宏静态分配和初始化：

    ```
    DEVICE_ATTR(_name, _mode, _show, _store)
    DEVICE_ATTR_RW(_name)
    DEVICE_ATTR_RO(_name)
    DEVICE_ATTR_WO(_name)
    ```

设备特定属性定义宏使用它们自己的变量名模式，即`dev_attr_<_name>`。因此，例如，`DEVICE_ATTR_RO(foo)`将生成一个名为`dev_attr_foo`的`struct device_attribute`对象。

因为所有这些宏都建立在`__ATTR_RW`、`__ATTR_RO`和`__ATTR_WO`之上，它们静态分配并初始化一个框架特定属性数据结构的单个实例，并假设显示/存储函数分别命名为`<attribute_name>_show`和`<attribute_name>_store`（记住，这是因为它们不依赖于`__ATTR`宏）。`DEVICE_ATTR()`有一个例外，它使用传入的显示/存储函数，不加任何后缀或前缀。这个例外是因为`DEVICE_ATTR`依赖`__ATTR`来定义属性。

正如我们所见，所有这些特定于框架的宏都使用预定义的前缀来命名生成的框架特定属性对象变量。让我们看看以下类属性：

```
static CLASS_ATTR_RW(foo);
```

这将创建一个名为`class_attr_foo`的`struct class_attribute`类型的静态变量，并假定其显示和存储函数分别命名为`foo_show`和`foo_store`。可以在一个组中通过其内部属性元素来引用，如下所示：

```
static struct attribute *fake_class_attrs[] = {
    &class_attr_foo.attr,
    [...]
    NULL,
};
static struct attribute_group fake_attr_group = {
    .attrs = fake_class_attrs,
};
```

创建相应文件时最重要的事情是，驱动程序可以从以下列表中选择适当的 API：

```
int device_create_file(struct device *device,
            const struct device_attribute *entry);
int driver_create_file(struct device_driver *driver,
            const struct driver_attribute *attr);
int bus_create_file(struct bus_type *bus,
            struct bus_attribute *);
int class_create_file(struct class *class,
            const struct class_attribute *attr)
```

在这里，`device`、`driver`、`bus`和`class`参数分别表示设备、驱动、总线和类别实体，属性必须添加到这些实体中。此外，属性将被创建在每个实体的内部 kobject 所对应的目录中，代码如下所示：

```
int device_create_file(struct device *dev,
                   const struct device_attribute *attr)
{
    [...]
    error = sysfs_create_file(&dev->kobj, &attr->attr);
    [...]
}
int class_create_file(struct class *cls,
                    const struct class_attribute *attr)
{
    [...]
    error =
        sysfs_create_file(&cls->p->class_subsys.kobj,
                          &attr->attr);
    return error;
}
int bus_create_file(struct bus_type *bus,
                   struct bus_attribute *attr)
{
    [...]
    error =
        sysfs_create_file(&bus->p->subsys.kobj,
                           &attr->attr);
    [...]
}
```

为了一举两得，前面的代码还展示了`device_create_file()`、`bus_create_file()`、`driver_create_file()`和`class_create_file()`都内部调用了`sysfs_create_file()`。

一旦完成每个属性对象，必须调用适当的移除方法。以下代码展示了可能的选项：

```
void device_remove_file(struct device *device,
             const struct device_attribute *entry);
void driver_remove_file(struct device_driver *driver,
                const struct driver_attribute *attr);
void bus_remove_file(struct bus_type *,
                struct bus_attribute *);
void class_remove_file(struct class *class,
                       const struct class_attribute *attr);
```

这些 API 都期望与创建属性时传递的参数相同。

既然你已经知道了如何调用`kobj_atribute`元素的内部 show/store 函数，那么你应该很清楚如何调用那些框架特定的 show/store 函数了。

让我们看看设备的实现。设备框架有一个内部的`kobj_type`，它实现了设备特定的 show 和 store 函数。这些函数将内部属性元素作为其中一个参数。然后，`container_of`宏会获取封闭数据结构的指针（即框架特定的属性数据结构），从而调用框架特定的 show 和 store 函数。

以下是`drivers/base/core.c`中的一段代码，展示了设备特定的`sysfs_ops`实现：

```
static ssize_t dev_attr_show(struct kobject *kobj,
                            struct attribute *attr,
                            char *buf)
{
    struct device_attribute *dev_attr = to_dev_attr(attr);
    struct device *dev = kobj_to_dev(kobj);
    ssize_t ret = -EIO;
    if (dev_attr->show)
          ret = dev_attr->show(dev, dev_attr, buf);
    if (ret >= (ssize_t)PAGE_SIZE) {
        print_symbol("dev_attr_show:
                        %s returned bad count\n",
                    (unsigned long)dev_attr->show);
    }
    return ret;
}
static ssize_t dev_attr_store(struct kobject *kobj,
                      struct attribute *attr,
                      const char *buf, size_t count)
{
    struct device_attribute *dev_attr = to_dev_attr(attr);
    struct device *dev = kobj_to_dev(kobj);
    ssize_t ret = -EIO;
    if (dev_attr->store)
        ret = dev_attr->store(dev, dev_attr, buf, count);
    return ret;
}
static const struct sysfs_ops dev_sysfs_ops = {
    .show     = dev_attr_show,
    .store    = dev_attr_store,
};
```

注意，在前面的代码中，`to_dev_attr()`，即使用`container_of`的宏，定义如下：

```
#define to_dev_attr(_attr) \
       container_of(_attr, struct device_attribute, attr)
```

总线（在`drivers/base/bus.c`中）、驱动（在`drivers/base/bus.c`中）和类别（在`drivers/base/class.c`中）属性的原理是相同的。

## 使 sysfs 属性兼容轮询和选择

尽管这不是处理 sysfs 属性的必需条件，但这里的主要思想是允许在给定属性上使用`poll()`或`select()`系统调用，来被动地等待变化。这种变化可以是固件变得可用、警报通知，或属性值发生变化。当用户在文件上休眠等待变化时，驱动程序必须调用`sysfs_notify()`来唤醒任何休眠的用户。

这个通知 API 的定义如下：

```
void sysfs_notify(struct kobject *kobj, const char *dir,
                  const char *attr)
```

如果`dir`参数不是`NULL`，则用于在`kobj`的目录中查找包含属性的子目录（假设是由`sysfs_create_group`创建的）。此调用将使任何轮询进程唤醒并处理事件（可能是读取新值、处理警报等）。

注意

如果没有此功能调用，则不会有任何通知；因此，任何轮询进程将最终无限期地等待（除非在系统调用中指定了超时）。

下面的代码展示了属性的`store()`函数，本书提供了该代码：

```
static ssize_t store(struct kobject *kobj,
                     struct attribute *attr,
                     const char *buf, size_t len)
{
    struct d_attr *da = container_of(attr, struct d_attr, 
                                      attr);
    sscanf(buf, "%d", &da->value);
    pr_info("sysfs_foo store %s = %d\n",
             a->attr.name, a->value);
    if (strcmp(a->attr.name, "foo") == 0){
        foo.value = a->value;
        sysfs_notify(mykobj, NULL, "foo");
    }
    else if(strcmp(a->attr.name, "bar") == 0){
        bar.value = a->value;
        sysfs_notify(mykobj, NULL, "bar");
    }
    return sizeof(int);
}
```

在前面的代码中，一旦值更新，调用`sysfs_notify()`是有意义的，这样用户代码就可以读取准确的值。

用户代码可以直接将已打开的属性文件传递给`poll()`或`select()`，而不必读取该属性的初始内容。这是开发人员的方便做法。然而，请注意，在通知时，`poll()`返回`POLLERR|POLLPRI`（这些是标志，用户在调用`poll()`时必须请求），而`select()`则返回文件描述符，指示它是等待读取、写入还是异常事件。

# 总结

完成本章后，您应该熟悉 LDM 及其数据结构（总线、类、设备和驱动程序），以及其低级数据结构，包括`kobject`、`kset`、`kobj_type`和`attributes`（或这些的组合）。您现在应该知道内核内对象是如何表示的（设备拓扑），并能够创建一个属性（或属性组），通过 sysfs 暴露设备或驱动程序的特性和属性。

在下一章中，我们将介绍**IIO**（**工业 I/O**）框架，它大量使用 sysfs 的功能。
