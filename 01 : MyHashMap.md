# MyHashMap<K, V> – Custom Hash Map in Kotlin

This is a simple implementation of a generic hash map (`MyHashMap<K, V>`) using **separate chaining with linked lists** to handle collisions. It supports basic operations like `put`, `get`, `remove`, `size` and automatic resizing when the load factor exceeds a threshold.

---

## Features

- Generic key-value storage
- Collision handling using `LinkedList`
- Automatic resizing based on load factor
- Basic map operations via a `MapInterface` interface

---

## Interface Definition

```kotlin
interface MapInterface<K, V> {
    fun put(key: K, value: V?)
    fun get(key: K): V?
    fun remove(key: K)
    fun size(): Int
}
```

## Class Definition
```kotlin
import kotlin.math.abs

class MyHashMap<K, V> : MapInterface<K, V> {
    inner class Entry<K, V>(var key: K, var value: V)

    private var capacity = 4
    private val loadFactor = 0.75f
    private var size = 0
    private var buckets = MutableList(capacity) { LinkedList<Entry<K, V?>>() }

    override fun put(key: K, value: V?) {
        val index = getBucketIndex(key)

        for (entry in buckets[index]) {
            if (entry.key?.equals(key) == true) {
                entry.value = value
                return
            }
        }

        buckets[index].add(Entry(key, value))
        size++
        if (size.toFloat() / capacity > loadFactor) {
            resize()
        }
    }

    override fun get(key: K): V? {
        val index = getBucketIndex(key)
        for (entry in buckets[index]) {
            if (entry.key?.equals(key) == true) {
                return entry.value
            }
        }
        return null
    }

    override fun remove(key: K) {
        val index = getBucketIndex(key)
        val iterator = buckets[index].iterator()
        while (iterator.hasNext()) {
            if (iterator.next().key?.equals(key) == true) {
                iterator.remove()
                size--
                break
            }
        }
    }

    override fun size(): Int = size

    private fun getBucketIndex(key: K): Int {
        return abs(key.hashCode()) % capacity
    }

    private fun resize() {
        val oldBuckets = buckets
        capacity *= 2
        buckets = MutableList(capacity) { LinkedList() }
        size = 0
        for (bucket in oldBuckets) {
            for (entry in bucket) {
                put(entry.key, entry.value)
            }
        }
    }
}
