[TOC]

## Handler、Looper、MessageQueue、Message之间的关系？

**工作的流程**

1、首先通过当前线程和`Looper.prepare()`创建`looper`。

2、创建Looper对象时会创建消息队列MessageQueue。

3、采用handler发送消息会创建Message信息，然后通过Handler发送Message到MessageQueue消息队列

4、然后再通过Looper.loop()取出消息再交给Handler进行处理。

## 一、Looper

要使用handle进行通信，首先需要通过`Looper.prepare()`创建`looper`对象，`Looper`中会持有一个消息队列`MessageQueue`，每个线程中只能创建的一个Looper和MessageQueue。主线程创建的Looper在`ActivityThread#main（）`方法中`Looper.prepareMainLooper();`

```java
public static void main(String[] args) {
        ....

        //创建Looper和MessageQueue对象，用于处理主线程的消息
        Looper.prepareMainLooper();

        //创建ActivityThread对象
        ActivityThread thread = new ActivityThread(); 

        //建立Binder通道 (创建新线程),binder通道为死循环创建一个新线程去运转
    	//便会创建一个Binder线程（具体是指ApplicationThread，Binder的服务端，用于接收系统服务AMS发送来的事件），该Binder线程通过Handler将Message发送给主线程
        thread.attach(false);

        Looper.loop(); //消息循环运行
        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```



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

##### Handle消息的纷发分为三种情况



```java

/**
     * Handle system messages here.
     */
    public void dispatchMessage(@NonNull Message msg) {
//callback是msg中的一个字段，是一个Runnable对象，当通过handler.post方法发送一个runnable的时候就会被封装到这个msg中
        if (msg.callback != null) {
            //1、如果Message的callback不为空，则走handleCallback(msg);场景：Activity implements Callback
            //此方法是Handler中的一个静态方法，方法体：message.callback.run();只有这一句，可以看出是直接调用的run()方法，没有新建线程，否则也不符合这里线程通信了对吧
            handleCallback(msg);
        } else {
            //mCallback就是Callback接口的对象，Handler.Callback
            if (mCallback != null) {
               //2
                //如果返回true直接return结束方法，不再调用handler中的handleMessage方法。
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //3 handler中的消息处理方法
            handleMessage(msg);
        }
    }

Message.java
Runnable callback;
 public static Message obtain(Handler h, Runnable callback) {
        Message m = obtain();
        m.target = h;
        m.callback = callback;

        return m;
    }
  /** @hide */
//对应上面的1
    @UnsupportedAppUsage
    public Message setCallback(Runnable r) {
        callback = r;
        return this;
    }

Handler.java
//对应上面的1
private static void handleCallback(Message message) {
    message.callback.run();
}
handler.post(r)
public final boolean post(@NonNull Runnable r) {
    //调用了getPostMessage，将r赋值给Message的Callback，对应1
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
 private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }
    
//对应上面的2 实现Callback接口，复写handleMessage方法
public interface Callback {
        /**
         * @param msg A {@link android.os.Message Message} object
         * @return True if no further handling is desired
         */
        boolean handleMessage(@NonNull Message msg);
}
    
/**
 * Subclasses must implement this to receive messages.
 * 继承Handler，对应上面的3
*/
public void handleMessage(@NonNull Message msg) {
}

```

##### 以上三种handlerMessage处理的场景

```java

//1、优先级最高 
mHandler.post(new Runnable() {
        @Override
        public void run() {

        }
});

//2、直接实现Handler.Callback#handleMessage
public class MainActivity extends AppCompatActivity implements Handler.Callback {

 	@Override
	public boolean handleMessage(@NonNull Message msg) {
        return false;
    }
}

//2、匿名实现Handler.Callback#handleMessage
//Handler.Callback内部接口：场景2，执行Handler.Callback的handleMessage
mHandler = new Handler(getMainLooper(), new Handler.Callback() {
    //Handler.Callback()的handleMessage
            @Override
            public boolean handleMessage(@NonNull Message msg) {
                return false;
            }
});

//3、优先级最低 
class CusHandler extends Handler{
        @Override
        public void handleMessage(@NonNull Message msg) {
            //handler的handleMessage
            super.handleMessage(msg);
        }
}
```



#### handler发送消息，入队过程

注意区分几个概念：

当前头部消息mMessage，msg为传递进去的Message，mMessage和msg不是一个，声明的p指向的也是队列的头部Message，跟mMessage指向一样

