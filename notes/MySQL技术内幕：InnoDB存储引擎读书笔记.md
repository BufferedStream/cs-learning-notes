## MySQL技术内幕：InnoDB存储引擎读书笔记



### 前言

MySQL 数据库独有的**插件式存储引擎架构**使其和其他任何数据库都不同。不同的存储引擎有着完全不同的功能，而 InnoDB 存储引擎的存在使得 MySQL 跃入了企业级数据库领域。本书完整地讲解了 InnoDB 存储引擎中最重要的一些内容，即 InnoDB 的体系结构和工作原理，并结合 InnoDB 的源代码讲解了它的内部实现机制。

本书不仅讲述了 InnoDB 存储引擎的诸多功能和特性，还阐述了如何正确地使用这些功能和特性，更重要的是，还尝试了教我们如何 Think Different。Think Different 是 20 世纪 90 年代苹果公司在其旷日持久的宣传活动中提出的一个口号，借此来重振公司的品牌，更重要的是，这个口号改变了人们对技术在日常生活中的作用的看法。需要注意的是，苹果的口号不是 Think Differently，是 Think Different，Different 在这里做名词，意味该思考些什么。

很多 DBA 和开发人员都相信某些 “神话”，然而这些 “神话” 往往都是错误的。无论计算机技术发展的速度变得多快，数据库的使用变得多么简单，任何时候 Why 都比 What 重要。只有真正理解了内部实现原理、体系结构，才能更好地去使用。这正是人类正确思考问题的原则。因此，对于当前出现的技术，尽管学习其应用很重要，但更重要的是，应当正确地理解和使用这些技术。关于本书，我的头脑里有很多个目标，但最重要的是想告诉大家如下几个简单的观点：

- 不要相信任何的“神话”，学会自己思考；
- 不要墨守成规，大部分人都知道的事情可能是错误的；
- 不要相信网上的传言，去测试，根据自己的实践做出决定；
- 花时间充分地思考，敢于提出质疑。

当前有关 MySQL 的书籍大部分都集中在教读者如何使用 MySQL，例如 SQL 语句的使用、复制的搭建的、数据的切分等。没错，这对快速掌握和使用 MySQL 数据库非常有好处，但是真正的数据库工作者需要了解的不仅仅是应用，更多的是内部的具体实现。

MySQL 数据库独有的插件式存储引擎使得想要在一本书内完整地讲解各个存储引擎变得十分困难，有的书可能偏重对 MyISAM 的介绍，有的可能偏重对 InnoDB 存储引擎的介绍。对于初级的 DBA 来说，这可能会使他们的理解变得更困难。对于大多数 MySQL DBA 和开发人员来说，他们往往更希望了解作为 MySQL 企业级数据库应用的第一存储引擎的 InnoDB，我想在本书中，他们完全可以找到他们希望了解的内容。

再强调一遍，任何时候 Why 都比 What 重要，本书从源代码的角度对 InnoDB 的存储引擎的整个体系架构的各个组成部分进行了系统的分析和讲解，剖析了 InnoDB 存储引擎的核心实现和工作机制，相信这在其他书中是很难找到的。



### 第1章 MySQL体系结构和存储引擎

MySQL 被设计为一个可移植的数据库，几乎在当前所有系统上都能运行，如 Linux，Solaris、FreeBSD、Mac 和 Windows。尽管各平台在底层（如线程）实现方面都各有不同，但是 MySQL 基本上能保证在各平台上的物理体系结构的一致性。因此，用户应该能很好地理解 MySQL 数据库在所有这些平台上是如何运作的。



#### 1.1 定义数据库和实例

在数据库领域中有两个词很容易混淆，这就是 “数据库”（database）和 “实例”（instance）。作为常见的数据库术语，这两个词的定义如下。

- **数据库**：物理操作系统文件或其他形式文件类型的集合。在 MySQL 数据库中，数据库文件可以是 frm、MYD、MYI、ibd 结尾的文件。当使用 NDB 引擎时，数据库的文件可能不是操作系统上的文件，而是存放于内存之中的文件，但是定义仍然不变。
- **实例**：MySQL 数据库由后台线程以及一个共享内存区组成。共享内存可以被运行的后台线程所共享。需要牢记的是，数据库实例才是真正用于操作数据库文件的。

这两个词有时可以互换使用，不过两者的概念完全不同。在 MySQL 数据库中，实例与数据库的关通常系是一一对应的，即一个实例对应一个数据库，一个数据库对应一个实例。但是，在集群情况下可能存在一个数据库被多个数据实例使用的情况。

MySQL 被设计为一个**单进程多线程架构**的数据库，这点与 SQL Server 比较类似，但与 Oracle 多进程的架构有所不同（Oracle 的 Windows 版本也是单进程多线程架构的）。这也就是说，MySQL 数据库实例在系统上的表现就是一个进程。

在 Linux 操作系统中通过以下命令启动 MySQL 数据库实例，并通过命令 ps 观察 MySQL 数据库启动后的进程情况：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE1.jpg"/>
</div>

注意进程号为 3578 的进程，该进程就是 MySQL 实例。在上述例子中使用了 mysqld_safe 命令来启动数据库，当然启动 MySQL 实例的方法还有很多，在各种平台下的方式可能又会有所不同。在这里不一一赘述。

当启动实例时，MySQL 数据库会去读取配置文件，根据配置文件的参数来启动数据库实例。这与 Oracle 的参数文件（spfile）相似，不同的是，Oracle 中如果没有参数文件，在启动实例时会提示找不到该参数文件，数据库启动失败。而在 MySQL 数据库中，可以没有配置文件，在这种情况下，MySQL 会按照编译时的默认参数设置启动实例。用以下命令可以查看当 MySQL 数据库实例启动时，会在哪些位置查找配置文件。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE2.jpg"/>
</div>

可以看到，MySQL 数据库是按 /etc/my.cnf→/etc/mysql/my.cnf→/usr/local/mysql/etc/my.cnf→～/.my.cnf 的顺序读取配置文件的。可能有读者会问：“如果几个配置文件中都有同一个参数，MySQL 数据库以哪个配置文件为准？”答案很简单，MySQL 数据库会以读取到的最后一个配置文件中的参数为准。在 Linux 环境下，配置文件一般放在 /etc/my.cnf 下。在 Windows 平台下，配置文件的后缀名可能是 .cnf，也可能是 .ini。例如在 Windows 操作系统下运行 mysql--help，可以找到如下类似内容：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE3.jpg"/>
</div>

配置文件中有一个参数 datadir，该参数指定了数据库所在的路径。在 Linux 操作系统下默认 datadir 为 /usr/local/mysql/data，用户可以修改该参数，当然也可以使用该路径，不过该路径只是一个链接，具体如下：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE4.jpg"/>
</div>

从上面可以看到，其实 data 目录是一个链接，该链接指向了 /opt/mysql_data 目录。当然，用户必须保证 /opt/mysql_data 的用户和权限，使得只有 mysql 用户和组可以访问（通常 MySQL 数据库的权限为 mysql∶mysql）。





#### 1.2 MySQL体系结构

由于工作的缘故，笔者的大部分时间需要与开发人员进行数据库方面的沟通，并对他们进行培训。不论他们是 DBA，还是开发人员，似乎都对 MySQL 的体系结构了解得不够透彻。很多人喜欢把 MySQL 与他们以前使用的 SQL Server、Oracle、DB2 作比较。因此笔者常常会听到这样的疑问：

- 为什么 MySQL 不支持全文索引？
- MySQL 速度快是因为它不支持事务吗？
- 数据量大于 1000 万时 MySQL 的性能会急剧下降吗？

........



对于 MySQL 数据库的疑问有很多很多，在解释这些问题之前，笔者认为不管对于使用哪种数据库的开发人员，了解数据库的体系结构都是最为重要的内容。

在给出体系结构图之前，用户应该理解了前一节提出的两个概念：数据库和数据库实例。很多人会把这两个概念混淆，即 MySQL 是数据库，MySQL 也是数据库实例。这样来理解 Oracle 和 Microsoft SQL Server 数据库可能是正确的，但是这会给以后理解 MySQL 体系结构中的存储引擎带来问题。从概念上来说，数据库是文件的集合，是依照某种数据模型组织起来并存放于二级存储器中的数据集合；数据库实例是程序，是位于用户与操作系统之间的一层数据管理软件，用户对数据库数据的任何操作，包括数据库定义、数据查询、数据维护、数据库运行控制等都是在数据库实例下进行的，应用程序只有通过数据库实例才能和数据库打交道。

如果这样讲解后读者还是不明白，那这里再换一种更为直白的方式来解释：数据库是由一个个文件组成（一般来说都是二进制的文件）的，要对这些文件执行诸如 SELECT、INSERT、UPDATE 和 DELETE 之类的数据库操作是不能通过简单的操作文件来更改数据库的内容，需要通过数据库实例来完成对数据库的操作。所以，用户把 Oracle、SQL Server、MySQL 简单地理解成数据库可能是有失偏颇的，虽然在实际使用中并不会这么强调两者之间的区别。

好了，在给出上述这些复杂枯燥的定义后，现在可以来看看 MySQL 数据库的体系结构了，其结构如图 1-1 所示（摘自 MySQL 官方手册）。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE5.jpg"/>
</div>

