# Concurrency

## Java Memory Model (JMM)

The Java Memory Model describes the behavior of threads in the Java runtime. It defines a set of inter-thread interaction actions (reading/writing variables, acquiring/releasing monitors, reading/writing volatile variables, starting threads, etc.) and the **happens-before** relationship.

If operation X *happens-before* operation Y, all code following Y in one thread sees all changes made before X in another thread.

**Key happens-before rules:**

| Rule                       | Description                                                                           |
|----------------------------|---------------------------------------------------------------------------------------|
| Program order              | Within a single thread, any operation happens-before any subsequent operation.        |
| Monitor unlock → lock      | Releasing a monitor happens-before acquiring the same monitor.                        |
| Synchronized exit → entry  | Exiting a synchronized block happens-before entering one on the same monitor.         |
| Volatile write → read      | Writing to a volatile field happens-before reading that same field.                   |
| `start()` → `run()`        | Calling `start()` on a thread happens-before the thread's `run()` executes.           |
| `run()` → `join()`         | Completion of `run()` happens-before `join()` returns or `isAlive()` returns `false`. |
| Constructor → `finalize()` | Completing a constructor happens-before `finalize()` is called.                       |
| `interrupt()` → detection  | Calling `interrupt()` happens-before the thread detects the interruption.             |
| Transitivity               | If X happens-before Y, and Y happens-before Z, then X happens-before Z.               |

### Memory Visibility

A thread may cache field values in registers or CPU caches. Other threads may not see the latest changes without synchronization. Java provides three keywords addressing visibility:

- `synchronized`
- `volatile`
- `final`

### Thread-Local Memory

Each thread has its own **working memory** (local copy of variables it uses). When a thread enters a `synchronized` block, it refreshes from main memory; on exit, it writes back. This ensures visibility only when threads synchronize on the same monitor.

### `volatile`
- Writes always go directly to the main memory.
- Reads always come directly from the main memory.
- Guarantees **visibility** but **not atomicity**.

### `final`
- After correct object construction, any thread can read `final` fields without synchronization.
- "Correct construction" means the reference must not escape before the constructor completes.
- Enables safely shared immutable objects.
- If `final` fields are modified via reflection after construction, visibility guarantees are lost.

### Instruction Reordering

Processors and compilers may reorder instructions for performance. From another thread's perspective, operations may not appear to execute in source-code order.

**Preventing reordering:** Reads and writes of `volatile` variables cannot be reordered with reads/writes of other variables, making `volatile` useful as a completion flag. Otherwise, happens-before rules provide ordering guarantees only in specific scenarios.

---

## Thread Safety

**Thread safety** is a property of code or an object that guarantees correct behavior when accessed concurrently by multiple threads - no race conditions, no inconsistent state.

---

## Concurrency vs. Parallelism

| Characteristic | Concurrency                                              | Parallelism                                                      |
|----------------|----------------------------------------------------------|------------------------------------------------------------------|
| What is it?    | Handling multiple tasks by interleaving their execution. | Executing multiple tasks (or parts of one) truly simultaneously. |
| Control flows  | Multiple                                                 | One or multiple                                                  |
| Result         | Non-deterministic                                        | Can be deterministic                                             |
| Example        | Web server handling multiple requests                    | Matrix multiplication split across CPU cores                     |

> **Important:** Concurrency does not require parallelism. Tasks can be interleaved on a single core without ever running simultaneously.

**Simple analogy:**
- *Concurrency*: one barista takes orders and prepares coffee, not always in the same order.
- *Parallelism*: two baristas each prepare different orders at the same time.

---

## Cooperative vs. Preemptive Multitasking

**Cooperative multitasking** - each thread voluntarily yields control. Simple, low context-switch overhead, but one hung thread can freeze the whole system.

**Java uses preemptive multitasking** - the OS decides when to switch threads, forcing switches at regular intervals via hardware timer interrupts. This prevents one blocked thread from stalling the system and improves responsiveness.

---

## Key Concurrency Concepts

