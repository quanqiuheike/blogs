### 一、CoordinatorLayout+AppBarLayout

CoordinatorLayout结合AppBarLayout可使得AppBarLayout内部的childView通过设置layout_scrollFlag属性来达到滚动的效果

**AppBarLayout**：通过设置**layout_scrollFlags**参数，来控制**AppBarLayout**中控件的行为

**childView** : 就是**AppBarLayout**内的某个孩子控件;

**ScrollingView** : 就是可以滚动的View;比如RecyclerView，NestedScrollView等等

**layout_scrollFlag**的5种属性|scroll |enterAlways| enterAlwaysCollapsed| exitUntilCollapsed |snap|，一般结合着来用

#### —scroll属性： app:layout_scrollFlags="scroll"—

Child View 伴随着scrollingView的滚动事件而滚出或滚进屏幕:	

##### 使用说明：

* 当ScrollView将要向下滚动的时候，优先滚动的是自己，当自己滚动到顶部头的时候，再开始触发滚动AppBarLayoout中的childView；

* 当ScrollView将要向上滚动的时候， 优先将AppBarLayout的childView滚出屏幕，然后ScrollView才开始滚动；

#### — scroll | enterAlways —

用enterAlways，必须要带上scroll,否则没有效果，同样使用后面哪一个都要有scroll;使用要两个一块使用；

enterAlways决定向下滚动时Scrolling View和Child View之间的滚动优先级问题。

##### 使用说明：

* 当ScrollView将要向下滚动的时候，优先滚动的是AppBarLayout中的childView，当childView完全滚动进入屏幕的时候，才开始滚动 ScrollView；

当ScrollView将要向上滚动的时候， 优先将AppBarLayout的childView滚出屏幕，然后ScrollView才开始滚动；

#### — scroll | enterAlways | enterAlwaysCollapsed —

enterAlwaysCollapsed:它是enterAlways的附加值。要使用它，必须三者一起使用

这里涉及到Child View的高度和最小高度，向下滚动时，Child View先向下滚动最小高度值，然后Scrolling View开始滚动，到达边界时，Child View再向下滚动，直至显示完全。

##### 使用说明：

* 当ScrollView将要向下滚动的时候，优先将AppBarLayout中的childView滚动到它的最小高度，滚动完成之后scrollview才开始自身的滚动，当scrollview滚动完成时，这个时候childView才会将自己高度完全滚动进入屏幕；

当ScrollView将要向上滚动的时候， 优先将AppBarLayout的childView滚出屏幕，然后ScrollView才开始滚动；

##### 这里小结一下:使用上面三种组合，当ScrollView要向上滚动的时候没有任何区别；区别只是在于scrollview要向下滚动的时候

### —scroll | exitUntilCollapsed —

这里也涉及到最小高度。发生向上滚动事件时，Child View向上滚动退出直至最小高度，然后Scrolling View开始滚动。也就是，Child View不会完全退出屏幕。要是exitUntilCollapsed就得这样用才有效果；

##### 使用说明：

* 当ScrollView将要向下滚动的时候，优先滚动的是自己，当自己滚动到顶部头的时候，再开始触发滚动AppBarLayoout中的childView；
  这和单纯使用scroll的效果是一致的；

* 当Scrollview将要向上滚动的时候，优先将AppBarLayout中的childView滚动至最小高度，然后scrollview才开始滚动

#### —scroll | snap —

snap简单理解，就是Child View滚动比例的一个吸附效果。

也就是说，Child View不会存在局部显示,之滚动Child View的部分的情况；当我们松开手指时，Child View要么向上全部滚出屏幕，要么向下全部滚进屏幕，有点类似ViewPager的左右滑动。snap 可以和上面任意一个组合使用，使用它可以确保childView不会滑动停止在中间的状态，当我们松开手指的时候，要么完全显示，要么完全隐藏;

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <!-- Your scrollable view 只支持实现了NestedScrollingChild的View-->
        <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">

            <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="200dp"
                android:minHeight="56dp"
                app:buttonGravity="bottom"
                app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed|snap"
                app:navigationIcon="@mipmap/ic_action_back">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_weight="1"
                        android:background="@android:color/holo_green_dark"
                        android:gravity="center"
                        android:text="@string/app_name"
                        android:textColor="@android:color/white"
                        android:textSize="30sp" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="56dp"
                        android:background="@android:color/holo_red_dark"
                        android:gravity="center"
                        android:text="最小高度"
                        android:textColor="@android:color/white"
                        android:textSize="20sp" />


                </LinearLayout>

            </androidx.appcompat.widget.Toolbar>

        </com.google.android.material.appbar.AppBarLayout>

        <androidx.core.widget.NestedScrollView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center_horizontal"
                android:lineSpacingExtra="2dp"
                android:lineSpacingMultiplier="1"
                android:padding="16dp"
                android:textAppearance="@style/TextAppearance.AppCompat.Inverse"
                android:textSize="18sp" />

        </androidx.core.widget.NestedScrollView>

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>

