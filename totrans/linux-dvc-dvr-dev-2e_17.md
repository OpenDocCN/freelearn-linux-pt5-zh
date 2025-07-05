# *Chapter 14*: Introduction to the Linux Device Model

Until version 2.5, the Linux kernel had no way to describe and manage objects, and its code reusability was not as enhanced as it is now. In other words, there was no device topology, nor organization as we know it is in sysfs nowadays. There was no information on subsystem relationships, nor on how the system is put together. Then came the **Linux Device Model** (**LDM**), which introduced the following features:

*   The concept of classes. They are used to group devices of the same type or that expose the same functionalities (for example, mice and keyboards are both input devices).
*   Communication with the user space through a virtual filesystem, allowing you to manage and enumerate devices and the properties they expose from user space.
*   Object life cycle management using reference counting.
*   A power management facility, allowing you to handle the order in which devices should shut down.
*   The reusability of the code. Classes and frameworks expose interfaces, behaving like a contract that any driver that registers with them must respect.
*   An **object-oriented** (**OO**)-like programming style and encapsulation in the kernel.

In this chapter, we will take advantage of LDM and export some properties to the user space through the sysfs filesystem. To do this, we will cover the following topics:

*   Introduction to LDM data structures
*   Getting deeper inside LDM
*   Overview of the device model from sysfs

# Introduction to LDM data structures

The Linux device model introduced device hierarchy. It is built on top of a few data structures. Among these is the bus, which is represented in the kernel as an instance of `struct bus_type`; the device driver, which is represented by a `struct device_driver` structure; and the device, which is the last element and is represented as an instance of the `struct device` structure. In this section, we will introduce all those structures and learn how they interact each with other.

## The bus data structure

A bus is a channel link between devices and the processor. The hardware entity that manages the bus and exports its protocol to devices is called the bus controller. For example, the USB controller provides USB support, while the I2C controller provides I2C bus support. However, the bus controller, being a device on its own, must be registered like any device. It will be the parent of the devices that need to sit on this bus. In other words, every device sitting on the bus must have its parent field pointing to the bus device. A bus is represented in the kernel by the `struct bus_type` structure, which has the following definition:

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

Only the elements that are relevant to this book have been listed here; let's take a look at them in more detail:

*   `match` is a callback that's invoked whenever a new device or driver is added to this bus. The callback must be smart enough and should return a nonzero value when there is a match between a device and a driver, both given as parameters. The main purpose of the `match` callback is to allow a bus to determine if a particular device can be handled by a given driver or the other logic if the given driver supports a given device. Most of the time, this verification is done by a simple string comparison (device and driver name, whether it's a table and device tree-compatible property, and so on). For enumerated devices (PCI, USB), verification is done by comparing the device IDs that are supported by the driver with the device ID of the given device, without sacrificing bus-specific functionality.
*   `probe` is a callback that's invoked when a new device or driver is added to this bus after the match has occurred. This function is responsible for allocating the specific bus device structure and calling the given driver's `probe` function, which is supposed to manage the device (allocated earlier).
*   `remove` is called when a device leaves this bus.
*   `suspend` is a method that's called when a device on the bus needs to be put into sleep mode.
*   `resume` is called when a device on the bus must be brought out of sleep mode.
*   `pm` is the set of power management operations for this bus, which may also call driver-specific power management operations.
*   `drv_groups` is a pointer to a list (array) of `struct attribute_group` elements, each of which has a pointer to a list (array) of `struct attribute` elements. It represents the default attributes of the device drivers on the bus. The attributes that are passed to this field will be given to every driver that's registered with the bus. Those attributes can be found in the driver' directory in `/sys/bus/<bus-name>/drivers/<driver-name>`.
*   `dev_groups` represents the default attributes of the devices on the bus. Any attributes that are passed (through the list/array of `struct attribute_group` elements) to this field will be given to every device that's registered with the bus. Those attributes can be found in the device's directory in `/sys/bus/<bus-name>/devices/<device-name>`.
*   `bus_group` holds the set (group) of default attributes that are added automatically when the bus is registered with the core.

Apart from defining `bus_type`, the bus driver must define a bus-specific driver structure that extends the generic `struct device_driver`, as well as a bus-specific device structure that extends the generic `struct device` structure. Both are part of the device model core. The bus driver must also allocate a bus-specific device structure for each physical device that's discovered when probing. It's also in charge of setting up the device's bus and parent fields, as well as registering them with the LDM core. These fields must point to the `bus_type` and the `bus_device` structures that are defined in the bus driver The LDM core uses them to build device hierarchy and initialize other fields.

Each bus internally manages two important lists: the list of devices that have been added and sitting on it, and the list of drivers that have been registered with it. Whenever you add/register or remove/unregister a device/driver with the bus, the corresponding list is updated with the new entry. The bus driver must provide helper functions to register/unregister device drivers that can handle devices on that bus, as well as helper functions to register/unregister devices sitting on the bus. These helper functions always wrap the generic functions that are provided by the LDM core, which are `driver_register()`, `device_register()`, `driver_unregister()`, and `device_unregister()`.

Now, let's start writing a new bus infrastructure called `PACKT`. `PACKT` is going to be our bus; the devices that will be sitting on this bus will be `PACKT` devices, while their drivers will be `PACKT` drivers. Let's start writing the helpers that will allow us to register the `PACKT` devices and drivers:

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

Now that we've created the registration helpers, let's write the only function that allows us to allocate a new `PACKT` device and register this device with the `PACKT` core:

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

The `packt_device_alloc()` function allocates a bus-specific device structure that must be used to register a `PACKT` device with the bus. At this stage, we should expose the helpers that allow us to allocate a `PACKT` controller device and register it – that is, register a new `PACKT` bus. To do this, we must define the `PACKT` controller data structure, like so:

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

In the preceding data structure, `name` represents the controller's name, `dev` represents the underlying struct device associated with this controller, `list` is used to insert this controller into the system's global list of `PACKT` controllers, and `send_msg` and `recv_msg` are hooks that must be provided by the controller to access the `PACKT` devices that are sitting on it:

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

