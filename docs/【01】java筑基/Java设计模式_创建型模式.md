

# 创建型模式分为以下几种。

- **单例（Singleton）模式：**某个类只能生成一个实例，该类提供了一个全局访问点供外部获取该实例，其拓展是有限多例模式。
- **原型（Prototype）模式**：将一个对象作为原型，通过对其进行复制而克隆出多个和原型类似的新实例。
- **工厂方法（FactoryMethod）模式**：定义一个用于创建产品的接口，由子类决定生产什么产品。
- **抽象工厂（AbstractFactory）模式**：提供一个创建产品族的接口，其每个子类可以生产一系列相关的产品。
- **建造者（Builder）模式**：将一个复杂对象分解成多个相对简单的部分，然后根据不同需要分别创建它们，最后构建成该复杂对象。

## 一、单例模式 （Singleton）

> 单例模式主要是为了避免因为创建了多个实例造成资源的浪费，且多个实例由于多次调用容易导致结果出现错误，而**使用单例模式能够保证整个应用中有且只有一个实例**。

| 定义                                           | 对比定义                            |
| ---------------------------------------------- | ----------------------------------- |
| (1) 不允许其他程序用new对象                    | (1) 私有化该类的构造函数            |
| (2) 在该类中创建对象                           | (2) 通过new在本类中创建一个本类对象 |
| (3) 对外提供一个可以让其他程序获取该对象的方法 | (2) 通过new在本类中创建一个本类对象 |



### 1.饿汉式[可用]：[SingletonEHan.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/singleton/ehan/SingletonEHan.java)

```java
package com.cc.design.singleton;

/**
 * 一、创建模式-单例模式1-饿汉式[可用]
 * (1)私有化该类的构造函数
 * (2)通过new在本类中创建一个本类对象
 * (3)定义一个公有的方法，将在该类中所创建的对象返回
 * <p>
 * 优点：从它的实现中我们可以看到，这种方式的实现比较简单，在类加载的时候就完成了实例化，避免了线程的同步问题。
 * 缺点：由于在类加载的时候就实例化了，所以没有达到Lazy Loading(懒加载)的效果，也就是说可能我没有用到这个实例，但是它
 * 也会加载，会造成内存的浪费(但是这个浪费可以忽略，所以这种方式也是推荐使用的)。
 */
public class SingletonEHan {
    private SingletonEHan() {
    }

    /**
     * 1.单例模式的饿汉式[可用]
     */
    private static SingletonEHan singletonEHan = new SingletonEHan();

    public static SingletonEHan getInstance() {
        return singletonEHan;
    }

    // 调用方式
//    SingletonEHan instance = SingletonEHan.getInstance();

    /**
     * 2. 单例模式的饿汉式变换写法[可用]
     * 基本没区别
     */
    private static SingletonEHan singletonEHanTwo = null;

    static {
        singletonEHanTwo = new SingletonEHan();
    }

    public static SingletonEHan getSingletonEHan() {
        if (singletonEHanTwo == null) {
            singletonEHanTwo = new SingletonEHan();
        }
        return singletonEHanTwo;
    }
// 调用方式
//     SingletonEHan instance= SingletonEHan.getSingletonEHan();

}
```

### 2.懒汉式[双重校验锁 推荐用]：[SingletonLanHan.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/singleton/lanhan/SingletonLanHan.java)

