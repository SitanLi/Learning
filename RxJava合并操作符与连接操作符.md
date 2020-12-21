### RxJava 合并操作符与连接操作符

RxJava 的合并操作符主要包括如下几个：

- startWith()：在数据序列开头增加一项数据。
- merge()：将多个 Observable 合并为一个。
- mergeDelayError()：合并多个 Observable，让没有错误的 Observable 都完成后再发射错误通知。
- zip()：使用一个函数组合多个 Observable 发射的数据集合，然后再发射这个结果。
- combineLastest()：当两个 Observable 中的任何一个发射了一个数据时，通过一个指定的函数组合每个 Observable 发射的最新数据（一共两个数据），然后发射这个函数的结果。
- join() and groupJoin()：无论何时，如果一个 Observable 发射了一个数据项，就需要在另一个 Observable 发射的数据项定义的时间窗口内，将两个 Observable 发射的数据合并发射。
- switchOnNext()：将一个发射的 Observable 的 Observable 转换成另一个 Observable，后者发射这些 Observable 最近发射的数据。

RxJava 的连接操作符，主要是 ConnectableObservable 所使用的操作符和 Observable 所使用的操作符。

- ConnectableObservable.connect()：指示一个可连接的 Observable 开始发射数据。
- Observable.publish()：将一个 Observable 转换为一个可连接的 Observable。
- Observable.replay()：确保所有的订阅者看到相同的数据序列，即使它们在 Observable 开始发射数据之后才订阅。
- ConnectableObservable.refCount()：让一个可连接的 Observable 表现得像一个普通的 Observable。

##### merge 和 zip

1. merge 操作符：将多个 Observable 的输出合并，使得它们就像是个单个的 Observable 一样。

   ```java
   public void helloRxJava1() {
       Observable<Integer> odds = Observable.just(1, 3, 5);
       Observable<Integer> evens = Observable.just(2, 4, 6);
   
       Observable.merge(odds, evens)
               .subscribe(System.out::println);
   }
   
   output-->1
   output-->3
   output-->5
   output-->2
   output-->4
   output-->6
   ```

   merge 是按照时间线并行的。如果传递给 merge 的任何一个 Observable 发射了 onError 通知终止，则 merge 操作符生成的 Observable 也会立即以 onError 通知终止。如果想让它继续发射数据，直到最后才报告错误，则可以使用 mergeDelayError 操作符。

   如果只是两个被观察者合并，则还可以使用 mergeWith 操作符，Observable.merge(odds,evens) 等价于 odds.mergeWith(evens)。

   merge 操作符最多只能合并 4 个被观察者，如果需要合并更多个被观察者，则可以使用 mergeArray 操作符。

2. zip：zip操作符返回一个 Observable，它使用一个函数按顺序结合两个或多个 Observable 发射的数据项，然后发射这个函数返回的结果。它按照严格的顺序应用这个函数，只发射与发射数据项最少的那个 Observable 一样多的数据。

   zip 的最后一个参数接收每个 Observable 发射的一项数据，返回被压缩后的数据，它可以接收1-9个参数：

   ```java
   public void helloRxJava2() {
       Observable<Integer> odds = Observable.just(1, 3, 5);
       Observable<Integer> evens = Observable.just(2, 4, 6);
   
       Observable.zip(odds, evens, Integer::sum)
               .subscribe(System.out::println);
   }
   
   output-->3
   output-->7
   output-->11
   ```

   zip 操作符相对于 merge 操作符，除发射数据外，还会进行合并操作。还有一点需要注意的是，BiFunction 相当于一个合并函数，并不一定要返回 Integer 类型，可以根据业务需要返回合适的类型。

##### combineLastest 和 join

combineLastest 操作符的行为类似于 zip，但是只有当原始的 Observable 中的每一个都发射了一条数据时 zip 才发射数据，而 combineLastest 则是当原始的 Observable 中任意一个发射了数据时就发射一条数据。当原始 Observables 的任何一个发射了一条数据时，combineLastest 使用一个函数结合它们最近发射的数据，然后发射这个函数的返回值。

```java
public void helloRxJava2() {
    Observable<Integer> odds = Observable.just(1, 3, 5);
    Observable<Integer> evens = Observable.just(2, 4, 6);

    Observable.zip(odds, evens, Integer::sum)
            .subscribe(System.out::println);
}

output-->7
output-->9
output-->11
```

