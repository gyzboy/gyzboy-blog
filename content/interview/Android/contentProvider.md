---
title: "ContentProvider"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}

## Q:介绍下FileProvider？
```java
<!-- 配置FileProvider-->

<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.provider"
    android:exported="false"//这里必须为false
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/provider_paths"/>
</provider>

//provider_paths.xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="external" path="."/>
</paths>
```