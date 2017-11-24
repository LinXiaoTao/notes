[文档](https://developer.android.com/training/data-storage/room/referencing-data.html)



# 使用 Room 引用复杂的数据结构

Room 提供了基本类型和包装类之间的转化功能，但不允许 Entity 之间的相互引用。



## 使用类型转换

有时候你想将某种自定义数据，存放到数据库表中的单行。可以通过提供 `TypeConverter` 来支持自定义类型。

``` java
public class Converters {
    @TypeConverter
    public static Date fromTimestamp(Long value) {
        return value == null ? null : new Date(value);
    }

    @TypeConverter
    public static Long dateToTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}
```

然后你需要将 `@TypeConverters` 注解添加到 `RoomDatabase` 中，以使用这些转换器。

``` java
@Database(entities = {User.class}, version = 1)
@TypeConverters({Converters.class})
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

使用类型转换器，你还可以在查询语句中使用自定义类型，就像使用基本类型一样。

``` java
@Entity
public class User {
    ...
    private Date birthday;
}

@Dao
public interface UserDao {
    ...
    @Query("SELECT * FROM user WHERE birthday BETWEEN :from AND :to")
    List<User> findUsersBornBetweenDates(Date from, Date to);
}
```

你还可以将 `@TypeConverters` 限制在不同的范围内。



## 理解为什么 Room 不允许对象引用

> Room 不允许 Entity 之间相互引用，相反，你必须明确的请求 APP 所需要的数据

将数据库中的关系映射到相应的对象模型是一种常见的做法。但是，在客户端，这种延迟加载是不可行的，因为它通常发生在 UI 线程上，并且在 UI 线程中查询磁盘上的信息会造成显着的性能问题。但是，如果您不使用延迟加载，则应用程序将获取比所需更多的数据，从而导致内存消耗问题。

对象关系映射通常将这个决定留给开发人员，以便他们可以做任何最适合他们的应用程序的用例。开发人员通常决定在应用程序和用户界面之间共享模型。但是，这种解决方案并不能很好地扩展，因为随着 UI 的变化，共享模型会产生难以让开发人员预测和调试的问题。

例如，考虑加载一个 `Book` 对象列表的 UI，每个书都有一个 `Author` 对象。您可能最初设计您的查询使用延迟加载，以便 `Book` 的实例使用 `getAuthor()` 方法返回作者。`getAuthor() `调用的第一个调用将查询数据库。一段时间后，你意识到你也需要在应用的用户界面中显示作者的名字。您可以轻松地添加方法调用，如以下代码片段所示：

``` java
authorNameTextView.setText(user.getAuthor().getName());
```

然而，这个操作会导致查询在主线程中进行。

如果提前查询作者信息，如果不再需要这些数据，则很难更改数据的加载方式。例如，如果您的应用程序的用户界面不再需要显示作者信息，您的应用程序将有效地加载不再显示的数据，浪费宝贵的内存空间。如果作者类引用另一个表（如Books），则应用程序的效率会进一步降低。

要使用 Room 同时引用多个 Entity，您需要创建一个包含每个 Entity 的 POJO，然后编写一个连接相应表的查询。这种结构良好的模型与 Room 健壮的查询验证功能相结合，可让您的应用程序在加载数据时消耗更少的资源，从而改善应用程序的性能和用户体验。