```java
  Message message = Message.obtain();
  mHandler.sendMessage(message);

Handler.java

public final boolean sendMessage(@NonNull Message msg) {
        return sendMessageDelayed(msg, 0);
    }

 public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }

 public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
        MessageQueue queue = mQueue;
        if (queue == null) {
            RuntimeException e = new RuntimeException(
                    this + " sendMessageAtTime() called with no mQueue");
            Log.w("Looper", e.getMessage(), e);
            return false;
        }
        return enqueueMessage(queue, msg, uptimeMillis);
    }
//统一调用enqueueMessage方法，最后走到MessageQueue#enqueueMessage（）
private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
            long uptimeMillis) {
    //target=this,this代表handler
        msg.target = this;
        msg.workSourceUid = ThreadLocalWorkSource.getUid();

    //判断是否为异步消息
        if (mAsynchronous) {
            //设置为异步消息，一般是在消息屏障后面
            msg.setAsynchronous(true);
        }
        return queue.enqueueMessage(msg, uptimeMillis);
    }

MessageQueue.java

/**
 *消息入队列当中，根据when比较排序
 */
boolean enqueueMessage(Message msg, long when) {
//屏障消息的target才会为null，区别于普通消息,屏障消息不会调用该方法
        if (msg.target == null) {
            throw new IllegalArgumentException("Message must have a target.");
        }

        synchronized (this) {
        // 一个Message，只能发送一次 ，同步锁
            if (msg.isInUse()) {
                throw new IllegalStateException(msg + " This message is already in use.");
            }

            if (mQuitting) {
                IllegalStateException e = new IllegalStateException(
                        msg.target + " sending message to a Handler on a dead thread");
                Log.w(TAG, e.getMessage(), e);
                //消息回收复用，初始化消息状态，链表++
                msg.recycle();
                return false;
            }
 			// 标记Message已经使用了 
            msg.markInUse();
            msg.when = when;
           // 得到当前消息队列的头部 
            Message p = mMessages;
            boolean needWake;
          	// 我们这里when为0，表示立即处理的消息 或者消息队列为空，或者当前时间比队列头部时间小，都插入消息队列的头部，都是立即执行系列
            if (p == null || when == 0 || when < p.when) {
                // New head, wake up the event queue if blocked.
                // 把消息插入到消息队列的头部 
                msg.next = p;
                //当前头部消息为发送过来的msg
                mMessages = msg;
                needWake = mBlocked;
            } else {
                // Inserted within the middle of the queue.  Usually we don't have to wake
                // up the event queue unless there is a barrier at the head of the queue
                // and the message is the earliest asynchronous message in the queue.
                // 根据延迟执行时间排序，根据需要把消息插入到消息队列的合适位置，通常是调用xxxDelay方法，延时发送消息，target=null是屏障消息，通常情况下不需要唤醒事件队列，除非链表的头是一个同步屏障，并且该条消息是第一条异步消息
                needWake = mBlocked && p.target == null && msg.isAsynchronous();
                //双向链表
                Message prev;
                for (;;) {
                	//指向头部
                    prev = p;
                    //头部后移，也就是发送过来的消息跟链表中的时间对比
                    p = p.next;
                    //如果时间排序正确或者队列为空则退出循环
                    if (p == null || when < p.when) {
                        break;
                    }
                    //如果是true满足上面同步屏障的条件，并且p（该p不是msg）也是异步，那就不需要阻塞，屏障只会拦截后面来的异步消息，在设置屏障之前的异步消息不用管，按照原逻辑处理
                    if (needWake && p.isAsynchronous()) {
                        needWake = false;
                    }
                }
                 // 把消息插入到合适位置 prev=当前头部消息，p等于合适的位置，最后prev指向了合适的位置p,跟mMessages无关，他还是头部消息，三个概念。
                msg.next = p; // invariant: p == prev.next
               
                prev.next = msg;
            }

            // We can assume mPtr != 0 because mQuitting is false.
           // 如果队列阻塞了，则唤醒 ,相当于唤醒looper中等待的线程（如果是即时消息并且是等待的线程）?
           // nativeWake 写入一个 IO 操作到描述符, nativePollOnce 在某个文件描述符上调用 epoll_wait等待,nativeWake 等同于 Object.notify()，nativePollOnce 大致等同于 Object.wait()
            if (needWake) {
            //如果发送的是异步消息(是插入的msg,不是p)，并且消息链表第一条消息是同步屏障消息。
            //或者消息链表中只有刚发送的这一个Message，并且mBlocked为true即，正在阻塞状态，收到一个消息后也进入唤醒
            //这里唤醒 nativePollOnce 的沉睡
                nativeWake(mPtr);
            }
        }
        return true;
    }



```



## 三、Message消息复用、回收原理

#### 消息的复用创建


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
                //将链表分为了两段，链表头部的m和sPool指向的链表头，得到的m则会复用的msg
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

#### 回收recycle()

然后再看看`sPoolSize`是什么时候自增的。按图索骥便可找到`recycle()`方法和`recycleUnchecked()`方法。前者供开发者调用进行回收，后者执行回收操作。来看看回收操作都干了啥：

```java
 public void recycle() {
     //判断isInUse()是否在使用这个Message
        if (isInUse()) {
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }

/*package*/ boolean isInUse() {
        return ((flags & FLAG_IN_USE) == FLAG_IN_USE);
    }

@UnsupportedAppUsage
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        //重置为初始状态
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = UID_NONE;
        workSourceUid = UID_NONE;
        when = 0;
        target = null;//handle
        callback = null;
        data = null;

        synchronized (sPoolSync) {
            //private static final int MAX_POOL_SIZE = 50;
            //缓冲池大小为50最大值
            if (sPoolSize < MAX_POOL_SIZE) {
                //消息next节点指向sPool链表头
                next = sPool;
                //链表头指向的是当前的Message
                sPool = this;
                //链表自增
                sPoolSize++;
            }
        }
    }
```

前半段不必多说，显然是“重置”改对象的个个字段。后半段又是一个同步代码段，同样用图来解释一下（假设当前代码为`message.recycle()`，则需要被回收的则是`message`对象）。
假设当前链表如下：

![](.\images\回收Message-sPool初始状态.png)

执行`next=sPool;`



![](.\images\回收-next=spool.png)



执行`sPool=this`;

![](.\images\回收-sPool=this.png)



**Message类本身就组织了一个栈结构的缓冲池。并使用`obtain()`方法和`recycler()`方法来取出和放入。**

只有调用了`recycle()`方法后才会`sPool`才会有值，所以，在其他`Message`对象执行`recycle()`方法前，使用`obtain()`方法实例化对象其实都是`new Message()`



## 四、Looper.loop()

