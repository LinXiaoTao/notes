## 自定义属性

[参考](http://www.jianshu.com/p/61b79e7f88fc)



* attr 和 styleable 的关系

  attr 不依赖于 styleable，使用 styleable，在 R 文件上会生成属性的 id 数组，更方便操作。

  ``` java
  public static final int[] SlideTapeView = {
              0x7f0100fa, 0x7f0100fb, 0x7f0100fc, 0x7f0100fd,
              0x7f0100fe
          };
  ```

  不使用 styleable，获取 TypedArray 的操作是这样的：

  ``` java
  int[] custom_attrs = {R.attr.custom_attr1,R.custom_attr2};
  TypedArray typedArray = context.obtainStyledAttributes(set,custom_attrs);
  ```

  使用 styleable，则是这样的：

  ``` java
  TypedArray typedArray = context.obtainStyledAttributes(set,R.styleable.custom_attrs);
  ```

* TypedArray 的作用

  一般情况下，要获取某个属性的值，是这样的：

  ``` java
  TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.SlideTapeView, 0, 0);
  //获取 int 类型属性值
  typedArray.getInt(R.styleable.SlideTapeView_shortPointCount, 0)
  ```

  如果不通过 TypedArray 呢？当然也是可以获取的：

  ``` java
  for (int i = 0; i < attrs.getAttributeCount(); i++) {
  //以 string 类型获取属性值
  Log.d(TAG, "init: getAttributeValue" + attrs.getAttributeValue(i));
  //当属性值为 reference 时，获取 reference 的资源值
  Log.d(TAG, "init: getAttributeResourceValue " +attrs.getAttributeResourceValue(i,0));
  //getAttributeNameResource 获取属性在 R 文件定义的资源值
  Log.d(TAG, "init: getAttributeNameResource " + attrs.getAttributeNameResource(i));
  }
  ```

  通过这种方式获取的属性值，需要自己去判断属性值的格式，并且当属性值为 reference（引用时）,返回打印的属性值为：

  ``` java
   Log.d(TAG, "init: getAttributeValue" + attrs.getAttributeValue(i));
  //init: getAttributeValue @2131558517
  //还需要去获取实际的值
  getResources().getString(attrs.getAttributeValue(i));
  ```

  所以，使用 TypeArray 可以简化获取属性值的操作。

* `TypeArray obtainStyledAttributes(AttributeSet set, @StyleableRes int[] attrs, @AttrRes int defStyleAttr,@StyleRes int defStyleRes)`

  `defStyleAttr`：表示默认的 style 属性

  举个例子：

  attrs 文件定义：

  ``` xml
  <attr name="custom_style" format="reference"></attr>
  ```

  styles 中定义：

  ``` xml
  <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        //配置style
      <item name="custom_style">@style/custom_theme</item>
    </style>
      //主题中配置的style
    <style name="custom_theme">
      <item name="custom_color1">#ff222222</item>
      <item name="custom_color2">#ff222222</item>
      <item name="custom_color3">#ff222222</item>
    </style>
  ```

  ``` java
  final TypedArray a = context.obtainStyledAttributes(
              set, R.styleable.custom_attrs, defStyle, 0);
  ```

  这样，就会从我们定义的 style 属性中去获取属性值了

  `defStyleRes`：表示默认 style

  还是上面那个例子，在 attrs 文件中：

  ``` xml
  <style name="default_style">
      <item name="custom_color1">#ff333333</item>
      <item name="custom_color2">#ff333333</item>
      <item name="custom_color3">#ff333333</item>
      <item name="custom_color4">#ff333333</item>
  </style>
  ```

  ``` java
  final TypedArray a = context.obtainStyledAttributes(
              set, R.styleable.custom_attrs, defStyle, R.style.default_style);
  ```

  这样，不仅会从定义的 style 属性中获取属性值，同时还会从默认 style 中去获取。

  总结：在自定义控件中，我们获取属性值的方式有以下（优先级从上到下）：

  1. 从 AttributeSet 获取的所有属性值，比如直接在 XML 中定义属性值
  2. AttributeSet 中使用 style 属性定义的属性值，比如在 XML 中引用一个 style 。
  3. 通过 defStyleAttr 和 defStyleRes 定义的默认属性值
  4. Theme（主题） 中定义的属性值

  > defStyleRes 只有在 defStyleAttr 没有值的时候才会生效，即当 defStyleAttr 指定，并且存在值，defStyleRes 不起作用。