#  ObjectBox Android上速度最快的数据库操作库

目前的技术解决方案
Serializable：序列化对象为文件，并保存在文件里
SharedPreferences：Android官方提供的缓存文件，以XML形式存储
SQLite：官方数据库
greenDAO( Google Room)：基于SQLite的轻量级ORM
Realm：第三方数据库
ObjectBox：第三方数据库

[ObjectBox 官网](http://objectbox.io/)

用过EventBus和GreenDao的都知道他GreenRobot，而 ObjectBox 就是GreenRobot 推出的移动端数据库架构。
## 使用 

### 1.引入
```java
buildscript {
    ext.objectboxVersion = '1.5.0'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.2'
        classpath "io.objectbox:objectbox-gradle-plugin:$objectboxVersion"
    }
}

在 build.gradle 中dependencies 配置 在最后 添加 apply plugin: 'io.objectbox'

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
//    implementation "io.objectbox:objectbox-gradle-plugin:1.5.0"
    annotationProcessor "io.objectbox:objectbox-processor:1.5.0"

    debugImplementation  "io.objectbox:objectbox-android-objectbrowser:1.5.0"
    releaseImplementation  "io.objectbox:objectbox-android:1.5.0"
}

apply plugin: 'io.objectbox'
```
注意：io.objectbox:objectbox-android-objectbrowser 后
出现这个错误：
```java
More than one file was found with OS independent path 'lib/armeabi-v7a/libobjectbox.so'
```
* 1.是  apply plugin: 'io.objectbox' 没有在最后添加；

### 2.编码 
* 1.初始化

官方推荐在 Application 中初始化 ObjectBox 的实例：
```java
public class MyApplication extends Application {
    private static BoxStore mBoxStore;

    @Override
    public void onCreate() {
        super.onCreate();
//        String inPath = getInnerSDCardPath() + "/db";
//        File file= new File(inPath);
//        if(!file.exists()){
//            file.mkdirs();
//        }
//        mBoxStore = MyObjectBox.builder().androidContext(this).directory(new File(inPath)).build();
        mBoxStore = MyObjectBox.builder().androidContext(this).build();
        if (BuildConfig.DEBUG) {
            // 添加调试
            boolean start=  new AndroidObjectBrowser(mBoxStore).start(this);
            Log.e("====start=======",start+"");
        }
        Log.d("App===", "Using ObjectBox " + BoxStore.getVersion() + " (" + BoxStore.getVersionNative() + ")");
    }

    public static BoxStore getBoxStore() {
        return mBoxStore;
    }

    /**
     * 获取内置SD卡路径
     *
     * @return
     */
    public String getInnerSDCardPath() {
        return Environment.getExternalStorageDirectory().getPath();
    }

}

```
<font color=#8B0000  >注意:</font>

    1. 记得在 AndroidManifest 引用自定义的 Application
    2. MyObjectBox 直接使用时找不到， 需要创建了对应的实体类后 （Ctrl+F9 ）Rebuild Project 才会出现

* 2.建立user 实体 

```java

@Entity
public class UserEntity  {
    @Id
    private long id;
    private String userName;
    private int age;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```

* 3.获取

```java

public class MainActivity extends AppCompatActivity {
    private Box<UserEntity> mBox;
    private Button mBtnPut;
    private Button mBtnGet;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        mBtnPut = findViewById(R.id.btn_put);
        mBtnGet = findViewById(R.id.btn_get);
        // 获取
        mBox = MyApplication.getBoxStore().boxFor(UserEntity.class);
        mBtnPut.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Long startTime = System.currentTimeMillis();
                // 添加
                for (int i = 0; i < 100; i++) {
                    UserEntity userEntity = new UserEntity();
                    userEntity.setAge(20);
                    userEntity.setUserName("jayqiu" + i);
                    mBox.put(userEntity);
                }

                Log.e("Put花费时间：", (System.currentTimeMillis() - startTime) + "");
            }
        });
        // 获取数据
        mBtnGet.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Long startTime = System.currentTimeMillis();
                List<UserEntity> userEntities = mBox.getAll();
                Log.e("Get花费时间：", (System.currentTimeMillis() - startTime) + "");
                if (userEntities != null && userEntities.size() > 0) {
                    for (UserEntity user : userEntities) {
                        Log.e("UserID===",user.getId()+"");
                    }
                }

            }
        });
    }
}

```

|注解|	说明|
|----- |----- |
|@Entity |	这个对象需要持久化。|
|@Id	|这个对象的主键。|
|@Index	|这个对象中的索引。对经常大量进行查询的字段创建索引，会提高你的查询性能。
|@NameInDb	|有的时候数据库中的字段跟你的对象字段不匹配的时候，可以使用此注解。
|@Transient	|如果你有某个字段不想被持久化，可以使用此注解。
|@Relation	|做一对多，多对一的注解。

运行后 默认的数据库位置在： /data/data/包名/files/objectbox/data.mdb 下

也可以在在初始化的时候定义位置：
```java
 BoxStore mBoxStore = MyObjectBox.builder().androidContext(this).directory(new File(inPath)).build();

```


### 3.调试

```java

    debugImplementation  "io.objectbox:objectbox-android-objectbrowser:1.5.0"
```

在电脑终端执行一个　adb 命令:  adb forward tcp:8090 tcp:8090

在电脑端 　http://localhost:8090/index.html　就可以查看数据库

<font color=#8B0000  >注意:</font>

boolean start=  new AndroidObjectBrowser(mBoxStore).start(this);
返回为false：

 添加后 new AndroidObjectBrowser(mBoxStore).start(this);
返回为true 但是 浏览器访问不了 ？

* 1.网络权限添加

```java
 <uses-permission android:name="android.permission.INTERNET" />
```
* 2.对应

```java

    annotationProcessor "io.objectbox:objectbox-processor:1.5.0"

    debugImplementation  "io.objectbox:objectbox-android-objectbrowser:1.5.0"
    releaseImplementation  "io.objectbox:objectbox-android:1.5.0" 
    // 使用releaseImplementation 不能使用implementation

```

![](https://jayqiu.github.io/blog/2018/04/img/04-27-16-44.png)





