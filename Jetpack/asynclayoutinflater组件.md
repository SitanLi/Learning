### AsyncLayoutInflate组件

该组件为异步初始化布局的组件，它的本质就是把布局文件的`inflate`放入到子线程里面，等到初始化成功后，再通过接口抛回主线程。

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.asynclayoutinflater:asynclayoutinflater:1.0.0"
  }
```

#### AsyncLayoutInflate的局限性

1. 所有构件的`View`中必须不能直接使用`Handler`或者是调用`Looper.myLooper()`，因为异步线程默认没有调用`Looper.prepare()`。
2. 异步转换出来的 View 并没有被加到 parent view 中，`AsyncLayoutInflater`是调用了`LayoutInflater.inflate(int,ViewGroup,false)`，因此如果需要加到 parent view 中，就需要手动添加。
3. `AsyncLayoutInflater`不支持设置`LayoutInflater.Factory`或者`LayoutInflater.Factory2`。
4. 同时缓存队列默认 10 的大小限制，如果超过了10个则会导致主线程等待。
5. 使用单线程来做全部的`inflate`工作，如果一个界面中有很多布局，则不一定能够满足。

#### 使用示例

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    AsyncLayoutInflater(this).inflate(
        R.layout.activity_main,
        null
    ) { view: View, _: Int, _: ViewGroup? ->
        setContentView(view)
    }
}
```
