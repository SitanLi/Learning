### 开发EventBus

#### 传统的 EventBus

EventBus 是事件总线框架，它是一种消息发布—订阅的模式，它的工作机制类似于观察者模式，通过通知者去注册观察者，最后由通知者向观察者发布消息。

在 Android 开发中，使用 EventBus 能够解耦 AsyncTask、Handler、Thread、Broadcast 等各种组件。除此之外，我们在开发 Android App 时还能使用 EventBus 来轻松实现跨越多个 Fragment 之间的通信。

EventBus 的主要角色如下：

- Event：传递的事件对象。
- Subscriber：事件的订阅者。
- Publisher：事件的发布者。
- ThreadMode：定义函数在何种线程中执行。

其中，ThreadMode 总共有四种线程模型。

1. Main UI 主线程。

   无论事件是在哪个线程中发布出来的。该事件处理函数都会在 UI 线程中执行。该方法可以用来更新 UI，但是不能处理耗时操作。

2. Background 后台线程。

   如果事件是在 UI 线程中发布出来的，那么该事件处理函数就会在新的线程中运行。如果事件本来就是在子线程中发布出来的，那么该事件处理函数将直接在当前线程中执行。在此事件处理函数中禁止进行 UI 更新操作。

3. Posting 和发布者处在同一个线程。

   无论该事件是在哪个线程发布出来的，事件处理函数都会在当前线程中运行。在线程模型为 PostThread 的事件处理函数中应尽量避免执行耗时操作，因为它会阻塞事件的传递，甚至可能会引起 ANR。

4. Async 异步线程。

   无论事件在哪个线程发布，该事件处理函数都会在新建的子线程中执行。同样，此事件处理函数中禁止进行 UI 的更新操作。

   Greenrobot 的 EventBus 还支持订阅事件的优先级，优先级越高则越快获得消息。

   EventBus 还有一个很重要的特性是支持 Sticky 事件。所谓 Sticky 事件是指事件消费者在事件发布之后才注册的也能接收到该事件的特殊类型。

#### 开发一个新的 EventBus（一）

EventBus 是一个很好的解耦工具，也是一个优秀的事件总线框架，而 RxJava 给我们带来了响应式的编程方式。如果在项目中使用了 RxJava，那么可以无须使用 EventBus，因为使用 RxJava 基本能够编写出实现 EventBus 的全部功能，而且代码量极少。

先来看看 RxJava 是如何编写一个最基本可用的 EventBus 版本。

```kotlin
object RxBus {
    private val mBus: Subject<Any> = PublishSubject.create<Any>().toSerialized()

    fun post(any: Any) {
        mBus.onNext(any)
    }

    fun <T> toObservable(clazz: Class<T>): Observable<T> = mBus.ofType(clazz)

    fun toObservable(): Observable<Any> = mBus

    fun hasObservers() = mBus.hasObservers()
}
```

在这个最基本的 EventBus 中，我们使用 Subject 来发送事件。Subject 既是 Observable，又是 Observer。

#### 开发一个新的 EventBus（二）

第一个 EventBus 只是完成了最基本的功能，虽然也考虑了多线程的处理，但是没有考虑背压的情况。因为 RxJava 2中，Subject 已经不再支持背压了。

所以，第二个 EventBus 在第一个 EventBus 的基础上增加了对背压的处理，将 Subject 替换成支持背压的 PublishProcessor。

```kotlin
object RxBus {
    private val mBus: FlowableProcessor<Any> = PublishProcessor.create<Any>().toSerialized()

    fun post(any: Any) {
        mBus.onNext(any)
    }

    fun <T> toFlowable(clazz: Class<T>): Flowable<T> = mBus.ofType(clazz)

    fun toFlowable(): Flowable<Any> = mBus

    fun hasSubscribers() = mBus.hasSubscribers()
}
```

#### 开发一个新的 EventBus（三）

RxJava 的操作符在链式调用中一旦有一个抛出了异常，Observer 就会直接执行 onError() 方法，从而导致整个链式调用的结束。这是 RxJava 本身的设计原则。

对于前面两种类型的 EventBus，在订阅者处理事件时如果遇到了异常情况，那么订阅者就会无法再收到事件。这对于 EventBus 而言是一个大麻烦。所以我们有引入 Jake Wharton 大神写的 RxRelay，引入它之后即使出现异常也不会终止订阅关系，从而保证 App 还能正常使用。

RxRelay 的 GitHub 地址：https://github.com/JakeWharton/RxRelay

```kotlin
object RxBus {
    private val mBus: Relay<Any> = PublishRelay.create<Any>().toSerialized()

    fun post(any: Any) {
        mBus.accept(any)
    }

    fun <T> toObservable(clazz: Class<T>): Observable<T> = mBus.ofType(clazz)

    fun toObservable(): Observable<Any> = mBus

    fun hasObservers() = mBus.hasObservers()
}
```

#### 开发一个新的 EventBus（四）

添加 Sticky 事件的 RxBus

```kotlin
object RxBus {
    private val mBus: Relay<Any> = PublishRelay.create<Any>().toSerialized()
    private val mStickyEventMap: MutableMap<Class<*>, Any> = ConcurrentHashMap()

    fun post(event: Any) {
        mBus.accept(event)
    }

    @Synchronized
    fun postSticky(event: Any) {
        mStickyEventMap[event::class.java] = event
        mBus.accept(event)
    }

    fun <T> toObservable(clazz: Class<T>): Observable<T> = mBus.ofType(clazz)

    /**
     * 根据传递的 eventType 类型返回特定类型的被观察者
     */
    @Suppress("UNCHECKED_CAST")
    @Synchronized
    fun <T> toObservableSticky(clazz: Class<T>): Observable<T> {
        val observable = mBus.ofType(clazz)
        val event = mStickyEventMap[clazz]
        if (event != null) {
            return observable.mergeWith(Observable.create { emitter ->
                emitter.onNext(event as T)
            })
        } else {
            return observable
        }
    }

    fun toObservable(): Observable<Any> = mBus

    fun hasObservers() = mBus.hasObservers()

    /**
     * 移除指定 eventType 的 Sticky 事件
     */
    @Suppress("UNCHECKED_CAST")
    @Synchronized
    fun <T> removeStickyEvent(clazz: Class<T>): T? {
        return mStickyEventMap.remove(clazz) as? T
    }

    /**
     * 移除所有的 Sticky 事件
     */
    @Synchronized
    fun removeAllStickyEvents() {
        mStickyEventMap.clear()
    }
}
```

