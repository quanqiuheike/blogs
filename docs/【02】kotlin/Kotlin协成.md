# Android Kotlin协程的使用

[官网：Android 上的 Kotlin 协程](https://developer.android.google.cn/kotlin/coroutines)

## 依赖项信息

```groovy
java
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
}

kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9")
}
```

* 所有协程都必须在一个作用域内运行。一个 `CoroutineScope` 管理一个或多个相关的协程。`viewModelScope` 是预定义的 `CoroutineScope`，包含在 `ViewModel` KTX 扩展中。

- `launch` 是一个函数，用于创建协程并将其函数主体的执行分派给相应的调度程序。
- `Dispatchers.IO` 指示此协程应在为 I/O 操作预留的线程上执行。



同步请求会阻塞线程，如需将执行操作移出主线程，最简单的方法是创建一个新的协程，然后在 I/O 线程上执行网络请求。

```kotlin
class LoginViewModel(
    private val loginRepository: LoginRepository
): ViewModel() {

    fun login(username: String, token: String) {
        // Create a new coroutine to move the execution off the UI thread
        viewModelScope.launch(Dispatchers.IO) {
            val jsonBody = "{ username: \"$username\", token: \"$token\"}"
            loginRepository.makeLoginRequest(jsonBody)
        }
    }
}
```





```kotlin
suspend fun fetchTwoDocs() =
    coroutineScope {
        val deferredOne = async { fetchDoc(1) }
        val deferredTwo = async { fetchDoc(2) }
        deferredOne.await()
        deferredTwo.await()
    }
    
    
    
    suspend fun fetchTwoDocs() =        // called on any Dispatcher (any thread, possibly Main)
    coroutineScope {
        val deferreds = listOf(     // fetch two docs at the same time
            async { fetchDoc(1) },  // async returns a result for the first doc
            async { fetchDoc(2) }   // async returns a result for the second doc
        )
        deferreds.awaitAll()        // use awaitAll to wait for both network requests
    }
```

[`Job`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/index.html) 是协程的句柄。使用 `launch` 或 `async` 创建的每个协程都会返回一个 `Job` 实例，该实例是相应协程的唯一标识并管理其生命周期。您还可以将 `Job` 传递给 `CoroutineScope` 以进一步管理其生命周期，如以下示例所示：

```kotlin
class ExampleClass {
    ...
    fun exampleMethod() {
        // Handle to the coroutine, you can control its lifecycle
        val job = scope.launch {
            // New coroutine
        }

        if (...) {
            // Cancel the coroutine started above, this doesn't affect the scope
            // this coroutine was launched in
            job.cancel()
        }
    }
}
```



协程又称微线程，从名字可以看出，协程的粒度比线程更小，并且是用户管理和控制的，多个协程可以运行在一个线程上面。目前线程中影响性能的特性：

* 使用锁机制
* 线程间的上下文切换
* 线程运行和阻塞状态的切换


以上任意一点都是很消耗cpu性能的。相对来说协程是由程序自身控制，没有线程切换的开销，且不需要锁机制，因为在同一个线程中运行，不存在同时写变量冲突，在协程中操作共享资源不加锁，只需要判断状态就行了，所以执行效率比线程高的多。

协程是我们在 Android 上进行异步编程的推荐解决方案。

- **轻量**：您可以在单个线程上运行多个协程，因为协程支持[挂起](https://kotlinlang.org/docs/reference/coroutines/basics.html)，不会使正在运行协程的线程阻塞。挂起比阻塞节省内存，且支持多个并行操作。
- **内存泄漏更少**：使用[结构化并发](https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency)机制在一个作用域内执行多项操作。
- **内置取消支持**：[取消](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html)操作会自动在运行中的整个协程层次结构内传播。
- **Jetpack 集成**：许多 Jetpack 库都包含提供全面协程支持的[扩展](https://developer.android.google.cn/kotlin/ktx)。某些库还提供自己的[协程作用域](https://developer.android.google.cn/topic/libraries/architecture/coroutines)，可供您用于结构化并发。

```kotlin
class MainActivity : AppCompatActivity() {

    var tv1: TextView? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        tv1 = findViewById(R.id.tv1)
        
        //在主线程启动一个协程,不会卡顿
        GlobalScope.launch(Dispatchers.Main) {
            // launch coroutine in the main thread
            for (i in 10 downTo 1) { // countdown from 10 to 1
                tv1?.text = "Countdown $i ..." // update text
              //挂起协程，不会阻塞线程，只能在协程中使用
                delay(1000) // wait a second
            }
            tv1?.text = "Done!"
        }
    }
}

```

还有一点需要注意，上面的代码中，我们使用 delay(1000) 来做延时操作，delay 是一个特殊的函数，这里暂且称之为挂起函数，它不会阻塞线程，但是会挂起协程，而且它只能在协程中使用。

再延伸一点，我们能否用 Thread.sleep(1000) 来代替 delay(1000) , 答案是不能的。我们的协程是在主线程的基础上创建的，本质上是主线程的小逻辑单元，用 Thread.sleep(1000) 会直接卡死 UI 主线程。

##### 取消协程工作

在java开发Android应用时，我们用子线程执行耗时操作，当然我们也会中断子线程来达到取消耗时操作的目的。
那么我们在协程中执行耗时操作的时候，改怎么取消呢？

GlobalScope.launch 的返回值是 Job 对象，用 job.cancel() 来取消协程。例子如下：

```kotlin
class MainActivity : AppCompatActivity() {

    var tv1: TextView? = null
    var mCancelButton: Button? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        tv1 = findViewById(R.id.tv1)
        mCancelButton = findViewById(R.id.cancel)

        //在主线程启动一个协程
        var job = GlobalScope.launch(Dispatchers.Main) {
            // launch coroutine in the main thread
            for (i in 10 downTo 1) { // countdown from 10 to 1
                tv1?.text = "Countdown $i ..." // update text
                delay(1000) // wait a second
            }
            tv1?.text = "Done!"
        }

        mCancelButton?.setOnClickListener {
            job.cancel() //取消协程工作
            mCancelButton?.text = "已经取消了"
        }
    }
}

```

## launch 参数详解

在上文中，我们已经学会了使用 `GlobalScope.launch` 创建一个协程，下面我们来看看创建协程所需要的参数,`launch` 的参数有三个，依次为`协程上下文`、`协程启动模式`、`协程体`：

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,  //上下文
    start: CoroutineStart = CoroutineStart.DEFAULT,   //启动模式
    block: suspend CoroutineScope.() -> Unit   //协程体
): Job

```

* 启动模式不是一个很复杂的概念，不过我们暂且不管，默认直接允许调度执行。

* 上下文可以有很多作用，包括携带参数，拦截协程执行等等，多数情况下我们不需要自己去实现上下文，只需要使用现成的就好。上下文有一个重要的作用就是线程切换，Dispatchers.Main就是一个官方提供的上下文，它可以确保 launch 启动的协程体运行在UI线程当中（除非你自己在 launch 的协程体内部进行线程切换、或者启动运行在其他有线程切换能力的上下文的协程）。

* 协程体就是我们具体执行的代码

|Dispatchers	|用途	|使用场景
|Dispatchers.Main	|主线程，和UI交互，执行轻量任务	|1.call suspend functions。2. call UI functions。 3. Update LiveData
|Dispatchers.IO|	用于网络请求和文件访问|	1. Database。 2.Reading/writing files。3. Networking
|Dispatchers.Default	|CPU密集型任务	|1. Sorting a list。 2.Parsing JSON。 3.DiffUtils
|Dispatchers.Unconfined	|不限制任何制定线程|	高级调度器，不应该在常规代码里使用

#### viewModelScope 的作用

```kotlin
viewModelScope.launch(Dispatchers.IO) {
    // do something
}
```

- viewModelScope 是一个内置的 CoroutineScope，包含在 **ViewModel** KTX 扩展中
- viewModelScope 需要在 ViewModel 中使用,`ViewModel` 包含一组可直接与协程配合使用的 KTX 扩展。这些扩展是 [`lifecycle-viewmodel-ktx` 库](https://developer.android.google.cn/kotlin/ktx#viewmodel)，在本指南中有用到。
- Dispatchers.IO 是为了注明当前 coroutine 需要运行在一个用来执行 I/O 操作的线程中

当用户从一个界面退出，其对应的 ViewModel 会被销毁，相应的此 ViewModel 的 viewModelScope 会被自动取消 (cancelled)，所有运行在 viewModelScope 中的 coroutine 也会被取消。

所以，没人会用 CoroutineScope.launch，原因就是当 UI 界面退出，但是 coroutine 运行时间过长，这个 coroutine 将得不到清理。

```kotlin
class HomeViewModel: BaseViewModel() {

    val bottomBarLiveData: MutableLiveData<HomePageImg> by lazy {
        MutableLiveData<HomePageImg>()
    }

    fun getBottomData(context: Context) {
        launch({
            val homePageImg: HomePageImg = withContext(Dispatchers.IO) {
                val tabLists = AssetsUtils.getString(context, AppConfigsContants.BOTTOM_TAB_NAV)
                val res =  GlobalGsonUtils.fromJson<HomePageImg>(tabLists, HomePageImg::class.java)
                res
            }
            bottomBarLiveData.value = homePageImg
        })
        viewModelScope.launch(Dispatchers.IO) {
            // do something
        }
    }
}
```



#### withContext的性能

对于提供主线程安全性，withContext 与回调或 RxJava一样快。在某些情况下，甚至可以使用协程上下文 withContext 来优化回调。如果一个函数将对数据库进行10次调用，那么您可以告诉 Kotlin在外部的 withContext中调用一次切换。尽管数据库会重复调用 withContext，但是他它将在同一个调度器下，寻找最快路径。此外，Dispatchers.Default和 Dispatchers.IO 之间的协程切换已经过优化，以尽可能避免线程切换。


```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //在子线程启动一个协程
        GlobalScope.launch(Dispatchers.IO) {

            //发起一个网络请求
            var result = HttpUtil.get("https://www.baidu.com")

            Log.e("zhaoyanjun:22", "${Thread.currentThread().name}")

            withContext(Dispatchers.Main) {
                //网络请求成功以后，到主线程更新UI
                Log.e("zhaoyanjun:33", "${Thread.currentThread().name}")
            }

            //再次回到子线程的协程
            Log.e("zhaoyanjun:44", "${Thread.currentThread().name}")
        }
    }
}

