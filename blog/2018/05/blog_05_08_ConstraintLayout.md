# Android ConstraintLayout 

在2016年的Google I/O大会上 , Google 发布了Android Studio 2.2预览版，同时也发布了Android 新的布局方案 ConstraintLayout ， 但是最近的一年也没有大规模的使用。2017年Google发布了 Android Studio 2.3 正式版，在 Android Studio 2.3 版本中新建的Module中默认的布局就是 ConstraintLayout 。

 我们都知道，在传统的Android开发当中，界面基本都是靠编写XML代码完成的，虽然Android Studio也支持可视化的方式来编写界面，但是操作起来并不方便，我也一直都不推荐使用可视化的方式来编写Android应用程序的界面。
ConstraintLayout还有一个优点，它可以有效地解决布局嵌套过多的问题。我们平时编写界面，复杂的布局总会伴随着多层的嵌套，而嵌套越多，程序的性能也就越差。ConstraintLayout则是使用约束的方式来指定各个控件的位置和关系的，它有点类似于RelativeLayout，但远比RelativeLayout要更强大。

build.gradle 引入

```java
    implementation 'com.android.support.constraint:constraint-layout:1.1.0'
```
```java
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
    android:layout_height="match_parent">

</android.support.constraint.ConstraintLayout>
```
常用方法：

```java
layout_constraintTop_toTopOf       // 将所需视图的顶部与另一个视图的顶部对齐。 

layout_constraintTop_toBottomOf    // 将所需视图的顶部与另一个视图的底部对齐。 

layout_constraintBottom_toTopOf    // 将所需视图的底部与另一个视图的顶部对齐。 

layout_constraintBottom_toBottomOf // 将所需视图的底部与另一个视图的底部对齐。 

layout_constraintLeft_toTopOf      // 将所需视图的左侧与另一个视图的顶部对齐。 

layout_constraintLeft_toBottomOf   // 将所需视图的左侧与另一个视图的底部对齐。 

layout_constraintLeft_toLeftOf     // 将所需视图的左边与另一个视图的左边对齐。 

layout_constraintLeft_toRightOf    // 将所需视图的左边与另一个视图的右边对齐。 

layout_constraintRight_toTopOf     // 将所需视图的右对齐到另一个视图的顶部。

layout_constraintRight_toBottomOf  // 将所需视图的右对齐到另一个的底部。

layout_constraintRight_toLeftOf    // 将所需视图的右边与另一个视图的左边对齐。

layout_constraintRight_toRightOf   // 将所需视图的右边与另一个视图的右边对齐。


layout_constraintHorizontal_bias  //控件的水平偏移比例

layout_constraintVertical_bias   //控件的垂直偏移比例
```

