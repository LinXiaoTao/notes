

* `@Scope`  用于表明注入实例的作用范围，在代码中的体现就是，变量的作用范围。例如：外部类变量，内部类变量。


* 按照注入作用来区分的话，可以分为 **提供注入器** 和 **提供注入实例** 两种。

  * 使用 `@ContributesAndroidInjector` 注解标示的，或者使用 `AndroidInjector`

    是用来提供 **注入器**。

    ``` java
    //使用 @ContributesAndroidInjector
    @ActivityScope
    @ContributesAndroidInjector(modules = { /* modules to install into the subcomponent */ })
    abstract YourActivity contributeYourActivityInjector();

    //使用 AndroidInjector
    @Subcomponent(modules = ...)
    public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {
      @Subcomponent.Builder
      public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
    }

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

  * 通常使用 `@Provides`，`@Binds` 或者 `@Inject` 构造方法等等，属于提供 **注入实例**。

  ​

  严格来说，其实不管是注入实例或者是注入器，本质上都是提供可用于注入的实例对象，但有点区别是注入器一般是用于 `Activity` ，或者 `Application` 这些不能从构造方法中去实现注入的类，在 `android.dagger` 之前，是通过提供具体 `Activity` 的 Component，在运行中去注入所需的实例。`dagger.android` 则是为了简化这个操作，通过下面这行代码就可以实现注入：

  ```java
  @Override
  protected void onCreate(@Nullable Bundle savedInstanceState) {
    AndroidInjection.inject(this);
    super.onCreate(savedInstanceState);
  }
  ```

  这就引出来了一个问题，我们调用 `AndroidInjection.inject(Activity)` ，传入的参数是 `Activity`，我们要如何得到正确的 `Activity` 子类，来实现注入，这就是注入器的作用，注入器的类名为 `AndroidInjector` ，它只有一个方法 `inject(T)` ，这个方法就是来对具体类的成员变量进行注入。
  首先，我们需要声明一个注入器：

  ``` java
  @Subcomponent(modules = ...)
  public interface YourActivitySubcomponent extends AndroidInjector<YourActivity> {

  @Subcomponent.Builder
  public abstract class Builder extends AndroidInjector.Builder<YourActivity> {}
  }
  ```

    然后，将注入器和具体的类绑定，使用 `@IntoMap` 会存放到 Map 中

  ``` java
  @Module(subcomponents = YourActivitySubcomponent.class)
  abstract class YourActivityModule {
  @Binds
  @IntoMap
  @ActivityKey(YourActivity.class)
  abstract AndroidInjector.Factory<? extends Activity>
      bindYourActivityInjectorFactory(YourActivitySubcomponent.Builder builder);
  }
  ```

    最后，在更高一级的 Parent Component 中注册，在这里是 Application：

  ``` java
  @Component(modules = {..., YourActivityModule.class})
  interface YourApplicationComponent {}
  ```

    这样就可以在 Activity 实现注入之前，由 Application Component 提供好对应关系，在 Activity 的 `onCreate()` 中，通过 `instance.getClass()` 获取对应的注入器，来实现注入。

    > 低级的 Component 可以继承高级 Component 的对应关系，比如 Activity 中获取对应关系，需要由 Application 提供，如果 Fragment 的对应关系是在 Activity 中注册的，则由 Activity 提供，如果在 Application 中提供的，则 Activity 继承 Application 的对应关系，同样可以从 Activity 中获取。

* 使用 `@Scope` 可以保证范围内的实例是单例，可以复用。