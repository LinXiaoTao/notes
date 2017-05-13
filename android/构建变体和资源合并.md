[add-resources](https://developer.android.com/studio/write/add-resources.html#resource_merging)

[build-variants](https://developer.android.com/studio/build/build-variants.html#flavor-dimensions)

#### 应用资源

应用程序资源被组织到每个模块的 **res/** 中的特定类型目录中，还可以针对不同设备配置进行优化的每个文件的替代版本。

#### 更改资源目录

默认情况下，资源目录位于 **module-name/src/source-set-name/res/ **。例如，主资源目录在 **src/main/res/** ，调试版本的资源目录在 **src/debug/res/ **。

但是可以更改默认资源目录路径：

``` groovy
android {
  sourceSets {
    main {
      res.srcDirs = ['resources/main']
    }
    debug {
      res.srcDirs = ['resources/debug']
    }
  }
}
```

还可以为一个源集指定多个资源目录，然后构建工具将它们合并在一起

``` groovy
android {
    sourceSets {
        main {
            res.srcDirs = ['res1', 'res2']
        }
    }
}
```

#### 配置构建变体

**每个构建变体都代表您可以为应用构建的一个不同版本**。

构建变体是 Gradle 按照[特定规则集](https://developer.android.com/studio/build/build-variants.html#sourceset-build)合并在构建类型和产品风味中配置的设置、代码和资源所生成的结果。您并不直接配置构建变体，而是配置组成变体的构建类型和产品风味。

#### 配置构建类型

您可以在模块级 `build.gradle` 文件的 `android {}` 代码块内部创建和配置构建类型。当您创建新模块时，Android Studio 会自动为您创建调试和发布这两种构建类型。尽管调试构建类型不会出现在构建配置文件中，Android Studio 会为其配置 [`debuggable true`](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html#com.android.build.gradle.internal.dsl.BuildType:debuggable)。

``` groovy
android {
    ...
    defaultConfig {...}
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        debug {
            applicationIdSuffix ".debug"
        }

        /**
         * The 'initWith' property allows you to copy configurations from other build types,
         * so you don't have to configure one from the beginning. You can then configure
         * just the settings you want to change. The following line initializes
         * 'jnidebug' using the debug build type, and changes only the
         * applicationIdSuffix and versionNameSuffix settings.
         */

        jnidebug {

            // This copies the debuggable attribute and debug signing configurations.
          	// 使用 initWith 可以从其他构建类型中复制配置
            initWith debug

            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
    }
}
```

#### 配置产品风味(Product Flavors)

创建产品风味与创建构建类型类似：只需将它们添加到 `productFlavors {}` 代码块并配置您想要的设置。产品风味支持与 `defaultConfig` 相同的属性，这是因为 `defaultConfig` 实际上属于 [`ProductFlavor`](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.ProductFlavor.html) 类。这意味着，您可以在 `defaultConfig {}` 代码块中提供所有风味的基本配置，每种风味均可更改任何这些默认值

``` groovy
android {
    ...
    defaultConfig {...}
    buildTypes {...}
    productFlavors {
        demo {
            applicationIdSuffix ".demo"
            versionNameSuffix "-demo"
        }
        full {
            applicationIdSuffix ".full"
            versionNameSuffix "-full"
        }
    }
}
```

Gradle 会根据您的构建类型和产品风味自动创建构建变体，并按照 `<product-flavor><Build-Type>` 的格式命名这些变体

#### 组合多个产品风味

可以通过适用于 Gradle 的 Android 插件创建产品风味组，称为风味维度。构建您的应用时，Gradle 会将您定义的每个风味维度中的产品风味配置与构建类型配置组合来创建最终构建变体。Gradle 不会组合属于相同风味维度的产品风味。

下面的代码示例使用 [`flavorDimensions`](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.AppExtension.html#com.android.build.gradle.AppExtension:flavorDimensions(java.lang.String[])) 属性创建一个“模式”风味维度以组织“完整”和“演示”产品风味，以及一个“api”风味维度以基于 API 级别组织产品风味配置：

``` groovy
android {
  ...
  buildTypes {
    debug {...}
    release {...}
  }

  // Specifies the flavor dimensions you want to use. The order in which you
  // list each dimension determines its priority, from highest to lowest,
  // when Gradle merges variant sources and configurations. You must assign
  // each product flavor you configure to one of the flavor dimensions.
  //dimension根据书写的先后顺序决定它们的优先级别
  flavorDimensions "api", "mode"

  productFlavors {
    demo {
      // Assigns this product flavor to the "mode" flavor dimension.
      dimension "mode"
      ...
    }

    full {
      dimension "mode"
      ...
    }

    // Configurations in the "api" product flavors override those in "mode"
    // flavors and the defaultConfig {} block. Gradle determines the priority
    // between flavor dimensions based on the order in which they appear next
    // to the flavorDimensions property above--the first dimension has a higher
    // priority than the second, and so on.
    minApi24 {
      dimension "api"
      minSdkVersion '24'
      // To ensure the target device receives the version of the app with
      // the highest compatible API level, assign version codes in increasing
      // value with API level. To learn more about assigning version codes to
      // support app updates and uploading to Google Play, read Multiple APK Support
      versionCode 30000 + android.defaultConfig.versionCode
      versionNameSuffix "-minApi24"
      ...
    }

    minApi23 {
      dimension "api"
      minSdkVersion '23'
      versionCode 20000  + android.defaultConfig.versionCode
      versionNameSuffix "-minApi23"
      ...
    }

    minApi21 {
      dimension "api"
      minSdkVersion '21'
      versionCode 10000  + android.defaultConfig.versionCode
      versionNameSuffix "-minApi21"
      ...
    }
  }
}
...

```

Gradle 创建的构建变体数量等于每个风味维度中的风味数量与您配置的构建类型数量的乘积。在 Gradle 为每个构建变体或对应 APK 命名时，属于较高优先级风味维度的产品风味首先显示，之后是较低优先级维度的产品风味，再之后是构建类型。

#### 过滤变体

Gradle 会为您配置的产品风味与构建类型的每个可能的组合创建构建变体。不过，某些特定的构建变体在您的项目环境中并不必要，也可能没有意义。您可以在模块级 `build.gradle` 文件中创建一个变体过滤器，以移除某些构建变体配置。

``` groovy
android {
  ...
  buildTypes {...}

  flavorDimensions "api", "mode"
  productFlavors {
    demo {...}
    full {...}
    minApi24 {...}
    minApi23 {...}
    minApi21 {...}
  }

  variantFilter { variant ->
      def names = variant.flavors*.name
      // To check for a certain build type, use variant.buildType.name == "<buildType>"
      if (names.contains("minApi21") && names.contains("demo")) {
          // Gradle ignores any variants that satisfy the conditions above.
          // Gradle 会忽略符合条件的变体
          setIgnore(true)
      }
  }
}
...
```

#### 创建源集

默认情况下，Android Studio 会创建 **main/** 源集和目录，用于存储所有构建变体之间共享的一切资源。

然而，您可以创建新的源集来控制 Gradle 要为特定的构建类型、产品风味（以及使用[风味维度](https://developer.android.com/studio/build/build-variants.html#flavor-dimensions)时的产品风味组合）和构建变体编译和打包的确切文件。

Gradle 要求您按照与 `main/` 源集类似的特定方式组织源集文件和目录。

#### 更改默认源集配置

如果您的源未组织到 Gradle 期望的默认源集文件结构中（如上面的[创建源集](https://developer.android.com/studio/build/build-variants.html#sourcesets)部分中所述），您可以使用 [`sourceSets {}`](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.api.AndroidSourceSet.html) 代码块更改 Gradle 希望为源集的每个组件收集文件的位置。您不需要重新定位文件；只需要为 Gradle 提供相对于模块级 `build.gradle` 文件的路径，Gradle 应当可以在此路径下为每个源集组件找到文件。

``` groovy
android {
  ...
  sourceSets {
    // Encapsulates configurations for the main source set.
    main {
      // Changes the directory for Java sources. The default directory is
      // 'src/main/java'.
      java.srcDirs = ['other/java']

      // If you list multiple directories, Gradle uses all of them to collect
      // sources. Because Gradle gives these directories equal priority, if
      // you define the same resource in more than one directory, you get an
      // error when merging resources. The default directory is 'src/main/res'.
      res.srcDirs = ['other/res1', 'other/res2']

      // Note: You should avoid specifying a directory which is a parent to one
      // or more other directories you specify. For example, avoid the following:
      // res.srcDirs = ['other/res1', 'other/res1/layouts', 'other/res1/strings']
      // You should specify either only the root 'other/res1' directory, or only the
      // nested 'other/res1/layouts' and 'other/res1/strings' directories.

      // For each source set, you can specify only one Android manifest.
      // By default, Android Studio creates a manifest for your main source
      // set in the src/main/ directory.
      manifest.srcFile 'other/AndroidManifest.xml'
      ...
    }

    // Create additional blocks to configure other source sets.
    androidTest {

      // If all the files for a source set are located under a single root
      // directory, you can specify that directory using the setRoot property.
      // When gathering sources for the source set, Gradle looks only in locations
      // relative to the root directory you specify. For example, after applying the
      // configuration below for the androidTest source set, Gradle looks for Java
      // sources only in the src/tests/java/ directory.
      setRoot 'src/tests'
      ...
    }
  }
}
...

```

#### 使用源集构建

您可以使用源集目录包含您希望仅针对某些配置打包的代码和资源。Gradle 会查看这些目录并赋予以下优先级顺序：

1. 构建变体源集
2. 构建类型源集
3. 产品风味源集
4. 主源集

> **注**：如果您[组合多个产品风味](https://developer.android.com/studio/build/build-variants.html#flavor-dimensions)，产品风味之间的优先级将由它们所属的风味维度决定。在列示具有 [`android.flavorDimensions`](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.AppExtension.html#com.android.build.gradle.AppExtension:flavorDimensions(java.lang.String[])) 属性的风味维度时，所列示的第一个风味维度中的产品风味比第二个维度中的产品风味拥有更高的优先级，以此类推。此外，与属于各个产品风味的源集相比，您为产品风味组合创建的源集拥有更高的优先级。

* 一起编译 `java/` 目录中的所有源代码以生成单一的输出。

  > 注：对于给定的构建变体，如果找到两个或两个以上定义同一 Java 类的源集目录，Gradle 就会引发一个构建错误。如果针对不同的构建类型需要不同版本的 `Utility.java`，您可以让每个构建类型定义其自己的文件版本，而不将其包含在 `main/` 源集中。

* 所有清单合并为单个清单。将按照上述列表中的相同顺序指定优先级。也就是说，某个构建类型的清单设置会替换某个产品风味的清单设置，依此类推。

* 同样，`values/` 目录中的文件也会合并在一起。如果两个文件同名，例如存在两个 `strings.xml` 文件，将按照上述列表中的相同顺序指定优先级。也就是说，在构建类型源集中的文件中定义的值将会替换产品风味中同一文件中定义的值，依此类推。

* `res/` 和 `asset/` 目录中的资源将打包到一起。如果两个或两个以上的源集中定义有同名资源，将按照上述列表中的相同顺序指定优先级。

* 最后，在构建 APK 时，Gradle 会为随库模块依赖项包含的资源和清单分配最低的优先级。

#### 声明依赖项

下面的示例可以在 `app/` 模块的 `build.gradle` 文件中声明三种不同类型的直接依赖项：

``` groovy
android {...}
...
dependencies {
    // The 'compile' configuration tells Gradle to add the dependency to the
    // compilation classpath and include it in the final package.

    // Dependency on the "mylibrary" module from this project
    compile project(":mylibrary")

    // Remote binary dependency
    compile 'com.android.support:appcompat-v7:25.3.1'

    // Local binary dependency
    compile fileTree(dir: 'libs', include: ['*.jar'])
}

```

##### 模块依赖项

`compile project(':mylibrary')` 行声明了一个名为 "mylibrary" 的本地Android库模块作为依赖项，并要求构建系统在构建应用时编译并包含该本地模块。

##### 远程二进制依赖项

`compile 'com.android.support:appcompat-v7:25.3.1'` 行会通过指定其 JCenter 坐标，针对 [Android 支持库](https://developer.android.com/topic/libraries/support-library/index.html?hl=zh-cn)的 25.3.1 版本声明一个依赖项。默认情况下，Android Studio 会将项目配置为使用顶级构建文件中的 JCenter 存储区。当您将项目与构建配置文件同步时，Gradle 会自动从 JCenter 中抽取依赖项。或者，您也可以通过[使用 SDK 管理器](https://developer.android.com/studio/intro/update.html?hl=zh-cn#sdk-manager)下载和安装特定的依赖项。

##### 本地二进制依赖项

`compile fileTree(dir: 'libs', include: ['*.jar'])` 行告诉构建系统在编译类路径和最终的应用软件包中包含 `app/libs/` 目录内的任何 JAR 文件。如果您有模块需要本地二进制依赖项，请将这些依赖项的 JAR 文件复制到项目内部的 `<moduleName>/libs` 中。

该模块的某些直接依赖项可能会有其自己的依赖项，这称为该模块的*传递依赖项*。

#### 配置依赖项

可以使用特定的配置关键字告诉 Gradle 如何以及何时使用某个依赖项

##### compile

指定编译时依赖项。Gradle 将此配置的依赖项添加到类路径和应用的 APK。这是默认配置。

##### apk

指定 Gradle 需要将其与应用的 APK 一起打包的仅运行时依赖项。您可以将此配置与 JAR 二进制依赖项一起使用，而不能与其他库模块依赖项或 AAR 二进制依赖项一起使用。

##### provided

指定 Gradle *不*与应用的 APK 一起打包的编译时依赖项。如果运行时无需此依赖项，这将有助于缩减 APK 的大小。您可以将此配置与 JAR 二进制依赖项一起使用，而不能与其他库模块依赖项或 AAR 二进制依赖项一起使用。

此外，您可以通过将构建变体或测试源集的名称应用于配置关键字，为特定的构建变体或[测试源集](https://developer.android.com/studio/test/index.html?hl=zh-cn#sourcesets)配置依赖项，如下例所示。

```` groovy
dependencies {
    ...
    // Adds specific library module dependencies as compile time dependencies
    // to the fullRelease and fullDebug build variants.
    fullReleaseCompile project(path: ':library', configuration: 'release')
    fullDebugCompile project(path: ':library', configuration: 'debug')

    // Adds a compile time dependency for local tests.
    testCompile 'junit:junit:4.12'

    // Adds a compile time dependency for the test APK.
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.2'
}

````

### 配置签署设置

除非您为发布构建显式定义签署配置，否则，Gradle 不会签署发布构建的 APK。要使用 Gradle 构建配置为您的发布构建类型手动配置签署配置：

1. *创建密钥库。***密钥库**是一个二进制文件，它包含一组私钥。您必须将密钥库存放在安全可靠的地方。

2. *创建私钥。***私钥**代表将通过应用识别的实体，如某个人或某家公司。

3. 将签署配置添加到模块级 `build.gradle` 文件中：

   ``` groovy
   ...
   android {
       ...
       defaultConfig {...}
       signingConfigs {
           release {
               storeFile file("myreleasekey.keystore")
               storePassword "password"
               keyAlias "MyReleaseKey"
               keyPassword "password"
           }
       }
       buildTypes {
           release {
               ...
               signingConfig signingConfigs.release
           }
       }
   }

   ```

> 将发布密钥和密钥库的密码放在构建文件中并不安全。作为替代方案，您可以将此构建文件配置为通过环境变量获取这些密码，或让构建流程提示您输入这些密码。

要通过环境变量获取这些密码：

``` groovy
storePassword System.getenv("KSTOREPWD")
keyPassword System.getenv("KEYPWD")
```

要让构建流程在您要从命令行调用此构建时提示您输入这些密码：

``` groovy
storePassword System.console().readLine("\nKeystore password: ")
keyPassword System.console().readLine("\nKey password: ")
```

