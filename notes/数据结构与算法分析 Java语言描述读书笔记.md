## 数据结构与算法分析 Java语言描述读书笔记

### 第 3 章 表、栈和队列

本章讨论最简单和最基本的三种数据结构。



#### 3.1	抽象数据类型

**抽象数据类型**（abstract data type，ADT）是带有一组操作的一些对象的集合。抽象数据类型是数学的抽象；在 ADT 的定义中没有地方提到关于这组操作是如何实现的任何解释。诸如表、集合、图以及与它们各自的操作一起形成的这些对象都可以被看做是抽象数据类型，这就像整数、实数、布尔数都是数据类型一样。整数、实数和布尔数各自都有与之相关的操作，而抽象数据类型也是如此。对于集合 ADT，可以有像添加（add）、删除（remove）以及包含（contain）这样一些操作。当然，也可以只要两种操作并（union）和查找（find），这两种操作又在这个集合上定义了一种不同的 ADT。

Java 类也考虑到 ADT 的实现，不过适当地隐藏了实现的细节。这样，程序中需要对 ADT 实施操作的任何其他部分可以通过调用适当的方法来进行。如果由于某种原因需要改变实现的细节，那么通过仅仅改变执行这些 ADT 操作的例程应该是很容易做到的。这种改变对于程序的其余部分是完全透明的。

对于每种 ADT 并不存在什么法则来告诉我们必须要有哪些操作，这是一个设计决策。错误处理和结构调整（在适当的地方）一般也取决于程序的设计者。本章中将要讨论的这三种数据结构是 ADT 的最基本的例子。我们将会看到它们中的每一种是如何以多种方法实现的，不过，当它们被正确地实现以后，使用它们的程序却没有必要知道它们是如何实现的。



#### 3.2	表 ADT

我们将处理形如 A~0~，A~1~，A~2~ 。。。A~N-1~ 的一般的表。我们说这个表的大小是 N。我们将大小为 0 的特殊的表称为**空表**（empty list）。

对于除空表外的任何表，我们说 A~i~ 后继 A~i-1~ 并称 A~i-1~ 前驱 A~i~ 。表中的第一个元素是 A~0~ ，而最后一个元素是 A~N-1~ 。我们将不定义 A~0~ 的前驱元，也不定义 A~n-1~ 的后继元。元素 A~i~ 在表中的位置为 i+1。为了简单起见，我们假设表中的元素是整数，但一般说来任意的复元素也是允许的（而且容易由 Java 泛型类处理）。

与这些 “定义” 相关的是要在表 ADT 上进行操作的集合。printList 和 makeEmpty 是常用的操作，其功能显而易见；find 返回某一项首次出现的位置；insert 和 remove 一般是从表的某个位置插入和删除某个元素；而 findKth 则返回（作为参数而被指定的）某个位置上的元素。

当然，一个方法的功能怎样才算恰当，完全要由程序设计者来确定，就像对特殊情况的处理那样。我们还可以添加一些操作，比如 next 和 previous，它们会取一个位置作为参数并分别返回其后继元和前驱元的位置。



##### 3.2.1	表的简单数组实现

对表的所有这些操作都可以通过使用数组来实现。虽然数组是由固定容量创建的，但在需要的时候可以用双倍的容量创建一个不同的数组。它解决由于使用数组而产生的最严重的问题，即从历史上看为了使用一个数组，需要对表的大小进行估计。而这种估计在 Java 或任何现代编程语言中都是不需要的。下列程序段解释一个数组 arr 在必要的时候如何被扩展（其初始长度为 10）：

```java
int[] arr = new int[10];
...
//下面我们决定需要扩大 arr
int[] newArr = new int[arr.length * 2];
for (int i = 0; i < arr.length; i++) {
    newArr[i] = arr[i];
}
arr = newArr;
```



数组的实现可以使得 printList 以线性时间被执行，而 findKth 操作则花费常数时间，这正是我们所能预期的。不过，插入和删除的花费却潜藏着昂贵的开销，这要看插入和删除发生在什么地方。最坏的情形下，在位置 0 的插入（即在表的前端插入）首先需要将整个数组后移一个位置以空出空间来，而删除第一个元素则需要将表中的所有元素前移一个位置，因此这两种操作的最坏情况为 O(N)。平均来看，这两种操作都需要移动表的一半的元素，因此仍然需要线性时间。另一方面，如果所有的操作（**笔记**：应该是指插入删除操作）都发生在表的高端（high end），那就没有元素需要移动，而添加和删除则只花费 O(1) 时间。

存在许多情形，在这些情形下的表是通过在高端进行插入操作建成的，其后只发生对数组的访问（即只有 findKth 操作）。在这种情况下，数组是表的一种恰当的实现。然而，如果发生对表的一些插入和删除操作，特别是对表的前端（front end）进行，那么数组就不是一种好的选择。下一节我们将讨论另一种可供选择的方案：**链表**（linked list）。



##### 3.2.2	简单链表

为了避免插入和删除的线性开销，我们需要保证表可以不连续存储，否则表的每个部分都可能需要整体移动。图 3-1 指出**链表**的一般想法。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE1.jpg"/>
</div>

链表由一系列节点组成，这些节点不必在内存中相连。每一个节点均含有表元素和到包含该元素后继元的节点的**链**（link）。我们称之为 next 链。最后一个单元的 next 链引用 null。

为了执行 printList 或 find(x)，只要从表的第一个节点开始然后用一些后继的 next 链遍历该表即可。这种操作显然是线性时间的，和在数组实现时一样，不过其中的常数可能会比用数组实现时要大。findKth 操作不如数组实现时的效率高；findKth(i) 花费 O(i) 的时间并以这种明显的方式遍历链表而完成。

remove 方法可以通过修改一个 next 引用来实现。图 3-2 给出在原表中删除第三个元素的结果。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE2.jpg"/>
</div>

insert 方法需要使用 new 操作符从系统取得一个新节点，此后执行两次引用的调整。其一般想法在图 3-3 中给出，其中的虚线表示原来的 next 引用。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE3.jpg"/>
</div>

我们看到，在实践中如果知道变动将要发生的地方，那么向链表插入或从链表中删除一项的操作不需要移动很多的项，而只涉及常数个节点链的改变。

