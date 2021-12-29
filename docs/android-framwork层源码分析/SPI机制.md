

# SPI机制

#### 一、SPI说明

[Android模块开发之SPI]: https://www.jianshu.com/p/deeb39ccdc53



> **其实就是为某个接口寻找服务的机制，有点类似IOC的思想，将装配的控制权移交给ServiceLoader**。
>
> 不同的模块可以基于接口编程，每个模块有不同的实现service provider，然后通过SPI机制自动注册到一个配置文件中，就可以实现在程序运行时扫描加载同一接口的不同service provider。
>
> 这样模块之间不会基于实现类硬编码，可插拔。
>
> 策略模式，为接口实现做匹配耦合度过高（只能修改）
>
> 通过一种方案主动为接口发现实现类，以写文件的形势实现，但是并不是为了解决耦合，是为接口提供服务的机制。服务发现机制
>
> 提供一个API把接口的实现类绑定进来

注意：

**1、在哪个模块实现了接口，该配置文件就在哪模块生成**

#### 二、实现步骤

##### 1.接口

​		首先就是模块之间通信接口的实现，我们这里单独抽出一个模块interface，后面接口的不同实现模块可以都依赖同一个接口模块。接口里面就是一个简单的接口：

```java
package com.example;

public interface Display {
    String display();
}
```



##### 2.模块实现	

- ###### 接下来就是用不同的模块实现接口，

  首先需要在模块的build.gradle中加入接口的依赖：然后一个简单的实现类实现接口Display。

```gradle
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    compile project(':interface')
}
```



- ###### adisplay模块的实现类就是ADisplay。

```java
package com.example;

public class ADispaly implements Display{
    @Override
    public String display() {
        return "This is display in module adisplay";
    }
}
```



- ###### bdisplay模块的实现类就是BDisplay。

```java


package com.example;

public class BDisplay implements Display{
    @Override
    public String display() {
        return "This is display in module bdisplay";
    }
}
```



- ###### 为接口实现类创建配置文件

接着就是要把这些接口实现类注册到配置文件中，spi的机制就是注册到META-INF/services中。

首先在src/main目录下，也就是java的同级目录中新建一个包目录resources，然后在resources新建一个目录META-INF/services，再新建一个file，file的名称就是接口的全限定名，在我们的栗子中就是：com.example.Display，文件中的内容就是不同实现类的全限定名，不同实现类分别一行。

**详细地址ModuleName/src/main/resources>META-INF.services/packageName.ClassName**