```java
 public static void loop() {
     //获取looper，不能为空，需要通过prepare创建
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        if (me.mInLoop) {
            Slog.w(TAG, "Loop again would have the queued messages be executed"
                    + " before this one completed.");
        }

        me.mInLoop = true;
     //拿到该looper实例中的消息队列
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride =
                SystemProperties.getInt("log.looper."
                        + Process.myUid() + "."
                        + Thread.currentThread().getName()
                        + ".slow", 0);

        boolean slowDeliveryDetected = false;

     //开启死循环
        for (;;) {
            //阻塞条件是消息屏障，只是阻塞，取出MessageQueue一条消息，如果没有消息则阻塞等待
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            // Make sure the observer won't change while processing a transaction.
            final Observer observer = sObserver;

            final long traceTag = me.mTraceTag;
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            Object token = null;
            if (observer != null) {
                token = observer.messageDispatchStarting();
            }
            long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
            try {
                //target为handle，即为handle的dispatchMessage（）
                msg.target.dispatchMessage(msg);
                if (observer != null) {
                    //
                    observer.messageDispatched(token, msg);
                }
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } catch (Exception exception) {
                if (observer != null) {
                    observer.dispatchingThrewException(token, msg, exception);
                }
                throw exception;
            } finally {
                ThreadLocalWorkSource.restore(origWorkSource);
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                            msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            //消息回收，此时才会有spool++
            msg.recycleUnchecked();
        }
    }
```

##### 消息队列的next

创建同步屏障消息时：没有指定消息的target：Message.target=null

创建同步/异步消息时：在消息入队enqueueMeesage()方法中，指定消息的target为发送者Handler本身。

创建Handler对象时，不指定异步，默认发送的消息是同步的

创建Message时，不调用Message#setAsynchronous(true)，默认创建的消息是同步的

```java
 @UnsupportedAppUsage
    Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
       // Java层中MessageQueue的mPtr属性，是native层的NativeMessageQueue对象的指针。
        final long ptr = mPtr;
        if (ptr == 0) {
            //从注释可以看出，只有looper被放弃的时候（调用了quit方法）才返回null，mPtr是MessageQueue的一个long型成员变量，关联的是一个在C++层的MessageQueue，阻塞操作就是通过底层的这个MessageQueue来操作的；当队列被放弃的时候其变为0。
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        //next中取消息会一直获取直到拿到结果，不管结果是什么
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }
  //阻塞方法，主要是通过native层的epoll监听文件描述符的写入事件来实现的。
           //如果nextPollTimeoutMillis=-1，一直阻塞不会超时。满了或者为空的情况为-1
           //如果nextPollTimeoutMillis=0，不会阻塞，立即返回。
           //如果nextPollTimeoutMillis>0，最长阻塞nextPollTimeoutMillis毫秒(超时)，如果期间有程序唤醒会立即返回。
            //  nativePollOnce 相当于object.wait()这里陷入沉睡, 等待唤醒对应enqueueMessage的nativeWake（）.nativePollOnce表示消息的处理已完成, 线程正在等待下一个消息.
            //如果队列为空(无返回值), 该方法将一直阻塞直到添加新消息为止
            //可以从该方法for前面默认值为0看到，第一次进来是不会阻塞，直接唤醒，往下执行，后面会再次计算时间是否超时，如果存在超时记录下超时时间，继续for循环的时候会阻塞等待
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                //查找是否存在同步屏障，通过postSyncBarrier方法发送。消息不为空，target为null则停止同步消息，异步消息继续执行
                //target为空一般都是消息屏障跟异步消息结合使用，直到在队列中找到下一个异步消息
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.//跳出while循环的情况：1，队列中不存在异步消息，全是同步消息，msg为null也就是指向队列末尾了，跳出while循环。2，队列中存在异步消息，从头找到第一个异步消息，赋值给msg后，跳出while循环。
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.当前消息执行时间未到，设置一个唤醒执行的时间
                        // 如果消息此刻还没有到时间，设置一下阻塞时间nextPollTimeoutMillis，进入下次循环的时候会调用nativePollOnce(ptr, nextPollTimeoutMillis)进行阻塞；
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.正常获取消息，为false代表目前无阻塞
                        mBlocked = false;
                        //prevMsg不为空，表示存在同步屏障和异步消息，当前msg为异步消息
                        if (prevMsg != null) {
                            //对应上面的有同步屏障的情况，异步消息需要取出来执行，则preMsg(前一个)的next要指向msg(当前消息)的next
                            prevMsg.next = msg.next;
                        } else {
                            //为空则表示不存在同步屏障，更新Message（头节点）为msg（当前消息）的next
                            mMessages = msg.next;
                        }
                        //断开即将执行的msg
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        //标记为使用状态
                        msg.markInUse();
                        //返回取出的消息，交给loop处理
                        return msg;
                    }
                } else {
                    // No more messages.，无消息会阻塞
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                //当Message为空，或者当前Message执行时间还未到pendingIdleHandlerCount前面默认可以看到设置值为-1.所以给pendingIdleHandlerCount赋值为mIdleHandlers空闲handles的数量
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                //如果不存在闲置的handle，也就是没有Looper.myQueue().addIdleHandler(IdleHandler),表示当前可能没有msg,或者msg没有到执行时间，或者闲置handle为0，这种条件下，阻塞。
                //继续走for循环，这个时候前面重新计算了nextPollTimeoutMillis，所以nativepollonce会继续阻塞等待唤醒
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                //默认空置handler至少4个.
                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    //返回的Boolean值用于下面的判断
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                //如果返回false，则只执行该handler一次，然后移除，如果为true，则为多次，每次都会执行到，没有移除操作
                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```



##### 发送同步屏障

该消息用于阻碍同步Message消息执行，优先执行异步的Message。特点是变量 target 为 null，在 `next()` 方法中有用到。

```java
 public int postSyncBarrier() {
        return postSyncBarrier(SystemClock.uptimeMillis());
    }

    private int postSyncBarrier(long when) {
        // Enqueue a new sync barrier token.
        // We don't need to wake the queue because the purpose of a barrier is to stall it.
        synchronized (this) {
            final int token = mNextBarrierToken++;
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;

            Message prev = null;
            Message p = mMessages;
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            return token;
        }
    }
```





