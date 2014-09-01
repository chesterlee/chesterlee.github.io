---
layout: post
title: "我的C++对象模型读书笔记--第四章 The Semantics of Function"
date: 2014-08-02 09:25:15 +0800
comments: true
categories: cpp
keywords: c++,cpp
description: 
---
####4.1 Member的各种调用方式

Nonstatic Member Functions：

速度和一般的Nonmember function 有相同的效率。 编译器会对nonstatic member function进行改造。加入this参数，改变函数内部的对member data 的使用方法(使用this指针来存取！)，改变函数的名字--相当于nonmember function。这里面有一个函数名字的处理问题(name mangled处理)。

Virtual Member Funciton：

首先虚函数的调用方式若是指针调用，则是通过指针，找到vptr，打出偏移找到函数地址进行调用。其次，vptr的名字也是会被“name mangled”的，因为存在较为复杂的派生体系。

在一个虚函数中调用另外一个虚函数，只需要写成类的函数(class::funcname())即可。因为在母函数中，执行期间已经决议出this指针指向的对象，因此上述做法可以压抑编译器的重复做法(使用vptr打偏移来做)。

通过一个object（对象）调用虚拟函数，这时如果需要通过vptr中转一次，就会显得较为多余，编译器在决议的时候，会直接决议成nonstatic member function。

Static Member Funciton：它的主要特性就是没有this指针，所以它不能够直接存取其class的nonstatic data members；也不能够被声明为virtual，const， volatile；它因此也就不需要经由某个class object 才能够被调用。他的调用速度和nonmember funciton差不多。如果取static member function的地址，那么将会是直接“Nonmember函数指针”，而不是“指向class member funciton的指针”。

 

####4.2 Virtual Member Function

多重继承下的Virtual Funciton :

当使用base  class pointer 调用virtual function的时候，可能出现两种情况。因为该base class pointer可能指向的位置处于derived class object的开始处，也可能有一个偏移量。一般规则时，经由指向“第二或后继之base class”的志真来调用derived class virtual funciton ,改掉用操作所连带的“必要的this指针调整”操作，必须在执行期完成。

那么怎么处理呢？ 可能本来不需要调整的也变得要调整了！ 真是讨厌！  嘿嘿！  thunk技术被引用到了编译器中！  怎么解决呢？ 

Sun编译器的方法是提供所谓的“split functions”技术；以相同的算法产生两个函数，其中第二个在返回之前，为指针加上必要的offset，于是不论通过base1或者derived指针调用函数，都不要调整返回值；而如果通过base2指针调用，则是另外一个函数 。

IBM的方法呢？"把thunk搂抱在真正被调用的virtual function中。函数移开实现（1）调整this指针，然后才（2）执行程序员写的函数码；至于不需要调整的函数调用操作，就直接进入（2）部分"！

MS的方法以所谓的'address points'来取代thunk策略，即将用来改写别人的函数（也就是overriding function)期待获得的是'引入virtual之class'的地址。这就是'address point'

 

####4.4  指向Member Function的指针

指向 nonstatic member function 指针 :取一个nonstatic member function 的地址，如果该函数是nonvirtual ,则得到的结果是他在内存中的真正的地址。然而这个值也是不完全的，需要帮盯在某个class object的地址上，才能够通过它调用该函数。所有的nonstatic member functions 都需要对象的地址（以this参数指出）。

指向 static member functions 指针 ：而static member functions的指针则是函数指针，而不是指向member function 的指针。

指向 virtual member functions 指针：对一个virtual member funciton 取地址，所能够获得的只是一个索引值。编译器在评估求值的时候，会(*ptr->vptr[(int)pmf])(ptr) !
 

后三部分摘录自互联网，出处不明。

<!-- Copyright Info BEGIN -->
<p class="post-footer">
    原文地址：<a href="http://chesterlee.github.io/blog/2014/08/02/wo-de-c-plus-plus-dui-xiang-mo-xing-du-shu-bi-ji-si-zhang-the-semantics-of-function/"> 我的C++对象模型读书笔记--第四章 the Semantics of Function </a >
    <br/>
    <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh" ></a>版权声明：自由转载-非商用-非衍生-保持署名 | <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh" >Creative Commons BY-NC-ND 3.0 </a> | <img alt="知识共享许可协议" style="border-width:0" src="http://i.creativecommons.org/l/by-nc-nd/3.0/80x15.png" />
</p>
<!-- Copyright Info END -->