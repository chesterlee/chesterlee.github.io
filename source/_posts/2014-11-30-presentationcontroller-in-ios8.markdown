---
layout: post
title: "iOS8中的PresentationController"
date: 2014-11-30 10:28:24 +0800
comments: true
categories: iOS
keywords: PresentationController, iOS8, iOS8 UI
description: 
---
在之前的一篇文章<a href = http://chesterlee.github.io/blog/2014/08/26/tan-tan-ioszhong-de-vctransitionfa-zhan/>谈谈iOS中的VCTransition发展</a>中，提到了Apple对iOS动画转场的不断改进，降低动画耦合。如今，在iOS8中的PresentationController，这样的改进更加彻底。下面对PresentationController进行简单的讲解： 


###Alert、ActionSheet和popOver的重定义
在iOS8中，Alert、ActionSheet和popOver都被重新定义了。我们一个一个说。  

####Alert和ActionSheet
对于**Alert**而言，使用全新的类UIAlertController来进行处理。比如要完成一个常用的点击事件的提示，提示按键点击回调等操作，比原有iOS7之前的Delegate系API，要简单很多，他更像是FRP的一种演化。  
例如，我们想对提示消息、确定、取消和删除按钮进行事件回调时，可以如下操作：

    
     let alertControllerSheet : UIAlertController = UIAlertController(title: "test",
            message: "AlertMessage",
            preferredStyle: .Alert)
        
        alertControllerSheet.addAction(UIAlertAction(title: "Confirm",
            style: .Default,
            handler: { (action: UIAlertAction!) -> Void in
            println("press confirm")
        }))
        
        alertControllerSheet.addAction(UIAlertAction(title: "Cancel",
            style: .Cancel,
            handler: { (action: UIAlertAction!) -> Void in
            println("press cancel")
        }))
        
        alertControllerSheet.addAction(UIAlertAction(title: "Delete",
            style: .Destructive,
            handler: { (action: UIAlertAction!) -> Void in
            println("press Delete")
        }))
当然，Alert也可以用来进行简单的表单输入，比如要添加textfiled来进行显示，可以使用如下代码：
    
     alertControllerSheet.addTextFieldWithConfigurationHandler { (textfield) -> Void in
            textfield.placeholder = "what's it?"
        }
而具体的如何抓取TextField的字符串数据来进行业务处理，方法不少，这里略过。

最后，我们在当前VC的接口中把Alert展示出来即可：
        
     self.presentViewController(alertControllerSheet, animated: true, completion: nil)

相比Alert而言**ActionSheet**更简单。他不能添加TextFiled，而用法和Alert一致，也是使用UIAlertController来显示，不同的是，UIAlertController的preferredStyle类型是ActionSheet而已：   

	let alertControllerSheet : UIAlertController = UIAlertController(title: "test", message: "ActionSheets", 
	preferredStyle: .ActionSheet)
	
	alertControllerSheet.addAction(UIAlertAction(title: "Confirm", style: .Default, handler: { (action: UIAlertAction!) -> Void in
            println("confirm")
        }))
        
        alertControllerSheet.addAction(UIAlertAction(title: "Cancel", style: .Cancel, handler: { (action: UIAlertAction!) -> Void in
            println("cancel")
        }))
        
        alertControllerSheet.addAction(UIAlertAction(title: "Delete", style: .Destructive, handler: { (action: UIAlertAction!) -> Void in
            println("Delete")
        }))
这样，ActionSheet和Alert就可以使用了。

####Popover
显然，Popover也被改造了。它在iOS8中作为一种present的类型，置于每一个VC之中。在iOS8中，即便是iPhone这样的设备，依然可以使用Popover，而不单单是iPad来是使用了。值得一提的是，对于默认的PopOver行为，在iPhone6+上，横屏竖屏由于Compact和Regular的不同，而显示行为不一致。有兴趣的童鞋可以简单写一个试试。  
这里说下Popover的玩法流程：RootVC present一个VC，这个VC的present类型是popover，即可。不同的情况是，你可以写一个专用的VC用于popover，或者某个VC在特定情况下是Popover，在另外的情况下是Present行为。这里我们只讲前者：  

    @IBAction func popOverTapped(sender: AnyObject) {
        
        let story : UIStoryboard = UIStoryboard(name: "Main", bundle: nil)
        let vc : PopoverViewController = story.instantiateViewControllerWithIdentifier("PopoverVCID") as PopoverViewController
        var popPresentationController : UIPopoverPresentationController? = vc.popoverPresentationController
        popPresentationController?.barButtonItem = UIBarButtonItem(customView: sender as UIView)
        popPresentationController?.permittedArrowDirections = .Any
        self.presentViewController(vc, animated: true, completion: nil)
        
    }
