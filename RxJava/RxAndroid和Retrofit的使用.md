### RxAndroid 和 Retrofit 的使用

#### RxAndroid

RxAndroid 下载地址：https://github.com/ReactiveX/RxAndroid

在 Android 中使用时需要新增一个调度器，用于将指定的操作切换到 Android 的主线程中运行，方便做一些更新 UI 的操作。RxAndroid 就提供了这样的调度器。

下面的例子展示了从网上获取图片，然后将图片转换成 Bitmap，最后显示到 ImageView 上的过程。

```kotlin
Observable.create<Bitmap> { emitter ->
    emitter.onNext(getBitmap())
}.subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .autoDispose(this)
    .subscribe {
        imageView.setImageBitmap(it)
    }
```

getBitmap() 方法显示了通过 URL 来请求网络图片，并将 HTTPURLConnection 获取的 InputStream 转换成 Bitmap。

```kotlin
private fun getBitmap(): Bitmap? {
    var con: HttpURLConnection? = null

    try {
        val url = URL("https://test.com/test.png")
        con = url.openConnection() as HttpURLConnection?
        con?.connectTimeout = 20000
        con?.connect()
        if (con?.responseCode == 200) {
            return BitmapFactory.decodeStream(con.inputStream)
        }
    } catch (e: Exception) {
        e.printStackTrace()
    } finally {
        con?.disconnect()
    }
    return null
}
```

#### Retrofit

Retrofit 是一个在 Android 开发中非常流行的网络框架，底层依赖 OKHttp。

Retrofit 的 GitHub 地址：https://github.com/square/retrofit

应用程序通过 Retrofit 请求网络，实际上是使用 Retrofit 接口层封装请求参数、Header、URL 等信息，之后由 OKHttp 完成后续的请求操作，在服务端返回数据之后，OKHttp 将原始的结果交给 Retrofit，Retrofit 再根据用户的需求对结果进行解析的过程。

Retrofit 支持大多数的 Http 方法。

Retrofit 的特点如下：

1. Retrofit 是可插拔的，允许不同的执行机制及其库用于执行 http 调用。允许 API 请求，与应用程序其余部分中任何现有线程模型或任务框架无缝组合。

   Retrofit 为常见的框架提供了适配器（Adapter），详见：https://github.com/square/retrofit/tree/master/retrofit-adapters

2. 允许不同的序列化格式及其库，用于将 Java 类型转换为其 http 表示形式，并将 http 实体解析为 Java 类型。

   Retrofit 为常见的序列化格式提供了转换器（Converter），详见：https://github.com/square/retrofit/tree/master/retrofit-converters

OKHttp 的特点如下：

- 支持 Http2/SPDY 黑科技。
- socket 自动选择最优路线，并支持自动重连。
- 拥有自动维护的 socket 连接池，减少握手次数。
- 拥有队列线程池，轻松写并发。
- 拥有 Interceptors 轻松处理请求与响应（比如透明 GZIP 压缩、LOGGING）。
- 基于 Headers 的缓存策略。

在实际开发中使用 Retrofit 时，一般都会对 OkHttp 做一些定制化的改动，以满足实际业务需求。

#### Retrofit 与 RxJava 的完美配合

Retrofit 是一个网络框架，如果想尝试响应式的编程方式，则可以结合 RxJava 一起使用。

下面会结合一个例子来讲解 Retrofit 和 RxJava 在 Android 上的使用。这个例子是将苏州市南门地区的 PM2.5、PM10、SO2 的数据展示到App上。在`http://pm25.in/`上可以找到获取这些数据的接口。在调用这些接口之前，需要去该网站注册，并申请一个 APPKey。

Retrofit 使用步骤如下：

1. ##### 添加 Retrofit 依赖。

   ```groovy
   implementation 'com.squareup.retrofit2:retrofit:2.9.0'
   implementation 'com.squareup.retrofit2:adapter-rxjava3:3.0.7'
   ```

2. ##### 创建 RetrofitManager。

   一般需要创建一个 Retrofit 的管理类，在这里创建一个名为 RetrofitManager 的类，方便在整个 App 中使用。

   ```kotlin
   object RetrofitManager {
       val retrofit: Retrofit
   
       init {
           val okHttpClient = OkHttpClient.Builder().apply {
               writeTimeout(30 * 1000, TimeUnit.MILLISECONDS)
               readTimeout(20 * 1000, TimeUnit.MILLISECONDS)
               connectTimeout(15 * 1000, TimeUnit.MILLISECONDS)
               addInterceptor(AndroidLoggingInterceptor.build())
           }.build()
           retrofit = Retrofit.Builder()
               .baseUrl(API_BASE_SERVICE_URL)
               .addCallAdapterFactory(RxJava3CallAdapterFactory.create())
               .addConverterFactory(GsonConverterFactory.create())
               .client(okHttpClient)
               .build()
       }
   }
   ```

   可以看到在 RetrofitManager 中对 OkHttp 添加了日志拦截器。它是用于记录 OKHttp 网络请求的日志拦截器，完全用 Kotlin 语言编写。

   GitHub 地址：https://github.com/fengzhizi715/saf-logginginterceptor

   在 RetrofitManager 中还添加了 Gson 的转换器，使用 Gson 来解析返回的 response 对象。

   ```groovy
   implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
   ```

