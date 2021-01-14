### RxJava 基础知识

​	RxJava通过`subscribe()`将`Observable`与`Observer`连接起来，`subscribe()`有多个重载方法：`subscribe(onNext)`、`subscribe(onNext,onError)`、`subscribe(onNext,onError,onComplete)`、`subscribe(onNext,onError,onComplete,onSubscribe)`，另一个重载方法Hello World的实现：

```java
public void helloRxJava1() {
        //common
        Observable.just("Hello World")
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(String s) throws Throwable {
                        System.out.println(s);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Throwable {
                        System.out.println(throwable.getMessage());
                    }
                }, new Action() {
                    @Override
                    public void run() throws Throwable {
                        System.out.println("onComplete()");
                    }
                });
        //lambda
        Observable.just("Hello World")
                .subscribe(System.out::println,
                        (throwable -> System.out.println(throwable.getMessage())),
                        () -> System.out.println("onComplete()"));
    }

output-->Hello World
output-->onComplete()
```

​	RxJava中`onNext`和`onError`需要参数`Consumer`，`onComplete`需要参数`Action`，`Action`与`Consumer`的区别如下：

- Action：无参数类型

- Consumer：单一参数类型

​	在RxJava2.X以上，`Observable`不再支持订阅`Subscriber`，而是需要使用`Observer`作为观察者：

```java
public void helloRxJava2() {
//common
Observable.just("Hello World")
    .subscribe(new Observer<String>() {
        @Override
        public void onSubscribe(@NonNull Disposable d) {
            System.out.println("subscribe");
        }

        @Override
        public void onNext(@NonNull String s) {
            System.out.println(s);
        }

        @Override
        public void onError(@NonNull Throwable e) {
            System.out.println(e.getMessage());
        }

        @Override
        public void onComplete() {
            System.out.println("onComplete()");
        }
    });
}

output-->subscribe
output-->Hello World
output-->onComplete()
```

​	RxJava 5种观察者模式的描述如下表：

| 类      型  |                          描      述                          |
| :---------: | :----------------------------------------------------------: |
| Observable  |          能够发射0或n个数据，并以成功或错误事件终止          |
|  Flowable   | 能够发射0或n个数据，并以成功或错误事件终止。支持背压，可以控制数据源发射的速度 |
|   Single    |                   只发射单个数据或错误事件                   |
| Completable | 从来不发射数据，只处理`onComplete`和`onError`事件。可以看成Rx的`Runnable` |
|    Maybe    | 能够发射0或者1个数据，要么成功，要么失败。有点类似于`Optional` |

##### do 操作符

​	do 操作符可以给`Observable`的生命周期的各个阶段加上一系列的回调监听，当`Observable`执行到这个阶段时，这些回调就会被处罚。在RxJava中包含了很多doXXX操作符：

```java
public void helloRxJava3() {
    //common
    Observable.just("Hello World")
            .doOnNext(s -> System.out.println("doOnNext：" + s))
            .doAfterNext(s -> System.out.println("doAfterNext：" + s))
            .doOnComplete(() -> System.out.println("doOnComplete"))
            //订阅之后回调的方法
            .doOnSubscribe(disposable -> System.out.println("doOnSubscribe："))
            .doAfterTerminate(() -> System.out.println("doAfterTerminate：注册一个动作，在Observable完成时使用"))
            .doFinally(() -> System.out.println("doFinally："))
            //Observable每发射一个数据就会触发这个回调，不仅包括onNext，还包括onError和OnComplete
            .doOnEach(stringNotification -> System.out.println("doOnEach：" + (stringNotification.isOnNext() ? "onNext" : stringNotification.isOnComplete() ? "onComplete" : "onError")))
            //订阅后可以取消订阅
            .doOnLifecycle(disposable -> System.out.println("doOnLifecycle：" + disposable.isDisposed()), () -> System.out.println("doOnLifecycle：onDispose."))
            .subscribe(s -> System.out.println("收到消息：" + s));
}

output-->doOnSubscribe：
output-->doOnLifecycle：false
output-->doOnNext：Hello World
output-->doOnEach：onNext
output-->收到消息：Hello World
output-->doAfterNext：Hello World
output-->doOnComplete
output-->doOnEach：onComplete
output-->doFinally：
output-->doAfterTerminate：注册一个动作，在Observable完成时使用
```

