---
layout: post
title: "macOS BigSur 适配指引"
date: 2020-09-07 17:06:28 +0800
comments: true
categories: ["macOS", "BigSur"]
---


## 在 Apple Silicon 上做性能分析

* 原生代码一般都会比转码性能更好
* 小心使用 Intel 专有的代码优化（SSE, AVX）
	* 可能需要针对 Apple Silicon 的优化版本
* 尽可能使用 Apple 提供的 AP (Accelerate framework)

## Apple Silicon 的非对称 CPU 核心

* Apple Silicon Mac 有两种类型的核心
	* 高性能核心（P Cores）
	* 节能核心（Cores)
	* 在高并发任务场景下，所有的核心可以同时开启 

## 避免使用忙等和 spinlock 
* 忙等会占用性能核心，进而导致任务延迟
* 推荐的阻塞同步原语
	* NSLock, os_unfair_lock, pthread mutexes
	* NSCondition, pthread 条件变量
* 避免按照 CPU 的数量划分工作任务

```
// Prefer blocking locks and condition variables
func performWorkUnderSpinlock () {
	os_unfair_lock_lock()
	performWork()
	os_unfair_lock_unlock()
}

func retrieveNextWorkTask() -> WorkTask {
	condition.lock()
	while !taskQueue.hasAnyWork {
		condition.wait()
	}
	let task = taskQueue.pop()
	condition.unlock()
	return task
}
```

## 在 Apple Silicon 上调试、测试和性能分析

* 构建工作可以使用任何 Mac，但是运行 arm64 代码只能在 Apple Silicon Mac 上
* 分别测试原生和 Rosetta 的运行方式
* 注意 Intel 专有代码和忙等的实现


|   | 第一方<br>从源码编译<br>和你的 app 一起发布 | 第三方<br>预先编译好的二进制<br>和 app 一起或者单独发布 |
|:------------- |:---------------:| -------------:|
| 进程内      | √ |         √ |
| 进程外      | √        | x<br>原生 app 只能加载原生插件<br>Rosetta 转码 ap 只能加载 x86_64 插件 |

## 通过 XPC 使用进程外插件

* 每个插件一个进程或者每种架构一个进程
* 更好的稳定性和安全性

## 为了插件使用 Rosetta

* 用户可以强制一个通用 app 通过 Rosetta 启动 
* 通过 Info.pist 关键字可以禁用
* 详见 macOS 移植文档

## 警惕多线程缺陷

* Intel CPU 和 Apple Silicon 采用不同的内存排序模型
* 正确的代码有相同的行为，有缺陷的代码（争条件，数据竟争）却各有各的表现
	* 一个数据竞争在 Intel CPU 上也许不显露，却有可能在 Apple Silicon 上导致崩溃
	* Rosetta 提供 Intel CPU 相同的内存排序
* 使用 Thread Sanitizer 来发现和防止数据竞争问题

## 兼容性

* 兼容的 app 自动可见
* 可以在 App Store Connect 管理


## 硬件的区别

* 鼠标和触控事件
* 环境传感器的区别
	* 陀螺仪
	* 指南针
	* 雷达摄像头
	* GPS
* 相机
	* 使用 AVCaptureDeviceDiscoverySession 自动兼容
* 照片选取