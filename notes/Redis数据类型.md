## Redis data types

Redis并不仅仅是一个键值存储器，实际上它是一个数据结构服务器，支持不同种类的键值。与传统的键值存储器最大的不同，就是它不仅仅能用 String 类型的键存储 String 类型的值，更能处理复杂得多的数据结构。以下就是它所支持的数据结构列表。

- Binary-safe strings：二进制安全的字符串
- Lists：按插入顺序排序的字符串元素的集合。实际上就是链表。
- Sets：不重复且无序的字符串元素的集合。
- Sorted sets，与 Sets 类似，但每个字符串元素都关联到一个叫 score 的浮点型值。里面的元素总是通过 score 进行排序，所以与 set 不同的是，它是可以检索的一系列元素。（例如你可能会问：给我前面 10 个或者后面 10 个元素）。
- Hashes，由 field 和关联的 value 组成的 map。field 和 value 都是字符串的。这和 Ruby、Python 的 hashes 很像。
- Bit arrays（或者说 simply bitmaps）：通过特殊的命令，你可以将 String 值当作一系列 bits 处理：可以设置和清除单独的 bits，输出所有设为 1 的 bits 的数量，找到最前的被设为 1 或 0 的 bit，等等。
- HyperLogLogs：这是被用于估计一个 set 中元素数量的概率性的数据结构。



#### Redis keys

Redis key值是二进制安全的，这意味着可以用任何二进制序列作为 key值，从形如 "foo" 的简单字符串到一个 JPEG 文件的内容都可以。空字符串也是有效 key值。

关于 key 的几条规则：

- 太长的键值不是个好主意，例如1024字节的键值就不是个好主意，不仅因为消耗内存，而且在数据中查找这类键值的计算成本很高。
- 太短的键值通常也不是好主意，如果你要用 “u:1000:pwd” 来代替 “user:1000:password”，这没有什么问题，但后者更易阅读，并且由此增加的空间消耗相对于 key object 和 value object 本身来说很小。
- 最好坚持一种模式。例如： "object-type:Id:field" 就是个不错的主意，像这样”user:1000:password”。我喜欢对多单词的字段名中加上一个点，就像这样：”commen​t:1234：​reply.to”。
- 键最大容量是 512 MB。



#### Redis Strings

这是最简单的 Redis 类型。如果你只用这种类型，Redis就像一个可以持久化的 memcached 服务器（注：memcached 的数据仅仅保存在内存中，服务器重启后，数据将丢失）。

由于Redis键是字符串，当我们使用字符串类型作为值时，我们将字符串映射到另一个字符串。字符串数据类型对于许多用例很有用，例如缓存 HTML 片段或页面。

```
> set mykey somevalue
> get mykey
"somevalue"
```

我们通常用 SET command 和 GET command 来设置和获取字符串值。请注意，即使 key值 与非字符串值相关联，SET 也将替换已存在于 key值中的任何现有值（如果 key值已存在）。

值可以是任何种类的字符串（包括二进制数据），例如你可以在一个键下保存一副 jpeg 图片。值的长度不能超过 512 MB。

SET命令有一些有趣的选项，可以通过附加参数设置。例如，如果密钥已经存在，我可以设置成 SET 操作一定失败，或者相反，可以设置成  SET 操作一定成功。

虽然字符串是 Redis 的基本值类型，但你仍然能通过它完成一些有趣的操作。例如：源自递增：

```
> set counter 100
ok
> incr counter
(integer) 101
> incr counter
(integer) 102
> incrby counter 50
(integer) 152
```

INCR 命令将字符串值解析成整型，将其加一，最后将结果保存为新的字符串值，类似的命令有

INCRBY，DECR 和 DECRBY。实际上他们在内部就是同一个命令，只是看上去有点儿不同。

INCR 是原子操作意味着什么呢？就是说即使多个客户端对同一个 key 发出 INCR 命令，也绝不会导致竞争的情况。