1.当首次进入或所有消息队列已经处理完成，由于此刻队列中没有消息（mMessages为null），这时nextPollTimeoutMillis = -1 ，然后会处理一些不紧急的任务（IdleHandler），之后线程会一直阻塞，直到被主动唤醒（插入消息后根据消息类型决定是否需要唤醒）。
2.读取列表中的消息，如果发现消息屏障，则跳过后面的同步消息。
3.如果拿到的消息还没有到时间，则重新赋值nextPollTimeoutMillis = 延时的时间，线程会阻塞，直到时间到后自动唤醒
4.如果消息是及时消息或延时消息的时间到了，则会返回此消息给looper处理。

#### CPP中的nativeWake和nativepollonce

#### `nativeWake`

```c++
void NativeMessageQueue::wake() {
    mLooper->wake();
}

void Looper::wake() {
    uint64_t inc = 1;
    ssize_t nWrite = TEMP_FAILURE_RETRY(write(mWakeEventFd, &inc, sizeof(uint64_t)));
    if (nWrite != sizeof(uint64_t)) {
        if (errno != EAGAIN) {
            LOG_ALWAYS_FATAL("Could not write wake signal to fd %d: %s",
                    mWakeEventFd, strerror(errno));
        }
    }
}
```

#### `nativePollOnce`:

```c++
void NativeMessageQueue::pollOnce(JNIEnv* env, jobject pollObj, int timeoutMillis) {
    mPollEnv = env;
    mPollObj = pollObj;
    mLooper->pollOnce(timeoutMillis);
    mPollObj = NULL;
    mPollEnv = NULL;

    if (mExceptionObj) {
        env->Throw(mExceptionObj);
        env->DeleteLocalRef(mExceptionObj);
        mExceptionObj = NULL;
    }
}

int Looper::pollOnce(int timeoutMillis, int* outFd, int* outEvents, void** outData) {
    int result = 0;
    for (;;) {
        while (mResponseIndex < mResponses.size()) {
            const Response& response = mResponses.itemAt(mResponseIndex++);
            int ident = response.request.ident;
            if (ident >= 0) {
                int fd = response.request.fd;
                int events = response.events;
                void* data = response.request.data;
                if (outFd != NULL) *outFd = fd;
                if (outEvents != NULL) *outEvents = events;
                if (outData != NULL) *outData = data;
                return ident;
            }
        }

        if (result != 0) {
            if (outFd != NULL) *outFd = 0;
            if (outEvents != NULL) *outEvents = 0;
            if (outData != NULL) *outData = NULL;
            return result;
        }

        result = pollInner(timeoutMillis);
    }
}

int Looper::pollInner(int timeoutMillis) {
    // Adjust the timeout based on when the next message is due.
    if (timeoutMillis != 0 && mNextMessageUptime != LLONG_MAX) {
        nsecs_t now = systemTime(SYSTEM_TIME_MONOTONIC);
        int messageTimeoutMillis = toMillisecondTimeoutDelay(now, mNextMessageUptime);
        if (messageTimeoutMillis >= 0
                && (timeoutMillis < 0 || messageTimeoutMillis < timeoutMillis)) {
            timeoutMillis = messageTimeoutMillis;
        }
    }

    // Poll.
    int result = POLL_WAKE;
    mResponses.clear();
    mResponseIndex = 0;

    // We are about to idle.
    mPolling = true;

    struct epoll_event eventItems[EPOLL_MAX_EVENTS];
    // 这里重点
    int eventCount = epoll_wait(mEpollFd, eventItems, EPOLL_MAX_EVENTS, timeoutMillis);

    // No longer idling.
    mPolling = false;

    // Acquire lock.
    mLock.lock();

    // Rebuild epoll set if needed.
    if (mEpollRebuildRequired) {
        mEpollRebuildRequired = false;
        rebuildEpollLocked();
        goto Done;
    }

    // Check for poll error.
    if (eventCount < 0) {
        if (errno == EINTR) {
            goto Done;
        }
        ALOGW("Poll failed with an unexpected error: %s", strerror(errno));
        result = POLL_ERROR;
        goto Done;
    }

    // Check for poll timeout.
    if (eventCount == 0) {
        result = POLL_TIMEOUT;
        goto Done;
    }

    // Handle all events.
    for (int i = 0; i < eventCount; i++) {
        int fd = eventItems[i].data.fd;
        uint32_t epollEvents = eventItems[i].events;
        if (fd == mWakeEventFd) {
            if (epollEvents & EPOLLIN) {
                awoken();
            } else {
                ALOGW("Ignoring unexpected epoll events 0x%x on wake event fd.", epollEvents);
            }
        } else {
            ssize_t requestIndex = mRequests.indexOfKey(fd);
            if (requestIndex >= 0) {
                int events = 0;
                if (epollEvents & EPOLLIN) events |= EVENT_INPUT;
                if (epollEvents & EPOLLOUT) events |= EVENT_OUTPUT;
                if (epollEvents & EPOLLERR) events |= EVENT_ERROR;
                if (epollEvents & EPOLLHUP) events |= EVENT_HANGUP;
                pushResponse(events, mRequests.valueAt(requestIndex));
            } else {
                ALOGW("Ignoring unexpected epoll events 0x%x on fd %d that is "
                        "no longer registered.", epollEvents, fd);
            }
        }
    }
Done: ;

    // Invoke pending message callbacks.
    mNextMessageUptime = LLONG_MAX;
    while (mMessageEnvelopes.size() != 0) {
        nsecs_t now = systemTime(SYSTEM_TIME_MONOTONIC);
        const MessageEnvelope& messageEnvelope = mMessageEnvelopes.itemAt(0);
        if (messageEnvelope.uptime <= now) {
            // Remove the envelope from the list.
            // We keep a strong reference to the handler until the call to handleMessage
            // finishes.  Then we drop it so that the handler can be deleted *before*
            // we reacquire our lock.
            { // obtain handler
                sp<MessageHandler> handler = messageEnvelope.handler;
                Message message = messageEnvelope.message;
                mMessageEnvelopes.removeAt(0);
                mSendingMessage = true;
                mLock.unlock();
                handler->handleMessage(message);
            } // release handler

            mLock.lock();
            mSendingMessage = false;
            result = POLL_CALLBACK;
        } else {
            // The last message left at the head of the queue determines the next wakeup time.
            mNextMessageUptime = messageEnvelope.uptime;
            break;
        }
    }

    // Release lock.
    mLock.unlock();

    // Invoke all response callbacks.
    for (size_t i = 0; i < mResponses.size(); i++) {
        Response& response = mResponses.editItemAt(i);
        if (response.request.ident == POLL_CALLBACK) {
            int fd = response.request.fd;
            int events = response.events;
            void* data = response.request.data;
            // Invoke the callback.  Note that the file descriptor may be closed by
            // the callback (and potentially even reused) before the function returns so
            // we need to be a little careful when removing the file descriptor afterwards.
            int callbackResult = response.request.callback->handleEvent(fd, events, data);
            if (callbackResult == 0) {
                removeFd(fd, response.request.seq);
            }

            // Clear the callback reference in the response structure promptly because we
            // will not clear the response vector itself until the next poll.
            response.request.callback.clear();
            result = POLL_CALLBACK;
        }
    }
    return result;
}

```





