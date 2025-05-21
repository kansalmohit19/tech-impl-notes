# LRU Cache

This Kotlin-based **LRU Cache** keeps track of the most recently used items. When the cache exceeds its capacity, it removes the **least recently used** item. **LRU Cache** keeps track of the most recently used items. When the cache exceeds its capacity, it removes the **least recently used** item.

---

## Features

- Lookup (`get`) and insert (`put`) operations are both **O(1)**.
- Internally uses:
  - **HashMap** for quick access
  - **Doubly Linked List** to maintain access order

---

## Class Definition
```kotlin
class LRUCache<K, V> {
    private inner class Node(val key: K?, var value: V?) {
        var prev: Node? = null
        var next: Node? = null
    }

    private var map = mutableMapOf<K, Node>()
    private var head = Node(null, null)
    private var tail = Node(null, null)
    private val capacity = 2

    init {
        head.next = tail
        tail.prev = head
    }

    fun put(key: K, value: V) {
        if (map.containsKey(key)) {
            val node = map[key]
            node?.value = value
            moveToFront(node)
        } else {
            if (map.size == capacity) {
                evictLRU()
            }

            val newNode = Node(key, value)
            addToFront(newNode)
            map[key] = newNode
        }
    }

    fun get(key: K): V? {
        return map.getOrDefault(key, null)?.let { node ->
            moveToFront(node)
            node.value
        } ?: run {
            null
        }
    }

    fun printCache() {
        var current = head.next
        print("Cache [MRU → LRU]: ")
        while (current != null && current != tail) {
            print("${current.key}=${current.value} ")
            current = current.next
        }
        println()
    }

    private fun evictLRU() {
        tail.prev?.let {
            remove(it)
            map.remove(it.key)
        }
    }

    private fun moveToFront(node: Node?) {
        remove(node)
        addToFront(node)
    }

    private fun remove(node: Node?) {
        node?.let {
            node.prev?.next = node.next
            node.next?.prev = node.prev
        }
    }

    private fun addToFront(node: Node?) {
        node?.let {
            node.prev = head
            node.next = head.next
            head.next?.prev = node
            head.next = node
        }
    }
}
```

---

## Usage
```kotlin
fun main() {
    val obj = LRUCache<String, String>()
    obj.put("1", "Name-1")
    obj.printCache()
    obj.put("2", "Name-2")
    obj.printCache()
    obj.get("1")
    obj.printCache()
    obj.put("2", "Name-3")
    obj.printCache()
}
```
