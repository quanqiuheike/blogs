### 本文主要讲Activity，Window，PhoneWindow，DecorView，ViewRootImpl，WindowManager，View绘制requestLayout等{docsify-ignore}

### [原文版权Android WMS流程](https://blog.csdn.net/qq_14876133/article/details/100103907){docsify-ignore}
### [Handler消息Message屏障消息](https://blog.csdn.net/mirkowug/article/details/115091337){docsify-ignore}
## WMS流程
![image.png](https://upload-images.jianshu.io/upload_images/2981395-f7a58d93a63d10b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```java
1、System.core.init.cpp文件的main方法

2、main方法会有LoadBootScripts()加载init.rc文件

3、 里面会有trigge -zygote-start方法启动init.zygote64.rc   sdk我查看是ZygoteInit.java中的main方法启动，里面会调用ZygoteService.java、ZygoteService#forkSystemService->Zygote.forkSystemService等。入口ZygoteInit#main

4、再加载serve zygote System.core.app_process64.rc命令启动app_process.rc中的音频，媒体，网络一些基本任务，从zygote中启动。

5、zygote fork出SystemService，开启binder线程池，开启SystemServiceManager，再就是AMS,PMS，WMS八十多个服务，startCoreService,startOtherService。

6、package/apps/定制应用

7、Launcher.java中的main方法，再执行onClick方法，启动startActivitySafty再到startActivity。
8、启动分为热启动和冷启动，不管哪种，都要到达startActivity，通过Instrumentation.execStartActivity执行。调用AMS一系列流程。冷启动是socket通信fork出新的进程。
9、调用ATMS的远程代理对象Binder调用它的startActivity方法，最后通知Zygote通过socket通信fork新的进程。
10、zygote 创建了新进程会启动ActivityThread.main()方法，ActivityThread通过attach通知AMS绑定Application，AMS接收到消息后初始化一系列工作通知ApplicationThread创建Application和回到里面的onCreate()方法。再走一系列很多省略，，，，在是handleLauncherActivity。
```

## Window & Activity & DecorView & ViewRoot关系
* Activity：
  * 处理器，控制生命周期和处理事件
  * 1个Activity包含1个Window
  * 通过回调与Window、View交互
* Window：
  * 实现类是PhoneWindow
  * 视图的承载器，用于管理View的显示
  * PhoneWindow内部包含一个DecorView
  * 内部通过WindowManager管理UI，将DecorView交给ViewRoot
* DecorView：
  * 顶层View，本质是FrameLayout
  * 显示和加载布局
  * View的事件最先经过DecorView，再分发到子View
* ViewRoot：
  * ViewRootImpl是其实现类
  * 用于连接WindowManager和DecorView
  * 完成View的三大流程：测量、布局、绘制
![image.png](https://upload-images.jianshu.io/upload_images/2981395-9c7c91a882cb9e36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/2981395-33de6fe5aafeddaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# ActivityThread

# 源码分析：
### ActivityThread#handleLaunchActivity()
1、初始化WMS服务
2、调用performLaunchActivity()
```java
 public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {
1、初始化WMS服务，initialize()内部会调用getWindowManagerService()拿到wms的远程代理Binder。IWindowManagerImpl extends IWindowManager.Default 
 WindowManagerGlobal.initialize();  

2、调用performLaunchActivity()初始化Activity
 final Activity a = performLaunchActivity(r, customIntent);

}

```
#@ ActivityThread#performLaunchActivity()
1、创建Activity
2、调用Activity#attach()
3、回调Activity#onCreate()

细化：1、创建Activity的context上下文对象；2、通过反射拿到Instrumentation的Activity；3、创建Application的上下文context；4、实例化window，他的唯一实现类PhoneWindow。5、拿到Activity的主题样式并设置。6、activity绑定appContextActivity和Activty的各个属性。7、通过Instrumentation调动callActivityOnCreate（）方法，内部走的activity的
### Activity 的 Window 创建过程

在 Activity 的创建过程中，最终会由 ActivityThread 的 performLaunchActivity() 来完成整个启动过程，该方法内部会通过类加载器创建 Activity 的实例对象，并调用 attach 方法关联一系列上下文环境变量。在 Activity 的 attach 方法里，系统会创建所属的 Window 对象并设置回调接口，然后在 Activity 的 setContentView 方法中将视图附属在 Window 上
### performLaunchActivity



创建ContextImpl ，给到Activity进行attach，context的实现类ContextWrap，实际上内部是用ContextImpl 调用。
```java
1、创建Activity的context上下文对象
ContextImpl appContext = createBaseContextForActivity(r);

2、通过反射拿到Instrumentation的Activity
  java.lang.ClassLoader cl = appContext.getClassLoader();
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);


