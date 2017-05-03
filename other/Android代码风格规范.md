[参考](https://github.com/ribot/android-guidelines/blob/master/project_and_code_guidelines.md)

1. ### 项目结构

   遵循 [Android Gradle](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Project-Structure) 定义的 Android Gradle 项目结构。[例子](https://github.com/ribot/android-boilerplate)

2. ### 文件命名

   * Class 文件

     采用**驼峰命名法**。对于继承 Android 组件的类应该以组件名结尾。

     例如：

     `LoginActivity`

   * 资源文件

     采用**小写_下划线**风格。

   * Drawable 文件

     | 类型           | 前缀              |
     | ------------ | --------------- |
     | Action bar   | `ab_`           |
     | Button       | `btn_`          |
     | Dialog       | `dialog_`       |
     | Divider      | `divider_`      |
     | Icon         | `ic_`           |
     | Menu         | `menu_`         |
     | Tabs         | `tab_`          |
     | Notification | `notification_` |

   * 图标的命名

     | 类型                              | 前缀               |
     | :------------------------------ | ---------------- |
     | Icons                           | `ic_`            |
     | Launcher icons                  | `ic_launcher`    |
     | Menu icons and Action Bar icons | `ic_menu`        |
     | Status bar icons                | `ic_stat_notify` |
     | Tab icons                       | `ic_tab`         |
     | Dialog icons                    | `ic_dialog`      |

   * 选择器状态文件的命名

     | 状态       | 后缀          |
     | -------- | ----------- |
     | Normal   | `_normal`   |
     | Pressed  | `_pressed`  |
     | Focused  | `_focused`  |
     | Disabled | `_disabled` |
     | Selected | `_selected` |

     >  如果同时存在两种状态，则叠加
     >
     >  例如：`btn_order_pressed_focused.png`

   * 布局文件

     布局文件和应用的 Android 组件匹配，将组件名称放置在开头。

     例如：`activity_login.xml`

     | 组件                  | 前缀          |
     | ------------------- | ----------- |
     | Activity            | `activity_` |
     | Fragment            | `fragment_` |
     | Dialog              | `dialog_`   |
     | AdapterView Item    | `item_`     |
     | Parial Layout(部分布局) | `partial_`  |

   * 菜单文件

     菜单文件应该与组件名称相匹配。如果菜单是用于 `UserActivity` ，那这个命名应该为 `activity_user.xml`。

   > 如果是多个页面通用的，则命名规则为`组件_功能.xml`。

   *  Values 文件

     在 `values` 文件夹中的资源文件名称应该使用复数。
     例如：`strings.xml`，`styles`。



3. ### Java风格规则

   * 字段的定义和命名

     字段应该定义在**文件最顶部**，遵守以下规则：

     * 私有，非静态字段以 **m** 开头
     * 私有，静态字段以 **s** 开头
     * 其他字段以小写字母开头
     * 常量**大写且用下划线分隔**
     * 控件字段应该与 id 保持一致

     例如：

     ``` java
     public class MyClass {
         public static final int SOME_CONSTANT = 42;
         public int publicField;
         private static MyClass sSingleton;
         int mPackagePrivate;
         private int mPrivate;
         protected int mProtected;
       	@BindView(R.id.btn_login)
       	Button mBtnLogin;
     }
     ```

   * 类成员的顺序

     1. 常量
     2. 字段
     3. 构造方法
     4. 覆盖方法和回掉(公有的或者私有的)
     5. 公有方法
     6. 私有方法
     7. 内部类或者接口

     例如：

     ``` java
     public class MainActivity extends Activity {

     	private String mTitle;
         private TextView mTextViewTitle;

         public void setTitle(String title) {
         	mTitle = title;
         }

         @Override
         public void onCreate() {
             ...
         }

         private void setUpView() {
             ...
         }

         static class AnInnerClass {

         }

     }
     ```

     如果类继承与 Android 组件，比如 Activity 和 Fragment，那覆盖方法顺序应该与生命周期相对应。

   * 方法参数顺序

     当方法涉及 Android 组件时候，如果存在 `Context` ，应该将其作为第一个参数。

     回调方法应该作为最后一个参数。

     例如：

     ``` java
     // Context always goes first
     public User loadUser(Context context, int userId);
     // Callbacks always go last
     public void loadUserAsync(Context context, int userId, UserCallback callback);
     ```

   * 字符串常量，命名和值

     1. 当常量用于定义不同的状态或者类型等，应以所属部分为前缀。

        例如：`ORDER_STATUS_CLOSE`，`ORDER_STATUS_OPEN`。

     2. 当用于 Android 组件作为键值时

        | 类型                 | 前缀        |
        | ------------------ | --------- |
        | SharedPreferences  | PREF_     |
        | Bundle             | BUNDLE_   |
        | Fragment Arguments | ARGUMENT_ |
        | Intent Extra       | EXTRA_    |
        | Intent Action      | ACTION_   |

   * 链式方法

     每个操作符都应该另起一行。

     例如：

     ``` java
     public Observable<Location> syncLocations() {
         return mDatabaseHelper.getAllLocations()
                 .concatMap(new Func1<Location, Observable<? extends Location>>() {
                     @Override
                      public Observable<? extends Location> call(Location location) {
                          return mRetrofitService.getLocation(location.id);
                      }
                 })
                 .retry(new Func2<Integer, Throwable, Boolean>() {
                      @Override
                      public Boolean call(Integer numRetries, Throwable throwable) {
                          return throwable instanceof RetrofitError;
                      }
                 });
     }
     ```

4. #### XML 风格规则

   * 当 xml 元素没有内容时，必须使用闭合标记。

     ``` xml
     <TextView
     	android:id="@+id/text_view_profile"
     	android:layout_width="wrap_content"
     	android:layout_height="wrap_content" />
     ```

   * 资源 id 使用**小写_下划线**风格。

     以元素作为 id 前缀。

     | 元素          | 前缀      |
     | ----------- | ------- |
     | `TextView`  | `text_` |
     | `ImageView` | `img_`  |
     | `Button`    | `btn_`  |
     | `Menu`      | `menu_` |

   * 字符串名称以标识其所属部分的前缀开头。例如，`login_email_hint`。如果是通用的话，则应该遵守以下规则：

     | 前缀        | 描述               |
     | --------- | ---------------- |
     | `error_`  | 一个错误消息           |
     | `msg_`    | 一个正常信息           |
     | `title_`  | 一个标题             |
     | `action_` | 一个任务比如"保存"或者"创建" |

   * 样式和主题

     样式的名称应遵守**大写驼峰法**

5. #### Git 提交规范

   [参考](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

   遵循**[Angular规范](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.7mqxm4jekyct)**

   每次提交，Commit message 都包括三个部分：Header，Body 和 Footer。

   ```
   <type>(<scope>): <subject>
   // 空一行
   <body>
   // 空一行
   <footer>
   ```

   > Head 是必需，Body 和 Footer 可以省略
   >
   > 任何一行都不得超过72个字符（或100个字符）

   * Header

     `type(必需)` 用于说明 commit 类别，只允许使用下面 7 个标识。

     * feat：新功能
     * fix：修补bug
     * docs：文档
     * style：格式
     * refactor：重构(对已有功能的修改)
     * test：增加测试
     * chore：构建过程或辅助工具的变动

     `scope(可选)` 用于说明 commit 影响的范围。

     `subject(必需)` 是 commit 的简短描述，不超过 50 个字符

   * Body

     Body 部分是对本次 commit 的详细描述，可以分成多行。

   * Footer

     Footer 部分只用于两种情况。

     1. 不兼容变动

        如果当前代码与上一个版本不兼容，则 Footer 部分以`BREAKING CHANGE`开头，后面是对变动的描述、以及变动理由和迁移方法。

     2. 关闭 Issue

        如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个 issue 。
