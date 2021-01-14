### RxJava Disposable 和 Transformer 的使用

在 RxJava 2.0 之后，直接返回 Disposable 用以取消订阅并通知 Subscriber，并允许垃圾回收机制释放对象，防止照成内存泄露。

```java
public interface Disposable {
    /**
     * Dispose the resource, the operation should be idempotent.
     */
    void dispose();
    
    /**
     * Returns true if this resource has been disposed.
     * @return true if this resource has been disposed
     */
    boolean isDisposed();
}
```

示例代码：

```java
Disposable disposable = Observable.just("Hello,World!")
    .subscribe(System.out::println);
disposable.dispose();
```

##### RxLifecycle 和 AutoDispose

在 Android 开发中，可以使用 Disposable 来管理一个订阅或者使用 CompositeDisposable 来管理多个订阅，防止由于没有及时取消，导致 Activity/Fragment 无法销毁而引起内存泄露。然而，也有一些比较成熟的库可以做这些事情。

1. ###### RxLifeCycle

   GitHub 下载地址：https://github.com/trello/RxLifecycle

   RxLifecycle 是配合 Activity/Fragment 生命周期来管理订阅的。由于 RxJava Observable 订阅后（调用了 subscribe 函数）,一般会在后台线程执行一些操作，当后台操作返回后，调用 Observer 的 onNext 等函数，更新 UI 状态。如果用户退出了当前界面，而之前的 Observable 又未取消订阅，就会导致之前的 Activity 无法被 JVM 回收，从而导致内存泄漏。这就是在 Android 开发中值得注意的地方，RxLifecycle 就是专门用来做这件事的。

   默认情况下，RxLifecycle 将在辅助生命周期事件中终止 Observable，所以如果在 Activity 的 OnCreate() 方法期间订阅了 Observable，则 RxLifecycle 将在该 Activity 的 onDestory() 方法中终止 Observable 序列。如果在 Fragment 的 onAttach() 方法中订阅，那么 RxLifecycle 将在 onDetach() 方法中终止该序列。

   使用 RxLifecycleAndroid.bindActivity 进行设置：

   ```java
   Observable<Integer> myObservable = Observable.range(0.25);
   ...
   
   @Override
   public void onResume(){
       super.onResume();
       myObservable
           .compose(RxLifecycleAndroid.bindActivity(lifecycle))
           .subscribe();
   }
   ```

   或者可以指定 lifecycle 事件，在该事件中，RxLifecycle 应该使用 RxLifecycle.bindUntilEvent 终止 Observable 序列：

   ```java
   @Override
   public void onResume(){
       super.onResume();
       myObservable
           .compose(RxLifecycle.bindUntilEvent(lifecycle,ActivityEvent.DESTORY))
           .subscribe();
   }
   ```

2. ###### AutoDispose

   GitHub 下载地址：https://github.com/uber/AutoDispose

   AutoDispose 是 Uber 开源的库。它与 RxLifecycle 的区别是，不仅可以在 Android 平台上使用，还可以在 Java（企业级）平台上使用，适用的范围更加宽广。

   AutoDispose 支持 Kotlin、Android Architecture Components，并且可以与 RxLifecycle 进行相互操作。

##### Transformer 在 RxJava 中的使用

1. ###### Transformer 的用途

   Transformer 是转换器的意思。在 RxJava 2.x 版本中有 ObservableTransformer、SingleTransformer、CompletableTransformer、FlowableTransformer 和 MaybeTransformer。

   Transformer 能够将一个 Observable/Flowable/Single/Completable/Maybe 对象转换成另一个 Observable/Flowable/Single/Completable/Maybe 对象，与调用一系列的内联操作符一模一样。

   举个简单的例子，写一个 transformer() 方法将一个发射整数的 Observable 转换为发射字符串的 Observable。

   ```java
   private ObservableTransformer<Integer, String> transformer() {
       return upstream -> upstream.map(String::valueOf);
   }
   ```

   接下来是使用 transformer()方法：

   ```java
   Observable.just(123, 456)
           .compose(transformer())
           .subscribe(System.out::println);
   ```