```



```kotlin
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
            name="vm"
            type="com.weifeng.hobby.homepage.found.subs.party.PartyViewModel" />

        <import type="android.view.View" />
    </data>

    <androidx.coordinatorlayout.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <com.google.android.material.appbar.AppBarLayout
            android:id="@+id/appbarLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                app:layout_scrollFlags="scroll|enterAlwaysCollapsed">

                <androidx.recyclerview.widget.RecyclerView
                    android:id="@+id/topRecyclerView"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:clipToPadding="false" />

                <include
                    android:id="@+id/tipsRoot"
                    layout="@layout/homepage_party_tip_provider" />

                <include
                    android:id="@+id/reviewRoot"
                    layout="@layout/homepage_party_review_provider" />

            </LinearLayout>

            <com.weifeng.hobby.commonui.views.tag.CommonTag
                android:id="@+id/com_tag"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginLeft="@dimen/commonMargin"
                android:layout_marginTop="@dimen/ui_dp10"
                android:layout_marginRight="@dimen/commonMargin"
                android:layout_marginBottom="@dimen/ui_dp10" />
        </com.google.android.material.appbar.AppBarLayout>

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior">

            <androidx.recyclerview.widget.RecyclerView
                android:id="@+id/hotNewRecyclerView"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:clipToPadding="false"
                android:paddingBottom="@dimen/ui_dp70" />

            <com.basestonedata.lego.ui.widget.loadinglayout.UILoadingLayout
                android:id="@+id/loading_Layout"
                android:layout_width="match_parent"
                android:layout_height="match_parent" />

        </FrameLayout>

    </androidx.coordinatorlayout.widget.CoordinatorLayout>

</layout>
```

### 二、Toolbar 的基本用法

##### ToolBar在XML中的用法

```xml
 <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorAccent"
                android:minHeight="56dp"
                app:buttonGravity="bottom"
                app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed|snap"
                app:navigationIcon="@mipmap/ic_action_back"
                app:title="标题"
                app:titleTextColor="@color/white">
   
   
      <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorAccent"
                app:logo="@mipmap/ic_launcher"
                app:navigationContentDescription=""
                app:navigationIcon="@drawable/ic_back_white_24dp"
                app:subtitle="子标题"
                app:subtitleTextColor="@color/white"
                app:title="标题"
                app:titleMarginStart="90dp"
                app:titleTextColor="@color/white">
```

代码中的写法

```java
 Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        toolbar.setTitle("标题");
        toolbar.setTitleTextColor(Color.WHITE);
        toolbar.setNavigationIcon(R.drawable.ic_action_back);

        //点击左边返回按钮监听事件
        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

            }
        });

```



**注意事项：**
 1：如果你添加了这行代码 setSupportActionBar(toolbar); 那么 toolbar.setNavigationOnClickListener监听方法，要放到其后面，否则点击事件，监听不到的。如果你用不到ActionBar的一些特性的话，建议setSupportActionBar(toolbar); 这行代码不用添加了。

如果你想修改主标题和子标题的文字大小，你可通过如下方式：
 首先定义一个style：

```xml
 <!--主标题-->
<style name="ToolbarTitle" parent="@style/TextAppearance.Widget.AppCompat.Toolbar.Title">
       <item name="android:textColor">#f0f0</item>
        <item name="android:textSize">15sp</item>
    </style>

 <!--子标题-->
<style name="ToolbarSubtitle" parent="@style/TextAppearance.Widget.AppCompat.Toolbar.Subtitle">
        <item name="android:textColor">#f0f0</item>
        <item name="android:textSize">10sp</item>
    </style>



 <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorAccent"
                app:logo="@mipmap/ic_launcher"
                app:navigationContentDescription=""
                app:navigationIcon="@drawable/ic_back_white_24dp"
                app:subtitle="子标题"
                app:subtitleTextColor="@color/white"
                app:title="标题"
                app:titleMarginStart="90dp"
                app:titleTextAppearance="@style/ToolbarTitle"
                app:titleTextColor="@color/white">
```

通过app:titleTextAppearance=”@style/ToolbarTitle”方法的设置，就能修改标题字体的大小，当然文字颜色也可以修改。

如果，我想要标题居中，怎么办呢？查看api，toolbar没有使其居中的方法，也就提供了使其距左右，上下边距大小的方法。不过不用担心，这里还是有办法的。看如下代码：

```xml
 <androidx.appcompat.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                android:background="@color/colorAccent">

                <TextView
                    android:id="@+id/title"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_gravity="center"
                    android:text="居中标题"
                    android:textColor="@color/white"
                    android:textSize="22sp" />

            </androidx.appcompat.widget.Toolbar>
```

**注意：** 此时 TextView 控件的宽和高都是自适应大小，java 代码中此行代码`setSupportActionBar(toolbar);`就不要添加了，否则就会显示不正常。如果你非要添加`setSupportActionBar(toolbar);`这行代码的话，TextView 控件的宽要用match_parent属性。这里再次建议`setSupportActionBar(toolbar);`这行代码就不要点添加了。
 至于它的作用，在此做一下简单的说明吧：
 1）在Toolbar这个控件出现之前，其实我们也可以通过 `ActionBar actionBar = getSupportActionBar();` 方法获取到acitonbar，（前提你的activity主题theme，是采用的带actionbar的主题，如果你采用这样的主题`android:theme="@style/Theme.AppCompat.Light.NoActionBar">`拿到的actionBar也是 null,显然是不行的）之后你就可以采用诸如下面的方面来操作actionbar啦。





#### CollapsingToolbarLayout







###### Refrence&Thanks

[AppBarLayout中的五种ScrollFlags使用方式汇总](https://blog.csdn.net/eyishion/article/details/80282204)

[Android Toolbar的详细介绍和自定义Toolbar](https://www.jianshu.com/p/103fa0c7e7f3)