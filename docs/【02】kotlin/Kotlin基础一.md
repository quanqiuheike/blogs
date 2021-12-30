### 07.Kotlin语言声明变量与内置数据类型
```kotlin
// TODO 07.Kotlin语言声明变量与内置数据类型
fun main() {

    println("Hello World")

    // TODO ================== 声明变量

    /*
       可读可改  变量名    类型      值
       var      name  : String = "Derry"
     */
    var name : String = "Derry"
    // name = "Lance"
    println(name)

    // TODO ================= 内置数据类型
    /*
        String      字符串
        Char        单字符
        Boolean     true/false
        Int         整形
        Double      小数
        List        集合
        Set         无重复的元素集合
        Map         键值对的集合

        Int   ---> java int
        Float  ---> java float
     */
```
---
### 08.Kotlin语言的只读变量
```kotlin
// TODO 08.Kotlin语言的只读变量
fun main() {

    // var 默认提示：变量永远不会被修改，建议修改成 val == 不可改变的(只读)
    val info : String = "MhyInfo"
    // info = "AA" // 不能修改
    println(info)

    /*
      只读     变量名   类型   值
      val     age   : Int = 99
     */
    val age: Int = 99
    // age = 99 // 不能修改
    println(age)
}
```
---
### 09.Kotlin语言的类型推断
```kotlin
// TODO 09.Kotlin语言的类型推断
fun main() {
    // 提示：Explicitly given type is redundant here
    //      显示给定的类型在这里是多余的
    val info : String = "Derry is Success"
    println(info)

    val age = 98
    println(age)

    val xxx = 'A'
}
```
---
### 10.Kotlin语言的编译时常量
```kotlin
const val PI = 3.1415 // 定义编译时常量

// TODO 10.Kotlin语言的编译时常量
// 编译时常量只能是常用的基本数据类型：（String，Double，Int，Float，Long，Short，Byte，Char，Boolean）
// 编译时常量只能在函数之外定义，为什么？答：如果在函数之内定义，就必须在运行时才能调用函数赋值，何来编译时常量一说
// 结论：编译时常量只能在函数之外定义，就可以在编译期间初始了

fun main() {
    val info = "Derry info" // 这个称为 只读类型的变量

    // 提示：修饰符const不适用于 局部变量
    // const val PI = 45

    // const val PI2 = 3.1415 // 定义编译时常量
}
```
---
### 11.学习查看Kotlin反编译后字节码
```kotlin
const val PI1 = 3.1415
const val PI2 = 3.1415
const val age = 99

// TODO 11.学习查看Kotlin反编译后字节码
fun main() {
    val name = "Derry"
    val number = 108
    val number2 = 200
    val isOk = false

    val s = 'A'
}
```
---
### 12.Kotlin语言的引用类型学习
```kotlin
// TODO 12.Kotlin语言的引用类型学习
// Java语言有两种数据类型：
//      第一种：基本类型：int double 等
//      第二种：引用类型 String 等

// Kotlin语言只有一种数据类型：
//      看起来都是引用类型，实际上编译器会在Java字节码中，修改成 “基本类型”
fun main() {
    val age: Int = 99
    val pi: Float = 3.1415f
    val pi2: Double = 3.1415
    val isOk: Boolean = true
}
```
---
### 13.Kotlin语言的range表达式
```kotlin
// TODO 13.Kotlin语言的range表达式
fun main() {
    val number = 148

    // range 范围 从哪里 到哪里

    if (number in 10..59) {
        println("不及格")
    } else if (number in 0..9) {
        println("不及格并且分数很差")
    } else if (number in 60..100) {
        println("合格")
    } else if (number !in 0..100) {
        println("分数不在0到100范围内")
    }
}
```
---
### 14.Kotlin语言的when表达式
```kotlin

// TODO 14.Kotlin语言的when表达式
fun main() {
    val week = 0

    // Java的 if 语句
    // KT的 if 是表达式 有返回值的

    val info = when(week) {
        1 -> "今天是星期一，非常忙碌的一天开会"
        2 -> "今天是星期二，非常辛苦的写需求"
        3 -> "今天是星期三，努力写Bug中"
        4 -> "今天是星期四，发布版本到凌晨"
        5 -> "今天是星期五，淡定喝茶，一个Bug改一天"
        6 -> "今天是星期六，稍微加加班"
        7 -> "今天是星期七，看剧中，游玩中"
        else -> {
            println("养猪去了，忽略星期几")
        }
    }
    println(info) // void 关键字 无返回    用Unit类代替了java的void关键字
}
```
---
### 15.Kotlin语言的String模版
```kotlin
// TODO 15.Kotlin语言的String模版
fun main() {
    val garden = "黄石公园"
    val time = 6

    println("今天天气很晴朗，去玩" + garden + "，玩了" +time +" 小时") // Java的写法
    println("今天天气很晴朗，去${garden}玩，玩了$time 小时") // 字符串模版的写法

    // KT的if是表达式，所以可以更灵活，  Java的if是语句，还有局限性
    val isLogin = false
    println("server response result: ${if (isLogin) "恭喜你，登录成功√" else "不恭喜，你登录失败了，请检查Request信息"}")
}
```
---
### 16.Kotlin语言的函数头学习
```kotlin
// TODO 16.Kotlin语言的函数头学习
fun main() {
    method01(99, "lisi")
}

// 函数默认都是public
// 其实Kotlin的函数，更规范，先有输入，再有输出
private fun method01(age: Int, name: String) : Int {
    println("你的姓名是:$name,你的年龄是:$age")

    return 200
}

/* 上面的Kt函数，背后会变成下面的Java代码：
   private static final int method01(int age, String name) {
       String var2 = "你的姓名是:" + name + ",你的年龄是:" + age;
       System.out.println(var2);
       return 200;
   }
 */

/*
   private static final int method01(int age, String name) {
      String var2 = "你的姓名是:" + name + ",你的年龄是:" + age;
      boolean var3 = false;
      System.out.println(var2);
      return 200;
   }
 */
```
---
### 17.Kotlin中函数参数的默认参数
```kotlin
// TODO 17.Kotlin中函数参数的默认参数
fun main() {
    action01("lisi", 89)
    action02("wangwu")
    action03()

    action03("赵六", 76)
}

private fun action01(name: String, age: Int) {
    println("我的姓名是:$name, 我的年龄是:$age")
}

private fun action02(name: String, age: Int = 77) {
    println("我的姓名是:$name, 我的年龄是:$age")
}

private fun action03(name: String = "王五", age: Int = 67) {
    println("我的姓名是:$name, 我的年龄是:$age")
}
```
---
### 18.Kotlin语言的具名函数参数
```kotlin
// TODO 18.Kotlin语言的具名函数参数
fun main() {
    loginAction(age = 99, userpwd = "123", usernam = "de", username = "Derry", phonenumber = "123456")
}

private fun loginAction(username: String, userpwd: String, phonenumber: String, age: Int, usernam: String) {
    println("username:$username, userpwd:$userpwd, phonenumber:$phonenumber, age:$age")
}
```
---
### 19.Kotlin语言的Unit函数特点
```kotlin
// TODO 19.Kotlin语言的Unit函数特点
fun main() {

}

// Java语言的void关键字(void是 无参数返回的 忽略类型) 但是他是关键帧啊，不是类型，这很矛盾
// : Unit不写，默认也有，Unit代表  无参数返回的 忽略类型 == Unit类型类
private fun doWork() : Unit {
    return println()
}

private fun doWork2() {
    return println()
}

private fun doWork3() /*: Unit*/ {
    println()
}
```
---
### 20.Kotlin语言的Nothing类型特点
```kotlin

// TODO 20.Kotlin语言的Nothing类型特点
fun main() {
    show(99)
}

private fun show(number: Int) {
    when(number) {
        -1 -> TODO("没有这种分数")
        in 0..59 -> println("分数不及格")
        in 60..70 -> println("分数及格")
        in 71..100 -> println("分数优秀")
    }
}

interface A {
    fun show()
}

class AImpl : A {
    override fun show() {
        // 下面这句话，不是注释提示，会终止程序的
        TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
    }

}
```
---

### 21.在java里面就是普通函数
```java
public class KtBase21 {

    // in  is  在java里面就是普通函数

    public  static final void in() {
        System.out.println("in run success");
    }

    public  static final void is() {
        System.out.println("is run success");
    }
}

```
---
### 21.Kotlin语言的反引号中函数名特点
```kotlin
// TODO 21.Kotlin语言的反引号中函数名特点
fun main() {
    // 第一种情况：
    `登录功能 2021年8月8日测试环境下 测试登录功能 需求编码人是Derry`("Derry", "123456")

    // 第二种情况：// in  is  在kt里面就是关键字，怎么办呢？ 使用反引号
    KtBase21.`is`()
    KtBase21.`in`()


    // 第三种情况： 很少发生
    `65465655475`()
}

private fun `登录功能 2021年8月8日测试环境下 测试登录功能 需求编码人是Derry`(name: String, pwd: String) {
    println("模拟：用户名是$name, 密码是:$pwd")
}

private fun `65465655475`() {
    // 写了很复杂的功能，核心功能
    // ...
}

// 公司加密私有的文档     65465655475  === 此函数的作用 xxxx
// 公司加密私有的文档     55576575757  === 此函数的作用 xxxx
```
