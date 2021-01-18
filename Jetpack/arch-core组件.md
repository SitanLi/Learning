### Arch Core组件

该组件为LiveData线程切换的依赖，可以理解为arch的基础工具包。

#### 依赖声明

AndroidStudio 默认导入。

#### core-common 包含的类：

- **SafeIterableMap**：支持键值对存储，用链表实现，模拟成Map的接口；支持在遍历的过程中删除任意元素，不会触发 ConcurrentModifiedException；非线程安全。
- **FastSafeIterableMap**：SafeIterableMap的扩展类。

#### core-runtime 包含的类：

- **ArchTaskExecutor**：LiveData的线程处理器。
- **DefaultTaskExecutor**：默认的`ArchTaskExecutor`。