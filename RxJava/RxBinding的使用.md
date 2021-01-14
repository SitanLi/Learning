### RxBinding 的使用

#### RxBinding介绍

RxBinding 是 Jake Wharton 大神写的框架，它的 API 能够把 Android 平台和兼容包内的 UI 控件变为 Observable 对象，这样就可以把 UI 控件的事件当作 RxJava 中的数据流来使用了。

RxBinding 的 GitHub 地址：https://github.com/JakeWharton/RxBinding

RxBinding 的优点：

- 它是对 Android View 事件的拓展，它使得开发者可以对 View 事件使用 RxJava 的各种操作。
- 提供了与 RxJava 一致的回调，使得代码简洁明了，尤其是页面上充斥着大量的监听时间，以及各种各样的匿名内部类。
- 几乎支持所有的常用控件及事件（v4、v7、design、recyclerview等），另外每个库还有对应的 Kotlin 支持库。

#### 响应式的 Andnroid UI

对 UI 事件的响应几乎是 Android App 开发的基本部分，但是 Android SDK 对 UI 事件的处理有些复杂，我们通常需要使用各种 listeners、handlers、TextWatchers 和其他组件等组合来响应 UI 事件。这些组件中的每一个都需要编写大量的样板代码，更为糟糕的是，实现这些不同组件的方式并不一致。例如  OnClickListener 处理 OnClick 事件与实现 TextWatcher 的方式完全不同。

RxBinding就是将处理 UI 事件的方法标准化。一旦将 View 事件转换为 Observable，它将发射数据流形式的 UI 事件，我们就可以订阅这个数据流，这与订阅其他 Observable 的方式相同。接下来，看看如何实现 OnClick 事件：

```kotlin
val button = findViewById<Button>(R.id.button)
button.clicks().subscribe {
    Toast.makeText(this, "click", Toast.LENGTH_SHORT).show()
}
```

这种方法不仅更简洁，而且是一种标准的实现方式，我们可以将其应用于整个 App 的所有 UI 事件。例如，捕获文本输入与捕获点击事件的模式是一样的：

```kotlin
val editText = findViewById<EditText>(R.id.edit_text)
editText.textChanges().subscribe {
    Toast.makeText(this, it, Toast.LENGTH_SHORT).show()
}
```

#### RxBinding 使用场景

RxBinding 可以应用于整个 App 的所有 UI 事件，下面列举一些 RxBinding 比较常见的使用场景。

1. ##### 点击事件

   按钮点击事件是每一个 App 都会用到的场景，可以使用 RxView 的 ckicks(@NonNull View view) 方法来绑定 UI 控件，kotlin 使用 View 的拓展方法 clicks() 来绑定事件。

   ```kotlin
   val button = findViewById<Button>(R.id.button)
   button.clicks().subscribe {
       Toast.makeText(this, "click", Toast.LENGTH_SHORT).show()
   }
   ```

2. ##### 长点击事件

   长点击事件也是一个比较常见的事件，可以使用 RxView 的 longClicks(@NonNull View view) 方法来绑定 UI 控件。

   ```kotlin
   button.longClicks().subscribe {
       Toast.makeText(this, "longClick", Toast.LENGTH_SHORT).show()
   }
   ```

3. ##### 防止重复点击

   在弱网环境下，经常会遇到点击某个按钮没有响应的情况，使得用户多次点击按钮，从而造成事件的多次触发，显然这是我们不愿意看到的情况。可以利用 throttleFirst 操作符获取某段时间内的第一次点击事件。

   ```kotlin
   fun <T> preventDuplicateClicksTransformer() =
       ObservableTransformer<T, T> { upstream ->
           upstream.throttleFirst(1000, TimeUnit.MILLISECONDS)
       }
   ```

   调用方式：

   ```kotlin
   var count = 0
   button.clicks().compose(RxJavaUtils.preventDuplicateClicksTransformer())
   	//使用AutoDispose监听生命周期取消订阅
       .autoDispose(this)
       .subscribe {
           Toast.makeText(this, "click:${count++}", Toast.LENGTH_SHORT).show()
       }
   ```

