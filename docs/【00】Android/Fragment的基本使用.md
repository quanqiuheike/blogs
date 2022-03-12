#### fragment的使用方法

```java
FragmentManager fm = getActivity().getSupportFragmentManager();
FragmentTransaction ft = fm.beginTransaction();
if(!sub1Fragment.isAdded()){
  ft.add(R.id.rl_fragment_container, sub1Fragment).commit();
}
```

#### 嵌套fragment的使用方法

```java
FragmentManager fm = getChildFragmentManager(); 
FragmentTransaction ft = fm.beginTransaction(); 
if(!sub1Fragment.isAdded()){ 
　　ft.add(R.id.rl_fragment_container, sub1Fragment).commit(); 
}
```

#### Kotlin中的使用

```kotlin
// fragment嵌套fragment的使用方法
var fragmentTransaction =childFragmentManager.beginTransaction();
//Activity中fragment的使用方法
var fragmentTransaction =activity?.supportFragmentManager?.beginTransaction();
    ragmentTransaction?.replace(
        R.id.top_contains,
        starsManagerTopFragment );
    fragmentTransaction?.commit();
```