join 操作符结合两个 Observable 发射的数据，基于时间窗口（针对每条数据特定的原则）选择待集合的数据项。将这些时间窗口实现为一些 Observables，它们的生命周期从任何一条 Observable 发射的每一条数据开始。当这个定义时间窗口的 Observable 发射了一条数据或者完成时，与这条数据关联的窗口也会关闭。只要这条数据的窗口是打开的，它就继续结合其他 Observable 发射的任何数据项。

```java
public void helloRxJava4() {
    Observable<Integer> odds = Observable.just(1, 2, 3);
    Observable<Integer> evens = Observable.just(4, 5, 6);

    odds.join(evens,
            integer -> Observable.just(String.valueOf(integer)).delay(200, TimeUnit.MILLISECONDS),
            integer -> Observable.just(String.valueOf(integer)).delay(200, TimeUnit.MILLISECONDS),
            (BiFunction<Integer, Integer, String>) (integer, integer2) -> integer + "：" + integer2)
            .subscribe(System.out::println);
}

output-->1：4
output-->2：4
output-->3：4
output-->1：5
output-->2：5
output-->3：5
output-->1：6
output-->2：6
output-->3：6
```

`join(Observable,Function,Function,BiFunction)`有四个参数，下面分别解释它们的用途：

- Observable：源 Observable 需要组合的 Observable，这里可以称之为目标 Observable。
- Function：接收从源 Observable 发射来的数据，并返回一个 Observable，这个 Observable 的生命周期决定了源 Observable 发射数据的有效期。
- Function：接收目标 Observable 发射的数据，并返回一个 Observable，这个 Observable 的生命周期决定了目标 Observable 发射数据的有效期。
- BiFunction：接收从源 Observable 和目标 Observable 发射的数据，并将这两个数据组合后返回。

join 操作符的效果类似于排列组合，把第一个数据源 A 作为基座窗口，它根据自己的节奏不断发射数据元素；第二个数据源 B，每发射一个数据，我们都把它和第一个数据源 A 中已经发射的数据进行一对一匹配。

##### startWith

如果想让一个 Observable 在发射数据之前先发射一个指定的数据序列，则可以使用 startWith 操作符。如果想在一个 Observable 发射数据的末尾追加一个数据序列，则可以使用 concat 操作符。

```java
public void helloRxJava5() {
    Observable.just("Hello Java", "Hello Kotlin", "Hello Scala")
            .startWith(Observable.just("Hello Rx"))
            .subscribe(System.out::println);
}

output-->Hello Rx
output-->Hello Java
output-->Hello Kotlin
output-->Hello Scala
```

##### connect、push 和 refCount

connect 和 refCount 是 ConnectableObservable 所使用的操作符，ConnectableObservable 继承自 Observable，然后它并不是在调用 `subscribe()`的时候发射数据，而是只有对其使用 connect 操作符时它才会发射数据，所以可以用来更灵活地控制数据发射的时机。ConnectableObservable  是 Hot Observable。

push 操作符是将普通的 Observable 转换成 ConnectableObservable 。

connect 操作符用来触发 ConnectableObservable 发射数据。我们可以等所有的观察者都订阅了 ConnectableObservable 之后再发射数据。

```java
public void helloRxJava6() throws InterruptedException {
    SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
    Observable<Long> obs = Observable.interval(1, TimeUnit.SECONDS).take(6);
    ConnectableObservable<Long> connectableObservable = obs.publish();
    connectableObservable.subscribe(aLong -> System.out.println("Subscribe1:onNext:" + aLong + "->time:" + sdf.format(new Date())));
    connectableObservable.delaySubscription(3, TimeUnit.SECONDS)
            .subscribe(aLong -> System.out.println("Subscribe2:onNext:" + aLong + "->time:" + sdf.format(new Date())));
    connectableObservable.connect();

    Thread.sleep(6000);
}

output-->Subscribe1:onNext:0->time:11:27:57
output-->Subscribe1:onNext:1->time:11:27:58
output-->Subscribe1:onNext:2->time:11:27:59
output-->Subscribe2:onNext:2->time:11:27:59
output-->Subscribe1:onNext:3->time:11:28:00
output-->Subscribe2:onNext:3->time:11:28:00
output-->Subscribe1:onNext:4->time:11:28:01
output-->Subscribe2:onNext:4->time:11:28:01
output-->Subscribe1:onNext:5->time:11:28:02
output-->Subscribe2:onNext:5->time:11:28:02
```

refCount 操作符是将 ConnectableObservable 转换成普通的 Observable，同时又保持了 Hot Observable 的特性。当出现第一个订阅者时，refCount 会调用 connect()。每个订阅者每次都会接收到同样的数据，但是当所有订阅者都取消订阅时，refCount 会自动 dipose 上游的 Observable。