2. ###### 与 compose 操作符结合使用

   compose 操作符能够从数据流中得到原始的被观察者。当创建被观察者时，compose 操作符会立即执行，而不像其他的操作符需要在 onNext() 调用后才能执行。

   常用场景：

   1. 切换到主线程

      对于网络请求，我们通常会做如下操作来切换线程：

      ```java
      .subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      ```

      做一个简单的封装：

      ```kotlin
      import io.reactivex.rxjava3.android.schedulers.AndroidSchedulers
      import io.reactivex.rxjava3.core.FlowableTransformer
      import io.reactivex.rxjava3.core.ObservableTransformer
      import io.reactivex.rxjava3.schedulers.Schedulers
      
      object RxJavaUtils {
          @JvmStatic
          fun <T> observableToMain(): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.subscribeOn(Schedulers.io())
                      .observeOn(AndroidSchedulers.mainThread())
              }
      
          @JvmStatic
          fun <T> flowableToMain(): FlowableTransformer<T, T> =
              FlowableTransformer { upstream ->
                  upstream.subscribeOn(Schedulers.io())
                      .observeOn(AndroidSchedulers.mainThread())
              }
      }
      ```

      对于 Flowable 切换到主线程的操作，可以这样使用

      ```java
      .compose(RxJavaUtils.flowableToMain())
      ```

   2. RxLifecycle 中的LifecycleTransformer

      RxLifecycle 能够配合 Android 的生命周期，防止 App 内存泄漏，其中就使用了 LifecycleTransformer。知乎也做了一个类似的 RxLifecycle，能够做同样的事情。

      下面使用了知乎的 RxLifecycle，并且对 LifecycleTransformer 稍微做了一些修改，将五个Transformer 合并成了一个。

      ```java
      import org.reactivestreams.Publisher;
      
      import io.reactivex.rxjava3.annotations.NonNull;
      import io.reactivex.rxjava3.core.Completable;
      import io.reactivex.rxjava3.core.CompletableSource;
      import io.reactivex.rxjava3.core.CompletableTransformer;
      import io.reactivex.rxjava3.core.Flowable;
      import io.reactivex.rxjava3.core.FlowableTransformer;
      import io.reactivex.rxjava3.core.Maybe;
      import io.reactivex.rxjava3.core.MaybeSource;
      import io.reactivex.rxjava3.core.MaybeTransformer;
      import io.reactivex.rxjava3.core.Observable;
      import io.reactivex.rxjava3.core.ObservableSource;
      import io.reactivex.rxjava3.core.ObservableTransformer;
      import io.reactivex.rxjava3.core.Single;
      import io.reactivex.rxjava3.core.SingleSource;
      import io.reactivex.rxjava3.core.SingleTransformer;
      import io.reactivex.rxjava3.processors.BehaviorProcessor;
      
      public class LifecycleTransformer<T> implements ObservableTransformer<T, T>,
              FlowableTransformer<T, T>,
              SingleTransformer<T, T>,
              MaybeTransformer<T, T>,
              CompletableTransformer {
      
          private final BehaviorProcessor<Integer> lifecycleBehavior;
      
          public LifecycleTransformer(@NonNull BehaviorProcessor<Integer> lifecycleBehavior) {
              this.lifecycleBehavior = lifecycleBehavior;
          }
      
          @Override
          public @NonNull ObservableSource<T> apply(@NonNull Observable<T> upstream) {
              return upstream.takeUntil(
                      lifecycleBehavior.skipWhile(
                              event -> event != LifecyclePublisher.ON_DESTROY_VIEW
                                      && event != LifecyclePublisher.ON_DESTROY
                                      && event != LifecyclePublisher.ON_DETACH
                      ).toObservable()
              );
          }
      
          @Override
          public @NonNull Publisher<T> apply(@NonNull Flowable<T> upstream) {
              return upstream.takeUntil(
                      lifecycleBehavior.skipWhile(
                              event -> event != LifecyclePublisher.ON_DESTROY_VIEW
                                      && event != LifecyclePublisher.ON_DESTROY
                                      && event != LifecyclePublisher.ON_DETACH
                      )
              );
          }
      
          @Override
          public @NonNull SingleSource<T> apply(@NonNull Single<T> upstream) {
              return upstream.takeUntil(
                      lifecycleBehavior.skipWhile(
                              event -> event != LifecyclePublisher.ON_DESTROY_VIEW
                                      && event != LifecyclePublisher.ON_DESTROY
                                      && event != LifecyclePublisher.ON_DETACH
                      )
              );
          }
      
          @Override
          public @NonNull MaybeSource<T> apply(@NonNull Maybe<T> upstream) {
              return upstream.takeUntil(
                      lifecycleBehavior.skipWhile(
                              event -> event != LifecyclePublisher.ON_DESTROY_VIEW
                                      && event != LifecyclePublisher.ON_DESTROY
                                      && event != LifecyclePublisher.ON_DETACH
                      )
              );
          }
      
          @Override
          public @NonNull CompletableSource apply(@NonNull Completable upstream) {
              return upstream.ambWith(
                      lifecycleBehavior.skipWhile(
                              event -> event != LifecyclePublisher.ON_DESTROY_VIEW
                                      && event != LifecyclePublisher.ON_DESTROY
                                      && event != LifecyclePublisher.ON_DETACH
                      ).take(1).flatMapCompletable(integer -> Completable.complete())
              );
          }
      }
      ```

   3. 缓存的使用

      对于缓存，我们一般是这样写：

      ```java
      cache.put(key,value);
      ```

      如果想在 RxJava 的链式调用中也使用缓存，还可以考虑使用 transformer 的方式：

      ```java
      import io.reactivex.rxjava3.core.FlowableTransformer;
      
      public class RxCache {
          public static <T> FlowableTransformer<T, T> transformer(final String key, final Cache cache) {
              return upstream -> upstream.map(t -> {
                  cache.put(key, t);
                  return t;
              });
          }
      }
      ```

      结合上述三种使用场景，封装了一个方法用于获取内容。在这里网络框架使用 Retrofit。虽然 Retrofit 本身支持通过 Interceptor 的方式来添加 Cache，但是某些业务场景下可能还是想用自己的 Cache，那么可以采用类似下面的封装。

      ```java
      public Flowable<ContentModel> getContent(Fragment fragment,ContentParam param,String cacheKey){
          return apiService.loadVideoContent(param)
              .compose(RxLifecycle.bind(fragment).<ContentModel>toLifecycleTransformer())
              .compose(RxJavaUtils.<ContentModel>flowableToMain())
              .compose(RxCache.<ContentModel>transformer(cacheKey,App.getInstance().cache));
      }
      ```

   4. 追踪 RxJava 的使用

      初学者可能会对 RxJava 内部的数据流向感到困惑，所以笔者写了一个类，用于追踪 RxJava 的数据流向，同时对于调试代码也很有帮助。
      
      下面是追踪 RxJava使用的源码：
      
      ```kotlin
      import android.os.Build
      import android.util.Log
      import androidx.annotation.RequiresApi
      import io.reactivex.rxjava3.core.ObservableTransformer
      import java.util.function.Function
      
      object RxTrace {
          @JvmField
          val LOG_NEXT_DATA = 1
      
          @JvmField
          val LOG_NEXT_EVENT = 2
      
          @JvmField
          val LOG_ERROR = 4
      
          @JvmField
          val LOG_COMPLETE = 8
      
          @JvmField
          val LOG_SUBSCRIBE = 16
      
          @JvmField
          val LOG_TERMINATE = 32
      
          @JvmField
          val LOG_DISPOSE = 64
      
          @RequiresApi(Build.VERSION_CODES.N)
          @JvmStatic
          fun <T> logObservable(tag: String, bitMask: Int): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  var upstream = upstream
      
                  if (bitMask and LOG_SUBSCRIBE > 0) {
                      upstream = upstream.compose(oLogSubscribe(tag))
                  }
      
                  if (bitMask and LOG_TERMINATE > 0) {
                      upstream = upstream.compose(oLogTerminate(tag))
                  }
      
                  if (bitMask and LOG_ERROR > 0) {
                      upstream = upstream.compose(oLogError(tag))
                  }
      
                  if (bitMask and LOG_COMPLETE > 0) {
                      upstream = upstream.compose(oLogComplete(tag))
                  }
      
                  if (bitMask and LOG_NEXT_DATA > 0) {
                      upstream = upstream.compose(oLogNext(tag))
                  } else if (bitMask and LOG_NEXT_EVENT > 0) {
                      upstream = upstream.compose(oLogNextEvent(tag))
                  }
      
                  if (bitMask and LOG_DISPOSE > 0) {
                      upstream = upstream.compose(oLogDispose(tag))
                  }
      
                  upstream
              }
      
          @RequiresApi(Build.VERSION_CODES.N)
          @JvmStatic
          fun <T> log(tag: String): ObservableTransformer<T, T> = ObservableTransformer { upstream ->
              upstream.compose(oLogAll(tag))
                  .compose(oLogNext(tag))
          }
      
          @RequiresApi(Build.VERSION_CODES.N)
          private fun <T> oLogAll(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.compose(oLogError(tag))
                      .compose(oLogComplete(tag))
                      .compose(oLogSubscribe(tag))
                      .compose(oLogTerminate(tag))
                      .compose(oLogDispose(tag))
              }
      
          private fun <T> oLogDispose(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.doOnDispose {
                      Log.i(tag, String.format("[dispose]"))
                  }
              }
      
          private fun <T> oLogNextEvent(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.doOnNext {
                      Log.i(tag, String.format("[onNext]"))
                  }
              }
      
          private fun <T> oLogNext(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.doOnNext { data ->
                      Log.i(tag, String.format("[onNext] -> %s", data))
                  }
              }
      
          private fun <T> oLogComplete(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.doOnComplete {
                      Log.i(tag, String.format("[onComplete]"))
                  }
              }
      
          @RequiresApi(Build.VERSION_CODES.N)
          private fun <T> oLogError(tag: String): ObservableTransformer<T, T> {
              val message = Function<Throwable, String> { throwable ->
                  if (throwable.message != null) throwable.message!! else throwable.javaClass.simpleName
              }
      
              return ObservableTransformer { upstream ->
                  upstream.doOnError { throwable ->
                      Log.i(tag, String.format("[onError] -> %s", message.apply(throwable)))
                  }
              }
          }
      
      
          private fun <T> oLogTerminate(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.doOnTerminate {
                      Log.i(tag, String.format("[terminate]"))
                  }
              }
      
          private fun <T> oLogSubscribe(tag: String): ObservableTransformer<T, T> =
              ObservableTransformer { upstream ->
                  upstream.doOnSubscribe {
                      Log.i(tag, String.format("[subscribe]"))
                  }
              }
      }
      ```
      
      以下是调用方式：
      
      ```kotlin
      Observable.just("tony", "cafei", "aaron")
          .compose(RxTrace.logObservable("first", RxTrace.LOG_SUBSCRIBE or RxTrace.LOG_NEXT_DATA))
          .subscribe(System.out::println)
      
      output-->first: [subscribe]
      output-->first: [onNext] -> tony
      output-->System.out: tony
      output-->first: [onNext] -> cafei
      output-->System.out: cafei
      output-->first: [onNext] -> aaron
      output-->System.out: aaron
      
      Observable.just("tony", "cafei", "aaron")
          .compose(RxTrace.logObservable("first", RxTrace.LOG_SUBSCRIBE or RxTrace.LOG_NEXT_DATA))
          .map { s -> s.toUpperCase(Locale.ROOT) }
          .compose(RxTrace.logObservable("second", RxTrace.LOG_NEXT_DATA))
          .subscribe(System.out::println)
      
      output-->first: [subscribe]
      output-->first: [onNext] -> tony
      output-->second: [onNext] -> TONY
      output-->System.out: TONY
      output-->first: [onNext] -> cafei
      output-->second: [onNext] -> CAFEI
      output-->System.out: CAFEI
      output-->first: [onNext] -> aaron
      output-->second: [onNext] -> AARON
      output-->System.out: AARON
      ```

compose 操作符和 Transformer 结合使用，一方面可以让代码看起来更加整洁，另一方面能够提高代码的复用性。RxJava 提倡链式调用。compose 能够防止链式被打破。

