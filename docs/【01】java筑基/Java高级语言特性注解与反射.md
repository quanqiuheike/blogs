#### [Java反射机制及其应用](https://www.jianshu.com/p/7e3279f9729f)
#### [Java 的注解](https://www.jianshu.com/p/53d4962bbc39)

## 什么是注解
>An annotation is a form of metadata, that can be added to Java source code. Classes, methods, variables, parameters and packages may be annotated. Annotations have no direct effect on the operation of the code they annotate.

注解是一种`元数据`, 可以添加到`java`代码中`类、方法、变量、参数、包`都可以被注解，注解对注解的代码没有直接影响。之所以产生作用, 是对其解析后做了相应的`处理`。注解仅仅只是个标记罢了。
### 定义注解关键字：`@interface`
### 元注解
`java`内置的注解有`Override, Deprecated, SuppressWarnings`等, 作用相信大家都知道。

现在查看`Override`注解的源码
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```
发现`Override`注解上面有两个注解, 这就是`元注解`。 `元注解`就是用来`定义注解`的`注解`.其作用就是定义注解的作用范围, 使用在什么元素上等等。
元注解共有四种`@Retention, @Target, @Inherited, @Documented`
#### @Retention 保留的范围，默认值为`CLASS. `可选值有三种
* `SOURCE`, 只在`源码`中可用
* `CLASS`, 在`源码`和`字节码`中可用
* `RUNTIME`, 在`源码`、`字节码`、运行时`均可用。

其中, `@Retention`是定义`保留策略`, 直接决定了我们用何种方式`解析`， `SOUCE`级别的注解是用来标记的, 比如`Override, SuppressWarnings`. 我们真正使用的类型是`CLASS(编译时)`和`RUNTIME(运行时)`。
#### @Target 用来修饰哪些程序元素，
如 `TYPE, METHOD, CONSTRUCTOR, FIELD, PARAMETER`等，未标注则表示可修饰所有。
#### @Inherited 是否可以被继承，默认为`false`

#### @Documented 是否会保存到 `Javadoc `文档中


