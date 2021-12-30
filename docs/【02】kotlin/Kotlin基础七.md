
---
###   130.Kotlin语言的单例模式
```kotlin
// TODO 130.Kotlin语言的单例模式
// 1.饿汉式的实现  Java版本   ----  KT版本
// 2.懒汉式的实现  Java版本   ----  KT版本
// 3.懒汉式的实现 安全  Java版本  ----  KT版本
// 4.懒汉式的实现 双重校验安全  Java版本  ----  KT版本

// 1.饿汉式的实现  Java版本
public class SingletonDemo {

    private static SingletonDemo instance =  new SingletonDemo();

    private SingletonDemo() {}

    public static SingletonDemo getInstance() {
        return instance;
    }
}

// 2.懒汉式的实现  Java版本
public class SingletonDemo2 {

    private static SingletonDemo2 instance;

    private SingletonDemo2() {}

    public static SingletonDemo2 getInstance() {
        if (instance == null) {
            instance = new SingletonDemo2();
        }
        return instance;
    }

    public void show() {
        System.out.println("show");
    }

    public static void main(String[] args) {
        SingletonDemo2.getInstance().show();
    }
}


// 2.懒汉式的实现  KT版本
class SingletonDemo2Kt {

    companion object {

        private var instance : SingletonDemo2Kt? = null
            get() {
                if (field == null) {
                    field = SingletonDemo2Kt()
                }
                return field
            }

        fun getInstanceAction() = instance!!
    }

    fun show() {
        println("show")
    }
}

fun main() {
    SingletonDemo2Kt.getInstanceAction().show()
}


// 3.懒汉式的实现  Java版本 安全
public class SingletonDemo3 {

    private static SingletonDemo3 instance;

    private SingletonDemo3() {}

    public static synchronized SingletonDemo3  getInstance() {
        if (instance == null) {
            instance = new SingletonDemo3();
        }
        return instance;
    }

    public void show() {
        System.out.println("show");
    }

    public static void main(String[] args) {
        SingletonDemo3.getInstance().show();
    }
}


// 3.懒汉式的实现  KT版本 安全
class SingletonDemo3Kt {

    companion object {

        private var instance : SingletonDemo2Kt? = null
            get() {
                if (field == null) {
                    field = SingletonDemo2Kt()
                }
                return field
            }

        @Synchronized
        fun getInstanceAction() = instance!!
    }

    fun show() {
        println("show")
    }
}


// 4.懒汉式的实现 双重校验安全  Java版本
public class SingletonDemo4 {

    private volatile static SingletonDemo4 instance;

    private SingletonDemo4() {}

    public static SingletonDemo4 getInstance() {
        if (instance == null) {
            synchronized (SingletonDemo4.class) {
                instance = new SingletonDemo4();
            }
        }
        return instance;
    }

    public void show() {
        System.out.println("show");
    }

    public static void main(String[] args) {
        SingletonDemo4.getInstance().show();
    }
}


// 4.懒汉式的实现 双重校验安全  KT版本
class SingletonDemo4Kt private constructor() {

    companion object {
        val instance : SingletonDemo4Kt by lazy (mode = LazyThreadSafetyMode.SYNCHRONIZED) { SingletonDemo4Kt() }
    }

    fun show() {
        println("show")
    }
}

fun main() {
    SingletonDemo4Kt.instance.show()
}



// 1.饿汉式的实现  KT版本
object SingletonDemoKt


```
---
```java
public class KtBase131 {

    public static void main(String[] args) {
        // KtBase131Kt.getStudentNameValueInfo("Derry is OK");

        Stu.getStudentNameValueInfo("Derry is OK a");
    }
}

```
---
###  131.注解@JvmName与Kotlin
```kotlin

package com.s7

// @file:JvmName("Stu") 注意：必须写在 包名的外面

// TODO 131-注解@JvmName与Kotlin

fun getStudentNameValueInfo(str : String) = println(str)

fun main() {}

/* 背后生成的代码：

    public final class KtBase131Kt {

        public static final void getStudentNameValueInfo(@NotNull String str) {
            System.out.println(str);
        }

        public static final void main() {
        }

        public static void main(String[] args) {
            main();
        }
    }


    @file:JvmName("Stu") 背后的原理
    public final class Stu {

        public static final void getStudentNameValueInfo(@NotNull String str) {
            System.out.println(str);
        }

        public static final void main() {
        }

        public static void main(String[] args) {
            main();
        }
    }
 */
```
---
```java

public class KtBase132 {

    public static void main(String[] args) {
        Person person = new Person();
        for (String name : person.names) {
            System.out.println(name);
        }
    }
}

```
---
###  132.注解@JvmField与Kotlin
```kotlin

// TODO 132.注解@JvmField与Kotlin
class Person {
    @JvmField
    val names = listOf("Zhangsan", "Lisi", "Wangwu")
}

/* 背后的原理代码：

    public final class Person{

        @NotNull
        private final List names = CollectionsKt.listOf(new String[]{"Zhangsan", "Lisi", "Wangwu"});

        // val 只读的，只有 getNames
        public final List getNames() {
            return this.names;
        }
    }



    @JvmField 背后会剔除私有代码 成员
    public final class Person {
       @JvmField
       @NotNull
       public final List names = CollectionsKt.listOf(new String[]{"Zhangsan", "Lisi", "Wangwu"});
    }
 */
```
---
```java

public class KtBase133 {

    public static void main(String[] args) {
        // Java端
        // KtBase133Kt.show("张三") // Java无法享用 KT的默认参数

        KtBase133Kt.toast("张三"); // 相当于 Java 享用 KT的默认参数
    }
}

```
---
###   133.注解@JvmOverloads与Kotlin
```kotlin
// 默认参数
fun show( name : String, age : Int = 20, sex : Char = 'M') {
    println("name:$name, age:$age, sex:$sex")
}

// 默认参数
@JvmOverloads // 原理：编译器环节 专门重载一个函数，专门给 Java用
fun toast( name : String, sex : Char = 'M') {
    println("name:$name, sex:$sex")
}

// TODO 133-注解@JvmOverloads与Kotlin

fun main() {
    // KT端
    show("张三")
    toast("李四")
}
```
---
```java

public class KtBase134 {

    public static void main(String[] args) {
        // Java 端
        System.out.println(MyObject.TARGET);

        MyObject.showAction("Kevin");
    }
}

```
---
###  134.注解@JvmStatic与Kotlin关系
```kotlin
class MyObject {

    companion object {

        @JvmField
        val TARGET = "黄石公园"

        @JvmStatic
        fun showAction(name: String) = println("$name 要去 $TARGET 玩")
    }

}

// TODO 134-注解@JvmStatic与Kotlin关系
fun main() {
    // KT 端
    MyObject.TARGET

    MyObject.showAction("Derry")
}
```
---
```kotlin


// 手写RxJava，全部用KT的基础来写
fun main() {
    // create 输入源，没有任何参数给你，  输出源：你是输出就行（所有类型，万能类型）
    // map 输入源 就是create的输出源的valueItem，  输出源：你是输出就行（所有类型，万能类型）
    // observer 输入源 就是 map 存储的 valueItem，  消费完成就行，全部结束

    create {
        // .... 省略 ，万能类型，几百种类型，其实都一样的
        "Derry"
        123
        true
        "AAAAAAAA"
        5435.54f // 最后一行
    }.map {
        "你的值是:$this" // 最后一行
    }.map {
        "[$this]"
    }.map {
        "@@$this@@"
    }.observer {
        // 只需要把上面输入的内容，打印输出即可，所以不需要管输出
        println(this)
    }
}

// 中转站，保存我们的记录  // valueItem == create操作符 最后一行的返回值 流向此处了
class RxJavaCoreClassObject<T>(var valueItem : T) // 主构造，接收你传递进来的信息，此消息就是create最后一行的返回

inline fun <I> RxJavaCoreClassObject<I>.observer ( observerAction : I.() -> Unit ) = observerAction(valueItem)

inline fun<I, O> RxJavaCoreClassObject<I>.map(mapAction : I.() -> O) = RxJavaCoreClassObject(mapAction(valueItem))

inline fun <OUTPUT> create(action : () -> OUTPUT) = RxJavaCoreClassObject((action()))

/**
 *   140 个视频 Derry主动给大家增加的录制的，送给大家的礼物
 *   NDK主动增加两个月课
 *   ...
 */
```
---