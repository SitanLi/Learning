### RxJava 变换操作符和过滤操作符

RxJava 的变换操作符主要包括以下几种：

- map()：对序列的每一项都用一个函数来变换 Observable 发射的数据序列。
- flatMap()、concatMap()和flatMapIterable()：将 Observable 发射的数据集合变换为 Observables 集合，然后将这些 Observable 发射的数据平坦化地放进一个单独的 Observable 中。
- switchMap()：将 Observable 发射的数据集合变换为 Observables 集合，然后只发射这些 Observables 最近发射过的数据。
- scan()：对 Observable 发射的每一项数据应用一个函数，然后按顺序依次发射每一个值。
- groupBy()：将 Observable 拆分为 Observable 集合，将原始 Observable 发射的数据按 Key 分组，每一个 Observable 发射过一组不同的数据。
- buffer()：定期从 Observable 收集数据到一个集合，然后把这些数据集合打包发射，而不是一次发射一个。
- window()：定期将来自 Observable 的数据拆分成一些 Observable 窗口，然后发射这些窗口，而不是每次发射一项。
- cast()：在发射之前强制将 Observable 发射的所有数据转换为指定类型。

RxJava 的过滤操作符主要包括以下几种：

- filter()：过滤数据。
- takeLast()：只发射最后的*N*项数据。
- last()：只发射最后一项数据。
- lastOrDefault()：只发射最后一项数据，如果 Observable 为空，就发射默认值。
- takeLastBuffer()：将最后的*N*项数据当作单个数据发射。
- skip()：跳过开始的*N*项数据。
- skipLast()：跳过最后的*N*项数据。
- take()：只发射开始的*N*项数据。
- first()、takeFirst()：只发射第一项数据，或者满足条件的第一项数据。
- firstOrDefault()：只发射第一项数据，如果 Observable 为空，就发射默认值、
- elementAt()：发射第*N*项数据。
- elementAtOrDefault()：发射第*N*项数据，如果 Observable 数据少于*N*项，就发射默认值。
- sample()、throttleLast()：定期发射 Observable 最近的数据。
- throttleFirst()：定期发射 Observable 发射的第一项数据。
- throttleWithTimeout()、debounce()：只有当 Observable 在指定的时间段后还没有发射数据时，才发射一个数据。
- timeout()：如果在一个指定的时间段后还没发射数据，就发射一个异常。
- distinct()：过滤掉重复的数据。
- distinctUntilChanged()：过滤掉连续重复的数据。
- ofType()：只发射指定类型的数据。
- igonreElements()：丢弃所有的正常数据，只发射错误或完成通知。

##### map 和 flatMap

1. map 操作符：对原始 Observable 发射的每一项数据应用一个你选择的函数，然后返回一个发射这些结果的 Observable：

   ```java
   public void helloRxJava1() {
       Observable.just("HELLO")
               .map(String::toLowerCase)
               .map(s -> s + " world")
               .subscribe(System.out::println);
   }
   
   output-->hello world
   ```

2. flatMap 操作符：将一个发射数据的 Observable 变换为多个Observables，然后将它们发射的数据合并后放进一个单独的 Observable。

   flatMap 操作符使用一个指定的函数对原始 Observable 发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的 Observable，然后 flatMap 合并这些 Observables 发射的数据，最后将合并后的结果当作它自己的数据序列发射。

   下面看一个例子。先定义一个用户对象，包含用户名和地址，由于地址可能会包括生活、工作等地方，所以使用一个 List 对象来表示用户地址：

   ```java
   public class User {
       public String name;
       public List<Address> addresses;
   
       public static class Address {
           public String street;
           public String city;
       }
   }
   ```

   如果想打印出某个用户所有的地址，那么可以借助map操作符返回一个地址的列表：

   ```java
   public void helloRxJava2() {
       User user = new User();
       user.name = "tony";
       user.addresses = new ArrayList<>();
       User.Address address1 = new User.Address();
       address1.street = "ren ming road";
       address1.city = "Su zhou";
       user.addresses.add(address1);
   
       User.Address address2 = new User.Address();
       address2.street = "dong wu bei road";
       address2.city = "Su zhou";
       user.addresses.add(address2);
   
       Observable.just(user)
               .map(user1 -> user1.addresses)
               .subscribe(addresses -> {
                   for (User.Address address : addresses) {
                       System.out.println(address.street);
                   }
               });
   }
   
   output-->ren ming road
   output-->dong wu bei road
   ```

   换成 flatMap 操作符之后，flatMap 内部将用户的地址列表转换成一个 Observable。

   ```java
   Observable.just(user)
           .flatMap(user1 -> Observable.fromIterable(user1.addresses))
           .subscribe(address -> {
               System.out.println(address.street);
           });
   ```

   执行结果与上面的相同。flatMap 对这些 Observables 发射的数据做的是合并（merge）操作，因此它们可能是交错的。还有一个操作符不会让变换后的 Observables 发射的数据交错，它严格按照顺序发射这些数据，这个操作符就是 concatMap。