从图 1-1 可以发现，MySQL 由以下几部分组成：

- 连接池组件
- 管理服务和工具组件
- SQL 接口组件
- 查询分析器组件
- 优化器组件
- 缓冲（Cache）组件
- 插件式存储引擎
- 物理文件

从图 1-1 还可以发现，MySQL 数据库区别于其他数据库的最重要的一个特点就是其插件式的表存储引擎。MySQL 插件式的存储引擎架构提供了一系列标准的管理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如 SQL 分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引擎开发者可以按照自己的意愿来进行开发。

**需要特别注意的是，存储引擎是基于表的，而不是数据库。**此外，要牢记图 1-1 的 MySQL 体系结构，它对于以后深入理解 MySQL 数据库会有极大的帮助。



#### 1.3 MySQL存储引擎

通过 1.2 节大致了解了 MySQL 数据库独有的插件式体系结构，并了解到存储引擎是 MySQL 区别于其他数据库的一个最重要特性。存储引擎的好处是，每个存储引擎都有各自的特点，能够根据具体的应用建立不同存储引擎表。对于开发人员来说，存储引擎对其是透明的，但了解各种存储引擎的区别对于开发人员来说也是有好处的。对于 DBA 来说，他们应该深刻地认识到 MySQL 数据库的核心在于存储引擎。

由于 MySQL 数据库的开源特性，用户可以根据 MySQL 预定义的存储引擎接口编写自己的存储引擎。若用户对某一种存储引擎的性能或功能不满意，可以通过修改源码来得到想要的特性，这就是开源带给我们的方便与力量。比如，eBay 的工程师 Igor Chernyshev 对 MySQL Memory 存储引擎的改进（http：//code.google.com/p/mysql-heap-dynamic-rows/）并应用于 eBay 的 Personalization Platform，类似的修改还有 Google 和 Facebook 等公司。笔者曾尝试过对 InnoDB 存储引擎的缓冲池进行扩展，为其添加了基于 SSD 的辅助缓冲，通过利用 SSD 的高随机读取性能来进一步提高数据库本身的性能。当然，MySQL 数据库自身提供的存储引擎已经足够满足绝大多数应用的需求。如果用户有兴趣，完全可以开发自己的存储引擎，满足自己特定的需求。MySQL 官方手册的第 16 章给出了编写自定义存储引擎的过程，不过这已超出了本书所涵盖的范围。

由于 MySQL 数据库开源特性，存储引擎可以分为 MySQL 官方存储引擎和第三方存储引擎。有些第三方存储引擎很强大，如大名鼎鼎的 InnoDB 存储引擎（最早是第三方存储引擎，后被 Oracle 收购），其应用就极其广泛，甚至是 MySQL 数据库 OLTP（Online Transaction Processing 在线事务处理）应用中使用最广泛的存储引擎。还是那句话，用户应该根据具体的应用选择适合的存储引擎，以下是对一些存储引擎的简单介绍，以便于读者选择存储引擎时参考。



##### 1.3.1 InnoDB存储引擎

InnoDB 存储引擎支持事务，其设计目标主要面向**在线事务处理（OLTP）**的应用（on-line transaction processing）。其特点是**行锁设计、支持外键**，并支持类似于 Oracle 的**非锁定读**，即默认读取操作不会产生锁。从 MySQL 数据库 5.5.8 版本开始，InnoDB 存储引擎是默认的存储引擎。

InnoDB 存储引擎将数据放在一个逻辑的表空间中，这个表空间就像黑盒一样由 InnoDB 存储引擎自身进行管理。从 MySQL 4.1（包括 4.1）版本开始，它可以将每个 InnoDB 存储引擎的表单独存放到一个独立的 ibd 文件中。此外，InnoDB 存储引擎支持用裸设备（row disk）用来建立其表空间。

InnoDB 通过使用多版本并发控制（MVCC，Multi-Version Concurrency Control）来获得高并发性，并且实现了 SQL 标准的 4 种隔离级别，默认为 REPEATABLE 级别。同时，使用一种被称为 next-keylocking 的策略来避免幻读（phantom）现象的产生。除此之外，InnoDB 储存引擎还提供了插入缓冲（insert buffer）、二次写（double write）、自适应哈希索引（adaptive hash index）、预读（read ahead）等高性能和高可用的功能。

对于表中数据的存储，InnoDB 存储引擎采用了聚集（clustered）的方式，因此每张表的存储都是按主键的顺序进行存放。**如果没有显式地在表定义时指定主键，InnoDB 存储引擎会为每一行生成一个 6 字节的 ROWID，并以此作为主键。**

InnoDB 存储引擎是 MySQL 数据库最为常用的一种引擎，而 Facebook、Google、Yahoo！ 等公司的成功应用已经证明了 InnoDB 存储引擎具备的高可用性、高性能以及高可扩展性。



##### 1.3.2 MyISAM存储引擎

MyISAM 存储引擎不支持事务、表锁设计，支持全文索引，主要面向一些**在线分析处理 OLAP** 数据库应用（On-Line Analytical Processing）。在 MySQL 5.5.8 版本之前 MyISAM 存储引擎是默认的存储引擎（除 Windows 版本外）。**数据库系统与文件系统很大的一个不同之处在于对事务的支持**，然而 MyISAM 存储引擎是不支持事务的。究其根本，这也不是很难理解。试想用户是否在所有的应用中都需要事务呢？在数据仓库中，如果没有 ETL 这些操作，只是简单的报表查询是否还需要事务的支持呢？（**注释**：字面含义：ETL 是 Extract－抽取、Transformation－转换、Load－载入的缩写。简单定义：将数据从 OLTP 系统中转移到数据仓库中的一系列操作的集合。）此外，MyISAM 存储引擎的另一个与众不同的地方是它的**缓冲池只缓存（cache）索引文件**，而不缓冲数据文件，这点和大多数的数据库都非常不同。

MyISAM 存储引擎表由 MYD 和 MYI 组成，MYD 用来存放数据文件，MYI 用来存放索引文件。可以通过使用 myisampack 工具来进一步压缩数据文件，因为 myisampack 工具使用赫夫曼（Huffman）编码静态算法来压缩数据，因此使用 myisampack 工具压缩后的表是只读的，当然用户也可以通过 myisampack 来解压数据文件。

在 MySQL 5.0 版本之前，MyISAM 默认支持的表大小为 4GB，如果需要支持大于 4GB 的 MyISAM 表时，则需要制定 MAX_ROWS 和 A VG_ROW_LENGTH 属性。从 MySQL 5.0 版本开始，MyISAM 默认支持 256TB 的单表数据，这足够满足一般应用需求。

**注意** 对于 MyISAM 存储引擎表，MySQL 数据库只缓存其索引文件，数据文件的缓存交由操作系统本身来完成，这与其他使用 LRU 算法缓存数据的大部分数据库大不相同。此外，在 MySQL 5.1.23 版本之前，无论是在 32 位还是 64 位操作系统环境下，缓存索引的缓冲区最大只能设置为 4GB。在之后的版本中，64 位系统可以支持大于 4GB 的索引缓冲区。



##### 1.3.3 NDB存储引擎

2003 年，MySQL AB 公司从 Sony Ericsson 公司收购了 NDB 集群引擎（见图 1-1）。NDB 存储引擎是一个集群存储引擎，类似于 Oracle 的 RAC 集群，不过与 Oracle RAC share everything 架构不同的是，其结构是 share nothing 的集群架构，因此能提供更高的可用性。NDB 的特点是数据全部放在内存中（从 MySQL5.1 版本开始，可以将非索引数据放在磁盘上），因此主键查找（primary keylookups）的速度极快，并且通过添加 NDB 数据存储节点（Data Node）可以线性地提高数据库性能，是高可用、高性能的集群系统。

关于 NDB 存储引擎，有一个问题值得注意，那就是 NDB 存储引擎的连接操作（JOIN）是在 MySQL 数据库层完成的，而不是在存储引擎层完成的。这意味着，复杂的连接操作需要巨大的网络开销，因此查询速度很慢。如果解决了这个问题，NDB 存储引擎的市场应该是非常巨大的。

**注意** MySQL NDB Cluster 存储引擎有社区版本和企业版本两种，并且 NDB Cluster 已作为 Carrier Grade Edition 单独下载版本而存在，可以通过 http：//dev. mysql.com/downloads/cluster/index.html 获得最新版本的 NDB Cluster 存储引擎。



##### 1.3.4 Memory存储引擎

Memory 存储引擎（之前称 HEAP 存储引擎）将表中的数据存放在内存中，如果数据库重启或发生崩溃，表中的数据都将消失。它非常适合用于存储临时数据的临时表，以及数据仓库中的纬度表。Memory 存储引擎默认使用哈希索引，而不是我们熟悉的 B+ 树索引。

虽然 Memory 存储引擎速度非常快，但在使用上还是有一定的限制。比如，只支持表锁，并发性能较差，并且不支持 TEXT 和 BLOB 列类型。最重要的是，存储变长字段（varchar）时是按照定常字段（char）的方式进行的，因此会浪费内存（这个问题之前已经提到，eBay 的工程师 Igor Chernyshev 已经给出了 patch 解决方案）。

