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