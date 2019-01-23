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

评价单位承担的风险可以取得多少正向收益。在同种时间跨度下，越高越好。  
#### 计算方法：
* 平均收益率超过无风险收益率的部分除以收益率的标准差。  
* 如果通过高频夏普率（比如1d）推测低频夏普率(一般是年)，若每年有255个交易日，一般是再乘上根号下255。像币圈高频夏普率（4h）推导年，则是再乘上根号下 365 * 24 / 4。  
#### 缺陷：
* 由于标准差是正值，所以无法区分上行风险和下行风险。  
#### 优点：
* 国内外知名度高。   

附两个知乎的讲解链接：  
[夏普率越高越好吗？ - 石川的回答 - 知乎](https://www.zhihu.com/question/264210987/answer/421333614)  
[求问基金风险指标的计算：夏普比率，索提诺比率，阿尔法系数等？ - 财小鲸之秦岭的回答 - 知乎](https://www.zhihu.com/question/38316057/answer/75848881)  


```
import numpy

def sharpe_ratio(returns, risk_free_rate=0.0):
    mean = numpy.mean(returns)
    return (mean - risk_free_rate) / numpy.std(returns)
```

其中 returns 为每条 ohlc 的变化率，比如 [0.02, 0.03, -0.03, ...]  
risk_free_rate 为每条 ohlc 的无风险收益率。比如okex余币宝年化1%，传入的是4小时returns，这里应该传入 1% / (365 * 24 / 4)  

可以通过历史净值列表（equity）生成 returns：

```
def returns_from_equity(in_equity: list):
    returns_np = numpy.array(in_equity)
    returns_np = returns_np[1:] / returns_np[:-1]
    returns_np = returns_np - 1
    return returns_np
```

然后把返回值代入下一步计算。


<!--more-->

### 索提诺比率(Sortino Ratio)

计算方法和夏普率相似，但是用低于目标收益率的收益率或亏损时的收益率所计算出的“下行标准差”取代了正常的标准差。和夏普比率一样，在同种时间跨度下，越高越好。  


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


