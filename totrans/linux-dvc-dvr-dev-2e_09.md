# *Chapter 7*: Understanding the Concept of Platform Devices and Drivers

The Linux kernel handles devices by using the concept of buses, that is, the links between the CPU and these devices. Some buses are smart enough and embed a discoverability logic to enumerate devices sitting on them. With such buses, early in the bootup phase, the Linux kernel requests these buses for the devices they have enumerated as well as the resources (such as interrupt lines and memory regions) they need to work correctly. PCI, USB, and SATA buses all come under this family of discoverable buses.

Unfortunately, the reality is not always so beautiful. There are a number of devices that the CPU is still unable to detect. Most of these non-discoverable devices are on-chip, although some of them sit on slow or dumb buses that do not support device discoverability.

As a result, the kernel must provide mechanisms for receiving information about the hardware and users must inform the kernel where these devices can be found. In the Linux kernel, these non-discoverable devices are known as **platform devices**. Because they do not sit on known buses such as I2C, SPI, or any non-discoverable bus, the Linux kernel has implemented the concept of a **platform bus** (also referred to as a **pseudo platform bus**) in order to maintain the paradigm according to which devices are always connected to CPUs through buses.

In this chapter, we will learn how and where to instantiate platform devices as well as their resources, and learn how to write their drivers, that is, platform drivers. To achieve that, the chapter will be split into the following topics:

*   Understanding the platform core abstraction in the Linux kernel
*   Dealing with platform devices
*   Platform driver abstraction and architecture
*   Example of writing a platform driver from scratch

# Understanding the platform core abstraction in the Linux kernel

To cover the long list of non-discoverable devices that are being increasingly used as **System on Chips** (**SoCs**) are becoming more and more popular, the platform core has been introduced. Within this framework, the three most important data structures are as follows: the one representing the platform device, another representing its resource, and the final data structure, representing the platform driver.

A platform device is represented in the kernel as an instance of `struct platform_device`, defined in `<linux/platform_device.h>` as follows:

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

In the previous data structure, `name` is the platform device name. The name assigned to the platform device must be chosen with care. Platform devices are matched against their respective drivers in the pseudo platform bus matching function, that is, `platform_match()`. In this function, under certain circumstances (no device tree or ACPI support and no `id` table match), matching falls back to name matching, comparing the driver's name and the platform device's name.

`dev` is the underlying device structure for the Linux device model, and `id` is used to extend this device name. The following describes how `id` is used:

*   Where `id` is `-1` (which corresponds to the `PLATFORM_DEVID_NONE` macro), the underlying device name will be the same as the platform device name. The platform core will do the following:

    ```
    dev_set_name(&pdev->dev, "%s", pdev->name);
    ```

*   Where `id` is `-2` (which corresponds to the `PLATFORM_DEVID_AUTO` macro), the kernel will automatically generate a valid ID and will name the underlying device as follows:

    ```
    dev_set_name(&pdev->dev, "%s.%d.auto", pdev->name,
                 <auto_id>);
    ```

*   In any other case, `id` will be used as follows:

    ```
    dev_set_name(&pdev->dev, "%s.%d", pdev->name,
                 pdev->id);
    ```

`resource` is an array of resources assigned to the platform device, and `num_resources` is the number of elements in that array.

In the case of a platform device and driver matching by ID table, `pdev->id_entry`, which is of the `struct platform_device_id` type, will point to the matching ID table entry that caused the platform driver to match with this platform device.

Regardless of how platform devices have been registered, they need to be driven by appropriate drivers, that is, platform drivers. Such drivers must implement a set of callbacks that are used by the platform core as and when devices appear/disappear on the platform bus.

Platform drivers are represented in the Linux kernel as an instance of `struct platform_driver`, defined as follows:

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

The following describes the elements used in this data structure:

*   `probe()`: This is the function that gets called when a device claims your driver after a match occurs. Later, we will see how `probe` is called by the core. Its declaration is as follows:

    ```
    int my_pdrv_probe(struct platform_device *pdev)
    ```

