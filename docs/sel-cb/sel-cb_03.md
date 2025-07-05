# 第三章。限制 Web 应用程序

在本章中，我们将涵盖 Web 服务器域的默认限制，并练习如何调整此策略以适应我们的需求。我们还将研究`mod_selinux`及其如何进一步限制 Web 应用程序。所有这些将通过以下方法处理：

+   列出条件性策略支持

+   启用用户目录支持

+   分配 Web 内容类型

+   使用不同的 Web 服务器端口

+   使用自定义内容类型

+   创建自定义 CGI 域

+   设置 mod_selinux

+   使用有限清除启动 Apache

+   将 HTTP 用户映射到上下文

+   使用源地址映射来决定上下文

+   使用 mod_selinux 分隔虚拟主机

# 介绍

Web 应用程序是 SELinux 可以证明其有效性的主要示例。它们经常面对（不受信任的）互联网，并且是攻击的热门目标。尽管保护 Web 服务器和 Web 应用程序只是基本的缓解策略之一，但通过限制 Web 服务器，我们进一步减少了成功利用的结果。

一个良好限制的 Web 服务器只允许向操作系统进行服务的可接受行为。但考虑到可以通过 Web 服务器提供的广泛服务领域，我们必须小心不要开放过多的权限。

策略开发人员已预见到 Web 服务器域可能在其权限上过于广泛，并且已使 Web 服务器域（`httpd_t`）不仅非常多才多艺，而且非常可配置。在本章中，我们将更详细地研究该域。

# 列出条件性策略支持

SELinux Web 服务器域策略的第一个可配置方面是其广泛使用 SELinux 布尔值。通过这些布尔值，可以选择性地启用或禁用附加策略规则。在本教程中，我们将查看这些布尔值，并了解如何切换它们。

## 如何做...

为了列出条件性策略支持，请执行以下步骤：

1.  请求所有 SELinux 布尔值的列表，并有选择地显示以`httpd_`开头的那些：

    ```
    ~# getsebool –a | grep httpd_

    ```

1.  要获得与布尔值一起的简短描述，我们可以使用`semanage`：

    ```
    ~# semanage boolean –l | grep httpd_

    ```

1.  如果布尔值的描述不足够，我们可以请求 SELinux 实用程序显示将在设置布尔值时启用（或禁用）的 SELinux 规则：

    ```
    ~# sesearch –b httpd_enable_ftp_server –AC
    Found 3 semantic av rules:
    DT allow httpd_t httpd_t : capability net_bind_service ; [ httpd_enable_ftp_server ]
    DT allow httpd_t ftp_port_t : tcp_socket { recv_msg send_msg name_bind } ; [ httpd_enable_ftp_server ]
    DT allow httpd_t ftp_server_packet_t : packet { send recv } ; [ httpd_enable_ftp_server ]

    ```

## 工作原理...

通过 SELinux 布尔值提供条件性 SELinux 策略支持。这些是可配置的参数（具有`true`/`false`值），管理员可以使用`setsebool`或`semanage boolean`命令启用或禁用。

使用`getsebool`命令，我们请求所有 SELinux 布尔值的概述。最近的策略分配了几百个布尔值，但幸运的是，大多数布尔值遵循以下两种命名约定之一，这使得过滤更容易：

+   布尔值以`allow_`或`use_`开头

+   布尔值以 SELinux 策略模块前缀开头

以`allow_`或`use_`开头的布尔值被认为是全局布尔值，通常会影响多个 SELinux 策略模块。一个很好的例子是`allow_execmem`，它允许多个域在可写内存中执行代码，而不是在只读内存中执行代码（这是一个有害但有时不可避免的内存权限设置）。

大多数，甚至所有其他布尔值都以它们所应用的 SELinux 策略模块前缀开始。对于 Web 服务器来说，这是`httpd_`（尽管策略名为 apache，但选择`httpd_`前缀是因为该策略可以直接应用于各种 Web 服务器，而不仅仅是 Apache HTTPd）。

当我们使用`semanage boolean`命令时，会提供关于布尔值的简短描述。这个描述来自一个名为`policy.xml`的 XML 文件，可以在`/usr/share/selinux/devel/`目录下找到。该 XML 文件是在构建基本 SELinux 策略时生成的。

然而，布尔值的最准确描述是它启用或禁用时会触发的一组规则。这就是`sesearch`命令发挥作用的地方。

如示例所示，布尔值将触发一个或多个允许规则。`sesearch`输出的前缀告诉我们，如果布尔值为真（`T`）或假（`F`），显示的规则是否处于活动状态，以及该规则是否在策略中启用（`E`）或禁用（`D`）。

在使用`sesearch`查询 SELinux 策略时，一个不错的技巧是同时查询布尔值管理的规则（无论它们当前是否启用或禁用）。这可以通过添加`–C`选项来实现（该选项是`--show_cond`的简写）。例如，要查找`newrole_t`域的转换，可以使用以下命令：

