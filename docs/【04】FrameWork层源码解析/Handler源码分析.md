## Handler、Looper、MessageQueue、Message之间的关系？

1、工作的流程，首先通过当前线程和`Looper.prepare()`创建`looper`。

2、创建Looper对象时会创建消息队列MessageQueue。

3、采用handler发送消息会创建Message信息，然后通过Handler发送Message到MessageQueue消息队列

4、然后再通过Looper取出消息再交给Handler进行处理。

## 一、Looper

要使用handle进行通信，首先需要通过`Looper.prepare()`创建`looper`对象，`Looper`中会持有一个消息队列`MessageQueue`，每个线程中只能创建的一个Looper和MessageQueue。主线程创建的Looper在ActivityThread#main（）方法中`Looper.prepareMainLooper();`

```java
//准备创建Looper调用方式
Looper.prepare();

Looper.java

public static void prepare() {
        prepare(true);
 }


 private static void prepare(boolean quitAllowed) {
 //如果该线程存在looper则报错，一个线程只能有一个looper
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        //创建looper实例对象，存储到ThreadLocal中
        sThreadLocal.set(new Looper(quitAllowed));
    }

//实例化looper对象会创建一个MessageQueue
 private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
    
//主线程Looper的创建
@Deprecated
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
//获取Looper
  public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }

```

## 二、Handler的创建

Handler是Andorid给我们提供的一套UI更新机制，同时它也是一套消息处理机制。存在多个构造方法。创建handle的时候会指明该Handler所在的looper环境，如果不存在，则抛异常，所以先创建looper。

```java
public Handler(@NonNull Looper looper) {
        this(looper, null, false);
}

public Handler(@NonNull Looper looper, @Nullable Callback callback) {
        this(looper, callback, false);
 }
...
    
public Handler(@Nullable Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
		//获取Looper中的looper,不存在则报错，主线程在ActivityThread#main()方法中创建了looper,子线程要想使用则需要手动通过Looper.loop()开启循环
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread " + Thread.currentThread()
                        + " that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }

//三个参数的构造方法
 public Handler(@NonNull Looper looper, @Nullable Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```

Hanlde的使用方式多种，一般得指定Looper。不指定的方式已弃用：

```java
//指定Looper
mHandler = new Handler(Looper.myLooper());
//指定Looper和Callback接收消息
mHandler = new Handler(getMainLooper(), new Handler.Callback() {
            @Override
            public boolean handleMessage(@NonNull Message msg) {
                return false;
            }
});


```

Handle消息的纷发分为三种情况

```java
	/**
     * Handle system messages here.
     */
    public void dispatchMessage(@NonNull Message msg) {

        if (msg.callback != null) {
            //1、如果Message的callback不为空，则走handleCallback(msg);场景：Activity implements Callback
            handleCallback(msg);
        } else {
            if (mCallback != null) {
               //2
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //3
            handleMessage(msg);
        }
    }
```



## 消息Message


```java
使用
// 直接new Message
Message message=new Message();
//通过obtain获取
Message message = Message.obtain();

Message.java
//消息缓存，从单链表中取，将sPool指向当前Message，Message的next指向下一个Message
public static Message obtain() {
//主要是给Message加一个对象锁，不允许多个线程同时访问Message类和obtain方法，保证获取到的sPool是最新的
        synchronized (sPoolSync) {
        //Message sPool存储我们循环利用Message的单链表;sPool是链表的头节点，判断链表是否为空链表
            if (sPool != null) {
            //将链表头节点移除作为重用的Message对象,将m指向sPool链表头
                Message m = sPool;
                //sPool指向消息m的下一个message，实际上就是链表sPool右移一位
                sPool = m.next;
                //将消息的next下一个指向为null代表链表断开
                m.next = null;
                m.flags = 0; // clear in-use flag
                //单链表的链表的长度即存储的Message对象的个数，链表大小自减1
                sPoolSize--;
                //将链表分为了两段，链表头部的m和sPool指向的链表头
                return m;
            }
        }
        return new Message();
    }
```

`sPoolSync`即锁对象，该对象在定义时即被初始化`private static final Object sPoolSync = new Object();`，随后便只读不写。
然后便是`sPool`，后面还有`Message m = sPool;sPool = m.next;`，很明显可以看出来，这是一个链表结构。`sPool指向当前message`，`next指向下一个message`。
在解释这段代码前，需要先明确两点：sPool声明为private static Message sPool;；next声明为/*package*/ Message next;。即前者为**该类所有示例共享**，后者则每个实例都有。
现在为了便于理解，我们将Message抽象为C语言中的链表节点结构体，指针域便是用于指向下一个消息的next字段，其他则都视为数据域。

* 假设该链表初始状态如下

  

  ![](.\images\Message-sPool链表初始状态.png)

* 执行`Message m = sPool;`就变成下图



![](.\images\Message-sPool链表将当前消息给到Message.png)



* 继续`sPool = m.next;`

![](.\images\Message-sPool链表指针后移.png)



* 然后`m.next = null;`

![](.\images\Message.next断开链表指向.png)

接下来`m.flags=0;sPoolSize--;return m;`便是表示m指向的对象已经从链表中取出并返回了。

![Handle消息复用obtain原理](.\images\Handle消息复用obtain原理.png)





## 消息的分类

**Message分为3中：普通消息（同步消息）、屏障消息（同步屏障）和异步消息**。我们通常使用的都是普通消息，而屏障消息就是在消息队列中插入一个屏障，在屏障之后的所有普通消息都会被挡着，不能被处理。不过异步消息却例外，屏障不会挡住异步消息。因此可以这样认为：屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。比如handleResumeActivity中的viewRootImpl.setView方法。 同步屏障后面会重点介绍，这里先了解异步消息。



### Handler、Looper、MessageQueue、Message之间的关系？

这里先简单概述下它们的工作的流程，首先是创建Message信息，然后通过Handler发送到MessageQueue消息队列，然后再通过Looper取出消息再交给Handler进行处理。

**总结**：这就是通过obtain方法获取Message对象的详情。通过obtain方法获取Message对象使得Message到了重复的利用，减少了每次获取Message时去申请空间的时间。同时，这样也不会永无止境的去创建新对象，减小了Jvm垃圾回收的压力，提高了效率。

### Reference

[荐：【Android自助餐】Handler消息机制完全解析（一）Message中obtain()与recycle()的来龙去脉](https://blog.csdn.net/xmh19936688/article/details/51901338)

[Handler消息机制之深入理解Message.obtain()](https://blog.csdn.net/chenbaige/article/details/79473475)


