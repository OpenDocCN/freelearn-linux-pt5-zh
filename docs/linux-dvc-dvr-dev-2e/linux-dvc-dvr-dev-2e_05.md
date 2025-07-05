# 第四章：*第四章*：编写字符设备驱动程序

基于 Unix 的系统通过特殊文件将硬件暴露给用户空间，这些文件在设备注册到系统后会在`/dev`目录下创建。愿意访问某个设备的程序必须在`/dev`中找到相应的设备文件，并在其上执行适当的系统调用，该系统调用将被重定向到与该特殊文件相关联的底层设备的驱动程序。尽管系统调用的重定向是由操作系统完成的，但支持哪些系统调用取决于设备类型和驱动程序的实现。

关于设备类型，硬件角度上有很多种，但它们被归为`/dev`目录下的两类特殊设备文件——**块设备**和**字符设备**。它们通过访问方式、速度以及数据在它们与系统之间传输的方式来区分。通常，字符设备比较慢，数据传输是按字节顺序从用户应用程序中逐个字节传输的（一个字符接一个字符——因此得名）。这样的设备包括串行端口和输入设备（键盘、鼠标、触控板、视频设备等）。另一方面，块设备比较快，因为它们的访问频繁且数据以块为单位传输。这类设备本质上是存储设备（硬盘、光盘驱动器、固态硬盘等）。

在本章中，我们将重点讨论字符设备及其驱动程序、它们的 API 以及它们的常见数据结构。我们将介绍大部分概念并编写我们的第一个字符设备驱动程序。

本章将涵盖以下主题：

+   主设备号和次设备号的概念

+   字符设备数据结构介绍

+   创建设备节点

+   实现文件操作

# 主设备号和次设备号的概念

Linux 一直通过一个唯一的标识符来强制执行设备文件的识别，该标识符由两部分组成，`/dev`，字符或块设备文件通过其类型来识别，可以通过`ls -l`命令查看：

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

从前面的摘录中，第一列中，`c`标识字符设备文件，`b`标识块设备文件。在第五列和第六列中，我们可以看到主设备号和次设备号。主设备号要么标识设备类型，要么与驱动程序绑定。次设备号要么在驱动程序中本地标识设备，要么标识同类型的设备。这就解释了为什么前面的输出中有些设备文件具有相同的主设备号。

现在我们已经掌握了 Linux 系统中字符设备的基本概念，我们可以开始探索内核代码，从介绍主要数据结构开始。

# 字符设备数据结构介绍

字符设备驱动程序是内核源代码中最基本的设备驱动程序。字符设备在内核中以`struct cdev`实例的形式表示，该结构在`include/linux/cdev.h`中声明：

```
struct cdev {
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops;
    dev_t dev;
[...]
};
```

前面的摘录仅列出了我们感兴趣的元素。以下显示了这些元素在此数据结构中的含义：

+   `kobj`: 这是该字符设备对象的基础内核对象，用于实施 Linux 设备模型。我们将在*第十四章*，*Linux 设备模型介绍*中讨论这一点。

+   `owner`: 应该使用`THIS_MODULE`宏进行设置。

+   `ops`: 这是与此字符设备关联的文件操作集合。

+   `dev`: 这是字符设备标识符。

引入了这个数据结构后，接下来讨论的逻辑下一个数据结构是暴露给系统调用依赖的文件操作。然后让我们介绍允许用户空间与内核空间通过字符设备进行交互的数据结构。

## 设备文件操作简介

`cdev->ops`元素指向给定设备支持的文件操作。每个这些操作都是特定系统调用的目标，当用户空间程序在字符设备上调用系统调用时，内核将此系统调用重定向到其在`cdev->ops`中的文件操作对应项。`struct file_operations`是保存这些操作的数据结构。它看起来像这样：

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

前面的摘录仅列出了结构的重要方法，特别是与本书需求相关的方法。完整代码在内核源码的`include/linux/fs.h`中。每个这些回调函数是系统调用的后端，并且它们都不是强制性的。以下解释了结构中元素的含义：

