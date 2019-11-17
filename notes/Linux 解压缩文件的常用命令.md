## Linux 操作文件的常用命令

- **tar**：建立新的目录
- **jar**：解压和打包 jar 文件



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