| Concept                    | Description                                                                                                                                                                            |
|----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Ordering**               | Determines when one thread may observe out-of-order instruction execution from another thread.                                                                                         |
| **As-If-Serial Semantics** | Within a single thread, all operations appear to execute in source-code order, regardless of internal optimizations.                                                                   |
| **Sequential Consistency** | All operations across threads appear in a single global order, as if executed one at a time.                                                                                           |
| **Visibility**             | Defines when actions by one thread become visible to another. Without synchronization, changes may not propagate.                                                                      |
| **Happens-Before**         | If X happens-before Y, all effects of X are visible before Y executes.                                                                                                                 |
| **Atomicity**              | An operation is indivisible - it either completes entirely or not at all.                                                                                                              |
| **Mutual Exclusion**       | Only one thread at a time can access a shared resource. Implemented via `synchronized`, `Lock`, `Semaphore`.                                                                           |
| **Safe Publication**       | Ensuring objects are correctly shared between threads. Methods: static initializers, `volatile`, `Atomic*` variables, `synchronized`, `final` fields in correctly constructed objects. |

---

## Process vs. Thread

|               | Process                                                                   | Thread                                         |
|---------------|---------------------------------------------------------------------------|------------------------------------------------|
| Definition    | An independent instance of a program with its own address space.          | A lightweight execution unit within a process. |
| Memory        | Separate address space - cannot directly access another process's memory. | Shared address space within the same process.  |
| Communication | Requires IPC (pipes, sockets, shared memory, files).                      | Direct access to shared data structures.       |
| Creation cost | Expensive                                                                 | Much cheaper                                   |

---

## Green Threads and Virtual Threads

**Green threads** are threads managed by the runtime (JVM) rather than the OS. The JVM schedules them; the OS sees only one native thread.

|               | Advantage                                                 | Disadvantage                                                           |
|---------------|-----------------------------------------------------------|------------------------------------------------------------------------|
| Green threads | Cheap to create, fast context switching, high scalability | No true parallelism; one blocking system call blocks all green threads |

Java originally supported green threads but switched to **native OS threads** in Java 1.2 to use multicore processors.

**Project Loom (Java 19+)** reintroduced lightweight **virtual threads** that work similarly to green threads but can run alongside native threads efficiently.

---

## `Thread` vs. `Runnable`

- **`Thread`** - a class that wraps a native OS thread.
- **`Runnable`** - an interface representing a task to execute.

Prefer `Runnable` because:
- Avoids the multiple-inheritance constraint (you can extend another class while implementing `Runnable`).
- Decouples task logic from thread management.

---

## `start()` vs. `run()`

- **`start()`** - creates a new thread and executes `run()` within it.
- **`run()`** - executes in the current thread, like a regular method call. No new thread is created.

---

## Monitor and Synchronization

A **monitor** (mutex) controls exclusive access to a shared resource. Every Java object has a built-in monitor managed by the JVM. Only one thread can hold the monitor at a time; others block until it is released.

```java
class SharedResource {
    synchronized void access() { // Acquires monitor on 'this'
        System.out.println(Thread.currentThread().getName() + " is accessing...");
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
        System.out.println(Thread.currentThread().getName() + " is done.");
    }
}
```

**For static synchronized methods**, synchronization is on the `Class` object (`SomeClass.class`), not an instance. The following is equivalent:

```java
public static synchronized void someMethod() { ... }

public static void someMethod() {
    synchronized (SomeClass.class) { ... }
}
```

### Synchronization Techniques

| Technique                   | Description                                                                                                                                            |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `synchronized` block/method | Mutual exclusion - only one thread executes the block at a time.                                                                                       |
| `wait()` / `notify()`       | Thread coordination - `wait()` releases the lock and suspends; `notify()` wakes one waiting thread. Both must be called inside a `synchronized` block. |
| `join()`                    | Makes one thread wait until another thread finishes.                                                                                                   |
| `java.util.concurrent`      | Advanced utilities: `Lock`, `Semaphore`, `Atomic*` classes, etc.                                                                                       |

```java
// wait() / notify() example
synchronized void waitForSignal() throws InterruptedException {
    wait();   // releases lock, suspends thread
}
synchronized void sendSignal() {
    notify(); // wakes one waiting thread
}
```

```java
// join() example
t1.start();
t1.join(); // the main thread waits for t1 to finish
```

```java
// ReentrantLock example
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock(); // always release in finally
}
```

---

## `Thread.sleep()` vs. `Thread.yield()`

