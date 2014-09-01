---
layout: post
title: "我的C++对象模型读书笔记--第二章 构造函数语义学"
date: 2014-07-26 22:13:03 +0800
comments: true
categories: cpp
keywords: cpp,c++
description: 
---
#####2.1  Default Constructor 的构建操作

Default Constructors只有当编译器需要default constructor的时候才会合成出一个constructor, 只有下面的四种情况才会有nontrivial default constructor产生，其余的都是trivial default constructor。（nontrivial是合成的有用的默认函数，trivial则是没多大用处的构造器）

 * a、**带有Default Constructor 的 Member Class Object**

如果一个class自己没有定义constructor，但是内含一个member object，而后者含有default constructor，那么此class的implicit default constructor（隐式构造函数）就是nontrivial的，编译器会合成一个constructor。注意被合成出来的constructor只满足编译器的需要，而不是程序员的需要（只完成编译器向要做的事情）。（例如：一个类中包含了两个成员，一个成员是一个对象，而另一个成员是built-in的变量，那么编译器会调用这个member对象的默认构造器，built-in变量就会置之不理。J）

如果含有多个class member object，那么constructor要求初始化操作按照member object 在class 中的申明次序调用各个的constructors。

注意：这样的编译器默认合成构造器，是隐式的，如果类本身存在构造器，那么编译器会在程序需要的(可以理解为user code J)代码之前，安插必要的default constructor。

* b、**带有Default Constructor的Base Class**

如果没有定义构造函数，编译器会产生一个nontrivial constructor，先调用相应的base class's constructor，如果用户自己定义了constructor，那么编译器将会安插调用所必要的default constructors的程序代码到用户自己定义的constructor中。（通俗地讲：用户已经定义了构造函数，但是没有定义默认构造函数，那么编译器会把默认的东西加在进去，但不会重新写一个default constructor出来）。

* c、**带有一个Virtual Function 的 Class**

因为编译器必须要初始化vptr 和vtable。所以如果类本身包含了虚函数，或者继承链中包含了虚函数，那么构造器是必须要履行它的职责：调用的时候修改vptr和vtable的内容。这个过程，编译器是需要插足的。

* d、**带有一个Virtual Base Class 的 Class**

在此之中，如果向要通过一个指针访问最顶端的class的data member,必须要通过"指向基类的指针"来完成。 pa->_vbcx->i=1024; 而_vbcx则是在class object的创建的时候完成的(此时编译器也自己做出了决定J)。

* * *

#####因此需要辨析常见的两个误解：

1、任何类如果程序员没有定义default constructor，编译器就一定会合成出一个来。（以上说明了4种情况，编译器才会自己合成或者安插默认行为）

2、编译器合成出来的default constructor，会将class内的每个data member都设定为默认值。（以上第一点说明了，编译器只做自己“分内”的事情，built-in变量，不是它的职责。）

 

#####2.2 copy constructor 语意学

由三种情况使用到Copy Constructor: 1、object作为函数参数传递。2、函数的返回值是object。3、直接用一个同类实体最为初值而初始化。

如果没有用户定义显示拷贝构造函数（explicit copy construtor），那么当进行同类对象赋值的时候，则是以Default Member-wise Initialization的形式递归进行的。（做法细节是：把一个对象的成员值，赋值到另外一个对象对应的成员上，但是对象内部的member object不能直接赋值，而是继续调用该member object的默认构造进行Default Member-wise Initialization。）（与构造函数类似，也存在trivial和nontrivial之分。）

以上的做法，是包括着两种不同的判断行为，这样的行为取决于Bitwise  Copy Semantics的存在与否。（解释下：Bitwise Copy Semantics指按位逐次拷贝，那么在类的member data中，只包含了built-in类型的member的时候就具备了按位逐次拷贝的语义（对应对象值的拷贝）。在此情况下，编译器无需构建一个拷贝构造函数。）

当存在以下情况，我们就称一个类不具备Bitwise Copy Semantics的语义，编译器需要构建一个拷贝构造函数，且这个拷贝构造函数是nontrivial的：

1) class 内含一个member object， 而后者的class具有了一个copy constructor。

2)当class 继承一个base class,而base class 好有一个copy constructor。

3) class含有Virtual function。

4) class派生自一个继承串链，其中具有Virtual base classes。

后面两种比较复杂。当出现derived class 和base class之间的对象赋值的情况，必须要保证它们包含vptr的正确性，因此Bitwise Copy Semantics的语义不可行，所以也不具备。

注意：“必须要保证它们包含vptr的正确性”的详细理解是：在多态体系下，如果对象实现对应按位拷贝的时候，若发生的切割(sliced)的话，子类的vptr就会blow up，从而失去了多态特性了，这是不允许的。因此不能再此情况下实行按位的拷贝操作。

类似的，当存在虚基类(vitrtual base class)的时候，由于存在bptr于对象模型中需要妥善保存，如果出现了sliced情况，同样按位拷贝也会导致失去虚继承的情况。

 

