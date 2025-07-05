# *Chapter 16*: Getting the Most Out of the Pin Controller and GPIO Subsystems

**System-on-chips** (**SoCs**) are becoming more and more complex and feature-rich. These features are mostly exposed through electrical lines originating from the SoC and are called pins. Most of these pins are routed to or multiplexed with several functional blocks (for instance, UART, SPI, RGMI, **General-Purpose Input Output** (**GPIO**), and so on), and the underlying device responsible for configuring these pins and switching between operating modes (switching between functional blocks) is called the **pin controller**.

One mode in which such pins can be configured is **GPIO**. Then comes the Linux GPIO subsystem, which enables drivers to read signals on GPIO configured pins as high or low and to drive the signal high/low on GPIO configured pins. On the other hand, the **pin control** (abbreviated **pinctrl**) subsystem enables multiplexing of some pin/pin groups for different functions, and the capability to configure the electrical properties of pins, such as slew rate, pull up/down resistor, hysteresis, and so on.

To summarize, the pin controller mainly does two things: pin multiplexing, that is, reusing the same pin for different purposes, and pin configuration, that is, configuring electronic properties of pins. Then, the GPIO subsystem allows driving pins, provided these pins are configured to work in GPIO mode by the pin controller.

In this chapter, pin controller and GPIO subsystems will be introduced via the following topics:

*   Introduction to some hardware terms
*   Introduction to the pin control subsystem
*   Dealing with the GPIO controller interface
*   Getting the most out of the GPIO consumer interface
*   Learning how not to write GPIO client drivers

# Introduction to some hardware terms

The Linux kernel GPIO subsystem is not just about GPIO toggling. It is tightly coupled to the pin controller subsystem; they share some terms and concepts that we need to introduce:

*   **Pin** and **pad**: A pin is a physical input or output wire/line that transports an electrical signal from or to a component. In schematics, the term "pin" is widely used. Contact pads, on the other hand, are the contact surface areas of a printed circuit board or an integrated circuit. As a result, a pin comes from a pad, and a pin is a pad by default.
*   **GPIO**: Most MCUs and CPUs can share one pad among several functional blocks. This is accomplished by multiplexing the input and output signals of the pad. The different modes the pin/pad can operate in are known as **ALT modes** (or alternate modes), and it is common for CPUs to support up to eight settings (or modes) per pad. GPIO is one of these modes. It allows changing the pin direction and reading its value when it is configured as input or setting its value when it is configured as output. Other modes are ADC, UART Tx, UART Rx, SPI MOSI, SPI MISO, PWM, and so on.
*   **Pin controller**: This is the underlying device or controller (or rather, a group of registers) allowing you to perform **pin multiplexing** (also referred to as **pinmux** or **pinmuxing**) to reuse the same pin for different purposes. Apart from pinmuxing, it allows pin configuration, that is, configuring electronic properties of pins. The following are some of these properties:
    *   **Biasing**, that is, setting the initial operating conditions, for example, grounding the pins or connecting them to Vdd. This is not to be confused with pull up and pull down, which is another property.
    *   **Pin debounce**, which is the time after which a state should be considered valid. This, for example, prevents multiple key pushes on keypads attached to GPIO lines.
    *   **Slew rate**, which determines how fast the pin toggles between the two logic states. It allows us to control the rise and fall time for the output signals. A trade-off has to be found because rapidly changing states consume more power and generate spikes, thus low slew rates are preferred, except for quick control signals such as parallel interfaces: EIM, EB&, SPI, or SDRAM, which need fast toggling.
    *   Pull Up/Down resistors
*   **GPIO controller**: This is the device allowing you to drive pins when they are put in GPIO mode. It allows changing the GPIO direction and value.

Following the previous definitions, certain general rules have been established for writing pin controllers or GPIO controller drivers, and they are as follows:

*   If your GPIO/pin controller can only do simple GPIO, implement just `struct gpio_chip` in `drivers/gpio/gpio-foo.c` and leave it there. Do not use the generic or old-style number-based GPIO.
*   Keep your GPIO/pin controller in `drivers/gpio` if it can generate interrupts in addition to GPIO capabilities; simply fill in `struct irq_chip` and register it with the IRQ subsystem.
*   Implement composite pin controller drivers in `drivers/pinctrl/pinctrl-foo.c` if this controller supports pinmuxing, advanced pin driver strength, complicated biasing, and so on.
*   Maintain the `struct gpio_chip`, `struct irq_chip`, and `struct pinctrl_desc` interfaces.

Now that we are familiar with the terms related to the underlying hardware devices, let's introduce the Linux implementation, starting with the pin control subsystem.

# Introduction to the pin control subsystem

The pin controller allows gathering pins, the modes these pins should operate in, and their configurations. The driver is responsible for providing the appropriate set of callbacks according to the features that are to be implemented, provided the underlying hardware supports these features.

The pin controller descriptor data structure is defined as follows:

```
struct pinctrl_desc {
     const char *name;
     const struct pinctrl_pin_desc *pins;
     unsigned int npins;
     const struct pinctrl_ops *pctlops;
     const struct pinmux_ops *pmxops;
     const struct pinconf_ops *confops;
     struct module *owner;
[...]
};
```

In that pin controller data structure, only relevant elements have been listed, and the following are their meanings:

*   `name` is the name of the pin controller.
*   `pins`: An array of pin descriptors that describe all the pins that this controller can handle. It has to be noted that the controller side represents each pin/pad as an instance of `struct pinctrl_pin_desc`, defined as follows:

    ```
    struct pinctrl_pin_desc {
         unsigned number;
         const char *name;
    [...]
    };
    ```

In the preceding data structure, `number` represents the unique pin number from the global pin number space of the pin controller, and `name` is the name of this pin.

*   `npins`: The number of descriptors in the `pins` array, usually obtained using `ARRAY_SIZE()` in the `pins` field.
*   `pctlops` stores the pin control operation table, to support global concepts such as the grouping of pins. This is optional.
*   `pmxops` represents the **pinmux** operations table if you support pinmuxing in your driver.
*   `confops`: The pin configuration operations table if you support pin configuration in your driver.

Once the appropriate callbacks are defined and this data structure has been initialized, it can be passed to `devm_pinctrl_register()`, defined as the following:

```
struct pinctrl_dev *devm_pinctrl_register(
                      struct device *dev,
                      struct pinctrl_desc *pctldesc,
                      void *driver_data);
```

The preceding function will register the pin controller with the system, returning in the meantime a pointer to an instance of `struct pinctrl_dev`, representing the pin controller device, passed as a parameter to most (if not all) of the callback operations exposed by the pin controller driver. On error, the function returns an error pointer, which can be handled with `PTR_ERR`.

The controller's control, multiplexing, and configuration operation tables are to be set up according to the features supported by the underlying hardware. Their respective data structures are defined in the header files that must also be included in the driver, as follows:

```
#include <linux/pinctrl/pinconf.h>
#include <linux/pinctrl/pinconf-generic.h>
#include <linux/pinctrl/pinctrl.h>
#include <linux/pinctrl/pinmux.h>
```

When it comes to the pin control consumer interface, the following header must be used instead:

```
#include <linux/pinctrl/consumer.h>
```

Before being accessed by consumer drivers, pins must be assigned to the devices that need to control them. The recommended way to assign pins to devices is from the **device tree** (**DT**). How pins groups are assigned in the DT closely depends on the platform, thus the pin controller driver and its binding.

Every pin control state is assigned a contiguous integer ID that starts at 0\. A `name` property list can be used to map strings on top of these IDs to ensure that the same name always points to the same ID. It goes without saying that the set of states that must be defined in each device's DT node is determined by the binding of this device. This binding also determines whether to define the set of state IDs that must be provided, or whether to define the set of state names that must be provided. In any case, two properties can be used to assign a pin configuration node to a device:

*   `pinctrl-<ID>`: This allows you to provide the list of pin configurations needed for particular states of the device. It is a list of `phandles` identified by `<ID>`, each of which points to a pin configuration node. These referenced pin configuration nodes must be child nodes of (or nested in) the pin controller node they belong to. This property can accept multiple entries so that multiple groups of pins may be configured and used for a particular device state, allowing in the meantime to specify pins from different pin controllers.
*   `pinctrl-names`: This allows giving names to `pinctrl-<ID>` properties according to the state of the device owning the group(s) of pins. List entry 0 defines the name for the state whose ID is 0, list entry 1 for state ID 1, and so on. State ID 0 is commonly given the name `default`. A list of standardized states can be found in `include/linux/pinctrl/pinctrl-state.h`. However, clients or consumer drivers are free to implement any state they need, provided this state is documented in the device binding description.

Here is an excerpt of the DT, showing some device nodes, along with their pin control nodes. Let's name this excerpt `pinctrl-excerpt`:

```
&usdhc4 {
[...]
     pinctrl-0 = <&pinctrl_usdhc4_1>;
     pinctrl-names = "default";
};
gpio-keys {
    compatible = "gpio-keys";
    pinctrl-names = "default";
    pinctrl-0 = <&pinctrl_io_foo &pinctrl_io_bar>;
};
iomuxc@020e0000 { /* Pin controller node */
    compatible = "fsl,imx6q-iomuxc";
    reg = <0x020e0000 0x4000>;
    /* shared pinctrl settings */
    usdhc4 { /* first node describing the function */
        pinctrl_usdhc4_1: usdhc4grp-1 { /* second node */
            fsl,pins = <
                MX6QDL_PAD_SD4_CMD__SD4_CMD    0x17059
                MX6QDL_PAD_SD4_CLK__SD4_CLK    0x10059
                MX6QDL_PAD_SD4_DAT0__SD4_DATA0 0x17059
                MX6QDL_PAD_SD4_DAT1__SD4_DATA1 0x17059
                MX6QDL_PAD_SD4_DAT2__SD4_DATA2 0x17059
                MX6QDL_PAD_SD4_DAT3__SD4_DATA3 0x17059
                [...]
            >;
        };
    };
    [...]
    uart3 {
        pinctrl_uart3_1: uart3grp-1 {
            fsl,pins = <
                MX6QDL_PAD_EIM_D24__UART3_TX_DATA 0x1b0b1
                MX6QDL_PAD_EIM_D25__UART3_RX_DATA 0x1b0b1
            >;
        };
    };
    // GPIOs (Inputs)
   gpios {
        pinctrl_io_foo: pinctrl_io_foo {
           fsl,pins = <
              MX6QDL_PAD_DISP0_DAT15__GPIO5_IO09  0x1f059
              MX6QDL_PAD_DISP0_DAT13__GPIO5_IO07  0x1f059
           >;
        };
        pinctrl_io_bar: pinctrl_io_bar {
           fsl,pins = <
              MX6QDL_PAD_DISP0_DAT11__GPIO5_IO05  0x1f059
              MX6QDL_PAD_DISP0_DAT9__GPIO4_IO30   0x1f059
              MX6QDL_PAD_DISP0_DAT7__GPIO4_IO28   0x1f059
           >;
        };
    };
};
```

In the preceding example, a pin configuration is given in the form `<PIN_FUNCTION> <PIN_SETTING>`, where `<PIN_FUNCTION>` can be seen as the pin function or pin mode, and `<PIN_SETTING>` represents the pin's electrical properties:

```
MX6QDL_PAD_DISP0_DAT15__GPIO5_IO09 0x80000000
```

In the excerpt, `MX6QDL_PAD_DISP0_DAT15__GPIO5_IO09` represents the pin function/mode, which is GPIO in this case, and `0x80000000` represents the pin settings or electrical properties.

Let's consider another example as follows:

```
MX6QDL_PAD_EIM_D25__UART3_RX_DATA 0x1b0b1
```

In that excerpt, `MX6QDL_PAD_EIM_D25__UART3_RX_DATA` represents the pin function, which is the RX line of UART3, and `0x1b0b1` represents its electrical settings.

The pin function is a macro whose value is meaningful for the pin controller driver only. These are generally defined in header files located in `arch/<arch>/boot/dts/`. If you use an UDOO quad, for example, which has an i.MX6 quad core (32-bit ARM), the pin function header would be `arch/arm/boot/dts/imx6q-pinfunc.h`. The following is the macro corresponding to the fifth line of the GPIO5 controller:

```
#define MX6QDL_PAD_DISP0_DAT11__GPIO5_IO05 0x19c 0x4b0 0x000 0x5 0x0
```

`<PIN_SETTING>` can be used to set up things such as pull-ups, pull-downs, keepers, drive strength, and so on. How it should be specified depends on the pin controller binding, and the meaning of its value depends on the SoC datasheet, generally in the IOMUX section. On i.MX6 IOMUXC, only the lower 17 bits are used for this purpose.

Back to `pinctrl-excerpt`, prior to selecting a pin group and applying its configuration, the driver must first obtain a handle to this group of pins using the `devm_inctrl_get()` function and then select the appropriate state using `pinctrl_lookup_state()` before finally applying the corresponding configuration state to hardware thanks to `pinctrl_select_state()`.

The following is an example that shows how to grab a pin control group and apply its default configuration:

```
#include <linux/pinctrl/consumer.h>
int ret;
struct pinctrl_state *s;
struct pinctrl *p;
foo_probe()
{
    p = devm_pinctrl_get(dev);
    if (IS_ERR(p))
        return PTR_ERR(p);
    s = pinctrl_lookup_state(p, name);
    if (IS_ERR(s))
        return PTR_ERR(s);
    ret = pinctrl_select_state(p, s);
    if (ret < 0) // on error
        return ret;
[...]
}
```

