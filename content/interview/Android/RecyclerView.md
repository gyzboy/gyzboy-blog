---
title: "RecyclerView"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}
## Q:RecyclerView中onLayout有哪些步骤流程?
```java
RecyclerView.onLayout(...)
//布局放置
->RecyclerView.dispatchLayout()
  //dispatchLayoutStep1方法等同于pre layout,预布局阶段,记录ViewHolder位置信息；;
  ->RecyclerView.dispatchLayoutStep1()
  ///dispatchLayoutStep2方法处理真正布局的地方
  ->RecyclerView.dispatchLayoutStep2()
  //开启动画阶段
  ->RecyclerView.dispatchLayoutStep3()

->RecyclerView.dispatchLayoutStepX()
  //mLayout是LayoutManager的实例\
  ->mLayout.onLayoutChildren(mRecycler, mState);\
  //此处查看LinearLayoutManager.fill() 注释：填充给定Layout\
  ->LinearLayoutManager.fill(recycler, mLayoutState, state, false);\
  //循环调用，每次返回一个\
  ->LinearLayoutManager.layoutChunk(recycler, layoutState) \
  ->LinearLayoutManager.LayoutState.next()\
  //通过 Recycler 获取指定位置的 ItemView\
  ->RecyclerView.recycler.getViewForPosition(int position)\
  //获取ViewHolder 返回ViewHolder中的ItemView,在这里决定是onCreateView还是bindView\
  ->RecyclerView.tryGetViewHolderForPositionByDeadline(***)
```

## Q:RecyclerView中是如何获取ViewHolder的？有哪些步骤流程?
![image](/viewHolder.png)
1. 如果是预布局，尝试从mChangedScrap 中获取ViewHolder
2. 尝试从mAttachedScrap/mHiddenViews/mCachedViews 中获取ViewHolder
3. 如果存在StableId 尝试使用ID从mAttachedScrap中获取ViewHolder
4. 如果用户有自定义缓存，尝试从mViewCacheExtension中获取ViewHolder，一般不会自定义
5. 尝试从mRecyclerPool中获取ViewHolder
6. 如果以上方法均未获取到则创建一个ViewHolder

## Q:什么是pre-layout?什么时候调用?
当adapter调用notifyItemChanged()或者notifyItemRangeChanged()的时候，onLayoutChildren()会调用两次，一次是预布局，一次是实际布局。通过对比两次布局的不同，RecyclerView可以完成预测动画。

## Q:stableID是什么？有什么作用?
stableID 作用在于调用notifyDataSetChanged方法后，LayoutManager重新布局的的时候将ViewHolder回收到何处
没有设置StableId，viewHolder被回收到RecyclerViewPool 如果设置了StableId，viewHolder被回收到Scrap中

## Q:什么是Scrap？设计思想是什么?
mChangedScrap 和 mAttachedScrap 是RecyclerView最先查找ViewHolder的地方， 只在布局阶段使用，布局完成后这两个地方的ViewHolder会移到mCachedViews 或者mRecyclerPool中。

## Q:mChangedScrap 和 mAttachedScrap区别?
1. 添加时机不同，只有在Item发生了变化（notifyItemChanged或者notifyItemRangeChanged被调用），并且ItemAnimator调用canReuseUpdatedViewHolder()返回false时才会添加到mAttachedScrap，否则添加到mChangedScrap中
CanReuseUpdateViewHolder返回‘false’表示要使用不同的ViewHolder来完成动画，true表示使用相同的ViewHolder完成动画例如淡入淡出
2. mChangedScrap 只在预布局的时候会使用到，mAttachedScrap在整个布局中都可以使用

## Q:如何实现RecyclerView的拖拽排序?
ItemTouchHelper

## Q:当item超过一定数量，如何正确的设置RecyclerView的maxHeight？
重写Layoutmanager的isAutoMeasureEnabled,当超过maxHeight时自己进行measure

## Q:RecyclerView中的复用的几个问题
1. 复用什么？  在RV中,复用的是ViewHolder

