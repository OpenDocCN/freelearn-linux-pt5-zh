# 第二章：准备将 FreeNAS 添加到网络中

就像所有系统部署一样，NAS 也需要正确规划以最大化成功。在本章中，我们将讨论一些基本的规划要点，包括：

+   容量规划

+   硬件要求

+   备份规划

+   冗余需求

+   网络基础设施

本章可能看起来不像是“动手操作”的内容，但成功规划和部署 NAS 需要做出重要决策并采取相应的行动。

# 规划你的 NAS

根据我的经验，计算机领域的人可以分为两种类型，一种是在添加任何新硬件或服务到网络之前进行精心规划的人，另一种是直接添加现有资源并寄希望于好运的人。你可能会很忙，觉得适当的规划是一个额外且不必要的步骤。但同样也有一个事实是，系统部署后解决问题的成本远高于在“上线”之前就解决它们。

比如，假设你没有正确规划硬盘需求，事实上，你现有的服务器无法容纳更多的硬盘。现在该怎么办？再买一台服务器？最好一开始就选对合适的服务器。

## 容量规划

部署 FreeNAS 的计划受到两个主要因素的限制，第一个是你可用的资源（即你已经拥有的 PC 或服务器，或者购买新设备的资金）以及你希望在 NAS 中拥有的容量。

### 注意

永远不要低估对磁盘空间的需求。视频文件、音频文件、电子邮件、软件下载；我们存储的数据类型清单在不断增长。我记得当我为我的 386 PC 购买第一个 170MB 的硬盘时，我还在想怎么可能填满 170MB 的空间。今天，一段简短的视频就有 170MB！

你拥有的资源越多，你能拥有的容量就越大，这是一种简单的关系。当然，FreeNAS 在这方面提供了多种帮助，首先，它是免费的。没有许可费用。如果你需要 2 个用户或 20 个用户，费用都是一样的……$0。此外，FreeNAS 对系统要求较轻，你不需要 4GB 的内存来运行这个服务器。

所以，第一个大问题是有多少用户将使用这个服务器？如果你是家庭用户，那么答案可能是不到 5 个人。也许，你希望 FreeNAS 充当一个简单的多媒体文件存储库，任何家庭中的 PC 都能访问。如果你在一个小型办公室工作，那么答案可能是不到 15 个用户，大型办公室则是不到 25 个用户。对于任何企业级部署，人数可能是 25 人及以上。

确定了这个数字后，你需要考虑这些用户中有多少人将向 NAS 写入，换句话说，将向 NAS 添加文件，以及有多少人只是读取已有的内容。我们将这些称为*写入*用户和*读取*用户。再次强调，在家庭环境中，可能只有 1 人实际上会将文件复制到 NAS，而其他 2 或 3 人可能只是使用它们。在办公环境中，情况就更难说了，这完全取决于您计划如何使用 FreeNAS。

现在，这一节的最后一个问题是什么？每个*写入*用户在服务器上需要多少空间？

现在你只需要做乘法：

*需要写入用户数量 X 需要的千兆字节*

因此，如果我们有 2 名*写入*用户，每个需要 5GB，那么您需要从 10GB 磁盘空间开始。如果您有 25 名*写入*用户，每个需要 10GB 磁盘空间，那么您需要 250GB 磁盘空间，以此类推。对于家庭用户，也许只有 1 个写入用户，但您想要 500GB，那么 1 X 500GB 就是 500GB！！！

### 注意

**现在翻倍**

无论你现在有多少数字，都要翻倍。你肯定在某处低估了，你不知道但确实如此。无论是用户数量还是他们需要的数据量。所以现在最安全的做法就是翻倍。这样，你就不会在六个月后因为缺少磁盘空间而被抓个正着了。

现在下一个计算有点棘手。我们需要计算您的数据增长速度。它的增长取决于您在 FreeNAS 上存储的内容。例如，如果您将 FreeNAS 用作备份服务器，那么随着用户创建文档、接收电子邮件、从互联网下载内容，备份其 PC 所需的磁盘空间将增加。这里没有固定的规则，您需要计算出来。中档和经济实惠（而不是顶级、尖端和昂贵的）硬盘，平均每年的容量增长约为 25%至 50%。它们的增长不是由需求驱动的，而是由技术驱动的，因此这实际上并不能为我们提供数据增长的指导。话虽如此，总是倾向于使用所有可用的磁盘空间。可用的磁盘空间越多，用户就越有办法把它填满。

我记得曾经为一家中型 IT 公司工作，服务器的磁盘空间不足了。发了封电子邮件要求大家从服务器上删除不必要的文件。删除后，磁盘空间释放了超过 50%。

