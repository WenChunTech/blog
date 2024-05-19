# Golang内存模型


<!--more-->

# Golang内存模型

## 介绍

Golang内存模型是Golang程序员必须了解的一个重要概念，它定义了Golang程序中的内存访问规则，保证了Golang程序的正确性。Golang内存模型是基于happens-before关系的，happens-before关系是一个偏序关系，它定义了程序中事件的顺序关系。在Golang内存模型中，happens-before关系定义了内存访问的顺序关系，保证了Golang程序的正确性。

## happens-before关系

happens-before是很多语言都拥有的一个术语，定义通常是：假设A和B表示一个多线程的程序执行的两个操作。如果A happens-before B，那么A操作对内存的影响 将对执行B的线程(且执行B之前)可见。

它有几个特点：
1. A happens-before B并不意味着A在B之前发生
2. A在B之前发生并不意味着A happens-before B
3. 具有传递性：如果A happens-before B，B happens-before C，那么A happens-before C

> 注意：happens-before并不是指时序关系，并不是说A happens-before B就表示操作A在操作B之前发生，它就是一个术语，就像光年不是时间单位一样

## happens-before关系在Golang中的场景定义

1. 如果操作A和B在相同的线程中执行，并且A操作的声明在B之前，那么A `happens-before` B。（这条规则基本上大部分语言都适用）
2. 初始化：main.init `happens-before` main.main
3. goroutine：如果操作A是一个goroutine的创建操作，操作B是对应goroutine的启动操作，那么A `happens-before` B。
4. goroutine：如果操作A是一个goroutine的启动操作，操作B是对应goroutine的销毁操作，那么A `happens-before` B。
5. channel：如果 ch 是一个 buffered channel，则 ch<-val(写) `happens-before` val <- ch(读)
6. channel: 如果 ch 是一个 buffered channel 则 close(ch) `happens-before` val <- ch(读) & val == isZero(val)
7. channel: 如果 ch 是一个 unbuffered channel 则，val<-ch(读) `happens-before` ch<-val(写)
8. once: f() 在 once.Do(f)中的调用 `happens-before` once.Do(f)的返回

## 实例
  ```go
package main

var a string
var done bool

func setup() {
    a = "hello, world"
    done = true
}

func main() {
    go setup()
    for !done {
    }
    print(a)
}
  ```
<details>
  <summary>问：上面的代码会输出什么？</summary>
   1. 第一种情况：输出hello, world
   2. 第三种情况：输出空字符串
   3. 第二种情况：死循环
</details>


## 参考链接

1. https://tiancaiamao.gitbooks.io/go-internals/content/zh/10.1.html
2. https://go.cyub.vip/concurrency/memory-model/
3. https://research.swtch.com/plmm
4. https://go.dev/ref/mem
5. https://golang.design/under-the-hood/zh-cn/part1basic/ch05sync/mem/


---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/golang%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/  