+   `struct module *owner`: 这是一个必填字段，应指向拥有此结构的模块。它用于正确的引用计数。大多数情况下，它设置为`THIS_MODULE`，这是在`<linux/module.h>`中定义的宏。

+   `loff_t (*llseek) (struct file *, loff_t, int);`: 此方法用于移动文件中当前光标位置，第一个参数是文件。成功移动后，函数必须返回新位置，否则必须返回负值。如果未实现此方法，则在该文件上执行的每个查找都将通过修改文件结构（`file->f_pos`）中的位置计数器成功执行，除了相对于文件末尾的查找将失败。

+   `ssize_t (*read) (struct file *, char *, size_t, loff_t *);`: 此函数的作用是从设备中检索数据。由于返回值是"有符号大小"类型，因此此函数必须返回成功读取的字节数（正数），否则在错误时返回适当的负代码。如果未实现此函数，则对设备文件的任何`read()`系统调用都将失败，并返回`-EINVAL`（"无效参数"）。

+   `ssize_t (*write) (struct file *, const char *, size_t, loff_t *);`：此函数的作用是向设备发送数据。像`read()`函数一样，它必须返回一个正数，表示已成功写入的字节数，否则返回适当的负值代码以指示错误。同样，如果驱动程序中未实现此功能，则尝试执行`write()`系统调用时将失败，并返回`-EINVAL`。

+   `int (*flush) (struct file *, fl_owner_t id);`：当文件结构被释放时，将调用此操作。与`open`类似，`release`可以是`NULL`。

+   `unsigned int (*poll) (struct file *, struct poll_table_struct *);`：此文件操作必须返回一个位掩码，描述设备的状态。它是`poll()`和`select()`系统调用的内核后端，二者用于查询设备是否可写、可读，或是否处于某些特殊状态。调用此方法的任何程序都会阻塞，直到设备进入请求的状态。如果此文件操作未实现，则设备始终被假定为可读、可写且不处于特殊状态。

+   `int (*mmap) (struct file *, struct vm_area_struct *);`：此操作用于请求将设备内存的部分或全部映射到进程的地址空间。如果此文件操作未实现，则对设备文件调用`mmap()`系统调用时会失败，并返回`-ENODEV`。

+   `int (*open) (struct inode *, struct file *);`：此文件操作是`open()`系统调用的后端。如果未实现（即`NULL`），则任何尝试打开设备的操作都会成功，并且驱动程序不会收到此操作的通知。

+   `int (*release) (struct inode *, struct file *);`：当文件被释放时，响应`close()`系统调用时调用此操作。像`open`一样，`release`不是强制性的，可以是`NULL`。

+   `int (*fsync) (struct file *, loff_t, loff_t, int datasync);`：此操作是`fsync()`系统调用的后端，旨在刷新任何待处理的数据。如果未实现，对设备文件的任何`fsync()`调用都会失败，并返回`-EINVAL`。

+   `long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);`：这是`ioctl`系统调用的后端，其目的是扩展可以发送到设备的命令（例如格式化软盘的某个磁道，这既不是读取也不是写入）。由此函数定义的命令将扩展一组已经被内核识别的预定义命令，而无需参考此文件操作。因此，对于任何未定义的命令（无论是因为此函数未实现，还是因为它不支持指定的命令），系统调用将返回`-ENOTTY`，表示*“设备没有此类 ioctl”*。此函数返回的任何非负值都将传回调用程序，以表示成功完成。

现在我们已经熟悉了文件操作回调，让我们深入了解内核的机制，学习文件如何处理，从而更好地理解字符设备背后的机制。

## 内核中的文件表示

查看文件操作表时，每个操作的至少一个参数要么是`struct inode`类型，要么是`struct file`类型。`struct inode`表示磁盘上的一个文件。但是，要引用一个打开的文件（与进程中的文件描述符相关联），则使用`struct file`结构。

以下是`inode`结构的声明：

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

