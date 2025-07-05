# *Chapter 9*: Writing SPI Device Drivers

The **Serial Peripheral Interface** (**SPI**) is (at least) a 4-wire bus – **Master Input Slave Output** (**MISO**), **Master Output Slave Input** (**MOSI**), **Serial Clock** (**SCK**), and **Chip Select** (**CS**) – which is used to connect serial flash and analog-to-digital/digital-to-analog converters. The master always generates the clock. Its speed can reach up to 80 MHz, though there is no real speed limitation (this is much faster than I2C as well). The same applies to the CS line, which is always managed by the master.

Each of these signal names has a synonym:

*   Whenever you see **Slave Input Master Output** (**SIMO**), **Slave Data Input** (**SDI**), or **Data Input** (**DI**), they refer to MOSI.
*   **Slave Output Master Input** (**SOMI**), **Slave Data Output** (**SDO**), and **Data Output** (**DO**) refer to MISO.
*   **Serial Clock** (**SCK**), **Clock** (**CLK**), and **Serial Clock** (**SCL**) refer to SCK.
*   S̅ S̅ is the Slave Select line, also called CS. CSx can be used (where *x* is an index such as CS0, CS1), EN and ENB too, meaning enable. CS is usually an active low signal.

The following diagram shows how SPI devices are connected to the controller via the bus it exposes:

![Figure 9.1 – SPI slave devices and master interconnection
](img/B17934_09_001.jpg)

Figure 9.1 – SPI slave devices and master interconnection

From the preceding diagram, we can represent the Linux kernel SPI framework as follows:

```
CPU <--platform bus--> SPI master <---SPI bus---> SPI slave
```

The CPU is the master hosting the SPI controller, also known as the SPI master, which manages the bus segment hosting the SPI slave devices. In the kernel SPI framework, the bus is managed by a platform driver while the slave is driven by an SPI device driver. However, both drivers use APIs provided by the SPI core. In this chapter, we will be focusing on SPI (slave) device drivers, though references to the controller will be mentioned if necessary.

This chapter will walk through SPI driver concepts such as the following:

*   Understanding the SPI framework abstraction in the Linux kernel
*   Dealing with the SPI driver abstraction and architecture
*   Learning how not to write SPI device drivers

# Understanding the SPI framework abstractions in the Linux kernel

The Linux kernel SPI framework is made up of a few data structures, the most important of which are the following:

*   `spi_controller`, used to abstract the SPI master device.
*   `spi_device`, used to abstract a slave device sitting on the SPI bus.
*   `spi_driver`, the driver of the slave device.
*   `spi_transfer`, which is the low-level representation of one segment of a protocol. It represents a single operation between the master and slave. It expects Tx and/or Rx buffers as well as the length of the data to be exchanged and an optional CS behavior.
*   `spi_message`, which is an atomic sequence of transfers.

Let's now introduce each of these data structures, one after the other, starting with the most complex, which represents the SPI controller's data structure.

## Brief introduction to struct spi_controller

Throughout this chapter, we will reference the controller because it is deeply coupled with the slaves and other data structures that the SPI framework is made up of. It is necessary therefore to introduce its data structure, represented by `struct spi_controller` and defined as follows:

```
struct spi_controller {
    struct device     dev;
    u16               num_chipselect;
    u32               min_speed_hz;
    u32               max_speed_hz;
    int               (*setup)(struct spi_device *spi);
    int (*set_cs_timing)(struct spi_device *spi,
                          struct spi_delay *setup,
                          struct spi_delay *hold,
                          struct spi_delay *inactive);
    int    (*transfer)(struct spi_device *spi,
                         struct spi_message *mesg);
    bool    (*can_dma)(struct spi_controller *ctlr,
                        struct spi_device *spi,
                        struct spi_transfer *xfer);
    struct kthread_worker  *kworker;
    struct kthread_work    pump_messages;
    spinlock_t             queue_lock;
    struct list_head       queue;
    struct spi_message     *cur_msg;
    bool                   busy;
    bool                   running;
    bool                   rt;
    int (*transfer_one_message)(
                      struct spi_controller *ctlr,
                      struct spi_message *mesg);
[...]
    int (*transfer_one_message)(
            struct spi_controller *ctlr,
            struct spi_message *mesg);
    void (*set_cs)(struct spi_device *spi, bool enable);
    int (*transfer_one)(struct spi_controller *ctlr,
                    struct spi_device *spi,
                    struct spi_transfer *transfer);
[...]
    /* DMA channels for use with core dmaengine helpers */
    struct dma_chan    *dma_tx;
    struct dma_chan    *dma_rx;
    /* dummy data for full duplex devices */
    Void              *dummy_rx;
    Void              *dummy_tx;
};
```

Only the important elements required for a better understanding of the data structure used in this chapter are listed in the preceding code. The following list explains their use:

*   `num_chipselect` indicates the number of CSs assigned to this controller. CSs are used to distinguish individual SPI slaves and are numbered from 0.
*   `min_speed_hz` and `max_speed_hz` are the lowest and the highest transfer speeds supported by this controller, respectively.
*   `set_cs_timing` is a method provided if the SPI controller supports CS timing configuration, in which case the client drivers would call `spi_set_cs_timing()` with the requested timings. It has been deprecated in recent kernel versions by this patch: [https://lore.kernel.org/lkml/20210609071918.2852069-1-gregkh@linuxfoundation.org/](https://lore.kernel.org/lkml/20210609071918.2852069-1-gregkh@linuxfoundation.org/)).
*   `transfer` adds a message to the transfer queue of the controller. On the controller registration path (thanks to `spi_register_controller()`), the SPI core checks whether this field is `NULL` or not:
    *   If `NULL`, the SPI core will check if either `transfer_one` or `transfer_one_message` is set, in which case it assumes this controller supports message queuing and invokes `spi_controller_initialize_queue()`, which will set this field with `spi_queued_transfer` (which is the SPI core helper to queue SPI messages to the controller's queue and to schedule the message pump `kworker` if it is not already running or busy).
        *   Moreover, `spi_controller_initialize_queue()` will create both a dedicated kthread worker (`kworker` element) and a work struct (`pump_messages` element) for this controller. This worker will be scheduled quite often in order to process the message queue in a FIFO order.
        *   Next, the controller's `queued` element is set to true by the SPI core.
        *   Finally, if the controller's `rt` element has been set to true by the driver prior to calling the registration API, the SPI core will set the scheduling policy of the worker thread to the real-time FIFO policy, with a priority of 50.
    *   If `NULL`, and both `transfer_one` and `transfer_one_message` are also `NULL`, this is an error, and the controller is not registered.
    *   If not `NULL`, the SPI core assumes the controller does not support queuing and does not call `spi_controller_initialize_queue()`.
*   `transfer_one` and `transfer_one_message` are mutually exclusive. If both are set, the former won't be invoked by the SPI core. `transfer_one` transfers a single SPI transfer and has no notion of `spi_message`. `transfer_one_message`, if provided by the driver, must work on the basis of `spi_message` and will be responsible for all the transfers in the messages. Controller drivers that need not bother with message-handling algorithms just have to set the `transfer_one` callback, in which case the SPI core will set `transfer_one_message` to `spi_transfer_one_message`. `spi_transfer_one_message` will take care of all the message logic, timings, CS, and other hardware-related properties prior to calling the driver provided `transfer_one` callback for each transfer in the message. CS remains active throughout the message transfers unless it is modified by a transfer that has `spi_transfer.cs_change = 1`. The message transfers will be performed using the clock and SPI mode parameters previously applied by `setup()` for this device.
*   `kworker`: This is the kernel thread dedicated to the message pump processing.
*   `pump_messages`: This is an abstraction of a work struct data structure for scheduling the function that processes the SPI message queue. It is scheduled in `kworker`. This work struct is backed by the `spi_pump_messages()` method, which checks if there are any SPI messages in the queue that need to be processed, and if so, it calls the driver to initialize the hardware and transfer each message.
*   `queue_lock`: The spinlock to synchronize access to the message queue.
*   `queue`: The message queue for this controller.
*   `idling`: This indicates whether the controller device is entering an idle state.
*   `cur_msg`: The currently in-flight SPI message.
*   `busy`: This indicates the busyness of the message pump.
*   `running`: This indicates that the message pump is running.
*   `Rt`: This indicates whether `kworker` will run the message pump with real-time priority.
*   `dma_tx`: The DMA transmit channel (when supported by the controller).
*   `dma_rx`: The DMA receiving channel (when supported by the controller).

SPI transfers always read and write the same number of bytes, which means even when the client driver issues a half-duplex transfer, full duplex is emulated by the SPI core with `dummy_rx` and `dummy_tx` used to achieve this purpose:

*   `dummy_rx`: This is a dummy receive buffer used for full-duplex devices, such that if a transfer's receive buffer is `NULL`, received data will be shifted to this dummy receive buffer before being discarded.
*   `dummy_tx`: This is a dummy transmit buffer used for full-duplex devices, such that if a transfer's transmit buffer is `NULL`, this dummy transmit buffer will be zero-filled and used as a transmit buffer for the transfer.

Do note that the SPI core names the SPI message pump worker task with the controller device name (dev->name), set in `spi_register_controller()` as follows:

```
dev_set_name(&ctlr->dev, "spi%u", ctlr->bus_num);
```

Later, when the worker is created during the queue initialization (remember, `spi_controller_initialize_queue()`), it is given this name, as follows:

```
ctlr->kworker = kthread_create_worker(0, dev_name(&ctlr->dev));
```

To recognize the SPI message pump worker on your system, you can run the following command:

```
root@yocto-imx6:~# ps | grep spi
65 root         0 SW   [spi1]
```

In the preceding snippet, we can see the worker's name made up of the bus name along with the bus number.

In this section, we analyzed the concepts on the controller side to help get an understanding of the whole SPI slave implementation in the Linux kernel. The importance of this data structure is so great that I recommend you read this section whenever you feel you don't understand any mechanism in the coming sections. Now we can switch to SPI device data structures for real.

## The struct spi_device structure

The first and most obvious data structure, `struct spi_device` represents an SPI device and is defined in `include/linux/spi/spi.h`:

```
struct spi_device {
    struct device dev;
    struct spi_controller  *controller;
    struct spi_master *master;
    u32         max_speed_hz;
    u8          chip_select;
    u8          bits_per_word;
    bool        rt;
    u16         mode;
    int          irq;
    [...]
    int cs_gpio; /* LEGACY: chip select gpio */
    struct gpio_desc *cs_gpiod; /* chip select gpio desc */
    struct spi_delay word_delay; /* inter-word delay */
    /* the statistics */
    struct spi_statistics statistics;
};
```

For the sake of readability, the number of fields listed is reduced to the strict minimum needed for the purpose of the book. The following list details the meaning of each element in this structure:

*   `controller` represents the SPI controller this slave device belongs to. In other words, it represents the SPI controller (bus) on which the device is connected.
*   The `master` element is still there for compatibility reasons and will be deprecated soon. It was the old name of the controller.
*   `max_speed_hz` is the maximum clock rate to be used with this slave; this parameter can be changed from within the driver. We can override that parameter using `spi_transfer.speed_hz` for each transfer. We will discuss SPI transfer later.
*   `chip_select` is the CS line assigned to this device. It is active low by default. This behavior can be changed in `mode` by adding the `SPI_CS_HIGH` flag.
*   `rt`, if `true`, will make the message pump worker of the `controller` run as a real-time task
*   `mode` defines how data should be clocked. The device driver may change this. The data clocking is MSB by default for each word in a transfer. This behavior can be overridden by specifying `SPI_LSB_FIRST`.
*   `irq` represents the interrupt number (registered as a device resource in your board initialization file or through the device tree) you should pass to `request_irq()` to receive interrupts from this device.
*   `cs_gpio` and `cs_gpiod` are both optional. The former is the legacy integer-based GPIO number of the CS line, while the latter is the new and recommended interface, based on the GPIO descriptor.

A word about SPI modes – they are built using two characteristics:

*   CPOL, which is the initial clock polarity:
    *   `0`: The initial clock state is low, and the first edge is rising.
    *   `1`: The initial clock state is high, and the first state is falling.
*   CPHA is the clock phase, determining at which edge the data will be sampled:
    *   `0`: Data is latched at the falling edge (high to low transition), whereas the output changes at the rising edge.
    *   `1`: Data is latched at rising edge (low to high transition), and the output changes at the falling edge.

This allows us to distinguish four SPI modes, which are derived macros made up of a mix of two main macros, defined in `include/linux/spi/spi.h` as follows:

```
#define    SPI_CPHA    0x01
#define    SPI_CPOL    0x02
```

The combinations of these macros give the following SPI modes:

![Table 9.1 – SPI modes kernel definition
](img/Table_9.1.jpg)

Table 9.1 – SPI modes kernel definition

The following diagram is the representation of each SPI mode, in the same order as defined in the preceding array. That being said, only the MOSI line is represented, but the principle is the same for MISO.

![Figure 9.2 – SPI operating modes
](img/B17934_09_002.jpg)

Figure 9.2 – SPI operating modes

Now that we are familiar with the SPI device data structure and the modes such a device can operate in, we can switch to the second-most important structure, the one representing the SPI device driver.

## The spi_driver structure

Also called the protocol driver, an SPI device driver is responsible for driving devices sitting on the SPI bus. It is abstracted in the kernel by `struct spi_driver`, declared as follows:

```
struct spi_driver {
   const struct spi_device_id *id_table;
   int         (*probe)(struct spi_device *spi);
   int         (*remove)(struct spi_device *spi);
   void        (*shutdown)(struct spi_device *spi);
   struct device_driver    driver;
};
```

The following list outlines the meanings of the elements in this data structure:

*   `id_table`: This is the list of SPI devices supported by this driver.
*   `probe`: This method binds this driver to the SPI device. This function will be invoked on any device claiming this driver and will decide whether this driver is in charge of that device or not. If yes, the binding process occurs.
*   `remove`: Unbinds this driver from the SPI device.
*   `shutdown`: This is invoked during system state changes such as powering down and halting.
*   `driver`: This is the low-level driver structure for the device and driver model.

This is all we can say for now on this data structure, except that each SPI device driver must fill and expose one instance of this type.

## The message transfer data structures

The SPI I/O model consists of a set of queued messages, each of which can be made up of one or more SPI transfers. While a single message consists of one or more `struct spi_transfer` objects, each transfer represents a full duplex SPI transaction. Messages are submitted and processed either synchronously or asynchronously. The following is a diagram explaining the concept of message and transfer:

![Figure 9.3 – Example SPI message structure
](img/B17934_09_003.jpg)

Figure 9.3 – Example SPI message structure

Now that we are familiar with the theoretical aspects, we can introduce the SPI transfer data structure, declared as follows:

```
struct spi_transfer {
    const void    *tx_buf;
    void          *rx_buf;
    unsigned    len;
    dma_addr_t    tx_dma;
    dma_addr_t    rx_dma;
    struct sg_table tx_sg;
    struct sg_table rx_sg;
    unsigned    cs_change:1;
    unsigned    tx_nbits:3;
    unsigned    rx_nbits:3;
#define    SPI_NBITS_SINGLE 0x01 /* 1bit transfer */
#define    SPI_NBITS_DUAL        0x02 /* 2bits transfer */
#define    SPI_NBITS_QUAD        0x04 /* 4bits transfer */
    u8        bits_per_word;
    u16       delay_usecs;
    struct    spi_delay    delay;
    struct spi_delay  cs_change_delay;
    struct spi_delay  word_delay;
    u32        speed_hz;
    u32        effective_speed_hz;
[...]
    struct list_head transfer_list;
#define SPI_TRANS_FAIL_NO_START  BIT(0)
    u16        error;
};
```

The following are the meanings of each element in the data structure:

*   `tx_buf` is a pointer to the buffer that contains the data to be written. If set to `NULL`, this transfer will be considered as half duplex as a read-only transaction. It should be DMA-safe when you need to perform an SPI transaction through DMA.
*   `rx_buf` is a buffer for data to be read (with the same properties as `tx_buf`), or `NULL` in a write-only transaction.
*   `tx_dma` is the `tx_buf`, in case `spi_message.is_dma_mapped` is set to `1`.
*   `rx_dma` is the same as `tx_dma`, but for `rx_buf`.
*   `len` represents the size of the `rx` and `tx` buffers in bytes. Only `len` bytes shift out (or in) and attempting to shift out a partial word would result in an error.
*   `speed_hz` supersedes the default speed specified in `spi_device.max_speed_hz`, but only for the current transfer. If `0`, the default (from `spi_device`) is used.
*   `bits_per_word`: A data transfer involves one or more words. A word is a unit of data whose size in bits varies according to the needs. Here, `bits_per_word` represents the size in bits of a word for this SPI transfer. This overrides the default value provided in `spi_device.bits_per_word`. If `0`, the default (from `spi_device`) is used.
*   `cs_change` determines whether the CS becomes inactive after this transfer completes. All SPI transfers begin with the appropriate CS signal active. Normally, it remains selected until the last transfer in the message is completed. Using `cs_change`, drivers can change the CS signal.

This flag is used to make the CS temporarily inactive in the middle of the message (that is, before processing the `spi_transfer` on which it is specified) if the transfer isn't the last one in the message. Toggling CS in this way may be required to complete a chip command, allowing a single SPI message to handle the entire set of chip transactions.

*   `delay_usecs` represents the delay (in microseconds) following this transfer before (optionally) changing the `chip_select` status, then starting the next transfer or completing this `spi_message`.

    Note

    SPI transfers always write the same number of bytes as they read, even in half-duplex transactions. The SPI core achieves this thanks to the controller's `dummy_rx` and `dummy_tx` elements. When the transmit buffer is null, `spi_transfer->tx_buf` will be set with the controller's `dummy_tx`. Then, zeroes will be shifted out while filling `rx_buf` with the data coming from the slave. If the receive buffer is null, `spi_transfer->rx_buf` will be set with the controller's `dummy_rx` and the data shifted in will be discarded.

### struct spi_message

`spi_message` is used to atomically issue a sequence of transfers, each represented by a `struct spi_transfer` instance. We say *atomically* because no other `spi_message` may use that SPI bus until the ongoing sequence completes. Do however note that there are platforms that can handle many such sequences with a single programmed DMA transfer. An SPI message structure has the following declaration:

```
struct spi_message {
       struct list_head     transfers;
       struct spi_device    *spi;
       unsigned       is_dma_mapped:1;
       /* completion is reported through a callback */
       void                 (*complete)(void *context);
       void                 *context;
       unsigned       frame_length;
       unsigned       actual_length;
       int                  status;
    };
```

The following list outlines the meanings of elements in this data structure:

*   `transfers` is the list of transfers that constitute the message. We will see later how to add a transfer to this list. Using the `spi_transfer.cs_change` flag on the last transfer in that atomic group may potentially save costs for chip deselect and select operations.
*   `is_dma_mapped` informs the controller whether to use DMA (or not) to perform the transaction. Your code is then responsible for providing DMA and CPU virtual addresses for each transfer buffer.
*   `complete` is a callback called when the transaction is done, and `context` is the parameter to be given to the callback.
*   `frame_length` will be set automatically with the total number of bytes in the message.
*   `actual_length` is the number of bytes transferred in all successful segments.
*   `status` reports the transfer's status. This is `0` on success; otherwise, it's `-errno`.

`spi_transfer` elements in a message are processed in FIFO order. Until the message is completed (that is, until the completion callback is executed), you must make sure not to use transfer buffers in order to avoid data corruption. The code that submits a `spi_message` (and its `spi_transfers`) to the lower layers is responsible for managing its memory. Drivers must ignore the message (and its transfers) once submitted at least until its completion callback is invoked.

## Accessing the SPI device

An SPI controller is able to communicate with one or more slaves, that is, one or more `struct spi_device`. They form a tiny bus that shares MOSI, MISO, and SCK signals but not CS. Because those shared signals are ignored unless the chip is selected, each device can be programmed to utilize a different clock rate. The SPI controller driver manages communication with those devices through a queue of `spi_message` transactions, moving data between CPU memory and an SPI slave device. For each message instance it queues, it calls the message's completion callback when the transaction completes.

Before a message can be submitted to the bus, it has to be initialized with `spi_message_init()`, which has the following prototype:

```
void spi_message_init(struct spi_message *message)
```

This function will zero each element in the structure and initialize the transfers list. For each transfer to be added to the message, you should call `spi_message_add_tail()` on that transfer, which will result in enqueuing the transfer into the message's transfers list. It has the following declaration:

```
spi_message_add_tail(struct spi_transfer *t, struct spi_message *m)
```

Once this is done, you have two choices to start the transaction:

*   `int spi_sync(struct spi_device *spi, struct spi_message *message)`, which returns `0` on success, else a negative error code. This function may sleep and is not to be used in interrupt contexts. Do however note that this function may sleep in a non-interruptible manner, and does not allow specifying a timeout. A DMA-capable controller's driver may leverage this DMA feature to push/pull data directly into/from the message buffers.

The SPI device's CS is activated by the core during an entire message (from the first transfer to the last), and is then normally disabled between messages. There are drivers which, in order to minimize the impacts of selecting a chip (to save power for example), leave it selected, anticipating that the next message will go to the same chip.

*   `spi_async()` function, which can be used in an any context (atomic or not), and whose prototype is `int spi_async(struct spi_device *spi, struct spi_message *message)`. This function is context agnostic since only submission is done and the processing is asynchronous. However, the completion callback is invoked in a context that can't sleep. Before this callback is invoked, the value of `message->status` is undefined. At the time it is invoked, `message->status` holds the completion status, which is either `0` (to indicate complete success) or a negative error code.

After that callback returns, the driver that initiated the transfer request may deallocate the associated memory since it's no longer in use by any SPI core or controller driver code. Until the completion callback of the currently processed message returns, no subsequent `spi_message` queued to that device will be processed. This rule applies to synchronous transfer calls as well, since they are wrappers around this core asynchronous primitive. This function returns `0` on success, else a negative error code.

The following is an excerpt from a driver demonstrating SPI message and transfer initialization and submission:

```
Static int regmap_spi_gather_write(
                    void *context, const void *reg,
                    size_t reg_len, const void *val,
                    size_t val_len)
{
    struct device *dev = context;
    struct spi_device *spi = to_spi_device(dev);
    struct spi_message m;
    u32 addr;
    struct spi_transfer t[2] = {
      { .tx_buf = &addr, .len = reg_len, .cs_change = 0,},
      { .tx_buf = val, .len = val_len, },
    };
    addr = TCAN4X5X_WRITE_CMD  |
             (*((u16 *)reg) << 8) | val_len >> 2;
    spi_message_init(&m);
    spi_message_add_tail(&t[0], &m);
    spi_message_add_tail(&t[1], &m);
    return spi_sync(spi, &m);
}
```

The preceding excerpt however shows static initialization, on the fly, where both messages and transfers are discarded on the return path of the function. There may be cases where the driver would like to pre-allocate messages along with their transfers for the lifetime of the driver in order to avoid a frequent initialization overhead. In such cases, dynamic allocation can be used thanks to `spi_message_alloc()`, and freed using `spi_message_free()`. They have the following prototypes:

```
struct spi_message *spi_message_alloc(unsigned ntrans,
                                      gfp_t flags)
void spi_message_free(struct spi_message *m)
```

In the preceding snippet, `ntrans` is the number of transfers to allocate for this new `spi_message`, and `flags` represents the flags for the freshly allocated memory, where using `GFP_KERNEL` is enough. On success, this function returns the new allocated message structure along with its transfers. You can access transfer elements using kernel list-related macros such as `list_first_entry`, `list_next_entry`, or even `list_for_each_entry`. The following is an example showing the usage of these macros:

```
/* Completion handler for async SPI transfers */
static void my_complete(void *context)
{
    struct spi_message *msg = context;
    /* doing some other stuffs */
    […]
    spi_message_free(m);
}
static int example_spi_async(struct spi_device *spi,
              struct my_fake_spi_reg *cmds, unsigned len)
{
    struct spi_transfer *xfer;
    struct spi_message *msg;
    msg = spi_message_alloc(len, GFP_KERNEL);
    if (!msg)
         return -ENOMEM;
    msg->complete = my_complete;
    msg->context = msg;
    list_for_each_entry(xfer, &msg->transfers,
               transfer_list) {
        xfer->tx_buf = (u8 *)cmds;
        /* feel free to handle .rx_buf, and so on */
        [...]
        xfer->len = 2;
        xfer->cs_change = true;
         cmds++;
    }
    return spi_async(spi, msg);
}
```

In the preceding excerpt, we have not only shown how to use dynamic message and transfer allocation. We have also seen how `spi_async()` is used. This example is quite useless since the allocated message and transfers are immediately freed upon completion. A best practice with dynamic allocation is to allocate Tx and Rx buffers dynamically as well, and keep them within arm's reach for the lifetime of the driver.

Note however that the device driver is responsible for organizing the messages and transfer in the most appropriate way for the device, as follows:

*   When bidirectional reads and writes start and how its sequence of `spi_transfer` requests are arranged
*   I/O buffer preparation, knowing that each `spi_transfer` wraps a buffer for each transfer direction, supporting full duplex transfers (even if one pointer is `NULL`, in which case the controller will use one of its dummy buffers)
*   Optionally using `spi_transfer.delay_usecs` to define short delays after transfers
*   Whether CS should change (becoming inactive) after a transfer and any delay by using the `spi_transfer.cs_change` flag

With `spi_async`, the device driver queues the messages, registers a completion callback, wakes the message pump, and immediately returns. The completion callback will be invoked when the transfers are complete. Because neither message queuing nor message pump scheduling can block, the `spi_async` function is considered context agnostic. However, it requires that you wait for the completion callback before you can access the buffers in the `spi_transfer` pointers you submitted. On the other hand, `spi_sync` queues the messages and blocks until they are complete. It does not require completion callback. When `spi_sync` returns, it is safe to access your data buffers. If you look at its implementation in `drivers/spi/spi.c`, you'll see it uses `spi_async` to put the calling thread to sleep until the completion callback is called. Since the 4.0 kernel there has been an improvement for `spi_sync` where, when there is nothing in the queue, the message pump will get executed in the context of the caller instead of the message pump thread, which avoids the cost of a context switch.

After the most important data structures and APIs of the SPI framework have been introduced, we can discuss the real driver implementation.

# Dealing with the SPI driver abstraction and architecture

This is where the driver logic takes place. It consists of filling `struct spi_driver` with a set of driving functions that allow probing and controlling the underlying device.

## Probing the device

The SPI device is probed by the `spi_driver.probe` callback. The probe callback is responsible for making sure the driver recognizes the given device before they can be bound together. This callback has the following prototype:

```
int probe(struct spi_device *spi)
```

This method must return `0` on success, or a negative error number otherwise. The only argument is the SPI device to be probed, whose structure has been pre-initialized by the core according to its description in the device tree.

However, most (if not all) of the properties of the SPI device can be overridden, as we have seen while describing its data structure. SPI protocol drivers may need to update the transfer mode if the device doesn't work with its default. They may likewise need to update clock rates or word sizes from their initial values. This is possible thanks to the `spi_setup()` helper, which has the following prototype:

```
int spi_setup(struct spi_device * spi)
```

This function must be called from a context that can sleep exclusively. It expects as a parameter an SPI device structure whose properties to override must have been set in their respective fields. Changes will be effective at the next device access (either for a read or write operation after it has been selected) except for `SPI_CS_HIGH`, which takes effect immediately. The SPI device is deselected on the return path of this function. This function returns `0` on success or a negative error on failure. It is worth paying attention to its return value because this call won't succeed if the driver provides an option that is not supported by the underlying controller or its driver. For instance, some hardware handles wire transfers using nine-bit words, **least significant bit** (**LSB**)-first wire encoding, or active-high CS, and others do not.

You likely want to call `spi_setup()`from `probe()` before submitting any I/O request to the device. However, it can be called anywhere in the code provided no message is pending for that device.

The following is a probing example that sets up the SPI device, checks its family ID, and returns `0` (device recognized) on success:

```
#define FAMILY_ID 0x57
static int fake_probe(struct spi_device *spi)
{
    int err;
    u8 id;
    spi->max_speed_hz =
               min(spi->max_speed_hz, DEFAULT_FREQ);
    spi->bits_per_word = 8;
    spi->mode = SPI_MODE_0;
    spi->rt = true;
    err = spi_setup(spi);
    if (err)
        return err;
    /* read family id */
    err = get_chip_version(spi, &id);
    if (err)
         return -EIO;
    /* verify family id */
    if (id != FAMILY_ID) {
        dev_err(&spi->dev"
    "chip family: expected 0x%02x but 0x%02x rea"\n",
               FAMILY_ID, id);
          return -ENODEV;
    }
    /* register with other frameworks */
    [...]
    return 0;
}
```

A real probing method would also probably deal with some driver state data structures or other per-device data structures. Regarding the `get_chip_version()` function, it may have the following body:

```
#define REG_FAMILY_ID 0x2445
#define DEFAULT_FREQ 10000000
static int get_chip_version(spi_device *spi, u8 *id)
{
    struct spi_transfer t[2];
    struct spi_message m;
    u16 cmd;
    int err;
    cmd = REG_FAMILY_ID;
    spi_message_init(&m);
    memset(&t, 0, sizeof(t));
    t[0].tx_buf = &cmd;
    t[0].len = sizeof(cmd);
    spi_message_add_tail(&t[0], &m);
    t[1].rx_buf = id;
    t[1].len = 1;
    spi_message_add_tail(&t[1], &m);
    return spi_sync(spi, &m);
}
```

Now that we have seen how to probe an SPI device, it will be useful to discuss how to tell the SPI core which devices the driver can support.

Note

The SPI core allows setting/getting driver state data using `spi_get_drvdata()` and `spi_set_drvdata()` in the same way as we did while discussing I2C device drivers in [*Chapter 8*](B17934_08_Epub.xhtml#_idTextAnchor117)*, Writing I2C Device Drivers*.

## Provisioning devices in the driver

As we need a list of i`2c_device_id` to tell I2C core what devices an I2C driver can support, we must provide an array of `spi_device_id` to inform the SPI core what devices our SPI driver supports. After that array has been filled, it must be assigned to the `spi_driver.id_table` field. Additionally, for device matching and module loading purposes, this same array needs to be given to the `MODULE_DEVICE_TABLE` macro. `struct spi_device_id` has the following declaration in `include/linux/mod_devicetable.h`:

```
struct spi_device_id {
   char name[SPI_NAME_SIZE];
   kernel_ulong_t driver_data;
};
```

In the preceding data structure, `name` is a descriptive name for the device, and `driver_data` is the driver state value. It can be set with a pointer to a per-device data structure. The following is an example:

```
#define ID_FOR_FOO_DEVICE   0
#define ID_FOR_BAR_DEVICE   1 
static struct spi_device_id foo_idtable[] = {
  "{ ""oo", ID_FOR_FOO_DEVICE },
  "{ ""ar", ID_FOR_BAR_DEVICE },
   { },
};
MODULE_DEVICE_TABLE(spi, foo_idtable);
```

To be able to match the devices declared in the device tree, we need to define an array of `struct of_device_id` elements and both assign it to `spi_driver.of_match_table` and call the `MODULE_DEVICE_TABLE` macro on it. The following is an example, which also shows what the final `spi_driver` structure would look like when it is set up:

```
static const struct of_device_id foobar_of_match[] = {
        { .compatible"= "packtpub,foobar-dev"ce" },
        { .compatible"= "packtpub,barfoo-dev"ce" },
        {},
};
MODULE_DEVICE_TABLE(of, foobar_of_match);
```

The following excerpt shows the final `spi_driver` content:

```
static struct spi_driver foo_driver = {
    .driver         = {
        .name  "= ""oo",
        /* The below line adds Device Tree support */
        .of_match_table = of_match_ptr(foobar_of_match),
    },
    .probe          = my_spi_probe,
    .id_table       = foo_idtable,
};
```

In the preceding, we can see what an SPI driver structure looks like after it has been set up. There is however a missing element, the `spi_driver.remove` callback, which is used to undo what was done in the probing function.

## Implementing the spi_driver.remove method

The `remove` callback must be used to release every resource grabbed and undo what was done at probing. This callback has the following prototype:

```
static int remove(struct spi_device *spi)
```

In the preceding snippet, `spi` is the SPI device data structure, the same given to the `probe` callback, which simplifies device state data structure tracking between the probing and the removal of the device. This method returns `0` on success or a negative error code on failure. You must make sure that the device is left in a coherent and stable state as well. The following is an example implementation:

```
static int mc33880_remove(struct spi_device *spi)
{
    struct mc33880 *mc;
    mc = spi_get_drvdata(spi); /* Get our data back */
    if (!mc)
        return -ENODEV;
    /*
     * unregister from frameworks with which we
     * registered in the probe function
     */
    gpiochip_remove(&mc->chip);
    [...]
    /* releasing any resource */
    mutex_destroy(&mc->lock);
    return 0;
}
```

In the preceding example, the code dealt with unregistering from the frameworks and releasing the resources. This is the classic case that you will face in 90% of cases.

## Driver initialization and registration

At this step of implementation, your code is almost ready, and you would like to inform the SPI core of your SPI driver. This is driver registration. For SPI device drivers, the SPI core provides `spi_register_driver()` and `spi_unregister_driver()` both to register and unregister and SPI device driver with the SPI core. Those methods have the following prototypes:

```
int spi_register_driver(struct spi_driver *sdrv);
void spi_unregister_driver(struct spi_driver *sdrv);
```

In both functions, `sdrv` is the SPI driver structure that has been previously set up. The registration API returns zero on success or a negative error code on failure.

Driver registration and unregistering usually take place in the module initialization and module exit method. The following is a typical demonstration of SPI driver registration:

```
static int __init foo_init(void)
{
   [...] /*My init code */
   return spi_register_driver(&foo_driver);
}
module_init(foo_init);
static void __exit foo_cleanup(void)
{
   [...] /* My clean up code */
   spi_unregister_driver(&foo_driver);
}
module_exit(foo_cleanup);
```

If you do nothing at module initialization other than registering/unregistering the driver, you can use `module_spi_driver()` to factor your code as follows:

```
module_spi_driver(foo_driver);
```

This macro will populate module initialization and cleanup functions and will call `spi_register_driver` and `spi_unregister_driver` inside.

## Instantiating SPI devices

SPI slave nodes must be children of the SPI controller node. In master mode, one or more slave nodes (up to the number of CSs) can be present.

The required properties are the following:

*   `compatible`: The compatible string as defined in the driver for matching
*   `reg`: The CS index of the device relative to the controller
*   `spi-max-frequency`: The maximum SPI clocking speed of the device in Hz

All slave nodes can contain the following optional properties:

*   `spi-cpol`: Boolean property which, if present, indicates that the device requires inverse **clock polarity** (**CPOL**) mode.
*   `spi-cpha`: Boolean property indicating that this device requires shifted **clock phase** (**CPHA**) mode.
*   `spi-cs-hi–h`: Empty property indicating that the device requires CS active high.
*   `spi-3wire`: This is a Boolean property that indicates that this device requires 3-wire mode to work properly.
*   `spi-lsb-first`: This is a Boolean property that indicates that this device requires LSB first mode.
*   `spi-tx-bus-width`: This property indicates the bus width used for MOSI. If not present, it defaults to `1`.
*   `spi-rx-bus-width`: This property is used to indicate the bus width used for MISO. If not present, it defaults to `1`.
*   `spi-rx-delay-–s`: This is used to specify a delay in microseconds after a read transfer.
*   `spi-tx-delay-us`: This is used to specify a delay in microseconds after a write transfer.

The following is a real device tree listing for SPI devices:

```
ecspi1 {
    fsl,spi-num-CSs = <3>;
    cs-gpios = <&gpio5 17 0>, <&gpio5 17 0>, <&gpio5 17 0>;
    pinctrl-0 = <&pinctrl_ecspi1 &pinctrl_ecspi1_cs>;
    #address-cells = <1>;
    #size-cells = <0>;
    compatible"= "fsl,imx6q-ec"pi", "fsl,imx51-ec"pi";
    reg = <0x02008000 0x4000>;
    status"= "o"ay";
    ad7606r8_0: ad7606r8@0 {
        compatible"= "ad760"-8";
        reg = <0>;
        spi-max-frequency = <1000000>;
        interrupt-parent = <&gpio4>;
        interrupts = <30 0x0>;
   };
   label: fake_spi_device@1 {
        compatible"= "packtpub,foobar-dev"ce";
        reg = <1>;
        a-string-param"= "stringva"ue";
        spi-cs-high;
   };
   mcp2515can: can@2 {
        compatible"= "microchip,mcp2"15";
        reg = <2>;
        spi-max-frequency = <1000000>;
        clocks = <&clk8m>;
        interrupt-parent = <&gpio4>;
        interrupts = <29 IRQ_TYPE_LEVEL_LOW>;
    };
};
```

In the preceding device tree excerpt, `ecspi1` represents the master SPI controller. `fake_spi_device` and `mcp2515can` represent SPI slave devices, and their `reg` properties represents their respective CS indices relative to the master.

Now that we are familiar with all the kernel aspects of the SPI slave-oriented framework, let's see how we might avoid dealing with the kernel and try to implement everything in the user space.

# Learning how not to write SPI device drivers

The usual way to deal with SPI devices is to write kernel code to drive this device. Nowadays the `spidev` interface makes it possible to deal with such devices without even writing a line of kernel code. The use of this interface should be limited, however, to simple use cases such as talking to a slave microcontroller or for prototyping. Using this interface, you will not be able to deal with various **interrupts** (**IRQs**) the device may support nor leverage other kernel frameworks.

The `spidev` interface exposes a character device node in the form `/dev/spidevX.Y` where `X` represents the bus our device sits on, and `Y` represents the CS index (relative to the controller) assigned to the device node in the device tree. For example, `/dev/spidev1.0` means device `0` on SPI bus `1`. The same applies to the sysfs directory entry, which would be in the form `/sys/class/spidev/spidevX.Y`.

Prior to the character device appearing in the user space, the device node must be declared in the device tree as a child of the SPI controller node. The following is an example:

```
&ecspi2 {
    pinctrl-names"= "defa"lt";
    pinctrl-0 = <&pinctrl_teoulora_ecspi2>;
    cs-gpios = <&gpio2 26 1
                &gpio2 27 1>;
    num-cs = <2>;
    status"= "o"ay";
    spidev@0 {
        reg = <0>;
        compatib"e="semtech,sx1"01";
        spi-max-frequency = <20000000>;
    };
};
```

In the preceding snippet, `spidev@0` corresponds to our SPI device node. `reg = <0>` tells the controller that this device is using the first CS line (index starting from 0). The `compatible="semtech,sx1301"` property is used to match an entry in the `spidev` driver. It is no longer recommended to use `"spidev"` as a compatible string – you'll get a warning if you try. Finally, `spi-max-frequency = <20000000>` sets the default clock speed (20 MHz in this case) that our device will operate at, unless it is changed using the appropriate API.

From the user space, the required header files to deal with the `spidev` interface are as follows:

```
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>
#include <linux/types.h>
#include <linux/spi/spidev.h>
```

Because it is a character device, it is allowed (this is the only option, in fact) to use basic system calls such as `open()`, `read()`, `write()`, `ioctl()`, and `close()`. The following example shows some basic usage, with `read()` and `write()` operations only:

```
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char **argv) 
{
   int i,fd;
   char *device = "/dev/spidev0.0";
   char wr_buf[]={0xff,0x00,0x1f,0x0f};
   char rd_buf[10]; 

   fd = open(device, O_RDWR);
   if (fd <= 0) { 
         printf("Failed to open SPI device %s\n", device);
         exit(1);
   }

   if (write(fd, wr_buf, sizeof(wr_buf)) != sizeof(wr_buf))
         perror("Write Error");
   if (read(fd, rd_buf, sizeof(rd_buf)) != sizeof(rd_buf))
         perror("Read Error");
   else
         for (i = 0; i < sizeof(rd_buf); i++)
             printf("0x%02X ", rd_buf[i]);
   close(fd);
   return 0;
}
```

In the preceding code, you should note that the standard `read()` and `write()` operations are half-duplex only, and that the CS is deactivated between each operation. To be able to work in full duplex, you have no choice but to use the `ioctl()` interface, where you can pass both input and output buffers at your convenience. Moreover, with the `ioctl()` interface, you can use a set of `SPI_IOC_RD_*` and `SPI_IOC_WR_*` commands to get `RD` and set `WR` to override the device's current setting. The complete list and documentation for this can be found in `Documentation/spi/spidev` in the kernel sources.

The `ioctl()` interface allows composite operations without CS deactivation and is available using the `SPI_IOC_MESSAGE(N)` request. A new data structure takes place, the `struct spi_ioc_transfer`, which is the user space equivalent of `struct spi_transfer`. The following is an example of the ioctl commands:

```
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
/* include required headers, listed early in the section */
[...]
static int pabort(const char *s)
{
    perror(s);
    return -1;
}
static int spi_device_setup(int fd)
{
    int mode, speed, a, b, i;
    int bits = 8;
    /* spi mode: mode 0 */
    mode = SPI_MODE_0;
    a = ioctl(fd, SPI_IOC_WR_MODE, &mode); /* set mode */
    b = ioctl(fd, SPI_IOC_RD_MODE, &mode); /* get mode */
    if ((a < 0) || (b < 0)) {
        return pabort("can't set spi mode");
    }
    /* Clock max speed in Hz */
    speed = 8000000; /* 8 MHz */
    a = ioctl(fd, SPI_IOC_WR_MAX_SPEED_HZ, &speed); /* set */
    b = ioctl(fd, SPI_IOC_RD_MAX_SPEED_HZ, &speed); /* get */
    if ((a < 0) || (b < 0))
        return pabort("fail to set max speed hz");
    /*
     * Set SPI to MSB first.
     * Here, 0 means "not to use LSB first".
     * To use LSB first, argument should be > 0
     */
    i = 0;
    a = ioctl(dev, SPI_IOC_WR_LSB_FIRST, &i);
    b = ioctl(dev, SPI_IOC_RD_LSB_FIRST, &i);
    if ((a < 0) || (b < 0))
        pabort("Fail to set MSB first\n");

    /* setting SPI to 8 bits per word */
    bits = 8;
    a = ioctl(dev, SPI_IOC_WR_BITS_PER_WORD, &bits); /* set */
    b = ioctl(dev, SPI_IOC_RD_BITS_PER_WORD, &bits); /* get */
    if ((a < 0) || (b < 0))
        pabort("Fail to set bits per word\n");

    return 0;
}
```

In the preceding example, getters are used for demonstration purposes only. It is not mandatory to issue the `SPI_IOC_RD_*` command after you have executed its `SPI_IOC_WR_*` equivalent. Now that we have seen most of those ioctl commands, let's see how to start transfers:

```
static void do_transfer(int fd)
{
    int ret;
    char txbuf[] = {0x0B, 0x02, 0xB5};
    char rxbuf[3] = {0, };
    char cmd_buff = 0x9f;
    struct spi_ioc_transfer tr[2] = {
        0 = {
          .tx_buf = (unsigned long)&cmd_buff,
          .len = 1,
          .cs_change = 1;    /* We need CS to change */
          .delay_usecs = 50, /* wait after this transfer */
          .bits_per_word = 8,
        },
        [1] = {
          .tx_buf = (unsigned long)tx,
          .rx_buf = (unsigned long)rx,
          .len = txbuf(tx),
          .bits_per_word = 8,
        },
    };
    ret = ioctl(fd, SPI_IOC_MESSAGE(2), &tr);
    if (ret == 1){
        perror("can't send spi message");
        exit(1);
    }
    for (ret = 0; ret < sizeof(tx); ret++)
        printf("%.2X ", rx[ret]);
    printf("\n");
}
```

The preceding shows the concept of message and transfer transactions in the user space. Now that our helpers have been defined, we can write the main code to use them as follows:

```
int main(int argc, char **argv)
{
    char *device = "/dev/spidev0.0";
    int fd;
    int error;
    fd = open(device, O_RDWR);
    if (fd < 0)
        return pabort("Can't open device ");
    error = spi_device_setup(fd);
    if (error)
        exit (1);

    do_transfer(fd);

    close(fd);
    return 0;
}
```

We are now done with the main function. This section taught us to use the user space SPI APIs and commands to interact with the device. We are limited, however, in that we can't take advantage of device interrupt lines or other kernel frameworks.

# Summary

In this chapter, we tackled SPI drivers and can now take the advantage of this serial (and full duplex) bus, which is way faster than I2C. We walked through all the data structures in this framework and discussed transferring over SPI, which is the most important section we covered. That said, the memory we accessed over those buses was off-chip – we may need more abstraction in order to avoid the SPI and I2C APIs.

This is where the next chapter comes in, dealing with the regmap API, which offers a higher and more unified level of abstraction so that SPI (and I2C) commands will become transparent to you.