```



#### withContext(Dispatchers.IO) 与 viewModelScope.launch(Dispatchers.IO) 的区别

首先 withContext 也是一个 suspend function，所以 withContext 必须在 suspend 函数，或者 coroutine 中被调用。

- viewModelScope.launch(Dispatchers.IO)，即 launch 的 coroutine 运行之后，不再返回数据
- withContext(Dispatchers.IO) 会将 coroutine 的返回值，作为当前函数的返回

例如：

```kotlin
suspend fun updatePage() {                      
    val result = getDataFromServer("www.sunzhongwei.com")  
    show(result)                               
}

suspend fun getDataFromServer(url: String) =                 
    withContext(Dispatchers.IO) {              
        // 执行网络请求          
    }                                          
```



## withContext

withContext这个函数主要可以切换到指定的线程，并在闭包内的逻辑执行结束之后，自动把线程切回去继续执行

```kotlin
public suspend fun <T> withContext(
    context: CoroutineContext,
    block: suspend CoroutineScope.() -> T
):
```

`withContext` 被 `suspend` 修饰，说明 `suspend` 是一个挂起函数。

`withContext(Dispatchers.IO)` 定义一段代码块，这个代码块将在调度器 `Dispatchers.IO`中运行,方法块中的任何代码总是会运行在 `IO`调度器中。

```kotlin
// Dispatchers.Main
suspend fun fetchDocs() {
    // Dispatchers.Main
    val result = get("developer.android.com")
    // Dispatchers.Main
    show(result)
}