##### groupBy

groupBy 操作符将一个 Observable 拆分为一些 Observables 集合，它们中的每一个都发射原始 Observable 的一个子序列。

哪个数据项由哪一个 Observable 发射是由一个函数判定的，这个函数给每一项指定一个Key，Key 相同的数据会被同一个 Observable 发射。

最终返回的是 Observable 的一个特殊子类 GroupedObservable。它是一个抽象类。getKey() 方法是 GroupedObservable 的方法，这个 Key 用于将数据分组到指定的 Observable。

```java
public void helloRxJava3() {
    Observable.range(1, 8)
            .groupBy(integer -> integer % 2 == 0 ? "偶数组" : "奇数组")
            .subscribe(stringIntegerGroupedObservable -> {
                if (stringIntegerGroupedObservable.getKey().equals("奇数组")) {
                    stringIntegerGroupedObservable.subscribe(integer ->
                      System.out.println(stringIntegerGroupedObservable.getKey() + " member：" + integer));
                }
            });
}

output-->奇数组 member：1
output-->奇数组 member：3
output-->奇数组 member：5
output-->奇数组 member：7
```

##### buffer 和 window

buffer 会定期收集 Observable 数据并放进一个数据包裹，然后发射这些数据包裹，而不是一次发射一个值。

buffer 操作符将一个 Observable 变换为另一个，原来的 Observable 正常发射数据，由变换产生的 Observable 发射这些数据的缓存集合：

```java
public void helloRxJava4() {
    Observable.range(1, 10)
            .buffer(5)
            .subscribe(System.out::println);
}

output-->[1, 2, 3, 4, 5]
output-->[6, 7, 8, 9, 10]
```

上诉代码发射了从1到10这10个数字，由于使用了 buffer 操作符，它会将原先的 Observable 转换成新的 Observable，而新的 Observable 每次可发射五个数字，发射完毕后调用 `onComplete()` 方法。

查看 buffer 操作符的源码，可以看到使用 buffer 操作符之后转换成 `Onservable<List<T>>`。

在 RxJava 中有许多 buffer 的重载方法，例如比较常用的 `buffer(count,skip)`，从原始 Observable 的第一项数据开始创建新的缓存，此后每当受到 skip 项数据，就用 count 项数据填充缓存，这些缓存可能会有重叠部分，也可能会有间隙，取决于 count 和 skip 的值：

```java
public void helloRxJava4() {
    Observable.range(1,11)
            .buffer(5,3)
            .subscribe(System.out::println);
}

output-->[1, 2, 3, 4, 5]
output-->[4, 5, 6, 7, 8]
output-->[7, 8, 9, 10, 11]
output-->[10, 11]
```

如果原来的 Observable 发射了一个 onError 通知，那么 buffer 会立即传递这个通知，而不是首先发射缓存的数据，即使在这之前缓存中包含了原始 Observable 发射的数据。

window 操作符与 buffer 类似，但它在发射前是把收集到的数据放进单独的 Observable，而不是放进一个数据结构。

window 发射的不是原始的 Observable 的数据包，而是 Observables，这些 Observables 中的每一个都发射原始 Observable 数据的一个子集，最后发射一个 onComplete 通知。

```java
public void helloRxJava5() {
    Observable.range(1, 5)
            .window(2)
            .subscribe(integerObservable -> {
                System.out.println("onNext：");
                integerObservable.subscribe(integer -> {
                    System.out.println("accept：" + integer);
                });
            });
}

output-->onNext：
output-->accept：1
output-->accept：2
output-->onNext：
output-->accept：3
output-->accept：4
output-->onNext：
output-->accept：5
```

##### first 和 last

1. first 操作符：只发射第一项（或者满足某个条件的第一项）数据。在 RxJava 2.x 中，使用 first() 会返回 Single 类型。

