# AIDL和Binder跨进程通信

## 一、Binder基于C/S模型

##### 客户端实现：remoteBinder.transact（），IBinder的主要API是transact()

##### 客户端向transact向Binder驱动发送消息，向远端IBinder发送消息

##### Binder.onTransact()在远程的service中能响应接收到消息

##### transact()和onTransact()是同步执行，transact（）内部调用了onTransact()再返回

##### mRemote代表着Binder对象的本地代理

### 1、Binder到底是什么？

- 通常意义下，`Binder`指的是一种通信机制；我们说AIDL使用Binder进行通信，指的就是**Binder这种IPC机制**。
- 对于`Server`进程来说，`Binder`指的是**Binder本地对象**
- 对于`Client`来说，`Binder`指的是**Binder代理对象**，它只是**Binder本地对象**的一个远程代理；对这个`Binder`代理对象的操作，会通过驱动最终转发到Binder本地对象上去完成；对于一个拥有`Binder`对象的使用者而言，它无须关心这是一个`Binder`代理对象还是`Binder`本地对象；对于代理对象的操作和对本地对象的操作对它来说没有区别。
- 对于传输过程而言，`Binder`是可以进行跨进程传递的对象；**Binder驱动**会对具有跨进程传递能力的对象做特殊处理：自动完成代理对象和本地对象的转换。

##### 1.1、我们来了解下socket

**目前`linux`支持的`IPC`包括`传统的管道`，`System V IPC`，`即消息队列/共享内存/信号量`，以及`socket`。**

`socket`作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。

其中只有`socket`支持`Client-Server`的通信方式。当然也可以在这些底层机制上架设一套协议来实现`Client-Server`通信，但这样增加了系统的复杂性，在手机这种条件复杂，资源稀缺的环境下可靠性也难以保证。
另一方面是传输性能。`socket`作为一款通用接口，其传输效率低，开销大，主要用在跨网络的进程间通信和本机上进程间的低速通信。消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的缓存区中，然后再从内核缓存区拷贝到接收方缓存区，至少有两次拷贝过程。共享内存虽然无需拷贝，但控制复杂，难以使用。

##### 1.2、内核模块/驱动

通过系统调用，用户空间可以访问内核空间，那么如果一个用户空间想与另外一个用户空间进行通信怎么办呢？很自然想到的是让操作系统内核添加支持；传统的Linux通信机制，比如Socket，管道等都是内核支持的；但是Binder并不是Linux内核的一部分，它是怎么做到访问内核空间的呢？Linux的动态可加载内核模块（Loadable Kernel Module，LKM）机制解决了这个问题；模块是具有独立功能的程序，它可以被单独编译，但不能独立运行。它在运行时被链接到内核作为内核的一部分在内核空间运行。这样，Android系统可以通过添加一个内核模块运行在内核空间，用户进程之间的通过这个模块作为桥梁，就可以完成通信了。

### 2、IBinder/IInterface/Binder/BinderProxy/Stub的关系

创建`AIDL`文件，生成默认的实现静态内部类`Default、Stub、Proxy`。我们使用`AIDL`接口的时候，经常会接触到这些类，那么这每个类代表的是什么呢？

- `IBinder`是一个接口，它代表了**一种跨进程传输的能力**；只要实现了这个接口，就能将这个对象进行跨进程传递；这是驱动底层支持的；在跨进程数据流经驱动的时候，驱动会识别`IBinder`类型的数据，从而自动完成不同进程`Binder`本地对象以及`Binder`代理对象的转换。
- `IBinder`负责数据传输，那么`client与server端`的调用契约（这里不用接口避免混淆）呢？这里的`IInterface`代表的就是远程`server`对象具有什么能力。具体来说，就是`aidl`里面的接口。
- `Java`层的`Binder`类，代表的其实就是**Binder本地对象**。`BinderProxy`类是`Binder`类的一个内部类，它代表远程进程的`Binder`对象的本地代理；这两个类都继承自IBinder, 因而都具有跨进程传输的能力；实际上，在跨越进程的时候，`Binder`驱动会自动完成这两个对象的转换。
- 在使用`AIDL`的时候，编译工具会给我们生成一个`Stub`的静态内部类；这个类继承了`Binder`, 说明它是一个`Binder`本地对象，它实现了`IInterface`接口，表明它具有远程Server承诺给Client的能力；`Stub`是一个抽象类，具体的`IInterface`的相关实现需要我们手动完成，这里使用了策略模式。

