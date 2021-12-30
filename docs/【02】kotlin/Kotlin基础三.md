
---
### 35.Kotlin语言的可空性特点
```kotlin
// TODO 35.Kotlin语言的可空性特点
fun main() {

    // TODO 第一种情况：默认是不可空类型，不能随意给null
    var name: String = "Derry"

    // 提示：不能是非空类型String的值
    // name = null

    println(name)

    // TODO 第二种情况：声明时指定为可空类型
    var name2: String ?
    name2 = null
    // name2 = "Derry"
    println(name2)
}
```
---
### 36.Kotlin语言的安全调用操作符
```kotlin
// TODO 36.Kotlin语言的安全调用操作符
fun main() {
    var name: String? = "zhangsan"
    // name = null

    // name.capitalize() // name是可空类型的 可能是null，想要使用name，必须给出补救措施

    val r = name?.capitalize() // name是可空类型的 如果真的是null，?后面这一段代码不执行，就不会引发空指针异常
    println(r)
}
```
---
### 37.在Kotlin中使用带let的安全调用
```kotlin
// TODO 37.在Kotlin中使用带let的安全调用
fun main() {
    var name: String? = null
    name = "Derry"
    name = ""

    // name是可空类型的 如果真的是null，?后面这一段代码不执行，就不会引发空指针异常
    val r =name?.let {
        // it == name 本身
        // 如果能够执行到这里面的，it 一定不为null

        if (it.isBlank()) { // 如果name是空值 "" 没有内容
            "Default"
        } else {
            "[$it]"
        }
    }
    println(r)
}
```
---
### 38.Kotlin语言中的非空断言操作符特点
```kotlin
// TODO 38.Kotlin语言中的非空断言操作符特点
fun main() {
   var name: String? = null

    // name.capitalize() // name是可空类型的 可能是null，想要使用name，必须给出补救措施

    // name = "derry"
    // 补救措施  我们已经学习了 ?
    val r = name!!.capitalize() // !! 断言 不管name是不是null，都执行，这个和java一样了
    println(r)

    // 结论：规矩：如果百分百能够保证name是有值的，那么才能使用断言 !!， 否则有Java 空指针异常的风险
}
```

---
### 39.Kotlin语法中对比使用if判断null值情况
```kotlin

// TODO 39.Kotlin语法中对比使用if判断null值情况
fun main() {
    var name: String? = null
    name = "DDD"

    // name.capitalize() // name是可空类型的 可能是null，想要使用name，必须给出补救措施

    if (name != null) { // if也算是补救措施，和Java一样了
        val r = name.capitalize()
        println(r)
    } else {
        println("name is null")
    }
}
```
---
### 40.在Kotlin空合并操作符
```kotlin

// TODO 40.在Kotlin空合并操作符
fun main() {
    var info: String? = "李小龙"
     info = null

    // 空合并操作符  xxx ?: "原来你是null啊"    "如果xxx等于null，就会执行 ?: 后面的区域"
    println( info ?: "原来你是null啊" )


    // let函数 + 空合并操作符
    println(info?.let { "【$it】" } ?: "原来你是null啊2")
}
```
---
### 41.在Kotlin语法中异常处理与自定义异常特点
```kotlin

import java.lang.Exception
import java.lang.IllegalArgumentException

// TODO 41.在Kotlin语法中异常处理与自定义异常特点
fun main() {
   try {
       var info: String? = null

       checkException(info)

       println(info!!.length)

   }catch (e: Exception) {
       println("啊呀:$e")
   }
}

fun checkException(info: String?) {
   info ?: throw CustomException()
}

class CustomException : IllegalArgumentException("你的代码太不严谨了")

```
---
### 42.Kotlin语言的先决条件函数

// TODO 42.Kotlin语言的先决条件函数
fun main() {
    var value1: String ? = null
    var value2: Boolean = false

    // checkNotNull(value1) // java.lang.IllegalStateException: Required value was null.

    // requireNotNull(value1) // java.lang.IllegalArgumentException: Required value was null.

    require(value2) // java.lang.IllegalArgumentException: Failed requirement.
}
```
---
### 43.Kotlin语言的substring
```kotlin
const val INFO = "Derry is Success Result"

// TODO 43.Kotlin语言的substring
fun main() {
    val indexOf = INFO.indexOf('i')
    println(INFO.substring(0, indexOf))
    println(INFO.substring(0 until indexOf)) // KT基本上用此方式： 0 until indexOf
}
```
---
### 44.Kotlin语言的split操作
```kotlin
// TODO 44.Kotlin语言的split操作
fun main() {
    val jsonText = "Derry,Leo,Lance,Alvin"
    // list 自动类型推断成 list == List<String>
    val list = jsonText.split(",")

    // 直接输出 list 集合，不解构
    println("分割后的list里面的元素有:$list")

    // C++ 解构  Kt也有解构
    val (v1, v2, v3, v4) = list
    println("解构四个只读变量的值是:v1:$v1, v2:$v2, v3:$v3, v4:$v4")
}
```
---
### 45.Kotlin语言的replace完成加密解码操作
```kotlin

