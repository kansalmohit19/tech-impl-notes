# Singleton Pattern in Kotlin — Double Check

This example demonstrates **how to implement and verify a Singleton** in Kotlin

---

## Code Example

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

// -------------------------------------------
// 1️⃣ Thread-safe Singleton (Double-Checked Locking)
// -------------------------------------------

class Database private constructor() {

    init {
        println("Database instance created on ${Thread.currentThread().name}")
    }

    companion object {
        @Volatile
        private var INSTANCE: Database? = null

        fun getInstance(): Database {
            // First check (no locking) for performance
            val temp = INSTANCE
            if (temp != null) return temp

            synchronized(this) {
                val again = INSTANCE
                if (again != null) return again

                val instance = Database()
                INSTANCE = instance
                return instance
            }
        }
    }
}

// -------------------------------------------
// 2️⃣ Verification
// -------------------------------------------

fun main() = runBlocking {
    println("CPU cores: ${Runtime.getRuntime().availableProcessors()}\n")

    // Sequential call check
    println("---- Sequential Singleton Check ----")
    val db1 = Database.getInstance()
    val db2 = Database.getInstance()
    println("Same instance (sequential)? ${db1 === db2}\n")

    // Concurrent access check
    println("---- Concurrent Singleton Check ----")
    val time = measureTimeMillis {
        val jobs = (1..10).map {
            async(Dispatchers.Default) {
                val instance = Database.getInstance()
                instance.hashCode()
            }
        }
        jobs.awaitAll()
    }
    println("\nConcurrent initialization completed in $time ms")

    println("\n---- Using Basic Singleton (object) ----")
}
```
---

## Explanation
1. **Manual Singleton (Double-Checked Locking)**
    - Needed if you must control creation manually (e.g., dependency injection, configuration).
    - Uses a @Volatile variable and synchronized block.
    - Ensures that even if multiple threads call getInstance() simultaneously, only one instance is created.

2. **Verification**
    - The program runs getInstance() sequentially and then concurrently.
    - You’ll see "Database instance created..." printed only once, proving that it’s a true singleton.
    - The hashCode for all concurrent calls remains identical.

## Key Takeaways
- Use object for simple singletons — Kotlin handles thread safety automatically.
- Use double-checked locking when you need custom initialization logic.
- Use @Volatile to ensure visibility of the instance across threads.
- Always test singletons with concurrent access to validate thread safety.
