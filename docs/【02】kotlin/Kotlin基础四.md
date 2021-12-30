
---
### 57.Kotlin语言的List创建与元素获取学习
```kotlin

// TODO 57.Kotlin语言的List创建与元素获取学习
// 普通取值方式：    索引
// 防止奔溃取值方式： getOrElse getOrNull
fun main() {
    val list = listOf("Derry", "Zhangsan", "Lisi", "Wangwu")

    // 普通取值方式：    索引  内部是运算符重载 [] == get
    println(list[0])
    println(list[1])
    println(list[2])
    println(list[3])
    // println(list[4]) // 奔溃  java.lang.ArrayIndexOutOfBoundsException: 4

    println()

    // 我们写KT代码，一定不会再出现，空指针异常，下标越界异常
    // 防止奔溃取值方式： getOrElse getOrNull
    println(list.getOrElse(3) {"越界"})
    println(list.getOrElse(4) {"你越界了"})
    println(list.getOrElse(4402) {"你越界了啊"})

    println()

    println(list.getOrNull(1))
    println(list.getOrNull(4))
    println(list.getOrNull(111))
    // getOrNull + 空合并操作符
    println(list.getOrNull(222) ?: "你越界了哦哦")

    // 小结：开发过程中，尽量使用 getOrElse 或 getOrNull，才能体现KT的亮点
}
```
---
### 58.Kotlin语言的可变List集合学习
```kotlin

// TODO 58.Kotlin语言的可变List集合学习
// 可变的集合
// 不可变集合 to 可变集合
// 可变集合 to 不可变集合
fun main() {
    // 可变的集合
    val list = mutableListOf("Derry", "Zhangsna", "Wangwu")
    // 可变的集合，可以完成可变的操作
    list.add("赵六")
    list.remove("Wangwu")
    println(list)

    // 不可变集合 to 可变集合
    val list2 = listOf(123, 456, 789)
    // 不可以的集合，无法完成可变的操作
    // list2.add
    // list2.remove

    val list3 : MutableList<Int> = list2.toMutableList()
    // 可变的集合，可以完成可变的操作
    list3.add(111)
    list3.remove(123)
    println(list3)

    // 可变集合 to 不可变集合
    val list4: MutableList<Char> = mutableListOf('A', 'B', 'C')
    // 可变的集合，可以完成可变的操作
    list4.add('Z')
    list4.remove('A')
    println(list4)

    val list5: List<Char> = list4.toList()
    // 不可以的集合，无法完成可变的操作
    /*list5.add
    list5.remove*/
}
```
---
### 59.Kotlin语言的mutator函数学习
```kotlin

// TODO 59.Kotlin语言的mutator函数学习
// 1.mutator += -= 操作
// 2.removeIf
fun main() {
    // 1.mutator += -= 操作
    val list : MutableList<String> = mutableListOf("Derry", "DerryAll", "DerryStr", "Zhangsan")
    list += "李四" // mutator的特性 +=  -+ 其实背后就是 运算符重载而已
    list += "王五"
    list -= "Derry"
    println(list)

    // 2.removeIf
    // list.removeIf { true } // 如果是true 自动变量整个可变集合，进行一个元素一个元素的删除
    list.removeIf { it.contains("Derr") } // 过滤所有的元素，只要是有 Derr 的元素，就是true 删除
    println(list)
}
```
---
### 60.Kotlin语言的List集合遍历学习
```kotlin

// TODO  60.Kotlin语言的List集合遍历学习
fun main() {
    val list = listOf(1, 2, 3, 4, 5, 6, 7)

    println(list) // 输出list详情而已，这个不是遍历集合

    // 第一种 遍历方式：
    for (i in list) {
        print("元素:$i  ")
    }

    println()

    // 第二种 遍历方式：
    list.forEach {
        // it == 每一个元素
        print("元素:$it  ")
    }

    println()

    // 第三种 遍历方式：
    list.forEachIndexed { index, item ->
        print("下标:$index, 元素:$item    ")
    }
}
```