此外有一点容易被忽视，MySQL 数据库使用 Memory 存储引擎作为临时表来存放查询的中间结果集（intermediate result）。如果中间结果集大于 Memory 存储引擎表的容量设置，又或者中间结果含有 TEXT 或 BLOB 列类型字段，则 MySQL 数据库会把其转换到 MyISAM 存储引擎表而存放到磁盘中。之前提到 MyISAM 不缓存数据文件，因此这时产生的临时表的性能对于查询会有损失。



##### 1.3.5 Archive存储引擎

Archive 存储引擎只支持 INSERT 和 SELECT 操作，从 MySQL 5.1 开始支持索引。Archive 存储引擎使用 zlib 算法将数据行（row）进行压缩后存储，压缩比一般可达 1∶10。正如其名字所示，Archive 存储引擎非常适合存储归档数据，如日志信息。Archive 存储引擎使用行锁来实现高并发的插入操作，但是其本身并不是事务安全的存储引擎，其设计目标主要是提供高速的插入和压缩功能。



##### 1.3.6 Federated存储引擎

Federated 存储引擎表并不存放数据，它只是指向一台远程 MySQL 数据库服务器上的表。这非常类似于 SQL Server 的链接服务器和 Oracle 的透明网关，不同的是，当前 Federated 存储引擎只支持 MySQL 数据库表，不支持异构数据库表。



##### 1.3.7 Maria存储引擎

Maria 存储引擎是新开发的引擎，设计目标主要是用来取代原有的 MyISAM 存储引擎，从而成为 MySQL 的默认存储引擎。Maria 存储引擎的开发者是 MySQL 的创始人之一的 Michael Widenius。因此，它可以看做是 MyISAM 的后续版本。Maria 存储引擎的特点是：支持缓存数据和索引文件，应用了行锁设计，提供了 MVCC 功能，支持事务和非事务安全的选项，以及更好的 BLOB 字符类型的处理性能。



##### 1.3.8 其他存储引擎

除了上面提到的 7 种存储引擎外，MySQL 数据库还有很多其他的存储引擎，包括 Merge、CSV、Sphinx 和 Infobright，它们都有各自使用的场合，这里不再一一介绍。在了解 MySQL 数据库拥有这么多存储引擎后，现在我可以回答 1.2 节中提到的问题了。

- 为什么 MySQL 数据库不支持全文索引？不！MySQL 支持，MyISAM、InnoDB（1.2 版本）和 Sphinx 存储引擎都支持全文索引。
- MySQL 数据库速度快是因为不支持事务？错！虽然 MySQL 的 MyISAM 存储引擎不支持事务，但是 InnoDB 支持。“快” 是相对于不同应用来说的，对于 ETL 这种操作，MyISAM 会有其优势，但在 OLTP 环境中，InnoDB 存储引擎的效率更好。
- 当表的数据量大于 1000 万时 MySQL 的性能会急剧下降吗？不！MySQL 是数据库，不是文件，随着数据行数的增加，性能当然会有所下降，但是这些下降不是线性的，如果用户选择了正确的存储引擎，以及正确的配置，再多的数据量 MySQL 也能承受。如官方手册上提及的，Mytrix 和 Inc. 在 InnoDB 上存储超过 1 TB 的数据，还有一些其他网站使用 InnoDB 存储引擎，处理插入/更新的操作平均 800 次/秒。



#### 1.4 各存储引擎之间的比较

通过 1.3 节的介绍，我们了解了存储引擎是 MySQL 体系结构的核心。本节我们将通过简单比较几个存储引擎来让读者更直观地理解存储引擎的概念。图 1-2 取自于 MySQL 的官方手册，展现了一些常用 MySQL 存储引擎之间的不同之处，包括存储容量的限制、事务支持、锁的粒度、MVCC 支持、支持的索引、备份和复制等。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE6.jpg"/>
</div>

可以看到，每种存储引擎的实现都不相同。有些竟然不支持事务，相信在任何一本关于数据库原理的书中，可能都会提到数据库与传统文件系统的最大区别在于数据库是支持事务的。而 MySQL 数据库的设计者在开发时却认为可能不是所有的应用都需要事务，所以存在不支持事务的存储引擎。更有不明其理的人把 MySQL 称做文件系统数据库，其实不然，只是 MySQL 数据库的设计思想和存储引擎的关系可能让人产生了理解上的偏差。

可以通过 SHOW ENGINES 语句查看当前使用的 MySQL 数据库所支持的存储引擎，也可以通过查找 information_schema 架构下的 ENGINES 表，如下所示：

略



下面将通过 MySQL 提供的示例数据库来简单显示各存储引擎之间的不同。这里将分别运行以下语句，然后统计每次使用各存储引擎后表的大小。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE7.jpg"/>
</div>

通过每次的统计，可以发现当最初表使用 MyISAM 存储引擎时，表的大小为 40.7MB，使用 InnoDB 存储引擎时表增大到了 113.6MB，而使用 Archive 存储引擎时表的大小却只有 20.2MB。该例子只从表的大小方面简单地揭示了各存储引擎的不同。

注意 MySQL 提供了一个非常好的用来演示 MySQL 各项功能的示例数据库，如 SQL Server 提供的 AdventureWorks 示例数据库和 Oracle 提供的示例数据库。据我所知，知道 MySQL 示例数据库的人很少，可能是因为这个示例数据库没有在安装的时候提示用户是否安装（如 Oracle 和 SQLServer）以及这个示例数据库的下载竟然和文档放在一起。用户可以通过以下地址找到并下载示例数据库：http：//dev.mysql.com/doc/。



#### 1.5 连接MySQL

本节将介绍连接 MySQL 数据库的常用方式。需要理解的是，连接 MySQL 操作是一个连接进程和 MySQL 数据库实例进行通信。从程序设计的角度来说，本质上是进程通信。如果对进程通信比较了解，可以知道常用的进程通信方式有管道、命名管道、命名字、TCP/IP 套接字、UNIX 域套接字。MySQL 数据库提供的连接方式从本质上看都是上述提及的进程通信方式。



##### 1.5.1 TCP/IP

TCP/IP 套接字方式是 MySQL 数据库在任何平台下都提供的连接方式，也是网络中使用得最多的一种方式。这种方式在 TCP/IP 连接上建立一个基于网络的连接请求，一般情况下客户端（client）在一台服务器上，而 MySQL 实例（server）在另一台服务器上，这两台机器通过一个 TCP/IP 网络连接。例如用户可以在 Windows 服务器下请求一台远程 Linux 服务器下的 MySQL 实例，如下所示：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE8.jpg"/>
</div>

这里的客户端是 Windows，它向一台 Host IP 为 192.168.0.101 的 MySQL 实例发起了 TCP/IP 连接请求，并且连接成功。之后就可以对 MySQL 数据库进行一些数据库操作，如 DDL 和 DML 等。

这里需要注意的是，在通过 TCP/IP 连接到 MySQL 实例时，MySQL 数据库会先检查一张权限视图，用来判断发起请求的客户端 IP 是否允许连接到 MySQL 实例。该视图在 mysql 架构下，表名为 user，如下所示：

略



从这张权限表中可以看到，MySQL 允许 david 这个用户在任何 IP 段下连接该实例，并且不需要密码。此外，给出了 root 用户在各个网段下的访问控制权限。



##### 1.5.2 命名管道和共享内存

在 Windows 2000、Windows XP、Windows 2003 和 Windows Vista 以及在此之上的平台上，如果两个需要进程通信的进程在同一台服务器上，那么可以使用命名管道，Microsoft SQL Server 数据库默认安装后的本地连接也是使用命名管道。在MySQL 数据库中须在配置文件中启用 --enable-named-pipe 选项。在 MySQL 4.1 之后的版本中，MySQL 还提供了共享内存的连接方式，这是通过在配置文件中添加 --shared-memory 实现的。如果想使用共享内存的方式，在连接时，MySQL 客户端还必须使用 --protocol=memory 选项。



##### 1.5.3 UNIX域套接字

在 Linux 和 UNIX 环境下，还可以使用 UNIX 域套接字。UNIX 域套接字其实不是一个网络协议，所以只能在 MySQL 客户端和数据库实例在一台服务器上的情况下使用。用户可以在配置文件中指定套接字文件的路径，如 --socket=/tmp/mysql.sock。当数据库实例启动后，用户可以通过下列命令来进行 UNIX 域套接字文件的查找：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE9.jpg"/>
</div>



#### 1.6 小结

本章首先介绍了数据库和数据库实例的定义，紧接着分析了 MySQL 数据库的体系结构，从而进一步突出强调了 “实例” 和 “数据库” 的区别。相信不管是 MySQLDBA 还是 MySQL 的开发人员都应该从宏观上了解了 MySQL 体系结构，特别是 MySQL 独有的插件式存储引擎的概念。因为很多 MySQL 用户很少意识到这一点，这给他们的管理、使用和开发带来了困扰。

本章还详细讲解了各种常见的表存储引擎的特性、适用情况以及它们之间的区别，以便于大家在选择存储引擎时作为参考。最后强调一点，虽然 MySQL 有许多的存储引擎，但是它们之间不存在优劣性的差异，用户应根据不同的应用选择适合自己的存储引擎。当然，如果你能力很强，完全可以修改存储引擎的源代码，甚至是创建属于自己特定应用的存储引擎，这不就是开源的魅力吗？



### 第2章 InnoDB存储引擎