The kernel is responsible for providing the `platform_device` parameter. `probe` is called by the bus driver when the device driver is registered in the kernel.

*   `remove()`: This is called to get rid of the driver when it is no longer needed by devices and its declaration looks like the following:

    ```
    static void my_pdrv_remove(struct platform_device *pdev)
    ```

*   `driver` is the underlying driver structure for the device model and must be provided with a name (which must be chosen with care as well), an owner, and some other fields (such as a device tree matching table), which we will see later. When it comes to the platform driver, before the driver and device match, the `platform_device.name` and `platform_driver.driver.name` fields must be identical.
*   `id_table` is one of the ways provided by the platform driver to the bus code to bind actual devices to the driver. The other way is via the device tree, which will be discussed in the *Provisioning supported devices in the driver* section.

Now that we have introduced both platform device and platform driver data structures, let's move beyond this and try to understand how they are created and registered with the system.

# Dealing with platform devices

Before we start writing platform drivers, this section will teach you how and where to instantiate platform devices. Only after that will we delve into the platform driver implementation. Platform device instantiation in the Linux kernel has existed for a long time and has been improved all over the kernel versions, and we will discuss the specificities of each instantiation method in this section.

## Allocating and registering platform devices

Since there is no way for platform devices to make themselves known to the system, they must be populated manually and registered with the system along with their resources and private data. In the early platform core days, platform devices were declared in the board file, `arch/arm/mach-*` (which is `arch/arm/mach-imx/mach-imx6q.c` for i.MX6), and made known to the kernel with `platform_device_add()` or `platform_device_register()`, depending on how each platform device has been allocated.

This leads us to conclude that there are two methods for allocating and registering platform devices.

A static method, which entails enumerating platform devices in the code and calling `platform_device_register()` on each of them for their registration. This API is defined as follows:

```
int platform_device_register(struct platform_device *pdev)
```

One example of such usage is the following:

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

With static registration, platform devices are statically initialized and passed to `platform_device_register()`. This method also allows for the addition of bulk platform devices. The corresponding code can use `platform_add_devices()`, which accepts an array of pointers to platform devices and the number of elements in the array. This function is defined as follows:

```
int platform_add_devices(struct platform_device **devs,
                          int num);
```

On the other hand, dynamic allocation requires an initial call to `platform_device_alloc()` to dynamically allocate and initialize the platform device, defined as follows:

```
struct platform_device *platform_device_alloc(
                          const char *name, int id);
```

In parameters, `name` is the base name of the device we're adding, and `id` is the instance ID of the platform device. In the event of success, this function returns a valid platform device object, or `NULL` on error.

Platform devices allocated in this way are registered with the system using the `platform_device_add()` API exclusively, defined as follows:

```
int platform_device_add(struct platform_device *pdev);
```

As an only parameter, the platform device has been allocated using `platform_device_alloc()`. In the event of success, this function returns `0`, or a negative error code in the event of failure.

If platform device registration with `platform_device_add()` fails, you should release the memory occupied by the platform device structure using `platform_device_put()`, defined as follows:

```
void platform_device_put(struct platform_device *pdev);
```

The following is an example of how it is used:

```
status = platform_device_add(evm_led_dev);
if (status < 0) {
     platform_device_put(evm_led_dev);
[...]
}
```

That said, whatever way the platform device has been allocated and registered, it must be unregistered with `platform_device_unregister()`, defined as follows:

```
void platform_device_unregister(struct platform_device *pdev)
```

Do note that `platform_device_unregister()` internally calls `platform_device_put()`.

Now that we know how to instantiate platform devices in the traditional way, let's see how minimalistic it could be to achieve the same goal by adopting another approach, the device tree.

## How not to allocate platform devices to your code

As we said earlier, platform devices used to be populated either in board files or from other drivers. There was no flexibility in using this method. Adding/removing platform devices would require, in the best case, recompiling a module, and, in the worst case, recompiling the whole kernel. This deprecated method is not portable either.

