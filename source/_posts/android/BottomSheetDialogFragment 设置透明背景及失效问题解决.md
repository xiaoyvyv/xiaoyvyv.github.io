---
title: BottomSheetDialogFragment 去除白色背景，设置透明背景
date: 2022-02-11 14:46:30
tags: [android]
---
### BottomSheetDialogFragment 去除白色背景，设置透明背景

先直接贴代码：

```kotlin
    /**
     * 设置透明背景 BottomSheetDialogFragment
     *
     * @param skipCollapsed  是否跳过折叠
     * @param dimAmount      对话框外部阴影比重(0f ~ 1f)
     * @param dialogBehavior 其他交互控制
     */
    @JvmStatic
    fun BottomSheetDialogFragment.onStartTransparentDialog(
        skipCollapsed: Boolean = true,
        @FloatRange(from = 0.0, to = 1.0) dimAmount: Float = 0.25f,
        dialogBehavior: BottomSheetBehavior<FrameLayout>.() -> Unit = {}
    ) {
        val dialog = dialog as? BottomSheetDialog ?: return
        val window = dialog.window ?: return
        val bottomSheet = window.findViewById<View>(R.id.design_bottom_sheet)

        window.setDimAmount(dimAmount)
        window.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))

        dialogBehavior(dialog.behavior)
        dialog.behavior.skipCollapsed = skipCollapsed

        // 设置透明背景，光设置 setBackgroundColor 还不够，高版本的库默认加入了 backgroundTintList 的属性
        // 所以还要修改背景色的着色为透明才行
        bottomSheet?.backgroundTintMode = PorterDuff.Mode.CLEAR
        bottomSheet?.backgroundTintList = ColorStateList.valueOf(Color.TRANSPARENT)
        bottomSheet?.setBackgroundColor(Color.TRANSPARENT)
    }

```

然后在 BottomSheetDialogFragment 的 onStart 回调内调用该拓展方法即可，也可传入相关参数调整样式。

```kotlin

    override fun onStart() {
        super.onStart()
        onStartTransparentDialog()
    }

```

### 查看白色背景在源码中哪里设置的
查看 `BottomSheetDialog` 源码，在下面的方法中，引入了一个布局 `design_bottom_sheet_dialog.xml`，其中的 `bottomSheet` 就是我们自定义传入视图的容器控件。


```java

  private FrameLayout ensureContainerAndBehavior() {
    if (container == null) {
      container =
          (FrameLayout) View.inflate(getContext(), R.layout.design_bottom_sheet_dialog, null);

      coordinator = (CoordinatorLayout) container.findViewById(R.id.coordinator);
      bottomSheet = (FrameLayout) container.findViewById(R.id.design_bottom_sheet);

      behavior = BottomSheetBehavior.from(bottomSheet);
      behavior.addBottomSheetCallback(bottomSheetCallback);
      behavior.setHideable(cancelable);
    }
    return container;
  }

```
布局文件 `design_bottom_sheet_dialog.xml` 内容如下

```xml

<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

  <androidx.coordinatorlayout.widget.CoordinatorLayout
      android:id="@+id/coordinator"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:fitsSystemWindows="true">

    <View
        android:id="@+id/touch_outside"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:focusable="false"
        android:importantForAccessibility="no"
        android:soundEffectsEnabled="false"
        tools:ignore="UnusedAttribute"/>

    <FrameLayout
        android:id="@+id/design_bottom_sheet"
        style="?attr/bottomSheetStyle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal|top"
        app:layout_behavior="@string/bottom_sheet_behavior"/>

  </androidx.coordinatorlayout.widget.CoordinatorLayout>

</FrameLayout>

```

可以看出 `bottomSheet` 默认会去引入一个全局的主题样式 `bottomSheetStyle`，点进去发现是引入的 `Widget.MaterialComponents.BottomSheet.Modal`，挨着挨着查找下去：

```xml

  <style name="Base.V14.ThemeOverlay.MaterialComponents.BottomSheetDialog" parent="Base.ThemeOverlay.MaterialComponents.Dialog">
    <item name="android:windowIsFloating">false</item>
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:windowAnimationStyle">@style/Animation.MaterialComponents.BottomSheetDialog</item>
    <item name="bottomSheetStyle">@style/Widget.MaterialComponents.BottomSheet.Modal</item>
    <item name="enableEdgeToEdge">true</item>
    <item name="paddingBottomSystemWindowInsets">true</item>
    <item name="paddingLeftSystemWindowInsets">true</item>
    <item name="paddingRightSystemWindowInsets">true</item>
    <item name="paddingTopSystemWindowInsets">true</item>
  </style>

  ...

  <style name="Widget.MaterialComponents.BottomSheet.Modal" parent="Widget.MaterialComponents.BottomSheet">
    <item name="android:elevation" ns1:ignore="NewApi">
      @dimen/design_bottom_sheet_modal_elevation
    </item>
  </style>
  
  ...

  <style name="Widget.MaterialComponents.BottomSheet" parent="Widget.Design.BottomSheet.Modal">
    ...
    <item name="android:background">@null</item>
    <item name="backgroundTint">?attr/colorSurface</item>
    ...
  </style>

  ...

  <style name="Base.V14.Theme.MaterialComponents.Bridge" parent="Platform.MaterialComponents">
    ...
    <item name="colorSurface">@color/design_dark_default_color_surface</item>
    ...
  </style>

  ...

  <color name="design_dark_default_color_surface">#121212</color>

```
很容易可以看出，这个默认白色背景的来源了。

