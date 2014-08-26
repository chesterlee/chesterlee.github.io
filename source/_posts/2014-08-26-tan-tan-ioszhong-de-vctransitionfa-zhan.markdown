---
layout: post
title: "谈谈iOS中的VCTransition发展"
date: 2014-08-26 21:19:13 +0800
comments: true
categories: iOS
keywords: iOS Transition
description: 
---
##前言
iOS的VCTransition一直是在跟着iOS不断变化着的，最大的感觉就是不断地解耦，动画之间VC的关联变得薄弱，或者都抽象到一个统一的位置，进行处理。下面都说下：  

####Navigation、TabBar等切换  
  这个是最早(传统的)的VC风格了。比如使用如下代码： 

      -(IBAction)PushA:(id)sender 
      {  
         UIStoryboard *story = [UIStoryboard storyboardWithName:@"Main" bundle:nil];
         AViewController *aVC = [story instantiateViewControllerWithIdentifier:@"AVCID"];  
         [self.navigationController pushViewController:aVC animated:YES]; 
      }
  这个方法是使用Navigation来管理VCs的一个Transition方式。比较简单，直接。但是耦合性比较大，在VC中必须知道自己要跳转到对应到VC，而且动画要写到对应的VC中，如果动画复杂，会很容易造成VC臃肿起来。
  
  
#### VC容器的Transition  
由于iOS5.0后续版本新增了自定义的VC容器，那么从在容器中进行child vc之间到转场，会有专门的方法。比如如下代码:  

      [self transitionFromViewController:self.goodsCollectionVC
                      toViewController:self.mainPageVC
                              duration:SWITCH_VCS_TIME
                              options:UIViewAnimationOptionTransitionCrossDissolve animations:^{
                      // do animation things
    } completion:^(BOOL finished) {
        // do animation completion things
    }];
        

这个方法在容器中切换subVC比较方便，但是只实现了简单的Transition动画，并且依然是有耦合的。因为Container做动画的切换VC的时候，如果动画非常复杂，依然会有动画无法复用的状况。

#### StoryBoard和Segue  
  这个是iOS5时提供的转场方式：通过在Storyboard中或代码中定义Push、Modal或 Custom segue，定义一个SegueID，然后需要做Transition的时候，去Perform一个Segue就可以了，代码如下：

      [self performSegueWithIdentifier:@"PushSegueID" sender:self];  
           
在执行动画之前，自己的VC就要做些preparation可以在如下代码中调用

      -(void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender NS_AVAILABLE_IOS(5_0){} 
  

  特别需要说一下，如果动画处理复杂的，通过Custom Segue，就可以把动画挪到自定义segue中，降低sourceVC的代码量，动画也可以解耦甚至抽象。主要步骤就有两个：  
  - 在Storybaord或代码中实例化自定义的segue。
  - 自定义Segue类复写perform方法。  
  关键代码如下：  

       -(void)perform
       {
    	  id sourceVC = self.sourceViewController;
    	  id destVC = self.destinationViewController;
    	  //Do animation  
    	}
   
 segue 最关键的两个property，一个是sourceViewController，一个是destinationViewController，就实现一个动画而言，这两个变量就够了。
#### iOS7的Transition  
iOS7提供了新的transition方式。默认提供了Present和Push的抽象动画，新增了percent的手势交互动画。例如，如果要自定义navigation push的动画就可以实现一个支持UIViewControllerAnimatedTransitioning的对象。在这个transition对象中，来设计自己的动画。和segue类似，代码可见： 

       -(void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
       {
    		UIViewController *destController =  [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];  
    		UIViewController *fromController =  [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    		UIView *containerView = [transitionContext containerView];  
    		[containerView addSubview:destController.view];
       }

和Segue相同的是，都可以通过对象来获取Source和DestinationVC。不同的是，iOS7新提供的Transition提供了ContainerView，来做动画交互的容器。  
对于外部而言，在代理中制定对应的Custom Transition对象即可使用他们了，可定制化更高：  
比如在VC Controller中实例化对应的代理方法：

       -(id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC
                                             {
    											if (someCondition) 
    											{
        										  _naviTransitioning.operation = operation;
        										  return  _naviTransitioning;
    											}
    										    return nil;
    										  }
  
至于iOS7的UIPercentDrivenInteractiveTransition，这里不展开讲了，具体可以搜搜相关文档。  
经过Transition的不断强化，iOS7新增的Transition，大大增强了动画的复用性，降低了耦合度，增加了交互性，给编码人员提供更多福利。