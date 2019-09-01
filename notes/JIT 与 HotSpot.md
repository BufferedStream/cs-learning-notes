## JIT 与 HotSpot

**动态编译**是某些程序语言在执行时用来增进效能的方法。尽管这技术源于 SELF，但使用此技术最为人所知的是 Java。此技术可以做到一些只在执行时才能完成的最佳化。使用动态编译的执行环境一开始执行速度较慢，之后，完成大部分的编译和再编译后，会执行得比非动态编译程式快很多。因为初始化时的效能延迟，动态编译不适用于一些情况。在许多实作中，一些可以在编译时期做的最佳化被延到执行时期才编译，导致不必要的效能降低。即时编译是一种动态编译的形式。

**即时编译**（英语：Just-in-time compilation，缩写：**JIT**），又称及时编译，实时编译，动态编译的一种形式，是一种提高程序运行效率的方法。通常，程序有两种运行方式：静态编译与动态解释。静态编译的程序在执行前全部被翻译为机器码，而解释执行的则是一句一句边运行边翻译。

即时编译器则混合了这两者，一句一句编译源代码，但是会将翻译过的代码缓存起来以降低性能消耗。相对于静态编译代码，即时编译的代码可以处理延迟绑定并增强安全性。

即时编译器有两种类型，一是字节码翻译，二是动态编译翻译。

微软的 .NET Framework，还有绝大多数的 Java 实现，都依赖即时编译以提供高速的代码执行。 谷歌的 V8 引擎和 Mozilla Firefox 使用的 JavaScript 引擎 SpiderMonkey 也用到了 JIT 的技术。 Ruby 的第三方实现 Rubinius 和 Python 的第三方实现 PyPy 也都通过 JIT 来明显改善了解释器的性能。



摘自知乎的 stackoverflow 回答：（已经非常通俗易懂）

This is in contrast to a traditional compiler that compiles **all** the code to machine language **before** the program is first run.

To paraphrase, conventional compilers build the whole program as an EXE file BEFORE the first time you run it. For newer style programs, an assembly is generated with pseudocode (p-code). Only AFTER you execute the program on the OS (e.g., by double-clicking on its icon) will the (JIT) compiler kick in and generate machine code (m-code) that the Intel-based processor or whatever will understand.



一个 JIT 编译器在程序启动后运行，并将代码（通常是字节码或某种 VM 指令）动态编译（或者称为及时编译）成一个普遍更快的格式，通常是主机 CPU 的源生指令集。 JIT 可以访问动态运行时的信息（而标准编译器不能做到），并且可以进行更好的优化，例如频繁使用的内联函数。

这与在程序首次运行之前将所有代码编译为机器语言的传统编译器形成了鲜明对比。

换句话说，传统编译器在第一次运行之前将整个程序构建为 EXE 文件。而 JIT 对于较新的样式程序，使用伪代码（ p 代码）生成程序集。只有在 OS 上执行程序之后（例如，通过双击其图标），（JIT）编译器才会启动并生成基于 Intel 的处理器或任何能够被识别的机器代码（ m 代码）。

Java 虚拟机的一般编译流程如下图所示：


<div align=center>
	<img src="https://github.com/BufferedStream/cs-learning-notes/blob/master/notes/images/JIT%20%E5%92%8C%20HotSpot1.jpg"/>
</div>


**HotSpot**的正式发布名称为"Java HotSpot Performance Engine"，是 Java 虚拟机的一个实现，包含了服务器版和桌面应用程序版，现时由 Oracle 维护并发布。它利用 JIT 及自适应优化技术（自动查找性能热点并进行动态优化，这也是 HotSpot 名字的由来）来提高性能。