**Handler.sendMessageDelayed()怎么实现延迟的？**
前面我们分析了如果拿到的消息还**没有到时间**，则会**重新设置超时**时间并赋值给`nextPollTimeoutMillis`，然后调用`nativePollOnce(ptr, nextPollTimeoutMillis)`进行阻塞，这是一个本地方法，会调用底层C++代码，`C++`代码最终会通过`Linux的epoll`监听`文件描述符的写入事件`来实现延迟的。

**Looper.loop是一个死循环，拿不到需要处理的Message就会阻塞，那在UI线程中为什么不会导致ANR？**

首先我们来看造成ANR的原因：
 1.当前的事件没有机会得到处理（即主线程正在处理前一个事件，没有及时的完成或者looper被某种原因阻塞住了）
 2.当前的事件正在处理，但没有及时完成

APP的入口ActivityThread的main方法：如果main方法中没有looper进行死循环，那么主线程一运行完毕就会退出，会导致直接崩溃，还玩什么！

**现在我们知道了消息循环的必要性，那为什么这个死循环不会造成ANR异常呢？**

我们知道Android 的是由事件驱动的，looper.loop() 不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在 Looper的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它，这也就是我们为什么不能在UI线程中处理耗时操作的原因。
 主线程Looper从消息队列读取消息，当读完所有消息时，主线程阻塞。子线程往消息队列发送消息，唤醒主线程，主线程被唤醒只是为了读取消息，当消息读取完毕，再次睡眠。因此loop的循环并不会对CPU性能有过多的消耗。



![](.\images\MessageQueue.next()同步屏障消息.png)





**总结**：这就是通过obtain方法获取Message对象的详情。通过obtain方法获取Message对象使得Message得到了重复的利用，减少了每次获取Message时去申请空间的时间。同时，这样也不会永无止境的去创建新对象，减小了Jvm垃圾回收的压力，提高了效率。



## Handler的消息屏障

##### 消息的分类

**Message分为3中：普通消息（同步消息）、屏障消息（同步屏障）和异步消息**。

我们通常使用的都是普通消息，而屏障消息就是在消息队列中插入一个屏障，在屏障之后的所有普通消息都会被挡着，不能被处理。不过异步消息却例外，屏障不会挡住异步消息。因此可以这样认为：屏障消息就是为了确保异步消息的优先级，设置了屏障后，只能处理其后的异步消息，同步消息会被挡住，除非撤销屏障。

源码中例子：`ActivityThread#handleResumeActivity()`中的`viewRootImpl.setView`方法。为了加快UI的渲染。

#### Handler消息机制，

Handler消息机制，在Java层的MessageQueue中，调用了native方法实现消息循环的休眠和唤醒（也就是线程的休眠和唤醒）。native方法nativePollOnce实现了消息循环的休眠，而方法nativeWake实现了消息循环的唤醒。

在native层，消息循环的休眠和唤醒使用了Linux内核的epoll机制和pipe。下面我们简单介绍一下epool机制是什么？

#### 文件描述符fd(句柄)：

​         在Linux系统中一切皆可以看成是文件，文件又可分为：普通文件、目录文件、链接文件和设备文件。`文件描述符（file descriptor）`是内核为了高效管理已被`打开的文件`所创建的`索引`，其是一个非负整数（通常是小整数），用于指代被打开的文件，**所有执行I/O操作的系统调用都通过文件描述符。**

​    	在linux系统中，设备也是以文件的形式存在，**要对该设备进行操作就必须先打开这个文件**，**打开文件就会获得文件描述符**，它是个很小的正整数。每个进程在PCB（Process Control Block）中保存着一份文件描述符表，文件描述符就是这个表的索引，每个表项都有一个指向已打开文件的指针。文件描述符的优点：兼容POSIX标准，许多Linux和UNIX系统调用都依赖于它。文件描述符的缺点：**不能移植到UNIX以外的系统上去，也不直观。**

#### pipe（管道）

