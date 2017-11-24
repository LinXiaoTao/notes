[文档](https://developer.android.com/training/data-storage/room/index.html)



# 使用 Room 持久化库保存数据

Room 提供了 SQLite 的抽象层，来充分的使用 SQLite，并提供流畅的数据库访问。

Room 主要由以下三部分组成：

* `Database`：包含数据库持有者，并作为与应用持久关联数据的底层连接的主要访问点。

  使用 `@Database` 注解，并满足以下条件：

  * 继承于 `RoomDatabase` 的抽象类。
  * 在注解中，包含相关的 Entity 列表。
  * 包含一个没有的参数抽象方法，并返回使用 `@Dao` 标注的类。

  在运行时，你可以通过调用 `Room.databaseBuilder()` 或者 `Room.isMemoryDatabaseBuilder()` 获取一个 `Database` 的实例。

* `Entity`：表示数据库的表。

* `DAO`：包含用来访问数据库的方法。

![结构图](https://developer.android.com/images/training/data-storage/room_architecture.png)

``` java
@Entity
public class User {
    @PrimaryKey
    private int uid;

    @ColumnInfo(name = "first_name")
    private String firstName;

    @ColumnInfo(name = "last_name")
    private String lastName;

    // Getters and setters are ignored for brevity,
    // but they're required for Room to work.
}
```

``` java
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND "
           + "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert
    void insertAll(User... users);

    @Delete
    void delete(User user);
}
```

``` java
@Database(entities = {User.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

``` java
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();
```

> `RoomDatabase` 应该使用单例模式。