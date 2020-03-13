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

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE13.jpg"/>
</div>

在图 3-17 中，ArrayListIterator 是泛型类，它存储当前位置，程序在 next 方法中试图使用当前位置作为下标访问数组元素然后将当前位置向后推进。注意，如果 arr 是一个数组，则 aee[idx++] 对数组使用 idx，然后向后推进 idx。图 3-17 中的问题在于，theItems[current++] 是非法的，因为 theItems 不是 ArrayListIterator 的一部分；它是 MyArrayList 的一部分。因此程序根本没有意义。

最简单的解决方案见图 3-18，不过它也有缺点，但是以更微小的方式呈现。在图 3-18 中，我们通过让迭代器存储 MyArrayList 的引用来解决在迭代器中没有数组的问题。这个引用是第二个数据域，是通过 ArrayListIterator 的一个新的单参数构造器而被初始化的。既然有一个 MyArrayList 的引用，那么就可以访问包含于 MyArrayList 中的数组域（还可得到 MyArrayList 的大小，该大小在 hasNext 中是需要的）。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE14.jpg"/>
</div>

图 3-18 中的问题在于，theItems 是 MyArrayList 中的私有（private）域，而由于 ArrayListIterator 是一个不同的类，因此在 next 方法中访问 theItems 是非法的。最简单的修正方法是改变 theItems 在 MyArrayList 中的可见性，从 private 改成某种稍宽松的可见性（如 public，或默认的可见性，它也被称为**包可见性**（package visibility））。不过，这违反了良好的面向对象编程的基本原则，它要求数据应尽可能地隐蔽。

图 3-19 显示另一种解决方案，这种方案能够正确运行：使 ArrayListIterator 为**嵌套类**（nested class）。当我们让 ArrayListIterator 为一个嵌套类时，该类将被放入另一个类（此时就是 MyArrayList）的内部，这个类就叫作**外部类**（outer class）。我们必须用 static 来表示它是嵌套的；若无 static，将得到一个内部类，这有时候好，有时候也不好。嵌套类是许多编程语言的典型的类型。注意，嵌套类可以被设计成 private，这很好，因为此时该嵌套类除能够被外部类 MyArrayList 访问外，其他是不可访问的。更为重要的是，因为嵌套类被认为是外部类的一部分，所以不存在产生不可见问题：theItems 是 MyArrayList 类的可见成员，因为 next 是 MyArrayList 的一部分。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE15.jpg"/>
</div>

既然我们有了嵌套类，那么就可以讨论内部类。嵌套类的问题在于，在我们的原始设计中，当编写 theItems 而不引用其所在的 MyArrayList 的时候，代码看起来还可以，也似乎有意义，但却是无效的，因为编译器不可能计算出哪个 MyArrayList 在被引用。要是我们自己不必明了这一点那就好多了，而这恰是内部类要求我们所要做的。

当声明一个内部类时，编译器则添加对外部类对象的一个隐式引用，该对象引起内部类对象的构造。如果外部类的名字是 Outer，则隐式引用就是 Outer.this。因此，如果 ArrayListIterator 是作为一个内部类被声明而且没有注明 static，那么 MyArrayList.this 和 theList 就都会引用同一个 MyArrayList。这样，theList 就是多余的，并可能被删除。

在每一个内部类的对象都恰好与外部类对象的一个实例相关联的情况下，内部类是有用的。在这种情况下，内部类的对象在没有外部类对象与其关联时是永远不可能存在的。对于 MyArrayList 及其迭代器的情形，图 3-20 指出了 MyArrayList 类和迭代器之间的关系，此时这些内部类都用来实现该迭代器。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE16.jpg"/>
</div>

theList.theItems 的使用可以由 MyArrayList.this.theItems 代替。这很难说是一种改进，但进一步的简化还是可能的。正如 this.data 可以简写为 data 一样（假设不存在引起冲突的也叫作 data 的另外的变量），MyArrayList.this.theItems 可以简写为 theItems。图 3-21 指出 ArrayListIterator 的简化。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE17.jpg"/>
</div>

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

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE18.jpg"/>
</div>

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

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE19.jpg"/>
</div>

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

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE20.jpg"/>
</div>

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

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE21.jpg"/>
</div>





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

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE22.jpg"/>
</div>

从递归定义中我们发现，一棵树是 N 个节点和 N-1 条边的集合，其中的一个节点叫作根。存在 N-1 条边的结论是由下面的事实得出的：每条边都将某个节点连接到它的父亲，而除去根节点外每一个节点都有一个父亲（见图 4-2）。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE23.jpg"/>
</div>

