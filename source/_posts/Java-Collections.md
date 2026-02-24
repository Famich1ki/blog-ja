---
title: Summary of Java Collections Principles and Underlying Data Structures
date: 2024-08-25 23:05:58
tags: 
  - Java
  - Java Collections
categories:
  - [Java SE, Collections]
cover: https://pics.findfuns.org/java-collections.png
---

# First, a Collection Hierarchy Diagram

```html
<img src="https://pics.findfuns.org/Collections-hierarchy.png" alt="collectionsHierarchy" style="zoom:50%;" />
```

# List

The main characteristics of a `List` are **ordering** and **allowing duplicate elements**.

## `ArrayList`

`ArrayList` allows any element, including `null`. Compared with `Vector`, it is **not thread-safe**. If necessary, it can be wrapped with `Collections.synchronizedList()` to make it thread-safe:

```java
List list = Collections.synchronizedList(new ArrayList());
```

Internally, it maintains an `Object` array to store elements:

```java
transient Object[] elementData;
```

It defines a `grow()` method. When calling a series of overloaded `add` methods, if the current size reaches the `capacity`, `grow()` will be invoked to ensure sufficient capacity.

---

## `PriorityQueue`

`PriorityQueue` maintains a **min-heap** internally, ensuring that the head element is always the smallest element in the queue. “Smallest” depends on either the element’s `Comparator` or `Comparable` implementation.

It also provides a constructor that accepts a custom `Comparator`:

```java
public PriorityQueue(Comparator<? super E> comparator)
```

Internal fields:

```java
transient Object[] queue;

int size;

private final Comparator<? super E> comparator;

transient int modCount;
```

`modCount` records the number of structural modifications (such as adding, removing, or reordering elements). If an iterator is created and `modCount` changes during iteration, a `ConcurrentModificationException` will be thrown.

`PriorityQueue` does **not** allow `null` elements.

### Time Complexity

- Inserting and removing the head element: `O(log n)`
- Searching or removing a specific element: `O(n)`

### Insertion

- If capacity is exceeded:
    - Requires `O(n)` to copy the array into a larger one, plus `O(1)` insertion → overall `O(n)`
- If capacity is not exceeded:
    - Insert at tail: `O(1)`
    - Insert elsewhere: `O(n)`

### Deletion

- Remove tail: `O(1)`
- Remove from other positions: `O(n)`

---

## `RandomAccess`

`ArrayList` implements the `RandomAccess` interface, indicating that it supports random access (constant-time index-based access). Since it is backed by an array, accessing elements by index is `O(1)`.

---

## `LinkedList`

`LinkedList` maintains a **doubly linked list** internally:

```java
transient int size = 0;

transient Node<E> first;

transient Node<E> last;
```

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

Each `LinkedList` contains two references: one to the head node and one to the tail node.

### RandomAccess

`LinkedList` does **not** support random access. Searching requires linear traversal with time complexity `O(n)`, unlike arrays which support `O(1)` access.

---

### Methods

There are two main method groups for insertion and deletion:

1. `add` and `remove`
    - `addFirst()`, `addLast()`
    - `removeFirst()`, `removeLast()`

2. `offer` and `poll`
    - `offerFirst()`, `offerLast()`
    - `pollFirst()`, `pollLast()`

The key difference:
- `remove` throws an exception if the list is empty.
- `poll` returns `false` (or `null`) instead of throwing an exception.

In fact, many `offer` methods internally call the corresponding `add` methods:

```java
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```

### Time Complexity

- Insert/remove at head or tail: `O(1)`
- Insert/remove elsewhere: `O(n)`

---

# Queue

A `Queue` follows the FIFO (First In First Out) principle, making it suitable for ordered processing scenarios.

Under `Queue`, there is a sub-interface called `Deque` (double-ended queue), which supports operations at both ends and can simulate a stack.

Both `ArrayDeque` and `LinkedList` can implement the `Queue` interface. However, `ArrayDeque` does **not** allow `null` values.

When simulating a stack, `ArrayDeque` is more efficient than `Stack`. When used as a queue, it is also generally more efficient than `LinkedList`.

---

## `ArrayDeque`

`ArrayDeque` maintains a **circular array** internally, which enables double-ended operations.

Except for `remove`-related methods and `contains`, most operations run in constant time.

```java
transient Object[] elements;

transient int head;

transient int tail;
```

When adding elements, only the head or tail pointer needs to be adjusted.

---

# Set

The main use case of `Set` is **removing duplicates**.

---

## `HashSet`

Internally, `HashSet` maintains a `HashMap`. All inserted elements become keys in the map, and their values are set to a constant object:

```java
private static final Object PRESENT = new Object();
```

When calling `add()`:

```java
public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

---

## `TreeSet`

`TreeSet` is backed by a **red-black tree**, enabling automatic sorting of elements.

### `Comparator` and `Comparable`

Sorting depends on either:
- The element implementing `Comparable`
- Providing a `Comparator` to the constructor

```java
TreeSet(Comparator<? super E> comparator)
```

If elements do not implement `Comparable`, and no `Comparator` is provided, inserting elements into a `TreeSet` will cause a `ClassCastException`.

If a `Comparator` is provided in the constructor, it overrides any `Comparable` implementation.

---

### Example

```java
class Person implements Comparable<Person> {
    private String name;
    private Integer age;

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Person o) {
        if (this == o) {
            return 0;
        }

        int ageComparison = this.age.compareTo(o.age);
        if (ageComparison != 0) {
            return ageComparison;
        }

        return this.name.compareTo(o.name);
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

---

# Map

`Map` stores data in key-value pairs (`<key, value>`).

Before Java 8, `HashMap` used an **array + linked list** structure. Each array slot (bucket) could store multiple elements via a linked list in case of hash collisions.

After Java 8, when the linked list length exceeds a threshold (default 8), it is converted into a red-black tree, improving lookup efficiency from `O(n)` to `O(log n)`.

---

## `HashMap`

### `null` Handling

- One `null` key allowed
- Multiple `null` values allowed

### Thread Safety

`HashMap` is **not thread-safe**. In concurrent scenarios:

```java
Map m = Collections.synchronizedMap(new HashMap<>());
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
```

---

### Iteration Order

`HashMap` does not guarantee iteration order. Rehashing or structural modifications may change the order.

---

### `initial capacity` and `load factor`

Constructors:

```java
HashMap()
HashMap(int initialCapacity)
HashMap(int initialCapacity, float loadFactor)
HashMap(Map<? extends K,? extends V> m)
```

Load factor formula:

```
Load Factor (α) = Number of Elements / Capacity of Hash Table
```

Default load factor is 0.75.  
When size reaches 75% of capacity, rehashing occurs and capacity doubles.

Average time complexity for `get()` and `put()` is `O(1)`.

---

### Internal Structure

```java
transient Node<K,V>[] table;
```

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;
}
```

---

## `LinkedHashMap`

`LinkedHashMap` extends `HashMap` and adds a **doubly linked list** to maintain iteration order.

### `accessOrder`

```java
LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder)
```

- `false` → insertion order
- `true` → access order (useful for LRU cache)

---

### LRU Example

```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final Integer capacity;

    public LRUCache(Integer capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return this.size() > capacity;
    }
}
```

`removeEldestEntry` is triggered during `put()`. If it returns `true`, the eldest entry is removed.

`LinkedHashMap` is also **not thread-safe** and should be wrapped with `Collections.synchronizedMap()` in concurrent environments.