In these two functions, we have demonstrated what the controller allocation and registration operations look like. The allocation method allocates memory and does some basic initialization, leaving room for the driver to do the rest. Note that after registering a controller, it will appear under `/sys/devices` in sysfs. Any devices that are added to this bus will appear under `/sys/devices/packt-0/`.

### Bus registration

The bus controller is a device itself, and in most cases, buses are memory-mapped platform devices (even buses are, which support device enumeration). For example, the PCI controller is a platform device and so is its respective driver. We should use the `bus_register(struct *bus_type)` function to register a bus with the kernel. The `PACKT` bus structure looks as follows:

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

Now that the basic bus operations have been defined, we need to register the `PACKT` bus framework and make it available for both the controller and slave drivers. The bus controller is a device itself; it must be registered with the kernel and will be used as the parent of the device that's sitting on the bus. This is done in the bus controller's probe or `init` function. In the case of the `PACKT` bus, the code would be as follows:

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

When a device is registered by the bus controller driver, the `parent` member of the device must point to the bus controller device (this should be done by the device driver) and its `bus` property must point to the `PACKT` bus type (this is done by the core) to build the physical device tree. To register a `PACKT` device, you must call `packt_device_register()`, which should take as an argument a `PACKT` device allocated with `packt_device_alloc()`:

```
int packt_device_register(struct packt_device *packt)
{
    packt->dev.bus = &packt_bus_type;
    return device_register(&packt->dev);
}
EXPORT_SYMBOL(packt_device_register);
```

Now that we are done with bus registration, let's look at the driver's infrastructure and see how it is designed.

## The driver data structure

A driver is a set of methods that allow us to drive a given device. A global device hierarchy allows all the system's devices to be represented in the same way. This makes it simple for the core to navigate the device tree and perform tasks such as properly ordered power management transitions.

Each device driver is represented as an instance of `struct device_driver`, which is defined like so:

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

Let's look at the elements in this data structure:

*   `name` represents the driver's name. It can be used for matching, by comparing it with the device's name.
*   `bus` represents the bus where this driver sits on. The `bus` driver must fill this field.
*   `module` represents the module that owns this driver. In 99% of cases, you must set this field to `THIS_MODULE`.
*   `of_match_table` is a pointer to the array of `struct of_device_id`. The `struct of_device_id` structure is used to perform open firmware matches through a special file called the device tree, which is passed to the kernel during the boot process:

    ```
    struct of_device_id {
        char       compatible[128];
        const void *data;
    };
    ```

*   `suspend` and `resume` are power management callbacks that are invoked to put the device to sleep or wake it up from a sleep state, respectively. The `remove` callback is called when the device is physically removed from the system or when its reference count reaches `0`. The `remove` callback is also called during system reboot.
*   `probe` is the probe callback that runs when you're attempting to bind a driver to a device. The bus driver is in charge of calling the device driver's probe function.
*   `group` is a pointer to a list (array) of `struct attribute_group` and is used as a default attribute for the driver. Prefer this method instead of creating an attribute separately.

Now that we are familiar with the driver data structure and all its elements, let's learn what APIs the kernel provides so that we can register it.

### Driver registration

The low-level `driver_register()` function is used to register a device driver with the bus and it is added to the bus's driver list. When a device driver registers with the bus, the core travels through the list of devices on this same bus and calls the bus's match callback for each device that does not have a driver. It does this to find out if there are any devices that the driver can handle.

The following is the declaration of our driver infrastructure:

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

In our example, there are two helper macros to get the `PACKT` device and the `PACKT` driver, given a generic `struct device` or `struct driver`.

Then comes the structure that's used to identify a `PACKT` device, which is defined as follows:

```
struct packt_device_id {
    char name[PACKT_NAME_SIZE];
    kernel_ulong_t driver_data;   /* Data private to the driver */
};
```

The device and the device driver are bound together when they match. Binding is the process of associating a device with a device driver, and it is performed by the bus framework.

Now, let's get back to registering our drivers with our `PACKT` bus. The drivers must use `packt_register_driver(struct packt_driver *driver)`, which is a wrapper around `driver_register()`. The `driver` parameter must have been filled in before registering the `PACKT` driver. The LDM core provides helper functions for iterating over the list of drivers that have been registered with the bus:

```
int bus_for_each_drv(struct bus_type * bus,
                struct device_driver * start, void * data,
                int (*fn)(struct device_driver *, void *));
```

This helper iterates over the bus's list of drivers and calls the `fn` callback for each driver in the list.

## The device data structure

`struct device` is the generic data structure that's used to describe and characterize each device on the system, whether it is physical or not. It contains details about the physical attributes of the device and provides proper linkage information to help build suitable device trees and reference counting:

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

The preceding data structure has been shortened for the sake of readability. That said, let's take a look at the elements that have been provided:

*   `parent` represents the device's parent and is used to build the device tree hierarchy. When registered with a bus, the bus driver is responsible for setting this field with the bus device.
*   `bus` represents the bus where this device sits. The bus driver must fill this field.
*   `type` identifies the device's type.
*   `kobj` is the kobject and handles reference counting and device model support.
*   `of_node` is a pointer to the open firmware (device tree) node associated with the device. It is up to the bus driver to set this field.
*   `platform_data` is a pointer to the platform data that's specific to the device. It's usually declared in a board-specific file during device provisioning.
*   `driver_data` is a pointer to private data for the driver.
*   `class` is a pointer to the class that this device belongs to.
*   `group` is a pointer to a list (array) of `struct attribute_group` and is used as the default attributes for the device. You should use this instead of creating the attributes separately.
*   `release` is a callback that's called when the device reference count reaches zero. The bus has the responsibility of setting up this field. The `PACKT` bus driver shows you how to do that.

Now that we have described the device's structure, let's learn how to make it part of the system by registering it.

### Device registration

`device_register()` is a function that's provided by the LDM core to register a device with the bus. After this call, the bus's list of drivers is iterated over to find the driver that supports this device; then, this device is added to the bus's device list. `device_register()` internally calls `device_add()`:

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

The helper function that's provided by the kernel to iterate over the bus's device list is `bus_for_each_dev`. It's defined as follows:

```
int bus_for_each_dev(struct bus_type * bus,
                    struct device * start, void * data,
                    int (*fn)(struct device *, void *));
```

