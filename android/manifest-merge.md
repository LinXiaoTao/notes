[参考](https://developer.android.com/studio/build/manifest-merge.html?hl=zh-cn)

APK 文件只能包含一个 `AndroidManifest.xml` 文件，但 Android Studio 项目可以包含多个文件（通过主源集、构建变体和导入的库提供）。

### 合并优先级

合并工具根据每个清单文件的优先级将所有清单文件按顺序合并到一个文件中，由低到高。

有 3 种基本的清单文件可以互相合并，它们的合并优先级如下（按优先级由高到低的顺序）：

1. 清单文件的构建变体

   如果您的变体有多个源集，则其清单优先级如下：

   1. 构建变体清单

   2. 构建类型清单

   3. 产品定制清单

      如果您使用的是定制维度，清单优先级将与每个维度在 `flavorDimensions` 属性中的列示顺序（按优先级由高到低的顺序排列）对应。

2. 应用模块的主清单文件

3. 所包括库中的清单文件

   如果您有多个库，则其清单优先级与依赖顺序（库出现在 Gradle `dependencies` 块中的顺序）匹配。

注：这些是对所有源集都相同的合并优先级，如[使用源集构建](https://developer.android.com/studio/build/build-variants.html?hl=zh-cn#sourceset-build) 中所述。

> 重要说明： `build.gradle` 文件中的构建配置将替换合并清单文件中的任何对应属性。

### 合并冲突启发式算法

合并工具可以在逻辑上将一个清单中的每个 XML 元素与另一个清单中的对应元素相匹配。 

如果优先级较低的清单中的元素与优先级较高的清单中的任何元素均不匹配，则该元素将被添加至合并清单。 但是，如果*有*匹配元素，则合并工具会尝试将其中的所有属性合并到相同元素中。如果工具发现两个清单包含相同属性，但值不相同，则会出现合并冲突。

**表 1.** 属性值的默认合并行为

| 高优先级属性 | 低优先级属性 | 属性的合并结果 |
| ------ | :----- | ------- |
| 没有值    | 没有值    | 没有值     |
| 没有值    | 值 B    | 值 B     |
| 值 A    | 没有值    | 值 A     |
| 值 A    | 值 A    | 值 A     |
| 值 A    | 值 B    | 冲突错误    |

在某些情况下，合并工具会采取其他行为方式以避免合并冲突：

* `<manifest>` 元素中的属性绝不合并—仅使用优先级最高的清单中的属性。
* [android:required 属性 >``](https://developer.android.com/guide/topics/manifest/uses-feature-element.html?hl=zh-cn) and [``](https://developer.android.com/guide/topics/manifest/uses-library-element.html?hl=zh-cn) 元素使用 *OR* 合并，因此如果出现冲突，系统将应用 `"true"` 并始终包括某个清单所需的功能或库。
* [`<used-sdk>`](https://developer.android.com/guide/topics/manifest/uses-sdk-element.html?hl=zh-cn) 元素始终使用优先级较高的清单中的值，但以下情况除外：
  * 如果低优先级清单的 `minSdkVersion` 值*较高*，除非您应用 [`overrideLibrary`](https://developer.android.com/studio/build/manifest-merge.html?hl=zh-cn#override_wzxhzdk35uses-sdk_for_imported_libraries) 合并规则。
  * 如果低优先级清单的 `targetSdkVersion` 值*较低*，合并工具将使用高优先级清单中的值，但也会添加任何必要的系统权限，以确保所导入的库继续正常工作（适用于较高的 Android 版本具有更多权限限制的情况）。
* 绝不会在清单之间匹配 `<intent-filter>` 元素。 每个元素都被视为唯一元素，并添加至合并清单中的常用父元素。


> **不依赖于默认属性值。**由于所有唯一属性都合并到相同元素中，如果高优先级清单实际上依赖于属性的默认值而不需要声明，则可能会导致意外结果。例如，如果高优先级清单*不*声明`android:launchMode` 属性，则会使用 `"standard"` 的默认值；但如果低优先级清单声明此属性具有其他值，该值将应用于合并清单（替代默认值）。因此，您应该按期望明确定义每个属性。

### 合并规则标记

合并规则标记是一个 XML 属性，可用于表达您对关于如何解决合并冲突或删除不需要的元素和属性的首选项。 您可以对整个元素或只对元素中的特定属性应用标记。

所有标记均属于 Android `tools` 命名空间，因此您必须先在 `<manifest>` 元素中声明此命名空间，如下文所示：

``` xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp"
    xmlns:tools="http://schemas.android.com/tools">
```

#### 节点标记

* tools:node="merge"

  如果使用[合并冲突启发式算法](https://developer.android.com/studio/build/manifest-merge.html?hl=zh-cn#merge_conflict_heuristics)时没有冲突，则合并此标记中的所有属性以及所有嵌套元素。**这是元素的默认行为。**

* tools:node="merge-only-attributes"

  仅合并此标记中的属性，不合并嵌套元素。

* `tools:node="remove"`

  从合并清单中删除此元素。 尽管您似乎应该仅删除此元素，但如果您发现合并清单中有不需要的元素，则必须使用此选项。该选项由不受您控制的低优先级清单（如导入的库）提供(用于删除由低优先级合并来的元素)。

* tools:node="removeAll"

  与 `tools:node="remove"` 类似，但它会删除与此元素类型相匹配的所有元素（同一父元素内）。

* tools:node="replace"

  完全替换低优先级元素。 也就是说，如果低优先级清单中有匹配元素(同一父元素内，和 "removeAll" 一样的范围)，请将其忽略并完全按照其在此清单中显示样子来使用该元素。

* `tools:node="strict"`

  当此元素在低优先级清单中的情况与在高优先级清单中的情况不完全匹配时生成构建故障（除非已通过其他合并规则标记解决）。 这将替换[合并冲突启发式算法](https://developer.android.com/studio/build/manifest-merge.html?hl=zh-cn#merge_conflict_heuristics)。

  > 严格模式，用于生成合并错误来提醒。通常，这两个元素会完全地合并在一起，如以上 `tools:node="merge"` 示例所示，但这样会带来一些意想不到的错误。

#### 属性标记

要改为仅向清单标记中的特定属性应用合并规则，请使用以下属性。每个属性接受一个或多个属性名称（包括属性命名空间），并以逗号分隔。

* `tools:remove="attr, ..."`

  从合并清单中删除指定属性。 尽管 您似乎可以仅删除这些属性，但如果 低优先级清单文件*不包括*这些 属性，而且您希望确保它们不纳入合并 清单，则必须使用此选项。

* tools:replace="attr, ..."

  将低优先级清单中的指定属性替换为 此清单中的属性。 换言之，始终保持 高优先级清单的值。

* tools:strict="attr, ..."

  当这些属性在低优先级清单中的情况与 在高优先级 清单中的不完全匹配时生成构建故障。 **这是所有属性的默认行为**，具有 [合并冲突启发式算法](https://developer.android.com/studio/build/manifest-merge.html?hl=zh-cn#merge_conflict_heuristics)中介绍的特殊行为的属性除外。

您也可以对一个元素应用多个标记

#### 标记选择器

如果您想仅对某个特定的导入库应用合并规则标记，请添加具有库包名称的 `tools:selector` 属性。

> **注：** 如果您将此功能与其中一个属性标记配合使用，它将应用至标记中指定的所有选项。

#### 对于所有导入库，将替换 <used-sdk>

默认情况下，导入 `minSdkVersion` 值*高于*主清单文件的库时会出错，而且无法导入该库。 要使合并工具忽略此冲突并导入库，同时保持应用的低 `minSdkVersion` 值，请将 `overrideLibrary` 属性添加至 `<uses-sdk>` 标记。属性值可以是一个或多个库包名称（以逗号分隔），指明可能替换主清单的 `minSdkVersion` 的库。

#### 隐式系统权限

在最近的 Android 版本中，应用曾经可以自由访问的某些 Android API 已受 [系统 权限](https://developer.android.com/guide/topics/security/permissions.html?hl=zh-cn)限制。 为了避免中断预期会访问这些 API 的应用，最近的 Android 版本允许应用在无权限的情况下继续访问这些 API，前提是它们已将 `targetSdkVersion` 设置为低于添加限制的版本的值。此行为有效地向应用授予了*隐式 权限*，以允许访问 API。

如果低优先级清单文件提供隐式权限的 `targetSdkVersion` 值较低，而且高优先级清单*没有*相同的隐式权限（由于其 `targetSdkVersion` 等于或高于添加限制的版本），合并工具将向合并清单*显式*添加系统权限。

#### 会检查合并清单并查找冲突

### 附录：合并策略

清单合并工具可以在逻辑上将某个清单中的每个 XML 元素与其他清单中的对应元素相匹配。 合并工具会使用“匹配关键字”匹配每个元素，匹配关键字可以是唯一的属性值（如 `android:name`或标记本身的天然唯一性（例如，只能有一个 `<supports-screen>` 元素）。 如果两个清单具有相同的 XML 元素，工具将采用三种合并策略中的一种来合并这两个元素：

合并

将所有非冲突属性合并到同一标记中， 然后按其各自的合并策略合并子元素。 如果任何属性 相互冲突，请使用[合并规则标记](https://developer.android.com/studio/build/manifest-merge.html?hl=zh-cn#merge_rule_markers)将它们合并在一起。

仅合并子项

不整合或合并属性（仅保留 优先级最高的清单文件提供的属性）并按照其合并策略 合并子项。

保留

将元素“按原样”保留，然后将其添加至 合并文件中的常用父元素。 此策略仅在可接受相同元素的多个 声明时使用。

| 元素                       | 合并策略 | 合并关键字                                    |
| ------------------------ | :--- | ---------------------------------------- |
| `<action>`               | 合并   | `android:name` 属性                        |
| `<activity>`             | 合并   | `android:name` 属性                        |
| `<application>`          | 合并   | 每个 `<manifest>` 仅一个                      |
| `<category>`             | 合并   | `android:name` 属性                        |
| `<data>`                 | 合并   | 每个 `<intent-filter>` 仅 1 个               |
| `<grant-uri-permission>` | 合并   | 每个 `<provider>` 仅 1 个                    |
| `<instrumentation>`      | 合并   | `android:name` 属性                        |
| `<intent-filter>`        | 保留   | 不匹配；允许父元素内的多个声明                          |
| `<manifest>`             | 合并   | 每个文件仅 1 个                                |
| `<meta-data>`            | 合并   | `android:name` 属性                        |
| `<path-permission>`      | 合并   | 每个 `<provider>` 仅 1 个                    |
| `<permission-group>`     | 合并   | `android:name` 属性                        |
| `<permission>`           | 合并   | `android:name` 属性                        |
| `<permission-tree>`      | 合并   | `android:name` 属性                        |
| `<provider>`             | 合并   | `android:name` 属性                        |
| `<receiver>`             | 合并   | `android:name` 属性                        |
| `<screen>`               | 合并   | `android:screenSize` 属性                  |
| `<service>`              | 合并   | `android:name` 属性                        |
| `<supports-gl-texture>`  | 合并   | `android:name` 属性                        |
| `<supports-screen>`      | 合并   | 每个 `<manifest>` 仅 1 个                    |
| `<uses-configuration>`   | 合并   | 每个 `<manifest>` 仅 1 个                    |
| `<uses-feature>`         | 合并   | `android:name` 属性（如果不存在，则使用 `android:glEsVersion` 属性） |
| `<uses-library>`         | 合并   | `android:name` 属性                        |
| `<uses-permission>`      | 合并   | `android:name` 属性                        |
| `<uses-sdk>`             | 合并   | 每个 `<manifest>` 仅 1 个                    |
| 自定义元素                    | 合并   | 无匹配；合并工具不了解这些信息，因此它们始终 包括在合并清单中          |