```java
public void helloRxJava6() {
    Observable.just(1, 2, 3)
            .first(1)
            .subscribe(integer -> System.out.println("onNext：" + integer));
}

output-->onNext：1
```

如果 Observable 不发射任何数据，那么 first 操作符的默认值就起了作用。

在 RxJava 2.x 中，还有 firstElement 操作符表示只取第一个数据，没有默认值。firstOrError 操作符表示要么能取到第一个数据，要么执行 onError 方法，他们分别返回 Maybe 类型 和 Single 类型。

2. last 操作符：如果只对 Observable 发射的最后一项数据，或者满足某个条件的最后一项数据感兴趣，那么可以使用 last 操作符。

   last 操作符跟 first 操作符类似，需要一个默认的 item，也是返回 Single 类型：

```java
public void helloRxJava7() {
    Observable.just(1, 2, 3)
            .last(3)
            .subscribe(integer -> System.out.println("onNext：" + integer));
}

output-->onNext：3
```

​	在 RxJava 中，有 lastElement 操作符 和 lastOrError 操作符。

##### take 和 takeLast

1. take 操作符

   使用 take 操作符可以只修改 Observable 的行为，返回前面的 *n* 项数据，发射完成通知，忽略剩余的数据：

   ```java
   public void helloRxJava8() {
       Observable.just(1, 2, 3, 4, 5)
               .take(3)
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：1
   output-->Next：2
   output-->Next：3
   output-->Sequence complete.
   ```

   如果对一个 Observable 使用 take(n) 操作符，而这个 Observable 发射的数据少于 n 项，那么 take 操作符生成的 Observable 不会抛出异常或者发射 onError 通知。

   take 有一个重载方法能够接受一个时长而不是数量的参数。它会丢掉发射 Observable 开始的那段时间发射的数据，时长和时间单位通过参数指定。

   ```java
   public void helloRxJava9() throws InterruptedException {
       Observable.intervalRange(0, 10, 1, 1, TimeUnit.SECONDS)
               .take(3, TimeUnit.SECONDS)
               .subscribe(aLong -> System.out.println("Next：" + aLong));
   
       Thread.sleep(10000);
   }
   
   output-->Next：0
   output-->Next：1
   output-->Next：2
   ```

   上述代码使用了 intervalRange 操作符表示每隔 1s 会发射 1 个数据，由于这里使用了 take 操作符，最后只打印了前三个数据。take 的这个重载方法默认在 computation 调度器上执行，也可以使用参数来指定其它调度器。

2. takeLast 操作符

   使用 takeLast 操作符修改原始 Observable，我们可以只发射 Observable 发射的最后 n 项数据，忽略前面数据：

   ```java
   public void helloRxJava10() {
       Observable.just(1, 2, 3, 4, 5)
               .takeLast(3)
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：3
   output-->Next：4
   output-->Next：5
   output-->Sequence complete.
   ```

   takeLast 也有一个重载方法能够接受一个时长而不是数量参数，它会发射在原始 Observable 生命周期内最后一段时间发射的数据：

   ```java
   public void helloRxJava11() throws InterruptedException {
       Observable.intervalRange(0, 10, 1, 1, TimeUnit.SECONDS)
               .takeLast(3, TimeUnit.SECONDS)
               .subscribe(aLong -> System.out.println("Next：" + aLong));
   
       Thread.sleep(10000);
   }
   
   output-->Next：7
   output-->Next：8
   output-->Next：9
   ```

   takeLast 的这个重载方法默认在 computation 调度器上执行，也可以使用参数来指定其它调度器。

##### skip 和 skipLast

1. skip 操作符：使用 skip 操作符，可以忽略 Observable 发射的前 n 项数据，只保留之后的数据：

   ```java
   public void helloRxJava12() {
       Observable.just(1, 2, 3, 4, 5)
               .skip(3)
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：4
   output-->Next：5
   output-->Sequence complete.
   ```

   skip有一个重载方法能够接受一个时长而不是数量参数，它会丢弃原始 Observable 开始那段时间发射的数据：

   ```java
   public void helloRxJava13() throws InterruptedException {
       Observable.intervalRange(0, 10, 1, 1, TimeUnit.SECONDS)
               .skip(7, TimeUnit.SECONDS)
               .subscribe(aLong -> System.out.println("Next：" + aLong));
   
       Thread.sleep(10000);
   }
   
   output-->Next：7
   output-->Next：8
   output-->Next：9
   ```

   这个重载方法默认在 computation调度器上执行。

