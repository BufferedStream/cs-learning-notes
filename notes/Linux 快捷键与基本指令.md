## Linux 快捷键与基本指令

- **Tab**：命令和文件名补全

- `Ctrl + C`：该键是 Linux 下面默认的中断键（Interrupt Key），当键入`Ctrl + C`时，系统会发送一个中断信号给正在运行的程序和 shell。具体的响应结果会根据程序的不同而不同。一些程序在收到这个信号后，会立即结束并退出程序，一些程序可能会忽略这个中断信号，还有一些程序在接受到这个信号后，会采取一些其他的动作（Action）。当 shell 接受到这个中断信号的时候，它会返回到提示界面，并等待下一个命令。

- `Ctr + Z`：该键是 Linux 下面默认的挂起键（Suspend Key），当键入`Ctr + Z`时，系统会将正在运行的程序挂起，然后放到后台，同时给出用户相关的 job 信息。此时，程序并没有真正的停止，用户可以通过使用`fg`、`bg`命令将 job 恢复到暂停前的上下文环境，并继续执行。一个比较常用的功能：正在使用 vi 编辑一个文件时，需要执行 shell 命令查询一些需要的信息，可以使用`Ctr + Z`挂起 vi，等指向完 shell 命令后再使用`fg`恢复 vi 继续编辑你的文件。当然，也可以在 vi 中使用`!command`方式执行 shell 命令，但是没有该方法方便。

- `Ctrl + D`：该键是 Linux 下面标准输入输出的`EOF`。在使用标准输入输出的设备中，遇到该符号，会认为读到了文件的末尾，因此结束输入或输出。。

- **help**：用于显示 shell 内部命令的帮助信息。 help 命令只能显示 shell 内部的命令帮助信息。而对于外部命令的帮助信息只能使用 man 或 info 命令查看。

  

![Aaron Swartz](https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/linux%E5%BF%AB%E6%8D%B7%E9%94%AE%E4%B8%8E%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A41.png)



![Aaron Swartz](https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/linux快捷键与常用指令2.png)



Linux 命令有内部命令（内建命令）和外部命令之分，内部命令和外部命令功能基本相同，但也有些细微差别。

内部命令实际上是 shell 程序的一部分，其中包含的是一些比较简单的 Linux 系统命令，这些命令由 shell 程序识别并在 shell 程序内部完成运行，通常在 Linux 系统加载运行时 shell 就被加载并驻留在系统内存中。内部命令是写在 bash 源码里面的，其执行速度比外部命令快，因为解析内部命令 shell 不需要创建子进程。比如：`exit，history，cd，echo`等。

外部命令是 Linux 系统中的实用程序部分，因为实用程序的功能通常都比较强大，所以其包含的程序量也会很大，在系统加载时并不随系统一起被加载到内存中，而是在需要时才将其调入内存。通常外部命令的实体并不包含在 shell 中，但是其命令执行过程是由 shell 程序控制的。 shell 程序管理外部命令执行的路径查找、加载存放，并控制命令的执行。外部命令是在bash之外额外安装的，通常放在`/bin，/usr/bin，/sbin，/usr/sbin......`等等。可通过 “echo $PATH” 命令查看外部命令的存储路径，比如：ls、vi等。

可以通过 type 命令分辨内部命令和外部命令：



