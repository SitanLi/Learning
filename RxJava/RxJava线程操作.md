### RxJava 线程操作

##### 调度器（Scheduler）种类

​	RxJava 是一个为异步编程而实现的库，合理地利用异步编程能够提高体统的处理速度。在默认情况下，RxJava 只在当前线程中运行，它是单线程的。RxJava 中的多线程实现，使用调度器（Scheduler）来实现。

​	Scheduler 是 RxJava 对线程控制器的一个抽象，RxJava 内置了多个 Scheduler 的实现，它们基本满足绝大多数使用场景，如表所示：

|    Scheduler    |                          作      用                          |
| :-------------: | :----------------------------------------------------------: |
|     single      | 使用定长为1的线程池（`new Scheduled Thread Pool(1)`），重复利用这个线程 |
|    newThread    |            每次都启用新线程，并在新线程中执行操作            |
|   computation   | 使用固定线程池（`Fixed Scheduler Pool`），大小为 CPU 核数，适用于 CPU 密集型计算 |
|       io        | 适合 I/O 操作（读写文件、读写数据库、网络信息交互等）。行为模式和`newThread()`差不多，区别在于 io() 的内部实现是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下，io() 比 newThread() 更有效率 |
|   trampoline    | 直接在当前线程运行，如果当前线程有其他任务正在执行，则会暂停其他任务 |
| Schedulers.from | 将 java.util.concurrent.Executor 转换成一个调度器实例，即可以自定义一个 Excutor 来作为调度器 |

​	如果内置的 Scheduler 不能满足业务需求，那么可以使用自定义的 Executor 作为调度器，以满足个性化需求。

​	下面是使用 Scheduler 的简单示例：

```java
public void helloRxJava1() {
    Observable.<String>create(emitter -> {
        emitter.onNext("hello");
        emitter.onNext("world");
    }).observeOn(Schedulers.newThread())
            .subscribe(s -> {
                System.out.println(Thread.currentThread().getName() + "：" + s);
            });
}

output-->RxNewThreadScheduler-1：hello
output-->RxNewThreadScheduler-1：world
```

##### RxJava 线程模型

​	RxJava 的被观察者们在使用操作符时可以利用线程调度器——Scheduler 来切换线程。例如：

```java
public void helloRxJava2() throws InterruptedException {
    Observable.just("aaa", "bbb")
            .observeOn(Schedulers.newThread())
            .map(s -> {
                System.out.println(threadName() + "map");
                return s.toUpperCase(Locale.getDefault());
            }).subscribeOn(Schedulers.single())
            .observeOn(Schedulers.io())
            .subscribe(s -> {
                System.out.println(threadName() + s);
            });
}
```

​	默认情况下 RxJava 不做任何线程处理，Observable 和 Observer 处于同一线程中。如果想要切换线程，则可以使用 subscribeOn() 和 observeOn()。

1. subscribeOn

   subscribeOn 通过接收一个Scheduler参数，来指定对数据的处理运行在特定的线程调度器Scheduler上，若多次执行subscribeOn，则只有一次起作用。

   查阅 subscribeOn() 的源码可以看到，每次调用 subscribeOn() 都会创建一个 ObservableSubscribeOn 对象。

   ```java
   public final Observable<T> subscribeOn(@NonNull Scheduler scheduler) {
       Objects.requireNonNull(scheduler, "scheduler is null");
       return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<>(this, scheduler));
   }
   ```

   ObservableSubscribeOn 真正发生订阅的方法是`subscribeActual(final Observer<? super T> observer)`：

   ```java
   public void subscribeActual(final Observer<? super T> observer) {
       final SubscribeOnObserver<T> parent = new SubscribeOnObserver<>(observer);
   
       observer.onSubscribe(parent);
   
       parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
   }
   ```

   其中，`SubscribeOnObserver`是下游的 Observer 通过装饰器模式生成的，它实现了Onserver、Disposable 接口。

   接下来，在上游的线程中执行下游 Observer 的 `onSubscribe(Disposable d)`。然后，将子线程的操作加入 Disposable 管理中，加入 Disposable 后可以方便上下游的统一管理：

   ```java
   parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
   ```

   在这里，已经调用了对于 scheduler 的 `scheduleDirect()`方法。传入的是一个Runnable，也就是下面的 SubscribeTask：

   ```java
   final class SubscribeTask implements Runnable {
       private final SubscribeOnObserver<T> parent;
   
       SubscribeTask(SubscribeOnObserver<T> parent) {
           this.parent = parent;
       }
   
       @Override
       public void run() {
           source.subscribe(parent);
       }
   }
   ```

   SubscribeTask 会执行 run() 对上游的 Observable，从而进行订阅。此时，已经在对应的 Scheduler 线程中运行了：

   ```java
   source.subscribe(parent);
   ```

   在 RxJava 的链式操作中，数据的处理是自下而上的，这点与数据发射正好相反。如果多次调用 subscribeOn，则最上面的线程切换最晚执行，所以就变成了只有第一次切换线程才有效。

