# 第十章：处理 SELinux 感知应用程序

在本章中，我们将通过以下几种方案介绍如何处理 SELinux 感知应用程序：

+   控制 D-Bus 消息流

+   限制服务的所有权

+   理解 udev 的 SELinux 集成

+   使用 cron 和 SELinux

+   以编程方式检查 SELinux 状态

+   在 C 中查询 SELinux 用户空间配置

+   通过代码方式查询 SELinux 子系统

+   在新上下文中运行新进程

+   读取资源的上下文

# 介绍

对于大多数应用程序，Linux 内核中的 SELinux 子系统能够在不与其他应用程序和组件进一步交互的情况下执行安全控制。然而，存在一些无法由 SELinux 子系统自动处理的操作。一些应用程序为特定用户执行命令，但目标域无法从正在执行的应用程序的路径中推断出来，因此无法基于标签进行类型转换。

解决这个问题的一个方法是使应用程序成为 SELinux 感知的，让应用程序查询 SELinux 子系统，以确定新执行的应用程序应该使用什么上下文。一旦获得上下文，应用程序就可以指示 SELinux 子系统将该上下文分配给接下来将启动的进程。

当然，问题不仅仅是决定进程应该处于哪个上下文。应用程序也可以检查 SELinux 策略，并根据策略进行操作，而不是通过 Linux 内核强制执行策略。如果应用程序使用 SELinux 获取更多有关会话的信息，并根据这些信息设置上下文，那么我们称这些应用程序为 SELinux 感知的。

查看应用程序是否为 SELinux 感知的最简单方法是检查文档，或者检查它是否与 `libselinux.so` 库链接：

```
~$  ldd /usr/sbin/crond | grep selinux
 libselinux.so.1 => /lib64/libselinux.so.1 (0x00007fa53299a000)

```

一些 SELinux 感知应用程序不仅查询信息，还对 SELinux 子系统无法控制的对象执行决策。这些对象的例子包括 **安全增强型 PostgreSQL** (**SEPostgreSQL**) 应用程序中的数据库对象或 D-Bus 服务。尽管它们在 SELinux 策略中有所表示，但它们并不是常规 Linux 操作系统的一部分，而是由应用程序本身拥有的。这些 SELinux 感知应用程序被称为 **用户空间对象管理器**。

无论应用程序如何处理其 SELinux 特定代码，在系统上使用这些应用程序时，了解应用程序中 SELinux 代码的工作方式非常重要，因为标准方法（查看 AVC 拒绝记录，看看是否需要更改上下文或调整策略）在这些情况下可能根本不起作用。

# 控制 D-Bus 消息流

D-Bus 在 Linux 上的实现是一个 SELinux 感知的应用程序示例，充当用户空间对象管理器。应用程序可以在总线上注册自己，并且可以通过 D-Bus 在应用程序之间发送消息。这些消息也可以通过 SELinux 策略进行控制。

## 准备工作

在查看与消息流相关的 SELinux 访问控制之前，重要的是先关注一个 D-Bus 服务，查看它的认证方式（以及消息如何在 D-Bus 中传递），因为这会反映在 SELinux 集成中。

转到 `/etc/dbus-1/system.d/`（该目录存放 D-Bus 服务的配置文件），查看其中的配置文件。例如，`dnsmasq` 的服务配置文件如下所示：

```
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN" "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <policy user="root">
    <allow own="uk.org.thekelleys.dnsmasq"/>
    <allow send_destination="uk.org.thekelleys.dnsmasq"/>
  </policy>
  <policy context="default">
    <deny own="uk.org.thekelleys.dnsmasq"/>
    <deny send_destination="uk.org.thekelleys.dnsmasq"/>
  </policy>
</busconfig>
```

该配置告诉 D-Bus，只有 root Linux 用户被允许拥有 `uk.org.thekelleys.dnsmasq` 服务，并向该服务发送消息。其他用户（根据默认策略进行管理）被拒绝执行这些操作。

在启用 SELinux 的系统中，单纯使用 root 作为最小粒度的权限是不够的。因此，我们需要了解 SELinux 策略如何在 D-Bus 中提供细粒度的访问控制。

## 如何操作……

要通过 SELinux 控制 D-Bus 消息流，请执行以下步骤：