在图 4-2 的树中，节点 A 是根。节点 F 有一个父亲 A 并且有儿子 K、L 和 M。每一个节点可以有任意多个儿子，也可能是零个儿子。没有儿子的节点称为**树叶**（leaf）；上图中的树叶是 B、C、H、I、P、Q、K、L、M 和 N。具有相同父亲的节点为**兄弟**（siblings）。因此，K、L 和 M 都是兄弟。用类似的方法可以定义**祖父**（grandparent）和**孙子**（grandchild）关系。

从节点 n~1~ 到 n~k~ 的**路径**（path）定义为节点 n~1~，n~2~，。。。，n~k~ 的一个序列，使得对于 1≤i＜k 节点 n~i~ 是 n~i+1~ 的父亲。这条路径的长（length）是为该路径上的边的条数，即 k-1。从每一个节点到它自己有一条长为 0 的路径。注意，在一棵树中从根到每个节点恰好存在一条路径。

对任意节点 n~i~，n~i~ 的**深度**（depth）为从根到 n~i~ 的唯一的路径的长。因此，根的深度为 0。n~i~ 的高（height）是从 n~i~ 到一片树叶的最长路径的长。因此所有的树叶的高都是 0。一棵树的高等于它的根的高。对于图 4-2 中的树，E 的深度为 1 而高为 2；F 的深度为 1 而高也是 1；该树的高为 3。一棵树的深度等于它的最深的树叶的深度；该深度总是等于这棵树的高。

如果存在从 n~1~ 到 n~2~ 的一条路径，那么 n~1~ 是 n~2~ 的一位**祖先**（ancestor）而 n~2~ 是 n~1~ 的一个**后裔**（descendant）。如果 n~1~ ≠ n~2~，那么 n~1~ 是 n~2~ 的**真祖先**（proper ancestor）而 n~2~ 是 n~1~ 的**真后裔**（proper descendant）。



##### 4.1.1	树的实现

实现树的一种方法可以是在每一个节点除数据外还要有一些链，使得该节点的每一个儿子都有一个链指向它。然而，由于每个节点的儿子数可以变化很大并且事先不知道，因此在数据结构中建立到各（儿）子节点直接的链接是不可行的，因为这样会产生太多浪费的空间。实际上解决方法很简单：将每个节点的所有儿子都放在树节点的链表中。图 4-3 中的声明就是典型的声明。

图 4-4 指出一棵树如何用这种实现方法表示出来。图中向下的箭头是指向 firstChild （第一儿子）的链，而水平箭头是指向 nextSibling（下一兄弟）的链。因为 null 链太多了，所以没有把它们画出。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE24.jpg"/>
</div>

在图 4-4 的树中，节点 E 有一个链指向兄弟（F），另一链指向儿子（I），而有的节点这两种链都没有。



##### 4.1.2	树的遍历及应用

树有很多应用。流行的用法之一是包括 UNIX 和 DOS 在内的许多常用操作系统中的目录结构。图 4-5 是 UNIX 文件系统中一个典型的目录。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE25.jpg"/>
</div>

这个目录的根是 /usr（名字后面的星号指出 /usr 本身就是一个目录）。/usr 有三个儿子：mark、alex 和 bill，它们自己也都是目录。因此，/user 包含三个目录并且没有正规的文件。文件名 /usr/mark/book/ch1.r 先后三次通过最左边的子节点而得到。在第一个 / 后的每个 / 都表示一条边；结果为一全**路径名**（pathname）。这个分级文件系统非常流行，因为它能够使得用户逻辑地组织数据。不仅如此，在不同目录下地两个文件还可以享有相同地名字，因为它们必然有从根开始的不同的路径从而具有不同的路径名。在 UNIX 文件系统中的目录就是含有它的所有儿子的一个文件，因此，这些目录几乎是完全按照上述的类型声明构造的。（在 UNIX 文件系统中每个目录还有一项指向该目录本身以及另一项指向该目录的父目录。因此，从技术上说，UNIX 文件系统不是树，而是类树。）事实上，按照 UNIX 的某些版本，如果将打印一个文件的标准命令应用到一个目录中，那么在该目录中的这些文件名能够在（与其他非 ASCII 信息一起的）输出中被看到。

设我们想要列出目录中所有文件的名字。输出格式将是：深度为 d~i~ 的文件将被 d~i~ 次跳格（tab）缩进后打印其名。该算法在图 4-6 中以伪码给出。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE26.jpg"/>
</div>

