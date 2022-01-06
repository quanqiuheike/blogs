# Android的协程概念以及使用

## [GitHub地址](https://github.com/hltj/kotlinx.coroutines-cn/blob/master/README.md#using-in-your-projects)|[Kotlin中文官网地址](https://www.kotlincn.net/docs/reference/coroutines/basics.html)

## Gradle中的环境配置依赖

```groovy
 implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.6.0-RC'
 implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.6.0-RC'

```

### 创建协程的方式



```kotlin
方式1：
GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L) // 非阻塞的等待 1 秒钟（默认时间单位是毫秒）
        println("World!") // 在延迟后打印输出
}

方式2：
fun main() = runBlocking {
    launch {
      delay(5000L)
      print(".")
    }
}

```

### 第一个协程程序



`GlobalScope.launch`和`delay`一起使用

`Thread`和`Thread.sleep`一起使用

**Delay**:是一个特殊的 挂起函数 ，它不会造成线程阻塞，但是会挂起协程，并且只能在协程中使用。

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L) // 非阻塞的等待 1 秒钟（默认时间单位是毫秒）
        println("World!") // 在延迟后打印输出
    }
    println("Hello,") // 协程已在等待时主线程还在继续
    Thread.sleep(2000L) // 阻塞主线程 2 秒钟来保证 JVM 存活，如果此时阻塞为1秒钟，则不打印World.
}
```

可以将 `GlobalScope.launch { …… }` 替换为 `thread { …… }`，并将 `delay(……)` 替换为 `Thread.sleep(……)` 达到同样目的。 试试看（不要忘记导入 `kotlin.concurrent.thread`）。

如果你首先将 `GlobalScope.launch` 替换为 `thread`，编译器会报以下错误：

```bash
Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function
```

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L) // 非阻塞的等待 1 秒钟（默认时间单位是毫秒）
        println("World!") // 在延迟后打印输出
    }
    println("Hello,") // 协程已在等待时主线程还在继续
    Thread.sleep(2000L) // 阻塞主线程 2 秒钟来保证 JVM 存活
}
```

