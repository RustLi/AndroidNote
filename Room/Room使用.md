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
~~~

### 三要素

* Dao：增删改查操作

* Entity：实体类，一个实体对应数据库中的一个表

* Database：抽象类，继承RoomDatabase，编译的时候实现Dao方法

* 三者关系图

  

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

   

   