## Linux 快捷键与基本指令

- Tab：命令和文件名补全

- `Ctrl + C`：该键是 Linux 下面默认的中断键（Interrupt Key），当键入`Ctrl + C`时，系统会发送一个中断信号给正在运行的程序和 shell。具体的响应结果会根据程序的不同而不同。一些程序在收到这个信号后，会立即结束并退出程序，一些程序可能会忽略这个中断信号，还有一些程序在接受到这个信号后，会采取一些其他的动作（Action）。当shell接受到这个中断信号的时候，它会返回到提示界面，并等待下一个命令。

- `Ctr + Z`：该键是 Linux 下面默认的挂起键（Suspend Key），当键入`Ctr + Z`时，系统会将正在运行的程序挂起，然后放到后台，同时给出用户相关的 job 信息。此时，程序并没有真正的停止，用户可以通过使用`fg`、`bg`命令将 job 恢复到暂停前的上下文环境，并继续执行。一个比较常用的功能：正在使用 vi 编辑一个文件时，需要执行 shell 命令查询一些需要的信息，可以使用`Ctr + Z`挂起 vi，等指向完 shell 命令后再使用`fg`恢复 vi 继续编辑你的文件。当然，也可以在 vi 中使用`!command`方式执行 shell 命令，但是没有该方法方便。

- `Ctrl + D`：该键是 Linux 下面标准输入输出的`EOF`。在使用标准输入输出的设备中，遇到该符号，会认为读到了文件的末尾，因此结束输入或输出。。

- help：用于显示 shell 内部命令的帮助信息。 help 命令只能显示 shell 内部的命令帮助信息。而对于外部命令的帮助信息只能使用 man 或 info 命令查看。

  

![Aaron Swartz](https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/linux%E5%BF%AB%E6%8D%B7%E9%94%AE%E4%B8%8E%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A41.png)



![Aaron Swartz](https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/linux快捷键与常用指令2.png)



Linux命令有内部命令（内建命令）和外部命令之分，内部命令和外部命令功能基本相同，但也有些细微差别。

内部命令实际上是 shell 程序的一部分，其中包含的是一些比较简单的 Linux 系统命令，这些命令由 shell 程序识别并在 shell 程序内部完成运行，通常在 Linux 系统加载运行时 shell 就被加载并驻留在系统内存中。内部命令是写在 bash 源码里面的，其执行速度比外部命令快，因为解析内部命令 shell 不需要创建子进程。比如：`exit，history，cd，echo`等。

外部命令是 Linux 系统中的实用程序部分，因为实用程序的功能通常都比较强大，所以其包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调入内存。通常外部命令的实体并不包含在 shell 中，但是其命令执行过程是由 shell 程序控制的。 shell 程序管理外部命令执行的路径查找、加载存放，并控制命令的执行。外部命令是在bash之外额外安装的，通常放在`/bin，/usr/bin，/sbin，/usr/sbin......`等等。可通过 “echo $PATH” 命令查看外部命令的存储路径，比如：ls、vi等。

可以通过 type 命令分辨内部命令和外部命令：



![Aaron Swartz](https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/linux%E5%BF%AB%E6%8D%B7%E9%94%AE%E4%B8%8E%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A43.png)



- man： man 是 manual 的缩写，将指令的具体信息显示出来。当执行 man date 时，有 DATE(1) 出现，其中的数字代表指令的类型，常用的数字及其类型如下：

  | 代号 | 类型                                            |
  | :--: | ----------------------------------------------- |
  |  1   | 用户在 shell 环境中可以操作的指令或者可执行文件 |
  |  5   | 配置文件                                        |
  |  8   | 系统管理员可以使用的管理指令                    |



- info： info 与 man 类似，但是 info 将文档分成一个个页面，每个页面可以进行跳转。

- ip addr show：显示 ip 信息