​        在Linux上，使用POSIX的condition+mutex，也能完成线程间通知，但是这种方式在linux中用得相对较少，而是`大量使用pipe，创建两个fd(writeFD,readFD)`。当线程1想唤醒线程2的时候，就可以往writeFD中写数据，这样线程2阻塞在readFD中就能返回。

  	为何要使用pipe？因为Linux上阻塞的方法就是用select，poll和epoll，其中等待的都是`FD`，那么采用FD这种方式，能够统一调用方法。

​		pipe是Linux中最基本的一种IPC机制，可以用来实现进程、线程间通信

​	**pipe特点：**

- 调用pipe系统函数即可创建一个管道。
- 其本质是一个伪文件(实为内核缓冲区)。
- 由两个文件描述符引用，一个表示读端，一个表示写端。
- 规定数据从管道的写端流入管道，从读端流出。

#### Handler机制中的使用：

首先使用pipe创建两个fd：writeFD、readFD。当线程1想唤醒线程2的时候，就可以往writeFD中写数据，这样线程2阻塞在readFD中就能返回。

也就是说，当文件描述符指向的文件（内核缓冲区）为空时，则线程进行休眠。当另一个线程向缓冲区写入内容时，则将当前线程进行唤醒。

#### Linux上阻塞的方法

首先我们来定义流的概念，一个流可以是文件，socket，pipe等等可以进行I/O操作的内核对象。

不管是文件，还是套接字，还是管道，我们都可以把他们看作流。

I/O的操作，通过read，我们可以从流中读入数据；通过write，我们可以往流写入数据。现在假定一个情形，我们需要从流中读数据，但是流中还没有数据，（典型的例子为，客户端要从socket读如数据，但是服务器还没有把数据传回来），这时候该怎么办？

- 阻塞：阻塞是个什么概念呢？比如某个时候你在等快递，但是你不知道快递什么时候过来，而且你没有别的事可以干（或者说接下来的事要等快递来了才能做）；那么你可以去睡觉了，因为你知道快递把货送来时一定会给你打个电话（假定一定能叫醒你）。

  `经济而简单，经济是指消耗很少的CPU时间，如果线程睡眠了，就掉出了系统的调度队列，暂时不会去瓜分CPU宝贵的时间片了。`

- 非阻塞忙轮询：接着上面等快递的例子，如果用忙轮询的方法，那么你需要知道快递员的手机号，然后每分钟给他挂个电话："你到了没？"(浪费时间、资源，效率低)

#### 缓冲区

为了了解阻塞是如何进行的，我们来讨论缓冲区，以及内核缓冲区，最终把I/O事件解释清楚。

缓冲区的引入是为了减少频繁I/O操作而引起频繁的系统调用（你知道它很慢的），当你操作一个流时，更多的是以缓冲区为单位进行操作，这是相对于用户空间而言。对于内核来说，也需要缓冲区。

假设有一个管道，进程A为管道的写入方，Ｂ为管道的读出方。

1. 假设一开始内核缓冲区是空的，B作为读出方，被阻塞着。然后首先A往管道写入，这时候内核缓冲区由空的状态变到非空状态，内核就会产生一个事件告诉Ｂ该醒来了，这个事件姑且称之为`缓冲区非空`。
2. 但是`缓冲区非空`事件通知B后，B却还没有读出数据；且内核许诺了不能把写入管道中的数据丢掉这个时候，Ａ写入的数据会滞留在内核缓冲区中，如果内核也缓冲区满了，B仍未开始读数据，最终内核缓冲区会被填满，这个时候会产生一个I/O事件，告诉进程A，你该等等（阻塞）了，该事件可认为`缓存区已满`
3. 假设后来Ｂ终于开始读数据了，于是内核的缓冲区空了出来，这时候内核会告诉A，内核缓冲区有空位了，你可以从长眠中醒来了，继续写数据了，我们把这个事件Y1叫做`缓冲区非满`。
4. 也许事件Y1已经通知了A，但是A也没有数据写入了，而Ｂ继续读出数据，直到内核缓冲区空了。这个时候内核就告诉B，你需要阻塞了！，我们把这个时间定为`缓冲区空`。

**四个I/O事件是进行阻塞同步的根本。**

#### select/poll

然后我们来说说阻塞I/O的缺点。阻塞I/O模式下，一个线程只能处理一个流的I/O事件。如果想要同时处理多个流，要么多进程(fork)，要么多线程(pthread_create)，很不幸这两种方法效率都不高。

于是再来考虑**非阻塞忙轮询的I/O方式**，我们发现我们**可以同时处理多个流**

```java
while true {
    for i in stream[]; {
        if i has data
            read until unavailable
    }
}
```

我们只要不停的把所有流从头到尾问一遍，又从头开始。这样就可以处理多个流了，但这样的做法显然不好，因为如果所有的流都没有数据，那么只会白白浪费CPU。这里要补充一点，`阻塞模式下`，内核对于I/O事件的处理是`阻塞或者唤醒`，而`非阻塞模式下`则把`I/O事件交给其他对象`（后文介绍的select以及epoll）处理甚至直接忽略。

​    为了避免CPU空转，可以引进了一个**代理**（一开始有一位叫做`select`的代理，后来又有一位叫做`poll`的代理，不过两者的本质是一样的）。这个代理比较厉害，可以同时观察许多流的I/O事件，在`空闲`的时候，会把当前线程`阻塞`掉，当有`一个或多个流有I/O事件时`，就从阻塞态中`醒来`，于是我们的程序就会轮询一遍所有的流（于是我们可以把"忙"字去掉了）。代码长这样:

```java
while true {
    select(streams[])
    for i in streams[] {
        if i has data
            read until unavailable
    }
}

```

于是，如果`没有I/O事件`产生，我们的程序就会`阻塞在select处`。但是依然有个问题，我们从`select`那里仅仅知道了，有I/O事件发生了，但却并不知道是那几个流（可能有一个，多个，甚至全部），我们只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。

