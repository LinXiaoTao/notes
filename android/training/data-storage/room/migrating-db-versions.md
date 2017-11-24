# Room 数据库迁移

当你在 APP 中添加或者修改功能，需要修改 Entity 来适应。Room 允许你编写 `Migration` 类来保留用户数据。每个 `Migration` 类指定 `startVersion` 和 `endVersion` 。在运行时，Room 会运行每个 `Migration` 类的 `migrate()` 方法，使用正确的顺序去迁移数据库到最新的版本。

> 如果你修改了 Entity，并且没有提供必需的 `Migration`，Room 将重建数据库。

``` java
Room.databaseBuilder(getApplicationContext(), MyDb.class, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build();

static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, "
                + "`name` TEXT, PRIMARY KEY(`id`))");
    }
};

static final Migration MIGRATION_2_3 = new Migration(2, 3) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE Book "
                + " ADD COLUMN pub_year INTEGER");
    }
};
```

> 要想保证迁移工作的预期进行，使用完整查询，而不是引用表示查询的常量。

在迁移处理结束后，Room 会验证数据库的 schema 确保迁移工作的正确处理。如果 Room 发现问题，将抛出包含不匹配信息的异常。



### 测试迁移

Room 提供了 Maven 库来进行测试处理。



### 导出 schemas

Room 导出你的数据库的 schema 信息到一个 JSON 文件中

``` groovy
android {
    ...
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation":
                             "$projectDir/schemas".toString()]
            }
        }
    }
}
```

你应该将导出的 JSON 文件保存到版本库中，这个文件能允许 Room 创建数据库的旧版本用于测试。

添加 `android.arch.persistence.room:testing` Maven 组件到你的测试依赖中，并且将存放 schema 路径作为 asset 文件夹。

``` groovy
android {
    ...
    sourceSets {
        androidTest.assets.srcDirs += files("$projectDir/schemas".toString())
    }
}
```

测试包提供了 `MigrationTestHelper` 用于读取 schema 文件。

``` java
@RunWith(AndroidJUnit4.class)
public class MigrationTest {
    private static final String TEST_DB = "migration-test";

    @Rule
    public MigrationTestHelper helper;

    public MigrationTest() {
        helper = new MigrationTestHelper(InstrumentationRegistry.getInstrumentation(),
                MigrationDb.class.getCanonicalName(),
                new FrameworkSQLiteOpenHelperFactory());
    }

    @Test
    public void migrate1To2() throws IOException {
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 1);

        // db has schema version 1. insert some data using SQL queries.
        // You cannot use DAO classes because they expect the latest schema.
        db.execSQL(...);

        // Prepare for the next version.
        db.close();

        // Re-open the database with version 2 and provide
        // MIGRATION_1_2 as the migration process.
        db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2);

        // MigrationTestHelper automatically verifies the schema changes,
        // but you need to validate that the data was migrated properly.
    }
}
```

