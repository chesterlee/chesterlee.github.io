---
layout: post
title: "谈谈iOS中的HTTP缓存策略"
date: 2014-08-24 09:03:05 +0800
comments: true
categories: iOS
keywords: iOS,HTTPCache
description: 
---
iOS中的HTTP URL Cache机制是看上较为简单，但是暗含一些机制的东西。这里将它细化下来。让自己也加深下相应的记忆。  

首先，说下概念，HTTP URLCache是HTTP协议中的一部分，用于缓存Response，以达到减轻服务器负担的目的。在iOS中的URL Loading System中，提供了名字叫NSURLCache的类来设置URL缓存的配置信息。    

在iOS中的URL缓存容器是共享的，因而我们可以自己继承NSURLCache来配置自己的cache，自定义化它的内存大小，磁盘大小甚至过期时间等等，最后设置为share cache即可。  
  
HTTP缓存在iOS中需要，配合着NSURLRequest的缓存策略来实现的。比如，如果指定了一个Request的缓存策略，那么对应Request就会按既定策略进行处理。策略常用如下：  

*  **NSURLRequestUseProtocolCachePolicy**。一个Request的默认缓存策略。他是最为隐晦的，我们最后说。
*  **NSURLRequestReturnCacheDataElseLoad**。如果在之前的网络请求中，我们获取了Cache的Response，那么本次请求同样的接口，就直接从Cache中去抓。反之，从服务器上去获取，并在本地cache起来。(这里的问题是，如果cache存在，就一直拉不到服务器数据，所以用这个机制，有它的弊端，当然也要看具体的需求情况。可以通过自定义自类化NSURLCache来自定义cache time，但也仅仅适用于Data改变不大的项目中。)
*  **NSURLRequestReturnCacheDataDontLoad**。只从缓存中获取cached response而不从服务器获取。实属离线版本。  

好，这里展开说一下最为复杂的ProtocolCachePolicy机制：  
1、如果一个Request的NSCachedURLResponse不存在，就去请求网络。  
2、如果一个Request的NSCachedURLResponse存在，就去检查response去决定是否需要刷新。检查Response header的Cache-Control字段是否含有must-revalidated字段(HTTP1.1所带)。  
3、如果包含must-revalidated字段，就通过HEAD方法请求服务器，判断Response头是否有更新，如果有，则去拉数据下来。如果没有，则使用Cache资源。  
4、如果不包含must-revalidated字段，就看看Cache-Control是否包含其他字段，比如max-age等等看看是否过期，如果过期，继续用HEAD去检查Response头，是否为最新数据，如果有则去请求服务器，反之取cache。如果没有过期，则直接取cache。  

*  注意：由于HEAD是URL Loading System自身机制，由于没有抓包，所以没去验证。HEAD方法默认使用可以参见水果文档中的Cache Use Semantics for the HTTP Protocol。  

  
然后说下通用HTTP的缓存策略：  
  
  HTTP缓存策略中，我们从服务器获取Response后，可以找到(如果有)Response中包含Etag或则Last-Modified字段。当我们做第二次重复请求的时候，可以从CachedURLResponse取出来，把相应字段拼接在HTTPRequestHeader中（例如，IMS，If-Modified-Since配合Last_Modified），然后发送请求，服务端收到后，如果客户端的资源是最新的，那么就会返回304为Response，而不返回任何内容。反之，如果客户端资源落后了，则直接返回200，并返回Data给客户端。

最后谈谈修改NSCachedURLResponse：  
在方法-(NSCachedURLResponse*)connection:(NSURLConnection*)connectionwillCacheResponse:(NSCachedURLResponse*)cachedResponse中，我们可以对即将缓存的Response进行修改，以达到自己的需求。