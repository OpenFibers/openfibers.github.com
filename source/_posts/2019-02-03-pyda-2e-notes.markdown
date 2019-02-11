---
layout: post
title: "利用pandas进行数据分析第二版 笔记"
date: 2019-02-03 16:10:22 +0800
comments: true
categories: [python, numpy, fin-tech, AI]
---

先放书的链接： [https://www.amazon.cn/dp/B07G2PK49V](https://www.amazon.cn/dp/B07G2PK49V)

<!--more-->

# 第二章 Python 语法基础

### 强类型/动态性

使用`type()`判断类型：  

```python
In [12]: a = 5 
In [13]: type(a)
Out[13]: int
```

使用`isinstance()`检查实例是否为某种类型：  

```python
In [23]: a = 5; b = 4.5 
In [24]: isinstance(a, (int, float)) 
Out[24]: True 
In [25]: isinstance(b, (int, float)) 
Out[25]: True
```

使用`getattr()`反射对象的属性：  

```python
In [1]: a = 'foo'
In [27]: getattr(a, 'split') 
Out[27]: <function str.split>
```

除此之外可以使用`hasattr`和`setattr`判断是否有属性和对属性赋值。  

### 鸭子类型

经常地，你可能不关心对象的类型，只关心对象是否有某些方法或用途。这通常被称为“鸭子类型”，来自“走起来像鸭子、叫起来像鸭子，那么它就是鸭子”的说法。例如，你可以通过验证一个对象是否遵循迭代协议，判断它是可迭代的。对于许多对象，这意味着它有一个 `__iter__` 魔术方法，其它更好的判断方法是使用 iter 函数：

用这个功能编写可以接受多种输入类型的函数。常见的例子是编写一个函数可以接受任意类型的序列（list、tuple、ndarray）或是迭代器。你可先检验对象是否是列表（或是NUmPy数组），如果不是的话，将其转变成列表：

```python
if not isinstance(x, list) and isiterable(x):
     x = list(x)
```

### 禁止字符串内转义序列

在字符串的单引号前面加r，可以避免字符串内的\产生转义：  

```python
In [69]: s = r'this\has\no\special\characters' 
In [70]: s Out[70]: 'this\\has\\no\\special\\characters'
```

### 字符串格式化

```python
In [74]: template = '{0:.2f} {1:s} are worth US${2:d}'
In [75]: template.format(4.5560, 'Argentine Pesos', 1) 
Out[75]: '4.56 Argentine Pesos are worth US$1'
```

在这个字符串中:  
{0:.2f} 表示格式化第一个参数为带有两位小数的浮点数。  
{1:s} 表示格式化第二个参数为字符串。  
{2:d} 表示格式化第三个参数为一个整数。 

### 字符串编码

使用`encode('utf-8')`将其转换成 unicode 字节码：  

```python
In [76]: val = "español" 
In [78]: val_utf8 = val.encode('utf-8') 
In [79]: val_utf8 
Out[79]: b'espa\xc3\xb1ol' 

In [80]: type(val_utf8) 
Out[80]: bytes
```

使用`decode('utf-8')`将 unicode 字节码转回 str：  

```python
In [81]: val_utf8.decode('utf-8') 
Out[81]: 'español'
```

可以在引号前面加b，表示直接生成unicode字节码：  

```python
In [85]: bytes_val = b'this is bytes' 
In [86]: bytes_val 
Out[86]: b'this is bytes' 
In [87]: decoded = bytes_val.decode('utf8') 
In [88]: decoded  # this is str (Unicode) now 
Out[88]: 'this is bytes'
```

### Control Flow

#### 连续比较：  

```python
In [120]: 4 > 3 > 2 > 1 
Out[120]: True
```

#### range:  


range函数返回一个迭代器，它产生一个均匀分布的整数序列，产生的序列不包括终点。虽然range可以产生任意大的数，但任意时刻耗用的内存却很小。： 

```python
In [122]: range(10) 
Out[122]: range(0, 10) 
In [123]: list(range(10)) 
Out[123]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] 
```

range的三个参数是（起点，终点，步进）： 

```python
In [124]: list(range(0, 20, 2)) 
Out[124]: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18] 
In [125]: list(range(5, 0, -1)) 
Out[125]: [5, 4, 3, 2, 1]
```

#### 三元表达式：  

```python
value = true-expr if condition else false-expr
```


# 第三章 Python的数据结构、函数和文件

### 元组

元组是一个固定长度，不可改变的Python序列对象。创建元组的最简单方式，是用逗号分隔一列值： 

```python
In [1]: tup = 4, 5, 6 
In [2]: tup 
Out[2]: (4, 5, 6)
```

用  tuple 可以将任意序列或迭代器转换成元组，元组可以使用下标访问对象： 

```python
In [5]: tuple([4, 0, 2]) 
Out[5]: (4, 0, 2) 
In [6]: tup = tuple('string') 
In [7]: tup 
Out[7]: ('s', 't', 'r', 'i', 'n', 'g')

In [8]: tup[0] 
Out[8]: 's'
```

可以用加号运算符将元组串联起来： 

```python
In [13]: (4, None, 'foo') + (6, 0) + ('bar',)
Out[13]: (4, None, 'foo', 6, 0, 'bar')
```

元组乘以一个整数，像列表一样，会将几个元组的复制串联起来；对象本身并没有被复制，只是引用了它。： 

```python
In [14]: ('foo', 'bar') * 4 
Out[14]: ('foo', 'bar', 'foo', 'bar', 'foo', 'bar', 'foo', 'bar') 
```

### 拆分元组

如果你想将元组赋值给类似元组的变量，Python会试图拆分等号右边的值： 

```python
In [15]: tup = (4, 5, 6) 
In [16]: a, b, c = tup 
In [17]: b 
Out[17]: 5 
```

即使含有元组的元组也会被拆分： 
```python
In [18]: tup = 4, 5, (6, 7) 
In [19]: a, b, (c, d) = tup 
In [20]: d 
Out[20]: 7
```

用这个功能，swap工作变得很简单：  

```python
In [21]: a, b = 1, 2 
In [22]: a 
Out[22]: 1 
In [23]: b 
Out[23]: 2 
In [24]: b, a = a, b 
In [25]: a 
Out[25]: 2 
In [26]: b 
Out[26]: 1
```

或是从元组中摘取元素：  

```
In [29]: values = 1, 2, 3, 4, 5 In [30]: a, b, *rest = values 
In [31]: a, b 
Out[31]: (1, 2) 
In [32]: rest 
Out[32]: [3, 4, 5]
```

或者：

```
In [33]: a, b, *_ = values
```

### 列表

































