---
layout: post
title: "Shell script 学习笔记"
tagline: ""
description: ""
category: Linux
tags: [Linux, Shell, Bash]
last_updated: 
---

上一篇写 [Bash 的文章](/post/2015/09/bash-learning-notes.html) ，只是介绍了 terminal 下面的一些快捷键，并没有对 Bash 和 Shell 本身展开多少的介绍，因此正好趁此机会。

sh(或者叫做 Shell Command Language), 是由 [POSIX 标准](http://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html) 定义的一门语言。 它有很多实现（ksh88，dash...）等等， `bash` 同样可以认为是 sh 的一个实现。[^sh]

因为 `sh` 是一个规范，而不是实现，所以在大部分的 POSIX 系统上 `/bin/sh` 是一个 symlink。比如在 Debian/Ubtuntu 中， `sh` 便指向 `dash`.

`bash` 在创造初期是作为 `sh` 兼容的实现，但是随着时间发展，延伸出了很多扩展功能。而许多新功能改变了原来 POSIX shell 的设计。因此， `bash` 自身并不是正宗的POSIX shell，而是 POSIX shell 的一种方言。

`bash` 支持 `--posix` 模式，用来兼容 POSIX shell。很长一段时间来， 在 GNU/Linux 系统中 `/bin/sh` 都指向 `/bin/bash`，久而久之人们便忘记了这两者之间的区别。

一些非常著名的系统 `/bin/sh` 并不指向 `/bin/bash` 的例子（甚至在有些系统中 `/bin/bash` 甚至不存在）：

1. Debian/Ubuntu 系统中 `sh` 默认指向 `dash`
2. [BusyBox](https://en.wikipedia.org/wiki/BusyBox) 使用 `ash` 来作为 sh 的实现
3. BSDs， OpenBSD 使用 `pdksh` .

到此，基本也就解释了 sh 和 bash 的重要区别，但是就目前的学习， bash 因为实现比较流行，几乎绝大部分 shell 脚本都交由 bash 解释执行，并且 bash 相比较于 sh，功能更加强大，语法更加简洁。

[^sh]: <http://stackoverflow.com/a/5725402/1820217>

## Shell 实现

- sh sh 由Steve Bourne开发，是Bourne Shell的缩写，sh 是Unix 标准默认的shell。
- ash shell 是由 Kenneth Almquist 编写的，Linux中占用系统资源最少的一个小shell，它只包含24个内部命令。
- csh 它由以William Joy为代表的共计47位作者编成。它产生于 Unix 第六版 /bin/sh， 是 Bourne Shell 的前身。
- ksh 是 Korn shell 的缩写，由 David G. Korn 编写。该shell最大的优点是几乎和商业发行版的ksh完全兼容，这样就可以在不用花钱购买商业版本的情况下尝试商业版本的性能了。
- bash 由Brian Fox和Chet Ramey共同完成，是BourneAgain Shell的缩写。

下面的命令和使用基于 bash。

## Shell 适用

因为Shell似乎是各UNIX系统之间通用的功能，并且经过了POSIX的标准化。因此，Shell脚本只要“用心写”一次，即可应用到很多系统上。因此，之所以要使用Shell脚本是基于：

- 简单性：Shell是一个高级语言；通过它，你可以简洁地表达复杂的操作。
- 可移植性：使用POSIX所定义的功能，可以做到脚本无须修改就可在不同的系统上执行。
- 开发容易：可以在短时间内完成一个功能强大又妤用的脚本。

但是，考虑到Shell脚本的命令限制和效率问题，下列情况一般不使用Shell：

- 资源密集型的任务，尤其在需要考虑效率时（比如排序，hash等等）。
- 需要处理大任务的数学操作，尤其是浮点运算，精确运算，或者复杂的算术运算（这种情况一般使用C++或FORTRAN 来处理）。
- 有跨平台（操作系统）移植需求（一般使用C 或Java）。
- 复杂的应用，在必须使用结构化编程的时候（需要变量的类型检查，函数原型，等等）。
- 对于影响系统全局性的关键任务应用。
- 对于安全有很高要求的任务，比如你需要一个健壮的系统来防止入侵、破解、恶意破坏等等。
- 项目由连串的依赖的各个部分组成。
- 需要大规模的文件操作。
- 需要多维数组的支持。
- 需要数据结构的支持，比如链表或数等数据结构。
- 需要产生或操作图形化界面 GUI。
- 需要直接操作系统硬件。
- 需要 I/O 或 socket 接口。
- 需要使用库或者遗留下来的老代码的接口。
- 私人的、闭源的应用

如果你的应用符合上面的任意一条，那么就考虑一下更强大的语言吧——或许是Perl、Python、Ruby——或者是编译语言比如C/C++，或者是Java。

## 初识Shell

    #!/bin/bash
    echo "Hello World !"

“#!” 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行。通常这一行被称作 [Shebang](https://en.wikipedia.org/wiki/Shebang_%28Unix%29)。

	chmod +x ./test.sh  #使脚本具有执行权限
	./test.sh  #执行脚本

Linux 系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里 要用 `./test.sh` 告诉系统说，就在当前目录找。

再一个稍微复杂一点的脚本：

    echo "What is your name?"
    read PERSON
    echo "Hello, $PERSON"

## 变量
学习每一门编程语言都是从变量开始。

### 定义

变量定义时，变量名不加 $，如

    variableName="value"

变量名和等号**之间不能有空格**，变量名需遵循规则：

- 首字符必须为 [^a-zA-z]
- 中间不能有空格，可是有下划线 _
- 不能使用标点
- 不能使用 bash 关键字， help 命令可查看保留关键字

### 引用

使用定义过的变量，在变量名之前添加美元符号 ${variableName}

    yourName="einverne"
    echo $yourName
    echo ${yourName}

花括号是可选的，用来识别变量边界，建议每次都加上。

### 重新定义变量

不加美元符号，直接重新赋值即可

	yourName="einverne"
    echo "your name is ${yourName} !!"
    yourName="Ein Verne"
    echo "your name is ${yourName} !!"

### 只读变量

	yourName="einverne"
    echo "your name is ${yourName} !!"
    readonly yourName
    yourName="Ein Verne"

会发生错误，

    your name is einverne !!
    ./demo.sh: line 25: yourName: readonly variable

### 删除变量

使用 unset 命令删除变量

    unset variableName

unset **不能删除只读变量**


### 变量类型

同时存在三种变量：

1. 局部变量，字脚本或者命令中定义，仅在当前 shell 实例中有效，其他启动的程序不能访问
2. 环境变量，所有陈旭，包括 shell 启动的程序都能访问，有些程序需要依赖环境变量 PATH 来保证其正常运行
3. shell 变量

有 shell 程序设置的特殊变量

### shell 特殊变量

变量  |  含义 |
---------|---------------------|
$$        | 当前Shell进程ID ， PID |
$0        | 当前脚本的文件名 |
$n        | 传递给脚本或函数的参数， n 是数字，表示第几个参数 |
$#        | 传递给脚本或函数的参数个数 |
$*        | 传递给脚本的所有参数 |
$@		  | 传递给脚本的所有参数，被双引号包围稍有不同 |
$?		  |  上个命令的退出状态，或者函数的返回值  |


$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。

但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

测试脚本

    set -o nounset                              # Treat unset variables as an error

    echo "File name is $0"
    echo "First parameter : $1"
    echo "First parameter : $2"
    echo "This script have $# parameters"
    echo "They are $*"
    echo "They are $@"

    for var in "$*"
    do
        echo "${var}"
    done

    for var in "$@"
    do
        echo "${var}"
    done

结果：

    ./demo.sh p1 p2
    File name is ./demo.sh
    First parameter : p1
    First parameter : p2
    This script have 2 parameters
    They are p1 p2
    They are p1 p2
    p1 p2
    p1
    p2

### 转义

转义表

转义字符    |  含义 |
--------------|------------------|
\\  		|  反斜杠
\a 			| 报警
\b 			| 退格
\f			| 换页，将当前位置移到下页开头
\n			| 换行
\r 			| 回车
\t			| Tab
\v			| 垂直制表符

使用 `echo -e "value is ${variableName} \n"` 中的 `-e` 选项来进行转义，否则会原样输出。

echo 的 `-E` 禁止转义，默认不转义， `-n` 来禁止插入换行符。

### 结果暂存
将输出结果保存到变量中，在合适的地方使用

	Data=`date`
    echo "Date is ${Date}"

### 变量替换
根据变量的状态，来改变变量的值

形式       |   说明  |
-----------|---------------|
${var} 		| 变量原本值
${var:-word} 	| 如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。
${var:=word} 	| 如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。
${var:?message} | 如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。<br/>若此替换出现在Shell脚本中，那么脚本将停止运行。 
${var:+word} 	| 如果变量 var 被定义，那么返回 word，但不改变 var 的值。

## 表达式

### 基本表达式

    #!/bin/sh

    a=10
    b=20
    val=`expr $a + $b`
    echo "a + b : $val"

    val=`expr $a - $b`
    echo "a - b : $val"

    val=`expr $a \* $b`
    echo "a * b : $val"

    val=`expr $b / $a`
    echo "b / a : $val"

    val=`expr $b % $a`
    echo "b % a : $val"

    if [ $a == $b ]
    then
       echo "a is equal to b"
    fi

    if [ $a != $b ]
    then
       echo "a is not equal to b"
    fi

- 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2
- 乘号(*)前边必须加反斜杠(\)才能实现乘法运算
- 条件表达式要放在方括号之间，并且要有空格，例如 [$a==$b] 是错误的，必须写成 [ $a == $b ]

### 关系运算符
关系运算符只能用于数字，或者字符串中只能包含数字

运算符        | 说明
---------------|------------------
-eq  		| 相等返回 true
-ne 		| 不相等返回 true
-gt 		| great than 返回true
-lt 		| less than 返回 true
-ge 		| great or equal than ,返回 true
-le 		| less or equal than , 返回 true

### 布尔运算

运算符 | 说明
------|------
!   	| 非
-o 		| 或 or
-a 		| 与 and

### 字符串

    # 单引号中的任何字符都原样输出
    # 单引号中不能出现单引号
    echo '${hi}: "einverne"\t\n\r'

    # 双引号可以有变量
    # 双引号可以出现转义字符

    yourName='einverne'
    hi="hi, ${yourName} !\n"

    # 字符长度
    echo "your name length: ${#yourName}"

    echo "first two character of your name ${yourName:0:2}"

    text="0123456789-0123456789"
    echo $text
    echo '${text:position}' "${text:1}"
    echo '${text:position:length}' "${text:1:3}"
    # 从变量$string的开头, 删除最短匹配$substring的子串
    echo '${text#01}' "${text#01}"
    # 从变量$string的开头, 删除最长匹配$substring的子串
    echo '${text##01}' "${text##01}"
    # 从变量$string的结尾, 删除最短匹配$substring的子串
    echo '${text%89}' "${text%89}"
    # 从变量$string的结尾, 删除最长匹配$substring的子串
    echo '${text%%89}' "${text%%89}"

    # 使用$replacement, 来代替第一个匹配的$substring
    echo '${text/substring/replacement} ${text/01/ab}' "${text/01/ab}"
    # 使用$replacement, 代替所有匹配的$substring
    echo '${text//substring/replacement} ${text//01/ab}' "${text//01/ab}"

    # 如果$string的前缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
    echo '${text/#substring/replacement} ${text/#01/ab}' "${text/#01/ab}"

    # 如果$string的后缀匹配$substring, 那么就用$replacement来代替匹配到的$substring
    echo '${text/%substring/replacement} ${text/%01/ab}' "${text/%01/ab}"

### 数组

	array_name=(value0 value1 value2 value3)
    # 或者
	array_name=(
    value0
    value1
    value2
    value3
    )
    # 或者
    array_name[0]=value0
    array_name[1]=value1
    array_name[2]=value2
    # 读取数组
    valuen=${array_name[2]}
    # 获取所有元素
    ${array_name[*]}
    ${array_name[@]}
    # 长度
    # 取得数组元素的个数
    length=${#array_name[@]}
    # 或者
    length=${#array_name[*]}
    # 取得数组单个元素的长度
    lengthn=${#array_name[n]}

## 条件语句

### IF

    if [ expression ]
    then
       Statement(s) to be executed if expression is true
    fi


    if [ expression ]
    then
       Statement(s) to be executed if expression is true
    else
       Statement(s) to be executed if expression is not true
    fi

    # 更复杂
    if [ expression 1 ]
    then
       Statement(s) to be executed if expression 1 is true
    elif [ expression 2 ]
    then
       Statement(s) to be executed if expression 2 is true
    elif [ expression 3 ]
    then
       Statement(s) to be executed if expression 3 is true
    else
       Statement(s) to be executed if no expression is true
    fi

### CASE

    echo 'Input a number between 1 to 4'
    echo 'Your number is:\c'
    read aNum
    case $aNum in
        1)  echo 'You select 1'
        ;;
        2)  echo 'You select 2'
        ;;
        3)  echo 'You select 3'
        ;;
        4)  echo 'You select 4'
        ;;
        *)  echo 'You do not select a number between 1 to 4'
        ;;
    esac


## 循环语句

    for 变量 in 列表
    do
        command1
        command2
        ...
        commandN
    done


    while command
    do
       Statement(s) to be executed if command is true
    done

## 函数

    function function_name () {
        list of commands
        [ return value ]
    }

## reference

- <http://c.biancheng.net/cpp/shell/>



