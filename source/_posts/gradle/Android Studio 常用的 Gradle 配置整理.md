---
title: Android Studio Gradle Usage
tags: [android, gradle]
date: 2022-03-06 21:36:58
---

### Android Studio 常用的 Gradle 配置

#### 1、排除第三方库的某些依赖

单条依赖排除引用的某个第三方库

```groovy
implementation("android.support.v7.widget.Toolbar") {
	exclude group: 'com.android.support', module: 'support-annotations'
}
```

全局配置排除

在需要排除的模块的配置文件 `build.gradle` 中，顶层插入下方配置相关的内容，在 `all` 内一行行添加需要排除的库即可，就不用在单个依赖下去排除。

```groovy
configurations {
    all {
        // 排除某个组全部内容
        exclude group: 'com.android.support'
        // 排除某个组的具体模块
        exclude group: 'com.android.support', module: 'support-annotations'
    }
}
```

#### 2、开启 ViewBinding

在 `Android` 项目的配置文件 `build.gradle` 中的 `android` 块中添加配置，重新同步即可

```groovy
android {
    //...
    
    buildFeatures {
        viewBinding true
    }
    
    //...
}
```

#### 3、MultiDex 定义 main dex 中必须保留的类

```groovy
android {
    //...

    defaultConfig {
        //...

        // 定义 main dex 中必须保留的类
        multiDexKeepProguard file('mainDexClasses.pro')
    }
}
```

mainDexClasses.pro

```groovy
# 这里只是示例
-keep public class * extends java.lang.Thread { *; }

-keep public class com.synaric.common.utils.SystemUtils { *; }

-keep public class com.synaric.common.utils.SPUtils { *; }

-keep interface android.content.SharedPreferences { *; }

-keep class android.os.Handler { *; }

-keep class com.synaric.common.BaseSPKey { *; }

-keep class android.os.Messenger { *; }

-keep class android.content.Intent { *; }
```