|           | `sleep(time)`                                                         | `yield()`                                                               |
|-----------|-----------------------------------------------------------------------|-------------------------------------------------------------------------|
| Effect    | Moves thread from Running → Timed Waiting for the specified duration. | Moves thread from Running → Runnable, hinting the scheduler to switch.  |
| Guarantee | Thread resumes after the sleep period (or on interrupt).              | No guarantee - the scheduler may immediately re-select the same thread. |

---

## Deadlock

A **deadlock** occurs when threads wait indefinitely for resources held by each other. Four conditions must all hold:

1. **Mutual Exclusion** - at least one resource is held exclusively.
2. **Hold and Wait** - a thread holds a resource while waiting for another.
3. **No Preemption** - resources cannot be forcibly taken.
4. **Circular Wait** - a circular chain of threads each waiting for the next.

```java
// Classic deadlock: t1 locks resource1 then waits for resource2;
// t2 locks resource2 then waits for resource1.
Thread t1 = new Thread(() -> {
    synchronized (resource1) {
        synchronized (resource2) { /* ... */ }
    }
});
Thread t2 = new Thread(() -> {
    synchronized (resource2) {
        synchronized (resource1) { /* ... */ }
    }
});
```

**Prevention:**
- Acquire locks in a consistent global order.
- Use `tryLock()` with a timeout (`ReentrantLock`).
- Apply lock hierarchies.

---

## Livelock

A **livelock** occurs when threads continuously react to each other's state without making actual progress. Unlike deadlock, threads are not blocked - they are active but stuck in an unproductive cycle.

**Prevention:**
- Introduce randomness or backoff strategies to break the cycle.
- Use proper synchronization to ensure progress.

---

## Race Condition

A **race condition** is a bug where program behavior depends on the timing of thread execution. It occurs when multiple threads access a shared state without proper synchronization.

**Solutions:**
- Use a local copy of a shared variable (only safe if the variable is atomic or `volatile`).
- Wrap access in a `synchronized` block.
- Use `Atomic*` classes for compound operations.

> There is no universal detector for race conditions. The best approach is the correct design from the start.

---

## `volatile`, `synchronized`, `transient`, `native`

| Keyword        | Multithreading? | Purpose                                                                                                                                                                                                                      |
|----------------|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `volatile`     | Yes             | Forces all reads/writes to go to main memory. Guarantees visibility but **not** atomicity. Safe for single reads/writes of primitives; for references, only the reference itself is synchronized, not the object's contents. |
| `synchronized` | Yes             | Guarantees both visibility and atomicity. Only one thread executes the block at a time.                                                                                                                                      |
| `transient`    | No              | Excludes a field from serialization.                                                                                                                                                                                         |
| `native`       | No              | Marks a method implemented in platform-dependent code (C/C++ via JNI).                                                                                                                                                       |

---

## `volatile` vs. `Atomic*`

| Feature           | `volatile`                   | `Atomic*` (e.g., `AtomicInteger`)                 |
|-------------------|------------------------------|---------------------------------------------------|
| Guarantees        | Visibility only              | Visibility + Atomicity                            |
| Atomic operations | No - `count++` is NOT atomic | Yes - `getAndIncrement()` is atomic               |
| Use case          | Single reads/writes          | Compound operations (increment, compare-and-swap) |

---

## `compareAndSwap()` vs. `weakCompareAndSwap()`

|                | `compareAndSwap()`                             | `weakCompareAndSwap()`                                                        |
|----------------|------------------------------------------------|-------------------------------------------------------------------------------|
| Memory barrier | Yes - establishes happens-before               | No                                                                            |
| Reliability    | Always retries if expected value doesn't match | May return false spuriously                                                   |
| Performance    | Slightly slower                                | Faster                                                                        |
| Use when       | Strong correctness is required                 | Performance is critical and occasional failures are handled at a higher level |

---

## Thread Priority

Threads have a priority between `Thread.MIN_PRIORITY` (1) and `Thread.MAX_PRIORITY` (10), default `Thread.NORM_PRIORITY` (5). Higher priority threads are given preference by the scheduler, but actual behavior depends on the OS.

```java
thread.setPriority(Thread.MAX_PRIORITY);
int p = thread.getPriority();
```

---

## Daemon Threads