InnoDB 是事务安全的 MySQL 存储引擎，设计上采用了类似于 Oracle 数据库的架构。通常来说，InnoDB 存储引擎是 OLTP 应用中核心表的首选存储引擎。同时，也正是因为 InnoDB 的存在，才使 MySQL 数据库变得更有魅力。本章将详细介绍 InnoDB 存储引擎的体系架构及其不同于其他存储引擎的特性。



#### 2.1 InnoDB存储引擎概述

InnoDB 存储引擎最早由 Innobase Oy 公司（2006 年该公司已经被 Oracle 公司收购）开发，被包括在 MySQL 数据库所有的二进制发行版本中，从 MySQL 5.5 版本开始是默认的表存储引擎（之前的版本 InnoDB 存储引擎仅在 Windows 下为默认的存储引擎）。该存储引擎是第一个完整支持 ACID 事务的 MySQL 存储引擎（BDB 是第一个支持事务的 MySQL 存储引擎，现在已经停止开发），其特点是行锁设计、支持 MVCC、支持外键、提供一致性非锁定读，同时被设计用来最有效地利用以及使用内存和 CPU。

Heikki Tuuri（1964 年，芬兰赫尔辛基）是 InnoDB 存储引擎的创始人，和著名的 Linux 创始人 Linus 是芬兰赫尔辛基大学校友。在 1990 年获得赫尔辛基大学的数学逻辑博士学位后，他于 1995 年成立 Innobase Oy 公司并担任 CEO。同时，在 InnoDB 存储引擎的开发团队中，有来自中国科技大学的 Calvin Sun。而最近又有一个中国人 Jimmy Yang 也加入了 InnoDB 存储引擎的核心开发团队，负责全文索引的开发，其之前任职于 Sybase 数据库公司，负责数据库的相关开发工作。

InnoDB 存储引擎已经被许多大型网站使用，如用户熟知的 Google、Yahoo！、Facebook、YouTube、Flickr，在网络游戏领域有《魔兽世界》、《SecondLife》、《神兵玄奇》等。我不是 MySQL 数据库的布道者，也不是 InnoDB 的鼓吹者，但是我认为当前实施一个新的 OLTP 项目不使用 MySQL InnoDB 存储引擎将是多么的愚蠢。

从 MySQL 数据库的官方手册可得知，著名的 Internet 新闻站点 Slashdot.org 运行在 InnoDB 上。Mytrix、Inc. 在 InnoDB 上存储超过 1 TB 的数据，还有一些其他站点在 InnoDB 上处理插入/更新操作的速度平均为 800 次/秒。这些都证明了 InnoDB 是一个高性能、高可用、高可扩展的存储引擎。

InnoDB 存储引擎同 MySQL 数据库一样，在 GNU GPL 2 下发行。更多有关 MySQL 证书的信息，可参考 http：//www.mysql.com/about/legal/，这里不再详细介绍。



#### 2.2 InnoDB存储引擎的版本

InnoDB 存储引擎被包含于所有 MySQL 数据库的二进制发行版本中。早期其版本随着 MySQL 数据库的更新而更新。从 MySQL 5.1 版本时，MySQL 数据库允许存储引擎开发商以动态方式加载引擎，这样存储引擎的更新可以不受 MySQL 数据库版本的限制。所以在 MySQL 5.1 中，可以支持两个版本的 InnoDB，一个是静态编译的 InnoDB 版本，可将其视为老版本的 InnoDB；另一个是动态加载的 InnoDB 版本，官方称为 InnoDB Plugin，可将其视为 InnoDB 1.0.x 版本。MySQL 5.5 版本中又将 InnoDB 的版本升级到了 1.1.x。而在最近的 MySQL 5.6 版本中 InnoDB 的版本也随着升级为 1.2.x 版本。表 2-1 显示了各个版本中 InnoDB 存储引擎的功能。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE10.jpg"/>
</div>

在现实工作中我发现很多 MySQL 数据库还是停留在 MySQL 5.1 版本，并使用 InnoDB Plugin。很多 DBA 错误地认为 InnoDB Plugin 和 InnoDB 1.1 版本之间是没有区别的。但从表 2-1 中还是可以发现，虽然都增加了对于 compress 和 dynamic 页的支持，但是 InnoDB Plugin 是不支持 Linux Native AIO 功能的。此外，由于不支持多回滚段，InnoDB Plugin 支持的最大支持并发事务数量也被限制在 1023。而且随着 MySQL 5.5 版本的发布，InnoDB Plugin 也变成了一个历史产品。



#### 2.3 InnoDB体系架构

通过第 1 章读者已经了解了 MySQL 数据库的体系结构，现在可能想更深入地了解 InnoDB 存储引擎的架构。图 2-1 简单显示了 InnoDB 的存储引擎的体系架构，从图可见，InnoDB 存储引擎有多个内存块，可以认为这些内存块组成了一个大的内存池，负责如下工作：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE11.jpg"/>
</div>



- 维护所有进程/线程需要访问的多个内部数据结构。
- 缓存磁盘上的数据，方便快速地读取，同时在对磁盘文件的数据修改之前在这里缓存。
- 重做日志（redo log）缓冲。

   ……



后台线程的主要作用是负责刷新内存池中的数据，保证缓冲池中的内存缓存的是最近的数据。此外将已修改的数据文件刷新到磁盘文件，同时保证在数据库发生异常的情况下 InnoDB 能恢复到正常运行状态。





##### 2.3.1	后台线程

InnoDB 存储引擎是多线程的模型，因此其后台有多个不同的后台线程，负责处理不同的任务。

**1.Master Thread**

Master Thread 是一个非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲（INSERTBUFFER）、UNDO 页的回收等。2.5 节会详细地介绍各个版本中 Master Thread 的工作方式。



**2.IO Thread**

在 InnoDB 存储引擎中大量使用了 AIO（Async IO）来处理写 IO 请求，这样可以极大提高数据库的性能。而 IO Thread 的工作主要是负责这些 IO 请求的回调（callback）处理。InnoDB 1.0 版本之前共有 4 个 IO Thread，分别是 write、read、insert buffer 和 log IO thread。在 Linux 平台下，IO Thread 的数量不能进行调整，但是在 Windows 平台下可以通过参数 innodb_file_io_threads 来增大 IOThread。从 InnoDB 1.0.x 版本开始，read thread 和 write thread 分别增大到了 4 个，并且不再使用 innodb_file_io_threads 参数，而是分别使用 innodb_read_io_threads 和 innodb_write_io_threads 参数进行设置，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE12.jpg"/>
</div>

可以看到 IO Thread 0 为 insert buffer thread。IO Thread 1 为 log thread。之后就是根据参数 innodb_read_io_threads 及 innodb_write_io_threads 来设置的读写线程，并且读线程的 ID 总是小于写线程。



**3.Purge Thread**

事务被提交后，其所使用的 undolog 可能不再需要，因此需要 PurgeThread 来回收已经使用并分配的 undo 页。在 InnoDB 1.1 版本之前，purge 操作仅在 InnoDB 存储引擎的 Master Thread 中完成。而从 InnoDB 1.1 版本开始，purge 操作可以独立到单独的线程中进行，以此来减轻 Master Thread 的工作，从而提高 CPU 的使用率以及提升存储引擎的性能。用户可以在 MySQL 数据库的配置文件中添加如下命令来启用独立的 Purge Thread：

```
[mysqld]
innodb_purge_threads=1
```



在 InnoDB 1.1 版本中，即使将 innodb_purge_threads 设为大于 1，InnoDB 存储引擎启动时也会将其设为 1，并在错误文件中出现如下类似的提示：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE13.jpg"/>
</div>

从 InnoDB 1.2 版本开始，InnoDB 支持多个 Purge Thread，这样做的目的是为了进一步加快 undo 页的回收。同时由于 Purge Thread 需要离散地读取 undo 页，这样也能更进一步利用磁盘的随机读取性能。如用户可以设置 4 个 Purge Thread：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE14.jpg"/>
</div>





**4.Page Cleaner Thread**

Page Cleaner Thread 是在 InnoDB 1.2.x 版本中引入的。其作用是将之前版本中脏页的刷新操作都放入到单独的线程中来完成。而其目的是为了减轻原 MasterThread 的工作及对于用户查询线程的阻塞，进一步提高 InnoDB 存储引擎的性能。





##### 2.3.2 内存

**1.缓冲池**

InnoDB 存储引擎是基于磁盘存储的，并将其中的记录按照页的方式进行管理。因此可将其视为基于磁盘的数据库系统（Disk-base Database）。在数据库系统中，由于 CPU 速度与磁盘速度之间的鸿沟，基于磁盘的数据库系统通常使用缓冲池技术来提高数据库的整体性能。

缓冲池简单来说就是一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，这个过程称为将页 “FIX” 在缓冲池中。下一次再读相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中被命中，直接读取该页。否则，读取磁盘上的页。

对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。这里需要注意的是，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种称为 Checkpoint 的机制刷新回磁盘。同样，这也是为了提高数据库的整体性能。

综上所述，缓冲池的大小直接影响着数据库的整体性能。由于 32 位操作系统的限制，在该系统下最多将该值设置为 3G。此外用户可以打开操作系统的 PAE 选项来获得 32 位操作系统下最大 64GB 内存的支持。随着内存技术的不断成熟，其成本也在不断下降。单条 8GB 的内存变得非常普遍，而 PC 服务器已经能支持 512GB 的内存。因此为了让数据库使用更多的内存，强烈建议数据库服务器都采用 64 位的操作系统。