// TODO 45.Kotlin语言的replace完成加密解码操作
// ABCDEFGHIJKLMNOPQRSTUVWXYZ
fun main() {
   val sourcePwd = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    println("原始密码是:$sourcePwd")

    // 加密操作:就是把字符替换成数字 打乱了，就属于加密了
    val newPwd = sourcePwd.replace(Regex("[AKMNO]")) {
        it.value // 完全没有做任何事情

        when(it.value) { // 这里的每一个字符 A B C D ...
            "A" -> "9"
            "K" -> "3"
            "M" -> "5"
            "N" -> "1"
            "O" -> "4"
            else -> it.value // 就啥事不做，直接返回 字符本身 A B C D ...
        }
    }
    println("加密后的密码是:$newPwd")

    // 解密操作
    val sourcePwdNew = newPwd.replace(Regex("[9514]")) {
        when(it.value) {
            "9" -> "A"
            "3" -> "K"
            "5" -> "M"
            "1" -> "N"
            "4" -> "O"
            else -> it.value // 就啥事不做，直接返回 字符本身 A B C D ...
        }
    }
    println("解密后的密码是:$sourcePwdNew")
}
```
---
### 46.Kotlin语言的==与===比较操作
```kotlin
// TODO 46.Kotlin语言的==与===比较操作
fun main() {
    // == 值 内容的比较  相当于Java的equals
    // === 引用的比较

    val name1 : String = "Derry"
    val name2 : String = "Derry"
    val name3 = "ww"

    // 小结：name1.equals(name2)  等价于 name1 == name2  都是属于 值 内容的比较
    println(name1.equals(name2)) // java
    println(name1 == name2) // kt

    // 引用的比较
    println(name1 === name2) // true
    println(name1 === name3) // false

    // 引用的比较 难度高一点点
    val name4 = "derry".capitalize() // 修改成"Derry"
    println(name4)
    println(name4 === name1)
}
```

---
### 47.Kotlin语言的字符串遍历操作
```kotlin
// TODO 47.Kotlin语言的字符串遍历操作
// "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
fun main() {
    val str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    str.forEach {  c -> // 覆盖默认的it参数名，修改参数名为 c
        // it == str的每一个字符 A B C D ...
        // print("所有的字符是:$it  ")
        print("所有的字符是:$c  ")
    }
}
```
---
### 48.Kotlin语言中数字类型的安全转换函数
```kotlin

// TODO 48.Kotlin语言中数字类型的安全转换函数
fun main() {
    val number: Int = "666".toInt()
    println(number)

    // 字符串里面放入了Double类型，无法转换成Int，会奔溃
    // val number2: Int = "666.6".toInt()
    // println(number2)

    // 解决什么奔溃的问题
    val number2: Int? = "666.6".toIntOrNull()
    println(number2)

    val number3: Int? = "888".toIntOrNull()
    println(number3)

    val number4: Int? = "888.8".toIntOrNull()
    println(number4 ?: "原来你是null啊")

    // 小结：以后字符串有整形相关的转换，尽量用 toIntOrNull 此函数
}
```
---
### 49.Kotlin语言中Double转Int与类型格式化
```kotlin
import kotlin.math.roundToInt

// TODO 49.Kotlin语言中Double转Int与类型格式化
// 65.4645654 65.8343433
fun main() {
    println(65.4645654.toInt()) // 65 四舍五入

    println(65.4645654.roundToInt())  // 65 四舍五入

    println(65.8343433.roundToInt()) // 66 四舍五入

    // 结论：用 roundToInt()函数，保证 Double ->转Int 持有四舍五入的效果

    // r的类型： String
    val r  = "%.3f".format(65.8343433)
    println(r)

}
```
---
### 50.Kotlin语言的apply内置函数
```kotlin

import java.io.File

