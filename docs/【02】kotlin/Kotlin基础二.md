
---
### 23.Kotlin语言的匿名函数学习
```kotlin
// TODO 23.Kotlin语言的匿名函数学习
fun main() {
    val len = "Derry".count()
    println(len)

    val len2 = "Derry".count {
        // it 等价于 D e r r y 的字符 Char
        it == 'r'
    }
    println(len2)
}

```
---
### 24.Kotlin语言的函数类型&隐式返回学习
```kotlin
// TODO 24.Kotlin语言的函数类型&隐式返回学习
fun main() {
    // 我们现在在写函数

    // 第一步：函数输入输出的声明
    val methodAction : () -> String

    // 第二步：对上面函数的实现
    methodAction = {
        val inputValue = 999999
        "$inputValue Derry" // == 背后隐式 return "$inputValue Derry";
        // 匿名函数不要写return，最后一行就是返回值
    }

    // 第三步：调用此函数
    println(methodAction())
}

/*
fun methodAction() : String {
    return "Derry"
}
 */
```
---
### 25.Kotlin语言的函数参数学习
```kotlin

// TODO 25.Kotlin语言的函数参数学习
fun main() {

    // 我们现在在写函数

    // 第一步：函数输入输出的声明   第二步：对声明函数的实现
    val methodAction : (Int, Int, Int) -> String = { number1, number2, number3 ->
        val inputValue = 999999
        "$inputValue Derry 参数一:$number1, 参数二:$number2, 参数三:$number3"
    }
    // 第三步：调用此函数
    println(methodAction(1, 2, 3))
}

/*
fun methodAction(number1: Int, number2: Int, number3: Int) : String {
    val inputValue = 999999
    return "$inputValue Derry 参数一:$number1, 参数二:$number2, 参数三:$number3"
}
 */
```
---
### 26.Kotlin语言的it关键字特点
```kotlin
// TODO 26.Kotlin语言的it关键字特点
fun main() {
    val methodAction : (Int, Int, Int) -> String = { n1, n2, n3 ->
        val number = 24364
        println("$number Derry ,n1:$n1, n2:$n2, n3:$n3")
        "$number Derry ,n1:$n1, n2:$n2, n3:$n3"
    }
    // methodAction.invoke(1,2,3)
    methodAction(1,2,3)

    val methodAction2 : (String) -> String = { "$it Derry" }
    println(methodAction2("DDD"))

    val methodAction3 : (Double) -> String = { "$it Derry2" }
    println(methodAction3(5454.5))
}

/*
    fun methodAction2(it : String) : String { return "$it Derry" }
 */
```

