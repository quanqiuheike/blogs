
---
###  100.Kotlin语言的接口定义
```kotlin
interface IUSB {
    var usbVersionInfo: String // USB版本相关的信息
    var usbInsertDevice: String // USB插入的设备信息

    fun insertUBS() : String
}

// 鼠标UBS实现类
class Mouse(override var usbVersionInfo: String = "USB 3.0",
            override var usbInsertDevice: String = "鼠标接入了UBS口") :IUSB {

    override fun insertUBS() = "Mouse $usbVersionInfo, $usbInsertDevice"
}

// 键盘USB实现类
class KeyBoard : IUSB {

    override var usbVersionInfo: String = "USB 3.1"
        // 下面的 set get 都会持有 field，现在是你没有给 usbVersionInfo 赋值， 意味着field是没法让set/get持有的
        get() = field
        set(value) {
            field = value
        }

    override var usbInsertDevice: String = "键盘接入了UBS口"
        get() {
            println("@你get了[${field}]值出去了")
            return field
        }
        set(value) {
            field = value
            println("@你set了[${value}]值进来了")
        }

    override fun insertUBS(): String = "KeyBoard $usbVersionInfo, $usbInsertDevice"
}

// TODO 100-Kotlin语言的接口定义
// 1.接口里面的所有成员 和 接口本身 都是 public open 的，所以不需要open，这个是接口的特殊
// 2.接口不能有主构造，反正就是没有构造
// 3.实现类不仅仅要重写接口的函数，也要重写 接口的成员
// 4.接口实现代码区域，全部都要增加 override 关键字来修饰
fun main() {
    val iusb1 : IUSB = Mouse()
    println(iusb1.insertUBS())

    println()

    val iusb2: IUSB = KeyBoard()
    println(iusb2.insertUBS())

    iusb2.usbInsertDevice = "AAA"
}
```
---
###  101.Kotlin语言的接口的默认实现
```kotlin

interface USB2 {

    // 1.接口 var 也是不能给接口的成员赋值的 （但是有其他办法）
    // 2.任何类 接口 等等  val 代表只读的，是不可以在后面动态赋值 （也有其他办法）

    val usbVersionInfo: String // USB版本相关的信息
       get() = (1..100).shuffled().last().toString()
       // val 不需要set

    val usbInsertDevice: String // USB插入的设备信息
        get() = "高级设备接入USB"
        // val 不需要set

    fun insertUBS() : String
}

// 鼠标UBS实现类
class Mouse2 : USB2 {

    override val usbInsertDevice: String
        get() = super.usbInsertDevice

    override val usbVersionInfo: String
        get() = super.usbVersionInfo

    override fun insertUBS() = "Mouse $usbVersionInfo, $usbInsertDevice"
}

// TODO 101-Kotlin语言的接口的默认实现
// 1.注意：这样做是不对的，因为接口成员本来就是用来声明标准的
//        但是可以在接口成员声明时，完成对接口成员的实现
fun main() {
    val iusb1 = Mouse2()
    println(iusb1.insertUBS())
}
```
---
###  102.Kotlin语言的抽象类学习
```kotlin

abstract class BaseActivity {

    fun onCreate() {
        setContentView(getLayoutID())

        initView()
        initData()
        initXXX()
    }

    private fun setContentView(layoutID: Int) = println("加载{$layoutID}布局xml中")

    abstract fun getLayoutID(): Int
    abstract fun initView()
    abstract fun initData()
    abstract fun initXXX()
}

class MainActivity : BaseActivity() {

    override fun getLayoutID(): Int = 564

    override fun initView() = println("做具体初始化View的实现")

    override fun initData() = println("做具体初始化数据的实现")

    override fun initXXX() = println("做具体初始化XXX的实现")

    fun show() {
        super.onCreate()
    }
}

class LoginActivity : BaseActivity() {

    override fun getLayoutID(): Int = 564

    override fun initView() = println("做具体初始化View的实现")

    override fun initData() = println("做具体初始化数据的实现")

    override fun initXXX() = println("做具体初始化XXX的实现")

    fun show() {
        super.onCreate()
    }
}

// TODO 102-Kotlin语言的抽象类学习
fun main() = LoginActivity().show()
```
---
### 103.Kotlin语言的定义泛型类
```kotlin

class KtBase103<T> (private val obj: T) { // 万能输出器
    fun show() = println("万能输出器:$obj")
}

data class Student(val name: String , val age: Int, val sex: Char)
data class Teacher(val name: String , val age: Int, val sex: Char)

// TODO 103-Kotlin语言的定义泛型类
// 1.定义 对象输出器 println(obj)
// 2.定义两个对象，三个属性
// 3.对象 String Int Double Float Char 等 测试 对象输出器
fun main() {
    val stu1 = Student("张三", 88, '男')
    val stu2 = Student("李四", 78, '女')

    val tea1 = Teacher("王五", 77, '男')
    val tea2 = Teacher("赵六", 89, '女')

    KtBase103(stu1).show()
    KtBase103(stu2).show()
    KtBase103(tea1).show()
    KtBase103(tea2).show()

    KtBase103(String("刘一".toByteArray())).show()

    KtBase103(575).show()
    KtBase103(53456.45).show()
    KtBase103(4645.5f).show()
    KtBase103('男').show()

}
```