一旦确定了每年需要多少额外的磁盘空间，您可以计算接下来三年的磁盘需求。在这个例子中，我们将使用 25 名初始需要 2GB 磁盘空间的用户，每年增长 25%。

初始所需空间：25 X 2 = 50GB

翻倍：= 100GB

第一年增长 25%：100 X 25% = 125GB

第二年增长 25%：125 X 25% = 156GB

第三年增长 25%：156 X 25% = 195GB

从这些数据可以看出，三年内磁盘使用量可能会翻倍。根据你的业务类型以及 FreeNAS 的使用方式，增长率甚至可能更高。如果数据增长率为 40%，那么所需的存储空间在两年内可能翻倍。

为了进一步调整这个公式，你可以考虑任何计划中的员工增长，因为每增加一名成员初期将需要额外 4GB 的磁盘空间，三年后将达到近 8GB。

## 选择硬件

FreeNAS 运行在 PC 平台上。其最低要求为一台“兼容 IBM PC”的机器，配备 Pentium 处理器、至少 96MB 的内存，以及可引导的 CD-ROM 驱动器和硬盘用于存储。然而，实际的最低要求是 Pentium II 处理器和 128MB 内存，当然还需要 CD-ROM 和硬盘。

### CPU

不可能列出 FreeNAS（或更具体地说，FreeBSD 操作系统）支持的所有制造商、主板和 CPU，但这里有一些通用的指南：

+   所有 Intel 处理器（从 Pentium 开始）都受支持，包括 Pentium、Pentium Pro、Pentium II、Pentium III、Pentium 4（及其变种如 Xeon 和 Celeron 处理器），以及 Intel Core 系列处理器（包括 Core Solo、Core Duo 和 Core 2 Duo）。

+   所有兼容 i386 的 AMD 处理器也受到支持，包括 Am486、Am5x86、K5、K6（及其变种）、Athlon（包括 Athlon MP、Athlon XP 和 Athlon Thunderbird）、Duron 和 Opteron 处理器。

+   支持所有标准 PC 总线，包括 ISA、AGP、PCI 和 PCI-X。不支持 IBM PS/2 系列 PC 所使用的 MicroChannel 扩展总线。

+   支持配有多个 CPU 的 PC，以及双核或四核的 PC。FreeBSD 还可以利用支持超线程技术（HT）的 Intel CPU。FreeBSD 将把这些附加的逻辑处理器当作额外的物理处理器来识别。FreeBSD 不会尝试根据逻辑处理器之间共享的资源来优化调度决策。

CPU 的选择对你的 NAS 非常重要。虽然 FreeNAS 可以在配备少于 128MB 内存的 Pentium 1 上运行，但在实际使用环境中它的性能表现不佳。虽然运行 NAS 在复杂数学运算上对 CPU 的需求不高，但由于需求量大，它可能对 CPU 有较高的消耗。如果有 5 个人同时访问文件，CPU 将被大量使用。我实验室中的一台测试机是 Pentium III，主频为 466MHz。使用非常快的网络连接向 FreeNAS 复制大文件时，CPU 的使用率达到了 100%。

以下是帮助你选择 CPU 的一些建议：

+   如果你只是想尝试 FreeNAS，并计划在小规模上使用它，那么一台老款的 233MHz 或更高频率的 Pentium II 机器（或 AMD 同类处理器）将是完美的选择。

+   对于家庭使用、备份或存储多媒体文件，最低要求是 1GHz 的 Pentium III（或相当于 AMD 的处理器）。这样的机器可以处理软件 RAID 和最多 10 个客户端。

+   对于小型办公环境，最低可接受的 CPU 是至少 1.3GHz 的 Pentium 4（或相当于 AMD 的处理器）。

+   对于大型安装，运行在 3GHz 的 Pentium 4（或相当于 AMD 的处理器）是最低要求，理想情况下，快速的双核或双处理器机器会更好。

### 注

**前端总线（FSB）**

FSB 将 CPU 与主内存连接起来。FSB 越快，数据传输到 CPU 的速度越快。通常，FSB 的速度（以 MHz 为单位）与 CPU 的速度成正比。然而，有一些主板/CPU 组合即使是更高速度的 CPU，也使用较低的 FSB。这将降低系统的整体性能。确保你选购的机器具有良好的 FSB 速度。

处理器速度并不是影响传输速度和并发用户容量的唯一因素。网络是一个非常重要的因素，我们很快会讨论到这一点，此外，机器中磁盘的类型也非常重要。

### 磁盘

处理器的任务是协调来自网络的请求与机器中的磁盘之间的操作。如果 CPU 较慢，那么协调工作就会变慢，整体性能也会下降。同样，如果磁盘较慢，系统的性能也会受到影响。NAS 的最终目标是将数据读写到硬盘。硬盘的速度决定了数据读取和写入的速度。