1.  确定我们感兴趣的 D-Bus 服务所属应用的域。对于 `dnsmasq` 应用来说，这将是 `dnsmasq_t`：

    ```
    ~# ps -eZ | grep dnsmasq | awk '{print $1}'
    system_u:system_r:dnsmasq_t:s0-s0:c0.c1023

    ```

1.  确定希望向服务发送消息的应用的域。例如，这可以是 `sysadm_t` 用户域。

1.  允许这两个域通过 D-Bus 消息相互交互，方法如下：

    ```
    gen_require(`
      class dbus send_msg;
    ')
    allow sysadm_t dnsmasq_t:dbus send_msg;
    allow dnsmasq_t sysadm_t:dbus send_msg;
    ```

## 它是如何工作的……

当一个应用连接到 D-Bus 时，其连接的 SELinux 标签会作为发送消息时检查的标签。由于这种连接没有过渡，因此连接的标签就是进程本身的上下文（域）；因此在示例中选择了 `dnsmasq_t`。

当 D-Bus 接收到发送消息到服务的请求时，D-Bus 会检查 SELinux 策略中的 `send_msg` 权限。它通过将会话的信息（源和目标上下文以及请求的权限）传递给 SELinux 子系统，来计算是否应允许访问。然而，访问控制本身并不由 SELinux 强制执行（它只是提供反馈），而是由 D-Bus 自身执行，因为管理消息流是 D-Bus 的责任。

这也是为什么在开发 D-Bus 相关策略时，需要在策略模块中显式提及类和权限。没有这一点，开发环境可能会报错，声称 `dbus` 不是有效的类。

D-Bus 检查发送消息的客户端的上下文以及服务的连接上下文（这两个都是域标签），并查看是否允许`send_msg`权限。由于大多数通信是双向的（发送消息然后接收回复），因此权限在两个方向上都会被检查。毕竟，发送回复本质上就是在相反方向上发送消息（从策略角度来看）。

如果规则位于用户域中，可以使用`dbus-send`验证此行为。例如，要查看服务提供的对象，可以对该服务调用 D-Bus introspection：

```
~# dbus-send --system --dest=uk.org.thekelleys.dnsmasq --print-reply /uk/org/thekelleys/dnsmasq org.freedesktop.DBus.Introspectable.Introspect

```

当 SELinux 没有正确的`send_msg`允许规则时，D-Bus 会在其服务日志中记录以下错误（但不会出现 AVC 拒绝，因为不是 SELinux 子系统拒绝了访问）：

```
Error org.freedesktop.DBus.Error.AccessDenied: An SELinux policy prevents this sender from sending this message to this recipient. 0 matched rules; type="method_call", sender=":1.17" (uid=0 pid=6738 comm="") interface="org.freedesktop.DBus.Introspectable" member="Introspect" error name="(unset)" requested_reply="0" destination="uk.org.thekelleys.dnsmasq" (uid=0 pid=6635 comm="")