---
###  104.Kotlin语言的泛型函数学习
```kotlin

// 1.万能对象返回器 Boolean来控制是否返回 运用 takeIf
class KtBase104<T>(private val isR: Boolean, private val obj: T) {

    fun getObj() : T? = obj.takeIf { isR }

}

// TODO 104-Kotlin语言的泛型函数学习
// 1.万能对象返回器 Boolean来控制是否返回 运用 takeIf
// 2.四个对象打印
// 3.对象打印 + run + ?:
// 4.对象打印 + apply + ?:
// 5.show(t: T) + apply + ?:
fun main() {
    val stu1 = Student("张三", 88, '男')
    val stu2 = Student("李四", 78, '女')

    val tea1 = Teacher("王五", 77, '男')
    val tea2 = Teacher("赵六", 89, '女')

    // 2.四个对象打印
    println(KtBase104(true, stu1).getObj())
    println(KtBase104(true, stu2).getObj())
    println(KtBase104(true, tea1).getObj())
    println(KtBase104(true, tea2).getObj())

    println(KtBase104(false, tea2).getObj() ?: "大哥，你万能对象返回器，是返回null啊")

    println()

    // 3.对象打印 + run + ?:
    val r : Any = KtBase104(true, stu1).getObj() ?.run {
        // 如果 getObj 返回有值，就会进来
        // this == getObj本身
        println("万能对象是:$this") // 返回Unit
        545.4f // 返回Float
    } ?: println("大哥，你万能对象返回器，是返回null啊") // 返回Unit
    println(r)

    println()

    // apply特点：永远都是返回 getObj.apply  getObj本身
    val r2 : Student = KtBase104(true, stu2).getObj().apply {  }!!
    println("r2:$r2")

    // 4.对象打印 + apply + ?:
    val r3: Teacher = KtBase104(true, tea1).getObj() .apply {
        // this == getObj本身

        if (this == null) {
            println("大哥，你万能对象返回器，是返回null啊")
        } else {
            println("万能对象是:$this")
        }
    }!!
    println("r3:$r3")

    println()

    show("Derry")
    show("Kevin")
    show("OK")
    show(null)

    println()

    show2("Derry")
    show2("Kevin")
    show2("OK")
    show2(null)
}

// 5.show(t: T) + also + ?:
fun <B> show(item: B) {
    item ?.also {
        // it == item本身
        println("万能对象是:$it")
    } ?: println("大哥，你万能对象返回器，是返回null啊")
}

fun <B> show2(item: B) {
    // var r0 = item

    var r : B? = item ?.also {
       if (it == null) {
           println("大哥，你万能对象返回器，是返回null啊")
       } else {
           println("万能对象是:$it")
       }
    } ?: null
    println("show2: 你传递进来的r:$r")
}
```
---
###  105.Kotlin语言的泛型变换实战
```kotlin

// 1.类 isMap map takeIf  map是什么类型
class KtBase105<T>(val isMap: Boolean = false, val inputType: T) {

    // 模仿RxJava  T是要变化的输入类型   R是变换后的输出类型
    // 要去map返回的类型是 R?  == 有可能是R 有可能是null
    inline fun <R> map(mapAction: (T) -> R) = mapAction(inputType).takeIf { isMap }
}

inline fun <I, O> map(inputValue : I , isMap: Boolean = true, mapActionLambda : (I) -> O) =
    if (isMap) mapActionLambda(inputValue) else null

// TODO 105-Kotlin语言的泛型变换实战
// 1.类 isMap map takeIf  map是什么类型
// 2.map int -> str 最终接收是什么类型
// 3.map per -> stu 最终接收是什么类型
// 4.验证是否是此类型 与 null
fun main() {

    // 2.map int -> str 最终接收是什么类型
    val p1 = KtBase105(isMap = /*true*/ false, inputType = 5434)

    val r = p1.map {
        it
        it.toString() // lambda最后一行是 返回值
        "我的it是:$it" // lambda最后一行是 返回值
    }

    // 4.验证是否是此类型 与 null
    val str1: String = "OK1"
    val str2: String? = "OK2"
    println(r is String)
    println(r is String?)
    println(r ?: "大哥你是null，你在搞什么飞机...,你是不是传入了isMap是false")

    println()

    // 3.map per -> stu 最终接收是什么类型
    val p2 = KtBase105(true, Persons("李四", 99))
    val r2 : Students? = p2.map {
        // it == Persons对象 == inputType
        it
        Students(it.name, it.age)
    }
    println(r2)

    println()

    // map函数 模仿RxJava变换操作
    val r3 = map(123) {
        it.toString()
        "map包裹[$it]" // lambda表达式最后一行，就是返回值
    }
    println(r3)

    123.run {  }
}

data class Persons(val name: String, val age: Int)
data class Students(val name: String, val age: Int)
```
---
###  106.Kotlin语言的泛型类型约束学习
```kotlin

import java.io.File

open class MyAnyClass(name: String) // 祖宗类 顶级父类

open class PersonClass(name: String) : MyAnyClass(name = name) // 父类

class StudentClass(name: String) : PersonClass(name = name) // 子类

class TeacherClass(name: String) : PersonClass(name = name) // 子类

class DogClass(name: String) // 其他类 另类

class CatClass(name: String) // 其他类 另类

// TODO 106-Kotlin语言的泛型类型约束学习
// T : PersonClass   相当于  Java的 T extends PersonClass
// PersonClass本身 与 PersonClass的所有子类 都可以使用， 其他的类，都不能兼容此泛型
class KtBase106<T : PersonClass> (private val inputTypeValue: T, private val isR: Boolean = true) {

    // 万能对象返回器
    fun getObj() = inputTypeValue.takeIf { isR }
}

fun main() {
    val any = MyAnyClass("Derry1")// 祖宗类 顶级父类

    val per = PersonClass("Derry1") // 父类

    val stu = StudentClass("Derry1") // 子类
    val pea = TeacherClass("Derry1") // 子类

    val dog = DogClass("Derry1") // 其他类 另类

    /*val r1 = KtBase106(any).getObj() // 报错了，类型限定了
    println(r1)*/

    val r2 = KtBase106(per).getObj()
    println(r2)

    val r3 = KtBase106(stu).getObj()
    println(r3)

    val r4 = KtBase106(stu).getObj()
    println(r4)

    /*val r5 = KtBase106(dog).getObj() // 报错了，类型限定了
    println(r5)*/

    // KtBase106(CatClass("cat 小白")) // 报错了，类型限定了
}
```
---
###  107.Kotlin语言的vararg关键字(动态参数)
```kotlin

// 纠正： 为什么 it 是 String ? , 是因为你的  lambda (T ?) -> O  T? 指定了 ？
class KtBase107<T> (vararg objects : T, var isMap: Boolean) {

    // 1.objectArray:Array<T>
    // out 我们的T只能被 读取，不能修改   T只能读取
    private val objectArray : Array<out T> = objects

    // 2.showObj(index)  "你${index}下标去的对象是null"
    fun showObj(index: Int) : T? = objectArray[index].takeIf { isMap } ?: null /*objectArray[index]*/

    // 3.mapObj(index, 变换lambda)   objectArray[index]
    fun <O> mapObj(index: Int, mapAction: (T ?) -> O)  = mapAction( objectArray[index].takeIf { isMap }  /*objectArray[index]*/ )
}

// TODO 107-Kotlin语言的vararg关键字(动态参数)
// 1.objectArray:Array<T>
// 2.showObj(index)
// 3.mapObj(index,变换lambda)
// 4.p.showOBj  p.mapObj(int -> str)
// 5.p的类型  it的类型
fun main() {
    // * Java ?
    // p的类型 ?

    // 由于你使用的 太多类型的混合了，泛型 这个才是他真正的类型 : KtBase107<{Comparable<*> & java.io.Serializable}>
    //  由于不允许我们这样写 : KtBase107<{Comparable<*> & java.io.Serializable}> 所以我们用父类 Any? 代替
    val p : KtBase107<Any?>  = KtBase107("Derry", false, 53454, 4543.3f, 4554.54, null, 'C', isMap = true)

    println(p.showObj(0))
    println(p.showObj(1))
    println(p.showObj(2))
    println(p.showObj(3))
    println(p.showObj(4)) // 4554.54
    println(p.showObj(5)/*?.特殊操作 如果是null 会引发奔溃*/) // null
    println(p.showObj(6)) // C

    println()

    // mapObj
    // it的类型  实际上 真正的类型 {Comparable<*> & java.io.Serializable}  需要转换一下才行 例如：it.toString
    val r : Int = p.mapObj(0) {
        it
        it.toString()
        it.toString().length
    }
    println("第零个元素的字符串长度是:$r")

    // it的类型  实际上 真正的类型 {Comparable<*> & java.io.Serializable}  由于我们的第三个元素是 Int类型，所以不需要转换，自动转的
    val r2 : String = p.mapObj(2) {
        "我的第三个元素是:$it"
    }
    println(r2)

    // >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    val p2 : KtBase107<String> = KtBase107("AAA", "BBB", "CCC", isMap = true)
    val r3 = p2.mapObj(2) {
        it
        // it 是什么类型 ？  String ?
        "我要把你变成String类型 it:$it"
    }
    println(r3)
}
```
---
###  108.Kotlin语言的[ ]操作符学习
```kotlin

class KtBase108 <INPUT> (vararg objects: INPUT, val isR: Boolean = true) {

    // 开启INPUT泛型的只读模式
    private val objectArray: Array<out INPUT> = objects

    // 1.5种返回类型变化的解释
    fun getR1() : Array<out INPUT> ? = objectArray.takeIf { isR }

    // 有可能是 Array<out INPUT>   也有可能是: String     就属于这种类型：Serializable， 用Any代替
    fun getR2() : Any = objectArray.takeIf { isR } ?: "你是null了"

    // 有可能是 Array<out INPUT>   也有可能是: String  也有可能是 ?   就属于这种类型：Serializable?  用Any?代替
    fun getR3() : Any? = objectArray.takeIf { isR } ?: "你是null了" ?: null

    fun getR4(index: Int) : INPUT ? = objectArray[index].takeIf { isR } ?: null

    // INPUT Float Int Char String ... = Any ?
    fun getR5(index: Int) : Any ? = objectArray[index].takeIf { isR } ?: "AAA" ?: 546 ?: 6445.546f ?: 'C' ?: false ?: null

    // 运算符重载
    operator fun get(index: Int) : INPUT ? = objectArray[index].takeIf { isR }
}

// 2.给泛型传入null后，直接操作
// 泛型是很大的范围类型，可以接收很多类型，当然也可以接收null，但是接收null后，要处理好
fun <INPUT> inputObj(item: INPUT) {
    // println((item as String).length) // 在泛型中，不能这样做，这个是不标准的

    // 泛型是很大的范围类型，可以接收很多类型，当然也可以接收null，但是接收null后，要处理好，
    // String? 能够接收 "Derry" "Kevins" 还可以接收null，所以 Stirng? 比 String 功能强大
    // 小结：异步处理泛型接收，都用 String? 处理  规范化
    println((item as String?)?.length ?: "你个货传递的泛型数据是null啊")
}

// TODO 108-Kotlin语言的[ ]操作符学习
// 1.5种返回类型变化的解释
// 2.给泛型传入null后，直接操作
fun main() {
    inputObj("Derry")
    inputObj("Kevins")
    inputObj(null)

    println()

    // 只要有一个元素是null，那么所有的元素都是 String?
    val p1 : KtBase108<String?> = KtBase108("张三", "李四", "王五", null)

    var r : String? = p1[0]
    val r2 : String ? = p1[3]

    println(r)
    println(p1[1])
    println(p1[2])
    println(r2)
}
```
---