### 注

**质量**

在讨论磁盘和控制其性能的各种参数时，重要的是不要忽视磁盘的质量。所谓质量，是指磁盘故障的可能性。在你的 FreeNAS 服务器中，CPU 可能会失败，内存可能会失败，网络可能会失败，等等。但如果磁盘故障，你就有可能丢失数据。

在查看磁盘时，我们需要关注两点：首先是磁盘如何连接到 PC，其次是磁盘本身。

#### 总线

每个硬盘都通过一种叫做接口的东西与 PC 连接。接口是数据在硬盘和 PC 之间传输的管道。接口越快，数据传输的速度就越快。随着时间的推移，PC 采用了不同类型的硬盘接口，每次更新时，最高传输速度都会有所提高。

**IDE/ATA—** 最常见的硬盘接口（直到大约 2004 年）是先进技术附件（ATA）接口或集成驱动电子（IDE）接口。它是由西部数据公司在 1980 年代中期开发的，二十多年间是将硬盘和 CD-ROM 连接到 PC 的主要接口。ATA 有几个同义词或变体，包括增强型 IDE（EIDE）和 AT 附件数据包接口（ATAPI），但本质上它们是相同的技术，只是经过精炼和重复使用。2003 年，推出了一种新接口，称为串行 ATA（SATA），因此 ATA 被追溯性地重新命名为并行 ATA（PATA），以区别于新的接口。尽管作为连接硬盘到 PC 的方式逐渐退场，ATA 仍然是连接 CD-ROM 和 DVD-ROM 到 PC 的流行方式，许多主板都配有 PATA 和 SATA 接口。ATA 接口很容易识别，因为它们使用 40 或 80 根的排线将硬盘连接到 PC。

**SATA—** 串行高级技术附件（SATA）接口于 2003 年发布，并在 2004/2005 年间开始流行。SATA 比 PATA 更快，并且增加了在操作过程中移除或添加设备的能力（这叫热插拔）。SATA 使用比 PATA 更薄的电缆（不再是扁平的 80 针电缆），这使得空气冷却在 PC 中更高效。

**SCSI—** 一直以来，PATA 和 SATA 都有一种替代方案，那就是小型计算机系统接口，简称 SCSI。与 ATA 一样，SCSI（通常发音为“scuzzy”）已经存在很长时间了。它最常见于 PC 服务器，并且也在苹果的 Macintosh 电脑和 Sun Microsystems 中得到广泛应用。像并行 ATA 一样，SCSI 正被一种使用更小电缆、运行速度更快的串行版本所替代，这些新版本通常被称为串行 SCSI。

每种接口类型都有其优缺点。如果你正在使用现有的 PC 作为你的 FreeNAS 服务器，那么这些差异大多是理论上的，因为你只能使用机器中已有的接口。但如果你为你的 FreeNAS 服务器购买新的硬件，那么这些差异可能变得非常重要。

第一个差异是传输速度。最后一个版本的 PATA 可以以 133 MB 每秒（MB/s）的速度传输数据。这意味着 700MB 的数据（与 CD 相同）将需要略超过 5 秒的时间来复制。SATA 目前有两种速度，SATA 150，可以传输 187.5 MB/s，和 SATA 300，最高可达 375MB/s。复制一张 CD 的数据分别需要不到 4 秒和不到 2 秒。SCSI 有许多不同的变体，最新的版本 Ultra-320 和 Ultra-640 分别具有 320MB/s 和 640MB/s 的传输速度。700MB 的文件在 Ultra-640 系统上将需要略超过 1 秒的时间来传输。所有这些速度都是理论上的峰值传输速率，但它们确实提供了一个很好的传输速度差异的指导。

成本也是一个区别。PATA 和 SATA 的成本相似，目前 SATA 稍便宜一些，因为 PATA 硬盘已经不那么流行了。然而，SCSI 硬盘的历史价格一直更高。SCSI 硬盘主要被视为“专业”存储选项，因此具有更高质量的零部件，但销量较少，因此价格更高。

另一个区别在于可以连接到 PC 的硬盘数量。传统上，ATA 限制为四个硬盘。大多数主板配备两个 IDE 接口，称为主接口和从接口。每个接口可以连接两个被称为主驱动器和从驱动器的驱动器。在传统 PC 中，从接口的主驱动器是 CD 或 DVD ROM 驱动器，主接口的主驱动器是硬盘。可以通过将另一个硬盘作为主从驱动器或从从驱动器的方式来扩展 PC。

### 注意

ATA/IDE 系统的一个问题是，当两个设备连接到同一电缆时，电缆上只能有一个设备执行读取或写入操作。因此，在与使用频繁的 DVD 同一电缆上的硬盘将发现，几乎每次被要求执行传输时，它都必须等待 DVD 先完成自己的传输。如果 DVD 没有使用，则不会出现这个问题。

