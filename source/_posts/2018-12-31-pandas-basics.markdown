---
layout: post
title: "pandas的一些基础用法"
date: 2018-12-31 13:53:58 +0800
comments: true
categories: 
---

首先我们导入一些‘业界标准’库

```
import pandas as pd
import numpy as np
import talib as ta
import matplotlib.pyplot as plt
```

### 追加数据

```
df = df.append(series, ignore_index=True)
```

### 获取名为 close 的 column，作为 series

```
series = df.close
或者
series = df['close']
```

<!--more-->

### series 转 np array

```
np_array = np.array(series)
或
np_array = np.array(df.close)
或
np_array = np.array(df['close'])
```

### 结合 talib 计算

```
ta.RSI(np.array(bars.close))
```

### 可视化

```
df = pd.DataFrame({
    'close': ... ,
    'open': ... ,
    'high': ... ,
    'low': ... ,
    'timestamp', ...
})

# figure 1
df.plot(x='timestamp', y=['close', 'high', 'low'])  # 展示 close high low 三个属性

# figure 2
df.plot(x='timestamp')  # 展示全部属性

# 展示上面两幅绘制过的图
plt.show()
```


Over