```java

public class KtBase109 {

    public static void main(String[] args) {
        List<CharSequence> list = new ArrayList<CharSequence>();

        // 泛型默认情况下是：泛型的子类对象 不可以赋值给 泛型的父类对象
        // List<CharSequence> list1 = new ArrayList<String>();
        // 泛型默认情况下是：泛型具体处的子类对象  不可以赋值给 泛型声明处的父类对象

        // CharSequence父类        String子类

        // ? extends T 就相当于 KT里面的out，所以才可以 泛型子类对象 赋值给 泛型父类对象
        // out: 泛型具体出的子类对象 可以赋值给 泛型声明处的父类对象

        List<? extends CharSequence> list2 = new ArrayList<String>();

        // 协变：父类 泛型声明处  可以接收   子类 泛型具体处
    }
}
```
---
###   109.Kotlin语言的out-协变学习
```kotlin

// 生产者 out T  协变 [out T 此泛型能够被获取 读取 所以是out]
interface Producer<out T> {

    // out T  代表整个生产者类里面  这个T  只能被读取，不能被修改了

    // 不能被修改了 （编译不通过）
    // fun consumer(itme: T)  /*{  消费代码  }*/

    // 只能被读取
    fun producer() : T
}

// 消费者 in T  逆变 [in T 此泛型只能被修改 更新 所以是in]
interface Consumer <in T> {
    // out T  代表整个生产者类里面  这个T  只能被读取，不能被修改了

    // 只能被修改了
    fun consumer(itme : T) /*{  消费代码  }*/

    // 不能被读取 （编译不通过）
    // fun producer() : T
}

// 生产者&消费者 T  默认情况下，是不变
interface ProducerAndConsumer<T> {
    // 能被修改了
    fun consumer(itme : T) /*{  消费代码  }*/

    // 能被读取
    fun producer() : T
}

open class Animal // 动物
open class Humanity : Animal() // 人类
open class Man : Humanity() // 男人
open class WoMan : Humanity() // 女人

// >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  只管生产者
class ProducerClass1 : Producer<Animal> {
    override fun producer(): Animal {
        println("生产者 Animal")
        return Animal()
    }
}

class ProducerClass2 : Producer<Humanity> {
    override fun producer(): Humanity {
        println("生产者 Humanity")
        return Humanity()
    }
}

class ProducerClass3 : Producer<Man> {
    override fun producer(): Man {
        println("生产者 Man")
        return Man()
    }
}

class ProducerClass4 : Producer<WoMan> {
    override fun producer(): WoMan {
        println("生产者 WoMan")
        return WoMan()
    }
}

// TODO 109-Kotlin语言的out-协变学习
// 1.Producer Consumer 不变
// 2.ProducerClass4
// 3.main 测试
fun main() {
    val p1 : Producer<Animal> = ProducerClass1() // ProducerClass1他本来就是 传递 Animal ，当然是可以的

    val p2 : Producer<Animal> = ProducerClass2() // ProducerClass2他本来就是 传递 Humanity，居然也可以，因为out
    val p3 : Producer<Animal> = ProducerClass3() // ProducerClass3他本来就是 传递 Man，居然也可以，因为out
    val p4 : Producer<Animal> = ProducerClass4() // ProducerClass4他本来就是 传递 WoMan，居然也可以，因为out

    // 泛型默认情况下是：泛型的子类对象 不可以赋值给 泛型的父类对象
    // 泛型默认情况下是：泛型具体处的子类对象  不可以赋值给 泛型声明处的父类对象

    // out: 泛型的子类对象 可以赋值给 泛型的父类对象
    // out: 泛型具体出的子类对象 可以赋值给 泛型声明处的父类对象

    // 协变：父类 泛型声明处  可以接收   子类 泛型具体处
}

```
---

```kotlin

public class KtBase110 {

    public static void main(String[] args) {
        List<CharSequence> list = new ArrayList<CharSequence>();

        // 泛型默认情况下是：泛型的父类对象 不可以赋值给 泛型的子类对象
        // List<String> list1 = new ArrayList<CharSequence>();

        // 泛型默认情况下是：泛型具体处的父类对象  不可以赋值给  泛型声明处的子类对象

        // CharSequence父类        String子类

        // ? super T 就相当于 KT里面的in，所以才可以 泛型父类对象 赋值给 泛型子类对象
        // in: 泛型具处的父类对象 可以赋值给 泛型声明处的子类对象

        List<? super String> list2 = new ArrayList<CharSequence>();

        // 逆变：子类 泛型声明处  可以接收   父类 泛型具体处
    }
}
```

---
###  110.Kotlin语言的in-逆变学习
```kotlin

// >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  只管消费者
class ConsumerClass1 : Consumer<Animal> {
    override fun consumer(item: Animal) {
        println("消费者 Animal")
    }
}

class ConsumerClass2 : Consumer<Humanity> {
    override fun consumer(item: Humanity) {
        println("消费者 Humanity")
    }
}

class ConsumerClass3 : Consumer<Man> {
    override fun consumer(item: Man) {
        println("消费者 Man")
    }
}

class ConsumerrClass4 : Consumer<WoMan> {
    override fun consumer(item: WoMan) {
        println("消费者 WoMan")
    }
}

// TODO 110-Kotlin语言的in-逆变学习
fun main() {
    val p1 : Consumer<Man> = ConsumerClass1() // ConsumerClass1他本来就是 传递 Animal，居然也可以，因为in
    val p2 : Consumer<WoMan> = ConsumerClass2() // ConsumerClass1他本来就是 传递 Humanity，居然也可以，因为in

    // 默认情况下： 泛型具体出的父类  是不可以赋值给  泛型声明处的子类的
    // in：泛型具体出的父类  是可以赋值给  泛型声明处的子类的

    // 逆变：子类 泛型声明处  可以接收   父类 泛型具体处


    // 协变：out 父类 = 子类
    // 逆变：in  子类 = 父类
}
```
---