4. ##### 表单的验证

   App内常见的表单验证是用户登录页面，我们需要对用户名、密码做一些校验。

   ```kotlin
   Observable.combineLatest(
       phone.textChanges(),
       password.textChanges(),
       BiFunction<CharSequence, CharSequence, ValidationResult> { t1, t2 ->
           commit.isEnabled = t1.isNotEmpty() && t2.isNotEmpty()
           when {
               t1.isEmpty() -> {
                   result.flag = false
                   result.message = "手机号不能为空"
               }
               t1.length != 11 -> {
                   result.flag = false
                   result.message = "手机号码需要11位"
               }
               t2.isEmpty() -> {
                   result.flag = false
                   result.message = "密码不能为空"
               }
               else -> result.flag = true
           }
           result
       }).autoDispose(this)
       .subscribe(System.out::println)
   
   commit.clicks().compose(RxJavaUtils.preventDuplicateClicksTransformer())
       .autoDispose(this)
       .subscribe {
           if (!result.flag) {
               Toast.makeText(this, result.message, Toast.LENGTH_SHORT).show()
               return@subscribe
           }
           Toast.makeText(this, "模拟登陆成功！", Toast.LENGTH_SHORT).show()
       }
   ```

5. ##### 获取验证码倒计时

   用户注册账号时，一般需要获取验证码来验证手机号码，在验证码的等待过程中，App的界面上通常会有一个倒计时，提示我们剩余xx秒可以重新获取验证码。

   ```kotlin
   val MAX_COUNT_TIME = 60L
   
   request.clicks()
       .throttleFirst(MAX_COUNT_TIME, TimeUnit.SECONDS)
       .flatMap {
           //TODO 请求验证码网络请求
           request.isEnabled = false
           request.text = "剩余${MAX_COUNT_TIME}秒"
           Observable.interval(1, TimeUnit.SECONDS, Schedulers.io()).take(MAX_COUNT_TIME)
       }.map { time ->
           //将递增的数字替换成递减的倒计时数字
           MAX_COUNT_TIME - (time + 1)
       }.observeOn(AndroidSchedulers.mainThread())
       .autoDispose(this)
       .subscribe {
           if (it == 0L) {
               request.isEnabled = true
               request.text = "获取验证码"
           } else {
               request.text = "剩余${it}秒"
           }
       }
   ```

6. ##### 对 RecyclerView 的支持

   RxBinding 提供了一个`rxbinding-recyclerview`的库，专门用于对 RecyclerView 的支持。

   其中，RxRecyclerView 提供了几个状态的观察：

   - scrollStateChanges 观察 RecyclerView 的滚动状态。
   - scrollEvents 观察 RecyclerView 的滚动事件
   - childAttachStateChangeEvents 观察 child view 的 detached 状态，当 LayoutManager 或者 RecyclerView 认为不再需要一个 child view 时，就会调用这个方法。如果 child view 占用资源，则应当释放资源。

   下面是观察 RecyclerView 滚动状态的示例：

   ```kotlin
   recyclerView.scrollStateChanges()
       .autoDispose(this)
       .subscribe {
           Log.i("scrollState", "scrollState = $it")
       }
   ```

   在 Adapter 的 onBindViewHolder() 中，可以使用 clicks() 来绑定 itemView 的点击事件。

   ```kotlin
   override fun onBindViewHolder(holder: ViewHolder, position: Int) {
       holder.textView.text = list[position]
       holder.textView.clicks().subscribe {
           Toast.makeText(holder.textView.context, list[position], Toast.LENGTH_SHORT).show()
       }
   }
   ```

