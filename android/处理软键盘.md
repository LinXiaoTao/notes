### 软键盘

当你的界面中文本字段(比如文本输入框)获取到焦点时，Android系统显示屏幕上的键盘，称为**软键盘**。

### 指定输入法类型

* 指定键盘类型

  可以通过对`EditText`设置`android:inputType`属性指定输入类型

  ``` xml
  <!-- 输入类型为手机号码 -->
  android:inputType="phone" 
  <!-- 输入类型为密码 -->
  android:inputType="textPassword"
  ```

* 启用拼写建议和其他行为

  ``` xml
  <!-- 启用拼写检查 -->
  android:inputType="textAutoCorrect"
  <!-- 启用拼写检查和首字母大写 -->
  android:inputType="textCapSentences|textAutoCorrect"
  ```

* 指定输入法操作

  大多数软键盘在底部提供了适合当前文本字段的用户操作按钮。

  可以通过使用`android:imeOptions`属性来指定键盘操作按钮，比如`actionSend`或者`actionSearch`。例如：

  ``` xml
  <!-- 要指定inputType -->
  <EditText
      android:id="@+id/search"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content"
      android:hint="@string/search_hint"
      android:inputType="text"
      android:imeOptions="actionSend" />
  ```

  ![img](https://developer.android.google.cn/images/ui/edittext-actionsend.png)

  可以通过定义`TextView.OnEditorActionListener`来监听`EditText`的操作按钮。例如：

  ``` java
  EditText editText = (EditText) findViewById(R.id.search);
  editText.setOnEditorActionListener(new OnEditorActionListener() {
      @Override
      public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
          boolean handled = false;
          if (actionId == EditorInfo.IME_ACTION_SEND) {
              sendMessage();
              handled = true;
          }
          return handled;
      }
  });
  ```

* 提供自动完成建议

  如果要在用户输入时提供建议，可以使用`EditText`的子类`AutoCompleteTextView`，要实现自动完成，指定一个提供文本建议的适配器。

  ``` xml
  <?xml version="1.0" encoding="utf-8"?>
  <AutoCompleteTextView xmlns:android="http://schemas.android.com/apk/res/android"
      android:id="@+id/autocomplete_country"
      android:layout_width="fill_parent"
      android:layout_height="wrap_content" />
  ```

  ``` xml
  <?xml version="1.0" encoding="utf-8"?>
  <resources>
      <string-array name="countries_array">
          <item>Afghanistan</item>
          <item>Albania</item>
          <item>Algeria</item>
          <item>American Samoa</item>
          <item>Andorra</item>
          <item>Angola</item>
          <item>Anguilla</item>
          <item>Antarctica</item>
          ...
      </string-array>
  </resources>
  ```

  ``` java
  AutoCompleteTextView textView = (AutoCompleteTextView) findViewById(R.id.autocomplete_country);
  String[] countries = getResources().getStringArray(R.array.countries_array);
  ArrayAdapter<String> adapter =
          new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, countries);
  textView.setAdapter(adapter);
  ```

### 处理输入法可见性

当输入焦点移入或移出可编辑文本字段时，Android会根据需要显示或隐藏输入法（如屏幕键盘）。

* 当Activity开始时显示输入法

  ```xml
  <application ... >
      <activity
          android:windowSoftInputMode="stateVisible" ... >
          ...
      </activity>
      ...
  </application>
  ```

* 在需要的时候显示输入法

  ```java
  public void showSoftKeyboard(View view) {
      if (view.requestFocus()) {
          InputMethodManager imm = (InputMethodManager)
                  getSystemService(Context.INPUT_METHOD_SERVICE);
          imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
      }
  }
  ```

* 指定界面如何响应输入法

  当屏幕出现输入法时，会减少UI界面的可用空间。

  使用`adjustResize`，系统将布局调整到可用空间，确保所有布局内容都可见

  ```xml
  <application ... >
      <activity
          android:windowSoftInputMode="adjustResize" ... >
          ...
      </activity>
      ...
  </application>
  ```

  可以和输入法可见性结合使用

  ```xml
  <activity
          android:windowSoftInputMode="stateVisible|adjustResize" ... >
          ...
      </activity>
  ```

### 支持键盘导航

键盘不仅提供了方便的文本输入模式，而且还为用户提供导航和与您的应用互动的方式。

* 处理使用TAB导航

  当用户使用键盘Tab键导航您的应用程序时，系统会根据它们在布局中显示的顺序，在元素之间传递输入焦点。

  可以使用`android:nextFocusForward`来指定确认获取焦点的顺序

* 处理定向导航(使用方向键或轨迹球等等)

  可以使用以下属性定义定向导航焦点顺序

  * `android:nextFocusUp`
  * `android:nextFocusDown`
  * `android:nextFocusLeft`
  * `android:nextFocusRight`

### 处理键盘操作

可以使用`KeyEvent.Callback`接口来拦截或直接处理键盘(**物理键盘包括虚拟键，不包括软键盘**)输入。

Activity和View类都实现了KeyEvent.Callback接口，所以你一般应该覆盖这些类的扩展中的回调方法。

**注意：当你使用`KeyEvent`类和有关API来处理键盘事件，你应该知道所有事件只来源硬件键盘**

* 处理单个按键

  通常，如果要确保您只收到一个事件，您应该使用onKeyUp（）。如果用户按住按钮，那么onKeyDown（）被多次调用。

  ```java
  @Override
  public boolean onKeyUp(int keyCode, KeyEvent event) {
      switch (keyCode) {
          case KeyEvent.KEYCODE_D:
              moveShip(MOVE_LEFT);
              return true;
          case KeyEvent.KEYCODE_F:
              moveShip(MOVE_RIGHT);
              return true;
          case KeyEvent.KEYCODE_J:
              fireMachineGun();
              return true;
          case KeyEvent.KEYCODE_K:
              fireMissile();
              return true;
          default:
              return super.onKeyUp(keyCode, event);
      }
  }
  ```

* 处理修饰按键

  几种方法提供有关修饰键的信息，如getModifiers（）和getMetaState（）。然而，最简单的解决方法是检查您关心的确切修饰符键是否被诸如isShiftPressed（）和isCtrlPressed（）等等。

  ```java
  @Override
  public boolean onKeyUp(int keyCode, KeyEvent event) {
      switch (keyCode) {
          ...
          case KeyEvent.KEYCODE_J:
              if (event.isShiftPressed()) {
                  fireLaser();
              } else {
                  fireMachineGun();
              }
              return true;
          case KeyEvent.KEYCODE_K:
              if (event.isShiftPressed()) {
                  fireSeekingMissle();
              } else {
                  fireMissile();
              }
              return true;
          default:
              return super.onKeyUp(keyCode, event);
      }
  }
  ```

  ​