```java
package com.cc.design.singleton;

/**
 * 一、创建模式-单例模式2-懒汉式
 */
public class SingletonLanHan {

    private SingletonLanHan() {
    }

    /**
     * 3.单例模式的懒汉式[线程不安全，不可用]
     */
    private static SingletonLanHan singletonLanHan;

    public static SingletonLanHan getInstance() {
        // 这里线程是不安全的,可能得到两个不同的实例，DCL模式，双重判空对象锁
        if (singletonLanHan == null) {
            singletonLanHan = new SingletonLanHan();
        }
        return singletonLanHan;
    }

    /**
     * 4. 懒汉式线程安全的:[线程安全，效率低不推荐使用]
     * <p>
     * 缺点：效率太低了，每个线程在想获得类的实例时候，执行getInstance()方法都要进行同步。
     * 而其实这个方法只执行一次实例化代码就够了，
     * 后面的想获得该类实例，直接return就行了。方法进行同步效率太低要改进。
     */
    private static SingletonLanHan singletonLanHanTwo;

    public static synchronized SingletonLanHan getSingletonLanHanTwo() {
        // 这里线程是不安全的,可能得到两个不同的实例
        if (singletonLanHanTwo == null) {
            singletonLanHanTwo = new SingletonLanHan();
        }
        return singletonLanHanTwo;
    }

    /**
     * 5. 单例模式懒汉式[线程不安全，不可用]
     * <p>
     * 虽然加了锁，但是等到第一个线程执行完instance=new Singleton()跳出这个锁时，
     * 另一个进入if语句的线程同样会实例化另外一个Singleton对象，
     * 线程不安全的原理跟3类似。
     */
    private static SingletonLanHan instanceThree = null;

    public static SingletonLanHan getSingletonLanHanThree() {
        if (instanceThree == null) {
            synchronized (SingletonLanHan.class) {// 线程不安全
                instanceThree = new SingletonLanHan();
            }
        }
        return instanceThree;
    }

    /**
     * 6.单例模式懒汉式双重校验锁[推荐用]
     * 懒汉式变种,属于懒汉式的最好写法,保证了:延迟加载和线程安全
     */
    private static SingletonLanHan singletonLanHanFour;

    public static SingletonLanHan getSingletonLanHanFour() {
        //提升效率
        if (singletonLanHanFour == null) {
            synchronized (SingletonLanHan.class) {
                //安全校验
                if (singletonLanHanFour == null) {
                    singletonLanHanFour = new SingletonLanHan();
                }
            }
        }
        return singletonLanHanFour;
    }
}
```

### 3.单例模式-内部类[推荐用]：[SingletonIn.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/singleton/inclass/SingletonIn.java)

```java
package com.cc.design.singleton;

/**
 * 一、创建模式-单例模式3-内部类[推荐用]
 * <p>
 * 这种方式跟饿汉式方式采用的机制类似，但又有不同。
 * 两者都是采用了类装载的机制来保证初始化实例时只有一个线程。
 * 不同的地方:
 * 在饿汉式方式是只要Singleton类被装载就会实例化,
 * 内部类是在需要实例化时，调用getInstance方法，才会装载SingletonHolder类
 * <p>
 * 优点：避免了线程不安全，延迟加载，效率高。
 */

public class SingletonIn {
    private SingletonIn() {
    }

    private static class SingletonInHodler {
        private static SingletonIn singletonIn = new SingletonIn();
    }

    //7. 内部类[推荐用]
    public static SingletonIn getSingletonIn() {
        return SingletonInHodler.singletonIn;
    }
}

```

### 4.单例模式枚举[推荐用]：[SingletonEnum.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/singleton/enums/SingletonEnum.java)

```java
package com.cc.design.singleton;

/**
 * 一、创建模式-单例模式4-枚举[极推荐使用]
 * 这里SingletonEnum.instance
 * 这里的instance即为SingletonEnum类型的引用所以得到它就可以调用枚举中的方法了。
 * 借助JDK1.5中添加的枚举来实现单例模式。不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象
 */

public enum SingletonEnum {
    //8. 枚举[极推荐使用]
    instance;

    private SingletonEnum() {
    }

    public void whateverMethod() {

    }

    // SingletonEnum.instance.method();

}

```

## 二、工厂模式

### 1.简单工厂模式静态工厂模式

这个最常见了，项目中的辅助类，TextUtil.isEmpty等，类+静态方法。

