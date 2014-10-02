---
layout: post
title: "使用ReactiveCocoa"
date: 2014-09-15 22:21:07 +0800
comments: true
categories: iOS
keywords: iOS, ReactiveCocoa, MVVM
description: 
---
本篇的文章的目的很明确，就是要会使用RAC。这里对RAC的关键概念和行为做出较为清晰的说明，便于使用。

ReactiveCocoa衍生自FRP（响应链编程）的一种，它是用OC语言来描述FRP的一个框架。其根源可以追述到如下论坛：<a href= http://www.haskell.org/haskellwiki/Haskell>Haskell</a>  

要会使用RAC，首先要了解FRP，了解了响应链编程之后，才可以顺利的使用RAC。PS：其实回过头来思考，RAC使用的语言和方法并不重要，重要的是这些概念。如果后续要转入Swift的FRP，也是一样。  

核心概念：  
 
  * 信号(Signal)：信号是RAC的核心。我的理解是，在RAC的编码中，对于数据的反射式响应传输，都是以信号为基础来实现的。没有它，就没有响应式自动化。

  * 订阅者 (Subscriber)：订阅者是使信号有效的一个重要角色。在FRP中，一个信号创建之后，是没有意义的，此时它不知道给谁传送数据，此时它是Cold的。而只有当他被一个或多个Subscriber订阅之后，信号接到事件源之后，就会触发响应，并发送数据给订阅者。
  
通常的使用做法是：  
a、使用一个既有信号(RAC已经给你wrapper好的Category)，然后使用匿名subscriber去做订阅行为，获取到next送来的值，或者error、completed的block，来做相应的业务逻辑操作。  
b、如果自己要自定义信号，则需要通过create信号的方式，建立一个信号，在这个信号中执行数据获取、异步计算等操作，并在信号中发送sendNext、sendCompleted或sendError等数据或行为。注意，在sendNext中发送的数据，就是subscriber接到的数据。  

另外，对于FRP而言，最为重要的就是信号和信号之间的关系：filter、flattenMap、CombineLatest、takeUntil、then等处理。  

我们可以看到，RAC可以在不影响原有业务逻辑的情况下，新增新的更复杂的业务逻辑。象积木一样不断积累和可扩展。而完全不必要地新增状态变量，让原有的代码发生更改。

文中具体的使用可以见UseRAC中的代码。

文中的概念参考文章有：  
1、<a href = http://www.infoq.com/cn/articles/functional-reactive-programming>函数式反应型编程(FRP) —— 实时互动应用开发的新思路</a>  
2、<a href = https://github.com/ReactiveCocoa/ReactiveCocoa>ReactiveCocoa in github</a>  
3、<a href = http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/>Getting Started with ReactiveCocoa</a>  
4、<a href = http://www.raywenderlich.com/74106/mvvm-tutorial-with-reactivecocoa-part-1>MVVM Tutorial with ReactiveCocoa: Part 1/2</a>  
5、<a href = http://www.raywenderlich.com/74131/mvvm-tutorial-with-reactivecocoa-part-2>MVVM Tutorial with ReactiveCocoa: Part 2/2 </a>  
6、<a href = http://limboy.me/tech/2014/06/06/deep-into-reactivecocoa2.html>ReactiveCocoa2实战</a>