Daemon threads run in the background supporting other threads. When all non-daemon threads finish, the JVM terminates all remaining daemon threads automatically. The Garbage Collector is a daemon thread.

```java
thread.setDaemon(true); // must be called before start()
boolean d = thread.isDaemon();
```

> The main thread cannot be made a daemon - if it were, the JVM could terminate before important logic executes.

---

## How to Stop a Thread

**Deprecated:** `stop()`, `suspend()`, `resume()` - removed due to deadlock risk.

**Recommended: use `interrupt()`**

```java
class MyThread extends Thread {
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                break; // exit on interrupt
            }
        }
    }
}
thread.interrupt(); // request stop
```

**Alternative: volatile flag**

```java
private volatile boolean running = true;

public void run() {
    while (running) { /* work */ }
}
public void stop() { running = false; }
```

> Prefer `interrupt()` - a volatile flag cannot wake a blocked thread.

**Why `Thread.stop()` is dangerous:** it terminates the thread at an arbitrary point, potentially leaving shared resources in an inconsistent state, corrupting data, or causing deadlocks in waiting threads.

---

## `interrupted()` vs. `isInterrupted()`

| Method                 | Static | Clears the interruption flag?        |
|------------------------|--------|--------------------------------------|
| `Thread.interrupted()` | Yes    | Yes - resets the flag after checking |
| `isInterrupted()`      | No     | No - only checks, does not modify    |

---

## Exception Handling in Threads

If an exception in a thread is not caught, the thread terminates and enters the DEAD state. Other threads are unaffected (unless they share resources with them).

Use `Thread.UncaughtExceptionHandler` to handle uncaught exceptions:

```java
thread.setUncaughtExceptionHandler((t, e) ->
    System.out.println(t.getName() + " threw: " + e.getMessage())
);
```

---

## `Runnable` vs. `Callable`

| Feature            | `Runnable`    | `Callable<V>` |
|--------------------|---------------|---------------|
| Introduced         | Java 1.0      | Java 5        |
| Method             | `run()`       | `call()`      |
| Return value       | None (`void`) | Returns `V`   |
| Checked exceptions | Cannot throw  | Can throw     |

---

## `FutureTask`

`FutureTask<V>` implements both `Future<V>` and `Runnable` - it represents a cancellable asynchronous computation.

| Method                         | Description                            |
|--------------------------------|----------------------------------------|
| `run()`                        | Starts computation.                    |
| `cancel(boolean mayInterrupt)` | Cancels execution.                     |
| `isCancelled()`                | Checks if cancelled.                   |
| `isDone()`                     | Checks if complete.                    |
| `get()`                        | Retrieves result (blocks until ready). |

```java
FutureTask<Integer> futureTask = new FutureTask<>(() -> 42);
new Thread(futureTask).start();
System.out.println(futureTask.get()); // blocks until done
```

---

## `CountDownLatch` vs. `CyclicBarrier`

| Feature       | `CountDownLatch`                                                 | `CyclicBarrier`                                      |
|---------------|------------------------------------------------------------------|------------------------------------------------------|
| Purpose       | Wait for N operations in other threads to complete.              | Wait until N threads all reach a barrier point.      |
| Reusable      | No - single use only.                                            | Yes - resets automatically.                          |
| Mechanism     | `countDown()` decrements; `await()` blocks until 0.              | `await()` blocks each thread until all have arrived. |
| On completion | No built-in action.                                              | Can run a `barrierAction` automatically.             |
| Analogy       | Tourists wait for a set number of people before starting a tour. | Cyclists all wait at the start line before racing.   |

```java
CountDownLatch latch = new CountDownLatch(3);

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        System.out.println(Thread.currentThread().getName() + " finished");
        latch.countDown();
    }).start();
}
latch.await(); // the main thread waits for all 3
System.out.println("All workers finished.");
```

---

## Thread Pool

A thread pool manages a set of reusable worker threads, avoiding the overhead of creating and destroying threads per task.

| Method                      | Description                                                                         |
|-----------------------------|-------------------------------------------------------------------------------------|
| `newFixedThreadPool(n)`     | Fixed number of threads. Best for CPU-bound tasks.                                  |
| `newCachedThreadPool()`     | Creates threads as needed; removes idle ones after 60s. Best for small async tasks. |
| `newSingleThreadExecutor()` | Single thread; ensures sequential execution.                                        |
| `newScheduledThreadPool(n)` | Supports delayed or periodic task execution.                                        |

