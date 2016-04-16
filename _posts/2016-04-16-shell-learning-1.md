---
layout: post
title: Shell简明教程 1
author: dyxu
keywords: 
description:
categories:
  - 技术
tags: ["shell"]

---
{% include JB/setup %}

## Shell脚本语法

Shell中一条语句一行，如果想把多条语句写在同一行，则用分号;隔开。

### 变量

Shell中的变量是“弱”变量，正常情况下，被保存字符串，若要进行数学运算需要进行转换，如$((EXPR))。变量名的格式和C语言中相同
通过`NAME=VALUE`定义变量，`unset NAME`来清除变量。Shell中提供了丰富的变量操作语法，如下图所示(图片取自
[博客](http://www.cnblogs.com/barrychiao/archive/2012/10/22/2733210.html))

![图片](/images/2016/04/shell_var_op0.png)
![图片](/images/2016/04/shell_var_op1.png)
![图片](/images/2016/04/shell_var_op2.png)

Shell还定义了一些系统环境变量和特殊的变量，环境变量用`env`指令查看，特殊变量如下表所示

|| 特殊变量 || 含义 ||
|| $num     || num=0...n，含义等同于C语言中argv[num]，表示参数表 ||
|| $#       || 等于argc - 1,表示参数个数 ||
|| $@       || 表示参数列表"$1","$2"... ||
|| $*       || 表示参数列表"$1 $2 $3 ..."（注意和$@的区别） ||
|| $?       || 上一条指令的执行状态 ||
|| $$       || 当前进程号 ||
|| $_       || 之前命令的最后一个参数 ||

### 条件测试

条件测试语句关键词是`test` 或 `[  ]`，二者等效。需要特别注意的是，如果测试为真，**返回0**;否则返回为1（这点正好和C语言中
相反）。参数表及其含义如下

|| 命令                   || 含义                            ||
|| [ -d DIR ]             || DIR存在且为一个目录则为真       ||
|| [ -f FILE ]            || FILE存在且为一个普通文件则为真  ||
|| [ -z STRING ]          || STRING长度为0则为真             ||
|| [ -n STRING ]          || STRING长度非0则为真             ||
|| [ STRING1 = STRING2 ]  || 如果两个字符串相等则为真        ||
|| [ STRING1 != STRING2 ] || 如果两个字符串不等则为真        ||
|| [ ARG1 OP ARG2 ]       || 满足OP条件则为真，其中参数的值必须是整数，OP操作可以是-eq、-ne、-lt、-le、-gt、-ge ||
|| [ ! EXPR ]             || 取非                            ||
|| [ EXPR1 -a EXPR2 ]     || 且关系                          ||
|| [ EXPR1 -o EXPR2 ]     || 或关系                          ||

例如：

    if [ -f ~/.bashrc ]; then
        source ~/.bashrc
    fi

### 分支控制

Shell中用if、then、elif、else和fi来进行分支控制，例如:

    if :; then
        echo "always true"
        echo "continue..."
    fi

其中:是一个特殊的指令，它什么也不执行，但返回为真。或者

    echo "enter a filename or dirname: "
    read STRING
    if [ -f "$STRING" ]; then
        echo "$STRING is a file"
    elif [ -d "$STRING" ]; then
        echo "$STRING is a directory"
    else
        echo "Error"
        exit 1
    fi
    exit 0

同时，Shell也支持&&和||操作，含义和C语言中相同且同样支持短路特性。与之前`test`语句中-a和-o参数的不同是后者只能用在测试
命令中。

另外，Shell还提供了case语句来进行分支控制，与C语言中switch语句不同的是，条件匹配后不需要用break跳出，而是执行完特定语句
后自动跳到esac。case语句支持通配符，含义如下:

||通配符 ||含义                  ||
||*      ||匹配任意0个或多个字符 ||
||？     ||匹配任意一个字符      ||
||[  ]  ||匹配方括号内单个字符，例如用[A-Za-z]匹配一个字母 ||

case语句的语法用一个例子阐述：

    echo "enter a single char: "
    read CHAR
    case "$CHAR" in
        [A-Za-z])
            echo "$CHAR is a letter";;
        1 | 2 | 3)
            echo "$CHAR is a specified number";;
        *)
            echo "this an else-case";;
    esac

### 循环语句

Shell中提供了三中的循环语句:while、until和for语句，其中前两个和C语言中的while和do...while类似，而for语句则更像是python中
的for语句，它们的语法分别用三个例子来阐述：

    # while statement
    CNT=0
    while [ "$CNT" -lt 10]; do
        echo "$CNT"
        CNT=$(($CNT + 1))
    done

    # until statement
    CNT=0
    until [ "$CNT" -be 10]; do
        echo "$CNT"
        CNT=$(($CNT + 1))
    done

    # for statement
    for FILE in `ls .`; do
        echo "operations on $(FILE)"
    done

### 函数

Shell中的函数不带任何参数, 通过$num来传递参数，用return提供0-255区间的返回值，在函数调用后用`$?`查看，在函数体内可以用
`local name=vale`定义局部变量，而变量的作用域和C语言中相同--局部覆盖全局，具体语法如下

    func_name ()
    {
        statement
        [return int]
    }

    num=10
    print_num() 
    {
        local num=100
        num=$((num+1))
        echo "local num=$num, paras: $@"
        return $num
    }

    print_num 1 2 3   # => local num=101, paras: 1 2 3
    echo "$?"         # => 101
    echo $num         # => 10

## Shell调试

Shell运行脚本时可以通过下列参数进行调试

|| 参数 || 作用 ||
|| -n   || 读一遍脚本中的命令但不执行，用于语法检查 ||
|| -v   || 一边执行脚本，一边将执行过的指令打印到标准错误 ||
|| -x   || 跟踪执行信息，将执行的每一条命令和结果依次打印 ||

而设置这些选项有以下三种方法

* 在命令行中提供参数 `sh -x test.sh`
* 在脚本开头提供     `#! /bin/sh -x`
* 在脚本中利用set开关参数 `set -x` 和 `set +x`



