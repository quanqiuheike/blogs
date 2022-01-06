# [RxJava2绑定流程分析之——观察者和被观察者是如何实现绑定的](https://www.cnblogs.com/tony-yang-flutter/p/12322827.html)
设计模式：响应式编程、装饰模式、观察者模式
一、概述

　　在上一节中我们分析了一个最简单的从观察者被观察者的创建、数据的发送到接收的流程。本节会着重分析一下Map操作符的原理以及源代码级别的具体实现。

二、最简单的RxJava，从创建观察者、绑定观察者、发射数据到接收过程回顾（温故而知新，如果觉得不够清晰可以先看上一节的代码分析）

　　1.创建观察者：调用`Observable.create(ObservableOnSubscribe source)`创建一个叫做`ObservableCreate`的被观察者，其持有`ObservableOnSubscribe`的实例。

　　2.创建被观察者：调用`Observable.subscribe(Consumer consumer)`发起订阅。会调用`subscribe`方法来创建一个`LambdaObserver`观察者对象，这个就是真正的观察者对象。然后调用`ObservableCreate`的`subscribeActual`方法实现观察者和被观察者真正的绑定。在`subscribeActual`方法中首先会创建一个`CreateEmitter`发射器，其实现了`ObservableEmitter`接口。`CreateEmitter`持有`Observer`对象。并调用`Observable.subscribe`来实现回调函数的调用，即设置`ObservableOnSubscribe`实例。

　　3.从数据的发送到接收：由于`ObservableEmitter`是一个接口真正的实现是`CreateEmitter`，所以调用`emitter.onNext`方法会间接调用`LambdaObserver`的`onNext`方法，而`LambdaObser`持有`Consumer`接口实例，所以`LambdaObserver`又会间接的调用`Consumer`接口实例的`accept`方法。最终完成数据的接收。

### 一、概述

　　本节将分析RxJava2的线程切换模型。通过对线程切换源代码的分析到达对RxJava2线程切换彻底理解的目的。通过对本节的学习你会发现，RxJava2线程切换是如此的简单，仅仅是通过两个操作符就能完成从子线程到主线程，或者主线程到子线程，再或者从子线程到子线程的切换。`对应的操作符为：observerOn：指定观察者运行的线程。subscribeOn：执行被观察者运行的线程`。

### 二、简单例子入手
#### 1、 从创建被观察者开始create（）

