---
layout: post
title: Go语言简明教程
author: dyxu
keywords: ["GO"]
description:
categories:
  - 程序设计语言
tags: ["GO", "程序设计语言"]

---

{% include JB/setup %}

本文将对Go语言的语法和使用技巧做初步介绍，权威参考[官方网站](https://golang.org/doc/)

## 1 基础知识

### 1.1 基本要素
Go程序是由一个个包组合而成的，每个Go文件有且仅属于一个包，一个包中可以包含多个文件。源文件中必须在非注释的第一行中指定所属包的名称`package xxx`。像多数语言一样，Go程序的入口为main包，例如：

```go
package main

import (
    "fmt"
)

func main() {
	fmt.Println("Hello World")
}
// Hello World
```

### 1.2 函数

Go中函数定义的语法如下

```go
func funcName(arg1 int, arg2 int) int {

}
```
当函数的连续多个参数类型相同时，可以合并 `arg1 int, arg2 int` 成 `arg1, arg2 int`。
函数可以返回任意多个返回值，用逗号分割。如`return 1, 2` 此时函数的返回值定义为`func funcName(x, y int) (int, int)`。
函数返回值在申明时可以被命名，例如`func funcName() (ret int)`。
当然，当函数没有返回值时，直接`return`。
函数变长参数的语法为`func funcName(prefix string, others ...string)`，此时传入的others为一个`[]string`类型。

```go
package main

import "fmt"

func sum(args ...int) (res int, err error) {
	for _, x := range args {
		res += x
	}

	return res, err
}

func main() {
	res, _ := sum(1, 2, 3)
	fmt.Printlf("sum = %d\n", res)
}
// sum = 6
```

### 1.3 变量

Go中使用`var`定义变量列表，语法如下`var varName varType`，多个类型间可以用逗号分割，例如： `var x, y int`。需要注意的时变量是定义在包或者函数级别的。
变量定义时可以初始化，支持类型推导，此时类型信息可以省略，例如：`var x, y = 1, 2`。
**在函数中**变量定义甚至可以省略`var`关键字，采用`:=`形式，例如：

```go
func funcName() {
    intType := 0
    floatType := 1.00
    stringType := "Hello World"
}
```

Go中基本类型大致有7种，分别是

 1. bool
 2. int int8 int16 int32 int64 uint8 uint16 uint32 uint64 uintptr
 3. float32 float64
 4. string
 5. byte (= uint8)
 6. rune (= int32 表示一个Unicode)
 7. complex64 complex128 表示复数

其中int、uint、uintptr在32位系统中为32位，在64位系统中为64位。变量在定义时，未明确初始化话时，自动赋值为**零值**。

 - 数值类型： 0
 - 布尔类型：false
 - 字符串： ""

与C、C++中不同，不同类型变量间赋值需要显示类型转换，语法为`T(v)`，例如：

```go
var x = 1
var y = float32(x)
var z = string(x)
``