3、创建Application的上下文context
Application app = r.packageInfo.makeApplication(false, mInstrumentation);

4、实例化window，他的唯一实现类PhoneWindow。
    window = r.mPendingRemoveWindow;

5、activity绑定appContextActivity和Activty的各个属性
activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);

6、拿到Activity的主题样式并设置
 int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }

7、通过Instrumentation调动callActivityOnCreate（）方法，内部走的activity的performCreate（），方法最终调用Activity#onCreate()
Activity.performCreate（）内部走的Activity.onCreate()
if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }

```

#### 接着看Activity#.attach()方法
```java
final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
1、attachBaseContext最终给到contextWrap.java，传递过去的是ContextImpl
 attachBaseContext(context);
 
2、实例化创建PhoneWindow，实现类PhoneWindow
 mWindow = new PhoneWindow(this, window, activityConfigCallback);


3、window绑定WindowManager服务，setWindowManager()会创建一个WindowManager
 mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
}

4、设置软键盘
   if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }

4、在Window#setWindowManager()中创建WindowManager
 mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);

 mWindowManager = mWindow.getWindowManager();
5、不知道是不是这调用的setContentView()，如果不是那就直接看performLauncherActivity中Instrumentation#callActivityOnCreate方法即可，里面执行的oncreate会有setContentView()
setContentCaptureOptions(application.getContentCaptureOptions());

```
## Activity#setContentView()
* 最终调用PhoneWindow#setContentView()
```java
@Override
public void onCreate(Bundle savedInstanceState) {
1、super.onCreate-》setContentView开始调用
    super.onCreate(savedInstanceState);

    setContentView(R.layout.activity_main);
}
点击super.oncreate往上查找，MainActivity-》FragmentActivity-》ComponentActivity#setContentView-》Activity的setContentView
 public void setContentView(@LayoutRes int layoutResID) {
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
    }
```
## PhoneWindow#setContentView()
* 创建DecorView
* 将Activity的XML布局加载到DecorView中
```java
 @Override
    public void setContentView(int layoutResID) {
        // Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
        // decor, when theme attributes and the like are crystalized（透明的）. Do not check the feature
        // before this happens.
   /*
    mContentParent为DecorView里的内容栏，DecorView相当于ViewGroup，
    当mContentParent==null时，会调用installDecor()创建一个DecorView;
    当mContentParent!=null时，会删除所有子View
    */
1、有可能被设置是否存在透明属性，存在就不用检查特性？上面的英文注释是否存在decorView?
        if (mContentParent == null) {
            installDecor();
        } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            mContentParent.removeAllViews();
        }

        if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
            final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
                    getContext());
            transitionTo(newScene);
        } else {
2、加载解析XML布局，并添加到mContentParent内容栏里
            mLayoutInflater.inflate(layoutResID, mContentParent);
        }
        mContentParent.requestApplyInsets();
3、内容发生变化通知回调
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();
        }
        mContentParentExplicitlySet = true;
    }

```
## PhoneWindow#installDecor()

```java
private void installDecor() {
        mForceDecorInstall = false;
1、 创建DecorView
        if (mDecor == null) {
            mDecor = generateDecor(-1);
            }
2、为DecorView设置布局格式
     if (mContentParent == null) {
返回一个ViewGroup,内部是DecorView添加进去的布局
              mContentParent = generateLayout(mDecor);
      }
}

3、创建DecorView
protected DecorView generateDecor(int featureId) {
    return new DecorView(context, featureId, this, getAttributes());
}

