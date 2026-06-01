# Java Collections

## What is a "collection"?

A **collection** is a data structure - a set of certain objects. The data (objects in the set) can be numbers, strings, user-defined class objects, etc.

---

## Interfaces in the JCF (Java Collections Framework) and their implementations

At the top of the hierarchy in the Java Collections Framework there are two interfaces: `Collection` and `Map`. These divide all collections into two parts based on data storage type: simple sequential sets of elements, and key-value pairs, respectively.

The `Collection` interface is extended by:

- **`List`** - allows duplicate values. Implementations:
    - `ArrayList` - encapsulates a regular array whose length automatically increases when new elements are added. Elements are numbered from zero and can be accessed by index.
    - `LinkedList` - a doubly linked list; each node contains data and references to the next and previous nodes.
    - `Vector` - a dynamic array of objects with synchronized methods.
    - `Stack` - a LIFO (last-in-first-out) stack implementation.

- **`Set`** - an unordered collection with no duplicate elements. Implementations:
    - `HashSet` - uses a `HashMap` internally; the added element is the key, a placeholder `Object` is the value. Element order is not guaranteed.
    - `LinkedHashSet` - iteration order matches insertion order.
    - `TreeSet` - elements are ordered using a `Comparator` or natural ordering.

- **`Queue`** - stores elements with FIFO (first-in-first-out) retrieval. Implementations:
    - `PriorityQueue` - orders elements using a `Comparator` or natural ordering.
    - `ArrayDeque` - implements `Deque`, which extends `Queue` with LIFO support.

The `Map` interface is implemented by:

- `Hashtable` - synchronized hash table; does not allow `null` keys or values; not ordered.
- `HashMap` - hash table allowing `null` keys and values; not ordered.
- `LinkedHashMap` - ordered hash table; iteration order matches insertion order.
- `TreeMap` - red-black tree; ordered by natural ordering or a `Comparator`.
- `WeakHashMap` - hash table using weak references for keys; the GC automatically removes entries when no strong references to the key remain.

---

## Place the following interfaces in hierarchical order

`List`, `Set`, `Map`, `SortedSet`, `SortedMap`, `Collection`, `Iterable`, `Iterator`, `NavigableSet`, `NavigableMap`

```
Iterable
└── Collection
    ├── List
    └── Set
        └── SortedSet
            └── NavigableSet
Map
└── SortedMap
    └── NavigableMap
Iterator
```

---

## Why is `Map` not a `Collection` whereas `List` and `Set` are?

A `Collection` is a set of individual elements. A `Map` is a set of key-value pairs - a fundamentally different concept.

---

## What is the difference between `java.util.Collection` and `java.util.Collections`?

- `java.util.Collection` is one of the core **interfaces** of the Java Collections Framework.
- `java.util.Collections` is a **utility class** providing static methods for working with collections (sorting, searching, synchronizing, etc.).

---

## What is "fail-fast behavior"?

Fail-fast behavior means that when an error (or a condition likely to lead to an error) occurs, the system immediately stops execution and reports it.

In the Java Collections API, some iterators exhibit fail-fast behavior and throw `ConcurrentModificationException` if the collection is structurally modified after the iterator is created (i.e., an element is added or removed directly on the collection rather than via iterator methods).

This is implemented by tracking a **modification count**:
1. When the collection is modified, its modification counter is incremented.
2. When an iterator is created, it captures the current count.
3. On each iterator access, the captured count is compared to the current one. A mismatch throws `ConcurrentModificationException`.

---

## What is the difference between fail-fast and fail-safe?

Fail-safe iterators do **not** throw exceptions when the collection's structure is modified because they operate on a **clone** of the collection rather than the original.

Examples of fail-safe iterators:
- `CopyOnWriteArrayList`'s iterator
- The `keySet()` view iterator of `ConcurrentHashMap`

---

## What is the difference between `Enumeration` and `Iterator`?

Both interfaces iterate over collections but differ significantly:

- `Enumeration` does not support adding or removing elements; `Iterator` does (via `remove()`).
- `Iterator` has more readable method names: `hasNext()` / `next()` vs. `hasMoreElements()` / `nextElement()`.
- `Enumeration` is found in legacy classes (`Vector`, `Stack`); `Iterator` is available in all modern collections.

