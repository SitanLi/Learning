### RxJava 的并行编程

##### RxJava 并行操作

被观察者（Observable/Flowable/Single/Completable/Maybe）发射的数据流可以经历各种线程切换，但是数据流的各个元素之间不会产生并行执行的效果。并行不是并发，也不是同步，更不是异步。

并发是指一个处理器同时处理多个任务。并行是多个处理器或者是多核处理器同时处理多个不同的任务。并行是同时发生的多个并发时间，具有并发的含义，而并发不一定是并行。

Java8 新增了并行流来实现并行的效果，只需要在集合上调用 parallelStream() 方法即可。

```java
public void helloRxJava1() {
    List<Integer> result = new ArrayList();
    for (int i = 0; i < 100; i++) {
        result.add(i);
    }

    result.parallelStream()
            .map(Object::toString)
            .forEach(s -> System.out.println("s = " + s + ";Current Thread Name=" + Thread.currentThread().getName()));
}

output-->s = 67;Current Thread Name=main
output-->s = 68;Current Thread Name=main
output-->s = 31;Current Thread Name=ForkJoinPool.commonPool-worker-2
output-->s = 12;Current Thread Name=ForkJoinPool.commonPool-worker-1
```

从执行结果中可以看到，Java 8 借助了 JDK 的 fork/join 框架来实现并行编程。

1. ###### 借助 flatMap 实现并行

   在 RxJava 中可以借助 flatMap 操作符实现类似于 Java8 的并行执行效果。

   ```java
   public void helloRxJava2() {
       Observable.range(1, 100)
               .flatMap(integer -> Observable.just(integer)
                       .subscribeOn(Schedulers.computation())
                       .map(Object::toString))
               .subscribe(System.out::println);
   }
   ```

   flatMap 操作符的原理是将这个 Observable 转化为多个以原 Observable 发射的数据作为源数据 Observable，然后再将这多个 Observable 发射的数据整合发射出来。需要注意的是，最后的顺序可能会交错地发射出来。

   flatMap 会对原始 Observable 发射的每一项数据执行变换操作。在这里，生成的每个 Observable 使用线程池并发地执行。

   当然，我们还可以使用 ExecutorService 来创建一个 Scheduler，对刚才的代码稍微做一些改动：

   ```java
   public void helloRxJava3() {
       int threadNum = Runtime.getRuntime().availableProcessors() + 1;
       ExecutorService executor = Executors.newFixedThreadPool(threadNum);
       Scheduler scheduler = Schedulers.from(executor);
   
       Observable.range(1, 100)
               .flatMap(integer -> Observable.just(integer)
                       .subscribeOn(scheduler)
                       .map(Object::toString))
               //完成所有操作后关闭线程池
               .doFinally(executor::shutdown)
               .subscribe(System.out::println);
   }
   ```

2. ###### 通过 Round-Robin 算法实现并行

   Round-Robin 算法是最简单的一种负载均衡算法。它的原理是把来自用户的请求轮流分配给内部的服务器：从服务器 1 开始，知道服务器 N ，然后重新开始循环，也被称为哈希取模法，是非常常用的数据分片方法。Round-Robin 算法的优点是简洁，它无需记录当前所有连接的状态，所以是一种无状态调度。

   通过 Round-Robin 算法把数据按线程数分组，例如分成 5 组，每组个数相同，一起发送处理。这样做的目的是可以减少 Observable 的创建，从而节省系统资源，但是会增加处理时间。Round-Robin 算法可以看成是对实践和空间的综合考虑。

   ```java
   public void helloRxJava4() {
       final AtomicInteger batch = new AtomicInteger(0);
   
       Observable.range(1, 100)
               .groupBy(integer -> batch.getAndIncrement() % 5)
               .flatMap(integerIntegerGroupedObservable ->
                 integerIntegerGroupedObservable.observeOn(Schedulers.computation())
                               .map(Objects::toString)
               )
               .subscribe(System.out::println);
   }
   ```

##### ParallelFlowable

RxJava 2.0.5 版本新增了 ParallelFlowable API，它允许并行地执行一些操作符，例如 map、filter、concatMap、flatMap、collect、reduce等。

ParallellFlowable 是并行的 Flowable 版本，并不是新增的被观察者类型。在 ParallelFlowable 中，很多典型的操作符（如take、skip等）是不可用的。

