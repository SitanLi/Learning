### Collection 组件

重写 Java 中的小型集合，降低对内存的影响。

这些容器旨在更好地平衡内存的使用，与大多数其它标准 Java 容器不同，性能上说，原生 Java 容器在容量不大（几百条）的情况性能差异不大，但是如果数据结构庞大，则不建议使用该容器。

#### 依赖声明

```groovy
dependencies {
    def collection_version = "1.1.0"
    // Java language implementation
    implementation "androidx.collection:collection:$collection_version"
    // Kotlin
    implementation "androidx.collection:collection-ktx:$collection_version"
}
```

#### 版本说明

##### 版本 1.1.0

- 对 ”collection-ktx“工件中的`contains`和`isNotEmpty`函数使用了更高效的实现。

#### 重新实现的集合

- ArrayMap
- ArraySet
- LongSparseArray
- LruCache
- SparseArray