该结构体中最重要的字段是`union`，特别是`i_cdev`元素，当底层文件是字符设备时会设置此元素。这使得在`struct inode`和`struct cdev`之间来回切换成为可能。

另一方面，`struct file`是一个文件系统数据结构，保存有关文件的信息（如文件类型、字符、块、管道等），其中大多数信息仅与操作系统相关。`struct file`结构（定义在`include/linux/fs.h`中）具有以下定义：

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

在上述数据结构中，`f_path`表示文件在文件系统中的实际路径，`f_inode`是指向此打开文件的底层`inode`。这使得通过`f_inode`元素在`struct file`和底层`cdev`之间来回切换成为可能。`f_op`表示文件操作表。由于`struct file`表示一个打开的文件描述符，它追踪其打开实例中的当前读写位置。这是通过`f_pos`元素完成的，`f_pos`是当前的读写位置。

# 创建设备节点

设备节点的创建使其对用户可见，并允许用户与底层设备进行交互。Linux 在创建设备节点之前需要经过一些中间步骤，接下来的部分将讨论这些步骤。

## 设备标识

为了精确标识设备，它们的标识符必须是唯一的。虽然标识符可以动态分配，但大多数驱动程序仍然使用静态标识符以兼容为主。不论分配方式如何，Linux 内核会将文件设备号存储在`dev_t`类型的元素中，它是一个 32 位无符号整数，其中前`12`位表示主设备号，剩余的`20`位表示次设备号。

所有这些内容都在`include/linux/kdev_t.h`中声明，该文件包含了多个宏，其中一些宏可以根据给定的`dev_t`类型变量，返回设备的次设备号或主设备号：

```
#define MINORBITS    20
#define MINORMASK    ((1U << MINORBITS) - 1)
#define MAJOR(dev)    ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev)    ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi)  (((ma) << MINORBITS) | (mi))
```

最后的宏接受一个次设备号和主设备号，返回一个`dev_t`类型的标识符，内核使用这个标识符来存储设备标识符。前面的摘录也描述了如何通过位移构建`字符设备`标识符。到此为止，我们可以深入到代码中，使用内核提供的 API 来进行代码分配。

## 字符设备号的注册与注销

处理设备号的方式有两种——**注册**（静态方法）和**分配**（动态方法）。注册，也叫**静态分配**，只有在你提前知道想要开始使用的主设备号时才有用，前提是确保它不会与另一个驱动程序使用相同的主设备号（虽然这并不总是可预测的）。注册是一种暴力方法，你通过提供起始的主设备号/次设备号对和次设备号的数量，让内核知道你想要哪些设备号，内核会根据可用性来分配它们或不分配。用于设备号注册的函数如下：

```
int register_chrdev_region(dev_t first, unsigned int count,
                           char *name);
```

该方法在成功时返回`0`，失败时返回负错误代码。`first`参数是你必须使用主设备号和所需范围的第一个次设备号构建的标识符。你可以使用`MKDEV(maj, min)`宏来实现。`count`是所需连续设备次设备号的数量，`name`应该是关联的设备或驱动程序的名称。

然而，请注意，`register_chrdev_region()`如果你确切知道想要的设备号，它会表现得很好，当然，这些号必须在你运行的系统中可用。由于这可能与其他设备驱动程序发生冲突，因此建议使用动态分配方式，内核可以自动为你分配一个主设备号。你必须使用`alloc_chrdev_region()`来进行动态分配。以下是它的原型：

```
int alloc_chrdev_region(
                      dev_t *dev, unsigned int firstminor,
                      unsigned int count, char *name);
```

该方法在成功时返回`0`，失败时返回负错误代码。`dev`是唯一的输出参数。它表示内核分配的第一个号（使用分配的主设备号和请求的第一个次设备号构建）。`firstminor`是请求的次设备号范围中的第一个，`count`是你需要的连续次设备号的数量，`name`应该是关联的设备或驱动程序的名称。