#### 1、AIDL支持的数据类型：

- 八种基本数据类型：byte、char、short、int、long、float、double、boolean
- String，CharSequence
- 实现了Parcelable接口的数据类型
- List 类型。List承载的数据必须是AIDL支持的类型，或者是其它声明的AIDL对象
- Map类型。Map承载的数据必须是AIDL支持的类型，或者是其它声明的AIDL对象

#### 2、AIDL文件可以分为两类

* 一类用来声明实现了Parcelable接口的数据类型，以供其他AIDL文件使用那些非默认支持的数据类型。
* 还有一类是用来定义接口方法，声明要暴露哪些接口给客户端调用，定向Tag就是用来标注这些方法的参数值。

####  3、什么是定向Tag

* 定向Tag表示在跨进程通信中数据的流向，用于标注方法的参数值，分为 in、out、inout 三种。

* 其中 in 表示数据只能由客户端流向服务端.
* out 表示数据只能由服务端流向客户端，
* 而 inout 则表示数据可在服务端与客户端之间双向流通。
* 此外，如果AIDL方法接口的参数值类型是：基本数据类型、String、CharSequence或者其他AIDL文件定义的方法接口，那么这些参数值的定向 Tag 默认是且只能是 in，所以除了这些类型外，其他参数值都需要明确标注使用哪种定向Tag。

#### 4、明确导包

在AIDL文件中需要明确标明引用到的数据类型所在的包名，即使两个文件处在同个包名下。



### 完整的AIDL生成文件

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.cc.xiangxue;
// Declare any non-default types here with import statements

public interface ITest extends android.os.IInterface {
    /**
     * Default implementation for ITest.
     */
    public static class Default implements com.cc.xiangxue.ITest {
        /**
         * Demonstrates some basic types that you can use as parameters
         * and return values in AIDL.
         */
        @Override
        public java.lang.String test(java.lang.String str1, int age1) throws android.os.RemoteException {
            return null;
        }

