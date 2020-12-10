### RxJava 创建操作符

​	操作符是RxJava的重要组成部分。RxJava的操作符大致可以按照下图分类：

![RxJava常用操作符](pic\RxJava常用操作符.png)

​	RxJava 的创建操作符主要包括如下内容：

- just()：将一个或多个对象转换成发射这个或这些对象的一个Observable。
- from()：将一个Iterable、一个Future或者一个数组转换正一个Observable。
- create()：使用一个函数从头创建一个Observable。
- defer()：只有当订阅者订阅才创建Observable，为每个订阅者创建一个新的Observable。
- range()：创建一个发射指定范围的整数序列的Observable。
- interval()：创建一个按照给定时间间隔发射整数序列的Observable。
- timer()：创建一个在给定的延时之后发射单个数据的Observable。
- empty()：创建一个什么都不做直接通知完成的Observable。
- error()：创建一个什么都不做直接通知错误的Observable。
- never()：创建一个不发射任何数据的Observable。

##### create、just 和 from

​	create：使用 create 操作符从头开始创建一个 Observable，给这个操作符传递一个接受观察者作为参数的函数，恰当地调用`onNext`、`onError`、`onComplete`方法。一个正确的创建方法必须尝试调用观察者的`onComplete()`一次或者它的`onError()`一次，而且此后不能在调用观察者的任何其他方法。

​	RxJava 建议我们在传递给 create 方法的函数时，先检查一下观察者的 isDisposed 状态，以便在没有观察者的时候，让 Observable 停止发射数据，防止运算昂贵：

```java
public void helloRxJava1() {
    Observable.create(emitter -> {
        try {
            if (!emitter.isDisposed()) {
                for (int i = 0; i < 10; i++) {
                    emitter.onNext(i);
                }
                emitter.onComplete();
            }
        } catch (Exception e) {
            emitter.onError(e);
        }
    }).subscribe(o -> {
        System.out.println("Next：" + o);
    }, throwable -> {
        System.out.println("Error：" + throwable.getMessage());
    }, () -> {
        System.out.println("Sequence complete.");
    });
}
```

​	just：创建一个发射指定值的 Observable：

```java
public void helloRxJava2() {
    Observable.just("hello just").subscribe(System.out::println);
}
```

​	just 类似于 from，但是 from 会将数组或 Iterable 的数据取出然后逐个发射，而 just 只是简单地原样发射，将数组或 Iterable 当作单个数据。它可以接受一至十个参数，返回一个按参数列表顺序发射这些数据的Observable。

​	在RxJava 2.0中，如果在just()中传入null，则会抛出空指针异常（`NullPointerException`）。

​	from：在RxJava中，from 操作符可以将`Future`、`Iterable`和数组转换成Observable。对于Iterable 和数组，产生的Observable会发射 Iterable 或数组的每一项数据：

```java
public void helloRxJava3() {
    Observable.fromArray("hello", "from").subscribe(System.out::println);
}

public void helloRxJava4() {
    List<Integer> items = new ArrayList<>();
    for (int i = 0; i < 10; i++) {
        items.add(i);
    }
    Observable.fromIterable(items)
            .subscribe(integer -> {
                System.out.println("Next：" + integer);
            }, throwable -> {
                System.out.println(throwable.getMessage());
            }, () -> {
                System.out.println("Sequence complete.");
            });
}
```

​	对于 Future，它会发射Future.get()方法返回的单个数据：

```java
public void helloRxJava5() {
    ExecutorService executorService = Executors.newCachedThreadPool();
    Future<String> future = executorService.submit(new MyCallable());
    Observable.fromFuture(future)
            .subscribe(System.out::println);
}

static class MyCallable implements Callable<String> {

    @Override
    public String call() throws Exception {
        System.out.println("模拟一些耗时任务...");
        Thread.sleep(5000);
        return "OK";
    }
}
```

​	`fromFuture()`还有一个可接受两个可选参数的版本，分别指定超时时长和时间单位。如果过了指定的超时时长，Future 还没有返回一个值，那么这个 Observable 就会发射错误通知并终止：

```java
Observable.fromFuture(future, 4, TimeUnit.SECONDS)
                .subscribe(System.out::println);

output-->java.util.concurrent.TimeoutException
```

##### repeat

​	创建一个发射特定数据重复多次的Observable，repeat不是创建一个Observable，而是重复发射原始Observable的数据序列，这个序列或者是无限的，或者是通过`repeat(n)`指定重复次数：

```java
public void helloRxJava6() {
    Observable.just("hello repeat")
            .repeat(3) //指定次数
            .subscribe(System.out::println);
}
```

​	在 RxJava 2.x 中还有两个 repeat 相关的操作符：`repeatWhen`和`repeatUntil`。

​	`repeatWhen`是有条件地重新订阅和发射原来的Observable。将原始 Observable 的终止通知（完成或错误）当作一个 void 数据传递给一个通知处理器，以此来决定是否要重新订阅和发射原来的 Observable。这个通知处理器就像一个 Observable 操作符，接受一个发射 Void 通知的 Observable作为输入，返回一个发射 Void 数据（意思是，重新订阅和发射原始Observable）或者直接终止（即使用repeatWhen终止发射数据）的Observable：

```java
public void helloRxJava7() throws InterruptedException {
    Observable.range(0, 9)
            .repeatWhen(objectObservable -> Observable.timer(10, TimeUnit.SECONDS))
            .subscribe(System.out::println);

    Thread.sleep(12000);
}
```

​	在这里，会先发射0到8这9个数字，由于使用了repeatWhen操作符，因此在10s之后还会再发射一次这些数据。

​	repeatUntil 是 RxJava 2.x 新增的操作符，表示直到某个条件就不再重复发射数据。当`BooleanSupplier`的`getAsBoolean()`返回false时，表示重复发射上游的Observable；当`getAsBoolean()`为true时，表示终止重复发射上游的Observable：

```java
public void helloRxJava8() throws InterruptedException {
    final long startTimeMillis = System.currentTimeMillis();
    Observable.interval(500, TimeUnit.MILLISECONDS)
            .take(3)
            .repeatUntil(() -> {
                //返回true时，终止重复发射上游数据
                return System.currentTimeMillis() - startTimeMillis > 5000;
            })
            .subscribe(System.out::println);

    Thread.sleep(10000);
}
```

##### defer、interval 和 timer

​	defer 操作符会一直等待直到有观察者订阅它，然后它使用 Observable 工厂方法生成一个Observable：

```java
public void helloRxJava9() {
    Observable.defer(() -> Observable.just("hello defer1", "hello defer2"))
            .subscribe(System.out::println);
}
```

​	interval 操作符创建一个按固定时间间隔发射整数序列的Observable，interval接受一个表示时间间隔的参数和一个时间单位的参数。interval 默认在 computation 调度器上执行：

```java
public void helloRxJava10() throws InterruptedException {
    Observable.interval(1, TimeUnit.SECONDS)
            .subscribe(System.out::println);
    Thread.sleep(10000);
}
```

​	timer 操作符创建一个 Observable，它在一个给定的延迟后发射一个特殊的值，timer 操作符默认在 computation 调度器上执行：

```java
public void helloRxJava11() throws InterruptedException {
    Observable.timer(2, TimeUnit.SECONDS)
            .subscribe(aLong -> {
                System.out.println("hello timer");
            });
    Thread.sleep(10000);
}
```

