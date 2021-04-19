---
title: "Glide"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}
## Q:Glide优势?
1. 多种图片格式的缓存，适用于更多的内容表现形式（如Gif、WebP、缩略图、Video）
2. 生命周期集成（根据Activity或者Fragment的生命周期管理图片加载请求）
    >- 通过为每个activity或者Fragment创建一个无界面的SupportRequestManagerFragment来进行生命周期的绑定
    >- 在创建SupportRequestManagerFragment时会讲一个当前对象存储在map中,这是因为事务的提交commitAllowingStateLoss是post到队列中,防止重新创建fragment
    >- 在创建RequestManager时获取SupportRequestManagerFragment的Lifecycle对象
3. 高效处理Bitmap（bitmap的复用和主动回收，减少系统回收压力）
    >- decodeFromWrappedStreams中会调用BitmapPool复用池,通过inBitmap获取复用图片
4. 高效的缓存策略，灵活（Picasso只会缓存原始尺寸的图片，Glide缓存的是多种规格），加载速度快且内存开销小（默认Bitmap格式的不同，使得内存开销是Picasso的一半）

## Q:自己实现一个图片加载框架,需要考虑什么?
* 异步加载：线程池
* 切换线程：Handler，没有争议吧
* 缓存：LruCache、DiskLruCache
* 防止OOM：软引用、LruCache、图片压缩、Bitmap像素存储位置
* 内存泄露：注意ImageView的正确引用，生命周期管理
* 列表滑动加载的问题：加载错乱、队满任务过多问题


## Q:Glide中这几个点都是如何实现的?
* 异步加载
```java
  private GlideExecutor sourceExecutor; //加载源文件的线程池，包括网络加载
  private GlideExecutor diskCacheExecutor; //加载硬盘缓存的线程池
  ...
  private GlideExecutor animationExecutor; //动画线程池
```

## Q:Glide是如何加载动图的?
从流里读取前3个字节用于判断是否为GIF图,确认为 GIF 动图后，会构建一个 GIF 的解码器（StandardGifDecoder），它可以从 GIF 动图中读取每一帧的数据并转换成 Bitmap，然后使用 Canvas 将 Bitmap 绘制到 ImageView 上，下一帧则利用 Handler 发送一个延迟消息实现连续播放，所有 Bitmap 绘制完成后又会重新循环，所以就实现了加载 GIF 动图的效果
>- 优化方式
>>- 1、使用GIFLIB+双缓冲的实现,只会创建两个Bitmap,并且内存消耗非常之稳定
>>- 2、相比Glide的原生加载,当加载过大的GIF图时,超过了BitmapPool的可用大小,还是会直接创建Bitmap的.
>>- 3、使用GIFLIB是直接在native层对GIF数据进行解码的,这一点对Glide来说,效率和内存消耗情况都比较占优.
>>- 4、Glide 构建当前帧数据和下一帧数据是串行的,而 FrameSequenceDrawable 则是利用了双缓冲以及解码子线程来实现近似同步的完成上一帧和下一帧数据的无缝衔接的.

## Q:Glide中的缓存设计?
主要有四级缓存:
1. 活动资源 (Active Resources)：正在使用的图片 采用一个存储了弱引用的HashMap来缓存活动资源
  >- 当我们从一屏滑动到下一屏的时候，上一屏的图片就会看不到，这个时候就从活动资源转为内存资源
  >- 我们关闭当前显示图片的页面时,会从活动资源转为内存资源
  >- 优势:
    1. 提高访问效率,HashMap访问比LinkedHashMap快
    2. 防止内存泄漏
2. 内存缓存 (Memory cache)：内存缓存中的图片 LruCache
3. 资源类型（Resource）：磁盘缓存中转换过后的图片
4. 数据来源 (Data)：磁盘缓存中的原始图片

## Q:如何解决Glide缓存变化但图片没有更新问题?
1. 图片 url 不要固定,也就是说如果某个图片改变了，那么该图片的 url 也要跟着改变。
2. 使用 signature() 更改缓存 Key
3. 禁用缓存