SATA 不同之处在于每个物理电缆只连接到一个硬盘。你的 PC 上可以连接的 SATA 硬盘数量取决于主板上有多少连接器。一些主板只有两个连接器，但也有一些主板有多达八个连接器。还可以通过在 PCI 插槽中安装另一个 SATA 控制器来添加额外的 SATA 连接器。

SCSI 是在你的 PC 或服务器上添加额外驱动器的冠军，使用标准的 SCSI 控制器可以添加最多 16 个驱动器。许多服务器通常内置 2 或 3 个 SCSI 控制器，因此最多可以添加 48 个驱动器。SCSI 是一种成熟的技术，SCSI 驱动器通常比 PATA/SATA 硬盘质量更高。然而，它们更昂贵。

#### 驱动器

在购买硬盘时，我们经常首先关注的是硬盘的容量，即多少吉字节（甚至是特字节）。但除了容量外，质量和速度也是问题。硬盘是物理机制，这意味着与计算机内存或硬盘接口相比，它们相当慢。在为你的 FreeNAS 服务器购买硬盘时，除了容量外，还有几个重要因素需要注意。

硬盘性能由三个因素决定：寻道时间、主轴转速和总体传输速度。

**寻道时间—** 为了在磁盘的特定位置读取或写入数据，磁盘的读写头需要被物理地移动到该位置。这个过程叫做寻道，移动到正确位置所需的时间就是寻道时间。寻道时间因目标位置的远近而有所不同，因此寻道时间通常表示为平均值。桌面硬盘的典型寻道时间大约是 7 毫秒到 8 毫秒（并且正在下降），而服务器或高端磁盘的寻道时间大约是 3 毫秒到 4 毫秒。这意味着在较便宜的硬盘上，查找数据或找到写入数据的正确位置所需的时间可能是较贵硬盘的两倍。我们当然是在讨论毫秒，但将其乘以磁盘上发生的寻道次数，你就会看到更高质量的磁盘能显著提升速度。对于大文件的单一传输，性能变化不大，因为一旦磁盘找到了文件（假设文件是连续的），它就可以直接读取，不需要在读取之间进行长时间的寻道。但在多用户的情况下，磁盘会尝试同时读取和写入多个文件。

**主轴速度—** 硬盘由一个或多个涂有磁性材料的盘片组成。正是这些盘片使我们得以使用“磁盘”一词，有时人们也喜欢说“光盘”。每个盘片在磁盘驱动单元内旋转，读写头在盘片表面上下摆动，读取和写入数据。盘片旋转的速度是影响磁盘性能的一个因素，原因有二：首先，当等待查找磁盘上的某个位置时，磁盘旋转越快，那个位置越快就会出现在读写头下方；其次，一旦开始读写，磁盘旋转得越快，数据写入或读取的速度也越快。如今，磁盘的旋转速度有很多种，从每分钟 4200 转（RPM）开始，接着是 5400 RPM、7200 RPM、10,000 RPM 和 15,000 RPM。10,000 RPM 和 15,000 RPM 类型的磁盘有时被称为 10K RPM 或 15K RPM 磁盘。传统上，笔记本电脑使用较慢的磁盘来节省电池电量。常见的笔记本磁盘速度是 4200 RPM 和 5400 RPM，但现在也有一些 7200 RPM 的笔记本磁盘。有趣的是，一些高端磁盘制造商现在采用 2.5 英寸笔记本格式用于服务器，因为他们发现可以在更小的物理体积中制造出更可靠、更快速、噪音更小的磁盘，并且磁盘驱动技术的进步意味着这些磁盘的容量不一定较小。桌面计算机主要使用 5400 RPM 和 7200 RPM 的磁盘，高端服务器则使用昂贵的 10K RPM 和 15K RPM 磁盘。

**传输速度—** 现在，硬盘正在高速旋转，磁头以最大速度来回飞行，真正的问题是数据能以多快的速度从硬盘读取或写入。不幸的是，这个问题并不好回答。我们可以测量几种不同类型的传输速度。首先是*内部介质传输速率*，即硬盘能够从盘片表面读取或写入数据的实际速度。这个数据对我们没有太大用处（尽管我相信也有些人觉得它很有趣），因为它只是一个*内部*传输速度，不包括任何寻址时间。它并不反映硬盘向 PC 输出的数据。一个更好的（但不完美的）硬盘性能衡量标准是硬盘的*持续传输速率*。这指的是硬盘能够从硬盘的多个部分按顺序传输数据的速度，包括寻址所需的开销。然而，值得注意的是，硬盘外部接口的速度（例如 SATA 150）**并不**反映硬盘的外部传输速率。如今最好的硬盘的传输速率低于 100MB/s。而 SATA 接口的最低速度为 150MB/s，最快为 300MB/s。许多硬盘在硬盘上包括一个内存缓存，这意味着如果条件合适，硬盘可以在短短的几毫秒内以总线速度输出数据。但这些缓存相对于硬盘的大小来说非常小，只有几兆字节。所以，通常情况下，硬盘是直接从硬盘盘片获取数据的，因此，它**永远**无法达到外部总线的速度。

