### Acticity 组件

#### 依赖声明

```groovy
dependencies {
    def activity_version = "1.1.0"

    // Java language implementation
    implementation "androidx.activity:activity:$activity_version"
    // Kotlin
    implementation "androidx.activity:activity-ktx:$activity_version"
}
```

#### 版本更新

##### 1.2.0 主要功能

- **Activity Result API**：`ComponentActivity`现在提供了一个`ActivityResultRegistry`，让您无需替换 Activity 或 Fragment 中的方法，即可处理`startActivityForResult()`+`onActivityResult()`以及`requestPermissions()`+`onRequestPermissionsResult()`流程，通过`ActivityResultContract`提高了类型安全性。使用示例请参考[获取 Activity 的结果](https://developer.android.com/training/basics/intents/result)一文。
- **ContextAware**：`ComponentActivity`现在实现了`ContextAware`，可添加一个或多个`OnContextAvailableListener`实例，他们将在`Activity.onCreate()`之前接收回调。
  - 通过挂起（协程）的 Kotlin 扩展`withContextAvailable()`，可以在 Context 变为可用时运行非挂起代码块，并返回结果。
  - 此 API 由 Fragment 1.3.0 中的`FragmentActivity`用来恢复`FragmentManager`的状态。向`FragmentActivity`的子类添加的任何监听器都将在该监听器之后运行。
  - 此 API 由 AppCompat 1.3.0-alpha02 或更高版本中的`AppCompatActivity`使用。向`AppCompatActivity`的子类添加的任何监听器都将在该监听器之后运行。
- **ViewTree 支持**：`ComponentActivity`现在支持在 Lifecycle 2.3.0 和 SavedState 1.1.0 中添加的`ViewTreeLifecycleOwner.get(View)`、`ViewTreeViewModelStoreOwner.get(View)`和`ViewTreeSavedStateRegistryOwner` API，以便针对直接添加到 `ComponentActivity` 中的任何 View 将相应 Activity 返回为 `LifecycleOwner`、`ViewModelStoreOwner`和`SavedStateRegistryOwner`。
- **reportFullyDrawn()向后移植**：`reportFullyDrawn()`的`Activity`方法已反向移植到`ComponentActivity`中，以便在所有 API 级别上使用，从而修复了 API 19 上的崩溃问题并为所有 API 级别添加了对此方法的跟踪。

##### 1.1.0 主要功能

- **Lifecycle ViewModel SavedState 集成**：现在将`by viewModels()、ViewModelProvider`构造函数或`ViewModelProviders.of()`与`ComponentActivity`或其子类一起使用时，会使用`SavedStateViewModelFactory`作为默认出厂设置。

##### 1.0.0 主要功能

- **ComponentActivity**：`ComponentActivity` 在 Fragment 1.1.0 中充当`FragmentActivity`的新基类，由此引申开来，它在 AppCompat 1.1.0 中充当`AppCompatActivity`的新基类。

- **activity-ktx**：`activity-ktx`模块包含用于访问 ViewModel 的`by viewModels`Kotlin 属性扩展。当您添加 Fragment 1.1.0 中的 `fragment-ktx`时，系统会自动添加此模块。

  ```kotlin
  private val viewModel by viewModels<MainViewModel>()
  ```

- **onBackPressedDispatcher**：作为替换`onBackPressed()`的可组合替代方案，您现在可以从任何`LifecycleOwner`（如 Fragment ）注册`onBackPressedCallback`来拦截系统返回按钮事件。具有接收器版本`addCallback`的 lambda 已添加到`activity-ktx`。

  ```kotlin
  /**
   * @param enabled 是否启用拦截，false-不启用；true-启用
   */
  onBackPressedDispatcher.addCallback(this, true) {
      Log.d("TAG", "onCreate: on back press.")
  }
  ```