算法的核心为递归方法 listAll。为了显示根时不进行缩进，该例程需要从深度 0 开始。这里的深度是一个内部簿记变量，而不是主调例程能够期望知道的参数。因此，驱动例程用于将递归例程和外界连接起来。

算法逻辑简单易懂。文件对象的名字和适当的跳格次数一起打印出来。如果是一个目录，那么以递归方式一个一个地处理它所有的儿子。这些儿子均处在下一层的深度上，因此需要缩进一个附加的空间。整个输出在图 4-7 中表示。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE27.jpg"/>
</div>

这种遍历策略叫作**先序遍历**（preorder traversal）。在先序遍历中，对节点的处理工作是在它的诸儿子节点被处理之前（pre）进行的。很显然，当该程序运行时，第 1 行对每个节点恰好执行一次，因为每个名字只输出一次。由于第 1 行对每个节点最多执行一次，因此第 2 行也必然对每个节点执行一次。不仅如此，对于每个节点的每一个子节点第 4 行最多只能执行一次。但是，儿子的个数恰好比节点的个数少 1。最后，第 4 行每执行一次，for 循环就迭代一次，每当循环结束时再加上一次。因此，在每个节点上总的工作量是常数。如果有 N 个文件名需要输出，则运行时间就是 O(N)。

另一种遍历树的常用方法是**后序遍历**（postorder traversal）。在后序遍历中，一个节点处的工作是在它的诸儿子节点被计算后进行的。例如，图 4-8 表示的是与前面相同的目录结构，其中圆括号内的数字代表每个文件占用的**磁盘区块**（disk blocks）的个数。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE28.jpg"/>
</div>

由于目录本身也是文件，因此它们也有大小。设我们想要计算被该树所有文件占用的磁盘区块的总数。最自然的做法是找出含于子目录 /usr/mark(30)、/user/alex(9) 和 /usr/bill(32) 的区块的个数。于是，磁盘区块的总数就是子目录中的区块的总数（71）加上 /usr 使用的一个区块，共 72 个区块。图 4-9 中的伪码方法 size 实现这种遍历策略。

如果当前对象不是目录，那么 size 只返回它所占用的区块数。否则，被该目录占用的取快数将被加到在其所有子节点（递归地）发现的区块数中去。为了区别后序遍历策略和先序遍历策略之间的不同。图 4-10 显示每个目录或文件的大小是如何由该算法产生的。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE29.jpg"/>
</div>

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE30.jpg"/>
</div>









#### 4.2	二叉树

**二叉树**（binary tree）是一棵树，其中每个节点都不能有多于两个的儿子。

图 4-11 显示一颗由一个根和两棵子树组成的二叉树，子树 T~L~ 和 T~R~ 均可能为空。

二叉树的一个性质是一颗平均二叉树的深度要比节点个数 N 小得多，这个性质有时很重要。分析表明，其平均深度为 O(根号N)，而对于特殊类型的二叉树，即**二叉查找树**（binary search tree），其深度的平均值是 O(logN)。不幸的是，正如图 4-12 中的例子所示，这个深度是可以大到 N-1 的。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE31.jpg"/>
</div>



##### 4.2.1	实现

因为一个二叉树结点最多有两个子节点，所以我们可以保存直接链接到它们的链。树节点的声明在结构上类似于双链表的声明，在声明中，节点就是由 **element**（元素）的信息加上两个到其他节点的引用（left 和 right）组成的结构（见图 4-13）。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE32.jpg"/>
</div>

我们习惯上在画链表时使用矩形框画出二叉树，但是，树一般画成圆圈并用一些直线连接起来，因为它们实际上就是图（graph）。当涉及树时，我们也不明显地画出 null 链，因为具有 N 个节点地每一棵二叉树都将需要 N+1 个 null 链。

二叉树有许多与搜索无关地重要应用。二叉树的主要用处之一是在编译器的涉及领域，我们现在就来探索这个问题。



##### 4.2.2	例子：表达式树

图 4-14 显示一个**表达式树**（expression tree）的例子。表达式树的树叶是**操作数**（operand），如常数或变量名，而其他的节点为**操作符**（operator）。由于这里所有的操作都是二元的，因此这颗特定的树正好是二叉树，虽然这是最简单的情况，但是节点还是有可能含有多于两个的儿子。一个节点也有可能只有一个儿子，如具有**一目减算符**（unaryminus operator）的情形。我们可以将通过递归计算左子树和右子树所得到的值应用在根处的运算符上而算出表达式树 T 的值。在本例中，左子树的值是 a+(b*c)，右子树的值是((d * e)+f)*g，因此整个树表示 (a+(b * c))+(((d * e)+f)*g)。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE33.jpg"/>
</div>