​	执行结果显示了RxJava的内部数据流向。最开始是`doOnSubscribe`，等到观察者消费完之后，会执行`doFinally`、`doAfterTerminate`。下边总结了一些常用的do操作符用途：

|       操作符       |                          用      途                          |
| :----------------: | :----------------------------------------------------------: |
|  `doOnSubscribe`   |           一旦观察者订阅了Observable，它就会被调用           |
|  `doOnLifeCycle`   |            可以在观察者订阅之后，设置是否取消订阅            |
|     `doOnNext`     | 它产生的`Observable`每发射一项数据就会调用它一次，它的`Consumer`接受发射的数据项。一般用于在`subscribe`之前对数据进行处理 |
|     `doOnEach`     | 它产生的`Observable`没发射一项数据就会调用它一次，不仅包括`onNext`，还包括`OnError`和`OnComplete` |
|   `doAfterNext`    |    在`onNext`之后执行，而`doOnNext()`是在`onNext`之前执行    |
|   `doOnComplete`   |   当它产生的`Observable`在正常终止调用`onComplete`时会调用   |
|    `doFinally`     | 当它产生的`Observable`终止之后会被调用，无论是正常终止还是异常终止。`doFinally`优先于`doAfterTerminate`的调用 |
| `doAfterTerminate` | 注册一个`Action`，当`Observable`调用`onComplete`或`onError`时触发 |

##### Hot Observable 和 Cold Observable

​	在RxJava中，Observable有 Hot 和 Cold 之分。

​	Hot Observable 无论有没有观察者进行订阅，事件始终都会发生。当 Hot Observable 有多个订阅者时（多个观察者进行订阅时），Hot Observable 与订阅者们的关系是一对多的关系，可以与多个订阅者共享信息。

​	Cold Observable 是只有观察者订阅了，才开始执行发射数据流的代码。并且Cold Observable 和 Observer 只能是一对一的关系。当有多个不同的订阅者时，消息是重新完整发送的。也就是说，对Cold Observable 而言，有多个Observer的时候，他们各自的事件是独立的。

​	`Observable`的`just`、`create`、`range`、`fromXXX`等操作符都能生成Cold Observable：

```java
public void helloRxJava4() {
    Consumer<Long> subscribe1 = aLong -> {
        System.out.println("subscribe1：" + aLong);
    };
    Consumer<Long> subscribe2 = aLong -> {
        System.out.println("    subscribe2：" + aLong);
    };
    @NonNull Observable<Long> observable = Observable.create(emitter -> {
        Observable.interval(10, TimeUnit.MICROSECONDS, Schedulers.computation())
                .take(Integer.MAX_VALUE)
                .subscribe(emitter::onNext);
    });

    observable.subscribe(subscribe1);
    observable.subscribe(subscribe2);

    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

output-->subscribe1：0
output-->    subscribe2：0
output-->subscribe1：1
output-->    subscribe2：1
output-->subscribe1：2
output-->    subscribe2：2
output-->subscribe1：3
output-->    subscribe2：3
output-->subscribe1：4
output-->    subscribe2：4
output-->subscribe1：5
output-->subscribe1：6
```

​	可以看出，`subscribe1`和`subscribe2`的结果并不一定是相同的，他们二者是完全独立的。`create`操作符创建的`Observable`是Cold Observable。

​	当UI交互事件、网络环境变化、地理位置变化、服务器推送消息的到达等，需要使用Hot Observable，因此需要使用`publish`，将`Observable`转化成`ConnectableObservable`：

```java
public void helloRxJava5() {
    Consumer<Object> subscribe1 = aLong -> {
        System.out.println("subscribe1：" + aLong);
    };
    Consumer<Object> subscribe2 = aLong -> {
        System.out.println("    subscribe2：" + aLong);
    };
    @NonNull ConnectableObservable<Object> observable = Observable.create(emitter -> {
        Observable.interval(10, TimeUnit.MICROSECONDS, Schedulers.computation())
                .take(Integer.MAX_VALUE)
                .subscribe(emitter::onNext);
    }).publish();
    //注意需要调用连接方法
    observable.connect();

    observable.subscribe(subscribe1);
    observable.subscribe(subscribe2);

    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

output-->subscribe1：0
output-->    subscribe2：0
output-->subscribe1：1
output-->    subscribe2：1
output-->subscribe1：2
output-->    subscribe2：2
output-->subscribe1：3
output-->    subscribe2：3
output-->subscribe1：4
output-->    subscribe2：4
output-->subscribe1：5
output-->    subscribe2：5
```

