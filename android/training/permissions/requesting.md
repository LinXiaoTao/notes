[参考](https://developer.android.com/training/permissions/requesting.html?hl=zh-cn)

从 Android 6.0（API 级别 23）开始，用户开始在应用运行时向其授予权限，而不是在应用安装时授予。

系统权限分为两类：**正常权限** 和 **危险权限**：

- 正常权限不会直接给用户隐私权带来风险。如果您的应用在其清单中列出了正常权限，系统将自动授予该权限。
- 危险权限会授予应用访问用户机密数据的权限。如果您的应用在其清单中列出了正常权限，系统将自动授予该权限。如果您列出了危险权限，则用户必须明确批准您的应用使用这些权限。



在所有版本的 Android 中，您的应用都需要在其应用清单中同时声明它需要的正常权限和危险权限。不过，该声明的 **影响** 因系统版本和应用的目标 SDK 级别的不同而有所差异：

- 如果设备运行的是 Android 5.1 或更低版本，或者应用的目标 SDK 为 22 或更低：如果您在清单中列出了危险权限，则用户必须在安装应用时授予此权限；如果他们不授予此权限，系统根本不会安装应用。


- 如果设备运行的是 Android 6.0 或更高版本，**或者**应用的目标 SDK 为 23 或更高：应用必须在清单中列出权限，并且它必须在运行时请求其需要的每项危险权限。用户可以授予或拒绝每项权限，且即使用户拒绝权限请求，应用仍可以继续运行有限的功能。



### 检测权限

如果您的应用需要危险权限，则每次执行需要这一权限的操作时您都必须检查自己是否具有该权限。用户始终可以自由调用此权限

``` java
// Assume thisActivity is the current activity
int permissionCheck = ContextCompat.checkSelfPermission(thisActivity,
        Manifest.permission.WRITE_CALENDAR);
```

如果应用具有此权限，方法将返回 `PackageManager.PERMISSION_GRANTED`，并且应用可以继续操作。如果应用不具有此权限，方法将返回 `PERMISSION_DENIED`，且应用必须明确向用户要求权限。

### 请求权限

如果您的应用需要应用清单中列出的危险权限，那么，它必须要求用户授予该权限。Android 为您提供了多种权限请求方式。调用这些方法将显示一个标准的 Android 对话框。



#### 解释应用为什么需要权限

为了帮助查找用户可能需要解释的情形，Android 提供了一个实用程序方法，即 `shouldShowRequestPermissionRationale()`。如果应用之前请求过此权限但用户拒绝了请求，此方法将返回 `true`。

> 如果用户在过去拒绝了权限请求，并在权限请求系统对话框中选择了 **Don't ask again** 选项，此方法将返回 `false`。如果设备规范禁止应用具有该权限，此方法也会返回 `false`。



#### 请求您需要的权限

如果应用尚无所需的权限，则应用必须调用一个 `requestPermissions()` 方法，以请求适当的权限。

``` java
// Here, thisActivity is the current activity
if (ContextCompat.checkSelfPermission(thisActivity,
                Manifest.permission.READ_CONTACTS)
        != PackageManager.PERMISSION_GRANTED) {

    // Should we show an explanation?
    if (ActivityCompat.shouldShowRequestPermissionRationale(thisActivity,
            Manifest.permission.READ_CONTACTS)) {

        // Show an expanation to the user *asynchronously* -- don't block
        // this thread waiting for the user's response! After the user
        // sees the explanation, try again to request the permission.
		//用户之前已经拒绝过
    } else {

        // No explanation needed, we can request the permission.
		//直接请求权限
        ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);

        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
        // app-defined int constant. The callback method gets the
        // result of the request.
    }
}
```



#### 处理权限请求响应

当用户响应时，系统将调用应用的 `onRequestPermissionsResult()` 方法，向其传递用户响应。您的应用必须替换该方法，以了解是否已获得相应权限。

``` java
@Override
public void onRequestPermissionsResult(int requestCode,
        String permissions[], int[] grantResults) {
    switch (requestCode) {
        case MY_PERMISSIONS_REQUEST_READ_CONTACTS: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
				//用户授予了权限
                // permission was granted, yay! Do the
                // contacts-related task you need to do.

            } else {
				//用户拒绝了授予
                // permission denied, boo! Disable the
                // functionality that depends on this permission.
            }
            return;
        }

        // other 'case' lines to check for other
        // permissions this app might request
    }
}
```



>  当系统要求用户授予权限时，用户可以选择指示系统不再要求提供该权限。这种情况下，无论应用在什么时候使用 `requestPermissions()` 再次要求该权限，系统都会立即拒绝此请求。系统会调用您的 `onRequestPermissionsResult()` 回调方法，并传递 `PERMISSION_DENIED`，如果用户再次明确拒绝了您的请求，系统将采用相同方式操作。这意味着当您调用 `requestPermissions()` 时，您不能假设已经发生与用户的任何直接交互。