我们可以通过递归地产生一个带括号的左表达式，然后打印出在根处的运算符，最后再递归地产生一个带括号的右表达式而得到一个（对两个括号整体进行运算的）中缀表达式。这种一般的方法（左，节点，右）称为**中序遍历**（inorder traversal）。由于其产生的表达式类型，这种遍历很容易记忆。

另一种遍历策略是递归地打印出左子树、右子树，然后打印运算符。如果我们将这种策略应用于上面的树，则将输出 abc * + de * f + g * +，显然易见，它就是 3.6.3 节中的后缀表示法。这种遍历策略一般称为**后序遍历**。我们稍早已在 4.1 节见过这种遍历方法。



**构造表达式树**

我们现在给出一种算法来把后缀表达式转变成表达式树。由于我们已经有了将中缀表达式转变成后缀表达式的算法，因此我们能够从这两种常用类型的输入生产表达式树。这里所描述的方法酷似 3.6.3 节的后缀求值算法。我们一次一个符号地读入表达式。如果符号是操作数，那么就建立一个单节点树并将它推入栈中。如果符号是操作符，那么就从栈中弹出两棵树 T~1~ 和 T~2~（T~i~ 先弹出）并形成一棵新的树，该树的根就是操作符，它地左、右儿子分别是 T~2~ 和 T~1~。然后将这棵新树压入栈中。

例：略。





#### 4.3	查找树 ADT——二叉查找树

二叉树的一个重要的应用是它们在查找中的使用。假设树中的每个节点存储一项数据。在我们的例子中，虽然任意复杂的项在 Java 中都容易处理，但为简单起见还是假设它们是整数。还将假设所有的项都是互异的，以后再处理有重复元的情况。

使二叉树成为二叉查找树的性质是，对于树中的每个节点 X，它的左子树中所有项的值小于 X 中的项，而它的右子树中所有项的值大于 X 中的项。注意，这意味着该树所有的元素可以用某种一致的方式排序。在图 4-15 中，左边的树是二叉查找树，但右边的树则不是。右边的树在其项是 6 的节点（该节点正好是根节点）的左子树中，有一个节点的项是 7。

<div align=center>
	<img src="https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95%E5%88%86%E6%9E%90%20-%20%E5%9B%BE34.jpg"/>
</div>

现在给出通常对二叉查找树进行的操作的简要描述。注意，由于树的递归定义，通常是递归地编写这些操作的例程。因为二叉查找树的平均深度是 O(logN)，所以一般不必担心栈空间被用尽。

二叉查找树要求所有的项都能够排序。要写出一个一般的类，我们需要提供一个 interface（接口）来表示这个性质。这个接口就是 Comparable，第 1 章曾经描述过。该接口告诉我们，树中的两项总可以使用 compareTo 方法进行比较。由此，我们能够确定所有其他可能的关系。特别是我们不使用 equals 方法，而是根据两项相等当且仅当 compareTo 方法返回 0 来判断相等。另一种方法是使用一个函数对象，将在 4.3.1 节中描述。图 4-16 还指出，BinaryNode 类像链表中的节点类一样，是一个嵌套类。

图 4-17 显示 BinarySearchTree 类架构，其中唯一的数据域是对根节点的引用，这个引用对于空树来说是 null。这些 public 方法使用了调用者 private 递归方法的一般技巧。

现在描述某些私有方法。

