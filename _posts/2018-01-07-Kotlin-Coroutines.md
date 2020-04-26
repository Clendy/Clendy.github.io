---
layout:     post
title:      Kotlin Coroutines
subtitle:   
date:       2018-01-07
author:     Clendy
catalog: true
tags:
    - Kotlin
---
<!-- MarkdownTOC -->

<!-- /MarkdownTOC -->

# Kotlin协程和作用

>一些耗时操作(网络IO、文件IO、CPU/GPU密集型任务)会阻塞线程直到操作完成，
`Kotlin`的协程提供一种避免阻塞且更廉价可控的操作: 协程挂起(coroutine suspension),
协程将复杂异步操作放入底层库中,程序逻辑可顺序表达,以此简化异步编程，
该底层库将用户代码包装为回调/订阅事件,在不同线程(甚至不同机器)调度执行!

`Kotlin`的协程还能实现其它语言的异步机制(`asynchronous mechanisms`),
例如源于C#和ECMAScript(js)的`async/await`机制，
源于Go的`channel/select`机制,源于C#和Python的`generators/yield`机制。


# 线程阻塞和协程挂起的区别(Blocking VS Suspending)
协程是通过编译技术实现(不需要虚拟机VM/操作系统OS的支持),通过插入相关代码来生效！
与之相反,线程/进程是需要虚拟机VM/操作系统OS的支持,通过调度CPU执行生效!

线程阻塞的代价昂贵,
尤其在高负载时的可用线程很少,阻塞线程会导致一些重要任务缺少可用线程而被延迟!

协程挂起几乎无代价,无需上下文切换或涉及OS,
最重要的是,协程挂起可由用户控制:可决定挂起时发生什么,并根据需求优化/记录日志/拦截!

另一个不同之处是,协程不能在随机指令中挂起,只能在挂起点挂起(调用标记函数)!

# 挂起函数(Suspending functions)
当调用[suspend修饰的函数]时会发生协程挂起：
```
    suspend fun doSomething(foo: Foo): Bar {           
    }        
```
该函数称为挂起函数,调用它们可能挂起协程(如果调用结果已经可用,协程库可决定不挂起)
挂起函数能像普通函数获取参数和返回值,但只能在协程/挂起函数中被调用!

* 启动协程,至少要有一个挂起函数,通常是匿名的(即挂起lambda表达式),
一个简化的async函数(源自kotlinx.coroutines库)：
```
    //async函数是一个普通函数(不是挂起函数)
    //block参数有suspend修饰,是一个匿名的挂起函数(即挂起lambda表达式)
    fun <T> async(block: suspend () -> T)

    async {
        //doSomething在挂起函数(挂起lambda)中被调用
        doSomething(foo)
        ...
    }

    async {
        ...
        //await()是挂起函数,该函数挂起一个协程,直到执行完成返回结果
        val result = computation.await()
        
```
* 挂起函数不能在普通函数中被调用：
```
    fun main(args: Array<String>) {
        doSomething() //错误: 挂起函数不能在非协程中被调用
    }
```
* 挂起函数可以是虚拟的,当覆盖它们时,必须指定suspend修饰符：
```
    interface Base {
        suspend fun foo()
    }

    class Derived: Base {
        override suspend fun foo() {                
        }
    }
```

# 协程内部机制原理(inner workings)
粗略地认识协程机制原理是相当重要的! 

协程是通过编译技术实现(不需要虚拟机VM/操作系统OS的支持),通过插入相关代码来生效！
与之相反,线程/进程是需要虚拟机VM/操作系统OS的支持,通过调度CPU执行生效!

基本上,每个挂起函数都转换为状态机,对应于挂起调用;
在挂起协程前,下一状态和相关局部变量等被存储在编译器生成的类字段中;
在恢复协程时,状态机从下一状态进行并恢复局部变量！

一个挂起的协程可作为保存挂起状态和局部变量的对象,对象类型是Continuation,
在底层,挂起函数有一个Continuation类型的额外参数

关于协程工作原理的更多细节可查看:
[https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)


