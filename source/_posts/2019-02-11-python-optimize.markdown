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

# numexpr
比原生 Python 快 25 倍左右。  

<!--more-->

# 多线程与多进程并发

```python
concurrent.futures.ThreadPoolExecutor(cpu_count)
# 或者
concurrent.futures.ProcessPoolExecutor(cpu_count)
```

# CuPy
使用 CUDA 计算，直接将 numpy 替换成 cupy。  
比原生 Python 快 250 倍左右。  

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

Over