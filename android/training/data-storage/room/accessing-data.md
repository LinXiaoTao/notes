# 使用 DAO 访问数据

通过使用 DAO 访问数据库数据，而不是通过查询构造器或者直接查询语句，这样可以分离数据库组件的不同模块。还可以在测试代码中很容易的 mock 数据库访问。

DAO 可以是一个接口或者抽象类。如果你使用抽象类，你应该使用只有 `RoomDatabase` 作为唯一参数的构造方法。Room 会在编译时创建每个 DAO 的实现类。

> Room 不支持在主线程中访问数据库，除非使用了 `allowMainThreadQueries()` 。



## 定义便利方法

### Insert

使用 `@Insert` 会将方法中的参数，在单次事务中插入的数据库中。

``` java
@Dao
public interface MyDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void insertUsers(User... users);

    @Insert
    public void insertBothUsers(User user1, User user2);

    @Insert
    public void insertUsersAndFriends(User user, List<User> friends);
}

```

如果参数只有一个，那么方法可以返回 `long` 类型值，表示当前插入的 rowId。如果参数是数组或者集合，方法应该返回 `long[]` 或 `List<Long` 值。



### Update

使用 `@Update` 会根据参数中的 Entity 的主键去匹配，从而替换为新的 Entity。

``` java
@Dao
public interface MyDao {
    @Update
    public void updateUsers(User... users);
}
```

可以通过返回 `int` 值，来表示更新的 rowId。



### Delete

使用 `@Delete` 会根据参数中的 Entity 的主键去匹配，从而删除 Entity。

``` java
@Dao
public interface MyDao {
    @Delete
    public void deleteUsers(User... users);
}
```



### Query

`@Query` 是 DAO 中的主要注解。它允许你对数据库进行 读/写 操作。每个 `@Query` 方法会在编译时进行验证，如果存在问题，会发生编译错误。

Room 还会验证查询的返回值，如果出现返回值的字段和查询列的名称不符合，Room 会用以下两种方式提示你：

* 如果只有一些字段名称匹配，将会产生一个警告
* 如果没有字段匹配，将会产生一个错误



### 简单查询

``` java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user")
    public User[] loadAllUsers();
}
```

如果查询包含语法错误或者 user 表不存在于数据库，Room 将会在编译时显示一个错误提示。



### 传递参数到查询语句中

``` java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge")
    public User[] loadAllUsersOlderThan(int minAge);
}
```

Room 将通过参数名进行匹配，如果没有匹配成功，将在编译时产生一个错误。

你可以传递多个参数或者在查询语句中使用多次。

``` java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
    public User[] loadAllUsersBetweenAges(int minAge, int maxAge);

    @Query("SELECT * FROM user WHERE first_name LIKE :search "
           + "OR last_name LIKE :search")
    public List<User> findUserWithName(String search);
}

```



### 返回列的子集

多数情况下，你只需要获取 Entity 中的部分字段。Room 允许你从查询中返回任何基于 Java 的对象，只要这个 Java 对象能和列信息对应上。

``` java
public class NameTuple {
    @ColumnInfo(name="first_name")
    public String firstName;

    @ColumnInfo(name="last_name")
    public String lastName;
}
```

> 这些基于 Java 的对象 POJO 也可以使用 `@Embedded`。



### 使用集合参数

当你需要使用不确定长度的参数时，可以使用集合参数

``` java
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
}

```



### Observable 查询

可以使用 `LiveData` 作为查询方法的返回值，当数据库更新时，Room 会生成所有必需的代码去更新 `LiveData`。

``` java
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public LiveData<List<User>> loadUsersFromRegionsSync(List<String> regions);
}
```

> 在版本 1.0 中，Room 使用查询中的表的列数来判断是否更新 `LiveData` 实例。



### 使用 RxJava 检索查询

Room 也可以从查询中返回 RxJava2 的 `Publisher` 和 `Flowable` 对象。要使用这个功能，添加 `android.arch.persistence.room:rxjava2` 。

``` java
@Dao
public interface MyDao {
    @Query("SELECT * from user where id = :id LIMIT 1")
    public Flowable<User> loadUserById(int id);
}
```



### 直接 cursor 访问

可以从查询中返回 `Cursor` 对象。

``` java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
    public Cursor loadRawUsersOlderThan(int minAge);
}
```

> 不推荐直接使用 `Cursor` 



### 查询多表

Room 允许你编写任何正确的查询语句，所有你可以使用 join 。

``` java
@Dao
public interface MyDao {
    @Query("SELECT * FROM book "
           + "INNER JOIN loan ON loan.book_id = book.id "
           + "INNER JOIN user ON user.id = loan.user_id "
           + "WHERE user.name LIKE :userName")
   public List<Book> findBooksBorrowedByNameSync(String userName);
}
```

同样可以在返回值中使用 POJO 对象，来获取不同表的列数。

``` java
@Dao
public interface MyDao {
   @Query("SELECT user.name AS userName, pet.name AS petName "
          + "FROM user, pet "
          + "WHERE user.id = pet.user_id")
   public LiveData<List<UserPet>> loadUserAndPetNames();

   // You can also define this class in a separate file, as long as you add the
   // "public" access modifier.
   static class UserPet {
       public String userName;
       public String petName;
   }
}
```

