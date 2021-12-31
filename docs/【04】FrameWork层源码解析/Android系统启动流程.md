# [原文：Android系统启动流程](https://www.jianshu.com/p/2c1318b0f527){docsify-ignore}
Init->init.rc->init.cpp#main()->app_process.cpp#main()->AndroidRuntime.cpp#start()->ZygoteInit#main()->ZygoteInit#startSystemServer()->Zygote#forkSystemServer()->ZygoteInit#handleSystemServerProcess(启动systemservice)->ZygoteInit#zygoteInit()->RuntimeInit#applicationInit()->RuntimeInit#findStaticMain()->反射加载SystemService#main()->SystemService#run()->startBootstrapServices/startCoreServices/startOtherServices->AMS,ATMS,WMS
### 一：Init
当用户按下开机键的时候，引导芯片加载BootLoader到内存中，开始拉起Linux OS，一旦Linux内核启动完毕后，它就会在系统文件中寻找 init.rc 文件，并启动init 进程。
什么是 init.rc 文件？
init.rc文件是一个非常重要的配置文件，它由AIL语言（Android Init Language）编写。
（init.rc的一个小细节：Android8.0后对init.rc文件进行了拆分，比如64位的Zygote的启动脚本在init.zygote64.rc文件中）
在启动的init进程中，会进入system/core/init/init.cpp文件的main方法中，关键代码如下：
```java
int main(int argc,char ** argv){
    ...
    if(is_first_stage){
        //创建和挂在启动所需要的文件目录
        mount("tmpfs","/dev","tmpfs",MS_NOSUID,"mode=0755");
        mkdir("/dev/pts",0755);
        //创建和挂在很多...
        ...
    }
    ...
    //对属性服务进行初始化
    property_init();
    ...
    //用于设置子进程信号处理函数（如Zygote），如果子进程异常退出，init进程会调用该函数中设定的信号处理函数来处理
    signal_handler_init();
    ...
    //启动属性服务
    start_property_service();
    ...
    //解析init.rc配置文件
    parser.ParseConfig("/init.rc");
}
```
上述代码中，有一个概念是 属性服务,经常玩Windows 的同学应该知道WIndows注册表，注册表采用键值对的形式来记录用户、软件的一些使用信息，这样即使系统或者软件重启也能够根据之前注册表中的记录来进行相应的初始化。init进程启动的属性服务，就是用来储存这些属性的。
属性服务的查询和修改都是通过Socket进行的，系统文件内定义对多为8个用户提供服务。并且属性服务中的系统属性分为两种：普通属性和控制属性。控制属性用来执行一些命令，比如开机的动画就使用了这种属性。
在处理了繁多的任务后，init进程会进行最关键的一部操作: 启动Zygote
代码在 frameworks/base/cmds/app_process/app_main.cpp中：
```java
int main(int argc ,char* const argv[]){
    ...
    if(zygote){
        //启动Zygote进程
        runtime.start("com.android.internal.os.ZygoteInit",args,zygote);
    }
}
```
#### 总结一下，init进程启动后做了哪几件事：
(1)创建和挂载启动所需要的文件和目录
(2)初始化和启动属性服务。
(3)解析init.rc配置文件，并且启动了Zygote进程，入口ZygoteInit.java
### 二：Zygote进程ZygoteInit.java
（1）Zygote概述:

在Android系统中，DVM(Dalvik虚拟机)和ART、应用程序进程以及运行系统关键服务的SystemServer进程都是由Zygote进程创建的，我们也将它称为孵化器。它通过fork复制进程的形势来创建应用进程和SystemServer进程，由于Zygote进程在启动时会创建DVM或者ART，因此通过fork而创建的应用程序进程和SystemServer进程可以在内部获取一个DVM或者ART的实例副本。
init.rc文件引入Zygote的启动脚本，这些脚本都是由AIL编写的。
由于Android 5.0后,Android开始支持64位程序，Zygote也就有了32位与64位之分。一共有四种Zygote启动脚本：
init.zygote32.rc
init.zygote32_64.rc
init.zygote64.rc
init.zygote64_32.rc
####（2）Zygote启动流程

