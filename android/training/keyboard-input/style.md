[参考](https://developer.android.com/training/keyboard-input/style.html?hl=zh-cn#Type)

# 指定输入法类型

所有的文本字段都需要某种文本输入类型，比如电子邮件，手机号码，或者只是纯文本。所以为所有的文本字段指定输入类型很重要，这样系统会展示适当的输入法（比如一个屏幕键盘）。

除了制定输入法的按钮类型是否可用，还应该指定诸如，输入法是否提供拼写建议，首字母大小写以及使用 **完成** 或 **下一步** 等操作按钮来替换回车键的行为。



## 指定键盘类型

可以通过对 `EditText` 设置 `android:inputType` 属性指定输入类型

``` xml
<!-- 输入类型为手机号码 -->
android:inputType="phone" 
<!-- 输入类型为密码 -->
android:inputType="textPassword"
```



## 启用拼写建议和其他行为

``` xml
<!-- 启用拼写检查 -->
android:inputType="textAutoCorrect"
<!-- 启用拼写检查和首字母大写 -->
android:inputType="textCapSentences|textAutoCorrect"
```



## 指定输入法操作

大多数软键盘在底部提供了适合当前文本字段的用户操作按钮。

可以通过使用 `android:imeOptions` 属性来指定键盘操作按钮，比如 `actionSend` 或者 `actionSearch`。例如：

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

可以通过定义 `TextView.OnEditorActionListener` 来监听 `EditText` 的操作按钮。例如：

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



## 提供自动完成建议

如果要在用户输入时提供建议，可以使用 `EditText` 的子类 `AutoCompleteTextView`，要实现自动完成，指定一个提供文本建议的适配器。

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