# JetPack之DataBinding的使用

## 一、概述

DataBinding可以实现数据单向或者双向的绑定，让开发比较方便。通过数据变化更新UI，完全由数据驱动。

## 二、使用

### 1、Module.gradle集成DataDinding

```groovy
android {
	...
	//新版本添加这行，标识开启了viewBinding和dataBinding，上面过时
    buildFeatures{
        viewBinding=true
        dataBinding=true
    }

}
```

##### 完整版

```groovy
plugins {
    id 'com.android.application'
    //使用Kotlin插件
    id 'kotlin-android'
}

android {
    compileSdkVersion 30

    defaultConfig {
        applicationId "com.cc.learn"
        minSdkVersion 21
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

//    viewBinding {
//        enabled = true
//    }
//    dataBinding {
//        enabled = true
//    }
    //新版本添加这行，标识开启了viewBinding和dataBinding，上面过时
    buildFeatures{
        viewBinding=true
        dataBinding=true
    }
}

dependencies {
    //添加Kotlin 标准库
    implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"

    implementation 'org.jetbrains:annotations:15.0'

    testImplementation 'junit:junit:4.+'

    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'

    //AndroidX https://developer.android.google.cn/jetpack/androidx/releases/appcompat 找版本说明，直接复制

    def appcompat_version = "1.3.1"
    implementation "androidx.appcompat:appcompat:$appcompat_version"
    implementation "androidx.appcompat:appcompat-resources:$appcompat_version"
    implementation "androidx.coordinatorlayout:coordinatorlayout:1.1.0"
    implementation "androidx.constraintlayout:constraintlayout:2.1.0"
    implementation "androidx.recyclerview:recyclerview:1.2.1"
    implementation "androidx.cardview:cardview:1.0.0"
    implementation 'com.google.android.material:material:1.2.0'
    implementation 'com.jakewharton.rxbinding4:rxbinding:4.0.0'
    def lifecycle_version = "2.2.0"
//    implementation "androidx.lifecycle:lifecycle-livedata-ktx:2.2.0"
//    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"

    //RxJava内存自动释放管理：AutoDispose 2.x
    implementation 'com.uber.autodispose2:autodispose:2.1.0'
    implementation 'com.uber.autodispose2:autodispose-lifecycle:2.1.0'
    implementation 'com.uber.autodispose2:autodispose-android:2.1.0'
    implementation 'com.uber.autodispose2:autodispose-androidx-lifecycle:2.1.0'

    //https://github.com/ReactiveX/RxAndroid
    implementation 'io.reactivex.rxjava3:rxandroid:3.0.0'
    //https://github.com/ReactiveX/RxJava
    implementation 'io.reactivex.rxjava3:rxjava:3.1.0'


    //AndroidX kotlin

    //implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    //implementation 'androidx.appcompat:appcompat:1.2.0'
    //implementation 'androidx.core:core-ktx:1.3.2'
    //implementation 'androidx.constraintlayout:constraintlayout:2.0.4'

    //baserecyclerview
    implementation "com.github.CymChad:BaseRecyclerViewAdapterHelper:$baseRecyclerViewAdapter"

    annotationProcessor rootProject.googleAutoService
    implementation rootProject.googleAutoService
    implementation rootProject.squareJavaPoet
}
```

### 2、实现xml

#### 最外层包裹外layout，快捷键alt+enter

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <Button
            android:id="@+id/jetpack_btn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="JetPack组件-lifecycle等使用"
            app:layout_constraintTop_toTopOf="parent" />

        <Button
            android:id="@+id/singleinstance_btn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="singleinstance_的使用"
            app:layout_constraintTop_toBottomOf="@id/login_btn" />

		....
        <Button
            android:id="@+id/AutoService"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="AutoService的使用"
            app:layout_constraintTop_toBottomOf="@id/recyclerview" />


        <TextView
            android:id="@+id/textview_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="textview_text"
            app:layout_constraintBottom_toBottomOf="parent" />


    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

#### 3、Activity中添加测试代码

