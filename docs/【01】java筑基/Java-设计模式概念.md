---
## Reference

## [Java设计模式：23种设计模式全面解析（超级详细）](http://c.biancheng.net/design_pattern/) {docsify-ignore}

#### [Github DesignPattern](https://github.com/youlookwhat/DesignPattern){docsify-ignore}



# 一、设计模式
设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。

一句话： 降低对象之间的耦合，增加程序的可复用性、可扩展性和可维护性

> 记忆口诀：访问加限制，函数要节俭，依赖不允许，动态加接口，父类要抽象，扩展不更改。

# 二、设计模式的七大原则
### 1、开闭原则（Open Close Principle）
开闭原则就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，实现一个热插拔的效果。所以一句话概括就是：为了使程序的扩展性好，易于维护和升级。想要达到这样的效果，我们需要使用接口和抽象类。
### 2、单一指责原则(Single Responsibility Principle，SRP)
单一职责原则规定一个类应该有且仅有一个引起它变化的原因，否则类应该被拆分。核心就是控制类的粒度大小、将对象解耦、提高其内聚性。

单一职责原则的优点

- 降低类的复杂度。一个类只负责一项职责，其逻辑肯定要比负责多项职责简单得多。
- 提高类的可读性。复杂性降低，自然其可读性会提高。
- 提高系统的可维护性。可读性提高，那自然更容易维护了。
- 变更引起的风险降低。变更是必然的，如果单一职责原则遵守得好，当修改一个功能时，可以显著降低对其他功能的影响。

### 3、里氏替换原则（Liskov Substitution Principle）
**子类可以扩展父类的功能，但不能改变父类原有的功能。**也就是说：子类继承父类时，除添加新的方法完成新增功能外，尽量不要重写父类的方法。里氏替换原则主要阐述了有关继承的一些原则，也就是什么时候应该使用继承，什么时候不应该使用继承，以及其中蕴含的原理。里氏替换原是继承复用的基础，它反映了基类与子类之间的关系，是对开闭原则的补充，是对实现抽象化的具体步骤的规范。

>* 里氏替换原则是实现开闭原则的重要方式之一。
>
>* 它克服了继承中重写父类造成的可复用性变差的缺点。
>
>* 它是动作正确性的保证。即类的扩展不会给已有的系统引入新的错误，降低了代码出错的可能性。
>
>* 加强程序的健壮性，同时变更时可以做到非常好的兼容性，提高程序的维护性、可扩展性，降低需求变更时引入的风险。

### 4、依赖倒置原则（Dependence Inversion Principle）
高层模块不应该依赖低层模块，两者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。

其核心思想是：要面向接口编程，不要面向实现编程。依赖于抽象而不依赖于具体。

`依赖倒置原则是实现开闭原则的重要途径之一，它降低了客户与实现模块之间的耦合。`

### 5、接口隔离原则（Interface Segregation Principle，ISP）
**定义：**使用多个隔离的接口，比使用单个接口要好。降低依赖，降低耦合。是一个降低类之间的耦合度的意思，一个类对另一个类的依赖应该建立在最小的接口上

**接口隔离原则**和**单一职责**都是为了提高类的内聚性、降低它们之间的耦合性，体现了封装的思想

**区别：**

- 单一职责原则注重的是职责，而接口隔离原则注重的是对接口依赖的隔离。
- 单一职责原则主要是约束类，它针对的是程序中的实现和细节；接口隔离原则主要约束接口，主要针对抽象和程序整体框架的构建。

### 6、迪米特法则（最少知道原则）（Law of Demeter，LoD）
**定义：**一个实体应当尽量少的与其他实体之间发生相互作用，使得系统功能模块相对独立。如果两个软件实体无须直接通信，那么就不应当发生直接的相互调用，可以通过第三方转发该调用。其目的是降低类之间的耦合度，提高模块的相对独立性

**优点：**

1. 降低了类之间的耦合度，提高了模块的相对独立性。
2. 由于亲合度降低，从而提高了类的可复用率和系统的扩展性。

