[Glide生命周期管理](https://www.jianshu.com/p/190285e18ae1)
[带着问题学Glide做了哪些优化?](https://juejin.cn/post/6970683481127043085)
[Glide 源码学习,了解 Glide 图片加载原理](https://www.jianshu.com/p/9d8aeaa5a329)
[Glide源码解析一，初始化](https://www.cnblogs.com/tangZH/p/12409849.html)

### 集成
##### Gradle中的依赖
```java
//implementation 'com.github.bumptech.glide:glide:4.12.0'
// 作为依赖库使用api
api 'com.github.bumptech.glide:glide:4.12.0'
annotationProcessor 'com.github.bumptech.glide:compiler:4.12.0'
```
##### 添加注解通过APT生成注解处理器文件
```java

@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}
```

#### Glide的使用
```java
Glide.with(this)
 //  .asBitmap()//只允许加载静态图片，若传入gif图会展示第一帧（要在load之前）
 //  .asGif()//指定gif格式（要在load之前）
 //  .asDrawable()//指定drawable格式（要在load之前）
     .load(imageUrl)//被加载图像的url地址
     .placeholder(R.drawable.ic_placeholder)//占位图片
     .error(R.drawable.ic_error)//错误图片
     .transition(GenericTransitionOptions.with(R.anim.zoom_in))//图片动画
     .override(800,800)//设置加载尺寸
     .skipMemoryCache(true)//禁用内存缓存功能
     .diskCacheStrategy(DiskCacheStrategy.NONE)//不缓存任何内容
  // .diskCacheStrategy(DiskCacheStrategy.DATA)//只缓存原始图片
  // .diskCacheStrategy(DiskCacheStrategy.RESOURCE)//只缓存转换后的图片
  // .diskCacheStrategy(DiskCacheStrategy.ALL)//缓存所有
  // .diskCacheStrategy(DiskCacheStrategy.AUTOMATIC)//Glide根据图片资源智能地选择使用哪一种缓存策略（默认）
     .listener(new RequestListener<Drawable>() {//监听图片加载状态
        //图片加载完成
         @Override
         public boolean onLoadFailed(@Nullable GlideException e, Object model, Target<Drawable> target, boolean isFirstResource) {
            return false;
         }
         //图片加载失败
         @Override
         public boolean onResourceReady(Drawable resource, Object model, Target<Drawable> target, DataSource dataSource, boolean isFirstResource) {
             return false;
         }
     })
    .into(imageView);//图片最终要展示的地方
```

#### Activity中调用Glide.with(this)流程
1、在Activity中调用Glide.with(this)的时候会创建一个空白的Fragment
```java
Glide.java

@NonNull
  public static RequestManager with(@NonNull FragmentActivity activity) {
    return getRetriever(activity).get(activity);
  }
  
with中调用getRetriever()方法

->
getRetriever()方法内部调用get(context)方法
->
get(context)方法内部调用getAnnotationGeneratedGlideModules（）和checkAndInitializeGlide()方法
->
getAnnotationGeneratedGlideModules()方法里面通过反射获取被@GlideModule注解生成的java文件通过类加载方式获取他的构造方法，获取实例对象
->checkAndInitializeGlide()方法内部调用->initializeGlide(context, generatedAppGlideModule);方法
->
initializeGlide()内部继续调用initializeGlide(不同参数的重载函数)方法
->
initializeGlide()调用builder.build(applicationContext);方法(重要，build（）
->

GlideBuilder.java
->
build.build()方法很重要。build方法方法获得Glide对象，
该对象只创建这么一次，Glide添加了注解，生成的java文件会解析，对比3.x解析清单文件，如果注解有了就移除
清单文件的，在build里面创建缓存策略线程池，sourceExecutor，diskCacheExecutor，animationExecutor，memorySizeCalculator)
->
->

build()方法调用了构造方法 new RequestManagerRetriever()
->
RequestManagerRetriever()构造方法实例化了RequestManagerFactory(),
->
RequestManagerFactory()中实例化了new RequestManager()
->
RequestManager()实现了LifeCyclerListener，在构造方法中LifeCycle将RequestManager对象添加监听，而LifeCycle对应为多样，比如ActivityFragmentLifecycle，ApplicationLifecycle，
->
ActivityFragmentLifecycle中添加了RequestManager监听

->
也就完成了Frament的生命周期中调用了LifeCycle的生命周期方法，而Lifecycle生命周期中调用了LifeCycleListener的生命周期方法。对应于
->
SupportRequestManagerFragment->onstart()
->
ActivityFragmentLifecycle->onstart()
->
RequestManager->onstart()

```

```java
 RequestManagerRetriever.java

@NonNull
  public RequestManager get(@NonNull FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else {
      assertNotDestroyed(activity);
      frameWaiter.registerSelf(activity);
      FragmentManager fm = activity.getSupportFragmentManager();
      return supportFragmentGet(activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }
```
2、这个Fragment为SupportRequestManagerFragment ，在RequestManagerRetriever.java中会调用它的get（）方法，get方法会调用supportFragmentGet（）或者fragmentGet()方法。
```java
 RequestManagerRetriever.java
  @NonNull
  private RequestManager supportFragmentGet(
      @NonNull Context context,
      @NonNull FragmentManager fm,
      @Nullable Fragment parentHint,
      boolean isParentVisible) {
    SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm, parentHint);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
      // TODO(b/27524013): Factor out this Glide.get() call.
      Glide glide = Glide.get(context);
      requestManager =
          factory.build(
              glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
      if (isParentVisible) {
        requestManager.onStart();
      }
      current.setRequestManager(requestManager);
    }
    return requestManager;
  }

```
3、在supportFragmentGet（）方法中会创建空白的Fragment（1，通过Tag查找，2，通过集合里面取，3，直接new SupportRequestManagerFragment（）），将创建的空白Fragment跟RequestManager绑定起来，
```java
 private static final RequestManagerFactory DEFAULT_FACTORY =
      new RequestManagerFactory() {
        @NonNull
        @Override
        public RequestManager build(
            @NonNull Glide glide,
            @NonNull Lifecycle lifecycle,
            @NonNull RequestManagerTreeNode requestManagerTreeNode,
            @NonNull Context context) {
            //1创建RequestManager的构造方法，
          return new RequestManager(glide, lifecycle, requestManagerTreeNode, context);
        }
      };

//2,RequestManager实现了LifecycleListener，被LifeCycle生命周期拥有者添加了监听，而LifeCycle里面的
addListener()方法内部调用了LifecycleListener的addListener()方法。
 RequestManager(
      Glide glide,
      Lifecycle lifecycle,
      RequestManagerTreeNode treeNode,
      RequestTracker requestTracker,
      ConnectivityMonitorFactory factory,
      Context context) {
    this.glide = glide;
    this.lifecycle = lifecycle;
    this.treeNode = treeNode;
    this.requestTracker = requestTracker;
    this.context = context;

    connectivityMonitor =
        factory.build(
            context.getApplicationContext(),
            new RequestManagerConnectivityListener(requestTracker));

    if (Util.isOnBackgroundThread()) {
      Util.postOnUiThread(addSelfToLifecycle);
    } else {
      lifecycle.addListener(this);
    }
    lifecycle.addListener(connectivityMonitor);

    defaultRequestListeners =
        new CopyOnWriteArrayList<>(glide.getGlideContext().getDefaultRequestListeners());
    setRequestOptions(glide.getGlideContext().getDefaultRequestOptions());

    glide.registerRequestManager(this);
  }

public RequestManagerRetriever(
      @Nullable RequestManagerFactory factory, GlideExperiments experiments) {
    //2
    this.factory = factory != null ? factory : DEFAULT_FACTORY;
    handler = new Handler(Looper.getMainLooper(), this /* Callback */);

    frameWaiter = buildFrameWaiter(experiments);
  }  

requestManager =
    //3
          factory.build(
              glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
   
```

4、RequestManager 实现了LifecycleListener接口，LifecycleListener接口有onstart，onstop，ondestory方法。
5、SupportRequestManagerFragment这个Fragment中持有生命周期拥有者ActivityFragmentLifeCycle，该类实现了LifeCycle接口，LifeCycle中存在onstart，onstop，ondestory方法。
6、 在Fragment生命周期发生变化时，该生命周期拥有者ActivityFragmentLifecycle 会调用它自身的onstart，onstop，ondestory等方法，ActivityFragmentLifecycle 中的onstart，onstop，ondestory方法会调用LifecycleListener的onstart，onstop，ondestory方法。
监听者LifecycleListener会调用它自身的onstart，onstop等方法，可以添加和移除实现了LifecycleListener接口的监听，而RequestManager实现了LifecycleListener，也就是Fragment生命周期变化的时候RequestManager内部实现的LifecycleListener接口的onstart，onstop，ondestory会被调用。
```java
    GlideApp.with(this).load("").into(mDataBinding.imageview);
   
	Glide.with(this).load("")
                .placeholder(R.drawable.ic_launcher_background)
                .error("")
                .into(mDataBinding.imageview);

Glide.java
  @NonNull
  public static RequestManager with(@NonNull FragmentActivity activity) {
    return getRetriever(activity).get(activity);
  }
  
RequestManagerRetriever.java
  @NonNull
  public RequestManager get(@NonNull FragmentActivity activity) {
    if (Util.isOnBackgroundThread()) {
      return get(activity.getApplicationContext());
    } else {
      assertNotDestroyed(activity);
      frameWaiter.registerSelf(activity);
      FragmentManager fm = activity.getSupportFragmentManager();
      return supportFragmentGet(activity, fm, /*parentHint=*/ null, isActivityVisible(activity));
    }
  }

....java
    /**
    *该方法获取RequestManager，里面存储了当前Glide.with(this)的fragment，
    *
    */
 @NonNull
  private RequestManager supportFragmentGet(
      @NonNull Context context,
      @NonNull FragmentManager fm,
      @Nullable Fragment parentHint,
      boolean isParentVisible) {
    //1
    SupportRequestManagerFragment current = getSupportRequestManagerFragment(fm, parentHint);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
      // TODO(b/27524013): Factor out this Glide.get() call.
      Glide glide = Glide.get(context);
//而在构建RequestManager的时候，传进去了一个current.getGlideLifecycle()
      requestManager =
          factory.build(
              glide, current.getGlideLifecycle(), current.getRequestManagerTreeNode(), context);
    
      if (isParentVisible) {
        requestManager.onStart();
      }
      current.setRequestManager(requestManager);
    }
    return requestManager;
  }
/**
*先通过tag取，取不到的情况,下在集合里面取，实在取不到就new SupportRequestManagerFragment,返回到1处
*/
@NonNull
  private SupportRequestManagerFragment getSupportRequestManagerFragment(
      @NonNull final FragmentManager fm, @Nullable Fragment parentHint) {
    SupportRequestManagerFragment current =
        (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
      current = pendingSupportRequestManagerFragments.get(fm);
      if (current == null) {
        current = new SupportRequestManagerFragment();
        current.setParentFragmentHint(parentHint);
        pendingSupportRequestManagerFragments.put(fm, current);
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
        handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }


```

Glide.with(this)绑定了Activity的生命周期。在获取Glide对象时，通过GlideBuilder.build（）方法初始化缓存线程池。在Activity内新建了一个无UI的Fragment，这个Fragment持有一个Lifecycle，通过Lifecycle在Fragment关键生命周期通知RequestManager进行相关从操作。在生命周期onStart时继续加载，onStop时暂停加载，onDestory时停止加载任务和清除操作。
### build方法里面是对glide的各种配置，比如上面的：
 
> 1、加载源数据的线程池
2、加载磁盘缓存中数据的线程池（不能用来加载url网络数据）
3、动画加载请求器
4、内存计算器
5、网络状态检测器
6、bitmap缓存池（避免不断不断创建与回收bitmap导致内存抖动）
7、数组资源缓存池
8、内存缓存，缓存完成加载和显示的图片数据资源
9、本地磁盘缓存，默认存储在app内部私密目录 

### 然后创建图片加载引擎。new Engine();最后再将上面的缓存池，引擎等作为参数，构建一个Glide()
 这个Glide中的构造函数非常长，我们只挑一些重点的来讲，其它的可以类推：

##### 1、创建各种decoder，也就是解码器，比如:
##### 将ByteBuffer解码为GifDrawable。
```java
ByteBufferGifDecoder byteBufferGifDecoder =
        new ByteBufferGifDecoder(context, imageHeaderParsers, bitmapPool, arrayPool);

```
##### 将资源uri解码为Drawable：
```java
ResourceDrawableDecoder resourceDrawableDecoder = new ResourceDrawableDecoder(context);
```
##### 2、创建各种factory，即模型装换器，比如：
```java
ResourceLoader.StreamFactory resourceLoaderStreamFactory =
        new ResourceLoader.StreamFactory(resources);
```
##### 将Android资源ID转换为Uri，在加载成为InputStream
```java
 ResourceLoader.UriFactory resourceLoaderUriFactory = new ResourceLoader.UriFactory(resources);
  
```
将资源ID转换为Uri

#### 3、创建各种Transcoder，即转码器，比如
```
 BitmapBytesTranscoder bitmapBytesTranscoder = new BitmapBytesTranscoder();
```
##### 将Bitmap转码为Byte arrays
```java
GifDrawableBytesTranscoder gifDrawableBytesTranscoder = new GifDrawableBytesTranscoder();

```
将将GifDrawable转码为Byte arrays

#### 4、创建各种encoder，即编码器，比如：
```java
BitmapEncoder bitmapEncoder = new BitmapEncoder(arrayPool);
```
将Bitmap数据缓存为File

#### 5、注册：注册将Byte数据缓存为File的编码器，注册将InputStream缓存为File的编码器。将ByteBuffer解码为Bitmap的解码器，将inputStreams解码为Bitmap的编码器。

```java
 registry
        .append(ByteBuffer.class, new ByteBufferEncoder())
        .append(InputStream.class, new StreamEncoder(arrayPool))
        /* Bitmaps */
        .append(Registry.BUCKET_BITMAP, ByteBuffer.class, Bitmap.class, byteBufferBitmapDecoder)
        .append(Registry.BUCKET_BITMAP, InputStream.class, Bitmap.class, streamBitmapDecoder);

```
## load()

## inito()
##### RequestBuilder.java

1、从into()->into()->buildRequest()获取一个Request。
2、在buildRequest中调用buildRequestRecursive（）方法，这个方法里面有主要的请求和错误的请求，我们看主要的mainRequest的实例化,会调用到buildThumbnailRequestRecursive（）这个方法拿到Request。
3、在上面方法中会得到fullRequest和thumbRequest，分别通过obtainRequest（）和buildRequestRecursive（）方法。
4、先看obtainRequest（）方法，可以看到会调用到SingleRequest的obtain（）方法。得到SingleRequest的对象。
接下来就是SingleRequest.java里面的内容
5、SingleRequest中的obtain（）会调用到SingleRequest（）的构造方法，实例化SingleRequest对象。
6、在步骤1中第二个into的时候，获取了request对象，into里面会调用previous.begin();或者requestManager.track(target, request);方法。
7、begin（）方法直接请求网络
8、requestManager.track（）方法会执行RequestTracker的runRequest()方法
接下来就是在RequestTracker.java中
9、接下来也是执行Request中的begin（）方法。
10、在begin()方法中回调用onSizeReady（），而onSizeReady回调用Engin的load()方法。 loadStatus =engine.load(）
接下来就是在Engin.java类中
11、Engin.load()在load方法中会调用，waitForExistingOrStartNewJob（）方法，在这个方法里面有两个关键类，EnginJob（内部类EngineJobFactory）和DecodeJob(内部类DecodeJobFactory)。
接下来看EnginJob。java
12、EngineJob<R> engineJob =engineJobFactory.build(），接着在build()中调用init()
接下来看DecodeJob。java
13、DecodeJob>builder().init()->DecodeHelper.init()
14、接着回到11步骤的waitForExistingOrStartNewJob（）方法，在获取了EngineJob和DecodeJob后会启动以下关键代码。EngineJob会给到DecodeJob的build中作为参数
```java

   jobs.put(key, engineJob);
  
    engineJob.addCallback(cb, callbackExecutor);
    engineJob.start(decodeJob);
```
15、engineJob的addCallBack方法会把线程池和cd传递过去执行了CallResourceReady这个实现了Runnable接口的线程，不知道执行了一大堆什么乱七八糟的
16、接下来看engineJob的start（decodeJob）方法，该方法会有一个GlideExecutor线程池，而decodeJob也实现了Runnable接口。 GlideExecutor.execute(decodeJob);就会执行到decodeJob的run（）方法。
继续再DecodeJob中来看
17、DeccorderJob中的run()方法中回调用runWrapped（）方法，该方法会调用到runGenerators（）方法。
18、在runGenerators中会有个while循环，会调用currentGenerator.startNext()，而currentGenerator是DataFetcherGenerator.java的对象，DataFetcherGenerator接口有三个实现类，我们选取SourceGenerator来看下，反正也不知道是什么一坨。。。
![image.png](https://upload-images.jianshu.io/upload_images/2981395-35a29d42aa6904eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来看SourceGenerator。java
19、SourceGenerator中的startNext（）中回调用startNextLoad（）方法，该方法中会调用 loadData.fetcher.loadData(x，new DataCallback()），其中loadData为ModelLoader的内部类LoadData，fetcher为DataFetcher。而DataFetcher的实现类挺多，可以随意找一个，比如现在要看into的网络请求HttpUrlFetcher.java类。。。。emmm......
![image.png](https://upload-images.jianshu.io/upload_images/2981395-1630f34a94eab2ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来就看HttpUrlFetcher。java
20、HttpUrlFetcher中的loadData（）方法，内部会调用loadDataWithRedirects（）会返回一个InputStream，猜测就是网络请求拿到的图片结果。
21、loadDataWithRedirects（）方法中，会有一个HttpUrlConnection用于网络请求，通过buildAndConfigureConnection（）得到该HttpUrlConnection的对象，·
```java
private HttpURLConnection buildAndConfigureConnection(URL url, Map<String, String> headers)
      throws HttpException {
    HttpURLConnection urlConnection;
    try {
      urlConnection = connectionFactory.build(url);
    } catch (IOException e) {
      throw new HttpException("URL.openConnection threw", /*statusCode=*/ 0, e);
    }
    for (Map.Entry<String, String> headerEntry : headers.entrySet()) {
      urlConnection.addRequestProperty(headerEntry.getKey(), headerEntry.getValue());
    }
    urlConnection.setConnectTimeout(timeout);
    urlConnection.setReadTimeout(timeout);
    urlConnection.setUseCaches(false);
    urlConnection.setDoInput(true);
    // Stop the urlConnection instance of HttpUrlConnection from following redirects so that
    // redirects will be handled by recursive calls to this method, loadDataWithRedirects.
    urlConnection.setInstanceFollowRedirects(false);
    return urlConnection;
  }
```java
好了，综上假装上述主线流程没有问题，那么就拿到了图片流。。。

### ImageView设置图片的主线流程
1、RequestBuilder中的into（）方法，内部调用了 requestManager.track(target, request);，接下来看RequestManager中的track方法。
2、RequestManager的track（）会调用RequestTracker的runRequest（）方法，接着会调用  request.begin();而SingleRequest实现了Request的begin（）方法。
3、接下来看SingleRequest的begin（），可以看到会调用onResourceReady（）方法
4、onResourceReady中会继续调用onResourceReady（）第二个 onResourceReady回调用target.onResourceReady(result, animation);方法，而ImageVIewTarget实现了接口Target的onResourceReady方法，在该ImageViewTarget中可以看到setResourceInternal（）方法。
```java
if (transition == null || !transition.transition(resource, this)) {
      setResourceInternal(resource);
    } else {
      maybeUpdateAnimatable(resource);
    }
```
5、setResourceInternal（）调用了 setResource(resource)抽象方法，实现了该ImageVIewTarget的setResource抽象方法均为设置ImageView的ImageDrawable属性。
```java
  @Override
  protected void setResource(@Nullable Drawable resource) {
    view.setImageDrawable(resource);
  }
```
以上粗略认为是into设置的图片主线。

        
          





















#### Glide为什么对Fragment做缓存？

```java
 private SupportRequestManagerFragment getSupportRequestManagerFragment(
      @NonNull final FragmentManager fm, @Nullable Fragment parentHint) {
    SupportRequestManagerFragment current =
        (SupportRequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
      current = pendingSupportRequestManagerFragments.get(fm);
      if (current == null) {
        current = new SupportRequestManagerFragment();
        current.setParentFragmentHint(parentHint);
//添加到Map缓存中（防止fragment重复创建）
        pendingSupportRequestManagerFragments.put(fm, current);
///将fragment绑定到activity
        fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
////添加后发送清理缓存的消息
        handler.obtainMessage(ID_REMOVE_SUPPORT_FRAGMENT_MANAGER, fm).sendToTarget();
      }
    }
    return current;
  }
```
一个场景：通过Glide加载两张图片并设置到两个ImageView上。
```java
Glide.with(this).load(imageUrl1).into(imageView1);//msg1
Glide.with(this).load(imageUrl2).into(imageView2);//msg2
```
 执行到getRequestManagerFragment这个方法时，会通过开启事务的方式绑定fragment到activity，绑定操作最终是通过主线程的handler发送消息处理的，Handler中的消息时按照顺序执行的。如果不把msg1构建的RequestManagerFragment放到pendingRequestManagerFragments中，那么在执行msg2的时候会重新创建一个重复的fragment并add。最后发消息清理缓存（避免内存泄漏和减少内存压力），因为消息是按照顺序执行的，执行清理缓存的时候msg1和msg2已经执行完毕。
####Glide如何监听网络变化？
在构建RequestManager的时候通过lifecycle.addListener(connectivityMonitor);添加网络变化的监听 ，Fragment生命周期的变化会通知到默认实现类DefaultConnectivityMonitor中对应的方法。在onStart中registerReceiver（注册监听手机网络变化的广播）， 在onStop中unregisterReceiver。有网络重连后重启请求。
```java
/** Uses {@link android.net.ConnectivityManager} to identify connectivity changes. */
final class DefaultConnectivityMonitor implements ConnectivityMonitor {
  private static final String TAG = "ConnectivityMonitor";
  private final Context context;

  @SuppressWarnings("WeakerAccess")
  @Synthetic
  final ConnectivityListener listener;

  @SuppressWarnings("WeakerAccess")
  @Synthetic
  boolean isConnected;

  private boolean isRegistered;

  private final BroadcastReceiver connectivityReceiver =
      new BroadcastReceiver() {
        @Override
        public void onReceive(@NonNull Context context, Intent intent) {
          boolean wasConnected = isConnected;
          isConnected = isConnected(context);
          if (wasConnected != isConnected) {
            if (Log.isLoggable(TAG, Log.DEBUG)) {
              Log.d(TAG, "connectivity changed, isConnected: " + isConnected);
            }

            listener.onConnectivityChanged(isConnected);
          }
        }
      };

  DefaultConnectivityMonitor(@NonNull Context context, @NonNull ConnectivityListener listener) {
    this.context = context.getApplicationContext();
    this.listener = listener;
  }

  private void register() {
    if (isRegistered) {
      return;
    }

    // Initialize isConnected.
    isConnected = isConnected(context);
    try {
      // See #1405
      context.registerReceiver(
          connectivityReceiver, new IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION));
      isRegistered = true;
    } catch (SecurityException e) {
      // See #1417, registering the receiver can throw SecurityException.
      if (Log.isLoggable(TAG, Log.WARN)) {
        Log.w(TAG, "Failed to register", e);
      }
    }
  }

  private void unregister() {
    if (!isRegistered) {
      return;
    }

    context.unregisterReceiver(connectivityReceiver);
    isRegistered = false;
  }

  @SuppressWarnings("WeakerAccess")
  @Synthetic
  // Permissions are checked in the factory instead.
  @SuppressLint("MissingPermission")
  boolean isConnected(@NonNull Context context) {
    ConnectivityManager connectivityManager =
        Preconditions.checkNotNull(
            (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE));
    NetworkInfo networkInfo;
    try {
      networkInfo = connectivityManager.getActiveNetworkInfo();
    } catch (RuntimeException e) {
      // #1405 shows that this throws a SecurityException.
      // b/70869360 shows that this throws NullPointerException on APIs 22, 23, and 24.
      // b/70869360 also shows that this throws RuntimeException on API 24 and 25.
      if (Log.isLoggable(TAG, Log.WARN)) {
        Log.w(TAG, "Failed to determine connectivity status when connectivity changed", e);
      }
      // Default to true;
      return true;
    }
    return networkInfo != null && networkInfo.isConnected();
  }

  @Override
  public void onStart() {
    register();
  }

  @Override
  public void onStop() {
    unregister();
  }

  @Override
  public void onDestroy() {
    // Do nothing.
  }
}
```

回调ConnectivityListener的onConnectivityChanged来处理请求
```java
 private class RequestManagerConnectivityListener
      implements ConnectivityMonitor.ConnectivityListener {
    @GuardedBy("RequestManager.this")
    private final RequestTracker requestTracker;

    RequestManagerConnectivityListener(@NonNull RequestTracker requestTracker) {
      this.requestTracker = requestTracker;
    }

    @Override
    public void onConnectivityChanged(boolean isConnected) {
      if (isConnected) {
        synchronized (RequestManager.this) {
          //网络重连后重启请求
          requestTracker.restartRequests();
        }
      }
    }
  }
```