# 第二章 处理文件标签

在本章中，我们将讨论如何设置和管理文件标签，并学习如何自己配置 SELinux 策略以使用和分配正确的文件标签。本章所涵盖的内容如下：

+   通过模式定义文件上下文

+   使用替代定义

+   通过文件转换增强 SELinux 策略

+   设置资源敏感性标签

+   配置敏感性类别

# 介绍

设置、重置和管理文件标签是管理员在启用 SELinux 的系统上最常执行的任务。策略开发者和 Linux 发行版提供了合理的默认设置，但许多实现可能有不同的服务和文件位置。公司通常会将自定义脚本和日志文件安装在非默认位置，许多守护进程可以配置为在同一系统上支持多个实例——每个实例使用不同的基础目录。

系统管理员将了解如何通过`semanage`应用程序设置上下文定义，然后使用`setfiles`或`restorecon`重置目标文件的上下文：

```
~# semanage fcontext –a –t httpd_sys_content_t "/srv/web/zone/htdocs(/.*)?"
~# restorecon –RF /srv/web/zone/htdocs

```

然而，这是一个本地定义，如果需要，必须导出并导入，以便将其传输到其他系统：

```
~# semanage export -f local_selinux.mods
~# semanage import -f local_selinux.mods

```

通过将上下文定义移入 SELinux 策略领域，这些定义可以轻松地在多个系统上安装并集中管理，正如我们在 SELinux 策略模块中所看到的那样。

# 通过模式定义文件上下文

SELinux 策略模块可以通过其`.fc`文件包含文件上下文定义。在这些文件中，路径表达式用于指向应该匹配特定文件上下文的各种位置，类标识符用于根据文件类别（目录、常规文件、符号链接等）区分文件上下文定义。

在这个教程中，我们将创建一个`mylogging` SELinux 模块，定义与日志相关的上下文的额外路径规范。我们将使用直接的文件路径以及正则表达式，并查看各种类标识符。

## 如何操作……

要通过 SELinux 策略模块定义文件上下文，请使用以下方法：

1.  使用`matchpathcon`，我们可以检查 SELinux 工具将资源重置为的上下文：

    ```
    ~# matchpathcon /service/log
    /service/log  system_u:object_r:default_t

    ```

1.  创建`mylogging.te`文件，在其中提及将用于定义的类型。最佳实践是通过不同的 SELinux 模块处理 SELinux 模块本身未定义的类型。然而，在此示例中，我们也声明了`var_t`以保持示例简单：

    ```
    policy_module(mylogging, 0.2)
    gen_require(`
      type var_t;
      type var_log_t;
      type auditd_log_t;
    ')
    ```

1.  接下来，创建`mylogging.fc`文件，在其中声明路径表达式及其关联的文件上下文：

    ```
    /service(/.*)?    gen_context(system_u:object_r:var_t,s0)
    /service/log(/.*)?    gen_context(system_u:object_r:var_log_t,s0)
    /service/log/audit(/.*)?    gen_context(system_u:object_r:auditd_log_t,s0)
    /lxc/.*/log  -d  gen_context(system_u:object_r:var_log_t,s0)
    /var/opt/oracle/listener\.log  --  gen_context(system_u:object_r:var_log_t,s0)
    ```

1.  现在，构建策略模块并加载它：

    ```
    ~$ make mylogging.pp
    ~$ semodule –i mylogging.pp

    ```

1.  使用`matchpathcon`，我们现在可以验证 SELinux 工具已知的上下文是否正确：

    ```
    ~# matchpathcon /service/log
    /service/log  system_u:object_r:var_log_t

    ```

## 它是如何工作的……

SELinux 策略模块包含 SELinux 正确处理一组策略规则所需的所有内容。包括规则本身（在 `.te` 文件中声明），以及可选的接口声明（在 `.if` 文件中），这些接口声明定义了其他策略可以调用的接口，以便生成特定的 SELinux 规则。SELinux 策略模块的第三部分和最后一部分是相关的文件上下文文件——因此是 `.fc` 文件后缀。

### 注意

`.fc` 文件中的上下文声明不会自动强制执行并设置这些上下文。这些仅仅是 SELinux 工具和库在重新标记操作发生时使用的定义。

该上下文文件包含每一行的内容：

+   一个路径表达式，应该匹配到一个绝对文件路径

+   一个可选的类标识符，用于区分上下文（文件、目录、套接字、符号链接等）

+   要分配给此路径的上下文

上下文定义的每一部分都以空格分隔：

```
<path>  [<class identifier>]  <context>
```

这些行可以按照政策开发人员的喜好进行排序。大多数开发人员将路径按字母顺序排列，并根据顶级目录进行分组。

### 路径表达式

SELinux 工具和库中的正则表达式支持基于 **Perl 兼容正则表达式**（**PCRE**）。

在所有可能的表达式中，最简单的表达式是没有通配符的表达式，例如以下代码：

```
/var/opt/oracle/listener\.log
```

其中一个重要部分是对句点的转义——如果我们不转义句点，那么 PCRE 支持会将句点视为任何字符，这样不仅匹配到 `listener.log` 文件，还可能匹配 `listener_log` 或 `listenerslog`。

一个非常常见的表达式是匹配特定目录及其所有子目录和文件，这在以下示例中表示：

```
/service(/.*)?
```

这确保了每个文件或目录都有一个上下文定义。

### 处理顺序

给定常规系统中路径表达式的详尽列表，一个文件路径可以匹配多个规则，那么 SELinux 工具将使用哪个规则呢？

基本上，SELinux 工具遵循 *最具体优先* 的原则。给定 A 行和 B 行，将按以下顺序检查，其中第一个匹配的获胜：

1.  如果 A 行中有正则表达式而 B 行没有，那么 B 行更为具体。

1.  如果 A 行中第一个正则表达式之前的字符数少于 B 行中第一个正则表达式之前的字符数，则 B 行更为具体。

1.  如果 A 行的字符数少于 B 行的字符数，则 B 行更为具体。

1.  如果 A 行没有指定 SELinux 类型（因此它的上下文部分是 `<<none>>`），而 B 行有，那么 B 行更为具体。

SELinux 工具将加载通过 `/etc/selinux/mcs/contexts/files/` 中的文件给出的定义，但会优先考虑 `file_contexts.local`（然后是 `file_contexts.homedirs`）中的定义，因为这些是由系统管理员本地创建的定义。然而，如果本地定义使用正则表达式而策略提供的定义没有使用，则仍然使用策略提供的定义。这是各上下文文件之间优先规则的唯一例外。

SELinux 工具提供了一个名为 `findcon` 的工具（属于 `setools` 或 `setools-console` 包），可用于分析这个排序，显示单个（!）上下文定义文件中的匹配模式，并按照从最不具体到最具体的顺序排列：

```
~$ findcon /etc/selinux/mcs/contexts/files/file_contexts -p /var/log/aide
/.*    system_u:object_r:default_t:s0
/var/.*    system_u:object_r:var_t:s0
/var/log/.*  system_u:object_r:var_log_t:s0
/var/log/aide(/.*)?  system_u:object_r:aide_log_t:s0