Like other resources (such as memory regions, clocks, and so on), it is a good practice to grab pins and apply their configuration from within the `probe()` function. However, this operation is so common that it has been integrated into the Linux device core as a step while probing devices. Thus, when a device is being probed, the device core will do the following:

*   Grab the pins assigned to the device that is just about to probe using `devm_pinctrl_get()`.
*   Look for the default pin state (`PINCTRL_STATE_DEFAULT`) using `pinctrl_lookup_state()`.
*   Look in the meantime for an init (which means during the device initialization) pin state (`PINCTRL_STATE_INIT`) using the same API.
*   Apply the init pin state if any, otherwise apply the default pin state.
*   If power management is enabled, look for the optional sleep (`PINCTRL_STATE_SLEEP`) and idle (`PINCTRL_STATE_IDLE`) pin states for later use, during power management related operations.

See the `pinctrl_bind_pins()` function (defined in `drivers/base/pinctrl.c`), and the `really_probe()` function (defined in `drivers/base/dd.c`), which calls the former. These functions will help you understand how pins are bound to the device on its probing path.

Note

`pinctrl_select_state()` internally calls `pinmux_enable_setting()`, which in turn calls `pin_request()` on each pin in the pin control (group of pins) node.

The `pinctrl_put()` function can be used to release a pin control that has been requested using the non-managed API, that is, `pinctrl_get()`. That said, you can use `devm_pinctrl_get_select()`, given the name of the state to select, in order to configure pinmux in a single shot. This function is defined in `include/linux/pinctrl/consumer.h` as follows:

```
static struct pinctrl *devm_pinctrl_get_select(
                  struct device *dev, const char *name)
```

In the previous prototype, `name` is the name of the state as written in the `pinctrl-name` property. If the name of the state is `default`, the helper `devm_pinctr_get_select_default()` can be used, which is a wrapper around `devm_pinctl_get_select()` as follows:

```
static struct pinctrl * pinctrl_get_select_default(
                                      struct device *dev)
{
   return pinctrl_get_select(dev, PINCTRL_STATE_DEFAULT);
}
```

Now that we are familiar with the pin control subsystem (with both controller and consumer interfaces), we can learn how to deal with GPIO controllers, knowing that GPIO is an operating mode that a pin can work in.

# Dealing with the GPIO controller interface

The GPIO controller interface is designed around a single data structure, `struct gpio_chip`. This data structure provides a set of functions, among which are methods to establish GPIO direction (input and output), methods used to access GPIO values (get and set), methods to map a given GPIO to IRQ and return the associated Linux interrupt number, and the `debugfs` dump method (showing extra state like pull-up config). Apart from these functions, that data structure provides a flag to determine the nature of the controller, that is, to allow checking whether this controller's accessors may sleep or not. Still from within this data structure, the driver can set the GPIO base number, from which GPIO numbering should start.

Back to the code, a GPIO controller is represented as an instance of `struct gpio_chip`, defined in `<linux/gpio/driver.h>` as follows:

```
struct gpio_chip {
     const char       *label;
     struct gpio_device    *gpiodev;
     struct device         *parent;
     struct module         *owner;
     int        (*request)(struct gpio_chip *gc,
                           unsigned int offset);
     void       (*free)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*get_direction)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*direction_input)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*direction_output)(struct gpio_chip *gc,
                      unsigned int offset, int value);
     int       (*get)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*get_multiple)(struct gpio_chip *gc,
                           unsigned long *mask,
                           unsigned long *bits);
     void       (*set)(struct gpio_chip *gc,
                      unsigned int offset, int value);
     void       (*set_multiple)(struct gpio_chip *gc,
                           unsigned long *mask,
                           unsigned long *bits);
     int        (*set_config)(struct gpio_chip *gc,
                            unsigned int offset,
                            unsigned long config);
     int        (*to_irq)(struct gpio_chip *gc,
                           unsigned int offset);
     int       (*init_valid_mask)(struct gpio_chip *gc,
                              unsigned long *valid_mask,
                              unsigned int ngpios);
     int       (*add_pin_ranges)(struct gpio_chip *gc);
     int        base;
     u16        ngpio;
     const char *const *names;
     bool       can_sleep;
#if IS_ENABLED(CONFIG_GPIO_GENERIC)
     unsigned long (*read_reg)(void __iomem *reg);
     void (*write_reg)(void __iomem *reg, unsigned long data);
     bool be_bits;
     void __iomem *reg_dat;
     void __iomem *reg_set;
     void __iomem *reg_clr;
     void __iomem *reg_dir_out;
     void __iomem *reg_dir_in;
     bool bgpio_dir_unreadable;
     int bgpio_bits;
     spinlock_t bgpio_lock;
     unsigned long bgpio_data;
     unsigned long bgpio_dir;
#endif /* CONFIG_GPIO_GENERIC */
#ifdef CONFIG_GPIOLIB_IRQCHIP
     struct gpio_irq_chip irq;
#endif /* CONFIG_GPIOLIB_IRQCHIP */
     unsigned long *valid_mask;
#if defined(CONFIG_OF_GPIO)
     struct device_node *of_node;
     unsigned int of_gpio_n_cells;
     int (*of_xlate)(struct gpio_chip *gc,
                const struct of_phandle_args *gpiospec,
                u32 *flags);
#endif /* CONFIG_OF_GPIO */
};
```

The following are the meanings of each element in the structure:

*   `label`: This is the GPIO controller's functional name. It could be a part number or the name of the SoC IP block implementing it.
*   `gpiodev`: This is the internal state container of the GPIO controller. It is also the structure through which the character device associated with this GPIO controller will be exposed.
*   `request` is an optional hook for chip-specific activation. If provided, it is executed prior to allocating GPIO whenever you call `gpio_request()` or `gpiod_get()`.
*   `free` is an optional hook for chip-specific deactivation. If provided, it is executed before the GPIO is deallocated whenever you call `gpiod_put()` or `gpio_free()`.
*   `get_direction` is executed whenever you need to know the direction of the GPIO offset. The return value should be `0` to mean out, and `1` to mean in (the same as `GPIOF_DIR_XXX`) or a negative error.
*   `direction_input` configures the signal offset as input, or returns an error.
*   `get` returns the value of the GPIO offset; for output signals, this returns either the value actually sensed or zero.
*   `set` assigns an output value to the GPIO offset.
*   `set_multiple` is called when you need to assign output values for multiple signals defined by `mask`. If not provided, the kernel will install a generic hook that will walk through mask bits and execute `chip->set(i)` on each bit set. See here how you can implement this function:

    ```
     static void gpio_chip_set_multiple(
                        struct gpio_chip *chip,
                        unsigned long *mask,
                        unsigned long *bits)
    {
        if (chip->set_multiple) {
            chip->set_multiple(chip, mask, bits);
        } else {
            unsigned int i;
            /*
             * set outputs if the corresponding
             * mask bit is set
             */
            for_each_set_bit(i, mask, chip->ngpio)
                chip->set(chip, i, test_bit(i, bits));
            }
    }
    ```

*   `set_debounce` if supported by the controller, this hook is an optional callback provided to set the debounce time for the specified GPIO.
*   `to_irq` is an optional hook to provide GPIO to IRQ mapping. This is called whenever you want to execute the `gpio_to_irq()` or `gpiod_to_irq()` function. This implementation may not sleep.
*   `base` indicates the first GPIO number handled by this chip; or, if negative during registration, the kernel will automatically (dynamically) assign one.
*   `ngpio` is the number of GPIOs this controller provides, starting from base to (`base` + `ngpio - 1`).
*   `names`, if not `NULL`, must be a list (an array) of strings to use as alternative names for the GPIOs in this chip. The array must be `ngpio` sized, and any GPIO that does not need an alias may have its entry set to `NULL` in the array.
*   `can_sleep` is a Boolean flag to be set if `get()`/`set()` methods may sleep. It is the case for GPIO controllers (also known as GPIO expanders) sitting on buses such as I2C or SPI, whose accesses may lead to sleep. This means that if the chip supports IRQs, these IRQs must be threaded because the chip access may sleep when, for example, reading out the IRQ status registers. For a GPIO controller mapped to memory (part of SoC), this can be set to `false`.
*   Elements that are conditioned by the enabling of `CONFIG_GPIO_GENERIC` are related to a generic memory-mapped GPIO controller, with a standard register set.
*   `irq`: The IRQ chip of this GPIO controller if the controller can map GPIOs to IRQs. In such cases, this field must be set up before the GPIO chip is registered.
*   `valid_mask`: If not `NULL`, this element contains a bitmask of GPIOs that are valid to be used from the chip.
*   `of_node` is a pointer to the device tree node representing this GPIO controller.
*   `of_gpio_n_cells` is the number of cells used to form the GPIO specifier.
*   `of_xlate` is a callback to translate a device tree GPIO specifier into a chip relative GPIO number and flags. This hook is invoked when a GPIO line from this controller is specified in the device tree and must be parsed. If not provided, the GPIO core will set it to `of_gpio_simple_xlate()` by default, which is a generic GPIO core helper that supports two cell specifiers. The first cell identifies the GPIO number, and the second one, the GPIO flags. Additionally, when setting the default callback, `of_gpio_n_cells` will be set to `2`. If the GPIO controller needs one or more than two cell specifiers, you'll have to implement the corresponding translation callback.

Each GPIO controller exposes a number of signals that are identified in function calls by offset values in the range of `0` to (`ngpio - 1`). When those GPIO lines are referenced through calls such as `gpio_get_value(gpio)`, the offset is determined by subtracting `base` from the GPIO number and passed to the underlying driver function (`gpio_chip->get()` for example). The controller driver should then have the logic to map this offset to the control/status register associated with the GPIO.

## Writing a GPIO controller driver

A GPIO controller needs nothing but the set of callbacks that correspond to the features it supports. After every callback of interest has been defined and other fields set, the driver should call `devm_gpiochip_add_data()` on the configured `struct gpio_chip` structure in order to register the controller with the kernel. You have probably guessed that you'd better use the managed API (the `devm_` prefix) since it takes care of chip removal when necessary and releasing resources. If, however, you used the classical method, you will have to use `gpiochip_remove()` to remove the chip if necessary:

```
int gpiochip_add_data(struct gpio_chip *gc, void *data)
int devm_gpiochip_add_data(struct device *dev,
                        struct gpio_chip *gc, void *data)
```

In the preceding prototypes, `gc` is the chip to register and `data` is the driver's private data associated with this chip. They both return a negative error code if the chip can't be registered, such as because `gc->base` is invalid or already associated with a different chip. Otherwise, they return zero as a success code.

There are, however, pin controllers that are tightly coupled to the GPIO chip, and both are implemented in the same driver, much of which being in `drivers/pinctrl/pinctrl-*.c`. In such drivers, when `gpiochip_add_data()` is invoked, for device-tree-supported systems, the GPIO core will check the pin control's device node for the `gpio-ranges` property. If it is present, it will take care of adding the pin ranges for the driver.

However, the driver must call `gpiochip_add_pin_range()` in order to be compatible with older, existing device tree files that don't set the `gpio-ranges` attribute or systems that use ACPI. The following is an example:

```
if (!of_find_property(np, "gpio-ranges", NULL)) {
     ret = gpiochip_add_pin_range(chip,
                   dev_name(hw->dev), 0, 0, chip->ngpio);
     if (ret < 0) {
           gpiochip_remove(chip);
           return ret;
     }
}
```

Once more, it must be noted that the preceding is used just for backward compatibility for these old `pinctrl` nodes without the `gpio-ranges` property. Otherwise, calling `gpiochip_add_pin_range()` directly from a device tree-supported pin controller driver is deprecated. Please see *Section 2.1* of `Documentation/devicetree/bindings/gpio/gpio.txt` on how to bind pin controller and GPIO drivers via the `gpio-ranges` property.