在并行处理中背压是必不可少的，否则会淹没在并行操作符的内部队列中，因此不存在ParallelObservable、ParallelSingle、ParallelCompletable 和 ParallelMaybe。

1. ###### ParallelFlowable 实现并行

   类似于 Java 8 的并行流，在相应的操作符上调用 Flowable 的 parallel() 就会返回 ParallelFlowable。

   ```java
   public void helloRxJava5() {
       ParallelFlowable<Integer> parallelFlowable = Flowable.range(1, 100).parallel();
       parallelFlowable.runOn(Schedulers.computation())
               .map(Objects::toString)
               .sequential()
               .subscribe(System.out::println);
   }
   ```

   其中，parallel() 调用了`ParallelFlowable.from(Publisher<? extends T> source)`

   ```java
   public final ParallelFlowable<T> parallel() {
       return ParallelFlowable.from(this);
   }
   ```

   ParallelFlowable 的 from() 方法是通过 Publish 并以循环的方式在多个“轨道”（CPU数）上消费它的。

   ```java
   public static <@NonNull T> ParallelFlowable<T> from(@NonNull Publisher<@NonNull ? extends T> source) {
       return from(source, Runtime.getRuntime().availableProcessors(), Flowable.bufferSize());
   }
   ```

   默认情况下，并行级别被设置为可用 CPU 的数量（`Runtime.getRuntime().availableProcessors()`），并且顺序源的预取量设置为`Flowable.bufferSize()`。两者都可以通过重载 parallel() 方法来指定。

   ```java
   public final ParallelFlowable<T> parallel(int parallelism) {
       return ParallelFlowable.from(this, parallelism);
   }
   
   public final ParallelFlowable<T> parallel(int parallelism, int prefetch) {
       return ParallelFlowable.from(this, parallelism, prefetch);
   }
   ```

   最后，如果已经使用了必要的并行操作，则可以通过 `ParallelFlowable.sequential()` 操作符返回到顺序流。

2. ###### ParallelFlowable 与 Scheduler

   ParallelFlowable 遵循与 Flowable 相同的异步原理，因此 parallel() 本身并不引入顺序源的异步消耗，只准备并行流，但是可以通过 runOn(Scheduler) 操作符定义异步。这点与 Flowable 有很大不同，Flowable 使用 subscribeOn、observeOn 操作符。

   ```java
   ParallelFlowable<Integer> psource = source.runOn(Schedulers.io());
   ```

   runOn() 可以指定 prefetch 的数量。

   ```java
   public final ParallelFlowable<T> runOn(@NonNull Scheduler scheduler) {
       return runOn(scheduler, Flowable.bufferSize());
   }
   
   public final ParallelFlowable<T> runOn(@NonNull Scheduler scheduler, int prefetch) {
       Objects.requireNonNull(scheduler, "scheduler is null");
       ObjectHelper.verifyPositive(prefetch, "prefetch");
       return RxJavaPlugins.onAssembly(new ParallelRunOn<>(this, scheduler, prefetch));
   }
   ```

##### ParallelFlowable 的操作符

并非所有的顺序操作在并行世界中都是有意义的。目前ParallelFlowable只支持如下操作：

map、filter、flatMap、concatMap、reduce、collect、sorted、toSortedList、compose、fromArray、doOnCancel、doOnError、doOnComplete、doOnNext、doAfterNext、doOnSubscribe、doAfterTerminated、doOnRequest

这些 ParallelFlowable 可用的操作符，使用方法与 Flowable 中的方法一样。

##### ParallelFlowable 和 Flowable.flatMap 比较

Flowable.flatMap 实现并行的原理和 Observable.flatMap 实现并行的原理相同。

那么什么时候使用 flatMap 进行处理比较好，什么时候使用 ParallelFlowable 比较好呢？

RxJava 本质上是连续的，借助 flatMap 操作符进行分离和加入一个序列可能会变得很复杂，并引起一定的开销。但是如果使用 ParalleFlowable，则开销会更小。

然而，ParallelFlowable 的操作符很有限，如果有一些特殊的操作需要并行执行，而这些操作不能用 ParalleFlowable 所支持的操作符来表达，那么就应该使用基于 Flowable.flatMap 来实现。

因此，优先推荐使用 ParallelFlowable，对于无法使用 ParallelFlowable 的操作符，则可以使用 flatMap 来实现并行。