### 自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface TestAnnotation {
    String value();
    String[] value2() default "value2";
}
```
> 类型 参数名() default 默认值;

其中默认值是可选的, 可以定义, 也可以不定义.
### 处理运行时注解
`Retention`的值为`RUNTIME`时, 注解会保留到运行时, 因此使用反射来解析注解。
使用的注解就是上一步的`@TestAnnotation`。


## 先了解一下反射：
### 一、反射的概述
`JAVA`反射机制是在`运行状态`中，对于任意一个`类`，都能够知道这个`类`的所有 `属性和方法`；对于任意一个`对象`，都能够`调用`它的任意一个`方法和属性`；这种`动态`获取的信息以及动态调用对象的方法的功能称为`java`语言的反射机制。

`Java`程序可以加载一个运行时才得知名称的`class`，获悉其完整构造（但不包括`methods`定义），并生成其对象实体、或对其`fields`设值、或唤起其`methods`。

要想解剖一个类,必须先要获取到**该类** 的` 字节码文件对象`。而解剖使用的就是`Class`类中的方法.所以先要获取到每一个`字节码文件`对应的`Class类型的对象`。

使用的前提条件：必须先得到代表的字节码的`Class`，`Class`类用于表示`.class`文件（字节码）反射是框架设计的灵魂，以上的总结就是什么是反射

### 反射就是把`java`类中的各种成分映射成一个个的`Java对象`

例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。
     （其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）
如图是类的正常加载过程：反射的原理在于`class对象`。
加载的时候：`Class对象`的由来是将`class文件`读`入内存`，并为之创建一个`Class对象`。

在运行期间，如果我们要产生某个类的对象，`Java`虚拟机`(JVM)`会检查该类型的`Class`对象是否已被加载。如果没有被加载，`JVM`会根据类的名称找到`.class`文件并加载它。一旦某个类型的`Class`对象已被加载到内存，就可以用它来产生该类型的所有对象。
![image.png](_images/反射图片.webp)
![logo](https://docsify.js.org/_media/icon.svg ':size=100')
### 反射机制主要提供的功能
* 在运行时判断任意一个对象所属的类；
* 在运行时构造任意一个类的对象；
* 在运行时判断任意一个类所具有的成员变量和方法；
* 在运行时调用任意一个对象的方法；
### Java中创建对象大概有这几种方式：
* 1、使用new关键字：这是我们最常见的也是最简单的创建对象的方式

* 2、使用Clone的方法：无论何时我们调用一个对象的clone方法，JVM就会创建一个新的对象，将前面的对象的内容全部拷贝进去

* 3、使用反序列化：当我们序列化和反序列化一个对象，JVM会给我们创建一个单独的对象
* 4、反射
### 获取类的字节码：
* (1)、Class.forName("com.test.User"); 

* (2)、对象.getClass();

* (3)、类名.class;
### java中的Class中一些重要的方法
* `public Annotation[] getAnnotations () `获取这个类中所有注解
*  `getClassLoader()  `获取加载这个类的类加载器
*  `getDeclaredMethods() ` 获取这个类中的所有方法
*  `getReturnType()  `获取方法的返回类型
* `getParameterTypes()` 获取方法的传入参数类型* 
* `isAnnotation()` 测试这类是否是一个注解类
* `getDeclaredConstructors()` 获取所有的构造方法
* `getDeclaredMethod(String name, Class… parameterTypes)` 获取指定的构造方法（参数：参数类型`.class`）
* `getSuperclass()` 获取这个类的父类
* `getInterfaces() `获取这个类实现的所有接口
* `getFields()` 获取这个类中所有被`public`修饰的成员变量
* `getField(String name)` 获取指定名字的被`public`修饰的成员变量
* `newInstance()` 返回此`Class`所表示的类，通过调用默认的（即无参数）构造函数创建的一个新实例
### 注意：
在反射私有的构造函数时，用普通的`clazz.getConstructor（）`会报错，因为它是私有的，所以提供了专门反射私有构造函数的方法。
//读取私有的构造函数，用这个方法读取完还需要设置一下暴力反射才可以
* ` clazz.getDeclaredConstructor(int.class);`
//暴力反射
* `c.setAccessible(true)`;



### User类
```java
package com.test;

