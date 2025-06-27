WindowManagerService 源码
https://cs.android.com/android/platform/superproject/+/android14-qpr3-release:frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java


窗口动画由 WMS 的动画子系统来负责，动画子系统的管理者为 WindowAnimator。
https://cs.android.com/android/platform/superproject/+/android14-qpr3-release:frameworks/base/services/core/java/com/android/server/wm/WindowAnimator.java
WindowManagerPolicy 
https://cs.android.com/android/platform/superproject/+/android14-qpr3-release:frameworks/base/services/core/java/com/android/server/policy/WindowManagerPolicy.java
WindowManagerGlobal
https://cs.android.com/android/platform/superproject/+/android14-qpr3-release:frameworks/base/core/java/android/view/WindowManagerGlobal.java

###WindowManager 解析
AMS（ActivityManagerService）应用进程的启动、切换和调度、四大组件的启动和管理
WMS（WindowManagerService）管理的整个系统所有窗口的UI
PMS（PackageManagerService）处理包管理相关的工作，常见的比如安装、卸载应用等
###初识WMS
因为所有页面的展示都是基于Window的,在退出后台之后会在前台有一个悬浮窗口，我们通过WindowManager完成

######WindowManager涉及到的类图关系
![类图](https://upload-images.jianshu.io/upload_images/2119112-846513b25ab5899d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


####Window的创建
Activity android.app.Activity (https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/core/java/android/app/Activity.java)  88877 line
```
@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.R, trackingBug = 170729553)
    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken,
            IBinder shareableActivityToken) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);
        mActivityInfo = info;
        // 创建Window
        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(mWindowControllerCallback);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
            mWindow.setUiOptions(info.uiOptions);
        }
        ...
      //获得WindowManager
        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;

        mWindow.setColorMode(info.colorMode);
        mWindow.setPreferMinimalPostProcessing(
                (info.flags & ActivityInfo.FLAG_PREFER_MINIMAL_POST_PROCESSING) != 0);

        getAutofillClientController().onActivityAttached(application);
        setContentCaptureOptions(application.getContentCaptureOptions());
    }
```

Activity. attach() 创建Window 获得WindowManager，拿到的就是WindowManagerImpl实例
https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/core/java/android/view/Window.java

```
 public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
            boolean hardwareAccelerated) {
        mAppToken = appToken;
        mAppName = appName;
        mHardwareAccelerated = hardwareAccelerated;
        if (wm == null) {
            wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        }
        mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    }
```

#### WindowManager
WindowManager是一个接口类，继承接口ViewManager，分别用来添加、更新和删除,而且Android 所有的控件ViewGroup 也是继承ViewManager 接口
```
public interface ViewManager
{
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```
####addView  添加流程
初始化获取WindowManager

```
    private fun initWindowManager() {
        mWindowManager = mContext.getSystemService(WINDOW_SERVICE) as WindowManager
       val mInterPhoneFloatingWindowView = InterPhoneFloatingWindowView(mContext)
       val layoutParams = WindowManager.LayoutParams(
                WindowManager.LayoutParams.WRAP_CONTENT,
                WindowManager.LayoutParams.WRAP_CONTENT,
                WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,
                WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
                PixelFormat.RGBA_8888 //OPAQUE
            )
       mWindowManager?.addView(mInterPhoneFloatingWindowView!!, layoutParams)
    }
```
WindowManager 的实现类 WindowManagerImpl
[WindowManagerImpl](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/core/java/android/view/WindowManagerImpl.java
)

```
  @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyTokens(params);
        mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                mContext.getUserId());
    }
```
这里的 mGlobal 是
```
private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
```
WindowManagerImpl虽然是WindowManager的实现类，但是却没有实现什么功能，而是将功能实现委托给了WindowManagerGlobal,这里用到的就是桥接模式，而且 WindowManagerGlobal是一个单例，说明在一个进程中只有一个WindowManagerGlobal实例
mViews：存储当前应用中所有窗口的根视图（View）。
mRoots：存储每个窗口对应的 ViewRootImpl，负责窗口的绘制和事件分发。
mParams：存储每个窗口的布局参数（WindowManager.LayoutParams），用于描述窗口的属性。
WindowManagerGlobal.addView()
```
 public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
         ViewRootImpl root;
           ...
         IWindowSession windowlessSession = null;
            // If there is a parent set, but we can't find it, it may be coming
            // from a SurfaceControlViewHost hierarchy.
            if (wparams.token != null && panelParentView == null) {
                for (int i = 0; i < mWindowlessRoots.size(); i++) {
                    ViewRootImpl maybeParent = mWindowlessRoots.get(i);
                    if (maybeParent.getWindowToken() == wparams.token) {
                        windowlessSession = maybeParent.getWindowSession();
                        break;
                    }
                }
            }

            if (windowlessSession == null) {
                root = new ViewRootImpl(view.getContext(), display);
            } else {
                root = new ViewRootImpl(view.getContext(), display,
                        windowlessSession, new WindowlessWindowLayout());
            }

            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);

            // do this last because it fires off messages to start doing things
            try {
                root.setView(view, wparams, panelParentView, userId);   // 设置View
            } catch (RuntimeException e) {
                final int viewIndex = (index >= 0) ? index : (mViews.size() - 1);
                // BadTokenException or InvalidDisplayException, clean up.
                if (viewIndex >= 0) {
                    removeViewLocked(viewIndex, true);
                }
                throw e;
            }
}
```

ViewRootImpl 
```
   public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
            int userId) {
    ...
       res = mWindowSession.addToDisplayAsUser(mWindow, mWindowAttributes,
                            getHostVisibility(), mDisplay.getDisplayId(), userId,
                            mInsetsController.getRequestedVisibleTypes(), inputChannel, mTempInsets,
                            mTempControls, attachedFrame, compatScale);
    ...
}
```

Session  是IWindowSession 实现类
```
class Session extends IWindowSession.Stub implements IBinder.DeathRecipient{
  @Override
    public int addToDisplay(IWindow window, WindowManager.LayoutParams attrs,
            int viewVisibility, int displayId, @InsetsType int requestedVisibleTypes,
            InputChannel outInputChannel, InsetsState outInsetsState,
            InsetsSourceControl.Array outActiveControls, Rect outAttachedFrame,
            float[] outSizeCompatScale) {
        return mService.addWindow(this, window, attrs, viewVisibility, displayId,
                UserHandle.getUserId(mUid), requestedVisibleTypes, outInputChannel, outInsetsState,
                outActiveControls, outAttachedFrame, outSizeCompatScale);
    }
    @Override
    public int addToDisplayAsUser(IWindow window, WindowManager.LayoutParams attrs,
            int viewVisibility, int displayId, int userId, @InsetsType int requestedVisibleTypes,
            InputChannel outInputChannel, InsetsState outInsetsState,
            InsetsSourceControl.Array outActiveControls, Rect outAttachedFrame,
            float[] outSizeCompatScale) {
        return mService.addWindow(this, window, attrs, viewVisibility, displayId, userId,
                requestedVisibleTypes, outInputChannel, outInsetsState, outActiveControls,
                outAttachedFrame, outSizeCompatScale);
    }
}
```

####WindowManagerService 解析
```
public class WindowManagerService extends IWindowManager.Stub
        implements Watchdog.Monitor, WindowManagerPolicy.WindowManagerFuncs {
        
    public int addWindow(Session session, IWindow client, LayoutParams attrs, int viewVisibility,
            int displayId, int requestUserId, @InsetsType int requestedVisibleTypes,
            InputChannel outInputChannel, InsetsState outInsetsState,
            InsetsSourceControl.Array outActiveControls, Rect outAttachedFrame,
            float[] outSizeCompatScale) {
       outActiveControls.set(null, false /* copyControls */);
        int[] appOp = new int[1];
        final boolean isRoundedCornerOverlay = (attrs.privateFlags
                & PRIVATE_FLAG_IS_ROUNDED_CORNERS_OVERLAY) != 0;
        int res = mPolicy.checkAddPermission(attrs.type, isRoundedCornerOverlay, attrs.packageName,
                appOp);//调用 WindowManagerPolicy 的 checkAddPermission() 方法来检查权限  
        if (res != ADD_OKAY) {
            return res;
        }
    ...
     // 通过DisplayId来获得窗口要添加到哪个DisplayContent上
        final DisplayContent displayContent = getDisplayContentOrCreate(displayId, attrs.token);
            if (displayContent == null) {
                ProtoLog.w(WM_ERROR, "Attempted to add window to a display that does "
                        + "not exist: %d. Aborting.", displayId);
                return WindowManagerGlobal.ADD_INVALID_DISPLAY;
            }
        //  type 在 FIRST_SUB_WINDOW 与 LAST_SUB_WINDOW 之间（1000~1999），说明是子窗口
          if (type >= FIRST_SUB_WINDOW && type <= LAST_SUB_WINDOW) {
                parentWindow = windowForClientLocked(null, attrs.token, false);
                if (parentWindow == null) {
                    ProtoLog.w(WM_ERROR, "Attempted to add window with token that is not a window: "
                            + "%s.  Aborting.", attrs.token);
                    return WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN;
                }
                if (parentWindow.mAttrs.type >= FIRST_SUB_WINDOW
                        && parentWindow.mAttrs.type <= LAST_SUB_WINDOW) {
                    ProtoLog.w(WM_ERROR, "Attempted to add window with token that is a sub-window: "
                            + "%s.  Aborting.", attrs.token);
                    return WindowManagerGlobal.ADD_BAD_SUBWINDOW_TOKEN;
                }
            }
} 
```
创建获取WindowToken
```
ActivityRecord activity = null;
            final boolean hasParent = parentWindow != null;
            // Use existing parent window token for child windows since they go in the same token
            // as there parent window so we can apply the same policy on them.
            WindowToken token = displayContent.getWindowToken(
                    hasParent ? parentWindow.mAttrs.token : attrs.token);
            // If this is a child window, we want to apply the same type checking rules as the
            // parent window type.
            final int rootType = hasParent ? parentWindow.mAttrs.type : type;

            boolean addToastWindowRequiresToken = false;

            final IBinder windowContextToken = attrs.mWindowContextToken;

            if (token == null) {
                if (!unprivilegedAppCanCreateTokenWith(parentWindow, callingUid, type,
                        rootType, attrs.token, attrs.packageName)) {
                    return WindowManagerGlobal.ADD_BAD_APP_TOKEN;
                }
                if (hasParent) {
                    // Use existing parent window token for child windows.
                    token = parentWindow.mToken;
                } else if (mWindowContextListenerController.hasListener(windowContextToken)) {
                    // Respect the window context token if the user provided it.
                    final IBinder binder = attrs.token != null ? attrs.token : windowContextToken;
                    final Bundle options = mWindowContextListenerController
                            .getOptions(windowContextToken);
                    token = new WindowToken.Builder(this, binder, type)
                            .setDisplayContent(displayContent)
                            .setOwnerCanManageAppTokens(session.mCanAddInternalSystemWindow)
                            .setRoundedCornerOverlay(isRoundedCornerOverlay)
                            .setFromClientToken(true)
                            .setOptions(options)
                            .build();
                } else {
                    final IBinder binder = attrs.token != null ? attrs.token : client.asBinder();
                    token = new WindowToken.Builder(this, binder, type)
                            .setDisplayContent(displayContent)
                            .setOwnerCanManageAppTokens(session.mCanAddInternalSystemWindow)
                            .setRoundedCornerOverlay(isRoundedCornerOverlay)
                            .build();
                }
            }
```
WindowState的创建和处理
```
 final WindowState win = new WindowState(this, session, client, token, parentWindow,
                    appOp[0], attrs, viewVisibility, session.mUid, userId,
                    session.mCanAddInternalSystemWindow);
            final DisplayPolicy displayPolicy = displayContent.getDisplayPolicy();
       // 根据窗口的type对窗口的LayoutParams的一些成员变量进行修改
            displayPolicy.adjustWindowParamsLw(win, win.mAttrs);
            attrs.flags = sanitizeFlagSlippery(attrs.flags, win.getName(), callingUid, callingPid);
            attrs.inputFeatures = sanitizeInputFeatures(attrs.inputFeatures, win.getName(),
                    callingUid, callingPid, win.isTrustedOverlay());
            win.setRequestedVisibleTypes(requestedVisibleTypes);
        //检查是否可以把这个窗口添加到系统中
        res = displayPolicy.validateAddingWindowLw(attrs, callingPid, callingUid);
            if (res != ADD_OKAY) {
                return res;
            }
    ...
          //将 WindowState 添加到该 WindowState 对应的 WindowToken 中
            win.mSession.onWindowAdded(win);
            mWindowMap.put(client.asBinder(), win);

```
WindowState 添加到 mWindowMap 中，然后将 WindowState 添加到该 WindowState 对应的
WindowToken 中（实际是保存在 WndowToken 的父类 WindowContainer 中），这样WindowToken就包含了同一个组件的WindowState
#### updateViewLayout 更新
``` 
  @Override
    public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyTokens(params);
        mGlobal.updateViewLayout(view, params);
    }
```
frameworks/base/core/java/android/view/WindowManagerGlobal.java  更新
```
    public void updateViewLayout(View view, ViewGroup.LayoutParams params) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }
        if (!(params instanceof WindowManager.LayoutParams)) {
            throw new IllegalArgumentException("Params must be WindowManager.LayoutParams");
        }

        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams)params;

        view.setLayoutParams(wparams);

        synchronized (mLock) {
          //查找
            int index = findViewLocked(view, true);
            ViewRootImpl root = mRoots.get(index);
          //替换
            mParams.remove(index);
            mParams.add(index, wparams);
            root.setLayoutParams(wparams, false);   //更新
        }
    }
```
frameworks/base/core/java/android/view/ViewRootImpl.java 
```
 public void setLayoutParams(WindowManager.LayoutParams attrs, boolean newView){
        scheduleTraversals();
}
```
scheduleTraversals() 最后会调用 performTraversals() 来开始 View 的测量、布局和绘制
```
   private void performTraversals() {
       relayoutResult = relayoutWindow(params, viewVisibility, insetsPending)  
       ...
        measureHierarchy(host, lp, mView.getContext().getResources(),
                    desiredWindowWidth, desiredWindowHeight, shouldOptimizeMeasure);
       ...
           performLayout(lp, mWidth, mHeight);
            ...
            performDraw();
}
   private boolean measureHierarchy(final View host, final WindowManager.LayoutParams lp,
            final Resources res, final int desiredWindowWidth, final int desiredWindowHeight,
            boolean forRootSizeOnly) {
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec); //
            
}
   
    private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "measure");
        try {
            mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
        mMeasuredWidth = mView.getMeasuredWidth();
        mMeasuredHeight = mView.getMeasuredHeight();
        mViewMeasureDeferred = false;
    }
```

performTraversals中会依次调用performMeasure、performLayout、performDraw方法来完成对setView方法设置进来的view的测量、布局、绘制。所以View的绘制是ViewRootImpl完成的，另外当手动调用invalidate，postInvalidate，requestInvalidate也会最终调用performTraversals，来重新绘制View
```
 private int relayoutWindow(WindowManager.LayoutParams params, int viewVisibility,
            boolean insetsPending) throws RemoteException {
           relayoutResult = mWindowSession.relayout(mWindow, params,
                        requestedWidth, requestedHeight, viewVisibility,
                        insetsPending ? WindowManagerGlobal.RELAYOUT_INSETS_PENDING : 0,
                        mRelayoutSeq, mLastSyncSeqId, mRelayoutResult);
}

```
Session  内部肯定会利用 WindowManagerService 来完成 Window 的更新。
```
   @Override
    public int relayout(IWindow window, WindowManager.LayoutParams attrs,
            int requestedWidth, int requestedHeight, int viewFlags, int flags, int seq,
            int lastSyncSeqId, WindowRelayoutResult outRelayoutResult) {
        Trace.traceBegin(TRACE_TAG_WINDOW_MANAGER, mRelayoutTag);
        int res = mService.relayoutWindow(this, window, attrs, requestedWidth,
                requestedHeight, viewFlags, flags, seq, lastSyncSeqId, outRelayoutResult);
        Trace.traceEnd(TRACE_TAG_WINDOW_MANAGER);
        return res;
    }
```
######WindowManagerService
[DisplayPolicy](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/services/core/java/com/android/server/wm/DisplayPolicy.java)


[DisplayContent](https://cs.android.com/android/platform/superproject/main/+/main:frameworks/base/services/core/java/com/android/server/wm/DisplayContent.java)

```
  private int relayoutWindowInner(Session session, IWindow client, LayoutParams attrs,
            int requestedWidth, int requestedHeight, int viewVisibility, int flags, int seq,
            int lastSyncSeqId, ClientWindowFrames outFrames,
            MergedConfiguration outMergedConfiguration, SurfaceControl outSurfaceControl,
            InsetsState outInsetsState, InsetsSourceControl.Array outActiveControls,
            Bundle outBundle, WindowRelayoutResult outRelayoutResult) {
            synchronized (mGlobalLock) {
              // 获取WindowState
              final WindowState win = windowForClientLocked(session, client, false);
              ...
            win.setRequestedSize(requestedWidth, requestedHeight);
            displayPolicy.adjustWindowParamsLw(win, attrs);
          }
}
```
DisplayContent：表示一个显示设备的内容，负责管理窗口集合及其布局。
DisplayPolicy：提供窗口布局和行为的策略，定义系统装饰区域和窗口显示规则。


###问题
1.布局外点击事件处理
2.动画卡顿，每个手机位置消耗的时间不一直，

#####WindowManager.LayoutParams 中的 type 定义了窗口的类型。不同类型的窗口具有不同的行为和权限。常见的窗口类型包括：

TYPE_APPLICATION：标准应用窗口。
TYPE_APPLICATION_OVERLAY：显示在应用窗口上层的窗口。
TYPE_SYSTEM_ALERT：系统级别的警告窗口，通常用于显示系统通知。
TYPE_SYSTEM_OVERLAY：系统级别的覆盖层窗口。
TYPE_TOAST：用于显示 Toast 消息的窗口。
###问题1
#####常用的 WindowManager.LayoutParams.flags 标志
1. FLAG_NOT_FOCUSABLE
作用：使窗口无法获得焦点。通常用于非交互性窗口，如悬浮窗。
使用场景：当你希望一个窗口显示在屏幕上，但不希望它拦截用户的输入或焦点（如悬浮窗、聊天气泡等）时，可以使用这个标志。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;
2. FLAG_KEEP_SCREEN_ON
作用：让窗口的内容保持屏幕常亮，防止屏幕进入休眠状态。
使用场景：适用于需要长时间显示内容的窗口，如视频播放、实时地图、电子书阅读器等。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON;
3. FLAG_FULLSCREEN
作用：使窗口占据整个屏幕，隐藏状态栏。
使用场景：用于全屏显示的场景，如视频播放器、游戏窗口等。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_FULLSCREEN;
4. FLAG_DIM_BEHIND
作用：使窗口后面的区域变暗，通常用于对话框或提示窗口。
使用场景：用于当窗口打开时，暗化背后的内容。例如，当弹出对话框时，背景通常会变暗以增强焦点效果。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_DIM_BEHIND;
5. FLAG_LAYOUT_IN_SCREEN
作用：让窗口的内容完全填充整个屏幕。
使用场景：当你希望窗口内容占满整个屏幕时，通常和 FLAG_NOT_FOCUSABLE 一起使用。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN;
6. FLAG_LAYOUT_INSET_DECOR
作用：使窗口的布局考虑到屏幕的装饰区域（如状态栏、导航栏等），即窗口内容不会延伸到这些区域。
使用场景：如果你希望确保窗口内容不覆盖系统的装饰区域（例如状态栏或导航栏），可以使用此标志。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_LAYOUT_INSET_DECOR;
7. FLAG_ALT_FOCUSABLE_IM
作用：使窗口可以获得焦点，并且输入法能够被自动弹出。通常与 FLAG_NOT_FOCUSABLE 配合使用。
使用场景：适用于需要输入框但不需要立即获得焦点的窗口。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM;
8. FLAG_NOT_TOUCHABLE
作用：使窗口不可触摸，用户无法与该窗口交互。
使用场景：当你希望展示一个窗口但不希望用户进行任何交互时，使用这个标志。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
9. FLAG_NOT_TOUCH_MODAL
作用：使窗口在接收触摸事件时不会阻止下层窗口接收触摸事件。通常与 FLAG_NOT_FOCUSABLE 一起使用，允许下层窗口继续接受触摸事件。
使用场景：当你希望让某个窗口透明并且不会干扰用户与下层窗口的交互时。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL;
10. FLAG_WATCH_OUTSIDE_TOUCH
作用：如果窗口外部被触摸，会触发窗口的 onTouchEvent()。
使用场景：当你需要监控用户点击窗口之外的区域时使用。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_WATCH_OUTSIDE_TOUCH;
11. FLAG_SHOW_WHEN_LOCKED
作用：允许窗口在锁屏状态下显示。通常用于显示某些系统提示或应用通知。
使用场景：当你希望在锁屏时展示窗口内容时（例如显示紧急通知）。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED;
12. FLAG_TURN_SCREEN_ON
作用：让屏幕在显示窗口时自动点亮（如果当前屏幕关闭）。
使用场景：在需要显示重要窗口时，自动点亮屏幕。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON;
13. FLAG_SECURE
作用：保护窗口内容，防止其被截图或录屏。屏幕截图、录制、或者内容截取将被禁用。
使用场景：通常用于保护敏感信息，如金融应用、支付窗口等。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_SECURE;
14. FLAG_SCALED
作用：允许窗口根据设备的显示比例进行缩放。
使用场景：如果应用的窗口大小不适配设备的分辨率，可以使用这个标志来自动调整窗口的显示比例。
java
复制代码
params.flags = WindowManager.LayoutParams.FLAG_SCALED;
15. FLAG_TOUCHABLE
作用：使窗口可以接收触摸事件（通常默认会启用）

```
layoutParams.flags =
                layoutParams.flags or WindowManager.LayoutParams.FLAG_WATCH_OUTSIDE_TOUCH

this.setOnTouchListener(object : OnTouchListener {
            @SuppressLint("WrongConstant")
            override fun onTouch(v: View?, event: MotionEvent?): Boolean {
                if (event?.action == MotionEvent.ACTION_OUTSIDE) {
                
                    return true
                }
                return false
            }

        })

```
### 问题2.

```
 public void setCanPlayMoveAnimation(boolean enable) {
            if (enable) {
                privateFlags &= ~PRIVATE_FLAG_NO_MOVE_ANIMATION;
            } else {
                privateFlags |= PRIVATE_FLAG_NO_MOVE_ANIMATION;
            }
        }
   public boolean canPlayMoveAnimation() {
            return (privateFlags & PRIVATE_FLAG_NO_MOVE_ANIMATION) == 0;
        } 
```
WindowState
frameworks/base/services/core/java/com/android/server/wm/WindowState.java
``` 
void handleWindowMovedIfNeeded() {
...
canPlayMoveAnimation()
}
   private boolean canPlayMoveAnimation() {
        // During the transition from pip to fullscreen, the activity windowing mode is set to
        // fullscreen at the beginning while the task is kept in pinned mode. Skip the move
        // animation in such case since the transition is handled in SysUI.
        final boolean hasMovementAnimation = getTask() == null
                ? getWindowConfiguration().hasMovementAnimations()
                : getTask().getWindowConfiguration().hasMovementAnimations();
        return mToken.okToAnimate()
                && (mAttrs.privateFlags & PRIVATE_FLAG_NO_MOVE_ANIMATION) == 0
                && !isDragResizing()
                && hasMovementAnimation
                && !mWinAnimator.mLastHidden
                && !mSeamlesslyRotated;
    }
```
frameworks/base/services/core/java/com/android/server/wm/DisplayContent.java
```
private final Consumer<WindowState> mApplySurfaceChangesTransaction = w -> {
     WindowState.handleWindowMovedIfNeeded()
}
```


解决方法：
PRIVATE_FLAG_NO_MOVE_ANIMATION   永远不要为窗口的位置变化制作动画。在API 34 生效，   minSdk 28 只能使用反射处理
```
  val className = "android.view.WindowManager\$LayoutParams"
            try {
                val layoutParamsClass = Class.forName(className)
                val privateFlags = layoutParamsClass.getField("privateFlags")
                val noAnim = layoutParamsClass.getField("PRIVATE_FLAG_NO_MOVE_ANIMATION")
                var privateFlagsValue = privateFlags.getInt(layoutParams)
                val noAnimFlag = noAnim.getInt(layoutParams)
                privateFlagsValue = privateFlagsValue or noAnimFlag
                privateFlags.setInt(layoutParams, privateFlagsValue)
            } catch (e: Exception) {
                Log.e("IMInterPhoneFloatingWindowManager", " IMInterPhoneFloatingWindowManager init noAnim error : ${e.localizedMessage}")
            }
```
设置该窗口上位置变化是否可以播放动画。如果禁用，窗口将立即移动到新位置，而不会产生动画。


###总结
 ####1.窗口添加流程
调用 WindowManager.addView，最终调用到 WindowManagerGlobal.addView。
创建 ViewRootImpl，将视图绑定到其上。
将窗口的 View、ViewRootImpl 和布局参数存储到全局列表中。
调用系统服务 WindowManagerService 完成窗口的实际添加。
 ####2.窗口更新流程
调用 WindowManager.updateViewLayout，最终调用到 WindowManagerGlobal.updateViewLayout。
根据视图查找到对应的 ViewRootImpl。
更新窗口的布局参数，通过 WindowManagerService 通知系统更新窗口。
 ####3.窗口移除流程
调用 WindowManager.removeView，最终调用到 WindowManagerGlobal.removeView。
找到视图对应的 ViewRootImpl，销毁窗口并释放资源。
从全局列表中移除相关的视图和布局参数。








