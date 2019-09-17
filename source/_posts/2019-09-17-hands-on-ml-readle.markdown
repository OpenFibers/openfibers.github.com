---
layout: post
title: "hands-on-ml-readle"
date: 2019-09-17 23:39:52 +0800
comments: true
categories: 
---


### 下载数据集

from sklearn.datasets import fetch_openml

mnist = fetch_openml('mnist_784', data_home='./')  # downloaded from www.openml.org
print(mnist)

如果提示 SSL: CERTIFICATE_VERIFY_FAILED，需要先更新证书：  


```
sudo /Applications/Python\ 3.7/Install\ Certificates.command
```
