---
layout: post
title: "OTHTTPRequest v2.0 release"
date: 2016-02-26 14:49:47 +0800
comments: true
categories: ['iOS', 'Network', 'OTHTTPRequest']
---

这几天重写了OTHTTPRequest的大部分代码，发布了2.0版。  
改动主要是上传文件功能，以前做的比较粗，是整个文件读到内存再上传的，不能支撑大文件上传，现在multipart/form请求全部改为流上传。此外增加了上传速度计算的功能。  

最早做这个的原因是13年5月份做网易云音乐的下载性能优化，ASIHTTPRequest下载文件占用CPU时间太长，而当时iPhone4还有一定占有量，做了这个库来提升下载性能。  
这次发布2.0，修改上传功能，是因为用ASIHTTPRequest做大文件上传时，进度回调有bug，尝试修结果没修成，于是花了点时间做了这个版本。  

和1.0一样，整个框架基于NSURLRequest和NSURLConnection。  

[Github链接](https://github.com/openfibers/othttprequest)  

<!--more-->

用法和大部分网络框架一样，都是很简洁的。  

#### Get request

```objective-c
OTHTTPRequest *request = [[OTHTTPRequest alloc] initWithURL:[NSURL URLWithString:@"https://www.google.com"]];
request.delegate = self;
request.getParams = @{@"gws_rd": @"ssl"};
[request start];
```

#### Post request

```objective-c
OTHTTPRequest *request = [[OTHTTPRequest alloc] initWithURL:[NSURL URLWithString:@"https://www.google.com"]];
request.delegate = self;
request.postParams = @{@"gws_rd": @"ssl"};
[request start];
```

#### Upload request
```objective-c
OTHTTPRequest *request = [[OTHTTPRequest alloc] initWithURL:[NSURL URLWithString:@"https://www.google.com"]];
request.delegate = self;
request.postParams = @{@"gws_rd": @"ssl"};
[request addFileForKey:@"file" filePath:filePath fileName:@"Default.png" MIMEType:nil];
[request start];
```

#### Download request
```objective-c
OTHTTPDownloadRequest *request = [[OTHTTPDownloadRequest alloc] initWithURL:downloadURLString
                                                cacheFile:cacheFilePath
                                         finishedFilePath:finishedFilePath];
request.delegate = self;
[request start];
```

Over
