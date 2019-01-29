---
layout: post
title: "python talib 的一些基础用法"
date: 2018-12-31 14:27:06 +0800
comments: true
categories: [python, ta-lib, fin-tech]
---

首先还是导入一些业界标准库：

```
import pandas as pd
import numpy as np
import talib as ta
```


### 计算RSI
```
close = np.array(bars.close)
print(ta.RSI(close))  # 默认15根bars
```

### 计算MA

```
ta.MA(close)  # 默认 30 根 bars，即 MA30
```

### 设置计算用的时长，比如计算 MA5

```
ta.MA(close, timeperiod=5)
```

### 计算 EMA11 和 EMA22

```
ta.EMA(close, timeperiod=11)
ta.EMA(close, timeperiod=22)
```


### 全部指标和方法列表

[http://mrjbq7.github.io/ta-lib/funcs.html](http://mrjbq7.github.io/ta-lib/funcs.html)