```java
//图4-17  二叉查找树架构
public class BinarySearchTree<AnyType extends Comparable<? super AnyType>> {

    //图4-16  BinaryNode类
    private static class BinaryNode<AnyType> {

        //Constructors
        BinaryNode(AnyType theElement) {
            this(theElement, null, null);
        }

        BinaryNode(AnyType theElement, BinaryNode<AnyType> lt, BinaryNode<AnyType> rt) {
            element = theElement;
            left = lt;
            right = rt;
        }

        AnyType element;            //The data in the node
        BinaryNode<AnyType> left;   //Left child
        BinaryNode<AnyType> right;  //Right child
    }

    private BinaryNode<AnyType> root;

    public BinarySearchTree() {
        root = null;
    }

    public void makeEmpty() {
        root = null;
    }

    public boolean isEmpty() {
        return root == null;
    }

    public boolean contains(AnyType x) {
        return contains(x, root);
    }

    public AnyType findMin() {
        if(isEmpty()) {
            throw new UnderflowException();
        }
        return findMin(root).element;
    }

    public AnyType findMax() {
        if(isEmpty()) {
            throw new UnderflowException();
        }
        return findMax(root).element;
    }

    public void insert(AnyType x) {
        root = insert(x, root);
    }

    public void remove(AnyType x) {
        root = remove(x, root);
    }

    public void printTree() {

    }

    /**
     * 图4-18  二叉查找树的 contains 操作
     * Internal method to find an item in a subtree
     * @param x is item to search for
     * @param t the node that roots the subtree
     * @return true if the tiem is found; false otherwise
     */
    private boolean contains(AnyType x, BinaryNode<AnyType> t) {
        if (t == null) {
            return false;
        }

        int compareResult = x.compareTo(t.element);

        if(compareResult < 0) {
            return contains(x, t.left);
        } else if (compareResult > 0) {
            return contains(x, t.right);
        } else {
            return true;    //Match
        }
    }

    /**
     * 图4-20  对二叉查找树的 findMin 的递归实现和 findMax 的非递归实现
     * Internal method to find the smallest item in a subtree
     * @param t the node that roots the subtree
     * @return node containing the smallest item
     */
    private BinaryNode<AnyType> findMin(BinaryNode<AnyType> t) {
        if(t == null) {
            return null;
        } else if(t.left == null) {
            return t;
        }

        return findMin(t.left);
    }

    /**
     * Internal method to find the largest item in a subtree
     * @param t the node that roots the subtree
     * @return node containing the largest item
     */
    private BinaryNode<AnyType> findMax(BinaryNode<AnyType> t) {
        if(t != null) {
          while(t.right != null) {
              t = t.right;
          }
        }

        return t;
    }

    /**
     * 图4-22  将元素插入到二叉查找树的例程
     * Internal method to insert into a subtree
     * @param x the item to insert
     * @param t the node that roots the subtree
     * @return the new root of the subtree
     */
    private BinaryNode<AnyType> insert(AnyType x, BinaryNode<AnyType> t) {
        if(t == null) {
            return new BinaryNode<>(x, null, null);
        }

        int compareResult = x.compareTo(t.element);

        if(compareResult < 0) {
            t.left = insert(x, t.left);
        } else if(compareResult > 0) {
            t.right = insert(x, t.right);
        } else {
               //Duplicate; do nothing
        }

        return t;
    }

    /**
     * 图4-25  二叉查找树的删除例程
     * Internal method to remove from a subtree
     * @param x the item to remove
     * @param t the node that roots the subtree
     * @return the new root of the subtree
     */
    private BinaryNode<AnyType> remove(AnyType x, BinaryNode<AnyType> t) {
        if(t == null) {
            return t;   //Item not found; do nothing
        }

        int compareResult = x.compareTo(t.element);

        if(compareResult < 0) {
            t.left = remove(x, t.left);
        } else if (compareResult > 0) {
            t.right = remove(x, t.left);
        } else if (t.left != null && t.right != null) { //Two children
            t.element = findMin(t.right).element;
            t.right = remove(t.element, t.right);
        } else {
            t = (t.left != null) ? t.left : t.right;
        }

        return t;
    }

    private void printTree(BinaryNode<AnyType> t) {

    }
}
```



##### 4.3.1	contains 方法

如果在树 T 中存在含有项 X 的节点，那么这个操作需要返回 true，如果这样的节点不存在则返回 false。树的结构使得这种操作很简单。如果 T 是空集，那么可以就返回 false。否则，如果存储在 T 处的项是 X，那么可以返回 true。否则，我们对树 T 的左子树或右子树进行一次递归调用，这依赖于 X 与存储在 T 中的项的关系。图 4-18 中的代码就是对这种方法的一种实现。

注意测试的顺序。关键的问题是首先钥对是否空树进行测试，否则我们就会生成一个企图通过 null 引用访问数据域的 NullPointerException 异常。剩下的测试应该使得最不可能的情况安排在最后进行。还要注意，这里的两个递归调用事实上都是尾递归并且可以用一个 while 循环很容易地代替。尾递归的使用在这里是合理的，因为算法表达式的简明性是以速度降低为代价的，而这里所使用的栈空间的量也只不过是 O(logN) 而已。图 4-19 显示需要使用一个函数对象而不是要求这些项是 Comparable 的。它模仿 1.6 节的风格。



