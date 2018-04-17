# Dagger2-Android   使用详解

Dagger2是Dagger的升级版，是一个依赖注入框架，现在由Google接手维护
https://google.github.io/dagger/


```java

    implementation 'com.google.dagger:dagger:2.7'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.7'
//
public class User  {
    private  String userName;
    @Inject
    public User() {

    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }
}
==============================Module========================================
//1.通过User   @Inject 实例
//Module
@Module
public class UserModule {
    private MainActivity mainActivity;

    public UserModule(MainActivity mainActivity) {
        this.mainActivity = mainActivity;
    }
}
---------------------------------------------------------------------------
//

//2Module类@Provide标注的方法直接提供实例 
@Module
public class UserModule {
    private MainActivity mainActivity;

    public UserModule(MainActivity mainActivity) {
        this.mainActivity = mainActivity;
    }
    //通过 Provides
    @Provides
    User provideUser(){
        return  new User();
    }

}
===============================Module========================================

//Component
@Component(modules = UserModule.class)
public interface  UserComponent {
    void inject(MainActivity activity);
}

Ctrl+ F9 进行编译后
DaggerUserComponent

Dagger+ Component 名词
//MainActivity
public class MainActivity extends AppCompatActivity {

    private TextView mTextMessage;
    private Button mButton;
    @Inject
    User user;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        DaggerUserComponent.builder().userModule(new UserModule(this)).build().inject(this);
        mTextMessage = (TextView) findViewById(R.id.message);
        mButton=(Button)findViewById(R.id.button);
        mButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                user.setUserName("=====");
                mTextMessage.setText(user.getUserName());
            }
        });

    }

}
```

@Inject ： 注入，被注解的构造方法会自动编译生成一个Factory工厂类提供该类对象。

@Component: 注入器，类似快递员，作用是将产生的对象注入到需要对象的容器中，供容器使用。

@Module: 模块，类似快递箱子，在Component接口中通过@Component(modules = 
xxxx.class),将容器需要的商品封装起来，统一交给快递员（Component），让快递员统一送到目标容器中。


## Module和Component

我们假设案例中的Activity代表家庭住址，Student代表某个商品，现在我们需要在家（Activity）中使用商品（Student），我们网购下单，商家（代表着案例中自动生成的Student_Factory工厂类）将商品出厂，这时我们能够在家直接获得并使用商品吗？

当然不可能，虽然商品（Student）已经从工厂（Factory）生产出来，但是并没有和家（Activity）建立连接，我们还需要一个新的对象将商品送货上门，这种英雄级的人物叫做——快递员（Component,注入器）。