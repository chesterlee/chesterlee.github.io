---
layout: post
title: "谈谈在UIWebView中的内存控制"
date: 2014-10-07 21:05:05 +0800
comments: true
categories: iOS
keywords: UIWebView内存管理, iOS, Memory Management
description: 
---
本文是一个webView内存管理的trick，可以直接拿来用。  
由于UIWebView的内容加载不受程序员直接控制，所以在UIWebView加载一些图片较多的网页时，如果图片较多，很容易就在网页读取阶段出现memory warning，而且当你把webview对象干掉时，内存依然不减。如果不处理的话，程序就会被kill掉。如何处理呢，下面是几个tip可以让君尝试：  

1、在每一次UIWebView读取页面结束后(Delegate)，调用UserDefault关闭缓存。代码：

       - (void)webViewDidFinishLoad:(UIWebView *)webView 
       {
         [[NSUserDefaults standardUserDefaults] setInteger:0 forKey:@"WebKitCacheModelPreferenceKey"];  
       } 

注意，不必担心这个WebKitCacheModelPreferenceKey会一直保持为0。因为每一次UIWebView加载页面的时候，都会把此值设置为1。

2、在离开webview的controller时，使用WebView加载nil的URL，并清理webview。  

    -(void)viewDidDisappear:(BOOL)animated
    {
      [super viewDidDisappear:animated];
      [self.webView loadRequest:nil];
      [self.webView removeFromSuperview];
      self.webView = nil;
      self.webView.delegate = nil;
      [self.webView stopLoading];
    } 

3、在收到系统警告的时候，清理NSURLCache的CachedResponse。

    -(void)didReceiveMemoryWarning
    {
      [super didReceiveMemoryWarning];
      [self.navigationController setNavigationBarHidden:NO];
    } 
trick结束:)。   
本文参考：  
1、<a href = http://twobitlabs.com/2012/01/ios-ipad-iphone-nsurlcache-uiwebview-memory-utilization/>Reduce iOS memory utilization by taming NSURLCache</a>  
2、<a href = http://www.codercowboy.com/code-uiwebview-memory-leak-prevention/>UIWebView Memory Leak Prevention </a>  
3、<a href = http://blog.techno-barje.fr//post/2010/10/04/UIWebView-secrets-part1-memory-leaks-on-xmlhttprequest/>UIWebView Secrets - Part1 - Memory Leaks on Xmlhttprequest </a>


