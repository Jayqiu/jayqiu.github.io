# getFragmentManager、getSupportFragmentManager和getChildFragmentManager 的区别

使用Fragment需要熟悉几个类，包括FragmentActivity、FragmentManager、 FragmentTranscation，一个FragmentActivity可以包含多个Fragment，谁来管理？

FragmentManager就起到了作用。做Fragment的增加、删除、替换的时候，事务FragmentTranslation 来负责执行。

        getFragmentManager()和getSupportFragmentManager()的区别很容易理解，android的v4扩展包中的FragmentActivity中获取FragmentManager使用的就是getSupportFragmentManager()，android.app中获取管理类的方法就是getFragmentManager()。

 getChildFragmentManager() 属于 android.support.v4.app.Fragment
 ```java
     /**
     * Return a private FragmentManager for placing and managing Fragments
     * inside of this Fragment.
     */
    @NonNull
    final public FragmentManager getChildFragmentManager() {
        if (mChildFragmentManager == null) {
            instantiateChildFragmentManager();
            if (mState >= RESUMED) {
                mChildFragmentManager.dispatchResume();
            } else if (mState >= STARTED) {
                mChildFragmentManager.dispatchStart();
            } else if (mState >= ACTIVITY_CREATED) {
                mChildFragmentManager.dispatchActivityCreated();
            } else if (mState >= CREATED) {
                mChildFragmentManager.dispatchCreate();
            }
        }
        return mChildFragmentManager;
    }



   
 ```

getFragmentManager()在Activity中也在 android.support.v4.app.Fragment
```java
 public FragmentManager getFragmentManager() {
        return mFragments.getFragmentManager();
    }
    // android.support.v4.app.Fragment 中
    @Nullable
    final public FragmentManager getFragmentManager() {
        return mFragmentManager;
    }
```

getSupportFragmentManager() 在 android.support.v4.app.FragmentActivity

```java
   /**
     * Return the FragmentManager for interacting with fragments associated
     * with this activity.
     */
    public FragmentManager getSupportFragmentManager() {
        return mFragments.getSupportFragmentManager();
    }

```