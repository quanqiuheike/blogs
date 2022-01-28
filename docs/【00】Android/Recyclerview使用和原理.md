

#### RecyclerView预取机制 

##### 1）首先说下RecyclerView的缓存结构： 

`Recyclerview`有四级缓存，分别是` mAttachedScrap(屏幕内)，mCacheViews(屏幕外)， mViewCacheExtension(自定义缓存)，mRecyclerPool(缓存池) `

* **mAttachedScrap(屏幕内)** ，用于屏幕内`itemview`快速重用，不需要重新`createView`和`bindView `。

* **mCacheViews(屏幕外)** ，保存最近移出屏幕的`ViewHolder`，包含数据和`position`信息，复用时必须是相同位置的`ViewHolder`才能复用，应用场景在那些需要来回滑动的列表中，当往回滑动时，能直接复用`ViewHolder`数据，不需要重新`bindView`。 

* **mViewCacheExtension(自定义缓存)** ，不直接使用，需要用户自定义实现，默认不实现。 

* **mRecyclerPool(缓存池)** ，当`cacheView`满了后或者`adapter`被更换，将`cacheView`中移出的`ViewHolder`放到`mRecyclerPool`中，放之前会把`ViewHolder`数据清除掉，所以复用时需要重新`bindView`。 

##### 2）四级缓存按照顺序需要依次读取。所以**完整缓存流程**是： 

1. 保存缓存流程： 

* 插入或是删除` itemView `时，先把屏幕内的`ViewHolder`保存至 `AttachedScrap `中。
* 滑动屏幕的时候，先消失的`itemview`会保存到 `CacheView `，`CacheView`大小默认是2，超过数量的话按照先入先出原则，移出头部的`itemview`保存到 `RecyclerPool`缓存池 （如果有自定义缓存就会保存到自定义缓存里），`RecyclerPool`缓存池会按照`itemview`的 `itemtype `进行保存，每个`itemType`缓存个数为`5`个，超过就会被回收。 

2. 获取缓存流程： 

* `AttachedScrap`中获取，通过`pos`匹配`holder`——>获取失败，从 `CacheView `中获取，也是通过`pos`获取`holder`缓存 

  ——>获取失败，从 自定义缓存 中获取缓存——>获取失败，从 `mRecyclerPool` 中获取 

  ——>获取失败，重新创建` viewholder ——createViewHolder并bindview。 `

##### 3）了解了缓存结构和缓存流程，我们再来看看具体的问题 

滑动10个，再滑回去，会有几个执行`onBindView`？ 

* 由之前的缓存结构可知，需要重新执行` onBindView `的只有一种缓存区，就是缓存池`mRecyclerPool `。

所以我们假设从加载` RecyclView `开始盘的话（页面假设可以容纳7条数据）：

* 首先，7条数据会依次调用`onCreateViewHolder `和`onBindViewHolder `。 

* 往下滑一条（`position=7`），那么会把`position=0`的数据放到` mCacheViews` 中。此时`mCacheViews `缓存区数量为1，` mRecyclerPool `数量为0。然后新出现的`position=7`的数据通过`postion`在` mCacheViews `中找不到对应的` ViewHolder `，通过` itemtype` 也在 `mRecyclerPool `中找不到对应的数据，所以会调用 `onCreateViewHolder `和` onBindViewHolder` 方法。 

* 再往下滑一条数据（`position=8`），如上。 

* 再往下滑一条数据（`position=9`），`position=2`的数据会放到 `mCacheViews `中，但是由于 `mCacheViews` 缓存区默认容量为2，所以`position=0`的数据会被清空数据然后放到 `mRecyclerPool`缓存池中。而新出现的`position=9`数据由于在 `mRecyclerPool `中还是找不到相应 `type`的`ViewHolder`，所以还是会走`onCreateViewHolder `和 `onBindViewHolder `方法。所以此时`mCacheViews `缓存区数量为2， `mRecyclerPool `数量为1。 

* 再往下滑一条数据（`position=10`），这时候由于可以在 `mRecyclerPool `中找到相同`viewtype`的 

`ViewHolder`了。所以就直接复用了，并调用 `onBindViewHolder `方法绑定数据。 

* 后面依次类推，刚消失的两条数据会被放到 `mCacheViews `中，再出现的时候是不会调用 

