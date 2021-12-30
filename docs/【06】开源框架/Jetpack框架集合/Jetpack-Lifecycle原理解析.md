
# [官网Lifecycle文档](https://developer.android.com/reference/android/arch/lifecycle/Lifecycle)

# 一：Lifecycle的集成

## 1.1采用下面的方式，添加注解和集成implementation找到默认的DefaultLifecycleObserver
```
老版本  
    def lifecycle_version = "2.4.0"
annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
    // alternately - if using Java8, use the following instead of lifecycle-compiler
  implementation "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"
      
新版本
  // Annotation processor
    annotationProcessor "androidx.lifecycle:lifecycle-compiler:$lifecycle_version"
    // alternately - if using Java8, use the following instead of lifecycle-compiler存在DefaultLifecycleObserver
    api "androidx.lifecycle:lifecycle-common-java8:$lifecycle_version"
    //暂时还么有生效，采用lifecycle-common-java8:
//    api "androidx.lifecycle:lifecycle-common:$lifecycle_version"

```
截止2021.12.17更新的时候发现最新并不是2.4.0，二是2.3.1，但是该版本并没有默认的DefaultLifecycleObserver，所以还是改用老方式，或者自己实现FullLifecycleObserver是一样的结果，自己复写里面的oncreate等方法，实现需要的逻辑
![image.png](https://upload-images.jianshu.io/upload_images/2981395-d18eb5698397b8d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## ![image.png](https://upload-images.jianshu.io/upload_images/2981395-fd6d384da56ff486.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "image.png") 
![image.png](https://upload-images.jianshu.io/upload_images/2981395-7d1968000493936b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 二：使用样例

### 2.1推荐使用DefaultLifecycleObserver或者LifecycleEventObserver 
> [官网说明](https://developer.android.com/jetpack/androidx/releases/lifecycle#lifecycle-*-2.4.0)：自 2.3.0 以来的重要变更废弃了 @OnLifecycleEvent。应改用 LifecycleEventObserver 或 DefaultLifecycleObserver。
##### 2.1.1:创建观察者，默认继承DefaultLifecycleObserver，实现里面的默认生命周期
```
package com.cc.qq.jetpack;


import androidx.annotation.NonNull;
import androidx.lifecycle.DefaultLifecycleObserver;
import androidx.lifecycle.LifecycleOwner;

public class LifeCycleListener implements DefaultLifecycleObserver {
    @Override
    public void onCreate(@NonNull LifecycleOwner owner) {

    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {

    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {
        
    }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) {

    }
}

```

#### 2.1.2：MainActivity添加创建的Observer：LifeCycleListener

```
package com.cc.qq.jetpack;

import android.os.Bundle;

import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;

import com.cc.qq.R;


public class LifeCyclerActivity extends AppCompatActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_rx);
        getLifecycle().addObserver(new LifeCycleListener());
    }
}

```

# 三：原理分析

#### 3.1：LifeCyclerActivity层级关系

![image.png](https://upload-images.jianshu.io/upload_images/2981395-61fdb7ba3433d583.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240 "image.png") 

#### 3.2：分析父类在app/ComponetActivity和activity/CompoentActivity

*   1、两个目录下的Activity均存在一个ReportFragment
*   2、两个ComponetActivity均实现了LifecycleOwner接口
*   3、LifecycleOwner中持有一个Lifecycle生命周期，在两个ComponetActivity都实现了LifecycleOwner接口的getLifecycle（）方法获取生命周期，该生命周期实现类为LifecycleRegistry

##### 3.2.1：先看看ComponentActivity

```
1、 app/ComponetActivity和activity/CompoentActivity
@SuppressLint("RestrictedApi")
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ReportFragment.injectIfNeededIn(this);
    }

    private LifecycleRegistry mLifecycleRegistry = new LifecycleRegistry(this);
	
	@NonNull
    @Override
    public Lifecycle getLifecycle() {
        return mLifecycleRegistry;
    }
```

##### 3.2.2：看看 ReportFragment中的injectIfNeededIn创建Fragment，Fragment中还有生命周期和dispatch方法

当ReportFragment生命周期变化的时候会调用dispatch（event）发方法，犹豫Activity实现了LifecycleOwner接口，所以会调用LifecycleRegistry#handleLifecycleEvent（）

```
public static void injectIfNeededIn(Activity activity) {
        // ProcessLifecycleOwner should always correctly work and some activities may not extend
        // FragmentActivity from support lib, so we use framework fragments for activities
        android.app.FragmentManager manager = activity.getFragmentManager();
        if (manager.findFragmentByTag(REPORT_FRAGMENT_TAG) == null) {
            manager.beginTransaction().add(new ReportFragment(), REPORT_FRAGMENT_TAG).commit();
            // Hopefully, we are the first to make a transaction.
            manager.executePendingTransactions();
        }
    }

    @Override
    public void onStop() {
        super.onStop();
        dispatch(Lifecycle.Event.ON_STOP);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        dispatch(Lifecycle.Event.ON_DESTROY);
        // just want to be sure that we won't leak reference to an activity
        mProcessListener = null;
    }

private void dispatch(Lifecycle.Event event) {
        Activity activity = getActivity();
        if (activity instanceof LifecycleRegistryOwner) {
            ((LifecycleRegistryOwner) activity).getLifecycle().handleLifecycleEvent(event);
            return;
        }

        if (activity instanceof LifecycleOwner) {
            Lifecycle lifecycle = ((LifecycleOwner) activity).getLifecycle();
            if (lifecycle instanceof LifecycleRegistry) {
                ((LifecycleRegistry) lifecycle).handleLifecycleEvent(event);
            }
        }
    }
```

##### 3.2.3：LifecycleRegistry#handleLifecycleEvent方法

根据handleLifecycleEvent传递进来的Lifecycle.Event 参数，返回对应的State

```
1、LifecycleRegistry#handleLifecycleEvent
public void handleLifecycleEvent(@NonNull Lifecycle.Event event) {
        State next = getStateAfter(event);
        moveToState(next);
    }
2、LifecycleRegistry#getStateAfter(event);
static State getStateAfter(Event event) {
        switch (event) {
            case ON_CREATE:
            case ON_STOP:
                return CREATED;
            case ON_START:
            case ON_PAUSE:
                return STARTED;
            case ON_RESUME:
                return RESUMED;
            case ON_DESTROY:
                return DESTROYED;
            case ON_ANY:
                break;
        }
        throw new IllegalArgumentException("Unexpected event value " + event);
    }
3、LifecycleRegistry#3、LifecycleRegistry#moveToState
 private void moveToState(State next) {
        if (mState == next) {
            return;
        }
        mState = next;
        if (mHandlingEvent || mAddingObserverCounter != 0) {
            mNewEventOccurred = true;
            // we will figure out what to do on upper level.
            return;
        }
        mHandlingEvent = true;
        sync();
        mHandlingEvent = false;
    }
4、LifecycleRegistry#sync
private void sync() {
		//得到的是Activity
        LifecycleOwner lifecycleOwner = mLifecycleOwner.get();
        if (lifecycleOwner == null) {
            throw new IllegalStateException("LifecycleOwner of this LifecycleRegistry is already"
                    + "garbage collected. It is too late to change lifecycle state.");
        }
        //由下面5处得知isSynced为false,则继续往下执行。
        //枚举比较大小，越前面越小，
        //a、如果新传递过来的生命周期状态比所有观察者中最先添加进去的观察者生命周期还小
        //那么执行backwardPass（lifecycleOwner被观察者Activity），什么情况下会触发次场景呢？
        //b、如果新传递过来的生命周期状态比所有观察者中最先添加进去的观察者生命周期还大
        //那么执行forwardPass（lifecycleOwner被观察者Activity），什么情况下会触发次场景呢？
        
        while (!isSynced()) {
            mNewEventOccurred = false;
            // no need to check eldest for nullability, because isSynced does it for us.
            if (mState.compareTo(mObserverMap.eldest().getValue().mState) < 0) {
                backwardPass(lifecycleOwner);
            }
            Entry<LifecycleObserver, ObserverWithState> newest = mObserverMap.newest();
            if (!mNewEventOccurred && newest != null
                    && mState.compareTo(newest.getValue().mState) > 0) {
                forwardPass(lifecycleOwner);
            }
        }
        mNewEventOccurred = false;
    }
 5、 LifecycleRegistry#isSynced
 private boolean isSynced() {
        if (mObserverMap.size() == 0) {
            return true;
        }
        //匹配里面mObserverMap所有的观察者最老的和最新的状态是否一致
        State eldestObserverState = mObserverMap.eldest().getValue().mState;
        State newestObserverState = mObserverMap.newest().getValue().mState;
        //先检查是否一致
        //mState = INITIALIZED;mState默认初始状态
        //当Fragment生命周期发生变化时将Event生命周期事件传递过来moveToState（event）
        //传递过来后调用mState=event，
        //判断条件1是否只存在一个观察者：一般不止一个
        //判断条件2最新的观察者生命周期是否为传递过来的：一般不相等，所以一般是false，走到上面分析4处
        return eldestObserverState == newestObserverState && mState == newestObserverState;
    }
 6、LifecycleRegistry#mObserverMap是一个存有State的观察者，addObserve的时候添加值
 private FastSafeIterableMap<LifecycleObserver, ObserverWithState> mObserverMap =
            new FastSafeIterableMap<>();
7、
public enum State {
    //按照这个正向走则是forwardPass，因为传递过来的往后，相当于创建Activity的过程。
    //逆向走则是往后backwardPass，因为是倒着走可以看出来是生命周期结束的过程
    DESTROYED，INITIALIZED，CREATED，STARTED，RESUMED
}
8、
public enum Event {
    ON_CREATE，ON_START，ON_RESUME，ON_PAUSE，ON_STOP，ON_DESTROY，ON_ANY
}

private static Event downEvent(State state) {
        switch (state) {
            case INITIALIZED:
                throw new IllegalArgumentException();
            case CREATED:
                return ON_DESTROY;
            case STARTED:
                return ON_STOP;
            case RESUMED:
                return ON_PAUSE;
            case DESTROYED:
                throw new IllegalArgumentException();
        }
        throw new IllegalArgumentException("Unexpected state value " + state);
    }

    private static Event upEvent(State state) {
        switch (state) {
            case INITIALIZED:
            case DESTROYED:
                return ON_CREATE;
            case CREATED:
                return ON_START;
            case STARTED:
                return ON_RESUME;
            case RESUMED:
                throw new IllegalArgumentException();
        }
        throw new IllegalArgumentException("Unexpected state value " + state);
    }



9、
  private void forwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> ascendingIterator =
                mObserverMap.iteratorWithAdditions();
        //循环遍历mObserverMap观察者
        while (ascendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = ascendingIterator.next();
            ObserverWithState observer = entry.getValue();
            //小于《状态顺流走，也就是往右走，往下走，往前走
            while ((observer.mState.compareTo(mState) < 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                    //添加新的状态进去
                pushParentState(observer.mState);
                //观察者分发事件，执行装饰模式后的Observable的onStateChanged（）
                //DefaultLifecycleObserver extends FullLifecycleObserver，
                //FullLifecycleObserverAdapter extends LifecycleEventObserver extends LifecycleObserver
                //FullLifecycleObserverAdapter#onStateChanged（）通过switch语句判断生命周期，
                //从而让观察者调用自己实现的生命周期方法，就是上面自定义的LifecycleListener，可以换一个名字。。
                //注意：如果直接 extends LifecycleObserver，那需要添加事件注解，通过反射来达到目的，损失性能
                所以官方推荐DefaultLifecycleObserver或者LifecycleEventObserver
                observer.dispatchEvent(lifecycleOwner, upEvent(observer.mState));
                popParentState();
            }
        }
    }

//同forwardPass
    private void backwardPass(LifecycleOwner lifecycleOwner) {
        Iterator<Entry<LifecycleObserver, ObserverWithState>> descendingIterator =
                mObserverMap.descendingIterator();
        while (descendingIterator.hasNext() && !mNewEventOccurred) {
            Entry<LifecycleObserver, ObserverWithState> entry = descendingIterator.next();
            ObserverWithState observer = entry.getValue();
            while ((observer.mState.compareTo(mState) > 0 && !mNewEventOccurred
                    && mObserverMap.contains(entry.getKey()))) {
                Event event = downEvent(observer.mState);
                pushParentState(getStateAfter(event));
                observer.dispatchEvent(lifecycleOwner, event);
                popParentState();
            }
        }
    }
```

## 结论：

1、以上说明Activity实现了LifecycleOwner并持有一个生命周期，在创建Activity的时候也同时创建了ReportFragment，ReportFragment生命周期变化时会触发dispatch事件。

2、由LifecycleRegistry让MainActivity得到一个Lifecycle，也就是MainActivity通过实现了LifecycleOwner持有生命周期Lifecycle。

3、通过添加addObserver(观察者)得到一个ObserverWithState对象，将ObserverWithState对象添加到mObserverMap中。

4、创建ObserverWithState该对象的时候会创建新的装饰的观察者FullLifecycleObserverAdapter implements LifecycleEventObserver。有个onStateChange()方法，该方法由上面的1生命周期变化的时候触发。onStateChange该方法有个switch判断生命周期事件，从而调用我们自己创建的观察者对应的方法。

5、ReportFragment什么周期变化时触发的dispatch会经过一系列转换，最后通过forwardPass/backwardPass调用FullLifecycleObserverAdapter 的onStateChange。转换过程中涉及到的类LifecycleRegister。