4、为DecorView设置布局格式，得点看源码看，里面很多主题设置，根据设置的判断来分析
DecorView本质是一个FrameLayout里面包含：TitleView和ContentView
protected ViewGroup generateLayout(DecorView decor) {
    //从主题文件中获取样式信息
    TypedArray a = getWindowStyle();

 mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
        int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
                & (~getForcedWindowFlags());
        if (mIsFloating) {
            setLayout(WRAP_CONTENT, WRAP_CONTENT);
            setFlags(0, flagsToUpdate);
        } else {
            setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
            getAttributes().setFitInsetsSides(0);
            getAttributes().setFitInsetsTypes(0);
        }

        if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {
            requestFeature(FEATURE_NO_TITLE);
        } else if (a.getBoolean(R.styleable.Window_windowActionBar, false)) {
            // Don't allow an action bar if there is no title.
            requestFeature(FEATURE_ACTION_BAR);
        }

        if (a.getBoolean(R.styleable.Window_windowActionBarOverlay, false)) {
            requestFeature(FEATURE_ACTION_BAR_OVERLAY);
        }
。。。。
 if (a.getBoolean(R.styleable.Window_windowShowWallpaper, false)) {
            setFlags(FLAG_SHOW_WALLPAPER, FLAG_SHOW_WALLPAPER&(~getForcedWindowFlags()));
        }

        if (a.getBoolean(R.styleable.Window_windowEnableSplitTouch,
                getContext().getApplicationInfo().targetSdkVersion
                        >= android.os.Build.VERSION_CODES.HONEYCOMB)) {
            setFlags(FLAG_SPLIT_TOUCH, FLAG_SPLIT_TOUCH&(~getForcedWindowFlags()));
        }
。。。
 if (a.getBoolean(R.styleable.Window_windowLightStatusBar, false)) {
            decor.setSystemUiVisibility(
                    decor.getSystemUiVisibility() | View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
        }
        if (a.getBoolean(R.styleable.Window_windowLightNavigationBar, false)) {
            decor.setSystemUiVisibility(
                    decor.getSystemUiVisibility() | View.SYSTEM_UI_FLAG_LIGHT_NAVIGATION_BAR);
        }

。。。


  
    //根据主题样式，加载窗口布局
    int layoutResource;
    int features = getLocalFeatures();
    //加载layoutResource
    View in = mLayoutInflater.inflate(layoutResource, null);

    //往DecorView中添加子View
    decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT)); 
    mContentRoot = (ViewGroup) in;
mDecor.startChanging();
        mDecor.onResourcesLoaded(mLayoutInflater, layoutResource);
    //这里获取的是mContentParent = 即为内容栏（content）对应的DecorView = FrameLayout子类
    ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT); 
    return contentParent;
}


```
## 回到调上面的ActivityThread#handleResumeActivity()
* 回调Activity#onResume()
* 获取PhoneWindow和DecorView，将Activity与DecorView绑定
* 调用WindowManageImpl#addView()将DecorView传入
```java
public void handleResumeActivity(IBinder token, boolean finalStateRequest, 
                                 boolean isForward, String reason) {

    //最终会调用Activity#onResume()
    final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);

    final Activity a = r.activity;

    if (r.window == null && !a.mFinished && willBeVisible) {
        //获取PhoneWindow对象
        r.window = r.activity.getWindow();
        //获取DecorView对象
        View decor = r.window.getDecorView();
        
        //DecorView不可见
        decor.setVisibility(View.INVISIBLE);
        ViewManager wm = a.getWindowManager();
        WindowManager.LayoutParams l = r.window.getAttributes();
        
        //DecorView与Activity建立关联
        a.mDecor = decor;

        if (a.mVisibleFromClient) {
            if (!a.mWindowAdded) {
                a.mWindowAdded = true;
                //调用WindowManageImpl#addView()
                wm.addView(decor, l);
            } 
        }
    } 

    if (!r.activity.mFinished && willBeVisible && r.activity.mDecor != null && !r.hideForNow) {
        if (r.activity.mVisibleFromClient) {
            //这时候DecorView才可见
            r.activity.makeVisible();
        }
    }
}

```

### WindowManagerImpl#addView()
```java
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    //WindowManagerGlobal#addView()    
    mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                    mContext.getUserId());
}


```
### WindowManagerGlobal#addView()
创建ViewRootImpl
将DecorView交给ViewRootImpl处理

```java
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow, int userId) {

    final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;
    if (parentWindow != null) {
        parentWindow.adjustLayoutParamsForSubWindow(wparams);
    } else {
        final Context context = view.getContext();
        if (context != null
                && (context.getApplicationInfo().flags
                        & ApplicationInfo.FLAG_HARDWARE_ACCELERATED) != 0) {
            wparams.flags |= WindowManager.LayoutParams.FLAG_HARDWARE_ACCELERATED;
        }
    }

    ViewRootImpl root;
    synchronized (mLock) {    
        //创建ViewRootImpl
        root = new ViewRootImpl(view.getContext(), display);
        view.setLayoutParams(wparams);
        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);
        try {
            //调用ViewRootImpl#setView()
            root.setView(view, wparams, panelParentView, userId);
        } catch (RuntimeException e) {
        }
    }
}