在表的前端添加项或删除第一项的特殊情形此时也属于常数时间的操作，当然要假设到链表前端的链是存在的。只要我们拥有到链表最后节点的链，那么在链表末尾进行添加操作的特殊情形（即让新的项成为最后一项）可以花费常数时间。因此，典型的链表拥有到该表两端的链。删除最后一项比较复杂，因为必须找出指向最后节点的项，把它的 next 链改成 null，然后再更新持有最后节点的链。在经典的链表中，每个节点均存储到其下一节点的链，而拥有指向最后节点的链并不提供最后节点的前驱节点的任何信息。

保留指向最后节点的节点的第 3 个链的想法行不通，因为它在删除操作期间也需要更新。我们的做法是，让每一个节点持有一个指向它在表中的前驱节点的链，如图 3-4 所示，我们称之为**双链表**（doubly linked list）。

 <div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE4.jpg"/>
</div>



#### 3.3	Java Collections API 中的表

在类库中，Java 语言包含有一些普通数据结构的实现。该语言的这一部分通常叫做 **Collections API**。表 ADT 是在 Collections API 中实现的数据结构之一。我们将在第 4 章看到其他一些数据结构。



##### 3.3.1	Collections 接口

Collections API 位于 java.util 包中。集合（collection）的概念在 Collection 接口中得到抽象，它存储一组类型相同的对象。图 3-5 显示该接口最重要的部分（但一些方法未被显示）。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE5.jpg"/>
</div>

在 Collection 接口中的许多方法所做的工作由它们的英文名称可以看出，因此 size 返回集合中的项数；isEmpty 返回 true 当且仅当集合的大小为 0。如果 x 在集合中，则 contains 返回 true。注意，这个接口并不规定集合如何决定 x 是否属于该集合——这要由实现该 Collection 接口的具体的类来确定。add 和 remove 从集合中添加和删除 x，如果操作成功则返回 true，如果因某个看似有理（非异常）的原因失败则返回 false。例如，如果要删除的项不在集合中，则 remove 可能失败，而如果特定的集合不允许重复，那么当企图插入一项重复项时，add 操作就可能失败。

Collection 接口扩展了 Iterable 接口。实现 Iterable 接口的那些类可以拥有增强的 for 循环，该循环施于这些类之上以观察它们所有的项。例如，图 3-6 中的例程可以用来打印任意集合中的所有的项。这种方式的 print 的实现和当 coll 具有类型 AnyType[] 时能够使用的相应的实现是完全相同的，它们逐个字符都是一样的。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE6.jpg"/>
</div>



##### 3.3.2	Iterator 接口

实现 Iterable 接口的结合必须提供一个称为 iterator 的方法，该方法返回一个 Iterator 类型的对象。该 Iterator 是一个在 java.util 包中定义的接口，见图 3-7。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE7.jpg"/>
</div>

Iterator 接口的思路是，通过 iterator 方法，每个集合均可创建并返回给客户一个实现 Iterator 接口的对象，并将当前位置的概念在对象内部存储下来。

每次对 next 的调用都给出集合的（尚未见到的）下一项。因此，第 1 次调用 next 给出第 1 项，第 2 次调用给出第 2 项，等等。hasNext 用来告诉是否存在下一项。当编译器见到一个正在用于 Iterable 的对象的增强的 for 循环的时候，它用对 iterator 方法的那些调用代替增强的 for 循环以得到一个 Iterator 对象，然后调用 next 和 hasNext。因此，前面看到的 print 例程由编译器重写，见图 3-8 所示。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE8.jpg"/>
</div>

由于 Iterator 接口中的现有方法有限，因此，很难使用 Iterator 做简单遍历 Collection 以外的任何工作。Iterator 接口还包含一个方法，叫做 remove。该方法可以删除由 next 最新返回的项（以后，我们不能再调用 remove，直到对 next 再一次调用以后）。虽然 Collection 接口也包含一个 remove 方法，但是，使用 Iterator 的 remove 方法可能有更多的优点。

Iterator 的 remove 方法的主要优点在于，Collection 的 remove 方法必须首先找出要被删除的项。如果知道所要删除的项的准确位置，那么删除它的开销很可能要小得多。下一节我们将要看到一个例子，是在集合中每隔一项删除一项。这个程序用迭代器（iterator）很容易编写，而且比用 Collection 的 remove 方法潜藏着更高的效率。

当直接使用 Iterator（而不是通过一个增强的 for 循环间接使用）时，重要的是要记住一个基本法则：如果对正在被迭代的集合进行结构上的改变（即对该集合使用 add、remove 或 clear 方法），那么迭代器就不再合法（并且在其后使用该迭代器时将会有 ConcurrentModificationException 异常被抛出）。为避免迭代器准备给出某一项作为下一项（next item）而该项此后或者被删除，或者也许一个新的项正好插入该项的前面这样一些讨厌的情形，有必要记住上述法则。这意味着，只有在需要立即使用一个迭代器的时候，我们才应该获取迭代器。然而，如果迭代器调用了它自己的 remove 方法，那么这个迭代器就仍然是合法的。这是有时候我们更愿意使用迭代器的 remove 方法的第二个原因。



##### 3.3.3	List 接口、ArrayList 类和 LinkedList 类

本节跟我们关系最大的集合就是表（list），它由 java.util 包中的 List 接口指定。List 接口继承了 Collection 接口，因此它包含 Collection 接口的所有方法，外加其他一些方法。图 3-9 解释其中最重要的一些方法。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE9.jpg"/>
</div>

get 和 set 方法使得用户可以访问或改变通过由位置索引 idx 给定的表中指定位置上的项。索引 0 位于表的前端，索引 size() - 1 代表表中的最后一项，而索引 size() 则表示新添加的项可以被放置的位置。add 使得在位置 idx 处置入一个新的项（并把其后的项向后推移一个位置）。于是，在位置 0 处 add 是在表的前端进行的添加，而在位置 size() 处的 add 是把被添加项作为新的最后项添入表中。除以 AnyType 作为参数的标准的 remove 外，remove 还被重载以删除指定位置上的项。最后，List 接口指定 listIterator 方法，它将产生比通常认为的还要复杂的迭代器。ListIterator 接口将在 3.3.5 节讨论。