代码中我们准备Present一个类型为PopoverViewController的VC。在这里我们指定了弹出VC方向及其位置，而在另一个方面，在PopoverViewController中，我们约定，只要这个PopoverViewController被弹出来，就使用popOver。

    required init(coder aDecoder: NSCoder)
    {
        super.init(coder: aDecoder)
        
        //cancel button
        navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: UIBarButtonSystemItem.Done,
            target: self,
            action: "done:")
        
        // popover settings
        modalPresentationStyle = .Popover
        self.popoverPresentationController?.delegate = self;
    }
另外，如果我们要修改在iphone6+以下的popOver行为，可以实现当前VC自己的popoverPresentationController的代理，自己进行配置。

    // if you want to adaptive the popover show, this is useful
    func adaptivePresentationStyleForPresentationController(PC: UIPresentationController!) -> UIModalPresentationStyle{
        
        let size = PC.presentingViewController.view.bounds.size

        if (size.width > 320.0)
        {
            //landscape, popover
            return .None
        }
        else
        {
            // portrait, present
            return .FullScreen
        }
    }
上面代码就说如果是Land就popover，如果是Portrait就present。你可以看需求任意设置。


###自定义Present的理念和玩法
前面说Popover提到了每一个概念，在VC中，Present是有类型的。那么自定义Present类型就是Custom。下面我们展开看看。
实现一个自定义的Present需要至少4个对象来参与：

* Presenting的VC
* Presented的VC
* Present的Controller
* PresentAnimation的Controller

Presented的VC用于展示Present的具体内容。Presenting的VC用于展示PresentedVC。Present的Controller用于对Custom展示的开始、结束，消失的开始、结束进行回调控制。而PresentAnimation的Controller用于控制具体展示出来的动画时长、动画细节等等。
下面结合代码说下，在Presenting的VC中展示CustomPresentViewController，此时，CustomPresentViewController就是PresentedVC。

    @IBAction func customPresentTapped(sender: AnyObject){
        
        let story : UIStoryboard = UIStoryboard(name: "Main", bundle: nil)
        let vc : CustomPresentViewController = story.instantiateViewControllerWithIdentifier("CustomPresentVCID") as CustomPresentViewController
        self.presentViewController(vc, animated: true, completion: nil)
    }
而在CustomPresentVC中，标明他的动画是自定义的，并指定转场代理：

    func statementInit() {
        self.modalPresentationStyle = .Custom
        self.transitioningDelegate = self
    }

而后分别实现，以下三个回调：

	func presentationControllerForPresentedViewController(presented: UIViewController!, presentingViewController presenting: UIViewController!, sourceViewController source: UIViewController!) -> UIPresentationController!
	
	func animationControllerForPresentedController(presented: UIViewController!, presentingController presenting: UIViewController!, sourceController source: UIViewController!) -> UIViewControllerAnimatedTransitioning!
	
	func animationControllerForDismissedController(dismissed: UIViewController!) -> UIViewControllerAnimatedTransitioning!
	
第一个回调用于指定Present的Controller，后两个回调分别用于指定展示和消失状况下的PresentAnimation的Controller对象，用于动画控制。

在Present的Controller这个对象中，它本身继承于UIPresentationController。只要在其中的：

	override func presentationTransitionWillBegin()
	override func presentationTransitionDidEnd(completed: Bool)
	override func dismissalTransitionWillBegin()
	override func dismissalTransitionDidEnd(completed: Bool)
这几个接口进行实现，就可以完成对转场关键节点的控制。而也可以通过如下代码，对PresentedVC的大小，进行控制：

	override func frameOfPresentedViewInContainerView() -> CGRect

而对于动画控制对象而言，它只需要实现动画控制协议UIViewControllerAnimatedTransitioning即可接入动画接口，完成对具体转场动画的描述了。其内部的动画结构和iOS7的Navigation自定义转场的动画抽象方式是一样一样的，都有ContainerView、动画时间、转场动画接口等：
	
	func transitionDuration(transitionContext: UIViewControllerContextTransitioning) -> NSTimeInterval
	func animateTransition(transitionContext: UIViewControllerContextTransitioning)

这样就可以对自定义转场进行控制了。



###搜索和UISearchController
在iOS8中的搜索也被更新了，使用UISearchController来完成搜索框和具体内容展示VC(ResultViewController)的衔接。这个后续结合代码详述。


###结语
这里通篇都没有提到旧版本iOS实现Popover、Action、Alert对应的类名，这里其实意在忘记。记住新的就可以了。
其中的代码除了Search以外，都有demo，可以直接到<a href = https://github.com/chesterlee/LearnPresentControl>LearnPresentControl</a>中查看。