// Dispatchers.Main
suspend fun get(url: String) =
    // Dispatchers.IO
    withContext(Dispatchers.IO) {
        // Dispatchers.IO
        /* perform blocking network IO here */
    }
    // Dispatchers.Main
}

```



#### kotlin 中 GlobalScope 类提供了几个创建协程的构造函数：

- launch： 创建协程
- async ： 创建带返回值的协程，返回的是 Deferred 类
- withContext：不创建新的协程，指定协程上运行代码块
- runBlocking：不是 GlobalScope 的 API，可以独立使用，区别是 runBlocking 里面的 delay 会阻塞线程，而 launch 创建的不会



```kotlin
coroutineScope.launch(Dispatchers.Main) {      //  在 UI 线程开始
    val image = withContext(Dispatchers.IO) {  // 切换到 IO 线程，并在执行完成后切回 UI 线程
        getImage(imageId)                      // 将会运行在 IO 线程
    }
    avatarIv.setImageBitmap(image)             // 回到 UI 线程更新 UI
} 
```



```kotlin
 launch({
            val homePageImg: HomePageImg = withContext(Dispatchers.IO) {
                val tabLists = AssetsUtils.getString(context, AppConfigsContants.BOTTOM_TAB_NAV)
                val res =  GlobalGsonUtils.fromJson<HomePageImg>(tabLists, HomePageImg::class.java)
                res
            }
            bottomBarLiveData.value = homePageImg
        })