#####2.3 程序转换语义学(Program Transformation Semantics)

这里讲解的是，编译器对用户程序进行处理的方法及原则：

2.3.1、显示的初始化操作(Explicit Initialization)

当用户使用显示初始化的时候，编译器会进行对象重新定义，并调用对应的拷贝构造函数。例如，已知X x0; 的时候，我们使用的时候，编译器会剖开成这样：

X x1(x0);          左边转化成右边           X x1,x2;

X x2(x0);          为编译器自行转换        x1.X::X(x0);

x2.X::X(x0);       编译器在此时如是处理的。

2.3.2、参数初始化(Argument Initialization)

当函数调用参数是对象(object)的时候，那么编译器可能会做如下两种不同的操作：

操作一：导入暂时性object。

例如：函数声明 void foo(X x0);

调用进入后，
1、编译器实作一个X _temp;

2、_temp.X::X(xx)。xx为实参。

3、重新改写函数调用操作，foo(_temp);

4、修改函数声明为引用。void foo(X &x0);

这里需要记忆的是，编译器会进行这样细节的实现。

操作二：如borland C++编译器，可能会把实参直接建构到对应堆栈上。（此书无详述）

 

2.3.3、返回值初始化(Return Value Initialization)

返回值初始化是函数的返回值使用对象(object)做返回的情形，编译器需要做额外的处理。具体处理如下：

编译器会产生临时的引用参数，并在return之前安插此引用参数的拷贝构造。

例如：X bar() { X xx; return xx;} 会被转化为：

void bar(X & _res)      //这里安插了临时引用参数

{

X xx;

xx.X::X();

_res.X::X(xx);              //这里安插了临时引用的拷贝构造函数

return;

}

 

而相应的 bar().memfunc(); //memfunc()为X的成员函数

会被编译器转化为：

X temp0;

(bar(temp0),temp0).memfunc();

 

2.3.4、使用者初始化(Optimization at the User Level)

在返回值初始化的编译器处理基础之上，用户自行定义一个“计算“的构造函数。具备抽象，但缺乏效率。

2.3.5、编译器初始化(Optimization at the Compiler Level)

 编译器的优化和返回值初始化的编译器处理方式一样。这样的优化方式叫做NRV(Named Return Value)。但是，不可或缺的是，我们需要一个拷贝构造函数，去开启编译器的优化。因此，在于是否一个类需要用户自己定义拷贝构造函数，这样的问题，需要如下解释：

例如：Class Point只包含了built-in的成员。那么，默认的拷贝构造函数则是trivial的。此时，我们无需去写一个显示的拷贝构造函数（explicit copy constructor），因为编译器会自动加上bitwise copy的行为，这里多此一举了。但是，如果class面临了很多以传值的方式做return value的时候，我们就需要去显示地提供一个拷贝构造，以开启编译器的NRV优化策略。

 

#####2.4 成员初始化队伍(列表)(Member Initialization List)

这里讲解的是，编译器如何处理初始化成员列表的。

下列情况中，必须要使用member initialization list进行data member 的初始化：

1) 当初始化一个reference member。

2) 当初始化一个const member。

3) 当调用一个base class 的constructor，而他拥有一组参数的时候。

4) 调用一个member class 的constructor ,而他有一组参数。

        

编译器会一一操作初始化列表的成员，以适当的顺序在class的constructor中插入初始化操作，并且保证它们都发生在任何explicit user code 之前。

这里总的需要留意两点：

第一个：member object的初始化，最好放到初始化列表里面。若放置于构造器中，则会产生临时的object0，初始化之，在做赋值运算给object，然后object0自行销毁，期间耗时耗力。若置于初始化列表，则编译器会在构造函数中，user code之前，调用object的构造函数，予以初始化。

      

第二个：初始化列表的初始化次序。初始化次序和member在类中的声明次序一致。相互关联的member，需要十分留意初始化列表中，其中依赖的次序。解决的办法：把其中一部分使用初始化列表初始化，而另一部分放置到构造函数中使用user code予以表达，这样即便次序存在依赖，也会只“先执行合成的，再执行user的code”。

<!-- Copyright Info BEGIN -->
<p class="post-footer">
    原文地址：<a href="http://chesterlee.github.io/blog/2014/07/26/wo-de-c-plus-plus-dui-xiang-mo-xing-du-shu-bi-ji-er-zhang-gou-zao-han-shu-yu-yi-xue/"> 我的C++对象模型读书笔记—第二章 构造函数语义学</a >
    <br/>
    <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh" ></a>版权声明：自由转载-非商用-非衍生-保持署名 | <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh" >Creative Commons BY-NC-ND 3.0 </a> | <img alt="知识共享许可协议" style="border-width:0" src="http://i.creativecommons.org/l/by-nc-nd/3.0/80x15.png" />
</p>
<!-- Copyright Info END -->