![img](https://upload-images.jianshu.io/upload_images/2608779-9096d6fcc8692872.png?imageMogr2/auto-orient/strip|imageView2/2/w/340/format/webp)



##### 3、APP模块加载不同服务

调用

在配置文件`com.example.Display`中的内容就是`com.example.juexingzhe.spidemo.DisplayImpl`
 在界面上放置一个按钮，点击按钮会记载所有模块配置文件中的实现类：

```java
mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loadModule();
            }
});

```



当然在主程序模块中也可以有自己的接口实现：

```java
package com.example.juexingzhe.spidemo;

import com.example.Display;

public class DisplayImpl implements Display {
    @Override
    public String display() {
        return "This is display in module app";
    }
}
```

##### MainActivity.java

```java
package com.example.juexingzhe.spidemo;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity {

    TextView mTextView;
    Button mButton;
    StringBuilder mStringBuilder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mTextView = (TextView)findViewById(R.id.tv);
        mButton = (Button)findViewById(R.id.btn);

        mStringBuilder = new StringBuilder();

        mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loadModule();
            }
        });

    }

    private void loadModule(){
        String display = "";
        mButton.setClickable(false);
        while (DisplayFactory.getSingleton().hasNextDisplay()){
            display = DisplayFactory.getSingleton().getDisplay().display();
            mStringBuilder.append(display).append("\n");
        }
        mTextView.setText(mStringBuilder.toString());
    }



}
```

##### DisplayFactory.java

```java
package com.example.juexingzhe.spidemo;

import com.example.Display;

import java.util.Iterator;
import java.util.ServiceLoader;

/**
 * Created by juexingzhe on 2017/6/3.
 */

class DisplayFactory {

    private static DisplayFactory mDisplayFactory;

    private Iterator<Display> mIterator;

    private DisplayFactory() {
        ServiceLoader<Display> loader = ServiceLoader.load(Display.class);
        mIterator = loader.iterator();
    }

    static DisplayFactory getSingleton() {
        if (null == mDisplayFactory) {
            synchronized (DisplayFactory.class) {
                if (null == mDisplayFactory) {
                    mDisplayFactory = new DisplayFactory();
                }
            }
        }
        return mDisplayFactory;
    }

    Display getDisplay() {
        return mIterator.next();
    }

    boolean hasNextDisplay() {
        return mIterator.hasNext();
    }

}
```







通过ServiceLoader来加载接口的不同实现类，然后会得到迭代器，在迭代器中可以拿到不同实现类全限定名，然后通过反射动态加载实例就可以调用display方法了。**

```java
ServiceLoader<Display> loader = ServiceLoader.load(Display.class);
mIterator = loader.iterator();
while (mIterator.hasNext()){
      mIterator.next().display();
}
```



#### 三、源码分析

- 主要工作都是在在**java.util包的ServiceLoader**中。
- 先看下几个重要的成员变量, **PREFIX**就是配置文件所在的包目录路径；
- service就是接口名称，在我们这个例子中就是**Display**；
- loader就是类加载器，其实最终都是通过反射加载实例；
- providers就是不同实现类的缓存，key就是实现类的全限定名，value就是实现类的实例；
- lookupIterator就是内部类LazyIterator的实例。主要工作都是在在**java.util包的ServiceLoader**中。先看下几个重要的成员变量, **PREFIX**就是配置文件所在的包目录路径；service就是接口名称，在我们这个例子中就是**Display**；loader就是类加载器，其实最终都是通过反射加载实例；providers就是不同实现类的缓存，key就是实现类的全限定名，value就是实现类的实例；lookupIterator就是内部类LazyIterator的实例。



```java

    private static final String PREFIX = "META-INF/services/";

    // The class or interface representing the service being loaded
    private Class<S> service;

    // The class loader used to locate, load, and instantiate providers
    private ClassLoader loader;

    // Cached providers, in instantiation order
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // The current lazy-lookup iterator
    private LazyIterator lookupIterator;
```

##### **重点三步骤**

```java
1、ServiceLoader<Display> loader = ServiceLoader.load(Display.class);

2、mIterator = loader.iterator();

3、while (mIterator.hasNext()){
      mIterator.next().display();
}
```

**1、第一个步骤load**：ServiceLoader提供了两个静态的load方法,如果我们没有传入类加载器，ServiceLoader会自动为我们获得一个当前线程的类加载器，最终都是调用构造函数。

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl =Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,ClassLoader loader)
{
        return new ServiceLoader<>(service, loader);
}
```

在构造函数中工作很简单就是清除实现类的缓存，实例化迭代器。

```java
public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = svc;
    loader = cl;
    reload();
}

private LazyIterator(Class<S> service, ClassLoader loader) {
            this.service = service;
            this.loader = loader;
}
```

**注意了，我们在外面通过ServiceLoader.load(Display.class)并不会去加载service provider(？)，也就是懒加载的设计模式，这也是Java SPI的设计亮点。**

那么service provider在什么地方进行加载？我们接着看:

**2、第二个步骤`loader.iterator()`**其实就是返回一个迭代器。我们看下官方文档的解释,这个就是懒加载实现的地方，首先会到providers中去查找有没有存在的实例，有就直接返回，没有再到LazyIterator中查找。

```
 * Lazily loads the available providers of this loader's service.
 
 * <p> The iterator returned by this method first yields all of the
 * elements of the provider cache, in instantiation order.  It then lazily
 * loads and instantiates any remaining providers, adding each one to the
 * cache in turn.
