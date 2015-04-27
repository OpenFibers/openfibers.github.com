---
layout: post
title: "transparent window in osx"
date: 2015-04-27 19:22:15 +0800
comments: true
categories: ["OSX"]
published: false
---


10.7太老不予支持. 从10.8开始讲起.

##10.8

```objective-c
- (void)applyGaussianBlurWithRadius:(CGFloat)radius
{
	[self addCGSFilterWithName:@"CIZoomBlur"
				   withOptions:@{@"inputRadius": @(radius)}
				backgroundOnly:YES];
}

- (void)addCGSFilterWithName:(NSString *)filterName
				  withOptions:(NSDictionary *)filterOptions
			   backgroundOnly:(BOOL)backgroundOnly
{
	CGSConnection conn = _CGSDefaultConnection();
	
	if (conn)
	{
		CGSWindowFilterRef filter = NULL;
        
		//Create a CoreImage gaussian blur filter.
		CGSNewCIFilterByName(conn, (__bridge CFStringRef)filterName, &filter);
		
		if (filter)
		{
			CGSWindowID windowNumber = (CGSWindowID)self.windowNumber;
			int compositingType = (int)backgroundOnly;
			
			CGSSetCIFilterValuesFromDictionary(conn, filter, (__bridge CFDictionaryRef)filterOptions);
			
			CGSAddWindowFilter(conn, windowNumber, filter, compositingType);
			
			//Clean up after ourselves.
			CGSReleaseCIFilter(conn, filter);
		}
	}
}

```

##10.9

```
- (void)setWindowBackgroundBlurRadius:(int)radius
{
    CGSConnection connection = CGSDefaultConnectionForThread();
    CGSSetWindowBackgroundBlurRadius(connection, [self windowNumber], radius);
}
```

##10.10