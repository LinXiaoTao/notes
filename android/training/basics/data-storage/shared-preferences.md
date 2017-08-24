[参考](https://developer.android.com/training/basics/data-storage/shared-preferences.html)

# 保存键值对

如果您有想要保存的相对较小键值集合，您应使用 [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html) API。



## 获取共享首选项的句柄

您可以通过调用以下两种方法之一创建新的共享首选项文件或访问现有的文件：

* [getSharedPreferences()](https://developer.android.com/reference/android/content/Context.html#getSharedPreferences(java.lang.String, int)) - 如果您需要按照您用第一个参数指定的名称来识别多个共享首选项文件。
* [getPreferences()](https://developer.android.com/reference/android/app/Activity.html#getPreferences(int)) - 如果您只需使用 Activity 的一个共享首选项。因为此方法会检索属于该 Activity 的默认共享首选项文件 (当前 Activity 的类名，再删除掉包名前缀)。

> **注意：**如果您创建 [MODE_WORLD_READABLE](https://developer.android.com/reference/android/content/Context.html#MODE_WORLD_READABLE) 或 [MODE_WORLD_WRITEABLE](https://developer.android.com/reference/android/content/Context.html#MODE_WORLD_WRITEABLE) 的共享首选项文件，那么知道文件标识符的任何其他应用都可以访问您的数据。(这两种文件模式在 API 17 后就被弃用了，如果在 [Android N](https://developer.android.com/reference/android/os/Build.VERSION_CODES.html#N) 上使用它们，将会抛出 [SecurityException](https://developer.android.com/reference/java/lang/SecurityException.html))



## 写入共享首选项

要写入共享首选项文件，请通过对您的 [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html) 调用 `edit()` 来创建一个 [SharedPreferences.Editor](https://developer.android.com/reference/android/content/SharedPreferences.Editor.html)。

传递您想要使用诸如 `putInt()` 和 `putString()` 方法写入的键和值，然后，调用 `commit()` 以保存所做的更改。



## 从共享首选项读取信息

要从共享首选项文件中检索值，请调用诸如 `getInt()` 和 `getString()` 等方法，为您想要的值提供键，并根据需要提供在键不存在的情况下返回的默认值。