```

如果只需要实际的上下文定义（而不是 `findcon` 所显示的完整匹配表达式及其优先顺序），则可以改用 `matchpathcon`：

```
~# matchpathcon /var/log/aide
/var/log/aide  system_u:object_r:aide_log_t:s0

```

### 类标识符

上下文定义的第二部分是可选部分——类标识符。通过类标识符，开发者可以告诉 SELinux 工具，只有当路径表达式匹配特定文件类时，上下文定义才适用。如果省略类标识符，则任何类都会匹配。

如果显示了类标识符，则可以使用以下标识符之一（每行一个）：

+   '`--`' 标识符用于常规文件

+   '`-d`' 标识符用于目录

+   '`-l`' 标识符用于符号链接

+   '`-b`' 标识符用于块设备

+   '`-c`' 标识符用于字符设备

+   '`-p`' 标识符用于 FIFO 文件

+   '`-s`' 标识符用于套接字

### 上下文声明

上下文定义的最后部分是要分配给所有匹配资源的上下文。它是通过 `gen_context` 宏生成的，如下所示：

```
gen_context(system_u:object_r:var_t,s0)
```

`gen_context` 宏用于基于策略特性区分上下文定义。如果目标策略不支持 MLS，则仅使用第一个参数（示例中的 `system_u:object_r:var_t`）。如果策略支持 MLS，但仅支持单一敏感性（`s0`），则会在上下文中附加 `:s0`。否则，会附加第二个参数（示例中恰好也是 `s0`，前面带冒号）。

通常，上下文只在 SELinux 类型上有所不同。资源的 SELinux 所有者和 SELinux 角色通常分别保持为 `system_u` 和 `object_r`。

上下文的特殊值是 `<<none>>`，如下所示：

```
/proc  -d  <<none>>
```

这告诉 SELinux 工具，它们永远不应尝试设置此资源的上下文。每当管理员触发文件系统重新标记操作时，这些特定位置的标签将不会被更改（无论当前标签是什么）。这*并不*意味着应该移除现有上下文！

## 还有更多...

在本教程中，我们详细介绍了如何定义标签。如果进行大量更改，强制对整个系统进行重新标记是有意义的。在 Red Hat 系统中，可以通过创建一个标志文件并重启系统来实现：

```
~# touch /.autorelabel
~# reboot

```

在 Gentoo 系统中，可以使用`rlpkg`命令重新标记整个系统：

```
~# rlpkg -a -r

```

在 Red Hat 系统中，用于重新标记系统的命令是`fixfiles`：

```
~# fixfiles relabel

```

如果系统（临时）以没有 SELinux 支持或禁用 SELinux 的状态启动，文件将会被创建为没有文件上下文。当 SELinux 启用的系统再次启动时，它将把这些文件标记为`unlabeled_t`，这是大多数域无法访问的类型（在 SELinux 方面）。

# 使用替代定义

有时，应用程序及其资源被安装在与 SELinux 策略预期的位置不同的地方。试图通过为每个子目录定义额外的上下文定义来适应这种情况，容易变得无法管理。

为了帮助管理员，SELinux 工具支持替代条目，这些条目告诉 SELinux：“如果路径以*这个*开头，那么就像它以*那个*开头一样标记它”。管理员可以使用`semanage`设置这样的替代（称为**等价类**），如下所示：

```
~# semanage fcontext –a –e / /mnt/chroots/bind

