---
layout: post
title: "我的C++对象模型读书笔记-第一章 关于对象"
date: 2014-07-26 00:18:06 +0800
comments: true
categories: CPP
keywords: C++,C Plus Plus,
description: 
---

多亏几年前的积累，现在看来，之前的总结一点也没过时，相反，益处良多。必须转入。    

从准备找工作至今，才阅读完B.Lippman的Inside The C++ Object Model。所以索性整理一下对C++对象模型一书的读书心得。
* * *
###第一章         关于对象

首先应该说下C的“算法驱动”思路：C语言中，“数据”和“处理数据的操作”十分开来的。简言之，这样的程序方法叫做“程序性的”，由以功能为导向的函数的算法所驱动，处理的是来自外部的数据。

而在C++中，则使用的是独立的“抽象数据类型abstract data type（ADT)”所驱动。这样符合软件工程的设计。如果不使用virtual，C++本身的class布局及存取时间和struct没有区别。言外之意，只有存在虚函数(virtual function)和虚基类(virtual base class)这样的形式，才使得class实例化一个对象时，存在额外的负担。

 

####1.1  C++对象模型(The C++ Object Model)

C++中，有两种数据成员(class data member):静态成员(static)和非静态成员(nonstatic)。有三种成员函数(class member function)：静态成员函数(static)、非静态成员函数(nonstatic)和虚函数(virtual)。

对象模型的设计方式（其实也是设计对象模型时候的一些思路）

1、简单对象模型：

这个模型的设计目的是：降低设计复杂度。但是相应的陪上了执行期的空间和效率。设计思路如下：

一个object 是一系列的slots(指针)，每一个slot指向一个member（成员） : data member and function slot（数据成员或函数成员）。每个data member and function member都有自己的slot。此模型中，meember 不放在object中，而是存放slot，由slot指向member。

2、表格驱动对象模型：另一种方法。把所有member(成员)分发到两个table（表格）中，然后用class的对象的两个slot指向这两个table。

3、C++对象模型：由简单对象模型演化过来。设计方式如下：

Nonstatic data members（非静态成员） 存放在一个class object内，static data member（静态成员）放在class object 外，而static and nonstatic function memebers 放在所有的class object 之外。而virtual functions分两个步骤完成：

a、每一个class（类）产生出一对之想virtual functions的指针，放在virtual table（虚表）中。

b、每一个class object（对象）添加一个指针（vptr），指向相应的virtual table（虚表）。而vptr的设定和重置是由constructor（构造函数），destructor（析构函数），copy constructor（拷贝构造函数）完成的。而每一个class 所关联的type_info存放在表格的第一个slot中。  
***

__C++对象模型的优点是：综合了存取时间和空间效率__。

缺点是：（任何一种对象模型的设计都存在缺点，不是吗J）如果应用程序的代码本身没改变（即处理逻辑、函数等没变化），但是用到的类对象的非静态数据成员有所修改，那么与其相关的程序代码都要重新编译。

如果对以上三种模型设计方式加上了继承，结果如下：

1、简单模型：每一个base class（基类）可以被derived class object（子类）内的一个slot指出，该slot内含base class subobject的地址。    缺点：间接性导致了空间和存取时间上的负担， 优点：class object的大小不因base class的改变而受影响。

（这里简单而言：缺点是如果继承的层次较多，那么子类包含的“指向基类的指针”就会越来越多，那么“负担”会越来越重。但是相应的好处是：如果基类发生成员改变等变化了，子类不需要重新编译，因为与之关联的仅仅是一个指针而已。）

2、Base Table 模型。base class table产生出来，表格中的每一个slot内含一个相关的base class地址，类似virtual table内含一个virtual funciton地址。每个对象含有一个BPTR，指向base class table。缺点：由于间接性导致了空间和存取时间上的额外负担。优点：每个class object对于继承都有一直的表现方式：每个class object 都应该在一个固定的位置上安放一个base table指针，与base classes的大小和树木无关，而且不改变class objects的本身的大小。

  3、 C++的模型。base class subject的data members直接放在derived class object中，提高了存取的效率，却使得只要每个base class data member改变，derived class 就需要重新编译。而Virtual base class的产生，需要一个间接的base class的表现方法。

对象模型如何影响程序：不同的对象模型，会导致现有的程序代码必须修改以及必须加入新的程序代码两种结果。所以不管是哪种模型方式，编译器都采取这样的两种额外处理，使得程序结果正常。

 

####1.2关键词带来差异(A Keyword Distinction)

为了支持8种整数的运算，所以C++支持了运算符重载。在C++中，必须保证同一个access section内的数据，其声明的次序出现在内存布局中，而不同的access sections的存放次序是不定的。同样，base class 和derived class的布局次序也是没有规定的。

 

####1.3 对象的差异(An Object Distinction)

C++支持的三种programming paradigms:  

1、__程序模型__：类似于C语言风格。

2、__抽象数据类型模型__：一组抽象的数据和它的相应表达式。如今叫OB（基于对象），不需要扩展，空间紧凑，速度快，编译期决定了函数引发的操作。

3、__面向对象模型__：彼此类似的类型，通过一个抽象的base class被封装起来。
C++以下列的方法支持多态：

a、经由一组隐含的转化操作，将derived class object的地址赋给base class pointer  : shapre *ps = new circle();

b、经由virtual function机制： ps->ritate();

c、经由dynamic_case和typeid运算符：  if (circle * pc= dynamic_case <circle *> (ps) )  

需要多少的内存才能表现一个class object，一般而言有下面三种：

1) 其nonstatic data members的大小 

2) alignment的需求

3) Virtual funciton and Virtual base class的额外负担
 

指针的类型：每个指针的大小是一样的，但是有“指向不同类型之各指针“间的区别，每个指针只能覆盖其指向对象的内存范围。

指针类型会交到编译器如何解释某特定地址中的内存内容及其大小。

对于多态情形：

指针在编译期的时候，会被决定两点：

1、当前指针类型只能调用的接口。

2、能够调用接口的访问权限。

但是在执行多态的情况下，编译期没有落实对应的函数执行实体，因为具体对象型别没有确定，故只有放到执行期，通过vptr找vtabl来落实具体的接口实现。
