---
layout: post
title: "我的C++对象模型读书笔记--第三章 Data语义学"
date: 2014-07-26 22:16:51 +0800
comments: true
categories: cpp
keywords: c++,cpp
description: 
---
###我的C++对象模型读书笔记--第三章 Data语义学  

 C++ standard 不强制规定“base class subjects的排列次序”“不同存取层记得data members的排列次序” ，也不规定“virtual funcitons 和 virtual base class“的实现细节。
当一个类并没有任何声明的时候，它实做一个对象的时候，实际分配了1个字节的大小的空间。从而使得这个对象能够有独一无二的地址空间。

而一个对象的大小不仅仅包含了该类所声明的nonstatic data members，而且还受到了，

1、支持语言特性（各种virtual特性）。

2、边界对齐特性。

两个部分的影响。而形成其最终大小。

***

####3.1 Data Member的绑定(The Binding of a Data Member)

 数据成员的绑定指数据成员与函数成员的一体性，因此无需使用“防御性编码风格”。

 所谓防御性编码风格，在C++具体所指是：

1、所有的data member都放到了class声明的开始处。（原因：成员函数调用成员变量的时候，需要知道成员变量是哪一个成员变量，早期c++没有做到绑定，那么就需要类似“前置声明”一样的做法告诉编译器。）

2、所有的inline function，不管长度都放置到class声明之外。（原因：当类的声明结束之后，member function本身才会开始被分析，而member function对数据成员的绑定操作会在整个class声明完成之后才发生。）

但是以上两种基本无需实现，因为C++编译器已经能够解决。

反倒是函数参数列表的形式，必须使用防御性编码风格。因为它不是在类完成之后被决议的，反而是在他们第一次遇到编译器的时候，就决议了。因此，例如，若在全局范围做typedef int length;并在class中先做了function的处理声明。那么若在随后的member data声明的时候加上typedef float length;则会出现参考操作不合法的情况。

 

####3.2  Data member 的布局

C++ standard要求：在同一个access section 的member只要符合“较晚出现的members在class object中有较高的内存地址”即可，各个member之间并不一定需要得到连续的排列。当前编译器都是把一个以上的access section连锁到一起存放于内存，形成连续区块。可以使用内存高低的办法，去写一个函数，判断同一个类中，哪一个access section先出现。

VPTR的存放位置也不一定，个个编译器自行规定。

 

####3.3 Data Member的存取

static data member：只要是static member，那么存取就是一个固定的时间，因为他相当于一个global变量。如果取一个static data member的地址，就会得到一个指向其数据类型的指针，而不是一个指向其class member的指针。因为static member并不包含在一个class object 之中。既然static member放在global data segment之中，就需要对他进行命名处理，防止命名的冲突（注意，这里应该使用的name-mangling处理方法，进行1，名称计算。2名称还原。）

再有，存取static data member不需要用对象(object)去获取，这只是一种语法上的便宜行事而已。

Nonstatic data member：对一个nonstatic data member进行存取操作，编译器需要把class object的首地址加上data member在对象中的偏移量(offset)。origin._x---等价于-->  &origin+(&Point3D::x-1); （-1的操作是用于判断指针是指向class的第一个member还是什么都没有指向。后面详述。）

因为它的偏移量会在编译期间得到，因此如果该data member属于一个base class subjects的时候情况也一样。

在虚继承的情况下：“经由base class subject存取class members”会增加一层新的间接性(bptr)。如果是由对象访问base class的数据成员(origin.x = 0;)，和上面的时间一样，因为它的偏移量在编译期间已经固定了，而如果是有一个指针来访问(如，pt->x = 0;)，则会很复杂，因为不知道该指针是指向什么类型的的对象。可能是base class，也可能是一个derived class，所以这个操作需啊放置到执行期能解决。

 

####3.4“继承”和 Data Member

C++中，一个derived class object的空间是自己的members加上base class members。而之间的排列次序在C++  standard中并未强制指定。大部分编译器而言，都是把base class members放在前面。

 

单独“继承”时候的Data Member：

这里书上提到了一个值得讨论的地方：

关于基类和子类的Data Member在补齐上的问题，是否需要以每个base member的空间+补齐空间+derive空间来实例化一个derive object？是的。因为如果每个class的对象空间补齐部分被填入member，那么对象之间的member wise复制操作将会破坏被复制的对象member。（当然，这是编译器落实的事情J）

 

##### 加上多态

加上多态之后，将会给类的存取时间和空间带来额外的负担：

1、加入了vtabl（对于类）。

2、加入了vptr（对于对象）。(C++ 1.0置于对象空间尾部，2.0之后置于首部。)

3、加强了构造器的能力。（构造器中，设置vptr指向的初值）

4、加强了析构器的能力。（析构器重，抹掉vptr指向的值）

 

#####加上多重继承：

注意，在多继承中，存在基类没有virtual function，子类存在virtual function的情况，那么自然多态就会被打破，编译器需要介入。在把子类指针赋值给基类ptr的时候，编译器也需要介入。

在进行指针转换的情况下，我们总需要进行基地址+offset的方法来调整(编译器是这样做的J)。通过这样的调整，能够使得基类指针类型，指向子类中基类的那个部分的首地址。（如 adrr = base +sizeof(XXX)）

 

#####虚拟继承：

策略是先安排好derived class的不变部分，而后再建立其共享部分。建立共享部分则需要通过一个指向该base class的指针，以访问virtual base class的成员。而不是像以前一样直接根据计算得到的偏移量得到。

因此存在 this->vbtpr->datamember = XXXX;这样的操作。vbptr意为指向虚基类的不变部分的member地址。

注意这里初期c++编译器对虚拟继承的设计，会对class的data member的“不良影响”如下：

* 1、每个对象都要为每一个virtual base class背负这额外的指针，来获得对应的虚基类“共享的部分”。(vbptr)。

* 2、如果虚拟继承的串链很长，那么间接存取的次数会增多(这里指多个层次的虚继承)，直接影响访问data member的效率。

我们如何处理这两个问题？使得，` 1：对象有固定的负担。2：固定的存取时间` 。

解决方法有这些：

第1个问题的解决方法：

(在vc9.0中，多一个虚继承的基类，确乎会多一个vbptr，下面的优化貌似没有实现。不知何故！)

微软编译器：在每一个类中，安插一个vbptr指向一个table，这个table包含了对应的虚基类的指针。

Bjarne的做法：使用offset。把virtual base class和 virtual function entry混合在一起，即，vtable使用正值负值来进行“索引”，若是正值，索引得到的是虚函数的地址；若是负值，索引得到的是从基地址（子类对象的）到继承的虚基类member地址的偏移。  

如图：  
  {% img /images/cppvirtualtable.jpg %}
    
一般而言，virtual base class最有效的应用形式是：一个抽象的virtual base，不包含任何data member。

#####3.5 对象成员的效率(Object Member Efficiency)（略过）

#####3.6 指向data member 的指针

   

对于指向 nonstatic data member 的指针（如：& Point3d::z），实际上是nonstatic data member在 class中的偏移量(注意：不能说成是class object中的偏移量)；更确切的说是 offset +1——也就是说，位于object 头部的成员对应的data member pointer 的值为1。但是，若对于取已经绑定于对象上的data member(如：&origin.z)，取得的是在内存中实实在在的地址。

这种位移从1开始的特性是基于如下考虑的：如何区分"没有指向任何data member"的data member pointer 和 "指向第一个data member "的data member pointer。

注意以上的位移1的处理，编译器已经计算得符合“通常思考原则”。1的额外计算，不需要程序员担心。


      