        @Override
        public android.os.IBinder asBinder() {
            return null;
        }
    }

    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.cc.xiangxue.ITest {
        private static final java.lang.String DESCRIPTOR = "com.cc.xiangxue.ITest";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.cc.xiangxue.ITest interface,
         * generating a proxy if needed.
         */
        public static com.cc.xiangxue.ITest asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.cc.xiangxue.ITest))) {
                return ((com.cc.xiangxue.ITest) iin);
            }
            return new com.cc.xiangxue.ITest.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_test: {
                    data.enforceInterface(descriptor);
                    java.lang.String _arg0;
                    _arg0 = data.readString();
                    int _arg1;
                    _arg1 = data.readInt();
                    java.lang.String _result = this.test(_arg0, _arg1);
                    reply.writeNoException();
                    reply.writeString(_result);
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.cc.xiangxue.ITest {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            /**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public java.lang.String test(java.lang.String str1, int age1) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.lang.String _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeString(str1);
                    _data.writeInt(age1);
                    boolean _status = mRemote.transact(Stub.TRANSACTION_test, _data, _reply, 0);
                    if (!_status && getDefaultImpl() != null) {
                        return getDefaultImpl().test(str1, age1);
                    }
                    _reply.readException();
                    _result = _reply.readString();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            public static com.cc.xiangxue.ITest sDefaultImpl;
        }

        static final int TRANSACTION_test = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);

        public static boolean setDefaultImpl(com.cc.xiangxue.ITest impl) {
            // Only one user of this interface can use this function
            // at a time. This is a heuristic to detect if two different
            // users in the same process use this function.
            if (Stub.Proxy.sDefaultImpl != null) {
                throw new IllegalStateException("setDefaultImpl() called twice");
            }
            if (impl != null) {
                Stub.Proxy.sDefaultImpl = impl;
                return true;
            }
            return false;
        }

        public static com.cc.xiangxue.ITest getDefaultImpl() {
            return Stub.Proxy.sDefaultImpl;
        }
    }

    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    public java.lang.String test(java.lang.String str1, int age1) throws android.os.RemoteException;
}

```



可以看到`Stub`类和`Stub.Proxy`类都实现了`IUserManager`接口，这就是一个典型的代理模式，它们的`getUser`方法有着不同的实现，`Stub`类它将会在远程的服务端完成`getUser`方法的具体实现，而`Stub.Proxy`类是本地客户端的一个代理类，它已经替我们默认的实现了`getUser`方法，该方法里面通过`mRemote`这个`Binder`引用的`transact`方法把请求通过**驱动**发送给服务端，我们注意到`mRemote`发送请求时还传进了`TRANSACTION_getUser`这个代表着`getUser`方法的标识名，这表示客户端告诉服务端我要调用`getUser`这个方法，当驱动把请求转发给服务端后，服务端的`Stub`类的`onTransact`方法就会回调，它里面有一个`switch`语句，根据`code`来调用不同的方法，这时它就会走到`case TRANSACTION_getUser`这个分支，然后调用`getUser`方法的在服务端的具体实现，如果有返回值的话，还会通过`reply`返回给客户端，这样就通过`Binder`驱动完成了一次远程方法调用(RPC)。


```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
 
    private EditText edit_num;
    private Button btn_query;
    private TextView txt_result;
    private IBinder mIBinder;
    private ServiceConnection PersonConnection = new ServiceConnection() {
        @Override
        public void onServiceDisconnected(ComponentName name) {
            mIBinder = null;
        }
 
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mIBinder = service;
        }
    };
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindViews();
 
        // 绑定远程Service
        Intent service = new Intent("android.intent.action.IPCService");
        service.setPackage("com.jay.ipcserver");
        bindService(service, PersonConnection, BIND_AUTO_CREATE);
        btn_query.setOnClickListener(this);
    }
 
    private void bindViews() {
        edit_num = (EditText) findViewById(R.id.edit_num);
        btn_query = (Button) findViewById(R.id.btn_query);
        txt_result = (TextView) findViewById(R.id.txt_result);
    }
 
    @Override
    public void onClick(View v) {
        int num = Integer.parseInt(edit_num.getText().toString());
        if (mIBinder == null) {
            Toast.makeText(this, "未连接服务端或服务端被异常杀死", Toast.LENGTH_SHORT).show();
        } else {
            android.os.Parcel _data = android.os.Parcel.obtain();
            android.os.Parcel _reply = android.os.Parcel.obtain();
            String _result = null;
            try {
                _data.writeInterfaceToken("IPCService");
                _data.writeInt(num);
                mIBinder.transact(0x001, _data, _reply, 0);
                _reply.readException();
                _result = _reply.readString();
                txt_result.setText(_result);
                edit_num.setText("");
            } catch (RemoteException e) {
                e.printStackTrace();
            } finally {
                _reply.recycle();
                _data.recycle();
            }
        }
    }
}
```



##### 服务端实现：onTransact



```java
public class IPCService extends Service {
 
    private static final String DESCRIPTOR = "IPCService";
    private final String[] names = { "B神", "艹神", "基神", "J神", "翔神" };
    private MyBinder mBinder = new MyBinder();
 
    private class MyBinder extends Binder {
        @Override
        protected boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
            switch (code) {
            case 0x001: {
                data.enforceInterface(DESCRIPTOR);
                int num = data.readInt();
                reply.writeNoException();
                reply.writeString(names[num]);
                return true;
            }
            }
            return super.onTransact(code, data, reply, flags);
        }
    }
 
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}
```



## WebView跨进程调用

* 创建接口、实现接口、在实现的接口上添加`AutoService`注解

* `APP`壳子通过`ServiceLoad`加载找到实现类`Activity`，直接跳转到目标`Activity`，为`ModuleA`

* `ModuleA`中创建`AIDL`，生成一个接口`A`，内部方法`exe()`

* 接口`A`的`AIDL`生成的`java`文件实现的静态内部类A，内部类Stub、Proxy，Proxy命名为AProxy，由于是跨进程的情况，那就通过AProxy代理接口A，实现了**Binder和Interface**，**具备跨进程能力和提供服务的能力**。

* `B  extends  ServiceConnection`，在重写`onServiceConnected（ComponentName name, IBinder service）`中

  ```
  A a=A.Stub.asInterface(service);
  //-1
  a.exe()
  ```

* `CClass  extends A.Stub`，重写A的提供的服务`exe()`方法，相当于-1处调用的CClass中的exe()，传递过去要跳转的全路径包名+类名

* CClass中采用了AutoService，可调用AutoService的实现接口D的实现类，DClass，里面有跳转方法和该方法对应的实现类名，通过map存储该实现类名为key，跳转的为Activity路径

* newPage()，exe()内部直接调用该跳转到新activity的方法，跳转之前有AutoService。load(指定的实现类)



## 完整的AIDL生成文件

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.cc.xiangxue;
// Declare any non-default types here with import statements

public interface ITest extends android.os.IInterface {
    /**
     * Default implementation for ITest.
     */
    public static class Default implements com.cc.xiangxue.ITest {
        /**
         * Demonstrates some basic types that you can use as parameters
         * and return values in AIDL.
         */
        @Override
        public java.lang.String test(java.lang.String str1, int age1) throws android.os.RemoteException {
            return null;
        }

        @Override
        public android.os.IBinder asBinder() {
            return null;
        }
    }

    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements com.cc.xiangxue.ITest {
        private static final java.lang.String DESCRIPTOR = "com.cc.xiangxue.ITest";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an com.cc.xiangxue.ITest interface,
         * generating a proxy if needed.
         */
        public static com.cc.xiangxue.ITest asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof com.cc.xiangxue.ITest))) {
                return ((com.cc.xiangxue.ITest) iin);
            }
            return new com.cc.xiangxue.ITest.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_test: {
                    data.enforceInterface(descriptor);
                    java.lang.String _arg0;
                    _arg0 = data.readString();
                    int _arg1;
                    _arg1 = data.readInt();
                    java.lang.String _result = this.test(_arg0, _arg1);
                    reply.writeNoException();
                    reply.writeString(_result);
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements com.cc.xiangxue.ITest {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            /**
             * Demonstrates some basic types that you can use as parameters
             * and return values in AIDL.
             */
            @Override
            public java.lang.String test(java.lang.String str1, int age1) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.lang.String _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeString(str1);
                    _data.writeInt(age1);
                    boolean _status = mRemote.transact(Stub.TRANSACTION_test, _data, _reply, 0);
                    if (!_status && getDefaultImpl() != null) {
                        return getDefaultImpl().test(str1, age1);
                    }
                    _reply.readException();
                    _result = _reply.readString();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            public static com.cc.xiangxue.ITest sDefaultImpl;
        }

        static final int TRANSACTION_test = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);

        public static boolean setDefaultImpl(com.cc.xiangxue.ITest impl) {
            // Only one user of this interface can use this function
            // at a time. This is a heuristic to detect if two different
            // users in the same process use this function.
            if (Stub.Proxy.sDefaultImpl != null) {
                throw new IllegalStateException("setDefaultImpl() called twice");
            }
            if (impl != null) {
                Stub.Proxy.sDefaultImpl = impl;
                return true;
            }
            return false;
        }

        public static com.cc.xiangxue.ITest getDefaultImpl() {
            return Stub.Proxy.sDefaultImpl;
        }
    }

    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    public java.lang.String test(java.lang.String str1, int age1) throws android.os.RemoteException;
}

```

### Reference&Thanks

[Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)

[Android进程间通信（IPC）机制Binder简要介绍和学习计划](https://blog.csdn.net/luoshengyang/article/details/6618363)

[Binder学习指南重要](https://weishu.me/2016/01/12/binder-index-for-newer/)

[Android AIDL 使用](https://www.cnblogs.com/tangZH/p/10775848.html)

[Android进程间通信的方式之AIDL](https://blog.csdn.net/weixin_45365889/article/details/102696348)