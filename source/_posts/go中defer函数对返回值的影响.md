---
title: go中defer函数对返回值的影响
date: 2018-02-14 15:07:53
tags: [go]
categories: [go]
---

# <center>go中defer函数对返回值的影响</center>  

+ 具体详见[github](https://github.com/ruiyr/go_learn)

> defer函数本是用于关闭连接，从panic中recover等，也会对函数返回值有影响  
 defer总在最后执行，而且是先定义后执行（栈），但会在return之前进行
  
+ defer函数又会对返回值有何影响呢？(主要发生在匿名函数中) 
 
<!-- more -->


```  
    package main
    
    import (
    	"fmt"
    )
    
    func main() {
    	a := t_defer(5)
    	fmt.Println(a)
    }
    
    func t_defer(x int) (xx int) {
    	defer func() int {
    		xx += 1
    		return xx
    	}()
    	return x
    }

```  




+ 以上代码执行返回6 （即 5 + 1 ） why?
+ **最主要的一点：ruturn 并不是原子操作**
+ 通过go tool compile -S defer_t.go>>defer.s得到汇编代码,以下为跳转部分  


```  
    .t_defer STEXT size=135 args=0x10 locals=0x28
    	0x0000 00000 (defer_t.go:12)	TEXT	"".t_defer(SB), $40-16
    	0x0000 00000 (defer_t.go:12)	MOVQ	TLS, CX
    	0x0009 00009 (defer_t.go:12)	MOVQ	(CX)(TLS*2), CX
    	0x0010 00016 (defer_t.go:12)	CMPQ	SP, 16(CX)
    	0x0014 00020 (defer_t.go:12)	JLS	125
    	0x0016 00022 (defer_t.go:12)	SUBQ	$40, SP
    	0x001a 00026 (defer_t.go:12)	MOVQ	BP, 32(SP)
    	0x001f 00031 (defer_t.go:12)	LEAQ	32(SP), BP
    	0x0024 00036 (defer_t.go:12)	FUNCDATA	$0, gclocals·f207267fbf96a0178e8758c6e3e0ce28(SB)
    	0x0024 00036 (defer_t.go:12)	FUNCDATA	$1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
    	0x0024 00036 (defer_t.go:12)	MOVQ	$0, "".xx+56(SP)
    	0x002d 00045 (defer_t.go:12)	LEAQ	"".xx+56(SP), AX
    	0x0032 00050 (defer_t.go:16)	MOVQ	AX, 16(SP)
    	0x0037 00055 (defer_t.go:13)	MOVL	$16, (SP)
    	0x003e 00062 (defer_t.go:13)	LEAQ	"".t_defer.func1·f(SB), AX
    	0x0045 00069 (defer_t.go:13)	MOVQ	AX, 8(SP)
    	0x004a 00074 (defer_t.go:13)	PCDATA	$0, $0
    	0x004a 00074 (defer_t.go:13)	CALL	runtime.deferproc(SB)
    	0x004f 00079 (defer_t.go:13)	TESTL	AX, AX
    	0x0051 00081 (defer_t.go:13)	JNE	109
    	0x0053 00083 (defer_t.go:17)	MOVQ	"".x+48(SP), AX
    	0x0058 00088 (defer_t.go:17)	MOVQ	AX, "".xx+56(SP)
    	0x005d 00093 (defer_t.go:17)	PCDATA	$0, $0
    	0x005d 00093 (defer_t.go:17)	XCHGL	AX, AX
    	0x005e 00094 (defer_t.go:17)	CALL	runtime.deferreturn(SB)
    	0x0063 00099 (defer_t.go:17)	MOVQ	32(SP), BP
    	0x0068 00104 (defer_t.go:17)	ADDQ	$40, SP
    	0x006c 00108 (defer_t.go:17)	RET
    	0x006d 00109 (defer_t.go:13)	PCDATA	$0, $0
    	0x006d 00109 (defer_t.go:13)	XCHGL	AX, AX
    	0x006e 00110 (defer_t.go:13)	CALL	runtime.deferreturn(SB)
    	0x0073 00115 (defer_t.go:13)	MOVQ	32(SP), BP
    	0x0078 00120 (defer_t.go:13)	ADDQ	$40, SP
    	0x007c 00124 (defer_t.go:13)	RET
    	0x007d 00125 (defer_t.go:13)	NOP
    	0x007d 00125 (defer_t.go:12)	PCDATA	$0, $-1
    	0x007d 00125 (defer_t.go:12)	CALL	runtime.morestack_noctxt(SB)
    	0x0082 00130 (defer_t.go:12)	JMP	0
```  


+ 可以看到return x(第17行)完后并没有立即返回（JMP），而是转去13行（defer定义处）
+ 可以简单理解为：


>   return x可分解为以下两步：  


```  
    	xx=x  
    	return xx  
```  

> 	而defer将在两者之间执行  

```  
   	xx=x  
    xx = xx + 1  
    return xx  
```   

+ 而以下代码返回值为5  

> 仅在defer定义处指定返回(xx int) ，而上面返回值只定义int，并没有指定xx

```  
    package main
    
    import (
    	"fmt"
    )
    
    func main() {
    	a := t_defer(5)
    	fmt.Println(a)
    }
    
    func t_defer(x int) (xx int) {
    	defer func() (xx int) {
    		xx += 1
    		return xx
    	}()
    	return x
    }

```   

+ defer函数还是不要乱用

+ 参考自：https://studygolang.com/articles/742