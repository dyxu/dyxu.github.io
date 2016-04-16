---
layout: post
title: 简明shell教程
author: dyxu
keywords: 
description:
categories:
  - 技术
tags: ["shell"]

---
{% include JB/setup %}

## Shell配置

### Bash Shell启动配置文件

用户在登录shell时，会执行相应的配置文件，对shell进行配置和定制。主要分为三种类型

1. 用户登录主机时，loginshell先执行/etc/profile，接着依次检查用户主目录是否有.bash\_profile 或 .bash\_login 或 .profile
文件，若存在，则执行且只执行其中的一个。
2. 在登录后执行shell时，分为两种情况：
    1. 执行交付式的shell。例如打开终端，此时bash会以此执行/etc/bash.bashrc和主目录下.bashrc文件。
    2. 执行shell脚本。例如sh test.sh，此时bash会检测BASH_ENV变量的内容，如果该变量有定义，就执行所提供的配置文件的内容。

最后需要注意的是：shell注销后，其主目录下.bash_logout文件会被执行，类似析构函数的作用。

### 管理Shell配置文件

对于管理员，可以通过修改以下三个文件或目录统一管理shell的配置

1. /etc/profile 只要用户登录，都要执行
2. /etc/bash.bashrc 所有用户启动交互式shell时都要执行
3. /etc/skel/ 该目录中存放着.bash\_logout、.bashrc和.profile文件，当新用户被创建时，这三个文件就会被拷贝到其主目录下作为
默认配置文件

## Shell脚本语法

Shell中一条语句一行，如果想把多条语句写在同一行，则用分号;隔开。

### 条件测试

条件测试语句关键词是`test` 或 `[  ]`，二者等效。需要特别注意的是，如果测试为真，**返回0**;否则返回为1（这点正好和C语言中相
反）。参数表及其含义如下

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








