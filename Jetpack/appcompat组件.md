### Appcompat 组件

#### 依赖声明

```groovy
dependencies {
    def appcompat_version = "1.2.0"

    implementation "androidx.appcompat:appcompat:$appcompat_version"
    // For loading and tinting drawables on older versions of the platform
    implementation "androidx.appcompat:appcompat-resources:$appcompat_version"
}
```

#### 新功能

##### 版本1.2.0

- 修复了对配置覆盖使用情形（包括自定义语言区域和字体缩放）的支持。
- 已弃用`AppCompatDelegate.attachBaseContext()`。替代方法为`AppcompatDelegate.attachBaseContext2()`。
- 已弃用`CollapsibleActionView`。不再需要此接口，请使用平台提供的`android.view.CollapsibleActionView`。

##### 版本1.1.0

- **深色模式改进**：弃用了`MODE_NIGHT_AUTO`和基于当前时间的深色/浅色模式切换。考虑使用显示设置或`MODE_NIGHT_AUTO_BATTERY`。

- **Activity 1.0**：`AppCompatActivity`现在通过 Fragment 1.1.0 从 Activity 1.0.0 的`ComponentActivity`进行过渡扩展。

- **AppcompatActivity LayoutId 构造函数**：`AppCompatActivity`的子类现在可以选择性地对`AppCompatActivity`调用采用`R.layout` ID 的构造函数，以指明应设置为内容视图的布局，作为调用`onCreate()`中的`setContentView()`的替代方法。这不会改变子类必须具有无参构造函数的要求。

  ```kotlin
  class MainActivity(@LayoutRes layoutRedId: Int) : AppCompatActivity(layoutRedId) {
      @Keep
      constructor() : this(R.layout.activity_main)
  }
  ```

  

##### 版本1.0.2

`core-1.0.0`和`appcompat-1.0.2`的问题修复版本。

##### 版本1.0.0

新增`AnimatedStateListDrawableCompat`可提供可绘制对象状态之间的动画转换。

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<animated-selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/off" android:drawable="@mipmap/ic_launcher" android:state_selected="false"/>
    <item android:id="@+id/on" android:drawable="@mipmap/ic_launcher_round" android:state_selected="true"/>

    <transition
        android:fromId="@id/off"
        android:toId="@id/on">
        <animation-list>
            <item android:drawable="@android:color/black" android:duration="500"/>
            <item android:drawable="@android:color/holo_blue_light" android:duration="500"/>
            <item android:drawable="@android:color/holo_red_light" android:duration="500"/>
            <item android:drawable="@mipmap/ic_launcher_round" android:duration="500"/>
        </animation-list>
    </transition>

    <transition
        android:fromId="@id/on"
        android:toId="@id/off">
        <animation-list>
            <item android:drawable="@android:color/holo_red_light" android:duration="500"/>
            <item android:drawable="@android:color/holo_blue_light" android:duration="500"/>
            <item android:drawable="@android:color/black" android:duration="500"/>
            <item android:drawable="@mipmap/ic_launcher" android:duration="500"/>
        </animation-list>
    </transition>
</animated-selector>

<ImageView
	...
    android:background="@drawable/animation_state_list"/>
```