**When the queue is full:** by default, `ThreadPoolExecutor` throws `RejectedExecutionException`. Use a custom `RejectedExecutionHandler` to override this.

### `execute()` vs. `submit()`

| Method                      | Returns     | Exception handling           |
|-----------------------------|-------------|------------------------------|
| `execute(Runnable)`         | Nothing     | Thrown directly              |
| `submit(Callable/Runnable)` | `Future<T>` | Captured inside the `Future` |

---

## `synchronized` vs. `ReentrantLock`

| Feature                      | `synchronized`             | `ReentrantLock`                            |
|------------------------------|----------------------------|--------------------------------------------|
| Lock release                 | Automatic (JVM guarantees) | Manual - must call `unlock()` in `finally` |
| Fairness                     | No                         | Optional FIFO fairness policy              |
| Try-lock                     | No                         | `tryLock()` - avoids blocking              |
| Interruptibility             | No                         | Can be interrupted while waiting           |
| Performance under contention | Lower                      | Higher scalability                         |

Both are **reentrant**: if a thread already holds the lock and tries to acquire it again, the count increments and the lock is only released when it has been released as many times as acquired.

---

## Stack vs. Heap in Multithreading

| Feature       | Stack                         | Heap                                                        |
|---------------|-------------------------------|-------------------------------------------------------------|
| Scope         | Private to each thread        | Shared across all threads                                   |
| Stores        | Local variables, method calls | Objects and instance variables                              |
| Thread safety | Inherently safe               | Requires synchronization (`synchronized`, `volatile`, etc.) |

---

## `ThreadLocal`

`ThreadLocal<T>` gives each thread its own independent copy of a variable. The value is set with `set()` and retrieved with `get()`.

```java
ThreadLocal<Object> local = new ThreadLocal<>();
local.set(myObject); // stored for the current thread only
local.get();         // returns the current thread's value
```

> **Important:** `ThreadLocal` isolates references, not objects. If two threads' `ThreadLocal` variables point to the same object, collisions can still occur. Also, initializing a `ThreadLocal` in the main thread and reading it in another returns `null` - values are thread-specific.

---

## Sharing Data Between Threads

- Use a shared object (requires synchronization).
- Use concurrent collections: `BlockingQueue`, `ConcurrentHashMap`, etc.
- Use `Exchanger<V>` to swap data between exactly two threads at a synchronization point.

---

## Semaphore

A `Semaphore` controls access to a shared resource via a counter:
- Counter decrements when a thread enters.
- Counter increments when a thread exits.
- When the counter reaches zero, additional threads block until another exits.

Useful for limiting access to resource pools (e.g., database connections).

---

## Fork/Join Framework

Introduced in JDK 7. Designed for tasks that can be recursively divided into smaller subtasks and processed in parallel.

- **Fork** - split a task into subtasks; recurse until trivial.
- **Join** - combine results of subtasks.

Uses a **work-stealing algorithm**: threads that finish their subtasks steal work from threads that are still busy, maximizing CPU utilization.

---

## Double-Checked Locking Singleton

A thread-safe Singleton that only synchronizes when necessary:

```java
class Singleton {
    private static volatile Singleton instance;

    static Singleton getInstance() {
        Singleton current = instance;
        if (current == null) {
            synchronized (Singleton.class) {
                current = instance;
                if (current == null) {
                    instance = current = new Singleton();
                }
            }
        }
        return current;
    }
}
```

> `volatile` is **mandatory**. Without it, a thread may see a partially constructed object due to instruction reordering.

---

## Immutable Objects

Immutable objects are thread-safe without synchronization - their state never changes after construction, eliminating race conditions.

**How to create an immutable class:**
1. All fields must be `final`, initialized in the constructor.
2. The class must be `final` (prevent subclassing).
3. No setters.
4. Return defensive copies of any mutable field (e.g., `List`).

```java
public final class ImmutableExample {
    private final String name;
    private final int age;

    public ImmutableExample(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}
```

---

## Compare-And-Swap (CAS)

### What is CAS?

