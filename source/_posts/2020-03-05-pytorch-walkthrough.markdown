---
layout: post
title: "Pytorch Walkthrough on macOS"
date: 2020-03-05 14:25:58 +0800
comments: true
categories: 
---


# Install

```bash
pip3 install torch
```

安装成功后发现报错：

```
>>> import torch
toTraceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/torch/__init__.py", line 97, in <module>
    from torch._C import *
ImportError: dlopen(/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/torch/_C.cpython-38-darwin.so, 9): Library not loaded: @rpath/libc++.1.dylib
  Referenced from: /Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/torch/_C.cpython-38-darwin.so
  Reason: image not found
```

`libc++.1.dylib` 在 `/usr/lib` 下，使用 `install_name_tool` 解决:  

```
install_name_tool -add_rpath /usr/lib /Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/site-packages/torch/_C.cpython-38-darwin.so
```