##### 4.3.2	findMin 方法和 findMax 方法

这两个 private 例程分别返回树中包含最小元和最大元的节点的引用。为执行 findMin，从根开始并且只要有左儿子就向左进行。终止点就是最小的元素。findMax 例程除分支朝向右儿子外其余过程相同。

这种递归是如此容易以至于许多程序设计员不厌其烦地使用它。我们用两种方法编写这两个例程，用递归编写 findMin 而用非递归编写 findMax（见图 4-20）。

注意，我们是如何小心地处理空树的退化情况的。虽然这样做总是重要的，但是特别在递归程序中它尤其重要。此外，还要注意，在 findMax 中对 t 的改变是安全的，因为我们只用到引用的拷贝来进行工作。不管怎么说，还是应该随时特别小心，因为诸如 t.right = t.right.right 这样的语句将会产生一些变化。



##### 4.3.3	insert 方法

进行插入操作的例程在概念上是简单的。为了将 X 插入到树 T 中，你可以像用 contains 那样沿着树查找。如果找到 X，则什么也不用做（或做一些 “更新”）。否则，将 X 插入到遍历的路径上的最后一点上。图 4-21 显示实际的插入情况。为了插入 5，我们遍历该树就好像在运行 contains。在具有关键字 4 的节点处，我们需要向右行进，但右边不存在子树，因此 5 不在这棵树上，从而这个位置就是所要插入的位置。





重复元的插入可以通过在节点记录中保留一个附加域以指示发生的频率来处理。这对整个的树增加了某些附加空间，但是，却比将重复信息放到树中要好（它将使树的深度变得很大）。当然，如果 compareTo 方法使用的关键字只是一个更大结构的一部分，那么这种方法行不通，此时我们可以把具有相同关键字的所有结构保留在一个辅助数据结构中，如表或是另一颗查找树。

图 4-22 显示插入例程的代码。由于 t 引用该树的根，而根又在第一次插入时变化，因此 insert 被写成一个返回对新树根的引用的方法。第 15 行和第 17 行递归地插入 x 到适当的子树中。



##### 4.3.4	remove 方法

正如许多数据结构一样，最困难的操作是 remove（删除）。一旦我们发现要被删除的节点，就需要考虑几种可能的情况。

如果节点是一片树叶，那么它可以被立即删除。如果节点有一个儿子，则该节点可以在其父节点调整自己的链以绕过该节点后被删除（为了清楚起见，我们将明确地画出链的指向），见图 4-23。







复杂的情况是处理具有两个儿子的节点。一般的删除策略是用其右子树的最小的数据（很容易找到）代替该节点的数据并递归地删除那个节点（现在它是空的）。因为右子树中的最小的节点不可能有左儿子，所以第二次 remove 更容易。图 4-24 显示一棵初始的树及其中一个节点被删除后的结果。要被删除的节点是根的左儿子；其关键字是 2。它被其右子树中的最小数据 3 所代替，然后关键字是 3 的原节点如前例那样被删除。

图 4-25 中的程序完成删除的工作，但它的效率并不高，因为它沿该树进行两趟搜索以查找和删除右子树中最小的节点。通过写一个特殊的 removeMin 方法可以很容易地改变这种效率不高地缺点，我们这里将它略去只是为了简明。

如果删除的次数不多，通常使用的策略是**懒惰删除**（lazy deletion）；当一个元素要被删除时，它仍留在树中，而只是被标记为删除。这特别是在有重复项时很常用，因为此时记录出现频率数的域可以减 1。如果树中的实际节点数和 “被删除” 的节点数相同，那么树的深度预计只上升一个小的常数（为什么？），因此，存在一个与懒惰删除相关的非常小的时间损耗。再有，如果被删除的项是重新插入的，那么分配一个新单元的开销就避免了。



##### 4.3.5	平均情况分析

直观上，我们期望前一节所有的操作都花费 O(logN) 时间，因为我们用常数时间在树中降低了一层，这样一来，对其进行操作的树大致减小一半左右。因此，所有操作的运行时间都是 O(d)，其中 d 是包含所访问的项的节点的深度。

证明：略

上面的讨论主要是说明，决定 “平均” 意味着什么一般是极其困难的，可能需要一些假设，这些假设可能合理，也可能不合理。不过，在没有删除或是使用懒惰删除的情况下，我们可以断言：上述那些操作的平均运行时间都是 O(logN)。除像上面讨论的一些个别情形外，这个结果与实际观察到的情形是非常一致的。

