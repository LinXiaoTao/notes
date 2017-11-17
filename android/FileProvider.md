[API](https://developer.android.com/reference/android/support/v4/content/FileProvider.html)

FileProvider 是 `ContentProvider` 的一个特殊子类，它通过创建 `content://Uri` 来代替 `file://Uri`，来促进与应用程序相关联的文件的安全共享。



### 定义 FileProvider

由于 FileProvider 的默认功能包括文件的内容URI生成，因此您不需要在代码中定义子类。相反，您可以在应用程序中包含一个 FileProvider，它可以用 XML 完全指定。

* grantUriPermissions 临时访问权限

``` xml
<manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.mydomain.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            ...
        </provider>
        ...
    </application>
</manifest>
```



### 指定可用文件

FileProvider 只能为您先前指定的目录中的文件生成 `content://Uri`。要指定目录，请使用元素的子元素在 XML 中指定其存储区域和路径 `<paths>`。

``` xml
<paths xmlns:android="http://schemas.android.com/apk/res/android">
  <!--name: URI 路径段，这个值隐藏您共享的子目录的名称-->
  <!--path: 实际共享的子目录名称，注意只能是目录，不能是文件-->
    <files-path name="my_images" path="images/"/>
    ...
</paths>
```

> `<paths` 元素必须包含至少一个子元素

* `<files-path name="name" path="path" />` 相当于 `Context.getFilesDir()`
* `<cache-path name="name" path="path" />` 相当于 `Context.getCacheDir()`
* `<external-path name="name" path="path" />` 相当于 `Enviroment.getExternalStorageDirectory()`
* `<external-files-path name="name" path="path" />` 相当于 `Context.getExternalFilesDir()`
* `<external-cache-path name="name" path="path" />` 相当于 `Context.getExternalCacheDir()`

将该文件放在 `res/xml/` 目录下，使用 `<meta-data` 将该文件添加到 `<provider` 元素里。

``` xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.mydomain.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
  <!--resource 是 paths 路径，不需要 .xml 后缀-->
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>

```



### 生成文件的 Content URI

通过使用 `getUriForFile()` 来为分享的文件生成 Content Uri。

``` java
File imagePath = new File(Context.getFilesDir(), "images");
File newFile = new File(imagePath, "default_image.jpg");
//生成的 Uri 类似：content://com.mydomain.fileprovider/my_images/default_image.jpg
Uri contentUri = getUriForFile(getContext(), "com.mydomain.fileprovider", newFile);
```



### 授予 URI 的临时权限

授予 URI 权限有两种方式：

* 调用 `Context.grantUriPermission(package,Uri,mode_flags)` 为 `context://Uri` ，使用所期望的 flags，这将授予指定的包使用 Content URI  的访问权限，该权限一直有效，直到您通过调用 `revokeUriPermission()` 或者设备重新启动来撤销该权限。

* Intent 通过 `setData()` 将 Uri 放入，`Intent.setFlags()` 使用设置的 flags

  最后，发送 Intent。权限有效期截止至其它应用所处的堆栈销毁，并且一旦授权给某一个组件后，该应用的其它组件拥有相同的访问权限。



### 向其他应用提供 Content URI

当其他应用通过 `startActivityForResult()` 启动您的 Activity 时，通过 `setResult()` 提供。

还可以将 URI 放在 `ClipData` 对象中，然后将对象添加到 Intent，发送到其他应用，可以添加多个 `ClipData` 对象，每个对象都有自己的 Content URI，当您调用 `Intent.setFlags()` 设置临时访问权限时，所有的 Content URI 都会应用。