#### 多硬盘驱动器

所以，如果硬盘的速度无法与硬盘接口匹配，为什么还要制定这些越来越快的规范呢？好消息是，因为你可以为 PC 安装多个硬盘，你可以通过同时从两个硬盘读取和写入来提高整体性能。这有两个方面，首先，如果你在 FreeNAS 服务器上安装多个硬盘并将它们作为独立的资源提供，那么硬盘上的负载将根据有多少人访问硬盘一上的文件和有多少人访问硬盘二上的数据进行分配。由于硬盘接口可以同时处理两个硬盘的操作，你就实现了系统性能的翻倍。第二种方法是将硬盘用作阵列，其中硬盘包含相同数据的副本，这样 FreeNAS 服务器就可以在两个地方找到相同的数据，并从较空闲的硬盘读取数据。这被称为廉价磁盘冗余阵列（RAID），我们将在本章稍后更详细地讨论它。

#### 内存、网卡、PCI 和 USB

在决定 FreeNAS 服务器硬件时，还有一些最后需要提到的事项。

第一个需要考虑的因素是内存，或者更具体地说是内存的大小。FreeNAS 的最低内存要求是 96MB，但如果你打算使用 iSCSI，则至少需要 256MB。额外的内存总是有好处的，因为 FreeNAS 会利用额外的内存进行磁盘缓存（也就是说，额外的内存将用于将常用的数据存储在内存中，从而减少从磁盘读取数据的需求）。

接下来需要考虑的是你的网络；这一部分将在本章后续内容中详细讨论，但值得在这里提到的是，你需要一个网络接口来将你的 NAS 连接到网络。许多主板自带网络接口，它们是一个不错的起点，但可能不是最佳选择。简而言之，常见的以太网网络接口有三种速度：10Mb/s、100Mb/s 和 1000Mb/s。请注意，这些速度单位是兆比特每秒**而不是**兆字节每秒。显然，网络接口的速度越快，数据从 FreeNAS 服务器的传输速度也会越快。

然而，值得一提的是，网络性能直接与 CPU（和主板）的速度相关。将一个千兆以太网卡插入一台奔腾 II 处理器的电脑，并**不会**使网络传输速度提高十倍。

若要将网络卡或可能的硬件 RAID 控制器连接到主板，你需要有空余的 PCI 插槽。99.9%的主板都有空闲的 PCI 插槽，但值得提到的是，PCI 的速度限制为 132MB/s。对于网络来说，这应该不是问题，但如果你添加了一个高速的硬件 RAID 控制器，数据传输速率可能会超过 PCI 总线的带宽。现在有一种新的 PCI 版本，叫做 PCI Express，它有不同的速度配置（1x、2x、4x、8x、16x 和 32x），但这些配置的带宽都比传统 PCI 要大得多。

最后，如果你打算使用 USB 闪存盘存储配置数据，或者希望将 FreeNAS 安装在 USB 闪存盘上，那么你的 FreeNAS 服务器需要一个 USB 2.0 端口。配置数据可以存储在软盘、USB 闪存盘或 PC 中的硬盘上。将配置数据存储在 USB 闪存盘上的好处是，你可以将硬盘完全留给存储使用。将 FreeNAS 安装在 USB 闪存盘上也有类似的好处。这样可以让所有硬盘完全用于存储，同时提高启动速度，因为从 USB 启动比从 CDROM 启动要快。请注意，如果你打算将 FreeNAS 安装在 USB 闪存盘上，你的 BIOS 需要支持从 USB 启动。

# 备份规划

在决定如何使用和部署你的 FreeNAS 服务器时，你需要考虑备份需求。备份遗憾的是常常被当作低优先级任务处理，直到突然发生问题，大家才会问备份在哪里。

FreeNAS 并没有提供许多自动化备份选项。FreeNAS 机器不支持刻录 DVD，也不支持连接磁带驱动器。这让我们只剩下两个选择：

