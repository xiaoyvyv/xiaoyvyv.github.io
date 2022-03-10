---
title: Gradle 配置请求超时时间
date: 2022-03-04 14:46:30
tags: 
- gradle
- androidstudio
---
### ### Gradle 配置请求超时时间

打开 `gradle.properties` 文件

增加下面的内容：

```properties
systemProp.org.gradle.internal.http.socketTimeout=10000
systemProp.org.gradle.internal.http.connectionTimeout=10000
```

默认 Gradle  超时设置的较长，某些情况依赖加载不了或仓库一直阻塞不响应，但 Gradle 不会及时抛出超时，造成程序任务一直处于构建中，也不知道具体是哪个环节卡住，配置一个较短的超时可以及时发现每个依赖或仓库的连通性有问题，避免浪费宝贵的时间去找问题。