3. ##### 创建 APIService

   ```kotlin
   val API_BASE_SERVICE_URL = "http://www.pm25.in/"
   
   interface ApiService {
       @GET("api/querys/pm2_5.json")
       fun pm25(@Query("city") city: String, @Query("token") token: String): Maybe<List<PM25Model>>
   
       @GET("api/querys/pm10.json")
       fun pm10(@Query("city") city: String, @Query("token") token: String): Maybe<List<PM10Model>>
   
       @GET("api/querys/so2.json")
       fun so2(@Query("city") city: String, @Query("token") token: String): Maybe<List<SO2Model>>
   }
   ```

4. ##### Retrofit 的使用

   下面的代码分别调用了 3 个接口，并过滤出了南门地区的相关数据。

   ```kotlin
   val apiService = RetrofitManager.retrofit.create(ApiService::class.java)
   
   apiService.pm25(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
       .observeOn(AndroidSchedulers.mainThread())
       .autoDispose(this)
       .subscribe({
           if (it == null) return@subscribe
           textView.append(
               "空气质量指数：${it.quality}\n" +
                       "PM 2.5 1 小时内平均：${it.pm2_5}\n" +
                       "PM 2.5 24 小时内滑动平均：${it.pm2_5_24h}\n"
           )
       }, { t: Throwable? ->
           t?.printStackTrace()
       })
   
   apiService.pm10(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
       .observeOn(AndroidSchedulers.mainThread())
       .autoDispose(this)
       .subscribe({
           if (it == null) return@subscribe
           textView.append(
               "PM 10 1 小时内平均：${it.pm10}\n" +
                       "PM 10 24 小时内滑动平均：${it.pm10_24h}\n"
           )
       }, { t: Throwable? ->
           t?.printStackTrace()
       })
   
   apiService.so2(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
       .observeOn(AndroidSchedulers.mainThread())
       .autoDispose(this)
       .subscribe({
           if (it == null) return@subscribe
           textView.append(
               "二氧化硫 1 小时内平均：${it.so2}\n" +
                       "二氧化硫 24 小时内滑动平均：${it.so2_24h}\n"
           )
       }, { t: Throwable? ->
           t?.printStackTrace()
       })
   ```

5. ##### 合并多个网络请求

   如果需要将多个请求合并起来，等所有请求完成之后，再使用合并函数将结果呈现给用户，可以考虑使用 zip 操作符：

   ```kotlin
   val pm25Maybe = apiService.pm25(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
   
   val pm10Maybe = apiService.pm10(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
   
   
   val so2Maybe = apiService.so2(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
   
   Maybe.zip<PM25Model, PM10Model, SO2Model, ZipObject>(pm25Maybe, pm10Maybe, so2Maybe,
       Function3<PM25Model, PM10Model, SO2Model, ZipObject> { t1, t2, t3 ->
           ZipObject(t1.aqi, t1.pm2_5, t1.pm2_5_24h, t2.pm10, t2.pm10_24h, t3.so2, t3.so2_24h)
       })
       .subscribeOn(Schedulers.io())
       .observeOn(AndroidSchedulers.mainThread())
       .autoDispose(this)
       .subscribe({
           textView.append(
               "空气质量指数：${it.quality}\n" +
                       "PM 2.5 1 小时内平均：${it.pm2_5}\n" +
                       "PM 2.5 24 小时内滑动平均：${it.pm2_5_24h}\n" +
                       "PM 10 1 小时内平均：${it.pm10}\n" +
                       "PM 10 24 小时内滑动平均：${it.pm10_24h}\n" +
                       "二氧化硫 1 小时内平均：${it.so2}\n" +
                       "二氧化硫 24 小时内滑动平均：${it.so2_24h}\n"
           )
       }, { t: Throwable? ->
           t?.printStackTrace()
       })
   ```

6. ##### 返回默认值

   有时，网络请求失败可以使用 `onErrorReturn`操作符，返回一个空的对象作为默认值。

7. ##### 多个网络请求嵌套使用

   若是 A 请求完成之后，才能去调用 B 请求，则可以考虑使用 flatMap 操作符。

8. ##### 重试机制

   对于一些重要的接口，需要采用重试机制。因为有些时候用户的网络环境比较差，第一次请求接口超时了，那么再一次请求可能就会成功。
   
   ```kotlin
   class RetryWithDelay(private val maxRetries: Int, private val retryDelayMillis: Int) :
       Function<Flowable<Throwable>, Publisher<*>> {
       private var retryCount: Int = 0
   
       init {
           this.retryCount = 0
       }
   
       override fun apply(attempts: Flowable<Throwable>): Publisher<*> {
           return attempts.flatMap { t: Throwable? ->
               if (++retryCount <= maxRetries) {
                   Log.i(
                       "retry",
                       "apply: get error, it will try after $retryDelayMillis millisecond,retry count $retryCount"
                   )
                   Flowable.timer(retryDelayMillis.toLong(), TimeUnit.MILLISECONDS)
               } else {
                   Flowable.error(t)
               }
           }
       }
   }
   
   //调用方式
   apiService.pm25(Constant.CITY, Constant.TOKEN)
       .subscribeOn(Schedulers.io())
       .flatMap { list ->
           Maybe.just(list.firstOrNull { it.position_name == "南门" })
       }
   	//延迟一秒，重试3次
       .retryWhen(RetryWithDelay(3, 1000))
       .autoDispose(this)
       .subscribe({}, { t: Throwable? -> t?.printStackTrace() })
   ```
   
   
