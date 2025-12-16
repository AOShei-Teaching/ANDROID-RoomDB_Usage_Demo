# 🗄️ Room Database Demo

A simple Android application demonstrating how to implement a local SQLite database using the [Room Persistence Library](https://developer.android.com/training/data-storage/room).

This project serves as a practical example for students and developers learning how to perform CRUD (Create, Read) operations, manage database entities, and observe data changes in real-time using Kotlin Coroutines and Flows.

---

## 🚀 Features

* **Data Persistence:** Saves user data (First Name, Last Name, Age) locally on the device using SQLite via Room.
* **Real-time Updates:** Uses `Flow` to automatically update the UI whenever data in the database changes.
* **Structured Data:** Defines a clear schema using Room Entities.
* **Safe Database Access:** Uses the Singleton pattern to ensure only one database instance exists.
* **Coroutines:** Performs database operations off the main thread to keep the UI responsive.

---

## 🛠 Tech Stack

* **Language:** Kotlin
* **Database:** Room (SQLite abstraction layer)
* **Concurrency:** Kotlin Coroutines & Flow
* **UI:** XML Layouts (TableLayout for displaying records)
* **Architecture:** DAO (Data Access Object) pattern

---

## 🧑‍💻 How It Works (Code Highlights)

### 1. The Data Entity (`Person.kt`)
The database schema is defined using data classes annotated with `@Entity`.
* **`@Entity`:** Marks the class as a database table named "persons".
* **`@PrimaryKey`:** Auto-generates unique IDs for each new entry.

```kotlin
@Entity(tableName = "persons")
data class Person(
    @PrimaryKey(autoGenerate = true)
    val id: Int = 0,
    val firstName: String,
    val lastName: String,
    val age: Int
)

```

###2. The DAO (`PersonDao.kt`)The **Data Access Object (DAO)** defines how we interact with the database. We use interfaces instead of writing raw SQL queries manually for every operation.

* **`@Insert`:** Inserts a new row.
* **`@Query`:** returns a `Flow<List<Person>>`. This is powerful because it emits a new list every time the database changes, allowing the UI to react instantly.

```kotlin
@Dao
interface PersonDao {
    @Query("SELECT * FROM persons ORDER BY id DESC")
    fun getAllPersons(): Flow<List<Person>>

    @Insert
    suspend fun insertPerson(person: Person)
}

```

###3. The Database Singleton (`PersonDatabase.kt`)We use the **Singleton pattern** to create the database. This prevents having multiple expensive connections to the database open at the same time.

* `@Database`: Lists all entities and the version number.
* `getDatabase()`: Returns the single instance of the database, creating it if it doesn't exist yet.

###4. Observing Data (`MainActivity.kt`)In the main activity, we observe the `Flow` from the DAO.

* `collect`:  A coroutine collects the stream of data. whenever a new person is added, `updateTable` is automatically called to refresh the UI.

```kotlin
lifecycleScope.launch {
    database.personDao().getAllPersons().collect { persons ->
        updateTable(persons)
    }
}

```

---

##📱 How to Run
1. **Clone the repository** and open it in **Android Studio**.
2. **Sync Gradle** to ensure Room dependencies are downloaded.
3. **Run the app** on an emulator or physical device.

**Test:**
  * Enter a First Name, Last Name, and Age.
  * Click **Submit**.
  * Watch the table below automatically update with the new entry!
  * Restart the app—your data will still be there (persistence!).

---

##⚠️ Troubleshooting
* **"Unresolved reference: Room"**: Make sure your `build.gradle` file includes the necessary Room dependencies and the `ksp` (or `kapt`) plugin.
* **Schema Build Errors**: If you change the `Person` class (e.g., add a field) after running the app, you must increment the `version` number in `PersonDatabase.kt` or uninstall the app from the emulator to reset the database.