List ADT 有两种流行的实现方式。ArrayList 类提供了 List ADT 的一种可增长数组的实现。使用 ArrayList 的优点在于，对 get 和 set 的调用花费常数时间。其缺点是新项的插入和现有项的删除代价昂贵，除非变动是在 ArrayList 的末端进行。LinkedList 类则提供了 List ADT 的双链表实现。使用 LinkedList 的优点在于，新项的插入和现有项的删除均开销很小，这里假设变动项的位置是已知的。这意味着，在表的前端进行添加和删除都是常数时间的操作，由此 LinkedList 更提供了方法 addFirst 和 removeFirst、addLast 和 removeLast、以及 getFirst 和 getLast 等以有效地添加、删除和访问表两端的项。使用 LinkedList 的缺点是它不容易作索引，因此对 get 的调用是昂贵的，除非调用非常接近表的端点（如果对 get 的调用是对接近表后部的项进行，那么搜索的进行可以从表的后部开始）。为了看出差别，我们考察对一个 List 进行操作的某些方法。首先，设我们通过在末端添加一些项来构造一个 List。

```java
public static void makeList1(List<Integer> lst, int N) {
	lst.clear();
	for (int i = 0; i < N; i++) {
		lst.add(i);
	}
}
```



不管 ArrayList 还是 LinkedList 作为参数被传递，makeList1 的运行时间都是 O(N)，因为对 add 的每次调用都是在表的末端进行从而均花费常数时间（可以忽略对 ArrayList 偶尔进行的扩展）。另一方面，如果我们通过在表的前端添加一些项来构造一个 List：

```java
public static void makeList2(List<Integer> lst, int N) {
	lst.clear();
	for (int i = 0; i < N; i++) {
		lst.add(0, i);
	}
}
```



那么，对于 LinkedList 它的运行时间是 O(N)，但是对于 ArrayList 其运行时间则是 O(N^2^)，因为在 ArrayList 中，在前端进行添加是一个 O(N) 操作。

下一个例程是计算 List 中的数的和：

```java
public static int sum(List<Integer> lst) {
    int total = 0;
    for (int i = 0; i < N; i++) {
        total += lst.get(i);
    }
    return total;
}
```



这里，ArrayList 的运行时间是 O(N)，但对于 LinkedList 来说，其运行时间则是 O(N^2^)，因为在 LinkedList 中，对 get 的调用为 O(N) 操作。可是，要是使用一个增强的 for 循环，那么它对任意 List 的运行时间都是 O(N)，因为迭代器将有效地从一项到下一项推进。

对搜索而言，ArrayList 和 LinkedList 都是低效的，对 Collection 的 contains 和 remove 两个方法（它们都以 AnyType 为参数）的调用均花费线性时间。

在 ArrayList 中有一个容量的概念，它表示基础数组的大小。在需要的时候，ArrayList 将自动增加其容量以保证它至少具有表的大小。如果该大小的早期估计存在，那么 ensureCapacity 可以设置容量为一个足够大的量以避免数组容量以后的扩展。再有，trimToSize 可以在所有的 ArrayList 添加操作完成之后使用以避免浪费空间。



##### 3.3.4	例子：remove 方法对 LinkedList 类的使用

作为一个例子，我们提供一个例程，将一个表中所有具有偶数值的项删除。于是，如果表包含 6，5，1，4，2，则在该方法调用之后，表中仅有元素 5，1。

当遇到表中的项时将其从表中删除的算法有几种可能的想法：当然，一种想法是构造一个包含所有的奇数的新表，然后清除原表，并将这些奇数拷贝回原表。不过，我们更有兴趣的是写一个干净的避免拷贝的表，并在遇到那些偶数值时将它们从表中删除。

对于 ArrayList 这几乎就是一个失败策略。因为从一个 ArrayList 的几乎是任意的地方进行删除都是昂贵的操作。不过，在 LinkedList 中却存在某种希望，因为我们知道，从已知位置的删除操作都可以通过重新安排某些链而被有效地完成。

图 3-10 显示第一种想法。在一个 ArrayList 上，我们知道，remove 的效率不是很高的，因此该程序花费的是二次时间。LinkedList 暴露两个问题。首先，对 get 调用的效率不高，因此例程花费二次时间。而且，对 remove 的调用同样地低效，因为达到位置 i 的代价是昂贵的。（**笔记**：细看 JDK 的源码，就可以发现，LinkedList 的 remove(int index) 和 remove(Object o) 这两个方法都做不到 O(1) 的时间，而是 O(n)。这是因为上面说的数据结构中的 O(1) 时间，是对于某个已经确定的节点。而 LinkedList 中，首先必须通过一个循环，找到第一个出现的 Object o，或者走到 index 这个位置，再进行操作。也就是，有一个 get 的过程。）

```java
//图 3-10  删除表中的偶数：算法对所有类型的表都是二次的
public static void removeEvensVer1(List<Integer> lst) {
    int i = 0;
    while (i < lst.size()) {
        if(lst.get(i) % 2 == 0) {
            lst.remove(i);
        } else {
            i++;
        }
    }
}
```



图 3-11 显示矫正该问题的一种思路。我们不是用 get，而是使用一个迭代器一步步遍历该表。这是高效率的。但是我们使用 Collection 的 remove 方法来删除一个偶数值的项。这不是高效的操作，因为 remove 方法必须再次搜索该项，它花费线性时间。但是我们运行这个程序会发现情况更糟：该程序产生一个异常，因为当一项被删除时，由增强的 for 循环所使用的迭代器是非法的。（图 3-10 中的代码解释为什么这样的原因：我们不能期待增强的 for 循环懂得只有当一项不被删除时它才必须向前推进。）

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE10.jpg"/>
</div>

图 3-12 指出一种成功的想法：在迭代器找到一个偶数值项之后，我们可以使用该迭代器来删除这个它刚看到的值。对于一个 LinkedList，对该迭代器的 remove 方法的调用只花费常数时间，因为该迭代器位于需要被删除的节点（或在其附近）。因此，对于 LinkedList，整个程序花费线性时间，而不是二次时间。对于一个 ArrayList，即使迭代器位于需要被删除的节点上，其 remove 方法仍然是昂贵的，因为数组的项必须要移动，正如所料，对于 ArrayList，整个程序仍然花费二次时间。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE11.jpg"/>
</div>