+   第一个选择是在机器中加入足够的磁盘，以便可以在内部复制数据，第二个选择是将数据复制到另一台机器上，然后将其刻录到 DVD 或将其作为备份服务器保留。这个过程可以通过两种方式实现。可以将机器中的磁盘数量加倍，作为一项手动任务，系统管理员可以将数据从“活动”磁盘复制到“备份”磁盘。这项任务也可以自动化，更多细节将在第七章中探讨。另一种方法是使用 RAID，我们将在本章稍后更详细地研究。通过镜像技术，可以将 FreeNAS 服务器配置为将所有写入一个磁盘的数据自动复制到另一个磁盘，从而设置数据的备用副本。这个备用副本在读取操作时也会被使用，从而提高整体存储性能。

### 注意

内部复制数据作为唯一备份手段的缺点是，如果服务器本身发生故障，备份数据也会丢失。其次，这种方法（特别是 RAID 变体）无法恢复误删除的文件，也无法恢复几个月前的文件。

+   实现 FreeNAS 服务器备份的第二种方式是将数据复制到另一台机器上。这台机器可以作为备份服务器，仅保存数据，或者作为数据写入 DVD 或其他备份介质之前的中转点。FreeNAS 提供了多种方式来将数据从服务器复制出去。事实上，像 CIFS、NFS 和 FTP 等访问协议都可以用来将数据复制到另一台机器上。此外，FreeNAS 还支持 RSYNC 协议，其主要目标是实现数据从一台机器到另一台机器的镜像（或精确复制）。RSYNC 还很智能，它只会复制需要复制的数据（因为数据已发生更改），从而在可能的情况下节省网络带宽。

如果你只需要一个备份服务器，那么另一台 FreeNAS 服务器将是理想的选择。通过 RSYNC 协议，两个 FreeNAS 服务器可以配置为在特定时间进行备份。确保备份服务器不与 FreeNAS 服务器放在一起是一个好习惯。为什么？如果天花板塌了或者空调突然把东西倒到服务器上，两个机器并排放置不会对你的备份策略有太大帮助!!!

如果你需要将数据写入 DVD 或其他类型的介质，那么你需要使用像 Windows、OS X 或 Linux 计算机这样的客户端机器来复制数据，然后将其放入 DVD 等介质中。你应该调查一下在这些平台上可用的备份解决方案。

# 什么是 RAID？我需要它吗？

对科幻作家来说，似乎有一个经验法则，就是一旦飞船受到敌方攻击，所有看似都失去了，直到紧急备份系统启动，然后一切恢复到满电状态，除了可能有一些闪烁的灯光和从附近控制台冒出的偶尔火花。

幸运的是，你不需要担心你的 FreeNAS 服务器被激光束击中，但你**确实**需要担心硬盘故障、电压激增、爆裂的水管（或冷却系统）、火灾，甚至可能是地震。

RAID（冗余独立磁盘阵列）概念并不是解决所有备份和冗余问题的魔法方案。首先，它无法保护你的服务器免受地震的影响！然而，作为一种提高存储访问速度并应对磁盘故障的手段，RAID 是一个优秀的系统。

那么，它是什么呢？RAID 是一种将数据分割并复制到多个硬盘上的系统，从而提供冗余并消除读取瓶颈。根据使用的方案，你的数据将会被完整或部分复制到 RAID 阵列中的其他磁盘上，如果其中一块磁盘发生故障，其他磁盘（上面有数据的副本）将继续工作，整个数据保持完整。RAID 提供了两个主要优势：一是数据的复制，防止磁盘故障；二是存储性能的提升，因为多个磁盘可以用来读取相同的数据，并迅速返回给客户端。最简单的 RAID 设置需要至少两块硬盘。

有几种不同的 RAID 配置，称为 RAID 等级。最初，有 5 种 RAID 等级，其中两个最常用（RAID 1 和 RAID 5）。还有一些扩展功能，改进、组合或嵌套了原始的 RAID 等级。如果在使用冗余的 RAID 等级的阵列中某个磁盘发生故障，当插入新磁盘替代故障磁盘时，它将会从其他磁盘中重建数据，阵列将重新变为完全可用。也可以指定某个磁盘作为“热备件”，它会在没有外部干预的情况下自动替代故障磁盘。然后，故障磁盘可以被替换，并成为“热备件”。对于 RAID，所有磁盘需要具有相同的大小。对于 FreeNAS 中的软件 RAID 功能，如果磁盘的大小不同，则会使用较小的磁盘大小。

下面是最常见的 RAID 等级概述。并非所有这些 RAID 等级都可以通过 FreeNAS 的软件 RAID 功能实现，但如果你使用的是专用的硬件 RAID 卡，它们可能是可用的。

**RAID 0**（无冗余的条带化集群）—RAID 0 是一种将两个硬盘连接在一起以创建一个大硬盘的方法。数据在两个硬盘之间交错分布，因此提高了性能，但没有容错能力。任何硬盘故障都会摧毁整个阵列，数据将丢失。单独使用时，这种配置并不十分有用，但可以与 RAID 1 结合形成 RAID 10 系统。请见下文。

