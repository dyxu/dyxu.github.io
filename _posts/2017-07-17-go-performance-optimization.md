---
layout: post
title: Go性能优化【转载】
author: reprint
keywords: ["Go"]
description:
categories:
  - 程序设计语言
tags: ["Go", "性能优化"]

---

{% include JB/setup %}

**本文转载自[知乎专栏](https://zhuanlan.zhihu.com/p/27800985?utm_source=tuicool&utm_medium=referral)**

Go的性能优化其实总的来说和C／C++等这些都差不多，但也有它自己独有的排查方法和陷阱，这些都来源于它的语言特性和环境。

## 性能优化前提——任何好的东西都是在正确的前提上

代码界的很多事是和我们生活的哲学息息相关的，我们想要做好一件事，首先要保证我们能按时完成我们的任务，其次再去想如何把工作做的更好。如果一味只去要求做的尽善尽美可能会导致延期，失败，半途而废。

所以，先写正确的代码，再去考虑如何去让代码更快更好的运行；先完成基本的功能，再去想如何优化它。正确是优化的基础，没有这个基础，任何的优化都是毫无意义的。

## 性能优化限制——架构设计和硬件资源

良好的架构设计是我们能够发挥性能的前提，一个设计不当的架构付出再多精力优化效果也是大打折扣。这也是我们为什么经常会看到随着业务量或者用户数的增加后天架构会不断演进变化，如果说一开始设计的架构可以一直支撑下去，那么请大神请收下我的膝盖！

硬件资源更好理解，一个16核，64G内存的服务器和4核，4G内存的垃圾机器对比简直是天与地。毋庸多说。

## 什么时候做性能优化

> We should forget about small efficiencies, say about 97% of the time; premature optimization is the root of all evil(大概97％的时间，我们应该忘记小的优化， 过早优化是所有邪恶的根源). —— Donald E.Knuth

这句话不是说不去优化，不去思考算法，而是在早期我们应该更加专注于程序的实现，而不是一开始就去想着优化，你大可以放开去写。慢慢的会有驱动力让我们不自觉去优化。

正常情况下这种驱动我觉得有两种，一种是自我驱动，比如经历过搞过ACM或者算法竞赛的童鞋们在面对一个问题时会不自觉地从复杂度角度分析问题；或者一个“强迫症患者”不能忍受慢，卡，崩等等情况。

另一种是环境驱动，比如高并发环境，高精度环境，低延迟环境，大数据环境等等对于我们系统的某一方面甚至多个方面都有苛刻的要求，逼这着我们需要不断优(jia)化(ban)，优(jia)化(ban)，再优(jia)化(ban)。

当你意识到这个函数可能会被经常调用，就需要想办法的优化
当你意识到这个数据结构设计不合理导致内存占用过高，就需要想办法优化
当用户反映服务响应太慢，就需要优化
当老板既要好的服务又不想再花钱买机器，就需要优化
当代码太乱，问题百出，经常报警告打扰和女票玩耍，就需要优化

## 花多长时间来做性能优化

有人说是二八定律，又名80/20定律、帕累托法则（定律）也叫巴莱特定律、最省力的法则、不平衡原则等，被广泛应用于社会学及企业管理学等。是19世纪末20世纪初意大利经济学家巴莱多发现的。他认为，在任何一组东西中，最重要的只占其中一小部分，约20%，其余80%尽管是多数，却是次要的，因此又称二八定律。—— 百度百科

我觉得虽然我们不必一定按照二比八的要求去执行，但毫无疑问的是优化会耗费我们非常多的时间和精力，并且远远大于我们系统实现的时间，或者说自从第一次开发完，以后所有的时间都是在做优化。自己的曾经的经历，当时为了给某银行做60W终端测试优化一个API缓存系统，基本功能实现两周就完成了，后面我和性能QA童鞋一波波优化——测试——优化——测试，花费了一个多月时间做这件事，这还没完，后面在真实环境测试过程仍然暴露了很多问题，例如goruntine暴增积压，CPU暴增等等，后来发现是架构设计和组件使用上的问题，是的，当出现这样的问题时不是不可以解决，但是为了解决这样的问题会把系统搞的复杂，臃肿，虽然开发经验不多，但我觉得该是代码实现的代码实现，该是组件解决的问题就应该组件来解决，架构设计问题就是架构需要改进，不要说都可以在代码中解决，除非是不得已。

## 工欲善其事，必先利其器

首先是代码层次，好的代码是性能的关键因素，实现函数效率怎么样，排序是不是高效，操作并发性高不高等等，你可以使用代码质量评估工具来做评估，当然最好还是让有经验的司机们手把手指导。

Go代码评估工具：

1. goreporter – 生成Go代码质量评估报告
2. dingo-hunter – 用于在Go程序中找出deadlocks的静态分析器
3. flen – 在Go程序包中获取函数长度信息
4. go/ast – Package ast声明了关于Go程序包用于表示语法树的类型
5. gocyclo – 在Go源代码中测算cyclomatic函数复杂性
6. Go Meta Linter – 同时Go lint工具且工具的输出标准化
7. go vet – 检测Go源代码并报告可疑的构造
8. ineffassign – 在Go代码中检测无效赋值
9. safesql – Golang静态分析工具，防止SQL注入


然后是如何在运行过程来调试Go程序，Go自带了一个pprof工具，这个工具可以做CPU和内存的profiling。

```go
package main
import
   (
    "log"
    "net/http"
   _"net/http/pprof"
   )
func main()  {
      go func() {
        //port is you coustom define.
        log.Println(http.ListenAndServe("localhost:7000", nil))
      }()
      //do your stuff.
}
```

只需要引入net/http 和 _"net/http/pprof"即可，然后配合工具生成流程图，占比图清晰明了。

或者对于一些程序你还可以在运行时去改变它，调试它，使用google/gops 谷歌出品，你可以去查看栈，内存，CPU，heap等等信息，很不错，但是我不喜欢它开启了服务端口，这个项目刚开始是不需要使用新的端口，直接使用套接字文件通信，但是因为无法在windows上实现，最后作罢，从此好感降低了！

当然你还可以使用GDB工具，最新的GDB貌似还加入了查看goruntine的命令，很棒！

## 算法与优化思路

这个不用多说，说实话个人觉得算法是区分工程师和码农的一个很大分界点，算法可以说是基本能力，很多人不以为然觉得只是面试门槛，但是看看代码实现中数据结构的设计和算法实现就明白了！当遇到问题会不自觉的想到一个算法，这个目的就够了，其实并没有说算法非常牛逼，其实之前老司机聊天也说过只要你能在遇到问题能够想到用什么算法解决即可。

各种排序，集合操作，查询等等，没有最好的算法，只要最适合的算法。

至于哪些方面需要优化，一方面是算法的效率还要就是现象，例如CPU特别高那么看看goruntine的调度，哪个函数占用比高，是不是存在死循环；内存大，看看是不是有大的内存分配没有及时回收，或者是不是有频繁的内存分配，是不是有内存泄露？响应慢是卡在哪里，是执行效率还是和组件通信等等。



## Go的陷阱与技巧

- make的陷阱

```go
func main() {
	s := make([]int, 3)
	s = append(s, 1, 2, 3)
	fmt.Println(s)
}
// [0 0 0 1 2 3]
```

- map读写冲突，产生竞态
- 文件打开，数据库连接记得一定要关闭或释放，一般使用defer
- 对于一个struct值的map，你无法更新单个的struct值
- 简化range

```go
for range m {
}
```

- defer的陷阱

有名返回值则是在函数声明的同时就已经被声明，匿名返回值是在return执行时被声明，所以在defer语句中只能访问有名返回值，而不能直接访问匿名返回值。

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("return:", defer_call())
}