```java
import com.cc.design.entity.Fruit;
import com.cc.design.entity.fruit.Apple;
import com.cc.design.entity.fruit.Banana;
import com.cc.design.entity.fruit.Orange;

/**
 * 简单工厂模式 --- 静态工厂模式
 */
public class StaticFactory {
    public static final int TYPE_APPLE = 1;//苹果
    public static final int TYPE_ORANGE = 2;//桔子
    public static final int TYPE_BANANA = 3;//香蕉

    public static Fruit getFruit(int type){
        if(TYPE_APPLE == type){
            return new Apple();
        } else if(TYPE_ORANGE == type){
           return new Orange("cheng",80);
        } else if(TYPE_BANANA == type){
            return new Banana();
        }
        return null;
    }

    /**
     * 多方法工厂
     * @return
     */
    public static Fruit getFruitApple(){
        return new Apple();
    }

    public static Fruit getFruitOrange(){
        return new Orange("cheng",80);
    }

    public static Fruit getFruitBanana(){
        return new Banana();
    }
}



/**
 * 吃苹果
 */
public class StaticFactoryClient {


    public static void main(String[] args) {
//        peterdo();
        jamesdo();
//        lisondo();
    }

    //自己吃水果
    public static void peterdo() {
        Fruit fruit = StaticFactory.getFruit(StaticFactory.TYPE_BANANA);
        fruit.draw();
        //。。。直接啃着吃，吃掉了
        System.out.println("-----------------");
    }

    //送给james，切开吃
    public static void jamesdo() {
        Fruit fruit = StaticFactory.getFruitBanana();
        fruit.draw();
        //。。。切开吃
        System.out.println("-----------------");
    }

    //送给lison榨汁喝
    public static void lisondo() {
        Fruit fruit = StaticFactory.getFruit(StaticFactory.TYPE_APPLE);
        fruit.draw();
        //。。。榨汁动作
        System.out.println("-----------------");
    }
}

package com.cc.design.create.factory.jdgc;

/**
 * 静态工厂模式
 */
public class Client {
    public static void main(String[] args) {
        SimpleFactory.makeProduct(Const.PRODUCT_A);
    }
    //抽象产品
    public interface Product {
        void show();
    }

    //具体产品：ProductA
    static class ConcreteProduct1 implements Product {
        @Override
        public void show() {
            System.out.println("具体产品1显示...");
        }
    }

    //具体产品：ProductB
    static class ConcreteProduct2 implements Product {
        @Override
        public void show() {
            System.out.println("具体产品2显示...");
        }
    }

    final class Const {
        static final int PRODUCT_A = 0;
        static final int PRODUCT_B = 1;
        static final int PRODUCT_C = 2;
    }

    static class SimpleFactory {
        public static Product makeProduct(int kind) {
            switch (kind) {
                case Const.PRODUCT_A:
                    return new ConcreteProduct1();
                case Const.PRODUCT_B:
                    return new ConcreteProduct2();
            }
            return null;
        }
    }
}

```

### 2.简单工厂模式