![Aaron Swartz](https://raw.githubusercontent.com/BufferedStream/cs-learning-notes/master/notes/images/linux%E5%BF%AB%E6%8D%B7%E9%94%AE%E4%B8%8E%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A43.png)



- **man**： man 是 manual 的缩写，将指令的具体信息显示出来。当执行 man date 时，有 DATE(1) 出现，其中的数字代表指令的类型，常用的数字及其类型如下：

  | 代号 | 类型                                            |
  | :--: | ----------------------------------------------- |
  |  1   | 用户在 shell 环境中可以操作的指令或者可执行文件 |
  |  5   | 配置文件                                        |
  |  8   | 系统管理员可以使用的管理指令                    |



- **info**： info 与 man 类似，但是 info 将文档分成一个个页面，每个页面可以进行跳转。

- **ip addr show**：显示 ip 信息

- **uname**：

  用法：uname [选项]...
  输出一组系统信息。如果不跟随选项，则视为只附加 -s 选项。

    -a, --all                     以如下次序输出所有信息。其中若-p 和 -i 的探测结果不可知则被省略：
    -s, --kernel-name             输出内核名称
    -n, --nodename                输出网络节点上的主机名
    -r, --kernel-release          输出内核发行号
    -v, --kernel-version          输出内核版本
    -m, --machine         输出主机的硬件架构名称
    -p, --processor               输出处理器类型或"unknown"
    -i, --hardware-platform       输出硬件平台或"unknown"
    -o, --operating-system        输出操作系统名称
        --help            显示此帮助信息并退出
        --version         显示版本信息并退出



- **lsb_release**：lsb_release 命令用来查看当前系统的发行版信息（prints certain LSB (Linux Standard Base) and Distribution information.）。有了这个命令就可以清楚的知道到底是 RedHat 的、还是别的发行版，还有具体的版本号，比如 3.4 还是 5.4 等等。有些系统上不一定安装了这个命令，可以通过查看 /etc/issue 文件得到发行版信息。**也可以通过 cat /etc/issue 查看系统版本信息。**

  -v, --version

  ​          Display the version of the LSB specification against which the distribution is compliant.

  ​		  显示版本信息。

     -i, --id

  ​          Display the string id of the distributor.

  ​		  显示发行版的id。

     -d, --description

  ​          Display the single line text description of the distribution.

  ​		  显示该发行版的描述信息。

     -r, --release

  ​          Display the release number of the distribution.

  ​		  显示当前系统发行版的具体版本号。

     -c, --codename

  ​          Display the codename according to the distribution release.

  ​		  发行版代号。

     -a, --all

  ​          Display all of the above information.

     -s, --short

  ​          Use short output format for information requested by other options (or version if none).

     -h, --help

  ​          Display this message.



- **File**：

  描述：

  file 测试每个参数以尝试对其进行分类。按顺序执行三组测试：文件系统测试、魔数测试、语言测试。第一个成功的测试会打印文件类型。

  打印的类型通常包含以下的某个单词：

  - text（文本，文件仅包含打印字符和一些常用控制字符，并且可以在 ascii 终端上安全读取）；

  - executable（文件包含以某种 Unix 内核或其他内核可以理解的形式编译程序的结果）；

  - data 意味着其他任何东西（数据通常是 “二进制” 或不可打印的）。例外是已知包含二进制数据的众所周知的文件格式（核心文件，tar 档案）。

    

  修改魔术文件或程序本身时，请确保保留这些关键字。用户必须看到目录中的所有可读文件都有

   “text” 标识。不要像伯克利那样做，把 “shell commands test” 改成 “shell script”。

  **文件系统测试**：基于检查 stat(2) 系统调用的返回。程序检查文件是否为空，或者它是否是某种特殊文件。如果在系统头文件 <sys/stat.h> 中定义了文件类型，则凭直觉判断适合于当前运行的系统的任何已知文件类型（在那些实现套接字、符号链接、命名管道（FIFO）的系统上）。

  **魔数测试**：用于检查具有特定固定格式的数据的文件。这个规范的例子是二进制可执行文件（编译程序）a.out 文件，其格式在标准 include 目录中的 <elf.h>，<a.out.h>、<exec.h> 中定义。这些文件有一个“魔数”存储在文件开头附近的特定位置，用于告诉 UNIX 操作系统该文件是二进制可执行文件，以及其中的几种类型。“魔数”的概念已通过扩展应用于数据文件。在文件的小固定偏移处具有一些不变标识符，可以用这种方式来描述任何文件。标识这些文件的信息从 /etc/magic 和编译的魔术文件 /usr/share/misc/magic.mgc 中获取，或如果编译的文件不存在，则从目录 /usr/share/misc/magic 中的文件中读取。此外，如果存在 $HOME/.magic.mgc 或 $HOME/.magic，它将优先于系统魔术文件使用。

  **语言测试**：如果文件与魔术文件中的任何条目都不匹配，则会检查它是否看起来像是文本文件。通过构成每个集合中的可打印文本的不同范围和字节序列，以此来区分 ASCII，ISO-8859-x，非 ISO 8 位扩展 ASCII 字符集（例如 Macintosh 和 IBM PC 系统上使用的字符集），UTF-8 编码的 Unicode，UTF-16 编码的 Unicode， EBCDIC 字符集。如果文件通过任何这些测试，则报告其字符集。ASCII，ISO-8859-x，UTF-8，扩展 ASCII 文件被标识为 “text”，因为它们几乎可以在任何终端上读取；UTF-16 和 EBCDIC 只是 “character data”，因为虽然它们包含文本，但是它的文本在可以读取之前需要翻译。此外，file 将尝试确定文本类型文件的其他特征。如果文件的行由 CR，CRLF，NEL 终止，而不是 Unix 标准的 LF，则会进行报告。还将识别包含嵌入式转义序列或加粗的文件。

  一旦文件确定了文本类型文件中使用的字符集，它将尝试确定文件以何种语言编写。语言测试查找特定字符串（参见 <names.h>），它们可以出现在文件的前几个块中的任何位置。例如，关键字 .br 表示该文件很可能是 troff(1) 输入文件，就像关键字 struct 表示 C 程序一样。这些测试不如前两组可靠，因此它们最后执行。语言测试例程还测试一些杂项（例如 tar(1) 档案）。

  任何无法识别为已经使用上面列出的任何字符集编写的文件都被简单地称为 “data”。