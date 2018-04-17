# Android Lifecycle

Lifecycle组件包括LifecycleOwner、LifecycleObserver。为什么需要Lifecycle组件？
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
 


