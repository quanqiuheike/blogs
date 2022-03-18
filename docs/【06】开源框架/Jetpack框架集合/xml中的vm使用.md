#### RecyclerView在XML中布局的使用

* 布局预览：tools:listitem="@layout/user_center_item_star_collection_item"
* layoutManager设置风格：app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
* adapter在xml中绑定设置：app:adapter

```xml
<androidx.recyclerview.widget.RecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            tools:listitem="@layout/user_center_item_star_collection_item"
            app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
/>

```

