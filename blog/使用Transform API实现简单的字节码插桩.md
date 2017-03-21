**Transform API**是从Gradle1.5.0-beta1开始新增的Api，作用在于可以允许在class文件转换成dex时候，进行操作。[具体说明](http://tools.android.com/tech-docs/new-build-system/transform-api)。

在这里例子中，我们用Transform API实现了一个简单的方法计时功能，就是在方法进入的地方记录开始时间，方法结束的地方计算耗时。

首先我们新建一个Gradle 插件，在`apply`方法中去注册一个`Transform`来操作class文件

``` java
public class AspectPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        def android = project.extensions.getByType(AppExtension)
        android.registerTransform(new AspectTransform())
    }
}
```

关于Gradle 插件的实现方式有几种，在这里我们使用比较方便的`buildSrc`，Gradle会自动识别这个文件夹。

在`Transform`中，我们使用`ASM`去操作生成新的class文件，关于`ASM`，大家可以自行搜索，在这里我们只要知道，它可以对java字节码进行操作。主要代码如下，`ClassReader`，`ClassWriter`都是`ASM`的类，可以用来访问和输出class文件。具体逻辑代码可以查看github。

``` java
private void transformFile(File file) {
        if (file.absolutePath.contains("R.class") || file.absolutePath.contains('BuildConfig.class')
                || file.absolutePath.contains("R\$"))
            return
        println "输入文件路径:" + file.absolutePath
        ClassReader classReader = new ClassReader(new FileInputStream(file))
        ClassWriter classWriter = new ClassWriter(classReader, ClassWriter.COMPUTE_MAXS)
        MonitorClassVisitor monitorClassVisitor = new MonitorClassVisitor(Opcodes.ASM5, classWriter)
        classReader.accept(monitorClassVisitor, ClassReader.EXPAND_FRAMES)
        FileOutputStream fileOutputStream = new FileOutputStream(file)
        fileOutputStream.write(classWriter.toByteArray())
    }
```

使用方式：在需要计算的类和方法前增加注解`@Monitor`，例子如下：

``` java
@Monitor
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);


        test();

    }

    @Monitor
    private void test(){
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

这就是大概的逻辑，具体可以参考github代码。

其实除了Transform API，我们还有另外一种方式去实现操作类文件，就是使用Java的Agent机制，它分别是Java SE5新增的**premain**和Java SE6的**agentmain**，具体使用方式可以自定搜索。那这两种方式又应该如何用在android上呢，这里只说思路：