​	可以看到，多个订阅的`subscribe`共享同一时间。在这里，`ConnectableObservable`是线程安全的。

​	另一种将Cold Observable转化成Hot Observable的方法是使用`Subject/Processor`，`Subject`和`Processor`的zuo'y相同，`Processor`是RxJava 2.x 新增的类，继承自Flowable，支持背压控制（Back Presure）,而`Subject`则不支持背压控制：

```java
public void helloRxJava6() {
    Consumer<Long> subscriber1 = aLong -> {
        System.out.println("subscriber1：" + aLong);
    };
    Consumer<Long> subscriber2 = aLong -> {
        System.out.println("    subscriber2：" + aLong);
    };
    Consumer<Long> subscriber3 = aLong -> {
        System.out.println("        subscriber3：" + aLong);
    };
    Observable<Long> observable = Observable.create(emitter -> {
        Observable.interval(10,TimeUnit.MICROSECONDS)
                .take(Integer.MAX_VALUE)
                .subscribe(emitter::onNext);
    });
    PublishSubject<Long> subject = PublishSubject.create();
    observable.subscribe(subject);
    subject.subscribe(subscriber1);
    subject.subscribe(subscriber2);
    try {
        Thread.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    subject.subscribe(subscriber3);
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

​	执行结果与使用`publish`操作符相同。`Subject`既是Observable，又是Observer。`Subject`作为管擦在，可以订阅目标Cold Observable，使对方开始发送时间。同时它又作为Observable转发或发送新的实践，让Cold Observable借助`Subject`转换为Hot Observable。

​	`Subject`并不是线程安全的，如果想要其线程安全，则需要调用`toSerualized()`方法。

​	某些情况下，需要将Hot Observable转换成Cold Observable，一种方式是使用`ConnectableObservable`的`refCount`操作符。

​	`refCount`操作符把从一个可连接的Observable连接和断开的过程自动化了。它操作一个可连接的Observable，返回一个普通的Observable。当第一个订阅者/观察者订阅这个Observable时，RefCount连接到下层的可连接Observable。RefCount跟踪有多少个观察者订阅它，直到最后一个观察者完成，才断开与下层可连接Observable的连接。

​	如果所有的订阅者/观察者都取消订阅了，则数据流停止；如果重新订阅，则重新开始数据流：

```java
public void helloRxJava7() {
    Consumer<Object> subscriber1 = aLong -> {
        System.out.println("subscriber1：" + aLong);
    };
    Consumer<Object> subscriber2 = aLong -> {
        System.out.println("    subscriber2：" + aLong);
    };
    ConnectableObservable<Object> connectableObservable = Observable.create(emitter -> {
        Observable.interval(10,TimeUnit.MICROSECONDS)
                .take(Integer.MAX_VALUE)
                .subscribe(emitter::onNext);
    }).publish();
    connectableObservable.connect();

    @NonNull Observable<Object> observable = connectableObservable.refCount();

    Disposable disposable1 = observable.subscribe(subscriber1);
    Disposable disposable2 = observable.subscribe(subscriber2);

    try {
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    disposable1.dispose();
    disposable2.dispose();

    System.out.println("重新开始数据流");
    observable.subscribe(subscriber1);
    observable.subscribe(subscriber2);

    try {
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

output-->subscriber1：0
output-->	subscriber2：0
output-->subscriber1：1
output-->	subscriber2：1
output-->重新开始数据流
output-->subscriber1：0
output-->	subscriber2：0
output-->subscriber1：1
output-->	subscriber2：1
```

​	如果不是所有的订阅者/观察者都取消了订阅，而只是部分取消，则部分的订阅者/观察者重新开始订阅时，不会从头开始数据流：

```java
public void helloRxJava8() {
    Consumer<Object> subscriber1 = aLong -> {
        System.out.println("subscriber1：" + aLong);
    };
    Consumer<Object> subscriber2 = aLong -> {
        System.out.println("    subscriber2：" + aLong);
    };
    Consumer<Object> subscriber3 = aLong -> {
        System.out.println("        subscriber3：" + aLong);
    };
    ConnectableObservable<Object> connectableObservable = Observable.create(emitter -> {
        Observable.interval(10,TimeUnit.MICROSECONDS)
                .take(Integer.MAX_VALUE)
                .subscribe(emitter::onNext);
    }).publish();
    connectableObservable.connect();

    @NonNull Observable<Object> observable = connectableObservable.refCount();

    Disposable disposable1 = observable.subscribe(subscriber1);
    Disposable disposable2 = observable.subscribe(subscriber2);
    observable.subscribe(subscriber3);

    try {
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    disposable1.dispose();
    disposable2.dispose();

    System.out.println("subscriber1、subscriber2 重新订阅");
    observable.subscribe(subscriber1);
    observable.subscribe(subscriber2);

    try {
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}

output-->subscriber1：0
output-->	subscriber2：0
output-->		subscriber3：0
output-->subscriber1：1
output-->	subscriber2：1
output-->		subscriber3：1
output-->subscriber1、subscriber2 重新订阅
output-->		subscriber3：2
output-->subscriber1：2
output-->	subscriber2：2
```

​	再这里，`subscriber1`和`subscriber2`先取消了订阅，`subscriber3`并没有取消订阅。之后，`subscriber1`和`subscriber2`又重新订阅。最终`subscriber1`、`subscriber2`、`subscriber3`的值保持一致。

​	第二种方式是使用Observable的`share`操作符，其封装了`publish().refCount()`的调用：

```java
public void helloRxJava9() {
    Consumer<Object> subscriber1 = aLong -> {
        System.out.println("subscriber1：" + aLong);
    };
    Consumer<Object> subscriber2 = aLong -> {
        System.out.println("    subscriber2：" + aLong);
    };
    Consumer<Object> subscriber3 = aLong -> {
        System.out.println("        subscriber3：" + aLong);
    };
    @NonNull Observable<Object> observable = Observable.create(emitter -> {
        Observable.interval(10,TimeUnit.MICROSECONDS)
                .take(Integer.MAX_VALUE)
                .subscribe(emitter::onNext);
    }).share();

    Disposable disposable1 = observable.subscribe(subscriber1);
    Disposable disposable2 = observable.subscribe(subscriber2);
    observable.subscribe(subscriber3);

    try {
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    disposable1.dispose();
    disposable2.dispose();

    System.out.println("subscriber1、subscriber2 重新订阅");
    observable.subscribe(subscriber1);
    observable.subscribe(subscriber2);

    try {
        Thread.sleep(1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

##### Flowable

​	在RxJava 2.x 中，Observable 不再支持背压（Back Pressure），而改由Flowable来支持非阻塞式的背压。Flowable是 RxJava 2.x 新增的被观察者，Flowable 可以看成 Observable 新的实现，它支持背压，同时实现 Reactive Streams 的 Pbulisher 接口。Flowable与Observable的使用场景如下：

​		使用Observable较好的场景如下：

- 一般处理最大不超过1000条数据，并且几乎不会出现内存溢出；

- GUI 鼠标事件，基本不会背压；

- 处理同步流。

  使用Flowable较好的场景如下：

- 处理以某种方式产生超过10KB的元素；

- 文件读取与分析；

- 读取数据库记录，也是一个阻塞的和基于拉取模式

- 网络I/O流；

- 创建一个响应式非阻塞接口。

##### Single、Completable和Maybe

​	除 Observable 和 Flowable 外，在 RxJava 2.x 中还有三种类型的被观察者：Single、Completable和 Maybe。

​	从SingleEmitter的源码可以看出，Single 只有`onSuccess`和`onError`事件。`onSuccess`用于发射数据，且只能发射一个数据，后边即使再发射数据也不会做任何处理。Single 的 SingleObserver 中只有`onSuccess`和`onError`，这也是 Single与其他4种被观察者之间最大的区别：

```java
public void helloRxJava10() {
    Single.create((SingleOnSubscribe<String>) emitter -> {
        emitter.onSuccess("test");
    }).subscribe(System.out::println, Throwable::printStackTrace);
}

output-->test
```

​	Single 可以通过toXXX方法转换成Observable、Flowable、Completable及Maybe。

​	Completable在创建后，不会发射任何数据。Completable 只有`onComplete`和`onError`事件，同时 Completable 并没有`map`、`flatMap`等操作符，它的操作符比起 Observable/Flowable 要少得多。

​	我们可以通过fromXXX操作符来创建一个Completable。下面是一个Completable版本的Hello World：

```java
public void helloRxJava11() {
    Completable.fromAction(() -> System.out.println("Hello World")).subscribe();
}
```

​	Completable 经常结合 `andThen` 操作符使用：

```java
public void helloRxJava11() {
    Completable.create(emitter -> {
        try {
            TimeUnit.SECONDS.sleep(1);
            emitter.onComplete();
        } catch (InterruptedException e) {
            emitter.onError(e);
        }
    }).andThen(Observable.range(1, 10))
            .subscribe(System.out::println);
}

output-->1
output-->2
...
```

​	在这里，`emitter.onComplete()`执行完成之后，表明Completable已经执行完毕，接下来执行`andThen`里的操作。

​	在Completable中，andThen 有多个重载方法，正好对应了5种被观察者的类型：

| Completable    andThen (CompletableSource next)         |
| ------------------------------------------------------- |
| <T> Maybe<T>    andThen (MaybeSource<T> next)           |
| <T> Observable<T>    andThen (ObservableSource<T> next) |
| <T> Flowable<T>    andThen (Publisher<T> next)          |
| <T>Single<T>    andThen (SingleSource<T> next)          |

​	Completable 也可以通过 toXXX 方法转换成`Observable`、`Flowable`、`Single`以及`Maybe`。

​	在网络操作中，如果遇到更新的情况，也就是 Restful 架构中的 PUT 操作，一般要么返回原先的对象，要么只提示更新成功。下面两个接口使用了 Retrofit 框架，分别用于短信验证码和更新用户信息：

```java
/**
* 获取短信验证码
*/
@POST("v1/user-auth")
Completable getVerificationCode(@Body VerificationParam param);

/**
* 用户信息更新接口
*/
@POST("v1/user-auth")
Completable update(@Body UpdateParam param);
```

​	在 model 类中大致会这样写：

```java
/**
* 获取验证码
*/
public Completable getVerificationCode(AppCompatActivity activity,VerificationParam param){
    return apiServiice.getVerificationCode(param)
        .compose(RxJavaUtils.<VerificationCodeModel>completableToMain())
        .compose(RxLifecycle.bind(activity).<VerificationCodeModel>toLifecycleTransformer());
}
```

​	需特别注意的是，`getVerificationCode`返回的是 Completable 而不是 Completable<T>。获取验证码成功则给出相应的 toast 提示，失败则做出对应处理：

```java
VerificationCodeModel model = new VerificationCodeModel();
model.getVerificationCode(RegisterActivity.this,param)
    .subscribe(s -> showShort(RegisterActivity.this,"发送验证码成功"),throwable -> throwable.printStackTrace());
```

​	Maybe 是 RxJava 2.x 之后才有的新类型，可以看成是`Single`和`Completable`的结合。Maybe 创建之后，`MaybeEmitter`和`SingleEmitter`一样，并没有`onNext()`方法，同样需要通过`onSuccess()`方法来发射数据：

```java
public void helloRxJava12() {
    Maybe.create((MaybeOnSubscribe<String>) emitter -> emitter.onSuccess("test"))
            .subscribe(System.out::println);
}
```

​	Maybe 也只能发射0或者1个数据，即使发射多个数据，后面发射的数据也不会处理。

​	Maybe 在没有数据发射时，subscribe 会调用 MaybeObserver 的 onComplete()。如果 Maybe 有数据发射或者调用了 onError() ，则不会执行 MaybeObserver 的 onComplete()。

​	Maybe也可以通过 toXXX 转换成`Observable`、`Flowable`、`Single`。

##### Subject 和 Processor

​	Subject 既是 Observable，又是 Observer。官网称可以将 Subject看作一个桥梁或者代理。Subject 包含4种类型，分别是 AsyncSubject、BehaviorSubject、ReplaySubject和PublishSubject。

​	Observer 会接收 AsyncSubject 的 onComplete() 之前的最后一个数据。

```java
public void helloRxJava13() {
    AsyncSubject<String> subject = AsyncSubject.create();
    subject.onNext("asyncSubject1");
    subject.onNext("asyncSubject2");
    subject.onComplete();
    subject.subscribe(s -> {
        System.out.println("asyncSubject：" + s);
    }, throwable -> {
        //不输出，异常才会输出
        System.out.println("asyncSubject onError");
    }, () -> {
        //输出 asyncSubject onComplete
        System.out.println("asyncSubject：onComplete");
    });
    subject.onNext("asyncSubject3");
    subject.onNext("asyncSubject4");
}

output-->asyncSubject：asyncSubject2
output-->asyncSubject：onComplete
```

​	改一下代码，将subject.onComplete()放在最后：

```java
public void helloRxJava13() {
    AsyncSubject<String> subject = AsyncSubject.create();
    subject.onNext("asyncSubject1");
    subject.onNext("asyncSubject2");
    subject.subscribe(s -> {
        System.out.println("asyncSubject：" + s);
    }, throwable -> {
        //不输出，异常才会输出
        System.out.println("asyncSubject onError");
    }, () -> {
        //输出 asyncSubject onComplete
        System.out.println("asyncSubject：onComplete");
    });
    subject.onNext("asyncSubject3");
    subject.onNext("asyncSubject4");
    subject.onComplete();
}

output-->asyncSubject：asyncSubject4
output-->asyncSubject：onComplete
```

​	注意，subject.onComplete()必须要调用才会开始发送数据，否则观察者将不接收任何数据。

​	Observer 会先接收到 BehaviorSubject 被订阅之前的最后一个数据，再接收订阅之后发射过来的数据。如果 BehaviorSubject 被订阅之前没有发送任何数据，则会发送一个默认数据：

```java
public void helloRxJava14() {
    BehaviorSubject<String> subject = BehaviorSubject.createDefault("behaviorSubject1");
    //订阅前发射了behaviorSubject2，behaviorSubject1将被丢弃
    subject.onNext("behaviorSubject2");
    subject.subscribe(s -> {
        System.out.println("behaviorSubject：" + s);
    },throwable -> {
        //不输出，异常才会输出
        System.out.println("behaviorSubject onError");
    },() -> {
        //输出 behaviorSubject onComplete
        System.out.println("behaviorSubject：onComplete");
    });
    subject.onNext("behaviorSubject3");
    subject.onNext("behaviorSubject4");
}

output-->behaviorSubject：behaviorSubject2
output-->behaviorSubject：behaviorSubject3
output-->behaviorSubject：behaviorSubject4
```

​	ReplaySubjec 会发射所有来自原始 Observable 的数据给观察者，无论它们是何时订阅的。

```java
public void helloRxJava15() {
    ReplaySubject<String> subject = ReplaySubject.create();
    subject.onNext("replaySubject1");
    subject.onNext("replaySubject2");
    subject.subscribe(s -> {
        System.out.println("replaySubject：" + s);
    }, throwable -> {
        //不输出，异常才会输出
        System.out.println("replaySubject onError");
    }, () -> {
        //输出 replaySubject onComplete
        System.out.println("replaySubject：onComplete");
    });
    subject.onNext("replaySubject3");
    subject.onNext("replaySubject4");
}

output-->replaySubject：replaySubject1
output-->replaySubject：replaySubject2
output-->replaySubject：replaySubject3
output-->replaySubject：replaySubject4
```

​	修改下代码，将`create()`改成`createWithSize(1)`，表示只缓存订阅前最后发送的一条数据。

```java
public void helloRxJava15() {
    ReplaySubject<String> subject = ReplaySubject.createWithSize(1);
    subject.onNext("replaySubject1");
    subject.onNext("replaySubject2");
    subject.subscribe(s -> {
        System.out.println("replaySubject：" + s);
    }, throwable -> {
        //不输出，异常才会输出
        System.out.println("replaySubject onError");
    }, () -> {
        //输出 replaySubject onComplete
        System.out.println("replaySubject：onComplete");
    });
    subject.onNext("replaySubject3");
    subject.onNext("replaySubject4");
}

output-->replaySubject：replaySubject2
output-->replaySubject：replaySubject3
output-->replaySubject：replaySubject4
```

​	这个执行结果与 BehaviorSubject 的相同。但是从并发的角度来看， ReplaySubject 在处理并发 `subscribe()`和`onNext()`时会更复杂。

​	ReplaySubject 除了可以限制缓存数据的数量，还能限制缓存的事件，使用 `createWithTime()`即可。

​	Observer 只接收 PublishSubject 被订阅之后发送的数据。

```java
public void helloRxJava16() {
    PublishSubject<String> subject = PublishSubject.create();
    subject.onNext("publishSubject1");
    subject.onNext("publishSubject2");
    subject.onComplete();
    subject.subscribe(s -> {
        System.out.println("publishSubject：" + s);
    }, throwable -> {
        //不输出，异常才会输出
        System.out.println("publishSubject onError");
    }, () -> {
        //输出 publishSubject onComplete
        System.out.println("publishSubject：onComplete");
    });
    subject.onNext("publishSubject3");
    subject.onNext("publishSubject4");
}

output-->publishSubject：onComplete
```

​	因为 subject 在订阅之前已经执行了`onComplete()`方法，所以无法发射数据。稍微修改一下代码，将`onComplete()`方法放在最后。

```java
public void helloRxJava16() {
    PublishSubject<String> subject = PublishSubject.create();
    subject.onNext("publishSubject1");
    subject.onNext("publishSubject2");
    subject.subscribe(s -> {
        System.out.println("publishSubject：" + s);
    }, throwable -> {
        //不输出，异常才会输出
        System.out.println("publishSubject onError");
    }, () -> {
        //输出 publishSubject onComplete
        System.out.println("publishSubject：onComplete");
    });
    subject.onNext("publishSubject3");
    subject.onNext("publishSubject4");
    subject.onComplete();
}

output-->publishSubject：publishSubject3
output-->publishSubject：publishSubject4
output-->publishSubject：onComplete
```

​	最后，用下表总结4个 Subject 的 特性：

|     Subject     |                发 射 行 为                 |
| :-------------: | :----------------------------------------: |
|  AsyncSubject   |  不论订阅什么时候发生，只发射最后一个数据  |
| BehaviorSubject | 发送订阅之前的一个数据和订阅之后的全部数据 |
|  ReplaySubject  |    不论订阅什么时候发生，都发射全部数据    |
| PublishSubject  |           发送订阅之后的全部数据           |

​	下面的代码实现了一个简化版本的 Android EventBus，在这里使用了 PublishSubject。因为事件总线是基于`发布、订阅模式`实现的，一个事件在 Activity/Fragment 中被订阅，则在App的任意地方一旦发布该事件，则订阅的地方均能收到这一事件，这很符合 Hot Observable 的特性，所以使用`PublishSubject`，考虑到多线程的情况，还需要使用 Subject 的`toSerialized()` 方法。

```kotlin
object RxBus {
    private val mBus = PublishSubject.create<Any>().toSerialized()

    fun post(any: Any) {
        mBus.onNext(any)
    }

    fun <T> toObservable(tClass: Class<T>): Observable<T> {
        return mBus.ofType(tClass)
    }

    fun toObservable(): Observable<Any> = mBus

    fun hasObservers():Boolean = mBus.hasObservers()
}
```

​	这个版本的EventBus比较简单，没有考虑背压的情况。如果要增加背压的处理，可以使用`Processor`，在后续，还会详细地开发一个RxBus。

​	使用`BehaviorSubject`实现预加载。预加载可以很好地提高程序的用户体验。每当用户处于弱网络时，如果能够预先加载一些数据，比如保存上一次打开App的数据，则用户体验会有很好的提升，下面是借助`BehaviorSubjct`的特性来实现一个简单的预加载类`RxPreLoader`，可以考虑在App基类的 Activity/Fragment 中也实现一个类似的 `RxPreLoader`：

```kotlin
class RxPreLoader<T>(defaultValue: T) {
    //缓存订阅之前的最新数据
    private val mData: BehaviorSubject<T> = BehaviorSubject.createDefault(defaultValue)
    private var disposable: Disposable? = null

    /**
     * 发送事件
     */
    fun publish(value: T) {
        mData.onNext(value)
    }

    /**
     * 订阅事件
     */
    fun subscribe(onNext: Consumer<T>): Disposable {
        disposable = mData.subscribe(onNext)
        return disposable!!
    }

    /**
     * 取消订阅
     */
    fun dispose() {
        if (disposable != null && !disposable!!.isDisposed) {
            disposable!!.dispose()
            disposable = null
        }
    }

    /**
     * 获取缓存数据的Subject
     */
    fun getCacheDataSubject(): BehaviorSubject<T> {
        return mData
    }

    /**
     * 直接获取最后一个数据
     */
    fun getLastCacheData(): T {
        return mData.value
    }
}
```

​	Processor 和 Subject 的作用相同。Processor 是 RxJava 2.0 新增的功能，它是一个接口，继承自 Subscribe、Publisher，能够支持背压（Back Pressure）控制。