// TODO 50.Kotlin语言的apply内置函数
fun main() {
    val info = "Derry You Hao"

    // 普通的方式
    println("info字符串的长度是:${info.length}")
    println("info最后一个字符是:${info[info.length -1]}")
    println("info全部转成小写是:${info.toLowerCase()}")

    println()

    // apply内置函数的方式
    // info.apply特点：apply函数始终是返回 info本身 String类型
    val infoNew : String = info.apply {
        // 一般大部分情况下，匿名函数，都会持有一个it，但是apply函数不会持有it，却会持有当前this == info本身
        println("apply匿名函数里面打印的:$this")

        println("info字符串的长度是:${length}")
        println("info最后一个字符是:${this[length -1]}")
        println("info全部转成小写是:${toLowerCase()}")
    }
    println("apply返回的值:$infoNew")

    println()

    // 真正使用apply函数的写法规则如下：
    // info.apply特点：apply函数始终是返回 “info本身”，所以可以链式调用
    info.apply {
        println("长度是:$length")
    }.apply {
        println("最后一个字符是:${this[length -1]}")
        true
        true
        true
    }.apply {
        println("全部转成小写是:${toLowerCase()}")
    }

    println()

    // 普通写法
    val file = File("D:\\a.txt")
    file.setExecutable(true)
    file.setReadable(true)
    println(file.readLines())

    println()

    // apply写法
    // 匿名函数里面 持有的this == file本身
    /*val fileNew: File =*/ file.apply {
        setExecutable(true)
    }.apply {
        setReadable(true)
    }.apply {
        println(file.readLines())
    }
}
```

---
### 51.Kotlin语言的let内置函数
```kotlin

// TODO 51.Kotlin语言的let内置函数
// 普通方式 对集合第一个元素相加
// let方式 对集合第一个元素相加
// 普通方式 对值判null，并返回
// let方式 对值判null，并返回
fun main() {
    // 普通方式 对集合第一个元素相加
    val list = listOf(6, 5, 2, 3, 5, 7)
    val value1 = list.first() // 第一个元素
    val result1 = value1 + value1
    println(result1)

    // let方式 对集合第一个元素相加
    val result2 = listOf(6, 5, 2, 3, 5, 7).let {
        // it == list集合
        it.first() + it.first() // 匿名函数的最后一行，作为返回值，let的特点，   但是前面学的apply永远是返回info本身
        /*true
        true
        true*/
    }
    println(result2)

    println()

    // 普通方式 对值判null，并返回
    println(getMethod1(/*null*/ "Derry"))

    // let方式 + 空合并操作符 对值判null，并返回
    println(getMethod3(/*null*/ "Derry"))
}

// 普通方式 对值判null，并返回
fun getMethod1(value: String?) : String {
    return if (value == null) "你传递的内容是null，你在搞什么飞机" else "欢迎回来${value}非常欢迎"
}
// 普通方式 简化版本
fun getMethod2(value: String?) = if (value == null) "你传递的内容是null，你在搞什么飞机" else "欢迎回来${value}非常欢迎"

// let方式 + 空合并操作符 对值判null，并返回
fun getMethod3(value: String?) : String {
    return value?.let {
        "欢迎回来${it}非常欢迎"
    } ?: "你传递的内容是null，你在搞什么飞机"
}

// let方式 + 空合并操作符 对值判null，并返回 简化版本
fun getMethod4(value: String?) =
     value?.let {
        "欢迎回来${it}非常欢迎"
    } ?: "你传递的内容是null，你在搞什么飞机"


```
---
### 52.Kotlin语言的run内置函数
```kotlin

// TODO 52.Kotlin语言的run内置函数
// 1.run函数的特点 字符串延时
// 2.具名函数判断长度 isLong
// 3.具名函数监测合格 showText
// 4.具名函数增加一个括号 mapText
// 5.具名函数输出内容
fun main() {
    val str = "Derry is OK"
    val r1 : Float = str.run {
        // this == str本身
        true
        5435.5f
    }
    println(r1)

    // 下面是 具名函数 配合 run函数

    // 2.具名函数判断长度 isLong

    // 这个是属于 匿名函数 配合 run
    str.run {
        // this == str本身
    }

    // 这个是属于具名函数
    // str.run(具名函数)
    str
        .run(::isLong) // this == str本身
        .run(::showText) // this == isLong返回的boolean值
        .run(::mapText)
        .run(::println)

    println()

    // let函数持有it，run函数持有this 都可以很灵活的，把上一个结果值 自动给 下一个函数
    str.let(::isLong) // it == str本身
    .let(::showText) // it == isLong返回的boolean值
    .let(::mapText) // it == str本身
    .let(::println) // it == str本身

    println()

    // >>>>>>>>>>>>>>>>>>>>>> 上面全部都是具名函数调用给run执行  下面全部是 匿名函数调用给run执行
    str
        .run {
            if (length > 5) true else false
        }
        .run {
            if (this) "你的字符串合格" else "你的字符串不合格"
        }
        .run {
            "【$this】"
        }
        .run {
            println(this)
        }
}

fun isLong(str: String) /* : Boolean */ = if (str.length > 5) true else false

