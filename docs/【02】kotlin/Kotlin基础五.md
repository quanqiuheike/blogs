
---
###  83.Kotlin语言的继承与重载的open关键字学习
```kotlin

// KT所有的类，默认是final修饰的，不能被继承，和Java相反
// open：移除final修饰
open class Person(private val name: String) {

    private fun showName() = "父类 的姓名是【$name】"

    // KT所有的函数，默认是final修饰的，不能被重写，和Java相反
    open fun myPrintln() = println(showName())
}

class Student(private val subName: String) : Person(subName) {

    private fun showName() = "子类 的姓名是【${subName}】"

    override fun myPrintln() = println(showName())

}

// TODO 83.Kotlin语言的继承与重载的open关键字学习
// 1.父类 val name  showName()->String  myPrintln->Unit
// 2.子类 myPrintln->Unit
fun main() {
    val person: Person = Student("张三")
    person.myPrintln()
}
```
---
###  84.Kotlin语言的类型转换学习
```kotlin

import java.io.File

open class Person2(private val name: String) {

    fun showName() = "父类 的姓名是【$name】"

    // KT所有的函数，默认是final修饰的，不能被重写，和Java相反
    open fun myPrintln() = println(showName())
}

class Student2(private val subName: String) : Person2(subName) {

    fun showName2() = "子类 的姓名是【${subName}】"

    override fun myPrintln() = println(showName2())

}

// TODO 84.Kotlin语言的类型转换学习
// 1.普通运行子类输出
// 2.is Person Student File
// 3.is + as 转换
fun main() {
    val p: Person2 = Student2("王五")
    p.myPrintln()

    println(p is Person2)
    println(p is Student2)
    println(p is File)

    // is + as = 一般是配合一起使用
    if (p is Student2) {
        (p as Student2).myPrintln()
    }

    if (p is Person2) {
        // (p as Person2).myPrintln() // 因为子类重写了父类
        println((p as Person2).showName())
    }
}
```
---
###  85.Kotlin语言的智能类型转换学习
```kotlin

open class Person3(val name: String) {
    private fun showName() = "父类显示:$name"

    open fun myPrintln() = println(showName())

    fun methodPerson() = println("我是父类的方法...") // 父类独有的函数
}

class Student3(val nameSub: String) : Person3 (nameSub) {
    override fun myPrintln() = println("子类显示:$nameSub")

    fun methodStudent() = println("我是子类的方法...") // 子类独有的函数
}

// TODO 85.Kotlin语言的智能类型转换学习
fun main() {
    val p : Person3 = Student3("李四")

    (p as Student3).methodStudent()

    p.methodStudent()

    p.methodStudent()

    p.methodStudent()

    // 智能类型转换：会根据上面 as 转成的类型，自动明白，你现在的类型就是上面的类型
}

```
---
###  86.Kotlin语言的Any超类学习
```kotlin

// 在KT中，所有的类，都隐士继承了 : Any() ,你不写，默认就有
// Any类在KT设计中：只提供标准，你看不到实现，实现在各个平台处理好了
class Obj1 : Any()
class Obj2 : Any()
class Obj3 : Any()
class Obj4 : Any()
class Obj5 : Any()
class Obj6 : Any()
// ..

// TODO 86.Kotlin语言的Any超类学习
// Any == java Object
fun main() {
    println(Obj1().toString())
}

```