---
### 27.Kotlin语言的匿名函数的类型推断
```kotlin
// TODO 27.Kotlin语言的匿名函数的类型推断
fun main() {

    // 匿名函数，类型推断为String
    // 方法名 : 必须指定 参数类型 和 返回类型
    // 方法名 = 类型推断返回类型
    val method1 = { v1:Double, v2:Float, v3:Int ->
       "v1:$v1, v2:$v2, v3:$v3"
    } // method1 函数： (Double, Float, Int) -> String
    println(method1(454.5, 354.3f, 99))

    val method2 = {
        3453.3f
    } // method2 函数： () -> Unit
    println(method2())

    val method3 = { number: Int ->
        number
    } // method3 函数： (Int) -> Int
    println(method3(9))
}
```
---
### 28.Kotlin语言的lambda学习
```kotlin

// TODO 28.Kotlin语言的lambda学习
fun main() {

    // 匿名函数 == lambda表达式
    val addResultMethod = { number1 : Int, number2: Int ->
        "两数相加的结果是:${number1 + number2}"
    } // addResultMethod 函数： (Int, Int) -> String
    println(addResultMethod(1, 1))

    // 匿名函数 入参 Int,          返回 Any类型
    // lambda表达式的参数 Int,    lambda表达式的结果Any类型
    val weekResultMethod = { number: Int ->
        when(number) {
            1 -> "星期1"
            2 -> "星期2"
            3 -> "星期3"
            4 -> "星期4"
            5 -> "星期5"
            else -> -1
        }
    } // weekResultMethod 函数： (Int) -> Any
    println(weekResultMethod(2))

    // 匿名函数 属于 lambda
}
```
---
### 29.模拟：数据库SQLServer
```java

interface ResponseResult {
    void result(String msg, int code);
}

public class KTBase29 {

    // 模拟：数据库SQLServer
    public final static String USER_NAME_SAVE_DB = "Derry";
    public final static String USER_PWD_SAVE_DB = "123456";

    public static void main(String[] args) {
        loginAPI("Derry2", "123456", new ResponseResult() {

            @Override
            public void result(String msg, int code) {
                System.out.println(String.format("最终登录的情况如下: msg:%s, code:%d", msg, code));
            }
        });
    }

    // 登录API 模仿 前端
    public static void loginAPI(String username, String userpwd, ResponseResult responseResult) {
        if (username == null || userpwd == null) {
            // TODO("用户名或密码为null") // 出现问题，终止程序
        }

        // 做很多的校验 前端校验
        if (username.length() > 3 && userpwd.length() > 3) {
            if (wbeServiceLoginAPI(username, userpwd)) {
                // 登录成功
                // 做很多的事情 校验成功信息等
                // ...
                responseResult.result("login success", 200);
            } else {
                // 登录失败
                // 做很多的事情 登录失败的逻辑处理
                // ...
                responseResult.result("login error", 444);
            }
        } else {
            // TODO("用户名和密码不合格") // 出现问题，终止程序
        }
    }

    // 登录的API暴露者 服务器
    private static boolean wbeServiceLoginAPI(String name, String pwd) {
        // kt的if是表达式(很灵活)     java的if是语句(有局限性)

        // 做很多的事情 登录逻辑处理
        // ...

        if (name == USER_NAME_SAVE_DB && pwd == USER_PWD_SAVE_DB)
            return true;
        else
            return false;
    }

}
```
---
###  29.在函数中定义参数是函数的函数
```kotlin

// TODO 29.在函数中定义参数是函数的函数
fun main() {
    loginAPI("Derry", "123456") { msg: String, code: Int ->
        println("最终登录的情况如下: msg:$msg, code:$code")
    }
}

// 模拟：数据库SQLServer
const val USER_NAME_SAVE_DB = "Derry"
const val USER_PWD_SAVE_DB = "123456"

// 登录API 模仿 前端
public fun loginAPI(username: String, userpwd: String, responseResult: (String, Int) -> Unit) {
    if (username == null || userpwd == null) {
        TODO("用户名或密码为null") // 出现问题，终止程序
    }

    // 做很多的校验 前端校验
    if (username.length > 3 && userpwd.length > 3) {
        if (wbeServiceLoginAPI(username, userpwd)) {
            // 登录成功
            // 做很多的事情 校验成功信息等
            // ...
            responseResult("login success", 200)
        } else {
            // 登录失败
            // 做很多的事情 登录失败的逻辑处理
            // ...
            responseResult("login error", 444)
        }
    } else {
        TODO("用户名和密码不合格") // 出现问题，终止程序
    }
}

// 登录的API暴露者 服务器
private fun wbeServiceLoginAPI(name: String, pwd: String) : Boolean {
    // kt的if是表达式(很灵活)     java的if是语句(有局限性)

    // 做很多的事情 登录逻辑处理
    // ...

    return if (name == USER_NAME_SAVE_DB && pwd == USER_PWD_SAVE_DB) true else false
}
```
---
### 30.Kotlin语言的简略写法学习
```kotlin

// TODO 30.Kotlin语言的简略写法学习
fun main() {
    // 第一种方式
    loginAPI2("Derry", "123456", { msg: String, code:Int ->
        println("最终登录的情况如下: msg:$msg, code:$code")
    })

    // 第二种方式
    loginAPI2("Derry", "123456", responseResult = { msg: String, code: Int ->
        println("最终登录的情况如下: msg:$msg, code:$code")
    })

    // 第三种方式
    loginAPI2("Derry", "123456") { msg: String, code: Int ->
        println("最终登录的情况如下: msg:$msg, code:$code")
    }
}

// 模拟：数据库SQLServer
const val USER_NAME_SAVE_DB2 = "Derry"
const val USER_PWD_SAVE_DB2 = "123456"

// 登录API 模仿 前端
public fun loginAPI2(username: String, userpwd: String, responseResult: (String, Int) -> Unit) {
    if (username == null || userpwd == null) {
        TODO("用户名或密码为null") // 出现问题，终止程序
    }

    // 做很多的校验 前端校验
    if (username.length > 3 && userpwd.length > 3) {
        if (wbeServiceLoginAPI2(username, userpwd)) {
            // 登录成功
            // 做很多的事情 校验成功信息等
            // ...
            responseResult("login success", 200)
        } else {
            // 登录失败
            // 做很多的事情 登录失败的逻辑处理
            // ...
            responseResult("login error", 444)
        }
    } else {
        TODO("用户名和密码不合格") // 出现问题，终止程序
    }
}

// 登录的API暴露者 服务器
private fun wbeServiceLoginAPI2(name: String, pwd: String) : Boolean {
    // kt的if是表达式(很灵活)     java的if是语句(有局限性)

    // 做很多的事情 登录逻辑处理
    // ...

    return if (name == USER_NAME_SAVE_DB && pwd == USER_PWD_SAVE_DB) true else false
}
```
---
### 31.Kotlin语言的函数内联学习
```kotlin
// TODO 31.Kotlin语言的函数内联学习
fun main() {
    loginAPI3("Derry", "123456") { msg: String, code: Int ->
        println("最终登录的情况如下: msg:$msg, code:$code")
    }
}

// 模拟：数据库SQLServer
const val USER_NAME_SAVE_DB3 = "Derry"
const val USER_PWD_SAVE_DB3 = "123456"

// 此函数有使用lambda作为参数，就需要声明成内联
// 如果此函数，不使用内联，在调用端，会生成多个对象来完成lambda的调用（会造成性能损耗）
// 如果此函数，使用内联，相当于 C++ #define 宏定义 宏替换，会把代码替换到调用处，调用处 没有任何函数开辟 对象开辟 的损耗
// 小结：如果函数参数有lambda，尽量使用 inline关键帧，这样内部会做优化，减少 函数开辟 对象开辟 的损耗

// 登录API 模仿 前端
public inline fun loginAPI3(username: String, userpwd: String, responseResult: (String, Int) -> Unit) {
    if (username == null || userpwd == null) {
        TODO("用户名或密码为null") // 出现问题，终止程序
    }

    // 做很多的校验 前端校验
    if (username.length > 3 && userpwd.length > 3) {
        if (wbeServiceLoginAPI3(username, userpwd)) {
            // 登录成功
            // 做很多的事情 校验成功信息等
            // ...
            responseResult("login success", 200)
        } else {
            // 登录失败
            // 做很多的事情 登录失败的逻辑处理
            // ...
            responseResult("login error", 444)
        }
    } else {
        TODO("用户名和密码不合格") // 出现问题，终止程序
    }
}

// 此函数没有使用lambda作为参数，就不需要声明成内联
// 登录的API暴露者 服务器
fun wbeServiceLoginAPI3(name: String, pwd: String) : Boolean {
    // kt的if是表达式(很灵活)     java的if是语句(有局限性)

    // 做很多的事情 登录逻辑处理
    // ...

    return name == USER_NAME_SAVE_DB && pwd == USER_PWD_SAVE_DB
}
```
---
### 32.Kotlin语言的函数引用学习
```kotlin

// TODO 32.Kotlin语言的函数引用学习
fun main() {
    // 函数引用
    // lambda属于函数类型的对象，需要把methodResponseResult普通函数变成 函数类型的对象（函数引用）

//     login("Derry2", "123456", ::methodResponseResult)

    val obj = ::methodResponseResult
    val obj2 = obj
    val obj3 =  obj2

    login("Derry", "123456", obj3)
}

fun methodResponseResult(msg: String, code: Int) {
    println("最终登录的成果是:msg:$msg, code:$code")
}


// 模拟：数据库SQLServer
const val USER_NAME_SAVE_DB4 = "Derry"
const val USER_PWD_SAVE_DB4= "123456"

inline fun login(name: String, pwd: String, responseResult: (String, Int) -> Unit) {
    if (USER_NAME_SAVE_DB4 == name && USER_PWD_SAVE_DB4 == pwd) {
        // 登录成功
        // 做很多的事情 校验成功信息等
        responseResult("登录成功", 200)
        // ...
    } else {
        // 登录失败
        // 做很多的事情 登录失败的逻辑处理
        // ...
        responseResult("登录失败错了", 444)
    }
}
```
---
###  33.Kotlin语言的函数类型作为返回类型
```kotlin

// TODO 33.Kotlin语言的函数类型作为返回类型
fun main() {
    val r = show("学习KT语言")
    // r 是show函数的 返回值

    val niming_showMethod = showMethod("show")
    // niming_showMethod 是 showMethod函数的返回值 只不过这个返回值 是一个 函数

    // niming_showMethod == 匿名函数
    println(niming_showMethod("Derry", 33))
}

fun show(info: String): Boolean {
    println("我是show函数 info:$info")
    return true
}

fun show2(info: String): String {
    println("我是show函数 info:$info")
    return "DDD"
}

fun show3(info: String): String {
    println("我是show函数 info:$info")
    return /*888*/ ""
}

// showMethod函数 再返回一个 匿名函数
fun showMethod(info: String): (String, Int) -> String {
    println("我是show函数 info:$info")

    // return 一个函数 匿名函数
    return { name: String, age: Int ->
        "我就是匿名函数：我的name:$name, age:$age"
    }
}
```

