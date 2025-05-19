# RateLimiter – Thread-safe Singleton

This Kotlin-based RateLimiter class provides per-user request limiting logic in a multi-threaded environment. It ensures that a user can only make a defined number of requests within a fixed time window (e.g., 60 seconds).

---

## Features

- Thread-safe singleton using `@Volatile` and `synchronized`
- Per-user request counting and timestamp tracking
- Safe for use in multi-threaded environments
- Uses `ConcurrentHashMap` for concurrent access

---

## Class Definition
```kotlin
class RateLimiter private constructor(private val maxLimit: Int) {

    private val userTimestamps = ConcurrentHashMap<Any, Long>()
    private val userRequestCounts = ConcurrentHashMap<Any, Int>()

    @Synchronized
    fun isRequestAllowed(userIdentifier: Any): Boolean {
        val currentTime = System.currentTimeMillis()
        val firstRequestTime = userTimestamps[userIdentifier]
        return if (firstRequestTime == null || currentTime - firstRequestTime > 60000) {
            userTimestamps.put(userIdentifier, System.currentTimeMillis())
            userRequestCounts.put(userIdentifier, 1)
            true
        } else {
            val requestCount = userRequestCounts[userIdentifier] ?: 0
            if (requestCount < maxLimit) {
                userRequestCounts.put(userIdentifier, requestCount + 1)
                true
            } else {
                false
            }
        }
    }

    companion object {
        @Volatile
        private var INSTANCE: RateLimiter? = null

        fun init(maxLimit: Int) {
            synchronized(this) {
                if (INSTANCE == null) {
                    INSTANCE = RateLimiter(maxLimit)
                }
            }
        }

        fun getInstance(): RateLimiter {
            return INSTANCE ?: throw IllegalStateException("RateLimiter not initialized. Call init() first.")
        }
    }
}
```

---

## Usage
```kotlin
fun main() {
    RateLimiter.init(maxLimit = 20)
    val thread1 = thread {
        repeat(11) {
            val allowed = RateLimiter.getInstance().isRequestAllowed("user1")
            println("Thread 1 - Request #${it + 1}: $allowed")
        }
    }

    val thread2 = thread {
        repeat(11) {
            val allowed = RateLimiter.getInstance().isRequestAllowed("user1")
            println("Thread 2 - Request #${it + 1}: $allowed")
        }
    }

    thread1.join()
    thread2.join()
}
```