```java
// in T  out T 声明处指定关系  声明处泛型  这个是Java没有的功能
public class KtBase111</*? extends */T> {
}

```
---
###  111.Kotlin语言中使用in和out
```kotlin

// in T  out T 声明处指定关系  声明处泛型  这个是Java没有的功能

// 整个 SetClass 里面的所有成员 泛型相关，只能修改 更改，
//                                    不能获取人家 读取人家的
// 小结：当我们 对这个整个类里面的泛型，只能修改 ，不能让外界读取时，可以声明 in T 逆变
class SetClass<in T>() {

    // 200个函数 这200个函数 对T只能修改，不能给外界读取
    // ...

    fun set1(item: T) {
        println("set1 你要设置的item是:$item")
    }

    fun set2(item: T) {
        println("set2 你要设置的item是:$item")
    }

    fun set3(item: T) {
        println("set3 你要设置的item是:$item")
    }

    // ...

    // 不能给外界读取 (增加in后，不能给外界读取，所以编译不通过)
    /*fun get1() : T? {
        return null
    }

    fun get2() : T? {
        return null
    }

    fun get3() : T? {
        return null
    }*/

    // ...
}


// 整个 GetClass 里面的所有成员 泛型相关，不能修改 更改，
//                                    只能获取人家 读取人家的
// 小结：当我们 对这个整个类里面的泛型，只能给读取 ，不能修改 更改，可以声明 out T 协变
class GetClass<out T>(_item: T) {

    val item: T = _item

    // 200个函数 这200个函数 对T只能读取，不能给外界修改 更改
    // ...

    // 不能给外界修改 更改 (增加out后，不能给外界修改 更改，所以编译不通过)
    /*fun set1(item : T) {
        println("set1 你要设置的item是:$item")
    }

    fun set2(item : T) {
        println("set2 你要设置的item是:$item")
    }

    fun set3(item : T) {
        println("set3 你要设置的item是:$item")
    }*/

    // ...


    fun get1(): T {
        return item
    }

    fun get2(): T {
        return item
    }

    fun get3(): T? {
        return item
    }

    // ...
}

// TODO 111-Kotlin语言中使用in和out
fun main() {
    // 逆变 in T  SetClass 只能修改 更改 不能给外界读取
    val p1 = SetClass<String>()
    p1.set1("Derry")
    p1.set2("Kevin")

    println()

    // 协变 out T GetClass 只能读取，不能修改 更改
    val p2 = GetClass("李四")
    println(p2.get1())
    val p3 = GetClass("王五")
    println(p3.get3())
}
```

---
###  112.Kotlin语言的reified关键字学习
```kotlin

// 1.定义3个Obj类
data class ObjectClass1(val name: String, val age: Int, val study: String)
data class ObjectClass2(val name: String, val age: Int, val study: String)
data class ObjectClass3(val name: String, val age: Int, val study: String)

class KtBase112 {

    // 所有的功能，写在函数上
    // 默认随机输出一个对象，如果此对象和用户指定的对象不一致，我们就启用备用对象，否则就直接返回对象
    inline fun <reified T> randomOrDefault(defaultLambdaAction: () -> T ) :T? {
        val objList : List<Any> = listOf(ObjectClass1("obj1 李四", 22, "学习C"),
                                         ObjectClass2("obj2 王五", 23, "学习C++"),
                                         ObjectClass3("obj3 赵六", 24, "学习C#"))

        val randomObj : Any? = objList.shuffled().first()

        println("您随机产生的对象 幸运儿是:$randomObj")

        // return randomObj.takeIf { it is T } as T ?: null     :T? {

        // T  与  T?  是不同的 ？
        // 答： it is T false  takeIf  null    null as T 奔溃了，解决思路： null as T?

        // 如果  it随机产生的对象 等于 T类型的，就会走 as T 直接返回了
        return randomObj.takeIf { it is T } as T?  // null as T     null as T?
            // 如果  it随机产生的对象 不等于 T类型的，就会走下面这个备用环节
            ?: defaultLambdaAction()
    }

}

// TODO 112-Kotlin语言的reified关键字学习
// 1.定义3个Obj类
// 2.randomOrDefault函数 备用机制的lambda
// 3.lists.shuffled()
fun main() {
    val finalResult = KtBase112().randomOrDefault<ObjectClass1> {
        println("由于随机产生的对象 和 我们指定的ObjectClass1不一致，所以启用备用对象")
        ObjectClass1("备用 obj1 李四", 22, "学习C") // 最后一行的返回
    }
    println("客户端最终结果:$finalResult")
}
```
---
###  113.Kotlin语言的定义扩展函数学习