如果我们传递一个 LinkedList<Integer> 运行图 3-12 中的程序，对于一个 400 000 项的 lst，花费的时间是 0.031 秒，而对于一个 800 000 项的 LinkedList 则花费 0.062 秒，显然这是线性时间例程，因为运行时间于输入大小增加相同的倍数。当我们传递一个 ArrayList<Integer> 时，对于一个 40 000 项的 ArrayList 程序几乎花费 2.5 分钟，而对于 800 000 项的 ArrayList 程序花费大约 10 分钟；当输入增加到 2 倍时运行时间增加到 4 倍，这与二次的特征是一致的。



##### 3.3.5	关于 ListIterator 接口

图 3-13 指出，ListIterator 扩展了 List 的 Iterator 的功能。方法 previous 和 hasPrevious 使得对表从后向前的遍历得以完成。add 方法将一个新的项以当前位置放入表中。当前项的概念通过把迭代器看做是在对 next 的调用所给出的项和对 previous 的调用所给出的项之间抽象出来的。图 3-14 解释了这种抽象。对于 LinkedList 来说，add 是一种常数时间的操作，但对于 ArrayList 则代价昂贵。set 改变倍迭代器看到的最后一个值，从而对 LinkedList 很方便。例如，它可以用来从 List 的所有的偶数中减去 1，而这对于 LinkedList 来说，不使用 ListIterator 的 set 方法是很难做到的。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE12.jpg"/>
</div>



#### 3.4	ArrayList 类的实现

在这一节，我们提供便于使用的 ArrayList 泛型类的实现。为避免与类库中的类相混，这里将把我们的类叫作 myArrayList。我们不提供 MyCollection 或 MyList 接口；MyArrayList 是独立的。在考查 MyArrayList 代码（接近 100 行）之前，先概括主要的细节。

1. MyArrayList 将保持基础数组，数组的容量，以及存储在 MyArrayList 中的当前项数。
2. MyArrayList  将提供一种机制以改变基础数组的容量。通过获得一个新数组，将老数组拷贝到新数组中来改变数组的容量，运行虚拟机回收老数组。
3. MyArrayList 将提供 get 和 set 的实现。
4. MyArrayList 将提供基本的例程，如 size、isEmpty 和 clear，它们是典型的单行程序；还提供 remove，以及两种不同版本的 add。如果数组的大小和容量相同，那么这两个 add 例程将增加容量。
5. MyArrayList 将提供一个实现 Iterator 接口的类。这个类将存储迭代序列中的下一项的下标，并提供 next、hasNext 和 remove 等方法的实现。MyArrayList 的迭代器方法直接返回实现 Iterator 接口的该类的新构造的实例。



##### 3.4.1	基本类

图 3-15 和图 3-16 显示 MyArrayList 类。像它的 Collections API 的对应类一样，存在某种错误检测以保证合理的限界；然而，为了把精力集中在编写迭代器类的基本方面，我们不检测可能使得迭代器无效的结构上的修改，也不检测非法的迭代器 remove 方法。这些检测将在此后的 3.5 节 MyLinkedList 的实现中指出，（**笔记**：迭代器错误检测）对于两种列表实现都是完全相同的。

```java
public class MyArrayList<AnyType> implements Iterable<AnyType> {

    private static final int DEFAULT_CAPACITY = 10;

    private int theSize;
    private AnyType[] theItems;

    public MyArrayList() {
        doClear();
    }

    public void clear() {
        doClear();
    }

    public void doClear() {
        theSize = 0;
        ensureCapacity(DEFAULT_CAPACITY);
    }

    public int size() {
        return theSize;
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    public void trimToSize() {
        ensureCapacity(size());
    }

    public AnyType get(int idx) {
        if(idx < 0 || idx >= size()) {
            throw new ArrayIndexOutOfBoundsException();
        }
        return theItems[idx];
    }

    public AnyType set(int idx, AnyType newVal) {
        if(idx < 0 || idx <= size()) {
            throw new ArrayIndexOutOfBoundsException();
        }
        AnyType old = theItems[idx];
        theItems[idx] = newVal;
        return old;
    }

    public void ensureCapacity(int newCapacity) {
        if(newCapacity < theSize) {
            return;
        }

        AnyType[] old = theItems;
        theItems = (AnyType [])new Object[newCapacity];
        for (int i = 0; i < size(); i++) {
            theItems[i] = old[i];
        }
    }

    public boolean add(AnyType x) {
        add(size(), x);
        return true;
    }

    public void add(int idx, AnyType x) {
        if (theItems.length == size()) {
            ensureCapacity(size()*2 + 1);
        }
        for (int i = theSize; i > idx; i--) {
            theItems[i] = theItems[i - 1];
        }
        theItems[idx] = x;

        theSize++;
    }

    public AnyType remove(int idx) {
        AnyType removedItem = theItems[idx];
        for (int i = idx; i < size() - 1; i++) {
            theItems[i] = theItems[i + 1];
        }
        theSize--;
        return removedItem;
    }

    public java.util.Iterator<AnyType> iterator() {
        return new ArrayListIterator();
    }

    private class ArrayListIterator implements java.util.Iterator<AnyType> {
        private int current = 0;

        public boolean hasNext() {
            return current < size();
        }

        public AnyType next() {
            if (!hasNext()) {
                throw new java.util.NoSuchElementException();
            }
            return theItems[current++];
        }

        public void remove() {
            MyArrayList.this.remove(--current);
        }
    }

}
```



如 5 到 6 行所示，MyArrayList 把大小及数组作为其数据成员进行存储。

从 11 行到 38 行，是几个短例程，即 clear、size、trimToSize、inEmpty、get 以及 set 的实现。

ensureCapacity 例程如 40 行到 49 行所示。容量的扩充是用与早先描述的相同的方法来完成的：第 45 行存储对原始数组的一个引用，第 46 行是为新数组分配内存，并在第 47 行和 48 行将旧内容拷贝到新数组中。如 42 行和 43 行所示，例程 ensureCapacity 也可以用于收缩基础数组，不过只是当指定的新容量至少和原大小一样时才适用。否则，ensureCapacity 的要求将被忽略。在第 46 行，我们看到一个短语是必需的，因为泛型数组的创建是非法的。我们的做法是创建一个泛型类型限界的数组，然后使用一个数组进行类型转换。这将产生一个编译器警告，但在泛型集合的实现中这是不可避免的。