---

## How are `Iterable` and `Iterator` related?

The `Iterable` interface has a single method, `iterator()`, which returns an `Iterator`.

---

## How are `Iterable`, `Iterator`, and for-each related?

Classes that implement `Iterable` can be used in a `for-each` loop. Internally, the for-each loop uses an `Iterator`.

---

## Compare `Iterator` and `ListIterator`

- `ListIterator` extends `Iterator`.
- `ListIterator` can only be used with `List` collections.
- `Iterator` traverses in one direction (`next()`); `ListIterator` traverses in both directions (`next()` and `previous()`).
- `ListIterator` does not point to a specific element - its position is *between* the elements returned by `previous()` and `next()`.
- `ListIterator` supports modification via `add()` and `remove()`; plain `Iterator` only supports `remove()`.

---

## What happens if `Iterator.next()` is called without first calling `Iterator.hasNext()`?

If the iterator is positioned at the last element, a `NoSuchElementException` is thrown. Otherwise, the next element is returned normally.

---

## How many elements are skipped if `Iterator.next()` is called after ten calls to `Iterator.hasNext()`?

None. `hasNext()` only checks whether a next element exists - it does not advance the iterator.

---

## What happens when `iterator.remove()` is called?

If `iterator.next()` was called beforehand, the element at the iterator's current position is removed. If `next()` was not called first, an `IllegalStateException` is thrown.

---

## How will an already-instantiated iterator behave if `collection.remove()` is called?

On the next call to any iterator method, a `ConcurrentModificationException` will be thrown.

---

## How can `ConcurrentModificationException` be avoided while iterating over a collection?

- Use or implement a fail-safe iterator.
- Use `ConcurrentHashMap` or `CopyOnWriteArrayList`.
- Copy the collection to an array and iterate over the array.
- Lock the collection during iteration using a `synchronized` block.

> The last two options may decrease performance.

---

## Which collection follows FIFO (First-In-First-Out)?

`Queue` - elements are inserted at the tail and removed from the head.

---

## Which collection follows FILO (First-In-Last-Out)?

`Stack` - elements are inserted and removed from the top.

---

## Why was `ArrayList` added if `Vector` already existed?

- `Vector` methods are synchronized; `ArrayList` methods are not (faster in single-threaded use).
- `Vector` doubles its size on resize; `ArrayList` grows by 50%.
- `Vector` is a legacy class and its use is not recommended.

---

## What is the difference between `ArrayList` and `LinkedList`? When to use each?

`ArrayList` is a dynamically resizable array; `LinkedList` is a doubly linked list.

**`ArrayList`:**
- Random access by index: O(1)
- Search by value: O(N)
- Append at the end: O(1) amortized
- Insert/delete at an arbitrary position: O(N) - elements must be shifted
- Minimal memory overhead - only stores elements

**`LinkedList`:**
- Access by index or value: O(N)
- Access to the first/last element: O(1)
- Insert/delete at the beginning or end: O(1)
- Insert/delete at arbitrary position: O(1) (O(N) to find the position)
- Higher memory usage - each node stores references to previous and next

**When to use which:**
- Use `ArrayList` when frequent random access and low-memory overhead are required.
- Use `LinkedList` when frequent insertions/deletions occur, especially at the start or end of the list.
- In general, `ArrayList` outperforms `LinkedList` in most scenarios.

---

## What is the worst-case complexity of `contains()` for `LinkedList` and `ArrayList`?

O(N) for both - the search is linear.

---

## What is the worst-case complexity of `add()` for `LinkedList`?

O(N). Adding at the beginning or end is O(1), but adding at an arbitrary position requires traversal: O(N).

---

## What is the worst-case complexity of `add()` for `ArrayList`?

O(N). Appending at the end is O(1) amortized, but if capacity is exhausted, a new array (1.5× the previous size) is allocated and all elements are copied: O(N).

---

## You need to add one million elements. Which data structure would you use?

The best choice depends on:
- Where elements are being added (beginning, middle, or end).
- Subsequent operations (random access, deletions).
- Memory and performance constraints.

For sequential appends at the end, `ArrayList` is generally better. For frequent insertions in the middle, `LinkedList` may be preferable.

---

