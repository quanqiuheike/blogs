### 1、路由跳转Activity中的使用

```kotlin
 /**
  * 路由常量
  */
 const val PARTY_DETAIL = "/party/detail"

 //Router跳转到Activity
 ARouter.getInstance().build(PartyScheme.PARTY_DETAIL)
                .withString("partyId", item.partyId)
        			  .withString("id", item.id)
                .withString("index", index).navigation()

 //Activity注册路由
 @Route(path = PartyScheme.PARTY_DETAIL)
 class PartyDetailActivity : BaseActivity<PartyActivityDetailBinding>() {
    @JvmField  
    @Autowired
    var index = ""
 }


```

#### 2、路由跳转到Fragment中的使用路由跳转

```kotlin
 /**
  * 路由path
  */
const val PARTY_DETAIL_FRAGMENT = "/party/detailfragment"

//跳转方式
val fragment = ARouter.getInstance().build(PartyScheme.PARTY_DETAIL_FRAGMENT)
 .with(intent?.extras)
 .navigation() as Fragment
 replaceFragment(R.id.content_container, fragment)

//Fragment中Route注册path
@Route(path = PartyScheme.PARTY_DETAIL_FRAGMENT)
class PartyDetailFragment : BaseFragment<PartyFragmentDetailBinding>(), ScreenAutoTracker {
  
 		//参数接收
    @JvmField
    @Autowired(name="id")
    var mId = ""//接收真正的参数为id，赋值给mID
}

```

##### xml

```kotlin
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="ResourceName">

    <data>

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/content_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</layout>
```



#### 3、参数传递接收

```kotlin
@Route(path = PartyScheme.PARTY_DETAIL)
class PartyDetailActivity : BaseActivity<PartyActivityDetailBinding>() {
    @JvmField  
    @Autowired
    var index = ""
    
    //参数接收
    @JvmField
    @Autowired(name="id")
    var mId = ""//接收真正的参数为id，赋值给mID
}
```



