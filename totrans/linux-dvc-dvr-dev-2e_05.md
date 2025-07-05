# *Chapter 4*: Writing Character Device Drivers

Unix-based systems expose hardware to user space by means of special files, all created in the `/dev` directory upon device registration with the system. Programs willing to access a given device must locate its corresponding device file in `/dev` and perform the appropriate system call on it, which will be redirected to the driver of the underlying device associated with that special file. Though system calls redirection is done by an operating system, what system calls are supported depends on the type of device and the driver implementation.

On the topic of types of devices, there are many of them from a hardware point of view, which are, however, grouped into two families of special device files in `/dev` – these are **block devices** and **character devices**. They are differentiated by the way they are accessed, their speed, and the way data is transferred between them and the system. Typically, character devices are slow and transfer data to or from user applications sequentially byte by byte (one character after another – hence their name). Such devices include serial ports and input devices (keyboards, mouses, touchpads, video devices, and so on). On the other hand, block devices are fast, since they are accessed quite frequently and transfer data in blocks. Such devices are essentially storage devices (hard drives, CD-ROMs, solid-state drives, and so on).

In this chapter, we will focus on character devices and their drivers, their APIs, and their common data structures. We will introduce most of their concepts and write our first character device driver.

We will cover the following topics in this chapter:

*   The concept of major and minor
*   Character device data structure introduction
*   Creating a device node
*   Implementing file operations

# The concept of major and minor

Linux has always enforced device file identification by a unique identifier, composed of two parts, a `/dev`, character or block device files are recognizable by their types, which can be seen using the `ls -l` command:

```
$ ls -la /dev
crw-------  1 root root    254,     0 août  22 20:28 gpiochip0
crw-------  1 root root    240,     0 août  22 20:28 hidraw0
[...]
brw-rw----  1 root disk    259,     0 août  22 20:28 nvme0n1
brw-rw----  1 root disk    259,     1 août  22 20:28 nvme0n1p1
brw-rw----  1 root disk    259,     2 août  22 20:28 nvme0n1p2
[...]
crw-rw----+ 1 root video    81,     0 août  22 20:28 video0
crw-rw----+ 1 root video    81,     1 août  22 20:28 video1
```

From the preceding excerpt, in the first column, `c` identifies character device files and `b` identifies block device files. In the fifth and sixth columns, we can see, respectively, major and minor numbers. The major number either identifies the type of device or can be bound to a driver. The minor number either identifies a device locally to the driver or devices of the same type. This explains the fact that some device files have the same major number in the preceding output.

Now that we are done with the basic concepts of character devices in a Linux system, we can start exploring the kernel code, starting from introducing the main data structures.

# Character device data structure introduction

A character device driver represents the most basic device driver in the kernel sources. Character devices are represented in the kernel as instances of `struct cdev`, declared in `include/linux/cdev.h`:

```
struct cdev {
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops;
    dev_t dev;
[...]
};
```

The preceding excerpt has listed elements of our interest only. The following shows the meaning of these elements in this data structure:

*   `kobj`: This is the underlying kernel object for this character device object, used to enforce the Linux device model. We will discuss this in [*Chapter 14*](B17934_14_Epub.xhtml#_idTextAnchor203), *Introduction to the Linux Device Model*.
*   `owner`: This should be set with the `THIS_MODULE` macro.
*   `ops`: This is the set of file operations associated with this character device.
*   `dev`: This is the character device identifier.

With this data structure introduced, the next logical one for discussion is the one exposing file operations that system calls will rely on. Let's then introduce the data structure that allows interaction between user space and kernel space through the character device.

## An introduction to device file operations

The `cdev->ops` element points to the file operations supported by a given device. Each of these operations is the target of a particular system call, in a manner that, when the system call is invoked by a program in user space on the character device, this system call is redirected in the kernel to its file operation counterpart in `cdev->ops`. `struct file_operations` is the data structure that holds these operations. It looks like the following:

```
struct file_operations {
   struct module *owner;
   loff_t (*llseek) (struct file *, loff_t, int);
   ssize_t (*read) (struct file *, char __user *,
                     size_t, loff_t *);
   ssize_t (*write) (struct file *, const char __user *,
                     size_t, loff_t *);
   unsigned int (*poll) (struct file *,
                         struct poll_table_struct *);
   int (*mmap) (struct file *, struct vm_area_struct *);
   int (*open) (struct inode *, struct file *);
   int (*flush) (struct file *, fl_owner_t id);
   long (*unlocked_ioctl) (struct file *, unsigned int,
                           unsigned long);
   int (*release) (struct inode *, struct file *);
   int (*fsync) (struct file *, loff_t, loff_t,
                 int datasync);
   int (*flock) (struct file *, int, struct file_lock *);
   [...]
};
```

The preceding excerpt lists only the important methods of the structure, especially the ones that are relevant to the needs of this book. The full code is in `include/linux/fs.h` in kernel sources. Each of these callbacks is the backend of a system call, and none of them are mandatory. The following explains the meanings of elements in the structure:

*   `struct module *owner`: This is a mandatory field that should point to the module owning this structure. It is used for proper reference counting. Most of the time, it is set to `THIS_MODULE`, a macro defined in `<linux/module.h>`.
*   `loff_t (*llseek) (struct file *, loff_t, int);`: This method is used to move the current cursor position in the file given as the first parameter. On a successful move, the function must return the new position, or else a negative value must be returned. If this method is not implemented, then every seek performed on this file will succeed by modifying the position counter in the `file` structure (`file->f_pos`), except the seek relative to end-of-file, which will fail.
*   `ssize_t (*read) (struct file *, char *, size_t, loff_t *);`: The role of this function is to retrieve data from the device. Since the return value is a "signed size" type, this function must return either the number (positive) of bytes successfully read, or else return an appropriate negative code on error. If this function is not implemented, then any `read()` system call on the device file will fail, returning with `-EINVAL` (an "invalid argument").
*   `ssize_t (*write) (struct file *, const char *, size_t, loff_t *);`: The role of this function is to send data to the device. Like the `read()` function, it must return a positive number, which, in this case, represents the number of bytes that have been written successfully, or else return an appropriately negative code on error. In the same way, if it is not implemented in the driver, then the `write()` system call attempt will fail with `-EINVAL`.
*   `int (*flush) (struct file *, fl_owner_t id);`: This operation is invoked when the file structure is being released. Like `open`, `release` can be `NULL`.
*   `unsigned int (*poll) (struct file *, struct poll_table_struct *);`: This file operation must return a bitmask describing the status of the device. It is the kernel backend for both `poll()` and `select()` system calls, both used to query whether the device is writable, readable, or in some special state. Any caller of this method will block until the device enters the requested state. If this file operation is not implemented, then the device is always assumed to be readable, writable, and in no special state.
*   `int (*mmap) (struct file *, struct vm_area_struct *);`: This is used to request part or all of the device memory to be mapped to a process address space. If this file operation is not implemented, then any attempt to invoke the `mmap()` system call on the device file will fail, returning `-ENODEV`.
*   `int (*open) (struct inode *, struct file *);` This file operation is the backend of the `open()` system call, which, if not implemented (if `NULL`), will result in the success of any attempt to open the device and the driver won't be notified of the operation.
*   `int (*release) (struct inode *, struct file *);`: This is invoked when the file is being released, in response to the `close()` system call. Like `open`, `release` is not mandatory and can be `NULL`.
*   `int (*fsync) (struct file *, loff_t, loff_t, int datasync);`: This operation is the backend of the `fsync()` system call, whose purpose is to flush any pending data. If it is not implemented, any call to `fsync()` on the device file will fail, returning `-EINVAL`.
*   `long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);`: This is the backend of the `ioctl` system call, whose purpose is to extend the commands that can be sent to the device (such as formatting a track of a floppy disk, which is neither reading nor writing). The commands defined by this function will extend a set of predefined commands that are already recognized by the kernel without referring to this file operation. Thus, for any command that is not defined (either because this function is not implemented or because it does not support the specified command), the system call will return `-ENOTTY`, to say *"No such ioctl for device"*. Any non-negative value returned by this function is passed back to the calling program to indicate successful completion.

Now that we are familiar with the file operation callbacks, let's delve into the kernel insights and learn how files are handled for a better understanding of the mechanisms behind character devices.

## File representation in the kernel

Looking at the file operation table, at least one of the parameters of each operation is either the `struct inode` or `struct file` type. `struct inode` refers to a file on the disk. However, to refer an open file (associated with a file descriptor within a process), the `struct file` structure is used.

The following is the declaration of an `inode` structure:

```
struct inode {
    [...]
    union {
        struct pipe_inode_info  *i_pipe;
        struct cdev    *i_cdev;
        char           *i_link;
        unsigned       i_dir_seq;
    };
    [...]
}
```

The most important field in the structure is the `union`, especially the `i_cdev` element, which is set when the underlying file is a character device. This makes it possible to switch back and forth between `struct inode` and `struct cdev`.

On the other hand, `struct file` is a filesystem data structure holding information about a file (its type, character, block, pipe, and so on), most of which is only relevant to the OS. The `struct file` structure (defined in `include/linux/fs.h`) has the following definition:

```
struct file {
[...]
   struct path f_path;
   struct inode *f_inode;
   const struct file_operations *f_op;
   loff_t f_pos;
   void *private_data;
[...]
}
```

In the preceding data structure, `f_path` represents the actual path of the file in the filesystem, and `f_inode` is the underlying `inode` that points to this opened file. This makes it possible to switch back and forth between `struct file` and the underlying `cdev`, through the `f_inode` element. `f_op` represents the file operation table. Because `struct file` represents an open file descriptor, it tracks the current read/write position within its opened instance. This is done through the `f_pos` element, and `f_pos` is the current read/write position.

# Creating a device node

The creation of a device node makes it visible to users and allows users to interact with the underlying device. Linux requires intermediate steps before the device node is created and the following section discusses these steps.

## Device identification

To precisely identify devices, their identifiers must be unique. Although identifiers can be dynamically allocated, most drivers still use static identifiers for compatibility reasons. Whatever the allocation method, the Linux kernel stores file device numbers in elements of `dev_t` type, which is a 32-bit unsigned integer in which the major is represented by the first `12` bits, and the minor is coded on the `20` remaining bits.

All of this is stated in `include/linux/kdev_t.h`, which contains several macros, including those that, given a `dev_t` type variable, can return either a minor or a major number:

```
#define MINORBITS    20
#define MINORMASK    ((1U << MINORBITS) - 1)
#define MAJOR(dev)    ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev)    ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi)  (((ma) << MINORBITS) | (mi))
```

The last macro accepts a minor and a major number and returns a `dev_t` type identifier, which the kernel uses to keep identifiers. The preceding excerpt also describes how the `character device` identifier is built using a bit shift. At this point, we can get deep into the code, using the APIs that the kernel provides for code allocation.

## Registration and deregistration of character device numbers

There are two ways to deal with device numbers – **registration** (the static method) and **allocation** (the dynamic method). Registration, also called **static allocation**, is only useful if you know in advance which major number you want to start with, after making sure it does not clash with another driver using the same major (though this is not always predictable). Registration is a brute-force method in which you let the kernel know what device numbers you want by providing the starting major/minor pair and the number of minors, and it either grants them to you or not (depending on availability). The function to use for device number registration is the following:

```
int register_chrdev_region(dev_t first, unsigned int count,
                           char *name);
```

This method returns `0` on success, or a negative error code when it fails. The `first` parameter is the identifier you must have built using the major number and the first minor of the desired range. You can use the `MKDEV(maj, min)` macro to achieve that. `count` is the number of consecutive device minors required, and `name` should be the name of the associated device or driver.

However, note that `register_chrdev_region()` works well if you know exactly which device numbers you want, and of course, those numbers must be available on your running system. Because this can be a source of conflict with other device drivers, it is considered preferable to use dynamic allocation, with which the kernel happily allocates a major number for you on the fly. `alloc_chrdev_region()` is the API you must use for dynamic allocation. The following is its prototype:

```
int alloc_chrdev_region(
                      dev_t *dev, unsigned int firstminor,
                      unsigned int count, char *name);
```

This method returns `0` on success, or a negative error code on failure. `dev` is the only output parameter. It represents the first number (built using the allocated major and the first minor requested) that the kernel assigned. `firstminor` is the first of the requested range of minor numbers, `count` is the number of consecutive minors you need, and `name` should be the name of the associated device or driver.

The difference between static allocation and dynamic allocation is that, with the former, you should know in advance which device number is needed. When it comes to loading the driver on another machine, there is no guarantee that the chosen number is free on that machine, and this can lead to conflict and trouble. New drivers are encouraged to use dynamic allocation to obtain a major device number, rather than choosing a number randomly from the ones that are currently free, which will probably lead to a clash. In other words, your drivers are better using `alloc_chrdev_region()` rather than `register_chrdev_region()`.

## Initializing and registering a character device on the system

The registration of a character device is made by specifying a device identifier (of `dev_t` type). In this chapter, we will be using dynamic allocation, using `alloc_chrdev_region()`. After the identifier has been allocated, you must initialize the character device and add it to the system using `cdev_init()` and `cdev_add(),` respectively. The following are their prototypes:

```
void cdev_init(struct cdev *cdev,
               const struct file_operations *fops);
int cdev_add (struct cdev * p, dev_t dev, unsigned count);
```

In `cdev_init()`, `cdev` is the structure to initialize, and `fops` the `file_operations` instance for this device, making it ready to add to the system. In `cdev_add()`, `p` is the `cdev` structure for the device, `dev` is the first device number for which this device is responsible (obtained dynamically), and `count` is the number of consecutive minor numbers corresponding to this device. When it succeeds, `cdev_add()` returns `0`, or else it returns a negative error code.

The respective reverse operation of `cdev_add()` is `cdev_del()`, which removes the character device from the system and has the following prototype:

```
void cdev_del(struct cdev *);
```

At this step, the device is part of the system but not physically present. In other words, it is not visible in `/dev` yet. For the node to be created, you must use `device_create()`, which has the following prototype:

```
struct device * device_create(struct class *class,
                              struct device *parent,
                              dev_t devt,
                              void *drvdata, 
                              const char *fmt, ...)
```

This method creates a device and registers it with Sysfs. In its argument, `class` is a pointer to `struct class` that this device should be registered to, `parent` is a pointer to the `struct device` parent of this new device if there is any, `devt` is the device number for the char device to be added, and `drvdata` is the data to be added to the device for callbacks.

Important Note

In case of multiple minors, the `device_create()` and `device_destroy()` APIs can be put in a `for` loop, and the `<device name format>` string can be appended with the loop counter, as follows:

`device_create(class, NULL, MKDEV(MAJOR(first_devt), MINOR(first_devt) + i), NULL, "mynull%d", i);`

Because a device needs an existing class before being created, you must either create a class or use an existing one. For now, we will create a class, and to do that, we need to use the `class_create()` function, declared as the following:

```
struct class * class_create(struct module * owner,
                            const char * name);
```

After then, the class will be visible in `/sys/class`, and we can create the device using that class. The following is a rough example:

```
#define EEP_NBANK 8
#define EEP_DEVICE_NAME "eep-mem"
#define EEP_CLASS "eep-class"
static struct class *eep_class;
static struct cdev eep_cdev[EEP_NBANK];
static dev_t dev_num;
static int __init my_init(void)
{
    int i;
    dev_t curr_dev;
    /* Request for a major and EEP_NBANK minors */
    alloc_chrdev_region(&dev_num, 0, EEP_NBANK, 
                        EEP_DEVICE_NAME);
    /* create our device class, visible in /sys/class */
    eep_class = class_create(THIS_MODULE, EEP_CLASS);
    /* Each bank is represented as a character device (cdev) */
    for (i = 0; i < EEP_NBANK; i++) {
        /* bind file_operations to the cdev */
        cdev_init(&my_cdev[i], &eep_fops);
        eep_cdev[i].owner = THIS_MODULE;
        /* Device number to use to add cdev to the core */
        curr_dev = MKDEV(MAJOR(dev_num),
                          MINOR(dev_num) + i);
        /* Make the device live for the users to access */
        cdev_add(&eep_cdev[i], curr_dev, 1);
        /* create a node for each device */
        device_create(eep_class,
              NULL,     /* no parent device */
              curr_dev,
              NULL,     /* no additional data */
              EEP_DEVICE_NAME "%d", i); /* eep-mem[0-7] */
    }
    return 0;
}
```

In the preceding code, `device_create()` will create a node for each device, – `/dev/eep-mem0`, `dev/eep-mem1`, and so on, with our class represented by `eep_class`. Additionally, devices can also be viewed under `/sys/class/eep-class`. In the meantime, the reverse operation is the following:

```
for (i = 0; i < EEP_NBANK; i++) {
    device_destroy(eep_class,
              MKDEV(MAJOR(dev_num), (MINOR(dev_num) +i)));
    cdev_del(&eep_cdev[i]);
}
class_unregister(eep_class);
class_destroy(eep_class);
unregister_chrdev_region(chardev_devt, EEP_NBANK);
```

In the preceding code, `device_destroy()` will remove a device node from `/dev`, `cdev_del()` will make the system forget about this character device, `class_unregister()` and `class_destroy()` will deregister and remove the class from the system, and finally, `unregister_chrdev_region()` will release our device number.

Now that we are familiar with all the prerequisites about character devices, we can start implementing a file operation, which allows users to interact with the underlying device.

# Implementing file operations

After introducing file operations in the previous section, it is time to implement those to enhance the driver capabilities and expose the device's methods to user space (by means of system calls, of course). Each of these methods has its particularities, which we will highlight in this section.

## Exchanging data between the kernel space and user space

As we have seen while introducing the file operation table, the `read` and `write` methods are used to exchange data with the underlying device. Both being system calls means that data will originate from or be in destination to user space. While looking at the `read` and `write` method prototypes, the first point that catches our attention is the use of `__user`. This is a cookie used by **Sparse** (a semantic checker used by the kernel to find possible coding faults) to let the developer know they are about to use an untrusted pointer (or a pointer that may be invalid in the current virtual address mapping) improperly, which they should not dereference but, instead, use dedicated kernel functions to access the memory to which this pointer points.

This leads us to two principal functions that allow us to exchange data between a kernel and user space, `copy_from_user()` and `copy_to_user()`, which copy a buffer from user space to kernel space and vice versa, respectively:

```
unsigned long copy_from_user(void *to,
               const void __user *from, unsigned long n)
unsigned long copy_to_user(void __user *to,
               const void *from, unsigned long n)
```

In both cases, pointers prefixed with `__user` point to the user space (untrusted) memory. `n` represents the number of bytes to copy, either to or from user space. `from` represents the source address, and `to` is the destination address. Each of these returns the number of bytes that could not be copied, if any, while they return `0` on success. Note that these routines may sleep as they run in a user context and do not need to be invoked in an atomic context.

## Implementing the open file operation

The `open` file operation is the backend of the open system call. One usually uses this method to perform device and data structure initializations, after which it should return `0` on success, or a negative error code if something went wrong. The prototype of the `open` file operation is defined as follows:

```
int (*open) (struct inode *inode, struct file *filp);
```

If it is not implemented, device opening will always succeed, but the driver won't be aware, which is not necessarily a problem if the device needs no special initialization.

### Per-device data

As we have seen in file operation prototypes, there is almost always a `struct file` argument. `struct file` has an element free of use, that is `private_data`. `file->private_data`, if set, will be available to other system calls invoked on the same file descriptor. You can use this field during the lifetime of the file descriptor. It is good practice to set this field in the `open` method as it is always the first system call on any file.

The following is our data structure:

```
struct pcf2127 {
    struct cdev cdev;
    unsigned char *sram_data;
    struct i2c_client *client;
    int sram_size;
    [...]
};
```

Given this data structure, the `open` method would look like the following:

```
static unsigned int sram_major = 0;
static struct class *sram_class = NULL;
static int sram_open(struct inode *inode,
                     struct file *filp)
{
    unsigned int maj = imajor(inode);
    unsigned int min = iminor(inode);
    struct pcf2127 *pcf = NULL;
    pcf = container_of(inode->i_cdev,
                        struct pcf2127, cdev);
    pcf->sram_size = SRAM_SIZE;
    if (maj != sram_major || min < 0 ){
        pr_err ("device not found\n");
        return -ENODEV; /* No such device */
    }
    /* prepare the buffer if the device is
     * opened for the first time
       */
    if (pcf->sram_data == NULL) {
        pcf->sram_data =
                  kzalloc(pcf->sram_size, GFP_KERNEL);
        if (pcf->sram_data == NULL) {
            pr_err("memory allocation failed\n");
            return -ENOMEM;
        }
    }
    filp->private_data = pcf;
    return 0;
}
```

Most of the time, the `open` operation does some initialization and requests resources that will be used while a user keeps an open instance of the device node. Everything that has to be done in this operation must be undone and released when the device is closed, as we'll see in the next operation.

## Implementing the release file operation

The `release` method is called when the device gets closed, the reverse of the `open` method. You must then undo everything you have done in the `open` operation. It could be literally, freeing any private memory allocated and shutting down the device (if supported), and discarding every buffer on the last closing (if the device supports multi-opening, or if the driver can handle more than one device at a time).

The following is an excerpt of a `release` function:

```
static int sram_release(struct inode *inode,
                        struct file *filp)
{
    struct pcf2127 *pcf = NULL;
    pcf = container_of(inode->i_cdev,
                        struct pcf2127, cdev);
    mutex_lock(&device_list_lock);
    filp->private_data = NULL;
    /* last close? */
    pcf2127->users--;
    if (!pcf2127->users) {
        kfree(tx_buffer);
        kfree(rx_buffer);
        tx_buffer = NULL;
        rx_buffer = NULL;
        [...]
        if (any_other_dynamic_struct)
            kfree(any_other_dynamic_struct);
    }
    mutex_unlock(&device_list_lock);
    return 0;
}
```

The preceding code releases all the resources acquired when the device node has been open. This is literally all that needs to be done in this file operation. If the device node is backed by a hardware device, this operation can also put this device in the appropriate state.

At this point, we are able to implement the entry (`open`) and exit (`release)` points for the character device. All that remains now is to implement each possible operation that can be done in between.

## Implementing the write file operation

The `write` method is used to send data to the device; whenever a user calls the `write()` system call on the device's file, the kernel implementation that is called ends up being invoked. Its prototype is as follows:

```
ssize_t(*write)(struct file *filp, const char __user *buf,
                size_t count, loff_t *pos);
```

This file operation must return the number of bytes (size) written, and the following are the definitions of its arguments:

*   `*buf` represents the data buffer coming from the user space.
*   `count` is the size of the requested transfer.
*   `*pos` indicates the start position from which data should be written in the file (or in the corresponding memory region if the character device file is memory-backed).

Generally, in this file operation, the first thing to do is to check for bad or invalid requests coming from the user space (for example, check for size limitations in case of a memory-backed device and size overflow). The following is an example:

```
/* if trying to Write beyond the end of the file,
 * return error. "filesize" here corresponds to the size
 * of the device memory (if any)
 */
if (*pos >= filesize) return –EINVAL;
```

After the checks, it is common to make some adjustments, especially with `count,` to not go beyond the file size. This step is not mandatory either:

```
/* filesize corresponds to the size of device memory */
if (*pos + count > filesize) 
    count = filesize - *pos;
```

The next step is to find the location from which you will start to write. This step is relevant only if the device is backed by physical memory in which the `write()` method is supposed to store given data:

```
/* convert pos into valid address */
void *from = pos_to_address(*pos); 
```

Finally, you can copy data from user space into kernel memory, after which you can perform the `write` operation on the backing device and adjust `*pos`, as in the following excerpt:

```
if (copy_from_user(dev->buffer, buf, count) != 0){
    retval = -EFAULT;
    goto out;
}
/* now move data from dev->buffer to physical device */
write_error = device_write(dev->buffer, count);
if (write_error)
    return –EFAULT;
/* Increase the current position of the cursor in the file,
 * according to the number of bytes written and finally,
 * return the number of bytes copied
 */
*pos += count;
return count;
```

The following is an example of the `write` method, which summarizes the steps described so far:

```
ssize_t 
eeprom_write(struct file *filp, const char __user *buf,
             size_t count, loff_t *f_pos)
{
    struct eeprom_dev *eep = filp->private_data;
    int part_origin = PART_SIZE * eep->part_index;
    int register_address;
    ssize_t retval = 0;
    /* step (1) */
    if (*f_pos >= eep->part_size) 
        /* Can't write beyond the end of a partition. */
        return -EINVAL;
    /* step (2) */
    if (*pos + count > eep->part_size)
        count = eep->part_size - *pos;
    /* step (3) */
    register_address = part_origin + *pos;
    /* step(4) */
    /* Copy data from user space to kernel space */
    if (copy_from_user(eep->data, buf, count) != 0)
        return -EFAULT;
    /* step (5) */
    /* perform the write to the device */
    if (write_to_device(register_address, buff, count)
        < 0){
        pr_err("i2c_transfer failed\n");  
        return –EFAULT;
     }
    /* step (6) */
    *f_pos += count;
    return count;
}
```

After the data has been read and processed, it might be necessary to write the processing output back. Since we have started with writing, the next operation we may think of is `read`, as we will see in the next section.

## Implementing the read file operation

The `read` method has the following prototype:

```
ssize_t (*read) (struct file *filp, char __user *buf,
                 size_t count, loff_t *pos);
```

This operation is the backend of the `read()` system call. Its arguments are described as follows:

*   `*buf` is the buffer we receive from user space.
*   `count` is the size of the requested transfer (the size of the user buffer).
*   `*pos` indicates the start position from which data should be read in the file.

It must return the size of the data that has been successfully read. This size can be less than `count` though (for example, when reaching the end of the file before reaching the `count` requested by the user).

Implementing the read operation looks like the write one, since some sanity checks need to be performed. First, you can prevent reading beyond the file size and return an end-of-file response:

```
if (*pos >= filesize)
    return 0; /* 0 means EOF */
```

Then, you should make sure the number of bytes read can't go beyond the file size and you can adjust `count` appropriately:

```
if (*pos + count > filesize)
    count = filesize – (*pos);
```

Next, you can find the location from which you will start the read, after which you can copy the data into the user space buffer and return an error on failure, and then advance the file's current position according to the number of bytes read and return the number of bytes copied:

```
/* convert pos into valid address */
void *from = pos_to_address (*pos); 
sent = copy_to_user(buf, from, count);
if (sent)
    return –EFAULT;
*pos += count;
return count;
```

The following is an example of a driver `read()` file operation, intended to give an overview of what can be done here:

```
ssize_t  eep_read(struct file *filp, char __user *buf,
                  size_t count, loff_t *f_pos)
{
    struct eeprom_dev *eep = filp->private_data;
    if (*f_pos >= EEP_SIZE) /* EOF */
        return 0;
    if (*f_pos + count > EEP_SIZE)
        count = EEP_SIZE - *f_pos;
    /* Find location of next data bytes */
    int part_origin  =  PART_SIZE * eep->part_index;
    int eep_reg_addr_start  =  part_origin + *pos;
    /* perform the read from the device */
    if (read_from_device(eep_reg_addr_start, buff, count)
        < 0){
        pr_err("i2c_transfer failed\n");  
        return –EFAULT;
    } 
    /* copy from kernel to user space */
    if(copy_to_user(buf, dev->data, count) != 0)
        return -EIO;
    *f_pos += count;
    return count;
}
```

Though reading and writing data moves the cursor position, there is an operation whose main purpose is to move the cursor position without touching data at all. Such an operation helps to start writing or reading data from anywhere by moving the cursor to the desired position.

## Implementing the llseek file operation

The `llseek` file operation is the kernel backend for the `lseek()` system call, used to move the cursor position within a file. Its prototype looks as follows:

```
loff_t(*llseek) (struct file *filp, loff_t offset,
                 int whence);
```

This callback must return the new position in the file. The following are the definitions of its parameters:

*   `loff_t` is an offset, relative to the current file position, which defines how much of it will be changed.
*   `whence` defines where to seek from. The possible values are as follows:
    *   `SEEK_SET`: To put the cursor to a position relative from the beginning of the file
    *   `SEEK_CUR`: To put the cursor to a position relative to the current file position
    *   `SEEK_END`: To adjust the cursor to a position relative to the end of the file

When implementing this operation, it is a good practice to use the `switch` statement to check every possible `whence` case, since they are limited, and adjust the new position accordingly:

```
switch( whence ){
    case SEEK_SET:/* relative from the beginning of file */
        newpos = offset; /* offset become the new position */
        break;
    case SEEK_CUR: /* relative to current file position */
        /* just add offset to the current position */
        newpos = file->f_pos + offset;
        break;
    case SEEK_END: /* relative to end of file */
        newpos = filesize + offset;
        break;
    default:
        return -EINVAL;
}
/* Check whether newpos is valid **/
if ( newpos < 0 )
    return –EINVAL;
/* Update f_pos with the new position */
filp->f_pos = newpos;
/* Return the new file-pointer position */
return newpos;
```

After the preceding kernel backend excerpt, the following is an example of a user program that will successively read and seek into a file. The underlying driver will then execute the `llseek()` file operation entry:

```
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <stdio.h>
#define CHAR_DEVICE "foo"
int main(int argc, char **argv)
{
    int fd = 0;
    char buf[20];
    if ((fd = open(CHAR_DEVICE, O_RDONLY)) < -1)
        return 1;
    /* Read 20 bytes */
    if (read(fd, buf, 20) != 20)
        return 1;
    printf("%s\n", buf);
    /* Move the cursor to ten time relative to
     * its actual position
     */
    if (lseek(fd, 10, SEEK_CUR) < 0)
        return 1;
    if (read(fd, buf, 20) != 20) 
        return 1;
    printf("%s\n",buf);
    /* Move the cursor seven time, relative from
     * the beginning of the file
     */
    if (lseek(fd, 7, SEEK_SET) < 0)
        return 1;
    if (read(fd, buf, 20) != 20)
        return 1;
    printf("%s\n",buf);
    close(fd);
    return 0;
}
```

The code produces the following output:

```
jma@jma:~/work/tutos/sources$ cat toto 
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
jma@jma:~/work/tutos/sources$ ./seek 
Lorem ipsum dolor si
nsectetur adipiscing
psum dolor sit amet,
jma@jma:~/work/tutos/sources$
```

In this section, we have explained the concept of seeking with an example, showing how relative and absolute seeking works. Now that we are done with operations moving data around, we can switch to the next operation, sensing the readability or writability of data in a character device.

## The poll method

The `poll` method is the backend of both the `poll()` and `select()` system calls. These system calls are used to passively (by sleeping, without wasting CPU cycles) sense the readability/writability in a file. To support these system calls, the driver must implement `poll`, which has the following prototype:

```
unsigned int (*poll) (struct file *, struct poll_table_struct *);
```

The kernel function at the heart of this method implementation is `poll_wait()`, defined in `<linux/poll.h>`, which is the header you must include in the driver code. It has the following declaration:

```
void poll_wait(struct file * filp,
               wait_queue_head_t * wait_address, poll_table *p)
```

`poll_wait()` adds the device associated with a `struct file` structure (given as the first parameter) to a list of those that can wake up processes (put to sleep in the `struct wait_queue_head_t` structure given as the second parameter), according to events registered in the `struct poll_table` structure given as the third parameter. A user process can call the `poll()`, `select()`, or `epoll()` system calls to add a set of files to a list on which it needs to wait, in order to be aware of the associated (if any) device's readiness. The kernel will then call the `poll` entry of the driver associated with each device file. The `poll` method of each driver should then call `poll_wait()` in order to register events for which the process needs to be notified with the kernel, put that process to sleep until one of these events occurs, and register the driver as one of those that can wake the process up. The usual way is to use a wait queue per event type (one for readability, another one for writability, and eventually one for an exception if needed), according to events supported by the `select()` (or `poll()`) system call.

The return value of the (`*poll`) file operation must have `POLLIN | POLLRDNORM` set if there is data to read, `POLLOUT | POLLWRNORM` if the device is writable, and `0` if there is no new data and the device is not yet writable. In the following example, we assume the device supports both blocking read and write. Of course, you can implement only one of these. If the driver does not define this method, the device will be considered as always readable and writable, and `poll()` or `select()` system calls return immediately.

Implementing the poll operation may require adapting the read or write file operations in a way that, on write, readers are notified of the readability, and on read, writers are notified of the writability:

```
#include <linux/poll.h>
/* declare a wait queue for each event type (read, write ...) */
static DECLARE_WAIT_QUEUE_HEAD(my_wq);
static DECLARE_WAIT_QUEUE_HEAD(my_rq);
static unsigned int eep_poll(struct file *file,
                             poll_table *wait)
{
    unsigned int reval_mask = 0;
    poll_wait(file, &my_wq, wait);
    poll_wait(file, &my_rq, wait);
    if (new_data_is_ready)
        reval_mask |= (POLLIN | POLLRDNORM);
    if (ready_to_be_written)
       reval_mask |= (POLLOUT | POLLWRNORM);
    return reval_mask;
}
```

In the preceding snippet, we have implemented the `poll` operation, which can put processes to sleep if a device is not writable or readable. However, there is no notification mechanism when any of those states change. Therefore, a `write` operation (or any operation making data available, such as an IRQ) must notify processes sleeping in the readability wait queue; the same applies to the `read` operation (or any operation making the device ready to be writable), which must notify processes sleeping in the writability wait queue. The following is an example:

```
wake_up_interruptible(&my_rq); /* Ready to read */
/* set flag accordingly in case poll is called */
new_data_is_ready = true;
wake_up_interruptible(&my_wq); /* Ready to be written to */
ready_to_be_written = true;
```

More precisely, you can notify a readable event either from within the driver's `write()` method, meaning that the written data can be read back, or from within an IRQ handler, meaning that an external device sent some data that can be read back. On the other hand, you can notify a writable event either from within the driver's `read()` method, meaning that the buffer is empty and can be filled again, or from within an IRQ handler, meaning that the device has completed a data-send operation and is ready to accept data again. Do not forget to set flags back to `false` when the state changes.

The following is an excerpt of code that uses `select()` on a given character device in order to sense data availability:

```
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/select.h>
#define NUMBER_OF_BYTE 100
#define CHAR_DEVICE "/dev/packt_char"
char data[NUMBER_OF_BYTE];
int main(int argc, char **argv)
{
    int fd, retval;
    ssize_t read_count;
    fd_set readfds;
    fd = open(CHAR_DEVICE, O_RDONLY);
    if(fd < 0)
        /* Print a message and exit*/
        [...]
    while(1){ 
        FD_ZERO(&readfds);
        FD_SET(fd, &readfds);
       ret = select(fd + 1, &readfds, NULL, NULL, NULL);
        /* From here, the process is already notified */
        if (ret == -1) {
            fprintf(stderr, "select: an error ocurred");
            break;
        }

        /* we are interested in one file only */
        if (FD_ISSET(fd, &readfds)) {
            read_count = read(fd, data, NUMBER_OF_BYTE);
            if (read_count < 0)
                /* An error occurred. Handle this */
                [...]
            if (read_count != NUMBER_OF_BYTE)
                /* We have read less than needed bytes */
                [...] /* handle this */
            else
            /* Now we can process the data we have read */
            [...]
        }
    }    
    close(fd);
    return EXIT_SUCCESS;
}
```

In the preceding code sample, we used `select()` without timeout, in a way that means we will be notified of "read" events only. From that line, the process is put to sleep until it is notified of the event for which it registered itself.

## The ioctl method

A typical Linux system contains around 350 `-ENOTTY` error to any `ioctl()` system call. The following is its prototype:

```
long ioctl(struct file *f, unsigned int cmd,
           unsigned long arg);
```

In the preceding prototype, `f` is the pointer to the file descriptor representing an opened instance of the device, `cmd` is the ioctl command, and `arg` is a user parameter, which can be the address of any user memory on which the driver can call `copy_to_user()` or `copy_from_user()`. To be concise, and for obvious reasons, an IOCTL command needs to be identified by a number, which should be unique to the system. The unicity of IOCTL numbers across the system will prevent us from sending the right command to the wrong device, or passing the wrong argument to the right command (with a duplicated IOCTL number). Linux provides four helper macros to create an IOCTL identifier, depending on whether there is data transfer or not and the direction of the transfer. Their respective prototypes are as follows:

```
_IO(MAGIC, SEQ_NO)
_IOR(MAGIC, SEQ_NO, TYPE)
_IOW(MAGIC, SEQ_NO, TYPE)
_IORW(MAGIC, SEQ_NO, TYPE)
```

Their descriptions are as follows:

*   `_IO`: The IOCTL command does not need data transfer.
*   `_IOR`: This means that we're creating an IOCTL command number for passing information from the kernel to user space (which is reading data). The driver will be allowed to return the `sizeof(TYPE)` bytes to the user without this return value being considered as an error.
*   `_IOW`: This is the same as `_IOR`, but the user sends data to the driver this time.
*   `_IOWR`: The IOCTL command needs both write and read parameters.

What their parameters mean (in the order they are passed) is described here:

*   A number coded on 8 bits (`0` to `255`), called the **magic number**.
*   A sequence number or command ID, also on 8 bits.
*   A data type, if any, that will inform the kernel about the size to be copied. This could be the name of a structure or a data type.

This is well documented in `Documentation/ioctl/ioctl-decoding.txt` in the kernel sources, and existing IOCTL commands are listed in `Documentation/ioctl/ioctl-number.txt`, a good place to start when you need to create your own IOCTL commands.

### Generating an IOCTL number (a command)

It is recommended to generate your own IOCTL numbers in a dedicated header file, since this header should be available in user space as well. In other words, you should handle the duplication (by means of symbolic links, for example) of the IOCTL header file so that there is one in the kernel and one in user space, which can be included in user apps. Let's now generate some IOCTL numbers in a real example, and let's call this header `eep_ioctl.h`:

```
#ifndef PACKT_IOCTL_H
#define PACKT_IOCTL_H
/* We need to choose a magic number for our driver,
 * and sequential numbers for each command:
 */
#define EEP_MAGIC 'E'
#define ERASE_SEQ_NO 0x01
#define RENAME_SEQ_NO 0x02
#define GET_FOO 0x03
#define GET_SIZE 0x04
/*
 * Partition name must be 32 byte max
 */
#define MAX_PART_NAME 32
/*
 * Now let's define our ioctl numbers:
 */
#define EEP_ERASE _IO(EEP_MAGIC, ERASE_SEQ_NO)
#define EEP_RENAME_PART _IOW(EEP_MAGIC, RENAME_SEQ_NO, \
                             unsigned long)
#define EEP_GET_FOO  _IOR(EEP_MAGIC, GET_FOO, \
                          struct my_struct *)
#define EEP_GET_SIZE _IOR(EEP_MAGIC, GET_SIZE, int *)
#endif
```

After the commands have been defined, the header needs to be included in the final code. Moreover, because they are all unique and limited, it is a good practice to use a `switch ... case` statement to handle each command and return a `-ENOTTY` error code when an undefined `ioctl` command is called. The following is an example:

```
#include "eep_ioctl.h"
static long eep_ioctl(struct file *f, unsigned int cmd,
                      unsigned long arg)
{
    int part;
    char *buf = NULL;
    int size = 2048;
    switch(cmd){
        case EEP_ERASE:
            erase_eepreom();
            break;
        case EEP_RENAME_PART:
            buf = kmalloc(MAX_PART_NAME, GFP_KERNEL);
            copy_from_user(buf, (char *)arg,
                            MAX_PART_NAME);
            rename_part(buf);
            break;
        case EEP_GET_SIZE:
            if (copy_to_user((int*)arg,
                           &size, sizeof(int)))
                return -EFAULT;
            break;
        default:
            return –ENOTTY;
    }
    return 0;
}
```

Both the kernel and user space must include the header files that contain the IOCTL commands. Therefore, in the first line of the preceding excerpt, we have included `eep_ioctl.h`, which is the header file where our IOCTL commands are defined.

If you think your IOCTL command will need more than one argument, you should gather those arguments in a structure and just pass a pointer to the structure to `ioctl`.

Now, from the user space, you must use the same `ioctl` header as in the driver's code:

```
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include "eep_ioctl.h" /* our ioctl header file */
int main()
{
    int size = 0;
    int fd;
    char *new_name = "lorem_ipsum";
    fd = open("/dev/eep-mem1", O_RDWR);
    if (fd < 0){
        printf("Error while opening the eeprom\n");
        return 1;
    }
    /* ioctl to erase partition */
    ioctl(fd, EEP_ERASE);
    /* call to get partition size */
    ioctl(fd, EEP_GET_SIZE, &size);
    /* rename partition */
    ioctl(fd, EEP_RENAME_PART, new_name);  
    close(fd);
    return 0;
}
```

In the preceding code, we have demonstrated the use of kernel IOCTL commands from user space. That said, all throughout this section, we have learned how to implement the character device's `ioctl` callback and how to exchange data between the kernel and user space.

# Summary

In this chapter, we have demystified character devices, and we have seen how to let users interact with our driver through device files. We learned how to expose file operations to user space and control their behavior from within the kernel. We went so far that you are even able to implement multi-device support.

The next chapter is a bit more hardware-oriented, as it deals with the device tree, a mechanism that allows hardware devices present on the system to be declared to the kernel. See you in the next chapter.