静态分配和动态分配的区别在于，前者需要你提前知道所需的设备号。将驱动程序加载到另一台机器上时，无法保证所选设备号在该机器上是空闲的，这可能导致冲突和问题。新驱动程序建议使用动态分配来获取主设备号，而不是从当前空闲的设备号中随机选择，这样可能会导致冲突。换句话说，你的驱动程序最好使用`alloc_chrdev_region()`，而不是`register_chrdev_region()`。

## 在系统上初始化并注册字符设备

字符设备的注册是通过指定设备标识符（`dev_t`类型）来进行的。在本章中，我们将使用动态分配，使用`alloc_chrdev_region()`。标识符分配后，必须使用`cdev_init()`和`cdev_add()`分别初始化字符设备并将其添加到系统中。以下是它们的原型：

```
void cdev_init(struct cdev *cdev,
               const struct file_operations *fops);
int cdev_add (struct cdev * p, dev_t dev, unsigned count);
```

在`cdev_init()`中，`cdev`是要初始化的结构体，`fops`是该设备的`file_operations`实例，使其准备好添加到系统中。在`cdev_add()`中，`p`是设备的`cdev`结构体，`dev`是该设备负责的第一个设备号（动态获取），`count`是与该设备对应的连续次设备号的数量。成功时，`cdev_add()`返回`0`，否则返回负错误代码。

`cdev_add()`的相应反向操作是`cdev_del()`，它将字符设备从系统中移除，具有以下原型：

```
void cdev_del(struct cdev *);
```

在这一步，设备已成为系统的一部分，但尚未物理存在。换句话说，它尚未在`/dev`中可见。为了创建节点，必须使用`device_create()`，它具有以下原型：

```
struct device * device_create(struct class *class,
                              struct device *parent,
                              dev_t devt,
                              void *drvdata, 
                              const char *fmt, ...)
```

该方法创建一个设备并将其注册到 Sysfs 中。在其参数中，`class`是指向`struct class`的指针，表示该设备应注册到的类，`parent`是指向该新设备父设备`struct device`的指针（如果有的话），`devt`是要添加的字符设备的设备号，`drvdata`是要添加到设备的回调数据。

重要提示

如果有多个次设备，`device_create()`和`device_destroy()` API 可以放入一个`for`循环中，并且`<device name format>`字符串可以与循环计数器一起拼接，如下所示：

`device_create(class, NULL, MKDEV(MAJOR(first_devt), MINOR(first_devt) + i), NULL, "mynull%d", i);`

因为设备在创建之前需要一个现有的类，你必须创建一个类或者使用一个已存在的类。现在，我们将创建一个类，为此，我们需要使用`class_create()`函数，声明如下：

```
struct class * class_create(struct module * owner,
                            const char * name);
```

之后，该类将在`/sys/class`中可见，我们可以使用该类创建设备。以下是一个粗略的示例：

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

在前面的代码中，`device_create()`将为每个设备创建一个节点——`/dev/eep-mem0`、`/dev/eep-mem1`，依此类推，我们的类由`eep_class`表示。此外，设备还可以在`/sys/class/eep-class`下查看。与此同时，反向操作如下所示：

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

在前面的代码中，`device_destroy()`将从`/dev`中删除设备节点，`cdev_del()`将使系统忘记这个字符设备，`class_unregister()`和`class_destroy()`将注销并从系统中移除类，最后，`unregister_chrdev_region()`将释放我们的设备号。

现在我们已经熟悉了关于字符设备的所有前提知识，接下来可以开始实现一个文件操作，让用户能够与底层设备交互。

# 实现文件操作

在上一节介绍了文件操作之后，是时候实现这些操作来增强驱动程序功能，并通过系统调用将设备的方法暴露给用户空间。每个方法都有其特定的特点，我们将在本节中重点介绍。

## 在内核空间和用户空间之间交换数据

正如我们在介绍文件操作表时所看到的，`read`和`write`方法用于与底层设备交换数据。它们都是系统调用，这意味着数据将从用户空间传输到内核空间，或从内核空间传输到用户空间。在查看`read`和`write`方法原型时，第一个引起我们注意的是`__user`的使用。它是**Sparse**（一个用于帮助内核检查潜在编码错误的语义检查工具）用来提醒开发者他们即将不正确地使用一个不可信的指针（或者在当前虚拟地址映射中可能无效的指针），开发者不应解引用该指针，而应该使用专门的内核函数来访问指针指向的内存。