---
###  61.Kotlin语言的解构语法过滤元素学习
```kotlin

// TODO 61.Kotlin语言的解构语法过滤元素学习
// 1.集合配合解构语法
// 2.反编译看Java给三个变量赋值的代码
// 3.解构屏蔽接收值
fun main() {
    val list: List<String> = listOf("李元霸", "李小龙", "李连杰")

    val(value1, value2, value3) = list
    // value1 = ""  val只读的
    println("value1:$value1, value2:$value2, value3:$value3")

    var(v1, v2, v3) = list
    // v1 = "OK"
    println("v1:$v1, v2:$v2, v3:$v3")

    // 用_内部可以不接收赋值，可以节约一点性能
    val(_ , n2, n3) = list
    // println(_) _不是变量名，是用来过滤解构赋值的，不接收赋值给我
    println("n2:$n2, n3:$n3")
}
```
---
###  62.Kotlin语言的Set创建与元素获取
```kotlin

// TODO 62.Kotlin语言的Set创建与元素获取
// 1.set 定义 不允许重复
// 2.普通方式elementAt 会越界奔溃
// 3.elementAtOrElse elementAtOrNull
fun main() {
    val set: Set<String> = setOf("lisi", "wangwu", "zhaoliu", "zhaoliu") // set集合不会出现重复元素的
    println(set)
    // set[0] 没有这样 [] 的功能 去Set集合元素
    println(set.elementAt(0)) // [0]
    println(set.elementAt(1))
    println(set.elementAt(2))
    // println(set.elementAt(3)) // [3] 奔溃 会越界
    // println(set.elementAt(4)) // [4] 奔溃 会越界

    println()

    // 使用 list 或 set 集合，尽量使用  此方式，防止越界奔溃异常
    println(set.elementAtOrElse(0) {"越界了"})
    println(set.elementAtOrElse(100) {"越界了了"})

    println(set.elementAtOrNull(0))
    println(set.elementAtOrNull(111))
    // OrNull + 空合并操作符  一起使用
    println(set.elementAtOrNull(88) ?: "你越界啦啊")
}
```
---
### 63.Kotlin语言的可变Set集合
```kotlin

// TODO 63.Kotlin语言的可变Set集合
fun main() {
   val set : MutableSet<String> = mutableSetOf("李元霸", "李连杰")
    set += "李俊"
    set += "李天"
    set -= "李连杰"
    set.add("刘军")
    set.remove("刘军")
    println(set)
}
```
---
### 64.Kotlin语言的集合转换与快捷函数
```kotlin

// TODO 64.Kotlin语言的集合转换与快捷函数学习
// 1.定义可变list集合
// 2.List 转 Set 去重
// 3.List 转 Set 转 List 也能去重
// 4.快捷函数去重 distinct
fun main() {
   val list : MutableList<String> = mutableListOf("Derry", "Derry", "Derry", "Leo", "Lance") // list 可以重复元素
    println(list)

    // List 转 Set 去重
    val set /*: Set<String>*/ = list.toSet()
    println(set)

    // List 转 Set 转 List 也能去重
    val list2 /*: List<String>*/ = list.toSet().toList()
    println(list2)

    // 快捷函数去重 distinct
    println(list.distinct()) // 内部做了：先转变成 可变的Set结合  在转换成 List集合
    println(list.toMutableSet().toList()) // 和上面代码等价
}
```
---
### 65.Kotlin中的数组类型
```kotlin

import java.io.File

// TODO 65.Kotlin中的数组类型
/*
    Kotlin语言中的各种数组类型，虽然是引用类型，背后可以编译成Java基本数据类型
    IntArray        intArrayOf
    DoubleArray     doubleArrayOf
    LongArray       longArrayOf
    ShortArray      shortArrayOf
    ByteArray       byteArrayOf
    FloatArray      floatArrayOf
    BooleanArray    booleanArrayOf
    Array<对象类型>           arrayOf         对象数组
*/
// 1.intArrayOf 常规操作的越界奔溃
// 2.elementAtOrElse elementAtOrNull
// 3.List集合转 数组
// 4.arrayOf Array<File>
fun main() {
    // 1.intArrayOf 常规操作的越界奔溃
    val intArray /*: IntArray*/ = intArrayOf(1, 2, 3, 4, 5)
    println(intArray[0])
    println(intArray[1])
    println(intArray[2])
    println(intArray[3])
    println(intArray[4])
    // println(intArray[5]) // 奔溃：会越界异常

    println()

    // 2.elementAtOrElse elementAtOrNull
    println(intArray.elementAtOrElse(0) { -1 })
    println(intArray.elementAtOrElse(100) { -1 })

    println(intArray.elementAtOrNull(0))
    println(intArray.elementAtOrNull(200))

    // OrNull + 空合并操作符 一起来用
    println(intArray.elementAtOrNull(666) ?: "你越界啦啊啊啊")

    println()

    // 3.List集合转 数组
    val charArray /*: CharArray*/ = listOf('A', 'B', 'C').toCharArray()
    println(charArray)

    // 4.arrayOf Array<File>
    val objArray /*: Array<File>*/ = arrayOf(File("AAA"), File("BBB"), File("CCC"))
}
```
---
### 66.Kotlin语言的Map的创建
```kotlin

// TODO 66.Kotlin语言的Map的创建
fun main() {
    val mMap1 : Map<String, Double> = mapOf<String, Double>("Derry" to(534.4), "Kevin" to 454.5)
    val mMap2 = mapOf(Pair("Derry", 545.4), Pair("Kevin", 664.4))
    // 上面两种方式是等价的哦
}
```
---
### 67.Kotlin语言的读取Map的值
```kotlin
// TODO 67.Kotlin语言的读取Map的值
// 方式一 [] 找不到会返回null
// 方式二 getOrDefault
// 方式三 getOrElse
// 方式四 与Java一样 会奔溃
fun main() {
    val mMap /*: Map<String, Int>*/ = mapOf("Derry" to 123,"Kevin" to 654)

    // 方式一 [] 找不到会返回null
    println(mMap["Derry"]) // 背后对[] 运算符重载了
    println(mMap["Kevin"])
    println(mMap.get("Kevin")) // get 与 [] 完完全全等价的
    println(mMap["XXX"]) // map通过key找 如果找不到返回null，不会奔溃

    println()

    // 方式二 getOrDefault
    println(mMap.getOrDefault("Derry", -1))
    println(mMap.getOrDefault("Derry2", -1))

    // 方式三 getOrElse
    println(mMap.getOrElse("Derry") {-1})
    println(mMap.getOrElse("Derry2") {-1})

    println()

    // 方式四 getValue 与Java一样 会奔溃  尽量不要使用此方式
    println(mMap.getValue("Derry"))
    println(mMap.getValue("XXX"))

    // map获取内容，尽量使用 方式二 方式三
}
```
---
###  68.Kotlin语言的遍历Map学习
```kotlin
// TODO 68.Kotlin语言的遍历Map学习
// 四种方式遍历
fun main() {
    val map /*: Map<String, Int>*/ = mapOf(Pair("Derry", 123), Pair("Kevin", 456), "Leo" to 789)

    // 第一种方式:
    map.forEach {
        // it 内容 每一个元素 (K 和 V)  每一个元素 (K 和 V)  每一个元素 (K 和 V)
        // it 类型  Map.Entry<String, Int>
        println("K:${it.key} V:${it.value}")
    }

    println()

    // 第二种方式：
    map.forEach { key: String, value: Int ->
        // 把默认的it给覆盖了
        println("key:$key, value:$value")
    }

    println()

    // 第三种方式：
    map.forEach { (k /*: String*/, v /*: Int*/) ->
        println("key:$k, value:$v")
    }

    println()

    // 第四种方式：
    for (item /*: Map.Entry<String, Int>*/ in map) {
        // item 内容 每一个元素 (K 和 V)  每一个元素 (K 和 V)  每一个元素 (K 和 V)
        println("key:${item.key} value:${item.value}")
    }

}
```

