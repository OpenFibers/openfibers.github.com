---
layout: post
title: "Demark Sequential Using Numpy"
date: 2019-07-22 15:35:07 +0800
comments: true
categories: 
---

```python
import numpy as np
```
<!--more-->

```python
def np_shift(xs, n):
    e = np.empty_like(xs)
    if n >= 0:
        e[:n] = np.nan
        e[n:] = xs[:-n]
    else:
        e[n:] = np.nan
        e[:n] = xs[-n:]
    return e


def td_sequential(close: list) -> np.array:
    shift_const: int = 4
    close_np = np.array(close)
    close_shift = np_shift(close_np, shift_const)
    compare_array = close_np > close_shift
    result = np.empty(len(close_np), int)
    counting_number: int = 0
    for i in range(len(close_np)):
        if np.isnan(close_shift[i]):
            result[i] = 0
        else:
            compare_bool = compare_array[i]
            if compare_bool:
                if counting_number >= 0:
                    counting_number += 1
                else:
                    counting_number = 1
            else:
                if counting_number <= 0:
                    counting_number -= 1
                else:
                    counting_number = -1
            result[i] = counting_number
    return result
```