7. ##### 对 UI 控件进行多次监听

   可以利用 RxJava 的操作符，例如 publish、share 或 replay，实现对 UI 控件的多次监听。

   ```kotlin
   val clickObservable = button.clicks().share()
   clickObservable
       .autoDispose(this)
       .subscribe {
           Toast.makeText(this, "对 button 的第一次监听", Toast.LENGTH_SHORT).show()
       }
   clickObservable
       .autoDispose(this)
       .subscribe {
           Toast.makeText(this, "对 button 的第二次监听", Toast.LENGTH_SHORT).show()
       }
   ```

#### RxBinding 结合 RxPermissions 的使用

1. ##### RxPermission 介绍

   在处理运行时权限使，通常需要两步：

   - 申请权限；
   - 处理权限回调，根据授权的情况进行回调。

   RxPermissions 的出现可以简化这些步骤，它是基于 RxJava 开发的 Android 框架，旨在帮助 Android 6.0 之后处理运行时权限的检测。

   RxPermissions 的 GitHub 地址：https://github.com/tbruyelle/RxPermissions

2. ##### RxBinding 结合 RxPermissions

   在 RxPermissions 使用之前，需要先创建 RxPermissions 的实例：

   ```kotlin
   val rxPermissions = RxPermissions(this)
   ```

   1. ###### 在 RxBinding 中使用 RxPermissions

      单击按钮后，会出现一个弹框，让用户选择是否授予权限。如果选择允许，就可以完成后面的过程。如果不允许，则下次点击的时候，还会出现授权对话框。

      ```kotlin
      button.clicks()
          .autoDispose(this)
          .subscribe {
              rxPermissions.request(Manifest.permission.CALL_PHONE)
                  .autoDispose(button)
                  .subscribe { granted ->
                      if (granted) {
                          val intent = Intent(Intent.ACTION_CALL)
                          intent.data = Uri.parse("tel:10000")
                          startActivity(intent)
                      } else {
                          Toast.makeText(this, "授权失败", Toast.LENGTH_SHORT).show()
                      }
                  }
          }
      ```

   2. ###### RxBinding 结合 compose，使用 RxPermissions

      对上述代码稍作修改，RxBinding 可以结合 compose 操作符来使用 RxPermissions。

      ```kotlin
      button.clicks()
          .compose(rxPermissions.ensure(Manifest.permission.CALL_PHONE))
          .autoDispose(this)
          .subscribe { granted ->
              if (granted) {
                  val intent = Intent(Intent.ACTION_CALL)
                  intent.data = Uri.parse("tel:10000")
                  startActivity(intent)
              } else {
                  Toast.makeText(this, "授权失败", Toast.LENGTH_SHORT).show()
              }
          }
      ```

   3. ###### 使用多个权限的用法

      RxPermissions 也支持申请多个权限，下面的例子展示了同时申请 CAMERA 和 READ_CONTACTS 的权限。

      ```kotlin
      button.clicks()
          .compose(
              rxPermissions.ensure(
                  Manifest.permission.CAMERA,
                  Manifest.permission.READ_CONTACTS
              )
          )
          .autoDispose(this)
          .subscribe { granted ->
              if (granted) {//两者权限都同意
                  val intent = Intent(Intent.ACTION_CALL)
                  intent.data = Uri.parse("tel:10000")
                  startActivity(intent)
              } else {//任意一次授权失败，均失败
                  Toast.makeText(this, "授权失败", Toast.LENGTH_SHORT).show()
              }
          }
      ```

   #### RxBinding 使用的注意点

   在 Android App 中使用 RxJava 存在着一个最大的缺点，即不完整的订阅会导致内存泄漏。当 Android 系统尝试销毁包含着正在运行 Observable 的 Activity/Fragment 时，由于 Observable 正在运行，其观察者仍然会持有对该 Activity/Fragment 的引用，因此系统无法对此 Activity/Fragment 进行垃圾回收。由于 Activity 是大的对象，因此可能会导致严重的内存管理问题，很可能会导致 OOM。

   因此要及时取消订阅，防止内存泄漏。