```kotlin

// 假设这个代码是，开源的，或者是很庞大JDK源码，或者是非常复杂的开源库
class KtBase113 (val name: String, val age: Int, val sex: Char)

// 增加扩展函数
fun KtBase113.show() {
    println("我是show函数, name:${name}, age:$age, sex:$sex")
}

// 增加扩展函数
fun KtBase113.getInfo() = "我是getInfo函数, name:${name}, age:$age, sex:$sex"

fun String.addExtAction(number: Int) =  this + "@".repeat(number)

fun String.showStr() = println(this)

/* 增加扩展函数后 的 背后代码

    public final class KtBase113Kt {

        public static final void show(KtBase113 $this$show) {
            System.out.println("我是show函数, name:" + $this$show.name + ", age:" + $this$show.age, sex:" + $this$show.sex);
        }

        public static final void getInfo(KtBase113 $this$getInfo) {
            return "我是getInfo函数, name:" + $this$show.name + ", age:" + $this$show.age, sex:" + $this$show.sex;
        }

        public static final void showStr(String $this$showStr) {
            System.out.println($this$showStr);
        }

        public static final String addExtAction(String $this$addExtAction) {
           return $this$addExtAction + StringsKt.repeat((CharSequence)"@", number);
        }

        public static void main(String [] args) {
            main();
        }

        public static void main() {
            // ...
        }
    }

 */

// KtBase113.xxx  xxx函数里面会持有this == KtBase113对象本身
// TODO 113-Kotlin语言的定义扩展函数学习
fun main() {
    val p = KtBase113("张三", 28, '男')
    p.show()
    println(p.getInfo())

    println("Derry".addExtAction(8))
    println("Kevin".addExtAction(3))

    "这个是我的日志信息".showStr()
    "Beyond".showStr()
}
```
---
###  114.Kotlin语言的超类上定义扩展函数
```kotlin

import java.io.File
import java.nio.charset.Charset
import java.util.ArrayList

data class ResponseResult1(val msg: String, val code: Int)
data class ResponseResult2(val msg: String, val code: Int)
data class ResponseResult3(val msg: String, val code: Int)
data class ResponseResult4(val msg: String, val code: Int)
// 省略几亿个类 ....

// 最超类进行 一个函数 扩展
fun Any.showPrintlnContent() = println("当前内容是:$this")

fun Any.showPrintlnContent2() : Any {
    println("当前内容是:$this")

    return this
}

// TODO 114-Kotlin语言的超类上定义扩展函数
// 1.扩展函数不允许被重复定义
// 2.对超类扩展函数的影响
// 3.扩展函数 链式调用
fun main() {
    ResponseResult1("login success", 200).showPrintlnContent()
    ResponseResult2("login success", 200).showPrintlnContent()
    ResponseResult3("login success", 200).showPrintlnContent()
    ResponseResult4("login success", 200).showPrintlnContent()

    "Derry1".showPrintlnContent()
    "Kevin1".showPrintlnContent()
    val number1 = 999999
    number1.showPrintlnContent()
    val number2 = 645654.6
    number2.showPrintlnContent()
    val number3 = 544354.5f
    number3.showPrintlnContent()
    val sex = '男'
    sex.showPrintlnContent()

    println()

    '女'.showPrintlnContent2().showPrintlnContent2().showPrintlnContent2()
    "DerryOK".showPrintlnContent2().showPrintlnContent2().showPrintlnContent2().showPrintlnContent2()

    println(File("D:\\a.txt").readLines())
}
// 第一点：如果我们自己写了两个一样的扩展函数，编译不通过

// 第二点：KT内置的扩展函数，被我们重复定义，属于覆盖，而且优先使用我们自己定义的扩展函数
public fun File.readLines(charset: Charset = Charsets.UTF_8): List<String> {
    val result = ArrayList<String>()
    forEachLine(charset) { result.add(it); }
    return result
}
```
---
###  115.Kotlin语言的泛型扩展函数
```kotlin

// 1.String类型就输出长度
fun <T> T.showContentInfo() = println("${if (this is String) "你的字符串长度是:$length" else "你不是字符串 你的内容是:$this"}")

// 2.显示调用时间
fun <I> I.showTime() = println("你当前调用的时间是:${System.currentTimeMillis()}, 内容是:$this")

// 3.显示调用者的类型
fun <INPUTTYPE> INPUTTYPE.showTypesAction() =
    when(this) {
        is String -> "原来你是String类型"
        is Int -> "原来你是Int类型"
        is Char -> "原来你是Char类型"
        is Float -> "原来你是Float类型"
        is Double -> "原来你是Double类型"
        is Boolean -> "原来你是Boolean类型"
        is Unit -> "原来你是无参返回函数类型"
        else -> "未知类型"
    }

fun commonFun() {}

// fun commonFun2() = "DDD" 这个不管他

// TODO 115-Kotlin语言的泛型扩展函数
// 1.String类型就输出长度
// 2.显示调用时间
// 3.显示调用者的类型
fun main() {
    345.showContentInfo()
    'C'.showContentInfo()
    false.showContentInfo()
    345.45f.showContentInfo()
    53454.45.showContentInfo()
    "Derry".showContentInfo()
    commonFun().showContentInfo()
    // commonFun2().showContentInfo()
    // 太多了，不写了 .... 省略

    // 所有类型 都是泛型，你对泛型扩展了showContentInfo，那么所有类型都可以使用showContentInfo

    println()

    345.showTime()
    'C'.showTime()
    false.showTime()
    345.45f.showTime()
    53454.45.showTime()
    "Derry".showTime()
    commonFun().showTime()
    // commonFun2().showTime()
    // 太多了，不写了 .... 省略

    println()

    println(345.showTypesAction())
    println('C'.showTypesAction())
    println(false.showTypesAction())
    println(345.45f.showTypesAction())
    println(53454.45.showTypesAction())
    println("Derry".showTypesAction())
    println(commonFun().showTypesAction())
    // println(commonFun2().showTypesAction())
}
```
---
###  116.Kotlin语言的标准函数与泛型扩展函数
```kotlin

// TODO 116-Kotlin语言的标准函数与泛型扩展函数
fun main() {
    val r: Char = "Derry".mLet {
        it
        true
        "OK"
        'A'
    }

    123.mLet {
        it
    }

    'C'.mLet {
        it
    }

    // 万能类型，任何类型，所有类型，都可以使用我的 mLet
    // 省略几万行代码 ...

    val r2 : String = "Derry2".let {
        it
        34543.45f
        "Derry"
    }
}

// private 私有化
// inline  我们的函数是高阶函数，所以用到内联，做lambda的优化，性能提高
// fun<I, O> 在函数中，申明两个泛型，函数泛型  I输入Input， O输出Output
// I.mLet 对I输入Input进行函数扩展，扩展函数的名称是 mLet，意味着，所有的类型，万能类型，都可以用 xxx.mLet
// lambda : (I) -> O   (I输入参数) -> O输出
//  : O  会根据用户的返回类型，变化而变化
// lambda(this) I进行函数扩展，在整个扩展函数里面，this == I本身
private inline fun<I, O> I.mLet(lambda : (I) -> O) : O = lambda(this)
```
---
###  117.Kotlin语言的扩展属性
```kotlin

// 你必须把前面的普通方式学会：
val myStr : String = "AAA"
/* 背后代码：
   public final class KtBase117Kt {

       @NotNull
       private static final String myStr = "AAA";

       @NotNull
       public static final String getMyStr() {
            return myStr;
       }
   }
 */

// 扩展属性：
val String.myInfo: String
    get() = "Derry"

/* 背后代码：

   public final class KtBase117Kt {

       真实情况：
       @NotNull
       public static final String getMyInfo(@NotNull String $this$myInfo) {
          Intrinsics.checkParameterIsNotNull($this$myInfo, "$this$myInfo");
          return "Derry";
       }
   }

 */

// 打印输出 并且 链式调用 (只有String有资格这样)
fun String.showPrintln() : String {
    println("打印输出 并且 链式调用 (只有String有资格这样)：内容$this")
    return this
}

val String.stringAllInfoValueVal
    get() = "当前${System.currentTimeMillis()}这个时间点被调用了一次，当前的值是:$this，当前字符串长度是:$length"

// TODO 117-Kotlin语言的扩展属性
fun main() {
    val str : String = "ABC"
    println(str.myInfo)

    str.showPrintln().showPrintln().showPrintln().showPrintln()

    str.myInfo.showPrintln().showPrintln().showPrintln()

    "Derry老师".stringAllInfoValueVal // 扩展属性
        .showPrintln().showPrintln().showPrintln().showPrintln() // 扩展函数

}
```
---
###  118.Kotlin语言的可空类型扩展函数
```kotlin

// 对 String?==可空类型的 进行函数扩展，并且有备用值
fun String?.outputStringValueFun(defalutValue : String) = println(this ?: defalutValue)

// 编译期非常智能：能够监测到你做了if判断（能够对你代码逻辑监测），就知道后续类型
fun String?.outputStringValueFunGet(defaultValue : String) = if (this == null) defaultValue else this

// TODO 118-Kotlin语言的可空类型扩展函数
// 如果是null，就输出默认值
fun main() {
    val infoValue : String ? = null // infoValue是可空的类型  String  String?==可空类型的
    infoValue.outputStringValueFun("我是默认值啊1")

    // String? 前面已经说过了，可以接收 可空数据  也可以接收 有值数据
    // String  前面已经说过了，只能接收 有值数据
    val name = "Derry"
    name.outputStringValueFun("我是默认值啊2")

    // >>>>>>>>>>>>>>
    println(infoValue.outputStringValueFunGet("我是默认值啊3"))
    println(name.outputStringValueFunGet("我是默认值啊4"))
}
```
---
###  119.Kotlin语言的infix关键字
```kotlin

// 自定义的中缀表达式 + 扩展函数 一起用的     使用者： "一".gogogo(1)  "一" gogogo 1
// 1.条件一  对第一个参数 C1.gogogo  函数扩展
// 2.条件二  需要在 括号(c2: C2) 参数里面，传递一个参数
private infix fun <C1, C2> C1.gogogo(c2: C2) {
    // 做很多的逻辑
    // ...
    // 省略几万行代码
    println("我们的中缀表达式，对一个参数的内容是:$this")
    println("我们的中缀表达式，对二个参数的内容是:$c2")
}

// TODO 119-Kotlin语言的infix关键字
// infix == 中缀表达式 可以简化我的代码
fun main() {
    // 下面是我们map自带的中缀表达式
    mapOf("零".to(0))

    mapOf("一" to 1)
    mapOf("二" to 2)
    mapOf("三" to 3)

    // 下面是我们自己写的中缀表达式
    123 gogogo '男'
    "Derry".gogogo('M')
    "Derry2" gogogo 'M'
}
```