如果向一棵树输入预先排好序的数据，那么一连串 insert 操作将花费二次的时间，而链表实现的代价会非常巨大，因为此时的树将只由那些没有左儿子的节点组成。一种解决方法就是要有一个称为**平衡**（balance）的附加的结构条件：任何节点的深度均不得过深。

许多一般的算法都能实现平衡树。但是，大部分算法都要比标准的二叉查找树复杂得多，而且更新要平均花费更长得时间。不过，它们确实防止了处理起来非常麻烦得一些简单情形。下面，我们将介绍最古老的一种平衡查找树，即 AVL 树。

另外，较新的方法是放弃平衡条件，允许树有任意的深度，但是在每次操作之后要使用一个调整规则进行调整，使得后面的操作效率要高。这种类型的数据结构一般属于自调整（self-adjusting）类结构。在二叉查找树的情况下，对于任意单个操作我们不再保证 O(logN) 的时间界，但是可以证明任意**连续 M** 次操作在最坏的情形下花费时间 O(M log N)。一般这足以防止令人棘手的最坏情形。我们将要讨论的这汇总数据结构叫作**伸展树**（splay tree）；它的分析相当复杂，我们将在第 11 章讨论。



#### 4.4	AVL 树

ABL（Adelson-Velskii 和 Landis）树是**带有平衡条件**（balance condition）的二叉查找树。这个平衡条件必须要容易保持，而且它保证树的深度须是 O(logN)。最简单的想法是要求左右子树具有相同的高度。如图 4-28 所示，这种想法并不强求树的深度要浅。

略



#### 4.5	伸展树

略



#### 4.6	再探树的遍历

略



#### 4.7	B 树

迄今为止，我们始终假设可以把整个数据结构存储到计算机的内存中。可是，如果数据更多装不下主存，那么这就意味着必须把数据结构放到磁盘上。此时，因为大 O 模型不再适用，所以导致游戏规则发生了变化。

问题子啊与，大 O 分析假设所有的操作耗时都是相等的。然而，现在这种假设就不合适了，特别是涉及磁盘 I/O 的时候。例如，一台 500-MIPS 的机器可能每秒执行 5 亿条指令。这是相当快的，主要是因为速度主要依赖于电的特性。另一方面，磁盘操作是机械运动，它的速度主要依赖于转动磁盘和移动磁头的时间。许多磁盘以 7200RPM 旋转。即 1 分钟转 7200 转；因此，1 转占用 1/120 秒，或即 8.3 毫秒。平均可以认为磁盘转到一半的时候发现我们要寻找的信息，但这又被移动磁盘磁头的时间抵消，因此我们得到访问时间为 8.3 毫秒（这是非常宽松的估计；9~11 毫秒的访问时间更为普通）。因此，每秒大约可以进行 120 次磁盘访问。若不和处理器的速度比较，那么听起来还是相当不错的。可是考虑到处理器的速度，5 亿条指令却花费相对于 120 次磁盘访问的时间。换句话说，一次磁盘访问的价值大约是 40 万条指令。当然，这里每一个数据都是粗略的计算，不过相对速度还是相当清楚的：磁盘访问的代价太高了。不仅如此，处理器的速度还在以比磁盘速度快得多的速度增长（增长相当快的是磁盘容量的大小）。因此，为了节省一次磁盘访问，我们愿意进行大量的计算。几乎在所有的情况下，控制运行时间的都是磁盘访问的次数。于是，如果把磁盘访问次数减少一半，那么运行时间也将减半。

在磁盘上，典型的查找树执行如下：设想要访问佛罗里达州公民的驾驶记录。假设有 1 千万项，每一个关键字是 32 字节（代表一个名字），而一个记录是 256 个字节。假设这些数据不能都装入主存，而我们是正在使用系统的 20 个用户中的一个（因此有 1/20 的资源）。这样，在 1 秒内，我们可以执行 2500 万次指令，或者执行 6 次磁盘访问。

不平衡的二叉查找树是一个灾难。在最坏情形下它具有线性的深度，从而可能需要 1 千万次磁盘访问。平均来看，一次成功的查找可能需要 1.38logN 次磁盘访问，由于 log 10 000 000 ≈ 24。因此平均一次查找需要 32 次磁盘访问，或 5 秒的时间。在一棵典型的随机构造的树中，我们预料会有一些节点的深度要深 3 倍；它们需要大约 100 次磁盘访问，或 16 秒的时间。AVL 树多少要好一些。1.44logN 的最坏情形不可能发生，典型的情形是非常接近于 logN。这样，一棵 AVL 树平均将使用大约 25 次磁盘访问，需要的时间是 4 秒。