## How does element deletion work in `ArrayList`? How does its size change?

When an element is removed:
- All elements to the right of the deleted element shift one position to the left.
- The underlying array **capacity** does not decrease automatically.
- Unlike automatic expansion, `ArrayList` does not shrink unless `trimToSize()` is explicitly called.

---

## Efficient algorithm for removing multiple adjacent elements from the middle of an `ArrayList`

Instead of removing elements one by one (causing repeated shifts), shift all elements right of `m + n` to the left by `n` positions in one pass. The most efficient approach is `subList(fromIndex, toIndex).clear()`, which uses `System.arraycopy()` internally.

---

## How much additional memory is allocated when calling `ArrayList.add()`?

- If sufficient capacity exists: no extra memory is allocated.
- If capacity is exhausted: a new array of 1.5× the current size is created and all elements are copied (JDK 1.7+).

---

## How much additional memory is allocated when calling `LinkedList.add()`?

A new instance of the inner `Node` class is created for the new element.

---

## Is adding an element in the middle slower for `ArrayList` or `LinkedList`?

**For `ArrayList`:** capacity check + possible resize + shift elements right (O(N)) + insert.

**For `LinkedList`:** find the insertion position (O(N)) + insert (O(1)).

In the worst case, `LinkedList` is more efficient. However, `ArrayList` typically performs better in practice due to `System.arraycopy()` being highly optimized (cache-friendly memory access).

---

## Why does `ArrayList` store both `Object[] elementData` and `int size`?

- `elementData.length` is the **capacity** (allocated memory).
- `size` is the **actual number of stored elements**.

Capacity is always ≥ size, so they cannot be used interchangeably. The array expands dynamically but does not shrink automatically unless `trimToSize()` is called.

---

## Compare `Queue` and `Deque`. Who extends whom?

`Deque` extends `Queue`.

- **`Queue`** is FIFO - elements are inserted at the tail and removed from the head. Some implementations (`PriorityQueue`) don't strictly follow FIFO, using natural ordering or a comparator instead.
- **`Deque`** (Double-Ended Queue) extends `Queue` and supports insertion and removal at both ends. Implementations can behave as either FIFO (queue) or LIFO (stack).

> Neither `Queue` nor `Deque` override `equals()` and `hashCode()`, so they rely on `Object`'s reference-based comparison.

---

## Why does `LinkedList` implement both `List` and `Deque`?

`LinkedList` adds elements at both ends in O(1), making it a natural fit for `Deque`. As a `List`, it supports indexed retrieval and list operations (though O(N) for random access).

---

## Is `LinkedList` singly, doubly, or four-way linked?

**Doubly linked** - each node stores a reference to both the previous and next node.

---

## How to iterate over `LinkedList` in reverse without using the slow `get(index)`?

Use the built-in descending iterator:

```java
Iterator<Integer> descendingIterator = linkedList.descendingIterator();
while (descendingIterator.hasNext()) {
    System.out.println(descendingIterator.next());
}
```

This avoids O(N²) complexity that would result from calling `get(index)` in a loop.

---

## What does `PriorityQueue` allow?

A `PriorityQueue` is a heap-backed queue where the element with the highest priority is always at the head, regardless of insertion order. By default it's a min-heap, so the smallest element (by natural ordering) is dequeued first.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(1);
pq.offer(3);

pq.poll(); // returns 1 — not 5 (insertion order is ignored)
pq.poll(); // returns 3
pq.poll(); // returns 5
```

To reverse the order or sort by a custom field, pass a `Comparator` to the constructor:

```java
// Max-heap: largest element dequeued first
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