- 定义：通过专门定义一个类来负责创建其他类的实例，被创建的实例通常都具有共同的父类。
- 根据类型直接创建肉夹馍：[SimpleRoujiaMoFactory.java](https://github.com/youlookwhat/DesignPattern/blob/master/app/src/main/java/com/example/jingbin/designpattern/factory/jdgc/SimpleRoujiaMoFactory.java)

```java

import com.cc.design.entity.Fruit;
import com.cc.design.entity.fruit.Apple;
import com.cc.design.entity.fruit.Banana;
import com.cc.design.entity.fruit.Orange;

/**
 * 简单工厂模式 --- 静态工厂模式
 */
public class StaticFactory {
    public static final int TYPE_APPLE = 1;//苹果
    public static final int TYPE_ORANGE = 2;//桔子
    public static final int TYPE_BANANA = 3;//香蕉

    public static Fruit getFruit(int type){
        if(TYPE_APPLE == type){
            return new Apple();
        } else if(TYPE_ORANGE == type){
           return new Orange("cheng",80);
        } else if(TYPE_BANANA == type){
            return new Banana();
        }
        return null;
    }

    /**
     * 多方法工厂
     * @return
     */
    public static Fruit getFruitApple(){
        return new Apple();
    }

    public static Fruit getFruitOrange(){
        return new Orange("cheng",80);
    }

    public static Fruit getFruitBanana(){
        return new Banana();
    }
}



/**
 * 吃苹果
 */
public class StaticFactoryClient {


    public static void main(String[] args) {
//        peterdo();
        jamesdo();
//        lisondo();
    }

    //自己吃水果
    public static void peterdo() {
        Fruit fruit = StaticFactory.getFruit(StaticFactory.TYPE_BANANA);
        fruit.draw();
        //。。。直接啃着吃，吃掉了
        System.out.println("-----------------");
    }

    //送给james，切开吃
    public static void jamesdo() {
        Fruit fruit = StaticFactory.getFruitBanana();
        fruit.draw();
        //。。。切开吃
        System.out.println("-----------------");
    }

    //送给lison榨汁喝
    public static void lisondo() {
        Fruit fruit = StaticFactory.getFruit(StaticFactory.TYPE_APPLE);
        fruit.draw();
        //。。。榨汁动作
        System.out.println("-----------------");
    }
}


package com.cc.design.create.factory.simple;

/**
 * 静态工厂模式
 */
public class SimpleStaticClient {
    public static void main(String[] args) {
        SimpleFactory.makeProduct(Const.PRODUCT_A);
    }
    //抽象产品
    public interface Product {
        void show();
    }

    //具体产品：ProductA
    static class ConcreteProduct1 implements Product {
        @Override
        public void show() {
            System.out.println("具体产品1显示...");
        }
    }

    //具体产品：ProductB
    static class ConcreteProduct2 implements Product {
        @Override
        public void show() {
            System.out.println("具体产品2显示...");
        }
    }

    final class Const {
        static final int PRODUCT_A = 0;
        static final int PRODUCT_B = 1;
        static final int PRODUCT_C = 2;
    }

    static class SimpleFactory {
        public static Product makeProduct(int kind) {
            switch (kind) {
                case Const.PRODUCT_A:
                    return new ConcreteProduct1();
                case Const.PRODUCT_B:
                    return new ConcreteProduct2();
            }
            return null;
        }
    }
}
```

### 3.工厂方法模式

#### 3.1 模式的结构角色

抽象工厂（Abstract Factory）：提供了创建产品的接口，调用者通过它访问具体工厂的工厂方法 newProduct() 来创建产品。

具体工厂（ConcreteFactory）：主要是实现抽象工厂中的抽象方法，完成具体产品的创建。

抽象产品（Product）：定义了产品的规范，描述了产品的主要特性和功能。

具体产品（ConcreteProduct）：实现了抽象产品角色所定义的接口，由具体工厂来创建，它同具体工厂之间一一对应

#### 3.2 代码如下

```java
import com.cc.design.entity.Fruit;

/**
 * 工厂方法接口
 */
public interface FruitFactory {
    public Fruit getFruit();//摘水果指令
}

/**
 * 工厂方法模式
 */
public interface BagFactory {
    public Bag getBag();//打包指令
}

/**
 * 工厂方法模式
 */
public class AppleFactory implements FruitFactory{
    @Override
    public Fruit getFruit(){
        return new Apple();
    }
}


/**
 * 工厂方法模式
 */
public class BananaFactory implements FruitFactory{
    @Override
    public Fruit getFruit(){
        return new Banana();
    }
}


/**
 * 工厂方法模式
 */
public class OrangeFactory implements FruitFactory{
    @Override
    public Fruit getFruit(){
        return new Orange("Peter",80);
    }
}

/**
 * 工厂方法模式
 */
public class AppleBagFactory implements BagFactory{
    @Override
    public Bag getBag(){
        return new AppleBag();
    }
}

/**
 * 工厂方法模式
 */
public class BananaBagFactory implements BagFactory{
    @Override
    public Bag getBag(){
        return new BananaBag();
    }
}


public class OrangeBagFactory implements BagFactory{
    @Override
    public Bag getBag() {
        return new OrangeBag();
    }
}


/**
 * 工厂方法模式测试
 */
public class FactoryTest {

    @Autowired
    private static FruitFactory fruitFactory;

    public static void main(String[] args) {
        //初始化苹果工厂
//        fruitFactory = new AppleFactory();//spring配置

        peterdo();
        jamesdo();
        lisondo();
    }

    //Peter自己吃水果
    public static void peterdo(){
        Fruit fruit = fruitFactory.getFruit();
        fruit.draw();
        //。。。直接啃着吃，吃掉了
        System.out.println("-----------------");
    }

    //送给james，切开吃
    public static void jamesdo(){
        Fruit fruit = fruitFactory.getFruit();
        fruit.draw();
        //。。。切开吃
        System.out.println("-----------------");
    }

    //送给lison榨汁喝
    public static void lisondo(){
        Fruit fruit = fruitFactory.getFruit();
        fruit.draw();
        //。。。榨汁动作
        System.out.println("-----------------");
    }

}


/**
 * 水果店测试
 */
public class FruitStoreTest {

    private static FruitFactory fruitFactory;
    private static BagFactory bagFactory;

    public static void main(String[] args) {
        pack();
    }

    /**
     * 邮寄打包
     */
    public static void pack(){
        //初始化苹果工厂
        fruitFactory = new AppleFactory();//猎取工厂不对应
        Fruit fruit = fruitFactory.getFruit();
        fruit.draw();

        //初始化苹果包装工厂
        bagFactory = new BananaBagFactory();
        Bag bag = bagFactory.getBag();
        bag.pack();

      
        //....邮寄业务
    }
}


```



### 3.抽象工厂模式

```java

/**
 * 抽象水果工厂
 */
public abstract class AbstractFactory {

    public abstract Fruit getFruit();

    public abstract Bag getBag();

}

/**
 * 水果工厂
 */
public class AppleFactory extends AbstractFactory{

    @Override
    public Fruit getFruit() {
        return new Apple();
    }

    @Override
    public Bag getBag() {
        return new AppleBag();
    }
}

/**
 * 水果工厂
 */
public class BananaFactory extends AbstractFactory{

    @Override
    public Fruit getFruit() {
        return new Banana();
    }

    @Override
    public Bag getBag() {
        return new BananaBag();
    }
}
/**
 * 水果工厂
 */
public class OrangeFactory extends AbstractFactory{

    @Override
    public Fruit getFruit() {
        return new Orange("cheng",50);
    }

    @Override
    public Bag getBag() {
        return new OrangeBag();
    }
}

public class Const {
    public static final int PRODUCT_A=1;
    public static final int PRODUCT_B=2;

    public static final int PRODUCT_C=3;

    public static final int PRODUCT_D=4;

}


/**
 * 抽象工厂模式测试
 * 按订单发送货品给客户
 */
public class OrderSendClient {
    public static void main(String[] args) {
        sendFruit();
    }

    public static AbstractFactory makeProduct(int kind) {
        switch (kind) {
            case Const.PRODUCT_A:
                return new AppleFactory();
            case Const.PRODUCT_B:
                return new OrangeFactory();
        }
        return new AppleFactory();
    }

    public static void sendFruit() {
        //初始化工厂 使用注入方式
//        AbstractFactory factory = new AppleFactory();
        AbstractFactory factory = makeProduct(Const.PRODUCT_A);

        //得到水果
        Fruit fruit = factory.getFruit();
        fruit.draw();
        //得到包装
        Bag bag = factory.getBag();
        bag.pack();
        //以下物流运输业务。。。。

    }
}
```

## [三. 建造者模式](http://c.biancheng.net/view/1354.html)

> 建造模式是对象的创建模式。建造模式可以将一个产品的内部表象（internal representation）与产品的生产过程分割开来，从而可以使一个建造过程生成具有不同的内部表象的产品对象。

#### 1. 模式的结构

建造者（Builder）模式的主要角色如下。

1. 产品角色（Product）：它是包含多个组成部件的复杂对象，由具体建造者来创建其各个零部件。
2. 抽象建造者（Builder）：它是一个包含创建产品各个子部件的抽象方法的接口，通常还包含一个返回复杂产品的方法 getResult()。
3. 具体建造者(Concrete Builder）：实现 Builder 接口，完成复杂产品的各个部件的具体创建方法。
4. 指挥者（Director）：它调用建造者对象中的部件构造与装配方法完成复杂对象的创建，在指挥者中不涉及具体产品的信息。

#### 2.模式的实现

(1) 产品角色：包含多个组成部件的复杂对象。

```java
/**
 * (1) 产品角色：包含多个组成部件的复杂对象。
 */
public class Product {
    private String partA;
    private String partB;
    private String partC;
    public void setPartA(String partA) {
        this.partA = partA;
    }
    public void setPartB(String partB) {
        this.partB = partB;
    }
    public void setPartC(String partC) {
        this.partC = partC;
    }
    public void show() {
        //显示产品的特性
    }
}
```

(2) 抽象建造者：包含创建产品各个子部件的抽象方法。

```java
/**
 * (2) 抽象建造者：包含创建产品各个子部件的抽象方法。
 */
public abstract class Builder {
    //创建产品对象
    protected Product product = new Product();
    public abstract void buildPartA();
    public abstract void buildPartB();
    public abstract void buildPartC();
    //返回产品对象
    public Product getResult() {
        return product;
    }
}
```

(3) 具体建造者：实现了抽象建造者接口。

```java

/**
 * (3) 具体建造者：实现了抽象建造者接口。
 */
public class ConcreteBuilder extends Builder {
    @Override
    public void buildPartA() {
        product.setPartA("建造 PartA");
    }

    @Override
    public void buildPartB() {
        product.setPartB("建造 PartB");
    }

    @Override
    public void buildPartC() {
        product.setPartC("建造 PartC");
    }
}
```

(4) 指挥者：调用建造者中的方法完成复杂对象的创建。

```java
/**
 * (4) 指挥者：调用建造者中的方法完成复杂对象的创建。
 */
class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    //产品构建与组装方法
    public Product construct() {
        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();
        return builder.getResult();
    }
}
```

(5) 客户类。

```java
/**
 * (5) 客户类。
 */
public class Client {
    public static void main(String[] args) {
        Builder builder = new ConcreteBuilder();
        Director director = new Director(builder);
        Product product = director.construct();
        product.show();
    }
}
```



```java

/**
 * 创建一个MealBuilder类，实际的builder类负责创建套餐Meal对象。
 */
public interface Builder {//也是工厂

    void buildApple(int price);//设置苹果
    void buildBanana(int price);//设置香蕉
    void buildOrange(int price);//设置桔子

    FruitMeal getFruitMeal();//返回创建的套餐
}


/**
 * 创建一个水果套餐Meal类
 */
public class FruitMeal {
    //苹果--价格
    private Apple apple;
    //香蕉价格
    private Banana banana;
    //桔子价格
    private Orange orange;
    //折扣价
    private int discount;
    //套餐总价
    private int totalPrice;

    public void setDiscount(int discount) {
        this.discount = discount;
    }

    public void setApple(Apple apple) {
        this.apple = apple;
    }

    public void setBanana(Banana banana) {
        this.banana = banana;
    }

    public void setOrange(Orange orange) {
        this.orange = orange;
    }

    public int cost(){
        return this.totalPrice;
    }

    public void init() {
        if (null != apple){
            totalPrice += apple.price();
        }
        if (null != orange){
            totalPrice += orange.price();
        }
        if (null != banana){
            totalPrice += banana.price();
        }
        if (totalPrice > 0){
            totalPrice -= discount;
        }
    }

    public void showItems() {
        System.out.println("totalPrice：" + totalPrice);
    }
}

/**
 * 桔子
 */
public class HolidayBuilder implements Builder {
    private FruitMeal fruitMeal = new FruitMeal();

    @Override
    public void buildApple(int price) {
        Apple apple = new Apple();
        apple.setPrice(price);
        fruitMeal.setApple(apple);
    }

    @Override
    public void buildBanana(int price) {
        Banana fruit = new Banana();
        fruit.setPrice(price);
        fruitMeal.setBanana(fruit);
    }

    @Override
    public void buildOrange(int price) {
        Orange fruit = new Orange("cheng",80);
        fruit.setPrice(price);
        fruitMeal.setOrange(fruit);
    }

    @Override
    public FruitMeal getFruitMeal() {
        //折扣价格对一个套餐来，是固定的
        fruitMeal.setDiscount(15);
        fruitMeal.init();
        return fruitMeal;
    }
}


/**
 * 桔子
 */
public class OldCustomerBuilder implements Builder {
    private FruitMeal fruitMeal = new FruitMeal();

    @Override
    public void buildApple(int price) {
        Apple apple = new Apple();
        apple.setPrice(price);
        fruitMeal.setApple(apple);
    }

    @Override
    public void buildBanana(int price) {
        Banana fruit = new Banana();
        fruit.setPrice(price);
        fruitMeal.setBanana(fruit);
    }

    @Override
    public void buildOrange(int price) {
        Orange fruit = new Orange("Peter",80);
        fruit.setPrice(price);
        fruitMeal.setOrange(fruit);
    }

    @Override
    public FruitMeal getFruitMeal() {
        fruitMeal.setDiscount(10);
        fruitMeal.init();
        return fruitMeal;
    }
}


/**
 * 桔子
 */
public class FruitMealController {//收银台---导演类

    public void construct() {
//        Builder builder = new HolidayBuilder();
        //spring注入方法，
        Builder builder = new OldCustomerBuilder();

        //以下代码模板，轻易是不变的
        //创建苹果设置价格
        builder.buildApple(120);
        //创建香蕉设置香蕉价格
        builder.buildBanana(80);
        //创建桔子设置价格
        builder.buildOrange(50);

        FruitMeal fruitMeal = builder.getFruitMeal();


        int cost = fruitMeal.cost();
        System.out.println("本套件花费："+cost);
    }

    public static void main(String[] args) {

        new FruitMealController().construct();
    }

}

package com.cc.design.create.builder;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;
import androidx.databinding.DataBindingUtil;

import com.cc.design.R;
import com.cc.design.databinding.ActivityBuilderBinding;

/**
 * 建造者模式（Builder Pattern）
 * 建造模式是对象的创建模式。建造模式可以将一个产品的内部表象（internal representation）与产品的生产过程分割开来，
 * 从而可以使一个建造过程生成具有不同的内部表象的产品对象。
 * <p>
 * Builder 类是关键，然后定义一个Builder实现类，再之后就是处理实现类的逻辑。
 * <p>
 * 优点：
 * 1. 首先，建造者模式的封装性很好。使用建造者模式可以有效的封装变化，在使用建造者模式的场景中，
 * 一般产品类和建造者类是比较稳定的，因此，将主要的业务逻辑封装在导演类中对整体而言可以取得比较好的稳定性。
 * 2. 其次，建造者模式很容易进行扩展。如果有新的需求，通过实现一个新的建造者类就可以完成，
 * 基本上不用修改之前已经测试通过的代码，因此也就不会对原有功能引入风险。
 * 总结：
 * 建造者模式与工厂模式类似，他们都是建造者模式，适用的场景也很相似。
 * 一般来说，如果产品的建造很复杂，那么请用工厂模式；如果产品的建造更复杂，那么请用建造者模式。
 */
public class BuilderActivity extends AppCompatActivity {
    ActivityBuilderBinding mDataBinding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mDataBinding = DataBindingUtil.setContentView(this, R.layout.activity_builder);

        construct();
    }

    public void construct() {
//        Builder builder = new HolidayBuilder();
        //spring注入方法，
        Builder builder = new OldCustomerBuilder();

        //以下代码模板，轻易是不变的
        //创建苹果设置价格
        builder.buildApple(120);
        //创建香蕉设置香蕉价格
        builder.buildBanana(80);
        //创建桔子设置价格
        builder.buildOrange(50);

        FruitMeal fruitMeal = builder.getFruitMeal();


        int cost = fruitMeal.cost();
        System.out.println("本套件花费：" + cost);
    }
}

```



## 四、原型模式

> 原型模式是用于创建重复的对象，同时又能保证性能。这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。

这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。例如，一个对象需要在一个高代价的数据库操作之后被创建。我们可以缓存该对象，在下一个请求时返回它的克隆，在需要的时候更新数据库，以此来减少数据库调用。

#### 1. 模式的结构

原型模式包含以下主要角色。

1. 抽象原型类：规定了具体原型对象必须实现的接口。
2. 具体原型类：实现抽象原型类的 clone() 方法，它是可被复制的对象。
3. 访问类：使用具体原型类中的 clone() 方法来复制新的对象。

#### 2. 模式的实现

原型模式的克隆分为浅克隆深克隆

- 浅克隆：创建一个新对象，新对象的属性和原来对象完全相同，对于非基本类型属性，仍指向原有属性所指向的对象的内存地址。
- 深克隆：创建一个新对象，属性中引用的其他对象也会被克隆，不再指向原有对象地址。

```java

public class Admin implements Cloneable {

    private int age;
    private String sex;

    public Admin(int age, String sex) {
        super();
        this.age = age;
        this.sex = sex;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public Admin clone() {
        try {
            return (Admin) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public String toString() {
        return "Admin [age=" + age + ", sex=" + sex + "]";
    }

}

/**
 * 声明此类可以被clone
 */
public class Sheep implements Cloneable {

    private int age;
    private String sex;
    private Admin admin;

    public Sheep(int age, String sex, Admin admin) {
        super();
        this.age = age;
        this.sex = sex;
        this.admin = admin;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Admin getAdmin() {
        return admin;
    }

    public void setAdmin(Admin admin) {
        this.admin = admin;
    }

    @Override
    public String toString() {
        return "Sheep [age=" + age + ", sex=" + sex + ", admin=" + admin + "]";
    }

    /**
     *     调用Object的clone方法
     */
    @Override
    public Sheep clone() {
        Sheep sheep = null;
        try {
            sheep = (Sheep) super.clone();
            sheep.admin = this.admin.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return sheep;
    }

}
/**
 * 具体原型类
 */
public class Realizetype implements Cloneable {
    Realizetype() {
        System.out.println("具体原型创建成功！");
    }
    @Override
    public Object clone() throws CloneNotSupportedException {
        System.out.println("具体原型复制成功！");
        return (Realizetype) super.clone();
    }
}

**
 * 原型模式的测试类
 */
public class PrototypeTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Realizetype obj1 = new Realizetype();
        Realizetype obj2 = (Realizetype) obj1.clone();
        System.out.println("obj1==obj2?" + (obj1 == obj2));

        Sheep old = new Sheep(2, "雄性", new Admin(25, "女"));
        System.out.println(old.toString());
        Sheep current = old.clone();
        System.out.println(current.toString());

        //对克隆羊做处理
        current.setAge(1);
        current.setSex("雌性");
        current.getAdmin().setAge(34);
        current.getAdmin().setSex("男");
        System.out.println(old.toString());
        System.out.println(current.toString());
    }
}
```

<!-- {docsify-updated} -->

#### Reference

http://c.biancheng.net/view/1343.html

