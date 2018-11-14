# SurfaceView 


View通过刷新来重绘视图，Android系统通过发出VSYNC信号来进行屏幕的重绘，刷新的时间间隔为16ms


在一些需要频繁刷新，执行很多逻辑操作的时候，超过了16ms，就会导致卡顿

SurfaceView继承之View，但拥有独立的绘制表面，即它不与其宿主窗口共享同一个绘图表面，可以单独在一个线程进行绘制，并不会占用主线程的资源。这样，绘制就会比较高效，游戏，视频播放，还有最近热门的直播，都可以用SurfaceView

SurfaceView有两个子类GLSurfaceView和VideoView

### SurfaceView和View的区别：

View主要适用于主动更新的情况下，而SurfaceView主要适用于被动更新，例如频繁地刷新
View在主线程中对画面进行刷新，而SurfaceView通常会通过一个子线程来进行页面的刷新
View在绘图时没有使用双缓冲机制，而SufaceView在底层实现机制中就已经实现了双缓冲机制

        如果自定义View需要频繁刷新，或者刷新时数据处理量比较大，就 可以考虑使用SurfaceView来取代View了


```java

public class CameraPreview extends SurfaceView implements SurfaceHolder.Callbac{
  
}

public interface Callback {
        /**
         * This is called immediately after the surface is first created.
         * Implementations of this should start up whatever rendering code
         * they desire.  Note that only one thread can ever draw into
         * a {@link Surface}, so you should not draw into the Surface here
         * if your normal rendering will be in another thread.
         *
         * @param holder The SurfaceHolder whose surface is being created.
         */
        public void surfaceCreated(SurfaceHolder holder);

        /**
         * This is called immediately after any structural changes (format or
         * size) have been made to the surface.  You should at this point update
         * the imagery in the surface.  This method is always called at least
         * once, after {@link #surfaceCreated}.
         *
         * @param holder The SurfaceHolder whose surface has changed.
         * @param format The new PixelFormat of the surface.
         * @param width The new width of the surface.
         * @param height The new height of the surface.
         */
        public void surfaceChanged(SurfaceHolder holder, int format, int width,
                int height);

        /**
         * This is called immediately before a surface is being destroyed. After
         * returning from this call, you should no longer try to access this
         * surface.  If you have a rendering thread that directly accesses
         * the surface, you must ensure that thread is no longer touching the
         * Surface before returning from this function.
         *
         * @param holder The SurfaceHolder whose surface is being destroyed.
         */
        public void surfaceDestroyed(SurfaceHolder holder);
    }
```


SurfaceHolder.CallBack有3个方法，分别在SurfaceView创建，改变，销毁时进行回调