Whenever a device is added, the core invokes the matching method of the bus driver (`bus_type->match`). If the matching function succeeds, the core will invoke the probing function of the bus driver (`bus_type->probe`), given that both the device and driver matched as parameters. Then, it is up to the bus driver to invoke the probing method of the device's driver (that is, `driver->probe`). For our `PACKT` bus driver, the function that's used to register a device is `packt_device_register(struct packt_device *packt)`, which internally calls `device_register()`. Here, the parameter is a `PACKT` device that's been allocated with `packt_device_alloc()`.

The bus-specific device data structure is then defined as follows:

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

In the preceding code, `dev` is the underlying `struct device` structure for the device model, while `name` is the name of the device.

Now that we have defined the data structure, let's learn about the underlying mechanisms of LDM.

# Getting deeper inside LDM

So far, we have discussed buses, drivers, and devices, which were used to build the system device topology. While this is true, the previous topics were the tip of the iceberg. Under the hood, LDM relies on the three lowest level data structures, which are `kobject`, `kobj_type`, and `kset`. These are used to link the objects.

Before we go any further, let's define some of the terms that will be used throughout this chapter:

*   `struct kobject`.
*   **Attribute**: An attribute (or sysfs attribute) appears as a file in sysfs. From within the kernel, it can be mapped to anything: a variable, a device property, a buffer, or anything useful to the driver that may need to be exported to the world.

In this section, we will learn how each of these structures is involved in the device model.

## Understanding the kobject structure

