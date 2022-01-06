# Android常用Library葵花宝典

## [Android官网](https://developer.android.com)

## 针对 MultiDex 配置应用65536

#### 直接设置 minSdkVersion 21则不需要配置

[multidex方法数超标：](https://developer.android.com/studio/build/multidex)https://developer.android.com/studio/build/multidex

Android 5.0（API 级别 21）及更高版本使用名为 ART 的运行时，它本身支持从 APK 文件加载多个 DEX 文件。ART 在应用安装时执行预编译，这会扫描查找 `classesN.dex` 文件，并将它们编译成单个 `.oat` 文件，以供 Android 设备执行。因此，如果您的 `minSdkVersion` 为 21 或更高版本，系统会默认启用 MultiDex，并且您不需要 MultiDex 库。

因此，如果您的 `minSdkVersion` 设为 21 或更高版本，系统会默认启用 MultiDex，并且您不需要 MultiDex 库。

不过，如果您的 `minSdkVersion` 设为 20 或更低版本，您必须使用 [MultiDex 库](https://developer.android.com/jetpack/androidx/releases/multidex)并对应用项目进行以下修改：

```java
Caused by: com.android.tools.r8.utils.b: Cannot fit requested classes in a single dex file (# methods: 78542 > 65536)
```



1. 修改模块级 `build.gradle` 文件以启用 MultiDex，并将 MultiDex 库添加为依赖项，如下所示：

```groovy
android {
    defaultConfig {
        ...
        minSdkVersion 15
        targetSdkVersion 28
        multiDexEnabled true
    }
    ...
}

dependencies {
    implementation "androidx.multidex:multidex:2.0.1"
}
```

2、根据您是否替换 [`Application`](https://developer.android.com/reference/android/app/Application) 类，执行以下某项操作：

- 如果您不替换 [`Application`](https://developer.android.com/reference/android/app/Application) 类，请修改清单文件以设置 `` 标记中的 `android:name`，如下所示：

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application
            android:name="androidx.multidex.MultiDexApplication" >
        ...
    </application>
</manifest>
```

* 如果您替换 [`Application`](https://developer.android.com/reference/android/app/Application) 类，请对其进行更改以扩展 `MultiDexApplication`（如果可能），如下所示：

```java
public class MyApplication extends MultiDexApplication { ... }

class MyApplication : MultiDexApplication() {...}
```

* 或者，如果您替换 [`Application`](https://developer.android.com/reference/android/app/Application) 类，但无法更改基类，则可以改为替换 [`attachBaseContext()`](https://developer.android.com/reference/android/content/ContextWrapper#attachBaseContext(android.content.Context)) 方法并调用 `MultiDex.install(this)` 以启用 MultiDex：

```java
public class MyApplication extends SomeOtherApplication {
  @Override
  protected void attachBaseContext(Context base) {
     super.attachBaseContext(base);
     MultiDex.install(this);
  }
}

class MyApplication : SomeOtherApplication() {

    override fun attachBaseContext(base: Context) {
        super.attachBaseContext(base)
        MultiDex.install(this)
    }
}
```