2. observerOn

   onserveOn 同样接收一个 Scheduler 参数，用来指定下游操作运行在特定的线程调度器 Scheduler 上。

   若多次执行 onserveOn()，则每次都起作用，线程会一直切换。

   查阅 observerOn() 的源码可以看到，每次调用 onserveOn() 都会创建一个ObservableObserveOn对象：

   ```java
   public final Observable<T> observeOn(@NonNull Scheduler scheduler) {
       return observeOn(scheduler, false, bufferSize());
   }
   
   public final Observable<T> observeOn(@NonNull Scheduler scheduler, boolean delayError, int bufferSize) {
       Objects.requireNonNull(scheduler, "scheduler is null");
       ObjectHelper.verifyPositive(bufferSize, "bufferSize");
       return RxJavaPlugins.onAssembly(new ObservableObserveOn<>(this, scheduler, delayError, bufferSize));
   }
   ```

   ObservableObserveOn 真正发生订阅的方法是`subscribeActual(Observer<? super T> observer)`：

   ```java
   protected void subscribeActual(Observer<? super T> observer) {
       if (scheduler instanceof TrampolineScheduler) {
           source.subscribe(observer);
       } else {
           Scheduler.Worker w = scheduler.createWorker();
   
           source.subscribe(new ObserveOnObserver<>(observer, w, delayError, bufferSize));
       }
   }
   ```

   如果 scheduler 是 TrampolineScheduler，则上游事件和下游事件会立即产生订阅。

   如果不是 TrampolineScheduler，则 scheduler 会创建自己的 Worker，然后上游事件和下游事件产生订阅，生成一个 ObserveOnObserver 对象，封装了下游真正的 Observer。

   ObserveOnObserver 是 ObservableObserveOn 的内部类，实现了 Observer、Runnable 接口。在 ObserveOnObserver 的 `onNext()`中，schedule() 执行了具体调度的方法：

   ```java
   public void onNext(T t) {
       if (done) {
           return;
       }
   
       if (sourceMode != QueueDisposable.ASYNC) {
           queue.offer(t);
       }
       schedule();
   }
   
   void schedule() {
       if (getAndIncrement() == 0) {
           worker.schedule(this);
       }
   }
   ```

   其中，worker 是当前 scheduler 创建的 Worker，this 指的是当前的 OnserveOnObserver 对象，this实现了 Runnable 接口，然后，再来看看 Runnable 接口的`run`方法，这个方法是在Worker 对应的线程里执行的。`drainNormal()`会取出 OnserveOnObserver 的`queue`里的数据进行发送：

   ```java
   public void run() {
       if (outputFused) {
           drainFused();
       } else {
           drainNormal();
       }
   }
   ```

   若下游多次调用 onserveOn()，则线程会一直切换。每次切换线程，都会把对应的 Onserver 对象的各个方法的处理执行在指定的线程中。

3. 实例：

   - 单独使用 subscribeOn：

   ```java
   public void helloRxJava3() throws InterruptedException {
       Observable.<String>create(emitter -> {
           System.out.println(threadName() + "emitter");
           emitter.onNext("hello");
           emitter.onNext("world");
       }).subscribeOn(Schedulers.newThread())
               .subscribe(s -> {
                   System.out.println(threadName() + s);
               });
   }
   
   output-->RxNewThreadScheduler-1：emitter
   output-->RxNewThreadScheduler-1：hello
   output-->RxNewThreadScheduler-1：world
   ```

   此时，所有的操作都是在 newThread 中运行的，包括发射数据。

   - 多次切换线程

   最后，举一个多次调用 subscribeOn 和 onserveOn 的例子：

   ```java
   public void helloRxJava4() throws InterruptedException {
       Observable.just("Hello World")
               .subscribeOn(Schedulers.single())
               .map(s -> {
                   s = s.toLowerCase();
                   System.out.println(threadName() + "map1-->" + s);//single
                   return s;
               }).observeOn(Schedulers.io())
               .map(s -> {
                   s = s + "tony.";
                   System.out.println(threadName() + "map2-->" + s);//io
                   return s;
               }).subscribeOn(Schedulers.computation())
               .map(s -> {
                   s = s + "it is a test.";
                   System.out.println(threadName() + "map3-->" + s);//io
                   return s;
               }).observeOn(Schedulers.newThread())
               .subscribe(s -> {
                   System.out.println(threadName() + "subscribe" + s);//newThread
               });
       Thread.sleep(1000);
   }
   
   output-->RxSingleScheduler-1：map1-->hello world
   output-->RxCachedThreadScheduler-1：map2-->hello worldtony.
   output-->RxCachedThreadScheduler-1：map3-->hello worldtony.it is a test.
   output-->RxNewThreadScheduler-1：subscribehello worldtony.it is a test.
   ```

   这个示例中第一个`subscribeOn(Schedulers.single())`影响了 map1 的线程调度，之后的线程均是通过 `observeOn` 进行调度的。

