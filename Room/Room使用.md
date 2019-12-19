## Room

Room提供对SQLite的抽象层，用于数据库操作。

### 依赖

~~~java
dependencies {
    def room_version = "2.1.0"

    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version" // For Kotlin use kapt instead of annotationProcessor
}
~~~

maver仓库

~~~java
allprojects {
    repositories {
        jcenter()
        maven {
            url 'https://maven.google.com'
        }
    }
}

or

allprojects {
    repositories {
        google()
        jcenter()
    }
}
~~~

### 三要素

* Dao：增删改查操作

* Entity：实体类，一个实体对应数据库中的一个表

* Database：抽象类，继承RoomDatabase，编译的时候实现Dao方法

* 三者关系图

  ![img](https://github.com/RustLi/JetpackLearning/blob/master/Room/room_architecture.png?raw=true)

### 使用

1. Dao

   ~~~java
   @Dao
   public interface UserDao {
   
       @Query("select * from room_user_test")
       List<User> getAll();
   
       @Query("select * from room_user_test where :userName")
       List<User> getUserByName(String userName);
   
       @Insert()
       void insertAll(User user);
   
       @Insert()
       void insertList(List<User> userEntities);
   
   }
   ~~~

2. Entity

   ~~~java
   @Entity(tableName = "room_user_test")
   public class User {
       @PrimaryKey
       private int uid;
   
       @ColumnInfo(name = "user_name")
       private String userName;
   
       @ColumnInfo(name = "gender")
       private String gender;
   
       @ColumnInfo(name = "age")
       private int age;
   ~~~

   

3. Database

   ~~~java
   @Database(entities = {User.class}, version = 1)
   public abstract class UserDataBase extends RoomDatabase {
       private static DataBase mInstance = null;
       public abstract UserDao getUserDao();
       
       public static UserDataBase getInMemoryDatabase(Context context){
           if (mInstance == null){
               mInstance = Room.databaseBuilder(context.getApplicationContext(),
                       UserDataBase.class, "RoomTest")
                       .build();
           }
   
           return mInstance;
       }
   }
   ~~~

   

4. 调用

   ~~~java
   private UserDao getUserDao(){
           if (userDao == null){
               userDao = UserDataBase.getInMemoryDatabase(this).getUserDao();
           }
           return userDao;
       }
   
   getUserDao().insertAll(new User(index,"jack","male",30));
   ~~~

### 数据库迁移

说明：使用Room的时候，如果你改变了数据库的schema但是没有更新version，app将会crash。而如果你更新了version但是没有提供迁移，数据库的表就会drop掉，用户将丢失数据。

~~~java
public static DataBase getInMemoryDatabase(Context context){
        if (mInstance == null){
            mInstance = Room.databaseBuilder(context.getApplicationContext(),
                    DataBase.class, "DeepCamFace")
                    .addMigrations(migration2_3, migration3_4)
                    .allowMainThreadQueries()
                    .fallbackToDestructiveMigration()
                    .build();
        }

        return mInstance;
    }

private static final Migration migration2_3 = new Migration(2,3) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            database.execSQL("ALTER TABLE visitorrecord ADD COLUMN timeStamp TEXT");
        }
    };

    private static final Migration migration3_4 = new Migration(3,4) {
        @Override
        public void migrate(@NonNull SupportSQLiteDatabase database) {
            database.execSQL("ALTER TABLE Face ADD COLUMN vipGroupName TEXT");
        }
    };
~~~

### 导出

build.gradle

~~~java
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
~~~

### 嵌套

采用@Embedded注解，将类中的变量都作为表中的字段。

~~~java
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
~~~

### 常见操作

* 插入

  ~~~java
  @Dao
  public interface MyDao {
      @Insert(onConflict = OnConflictStrategy.REPLACE)
      public void insertUsers(User... users);
   
      @Insert
      public void insertBothUsers(User user1, User user2);
   
      @Insert
      public void insertUsersAndFriends(User user, List<User> friends);
  }
  ~~~

* 更新

  ~~~java
  @Dao
  public interface MyDao {
      @Update
      public void updateUsers(User... users);
  }
  ~~~

* 删除

  ~~~java
  @Dao
  public interface MyDao {
      @Delete
      public void deleteUsers(User... users);
  }
  ~~~

  