对于 InnoDB 存储引擎而言，其缓冲池的配置通过参数 innodb_buffer_pool_size 来设置。下面显示一台 MySQL 数据库服务器，其将 InnoDB 存储引擎的缓冲池设置为 15GB。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE15.jpg"/>
</div>

具体来看，缓冲池中缓存的数据页类型有：索引页、数据页、undo 页、插入缓冲（insert buffer）、自适应哈希索引（adaptive hash index）、InnoDB 存储的锁信息（lock info）、数据字典信息（data dictionary）等。不能简单地认为，缓冲池只是缓存索引页和数据页，它们只是占缓冲池很大的一部分而已。图 2-2 很好地显示了 InnoDB 存储引擎中内存的结构情况。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE16.jpg"/>
</div>

从 InnoDB 1.0.x 版本开始，允许有多个缓冲池实例。每个页根据哈希值平均分配到不同缓冲池实例中。这样做的好处是减少数据库内部的资源竞争，增加数据库的并发处理能力。可以通过参数 innodb_buffer_pool_instances 来进行配置，该值默认为 1。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE17.jpg"/>
</div>

在配置文件中将 innodb_buffer_pool_instances 设置为大于 1 的值就可以得到多个缓冲池实例。再通过命令 SHOW ENGINE INNODB STATUS 可以观察到如下的内容：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE18.jpg"/>
</div>

这里将参数 innodb_buffer_pool_instances 设置为 2，即数据库用户拥有两个缓冲池实例。通过命令 SHOW ENGINE INNODB STATUS 可以观察到每个缓冲池实例对象运行的状态，并且通过类似---BUFFER POOL 0 的注释来表明是哪个缓冲池实例。

从 MySQL 5.6 版本开始，还可以通过 information_schema 架构下的表 INNODB_BUFFER_POOL_STATS 来观察缓冲的状态，如运行下列命令可以看到各个缓冲池的使用状态：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE19.jpg"/>
</div>



**2.LRU List、Free List和Flush List**

在前一小节中我们知道了缓冲池是一个很大的内存区域，其中存放各种类型的页。那么 InnoDB 存储引擎是怎么对这么大的内存区域进行管理的呢？这就是本小节要告诉读者的。

通常来说，数据库中的缓冲池是通过 LRU（Latest Recent Used，最近最少使用）算法来进行管理的。即最频繁使用的页在 LRU 列表的前端，而最少使用的页在 LRU 列表的尾端。当缓冲池不能存放新读取到的页时，将首先释放 LRU 列表中尾端的页。

在 InnoDB 存储引擎中，缓冲池中页的大小默认为 16KB，同样使用 LRU 算法对缓冲池进行管理。稍有不同的是 InnoDB 存储引擎对传统的 LRU 算法做了一些优化。在 InnoDB 的存储引擎中，LRU 列表中还加入了 midpoint 位置。新读取到的页，虽然是最新访问的页，但并不是直接放入到 LRU 列表的首部，而是放入到 LRU 列表的 midpoint 位置。这个算法在 InnoDB 存储引擎下称为 midpoint insertion strategy。在默认配置下，该位置在 LRU 列表长度的 5/8 处。midpoint 位置可由参数 innodb_old_blocks_pct 控制，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE20.jpg"/>
</div>

从上面的例子可以看到，参数 innodb_old_blocks_pct 默认值为 37，表示新读取的页插入到 LRU 列表尾端的 37% 的位置（差不多 3/8 的位置）。在 InnoDB 存储引擎中，把 midpoint 之后的列表称为 old 列表，之前的列表称为 new 列表。可以简单地理解为 new 列表中的页都是最为活跃的热点数据。

那为什么不采用朴素的 LRU 算法，直接将读取的页放入到 LRU 列表的首部呢？这是因为若直接将读取到的页放入到 LRU 的首部，那么某些 SQL 操作可能会使缓冲池中的页被刷新出，从而影响缓冲池的效率。常见的这类操作为索引或数据的扫描操作。这类操作需要访问表中的许多页，甚至是全部的页，而这些页通常来说又仅在这次查询操作中需要，并不是活跃的热点数据。如果页被放入 LRU 列表的首部，那么非常可能将所需要的热点数据页从 LRU 列表中移除，而在下一次需要读取该页时，InnoDB 存储引擎需要再次访问磁盘。

为了解决这个问题，InnoDB 存储引擎引入了另一个参数来进一步管理 LRU 列表，这个参数是innodb_old_blocks_time，用于表示页读取到 mid 位置后需要等待多久才会被加入到 LRU 列表的热端。因此当需要执行上述所说的 SQL 操作时，可以通过下面的方法尽可能使 LRU 列表中热点数据不被刷出。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE21.jpg"/>
</div>

如果用户预估自己活跃的热点数据不止 63%，那么在执行 SQL 语句前，还可以通过下面的语句来减少热点页可能被刷出的概率。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE22.jpg"/>
</div>

LRU 列表用来管理已经读取的页，但当数据库刚启动时，LRU 列表是空的，即没有任何的页。这时页都存放在 Free 列表中。当需要从缓冲池中分页时，首先从 Free 列表中查找是否有可用的空闲页，若有则将该页从 Free 列表中删除，放入到 LRU 列表中。否则，根据 LRU 算法，淘汰 LRU 列表末尾的页，将该内存空间分配给新的页。当页从 LRU 列表的 old 部分加入到 new 部分时，称此时发生的操作为 page made young，而因为 innodb_old_blocks_time 的设置而导致页没有从 old 部分移动到 new 部分的操作称为 page not made young。可以通过命令 SHOW ENGINE INNODB STATUS 来观察 LRU 列表及 Free 列表的使用情况和运行状态。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE23.jpg"/>
</div>

通过命令 SHOW ENGINE INNODB STATUS 可以看到：当前 Buffer pool size 共有 327679 个页，即 327679*16K，总共 5GB 的缓冲池。Free buffers 表示当前 Free 列表中页的数量，Database pages 表示 LRU 列表中页的数量。可能的情况是 Free buffers 与 Database pages 的数量之和不等于 Buffer pool size。正如图 2-2 所示的那样，因为缓冲池中的页还可能会被分配给自适应哈希索引、Lock 信息、Insert Buffer 等页，而这部分页不需要 LRU 算法进行维护，因此不存在于 LRU 列表中。

pages made young 显示了 LRU 列表中页移动到前端的次数，因为该服务器在运行阶段没有改变 innodb_old_blocks_time 的值，因此 not young 为 0。youngs/s、non-youngs/s 表示每秒这两类操作的次数。这里还有一个重要的观察变量——Buffer pool hit rate，表示缓冲池的命中率，这个例子中为 100%，说明缓冲池运行状态非常良好。通常该值不应该小于 95%。若发生 Buffer pool hit rate 的值小于 95% 这种情况，用户需要观察是否是由于全表扫描引起的 LRU 列表被污染的问题。

**注意** 执行命令 SHOW ENGINE INNODB STATUS 显示的不是当前的状态，而是过去某个时间范围内 InnoDB 存储引擎的状态。从上面的例子可以发现，Per second averages calculated from the last 24 seconds 代表的信息为过去 24 秒内的数据库状态。

从 InnoDB 1.2 版本开始，还可以通过表 INNODB_BUFFER_POOL_STATS 来观察缓冲池的运行状态，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE24.jpg"/>
</div>

此外，还可以通过表 INNODB_BUFFER_PAGE_LRU 来观察每个 LRU 列表中每个页的具体信息，例如通过下面的语句可以看到缓冲池 LRU 列表中 SPACE 为 1 的表的页类型：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE25.jpg"/>
</div>

InnoDB 存储引擎从 1.0.x 版本开始支持压缩页的功能，即将原本 16KB 的页压缩为 1KB、2KB、4KB 和 8KB。而由于页的大小发生了变化，LRU 列表也有了些许的改变。对于非 16KB 的页，是通过 unzip_LRU 列表进行管理的。通过命令 SHOW ENGINE INNODB STATUS 可以观察到如下内容：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE26.jpg"/>
</div>

可以看到 LRU 列表中一共有 1539 个页，而 unzip_LRU 列表中有 156 个页。这里需要注意的是，LRU 中的页包含了 unzip_LRU 列表中的页。

对于压缩页的表，每个表的压缩比率可能各不相同。可能存在有的表页大小为 8KB，有的表页大小为 2KB 的情况。unzip_LRU 是怎样从缓冲池中分配内存的呢？

首先，在 unzip_LRU 列表中对不同压缩页大小的页进行分别管理。其次，通过伙伴算法进行内存的分配。例如对需要从缓冲池中申请页为 4KB 的大小，其过程如下：

1）检查 4KB 的 unzip_LRU 列表，检查是否有可用的空闲页；

2）若有，则直接使用；

3）否则，检查 8KB 的 unzip_LRU 列表；

4）若能够得到空闲页，将页分成 2 个 4KB 页，存放到 4KB 的 unzip_LRU 列表；

