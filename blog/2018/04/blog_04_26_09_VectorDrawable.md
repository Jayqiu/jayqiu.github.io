#  Android 5.0  VectorDrawable(矢量图)

在 Android 5.0（API Level 21）中，Android 开始支持矢量图 VectorDrawable, VectorDrawable 的特点就是它不会因为图像的缩放而失真。这样在 Android 开发过程中你不需要为不同分辨率的设备定义不同大小的图片资源，只需一个VectorDrawable 就够了。 

另外的一个好处就是能缩减 apk 的大小，对于对 apk 大小很纠结的开发者来说是一个好消息，但是 VectorDrawable 只支持 Android 5.0 及以上，那么我们如何让 Android 5.0 以下支持 VectorDrawable 呢?

在Android studio 3.0 后 在新建项目的时候 选了有带系统图的一般就会自动的生成对 VectorDrawable 向 Android 5.0（API Level 21） 下支持的support-vector-drawable
如果没有我们怎添加呢？

首先 再 module 的 build.gradle 中defaultConfig 添加  vectorDrawables.useSupportLibrary= true

```java

android {
    defaultConfig {

        vectorDrawables.useSupportLibrary= true
    }

}
```

其次： dependencies 中 添加 com.android.support:support-vector-drawable
```java

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:support-vector-drawable:27.1.0'
}
```

最后： 添加布局 后就可以到正常的图片使用的了，是不是很方便的？

![](https://jayqiu.github.io/blog/2018/04/img/04-26-09.jpg)