func defer_call() int {
	var i int
	defer func() {
		i++
		fmt.Println("defer1:", i)
	}()
	defer func() {
		i++
		fmt.Println("defer2:", i)
	}()
	return i
}

// defer2: 1
// defer1: 2
// return: 0
// Q2.

package main

import (
	"fmt"
)

func main() {
	fmt.Println("return:", defer_call())
}

func defer_call() (i int) {
	defer func() {
		i++
		fmt.Println("defer1:", i)
	}()
	defer func() {
		i++
		fmt.Println("defer2:", i)
	}()
	return i
}

// defer2: 1
// defer1: 2
// return: 2
```

- 短式变量声明的陷阱

那些使用过动态语言的开发者而言对于短式变量声明的语法很熟悉，所以很容易让人把它当成一个正常的分配操作。这个错误，将不会出现编译错误，但将不会达到你预期的效果。

```go
package main
import "fmt"
func main() {  
    value := 1
    fmt.Println(value)     // prints 1
    {
        fmt.Println(value) // prints 1
        value := 2
        fmt.Println(value) // prints 2
    }
    fmt.Println(value)     // prints 1 (bad if you need 2)
}
```

这个说到底是代码边界和变量影响范围问题。

- nil和显式类型

nil标志符用于表示interface、函数、maps、slices和channels的“零值”。如果你不指定变量的类型，编译器将无法编译你的代码，因为它不知道具体的类型，同时你也不能给string赋nil值。

```go
package main