```
~# sesearch –s newrole_t –c process –p transition –AC
Found 5 semantic av rules:
 allow newrole_t newrole_t : process { … };
 allow newrole_t chkpwd_t : process transition;
 allow newrole_t updpwd_t : process transition;
EF allow newrole_t userdomain : process transition ; [ secure_mode ]
DT allow newrole_t unpriv_userdomain : process transition ; [ secure_mode ]

```

## 另请参见

+   `httpd_selinux`手册页面列出了所有适用于 Apache SELinux 模块的 SELinux 布尔值，并详细解释了它们的用途：

    ```
    ~$ man httpd_selinux

    ```

# 启用用户目录支持

让我们看一个如何使用适用于 Web 服务器安装的 SELinux 布尔值的示例。在这个示例中，我们将启用 Apache UserDir 支持（允许 Web 服务器在`http://sitename/~username`处提供本地用户账户的网页）。

## 准备工作

配置 Apache Web 服务器以提供用户内容。这里本应包括完整的 Apache 配置教程，但这超出了本书的范围。基本上，这是通过编辑`httpd.conf`文件并设置`UserDir`指令来完成的。

## 如何操作…

要启用用户目录支持，请按照以下步骤操作：

1.  确保用户的主目录对于 Apache 运行时账户是可访问的，使用以下命令。如果 Linux DAC 拒绝访问，SELinux 甚至不会处理该请求。

    ```
    ~$ chmod 755 ${HOME}/
    ~$ chmod 755 ${HOME}/public_html

    ```

1.  检查访问是否已经被允许，可以通过访问用户页面来确认。如果所有权限都正确，但 SELinux 拒绝访问，则页面应返回 403（禁止访问）错误，并且拒绝访问的操作应记录在审计日志中。Apache 错误日志会显示对该资源的权限拒绝。

1.  审计日志可能会显示 `httpd_t` 不允许对 `home_root_t` 或 `user_home_dir_t` 执行操作。通过查看 SELinux 布尔值，我们发现至少有两个有趣的布尔值（`httpd_enable_homedirs` 和 `httpd_read_user_content`）：

    ```
    ~# sesearch -s httpd_t -t home_root_t -c dir -p open -AC
    Found 2 semantic av rules:
    DT allow httpd_t home_root_t : dir { getattr search open } ; [ httpd_enable_homedirs ]
    DT allow httpd_t home_root_t : dir { getattr search open } ; [ httpd_read_user_content ]

    ```

1.  首先切换 `httpd_read_user_content`。这允许 Web 服务器访问所有用户文件，功能上是可以的，但这也立即赋予它对所有文件的访问权限：

    ```
    ~# setsebool httpd_read_user_content on

    ```

1.  另一种方法（但这种方法需要用户干预）是将 `~/public_html/` 标记为 `httpd_user_content_t`。完成后，可以关闭 `httpd_read_user_content` 并启用 `httpd_enable_homedirs`：

    ```
    ~$ chcon –R –t httpd_user_content_t public_html
    ~# setsebool httpd_read_user_content off
    ~# setsebool httpd_enable_homedirs on

    ```

1.  当更改工作正常时，我们可以使更改持久化，以便它们在重启后仍然有效：

    ```
    ~# setsebool –P httpd_enable_homedirs on

    ```

## 它是如何工作的……

SELinux 中的默认 Web 服务器策略不允许 Web 服务器访问用户的主目录内容。如果 Web 应用程序或 Apache Web 服务器本身存在漏洞，允许攻击者读取用户内容，SELinux 将阻止这种情况的发生。但有时，确实需要访问用户内容。

通过启用 `httpd_read_user_content` 布尔值，Web 服务器域（及所有相关域）将拥有对所有用户文件的完全读取权限。如果用户无法（或不知道如何）为其文件设置正确的上下文，则这是唯一合适的选项。

然而，更好的方法是启用 `httpd_enable_homedirs` 布尔值。这允许 Web 服务器访问主目录（`/home/user/`，标记为 `user_home_dir_t`），但不提供对用户内容（标记为 `user_home_t`）的读取权限。相反，Web 服务器所需的资源被标记为 `httpd_user_content_t` —— 这是常规用户可以重新标记资源的类型（或从中重新标记资源）。在 `httpd_user_content_t` 旁边，还可以定义以下内容类型：

+   `httpd_user_htaccess_t` 用于 `.htaccess` 文件

+   `httpd_user_script_exec_t` 用于用户提供的 CGI 脚本

+   `httpd_user_ra_content_t` 用于可附加的资源（供 Web 服务器使用）

+   `httpd_user_rw_content_t` 用于读/写资源（供 Web 服务器使用）

这些资源可以由最终用户设置，提供对 `~/public_html/` 目录中每个资源如何被 Web 服务器（及 Web 应用）处理的更精细控制。

## 还有更多……

一些支持 SELinux 的发行版有一个名为 `restorecond` 的守护进程，可以在文件创建/检测时自动设置文件的上下文，而无需在策略中进行文件转换。这样可以自动将 `~/public_html/` 标记为 `httpd_user_content_t`。

## 另见