##### Scheduler 的测试

​	TestScheduler 是专门用于测试的调度器，与其他调度器的区别是，TestScheduler 只有被调用了时间才会继续。TestScheduler 是一种特殊的、非线程安全的调度器，用于测试一些不引入真实并发性、允许手动推进虚拟时间的调度器。

​	TestScheduler 所包含的方法并不多，下面罗列几个关键方法。

1. advanceTimeTo：将调度器的时钟移动到某个特定时刻。

   例如，时钟移动到10ms：

   ```java
   scheduler.advanceTimeTo(10,TimeUnit.MILLISECONDS)
   ```

   下面的例子展示了0s、20s、40s各会打印什么结果：

   ```java
   public void helloRxJava5() {
       TestScheduler scheduler = new TestScheduler();
       scheduler.createWorker().schedule(() -> {
           System.out.println("immediate");
       });
       scheduler.createWorker().schedule(() -> {
           System.out.println("20s");
       }, 20, TimeUnit.SECONDS);
       scheduler.createWorker().schedule(() -> {
           System.out.println("40s");
       }, 40, TimeUnit.SECONDS);
   
       scheduler.advanceTimeTo(1, TimeUnit.MILLISECONDS);
       System.out.println("Virtual time：" + scheduler.now(TimeUnit.MILLISECONDS));
   
       scheduler.advanceTimeTo(20, TimeUnit.SECONDS);
       System.out.println("Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   
       scheduler.advanceTimeTo(40, TimeUnit.SECONDS);
       System.out.println("Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   }
   output-->immediate
   output-->Virtual time：1
   output-->20s
   output-->Virtual time：20
   output-->40s
   output-->Virtual time：40
   ```

   可以看到使用`advanceTimeTo`之后，移动不同的时间点会打印不同的内容。

   当然，`advanceTimeTo()`也可以传负数，表示回到过去的时间点，但是一般不推荐这种用法。

2. advanceTimeBy：将调度程序的时钟按指定的时间向前移动。

   例如，时钟移动了10ms：

   ```java
   scheduler.advanceTimeBy(10,TimeUnit.MILLISECONDS);
   ```

   再次调用刚才的方法，时钟又会移动10ms。此时，时钟移动到20ms，这是一个累加的过程。

   下面的例子使用了`timer`操作符，该例子展示了2s后amomicLong会自动加1：

   ```java
   public void helloRxJava6() {
       TestScheduler scheduler = new TestScheduler();
       final AtomicLong atomicLong = new AtomicLong();
       Observable.timer(2, TimeUnit.SECONDS, scheduler)
               .subscribe(aLong -> atomicLong.incrementAndGet());
   
       System.out.println("atomicLong's value = " + atomicLong.get() + "，Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   
       scheduler.advanceTimeBy(1, TimeUnit.SECONDS);
       System.out.println("atomicLong's value = " + atomicLong.get() + "，Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   
       scheduler.advanceTimeBy(-1, TimeUnit.SECONDS);
       System.out.println("atomicLong's value = " + atomicLong.get() + "，Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   
       scheduler.advanceTimeBy(2, TimeUnit.SECONDS);
       System.out.println("atomicLong's value = " + atomicLong.get() + "，Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   }
   
   output-->atomicLong's value = 0，Virtual time：0
   output-->atomicLong's value = 0，Virtual time：1
   output-->atomicLong's value = 0，Virtual time：0
   output-->atomicLong's value = 1，Virtual time：2
   ```

3. triggerActions：它执行计划中的但是未启动的任务，已经执行过的任务不会再启动：

   ```java
   public void helloRxJava7() {
       TestScheduler scheduler = new TestScheduler();
       scheduler.createWorker().schedule(() -> {
           System.out.println("immediate");
       });
       scheduler.createWorker().schedule(() -> {
           System.out.println("20s");
       }, 20, TimeUnit.SECONDS);
   
       scheduler.triggerActions();
       System.out.println("Virtual time：" + scheduler.now(TimeUnit.SECONDS));
   
       scheduler.advanceTimeBy(20,TimeUnit.SECONDS);
       scheduler.triggerActions();
   }
   
   output-->immediate
   output-->Virtual time：0
   output-->20s
   ```

   因为已经使用了 advanceTimeBy()，所以即使再调用 triggerActions()，也不会执行已经启动过的任务。