`onBindViewHolder`方法，而复用的第三条数据是从 `mRecyclerPool `中取得，就会调用 

`onBindViewHolder `方法了。 

##### 4）所以这个问题就得出结论了（假设 mCacheViews 容量为默认值2）： 

* 如果一开始滑动的是新数据，那么滑动10个，就会走10个` bindview `方法。然后滑回去，会走10-2个 `bindview `方法。一共18次调用。 

* 如果一开始滑动的是老数据，那么滑动10-2个，就会走8个 `bindview `方法。然后滑回去，会走10-2个 `bindview `方法。一共16次调用。 

**但是但是**，实际情况又有点不一样。因为 `Recyclerview`在v25版本引入了一个新的机制， 预取机制 。 

预取机制 ，就是在滑动过程中，会把将要展示的一个元素提前缓存到` mCachedViews` 中，所以滑动10个 

元素的时候，第11个元素也会被创建，也就多走了一次 `bindview` 方法。但是滑回去的时候不影响，因 

为就算提前取了一个缓存数据，只是把 `bindview` 方法提前了，并不影响总的绑定`item`数量。 

所以滑动的是新数据的情况下就会多一次调用` bindview` 方法。 

##### 5）总结，问题怎么答呢？ 

* 四级缓存和流程说一下。 

* 滑动10个，再滑回去， `bindview `可以是19次调用，可以是16次调用。 
* 缓存的其实就是缓存`item`的`view`，在`Recyclerview`中就是` viewholder `。
*  `cachedView `就是 `mCacheViews` 缓存区中的`view`，是不需要重新绑定数据的。 

#### 如何实现`RecyclerView`的局部更新，用过`payload`吗,`notifyItemChange`方法中的参数？

##### 关于`RecyclerView`的数据更新，主要有以下几个方法： 

* `notifyDataSetChanged() `，刷新全部可见的item。 

* `notifyItemChanged(int) `，刷新指定item。 

* `notifyItemRangeChanged(int,int) `，从指定位置开始刷新指定个item。 

* `notifyItemInserted(int)、notifyItemMoved(int)、notifyItemRemoved(int) `。插入、移 

动一个并自动刷新。 

* `notifyItemChanged(int, Object) `，局部刷新。 

可以看到，关于view的局部刷新就是`notifyItemChanged(int, Object)`方法，下面具体说说： 

`notifyItemChange` 有两个构造方法：

* `notifyItemChanged(int position, @Nullable Object payload) `

* `notifyItemChanged(int position) `

其中 `payload `参数可以认为是你要刷新的一个标示，比如我有时候只想刷新 `itemView `中的` textview` , 有时候只想刷新` imageview` ？又或者我只想某一个`view`的文字颜色进行高亮设置？那么我就可以通过 `payload` 参数来标示这个特殊的需求了。 

具体怎么做呢？比如我调用了` notifyItemChanged（14,"changeColor"）` ,那么在 `onBindViewHolder`回调方法中做下判断即可： 

```java

```



**RecyclerView****嵌套****RecyclerView****滑动冲突，** 

**NestScrollView****嵌套****RecyclerView****。** 

1） RecyclerView 嵌套 RecyclerView 的情况下，如果两者都要上下滑动，那么就会引起滑动冲突。默 

认情况下外层的RecyclerView可滑，内层不可滑。 

之前说过解决滑动冲突的办法有两种：**内部拦截法和外部拦截法**。 

这里我提供一种内部拦截法，还有一些其他的办法大家可以自己思考下。 

2）关于 ScrclerView 的滑动冲突还是同样的解决办法，就是进行事件拦截。 

还有一个办法就是用 Nestedscrollview 代替 ScrollView ， Nestedscrollview 是官方为了解决滑动 

冲突问题而设计的新的View。它的定义就是支持嵌套滑动的ScrollView。 

所以直接替换成 Nestedscrollview 就能保证两者都能正常滑动了。但是要注意设置 

RecyclerView.setNestedScrollingEnabled(false) 这个方法，用来取消RecyclerView本身的滑动 

效果。 

@Override 

