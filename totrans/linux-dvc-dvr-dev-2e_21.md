# *Chapter 17*: Leveraging the Linux Kernel Input Subsystem

Input devices are devices that you can use to interact with the system. Such devices include buttons, keyboards, touchscreens, mice, and more. They work by sending events that are caught and broadcast over the system by the input core. This chapter will explain each structure that's used by the input core to handle input devices, as well as how to manage events from the user space.

In this chapter, we will cover the following topics:

*   Introduction to the Linux kernel input subsystem – its data structures and APIs
*   Allocating and registering an input device
*   Using polled input devices

*   Generating and reporting input events
*   Handling input devices from the user space

# Introduction to the Linux kernel input subsystem – its data structures and APIs

The main data structures and APIs of this subsystem can be found in the `include`/`linux`/`input.h` files. The following line is required in any input device driver:

```
#include <linux/input.h>
```

Whatever type of input device it is, whatever type of event it sends, an input device is represented in the kernel as an instance of the struct `input_dev`:

```
struct input_dev {
  const char *name;
  const char *phys;
  unsigned long evbit[BITS_TO_LONGS(EV_CNT)];
  unsigned long keybit[BITS_TO_LONGS(KEY_CNT)];
  unsigned long relbit[BITS_TO_LONGS(REL_CNT)];
  unsigned long absbit[BITS_TO_LONGS(ABS_CNT)];
  unsigned long mscbit[BITS_TO_LONGS(MSC_CNT)];
  unsigned int repeat_key;
  int rep[REP_CNT];
  struct input_absinfo *absinfo;
  unsigned long key[BITS_TO_LONGS(KEY_CNT)];
  int (*open)(struct input_dev *dev);
  void (*close)(struct input_dev *dev);
  unsigned int users;
  struct device dev;
  unsigned int num_vals;
  unsigned int max_vals;
  struct input_value *vals;
  bool devres_managed;
};
```

For the sake of readability, some elements in the structure have been omitted. Let's look at these fields in more detail:

*   `name` represents the name of the device.
*   `phys` is the physical path to the device in the system hierarchy.
*   `evbit` is a bitmap of the types of events that are supported by the device. The following are some events:
    *   `EV_KEY` is for devices that support sending key events (for example, keyboards, buttons, and so on)
    *   `EV_REL` is for devices that support sending relative positions (for example, mice, digitizers, and so on)
    *   `EV_ABS` is for devices that support sending absolute positions (for example, joysticks) The list of events is available in the kernel source in the`include/linux/input-event-codes.h` file. You can use the `set_bit()` macro to set the appropriate bit, depending on your input device's capabilities. Of course, a device can support more than one type of event. For example, a mouse driver will set both `EV_KEY` and `EV_REL`, as shown here:

        ```
            set_bit(EV_KEY, my_input_dev->evbit);
            set_bit(EV_REL, my_input_dev->evbit);
        ```

*   `keybit` is for `EV_KEY` enabled devices and consists of a bitmap of keys/buttons that this device exposes; for example, `BTN_0`, `KEY_A`, `KEY_B`, and so on. The complete list of keys/buttons can be found in the `include/linux/input-event-codes.h` file.
*   `relbit` is for `EV_REL` enabled devices and consists of a bitmap of relative axes for the device; for example, `REL_X`, `REL_Y`, `REL_Z`, and so on. Have a look at `include/linux/input-event-codes.h` for the complete list.
*   `absbit` is for `EV_ABS` enabled devices and consists of a bitmap of absolute axes for the device; for example, `ABS_Y`, `ABS_X`, and so on. Have a look at the same previous file for the complete list.
*   `mscbit` is for `EV_MSC` enabled devices and consists of a bitmap of miscellaneous events that are supported by the device.
*   `repeat_key` stores the key code of the last key pressed; it is used when the autorepeat feature is implemented by the software.
*   `rep` stores the current values for auto repeat parameters, typically the delay and rate.
*   `absinfo` is an array of `&struct input_absinfo` elements that holds information about the absolute axes (the current value, `min`, `max`, `flat`, `fuzz`, and the resolution). You should use the `input_set_abs_params()` function to set those values:

    ```
    void input_set_abs_params(struct input_dev *dev,
                           unsigned int axis, int min,
                           int max, int fuzz, int flat)
    ```