```
### ViewRootImpl#setView()
* ViewRootImpl最终会触发遍历操作
* 依次执行View的测量、布局、绘制流程
```
public void setView(View view, WindowManager.LayoutParams attrs, 
                    View panelParentView, int userId) {
//大概1015行
    requestLayout();        
}

```
### ViewRootImpl#requestLayout()

```
public void requestLayout() {
    if (!mHandlingLayoutInLayoutRequest) {
        //检查是否在主线程，否则抛出异常
        checkThread();
//mLayoutRequested 是否measure和layout布局。
        mLayoutRequested = true;
        scheduleTraversals();
    }
}
//Android 应用框架中为了更快的响应UI刷新事件在 ViewRootImpl.scheduleTraversals 中使用了同步屏障
void scheduleTraversals() {
    if (!mTraversalScheduled) {
        mTraversalScheduled = true;
 //设置同步障碍，确保mTraversalRunnable优先被执行，发送postSyncBarrier（）
        mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
 //内部通过Handler发送了一个异步消息
        mChoreographer.postCallback(
            Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
        notifyRendererOfFramePending();
        pokeDrawLockIfNeeded();
    }
}

final class TraversalRunnable implements Runnable {
    @Override
    public void run() {
        doTraversal();
    }
}

//Choreographer#postCallback
->postCallbackDelayed->postCallbackDelayedInternal
 private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
        if (DEBUG_FRAMES) {
            Log.d(TAG, "PostCallback: type=" + callbackType
                    + ", action=" + action + ", token=" + token
                    + ", delayMillis=" + delayMillis);
        }

        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;
//action为runnable
            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

            if (dueTime <= now) {
                scheduleFrameLocked(now);
            } else {
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
//设置异步消息用于同步屏障查找使用
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }
```
### ViewRootImpl#doTraversal()
最终调用ViewRootImpl#performTraversals()
```
void doTraversal() {  
 
    performTraversals();
}
 void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
//移除消息同步屏障
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }
   //最终调用View的measure、layout、draw流程
            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```
### ViewRootImpl#performTraversals()
依次调用performMeasure() / performLayout() / performDraw() 完成View的三大流程
 ```
private void performTraversals() {
//2440：dispatchAttachedToWindow，给view.post(runnable) 初始化，用于发送消息
 host.dispatchAttachedToWindow(mAttachInfo, 0);
//2880:调用measure
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
//2938:调用layout（）
    performLayout(lp, mWidth, mHeight);

    /*		
    viewTreeObserver.addOnGlobalLayoutListener(object:ViewTreeObserver.OnGlobalLayoutListener{
            override fun onGlobalLayout() {
               //可以在这里获取View的宽高
            }
    })
    */
    mAttachInfo.mTreeObserver.dispatchOnGlobalLayout();
//3099行调用draw（）
    performDraw();
}

 ```

### ViewRootImpl#performMeasure()
 ```
private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
    if (mView == null) {
        return;
    }

    mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}

 ```
### ViewRootImpl#performLayout()
```
private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,
                           int desiredWindowHeight) {
    final View host = mView;
    if (host == null) {
        return;
    }

    host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
}

```
### ViewRootImpl#performDraw()
```
private void performDraw() {
    boolean canUseAsync = draw(fullRedrawNeeded);
}

private boolean draw(boolean fullRedrawNeeded) {
    drawSoftware(surface, mAttachInfo, xOffset, yOffset,
                 scalingRequired, dirty, surfaceInsets)
}

private boolean drawSoftware(Surface surface, AttachInfo attachInfo, int xoff, int yoff,
                             boolean scalingRequired, Rect dirty, Rect surfaceInsets) {
    mView.draw(canvas);
}

```
##### dispatchAttachedToWindow 绑定window，View和ViewGroup均有该方法。onDetachedFromWindow方法也是一样可参考
##### view.post(runnable)与handle.post(runnable)区别
[View的onAttachedToWindow和onDetachedFromWindow的](https://www.jianshu.com/p/e7b6fa788ae6)
从源码我们可以清晰地看到ViewGroup先是调用自己的onAttachedToWindow()方法，再调用其每个child的onAttachedToWindow()方法，这样此方法就在整个view树中遍布开了，注意到visibility并不会对这个方法产生影响。

```
ViewGroup#dispatchAttachedToWindow
 @Override
    @UnsupportedAppUsage
    void dispatchAttachedToWindow(AttachInfo info, int visibility) {
        mGroupFlags |= FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;
 // 先调用自己的父类也就是View#dispatchAttachedToWindow
        super.dispatchAttachedToWindow(info, visibility);
        mGroupFlags &= ~FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW;

        final int count = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < count; i++) {
            final View child = children[i];
 // 递归调用每个child的dispatchAttachedToWindow方法
           // 典型的深度优先遍历
            child.dispatchAttachedToWindow(info,
                    combineVisibility(visibility, child.getVisibility()));
        }
        final int transientCount = mTransientIndices == null ? 0 : mTransientIndices.size();
        for (int i = 0; i < transientCount; ++i) {
            View view = mTransientViews.get(i);
            view.dispatchAttachedToWindow(info,
                    combineVisibility(visibility, view.getVisibility()));
        }
    }

