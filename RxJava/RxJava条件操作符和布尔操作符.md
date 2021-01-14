### RxJava 条件操作符和布尔操作符

RxJava 的条件操作符主要包括以下几个：

- amb()：给定多个 Observable，只让第一个发射数据的 Observable 发射全部数据。
- defaultIfEmpty()：发射来自原始 Observable 的数据，如果原始 Observable 没有发射数据，则发射一个默认数据。
- skipUntil()：丢弃原始 Observable 发射的数据，直到第二个 Observable 发射了一个数据，然后发射原始 Observable 的剩余数据。
- skipWhile()：丢弃原始 Observable 发射的数据，直到一个特定的条件为假，然后发射原始 Observable 的剩余数据。
- takeUntil()：发射来自原始 Observable 的数据，直到第二个 Observable 发射了一个数据或一个通知。
- takeWhile() and takeWhileWithIndex()：发射原始 Observable 的数据，直到一个特定的条件为真，然后跳过剩余数据。

RxJava 的布尔操作符主要包括：

- all()：判断是否所有的数据项都满足条件。
- contains()：判断 Observable 是否会发射一个指定的值。
- exists() and isEmpty()：判断 Observable 是否发射了一个值。
- sequenceEqual()：判断两个 Observables 发射的序列是否相等。

##### all、contains 和 amb

1. all 操作符：传递一个函数给 all 操作符，这个函数接受原始 Observable 发射的数据，根据计算返回一个布尔值。all 返回一个只发射单个布尔值的 Observable，如果原始 Observable 正常终止并且每一项都满足条件，就返回 true；如果原始 Observable 的任意一项数据不满足条件，就返回 false。

   ```java
   public void helloRxJava19() {
       Observable.just(1, 2, 3, 4, 5)
               .all(integer -> integer < 10)
               .subscribe(System.out::println);
   }
   
   output-->true
   ```

   all 操作符默认不在任何特定的调度器上执行。

2. contains 操作符：给 contains 传一个指定的值，如果原始 Observable 发射了那个值，那么返回的 Observable 将发射 true，否则发射 false。与它相关的一个操作符是 isEmpty()，用于判定原始 Observable 是否未发射任何数据。

   ```java
   public void helloRxJava20() {
       Observable.just(2, 30, 22, 5, 60, 1)
               .contains(22)
               .subscribe(aBoolean -> {
                   System.out.println("contains(22)：" + aBoolean);
               });
   
       Observable.just(1, 2, 3, 4, 5)
               .isEmpty()
               .subscribe(aBoolean -> {
                   System.out.println("isEmpty()：" + aBoolean);
               });
   }
   
   output-->contains(22)：true
   output-->isEmpty()：false
   ```

   contains 默认不在任何特定的调度器上执行。

3. amb 操作符：当传递多个 Observable 给 amb 时，它只发射其中一个 Observable 的数据和通知。首先发送通知给 amb 的那个 Observable，不管发射的是一项数据，还是一个 onError 或是 onCompleted 通知。amb 将忽略和丢弃其他所有 Observables 的发射物。

   在RxJava 2.x 中，amb 需要传递一个 Iterable 对象，或者使用 ambArray 来传递可变参数。

   ```java
   public void helloRxJava3() {
       Observable.ambArray(
               Observable.just(1, 2, 3).delay(1, TimeUnit.SECONDS),
               Observable.just(4, 5, 6)
       ).subscribe(System.out::println);
   }
   
   output-->4
   output-->5
   output-->6
   ```

   由于第一个 Observable 延迟发生，因此消费了第二个 Observable，第一个 Observable 发射的数据就不再处理了。

##### defaultIfEmpty

如果原始 Observable 没有发射任何数据，就正常终止（以 onComplete 的形式）了，那么 defaultIfEmpty 返回的 Observable 就发射一个我们提供的默认值。

```java
public void helloRxJava4() {
    Observable.empty()
            .defaultIfEmpty(8)
            .subscribe(System.out::println);
}

output-->8
```

在 defaultIfEmpty 内部，其实调用的是 switchIfEmpty 操作符。defaultIfEmpty  和 switchIfEmpty 的区别是，defaultIfEmpty 操作符只能在被观察者不发送数据时发送一个默认的数据，如果想要发送更多的数据，则可以使用 switchIfEmpty 操作符，发送自定义的被观察者作为替代。

```java
public void helloRxJava5() {
    Observable.empty()
            .switchIfEmpty(Observable.just(1, 2, 3))
            .subscribe(System.out::println);
}

output-->1
output-->2
output-->3
```

##### sequenceEqual

判断两个 Observable 是否发射相同的数据序列。传递两个 Observable 给 sequenceEqual 操作符时，它会比较两个 Observable 的发射物，如果两个序列相同（相同的数据，相同的顺序，相同的终止状态），则发射 true，否则发射 false。

```java
public void helloRxJava6() {
    Observable.sequenceEqual(Observable.just(1, 2, 3, 4, 5),
            Observable.just(1, 2, 3, 4, 5))
            .subscribe(System.out::println);
    //output--> true
    Observable.sequenceEqual(Observable.just(1, 2, 3, 4, 5),
            Observable.just(1, 2, 3, 4, 5,6))
            .subscribe(System.out::println);
    //output--> false
}
```

sequenceEqual 还有一个版本接受第三个参数，可以传递一个函数用于比较两个数据项是否相同。对于复杂对象的比较，用三个参数的版本比较合适。

```java
public void helloRxJava6() {
    Observable.sequenceEqual(Observable.just(1, 2, 3),
            Observable.just(2, 3, 4), (integer, integer2) -> integer2 == integer + 1)
            .subscribe(System.out::println);
}

output-->true
```

##### skipUntil 和 skipWhile

skipUntil 订阅原始的 Observable，但是忽略它的发射物，直到第二个 Observable 发射一项数据那一刻，它才开始发射原始的Observable。

```java
public void helloRxJava7() throws InterruptedException {
    Observable.intervalRange(1, 9, 0, 1, TimeUnit.MILLISECONDS)
            .skipUntil(Observable.timer(8, TimeUnit.MILLISECONDS))
            .subscribe(System.out::println);

    Thread.sleep(1000);
}

output-->7
output-->8
output-->9
```

skipWhile 订阅原始的 Observable，但是忽略它的发射物，直到指定的某个条件变为 false，它才开始发射原始的 Observable。

```java
public void helloRxJava8() throws InterruptedException {
    Observable.just(1, 2, 3, 4, 5)
            .skipWhile(integer -> integer <= 3)
            .subscribe(System.out::println);
}

output-->4
output-->5
```

##### takeUntil 和 takeWhile

takeUntil 当第二个 Observable 发射了一项数据或者终止时，丢弃原始 Observable 发射的任何数据并终止。

```java
public void helloRxJava9() throws InterruptedException {
    Observable.just(1, 2, 3, 4, 5)
            .takeUntil(integer -> integer == 2)
            .subscribe(System.out::println);
}

output-->1
output-->2
```

takeWhile 发射原始的 Observable，直到某个指定的条件不成立，它会立即停止发射原始 Observable，并终止自己的Observable。

```java
public void helloRxJava9() throws InterruptedException {
    Observable.just(1, 2, 3, 4, 5)
            .takeWhile(integer -> integer <= 2)
            .subscribe(System.out::println);
}

output-->1
output-->2
```