这引出了两个主要函数，它们允许我们在内核空间和用户空间之间交换数据：`copy_from_user()`和`copy_to_user()`，分别用于将缓冲区从用户空间复制到内核空间，以及从内核空间复制到用户空间：

```
unsigned long copy_from_user(void *to,
               const void __user *from, unsigned long n)
unsigned long copy_to_user(void __user *to,
               const void *from, unsigned long n)
```

在这两种情况下，前缀为`__user`的指针指向用户空间（不可信）内存。`n`表示要复制的字节数，既可以是从用户空间到内核空间，也可以是从内核空间到用户空间。`from`表示源地址，`to`是目标地址。每个操作都会返回无法复制的字节数（如果有的话），如果成功则返回`0`。需要注意的是，这些例程可能会休眠，因为它们在用户上下文中运行，不需要在原子上下文中调用。

## 实现打开文件操作

`open`文件操作是`open`系统调用的后端。通常使用此方法进行设备和数据结构初始化，初始化后如果成功应返回`0`，如果发生错误则返回一个负的错误码。`open`文件操作的原型定义如下：

```
int (*open) (struct inode *inode, struct file *filp);
```

如果没有实现，设备打开将始终成功，但驱动程序不会意识到这一点。如果设备不需要特殊的初始化，这通常不会成为问题。

### 每个设备的数据

正如我们在文件操作原型中看到的，几乎总是有一个`struct file`参数。`struct file`有一个空闲使用的元素，即`private_data`。如果设置了`file->private_data`，它将在对相同文件描述符调用的其他系统调用中可用。你可以在文件描述符的生命周期内使用这个字段。在`open`方法中设置这个字段是一个好习惯，因为它总是任何文件上的第一个系统调用。

以下是我们的数据结构：

```
struct pcf2127 {
    struct cdev cdev;
    unsigned char *sram_data;
    struct i2c_client *client;
    int sram_size;
    [...]
};
```

基于此数据结构，`open` 方法将如下所示：

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

大多数时候，`open` 操作进行一些初始化并请求在用户保持设备节点的打开实例期间使用的资源。在设备关闭时，必须撤销和释放此操作中所做的所有工作，正如我们将在下一操作中看到的那样。

## 实现释放文件操作

`release` 方法在设备关闭时调用，是 `open` 方法的反向操作。你必须撤销在 `open` 操作中所做的一切。这可能包括释放分配的私有内存、关闭设备（如果支持），并在最后一次关闭时丢弃所有缓冲区（如果设备支持多次打开，或者驱动程序可以处理多个设备）。

以下是 `release` 函数的摘录：

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

上述代码释放了设备节点打开时获得的所有资源。这实际上是此文件操作中需要做的所有事情。如果设备节点由硬件设备支持，此操作还可以将设备置于适当的状态。

此时，我们能够实现字符设备的入口点（`open`）和出口点（`release`）。现在剩下的就是实现每个可能的操作。

## 实现写文件操作

`write` 方法用于向设备发送数据；每当用户在设备的文件上调用 `write()` 系统调用时，内核实现的相应方法会被调用。其原型如下：

```
ssize_t(*write)(struct file *filp, const char __user *buf,
                size_t count, loff_t *pos);
```

此文件操作必须返回已写入的字节数（大小），以下是其参数的定义：

+   `*buf` 表示来自用户空间的数据缓冲区。

+   `count` 是请求传输的大小。

+   `*pos` 表示从文件中的哪个位置开始写入数据（如果字符设备文件由内存支持，则表示相应内存区域的位置）。

通常，在执行此文件操作时，首先需要检查来自用户空间的无效请求或错误请求（例如，检查内存支持设备的大小限制以及大小溢出）。以下是一个示例：