---
###  Kotlin语言的匿名函数与具名函数
```java

interface IShowResult { // 接口的折中方案 解决 kt的lambda问题
    void result(String result);
}

// TODO 34.Kotlin语言的匿名函数与具名函数
public class KtBase34 {

    public static void main(String[] args) { // psv
        // 匿名函数 - 匿名接口实现
        showPersonInfo("lisi", 99, 'm', "study cpp", new IShowResult() {
            @Override
            public void result(String result) {
                System.out.println("显示结果:" + result);
            }
        });

        // 具名函数 - 具名接口实现 showResultImpl
        IShowResult showResultImpl = new MshowResultImpl();
        showPersonInfo("wangwu", 88, 'n', "study kt", showResultImpl);
    }

   static class MshowResultImpl implements IShowResult {

        @Override
        public void result(String result) {
            System.out.println("显示结果:" + result);
        }
    }

    static void showPersonInfo(String name, int age, char sex, String study, IShowResult iShowResult) {
        String str = String.format("name:%s, age:%d, sex:%c, study:%s", name, age, sex, study);
        iShowResult.result(str);
    }

}

```
---
###  34.Kotlin语言的匿名函数与具名函数
```kotlin

// TODO 34.Kotlin语言的匿名函数与具名函数
fun main() {

    // 匿名函数
    showPersonInfo("lisi", 99, '男', "学习KT语言") {
        println("显示结果:$it")
    }

    // 具名函数 showResultImpl
    showPersonInfo("wangwu", 89, '女', "学习C++语言", ::showResultImpl)

}

fun showResultImpl(result: String) {
    println("显示结果:$result")
}

inline fun showPersonInfo(name: String, age: Int, sex: Char, study: String, showResult: (String) -> Unit) {
    val str = "name:$name, age:$age, sex:$sex, study:$study"
    showResult(str)
}
```