---
###  87.Kotlin语言的对象声明学习
```kotlin

object KtBase87 {
    /* object 对象类背后做了什么事情

        public static final KtBase87 INSTANCE;

        private KtBase87() {} // 主构造废除一样的效果

        public final void show() {
            String var1 = "我是show函数...";
            ...
            System.out.println(var1);
        }

        // 这个区域是 object 不同点：
        static {
            KtBase87 var0 = new KtBase87();
            INSTANCE = var0;
            String var1 = "KtBase91 init...";
            ...
            System.out.println(var0);
        }

     */

    init {
        println("KtBase91 init...")
    }

    fun show() = println("我是show函数...")
}

// TODO 87.Kotlin语言的对象声明学习
fun main() {
    // object KtBase87 既是单例的实例，也是类名
    // 小结：既然是 单例的实例，又是类名，只有一个创建，这就是典型的单例
    println(KtBase87) // 背后代码：println(KtBase87.INSTANCE)
    println(KtBase87) // 背后代码：println(KtBase87.INSTANCE)
    println(KtBase87)
    println(KtBase87)
    println(KtBase87)
    println(KtBase87)

    // 背后代码：KtBase87.INSTANCE.show();
    println(KtBase87.show())
}
```
---
###  88.Kotlin语言的对象表达式学习
```kotlin

interface RunnableKT {
    fun run()
}

open class KtBase88 {

    open fun add(info: String) = println("KtBase88 add:$info")

    open fun del(info: String) = println("KtBase88 del:$info")
}

// TODO  88.Kotlin语言的对象表达式学习
// 1.add del println
// 2.匿名对象表达式方式
// 3.具名实现方式
// 4.对Java的接口 用对象表达式方式
fun main() {
    // 匿名对象 表达式方式
    val p : KtBase88 = object : KtBase88() {

        override fun add(info: String) {
            // super.add(info)
            println("我是匿名对象 add:$info")
        }

        override fun del(info: String) {
            // super.del(info)
            println("我是匿名对象 del:$info")
        }
    }
    p.add("李元霸")
    p.del("李连杰")


    // 具名实现方式
    val p2 = KtBase88Impl()
    p2.add("刘一")
    p2.del("刘二")

    // 对Java的接口 用   KT[对象表达式方式]  方式一
    val p3 = object : Runnable {
        override fun run() {
            println("Runnable run ...")
        }
    }
    p3.run()

    // 对Java的接口 用   Java最简洁的方式 方式二
    val p4 = Runnable {
        println("Runnable run2 ...")
    }
    p4.run()

    // 对KT的接口 用   KT[对象表达式方式]  方式一
    object : RunnableKT {
        override fun run() {
            println("RunnableKT 方式一 run ...")
        }
    }.run()

    // 对KT的接口 用   Java最简洁的方式 方式二
    /*RunnableKT {

    }*/
}

// 小结：Java接口，有两种方式 1（object : 对象表达式）  2简洁版，
//       KT接口，只有一种方式 1（object : 对象表达式）

// 具名实现  具体名字 == KtBase88Impl
class KtBase88Impl : KtBase88() {

    override fun add(info: String) {
        // super.add(info)
        println("我是具名对象 add:$info")
    }

    override fun del(info: String) {
        // super.del(info)
        println("我是具名对象 del:$info")
    }
}
```
---
###  89.Kotlin语言的伴生对象学习
```kotlin

class KtBase89 {

    // 伴生对象
    companion object {
        val info = "DerryINfo"

        fun showInfo() = println("显示:$info")

        val name = "Derry"
    }

    /* companion object {} 背后的逻辑

       private static final String name = "Derry";
       private static final String info = "DerryINfo";
       public static final KtBase89.Companion Companion = new KtBase89.Companion(xxx);

       public static final class Companion {

          @NotNull
          public final String getInfo() {
             return KtBase89.info;
          }

          @NotNull
          public final String getName() {
             return KtBase89.name;
          }

          public final void showInfo() {
             String var1 = "显示:" + ((KtBase89.Companion)this).getInfo();
             boolean var2 = false;
             System.out.println(var1);
          }

          private Companion() {}

          // $FF: synthetic method
          public Companion(DefaultConstructorMarker $constructor_marker) {
             this();
          }
        }

     */
}

// TODO 89.Kotlin语言的伴生对象学习
// 伴生对象的由来： 在KT中是没有Java的这种static静态，伴生很大程度上和Java的这种static静态 差不多的
// 无论 KtBase89() 构建对象多少次，我们的伴生对象，只有一次加载
// 无论 KtBase89.showInfo() 调用多少次，我们的伴生对象，只有一次加载
// 伴生对象只会初始化一次
fun main() {
    // 背后代码：System.out.println(KtBase89.Companion.getInfo())
    println(KtBase89.info)

    // 背后代码：System.out.println(KtBase89.Companion.getName())
    println(KtBase89.name)

    // 背后代码：KtBase89.Companion.showInfo()
    KtBase89.showInfo()

    // new KtBase89();
    KtBase89()
    KtBase89()
    KtBase89()
    KtBase89()
    // ...
}

```
---
###   90.Kotlin语言的嵌套类学习
```kotlin

// TODO 内部类
// 内部类的特点： 内部的类 能访问 外部的类
//              外部的类 能访问 内部的类
class Body(_bodyInfo: String) { // 身体类

    val bodyInfo = _bodyInfo

    fun show() {
        Heart().run()
    }

    // 默认情况下：内部的类 不能访问 外部的类，要增加修饰符inner 成为内部类才可以访问外部类
    inner class Heart { // 心脏类
        fun run() = println("心脏访问身体信息:$bodyInfo")
    }

    inner class Kidney { // 肾脏
        fun work() = println("肾脏访问身体信息:$bodyInfo")
    }

    inner class Hand { // 手
        inner class LeftHand { // 左手
            fun run() = println("左手访问身体信息:$bodyInfo")
        }

        inner class RightHand { // 右手
            fun run() = println("右手访问身体信息:$bodyInfo")
        }
    }
}

// TODO 嵌套类
// 默认情况下：就是嵌套类关系
// 嵌套类特点：外部的类 能访问 内部的嵌套类
//           内部的类 不能访问 外部类的成员
class Outer {

    val info: String  = "OK"

    fun show() {
        Nested().output()
    }

    class Nested {

        fun output() = println("嵌套类")

    }
}

// TODO 90.Kotlin语言的嵌套类学习
fun main() {
    // 内部类：
    Body("isOK").Heart().run()
    Body("isOK").Hand().LeftHand().run()
    Body("isOK").Hand().RightHand().run()

    // 嵌套类：
    Outer.Nested().output()

}
```
---
###  91.Kotlin语言的数据类学习
```kotlin

// 普通类
class ResponseResultBean1(var msg: String, var code: Int, var data: String) : Any()
// set get 构造函数

// 数据类 -- 一般是用于 JavaBean的形式下，用数据类
data class ResponseResultBean2(var msg: String, var code: Int, var data: String) : Any()
// set get 构造函数 解构操作 copy toString hashCode equals  数据类 生成 更丰富

// TODO 91.Kotlin语言的数据类学习
// 1.普通类 与 数据类 的 toString 背后原理
// 2.前面学习过的 == 与 ===
// 3.普通类的 == 背后原理
// 4.数据类的 == 背后原理
fun main() {
    // val (v1, v2, v3) =list  这个是list集合之前的 解构操作

    println(ResponseResultBean1("loginSuccess", 200, "登录成功的数据..."))
    // 普通类：: Any() toString Windows实现打印了   com.s5.ResponseResultBean1@266474c2

    println(ResponseResultBean2("loginSuccess", 200, "登录成功的数据..."))
    // 数据类：: Any() 默认重写了 父类的 toString  打印子类的toString详情  ResponseResultBean2(msg=loginSuccess, code=200, data=登录成功的数据...)

    println()
    // =====================

    // 前面我们学习  == 值的比较 相当于java的equals      ===引用 对象 比较

    println( // 推理 两个 普通类 的值 是一样的，应该是true ，实际背后并不是这样的
        ResponseResultBean1("loginSuccess", 200, "登录成功的数据...") ==
                ResponseResultBean1("loginSuccess", 200, "登录成功的数据...")
    )
    // Any父类的 equals 实现 （ResponseResultBean1对象引用 比较 ResponseResultBean1对象引用）


    println(
        ResponseResultBean2("loginSuccess", 200, "登录成功的数据...") ==
                ResponseResultBean2("loginSuccess", 200, "登录成功的数据...")
    )
    // Any父类的 equals 被 数据类 重写了 equals 会调用 子类的 equals函数（对值的比较）
}
```
---
###  92.Kotlin语言的copy函数学习
```kotlin

data class KtBase92 (var name: String, var age: Int) // 主构造
{
    var coreInfo : String = ""

    init {
        println("主构造被调用了")
    }

    // 次构造
    constructor(name: String) : this(name, 99) {
        println("次构造被调用")
        coreInfo = "增加非常核心的内容信息"
    }

    override fun toString(): String {
        return "toString name:$name, age:$age, coreInfo:$coreInfo"
    }
}

/* 生成的toString 为什么只有两个参数？
   答：默认生成的toString 或者 hashCode equals 等等... 主管主构造，不管次构造
    public String toString() {
      return "KtBase92(name=" + this.name + ", age=" + this.age + ")";
    }
 */

// TODO 92.Kotlin语言的copy函数学习
fun main() {
    val p1 = KtBase92("李元霸") // 调用次构造初始化对象
    println(p1)

    val newP2 = p1.copy("李连杰", 78)
    println(newP2)

    // copy toString hashCode equals 等等... 主管主构造，不管次构造
    // 注意事项：使用copy的时候，由于内部代码只处理主构造，所以必须考虑次构造的内容
}
```
---
###  93.Kotlin语言的解构声明学习
```kotlin
// 普通类
class Student1(var name: String , var age: Int, var sex: Char) {

    // 注意事项：component0 顺序必须是 component1 component2 component3 和成员一一对应，顺序下来的
    operator fun component1() = name
    operator fun component2() = age
    operator fun component3() = sex
}

// 数据类
data class Student2Data(var name: String , var age: Int, var sex: Char)

// TODO 93.Kotlin语言的解构声明学习
fun main() {
    val(name, age, sex) = Student1("李四", 89, '男')
    println("普通类 结构后:name:$name, age:$age, sex:$sex")

    val(name1, age1, sex1) = Student2Data("李四", 89, '男')
    println("数据类 结构后:name:$name1, age:$age1, sex:$sex1")

    val(_, age2, _) = Student1("李四", 89, '男')
    println("数据类 结构后: age2:$age2")
}
```
---
###  94.Kotlin语言的运算符重载学习
```kotlin
class AddClass(number1: Int, number2: Int)

// 写一个数据类，就是为了，toString 打印方便而已哦
data class AddClass2(var number1: Int, var number2: Int) {
    operator fun plus(p1: AddClass2) : Int {
        return (number1 + p1.number1) + (number2 + p1.number2)
    }

    // 查看 整个KT可以用的  运算符重载 方式
    // operator fun AddClass2.
}

// TODO 94-Kotlin语言的运算符重载学习
fun main() {
    // C++语言  +运算符重载就行了  -运算符重载就行了
    // KT语言  plus代表+运算符重载

    println(AddClass2(1, 1) + AddClass2(2, 2))
}
```

