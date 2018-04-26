## 来源

* [multi_project_builds](https://docs.gradle.org/current/userguide/multi_project_builds.html)

## 知识点

* Gradle 支持 multi project，根目录的 `build.gradle` 为 *rootProject*，通过在 `settings.gradle` 中 include 当前根目录下的其他 *subProject*
* multi project 可以使用 `—configure-on-demand` 启动按需配置
* multi project 可以使用 `—parallel` 并行执行，但这不会影响 evaluate 阶段 和 *rootProject* 最先执行


* task 执行体在创建时运行，`doLast` 则是在所有 project evaluate 后执行


* 使用 `project.xx` 访问属性，会继承 *rootProject* 属性，即如果当前 project 不存在该属性，则会访问 *rootProject* 的同名属性


* 在根目录的 `build.gradle` 访问 `project` 为 *rootProject*，则其他目录下的 `build.gradle` 访问 `project` 则为当前的 *subProject*，可以通过 `rootProject` 访问根目录的 *rootProject*


* 某个 project，不管是 *subProject* 还是 *rootProject* ，可以引用其他 project 的 task，但需要使用 project 作为前缀，比如：`:projectName:taskName`

* *subProject* 的 evaluate 默认根据名称的字母顺序，可以通过 `evaluationDependsOn` 修改，注意这不会影响 task 的执行顺序，可以使用 `evaluationDependsOnChildren` 让 *subProject* 先

* 要在 `setting.gradle` 中 include 更深目录结构的 *subProject*，比如 `rootProject/sub1/sub2`，应该这样使用 `include 'sub1:sub2'` 注意这样会同时将 sub1 include 进去

* 在 `setting.gradle` 中 include 的 *subProject*，不需要和实际目录对应，实际目录可以不存在，这样同样可以使用 `subProjects {}` 动态配置 *subProject*，如果存在 `build.gradle`  需要执行，则目录名要和 *subProject* 名称对应

* Multi project 如果开启了 `—parallel` 那么将并行执行 task，并行的每个工作线程将在各自的 project 执行 task，这将不依赖于默认的字母顺序，需要制定明确的依赖关系

* 调用 `build` 会编译测试当前指定 project，同时依赖 project 会进行编译，但不会测试

  * 增加 `-a` 参数那么不会调用依赖 project 进行编译
  * 如果将 `build` 更改为 `buildNeeded`  那么依赖 project 会进行编译，也会测试
  * 如果将 `build` 更改为 `buildDependents` 那么除了依赖 project 会进行编译，同时依赖于指定 project。的项目也会进行编译和测试

* multi project 的 *buildSrc* 只能存在一个，并且位于 *rootProject*

* multi project 中 *subProject* 具有继承(属性和方法)的特性，但不推荐使用，更好的选择是配置注入 (injected configuration)，例如在 *rootProject* 的 `build.gradle` 中使用 `subProjects`

  > 注意：配置注入暂时不支持方法




