---
title: "LeakCanary"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}

## Q:Java中各种引用类型?
### 强引用
>- 强引用是使用最普遍的引用。一个对象具有强引用，则在GC发生时，该对象将不会回收。当Jvm虚拟机内存空间不足时，虚拟机会抛出OutOfMemoryError错误，不会回收具有强引用的对象来解决内存不足的问题
### 软引用
>- 当一个对象只有软引用，若虚拟机内存空间足够，垃圾回收器就不会回收该对象；
若内存空间不足，下次GC时这些只有软引用对象将被回收。若垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。
在创建软引用实例时，可以传入一个引用队列（ReferenceQueue）将该软引用与引用队列关联，这样当软引用所引用的对象被垃圾回收器回收前，Jvm虚拟机会把这个软引用加入到与之关联的引用队列中
### 弱引用
>- 弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在GC发生时，若一个对象只有弱引用，不管虚拟机内存空间是否足够，都会回收它的内存。
在创建弱引用实例时，可以传入一个引用队列（ReferenceQueue）将该软引用与引用队列关联，这样当软引用所引用的对象被垃圾回收器回收前，Jvm虚拟机就会把这个软引用加入到与之关联的引用队列中
### 虚引用
>- 虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
在创建虚引用实例时，可以传入一个引用队列（ReferenceQueue）将该软引用与引用队列关联，这样当软引用所引用的对象被垃圾回收器回收前，Jvm虚拟机就会把这个软引用加入到与之关联的引用队列中
```java
String str = new String("abc");
ReferenceQueue queue = new ReferenceQueue();
// 创建虚引用，要求必须与一个引用队列关联
PhantomReference pr = new PhantomReference(str, queue);
```

## 如何通过引用队列判断是否发生内存泄漏?
软引用、弱引用、虚引用的构造方法均可以传入一个ReferenceQueue与之关联。在引用所指的对象被回收后，引用（reference)本身将会被加入到ReferenceQueue之中，此时引用所引用的对象reference.get()已被回收 (reference此时不为null，reference.get()此时为null)。
所以，在一个非强引用所引用的对象回收时，如果引用reference没有被加入到被关联的ReferenceQueue中，则表示还有引用所引用的对象还没有被回收。如果判断一个对象的非强引用本该出现在ReferenceQueue中，实际上却没有出现，则表示该对象发送内存泄漏

## LeakCanary原理?
* 在onDestory发生时创建一个弱引用指R向Activity，并关联一个RefrenceQuence,当Activity被正常回收，弱引用实例本身应该出现在该RefrenceQuence中，否则便可以判断该Activity存在内存泄漏。
* 通过Application.registerActivityLifecycleCallbacks()方法可以注册Activity生命周期的监听，每当一个Activity调用onDestroy进行页面销毁时，去获取到这个Activity的弱引用并关联一个ReferenceQuence，通过检测ReferenceQuence中是否存在该弱引用判断这个Activity对象是否正常回收。
* 当onDestory被调用后，初步观察到Activity未被GC正常回收时，手动触发一次GC，由于手动发起GC请求后并不会立即执行垃圾回收，所以需要在一定时延后再二次确认Activity是否已经回收，如果再次判断Activity对象未被回收，则表示Activity存在内存泄漏
* 当Activity的onDestory方法被调用后，LeakCanary将在RefWatcher的retainedKeys(CopyOnWriteArraySet)加入一条全局唯一的UUID，同时创建一个该Activityd的弱引用对象KeyedWeakReference，并将UUID写入KeyedWeakReference实例中，同时KeyedWeakReference与引用队列queue进行关联，这样当Activity对象正常回收时，该弱引用对象将进入队列当中。循环遍历获取queue队列中的KeyedWeakReference对象ref，将ref中的UUID取出，在retainedKeys中移除该UUID。如果遍历完成后retainedKeys中仍然存在该弱引用的UUID,则说明该Activity对象在onDestory调用后没有被正常回收。此时通过GcTrigger手动发起一次GC,再等待100ms，然后再次判断Activity是否被正常回收