### 7、合成复用原则（Composite Reuse Principle）
原则是尽量使用合成/聚合的方式，而不是使用继承。如果要使用继承关系，则必须严格遵循里氏替换原则。合成复用原则同里氏替换原则相辅相成的，两者都是开闭原则的具体实现规范。

|设计原则|归纳|目的|
|---|---|---|
|开闭原则|对扩展开放，对修改关闭|降低维护带来的新风险|
|依赖倒置原则|高层不应该依赖低层，要面向接口编程|更利于代码结构的升级扩展|
|单一职责原则|一个类只干一件事，实现类要单一|便于理解，提高代码的可读性|
|接口隔离原则|一个接口只干一件事，接口要精简单一|功能解耦，高聚合、低耦合|
|迪米特法则|不该知道的不要知道，一个类应该保持对其它对象最少的了解，降低耦合度|只和朋友交流，不和陌生人说话，减少代码臃肿|
|里氏替换原则|不要破坏继承体系，子类重写方法功能发生改变，不应该影响父类方法的含义|防止继承泛滥|
|合成复用原则|尽量使用组合或者聚合关系实现代码复用，少使用继承|降低代码耦合|



![](.\images\设计模式分类.png)

![](.\images\设计模式详解.png)



### 三、设计模式分类

- **创建型模式**：[单例模式](https://github.com/youlookwhat/DesignPattern#3-单例设计模式)、[抽象工厂模式](https://github.com/youlookwhat/DesignPattern#2-工厂模式)、[建造者模式](https://github.com/youlookwhat/DesignPattern#11-建造者模式)、[工厂模式](https://github.com/youlookwhat/DesignPattern#2-工厂模式)、[原型模式](https://github.com/youlookwhat/DesignPattern#12-原型模式)。
- **结构型模式**：[适配器模式](https://github.com/youlookwhat/DesignPattern#5-适配器模式)、[桥接模式](https://github.com/youlookwhat/DesignPattern#15-桥接模式)、[装饰模式](https://github.com/youlookwhat/DesignPattern#7-装饰者模式)、[组合模式](https://github.com/youlookwhat/DesignPattern#16-组合模式)、[外观模式](https://github.com/youlookwhat/DesignPattern#8-外观模式)、[享元模式](https://github.com/youlookwhat/DesignPattern#13-享元模式)、[代理模式](https://github.com/youlookwhat/DesignPattern#14-代理模式)。
- **行为型模式**：[模版方法模式](https://github.com/youlookwhat/DesignPattern#9-模板方法模式)、[命令模式](https://github.com/youlookwhat/DesignPattern#6-命令模式)、[迭代器模式](https://github.com/youlookwhat/DesignPattern#17-迭代器模式)、[观察者模式](https://github.com/youlookwhat/DesignPattern#1-观察者模式)、[中介者模式](https://github.com/youlookwhat/DesignPattern#18-中介者模式)、[备忘录模式](https://github.com/youlookwhat/DesignPattern#19-备忘录模式)、[解释器模式](https://github.com/youlookwhat/DesignPattern#20-解释器模式)、[状态模式](https://github.com/youlookwhat/DesignPattern#10-状态模式)、[策略模式](https://github.com/youlookwhat/DesignPattern#4-策略模式)、[责任链模式](https://github.com/youlookwhat/DesignPattern#21-责任链模式)、[访问者模式](https://github.com/youlookwhat/DesignPattern#22-访问者模式)。

##### 创建型模式分为以下几种。

- **单例（Singleton）模式：**某个类只能生成一个实例，该类提供了一个全局访问点供外部获取该实例，其拓展是有限多例模式。
- **原型（Prototype）模式**：将一个对象作为原型，通过对其进行复制而克隆出多个和原型类似的新实例。
- **工厂方法（FactoryMethod）模式**：定义一个用于创建产品的接口，由子类决定生产什么产品。
- **抽象工厂（AbstractFactory）模式**：提供一个创建产品族的接口，其每个子类可以生产一系列相关的产品。
- **建造者（Builder）模式**：将一个复杂对象分解成多个相对简单的部分，然后根据不同需要分别创建它们，最后构建成该复杂对象。

#### 



 