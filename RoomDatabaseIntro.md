# Room Overview

**What is Room?**

Room is an abstraction layer built on top of SQLite that enhances database access while maintaining SQLite's full capabilities. It simplifies database interactions by reducing boilerplate code compared to direct SQLite usage, making it easier for Android developers to work with databases.

---

**Why Use Room?**

Before Room's introduction in 2017, developers had to write repetitive and error-prone code to define table structures, create SQL queries, and manage database migrations. This manual process often led to crashes due to invalid queries and lacked compile-time verification.

---

**Advantages of Room:**

1. **Compile-Time Verification of SQL Queries**: 
   - Room checks queries and entities at compile time, helping to catch errors like missing tables early and preventing runtime crashes.
   *(Room checks your database commands while you write your code. This means it can catch mistakes (like forgetting to create a table) before you run your app, reducing the chances of crashes.)*

2. **Reduced Boilerplate Code**: 
   - It minimizes the boilerplate associated with SQLite, such as cursor management and the manual conversion of SQL results to Java objects (POJOs).
   *(It handles a lot of the messy details for you, making it easier to get your app up and running.)*


3. **Integration with Modern Android Architectures**: 
   - Room works well with components like LiveData, Kotlin Flows, and RxJava, facilitating reactive programming and modern app design.
   *(Room easily connects with other tools in Android, like LiveData and Kotlin Flows, which help you manage data updates smoothly.)*


---

## Room Components Skeleton

# Entity in Room

## What is an Entity?

An Entity in Room represents a table in the database. Each instance of an Entity corresponds to a row in the table, and each field in the Entity class corresponds to a column in that table. Entities are used to define the structure of the data you want to store.

---

## Key Components of an Entity

### 1. Annotations

Entities use annotations to define their properties:

- `@Entity`: Marks the class as a Room entity. You can specify the table name and other properties.
- `@PrimaryKey`: Indicates the primary key for the table. This field uniquely identifies each row.
- `@ColumnInfo`: Specifies additional details about a column, such as its name or default value.

### 2. Data Types

Entities can use standard Kotlin data types (e.g., `String`, `Int`, `Boolean`) for their properties, and Room will handle the conversion to the corresponding SQLite types.

---
```kotlin
import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "person_table")
data class Person(
    @PrimaryKey(autoGenerate = true) val id: Int = 0,
    val name: String,
    val age: Int,
    val city: String
)

```

# Data Access Object (DAO) 

## What is a DAO?

A Data Access Object (DAO) is an interface that provides methods for interacting with the Room database. It defines how you can perform operations like inserting, updating, deleting, and querying data. By using a DAO, you can keep your database code organized and separate from your application logic.

---

## Key Components of a DAO

### 1. Annotations

DAOs use various annotations to define the database operations:

- `@Dao`: Indicates that this interface is a DAO.
- `@Insert`: Used to insert data into the database.
- `@Update`: Used to update existing data.
- `@Delete`: Used to delete data from the database.
- `@Query`: Used to define custom SQL queries.

### 2. Coroutine Support

To perform database operations without blocking the main thread, DAOs can use Kotlin Coroutines or RxJava. This allows for asynchronous operations, enhancing app performance.

---

## Example DAO Implementation

Below is an example of a DAO for a `Person` entity:

```kotlin
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import kotlinx.coroutines.flow.Flow

@Dao
interface PersonDao {

    // Insert a new Person object into the database
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun savePerson(person: Person)

    // Retrieve all Person objects as a Flow
    @Query("SELECT * FROM person_table")
    fun getAllData(): LiveData<List<Person>>

    // Delete a Person from the database by ID
    @Query("DELETE FROM person_table WHERE id = :personId")
    suspend fun deletePerson(personId: Int)

    // Find a Person by name
    @Query("SELECT * FROM person_table WHERE name = :name")
    suspend fun findPersonByName(name: String): Person?
}

```

# Database in Room

## What is a Database?

In Room, the Database component represents the main access point to your application's SQLite database. It is responsible for creating and managing the database and providing instances of the DAOs for data operations.

---

## Key Components of a Database

### 1. Annotations

The Database class uses annotations to define its properties:

- `@Database`: Marks the class as a Room database. You specify the entities that belong to this database and the version number.
  
### 2. Singleton Pattern

To ensure that only one instance of the database is created throughout the application, a singleton pattern is typically used. This helps to manage database access efficiently.

---

## Example Database Implementation

Below is an example of a Room database for a `Person` entity:

```kotlin
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import android.content.Context

@Database(entities = [Person::class], version = 1, exportSchema = false)
abstract class AppDatabase : RoomDatabase() {

    // Define the DAO for the Person entity
    abstract fun personDao(): PersonDao

    // Singleton instance of the database
    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "app_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}

```

# Repository in Room

## What is a Repository?

In the context of Room and Android architecture, a Repository acts as a mediator between the data sources (such as the Room database, network services, or other data sources) and the rest of the application. It abstracts the data operations, providing a clean API for data access to the ViewModel and UI components.

---

## Key Responsibilities of a Repository

1. **Data Management**: Handles data retrieval, storage, and manipulation from various sources (local database, remote API, etc.).
  
2. **Data Transformation**: Converts data into a format that is easier for the UI to consume, often mapping data models to UI models.

3. **Error Handling**: Manages errors from data operations and provides fallback mechanisms if needed.

4. **Abstraction**: Provides a clean interface for data access, allowing the rest of the application to work with a consistent API regardless of where the data comes from.

---

## Example Repository Implementation

Below is an example of a Repository for the `Person` entity:

```kotlin
import kotlinx.coroutines.flow.Flow

class PersonRepository(private val personDao: PersonDao) {

    // Retrieve all persons as a Flow
    val allPersons: LiveData<List<Person>> = personDao.getAllData()

    // Insert a new person into the database
    suspend fun insert(person: Person) {
        personDao.savePerson(person)
    }

    // Delete a person by ID
    suspend fun delete(personId: Int) {
        personDao.deletePerson(personId)
    }

    // Find a person by name
    suspend fun findByName(name: String): Person? {
        return personDao.findPersonByName(name)
    }
}

```