---
### 69.Kotlin语言的可变Map集合学习
```kotlin
// TODO 69.Kotlin语言的可变Map集合学习
// 1.可变集合的操作 += [] put
// 2.getOrPut 没有的情况
// 3.getOrPut 有的情况
fun main() {
    // 1.可变集合的操作 += [] put
    val map : MutableMap<String, Int> = mutableMapOf(Pair("Derry", 123), "Kevin" to 456, Pair("Dee", 789))
    // 下面是可变操作
    map += "AAA" to(111)
    map += "BBB" to 1234
    map -= "Kevin"
    map["CCC"] = 888
    map.put("DDD", 999) // put 和 [] 等价的

    // 2.getOrPut 没有有的情况
    // 如果整个map集合里面没有 FFF的key 元素，我就帮你先添加到map集合中去，然后再从map集合中获取
    val r: Int = map.getOrPut("FFF") { 555 }
    println(r)
    println(map["FFF"]) // 他已经帮你加入进去了，所以你可以获取

    // 3.getOrPut 有的情况
    val r2 = map.getOrPut("Derry") {666} // 发现Derry的key是有的，那么就直接获取出来， 相当于666备用值就失效了
    println(r2)
}
```
---
### 70.Kotlin语言的定义类与field关键字学习
```kotlin
class KtBase70 {
    var name = "Derry"
        get() = field
        set(value) {
            field = value
        }
    /* 背后做的事情：

       @NotNull
       private String name = "Derry";

       public void setName( @NotNull String name) {
            this.name = name;
       }

       @NotNull
       public String getName() {
            return this.name;
       }

     */

    var value = "ABCDEFG"
        // 下面的隐式代码，不写也有，就是下面这个样子
        get() = field
        set(value) {
            field = value
        }

    var info = "abcdefg ok is success"
        get() = field.capitalize() // 把首字母修改成大写
        set(value) {
            field = "**【$value】**"
        }
    /* 背后做的事情：

        @NotNull
        private String info = "abcdefg ok is success";

        public void setInfo( @NotNull String info) {
            this.info = "**【" + info + "】**";
        }

        @NotNull
        public String getInfo() {
            return StringKt.capitalize(this.info)
        }

     */
}

// public final class KtBase70Kt { 下面的main函数 }

// TODO 70.Kotlin语言的定义类与field关键字学习
fun main() {
    // 背后隐式代码：new KtBase70().setName("Kevin");
    KtBase70().name = "Kevin"
    // 背后隐式代码：System.out.println(new KtBase70().getName());
    println(KtBase70().name)


    println(">>>>>>>>>>>>>>>>>>")


    // 背后隐式代码：System.out.println(new KtBase70().getInfo());
    println(KtBase70().info)

    // 背后隐式代码：new KtBase70().setInfo("学习KT");
    KtBase70().info = "学习KT"
}
```
---
### 71.Kotlin语言的 计算属性 与 防范竞态条件
```kotlin

class KtBase71 {
    val number : Int = 0
    /* 背后的代码：

       private int number = 0;

       public int getNumber() {
            return this.number;
       }

     */

    // 计算属性  下面这样写 get函数覆盖了 field 内容本身，相当于field失效了，无用了，以后用不到了
    val number2 : Int
        get() = (1..1000).shuffled().first() // 从1到1000取出随机值 返回给 getNumber2()函数
    /*
        背后隐式代码：

        为什么没有看到 number2 属性定义？
        答：因为属于 计算属性 的功能，根本在getNumber2函数里面，就没有用到 number2属性，所以 number2属性 失效了，无用了，以后用不到了

         public int getNumber2() {
            return (1..1000).shuffled().first()java的随机逻辑 复杂 ;
       }
     */

    var info: String ? = null // ""

    // 防范竞态条件  当你调用成员，这个成员，可能为null，可能为空值，就必须采用 防范竞态条件，这个是KT编程的规范化
    fun getShowInfo() : String {

        // 这个成员，可能为null，可能为空值，就启用 防范竞态条件
        // 这种写法，就属于 防范竞态条件，我们可以看到专业的KT开发者，有大量这种代码
        // also永远都是返回 info本身
        return info?.let {
            if (it.isBlank()) {
                "info你原来是空值，请检查代码..." // 是根据匿名函数最后一行的变化而变化
            } else {
                "最终info结果是:$it" // 是根据匿名函数最后一行的变化而变化
            }
        } ?: "info你原来是null，请检查代码..."
    }
}

// TODO 71.Kotlin语言的 计算属性 与 防范竞态条件
fun main() {
    // 背后隐式代码：System.out.println(new KtBase71().getNumber());
    println(KtBase71().number)

    // 背后隐式代码：new KtBase71().setNumber(9);
    // KtBase71().number = 9 // val 根本就没有 setXXX函数，只有 getXXX函数

    // 背后隐式代码：System.out.println(new KtBase71().getNumber2());
    println(KtBase71().number2)

    // 背后隐式代码：System.out.println(new KtBase71().getShowInfo());
    println(KtBase71().getShowInfo())
}
```
---
###  72.Kotlin语言的主构造函数学习
```kotlin
// 主构造函数：规范来说，都是增加_xxx的方式，临时的输入类型，不能直接用，需要接收下来 成为变量才能用
// _name 等等，都是临时的类型，不能直接要弄，需要转化一下才能用
class KtBase72(_name: String, _sex: Char, _age: Int, _info: String) // 主构造函数
{
    var name = _name
        get() = field // get不允许私有化
        private set(value) {
            field = value
        }
    val sex = _sex
        get() = field
        // set(value) {} 只读的，不能修改的，不能set函数定义

    val age: Int = _age
        get() = field + 1

    val info = _info
        get() = "【${field}】"

    fun show() {
        // println(_name) 临时的输入类型，不能直接用，需要接收下来 成为变量才能用
        println(name)
        println(sex)
        println(age)
        println(info)
    }
}

// TODO 72.Kotlin语言的主构造函数学习
fun main() {
    // KtBase72()
    var s=MutableList();

    val p = KtBase72(_name = "Zhangsan", _info = "学习KT语言", _age = 88, _sex = 'M')
    // println(p.name)
    // p.name = "AAA" 被私有化了，不能调用

    p.show()
}
```

