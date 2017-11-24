[文档](https://developer.android.com/training/data-storage/room/defining-data.html)



# 使用 Room Entity 定义数据

默认 Room 会为 Entity 的每一个字段创建列，如果实体中包含不想创建列的字段，可以使用 `@Ignore` 忽略。

``` java
@Entity
class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String lastName;

    @Ignore
    Bitmap picture;
}
```



对于能创建列的每一个字段，Room 必须能访问它。你可以将字段设置为 `public` 或者提供 get/set 方法。

> Entity 可以使用空构造方法（如果 DAO 类能访问所有能创建列的字段）或者参数的类型和名称于字段相对应的构造方法。



## 使用主键

每个 Entity 必须拥有一个使用 `@PrimaryKey` 定义的主键，即使当前 Entity 只有一个字段。如果你想要 Room 自动为 Entity 分配 ID，你可以使用 `@PrimaryKey` 的 `autoGenerate` 属性。如果你的实体拥有复合主键，你可以使用 `@Entity` 的 `primaryKeys` 属性。

``` java
@Entity(primaryKeys = {"firstName", "lastName"})
class User {
    public String firstName;
    public String lastName;

    @Ignore
    Bitmap picture;
}
```

默认，Room 使用类名作为数据库的表名。如果你想要表使用其他名称，可以设置 `@Entity` 的 `tableName` 属性。

``` java
@Entity(tableName = "users")
class User {
    ...
}
```

> SQLite 的表名是不区分大小写

和表名类似，Room 默认使用字段名作为表中的列名。如果你想要列使用不同的名称，可以添加 `@ColumnInfo` 在字段上。

```java
@Entity(tableName = "users")
class User {
    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}

```



## 索引和唯一性

使用 `@Entity` 的 `indices` 属性设置索引。

``` java
@Entity(indices = {@Index("name"),
        @Index(value = {"last_name", "address"})})
class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String address;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
```

可以通过设置`@Index` 的 `unique` 属性来设置唯一性。

``` java
@Entity(indices = {@Index(value = {"first_name", "last_name"},
        unique = true)})
class User {
    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
```



## 定义对象之间的关系

> [**Room 不允许 Entity 之间相互引用**](https://developer.android.com/training/data-storage/room/referencing-data.html#understand-no-object-references)

虽然你不可以在 Entity 之间使用直接关系，但可以仍然可以使用外键（`@ForeignKey`）来定义 Entity 之间的关系。

``` java
@Entity(foreignKeys = @ForeignKey(entity = User.class,
                                  parentColumns = "id",
                                  childColumns = "user_id"))
class Book {
    @PrimaryKey
    public int bookId;

    public String title;

    @ColumnInfo(name = "user_id")
    public int userId;
}
```

外键是非常有用的，比如它允许你定义当间接引用的 Entity 更新时进行的操作。比如，你可以通过在 `@ForeignKey` 中包含 `onDelete = CASCADE` 来执行当间接引用的 Entity 被删除时，相关联的 Entity 也被删除。

> SQLite 处理 `@Insert(onConflict = REPLACE)` 是作为 **REMOVE** 和 **REPLACE** 操作，而不是单个 **UPDATE** 操作。这个替换冲突值的方法可能会影响你的外键约束



## 创建嵌套对象

你可以使用 `@Embedded` 来拆分表中字段到不同的对象。

``` java
class Address {
    public String street;
    public String state;
    public String city;

    @ColumnInfo(name = "post_code")
    public int postCode;
}

@Entity
class User {
    @PrimaryKey
    public int id;

    public String firstName;

    @Embedded
    public Address address;
}
```

> 嵌入对象字段也可以包含其他嵌入对象字段。

如果多个嵌入对象包含相同的字段名，可以通过使用 `prefix` 属性来设置不同的字段前缀。