2. skipLast 操作符与 takeLake 操作符类似。

##### elementAt 和 ignoreElements

1. elementAt 操作符：获取原始 Observable 发射的数据序列指定索引位置的数据项，然后当做自己的唯一发射数据发射。

   它传递一个基于 0 的索引值，发射原始 Observable 数据序列对应索引位置的指，如果传递给 elementAt 的值为 5，那么它会发射第 6 项数据。如果传递的是一个负数，则将会抛出一个 IndexOutOfBoundsException 异常。

   ```java
   public void helloRxJava14() {
       Observable.just(1, 2, 3, 4, 5)
               .elementAt(2)
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：3
   ```

   elementAt(index) 返回一个 Maybe 类型。

   如果原始 Observable 的数据项小于 index+1，那么会调用 onComplete()方法。所以，elementAt 还提供了一个带默认值的方法，它返回一个 Single 类型。

   跟first、last 操作符类似，element 还有一个 elementAtOrError 操作符，它表示要么能取到指定索引位置的数据，要么执行 onError 方法，也就是返回 Single 类型。

2. ignoreElements操作符：抑制原始 Observable 发射的所有数据，只允许它的终止通知（onError 或 onComplete）通过。它返回一个 Completable 类型。

   ```java
   public void helloRxJava15() {
       Observable.just(1, 2, 3, 4, 5)
               .ignoreElements()
               .subscribe(() -> System.out.println("Sequence complete."));
   }
   
   output-->Sequence complete.
   ```

##### distinct 和 filter

1. distinct 操作符：只允许还没有发射过的数据项通过

   ```java
   public void helloRxJava16() {
       Observable.just(1, 2, 1, 2, 3, 4, 5, 5, 6)
               .distinct()
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：1
   output-->Next：2
   output-->Next：3
   output-->Next：4
   output-->Next：5
   output-->Next：6
   output-->Sequence complete.
   ```

   distinct 还能接受一个 Function 参数，这个函数根据原始 Observable 发射的数据项产生一个Key，然后，比较这些 Key 而不是数据本身，来判定两个数据是否不同。

   与 distinct 类似的是 distinctUntilChanged 操作符，该操作符与 distinct 的区别是，它只判定一个数据和它的直接前驱是否不同：

   ```java
   public void helloRxJava16() {
       Observable.just(1, 2, 1, 2, 3, 4, 5, 5, 6)
               .distinctUntilChanged()
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：1
   output-->Next：2
   output-->Next：1
   output-->Next：2
   output-->Next：3
   output-->Next：4
   output-->Next：5
   output-->Next：6
   output-->Sequence complete.
   ```

2. filter 操作符

   指定一个函数测试数据项，只有通过测试的数据才会被发射：

   ```java
   public void helloRxJava17() {
       Observable.just(2, 30, 22, 5, 60, 1)
               .filter(integer -> integer > 10)
               .subscribe(integer -> System.out.println("Next：" + integer),
                       throwable -> System.err.println(throwable.getMessage()),
                       () -> System.out.println("Sequence complete."));
   }
   
   output-->Next：30
   output-->Next：22
   output-->Next：60
   output-->Sequence complete.
   ```

##### debounce

仅在过了一段指定的时间还没发射数据时才发射一个数据，debounce 操作符会过滤掉发射速率过快的数据项：

```java
public void helloRxJava18() {
    Observable.create(emitter -> {
        if (emitter.isDisposed()) return;
        try {
            for (int i = 0; i <= 10; i++) {
                emitter.onNext(i);
                Thread.sleep(i * 100);
            }
            emitter.onComplete();
        } catch (Exception exception) {
            emitter.onError(exception);
        }
    })
            //如果发射数据间隔少于500ms，就过滤拦截
            .debounce(500, TimeUnit.MILLISECONDS)
            .subscribe(integer -> System.out.println("Next：" + integer),
                    throwable -> System.err.println(throwable.getMessage()),
                    () -> System.out.println("Sequence complete."));
}

output-->Next：6
output-->Next：7
output-->Next：8
output-->Next：9
output-->Next：10
output-->Sequence complete.
```

debounce 还有另外一种形式，使用一个 Function 函数来限制发送数据。

跟 debounce 类似的是 throttleWithTimeout 操作符，它与只使用时间参数来限流的 debounce 的功能相同。