---
title: "OkHttp"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}

## Q:OkHttp中用到了哪些设计模式
> 外观模式。通过okHttpClient这个外观去实现内部各种功能。
> 建造者模式。构建不同的Request对象。
> 工厂模式。通过OkHttpClient生产出产品RealCall。
> 享元模式。通过线程池、连接池共享对象。
> 责任链模式。将不同功能的拦截器形成一个链。