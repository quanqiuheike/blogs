# AutoService、APT、Annotation使用及原理	

## 一、AutoService基本使用

##### [GitHub的AutoService地址](https://github.com/google/auto)

##### [Github javapoet官方地址](https://github.com/square/javapoet)



#### Gradle项目依赖,AutoService，AnnotationProcessor、JavaPoet、

```java

//AutoService
annotationProcessor 'com.google.auto.service:auto-service:1.0.1'
implementation 'com.google.auto.service:auto-service:1.0.1'
    
//javapoet
implementation 'com.squareup:javapoet:1.13.0'

```



##### 如何查找版本

* ##### 点开官网GitHub地址，找到对应的AutoService，点击[AutoService maven](https://mvnrepository.com/artifact/com.google.auto.service/auto-service)地址，

  即可看到版本,比如**1.0.1**，然后**点击版本号1.0.1**进入版本具体使用。

<img src="C:\Users\chengqiuxia\AppData\Roaming\Typora\typora-user-images\image-20211119123551127.png" alt="image-20211119123551127" style="zoom: 50%;" />

* ##### 点击版本号后进入该版本详情界面

  ```java
  // https://mvnrepository.com/artifact/com.google.auto.service/auto-service
  implementation 'com.google.auto.service:auto-service:1.0.1'
      
   Kotlin版本
  // https://mvnrepository.com/artifact/com.google.auto.service/auto-service
  implementation("com.google.auto.service:auto-service:1.0.1")
  
  ```
  
  

<img src="C:\Users\chengqiuxia\AppData\Roaming\Typora\typora-user-images\image-20211119124323292.png" alt="image-20211119124323292" style="zoom:80%;" />



# APT 编译期处理注解

#### 1、APT：Annotation Processing Tools。编译期处理注解工具。

对源代码文件进行检测找出其中的`Annotation`，`Annotation`处理器在处理`Annotation`时可以根据源文件中的`Annotation`**生成额外的源文件和其它的文件，APT还会编译生成的源文件和原来的源文件，将它们一起生成class文件。**

简单的说：不影响性能的情况下，自动生成代码。

常用的`ButterKnife、Dagger`等都用注解生成代码的工具几乎都是用了`APT`。

#### 2、四个元注解

**@target:** 目标位置，也就是注解可以被用在那里，如方法、属性、类

**@Retention：**保留级别，也就是注解生效的时间，换言之理解为"生命周期"

**@Documented：**包含进文档

**@Inherited：**子类继承父类注解

#### 3、APT工作流程

1、定义注解

2、定义注解处理器 （AbstractProcess）

3、处理器中生成java代码 （javaPoet）

4、注册处理器 (AutoService)

##### AutoService注册

​		`AutoService`可以自动生成`META-INF/services/javax.annotation.processing.Processor`文件的。省去了打`jar`包这些繁琐的步骤。

​		由于处理器是`javac`的工具，因此我们必须将我们自己的处理器注册到javac中，在以前我们需要提供一个.jar文件，打包你的注解处理器到此文件中，并在在你的jar中，需要打包一个特定的文件 `javax.annotation.processing.Processor到META-INF/services`路径下
把`MyProcessor.jar`放到你的`builpath`中，javac会自动检查和读取`javax.annotation.processing.Processor`中的内容，并且注册`MyProcessor`作为注解处理器。

#### 4、注册处理器的大致流程（使用AutoService可以直接解决注册）

1、继承AbstractProcessor，process()方法处理主要逻辑。
2、处理器由相应工厂AnnotationProcessFactory工厂返回
3、两种获得工厂的方法。命令行直接运行已知工厂；jar指定路径检索。

## Annotation Processor使用须知

### 1.了解Annotation Processor 

Annotation Processor是javac的一个工具，它用来在**编译时扫描和处理注解**。通过Annotation Processor可以获取到注解和被注解对象的相关信息，然后根据注解自动生成Java代码，省去了手动编写，提高了编码效率。使用注解处理器先要了解AbstractProcessor类，这个类是一个抽象类，有四个核心方法，关于AbstractProcessor类后面再做详细解析。

### 2.在Android Studio使用Annotation Processor 

> 为什么要强调上述两个模块一定要是Java Library？如果创建Android Library模块你会发现不能找到AbstractProcessor这个类，这是因为Android平台是基于OpenJDK的，而OpenJDK中不包含APT的相关代码。因此，在使用APT时，必须在Java Library中进行。

刚接触Annotation Processor的同学可能会遇到找不到AbstractProcessor类的问题，大概率是因为直接在Android项目里边引用了AbstractProcessor，然而由于Android平台是基于OpenJDK的，而OpenJDK中不包含Annotation Processor的相关代码。因此，在使用Annotation Processor时，必须在新建Module时选择Java Library，处理注解相关的代码都需要在Java Library模块下完成。我们需要看一下整个项目的结构

* annotation模块（Java Library） 该模块存放的是我们自定义的注解，是一个Java Library 
* compiler模块 (Java Library) 依赖annotation模块，处理注解并自动生成代码等，同样也是Java Library。 *
* app (Android App) 依赖compiler模块，需要使用annotationProcessor依赖compiler模块 

()

[MAnnotation例子](https://github.com/zhpanvip/MAnnotation)

![image-20211119142834957](C:\Users\chengqiuxia\AppData\Roaming\Typora\typora-user-images\image-20211119142834957.png)





例子明天自写