```





```kotlin
GlobalScope.launch {
            kotlin.runCatching {
               val res =  withContext(Dispatchers.IO) {
                    NetworkManager.getInstance().getService(ShareService::class.java).share(map)
                }
                Log.d("LUO","------res: ${res.toString()}")
                handleResponse(res)
            }

        }


 GlobalScope.launch {
            withContext(Dispatchers.IO) {
                kotlin.runCatching {
                    NetworkManager.getInstance().getService(DetailService::class.java).sendComment(map)
                }.onSuccess {
                    withContext(Dispatchers.Main) {

                    }
                }.onFailure {
                    if (null != it && !TextUtils.isEmpty(it.localizedMessage))
                        LegoToastUtils.show(it.localizedMessage)
                }
            }
        }


fun loadRoamMsg(params: MsgReq, onSuccess: (result: MsgListResponse?) -> Unit, onError: (code: Int, desc: String?) -> Unit) {
        GlobalScope.launch {
            try {
                val responseData = withContext(Dispatchers.IO) {
                    NetworkManager.getInstance().getService(ChatService::class.java).loadRoamMsg(params)
                }
                val realData = handleResponse(responseData)
                if (200 == responseData.code) {
                    onSuccess.invoke(realData)
                } else {
                    onError.invoke(responseData.code, responseData.msg)
                }
            } catch (e: Exception) {
                onError.invoke(-1, e.message)
            }
        }
    }
    
       GlobalScope.launch(Dispatchers.Main) {
                                LegoToastUtils.show("分享成功")
                            }
```







##### Refrence&Thanks

##### [withContext(Dispatchers.IO) 与 viewModelScope.launch(Dispatchers.IO) 的区别](https://www.sunzhongwei.com/withcontext-dispatchers-io-and-viewmodelscope-launch-dispatchers-io-difference)

##### [Kotlin实战指南十三：协程](https://blog.csdn.net/zhaoyanjun6/article/details/95626034)



项目：