public class User {
    private int id;
    private String name;
    private String age;
    public User() {
    }
    public User(int id) {
        this.id = id;
    }
    private User(String name) {
        this.name = name;
    }
    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }
    public User(String name, String age) {
        this.name = name;
        this.age = age;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    private void setIdName(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

### 实例化反射对象 构造方法
`constructor.setAccessible（true）`是获取到`反射的权限`，如果构造方法为`私有`的则需要设置为`true`， 参数值为`true`，打开禁用访问控制检查，`setAccessible(true) `并不是将方法的访问权限改成了`public`，而是取消`java`的权限控制检查。所以即使是`public`方法，其`accessible `属相默认也是`false`

```java
            /***
             * 采用默认构造方法实例化对象
             * com.test.User想反射类的全路径
             */
            Class mClass = Class.forName("com.test.User");
            //得到User类的构造方法，可以创建对象
            Constructor constructor = mClass.getConstructor();
            Object mStu = constructor.newInstance();
            /***
             * 采用参数构造
             */
            Class mClass2 = Class.forName("com.test.User");
            Constructor constructor2 = mClass2.getConstructor( int.class,String.class);
            constructor2.setAccessible(true);
            Object object2 = constructor2.newInstance(10,"xx1");
            /***
             * 调用private的有参构造方法
             */
            Class mClass3 = Class.forName("com.test.User");
            Constructor constructor3 = mClass3.getDeclaredConstructor(String.class);
            //获取到反射的权限，如果构造方法为私有的则需要设置为true。
            constructor3.setAccessible(true);
            Object object3 = constructor3.newInstance("xx2");

```
### 反射调用方法
```java
            /***
             * 调用无参的方法
             */
            Method nameMethod = mClass.getDeclaredMethod("getName");
            nameMethod.setAccessible(true);
            nameMethod.invoke(object1);
            /***
             * 反射调用setName方法
             */
            Method getNameMethod = mClass.getDeclaredMethod("setName", String.class);
            getNameMethod.invoke(object3, "一个参数");

            /***
             * 反射调用setIdName
             */
            Method setNameAndAgeMethod = mClass.getDeclaredMethod("setIdName", int.class, String.class);
            setNameAndAgeMethod.setAccessible(true);
            setNameAndAgeMethod.invoke(object2,  8,  "2个参数");
```
### 反射设置属性的值
```java
    Field nameField = mClass.getDeclaredField("name");
    nameField.setAccessible(true);
    nameField.set(object3, "xxxx");
```

### Hook
```java
package com.cc.reflection;


import android.view.View;
import android.view.ViewGroup;
import android.widget.Toast;

import java.lang.reflect.Field;
import java.lang.reflect.Method;


public class HookClickListenerUtils {

    private static HookClickListenerUtils mHookClickListenerUtils;

    private HookClickListenerUtils() {
    }

    public static HookClickListenerUtils getInstance() {
        synchronized ("getInstance") {
            if (mHookClickListenerUtils == null) {
                mHookClickListenerUtils = new HookClickListenerUtils();
            }
        }
        return mHookClickListenerUtils;
    }

    /***
     * 递归调用
     * @param decorView
     */
    public void hookDecorViewClick(View decorView) {
        if (decorView instanceof ViewGroup) {
            int count = ((ViewGroup) decorView).getChildCount();
            for (int i = 0; i < count; i++) {
                if (((ViewGroup) decorView).getChildAt(i) instanceof ViewGroup) {
                    hookDecorViewClick(((ViewGroup) decorView).getChildAt(i));
                } else {
                    hookViewClick(((ViewGroup) decorView).getChildAt(i));
                }
            }
        } else {
            hookViewClick(decorView);
        }
    }

    public void hookViewClick(View view) {
        try {
            view.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {

                }
            });
            Class viewClass = Class.forName("android.view.View");
            Method getListenerInfoMethod = viewClass.getDeclaredMethod("getListenerInfo");
            if (!getListenerInfoMethod.isAccessible()) {
                getListenerInfoMethod.setAccessible(true);
            }
            // 反射view中的getListenerInfo方法
            Object listenerInfoObject = getListenerInfoMethod.invoke(view);

            // 第二步：获取到view中的ListenerInfo中的mOnClickListener属性
            Class mListenerInfoClass = Class.forName("android.view.View$ListenerInfo");
            Field mOnClickListenerField = mListenerInfoClass.getDeclaredField("mOnClickListener");
            mOnClickListenerField.setAccessible(true);
            // 通过Java反射机制操作成员变量, set 和 get,View.OnClickListener为Field.get()的值
            mOnClickListenerField.set(listenerInfoObject, new HookClickListener((View.OnClickListener) mOnClickListenerField.get(listenerInfoObject)));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static class HookClickListener implements View.OnClickListener {

        private View.OnClickListener onClickListener;

        public HookClickListener(View.OnClickListener onClickListener) {
            this.onClickListener = onClickListener;
        }
        @Override
        public void onClick(View v) {
            Toast.makeText(v.getContext(), "hook住点击事件了，禽兽", Toast.LENGTH_SHORT).show();
            if (onClickListener != null) {
                onClickListener.onClick(v);
            }
        }
    }

}

```
#### [Java基础之—反射（非常重要）](https://blog.csdn.net/sinat_38259539/article/details/71799078)
#### [夯实JAVA基本之二 —— 反射（1）：基本类周边信息获取](https://blog.csdn.net/harvic880925/article/details/50072739)
#### [Java中的反射机制介绍](https://blog.csdn.net/ju_362204801/article/details/90578678)
#### [android的hook技术之hook所有view的监听](https://blog.csdn.net/zhongwn/article/details/54381337)
#### [学习java应该如何理解反射？](https://www.zhihu.com/question/24304289)
#### [java反射之Method的invoke方法实现](https://blog.csdn.net/wenyuan65/article/details/81145900)
#### [Android反射机制:手把手教你实现反射](https://blog.csdn.net/kai_zone/article/details/80217219)
#### [Android反射](https://www.jianshu.com/p/90757ca4af8f)