`struct kobject`, which means kernel object (also abbreviated as `struct kobject` wandering around somewhere within the kernel. Additionally, a kobject can export one or more attributes, which appear in that Kobject's sysfs directory as files. Now, let's get back to the code – `struct kobject` is defined in the kernel like so:

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

In the preceding data structure, only the main elements have been listed. Let's take a look at them in more detail:

*   `name` is the name of this kobject. It can be modified using the `kobject_set_name(struct kobject *kobj, const char *name)` function. It is used as the name of this `kobject` directory.
*   `parent` is a pointer to another kobject, which is considered the parent of this kobject. It is used to build topologies and to describe the relationship between objects.
*   `sd` points to a `struct sysfs_dirent` structure that represents the directory of this kobject in sysfs. `name` will be used as the name of this directory. If `parent` is set, then this directory will be a sub-directory in the parent's directory.
*   `kref` provides reference counting for the kobject. It helps track whether an object is still in use or not and potentially releases it if it's not being used anymore. Alternatively, it can prevent it from being removed if it's still in use. When it's used from within a kernel object, its initial value is `1`.
*   `ktype` describes the kobject. Every kobject is given a set of default attributes when it is created. The `ktype` element, which belongs to the `struct kobj_type` structure, is used to specify these default attributes. Such a structure allows kernel objects to share common operations (`sysfs_ops`), whether those objects are functionally related or not.
*   `kset` tells us which set (group) of objects this object belongs to.

Before a kobject can be used, it must be (exclusively) dynamically allocated and then initialized. To do so, drivers can use the `kzalloc()` (or `kmalloc()`) or `kobject_create()` function. With `kzalloc()`, the object is allocated and empty and must be initialized using another API, `kobject_init()`. With `kobject_create()`, allocation and initialization are implicit. These APIs are defined as follows:

```
void kobject_init(struct kobject *kobj,
                  struct kobj_type *ktype)
struct kobject *kobject_create(void)
```

In the preceding code, `kobject_init()` expects, in its first parameter, a kobject that has been initially allocated, via `kzalloc()` for example. The second parameter, `ktype`, is mandatory, so it can't be `NULL`; otherwise, the kernel will complain (`dump_stack()`). `kobject_create()`, on the other hand, expects nothing; it does allocation and initialization implicitly (it calls `kzalloc()` and `kobject_init()` internally). On success, it returns a freshly initialized kobject object.

Once a kobject has been initialized, the driver can use `kobject_add()` to link this object with the system, creating a sysfs directory entry for this kobject. Where this directory will be created depends on the parent element of the kobject being set or not. That said, if this parent is not set, it can be specified while adding the kobject to the system using `kobject_add()`, which is defined as follows:

```
int kobject_add(struct kobject *kobj, struct kobject *parent,
                const char *fmt, ...);
```

In the preceding function, `kobj` is the kernel object to be added to the system and `parent` is its parent. The `kobject` directory will be created as a sub-directory of its parent. If `parent` is `NULL`, the directory will be created under `/sys/` directly.

Instead of using `kobject_init()` or `kobject_create()` and then `kobject_add()` individually, it is possible to use `kobject_init_and_add()`, which groups their actions. It's defined like so:

```
int kobject_init_and_add(struct kobject *kobj,
                         struct kobj_type *ktype,
                         struct kobject *parent,
                         const char *fmt, ...);
```

In the preceding interface, the object needs to be allocated first. There is another helper that will implicitly allocate, initialize, and add the kobject with the system. This function is `kobject_create_and_add()` and it's defined as follows:

```
struct kobject * kobject_create_and_add(const char *name,
                                     struct kobject *parent);
```

The preceding function takes the name of the kobject that will be used as the directory name, as well as a parent kobject whose created directory will be a sub-directory. Passing `NULL` as the second parameter will result in the directory being created under `/sys/` directly.

That said, some predefined kobjects in the kernel already represent some directories under `/sys/`. Let's look at a few of them:

*   `kernel_kobj`: This kobject is responsible for the `/sys/kernel` directory.
*   `mm_kobj`: This is responsible for `/sys/kernel/mm`.
*   `fs_kobj`: This is the filesystem kobject and it's responsible for `/sys/fs`.
*   `hypervisor_kobj`: This is responsible for `/sys/hypervisor`.
*   `power_kobj`: This is a power management kobject that's used at the origin of `/sys/power`.
*   `firmware_kobj`: This is the firmware kobject that owns the `/sys/firmware` directory.

Once you're done with a kobject, the driver should release it. The low-level function you can use to release a kobject is `kobject_release()`. However, this API does not consider other potential users of the kobject. It is raw and dummy. It is recommended to use `kobject_put()` instead, which will decrement the kobject's reference counter and then release this kobject if the new reference counter value is `0`. Remember that when a kobject is initialized, the reference counter value is set to `1`. Moreover, it is also recommended that users wrap the kobject's usage into the `kobject_get()` and `kobject_put()` functions, where `kobject_get()` will just increment the reference counter value.

These APIs have the following prototypes:

```
void kobject_put(struct kobject * kobj);
struct kobject *kobject_get(struct kobject *kobj);
```

In the preceding code, `kobject_get()` takes the kobject to increase the reference counter as a parameter and returns this same kobject once the parameter has been initialized. `kobject_put()` will decrement 1 from the reference counter and, if the new value is `0`, `kobject_release()` will be automatically called on the object, which will release it.

The following code shows how to combine `kobject_create()`, `kobject_init()`, and `kobject_add()` to create and add a kernel object to the system:

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

As we can see, we could use an all-in-one function, such as `kobject_create_and_add()`, which internally calls `kobject_create()` and `kobject_add()`. The following excerpt from `drivers/base/core.c` shows how to use it:

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

Keep in mind that for each `struct kobject`, the corresponding kobject directory can be found in `/sys/`, and the upper directory is pointed out by `kobj->parent`.

Note

Since it is mandatory to provide a `kobj_type` while initializing a kobject, all-in-one helpers use a default and kernel-provided `kobj_type`; that is, `dynamic_kobj_ktype`. Therefore, unless you have a good reason to initialize your `kobj_type` (most of the time, this will be if you wish to populate some default attributes), you should use `kobject_create*()` variants, which use the kernel-provided kobject type instead of `kobject_init*()`, which would require providing your own initialized `kobj_type`.

## Understanding the kobj_type structure

A `struct kobj_type` structure, which means kernel object type, is a data structure that defines the behavior of a kobject element and controls what happens to this kobject when it is created or destroyed. Additionally, a `kobj_type` contains the default attributes of the kobject, as well as the hooks that allow it to operate on these attributes.

Because most devices of the same type have the same attributes, these attributes are isolated and stored in the `ktype` element. This allows them to be managed flexibly. Every kobject must have an associated `kobj_type` structure. Its data structure is defined as follows:

```
struct kobj_type {
    void (*release)(struct kobject *);
    const struct sysfs_ops sysfs_ops;
    struct attribute **default_attrs;
};
```

In the preceding data structure, `release` is a callback that's called on the release path of the kobject to give drivers a chance to release resources that have been allocated for the kobject. This callback is implicitly run when `kobject_put()` is about to free the kobject. `default_attrs` is an array of pointers to `attribute` structures. This field lists the attributes to be created for every kobject of this type, while `sysfs_ops` provides a set of methods that allow you to access those attributes.

The following code shows the definition of the `struct sysfs_ops` data structure in the kernel:

```
struct sysfs_ops {
    ssize_t (*show)(struct kobject *kobj,
                   struct attribute *attr, char *buf);
    ssize_t (*store)(struct kobject *kobj,
                   struct attribute *attr, const char *buf,
                   size_t size);
};
```

In the preceding code, `show` is the callback that's invoked in response to a read operation of an attribute being exposed by this `kobj_type` – that is, whenever an attribute is read from the user space. `buf` is the output buffer. The buffer's size is fixed and is `PAGE_SIZE` in length. The data that must be exposed must be put inside `buf`, preferably using `scnprintf()`. Finally, if the callback succeeds, it must return the size (in bytes) of the data that was written into the buffer, or a negative error if it fails. Each attribute should contain/provide a single, human-readable value or property, according to the sysfs rules; if you have a lot of data to return, you should consider breaking it into numerous attributes.

`store` is called for writing purposes – that is, when users write something into an attribute. Its `buf` parameter is `PAGE_SIZE` at most but it can be smaller. It must return the size (in bytes) of the data that was read from the buffer on success or a negative error on failure (or if an unwanted value is received).

The `attr` pointer is passed as an argument into both methods and can be utilized to determine which attribute is being accessed. It is frequent for `show`/`store` methods to perform a series of tests you can perform on the attribute name to achieve that. Other implementations, on the other hand, wrap the attribute's structure in an enclosing data structure that provides the data that's required to return the attribute's value (`struct kobject_attribute`, `struct device_attribute`, `struct driver_attribute`, and `struct class_attribute` are some examples); in this case, the `container_of` macro is used to obtain a pointer to the embedding structure. This is the method that's used by `kobj_sysfs_ops`, which represents the operations that are provided by `dynamic_kobj_ktype`. Both methods are demonstrated in the examples provided with this book.

## Understanding the kset structure

The purpose of `struct kset` is mainly to group related kernel objects together. `kset` stands for kernel object sets, which can be interpreted as a collection of kobjects. In other words, a `kset` gathers related kobjects into a single place, such as all *block devices*.

The `kset` data structure is defined in the kernel like so:

```
struct kset {
    struct list_head list; 
    spinlock_t list_lock;
    struct kobject kobj;
 };
```

All the elements in the data structure are quite self-explanatory. Simply put, `list` is a linked list of all kobjects in `kset`, `list_lock` is a spinlock that protects linked list access (while adding or removing kobject elements in `kset`), and `kobj` represents the base class kobject for the set. This kobject will be used as the default parent of kobjects to be added in the set with a `NULL` parent.

Each registered `kset` corresponds to a sysfs directory that's created on behalf of its `kobj` element. A `kset` can be created and added using the `kset_create_and_add()` function and removed with `kset_unregister()`. The following code shows the definitions for both:

```
struct kset * kset_create_and_add(const char *name,
                      const struct kset_uevent_ops *u,
                      struct kobject *parent_kobj);
void kset_unregister (struct kset * k);
```

In the preceding APIs, `name` is the name of `kset`, which is also used as the name of the directory that will be created for `kset`. The `u` parameter is a pointer to a `struct uevent_ops`, which represents a set of `kset` so that, for example, it can add new environment variables or filter out the uevents if so desired. This parameter can be (and most of the time is) `NULL`. Finally, `parent_kobj` is the parent kobject of `kset`.

Adding a kobject to the set is as simple as specifying its `.kset` field for the right `kset`:

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

Once you're done with your `kset`, it can be released with `kset_unregister()`, after which it will be dynamically deallocated when it is no longer in use. The following code will release our example's `kset`:

```
kset_unregister(example_kset);
```

Now that we are familiar with kobjects and type structures, let's learn how to deal with non-default sysfs attributes.

## Working with non-default attributes

Attributes are sysfs files that are exported to the user space via kobjects. While default attributes might be enough most of the time, you can add other attributes. An attribute can be readable, writable, or both, from the user space.

An attribute definition looks as follows:

```
struct attribute {
        char              *name;
        struct module     *owner;
        umode_t           mode;
};
```

In the attribute data structure, `name` is the name of the attribute, which is also the name of the corresponding file entry. `owner` is the attribute owner – most of the time, this is `THIS_MODULE` – and `mode` specifies the read/write permissions for this attribute in **user-group-other** (**ugo**) format.

Default attributes are very convenient to use but are not flexible enough. Moreover, simple attributes cannot be read or written except by their `kobj_type` sysfs ops, which means that if there are too many attributes, the branches in the show/store functions will be messy. To address this, the kobject core provides a mechanism where each attribute is embedded in an enclosing and special data structure: `struct kobj_attribute`. This data structure exposes the wrapper routines for reading and writing.

`struct kobj_attribute` (defined in `include/linux/kobject.h`) looks as follows:

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

In this data structure, `attr` is the attribute representing the file to be created, `show` is a pointer to a function that will be called when the file is read from the user space, and `store` is a pointer to a function that will be called when the file is written, again from the user space.

Using the enclosing `kobj_attribute` structure makes developments more generic and extends attribute flexibility. This way, a pointer to `attr` is passed to either the `store` or `show` function and can be used not only to determine which attribute is being accessed but also to retrieve the enclosing data structure (that is, `kobj_attribute`) that this attribute's `show`/`store` method can be invoked from. To do so, you can use the `container_of` macro to obtain a pointer to the embedding structure.

The following is an excerpt from `lib/kobject.c` that demonstrates this generic mechanism in both the `show` and `store` methods of a kernel-provided sysfs operations element: `kobj_sysfs_ops`. This element is also the sysfs operations data structure (the `kobj_type->sysfs_ops` element) that's used by `dynamic_kobj_ktype`:

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

In the preceding code, the `container_of` macro does everything. This also reassures us that with this approach, we remain compatible with all the bus-, device-, class-, and driver-related kobject implementations, as we will see in the next section.

Let's go back to the APIs. You will probably always know which attributes you wish to expose in advance; thus, the attributes will almost always be declared statically. To help with this, the kernel provides the `__ATTR` macro for initializing `kobj_attribute`. This macro is defined as follows:

```
#define __ATTR(_name, _mode, _show, _store) {         \
    .attr = {.name = __stringify(_name),              \
           .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },\
     .show   = _show,                              \
     .store = _store,                               \
}
```

In the preceding macro definition, `_name` will be stringified and used as the attribute name, `_mode` represents the attribute mode, and `_show` and `_store` are pointers to the attribute's `show` and `store` methods, respectively.

The following is an example of two attribute's declarations, `bar` and `foo` (this example will be used as a base later in this section):

```
static struct kobj_attribute foo_attr =
    __ATTR(foo, 0660, attr_show, attr_store);
static struct kobj_attribute bar_attr =
    __ATTR(bar, 0660, attr_show, attr_store);
```

In the preceding example, we have two attributes with the `0660` permission. The first attribute is named `foo`, the second one is named `bar`, and both use the same `show` and `store` methods.

Now, we must create the underlying file. The low-level kernel APIs that are used to add/remove attributes from the sysfs filesystem are `sysfs_create_file()` and `sysfs_remove_file()`, respectively. They are defined as follows:

```
int sysfs_create_file(struct kobject * kobj,
                      const struct attribute * attr);
void sysfs_remove_file(struct kobject * kobj,
                        const struct attribute * attr);
```

`sysfs_create_file()` returns 0 on success or a negative error on failure. `sysfs_remove_file()` must be given the same parameters to remove the file attributes.

Let's use these APIs to add our `bar` and `foo` attributes to the system:

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

Once the preceding code has been executed, the `bar` and `foo` files will be visible in sysfs, in the `/sys/demo` directory. In our example, we used the `__ATTR` macro to define our attributes. We had to specify the name, the mode, and the `show`/`store` methods. The kernel provides convenience macros for the most frequent cases to make specifying attributes and writing code more succinct, readable, and easier. These macros are as follows:

*   `__ATTR_RO(name)`: This assumes that `name_show` is the show callback's name and sets the mode to `0444`.
*   `__ATTR_WO(name)`: This assumes that `name_store` is the store function's name and is restricted to mode `0200`, which means root write access only.
*   `__ATTR_RW(name)`: This assumes that `name_show` and `name_store` are for the `show` and `store` callbacks' names, respectively, and sets the mode to `0644`.
*   `__ATTR_NULL`: This is used as a list terminator. It sets both names to `NULL` and is used as an end of list indicator (see `kernel/workqueue.c`).

All these macros only expect the name of the attribute as a parameter. The difference with these macros is that unlike `__ATTR`, whose `store`/`show` function names can be arbitrary, the attributes here are built under the assumption that the `show` and `store` methods are named `<attribute_name>_show` and `<attribute_name>_store`, respectively. The following code demonstrates this with the `__ATTR_RW_MODE` macro, which is defined as follows:

```
#define __ATTR_RW_MODE(_name, _mode) {                 \
    .attr = { .name = __stringify(_name),              \
          .mode = VERIFY_OCTAL_PERMISSIONS(_mode) },\
    .show   = _name##_show,                            \
    .store  = _name##_store,                          \
}
```

As we can see, the `.show` and `.store` fields are set with their attribute names suffixed with `_show` and `_store`, respectively. Let's take a look at the following example:

```
static struct kobj_attribute attr_foo = __ATTR_RW(foo);
```

The preceding attribute declaration assumes that the `show` and `store` methods are defined as `foo_show` and `foo_store`, respectively.

Note

If you need to provide a single store/show operation pair for all the attributes, you should probably define these attributes with `__ATTR`. However, if processing the attributes requires providing a show/store pair per attribute, you can use the other attribute definition macros.

The following code shows the implementation of the show/store function for our previously defined `foo` and `bar` attributes:

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

In the preceding code, instead of providing a pair of show/store operations per attribute, we have used the same function pair for all the attributes, and we differentiated the attributes by their respective names. This is a common practice when you're using the generic `kobject_attribute` instead of framework-specific attributes. This is because they sometimes impose different show/store function names for each attribute since they do not rely on the `__ATTR` macro for defining attributes.

### Working with binary attributes

So far, we have become familiar with the sysfs statement and mentioned that an attribute must store a single property/value in a human-readable text format, as well as that such an attribute has a `PAGE_SIZE` limit. However, there could be situations, although rare, which would require larger data to be exchanged in binary format, for example, all with random access. An example of such a situation is a device firmware transfer, where the user space would upload some binary data to be pushed to the hardware or PCI devices, exposing part or all of their configuration address spaces.

To cover those cases, the sysfs framework provides binary attributes. Note that these attributes are for sending/receiving binary data that is not interpreted/manipulated by the kernel at all. It should *only* be used as a pass-through to and from hardware, with no interpretation by the kernel. The only manipulations you can perform are some checks on the magic number and size, for example.

Now, let's get back to the code A binary attribute is represented using a `struct bin_attribute` and is defined as follows:

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

In the preceding code, `attr` is the underlying classic attribute for this binary attribute and holds the name, owner, and permissions for the binary attribute. `size` represents the maximum size of the binary attribute (or zero if there is no maximum limit). `private` is a field that can be used for any convenience. Most of the time, it is assigned the buffer of the binary attribute. The `read()`, `write()`, and `mmap()` functions are optional and work similarly to the normal `char` driver equivalents. In their parameters, `filp` is an opened file pointer instance that's associated with the attribute and `kobj` is the underlying `kobject` associated with this this binary attribute. `buffer` is the output or input buffer for read or write operations, respectively. `off` is the same offset argument that's found in all read or write methods for all types of files. It refers to the offset from the start of the file – that is, offset into the binary data. Finally, `count` is the number of bytes to read or write.

Note

Though binary attributes may not have size limitations, larger data is always requested/sent on a `PAGE_SIZE` chunk basis. This means that, for example, the `write()` function can be called multiple times for a single load. However, this split is handled by the kernel, which means it's transparent for the driver. The disadvantage is that sysfs has no way of signaling the end of a series of write operations, so code that implements a binary attribute must figure it out some other way.

For a binary attribute to be created, it must be allocated and initialized. Like classic attributes, there are two ways to allocate binary attributes – either statically or dynamically. For static allocation, the framework provides the low-level `__BIN_ATTR` macro, which is defined as follows:

```
#define __BIN_ATTR(_name, _mode, _read, _write, _size) {  \
   .attr = { .name = __stringify(_name), .mode = _mode }, \
   .read    = _read,                        \
   .write   = _write,                       \
    .size      = _size,                          \
```

It works similarly to the `__ATTR` macro. In terms of parameters, `_name` is the binary attribute name, `_mode` represents its permissions, `_read` and `_write` are the read and write functions, respectively, and `_size` is the size of the binary attribute.

Like classic attributes, binary attributes have their own high-level helper macros to ease the process of defining them. Some of these macros are as follows:

```
BIN_ATTR_RO(name, size)
BIN_ATTR_WO(name, size)
BIN_ATTR_RW(name, size)
```

These macros declare a single instance of `struct bin_attribute`, whose corresponding variable is named `bin_attribute_<name>`, as shown in the following `BIN_ATTR` definition:

```
#define BIN_ATTR_RW(_name, _size)         \
struct bin_attribute bin_attr_##_name =   \
                __BIN_ATTR_RW(_name, _size)
```

Moreover, like classic attributes, these high-level macros expect the read/write methods to be named `<attribute_name>_read` and `<attribute_name>_write`, respectively, as shown in the following `__BIN_ATTR_RW` definition:

```
#define __BIN_ATTR_RW(_name, _size) \
 __BIN_ATTR(_name, 0644, _name##_read, _name##_write, \
             _size)
```

For dynamic allocation, a simple `kzalloc()` is enough. However, dynamically allocated binary attributes must be initialized using `sysfs_bin_attr_init()`, as shown here:

```
void sysfs_bin_attr_init(strict bin_attribute *bin_attr)
```

After this, the driver must set other properties, such as the underlying attribute's mode, name, and permission, and optionally the read/write/map functions.

Unlike classic attributes, which can be set up as default attributes, binary attributes must be created explicitly. This can be done using `sysfs_create_bin_file()`, as follows:

```
int sysfs_create_bin_file(struct kobject *kobj, 
                          struct bin_attribute *attr);
```

This function returns `0` on success or a negative error on failure. Once you're done with a binary attribute, it can be removed with `sysfs_remove_bin_file()`, which is defined as follows:

```
int sysfs_remove_bin_file(struct kobject *kobj, 
                          struct bin_attribute *attr);
```

The following is an excerpt (whose full version can be found in `drivers/i2c/i2c-slave-eeprom.c`) of a concrete example highlighting the use of a binary attribute that's been allocated and initialized dynamically:

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

Upon unloading the path of the module or when the device leaves, the associated binary file is removed, as follows:

```
static int i2c_slave_eeprom_remove(struct i2c_client *client)
{
   struct eeprom_data *eeprom = i2c_get_clientdata(client);
   sysfs_remove_bin_file(&client->dev.kobj, &eeprom->bin);
[...]
   return 0;
}
```

Then, when it comes to implementing the read/write function, data can be moved back and forth using `memcpy()`, as shown here:

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

In the preceding excerpt, the offset (the `off` parameter) points to where the data should be read/written, and `count` determines the size of this data.

### The concept of attribute group

So far, we have learned how to individually add (binary) attributes by calling the `sysfs_create_file()` or `sysfs_create_bin_file()` function. While this is enough if we have a few attributes to add, it may become painful as the number of attributes grows, either upon adding or removing them. The driver will have to loop over the attributes to create each of them or invoke `sysfs_create_file()` as many times as there are attributes. Here is where the attribute group comes in. It relies on the `struct attribute_group` structure, which is defined as follows:

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

If it is unnamed, an attribute group will place all the attributes directly in the kobject's directory when defining a group of attributes. If, however, a `name` is supplied, a subdirectory will be created for the attributes, with the directory's name being the name of the attribute group. `is_visible()` is an optional callback that intends to return the permissions associated with a specific attribute in the group. It will be called repeatedly for each (non-binary) attribute in the group. This callback must then return the read/write permission of the attribute, or `0` if the attribute is not supposed to be accessed at all. `is_bin_visible()` is the counterpart of `is_visible()` for binary attributes. The returned value/permission will replace the static permissions that have been defined in `struct attribute`. The `attrs` element is a pointer to a `NULL` terminated list of attributes, while `bin_attrs` is its counterpart for binary attributes.

The kernel functions that are used to add/remove group attributes to/from the filesystem are as follows:

```
int sysfs_create_group(struct kobject *kobj,
                       const struct attribute_group *grp)
void sysfs_remove_group(struct kobject * kobj,
                        const struct attribute_group * grp)
```

Back to our demo example with standard attributes, the two `bar` and `foo` attributes can be embedded into a `struct attribute_group`. This will allow us adding these to the system in a single shot, using one function call as follows:

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

Finally, to create the attributes in a single shot, we need to use `sysfs_create_group()`, as shown in the following code:

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

Here, we have demonstrated the importance of creating a group of attributes and how easy it is to use their APIs. While we have been generic so far, in the next section, we'll learn how to create framework-specific attributes.

### Creating symbolic links

Drivers can create/remove symbolic links on existing kobjects (directories) using `sysfs_{create|remove}_link()` functions, as shown here:

```
int sysfs_create_link(struct kobject * kobj,
                     struct kobject * target, char * name); 
void sysfs_remove_link(struct kobject * kobj, char * name);
```

This allows an object to exist in more than one place or even create a shortcut. The `create` function will create a symbolic link called `name` that points to the remote `target` kobject's sysfs entry. The link will be created in the `kobj` kobject directory. A well-known example is devices appearing in both `/sys/bus` and `/sys/devices` since a bus controller is first a device on its own before exposing a bus. However, note that any symbolic links that are created will be persistent (unless the system is rebooted), even after target removal. Thus, the driver must consider that when the associated device leaves the system or when the module is unloaded.

# Overview of the device model from sysfs

sysfs is a non-persistent virtual filesystem that provides a global view of the system and exposes the kernel objects hierarchy (topology) using their kobjects. Each kobject shows up as a directory. The files in these directories represent the kernel variables that are exported by the related kobject. These files are called attributes and can be read or written.

If any registered kobject creates a directory in sysfs, where the directory is created depends on the parent of this kobject (which is also a kobject, thus highlighting internal object hierarchies). In sysfs, top-level directories represent the common ancestors of object hierarchies or the subsystems that the objects belong to.

These top-level sysfs directories can be found in the `/sys/` directory, as follows:

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

`block` contains a directory per block device on the system. Each of these contains subdirectories for partitions on the device. `bus` contains the registered bus on the system. `dev` contains the registered device nodes in a raw way (no hierarchy), with each being a symbolic link to the real device in the `/sys/devices` directory. The `devices` directory gives the real view of the topology of devices in the system. `firmware` shows a system-specific tree of low-level subsystems such as ACPI, EFI, and OF (device tree). `fs` lists the filesystems that are used on the system. `kernel` holds the kernel configuration options and status information. Finally, `module` is a list of loaded modules and `power` is the system power management control interface from the user space.

Each of these directories corresponds to a kobject, some of which are exported as kernel symbols. These are as follows:

*   `kernel_kobj`, which corresponds to `/sys/kernel`.
*   `power_kobj`, which corresponds to `/sys/power`.
*   `firmware_kobj`, which corresponds to `/sys/firmware`. It's exported in the `drivers/base/firmware.c` source file.
*   `hypervisor_kobj`, which corresponds to `/sys/hypervisor`. It's exported in the `drivers/base/hypervisor.c` source file.
*   `fs_kobj`, which corresponds to `/sys/fs`. This is exported in the `fs/namespace.c` source file.

For the rest, `class/`, `dev/` and `devices/` are created at boot time by the `devices_init()` function in `drivers/base/core.c` in the kernel sources, `block/` is created in `block/genhd.c`, and `bus/` is created as a `kset` in `drivers/base/bus.c`.

## Creating device-, driver-, bus- and class-related attributes

So far, we have learned how to create dedicated kobjects to populate attributes inside. However, the device, driver, bus, and class frameworks provide attribute abstractions and file creation, where the attributes that are created are directly tied to the respective framework in the appropriate kobject directory.

To do so, each framework provides a framework-specific attribute data structure that encloses the default attribute and allows us to provide a custom show/store callback. These are `struct device_attribute`, `struct driver_attribute`, `struct bus_atttribute`, and `struct class_attribute` for the device, driver, bus, and class frameworks, respectively. They are defined like `kboj_attribute` is but use different names. Let's look at their respective data structures:

*   Devices have the following attribute data structure:

    ```
    struct driver_attribute {
        struct attribute attr;
        ssize_t (*show)(struct device_driver *driver,
                    char *buf);
        ssize_t (*store)(struct device_driver *driver,
                    const char *buf, size_t count);
    };
    ```

*   Classes have the following attribute data structure:

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

*   The bus framework has the following attribute data structure:

    ```
    struct bus_attribute {
        struct attribute  attr;
        ssize_t (*show)(struct bus_type *bus, char *buf);
        ssize_t (*store)(struct bus_type *bus,
                     const char *buf, size_t count);
    };
    ```

*   Devices have the following attribute data structure:

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

The preceding device-specific data structure's `show` function takes an additional `count` parameter, whereas the others do not.

They can be dynamically allocated with `kzalloc()` and initialized by setting the fields of their inner attribute elements and providing the appropriate callback functions. However, each framework provides a set of macros to statically allocate, initialize, and assign a single instance of their respective attribute data structure. Let's look at these macros:

*   The bus infrastructure provides the following macros:

    ```
    BUS_ATTR_RW(_name)
    BUS_ATTR_RO(_name)
    BUS_ATTR_WO(_name)
    ```

With these bus framework-specific macros, the resulting bus attribute variable is named `bus_attr_<_name>`. For example, the variable name that results from `BUS_ATTR_RW(foo)` will be `bus_attr_foo` and will be of the `struct bus_attribute` type.

*   For drivers, the following macros are provided:

    ```
    DRIVER_ATTR_RW(_name)
    DRIVER_ATTR_RO(_name)
    DRIVER_ATTR_WO(_name)
    ```

These driver-specific attribute definition macros will name the resulting variable using the `driver_attr_<_name>` pattern. Therefore, the variable that results from `DRIVER_ATTR_RW(foo)` will be of the `struct driver_attribute` type and will be named `driver_attr_foo`.

*   The class framework works with the following macros:

    ```
    CLASS_ATTR_RW(_name)
    CLASS_ATTR_RO(_name)
    CLASS_ATTR_WO(_name)
    ```

Using these class-specific macros, the resulting variable will be of the `struct class_atribute` type and will be named based on the `class_attr_<_name>` pattern. Thus, the resulting variable name of `CLASS_ATTR_RW(foo)` will be `class_attr_foo`.

*   Finally, device-specific attributes can be statically allocated and initialized using the following macros:

    ```
    DEVICE_ATTR(_name, _mode, _show, _store)
    DEVICE_ATTR_RW(_name)
    DEVICE_ATTR_RO(_name)
    DEVICE_ATTR_WO(_name)
    ```

Device-specific attributes definition macros use their own pattern for variable names, which is `dev_attr_<_name>`. Thus, for example, `DEVICE_ATTR_RO(foo)` will result in a `struct device_attribute` object named `dev_attr_foo`.

Because all these macros are built on top of `__ATTR_RW`, `__ATTR_RO`, and `__ATTR_WO`, they statically allocate and initialize a single instance of the framework-specific attribute data structure and assume the show/store functions are named `<attribute_name>_show` and `<attribute_name>_store`, respectively (remember, this is because they do not rely on the `__ATTR` macro). There is an exception for `DEVICE_ATTR()`, which uses the show/store function as it was passed, without any suffix or prefix. This exception is because `DEVICE_ATTR` relies on `__ATTR` to define attributes.

As we have seen, all these framework-specific macros use a predefined prefix to name the resulting framework-specific attribute object variable. Let's take a look at the following class attribute:

```
static CLASS_ATTR_RW(foo);
```

This will create a static variable of the `struct class_attribute` type named `class_attr_foo` and will assume that its show and store functions are named `foo_show` and `foo_store`, respectively. This can be referenced in a group using its inner attribute element, as shown here:

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

The most important thing when it comes to creating the respective files is that the driver can select the appropriate API from the following list:

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

Here, the `device`, `driver`, `bus`, and `class` arguments are the respective device, driver, bus, and class entities that the attribute must be added to. Moreover, the attribute will be created in the directory that corresponds to the inner kobject of each entity, as shown in the following code:

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

To kill two birds with one stone, the preceding code also shows that `device_create_file()`, `bus_create_file()`, `driver_create_file()` and `class_create_file()` all make an internal call to `sysfs_create_file()`.

Once you're done with each respective attribute object, the appropriate removal method must be invoked. The following code shows the possible options:

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

Each of these APIs expects the same arguments as those that are passed when the attributes are created.

Now that you know how the inner show/store functions of the `kobj_atribute` elements are invoked, it should be obvious to you how those framework-specific show/store functions are invoked as well.

Let's have a look at the device's implementation. The device framework has an internal `kobj_type` that implements device-specific show and store functions. These functions take in the inner attribute element as one of their arguments. After that, the `container_of` macro retrieves a pointer for the enclosing data structure (which is the framework-specific attribute data structure) that the framework-specific show and store functions are invoked from.

The following is an excerpt from `drivers/base/core.c` that shows the device-specific `sysfs_ops` implementation:

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

Note that in the preceding code, `to_dev_attr()`, which is the macro that makes use of `container_of`, is defined as follows:

```
#define to_dev_attr(_attr) \
       container_of(_attr, struct device_attribute, attr)
```

The principle is the same for the bus (in `drivers/base/bus.c`), driver (in `drivers/base/bus.c`), and class (in `drivers/base/class.c`) attributes.

## Making a sysfs attribute poll- and select-compatible

Though this is not a requirement for dealing with sysfs attributes, the main idea here is to allow the `poll()` or `select()` system calls to be used on a given attribute to passively wait for a change. This change could be firmware becoming available, an alarm notification, or information that the attribute value has changed. While the user would sleep on the file waiting for a change, the driver must invoke `sysfs_notify()`to release any sleeping user.

This notification API is defined as follows:

```
void sysfs_notify(struct kobject *kobj, const char *dir,
                  const char *attr)
```

If the `dir` parameter is not `NULL`, it is used to find a subdirectory from within the directory of `kobj`, which contains the attribute (presumably created by `sysfs_create_group`). This call will cause any polling process to wake up and process the event (which might be reading the new value, handling the alarm, and so on).

Note

There will be no notifications without this function call; therefore, any polling process will end up waiting indefinitely (unless a timeout was specified in the system call).

The following code, which shows the `store()` function of an attribute, is provided with this book:

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

In the preceding code, it makes sense to call `sysfs_notify()` once the value has been updated so that the user code can read the accurate value.

The user code can directly pass the opened attribute file to `poll()` or `select()` without having to read the initial content of this attribute. Doing so is at the convenience of the developer. However, note that upon notification, `poll()` returns `POLLERR|POLLPRI` (as are flags, which users must request while invoking `poll()`), while `select()` returns the file descriptor, whether it is waiting for read, write, or exception events.

# Summary

By completing this chapter, you should be familiar with LDM, its data structures (bus, class, device, and driver), and its low-level data structures, which are `kobject`, `kset`, `kobj_type`, and `attributes` (or a group of these). You should now know how objects are represented within the kernel (device topology) and be able to create an attribute (or group) that exposes your device or driver features and properties through sysfs.

In the next chapter, we will cover the **IIO** (**Industrial I/O**) framework, which heavily uses the power of sysfs.