2. 从哪里获得复用？优先级如何? Recycler有4个层次用于缓存 ViewHolder 对象，优先级从高到底依次为mAttachedScrap、mCachedViews、mViewCacheExtension、mRecyclerPool。如果四层缓存都未命中，则重新创建并绑定 ViewHolder 对象,RecycledViewPool 对 ViewHolder 按viewType分类存储（通过SparseArray），同类 ViewHolder 存储在默认大小为5的ArrayList中

3. 什么时候复用？从mRecyclerPool中复用的ViewHolder ，只能复用于viewType相同的表项，从mCachedViews中复用的 ViewHolder ，只能复用于指定位置的表项。mCachedViews用于缓存指定位置的 ViewHolder,会将viewHolder放入mRecyclerPool中 ，只有“列表回滚”这一种场景（刚滚出屏幕的表项再次进入屏幕），才有可能命中该缓存。该缓存存放在默认大小为 2 的ArrayList中

## Q:RecyclerView回收些什么?
回收那些子view顶部距离limit线超过view高度的view
Limit线可以理解为列表滑动停止后,列表顶部的位置

## Q:RecyclerView中的ViewHolder回收到哪里去?
滑出屏幕表项对应的 ViewHolder 会被回收到mCachedViews+mRecyclerPool 结构中
mCachedViews是 ArrayList ，默认存储最多2个 ViewHolder ，当它放不下的时候，按照先进先出原则将最先进入的 ViewHolder 存入回收池的方式来腾出空间。mRecyclerPool 是 SparseArray ，它会按viewType分类存储 ViewHolder ，默认每种类型最多存5个。

* 从mAttachedScrap 中复用的 ViewHolder 不需要重新创建也不需要重新绑定数据
* 从mCachedViews中复用的 ViewHolder 不需要重新绑定数据,不满足holder.isBound
* 从mRecyclerPool中复用的 ViewHolder 需要重新绑定数据

## Q:什么情况下,会调用bindViewHolder?
满足!holder.isBound() || holder.needsUpdate() || holder.isInvalid()

## Q:RecyclerView卡片中持有的资源，到底该什么时候释放？
在onViewRecycled中进行释放,因为这是标明item已经进入mRecyclerPool了，重新进入时会bind数据

## Q:RecyclerView中各级缓存的用途?各存储结构是什么样的?
* mAttachedScrap：用于布局过程中屏幕可见表项的回收和复用。没有大小限制，但最多包含屏幕可见表项,ArrayList<ViewHolder>
* mCachedViews：用于移出屏幕表项的回收和复用，且只能用于指定位置的表项，有点像“回收池预备队列”，即总是先回收到mCachedViews，当它放不下的时候，按照先进先出原则将最先进入的ViewHolder存入回收池。ArrayList<ViewHolder>
* mRecyclerPool：用于移出屏幕表项的回收和复用，且只能用于指定viewType的表项,对ViewHolder按viewType分类存储在SparseArray<ScrapData>中，同类ViewHolder存储在ScrapData中的ArrayList中

## Q:如何设计一个RecyclerView的item点击事件?
1. 层层传递点击监听回调,在viewHolder中绑定onClick事件
2. 将点击坐标转化成表项索引
通过GestureDetector监听onSingleTapUp事件,然后仿照AbsListView.pointToPosition()中的做法,将触摸点坐标通过getHitRect转换成点击位置position

## Q:RecyclerView的预加载是什么?
根据recyclerView的滑动方向,提前加载item内容

## Q:RecyclerView为什么进行预布局?
为了进行表项动画的展示
1. RecyclerView 为了实现表项动画，进行了 2 次布局，第一次预布局，第二次后布局，在源码上表现为 LayoutManager.onLayoutChildren() 被调用 2 次。

2. 预布局的过程始于 RecyclerView.dispatchLayoutStep1()，终于 RecyclerView.dispatchLayoutStep2()。

## Q:如何实现RecyclerView的局部更新，用过payload吗,notifyItemChange方法中的参数
payload参数可以认为是你要刷新的一个标示，比如我有时候只想刷新itemView中的textview,有时候只想刷新imageview？又或者我只想某一个view的文字颜色进行高亮设置？那么我就可以通过payload参数来标示这个特殊的需求了







