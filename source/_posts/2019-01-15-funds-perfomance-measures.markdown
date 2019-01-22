---
layout: post
title: "评价基金性能的常用指标"
date: 2019-01-15 12:19:56 +0800
comments: true
categories: 
---


### 最大回撤（Max Draw Down）

最大回撤为：在选定周期内任一历史时点往后推，产品净值走到最低点时的收益率回撤幅度的最大值。  

其计算的时间复杂度是O(n)的。有些O(n^2)的写法比较业余。

python:
```python
def get_max_draw_down(in_list: list):
    dd_list = []
    max_so_far = in_list[0]
    for i in range(len(in_list)):
        if in_list[i] > max_so_far:
            dd = 0
            dd_list.append(dd)
            max_so_far = in_list[i]
        else:
            current = in_list[i]
            dd = (max_so_far - current) / max_so_far
            dd_list.append(dd)
    mdd = max(dd_list)
    return mdd
```

### 夏普比率(Sharpe Ratio)

夏普比率：评价单位承担的风险可以取得多少正向收益。在同种时间跨度下，越高越好。  
其计算方法为：平均收益率超过无风险收益率的部分除以收益率的标准差。  
缺陷：由于标准差是正直，所以无法区分上行风险和下行风险。  
优点：国内外知名度高。  

```
import numpy

def sharpe_ratio(returns, risk_free_rate=0.0):
    mean = numpy.mean(returns)
    return (mean - risk_free_rate) / numpy.std(returns)
```


<!--more-->

### 索提诺比率(Sortino Ratio)

索提诺率：计算方法和夏普率相似，但是用低于目标收益率的收益率或亏损时的收益率所计算出的“下行标准差”取代了正常的标准差。和夏普比率一样，在同种时间跨度下，越高越好。  


```
import numpy
import math

def lpm(returns, threshold, order):
    threshold_array = numpy.empty(len(returns))
    threshold_array.fill(threshold)
    diff = threshold_array - returns
    diff = diff.clip(min=0)
    return numpy.sum(diff ** order) / len(returns)

def sortino_ratio(returns, risk_free_rate=0.0, target=0):
    mean = numpy.mean(returns)
    return (mean - risk_free_rate) / math.sqrt(lpm(returns, target, 2))
```


更多参数 python 计算，参考：  

[Measures of Risk-adjusted Return](http://www.turingfinance.com/computational-investing-with-python-week-one/)

Over