View#dispatchAttachedToWindow中的实现
 @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
    void dispatchAttachedToWindow(AttachInfo info, int visibility) {
        mAttachInfo = info;
        if (mOverlay != null) {
            mOverlay.getOverlayView().dispatchAttachedToWindow(info, visibility);
        }
        mWindowAttachCount++;
        // We will need to evaluate the drawable state at least once.
        mPrivateFlags |= PFLAG_DRAWABLE_STATE_DIRTY;
        if (mFloatingTreeObserver != null) {
            info.mTreeObserver.merge(mFloatingTreeObserver);
            mFloatingTreeObserver = null;
        }

        registerPendingFrameMetricsObservers();

        if ((mPrivateFlags&PFLAG_SCROLL_CONTAINER) != 0) {
            mAttachInfo.mScrollContainers.add(this);
            mPrivateFlags |= PFLAG_SCROLL_CONTAINER_ADDED;
        }
        // Transfer all pending runnables.
        if (mRunQueue != null) {
            mRunQueue.executeActions(info.mHandler);
            mRunQueue = null;
        }
        performCollectViewAttributes(mAttachInfo, visibility);
        onAttachedToWindow();

        ListenerInfo li = mListenerInfo;
        final CopyOnWriteArrayList<OnAttachStateChangeListener> listeners =
                li != null ? li.mOnAttachStateChangeListeners : null;
        if (listeners != null && listeners.size() > 0) {
            // NOTE: because of the use of CopyOnWriteArrayList, we *must* use an iterator to
            // perform the dispatching. The iterator is a safe guard against listeners that
            // could mutate the list by calling the various add/remove methods. This prevents
            // the array from being modified while we iterate it.
            for (OnAttachStateChangeListener listener : listeners) {
                listener.onViewAttachedToWindow(this);
            }
        }

        int vis = info.mWindowVisibility;
        if (vis != GONE) {
            onWindowVisibilityChanged(vis);
            if (isShown()) {
                // Calling onVisibilityAggregated directly here since the subtree will also
                // receive dispatchAttachedToWindow and this same call
                onVisibilityAggregated(vis == VISIBLE);
            }
        }

        // Send onVisibilityChanged directly instead of dispatchVisibilityChanged.
        // As all views in the subtree will already receive dispatchAttachedToWindow
        // traversing the subtree again here is not desired.
        onVisibilityChanged(this, visibility);

        if ((mPrivateFlags&PFLAG_DRAWABLE_STATE_DIRTY) != 0) {
            // If nobody has evaluated the drawable state yet, then do it now.
            refreshDrawableState();
        }
        needGlobalAttributesUpdate(false);

        notifyEnterOrExitForAutoFillIfNeeded(true);
        notifyAppearedOrDisappearedForContentCaptureIfNeeded(true);
    }
```

#### performTraversals执行dispatchAttachedToWindow，executeActions
```

//view.post(runnable)
 mDataBinding.rxButton.post(new Runnable() {
            @Override
            public void run() {

            }
        });

public boolean post(Runnable action) {
//只有onresume-》performTraversals-》onAttachedToWindow才会给mAttachInfo赋值
        final AttachInfo attachInfo = mAttachInfo;
//所以是在onresume后执行
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
//此处会添加到集合中，等待被执行post->HandleActionQuene#postDelayed
        getRunQueue().post(action);
        return true;
    }
//HandleActionQuene#postDelayed
  public void postDelayed(Runnable action, long delayMillis) {
        final HandlerAction handlerAction = new HandlerAction(action, delayMillis);

        synchronized (this) {
            if (mActions == null) {
                mActions = new HandlerAction[4];
            }
//存储到数组mActions
            mActions = GrowingArrayUtils.append(mActions, mCount, handlerAction);
            mCount++;
        }
    }


  // Execute enqueued actions on every traversal in case a detached view enqueued an action
        getRunQueue().executeActions(mAttachInfo.mHandler);