图中显示了两个版本的 add。第一个 add 是添加到表的末端并通过调用添加到指定位置的较一般的版本而得以简单实现。这种版本从计算上来说是昂贵的，因为它需要移动在指定位置上或指定位置后面的那些元素到一个更高的位置上。add 方法可能要求增加容量。扩充容量的代价是非常昂贵的，因此，如果容量被扩充，那么，它就要变成原来大小的两倍，以避免不得不再次改变容量，除非大小戏剧性地增加（+1 用于大小是 0 的情形）；

remove 方法类似于 add，只是那些位于指定位置上或指定位置后的元素向低位移动一个位置。

剩下的例程处理 iterator 方法和相关迭代器的实现。在图 3-16 中由第 77 行至第 96 行显示。iterator 方法直接返回 ArrayListIterator 类的一个实例，该类是一个实现 Iterator 接口的类。ArrayListIterator 存储当前位置的概念，并提供 hasNext、next 和 remove 的实现。当前位置表示要查看的下一元素（的数组下标），因此初始时当前位置为 0。



##### 3.4.2	迭代器、Java 嵌套类和内部类

ArrayListIterator 使用一个复杂 Java 结构，叫作**内部类**（inner class）。显然该类在 MyArrayList 类内部被声明，这是被许多语言支持的特性。然而，Java 中的内部类具有更微妙的性质。

为了了解内部类是如何工作的，图 3-17 描绘了迭代器的思路（不过，代码有缺欠），使 ArrayListIterator 成为一个顶级类。我们只着重讨论 MyArrayList 的数据域、MyArrayList 中的 iterator 方法以及 ArrayListIterator 类（而不是它的 remove 方法）。







在图 3-17 中，ArrayListIterator 是泛型类，它存储当前位置，程序在 next 方法中试图使用当前位置作为下标访问数组元素然后将当前位置向后推进。注意，如果 arr 是一个数组，则 aee[idx++] 对数组使用 idx，然后向后推进 idx。图 3-17 中的问题在于，theItems[current++] 是非法的，因为 theItems 不是 ArrayListIterator 的一部分；它是 MyArrayList 的一部分。因此程序根本没有意义。

最简单的解决方案见图 3-18，不过它也有缺点，但是以更微小的方式呈现。在图 3-18 中，我们通过让迭代器存储 MyArrayList 的引用来解决在迭代器中没有数组的问题。这个引用是第二个数据域，是通过 ArrayListIterator 的一个新的单参数构造器而被初始化的。既然有一个 MyArrayList 的引用，那么就可以访问包含于 MyArrayList 中的数组域（还可得到 MyArrayList 的大小，该大小在 hasNext 中是需要的）。







图 3-18 中的问题在于，theItems 是 MyArrayList 中的私有（private）域，而由于 ArrayListIterator 是一个不同的类，因此在 next 方法中访问 theItems 是非法的。最简单的修正方法是改变 theItems 在 MyArrayList 中的可见性，从 private 改成某种稍宽松的可见性（如 public，或默认的可见性，它也被称为**包可见性**（package visibility））。不过，这违反了良好的面向对象编程的基本原则，它要求数据应尽可能地隐蔽。

图 3-19 显示另一种解决方案，这种方案能够正确运行：使 ArrayListIterator 为**嵌套类**（nested class）。当我们让 ArrayListIterator 为一个嵌套类时，该类将被放入另一个类（此时就是 MyArrayList）的内部，这个类就叫作**外部类**（outer class）。我们必须用 static 来表示它是嵌套的；若无 static，将得到一个内部类，这有时候好，有时候也不好。嵌套类是许多编程语言的典型的类型。注意，嵌套类可以被设计成 private，这很好，因为此时该嵌套类除能够被外部类 MyArrayList 访问外，其他是不可访问的。更为重要的是，因为嵌套类被认为是外部类的一部分，所以不存在产生不可见问题：theItems 是 MyArrayList 类的可见成员，因为 next 是 MyArrayList 的一部分。







既然我们有了嵌套类，那么就可以讨论内部类。嵌套类的问题在于，在我们的原始设计中，当编写 theItems 而不引用其所在的 MyArrayList 的时候，代码看起来还可以，也似乎有意义，但却是无效的，因为编译器不可能计算出哪个 MyArrayList 在被引用。要是我们自己不必明了这一点那就好多了，而这恰是内部类要求我们所要做的。

当声明一个内部类时，编译器则添加对外部类对象的一个隐式引用，该对象引起内部类对象的构造。如果外部类的名字是 Outer，则隐式引用就是 Outer.this。因此，如果 ArrayListIterator 是作为一个内部类被声明而且没有注明 static，那么 MyArrayList.this 和 theList 就都会引用同一个 MyArrayList。这样，theList 就是多余的，并可能被删除。

在每一个内部类的对象都恰好与外部类对象的一个实例相关联的情况下，内部类是有用的。在这种情况下，内部类的对象在没有外部类对象与其关联时是永远不可能存在的。对于 MyArrayList 及其迭代器的情形，图 3-20 指出了 MyArrayList 类和迭代器之间的关系，此时这些内部类都用来实现该迭代器。







theList.theItems 的使用可以由 MyArrayList.this.theItems 代替。这很难说是一种改进，但进一步的简化还是可能的。正如 this.data 可以简写为 data 一样（假设不存在引起冲突的也叫作 data 的另外的变量），MyArrayList.this.theItems 可以简写为 theItems。图 3-21 指出 ArrayListIterator 的简化。







首先，ArrayListIterator 是隐式的泛型类，因为它现在依赖于 MyArrayList，而后者是泛型的；我们可以不必说这些。

其次，theList 没有了，我们用 size() 和 theItems[current ++] 作为 MyArrayList.this.size() 和 MyArrayList.this.theItems[current++] 的简记符。theList 作为数据成员，它的去除也删除了相关的构造器，因此程序又转变成 1 号版本的样式。