We can see how easy it is to write a GPIO controller driver. In the book sources repository, you will find a working GPIO controller driver, for the MCP23016 I2C I/O expander from microchip, whose datasheet is available at [http://ww1.microchip.com/downloads/en/DeviceDoc/20090C.pdf](http://ww1.microchip.com/downloads/en/DeviceDoc/20090C.pdf).

To write such drivers, the following header should be included:

```
#include <linux/gpio.h> 
```

Following is an excerpt from the driver we have written for our controller:

```
#define GPIO_NUM 16
struct mcp23016 {
    struct i2c_client *client;
    struct gpio_chip gpiochip;
    struct mutex lock;
};
static int mcp23016_probe(struct i2c_client *client)
{
  struct mcp23016 *mcp;

  if (!i2c_check_functionality(client->adapter,
      I2C_FUNC_SMBUS_BYTE_DATA))
    return -EIO;
  mcp = devm_kzalloc(&client->dev, sizeof(*mcp),
                      GFP_KERNEL);
  if (!mcp)
    return -ENOMEM;
  mcp->gpiochip.label = client->name;
  mcp->gpiochip.base = -1;
  mcp->gpiochip.dev = &client->dev;
  mcp->gpiochip.owner = THIS_MODULE;
  mcp->gpiochip.ngpio = GPIO_NUM; /* 16 */
  /* may not be accessed from atomic context */
  mcp->gpiochip.can_sleep = 1; 
  mcp->gpiochip.get = mcp23016_get_value;
  mcp->gpiochip.set = mcp23016_set_value;
  mcp->gpiochip.direction_output =
                          mcp23016_direction_output;
  mcp->gpiochip.direction_input =
                          mcp23016_direction_input;
  mcp->client = client;
  i2c_set_clientdata(client, mcp);
  return devm_gpiochip_add_data(&client->dev,
                                &mcp->gpiochip, mcp);
}
```

In the preceding excerpt, the GPIO chip data structure has been set up before being passed to `devm_gpiochip_get_data()`, which is called to register the GPIO controller with the system. As a result, a GPIO character device node will appear under `/dev`.

### IRQ chip enabled GPIO controllers

IRQ chip support can be enabled in a GPIO controller by setting up the `struct gpio_irq_chip` structure embedded into this GPIO controller data structure. This `struct gpio_irq_chip` structure is used to group all fields related to interrupt handling in a GPIO chip and is defined as follows:

```
struct gpio_irq_chip {
     struct irq_chip *chip;
     struct irq_domain *domain;
     const struct irq_domain_ops *domain_ops;
     irq_flow_handler_t handler;
     unsigned int default_type;
     irq_flow_handler_t parent_handler;
     union {
          void *parent_handler_data;
          void **parent_handler_data_array;
     };
     unsigned int num_parents;
     unsigned int *parents;
     unsigned int *map;
     bool threaded;
     bool per_parent_data;
     int (*init_hw)(struct gpio_chip *gc);
     void (*init_valid_mask)(struct gpio_chip *gc,
                      unsigned long *valid_mask,
                      unsigned int ngpios);
     unsigned long *valid_mask;
     unsigned int first;
     void       (*irq_enable)(struct irq_data *data);
     void       (*irq_disable)(struct irq_data *data);
     void       (*irq_unmask)(struct irq_data *data);
     void       (*irq_mask)(struct irq_data *data);
};
```

There are architectures that may be multiple interrupt controllers involved in delivering an interrupt from the device to the target CPU. This feature is enabled in the kernel by setting the `CONFIG_IRQ_DOMAIN_HIERARCHY` config option.

In the previous data structure, some elements have been omitted. These are elements conditioned by `CONFIG_IRQ_DOMAIN_HIERARCHY`, that is, IRQ domain hierarchy related fields, which we won't discuss in this chapter. For the remaining elements, the following are their definitions:

*   `chip` is the IRQ chip implementation.
*   `domain` is the IRQ interrupt translation domain associated with `chip`; it is responsible for mapping between the GPIO hardware IRQ number and Linux IRQ number.
*   `domain_ops` represents the set of interrupt domain operations associated with the IRQ domain.
*   `handler` is the high-level interrupt flow handler (typically a predefined IRQ core function) for GPIO IRQs. There is a note on this field later in the section.
*   `default_type` is the default IRQ triggering type applied during GPIO driver initialization.
*   `parent_handler` is the interrupt handler for the GPIO chip's parent interrupts. It may be `NULL` if the parent interrupts are nested rather than chained. Moreover, setting this element to `NULL` will allow handling the parent IRQ in the driver. `gpio_chip.can_sleep` cannot be set to `true` if this handler is supplied because you cannot have chained interrupts on a chip that may sleep.
*   `parent_handler_data` and `parent_handler_data_array` are data associated with, and passed to, the handler for the parent interrupt. This can either be a single pointer if `per_parent_data` is `false`, or an array of `num_parents` pointers otherwise. If `per_parent_data` is `true`, `parent_handler_data_array` cannot be `NULL`.
*   `num_parents` is the number of interrupt parents for the GPIO chip.
*   `parents` is a list of interrupt parents of the GPIO chip. Because the driver owns this list, the core will only refer to it, not edit it.
*   `map` is a list of interrupt parents for each line of the GPIO chip.
*   `threaded` indicates whether the interrupt handling is threaded (uses nested threads).
*   `per_parent_data` tells whether `parent_handler_data_array` describes a `num_parents` sized array to be used as parent data.
*   `init_hw` is an optional routine for initializing hardware before an IRQ chip will be added. This is extremely beneficial when a driver has to clear IRQ-related registers in order to avoid unwanted events.
*   `init_valid_mask` is an optional callback that can be used to initialize `valid_mask`, which is used if not all GPIO lines are valid interrupts. There might be lines that just cannot fire interrupts, and this callback, when defined, is passed a bitmap in `valid_mask`, which will have `ngpios` bits from `0..(ngpios-1)` set to `1` as valid. The callback can then directly set some bits to `0` if they cannot be used for interrupts.
*   `valid_mask`, if not `NULL`, contains a bitmask of GPIOs that are valid for inclusion in the IRQ domain of the chip.
*   `first` is necessary in the case of static IRQ allocation. If set, `irq_domain_add_simple()` will allocate (starting from this value) and map all IRQs during initialization.
*   `irq_enable`, `irq_disable`, `irq_unmask`, and `irq_mask` respectively store old `irq_chip.irq_enable`, `irq_chip.irq_disable`, `irq_chip.irq_unmask`, and `irq_chip.irq_mask` callbacks. See [*Chapter 13*](B17934_13_Epub.xhtml#_idTextAnchor194), *Demystifying the Kernel IRQ Framework*, for a detailed explanation.

Under the premise that your interrupts are 1-to-1 mapped to the GPIO line index, `gpiolib` will handle a significant portion of overhead code. In this 1-to-1 mapping, GPIO line offset 0 maps to hardware IRQ 0, GPIO line offset `1` maps to hardware IRQ `1`, and so on until GPIO line offset `ngpio-1`, which maps to hardware IRQ `ngpio-1`. The bitmask `valid_mask` and the flag `need_valid_mask` in `gpio_irq_chip` can be used to mask off some lines as invalid for associating with IRQs, provided some GPIO lines do not have corresponding IRQs.

We can divide GPIO IRQ chips into two broad categories:

*   **Cascaded interrupt chips**: This indicates that the GPIO chip has a single common interrupt output line that is triggered by any enabled GPIO line on that chip. The interrupt output line is subsequently routed to a parent interrupt controller one level up, which in the simplest case is the system's primary interrupt controller. An IRQ chip implements this by inspecting bits inside the GPIO controller to determine which line fired it. To figure this out, the IRQ chip part of the driver will need to inspect registers, and it will almost certainly need to acknowledge that it is handling the interrupt by clearing some bits (sometimes implicitly, by simply reading a status register), as well as setting up configurations such as edge sensitivity (rising or falling edge or a high/low level interrupt, for example).
*   **Hierarchical interrupt chips**: This means that each GPIO line is connected to a parent interrupt controller one level up by a dedicated IRQ line. It is not necessary to query the GPIO hardware to determine which line has fired, but you might need to acknowledge the interrupt and configure edge sensitivity.

Cascaded GPIO IRQ chips usually fall in one of three categories:

*   **Chained cascaded GPIO IRQ chips**: These are the types that are usually seen on SoCs. This means that the GPIOs have a fast IRQ flow handler that is called in a chain from the parent IRQ handler, which is usually the system interrupt controller. This means that the parent IRQ chip will immediately call the GPIO IRQ chip handler while holding the IRQs disabled. In its interrupt handler, the GPIO IRQ chip will then call something similar to this:

    ```
      static irqreturn_t foo_gpio_irq(int irq, void *data)
          chained_irq_enter(...);
          generic_handle_irq(...);
          chained_irq_exit(...);
    ```

Because everything happens directly in the callback, chained GPIO IRQ chips cannot set the `.can_sleep` flag on `struct gpio_chip` to `true`. In this case, no slow bus traffic like I2C can be used.

*   `CHAINED GPIO IRQCHIPS`, but they don't use chained IRQ handlers. GPIO IRQs are instead dispatched via a generic IRQ handler, which is specified using `request_irq()`. In this IRQ handler, the GPIO IRQ chip will then end up calling something similar to the following sequence:

    ```
    static irqreturn_t gpio_rcar_irq_handler(int irq,
                                        void *dev_id)
        /* go through the entire GPIOs and handle
         * all interrupts
         */
        for each detected GPIO IRQ
            generic_handle_irq(...);
    ```

*   **Nested thread GPIO IRQ chips**: Off-chip GPIO expanders and any other GPIO IRQ chip sitting on a sleeping bus, such as I2C or SPI, fall under this category.

Of course, such drivers who require sluggish bus traffic to read out IRQ status and other information, traffic which may result in further IRQs, cannot be accommodated in a rapid IRQ handler with IRQs disabled. Instead, they must create a thread and then mask the parent IRQ line until the interrupt is handled by the driver. This driver's distinguishing feature is that it calls something like the following in its interrupt handler:

```
static irqreturn_t pcf857x_irq(int irq,
                               void *data)
{
     struct pcf857x *gpio = data;
     unsigned long change, i, status;
     status = gpio->read(gpio->client);
     mutex_lock(&gpio->lock);
     change = (gpio->status ^ status) &
              gpio->irq_enabled;
     gpio->status = status;
     mutex_unlock(&gpio->lock);
     for_each_set_bit(i, &change, gpio->chip.ngpio)
      child_irq = irq_find_mapping(
                     gpio->chip.irq.domain, i);
        handle_nested_irq(child_irq);
     return IRQ_HANDLED;
}
```

Threaded GPIO IRQ chips are distinguished by the fact that they set the `.can_sleep` flag on `struct gpio_chip` to `true`, indicating that the chip can sleep when accessing the GPIOs.

Note

It is worth recalling that `gpio_irq_chip.handler` is the interrupt flow handler. It is the high-level IRQ-events handler, the one that calls the underlying handlers registered by client drivers using `request_irq()` or `request_threaded_irq()`. Its value depends on the IRQ being edge- or level-triggered. It is most often a predefined IRQ core function, one between `handle_simple_irq`, `handle_edge_irq`, and `handle_level_irq`. These are all kernel helper functions that do some operations before and after calling the real IRQ handler.

When the parent IRQ handler calls `generic_handle_irq()` or `handle_nested_irq()`, the IRQ core will look for the IRQ descriptor structure (Linux's view of an interrupt) corresponding to the Linux IRQ number passed as an argument (`struct irq_desc *desc = irq_to_desc(irq)`) and calling `generic_handle_irq_desc()` on this descriptor, which will result in `desc->handle_irq(desc)`. You should note that `desc->handle_irq` corresponds to the high-level IRQ handler supplied earlier, which has been assigned to the IRQ descriptor using `irq_set_chip_and_handler()` during the mapping of this IRQ. Guess what, the mapping of these GPIO IRQs is done in `gpiochip_irq_map`, which is the `.map` callback of the default IRQ domain operation table (`gpiochip_domain_ops`) assigned by the GPIO core to the GPIO IRQ chip if not provided by the driver.

To summarize, `desc->handle_irq = gpio_irq_chip.handler`, which may be `handle_level_irq`, `handle_simple_irq`, `handle_edge_irq`, or (rarely) a driver-provided function.

#### Example of adding IRQ chip support in a GPIO chip

In this section, we will demonstrate how to add the support of an IRQ chip into a GPIO controller driver. To do that, we will update our initial driver, more precisely, the probe method, as well as implementing an interrupt handler, which will hold the IRQ handling logic.

Let's consider the following figure:

![Figure 16.1 – Multiplexing IRQs
](img/B17934_16_001.jpg)

Figure 16.1 – Multiplexing IRQs

In the previous figure, let's consider that we have configured `io_0` and `io_1` as interrupt lines (this is what `DeviceA` and `deviceB` see).

Whether an interrupt happens on `io_0` or `io_1`, the same parent interrupt line will be raised on the GPIO chip. At this step, the GPIO chip driver must figure out, by reading the GPIO status register of the GPIO controller, which GPIO line (`io_0` or `io_1`) has really fired the interrupt. This is how, in the case of the MCP23016 chip, a single interrupt line (the parent actually) can be a multiplex for 16 GPIO interrupts.

Now let's update the initial GPIO controller driver. It must be noted that because the device sits on a slow bus, we have no choice but to implement nested (threaded) interrupt flow handling.

We start by defining our IRQ chip data structure, with a set of callbacks that can be used by the IRQ core. The following is an excerpt:

```
static struct irq_chip mcp23016_irq_chip = {
     .name = "gpio-mcp23016",
     .irq_mask = mcp23016_irq_mask,
     .irq_unmask = mcp23016_irq_unmask,
     .irq_set_type = mcp23016_irq_set_type,
};
```

In the preceding, the callbacks that have been defined depend on the need. In our case, we have only implemented interrupt (un)masking related callbacks, as well as the callback allowing you to set the IRQ type. To see the full description of a `struct irq_chip` structure, you can refer to [*Chapter 13*](B17934_13_Epub.xhtml#_idTextAnchor194), *Demystifying the Kernel IRQ Framework*.

Now that the IRQ chip data structure has been set up, we can modify the probe method as follows:

```
static int mcp23016_probe(struct i2c_client *client)
{
    struct gpio_irq_chip *girq;
    struct irq_chip *irqc;
[...]
    girq = &mcp->gpiochip.irq;
    girq->chip = &mcp23016_irq_chip;
    /* This will let us handling the parent IRQ in the driver */
    girq->parent_handler = NULL;
    girq->num_parents = 0;
    girq->parents = NULL;
    girq->default_type = IRQ_TYPE_NONE;
    girq->handler = handle_level_irq;
    girq->threaded = true;
[...]
    /*
     * Directly request the irq here instead of passing
     * a flow-handler.
     */
    err = devm_request_threaded_irq(
                      &client->dev,
                      client->irq,
                      NULL, mcp23016_irq,
                      IRQF_TRIGGER_RISING | IRQF_ONESHOT,
                      dev_name(&i2c->dev), mcp);
[...]

    return devm_gpiochip_add_data(&client->dev,
                                &mcp->gpiochip, mcp);
}
```

In the previous probe method update, we first initialized the `struct gpio_irq_chip` data structure embedded into `struct gpio_chip`, and then we registered an IRQ handler, which will act as the parent IRQ handler, responsible for enquiring the underlying GPIO chip for any IRQ-enabled GPIOs that have changed, and then running their IRQ handlers, if any.

Finally, the following is our IRQ handler, which must have been implemented before the `probe` function:

```
static irqreturn_t mcp23016_irq(int irq, void *data)
{
    struct mcp23016 *mcp = data;
    unsigned long status, changes, child_irq, i;
    status = read_gpio_status(mcp);
    mutex_lock(&mcp->lock);
    change = mcp->status ^ status;
    mcp->status = status;
    mutex_unlock(&mcp->lock);
    /* Do some stuff, may be adapting "change" according to level */
    [...]
    for_each_set_bit(i, &change, mcp->gpiochip.ngpio) {
        child_irq =
           irq_find_mapping(mcp->gpiochip.irq.domain, i);
        handle_nested_irq(child_irq);
    }
    return IRQ_HANDLED;
}
```

In our interrupt handler, we simply read the current GPIO, and we compare it with the old status to determine GPIOs that have changes. All the tricks are handled by `handle_nested_irq()`, which is explained in [*Chapter 13*](B17934_13_Epub.xhtml#_idTextAnchor194), *Demystifying the Kernel IRQ Framework*, as well.

Now that we are done and are familiar with the implementation of IRQ chips in GPIO controller drivers, we can learn about the binding of these GPIO controllers, which will allow declaring the GPIO chip hardware in the device tree in a way the driver understands.

## GPIO controller bindings

The device tree is the de facto standard to declare and describe devices on embedded platforms, especially on ARM architectures. It this then recommended for new drivers to provide the associated device bindings.

Back to GPIO controllers, there are mandatory properties that need to be provided:

*   `compatible`: This is the list of strings to match the driver(s) that will handle this device.
*   `gpio-controller`: This is a property that indicates to the device tree core that this node represents a GPIO controller.
*   `gpio-cells`: Tells how many cells are used to describe a GPIO specifier. It should correspond to `gpio_chip.of_gpio_n_cells`, or to a value that `gpio_chip.of_xlate` can deal with. It is typically `<2>` for a less complex controller, with the first cell identifying the GPIO number, and the second defining the flags.

There are additional mandatory properties to define if the GPIO controller has IRQ chip support, that is, if this controller allows mapping its GPIO lines to IRQs. With such GPIO controllers, the following mandatory properties must be provided:

*   `interrupt-controller`: Some controllers provide IRQs mapped to the GPIOs. In that case, the property `#interrupt-cells` should be set too and usually you use `2`, but it depends on your needs. The first cell is the pin number, and the second represents the interrupt flags.
*   `#interrupt-cells`: This must be defined as a value supported by the `xlate` hook of the IRQ domain, that is, `irq_domain_ops.xlate`. It is common for this hook to be set with `irq_domain_xlate_twocell`, which is a generic kernel IRQ core helper able to handle a two-cell specifier.

From the properties listed, we can declare our GPIO controller under its bus node, as follows:

```
&i2c1
    expander: mcp23016@20 {
        compatible = "microchip,mcp23016";
        reg = <0x20>;
        gpio-controller;
        #gpio-cells = <2>;
        interrupt-controller;
        #interrupt-cells = <2>;
        interrupt-parent = <&gpio4>;
        interrupts = <29 IRQ_TYPE_EDGE_FALLING>;
    };
};
```

This is all for the controller side. In order to demonstrate how clients can consume resources provided by the MCP23016, let's consider the following scenario: we have two devices, device A, named `foo`, and device B, named `bar`. The `bar` device consumes two GPIO lines from our controller (they will be used in output mode), and the `foo` device would like to map a GPIO to IRQ. This configuration could be declared in the device tree as follows:

```
parent_node {
    compatible = "simple-bus";
    foo_device: foo_device@1c {
        [...]
        reg = <0x1c>;
        interrupt-parent = <&expander>;
        interrupts = <2 IRQ_TYPE_EDGE_RISING>;
    };
    bar_device {
        [...]
        reset-gpios = <&expander 8 GPIO_ACTIVE_HIGH>;
        power-gpios = <&expander 12 GPIO_ACTIVE_HIGH>;
        [...]
    };
};
```

In the preceding excerpt, `parent_node` is given `simple-bus` as a `compatible` string in order to instruct the device tree core and the platform core to instantiate two platform devices, which correspond to our nodes. In that excerpt, we have also demonstrated how GPIOs, as well as IRQs from our controller, are specified. The number of cells used for each property matches the declaration in the controller binding.

### GPIO- and pin-controller interaction

These two subsystems are closely related. A pin controller can route some or all of the GPIOs provided by a GPIO controller to pins on the package. This allows those pins to be muxed (also known as pinmuxing) between GPIO and other functions.

It may then be useful to know which GPIOs correspond to which pins on which pin controllers. The `gpio-ranges` property, which will be described below, represents this correspondence with a discrete set of ranges that map pins from the pin controller local number space to pins in the GPIO controller local number space.

The `gpio-ranges` format is the following:

```
<[pin controller phandle], [GPIO controller offset], [pin controller offset], [number of pins]>;
```

The GPIO controller offset refers to the GPIO controller node containing the range definition. The bindings defined in `Documentation/pinctrl/pinctrl-bindings.txt` must be followed by the pin controller node referenced by `phandle`.

Each offset is a number between 0 and N. It is possible to stack any number of ranges with just one pin-to-GPIO line mapping if the ranges are concocted, but in practice, these ranges are generally gathered together as discrete sets.

The following is an example:

```
gpio-ranges = <&foo 0 20 10>, <&bar 10 50 20>;
```

This means the following:

*   Ten pins (`20..29`) on pin controller `foo` are mapped to GPIO lines `0..9`.
*   Twenty pins ( `50..69` ) on pin controller `bar` are mapped to GPIO lines `10..29`.

It must be noted that GPIOs have a global number space and the pin controller has a local number space, so we need to define a way to cross-reference them.

We want to map PIN `GPIO_5_29` with PIN number `89` in the pin controller number space. The following is the device tree property to define the mapping between the GPIO and pin control subsystem:

```
&gpio5 {
    gpio-ranges = <&pinctrl 29 89 1> ;
}
```

In the previous excerpt, 1 GPIO line from the 29th GPIO line of GPIO bank5 will be mapped to pin ranges from 89 on pin controller `pinctrl`.

To illustrate this on a real platform, let's consider the i.MX6 SoC, which has 32 GPIOs per bank. The following is an excerpt from the pin controller node of i.MX6 SoCs, declared in `arch/arm/boot/dts/imx6qdl.dtsi`, and whose driver is `drivers/pinctrl/freescale/pinctrl-imx6dl.c`:

```
iomuxc: pinctrl@20e0000 {
    compatible = "fsl,imx6dl-iomuxc", "fsl,imx6q-iomuxc";
    reg = <0x20e0000 0x4000>;
};
```

Now that the pin controller has been declared, a GPIO controller (bank3) is declared in the same base device tree, `arch/arm/boot/dts/imx6qdl.dtsi`, as follows:

```
gpio3: gpio@20a4000 {
    compatible = "fsl,imx6q-gpio", "fsl,imx35-gpio";
    reg = <0x020a4000 0x4000>;
    interrupts = <0 70 IRQ_TYPE_LEVEL_HIGH>,
                 <0 71 IRQ_TYPE_LEVEL_HIGH>;
    gpio-controller;
    #gpio-cells = <2>;
    interrupt-controller;
    #interrupt-cells = <2>;
};
```

For information, the driver of this GPIO controller is `drivers/gpio/gpio-mxc.c`. After the GPIO controller node has been declared in the base device tree, this same GPIO controller node is overridden in a SoC-specific base device tree, `arch/arm/boot/dts/imx6q.dtsi`, as follows:

```
&gpio3 {
     gpio-ranges = <&iomuxc 0 69 16>, <&iomuxc 16 36 8>,
                  <&iomuxc 24 45 8>;
};
```

The preceding override of the GPIO controller node means the following:

*   `<&iomuxc 0 69 16>` means that 16 pins, from pin 69 (to 84) on pin controller `iomuxc`, are mapped to GPIO lines starting from index 0 (to 15).
*   `<&iomuxc 16 36 8>` means that 8 pins, from pin 36 (to 43) on pin controller `iomuxc`, are mapped to GPIO lines starting from index 16 (to 23).
*   `<&iomuxc 24 45 8>` means that 8 pins, from pin 45 (to 52) on pin controller `iomuxc`, are mapped to GPIO lines starting from index 24 (to 31).

As expected, we have a 32-line GPIO bank, `16 + 8 + 8`.

As of now, we are able to both understand existing and instantiate new GPIO controllers from the device tree that interact with one or more pin controllers. As the last step in this GPIO controller binding learning curve, let's learn how to hog GPIOs in order to avoid writing a particular driver (or prevent existing ones) to control them.

### Pin hogging

The GPIO chip can contain GPIO hog definitions. GPIO hogging is a mechanism providing automatic GPIO requests and configuration as part of the GPIO controller's driver probe function. This means that as soon as the pin control device is registered, the GPIO core will attempt to call `devm_pinctrl_get()`, `lookup_state()`, and `select_state()` on it.

The following are properties required for each GPIO hog definition, which is represented as a child node of the GPIO controller:

*   `gpio-hog`: A property that indicates whether or not this child node represents a GPIO hog.
*   `gpios`: Contains the GPIO information (ID, flags, ...) for each GPIO to affect. Will contain an integer multiple of the number of cells specified in its parent node (GPIO controller node).
*   Only one of the following properties can be specified, scanned in the order they are listed. This means that when multiple properties are present, they will be searched in the order they are presented here, with the first match being considered as the intended configuration:
    *   `input`: A property that specifies that the GPIO direction should be set to input.
    *   `output-low`: A property that specifies that the GPIO should be configured as output, with an initial low state value.
    *   `output-high`: A property that specifies that the GPIO direction should be set to output with an initial high value.

An optional property is `line-name`: the GPIO label name. If it's not present, the node name is used.

The following is an excerpt, where we first declare (as GPIO) the pins we are interested in in the pin controller node:

```
&iomuxc {
[...]
    pinctrl_gpio3_hog: gpio3hoggrp {
        fsl,pins = <
            MX6QDL_PAD_EIM_D19__GPIO3_IO19      0x1b0b0
            MX6QDL_PAD_EIM_D20__GPIO3_IO20      0x1b0b0
            MX6QDL_PAD_EIM_D22__GPIO3_IO22      0x1b0b0
            MX6QDL_PAD_EIM_D23__GPIO3_IO23      0x1b0b0
        >;
    };
[...]
}
```

After the pins of interest are declared, we can hog each GPIO under the node of the GPIO controller this GPIO belongs to, as follows:

```
&gpio3 {
    pinctrl-names = "default";
    pinctrl-0 = <&pinctrl_gpio3_hog>;
    usb-emulation-hog {
        gpio-hog;
        gpios = <19 GPIO_ACTIVE_HIGH>;
        output-low;
        line-name = "usb-emulation";
    };
    usb-mode1-hog {
        gpio-hog;
        gpios = <20 GPIO_ACTIVE_HIGH>;
        output-high;
        line-name = "usb-mode1";
    };
    usb-pwr-hog {
        gpio-hog;
        gpios = <22 GPIO_ACTIVE_LOW>;
        output-high;
       line-name = "usb-pwr-ctrl-en-n";
    };
    usb-mode2-hog {
        gpio-hog;
        gpios = <23 GPIO_ACTIVE_HIGH>;
        output-high;
        line-name = "usb-mode2";
    };
};
```

It must be noted that hogging pins should be used for those pins that are not controlled by any particular driver.

GPIO hogging was the last part on the GPIO controller side. Now that controllers have no more secrets for us, let's switch to the consumer interface.

# Getting the most out of the GPIO consumer interface

The GPIO is a feature, or a mode in which a pin can operate, in terms of hardware. It is nothing more than a digital line that may be used as an input or output and has just two values (or states): 1 for high or 0 for low. The kernel's GPIO subsystem includes all of the functions you'll need to set up and manage GPIO lines from within your driver.

Before using a GPIO from within the driver, it must first be claimed by the kernel. It's a means to take control of a GPIO and prohibit other drivers from using it, as well as preventing the controller driver from being unloaded.

After claiming control of the GPIO, you can do the following:

*   Set the direction and, if needed, set the GPIO configuration.
*   If it's being used as an output, start toggling its output state (driving the line high or low).
*   If used as input, set the debounce-interval if needed and read the state. For a GPIO line mapped to an IRQ, configure at what edge/level the interrupt should be triggered, and register a handler that will be run when the interrupt occurs.

In the Linux kernel, there are two different ways to deal with GPIOs:

*   The legacy and deprecated integer-based interface, which uses integers to represent GPIOs.
*   The new descriptor-based interface, where a GPIO is represented and described by an opaque structure, with a dedicated API. This is the recommended way to go.

While we will discuss the two approaches in this chapter, let's start with the legacy interface.

## Integer-based GPIO interface – now deprecated

The integer-based interface is the most known usage of GPIOs in Linux systems, either in the kernel or in the user space. In this mode, the GPIO is identified by an integer, which is used for every operation that needs to be performed on this GPIO. The following is the header that contains the legacy GPIO access function:

```
#include <linux/gpio.h>
```

The integer-based interface relies on a set of functions, defined as follows:

```
bool gpio_is_valid(int number);
int  gpio_request(unsigned gpio, const char *label);
int  gpio_get_value_cansleep(unsigned gpio);
int  gpio_direction_input(unsigned gpio);
int  gpio_direction_output(unsigned gpio, int value);
void gpio_set_value(unsigned int gpio, int value);
int  gpio_get_value(unsigned gpio);
void gpio_set_value_cansleep(unsigned gpio, int value);
int gpio_get_value_cansleep(unsigned gpio);
void gpio_free(unsigned int gpio);
```

All the preceding functions are mapped to a set of callbacks provided by the GPIO controller through its `struct gpio_chip` structure, thanks to which it exposes a generic set of callback functions.

In all these functions, `gpio` represents the GPIO number we are interested in. Before using a GPIO, client drivers must call `gpio_request()` in order to take ownership of the line and, very importantly, to prevent this GPIO controller driver from being unloaded. In the same function, `label` is the label used by the kernel for labeling/describing the GPIO in sysfs as we can see in `/sys/kernel/debug/gpio`. `gpio_request``()` returns `0` on success, and a negative error code on error. If in doubt, before requesting the GPIO, you can use the `gpio_is_valid()` function to check whether the specified GPIO number is valid on the system prior to it being requested.

Once a driver owes the GPIO, it can change its direction, depending on the need, whether it should be an input or output, using the `gpio_direction_input()` or `gpio_direction_output()` functions. In these functions, `gpio` is the GPIO number the driver needs to set the direction, which should have already been requested. There is a second parameter when it comes to configuring the GPIO as output, `value`, which is the initial state the GPIO should be in once the output direction is effective. Here again, in both functions, the return value is `0` on success or a negative error code on failure. Internally, these functions are mapped to lower-level callback functions exported by the driver of the GPIO chip that provides the GPIO line `gpio`.

Some GPIO controllers allow you to adjust the GPIO debounce-interval (this is only useful when the GPIO line is configured as input). This parameter can be set using `gpio_set_debounce()`. In this function, the `debounce` argument is the debounce time in milliseconds.

As it is a good practice to grab and configure resources in the driver's probing method, GPIO lines must respect this rule.

It has to be noticed that GPIO management (either configuration or getting/setting values) is not context agnostic; that is, there are memory-mapped GPIO controllers that can be accessed from any context (process and atomic contexts). On the other hand, there are GPIOs provided by discrete chips sitting on slow buses (such as I2C or SPI) that can sleep (because sending/receiving commands on such buses requires waiting to get to the head of a queue to transmit a command and get its response). Such GPIOs must be manipulated from a process context exclusively. A well-designed controller driver must be able to inform clients whether calls to its GPIO driving methods may sleep or not. This can be checked with the `gpio_cansleep()` function. This function returns `true` for GPIO lines whose controller sits on a slow bus, and `false` for GPIOs that belong to a memory-mapped controller.

Now that the GPIOs are requested and configured, we can set/get their values using the appropriate APIs. Here again, the APIs to use are context-dependent. For memory-mapped GPIO controllers, their GPIO lines can be accessed using `gpio_get_value()` or `gpio_set_value()`. The first function returns a value that represents the GPIO state, and the second one will set the value of the GPIO, which should have been configured as an output using `gpio_direction_output()`. `value` can be considered as Boolean for both functions, with zero indicating a low level and a non-zero value indicating a high level.

In case of doubt about the kind of GPIO controller from where the GPIO originates, the driver should use the context agnostic APIs, `gpio_get_value_cansleep()` and `gpio_set_value_cansleep()`. These APIs are safe to use in threaded contexts but also work in an atomic context.

Note

The legacy (that is, integer-based) interface supports specifying GPIOs from the device tree, in which case the APIs to be used to grab those GPIOs will be `of_get_gpio()`, `of_get_named_gpio()`, or similar APIs. These are mentioned here for studying purposes and won't be discussed in this chapter.

### GPIO mapped to IRQ

There are GPIO controllers that allow their GPIO lines to be mapped to IRQs. These IRQs can be either edge- or level-triggered. The configuration depends on the needs. The GPIO controller is responsible for providing the mapping between the GPIO and its IRQ.

If the IRQ has been specified in the device tree and the underlying device is an I2C or SPI device, the consumer driver must just request the IRQ normally, since upon the device tree parsing, the GPIO mapped to IRQ specified in the device tree will be translated by the device tree core and assigned to your device structure, that is, `i2c_client.irq` or `spi_device.irq`. This is the case in `foo_device` from the example we have seen in the *GPIO controller bindings* section. For another device type, you'll have to call `irq_of_parse_and_map()` or a similar API.

If, however, the driver is given a GPIO (from module parameters, for example) or specified from the device tree without being mapped to IRQ there, the driver must use `gpio_to_irq()` to map the given GPIO number to its IRQ number:

```
int gpio_to_irq(unsigned gpio)
```

This function returns the corresponding Linux IRQ number, which can be passed to `request_irq()` (or the threaded counterpart, `request_threaded_irq()`) to register a handler for this IRQ:

```
int request_threaded_irq (unsigned int irq,
                   irq_handler_t handler,
                   irq_handler_t thread_fn,
                   unsigned long irqflags,
                   const char *devname,
                   void *dev_id); 
int request_any_context_irq (unsigned int irq,
                       irq_handler_t handler,
                       unsigned long flags,
                       const char * name,
                       void * dev_id);
```

`request_any_context_irq()` is smart enough to identify the underlying context supported by the IRQ chip integrated into the GPIO controller. If this controller's accessors can sleep, `request_any_context_irq()` will request a threaded IRQ, otherwise, it will request an atomic-context IRQ.

The following is a short example demonstrating what we have discussed so far:

```
static irqreturn_t my_interrupt_handler(int irq,
                                        void *dev_id)
{
    [...]
    return IRQ_HANDLED;
}
static int foo_probe(struct i2c_client *client)
{
    [...]
    struct device_node *np = client->dev.of_node;
    int gpio_int = of_get_gpio(np, 0);
    int irq_num = gpio_to_irq(gpio_int);
    int error =
        devm_request_threaded_irq(&client->dev, irq_num,
               NULL, my_interrupt_handler,
               IRQF_TRIGGER_RISING | IRQF_ONESHOT,
               input_dev->name, my_data_struct);
    if (error) {
        dev_err(&client->dev, "irq %d requested failed,
                 %d\n", client->irq, error);
        return error;
    }
    [...]
    return 0;
}
```

In the previous excerpt, we have demonstrated how to use a legacy integer-based interface to grab a GPIO specified in the device tree, as well as the old API to translate this GPIO into a valid Linux IRQ number. These were the main points to highlight.

Though deprecated, we briefly introduced the legacy GPIO APIs. As is recommended, in the next section, we will deal with the new descriptor-based GPIO interface.

## Descriptor-based GPIO interface: the new and recommended way

With the new descriptor-based GPIO interface, the subsystem has been oriented to the producer/consumer. The header required for the descriptor-based GPIO interface is the following:

```
#include <linux/gpio/consumer.h>
```

With the descriptor-based interface, a GPIO is described and characterized by a coherent data structure, `struct gpio_desc`, which is defined as follows:

```
struct gpio_desc {
     struct gpio_chip *chip;
     unsigned long    flags;
     const char       *label;
};
```

In the preceding data structure, `chip` is the controller providing this GPIO line; `flags` are the flags characterizing the GPIO and `label` is the name describing the GPIO.

Prior to requesting and acquiring ownership of GPIOs with the descriptor-based interface, these GPIOs must have been specified or mapped somewhere. It means they should be allocated to a driver, whereas with the legacy integer-based interface, a driver could just obtain a number from anywhere and request it as GPIO. Since descriptor-based GPIOs are represented by an opaque structure, such a method is not possible anymore.

With the new interface, GPIOs must exclusively be mapped to names or indexes, specifying at the same time the GPIO chips providing the GPIOs of interest. This gives us three ways to specify and assign GPIOs to drivers:

*   **Platform data mapping**: In such cases, for example, the mapping is done in the board file.
*   **Device tree**: The mapping is done in the device tree. This is the mapping we will discuss in this book.
*   **Advanced Configuration and Power Interface mapping (ACPI)**: This is ACPI-style mapping. On x86-based systems, this is the most common configuration.

Now that we are done with the GPIO descriptor interface introduction, let's learn how it is mapped and assigned to devices.

### GPIO descriptor mapping in the device tree and its APIs

GPIO descriptor mappings are defined in the device tree node of the consumer device. The GPIO descriptor mapping property must be named `<name>-gpios` or `<name>-gpio`, where `<name>` is meaningful enough to describe the function for which the GPIO(s) will be used. This is mandatory.

The reason is that descriptor-based GPIO lookup relies on the `gpio_suffixes[]` variable, a `gpiolib` variable defined in `drivers/gpio/gpiolib.h` as follows:

```
static const char * const gpio_suffixes[] =
                          { "gpios", "gpio" };
```

This variable is used in both device tree lookup and ACPI-based lookup. To see how it works, let's see how it is used in `of_find_gpio()`, the device tree's low-level GPIO lookup function defined as follows:

```
static struct gpio_desc *of_find_gpio(
                           struct device *dev,
                           const char *con_id,
                           unsigned int idx,
                           enum gpio_lookup_flags *flags)
{
    /* 32 is max size of property name */
    char prop_name[32];
    enum of_gpio_flags of_flags;
    struct gpio_desc *desc;
    unsigned int i;
    /* Try GPIO property "foo-gpios" and "foo-gpio" */
    for (i = 0; i < ARRAY_SIZE(gpio_suffixes); i++) {
        if (con_id)
            snprintf(prop_name, sizeof(prop_name),
                      "%s-%s", con_id,
                      gpio_suffixes[i]);
        else
            snprintf(prop_name, sizeof(prop_name), "%s",
                       gpio_suffixes[i]);
        desc = of_get_named_gpiod_flags(dev->of_node,
                                      prop_name, idx,
                                      &of_flags);
        if (!IS_ERR(desc) || PTR_ERR(desc) != -ENOENT)
            break;
[...]
}
```

Now let's consider the following node, which is an excerpt of `Documentation/gpio/board.txt`:

```
foo_device {
    compatible = "acme,foo";
    [...]
    led-gpios = <&gpio 15 GPIO_ACTIVE_HIGH>, /* red */
                <&gpio 16 GPIO_ACTIVE_HIGH>, /* green */
                <&gpio 17 GPIO_ACTIVE_HIGH>; /* blue */
    power-gpio = <&gpio 1 GPIO_ACTIVE_LOW>;
    reset-gpio = <&gpio 1 GPIO_ACTIVE_LOW>;
};
```

This is what a mapping should look like, with meaningful names, corresponding to the functions assigned to the GPIOs. This excerpt will be used as the basis for the rest of this section to demonstrate the use of the descriptor-based GPIO interface.

Now that the GPIOs have been specified in the device tree, the first thing to be done is to allocate GPIO descriptors and take the ownership of these GPIOs. This can be done using `gpiod_get()`, `gpiod_getindex()`, or `gpiod_get_optional()`, defined as follows:

```
struct gpio_desc *gpiod_get_index(struct device *dev,
                                 const char *con_id,
                                 unsigned int idx,
                                 enum gpiod_flags flags)
struct gpio_desc *gpiod_get(struct device *dev,
                            const char *con_id,
                            enum gpiod_flags flags)
struct gpio_desc *gpiod_get_optional(struct device *dev,
                                     const char *con_id,
                                     enum gpiod_flags flags);
```

It must be noted that we can also use the device-managed variant of these APIs, defined as follows:

```
struct gpio_desc *devm_gpiod_get_index(
                                struct device *dev,
                                const char *con_id,
                                unsigned int idx,
                                enum gpiod_flags flags);
struct gpio_desc *devm_gpiod_get(struct device *dev,
                               const char *con_id,
                               enum gpiod_flags flags);
struct gpio_desc *devm_gpiod_get_optional(
                                struct device *dev,
                                const char *con_id,
                                enum gpiod_flags flags);
```

Both non `_optional` functions will return `-ENOENT` if no GPIO with the given function is assigned or a negative error if another error occurred. On success, the GPIO descriptor corresponding to the GPIO is returned. The first method returns the GPIO descriptor structure for the GPIO at a particular index (useful when the specifier is a list of GPIOs), whereas the second function always returns the GPIO at index `0` (single GPIO mapping). The `_optional` variant is useful for drivers that need to deal with optional GPIOs; it's the same as `gpiod_get()`, except that it returns `NULL` when no GPIO has been assigned to the device (that is, specified in the device tree).

In parameters, `dev` is the device to which the GPIO descriptor will belong. It is the underlying `device` structure the driver is responsible for; for example, `i2c_client.dev`, `spi_device.dev`, or `platform_device.dev`. `con_id` is the function of the GPIO within the consumer interface. It corresponds to the `<name>` prefix of the GPIO specifier property name in the device tree. `idx` is the index (starting from `0`) of the GPIO in case the specifier contains a list of GPIOs. `flags` is an optional parameter that determines the GPIO initialization flags, to configure the direction and/or the initial output value. It is an instance of `enum gpiod_flags`, defined in `include/linux/gpio/consumer.h` as follows:

```
enum gpiod_flags {
    GPIOD_ASIS  = 0,
    GPIOD_IN = GPIOD_FLAGS_BIT_DIR_SET,
    GPIOD_OUT_LOW = GPIOD_FLAGS_BIT_DIR_SET |
                    GPIOD_FLAGS_BIT_DIR_OUT,
    GPIOD_OUT_HIGH = GPIOD_FLAGS_BIT_DIR_SET |
                     GPIOD_FLAGS_BIT_DIR_OUT |
                     GPIOD_FLAGS_BIT_DIR_VAL,
};
```

Let's demonstrate in the following how these APIs can be used in drivers:

```
struct gpio_desc *red, *green, *blue, *power;
red = gpiod_get_index(dev, "led", 0, GPIOD_OUT_HIGH);
green = gpiod_get_index(dev, "led", 1, GPIOD_OUT_HIGH);
blue = gpiod_get_index(dev, "led", 2, GPIOD_OUT_HIGH);
power = gpiod_get(dev, "power", GPIOD_OUT_HIGH);
```

For the sake of readability, the preceding code does not perform error checking. The LED GPIOs will be active-high, but the power GPIO will be active-low (that is, `gpiod_is_active_low(power)` returns `true` in this case).

Since the `flags` argument is optional, there might be situations where either the initial flags are not specified or when the initial function of the GPIO needs to be changed. To address this, drivers can use `gpiod_direction_input()` or `gpiod_direction_output()` to change the GPIO direction. These APIs are defined as follows:

```
int gpiod_direction_input(struct gpio_desc *desc);
int gpiod_direction_output(struct gpio_desc *desc,
                           int value);
```

In the preceding APIs, `desc` is the GPIO descriptor of the GPIO of interest, and `value` is the initial value to apply to this GPIO when it is configured as output.

It must be noted that the same attention must be paid as with the integer-based interface. In other words, the driver must take care of whether the underlying GPIO chip is memory-mapped (and thus can be accessed in any context) or sits on a slow bus (which would require accessing the chip in process or threaded context exclusively). This can be achieved using the `gpiod_cansleep()` function, defined as follows:

```
int gpiod_cansleep(const struct gpio_desc *desc);
```

This function returns `true` if the underlying hardware can put the caller to sleep while it is accessed. In such cases, drivers should use dedicated APIs.

The following are APIs to get or set the GPIO value on a controller that sits on a slow bus, that is, a GPIO descriptor for which `gpiod_cansleep()` returned `true`:

```
int gpiod_get_value_cansleep(const struct gpio_desc *desc);
void gpiod_set_value_cansleep(struct gpio_desc *desc,
                              int value);
```

If the underlying chip is memory mapped, the following APIs can be used instead:

```
int gpiod_get_value(const struct gpio_desc *desc);
void gpiod_set_value(struct gpio_desc *desc, int value);
```

The context must be considered only if the driver is intended to access the GPIO(s) from within an interrupt handler or from within any other atomic context. Otherwise, you can just use the normal APIs, that is, the ones without the `_cansleep` suffix.

`gpiod_to_irq()` can be used to get the IRQ number that corresponds to a GPIO descriptor mapped to IRQ:

```
int gpiod_to_irq(const struct gpio_desc *desc);
```

The resulting IRQ number can be used with the `request_irq()` function (or the threaded variant `request_threaded_irq()`). If the driver does not need to bother with the context supported by the underlying hardware chip, `request_any_context_irq()` can be used instead. That said, the driver can use the device managed variant of these functions, that is, `devm_request_irq()`, `devm_request_threaded_irq()`, or `devm_request_any_context_irq()`.

If for any reason the module needs to switch back and forth between the descriptor-based interface and the legacy integer-based interface, the APIs `desc_to_gpio()` and `gpio_to_desc()` can be used for translations. They are defined as follows:

```
/* Convert between the old gpio_ and new gpiod_ interfaces */
struct gpio_desc *gpio_to_desc(unsigned gpio);
int desc_to_gpio(const struct gpio_desc *desc);
```

In the preceding, `gpio_to_desc()` takes a legacy GPIO number in the parameter and returns the associated GPIO descriptor, while `desc_to_gpio()` does the opposite.

The advantage of using the device-managed APIs is that drivers need not care about releasing the GPIO at all, since it will be handled by the GPIO core. If, however, non-managed APIs were used to request a GPIO descriptor, this descriptor must explicitly be released with `gpiod_put()`, defined as follows:

```
void gpiod_put(struct gpio_desc *desc);
```

Now that we are done with the consumer side's descriptor-based APIs, let's summarize what we have learned in a concrete example, from the mapping from the device tree to the consumer code based on consumer APIs.

### Putting it all together

The following driver summarizes the concepts introduced in the descriptor-based interface. In this example, we need four GPIOs split as follows: two for LEDs (red and green, which are then configured as output) and two for buttons (thus configured as input). The logic to implement is that pushing button 1 toggles both LEDs only when button 2 is pushed as well.

To achieve that, let's consider the following mapping in the device tree:

```
foo_device {
    compatible = "packt,gpio-descriptor-sample";
    led-gpios = <&gpio2 15 GPIO_ACTIVE_HIGH>, // red 
                <&gpio2 16 GPIO_ACTIVE_HIGH>, // green 
    btn1-gpios = <&gpio2 1 GPIO_ACTIVE_LOW>;
    btn2-gpios = <&gpio2 31 GPIO_ACTIVE_LOW>;
};
```

Now that the GPIOs have been mapped in the device tree, let's write the platform driver that will leverage these GPIOs:

```
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/platform_device.h> /* platform devices */
#include <linux/gpio/consumer.h>   /* GPIO Descriptor */
#include <linux/interrupt.h>       /* IRQ */
#include <linux/of.h>              /* Device Tree */
static struct gpio_desc *red, *green, *btn1, *btn2;
static unsigned int irq, led_state = 0;
static irq_handler_t btn1_irq_handler(unsigned int irq,
                                      void *dev_id)
{
    unsigned int btn2_state;
    btn2_state = gpiod_get_value(btn2);
    if (btn2_state) {
        led_state = 1 – led_state;
        gpiod_set_value(red, led_state);
        gpiod_set_value(green, led_state);
    }
    pr_info("btn1 interrupt: Interrupt! btn2 state is %d)\n",
            led_state);
    return IRQ_HANDLED;
}
```

In the preceding, we have started with the IRQ handler. The toggling logic is implemented by `led_state = 1 – led_state`. Next, we implement the driver's `probe` method, as follows:

```
static int my_pdrv_probe (struct platform_device *pdev)
{
    int retval;
    struct device *dev = &pdev->dev;
    red = devm_gpiod_get_index(dev, "led", 0,
                               GPIOD_OUT_LOW);
    green = devm_gpiod_get_index(dev, "led", 1,
                                 GPIOD_OUT_LOW);
    /* Configure GPIO Buttons as input */
    btn1 = devm_gpiod_get(dev, "led", 0, GPIOD_IN);
    btn2 = devm_gpiod_get(dev, "led", 1, GPIOD_IN);
    irq = gpiod_to_irq(btn1);
    retval = devm_request_threaded_irq(dev, irq, NULL,
                         btn1_pushed_irq_handler,
                         IRQF_TRIGGER_LOW | IRQF_ONESHOT,
                         "gpio-descriptor-sample", NULL);
    pr_info("Hello! device probed!\n");
    return 0;
}
```

The preceding probe method is quite simple. We first start requesting the GPIOs, then we translate the button 1 GPIO line into a valid IRQ number, and then we register a handler for this IRQ. You should pay attention to the fact that we have exclusively used device-managed APIs in that method.

Finally, we set up a device ID table before filling and registering our platform device driver, as follows:

```
static const struct of_device_id gpiod_dt_ids[] = {
    { .compatible = "packt,gpio-descriptor-sample", },
    { /* sentinel */ }
};
static struct platform_driver mypdrv = {
    .probe      = my_pdrv_probe,
    .driver     = {
        .name     = "gpio_descriptor_sample",
        .of_match_table = of_match_ptr(gpiod_dt_ids),  
        .owner    = THIS_MODULE,
    },
};
module_platform_driver(mypdrv);
MODULE_AUTHOR("John Madieu <john.madieu@labcsmart.com>");
MODULE_LICENSE("GPL");
```

We may wonder why neither GPIO descriptors nor interrupts were released. This is because we exclusively used device-managed APIs in the `probe` function. Thanks to these, we do not need to release anything explicitly, thus we can get rid of the `remove` method of the platform driver.

If we use non-managed APIs, the `remove` method could look like the following:

```
static void my_pdrv_remove(struct platform_device *pdev)
{
    free_irq(irq, NULL);
    gpiod_put(red);
    gpiod_put(green);
    gpiod_put(btn1);
    gpiod_put(btn2);
    pr_info("good bye reader!\n");
}
static struct platform_driver mypdrv = {
    [...]
    .remove     = my_pdrv_remove,
    [...]
};
```

In the preceding, we can notice the use of regular `gpiod_put()` and `free_irq()` APIs to release GPIO descriptors and the IRQ line.

In this section, we have done with the kernel side of GPIO management, both on the controller and client sides. As we have learned all through this book, there are situations where we might want to avoid writing specific kernel code. Regarding GPIOs, the next section will teach us how not to write GPIO client drivers to control these GPIOs.

# Learning how not to write GPIO client drivers

There are situations where writing user space code would achieve the same goals as writing kernel drivers. Moreover, the GPIO framework is one of the most used frameworks in the user space. It then goes without saying that there are several possibilities to deal with it in the user space, some of which we will introduce in this chapter.

## Goodbye to the legacy GPIO sysfs interface

Sysfs has ruled GPIO management from the user space for quite a long time now. Though it is scheduled for removal, the sysfs GPIO interface still has a few days ahead of it. `CONFIG_GPIO_SYSFS` can still enable it, but its use is discouraged, and it will be removed from mainline Linux. This interface allows managing and controlling GPIOs through a set of files. It is located at `/sys/class/gpio/`, and the following are the common directory paths and attributes that are involved:

*   `/sys/class/gpio/`: This is where it all starts. There are two special files in this directory, `export` and `unexport`, and as many directories as there are GPIO controllers registered with the system:
    *   `export`: By writing the number of a GPIO to this file, we ask the kernel to export control of that GPIO to the user space. For example, typing `echo 21 > export` will create a `gpio21` node (resulting in a subdirectory of the same name) for GPIO `#21`, if this GPIO is not already requested by the kernel code.
    *   `unexport`: The effect of exporting to the user space is reversed by writing the same GPIO number to this file. For example, the `gpio21` node exported using the `export` file will be removed by typing `echo 21 > unexport`.

On successful `gpio_chip` registration, a directory entry with a path such as `/sys/class/gpio/gpiochipX/` will be created, where `X` is the GPIO controller base (the controller providing GPIOs starting at `#X`), having the following attributes:

*   `base`, whose value is the same as `X`, and which corresponds to `gpio_chip.base` (if assigned statically), and being the first GPIO managed by this chip.
*   `label`, which is provided for diagnostics (not always unique).
*   `ngpio`, which tells us how many GPIOs this controller provides (`N` to `N + ngpio - 1`). This is the same as defined in `gpio_chip.ngpios`.

*   `/sys/class/gpio/gpioN/`: This directory corresponds to the GPIO line `N`, exported either using the `export` file or directly from the kernel. `/sys/class/gpio/gpio42/` (for GPIO `#42`) is an example. The following read/write attributes are contained in such directories:
    *   `direction`: Use this file to get/set GPIO direction. Acceptable values are either `in` or `out` strings. This attribute will normally be written and writing the `out` value will initialize the GPIO value as `low` by default. To ensure glitch-free operation, values low and high may be written to configure the GPIO as an output with that initial value. If, however, the GPIO has been exported from the kernel (see the `gpiod_export()` or `gpio_export()` functions), then this attribute will be missing, disabling at the same time direction change.
    *   `value`: This attribute can be used to get or set the state of the GPIO line based on its direction, input, or output. If the GPIO is configured as an output, any non-zero value written will set the output high, while writing `0` will set this output low. If the pin can be set up as an interrupt-generating line and is set to do so, then the `poll()` system function can be used on that file and will return when an interrupt occurs. Setting the events `POLLPRI` and `POLLERR` is required when using `poll()`. If, however, `select()` is used instead, the file descriptor should be set in `exceptfds`. After `poll()` returns, the user code should either `lseek()` to the beginning of the sysfs file and read the new value or close the file and re-open it to read the value. It is the same principle as we discussed for the pollable sysfs attribute.
    *   `edge` determines the signal edge that will let the `poll()` or `select()` functions return. `none`, `rising`, `failing`, or `both` are acceptable values. This readable and writable file exists only if the GPIO can be configured as an interrupt generating input pin.
    *   `active_low`: When it is read, this attribute either returns `0` (for `false`) or `1` (meaning `true`). Writing any nonzero value will invert the `value` attribute for both reading and writing. Existing and subsequent `poll()`/`select()` support configuration through the `edge` attribute for rising and falling edges will follow this setting.

The following is a short sequence of commands demonstrating the use of the sysfs interface to drive GPIOs from the user space:

```
# echo 24 > /sys/class/gpio/export
# echo out > /sys/class/gpio/gpio24/direction
# echo 1 > /sys/class/gpio/gpio24/value
# echo 0 > /sys/class/gpio/gpio24/value
# echo high > /sys/class/gpio/gpio24/direction # shorthand for out/1
# echo low > /sys/class/gpio/gpio24/direction # shorthand for out/0
```

Let's not spend more time on this legacy interface. Without delay, let's switch to what kernel developers have provided as a new GPIO management interface from the user space, the `Libgpiod` library.

## Welcome to the Libgpiod GPIO library

The Kernel Linux GPIO user space sysfs is deprecated and has been discontinued. That said, it was suffering from many ailments, some of which are as follows:

*   State not tied to processes.
*   A lack of concurrent access management to sysfs attributes.
*   A lack of support for bulk GPIO operations, that is, performing operations on a set of GPIOs with a single command (in a single shot).
*   A lot of operations were needed just to set a GPIO value (opening and writing into the export file, opening and writing into the direction file, opening and writing into the value file).
*   Unreliable polling – user code had to poll on `/sys/class/gpio/gpioX/`value, and on each event, it was necessary to `close/re-open` or `lseek` in the file before re-reading the new value. This could lead to events being lost.
*   It was not possible to set GPIO electrical properties.
*   If the process crashed, the GPIOs remained exported; there was no context concept.

To address the limits of the sysfs interface, a new GPIO interface has been developed, the descriptor-based GPIO interface. It comes with GPIO character devices – a new user API, merged in Linux v4.8\. This new interface has introduced the following improvements:

*   One device file per GPIO chip: `/dev/gpiochip0`, `/dev/gpiochip1`, `/dev/gpiochipX` ….
*   It's similar to other kernel interfaces: `open()` + `ioctl()` + `poll()` + `read()` + `close()`.
*   It's possible to request multiple lines at once (for reading/setting values) using bulk-related APIs.
*   It's possible to find GPIO lines and chips by name, which is much more reliable.
*   Open source and open-drain flags, user/consumer strings, and uevents.
*   Reliable polling, preventing the loss of events.

`Libgpiod` is shipped with a C API allowing you to get the most out of any GPIO chip registered on the system. That said, the C++ and Python languages are supported as well. The API is well documented, and too extensive to fully cover here. The basic use cases usually follow these steps:

1.  Open the desired GPIO chip character device by calling one of the `gpiod_chip_open*` functions, such as `gpiod_chip_open_by_name()` or `gpiod_chip_open_lookup()`. This returns a pointer to `struct gpiod_chip`, which is used by subsequent API calls.
2.  Retrieve the handle to the desired GPIO line by calling `gpiod_chip_get_line()`, which will return a pointer to an instance of `struct gpiod_line`. While the previous API returns the handle to a single GPIO line, the function `gpiod_chip_get_lines()` can be used if several GPIO lines are needed in a single shot. `gpiod_chip_get_lines()` will return a pointer to an instance of `struct gpiod_line_bulk`, which can be used later for bulk operations. The other API that can return a set of GPIO handles is `gpiod_chip_get_all_lines()`, which returns all the lines of a given GPIO chip in `struct gpiod_line_bulk`. When you have such a set of GPIO objects, you can request a GPIO line at a specific index local to this bulk object by using the `gpiod_line_bulk_get_line()` API.
3.  Request the use of the line as an input or output by calling `gpiod_line_request_input()` or `gpiod_line_request_output()`. For bulk operations on a set of GPIO lines, `gpiod_line_request_bulk_input()` or `gpiod_line_request_bulk_output()` can be used instead.
4.  Read the value of input GPIO lines by calling `gpiod_line_get_value()` for a single GPIO or `gpiod_line_get_value_bulk()` in the case of a set of GPIOs. For output GPIO lines, the level can be set by calling `gpiod_line_set_value()` for a single GPIO line or `gpiod_line_set_value_bulk()` on a set of output GPIOs.
5.  When done, release the lines by calling `gpiod_line_release()` or `gpiod_line_release_bulk()`.
6.  Once all the GPIO lines have been released, the associated chips can be released using `gpiod_chip_close()` on each.

`gpiod_line_release()` is to be called once done with a GPIO line. The GPIO line to release is passed as a parameter. If it is, however, a set of GPIOs that needs to be released, `gpiod_line_release_bulk()` should be used instead. It has to be noted that if the lines were not previously requested together (were not requested with `gpiod_line_request_bulk()`), the behavior of `gpiod_line_release_bulk()` is undefined.

There are sanity APIs it might worth mentioning, which are defined as follows:

```
bool gpiod_line_is_free(struct gpiod_line *line);
bool gpiod_line_is_requested(struct gpiod_line *line);
```

In the preceding APIs, `gpiod_line_is_requested()` can be used to check if the calling user owns this GPIO line. This function returns `true` if `line` was already requested, or `false` otherwise. It is different from `gpiod_line_is_free()`, which is used to check if the calling user has neither requested ownership `line` nor set up any event notifications on it. It returns `true` if `line` is free, and `false` otherwise.

Other APIs are available for more advanced functions such as configuring pin modes for pullup or pulldown resistors or registering a callback function to be called when an event occurs, such as the level of an input pin changing, as we will see in the next section.

### Event- (interrupt-) driven GPIO

Interrupt-driven GPIO handling consists of grabbing one (`struct gpiod_line`) or more (`struct gpiod_line_bulk`) GPIO handles and listening for events on these GPIO lines, either infinitely or in a timed manner.

A GPIO line event is abstracted by a `struct gpiod_line_event` object, defined as follows:

```
struct gpiod_line_event {
    struct timespec ts;
    int event_type;
};
```

In the preceding data structure, `ts` is the time specifier data structure to represent the wait event timeout, and `event_type` is the type of event, which can be either `GPIOD_LINE_EVENT_RISING_EDGE` or `GPIOD_LINE_EVENT_FALLING_EDGE`, respectively for a rising edge event or a falling edge event.

After the GPIO handle(s) has been obtained using `gpiod_chip_get_line()` or `gpiod_chip_get_lines()` or `gpiod_chip_get_all_lines()`, the user code should request events of interest on these GPIO handles using one of the following APIs:

```
int gpiod_line_request_rising_edge_events(
                          struct gpiod_line *line,
                          const char *consumer);
int gpiod_line_request_bulk_rising_edge_events(
                          struct gpiod_line_bulk *bulk,
                          const char *consumer);
int gpiod_line_request_falling_edge_events(
                          struct gpiod_line *line,
                          const char *consumer);
int gpiod_line_request_bulk_falling_edge_events(
                          struct gpiod_line_bulk *bulk,
                          const char *consumer);
int gpiod_line_request_both_edges_events(
                          struct gpiod_line *line,
                          const char *consumer);
int gpiod_line_request_bulk_both_edges_events(
                          struct gpiod_line_bulk *bulk,
                          const char *consumer);
```

The preceding APIs request either rising edge, falling edge, or both edge events, respectively on a single GPIO line or on a set of GPIO (the bulk-related API).

After the events have been requested, the user code can wait on the GPIO lines of interest, waiting for the requested events to occur using one of the following APIs:

```
int gpiod_line_event_wait(struct gpiod_line *line,
                  const struct timespec *timeout);
int gpiod_line_event_wait_bulk(
                  struct gpiod_line_bulk *bulk,
                  const struct timespec *timeout,
                  struct gpiod_line_bulk *event_bulk);
```

In the preceding, `gpiod_line_event_wait()` waits for event(s) on a single GPIO line, while `gpiod_line_event_wait_bulk()` will wait on a set of GPIOs. In parameters, `line` is the GPIO line on which to wait events in the case of single GPIO monitoring, while `bulk` is the set of GPIO lines in the case of bulk monitoring. Finally, `event_bulk` is an output parameter, holding the set of GPIO lines on which the GPIO events of interest have occurred. These are all blocking APIs, which will continue execution flow only after the events of interest have occurred or after a timeout.

Once the blocking function returns, `gpiod_line_event_read()` must be used to read the events that occurred on the GPIO line(s) returned by the previously mentioned monitoring functions. This API has the following prototype:

```
int gpiod_line_event_read(struct gpiod_line *line,
                         struct gpiod_line_event *event);
```

On error, this API returns `-1`, otherwise, it returns `0`. In parameters, `line` is the GPIO line to read the events on, and `event` is an output parameter, the event buffer to which the event data will be copied.

The following is an example of requesting an event and reading and processing that event:

```
char *chipname = "gpiochip0";
int ret;
struct gpiod_chip *chip;
struct gpiod_line *input_line;
struct gpiod_line_event event;
unsigned int line_num = 25;  /* GPIO Pin #25 */
chip = gpiod_chip_open_by_name(chipname);
if (!chip) {
     perror("Open chip failed\n");
     return -1;
}
input_line = gpiod_chip_get_line(chip, line_num);
if (!input_line) {
     perror("Get line failed\n");
     ret = -1;
     goto close_chip;
}
ret = gpiod_line_request_rising_edge_events(input_line,
                                            "gpio-test");
if (ret < 0) {
     perror("Request event notification failed\n");
     ret = -1;
     goto release_line;
}
while (1) {
  gpiod_line_event_wait(input_line, NULL); /* blocking */
  if (gpiod_line_event_read(input_line, &event) != 0)
        continue;

    /* should always be a rising event in our example */
    if (event.event_type != GPIOD_LINE_EVENT_RISING_EDGE)
        continue;
    [...]
}
release_line:
     gpiod_line_release(input_line);
close_chip:
     gpiod_chip_close(chip);
     return ret;
```

In the preceding snippet, we first look up the GPIO chip by its name and use the returned GPIO chip handle to grab a handle on GPIO line #25\. Next, we request a rising events notification (interrupt-driven) on the GPIO line. After that, we loop on waiting for events to happen, read which event it was, and validate that it's a rising event.

Apart from the previous code example, let's now imagine a much complex example, where we monitor five GPIO lines, and let's start by feeding the required headers:

```
// file event-bulk.c
#include <gpiod.h>
#include <error.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/time.h>
```

Then, let's provide the static variables we will use in the program:

```
static struct gpiod_chip *chip;
static struct gpiod_line_bulk gpio_lines;
static struct gpiod_line_bulk gpio_events;
/* use GPIOs #4, #7, #9, #15, and #31 as input */
static unsigned int gpio_offsets[] = {4, 7, 9, 15, 31};
```

In the previous snippet, `chip` will hold the handle to the GPIO chip that we are interested in. `gpio_lines` will hold the handles of the event-driven GPIO lines, that is, the GPIO lines to be monitored. Finally, `gpio_events` will be given to the library so that upon monitoring, it is filled with the handles of GPIO lines on which events have occurred.

Finally, let's start implementing our `main` method:

```
int main(int argc, char *argv[])
{
    int err;
    int values[5] = {-1};
    struct timespec timeout;
    chip = gpiod_chip_open("/dev/gpiochip0");
    if (!chip) {
        perror("gpiod_chip_open");
        goto cleanup;
    }
```

In the previous snippet, we have simply opened the GPIO chip device and kept a pointer to it. Next, we will have to grab handles of the GPIO lines of interest and store them in `gpio_lines`:

```
    err = gpiod_chip_get_lines(chip, gpio_offsets, 5,
                               &gpio_lines);
    if (err) {
        perror("gpiod_chip_get_lines");
        goto cleanup;
    }
```

Then, we use these GPIO line handles to request event monitoring on their underlying GPIO lines. Because we are interested in more than one GPIO, we use the `bulk` API variant, as follows:

```
    err = gpiod_line_request_bulk_rising_edge_events(
                    &gpio_lines, "rising edge example");
    if(err) {
        perror(
           "gpiod_line_request_bulk_rising_edge_events");
        goto cleanup;
    }
```

In the previous snippet, `gpiod_line_request_bulk_rising_edge_events()` will request rising edge event notifications. Now that we have requested event-driven monitoring for our GPIO, we can call the blocking monitoring API on these GPIO lines, as follows:

```
    /* Timeout of 60 seconds, pass in NULL to wait forever */
    timeout.tv_sec = 60;
    timeout.tv_nsec = 0;
    printf("waiting for rising edge event \n");
marker1:
    err = gpiod_line_event_wait_bulk(&gpio_lines,
                                     &timeout, &gpio_events);
    if (err == -1) {
        perror("gpiod_line_event_wait_bulk");
        goto cleanup;
    } else if (err == 0) {
        fprintf(stderr, "wait timed out\n");
        goto cleanup;
    }
```

In the previous excerpt, since we need time-bounded event polling, we set up a `struct timespec` data structure with the desired timeout and we pass it to `gpiod_line_event_wait_bulk()`.

That said, reaching this step (passing the polling function) would mean that either the blocking monitoring API has timed out or that an event occurred on at least one of the GPIO lines that are monitored. The GPIO handles on which events occurred are stored in `gpio_events`, which is an output argument, and the list of monitored GPIO lines is passed in `gpio_lines`. It must be noted that both `gpio_lines` and `gpio_events` are bulk GPIO data structures.

If ever we are interested in reading the values of the GPIO lines on which events have occurred, we could do the following:

```
    err = gpiod_line_get_value_bulk(&gpio_events, values);
    if(err) {
        perror("gpiod_line_get_value_bulk");
        goto cleanup;
    }
```

If instead of reading the values of GPIO lines on which events occurred we needed to read the value of all the monitored GPIO lines, we would have replaced `gpio_events` with `gpio_lines` in the previous code.

Next, if we are interested in the type of event that occurred on each GPIO line in `gpio_events`, we can do the following:

```
    for (int i = 0;
         i < gpiod_line_bulk_num_lines(&gpio_events);
         i++) {
        struct gpiod_line* line;
        struct gpiod_line_event event;
        line = gpiod_line_bulk_get_line(&gpio_events, i);
        if(!line) {
           fprintf(stderr, "unable to get line %d\n", i);
           continue;
        }
        if (gpiod_line_event_read(line, &event) != 0)
            continue;
        printf("line %s, %d\n", gpiod_line_name(line),
           gpiod_line_offset(line));
    }
marker2:
```

In the preceding code, we iterate over each GPIO line in `gpio_events`, which represents the list of GPIO lines on which events have occurred. `gpiod_line_bulk_num_lines()` retrieves the number of GPIO lines held by the line bulk object, and `gpiod_line_bulk_get_line()` retrieves the line handle from a line bulk object at the given offset, local to this line bulk object. You should, however, note that to achieve the same goal, we could have used the `gpiod_line_bulk_foreach_line()` macro.

Then, on each GPIO line in the line bulk object, we invoke `gpiod_line_event_read()`, `gpiod_line_name()`, and `gpiod_line_offset()`. The first function will retrieve the event data structure corresponding to the event that occurred on that line. We could have then checked that the event type that occurred (especially when monitoring for both event types) is what we expected using something such as `if (event.event_type != GPIOD_LINE_EVENT_RISING_EDGE)`, for example. The second function is a helper that will retrieve the GPIO line name, while the third one, `gpiod_line_offset()`, will retrieve the GPIO line offset, global to the running system.

If we were interested in monitoring these GPIO lines infinitely or for a certain number of rounds, we could have wrapped the code between the `marker1` and `marker2` labels into a `while()` or a `for()` loop.

At the end of the execution flow, we do some cleaning, like the following:

```
cleanup:
    gpiod_line_release_bulk(&gpio_lines);
    gpiod_chip_close(chip);
    return EXIT_SUCCESS;
}
```

The previous cleaning code snippet first releases all the GPIO lines that we have requested, and then closes the associated GPIO chip.

Note

It must be noted that bulk GPIO monitoring must be done on a per GPIO chip basis. That is, it is not recommended to embed GPIO lines from different GPIO chips in the same line bulk object.

Now that we are done with API usage and have demonstrated it in a practical example, we can switch to command-line tools shipped with the `libgpiod` library.

### Command-line tools

If you simply need to perform simple GPIO operations, the `Gpiod` library includes a collection of command-line tools that are particularly handy for interactively exploring GPIO functions and can be used in shell scripts to avoid the need to write C or C++ code. There are the following commands available:

*   `gpiodetect`: Displays the list of all GPIO chips on the system, together with their names, labels, and the number of GPIO lines.
*   `gpioinfo`: Displays the names, consumers, direction, active status, and other flags for all lines of the selected GPIO chips. `gpioinfo gpiochip6` is an example. If no GPIO chip is given, the command will iterate through all GPIO chips on the system and list their associated lines.
*   `gpioget`: Gets the values of GPIO lines specified.
*   `gpioset`: Sets the values of specified GPIO lines, and potentially keeps them exported until a timeout, user input, or signal occurs.
*   `gpiofind`: Given a line name, this command finds the associated GPIO chip name and line offset.
*   `gpiomon`: Monitors GPIOs by waiting for events on these lines. This command allows you to specify which events to watch and how many of them should be processed before exiting or whether the events should be reported to the console.

Now that we have listed the available command-line tools, we can go on to learn about another mechanism offered by the GPIO subsystem, and that can be leveraged from the user space, thanks to which we can use the aforementioned tools.

## The GPIO aggregator

GPIO access control now uses permissions on `/dev/gpiochip*` with the new interface. The typical Unix filesystem permissions enable all-or-nothing access control to these character devices. Compared to the earlier `/sys/class/gpio` interface, this new interface provides a number of advantages, which we listed at the beginning of the *Welcome to the Libgpiod GPIO library* section. One disadvantage, however, is that it creates one device file per GPIO chip, implying that access privileges are defined on a per GPIO chip basis, rather than per GPIO line.

As a result, the `/dev/gpiochip*` device.

This feature is handy for designating a set of GPIOs to a certain user and implementing access control. Furthermore, exporting GPIOs to a virtual machine is simplified and hardened because the virtual machine can just grab the entire GPIO controller and no longer has to worry about which GPIOs to grab and which not to, decreasing the attack surface. `Documentation/admin-guide/gpio/gpio-aggregator.rst` is where you'll find its documentation.

To have GPIO aggregator support in your kernel, you must have `CONFIG_GPIO_AGGREGATOR=y` in your kernel configuration. This feature can be configured either via sysfs or the device tree, as we will see in the next sections.

### Aggregating GPIOs using sysfs

Aggregated GPIO controllers are instantiated and destroyed by writing to write-only attribute files in sysfs, mainly from the `/sys/bus/platform/drivers/gpio-aggregator/` directory.

This directory contains the followings attributes:

*   `new_device`: Used to ask the kernel to instantiate an aggregated GPIO controller by writing a string describing the GPIOs to aggregate. The `new_device` file understands the format `[<gpioA>] [<gpiochipB> <offsets>] ...`:
    *   `<gpioA>` is a GPIO line name.
    *   `<gpiochipB>` is a GPIO chip label.
    *   `<offsets>` is a comma-separated list of GPIO offsets and/or GPIO offset ranges denoted by dashes.
*   `delete_device`: Used to ask the kernel to destroy an aggregated GPIO controller after use.

The following is an example that instantiated a new GPIO aggregator by aggregating GPIO line 19 of `e6052000.gpio` and GPIO lines 20-21 of `e6050000.gpio` into a new `gpio_chip`:

```
# echo 'e6052000.gpio 19 e6050000.gpio 20-21' > /sys/bus/platform/drivers/gpio-aggregator/new_device
# gpioinfo gpio-aggregator.0
     gpiochip12 - 3 lines:
     line 0: unnamed unused input active-high
     line 1: unnamed unused input active-high
     line 2: unnamed unused input active-high
# chown geert /dev/gpiochip12
```

After use, the previously created aggregated GPIO controller can be destroyed using the following command, assuming it is named `gpio-aggregator.0`:

```
$ echo gpio-aggregator.0 > delete_device
```

From the previous example, the GPIO chip that resulted from the aggregation was `gpiochip12`, having three GPIO lines. Instead of `gpioinfo gpio-aggregator.0`, we could have used `gpioinfo gpiochip12`.

### Aggregating GPIOs from the device tree

The device tree can also be used to aggregate GPIOs. To do so, simply define a node with `gpio-aggregator` as a compatible string and set the `gpios` property to the list of GPIOs that you want to be part of the new GPIO chip. A unique feature of this technique is that, like any other GPIO controller, the GPIO lines can be named and subsequently queried by user-space applications using the `libgpiod` library.

In the following, we will demonstrate the use of the GPIO aggregator with several GPIO lines from the device tree. First, we enumerate the pins we need to use GPIOs in our new GPIO chip. We do this under the pin controller node as follows:

```
&iomuxc {
[...]
   aggregator {
      pinctrl_aggregator_pins: aggretatorgrp {
         fsl,pins = <
           MX6QDL_PAD_EIM_D30__GPIO3_IO30      0x80000000  
           MX6QDL_PAD_EIM_D23__GPIO3_IO23      0x80000000 
           MX6QDL_PAD_ENET_TXD1__GPIO1_IO29    0x80000000
           MX6QDL_PAD_ENET_RX_ER__GPIO1_IO24   0x80000000
           MX6QDL_PAD_EIM_D25__GPIO3_IO25      0x80000000
           MX6QDL_PAD_EIM_LBA__GPIO2_IO27      0x80000000
           MX6QDL_PAD_EIM_EB2__GPIO2_IO30      0x80000000
           MX6QDL_PAD_SD3_DAT4__GPIO7_IO01     0x80000000
         >;
      };
   };
}
```

Now that our pins have been configured, we can declare our GPIO aggregator as follows:

```
gpio-aggregator {
    pinctrl-names = "default";
    pinctrl-0 = <&pinctrl_aggregator_pins>;
    compatible = "gpio-aggregator";
    gpios = <&gpio3 30 GPIO_ACTIVE_HIGH>,
            <&gpio3 23 GPIO_ACTIVE_HIGH>,
            <&gpio1 29 GPIO_ACTIVE_HIGH>,
            <&gpio1 25 GPIO_ACTIVE_HIGH>,
            <&gpio3 25 GPIO_ACTIVE_HIGH>,
            <&gpio2 27 GPIO_ACTIVE_HIGH>,
            <&gpio2 30 GPIO_ACTIVE_HIGH>,
            <&gpio7 1 GPIO_ACTIVE_HIGH>;
    gpio-line-names = "line_a", "line_b", "line_c",
            "line_d", "line_e", "line_f", "line_g",
            "line_h";
};
```

In this example, `pinctrl_aggregator_pins` is the GPIO pin node, which must have been instantiated under the pin controller node. `gpios` contains the list of GPIO lines the new GPIO chip must be made of. At the end, the meaning of `gpio-line-names` is line 30 of GPIO controller `gpio3` is used and is named `line_a`, line 23 of GPIO controller `gpio3` is used and is named `line_b`, line 29 of GPIO controller `gpio1` is used and named `line_c`, and so on up to line 1 of GPIO controller `gpio7`, which is named `line_h`.

From the user space, we can see the GPIO chip and its aggregated lines:

```
# gpioinfo
[...]
gpiochip9 - 8 lines:
    line 0: "line_a" unused input active-high
    line 1: "line_b" unused input active-high
[...]
    line 7: "line_g" unused input active-high
[...]
```

We can search a GPIO chip and a line number by the line name:

```
# gpiofind 'line_b'
gpiochip9 1
```

We can access a GPIO line by its name:

```
# gpioget $(gpiofind 'line_b')
1
#
# gpioset $(gpiofind 'line_h')=1
# gpioset $(gpiofind 'line_h')=0
```

We can change the GPIO chip device file ownership to allow user or group to access the attached lines:

```
# chown $user:$group /dev/gpiochip9
# chmod 660 /dev/gpiochip9
```

The GPIO chip created by the aggregator can be retrieved from sysfs in `/sys/bus/platform/devices/gpio-aggregator/`.

### Aggregating GPIOs using a generic GPIO driver

Without a particular in-kernel driver, the GPIO aggregator can be used as a generic driver for a simple GPIO-operated device described in the device tree. Modifying the `gpio-aggregator` driver or writing to the `driver_override` file in sysfs are both options for binding a device to the GPIO aggregator.

Before we go further, let's talk about the `driver_override` file; this file is more precisely located in `/sys/bus/platform/devices/.../driver_override`. This file specifies the driver for a device, which will override the standard device tree, ACPI, ID table, and name matching, as we have seen in [*Chapter 6*](B17934_06_Epub.xhtml#_idTextAnchor095), *Introduction to Devices, Drivers, and Platform Abstraction*. It has to be noted that only a driver whose name matches the value written to `driver_override` will be able to bind to the device. The override is set by writing a string to the `driver_override` file (`echo vfio-platform > driver_override`), and it can be cleared by writing an empty string to the file (`echo > driver_override`). This reverts the device to its default binding of matching rules. It must, however, be noted that writing to driver override does not unbind the device from its existing driver or attempt to load the supplied driver automatically. The device will not bind to any driver if no driver with a matching name is currently loaded in the kernel. Devices can also use a `driver_override` name such as `none` to opt out of driver binding. There is no support for parsing delimiters, and only a single driver can be given in the override.

For example, given a `door` device, which is a GPIO-operated device described in the device tree, use its own compatible value as follows:

```
door {
     compatible = "myvendor,mydoor";
     gpios = <&gpio2 19 GPIO_ACTIVE_HIGH>,
           <&gpio2 20 GPIO_ACTIVE_LOW>;
     gpio-line-names = "open", "lock";
};
```

It can be bound to the GPIO aggregator with either of the following methods:

*   Adding its compatible value to `gpio_aggregator_dt_ids[]` in `drivers/gpio/gpio-aggregator.c`
*   Binding manually using `driver_override`

The first method is quite straightforward:

```
$ echo gpio-aggregator > /sys/bus/platform/devices/door/driver_override
$ echo door > /sys/bus/platform/drivers/gpio-aggregator/bind
```

In the previous commands, we have written the driver's name (`gpio-aggregator` in this case) in the `driver_override` file present in the device directory, `/sys/bus/platform/devices/<device-name>/`. After that, we have bound the device to the driver by writing the device name in the `bind` file present in the driver's directory, `/sys/bus/<bus-name>/drivers/<driver-name>/`. It has to be noted that `<bus-name>` corresponds to the bus framework the driver belongs to. It could be `i2c`, `spi`, `platform`, `pci`, `isa`, `usb`, and so on.

After the binding, a new GPIO chip, `door`, will be created. Its information can then be carried out as follows:

```
$ gpioinfo door
gpiochip12 - 2 lines:
     line   0:       "open"       unused   input  active-high
     line   1:       "lock"       unused   input  active-high
```

Next, the library APIs can be used on this GPIO chip like any other normal (non-virtual) GPIO chip.

We are now done with GPIO aggregation from the user space in particular, and with GPIO management from the user space in general. We have learned how to create virtual GPIO chips to isolate a set of GPIOs, and we have learned how to use the GPIO library to drive these GPIOs.

# Summary

In this chapter, we introduced the pin control framework and described its interaction with the GPIO subsystem. We learned how to deal with GPIOs, either as a controller or consumer, from both the kernel and the user space. Though the legacy integer-based interface is deprecated, it was introduced because it is still widely used. Additionally, we introduced some advanced topics such as IRQ chip support in the GPIO chip and the mapping of GPIOs to IRQs. We ended this chapter by learning how to deal with GPIOs from the user space, by writing C code or by using dedicated command-line tools provided by the standard Linux GPIO library, `libgpiod`.

In the next chapter, we deal with input devices, which can be implemented using GPIOs.