```

```java
public Iterator<S> iterator() {
        return new Iterator<S>() {
			//1-------------
            Iterator<Map.Entry<String,S>> knownProviders
                = providers.entrySet().iterator();

            public boolean hasNext() {
                if (knownProviders.hasNext())
                    return true;
                return lookupIterator.hasNext();
            }

            public S next() {
                if (knownProviders.hasNext())
                    return knownProviders.next().getValue();
                return lookupIterator.next();
            }

            public void remove() {
                throw new UnsupportedOperationException();
            }

        };
}
```

上面片段代码1处我们一开始**providers**中肯定是没有缓存的实例的，接着会到LazyIterator中查找，去看看LazyIterator,先看下hasNext()方法。

>1、首先拿到配置文件名fullName,我们这个例子中是`com.example.Display`
2、通过类加载器获得所有模块的配置文件`Enumeration configs configs`
3、依次扫描每个配置文件的内容，返回配置文件内容`Iterator pending`，每个配置文件中可能有多个实现类的全限定名，所以pending也是个迭代器。

```java
public boolean hasNext() {
    if (nextName != null) {
        return true;
    }
    if (configs == null) {
        try {
            //1、首先拿到配置文件名fullName,我们这个例子中是`com.example.Display`
            String fullName = PREFIX + service.getName();
            //2、通过类加载器获得所有模块的配置文件`Enumeration configs configs`
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
                configs = loader.getResources(fullName);
        } catch (IOException x) {
            fail(service, "Error locating configuration files", x);
        }
    }
    //3、依次扫描每个配置文件的内容，返回配置文件内容`Iterator pending`，每个配置文件中可能有多个实现类的全限定名，所以pending也是个迭代器。
    while ((pending == null) || !pending.hasNext()) {
        if (!configs.hasMoreElements()) {
            return false;
        }
        pending = parse(service, configs.nextElement());
    }
    nextName = pending.next();
    return true;
}
```

在上面**hasNext()**方法中拿到的**nextName**就是实现类的全限定名，接下来我们去看看具体实例化工作的地方**next()**:

>1.首先根据**nextName，Class.forName**加载拿到具体实现类的**class对象**
2.**Class.newInstance()**实例化拿到**具体实现类**的**实例对象**
3.将实例对象转换service.cast为**接口**
4.将**实例对象放到缓存中**，providers.put(cn, p)，key就是实现类的全限定名，value是实例对象。
5.返回实例对象

```java
public S next() {
    if (!hasNext()) {
        throw new NoSuchElementException();
    }
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    try {
        //1.首先根据**nextName，Class.forName**加载拿到具体实现类的**class对象**
        c = Class.forName(cn, false, loader);
    } catch (ClassNotFoundException x) {
        fail(service,
             "Provider " + cn + " not found", x);
    }
    if (!service.isAssignableFrom(c)) {
        ClassCastException cce = new ClassCastException(
                service.getCanonicalName() + " is not assignable from " + c.getCanonicalName());
        fail(service,
             "Provider " + cn  + " not a subtype", cce);
    }
    try {
       // 3.将实例对象转换service.cast为**接口**
        S p = service.cast(c.newInstance());
        //4.将实例对象放到缓存中，providers.put(cn, p)，key就是实现类的全限定名，value是实例对象。
        providers.put(cn, p);
        //5.返回实例对象
        return p;
    } catch (Throwable x) {
        fail(service,
             "Provider " + cn + " could not be instantiated: " + x,
             x);
    }
    throw new Error();          // This cannot happen
}
```

到这里我们的源码分析之旅就结束了，主要思想其实就是懒加载的思想。

#### 四、总结

通过Java的SPI机制也有一点缺点就是在运行时通过反射加载类实例，这个对性能会有点影响。但是瑕不掩瑜，SPI机制可以实现不同模块之间方便的面向接口编程，拒绝了硬编码的方式，解耦效果很好。用起来也简单，只需要在目录META-INF/services中配置实现类就行。源码中也用来了懒加载的思想，开发中可以借鉴。
例子[https://github.com/juexingzhe/SPIDemo.git](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fjuexingzhe%2FSPIDemo.git)

作者：juexingzhe
链接：https://www.jianshu.com/p/deeb39ccdc53
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。