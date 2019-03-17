---
layout: post
title: "Python/Numpy 性能优化"
date: 2019-02-11 10:53:30 +0800
comments: true
categories: [python, numpy, fin-tech, AI]
---

# Cython
将 Python 翻译成 c/c++ 再编译执行。  
比原生 Python 快 1.5 倍左右。  

# Numpy
比原生 Python 快 10 倍左右。  

# list.append() 代替 df = df.append()
如果有频繁的 append 操作，使用 list 而非 df，CPU 耗时、内存消耗都降低很多。3600行的回测，df 改为 list，运行时间从 49 秒降低至 13.35 秒，性能提升 267%。

# numexpr

```python
import numpy as np 
import numexpr as ne  
N = 10 ** 5
a = np.random.uniform(-1, 1, N)
b = np.random.uniform(-1, 1, N)
ne.evaluate('a ** 2 + b ** 2')
```

比 Numpy 快 2 到 10 倍。

<!--more-->

# 多线程与多进程并发

由于全局解释器的存在，多线程基本没用，用线程池最多优化个5%顶天了。

直接上进程池。进程池的好处是：  
    1) CPU 层面不受全局解释器的负面影响，可以发挥处理器的最大性能；  
    2) 内存层面，如果同一进程一直执行大内存操作（稍微大点的 DataFrame），进程会一直申请内存不释放(python 用完的对象的内存会留给后面的 python 对象使用，不会还给系统)。而单独的进程结束的时候，系统会释放掉进程分配的内存。参考以下两个链接：  
        * [Why doesn't Python release the memory when I delete a large object?](http://effbot.org/pyfaq/why-doesnt-python-release-the-memory-when-i-delete-a-large-object.htm)  
        * [https://stackoverflow.com/questions/15455048/releasing-memory-in-python](https://stackoverflow.com/questions/15455048/releasing-memory-in-python)

```python
import multiprocessing

def single_job(param):
    print(param)

if __name__ == '__main__':
    cpu_count_m = multiprocessing.cpu_count()
    pool = multiprocessing.Pool(cpu_count_m)
    result_m = pool.map(single_job, [param])
    pool.close()
    pool.join()
```

在4代i5上（双核四线程），测试了一个原本单进程执行的287秒的操作，设置进程池数量为2，最终时间为187秒（提升53%），进程池数量为4，最终时间为127秒（提升126%）。也就是线程池数量设置直接用`multiprocessing.cpu_count()`即可，不必考虑物理核数。  
内存从原本的峰值10+G降到了0.6G。4代i7（四核八线程）跑起来会更快（懒的测没测大概四倍多吧），内存峰值也稍微会多一点（1.2G）。老电脑没到期，8代i7要9月份才能换到，性能应该强更多了吧。。。。  

#### matplot 无法在进程池绘制问题

如果弹 The process has forked and you cannot use this CoreFoundation functionality safely. You MUST exec(). 错误：

```
The process has forked and you cannot use this CoreFoundation functionality safely. You MUST exec().
Break on __THE_PROCESS_HAS_FORKED_AND_YOU_CANNOT_USE_THIS_COREFOUNDATION_FUNCTIONALITY___YOU_MUST_EXEC__() to debug.
The process has forked and you cannot use this CoreFoundation functionality safely. You MUST exec().
```

不用把fork进程的数据回主进程渲染。直接更新 matplot 到 3.0.3 以上版本：  

```
pip3 install matplotlib --upgrade
```

# CuPy
使用 CUDA 计算，直接将 numpy 替换成 cupy。  
比原生 Python 快 250 倍左右。  
问题就是 Macbook Pro 新款都是 A 卡，真、苹果与狗不得入内。  

# 多显卡

使用 cupy.cuda.Device(cuda_index) 切换显卡设备：  

```python
with cupy.cuda.Device(1):
    x_on_gpu1 = cupy.array([1, 2, 3, 4, 5])
```

这里 x_on_gpu1 将在 GPU 1 上分配。  

# 使用 Chainer 简化主存/显存切换

本小节内容摘自 [在Chainer中使用GPU](https://bennix.github.io/blog/2017/12/14/chain_gpu/)，更多详细信息请参考原文。  

Chainer将CuPy的默认分配器更改为内存池，因此用户可以直接使用CuPy的功能而不需要处理内存分配器。  

Chainer提供了一些方便的功能来自动切换和选择设备。例如，chainer.cuda.to_gpu（）函数将numpy.ndarray对象复制到指定的设备：

```python
x_cpu = np.ones((5, 4, 3), dtype=np.float32)
x_gpu = cuda.to_gpu(x_cpu, device=1)
```

它相当于使用CuPy的以下代码：

```python
x_cpu = np.ones((5, 4, 3), dtype=np.float32)
with cupy.cuda.Device(1):
    x_gpu = cupy.array(x_cpu)
```

更多并发骚操作，参考[Python并行编程](https://python-parallel-programmning-cookbook.readthedocs.io/zh_CN/latest/index.html)。

Over