5）若不能得到空闲页，从 LRU 列表中申请一个 16KB 的页，将页分为 1 个 8KB 的页、2 个 4KB 的页，分别存放到对应的 unzip_LRU 列表中。同样可以通过 information_schema 架构下的表 INNODB_BUFFER_PAGE_LRU 来观察 unzip_LRU 列表中的页，如：

同样可以通过 information_schema 架构下的表 INNODB_BUFFER_PAGE_LRU 来观察 unzip_LRU 列表中的页，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE27.jpg"/>
</div>

在 LRU 列表中的页被修改后，称该页为脏页（dirty page），即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过 CHECKPOINT 机制将脏页刷新回磁盘，而 Flush 列表中的页即为脏页列表。需要注意的是，脏页既存在于 LRU 列表中，也存在于 Flush 列表中。LRU 列表用来管理缓冲池中页的可用性，Flush 列表用来管理将页刷新回磁盘，二者互不影响。

同 LRU 列表一样，Flush 列表也可以通过命令 SHOW ENGINE INNODB STATUS 来查看，前面例子中 Modified db pages 24673 就显示了脏页的数量。information_schema 架构下并没有类似 INNODB_BUFFER_PAGE_LRU 的表来显示脏页的数量及脏页的类型，但正如前面所述的那样，脏页同样存在于 LRU 列表中，故用户可以通过元数据表 INNODB_BUFFER_PAGE_LRU 来查看，唯一不同的是需要加入 OLDEST_MODIFICATION 大于 0 的 SQL 查询条件，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE28.jpg"/>
</div>

可以看到当前共有 5 个脏页及它们对应的表和页的类型。TABLE_NAME 为 NULL 表示该页属于系统表空间。



**3.重做日志缓冲**

从图 2-2 可以看到，InnoDB 存储引擎的内存区域除了有缓冲池外，还有重做日志缓冲（redo log buffer）。InnoDB 存储引擎首先将重做日志信息先放入到这个缓冲区，然后按一定频率将其刷新到重做日志文件。重做日志缓冲一般不需要设置得很大，因为一般情况下每一秒钟会将重做日志缓冲刷新到日志文件，因此用户只需要保证每秒产生的事务量在这个缓冲大小之内即可。该值可由配置参数 innodb_log_buffer_size 控制，默认为 8MB：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE29.jpg"/>
</div>

在通常情况下，8MB 的重做日志缓冲池足以满足绝大部分的应用，因为重做日志在下列三种情况下会将重做日志缓冲中的内容刷新到外部磁盘的重做日志文件中。

- Master Thread 每一秒将重做日志缓冲刷新到重做日志文件；
- 每个事务提交时会将重做日志缓冲刷新到重做日志文件； 
- 当重做日志缓冲池剩余空间小于 1/2 时，重做日志缓冲刷新到重做日志文件



**4.额外的内存池**

额外的内存池通常被 DBA 忽略，他们认为该值并不十分重要，事实恰恰相反，该值同样十分重要。在 InnoDB 存储引擎中，对内存的管理是通过一种称为内存堆（heap）的方式进行的。在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中进行申请。例如，分配了缓冲池（innodb_buffer_pool），但是每个缓冲池中的帧缓冲（framebuffer）还有对应的缓冲控制对象（buffer control block），这些对象记录了一些诸如 LRU、锁、等待等信息，而这个对象的内存需要从额外内存池中申请。因此，在申请了很大的 InnoDB 缓冲池时，也应考虑相应地增加这个值。



#### 2.4 Checkpoint技术

前面已经讲到了，缓冲池的设计目的为了协调 CPU 速度与磁盘速度的鸿沟。因此页的操作首先都是在缓冲池中完成的。如果一条 DML 语句，如 Update 或 Delete 改变了页中的记录，那么此时页是脏的，即缓冲池中的页的版本要比磁盘的新。数据库需要将新版本的页从缓冲池刷新到磁盘。

倘若每次一个页发生变化，就将新页的版本刷新到磁盘，那么这个开销是非常大的。若热点数据集中在某几个页中，那么数据库的性能将变得非常差。同时，如果在从缓冲池将页的新版本刷新到磁盘时发生了宕机，那么数据就不能恢复了。为了避免发生数据丢失的问题，当前事务数据库系统普遍都采用了 Write Ahead Log 策略，即当事务提交时，先写重做日志，再修改页。当由于发生宕机而导致数据丢失时，通过重做日志来完成数据的恢复。这也是事务 ACID 中 D（Durability 持久性）的要求。

思考下面的场景，如果重做日志可以无限地增大，同时缓冲池也足够大，能够缓冲所有数据库的数据，那么是不需要将缓冲池中页的新版本刷新回磁盘。因为当发生宕机时，完全可以通过重做日志来恢复整个数据库系统中的数据到宕机发生的时刻。但是这需要两个前提条件：

- 缓冲池可以缓存数据库中所有的数据；
- 重做日志可以无限增大。

对于第一个前提条件，有经验的用户都知道，当数据库刚开始创建时，表中没有任何数据。缓冲池的确可以缓存所有的数据库文件。然而随着市场的推广，用户的增加，产品越来越受到关注，使用量也越来越大。这时负责后台存储的数据库的容量必定会不断增大。当前 3TB 的 MySQL 数据库已并不少见，但是 3 TB 的内存却非常少见。目前 Oracle Exadata 旗舰数据库一体机也就只有 2 TB 的内存。因此第一个假设对于生产环境应用中的数据库是很难得到保证的。

再来看第二个前提条件：重做日志可以无限增大。也许是可以的，但是这对成本的要求太高，同时不便于运维。DBA 或 SA 不能知道什么时候重做日志是否已经接近于磁盘可使用空间的阈值，并且要让存储设备支持可动态扩展也是需要一定的技巧和设备支持的。

好的，即使上述两个条件都满足，那么还有一个情况需要考虑：宕机后数据库的恢复时间。当数据库运行了几个月甚至几年时，这时发生宕机，重新应用重做日志的时间会非常久，此时恢复的代价也会非常大。

因此 Checkpoint（检查点）技术的目的是解决以下几个问题：

- 缩短数据库的恢复时间；
- 缓冲池不够用时，将脏页刷新到磁盘；
- 重做日志不可用时，刷新脏页。

当数据库发生宕机时，数据库不需要重做所有的日志，因为 Checkpoint 之前的页都已经刷新回磁盘。故数据库只需对 Checkpoint 后的重做日志进行恢复。这样就大大缩短了恢复的时间。

此外，当缓冲池不够用时，根据 LRU 算法会溢出最近最少使用的页，若此页为脏页，那么需要强制执行 Checkpoint，将脏页也就是页的新版本刷回磁盘。

重做日志出现不可用的情况是因为当前事务数据库系统对重做日志的设计都是循环使用的，并不是让其无限增大的，这从成本及管理上都是比较困难的。重做日志可以被重用的部分是指这些重做日志已经不再需要，即当数据库发生宕机时，数据库恢复操作不需要这部分的重做日志，因此这部分就可以被覆盖重用。若此时重做日志还需要使用，那么必须强制产生 Checkpoint，将缓冲池中的页至少刷新到当前重做日志的位置。

对于 InnoDB 存储引擎而言，其是通过 LSN（Log Sequence Number）来标记版本的。而 LSN 是 8 字节的数字，其单位是字节。每个页有 LSN，重做日志中也有 LSN，Checkpoint 也有 LSN。可以通过命令 SHOW ENGINE INNODB STATUS 来观察：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE30.jpg"/>
</div>

在 InnoDB 存储引擎中，Checkpoint 发生的时间、条件及脏页的选择等都非常复杂。而 Checkpoint 所做的事情无外乎是将缓冲池中的脏页刷回到磁盘。不同之处在于每次刷新多少页到磁盘，每次从哪里取脏页，以及什么时间触发 Checkpoint。在 InnoDB 存储引擎内部，有两种 Checkpoint，分别为：

- Sharp Checkpoint
- Fuzzy Checkpoint

Sharp Checkpoint 发生在数据库关闭时将所有的脏页都刷新回磁盘，这是默认的工作方式，即参数 innodb_fast_shutdown=1。

但是若数据库在运行时也使用 Sharp Checkpoint，那么数据库的可用性就会受到很大的影响。故在 InnoDB 存储引擎内部使用 Fuzzy Checkpoint 进行页的刷新，即只刷新一部分脏页，而不是刷新所有的脏页回磁盘。

这里笔者进行了概括，在 InnoDB 存储引擎中可能发生如下几种情况的 FuzzyCheckpoint：

- Master Thread Checkpoint
- FLUSH_LRU_LIST Checkpoint
- Async/Sync Flush Checkpoint
- Dirty Page too much Checkpoint

对于 Master Thread（2.5 节会详细介绍各个版本中 Master Thread 的实现）中发生的 Checkpoint，差不多以每秒或每十秒的速度从缓冲池的脏页列表中刷新一定比例的页回磁盘。这个过程是异步的，即此时 InnoDB 存储引擎可以进行其他的操作，用户查询线程不会阻塞。

FLUSH_LRU_LIST Checkpoint 是因为 InnoDB 存储引擎需要保证 LRU 列表中需要有差不多 100 个空闲页可供使用。在 InnoDB1.1.x 版本之前，需要检查 LRU 列表中是否有足够的可用空间操作发生在用户查询线程中，显然这会阻塞用户的查询操作。倘若没有 100 个可用空闲页，那么 InnoDB 存储引擎会将 LRU 列表尾端的页移除。如果这些页中有脏页，那么需要进行 Checkpoint，而这些页是来自 LRU 列表的，因此称为 FLUSH_LRU_LIST Checkpoint。

