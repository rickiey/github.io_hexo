---
title: goroutine
date: 2018-02-25 19:12:17
tags: [go]
categories: [go]
---
# <center>了解goroutine</center>  

+ goroutine   
goroutine是Go并行设计的核心。goroutine说到底其实就是[协程](https://studygolang.com/articles/3098)，但是它比线程更小，十几个goroutine可能体现在底层就是五六个线程，Go语言内部帮你实现了这些goroutine之间的内存共享。执行goroutine只需极少的栈内存(大概是4~5KB)，当然会根据相应的数据伸缩。也正因为如此，可同时运行成千上万个并发任务。goroutine比thread更易用、更高效、更轻便。   
<!-- more -->    

+ goroutine是通过Go的runtime管理的一个线程管理器。goroutine通过go关键字实现了，其实就是一个普通的函数。  

+ 通过关键字go就启动了一个goroutine  

```
	package main

	import (
		"fmt"
	)

	func printgr(s string) {
			fmt.Println(s)
	}

	func main() {
		go printgr("new goroutine") //开一个新的Goroutines执行
		printgr("main goroutine") //当前Goroutines执行
	}  

```
   
+ 结果只是：main goroutine  
+ 对于上面的例子，main()函数启动了10个goroutine，然后返回，这时程序就退出了，而被启动的执行Add()的goroutine没来得及执行，再看下面例子  
  
```

	package main

	import (
		"fmt"
		"runtime"
	)

	func printgr(s string) {
		for i := 0; i < 5; i++ {
			runtime.Gosched()
			fmt.Println(s)
		}
	}

	func main() {
		go printgr("new goroutine") //开一个新的Goroutines执行
		printgr("main goroutine") //当前Goroutines执行
	}

```

结果为：  

```
	$ go run th.go
	main goroutine
	new goroutine
	main goroutine
	new goroutine
	main goroutine

```

+ 这里runtime.Gosched()表示让CPU把时间片让出来,下次某个时候继续恢复执行该goroutine  
+ 常用的runtime包中的函数：   
>   Goexit  
> 退出当前执行的goroutine，但是defer函数还会继续调用  
>  Gosched  
> 让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行  
>  NumCPU  
> 返回 CPU 核数量   
>  NumGoroutine  
> 返回正在执行和排队的任务总数   
>  GOMAXPROCS  
> 用来设置可以并行计算的CPU核数的最大值，并返回之前的值。  

-----------------------   

+ channels  
goroutine运行在相同的地址空间，因此访问共享内存必须做好同步。那么goroutine之间如何进行数据的通信呢，Go提供了一个很好的通信机制channel。channel可以与Unix shell 中的双向管道做类比：可以通过它发送或者接收值。这些值只能是特定的类型：channel类型。定义一个channel时，也需要定义发送到channel的值的类型。注意，必须使用make 创建channel：  

```
	ch:=make(chan int)
	//创建了一个int型 channel  
	ch:=make(chan int,5)
	//创建了一个int型 channel，buffer为5(int 型)

	cs := make(chan string)
	cf := make(chan interface{})

	ch <- v    // 发送v到channel ch.
	v := <-ch  // 从ch中接收数据，并赋值给v

```

+ 一个使用案例：  
  

```

	func sum(a ,b int, c chan int) {
		c <- (a+b)  // send total to c
		close(c)    //关闭c
	
	}

	func main() {
		ch:=make(chan int)
		go sum(666,888,ch)

		x:= <-ch  // c中读取数据 

		fmt.Println(x) //结果: 1554
	}

```
  

+  注意：应该在生产者的地方关闭channel，而不是消费的地方去关闭它，这样容易引起panic    
+ 默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，这样就使得Goroutines同步变的更加的简单，而不需要显式的lock。所谓阻塞，也就是如果读取（value := <-ch）它将会被阻塞，直到有数据接收。其次，任何发送（ch<-5）将会被阻塞，直到数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具    
+ 看看有缓冲的用法：   
```

	func main() {
		c := make(chan int, 2)//修改2为1就报错，修改2为3可以正常运行
		c <- 1
		c <- 2
		fmt.Println(<-c)
		fmt.Println(<-c)
	}
		
        
```

+  修改为1报如下的错误:     
> fatal error: all goroutines are asleep - deadlock!  
+ 可以通过range，像操作slice或者map一样操作缓存类型的channel   

```

	func fibonacci(n int, c chan int) {
		x, y := 1, 1
		for i := 0; i < n; i++ {
			c <- x
			x, y = y, x + y
		}
		close(c)
	}

	func main() {
		c := make(chan int, 10)
		go fibonacci(cap(c), c)
		for i := range c {
			fmt.Println(i)
		}
	}  

```

+ for i := range c能够不断的读取channel里面的数据，直到该channel被显式的关闭。上面代码我们看到可以显式的关闭channel，生产者通过内置函数close关闭channel。关闭channel之后就无法再发送任何数据了，在消费方可以通过语法v, ok := <-ch测试channel是否被关闭。如果ok返回false，那么说明channel已经没有任何数据并且已经被关闭  
+ 另外记住一点的就是channel不像文件之类的，不需要经常去关闭，只有当你确实没有任何发送数据了，或者你想显式的结束range循环之类的   

----------------------------  

+ select  
+ Go里面提供了一个关键字select，通过select可以监听channel上的数据流动。    

+ select默认是阻塞的，只有当监听的channel中有发送或接收可以进行时才会运行，当多个channel都准备好的时候，select是随机的选择一个执行的。   

```
    func fibonacci(c, quit chan int) {
    	x, y := 1, 1
    	for {
    		select {
    		case c <- x:
    			x, y = y, x + y
    		case <-quit:
    			fmt.Println("quit")
    			return
    		}
    	}
    }
    
    func main() {
    	c := make(chan int)
    	quit := make(chan int)
    	go func() {
    		for i := 0; i < 10; i++ {
    			fmt.Println(<-c)
    		}
    		quit <- 0
    	}()
    	fibonacci(c, quit)
    }

```

+ 在select里面还有default语法，select其实就是类似switch的功能，default就是当监听的channel都没有准备好的时候，默认执行的（select不再阻塞等待channel）。  

```
    select {
    case i := <-c:
    	// use i
    default:
    	// 当c阻塞的时候执行这里
    }

```

+ 超时  
+ 有时候会出现goroutine阻塞的情况，那么我们如何避免整个程序进入阻塞的情况呢？我们可以利用select来设置超时，通过如下的方式实现：   
```
    func main() {
    	c := make(chan int)
    	o := make(chan bool)
    	go func() {
    		for {
    			select {
    				case v := <- c:
    					println(v)
    				case <- time.After(5 * time.Second):
    					println("timeout")
    					o <- true
    					break
    			}
    		}
    	}()
    	<- o
    }

```

-------------------------------
主要参考自：[github.com/astaxie/build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.7.md)