```

当策略允许`send_msg`权限时， introspection 会返回一个 XML 输出，显示该服务提供的方法和接口。

## 还有更多…

当前的 D-Bus 实现是纯用户空间实现。随着越来越多的应用程序依赖于 D-Bus，正在进行创建基于内核的 D-Bus 实现，即 **kdbus**。该项目的具体实现细节尚未完成，因此尚不清楚目前适用于 D-Bus 的 SELinux 访问控制是否仍然适用于 kdbus。

# 限制服务所有权

在总线注册的应用程序拥有服务名称。`uk.org.thekelleys.dnsmasq` 服务名称就是其中一个例子。D-Bus 策略在 `/etc/dbus-1/system.d/`（如果服务用于会话总线而不是系统总线，则为`session.d/`）中的 `busconfig` XML 文件中声明，为 D-Bus 提供信息以决定何时允许获取特定服务的所有权。

多亏了 D-Bus 的 SELinux 集成，可以添加额外的约束，确保只有授权的应用程序才能拥有特定服务。

## 如何执行…

要通过 SELinux 策略限制服务所有权，请按照以下步骤操作：

1.  在服务的 D-Bus 配置文件中，确保`own`权限得到了适当保护。例如，确保只有`root` Linux 用户可以拥有该服务：

    ```
    <policy user="root">
      <allow own="uk.org.thekelleys.dnsmasq" />
    </policy>
    ```

1.  如果运行时服务帐户可能不同，则也可以声明`group=`参数，而不是`user=`参数。

1.  接下来，声明与服务关联的标签：

    ```
    <selinux>
      <associate own="uk.org.thekelleys.dnsmasq" context="dnsmasq_t" />
    </selinux>
    ```

1.  在 SELinux 策略中，声明哪些域可以获取该服务：

    ```
    gen_require(`
      class dbus acquire_svc;
    ')
    allow dnsmasq_t self:dbus acquire_svc;
    ```

## 它是如何工作的…

D-Bus 配置允许管理员定义何时可以获取特定服务的所有权。大多数服务会定义允许拥有服务的用户（或组），如示例所示。但对于系统服务，仅声明 Linux 根用户可以拥有特定服务显然不够精细。

进入 SELinux。通过在 `busconfig` XML 文件中的关联定义，D-Bus 会告知任何尝试拥有该特定服务的应用程序域必须具备在所述上下文中对 `dbus` 类的 `acquire_svc` 权限。

通过这种方法，管理员可以确保即使其他域以 Linux root 用户身份运行，也不允许拥有该服务。

尽管通常方法是为目标标签要求应用程序本身的上下文，但也可以使用不同的上下文。例如，可以声明一个新的类型，如 `dnsmasq_dbus_t`，然后将 SELinux 策略设置为如下：

```
allow dnsmasq_t dnsmasq_dbus_t:dbus acquire_svc;
```

## 还有更多……

D-Bus 应用程序在 `/etc/selinux/mcs/contexts/` 目录下有一个配置文件，文件结构相同，名为 `dbus_contexts`。这是 D-Bus 所有权的默认上下文定义（如果无法通过其他方式推断，则应使用哪个上下文）。默认情况下，D-Bus 已经完全意识到需要使用哪些上下文，因此不再提供任何 SELinux 特定的设置，并且不建议再修改此文件。

然而，了解该文件的存在及其使用仍然很有用，特别是当 D-Bus 在容器、chroot 或其他环境中执行时，因为如果该文件缺失，D-Bus 会抱怨：

```
Failed to start message bus: Failed to open "/etc/selinux/mcs/contexts/dbus_contexts": No such file or directory

```

如果需要禁用 D-Bus 中的 SELinux 支持（但不重新构建 D-Bus），则编辑 `/etc/dbus-1/system.conf` 和 `session.conf` 文件，并删除以下行：

```
<include if_selinux_enabled="yes" selinux_root_relative="yes">contexts/dbus_contexts</include>
```

# 了解 udev 的 SELinux 集成

udev 设备管理器负责在 `/dev/` 结构内处理设备文件的变动。由于许多设备文件具有不同的上下文，如果没有 SELinux 感知，udev 策略就需要通过许多命名文件转换来增强。例如，对于设备 `/dev/mydevice` 转换为 `mydevice_t` 类型的命名文件转换如下所示：

```
dev_filetrans(udev_t, mydevice_t, chr_file, "mydevice")
```

然而，当需要为 `/dev/mydevice1`、`/dev/mydevice2` 等设备文件打上标签时，每个可能的名称都需要在策略中迭代（命名文件转换不支持正则表达式）。幸运的是，udev 具有 SELinux 感知能力，因此无需为每个设备文件创建策略增强。

这个方法向我们展示了何时需要额外的策略增强，何时不需要。

## 如何做……

要了解 udev 的 SELinux 集成如何工作，可以遵循以下决策标准：

1.  每当 udev 在具有 `device_t` 标签的目录中创建设备文件时，如果目标类型被赋予 `device_node` 属性，则 udev 会根据 SELinux 子系统通过 `file_contexts` 定义的标签自动为设备文件打上标签。

1.  如果父目录未使用 `device_t` 类型，则确保 udev 对该目标类型具有管理权限。

1.  如果目标文件上下文没有与 `device_node` 属性相关联，则授予 udev 适当的 `relabelto` 权限。

1.  如果 udev 的规则配置为创建符号链接，则需要确保链接的标签保持为通用的 `device_t` 类型。

## 它是如何工作的……

udev 应用程序是一个标准的 SELinux 感知应用程序，它通过查询上下文定义与 SELinux 用户空间交互，或者通过查询的上下文创建新的设备文件，或在之后重新标记设备文件。

通过查询上下文定义（而不是依赖 SELinux 策略），管理员可以轻松修改不同设备名称的规则，或者为新设备类型提供支持，而无需增强与 `udev_t` 相关的策略。管理员只需要配置适当的文件上下文定义：

```
~# semanage fcontext -a -t mydevice_t -f -c /dev/mydevice[0-9]*

```

然而，如果目标设备类型（`mydevice_t`）没有与 `device_node` 属性关联，那么 `udev_t` 将无法重新标记该设备类型。这个属性对于 `udev_t` 的支持至关重要，因为它通过该属性对所有设备节点具有重新标记（和管理）权限。

如果一个 udev 规则要求创建一个未与 `device_node` 属性关联的设备文件（或者是其他文件——请求的文件不一定是设备文件），那么如果默认的上下文关联（即通过父目录的类型继承）不足以满足要求，则需要更新 SELinux 策略。

出于同样的原因，符号链接必须保持为 `device_t`，因为 SELinux 策略不处理符号链接的不同类型。

当然，udev 中的 SELinux 支持也有其影响，尤其是在设备文件在 udev 管理之外创建时。如果是这种情况，管理员必须确保文件的标签被正确修正，因为错误的设备类型可能导致系统故障。

一个常见的方法是重新标记整个 `/dev/` 结构（这通常是通过分发版的 `init` 脚本在初始 RAM 文件系统或 `devtmpfs` 挂载中对默认设备文件创建及其默认的 `device_t` 类型进行处理）：

```
~# restorecon -R /dev

```

# 使用 cron 与 SELinux

另一个 SELinux 感知应用程序的例子是 cron。实际上是多个 cron 实现，因为并不存在单一的 cron 应用程序。常见的 cron 实现包括 vixie-cron、cronie 和 fcron。

cron 实现调用命令时，会以特定的 Linux 用户身份执行这些命令。由于这些命令并不是一成不变的（cron 的主要目的就是允许为特定用户甚至系统本身执行任何命令），因此无法轻易创建出足够细粒度的策略来支持 cron 提供的所有功能。毕竟，对于 SELinux 本身来说，cron 为某个用户或另一个用户调用命令并没有区别：涉及的只是 cron 域（`crond_t`）和命令的目标类型（例如 `bin_t`）。

由于这个原因，许多 cron 实现都已支持 SELinux，使得 cron 实现能够选择正确的目标上下文。

## 如何操作…

为了正确与支持 SELinux 的 cron 交互，必须按照以下步骤操作：

1.  确保 crontab 文件正确标记：用户 crontab 使用 `user_cron_spool_t`，系统 crontab 使用 `system_cron_spool_t`。

1.  检查 `/etc/selinux/mcs/contexts/default_contexts` 或 `/etc/selinux/mcs/contexts/users/*` 以获取 `system_r:crond_t` 域的目标上下文。

1.  让 crontab 文件上下文作为目标域的入口点。例如，如果用户的目标域是其自身的用户域（如 `user_t`），则 `user_cron_spool_t` 必须作为 `user_t` 的入口点。

1.  如果用户作业的目标域是用户域，则将 `cron_userdomain_transition` 布尔值设置为 `on`；如果目标域应为 `cronjob_t` 域，则将其设置为 `off`。

## 它是如何工作的…

当 cron 支持 SELinux 时，确保它运行在 `crond_t` 域内至关重要。它的内部 SELinux 代码将查询 SELinux 策略，以查看通过应用程序的用户的目标域是什么，如果 cron 没有在 `crond_t` 域中运行，则此查询将无法返回正确的域集：

```
~# ps -efZ | grep fcron | awk '{print $1}'
system_u:system_r:crond_t:s0-s0:c0.c1023

```

在从 cron 启动用户作业之前，cron 应用程序会检查用户 crontab 文件的文件上下文。然后使用这个文件上下文来查看用户作业的目标域是否将用户 crontab 文件上下文作为入口点。

要了解当前的目标域是什么，我们可以使用 `getseuser` 辅助应用程序：

```
~# getseuser hannah system_u:system_r:crond_t:s0
seuser: user_u
Context 0    user_u:user_r:cronjob_t:s0

```

在这种情况下，目标域是 `cronjob_t`。可以通过 `default_contexts`（或用户特定上下文）文件来确认这一点：

```
~# grep crond_t /etc/selinux/mcs/contexts/users/user_u
system_r:crond_t  user_r:cronjob_t

```

如果目标域应为用户域，则需要切换正确的布尔值并相应地调整上下文文件：

```
~# setsebool cron_userdomain_transition on
~# grep crond_t /etc/selinux/mcs/contexts/users/user_u
system_r:crond_t  user_r:user_t

```

知道目标域后，最后需要的就是确保用户 cron 作业文件上下文作为该域的入口点，这通常是大多数 cron 实现会检查的一种访问控制：

```
~# sesearch -s user_t -t user_cron_spool_t -c file -p entrypoint -A
Found 1 semantic av rules:
 allow user_t user_cron_spool_t : file entrypoint ;

```

## 还有更多…

并非所有的 cron 实现都支持 SELinux。如果实现不支持 SELinux，则所有 cron 作业将都在一个单一的 cron 作业容器中运行（用户 cron 作业使用 `cronjob_t`，系统 cron 作业使用 `system_cronjob_t`），并且使用 `system_u` SELinux 用户和 `system_r` SELinux 角色。

# 通过编程方式检查 SELinux 状态

如果需要创建一个支持 SELinux 的应用程序，可以使用多种编程语言。`libselinux` 包通常提供多个编程和脚本语言的绑定。在接下来的示例中，将使用 C 编程语言作为示例实现。

支持 SELinux 的第一步是检查 SELinux 的状态。在这个示例中，我们将展示如何创建一个与 `libselinux` 库链接并检查 SELinux 状态的应用程序。

## 准备工作

由于我们要更新一个 C 应用程序，这一系列步骤假定你具有基本的 C 编程知识。本书下载包中可以找到一个示例 C 应用程序，使用了此（以及其他）食谱中的所有输入。

## 如何操作…

为了链接`libselinux`并检查当前的 SELinux 状态，可以使用以下步骤：

1.  创建一个 C 应用程序代码文件，并通过编译器指令引用 SELinux 头文件：

    ```
    #ifdef SELINUX
    #include <selinux/selinux.h>
    #include <selinux/av_permissions.h>
    #include <selinux/get_context_list.h>
    #endif
    ```

1.  在应用程序中，如果不需要内建 SELinux 支持（即编译器指令未设置时），则相关的 SELinux 函数调用应返回`success`：

    ```
    int selinux_prepare_fork(char * name) {
    #ifndef SELINUX
      return 0;
    #else
      …
    #endif
    };
    ```

1.  在 SELinux 函数内部，使用`is_selinux_enabled()`函数调用检查 SELinux 是否启用：

    ```
    int rc;
    rc = is_selinux_enabled();
    if (rc == 0) {
      … // SELinux is not enabled
    } else if (rc == -1) {
      … // Could not check SELinux state (call failed)
    } else {
      … // SELinux is enabled
    };
    ```

1.  添加一个检查，看看 SELinux 是处于宽容模式还是强制模式。当然，只有在启用 SELinux 时，才需要进行此检查：

    ```
    rc = security_getenforce();
    if (rc == 0) {
      … // SELinux is in permissive mode
    } else if (rc == 1) {
      … // SELinux is in enforcing mode
    } else {
      … // Failed to query state
    };
    ```

1.  在与`libselinux`链接时构建应用程序：

    ```
    ~# gcc -o test -DSELINUX -lselinux test.c

    ```

## 它是如何工作的…

`libselinux`库提供了所有必要的功能，以便应用程序查询 SELinux 并与 SELinux 子系统进行交互。当然，在开发应用程序时，仍然需要确保 SELinux 支持是一个编译时的可选项：并非所有 Linux 系统都启用了 SELinux，因此，如果应用程序默认与`libselinux`链接，那么所有目标系统都需要安装必要的依赖项。

但即便是与`libselinux`链接的应用程序，也必须能够支持 SELinux 已禁用的系统；因此，需要使用`is_selinux_enabled()`来检查 SELinux 状态。

然而，这个`is_selinux_enabled()`函数并不会返回其他信息（例如，加载了哪个策略）。要检查 SELinux 是否在宽容模式下运行，可以使用`security_getenforce()`进行调用。

一个定义良好的应用程序应当使用此状态来调整其行为：如果应用程序运行在宽容模式下，则应尽量避免在应用逻辑中强制执行 SELinux 策略相关的决策。

参考之前食谱中的 cron 示例：如果 crontab 文件的上下文未知，无法作为所选域的入口点，则应用程序应该记录这一点，但仍继续工作（因为模式设置为宽容模式）。遗憾的是，大多数支持 SELinux 的应用程序并未根据 SELinux 的宽容状态改变其行为，仍可能像在强制模式下运行一样失败（或执行不同的逻辑）。

## 还有更多…

还有其他类似的方法可以用来查询 SELinux 状态。

例如，`is_selinux_mls_enabled()`方法返回一个值，指示 SELinux 是否在 MLS 模式下运行。这很有用，因为某些与上下文相关的方法在启用 MLS 时需要级别信息，因此可能需要查询状态并根据 MLS 状态更改方法调用。

与`security_getenforce()`类似的函数是`security_setenforce()`。从名称可以推测，这允许应用程序切换 SELinux 的强制模式。当然，只有当应用程序所在的域具备适当的 SELinux 权限时，这才是可能的。

# 在 C 中查询 SELinux 用户空间配置

在本示例中，我们将查询 SELinux 用户空间，以根据当前进程的上下文为给定用户获取默认上下文。进程负责事先收集该用户的 Linux 用户名。

## 如何实现…

按如下方式查询 SELinux 配置：

1.  获取进程的当前上下文：

    ```
    char * curcon = 0;
    rc = getcon(&curcon);
    if (rc) {
      … // Getting context failed
      if (permissive) {
        … // Continue with the application logic, ignoring SELinux stuff
      } else {
        … // Log failure and stop application logic
      };
    };
    ```

1.  获取 Linux 用户名（假设存储在`name`变量中），并获取 SELinux 用户：

    ```
    char * sename = 0;
    char * selevel = 0;
    rc = getseuserbyname(name, &sename, &selevel);
    if (rc) {
      … // Call failed. Again check permissive state
      … // and take appropriate action.
      freecon(curcon);
    };
    ```

1.  现在，根据获取到的 SELinux 用户（`sename`）和当前上下文（该方法通过`NULL`变量自身处理）获取默认上下文：

    ```
    char * newcon = 0;
    rc = get_default_context(sename, NULL, &newcon);
    if (rc) {
      … // Call failed. Again check permissive state
      … // and take appropriate action.
      freecon(curcon);
    };
    ```

## 它的工作原理…

在第一个代码块中，通过`getcon()`方法获取当前进程的上下文。对于这个示例的最终结果，显式地获取当前上下文并非必须——稍后调用的`get_default_context()`方法会根据当前上下文做出决定（通过第二个参数，这在本示例中是`NULL`）。然而，知道当前上下文对于日志记录以及查询 SELinux 策略本身来说是非常重要的（就像我们将在下一个示例中所做的那样）。

下一步是根据给定的 Linux 用户获取 SELinux 用户。`sename`（SELinux 用户）和`selevel`（SELinux 敏感度）变量由`getseuserbyname()`方法填充，传入 Linux 用户名（它是一个常规的`char *`变量）。

最后，当 SELinux 用户可用时，调用`get_default_context()`获取默认上下文，并将其存储到第三个参数（`newcon`）中。如果需要从与当前上下文不同的上下文中获取默认上下文，那么第二个参数就不应是`NULL`，而应该是要查询的上下文：

```
rc = get_default_context(sename, curcon, &newcon);
```

## 还有更多…

在 SELinux 感知的应用程序中，其他一些方法可能也很有趣。

比如，`getprevcon()`方法返回的是进程的先前上下文，而不是当前上下文。这个先前的上下文通常是父进程的上下文，虽然对于可以执行动态转换的应用程序来说，这也可能是当前进程的先前上下文。

这些信息也可以从`/proc/`文件系统中获取，在进程的`attr/`子目录下可以查看`current`和`prev`文件：

```
~$ id -Z
staff_u:staff_r:staff_t:s0
~$ newrole -r sysadm_r
Password: 
~$ id -Z
staff_u:sysadm_r:sysadm_t:s0
~$ cat /proc/$$/attr/current
staff_u:sysadm_r:sysadm_t:s0
~$ cat /proc/$$/attr/prev
staff_u:staff_r:newrole_t:s0

```

如上所示，运行`newrole`切换角色后，进程所在的最后一个域是`newrole_t`域（然后执行域和角色的转换，切换到当前的上下文）。

允许执行动态转换（即不需要启动新命令）的应用程序，可以使用`setcon()`方法从当前上下文切换到新上下文。

`get_default_context()`方法也是一组更大方法家族的一部分。例如，当用户分配了多个角色时，特定转换可能允许多个上下文。`get_ordered_context_list()`方法返回支持的上下文列表（而`get_default_context()`方法只返回第一个）。通过使用`get_ordered_context_list_with_role()`方法，可以通过角色过滤出特定上下文。

在启用 MLS 的系统上，`get_default_context_with_level()`或`get_default_context_with_rolelevel()`将对结果上下文应用指定的级别。

另一个可用的方法是`get_default_type()`方法，它返回给定角色的默认类型。与其他方法一样，这会导致 SELinux 代码查询位于`/etc/selinux/`中的配置文件；在此特定情况下，是位于`/etc/selinux/mcs/contexts/`中的`default_type`文件。

# 从代码角度审查 SELinux 子系统

为了查询 SELinux 策略，我们已经看到了如何使用`sesearch`命令和其他 SELinux 工具。从代码角度来看，SELinux 策略可以通过`security_compute_av_flags`方法查询。

## 准备工作

`curcon`和`newcon`变量可以通过诸如`getcon()`（获取当前上下文）或`get_default_context()`（如我们在前面的示例中所见）等方法填充。

## 如何实现…

作为示例，我们希望查询两个进程域之间的转换权限。为此，使用以下方法：

1.  首先，调用`security_compute_av_flags()`方法：

    ```
    struct av_decision avd;
    rc = security_compute_av_flags(curcon, newcon, SECCLASS_PROCESS, PROCESS__TRANSITION, &avd);
    if (rc) {
      … // Method failed.
      freecon(curcon);
      freecon(newcon);
    };
    ```

1.  现在阅读响应：

    ```
    if (!(avd.allowed & PROCESS__TRANSITION)) {
      … // Transition is denied
    };
    ```

1.  检查当前上下文是否为宽容域：

    ```
    if (avd.flags & SELINUX_AVD_FLAGS_PERMISSIVE) {
      … // Domain is permissive
    };
    ```

## 它是如何工作的…

`security_compute_av_flags()`方法是 C 语言中与`sesearch`（粗略来说）等效的方法。它接受源和目标上下文、类别和权限，并将查询结果存储在一个特定的结构体中（`struct av_decision`）。

类别和权限条目可以从`flask.h`（类别声明）和`av_permissions.h`（权限声明）头文件中获取，这些头文件位于`/usr/include/selinux/`目录下。

查询结果是通过检查权限是否出现在决策结果中来获得的。

除了权限查询，另一个需要验证的重要方面（而这一点通常会被 SELinux 意识到的应用忽略）是检查域本身是否标记为宽容模式。毕竟，即使在启用了 SELinux 并且 SELinux 处于强制模式的系统上，某些域仍然可以标记为宽容模式。

`SELINUX_AVD_FLAGS_PERMISSIVE`标志是添加到查询响应中的标志（`struct av_decision`），它允许开发人员查询域的宽容状态。掌握这些信息后，SELinux-aware 应用程序仍然可以决定继续进行，即使策略拒绝某些活动，就像用户请求的那样。

## 还有更多…

还有其他方法可以查询 SELinux 策略，这些方法可能会被支持 SELinux 的应用程序使用。

例如，使用`selinux_check_access()`，应用程序可以查询 SELinux 策略，查看给定的源上下文是否对目标上下文的特定类和权限具有访问权限。这与`security_compute_av_flags()`不同，因为该方法使用类和权限的字符串，并且根据 SELinux 的强制执行状态或特定域的宽容性返回不同的结果。

# 在新上下文中运行新进程

有时候，在调用新任务或进程时，无法强制指定特定的域。只有在源域和文件上下文（应用程序或任务执行的上下文）对目标上下文的决定性非常明确时，才能通过 SELinux 策略启用默认的过渡规则。

在可以为不同目标域运行相同命令（或以相同上下文执行命令）的应用程序中，支持 SELinux 至关重要。

本示例将展示如何强制为新进程指定特定域。

## 准备就绪

本示例中使用的`newcon`变量可以通过像`get_default_context()`这样的方式来填充，正如我们在之前的示例中看到的那样。

## 它是如何做的…

要在特定上下文中启动进程，请执行以下步骤：

1.  告诉 SELinux 新上下文应该是什么：

    ```
    int rc = setexeccon(newcon);
    if (rc) {
      … // Call failed
      freecon(newcon);
    };
    ```

1.  分叉并执行命令。例如，要执行`id -Z`，可以使用以下代码：

    ```
    pid_t child;
    child = fork();
    if (child < 0) {
      … // Fork failed } else if (child == 0) {
      int pidrc;
      pidrc = execl("/usr/bin/id", "id", "-Z", NULL);
      if (pidrc != 0) {
        … // Command failed
      };
    } else {
      … // Parent process
      int status;
      wait(&status);
    };
    ```

## 它是如何工作的…

希望新执行任务在特定上下文中运行的应用程序，需要告知 SELinux 子系统，下一个`execve`、`execl`或其他`exec*`方法应该导致子进程在新域中运行。

当然，尽管政策中不再需要自动域过渡（因为这需要明确的决定，而如果源域和文件上下文对不同目标上下文相同时，这是不可能的），但 SELinux 策略仍然必须允许过渡策略。

```
allow crond_t self : process setexec;
allow crond_t staff_t : process transition;
```

`setexec`权限允许源域明确告诉 SELinux 子系统任务应该在哪个上下文中运行。没有此权限，`setexeccon()`调用将失败。

## 还有更多…

`setexeccon()`方法有一个类似的方法叫做`getexeccon()`。该方法返回执行新进程时将被分配的上下文（这将验证上一次`setexeccon()`调用的结果）。

另一个类似的方法是`setexecfilecon()`方法。该方法允许支持 SELinux 的应用程序在处理基于文件的过渡信息时，考虑 SELinux 策略的决策。因此，如果执行特定文件时已知的域过渡存在，则会遵循该域过渡。如果没有，则使用`setexecfilecon()`方法提供的回退类型：

```
char * fallbackcon = "system_u:object_r:openscap_helper_script_t:s0";
char * filename = "/usr/libexec/openscap/probe_process";
…rc = setexecfilecon(filename, fallbackcon);
```

在此示例中，如果`probe_process`文件的上下文在 SELinux 策略中被用来在当前应用程序调用时创建自动域转换，则该目标域将用于应用程序执行。然而，如果`probe_process`文件的上下文是不会触发任何自动域转换的，那么`fallbackcon`上下文将用于下一个应用程序执行。

# 读取资源的上下文

当然，如果应用程序是 SELinux 感知的，获取资源的上下文也很重要。这可能是为了日志记录，或决定要转换到哪个域（基于资源上下文、当前上下文、用户名等）。

## 如何执行...

要读取资源的上下文，可以使用以下方法：

1.  给定一个文件路径，以下调用`getfilecon()`将提供文件的上下文：

    ```
    security_context_t filecon = 0;
    char * path = "/etc/passwd";
    rc = getfilecon(path, &filecon);
    if (rc < 0) {
      … // Call failed
    };
    … // Do stuff with the context
    freecon(filecon);
    ```

1.  要获取一个进程的上下文，假设`pid`变量（类型为`pid_t`）已经包含正确的进程 ID，使用以下代码：

    ```
    security_context_t pidcon = 0;
    rc = getpidcon(pid, &pidcon);
    if (rc < 0) {
      … // Call failed
    };
    … // Do stuff with the context
    freecon(pidcon);
    ```

## 它是如何工作的...

SELinux 库提供了多种方法来获取资源的上下文。文件和进程类型在示例中有展示，但也有其他方法。例如，使用`fgetfilecon()`方法，可以获取文件描述符的上下文。所有这些方法都会以标准字符串（`char *`）格式返回上下文。

在获取资源的上下文后，当不再使用该上下文时，释放它非常重要。否则，应用程序将发生内存泄漏，因为没有其他方法清理上下文。

## 还有更多...

当使用标记化网络（例如，支持 CIPSO/NetLabel 或标记化 IPSec）时，可以使用`getpeercon()`方法获取参与通信会话的对等方的上下文。

在查询上下文的同时，也可以告诉 SELinux 子系统文件创建应该立即使用特定上下文创建该文件。为此，可以使用`setfscreatecon()`方法——这是近期 udev 版本在`/dev/`中创建新设备文件时使用的方法。
