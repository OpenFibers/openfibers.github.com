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
缺点是要写个 makefile

# pypy

优点是无需像 cython 一样需要修改代码，写 makefile 和 main，缺点是有些三方库不支持。  

安装：  

```bash
brew install pypy3
```

然后安装 pypy pip。注意 pypy pip 不支持 socks5 代理，可能需要关闭或指定 http 代理：  

```bash
pypy3 -m ensurepip
export ALL_PROXY=
pypy3 -m pip install pip --upgrade
pypy3 -m pip install setuptools --upgrade
```

将 pypy3 path 加入 $PATH 不然安装 tables 的时候报 warning:  

```bash
export PATH=$PATH:/usr/local/share/pypy3
```

安装依赖（举点例子）：  

```bash
pypy3 -m pip install numpy
pypy3 -m pip install TA-Lib
pypy3 -m pip install requests
pypy3 -m pip install ccxt
pypy3 -m pip install tables
pypy3 -m pip install matplotlib
pypy3 -m pip install coloredlogs
pypy3 -m pip install pandas
```

如果 macOS 遇到 pypy 安装 numpy 时提示：  

```bash
Checking for cc... ld: library not found for -lgcc_s.10.4
clang: error: linker command failed with exit code 1 (use -v to see invocation)
...
RuntimeError: Broken toolchain: cannot link a simple C program
```

尝试下面命令后再次重试安装 numpy：  

```bash
cd /usr/local/lib 
sudo ln -s ../../lib/libSystem.B.dylib libgcc_s.10.4.dylib
cd -
```

# Numpy
比原生 Python 快 10 倍左右。  

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

# Pandas: 避免大量 df.append() 或 df.iloc() 调用
如果有频繁的 append 操作，使用 list 而非 df，CPU 耗时、内存消耗都降低很多。3600 行的回测运行 10 次，df 改为 list，运行时间从 49 秒降低至 13.35 秒，性能提升 267%。  
如果有频繁的 iloc 操作，想办法使用 list + dict 代替 iloc + key 取数，还是刚才那个 3600 行跑 10 次，14.5 秒缩短到 9 秒，性能提升 55%。

# CPU 耗时分析

## pprofile

```bash
pip3 install pprofile
pprofile hard_work.py | grep -v 0.00% > hard_work.log    
```

结果如下：

```
Command line: hard_work.py
Total duration: 12.2936s
File: /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py
File duration: 2.11106s (17.17%)
Line #|      Hits|         Time| Time per hit|      %|Source code
------+----------+-------------+-------------+-------+-----------
(call)|         1|   0.00452113|   0.00452113|  0.04%|# <frozen importlib._bootstrap>:1009 _handle_fromlist
(call)|         8|   0.00171661|  0.000214577|  0.01%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/process.py:72 __init__
(call)|         1|    0.0330989|    0.0330989|  0.27%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:250 _setup_queues
(call)|         1|    0.0355132|    0.0355132|  0.29%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:227 _repopulate_pool
(call)|         1|  0.000757217|  0.000757217|  0.01%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/threading.py:758 __init__
(call)|         1|   0.00308776|   0.00308776|  0.03%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/threading.py:829 start
(call)|         1|  0.000803709|  0.000803709|  0.01%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/threading.py:829 start
   218|         8|   0.00123382|  0.000154227|  0.01%|            worker = self._pool[i]
(call)|         8|   0.00212884|  0.000266105|  0.02%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:152 Process
(call)|         8|    0.0311823|   0.00389779|  0.25%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/process.py:101 start
(call)|         1|   0.00201011|   0.00201011|  0.02%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:212 _join_exited_workers
(call)|         1|    0.0309622|    0.0309622|  0.25%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/context.py:109 SimpleQueue
(call)|         1|   0.00208473|   0.00208473|  0.02%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/context.py:109 SimpleQueue
(call)|         1|   0.00138021|   0.00138021|  0.01%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:650 get
   411|        19|  0.000627041|  3.30022e-05|  0.01%|        while thread._state == RUN or (pool._cache and thread._state != TERMINATE):
   412|        19|    0.0777183|   0.00409043|  0.63%|            pool._maintain_pool()
(call)|         1|   0.00205207|   0.00205207|  0.02%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:244 _maintain_pool
   413|        18|      2.02267|     0.112371| 16.45%|            time.sleep(0.1)
   422|         1|   0.00222588|   0.00222588|  0.02%|        for taskseq, set_length in iter(taskqueue.get, None):
(call)|         1|   0.00127983|   0.00127983|  0.01%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/threading.py:534 wait
(call)|         1|    0.0013001|    0.0013001|  0.01%|# /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/multiprocessing/pool.py:647 wait
```

## vprof

如果使用 cython，直接：

```bash
vprof -c h hard_work.py # code heatmap (first call below)
vprof -c p hard_work.py # code profiling (second call below)
```

如果不使用 cython 而是 pypy：

使用 pypy pip 安装 vprof。注意 pypy pip 不支持 socks5 代理，可能需要关闭或指定 http 代理： 

```
export ALL_PROXY=
pypy3 -m pip install vprof
```

然后 pypy3 下要安装原有的 pip 各种依赖，最后跑 vprof：

```bash
pypy3 -m vprof -c h hard_work.py # code heatmap (first call below)
pypy3 -m vprof -c p hard_work.py # code profiling (second call below)
```

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