// Custom object: lowest-deadline task first
PriorityQueue<Task> tasks = new PriorityQueue<>(Comparator.comparingLong(Task::getDeadline));
```

**A few things to know:**
- Iteration order is undefined — only `poll()` and `peek()` respect priority.
- `null` elements are not permitted and will throw `NullPointerException`.
- Not thread-safe; use `PriorityBlockingQueue` for concurrent access.

Common use cases: Dijkstra's algorithm, task scheduling, and any problem requiring repeated extraction of a minimum or maximum value.

---

## Why is `Stack` considered outdated, and what should replace it?

`Stack` extends `Vector`, inheriting unnecessary methods (e.g., random access by index), and `Vector` is synchronized, making `Stack` slower. The recommended replacement is **`ArrayDeque`**, which provides better performance and correctly models LIFO behavior.

---

## Why use `HashMap` instead of `Hashtable`?

- `HashMap` is not synchronized → faster in single-threaded use.
- `HashMap` allows one `null` key and multiple `null` values; `Hashtable` does not.
- `HashMap`'s iterator is fail-fast.
- If thread safety is required, `ConcurrentHashMap` is a better alternative than `Hashtable`.

---

## Difference between `HashMap` and `IdentityHashMap`. When to use `IdentityHashMap`?

`IdentityHashMap` compares keys using **reference equality** (`==`) instead of `equals()`. It uses system identity hash codes rather than `hashCode()`, making it faster when `equals()` and `hashCode()` are expensive to compute.

Use `IdentityHashMap` when **object identity matters** - for example, in serialization or cloning, where distinct instances should be treated separately even if they are logically equal.

---

## What is the difference between `HashMap` and `WeakHashMap`?

Java has four reference types: strong, soft (`SoftReference`), weak (`WeakReference`), and phantom (`PhantomReference`). If an object is reachable only through weak references, it will be marked for garbage collection.

`WeakHashMap` stores keys as weak references. A key-value pair is automatically removed when no strong references to the key remain. This is useful for attaching extra information to objects you don't control - the entry disappears when the object is GC'd.

---

## Why not create a `SoftHashMap` based on `SoftReferences`?

`SoftHashMap` exists in third-party libraries such as Apache Commons.

---

## Why not create a `PhantomHashMap` based on `PhantomReferences`?

`PhantomReference.get()` always returns `null`, making it practically unusable as a map key.

---

## What does `LinkedHashMap` inherit from `LinkedList` and from `HashMap`?

`LinkedHashMap` extends `HashMap` but maintains a **doubly linked list** over its entries to define iteration order. By default, iteration follows **insertion order**. Setting the constructor parameter `accessOrder = true` switches to **access order** - calling `get()` or `put()` moves the accessed entry to the end of the list. (This is the basis for implementing an LRU cache.)

---

## How does the "sorted" nature of `SortedMap` manifest besides `toString()`?

It also manifests during **iteration** - iterating over a `SortedMap` always yields elements in sorted key order.

---

## HashMap

### How is `HashMap` structured?

`HashMap` consists of **buckets** - elements of an array where each cell stores a reference to a linked list (or, since Java 8, a red-black tree). When adding a new key-value pair:
1. The key's hash code is computed.
2. The bucket index is derived from the hash code.
3. If the bucket is empty, the new entry is stored there.
4. If the bucket already has entries (collision), the list is traversed. If an entry with the same key is found, its value is replaced; otherwise, the new entry is appended.

---

### Open addressing vs. chaining - how is `HashMap` implemented and why?

`HashMap` uses **chaining** (each bucket holds a linked list). With chaining:
- The load factor can exceed 1.
- Performance degrades linearly as more elements are added.
- Well-suited when the number of elements is unknown in advance.

**Open addressing** variants (linear probing, quadratic probing, double hashing):
- The number of elements cannot exceed the array size.
- As load factor increases, performance drops sharply.
- Deletion is more complex.
- No overhead for linked list objects; simpler serialization.

---

### What happens when two elements have the same `hashCode()` but `equals()` returns `false`?

They hash to the same bucket. Since `equals()` returns `false`, the new element is appended to the end of that bucket's list (collision handled by chaining).

---

### What is the initial number of buckets in `HashMap`?

**16** by default. A custom initial capacity can be specified via parameterized constructors.

---

### What is the time complexity of `HashMap` operations? Is it guaranteed?

Generally O(1) for insertion, lookup, and deletion. However:
- With a good hash function: no worse than O(log N) (Java 8+ converts long chains to red-black trees).
- Worst case (all keys hash to the same bucket): O(N) - degrades to a linked list.

---

### Can `HashMap` degrade into a linked list even if keys have different `hashCode()` values?

Yes, if the method that maps hash codes to bucket indices returns the same index for different hash codes.

---

### In what case can an element be lost in a `HashMap`?

If a key is a mutable object and a field used in `hashCode()` is modified after the element is inserted. Subsequent lookups will compute a different hash, landing in a different bucket, and the element becomes effectively unreachable.

---

### Why can't `byte[]` be used as a key in `HashMap`?

Arrays don't override `hashCode()` (computed from object identity, not contents) and don't override `equals()` (compares by reference). This means you cannot retrieve a value using a different array with the same contents - only the exact same array reference would work.

---

### What is the role of `equals()` and `hashCode()` in `HashMap`?

- `hashCode()` determines **which bucket** to look in.
- `equals()` identifies **which entry within the bucket** matches the key.

---

### What is the maximum number of possible `hashCode()` values?

2³² - the full range of `int`.

---

### How does `ConcurrentHashMap` work internally?

`ConcurrentHashMap` is a thread-safe, high-performance map. The modern implementation (Java 8+) uses three strategies to minimize contention:

- **Lock-free reads**: `get()` uses `volatile` field visibility - no locking required.
- **CAS for empty buckets**: inserting into an empty bucket uses Compare-And-Swap - no lock needed.
- **Bucket-level synchronization**: collisions lock only the head node of the affected bucket, allowing concurrent modification of different buckets.

| Feature        | Implementation Detail                                                                         |
|----------------|-----------------------------------------------------------------------------------------------|
| Data structure | Array of `Node` buckets; lists treeify into red-black trees when a bucket exceeds 8 elements. |
| Locking        | Fine-grained - only the specific bucket being modified is locked.                             |
| Iteration      | Weakly consistent iterators; do not throw `ConcurrentModificationException`.                  |
| Null policy    | `null` keys and values are not allowed (prevents ambiguity in concurrent operations).         |
| Resizing       | Cooperative - multiple threads assist in moving entries to the new table.                     |

> **Java 7 and earlier**: used segment locking (sharding) - the map was divided into 16 segments, each with its own lock. More granular than `Hashtable` but less so than modern bucket-level locking.

---

### What is the worst-case complexity of `HashMap.get(key)`?

O(N) - when multiple keys hash to the same bucket, creating a degenerate linked list. Since Java 8, buckets exceeding a threshold (8) are converted to red-black trees, bounding worst-case to O(log N).

---

### Why can a linked list in a `HashMap` always be converted to a red-black tree, even without `Comparable`?

A red-black tree requires element comparison. Java uses the following fallback comparison algorithm when keys don't implement `Comparable`:
1. Compare hash codes.
2. If hash codes are equal and both keys implement `Comparable`, use `compareTo()`.
3. Otherwise, use `tieBreakOrder()`: first compare class names via `getClass().getName()`; if the same class, use `System.identityHashCode()`.

---

### How many steps does `HashMap.get(key)` take for an existing key?

- **`null` key**: 1 step - delegates to `getForNullKey()`.
- **Non-null key**: 4 steps - compute hash, determine bucket index, search the bucket, return the value.

---

### How many new objects are created when adding a new element to `HashMap`?

One - a new instance of the static nested class `Entry<K, V>` (or `Node<K, V>` in Java 8+).

---

### How and when does `HashMap` increase the number of buckets?

`HashMap` has a `loadFactor` (default `0.75`). When the number of entries exceeds `capacity × loadFactor`, the bucket array **doubles in size** and all entries are rehashed to new positions.

---

### Explain the parameters of `HashMap(int initialCapacity, float loadFactor)`

- `initialCapacity` - the number of buckets at creation time.
- `loadFactor` - the ratio of stored elements to capacity at which rehashing is triggered. A lower value reduces collisions but wastes memory; a higher value saves memory but increases collision likelihood.

---

### Will `HashMap` work if all keys have the same `hashCode()`?

Yes, but it degrades into a linked list (or red-black tree in Java 8+) within a single bucket, losing performance advantages.

---

### How can you iterate over all keys in a `Map`?

```java
for (K key : map.keySet()) { ... }
```

---

### How can you iterate over all values in a `Map`?

```java
for (V value : map.values()) { ... }
```

---

### How can you iterate over all key-value pairs in a `Map`?

```java
for (Map.Entry<K, V> entry : map.entrySet()) {
    K key = entry.getKey();
    V value = entry.getValue();
}
```

---

## `TreeSet` & `HashSet`

### What are the differences between `TreeSet` and `HashSet`?

- **`TreeSet`** - backed by a red-black tree; elements are always sorted. Basic operations are O(log N).
- **`HashSet`** - backed by a `HashMap` (an element serves as key and a dummy object as value); no ordering guarantee. Basic operations are O(1) on average.

---

### What happens if elements are added to `TreeSet` in ascending order?

`TreeSet` is backed by a self-balancing red-black tree, so insertion order does not matter - it always maintains its balance and performance.

---

### How does `LinkedHashSet` differ from `HashSet`?

`LinkedHashSet` is backed by `LinkedHashMap` instead of `HashMap`, maintaining a doubly linked list over entries. Iteration order matches **insertion order**. Re-inserting an existing element does not change its position.

---

### Why is there a special `EnumSet` for enums? Why weren't `HashSet` or `TreeSet` sufficient?

`EnumSet` uses a **bit vector** for storage, giving it exceptional compactness and speed. All basic operations run in O(1) and are generally faster than `HashSet`. Bulk operations (`containsAll()`, `retainAll()`) are significantly faster. Iteration follows the declaration order of the enum constants. It also provides convenient static factory methods.

---

## Code Snippets & Practical Tasks

### Ways to iterate over a `List`

```java
// Iterator
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
}