```

在这个例子中，`/mnt/chroots/bind/`下的任何位置都会被标记为如果它是从主`/`目录开始的（所以`/mnt/chroots/bind/etc/`变成了`etc_t`，因为`/etc/`是`etc_t`）。

`chroots`的目标位置是一个很好的应用场景。`chroot`是文件系统中的一个替代位置，它将作为一个或一组应用程序的根文件系统。

对于想要在多个系统上设置替代的管理员，无法将其作为 SELinux 策略模块的一部分。我们需要管理的文件称为`file_contexts.subs`（还有一个以`.subs_dist`结尾，由 Linux 发行版管理，我们不会修改它）。话虽如此，我们始终可以找到一种相对合理的方式来更新这个文件。

## 准备工作

最简单的方法是使用一个中央配置管理工具，如 Puppet、CFEngine、Chef 或 Ansible，因为这些系统允许管理员强制将特定文件的内容设置为特定值。使用配置管理工具本身就可以写成一本书，所以它超出了本书的范围。如果你确实想深入研究，记住`file_contexts.subs`文件也是由`semanage`命令管理的。管理员可能希望添加一些中央配置管理工具无法识别的本地定义（因此可能会恢复更改）。

在本教程中，我们将介绍一种通用的方法，但它确实要求能够进行文件传输并执行一条单行命令（以正确的权限执行）。然而，对于大多数系统管理员来说，这不应该是一个挑战。

## 如何操作…

为了将更改应用到广泛的系统，请按照以下步骤操作：

1.  在系统上本地应用更改：

    ```
    ~# semanage fcontext -a -e / /mnt/chroot/bind

    ```

1.  将定义导出到一个文件：

    ```
    ~# semanage export -f local_selinux.mods

    ```

1.  编辑 `local_selinux.mods` 文件，并删除所有与更改无关但需要分发的条目。

1.  将生成的文件分发到目标系统。

1.  在系统上本地应用更改：

    ```
    ~# semanage import -f local_selinux.mods

    ```

## 它是如何工作的…

`semanage fcontext` 命令为 `/mnt/chroot/bind/` 创建了一个等效类，其中所有子目录和文件都会被标记，就像它们位于根目录（`/`）下一样。这确保管理员不需要为他们管理的每个 `chroot` 位置定义大量的文件上下文。

然而，这可能会成为问题，因为 `semanage fcontext` 只会本地应用更改，在大型基础设施中，可能需要将相同的设置应用于多个系统。为此，可以使用 `semanage export` 和 `semanage import`。

`semanage export` 命令的输出是一组 `semanage` 指令，完全遵循 `semanage` 命令的语法。

导出 `semanage` 定义时，首先存储的命令集是 `delete all` 语句，如 `fcontext -D`（删除所有本地创建的 `semanage fcontext` 设置）。当然，如果我们只需要分发替代定义，则删除所有先前的本地语句是不正确的。因此，需要手动编辑 `local_selinux.mods` 文件。如果仅需分发等效类定义，则文件可能仅包含以下内容：

```
fcontext -a -e / /mnt/chroot/bind
```

然后，可以将导出的文件分发到所有目标系统，并通过 `semanage import` 命令加载，从而有效地将相同的一组更改应用到系统。

如果定义已经应用于某个系统，则 `import` 命令将失败：

```
~# semanage import -f local_selinux.mods
ValueError: Equivalence class for /mnt/chroot/bind already exists