对于字符串，另一个令人感兴趣的操作是 GETSET 命令，令如其名：他为 key 设置新值并且返回原值。这有什么用处呢？例如：你的系统每当有新用户访问时就用 INCR 命令操作一个 Redis key。你希望每小时对这个信息收集一次。你就可以 GETSET 这个 key 并给其赋值 0 并读取原值。

为减少等待时间，也可以一次存储或获取多个 key 对应的值，使用 MSET 和 MGET 命令：

```
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

MGET 命令返回由值组成的数组。



#### 修改或查询键空间

有些指令不是针对任何具体的类型定义的，而是用于和整个键空间交互的。因此，它们可被用于任何类型的键。

使用 EXISTS 命令返回 1 或 0 标识判断给定 key 的值是否存在，使用 DEL 命令可以删除 key 对应的值， DEL 命令返回 1 或 0 标识值是被删除（值存在）或者没被删除（ key 对应的值不存在）。

```
> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
> del mykey
(integer) 0
```

TYPE 命令可以返回 key 对应的值的存储类型：

```
> set mykey x
OK
> type mykey
string
> del mykey
(integer) 1
> type mykey
none
```



#### Redis 超时：数据在限定时间内存活

在介绍复杂类型前我们先介绍一个与值类型无关的 Redis 特性：超时。你可以对 key 设置一个超时时间，当这个时间到达后会被删除。

- 精度可以使用毫秒或秒，默认是秒。
- 但是，过期时间精度始终为1毫秒。
- 有关过期的信息被复制并保留在磁盘上（过期时间本身会被持久化），当 Redis 服务器停止服务时，这个时间照样计算（这意味着 Redis 会保存密钥过期的日期）

```
> set key some-value
OK
> expire key 10
(integer) 1
> get key (immediately)
"some-value"
> get key (after some time)
(nil)e
```

上面的例子使用了EXPIRE来设置超时时间(也可以再次调用这个命令来改变超时时间，或使用 PERSIST 命令去除超时时间 )。我们也可以使用 SET 命令在创建值的时候设置超时时间:

```
> set key 100 ex 10
OK
> ttl key
(integer) 9
```

TTL  time to live，以秒为单位，返回给定 key 的剩余生存时间。



#### Redis lists

要说清楚列表数据类型，最好先讲一点儿理论背景，在信息技术界 List 这个词常常被使用不当。例如 “Python Lists” 名不副实（名为 Linked Lists），但他们实际上师叔祖（同样的数据类型在 Ruby 中叫做数组）。

一般意义上讲，列表就是有序元素的序列： 10，20，1，2，3 就是一个列表。但用数组实现的 List 和用 LinkedList 实现的 List，在属性方面大不相同。

Redis lists 基于 Linked Lists 实现。这意味着即使在一个 list 中有数百万个元素，在头部或尾部添加一个元素的操作，其时间复杂度也是常数级别的。用 LPUSH 命令在十个元素的 list 头部添加新元素，和在千万元素 list 头部添加新元素的速度相同。

那么，坏消息是什么？在数组实现的 list 中利用索引访问元素的速度极快，而同样的操作在 linked list 实现的 list 上没有那么快。

Redis Lists 用 linked list 实现的原因是：对于数据库系统来说，至关重要的特性是：能非常快的在很大的列表上添加元素。另一个重要因素是，正如你将要看到的： Redis lists 能在常数时间取得常数长度。如果快速访问集合元素很重要，建议使用可排序集合（sorted sets）。可排序集合我们会随后介绍。



#### Redis lists 入门

LPUSH 命令可向 list 的左边（头部）添加一个新元素，而 RPUSH 命令可向 list 的右边（尾部）添加一个新元素。最后 LRANGE 命令可从 list 中取出一定范围的元素：

```
> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
```

LRANGE key start stop

下标（index）参数 start 和 stop 都以 0 为底，也就是说，以 0 表示列表的第一个元素，以 1表示列表的第二个元素，以此类推。

你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。



上面的所有命令的参数都可变，方便你一次向 list 存入多个值。

```
> rpush mylist 1 2 3 4 5 "foo bar"
(integer) 9
> lrange mylist 0 -1
1) "first"
2) "A"
3) "B"
4) "1"
5) "2"
6) "3"
7) "4"
8) "5"
9) "foo bar"
```

还有一个重要的命令是 pop，它从 list 中删除元素并同时返回删除的值。可以在左边或右边操作。

```
> rpush mylist a b c
(integer) 3
> rpop mylist
"c"
> rpop mylist
"b"
> rpop mylist
"a"
> rpop mylist
(nil)
```



#### List 的常用案例

Lists 对很多案例都很有用，以下是两个最有代表性的使用案例：

- 记录用户发布到社交网络的最新更新。
- 进程之间的通信，使用生产者 - 消费者模式，生产者将消息推入列表，消费者（通常是 worker 进程）消费这些消息并执行操作。 Redis 具有特殊的 list 命令，能够使这种用法更加可靠和高效。

例如，流行的 Ruby 库 resque 和 sidekiq 都使用 Redis list 来实现后台作业

大名鼎鼎的 Twitter 就将用户发布的最新推文数据收录到 Redis 列表中。

要逐步描述常见用例，请假设您的主页显示在照片共享社交网络中发布的最新照片，并且您希望加速访问。

- 每一次一个用户上传一张新的照片，我们用 LPUSH 命令给 list 添加它的 id。
- 当用户访问自己的主页，我们使用 LRANGE 0 9 去获取最新的 10 条数据。



#### Capped lists

在许多案例中我们仅仅希望使用 lists 去存储最新的数据记录，不管它们是社交网络更新的数据，日志或是什么别的东西。

Redis 允许我们使用 lists 作为一个定容集合，只记住最新的 N 条数据记录并使用 LTRIM 命令丢弃所有最旧的数据记录。

可以使用 LTRIM 把 list 从左边截取指定长度。

```
> rpush mylist 1 2 3 4 5
(integer) 5
> ltrim mylist 0 2
OK
> lrange mylist 0 -1
1) "1"
2) "2"
3) "3"
```

通过 LTRIM 命令我们可以使用一个非常简单但有用的模式：同时执行 List push 操作 +  List trim 操作以添加新元素并丢弃超出限制的元素。

```
LPUSH mylist <some element>
LTRIM mylist 0 999
```

以上命令为 list 新增了一个元素然后取出了最新的 1000 个元素放进 list。使用 LRANGE 命令你可以轻松取得最新的数据记录而不需要特意记录老的数据记录。

注意：因为 LRANGE 命令的时间复杂度是 O(N)，访问首尾的数据集的时间复杂度都是常数级的。



#### List 上的阻塞操作

列表具有使其适合实现队列的特性，并且通常作为进程间通信系统的构建块：阻塞操作。

假设你想要通过一个进程把一些数据储存进 list 里，然后通过另一个进程操作这些数据。这不就是生产者和消费者模型吗，我们可以通过下面的方式简单的完成。

- 通过 LPUSH 命令存储数据，实现生产者的功能
- 通过 RPOP 命令提取数据，实现消费者的功能

但我们会遇到 list 为空的情景，这时候消费者就需要轮询来获取数据，这样就会增加 redis 的访问压力、增加消费端的 cpu 时间，而很多访问都是无用的。为此 redis 提供了阻塞式访问 BRPOP 和 BLPOP 命令。消费者可以在获取数据时指定数据不存在时的阻塞时间，如果在时限内获得数据则立即返回，如果超时还没有数据则返回 null，阻塞时间设置为 0 表示一直阻塞。你还可以指定多个 list 而不仅仅是一个，以便同时在多个 list 上等待，并在第一个 list 接收到元素时收到通知。

关于 BRPOP 命令需要注意的几点：

1. 客户端接收的数据是有序的：第一个阻塞的客户端会第一个接收到新增的数据。
2. 与 RPOP 相比，返回值是不同的：它是一个双元素数组，因为它还包含键的名称，因为 BRPOP 和 BLPOP 能够阻塞等待来自多个列表的元素
3. 阻塞时间过期时，会返回 null



#### key 的自动创建和删除

目前为止，在我们的例子中，我们没有在推入元素之前创建空的 list，或者在 list 没有元素时删除它。在 list 为空时删除 key，并在用户试图添加元素（比如通过 LPUSH ）而键不存在时创建空 list，是 Redis 的职责。

这不光适用于 lists，还适用于所有包括多个元素的 Redis 数据类型 —— Sets，Sorted Sets 和 Hashes。基本上，我们可以用三条规则来概括它的行为：

1. 当我们向一个聚合数据类型中添加元素时，如果目标键不存在，就在添加元素前创建空的聚合数据类型。
2. 当我们从聚合数据类型中移除元素时，如果值空了，键自动被销毁。
3. 对一个空的 key 调用一个只读的命令，比如 LLEN （返回 list 的长度），或者一个删除元素的命令，将总是产生同样的结果。该结果和对一个空的聚合类型做同个操作的结果是一样的。



#### Redis Hashes

Redis hash 看起来就像一个 ”hash“ 的样子，由键值对组成：

```
> hmset user:1000 username antirez birthyear 1977 verified 1
OK
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```

Hash 便于表示 objects，实际上，你可以放入一个 hash 的域数量实际上没有限制（除了可用内存以外）。所以，你可以在你的应用中以不同的方式使用 hash。

HMSET 指令设置 hash 中的多个域，而 HGET 取回单个域。 HMGET 和 HGET 类似，但返回一系列值：

Redis 有检测成员的指令。一个特定的元素是否存在？

```
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```

“3” 是 set 的一个成员，而 “30” 不是。

Sets 适合用于表示对象间的关系。 例如，我们可以轻易使用 set 来表示标记。

一个简单的建模方式是，对每一个希望标记的对象使用 set。这个 set 包含和对象相关联的标签的 ID。

假设我们想要给新闻打上标签。 假设新闻 ID 1000 被打上了 1,2,5 和 77 四个标签，我们可以使用一个 set 把 tag ID 和新闻条目关联起来：

```
> sadd news:1000:tags 1 2 5 77
(integer) 4
```

但是，有时候我们可能也会需要相反的关系：所有被打上相同标签的新闻列表：

```
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:77:news 1000
(integer) 1
```

获取一个对象的所有 tag 是很方便的：

```
> smembers news:1000:tags
1. 5
2. 1
3. 77
4. 2
```

注意：在这个例子中，我们假设你有另一个数据结构，比如一个 Redis hash，把标签 ID 对应到标签名称。

使用 Redis 命令行，我们可以轻易实现其他一些有用的操作。比如，我们可能需要一个含有 1，2，10 和 27 标签的对象的列表。我们可以用 SINTER 命令来完成这件事。它获取不同 set 的交集。我们可以用：

```
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```

不光可以取交集，还可以取并集，差集，获取随机元素，等等。

获取一个元素的命令是 SPOP，它很适合对特定问题建模。比如，要实现一个基于 web 的扑克游戏，你可能需要用 set 来表示一副牌。假设我们用一个字符的前缀来表示不同花色。

```
>  sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
   H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
   S7 S8 S9 S10 SJ SQ SK
   (integer) 52
```

现在，我们想要给每个玩家 5 张牌。SPOP 命令删除一个随机元素，把它返回给客户端，因此它是完全合适的操作。

但是，如果我们对我们的牌直接调用它，在下一盘我们就需要重新填满这副牌。开始，我们可以复制 deck 键中的内容，并放入 game:1:deck 键中。

这是通过 SUNIONSTORE 实现的，它通常用于对多个集合取并集，并把结果存入另一个 set 中。但是，因为一个 set 的并集就是它本身，我可以这样复制我的牌：



