但是使用select，我们有O(n)的无差别轮询复杂度，同时处理的流越多，每一次无差别轮询时间就越长。再次说了这么多，终于轮到epoll了。终极大法

## epoll

**epoll可以理解为event poll，不同于忙轮询和无差别轮询，epoll会把哪个流发生了怎样的I/O事件通知我们。此时我们对这些流的操作都是有意义的。（复杂度降低到了O(1)）**

**`epoll`支持四类事件: **

> EPOLLIN(句柄可读)、
>
> EPOLLOUT(句柄可写),
>
> EPOLLERR(句柄错误)、
>
> EPOLLHUP(句柄断)。

 在讨论epoll的实现细节之前，先把`epoll`的相关操作列出：

- `epoll_create(int size) `

  创建一个`epoll`对象，该方法生成一个epoll专用的文件描述符。它其实是在内核申请一空间，用来存放你想关注的fd上是否发生以及发生了什么事件；size就是你在这个epoll fd上能关注的最大fd数。例如：epfd= epoll_create(10)。

- `epoll_ctl(int epfd, int op, int fd, struct epoll_event * event)` 
  
  用于控制epoll文件描述符上的事件，可以注册事件，修改事件，删除事件。（`epoll_add/epoll_del`的合体），往`epoll`对象中增加/删除某一个流的某一个事件。
  
  1、epfd：由epoll调用产生的文件描述符（epoll_create的返回值）；
  2、op：操作的类型。例如注册事件，可能的取值EPOLL_CTL_ADD（注册）、EPOLL_CTL_MOD（修改）、  EPOLL_CTL_DEL（删除）；
  3、fd：op实施的对象（关联的文件描述符）；
  4、event：事件类型（指向epoll_event的指针）；
  5、如果调用成功返回0，失败返回-1。
  
  - `epoll_ctl(epollfd, EPOLL_CTL_ADD, socket, EPOLLIN);`//注册缓冲区非空事件，即有数据流入
  - `epoll_ctl(epollfd, EPOLL_CTL_DEL, socket, EPOLLOUT);`//注册缓冲区非满事件，即流可以被写入
  - `epoll_ctl(epollfd, EPOLL_CTL_MOD, socket, EPOLLOUT);`//修改缓冲区非满事件，即流可以被修改
  
- `epoll_wait(int epfd, struct epoll_event * events, intmaxevents, int timeout)`：等待直到注册的事件发生

  1、参数events用来从内核得到事件的集合。
  2、maxevents告之内核这个events有多大(数组成员的个数)。
  3、参数timeout是超时时间（毫秒）。0表示立即返回，-1表示永久阻塞）。
  4、该函数返回需要处理的事件数目，如返回0表示已超时。
  5、返回的事件集合在events数组中，数组中实际存放的成员个数是函数的返回值。

（注：当对一个非阻塞流的读写发生缓冲区满或缓冲区空，write/read会返回-1，并设置errno=EAGAIN。而epoll只关心缓冲区非满和缓冲区非空事件。）

需要注意的是，当创建好epoll句柄后，它就是会占用一个fd值，可以在linux下/proc/进程id/fd/看到这个fd，所以在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

一个epoll模式的代码大概的样子是：

```java
while true {
    active_stream[] = epoll_wait(epollfd)
    for i in active_stream[] {
        read or write till
    }
}
```

##### 总结：

1、Handler消息机制，在Java层的MessageQueue中，调用了native方法实现消息循环的休眠和唤醒（也就是线程的休眠和唤醒）。native方法nativePollOnce实现了消息循环的休眠，而方法nativeWake实现了消息循环的唤醒。
2、在native层，消息循环的休眠和唤醒使用了Linux内核的epoll机制和pipe。
3、在Linux系统内核中，所有执行I/O操作的系统调用都会通过文件描述符。内核利用文件描述符来访问文件，文件描述符是非负整数。
4、pipe是Linux中最基本的一种IPC机制，可以用来实现进程、线程间通信。
5、pipe其本质是一个伪文件(实为内核缓冲区) ，它有两个文件描述符引用，一个表示读端，一个表示写端，规定数据从管道的写端流入管道，从读端流出。
6、Handler机制中，底层使用pipe创建两个fd：writeFD、readFD。当线程A想唤醒线程B的时候，就可以往writeFD中写数据，这样线程B阻塞在readFD中就能返回。 也就是说，当文件描述符指向的文件（内核缓冲区）为空时，则线程进行休眠。当另一个线程向缓冲区写入内容时，则将当前线程进行唤醒。
7、epoll机制是Linux最高效的I/O复用机制，在一处等待多个文件句柄的I/O事件。
8、nativeInit()方法是一个native方法，它返回了一个int值，作为native层在Java层的一个指针引用。
9、Java层中MessageQueue的mPtr属性，是native层的NativeMessageQueue对象的指针。
10、native层也对应了一套Handler机制相关的类：Looper、MessageQueue、MessageHandler、Message，它们与Java层相对应。
11、NativeMessageQueue构造函数中，获取一个线程的Looper对象，如果不存在在创建一个，并关联到线程。这个操作跟Java层非常类似，不过Java层是Looper创建了MessageQueue，而native层正好相反。
12、Looper构造函数中，初始化了一个管道，用于唤醒Looper。mWakeEventFd是一个android::base::unique_fd类型，它的内部实现了pipe管道的创建。
13、rebuildEpollLocked()方法负责创建一个epoll对象，然后将文件描述符事件添加到epoll监控，这里添加的是可读事件，当文件流可读时，则唤醒线程。
14、Looper对象的pollOnce方法实现线程的休眠操作，它的内部也定义了一个无限for循环，来处理native层的事件，以及通过epoll监控的文件描述符相关事件。
15、线程唤醒操作，是Looper的wake方法实现的，它只是向文件描述符mWakeEventFd中，写入数据1即可。
16、线程唤醒后，调用awoken方法，清空文件描述符mWakeEventFd中的数据，以便重新使用。

