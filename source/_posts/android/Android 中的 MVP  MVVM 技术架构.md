---
title: Android 中的 MVP | MVVM 技术架构
date: 2022-03-10 10:45:18
tags: [mvp, mvvm]
---

### Android 中的 MVP | MVVM 技术架构

### 1、MVP

全称：`Model-View-Presenter` ；`MVP` 是从经典的模式 `MVC` 演变而来，它们的基本思想有相通的地方 `Controller` / `Presenter` 负责逻辑的处理，`Model` 提供数据，`View` 负责显示。

![](https://upload-images.jianshu.io/upload_images/15226743-947a7c01f8199148.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

#### 优点

1. 模型与视图完全分离，我们可以修改视图而不影响模型

2. 可以更高效地使用模型，因为所有的交互都发生在一个地方——Presenter内部

3. 我们可以将一个Presenter用于多个视图，而不需要改变Presenter的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁。

4. 如果我们把逻辑放在Presenter中，那么我们就可以脱离用户接口来测试这些逻辑（单元测试）

#### 缺点

由于对视图的渲染放在了 `Presenter` 中，所以视图和 `Presenter` 的交互会过于频繁。还有一点需要明白，如果 `Presenter` 过多地渲染了视图，往往会使得它与特定的视图的联系过于紧密。一旦视图需要变更，那么 `Presenter` 也需要变更了。

### 2、MVVM

`MVVM` 是 `Model-View-ViewModel` 的简写。它本质上就是 `MVP` 的改进版。`MVVM` 模式将 `Presenter` 改名为 `ViewModel`，基本上与 `MVP` 模式完全一致。唯一的区别是，它采用双向绑定（DataBinding），View的变动，自动反映在 ViewModel，反之亦然。

其中的 `VM` 是 `ViewModel` 的缩写，`ViewModel` 可以理解成是 `View` 的数据模型和 `Presenter` 的合体，`ViewModel` 和 `View` 之间的交互通过 `DataBinding`完成，而 `DataBinding` 可以实现双向的交互，这就使得视图和控制层之间的耦合程度进一步降低，关注点分离更为彻底，同时减轻了 `Activity` 的压力。

![](https://upload-images.jianshu.io/upload_images/15226743-1b2adc4a66e12c6e.png?imageMogr2/auto-orient/strip|imageView2/2/w/715/format/webp)

#### 优点

1. 双向绑定技术，当 `Model` 变化时，`View-Model` 会自动更新，`View` 也会自动变化。很好做到数据的一致性，不用担心，在模块的这一块数据是这个值，在另一块就是另一个值了。所以 `MVVM` 模式有些时候又被称作：`model-view-binder` 模式。
2. `View` 的功能进一步的强化，具有控制的部分功能，若想无限增强它的功能，甚至控制器的全部功几乎都可以迁移到各个 `View` 上（不过这样不可取，那样 `View` 干了不属于它职责范围的事情）。`View` 可以像控制器一样具有自己的 `View-Model`。
3. 由于控制器的功能大都移动到 `View` 上处理，大大的对控制器进行了瘦身。不用再为看到庞大的控制器逻辑而发愁了。
4. 可以对 `View` 或 `ViewController` 的数据处理部分抽象出来一个函数处理 `model`。这样它们专职页面布局和页面跳转，它们必然更一步的简化。

#### 缺点

1. 数据绑定使得 `Bug` 很难被调试。你看到界面异常了，有可能是你 `View` 的代码有 `Bug`，也可能是 `Model` 的代码有问题。数据绑定使得一个位置的 `Bug` 被快速传递到别的位置，要定位原始出问题的地方就变得不那么容易了。
2. 一个大的模块中，`model` 也会很大，虽然使用方便了也很容易保证了数据的一致性，当时长期持有，不释放内存，就造成了花费更多的内存。
3. 数据双向绑定不利于代码重用。客户端开发最常用的重用是 `View`，但是数据双向绑定技术，让你在一个 `View` 都绑定了一个 `model` ，不同模块的 `model` 都不同。那就不能简单重用 `View` 了。


个人觉得 `MVVM` 有点像 `H5` 的 `Vue` 框架，微信小程序开发也是这种模式。
