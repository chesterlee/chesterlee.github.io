---
layout: post
title: "我的C++对象模型读书笔记-- 第五章 Data语义学"
date: 2014-08-02 09:27:24 +0800
comments: true
categories: cpp
keywords: c++,cpp
description: 
---
* 合理设计之一：将共享的成员放置到base class中。

* 合理设计之二：对于class的成员，应该在构造函数中或者其他的成员函数中初始化。如果让子类class去初始化base class的成员，将会破坏封装性。

纯虚析构函数设计者一定要定义它，因为子类调用析构函数的时候，会默认静态地调用积累的虚析构函数，因此中间层如果没有定义，则会出现连接失败。因此一个较好的方案是，不要把析构函数写成纯虚。

如果成员member function声明成虚函数，但是函数体是一个inline形式的（如果简单地return一个member data），那么每次被调用，而根本不会被子类复写的。这样的函数调用会消耗很大的性能。因此，不要把所有函数都写成虚函数，而去等编译器优化的时候去除虚函数声明。

虚函数是否需要const？如果不使用const，则无法获取一个const引用或者指针，如果使用const，如果derive的对象要修改某一个datamember。因此无法协调。所以只有不使用const为好。

 

####5.1无继承体系下的对象构造（关于编译器对代码“膨胀”的实现）

 

当类是一个Plain Ol’ Data的形式，只包含built-in数据的struct或者class的时候。那么当存在对象定义、复制、删除或者return的时候，理论上会出现trivial的构造函数、析构、赋值和拷贝构造函数。但是实际上，编译器要么是没有默认合成，要么是合成了却没有调用。

关于构造函数的“膨胀”：如类Point定义一个变量local(1.0,1.0,1.0)。那么对于编译器，会对默认构造函数进行内联膨胀，膨胀成 Point local; local._x = 1.0; local._y = 1.0; local._z = 1.0;。而对应Point *pheap = new Point;而言，会被编译器修改为如下代码：
    
     Point *pheap = new Point;   
     if(pheap) heap->Point::Point();

之后，再继续*pheap进行内联的膨胀。

加上virtual function之后，也会引起编译器对构造器，析构器的膨胀。加上virtual function之后，vptr的初始化就会执行，初始化于构造器中，置于所有user code之前。

因此，一个构造器名，如 Point::Point();会被膨胀为：

    Point::Point(Point *this)

而函数体最前面，会被编译器添加：

    this->_vptr = _vtbl_adrr;  //_vtbl_adrr指虚表首地址。

相应的，对于赋值构造、拷贝构造，编译器都会添加this,并用this去指向vptr，使得vptr被初始化，并指向当前的“合适的”虚表首地址。

既然标准保证在member initialization list对应的代码执行前，该对象的vptr已经被正确的设置好，那么在member initialization list中调用virtual member function，从vptr的角度来看是安全的；但如果考虑到可能存在的member之间的依赖关系，这种风格显然是不安全、不推荐使用的。

一般而言，如果程序中频繁的在函数中返回一个local class object，那么在设计class时，提供一个copy constructor就比较合理，它的存在会触发NRV优化。

 

####5.2继承体系下的对象构造（编译器对构造函数的扩充内容和方式）

 

编译器扩充构造函数的主要方面可能如下：

1、记录在initialization list 中的data members 初始化裁做会被放进 constructor的函数本身，并以members 的声明顺序为顺序。

2、如果有一个member 没出现在member initialization list 之中，但是他又一个default constructor ，那么该default constructor 必须被调用

3、在这之前呢，如果class object 具有virtual table pointers，那么它们必须被设定处置，以指向适当的virtual table。

4、在那之前，所有的上一层的base class constructors必须被调用，以base class 的声明顺序为顺序。

 a) 如果base class 被列于member initialization list 之中， 那么任何明确制定的参数都被传递过去。

 b) 如果base class 没有被列于member initialization list 之中，而它具有default constructor,那么会调用default constructor。

 c) 如果base class是多重继承下的第二或后继的base class,那么this 指针必须有所调整。

5、 在那之前，所有的virtual base class constructors 必须被调用，从做到又，从最深到最浅。

 a) 如果 class 被列于member initialization list 之中， 那么任何明确制定的参数都被传递过去， 如果 class 没有被列于member initialization list 之中，而它具有default constructor,那么会调用default construcotr

 b) class 中的每一个virtual base class subject 的便宜量必须在执行期间可被存取

 c) 如果class object是最底层的class, 其constructors可能被调用，某些用以支持这个行为的机制必须被放进来！

具体的例子说明，见书207页-210页。

 

* 关于虚拟继承，编译器需要对构造器处理的：

当加入虚继承的时候，每个子类所拥有的是一个bptr，但是在子类的构造中，不可以每个子类都去调用 this->bptr->base::base()，因为虚基类只能被构造一次，因此，只有到最底层的子类，再进行对虚基类的构造器的调用。因此，在处理虚拟继承类族中的构造函数时，都会在构造函数中加入bool参数：

如把derive::derive()改造为 derive::derive(derive *this,bool most_derive)。若bool值为真，则说明已经到了继承体系的最底层，则会调用虚拟基类的构造函数。否则不调用虚基类的构造函数。（这里的处理，编译器会在构造函数中自行加入。）

这里也引发了一个问题：当一个子类对象完整的被定义出来的时候，我们会调用虚拟类的构造函数，当一个子类对象没有完整定义出来的时候，此构造函数则不会调用。因此可以生产出更有效率的编译器，它可以自行“拆解”构造器，使其更有效率，使一个“完整对象”的构造器能调用虚基类，并使用vptr。而使得“不完整对象”的构造器，无法调用虚基类和设定vptr。

 

vptr初始化语义学(The Semantics of the vptr Initialization)

vptr初始化是在执行期完成的，一般会发生4件事：

1、        derived类无条件调用base类的constructor

2、        调用完毕base的 constructor之后，初始化vptr。

3、        构造函数的成员初值列如果存在，那么将会被扩展，并且置于vptr构建之后。

4、        执行用户代码。

 

####5.3 对象复制语意学

一个class的默认assignment operator，在如下情况下不具备bitwise copy语意：

class含有一个member object，且其具有assignment operator。

class的base class 具有 assignment operator。

class包含virtual function。

class有virtual base class。

概括的说，non-trivial的assignment operator不具备bitwise copy semantics。只有non-trivial的assignment operator 才会被编译器合成出来。

需要注意的时，并不存在与member initialization list 相对应的copy assignment list。

assignment operator 在 virtual inheritance 下表现不佳。

任何解决方案如果是以程序操作为基础，导致较高的复杂度和较大的错误倾向，一般公认，这表明语言在该方面存在弱点。

 

####5.5解构语义学

如果class内带了object，且此object带有析构函数，那么class的就会自动合成一个析构函数，否则视为不需要（即便class包含虚函数）。

顺带需要说的是，析构函数的执行顺序和构造函数相反，此刻包括了vptr的初始化、膨胀等操作的反操作。

 

部分段落摘录自网上，稍加修改，出处不明。