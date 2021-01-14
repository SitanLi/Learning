### RxJava 简介

​	RxJava是异步的双向push，Iterable是同步的单向pull，通过下表进行对比：

|   事件   |   Iterable(pull)   |           Observable(push)            |
| :------: | :----------------: | :-----------------------------------: |
| 获取数据 |     `T next()`     |     `Consumer<? super T> onNext`      |
| 异步处理 | `throws Exception` | `Consumer<? super Throwable> onError` |
| 任务完成 |    `!hasNext()`    |          `Action onComplete`          |

​	RxJava 2 的思维导图如下：

![RxJava 2.x](https://github.com/aheven/RxJavaLearning/blob/master/pic/RxJava%202.x.png)
