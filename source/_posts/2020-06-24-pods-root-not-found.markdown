---
layout: post
title: "diff: /Manifest.lock: No such file or directory: PODS_ROOT not defined"
date: 2020-06-24 18:25:29 +0800
comments: true
categories: 
---


WeexSDK 增加 pods 依赖的时候提示：

diff: /Manifest.lock: No such file or directory

查看 shell script：  

```bash
diff "${PODS_PODFILE_DIR_PATH}/Podfile.lock" "${PODS_ROOT}/Manifest.lock" > /dev/null
```

echo 发现 PODS_ROOT 为空，导致 diff 右值传递为 '/Manifest.lock'，所以文件找不到。  

查看 `Pods-WeexSDK.debug.xcconfig` 中对 `PODS_ROOT` 有定义：  

```
PODS_ROOT = ${SRCROOT}/Pods
```

而且 project Info -> Configuration 下面指定了正确的 xcconfig。怀疑 PODS_ROOT 被某优先级更高的设置覆盖为空了。  


全局搜索 `PODS_ROOT`，发现 target -> Build Settings -> User Defined 中对 `PODS_ROOT` 设置了空值。删除后问题解决。  