#### `nativePollOnce` 是什么

`nativePollOnce` 方法用于“等待”, 直到下一条消息可用为止. 如果在此调用期间花费的时间很长, 则您的主线程没有实际工作要做, 而是等待下一个事件处理.无需担心.

**说明:**

因为主线程负责绘制 UI 和处理各种事件, 所以主线程拥有一个处理所有这些事件的循环. 该循环由 `Looper` 管理, 其工作非常简单: 它处理 `MessageQueue` 中的所有 `Message`.
例如, 响应于输入事件, 将消息添加到队列, 帧渲染回调, 甚至您的 `Handler.post` 调用. 有时主线程无事可做（即队列中没有消息), 例如在完成渲染单帧之后(线程刚绘制了一帧, 并准备好下一帧, 等待适当的时间). `MessageQueue` 类中的两个 Java 方法对我们很有趣: `Message next()`和 `boolean enqueueMessage(Message, long)`. 顾名思义, `Message next()` 从队列中获取并返回下一个消息. 如果队列为空(无返回值), 则该方法将调用 `native void nativePollOnce(long, int)`, 该方法将一直阻塞直到添加新消息为止. 此时,您可能会问`nativePollOnce` 如何知道何时醒来. 这是一个很好的问题. 当将 `Message` 添加到队列时, 框架调用 `enqueueMessage` 方法, 该方法不仅将消息插入队列, 而且还会调用`native static void nativeWake(long)`. `nativePollOnce` 和 `nativeWake` 的核心魔术发生在 native 代码中. native `MessageQueue` 利用名为 `epoll` 的 Linux 系统调用, 该系统调用可以监视文件描述符中的 IO 事件. `nativePollOnce` 在某个文件描述符上调用 `epoll_wait`, 而 `nativeWake` 写入一个 IO 操作到描述符, `epoll_wait` 等待. 然后, 内核从等待状态中取出 `epoll` 等待线程, 并且该线程继续处理新消息. 如果您熟悉 Java 的 `Object.wait()`和 `Object.notify()`方法,可以想象一下 `nativePollOnce` 大致等同于 `Object.wait()`, `nativeWake` 等同于 `Object.notify()`,但它们的实现完全不同: `nativePollOnce` 使用 `epoll`, 而 `Object.wait` 使用 `futex` Linux 调用. 值得注意的是, `nativePollOnce` 和 `Object.wait` 都不会浪费 CPU 周期, 因为当线程进入任一方法时, 出于线程调度的目的, 该线程将被禁用(引用Object类的javadoc). 但是, 某些事件探查器可能会错误地将等待 `epoll` 等待(甚至是 Object.wait)的线程识别为正在运行并消耗 CPU 时间, 这是不正确的. 如果这些方法实际上浪费了 CPU 周期, 则所有空闲的应用程序都将使用 100％ 的 CPU, 从而加热并降低设备速度.



**Question:**

##### handler消息循环机制、View.post()与handler.post()的区别

##### 什么是handler消息屏障？消息屏障与正常message的区别是什么？消息屏障的作用及原理是什么？

##### message复用原理为什么高效obtain（）

单链表，不用创建新的内存空间以及创建会耗费时间

##### 主线程的死循环一直运行是不是特别消耗CPU资源呢？

其实不然，这里就涉及到**Linux pipe/epoll机制**，简单说就是在主线程的`MessageQueue`没有消息时，便阻塞在`loop`的`queue.next()`中的`nativePollOnce()`方法里，详情见[Android消息机制1-Handler(Java层)](https://link.zhihu.com/?target=http://www.yuanhh.com/2015/12/26/handler-message-framework/%23next)，此时主线程会释放CPU资源进入休眠状态，直到下个消息到达或者有事务发生，通过往`pipe`管道写端写入数据来唤醒主线程工作。这里采用的`epoll机制`，是一种`IO多路复用机制`，可以`同时监控多个描述符`，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 **所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。**

### Reference & Thanks {docsify-ignore}

[Handler.post和View.post的区别](https://www.jianshu.com/p/7280b2d3b4d1)

[View的onAttachedToWindow和onDetachedFromWindow的调](https://www.jianshu.com/p/e7b6fa788ae6)

[彻底明白Android消息机制的原理及源码](https://blog.csdn.net/weixin_42316079/article/details/112182628)

[MessageQueue#next() 方法图解](https://blog.csdn.net/qq_21586317/article/details/88780077)

[Handler消息Message屏障消息](https://blog.csdn.net/mirkowug/article/details/115091337)

[Handler之消息屏障你应该知道的](https://blog.csdn.net/my_csdnboke/article/details/109531168)

[你应该要知道的handler消息屏障](https://blog.csdn.net/lggisking/article/details/108091203)

[【Android自助餐】Handler消息机制完全解析（一）Message中obtain()与recycle()的来龙去脉](https://blog.csdn.net/xmh19936688/article/details/51901338)

[Handler消息机制之深入理解Message.obtain()](https://blog.csdn.net/chenbaige/article/details/79473475)

https://blog.csdn.net/chenbaige/article/details/79473475)

[Handler详解4-epoll、looper.loop主线程阻塞](https://www.cnblogs.com/muouren/p/11706457.html)

[Android 中 MessageQueue 的 nativePollOnce ](https://www.cnblogs.com/jiy-for-you/p/11707356.html)

[Android Q消息循环：休眠、唤醒的底层原理及native层源码分析](https://blog.csdn.net/u011578734/article/details/106241075)