---
title: Androis Studio 引入 AspectJx 报错：Invalid byte tag in constant pool 19 解决方案
date: 2022-02-11 14:46:30
tags: 
- android
- aspectjx
---

### Androis Studio 引入 AspectJx 报错：Invalid byte tag in constant pool 19 (Zip file is empty) 解决方案

​		一般出现该问题可能是项目使用了切面编程，同时又引入了 Kotlin，由于 AspectJx 版本过低未维护，不支持高版本的字节码导致的问题。
​		解决方法如下：

- ##### 在 `aspectjx` 配置代码块内，添加一个排除组 `versions.9` 

```groovy
aspectjx {
    enabled true
    exclude 'xxx', 'xxx', 'versions.9'
    include 'xxx', 'xxx'
}
```

**然后重新 Sync 即可。**