// Indexed for loop
for (int i = 0; i < list.size(); i++) {
    String element = list.get(i);
}

// while loop
int i = 0;
while (i < list.size()) {
    String element = list.get(i);
    i++;
}

// for-each
for (String element : list) {
    // use element
}
```

---

### How to get synchronized wrappers for standard collections?

Use the static methods of the `Collections` utility class:

```java
Map<K, V> syncMap = Collections.synchronizedMap(new HashMap<>());
List<T> syncList = Collections.synchronizedList(new ArrayList<>());
```

> Manual synchronization is still required when iterating over the collection.

Since Java 6,
the JCF have been extended with purpose-built concurrent collections such as `CopyOnWriteArrayList` and `ConcurrentHashMap`.

---

### How to get a read-only collection?

```java
Collections.unmodifiableList(list);
Collections.unmodifiableSet(set);
Collections.unmodifiableMap(map);
```

These methods return a read-only view containing the same elements. Mutation attempts throw `UnsupportedOperationException`.

---

### Single-threaded program that causes `ConcurrentModificationException`

```java
public static void main(String[] args) {
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(3);

    for (Integer integer : list) {
        list.remove(1); // Modifies the list during iteration → ConcurrentModificationException
    }
}
```

---

### Example that causes `UnsupportedOperationException`

```java
public static void main(String[] args) {
    List<Integer> list = Collections.emptyList();
    list.add(0); // UnsupportedOperationException - the list is unmodifiable
}
```

---

### Symmetric difference of two collections using `addAll`, `removeAll`, `retainAll`

The symmetric difference is the set of elements that belong to exactly one of the two collections (not both).

```java
<T> Collection<T> symmetricDifference(Collection<T> a, Collection<T> b) {
    // Union of both collections
    Collection<T> result = new ArrayList<>(a);
    result.addAll(b);

    // Intersection of both collections
    Collection<T> intersection = new ArrayList<>(a);
    intersection.retainAll(b);

    // Remove elements present in both
    result.removeAll(intersection);

    return result;
}
```

---

### Copy any collection to an array in one line

```java
Object[] array = collection.toArray();
```

---

### Get a sublist without the first and last three elements

```java
List<Integer> subList = list.subList(3, list.size() - 3);
```

---

### Convert `HashSet` to `ArrayList` in one line

```java
ArrayList<Integer> list = new ArrayList<>(hashSet);
```

---

### Convert `ArrayList` to `HashSet` in one line

```java
HashSet<Integer> set = new HashSet<>(arrayList);
```

---

### Create a `HashSet` from the keys of a `HashMap`

```java
HashSet<K> set = new HashSet<>(map.keySet());
```

---

### Create a `HashMap` from a `HashSet<Map.Entry<K, V>>`

```java
HashMap<K, V> map = new HashMap<>(set.size());
for (Map.Entry<K, V> entry : set) {
    map.put(entry.getKey(), entry.getValue());
}
```