---
### 73.Kotlin语言的主构造函数里定义属性
```kotlin

/* 伪代码：

class KtBase73 (_name: String, _sex: Char, _age: Int, _info: String)
{
    var name = _name
    val sex = _sex
    val age = _age
    var info = _info

    fun show() {
        println(name)
    }
}

 */

// var name: String  就相当于  var name = _name  这不过你看不到而已
// 一步到位，不像我们上一篇是分开写的
class KtBase73 (var name: String, val sex: Char, val age: Int, var info: String)
{
    fun show() {
        println(name)
        println(sex)
        println(age)
        println(info)
    }
}

// TODO 73.Kotlin语言的主构造函数里定义属性
fun main() {
    KtBase73(name = "Zhangsan", info = "学习KT语言", age = 88, sex = 'M').show()
}
```
---
### 74.Kotlin语言的次构造函数学习
```kotlin

class KtBase74(name: String) // 主构造
{
    // 2个参数的次构造函数，必须要调用主构造函数，否则不通过，  为什么次构造必须调用主构造？答：主构造统一管理 为了更好的初始化设计
    constructor(name: String, sex: Char) : this(name) {
        println("2个参数的次构造函数 name:$name, sex:$sex")
    }

    // 3个参数的次构造函数，必须要调用主构造函数
    constructor(name: String, sex: Char, age: Int) : this(name) {
        println("3个参数的次构造函数 name:$name, sex:$sex, age:$age")
    }

    // 4个参数的次构造函数，必须要调用主构造函数
    constructor(name: String, sex: Char, age: Int, info: String) : this(name) {
        println("4个参数的次构造函数 name:$name, sex:$sex, age:$age, info:$info")
    }
}

// TODO 74.Kotlin语言的次构造函数学习
// name: String, sex: Char, age: Int, info: String
fun main() {
    val p = KtBase74("李元霸") // 调用主构造

    KtBase74("张三", '男') // 调用 2个参数的次构造函数

    KtBase74("张三2", '男', 88) // 调用 3个参数的次构造函数

    KtBase74("张三3", '男', 78, "还在学校新语言") // 调用 4个参数的次构造函数
}
```
---
### 75.Kotlin语言的构造函数中默认参数学习
```kotlin

class KtBase75(name: String = "李元霸") // 主构造
{
    // 2个参数的次构造函数，必须要调用主构造函数
    constructor(name: String = "李连杰", sex: Char = 'M') : this(name) {
        println("2个参数的次构造函数 name:$name, sex:$sex")
    }

    // 3个参数的次构造函数，必须要调用主构造函数
    constructor(name: String = "李小龙", sex: Char = 'M', age: Int = 33) : this(name) {
        println("3个参数的次构造函数 name:$name, sex:$sex, age:$age")
    }

    // 4个参数的次构造函数，必须要调用主构造函数
    constructor(name: String = "李俊", sex: Char = 'W', age: Int = 87, info: String = "还在学校新开发语言") : this(name) {
        println("4个参数的次构造函数 name:$name, sex:$sex, age:$age, info:$info")
    }
}

// TODO 75.Kotlin语言的构造函数中默认参数学习
fun main() {
    val p = KtBase75("李元霸2") // 调用主构造

    KtBase75("张三", '男') // 调用 2个参数的次构造函数

    KtBase75("张三2", '男', 88) // 调用 3个参数的次构造函数

    KtBase75("张三3", '男', 78, "还在学校新语言") // 调用 4个参数的次构造函数

    KtBase75() // 到底是调用哪一个 构造函数，是次构造 还是 主构造 ？ 答：优先调用主构造函数
}
```
---
### 76.Kotlin语言的初始化块学习
```kotlin

/*class A {

    {

    }

    A() {

    }

}*/

// username: String, userage: Int, usersex: Char  临时类型，必须要二次转换，才能用
class KtBase76 (username: String, userage: Int, usersex: Char) // 主构造
{
    // 这个不是Java的 static{}
    // 相当于是Java的 {} 构造代码块
    // 初始化块  init代码块
    init {
        println("主构造函数被调用了 $username, $userage, $usersex")

        // 如果第一个参数是false，就会调用第二个参数的lambda
        // 判断name是不是空值 isNotBlank   ""
        require(username.isNotBlank()) { "你的username空空如也，异常抛出" }

        require(userage > 0) { "你的userage年龄不符合，异常抛出" }

        require( usersex == '男' || usersex == '女') { "你的性别很奇怪了，异常抛出" }
    }

    constructor(username: String) : this(username, 87, '男') {
        println("次构造函数被调用了")
    }

    fun show() {
        // println(username) // 用不了，必须要二次转换，才能用
    }
}

// TODO 76.Kotlin语言的初始化块学习
// 1.name,age,sex的主构造函数
// 2.init代码块学习 require
// 3.临时类型只有在 init代码块才能调用
fun main() {
    KtBase76("李四", userage = 88, usersex = '女')  // 调用主构造
    println()
    KtBase76("王五") // 调用次构造

    // KtBase76("") // 调用次构造

    // KtBase76("李四", userage = -1, usersex = 'M')  // 调用主构造

    KtBase76("李四", userage = 1, usersex = '男')  // 调用主构造
}
```
---
### 77.Kotlin语言的构造初始化顺序学习
```kotlin

// 第一步：生成val sex: Char
class KtBase77(_name: String, val sex: Char) // 主构造
{

    // 第二步： 生成val mName  // 由于你是写在 init代码块前面，所以先生成你， 其实类成员 和 init代码块 是同时生成
    val mName = _name

    init {
        val nameValue = _name // 第三步：生成nameValue细节
        println("init代码块打印:nameValue:$nameValue")
    }

    // 次构造 三个参数的  必须调用主构造
    constructor(name: String, sex: Char, age: Int) :this(name, sex) {
        // 第五步：生成次构造的细节
        println("次构造 三个参数的, name:$name, sex:$sex, age:$age")
    }

    // 第四步
    val = "AAA"

    // 纠正网上优秀博客的错误： 类成员先初始生成   再init代码块初始生成  错误了
    // Derry正确说法：init代码块 和 类成员 是同时的，只不过你写在 init代码块前面 就是先生成你
}

// TODO 77.Kotlin语言的构造初始化顺序学习
// 1.main 调用次构造 name sex age
// 2.主构造的 val 变量
// 3.var mName = _name
// 4.init { nameValue 打印 }
fun main() {
    // 调用次构造
    KtBase77("李元霸", '男', 88)  // 调用次构造
}
```
---
### 78.Kotlin语言的延迟初始化lateinit学习
```kotlin

class KtBase78 {

    // lateinit val AAA; // AAA 无法后面在修改了，我还怎么延时初始化？
    lateinit var responseResultInfo: String // 我等会儿再来初始化你，我先定义再说，所以没有赋值

    // 模拟服务器加载
    fun loadRequest() { // 延时初始化，属于懒加载，用到你在给你加载
        responseResultInfo = "服务器加载成功，恭喜你"
    }

    fun showResponseResult() {
        // 由于你没有给他初始化，所以只有用到它，就奔溃
        // if (responseResultInfo == null) println()
        // println("responseResultInfo:$responseResultInfo")

        if (::responseResultInfo.isInitialized) {
            println("responseResultInfo:$responseResultInfo")
        } else {
            println("你都没有初始化加载，你是不是忘记加载了")
        }
    }
}

// TODO 78.Kotlin语言的延迟初始化lateinit学习
// 1.lateinit responseResultInfo 定义
// 2.request 懒加载
// 3.showResponseResult
// 4.main 先请求 在显示
fun main() {
    val p = KtBase78()

    // 使用他之前，加载一下（用到它才加载，就属于，懒加载）
    p.loadRequest()

    // 使用他
    p.showResponseResult()
}
```
---
### 79.Kotlin语言的惰性初始化by lazy学习
```kotlin

class KtBase79 {

    // >>>>>>>>>>>>>>>>>>> 下面是 不使用惰性初始化 by lazy  普通方式(饿汉式 没有任何懒加载的特点)
    // val databaseData1 = readSQlServerDatabaseAction()

    // >>>>>>>>>>>>>>>>>>> 使用惰性初始化 by lazy  普通方式
    val databaseData2 by lazy { readSQlServerDatabaseAction() }

    private fun readSQlServerDatabaseAction(): String {
        println("开始读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("加载读取数据库数据中....")
        println("结束读取数据库数据中....")
        return "database data load success ok."
    }

}

// TODO 79.Kotlin语言的惰性初始化by lazy学习
// 1.不使用惰性初始化 databaseData1 = readSQLServerDatabaseAction()
// 2.使用惰性初始化  databaseData2 by lazy
// 3.KtBase82()  睡眠  db1.databaseData1

// lateinit 是在使用的时候，手动加载的懒加载方式，然后再使用
// 惰性初始化by lazy  是在使用的时候，自动加载的懒加载方式，然后再使用
fun main() {
    // >>>>>>>>>>>>>>>>>>> 下面是 不使用惰性初始化 by lazy  普通方式(饿汉式 没有任何懒加载的特点)
    /*val p = KtBase79()

    Thread.sleep(5000)

    println("即将开始使用")

    println("最终显示:${p.databaseData1}")*/


    // >>>>>>>>>>>>>>>>>>> 使用惰性初始化 by lazy  普通方式
    val p = KtBase79()

    Thread.sleep(5000)

    println("即将开始使用")

    println("最终显示:${p.databaseData2}")

}
```
---
###  80.Kotlin语言的初始化陷阱一学习
```kotlin

class KtBase80 {

    var number = 9

    init {
        number = number.times(9)
    }

}

// TODO 80.Kotlin语言的初始化陷阱一学习
fun main() {
    println(KtBase80().number)
}
```
---
###  81.Kotlin语言的初始化陷阱二学习
```kotlin

class KtBase81 {

    val info: String

    init {
        info = "DerryOK"
        getInfoMethod()
        // info = "DerryOK"
    }

    fun getInfoMethod() {
        println("info:${info[0]}")
    }

}

// TODO 81.Kotlin语言的初始化陷阱二学习
// 1.定义info
// 2.init getInfo
// 3.info = ""
fun main() {
    KtBase81().


        getInfoMethod()
}
```
---
###  82.Kotlin语言的.初始化陷阱三学习
```kotlin

class KtBase82 (_info: String) {
    private val info = _info

    val content : String = getInfoMethod()

    // private val info = _info // 把这种 转换info的代码，写到最前面，这样保证，就不会出现这种问题

    private fun getInfoMethod() = info // 当此函数调用info变量的时候，你以为是赋值好了，但是还没有赋值
}

// TODO 82.Kotlin语言的.初始化陷阱三学习
// 1.主构造 _info 放后面
// 2.value = initInfoActin() 放前面
// 3.p.value.length
fun main() {
    println("内容的长度是:${KtBase82("Derry").content.length}")
}
```