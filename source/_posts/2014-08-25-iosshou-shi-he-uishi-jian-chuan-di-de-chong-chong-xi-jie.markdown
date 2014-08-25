---
layout: post
title: "iOS手势和UI事件传递的种种细节"
date: 2014-08-25 20:03:53 +0800
comments: true
categories: iOS
keywords: UI，Gesture
description: 
---
做iOS交互，最为重要的就是UI的事件传递需要很清楚。当然也要包括各种"小"细节。
这里首先说说大的过程，也是UI事件传递的基本过程：当APP收到用户点击之后，就会做两个过程：

 * 1、遍历树，找哪个UIView是被Point击中。
 * 2、回溯tree node，形成响应者链。剩下的就是UIEvent的传递了。

现在让我们谈谈几个问题，一个View添加了Gesture了，那么在它之上的touches的回调会收到调用吗？一个view的superview的VC添加了手势，那么在该view中的touches会收到调用吗？如何正确地传递给superview相应touches调用？UITouch里面有什么东西？如果这些有些不太清楚，那么请往下看。  
  
####几个不得不提的概念:

* Touches began/Moved/Cancelled/touchesEnded是UIResponder的接口，用于接收触摸的各种事件。**过程简述为UserTouch-->UIApplication-->UIWindow-->UIView**。
* **UITouch代表一个手指，从开始接触屏幕到离开屏幕的触碰描述数据**。展开来说，UITouch包含了上一次手指的位置信息，当前的UITouch的状态TouchPhase，当前的时间戳以及UITouch要送往的那些以定义的手势。具体作用和关联，后面讲。
* 当发生了触摸时，UIEvent就会包含UITouch的数组，被App送到对应的Responder上（如果这个Responder书写了touchesBegan等UITouch的回调的话）。
* UITouch的状态机有TouchPhaseBegan、Moved、Ended几个。当然，分别对应那几个回调。这里请注意，**UITouch的状态机和手势的状态机如UIGestureRecognizerStateBegan，是完全不同的，千万不要混淆**.
* **Gesture的状态**：单独Gesture(如Tap)是有识别状态的，一个叫Possible，一个叫Recognized，一个叫Failed。而连续Gesture(如Pinch)相比之下多了began、changed等状态。

---

####UITouch的发送机制：  
一句话：手势先于UIView获取Touch的响应权。详细说来：当对屏幕进行点击的时候，如前面所说，UITouch的对象会从UIApplication被依次传递，这里如果View或者其superview添加了Gesture。默认状况下：UITouch就会被送到这个Gesture上去识别，如果Gesture无法识别则会送UITouch对象点到的那个UIView上去做响应。  

回答前两个问题：这个事件先发送到一个Gesture上，并不意味着会结束touches等的调用，这个调用只是delay了。所以，第一个问题，它的touches began会收到回调，它的superview及其VC都会收到touches began的回调。而且began，moved有可能还先于手势收到回调，原因是就是Gesture对UI事件调用的影响原则：  

* 如果在某个时刻，如果Gesture根据UITouch的变化识别出了自己的手势，把Possble状态跳到了Recognized状态。那么App将不会给对应的View或VC发送后续的UIEvent。转而，UITouch的事件将会没有走完，且会收到touches:Cancelled事件而不会收到touches:Ended事件。
* 如果整个过程中，Gesture都没有识别出自己的手势，那么它会设置自己的状态未Failed。那么这些UITouch对象会被转发给对应的UIView或VC。

最后说说UItouches中包含了一个gestureRecognizers数组作用是什么呢，注意这个小过程：

* UITouch是通过touchesBegan、touchesMoved、touchesEnded等方法被送到对应的Gesture上的。UITouch其实也就是记录了被注册的Gesture对象的指针，这里就被传递到手势上了。
* 因此，在默认状况下，如果一个view注册了touchesBegan、它或它的superview注册了Gesture，touchesbegan会先于Gesgure收到UITouch。

---

####所以，在这里总结下，默认状况下手势是通过touchesBegan等事件传递的方式获取UITouch，并根据UITouch的变化进行了手势自身的判断，如果手势判断成功，手势自身的状态就会从Possible变为Recognized，从而阻止了touchesEnded的调用。但是在这个过程中，默认touchesBegan是先被目标view调用的。  
最后说说delaysTouchesBegan 和 delaysTouchesEnded

* **概念**。这两个属性是手势的，默认delaysTouchesBegan为NO，delaysTouchesEnded为Yes。表示的涵义是：不延迟touchesBegan调用，目标View可以接到这些事件，而Delay TouchesEnded的调用。让手势获取最终的决定权。PS:他们的影响类似ScrollView中的delaysContentTouches变量，是否把延迟把UITouch传递给ScrollView中的subView。   
* **配置**。你可以把delaysTouchesBegan开启，阻止touchesBegan调用，直接收到手势。但是这样会有时让用户感到UI失去响应。
delaysTouchesEnded也可以被设置为NO，如果这样的话当delaysTouchesBegan为NO的情况，UITouch就会被传递到目标UIView上，相当于:UIView自己接事件，而且手势也去接事件。