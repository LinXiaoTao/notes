[参考](https://developer.android.com/training/basics/data-storage/databases.html)

# 在 SQL 数据库中保存数据

对于重复或结构化数据而言，将数据保存到数据库是理想之选。



## 定义架构和契约

SQL 数据库的主要原则之一是架构：数据库如何组织的正式声明。架构体现于您用于创建数据库的 SQL 语句。

契约类是用于定义 URI、表格和列名称的常数的容器。契约类允许您跨同一软件包中的所有其他类使用相同的常数。

组织契约类的一种良好方法是将对于您的整个数据库而言是全局性的定义放入类的根级别。然后为枚举其列的每个表格创建内部类。

> **注意：**通过实现 [BaseCoumns](https://developer.android.com/reference/android/provider/BaseColumns.html) 接口，您的内部类可继承名为 `_ID` 的主键字段，某些 Android 类（比如光标适配器）将需要内部类拥有该字段。这并非必须项，但可帮助您的数据库与 Android 框架协调工作。

``` java
public final class FeedReaderContract {
    // To prevent someone from accidentally instantiating the contract class,
    // make the constructor private.
    private FeedReaderContract() {}

    /* Inner class that defines the table contents */
    public static class FeedEntry implements BaseColumns {
        public static final String TABLE_NAME = "entry";
        public static final String COLUMN_NAME_TITLE = "title";
        public static final String COLUMN_NAME_SUBTITLE = "subtitle";
    }
}
```



## 使用 SQL 辅助工具创建数据库

这里有一些典型的表格创建和删除语句：

``` java
private static final String TEXT_TYPE = " TEXT";
private static final String COMMA_SEP = ",";
private static final String SQL_CREATE_ENTRIES =
    "CREATE TABLE " + FeedEntry.TABLE_NAME + " (" +
    FeedEntry._ID + " INTEGER PRIMARY KEY," +
    FeedEntry.COLUMN_NAME_TITLE + TEXT_TYPE + COMMA_SEP +
    FeedEntry.COLUMN_NAME_SUBTITLE + TEXT_TYPE + " )";

private static final String SQL_DELETE_ENTRIES =
    "DROP TABLE IF EXISTS " + FeedEntry.TABLE_NAME;
```

Android 将您的数据库保存在私人磁盘空间，即关联的应用。您的数据是安全的，因为在默认情况下，其他应用无法访问此区域。

[SQLiteOpenHelper](https://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html) 类提供一组有用的 API。当您调用 `getWritableDatabase()` 和 `getReadableDatabase()` 来获取您数据库的引用时，系统将只在需要之时而不是应用启动时，来执行可能长期运行的操作：创建和更新数据库。

> **注意：**由于它们可能长期运行，因此请确保在后台线程中调用。

下面是使用 SQLiteOpenHelper 的实现例子：

``` java
public class FeedReaderDbHelper extends SQLiteOpenHelper {
    // If you change the database schema, you must increment the database version.
    public static final int DATABASE_VERSION = 1;
    public static final String DATABASE_NAME = "FeedReader.db";

    public FeedReaderDbHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(SQL_CREATE_ENTRIES);
    }
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // This database is only a cache for online data, so its upgrade policy is
        // to simply to discard the data and start over
        db.execSQL(SQL_DELETE_ENTRIES);
        onCreate(db);
    }
    public void onDowngrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        onUpgrade(db, oldVersion, newVersion);
    }
}
```



## 将信息输入到数据库

通过将一个 [ContentValues](https://developer.android.com/reference/android/content/ContentValues.html) 对象传递至 [insert()](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#insert(java.lang.String, java.lang.String, android.content.ContentValues)) 方法将数据插入数据库：

``` java
// Gets the data repository in write mode
SQLiteDatabase db = mDbHelper.getWritableDatabase();

// Create a new map of values, where column names are the keys
ContentValues values = new ContentValues();
values.put(FeedEntry.COLUMN_NAME_TITLE, title);
values.put(FeedEntry.COLUMN_NAME_SUBTITLE, subtitle);

// Insert the new row, returning the primary key value of the new row
long newRowId = db.insert(FeedEntry.TABLE_NAME, null, values);
```



## 从数据库读取信息

要从数据库中读取信息，请使用 [query()](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#query(boolean, java.lang.String, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String, java.lang.String, java.lang.String, java.lang.String)) 方法，将这个方法传递选择条件和所需列。查询的结果将在 [Cursor](https://developer.android.com/reference/android/database/Cursor.html) 对象中返回给您。

``` java
SQLiteDatabase db = mDbHelper.getReadableDatabase();

// Define a projection that specifies which columns from the database
// you will actually use after this query.
String[] projection = {
    FeedEntry._ID,
    FeedEntry.COLUMN_NAME_TITLE,
    FeedEntry.COLUMN_NAME_SUBTITLE
    };

// Filter results WHERE "title" = 'My Title'
String selection = FeedEntry.COLUMN_NAME_TITLE + " = ?";
String[] selectionArgs = { "My Title" };

// How you want the results sorted in the resulting Cursor
String sortOrder =
    FeedEntry.COLUMN_NAME_SUBTITLE + " DESC";

Cursor c = db.query(
    FeedEntry.TABLE_NAME,                     // The table to query
    projection,                               // The columns to return
    selection,                                // The columns for the WHERE clause
    selectionArgs,                            // The values for the WHERE clause
    null,                                     // don't group the rows
    null,                                     // don't filter by row groups
    sortOrder                                 // The sort order
    );
```

要查看游标中的某一行，请使用 `Cursor` 移动方法，您必须在开始读取值之前调用这些方法。一般情况下，您应调用 [moveToFirst](https://developer.android.com/reference/android/database/Cursor.html#moveToFirst()) 开始，这个方法将 **读取位置** 置于结果中的第一个条目中。对于每一行，您可以通过调用 `Cursor` 获取方法来读取列的值，比如 `getString()` 等。对于每种获取方法，您必须传递所需列的索引位置，可以通过调用 [getColumnIndex()](https://developer.android.com/reference/android/database/Cursor.html#getColumnIndex(java.lang.String)) 或 [getColumnIndexOrThrow()](https://developer.android.com/reference/android/database/Cursor.html#getColumnIndexOrThrow(java.lang.String)) 获取。例如：

``` java
cursor.moveToFirst();
long itemId = cursor.getLong(
    cursor.getColumnIndexOrThrow(FeedEntry._ID)
);
```



## 从数据库删除信息

要从表格中删除行，您需要提供识别行的选择条件。 数据库 API 提供了一种机制，用于创建防止 SQL 注入的选择条件。 该机制将选择规范划分为选择子句和选择参数。 该子句定义要查看的列，还允许您合并列测试。 参数是根据捆绑到子句的项进行测试的值。由于结果并未按照与常规 SQL 语句相同的方式进行处理，它不受 SQL 注入的影响。

``` java
// Define 'where' part of query.
String selection = FeedEntry.COLUMN_NAME_TITLE + " LIKE ?";
// Specify arguments in placeholder order.
String[] selectionArgs = { "MyTitle" };
// Issue SQL statement.
db.delete(FeedEntry.TABLE_NAME, selection, selectionArgs);
```



## 更新数据库

当您需要修改数据库值的子集时，请使用 [update()](https://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#update(java.lang.String, android.content.ContentValues, java.lang.String, java.lang.String[]))。

``` java
SQLiteDatabase db = mDbHelper.getReadableDatabase();

// New value for one column
ContentValues values = new ContentValues();
values.put(FeedEntry.COLUMN_NAME_TITLE, title);

// Which row to update, based on the title
String selection = FeedEntry.COLUMN_NAME_TITLE + " LIKE ?";
String[] selectionArgs = { "MyTitle" };

int count = db.update(
    FeedReaderDbHelper.FeedEntry.TABLE_NAME,
    values,
    selection,
    selectionArgs);
```

