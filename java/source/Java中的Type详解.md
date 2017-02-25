# Java中的Type详解

---

[参考地址](http://loveshisong.cn/%E7%BC%96%E7%A8%8B%E6%8A%80%E6%9C%AF/2016-02-16-Type%E8%AF%A6%E8%A7%A3.html)

Type相关接口:
![type](http://loveshisong.cn/static/images/reflect_interface.png)

* Type
  `Type`是所有类型的父接口,子接口有`WildcardType`,`GenericArrayType`,`TypeVariable<D>`和`ParameterizedType`,实现类有`class`
* ParameterizedType
  表示一个参数化类型,比如`Collection<String>`.有如下方法:
  1. `Type getRawType()`:返回承载该泛型信息的对象
  2. `Type[] getActualTypeArguments()`:返回实际泛型类型列表
  3. `Type getOwnerType()`:返回这个类型的归属者.比如内部类,则返回外部类
  ``` java
  public class TestModel {
    public Map.Entry<String,Integer> data;
  }
  @Test
    public void testParameterizedType(){
    try {
        Field field = TestModel.class.getDeclaredField("data");
        System.out.println(field);
        System.out.println(field.getGenericType());
        ParameterizedType parameterizedType = (ParameterizedType) field.getGenericType();
        System.out.println(Arrays.toString(parameterizedType.getActualTypeArguments()));//String,Integer
        System.out.println(parameterizedType.getOwnerType());//Map
        System.out.println(parameterizedType.getRawType());//Map$Entry
    } catch (NoSuchFieldException e) {
        e.printStackTrace();
    }
    
    }
  ```
* TypeVariable
  类型变量,泛型信息在编译时会被转换为一个特定的类型.而`TypeVariable`就是用来反映在JVM编译该泛型前的信息.有如下方法:
  1. `Type[] getBounds()`:返回类型变量的上边界,若为明确声明上边界,则默认为`Object`
  2. `String getName()`:返回类型变量名称,即泛型名
  3. `D getGenericDeclaration()`:返回声明该类型变量实体
  4. `AnnotatedType[] getAnnotatedBounds()`

  ``` java
   public class TestModel<T extends TestModel,Y> {
       public T model;
       public Y object;
   }
   @Test
    public void testTypeVariable(){
        try {
            Field model = TestModel.class.getDeclaredField("model");
            Field object = TestModel.class.getDeclaredField("object");
            TypeVariable modelType = (TypeVariable) model.getGenericType();
            TypeVariable objectType = (TypeVariable) object.getGenericType();
            System.out.println(modelType.getName());//T
            System.out.println(objectType.getName());//Y
            System.out.println(Arrays.toString(modelType.getBounds()));//TestModel
            System.out.println(Arrays.toString(objectType.getBounds()));//Object
            System.out.println(modelType.getGenericDeclaration());//TestModel
            System.out.println(objectType.getGenericDeclaration());//TestModel
            System.out.println(Arrays.toString(modelType.getAnnotatedBounds()));//AnnotatedTypeFactory$AnnotatedTypeBaseImpl
            System.out.println(Arrays.toString(objectType.getAnnotatedBounds()));//AnnotatedTypeFactory$AnnotatedTypeBaseImpl
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
```
* GenericArrayType
泛型数组,组成数组的元素中有泛型则实现了该接口.它的组成元素为`ParameterizedType`和`TypeVariable`类型.它只有一个方法:
  1. `Type getGenericComponentType()`:返回数组的组成对象,即被JVM编译后的实际对象
  
  ``` java
  public class TestModel<T extends TestModel,Y> {
    public T[] mArrayT;
    public List<String>[] mArraysList;
}
@Test
    public void testGenericArrayType(){
        try {
            Field arraysT = TestModel.class.getDeclaredField("mArrayT");
            Field arraysList = TestModel.class.getDeclaredField("mArraysList");
            System.out.println(arraysT.getGenericType());
            System.out.println(arraysList.getGenericType());
            Assert.assertTrue(IsInstanceOf.instanceOf(GenericArrayType.class).matches(arraysT.getGenericType()));
            Assert.assertTrue(IsInstanceOf.instanceOf(GenericArrayType.class).matches(arraysList.getGenericType()));
            GenericArrayType arraysType = (GenericArrayType) arraysT.getGenericType();//T
            GenericArrayType arraysListType = (GenericArrayType) arraysList.getGenericType();//List<String>
            System.out.println(arraysType.getGenericComponentType());
            System.out.println(arraysListType.getGenericComponentType());
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }

    }
  ```
* WildcardType
  该接口表示通配符泛型.比如`? extends Number`和`? super Integer`.它有如下方法:
  1. `Type[] getUpperBounds()`:获取泛型变量的上边界
  2. `Type[] getLowerBounds()`:获取泛型变量的下边界

  ``` java
  public class TestModel<T extends TestModel,Y> {
  public List<? extends Number> mNumbers;
  public List<? super String> mObjects;
  }
  @Test
    public void testWildcardType(){
        try {
            Field numbers = TestModel.class.getDeclaredField("mNumbers");
            Field objects = TestModel.class.getDeclaredField("mObjects");
            ParameterizedType numbersType = (ParameterizedType) numbers.getGenericType();
            ParameterizedType objectsType = (ParameterizedType) objects.getGenericType();
            Type[] numbersArguments = numbersType.getActualTypeArguments();
            Type[] objectsArguments = objectsType.getActualTypeArguments();
            WildcardType numberArgumentsType = (WildcardType) numbersArguments[0];
            WildcardType objectArgumentsType = (WildcardType) objectsArguments[0];
            System.out.println("number");
            System.out.println(Arrays.toString(numberArgumentsType.getLowerBounds()));//
            System.out.println(Arrays.toString(numberArgumentsType.getUpperBounds()));//Number
            System.out.println("object");
            System.out.println(Arrays.toString(objectArgumentsType.getLowerBounds()));//String
            System.out.println(Arrays.toString(objectArgumentsType.getUpperBounds()));//Object
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }
  ```
* Type及其子接口的来历
  * 泛型出现之前的类型
    没有泛型的时候,只有原始类型.此时,所有的原始类型都通过字节码文件Class类进行抽象.Class类的一个具体对象就代表一个指定的原始类型
  * 泛型出现之后
    泛型出现之后,扩充了数据类型.从只有原始类型扩充了参数化类型,类型变量类型,通配符类型,泛型数组类型.
          
      