---
###  95.Kotlin语言的枚举类学习
```kotlin

// KT想表达：枚举其实也是一个class，为什么，就是为了 枚举可以有更丰富的功能
enum class Week {
    星期一,
    星期二,
    星期三,
    星期四,
    星期五,
    星期六,
    星期日
}

// TODO 95-Kotlin语言的枚举类学习
fun main() {
    println(Week.星期一)
    println(Week.星期四)

    // 枚举的值 等价于 枚举本身
    println(Week.星期二 is Week)
}
```
---
###  96.Kotlin语言的枚举类定义函数学习
```kotlin

// 四肢信息class，我就是为了方便toString打印
data class LimbsInfo (var limbsInfo: String, var length: Int) {
    fun show() {
        println("${limbsInfo}的长度是:$length")
    }
}

enum class Limbs(private var limbsInfo: LimbsInfo) {
    LEFT_HAND(LimbsInfo("左手", 88)), // 左手
    RIGHT_HAND(LimbsInfo("右手", 88)), // 右手

    LEFT_FOOT(LimbsInfo("左脚", 140)), // 左脚
    RIGHT_FOOT(LimbsInfo("右脚", 140)) // 右脚

    ; // 结束枚举值

    // 1. WEEK 这个时候 再定义单调的 枚举值，就报错了，必须所有枚举值，保持一致的效果
    // 2. 枚举的 主构造的参数 必须和 枚举(的参数) 保持一致

    fun show() = "四肢是:${limbsInfo.limbsInfo}的长度是:${limbsInfo.length}"

    fun updateData(limbsInfo: LimbsInfo) {
        println("更新前的数据是:${this.limbsInfo}")
        this.limbsInfo.limbsInfo = limbsInfo.limbsInfo
        this.limbsInfo.length = limbsInfo.length
        println("更新后的数据是:${this.limbsInfo}")
    }
}

// TODO 96-Kotlin语言的枚举类定义函数学习
fun main() {
    // 显示枚举值

    // 一般不会这样用
    /*println(Limbs.show())
    println(Limbs().show())*/

    // 一般的用法如下：
    println(Limbs.LEFT_HAND.show())
    println(Limbs.RIGHT_HAND.show())
    println(Limbs.LEFT_FOOT.show())
    println(Limbs.RIGHT_FOOT.show())

    println()

    // 更新枚举值
    Limbs.RIGHT_HAND.updateData(LimbsInfo("右手2", 99))
    Limbs.LEFT_HAND.updateData(LimbsInfo("左手2", 99))
    Limbs.LEFT_FOOT.updateData(LimbsInfo("左脚2", 199))
    Limbs.RIGHT_FOOT.updateData(LimbsInfo("右叫2", 199))
}
```
---
###  97-Kotlin语言的代数数据类型
```kotlin

enum class Exam {
    Fraction1, // 分数差
    Fraction2, // 分数及格
    Fraction3, // 分数良好
    Fraction4, // 分数优秀

    ; // 枚举结束

    // 需求 得到优秀的孩子姓名
    var studentName: String? = null
    // 我们用枚举类，要做到此需求，就非常的麻烦了，很难做到而已，不是做不到
    //  需求：引出 密封类
}

class Teacher (private val exam: Exam) {
    fun show() =
        when (exam) {
            Exam.Fraction1 -> "该学生分数很差"
            Exam.Fraction2 -> "该学生分数及格"
            Exam.Fraction3 -> "该学生分数良好"
            Exam.Fraction4 -> "该学生分数优秀"
            // else -> 由于我们的show函数，是使用枚举类类型来做判断处理的，这个就属于 代数数据类型，就不需要写 else 了
            // 因为when表达式非常明确了，就只有 四种类型，不会出现 else 其他，所以不需要写
        }
}

// TODO 97-Kotlin语言的代数数据类型
// 1.定义枚举Exam类，四个级别分数情况
// 2.定义Teacher老师类，when使用枚举类
// 3.需求 得到优秀的孩子姓名
fun main() {
    println(Teacher(Exam.Fraction1).show())
    println(Teacher(Exam.Fraction3).show())
}
```
---
###  98-Kotlin语言的密封类学习
```kotlin

// 密封类，我们成员， 就必须有类型 并且 继承本类
sealed class Exams {
    // object？ Fraction1 Fraction3 都不需要任何成员，所以一般都写成object，单例就单例，无所谓了
    object Fraction1 : Exams() // 分数差
    object Fraction2 : Exams() // 分数及格
    object Fraction3 : Exams() // 分数良好

    // 假设 Fraction4 是可以写object的，那么也不合理，因为对象不是单例的，有 对象1李四 对象2王五
    class Fraction4(val studentName : String) : Exams() // 分数优秀

    // 需求 得到优秀的孩子姓名
    // var studentName: String? = null
    // 我们用枚举类，要做到此需求，就非常的麻烦了，很难做到而已，不是做不到
    //  需求：引出 密封类
}

class Teachers (private val exam: Exams) {
    fun show() =
        when (exam) {
            is Exams.Fraction1 -> "该学生分数很差"
            is Exams.Fraction2 -> "该学生分数及格"
            is Exams.Fraction3 -> "该学生分数良好"
            is Exams.Fraction4 -> "该学生分数优秀：该学生的姓名是:${(this.exam as Exams.Fraction4).studentName}"
        }
}

// TODO 98-Kotlin语言的密封类学习
fun main() {
    println(Teachers(Exams.Fraction1).show())
    println(Teachers(Exams.Fraction2).show())
    println(Teachers(Exams.Fraction3).show())
    println(Teachers(Exams.Fraction4("李四")).show()) // 对象1
    println(Teachers(Exams.Fraction4("王五")).show()) // 对象2

    println(Exams.Fraction1 === Exams.Fraction1) // true, === 必须对象引用， object是单例 只会实例化一次

    println(Exams.Fraction4("AAA") === Exams.Fraction4("AAA")) // class 有两个不同的对象，所以是false
}
```
---
### 99-数据类使用条件
```kotlin

data class LoginRequest(var info: String)

// TODO 99-数据类使用条件

// 条件一：服务器请求回来的响应的 JavaBean  LoginResponseBean 基本上可以使用 数据类
// 条件二：数据类至少必须有一个参数的主构造函数
// 条件三：数据类必须有参数， var val 的参数
// 条件四：数据类不能使用 abstract，open，sealed，inner 等等 修饰 （数据类，数据载入的事情 数据存储）
// 条件五：需求 比较，copy，toString，解构，等等 这些丰富的功能时，也可以使用数据类
fun main() {}
```