```java
//创建create方法在Obserable里面，传入的ObservableOnSubscribe
Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> observableEmitter) throws Exception {
                  observableEmitter.onNext("测试");
            }
        }).map(new Function<String, List<String>>() {
            @Override
            public List<String> apply(@NonNull String s) throws Exception {
                return null;
            }
        }).subscribeOn(Schedulers.newThread())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<List<String>>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable disposable) {
                        
                    }

                    @Override
                    public void onNext(@NonNull List<String> strings) {

                    }

                    @Override
                    public void onError(@NonNull Throwable throwable) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });


	//create方法
	 @CheckReturnValue
    @SchedulerSupport("none")
    public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
        ObjectHelper.requireNonNull(source, "source is null");
        return RxJavaPlugins.onAssembly(new ObservableCreate(source));
    }

```
create方法创建会new ObservableCreate(ObservableOnSubscribe<T> source)，创建一个被观察者，传递ObservableOnSubscribe接口对象进去，该接口有个抽象方法subscribe(@NonNull ObservableEmitter<String> ），接下来看ObservableCreate的构造方法，将接口对象给ObservableCreate的变量初始化赋值，用于后面调用ObservableOnSubscribe的subscribe(ObservableEmitter)

```java
  public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }
```
#### 紧接着再看map操作符，可以看到该操作符在Observable.class文件会创建ObservableMap(this, mapper)，第一个参数就是被观察者Observable，而该this对应的Observable为上面create（）的创建被观察者即ObservableCreate对象，第二个参数为Function。里面涉及到类和相关关系为：
1、 `ObservableCreate extends Observable  implements ObservableSource  `

2、`ObservableMap extends AbstractObservableWithUpstream<T, U> extends Observable<U> implements HasUpstreamObservableSource<T>              `


```java
 @CheckReturnValue
    @SchedulerSupport("none")
    public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        ObjectHelper.requireNonNull(mapper, "mapper is null");
        return RxJavaPlugins.onAssembly(new ObservableMap(this, mapper));
    }

//ObservableMap.class
 public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
//此处给到AbstractObservableWithUpstream的构造方法
        super(source);
        this.function = function;
    }

AbstractObservableWithUpstream(ObservableSource<T> source) {
//该source 为Observable，也就是ObservableCreate
        this.source = source;
    }

```
### 将被观察者和观察者通过subscribe绑定起来，对应的被观察者ObservableCreate->ObservableMap->subscribe(Observer)
#### 可以看到subscribe中实际上调用从subscribe()->到subscribeActual()，
```java
Observable.subscribe(Observer)
在Observable.class中
@SchedulerSupport("none")
    public final void subscribe(Observer<? super T> observer) {
        ObjectHelper.requireNonNull(observer, "observer is null");
        try {
            observer = RxJavaPlugins.onSubscribe(this, observer);
            ObjectHelper.requireNonNull(observer, "Plugin returned null Observer");
//如果没有map就是ObservableCreate的subscribeActual，
//有map则为ObservableMap的subscribeActual
            this.subscribeActual(observer);
        } catch (NullPointerException var4) {
            throw var4;
        } catch (Throwable var5) {
        。。。。
        }
    }

```
### 在Observable中的subscribeActual被ObservableCreate实现类重写，实际上先从ObservableCreate中看，一、先通过2处订阅；二、再用ObservableOnSubscribe调用它的subscribe（）；三、该方法内部发送消息  observableEmitter.onNext("测试")。observableEmitter的实现类为CreateEmitter，在1处初始化观察者observer，也就是；observableEmitter.onNext("测试")发生在CreateEmitter中的onNext（）方法
# `ObservableCreate.CreateEmitter implements ObservableEmitter<T>`，
### 在ObservableCreate.CreateEmitter内部调用this.observer.onNext(t);，也就是观察者的接收方法onNext（）。这一系列为ObservableCreate跟Observer关系订阅流程
```java
protected void subscribeActual(Observer<? super T> observer) {
//1
        ObservableCreate.CreateEmitter<T> parent = new ObservableCreate.CreateEmitter(observer);
//2、观察者调用onSubscribe（）先订阅
        observer.onSubscribe(parent);

        try {
//3、source为创建观察者的时候的ObservableOnSubscribe接口匿名对象，该对象调用它的方法subscribe（ObservableEmitter<String> observableEmitter），而该方法通过    observableEmitter.onNext("测试");方法发送消息。
            this.source.subscribe(parent);
        } catch (Throwable var4) {
            Exceptions.throwIfFatal(var4);
            parent.onError(var4);
        }
    }
//ObservableCreate.CreateEmitter内部类CreateEmitter中的onNext，调用  this.observer.onNext(t);
public void onNext(T t) {
            if (t == null) {
                this.onError(new NullPointerException("onNext called with null. Null values are generally not allowed in 2.x operators and sources."));
            } else {
                if (!this.isDisposed()) {
                    this.observer.onNext(t);
                }

            }
        }

```
### 接下来看Map，Map中通过ObservableCreate转换到ObservableMap，在ObservableMap构造方法中传递的第一个参数this为ObservableCreate。
```java
 public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        ObjectHelper.requireNonNull(mapper, "mapper is null");
        return RxJavaPlugins.onAssembly(new ObservableMap(this, mapper));
    }
//ObservableMap
public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
//1,次数调用的是super2处的构造方法AbstractObservableWithUpstream
        super(source);
        this.function = function;
    }
// 2
AbstractObservableWithUpstream(ObservableSource<T> source) {
        this.source = source;
    }
```
### 接下来是ObservableMap与观察者绑定通过subscribe。subscribe（）中调用ObservableMap的subscribeActual，里面调用`this.source.subscribe(new ObservableMap.MapObserver(t, this.function));`ObservableMap，该source接收上面的ObservableCreate，也就是调用ObservableCreate的subscribe（）方法。而ObservableCreate的subscribe（）方法在Observable中，也就是又会走到,又回到上述分析流程了，继续走create中的ObservableOnSubscribe回调方法subscribe（）通过observableEmitter发送消息onNext()回调，
```java
 @SchedulerSupport("none")
    public final void subscribe(Observer<? super T> observer) {
        ObjectHelper.requireNonNull(observer, "observer is null");

        try {
            observer = RxJavaPlugins.onSubscribe(this, observer);
            ObjectHelper.requireNonNull(observer, "Plugin returned null Observer");
//该处为ObservableMap的subscribeActual
            this.subscribeActual(observer);
        } catch (NullPointerException var4) {
            throw var4;
        } catch (Throwable var5) {
            Exceptions.throwIfFatal(var5);
            RxJavaPlugins.onError(var5);
            NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
            npe.initCause(var5);
            throw npe;
        }
    }

//ObservableMap，该source接收上面的ObservableCreate，也就是调用ObservableCreate的subscribe（）方法。
    public void subscribeActual(Observer<? super U> t) {
//3，实际上source是ObservableCreate，而ObservableCreate中的subscribe在Observable中
        this.source.subscribe(new ObservableMap.MapObserver(t, this.function));
    }


@SchedulerSupport("none")
    public final void subscribe(Observer<? super T> observer) {
        ObjectHelper.requireNonNull(observer, "observer is null");

        try {
            observer = RxJavaPlugins.onSubscribe(this, observer);
            ObjectHelper.requireNonNull(observer, "Plugin returned null Observer");
//4,又回到上述分析流程了，继续走create中的ObservableOnSubscribe回调方法subscribe（）通过observableEmitter发送消息onNext()回调，
            this.subscribeActual(observer);
        } catch (NullPointerException var4) {
            throw var4;
        } catch (Throwable var5) {
            Exceptions.throwIfFatal(var5);
            RxJavaPlugins.onError(var5);
            NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
            npe.initCause(var5);
            throw npe;
        }
    }

```


```java
protected void subscribeActual(Observer<? super T> observer) {
        ObservableCreate.CreateEmitter<T> parent = new ObservableCreate.CreateEmitter(observer);
        observer.onSubscribe(parent);

        try {
            this.source.subscribe(parent);
        } catch (Throwable var4) {
            Exceptions.throwIfFatal(var4);
            parent.onError(var4);
        }

    }
```

#### Reference&Thanks

[RxJava2链式调用操作符源码分析之map操作符](https://www.cnblogs.com/tony-yang-flutter/p/12326408.html)

[RxJava2线程切换原理分析](https://www.cnblogs.com/tony-yang-flutter/p/12331753.html)