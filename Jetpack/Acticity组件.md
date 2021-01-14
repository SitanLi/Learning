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

1. ##### 1.1.0 主要功能

   - **Lifecycle ViewModel SavedState 集成**：现在将`by viewModels()、ViewModelProvider`构造函数或`ViewModelProviders.of()`与`ComponentActivity`或其子类一起使用时，会使用`SavedStateViewModelFactory`作为默认出厂设置。

2. ##### 1.0.0 主要功能

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