```
---
###  120.Kotlin语言的定义扩展文件
```kotlin

// 1.扩展文件一般都是public，如果private外界无法使用
// 2.Iterable<E> 子类 set list 都可以用，所以用父类
// 3.本次扩展函数的作用是，随机取第一个元素返回
fun <E> Iterable<E>.randomItemValue() = this.shuffled().first()

fun <T> Iterable<T>.randomItemValuePrintln() = println(this.shuffled().first())


// 导入扩展文件
// 在工作中非常有用，可以把很多的扩展操作，写到某一个地方，到时候引入过来用，比较独立化
import com..s6.com..randomItemValue
import com..s6.com..randomItemValuePrintln

// TODO 120-Kotlin语言的定义扩展文件
fun main() {
    val list : List<String> = listOf("李元霸", "李连杰", "李小龙")
    val set : Set<Double> = setOf(545.5, 434.5, 656.6)

    // 如果不使用 扩展文件
    println(list.shuffled().first())
    println(set.shuffled().first())

    println()

    // 使用 扩展文件
    println(list.randomItemValue())
    println(set.randomItemValue())

    println()

    list.randomItemValuePrintln()
    set.randomItemValuePrintln()
}
```
---
###  121.Kotlin语言的重命名扩展学习
```kotlin

import com.s6.com.randomItemValue as g  // as g 重命名扩展操作
import com.s6.com.randomItemValuePrintln as p // as g 重命名扩展操作

// TODO 121-Kotlin语言的重命名扩展学习
fun main() {

    val list : List<String> = listOf("李元霸", "李连杰", "李小龙")
    val set : Set<Double> = setOf(545.5, 434.5, 656.6)

    // 使用 扩展文件
    println(list.g())
    println(set.g())

    println()

    list.p()
    set.p()
}
```
---
###  122.Kotlin语言的apply函数详解
```kotlin

import java.io.File

// TODO 122-Kotlin语言的apply函数详解
fun main() {
    val r : File = File("D:\\a.txt")
        .mApply {
            // 输入的是 this == File对象本身
            setReadable(true)
            setWritable(true)
            println("1 ${readLines()}")
        }.mApply {
            // 输入的是 this == File对象本身
            setReadable(true)
            setWritable(true)
            println("2 ${readLines()}")
        }.mApply {
            // 输入的是 this == File对象本身
            setReadable(true)
            setWritable(true)
            println("3 ${readLines()}")
        }
        // ... 省略

    /*123.mApply()
    'C'.mApply()*/

    // 省略几万行代码
    // ...

    println()

    val r2 : File = File("D:\\a.txt")
        .apply {
            // 输入的是 this == File对象本身
            setReadable(true)
            setWritable(true)
            println("1 ${readLines()}")
        }.apply {
            // 输入的是 this == File对象本身
            setReadable(true)
            setWritable(true)
            println("2 ${readLines()}")
        }.apply {
            // 输入的是 this == File对象本身
            setReadable(true)
            setWritable(true)
            println("3 ${readLines()}")
        }
    // ... 省略
}

// private私有
// inline 因为我们的函数是高阶函数，需要使用内联对 lambda进行优化处理，提高性能
// fun <INPUT> 函数中声明一个泛型
// INPUT.mApply 让所有的类型，都可以 xxx.myApply  泛型扩展
//  INPUT.() -> Unit 让我们的匿名函数里面持有 this ,在lambda里面不需要返回值，因为永远都是返回INPUT本身
// lambda(this) 默认就有this
// 返回this的目的是可以链式调用
private inline fun <INPUT> INPUT.mApply(lambda : INPUT.() -> Unit) : INPUT  {
    lambda() // 省略this
    return this
}
```

---
###  123.Kotlin语言的DSL学习
```kotlin

import java.io.File

class Context {

    val info = "我就是Derry"
    val name = "DDD"

    fun toast(str: String) = println("toast:$str")
}

inline fun Context.apply5(lambda: Context.(String) -> Unit): Context {
    lambda(info)
    return this
}

// >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
inline fun File.applyFile(action: (String, String?) -> Unit): File {
    setWritable(true)
    setReadable(true)
    action(name, readLines()[0])
    return this
}

// TODO 123-Kotlin语言的DSL学习
// DSL简介 所谓DSL领域专用语言(Domain Specified Language/ DSL)
fun main() {

    // 其实apply5函数，就是DSL编程范式，定义输入输出等规则：
    // 1.定义整个lambda规则标准，输入 必须是Context这个类，才有资格调用apply5函数，匿名函数里面持有it 和 this
    // 2.定义整个lambda规则标准，输出 我们会始终返回Context本身，所以你可以链式调用
    // 然后main函数就可以根据DSL编程方式标准规则，来写具体的实现，这就是DSL编程范式
    val context : Context = Context().apply5 {
        // it == String == "我就是Derry"
        println("我的it是:$it，我的this是:$this")
        toast("success")
        toast(it)
        toast(name)
        true
    }.apply5 { }.apply5 { }.apply5 { } // ...
    println()
    println("我始终是输出Context本身：" + context.info)

    println()

    // 其实applyFile函数，就是DSL编程范式，定义输入输出等规则：
    // 1.定义整个lambda规则标准，输入 必须是File类，才有资格调用applyFile函数，匿名函数里面持有 fileName，data
    // 2.定义整个lambda规则标准，输出 我们始终返回File对象本身，所以你可以链式调用
    // 然后main函数就可以根据DSL编程方式标准规则，来写具体的实现，这就是DSL编程范式
    val file: File = File("D:\\a.txt")
        .applyFile { fileName, data ->
            println("你的文件名是:$fileName, 你的文件里面的数据是:$data")
            println("你的文件名是:$fileName, 你的文件里面的数据是:$data")
            println("你的文件名是:$fileName, 你的文件里面的数据是:$data")
            true
        }.applyFile { a, b -> }.applyFile { a, b -> }.applyFile { a, b -> } // ...

    println("我始终是输出File本身：${file.name}")
}
```
---
###  124.Kotlin语言的变换函数-map
```kotlin