//上面会走到这执行HandleActionQuene#executeActions，循环遍历
 public void executeActions(Handler handler) {
        synchronized (this) {
            final HandlerAction[] actions = mActions;
            for (int i = 0, count = mCount; i < count; i++) {
                final HandlerAction handlerAction = actions[i];
                handler.postDelayed(handlerAction.action, handlerAction.delay);
            }

            mActions = null;
            mCount = 0;
        }
    }
```

# performLaunchActivity阶段的onAttachedToWindow、onDetachedFromWindow
• 在performLaunchActivity阶段，会创建Activity、PhoneWindow、WindowManageImpl，并回调Activity#onCreate()，接着执行setContentView流程，创建DecorView并将XML布局解析加载到DecorView中。
• 在handleResumeActivity阶段，会调用Activity#onResume()，将Activity与DecorView建立关联，创建ViewRootImpl，将DecorView交给ViewRootImpl处理，最终ViewRootImpl依次执行View的三大流程。
• onAttachedToWindow方法是在Activity resume的时候被调用的，最终交给ViewRootImpl.setView方法，再到requestLayout（）->scheduleTraversals（）->TraversalRunnable#doTraversal（）->performTraversals（）也就是Activity对应的window被添加的时候，且每个view只会被调用一次，父view的调用在前，不论view的visibility状态都会被调用，适合做些view特定的初始化操作；
• onDetachedFromWindow方法是在Activity destroy的时候被调用的，也就是Activity对应的window被删除的时候，且每个view只会被调用一次，父view的调用在后，也不论view的visibility状态都会被调用，适合做最后的清理操作；
这些结论也正好解释了方法名里带有window的原因，有些人可能会想，那为啥不叫  onAttachedToActivity/onDetachedFromActivity，因为在Android里不止是Activity，这里说的内容同样适用于Dialog/Toast，Window只是个虚的概念，是Android抽象出来的，最终操作的实体还是View，这也说明了前面的WindowManager接口为啥是从ViewManager接口派生的，因为所有一切的基石归根结底还是对View的操作。
常见问题


# 常见问题
### invalidate和requestLayout区别
* requestLayout：requestLayout会直接递归调用父View的requestLayout，直到ViewRootImpl，然后触发ViewRoomtImpl的peformTraversals()方法，会依次调用onMeasure()和onLayout()，不一定会触发onDraw()；当在layout过程中发现四个顶点（left、top、right、bottom）和以前不一样时，会触发invalidate，调用onDraw，进行重新绘制。
* invalidate：View的invalidate不会触发ViewRootImpl的invalidate，而是递归调用父View的invalidateChildParent，知道ViewRootImpl的invalidateChildParent，然后触发perfromTraversals，由于mLayoutRequested为false，不会调用onMeasure和onLayout，只会调用onDraw，ViewGroup的onDraw不会被调用（除非设置背景），倒置当前View被重绘。











### Activity中的performCreate
#### 实际上调用了Activity 的onCreate（）和
```
    final void performCreate(Bundle icicle, PersistableBundle persistentState) {

1、调用Activity的onCreate方法
   if (persistentState != null) {
            onCreate(icicle, persistentState);
        } else {
            onCreate(icicle);
        }
2、调用Fragment的onCreate（）
 mFragments.dispatchActivityCreated();
FragmentController.java
public void dispatchActivityCreated() {
        mHost.mFragmentManager.dispatchActivityCreated();
    }
FragmentHostCallback.java
  final FragmentManagerImpl mFragmentManager = new FragmentManagerImpl();

FragmentManager.java
 public void dispatchActivityCreated() {
        mStateSaved = false;
        dispatchMoveToState(Fragment.ACTIVITY_CREATED);
    }
}

```

1、System.core.init.cpp文件的main方法

2、main方法会有LoadBootScripts()加载init.rc文件

3、 里面会有trigge -zygote-start方法启动init.zygote64.rc

4、再加载serve zygote System.core.app_process64.rc命令启动app_process.rc中的音频，媒体，网络一些基本任务，从zygote中启动。

5、zygote fork出SystemService，开启binder线程池，开启SystemServiceManager，再就是AMS,PMS，WMS八十多个服务，startCoreService,startOtherService。

package/apps/定制应用

![微信图片_20211215220616.png](https://upload-images.jianshu.io/upload_images/2981395-f5cfc8ac453deba1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