```
/* if trying to Write beyond the end of the file,
 * return error. "filesize" here corresponds to the size
 * of the device memory (if any)
 */
if (*pos >= filesize) return –EINVAL;
```

检查完成后，通常需要进行一些调整，尤其是对于 `count`，以确保不超过文件大小。此步骤也不是强制性的：

```
/* filesize corresponds to the size of device memory */
if (*pos + count > filesize) 
    count = filesize - *pos;
```

下一步是找到开始写入的位置。只有在设备由物理内存支持的情况下，这一步才相关，在这种情况下，`write()` 方法应当存储给定的数据：

```
/* convert pos into valid address */
void *from = pos_to_address(*pos); 
```

最后，你可以将数据从用户空间复制到内核内存，然后在支持设备上执行 `write` 操作，并调整 `*pos`，如下所示：

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

以下是 `write` 方法的示例，概述了目前为止描述的步骤：

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

在数据被读取并处理后，可能需要将处理结果写回文件。由于我们从写操作开始，接下来我们可能想到的是`read`操作，如下节所示。

## 实现读取文件操作

`read`方法具有以下原型：

```
ssize_t (*read) (struct file *filp, char __user *buf,
                 size_t count, loff_t *pos);
```

此操作是`read()`系统调用的后端。它的参数如下所示：

+   `*buf`是我们从用户空间接收的缓冲区。

+   `count`是请求的传输大小（用户缓冲区的大小）。

+   `*pos`表示数据应从文件中读取的起始位置。

它必须返回成功读取的数据大小。尽管这个大小可能小于`count`（例如，在到达文件末尾之前，未达到用户请求的`count`）。

实现读取操作看起来与写操作类似，因为需要执行一些健全性检查。首先，您可以防止读取超出文件大小，并返回文件结束响应：

```
if (*pos >= filesize)
    return 0; /* 0 means EOF */
```

然后，您应该确保读取的字节数不能超过文件大小，并可以适当调整`count`：

```
if (*pos + count > filesize)
    count = filesize – (*pos);
```

接下来，您可以找到开始读取的位置，然后将数据复制到用户空间的缓冲区中，并在失败时返回错误，然后根据读取的字节数更新文件的当前位置信息，并返回复制的字节数：

```
/* convert pos into valid address */
void *from = pos_to_address (*pos); 
sent = copy_to_user(buf, from, count);
if (sent)
    return –EFAULT;
*pos += count;
return count;
```

以下是一个驱动程序`read()`文件操作的示例，旨在概述这里可以做的事情：

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

尽管读取和写入数据会移动光标位置，但有一种操作的主要目的是在不接触数据的情况下仅移动光标位置。此操作有助于通过将光标移到所需位置，从任何地方开始读写数据。

## 实现`llseek`文件操作

`llseek`文件操作是`lseek()`系统调用的内核后端，用于在文件中移动光标位置。它的原型如下所示：

```
loff_t(*llseek) (struct file *filp, loff_t offset,
                 int whence);
```

此回调必须返回文件中的新位置。以下是其参数的定义：

+   `loff_t`是一个偏移量，相对于当前文件位置，定义了要更改的位置。

+   `whence`定义了从哪里开始定位。可能的值如下：

    +   `SEEK_SET`：将光标定位到相对于文件开头的位置

    +   `SEEK_CUR`：将光标定位到相对于当前文件位置的位置

    +   `SEEK_END`：将光标定位到相对于文件末尾的位置

在实现此操作时，使用`switch`语句检查每个可能的`whence`情况是一种良好的实践，因为它们是有限的，并相应地调整新位置：

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

在前面的内核后端摘录之后，以下是一个用户程序的示例，该程序将依次读取并定位到文件中。底层驱动程序随后将执行`llseek()`文件操作入口：

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

该代码产生以下输出：

```
jma@jma:~/work/tutos/sources$ cat toto 
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
jma@jma:~/work/tutos/sources$ ./seek 
Lorem ipsum dolor si
nsectetur adipiscing
psum dolor sit amet,
jma@jma:~/work/tutos/sources$
```

