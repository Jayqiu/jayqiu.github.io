# BottomNavigationView 底部导航栏


我们在使用底部导航栏的时候，通常是自定义用RadioButton和RadioGroup 结合；还有就是百花齐放的第三方控件，一直没有一个官方的最底部导航的支持的，比较的蛋疼；然而在
Android Support Library 25.0.0 版本中，新增加了一个API –> BottomNavigationView – 底部导航视图。

##  使用


添加依赖
```java
      compile 'com.android.support:design:27.1.0'
```

```java
    <android.support.design.widget.BottomNavigationView
        android:id="@+id/navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginEnd="0dp"
        android:layout_marginStart="0dp"
        android:background="?android:attr/windowBackground"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:itemIconTint="@color/selector_item_color"
        app:itemTextColor="@color/selector_item_color"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/navigation" />
```
实例化控件
```java
BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.navigation);

navigation.setOnNavigationItemSelectedListener(mOnNavigationItemSelectedListener);
```
实现监听

```java

private BottomNavigationView.OnNavigationItemSelectedListener mOnNavigationItemSelectedListener
            = new BottomNavigationView.OnNavigationItemSelectedListener() {

        @Override
        public boolean onNavigationItemSelected(@NonNull MenuItem item) {
            switch (item.getItemId()) {
                case R.id.navigation_home:
                    return true;
                case R.id.navigation_hot:
                    return true;
                case R.id.navigation_photo:
                    return true;
                case R.id.navigation_video:
                    return true;
                case R.id.navigation_my:
                    return true;
            }
            return false;
        }
    };
```
设置选中，默认ITEM、文字选中颜色和默认颜色；
```java



// java 代码实现 
   int[][] states = new int[][]{
            new int[]{-android.R.attr.state_checked},
            new int[]{android.R.attr.state_checked}
    };

            BottomNavigationView navigation = (BottomNavigationView) findViewById(R.id.navigation);
        int[] colors = new int[]{getResources().getColor(R.color.colorPrimaryDark),
                getResources().getColor(R.color.colorAccent)
        };
        ColorStateList csl = new ColorStateList(states, colors);
        navigation.setItemIconTintList(csl);
        navigation.setItemTextColor(csl);


// XML 布局实现
       app:itemIconTint="@color/selector_item_color"
        app:itemTextColor="@color/selector_item_color"  

//  在res文件下新建一个color 的文件 并在文件中 新建selector_item_color

<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="@color/colorAccent" android:state_checked="true"/>
    <item android:color="@color/colorAccent" android:state_pressed="true"/>
    <item android:color="@color/colorPrimaryDark"/>
</selector>      
```


background : 控件背景 

app:itemBackground : 子菜单背景 

app:itemIconTint : 图标颜色 

app:itemTextColor : 文本颜色 

app:menu : 菜单

### 注意事项
底部导航栏默认高度是56dp

菜单只能是3-5个
多了不是不能用，是有一个动画的效果；

## 源码分析