```

需要注意的是，如果文件中的某个命令未能应用，则文件中的所有命令都不会应用（文件是一次性处理的）。因此，`delete all` 规则最初被作为导出的命令集的一部分。

如果本地应用的更改也需要保留，那么分布式管理这些设置会变得更加具有挑战性，除非分布式的更改集是单一的（一个导出的指令，允许失败）。

## 还有更多…

`/etc/selinux/mcs/contexts/` 位置中的大多数文件不应通过任何工具进行管理，除非是 Linux 发行版包管理系统（通过安装基础 SELinux 策略）或 `semanage`。

话虽如此，这个位置内的大多数文件变化不大（除了`files/file_contexts`文件）。将这些文件与包管理系统连接以更新它们（如果支持）可能是有益的，或者直接接管这些文件的管理，前提是你密切跟踪分发版会进行的更改。

## 另请参见

以下资源将深入讨论本配方中涉及的主题：

+   要了解更多关于各种配置文件的信息，请查看 [`selinuxproject.org/page/PolicyConfigurationFiles`](http://selinuxproject.org/page/PolicyConfigurationFiles)

+   SELinux 与`chroots`的交互在第九章中讨论得更详细，*对齐 SELinux 与 DAC*

# 通过文件转换增强 SELinux 策略

到目前为止，我们只处理了文件上下文的配置部分：如果我们要求 SELinux 工具重新标记文件，那么我们所做的更改将生效。然而，除非你运行`restorecond`守护进程来检查所有可能的文件修改（这将非常消耗资源），或者定期手动运行`restorecon`对所有文件进行检查，否则新定义的上下文将不会应用到文件上。

我们需要做的是修改本地的 SELinux 策略，以便在创建这些文件时，Linux 内核会自动为这些文件分配正确的标签。这通过文件转换来处理，这是**类型转换**的一个特定情况。

在类型转换中，我们配置策略，使得如果某个域在具有特定标签的目录中创建文件（或其他资源类），则创建的对象应该自动获得一个特定的标签。从策略的角度来看，这是如下所写的：

```
type_transition <domain> <directory_label>:<resource_class> <specific_label>
```

SELinux 还增加了对命名文件转换的支持（从 Linux 2.6.39 版本开始，并在 Gentoo、Fedora 16+、Red Hat Enterprise Linux 7+中可用）。在这种情况下，仅当创建的资源与特定的文件名完全匹配时（即没有正则表达式）才会发生这种转换：

```
type_transition <domain> <directory_label>:<resource_class> <specific_label> <filename>
```

通过参考策略宏，这可以通过`filetrans_pattern`定义来支持。

## 准备工作

为了正确定义文件转换，我们需要知道负责创建资源的源域是什么。例如，`/var/run/snort/`目录可能由`init`脚本创建，但如果没有文件转换，那么该目录将以父目录的类型（即`var_run_t`）创建，而不是正确的类型（`snort_var_run_t`）。

所以在开始本配方之前，请确保写下所有相关的标签（例如，我们将使用`initrc_t`作为`init`脚本的标签，`var_run_t`作为父目录的标签，`snort_var_run_t`作为目标目录的标签）。

## 如何执行…

定义文件转换可以按照以下方式进行：

1.  在 SELinux 策略中搜索，看看是否有一个接口能够提供从给定域到`snort_run_t`的文件转换：

    ```
    ~$ sefindif filetrans.*snort_var_run_t

    ```

1.  假设没有找到相关内容，搜索允许`initrc_t`创建的资源转移到给定类型的接口：

    ```
    ~$ sefindif filetrans.*initrc_t
    system/init.if: interface(`init_daemon_pid_file',`
    system/init.if:   files_pid_filetrans(initrc_t, $1, $2, $3)

    ```

1.  成功！现在，让我们通过以下声明创建一个用于增强 snort SELinux 模块的策略（通过`mysnort`策略文件）：

    ```
    policy_module(mysnort, 0.1)
    gen_require(`
      type snort_t;
      type snort_var_run_t;
    ')
    # If initrc_t creates a directory called "snort" in a var_run_t dir,
    # make sure this one is immediately labeled as snort_var_run_t.
    init_daemon_pid_file(snort_var_run_t, dir, "snort")
    ```

1.  构建新的策略并加载它。然后使用`sesearch`检查是否确实声明了类型转换：

    ```
    ~$ sesearch –s initrc_t –t var_run_t –T | grep "snort"
    type_transition initrc_t var_run_t : dir snort_var_run_t "snort"

    ```

## 它是如何工作的……

支持 SELinux 的 Linux 发行版已经提供了一个在大多数部署中有效的 SELinux 策略。默认策略非常广泛，且大多能够开箱即用。如果需要特定的更改，那么这些特定的 SELinux 规则可能已经被定义（作为策略接口的一部分），只需实例化并加载即可。

策略接口通常存在以下两种类型：

+   主题通过参数传递的接口，其中对象（操作执行的对象）以及可能的目标（在我们的例子中，是转换应发生的目标）是硬编码的

+   主题硬编码且对象、目标或两者是接口参数的接口

在我们的示例中可以使用的第一种接口类型的示例代码如下所示：

```
interface(`snort_generic_pid_filetrans_pid',`
  gen_require(`
    type snort_var_run_t;
  ')
  files_pid_filetrans($1, snort_var_run_t, dir, $2)
')
```

然后我们可以像这样调用这个接口：

```
snort_generic_pid_filetrans_pid(initrc_t, "snort")
```

然而，这种接口会增加维护负担。对于系统中每添加一个守护进程支持，`init`策略就需要通过命名文件转换和新增的守护进程策略规则来进行修改。考虑到系统中可能运行的守护进程数量，`init`策略实际上会被填充大量的命名文件转换——每个守护进程至少一个。

我们在示例中遇到的接口声明要管理得多得多。该接口旨在由守护进程策略本身调用，并立即确保`initrc_t`类型可以在通用运行目录(`var_run_t`)中创建给定类型(`snort_var_run_t`)的目录。政策的新增不会影响`init`策略，使得策略的维护变得更加容易。

### 查找正确的搜索模式

要查找正确的模式，我们使用`sefindif`接口在可用接口中进行搜索。找到正确的表达式是一项经验性的任务。

如我们所知，我们想要查找文件转换，我们要查找的行将包含`filetrans_pattern`。然后，其中一个涉及的参数是我们将要转换到的类型（`snort_var_run_t`）。因此，我们在示例中使用的表达式更改为`filetrans.*snort_var_run_t`。由于没有结果，接下来的搜索涉及必须进行转换的域（`initrc_t`），因此表达式改为`filetrans.*initrc_t`。

然而，假设我们不知道需要搜索`filetrans_pattern`。类型本身（`snort_var_run_t`）或域（`initrc_t`）可能足以进行搜索，如以下的搜索所示：

```
~$ sefindif snort_var_run_t
~$ sefindif initrc_t

```

从结果列表中，我们可以看到是否有适合我们需求的接口。

### 模式

如`filetrans_pattern`这样的模式是参考策略中的重要支持定义。它们将一组与功能性方法相关的权限（如读取文件，这些通过`read_files_pattern`来处理）进行捆绑，并且不依赖于特定的类型（与接口不同）。

模式的需求来源于 SELinux 对 Linux 活动的非常细粒度的访问控制。读取文件是一个很好的例子：仅仅允许一个类型执行`read`操作是不够的：

```
allow initrc_t snort_var_run_t:file read;
```

大多数应用程序首先检查文件的属性（`getattr`）并打开文件，然后才能读取文件。根据目的，它们可能还想锁定文件或通过`ioctl`对其执行 I/O 操作。所以，不仅仅是前述的访问向量，规则已更改为：

```
allow initrc_t snort_var_run_t:file { getattr lock open read ioctl }
```

参考策略为此提供了一个称为`read_file_perms`的单一权限集，将访问向量转换为以下内容：

```
allow initrc_t snort_var_run_t:file read_file_perms;
```

其次，策略开发者通常希望允许一个域在一个标记相似的目录内读取文件。例如，一个`snort_var_run_t`文件可以位于`/var/run/snort/snort.pid`，而`/var/run/snort/`目录也标记为`snort_var_run_t`。因此，我们还需要授予`initrc_t`类型在该目录内的搜索权限——这同样是一组权限，可以从`search_dir_perms`定义中看到：

```
~$ seshowdef search_dir_perms
define(`search_dir_perms',`{ getattr search open }')

```

与其为此创建多个规则，不如创建一个模式，叫做`read_files_pattern`，它如下所示：

```
~$ seshowdef read_files_pattern
define(`read_files_pattern',`
 allow $1 $2:dir search_dir_perms;
 allow $1 $3:file read_file_perms;
')

```

这允许策略开发者使用一个单一的调用：

```
read_files_pattern(initrc_t, snort_var_run_t, snort_var_run_t)
```

要查看支持策略开发的各种模式，请使用`sefinddef`并带上`'define.*_pattern`'表达式：

```
~$ sefinddef define.*_pattern

```

使用模式允许开发者通过函数化方法创建可读的策略规则，而不是对每个单独的访问向量进行总结。

## 还有更多...

在之前展示的`snort_generic_pid_filetrans_pid`接口中，我们使用了命名文件转换：只有当传递的最后一个参数中的文件名与创建的文件的文件名匹配时，转换才会发生。

命名文件转换优先于普通文件转换。一个很好的例子是`initrc_t`域支持的文件转换：

```
~# semanage –s initrc_t –t var_run_t –T
Found 2 semantic te rules:
 type_transition initrc_t var_run_t : file initrc_var_run_t;
 type_transition initrc_t var_run_t : dir initrc_var_run_t;
Found 16 named file transition rules:
type_transition initrc_t var_run_t : dir udev_var_run_t "udev";
type_transition initrc_t var_run_t : dir tor_var_run_t "tor";
…

```

在这种情况下，如果一个`init`脚本创建了一个名为`udev`或`tor`的目录（或示例中未显示的任何其他转换规则），则会发生适当的文件转换。如果文件名不匹配，则会发生转换到`initrc_var_run_t`类型。

对常规文件和目录的文件转换最为常见，但转换也可以发生在其他各种类别上，如套接字、FIFO 文件、符号链接等。

## 另见

+   域转换（即为进程分配不同的上下文而非文件）将在第三章，*限制 Web 应用程序*中详细讨论，并在第四章，*创建桌面应用程序策略*和第五章，*创建服务器策略*中使用。

# 设置资源敏感度标签

当 SELinux 策略启用 MLS 并支持多个敏感度时（而 MCS 则没有，因为 MCS 只有一个敏感度），SELinux 可以根据域的许可级别和资源的敏感度级别管理信息流和资源访问。但是即使只有单一敏感度（如 MCS 的情况），SELinux 也提供了额外的约束支持，确保域无法访问分配了该域没有许可的类别的资源。

敏感度级别由一个敏感度（`s0`通常用于最低敏感度，而`s15`——这是一个策略构建时的常量，因此可以配置——是最高敏感度）以及一个类别集（可以是诸如`c0,c5,c8.c10`之类的列表）组成。

安全许可类似于敏感度级别，但显示的是一个敏感度范围（例如`s0-s3`），而不是单一的敏感度级别。安全许可可以看作是从最低敏感度级别到域允许的最高敏感度级别的范围。

在为此类系统开发策略时，上下文定义和策略规则可以考虑敏感度。在本教程中，我们将执行两个针对启用 MLS 系统最常见的操作：

+   定义一个更高敏感度的上下文

+   在域转换中按策略设置进程的许可级别

为了实现这一点，我们将以 snort 入侵检测系统为例，强制它始终以`s3`敏感度和所有可能的类别运行。

本示例还将向我们展示如何替换现有策略，而不是增强它，因为我们将更新一个定义，否则它将与现有定义发生冲突。

## 如何做到这一点…

要修改现有域以支持特定的敏感度级别，请执行以下步骤：

1.  将`snort.te`和`snort.fc`文件从分发策略库复制到本地环境：

    ```
    ~$ cp ${POLICY_LOCATION}/policy/modules/contrib/snort.* ${DEVROOT}/local

    ```

1.  将文件重命名为`mysnort`（或`customsnort`），这样我们就知道这是一个定制的策略。不要忘记更新`.te`文件中的`policy_module`调用。

1.  打开`mysnort.te`文件，查找`init_daemon_domain`调用。将该调用替换为以下内容：

    ```
    init_ranged_daemon_domain(snort_t, snort_exec_t,  s3:mcs_allcats)
    ```

1.  在`mysnort.fc`中，将 snort 资源标记为`s3`敏感度。例如，对于 snort 二进制文件，标记如下：

    ```
    /usr/bin/snort  --  gen_context(system_u:object_r:snort_exec_t,s3)
    ```

1.  构建 `mysnort` 策略，移除当前加载的 snort SELinux 策略模块，并加载 `mysnort` 模块：

    ```
    ~# /etc/init.d/snort stop
    ~# semodule –r snort
    ~# semodule –i mysnort.pp

    ```

1.  重新标记所有与 snort 相关的文件，然后再次启动 snort。

## 它是如何工作的……

这个方案有三个重要方面：

1.  我们替换整个策略，而不是创建一个增强。

1.  我们更新了策略以使用范围守护进程域。

1.  我们更新文件上下文，以使用正确的敏感度。

文件上下文的更新显而易见，但完全替换策略的原因可能不太明显。

### 完整策略替换

在这个示例中，我们复制了现有的 snort SELinux 模块策略，并在副本中进行了更新，而不是通过创建额外的模块来增强策略。

这是必需的，因为我们正在对 SELinux 策略进行更改，这些更改与当前运行的 SELinux 策略是互斥的。例如，文件上下文的更改会让 SELinux 感到困惑，因为它会通过策略模块有两个完全匹配的定义，但每个定义的上下文不同。

在这个示例中，我们仅复制了类型强制声明（`snort.te`）和文件上下文声明（`snort.fc`）。如果我们还复制接口定义（`snort.if`），那么策略构建时会给出警告，提示存在重复的接口定义——毕竟，Linux 发行版提供的接口定义仍然存在于系统中。

### 范围守护进程域

在 SELinux 策略本身中，我们将 `init_daemon_domain(snort_t, snort_exec_t)` 条目替换为以下内容：

```
init_ranged_daemon_domain(snort_t, snort_exec_t, s3:mcs_allcats)
```

让我们来看看这个接口的内容：

```
~$ seshowif init_ranged_daemon_domain
interface(`init_ranged_daemon_domain',`
 gen_require(`
 type initrc_t;
 ')
 init_daemon_domain($1, $2)
 ifdef(`enable_mcs',`
 range_transition initrc_t $2:process $3;
 ')
 ifdef(`enable_mls',`
 range_transition initrc_t $2:process $3;
 mls_rangetrans_target($1)
 ')
')

```

新调用的接口调用了原始的`init_daemon_domain`，但增加了 MCS 和 MLS 相关的逻辑。在这两种情况下，它都调用了 `range_transition`，这样当 snort 的 `init` 脚本（以 `initrc_t` 运行）过渡到 `snort_t` 域时，活动的敏感度范围也会被更改为第三个参数。

在我们的例子中，第三个参数是 `s3:mcs_allcats`，其中 `mcs_allcats` 是一个定义，扩展为策略支持的所有类别（例如，如果策略支持 256 类别，则为 `c0.c255`）。

在 MLS 情况下，它还调用了 `mls_rangetrans_target`，这是一个接口，将一个属性设置为 `snort_t` 域，这是策略中启用的 MLS 限制所需要的。

从扩展的代码中，我们可以看到有 `ifdef()` 语句。这些是 SELinux 策略规则的块，基于构建时的参数启用（或忽略）。如果启用了 MCS 或 MLS 策略，则会设置 `enable_mcs` 和 `enable_mls` 参数。其他常用的构建时参数包括发行版选择（例如，如果 SELinux 策略规则针对 Red Hat Enterprise Linux 和 Fedora 系统，则为 `distro_redhat`）和 `enable_ubac`（即启用了基于用户的访问控制时）。

### 限制

大多数（如果不是全部的话）SELinux 策略开发集中在类型强制规则和上下文定义上。SELinux 确实支持各种其他语句，其中之一是`constrain`语句，用于实现约束。

约束基于一组表达式进一步限制权限，这些表达式不仅涵盖对象或主体的类型，还包括 SELinux 角色和 SELinux 用户。与`mlsrangetrans`属性（由`mls_rangetrans_target`接口设置）相关的约束如下所示：

```
mlsconstrain process transition
  (( h1 dom h2 ) and
   (( l1 eq l2 ) or ( t1 == mlsprocsetsl ) or
    (( t1 == privrangetrans ) and ( t2 == mlsrangetrans ))));
```

约束告诉我们有关转换的以下信息：

+   只有当主体（域/执行者）的最高敏感度级别支配对象的最高敏感度级别时，转换才会发生。

+   主体的最低敏感度级别与对象的最低敏感度级别相同。

+   如果不是这样，则主体的类型必须设置`mlsprocsetsl`属性。

+   如果不是这样，则必须同时满足以下两个条件：

    +   主体的类型设置了`privrangetrans`属性。

    +   对象的类型设置了`mlsrangetrans`属性。

支配意味着第一个安全级别的敏感度级别等于或高于第二个安全级别的敏感度级别，并且第一个安全级别的类别与第二个安全级别的类别相同或是其超集。

SELinux 策略中的约束是基础策略集的一部分——这意味着我们不能通过可加载的 SELinux 策略添加约束。如果我们想要包括额外的约束，我们需要自己构建整个策略，修补策略库中`policy/`子目录下的`constraints`、`mls`和`mcs`文件。

了解约束是很重要的，但我们可能永远不需要自己编写约束。

## 另见

SELinux 项目网站是学习约束及其相关语句的良好起点：

+   可以参考 SELinux 语句[`selinuxproject.org/page/NB_MLS`](http://selinuxproject.org/page/NB_MLS)

+   约束语句可以参考[`selinuxproject.org/page/ConstraintStatements`](http://selinuxproject.org/page/ConstraintStatements)

# 配置敏感度类别

虽然 MCS 策略支持 MLS，但它们配置为仅支持单一敏感度（即`s0`）。尽管有这个限制，MCS 策略仍然非常有用，例如在系统为多个客户提供服务的情况下。这是因为 MCS 仍然可以根据类别受益于安全许可。

与敏感度不同，类别更像是一个自主访问控制系统。类别旨在由用户（或管理员）用来标记文件和其他资源，使其成为一个或多个类别的成员。对这些资源的访问则基于进程的许可级别和分配给资源的类别。类别也没有层次结构。

一个类别发挥重要作用的使用案例是在多租户部署中：这些系统为多个租户（多个客户）托管一个或多个服务，当然，需要适当的安全隔离，以确保一个租户无法访问另一个租户的资源。

在大多数情况下，管理员会尝试通过运行时用户（和组成员身份）来分隔这些服务。然而，这并非总是可能的。有些情况下，这些独立的进程仍然需要以相同的运行时用户身份运行（尽管通过支持额外的 Linux 安全子系统——如能力——这些情况已经显著减少）。

在本食谱中，我们将配置一个系统，使用多个类别来区分不同客户的资源，用于客户也能访问 shell 的 Web 服务器。通过类别，我们可以为其他客户的资源提供更多保护，以防其中一个客户能够执行提升其权限的漏洞。

## 准备工作

你需要为多个租户准备系统。例如，我们可以将整个网站内容托管在 `/srv/web/<companyname>/` 中，并将 Web 服务器配置放在 `/etc/apache/conf/<companyname>/` 中。

在本食谱中，作为示例，我们将为两个公司（`CompanyX` 和 `CompanyY`）配置系统。每个公司也有一个普通用户（第一家公司为 `userX`，第二家公司为 `userY`）。

## 如何操作……

要实例化不同的类别，按以下方法进行：

1.  确定不同客户的类别命名（和编号），并在 `/etc/selinux/mcs/` 中的 `setrans.conf` 文件中进行配置：

    ```
    s0:c100=CompanyX
    s0-s0:c100=CompanyXClearance
    s0:c101=CompanyY
    s0-s0:c101=CompanyYClearance
    ```

1.  重新启动 `mcstrans` 服务，使其能够识别此配置。

1.  列出类别，以确保更改能被正确解释：

    ```
    ~$ chcat –L
    s0    SystemLow
    s0-s0:c0.c1023  SystemLow-SystemHigh
    s0:c0.c1023  SystemHigh
    s0:c100    CompanyX
    s0-s0:c100  CompanyXClearance
    s0:c101    CompanyY
    s0-s0:c101  CompanyYClearance

    ```

1.  创建具有处理正确类别权限的 SELinux 用户：

    ```
    ~# semanage user –a –L s0 –r CompanyXClearance –R "user_r" userX_u
    ~# semanage user –a –L s0 –r CompanyYClearance –R "user_r" userY_u

    ```

1.  配置 Linux 用户（登录名）以获得正确的安全权限：

    ```
    ~# semanage login –m –s userX_u userX
    ~# semanage login –m –s userX_u userY

    ```

1.  为公司资源设置正确的类别：

    ```
    ~# chcon –l CompanyX –R /srv/web/www.companyX.com/ /etc/apache/conf/companyX/
    ~# chcon –l CompanyY –R /srv/web/www.companyY.com/ /etc/apache/conf/companyY/

    ```

1.  配置 Apache `init` 脚本，通过 `runcon` 启动 Apache，以正确的安全级别启动。例如，在 Red Hat Enterprise Linux 6 系统中，对于第一家公司的网站服务器，使用以下脚本：

    ```
    LANG=$HTTPD_LANG daemon --pidfile=${pidfile} runcon –t httpd_t –l CompanyX $httpd $OPTIONS
    ```

1.  （重新）启动 Web 服务器，并验证它是否以正确的安全级别运行：

    ```
    ~# ps –efZ | grep httpd

    ```

## 它是如何工作的……

我们从配置系统开始，使得可以命名类别和范围，而不必使用整数表示法。接着，我们为每个公司创建了一个 SELinux 用户，并将每个（普通）Linux 账户分配给正确的 SELinux 用户。在更新所有公司相关文件的上下文后，我们配置了 Apache 以正确的上下文启动。

### mcstrans 和 setrans.conf 文件

`setrans.conf` 文件是一个普通的文本文件，MCS 过渡守护进程（`mcstransd`）用它来将实际的安全级别（如 `s0:c100`）替换为人类可读的字符串（如 `CompanyX`）。

Linux 工具本身（如`ls`和`ps`）使用 SELinux 库获取文件和进程的上下文信息。这些库然后与`mcstransd`进程（通过`/var/run/setrans/.setrans-unix`套接字）连接，发送实际的安全级别并检索其人类可读的表示。

需要记住的是，这仅仅是一个表示，并非安全级别存储的方式。换句话说，不要在文件上下文定义文件中使用此表示（即 SELinux 策略`.fc`文件）。

### SELinux 用户和 Linux 用户映射

在这个示例中，为每个公司创建了一个 SELinux 用户。这个 SELinux 用户被授予了与该公司类别标记的资源进行工作的许可。然后，实际的 Linux 账户被映射到这个 SELinux 用户。

从示例中，我们看到每个公司都有两个定义：

```
s0:c100    CompanyX
s0-s0:c100  CompanyXClearance
```

第一个是安全级别，可以分配给资源以及进程（用户）。第二个是安全许可（一个范围）。在这个特定的示例中，许可告诉我们，高安全级别（可以视为*进程被允许访问的内容*）是公司的资源（`s0:c100`），而低安全级别（可以视为*进程本身的安全级别*）仅为`s0`。

因此，公司的用户被授权访问分配给其公司类别的文件（和其他资源）。然而，所有由这些用户帐户执行的活动默认不会获得此类别——用户需要使用`chcon`来设置类别，如下所示：

```
~$ chcon –l CompanyX public_html/index.html

```

可以将安全级别本身授予用户，而不是许可。当发生这种情况时，用户创建的任何资源也将获得适当的类别设置。但是，不要将此用作限制资源的方式——用户始终可以从资源中移除类别。

授予安全级别可以在 SELinux 用户级别完成，但也可以通过 SELinux 用户映射来实现，只要传递的范围受到 SELinux 用户级别设置的范围控制。例如，要将`CompanyX`（`s0:c100`）设置为安全级别，而不是默认的`CompanyXClearance`（这是映射到`userX_u` SELinux 用户的用户的默认设置），可以使用以下命令：

```
~# semanage login –m –r CompanyX user1

```

### 使用正确的上下文运行 Apache

示例中最后的变化是配置系统以在正确的安全级别下启动 Web 服务器。这是通过`runcon`命令完成的，我们传递敏感性级别（而不是安全许可），以确保通过 Web 服务器创建的每个资源都继承正确的类别以及目标类型。

SELinux 策略知道，如果一个`init`脚本启动 Apache 二进制文件(`httpd`)，则此应用程序必须在`httpd_t`域中运行。然而，现在`init`脚本启动了`runcon`—SELinux 策略将其视为常规二进制文件—因此应用程序将继续在`initrc_t`域中运行。因此，我们需要传递目标类型(`httpd_t`)。在没有非限制域的 SELinux 策略系统上，忘记这一点会阻止 Web 服务器的运行。在具有非限制域的 SELinux 策略系统上，这可能导致 Web 服务器在非限制域(`initrc_t`)中运行，从而有效地禁用了我们对 Web 服务器所需的 SELinux 保护！

## 另请参见

关于多租户和 SELinux 交互的更多示例如下：

+   sVirt ([`selinuxproject.org/page/SVirt`](http://selinuxproject.org/page/SVirt)) 使用 SELinux 分类将虚拟客户隔离开来。

+   Linux 容器，例如通过 LXC 项目 ([`linuxcontainers.org`](https://linuxcontainers.org))，利用 SELinux 进一步隔离容器与主系统。

+   Apache 通过`mod_selinux`模块支持多租户，在第三章中详细介绍了*限制 Web 应用程序*。