*   `min` and `max` specify the lower and upper bound values, respectively. `fuzz` indicates the expected noise on the specified channel of the specified input device. In the following examples, we're setting each channel's bound:

    ```
    #define ABSMAX_ACC_VAL      0x01FF
    #define ABSMIN_ACC_VAL      -(ABSMAX_ACC_VAL)
    [...]
    set_bit(EV_ABS, idev->evbit);
    input_set_abs_params(idev, ABS_X, ABSMIN_ACC_VAL,
                         ABSMAX_ACC_VAL, 0, 0);
    input_set_abs_params(idev, ABS_Y, ABSMIN_ACC_VAL,
                         ABSMAX_ACC_VAL, 0, 0);
    input_set_abs_params(idev, ABS_Z, ABSMIN_ACC_VAL,
                         ABSMAX_ACC_VAL, 0, 0);
    ```

*   `key` reflects the current state of the device's keys/buttons.
*   `open` is a method that's called when the very first user calls `input_open_device()`. Use this method to prepare the device, such as to interrupt a request, poll a thread start, and so on.
*   `close` is called when the very last user calls `input_close_device()`. Here, you can stop polling (which consumes a lot of resources).
*   `users` stores the number of users (input handlers) that opened this device. It is used by `input_open_device()` and `input_close_device()` to ensure that `dev->open()` is only called when the first user opens the device and that `dev->close()` is only called when the very last user closes the device.
*   `dev` is the struct device associated with this device (for device model).
*   `num_vals` is the number of values that are queued in the current frame.
*   `max_vals` is the maximum number of values that are queued in a frame.
*   `Vals` is the array of values that are queued in the current frame.
*   `devres_managed` indicates that the devices are managed with the **devres** framework and don't need to be explicitly unregistered or freed.

Now that you're familiar with the main input device's data structure, we can start registering such devices within the system.

# Allocating and registering an input device

Before the events that are supported by an input device can be seen by the system, memory needs to be allocated for this device first using the `devm_input_allocate_device()` API. Then, the device needs to be registered with the system using `input_device_register()`. The former API will take care of freeing up the memory and unregistering the device when it leaves the system. However, non-managed allocation is still available but not recommended, `input_allocate_device()`. By using non-managed allocation, the driver becomes responsible for making sure that `input_unregister_device()` and `input_free_device()` are called to unregister the device and free its memory when they're on the unloading path of the driver, respectively. The following are the respective prototypes of these APIs:

```
struct input_dev *input_allocate_device(void)
struct input_dev *devm_input_allocate_device(
                                       struct device *dev)
void input_free_device(struct input_dev *dev)
int input_register_device(struct input_dev *dev)
void input_unregister_device(struct input_dev *dev)
```

Device allocation may sleep, so it must not be called in the atomic context or with a spinlock being held. The following is an excerpt of the `probe` function of an input device sitting on the I2C bus:

```
struct input_dev *idev;
int error;
/*
 * such allocation will take care of memory freeing and
 * device unregistering
 */
idev = devm_input_allocate_device(&client->dev);
if (!idev)
    return -ENOMEM;
idev->name = BMA150_DRIVER;
idev->phys = BMA150_DRIVER "/input0";
idev->id.bustype = BUS_I2C;
idev->dev.parent = &client->dev;
set_bit(EV_ABS, idev->evbit);
input_set_abs_params(idev, ABS_X, ABSMIN_ACC_VAL,
                     ABSMAX_ACC_VAL, 0, 0);
input_set_abs_params(idev, ABS_Y, ABSMIN_ACC_VAL,
                     ABSMAX_ACC_VAL, 0, 0);
input_set_abs_params(idev, ABS_Z, ABSMIN_ACC_VAL,
                     ABSMAX_ACC_VAL, 0, 0);
error = input_register_device(idev);
if (error)
    return error;
error = devm_request_threaded_irq(&client->dev,
            client->irq,
            NULL, my_irq_thread,
            IRQF_TRIGGER_RISING | IRQF_ONESHOT,
            BMA150_DRIVER, NULL);
if (error) {
    dev_err(&client->dev, "irq request failed %d, 
           error %d\n", client->irq, error);
    return error;
}
return 0;
```

As you may have noticed, in the preceding code, no memory freeing nor device unregistering is performed when an error occurs because we have used the managed allocation for both the input device and the IRQ. That said, the input device has an IRQ line so that we're notified of a state change on the underlying device. This is not always the case as the system may lack available IRQ lines, in which case the input core will have to poll the device frequently so that it doesn't miss events. We discuss this in the next section.

