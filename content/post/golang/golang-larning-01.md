---
title: "Go语言从入门到放弃（一）"
date: 2018-09-09T17:21:04+08:00
draft: true
tags: ["go语言入门"]
categories: ["Golang"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

## go语言的特性
- 自带垃圾回收
- 天然支持并发


### 第一个go程序

按照国际惯例，来一个土得不能再土的 `hello world`

```go
package main

import "fmt"

func main()  {
	fmt.Print("hello world")
}

```

## go特性示例

### 协程

```go
package main

import (
	"fmt"
	"runtime"
)

func GoTest(i int) {
	fmt.Println(i)
}

func main()  {
	for i :=0 ; i < 1000; i++ {
		go GoTest(i)
	}
	fmt.Print("hello world")
	// main 函数也在一个 gorutine 中，所以main执行完成后要等待其他 gorutine 执行，否则整个进程都退出了 无法输出其他gorutine内容
	//runtime.Gosched()     //方案1，出让CPU时间
	runtime.GOMAXPROCS(2)   //方案2，设置使用2核心CPU
}
```

### channel

- 类似于unix中的pipe
- 多个goroutine通过channel通信
- 支持任何内容

```go
func main() {
	pipe := make(chan int, 3)
	pipe <- 1
	pipe <- 2
	pipe <- 3
	
	t1 := <-pipe

	fmt.Println(t1)
	pipe <- 5
	fmt.Println(len(pipe))
}
```

输出：
```output
1
3

Process finished with exit code 0

```

从管道中取出信息，在取的时候程序会阻塞

```go
var pipe chan int

func GoTest() {
	pipe <- 1
}

func main() {
	pipe = make(chan int, 1)
	go GoTest()
	t := <-pipe
	fmt.Println(t)
}

```


## 包的概念

- 和Python一样，把相同功能的代码放入到一个目录中，称之为包
- 包可以被其他包引用
- main包是用来生成可执行文件的。每个程序只有一个main包
- 包的主要用途是提高代码的可复用性