**RAID 1**（镜像）—在这种配置中，使用两个硬盘，其中一个硬盘镜像另一个硬盘的内容。写入到硬盘 1 的任何数据都会同时写入硬盘 2，方式完全相同。如果任一硬盘发生故障，RAID 继续使用剩余的硬盘。故障硬盘被替换后，新的硬盘将与好的硬盘同步，镜像过程将继续。写入性能通常会稍差于单一设备，因为需要将数据的相同副本写入另一块硬盘，但读取性能大大提高，因为数据在两个地方可用，读取操作可以在两个硬盘之间分配。可用的存储空间始终是两块硬盘总容量的一半，因为其中一块硬盘用于复制另一块硬盘。

**RAID 5**（带分布奇偶校验的条带化集群）—这是最常见、也许是最实用的 RAID 级别之一。它允许你结合多个物理硬盘，同时保持一定的冗余性。RAID 5 可以在三个或更多硬盘上使用。如果一个硬盘发生故障，数据仍然完好无损。RAID 5 可以容忍一个硬盘故障，但不能容忍两个或更多硬盘故障。RAID 5 的读取和写入性能通常都会提高。RAID 5 阵列的大小等于最小硬盘的大小乘以硬盘数量减去 1。如果你有三个 200GB 的硬盘，则阵列的总大小为 200 X (3-1) = 400GB。

**RAID 6**（带有双重分布奇偶校验的条带化集群）—这与 RAID 5 非常相似，只不过数据会分布到另外两个硬盘上，这意味着阵列可以从两个硬盘的故障中恢复。RAID 6 可以在四个或更多硬盘上使用。RAID 6 阵列的大小等于硬盘大小乘以硬盘数量减去 2。如果你有四个 250GB 的硬盘，则阵列的总大小为 250 X (4-2) = 500GB。

**RAID 10**（镜像条带化集群）—这种配置有时称为 RAID 1+0，是由两个 RAID 0 阵列组成的 RAID 1 阵列。因此，硬盘 1 和硬盘 2 以 RAID 0 配置形成一个大硬盘 A1。硬盘 3 和硬盘 4 也以 RAID 0 配置形成一个组合硬盘，称为 A2。然后 A1 和 A2 被用来在 RAID 1 配置中互为镜像。RAID 10 阵列能够承受多个硬盘故障，只要这两块故障硬盘属于**同一** RAID 0 阵列。读取性能较好，写入性能优于 RAID 1，因为数据写入是交错分布在镜像中的两个硬盘上的。

## 硬件 RAID 或软件 RAID

RAID 有两种形式。一种是由专用硬件控制的阵列，通常称为 RAID 控制器，另一种是由软件控制的阵列，更具体地说，是由操作系统控制，在这种情况下是 FreeBSD。

如果你有硬件 RAID 控制器，那么你可以使用它。FreeBSD 支持许多硬件 RAID 控制器，且该控制器负责管理 RAID 阵列。使用 RAID 控制器时，性能得到保障，因为没有额外的开销添加到本地 CPU 上来运行阵列。控制器仅向操作系统呈现一个逻辑磁盘，FreeNAS 将其视为一个磁盘，不管阵列由多少个磁盘组成。一些 RAID 卡还支持热插拔；允许在系统运行时更换故障的硬盘。

### 注意

**小心“假”硬件 RAID 控制器**

有一种便宜的 RAID 控制器，它没有专门的硬件来管理 RAID 阵列，而是使用标准的磁盘控制器芯片与特殊的固件和驱动程序相结合。虽然被称为 RAID 控制器，但 RAID 处理的负担被放在 CPU 上，而非 RAID 控制器本身。这些控制器通常被称为“假”RAID 控制器，并非因为它们没有正确实现 RAID，而是因为它们不是“真正”的硬件 RAID 控制器。FreeNAS 不支持这种类型的 RAID 控制器。

如果你没有 RAID 控制器，那么你可以使用 FreeNAS（在 FreeBSD 的帮助下）来运行 RAID 阵列。这不需要额外的控制器，并且 FreeNAS 自带此功能。我们将在第六章中详细讲解如何在 FreeNAS 上配置软件 RAID。

# 网络考虑因素

由于 FreeNAS 是**网络**附加存储，部署服务器时一个重要的因素是网络基础设施。如果你的网络速度慢，你的 FreeNAS 服务器对用户的响应也会显得很慢。

所有现代的局域网（LAN）都使用 Ethernet 或无线网络。这里的“局域”是指局限于一个本地区域，通常是一个建筑物，比如房屋或办公楼。Ethernet 发明于 1980 年代，并已成为家庭和办公室有线网络的标准。最初，它使用同轴电缆连接各个机器，但今天每台机器都通过一种叫做 5 类电缆（通常称为 Cat 5 电缆）的 8 根线直接连接到集线器或交换机。