而从 MySQL 5.6 版本，也就是 InnoDB1.2.x 版本开始，这个检查被放在了一个单独的 Page Cleaner 线程中进行，并且用户可以通过参数 innodb_lru_scan_depth 控制 LRU 列表中可用页的数量，该值默认为 1024，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE31.jpg"/>
</div>

Async/Sync Flush Checkpoint 指的是重做日志文件不可用的情况，这时需要强制将一些页刷新回磁盘，而此时脏页是从脏页列表中选取的。若将已经写入到重做日志的 LSN 记为 redo_lsn，将已经刷新回磁盘最新页的 LSN 记为 checkpoint_lsn，则可定义：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE32.jpg"/>
</div>

若每个重做日志文件的大小为 1GB，并且定义了两个重做日志文件，则重做日志文件的总大小为 2GB。那么 async_water_mark=1.5GB，sync_water_mark=1.8GB。则：

- 当 checkpoint_age<async_water_mark 时，不需要刷新任何脏页到磁盘；
- 当 async_water_mark<checkpoint_age<sync_water_mark 时触发 AsyncFlush，从 Flush 列表中刷新足够的脏页回磁盘，使得刷新后满足 checkpoint_age<async_water_mark；
-  checkpoint_age>sync_water_mark 这种情况一般很少发生，除非设置的重做日志文件太小，并且在进行类似 LOAD DATA 的 BULK INSERT 操作。此时触发 Sync Flush 操作，从 Flush 列表中刷新足够的脏页回磁盘，使得刷新后满足 checkpoint_age<async_water_mark。

可见，Async/Sync Flush Checkpoint 是为了保证重做日志的循环使用的可用性。在 InnoDB 1.2.x 版本之前，Async Flush Checkpoint 会阻塞发现问题的用户查询线程，而 Sync Flush Checkpoint 会阻塞所有的用户查询线程，并且等待脏页刷新完成。从 InnoDB 1.2.x 版本开始——也就是 MySQL 5.6 版本，这部分的刷新操作同样放入到了单独的 Page Cleaner Thread 中，故不会阻塞用户查询线程。

MySQL 官方版本并不能查看刷新页是从 Flush 列表中还是从 LRU 列表中进行 Checkpoint 的，也不知道因为重做日志而产生的 Async/Sync Flush 的次数。但是 InnoSQL 版本提供了方法，可以通过命令 SHOW ENGINE INNODB STATUS 来观察，如：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE33.jpg"/>
</div>

根据上述的信息，还可以对 InnoDB 存储引擎做更为深入的调优，这部分将在第 9 章中讲述。

最后一种 Checkpoint 的情况是 Dirty Page too much，即脏页的数量太多，导致 InnoDB 存储引擎强制进行 Checkpoint。其目的总的来说还是为了保证缓冲池中有足够可用的页。其可由参数 innodb_max_dirty_pages_pct 控制：

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/InnoDB%20-%20%E5%9B%BE34.jpg"/>
</div>

innodb_max_dirty_pages_pct 值为 75 表示，当缓冲池中脏页的数量占据 75% 时，强制进行 Checkpoint，刷新一部分的脏页到磁盘。在 InnoDB 1.0.x 版本之前，该参数默认值为 90，之后的版本都为 75。



#### 2.5 Master Thread工作方式

在 2.3 节中我们知道了，InnoDB 存储引擎的主要工作都是在一个单独的后台线程 Master Thread 中完成的，这一节将具体解释该线程的具体实现及该线程可能存在的问题。



##### 2.5.1 InnoDB 1.0.x版本之前的Master Thread

Master Thread 具有最高的线程优先级别。其内部由多个循环（loop）组成：主循环（loop）、后台循环（backgroup loop）、刷新循环（flush loop）、暂停循环（suspend loop）。Master Thread 会根据数据库运行的状态在 loop、background loop、flush loop 和 suspend loop 中进行切换。

Loop 被称为主循环，因为大多数的操作是在这个循环中，其中有两大部分的操作——每秒钟的操作和每 10 秒的操作。伪代码如下：

```c
void master_thread(){
    loop;
    for(int i = 0; i<10; i++) {
        do thing once per second
        sleep 1 second if necessary
    }
    do things onec per ten seconds
    goto loop;
}
```



可以看到，loop 循环通过 thread sleep 来实现，这意味着所谓的每秒一次或每 10 秒一次的操作是不精确的。在负载很大的情况下可能会有延迟（delay），只能说大概在这个频率下。当然，InnoDB 源代码中还通过了其他的方法来尽量保证这个频率。

每秒一次的操作包括：

- 日志缓冲刷新到磁盘，即使这个事务还没有提交（总是）；
- 合并插入缓冲（可能）；
- 至多刷新 100 个 InnoDB 的缓冲池中的脏页到磁盘（可能）；
- 如果当前没有用户活动，则切换到 background loop（可能）。

即使某个事务还没有提交，InnoDB 存储引擎仍然每秒会将重做日志缓冲中的内容刷新到重做日志文件。这一点是必须要知道的，因为这可以很好地解释为什么再大的事务提交（commit）的时间也是很短的。

合并插入缓冲（Insert Buffer）并不是每秒都会发生的。InnoDB 存储引擎会判断当前一秒内发生的 IO 次数是否小于 5 次，如果小于 5 次，InnoDB 认为当前的 IO 压力很小，可以执行合并插入缓冲的操作。

同样，刷新 100 个脏页也不是每秒都会发生的。InnoDB 存储引擎通过判断当前缓冲池中脏页的比例（buf_get_modified_ratio_pct）是否超过了配置文件中 innodb_max_dirty_pages_pct 这个参数（默认为 90，代表 90%），如果超过了这个阈值，InnoDB 存储引擎认为需要做磁盘同步的操作，将 100 个脏页写入磁盘中。

总结上述操作，伪代码可以进一步具体化，如下所示：

略。。。



#### 第3章 文件

本章将分析构成MySQL数据库和InnoDB存储引擎表的各种类型文件。这些文件有以下这些。

- 参数文件：告诉MySQL实例启动时在哪里可以找到数据库文件，并且指定某些初始化参数，这些参数定义了某种内存结构的大小等设置，还会介绍各种参数的类型。
- 日志文件：用来记录MySQL实例对某种条件做出响应时写入的文件，如错误日志文件、二进制日志文件、慢查询日志文件、查询日志文件等。
- socket文件：当用UNIX域套接字方式进行连接时需要的文件。
- pid文件：MySQL实例的进程ID文件。
- MySQL表结构文件：用来存放MySQL表结构定义文件。
- 存储引擎文件：因为MySQL表存储引擎的关系，每个存储引擎都会有自己的文件来保存各种数据。这些存储引擎真正存储了记录和索引等数据。本章主要介绍与InnoDB有关的存储引擎文件。



### 3.1 参数文件

在第1章中已经介绍过了，当MySQL实例启动时，数据库会先去读一个配置参数文件，用来寻找数据库的各种文件所在位置以及指定某些初始化参数，这些参数通常定义了某种内存结构有多大等。在默认情况下，MySQL实例会按照一定的顺序在指定的位置进行读取，用户只需通过命令mysql--help | grep my.cnf来寻找即可。

MySQL数据库参数文件的作用和Oracle数据库的参数文件极其类似，不同的是，Oracle实例在启动时若找不到参数文件，是不能进行装载（mount）操作的。MySQL稍微有所不同，MySQL实例可以不需要参数文件，这时所有的参数值取决于编译MySQL时指定的默认值和源代码中指定参数的默认值。但是，如果MySQL实例在默认的数据库目录下找不到mysql架构，则启动同样会失败，此时可能在错误日志文件中找到如下内容：

略



MySQL的mysql架构中记录了访问该实例的权限，当找不到这个架构时，MySQL实例不会成功启动。MySQL数据库的参数文件是以文本方式进行存储的。用户可以直接通过一些常用的文本编辑软件（如vi和emacs）进行参数的修改。



##### 3.1.1 什么是参数

简单地说，可以把数据库参数看成一个键/值（key/value）对。第2章已经介绍了一个对于InnoDB存储引擎很重要的参数innodb_buffer_pool_size。如我们将这个参数设置为1G，即innodb_buffer_pool_size=1G。这里的“键”是innodb_buffer_pool_size，“值”是1G，这就是键值对。可以通过命令SHOW VARIABLES查看数据库中的所有参数，也可以通过LIKE来过滤参数名。从MySQL5.1版本开始，还可以通过information_schema架构下的GLOBAL_VARIABLES视图来进行查找，如下所示。

略



无论使用哪种方法，输出的信息基本上都一样的，只不过通过视图GLOBAL_VARIABLES需要指定视图的列名。推荐使用命令SHOW VARIABLES，因为这个命令使用更为简单，且各版本的MySQL数据库都支持。

Oracle数据库存在所谓的隐藏参数（undocumented parameter），以供Oracle“内部人士”使用，SQL Server也有类似的参数。有些DBA曾问我，MySQL中是否也有这类参数。我的回答是：没有，也不需要。即使Oracle和SQL Server中都有些所谓的隐藏参数，在绝大多数的情况下，这些数据库厂商也不建议用户在生产环境中对其进行很大的调整。



