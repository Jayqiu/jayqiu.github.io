#  Android 7.0 (Nougat、牛轧糖)

Android 7.0是Google推出的智能手机操作系统，最终官方代号，定名为“Nougat”（牛轧糖）。2016年的Google I/O开发者大会在美国西部时间2016年5月18-20日召开，地点为山景城的Shoreline Ampitheatre圆形剧场。2016年8月22日，Google正式推送Android 7.0 Nougat正式版

2016年12月5日，Google为Android 7.0发布了重要的维护性更新，也就是Android 7.1。 [3]  Android 7.1的一个小版本更新——安卓7.1.2已于2017年4月3日推送。
2017年5月5日，Google正式向开发者发出通知，宣布Andrdoid 7.0的Beta项目正式停止，最终版本止步在Android 7.1.2，让位于Android O。

# 具体表现

## 1. FileProvider
随着Android版本越来越高，Android对隐私的保护力度也越来越大。从Android6.0引入的动态权限控制(Runtime Permissions)到Android7.0的“私有目录被限制访问”，“StrictMode API 政策”。这些更改在为用户带来更加安全的操作系统的同时也为开发者带来了一些新的任务。如何让你的APP能够适应这些改变而不是cash，是摆在每一位Android开发者身上的责任。
###  目录被限制访问
    在Android7.0中为了提高私有文件的安全性，面向 Android N 或更高版本的应用私有目录将被限制访问。对于这个权限的更改开发者需要留意一下改变：

* 私有文件的文件权限不在放权给所有的应用，使用 MODE_WORLD_READABLE 或 MODE_WORLD_WRITEABLE 进行的操作将触发 SecurityException。
* 给其他应用传递 file:// URI 类型的Uri，可能会导致接受者无法访问该路径。 因此，在Android7.0中尝试传递 file:// URI 会触发 FileUriExposedException。
* DownloadManager 不再按文件名分享私人存储的文件。COLUMN_LOCAL_FILENAME在Android7.0中被标记为deprecated ， 
旧版应用在访问 COLUMN_LOCAL_FILENAME时可能出现无法访问的路径。 面向 Android N 或更高版本的应用在尝试访问 COLUMN_LOCAL_FILENAME 时会触发 SecurityException。

### 应用间共享文件   
在Android7.0系统上，Android 框架强制执行了 StrictMode API 政策禁止向你的应用外公开 file:// URI。 如果一项包含文件 file:// URI类型 的 Intent 离开你的应用，应用
失败，并出现 FileUriExposedException 异常，如调用系统相机拍照，或裁切照片，下载安装Apk（我是在安装APK的时候）。

```java
    String cachePath = getApplicationContext().getExternalCacheDir().getPath();
    File file=new File(cachePath, "test.jpg");
    if (!file.getParentFile().exists())file.getParentFile().mkdirs();
    Uri imageUri = Uri.fromFile(file);
    Intent intent = new Intent();
    intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);//设置Action为拍照
    intent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);//将拍取的照片保存到指定URI
    startActivityForResult(intent,1000);
```


#### 使用到FileProvider
解决这个问题我们需要:
* 1. 在manifest 文件中注册provider

```java
 <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="您的包名称.fileProvider"
            android:grantUriPermissions="true"
            android:exported="false">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
```
    注意：
    exported:要求必须为false，为true则会报安全异常。

    grantUriPermissions:true，表示授予 URI 临时访问权限。

    authorities 组件标识,都以包名开头,避免和其它应用发生冲突。

* 2. 在 res 下新建xml 文件夹 ，创建 file_paths
```java
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <paths>
        <files-path name="my_images" path="images"/>
    </paths>
</resources>
```
name：一个引用字符串。

path：文件夹“相对路径”，完整路径取决于当前的标签类型。

    path可以为空，表示指定目录下的所有文件、文件夹都可以被共享。

<paths>这个元素内可以包含以下一个或多个，具体如下：

    <files-path name="name" path="path" />
    物理路径相当于Context.getFilesDir() + /path/。

    <cache-path name="name" path="path" />
    物理路径相当于Context.getCacheDir() + /path/。

    <external-path name="name" path="path" />
    物理路径相当于Environment.getExternalStorageDirectory() + /path/。

    <external-files-path name="name" path="path" />
    物理路径相当于**Context.getExternalFilesDir(String) **+ /path/。

    <external-cache-path name="name" path="path" />
    物理路径相当于Context.getExternalCacheDir() + /path/。

    注意：external-cache-path在support-v4:24.0.0这个版本并未支持，直到support-v4:25.0.0才支持.

    使用外置SD卡
    <root-path name="name" path="path" />
    物理路径相当于/path/。

* 3. 生成content://类型的Uri

    * 1.将 Uri的scheme类型为file的Uri改成了有FileProvider创建一个content类型的Uri。

    * 2.添加 intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);

        FLAG_GRANT_READ_URI_PERMISSION：表示读取权限；

        FLAG_GRANT_WRITE_URI_PERMISSION：表示写入权限。

安装APK

```java

    private void installFile() {
        File apkFile = new File(updateFilePath);
        Intent intent = new Intent(Intent.ACTION_VIEW);
        //判断是否是AndroidN以及更高的版本
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            Uri contentUri = FileProvider.getUriForFile(context, "您的包名称.fileProvider", apkFile);
            intent.setDataAndType(contentUri, "application/vnd.android.package-archive");
        } else {
            intent.setDataAndType(Uri.fromFile(apkFile), "application/vnd.android.package-archive");
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        }
        context.startActivity(intent);
    }
```

 