fun showText(isLong: Boolean) /*: String */ = if (isLong) "你的字符串合格" else "你的字符串不合格"

fun mapText(getShow: String) /*: String */ = "【$getShow】"
```
---
### 53.Kotlin语言的with内置函数
```kotlin

// TODO 53.Kotlin语言的with内置函数
fun main() {
    val str = "李元霸"

    // 具名操作
    /*with(str) {
        this == str本身
    }*/
    val r1 = with(str, ::getStrLen)
    val r2 = with(r1, ::getLenInfo)
    val r3 = with(r2, ::getInfoMap)
    with(r3, ::show)

    println()

    // 匿名操作
    with(with(with(with(str) {
        length
    }) {
        "你的字符串长度是:$this"
    }){
        "【$this】"
    }){
        println(this)
    }
}

fun getStrLen(str: String) = str.length
fun getLenInfo(len: Int) = "你的字符串长度是:$len"
fun getInfoMap(info: String) = "【$info】"
fun show(content: String) = println(content)

```
---
### 54.Kotlin语言的also内置函数
```kotlin

import java.io.File

// TODO 54.Kotlin语言的also内置函数
// "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
fun main() {
    val str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"

    val r1 : String = str.also {
        true
        354543.4f
        454
        'C'
    }

    val r2 : Int = 123.also {
        true
        354543.4f
        454
        'C'
        false
    }

    str.also {
        // it == str本身
    }

    // 真正使用also函数的写法规则如下：
    // str.also特点：also函数始终是返回 “str本身”，所以可以链式调用
    str.also {
        println("str的原始数据是:$it")
    }.also {
        println("str转换小写的效果是:${it.toLowerCase()}")
    }.also {
        println("结束了")
    }

    val file = File("D:\\a.txt")


    // 匿名函数里面做的事情，和sourceFile无关，因为永远都是返回 file本身
    val sourceFile = file.also {
        file.setReadable(true)
        file.setWritable(true)
        println(file.readLines())
        // 假设 做了很多很多的事情
        // ...
    }.also {
        file.setReadable(true)
        println(file.readLines())
        // 假设 做了很多很多的事情
        // ...
    }.also {
        file.setReadable(true)
        println(file.readLines())
        // 假设 做了很多很多的事情
        // ...
    }
    // sourceFile没有任何影响
}
```
---
###  55.Kotlin语言的takeIf内置函数
```kotlin

// TODO 55.Kotlin语言的takeIf内置函数
// "欢迎登录系统,拥有超级权限"
fun main() {
    val result = checkPermissionAction("Root2", "!@#$")
    // println("欢迎${result}尊贵的用户欢迎登录系统,拥有超级权限")
    if (result != null) {
        println("欢迎${result}尊贵的用户欢迎登录系统,拥有超级权限")
    } else {
        println("你的权限不够")
    }

    // name.takeIf { true/false }
    // true: 直接返回name本身
    // false: 直接放回null

    // 真正的用途
    println(checkPermissionAction2("Root", "!@#$"))

    // 小结：一般大部分情况下，都是 takeIf + 空合并操作符 = 一起使用
}

// 前端
public fun checkPermissionAction(name: String, pwd: String) : String? {
    return name.takeIf { permissionSystem(name, pwd) }
}

// takeIf + 空合并操作符
public fun checkPermissionAction2(name: String, pwd: String) : String {
    return name.takeIf { permissionSystem(name, pwd) } ?: "你的权限不够"
}

// 权限系统
private fun permissionSystem(username: String, userpwd: String) : Boolean {
    return if (username == "Root" && userpwd == "!@#$") true  else false
}
```
---
###  56.Kotlin语言的takeUnless内置函数
```kotlin
// TODO 56.Kotlin语言的takeUnless内置函数
// takeIf 和 takeUnless 功能是相反的
// name.takeIf { true/false }  true:返回name本身，false返回null
// name.takeUnless { true/false }  false:返回name本身，true返回null

// 为什么有 takeUnless 的出现，一个 takeIf 不就够了吗？

class Manager {

    private var infoValue: String? = null

    fun getInfoValue() /* : String? */ = infoValue

    fun setInfoValue(infoValue: String) {
        this.infoValue = infoValue
    }
}

fun main() {
    val manager = Manager()

    /*
    "Derry".takeIf { *//*it == "Derry"*//* }
    "Derry".takeUnless { *//*it == "Derry"*//* }
    */

    // manager.setInfoValue("AAA")

    // 小结：takeUnless+it.isNullOrBlank() 一起使用，可以验证字符串有没有初始化等功能
    val r  = manager.getInfoValue().takeUnless { it.isNullOrBlank() } ?: "未经过任何初始化值"
    println(r)
}
```
---