我们想要把磁盘访问次数减小到一个非常小的常数，比如 3 或 4；而且我们愿意写一个复杂的程序来做这件事，因为在合理情况下机器指令基本上是不占时间的。由于典型的 AVL 树接近到最优的高度，因此应该清楚的是，二叉查找树是不可行的。使用二叉查找树我们不能行进到低于 logN。解法直觉上看是简单的：如果有更多的分支，那么就有更少的高度。这样，31 个节点的理想二叉树（perfect binary tree）有 5 层，而 31 个节点的 5 叉树则只有 3 层，如图 4-59 所示。一棵 M 叉查找树（M-ary search tree）可以有 M 路分支。随着分支增加，树的深度在减少。一棵完全二叉树（complete binary tree）的高度大约为 log~2~N，而一棵完全 M 叉树（complete M-ary tree）的高度大约是 log~m~N。







我们可以以与建立二叉查找树大致相同的方式建立 M 叉查找树。在二叉查找树中，需要一个关键字来决定两个分支到底取用哪个分支；而在 M 叉查找树中需要 M1 个关键字来决定选取哪个分支。为使这种方案在最坏的情形下有效，需要保证 M 叉查找树以某种方式得到平衡。否则，像二叉查找树，它可能退化成一个链表。实际上，我们甚至想要更加限制性的平衡条件，即不想要 M 叉查找树退化到甚至是二叉查找树，因为那时我们又将无法摆脱 logN 次访问了。

实现这种想法的一种方法是使用 B 树。这里描述基本的 B 树（这里所描述的是通常称为 B^+^ 树）。许多的变种和改进都是可能的，但实现起来多少要复杂些，因为有相当多的情形需要考虑。不过，容易看到，原则上 B 树保证只有少数的磁盘访问。

阶为 M 的 B 树是一棵具有下列特性的树（法则 3 和 5 对于前 L 次插入必须要放宽）：

1.数据项存储在树叶上。

2.非叶节点存储直到 M-1 个关键字以指示搜索的方向；关键字 i 代表子树 i+1 中的最小的关键字。

3.树的根或者是一片树叶，或者其儿子数在 2 和 M 之间。

4.除根外，所有非树叶节点的儿子数在 [M/2] 和 M 之间。

5.所有的树叶都在相同的深度上并有 [L/2] 和 L 之间个数据项，L 的确定稍后描述。

图 4-60 显示 5 阶 B 树的一个例子。注意，所有的非叶节点的儿子数都在 3 和 5 之间（从而有 2 到 4 个关键字）；根可能只有两个儿子。这里，我们有 L=5（在这个例子中 L 和 M 恰好是相同的，但这不是必需的）。由于 L 是 5，因此每片树叶有 3 到 5 个数据项。要求节点半满将保证 B 树不致退化成简单的二叉树。虽然存在改变该结构的各种 B 树的定义，但大部分在一些次要的细节上变化，而我们这个定义是流行的形式之一。







每个节点代表一个磁盘区块，于是我们根据所存储的项的大小选择 M 和 L。例如，设一个区块能容纳 8192 字节。在上面的佛罗里达例子中，每个关键字使用 32 个字节。在一棵 M 阶 B 树种，有 M-1 个关键字，总数为 32M-32 字节，再加上 M 个分支。由于每个分支基本上都是另外的一些磁盘区块，因此可以假设一个分支是 4 个字节。这样，这些分支共用 4M 个字节。一个非叶节点总的内存需求为 36M-32 个字节。使得不超过 8192 字节的 M 的最大值是 228.因此，我们选择 L=32。这样就保证每片树叶有 16 到 32 个数据记录以及每个内部节点（除根外）至少以 114 种方式分叉。由于有 1 千万个记录，因此至多存在 625000 片树叶。由此得知，在最坏情形下树叶将在第 4 层上。更具体地说，最坏情形地访问次数近似地由 log~M/2~N 给出，这个数可以有 1 的误差（例如，根和下一层可以存放在主存中，使得经过长时间运行后磁盘访问将只对第 3 层或更深层是需要的）。

剩下的问题是如何向 B 树添加项和从 B 树删除项；下面将概述所涉及的想法。注意，许多论题以前见到过。





















