func main() {
    var value1 = nil // error
    _ = value1
    var value2 string = nil // error
    if value2 == nil { // error
        value2 = "test"
    }
}
// 应该
package main

func main() {
	var value1 interface{} = nil // error
	_ = value1
	var value2 string
	if value2 == "" {
		value2 = "test"
	}
}
```

- 数组全部是值传递，没有引用传递

如果你是一个C或则C++开发者，那么知道数组就是指针。当你向函数中传递数组时，函数会参照相同的内存区域，这样它们就可以修改原始的数据。但Go中的数组是数值，因此当你向函数中传递数组时，函数会得到原始数组数据的一份复制。如果你打算更新数组的数据，你将会失败。

- select下的所有case遍历是随机的，在使用的过程中要注意，这和switch是不同的

- 使用接口实现一个类型分类函数：

```go
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
        case bool:
            fmt.Printf("param #%d is a bool\n", i)
        case float64:
            fmt.Printf("param #%d is a float64\n", i)
        case int, int64:
            fmt.Printf("param #%d is an int\n", i)
        case nil:
            fmt.Printf("param #%d is nil\n", i)
        case string:
            fmt.Printf("param #%d is a string\n", i)
        default:
            fmt.Printf("param #%d’s type is unknown\n", i)
        }
    }
}
```

- Map值在获取的时候是无序的，所以当我们需要有序时就需要通过字符串数组排序间接得到

```go
package main

import (
    "fmt"
    "sort"
)

func main() {
    var m = map[string]int{
        "unix":         0,
        "python":       1,
        "go":           2,
        "javascript":   3,
        "testing":      4,
        "philosophy":   5,
        "startups":     6,
        "productivity": 7,
        "hn":           8,
        "reddit":       9,
        "C++":          10,
    }
    var keys []string
    for k := range m {
        keys = append(keys, k)
    }
    sort.Strings(keys)
    for _, k := range keys {
        fmt.Println("Key:", k, "Value:", m[k])
    }
}
```

- init函数

开发过程我们经常会遇到主要逻辑开始前要声明或者一些全局的变量或者初始化操作，或者有时候我们仅仅需要import一些包，并不需要使用里面的函数，那就需要使用init初始化函数，一个package中可以有多个init，比如你在demo/A.go，demo/B.go都有一个init那么它们都会执行。

- Go程序显示占用内存有时候并不是真正在用的内存，只是还没还给操作系统

- [雨痕学堂](https://segmentfault.com/blog/qyuhen)

- [Go的五十度灰：Golang新开发者要注意的陷阱和常见错误](http://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/)