// TODO 124-Kotlin语言的变换函数-map
fun main() {
    val list = listOf("李元霸", "李连杰", "李小龙")
    // T T T  --->  新的集合(R, R, R)
    // 原理：就是把你 匿名函数 最后一行的返回值 加入一个新的集合，新集合的泛型是R，并且返回新集合
    val list2 : List<Int> = list.map {
        // it = T == 元素 == String
        "【$it】"
        88
    }
    println(list2)

    // 用途： 和 RxJava的思路，一模一样
    val list3 : List<String> = list.map {
        "姓名是:$it"
    }.map {
        "$it，文字的长度是:${it.length}"
    }.map {
        "【$it】"
    }
    for (s in list3) {
        print("$s  ")
    }

    println()

    list.map {
        "姓名是:$it"
    }.map {
        "$it，文字的长度是:${it.length}"
    }.map {
        "【$it】"
    }.map {
        print("$it  ")
    }
}
```
---
###  125.Kotlin语言的变换函数-flatMap
```kotlin

// TODO 125-Kotlin语言的变换函数-flatMap
// map {返回类型：T String Int Boolean Char ...  是把每一个元素（String）加入到新集合，最后返回新集合 List<String>}
// flatMap {返回类型：每一个元素 T 集合1 集合2 集合3 ... 是把每一个元素（集合）加入到新集合，最后返回新集合 List<List<String>> 最终内部会处理成List<String>}

// TODO flatMap 相当于 List<List<String>> 集合的集合，有嵌套关系

fun main() {
    val list : List<String> = listOf("李四", "王五", "赵六", "初七")

    val newList : List<String> = list.map {
        "你的姓名是:$it" // 每次返回一个 String
    }.map {
        "$it, 你文字的长度是:${it.length}" // 每次返回一个 String
    }.flatMap {
        listOf("$it 在学习C++", "$it 在学习Java", "$it 在学习Kotlin") // 每次返回一个集合，四次
    }
    println(newList)

    println()

    /*
    [你的姓名是:李四, 你文字的长度是:8 在学习C++,
     你的姓名是:李四, 你文字的长度是:8 在学习Java,
     你的姓名是:李四, 你文字的长度是:8 在学习Kotlin,

     你的姓名是:王五, 你文字的长度是:8 在学习C++,
     你的姓名是:王五, 你文字的长度是:8 在学习Java,
     你的姓名是:王五, 你文字的长度是:8 在学习Kotlin,

     你的姓名是:赵六, 你文字的长度是:8 在学习C++,
     你的姓名是:赵六, 你文字的长度是:8 在学习Java,
     你的姓名是:赵六, 你文字的长度是:8 在学习Kotlin,

     你的姓名是:初七, 你文字的长度是:8 在学习C++,
     你的姓名是:初七, 你文字的长度是:8 在学习Java,
     你的姓名是:初七, 你文字的长度是:8 在学习Kotlin]
     */

    val newList2 : List<String> = list.flatMap {
        listOf("$it 在学习C++", "$it 在学习Java", "$it 在学习Kotlin")
    }
    println(newList2)
    //[李四 在学习C++,
    // 李四 在学习Java,
    // 李四 在学习Kotlin,
    // 王五 在学习C++,
    // 王五 在学习Java,
    // 王五 在学习Kotlin,
    // 赵六 在学习C++,
    // 赵六 在学习Java,
    // 赵六 在学习Kotlin,
    // 初七 在学习C++,
    // 初七 在学习Java,
    // 初七 在学习Kotlin]

    // 原理：就是把你 匿名函数 最后一行的返回值(又是一个集合listOf(......)) 加入一个新的集合，新集合的泛型是R，并且返回新集合
}
```
---
###  126.Kotlin语言的过滤函数-filter
```kotlin

// TODO 126-Kotlin语言的过滤函数-filter
fun main() {
    // println("126-Kotlin语言的过滤函数-filter")

    val nameLists = listOf(
        listOf("黄晓明", "李连杰", "李小龙"),
        listOf("刘军", "李元霸", "刘明"),
        listOf("刘俊", "黄家驹", "黄飞鸿")
    )

    nameLists.map {
        // it ==  nameLists的元素 == listOf("黄晓明", "李连杰", "李小龙"),
        println(it)
    }

    println()

    nameLists.flatMap {
        // it ==  nameLists的元素 == listOf("黄晓明", "李连杰", "李小龙"),
        println(it)

        listOf("")
    }

    println()

    nameLists.flatMap {

        // 进来了 3次
        it -> it.filter {
            println("$it filter") // 进来了 9次
            true
            // false

            // 原理：filter {true,false}  true他会加入到新的集合 进行组装新集合 返回，  否则false，过滤掉，不加入，返回空集合
        }
    }.map {
        print("$it   ")
    }

    println()
    println()

    nameLists.map {
        it -> it.filter {
            true
        }
    }.map {
        print("$it   ")
    }

    println()
    println()

    nameLists.flatMap {
        it -> it.filter {
            true
        }
    }.map {
        print("$it   ")
    }

    println()
    println()

    // List<T> 返回给 map 后的效果： [黄晓明, 李连杰, 李小龙]   [刘军, 李元霸, 刘明]   [刘俊, 黄家驹, 黄飞鸿]
    // List<T> 返回给 flatMap 效果: 黄晓明   李连杰   李小龙   刘军   李元霸   刘明   刘俊   黄家驹   黄飞鸿

    nameLists.flatMap {
            it -> it.filter {
                it.contains('黄') // 包含 ‘黄’ true，否则是false
            }
    }.map {
        print("$it   ")
    }

    println()
    println()

    nameLists.flatMap {
            it -> it.filter {
        it.contains('刘') // 包含 ‘刘’ true，否则是false
    }
    }.map {
        print("$it   ")
    }

    println()
    println()

    nameLists.flatMap {
            it -> it.filter {
        it.contains('李') // 包含 ‘李’ true，否则是false
    }
    }.map {
        print("$it   ")
    }
}
```
---
###  127.Kotlin语言的合并函数-zip
```kotlin

// TODO 127-Kotlin语言的合并函数-zip
fun main() {
    val names = listOf("张三", "李四", "王五")
    val ages = listOf(20, 21, 22)

    // RxJava zip 合并操作符
    // KT 自带就有zip 合并操作

    // 原理：就是把 第一个集合 和 第二个集合 合并起来，创建新的集合，并返回
    //      创建新的集合(元素，元素，元素) 元素Pair(K, V)  K代替第一个集合的元素   V代替第二个集合的元素
    val zip : List<Pair<String, Int>> = names.zip(ages)
    println(zip)
    println(zip.toMap())
    println(zip.toMutableSet())
    println(zip.toMutableList())

    println()

    // 遍历
    zip.forEach {
        // it == Pair<String, Int>
        println("姓名是:${it.first}, 年龄是:${it.second}")
    }

    println()

    // map 普通方式
    zip.toMap().forEach { k, v ->
        println("姓名是:${k}, 年龄是:${v}")
    }

    println()

    // map 解构的方式
    zip.toMap().forEach { (k, v) ->
        println("姓名是:${k}, 年龄是:${v}")
    }

    println()

    zip.toMap().forEach {
        // it == Map的元素 每一个元素 有K和V，  Map.Entry<String, Int>
        // it == Pair<String, Int>
        println("姓名是:${it.key}, 年龄是:${it.value}")
    }
}
```
---

```java

public class KtBase128 {

