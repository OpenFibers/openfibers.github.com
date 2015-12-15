---
layout: post
title: "iOS8的UITextView输入光标显示不全的hack"
date: 2015-11-30 14:47:21 +0800
comments: true
categories: ['iOS', 'UIKit']
---

在iOS8及以下版本的系统上，在定高的UITextView中，输入内容超过Text View高度后，输入光标有时会在Text View的底部显示不全，如[how-to-make-a-uitextview-scroll-while-typing-editing](http://stackoverflow.com/questions/18070537/how-to-make-a-uitextview-scroll-while-typing-editing)中截图所描述。  

尝试了各种方案，挑选了一种体验较好的。在`textViewDidChanged:`中，检测到正在编辑的区域在文字最下行，无动画滚动到结尾：  

```objective-c
- (void)textViewDidChange:(UITextView *)textView
{
    //hack for iOS8
    if (isLessThanIOS9)//in iOS9 Apple has already fixed this bug
    {
        CGRect line = [textView caretRectForPosition:
                       textView.selectedTextRange.start];
        CGFloat overflow = line.origin.y + line.size.height
        - (textView.contentOffset.y + textView.bounds.size.height
           - textView.contentInset.bottom - textView.contentInset.top);
        if (overflow > 0)//If at the bottom of text view
        {
            //disable animation. Otherwise, when a input confirm scroll animation is doing, input new text, animation will re-do from animation beginning, which looks strange.
            [UIView setAnimationsEnabled:NO];
            
            //scroll to text end
            [textView scrollRangeToVisible:NSMakeRange([textView.text length], 0)];
            [UIView setAnimationsEnabled:YES];
        }
    }
}
```