---
layout: post
title: "谈谈iOS中的多屏幕显示"
date: 2014-08-28 21:15:49 +0800
comments: true
categories: iOS
keywords: AirPlay,Extern Screen on iOS
description: 
---
最近接到需要进行ipad通过Apple TV进行多屏显示的需求，所以研究了下，可以在自己App中对外部荧幕进行Mirror和Different Content的切换。如果你没玩过，实在是有些可惜，不妨往下看看：

####初步的概念
iOS App中UIWindow通常包含了App的view，连通了当前的硬件设备。而和Window联系到一起的Screen代表了特定使用的显示内容。如果你的app允许用户使用外接显示器，我们就需要创建一个UIWIndow对象来管理对外界显示器的展示。
####要显示内容到外部屏幕？请添加UIWindow
如果你要支持外屏显示，你需要建立另外一个不同的window对象，从而让这个UIWindow对象展示的内容投射放到外部设备上。默认你可以在两个window上展示相同的内容，因为他们默认是mirroring的。当然，你可以把这个mirroring关闭了，赋值为nil或初始化一个UIWIndow到上面去。
####Window和Screen的概念
先说说UIWindow，它组合了所有需要显示的ViewTree，并负责对subview的展示，组织他们的继承体系，以及对它们的事件传递。通常每一个App都有一个window对象来展示app自己的UI（在iOS设备上）。如果有外部显示器链接到设备上，那么app就可以使用第二个window来进行大荧幕数据的展示。  
再说说UIScreen对象，对于UIWindow而言，每一个UIWindow中有一个UIScreen对象。UIScreen对象抽象地描述了一个具体显示设备的信息：bounds、mode、brightness等。你也可以监听各种消息，比如连接上显示器了，断开显示器了，显示器亮度调整了等的消息。
####Window和Screen的其他用途
1、通过UIWindow来调整Window的亮度。还有wantsSoftwareDimming等等。
2、通过UIScreen displayLinkWithTarget:selector: 接口来在runloop进行CoreAnimation每一帧的绘制，或使用OpenGL es直接绘制。
####StoryBoard做了点事情
使用Storyboard，建立默认的VC。它就帮你做了下面几件事情：

* Instantiates a window. （初始化Window）
* Loads the main storyboard and instantiates its initial view controller. (初始化Storyboard中的initial VC)
* Assigns the new view controller to the window’s rootViewController property and then makes the window visible.(把initial VC作为UIWindow中的RootVC，然后让Window可见)

####外接显示器的实际做法
默认的iOS行为：Mirror。如果默认什么都不处理，那么对于一个App而言，它的内容是直接镜像到荧幕上的。除非你要显示和iOS设备荧幕不一样的内容，你可以这样做：  

* 程序一启动，就要检查外部显示器的存在。并且注册Screen的连接和断开连接的Notification。
* 当外部显示器可用了，无论是程序启动检测，还是收到Screen连接的Notification，你都要创建并且配置一个UIWindow给这个外部设备。
* 连接这个UIWindow和这个合适的UIScreen对象，展示第二个UIWindow，并更新。

BTW：如果要开启镜像，请把多出来的UIWindow给干掉就好了。
####代码时间
好，接下来是代码时间，上面3步的代码如下：  
1、当外部设备Ready的时候，创建一个UIWindow匹配外部抽象的UIScreen对象。

    - (void)checkForExistingScreenAndInitializeIfPresent
    {
       if ([[UIScreen screens] count] > 1)
      {
         UIScreen *secondScreen = [[UIScreen screens] objectAtIndex:1];
         CGRect screenBounds = secondScreen.bounds;
         self.secondWindow = [[UIWindow alloc] initWithFrame:screenBounds];
         self.secondWindow.screen = secondScreen;
         self.secondWindow.hidden = NO;
      } 
    }
注意：UIWindow最好先和UIScreen连接起来，然后再显示UIWindow的内容，最好不要先显示UIWindow的内容，再链接UIScreen。因为非常耗资源。

2、注册Screen的连接和断开事件  
  
     - (void)setUpScreenConnectionNotificationHandlers
    {
        NSNotificationCenter *center = [NSNotificationCenter defaultCenter];
        [center addObserver:self selector:@selector(handleScreenDidConnectNotification:) name:UIScreenDidConnectNotification object:nil];
        [center addObserver:self selector:@selector(handleScreenDidDisconnectNotification:)
    }
3、处理断开和连接的事件


    - (void)handleScreenDidConnectNotification:(NSNotification*)aNotification
    {
       UIScreen *newScreen = [aNotification object];
       CGRect screenBounds = newScreen.bounds;
       if (!self.secondWindow)
       {
        self.secondWindow = [[UIWindow alloc] initWithFrame:screenBounds];
        self.secondWindow.screen = newScreen;
       }
    }
    
    - (void)handleScreenDidDisconnectNotification:(NSNotification*)aNotification
    {
      if (self.secondWindow)
      {
        // Hide and then delete the window.
        self.secondWindow.hidden = YES;
        self.secondWindow = nil;
      }
    }
 以上代码也可以在水果文档的多屏编码指南中找到。其实这只是一个开始，表明自己Create的UIWindow可以以抽象的方式来在外部显示器上展示，至于不断添加业务逻辑，肯定需要更好的代码封装。最后说一句，水果文档建议在AppDelegate中进行Screen的监听，其实可以按需求来进行工作。大概就这样啦。
