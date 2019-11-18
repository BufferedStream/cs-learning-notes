## Linux 操作文件的常用命令

- **tar**：归档使用工具，用来打包和备份
- **jar**：解压缩和打包 jar 文件



### tar命令

tar（Tapre ARchive，磁带归档的缩写，最初设计用于将文件打包到磁带上，现在我们大都使用它来实现备份某个分区或者某些重要的目录）是类 Unix 系统中使用最广泛的命令。

tar 命令可以为 linux 的文件和目录创建档案。利用 tar，可以为某一特定文件创建档案（备份文件），也可以在档案中改变文件，或者向档案中加入新的文件。tar 最初被用来在磁带上创建档案，现在，用户可以在任何设备上创建档案。利用 tar 命令，可以把一大堆的文件和目录全部打包成一个文件，这对于备份文件或将几个文件组合成为一个文件以便于网络传输是非常有用的。

首先要弄清两个概念：打包和压缩。打包是指将一大堆文件或目录变成一个总的文件；压缩则是将一个大的文件通过一些压缩算法变成一个小文件。

为什么要区分这两个概念呢？这源于 Linux 中很多压缩程序只能针对一个文件进行压缩，这样当你想要压缩一大堆文件时，你得先将这一大堆文件先打成一个包（tar 命令），然后再用压缩程序进行压缩（gzip bzip2 命令）。

linux 下最常用的打包程序就是 tar 了，使用 tar 程序打出来的包我们常称为 tar 包，tar 包文件的命令通常都是以 .tar 结尾的。生成 tar 包后，就可以用其它的程序来进行压缩。

**1．命令格式：**

tar[必要参数] [选择参数] [文件] 

**2．命令功能：**

用来压缩和解压文件。tar 本身不具有压缩功能。它是调用压缩功能实现的 

**3．命令参数：**

**必要参数有如下：**

-A 新增文件到已存在的压缩文件

-B 设置区块大小

-c 建立新的压缩文件

-d 记录文件的差别

-r 添加文件到已经压缩的文件

-u 添加改变了和现有的文件到已经存在的压缩文件

-x 从压缩的文件中提取文件

-t 显示压缩文件的内容

-z 支持gzip解压文件

-j 支持bzip2解压文件

-Z 支持compress解压文件

-v 显示操作过程

-l 文件系统边界设置

-k 保留原有文件不覆盖

-m 保留文件不被覆盖

-W 确认压缩文件的正确性



**可选参数如下：**

-b 设置区块数目

-C 切换到指定目录

-f 指定压缩文件

--help 显示帮助信息

--version 显示版本信息



**4．常见解压/压缩命令**



tar 
解包：tar xvf FileName.tar
打包：tar cvf FileName.tar DirName
（注：tar是打包，不是压缩！）


.gz
解压1：gunzip FileName.gz
解压2：gzip -d FileName.gz
压缩：gzip FileName

.tar.gz 和 .tgz
解压：tar zxvf FileName.tar.gz
压缩：tar zcvf FileName.tar.gz DirName

.bz2
解压1：bzip2 -d FileName.bz2
解压2：bunzip2 FileName.bz2
压缩： bzip2 -z FileName

.tar.bz2
解压：tar jxvf FileName.tar.bz2
压缩：tar jcvf FileName.tar.bz2 DirName

.bz
解压1：bzip2 -d FileName.bz
解压2：bunzip2 FileName.bz
压缩：未知

.tar.bz
解压：tar jxvf FileName.tar.bz
压缩：未知

.Z
解压：uncompress FileName.Z
压缩：compress FileName

.tar.Z
解压：tar Zxvf FileName.tar.Z
压缩：tar Zcvf FileName.tar.Z DirName

.zip
解压：unzip FileName.zip
压缩：zip FileName.zip DirName

.rar
解压：rar x FileName.rar
压缩：rar a FileName.rar DirName 







### jar 命令

JAR 包是 Java 中所特有一种压缩文档，其实大家就可以把它理解为 .zip 包。
当然也是有区别的，JAR 包中有一个 META-INF\MANIFEST.MF 文件,当你打成 JAR 包时，它会自动生成。
JAR 包是由 JDK 安装目录 \bin\jar.exe 命令生成的，当我们安装好 JDK，设置好 path 路径，就可以正常使用 jar.exe 命令，
它会用 lib\tool.jar 工具包中的类。



jar 命令参数：

jar 命令格式：jar {c t x u f }[ v m e 0 M i ] [-C 目录]文件名...

其中 {ctxu} 这四个参数必须选选其一。[v f m e 0 M i ] 是可选参数，文件名也是必须的。

-c  创建一个 jar 包
-t  显示 jar 中的内容列表
-x  解压 jar 包
-u  添加文件到 jar 包中
-f  指定 jar 包的文件名

-v  生成详细的报造，并输出至标准设备
-m  指定 manifest.mf 文件。(manifest.mf 文件中可以对 jar 包及其中的内容做一些设置)
-0  产生 jar 包时不对其中的内容进行压缩处理
-M  不产生所有文件的清单文件(Manifest.mf)。这个参数与忽略掉 -m 参数的设置
-i    为指定的 jar 文件创建索引文件
-C  表示转到相应的目录下执行 jar 命令，相当于 cd 到那个目录，然后不带 -C 执行 jar 命令