+   有关每个用户 Web 目录的更多信息，可以参考 [`httpd.apache.org/docs/2.4/howto/public_html.html`](https://httpd.apache.org/docs/2.4/howto/public_html.html)

# 分配 Web 内容类型

对于标准 Web 服务器配置（没有 SELinux），Web 服务器的资源访问权限完全依赖于文件的所有权（以及应用于它的访问掩码）。使用 SELinux 后，资源可以根据其功能意义进行更具体的标记。

Web 应用程序包含应为只读内容和应为读写内容的资源，但也有针对 `.htaccess` 文件等资源的特定类型。在这个示例中，我们将查看各种 Web 服务器内容类型，并将它们应用于正确的资源。

## 如何操作……

执行以下步骤，将特定的 Web 内容类型分配给正确的资源：

1.  通过请求 SELinux 显示所有具有 `httpdcontent` 属性的类型，来看一下可用于 Web 服务器的内容类型：

    ```
    ~$ seinfo –ahttpdcontent –x
     httpdcontent
     httpd_sys_content_t
     httpd_user_ra_content_t
     httpd_user_rw_content_t
     httpd_nagios_content_t
    …

    ```

1.  查询现有策略中的已知上下文分配（因为这些可以为我们提供哪些内容仍然缺失的线索）：

    ```
    ~$ semanage fcontext –l | grep httpd_nagios

    ```

1.  现在，将正确的上下文分配给那些尚未正确标记的资源。这里使用的路径是 Nagios 安装的示例：

    ```
    ~# semanage fcontext –a –t httpd_nagios_content_t /var/www/html/nagios(/.*)?
    ~# semanage fcontext –a –t httpd_nagios_script_exec_t /usr/local/lib/nagios/cgi-bin/.*
    ~# restorecon –R /var/www/html/nagios /usr/local/lib/nagios

    ```

## 它是如何工作的

Web 服务器策略支持用于 Web 应用程序的功能性内容类型。这些类型用于以下内容类型：

+   Web 应用程序的只读内容

+   Web 应用程序的可写内容（对于可写内容与仅可追加内容（如日志文件）之间的区分）

+   可执行脚本（用于 CGI 脚本和类似内容）

这个优势不在于只读与读写的区分，而在于这种支持是基于每个应用程序的，且具有特定于单个应用程序的类型。在此示例中，我们查看了 Nagios 监控应用程序的内容。

这使得管理员可以向特定的应用程序或用户提供对这些资源的访问权限。即使 `/var/www/html/` 中的所有内容都可能归 Apache Linux 用户所有，我们仍然可以向用户（和应用程序）授予对特定于应用程序的资源的访问权限，而无需授予这些用户或应用程序对所有 Apache 资源的完全权限。

对于只读内容，有常规的 Web 应用程序内容类型（`httpd_nagios_content_t`）和特殊的 `.htaccess` 内容类型（`httpd_nagios_htaccess_t`）。这种区分主要是因为对常规内容的访问权限更广泛（并且根据一些 SELinux 布尔值，这也可能成为可写内容），而 `.htaccess` 内容则保持只读。

为了查询可用的 web 服务器内容，我们使用了 `httpdcontent` 属性。此属性分配给所有内容，允许管理员创建管理所有 web 内容的策略。`httpdcontent` 属性赋给了所有这些类型，但也有名为 `httpd_rw_content`、`httpd_ra_content`、`httpd_htaccess_type` 和 `httpd_script_exec_type` 的属性，用于允许操作这些特定资源。

## 还有更多…

我们以 Nagios 作为示例 web 应用，它有一组与 web 应用相关的资源。许多其他 web 应用或具有 web 内容的应用已经在策略上被识别。

在所有已知策略默认加载的 Linux 发行版上，通过 `seinfo` 命令可以看到此概述，如我们前面的示例所示。如果不是这种情况，我们可以通过搜索 SELinux 策略来找出哪些模块调用了 `apache_content_template`——该接口会自动生成正确的 web 应用内容类型：

```
~$ grep apache_content_template ${POLICY_LOCATION}/policy/modules/*/*.te

```

当不同类型的资源变得比有用更麻烦时，可以要求 SELinux 策略将所有这些不同类型视为一个通用的 web 内容类型，处理完成即可。这是通过 `httpd_unified` 布尔值来支持的。当启用此布尔值时，web 服务器策略将把所有不同的 web 服务器资源类型视为一个统一类型。如果 `httpd_enable_cgi` 和 `httpd_builtin_scripting` 布尔值也被启用，那么 web 服务器域就拥有执行该内容的权限。

不用多说，统一 web 服务器资源上下文可能使管理变得更简单；但它也会增加 web 服务器域对各种 web 资源的权限，从而可能降低安全性。

# 使用不同的 web 服务器端口

默认情况下，web 服务器监听已知的 web 服务器端口（如端口`80`和`443`）。通常，管理员可能希望让 web 服务器监听非默认端口。SELinux 策略可能会拒绝这种做法，因为 web 服务器在其他不相关端口上监听并不是标准行为。

在本教程中，我们将告诉 SELinux，一个非默认端口仍应视为 web 服务器端口。

## 如何操作…

为了将标签分配给不同的端口，请执行以下步骤：

1.  若要查看所有匹配 `http_port_t` 的端口，请使用 `semanage port -l`：

    ```
    ~# semanage port -l | grep -w http_port_t
    http_port_t  tcp  80, 81, 443, 488, 8008, 8009, 8443, 9000

    ```

1.  查询 SELinux 策略以查看特定端口分配了哪个端口类型。例如，对于端口`8881`，可以使用以下命令：

    ```
    ~$ seinfo --portcon=8881

    ```

1.  如果端口被标识为 `unreserved_port_t`，则我们可以将其标记为 `http_port_t`：

    ```
    ~# semanage port -a -t http_port_t -p tcp 8881

    ```

1.  然而，如果端口已分配了特定类型，则需要更新 SELinux 策略，使 web 服务器允许在该特定类型的端口上监听。例如，对于端口 `9090`（`websm_port_t`），请执行以下步骤：

    1.  首先，找到允许绑定到 `websm_port_t` 的接口：

        ```
        ~$ sefindif websm_port_t.*bind

        ```

    1.  创建一个自定义 SELinux 策略（`myhttpd`），其内容如下：

        ```
        corenet_sendrecv_websm_server_packets(httpd_t)
        corenet_tcp_bind_websm_port(httpd_t)
        ```

    1.  加载策略以允许 Web 服务器绑定到指定的端口类型。

1.  最后，编辑 Web 服务器配置文件以监听正确的端口：

    ```
    Listen *:8881
    ```

## 它是如何工作的...

SELinux 使用标签管理所有资源，包括端口。在本示例中，我们关注的是允许 Web 服务器绑定的 TCP 端口类型。

使用`seinfo`，我们可以查看端口是否与已知声明匹配。默认情况下，值为 `1024` 或更高的端口会标记为 `unreserved_port_t`，而 `511` 或更低的端口则标记为 `reserved_port_t`，介于两者之间的端口标记为 `hi_reserved_port_t`。这些是默认设置，某些端口可能会为特定端口声明更具体的类型。

如果端口尚未分配特定类型，我们可以使用`semanage port`手动分配。这足以让 Web 服务器绑定到该端口（与文件或目录不同，端口不需要重新标签操作，因为 SELinux 子系统会立即完成这项工作）。

如果端口已分配特定类型，则无法通过其他策略或管理员覆盖。当发生这种情况时，SELinux 策略需要增强，以允许 Web 服务器绑定到该特定类型。

在示例中，我们搜索了允许 Web 服务器绑定到端口的接口，发现 `corenet_tcp_bind_websm_port` 是可以使用的接口。然而，我们还添加了另一个接口—这是由于 SELinux 中网络控制配置的方式，这可能在某些系统上是必要的，也可能不需要。额外的接口是 `corenet_sendrecv_websm_server_packets`。此接口用于允许 Web 服务器发送或接收标记为 `websm_server_packet_t` 的数据包。数据包标记使得应用程序特定的通信流治理成为可能，并通过 SELinux 域感知扩展了 Linux 操作系统的常规防火墙功能（主要集中在网络流量管理上）。

如果需要对数据包进行标记，可以通过在本地系统上使用`iptables`对数据包进行标记，如以下命令所示：

```
~# iptables –t mangle –A INPUT –p tcp --dport 9090 -j SECMARK --selctx system_u:object_r:websm_server_packet_t

```

如果系统没有基于 iptables 的标记（称为 SECMARK 标记），则不需要接口。

## 还有更多...

最近的 SELinux 用户空间工具提供了另一个命令，用于查询 SELinux 策略，叫做`sepolicy`。使用`sepolicy`搜索端口声明的方法如下：

```
~$ sepolicy network --port 8080
8080: tcp unreserved_port_t 1024-65535
8080: udp unreserved_port_t 1024-65535
8080: tcp http_cache_port_t 8080

```

此外，在 SELinux 策略规则中，我们会注意到网络通信通常会启用第三个接口。在我们的示例中，第三个接口将被称为`corenet_tcp_sendrecv_websm_port`。此访问向量将使域能够在`websm_port_t` TCP 套接字上发送和接收消息。然而，在最近的策略中，出于对 SECMARK 标记的支持，已禁用对该访问向量的支持。

## 另见

+   SECMARK 标签在第九章中进行了探讨，*将 SELinux 与 DAC 对齐*。

# 使用自定义内容类型

接下来是为尚未与策略关联的 Web 应用程序创建我们自己的内容类型。我们将以**DokuWiki**（可以访问 [`www.dokuwiki.org`](https://www.dokuwiki.org)）为例。

## 准备工作

通过 Linux 发行版的包管理器或通过从主站点下载发布版本手动安装 DokuWiki。在此示例中，我们假设 DokuWiki 安装在 `/srv/web/dokuwiki/` 目录下。

## 如何操作...

要使用自定义 Web 内容类型，请按照以下步骤操作：

1.  创建一个名为 `mydokuwiki.te` 的策略文件，内容如下：

    ```
    apache_content_template(dokuwiki)
    ```

1.  添加一个名为 `mydokuwiki.fc` 的文件上下文定义文件，内容如下：

    ```
    /srv/web/dokuwiki/lib/plugins(/.*)?  gen_context(system_u:object_r:httpd_dokuwiki_rw_content_t,s0)
    /srv/web/dokuwiki/conf(/.*)?  gen_context(system_u:object_r:httpd_dokuwiki_rw_content_t,s0)
    /srv/web/dokuwiki/data(/.*)?  gen_context(system_u:object_r:httpd_dokuwiki_rw_content_t,s0)
    /srv/web/dokuwiki/data/\.htaccess  --  gen_context(system_u:object_r:httpd_dokuwiki_htaccess_t,s0)
    /srv/web/dokuwiki(/.*)?  gen_context(system_u:object_r:httpd_dokuwiki_content_t,s0)
    ```

1.  构建并加载策略，然后使用以下命令重新标记所有 DokuWiki 文件：

    ```
    ~# semodule -i mydokuwiki.pp
    ~# restorecon -RvF /srv/web/dokuwiki

    ```

## 它是如何工作的...

所有与在 SELinux 中创建 Web 应用程序内容相关的魔法都由 `apache_content_template` 接口处理。使用 `seshowif`，可以显示所有底层的 SELinux 策略规则，如下所示：

+   创建了各种 SELinux 类型，如 `httpd_dokuwiki_content_t` 等，并为其分配了适当的属性（例如 `httpdcontent` 属性）。

+   创建了一个 SELinux 布尔值，它允许管理员启用或禁用 Web 应用程序写入公共文件（标记为`public_content_rw_t`）。这是一个用于多个服务共享资源的 SELinux 类型（例如 FTP 服务器、Web 服务器等）。

+   必要的权限授予 Web 服务器域，以便访问和处理新定义的类型，并启用 Web 应用程序的 CGI 域。对于我们的 DokuWiki 示例，通常不需要此操作，因为所有内容都由 Web 服务器本身解析并执行的 PHP 代码处理。

然后，我们根据 DokuWiki 的最佳实践标记了所有 DokuWiki 文件。某些管理员可能希望将 `conf/` 子目录标记为不可写资源，并仅在配置过程中（临时）启用此设置。虽然这是一个有效的方法，但使用 Linux DAC 文件访问控制可能足以实现相同的结果。

## 还有更多内容...

使用 `apache_content_template` 接口是创建 Web 内容类型的简单方法，但它的缺点是它是一个全有或全无的方法，并且该模块现在高度依赖于 Web 服务器模块（`apache`）。

经验丰富的用户可能希望选择性地创建内容并为其分配正确的属性，允许 Web 服务器域与资源进行交互，同时仍然对类型和资源保持细粒度的控制。

我们将把这个任务留给你来完成，看看如何实现它。

# 创建一个自定义 CGI 域

有时，可能不需要创建一整套类型。考虑一下触发的 CGI 脚本，但不需要特定的内容类型集。可以将脚本标记为`httpd_sys_script_exec_t`（如果它是系统的 CGI 脚本）或`httpd_user_script_exec_t`（如果它是用户自定义的 CGI 脚本），这样结果脚本将在`httpd_sys_script_t`或`httpd_user_script_t`域中运行。

但是，如果这些域没有足够的权限（或者权限过多），则最好创建一个自定义的 CGI 域。

## 如何操作…

创建自定义 CGI 域时，可以使用以下方法：

1.  创建一个自定义的 SELinux 策略模块（`mycgiscript.te`），内容如下：

    ```
    policy_module(mycgiscript, 0.1)
    type cgiscript_t;
    type cgiscript_exec_t;
    domain_type(cgiscript_t)
    domain_entry_file(cgiscript_t, cgiscript_exec_t)
    apache_cgi_domain(cgiscript_t, cgiscript_exec_t)
    ```

1.  创建适当的文件上下文文件（`mycgiscript.fc`），并将可执行文件标记为`cgiscript_exec_t`：

    ```
    /path/to/script  --gen_context(system_u:object_r:cgiscript_exec_t,s0)
    ```

1.  构建并加载模块。

1.  重新标记可执行文件并进行测试：

    ```
    ~# restorecon /path/to/script

    ```

1.  由于`cgiscript_t`域在权限上较为初级，脚本很可能无法正常工作——然而，不要将 SELinux 设置为宽容模式。审计日志会显示被拒绝的访问尝试。与其使用`audit2allow`自动授予所有权限，不如使用`sefindif`功能找到合适的接口。将正确的接口添加到模块中，然后重试，直到脚本正常工作。

## 它是如何工作的…

策略模块定义了一个域类型（`cgiscript_t`）和一个可执行类型（`cgiscript_exec_t`）。通过`domain_type`接口，`cgiscript_t`被标记为一个域（并且为处理这个新域创建了适当的 SELinux 规则）。通过`domain_entry_type`，SELinux 策略被更新，标记`cgiscript_exec_t`为可以用于过渡到`cgiscript_t`域的类型。

然后，我们调用`apache_cgi_domain`，允许 Web 服务器域（`httpd_t`）执行标记为`cgiscript_exec_t`的资源，并使结果进程在`cgiscript_t`域中运行。

然而，最初的策略模块非常初级，权限不足。更新策略需要通过反复试验。例如，假设脚本调用了一个二进制文件；审计日志可能会显示如下内容：

```
type=AVC msg=audit(1363205612.277:476924): avc: denied { execute } for pid=6855 comm="cgiscript.pl" name="perl" dev=sda3 ino=4325828 scontext=system_u:system_r:cgiscript_t:s0 tcontext=system_u:object_r:bin_t:s0 tclass=file
```

为了找出哪种策略接口允许这样做，我们可以再次使用`sefindif`：

```
~$ sefindif exec.*bin_t'
interface(`corecmd_exec_bin',`
 can_exec($1, bin_t)

```

开发自定义策略仍然是一个试错过程，但这是唯一的方法，确保只授予域必要的权限。一些策略开发人员建议开启宽容模式，并查看审计日志中的所有拒绝信息。使用这种方法的问题是，这些拒绝可能无法导致正确的 SELinux 策略规则。

例如，脚本可能需要调用另一个可执行文件（并进行域过渡）。在宽容模式下，过渡不会发生，看起来主域（`cgiscript_t`）需要所有目标命令所需的权限——即使实际上只需要一个适当的域过渡。

通过专注于强制模式，我们可以逐步增加策略，同时保持*最小特权*原则，只允许实际需要的权限。

# 设置 mod_selinux

在接下来的几个教程中，我们使用一个名为`mod_selinux`的 Apache 模块，使 Apache 能够感知 SELinux 并支持可配置的转换。换句话说，Apache 运行的上下文不再是一个静态定义的上下文，而是可以根据管理员的需求进行更改。

在本教程中，我们将从源代码安装`mod_selinux`，因为许多 Linux 发行版默认并未提供它，尽管它是 Web 服务器非常强大的一个扩展（这也是为什么`mod_selinux`的支持常被称为 Apache/SELinux Plus 的原因）。

## 如何操作……

您可以通过以下步骤设置`mod_selinux`：

1.  从[`github.com/kaigai/mod_selinux`](https://github.com/kaigai/mod_selinux)下载源代码。

1.  确保安装了 Apache 开发头文件（在 Red Hat 或 Fedora 系统中为`httpd-devel`）。

1.  使用`apxs`构建并安装`mod_selinux`共享库以供 Apache 使用：

    ```
    ~# apxs -c -i mod_selinux.c

    ```

    ### 注意

    如果构建时出现关于`client_ip`的错误，可能是因为此问题。如果是这种情况，编辑错误中显示的行号处的`mod_selinux.c`文件，并使用`remote_ip`替换`client_ip`，然后可以再次运行`apxs`命令。

1.  构建并安装`mod_selinux` SELinux 策略模块，其文件也是下载源代码的一部分：

    ```
    ~$ cp mod_selinux.te ${DEVROOT}/local
    ~$ cp mod_selinux.if ${DEVROOT}/local
    ~$ cd ${DEVROOT}/local && make mod_selinux.pp
    ~# semodule -i mod_selinux.pp

    ```

1.  编辑 Web 服务器配置文件（`httpd.conf`），并添加适当的`LoadModule`行：

    ```
    LoadModule selinux_module modules/mod_selinux.so
    ```

1.  重启 Web 服务器。其日志文件应显示 SELinux 策略支持已加载：

    ```
    [Fri Apr 18 13:11:23 2014] [notice] SELinux policy enabled; httpd running as context unconfined_u:system_r:httpd_t:s0-s0:c0.c1023
    ```

## 它是如何工作的……

`mod_selinux.c`文件包含 Apache 模块代码，可以使用`apxs`—Apache 扩展工具进行构建。此工具将执行以下任务：

+   使用正确的编译器参数调用编译器，构建一个可以在运行时由 Apache Web 服务器加载的动态共享对象

+   将生成的模块安装到适当的 Apache `modules/`目录中

本教程中提到的构建失败可能取决于所使用的 Apache 版本，其中一个变量的名称不同（`client_ip`而非`remote_ip`）。

接下来，我们像设置其他 SELinux 策略模块一样，复制并部署了`mod_selinux`的 SELinux 策略。

最后，Web 服务器被更新以启用`mod_selinux` Apache 模块。随着`mod_selinux`共享库的就位，Apache 现在准备好做出与 SELinux 相关的决策。

如果`mod_selinux`支持需要分发到多个系统，则只需要分发`mod_selinux.so`（现在已安装在 Web 服务器`modules/`目录中，例如`/usr/lib64/httpd/modules/`）和`mod_selinux.pp`文件（SELinux 策略模块）。

## 另见

+   关于`mod_selinux`的详细介绍可以在[`code.google.com/p/sepgsql/wiki/Apache_SELinux_plus`](http://code.google.com/p/sepgsql/wiki/Apache_SELinux_plus)找到。

# 使用有限权限启动 Apache

在上一章节中，我们操作了 `/etc/rc.d/init.d/httpd init` 脚本，使用 `runcon` 使 web 服务器以有限权限运行。但借助 `mod_selinux`，这一点可以变得可配置。

## 如何操作……

为了使用有限的安全许可启动 Apache，请按照以下步骤操作：

1.  编辑 Apache web 服务器配置文件（`httpd.conf`）并添加以下代码：

    ```
    <IfModule mod_selinux.c>
      selinuxServerDomain *:s0-s0:c0.c10
    </IfModule>
    ```

1.  撤销在上一章节中对服务脚本所做的更改。

1.  重新启动 web 服务器并通过执行以下命令确认它正在以 `s0-s0:c0.c10` 清除权限运行：

    ```
    ~# /etc/rc.d/init.d/httpd restart
    ~# ps -efZ | grep httpd
    system_u:system_r:httpd_t:s0-s0:c0.c10 root 2838 1  0 13:14 ?      00:00:00 /usr/sbin/httpd
    system_u:system_r:httpd_t:s0-s0:c0.c10 apache 2840 2838  0 13:14 ? 00:00:00 /usr/sbin/httpd

    ```

## 它是如何工作的……

如前所述，通过 `mod_selinux`，Apache web 服务器变得支持 SELinux，这意味着它可以根据配置设置和 SELinux 策略规则改变自己的行为并与 SELinux 子系统进行交互。

使用 `selinuxServerDomain` 配置指令，`mod_selinux` 会动态更改当前上下文到新上下文，这称为动态域过渡或动态范围过渡（如果类型发生变化则称为域过渡，若敏感度级别或安全许可发生变化则称为范围过渡）。只有应用程序是 SELinux 感知的情况下，这才是可能的。

现在，这样的过渡仍然通过 SELinux 策略进行控制。例如，Apache web 服务器可以过渡的范围必须由 Apache web 服务器最初拥有的范围主导（在我们的示例中为 `s0-s0:c0.c1024`）。

### 注意

`mod_selinux` 模块不支持上下文查询，因此无法使用人类可读的敏感度（如我们之前所见，通过 `mcstransd` 控制）。

## 还有更多……

可以定义不同的类型，允许整个 web 服务器运行在自定义域中。为了实现这一点，`httpd_t` 域必须具备动态过渡到目标类型的权限（即 `process` 类中的 `dyntransition` 权限）。然后，`selinuxServerDomain` 调用可能如下所示：

```
selinuxServerDomain myhttpd_t:s0-s0:c0.c10
```

当然，为了访问启动时 `httpd_t` 域已经可以访问的资源，还需要更多的权限，但 `dyntransition` 权限特定于希望支持动态域过渡的 SELinux 感知应用程序，而不是在进程执行时进行过渡。

# 将 HTTP 用户映射到上下文

应用程序通常在静态上下文中运行，这会限制应用程序所需的所有权限。即便是服务（守护进程）通常也会在服务生命周期内保持在自己的上下文中。但是，通过`mod_selinux`，可以根据已认证的用户将 web 服务器处理程序（负责处理特定请求的进程或线程）的上下文切换到另一个上下文。这允许管理员根据用户授予应用程序某些权限。当一个权限较低的用户利用 web 应用中的漏洞时，web 应用本身的权限限制可能会阻止成功的攻击。

## 如何操作……

通过以下步骤，我们将把 web 用户映射到一个特定的 SELinux 上下文：

1.  创建一个映射文件，其中列出了用户及其目标上下文。例如，将用户 John 的请求处理为敏感度`s0:c0,c2`，将用户 Cindy 的请求处理为敏感度`s0:c0.c5,c7`，所有未认证用户的请求处理为`anon_webapp_t:s0`，其他已认证用户的请求处理为`user_webapp_t:s0:c0`：

    ```
    john    *:s0:c0,c2
    cindy    *:s0:c0.c5,c7
    __anonymous__  anon_webapp_t:s0
    *      user_webapp_t:s0:c0
    ```

1.  将此文件保存到 web 服务器可读取的位置，例如`/etc/httpd/conf/mod_selinux.map`。

1.  编辑 web 服务器配置文件，并添加以下行：

    ```
    selinuxDomainMap  /etc/httpd/conf/mod_selinux.map
    ```

1.  重启 web 服务器。

## 它是如何工作的……

`mod_selinux`模块能够识别已认证用户的值，并且根据映射文件中的设置，它可以将请求处理程序的上下文切换到较低的敏感度范围（如前两个示例中所示），或者完全切换到不同的域。

不过，这里有一个重要的限制。请求处理程序可以过渡的目标上下文必须受主类型（`httpd_t`）的约束。这意味着授予目标上下文的权限必须是授予`httpd_t`权限的子集。这是通过`typebounds`语句来实现的，如下所示：

```
typebounds httpd_t anon_webapp_t;
```

这是因为 web 服务器处理程序通常是线程（或轻量级进程），而不是进程。线程共享许多资源，通常是以 SELinux 无法管理的方式共享的。因此，如果某个线程获得的权限超过 web 服务器的权限，那么整个 web 服务器的安全状态可能会受到威胁。此外，不同上下文之间的信息流动将变得困难，甚至几乎不可能进行管理。

# 使用源地址映射来决定上下文

`mod_selinux` Apache 模块可以访问除用户名之外的其他信息（对于已认证用户）。它可以访问环境变量（这些变量通过`SetEnvIf`指令在 Apache web 配置中使用），从而允许在应用程序中对 SELinux 上下文的处理更加灵活。

在本例中，我们将根据客户端的远程 IP 地址来更改请求处理程序的上下文。

## 如何操作……

除了 Web 用户，我们还可以使用源地址信息来决定上下文。可以通过完成以下步骤来实现：

1.  首先，我们在 Web 服务器配置（`httpd.conf`）中基于远程 IP 地址定义`TARGETDOMAIN`环境变量：

    ```
    SetEnvIf Remote_Addr "10\.0\.[0-9]+\.[0-9]+$" TARGETDOMAIN=user_webapp_t:s0
    SetEnvIf Remote_Addr "10\.1\.[0-9]+\.[0-9]+$" TARGETDOMAIN=anon_webapp_t:s0
    SetEnvIf TARGETDOMAIN ^$ TARGETDOMAIN=*:s0
    ```

1.  然后，在同一 Web 服务器配置中，我们调用`selinuxDomainEnv`指令，这将使处理程序的上下文过渡到`TARGETDOMAIN`中的值：

    ```
    selinuxDomainEnv TARGETDOMAIN
    ```

1.  重启 Web 服务器以使更改生效。

## 它是如何工作的...

在第一步中，我们使用了 Apache 的`SetEnvIf`指令（通过`mod_setenvif`提供）来检查客户端的远程 IP 地址（`Remote_Addr`）。如果它与给定的表达式匹配，我们就将`TARGETDOMAIN`变量设置为给定的上下文。在我们的示例中，我们为每个匹配使用了不同的类型，但也可以仅更改安全权限。最后，我们进行了一个检查，验证`TARGETDOMAIN`变量是否已被设置。如果没有，则分配一个默认值（`*:s0`）。

接下来，我们调用了`selinuxDomainEnv`指令，它会使 Web 服务器的上下文切换到`TARGETDOMAIN`变量中提供的域。

## 还有更多...

示例使用了`Remote_Addr`，但许多其他与请求相关的方面也可以使用：

+   使用`Remote_Host`，可以查询客户端的主机名并用来做出决策。

+   使用`Server_Addr`，可以使用 Web 服务器本身的地址（请求接收的服务器地址）。这在多宿主系统中非常有用，在这种系统中，Web 服务器绑定到所有可用的 IP 地址。

+   使用`Request_Method`，可以使用请求的类型（例如`GET`或`POST`）。

+   使用`Request_Protocol`，可以使用 HTTP 协议的名称和版本（例如`HTTP/1.0`或`HTTP/1.1`）。

+   使用`Request_URI`，可以使用请求的 URL 来调整上下文或权限。

## 另见

+   欲了解有关 Apache 的`mod_setenvif`支持的更多信息，请参考该模块文档：[`httpd.apache.org/docs/2.4/mod/mod_setenvif.html`](http://httpd.apache.org/docs/2.4/mod/mod_setenvif.html)

# 使用`mod_selinux`分离虚拟主机

Apache 的一个优势是，它可以根据用于连接服务器的名称来区分站点，而不仅仅是 IP 地址、端口和 URL。这被称为虚拟主机支持，是多租户网站和 Web 应用托管的非常流行的方法。

例如，运行在单一 IP 地址上的 Web 服务器仍然可以托管多个客户的网站，例如`www.companyX.com`和`www.companyY.com`。通过`mod_selinux`，我们可以根据关联的虚拟主机更改 Web 服务器请求处理程序的上下文或安全权限。

## 如何操作...

以下方法通过`mod_selinux`区分虚拟主机的限制：

1.  决定各个租户的上下文。在前一章中，我们为公司 X 使用了`s0:c100`，为公司 Y 使用了`s0:c101`。

1.  在每个虚拟主机中，设置正确的权限。例如，对于公司 X，设置权限如下：

    ```
    <VirtualHost *:443>
      ServerName www.companyX.com
     selinuxDomainVal *:s0-s0:c100
    </VirtualHost>
    ```

1.  重启 web 服务器以使更改生效。

## 它是如何工作的...

与针对整个 web 服务器的 `selinuxServerDomain` 指令不同，`selinuxDomainVal` 指令单独设置处理程序（虚拟主机）的上下文。正如我们在上一章中讨论的那样，为多租户系统使用多个类别是一种灵活的方式，可以处理租户之间的信息隔离。

然而，与上一章的一个重要区别是，`mod_selinux` 模块不使用 `mcstransd`。以下设置将失败：

```
selinuxDomainVal *:CompanyXClearance

```

这样的设置会导致 Apache 返回以下错误信息：

```
[error] (22)Invalid argument: SELinux: setcon_raw("unconfined_u:system_r:httpd_t:CompanyXClearance") failed
```

因此，我们需要使用标准的敏感性表示法。

## 另见

+   你可以在 [`httpd.apache.org/docs/2.4/vhosts/`](http://httpd.apache.org/docs/2.4/vhosts/) 上找到有关 Apache 虚拟主机支持的更多信息。
