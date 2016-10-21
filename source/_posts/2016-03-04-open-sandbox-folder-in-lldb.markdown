---
layout: post
title: "在lldb中一键打开模拟器sandbox路径"
date: 2016-03-04 10:39:55 +0800
comments: true
categories: ['Xcode', 'iOS', 'macOS', 'lldb']
---

打开~/.lldbinit，在里面加入一行：  
```bash
command alias sb script from subprocess import call; call(["open", '{0:s}'.format(lldb.frame.EvaluateExpression("NSHomeDirectory()")).split("\"")[1]]);
```

然后中断时，在lldb里打sb回车，就能打开模拟器当前运行的app的沙箱路径了。  

<!--more-->

### 原理

lldb中允许我们执行python代码，中断时使用script命令即可进入python模式：

```bash
(lldb) script
```

在python中我们可以通过lldb.frame操纵lldb，比如执行表达式，会返回一个lldb.SBValue对象：  

```python
//python code
lldb.frame.EvaluateExpression("NSHomeDirectory()")

//return
<lldb.SBValue; proxy of <Swig Object of type 'lldb::SBValue *' at 0x13ded2bd0> >
```

然后我们格式化成python字符串，会返回SBValue的description，类型是pystring：  

```python
//python code
'{0:s}'.format(lldb.frame.EvaluateExpression("NSHomeDirectory()"))

//return
'(NSPathStore2 *) $1 = 0x00007fdbc4c1a6d0 @"/Users/openthread/Library/Developer/CoreSimulator/Devices/5D48B08A-44C2-4AC5-B52D-725150EA1091/data/Containers/Data/Application/8331B6D2-FC68-4523-8093-94FD9487FF74"'
```

我们用引号split这个pystring，取数组中第一个元素，就是path：

```python
//python code
'{0:s}'.format(lldb.frame.EvaluateExpression("NSHomeDirectory()")).split("\"")[1]

//return
'/Users/openthread/Library/Developer/CoreSimulator/Devices/5D48B08A-44C2-4AC5-B52D-725150EA1091/data/Containers/Data/Application/8331B6D2-FC68-4523-8093-94FD9487FF74'
```

然后在python中使用call调用shell的open，就可以打开沙箱路径了：
```python  
//python code
from subprocess import call; call(["open", pathOfSandbox)
```

这些命令可以连起来在lldb中执行：  
```
(lldb) script from subprocess import call; call(["open", '{0:s}'.format(lldb.frame.EvaluateExpression("NSHomeDirectory()")).split("\"")[1]])
```

然而实在太长了，于是我们在lldb中加了个alias：  

```bash
(lldb) command alias sb script from subprocess import call; call(["open", '{0:s}'.format(lldb.frame.EvaluateExpression("NSHomeDirectory()")).split("\"")[1]])
```

### 扩展

lldb和python可以相互调用，给lldb增加了非常强的扩展性。  
facebook做了[chisel](https://github.com/facebook/chisel)，内含各种各样的扩展功能。  

需要自己开发新扩展，可以参考lldb关于文档：  
[Package lldb :: Class SBFrame](http://lldb.llvm.org/python_reference/lldb.SBFrame-class.html)  
[lldb::SBValue Class Reference](http://lldb.llvm.org/cpp_reference/html/classlldb_1_1SBValue.html)

Over