# Using polled input devices

Polled input devices are special input devices that rely on polling to sense device state changes; the generic input device type relies on IRQ to sense changes and send events to the input core.

A polled input device is described in the kernel as an instance of `struct input_polled_dev` structure, which is a wrapper around the generic `struct input_dev` structure. The following is its declaration:

```
struct input_polled_dev {
    void *private;
    void (*open)(struct input_polled_dev *dev);
    void (*close)(struct input_polled_dev *dev);
    void (*poll)(struct input_polled_dev *dev);
    unsigned int poll_interval; /* msec */
    unsigned int poll_interval_max; /* msec */
    unsigned int poll_interval_min; /* msec */

    struct input_dev *input;
    bool devres_managed;
};
```

Let's take a look at the elements in this structure:

*   `private` is the driver's private data.
*   `open` is an optional method that prepares the device for polling (enables the device and sometimes flushes the device's state).
*   `close` is an optional method that is called when the device is no longer being polled. It is used to put devices into low power mode.
*   `poll` is a mandatory method that's called whenever the device needs to be polled. It is called at the frequency of `poll_inteval`.
*   `poll_interval` is the frequency at which the `poll()` method should be called. It defaults to 500 milliseconds unless it's overridden when you're registering the device.
*   `poll_interval_max` specifies the upper bound for the poll interval. It defaults to the initial value of `poll_interval`.
*   `poll_interval_min` specifies the lower bound for the poll interval. It defaults to 0.
*   `input` is the input device that the polled device is built around. It must be initialized by the driver (by its ID, name, and bits). The polled input device just provides an interface to use polling instead of IRQ, to sense device state change.

Memory can be allocated for a polled input device using `devm_input_allocate_polled_device()`. This is a managed allocation API that takes care of freeing memory and unregistering the device as appropriate. Similarly, the non-managed API can be used for allocation, `input_allocate_polled_device()`, in which case you must take care of calling `input_free_polled_device()` by yourself. The following code shows the prototypes of those APIs:

```
struct input_polled_dev
     *devm_input_allocate_polled_device(
                                  struct device *dev)
struct input_polled_dev *input_allocate_polled_device(void)
void input_free_polled_device(struct 
                               input_polled_dev *dev)
```

For resource-managed devices, the `input_dev->devres_managed` field will be set to `true` by the input core. Then, you should take care of initializing the mandatory fields of the underlying `struct input_dev`, as we saw in the previous section. The polling interval must be set too; otherwise, it will default to 500 ms.

Once the fields have been allocated and initialized, the polled input device can be registered using `input_register_polled_device()`, which returns `0` on success. For managed allocation, unregistering is handled by the system; you need to call `input_unregister_polled_device()` by yourself to perform the reverse operation. The following are their prototypes:

```
int input_register_polled_device(
                        struct input_polled_dev *dev)
void  input_unregister_polled_device(
                        struct input_polled_dev *dev)
```

A typical example of the `probe()` function for such a device may look as follows. First, we define the driver data structure, which will gather all the necessary resources:

```
struct my_struct {
    struct input_pulled_dev *polldev;
    struct gpio_desc *gpio_btn;
    [...]
}
```

Once the driver data structure has been defined, the `probe()` function can be implemented. The following is its body:

```
static int button_probe(struct platform_device *pdev)
{
    struct my_struct *ms;
    struct input_dev *input_dev;
    int error;
    struct device *dev = &pdev->dev;
    ms = devm_kzalloc(dev, sizeof(*ms), GFP_KERNEL);
    if (!ms)
        return -ENOMEM;
    ms->polldev = devm_input_allocate_polled_device(dev);
    if (!ms->polldev)
        return -ENOMEM;
    /* This gpio is not mapped to IRQ */
    ms->gpio_btn = devm_gpiod_get(dev, "my-btn", GPIOD_IN);
    ms->polldev->private = ms;
    ms->polldev->poll = my_btn_poll;
    ms->polldev->poll_interval = 200;/* Poll every 200ms */
    ms->polldev->open = my_btn_open;
     /* Initializing the underlying input_dev fields */
    input_dev = ms->poll_dev->input;
    input_dev->name = "System Reset Btn";
    /* The gpio belongs to an expander sitting on I2C */
    input_dev->id.bustype = BUS_I2C; 
    input_dev->dev.parent = dev;
    /* Declare the events generated by this driver */
    set_bit(EV_KEY, input_dev->evbit);
    set_bit(BTN_0, input_dev->keybit); /* buttons */
    retval = input_register_polled_device(ms->poll_dev);
    if (retval) {
        dev_err(dev, "Failed to register input device\n");
        return retval;  
    }
    return 0;
}
```

Once again, neither unregistering nor freeing are handled by ourselves when an error occurs because we have used managed allocations.

The following is what our `open` callback may look like:

```
static void my_btn_open(struct input_polled_dev *poll_dev)
{
    struct my_strut *ms = poll_dev->private;
    dev_dbg(&ms->poll_dev->input->dev, "reset open()\n");
}
```

In our example, it does nothing. However, the `open` method is used to prepare the resources that are needed by the device.

Deciding whether you should implement a polled input device is straightforward. The usual way is to use classic input devices if an IRQ line is available; alternatively, you can fall back to the polled device:

```
if(client->irq > 0){
    /* Use generic input device */
} else {
    /* Use polled device */
}
```

Other elements may need to be considered when you're choosing between implementing a polled input device or an IRQ-based one; the preceding code is just a suggestion.

Now that we are familiar with this subset of input devices, we can consider registering and unregistering input devices. That said, even though the input device has been registered, we can't interact with it yet. In the next section, we will learn how the input device can report events to the kernel.

# Generating and reporting input events

Device allocation and registration are essential, but they are useless if the device is unable to report events to the input core, which is what input devices are designed to do. Depending on the type of event our device can support, the kernel provides the appropriate APIs to report them to the core.

Given an `EV_XXX` capable device, the corresponding report function would be `input_report_xxx()`. The following table shows the mappings between the most important event types and their report functions:

![Table 17.1 – Mapping the input device's capabilities and the report APIs
](img/B17934_Table_01.jpg)

Table 17.1 – Mapping the input device's capabilities and the report APIs

The prototypes for these report APIs are as follows:

```
void input_report_abs(struct input_dev *dev,
                      unsigned int code, int value)
void input_report_key(struct input_dev *dev,
                      unsigned int code, int value)
void input_report_rel(struct input_dev *dev,
                      unsigned int code, int value)
```

The list of available report functions can be found in `include/linux/input.h` in the kernel source file. They all have the same skeleton:

*   `dev` is the input device that's responsible for the event.
*   `code` represents the event code; for example, `REL_X` or `KEY_BACKSPACE`. The complete list can be found in `include/linux/input-event-codes.h`.
*   `value` is the value the event carries. For an `EV_REL` event type, it carries the relative change. For an `EV_ABS` (joysticks and so on) event type, it contains an absolute new value. For an `EV_KEY` event type, it should be set to `0` for key release, `1` for a keypress, and `2` for auto-repeat.

Once all these changes have been reported, the driver should call `input_sync()` on the input device to indicate that this event is complete. The input subsystem will collect these events into a single packet and send it through `/dev/input/event<X>`, which is the character device that represents our `struct input_dev` on the system. Here, `<X>` is the interface number that's been assigned to the driver by the input core:

```
void input_sync(struct input_dev *dev)
```

Let's look at an example of this. The following is an excerpt from the `drivers/input/misc/bma150.c`:

```
static void threaded_report_xyz(struct bma150_data *bma150)
{
  u8 data[BMA150_XYZ_DATA_SIZE];
  s16 x, y, z;
  s32 ret;
  ret = i2c_smbus_read_i2c_block_data(bma150->client,
                       BMA150_ACC_X_LSB_REG,
                       BMA150_XYZ_DATA_SIZE, data);
  if (ret != BMA150_XYZ_DATA_SIZE)
    return;
  x = ((0xc0 & data[0]) >> 6) | (data[1] << 2);
  y = ((0xc0 & data[2]) >> 6) | (data[3] << 2);
  z = ((0xc0 & data[4]) >> 6) | (data[5] << 2);
  /* sign extension */
  x = (s16) (x << 6) >> 6;
  y = (s16) (y << 6) >> 6;
  z = (s16) (z << 6) >> 6;
  input_report_abs(bma150->input, ABS_X, x);
  input_report_abs(bma150->input, ABS_Y, y);
  input_report_abs(bma150->input, ABS_Z, z);
  /* Indicate this event is complete */
  input_sync(bma150->input);
}
```

In the preceding excerpt, `input_sync()` tells the core to consider the three reports as the same event. This makes sense since the position has three axes (*X*, *Y*, and *Z*) and we do not want *X*, *Y*, or *Z* to be reported separately.

The best place to report the event is inside the poll function for a polled device or the IRQ routine (threaded part or not) for an IRQ-enabled device. If you perform some operations that may sleep, you should process your report inside the threaded part of the IRQ handler. The following code shows how our initial example could implement the `poll` method:

```
static void my_btn_poll(struct input_polled_dev *poll_dev)
{
    struct my_struct *ms = polldev->private;
    struct i2c_client *client = mcp->client;

    input_report_key(polldev->input, BTN_0,
                     gpiod_get_value(ms->rgpio_btn) & 1);
    input_sync(poll_dev->input);
}
```

In the preceding code, our input device reports the `0` key code. In the next section, we will discuss how the user space can handle those report events and codes.

# Handling input devices from the user space

A node will be created in the `/dev/input/` directory for each input device (polled or not) that has been successfully registered with the system. In my case, the node corresponds to `event0` because it is the first and only input device on my target board. You can use the `udevadm` tool to display information about the device:

```
# udevadm info /dev/input/event0 
P: /devices/platform/input-button.0/input/input0/event0
N: input/event0
S: input/by-path/platform-input-button.0-event
E: DEVLINKS=/dev/input/by-path/platform-input-button.0-event
E: DEVNAME=/dev/input/event0
E: DEVPATH=/devices/platform/input-button.0/input/input0/event0
E: ID_INPUT=1
E: ID_PATH=platform-input-button.0
E: ID_PATH_TAG=platform-input-button_0
E: MAJOR=13
E: MINOR=64
E: SUBSYSTEM=input
E: USEC_INITIALIZED=74842430
```

Another tool that you can use, which allows you to print the keys that are supported by the device, is `evetest`. It can also catch and print events when they are reported by the device. The following code shows its usage on our input device:

```
# evtest /dev/input/event0 
input device opened()
Input driver version is 1.0.1
Input device ID: bus 0x0 vendor 0x0 product 0x0 version 0x0
Input device name: "Packt Btn"
Supported events:
   Event type 0 (EV_SYN)
   Event type 1 (EV_KEY)
      Event code 256 (BTN_0)
```

Not only the input devices we have written drivers for can be managed with `evetest`. In the following example, I am using the USB-C headset that's connected to my computer. It has input device capabilities since it provides volume-related keys:

```
jma@labcsmart-sqy:~$ sudo evtest /dev/input/event4
Input driver version is 1.0.1
Input device ID: bus 0x3 vendor 0x12d1 product 0x3a07 version 0x111
Input device name: "Synaptics HUAWEI USB-C HEADSET"
Supported events:
  Event type 0 (EV_SYN)
  Event type 1 (EV_KEY)
    Event code 114 (KEY_VOLUMEDOWN)
    Event code 115 (KEY_VOLUMEUP)
    Event code 164 (KEY_PLAYPAUSE)
    Event code 582 (KEY_VOICECOMMAND)
  Event type 4 (EV_MSC)
    Event code 4 (MSC_SCAN)
Properties:
Testing ... (interrupt to exit)
Event: time 1640231369.347093, type 4 (EV_MSC), code 4 (MSC_SCAN), value c00e9
Event: time 1640231369.347093, type 1 (EV_KEY), code 115 (KEY_VOLUMEUP), value 1
Event: time 1640231369.347093, -------------- SYN_REPORT ------------
Event: time 1640231369.487017, type 4 (EV_MSC), code 4 (MSC_SCAN), value c00e9
Event: time 1640231369.487017, type 1 (EV_KEY), code 115 (KEY_VOLUMEUP), value 0
Event: time 1640231369.487017, -------------- SYN_REPORT ------------
```

In the preceding code, I pushed the up volume key to see how it is reported. `evtest` can even be used with your keyboard, with the only condition being that you identify the corresponding input device node in `/dev/input/`.

As we have seen, every registered input device is represented by a `/dev/input/event<X>` character device, which we can use to read the event from the user space. An application that's reading this file will receive event packets in the `struct input_event` format, which has the following declaration:

```
struct input_event {
  struct timeval time;
  __u16 type;
  __u16 code;
  __s32 value;
}
```

Let's look at the meaning of each element in the structure:

*   `time` is a timestamp that corresponds to the time when the event happened.
*   `type` is the event type; for example, `EV_KEY` for a keypress or release, `EV_REL` for a relative moment, or `EV_ABS` for an absolute one. More types are defined in `include/linux/input-event-codes.h`.
*   `code` is the event code; for example, `REL_X` or `KEY_BACKSPACE`. Again, a complete list can be found in `include/linux/input-event-codes.h`.
*   `value` is the value that the event carries. For an `EV_REL` event type, it carries the relative change. For an `EV_ABS` (joysticks and so on) event type, it contains the absolute new value. For an `EV_KEY` event type, it is set to `0` for a key release, `1` for a keypress, and `2` for auto-repeat.

A user space application can use blocking and non-blocking reads, but also `poll()` or `select()` system calls, to be notified of events after opening this device. The following is an example of the `select()` system call. Let's start by enumerating the headers we need to implement our example:

```
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <linux/input.h>
#include <sys/select.h>
```

Then, we must define our input device path as a macro as it will be used often:

```
#define INPUT_DEVICE "/dev/input/event0"
int main(int argc, char **argv)
{   
    int fd, ret;
    struct input_event event;
    ssize_t bytesRead;
    fd_set readfds;
```

Next, we must open the input device and keep its file descriptor for later use. Failing to open the input device is considered as an error, so we must exit the program:

```
    fd = open(INPUT_DEVICE, O_RDONLY);
    if(fd < 0){
        fprintf(stderr,
             "Error opening %s for reading", INPUT_DEVICE);
        exit(EXIT_FAILURE);
    }
```

Now, we have a file descriptor representing our opened input device. We can use the `select()` system call to sense any key press or release:

```
    while(1){ 
        FD_ZERO(&readfds);
        FD_SET(fd, &readfds);
        ret = select(fd + 1, &readfds, NULL, NULL, NULL);
        if (ret == -1) {
            fprintf(stderr,
                   "select call on %s: an error ocurred",
                    INPUT_DEVICE);
            break;
        }
        else if (!ret) { /* If we used timeout */
            fprintf(stderr,
                    "select on %s: TIMEOUT", INPUT_DEVICE);
            break;
        }
```

At this point, we have done the necessary sanity checks on the return path of `select()`. Note that `select()` returns zero if it timed out before any file descriptors became ready, hence `else if` in the preceding code.

The change is effective now, let's read the data to see what it corresponds to:

```
        /* File descriptor is now ready */
        if (FD_ISSET(fd, &readfds)) {
            bytesRead = read(fd, &event,
                             sizeof(struct input_event));
            if(bytesRead == -1)
                /* Process read input error*/
                [...]
            if(bytesRead != sizeof(struct input_event))
                /* Read value is not an input event */
                [...] /* handle this error */
```

If the execution flow reaches this mean, it means that everything went well. Now, we can walk through the events that are supported by our input device and compare them to the event that is reported by the input core before a decision is made:

```
            /* We could have done a switch/case if we had
             * many codes to look for */
            if (event.code == BTN_0) {
                /* it concerns our button */
                if (event.value == 0) {
                    /* Process keyRelease if need be */
                    [...]
                }
                else if(event.value == 1){
                    /* Process KeyPress */
                    [...]
                }
            }
        }
    }
    close(fd);
    return EXIT_SUCCESS;
}
```

For further debugging purposes, if your input device is based on GPIOs, you can successively push/release the button and check whether the GPIO's state has changed:

```
# cat /sys/kernel/debug/gpio  | grep button
 gpio-195 (gpio-btn         ) in  hi    
# cat /sys/kernel/debug/gpio  | grep button
 gpio-195 (gpio-btn         ) in  lo
```

Moreover, if the input device has an IRQ line, it may make sense to check the statistic for this IRQ line to make sure it is coherent. For example, here, we must check whether the request has succeeded and how many times it has been fired:

```
# cat /proc/interrupts | grep packt
160:      0      0      0      0  gpio-mxc   0  packt-input-button
```

In this section, we learned how to deal with the input device from the user space and provided some debugging tips for when something goes wrong. We used the `select()` system call to sense input events, though we could have used `poll()` as well.

# Summary

This chapter described the input framework and highlighted the difference between polled and interrupt-driven input devices. At this point, you should have the necessary knowledge to write a driver for any input driver, whatever its type, and whatever input event it supports. The user space interface was also discussed, and an example was provided.