集线器或交换机充当分发点，将网络流量转发给客户端。如今，Ethernet 网络有 3 种主要速度。最初的以太网网络工作在 10Mb/s（注意是兆比特而非兆字节），称为 10BaseT，接着是 100Mb/s 网络，称为 100BaseT，最近则是 1000Mb/s，也就是 1000BaseT。100BaseT 通常被称为快速以太网，而 1000BaseT 则是**千兆**以太网。

简单来说，你希望在 FreeNAS 服务器上使用最快的网络连接，这意味着目前使用千兆以太网。这并不意味着你需要在整个网络中都使用千兆以太网。如果你的网络已经使用了快速以太网，那么你不需要更换网络中的所有网卡和交换机。但是，你确实需要确保你的 FreeNAS 服务器配备了千兆网卡，并且连接到一个**优质**的千兆交换机。

这意味着，如果两台 PC（每台都具有快速以太网）正在从 FreeNAS 服务器复制数据，那么两者都会使用其最大可用的网络带宽（100Mb/s），但 FreeNAS 服务器因为使用的是千兆以太网，所以能够处理这两个请求（假设它拥有足够的硬件支持，如 CPU、磁盘等）。

## 交换机还是集线器？

要将你的 FreeNAS 服务器连接到网络，需要将它连接到一种叫做集线器或交换机的网络设备。

今天集线器相当少见，但从本质上来说，它是一个广播设备，将所有进入某个端口的分组广播到其他所有端口。它不会尝试以任何方式管理流量，也不会检查是否适合将某个特定分组转发出去。结果，网络碰撞会发生。这意味着 PC 在尝试使用网络时，其他 PC 也在同时使用网络。以太网有内建的系统来处理这个问题，但它会减慢流量的传输速度。

由于碰撞问题，在 10Mb/s 网络中只允许使用 4 个集线器，在 100Mb/s 网络中只允许使用 2 个集线器。对于 100Mb/s 及更高速率的网络，使用交换机要比集线器好得多。

在 10Mb/s 和 100Mb/s 网络之间交替使用时，产生了一种混合集线器，叫做双速集线器。实际上，这个集线器是两个较小的集线器（一个是 10Mb/s 集线器，另一个是 100Mb/s 集线器）组合成一个单元，它们之间通过一个链接连接。具有 100Mb/s 连接的 PC 将连接到 100Mb/s 集线器，而具有 10Mb/s 连接的 PC 将连接到 10Mb/s 集线器。它们之间有一个两端口桥接器。像单速集线器一样，由于快速以太网交换机的普及和低成本，这种设备今天已经很少见了。

交换机与集线器的不同之处在于，交换机会检查通过某个端口传入的流量，并计算应该将其发送到哪里。通过仅将每个分组发送到它本应发送的设备，网络交换机能够节省网络带宽，并且通常比集线器提供更好的性能。

### 注意

**警惕便宜的千兆交换机**

需要注意的是，虽然有些交换机自称为千兆以太网交换机，但这并不一定意味着它能够处理 1000Mb/s 的流量。一些便宜的交换机虽然能够理解 1000MB/s 网络的语言，因此它们被称为千兆以太网交换机，但它们并不提供完整的带宽。

千兆交换机的关键特点是支持巨型帧（即大帧）。没有巨型帧支持时，超过 100Mb/s 的网络提升将会很小。

## 关于无线连接？

无线连接如今已成为将家庭 PC 连接到 DSL（或 ADSL）互联网调制解调器以及连接家中各种设备（如互联网无线电设备、笔记本电脑甚至多媒体中心）的流行媒介。您可以将这些设备连接到电视，并通过无线连接流式传输音乐和电影到您的台式电脑和互联网。

FreeNAS 确实支持多种无线设备，并且在 Web 界面中支持配置无线卡。

然而，无线连接并非连接到 FreeNAS 的最佳方法。首先，无线带宽有限（54Mb/s），只有 100Mb/s 的一半。其次，从无线接入点到服务器的距离越远，带宽就会下降，特别是如果信号需要穿过墙壁时。实际结果是服务器需要靠近无线接入点，而且既然如此接近，您应该能够通过电缆连接到您的网络。

值得重申的是，FreeNAS 服务器最适合连接到千兆以太网交换机。

# 总结

在本章中，我们已经看到了您需要为组织中部署服务器而做出的决策。我们根据用户数量考虑了硬件要求，并研究了您可以放置在服务器中的不同类型的磁盘。我们还考虑了对您的网络可能产生的一些影响。

在下一章中，我们将亲手操作 FreeNAS 服务器并进行安装。
