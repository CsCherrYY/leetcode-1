# Shell

 基于ubuntu 16.04

### 一、什么是Bash?

#### 准备工作

编辑器：[vscode](https://code.visualstudio.com/)

安装 bashdb 用于调试

apt install bashdb

安装vscode bash debug 插件

#### Linux Shell分类

Linux由内核、Shell、GUI等几个主要部分组成。Shell翻译你的指令并送给内核。大部分Linux操作系统都封装了多种Shell

* Sh shell：这被称为Bourne shell，这是由AT＆T实验室由一个名叫Stephen Bourne的人开发的。
* Bash shell：也称为Bourne again shell，它兼容sh shell脚本
* Ksh shell：也称为Korn shell，它与sh和bash兼容。 Kshoffers对Bourne shell进行了一些改进.
* Csh和tcsh：Linux是使用C语言构建的，它促使开发者在伯克利大学开发一种C风格的shell，其语法类似于C语言。 Tcsh为csh添加了一些小的增强功能

bash脚本的基本思想是通过执行多个命令来实现工作自动化。

#### Bash命令架构

**type命令**

type命令告诉我们命令的类型。类型包括以下5种：

*  Alias 别名
* Function 函数
* Shell built-in Shell内建
* Keyword 关键词
* File 文件 \(二进制可执行文件、脚本文件等\)

linux下不同命令type示例：

```bash
root@mhn20:~# type ls quote pwd do id
ls is aliased to `ls --color=auto'
quote is a function
quote () 
{ 
    local quoted=${1//\'/\'\\\'\'};
    printf "'%s'" "$quoted"
}
pwd is a shell builtin
do is a shell keyword
id is /usr/bin/id
```

**PATH 环境变量**

linux会检查存在于环境变量PATH中的可执行文件。需要将文件的绝对或者相对路径提供给PATH。

比如，将当前路径添加到环境变量中：

```bash
export PATH=$PATH:.
```

PATH中的每一项使用:进行分隔。将可执行文件/脚本放入例如$home/bin目录下，可以方便调用。

检测目录是否存在，如果不存在就创建目录

```bash
test -d $HOME/bin || mkdir $HOME/bin
```

#### 创建与执行

**Hello world!**

创建脚本$home/bin/hello1.sh

```bash
#!/bin/bash
echo "hello world!"
exit 0 
```

\#是shell中的注释符。虽然\#!/bin/bash以\#开头，但系统仍然会识别这一行。它指定了我们使用哪一种shell来执行脚本。如果不指定，那么就使用当前的shell。

echo是shell内建命令，用于将信息输出到stdout。

exit 0 ：exit也是内建命令，用于终止退出脚本，带一个整型参数表示返回值。通常非零值表示非正常退出。

**执行脚本**

```bash
bash hello1.sh
```

这样并不方便，我们想要直接能够执行的shell脚本，所以需要给脚本添加可执行权限。

```bash
chmod +x $HOME/bin/hello1.sh
root@x:~/bin# hello1.sh 
hello world!
```

如果之前没有将当前目录添加到环境变量PATH中，则需要使用./来执行

```bash
./hello1.sh 
```

**检查退出状态**

```bash
$ command1 || command 2
```

如果command1成功，command2就不会执行。如果失败，则执行。

```bash
$ command1 && command2
```

当command1成功继续执行command2

为了在shell脚本中获取前一条指令的exit值，可以使用$?变量。

```bash
root@x:~/bin# echo $?
0
```

#### 脚本参数

$0:脚本名称

$1：第一个参数

${10}：当需要两个以上数字来表示第几个参数时，需要加上{}，否则默认解析单个数字。

$\#：命令行参数个数

$\*：所有参数

#### 引号的使用

shell中有''和""两类，用于保护特殊字符，比如两个单词之间的空格，如果不用引号，则会被认为是分开的两个参数。

''不会对中间的内容进行转义，""会。

例如echo ' hello $1'，只会打印hello $1

而echo "hello $1"，会对$1进行转义，输出hello &lt;第一个参数&gt;

### 声明变量

变量有两种类型：环境变量和用户定义变量。

#### 用户定义变量

使用=给变量赋值。

```bash
#!/bin/bash
name="Mokhtar"
age=35
total=16.5
echo $name #prints Mokhtar
echo $age #prints 35
echo $total #prints 16.5
```

注意，=两侧不能有空格，否则会被当做命令进行解析。

也可以定义数组。用\(\)表示

移除数组中的值可以用unset

```bash
#!/bin/bash
myarr=(one two three four five)
echo ${myarr[1]}    #prints two which is the second element
echo ${myarr[*]}
unset myarr[1]      #This will remove the second element
unset myarr         #This will remove all elements
```

#### 命令替换

有时候，我们需要把一个命令执行的结果作为值赋给一个变量，比如用pwd获得当前路径

可以用\`\` （Esc下面的那个键）或者$\(\)来获取命令执行后的值

```bash
#!/bin/bash
cur_dir=`pwd`
echo $cur_dir
cur_dir=$(pwd)
echo $cur_dir
```

#### 调试脚本

执行bash时有-v -x两个选项用于调试。