```java
package com.cc.xiangxue;

import android.app.Activity;
import android.app.ActivityManager;
import android.content.Intent;
import android.os.Build;
import android.os.Bundle;
import android.util.ArrayMap;
import android.view.View;

import androidx.annotation.Nullable;
import androidx.annotation.RequiresApi;
import androidx.appcompat.app.AppCompatActivity;
import androidx.databinding.DataBindingUtil;
import androidx.databinding.ViewDataBinding;
import androidx.viewbinding.ViewBinding;

import com.cc.xiangxue.databinding.ActivityMainBinding;
import com.cc.xiangxue.jetpack.JetpackActivity;
import com.cc.xiangxue.login.LoginActivity;
import com.cc.xiangxue.multipletask.WXMultipleTaskActivity;
import com.cc.xiangxue.rv.RVActivity2;
import com.cc.xiangxue.rxjava.RxJavaActivity;
import com.cc.xiangxue.service.BroadCastActivity;
import com.cc.xiangxue.service.ProgressServiceActivity;
import com.cc.xiangxue.service.ServiceActivity;

@RequiresApi(api = Build.VERSION_CODES.KITKAT)
public class MainActivity extends AppCompatActivity implements View.OnClickListener {
    private ActivityMainBinding mBinding;
//    private ViewDataBinding mBinding;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //viewding的使用
//        viewbinding = ActivityMainBinding.inflate(getLayoutInflater());
//        View view = viewbinding.getRoot();
//        setContentView(view);
        mBinding=DataBindingUtil.setContentView(this,R.layout.activity_main);
//        viewbinding.jetpackBtn.setOnClickListener(this);

    }


    @Override
    protected void onActivityResult(int requestCode, int resultCode,
                                    @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case 1:
                String str = data.getStringExtra("return1");
                break;
            default:
                break;
        }
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.jetpack_btn:
                startActivity(new Intent(this, JetpackActivity.class));
                break;
            case R.id.rxjava_btn:
                startActivity(new Intent(this, RxJavaActivity.class));
                break;
            case R.id.login_btn:
                startActivity(new Intent(this, LoginActivity.class));
                break;
            case R.id.singleinstance_btn:
                //绑定唯一标识 tag，存在 则根据 taskId 移动到前台
//                if (isActive("tag")) {
//                    resumeActivty("tag");
//                } else {
//                    add(this);
//                    Intent intent = new Intent(this, WXMultipleTaskActivity.class);
//
//                    //打开多任务窗口 flags
//                    intent.addFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT);
//                    intent.addFlags(Intent.FLAG_ACTIVITY_MULTIPLE_TASK);
//                    intent.putExtra("tag", "tag");
//                    this.startActivity(intent);
//                }
//                https://blog.csdn.net/gitzzp/article/details/54579375?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link

                startActivity(new Intent(this, WXMultipleTaskActivity.class));
                break;
            case R.id.service_btn:
                startActivity(new Intent(this, ServiceActivity.class));
                break;
            case R.id.service_progress_btn:
                startActivity(new Intent(this, ProgressServiceActivity.class));
                break;
            case R.id.service_broadcast_progress_btn:
                startActivity(new Intent(this, BroadCastActivity.class));
                break;
            case R.id.recyclerview:
                startActivity(new Intent(this, RVActivity2.class));
                break;
            default:
                break;
        }
    }

    /**
     * 判断 tag绑定的 MyActivity是否存在
     * https://blog.csdn.net/qq_29364417/article/details/109379915
     */
    private ArrayMap<String, MainActivity> mArrayMap = new ArrayMap<>();

    public boolean isActive(String tag) {
        Activity activity = mArrayMap.get(tag);
        if (activity != null) {
            //判断是否为 MyActivity,如果已经销毁，则移除
            if (activity.isFinishing() || activity.isDestroyed()) {
                mArrayMap.remove(tag);
                return false;
            } else {
                return true;
            }
        } else {
            return false;
        }
    }

    /**
     * 找到tag 绑定的 MyActivty ，通过 getTaskId() 移动到前台
     */
    public void resumeActivty(String tag) {
        Activity activity = mArrayMap.get(tag);
        if (activity != null && !activity.isFinishing() && !activity.isDestroyed()) {
            ActivityManager am = (ActivityManager) activity.getSystemService(ACTIVITY_SERVICE);
            //返回启动它的根任务（home 或者 MainActivity）
            am.moveTaskToFront(activity.getTaskId(), ActivityManager.MOVE_TASK_NO_USER_ACTION);
        }
    }

    /**
     * 把Activity添加到管理中
     */
    public void add(Activity activity) {
        if (activity != null) {
            //如果是 MyActivity 则缓存起来
            if (activity instanceof MainActivity) {
//                String tag = ((MainActivity) activity).getTag();
                //存入内存中
                mArrayMap.put("tag", (MainActivity) activity);
            }
        }
    }

}
```

#### 4、集成中问题

1、Activity需要集成AppCompatActivity
2、写完DataBindingUtil.setContentView(this, R.layout.activity_main)后，可以输入ActivityMainD...让编译器补全这个变量。这个变量是Android生成的，自己手写可能会有误差。

常用https://blog.csdn.net/huangbin123/article/details/105111818/



### 常用写法

https://www.jianshu.com/p/9e49f0ed1dfc