Compare-And-Swap (CAS) is a low-level atomic CPU instruction used to achieve thread-safe operations without locks. It is the foundation of Java's non-blocking concurrency utilities in `java.util.concurrent.atomic`.

The operation takes three arguments:

- **Memory location** - the variable to update.
- **Expected value** - the value the variable is assumed to currently hold.
- **New value** - the value to write if the assumption holds.

The CPU atomically checks whether the current value matches the expected value, and only then writes the new value. The entire check-and-set happens as a single, uninterruptible hardware instruction.

Pseudocode:

```
if (currentValue == expectedValue) {
    currentValue = newValue;
    return true;   // success
} else {
    return false;  // another thread changed it first
}
```

In Java, CAS is exposed through the `Atomic*` classes and, at a lower level, via `VarHandle` (Java 9+) and `sun.misc.Unsafe`.

```java
AtomicInteger counter = new AtomicInteger(0);

// Succeeds - current value is 0, updates to 1
boolean success = counter.compareAndSet(0, 1);

// Fails - current value is now 1, not 0
boolean failure = counter.compareAndSet(0, 99);
```

---

### How CAS Works Internally

Instead of acquiring a lock, a thread using CAS follows a retry loop:

1. Read the current value.
2. Compute the new value.
3. Attempt CAS - if it succeeds, done.
4. If it fails (another thread changed the value), go back to step 1 and retry.

This pattern is called an **optimistic spin loop** or **compare-and-swap loop**:

```java
AtomicInteger value = new AtomicInteger(0);

public void increment() {
    int current;
    do {
        current = value.get();
    } while (!value.compareAndSet(current, current + 1));
}
```

This is exactly how `AtomicInteger.incrementAndGet()` works under the hood.

---

### Tradeoffs

| Aspect                                   | Detail                                                                                                                                                                         |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **No locks**                             | Threads never block - no mutex acquisition, no context switching.                                                                                                              |
| **High throughput under low contention** | When conflicts are rare, CAS nearly always succeeds on the first try.                                                                                                          |
| **Spin waste under high contention**     | When many threads compete on the same variable, repeated CAS failures burn CPU cycles in busy-wait loops.                                                                      |
| **Not suitable for compound operations** | CAS protects a single memory location. Operations spanning multiple variables still require locks or other coordination.                                                       |
| **Progress guarantee**                   | CAS provides *lock-freedom* - at least one thread always makes progress, even if others keep retrying. This is stronger than a mutex, where a sleeping thread blocks everyone. |
| **Hardware dependency**                  | Relies on the CPU providing an atomic CAS instruction (virtually universal on modern x86/ARM).                                                                                 |

**Rule of thumb:** prefer CAS-based atomics for simple, frequently-updated counters and flags under moderate contention. For high-contention scenarios, consider `LongAdder` / `LongAccumulator`, which reduce contention by striping the value across multiple cells.

---

### The ABA Problem

#### What is it?

The ABA problem is a subtle correctness bug that can occur when using CAS. It happens when:

1. Thread A reads a value of **A**.
2. Thread B changes the value from **A → B → A** (back to A).
3. Thread A performs CAS expecting **A** - it succeeds, even though the value was changed in between.

From CAS's perspective, the value looks unchanged, but the state of the system may have changed meaningfully. The operation proceeds as if nothing happened, which can lead to silent data corruption.

---

### Solutions to the ABA Problem

#### 1. Stamped Reference - `AtomicStampedReference`

Pairs the value with an integer **stamp** (version counter). Both the value and the stamp must match for CAS to succeed. Each write increments the stamp, making A→B→A distinguishable from an unchanged A.

```java
AtomicStampedReference<Integer> ref =
        new AtomicStampedReference<>(100, 0);

// Read value and stamp together
int[] stampHolder = new int[1];
Integer value = ref.get(stampHolder);
int currentStamp = stampHolder[0];

// CAS requires both value AND stamp to match
boolean success = ref.compareAndSet(
        value,           // expected value
        200,             // new value
        currentStamp,    // expected stamp
        currentStamp + 1 // new stamp
);
```

After A→B→A the stamp will be 2 (not 0), so a stale CAS expecting stamp 0 will fail correctly.

#### 2. Marked Reference - `AtomicMarkableReference`