public void onBindViewHolder(ViewHolderholder, int position, List<Object> 

payloads) { 

if (payloads.isEmpty()) { 

// payloads为空，说明是更新整个ViewHolder 

onBindViewHolder(holder, position); 

} else { 

// payloads不为空，这只更新需要更新的View即可。 

String payload = payloads.get(0).toString(); 

if ("changeColor".equals(payload)) { 

holder.textView.setTextColor(""); 

} 

} 

}

holder.recyclerView.setOnTouchListener { v, event -> 

when(event.action){ 

//当按下操作的时候，就通知父view不要拦截，拿起操作就设置可以拦截，正常走父 

view的滑动。 

MotionEvent.ACTION_DOWN,MotionEvent.ACTION_MOVE -> 

v.parent.requestDisallowInterceptTouchEvent(true) 

MotionEvent.ACTION_UP -> 

v.parent.requestDisallowInterceptTouchEvent(false) 

}

false}这是因为RecyclerView默认是 setNestedScrollingEnabled(true) ，这个方法的含义是支持嵌套滚动 

的。也就是说当它嵌套在 NestedScrollView 中时,默认会随着 NestedScrollView 滚动而滚动,放弃了 

自己的滚动。所以给我们的感觉就是滞留、卡顿。所以我们将它设置为false就解决了卡顿问题，让他正 

常的滑动，不受外部影响。 

**说说****RecyclerView****性能优化。** 

bindViewHolder 方法是在UI线程进行的，此方法不能耗时操作，不然将会影响滑动流畅性。比如 

进行日期的格式化。 

对于新增或删除的时候，可以使用 diffutil 进行局部刷新，少用全局刷新 

对于 itemVIew 进行布局优化，比如少嵌套等。 

25.1.0 (>=21)及以上使用 Prefetch 功能，也就是预取功能，嵌套时且使用的是 

LinearLayoutManager，子RecyclerView可通过setInitialPrefatchItemCount设置预取个数 

加大 RecyclerView缓存 ，比如cacheview大小默认为2，可以设置大点，用空间来换取时间，提高 

流畅度 

如果高度固定，可以设置 setHasFixedSize(true) 来避免requestLayout浪费资源，否则每次更 

新数据都会重新测量高度。 

如果多个 RecycledView 的 Adapter 是一样的，比如嵌套的 RecyclerView 中存在一样的 

Adapter，可以通过设置 RecyclerView.setRecycledViewPool(pool); 来共用一个 

RecycledViewPool 。这样就减少了创建VIewholder的开销。 

在RecyclerView的元素比较高，一屏只能显示一个元素的时候，第一次滑动到第二个元素会卡顿。 

这种情况就可以通过设置额外的缓存空间，重写 getExtraLayoutSpace 方法即可。 

设置 RecyclerView.addOnScrollListener(); 来在滑动过程中停止加载的操作。 

减少对象的创建，比如设置监听事件，可以全局创建一个，所有view公用一个listener，并且放到 

CreateView 里面去创建监听，因为CreateView调用要少于bindview。这样就减少了对象创建所 

造成的消耗 

用 notifyDataSetChange 时，适配器不知道整个数据集中的那些内容以及存在，再重新匹配 

ViewHolder 时会花生闪烁。设置adapter.setHasStableIds(true)，并重写 getItemId() 来给每个 

Item一个唯一的ID，也就是唯一标识，就使itemview的焦点固定，解决了闪烁问题。 

void onItemsInsertedOrRemoved() { 

if (hasFixedSize) layoutChildren(); 

else requestLayout(); 

}

new LinearLayoutManager(this) { 

@Override 

protected int getExtraLayoutSpace(RecyclerView.State state) { 

return size; 

} 

};



讲一下 RecyclerView 的缓存机制,滑动10个，再滑回去，会有几个执行 onBindView 。缓存 

的是什么？ cachedView 会执行onBindView吗? 

##### RecyclerView 预取机制 

##### 如何实现 RecyclerView 的局部更新，用过 payload 吗,notifyItemChange方法中的参数？ 

##### RecyclerView 嵌套 RecyclerView 滑动冲突，NestScrollView嵌套RecyclerView。 

##### 说说 RecyclerView 性能优化。 

##### 讲一下RecyclerView的缓存机制滑动10个，再滑回去，会有几个执行 

#### onBindView。缓存的是什么？cachedView会执行onBindView吗? 