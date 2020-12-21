### RxJava 的背压

在 RxJava 中，会遇到被观察者发送消息太快以至于它的操作符或者订阅者不能及时处理相关的消息，这就是典型的背压（Back Pressure）场景。

背压是指在异步场景下，被观察者发送事件速度远快于观察者处理的速度，从而导致下游的 buffer 溢出。

在 RxJava 2.x 中，只有新增的 Flowable 类型是支持背压的，并且 Flowable 很多操作符内部都使用了背压策略，从而避免过多的数据填满内部的队列。

RxJava 支持五种背压策略：

1. MISSING

   此策略表示，通过 Create 方法创建的 Flowable 没有指定背压策略，不会对通过 OnNext 发射的数据做缓存或丢弃处理，需要下游通过背压操作符（onBackpressureBuffer()/onBackpressureDrop()/onBackpressureLastest()）指定背压策略。

2. ERROR

   此策略表示，如果放入 Flowable 的异步缓存池中的数据超限了，则会抛出 MissingBackpressureException 异常。

   ```kotlin
   fun clickButton(view: View) {
       Flowable.create(FlowableOnSubscribe<Int> { emitter ->
           (0..129).forEach { emitter.onNext(it) }
       }, BackpressureStrategy.ERROR)
           .subscribeOn(Schedulers.newThread())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe { Log.d("clickButton", "clickButton: $it") }
   }
   ```

   在 Android 中运行这段代码，会立刻引起 App Crash，查看 LogCat 之后发现如下异常：

   ```java
   Caused by: io.reactivex.rxjava3.exceptions.MissingBackpressureException: create: could not emit value due to lack of requests
   ```

3. BUFFER

   此策略表示，Flowable 的异步缓存池同 Observable 的一样，没有固定大小，可以无限制添加数据，不会抛出 MissingBackpressureException 异常，但会导致 OOM。

   ```kotlin
   fun clickButton(view: View) {
       Flowable.create(FlowableOnSubscribe<Int> { emitter ->
           repeat(Int.MAX_VALUE) {
               emitter.onNext(it)
           }
       }, BackpressureStrategy.BUFFER)
           .subscribeOn(Schedulers.newThread())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe { Log.d("clickButton", "clickButton: $it") }
   }
   ```

   上诉代码如果在 Android 中运行的话只会引起ANR。

4. DROP

   此策略表示，如果 Flowable 的异步缓存池满了，则会丢掉将要放入缓存池中的数据。

   ```kotlin
   fun clickButton(view: View) {
       Flowable.create(FlowableOnSubscribe<Int> { emitter ->
           (0..128).forEach { emitter.onNext(it) }
       }, BackpressureStrategy.DROP)
           .subscribeOn(Schedulers.newThread())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe { Log.d("clickButton", "clickButton: $it") }
   }
   ```

   在 Android 中运行这段代码，不会引起 Crash，但只会打印出0~127，第128则被丢弃，因为 Flowable 的内部队列已经满了。

5. LATEST

   此策略表示，如果缓存池满了，会丢掉将要放入缓存池中的数据。这一点与 DROP 策略一样，不同的是，不管缓存池的状态如何，LATEST 策略会将最后一条数据强行放入缓存池中。

   ```kotlin
   fun clickButton(view: View) {
       Flowable.create(FlowableOnSubscribe<Int> { emitter ->
           (0..1000).forEach { emitter.onNext(it) }
       }, BackpressureStrategy.LATEST)
           .subscribeOn(Schedulers.newThread())
           .observeOn(AndroidSchedulers.mainThread())
           .subscribe { Log.d("clickButton", "clickButton: $it") }
   }
   ```

   在 Android 中运行这段代码，也不会引起 Crash，并且会打印出0~127 以及 999。因为 999 是最后一条数据。

   Flowable 不仅可以通过 create 创建时指定背压策略，还可以在其它创建操作符，通过背压操作符指定背压策略。

   ```kotlin
   Flowable.interval(period,TimeUnit.MILLISECONDS)
   	.onBackpressureBuffer()
   	.subscribe { Log.d("clickButton", "clickButton: $it") }
   ```

   