我们可以通过调用 MyArrayList 的 remove 来实现迭代器的 remove 方法。由于迭代器的 remove 可能与 MyArrayList 的 remove 冲突，因此我们必须使用 MyArrayList.this.remove。注意，在该项被删除之后，一些元素需要移动，因此 current 被视为同一元素也必须移动。于是，我们使用 -- 而不是 -1。

内部类为 Java 程序员带来句法上的便利。它们不需要编写任何 Java 代码，但是它们在语言中的出现使 Java 程序员以自然的方式（如 1 号版本那样）编写程序，而编译器则编写使内部类对象和外部类对象相关联所需要的附加代码。



#### 3.5	LinkedList 类的实现

本节给出可以使用的 LinkedList 泛型类的实现。和在 ArrayList 类中的情形一样，我们这里的链表类将叫作 MyLinkedList 以避免与库中的类相混。

前面提到，LinkedList 将作为双链表来实现，而且我们还需要保留到该表两端的引用。这样做可以保持每个操作花费常数时间的代价，只要操作发生在已知的位置。这个已知的位置可以是端点，也可以是由迭代器指定的一个位置（不过，我们不实现 ListIterator，因此有些代码留给读者去完成）。

在考虑设计方面，我们将需要提供三个类：

1.MyLinkedList 类本身，它包含到两端的链、表的大小以及一些方法。

2.Node 类，它可能是一个私有的嵌套类。一个节点包含数据以及到前一个节点的链和到下一个节点的链，还有一些适当的构造方法。

3.LinkedListIterator 类，该类抽象了位置的概念，是一个私有类，并实现接口 Iterator。它提供了方法 next、hasNext 和 remove 的实现。

由于这些迭代器类存储 “当前节点” 的引用，并且终端标记是一个合理的位置，因此它对于在表的终端创建一个额外的节点来表示终端标记是有意义的。更进一步，我们还能够在表的前端创建一个额外的节点，逻辑上代表开始的标记。这些额外的节点有时候就叫作**标记节点**（sentinel node）；特别地，在前端的节点有时候也叫做**头节点**（header node），而在末端的节点有时候也叫作**尾节点**（tail node）。

使用这些额外节点的优点在于，通过排除许多特殊情形极大地简化了编码。例如，如果我们不是用头节点，那么删除第 1 个节点就变成了一种特殊的情况，因为在删除期间我们必须重新调整链表的到第 1 个节点的链，还因为删除算法一般还要访问被删除节点前面的那个节点（而若无头节点，则第 1 个节点前面没有节点）。图 3-22 显示一个带有头节点和尾节点的双链表。图 3-23 显示一个空链表。图 3-24 则显示 MyLinkedList 类的概要和部分的实现。







我们在第 3 行可以看到私有嵌套 Node 类声明的开头部分。图 3-25 显示这个由所存储的一项组成的 Node 类——它的连接到前一个 Node 的链和下一个 Node 的链，还有一个构造方法。所有的数据成员都是公用的。我们知道，在一个类中，数据成员通常是私有的。然而，在一个嵌套类中的成员甚至在外部类中也是可见的。由于 Node 类是私有的，因此在 Node 类中的那些数据成员的可见性是无关紧要的；那些 MyLinkedList 的方法能够见到所有的 Node 数据成员，而 MyLinkedList 外面的那些类则根本见不到 Node 类。

```java
//图3-24  MyLinkedList类
public class MyLinkedList<AnyType> implements Iterable<AnyType> {

    //图3-25  MyLinkedList类的嵌套Node类
    private static class Node<AnyType> {
        public Node(AnyType d, Node<AnyType> p, Node<AnyType> n) {
            data = d;
            prev = p;
            next = n;
        }

        public AnyType data;
        public Node<AnyType> prev;
        public Node<AnyType> next;
    }

    public MyLinkedList() {
        doClear();
    }

    //图3-26  MyLinkedList类的clear例程
    public void clear() {
        doClear();
    }

    private void doClear() {
        beginMarker = new Node<AnyType>(null, null, null);
        endMarker = new Node<AnyType>(null, beginMarker, null);
        beginMarker.next = endMarker;

        theSize = 0;
        modCount++;
    }

    public int size() {
        return theSize;
    }

    public boolean isEmpty() {
        return size() == 0;
    }

    public boolean add(AnyType x) {
        add(size(), x);
        return true;
    }

    public void add(int idx, AnyType x) {
        addBefore(getNode(idx, 0, size()), x);
    }

    public AnyType get(int idx) {
        return getNode(idx).data;
    }

    public AnyType set(int idx, AnyType newVal) {
        Node<AnyType> p = getNode(idx);
        AnyType oldVal = p.data;
        p.data = newVal;
        return oldVal;
    }

    public AnyType remove(int idx) {
        return remove(getNode(idx));
    }

    /**
     * 图3-28  MyLinkedList类的 add 例程
     * 改变指针而完成向一个双链表中的插入操作
     * Adds an item to this collection, at specified position p
     * Items at or after that position are slid one position higher
     * @param p Node to add before
     * @param x any object
     * @throws IndexOutOfBoundsException if idx is not between 0 and size()
     */
    private void addBefore(Node<AnyType> p, AnyType x) {
        Node<AnyType> newNode = new Node<>(x, p.prev, p);
        newNode.prev.next = newNode;
        p.prev = newNode;
        theSize++;
        modCount++;
    }

    /**
     * 图3-30  MyLinkedList类的remove例程
     * remove p the Node containing the object
     * @param p the Node containing the object
     * @return the item was removed from the collection
     */
    private AnyType remove(Node<AnyType> p) {
        p.next.prev = p.prev;
        p.prev.next = p.next;
        theSize--;
        modCount++;

        return p.data;
    }

    /**
     * 图3-31  MyLinkedList类的私有getNode例程
     * Gets the Node at position idx, which must range from 0 to siez() -1
     * @param idx index to search at
     * @return internal node corresponding to idx
     * @throws IndexOutOfBoundsException if idx is not
     * between 0 and size() - 1, inclusive
     */
    private Node<AnyType> getNode(int idx) {
        return getNode(idx, 0, size() -1 );
    }

    /**
     * Gets the Node at position idx, which must range from lower to upper
     * @param idx index to search at
     * @param lower lowest valid index
     * @param upper upper highest valid index
     * @return internal node corresponding to idx
     * @throws IndexOutOfBoundsException if idx is not
     * between lower and upper, inclusive
     */
    private Node<AnyType> getNode(int idx, int lower, int upper) {
        Node<AnyType> p;

        if (idx < lower || idx > upper) {
            throw new IndexOutOfBoundsException();
        }

        if (idx < size() / 2) {
            p = beginMarker.next;
            for (int i = 0; i < idx; i++) {
                p = p.next;
            }
        } else {
            p = endMarker;
            for (int i = size(); i > idx; i--) {
                p = p.prev;
            }
        }

        return p;
    }

    public java.util.Iterator<AnyType> iterator() {
        return new LinkedListIterator();
    }

    //图3-32  MyLinkedList类的内部Iterator类
    private class LinkedListIterator implements java.util.Iterator<AnyType> {
        private Node<AnyType> current = beginMarker.next;
        private int expectedModCount = modCount;
        private boolean okToRemove = false;

        public boolean hasNext() {
            return current != endMarker;
        }

        public AnyType next() {
            if (modCount != expectedModCount) {
                throw new java.util.ConcurrentModificationException();
            }
            if (!hasNext()) {
                throw new java.util.NoSuchElementException();
            }

            AnyType nextItem = current.data;
            current = current.next;
            okToRemove = true;
            return nextItem;
        }

        public void remove() {
            if (modCount != expectedModCount) {
                throw new java.util.ConcurrentModificationException();
            }
            if (!okToRemove) {
                throw new IllegalStateException();
            }

            MyLinkedList.this.remove(current.prev);
            expectedModCount++;
            okToRemove = false;
        }
    }

    private int theSize;
    private int modCount = 0;
    private Node<AnyType> beginMarker;
    private Node<AnyType> endMarker;

}
```



