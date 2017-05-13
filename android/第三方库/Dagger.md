[参考](https://google.github.io/dagger/android.html)

#### Dagger 在 Android 上的问题

使用Dagger编写Android应用程序的主要困难之一是许多Android框架类由操作系统本身实例化，例如Activity和Fragment，但是如果能够创建所有注入的对象，Dagger的效果最好。相反，您必须在生命周期方法中执行成员注入。这意味着很多类最终看起来像：

```java
public class FrombulationActivity extends Activity {
  @Inject Frombulator frombulator;

  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // DO THIS FIRST. Otherwise frombulator might be null!
    ((SomeApplicationBaseType) getContext().getApplicationContext())
        .getApplicationComponent()
        .newActivityComponentBuilder()
        .activity(this)
        .build()
        .inject(this);
    // ... now you can write the exciting code
  }
}
```

这有几个问题：

1. 复制代码使得以后很难重构。随着越来越多的开发人员复制粘贴该块，更少的知道它实际上做了什么。
2. 更重要的是，它要求注射类型（FrombulationActivity）知道其注射器。即使这是通过接口而不是具体类型完成的，它打破了依赖注入的核心原则：一个类不应该知道它是如何注入的。

#### 注入 Activity 实例

1. 安装 `AndroidInjectionModule` 在你的应用组件，确保这些疾病类型所需的所有绑定都可用。

2. 写一个 `@Subcomponent` 来实现 `AndroidInjector<YourActivity>` ，同时还有个 `@Subcomponent.Builder` 继承 `AndroidInjector.Builder<YourActivity> 。

   ```java
   @Subcomponent(modules = ...)
   public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {
     @Subcomponent.Builder
     public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
   }
   ```

3. 接着定义个 subcomponent，通过定义一个 module 去绑定 subcomponent builder 来添加到你的组件树，同时添加它到组件中，注入到 `Application` 中。

   ```java
   @Module(subcomponents = YourActivitySubcomponent.class)
   abstract class YourActivityModule {
     @Binds
     @IntoMap
     @ActivityKey(YourActivity.class)
     abstract AndroidInjector.Factory<? extends Activity>
         bindYourActivityInjectorFactory(YourActivitySubcomponent.Builder builder);
   }

   @Component(modules = {..., YourActivityModule.class})
   interface YourApplicationComponent {}
   ```

4. ​