Zygote启动时主要调用app_main.cpp的main()中的AppRuntime的start方法来启动Zygote进程。我们先展示app_main.cpp中的main函数。
```java
int main(int argc,char* const argv[]){
    ...
    
    while( i < argc ){
        const char* arg=argv[i++];
        if(strcmp(arg,"--zygote")==0){
            //如果当前进程在Zygote中，则设置zygote=true
            zygote=true;
            niceName=ZYGOTE_NICE_NAME;
        }else if(strcmp(arg,"--start-system-server")==0){
            //如果当前进程在SystemServer中，将startSystemServer=true
            startSystemServer=true;
        }
        ...
    }
    ...
    //承接上面Init进程中的代码
    if(zygote){
        //启动Zygote进程
        runtime.start("com.android.internal.os.ZygoteInit",args,zygote);
    }
}
```
由于Zygote进程都是通过fock自身来创建子进程，这样Zygote进程和他的子进程（比如SystemServer）都可以进入app_main.cpp的main函数中，因此需要区分一下当前进程运行在哪个进程里。
接下来我们进入 runtime.start中看看：
frameworks/base/core/jni/AndroidRuntime.cpp：
```java
void AndroidRuntime::start(const char* className,const Vector<String8>& options,bool zygote){
    ...
    //startVm函数用于启动Java虚拟机
    if(startVm(&mJavaVM,&env,zygote)!=0){
        return;
    }
    onVmCreated(env);
    //为Java虚拟机注册JNI方法
    if(startReg(env)<0){
        return;
    }
    ...
    //从app_main的main方法中得知，className是com.android.internal.os.ZygoteInit
    classNameStr=env->NewStringUTF(className);
    ...
    //找到ZygoteInit类
    jclass startClass=env->FindClass(slashClassName);
    ...
    //找到ZygoteInit的main方法
    jmethodID startMeth=env->GetStaticMethodID(startClass,"main","([Ljava/lang/String;)V");
    ...
    //通过JNI调用ZygoteInit的main方法
    env->CallStaticVoidMethod(startClass,startMeth,strArray);
}
```
以上代码就厉害了，它从Init进程中的AndroidRuntime的main函数，启动了Java虚拟机，并且通过JNI启动了Zygote，一波操作之后，Zygote顺利从Native层进入了Java层。
随后我们进入ZygoteInit中，看看它的main方法做了什么。
frameworks/base/core/java/com/android/internal/os/ZygoteInit.java:
```java
public static void main(String argv[]){
    ...
    //创建一个Server端的Socket,socketName值为：zygote
    zygoteServer.registerServerSocket(socketName)；
    if(!enableLazyPreload){
        ...
        //预加载类和资源
        preload(bootTimingsTraceLog);
    }else{
        ...
    }
    if(startSystemServer){
        //启动SystemServer进程
        startSystemServer(abiList,socketName,zygoteSerer);
    }
    //等待AMS的请求
    zygoteServer.runSelectLoop(abiList);
    zygoteServer.closeServerSocket();
}
```
#### 总结一下ZygoteInit的main方法都做了哪些事情：
1.创建了一个Server端的Socket
2.预加载类和资源
3.启动了SystemServer进程
4.等待AMS请求创建新的应用程序进程
最后再总结一下Zygote进程启动公做了几件事：
1.创建AndroidRuntime并调用其start方法，启动Zygote进程。
2.创建Java虚拟机并为Java虚拟机注册JNI方法。
3.通过JNI调用ZygoteInit的main函数进入Zygote的java框架层。
4.通过registerZygoteSocket方法创建服务端Socket，并通过runSelectLoop方法等待AMS的请求来创建新的应用程序进程。
5.启动SystemServer。
### 三：SystemServer
SystemServer进程主要用于创建系统服务，我们熟知的AMS、WMS、PMS都是由它来创建的。
在前面讲到过，Zygote进程启动了SystemServer进程，我们看一下启动部分的代码。
frameworks/base/core/java/com/android/internal/os/ZygoteInit.java:
```java
private static boolean startSystemServer(String abiList,String socketName){
    ...
    //判断进程是否是SystemServer
    if(pid == 0){
        ...
        //关闭Zygote的Socket
        zygoteServer.closeServerSocket();
        //启动SystemServer进程
        handleSystemServerProcess(parsedArgs);
    }
}
```
由于SystemServer是Zygote进程fork出来的，所以该进程也拥有一个ZygoteServer所开启等待AMS连接的Socket实例副本。在这里并不需要这个Socket，所以关闭。
接下来看看handleSystemServerProcess()方法。
frameworks/base/core/java/com/android/internal/os/ZygoteInit.java:
```java
private static void handleSystemServerProcess(ZygoteConnection.Arguments parsedArgs){
    ...
    ClassLoader cl=null;
    if(systemServerClasspath!=null){
        //在这里创建了PathClassLoader
        cl = createPathClassLoader(systemServerClasspath,parsedArgs.targetSdkVersion);
        Thread.currentThread().setContextClassLoader(cl);
    }
    //zygoteInit方法
    ZygoteInit.zygoteInit(parsedArgs.targetSdkVersion,parsedArgs.remainingArgs,cl);
}
```
就是这里，创建了大名鼎鼎的 PathClassLoader。
接下来看看zygoteInit方法。
frameworks/base/core/java/com/android/internal/os/ZygoteInit.java:
```java
public static final void zygoteInit(int targetSdkVersion,String[] argv,ClassLoader classLoader){
    ...
    //就在该方法中，启动了Binder线程池
    ZygoteInit.nativeZygoteInit();
    //进入SystemServer的main方法
    RuntimeInit.applicationInit(targetSdkVersion,argv,classLoader);
}
```
nativeZygoteInit方法一看名称，就知道是在Native层的代码。用来启动Binder线程池，这样SystemServer进程就可以使用Binder与其他进程进行通信了。
再讲一下RuntimeInit的applicationInit方法，该方法用于启动SystemServer（以及后来我们的应用程序进程的启动）。
frameworks/base/core/java/com/android/internal/os/RuntimeInit.java:
```java
protected static void applicationINit(int targetSdkVersion,String[] argv,ClassLoader classLoader){
    ...
    invokeStaticMain(args.startClass,args.startArgs,classLoader);
}

...

private static void invokeStaticMain(String className,String[] argv,ClassLoader classLoader){
    Class<?> cl;
    ...
    //className是com.android.server.SystemServer
    cl=Class.forName(className,true,classLoader);
    ...
    //找到SystemServer的main方法
    m=cl.getMethod("main",new Class[]{String[].class});
    ...
    //抛出异常，这里抛出异常中调用了SystemServer的main方法
    throw new Zygote.MethodAndArgsCaller(m,argv);
}
```
在invokeStaticMain方法最后，以抛出异常的方式调用了SystemServer的main方法（之后在启动其他应用进程的时候，也是这样调用ActivityThread的main方法的）。这种处理会清除所有设置过程需要的堆栈帧。
接下来我们解析一下SystemServer进程。
frameworks/base/services/java/com/android/server/SystemServer.java:
```java
public static void main(String[] args){
    //就一行代码
    new SystemServer().run();
}

private void run(){
    ...
    //创建消息Looper
    Looper.prepareMainLooper();
    //加载动态库
    System.loadLibrary("android_servers");
    //创建SystemServiceManager
    mSystemServiceManager=new SystemServiceManager(mSystemContext);
    ...
    //启动引导服务
    startBootstrapServices();
    //启动核心服务
    startCoreServices();
    //启动其他服务
    startOtherServices();
    ...
}
```
我们可以看到SystemServer在启动后，陆续启动了各项服务，包括ActivityManagerService，PowerManagerService，PackageManagerService等等，而这些服务的父类都是SystemService。
最后总结一下SystemServer进程：
1.启动Binder线程池
2.创建了SystemServiceManager（用于对系统服务进行创建、启动和生命周期管理）
3.启动了各种服务