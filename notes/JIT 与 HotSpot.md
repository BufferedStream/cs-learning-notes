## JIT 与 HotSpot

**动态编译**是某些程序语言(https://zh.wikipedia.org/wiki/程式語言)在执行时用来增进效能的方法。尽管这技术源于[Self](https://zh.wikipedia.org/wiki/Self)[[来源请求\]](https://zh.wikipedia.org/wiki/Wikipedia:列明来源)，但使用此技术最为人所知的是[Java](https://zh.wikipedia.org/wiki/Java)。此技术可以做到一些只在执行时才能完成的最佳化。使用动态编译的[执行环境](https://zh.wikipedia.org/wiki/执行环境)一开始执行速度较慢，之后，完成大部分的编译和再编译后，会执行得比非动态编译程式快很多。因为初始化时的效能延迟，动态编译不适用于一些情况。在许多实作中，一些可以在[编译时期](https://zh.wikipedia.org/wiki/編譯時期)做的最佳化被延到[执行时期](https://zh.wikipedia.org/wiki/執行時期)才编译，导致不必要的效能降低。[即时编译](https://zh.wikipedia.org/wiki/即時編譯)是一种动态编译的形式。

**即时编译**（英语：Just-in-time compilation，缩写：**JIT**），又称及时编译，实时编译，动态编译的一种形式