Nowadays, there is the device tree, which has existed for quite a long time now. This mechanism is used to declare non-discoverable devices present in the system. As a standalone entity, it can be built independently from the kernel or other modules. It turns out that this could be the best alternative for populating platform devices.

This is achieved by declaring these platform devices as nodes in the device tree, under a node whose compatible string property is `simple-bus`. This compatible string means the node is regarded as a bus that requires no specific handling or driver. Moreover, it indicates that there is no way to dynamically probe the bus and that the only way to locate the child devices of this bus is by using the address information in the device tree. Then Linux, in its implementation, ends up creating a platform device for each first-level sub-node under the node that has `simple-bus` in the `compatible` property.

The following is a demonstration:

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

In the preceding example, only the `bar` and `foz` nodes will be registered as platform devices. `baz` will not (its direct parent does not have `simple-bus` in a compatible string). Thus, if there is any platform driver with `company, product` and/or `labcsmart,something` in their compatible matching table, then these platform devices will be probed.

The common use is under the SoC node or on-chip memory mapped buses to declare on-chip devices. Another frequent usage is to declare regulator devices, as follows:

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

In the preceding example, two platform devices will be registered, each corresponding to a fixed regulator.

## Working with platform resources

At the opposite end of hot-pluggable devices that are enumerated, and which advertise resources they need, the kernel has no idea of what platform devices are present on your system, what they are capable of, or what they need in order to work properly. There is no auto-negotiation process, so any information provided to the kernel about the resources required by a given platform device would be welcome. Such resources could be IRQ lines, DMA channels, memory regions, I/O ports, and so on.

From within the platform code, a resource is represented as an instance of a struct resource, defined in `include/linux/ioport.h` as follows:

```
struct resource {
     resource_size_t start;
     resource_size_t end;
     const char *name;
     unsigned long flags;
};
```

In the preceding data structure, `start`/`end` point to the beginning/end of the resource. They indicate where I/O or memory regions begin and terminate. Because IRQ lines, buses, and DMA channels do not have ranges, it is usual to assign the same value to `start`/`end`.

`flags` is a mask that characterizes the type of resource, for example, `IORESOURCE_BUS`. Possible values are as follows:

*   `IORESOURCE_IO` for PCI/ISA I/O ports
*   `IORESOURCE_MEM` for memory regions
*   `IORESOURCE_REG` for register offsets
*   `IORESOURCE_IRQ` for IRQ lines
*   `IORESOURCE_DMA` for DMA channels
*   `IORESOURCE_BUS` for a bus

Finally, the `name` element identifies or describes the resource, since the resource can be extracted by name.

Assigning such resources to a platform device can be done in two ways, first, in the same compilation unit where the platform device has been declared and registered, and second, from the device tree.

In this section, we have described the resource and shown their usage. Let's now see how they can be fed to the platform device in the next section.

### Platform resource provisioning – the old and deprecated way

This method is to be used with kernels that do not support the device tree or when such a choice does not exist. It is mainly used with a **multi-function device** (**MFD**), where a master (the enclosing) chip shares its resources with subdevices.

With this method, resources are provided in the same way as platform devices. The following is an example:

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

The preceding excerpt shows three resources (two of the memory type and one of the IRQ type) whose types can be identified with `IORESOURCE_IRQ` and `IORESOURCE_MEM`. The first one is a 4 KB memory region, while the second is also a memory region whose range is defined by a macro, and finally, the IRQ 90\.

Assigning these resources to a platform device is a straightforward operation. If the platform device has been allocated statically, the resources should be assigned in the following manner:

```
static struct platform_device foo_pdev = {
     .name = "foo-device",
     .resource             = foo_resources,
     .num_ressources = ARRY_SIZE(foo_resources),
[...]
};
```

For a dynamically allocated platform device, this is done from within a function, and it should be something like the following:

```
struct platform_device *foo_pdev;
[...]
my_pdev = platform_device_alloc("foo-device", ...);
if (!my_pdev)
           return -ENOMEM;
my_pdev->resource = foo_resources;
my_pdev->num_ressources = ARRY_SIZE(foo_resources);
```

