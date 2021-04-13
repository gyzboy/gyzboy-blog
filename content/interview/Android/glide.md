---
title: "Glide"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}
## Q:Glide优势?
1. 多种图片格式的缓存，适用于更多的内容表现形式（如Gif、WebP、缩略图、Video）
2. 生命周期集成（根据Activity或者Fragment的生命周期管理图片加载请求）
3. 高效处理Bitmap（bitmap的复用和主动回收，减少系统回收压力）
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