在本节中，我们通过示例解释了寻址的概念，展示了相对寻址和绝对寻址是如何工作的。现在我们已经完成了数据操作的部分，可以转到下一个操作，感知字符设备中数据的可读性或可写性。

## `poll`方法

`poll`方法是`poll()`和`select()`系统调用的后端。这些系统调用用于被动地（通过休眠，避免浪费 CPU 周期）感知文件的可读性/可写性。为了支持这些系统调用，驱动程序必须实现`poll`，其原型如下：

```
unsigned int (*poll) (struct file *, struct poll_table_struct *);
```

这个方法实现的核心函数是`poll_wait()`，定义在`<linux/poll.h>`中，这是你必须在驱动程序代码中包含的头文件。它的声明如下：

```
void poll_wait(struct file * filp,
               wait_queue_head_t * wait_address, poll_table *p)
```

`poll_wait()`将与`struct file`结构相关联的设备（作为第一个参数传递）添加到可以唤醒进程的设备列表中（根据在第三个参数`struct poll_table`结构中注册的事件），并将进程置于由第二个参数`struct wait_queue_head_t`结构表示的等待队列中。用户进程可以调用`poll()`、`select()`或`epoll()`系统调用，将一组文件添加到等待列表中，以便了解相关设备的就绪情况。然后，内核会调用与每个设备文件关联的驱动程序的`poll`方法。每个驱动程序的`poll`方法应该调用`poll_wait()`，以便注册需要通知的事件，并将该进程休眠，直到其中一个事件发生，并将该驱动程序注册为能够唤醒该进程的设备。通常的做法是根据`select()`（或`poll()`）系统调用支持的事件类型为每个事件类型使用一个等待队列（一个用于可读性，另一个用于可写性，必要时还可以为异常使用一个）。

如果有数据可读，则（`*poll`）文件操作的返回值必须设置`POLLIN | POLLRDNORM`；如果设备是可写的，则设置`POLLOUT | POLLWRNORM`；如果没有新数据且设备尚不可写，则返回`0`。在以下示例中，我们假设设备支持阻塞的读写。当然，你也可以只实现其中一个。如果驱动程序没有定义此方法，设备将被视为始终可读和可写，`poll()`或`select()`系统调用将立即返回。

实现`poll`操作可能需要调整读写文件操作方式，在写入时通知读者数据可读，在读取时通知写者数据可写：

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

在前面的代码片段中，我们实现了`poll`操作，当设备不可写或不可读时，进程会被挂起。然而，当这些状态发生变化时，没有通知机制。因此，`write`操作（或任何使数据可用的操作，如 IRQ）必须通知在可读等待队列中挂起的进程；同样，`read`操作（或任何使设备准备好可写的操作）必须通知在可写等待队列中挂起的进程。以下是一个示例：

```
wake_up_interruptible(&my_rq); /* Ready to read */
/* set flag accordingly in case poll is called */
new_data_is_ready = true;
wake_up_interruptible(&my_wq); /* Ready to be written to */
ready_to_be_written = true;
```

更精确地说，你可以通过驱动程序的`write()`方法通知可读事件，这意味着写入的数据可以被读取，或者通过 IRQ 处理程序通知可读事件，这意味着外部设备发送了一些可以读取的数据。另一方面，你可以通过驱动程序的`read()`方法通知可写事件，这意味着缓冲区为空并且可以重新填充，或者通过 IRQ 处理程序通知可写事件，这意味着设备已完成数据发送操作并准备再次接受数据。不要忘记在状态改变时将标志重置为`false`。

以下是使用`select()`在给定字符设备上检测数据可用性的代码片段：

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

在前面的代码示例中，我们使用了不带超时的`select()`，这种方式意味着我们将只在“读取”事件发生时得到通知。从那一行开始，进程会被挂起，直到它收到注册的事件通知。

## ioctl 方法

一个典型的 Linux 系统包含大约 350 个`-ENOTTY`错误，这些错误会发生在任何`ioctl()`系统调用中。以下是其原型：