##### 3.1.2 参数类型

MySQL数据库中的参数可以分为两类：

- 动态（dynamic）参数
- 静态（static）参数

动态参数意味着可以在MySQL实例运行中进行更改，静态参数说明在整个实例生命周期内都不得进行更改，就好像是只读（read only）的。可以通过SET命令对动态的参数值进行修改，SET的语法如下：

略



这里可以看到global和session关键字，它们表明该参数的修改是基于当前会话还是整个实例的生命周期。有些动态参数只能在会话中进行修改，如autocommit；而有些参数修改完后，在整个实例生命周期中都会生效，如binlog_cache_size；而有些参数既可以在会话中又可以在整个实例的生命周期内生效，如read_buffer_size。举例如下：

略



上述示例中将当前会话的参数read_buffer_size从2MB调整为了512KB，而用户可以看到全局的read_buffer_size的值仍然是2MB，也就是说如果有另一个会话登录到MySQL实例，它的read_buffer_size的值是2MB，而不是512KB。这里使用了setglobal|session来改变动态变量的值。用户同样可以直接使用SET@@globl|@@session来更改，如下所示：

略



这次把read_buffer_size全局值更改为1MB，而当前会话的read_buffer_size的值还是512KB。这里需要注意的是，对变量的全局值进行了修改，在这次的实例生命周期内都有效，但MySQL实例本身并不会对参数文件中的该值进行修改。也就是说，在下次启动时MySQL实例还是会读取参数文件。若想在数据库实例下一次启动时该参数还是保留为当前修改的值，那么用户必须去修改参数文件。要想知道MySQL所有动态变量的可修改范围，可以参考MySQL官方手册的Dynamic SystemVariables的相关内容。

对于静态变量，若对其进行修改，会得到类似如下错误：

略



#### 3.2 日志文件

日志文件记录了影响MySQL数据库的各种类型活动。MySQL数据库中常见的日志文件有：

- 错误日志（error log）
- 二进制日志（binlog）
- 慢查询日志（slow query log）
- 查询日志（log）

这些日志文件可以帮助DBA对MySQL数据库的运行状态进行诊断，从而更好地进行数据库层面的优化。



##### 3.2.1 错误日志

错误日志文件对MySQL的启动、运行、关闭过程进行了记录。MySQL DBA在遇到问题时应该首先查看该文件以便定位问题。该文件不仅记录了所有的错误信息，也记录一些警告信息或正确的信息。用户可以通过命令SHOW VARIABLES LIKE'log_error'来定位该文件，如：

略



可以看到错误文件的路径和文件名，在默认情况下错误文件的文件名为服务器的主机名。如上面看到的，该主机名为stargazer，所以错误文件名为startgazer.err。当出现MySQL数据库不能正常启动时，第一个必须查找的文件应该就是错误日志文件，该文件记录了错误信息，能很好地指导用户发现问题。当数据库不能重启时，通过查错误日志文件可以得到如下内容：

略



这里，错误日志文件提示了找不到权限库mysql，所以启动失败。有时用户可以直接在错误日志文件中得到优化的帮助，因为有些警告（warning）很好地说明了问题所在。而这时可以不需要通过查看数据库状态来得知，例如，下面的错误文件中的信息可能告诉用户需要增大InnoDB存储引擎的redo log：

略



##### 3.2.2 慢查询日志

3.2.1小节提到可以通过错误日志得到一些关于数据库优化的信息，而慢查询日志（slow log）可帮助DBA定位可能存在问题的SQL语句，从而进行SQL语句层面的优化。例如，可以在MySQL启动时设一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询日志文件中。DBA每天或每过一段时间对其进行检查，确认是否有SQL语句需要进行优化。该阈值可以通过参数long_query_time来设置，默认值为10，代表10秒。

在默认情况下，MySQL数据库并不启动慢查询日志，用户需要手工将这个参数设为ON：

略



这里有两点需要注意。首先，设置long_query_time这个阈值后，MySQL数据库会记录运行时间超过该值的所有SQL语句，但运行时间正好等于long_query_time的情况并不会被记录下。也就是说，在源代码中判断的是大于long_query_time，而非大于等于。其次，从MySQL 5.1开始，long_query_time开始以微秒记录SQL语句运行的时间，之前仅用秒为单位记录。而这样可以更精确地记录SQL的运行时间，供DBA分析。对DBA来说，一条SQL语句运行0.5秒和0.05秒是非常不同的，前者可能已经进行了表扫，后面可能是进行了索引。

另一个和慢查询日志有关的参数是log_queries_not_using_indexes，如果运行的SQL语句没有使用索引，则MySQL数据库同样会将这条SQL语句记录到慢查询日志文件。首先确认打开了log_queries_not_using_indexes：

略



MySQL 5.6.5版本开始新增了一个参数log_throttle_queries_not_using_indexes，用来表示每分钟允许记录到slow log的且未使用索引的SQL语句次数。该值默认为0，表示没有限制。在生产环境下，若没有使用索引，此类SQL语句会频繁地被记录到slow log，从而导致slow log文件的大小不断增加，故DBA可通过此参数进行配置。

DBA可以通过慢查询日志来找出有问题的SQL语句，对其进行优化。然而随着MySQL数据库服务器运行时间的增加，可能会有越来越多的SQL查询被记录到了慢查询日志文件中，此时要分析该文件就显得不是那么简单和直观的了。而这时MySQL数据库提供的mysqldumpslow命令，可以很好地帮助DBA解决该问题：

略



如果用户希望得到执行时间最长的10条SQL语句，可以运行如下命令：

略



MySQL 5.1开始可以将慢查询的日志记录放入一张表中，这使得用户的查询更加方便和直观。慢查询表在mysql架构下，名为slow_log，其表结构定义如下：

略



参数log_output指定了慢查询输出的格式，默认为FILE，可以将它设为TABLE，然后就可以查询mysql架构下的slow_log表了，如：

略



参数log_output是动态的，并且是全局的，因此用户可以在线进行修改。在上表中人为设置了睡眠（sleep）10秒，那么这句SQL语句就会被记录到slow_log表了。

查看slow_log表的定义会发现该表使用的是CSV引擎，对大数据量下的查询效率可能不高。用户可以把slow_log表的引擎转换到MyISAM，并在start_time列上添加索引以进一步提高查询的效率。但是，如果已经启动了慢查询，将会提示错误：

略



不能忽视的是，将slow_log表的存储引擎更改为MyISAM后，还是会对数据库造成额外的开销。不过好在很多关于慢查询的参数都是动态的，用户可以方便地在线进行设置或修改。

MySQL的slow log通过运行时间来对SQL语句进行捕获，这是一个非常有用的优化技巧。但是当数据库的容量较小时，可能因为数据库刚建立，此时非常大的可能是数据全部被缓存在缓冲池中，SQL语句运行的时间可能都是非常短的，一般都是0.5秒。

InnoSQL版本加强了对于SQL语句的捕获方式。在原版MySQL的基础上在slow log中增加了对于逻辑读取（logical reads）和物理读取（physical reads）的统计。这里的物理读取是指从磁盘进行IO读取的次数，逻辑读取包含所有的读取，不管是磁盘还是缓冲池。例如：

略



从上面的例子可以看到该子查询的逻辑读取次数是91584次，物理读取为19次。从逻辑读与物理读的比例上看，该SQL语句可进行优化。

用户可以通过额外的参数long_query_io将超过指定逻辑IO次数的SQL语句记录到slow log中。该值默认为100，即表示对于逻辑读取次数大于100的SQL语句，记录到slow log中。而为了兼容原MySQL数据库的运行方式，还添加了参数slow_query_type，用来表示启用slow log的方式，可选值为：

- 0表示不将SQL语句记录到slow log
- 1表示根据运行时间将SQL语句记录到slow log
- 2表示根据逻辑IO次数将SQL语句记录到slow log
- 3表示根据运行时间及逻辑IO次数将SQL语句记录到slow log



##### 3.2.3 查询日志

查询日志记录了所有对MySQL数据库请求的信息，无论这些请求是否得到了正确的执行。默认文件名为：主机名.log。如查看一个查询日志：

略



通过上述查询日志会发现，查询日志甚至记录了对Access denied的请求，即对于未能正确执行的SQL语句，查询日志也会进行记录。同样地，从MySQL 5.1开始，可以将查询日志的记录放入mysql架构下的general_log表中，该表的使用方法和前面小节提到的slow_log基本一样，这里不再赘述。



##### 3.2.4 二进制日志

二进制日志（binary log）记录了对MySQL数据库执行更改的所有操作，但是不包括SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改。然而，若操作本身并没有导致数据库发生变化，那么该操作可能也会写入二进制日志。例如：

略



从上述例子中可以看到，MySQL数据库首先进行UPDATE操作，从返回的结果看到Changed为0，这意味着该操作并没有导致数据库的变化。但是通过命令SHOWBINLOG EVENT可以看出在二进制日志中的确进行了记录。

如果用户想记录SELECT和SHOW操作，那只能使用查询日志，而不是二进制日志。此外，二进制日志还包括了执行数据库更改操作的时间等其他额外信息。总的来说，二进制日志主要有以下几种作用。















