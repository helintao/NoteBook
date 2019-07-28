# targetSdkVersion
targetSdkVersion的属性值表示创建的Android项目使用哪个API版本。

## targetSdkVersion有什么用
每个Android版本都会对应一个API数字，例如Android 7.0对应的是API 24，当手机的Android系统版本升级的时候，会出现两种情况：

- 提供了新的接口。如果开发者想要在APP中使用Android 7.0提供的新功能，除了需要使用Android 7.0手机，还需要保证targetSdkVersion升级到至少24，从这个角度来说，升级 targetSdkVersion 的目的是为了使用新版本的功能。

- 旧接口的行为发生了变化。为了保证旧的APK的行为还是和以前兼容，在源码当中，有许多类似于ctx.getApplicationInfo().targetSdkVersion的判断。因此只要APK的targetSdkVersion不变，即使这个APK安装在新的Android系统上，其行为也不会发生变化，从这个角度来说，targetSdkVersion 保证了系统对旧应用的向前兼容性。

# compileSDKVersion
compileSDKVersion定义应用程序编译选择哪个Android SDK版本，通常设置为最新的API版本，它的属性值不会影响Android系统运行行为，仅仅是Android编译项目时其中的一项配置，不会打包到APK中，其目的是为了 在编译的时候检查代码的错误的警告，提示开发者修改和优化。

# minSdkVersion
minSdkVersion定义应用支持安装的最低Android版本，这个数值有两个作用：

- 告诉Google Play Store哪些Android版本的手机可以安装该APK。

- 默认情况下，lint会对代码中的API调用做出提示，假如你调用的API是在minSdkVersion之后才提供的，那么它会告诉你，虽然可以编译通过，但是会在运行的时候抛出异常。

如果调用的API是在minSdkVersion之后才提供的，解决的方案有两种：
- 运行时判断API Level，仅在足够高，有此方法的API Level系统中调用此方法。
```java
if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    //处理逻辑。
}
```

- 保证功能完整性，通过低版本的API实现功能。

# Android 6.0 适配

##  在运行时请求权限
从Android 6.0 (API >= 23)开始，用户开始在运行时向其授予权限，而不是在应用安装时授予。系统权限分为两类：

- 正常权限。如果在AndroidManifest.xml列出了正常权限，系统将自动授予该权限。

- 危险权限。如果在AndroidManifest.xml中列出了危险权限，用户必须明确批准您的应用使用这些权限。

# Android 7.0 适配

## 应用间共享文件限制
在Android 7.0系统上，Android框架强制执行了StrictMode API政策禁止向应用外公开file://URI，如果一项包含文件file://URI类型的Intent离开你的应用，即调用Uri.from(file)传递文件路径给第三方应用，会出现FileUriExposedException异常，如调用系统相机拍照、裁切照片、打开APK安装界面等。

如果要 在应用间共享文件，可以发送content://URI类型的Uri，并授予URI临时访问权限，进行此授权的最简单方式是使用FileProvider类。

步骤如下：
1. 在AndroidManiest.xml清单中注册provider
```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.banana.dome.provider"
    android:exported="false"
    android:grantUriPermissions="true">
    
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_provider_paths"/>
</provider>
```
exported为false，grantUriPermissions表示授予URI临时访问权限。

2. 指定共享目录
上文中的android:resource="@xml/file_provider_paths"指定了共享的目录，其配置如下：
```xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
<files-path
    name="captured_media"
    path="captured_media/" />
<external-path
    name="data"
    path="Android" />
<cache-path
    name="cache"
    path="appCache" />
<external-path
    name="external"
    path="" />
</paths>
```

3. 通过FileProvider，下面是打开下载完的APK的实例
```java
public static Intent getOpenFileIntent(Context context, DownloadResponse downloadResponse) {
        File file = new File(downloadResponse.getParentPath(), downloadResponse.getFileName());
        if (!file.exists()) {
            return null;
        }
        Intent intent = new Intent();
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setAction(Intent.ACTION_VIEW);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            Uri contentUri = FileProvider.getUriForFile(context, "com.banana.dome.provider", file);
            intent.setDataAndType(contentUri, downloadResponse.getMimeType());
        } else {
            intent.setDataAndType(Uri.fromFile(file), downloadResponse.getMimeType());
        }
        if (!(context instanceof Activity)) {
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        }
        return intent;
    }
```

## 系统广播删除
Android N关闭了三项系统广播：网络状态变更广播、拍照广播及录像广播。

只有在通过**动态注册**的方式才能收到网络变化的广播，在AndroidManifest.xml中静态的注册的无法收到。

# Android 8.0 适配

## 通知渠道
在Android 8.0中所有的通知都需要提供通知渠道，否则，所有通知在8.0系统上都不能正常显示。
```java
DownloadNotifier(Context context) {
    mContext = context;
    mManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        @SuppressWarnings("all") 
        final NotificationChannel channel = new NotificationChannel(
                CHANNEL_ID,
                CHANNEL_NAME,
                NotificationManager.IMPORTANCE_HIGH);
        mManager.createNotificationChannel(channel);
    }
}
```

## 悬浮窗
8.0中新增了一种悬浮窗的窗口类型，TYPE_APPLICATION_OVERLAY，如果应用使用SYSTEM_ALERT_WINDOW权限并且尝试使用以下窗口类型之一在其它应用和系统窗口上方显示提醒窗口，都会显示在TYPE_APPLICATION_OVERLAY窗口类型的下方。

- TYPE_PHONE
- TYPE_PRIORITY_PHONE
- TYPE_SYSTEM_ALERT
- TYPE_SYSTEM_OVERLAY
- TYPE_SYSTEM_ERROR
- TYPE_TOAST

如果该应用的targetSdkVersion >= 26，则应用只能使用TYPE_APPLICATION_OVERLAY窗口类型来创建悬浮窗。

## 透明窗口不允许锁定屏幕旋转
之前应用中的侧滑返回方案需要将窗口设为透明，但是由于没有适配横屏，因此将其屏幕方法锁定为竖屏。
```xml
<activity
    android:name=".circle.activity.CircleListActivity"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:screenOrientation="portrait"
    android:theme="@style/Base.Theme.CirclePage" />
```
**窗口透明 + 固定屏幕方向**会抛出下面的异常：
```java
Caused by: java.lang.IllegalStateException: Only fullscreen opaque activities can request orientation
```

解决方案有两种：

- 适配横屏，去掉固定屏幕方向的限制。
- 仅在滑动开始的时候设置窗口透明，具体实现方案 [SWipeBack](https://github.com/Simon-Leeeeeeeee/SLWidget/tree/master/swipeback)。

# Android 9.0 适配

## 明文流量的网络请求
Android 9.0限制了明文流量的网络请求，非加密的流量请求都会被系统禁止。

- 在res/xml文件夹下新建network_security_config.xml：
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true" />
</network-security-config>
```

- 在AndroidManifest.xml的application标签下进行配置：
```xml
<application
      android:networkSecurityConfig="@xml/network_security_config">
</application>
```