---
layout: post
title: "AutoLayout中的约束"
date: 2014-10-30 21:03:40 +0800
comments: true
categories: iOS
keywords: autolayout,autolayout概念
description: 
---
前段时间一直很忙，没有时间把AL的东西拿来总结下，今天整理下约束：

###约束的类型
对于Autolayout而言，最为核心的基础就是约束。那么在iOS中约束到底有几种？它们有那些作用呢？下面说下。


 * **NSLayoutConstraint**，开放类，几乎是程序员最常用的约束。它用于设置view在view tree之间的关系，自身大小等。
 
 * **NSContentSizeLayoutConstraint**，私有类。用于衡量view内容和大小相关的约束。比如Hugging和Compression，控制view的内容显示。
 
 * **NSAutoresizingMaskLayoutConstraint**，私有 类。由Autosizing mask转换到Autolayout系统中的约束表达。
 
 * **_UILayoutSupportConstraint**，私有类。布局支撑约束，它包括Top和Bottom的约束，用于控制view的显示边界，例如，它限制view的顶端显示不会和状态栏重合。

 * **NSIBPrototypingLayoutConstraint**，私有类。如果你丫在Storyboard中添加了一个UI控件，且没有在Storyboard中添加任何约束，但是标注了你要使用Autolayout，那么在实际的运行期，系统会默认为它添加NSIBPrototypingLayoutConstraint约束。
 
值得一提的是：NSContentSizeLayoutConstraint最主要的作用是和intrinsic size一起工作，通常这个约束和Layout约束共同决定view的显示方式。


###约束优先级
这里不去介绍约束优先级有哪些枚举值，而是说明它存在的意义：它通常是形容一个约束重要性的指标，通常两个约束如果共同决定一个显示属性的显示，当他们发生了抵触的时候，优先级表示其中一个约束，相比另外一个而言，更加重要。    

另外，**优先级就是浮点数，通常用其整数表达**。

###ContentSize约束
AL系统有根据显示内容来自适应尺寸的能力。这个能力是由，Intrinsic Content Size内容大小、content hugging约束、content compression约束来共同决定。

####intrinsic size
Intrinsic Content Size保证了当view的内容变化后，自己适应内容的大小的描述。  

自定义View需要自己计算适合自己的intrinsic size。而imageview、label等含有内容控件(除了UITextView)都会自己计算"适合内容的Size"，文档没有提是minSize来fit其中的Content，我就没这么写，不过应该可以这样理解。

####Content Hugging
抗拉伸属性。其实它是一个约束，它使得它本身的大小更加贴合显示的内容。例如：如果一个UIView的内容显示需要200的宽度，那么如果UIView本身有宽度约束(此约束设置宽度为400)，此时若抗拉伸属性的优先级比宽度约束要高，View本身的大小就不会变化，反之就会被拉伸。

####Compression Resistance
抗压缩属性。它的一切和抗拉伸相反。

####Layout Constraints
其实Layout约束是程序员必须熟悉的。一个线性公式就说明了它的意义：    

	view1.attr1 = view2.attr2 * multiplier + constant  
这里=可以是>=，<=，<，>等。

剩余就是描述Layout Constraint的**生成和装载方式**：

生成方式比如:Layout Constraints的描述方式可以是原生NSLayoutConstraint来实现，也可以是VFL，也可以是自己封装的库。
这里要进一步明确一个概念：约束是一种概念，它说明了一种关系。

####约束的装载方式

1、View形容它自己的约束(一元约束)，添加到自己或者直系SuperView中。  
2、View和View之间的约束，如果view之间是父子关系，添加到父亲上。如果view和view是兄弟关系，则添加到他们共同的父亲节点上。  
3、View和View之间的约束，如果他们是两个不同的分支上的，则需要把约束添加到他们共同的祖先上面。  
相信这样说应该比较清晰了。

####约束的比较

通常而言，比较对象，需要比较一个对象的所有成员变量，如果一一相等，则表明他们相等。
但是，对于约束而言，这个比较方式比较例外：除了优先级以外的参数都要一致，才是一样的约束。也就是说优先级是不需要参与比较的。例如：
	
	- (BOOL) isEqualToConstraint: (NSLayoutConstraint *) constraint 
	{
          if (self.firstItem != constraint.firstItem) return NO;
          if (self.secondItem != constraint.secondItem) return NO;
          if (self.firstAttribute != constraint.firstAttribute) return NO; 
          if (self.secondAttribute != constraint.secondAttribute) return NO; 
          if (self.relation != constraint.relation) return NO;
          if (self.multiplier != constraint.multiplier) return NO;
          if (self.constant != constraint.constant) return NO;
          return YES; 
	}

###约束使用的几个建议
1、充分利用约束的优先级。不要把什么约束的优先级都搞成Require。  
2、约束没有顺序而言。优先级越高越会得到满足。  
3、看上去有问题的，在iOS7和iOS8的AL系统对于Transform的行为，不太一致。transform的先后顺序不一致，有兴趣的朋友可以试试。  
4、不要无聊地建立一个ScrollView内部的view和Scrollview外部的view的约束。  
5、请注意UpdateConstraint、Layoutsubview、DrawRect的顺序，在这个管线中的特定阶段做自己想自定义的事情。  
6、注意约束描述的合理性，以避免自我矛盾的描述引起的Layout约束的冲突、歧义甚至崩溃。
