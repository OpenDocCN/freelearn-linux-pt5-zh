# 第十一章：备份 VMware View 基础架构

尽管 VMware View 环境中不应存在单点故障，但确保定期备份以便在发生故障时能快速恢复仍然非常重要。此外，如果设置发生损坏或更改，备份可以用于恢复到先前的时间点。VMware View 环境的备份应该根据组织现有的备份方法定期执行。VMware View 环境包含文件和数据库。

VMware View 环境的主要备份点如下：

+   VMware View ADAM 数据库

+   VMware View Composer 数据库

+   VMware vCenter 数据库

+   VMware View Connection Server 设置

通过备份所有前述组件，在发生故障时可以恢复 VMware View Server 基础架构。

为了最大化恢复环境中的成功机会，建议同时备份 View ADAM、View Composer 和 vCenter 数据库，以避免数据不一致。备份可以计划和自动执行，也可以手动执行；理想情况下，应该使用计划备份以确保定期执行和完成。

正确的设计要求始终应有两个或更多的 View Connection Server。由于同一副本池中的所有 View Connection Server 都包含相同的配置数据，因此只需要备份一个 View Connection Server。此备份通常配置为在标准模式下安装的第一个 View Connection Server。

# 备份 VMware View Connection Server 环境

可以通过 VMware View Admin 控制台配置 View Connection Server 备份。这些备份将配置文件和数据库信息转储到 VMware View Connection Server 上的一个位置，之后必须通过常规机制进行备份，如备份代理和计划任务。VMware View Connection Server 备份的工作流程如下：

1.  安排 VMware View 备份任务并导出到 `C:\View_Backup\`。

1.  第三方备份解决方案在 View Connection Server 上运行，并备份 `System State`、`Program Files` 和 `C:\View_Backup\` 文件夹。

在 VMware View Admin 控制台中，有三个主要选项必须配置以备份 VMware View Connection Server 设置：

+   **自动备份频率：** 这是自动执行备份的频率。

    +   **每日推荐：** 由于大多数服务器备份是每天执行的，如果在 Windows 服务器的完整备份之前进行自动的 View Connection Server 备份，该备份将包含在夜间备份中；根据需要进行调整。

+   **最大备份数量：** 这是 View Connection Server 上可以存储的最大备份数量；一旦达到最大数量，备份将根据年龄进行轮换，最旧的备份将被最新的备份替换。

    +   **建议 30 天：** 这将确保服务器上保留大约一个月的备份；根据需要进行调整

+   **文件夹位置：** 这是 View 连接服务器上将存储备份的地点

    +   确保第三方备份解决方案正在备份此位置

一旦备份被自动运行或手动执行，将会在备份位置保存两种类型的文件，如下所示：

+   LDF 文件：这些是来自 VMware View 连接服务器 ADAM 数据库的 LDIF 导出文件，存储着 VMware View 环境的配置设置

+   SVI 文件：这些是 VMware View Composer 数据库的备份

View 连接服务器的备份过程相当简单。虽然过程容易，但不应忽视。

## 安全服务器注意事项

出人意料的是，没有通过 VMware View 管理控制台备份 VMware View 安全服务器的选项。对于 View 连接服务器，可以通过选择服务器、选择**编辑**，然后选择**备份**来配置备份。突出显示 View 安全服务器并没有提供此功能。

相反，安全服务器应该通过正常的第三方机制进行备份。最需要关注的是安装目录，默认情况下为`C:\Program Files\VMware\VMware View\Server`。

在`...\sslgateway\conf`目录中是`.config`文件，其中包含以下设置：

+   `pcoipClientIPAddress:` 这是安全服务器使用的公共地址

+   `pcoipClientUDPPort:` 这是用于 UDP 流量的端口（默认值：4172）

此外，`settings`文件位于此目录中，包含如下设置：

+   `maxConnections:` 这是 View 安全服务器一次可以支持的最大并发连接数（默认值：2000）

+   `serverID:` 这是安全服务器使用的主机名

此外，定制证书和日志文件存储在 VMware View 安全服务器的安装目录中。因此，如果要保留日志文件数据（且没有将其导入到更大的企业日志文件解决方案中），定期备份这些数据是非常重要的。

## 转移服务器和 ThinApp 仓库注意事项

在环境中备份 VMware View 转移服务器有两个组成部分。第一个组成部分是转移服务器的 VMware 安装目录和服务器的注册表。第二个组成部分更为重要，即实际存储已发布 vDesktops 的仓库。

如果 View 转移服务器本身出现故障，但仓库仍在线，签出和签入任务将无法使用，但主要数据，即已发布的 vDesktops，将仍然得到保留。一旦 View 转移服务器重新构建并恢复，它应当正常运行。在大多数生产环境中，应该有两个转移服务器，通常指向相同的冗余文件共享。

如果视图传输服务器在线但存储库失败，任何与传输相关的活动（如结账、更新或签入）都将失败。

ThinApp 存储库的性质与传输服务器存储库类似，它应该位于一个冗余文件共享上，并定期进行备份。如果 ThinApp 包已配置为保留每个用户的沙箱，那么 ThinApp 存储库应该可能每晚进行备份。

# 恢复 VMware View 环境

执行视图连接服务器恢复的步骤主要可以在 VMware KB 文章 *执行 View Manager 3.x 或 4.x 的端到端备份与恢复* 中找到。

# 备份黄金模板

对于使用完全克隆作为虚拟桌面（vDesktops）配置技术的环境，应该定期备份黄金模板。黄金模板是所有其他虚拟桌面克隆自的母模板。VMware KB 文章 *使用 VMware API 备份和恢复虚拟机模板* 介绍了备份和恢复模板的步骤。简而言之，大多数备份解决方案要求将黄金模板从模板转换为常规虚拟机，然后才能进行备份。

# 备份母虚拟机

备份母虚拟机可能比较棘手，因为它是一个虚拟机，通常有多个时间点的快照。最常见的技术是在特定时间点的快照处合并虚拟机快照树，然后将新创建的虚拟机备份或复制到第二个数据存储区。通过将母虚拟机存储在冗余存储解决方案中，母虚拟机丢失的可能性非常小。更可能的情况是，可能会在母虚拟机处于非功能性或不理想状态时创建时间点快照。

# 总结

正如预期的那样，备份 VMware View 解决方案的基本组件非常重要。尽管弹性设计应该能缓解大多数类型的故障，但仍然有一些情况下需要备份才能将环境恢复到可操作的状态。在 附录 *其他工具* 中讨论了一些对设计 VMware View 解决方案的 VDI 架构师可能有用的额外工具。