所有的订阅者都取消订阅后，则数据流停止。如果重新订阅则数据流重新开始。如果不是所有的订阅者都取消了订阅，而是取消了部分，则部分订阅者/观察者重新开始订阅时，不会从头开始数据流。

```java
public void helloRxJava7() throws InterruptedException {
    SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
    Observable<Long> obs = Observable.interval(1, TimeUnit.SECONDS).take(6);
    ConnectableObservable<Long> connectableObservable = obs.publish();
    Observable<Long> obsRefCount = connectableObservable.refCount();

    obs.subscribe(aLong -> System.out.println("Subscribe1:onNext:" + aLong + "->time:" + sdf.format(new Date())));
    obs.delaySubscription(3, TimeUnit.SECONDS).subscribe(aLong -> System.out.println("Subscribe2:onNext:" + aLong + "->time:" + sdf.format(new Date())));

    obsRefCount.subscribe(aLong -> System.out.println("obsRefCount1:onNext:" + aLong + "->time:" + sdf.format(new Date())));
    obsRefCount.delaySubscription(3, TimeUnit.SECONDS).subscribe(aLong -> System.out.println("obsRefCount2:onNext:" + aLong + "->time:" + sdf.format(new Date())));

    Thread.sleep(15000);
}

output-->Subscribe1:onNext:0->time:11:42:04
output-->obsRefCount1:onNext:0->time:11:42:04
output-->Subscribe1:onNext:1->time:11:42:05
output-->obsRefCount1:onNext:1->time:11:42:05
output-->Subscribe1:onNext:2->time:11:42:06
output-->obsRefCount1:onNext:2->time:11:42:06
output-->Subscribe1:onNext:3->time:11:42:07
output-->Subscribe2:onNext:0->time:11:42:07
output-->obsRefCount1:onNext:3->time:11:42:07
output-->obsRefCount2:onNext:3->time:11:42:07
output-->Subscribe1:onNext:4->time:11:42:08
output-->obsRefCount1:onNext:4->time:11:42:08
output-->obsRefCount2:onNext:4->time:11:42:08
output-->Subscribe2:onNext:1->time:11:42:08
output-->Subscribe1:onNext:5->time:11:42:09
output-->Subscribe2:onNext:2->time:11:42:09
output-->obsRefCount1:onNext:5->time:11:42:09
output-->obsRefCount2:onNext:5->time:11:42:09
output-->Subscribe2:onNext:3->time:11:42:10
output-->Subscribe2:onNext:4->time:11:42:11
output-->Subscribe2:onNext:5->time:11:42:12
```

##### replay

replay 操作符返回一个 ConnectableObservable 对象，并且可以缓存发射过的数据，这样即使订阅者在发射数据之后进行订阅，也能收到之前发射过的数据。不过使用 replay 操作符最好还是先限定缓存的大小，否则缓存的数据太多时会占用很大一块内存。

```java
public void helloRxJava8() throws InterruptedException {
    SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
    Observable<Long> obs = Observable.interval(1, TimeUnit.SECONDS).take(6);
    ConnectableObservable<Long> connectableObservable = obs.replay();
    connectableObservable.connect();
    connectableObservable.subscribe(aLong -> System.out.println("Subscribe1:onNext:" + aLong + "->time:" + sdf.format(new Date())));
    connectableObservable.delaySubscription(3, TimeUnit.SECONDS).subscribe(aLong -> System.out.println("Subscribe2:onNext:" + aLong + "->time:" + sdf.format(new Date())));

    Thread.sleep(15000);
}

output-->Subscribe1:onNext:0->time:11:47:41
output-->Subscribe1:onNext:1->time:11:47:42
output-->Subscribe1:onNext:2->time:11:47:43
output-->Subscribe2:onNext:0->time:11:47:43
output-->Subscribe2:onNext:1->time:11:47:43
output-->Subscribe2:onNext:2->time:11:47:43
output-->Subscribe1:onNext:3->time:11:47:44
output-->Subscribe2:onNext:3->time:11:47:44
output-->Subscribe1:onNext:4->time:11:47:45
output-->Subscribe2:onNext:4->time:11:47:45
output-->Subscribe1:onNext:5->time:11:47:46
output-->Subscribe2:onNext:5->time:11:47:46
```

replay 有多个接收不同参数的重载方法，有的可以指定 replay 的最大缓存数量，有的可以指定调度器。

ConnectableObservable 的线程切换只能通过 replay 操作符实现，普通 Observable 的 subscribeOn() 和 onserverOn() 在 ConnectableObservable 中不起作用。replay 操作符可以通过指定线程的方式来切换线程。