```
long ioctl(struct file *f, unsigned int cmd,
           unsigned long arg);
```

在前面的原型中，`f`是指向文件描述符的指针，表示已打开的设备实例，`cmd`是 ioctl 命令，`arg`是一个用户参数，可以是任何用户内存的地址，驱动程序可以在其上调用`copy_to_user()`或`copy_from_user()`。为了简洁起见，并出于显而易见的原因，IOCTL 命令需要由一个唯一的编号来标识，这个编号应该在系统中唯一。IOCTL 编号的唯一性可以防止我们将正确的命令发送到错误的设备，或者将错误的参数传递给正确的命令（因为 IOCTL 编号重复）。Linux 提供了四个辅助宏来创建 IOCTL 标识符，具体取决于是否有数据传输以及传输的方向。它们各自的原型如下：

```
_IO(MAGIC, SEQ_NO)
_IOR(MAGIC, SEQ_NO, TYPE)
_IOW(MAGIC, SEQ_NO, TYPE)
_IORW(MAGIC, SEQ_NO, TYPE)
```

它们的描述如下：

+   `_IO`：IOCTL 命令不需要数据传输。

+   `_IOR`：这意味着我们正在创建一个 IOCTL 命令编号，用于将信息从内核传递到用户空间（即读取数据）。驱动程序将允许返回`sizeof(TYPE)`字节给用户，而不会将这个返回值视为错误。

+   `_IOW`：这与`_IOR`相同，但这次是用户向驱动程序发送数据。

+   `_IOWR`：IOCTL 命令需要同时提供写入和读取参数。

它们的参数含义（按传递顺序）在此处进行了描述：

+   一个由 8 位编码的数字（`0` 到 `255`），称为**魔术数字**。

+   一个序列号或命令 ID，大小也是 8 位。

+   一个数据类型（如果有的话），它将告诉内核要复制的大小。这可以是一个结构体或数据类型的名称。

这在内核源代码的`Documentation/ioctl/ioctl-decoding.txt`中有很好的文档记录，现有的 IOCTL 命令列在`Documentation/ioctl/ioctl-number.txt`中，这是创建你自己 IOCTL 命令时的一个好起点。

### 生成一个 IOCTL 数字（一个命令）

建议在专用的头文件中生成你自己的 IOCTL 数字，因为这个头文件也应该在用户空间中可用。换句话说，你应该处理 IOCTL 头文件的重复（例如通过符号链接），确保内核中有一个，用户空间中也有一个，这样用户应用程序可以包含它。现在让我们在一个实际示例中生成一些 IOCTL 数字，并称这个头文件为`eep_ioctl.h`：

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

在定义命令之后，头文件需要包含在最终代码中。此外，由于它们都是唯一且有限的，使用`switch ... case`语句处理每个命令，并在调用未定义的`ioctl`命令时返回`-ENOTTY`错误代码是一个好习惯。以下是一个示例：

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

内核和用户空间都必须包含包含 IOCTL 命令的头文件。因此，在前面摘录的第一行中，我们包含了`eep_ioctl.h`，这是定义我们 IOCTL 命令的头文件。

如果你认为你的 IOCTL 命令需要多个参数，你应该将这些参数收集到一个结构体中，并只需将结构体的指针传递给`ioctl`。

现在，从用户空间，你必须使用与驱动代码中相同的`ioctl`头文件：

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

在前面的代码中，我们展示了如何从用户空间使用内核的 IOCTL 命令。也就是说，在本节中，我们学习了如何实现字符设备的`ioctl`回调，以及如何在内核和用户空间之间交换数据。

# 总结

在本章中，我们已经解开了字符设备的神秘面纱，并且我们已经看到如何通过设备文件让用户与我们的驱动程序进行交互。我们学会了如何将文件操作暴露给用户空间，并在内核中控制它们的行为。我们甚至走得更远，现在你甚至能够实现多设备支持。

下一章将更侧重于硬件，因为它涉及设备树，这是一种机制，允许系统中的硬件设备向内核声明。下章见。
