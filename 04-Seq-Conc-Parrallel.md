# Concurrency vs Parallelism in Android (Kotlin Coroutines)

This example demonstrates the difference between **sequential execution**, **concurrent execution**, and **parallel execution** in Kotlin Coroutines.  

---

## 🔹 Code Example

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

fun main() = runBlocking {
    println("CPU cores: ${Runtime.getRuntime().availableProcessors()}")

    // Simulate a CPU heavy task
    suspend fun heavyTask(id: Int): Int {
        println("Task $id started on ${Thread.currentThread().name}")
        var sum = 0L
        for (i in 1..2_000_000) { sum += i } // heavy loop
        println("Task $id finished on ${Thread.currentThread().name}")
        return sum.toInt()
    }

    // ---------------------------
    // 1. Sequential Execution
    // ---------------------------
    val timeSequential = measureTimeMillis {
        val result1 = heavyTask(1)
        val result2 = heavyTask(2)
    }
    println("Sequential execution took $timeSequential ms")

    // ---------------------------
    // 2. Concurrency (IO Dispatcher)
    // ---------------------------
    val timeConcurrentIO = measureTimeMillis {
        val d1 = async(Dispatchers.IO) { heavyTask(1) }
        val d2 = async(Dispatchers.IO) { heavyTask(2) }
        d1.await()
        d2.await()
    }
    println("Concurrent (IO) took $timeConcurrentIO ms")

    // ---------------------------
    // 3. Parallelism (Default Dispatcher)
    // ---------------------------
    val timeParallel = measureTimeMillis {
        val d1 = async(Dispatchers.Default) { heavyTask(1) }
        val d2 = async(Dispatchers.Default) { heavyTask(2) }
        d1.await()
        d2.await()
    }
    println("Parallel (Default) took $timeParallel ms")
}
```

---

## 🔹 Explanation

1. **Sequential Execution**  
   - Runs one task after another on the same thread.  
   - Time ≈ `T1 + T2`.  

2. **Concurrent Execution (IO Dispatcher)**  
   - Tasks are launched concurrently on `Dispatchers.IO`.  
   - May still run sequentially on the same thread, depending on scheduling.  
   - Can sometimes be slower due to **thread switching overhead**, since `IO` dispatcher is not optimized for CPU-heavy work.  

3. **Parallel Execution (Default Dispatcher)**  
   - Optimized for CPU-bound work.  
   - Tasks are distributed across CPU cores (e.g., worker-1, worker-2).  
   - Time ≈ `max(T1, T2)`.  

---

## 🔹 Key Takeaways

- Use **`Dispatchers.IO`** for I/O-bound tasks (network calls, DB queries, file I/O).  
- Use **`Dispatchers.Default`** for CPU-bound tasks (loops, computation, image processing).  
- Concurrency does **not always** mean parallelism.  

