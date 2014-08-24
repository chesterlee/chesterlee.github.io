---
layout: post
title: "谈谈iOS中的NSURLProtocol"
date: 2014-08-15 10:23:37 +0800
comments: true
categories: iOS
keywords: NSURLProtocol
description: 
---
NSURLProtocol是比较"小众"，但是能量极大的一个东东。在NSHipster中matt对其有详细介绍。

  
它就像在少林寺里，武功最高的，就是在一旁扫落叶的僧人，默默无闻，你一旦发现他，各种绝学就来了。  

-_-扯远了，我们先看看它是什么，然后看看它能干什么。  

1、它的概念：

它是一个抽象类，我们可以通过继承它来定义新的或已存在的URL加载行为。

2、它的功效：

它可以做各种hack的事情：

* 拦截获取图片的Request，然后自行加载自己的图片。

* 自己拦截特定Request，然后自己做签名，header修改。

* 过滤Response、Request中的敏感信息，当然也可以加上敏感信息-0-。

* 支持NSURLConnection的callback协议，可以对上层完全透明。


3、怎么用？

只需要做两件事情：

a、子类化NSURLProtocol类，并注册。APP启动后，但做网络请求之前，就可以干这事情了。后续iOS必然是单例化你的类，然后吃各种callback了。

b、在自己的类的回调上，做一些hack的事情。

4、最简单的sample...-_-

``` objectivec DeriveNSURLProtocol:NSURLProtocol 
    
    + (BOOL)canInitWithRequest:(NSURLRequest*)theRequest
    {    
    	 //自己选择要截获怎样的request，如果要截获就返回YES
         if ([theRequest.URL.scheme caseInsensitiveCompare:@"testapp"] == NSOrderedSame) 
         {        
           return YES;    
         }    
         return NO;
    }
    
    + (NSURLRequest*)canonicalRequestForRequest:(NSURLRequest*)theRequest
    {   
    	// 正规化自己的request，因为request会用来找对应的cache，默认需要实现该方法，通常不需要修改。
    	return theRequest;
    }
    
    - (void)startLoading
    {    
      //这里就开始加载了，模拟NSURLConnection的callback流程，依次call client的方法来模拟Connection的callback次序
      NSLog(@"%@", request.URL);    
      NSURLResponse *response = [[NSURLResponse alloc] initWithURL:[request URL] MIMEType:@"image/png"  expectedContentLength:-1 textEncodingName:nil];    
      NSString *imagePath = [[NSBundle mainBundle] pathForResource:@"myImage" ofType:@"png"];
      NSData *data = [NSData dataWithContentsOfFile:imagePath];
      [[self client] URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed]; 
      [[self client] URLProtocol:self didLoadData:data];
      [[self client] URLProtocolDidFinishLoading:self];
      [response release];
    }
    
    - (void)stopLoading
    {    
      //加载结束
      NSLog(@"load over!");
    }

    
```

5、更多详细介绍可以参看matt的http://nshipster.com/nsurlprotocol/