现在回到图 3-24，第 44 行到第 47 行包含 MyLinkedList 的数据成员，即到头结点和到尾节点的引用。我们也掌握一个数据成员的大小，从而 size 方法可以以常数时间实现。在第 45 行有一个附加的数据域，用来帮助检测集合中的变化。modCount 代表自从构造以来对链表所做改变的次数。每次对 add 或 remove 的调用都将更新 modCount。其想法在于，当一个迭代器被建立时，他将存储集合的 modCount。每次对一个迭代器方法（next 或 remove）的调用都将用该链表内的当前 modCount 检测在迭代器内存储的 modCount，并且当这两个计数不匹配时抛出一个 ConcurrentModificationException 异常。

MyLinkedList 类的其余部分由构造方法、迭代器的实现以及一些方法组成。许多方法都只是一行代码。

图 3-26 中的 clear 方法由构造方法调用。它创建并连接头节点和尾节点，然后设置大小为 0。

在图 3-24 的第 41 行可以看到私有内部 LinkedListIterator 类的声明的开头部分。当我们在后面看到其具体实现时将讨论这些细节。







图 3-27 解释一个包含 x 的新节点是如何被拼接在由 p 引用的一个节点和 p.prev 之间的。这些节点链的赋值可以描述如下：

```java
Node newNode = new Node(x, p.prev, p);	//第1步和第2步
p.prev.next = newNode;	//第3步
p.prev = newNode;		//第4步
```



第 3 步和第 4 步可以合并，结果只有两行：

```java
Node newNode = new Node(x, p.prev, p);	//第1步和第2步
p.prev = p.prev.next = newNode;	//第3步和第4步
```



可是这两行还可以合并，得到：

```java
p.prev = p.prev.next = new Node(x, p.prev, p);
```



这就缩短了图 3-28 中的例程 addBefore。

图 3-29 指出删除一个节点的逻辑过程。如果 p 引用正在被删除的节点，那么在该节点被断开连接和可以被虚拟机回收之前只有两个链改动：

p.prev.next = p.next;

p.next.prev = p.prev;









图 3-30 显示基本的私有 remove 例程，该例程包含上述两行代码。

图 3-31 包含前面提到的私有 getNode 方法。如果索引表示该表前半部分的一个节点，那么在第 16 行到第 18 行我们将以向后的方向遍历该链表。否则，我们从终端开始向回走，如图中第 22 行到第 24 行所示。

如图 3-32 所示，LinkedListIterator 具有类似于 ArrayListIterator 的逻辑，但合并了重要的错误检测。该迭代器保留一个当前位置，如第 3 行所示。current 表示包含由调用 next 所返回的顶的节点。注意，当 current 被定位于 endMarker 时，对 next 的调用是非法的。

为了检测在迭代期间集合被修改的情况，迭代器在第 4 行将迭代器被构造时的链表的 modCount 存储在数据域 expectedModCount 中。在第 5 行，如果 next 已经被执行而没有其后的 remove，则布尔数据域 okToRemove 为 true。因此，okToRemove 初始为 false，在 next 方法中置为 true，在 remove 方法中置为 false。

hashNext 是一个简单的例程。和在 java.util.LinkedList 的迭代器中一样，它不仅检查链表的修改。

next 方法在获得（第 17 行）将要返回（第 20 行）的节点的值后向后推进 current（第 18 行）。okToRemove 在第 19 行被更新。

最后，迭代器的 remove 方法如第 23 行至第 32 行所示。该方法主要是错误检测（这就是为什么我们避免 ArrayListIterator 中的错误间的的原因）。在第 30 行上的具体的 remove 模仿 ArrayListIterator 中的逻辑。不过在这里 current 是保持不变的，因为 current 正在观察的节点不受前面节点被删除的影响（在 ArrayListIterator 中，项被移动，要求更新 current）。



#### 3.6	栈 ADT

##### 3.6.1	栈模型

栈（stack）是限制插入和删除只能在一个位置上进行的表，该位置是表的末端，叫作栈的顶（top）。对栈的基本操作有 push（进栈）和 pop（出栈），前者相当于插入，后者则是删除最后插入的元素。最后插入的元素可以通过使用 top 例程在执行 pop 之前进行考查。对空栈进行的 pop 或 top 一般被认为是栈 ADT 中的一个错误。另一方面，当运行 push 时空间用尽是一个实现限制，但不是 ADT 错误。

