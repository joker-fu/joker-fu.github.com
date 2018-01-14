---
title: Android 7.0权限适配：FileUriExposedException异常
copyright: true
date: 2018-01-02 22:34:28
categories: Android
tags: Android
---
今天来聊聊Android 7.0 FileUriExposedException异常，以及它的使用方法和使用场景

#### 一 描述
1. 问题
对于面向 Android 7.0 的应用，Android 框架执行的 [StrictMode](https://developer.android.com/reference/android/os/StrictMode.html)API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException异常
2. 解决方案
要在应用间共享文件，您应发送一项 content://URI，并授予 URI 临时访问权限。进行此授权的最简单方式除了将targetSdkVersion改成24以下，就是使用 [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)类

**官网对FileProvider描述：**
>FileProvider是ContentProvider的一个特殊子类，它通过创建内容来实现与应用程序相关联的文件的安全共享：// Uri用于文件，而不是文件：/// Uri。

>内容URI允许您使用临时访问权限来授予读取和写入访问权限。当您创建包含内容URI的Intent时，为了将内容URI发送到客户端应用程序，还可以调用Intent.setFlags（）来添加权限。只要接收活动的堆栈处于活动状态，客户端应用程序就可以使用这些权限。对于要访问服务的意图，只要服务正在运行，权限就可用。

>相比之下，为了控制对文件的访问：/// Uri你必须修改底层文件的文件系统权限。您提供的权限可用于任何应用程序，并在您更改之前保持有效。这种访问水平基本上是不安全的。

>内容URI提供的增加文件访问安全级别使FileProvider成为Android安全基础架构的关键部分。

#### 二 如何使用FileProvider
我们先看如何使用FileProvider，官网也有详细说明：https://developer.android.com/reference/android/support/v4/content/FileProvider.html

###### 1. 定义FileProvider
由于FileProvider的默认功能，包括内容URI代的文件，你不需要在代码中定义一个子类。我们在manifest中声明provider
```
<manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="包名.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths"/>
        </provider>
        ...
    </application>
</manifest>
```
android:name 【固定值】 FileProvider的包名+类名 
android:authorities	【自定义】 推荐以包名+”.fileprovider”方式命名，增加辨别性，系统唯一
android:exproted	要求必须为false，为true则会报安全异常
android:grantUriPermissions	是否允许为文件设置临时权限	“true”
android:resource="@xml/file_paths"就是我们的共享路径配置的xml文件
###### 2 . 配置file_paths
FileProvider只能生成你事先指定的 content URI，file_paths配置如下：
```
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path
        name="external"
        path=""/>
    <external-path
        name="my_images"
        path="Android/data/包名/files/Pictures/"/>
    <external-path
        name="images"
        path="Pictures/"/>
</paths>
```
<small>__注意：__ 注： XML文件是你可以指定你要共享的目录的唯一途径，你不能以编程方式添加一个目录，至少配置一个external-path节点</small>

在paths节点内部支持以下几个子节点，分别为：

- <root-path/> 代表设备的根目录new File("/")

- <files-path/> 代表该文件files/的应用程序的内部存储区的子目录，等同于context.getFilesDir()

- <cache-path/> 代表应用程序的内部存储区域的缓存子目录的文件，等同于context.getCacheDir()

- <external-path/> 代表在外部存储区根目录的文件，等同于Environment.getExternalStorageDirectory()

- <external-files-path> 代表应用程序的外部存储区根目录的文件，等同于Context.getExternalFilesDir(String) /Context.getExternalFilesDir(null)

- <external-cache-path> 代表应用程序的外部缓存区根目录的文件，等同于Context.getExternalCacheDir()

file_paths用来指定Uri共享和真实路径的映射关系，name属性的值可以自定义，path属性的值表示共享的具体位置，设置为空，就表示共享整个SD卡，也可指定对应的SDcard下的文件目录，根据需求自行定义
###### 3. 获得content uri
使用getUriForFile()将file:// 转换成 content://
 Uri fileUri = FileProvider.getUriForFile(this, "包名.fileprovider", file);
###### 4. 临时读写权限授权
需要对接收应用设置读权限或写权限亦或读写均设置：
FLAG_GRANT_READ_URI_PERMISSION：读权限
FLAG_GRANT_WRITE_URI_PERMISSION：写权限
授权方式：
1. 使用Intent.addFlags或setFlags，该方式授权的有效期限，权限截止于该 App 所处的堆栈被销毁自动回收（APP销毁），主要用于针对intent.setData，setDataAndType以及setClipData相关方式传递uri
2. 使用grantUriPermission(String toPackage, Uri uri, int modeFlags)来进行授权，该方式授权的有效期限，从授权一刻开始，手动调用 Context.revokeUriPermission() 方法或者设备重启才截止

#### 三 使用场景
a. 相机拍照
Android 7.0之前我们这样拍照，没有什么问题（**忽略6.0权限问题**）：
```
    private static final int REQUEST_TAKE_PHOTO = 0X11;
    private Uri imageUri ;
       private void takePhoto() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        //判断是否有相机应用
        if (takePictureIntent.resolveActivity(getActivity().getPackageManager()) != null) {
            //获取存储路径 没有则创建
            File directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);
            if (!directory.exists()) {
                if (!directory.mkdir()) {
                    return;
                }
            }
            File file = new File(directory.getAbsolutePath(), new SimpleDateFormat("yyyyMMdd-HHmmss", Locale.CHINA)
                    .format(new Date()) + ".jpeg");
            imageUri = Uri.fromFile(file);
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
            startActivityForResult(takePictureIntent, TAKE_PHOTO);
        } else {
            ToastUtil.showShort(getString(R.string.TakePhoto_Error));
        }
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == RESULT_OK && requestCode == REQUEST_TAKE_PHOTO) {
            // 通知图库更新
            getActivity().sendBroadcast(new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE, imageUri ));

        }
    }
```
如果我们使用Android 7.0或者以上的原生系统运行，发现应用直接停止运行，如文章开头所说抛出了android.os.FileUriExposedException：
```
android.os.FileUriExposedException: 
    file:///storage/emulated/0/Pictures/20170723-201847.jpeg exposed   beyond app through ClipData.Item.getUri()
    at android.os.StrictMode.onFileUriExposed(StrictMode.java:1799)
    at android.net.Uri.checkFileUriExposed(Uri.java:2346)
```
接下来根据官网的解决办法，如第二步所说配置好 [FileProvider](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)，更改拍照方法：
```
private void takePhoto() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        //判断是否有相机应用
        if (takePictureIntent.resolveActivity(getActivity().getPackageManager()) != null) {
            //获取存储路径 没有则创建
            File directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);
            if (!directory.exists()) {
                if (!directory.mkdir()) {
                    return;
                }
            }
            File file = new File(directory.getAbsolutePath(), new SimpleDateFormat("yyyyMMdd-HHmmss", Locale.CHINA)
                    .format(new Date()) + ".jpeg");
            Uri uri = imageUri = Uri.fromFile(file);
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
                //兼容7.0
                uri = FileProvider.getUriForFile(getApplication(), "包名.fileprovider", file);
                //添加权限 这一句表示对目标应用临时授权该Uri所代表的文件
                takePictureIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
                takePictureIntent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
            }
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
            startActivityForResult(takePictureIntent, TAKE_PHOTO);
        } else {
            ToastUtil.showShort(getString(R.string.TakePhoto_Error));
        }
    }
```
添加了版本判断，并使用 FileProvider.getUriForFile()获得content Uri，方法主要更改如下：
```
     if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        //兼容7.0
        uri = FileProvider.getUriForFile(getApplication(), "包名.fileprovider", file);
        //添加权限 这一句表示对目标应用临时授权该Uri所代表的文件
        takePictureIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
        takePictureIntent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
    }
```
当然也可以不用判断版本，直接使用FileProvider.getUriForFile(getApplication(), "包名.fileprovider", file)获得Uri替换Uri.fromFile(file)，但是切记需要进行授权和取消授权，否则4.4以下会报Permission Denial

b. 图片裁剪
```
/**
 * @param activity    当前activity
 * @param orgUri      剪裁原图的Uri
 * @param desUri      剪裁后的图片的Uri
 * @param aspectX     X方向的比例
 * @param aspectY     Y方向的比例
 * @param width       剪裁图片的宽度
 * @param height      剪裁图片高度
 * @param requestCode 剪裁图片的请求码
 */
public static void cropImageUri(Activity activity, Uri orgUri, Uri desUri, int aspectX, int aspectY, int width, int height, int requestCode) {
        Intent intent = new Intent("com.android.camera.action.CROP");
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
        }
        intent.setDataAndType(orgUri, "image/*");
        intent.putExtra("crop", "true");
        intent.putExtra("aspectX", aspectX);
        intent.putExtra("aspectY", aspectY);
        intent.putExtra("outputX", width);
        intent.putExtra("outputY", height);
        intent.putExtra("scale", true);
        //将剪切的图片保存到目标Uri中
        intent.putExtra(MediaStore.EXTRA_OUTPUT, desUri);
        intent.putExtra("return-data", false);
        intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
        intent.putExtra("noFaceDetection", true);
        activity.startActivityForResult(intent, requestCode);
    }
```
c. 安装apk
```
// 安装Apk
public void installApk(Context context) {
    File file = new File(Environment.getExternalStorageDirectory(), "app.apk");
    Intent intent = new Intent(Intent.ACTION_VIEW);
    Uri uri = Uri.fromFile(file);
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
        uri = FileProvider.getUriForFile(context, "包名.fileprovider", file);
        intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION | Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
    }
    intent.setDataAndType(uri, "application/vnd.android.package-archive");
    context.startActivity(intent);
}
```
大概使用就这么多，望多多指教。

另附上：[官网学习使用FileProvider地址](https://developer.android.com/training/secure-file-sharing/index.html)