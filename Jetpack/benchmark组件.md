### Benchmark组件

使用 Jetpack 基准库，可以从 AndroidStudio 中快速对基于 Kotlin 或 Java 的代码进行基准化分析。该库会处理预热，衡量代码性能和分配计数，并将基准化分析结果的更多详细信息输出到 Android Studio 控制台和 JSON 文件。

如果需要集成 Benchmark，可参考Api：https://developer.android.google.cn/studio/profile/benchmark

#### 依赖声明

```groovy
dependencies {
  androidTestImplementation
    "androidx.benchmark:benchmark-junit4:1.0.0"
}

android {
  ...
  defaultConfig {
    ...
    testInstrumentationRunner "androidx.benchmark.junit4.AndroidBenchmarkRunner"
  }
}
```

Benchmark 库还提供了可与基准模块搭配使用的 Gradle 插件。此插件可设置模块的默认构建配置，设置[将基准输出复制到主机](https://developer.android.com/studio/profile/benchmark#collect-data)，并提供`./gradlew lockClocks`任务。

要使用该插件，请在顶级`build.gradle`文件中添加以下`classpath`：

```groovy
buildscript {
    repositories {
        google()
    }
    dependencies {
        classpath "androidx.benchmark:benchmark-gradle-plugin:1.0.0"
    }
}
```

然后，将该插件应用到基准模块的`build.gradle`文件中：

```groovy
apply plugin: "androidx.benchmark"
```

#### 新功能

##### 版本 1.0.0

发布了`androidx.benchmark:benchmark-common:1.0.0`、`androidx.benchmark:benchmark-gradle-plugin:1.0.0`和`androidx.benchmark:benchmark-junit4:1.0.0`。

使用 Benchmark 库，可为应用代码编写性能基准测试并快速获得分析结果。

它可以防止出现构建和运行时配置问题并稳定设备性能，以确保衡量值的准确性和一致性。

主要功能包括：

- 时钟稳定
- 自动设置线程优先级
- 支持界面性能测试，例如在 RecyclerView 中
- JIT 感知预热和循环
- 用于后期处理的 JSON 基准输出