栈有时又叫做 LIFO（后进先出）表。在图 3-33 中描述的模型只象征着 push 是输入操作而 pop 和 top 是输出操作。普通的清空栈的操作和判断是否空栈的测试都是栈的操作指令系统的一部分，但是，我们对栈所能够做的，基本上也就是 push 和 pop 操作。

图 3-34 表示在进行若干操作后的一个抽象的栈。一般的模型是，存在某个元素位于栈顶，而该元素是唯一的可见元素。







##### 3.6.2	栈的实现

由于栈是一个表，因此任何实现表的方法都能实现栈。显然，ArrayList 和 LinkedList 都支持栈操作；99% 的时间它们都是最合理的选择。偶尔设计一种特殊目的的实现可能会更快（例如，如果被放到栈上的项属于基本类型）。因为栈操作是常数时间操作，所以，除非在非常独特的环境下，这是不可能产生任何明显的改进的。对于这些特殊的时机，我们将给出两个流行的实现方法，一种方法使用链式结构，而另一种方法则使用数组，二者均简化了在 ArrayList 和 LinkedList 中的逻辑，因此我们不提供代码。

**栈的链表实现**

栈的第一种实现方法是使用单链表。通过在表的顶端插入来实现 push，通过删除表顶端元素实现 pop。top 操作只是考查表顶端元素并返回它的值。有时 pop 操作和 top 操作合二为一。

**栈的数组实现**

另一种实现方法避免了链而且可能是更流行的解决方案。由于模仿 ArrayList 的 add 操作，因此相应的实现方法非常简单。与每个栈相关联的操作是 theArray 和 topOfStack，对于空栈它是 -1（这就是空栈初始化的做法）。为将某个元素 x 推入栈中，我们使 topOfStack 增 1 然后置 theArray[topOfStack] = x。为了弹出栈元素，我们置返回值为 theArray[topOfStack] 然后使 topOfStack 减 1。

注意，这些操作不仅以常数时间运行，而且是以非常快的常数时间允许。在某些机器上，若在带有自增和自减寻址功能的寄存器上操作，则（整数的）push 和 pop 都可以写成一条机器指令。最现代化的计算机将栈操作作为它的指令系统的一部分，这个事实强化了这样一种观念，即栈很可能是在计算机科学中在数组之后的最基本的数据结构。



##### 3.6.3	应用

略



#### 3.7	队列 ADT

像栈一样，**队列**（queue）也是表。然而，使用队列时插入在一端进行而删除则在另一端进行。

略





### 第 4 章

对于大量的输入数据，链表的线性访问时间太慢，不宜使用。本章讨论一种简单的数据结构，其大部分操作的运行时间平均为 O(logN)。我们还要简述对这种数据结构在概念上的简单的修改，它保证了在最坏情形下上述的时间界。此外，还讨论了第二种修改，对于长的指令序列它基本上给出每种操作的 O(logN) 运行时间。

这种数据结构叫作**二叉查找树**（binary search tree）。二叉查找树是两种库集合类 TreeSet 和 TreeMap 实现的基础，它们用于许多应用之中。在计算机科学中**树**（tree）是非常有用的抽象概念，因此，我们将讨论树在其他更一般的应用中的使用。在这一章，我们将

- 看到树是如何用于实现几个流行的操作系统中的文件系统的。
- 看到树是如何能够用来计算算术表达式的值。
- 指出如何利用树支持以 O(logN) 平均时间进行的各种搜索操作，以及如何细化以得到最坏情况时间界 O(logN)。我们还将讨论当数据被存放在磁盘上时如何来实现这些操作。
- 讨论并使用 TreeSet 类和 TreeMap 类。



#### 4.1	预备知识

**树**（tree）可以用几种方式定义。定义树的一种自然的方式是递归的方式。一棵树是一些节点的集合。这个集合可以是空集；若不是空集，则树由称作**根**（root）的节点 r 以及 0 个或多个非空的（子）树 T~1~，T~2~，。。。， T~k~ 组成，这些子树中每一棵的根都来自根 r 的一条有向的**边**（edge）所连接。

每一棵子树的根叫作根 r 的儿子（child），而 r 是每一棵子树的根的父亲（parent）。图 4-1 显示用递归定义的典型的树。







从递归定义中我们发现，一棵树是 N 个节点和 N-1 条边的集合，其中的一个节点叫作根。存在 N-1 条边的结论是由下面的事实得出的：每条边都将某个节点连接到它的父亲，而除去根节点外每一个节点都有一个父亲（见图 4-2）。









在图 4-2 的树中，节点 A 是根。节点 F 有一个父亲 A 并且有儿子 K、L 和 M。每一个节点可以有任意多个儿子，也可能是零个儿子。没有儿子的节点称为**树叶**（leaf）；上图中的树叶是 B、C、H、I、P、Q、K、L、M 和 N。具有相同父亲的节点为**兄弟**（siblings）。因此，K、L 和 M 都是兄弟。用类似的方法可以定义**祖父**（grandparent）和**孙子**（grandchild）关系。

从节点 n~1~ 到 n~k~ 的**路径**（path）定义为节点 n~1~，n~2~，。。。，n~k~ 的一个序列，使得对于 1≤i＜k 节点 n~i~ 是 n~i+1~ 的父亲。这条路径的长（length）是为该路径上的边的条数，即 k-1。从每一个节点到它自己有一条长为 0 的路径。注意，在一棵树中从根到每个节点恰好存在一条路径。

对任意节点 n~i~，n~i~ 的**深度**（depth）为从根到 n~i~ 的唯一的路径的长。因此，根的深度为 0。n~i~ 的高（height）是从 n~i~ 到一片树叶的最长路径的长。因此所有的树叶的高都是 0。一棵树的高等于它的根的高。对于图 4-2 中的树，E 的深度为 1 而高为 2；F 的深度为 1 而高也是 1；该树的高为 3。一棵树的深度等于它的最深的树叶的深度；该深度总是等于这棵树的高。

如果存在从 n~1~ 到 n~2~ 的一条路径，那么 n~1~ 是 n~2~ 的一位**祖先**（ancestor）而 n~2~ 是 n~1~ 的一个**后裔**（descendant）。如果 n~1~ ≠ n~2~，那么 n~1~ 是 n~2~ 的**真祖先**（proper ancestor）而 n~2~ 是 n~1~ 的**真后裔**（proper descendant）。







































