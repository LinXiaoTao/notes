[参考](https://developer.android.com/training/basics/data-storage/files.html#GetWritePermission)

# 保存文件

Android 使用与其他平台上基于磁盘的文件系统类似的文件系统。



## 选择内部或外部存储

所有的 Android 设备都有这两个文件存储区域：**内部** 和 **外部** 存储。

**内部存储：**

* 它始终可用。
* 只有您的应用可以访问此处保存的文件。
* 当用户卸载您的应用时，系统会从内部存储中移除您的应用的所有文件。

**外部存储：**

* 它并非始终可用，因为用户可采用 USB 存储设备的形式装载外部存储，并在某些情况下会从设备中将其移除。
* 它是全局可读的，因此此处保存的文件可能会被其他应用读取。
* 当用户卸载您的应用时，您通过 [getExternalFilesDir()](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 等获取的文件目录，并将文件保存在这个文件目录中，系统才会移除此处您应用的文件。

> **提示：**尽管应用默认安装在内部存储中，但您可以在清单文件中指定 [android:installLocation](https://developer.android.com/guide/topics/manifest/manifest-element.html#install) 属性，这样您的应用便可安装在外部存储中。



## 获取外部存储的权限

要向外部存储写入信息，您必须在您的清单文件中请求 [WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限。

> **注意：**目前，所有应用都可以读取外部存储，而无需特别的权限。但这在将来版本中会进行更改。
>
> 如果您的应用使用 [WRITE_EXTERNAL_STORAGE](https://developer.android.com/reference/android/Manifest.permission.html#WRITE_EXTERNAL_STORAGE) 权限，那么它也隐含读取外部存储的权限。



## 将文件保存在内部存储中

在内部存储中保存文件时，您可以通过调用以下两种方法之一获取作为 `File` 的相应目录：

* [getFilesDir()](https://developer.android.com/reference/android/content/Context.html#getFilesDir()) 返回表示您的应用的内部目录的 `File`。
* [getCacheDir()](https://developer.android.com/reference/android/content/Context.html#getCacheDir()) 返回表示您的应用临时缓存文件的内部目录的 `File`。需要对这个目录实现合理大小限制。如果在系统即将耗尽存储，它会在不进行警告的情况下删除您的缓存文件。

> **注意：**您的应用的内部存储设备目录由您的应用在Android 文件系统特定位置中的软件包名称指定。从技术上讲，如果您将文件模式设置为可读，那么，另一应用也可以读取您的内部文件。但是，此应用也需要知道您的应用的软件包名称和文件名。其他应用无法浏览您的内部目录并且没有读写权限，除非您明确将文件设置为可读或可写。

```
getFilesDir: /data/data/com.example.linxiaotao.paytestproject/files

getCacheDir: /data/data/com.example.linxiaotao.paytestproject/cache
```



## 将文件保存在外部存储中

由于外部存储可能不可用-比如，当用户已将存储装载到电脑或已移除提供外部存储的 SD 卡时-因此，在访问它之前，您应该始终确认其容量。您可以通过调用 [getExternalStorageState()](https://developer.android.com/reference/android/os/Environment.html#getExternalStorageState()) 查询外部存储状态。如果返回的状态为 [MEDIA_MOUNTED](https://developer.android.com/reference/android/os/Environment.html#MEDIA_MOUNTED) ，那么您可以对文件进行读写。

``` java
/* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

/* Checks if external storage is available to at least read */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}
```

尽管外部存储可被用户和其他应用进行修改，但您可以在此处保存两类文件：

> 使用 Environment 获取的文件夹都属于外部存储。

* 公共文件

  应供其他应用和用户自由使用的文件。当用户卸载您的应用时，用户应仍可以使用这些文件。

  如果您要将公共文件保存在外部存储设备上，请使用 [getExternalStoragePublicDirectory()]([getExternalStoragePublicDirectory()](https://developer.android.com/reference/android/os/Environment.html#getExternalStoragePublicDirectory(java.lang.String)) 方法获取表示外部存储设备上相应的 `File`。

  ```
  getDataDirectory: /data

  getDownloadCacheDirectory: /cache

  getExternalStorageDirectory: /storage/emulated/0

  getExternalStoragePublicDirectory(DIRECTORY_MUSIC): /storage/emulated/0/Music

  getRootDirectory: /system
  ```

* 私有文件

  属于您的应用且在用户卸载您的应用时应予以删除的文件。尽管这些文件在技术上可被用户和其他应用访问，但它们实际上不向您的应用之外的用户提供任何输出值。当用户卸载您的应用时，系统会删除应用外部私有目录中的所有文件。

  如果您要保存您的应用专用文件，您可以通过调用 [getExternalFilesDir()](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 并向其传递您想要的目录类型的名称，从而获取相应的目录。通过这种方法创建的各个目录将添加至封装您的应用的所有外部存储文件的父目录，当用户卸载您的应用时，系统会删除这些文件。

  ```
  getExternalCacheDirs:[/storage/emulated/0/Android/data/com.example.linxiaotao.paytestproject/cache]

  getExternalCacheDir:
  /storage/emulated/0/Android/data/com.example.linxiaotao.paytestproject/cache

  getExternalMediaDirs:[/storage/emulated/0/Android/media/com.example.linxiaotao.paytestproject]

  getExternalFilesDir: /storage/emulated/0/Android/data/com.example.linxiaotao.paytestproject/files

  getExternalFilesDirs:[/storage/emulated/0/Android/data/com.example.linxiaotao.paytestproject/files]

  getExternalFilesDir(DIRECTORY_MUSIC): /storage/emulated/0/Android/data/com.example.linxiaotao.paytestproject/files/Music

  getExternalFilesDirs(DIRECTORY_MUSIC):[/storage/emulated/0/Android/data/com.example.linxiaotao.paytestproject/files/Music]
  ```




## 查询可用空间

如果您事先知道将保存的数据量，您可以通过调用 [getFreeSpace()](https://developer.android.com/reference/java/io/File.html#getFreeSpace()) 和 [getTotalSpace()](https://developer.android.com/reference/java/io/File.html#getTotalSpace()) 来提供目前分区的可用空间和目前分区的总空间。

但是，系统并不保证您可以写入于 `getFreeSpace()` 指示一样多的字节。如果返回的数字比您要保存的数据大小大出几 MB，或如果文件系统所占空间不到 90%，则可以安全继续操作，否则，您可能不应写入存储。

> **注意：**您也可以无需检查可用空间量，尝试立刻写入文件，然后在 `IOException` 出现时将其捕获，如果您不知道所需的确切空间量。



## 删除文件

您应始终删除不再需要的文件。删除文件最直接的方法就是调用 `delete()`。

如果文件保存在内部存储中，还可以调用 [deleteFile()](https://developer.android.com/reference/android/content/Context.html#deleteFile(java.lang.String))。

> **注意：**当用户卸载您的应用时，Android 系统会删除以下：
>
> * 您保存在内部存储中的所有文件
> * 您使用 [getExternalFilesDir()](https://developer.android.com/reference/android/content/Context.html#getExternalFilesDir(java.lang.String)) 等保存在外部存储中的所有文件
>
> 但是，您应手动删除使用 [getCacheDir()](https://developer.android.com/reference/android/content/Context.html#getCacheDir()) 创建的所有缓存文件。