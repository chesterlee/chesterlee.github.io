---
layout: post
title: "Autolayout中的接口释义"
date: 2014-10-13 21:48:18 +0800
comments: true
categories: iOS
keywords: Autolayout接口, autolayout
description: 
---
下面是对最新iOS8sdk中Autolayout的category接口的解释，如果有错误，希望指正，谢谢！

#####UIConstraintBasedLayoutInstallingConstraints—-约束的添加和删除

    -(void)addConstraint:(NSLayoutConstraint *)constraint NS_AVAILABLE_IOS(6_0);    //添加约束  
    -(void)addConstraints:(NSArray *)constraints NS_AVAILABLE_IOS(6_0);             //添加约束集合
    -(void)removeConstraint:(NSLayoutConstraint *)constraint NS_AVAILABLE_IOS(6_0); //移除约束 
    -(void)removeConstraints:(NSArray *)constraints NS_AVAILABLE_IOS(6_0);          //移除约束集合       

#####UIConstraintBasedLayoutCoreMethods—-约束的核心函数

    -(void)updateConstraintsIfNeeded NS_AVAILABLE_IOS(6_0);  //立即更新约束
    -(void)updateConstraints NS_AVAILABLE_IOS(6_0);//更新约束的接口，可以overrite来更新你特别需要更新的约束
    -(BOOL)needsUpdateConstraints NS_AVAILABLE_IOS(6_0);   //查询是否当前view需要更新约束
    -(void)setNeedsUpdateConstraints NS_AVAILABLE_IOS(6_0);// 标记需要更新约束，在下一个cycle的时候，run一次约束更新业务流程

#####UIConstraintBasedCompatibility—-约束特性的打开和关闭


    //查询当前是否使用的Autoresizing 
    - (BOOL)translatesAutoresizingMaskIntoConstraints NS_AVAILABLE_IOS(6_0);  

    //如果手动添加约束，就需要call这个接口，默认的view是添加了autoresizing mask的，
    此时需要把他关闭，把此方法的flag传递为NO即可。但是SB/Xib会自动帮你做这个事情。
    仅仅手动添加约束需要。 
    - (void)setTranslatesAutoresizingMaskIntoConstraints:(BOOL)flag NS_AVAILABLE_IOS(6_0);
    
    //约束的Layout方式是lazily使用的。如果你在-updateConstraints中来初始化你的约束，
    但是如果没有添加约束的话，系统就不会调用这个-updateConstraints接口。
    这个就是鸡和蛋的问题，所以用这个方法在自定义的view中返回YES，表示一定要用AL来约束。
    比如在一个Cell内部，如果在-updateConstraints进行了constraint的更新，添加删除等，
    那么就调用此接口。  
    + (BOOL)requiresConstraintBasedLayout NS_AVAILABLE_IOS(6_0);

#####UIConstraintBasedLayoutLayering（baselayout）

    //设置view在当前Frame下对应的alignmentRect的值，这个是AL系统回调函数  
    - (CGRect)alignmentRectForFrame:(CGRect)frame NS_AVAILABLE_IOS(6_0);
      
    //设置view在当前alignmentRect下对应的frame的值，这个是AL系统回调函数  
    - (CGRect)frameForAlignmentRect:(CGRect)alignmentRect NS_AVAILABLE_IOS(6_0);
    
    // 设置当前view计算alignment rect所需要用到UIEdgeInsets值，比较常用  
    - (UIEdgeInsets)alignmentRectInsets NS_AVAILABLE_IOS(6_0); 
    
    // 使用baseline对齐的约束layout时。当前的自定义view告诉AL到底是以本view
    的哪个来做对齐的（返回当前view的subview，就以这个subview做基线对齐），
    或则是本view自己（这个是默认的对齐方法情况）  
    - (UIView *)viewForBaselineLayout NS_AVAILABLE_IOS(6_0);
    
    // 默认的IntrinsicSize使用的value，为(UIViewNoIntrinsicMetric,UIViewNoIntrinsicMetric)，也就是(-1, -1)// -1  
    UIKIT_EXTERN const CGFloat UIViewNoIntrinsicMetric NS_AVAILABLE_IOS(6_0);
    
    // 当前view展示其内容的最小的size
    - (CGSize)intrinsicContentSize NS_AVAILABLE_IOS(6_0);
    
    // 刷新展示其内容的最小的size，这个要自己主动call，否则系统无法检测。   
    - (void)invalidateIntrinsicContentSize NS_AVAILABLE_IOS(6_0);

    //获取当前view在axis方向上的抗拉伸优先级  
    - (UILayoutPriority)contentHuggingPriorityForAxis:(UILayoutConstraintAxis)axis NS_AVAILABLE_IOS(6_0); 
    
    //设置当前view在axis方向上的抗拉伸优先级   
    - (void)setContentHuggingPriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis NS_AVAILABLE_IOS(6_0);      
    
    //获取当前view在axis方向上的抗压缩优先级
    - (UILayoutPriority)contentCompressionResistancePriorityForAxis:(UILayoutConstraintAxis)axis NS_AVAILABLE_IOS(6_0);  
    
    //设置当前view在axis方向上的抗压缩优先级  
    - (void)setContentCompressionResistancePriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis NS_AVAILABLE_IOS(6_0);


#####UIConstraintBasedLayoutFittingSize(Autolaytou的size计算)

    // 一个标记，按可以展示内容最小的压缩大小来Layout对应的size（常用）  
    UIKIT_EXTERN const CGSize UILayoutFittingCompressedSize NS_AVAILABLE_IOS(6_0);  
    
    //一个标记，按可以展示内容可扩张最大的大小来Layout对应的size，没查到是以怎样的标准来计算的  
    UIKIT_EXTERN const CGSize UILayoutFittingExpandedSize NS_AVAILABLE_IOS(6_0);
    
    //配合UILayoutFittingCompressSize和UILayoutFittingExpandedSize来使用，
    在当前view的subviews的intrinsic size都可以计算的情况下，可以获取最小压缩（UILayoutFittingCompressSize）
    的size或最大（UILayoutFittingExpandedSize）的size。  
    
    - (CGSize)systemLayoutSizeFittingSize:(CGSize)targetSize NS_AVAILABLE_IOS(6_0);   
    
    //在两个方向上做不同类型的Fit size计算，相对而言，systemLayoutSizeFittingSize是在两个方向上都一致
    （都是Compressed或都是Expanded）的用法。  
    - (CGSize)systemLayoutSizeFittingSize:(CGSize)targetSize withHorizontalFittingPriority:(UILayoutPriority)horizontalFittingPriority verticalFittingPriority:(UILayoutPriority)verticalFittingPriority NS_AVAILABLE_IOS(8_0);

#####UIConstraintBasedLayoutDebugging(AutoLayout调试使用)
    
    //影响当前方向的约束集合  
    - (NSArray *)constraintsAffectingLayoutForAxis:(UILayoutConstraintAxis)axis NS_AVAILABLE_IOS(6_0);
    
    //是否含有含糊不清的约束 
    - (BOOL)hasAmbiguousLayout NS_AVAILABLE_IOS(6_0); 
    
    //待整理  
    - (void)exerciseAmbiguityInLayout NS_AVAILABLE_IOS(6_0);  


