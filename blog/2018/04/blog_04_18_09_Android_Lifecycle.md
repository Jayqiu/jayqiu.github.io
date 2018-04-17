# Android Lifecycle

Lifecycle组件包括LifecycleOwner、LifecycleObserver。为什么需要Lifecycle组件？并且 Lifecycle也正式植入进了SupportActivity（AppCompatActivity的基类）和Fragment中，
如果我们没有使用到Lifecycle 来管理生命周期，那么我们需要这样去处理  

```java
public interface IPresenter {

    void onCreate();

    void onStart();

    void onResume();

    void onPause();

    void onStop();

    void onDestroy();
}


public class MainPresenter implements IPresenter {
    @Override
    public void onCreate() {

    }

    @Override
    public void onStart() {

    }

    @Override
    public void onResume() {

    }

    @Override
    public void onPause() {

    }

    @Override
    public void onStop() {

    }

    @Override
    public void onDestroy() {

    }
}


public class ActMain extends AppCompatActivity {
    private MainPresenter mainPresenter;
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.act_main);
        mainPresenter=new MainPresenter();
        mainPresenter.onCreate();
    }

    @Override
    protected void onRestart() {
        super.onRestart();
    }

    @Override
    protected void onStart() {
        super.onStart();
        mainPresenter.onStart();
    }

    @Override
    protected void onStop() {
        super.onStop();
        mainPresenter.onStop();
    }

    @Override
    protected void onPause() {
        super.onPause();
        mainPresenter.onPause();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mainPresenter.onDestroy();
    }
}
```
 看了这样 是不是巨额在Actvity中都是需要去处理 这个生命周期，太臃肿了一点的；
 我们现在修改用Lifecycle组件
 ```java
public interface IPresenter extends LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    void onCreate(@NonNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    void onStart(@NonNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    void onResume(@NonNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    void onPause(@NonNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    void onStop(@NonNull LifecycleOwner owner);

    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    void onDestroy(@NonNull LifecycleOwner owner);
}


public class MainPresenter implements IPresenter {
    private static final String TAG = "MainPresenter";

    @Override
    public void onCreate(@NonNull LifecycleOwner owner) {
        Log.d(TAG, "onCreate");
    }

    @Override
    public void onStart(@NonNull LifecycleOwner owner) {
        Log.d(TAG, "onStart");
    }

    @Override
    public void onResume(@NonNull LifecycleOwner owner) {
        Log.d(TAG, "onResume");
    }

    @Override
    public void onPause(@NonNull LifecycleOwner owner) {
        Log.d(TAG, "onPause");
    }

    @Override
    public void onStop(@NonNull LifecycleOwner owner) {
        Log.d(TAG, "onStop");
    }

    @Override
    public void onDestroy(@NonNull LifecycleOwner owner) {
        Log.d(TAG, "onDestroy");

          owner.getLifecycle().removeObserver(this);
    }
}


public class ActMain extends AppCompatActivity {
    private MainPresenter mainPresenter;
    private static  final  String TAG="ActMain";

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d(TAG,"onCreate");
        setContentView(R.layout.act_main);
        mainPresenter = new MainPresenter();
        getLifecycle().addObserver(mainPresenter);

    }

    @Override
    protected void onRestart() {
        super.onRestart();
        Log.d(TAG,"onRestart");
    }

    @Override
    protected void onStart() {
        super.onStart();
        Log.d(TAG,"onStart");
    }

    @Override
    protected void onResume() {
        super.onResume();
        Log.d(TAG,"onResume");
    }

    @Override
    protected void onStop() {
        super.onStop();
        Log.d(TAG,"onStop");
    }

    @Override
    protected void onPause() {
        super.onPause();
        Log.d(TAG,"onPause");
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        Log.d(TAG,"onDestroy");
    }
}
输出：

D/ActMain: onCreate
D/MainPresenter: onCreate
D/ActMain: onStart
D/MainPresenter: onStart
D/ActMain: onResume
D/MainPresenter: onResume
D/MainPresenter: onPause
D/ActMain: onPause
D/MainPresenter: onStop
D/ActMain: onStop
D/MainPresenter: onDestroy
D/ActMain: onDestroy
 ```

 这样我们也达到了相同的效果，还不用比较臃肿的调用

Lifecycle 结构：

![](https://jayqiu.github.io/blog/2018/04/img/04-17-lifecycle.jpg)

* LifecycleObserver接口（ Lifecycle观察者）：实现该接口的类，通过注解的方式，可以通过被LifecycleOwner类的addObserver(LifecycleObserver o)方法注册,被注册后，LifecycleObserver便可以观察到LifecycleOwner的生命周期事件。

* LifecycleOwner接口（Lifecycle持有者）：实现该接口的类持有生命周期(Lifecycle对象)，该接口的生命周期(Lifecycle对象)的改变会被其注册的观察者LifecycleObserver观察到并触发其对应的事件。

* Lifecycle(生命周期)：和LifecycleOwner不同的是，LifecycleOwner本身持有Lifecycle对象，LifecycleOwner通过其Lifecycle getLifecycle()的接口获取内部Lifecycle对象。

* State(当前生命周期所处状态)：如图所示。

![](https://jayqiu.github.io/blog/2018/04/img/sequency_lifecycle.png)

* Event(当前生命周期改变对应的事件)：如图所示，当Lifecycle发生改变，如进入onCreate,会自动发出ON_CREATE事件。