There are several helper functions for getting data out of the resource array; these include the following:

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

The `n` parameter says which resource of that type is desired, with zero indicating the first one. Thus, for example, a driver could find its second MMIO region with the following:

```
r = platform_get_resource(pdev, IORESOURCE_MEM, 1);
```

The use of this function is explained in the *Handling resources* section of [*Chapter 5*](B17934_05_Epub.xhtml#_idTextAnchor082), *Understanding and Leveraging the Device Tree*.

### Understanding the concept of platform data

The struct resource that we have used so far is adequate to instantiate resources for a simple platform device, but many devices are more complex than that. This data structure can only encode a limited number of types of information. As an extension to that, `platform_device.device.platform_data` is used to assign any other extra information to the platform device.

Data can be enclosed in a bigger structure and assigned to this extra field. This data can be of whatever type the driver can understand, but most of the time, they are other data types that are not a part of the resource types we enumerated in the preceding section. These can be regulator constraints or even pointers to per-device functions, for example.

Let's describe, in the following code block, extra platform data that corresponds to a set of pointers to functions that are private to the platform device type:

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

For a statically allocated platform device, we would execute the following command:

```
static struct platform_device my_device = {
     .name = "foo-pdev",
     .dev  = {
          .platform_data   = (void*)&trizeps_pcmcia_ops,
     },
     [...]
};
```

If the platform device was found to be allocated dynamically, it would be assigned using the `platform_device_add_data()` helper, defined as follows:

```
int platform_device_add_data(struct platform_device *pdev,
                        const void *data, size_t size);
```

The preceding function returns `0` in the event of success. `data` is the data to use as platform data, and `size` is the size of this data.

Back to our example of a set of functions, we would do the following:

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

From within the platform driver, the platform device will have its `pdev->dev.platform_dat` element pointing to the platform data. Although we would dereference this field, it is recommended to use the kernel-provided function, `dev_get_platdata()`, defined as follows:

```
void *dev_get_platdata(const struct device *dev)
```

Then, to get back the enclosing set of function structures, the driver could do the following:

```
struct foo_low_level *my_ops =
          dev_get_platdata(&pdev->dev);
```

There is no way for the driver to check the type of data that is passed. Drivers must simply assume that they have been supplied a structure of the expected type because the platform data interface lacks any sort of type checking.

### Platform resource provisioning – the new and recommended way

The first resource provisioning method has a few drawbacks, including the fact that any change would require a rebuilding of either the kernel or the module that has changed and this could increase the kernel size.

With the arrival of the device tree, things have become simpler. To keep things compatible, memory regions, interrupts, and DMA resources specified in the device tree are converted into an instance of struct resources by the platform core so that `platform_get_resource()`, `platform_get_resource_by_name()`, or even `platform_get_irq()` can return the appropriate resource, irrespective of whether this resource has been populated the traditional way or from the device tree. This can be verified in the *Handling resources* section of [*Chapter 5*](B17934_05_Epub.xhtml#_idTextAnchor082), *Understanding and Leveraging the Device Tree*.

Needless to say, the device tree allows passing any type of information that the driver may need to know. Such data types can be used to pass any device-/driver-specific data. This can be accomplished by reading the *Extracting application-specific data* section, still in [*Chapter 5*](B17934_05_Epub.xhtml#_idTextAnchor082), *Understanding and Leveraging the Device Tree*.

However, the device tree code is unaware of the specific structure used by a given driver for its platform data, so it will be unable to provide that information in that form. To pass such extra data, drivers can use the `.data` field in the `of_device_id` entry that caused the platform device and the driver to match. This field could then point to the platform data. See the *Provisioning supported devices in the driver* section in this chapter.

Drivers expecting platform data in the traditional way should check the `platform_device->dev.platform_data` pointer. If there is a non-null value there, this means that the device was instantiated in the traditional way along with the platform data and the device tree was not used; the platform data should be used as usual. If, however, the device has been instantiated from the device tree code, the `platform_data` pointer will be `NULL`, indicating that the information must be acquired from the device tree directly. In this case, the driver will find a `device_node` pointer in the platform device's `dev.of_node` field. The various device tree access routines (most notably `of_get_property()`) can then be used to pull the required data from the device tree.

Now that we are familiar with platform resource provisioning, let's learn how to design a platform device driver.

# Platform driver abstraction and architecture

Let's get warned before going further. Not all platform devices are handled by platform drivers (or, should I say, pseudo platform drivers). Platform drivers are dedicated to devices not based on conventional buses. I2C devices or SPI devices are platform devices, but rely on I2C or SPI buses, respectively, and not on platform buses. Everything needs to be done manually with the platform driver.

## Probing and releasing the platform devices

The platform driver entry point is the probe method, invoked after a match with a platform device has occurred. This probe method has the following prototype:

```
int pdrv_probe(struct platform_device *pdev)
```

`pdev` corresponds to the platform device that has been instantiated in the traditional way or a fresh one allocated by the platform core because of the associated device tree node having a direct parent with `simple-bus` in its compatible property. Platform data and resources, if any, will also be set accordingly. If the platform device in the parameter is the one expected by the driver, this method must return `0`. Otherwise, an appropriate negative error code must be returned.

Whether the platform device has been instantiated the traditional way or from the device tree, its resources can be extracted using conventional platform core APIs, `platform_get_resource()`, `platform_get_resource_by_name()`, or even `platform_get_irq()`, and similar APIs.

The probe method must request any resource (GPIOs, clocks, IIO channels, and so on) and data required by the driver. If mappings need to be done, the probe method is the appropriate place to do that.

Everything that has been done in the `probe` method must be undone when the device leaves the system or when the platform driver is unregistered. The `remove` method is the appropriate place to achieve that. It must have the following prototype:

```
int pdrv_remove(struct platform_device *dev)
```

This function must return `0` only if everything has been undone and cleaned up. Otherwise, an appropriate error code must be returned so that users can be informed. In the parameter, the same platform device that has been passed to the `probe` function.

In the *Example of writing a platform driver from scratch* section, we discuss all the tips and particularities of implementing the `probe` and `remove` methods.

Now that the driver callbacks are ready, we can populate the devices that are going to be handled by this driver.

## Provisioning supported devices in the driver

Our platform driver is useless on its own. To make it useful to devices, it must inform the kernel what devices it can manage. To achieve that, an ID table must be provided, assigned to the `platform_driver.id_table` field. This will allow platform device matching. However, to extend this feature to module autoloading, the same table must be given to `MODULE_DEVICE_TABLE` so that module aliases are generated.

Back to the code, each entry in that table is of the `struct platform_device_id` type, defined as follows:

```
struct platform_device_id {
     char name[PLATFORM_NAME_SIZE];
     kernel_ulong_t driver_data;
};
```

In the preceding data structure, `name` is a descriptive name for the device, and `driver_data` is the driver state value. It can be set with a pointer to a per-device data structure. The following is an example, an excerpt from `drivers/mmc/host/mxs-mmc.c`:

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

When `mxs_ssp_ids` is assigned to the `platform_driver.id_table` field, platform devices will be able to match this driver based on their name matching any `platform_device_id.name` entry. `platform_device.id_entry` will point to the entry in this table that triggered the match.

To allow matching platform devices declared in the device tree using their compatible string, the platform driver must set `platform_driver.driver.of_match_table` with a list of elements of `struct of_device_id`; then, to allow module autoloading from device tree matching as well, this device tree match table must be given to `MODULE_DEVICE_TABLE`. The following is an example:

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

If `of_device_id` is set in the driver, the matching is judged by any `of_device_id.compatible` element matching the value of the compatible property in the device node. To get the `of_device_id` entry that caused the match, the driver should call `of_match_device()`, passing as parameters the device tree match table and the underlying device structure, `platform_device.dev`.

The following is an example:

```
static int mxs_mmc_probe(struct platform_device *pdev)
{
    const struct of_device_id *of_id =
        of_match_device(mxs_mmc_dt_ids, &pdev->dev);
    struct device_node *np = pdev->dev.of_node;
[...]
}
```

After these match tables have been defined, they can be assigned to the platform driver structure as follows:

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

With the preceding final platform driver data structure, we give devices a chance to match the platform driver based on either the device tree match table, or on the ID table, and finally, based on the driver's name.

## Driver initialization and registration

Registering a platform driver with the kernel is as simple as calling `platform_driver_register()` or `platform_driver_probe()` in the module initialization function. Then, to get rid of a platform driver that has been registered, the module must call `platform_driver_unregister()` to unregister this driver.

The following are the respective prototypes of these functions:

```
int platform_driver_register(struct platform_driver *drv);
void platform_driver_unregister(struct platform_driver *);
int platform_driver_probe(struct platform_driver *drv,
                  int (*probe)(struct platform_device *))
```

The difference between the two probing functions is as follows:

*   `platform_driver_register()` registers and puts the driver into a list of drivers maintained by the kernel, meaning that its `probe()` function can be called on demand whenever a new match with a platform device occurs. To prevent the driver from being inserted and registered in that list, the next function can be used. That said, any platform driver registered with `platform_driver_register()` must be unregistered with `platform_driver_unregister()`.
*   With `platform_driver_probe()`, the kernel immediately runs the matching loop to check whether there are platform devices that can be matched against this platform driver and will call the `probe` method for each device with a match. If no device is found at that time, the platform driver is simply ignored. This method prevents a deferred probe since it does not register the driver on the system. Here, the `probe` function is placed in an `__init` section, which is freed (provided the driver is compiled statically, that is, built-in) when the kernel boot has completed, thereby preventing a deferred probe, and reducing the driver's memory footprint. Use this method if you are 100% sure the device is present in the system:

    ```
    ret = platform_driver_probe(&mypdrv, my_pdrv_probe);
    ```

The `probe()` procedure can be placed in an `__init` section if the device is known not to be hot-pluggable.

The appropriate time to register the platform driver is right after the module gets loaded, which allows us to say that the appropriate place to do that is from the module initialization function. The same goes for unregistering the driver, which must be done on the unloading path of the module. Thanks to that, we can say that the `module_exit` function is the appropriate place to unregister a platform driver.

The following is a typical demonstration of simple platform driver registering with the platform core:

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

Our module does nothing else in the initialization/exit function apart from registering/unregistering with the platform driver with the platform bus core. This is a frequent situation, where the module init/exit methods are lite and dummy. In this case, we can get rid of `module_init()` and `module_exit()`, and use the `module_platform_driver()` macro, as follows:

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

Although we have only inserted snapshot code in the preceding example, this is enough to cover the demonstration of reducing the code and making it cleaner.

To make sure we are on the same knowledge base, let's summarize all the concepts we have learned in this chapter into a real platform driver.

# Example of writing a platform driver from scratch

This section will try to summarize as far as possible the knowledge that has been acquired so far throughout the chapter. Now, let's imagine a platform device that is memory-mapped and that the memory range through which it can be controlled starts at `0x02008000` and is `0x4000` in size. Then, let's say this platform device can interrupt the CPU upon completion of its jobs and that this interrupt line number is `31`. To keep things simple, let's not require any other resource for this device (we could have imagined clocks, DMAs, regulators, and so on).

To start with, let's instantiate this platform device from the device tree. If you remember, for a node to be registered as a platform device by the platform core, the direct parent of this node must have `simple-bus` in its compatible string list, and this is what is implemented here:

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

If we had to instantiate this platform device along with its resources in the traditional way, we would have to do something like the following, in a file that we will call `demo-pdev-init.c`:

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

In the preceding, we first defined the platform resources. Now we can instantiate the platform device and assign these resources as follows, still in `demo-pdev-init.c`:

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

In the preceding, we commented out the `dev`-related assignment just to show that we could have defined a bigger structure with any other information expected by the driver and passed this data structure as platform data.

Now that all the data structures are set up, we can register the platform device by adding the following content at the bottom of `demo-pdev-init.c`:

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

The preceding code does nothing other than register the platform device with the system. In this example, we have used static initialization. That said, things won't be that much different with dynamic initialization.

Now that we are done with platform device instantiation, let's focus on the platform driver itself, whose compilation unit is named `demo-pdriver.c`, and add the following header files to it:

```
#include <linux/module.h>
#include <linux/init.h>
#include <linux/interrupt.h>
#include <linux/of_platform.h>
#include <linux/platform_device.h>
```

Now that the headers that will allow us to support the necessary APIs and to use the appropriate data structures are in place, let's start by enumerating the devices that can be matched with this platform driver from the device tree:

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

In the preceding, the device tree match table enumerates the supported devices that are differentiated by their `compatible` property. Here, again, the `.data` assignment is commented out just to show how we could have passed platform-specific data according to the entry that caused the match with the platform device. This platform-specific structure could have been `big_struct_1` or `big_struct_2`. This is the way I recommend when you need to pass platform data and still want to use the device tree.

To provide an opportunity for matching using an ID table, we populate such a table as follows:

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

The preceding requires no special comments, except the commented-out part, which shows how extra data could be used depending on the device being handled.

At this stage, we are good to go with the implementation of the probe method. Still in our demo platform driver compilation unit, we add the following:

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

In the preceding probe method, a lot of things could have been added or done differently. The first case is if our driver was device tree-compliant only, and that the associated platform device was instantiated from the device tree as well. We could have done something like this:

```
struct device_node *np = pdev->dev.of_node;
const struct of_device_id *of_id =
        of_match_device(labcsmart_dt_ids, &pdev->dev);
struct big_struct *pdata_struct = (big_struct*)of_id->data;
```

In the preceding, we grab a reference to the device tree note associated with the platform device, on which we could use whatever device tree-related APIs (such as `of_get_property()`) to extract data from there. Next, in the case where a platform-specific data structure was passed when provisioning the supported device in the match table, we use `of_match_device()` to point to the entry that corresponds to the hit and we extract the platform-specific data.

If the match occurred thanks to the ID table, `pdev->id_entry` would point to the entry thanks to which the match occurred, and `pdev->id_entry->driver_data` would point to the appropriate bigger structure.

However, if the platform data were declared in the traditional method, we would use something as follows:

```
struct big_struct *pdata_struct =
                 (big_struct*)dev_get_platdata(&pdev->dev);
```

Now that we are done with the probe method, I would like to draw your attention to a particular point: in that method, we exclusively used the `devm_` prefixed function to deal with resources. These are functions that take care of releasing the resource at the appropriate time.

What this means is that we don't require a `remove` method. However, for the sake of pedagogy, the following is what our remove method would look like if we did not use resource-managed helpers:

```
int demo_pdrv_remove(struct platform_device *dev)
{
    free_irq(res_irq->start, NULL);
    iounmap(reg_base);
    return 0;
}
```

In the preceding, the IRQ resource is released, and the memory mapping is destroyed.

Well now, everything is in place. We can initialize and register the platform driver using the following:

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

There is another important point you need to consider: the names of the platform device, platform driver, and even the device node name in the device tree. They are all the same, that is, `demo_pdev`. This is a way to provide an opportunity to match even by platform device and platform driver names since matching by name is used as a fallback when all of the device tree, ID table, and ACPI matchings fail.

# Summary

The kernel pseudo platform bus no longer holds any secrets for you. With a bus matching mechanism, you can understand how, when, and why your driver has been loaded, as well as which device it was written for. We can implement any probe function, based on the matching mechanism we want. Since the main purpose of a device driver is to handle devices, we are now able to populate devices in the system, either in the traditional way or from the device tree. To finish in style, we implemented a functional platform driver example from scratch.

In the next chapter, we will continue learning how to write drivers for non-discoverable devices, but sitting on I2C buses this time.