    public static void main(String[] args) {
        // 1.定义name集合
        List<String> names = new ArrayList<>();
        names.add("Zhangsan");
        names.add("Lisi");
        names.add("Wangwu");

        // 2.定义age集合
        List<Integer> ages = new ArrayList<>();
        ages.add(20);
        ages.add(21);
        ages.add(22);

        // 3.合并以上两个集合
        Map<String, Integer> newMap = new HashMap<>();
        for (int i = 0; i < names.size(); i++) {
            newMap.put(names.get(i), ages.get(i));
        }

        // 4.给集合添加详细内容，方便输出
        List<String> showList = new ArrayList<>();
        for (Map.Entry<String, Integer> stringIntegerEntry : newMap.entrySet()) {
            String result = String.format("you name:%s, you age:%d",
                                           stringIntegerEntry.getKey(), stringIntegerEntry.getValue());
            showList.add(result);
        }

        // 5.输出最后的成果 结果
        for (int i = 0; i < showList.size(); i++) {
            System.out.println(showList.get(i));
        }
    }

}
```
---
###  128.Kotlin语言中使用函数式编程
```kotlin
// TODO 128-Kotlin语言中使用函数式编程
/*
fun main() {
    // 1.定义name集合
    val names = listOf("Zhangsan", "Lisi", "Wangwu")
    // 2.定义age集合
    val ages = listOf(20, 21, 22)
    // 3.合并以上两个集合
    // 4.给集合添加详细内容，方便输出
    // 5.输出最后的成果 结果
    names.zip(ages).toMap().map { "you name:${it.key}, you age:${it.value}" }.map { println(it) }
}
*/

fun main() {
    listOf("Zhangsan", "Lisi", "Wangwu").zip(listOf(20, 21, 22)).toMap().map { println("you name:${it.key}, you age:${it.value}") }
}

```
---
```java

public class KtBase129 {

    public String getInfo1() {
        return "Derry Info1";
    }

    public String getInfo2() {
        return null;
    }
}

```
---
###  129.Kotlin语言的互操作性与可空性
```kotlin

// TODO 129-Kotlin语言的互操作性与可空性
fun main() {
    // 下面是 Java 与 KT 交互 ，错误的案例
    println(KtBase129().info1.length)
    // println(KtBase129().info2.length) // 引发空指针异常

    // 下面是 Java 与 KT 交互 ，错误的案例
    // : String! Java 与 KT 交互的时候，Java给KT用的值，都是 : String! 这种类型
    val info1  = KtBase129().info1
    val info2 = KtBase129().info2
    println(info1.length)
    // println(info2.length) // 引发空指针异常


    // 下面是 Java 与 KT 交互 ，正确的案例1
    // : String! Java 与 KT 交互的时候，Java给KT用的值，都是 : String! 这种类型
    // 只要是看到有  String! 的类型，在使用的时候，必须 ?.xxx，这个是规则1 这个规则1不好，如果忘记写，就有风险
    val info1s  = KtBase129().info1
    val info2s = KtBase129().info2
    println(info1s?.length)
    println(info2s?.length)

    // 下面是 Java 与 KT 交互 ，正确的案例2 (推荐)
    // : String! Java 与 KT 交互的时候，Java给KT用的值，都是 : String! 这种类型
    // 只要是看到有  String! 的类型，在使用的时候，必须 : String? 来接收Java值，这个是规则2（直接限定你不会出错了）
    val info1ss : String?  = KtBase129().info1
    val info2ss : String? = KtBase129().info2
    println(info1ss?.length)
    println(info2ss?.length)
}
```
---
###  内置函数的总结：
```kotlin
/**
 * 内置函数的总结：
 *
 * TODO apply：info.apply
 * 1.apply函数返回类型，永远都是info本身 此条和 also 一模一样
 * 2.apply函数的 匿名函数里面持有的是this == info本身  此条和 run一模一样
 *
 * TODO let：集合.let
 * 1.let函数返回类型，是根据匿名函数最后一行的变化而变化  此条和 run 一模一样
 * 2.let函数的 匿名函数里面持有的是it == 集合本身  此条和 also 一模一样
 */

TODO run: str.run
1.run函数返回类型，是根据匿名函数最后一行的变化而变化  此条和 let一模一样
2.run函数的 匿名函数里面持有的是this == str本身     此条和 apply一模一样

TODO with with(str)   with和run基本上一样，只不过就是使用的时候不同
1.with函数返回类型，是根据匿名函数最后一行的变化而变化  此条和 let一模一样
2.with函数的 匿名函数里面持有的是this == str本身     此条和 apply一模一样

TODO also str.also
1.also函数返回类型，永远都是str本身  此条和 apply 一模一样
2.also函数的 匿名函数里面持有的是it == str  此条和 let 一模一样


Todo =================================   apply 与 also 是一个类别的 属于一类的    =================================
相同点：apply与also返回类型，是一样的： 他们永远都是返回info本身，匿名函数，最后一行无法作为返回值，不影响函数返回类型
不同点：匿名函数里面 apply{ 持有this setFilexxx() }  alos { 持有it it.setFilexxx() }
应用点：
       val file本身 = File("xx").apply { setFilexxx() ... }.apply { ... }.apply { ... } 链式调用
       val file本身 = FIle("xx").also { it.setFilexxx() ... }.also { ... }.also { ... } 链式调用

val info本身 = info.apply {
    this == info本身
    ...
    "Derry"
}.apply {
}.apply {
}
val info本身 = info.also {
    it == info本身
    ...
    true
}.also {
}.also {
}


Todo =================================   run 与 let 与 with 是一个类别的 属于一类的    =================================
相同点：run与let返回类型，是一样的，都会根据匿名函数最后一行返回类型而决定 run与let的返回类型（是根据匿名函数最后一行的变化而变化）
不同点：匿名函数里面 run持有this  let持有it
应用点：
      info.run { show("内容:$this") show("内容长度:$length") show("${if (this is String) 你是String类型 else 你不是String类型}") }
      info.let  { show("内容:it") show("内容长度:$it.length") show("${if (it is String) 你是String类型 else 你不是String类型}") }

val r : Boolean类型 = info.run {  this == info本身
    length

    545
    454.545
    534543.5f
    true // 最后一行
}

val r : Char类型 = info.let {  it == info本身
    it.length

    545
    454.545
    534543.5f
    true
    'A' // 最后一行
}

TODO >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> 下面是with

上面的 run 使用环节：  info.run {}
下面的 with 使用环节：  with(info) {}

相同点：with与run与let返回类型，是一样的，都会根据匿名函数最后一行返回类型而决定 run与let的返回类型（是根据匿名函数最后一行的变化而变化）
不同点：匿名函数里面 run持有this  let持有it  with持有this
应用点：
      info.run { show("内容:$this") show("内容长度:$length") show("${if (this is String) 你是String类型 else 你不是String类型}") }
      info.let  { show("内容:it") show("内容长度:$it.length") show("${if (it is String) 你是String类型 else 你不是String类型}") }
      with(info)  { show("内容:this") show("内容长度:$length") show("${if (this is String) 你是String类型 else 你不是String类型}") }


TODO =======================================  let 与 apply 内部源码原理分析

// 1. let的返回类型是 根据匿名函数的变化而变化（lambda的返回类型变化而变化）
// 2. 匿名函数里面持有的是 it == I == info本身
inline fun <I, O> I.let(lambda : (I) -> O) = lambda(this)

// 1. apply的返回类型是 永远都是I（所以你可以链式调用） （lambda的返回类型 无法变化，你写的是 Unit，并且 没有和lambda关联返回类型）
// 2. 匿名函数里面持有的是 this == I == info本身
inline fun <I> I.apply(lambda : I.() -> Unit) : I {
    lambda()
    return this
}

```