#  ObjectBox Android 上速度最快的数据库操作库

目前的技术解决方案
Serializable：序列化对象为文件，并保存在文件里

SharedPreferences：Android官方提供的缓存文件，以XML形式存储

SQLite：官方数据库

greenDAO( Google Room)：基于SQLite的轻量级ORM

Realm：第三方数据库

ObjectBox：第三方数据库

[ObjectBox 官网（点击查看）](http://objectbox.io/)

用过EventBus和GreenDao的都知道他GreenRobot，而 ObjectBox 就是GreenRobot 推出的移动端数据库架构，是基于NoSql的特性。
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


* 注意：

默认情况下，id是会被objectbox管理的，也就是自增id，如果你想手动管理id需要在注解的时候加上@Id(assignable = true)即可。当你在自己管理id的时候如果超过long的最大值，objectbox 会报错。id=0的表示此对象未被持久化，id的值不能为负数。id的数据类型只能是long.
。

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



## 升华

###  事务：

在前文中 直接的for 循环的去添加数据是不正确的，而且还是在主线程中，会使APP卡顿，或者crash；

通过源码分析看到几乎所有ObjectBox的操作都涉及事务。如果你调用put方法，会使用一个写事务。

```java
Box.java

    public long put(T entity) {
        Cursor<T> cursor = getWriter();
        try {
            long key = cursor.put(entity);
            commitWriter(cursor);
            return key;
        } finally {
            releaseWriter(cursor);
        }
    }

       void commitWriter(Cursor<T> cursor) {
        // NOP if TX is ongoing
        if (activeTxCursor.get() == null) {
            cursor.close();
            cursor.getTx().commitAndClose();
        }
    }

Cursor.java

  public Transaction getTx() {
        return tx;
    }

```
但是对应更复杂的应用，通常值得学习事务的知识，以使您的应用程序更加的高效。

BoxStore类 提供以下方法来执行显式事务：

runInTx：在事务内运行runnable 。

runInReadTx：在只读事务中运行runnable 。与写入事务不同，多个读取事务可以同时运行。

runInTxAsync将给定的Runnable作为单独线程中的事务运行。一旦事务完成，给定的callback 被调用（回调可能为空）。

callInTx: 类似runInTx, 支持一个返回值和异常抛出。


eg: 

```java
 MyApplication.getBoxStore().runInTx(new Runnable() {
                    @Override
                    public void run() {
                        for (int i = 0; i < 10000; i++) {
                            UserEntity userEntity = new UserEntity();
                            userEntity.setAge(20);
                            userEntity.setUserName("jayqiu" + i);
                            mBox.put(userEntity);
                        }
                    }
                });


 MyApplication.getBoxStore().runInTxAsync(new Runnable() {
                    @Override
                    public void run() {
                        for (int i = 0; i < 10000; i++) {
                            UserEntity userEntity = new UserEntity();
                            userEntity.setAge(20);
                            userEntity.setUserName("jayqiu" + i);
                            mBox.put(userEntity);
                        }
                    }
                }, new TxCallback<Void>() {
                    @Override
                    public void txFinished(@Nullable Void result, @Nullable Throwable error) {
                        Log.e("Put花费时间：", (System.currentTimeMillis() - startTime) + "");
                        if(error==null){
                            runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    Toast.makeText(MainActivity.this,"成功",Toast.LENGTH_SHORT).show();
                                }
                            });

                        }else {
                            runOnUiThread(new Runnable() {
                                @Override
                                public void run() {
                                    Toast.makeText(MainActivity.this,"失败",Toast.LENGTH_SHORT).show();
                                }
                            });

                        }
                    }
                });    
```
### Relations
[一对多，多对一 @Relations（点击查看）](http://objectbox.io/documentation/relations/)

我们在现实生活中一个用户可以对应多个的地址，比如家的地址，工作的地址，学校地址等

* 一对多
 ToOne<>

* 多对一
ToMany<>

```java
@Backlink
public ToMany<> ;
```

实体 AddressEntity.java
```java

 @Entity
public class AddressEntity {
    // 可以自定义ID 默认为自增
    @Id(assignable = true)
    private  long addId;
    public ToOne<UserEntity> user;
    private String address;

    public long getAddId() {
        return addId;
    }

    public void setAddId(long addId) {
        this.addId = addId;
    }

    public ToOne<UserEntity> getUser() {
        return user;
    }

    public void setUser(ToOne<UserEntity> user) {
        this.user = user;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```


 ```java
        mBtnrRlationsPut.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                UserEntity userEntity = new UserEntity();
                userEntity.setUserName(System.currentTimeMillis() + "NAME");
                userEntity.setAge(25);
                AddressEntity addressEntity = new AddressEntity();
                addressEntity.setAddress(System.currentTimeMillis()+"Lu");
                addressEntity.getUser().setTarget(userEntity);
                long addId = MyApplication.getBoxStore().boxFor(AddressEntity.class).put(addressEntity);

                Log.e("addId：", addId + "======");
            }
        });
        mBtnrRlationsGut.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                List<AddressEntity> addList = MyApplication.getBoxStore().boxFor(AddressEntity.class).getAll();
                Log.e("addList：", addList.size() + "======");
                if(addList!=null&& addList.size()>0){
                  AddressEntity addressEntity=  addList.get(0);
                 UserEntity userEntity= addressEntity.getUser().getTarget();

                    Log.e("userEntity：", userEntity.getUserName() + "======");
                }
            }
        });                          
```

id 只能是long的数据类型

```java
错误: [ObjectBox] An @Id property has to be of type Long (com.**.objectbox.AddressEntity.addId)	

```

# 更新

[ObjectBox更新数据库实例（点击查看）](http://objectbox.io/documentation/objectbox-entity-property-migration/)



添加@Uid注解。

make project
编译, 会报错, 点击as右下 Gradle Console 会有类似报错信息:

```java
注: [ObjectBox] Starting ObjectBox processor (debug: false)
错误: [ObjectBox] UID operations for property "LocationEntity.locationTime": [Rename] apply the current UID using @Uid(3939342872662404404L) - 

[Change/reset] apply a new UID using @Uid(7349095691908173825L)

```

你的类名获取字段名称, 编译即可完成


把报错信息里后面一个新的数填写到注解里, 此处为:　@Uid(3939342872662404404L)


响应的 在ObjectiveBox 都有非常详细的实例


[Objectbox 文档](http://objectbox.io/documentation/)

[Objectbox API](http://objectbox.io/files/objectbox-java/current/)



https://www.jianshu.com/p/38c5d6f239d2