Similar to `AtomicStampedReference` but uses a single **boolean mark** instead of an integer stamp. Useful when you only need to flag a value as logically deleted or invalid (e.g., in lock-free linked list deletion), rather than tracking a full version count.

```java
AtomicMarkableReference<String> ref =
        new AtomicMarkableReference<>("hello", false);

// Mark the node as logically deleted
ref.compareAndSet("hello", "hello", false, true);

// Check mark before acting
boolean[] markHolder = new boolean[1];
String val = ref.get(markHolder);
if (!markHolder[0]) {
    // safe to use val
}
```

#### 4. Garbage Collection (Java's built-in protection)

In Java, the ABA problem with **object identity** is largely mitigated by the garbage collector.
An object cannot be freed, and its memory reused while any thread holds a reference to it,
so the exact pointer-reuse scenario from C/C++ lock-free code is much rarer.
However, ABA can still occur with **logical state** (e.g., a node removed and re-inserted into a data structure),
so `AtomicStampedReference` is still needed in those cases.

---

### Summary

| Topic                     | Key Point                                                                                      |
|---------------------------|------------------------------------------------------------------------------------------------|
| CAS                       | Atomic hardware instruction: update a value only if it matches an expected value.              |
| Optimistic concurrency    | No locks - threads retry on conflict instead of blocking.                                      |
| Low contention            | CAS is very efficient; prefer it for simple shared counters and flags.                         |
| High contention           | Spinning wastes CPU; consider `LongAdder` or lock-based approaches.                            |
| ABA problem               | A value can change A→B→A between a read and a CAS, causing a stale update to succeed silently. |
| `AtomicStampedReference`  | Pairs the value with a version stamp - the canonical Java solution to ABA.                     |
| `AtomicMarkableReference` | Uses a boolean mark instead of a stamp - suited for logical deletion patterns.                 |
| Java GC                   | Reduces (but does not eliminate) ABA risk by preventing premature memory reuse.                |

---

## Busy Spin

A technique where a thread waits without calling `wait()`, `sleep()`, or `yield()` - instead it loops continuously:

```java
while (!conditionMet()) {
    // empty loop
}
```

**Why use it:** avoids context switching in multicore systems. When a thread is suspended, resuming it on a different core may invalidate CPU cache lines. Busy spin keeps the cache warm.

**Drawbacks:** high CPU usage; should only be used in latency-critical applications (e.g., high-frequency trading).

---

## Why is `Future.get()` Being Synchronous a Problem?

When a `Callable` is submitted to an `ExecutorService`, it returns a `Future`. Calling `get()` on that `Future` **blocks** the calling thread until the result is ready. This negates the benefit of async execution:

- **Thread waste** - the calling thread sits idle doing nothing.
- **UI freezing** - in GUI or web front-ends, blocking the main thread causes unresponsiveness.
- **Throughput reduction** - fewer requests can be handled in parallel.
- **Cascade failures** - in microservices, a blocked thread waiting for a hung downstream service can propagate failure across the system.
- **Infinite blocking** - without a timeout (`get(long, TimeUnit)`), the thread may block forever.

**Better alternative:** `CompletableFuture` with non-blocking callbacks (`thenApply`, `thenAccept`) that execute only when the result is ready, without blocking any thread.

---

## Multithreading Best Practices

1. **Give threads meaningful names** - `OrderProcessor` is more debuggable than `Thread-1`.
2. **Minimize lock scope** - use `synchronized` blocks over methods to lock only what's needed.
3. **Never ignore `InterruptedException`** - restore the interrupt flag or propagate it.
4. **Handle exceptions in thread pools** - `ThreadPoolExecutor` swallows exceptions; wrap `Runnable` in `try-catch` or use `Thread.UncaughtExceptionHandler`.
5. **Prefer synchronizers over `wait()`/`notify()`** - `CountDownLatch`, `Semaphore`, `CyclicBarrier` are simpler and less error-prone.
6. **Use concurrent collections** - `ConcurrentHashMap`, `CopyOnWriteArrayList` are more scalable than `Collections.synchronizedMap()`.

---

## `Thread.holdsLock()`

Returns `true` if the current thread holds the monitor for the specified object:

```java
synchronized (lock) {
    System.out.println(Thread.holdsLock(lock)); // true
}
System.out.println(Thread.holdsLock(lock)); // false
```
