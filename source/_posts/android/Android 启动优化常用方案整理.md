---
title: Android 启动优化常用方案整理
date: 2022-03-06 22:36:37
tags: [android, optimization]
---

### Android 启动优化常用的方案整理

首先附上启动流程：Launcher 点击图标开始

1. `Launcher startActivity` 
2. `AMS` 查找应用进程是否创建
3. 未创建, `AMS` 通知从 `Zygote` 进程中 `fork` 创建出一个新的进程分配给该应用　
4. 进程创建完毕, `AMS` 通过 `Binder` 通知应用进程
5. ActivityThread 调用 `performLaunchActivity` 
6. Application 构造函数及 `attachBaseContext()` ,  `onCreate()` 
7. `new Activity()` , 并为此 `Activity` 创建一个` new PhoneWindow(this) `
8. Activity `onCreate()` 
9. Activity `setContentView()` 
10. `new DecorView()` & `addView(contentRoot, contentParent)` 
11. `onFinishInflate()`: 此步骤只是 `inflate` 所有的 `DecorView` 上的布局 `views`，并不可见  
12. Activity `onStart()` 
13. Activity `onResume()` 
14. `window.addView(mDecorView)` 
15. View `onAttachedToWindow()` 、 `onMeasure()` 、 `onSizeChanged()` 、 `onLayout()` 、  `onDraw()` 
16. 至此步为止才把 `DecorView` 加给 `Window` , 应用首页才可见, `onWindowFocusChanged(true)` 
17. Activity `onPause() `
18. View `onWindowFocusChanged(false)`  
19. Activity `onStop()` 
20. Activity `onDestroy()` 
21. View `onDetackedFromWindow()`

#### 1、Application 初始化方面优化

在业务代码中，`Application` 启动时经常会初始化很多项目内的第三方库，这对 App 的启动会有很大影响，所以尽量把一些不太紧要的初始化异步执行，把必要的一些初始化才放到 `onCreate` 中执行。

至于异步初始化的方法则有：`其他线程` 、`IntentService` 、`WorkManager ` 、`MessageQueue.addIdleHandler()` 等等进行处理。

> `IntentService` Android 8.0+ 由于 `Service` 后台被限制已经弃用，推荐使用 `JobIntentService` 或 `Jetpack` 组件包的 `WorkManager` 进行代替使用。


#### 2、优化首次安装或冷启动时，卡在白屏的视觉效果优化

在首次安装后或冷启动时，从 Launcher 中点击图标到闪屏页，在性能一般的机器上面，会有短时间的白屏或黑屏，这样对用户而言显然是不太好的。

我们可以在闪屏页配置一个背景，通常是一个和闪屏页相似的背景，然后就不会出现白屏或黑屏，而是设置的闪屏页背景，然后在显示闪屏页。

> 闪屏页即启动的第一个 Activity，App 启动速度和闪屏页的 onCreate 方法也有关系，所以尽量不要在闪屏页的 onCreate 做太多操作。 

启动页主题

```xml
<style name="SplashTheme" parent="@android:style/Theme.Light.NoTitleBar.Fullscreen">
        <item name="android:windowBackground">@drawable/splash_bg</item>
        <item name="android:windowFullscreen">true</item>
</style>
```

`splash_bg` 可以是图片也可以是自定义的 `layer` 组合。

```xml
<activity
    android:name=".SplashActivity"
    android:configChanges="orientation|screenSize|keyboardHidden"
    android:screenOrientation="portrait"
    android:theme="@style/SplashTheme">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```
#### 3、MultiDex OPT 优化

Android 低版本`4.x及以下，SDK <  21`的设备，采用的 `Java` 运行环境是 `Dalvik` 虚拟机。它相比于高版本，最大的问题就是在安装或者升级更新之后，首次冷启动的耗时漫长。这常常需要花费几十秒甚至几分钟，用户不得不面对一片黑屏，熬过这段时间才能正常使用 APP。这个问题的根本原因就在于，安装或者升级后首次 `MultiDex` 花费的时间过于漫长。

##### 推荐使用 [抖音的 BoostMultiDex 优化](https://github.com/bytedance/BoostMultiDex) 

1. 利用系统隐藏函数，直接加载原始 `DEX` 字节码，避免 `ODEX` 耗时
2. 多级加载，在 `DEX` 字节码、`DEX` 文件、`ODEX` 文件中选取最合适的产物启动 APP
3. 单独进程做 `OPT` ，并实现合理的中断及恢复机制

### 4、利用 Redex 工具优化 Dex

`Redex` 是 `Facebook` 的一个开源工具，官方的介绍是 `An Android Bytecode Optimizer` ，`Android 字节码优化器`。可以减小 `Android` `Apk` 的大小和提高 `App` 启动速度。

根据官方文档的说明，主要的优化项有以下几点：

- **1、混淆和压缩**
  类似 `Android` 中的 `proguard` ，将字节码中的类名、方法名等替换成简短的字符串。

- **2、内联函数**
  `functionA` -> `functionB` -> `functionC` ，内联后变成 `functionA` (包含 `functionB` 代码)-> `functionC` ，减少函数调用时间。

- **3、删除代码**
  类似于标记回收算法，从某些入口函数可以遍历标记出可以访问到的方法，最后未标记到的方法可被移除。理论比较简单，但在 `Android` 中还有反射，或者资源文件中对代码的引用等特殊情况需要考虑到。

- **4、基于反馈的class分布**
  也就是可以对 `Dex` 进行重组，根据使用方提供的 `App` 启动时加载的类序列配置文件调整 `Dex` 中类的顺序，把 `App` 冷启动时需要加载的类，放到 Dex前部。

- **5、删除接口**
  删除只有一个实现类的接口，直接使用实现类。

- **6、删除 MetaData**
  `Dex` 文件中的一些元数据在运行中并不会使用到。所以可以用 `Dex` 中已有的字符串代替 `Java` 源文件引用，删除运行时使用不到的注解。

#### 5、其它优化方案

- 提前加载 `SharedPreferences`

  在 `Multidex` 之前CPU是空闲的，加载系统类是可行的，所以可充分利用这段时间加载`SharedPreferences` 。

- 启动阶段不启动子进程

  初始化子进程会消耗CPU资源，在启动阶段会导致主进程CPU资源紧张，导致启动阶段资源紧张初始化过慢。

- 启动阶段不启动 `Service` 、`ContentProvider`

  在 `Application` 生命周期方法 `attachBaseContext` 方法和 `onCreate` 方法中间还会执行`ContentProvider` 生命周期方法，如果在 `Application` 初始化过程中启动了 `Service` 或者 `Contentprovider` 会执行 `ContentProvider` 生命周期方法，非常耗时。

- 提前异步类加载

  对启动阶段用到的类进行提前异步加载，加载方法有两种：

  - `Class.forName()` 只加载类本身及其静态变量的引用类。
  - `new 类实例` 可以额外加载类成员变量的引用类。

- 启动阶段抑制 `GC`

  支付宝就用了该方案

- `CPU` 锁频











